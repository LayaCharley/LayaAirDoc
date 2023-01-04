# LayaAir Native

LayaAir Native是LayaAir引擎针对移动端原生APP的开发、测试、发布的一套完整的开发解决方案。

LayaAir Native基于C++的运行时，封装了原生的图形API（例如OpenGL ES），支持LayaAir引擎产品可打包成为APP上架移动端APP平台。

还支持反射机制，提供开发者在原生APP上进行二次开发和渠道对接的自定义扩展。

LayaAir Native的3.0版本，更是进行了大量的重构，在面向未来，以及开发者的扩展自由度上，进一步得到了加强。

例如，在渲染内核方面，除了OpenGL ES，新增支持Vulkan、Metal。

在核心算法层里，开发者可以直接用C++的算法，直接接管替换引擎的JS算法。

### LayaAir Native 3.0包括以下主要功能：

- ### LayaPlayer

  LayaPlayer是LayaNative核心的部分，它是一个基于JavaScript脚本引擎 + OpenGL ES硬件加速渲染的跨平台C++引擎，通过对内存与渲染流程进行极致优化，为LayaAir引擎脱离浏览器成为APP提供独立的底层运行环境。

  > 需要注意的是，LayaPlayer不等于浏览器，仅为LayaAir引擎运行提供支撑，不支持普通web链接的运行。

- #### 构建工具：

  构建工具可帮助开发者快速构建移动端APP项目工程，然后使用Android Studio、Eclipce、XCode 等开发工具打开->构建->运行；

- #### 反射机制

  通过反射机制，开发者可以实现JavaScript与原生语言(Android/Java 或 iOS/Objective-C)的相互调用，通过反射机制开发者可以很方便的对应用程序进行二次扩展；