# 第一章. 使用 Jasmine 入门

成为 JavaScript 开发人员是一个令人兴奋的时刻；技术已经成熟，Web 浏览器更加标准化，每天都有新的东西可以玩。JavaScript 已经成为一种成熟的语言，而 Web 是当今真正开放的平台。我们已经看到单页 Web 应用的兴起，**模型视图控制器**（**MVC**）框架的大量使用，如 Backbone.js 和 AngularJS，使用 Node.js 在服务器上使用 JavaScript，甚至使用诸如 PhoneGap 等技术完全使用 HTML、JavaScript 和 CSS 创建的移动应用程序。

从处理 HTML 表单的谦虚开始，到今天的大型应用程序，JavaScript 语言已经走了很远的路，随之而来的是一系列成熟的工具，以确保你在使用它时能够达到与其他语言相同的质量水平。

这本书是关于让你控制 JavaScript 开发的工具。

# JavaScript - 不好的部分

处理客户端 JavaScript 代码时会遇到许多复杂问题；显而易见的是，你无法控制客户端的运行时。在服务器上，你可以运行特定版本的 Node.js 服务器，但你无法强迫客户端运行最新版本的 Chrome 或 Firefox。

JavaScript 语言由 ECMAScript 规范定义；因此，每个浏览器都可以有自己的运行时实现，这意味着它们之间可能存在一些小的差异或错误。

此外，你还会遇到语言本身的问题。Brendan Eich 在 Netscape 受到很大的管理压力下，仅用 10 天时间开发了 JavaScript。尽管它在简洁性、一流函数和对象原型方面做得很好，但它也在试图使语言具有可塑性并允许其发展的过程中引入了一些问题。

每个 JavaScript 对象都是可变的；这意味着你无法阻止一个模块覆盖其他模块的部分。以下代码说明了覆盖全局`console.log`函数有多么简单：

```js
**console.log('test');**
**>> 'test'**
**console.log = 'break';**
**console.log('test');**
**>> TypeError: Property 'log' of object #<Console> is not a function**

```

这是语言设计上的一个有意识的决定；它允许开发人员对语言进行调整并添加缺失的功能。但是在拥有这样的权力的同时，很容易犯错。

ECMA 规范的第 5 版引入了`Object.seal`函数，一旦调用就可以防止对任何对象的进一步更改。但它目前的支持并不广泛；例如，Internet Explorer 只在其第 9 版上实现了它。

另一个问题是 JavaScript 处理类型的方式。在其他语言中，像`'1' + 1`这样的表达式可能会引发错误；在 JavaScript 中，由于一些不直观的类型强制转换规则，上述代码的结果是`'11'`。但主要问题在于它的不一致性；在乘法运算中，字符串被转换为数字，所以`'3' * 4`实际上是`12`。

这可能导致在大型表达式上出现一些难以发现的问题。假设你有一些来自服务器的数据，虽然你期望是数字，但一个值却是字符串：

```js
var a = 1, b = '2', c = 3, d = 4;
var result = a + b + c * d;
```

前面示例的结果值是`'1212'`，一个字符串。

这些只是开发人员面临的两个常见问题。在整本书中，你将应用最佳实践并编写测试，以确保你不会陷入这些和其他陷阱。

# Jasmine 和行为驱动开发

Jasmine 是由 Pivotal Labs 的开发人员创建的一个小型**行为驱动开发**（BDD）测试框架，允许你编写自动化的 JavaScript 单元测试。

但在我们继续之前，首先我们需要搞清楚一些基本知识，从测试单元开始。

测试单元是测试应用程序代码功能单元的一段代码。但有时，理解功能单元是什么可能会有些棘手，因此，为此，Dan North 提出了一种解决方案，即 BDD，这是对**测试驱动开发**（**TDD**）的重新思考。

在传统的单元测试实践中，开发人员在如何开始测试过程、要测试什么、测试的规模有多大，甚至如何调用测试等方面都没有明确的指导。

为了解决这些问题，丹从标准的敏捷构造中引入了**用户故事**的概念，作为编写测试的模型。

例如，音乐播放器应用程序可能有一个验收标准，如下所示：

**假设**有一个播放器，**当**歌曲被暂停时，**然后**它应该指示歌曲当前是暂停状态。

