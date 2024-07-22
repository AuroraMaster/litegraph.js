# LiteGraph

这个库分为四个层级：
* **LGraphNode**: 节点的基类（此库使用其自己的继承系统）
* **LGraph**: 由节点组成的整个图的容器
* **LGraphCanvas**: 负责在浏览器中渲染/交互节点的类。

在 ```src/``` 文件夹中还有一个包含的类：
* **LiteGraph.Editor**: LGraphCanvas 周围的一个包装器，添加了按钮。

## LGraph节点

LGraphNode 是用于所有节点类的基类。
为了扩展其他类，在注册时，LGraphNode.prototype 中包含的所有方法都会被复制到这些类中。

当你创建一个新节点类型时，不需要从该类继承，在节点注册时，所有方法都会被复制到你的节点原型中。这是在函数 ```LiteGraph.registerNodeType(...)``` 内完成的。

以下是创建你自己的节点的示例：

```javascript
// 节点构造函数类
function MyAddNode()
{
  // 添加一些输入插槽
  this.addInput("A", "number");
  this.addInput("B", "number");
  // 添加一些输出插槽
  this.addOutput("A+B", "number");
  // 添加一些属性
  this.properties = { precision: 1 };
}

// 在画布上显示的名称
MyAddNode.title = "Sum";

// 节点执行时调用的函数
MyAddNode.prototype.onExecute = function()
{
  // 从输入中检索数据
  var A = this.getInputData(0);
  if( A === undefined )
    A = 0;
  var B = this.getInputData(1);
  if( B === undefined )
    B = 0;

  // 将数据分配给输出
  this.setOutputData(0, parseFloat((A + B).toFixed(this.properties.precision)));
}

// 注册到系统中
LiteGraph.registerNodeType("basic/sum", MyAddNode);

// 可选：自定义节点设置
MyAddNode.prototype.onDrawForeground = function(ctx, graphcanvas)
{
  // 如果节点已折叠，则不绘制任何内容
  if (this.flags.collapsed)
    return;

  // 保存上下文
  ctx.save();

  // 设置填充颜色
  ctx.fillStyle = "#000";

  // 绘制一些文本
  ctx.fillText("Precision: " + this.properties.precision, 10, this.size[1] - 10);

  // 恢复上下文
  ctx.restore();
}

// 可选：捕捉属性更改
MyAddNode.prototype.onPropertyChanged = function(name, value)
{
  if (name === "precision") {
    this.properties.precision = value;
    return true;
  }
  return false;
}
```

### 节点设置

每个节点都可以定义或修改多个设置：
* **size**: ```[width,height]``` 节点内部区域的大小（不包括标题）。每行的高度是 LiteGraph.NODE_SLOT_HEIGHT 像素。
* **properties**: 包含用户可以配置的属性的对象，在保存图形时会被序列化
* **shape**: 对象的形状（可以是 LiteGraph.BOX_SHAPE, LiteGraph.ROUND_SHAPE, LiteGraph.CARD_SHAPE）
* **flags**: 用户可以更改的标志，在序列化时会被存储
  * **collapsed**: 是否显示为折叠（小尺寸）
* **redraw_on_mouse**: 当鼠标经过小部件时强制重绘
* **widgets_up**: 小部件不从插槽之后开始
* **widgets_start_y**: 小部件应从这个 Y 坐标开始绘制
* **clip_area**: 渲染节点时剪辑内容
* **resizable**: 是否可以通过拖动角落调整大小
* **horizontal**: 插槽是否应水平放置在节点的顶部和底部

用户可以定义多个回调：
* **onAdded**: 添加到图形时调用
* **onRemoved**: 从图形中移除时调用
* **onStart**: 图形开始运行时调用
* **onStop**: 图形停止运行时调用
* **onDrawBackground**: 在画布上渲染自定义节点内容（在实时模式中不可见）
* **onDrawForeground**: 在画布上渲染自定义节点内容（在插槽上方）
* **onMouseDown,onMouseMove,onMouseUp,onMouseEnter,onMouseLeave**: 捕捉鼠标事件
* **onDblClick**: 在编辑器中双击时调用
* **onExecute**: 执行节点时调用
* **onPropertyChanged**: 在面板中属性改变时调用（返回 true 以跳过默认行为）
* **onGetInputs**: 以 [ ["name","type"], [...], [...] ] 的形式返回可能的输入数组
* **onGetOutputs**: 返回可能的输出数组
* **onSerialize**: 序列化之前调用，接收一个用于存储数据的对象
* **onSelected**: 在编辑器中选中时调用，接收一个用于读取数据的对象
* **onDeselected**: 从编辑器中取消选中时调用
* **onDropItem**: DOM 项目拖放到节点上时调用
* **onDropFile**: 文件拖放到节点上时调用
* **onConnectInput**: 如果返回 false，传入的连接将被取消
* **onConnectionsChange**: 连接更改（新连接或移除连接）时调用（LiteGraph.INPUT 或 LiteGraph.OUTPUT, 插槽, 如果已连接则为 true, link_info, input_info）


