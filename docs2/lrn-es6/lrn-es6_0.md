# 前言

ECMAScript 是由 Ecma International 在 ECMA-262 规范和 ISO/IEC 16262 中标准化的脚本语言。JavaScript、JScript 和 ActionScript 等脚本语言是 ECMAScript 的超集。尽管 JavaScript、JScript 和 ActionScript 比 ECMAScript 具有更多功能，通过定义更多的对象和方法，这些语言的核心特性与 ECMAScript 相同。

ECMAScript 6 是 ECMAScript 语言的第六版和第七版。简而言之，它也被称为“ES6”。

尽管 JavaScript 非常强大且灵活，但它经常因不必要的冗余而受到批评。因此，JavaScript 开发者经常使用像 CoffeeScript 和 Typescript 这样的抽象，它们提供更简单的语法、强大的功能，并编译成 JavaScript。ES6 的引入是为了改进 JavaScript，并确保开发者不再需要使用抽象或其他技术来编写高质量的代码，这已经成为一个漫长的过程。

ES6 的特性继承自其他流行的抽象语言，如 CoffeeScript。因此，ES6 语言特性在其他语言中的行为方式相同，即使在 JavaScript 中它们是新的，在编程世界中也不算新。

本书为 ECMAScript 6 新版本的所有特性提供了带示例的解释。本书是关于 ECMAScript 6 的 JavaScript 实现的。本书中的所有特性和示例都在所有 JavaScript 环境中工作，例如浏览器、Node.js、Cordova 等。

# 本书涵盖的内容

第一章, *玩转语法*，介绍了创建变量和函数参数的新方法。本章更深入地讨论了新的对象和函数语法。

第二章, *了解你的库*，介绍了现有对象的新基于原型的方法。

第三章, *使用迭代器*，展示了 ES6 中可用的不同类型的迭代器以及如何创建自定义迭代器。它还讨论了 ES6 中的尾调用优化。

第四章, *异步编程*，说明了 Promise 如何使编写异步执行代码更容易。

第五章, *实现 Reflect API*，提供了对 ES6 中对象反射的深入指南。

第六章, *使用代理*，展示了如何使用 ES6 代理定义对对象的基本操作的自定义行为。

第七章, *带你走进类*，介绍了使用 ES6 类进行面向对象编程。这里解释了诸如继承、构造函数、抽象、信息隐藏等概念。

第八章，*模块化编程*，解释了使用 JavaScript 创建模块的不同方法。涵盖了诸如 IIFE、CommonJS、AMD、UMD 和 ES6 模块等技术。

# 你需要这本书什么

如果你是在所有 JavaScript 引擎完全支持 ES6 之后阅读这本书，那么你不需要设置任何特定的测试环境。你可以在你选择的任何引擎上简单地测试示例。

如果你在所有 JavaScript 引擎完全支持 ES6 之前阅读这本书，那么请继续阅读这本书并执行可以使用 ES6 编译器转换的代码片段。如果你想在浏览器环境中运行代码示例，那么请使用这个示例网页模板，它已经附带了 Traceur 编译器，用于在每次页面加载时将 ES6 转换为 ES5：

```js
<!doctype html>
<html>
<head>...</head>
<body>
...

<script src="img/traceur.js"></script>
<script src="img/bootstrap.js"></script>
<script type="module">
            //Place ES6 code here
</script>
</body>
</html>
```

