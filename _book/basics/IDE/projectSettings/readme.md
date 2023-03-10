# 项目设置详解

> Author：Charley

IDE的项目设置包括两个部分，运行设置与编辑器设置。

## 一、运行设置

### 1.1 屏幕适配设置 Resolution

屏幕适配的设置会影响IDE内的预览效果，以及项目运行时的画布宽高、适配模式、对齐方式，画布背景色等。可设置属性如图1所示：

<img src="img/1.png" style="zoom:60%;" />  

（图1）

#### 1.1.1 画面宽高适配

影响产品画面宽高的三个设置，分别是设计宽高（Design Width、Design Height），缩放适配模式(Scale Mode)。

设计宽高，就是我们在IDE里的设置并看到的宽高，

这个宽高会影响在IDE里的UI场景背景大小，以及IDE的预览运行模式下，也是基于这个宽高查看效果。

在实际的运行环境里，例如不同的手机。由于屏幕比例的不同，肯定不可能与设计宽高完全吻合。

所以，就需要用引擎自带的缩放适配模式，进行缩放，从而来满足开发者的屏幕需求。

缩放适配模式涉及画布、舞台、适配算法等诸多知识点，我们在另一篇文档[《屏幕适配》](../../common/adaptScreen/readme.md)里详细介绍。

#### 1.1.2 横竖屏适配ScreenMode

有的时候，我们需要根据屏幕的比例强制横竖屏设置，在IDE中可以通过设置屏幕模式（Screen Mode）来设置。

横竖屏有三种适配模式，如图2-1所示。

 <img src="img/2-1.png" alt="图3" style="zoom:60%;" />

（图2-1）

#### 无变化：none

none时，无论屏幕方向如何旋转，游戏的水平方向都不会产生跟随屏幕旋转的变化。

效果如动图2-2所示。

![](img/2-2.gif) 

(动图2-2) 

通过动图2-2发现，none值时，当屏幕发生旋转后，基于竖屏设计的界面在横屏下就会变的不适合，同理，基于横屏设计的界面在竖屏下，也会变的不适合。

当然，如果我们的布局策略运用的比较合理，也许也可以兼顾横竖屏的体验。效果如动图2-3所示。

![](img/2-3.gif) 

(动图2-3)

虽然不那么难看了，但为了达到最佳的效果，最好的方案，还是竖屏始终与设备竖屏的方向保持一致，横屏与设备横屏的方向保持一致。

#### 始终横屏：horizontal

当我们设置的宽高就是横屏产品时，horizontal无疑是最佳的体验，效果如动图2-4所示。

![](img/2-4.gif)  

(动图2-4)

通过动图2-4发现，screenMode属性值设置为horizontal时，无论屏幕方向如何旋转，设计上的水平方向都会与屏幕最短的边始终保持垂直，所以用户设备竖屏时看到横屏画面，自然就会把设备横过来，从而吻合了产品的设计。

#### 始终竖屏：vertical

当我们设置的宽高就是竖屏产品时，vertical无疑是最佳的体验，效果如动图2-5所示。

![](img/2-5.gif)  

(动图2-5)  

通过动图2-5发现，screenMode属性值设置为vertical时，无论屏幕方向如何旋转，游戏的水平方向都会与屏幕较长的边始终保持垂直。所以用户哪怕是把设备横屏了，仍然看到的是竖屏画面，自然就会把设备恢复竖屏，从而吻合了产品的设计。

> [!Tip]
>
> 需要注意的是，浏览器中运行的时候，引擎的自动横屏和自动竖屏，只能对画布进行旋转，如果用户的手机锁屏了，虽然画面自动旋转过来了，但是浏览器没有旋转过来，会导致输入法依然按浏览器的方向弹出，此时，可能会导致输入法与浏览器的显示呈90度。
>
> 在小游戏平台中运行，由于小游戏底层有横屏还是竖屏的配置，不会出现这个问题。

#### 1.1.3 画布对齐适配AlignV、AlignH