### 节点槽位

每个节点可以有多个插槽，存储在 `node.inputs` 和 `node.outputs` 中。

你可以通过调用 `node.addInput` 或 `node.addOutput` 添加新的插槽。

输入和输出之间的主要区别在于，输入只能有一个连接链接，而输出可以有多个。

要获取有关插槽的信息，可以访问 `node.inputs[slot_index]` 或 `node.outputs[slot_index]`。

插槽具有以下信息：

* **name**: 插槽的名称字符串（也用于在画布中显示）
* **type**: 指定通过此链接传输的数据类型的字符串
* **link 或 links**: 取决于插槽是输入还是输出，包含链接的 ID 或 ID 数组
* **label**: 可选，在画布中显示时用于重命名名称的字符串
* **dir**: 可选，可以是 `LiteGraph.UP`, `LiteGraph.RIGHT`, `LiteGraph.DOWN`, `LiteGraph.LEFT`
* **color_on**: 连接时渲染的颜色
* **color_off**: 未连接时渲染的颜色

要检索通过链接传输的数据，可以调用 ```node.getInputData``` 或 ```node.getOutputData```。

### 定义图节点

在为图节点创建类时，这些要点可能会有帮助：

- 构造函数应创建默认的输入和输出（使用 ```addInput``` 和 ```addOutput```）
- 可以编辑的属性存储在 ```this.properties = {};``` 中
- ```onExecute``` 是图执行时调用的方法
- 可以通过定义 ```onPropertyChanged``` 来捕捉属性是否发生了变化
- 必须使用 ```LiteGraph.registerNodeType("type/name", MyGraphNodeClass );``` 注册你的节点
- 可以通过定义 ```MyGraphNodeClass.priority``` 来改变默认的执行优先级（默认是 0）
- 可以使用 ```onDrawBackground``` 和 ```onDrawForeground``` 覆盖节点的渲染方式

### 自定义节点外观

如果想让节点的形状或标题颜色不同于主体颜色，可以进行配置：
```js
MyNodeClass.title_color = "#345";
MyNodeClass.shape = LiteGraph.ROUND_SHAPE;
```

可以使用回调 ```onDrawForeground``` 和 ```onDrawBackground``` 在节点内部绘制内容。唯一的区别是 onDrawForeground 在实时模式中被调用，而 onDrawBackground 不会。

这两个函数接收 [Canvas2D 渲染上下文](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D) 和渲染节点的 LGraphCanvas 实例。

无需担心坐标系统，(0,0) 是节点内容区域的左上角（不包括标题）。

```js
node.onDrawForeground = function(ctx, graphcanvas)
{
  if(this.flags.collapsed)
    return;
  ctx.save();
  ctx.fillColor = "black";
  ctx.fillRect(0,0,10,this.size[1]);
  ctx.restore();
}
```

### 自定义节点行为

如果你的节点具有某种特殊的交互性，可以捕获鼠标事件。

第二个参数是节点坐标中的位置，其中 0,0 代表节点内容的左上角（标题下方）。

```js
node.onMouseDown = function( event, pos, graphcanvas )
{
    return true; //如果事件被节点使用，则返回 true 以阻止其他行为
}
```

其他方法有：
- onMouseMove
- onMouseUp
- onMouseEnter
- onMouseLeave
- onKey

### 节点小部件

可以在节点内添加小部件来编辑文本、值等。

为此，必须在构造函数中通过调用 ```node.addWidget``` 创建它们，返回的值是包含所有小部件信息的对象，方便存储，以便以后从代码中更改值。

语法如下：

```js
function MyNodeType()
{
  this.slider_widget = this.addWidget("slider","Slider", 0.5, function(value, widget, node){ /* do something with the value */ }, { min: 0, max: 1} );
}
```
以下是支持的小部件列表：
* **"number"** 用于更改数字值，语法如下：```this.addWidget("number","Number", current_value, callback, { min: 0, max: 100, step: 1, precision: 3 } );```
* **"slider"** 通过拖动鼠标更改数字值，语法与 number 相同。
* **"combo"** 用于在多个选项中进行选择，语法如下：

  ```this.addWidget("combo","Combo", "red", callback, { values:["red","green","blue"]} );```

  或者如果想使用对象：

  ```this.addWidget("combo","Combo", value1, callback, { values: { "title1":value1, "title2":value2 } } );```

* **"text"** 用于编辑短字符串
* **"toggle"** 像一个复选框
* **"button"**