从[`google.github.io/traceur-compiler/bin/traceur.js`](https://google.github.io/traceur-compiler/bin/traceur.js)下载`traceur.js`脚本，从[`google.github.io/traceur-compiler/src/bootstrap.js`](https://google.github.io/traceur-compiler/src/bootstrap.js)下载`bootstrap.js`脚本。然后，将它们放在包含前面代码的 HTML 文件相同的目录下。

在练习文件（代码包）中，Traceur 编译器和填充程序已经附上。练习文件是为了在浏览器上测试代码示例而创建的。

对于第四章，*异步编程*，你将不得不使用浏览器环境进行测试，因为我们已经在示例中使用了 jQuery 和 AJAX。你还需要一个 Web 服务器来支持它。

对于第八章，*模块化编程*，如果你使用浏览器环境进行测试，那么你需要一个 Web 服务器。但如果你使用 Node.js 环境，那么你不需要 Web 服务器。

## 与 ECMAScript 6 的兼容性

这本书是在所有 JavaScript 引擎开始支持 ES6 的所有特性之前编写的。

ES6 的规范已经完成。只是并非所有 JavaScript 引擎都完成了 ES6 所有特性的实现。我相当确信到 2016 年底，所有 JavaScript 引擎都将支持 ES6。

Kangax 创建了一个 ES6 兼容性表格，你可以在这里跟踪各种 JavaScript 引擎对 ES6 各种特性的支持情况。你可以在这个表格中找到[`kangax.github.io/compat-table/es6/`](http://kangax.github.io/compat-table/es6/)。

## 在不兼容的引擎中运行 ECMAScript 6

如果你想在不支持 ES6 的引擎中运行 ES6，那么你可以使用 ES6 填充程序或 ES6 编译器。

Polyfill 是一段代码，它提供了开发者期望 JavaScript 引擎原生提供的功能。请记住，并非每个 ES6 功能都有 Polyfill，而且它们也不能被创建。所有可用的 Polyfill 及其下载链接可在 [`github.com/Modernizr/Modernizr/wiki/HTML5-Cross-Browser-Polyfills#ecmascript-6-harmony`](https://github.com/Modernizr/Modernizr/wiki/HTML5-Cross-Browser-Polyfills#ecmascript-6-harmony) 找到。

ES6 编译器将 ES6 源代码转换为 ES5 源代码，这与所有 JavaScript 引擎兼容。编译器支持比 Polyfill 转换更多功能，但可能不支持 ES6 的所有功能。有各种编译器可用，例如 Google Traceur ([`github.com/google/traceur-compiler`](https://github.com/google/traceur-compiler))、Google Caja ([`developers.google.com/caja/`](https://developers.google.com/caja/))、Babel ([`babeljs.io/`](https://babeljs.io/))、Termi ES6 Transpiler ([`github.com/termi/es6-transpiler`](https://github.com/termi/es6-transpiler)) 等。您应该在将 ES6 代码附加到网页之前始终将其转换为 ES5，而不是每次页面加载时在客户端进行转换，这样网页就不会加载得更慢。

因此，通过使用编译器和/或 Polyfill，您可以在所有引擎完全支持 ES6 以及非 ES6 引擎变得过时之前，就开始编写用于分发的 ES6 代码。

# 本书面向对象

本书面向任何熟悉 JavaScript 的人。您不必是 JavaScript 专家就能理解这本书。这本书将帮助您将 JavaScript 知识提升到下一个层次。

# 惯例

在本书中，您将找到多种文本样式，以区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 处理方式如下：“我们可以通过使用`include`指令来包含其他上下文。”

代码块设置如下：

```js
var a = 12; //accessible globally

function myFunction()
{
  console.log(a);

  var b = 13; //accessible throughout function

  if(true)
  {
    var c = 14; //accessible throughout function
    console.log(b);
  }

  console.log(c);
}

myFunction();
```

**新术语**和**重要词汇**以粗体显示。

### 注意

警告或重要注意事项以如下框的形式出现。

### 小贴士

小技巧和技巧如下所示。

# 读者反馈

我们欢迎读者的反馈。请告诉我们您对这本书的看法——您喜欢什么或可能不喜欢什么。读者反馈对我们开发您真正能从中受益的标题非常重要。

要向我们发送一般反馈，只需发送电子邮件到 `<feedback@packtpub.com>`，并在邮件主题中提及书名。

如果您在某个主题上具有专业知识，并且您有兴趣撰写或为书籍做出贡献，请参阅我们的作者指南 [www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 书籍的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)的账户下载您购买的所有 Packt 书籍的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 勘误

尽管我们已经尽一切努力确保我们内容的准确性，但错误仍然可能发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站，或添加到该标题的勘误部分下的现有勘误列表中。任何现有勘误都可以通过从[`www.packtpub.com/support`](http://www.packtpub.com/support)选择您的标题来查看。

## 盗版

互联网上对版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现任何我们作品的非法副本，无论形式如何，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过`<copyright@packtpub.com>`与我们联系，并提供疑似盗版材料的链接。

我们感谢您在保护我们作者和我们为您提供有价值内容的能力方面的帮助。

## 询问

如果您在本书的任何方面遇到问题，可以通过`<questions@packtpub.com>`联系我们，我们将尽力解决。
