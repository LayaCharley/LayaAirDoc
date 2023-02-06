# 自定义脚本组件

LayaAir2.0开始，支持自定义脚本到编辑器，方便扩展已有组件的功能。自定义脚本继承自Laya.Script类，定了组件的事件和自身生命周期方法，不需要再自己绑定事件即可快速实现逻辑。任何组件都可以使用同一自定义脚本来实现相同的功能，比如不同页面上需要对相同的组件实现同一功能。

另外不同的页面在使用同一脚本时，可以给同一个属性指定不同的组件，可通过把属性暴露给IDE，在IDE上指定不同的组件。



## 一、IDE中暴露属性

在LayaAir 3.0IDE中，如果想在IDE内展示脚本定义的属性，可用通过加入装饰器来实现。

装饰器是TypeScript的一种语言特性，它通常以@开头，出现在类定义、类属性定义或者类方法定义上面。开发者自己编写的组件或者其他的一些类，如果需要IDE识别并显示到属性面板上，或者需要序列化保存，则需要用到LayaAir提供的装饰器。



### 1.1 regClass

所有开发者编写的Laya.Script都需要使用这个装饰器才能被IDE识别。例如

```Go
const { regClass } = Laya;

@regClass()
export class MyScript extends Laya.Script {
}
```

所有需要被IDE显示和序列化保存的类型都需要使用这个装饰器。例如下面这个例子，虽然Animal不是脚本类，但它们需要被MyScript这个脚本暴露给开发者，所以也需要regClass。

```Go
//MyScript.ts
const { regClass } = Laya;

@regClass()
export class MyScript extends Laya.Script {
    @property()
    animal : Animal;
}
//Animal.ts
const { regClass } = Laya;

@regClass()
export default class Animal {
    @property()
    weight : number;
}
```

**注意：**

**一个TS文件只能有一个类使用regClass !**

标记了regClass的类，在编辑器环境内都会被编译，但是最终发布时，如果这个类没有被其他类import，也没有被添加到节点上或者所在的预制体/场景没有发布，则这个类会被裁剪。



### 1.2 property

property用在类的属性上，表示这个属性需要保存和暴露给用户编辑。

```Go
const { regClass, property } = Laya;

@regClass()
class Animal {
    @property()
    weight : number;
}
```



### 1.3 代码示例

比如下面的脚本类：

<img src="images/1-1.png" alt="image-20221101102208573" style="zoom:50%;" /> 

（图1-1）

其中：

```
const { regClass, property } = Laya;
```

写在脚本类的最上面，声明此类支持暴露类中的属性

```
@regClass()
export class Main extends Laya.Script
```

在自定义脚本类的上一行，加入@regClass()

```
@property()
private btn_1: Laya.Button; 
```

在自定义属性的上一行，加入@property()，则可以在IDE暴露此属性，如图1-2所示

<img src="images/1-2.png" alt="image-20221101103355334" style="zoom: 50%;" /> 

（图1-2）

此时可以拖拽场景中的相对应的节点到属性中，如图1-3所示

<img src="images/1-3.png" alt="image-20221101103614591" style="zoom:67%;" /> 

（图1-3）



## 二、组件的事件方法

当使用自定义脚本类时，可以实现如下事件方法，方便快速开发业务逻辑

