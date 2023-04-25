# 实体组件系统（ECS）

> Author：Charley 、谷主、孟星煜

## 一、 什么是ECS

ECS是Entity-Component-System（实体-组件-系统）的简写，这是一种基于数据驱动的游戏设计模式。

LayaAir的ECS，将场景中每一个有着唯一ID的显示对象节点都被看做一个个的实体。每一个实体都可以为其添加一个或多个不同的组件系统脚本。

在这里，组件系统是组件与系统两个组成部分，组件只包含数据，不包含逻辑，游戏对象的逻辑行为由系统控制，所以系统是实体的逻辑控制部分，组件是系统与外界的数据接口部分。LayaAir通过装饰器将接口暴露在IDE中，方便开发者直观的传入数据。引擎为组件系统提供的生命周期方法与的事件方法可以作为系统逻辑控制的入口。

开发者通过继承引擎的组件脚本类`Laya.Script`，可以实现组件系统脚本的完整功能，**我们通常将组件系统脚本简称为组件脚本**。然后通过IDE或者代码的方式添加到实体上，实现完整的ECS功能。

> 组件脚本，在原则上是解耦且具有单一职责的，这样方便多个实体可以共用同一个组件系统。



## 二、组件脚本的内置方法

继承引擎的组件脚本类`Laya.Script`之后，就可以直接使用引擎为组件脚本内置的生命周期方法与事件方法，这些方法可以用于组件脚本逻辑的执行入口。如下图所示：

![]()

### 2.1 组件的生命周期方法

生命周期方法是指在物体的创建、销毁、激活、禁用等过程中，会自动调用的方法。当使用自定义的组件脚本时，可以实现如下生命周期方法，方便快速开发业务逻辑。可以在每个方法中打印一条日志，方便开发者进行测试。

```typescript
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
     * 第一次执行update之前执行，只会执行一次
     */
    onStart?(): void {
        console.log("Game onStart");
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



事件方法是指在某些特定的情况下，会根据条件自动触发的方法，例如碰撞事件只有在物体发生碰撞时才会触发。当使用自定义的组件脚本时，可以实现如下事件方法，方便快速开发业务逻辑。

**1，物理事件**

```typescript
    /**
     * 开始碰撞时执行
     */
    onTriggerEnter(other: Laya.PhysicsComponent | Laya.ColliderBase, self?: Laya.ColliderBase, contact?: any): void {
    }

    /**
     * 持续碰撞时执行
     */
    onTriggerStay(other: Laya.PhysicsComponent | Laya.ColliderBase, self?: Laya.ColliderBase, contact?: any): void {
    }

    /**
     * 结束碰撞时执行
     */
    onTriggerExit(other: Laya.PhysicsComponent | Laya.ColliderBase, self?: Laya.ColliderBase, contact?: any): void {     
    }

    /**
     * 开始碰撞时执行
     */
    onCollisionEnter(collision: Laya.Collision): void {
    }

    /**
     * 持续碰撞时执行
     */
    onCollisionStay(collision: Laya.Collision): void {
    }

    /**
     * 结束碰撞时执行
     */
    onCollisionExit(collision: Laya.Collision): void {
    }

    /**
     * 关节破坏时执行此方法
     */
    onJointBreak(): void {
    }
```

下面以一个小球碰撞的例子，演示物理事件。以下是程序中碰撞部分的代码片段：

```typescript
	//碰撞进入后，物体改变颜色
    public onTriggerEnter(other:Laya.PhysicsComponent):void {
		(this.owner.getComponent(Laya.MeshRenderer).material as Laya.BlinnPhongMaterial).albedoColor = new Laya.Color(0.0, 1.0, 0.0, 1.0);//绿色
	}
	
    //持续碰撞时，打印日志
	public onTriggerStay(other:Laya.PhysicsComponent):void {
        console.log("peng");
	}
	
    //碰撞离开后，物体变回原本颜色
	public onTriggerExit(other:Laya.PhysicsComponent):void {
		(this.owner.getComponent(Laya.MeshRenderer).material as Laya.BlinnPhongMaterial).albedoColor = new Laya.Color(1.0, 1.0, 1.0, 1.0);//白色
	}
