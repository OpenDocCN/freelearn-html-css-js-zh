# 使用 Canvas 和 D3 绘图

到目前为止，您主要使用 D3 通过 SVG 和 HTML 元素来渲染您的可视化。在本节中，您将学习如何使用 HTML5 Canvas 来绘制和动画化您的可视化。Canvas 可以作为 SVG 的替代品，尤其是在您想在屏幕上渲染更多元素时。您将了解 Canvas 是什么以及它与 SVG 的比较。您将学习如何使用 Canvas 绘制和动画化，以及如何与 D3 一起使用。

在覆盖了基础知识之后，我们将首先使用 SVG，然后使用 Canvas 来可视化飞行路径，以便实际对比和比较两种渲染方法。首先，这将让您对 Canvas 作为 SVG 的替代品有实际的理解。其次，它将展示 Canvas 如何解决您在用 SVG 同时动画化数千个点时可能遇到的问题，因为浏览器在绘制数千个元素的单一图片方面比在内存中构建、存储和维护数千个元素的树结构要快得多。

本章我们将涵盖以下主题：

+   Canvas 及其工具概述：Canvas 上下文

+   如何使用工具绘制 Canvas

+   如何使用 Canvas 动画化绘图

+   如何将 D3 生命周期应用于 Canvas 绘图的各个部分

# 介绍 Canvas

在开始使用 Canvas 绘图之前，让我们简要地看一下其概念——这个心理模型将帮助您接近、规划和编写您的应用程序。Canvas 在其物质形式上是一个单一的 HTML5 元素。它字面上是一个可以绘制的空白画布。对于实际的绘图，您使用**Canvas 上下文**——Canvas API。上下文可以被视为您的工具箱，可以使用 JavaScript 进行操作。

您可以将 Canvas 元素与根 SVG 元素进行比较，因为它们都包含绘图的全部部分。然而，关键的区别在于 SVG（像 HTML 一样）在**保留模式**下操作。浏览器保留了一个列表，其中包含在**文档对象模型**（**DOM**）中绘制到 SVG（或 HTML）画布上的所有对象——您的 web 应用的场景图。这使得您的绘图几乎像物质一样。您产生一个对象列表，通过代码更改样式和属性，并且您可以在任何时候引用这些元素。您可以更改它们的位置，在 DOM 中上下移动它们，并且对于交互来说非常重要——您可以轻松地附加和移除事件监听器。

与之相反，Canvas 以**立即模式**运行。你使用 Canvas 绘制的任何内容都会立即发生，并作为图像保留在画布上。Canvas 中的图像是位图，由像素矩阵或网格组成的数字图像。当你使用 Canvas 绘制时，你使用工具准备每个像素（或更确切地说，指定像素区域）的属性，然后在画布上绘制它们。如果你想改变图像中一个、几个或所有像素的颜色，你需要移除整个图像并生成一个新的图像。与 SVG 不同，你不能回到你想更改的像素，因为它不是以文档树或类似的形式在内存中表示，而是**一次性**烧录到屏幕上。但别担心，你仍然有 Canvas 上下文，它代表了你工具的状态，间接地代表了绘图本身。

你可以将 Canvas 想象成一幅画，而 SVG 或 HTML 则像乐高结构。前者在表示上是静态的。如果你画了一个站在桥上尖叫的人，你不能只是转动他的头。你需要画第二幅画来精确表达这一点。如果你用乐高搭建同样的场景，你可以抓起头部转动它，就像这样：

![](img/18ca93ab-fcee-496c-9982-eddcfde75bdd.png)

立即 Canvas 和保留的 SVG 的戏剧化对比（SVG 图像由 Marco Pece 提供，www.udronotto.it）

考虑到使用 Canvas 进行动画可能需要投入的大量工作，这可能会显得有些繁琐。不仅需要承受绘制这么多图片的心理压力，还需要计算能力来快速连续地重绘一切。但是，正如你将在接下来的章节中看到的，有一些简单的模式可以使 Canvas 动画化和交互式。

# 使用 Canvas 绘制

在我们回到 SVG 和 Canvas 的更详细比较，并看到具体的使用场景之前，让我们学习如何使用 Canvas 绘制。我们将从一个非常简单的例子开始，以了解绘制 Canvas 时涉及到的三个主要步骤。然后你将绘制一系列形状，以熟悉其工具箱。 

作为旁注，当你在查看代码示例时，我强烈建议你使用最新的 Chrome 浏览器。所有代码都在现代浏览器上进行了测试，但主要考虑了 Chrome，因此它将是工作最安全的浏览器。

# 每个 Canvas 视觉的三个绘图步骤

我们可以将 Canvas 绘图分解为三个简单的步骤：

1.  **创建**画布及其上下文。

1.  **配置**上下文。

1.  **渲染**生成的位图。

在你的 DOM 中挂载画布，你需要在 HTML 中创建一个`<canvas>`元素：

```js
<canvas id=”main-canvas” width = “600” height=”400”></canvas>
```

它看起来会空空如也，正如预期的那样：

![](img/6fbdd403-bb07-4971-9b23-ae025dadf1e0.png)

