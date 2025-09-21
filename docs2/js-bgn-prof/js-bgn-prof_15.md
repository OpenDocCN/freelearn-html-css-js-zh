

# 下一步

你已经走得很远了！在这个阶段，你应该已经掌握了 JavaScript 的基础知识。你能够创建应用程序、编写巧妙的脚本，并且能够阅读大量的代码。这为接下来的重要步骤打下了坚实的基础。在本章中，我们将通过实践和探索 JavaScript 提供的无限可能性中你感兴趣的部分，将你所学的知识提升到下一个层次。

我们不会在这里过多地详细介绍所有这些主题。这些细节很快就会过时，并且互联网上有大量针对每个主题精心制作的教程和信息。可能性很大，在你阅读这段文字的时候，我们推荐的框架和库已经过时了。好消息是，下一个大事件使用相同概念的可能性非常大。

本章将作为你在 JavaScript 中下一步的起点。我们将涵盖以下主题：

+   库和框架

+   学习后端

+   下一步

    注意：练习、项目和自我检查测验的答案可以在*附录*中找到。

# 库和框架

让我们从库和框架开始。库基本上是预先编程的 JavaScript 模块，你可以使用它们来加速你的开发过程。它们通常为你做一件特定的事情。框架非常相似，它们也是预先编程的，但它们不仅仅为你做一件事情，而是安排了一系列的事情。这就是为什么它被称为框架，它确实为你提供了一个坚实的起点，并且通常要求你的文件有一定的结构才能做到这一点。框架通常是一组提供一站式解决方案的库。或者至少是多合一的解决方案。你最终甚至会发现自己在框架之上使用外部库。

以一个非代码的例子来说明，如果我们开始建造一辆汽车，我们可以从头开始，并且自己制作汽车的所有部件。这正是我们在这本书中到目前为止所做的事情。有了库，我们可以获得现成的部件——在我们的汽车例子中，我们可以获得完全建成的椅子，我们只需要将其安装到我们已建造的汽车底盘上。如果我们使用框架来制作汽车，我们会得到汽车的骨架，其中已经包含了所有必要的部件，而且它可能已经能够驾驶了。我们只需要专注于定制汽车并确保它包含我们想要和需要的所有特殊功能。在这样做的时候，我们必须牢记我们已有的汽车骨架，并继续以这种方式进行。

如您所想，如果我们使用库和框架，我们的汽车项目将会更快完成。此外，使用库和框架时遇到的问题也会更少，因为预先准备的部分已经被许多人测试过了。如果我们从头开始制作自己的汽车座椅，那么一年后它们可能就不再舒适了，而标准的解决方案已经被彻底检查过了。

因此，库和框架不仅加快了进程，还为你提供了一个更稳定、经过更好测试的解决方案。难道没有缺点吗？当然有。其中最重要的可能是灵活性，因为你将不得不遵循你所使用的框架的结构。在某种程度上，这也可以是一个优点，因为它通常要求你采用良好的编码风格，这将提高代码质量。

另一个缺点是，你将不得不在所使用的框架或库更新时不断更新你的应用程序。这非常重要，尤其是当更新是针对安全问题的修复时。一方面，框架和库非常可靠，但由于它们被广泛使用，黑客发现弱点并不罕见。如果他们找到了一个，这将给他们提供许多应用程序的机会，包括你的应用程序。另一方面，你自己的代码可能比平均框架要弱得多。

然而，在许多情况下，破解你定制的应用程序可能成本太高。例如，当你只是有一个在线的爱好项目时，你可能不会支付大量的赎金给黑客，而且你应用程序中的数据也可能不值得黑客的努力。而一个试图利用常用框架在随机数量网站上弱点的脚本却是常见的。为了最小化风险，经常更新你的依赖项，并关注你库或框架所有者报告的弱点。

## 库

技术上，我们无法使用框架和库做比不用它们更多的事情。也就是说，如果我们不考虑时间。框架和库使我们能够更快地开发出高质量的软件，这也是它们如此受欢迎的原因。

我们将在这里讨论一些最受欢迎的库。这绝对不是一个独家列表，它也非常动态，所以一年后其他库或框架可能更受欢迎。这就是为什么我们不会在这里涵盖完整的教程和如何开始的原因。我们只会解释基本原理并展示一些代码片段。然而，这仍然是你在开发生涯中迈出下一步的稳固基础。

