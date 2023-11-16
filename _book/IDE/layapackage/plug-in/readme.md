# 插件开发说明

## 一、开始

编辑器使用Electron框架开发，本质是对Chrome浏览器的封装。编辑器实际是一个HTML+JS+CSS制作的单页应用。但我们封装了大部分功能，所以游戏开发者和插件开发者都不需要学习HTML/CSS或者相关的前端框架。我们在开发插件时，需要使用编辑器提供的UI框架，一般不允许直接去修改编辑器的DOM结构。考虑到用户体验和兼容性，资源商店也一般不允许这类插件上架。

在项目的engine/types目录，有三个与编辑器插件开发相关的声明文件，editor.d.ts, editor-ui.d.ts, editor-env.d.ts，他们包含了大量编辑器的扩展API。

editor.d.ts 是编辑器UI进程的API。常用的是全局对象Editor和IEditor命名空间下的类和接口。

editor-env.d.ts 是场景场景进程的API。常用的是全局对象EditorEnv和IEditorEnv命名空间下的类和接口。

editor-ui.d.ts 编辑器UI库。使用IEditorUI命名空间下的类和接口。

其中IEditor.utils/IEditorEnv.utils暴露了大量实用的工具函数，包括UUID生成，加密解密，ZIP压缩/解压，文件/目录拷贝/移动，HTTP请求，上传/下载等等。



## 二、插件运行环境

编辑器是多进程体系，主要有三个进程：Main进程/UI进程/Scene进程。插件只能运行在UI进程和Scene进程。UI进程没有载入引擎库，也就是没有LayaAir引擎环境；Scene进程有LayaAir引擎环境，与UI通讯只能通过编辑器提供的异步通讯API。此外，插件脚本也可能在预览（含编辑器内预览和外部浏览器预览）中运行，它与Scene进程的区别是，预览进程中没有node环境。总结一下，插件脚本可能运行的环境有3种：

1) UI进程：可直接使用node等本地模块，不可使用Laya引擎；

2) Scene进程：可直接使用node等本地模块，可使用Laya引擎；

3) 预览进程：不可使用node等本地模块，可使用Laya引擎；

如果插件脚本错误使用了当前环境中不存在的特性，脚本可能无法运行。例如，在一个Laya自定义组件脚本里使用node的fs模块去访问文件系统，会导致脚本在预览或者最终发布时运行报错。

为了脚本能在各种环境正确运行，开发者编写脚本时，需要自行在文件级别隔离这三种代码。比如，一个自定义面板的TS文件，不能引用Laya引擎，因为它是运行在UI进程里的。又比如，一个Laya.Script，不能使用fs，path等模块，因为它可能会运行在预览进程。但这两种需求也是常见的，应该怎么办？

1、在Laya.Script中使用本地能力，下面是一个推荐做法：

```TypeScript
//Script.ts
@Laya.regClass()
class Script extends Laya.Script {

    wantToUseNode() {
        IEditorEnv.scene.runScript("TestSceneScript.visitNode");
    }
}

//TestSceneScript.ts
import fs from "fs";

//注意是IEditorEnv.regClass，不是Laya.regClass!!
@IEditorEnv.regClass()
class TestSceneScript {

    static visitNode() {
        fs.readFileSync(....)        
    }
}
```

> TestSceneScript.ts这个文件会在发布中被裁剪，因为它没有被Laya.Script直接引用。



2、UI进程脚本和Scene进程脚本通讯的方式有以下几种：

1) 设置节点/组件属性后，场景里的节点/组件会自动刷新，无需代码。例如：

```TypeScript
//下面是UI进程代码

//获取选中的节点
let node = Editor.scene.getSelection()[0];

//修改节点属性，场景里的对象会自动同步，无需手动
node.props.x = 100;

//修改组件属性，场景里的对象会自动同步，无需手动
node.getComponent("MeshRenderer").props.enabled = false;
```

2) 调用节点/组件的一个方法，并返回值。例如：

```TypeScript
//下面是UI进程代码

//获取选中的节点
let node = Editor.scene.getSelection()[0];

//调用MyScript组件里的test方法，传入参数abc
let ret = await Editor.scene.runNodeScript(node.id, node.getComponent("MyScript").id, "test", "abc");
console.log(ret);
```