一个空的 Canvas 元素

这将是你在 DOM 中看到的画布的全部内容。所有其他操作将通过 JavaScript 中的 Canvas 上下文来完成。

如果你想在画布上绘制一个矩形，比如使用`royalblue`颜色，你将直接进入 JavaScript 编写代码，而无需回到 HTML：

```js
var canvas = d3.select('#main-canvas').node();
var context = canvas.getContext('2d');

context.fillStyle = 'royalblue';
context.fillRect(50, 50, 200, 100);
```

您可以在此章节的所有代码[`github.com/larsvers/Learning-D3.js-4-Mapping`](https://github.com/larsvers/Learning-D3.js-4-Mapping)中找到。

让我们一步一步地过一遍我们的第一步：

1.  首先，您需要在变量中引用画布。我们将使用`d3.select()`来完成此操作。由于您需要引用画布元素本身，而不是选择，您应该使用 D3 的`selection.node()`方法来获取元素本身。下一行引用了绘制上下文到这个特定的画布元素。上下文包含了您可以使用来绘制的所有工具。您可以通过`console.log('context')`来查看上下文对象：

![图片](img/25d94e62-a290-404d-9f71-8eadfc083561.png)

显示所有属性的上下文对象

上下文在内部被称为`CanvasRenderingContext2D`，但我们将简单地称之为`context`。它包含了您可以用来绘制视觉的所有属性。如果您愿意，也可以扩展`__proto__`对象，这将显示您可用的所有方法。我们将随着解释关键属性和方法，而不会逐个深入。上下文对象的重要之处在于理解有一个对象在帮助您构建绘图。它始终在您身边，允许您使用其方法和更改其属性。

了解上下文 API、其属性和方法的一个好地方是 Mozilla 开发者网络上的文档，网址为[`developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D`](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D)。

在这一点上，您已经有了可以绘制的画布和绘图工具，但您还没有绘制任何东西。

1.  在第二步中，您准备绘图。您配置上下文以产生所需的绘图。示例故意很简单，因为唯一的配置是将我们尚未存在的对象的填充设置为`royalblue`。请注意，`context.fillStyle`是 Canvas 上下文的一个属性，而不是一个方法。就像您是一个画家，告诉您的画笔盒您想要为下一个绘制的对象使用什么颜色一样。

1.  第三步会产生以下图像。`context.fillRect()`接受四个参数：矩形的起始点的`x`和`y`位置以及`width`和`height`。Canvas - 类似于 SVG - 使用一个以左上角为原点 0,0 的笛卡尔坐标系，向右和向下增加。所有值都是以像素为单位：

![图片](img/d2a012ff-fec0-4d76-8143-815ef901f406.png)

一个非常皇家蓝色的画布矩形。

这个矩形不在 DOM 中。您可以在 JavaScript 中看到其配置以及绘制到 DOM 中的画布，但没有`<rect>`元素或类似的内容我们可以引用。再次提醒，不要担心，我们将在接下来的两章中巧妙地重新定位它。

在浏览器中查看此步骤：[`larsvers.github.io/learning-d3-mapping-8-1`](https://larsvers.github.io/learning-d3-mapping-8-1)。代码示例[08_01.html](https://github.com/larsvers/Learning-D3.js-4-Mapping/blob/master/Chapter%2008%20-%20Drawing%20with%20D3%20and%20Canvas/08_01.html)。

在每个步骤的结尾，你将在靠近相关图像的信息框中找到两个链接。第一个链接将带你到一个可以在浏览器中查看的此步骤的工作实现。第二个*代码示例*链接将带你到完整的代码。如果你在读印刷版，你可以在[`github.com/larsvers/Learning-D3.js-4-Mapping`](https://github.com/larsvers/Learning-D3.js-4-Mapping)的相关章节中找到所有代码示例。

你已经看到了在 Canvas 中制作几乎所有绘图的基本步骤。这些关键概念步骤将帮助你接近任何 Canvas 绘图。现在让我们继续绘制。

# 使用 Canvas 绘制各种形状

让我们添加一些其他基本几何形状或**图形原语**到我们的画布中。由于它们是所有可视绘制的构建块，一些练习将对我们有益。以下是我们要绘制的：

![图片](img/81bb3ffd-9c3a-4763-9265-ea6a6909eb7e.png)

一座房子和一棵树。或者一个三角形旁边的路径和圆下的三个矩形。

在浏览器中查看此步骤：[`larsvers.github.io/learning-d3-mapping-8-2`](https://larsvers.github.io/learning-d3-mapping-8-2)。代码示例[08_02.html](https://github.com/larsvers/Learning-D3.js-4-Mapping/blob/master/Chapter%2008%20-%20Drawing%20with%20D3%20and%20Canvas/08_02.html)。

你可以在 JavaScript 控制台右侧看到代码，在我们逐步执行它之前，让我们注意一些一般观察。首先，每一行都以`context`开头。Canvas 上下文确实是我们的绘图开始的地方。其次，Canvas 代码是以过程式风格编写的。这对初学者来说可能是一个好处，因为它是线性的。没有回调，没有嵌套元素结构，只有一条执行线。这种线性也将扩展到时间，一旦你开始动画画布。你首先写第一帧，然后改变场景，然后写第二帧。就像翻书一样简单。让我们逐步查看代码，看看如何详细创建这些元素。我建议的第一件事是给画布添加一个边框。由于画布元素是一个 HTML 元素，你可以用 CSS 来样式化它，但在这里我们使用 JavaScript 来展示画布本身的两个属性：`width`和`height`：

```js
context.strokeStyle = '#CCCCCC';
context.strokeRect(0, 0, canvas.width, canvas.height);
```

`width`和`height`是画布元素拥有的唯一属性。我们在这里使用它们来读取元素的值，然而，它们是可读和可写的。这很好，因为当你想要在动画过程中调整画布大小时，你可以通过编程方式更改画布大小。接下来，我们构建我们的平顶蓝色房子：

```js
context.fillStyle = 'royalblue';
context.fillRect(50, 150, 200, 100);
```

这里没有太多可看的，我们之前已经做过。门也不会让你感到费劲，因为它和房子一样，只是颜色不同：

```js
context.fillStyle = 'rgba(255, 255, 255, 0.9)';
context.fillRect(60, 190, 40, 60);
```

然而，我们使用不同的方法来描述颜色。你可以使用所有 CSS 颜色概念，如命名颜色值和十六进制颜色值，以及 `rgb()`、`rgba()`、`hsl()` 和 `hsla()` 颜色方法。窗口使用 `context.translate()` 放置得略有不同：

```js
context.save();
context.translate(140, 190);
context.fillRect(0, 0, 60, 30);
context.restore();
```

在这种情况下，我们移动的不是矩形，而是整个坐标系！`translate()` 方法接受两个参数：你想要移动坐标系到的 *x* 和 *y* 位置。你已经从使用 `transform, translate(x,y)` 的例子中知道了这个概念，这通常用于在 D3 中移动 `svg:g` 元素并创建它们自己的坐标系。然而，当应用于 `svg:g` 元素时，变换后的坐标系适用于嵌套在 g 元素内的所有对象。如上所述，`g` 元素及其子元素，包括其坐标系，作为场景图表示保留在 DOM 中。在 Canvas 中，我们不能将此信息移动到我们绘图的表示中——没有这样的东西。这取决于你确保只有你想在另一个坐标系上显示的元素才会这样做。记住，当我们谈论 Canvas 代码的进程式风格时？这正是我们在这里必须牢记的。当我们更改 `context` 中的内容时，它将一直持续到我们再次更改它。要更改坐标系，我们可以选择将其移动到我们想要的位置，然后再移动回来，如下所示：

```js
context.translate(140, 190);
context.fillRect(0, 0, 60, 30);
context.translate(-140, -190);
```

但我们更倾向于使用通用的 `context.save()` 和 `context.restore()` 方法。`context.save()` 在代码的这个点保存状态并将其推入 **栈**，而 `context.restore()` 则从栈中弹出最后保存的状态并恢复上下文的先前状态。如果你之前没有遇到过栈，这里有一张图片解释了它是如何工作的：

![](img/55c9677b-a0a1-480e-aa9c-44ba291f5745.png)

数据堆叠的塔楼。

简而言之，栈是一种类似于数组或对象的数据类型。然而，栈仅限于两种操作：在栈顶添加元素（push）和从栈顶移除元素（pop）。它就像一座砖塔。这种对我们应用程序状态的维护是 Canvas 的一个定义性特征，也是与 SVG 的一个关键区别。

接下来，我们给房子加上一个三角形的屋顶。Canvas 中没有 `triangle()` 函数，所以你需要绘制一个路径：

```js
context.beginPath();
context.moveTo(50, 150);
context.lineTo(250, 150);
context.lineTo(50+200/2, 100); // you can use calculations as inputs!
context.closePath();
context.fillStyle = '#A52A2A';
context.fill();
```

最后，我们绘制树。树有一个棕色的茎，你可以将其实现为直线路径，以及一个绿色的树顶，你可以将其绘制为绿色圆圈：

```js
context.beginPath();
context.lineWidth = 10;
context.strokeStyle = 'brown'
context.moveTo(300, 250);
context.lineTo(300, 200);
context.stroke();

context.beginPath();
context.fillStyle = 'green';
context.arc(300, 175, 25, 0, Math.PI * 2);
context.fill();
```

这里有两点需要注意。首先，所有路径代码块都由 `beginPath()` 和 `stroke()`（茎）或 `fill()`（屋顶和树顶）括起来：

```js
context.beginPath();
// configure your path here
context.stroke();

context.beginPath();
// configure your path here
context.fill();
```

`beginPath()`表示开始绘制新路径并移除所有当前路径（或子路径）实现。`stroke()`和`fill()`表示路径的结束，将在屏幕上生成路径。`fill()`将使用设置的`fillStyle`颜色填充路径主体，而`stroke()`将仅使用设置的`strokeStyle()`方法绘制路径轮廓。每次你绘制路径时，你都需要这些起始和结束方法。实际上，每次你绘制任何东西时，你都需要它们。`fillRect()`或`strokeRect()`，如之前所用的，只是开始路径、绘制路径和填充或描边路径的包装器。你可能已经注意到，我们只绘制了三角形屋顶的两边，然后使用了`closePath()`来连接路径的当前终点和起点。`fill()`方法也会为你关闭路径，但明确这样做更为彻底，性能更好，并且有助于你代码的读者（包括你自己）。第二件事要注意的是，即使是圆形也是一个路径。事实上，Canvas API 提供的唯一超出路径的原始形状是矩形。SVG 简化了`<rect>`、`<circle>`、`<ellipse>`、`<line>`、`<polyline>`、`<polygon>`和`<path>`的使用，而 Canvas 只提供路径和矩形。然而，使用路径绘制形状很快就会变得常规。虽然没有预定义的圆形，但有`arc()`和`arcTo()`方法，它们几乎为你完成了圆形绘制。你只需要给它添加颜色，并将其包裹在路径的起始和结束方法中。`arc()`接受五个参数，即*x*和*y*位置、*半径*、弧的*起始*和*结束*角度。这两个角度都是以弧度测量的。

弧度？一个**弧度**等于*57.3*度。弧度是角度的另一种度量单位。数学家们非常喜欢它，因为它们在几何计算中非常有意义。要得到一个弧度，你取圆的半径并将其绕圆周缠绕——如果你能想象半径线是可弯曲的：

![](img/53c20cea-49e6-4796-a838-b1b48daf4bd3.png)

如何得到一个弧度

它们的数学优势是它们可以直接从圆的半径中导出。更美妙的是，半圆（如 180 度）正好是*PI*弧度。因此，一个完整的圆等于*2 * PI*弧度。

度数可能对你来说更有意义。那很好。如果你想在屏幕上移动对象，它们也更有意义。你可以通过使用以下公式轻松地在弧度和度之间进行转换：**(**PI / 180) * degrees**。*PI*是弧度的一半圆，180 是度数的一半圆。通过将它们相除，你将一个度数表示为弧度，等于 0.0175。将任何你想要的度数乘以 0.0175，并将结果用作弧度。

好的！我们已经画出了一个带有房子的风景——那太棒了。Canvas 当然还有更多内容，但通过遵循这些简单的步骤，你已经学到了很多。你学习了使用 Canvas 绘图的概念以及编写过程式代码的含义。你看到了如何使用 Canvas 绘制单个形状，如何使用平移变换移动单个对象，以及 Canvas 中每个形状的原子单位是路径。现在，让我们提高难度，在用 D3 方法之前，先以 Canvas 方式动画化我们的风景。

# 以 Canvas 方式动画化

Canvas 的一个关键优势是动画。当浏览器必须努力重新计算和重新渲染 DOM 中保留的许多元素时，它相对轻松地重新绘制位图图像。在下一节中，你将学习如何使用 Canvas 进行动画。让我们首先看看如何以纯、原生的 Canvas 方式做到这一点。之后，让我们看看我们是否可以使用 D3 的过渡和生命周期进入-更新-退出模式来帮助我们进行动画。这两种方法在用 D3 和 Canvas 构建可视化时都将非常有帮助，因为你将能够为你的想法选择正确的技术或补充这两种方法。

# 以 Canvas 方式动画化

让我们回到我们的房子，并测试它的屋顶是否下雨：

![](img/340c42b9-f372-4ebb-9e18-70752f326cf3.png)

真的在下雨。

在浏览器中查看此步骤：[`larsvers.github.io/learning-d3-mapping-8-3`](https://larsvers.github.io/learning-d3-mapping-8-3)。代码示例 [08_03.html](https://github.com/larsvers/Learning-D3.js-4-Mapping/blob/master/Chapter%2008%20-%20Drawing%20with%20D3%20and%20Canvas/08_03.html)。

在静态图像中很难看到，但在浏览器中查看时，蓝色的小点实际上是在向下移动。它们还以不同的速度移动，这使得雨看起来更加逼真。

# 获得一般概述

在抽象的最高级别，我们的代码如下所示：

```js
var canvas = d3.select('#main-canvas').node();  // set up
var context = canvas.getContext('2d');

var rain = { }                                  // produce data
d3.interval(function() {
    update();                                   // update/process the data
    animate();                                  // re-draw the canvas
}, 10);                                         
```

在设置好画布之后，你将生成一些数据——雨滴。然后你将进入一个循环，在这个循环中，你将更新下一场景的数据，然后绘制它。在我们的例子中，`update()` 函数会改变雨滴的位置，而 `animate()` 函数将清除当前图像并使用更新后的雨滴位置绘制一个新的图像。

这个循环（或者至少是一个非常相似的版本）被称为 **游戏循环**，因为它在 Canvas 游戏编程中被使用。你处理玩家的输入，相应地更新游戏数据，并绘制新的场景。我们将很快习惯这种模式。现在，让我们看看细节。

# 准备雨数据

你正在处理的是雨滴元素。在我们更新或动画化单个雨滴之前，我们首先生成它们。我们正在构建一个所谓的 *对象字面量模块*，名为 `rain`（它是一个 *对象字面量*），它知道如何生成雨滴，并且将单个雨滴保存在一个名为 `items` 的数组中。它看起来会是这样：

```js
var rain = {
    items: [],
    maxDrops: 200,
    getDrop: function() {
      var obj = {};
      obj.xStart = Math.floor(Math.random() * canvas.width);
      obj.yStart = Math.floor(Math.random() * -canvas.height);
      obj.x = null;
      obj.y = null;
      obj.speed = Math.round(Math.random() * 2) + 5;

      return obj;
    },
    updateDrop: // see below
    }
```

`rain` 对象由这个目前为空的数组 `items` 组成，该数组将保存我们产生的所有雨滴对象，还有一个名为 `maxDrops` 的变量，将雨滴数量（`items` 的长度）限制在本例中的 200 个。这可以被认为是小雨。如果你想淹没树木或测试应用程序的性能，可以将这个数字提高到更高的数值。由于我们喜欢这棵树，并且将在接下来的示例中测试性能，所以现在 200 个就足够了。

两个函数将帮助生成和更新雨滴。`getDrop()` 将起始位置分配在画布上方看不见的地方，以及空的 `x` 和 `y` 位置，这些位置将在更新时被填充。你还定义了雨滴的速度，它可以取五到七之间的值。速度将是雨滴在每次更新中向前移动的像素数。一个较低的数值会产生慢雨，而一个较高的数值会产生快雨。

`updateDrop()` 函数可以在我们想要更新雨滴位置的情况下调用。现在让我们这样做。

# 更新每个雨滴

网站加载后，将使用 `d3.interval` 函数启动操作，该函数每 10 毫秒调用它包含的所有函数。首先，它将调用 `update()`，该函数返回一个对象数组。每个对象都是一个雨滴，最显著的特征是任意的 `x` 和 `y` 位置。这就是它的样子：

```js
function update() {
  if (!rain.items.length) {
    d3.range(rain.maxDrops).forEach(function(el) {
      var drop = rain.getDrop(el);
      rain.updateDrop(drop);
      rain.items.push(drop);
    });
  } else {
    rain.items.forEach(function(el) {
      rain.updateDrop(el);
    });
  }
}
```

第一次调用 `update()` 时，它会生成一个雨滴，更新其位置，并将其推入雨滴项目数组。在其他任何情况下，它只是更新雨滴的位置。我们使用 `d3.range` 作为这个循环的便捷方法。它接受一个整数作为输入，并返回一个从 0 开始的整数数组，长度等于你传入的数字。在这里，它帮助我们创建与 `maxDrops` 中指定的数量相等的雨滴。

然后我们使用之前开始描述的 `updateDrop()` 函数更新雨滴的位置：

```js
updateDrop: function(drop) {
  drop.x = drop.x === null ? drop.xStart : drop.x;
  drop.y = drop.y === null ? drop.yStart : drop.y + drop.speed;
  drop.y = drop.y > canvas.height ? drop.yStart : drop.y;
}
```

如果雨滴的 `x` 和 `y` 位置还不存在（如果它们是 `null`），我们将 `xStart` 或 `yStart` 的值分配给它。如果已经存在，我们保持 `x` 位置不变，因为没有雨滴会移动到任何一边，我们将 `y` 位置向下移动一定的速度。这将使雨滴每 10 毫秒向下移动五到七像素。最后一行将每个雨滴回收。一旦它到达底部画布的最终边界，我们只需将其 `y` 值设置为初始的 `yStart` 值。

# 一帧一帧地绘制

在我们的 `d3.interval` 循环中，`update()` 已经运行，我们有了所有雨滴的位置。接下来，我们将处理它们的绘制。如果我们有一个 DOM，我们将与无处不在的 200 个 SVG 圆圈交互，并请求它们友好地向下移动一点。但我们是生成一个静态图像，我们只能绘制而不能更改。所以，我们绘制。就像翻书一样，我们丢弃旧图像并绘制一个新图像。让我们重复这个过程。每次我们想在画布上移动某个东西时，我们**移除****旧图像**并**绘制**一个带有新位置的**新图像**。

这很简单：

```js
function animate() {
  context.clearRect(0, 0, canvas.width, canvas.height);     
  drawScene();
  rain.items.forEach(function(el) {
    circle(context, el.x, el.y, 1.5, 'blue');
  });
}
```

`animate()`使用上下文的`clearRect()`函数，正如其名。你传递给它你想清除的区域——在我们的案例中是整个画布——然后它会将其清除。你也可以填充一个白色矩形或更改`canvas.width`和`canvas.height`的值，但`clearRect()`比第一种方法更快，比第二种方法更清晰。

接下来，你运行`drawScene()`函数，它绘制我们的场景：房子和树。这就是你在上一节中构建的内容，只是封装在一个恰如其分的函数中。

最后，我们将每个雨滴绘制到画布上。你问“`circle()`是什么？”？这是一个构建视觉原语的帮助函数——在我们的案例中是一个圆。它已经被添加到代码的顶部：

```js
function circle(ctx, x, y, r, color) {
  ctx.beginPath();
  ctx.fillStyle = color;
  ctx.arc(x, y, r, 0, 2 * Math.PI);
  ctx.fill();
}
```

两个主要函数`update()`和`animate()`会一直运行，直到你的浏览器标签页的会话结束；这可能意味着一段时间内会有坏天气。

# 画布和 D3

D3 为在网络上生成数据可视化提供了无与伦比的功能。你可能已经意识到了这一点，因为你正在阅读这本书。D3 提供的一个重要部分是其数据注入元素在屏幕上如何演化的模型。它对每个元素的生命周期有特定的思考方式。

在实践中，你将数据注入到一个尚不存在的 DOM 中，D3 会根据你注入的数据创建你选择的新元素，通常每个数据点一个元素。如果你想将新数据注入到 DOM 中，你可以这样做，D3 会识别哪些元素需要新创建，哪些元素可以保留，以及哪些元素应该收拾行李离开屏幕。这样，你可以表示一个常见数据流的三个不同状态：**进入**数据，**更新**数据，和**退出**数据。然后你可以获取这些选择，使用 D3 内置的插值器操作它们的视觉属性，并在它们之间进行过渡。

这与在 DOM 中表现出来的保留 SVG 元素很好地配合。然而，我们在画布中没有 DOM 元素，因此必须稍微聪明一点来生成它们。让我们看看我们如何遵循 D3 的生命周期模型，同时使用 D3 的过渡来在这三种状态之间进行插值。

# 概览我们的实验

我们仍然会让雨在我们的小房子及其树上落下，但现在雨会遵循其自然过程——可以说是雨的生命周期。它将以云的形式**进入**，随着雨移动到地面上一个大水坑中而**更新**，最后它将**退出**，将水坑变成一片郁郁葱葱的草地：

**![](img/b3ca6039-2d84-45f6-bd58-393a6cb80d34.png)**

雨的进入、更新和退出

在浏览器中查看此步骤：[`larsvers.github.io/learning-d3-mapping-8-4b`](https://larsvers.github.io/learning-d3-mapping-8-4b)。代码示例[08_04b.html](https://github.com/larsvers/Learning-D3.js-4-Mapping/blob/master/Chapter%2008%20-%20Drawing%20with%20D3%20and%20Canvas/08_04b.html)。

正如你所见，右侧有按钮允许你控制三个状态变化。

代码的结构将与之前的纯 Canvas 动画类似。从概念上讲，你首先计算元素（雨滴）的位置，然后绘制。然而，我们实现这种交互的方式完全是通过 D3。为了提前揭开盖子，你将数据绑定到虚拟 DOM 元素上。这些 DOM 元素是“虚拟”的。由于 Canvas 没有 DOM，你可以在内存中创建一个基本的 DOM 结构，这样我们就可以使用 D3 的选择、数据绑定以及随后的 Enter-Update-Exit API。此外，应用程序还将有按钮交互，根据更改的数据更改元素的状态。我们已经讨论了 Canvas 设置以及数据准备，所以让我们专注于本节的核心新特性，即数据绑定和绘制！

# 数据

话虽如此，我们应该简要地看一下我们将使用名为`getRainData()`的函数生成的数据。它将给我们 2,500 个雨滴（这次是暴雨），这些雨滴与之前的示例略有不同：

![图片](img/f4cc465b-c0ab-408e-8d50-5d1e0aee1ea2.png)

使用纯 Canvas 方式与使用 D3-and-Canvas 方式创建的雨滴

关键区别在于，对于 D3 雨滴版本，你不需要速度，因为我们计划让 D3 的过渡实现动画。此外，**D3 和 Canvas**雨滴具有一组状态属性，而**纯 Canvas**雨滴只有起始位置和当前位置。

# 更新每个雨滴

数据已经就绪，现在是时候让它动起来了。在纯 Canvas 示例中，你是在画布上绘制每个点，然后计算新点，将前一个点的位置增加五个像素，移除旧图像，并使用高级滴落绘制新图像。你自己转换了这些点。

使用 D3 的过渡方法不同之处在于我们不需要计算新位置，D3 会为我们做这件事。你将数据绑定到选择上，请求 D3 过渡值，在过渡期间，你将重新绘制画布，直到过渡完成。在顶层，你只需要两个函数来完成这个任务：

```js
databind(data) {
  // Bind data to custom elements.
}
draw() {
  // Draw the elements on the canvas.
}
```

这非常直接。

# 绑定数据

然而，D3 在选择上实现过渡，而我们还没有选择。一个 D3 选择是一个绑定数据的元素。使用 D3，你选择一个 DOM 元素，通常是 SVG，将其数据绑定到它上面，你就有了具有所有奇妙方法的选择：显式的`enter()`和`exit()`方法，由`data()`触发的隐式`update()`方法，以及`transition()`及其辅助方法`duration()`和`delay()`，它们控制过渡。

要创建选择，你只需创建类似 DOM 的元素，而且很棒的是，你不需要实体的 DOM 来做这件事。你可以在内存中创建它们。下面是如何做的：

```js
var customBase = document.createElement('custom')
var custom = d3.select(customBase);
```

你可以将 `customBase` 想象成一个根 SVG 元素的替代品，而 `custom` 则是一个完整的 D3 选择。在你的基础搭建好之后，你可以使用 `databind()` 函数将数据绑定到你的自定义元素上：

```js
function databind(data) { }
```

首先，我们将传递给 `databind()` 函数的数据进行连接：

```js
    var join = custom.selectAll('custom.drop')
        .data(data, function(d) { return d.currentIndex; });
```

作为 `data` 第二个参数传入的关键函数在这个情况下并不是必需的，但作为良好实践，它使得连接变得明确，并且可能带来性能上的好处。

现在你创建你的选择状态。`enter` 选择是第一个：

```js
var enter = join
  .enter().append('custom')
    .attr('class', 'drop')
    .attr('cx', function(d) { return d.xCloud; })
    .attr('cy', function(d) { return d.yCloud; })
    .attr('r', function(d) { return d.radiusCloud; })
    .attr('fillStyle', 'rgba(0, 0, 255, 0')
  .transition().delay(function(d, i) { return i * 2; })
    .attr('fillStyle', 'rgba(0, 0, 255, 0.2');
```

关于设置 `fillStyle` 属性的最后一行，有两点需要注意。当你使用 SVG 时，最后一行会是：

```js
.style('color', 'rgba(0, 0, 255, 0.2')
```

但在 Canvas 中，你使用 `.attr()`。为什么？你在这里的主要兴趣是找到一种无痛苦的方式来传输一些元素特定的信息。在这里，你希望将颜色字符串从 `databind()` 传输到 `draw()` 函数。你只是将元素作为一个容器，将你的数据传输到它被渲染到画布上的地方。

这是一个非常重要的区别：当使用 SVG 或 HTML 时，你可以将数据绑定到元素上，并在一步中绘制或应用样式到元素。在 Canvas 中，你需要两步。首先，绑定数据，然后绘制数据。在绑定时不能对元素进行样式设置。它们只存在于内存中，Canvas 不能通过 CSS 样式属性进行样式设置，这正是你使用 `.style()` 时访问的内容。

让我们快速看一下在创建并将 `enter` 选择附加到 `customBase` 元素之后，`customBase` 元素看起来是什么样子：

![图片](img/090c66aa-0950-42d4-bdb3-be14185c7b10.png)

我们的自定义根元素显示了 30 个雨滴的进入状态。

结构看起来很熟悉，不是吗？

接下来，你定义 `update` 选择，最后是 `exit` 选择：

```js
var update = join
  .transition()
    .duration(function() { return Math.random() * 1000 + 900; })
    .delay(function(d,i) { return (i / data.length) * dur; })
    .ease(d3.easeLinear)
    .attr('cx', function(d) { return d.xPuddle; })
    .attr('cy', function(d) { return d.yPuddle; })
    .attr('r', function(d) { return d.radiusPuddle; })
    .attr('fillStyle', '#0000ff');

var exit = join
  .exit().transition()
    .duration(dur)
    .delay(function(d,i) { return i ; })
    .attr('r', function(d) { return d.radiusGrass; })
    .attr('fillStyle', '#01A611');
```

那就是所有放入 `databind()` 中的内容。

# 绘制数据

现在你需要编写 `draw()` 函数来获取屏幕上的元素。在这里我们只是做个笔记，目前还没有发生任何事情。你还没有调用 `databind()`，因为你需要先找到一种方法将它绘制到画布上。所以，我们继续前进。

`draw()` 函数接受你想要绘制的上下文作为参数：

```js
function draw(ctx) {
  ctx.clearRect(0, 0, canvas.width, canvas.height);

  drawRainScene();
  drawScene();

  var elements = custom.selectAll('custom.drop');
  elements.each(function(d, i) {
    var node = d3.select(this);
    ctx.save();
    ctx.beginPath();
    ctx.globalCompositeOperation = 'source-atop'
    ctx.fillStyle = node.attr('fillStyle');
    ctx.arc(node.attr('cx'), node.attr('cy'), node.attr('r'), 0, 2 *
    Math.PI);
    ctx.fill();
    ctx.restore();
  });
```

然后它执行以下操作：

1.  它清除画布。

1.  绘制背景场景，包括房屋和树木，以及 `drawRainScene()` 中绘制的云和积水。

1.  它会遍历我们的每个虚拟元素，并根据我们在 `databind()` 中指定的属性来绘制它。

就这样！你可以关闭 `draw()` 函数了。

看到这一行 `ctx.globalCompositeOperation = 'source-atop'` 吗？`globalCompositeOperation` 允许我们将形状融合或混合。它作用于源形状，即我们即将绘制的形状，以及目标，即源形状下方的画布内容。你可以应用多种合成效果，但我们在这里使用 `source-atop`。

检查[`developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/globalCompositeOperation`](https://developer.mozilla.org/en-US/docs/Web/API/CanvasRenderingContext2D/globalCompositeOperation)以获取所有合成选项。

因此，新形状仅在它与现有画布内容重叠的地方绘制。在没有绘制的画布区域中，形状将不可见。这就是为什么我们需要在`drawRainScene()`中的所有对象。它们构成了雨滴的背景，雨滴无法逃离。顺便说一句，如果你不想手动绘制所有复杂的形状，你可以使用如 Illustrator 这样的矢量图形软件绘制它们，将它们保存为 SVG 格式，然后使用诸如[`www.professorcloud.com/svg-to-canvas/`](http://www.professorcloud.com/svg-to-canvas/)的*SVG 到 HTML5 Canvas 转换器*应用将 SVG 路径转换为 Canvas 命令。

# 运行应用

到目前为止，还没有雨滴出现，但你已经设置了背景场景：

![](img/14a09a8a-eb4b-4e99-a3ea-2a0aafece9ba.png)

初始场景

你有使用`databind()`和`draw()`函数来动画化生命周期的手段。你只需按顺序调用它们，并将这些调用与按钮连接起来。让我们通过进入案例来运行这个。云层是空的，我们希望雨滴从 0 到 0.2 的不透明度过渡到活动状态，这就是我们在`databind()`函数中指定的。我们可以直接运行：

```js
databind(raindata);
draw(context);
```

这将把数据连接到自定义元素中，并且随着所有元素都连接了新数据，将绘制整个选择区域——只需一次！但是我们需要显示过渡效果，因此在过渡发生时需要重复绘制。你可以通过一个连续循环来实现这一点：

```js
databind(data);
var t = d3.timer(function(elapsed) {
  draw(context);
   if (elapsed > dur * 2) t.stop();
});
```

首先，我们将传入的数据绑定到我们的自定义元素上。然后我们重复绘制。`d3.timer()`会反复调用其回调函数，直到你告诉它停止。回调函数接受一个我们称之为`elapsed`的参数，它是计时器运行的时间（以毫秒为单位）。`draw()`函数将会多次运行，绘制背景场景以及每个雨滴。同时，在`databind()`中运行一个过渡，稍微改变每个雨滴的位置。`draw()`函数会在每次调用时，在循环中为每个雨滴选择这些微小的位置变化，并在`databind()`在这个特定时刻设置的非常位置绘制雨滴。你可以将其视为两个同时发生的过程：`databind()`中的过渡提供新的雨滴位置，以及`draw()`中的重复画布绘制移除之前的画布并绘制这些新的雨滴位置。

过渡完成后，我们想要停止计时器。工作完成。`databind()`中的过渡持续 2000 毫秒，正如我们在`dur`变量中设置的。我们现在将使用`dur`来清理我们的工作。我们可以从调用计时器的`.stop()`方法中停止任何计时器。为了安全起见，我们在超过双倍持续时间`dur`（*4000*毫秒）后调用`t.stop()`以适应延迟的过渡。

这就是 D3 在 Canvas 中如何进行过渡。你反复调用绘图函数，与绑定函数大致并行。无论你的 D3 元素设置了哪些过渡样式或位置属性（例如*x*，*y*，*color*，*width*，*height*），它们都将被重新绘制多次，每次绘制都有微小的增量变化。

将其连接到按钮是形式上的。只需将`databind()`和`draw()`函数包装到一个函数中（我们将称之为`rainAnimation()`），当按下进入或更新按钮时传递`raindata`，当按下退出按钮时传递一个空数组。

就这样！

当按下 Enter 按钮时，雨云将出现在屏幕上：

![图片](img/8fa57f3c-d8cf-4b3b-8c49-f520e046a311.png)

将雨进入云中

更新按钮将雨的位置从云更新到水坑：

![图片](img/b81b6df5-fb62-4bf2-a7c9-a47934e6fc74.png)

更新雨到地面

最后，按下 Exit 按钮将雨变成草：

![图片](img/2ba74abc-fe1f-4eb9-a36d-3a9c06b8a475.png)

让雨退出变成草

在浏览器中查看此步骤：[`larsvers.github.io/learning-d3-mapping-8-4a`](https://larsvers.github.io/learning-d3-mapping-8-4a)。代码示例[08_04a.html](https://github.com/larsvers/Learning-D3.js-4-Mapping/blob/master/Chapter%2008%20-%20Drawing%20with%20D3%20and%20Canvas/08_04a.html)。

这是一个自然的 D3 生命周期演示！

# 概述

你已经走得很远了。你已经学会了如何使用 Canvas 绘图，如何以纯 Canvas 方式对 Canvas 进行动画处理，以及如何使用 D3 过渡和 Canvas 的 Enter-Update-Exit 模式。虽然纯 Canvas 方式对于许多应用来说完全合理，但 D3 提供了用于数据可视化的成熟功能，你不必放弃。在构建你的应用程序时，这需要思维上的转变，但它可以特别有利于绘制和动画化大量点。它将以有价值的方式扩展你的工具箱，尤其是在大量数据可能需要元素密集型表示的时候。

在下一章中，我们将回顾 SVG 中的地图可视化，然后在 Canvas 中构建一个。通过这样做，你不仅能够应用本章的学习内容，而且你还将了解更多关于两种方法之间的差异和相似之处，Canvas 如何帮助解决性能瓶颈，以及 D3 如何帮助处理一些繁重的工作。

让我们开始吧！
