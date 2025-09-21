# 前言

JavaScript 是网络开发和服务器端编程的一个组成部分。理解 JavaScript 的基础知识不仅能帮助人们创建交互式网络应用程序，还能帮助设置网络服务器，通过 React Native 等框架创建移动应用程序，甚至使用 electronJS 等框架创建桌面应用程序。

本书以 ECMAScript 2017（ES8）的形式介绍了 JavaScript 的新鲜和核心概念，其中包括您开始使用 JavaScript 并具备从基础到高级理解所需的一切，以便您能够实现第一段中提到的所有内容。

# 本书面向对象

本书适合对 JavaScript 一无所知且愿意学习这项技术的人。本书也可供熟悉旧版 JavaScript 并希望将知识提升到最新标准和使用技术的人使用。对于更高级的用户，本书可用于复习模块化、Web Workers 和共享内存等概念。

# 本书涵盖内容

第一章，*开始使用 ECMAScript*，讨论了 ECMAScript 到底是什么以及为什么我们称之为 ECMAScript 而不是 JavaScript。它还讨论了如何创建变量、执行基本操作，并在 ES8 中提供执行这些操作的新方法。

第二章，*了解您的库*，展示了作为初学者和中级 JavaScript 开发者需要了解的所有函数，以便在各种项目中顺利工作。本章教授的函数是通用和泛型函数，您将能够在需要的地方理解和应用它们。

第三章，*使用迭代器*，介绍了如何在 JavaScript 中以正确的方式遍历可迭代事物。我们讨论了 Symbol，这是一种新的原生 JavaScript 类型，它是什么，以及为什么我们需要它。我们还讨论了尾调用优化技术，这是浏览器为了加快代码执行而实现的。

第四章，*异步编程*，探讨了实现异步编程的现代方法，并将其与不那么美好的过去方法进行了比较，包括回调地狱。它将教会您以同步方式实现异步编程。

第五章，*模块化编程*，讨论了如何将您的 JavaScript 代码模块化到不同的文件中，以便于重用和调试单个模块。我们首先介绍早期可用的基本和第三方解决方案，然后介绍浏览器为世界带来的原生支持。

第六章，*实现 Reflect API*，展示了 JavaScript 中提供的 Reflect API 的信息，它基本上有助于操作对象的属性和方法。

第七章，*代理*，介绍了 JavaScript 中的一种新实现，即对象上的代理。它具有许多优点，例如隐藏私有属性、为对象属性和方法设置默认值，以及创建令人惊叹的自定义功能。例如，JavaScript 的类似 Python 的数组切片。

第八章，*类*，探讨了类、它们的实现、类中的继承以及类最终只是函数实现上的语法糖。这很重要，因为类使得代码对有面向对象编程背景的人来说更易于阅读和理解。

第九章，*网页上的 JavaScript*，探讨了在网站上使用 JavaScript 的基础知识，浏览器在网页上向开发者公开的一些流行 API，以及 JavaScript 如何用于与 DOM 交互以操纵网页上的内容。

第十章，*JavaScript 中的存储 API*，探讨了浏览器中可用的存储 API，并展示了如何利用它们在用户的计算机上本地存储数据。

第十一章，*Web 和 Service Workers*，讨论了 HTML5 中可用的 Web Workers，渐进式 Web 应用的服务 Workers，并展示了如何有效地使用这些 Workers 来分配任务负载。

第十二章，*共享内存和原子操作*，教我们如何利用 Web Workers 提供的多线程环境，通过`SharedArrayBuffer`允许 Web Workers 通过共享内存实现快速访问内存。它涵盖了与线程共享相同数据的一些常见问题，并提供了这些问题的解决方案。

# 要充分利用这本书

虽然这不是严格要求的，但如果你对 HTML/CSS 有些了解，并且对如何使用它创建基本的网页有些了解，那就太好了。

您应该熟悉现代浏览器（首选 Chrome 或 Firefox）在桌面/笔记本电脑上。

要充分利用这本书，不要仅仅阅读书籍；保持其他学习资源开放，并尽可能多地实现演示和示例代码。

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的账户下载这本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)登录或注册。

1.  选择 SUPPORT 标签。

1.  点击代码下载与勘误。

1.  在搜索框中输入书名，并遵循屏幕上的说明。

下载文件后，请确保使用最新版本的以下软件解压缩或提取文件夹：

+   WinRAR/7-Zip for Windows

+   适用于 Mac 的 Zipeg/iZip/UnRarX

+   适用于 Linux 的 7-Zip/PeaZip

书籍的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Learn-ECMAScript-Second-Edition`](https://github.com/PacktPublishing/Learn-ECMAScript-Second-Edition)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有其他来自我们丰富的图书和视频目录的代码包，可在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**上找到。查看它们吧！

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个例子：“仔细看看，我们并没有反复执行`counter()`。”

代码块设置如下：

```js
let myCounter = counter(); // returns a function (with count = 1)
myCounter(); // now returns 2
myCounter(); // now returns 3
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```js
var ob1 = {
   prop1 : 1,
    prop2 : {
        prop2_1 : 2 
    }
};
Object.freeze( ob1 );
```

**粗体**：表示新术语、重要词汇或屏幕上看到的词汇。例如，菜单或对话框中的文字会像这样显示。以下是一个例子：“然而，您可以指定移动的距离，也就是说`history.go(5);`与用户在浏览器中点击前进按钮五次是等效的。”

警告或重要提示看起来是这样的。

小贴士和技巧看起来是这样的。

# 联系我们

我们欢迎读者的反馈。

**一般反馈**：请通过电子邮件`feedback@packtpub.com`发送反馈，并在邮件主题中提及书名。如果您对本书的任何方面有疑问，请通过`questions@packtpub.com`发送电子邮件给我们。

**勘误表**：尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告，我们将不胜感激。请访问[www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误表提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上以任何形式遇到我们作品的非法副本，我们将不胜感激，如果您能提供位置地址或网站名称，我们将不胜感激。请通过版权邮箱 copyright@packtpub.com 与我们联系，并提供材料的链接。

**如果您有兴趣成为作者**：如果您在某个主题上具有专业知识，并且您有兴趣撰写或为本书做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。一旦您阅读并使用了这本书，为什么不在您购买它的网站上留下评论呢？潜在读者可以查看并使用您的客观意见来做出购买决定，Packt 可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

如需了解 Packt 的更多信息，请访问[packtpub.com](https://www.packtpub.com/)。