```

如动图2-1所示，开始碰撞时执行onTriggerEnter，小球和立方体进入碰撞，小球变为绿色；持续碰撞时执行onTriggerStay，打印日志“peng”；碰撞离开后执行onTriggerExit，小球变为原来的颜色，立方体变为白色。

![2-1](images/2-1.gif)

（动图2-1）



**2，鼠标事件**

```typescript
    /**
     * 鼠标按下时执行
     */
    onMouseDown(evt: Laya.Event): void {
    }

    /**
     * 鼠标抬起时执行
     */
    onMouseUp(evt: Laya.Event): void {
    }

    /**
     * 鼠标右键或中键按下时执行
     */
    onRightMouseDown(evt: Laya.Event): void {
    }

    /**
     * 鼠标右键或中键抬起时执行
     */
    onRightMouseUp(evt: Laya.Event): void {
    }

    /**
     * 鼠标在节点上移动时执行
     */
    onMouseMove(evt: Laya.Event): void {
    }

    /**
     * 鼠标进入节点时执行
     */
    onMouseOver(evt: Laya.Event): void {
    }

    /**
     * 鼠标离开节点时执行
     */
    onMouseOut(evt: Laya.Event): void {
    }

    /**
     * 鼠标按住一个物体后，拖拽时执行
     */
    onMouseDrag(evt: Laya.Event): void {
    }

    /**
     * 鼠标按住一个物体，拖拽一定距离，释放鼠标按键后执行
     */
    onMouseDragEnd(evt: Laya.Event): void {
    }

    /**
     * 鼠标点击时执行
     */
    onMouseClick(evt: Laya.Event): void {
    }

    /**
     * 鼠标双击时执行
     */
    onMouseDoubleClick(evt: Laya.Event): void {
    }

    /**
     * 鼠标右键点击时执行
     */
    onMouseRightClick(evt: Laya.Event): void {
    }
    
```

下面以onMouseDown和onMouseUp为例，在自定义的组件脚本“Script.ts”中加入以下代码：

```typescript
const { regClass, property } = Laya;

@regClass()
export class Script extends Laya.Script {
    /**
     * 鼠标按下时执行
     */
	onMouseDown(evt: Laya.Event): void {
        console.log("onMouseDown");
    }
    /**
     * 鼠标抬起时执行
     */
    onMouseUp(evt: Laya.Event): void {
        console.log("onMouseUp");
    }
}
```

如图2-2所示，将组件脚本添加到Scene2D的属性面板后，先不勾选 Mouse Through，因为如果勾选它，Scene2D下鼠标事件将不会响应。如果是一个3D场景，它会传递到Scene3D中。

![2-2](images/2-2.png)

（图2-2）

运行项目，如动图2-3所示，当鼠标按下时执行onMouseDown，打印“onMouseDown”；松开鼠标，鼠标弹起时执行onMouseUp，打印“onMouseUp”。

![2-3](images/2-3.gif)

（动图2-3）



**3，键盘事件**

```typescript
    /**
     * 键盘按下时执行
     */
    onKeyDown(evt: Laya.Event): void {
    }

    /**
     * 键盘产生一个字符时执行
     */
    onKeyPress(evt: Laya.Event): void {
    }

    /**
     * 键盘抬起时执行
     */
    onKeyUp(evt: Laya.Event): void {
    }
```

> 注意：onKeyPress是产生一个字符时执行，例如字母“a”、“b”，“c”等。像上、下、左、右键，F1、 F2等不是字符输入的按键，就不会执行此方法。











## 三、组件在IDE的暴露方式

在LayaAir 3.0 IDE中，如果想在IDE内展示组件脚本的属性，需要通过装饰器的规则来实现。

### 3.1 组件脚本的识别`@regClass()`

开发者编写的组件脚本，需要在类定义之前使用装饰器的标识`@regClass()`，示例代码如下所示：

```typescript
const { regClass } = Laya;

@regClass()
export class Script extends Laya.Script {
}
```

只有使用了上述的这个装饰器标识，开发者自定义的组件脚本才会被IDE识别为组件，可以被节点（实体）的`属性面板 -> 增加组件 -> 自定义组件脚本`所添加。如动图1-1所示。

![1-1](小孟：这里放一个点开自定义组件脚本的动图效果)

（动图1-1）

> [!Tip]
>
> 一个TS文件只能有一个类使用@regClass() 
>
> 标记了@regClass()的类，在IDE环境内都会被编译，但最终发布时，如果这个类没有被其他类引用，也没有被添加到节点上，或者所在的预制体/场景没有发布，则这个类会被裁剪。

### 3.2 组件属性的识别`@property()`

#### 3.2.1 组件属性的常规使用

当开发者想将组件的属性，通过IDE暴露给外界编辑来传入数据。需要在类属性定义之前使用装饰器的标识`@property()`，示例代码如下所示：

```typescript
const { regClass, property } = Laya;