```
    /**
     * 开始碰撞时执行
     */
    onTriggerEnter?(other: PhysicsComponent | ColliderBase, self?: ColliderBase, contact?: any): void;

    /**
     * 持续碰撞时执行
     */
    onTriggerStay?(other: PhysicsComponent | ColliderBase, self?: ColliderBase, contact?: any): void;

    /**
     * 结束碰撞时执行
     */
    onTriggerExit?(other: PhysicsComponent | ColliderBase, self?: ColliderBase, contact?: any): void;

    /**
     * 开始碰撞时执行
     */
    onCollisionEnter?(collision: Collision): void;

    /**
     * 持续碰撞时执行
     */
    onCollisionStay?(collision: Collision): void;

    /**
     * 结束碰撞时执行
     */
    onCollisionExit?(collision: Collision): void;

    /**
     * 关节破坏时执行此方法
     */
    onJointBreak?(): void;

    /**
     * 鼠标按下时执行
     */
    onMouseDown?(evt: Event): void;

    /**
     * 鼠标抬起时执行
     */
    onMouseUp?(evt: Event): void;

    /**
     * 鼠标右键或中键按下时执行
     */
    onRightMouseDown?(evt: Event): void;

    /**
     * 鼠标右键或中键抬起时执行
     */
    onRightMouseUp?(evt: Event): void;

    /**
     * 鼠标在节点上移动时执行
     */
    onMouseMove?(evt: Event): void;

    /**
     * 鼠标进入节点时执行
     */
    onMouseOver?(evt: Event): void;

    /**
     * 鼠标离开节点时执行
     */
    onMouseOut?(evt: Event): void;

    /**
     * 鼠标按住一个物体后，拖拽时执行
     */
    onMouseDrag?(evt: Event): void;

    /**
     * 鼠标按住一个物体，拖拽一定距离，释放鼠标按键后执行
     */
    onMouseDragEnd?(evt: Event): void;

    /**
     * 鼠标点击时执行
     */
    onMouseClick?(evt: Event): void;

    /**
     * 鼠标双击时执行
     */
    onMouseDoubleClick?(evt: Event): void;

    /**
     * 鼠标右键点击时执行
     */
    onMouseRightClick?(evt: Event): void;

    /**
     * 键盘按下时执行
     */
    onKeyDown?(evt: Event): void;

    /**
     * 键盘产生一个字符时执行
     */
    onKeyPress?(evt: Event): void;

    /**
     * 键盘抬起时执行
     */
    onKeyUp?(evt: Event): void;
```



## 三、组件的生命周期方法

当使用自定义脚本类时，可以实现如下生命周期方法，方便快速开发业务逻辑

```
    /**
     * 被添加到节点后调用，和Awake不同的是即使节点未激活onAdded也会调用。
     */
    onAdded(): void {
    }

    /**
     * 重置组件参数到默认值，如果实现了这个函数，则组件会被重置并且自动回收到对象池，方便下次复用
     * 如果没有重置，则不进行回收复用

     */
    onReset?(): void;

    /**
     * 组件被激活后执行，此时所有节点和组件均已创建完毕，次方法只执行一次
     */
    onAwake(): void {
    }

    /**
     * 组件被启用后执行，比如节点被添加到舞台后
     */
    onEnable(): void {
    }

    /**
     * 第一次执行update之前执行，只会执行一次
     */
    onStart?(): void;

    /**
     * 每帧更新时执行，尽量不要在这里写大循环逻辑或者使用getComponent方法
     */
    onUpdate?(): void;

    /**
     * 每帧更新时执行，在update之后执行，尽量不要在这里写大循环逻辑或者使用getComponent方法
     */
    onLateUpdate?(): void;

    /**
     * 渲染之前执行
     */
    onPreRender?(): void;

    /**
     * 渲染之后执行
     */
    onPostRender?(): void;

    /**
     * 组件被禁用时执行，比如从节点从舞台移除后
     */
    onDisable(): void {
    }

    /**
     * 手动调用节点销毁时执行
     */
    onDestroy(): void {
    }
```

### 3.1 参考示例

如下示例，在代码中实现了自定义脚本的暴露属性的标识符，生命周期方法onStart()和事件mouseDown()的实现

<img src="images/3-1.png" alt="image-20221031205632806" style="zoom: 25%;" /> 

（图3-1）

```
const { regClass, property } = Laya;
import { Particle3D } from "./Particle3D";
import Sprite3D = Laya.Sprite3D;
import Button = Laya.Button;
import Event = Laya.Event;

@regClass()
export class Main extends Laya.Script {

    @property()
    private btn_1: Button;  
    @property()
    private p_1 : Sprite3D;                

    private particleList: Array<Sprite3D> = [];

    onStart() {
        console.log("Game start");

        this.particleList.push(this.p_1);
        
        this.btn_1.on(Event.MOUSE_DOWN, this, () => {
            this.hideAll();
            this.p_1.active = true;
        });                                    
    }

    hideAll(): void {
        this.particleList.forEach(element => {
            (element as Sprite3D).active = false;
        });
    }

    mouseDown(e: Event): void {
        this.hideAll();           
    }
 
}
```



