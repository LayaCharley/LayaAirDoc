# 使用3D UI



## 一、概述

2D UI都是纯粹的2D图片按层次显示，不会出现三维立体效果，所以都是直接紧贴着视窗上。而3D UI的原理是，创建的UI控件都在一个三维立体空间中，摄像机是一个透视的摄像机，这和2D UI有着截然不同的区别，因为2D UI是一个正交摄像机。因此如果要出现UI有三维变换的效果，就必须用3D UI

### 1.1 3D UI的本质

3D UI也是UI，就需要承担UI的交互职能。例如，当我们点击UI上的按钮时，按钮会带来交互反馈，并触发设定的事件，以达到逻辑运行的目的。

<img src="img/1-1.gif" alt="image-20221226152358932" style="zoom: 80%;" /> 

（动图1-1）



### 1.2 3D UI的分类

**场景化UI**

3D UI是位于3D场景中，不跟随窗口运动而运动的UI。它更像是一个位于3D场景中的物件，并带有UI的交互特征。

 <img src="img/1-2.gif" style="zoom:50%;" />

（动图1-2）

**透视UI**

3D UI是始终位于窗口上，和常规的UI一样。但3D UI可以进行XYZ三个轴上的运动，带来明显的透视变化。

<img src="img/1-3.gif" style="zoom:50%;" /> 

（动图1-3）



## 二、IDE中使用UI3D组件

### 2.1 创建一个2D的Prefab

在IDE中使用3D UI，首先需要我们创建一个用于在3D场景中展示的2D UI，这里必须使用Prefab2D来实现。首先，我们先创建一个Prefab2D，在Prefab2D中搭建一个希望实现的2D UI，例如，我们要做一个游戏中人物战斗中头顶的血条，如图2-1所示

<img src="img/2-1.png" style="zoom:50%;" />

（图2-1）

在Prefab2D中，创建一个Progress组件，因为血条有当前血量和总体血量构成，因此Progress正好符合我们的要求。并且血条上面使用Label显示人物的名字。另外注意Prefab的根节点Box的size最好改为2的N次幂，这符合纹理的2的N次幂原则。



### 2.2 创建Sprite3D，添加UI3D组件

在IDE的Scene3D节点下，创建一个Sprite3D对象，添加UI3D组件，如动图2-2所示

<img src="img/2-2.gif" style="zoom:50%;" /> 

（动图2-2）

在Sprite3D节点属性面板中，点击添加组件，选择渲染，选择UI3D组件。可以看到，当添加UI3D组件后，场景中在Sprite3D节点的位置，多了一个白色的纹理，这个纹理就是用来显示UI的。



### 2.3 添加2D Prefab资源

准备好UI3D组件后，下一步就是要把之前做好的2D Prefab UI拖入到UI3D组件的Prefab属性中，如动图2-3所示

<img src="img/2-3.gif" style="zoom:50%;" /> 

（动图2-3）

拖入Prefab后，纹理会立即显示2D UI。但是默认是OPAQUE渲染模式，纹理有黑色的背景色。下面会介绍UI3D的属性，来调整纹理效果



### 2.4 更改UI3D属性

对于UI3D组件，如图2-4所示，有如下一些属性：

<img src="img/2-4.png" style="zoom: 67%;" />  

（图2-4）

`Render Mode`：渲染模式，通常我们使用 Transparent 模式，也就是支持透明色，如图2-5所示，血条背景变成我们期望的透明了

<img src="img/2-5.png" style="zoom:50%;" />  

（图2-5）

`Prefab`：需要显示的2D Prefab资源文件

`Scale`：纹理的缩放值

`Resolution Rate`：纹理的分辨率，当拖入Prefab时，会自动识别Prefab下节点的size，来动态调整纹理的分辨率

`Billboard`：是否总是朝向摄像机，如果不选的话，会朝向Z轴方向，也就是可以实现UI在场景中的透视效果

`Enable Hit`：是否响应鼠标点击，默认是不勾选。如果勾选后，可以实现按钮的响应，滑动条的拖动，List组件的滑动等等



### 2.5 调整UI3D位置

我们的需求是做人物的血条，那首先把我们做好的但丁人物拖入到场景中，并设置为（0,0,0）点，如图2-6所示

<img src="img/2-6.png" style="zoom:50%;" /> 

（图2-6）

但是可以看到，由于之前创建的Sprite3D也在（0,0,0）点，那么位置就会在人物脚下，这时需要调整Sprite3D的位置，来符合血条的效果，如动图2-7所示

<img src="img/2-7.gif" style="zoom:50%;" /> 

（动图2-7）

这时来看看运行的效果：

<img src="img/2-8.gif" style="zoom: 33%;" />  

（动图2-8）

可以看到随着人物在摄像机前拉进和拉远，血条也在变大变小，很符合实际的效果 。如果用2D UI来实现的话，就需要动态去计算人物相对摄像机的位置来缩放UI的大小，效果肯定不好。

当然，我们也可以不勾选UI3D的 `Billboard` 属性，调整XYZ轴的旋转，让血条随着人物的旋转而改变朝向。

<img src="img/2-9.gif" style="zoom:50%;" /> 

（动图2-9）



### 2.6 脚本控制UI3D

通常我们需要对UI中的内容进行操作，此例中血条中血量比例的变化，有向上飘动的伤害数等等，这些都是通过对2D Prefab中UI组件控制来实现的。而UI3D组件通常是用来控制显示效果，比如透视效果，位置信息等。

就像处理2D UI的操作一样，我们再次打开Prefab2D，在根节点上添加Runtime类，添加逻辑代码如下：

```typescript
import { BloodBarBase } from "./BloodBar.generated";
import { Main } from "./Main";

const { regClass, property } = Laya;

@regClass()
export class BloodBar extends BloodBarBase {
    //declare owner : Laya.Sprite3D;

    constructor() {
        super();
    }

    onAwake(): void {

        this.bar.value = 1;
        this.value.visible = false;
        Laya.stage.on( Laya.Event.CLICK, this, this.onHurt );

    }

    onHurt(): void {
        
        this.bar.value = this.bar.value - 0.1;
        this.value.y = 35;
        this.value.visible = true;
        Main.player.getComponent(Laya.Animator).play("Stun");
        Laya.Tween.to( this.value, { y : -20 }, 500, null, Laya.Handler.create(this, this.end))
	}

	private end(): void {
		this.value.visible = false;
	}

}
```

最后来看看运行效果：

<img src="img/2-10.gif" style="zoom:50%;" /> 

（动图2-10）

到此为止，UI3D组件已经介绍完了，开发者可以在项目中通过使用UI3D组件来实现更多的3D UI效果。





