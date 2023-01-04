
# LayaNative首页说明

> Update：2021-10-12

重要的事情需要说三遍：

##### LayaNative不是浏览器！

##### LayaNative不是浏览器！

##### LayaNative不是浏览器！

### 1、LayaNative的启动入口

由于LayaNative不是浏览器，也不是通过封装浏览器或者webkit之类的控件来运行html的内容。

所以，LayaNative不能启动和运行html页面文件。

引擎项目bin目录下的index.html可以作为浏览器里的入口，但是不能作为LayaNative的启动入口。

**LayaNative的启动入口默认为`引擎项目bin目录下的index.js`**，开发者也可以在该目录下手动创建runtime.json作为启动入口，后面会分别说明。

到底是用index.js还是runtime.json作为入口，通过LayaAirIDE的菜单栏`工具`--> `app构建` ，打开的构建项目窗口里，URL那里配置好即可，配置方式如图1所示。

![](img/1.png) 

在图1里，URL是bin目录下的项目地址，入口默认为index.js，如果开发者想改用runtime.json，可以创建并修改这里的URL入口地址。

### 2、LayaNative的启动文件配置说明

无论是index.js还是runtime.json，作为项目启动入口。入口文件主要提供两个功能。

* 确定项目运行时需要加载的js文件。
* 对横竖屏进行设置。

具体的修改方式如下：

#### 2.1 index.js的启动配置

如果我们使用项目bin目录下的index.js作为LayaNative的启动入口文件，

需要使用loadLib函数确定项目运行时需要加载的js文件，

如果想设置横竖屏，而需要按照下面代码的示例和注释说明来修改window.screenOrientation变量值。

示例代码如下：

```javascript
/**
 * 设置LayaNative屏幕方向，可设置以下值
 * landscape           横屏
 * portrait            竖屏
 * sensor_landscape    横屏(双方向)
 * sensor_portrait     竖屏(双方向)
 */
window.screenOrientation = "landscape"; // 设置屏幕为横屏
//-----引擎库开始-----
loadLib("libs/laya.core.js")
loadLib("libs/laya.ui.js")
loadLib("libs/laya.d3.js")
//-----引擎库结束-------
loadLib("js/bundle.js");//项目代码js
```


**注意：** 请不要在index.js文件里编写任何逻辑代码，如果编写可能会发生未知的错误。

#### 2.2  runtime.json的启动配置

如果开发者想使用runtime.json文件作为启动文件。

则需要先在项目的bin目录下创建一个runtime.json。

然后，使用 "scripts" 配置需要加载的js文件，使用"screenOrientation" 对横竖屏进行设置。

例如，我们将上面的index.js配置改成runtime.json的配置，代码如下：

```json
{
	"scripts": ["libs/laya.core.js","libs/laya.ui.js","libs/laya.d3.js","js/bundle.js"],
	"screenOrientation": "landscape"
}
```