引擎中的提供的alignV（垂直对齐）与alignH（水平对齐）是对画布进行对齐。设置方式如图3所示：

<img src="img/3.png" style="zoom: 60%;" />  

（图3）

参数说明如下：

AlignV垂直对齐的参数为：top（顶部对齐）、middle（垂直居中对齐）、bottom（底部对齐）。

AlignH水平对齐的参数为：left（居左对齐）、center（水平居中对齐）、right（居右对齐）。

> [!Tip]
>
> 画布对齐不能理解为UI界面基于stage舞台的对齐，只是画布canvas相对于整个物理屏幕的对齐。
>
> 该设置在移动端，基本用不上，移动端绝大多数都需要全屏适配。当画布已经铺满整个屏幕时，设置就没有了意义。
>
> 通常是在PC端，非全屏的模式下使用，例如在画布非全屏适配的模式（showall和noscale）的情况下使用。

#### 1.1.4 画布背景色设置BackgroundColor

画布背景色，其实就是给画布设置一个颜色，默认值为`#888888`，如图4所示：

<img src="img/4.png" style="zoom: 60%;" />  

（图4）

### 1.2 引擎初始化设置 Engine Options

有一些引擎的配置项，需要在引擎初始化的时候设置，而设置的入口就是如图5所示：

<img src="img/5.png" style="zoom: 60%;" />  

（图5）

属性参数说明

| 属性名称                     | 属性说明                                                     |
| ---------------------------- | ------------------------------------------------------------ |
| FPS                          | 设备帧率；用于计算两帧之间的渲染最大间隔时长，通常设备上的帧率是最大60，也就是一秒钟内最多只有 60 帧会出现在屏幕上，那么两帧之间的时长为1000ms/60。对于高帧率设备，我们可以修改FPS值，例如120帧的设备，那两帧之间的时长则为1000ms/120。 |
| Is Antialias                 | 是否开启抗锯齿；用于设置webGL上下文的antialias抗锯齿开关属性，会产生额外的性能消耗，主要用于2D非矩形的矢量绘图抗锯齿，无矢量绘制图形或性能压力大时，可以选择不开启。3D抗锯齿推荐使用摄像机的Fxaa或Msaa。 |
| Use Retinal Canvas           | 使用高清画布模式；开启后无论任何适配模式，画布均采用物理分辨率大小，开启后会比不采用物理分辨率多一些性能消耗，但会让文本等保持最佳清晰度。 |
| Is Alpha                     | 是否画布透明；默认状态画布有背景色，开启后，可以设置画布为无色透明。 |
| MeshAlloc MaxMem             | 是否分配最大的VB缓冲区；开启后，渲染2D的时候，每次创建vb直接分配足够64k个顶点的缓存。这样可以提高效率。关闭后，可节省64k显存，但会牺牲性能效率，如果包含2D时，建议保持默认开启。 |
| Enable Dynamic Batch         | 启用动态合批；开启动态合并后，满足 **实例合并**（同Mesh且同材质） ，即可减少RenderBatches渲染批次与Shader提交次数。 |
| Enable Uniform Buffer Object | 启用Uniform Buffer；当启用Uniform Buffer缓存后，可以减少CUP传递至GPU的数据量。 |
| Pixel Ratio                  | 设置3D的分辨率倍数，默认值为1 ；降低3D分辨率，不会影响2D UI的分辨率，适当的调节可降低性能的消耗。 |
| Enable Multi Light           | 是否开启多光源；如果不需要多光源，关闭后可减少性能的消耗。   |
| Max Light Count              | 最大光源数量；默认值为32个。                                 |
| Light Cluster Count          | x、y、z轴的光照集群数量；z值会决定Cluster受区域光（点光、聚光）影响的数量，Math.floor(2048 / lightClusterCount.z - 1) * 4 为每个Cluster的最大平均接受区域光数量,如果每个Cluster所接受光源影响的平均数量大于该值，则较远的Cluster会忽略其中多余的光照影响。 |