@regClass()
export class NewScript1 extends Laya.Script {
    //装饰器属性的标准写法，适用于IDE的需要显示Tips或属性的中文别名等完整功能需求
    @property({ type: String, caption: "IDE显示用的别名", tips: "这是一个文本对象，只能输入文本哦" }) 
    public text1: string = "";

    //装饰器属性类型的简写方式，适用于只定义类型的需求
    @property(String)   
    public text2: string = "";

    constructor() {
        super();
    }
}
```
`@property()`是IDE识别组件属性并显示到IDE属性面板上的装饰器标识，类型是装饰器属性标识必须携带的参数。

如果我们不需要给属性写一个tips说明，也不需要给属性重新定义一个在IDE里显示的别名，等需求。那按上面示例的简写方式即可。

> 如果简写方式有语法警告，请用新版本IDE，并通过IDE的`开发者 -> 更新引擎d.ts文件`功能来解决，或者使用标准写法来解决。

#### 3.2.2 属性访问器的装饰器使用

有的时候，开发者会通过属性访问器(getter)和属性设置器(setter)来控制属性的读写行为。

当属性访问器和属性设置器同时存在时，装饰器的属性标识`@property()`直接用于属性访问器之前即可，此时的组件属性与上一小节中介绍的常规使用方式一样，都是可读写的。

如果，该脚本只有只有属性访问器，那这个属性则是只读的，仅可以在IDE中显示，但不能编辑。

getter和setter同时存在的装饰器使用示例代码如下：

```typescript
const { regClass, property } = Laya;

@regClass()
class Animal {
    private _weight: number = 0;
    
    @property( { type : Number } )
    get weight() : number {
        return this._weight;
    }
    
    set weight(value: number) {
        this._weight = value;
    }
}
```
#### 3.2.3 是否序列化保存

通过装饰器定义为组件属性后，默认状态下，属性名与值都会被序列化保存到组件被添加的场景文件或预制体文件里。例如，scene.ls里添加完自定义组件，通过vscode打开这个scene.ls，可以找到序列化保存后的组件属性名称与值，效果如图1-2所示。

（这里配个图）

![]()

序列化保存后，不仅方便在IDE中直观查看与编辑组件属性值。在运行阶段，也可以直接使用序列化存储的值，对于结构复杂的数据，直接使用序列化的值还可以节省数据结构生成带来的开销。所以，有些时候，即便是不需要在属性面板上显示与编辑，也可以通过装饰器设置为组件属性，将值序列化存储在场景或预制体文件中。

但是，也有的时候，我们的组件属性只是为了方便在IDE中理解与调整，在使用的时候，这些值其实用不到，所以，还提供了是否序列化保存的控制。当装饰器属性定义的时候，**对象参数中传入serializable为false，那么该属性就不会被序列化**。

例如，开发者的需求是序列化保存弧度值，但弧度值在人为调整数值的时候并不直观，此时，可以在IDE里直接输入角度值但不保存，仅将转换后的弧度值存起来。示例代码如下：

```typescript
const { regClass, property } = Laya;

@regClass()
export class Main extends Laya.Script {
    @property({ type: Number })
    _radian: number = 0;  //带下划线的属性，默认不会出现在IDE的属性面板上，只是用来存储输入的弧度
    
    @property({ type: Number, caption: "角度", serializable: false }) //这里设置serializable为false，所以degree不会被保存到场景文件中
     get degree() {
        return this._radian * (180 / Math.PI);//由于自己没有序列化保存，需要把_radian存下来的弧度反算回角度，用于IDE属性面板显示
    }
    set degree(value: number) {
        this._radian = value * (Math.PI / 180);//把输入的角度值，转成弧度给_radian存起来。
    }
    
