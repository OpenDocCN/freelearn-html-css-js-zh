# 序言

似乎已经写下了所有需要关于 JavaScript 的东西。坦白说，要找到一个关于 JavaScript 还没有被详尽讨论的话题是困难的。然而，JavaScript 正在迅速变化。ECMAScript 6 有潜力改变这门语言以及我们用它编写的代码方式。Node.js 已经改变了我们用 JavaScript 编写服务器的方式。像 React 和 Flux 这样的新想法将推动语言的下一轮迭代。虽然我们花时间学习新特性，但不可否认的是，必须掌握 JavaScript 的基础理念。这些理念是基础且需要关注。如果你已经是一个有经验的 JavaScript 开发者，你会意识到现代 JavaScript 与大多数人所知的那门语言大相径庭。现代 JavaScript 要求特定的风格纪律和思维的严谨性。工具变得更加强大，并逐渐成为开发工作流程的一个重要组成部分。尽管语言似乎在变化，但它建立在一些非常坚实且恒定的理念之上。这本书强调的就是这些基本理念。

在撰写这本书的过程中，JavaScript 领域的很多事情都在不断变化。幸运的是，我们成功地在这本书中包括了所有重要的相关更新。

《精通 JavaScript》为你提供了对语言基础和一些现代工具和库（如 jQuery、Underscore.js 和 Jasmine）的详细概述。

我们希望你能像我们享受写作一样享受这本书。

# 本书内容概览

第一章，*JavaScript 入门*，专注于语言构造，而不花太多时间在基本细节上。我们将涵盖变量作用域和循环的更复杂部分以及使用类型和数据结构的最佳实践。我们还将涵盖大量的代码风格和推荐的代码组织模式。

第二章，*函数、闭包和模块*，涵盖了语言复杂性的核心。我们将讨论使用函数方面以及在 JavaScript 中对待闭包的不同处理方法的复杂性。这是一个谨慎且详尽的讨论，将为你进一步探索更高级的设计模式做好准备。

第三章，*数据结构及其操作*，详细介绍了正则表达式和数组。数组是 JavaScript 中的一个基本数据类型，本章将帮助你有效地使用数组。正则表达式可以使你的代码简洁—我们将详细介绍如何在你的代码中有效地使用正则表达式。

第四章《面向对象的 JavaScript》，讨论了 JavaScript 中的面向对象。我们将讨论继承和原型链，并专注于理解 JavaScript 提供的原型继承模型。我们还将讨论这个模型与其他面向对象模型的不同之处，以帮助 Java 或 C++ 程序员熟悉这种变化。

第五章《JavaScript 模式》，讨论了常见的设计模式以及如何在 JavaScript 中实现它们。一旦你掌握了 JavaScript 的面向对象模型，理解设计和编程模式就会更容易，写出模块化且易于维护的代码。

第六章《测试与调试》，涵盖了各种现代方法来测试和调试 JavaScript 代码中的问题。我们还将探讨 JavaScript 的持续测试和测试驱动方法。我们将使用 Jasmine 作为测试框架。

第七章《ECMAScript 6》，专注于由 ECMAScript 6 (ES6) 引入的新语言特性。它使 JavaScript 更加强大，本章将帮助你理解新特性以及如何在代码中使用它们。

第八章《DOM 操作与事件》，详细探讨了 JavaScript 作为浏览器语言的部分。本章讨论了 DOM 操作和浏览器事件。

第九章《服务器端 JavaScript》，解释了如何使用 Node.js 在 JavaScript 中编写可扩展的服务器系统。我们将讨论 Node.js 的架构和一些有用的技术。

# 本书你需要什么

本书中的所有示例都可以在任何现代浏览器上运行。对于最后一章，你需要 Node.js。为了运行本书中的示例和样本，你需要以下先决条件：

+   安装有 Windows 7 或更高版本、Linux 或 Mac OS X 的计算机。

+   最新版本的 Google Chrome 或 Mozilla Firefox 浏览器。

+   你选择的文本编辑器。Sublime Text、vi、Atom 或 Notepad++ 都是理想的选择。完全由你决定。

# 本书适合谁

本书旨在为你提供掌握 JavaScript 的必要细节。本书将对以下读者群体有用：

