# WebSocket通信

WebSocket 是一种在单个 TCP 连接上进行全双工通信的协议。它使得客户端（如 Web 浏览器）和服务器之间能够建立持久的连接，并且双方可以在这个连接上随时互相发送数据，而不像 HTTP 那样需要客户端发起请求才能得到服务器的响应。

websocket发送数据的格式一般为二进制和字符串。LayaAir引擎已经为我们封装好了 Socket 和 Byte 类，收发数据结合Byte类就可以完成。

### 1.1 Laya.Sokcet

在LayaAir引擎中 `Socket` 就是我们使用 WebSocket 的基本类。 `Socket` 封装了 HTML5 WebSocket ，允许服务器端与客户端进行全双工（full-duplex）的实时通信，并且允许跨域通信。在建立连接后，服务器和 Browser/Client Agent 都能主动的向对方发送或接收文本和二进制数据。我们先来了解下 `Socket` 的用法

#### 1.1.1 Connect 服务器

Socket 连接服务器有三种方式：

| 方式             | 说明                                                         |
| :--------------- | :----------------------------------------------------------- |
| 构造函数传参     | 立即连接 比如 new Socket(“192.168.1.2”,8899)；注意这里的host参数没有ws前缀。 |
| connect方法      | 传递url和端口号，连接服务器；socket.connect(“192.168.0.1.2”，8989)；注意这里的host参数没有ws前缀。 |
| connectByUrl方法 | 传递整个url，比如 socket.connectByUrl(“ws://localhost:8989”)；这里有ws前缀。 |



#### 1.1.2 发送数据

发送数据很简单，只需要调用Socket的send函数即可，参数可以是string或者是ArrayBuffer。

- **发送字符串格式：**

```
this.socket.send("hello world");//这是发送字符串的形式。
```

- **发送二进制格式的数据：**

```typescript
//写入一个字节
this.byte.writeByte(1);
//写入一个int16的数据
this.byte.writeInt16(20);
//写入一个32位的浮点数据
this.byte.writeFloat32(20.5);
// 写入一个字符串；
this.byte.writeUTFString("hello");
//这里声明一个临时Byte类型
var by:Laya.Byte = new Laya.Byte();
//设置endian；
by.endian = Laya.Byte.LITTLE_ENDIAN;
//写入一个int32数据
by.writeInt32(5000);
//写入一个uint16 数据
by.writeUint16(16);
//把临时字节数据的数据写入byte中，这里注意写入的是by.buffer;
this.byte.writeArrayBuffer(by.buffer);
//这里是把字节数组的数据通过socket发送给服务器。
this.socket.send(this.byte.buffer);
//清除掉数据;方便下次读写；
this.byte.clear();
```

上面我们看到，通过一个字节数组把我们需要的数据读入一个Byte数组，最后发送给服务器的是`byte.buffer`，这是一个ArrayBuffer的数据类型。这里一定要注意send的参数是 ArrayBuffer，很多开发者可能不注意，直接传递成了Byte，导致发送数据不正确。假如写成 `this.socket.send(this.byte)`；这是错误的，这点一定要注意。



#### 1.1.3 接收数据

客户端从服务器接收到的数据都会派发到 Event.MESSAGE 监听函数中。receiveHandler的参数就是服务器发送回来的数据。可能是字符串，也可能是二进制ArrayBuffer。接收到的是字符串我们不用读，拿来直接用就可以。但是接收到的是二进制的话我们需要读取出来，转成我们需要的类型。

```typescript
 private receiveHandler(msg: any = null): void {
   ///接收到数据触发函数
   //.............这里我们假设收到的是二进制ArrayBuffer
   this.byte.clear();
   this.byte.writeArrayBuffer(msg);//把接收到的二进制数据读进byte数组便于解析。
   this.byte.pos = 0;//设置偏移指针；
   ////下面开始读取数据，按照服务器传递过来的数据，按照顺序读取
   var a:number = this.byte.getByte();
   var b:number = this.byte.getInt16();
   var c:number = this.byte.getFloat32();
   var d:string = this.byte.getString();
   var e:string = this.byte.getUTFString();
 }
```



#### 1.1.4 支持的事件类型

我们常用的基本就是连接建立成功，接收到数据，连接被关闭，出现异常后调度等

```typescript
/**
 * 连接建立成功后调度。
 * @eventType Event.OPEN
 * */
/*[Event(name = "open", type = "laya.events.Event")]*/
/**
 * 接收到数据后调度。
 * @eventType Event.MESSAGE
 * */
/*[Event(name = "message", type = "laya.events.Event")]*/
/**
 * 连接被关闭后调度。
 * @eventType Event.CLOSE
 * */
/*[Event(name = "close", type = "laya.events.Event")]*/
/**
 * 出现异常后调度。
 * @eventType Event.ERROR
 * */
/*[Event(name = "error", type = "laya.events.Event")]*/
```



#### 1.1.5 在代码中怎么使用

我们举一个简单的发送和接收数据的 WebSocket 代码示例：

```typescript
	private connect(): void {
	
		//创建Socket对象
		this.socket = new Socket();
		
		//对服务器建立连接
		this.socket.connectByUrl("ws://echo.websocket.org:80");
		
		//表示需要发送至服务端的缓冲区中的数据
		this.output = this.socket.output;
		
		//添加监听事件
		this.socket.on(Event.OPEN, this, this.onSocketOpen);
		this.socket.on(Event.CLOSE, this, this.onSocketClose);
		this.socket.on(Event.MESSAGE, this, this.onMessageReveived);
		this.socket.on(Event.ERROR, this, this.onConnectError);
	}

	//连接建立成功回调
	private onSocketOpen(e: any = null): void {
		console.log("Connected");

		// 发送字符串
		this.socket.send("demonstrate <sendString>");

		// 使用output.writeByte发送
		var message: string = "demonstrate <output.writeByte>";
		for (var i: number = 0; i < message.length; ++i) {
			// 直接写缓冲区中的数据
			this.output.writeByte(message.charCodeAt(i));
		}
		
		// 发送缓冲区中的数据到服务器
		this.socket.flush();
	}

	// 连接断开后的事件回调
	private onSocketClose(e: any = null): void {
		console.log("Socket closed");
	}

	// 有数据接收时的事件回调
	private onMessageReveived(message: any = null): void {
		console.log("Message from server:");
		if (typeof (message) == 'string') {
			console.log(message);
		}
		else if (message instanceof ArrayBuffer) {
			console.log(new Byte(message).readUTFBytes());
		}
		// 清理缓存的服务端发来的数据
		this.socket.input.clear();
	}

	// 出现异常后的事件回调
	private onConnectError(e: Event = null): void {
		console.log("error");
	}
```



### 2.2 Laya.Byte 二进制读写

在开发项目中，二进制的操作是不可或缺的。在html5时代，对二进制的支持已经有了很大的突破。但是api的繁琐，对开发者开发项目来说不太方便。在页游时代，ActionScript3.0的二进制数组ByteArray，功能完善，api操作简单易懂，因此LayaAir的Byte在参考ByteArray的同时承接了html5的TypedArray类型化数组的特点。下面看下主要的用法

#### 2.2.1 常用方法

- **构造方法**

  参数：

  `length` ：长度

  当传入length参数时，一个内部数组缓冲区被创建,该缓存区的大小是传入的length大小。

  `typedArray`：类型化数组

  当传入一个包含任意类型元素的任意类型化数组对象(typedArray) (比如 **Int32Array)**作为参数时，typeArray被复制到一个新的类型数组。typeArray中的每个值会在复制到新的数组之前根据构造器进行转化。新的生成的类型化数组对象将会有跟传入的数组相同的length(比如原来的typeArray.length==2，那么新生成的数组的length也是2，只是数组中的每一项进行了转化)。

  `ArrayBuffer`：二进制数据缓冲区。

  上面的三种方法都可以实例化一个Byte，根据参数的不同创建二进制数据。

  ```typescript
  //实例化一个二进制数组Byte
  var byte:Laya.Byte = new Laya.Byte();
  //或者传入一个类型化数组
  var uint8Byte:Uint8Array = new Uint8Array(10);
  var byte:Laya.Byte = new Laya.Byte(uint8Byte);
  //或者传入一个ArrayBuffer类型
  var buffer:ArrayBuffer = new ArrayBuffer(20);
  var byte:Laya.Byte = new Laya.Byte(buffer);
  ```

- **writeArrayBuffer**(arraybuffer:*, offset:number = 0, length:number = 0):void

  写入指定的二进制缓冲数据。指定数据的偏移量和长度，如下：

  ```typescript
  var byte:Laya.Byte = new Laya.Byte();
  var byte1:Laya.Byte = new Laya.Byte();
  byte1.writeFloat32(20.0);//写入一个四个字节的浮点数
  byte1.writeInt16(16);//写入一个两个字节的整数
  byte1.writeUTFString("hell world");//写入一个字符串；
  byte.writeArrayBuffer(byte1.buffer,6);//把byte1的数据从第六个字节开始读入byte中。省略其中的浮点数20.0和整数16
  byte.pos = 0;//
  console.log(byte.readUTFString())//从byte中读出字符串。
  ```

- **读取数据**

  `getByte():number`

  从字节流中读取带符号的字节。

  `getInt16():number`

  从字节流的当前字节偏移量位置处读取一个 Int16 值。

  `getInt32():number`

  从字节流的当前字节偏移量位置处读取一个 Int32 值。

  `getFloat32():number`

  从字节流的当前字节偏移位置处读取一个 IEEE 754 单精度（32 位）浮点数。

  `getFloat32Array(start:number, len:number)any`

  从指定的位置读取指定长度的数据用于创建一个 Float32Array 对象并返回此对象。

  `getFloat64():number`

  从字节流的当前字节偏移量位置处读取一个 IEEE 754 双精度（64 位）浮点数。

  `getInt16():number `

  从字节流的当前字节偏移量位置处读取一个 Int16 值。

  `getInt32():number`

  从字节流的当前字节偏移量位置处读取一个 Int32 值。

  `getUint8():number`

  从字节流的当前字节偏移量位置处读取一个 Uint8 值。

  `getUint16():number`

  从字节流的当前字节偏移量位置处读取一个 Uint16 值。

  `getUint32():number`

  从字节流的当前字节偏移量位置处读取一个 Uint32 值。

  `getInt16Array(start:number, len:number):any`

  从指定的位置读取指定长度的数据用于创建一个 Int16Array 对象并返回此对象。

  `getString():string`

  读取字符型值。

  `getUTFBytes(len:number = -1):string `

  读字符串，必须是 writeUTFBytes 方法写入的字符串。

  `getUTFString():string `

  读取 UTF-8 字符串。

  

- **写入数据**

  `writeByte(value:number):void`在字节流中写入一个字节。	

```typescript
 var byte:Laya.Byte = new Laya.Byte(); 
 byte.writeByte(10);//0-255之间
```

​	  `writeFloat32(value:number):void`在当前字节偏移量位置处写入 Float32 值。范围是$\left[-2^{128}, 2^{127}\right]$，约为-3.4E38—3.4E+38。

```typescript
var byte:Laya.Byte = new Laya.Byte();
byte.writeFloat32(10.021);
```

​	  `writeFloat64(value:number):void`写入float64位数值 其数值范围为-1.7E308～1.7E+308。

​	  `writeInt16(value:number):void`在当前字节偏移量位置处写入 Int16 值。范围-32768 到 +32767之间。	

```typescript
var byte:Laya.Byte = new Laya.Byte();
byte.writeInt16(120);
```

​	  `writeInt32(value:number):void`在当前字节偏移量位置处写入 Int32 值。-2,147,483,648 到 +2,147,483,647 之间的有符号整数。

```typescript
 **writeUint16**(value:number):void在当前字节偏移量位置处写入 Uint16 值。
```

​	  `writeUint32(value:number):void`在当前字节偏移量位置处写入 Uint32 值。

​	  `writeUint8(value:number):void`在当前字节偏移量位置处写入 Uint8 值。

​	  `writeUTFBytes(value:string):void`写入字符串，该方法写的字符串要使用 readUTFBytes 方法读取。

​	  `writeUTFString(value:string):void`将 UTF-8 字符串写入字节流。



- `clear():void`清除数据。

  ```typescript
  var byte:Laya.Byte = new Laya.Byte();
  byte.writeInt16(120);
  byte.pos =0;//读取位置归零。
  ```

  

- `getSystemEndian():string[static]`获取系统的字节存储顺序。

  ```typescript
  console.log(Laya.Byte.getSystemEndian());//打印系统的字节顺序
  ```



#### 2.2.2 属性

- `BIG_ENDIAN : string= bigEndian[static]` 表示多字节数字的最高有效字节位于字节序列的最前面。

- `LITTLE_ENDIAN : string= littleEndian[static]` 表示多字节数字的最低有效字节位于字节序列的最前面。

- `pos `number当前读取到的位置。

  ```typescript
  var byte:Laya.Byte = new Laya.Byte();
  byte.writeInt16(120);
  byte.pos =0;//读取位置归零。
  ```

- `length: number`字节长度。

- `endian : string`字节顺序。

  ```typescript
  var byte:Laya.Byte = new Laya.Byte();
  byte.endian = Laya.Byte.BIG_ENDIAN;//设置为大端；
  ```

- `bytesAvailable : number[read-only]`可从字节流的当前位置到末尾读取的数据的字节数。

  ```typescript
  var byte:Laya.Byte = new Laya.Byte();
  byte.writeFloat32(20.0);
  byte.writeInt16(16);
  byte.writeUTFString("hell world");
  byte.pos = 6;
  console.log(byte.bytesAvailable)
  ```



#### 2.2.3 代码演示

下面我们通过一个完整的代码来演示下这个类的应用，比如网络连接中，我们接收和发送网络消息。

```typescript
var msg:any ={name:"xxx",age:18,weight:65.5,height:175};
var byte:Laya.Byte = new Laya.Byte();
//实例化byte数组
byte.endian = Laya.Byte.LITTLE_ENDIAN;
//设置大小端
byte.writeUTFString(msg.name);
//写入数据
byte.writeByte(msg.age);
byte.writeFloat32(msg.weight);
byte.writeInt16(msg.height);
```

输出看下结果：

```typescript
//设置pos为0 开始从头开始按照写入的顺序读取读取
byte.pos = 0;
console.log(byte.getUTFString());
console.log(byte.getByte());
console.log(byte.getFloat32());
console.log(byte.getInt16());
```



#### 2.2.4 类型化数组

Laya的byte封装的就是类型化数组，开发者可以参考mdn的官方api说明。来扩展自己的项目的应用。

- [DataView ](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/DataView)视图提供了一个与平台中字节在内存中的排列顺序(字节序)无关的从[`ArrayBuffer`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/ArrayBuffer)读写多数字类型的底层接口。
- [Uint8Array](https://developer.mozilla.org/zh_CN/docs/Web/JavaScript/Reference/Global_Objects/Uint8Array) 数组类型表示一个8位无符号整型数组，创建时内容被初始化为0。创建完后，可以以对象的方式或使用数组下标索引的方式引用数组中的元素。
- **Int8Array** :类型数组表示二进制补码8位有符号整数的数组。内容初始化为0。 一旦建立，你可以使用对象的方法引用数组中的元素，或使用标准数组索引语法。
- **Int16Array()**;类型数组表示二进制补码16位有符号的数组。
- **Uint16Array()**;类型数组表示二进制补码16位无符号的数组
- **Int32Array()**;类型数组表示二进制补码32位有符号的数组
- **Uint32Array()**;类型数组表示二进制补码32位无符号的数组
- **Float32Array()**;类型数组表示32位浮点数数组。
- **Float64Array()**;类型数组表示64位浮点数数组。