    onStart() {
        console.log(this._radian); 
    }
}
```

#### 3.2.4 组件属性是否在IDE中显示

在默认情况下，装饰器属性规则只会对非下划线的类属性标记为IDE的组件属性。

对于有下划线的属性，其实是不会被显示到IDE里，此时该组件属性的价值只剩下将值保存到场景文件中了，这一点上文有所提及，示例也有应用。

> 带下划线的属性如果没有序列化保存到场景文件的需求，那就不必使用装饰器了。

假如，开发者想对有下划线的属性，也要显示到IDE上，也可以做到。将修饰器属性标识的传入对象中，设置参数private为false即可。、

示例代码如下：

```typescript
@property({ type: "number", private: false })
_velocity: number = 0;
```

private参数不仅可以使得下划线属性显示，也可以通过将private设置为true，使得不带下划线的属性，不在IDE的属性面板出现。

这里，我们将前文的弧度转换示例稍作修改，代码如下：

```typescript
const { regClass, property } = Laya;

@regClass()
export class Main extends Laya.Script {
    @property({ type: Number , private: true })
    radian: number = 0;  //private设置为true之后，radian不会出现在IDE的属性面板上，只是用来存储输入的弧度
    
    @property({ type: Number, caption: "角度", serializable: false }) //这里设置serializable为false，所以degree不会被保存到场景文件中
     get degree() {
        return this.radian * (180 / Math.PI);//由于自己没有序列化保存，需要把radian存下来的弧度反算回角度，用于IDE属性面板显示
    }
    set degree(value: number) {
        this.radian = value * (Math.PI / 180);//把输入的角度值，转成弧度给radian存起来。
    }
    
    onStart() {
        console.log(this._radian); 
    }
}
```

#### 3.2.5 装饰器属性标识的类型

装饰器属性标识的类型支持引擎对象类型（例如：Laya.Vector3、Laya.Sprite3D、Laya.Camera、等）、自定义的对象类型（需要标记`＠regClass()`）、以及TS语言的基本类型。

**引擎对象类型**的理解比较简单，暴露组件属性之后，直接传入对应类型的值就可以。例如Laya.Sprite3D就只能传入3D节点，2D节点或试图拖入资源都是禁止的。

常用的引擎对象类型使用示例如下：

> (小孟：除了测试用法，比如组件类型等，每一个不同类型的类型，要加上获取的使用示例，相同的给一个就行，比如节点的。也再想想还有没有其它常用的，实在想不到就算了，一定要保障示例可用。)，改完删除

```typescript
const { regClass, property } = Laya;

@regClass()
export class Main extends Laya.Script {

    @property( { type:Laya.Camera } ) //摄像机类型
    private camera: Laya.Camera;  

    @property( { type:Laya.Scene3D } ) //3D场景根节点类型
    private scene3D: Laya.Scene3D;

    @property( { type:Laya.DirectionLightCom } ) //DirectionLight组件类型
    private directionLight: Laya.DirectionLightCom;

    @property( { type:Laya.Sprite3D } ) //Sprite3D节点类型
    private cube: Laya.Sprite3D;  

    @property( { type:Laya.Prefab } ) //加载 Prefab 拿到的对象
    private prefabFromResource: Laya.Prefab;    

    @property( { type:Laya.ShurikenParticleRenderer } ) //ShurikenParticleRenderer组件类型
    private particle3D: Laya.ShurikenParticleRenderer;  

    @property( { type:Laya.Node } ) //节点类型
    private scnen2D: Laya.Node; 

    @property( { type:Laya.Box } ) //拿到 Box 组件
    private box: Laya.Box; 

    @property( { type:Laya.List } ) //拿到 List 组件
    private list: Laya.List; 

    @property( { type:Laya.Image } ) //拿到 Image 组件
    private image: Laya.Image; 

    @property( { type:Laya.Label } ) //拿到 Label 组件
    private label: Laya.Label; 

    @property( { type:Laya.Button } ) //拿到 Button 组件
    private button: Laya.Button; 

    @property( { type:Laya.Sprite } ) //拿到 Sprite 组件
    private sprite: Laya.Sprite; 

    @property( { type:Laya.Animation } ) //拿到 Animation 组件
    private anmation: Laya.Animation; 

    @property( { type:Laya.Vector3 } ) //Laya.Vector3类型
    private vector3 : Laya.Vector3;
}
```

**自定义对象类型**，就是设置一个自定义的引入对象。按该对象的装饰器属性标识来暴露组件属性。

例如，下面这两个TS代码：

```typescript
//MyScript.ts
const { regClass, property } = Laya;