## 四、IDE中执行生命周期方法（runInEditor）

在了解regClass和property后，开发者还可以加入 **runInEditor **装饰器来指定组件在编辑器内加载时是否触发生命周期函数（onEnable，onStart等）。默认是不触发的。

如图4-1所示，通过添加@property代码，创建了一个cube属性

<img src="images/4-1.png" alt="image-20230206173238445" style="zoom:50%;" /> 

（图4-1）

在IDE中，添加一个Cube，并将Cube拖入到上述cube属性中，如图4-2所示

![image-20230206173443695](images/4-2.png)

（图4-2）

此时，在IDE中Cube是静止不动的，我们为Cube添加一些代码，让Cube可以围绕自身旋转，代码如下所示：

```
const { regClass, property } = Laya;

@regClass()
export class Main extends Laya.Script {

    @property()
    cube : Laya.Sprite3D;

    private rotation: Laya.Vector3 = new Laya.Vector3(0, 0.01, 0);

    onStart() {
        console.log("Game onStart");
        Laya.timer.frameLoop(1, this, ()=> {
            this.cube.transform.rotate(this.rotation, false);
        });
    }
}
```

通过IDE的运行，可以看到如下效果，如动图4-3所示

<img src="images/4-3.gif" style="zoom:50%;" /> 

（动图4-3）

### 实时看到预览效果

通常每次都需要运行才能在IDE中看到效果，如果想在IDE中实时看到预览效果，需要怎么做呢？如图4-4所示

<img src="images/4-4.png" style="zoom:50%;" /> 

（图4-4）

其中：

```
const { regClass, property, runInEditor } = Laya;
```

写在脚本类的最上面的声明中，加入runInEditor声明

```
@regClass()
@runInEditor
export class Main extends Laya.Script {
```

在@regClass()下，加入@runInEditor

这时就可以在IDE中看到Cube自身旋转的效果了，由于IDE的update只会在有变化的情况下调用，或者一秒几帧，因此对象的动画效果会比预览时要慢一些。

对于其它的生命周期方法同样也可以在触发时执行，开发者根据需要来添加生命周期方法

```
    /**
     * 被添加到节点后调用，和Awake不同的是即使节点未激活onAdded也会调用。
     */
    onAdded(): void {
        console.log("Game onAdded");
    }

    /**
     * 重置组件参数到默认值，如果实现了这个函数，则组件会被重置并且自动回收到对象池，方便下次复用
     * 如果没有重置，则不进行回收复用

     */
    onReset(): void{
        console.log("Game onReset");
    }

    /**
     * 组件被激活后执行，此时所有节点和组件均已创建完毕，次方法只执行一次
     */
    onAwake(): void {
        console.log("Game onAwake");
    }

    /**
     * 组件被启用后执行，比如节点被添加到舞台后
     */
    onEnable(): void {
        console.log("Game onEnable");
    }

    /**
     * 每帧更新时执行，尽量不要在这里写大循环逻辑或者使用getComponent方法
     */
    onUpdate(): void{
        console.log("Game onUpdate");
    }

    /**
     * 每帧更新时执行，在update之后执行，尽量不要在这里写大循环逻辑或者使用getComponent方法
     */
    onLateUpdate(): void{
        console.log("Game onLateUpdate");
    }

    /**
     * 渲染之前执行
     */
    onPreRender(): void{
        console.log("Game onPreRender");
    }

    /**
     * 渲染之后执行
     */
    onPostRender(): void{
        console.log("Game onPostRender");
    }

    /**
     * 组件被禁用时执行，比如从节点从舞台移除后
     */
    onDisable(): void {
        console.log("Game onDisable");
    }

    /**
     * 手动调用节点销毁时执行
     */
    onDestroy(): void {
        console.log("Game onDestroy");
    }    
```



## 五、IDE中添加类信息（classInfo）

当一个类被regClass装饰后，可以使用classInfo提供更多的选项。如图5-1所示

