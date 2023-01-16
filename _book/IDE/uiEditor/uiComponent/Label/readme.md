# Label 组件参考



## 一、通过LayaAirIDE创建Label组件

###     1.1 创建Label

​        Label 组件用来显示一段文字，文字可以是系统字体或者 BMFont 字体。

​        点击资源面板里的 Label 组件，拖放到页面编辑区，即可添加 Label 组件到页面上。
​        Label 组件的脚本接口请参考 [Label API](https://layaair2.ldc2.layabox.com/api2/Chinese/index.html?version=2.9.0beta&type=2D&category=UI&class=laya.ui.Label)。

​        Label 组件的资源示例：

​        ![图片0.png](img/1.png)<br/>

​    （图1）

​        Label组件拖放到编辑区后，设置 text 属性的值为 LayaAir IDE 后的显示效果如下：

​        ![图片0.png](img/2.png)<br/>

​    （图2）

 

 

###  1.2 Label 属性

​        ![图片0.png](img/3.png)<br/>

​    （图3）

 

| **属性**       | 功能说明                                                     |
| -------------- | ------------------------------------------------------------ |
| text           | 文本内容字符串。                                             |
| align          | 文本的对齐方式。                                             |
| color          | 文本的颜色值。默认为黑色。                                   |
| bgColor        | 文本背景颜色。                                               |
| font           | 文本的字体名称。                                             |
| fontSize       | 文本的字体大小。                                             |
| style          | 文本是否为粗体字显示。                                       |
| italic         | 文本是否显示为斜体。                                         |
| wordWrap       | 文本是否换行，默认为false，不可自动换行。                    |
| stroke         | 文本的描边宽度。                                             |
| strokeColor    | 文本的描边颜色。                                             |
| leading        | 文本的垂直行间距。                                           |
| padding        | 文本的边距。                                                 |
| overflow       | 文本超出文本域后的行为。visible：不进行任何裁剪。hidden：不显示超出文本域后的字符。scroll：不显示文本域外的字符像素，并且支持scroll接口。 |
| bordercolor    | 文本边框颜色。                                               |
| underlinecolor | 下划线颜色。                                                 |

 

## 二、通过代码创建Label组件

 	在我们进行书写代码的时候，免不了通过代码控制UI，创建`UI_Label`类，通过代码设定Label相关的属性。

**运行示例效果:**

​	![5](img/4.png) <br/>

​    (图5)通过代码创建Label

Label的其他属性也可以通过代码来设置，下述示例代码演示了如何通过代码创建不同皮肤（样式）的Label，有兴趣的读者可以自己通过代码设置Label，创建出符合自己需要的文字效果。

更多的文字效果可以去查看2D基础篇中的文本部分。

**示例代码：**

```javascript
const { regClass, property } = Laya;
const Label = Laya.Label;
@regClass()
export class UI_Label extends Laya.Script {


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
		this.createLabel("#FFFFFF", null).pos(30, 50);
		this.createLabel("#00FFFF", null).pos(290, 50);
		this.createLabel("#FFFF00", "#FFFFFF").pos(30, 100);
		this.createLabel("#000000", "#FFFFFF").pos(290, 100);
		this.createLabel("#FFFFFF", "#00FFFF").pos(30, 150);
		this.createLabel("#0080FF", "#00FFFF").pos(290, 150);
	}

	private createLabel(color: string, strokeColor: string): Laya.Label {
		const STROKE_WIDTH: number = 4;

		var label: Laya.Label = new Label();
		label.font = "Microsoft YaHei";
		label.text = "SAMPLE DEMO";
		label.fontSize = 30;
		label.color = color;

		if (strokeColor) {
			label.stroke = STROKE_WIDTH;
			label.strokeColor = strokeColor;
		}

		this.owner.addChild(label);

		return label;
	}
```



 	