import Animal from "./Animal";

@regClass()
export class MyScript extends Laya.Script  {
    @property({ type : Animal })
    animal : Animal;
}
```

```typescript
//Animal.ts
const { regClass, property } = Laya;

@regClass()
export default class Animal {
    @property({ type : Number })
    weight : number;
}
```

组件脚本MyScript中引用了Animal对象 ，并将装饰器属性标识的类型设置为Animal，尽管Animal不是继承于Laya.Script的组件脚本，但由于被组件脚本MyScript所引用并暴露给IDE，所以Animal类定义之前也需要标记`＠regClass()`，该类下使用了`@property()`标识的属性，也可以出现在IDE属性面板中。

最后就是常用的**TS语言基本类型**，不过需要注意的是，基本类型需要使用字符串的方式来描述，只有数字、字符串、布尔类型，可以用其对象类型来标记。

| 类型               | 类型书写示范                                                 | 类型说明                                            |
| ------------------ | ------------------------------------------------------------ | --------------------------------------------------- |
| 数字类型           | "number"                                                     | 也可以用Number来标记该类型                          |
| 单行字符串文本类型 | "string"                                                     | 也可以用String来标记该类型                          |
| 布尔值类型         | "boolean"                                                    | 也可以用Boolean来标记该类型                         |
| 整数类型           | "int"                                                        | 等价于 { type: Number, fractionDigits: 0 }          |
| 正整数类型         | "uint"                                                       | 等价于 { type: Number, fractionDigits: 0 , min: 0 } |
| 多行字符串文本类型 | "text"                                                       | 等价于 { type: string, multiline: true }            |
| 任意类型           | "any"                                                        | 类型只会被序列化，不能显示和编辑。                  |
| 类型化数组类型     | Int8Array、Uint8Array、<br />Int16Array、Uint16Array、<br />Int32Array、Uint32Array、Float32Array、<br />Float64Array | 支持8种类型化数组类型                               |
| 数组类型           | ["number"]、["string"]                                       | 用中括号包含数组元素类型，                          |

使用示例代码如下：(补充上上面表格的示例代码)

```typescript
const { regClass } = Laya;
//枚举
enum TestEnum {
    A,
    B,
    C
};
//字符串形式的枚举
enum Direction {
    Up = 'UP',
    Down = 'DOWN',
    Left = 'LEFT',
    Right = 'RIGHT'
};
@regClass()
export class Script extends Laya.Script {
    @property(String)//单行字符串文本类型，等价于 { type: "string"}
	str : string;
    
    @property("any")//注意：any类型只会被序列化，不能显示和编辑。
	a : any; 

	@property("int")//整数类型，等价于 { type: Number, fractionDigits: 0 }
	int : number; 

    @property("uint") //正整数类型，等价于 { type: Number, fractionDigits: 0 , min: 0 }
    uint : number; 

    @property("text")//多行字符串文本类型，等价于 { type: string, multiline: true }
    text : string; 
    
    @property(Int8Array)//类型化数组类型,除了Int8Array，还支持Uint8Array、Int16Array、Uint16Array、Int32Array、Uint32Array、Float32Array、Float64Array，使用方式都类似
    i8a: Int8Array;
        
    @property({ type: ["number"] })//数组类型，用中括号包含数组元素类型
    arr1: number[];

    @property({ type: ["string"] })//数组类型，用中括号包含数组元素类型
    arr2: Array<string>;
    
    //普通的枚举类型（可以类型简写），会显示为下拉框供用户选择
    @property(TestEnum)
    enum: TestEnum;
    
	//字符串形式的枚举，不能使用类型简写，如：@property(Direction)。必须下面带type参数指定的标准写法
    @property({ type: Direction })
    direc: Direction;
    