### 1.3 项目启动设置Startup

#### 1.3.1 入口启动场景 Startup Scene

LayaAir 3.0项目运行入口的设置有两种方式，一种是将当前场景Current Scene（正编辑的场景）作为项目运行的入口，另一种是设置一个固定的项目入口场景。

当我们在**构建发布**里设置了**启动场景**，并且**勾选**了启动场景作为入口，如图6所示。在运行项目时，引擎初始化之后，就会先运行设置的启动场景。

<img src="img/6.png" style="zoom: 60%;" />  

（图6）

#### 1.3.2 引擎库模块

LayaAir引擎由多个模块组件，默认只引入了较为基础的模块，如图7所示。

<img src="img/7.png" style="zoom: 60%;" />  

(图7) 

如果应用到其它模块，需要勾选对应的模块，才可以使用其API，否则项目运行时会导致报错。

引擎库模块说明：

| 引擎库模块名        | 引擎库模块说明                                               |
| ------------------- | ------------------------------------------------------------ |
| laya.d3             | 3D基础模块，使用3D的必选库                                   |
| laya.ui             | ui模块，包括常用的ui组件，使用2D UI组件的必选库              |
| laya.ani            | 2D动画模块，包括2D节点动画（序列帧、图集动画）、内置的骨骼动画等 |
| laya.device         | 陀螺仪、加速计、地理位置、摄像头、麦克风等设备接口调用封装   |
| laya.html           | 原生DOM相关的接口封装                                        |
| laya.tiledmap       | tiledmap地图接口封装                                         |
| laya.particle       | 2D粒子的封装，不推荐使用                                     |
| laya.spine          | spine动画引擎库                                              |
| laya.gltf           | 代码直接使用gltf模型的加载解析库                             |
| laya.physics        | Box2D物理库的封装                                            |
| laya.physics3D      | Bullet 3D物理库                                              |
| laya.physics3D.wasm | WebAssembly的Bullet 3D物理库                                 |

### 1.4 调试启动设置Debug

IDE可以开启两种调试模块，分别是统计信息Stat与控制台V Console，如图8-1所示。

<img src="img/8-1.png" alt=" " style="zoom: 60%;" />  

（图8-1）

#### 1.4.1 统计信息 Stat

勾选统计信息`Stat`之后，可以查看当前帧率、内存占用、节点等信息，用于项目的分析与优化。如图8-2所示。

<img src="img/8-2.png" style="zoom:67%;" />  

（图8-2）

如想了解更详细的统计信息面板上的参数，请查阅文档[《性能统计与优化》](../../common/Stat/readme.md) 

#### 1.4.2 移动端调试工具 V Console

在移动端调试，通常需要联到电脑端的浏览器上。

如果开发者不需要断点，只是一些常用的日志打印、加载等查看等，开启`V Console`，在移动端会出现如图8-3所示的调试工具面板。

![](img/8-3.png)  

（图8-3）



## 二、编辑器设置Editor

### 2.1 3D预制体编辑环境 Prefab Edit Env

默认情况下，3D预制体是位于一个专用的系统空场景（DefaultPrefabEditEnv）的环境下进行编辑。

如果我们通过 Prefab Edit Env，指定了一个目标场景，相当于直接位于某个3D场景中进行编辑，这样当切换到3D场景中，就会更加符合需求。操作如图9-1所示：

<img src="img/9-1.png" style="zoom: 60%;" />   

（图9-1）

效果如图9-2所示：

<img src="img/9-2.png" style="zoom:60%;" /> 

### 2.2 3D节点层级设置

3D节点，我们可以选择层级并设置，而编辑器设置EditorSettings中，正是增加、删除层级，以及为层级命名的地方。

效果如图10所示。

<img src="img/10.png" alt="img" style="zoom: 60%;" />  

(图10)

关于层级Layer的更多介绍，可前往IDE文档[《使用3D精灵》](../../../3D/Sprite3D/readme.md)进行查看。

