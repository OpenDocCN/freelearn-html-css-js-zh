# 前言

欢迎阅读《学习 Node.js 开发》。本书充满了大量的内容、项目、挑战和真实世界的例子，所有这些都旨在通过实践教授 Node。这意味着在接下来的章节中，您将很早就开始动手写一些代码，并且您将为每个项目编写代码。您将编写支持我们应用程序的每一行代码。现在，我们需要一个文本编辑器。我们有各种文本编辑器选项可供选择。我始终建议使用 Atom，您可以在[atom.io](http://atom.io)找到它。它是免费的、开源的，并且适用于所有操作系统，即 Linux、macOS 和 Windows。它是由 GitHub 背后的人员创建的。

本书中的所有项目都很有趣，并且它们旨在教会您启动自己的 Node 应用程序所需的一切，从规划到开发、测试到部署。现在，当您启动这些不同的 Node 应用程序并阅读本书时，您将遇到错误，这是不可避免的。也许某些东西没有按预期安装，或者您尝试运行一个应用程序，而不是得到预期的输出，您得到了一个非常长的晦涩的错误消息。不要担心，我会在章节中向您展示通过这些错误的技巧和窍门。让我们继续并开始吧。

# 本书适合对象

本书面向希望启动自己的 Node 应用程序、转行或作为 Node 开发人员自由职业的任何人。您应该对 JavaScript 有基本的了解才能跟上本书的内容。

# 本书涵盖的内容

第一章《设置》，讨论了 Node 是什么以及为什么要使用它。在本章中，您将学习 Node 的安装，到本章结束时，您将能够运行您的第一个 Node 应用程序。

第二章《Node 基础知识-第一部分》讨论了构建 Node 应用程序。《Node 基础知识》主题已分为 3 部分。本主题的第一部分包括模块基础知识、需要自己的文件以及第三方 NPM 模块。

第三章《Node 基础知识-第二部分》继续讨论一些更多的 Node 基础知识。本章探讨了 yargs、JSON、addNote 函数和重构，将功能移入单独的函数并测试功能。

第四章《Node 基础知识-第三部分》包括从文件系统中读取和写入内容等内容。我们将深入研究高级 yargs 配置、调试故障应用程序以及一些新的 ES6 函数。

第五章《Node.js 异步编程基础》涵盖了与异步编程相关的基本概念、术语和技术，使其在我们的天气应用程序中变得非常实用。

第六章《异步编程中的回调》是 Node 中异步编程的第二部分。我们将研究回调、HTTPS 请求以及在回调函数中的错误处理。我们还将研究天气预报 API，并获取我们地址的实时天气数据。

第七章《异步编程中的 Promise》是 Node 中异步编程的第三部分，也是最后一部分。本章重点介绍 Promise，它的工作原理，为什么它们有用等等。在本章结束时，我们将在我们的天气应用程序中使用 Promise。

第八章《Node 中的 Web 服务器》讨论了 Node Web 服务器以及将版本控制集成到 Node 应用程序中。我们还将介绍一个名为 Express 的框架，这是最重要的 NPM 库之一。

第九章，*将应用部署到 Web*，讨论了将应用部署到 Web。我们将使用 Git、GitHub，并使用这两项服务将我们的实时应用程序部署到 Web。

第十章，*测试 Node 应用程序-第一部分*，讨论了我们如何测试代码以确保其按预期工作。我们将开始设置测试，然后编写我们的测试用例。我们将研究基本的测试框架和异步测试。

第十一章，*测试 Node 应用程序-第二部分*，继续我们测试 Node 应用程序的旅程。在本章中，我们将测试 Express 应用程序，并研究一些高级的测试方法。

# 为了充分利用本书

Web 浏览器，我们将在整本书中使用 Chrome，但任何浏览器都可以，以及终端，有时在 Linux 上称为命令行，Windows 上称为命令提示符。Atom 作为文本编辑器。以下模块列表将在本书的整个过程中使用：

+   lodash

+   nodemon

+   yargs

+   请求

+   axios

+   express

+   hbs

+   heroku

+   rewire

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了这本书，可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便文件直接通过电子邮件发送给您。

您可以按照以下步骤下载代码文件：

1.  登录或注册[www.packtpub.com](http://www.packtpub.com/support)。

1.  选择“支持”选项卡。

1.  单击“代码下载和勘误”。

1.  在搜索框中输入书名，然后按照屏幕上的说明操作。

文件下载后，请确保使用最新版本的以下软件解压或提取文件夹：

+   WinRAR/7-Zip 适用于 Windows

+   Zipeg/iZip/UnRarX 适用于 Mac

+   7-Zip/PeaZip 适用于 Linux

本书的代码捆绑包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Learning-Node.js-Development`](https://github.com/PacktPublishing/Learning-Node.js-Development)。我们还有来自丰富书籍和视频目录的其他代码捆绑包可用于**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。去看看吧！

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄。例如："将下载的`WebStorm-10*.dmg`磁盘映像文件挂载为系统中的另一个磁盘。"

代码块设置如下：

```js
console.log('Starting app.js');

const fs = require('fs');
const _ = require('lodash');
const yargs = require('yargs');
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目将以粗体显示：

```js
const argv = yargs.argv;
var command = process.argv[2];
console.log('Command:', command);
console.log('Process', process.argv); console.log('Yargs', argv);
```

任何命令行输入或输出都以以下方式编写：

```js
cd hello-world node app.js
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词会在文本中出现。例如："从管理面板中选择系统信息。"

警告或重要说明看起来像这样。

提示和技巧看起来像这样。
