# 使用二进制图片



## 一、图片与二进制

在页游时代，为了防止资源被盗取，通常的做法就是把图片等资源进行加密。所谓的加密就是打乱资源本来的存储字节，或者穿插一些东西。但是到了html5时代，发现基本都是直接加载的图片，为什么和页游时代做法不一样了呢？是不是html5不能加载解码二进制图片？当然不是。之所以不进行加密这层操作，主要是我们项目的源码完全暴露在浏览器端，根本没有什么秘密可言，即便加密了，写个脚本执行下就能拿到你的源码。但是为了满足开发者这方面的需求，我们来讲解下LayaAir3.0是如何进行二进制图片操作的。



## 二、如何加载

关于如何加载，这里我们先从原生开始，然后在过渡到LayaAir引擎，这样开发者可以理解其中的含义。以二进制流的方式加载，这里我们采用XMLHttpRequest二进制流的方式来加载。关于XMLHttpRequest的操作我们这里不在陈述，我们先按照二进制的方式来加载试试。这里我们先用js脚本进行操作。代码如下：

```javascript
var xhr = new XMLHttpRequest();
xhr.open("get", "res/atlas/comp.png", true);
xhr.responseType = "arraybuffer";
xhr.onload = function () {
    if (this.status == 200) {
        var blob = new Blob([this.response], { type: "image/png" });
        var img = document.createElement("img");
        img.onload = function (e) {
            window.URL.revokeObjectURL(img.src); // 清除释放;
        };
        img.src = window.URL.createObjectURL(blob);
        document.body.appendChild(img);
    }
}
xhr.send();
```

上面这个方法是用了浏览器自身提供的方法来把二进制转换成图片，二进制转换成图片其实还有很多种方法，比如加载进来二进制，解码成base64，然后在赋值给你img，或者把二进制数据用canvas绘制出图片，然后toDataURL赋值给你img的src等等，方法很多，我们这里就用最简单有效的办法转换图片。

图片加载完成之后，实例化一个XMLHttpRequest对象xhr ，`responseType`属性设置成 `arraybuffer`，实例化一个Blob对象`blob`，用来创建一个img标签，`window.URL.createObjectURL(blob)`创建一个指向该参数对象的URL，把创建的img对象我们添加到网页的body上进行显示。把这段代码嵌入到index.html文件中，运行可以看到网页已经正常的显示我们的图片。



## 三、Laya中如何使用

上面的简单例子我们是用的js脚本书写，那么在LayaAir3.0项目中是怎么使用的呢

在项目中的脚本中添加如下代码：

```typescript
//test.bin为二进制图片，图片加密数据是在图片的前面写入了四个字节的数据
Laya.loader.fetch("resources/res/test.bin","arraybuffer").then((res)=>{

    //获得res的ArrayBuffer数据
    let arraybuffer: ArrayBuffer = res;
    //Byte数组接收arraybuffer
    let byte:Byte = new Byte(arraybuffer);
    //从第四个字节开始读取数据
    byte.writeArrayBuffer(arraybuffer,4);
    //获得最终的ArrayBuffer
    let imageArrayBuffer = byte.buffer;
    //实例化一个Blob对象blob，用来创建一个img标签
    let imgBlob = new Blob([imageArrayBuffer], { type: "image/png" });
    
    //转换为Base64图片格式
    let reader = new FileReader();
    reader.readAsDataURL(imgBlob);
    reader.onload = (e)=> {     
        
		let sp1:Sprite = new Sprite();
        //加载Base64图片数据
        sp1.loadImage(e.target.result as string);     
        this.owner.addChild(sp1);
    }	  

});	
```

上述代码中，用 Laya.loader.fetch 加载图片二进制数据，根据自定义的规则，可以解析数据加密方式，并获得完整图片数据。在这里我们更多的介绍一下 LayaAir3.0引擎的 Laya.loader.fetch 方法。使用 Laya.loader.fetch 的好处是它是较为底层的下载资源的方法，它和load方法不同，不对返回的数据进行解析，也不会缓存下载的内容。

当选取图片数据的ArrayBuffer后，可以创建Image的Blob对象，通过FileReader类可以转换为Base64图片数据显示图片。这里不是用DOM来显示图片的，而是通过Laya.Sprite绘制。

开发者也可以使用Laya.Texture的方式，通过Laya.Sprite的drawTexture方式渲染，代码如下：

```typescript
//创建一个url对象；
var url:string = Laya.Browser.window.URL.createObjectURL(imgBlob);
//加载URL获得HTMLImageElement
Laya.loader.fetch( url,"image" ).then((res)=>{

    //创建Texture2D
    var t2d: Texture2D = new Texture2D(res.width, res.height, TextureFormat.R8G8B8A8, false, false, true);
    t2d.setImageData(res, true, false);

    //创建Texture
    var texture: Texture = new Texture(t2d);

    let sp2:Sprite = new Sprite();
    //使用Sprite对象的绘制纹理方式
    sp2.graphics.drawTexture(texture, 150, 0);
    this.owner.addChild(sp2);

});
```

当创建好Blob对象后，通过`window.URL.createObjectURL(blob)`创建一个指向该参数对象的URL，再通过Laya.loader.fetch 加载URL获得HTMLImageElement 对象，通过Laya.Texture2D 的setImageData 可以把HTMLImageElement 对象数据转换为 Laya.Texture2D，最后创建Laya.Texture来绘制

在上述代码中，也可以通过传递 Option 参数来使用Laya.loader.fetch，可以把 blob对象作为参数直接传递，代码如下：

```typescript
//创建Option
let option:any = {};
option.blob = imgBlob;
//通过传递Option参数，其中包含blob对象，来获得HTMLImageElement对象
Laya.loader.fetch( "" ,"image", null, option).then((res)=>{

    //创建Texture2D
    var t2d: Texture2D = new Texture2D(res.width, res.height, TextureFormat.R8G8B8A8, false, false, true);
    t2d.setImageData(res, true, false);

    //创建Texture
    var texture: Texture = new Texture(t2d);

    let sp2:Sprite = new Sprite();
    //使用Sprite对象的绘制纹理方式
    sp2.graphics.drawTexture(texture, 150, 0);
    this.owner.addChild(sp2);
});
```

以上方法就是二进制图片的处理方法，开发者可以根据需求制定更多的二进制数据规则，可以把很多图片打包成一个图片集合文件，一次性解析并加载等待。