    //字典类型，需要用数组参数来设置类型，下面示例中的Record类型需要放到字符串内作为数组参数的第一个元素，数组参数的第二个元素是字典输入值的类型，用于决定属性面板的输入控件类型。
    @property({ type: ["Record", Number] })
    dict: Record<string, number>; 
}
```



示例效果如动图xxx所示：（动图，只演示几个不常见的复杂类型，例如字典，枚举，其它的，显示在面板上，能看到即可）

![]()

#### 3.2.6 组件属性值的输入控件

IDE内置了number（数字输入）、string（字符串输入）、boolean（多选框）、color（颜色框+调色盘+拾色器）、vec2（XY输入组合）、vec3（XYZ输入组合）、vec4（XYZW输入组合）、asset（选择资源），这些输入控件。

通常情况下，IDE会根据组件属性类型自动选择对应的属性值输入控件。

但在某些情况下，也需要强制指定输入控件。例如，数据类型是Vector4，但其实它表达的是颜色，用默认编辑Vector4的控件不适合，需要在这里设置组件属性标识的参数inspector为“color”。示例代码如下：

    @property({ type: String, inspector: "color"})
    color: string;
效果如动图XXX所示

![]()

(动图XXX)

如果inspector参数为null，则不会为属性构造属性输入控件，这与hidden参数设置为true不同。hidden为true是创建但不可见，inspector为null则是完全不创建。

#### 3.2.7 组件属性分类与排序

组件的属性默认会统一显示在以组件脚本名称的属性分类栏目下，效果如图XXX所示

![]()

如果开发者想对组件内的某些属性进行归类，可以通过装饰器属性标识的对象参数catalog来实现，示例代码如下：

```typescript
    @property({ type: String, inspector: "color" ,catalog:"adv"})
    d: string;
```

通过上面的代码可以看出，当为多个属性设置相同的catalog名称，就会按catalog名称进行分类。效果如图xxx所示：

![]()

如果我们想给这个分类再起个中文别名，可以通过参数catalogCaption来实现，示例代码如下：

```typescript
    @property({ type: String, inspector: "color" ,catalog:"adv", catalogCaption:"高级组件"})
    d: string;
```

效果如图xxx所示：

![]()

在面对多个组件属性分类的时候，我们还可以通过参数catalogOrder对栏目的显示顺序自定义排序，示例代码如下：

```typescript
       //显示为颜色输入（如果类型是Laya.Color，则不需要这样定义，如果是字符串类型，则需要）
    @property({ type: String, inspector: "color" ,catalog:"adv", catalogCaption:"高级组件2",catalogOrder:2})
    d: string;

    @property({ type: _Int8Array, private: false ,catalog:"adv"})
    _Int8Array: Int8Array;

    //普通的枚举类型（可以类型简写），会显示为下拉框供用户选择
    @property(TestEnum)
    enum: TestEnum;

    //重要提醒：字符串形式的枚举，不能使用类型简写，如：@property(Direction)。必须下面带type参数指定的标准写法
    @property({ type: Direction,catalog:"bb"})
    direc: Direction;

    //装饰器属性的标准写法，适用于IDE的需要显示Tips或属性的中文别名等完整功能需求
    @property({ type: String, caption: "IDE显示用的别名", tips: "这是一个文本对象，只能输入文本哦",catalog:"bb" , catalogCaption:"高级组件",catalogOrder:1})
    public text1: string = "";
```

效果如图xxx所示：

![]()

>  属性分类名称catalogCaption与属性分类排序catalogOrder，在任意一个catalog相同名称的属性里配置即可，无需所有的属性都配置一次。

#### 3.2.8 装饰器属性标识参数总结

上文介绍了常用装饰器属性标识的参数作用，这里我们概述总结一下全部的参数。

| 参数名         | 参数使用示例                                             | 说明                                                         |
| -------------- | -------------------------------------------------------- | ------------------------------------------------------------ |
| type           | type: "string"                                           | 组件属性可输入值的类型，参照上文的介绍。                     |
| caption        | caption: "角度"                                          | 组件属性的别名，常用于中文，可以不设置，默认会用组件属性名   |
| tips           | tips: "这是一个文本对象，只能输入文本哦"                 | 组件属性的Tips说明，用于进一步描述该属性的作用等用途         |
| private        | private：false                                           | 控制组件属性是否显示在IDE里，false：显示，true：不显示       |
| serializable   | serializable： false                                     | 控制组件属性是否序列化保存，true：序列化保存，false：不序列化保存 |
| enumSource     | enumSource: [{name:"Yes", value:1}, {name:"No",value:0}] | 组件属性以下拉框的形式来展示与输入值。                       |
| inspector      | inspector: "color"                                       | 属性值输入控件，内置有：number,string,boolean,color,vec2,vec3,vec4,asset。 |
| range          | range: [0, 5]                                            | 数字类型时，组件属性在一个范围内以滑动杆的方式显示与输入值。 |
| step           | step: 0.5                                                | 数字类型时，在输入框的鼠标滑动或滚轮滚动的最小更改精度值。   |
| fractionDigits | fractionDigits: 3                                        | 数字类型时，属性值的小数点后保留几位。                       |
| multiline      | multiline: true                                          | 字符串类型时，是否为多行输入，true：是，false：不是。        |
| onChange       | onChange: "onChangeTest"                                 | 当属性改变时，调用名称为onChangeTest的函数。函数需要在当前组件类上定义 |

> 表格与代码都要继续补充，打开引擎源码，查看Decorators.ts的PropertyDescriptor接口说明

代码示例如下：

```typescript
const { regClass, property} = Laya;