3) 自定义一个函数，并执行。例如

```TypeScript
//下面是场景进程的代码

//注意：IEditorEnv.regClass是必须的
@IEditorEnv.regClass()
export class TestSceneScript {
    //注意：this是当前的IEditorEnv.IGameScene对象，如果不需要，也可以省略这个声明
    static test(this: IEditorEnv.IGameScene, msg: string) {
        console.log(msg); //hello
        
        return "ok";
    }
}
//下面是UI进程的代码

let ret = await Editor.scene.runScript("TestSceneScript.test", "hello");
console.log(ret); //ok
```

编辑器实现脚本隔离的机制是将脚本编译成三个js，bundle.js包含可以在预览环境，也就是在浏览器可以运行的版本；bundle.editor.js是在UI进程运行的脚本；bundle.scene.js是在Scene进程的脚本。这个机制是自动的，但开发者需了解这个机制的运作方式：

1）所有含有@Laya.regClass装饰器的脚本会编译进bundle.js和bundle.scene.js。注意，这里指的“所有”，仅限于Debug版本。在release版本的bundle.js里，只会包含有被场景引用的脚本。bundle.scene.js不会出现在发布版本中。

2）所有含有@IEditorEnv.regClass装饰器的脚本会编译进bundle.scene.js。这个js只在编辑器内部运行，可以放心使用node能力。

3）所有含有@IEditor.xxx装饰器的脚本会编译进bundle.editor.js。这个js只运行在UI进程，可以放心使用node能力，但如果引用Laya的类会报错。

虽然这种识别机制能够解决标记不同脚本使用用途的问题，但还是建议开发者自行用目录方式进行隔离。例如将UI进程运行的脚本放入到editor名称的目录，将只在编辑器内运行的脚本放入到scene名称的目录，这样维护起来会更清晰。



## 三、制作编辑器UI

LayaAirIDE提供了开发编辑器UI的可视化编辑器。在项目资源的菜单里，新建一个编辑器界面预制体，例如MyWidget.widget。建议将此类预制体放置到editorResources目录或其子目录。

> editorResources目录是一个LayaAirIDE里约定名称的特殊目录，他和resources的特性类似，也就是无论它放在多深的子目录下，资源的路径都可以从editorResources开始。例如一个资源位于/MaDaHa/v1/editoResources/a/b.png，在引用资源时可以直接填写editorResources/a/b.png，而不用理会前面的MaDaHa/v1/。这在插件体系中是很重要的，因为在开发插件时，插件开发者无法确定用户使用时会将插件资源放置在什么目录下，只能确定用户不会破坏插件内部的目录结构。
>
> resources目录里具有相同的特性，差别在于，resources目录在发布时会拷贝到发布目录里，但editorResources则永远不会。

双击打开MyWidget.widget，使用内置UI编辑器制作好插件需要的UI界面。

![3-1](img/3-1.png)

以面板为例，代码里载入该预制体的方法为：

```TypeScript
@IEditor.panel("Test")
export class MyPanel extends IEditor.EditorPanel {
    async create() {
        this._panel = await gui.UIPackage.createWidget("editorResources/UI/MyWidget.widget");
        let input: gui.TextInput = this._panel.getChild("TextInput").getChild("title");
        input.on("changed", () => {
            console.log("改变了！");
        })
    }
}
```



## 四、自定义Inspector字段编辑界面

当我们编写一个组件，并暴露某些字段到IDE编辑后，有时希望能够自定义某个字段的编辑界面，可以通过以下步骤：

1、编写一个InspectorField

 ```TypeScript
   @IEditor.inspectorField("MyTestField")
   export class TestField extends IEditor.PropertyField {
       @IEditor.onLoad
       static async onLoad() {
           await gui.UIPackage.resourceMgr.load("MyField.widget");
       }
       
       create() {
           let input = gui.UIPackage.createWidgetSync("MyField.widget");
   
           return { ui: input };
       }
       
       refresh() {
           //这里负责将数据设置到界面上
       }
   }
 ```

 MyTestField是注册的名字，实际应用需要保证不要和其他人取的名字冲突，所以建议取"com.layabox.test"这样的名字。