许多库可以通过在 HTML 的头部添加一个 script 标签来包含在页面中，如下所示：

```js
<script src="img/librarycode.js"></script> 
```

我们将从讨论一些常见的库开始。

### jQuery

**jQuery** 可以说是最著名的 JavaScript 库。在过去，它会被编译成特定浏览器的最新版本的 JavaScript，使用起来非常方便。如今，它只是以不同的方式编写我们在书中看到的一些东西。你可以通过代码中的美元符号数量轻松识别 jQuery。你还可以通过在网站的控制台中输入 `$` 或 `jQuery` 来判断一个网站是否使用了 jQuery，如果它返回 jQuery 对象，则表示该网站正在使用 jQuery。jQuery 库主要关注从 DOM 中选择 HTML 元素，并与它们交互和操作。它大致看起来像这样：

```js
$(selector).action(); 
```

使用美元符号表示你想要开始使用 jQuery，而使用选择器可以选中 HTML 中的元素。这里的符号有点像 CSS：

+   一个简单的字符串值可以定位一个 HTML 元素：`$("p")`

+   一个单词或短语前的点号表示你想要选择具有特定类的所有元素：`$(".special")`

+   一个井号可以定位具有特定 ID 的元素：`$("#unique")`

+   你也可以使用任何其他 CSS 选择器，包括更复杂的链式选择器

下面是一个例子，jQuery 库从第 3 行开始导入到 `script` 元素中：

```js
<html>
  <head>
    <script src="img/jquery.min.js"></script>
  </head>
  <body>
    <p>Let's play a game!</p>
    <p>Of hide and seek...</p>
    <p class="easy">I'm easy to find!</p>
    <button id="hidebutton">Great idea</button>
    <button id="revealbutton">Found you!</button>
    <script>
      $(document).ready(function () {
        $("#hidebutton").click(function () {
          $("p").hide();
        });
        $("#revealbutton").click(function () {
          $(".easy").show();
        });
      });
    </script>
  </body>
</html> 
```

这就是页面看起来像这样：

![文本自动生成的描述，置信度中等](img/B16682_15_01.png)

图 15.1：包含简单 jQuery 脚本的页面

当你点击 **好主意** 按钮时，所有段落都将被隐藏。这是在 jQuery 添加的事件中完成的。首先，我们选择了具有 ID `hidebutton` 的按钮，然后我们调用它的 `click` 函数，该函数指定了点击时会发生什么。在该函数中，我们声明将选择所有 `p` 元素并将它们隐藏。`hide` 是一个特殊的 jQuery 函数，它将 `display:none` 样式添加到 HTML 元素中。

因此，点击后，所有段落都消失了。当我们点击 **找到你了** 时，只有一个返回，最后一个读作 **我容易找到**。这是因为当具有 ID `revealbutton` 的按钮被点击时，它会选择所有具有类 `easy` 的元素，并使用 jQuery 的 `show` 函数移除样式中的 `display:none`。

这就是 jQuery 真正的核心所在：

+   掌握选择器

+   了解一些额外的或不同命名的函数来操作元素

你可以在你的代码中使用 jQuery，但这不会扩大你使用 JavaScript 做更多事情的可能性。它只会让你用更少的代码字符做同样的事情。jQuery 非常受欢迎的原因是，在浏览器标准化程度较低的时候，它增加了许多价值，在这种情况下，使用 jQuery 实际上提供了跨多个浏览器标准化 JavaScript 的解决方案。如今，这已经没有多少用处了，如果你要编写新的代码，最好直接使用 JavaScript。然而，无论何时你在处理旧代码，你很可能遇到 jQuery，因此了解它是如何工作的将肯定有助于你处理这些情况。