+   有经验的开发者，熟悉其他面向对象语言。本书的信息将使他们能够利用现有的经验转向 JavaScript。

+   有一定经验的 Web 开发者。这本书将帮助他们学习 JavaScript 的高级概念并完善他们的编程风格。

+   初学者想要理解并最终掌握 JavaScript。这本书为他们提供了开始所需的信息。

# 约定

在这本书中，您会发现有一些文本样式用于区分不同类型的信息。以下是一些这些样式的示例及其含义解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、假 URL、用户输入和 Twitter 处理方式如下所示："首先，`<head>`中的`<script>`标签导入了 JavaScript，而第二个`<script>`标签用于嵌入内联 JavaScript。"

代码块如下所示：

```js
function sayHello(what) {
  return "Hello " + what;
}
console.log(sayHello("world"));
```

当我们要引您注意代码块中的某个特定部分时，相关的行或项目会被加粗：

```js
<head>
 <script type="text/javascript" src="img/script.js"></script>
 <script type="text/javascript">
 var x = "Hello World";
 console.log(x);
 </script>
</head>
```

任何命令行输入或输出都如下所示：

```js
EN-VedA:~$ node
> 0.1+0.2
0.30000000000000004
> (0.1+0.2)===0.3
false

```

**新术语** 和 **重要词汇** 以粗体显示。例如，在菜单或对话框中看到的屏幕上的词，会在文本中这样显示："You can run the page and inspect using Chrome's **Developer Tool**"

### 注意

警告或重要说明以这样的盒子出现。

### 提示

技巧和建议以这样的形式出现。

# 读者反馈

读者对我们书籍的反馈总是受欢迎的。让我们知道您对这本书的看法——您喜欢或不喜欢的地方。读者反馈对我们很重要，因为它帮助我们开发出您会真正从中受益的标题。

要发送给我们一般性反馈，只需电子邮件 `<feedback@packtpub.com>`，并在您消息的主题中提到书的标题。

如果您在某个主题上有专业知识，并且有兴趣撰写或贡献一本书，请查看我们的作者指南：[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

既然您已经成为 Packt 图书的自豪拥有者，我们有很多事情可以帮助您充分利用您的购买。

## 下载示例代码

您可以从您在 [`www.packtpub.com`](http://www.packtpub.com) 的账户上下载本书中的示例代码文件，您购买的 Packt Publishing 所有的书籍都可以。如果您在其他地方购买了这本书，您可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 注册，以便将文件直接通过电子邮件发送给您。

## 下载本书彩色图片

我们还为您提供了一个 PDF 文件，其中包含本书中使用的屏幕快照/图表的彩色图片。这些彩色图片将帮助您更好地理解输出中的变化。您可以从 [`www.packtpub.com/sites/default/files/downloads/MasteringJavaScript_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/MasteringJavaScript_ColorImages.pdf) 下载这个文件。

## 勘误

虽然我们已经尽一切努力确保我们内容的准确性，但错误确实会发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——我们将非常感激如果您能向我们报告。通过这样做，您可以节省其他读者不必要的挫折，并帮助我们改进本书的后续版本。如果您发现任何错误，请通过访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**错误提交表单**链接，并输入您错误的详细信息。一旦您的错误得到验证，您的提交将被接受，错误将被上传到我们的网站，或添加到该标题的错误部分已有的错误列表中。

要查看之前提交的错误，请前往 [`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support) 并在搜索框中输入书籍名称。所需信息将在**错误**部分出现。

## 盗版

互联网上的版权材料盗版是一个持续存在的问题，涉及所有媒体。在 Packt，我们非常严肃地对待我们版权和许可的保护。如果您在互联网上以任何形式发现我们作品的非法副本，请立即提供给我们位置地址或网站名称，这样我们可以寻求一个补救措施。

如果您发现可疑的盗版材料，请联系我们 `<copyright@packtpub.com>`并提供链接。

我们感激您在保护我们的作者和我们提供有价值内容的能力方面所提供的帮助。

## 问题

如果您在这本书的任何方面遇到问题，您可以通过 `<questions@packtpub.com>` 联系我们，我们会尽最大努力解决问题。