如下列表所示，这个验收标准是按照一个基本模式编写的：

+   **假设**：这提供了一个初始上下文

+   **当**：这定义了发生的事件

+   **然后**：这确保了一个结果

在 Jasmine 中，这转化为一种非常富有表现力的语言，允许以反映实际业务价值的方式编写测试。前面的验收标准写成 Jasmine 测试单元将如下所示：

```js
describe("Player", function() {
  describe("when song has been paused", function() {
    it("should indicate that the song is paused", function() {

    });
  });
});
```

你可以看到标准很好地转化为了 Jasmine 语法。在下一章中，我们将详细介绍这些函数的工作原理。

使用 Jasmine，与其他 BDD 框架一样，每个验收标准直接转化为一个测试单元。因此，每个测试单元通常被称为**规范**。在本书的过程中，我们将使用这个术语。

# 下载 Jasmine

开始使用 Jasmine 实际上非常简单。

打开 Jasmine 网站[`jasmine.github.io/2.1/introduction.html#section-Downloads`](http://jasmine.github.io/2.1/introduction.html#section-Downloads)，并下载**独立版本**（本书将使用 2.1.3 版本）。

在 Jasmine 网站上，您可能会注意到它实际上是一个执行其中包含的规范的实时页面。这是由于 Jasmine 框架的简单性所实现的，使其能够在最不同的环境中执行。

下载了分发并解压缩后，您可以在浏览器中打开`SpecRunner.html`文件。它将显示一个示例测试套件的结果（包括我们之前向您展示的验收标准）：

![下载 Jasmine](img/B04138_01_01.jpg)

这显示了在浏览器上打开的 SpecRunner.html 文件

这个`SpecRunner.html`文件是一个 Jasmine 浏览器规范运行器。这是一个简单的 HTML 文件，引用了 Jasmine 代码、源文件和测试文件。出于约定目的，我们将简称这个文件为**runner**。

你可以通过在文本编辑器中打开它来看到它有多简单。这是一个引用了 Jasmine 源代码的小型 HTML 文件：

```js
<script src="lib/jasmine-2.1.3/jasmine.js"></script>
<script src="lib/jasmine-2.1.3/jasmine-html.js"></script>
<script src="lib/jasmine-2.1.3/boot.js"></script>
```

runner 引用了源文件：

```js
<script type="text/javascript" src="src/Player.js"></script>
<script type="text/javascript" src="src/Song.js"></script>
```

runner 引用了一个特殊的`SpecHelper.js`文件，其中包含在规范之间共享的代码：

```js
<script type="text/javascript" src="spec/SpecHelper.js"></script>
```

runner 还引用了规范文件：

```js
<script type="text/javascript" src="spec/PlayerSpec.js"></script>
```

### 提示

**下载示例代码**

您可以从[`www.packtpub.com`](http://www.packtpub.com)的帐户中下载您购买的所有 Packt 图书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便直接通过电子邮件接收文件。

Jasmine 框架设置在`lib/jasmine-2.1.3/boot.js`文件中，虽然它是一个庞大的文件，但它的大部分内容都是关于设置实际发生的文档。建议您在文本编辑器中打开它并研究其内容。

尽管目前我们是在浏览器中运行规范，在第八章*构建自动化*中，我们将使相同的规范和代码在**无头浏览器**（如 PhantomJS）上运行，并将结果写入控制台。

无头浏览器是一个没有图形用户界面的浏览器环境。它可以是一个实际的浏览器环境，比如使用 WebKit 渲染引擎的 PhantomJS，也可以是一个模拟的浏览器环境，比如 Envjs。

虽然本书未涉及，但 Jasmine 也可以用于测试为诸如 Node.js 等环境编写的服务器端 JavaScript 代码。

这种 Jasmine 的灵活性令人惊叹，因为你可以使用同样的工具来测试各种类型的 JavaScript 代码。

# 总结

在本章中，你看到了测试 JavaScript 应用程序的动机之一。我向你展示了 JavaScript 语言的一些常见陷阱，以及 BDD 和 Jasmine 如何帮助你编写更好的测试。

你也看到了使用 Jasmine 进行下载和入门是多么简单。

在下一章中，你将学习如何以 BDD 的方式思考并编写你的第一个规范。
