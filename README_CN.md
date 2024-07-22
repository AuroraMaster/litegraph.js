# litegraph.js

LiteGraph.js 是一个JavaScript库，用于在浏览器中创建类似于Unreal Blueprints的图形。节点可以轻松编程，并且包含一个编辑器，用于构建和测试图形。

可以轻松集成到任何现有的Web应用程序中，图形可以在不需要编辑器的情况下运行。

在[演示站点](https://tamats.com/projects/litegraph/editor)尝试。

![Node Graph](https://tamats.com/projects/litegraph/imgs/node_graph_example.png "WebGLStudio")

## 特性
- 在Canvas2D上渲染（缩放和平移，易于渲染复杂界面，可以在WebGLTexture内部使用）
- 易于使用的编辑器（搜索框、键盘快捷键、多选、上下文菜单等）
- 优化以支持每个图形中的数百个节点（在编辑器中和执行时）
- 可定制的主题（颜色、形状、背景）
- 回调以个性化节点的每个动作/绘图/事件
- 子图（包含图形的节点）
- 直播模式系统（隐藏图形但调用节点渲染任何内容，有助于创建用户界面）
- 图形可以在NodeJS中执行
- 高度可定制的节点（颜色、形状、插槽垂直或水平、小部件、定制渲染）
- 易于集成到任何JS应用程序中（单个文件，无依赖项）
- TypeScript支持

## 提供的节点
虽然创建新的节点类型很容易，但LiteGraph提供了一些默认节点，可用于许多情况：
- 接口（小部件）
- 数学（三角函数、数学运算）
- 音频（AudioAPI和MIDI）
- 3D图形（WebGL中的后期处理）
- 输入（读取Gamepad）

## 安装

你可以使用npm安装
```
npm install litegraph.js
```

或者从此库下载`build/litegraph.js`和`css/litegraph.css`版本。

## 第一个项目 ##

```html
<html>
<head>
	<link rel="stylesheet" type="text/css" href="litegraph.css">
	<script type="text/javascript" src="litegraph.js"></script>
</head>
<body style='width:100%; height:100%'>
<canvas id='mycanvas' width='1024' height='720' style='border: 1px solid'></canvas>
<script>
var graph = new LGraph();

var canvas = new LGraphCanvas("#mycanvas", graph);

var node_const = LiteGraph.createNode("basic/const");
node_const.pos = [200,200];
graph.add(node_const);
node_const.setValue(4.5);

var node_watch = LiteGraph.createNode("basic/watch");
node_watch.pos = [700,200];
graph.add(node_watch);

node_const.connect(0, node_watch, 0 );

graph.start()
</script>
</body>
</html>
```

## 如何编写新的节点类型

以下是如何构建一个求和两个输入的节点的示例：

```javascript
// 节点构造函数类
function MyAddNode()
{
  this.addInput("A", "number");
  this.addInput("B", "number");
  this.addOutput("A+B", "number");
  this.properties = { precision: 1 };
}

// 显示的名称
MyAddNode.title = "Sum";

// 节点执行时调用的函数
MyAddNode.prototype.onExecute = function()
{
  var A = this.getInputData(0);
  if( A === undefined )
    A = 0;
  var B = this.getInputData(1);
  if( B === undefined )
    B = 0;
  this.setOutputData( 0, parseFloat((A + B).toFixed(this.properties.precision)));
}

// 注册到系统中
LiteGraph.registerNodeType("basic/sum", MyAddNode );

```

或者你可以包装一个现有的函数：

```js
function sum(a, b)
{
   return a + b;
}

LiteGraph.wrapFunctionAsNode("math/sum", sum, ["Number", "Number"], "Number");
```

## 服务器端

它也可以在NodeJS中使用，但某些节点在服务器上不工作（音频、图形、输入等）。

```js
var LiteGraph = require("./litegraph.js").LiteGraph;

var graph = new LiteGraph.LGraph();

var node_time = LiteGraph.createNode("basic/time");
graph.add(node_time);

var node_console = LiteGraph.createNode("basic/console");
node_console.mode = LiteGraph.ALWAYS;
graph.add(node_console);

node_time.connect(0, node_console, 1);

graph.start()
```

## 使用此库的项目

### [comfyUI](https://github.com/comfyanonymous/ComfyUI)

![screenshot](https://github.com/comfyanonymous/ComfyUI/blob/6efe561c2a7321501b1b27f47039c7616dda1860/comfyui_screenshot.png)

### [webglstudio.org](http://webglstudio.org)

![WebGLStudio](https://tamats.com/projects/litegraph/imgs/webglstudio.gif "WebGLStudio")

### [MOI Elephant](http://moiscript.weebly.com/elephant-systegraveme-nodal.html)

![MOI Elephant](https://tamats.com/projects/litegraph/imgs/elephant.gif "MOI Elephant")

### Mynodes

![MyNodes](https://tamats.com/projects/litegraph/imgs/mynodes.png "MyNodes")

## 实用工具

它包括几个实用工具文件夹中的命令来生成文档、检查错误和构建压缩版本。

## 演示
-----

该演示包含一些图形示例。为了尝试它们，你可以访问[演示站点](http://tamats.com/projects/litegraph/editor)或在本地计算机上安装它。为此，你需要`git`、`node`和`npm`。假设这些依赖项已经安装，运行以下命令来尝试：

```sh
$ git clone https://github.com/jagenjo/litegraph.js.git
$ cd litegraph.js
$ npm install
$ node utils/server.js
Example app listening on port 80!
```

打开浏览器并指向 http://localhost:8000/。你可以从页面顶部的下拉列表中选择一个演示。

## 反馈
--------

你可以将任何反馈发送至 javi.agenjo@gmail.com

## 贡献者

- atlasan
- kriffe
- rappestad
- InventivetalentDev
- NateScarlet
- coderofsalvation
- ilyabesk
- gausszhou
