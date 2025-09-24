# 前言

JavaScript 是由 Brendan Eich 在 1995 年左右在 Netscape 工作时编写的一种古怪的小语言。它是第一种基于浏览器的脚本语言，当时只能在 Netscape Navigator 中运行，但它最终进入了大多数其他 Web 浏览器。当时，网页几乎完全由静态标记组成。JavaScript（最初命名为 LiveScript）的出现是为了满足使页面动态化以及将完整脚本语言的力量带给浏览器开发者的需求。

语言的设计决策很大程度上是由对简单性和易用性的需求驱动的，尽管在当时，其中一些决策是出于 Netscape 的纯粹营销原因。选择“JavaScript”这个名字是为了将其与 Sun 微系统公司的 Java 联系起来，尽管 Sun 实际上与此无关，而且它在概念上与其名称相似之处甚少。

除了一个方面，即它的语法大部分是从 Java、C 和 C++ 中借用的，以便让来自这些语言的程序员感到熟悉。但尽管看起来相似，实际上它是一个完全不同的动物，并且与更奇特的语言（如 Self、Scheme 和 Smalltalk）具有相似的特征。其中包括动态类型、原型继承、一等函数和闭包。

因此，我们最终得到了一种看起来与当时的主流语言非常相似的语言，并且可以被诱导出类似的行为，但它的核心思想却截然不同。这导致它在很多年里被严重误解。许多程序员从未将其视为一种“严肃”的编程语言，因此在编写浏览器代码时，没有应用几十年来积累的许多最佳开发实践。

那些深入研究这种语言的人肯定会发现很多奇怪之处。Eich 本身也承认，这种语言大约在 10 天内就完成了原型设计，尽管他所创造出的东西令人印象深刻，但 JavaScript 并非没有（许多）缺陷。这些缺陷也并没有真正帮助提升其知名度。

尽管存在所有这些问题，JavaScript 仍然成为了世界上使用最广泛的编程语言之一，如果不是仅仅因为互联网的爆炸式增长和 Web 浏览器的普及。跨众多浏览器的支持似乎是一件好事，但它也由于语言和 DOM 实现上的差异而造成了混乱。

大约在 2005 年，人们创造了 AJAX 这个术语来描述一种由浏览器中引入的 `XMLHTTPRequest` 对象所实现的 JavaScript 编程风格。这意味着开发者可以编写客户端代码，直接通过 HTTP 与服务器通信，并在不重新加载页面的情况下更新页面元素。这实际上标志着语言历史上的一个转折点。突然之间，它开始被用于“严肃”的 Web 应用程序中，人们开始以不同的眼光看待这种语言。

2006 年，约翰·雷斯吉（John Resig）将 jQuery 发布到世界。它旨在简化客户端脚本、DOM 操作和 AJAX，以及抽象化浏览器之间的许多不一致性。它成为许多 JavaScript 程序员的必备工具。到目前为止，它被世界上排名前 10,000 的网站中的 55%所使用。

2009 年，瑞安·达尔（Ryan Dahl）创建了 Node.js，这是一个基于 Google V8 JavaScript 引擎的事件驱动网络应用程序框架。它迅速变得非常受欢迎，尤其是在编写 Web 服务器应用程序方面。其成功的一个大因素是您现在可以在服务器上以及浏览器上编写 JavaScript。围绕这个框架出现了一个庞大而杰出的社区，目前 Node.js 的前景看起来非常光明。

2010 年初，杰里米·阿什肯纳斯（Jeremy Ashkenas）创建了 CoffeeScript，这是一种编译成 JavaScript 的语言。其目标是创建更干净、更简洁、更符合语言习惯的 JavaScript，并使其更容易使用语言更好的特性和模式。它去除了 JavaScript 中许多语法冗余，减少了行噪声，通常创建出更短、更清晰的代码。

受到 Ruby、Python 和 Haskell 等语言的影响，它借鉴了这些语言的一些强大而有趣的功能。尽管它看起来可能相当不同，但 CoffeeScript 代码通常与其生成的 JavaScript 非常接近。它迅速成为一夜之间的成功，很快被 Node.js 社区采用，并被包含在 Ruby on Rails 3.1 中。

布兰登·艾奇也对 CoffeeScript 表示了钦佩，并将其作为他希望在未来 JavaScript 版本中看到的一些事物的例子。

本书不仅介绍了这门语言，还探讨了为什么您应该在可能的情况下用 CoffeeScript 而不是 JavaScript 来编写代码。它还探讨了使用 jQuery 和 Ruby on Rails 在浏览器中使用 CoffeeScript，以及在服务器上使用 Node.js。