第四个可选参数可以是小部件的选项，接受的参数有：
* **property**: 指定小部件更改时要修改的属性名称
* **min**: 最小值
* **max**: 最大值
* **precision**: 设置小数点后的位数
* **callback**: 值更改时调用的函数。

存储节点状态时，默认情况下不会序列化小部件的值，但如果你想存储小部件的值，只需将 serialize_widgets 设置为 true：

```js
function MyNode()
{
  this.addWidget("text","name","");
  this.serialize_widgets = true;
}
```

或者如果你想将小部件与节点的属性关联起来，则在选项中指定它：

```js
function MyNode()
{
  this.properties = { surname: "smith" };
  this.addWidget("text","Surname","", { property: "surname"}); //这将修改 node.properties
}
```
## LGraphCanvas
LGraphCanvas 是负责在浏览器中渲染/交互节点的类。

## LGraphCanvas 设置
可以定义或修改图画布设置以更改行为：

* **allow_interaction**: 设置为 `false` 时禁用与画布的交互（节点上的 `flags.allow_interaction` 可用于覆盖图画布设置）

### 画布快捷键
* 空格键 - 按住空格键同时移动光标会移动画布。当按住鼠标按钮时也可以使用，这样当画布变得过大时可以更容易地连接不同的节点。
* Ctrl/Shift + 点击 - 将点击的节点添加到选择中。
* Ctrl + A - 选择所有节点
* Ctrl + C/Ctrl + V - 复制并粘贴选定的节点，不保留与未选定节点输出的连接。
* Ctrl + C/Ctrl + Shift + V - 复制并粘贴选定的节点，并保留从未选定节点的输出到新粘贴节点的连接。
* 按住 Shift 并拖动选定的节点 - 同时移动多个选定的节点。

# 执行流程
要执行图表，必须调用 ```graph.runStep()```。

此函数将为图表中的每个节点调用方法 ```node.onExecute()```。

执行顺序由系统根据图表的形态确定（没有输入的节点被认为是第 0 级，然后连接到第 0 级节点的节点是第 1 级，以此类推）。此顺序仅在图表形态更改时计算（创建新节点、更改连接）。

开发人员需要决定如何从节点内部处理输入和输出。

通过 ```this.setOutputData(0,data)``` 发送的数据存储在链接中，因此如果通过该链接连接的节点执行 ```this.getInputData(0)```，它将收到发送的相同数据。

对于渲染，根据用户与 GraphCanvas 的交互情况（单击的节点会移动到数组的后面，以便最后渲染），节点按 ```graph._nodes``` 数组中的顺序执行。

## 集成

在你的 HTML 应用中集成：

```js
var graph = new LiteGraph.LGraph();
var graph_canvas = new LiteGraph.LGraphCanvas(canvas, graph);
```

如果想启动图表：

```js
graph.start();
```

## 事件

当我们在图表中运行一步（使用 ```graph.runStep()```）时，每个节点的 onExecute 方法都会被调用。
但有时你希望动作仅在某些触发器被激活时执行，这种情况下可以使用事件。

事件允许在从一个节点派发事件时仅在节点中触发执行。

要为节点定义插槽，必须使用类型 LiteGraph.ACTION 作为输入，LIteGraph.EVENT 作为输出：

```js
function MyNode()
{
  this.addInput("play", LiteGraph.ACTION );
  this.addInput("onFinish", LiteGraph.EVENT );
}
```

现在要在从输入接收到事件时执行一些代码，必须定义方法 onAction：

```js
MyNode.prototype.onAction = function(action, data)
{
   if(action == "play")
   {
     //执行你的动作...
   }

}
```

最后是当节点中发生某些事情时触发事件。可以从 onExecute 中或任何其他交互中触发它们：

```js
MyNode.prototype.onAction = function(action, data)
{
   if( this.button_was_clicked )
    this.triggerSlot(0); //触发插槽 0 中的事件
}
```

已经有一些节点可以处理事件，如延迟、计数等。

### 自定义链接工具提示

在连接两个节点的链接上悬停时，会显示一个工具提示，允许用户查看从一个节点输出到另一个节点的数据。

有时，你可能会有一个节点输出一个对象，而不是一个可以轻松表示的原始值（例如字符串）。在这些情况下，工具提示将默认显示 `[Object]`。

如果需要更具描述性的工具提示，可以通过向你的对象添加一个 `toToolTip` 函数来实现，该函数返回你希望在工具提示中显示的文本。

例如，要确保从输出插槽 0 显示 `A useful description`，输出对象如下：

```javascript
this.setOutputData(0, {
  complexObject: {
    yes: true,
  },
  toToolTip: () => 'A useful description',
});
```

这样，当用户将鼠标悬停在输出插槽 0 的链接上时，他们将看到 `A useful description` 而不是默认的 `[Object]`。









