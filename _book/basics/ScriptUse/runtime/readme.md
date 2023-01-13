# runTime的使用

推荐阅读视频教程，有关LayaAir脚本与Runtime开发的系统全面讲解与实战介绍

脚本组件课链接： [https://ke.qq.com/course/427324](https://ke.qq.com/course/427324 ) 

UI与场景编辑、组件化开发、屏幕适配与抗锯齿的合集课程链接：[https://ke.qq.com/course/417469](https://ke.qq.com/course/417469 ) 

----------

在LayaAirIDE中资源面板下所有的组件均有runtime的属性，runtime是该组件运行时的逻辑类；相同组件可使用同一runtime类来实现相同的功能，比如不同页面上需要对相同的组件实现同一功能。**需要注意的是组件的runtime逻辑类如果不继承组件自身，并且继承的对象中没有该组件的属性时，这个属性则会失效。**

**runTime脚本与script脚本类似，不同的是runtime脚本的方式实现，继承页面，场景或组件类，实现逻辑。在IDE里面设置场景的Runtime属性即可和场景或对象进行关联**

- **相比script脚本方式，继承式页面类，可以直接使用页面定义的属性（通过IDE内var属性定义），比如this.tipLbll，this.scoreLbl，具有代码提示效果。而script脚本获取只能通过this.owner.getChildByName("xxx") 等方式获取节点**
- **建议：如果是页面级的逻辑，需要频繁访问页面内多个元素，使用runtime继承式写法，如果是独立小模块，功能单一，建议用script脚本方法，参见IDE新建2d示例项目 **

**本篇文章将对两个不同页面中的Image组件设置同一个runtime逻辑类来实现相同的功能，运行效果如动图0所示：**

![0](img\0.gif)(图0)

### 一、给页面中的组件设置runtime类

在页面管理目录下创建两个UI页面，分别叫MonkeyPage和BGPage。如下图，

![1](img\ide1.png) 

两个UI页面中各拖入一张Image组件，点击Scene2D设置runtime属性为game.ImageRunTime。(将脚本拖拽到runtime的script图标上)。 如图1图2所示：

![1](img\ide3.png)(图1)

![2](img\ide2.png)(图2)

设置完成之后保存导出UI，开始编写逻辑代码。



### 二、代码逻辑处理

切换到代码模式下，

然后在ImageRunTime类中编写我们想要实现的效果，比如实现一个点击缩放（类似按钮）的功能，全部代码如下所示：

```typescript

    /*
    ImageRunTime逻辑类 
    */
    export default class ImageRunTime extends Laya.Image{
        public scaleTime:number = 100;
        constructor() {
            super();
            //设置组件的中心点
			this.anchorX = this.anchorY = 0.5;
			//添加鼠标按下事件侦听。按时时缩小按钮。
			this.on(Laya.Event.MOUSE_DOWN,this,this.scaleSmall);
			//添加鼠标抬起事件侦听。抬起时还原按钮。
			this.on(Laya.Event.MOUSE_UP,this, this.scaleBig);
			//添加鼠标离开事件侦听。离开时还原按钮。
			this.on(Laya.Event.MOUSE_OUT,this, this.scaleBig);
        }
        private scaleBig():void
		{
			//变大还原的缓动效果
			Laya.Tween.to(this, {scaleX:1,scaleY:1},this.scaleTime);
		}
		private scaleSmall():void
		{
			//缩小至0.8的缓动效果
			Laya.Tween.to(this,{scaleX:0.8,scaleY:0.8},this.scaleTime);
		}
    }

```

在主运行类中实例化这两个UI界面，代码如下所示：

```typescript
import GameConfig from "./GameConfig";
import { ui } from "./ui/layaMaxUI";
class Main {
	constructor() {
		//根据IDE设置初始化引擎		
		if (window["Laya3D"]) Laya3D.init(GameConfig.width, GameConfig.height);
		else Laya.init(GameConfig.width, GameConfig.height, Laya["WebGL"]);
		Laya["Physics"] && Laya["Physics"].enable();
		Laya["DebugPanel"] && Laya["DebugPanel"].enable();
		Laya.stage.scaleMode = GameConfig.scaleMode;
		Laya.stage.screenMode = GameConfig.screenMode;

		//打开调试面板（通过IDE设置调试模式，或者url地址增加debug=true参数，均可打开调试面板）
		if (GameConfig.debug || Laya.Utils.getQueryString("debug") == "true") Laya.enableDebugPanel();
		if (GameConfig.stat) Laya.Stat.show();
		Laya.alertGlobalError = true;

		//激活资源版本控制，version.json由IDE发布功能自动生成，如果没有也不影响后续流程
		Laya.ResourceVersion.enable("version.json", Laya.Handler.create(this, this.onVersionLoaded), Laya.ResourceVersion.FILENAME_VERSION);
	}

	onVersionLoaded(): void {
		//激活大小图映射，加载小图的时候，如果发现小图在大图合集里面，则优先加载大图合集，而不是小图
		Laya.AtlasInfoManager.enable("fileconfig.json", Laya.Handler.create(this, this.onConfigLoaded));
	}

	onConfigLoaded(): void {
		//加载IDE指定的场景, 如果在编辑器中制作场景就打开下面一行注释，把实例页面的代码注掉
		//GameConfig.startScene && Laya.Scene.open(GameConfig.startScene);

		 //实例化BGPageUI页面
		 var bgPage: ui.BGPageUI = new ui.BGPageUI();
		 //为了能够清楚的看到这个页面所在的位置，在此设置设置一个背景色
		 bgPage.graphics.drawRect(0, 0, 300, 300, "#ffcccc");
		 //添加到stage
		 Laya.stage.addChild(bgPage);
		 //实例化MonkeyPageUI页面
		 var monkeyPage: ui.MonkeyPageUI = new ui.MonkeyPageUI();
		 //为了能够清楚的看到这个页面所在的位置，在此设置设置一个背景色
		 monkeyPage.graphics.drawRect(0, 0, 300, 300, "#ffcccc");
		 //添加到stage
		 Laya.stage.addChild(monkeyPage);
		 //设置第二个页面的坐标
		 monkeyPage.x = 350;

	}
}
//激活启动类
new Main();
```

以上是兼容1.0的代码。

### 三、如果runtime逻辑类继承的对象非自身组件

在以上代码中我们演示了继承自身组件Image所实现的效果，如果继承一个Button组件类会出现什么情况呢？我们来操作看下。代码以及实现效果如下所示：

```typescript
module game {
    /*
    ImageRunTime逻辑类 
    */
    export class ImageRunTime extends Laya.Button{
        public scaleTime:number = 100;
        constructor() {
            super();
            //设置组件的中心点
			this.anchorX = this.anchorY = 0.5;
			......
        }
        ......
    }
}
```

![5](img\5.gif)(图5)

这时我们会发现UI页面上的资源显示的很怪异，这时因为按钮的skin默认是三态的，当Image的runtime逻辑类继承自Button组件后，它就不再是一个Image组件了，而是一个Button组件。