# 本书涵盖的内容

第一章, *为什么选择 CoffeeScript?*，介绍了 CoffeeScript 并深入探讨了它与 JavaScript 之间的区别，特别是 CoffeeScript 旨在改进的 JavaScript 部分。

第二章，*运行 CoffeeScript*，简要介绍了 CoffeeScript 的堆栈及其通常的打包方式。您将学习如何使用 Node.js 和 npm 在 Windows、Mac 和 Linux 上安装 CoffeeScript。您还将了解 CoffeeScript 编译器（`coffee`），以及熟悉一些有助于日常 CoffeeScript 开发的工具和资源。

第三章，*CoffeeScript 和 jQuery*，介绍了使用 jQuery 和 CoffeeScript 进行客户端开发。我们还开始使用这些技术实现本书的示例应用程序。

第四章，*CoffeeScript 和 Rails*，首先简要概述了 Ruby on Rails 及其与 JavaScript 框架的历史。我们介绍了 Rails 3.1 中的 Asset Pipeline 以及它是如何与 CoffeeScript 和 jQuery 集成的。然后我们转向使用 Rails 为我们的示例应用添加后端。

第五章，*CoffeeScript 和 Node.js*，首先简要概述了 Node.js、其历史和哲学。然后演示了使用 Node.js 用 CoffeeScript 编写服务器端代码是多么容易。然后我们使用 WebSockets 和 Node.js 实现了示例应用的最后一部分。

# 您需要这本书的内容

要使用这本书，您需要一个运行 Windows、Mac OS X 或 Linux 的计算机和一个基本的文本编辑器。在整个书中，我们将从互联网下载一些我们需要的软件，所有这些软件都将免费且为开源。

# 这本书面向的对象

这本书是为希望了解更多关于 CoffeeScript 的现有 JavaScript 程序员而写的，或者是有一些编程经验并希望学习更多使用 CoffeeScript 进行 Web 开发的人。它还作为 jQuery、Ruby on Rails 和 Node.js 的绝佳入门书籍。即使您有使用一个或多个这些框架的经验，这本书也会向您展示如何使用 CoffeeScript 使您的体验变得更好。

# 习惯用法

在这本书中，您会发现多种文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码词如下所示："您会看到`if`语句的子句不需要被括号括起来"。

代码块如下设置：

```js
gpaScoreAverage = (scores...) ->
   total = scores.reduce (a, b) -> a + b
   total / scores.length 
```

当我们希望引起您对代码块中特定部分的注意时，相关的行或项目将以粗体显示：

```js
create: (e) ->
    $input = $(event.target)
    val = ($.trim $input.val())
```

任何命令行输入或输出都如下所示：

```js
coffee -co public/js -w src/

```

**新术语**和**重要词汇**以粗体显示。您在屏幕上看到的单词，例如在菜单或对话框中，在文本中如下所示："页脚将有一个**清除已完成**按钮"。

### 注意

警告或重要注意事项如下所示。

### 小贴士

小技巧和技巧如下所示。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们您对这本书的看法——您喜欢什么或可能不喜欢什么。读者的反馈对我们开发您真正能从中获得最大收益的标题非常重要。

要向我们发送一般反馈，只需发送一封电子邮件到`<feedback@packtpub.com>`，并在邮件的主题中提及书名。

如果您在某个主题上具有专业知识，并且您有兴趣撰写或为书籍做出贡献，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 书籍的骄傲拥有者，我们有多个方面可以帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从您在 [`www.PacktPub.com`](http://www.PacktPub.com) 的账户下载您购买的任何 Packt 书籍的示例代码文件。如果您在其他地方购买了这本书，您可以访问 [`www.PacktPub.com/support`](http://www.PacktPub.com/support) 并注册，以便将文件直接通过电子邮件发送给您。

## 勘误

尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问 [`www.packtpub.com/support`](http://www.packtpub.com/support)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站，或添加到该标题的勘误部分下的现有勘误列表中。您可以通过从 [`www.packtpub.com/support`](http://www.packtpub.com/support) 选择您的标题来查看任何现有勘误。

## 盗版

互联网上对版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现我们作品的任何非法副本，无论形式如何，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过 `<copyright@packtpub.com>` 联系我们，并提供涉嫌盗版材料的链接。

我们感谢您在保护我们的作者以及为我们提供有价值内容的能力方面提供的帮助。

## 问题

如果您在本书的任何方面遇到问题，可以通过 `<questions@packtpub.com>` 联系我们，我们将尽力解决。