import Animal from "./Animal"

enum TestEnum {
    A,
    B,
    C
};

enum StringEnum {
    A = "a",
    B = "b",
    C = "c"
};

@regClass()
export class Property extends Laya.Script {

    @property( { type : String } )
    a : string;
    
    @property( { type : Laya.Vector3 } )
    b : Laya.Vector3;
    
    @property( { type : Animal } )
    c : Animal; //沿用上个例子，Animal已经被regClass
    
    @property( { type : TestEnum } )
    d : TestEnum; //枚举类型，会显示为下拉框供用户选择

    @property({ type: StringEnum })
    e : StringEnum; //对于字符串形式的枚举，不能使用@property(StringEnum),必须用集合里的type参数指定
    
    @property( { type : [Number] } )
    f : number[]; //数组，用中括号包含数组元素类型
    
    @property( { type : ["Record", String] } )
    g : Record<string, string>; //字典，中括号里第一个元素固定是字符串Record，第二个元素是字典value类型。
    
    @property( { type : "any" } )
    h : any; //any类型只会被序列化，不能显示和编辑。
    
    @property( { type : "int" } )
    i : number; //int等价于 { type: Number, fractionDigits: 0 }
    
    @property( { type : "uint" } )
    j : number; //uint等价于 { type: Number, fractionDigits: 0 , min: 0 }
    
    @property( { type : "text" } )
    k : string; //text等价于 { type: string, multiline: true }

    //设置标题
    @property({ type: String, caption: "速度", tips: "设置速度" })
    l : string;

    //显示为下拉框
    @property({ type: Number, enumSource: [{name:"Yes", value:1}, {name:"No",value:0}] })
    m : number;

    //控制数字输入的精度和范围
    @property({ type: Number, range:[0,1], factionDigits: 3 })
    n : number

    //文本显示为多行输入
    @property({ type: String, multiline: true })
    o : string;
    
        //显示为下拉框
    @property({ type: Number, enumSource: [{name:"Yes", value:1}, {name:"No",value:0}] })
    m : number;

    //显示为颜色输入（如果类型是Laya.Color，则不需要这样定义，如果是字符串类型，则需要）
    @property({ type: String, inspector: "color" })
    p: string;
    
    //当属性改变时，调用名称为onChangeTest的函数
    @property({ type: Boolean, onChange: "onChangeTest"})
    q: boolean;

    onChangeTest() {
        console.log("onChangeTest");
    }
}
```

### 3.3 IDE中执行生命周期方法`@runInEditor`

除了在IDE属性面板上暴露组件属性，开发者还可以通过装饰器标识 `@runInEditor`来让组件在IDE内加载时也可以触发生命周期方法（onEnable、onStart等所有的组件脚本生命周期方法）。示例代码如下：

```typescript
const { regClass, property, runInEditor } = Laya;

@regClass() @runInEditor     //重点看这里，要放到类之前，@regClass()与@runInEditor谁先谁后都可以。
export class NewScript2 extends Laya.Script {
    @property({ type: Laya.Sprite3D })
    sp3: Laya.Sprite3D;

    constructor() {
        super();
    }

