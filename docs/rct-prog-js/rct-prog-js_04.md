# 第四章。演示非功能性反应式编程-一个实时示例

在本章中，我们将看一个实时示例，其中融合了一些反应式编程原则与 ReactJS。程序的一些部分仍然是命令式的，作为之前用 jQuery 编写的东西的一个端口，经过 HP-28S RPN 和 Unix C 的其他端口之后，但是 ReactJS 的强大仍然闪耀，即使像现实世界中的大部分代码一样，它已经经历了多次迭代。我们将简要地看一下网页的 HTML 要求，然后再看 JavaScript 中真正的内容。该网页提供了一个最初在 HP-28S 图形科学计算器上开发的视频游戏的端口，并保留了计算器的外观和感觉。

在本章中，我们将涵盖以下主题：

+   网页的 HTML

+   动画网页的 JavaScript

在这里，我们看到游戏，渲染在经典的 HP28S 计算器的背景下。已经采取了一些措施，使字符图形模仿了 LED 屏幕上存在的暗色和亮色：

![演示非功能性反应式编程-一个实时示例](img/B04108_04_1.jpg)

# 具有多个端口的游戏的历史

标题指定我们正在制作 HP28S RPN 游戏的一个端口，所以让我们来看一下我们正在实现的特定游戏的一点历史。