InspectorField的create方法是同步的，所以这里不能用createWidget，而需要用createWidgetSync。这需要确保预制体在创建之前已经载入。所以这里使用了一个IEditor.onLoad的回调用于提前载入资源。

2、设置字段的inspector属性为刚才取的名字，这里为MyTestField

 ```TypeScript
      @Laya.regClass()
      export class Script extends Laya.Script {
           @property({ type : Laya.Node, inspector: "MyTestField" })
           public node: Laya.Node;
      }
 ```

3、实际效果：

<img src="img/4-1.png" alt="4-1" style="zoom:50%;" />



## 五、自定义Inspector数据流和布局

当选定场景中一个节点，或者一个资源后，在Inspector中如何读取数据，如何展现数据，这部分可以由开发者完全控制，方法是编写一个InspectorLayout类。

```TypeScript
@IEditor.inspectorLayout("asset")
export class TestInsLayout implements IEditor.IInspectorLayout {
    accept(item: IEditor.IAssetInfo): boolean {
        return item.type == IEditor.AssetType.Image;
    }
    onRender(items: readonly any[], inspectors: IEditor.IInspectorHelper, componentInspectors: IEditor.IInspectorHelper): Promise<void> {
        //这里负责读取数据，并填充inspectors
        
    }
}
```

这里"asset"表示这个InspectorLayout触发的类型，可用值有asset/node/等自定义的类型。



## 六、使用面板

可以通过以下方式给编辑器增加一个面板

```TypeScript
@IEditor.panel("test", {
    title: "Test",
    icon : "editorResources/20230710-161955.png"
})
export class TestPanel extends IEditor.EditorPanel {

    async create() {
        this._panel = await gui.UIPackage.createWidget("MyPanel.widget");
    }
}
```

效果如下图：

<img src="img/6-1.png" alt="6-1" style="zoom:50%;" />



## 七、使用对话框

可以通过以下方式创建一个弹出的对话框：

```TypeScript
//MyDialog.ts
export class MyDialog extends IEditor.Dialog {
    async create() {
        this.contentPane = await gui.UIPackage.createWidget("MyDialog.widget");
    }
    
    onShown() {
    }
    
    onHide() {
    }
}
```

在编辑器内，所有对话框都是单例。显示这个对话框的方式为：

```TypeScript
import { MyDialog } from "./MyDialog";

Editor.showDialog(MyDialog, null);
```



## 八、扩展内置菜单

支持对编辑器现有菜单的扩展。如以下代码，在应用程序菜单栏的工具菜单下，新增了一个test的菜单，并且点击菜单会调用test函数。

```TypeScript
class AnyName {
    @IEditor.menu("App/tool/test")
    static test() {
        console.log("click menu");
    }
}
```

menu的第一个参数表示菜单的路径，路径用"/"分隔，"App/tool/test"表示App菜单下的tool子菜单的test子项。注意这里的路径使用的是ID，不是菜单显示的文字。编辑器内部支持扩展的所有菜单名称和它的子菜单可以通过下面的方法打印出来参考：

<img src="img/8-1.png" alt="8-1" style="zoom:50%;" />

menu方法的第二个参数是可选参数，通过它可以进行一些额外的配置。例如：

```TypeScript
class AnyName {
    @IEditor.menu("App/tool/test", { position: "before openDevTools" } )
    static test() {
        console.log("click menu");
    }
}
```

通过position选项可以设置这个新加的test菜单显示在原有菜单“打开开发者工具 - 编辑器"的前面，而不是默认加到最后。

常用的选项有：

`position` :  设置菜单的位置，支持的语法: "first" / "last" / "before ids" / "after ids" / "beforeGroup ids" / "afterGroup ids"

​      \* "before"和"beforeGroup"的区别是，"before"是插入到参考菜单的前面，而"beforeGroup"是插入到参考菜单前面最近的一个分割线之前。

​      \* "after"和"afterGroup"的区别是，"after"是插入到参考菜单的后面，而"afterGroup"是插入到参考菜单后面最近的一个分割线之后。