    onEnable() {
        console.log("Game onStart", this.sp3.name);
    }
}
```

除非有特别的需求，我们并不建议这样做，一方面是因为静态物体更有利于IDE内进行编辑。另一方面是因为场景编辑器为了性能优化，帧率刷新要比正常运行慢很多，因此效果会与正常运行有明显差异。

### 3.4 加入IDE的组件列表`@classInfo()`

开发者的自定义组件默认都位于XXXX的下面，如图XXX所示。



如果我们想在这个组件列表中，将该组件加入自己定义的组件列表分类中，可以使用装饰器标识`@regClass()`示例代码如下所示：

```typescript
@classInfo( {
    menu : "MyScript",
    caption : "Main",
})
这里要用完整代码
```

然后我们保存代码，回到IDE，会发现自定义的分类已出现在组件列表中。如动图XX所示。

<img src="images/5-2.gif" style="zoom:50%;" /> 

（动图5-2）

## 四、代码中使用属性

前文已经介绍了组件组件的添加与识别。相信有一定基础的开发者已经可以直接使用LayaAir的实体组件系统了。

但针对一些新手开发者朋友，本小节通过几种常用类型的属性使用示例，进一步帮助大家理解组件化开发的基础。

### 4.1 节点类型方式

LayaAir分为2D节点与3D节点类型，当设置为2D节点Laya.Sprite时，不能将3D节点作为其属性值。当设置为3D节点Laya.Sprite3D时，不能将2D节点作为其属性值。

#### 4.1.2 2D节点的使用

#### 4.1.3 3D节点的基础使用

#### 4.1.4 3D节点的进阶使用

```typescript
    @property( { type :Laya.Sprite3D } ) //节点类型
    public p3d: Laya.Sprite3D;

    onAwake(): void {

        this.p3d.transform.localPosition = new Laya.Vector3(0,5,5);
        let p3dRenderer = this.p3d.getComponent(Laya.ShurikenParticleRenderer);
        p3dRenderer.particleSystem.simulationSpeed = 10;
    }
```

通过暴露@property( { type :Laya.Sprite3D } )节点类型属性，来拖入particle节点，可以获得particle节点对象。transform可以直接修改，而simulationSpeed属性则通过getComponent(Laya.ShurikenParticleRenderer).particleSystem的方式获取。



### 4.2 组件类型的使用

```typescript
    @property( { type : Laya.ShurikenParticleRenderer } ) //组件类型
    public p3dRenderer: Laya.ShurikenParticleRenderer;

    onAwake(): void {

        (this.p3dRenderer.owner as Laya.Sprite3D).transform.localPosition = new Laya.Vector3(0,5,5);
        this.p3dRenderer.particleSystem.simulationSpeed = 10;
    }
```

通过暴露@property( { type : Laya.ShurikenParticleRenderer } )组件类型属性，来拖入particle节点，可以获得particle的ShurikenParticleRenderer组件。transform可以通过(this.p3dRenderer.owner as Laya.Sprite3D)修改，而simulationSpeed属性则通过this.p3dRenderer.particleSystem的方式获取。

#### 下面这个，不需要专门讲，前面有讲正确的用法，最多是tips说明里一下，或者在前文的代码里，把错误的用法说一下，给注释掉。提一下不能怎么用就好。

```typescript
    @property( { type : Laya.ShuriKenParticle3D } ) //不支持的类型
    public p3d: Laya.ShuriKenParticle3D;

    onAwake(): void {

        this.p3d.transform.localPosition = new Laya.Vector3(0,5,5);
        this.p3d.particleSystem.simulationSpeed = 10;
    }
```

不能通过直接使用Laya.ShuriKenParticle3D作为属性类型，IDE无法识别，只有节点和组件类型可以识别。


```typescript
    @property( { type : Laya.Sprite3D } )
    public p3d: Laya.ShuriKenParticle3D; //无法转换成Laya.ShuriKenParticle3D

    onAwake(): void {

        this.p3d.transform.localPosition = new Laya.Vector3(0,5,5);
        this.p3d.particleSystem.simulationSpeed = 10;
    }
```

上述方式也不行，IDE虽然标识了属性是Sprite3D节点，但是也无法转换为Laya.ShuriKenParticle3D对象。

报错信息：*[Game] Uncaught (in promise) TypeError: Cannot set properties of undefined (setting 'simulationSpeed')*



### 4.3 Prefab类型属性

当使用Laya.Prefab作为属性时，例如：

```typescript
@property( { type : Laya.Prefab } ) //加载 Prefab 的对象
private prefabFromResource: Laya.Prefab;    
```

此时，需要从assets目录下，拖入prefab资源，运行时会直接获取到加载实例化后的prefab（如图1-5）。

<img src="images/1-5.png" style="zoom:50%;" /> 

（图1-5）

再想想还有没有其它常用的类型，但又不太好理解的。也可以列出来。