这个游戏有不同的实现和不同的端口，包括在 C 中的重新实现，以及使用 HTML 或 JavaScript 的几种方式。原始版本是在 HP28S 上，这是一个黑客式的科学计算器，可以有 32KB 或 512KB 的 RAM（我的有 512KB）。编程和使用（两者在 Unix/Linux shell 编程中并没有太大的不同）**逆波兰表示法**（**RPN**）（[`en.wikipedia.org/wiki/Reverse_Polish_notation`](http://en.wikipedia.org/wiki/Reverse_Polish_notation)）。在计算器中有很多有趣的深度，我记得我做了两个程序。一个是一个具有谦卑的二维*蹒跚醉酒*算法的分形屏幕保护程序（参见[`tinyurl.com/reactjs-staggering-drunk`](http://tinyurl.com/reactjs-staggering-drunk)），另一个是将在这里重新实现的视频游戏。

基本游戏采用 dingbat 字符图形实现，一艘太空船从左到右移动，在一个从级别到级别变得更加密集的小行星场中。主要的游戏机制旨在躲避你所经历的小行星。你也可以射击小行星。这真的是必要的，因为一些（天真地）随机绘制的级别并不一定有明确的路线可用。为了阻止游戏机制简单地射击通过每个级别，射击小行星是受到惩罚的，并且意味着更多的是最后的手段而不是旨在作为主要游戏机制的操纵。我的一个朋友评论说，这是他所知道的第一个玩家实际上因射击东西而失去分数的视频游戏。

# 网页的 HTML

我们以标准的 HTML5 DOCTYPE 开头：

```js
<!DOCTYPE html>
```

随后，我们打开文档，指定 UTF-8 作为字符集。如果网页正确提供，字符集应该在页面下载时被指定，但这在防御性编码方面仍然可能有所帮助，这总是值得记住的事情：

```js
<html lang="en">
  <head>
    <meta charset="utf-8" />
```

因此文档标题：

```js
    <title>A video game on an HP-28S scientific
      calculator</title>
```

这里使用的字体是复古的 VT 系列字体，与受人尊敬的 VT100 和其他系列的 Unix 终端相关联。请注意-正如稍后将在代码中看到的那样-虽然 VT100 系列是等宽终端，但该字体并不严格是等宽字体，只是在行内显示每行空格或小行星将产生不希望的间距，因此每个字符都被绝对定位。也许另一个字体可能不会有这个问题，但 VT100 字体有一种很好的复古色彩。

请注意，我们将为大部分字符图形包括装饰符号。它们在 JavaScript 中处理。

像其他在 HTML 中使用的标签一样，字体标签是通过 HTTP/HTTPS 两个斜杠的模糊格式编写的，`http:`或`https:`不指定，并且被提供为与网页中相同的格式：

```js
  <link
    href='//fonts.googleapis.com/css?family=VT323'
     rel='stylesheet' type='text/css' />
```

## 在任何可以的地方使用内容分发网络

我们从**内容分发网络**（**CDN**）加载 ReactJS，遵循史蒂夫·索德斯广泛建立的 Yslow（“我的网页为什么加载慢？”）建议。

### 注意

史蒂夫·索德斯（[`SteveSouders.com`](http://SteveSouders.com)）最初在雅虎发现，渲染网页更快实际上并不是关于削减服务器端性能的毫秒或微秒。在影响客户端更高效方面有一个显著的低成本果实，例如，当可以从计算机缓存中以闪电般的速度加载资源时，不再一遍又一遍地从网络加载相同的资源。

有很多 JavaScript 库和框架可以从 CDN 中获取，包括 ReactJS，但几乎任何其他你想要使用的主要或次要 JavaScript 工具也都可以找到。

## 一些简单的样式

我们为页面添加了一些基本样式。背景图像是从[`haywardfamily.org/hp28s.png`](http://haywardfamily.org/hp28s.png)加载的。如果需要，你可以制作本地副本，或者如果在 HTTPS 上有问题，或者如果你在本地提供文件时 HTTP/HTTPS 模糊格式遗憾地无法工作。

**p#display**中的文本颜色取自 HP28S 计算器的屏幕截图：

![一些简单的样式](img/B04108_04_2.jpg)

```js
<style type="text/css">
  body
  {
  background-image:
  url(//haywardfamily.org/hp28s.png);
  background-position: top left;
  background-repeat: no-repeat;
  height: 670px;
  width: 860px;
  }
  div#main
  {
  height: 670px;
  width: 860px;
  }
  p#display
  {
  color: #4f5c65;
  font-family: VT323, courier, sans;
  font-size: 18px;
  letter-spacing: 4px;
  left: 565px;
  top: 180px;
  position: absolute;
  }
  p#legend
  {
  background-color: rgba(0, 0, 0, .6);
  border-radius: 20px;
  color: white;
  font-family: Verdana, arial, sans;
  margin-left: 40px;
  margin-right: 90px;
  margin-top: 40px;
  padding: 10px;
  }
</style>
</head>
```

这是页面头部的最后一部分。

## 一个相当简单的页面主体

我们构建页面主体，其中包含 HP-28S 计算器的图像作为背景。我们还包括一个简短的图例和游戏在虚拟计算器屏幕上显示的空间：

```js
<body>
  <div id="main">
    <p id="legend">
      Arrow keys to move, Space to shoot.
    </p>
    <p id="display">
    </p>
  </div>
```

在关闭 body 标签之前，我们加载主要脚本，它将使用 ReactJS 来为游戏添加动画：

```js
<body>
  <script
    src="img/react.js"></script>
  <script type="text/javascript"
    src="img/hp28s.js"></script>
</body>
</html>
```

这是页面的 HTML 结束。现在将跟随包含真正的编程内容的 JavaScript。

# 动画页面的 JavaScript

我们可能简要指出，脚本是常规的 JavaScript，而不是 ReactJS 的 JSX 格式，它允许混合类似 HTML 的 XML 和 JavaScript，并被称为在脚本中放置尖括号的工具。并非所有人都会使用 JSX，但如果没有其他选择，了解这个工具也是值得的。

JSX 有很多优点，值得考虑。它被一些非 Facebook ReactJS 用户使用，但并非所有人都使用，同时也被 Facebook 使用。Facebook 一直支持 JSX，但并没有要求使用 JSX 来使用 ReactJS。开发目的上，JSX 脚本可以在网页加载后从[`cdnjs.cloudflare.com/ajax/libs/react/0.13.3/JSXTransformer.js`](http://cdnjs.cloudflare.com/ajax/libs/react/0.13.3/JSXTransformer.js)加载，并在浏览器中编译。生产目的上，它们需要在 JavaScript 中编译，这时你可以运行`npm install jsx`，然后从命令行运行`jsx`编译器，具体操作可参考[`www.npmjs.com/package/jsx`](https://www.npmjs.com/package/jsx)。

## 简短的语法说明 - 立即调用的函数表达式

在我们的脚本中，我们使用**立即调用函数表达式**（**IIFE**），以便我们的局部变量，使用`var`关键字在函数或其依赖项的某处定义，将受到闭包内的私有保护。这将避免共享可变状态的问题（因为它具有非共享的可变状态）。共享的可变状态会使程序的稳定性取决于任何有足够访问权限来修改状态的人。由于 JavaScript 语法的怪癖，该函数被括号包裹，其中以函数开头的行被视为函数定义，因此以下语法将不起作用：

```js
function() {
}();
```

解决方案是将函数放在一对括号中，然后它将正常工作：

```js
(function()
    {
    })();
```

回到我们的主脚本！

## 变量声明和初始化

我们的主`wrapper`函数通过编写仅在函数中使用的状态变量，开始非反应性和命令性地运行：

```js
function()
{
  var game_over_timeout = 2000;
  var game_over_timestamp = 0;
  var height = 4;
  var tick_started = false;
  var width = 23;
  var chance_clear;
  var game_over;
  var level;
  var rows;
  var score;
  var position;
  var row;
```

声明并在某些情况下初始化了这些变量后，我们继续进行游戏启动的函数。这将初始化或重新初始化变量，但不包括初始化关卡。它通过将`game_over`变量设置为`false`，将玩家放在第 1 级，设置（水平）位置在屏幕/小行星区域的左侧 23 个字符宽度的开始处，以及垂直位置在第 1 行（顶部下方的第 1 行，共 4 行），得分为 0，并且大多数空间都是清晰的（即，没有小行星，因此玩家的飞船可以安全地行驶）但是小行星的密度不断增加！这 5/6 是空间不被小行星占据的机会呈指数衰减的开始。后者是一个可以调整的参数，以影响游戏的整体难度，较低的值会使领域更难以导航。在关卡之间，在空间清晰的机会呈指数衰减；指数衰减的速率，或者该功能的其他方面也是可以修改以影响关卡之间游戏难度的。

在这里，我们可以看到当玩家几乎清除了第一关时显示的样子：

！[变量声明和初始化]（img/B04108_04_3.jpg）

生成的关卡大部分是空格，有随机的可能性出现小行星，但最初容纳飞船的空间和其前面的空间总是清晰的。这是为了让玩家有一些空间来做出反应，而不是自动死亡。

## 用于启动或重新启动游戏的函数

在声明后立即调用该函数：

```js
    var start_game = function()
    {
      game_over = false;
      level = 1;
      position = 0;
      row = 1;
      score = 0;
      chance_clear = 5 / 6;
    }
    start_game();
```

## 创建游戏关卡的函数

在`get_level()`函数中，构建了一个级别。空间清晰的概率经历了指数衰减（衰减后的第一个级别为 0.75，然后为 0.675，然后为 0.6075，依此类推），随着小行星的密度相应增加，然后构建了一个矩形的*字符数组的数组*（字符数组用于集合中的字符，这些字符经历了接近恒定的变化，而不是字符串，字符串是不可变的，即使原始实现操作了字符串）。请注意，在这里的内部表示中，事物是由字符代码表示的：`a`表示小行星，`s`表示玩家的飞船，空格表示空白，依此类推。（现在将一个数组的数组的字符存储为对其他字符的引用可能有点奇怪。在原始的遗留系统上，现在显而易见的方法尚不可用。现在可能会对其进行重构，但是本章是一个旨在类似于处理遗留代码时获得良好结果的代码章节，这个瑕疵的存在是有意的。大多数开发人员所做的工作包括与遗留功能进行接口。）最初，所有空间都有可能被小行星占据。之后，清除了飞船的初始槽和其前面的空间。这是一个例子：

```js
var get_level = function(){
  level += 1;
  rows = [];
  result = {};
  chance_clear *= (9 / 10);
  for(var outer = 0; outer < height; ++outer)
  {
    rows.push([]);
    for(var inner = 0; inner < width; ++inner)
    {
      if (Math.random() > chance_clear)
      {
        rows[outer].push('a');
      }
      else
      {
        rows[outer].push(' ');
      }
    }
  }
  rows[1][0] = 's';
  rows[1][1] = ' ';
  return rows;
}
```

虽然这个函数返回行的网格，但是行的网格将被分配为将与 ReactJS 一起使用的对象的字段。ReactJS 与对象上的属性一起工作得更好，而不是与数组上的属性一起工作。

前面函数调用的结果存储在 board 变量的一个字段中，并为按键定义了一个数组。在移动结束时，最后一个按键（如果有）从`keystrokes`数组中取出，然后清空数组，以便飞船根据上一次按键（如果有）在回合期间输入的按键移动。所有其他按键都将被忽略：

```js
    var board = {rows: get_level()};
    var keystrokes = [];
```

## 使用 ReactJS 类来动手实践

现在我们将直接开始与 ReactJS 进行交互。我们创建一个 ReactJS 类，使用特定字段命名的函数哈希。例如，`componentDidMount()`函数和字段是在 ReactJS 组件挂载时调用的函数。这意味着它基本上是在 DOM 中显示和表示的。在这种情况下，我们向文档的主体添加事件侦听器，而不是直接向 ReactJS 组件添加事件。这是因为我们想要监听按键按下/按键事件，而很难让`DIV`对这些事件做出响应。因此，主体添加了事件侦听器。它们将处理 ReactJS 中的事件处理程序，这些事件处理程序仍然应该像通常显示它们一样定义。请注意，一些其他类型的事件，例如一些鼠标事件（至少），将通过 ReactJS 通常的方式注册，如下所示：

```js
    var DisplayGrid = React.createClass({
      componentDidMount: function()
      {
        document.body.addEventListener("keypress",
        this.onKeyPress);
        document.body.addEventListener("keydown",
        this.onKeyDown);
      },
```

ReactJS 中的组件具有**属性**和**状态**。属性是一次定义的东西，不能更改。它们在 ReactJS 内部可用，并且应该被视为不可变的。状态是可变的信息。属性和状态都在 ReactJS 中可用。我们可以简要地评论说，Facebook 和 ReactJS 正确地将共享的可变状态视为引发 Heisenbugs。在这里，我们从闭包中处理所有可变状态。可变状态不是共享的，也不应该共享（换句话说，它是非共享的可变状态）。

```js
      getDefaultProps: function()
      {
        return null;
      },
      getInitialState: function()
      {
        return board;
      },
```

接下来，我们定义按键和按键事件处理程序，就像它们通常被使用的那样，或者至少在 DIV 响应按键事件时通常被处理的那样。（实际上，我们将监视 body，因为与悬停或鼠标点击不同，按键相关事件不会传播到包含的 DIV。这近似于您通常如何演示 ReactJS 中的事件处理。我们正在监听的特定按键，箭头键和空格键，存在一个问题。实质上，箭头键触发按键按下事件，但不触发按键事件（大多数其他键会触发按键事件）。这种行为很愚蠢，但它已经深入到 JavaScript 中，现在基本上是不可协商的。我们委托给一个通用的事件处理程序来处理这两个事件。在这里，按键被转换为键码：左或上箭头键向上移动（或向左，从游戏的方向来看），右或下箭头键向下移动（或向右，从游戏的方向来看），空格键射击。这些分别由`keystrokes`数组中的`u`、`d`和`s`表示。

```js
      onKeyDown: function(eventObject)
      {
        this.onKeyPress(eventObject);
      },
      onKeyPress: function(eventObject)
      {
        if (eventObject.which === 37 ||
        eventObject.which === 38)
        {
          keystrokes.push('u');
        }
        else if (eventObject.which === 39 ||
        eventObject.which === 40)
        {
          keystrokes.push('d');
        }
        else if (eventObject.which === 32)
        {
          keystrokes.push('s');
        }
      },
```

在这一点上，我们创建了`render()`函数，这是一个核心的 ReactJS 成员来定义。这个渲染函数的作用是创建 DIV 和 SPAN，以适当的方式表示空格和符号的网格。叶节点被绝对定位。

在构建了叶 SPAN 节点和中间 DIV 后，我们构建到主 DIV 元素。

`out_symbol`变量是一个 UTF-8 字符，而不是 ASCII 转义；这是有一个非常具体的原因。尽管 ReactJS 有一个明确定义的转义口 `dangerouslySetInnerHTML()`（参见[`tinyurl.com/reactjs-inner-html`](http://tinyurl.com/reactjs-inner-html)），通常设置为抵抗 XSS（跨站脚本）攻击。因此，它的正常行为是转义尖括号和大量的和符号使用。这意味着`&nbsp;`将被渲染为源码中的`&nbsp;`，而不是一个（不换行和不折叠）空格。因此，我们使用的 dingbat 符号不像其他地方那样使用转义码（尽管这些转义码在这里作为注释留下了），而是作为 UTF-8 存储在 JavaScript 中。

如果您不确定如何输入 dingbats，您可以简单地使用其他东西。或者，您可以将注释中的转义复制到**普通的简单 HTML**（**POSH**）文件中，然后从渲染的 POSH 页面中复制并粘贴半打符号到您的 JavaScript 源代码中。您的 JavaScript 源代码应该被视为 UTF-8。

```js
  render: function()
  {
  var children = ['div', {}];
  for(var outer = 0; outer <
  this.state.rows.length; outer += 1)
  {
    var subchildren = ['div', null];
    for(var inner = 0; inner <
    this.state.rows[outer].length;
    inner += 1)
    {
      (var symbol =
      this.state.rows[outer][inner];)
      var out_symbol; 
      if (symbol === 'a')
        {
          // out_symbol = '&#9632;';
          out_symbol = '■';
        }
        else if (symbol === 's')
        {
          // out_symbol = '&#9658;';
          out_symbol = '►';
        }
        else if (symbol === ' ')
        {
          // out_symbol = '&nbsp;';
          out_symbol = ' ';
        }
        else if (symbol === '-')
        {
          out_symbol = '-';
        }
        else if (symbol === '*')
        {
        out_symbol = '*';
        }
        else
        {
          console.log('Missed character: '
          + symbol);
        }
        subchildren.push(
        React.createElement('span',
        {'style': {'position': 'absolute',
          'top': 18 * outer - 20), 'left':
          (12 * inner - 75)}}, out_symbol));
        }
        children.push(
        React.createElement.apply(this,
        subchildren));
      }
      return React.createElement.apply(this,
      children);
    }
  });
```

在前面的代码中定义的 children 和 subchildren 填充了`React.createElement()`的参数列表。

在内部循环中构建完毕后，我们向`subchildren`数组添加一个叶节点。它被指定为一个 span，内联 CSS 样式以哈希的形式传递，并且内容等于`out_symbol`变量。然后将其添加到`children`数组中。它包含屏幕的行，这些行最终构建成完整的棋盘。

在 ReactJS 中，组件是在`React.createElement()`中定义的，随后可以供使用。`React.createElement()`的常规调用方式是`React.createElement( 'div', null, ...)`，省略号部分包含所有的子元素。我们使用`apply()`来调用`React.createElement()`，并传入所需的初始参数，然后在数组中指定参数。

## 滴答滴答，游戏的时钟在滴答

这关闭了`render()`字段和`React.createElement()`类定义。在源代码中，我们继续进行`tick()`函数的处理。它处理每一轮应该发生的事情。目前，代码以 300 毫秒（0.3 秒）的间隔调用`tick()`，尽管这是可以调整的，以影响游戏玩法，或者稍微重构以使游戏玩法随着级别的提高而加速。

如果游戏结束，这只能是因为飞船撞到了小行星，`tick()`调用中什么也不会发生：

```js
    var tick = function()
    {
      if (game_over)
      {
        return;
      }
```

接下来，调用`React.render()`，指定要呈现的类以及要呈现到的 HTML 元素。`React.render()`应该至少在每次想要呈现东西的时候调用。如果你只调用一次，它将只呈现一次，这意味着如果你想要重复的更新显示，就需要重复调用它。在这里，我们在`tick()`方法的每次调用中调用它，要求基于前面大部分代码中定义的`DisplayGrid`创建一个元素，并将呈现的 HTML 放入具有显示 ID 的 DIV 中：

```js
    React.render(
      React.createElement(DisplayGrid, {}),
      document.getElementById('display'));
```

在这里，我们看到玩家射击了一个小行星的屏幕。小行星爆炸成了一个星号！

![滴答，滴答——游戏的时钟在滴答](img/B04108_04_4.jpg)

如果在上一轮中，飞船射击了一个小行星（在字符符号中表示为零个或多个连字符和右侧的星号；连字符填充了射击到的小行星之前的空间，星号代表了射击命中小行星的爆炸），我们清除显示在那一轮中射击的槽：

```js
    for(var outer = 0; outer < height; outer += 1)
    {
      for(var inner = 0; inner < width; inner += 1)
      {
        if (board.rows[outer][inner] === '-' ||
        board.rows[outer][inner] === '*')
        {
          board.rows[outer][inner] = ' ';
        }
      }
    }
```

做完这些之后，我们清除了指示已经射击的变量：

```js
    var shot_taken = false;
```

我们清除了飞船所在的空间：

```js
    board.rows[row][position - 1] = ' ';
```

在每个滴答结束时，`keystrokes`数组被清除，并且我们注意存储的最后一个击键。换句话说，我们关注上一个回合之后存储的最后一个击键。击键在回合之间不会累积。在回合结束时，最后一个击键是唯一获胜的击键。

`keystroke`数组存储的是键码，而不是确切的击键。箭头键已经被处理，左箭头或上箭头按下将存储一个`u`表示向上，右箭头或下箭头按下将存储一个`d`表示向下，空格键按下将存储一个`s`表示射击。如果有人输入上或下，飞船将在边界内向上或向下移动：

```js
    if (keystrokes.length)
    {
      var move = keystrokes[keystrokes.length – 1];
      if (move === 'u')
      {
        row -= 1;
        if (row < 0)
        {
          row = 0;
        }
      }
      else if (move === 'd')
      {
        row += 1; 
        if (row > height - 1)
        {
          row = height - 1;
        }
      }
```

如果用户射击了一个小行星，在下一轮中，一排连字符将从飞船的前端延伸到小行星，小行星将变成一个星号，表示爆炸：

```js
    else if (move === 's')
    {
      shot_taken = true;
      score -= 1;
      var asteroid_found = false;
      for(var index = position + 1; index <
      width && !asteroid_found; index += 1)
      {
        if (board.rows[row][index] === 'a')
        {
          board.rows[row][index] = '*';
          asteroid_found = true;
        }
        else
        {
          board.rows[row][index] = '-';
        }
      }
    }
    keystrokes = [];
  }
```

### 提示

**下载示例代码**

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)的帐户中下载您购买的所有 Packt Publishing 图书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便直接通过电子邮件接收文件。

## 游戏结束

如果用户撞到了小行星，游戏结束。然后我们显示一个**游戏结束**屏幕，并停止进一步处理，就像这样：

![游戏结束](img/B04108_04_5.jpg)

```js
        if (position < width)
            {
            if (rows[row][position] === 'a')
                {
                game_over = true;
                game_over_timestamp = (new
                  Date()).getTime();
                (document.getElementById(
                  'display').innerHTML =
                  '<span style="font-size: larger;">' +
                  'GAME OVER' +
                  '<br />' +
                  'SCORE: ' + score + '</span>');
                return;
                }
```

只要用户没有撞到小行星，游戏仍在进行，我们用飞船的标记替换行中的当前槽。然后增加玩家的（水平）位置：

```js
            board.rows[row] = board.rows[row].slice(0,
              position).concat(['s']).concat(
              board.rows[row].slice(position + 1));
            position += 1;
            }
```

如果用户“掉出了屏幕的右边缘”，我们将游戏提升到下一个级别：

```js
        else
            {
            rows = get_level();
            score += level * level;
            position = 0;
            row = 1;
            }
        }
```

定义了所有这些之后，我们开始游戏，如果我们还没有启动滴答声，我们会以 300 毫秒的间隔启动滴答声（这个值可以调整，以使游戏更容易或更难；它可以成为一个可配置的间隔，随着游戏的进行而加快）：

```js
    start_game();
    if (!tick_started)
        {
        setInterval(tick, 300);
        tick_started = true;
        }
    })();
```

# 总结

在本章中，我们涵盖了很多内容。早些时候，我们已经涵盖了一些理论，但是在这里，我们开始使用一些 ReactJS 来拼凑一个应用程序。以后，我们将使用一个更深入的应用程序。

涵盖的主题包括网页的 HTML。这是一个简单的 HTML 骨架，用作保存反应式 JavaScript 的架子。另一个涵盖的主题是反应式 JavaScript。这包括了 JavaScript 的混合，其中有一个清晰的示例，展示了如何为 ReactJS 编写反应式 JavaScript。

我们将在下一章继续介绍函数式编程。