​      \* 在同一个类的扩展定义里，默认是添加到上一个扩展的菜单的后面。如果不在同一个类里，不指定position则默认添加到菜单的最后面。

​      \* 多个参考菜单的id值用逗号分隔。

`checkbox` :  设置菜单为一个可以打勾的效果。

`sepBefore` : 在此菜单之前显示一条分割线。

`sepAfter`: 在此菜单之后显示一条分割线。

`enableTest` : 给定返回布尔值的一个函数，在菜单显示之前会执行此函数，用于决定菜单的激活（变灰）状态。**App菜单不支持。**

`visibleTest`: 给定返回布尔值的一个函数，在菜单显示之前会执行此函数，用于决定菜单的显隐状态。仅调用show方法弹出的菜单有效。**App菜单不支持。**

`checkedTest`：给定返回布尔值的一个函数，在菜单显示之前会执行此函数，用于决定菜单的显隐状态。**App菜单不支持。**

下面的例子演示了enableTest的用法。这个新加的菜单， 如果场景中没有选中的物体，则会显示变灰并且无法触发点击回调。

```TypeScript
class AnyName {
    static testEnable() {
        return Editor.secene.getSelection().length > 0;
    }
    
    @IEditor.menu("Hierarchy/test", { enableTest: ()=> AnyName.testEnable() } )
    static test() {
        console.log("click menu");
    }
}
```

## 九、创建菜单

可以创建新菜单，并用代码控制弹出。方法为：

```TypeScript
let menu = IEditor.Menu.create([ 
    { label: "test" , click : function() { console.log("clicked"); } }
 ]);

//当需要弹出时
menu.show();
```

菜单也支持级联，并且不限层数。例如：

```TypeScript
IEditor.Menu.create([ 
    { 
        label: "test" , 
        submenu: [ 
            { label : "a" },
            { 
                label : "b",
                submenu : [
                    { label : "c" }
                ]
            }
        ]
    }
 ]);
```

可以给菜单指定一个ID，通过ID引用菜单。但要注意ID值不要和编辑器内置的菜单或者其他人的菜单的ID冲突。

```TypeScript
IEditor.Menu.create("MyTestMenu", [ 
    { label: "test" , click : function() { console.log("clicked"); } }
 ]);
 
 //当需要弹出时
 IEditor.Menu.getById("MyTestMenu").show();
```

## 十、在场景视图中绘制形状以及交互式手柄

使用IEditorEnv.Gizmos/IEditorEnv.Handles/IEditorEnv.Gizmos2D提供的接口，在场景视图中绘制形状和交互式手柄。假设我们已经有一个自定义的组件Script1，通过IEditorEnv.customEditor这个装饰器，给Script1绑定一个CustomEditor脚本，以实现在编辑器内的自定义编辑。

```TypeScript
//Script1.ts

@regClass()
export class Script1 extends Laya.Script {
    declare owner : Laya.Sprite3D;
}
//TestCustomEditor.ts

@IEditorEnv.customEditor(Script1)
export class TestCustomEditor extends IEditorEnv.CustomEditor {
    declare owner: Laya.Sprite3D;

    onSceneGUI(): void {
        IEditorEnv.Handles.drawHemiSphere(this.owner.transform.position, 2);
    }
    
    onDrawGizmos(): void {
        IEditorEnv.Gizmos.drawIcon(this.owner.transform.position, "editorResources/UI/ready1.png");
    }
}
```

实现效果如下：

![10-1](img/10-1.png)

2D的实现方式有所不同，它必须通过IEditorEnv.Gizmos2D接口，并且目前只支持onDrawGizmosSelected事件，不支持onDrawGizmos和onSceneGUI事件。

以下是2D的一个例子：

```TypeScript
@IEditorEnv.customEditor(Script2)
export class TestCustomEditor extends IEditorEnv.CustomEditor {
    private _c: IEditorEnv.IGizmoCircle;

    onDrawGizmosSelected(): void {
        if (!this._c) {
            let manager = IEditorEnv.Gizmos2D.getManager(this.owner);
            this._c = manager.createCircle(10);
            this._c.fill("#ff0");
        }
        this._c.setLocalPos(10, 10);
    }
}
```

实现效果如下：

![10-2](img/10-2.png)