在撰写本文时，你可以在以下位置找到 jQuery 文档：[`api.jquery.com/`](https://api.jquery.com/)。

### D3

**D3** 代表三个“D”：**数据驱动文档**。它是一个 JavaScript 库，可以帮助基于数据进行文档操作，并且可以使用 HTML、SVG 和 CSS 来可视化数据。对于需要包含任何类型数据表示的仪表板来说，它非常有用。

使用 D3，你可以用很多特性制作几乎任何你想要的图表。它可能看起来相当令人畏惧，因为需要设置图表图形的所有设置。深入其中，将其分解成小块，将确保你克服任何障碍。下面是一个非常基本的示例，使用 D3 向 SVG 添加三个球体：

```js
<!DOCTYPE html>
<html>
<head>
    <script src="img/d3.v7.min.js"></script>
    <style>
        svg {
            background-color: lightgrey;
        }
    </style>
</head>
<body>
    <svg id="drawing-area" height=100 width=500></svg>
    <script>
        let svg = d3.select("#drawing-area");
        svg.append("circle")
            .attr("cx", 100).attr("cy", 50).attr("r", 20).style("fill", "pink");
        svg.append("circle")
            .attr("cx", 200).attr("cy", 20).attr("r", 20).style("fill", "black");
        svg.append("circle")
            .attr("cx", 300).attr("cy", 70).attr("r", 20).style("fill", "grey");
    </script>
</body>
</html> 
```

D3 库在第一个`script`标签中导入。`svg`变量是通过在具有 ID `drawing-area` 的`svg`上使用`d3.select`方法创建的。

我们并没有完全公正地对待 D3 的可能性——在这种情况下，这并不比只用 canvas 做更多。然而，你可以制作漂亮的数据动画，例如缩放效果、可排序的条形图、球体的旋转效果等等。不过，这些代码将占据这本书的多个页面。

在撰写本文时，你可以在以下位置找到完整的文档：[`devdocs.io/d3~4/`](https://devdocs.io/d3~4/)。

### Underscore

Underscore 是一个 JavaScript 库，可以概括为函数式编程的工具包。函数式编程可以被认为是一种编程范式，它围绕使用描述性函数的序列进行，而不是单独的例子。**面向对象编程**（**OOP**）也是一种编程范式，它全部关于对象及其状态，数据可以被封装并隐藏在外部代码之外。在函数式编程中，函数非常重要，但需要关注的州更少。这些函数总是用不同的参数做同样的事情，并且可以很容易地串联起来。

```js
1, 2, and 3:
```

```js
<!DOCTYPE html>
<html>
<head>
    <script src="img/underscore-umd-min.js"></script>
</head>
<body>
    <script>
        _.each([1, 2, 3], alert);
    </script>
</body>
</html> 
```

有许多其他用于过滤、分组元素、转换元素、获取随机值、获取当前时间等功能。

这段代码可能也解释了名字的由来，因为我们通过下划线访问 Underscore 函数。不过，你首先需要安装 Underscore，否则解释器将不理解语法。

在撰写本文时，你可以在以下位置找到完整的文档：[`devdocs.io/underscore/`](https://devdocs.io/underscore/)。

### React

React 是我们将要讨论的最后一个前端库。如果你认为 React 是一个框架，你并不完全错误，但也不完全正确。我们之所以将 React 视为一个库，是因为你需要使用其他一些库才能达到它像框架一样的效果。

React 用于构建美观且动态的用户界面。它将页面拆分为不同的组件，并且当数据发生变化时，数据在组件之间发送和更新。以下是一个非常基础的示例，它只是触及了 React 能做什么的皮毛。这个 HTML 将在页面上显示这句话：**Hi Emile, what's up?**：

```js
<div id="root"></div> 
```

当以下 JavaScript 与之关联时，它将这样做：

```js
ReactDOM.render(
  <p> Hi Emile, what's up?</p>,
  document.getElementById('root');
); 
```

这只会在 React 库可用时才有效。它将渲染 DOM，用 `render` 函数的第一个参数替换 `div` 的 `innerHTML`。我们可以通过在头部添加一个 `script` 元素来实现这一点，而不需要在我们的系统上安装任何东西。完整的脚本看起来像这样：

```js
<!DOCTYPE html>
<html>
<head>
    <script src="img/react.development.js" crossorigin></script>
    <script src="img/react-dom.development.js" crossorigin></script>
</head>
<body>
    <div id="root"></div>
    <script>
        let p = React.createElement("p", null, "Hi Emile, what's up?");
        ReactDOM.render(
            p,
            document.getElementById("root");
        );
    </script>
</body>
</html> 
```

这将使用在 `script` 标签中手动创建的 React 元素将 **Hi Emile, what's up?** 写入页面。不过，对于大型项目来说，你不应该这样做。使用包管理器（如 **Node Package Manager** （**NPM**））设置 React 和所有你需要的东西要更有价值。这将允许你轻松管理所有依赖项并保持你的代码井井有条。

在撰写本文时，更多信息可以在这里找到：[`reactjs.org/docs/getting-started.html`](https://reactjs.org/docs/getting-started.html)。

## 框架

框架更复杂，通常你需要在你的电脑上安装它们。如何在特定框架的在线文档中找到如何安装的信息。并且，每当你完成编码并想要运行你的代码时，你都需要运行一个命令，该命令将处理你的代码，使其成为浏览器可以理解的东西。当我们这样做时，我们“提供”应用程序。

### Vue.js

Vue.js 是一个轻量级的 JavaScript 框架。它可以用来构建用户界面和 **单页应用程序** （**SPAs**）。使用 Vue.js 编写的用户界面可能在你第一次遇到它时难以理解。看看这个代码示例：

```js
<!DOCTYPE html>
<html>
  <script src="img/vue"></script>
  <body>
    <div id="app">
      <p v-if="!hide">
        Let's play hide and seek. <br />
        Go to the console and type: <br />
        obj._data.hide = true <br />
      </p>
    </div>
    <script>
      let obj = new Vue({
        el: "#app",
        data: {
          hide: false,
        },
      });
    </script>
  </body>
</html> 
```

这是一个简单的 HTML 页面，从 Vue 导入了一个 JavaScript 链接。在 `<p>` 标签的 HTML 中有一些奇怪的事情发生：有一个 `v-if` 元素。这个元素只有在 `v-if` 中的条件为真时才会显示。

在这种情况下，它正在查看我们 Vue 实例中数据对象的 `hide` 属性。如果你将这个 `hide` 的值改为 `true`，否定的 `hide` 语句将变为 `false`，元素将消失。这本来也是我们可以不用 Vue 就做到的，但那样的话，我们就需要指定一个 JavaScript 事件来改变值，并使用 JavaScript 来编辑 CSS 以隐藏段落。

你甚至可以看到对你来说全新的 HTML 元素。这是因为这些不是常规的 HTML 元素，而是来自 Vue，它允许你定义自己的元素。你可能会遇到如下这样的 HTML：

```js
<div id="custom-component">
  <maaike></maaike>
</div> 
```

当你打开与之关联的网页时，它会显示：

```js
Maaike says: good job! 
```

```js
maaike component. Here is the snippet:
```

```js
<script>
    Vue.component("maaike", {
        template: "<p>Maaike says: good job!</p>",
    });
    new Vue({ el: "#app" });
</script> 
```

在前面的代码中，创建了一个新的 Vue 组件，实际上它可以持有数据，也可以有函数，但这个组件非常基础，只是为了说明我们可以在`template`属性中添加 HTML 模板。指定了一段段落。当网页加载时，`<maaike>`组件将被模板中的内容所替换。

一页的内容可以来自多个文件。通常这些组件各自都有自己的文件。一旦你深入到 Vue.js 中，你会了解到更多官方的 Vue 工具。实际上，它是一个非常适合初学者的框架，因为它非常清晰，是理解框架的绝佳起点。

在撰写本文时，你可以在以下链接找到完整的 Vue 文档：[`v3.vuejs.org/guide/introduction.html`](https://v3.vuejs.org/guide/introduction.html)。

### Angular

Angular 是一个由 Google 发起并（目前）维护的框架。Angular 比 Vue.js 重得多，但可以被视为一个完整的包。这意味着 Angular 占用更多的磁盘空间，通常这意味着编译和安装速度较慢。查看 Angular 代码与 Vue.js 并没有太大的区别。然而，Angular 使用 TypeScript 而不是 JavaScript。TypeScript 是 JavaScript 的超集，会被编译成 JavaScript，但它更严格，也有不同的语法。

Angular 可以通过 HTML 中的`ng`属性来识别。我们不会展示一个完整的示例，但这里有一个 HTML 示例，它将显示待办事项列表上的所有任务（当周围的代码设置正确时）：

```js
<ul>
  <li ng-repeat="task in tasks">
    {{task}}<span ng-click="deleteTask($index)">Done</span>
  </li>
</ul> 
```

`ng-repeat`属性指定了重复操作，对于任务列表上的每个任务，它应该创建一个`<li>`元素。`task`也可以在`<li>`内部作为变量使用，如`{{ task }}`所示。

还有另一个 Angular 特有的功能，`ng-click`，它告诉 Angular 当元素被点击时应该做什么。这与 JavaScript 的`onclick`事件类似，但现在它可以动态绑定。这意味着在编写代码时，你不需要了解`onclick`。显然，你可以在 JavaScript 中通过指定将导致`onclick`属性（以及必要时整个元素）变化的事件来实现相同的功能，但这需要编写更多的代码。这适用于 Angular 中的任何内容：它可以用 JavaScript 完成，但这需要更多的工作（这实际上可能是一个低估，取决于情况的复杂性）。

在撰写本文时，你可以在以下链接找到完整的文档：[`angular.io/docs`](https://angular.io/docs)。

学习如何使用 React、Angular 或 Vue 等库和框架是一个非常合理甚至可以说是必须的下一步，如果你希望成为一名前端开发者。在作者看来，这些选项的难度并没有太大的区别。哪个是最好的选择取决于你想要工作的地点和你所在的地区，因为这些框架和库在不同地区有不同的偏好。

# 学习后端

到目前为止，我们只处理了前端。前端是运行在客户端的部分，可以是用户使用的任何设备，例如手机、笔记本电脑或平板电脑。为了使网站能够执行有趣的功能，我们还需要后端。例如，如果你想登录到一个网站，这个网站需要以某种方式知道这个用户是否存在。

这就是服务器端代码，后端的工作。这是在用户的设备上运行，而是在某个服务器上运行的代码，通常由托管网站的公司的所有或租赁。托管网站通常意味着他们通过将网站放置在可以接受外部请求的 URL 上的服务器，使其对整个互联网可用。

服务器上的代码执行了许多与更深层逻辑和数据相关的事情。例如，一个电子商务商店在商店中有许多来自数据库的商品。服务器从数据库中获取商品，解析 HTML 模板，并将 HTML、CSS 和 JavaScript 发送到客户端。

登录也是如此：当你在一个网站上输入用户名和密码并点击登录时，服务器上的代码会被触发。这段代码将验证你输入的详细信息与数据库中的信息是否匹配。如果你有正确的详细信息，它将发送一个登录用户的门户页面给你。如果你输入了错误的详细信息，它将向客户端发送错误信息。

在本节中，我们将介绍前端和后端之间通信的基础，并展示你如何使用 Node.js 编写后端代码。

## APIs

**API**（**应用程序编程接口**）本质上是一个用更多代码编写的代码接口。可以使用（例如）URL 来向 API 发出请求。这将触发一段特定的代码，这段代码将返回一个特定的响应。

这一切都很抽象，所以让我们用一个例子来说明。如果我们有一个酒店网站，人们能够在线预订房间是有意义的。这需要我们有一种 API。每当用户填写完所有字段并点击 **提交预订** 时，API 将通过调用 URL 并将用户输入的所有数据发送到该端点（一个特定的 URL）来触发，例如：`www.api.hotelname.com/rooms/book`。这个 API 将处理和验证我们的数据，当一切正常时，它将在我们的数据库中存储房间预订，并可能向我们的客人发送确认邮件。

当酒店职员去检查预订时，将使用其中一个端点发起另一个 API 调用。例如，这个端点可能看起来像这样：`www.api.hotelname.com/reservations`。这将首先检查我们的员工是否以正确的角色登录，如果是，它将从数据库中检索所选日期范围内的所有预订，并将包含结果的页面发送回我们的员工，然后员工可以看到所有预订。因此，API 是逻辑、数据库和前端之间的连接点。

API 通过 **Hypertext Transfer Protocol** （**HTTP**） 调用工作。HTTP 只是一个用于双方（客户端和服务器，或服务器和另一个服务器，其中请求服务器充当客户端）之间通信的协议。这意味着它必须遵守对方期望的某些约定和规则，对方将以某种方式做出回应。例如，这意味着使用特定的格式来指定头信息，使用 GET 方法获取信息，使用 POST 方法在服务器上创建新信息，以及使用 PUT 方法更改服务器上的信息。

可以使用 API 做更多的事情，例如，你的计算机和打印机也通过 API 进行通信。然而，从 JavaScript 的角度来看，这并不太相关。

你将在 *AJAX* 部分看到如何消费这些 API。你也可以编写自己的 API，而如何做到这一点的最终基础知识可以在 *Node.js* 部分找到。

## AJAX

**AJAX** 代表 **Asynchronous JavaScript and XML**，这是一个误称，因为如今更常见的是使用 JSON 而不是 XML。我们使用它来从前端向后台发起调用，而不需要刷新页面（异步）。AJAX 不是一个编程语言或库，它是浏览器内置的 `XMLHttpRequest` 对象和 JavaScript 语言的组合。

作为前端开发者，你可能现在不会在日常工作中使用纯 AJAX，但它被隐藏在表面之下，所以了解它是如何工作的不会有害。以下是一个使用 AJAX 调用后端的示例：

```js
let xhttp = new XMLHttpRequest();
let url = "some valid url";
xhttp.load = function () {
    if (this.status == 200 && this.readyState == 4) {
        document.getElementById("content").innerHTML = this.responseText;
    }
};
xhttp.open("GET", url, true);
xhttp.send(); 
```

这不是一个工作示例，因为没有有效的 URL，但它演示了 AJAX 是如何工作的。它设置了当请求被加载时需要执行的操作，在这种情况下，用链接返回的内容替换元素 ID 为`content`内的 HTML。这可以是一个文件的链接，或者是一个调用数据库的 API。当数据库中有其他（或没有）数据时，它可以给出不同的响应。这个响应是 JSON 格式的，但它也可以是 XML 格式的。这取决于服务器是如何编写的。

现在更常见的是使用**Fetch API**进行 AJAX 请求。这与我们可以用`XMLHttpRequest`做到的事情类似，但它提供了一套更灵活、更强大的功能。例如，在下面的代码中，我们通过`json()`方法从 URL 获取数据，将其转换为 JSON，并将其输出到控制台：

```js
let url = "some valid url";
fetch(url)
  .then(response => response.json())
  .then(data => console.log(data)); 
```

Fetch API 与承诺（promises）一起工作，这一点到现在应该已经很熟悉了。所以当承诺被解决后，会通过`then`创建一个新的承诺，当这个承诺被解决后，下一个`then`会被执行。

在撰写本文时，更多相关信息可以在这里找到：[`developer.mozilla.org/en-US/docs/Web/Guide/AJAX/Getting_Started`](https://developer.mozilla.org/en-US/docs/Web/Guide/AJAX/Getting_Started).

### 练习题 15.1

创建一个 JSON 文件，并使用`fetch`将结果作为可用的对象返回到你的 JavaScript 代码中：

1.  创建一个 JSON 对象，并将其保存到名为`list.json`的文件中。

1.  使用 JavaScript，将文件名和路径分配给名为`url`的变量。

1.  使用`fetch`向文件 URL 发起请求。将结果作为 JSON 返回。

1.  一旦响应对象准备就绪，遍历数据并将结果输出到 JSON 文件中每个项目的控制台。

## Node.js

我们可以使用 Node.js 在 JavaScript 中编写 API。Node.js 是一个非常聪明的运行环境，它采用了 Google JavaScript 引擎，进行了扩展，并使得 JavaScript 能够在服务器上运行，通过 JavaScript 与文件系统协议和 HTTP 进行交互。正因为如此，我们可以使用 JavaScript 进行后端开发。这意味着你可以只用一种语言（连同 HTML 和 CSS）来编写前后端。如果没有 Node.js，你将不得不使用其他语言，如 PHP、Java 或 C#来编写后端。

为了运行 Node.js，你首先需要设置它，然后运行`node nameOfFile.js`命令。你可以在官方 Node.js 文档中找到如何在你的系统上设置它的方法。通常需要下载和安装某些东西，然后你就可以完成了。

在撰写本文时，下载说明可以在[`nodejs.org/en/download/`](https://nodejs.org/en/download/)找到。

这里有一些代码示例，它将接收可以编写在 Node.js 中的 HTTP 调用：

```js
const http = require("http");
http.createServer(function(req, res){
    res.writeHead(200, {"Content-Type": "text/html"}); //header status
    let name = "Rob";
    res.write(`Finally, hello ${name}`); //body
    res.end();
}).listen(8080); //listen to port 8080
console.log("Listening on port 8080... "); 
```

我们首先导入 `http` 模块。这是一个需要导入以运行的外部代码文件。`http` 模块随 Node.js 一起提供，但其他模块可能需要安装。你将使用包管理器来完成这项工作，例如 NPM，它将帮助安装所有依赖项并管理所有外部模块的不同版本。

上面的代码设置了一个监听端口 `8080` 的服务器，每次访问时，它将返回 `Finally, hello Rob`。我们使用导入的 `http` 模块的 `createServer` 方法创建服务器。然后我们说明了对我们的服务器进行调用时需要发生什么。我们以 200 状态（表示“OK”）响应，并将 `Finally, hello Rob` 写入响应。然后我们指定默认端口 `8080` 作为监听端口。

这个例子使用了 Node.js 的内置 `http` 模块，这对于创建 API 非常强大。这绝对是一件事，值得有一些经验。能够编写自己的 API 将使你能够自己编写完整的应用程序。当我们添加 Express 时，这变得更加容易。

### 使用 Express Node.js 框架

Node.js 不是一个框架，也不是一个库。它是一个运行环境。这意味着它可以运行和解释编写的 JavaScript 代码。有针对 Node.js 的框架，目前 Express 是最受欢迎的一个。

这里是一个非常基础的 Express 应用程序——同样，你将不得不首先设置 Node.js，然后添加 Express 模块（如果你使用 NPM，`npm install express` 将会完成）并使用 `node nameOfRootFile.js` 命令运行它：

```js
const express = require('express');
const app = express();
app.get('/', (request, response) => {
  response.send('Hello Express!');
});
app.listen(3000, () => {
  console.log('Express app at http://localhost:3000');
}); 
```

运行此代码并访问 `localhost:3000`（假设你在本地主机上运行它），你将在浏览器中看到消息 **Hello Express!**。在你运行 `Node` 应用的终端中，它将在加载后打印控制台日志消息。

你可以在 Node.js 文档中找到更多信息，撰写本文时，该文档的地址如下：[`nodejs.org/en/docs/`](https://nodejs.org/en/docs/)。

对于 Express 模块，你可以访问 [`expressjs.com/en/5x/api.html`](https://expressjs.com/en/5x/api.html)。

# 下一步

在这本书和这一章中，你已经学到了很多关于 JavaScript 的知识，通过这一章，你应该对可能的下一步行动有一个大致的想法。这一章并没有深入教授所有这些主题，因为关于每个主题都可以（并且已经）写出整本书，但你应该有一个很好的方向去寻找你的下一步行动，并在决定采取哪一步时考虑什么。

最佳的学习方式是通过实践。所以我们强烈建议你提出一个有趣的项目想法，然后尝试去实现它。或者，有了这些知识，你可能觉得自己已经准备好进入 JavaScript 的初级职位！你还可以在线做教程，甚至作为一个初级成员加入项目团队，使用 Upwork 或 Fiverr 等自由职业平台来获取项目。这些项目很难找到，我们可以想象你将首先学习一个框架或获得一些 Node.js 的经验。然而，如果你能在招聘过程中展示你的技能和潜力，这通常在工作时是可能的。

# 章节项目

## 处理 JSON

在本地创建一个 JSON 文件，连接到 JSON 和数据，并将 JSON 文件中的数据输出到你的控制台：

1.  创建一个扩展名为 JSON 的文件，命名为`people.json`。

1.  在`people.json`中创建一个包含多个对象的数组。数组中的每个项目都应该是一个具有相同结构的对象，使用`first`、`last`和`topic`作为属性名。确保在属性名和值周围使用双引号，因为这才是正确的 JSON 语法。

1.  使用相同的对象结构为每个项目添加三个或更多条目。

1.  创建一个 HTML 文件并添加一个 JavaScript 文件。在 JavaScript 文件中，使用`people.json`作为 URL。使用`fetch`连接到 URL 并检索数据。由于这是一个 JSON 格式的文件，一旦获取到响应数据，就可以使用`fetch`中的`.json()`方法将其格式化为 JSON。

1.  将数据的全部内容输出到控制台。

1.  使用`foreach`循环遍历数据中的项目，并将值输出到控制台。您可以使用模板字符串并输出每个值。

## 列表制作项目

创建一个列表，将其保存到本地存储，这样即使刷新页面，数据也会在浏览器中持续存在。如果页面首次加载时本地存储为空，则设置一个 JSON 文件，将其加载到本地存储，并保存为默认列表以开始列表：

1.  设置一个 HTML 文件，添加一个`div`来输出列表结果，以及一个带有按钮的输入字段，可以点击。

1.  使用 JavaScript，将页面元素作为对象添加，以便在代码中使用。

1.  创建你的默认 JSON 文件（可以是空的），并使用名为`url`的变量将文件的路径添加到你的 JavaScript 代码中。

1.  为按钮元素添加一个事件监听器，该监听器将运行一个名为`addToList()`的函数。

1.  在`addToList()`中，检查输入字段的值长度是否为 3 或更多。如果是，则创建一个具有名称和输入字段值的对象。创建一个名为`myList`的全局变量来保存列表，并在`addToList()`中将新的对象数据推入`myList`。

1.  创建一个名为`maker()`的函数，该函数将创建页面元素并将文本添加到元素中，将其附加到输出元素。在`addToList()`函数中调用`maker()`以添加新项目。

1.  此外，将项目保存到本地存储，以便`myList`的视觉内容与本地存储中保存的值同步。为此，创建一个名为`savetoStorage()`的函数，并在每次更新脚本中的`myList`时调用它。

1.  在`savetoStorage()`函数中，使用`setItem`将`myList`的值设置到`localStorage`中。您需要将`myList`转换为字符串值以保存到`localStorage`中。

1.  向代码中添加`getItem()`以从`localStorage`检索`myList`的值。为`myList`数组设置一个全局变量。

1.  添加一个事件监听器来监听`DOMContentLoaded`。在函数中，检查是否从本地存储加载了值。如果是，则从本地存储获取`myList`并将其从字符串转换为 JavaScript 对象。清除输出元素的 内容。遍历`myList`中的项目，使用之前创建的`maker()`函数将它们添加到页面中。

1.  如果`localStorage`没有内容，使用`fetch`加载带有默认值的 JSON 文件。一旦数据加载，将其分配给全局的`myList`值。遍历`myList`中的项目，使用`maker()`函数将它们输出到页面。别忘了在之后调用`savetoStorage()`，这样存储将包含与页面上可见的相同列表项。

# 自我检查测验

1.  什么是 JavaScript 库和框架？

1.  如何判断一个网页是否使用了 jQuery 库？

1.  哪个库包含了大量用于操作数据的功能？

1.  当 Node.js 安装后，如何运行一个 Node.js 文件？

# 摘要

在本章中，我们探索了一些继续你的 JavaScript 之旅并不断提升自己的可能性。我们首先讨论了前端以及库和框架。库和框架都是预先编写的代码，你可以在项目中使用，但库通常解决一个问题，而框架提供了一种标准解决方案，通常控制你构建应用程序的方式，并可能带来一些限制。另一方面，框架非常适合你想要在 Web 应用程序中做的很多事情。

我们接着转向查看后端。后端是运行在服务器上的代码，当我们使用 Node.js 时，我们可以用 JavaScript 编写这段代码。Node.js 是一个可以处理 JavaScript 的运行时引擎，并且为 JavaScript 提供了一些额外的功能，这些功能在使用浏览器中的 JavaScript 时是没有的。

就这样。在这个阶段，你对 JavaScript 有了非常扎实的理解。你已经看到了所有主要的构建块，并在许多小练习和大型项目中进行了大量实践。有几件事是肯定的：你永远不会完成作为 JavaScript 程序员的学业，而且随着你不断进步，你将不断地用你可以制作的东西来让自己感到惊讶。

不要忘记享受乐趣！