<img src="images/5-1.png" alt="image-20230206191353279" style="zoom:50%;" /> 

（图5-1）

通过在Main的@regClass()下加入

```Go
@classInfo( {
    menu : "MyScript",
    caption : "Main",
})
```

可以在IDE中像添加系统组件库一样添加脚本，如动图5-2所示

<img src="images/5-2.gif" style="zoom:50%;" /> 

（动图5-2）



## 六、其它property属性规则

1，如果属性只需要暴露给用户编辑，不需要序列化保存：@property( { serializable: false });

2，如果属性只需要序列化保存，不需要暴露给用户编辑，那么属性名可以使用下划线开头，也可以：@property( { "private" : true });

3，属性类型是以下类型时能自动识别：

- number
- string
- boolean
- 标记了regClass的类
- LayaAir引擎里的大部分类型

不属于以上的，包括数组、字典、枚举等，则需要额外设置type属性。例如

```Go
@property()
a : string; //自动识别

@property()
b : Laya.Vector3; //自动识别

@property()
c : Animal; //沿用上个例子，Animal已经被regClass，自动识别

enum TestEnum {
    A,
    B,
    C
};
@property({ type : TestEnum })
d : TestEnum; //Enum无法自动识别，需要自己指定type

@property({ type : [Number]})
e : number[]; //数组无法自动识别，请注意type的格式，用中括号包含数组元素类型

@property({ type: ["Record", String]})
f : Record<string, string>; //字典无法识别，请注意type的格式，中括号里第一个元素固定是字符串Record，第二个元素是字典value类型。

@property({ type : "any" })
g : any; //any类型只会被序列化，不能显示和编辑。
```

property除了设置属性类型和序列化开关外，还有丰富的样式控制。下面举一些例子，更多内容可参考装饰器注释。

```Go
//显示为下拉框
@property({ enumSource: [{name:"Yes", value:1}, {name:"No",value:0}] })
a : number;

//控制数字输入的精度和范围
@property({ range:[0,1], fractionDigits: 3 })
b : number

//文本显示为多行输入
@property({ multiline: true })
c : string;

//显示为颜色输入（如果类型是Laya.Color，则不需要这样定义，如果是字符串类型，则需要）
@property({ inspector: "color" })
d: string;

//当属性改变时，调用名称为onChangeTest的函数
@property({ onChange: "onChangeTest"})
e: boolean;
```

运行效果动图

<img src="images/6-2.gif" style="zoom:50%;" /> 

代码示例：

Property.ts

```
const { regClass, property} = Laya;

import Animal from "./Animal"

enum TestEnum {
    A,
    B,
    C
};

@regClass()
export class Property extends Laya.Script {

    @property()
    a : string; //自动识别

    @property()
    b : Laya.Vector3; //自动识别

    @property()
    c : Animal; //沿用上个例子，Animal已经被regClass，自动识别

    @property({ type : TestEnum })
    d : TestEnum; //Enum无法自动识别，需要自己指定type

    @property({ type : [Number]})
    e : number[]; //数组无法自动识别，请注意type的格式，用中括号包含数组元素类型

    @property({ type: ["Record", String]})
    f : Record<string, string>; //字典无法识别，请注意type的格式，中括号里第一个元素固定是字符串Record，第二个元素是字典value类型。

    @property({ type : "any" })
    g : any; //any类型只会被序列化，不能显示和编辑。


    @property({ enumSource: [{name:"Yes", value:1}, {name:"No",value:0}] })
    h : number;

    @property({ range:[0, 1], fractionDigits: 3 })
    i : number;
    
    //文本显示为多行输入
    @property({ multiline: true })
    j : string;
    
    //显示为颜色输入（如果类型是Laya.Color，则不需要这样定义，如果是字符串类型，则需要）
    @property({ inspector: "color" })
    k: string;
    
    //当属性改变时，调用名称为onChangeTest的函数
    @property({ onChange: "onChangeTest"})
    l: boolean;

    onChangeTest() {
        console.log("onChangeTest ");
    }
}
```
Animal.ts

```
const { regClass, property } = Laya;

@regClass()
export default class Animal {
    @property()
    weight : number;
}

```

