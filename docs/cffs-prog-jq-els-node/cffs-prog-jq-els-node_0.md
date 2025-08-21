# 前言

JavaScript 是由 Brendan Eich 在 1995 年左右在网景工作时编写的一种古怪的小语言。它是第一种基于浏览器的脚本语言，当时只在网景导航器中运行，但最终它找到了自己的位置，进入了大多数其他的 Web 浏览器。当时，网页几乎完全由静态标记组成。JavaScript（最初名为 LiveScript）的出现是因为需要使页面动态化，并为浏览器开发人员带来完整脚本语言的功能。

语言的许多设计决策都是出于简单和易用的需要，尽管当时有些决策纯粹是出于网景的营销原因。选择“JavaScript”这个名字是为了将它与 Sun Microsystems 的 Java 联系起来，尽管事实上 Sun 与之无关，而且它在概念上与 Java 有很大不同。

除了一种方式，即大部分语法都是从 Java、C 和 C++中借鉴而来，以便让那些熟悉这些语言的程序员感到熟悉。但尽管看起来相似，实际上它在内部是一个非常不同的东西，并且具有与更奇特的语言（如 Self、Scheme 和 Smalltalk）相似的特征。其中包括动态类型、原型继承、一级函数和闭包。

因此，我们最终得到了一种看起来很像当时的一些主流语言，并且可以被迫以与它们非常不同的中心思想行事的语言。这导致它在很多年里被人们误解。很多程序员从未将它视为一种“严肃”的编程语言，因此在编写浏览器代码时，他们没有应用几十年来积累的许多最佳开发实践。

那些深入研究这门语言的人肯定会发现很多奇怪之处。Eich 本人承认，这门语言的原型大约在 10 天内完成，尽管他的成果令人印象深刻，但 JavaScript 并非没有（很多）缺陷。这些缺陷并没有真正有助于提升它的声誉。

尽管存在所有这些问题，JavaScript 仍然成为世界上最广泛使用的编程语言之一，这不仅仅是因为互联网的爆炸和 Web 浏览器的普及。在众多浏览器上的支持似乎是一件好事，但它也因为在语言和 DOM 的实现上的差异而造成了混乱。

大约在 2005 年，AJAX 这个术语被创造出来，用来描述一种 JavaScript 编程风格，这种风格是由浏览器中`XMLHTTPRequest`对象的引入所可能的。这意味着开发人员可以编写客户端代码，直接使用 HTTP 与服务器通信，并在不重新加载页面的情况下更新页面元素。这真的是语言历史上的一个转折点。突然之间，它被用于“严肃”的 Web 应用程序，并且人们开始以不同的方式看待这门语言。

2006 年，John Resig 向世界发布了 jQuery。它旨在简化客户端脚本编写、DOM 操作和 AJAX，以及抽象掉许多浏览器之间的不一致性。它成为了许多 JavaScript 程序员的必备工具。迄今为止，它在全球前 10,000 个网站中被使用了 55%。

2009 年，Ryan Dahl 创建了 Node.js，这是一个基于 Google V8 JavaScript 引擎编写的事件驱动网络应用程序框架。它迅速变得非常流行，特别是用于编写 Web 服务器应用程序。它成功的一个重要因素是，现在你可以在服务器上编写 JavaScript，而不仅仅是在浏览器中。围绕这个框架形成了一个复杂而杰出的社区，目前 Node.js 的未来看起来非常光明。

2010 年初，Jeremy Ashkenas 创建了 CoffeeScript，这是一种编译成 JavaScript 的语言。它的目标是创建更清洁、更简洁、更惯用的 JavaScript，并使其更容易使用语言的更好特性和模式。它摒弃了 JavaScript 的许多语法冗长，减少了行噪音，通常创建了更短更清晰的代码。

受到 Ruby、Python 和 Haskell 等语言的影响，它借用了这些语言的一些强大和有趣的特性。尽管它看起来可能相当不同，但 CoffeeScript 代码通常与生成的 JavaScript 非常接近。它已经成为一夜成功，很快被 Node.js 社区采纳，并被包含在 Ruby on Rails 3.1 中。

Brendan Eich 也表达了对 CoffeeScript 的钦佩，并将其用作他希望在未来版本的 JavaScript 中看到的一些东西的例子。

本书既是对该语言的介绍，也是为什么您应该在尽可能的地方使用 CoffeeScript 而不是 JavaScript 的动机。它还探讨了在浏览器中使用 CoffeeScript 使用 jQuery 和 Ruby on Rails，以及在服务器上使用 Node.js。

# 本书涵盖的内容

第一章，为什么使用 CoffeeScript，介绍了 CoffeeScript 并深入探讨了它与 JavaScript 之间的区别，特别关注 CoffeeScript 旨在改进的 JavaScript 部分。

第二章，运行 CoffeeScript，简要介绍了 CoffeeScript 堆栈以及它通常是如何打包的。您将学习如何在 Windows、Mac 和 Linux 上使用 Node.js 和 npm 安装 CoffeeScript。您将了解 CoffeeScript 编译器（`coffee`）以及熟悉一些有用的工具和日常开发资源。

第三章，CoffeeScript 和 jQuery，介绍了使用 jQuery 和 CoffeeScript 进行客户端开发。我们还开始使用这些技术来实现本书的示例应用程序。

第四章，CoffeeScript 和 Rails，首先简要概述了 Ruby on Rails 及其与 JavaScript 框架的历史。我们介绍了 Rails 3.1 中的 Asset Pipeline 以及它如何与 CoffeeScript 和 jQuery 集成。然后我们使用 Rails 为我们的示例应用程序添加后端。

第五章，CoffeeScript 和 Node.js，首先简要概述了 Node.js，它的历史和哲学。然后演示了使用 Node.js 在 CoffeeScript 中编写服务器端代码有多么容易。然后我们使用 WebSockets 和 Node.js 实现了示例应用程序的最后一部分。

# 你需要什么来阅读本书

要使用本书，您需要一台运行 Windows、Mac OS X 或 Linux 的计算机和一个基本的文本编辑器。在整本书中，我们将从互联网上下载一些我们需要的软件，所有这些软件都将是免费和开源的。

# 这本书适合谁

这本书适合现有的 JavaScript 程序员，他们想了解更多关于 CoffeeScript 的知识，或者有一些编程经验并想了解更多关于使用 CoffeeScript 进行 Web 开发。它还是 jQuery、Ruby on Rails 和 Node.js 的绝佳入门书籍。即使您有这些框架中的一个或多个的经验，本书也会向您展示如何使用 CoffeeScript 使您的体验变得更好。

# 约定

在本书中，您会发现一些区分不同信息类型的文本样式。以下是一些这些样式的示例，以及它们的含义解释。

文本中的代码词显示如下：“您会发现`if`语句的子句不需要用括号括起来”。

代码块设置如下：

```js
gpaScoreAverage = (scores...) ->
   total = scores.reduce (a, b) -> a + b
   total / scores.length 
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目会以粗体显示：

```js
create: (e) ->
    $input = $(event.target)
    val = ($.trim $input.val())
```

任何命令行输入或输出都以以下方式书写：

```js
coffee -co public/js -w src/

```

**新术语**和**重要单词**以粗体显示。您在屏幕上看到的单词，比如菜单或对话框中的单词，会在文本中以这种方式出现："页脚将有**清除已完成**按钮"。

### 注意

警告或重要提示会以这样的框出现。

### 提示

提示和技巧会以这种方式出现。
