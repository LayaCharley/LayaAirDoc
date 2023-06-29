# Image 组件参考



## 一、通过LayaAirIDE创建Image组件

###        1.1 创建image

Image 是 UI 里最常见的显示图像的组件，用来显示位图图像。可以设置 Image 组件的 skin 属性来改变 Image 组件呈现的图像。Image 组件支持九宫格数据设定，用于实现图像放大后图像显示不失真的效果。

点击资源面板里的 Image 组件，拖放到页面编辑区，即可添加 Image组件到页面上。单击选中 Image ，可以在属性面板里设置 Image 的常用属性的值。
        Image 组件的脚本接口请参考 [Image API](https://layaair.com/3.x/api/Chinese/index.html?version=3.0.0&type=2D&category=UI&class=laya.ui.Image)。

 **Image 组件的资源示例：**

​        ![图片0.png](img/1.png) 

​    （图1）

**Image 组件拖放到编辑区后显示效果：**

![图片0.png](img/2.png) 

(图2)    

### 1.2 Image 组件的常用属性

![图片0.png](img/3.png) 

（图3）

| **属性** | **功能说明**                           |
| -------- | -------------------------------------- |
| sizeGrid | 位图的有效缩放网格数据（九宫格数据）。 |
| skin     | 位图的资源。                           |
| Group    | 加载分组，设置后可以按组管理资源。     |

添加 Image 组件之后，可以通过从资源面板中拖拽图片资源到 Image 的 skin 属性框，来修改 Image 组件的显示资源图像。

## 二、通过代码创建Image组件

在我们进行书写代码的时候，免不了通过代码控制UI，创建`UI_Image`类，通过代码设定Image相关的属性。

**运行示例效果:**
​	![5](img/4.png) 

​	(图4)通过代码创建Image

Image的其他属性也可以通过代码来设置，下述示例代码演示了如何通过代码创建不同皮肤（样式）的Image，有兴趣的读者可以自己通过代码设置Image，创建出符合自己需要的图片。

**示例代码：**

```javascript
const { regClass, property } = Laya;

@regClass()
export class UI_Image extends Laya.Script {


    constructor() {
        super();
    }

    /**
     * 组件被激活后执行，此时所有节点和组件均已创建完毕，此方法只执行一次
     */
    onAwake(): void {

		this.setup();
	}

	private setup(): void {
		var dialog: Laya.Image = new Laya.Image("resources/res/ui/dialog (3).png");
		dialog.pos(165, 62.5);
		this.owner.addChild(dialog);
	}
}
```

