# 前言

在学习 Python 时，您通过学习 Python 的基础知识、其优雅和编程原则，迈出了软件工程职业生涯的第一步。在您职业生涯的下一个阶段，让我们学习如何将您的编程知识转移到 JavaScript 上，以处理前端任务，包括 UX/UI 工作、表单验证、前端动画等。您可能熟悉使用 Flask 渲染前端，但 JavaScript 将使您能够实时创建用户界面并对用户输入做出反应。

我们将深入探讨两种语言之间的差异，不仅在语法层面上，还在语义层面上：*为什么*和*何时*我们使用 JavaScript 而不是 Python，它们的关注点分离是什么，如何使用 Node.js 在前端和后端连接我们现有的 HTML 和 CSS，以创建引人入胜的用户体验，以及如何利用 Web 应用程序的所有层创建全栈应用程序。

# 本书的受众

在软件工程中，一刀切并不适用。Python 是一种适用、可扩展的语言，专为后端 Web 工作设计，但也可以引发对前端的好奇。本书是为具有 1-3 年 Python 经验的程序员编写的，他们希望扩展对前端编程世界的了解，该世界由 JavaScript 实现，并了解如何在前端和后端都使用 JavaScript（通过 Node.js）可以实现高效的编码和工作流程。

对数据类型、函数和作用域的扎实理解对于掌握本书中阐述的概念至关重要。熟悉 HTML、CSS、文档对象模型（DOM）以及 Flask 和/或 Django 将会很有帮助。

# 本书涵盖的内容

第一章，“JavaScript 进入主流编程”，我们将了解 JavaScript 的重要性。

第二章，“我们可以在服务器端使用 JavaScript 吗？当然可以！”，深入探讨了服务器端 JavaScript。JavaScript 的使用不仅限于浏览器端，还可以用于丰富、复杂的基于服务器的应用程序。

第三章，“细枝末节的语法”，是您将学习如何编写 JavaScript 以及它的语法与 Python 的不同之处的细节。

第四章，“数据及其朋友 JSON”，涵盖了数据。每个计算机程序都必须处理某种数据。您将学习如何在 JavaScript 中与数据交互。

第五章，“Hello World！及更多：您的第一个应用程序”，让您编写您的第一个 JavaScript 程序！

第六章，“文档对象模型（DOM）”，教会您如何使用网页的基础知识，以便将 JavaScript 与用户交互连接起来。

第七章，“事件、事件驱动设计和 API”，将带您超越基本交互，向您展示如何将动态数据纳入您的程序中。

第八章，“使用框架和库”，介绍了一些现代 JavaScript 程序的支架，以扩展您对行业标准应用的了解。

第九章，“解读错误消息和性能泄漏”，涵盖了错误。错误是难免的！我们应该了解一些如何处理它们并调试我们的程序的知识。

第十章，“JavaScript，前端的统治者”，更详细地介绍了 JavaScript 如何将前端整合在一起。

第十一章，“什么是 Node.js？”，深入探讨了 Node.js。由于已经研究了 JavaScript 在前端的使用，本章将探讨它在“JavaScript 无处不在”范式中使用 Node.js 的角色。

第十二章，*Node.js 与 Python 对比*，问，为什么开发人员选择 Node.js 而不是 Python？它们可以一起工作吗？我们如何安装我们需要创建和运行程序的软件包？

第十三章，*使用 Express*，介绍了 Express.js（或只是 Express），这是一个 Web 应用程序框架，被认为是 Node.js 的事实标准 Web 服务器。

第十四章，*使用 Django 的 React*，探索 Django。您可能已经将 Django 作为 Python 框架，让我们看看它与前端和后端的 JavaScript 框架有何不同。

第十五章，*将 Node.js 与前端结合*，将前端和后端连接在一起。我们将为（几乎）全栈功能构建两个小型应用程序。

第十六章，*进入 Webpack*，涉及部署工具，这对于高效的 JavaScript 至关重要。

第十七章，*安全和密钥*，深入探讨安全性。JavaScript 需要了解安全资源，那么我们该如何处理呢？

第十八章，*Node.js 和 MongoDB*，转向 MongoDB。MongoDB 是如何与 JavaScript 一起使用数据库的一个很好的例子。我们将使用它作为我们的示例 NoSQL 数据库，因为它与 JSON 数据很好地配合。

第十九章，*将所有内容放在一起*，让您使用完整的现代 JavaScript 堆栈创建最终项目。

# 要充分利用本书

由于我们将首先使用 JavaScript，因此您需要在计算机上安装代码编辑器，如 Visual Studio Code，Sublime Text 或其他通用编程环境。由于编码环境的限制，平板电脑等移动设备可能不是合适的环境，但较低配置的计算机可以使用。我们将使用命令行工具，因此熟悉 macOS 终端将会很有用；Windows 操作系统用户应下载并安装 Git Bash 或类似的终端程序，因为标准的 Windows 命令提示符将不够。

需要使用现代浏览器来使用我们的程序。推荐使用 Chrome。我们将在整个 JavaScript 工作中使用 ECMAScript 2015（也称为 ES6）。

我们将安装系统的各种其他组件，如 Node.js 和 Node Package Manager，Angular 和 React。每个必需组件的安装说明将在章节中提供。可能需要管理员权限才能完成所有安装步骤。

**如果您使用的是本书的数字版本，我们建议您自己输入代码或通过 GitHub 存储库（链接在下一节中提供）访问代码。这样做将有助于避免与复制和粘贴代码相关的任何潜在错误。**

我们的一些项目还需要访问网站，因此需要一个活跃的互联网连接。也建议具有一点幽默感。

## 下载示例代码文件

您可以从[www.packt.com](http://www.packt.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了本书，可以访问[www.packtpub.com/support](https://www.packtpub.com/support)注册，将文件直接发送到您的邮箱。

您可以按照以下步骤下载代码文件：

1.  在[www.packt.com](http://www.packt.com)上登录或注册。

1.  选择“支持”选项卡。

1.  点击“代码下载”。

1.  在搜索框中输入书名，然后按照屏幕上的说明操作。

下载文件后，请确保使用以下最新版本解压或提取文件夹：

+   Windows 使用 WinRAR/7-Zip

+   Mac 使用 Zipeg/iZip/UnRarX

+   Linux 使用 7-Zip/PeaZip

该书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Hands-on-JavaScript-for-Python-Developers`](https://github.com/PacktPublishing/Hands-on-JavaScript-for-Python-Developers)。如果代码有更新，将在现有的 GitHub 存储库上进行更新。

我们还有其他代码包，来自我们丰富的图书和视频目录，可在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**上找到。去看看吧！

## 下载彩色图片

我们还提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图片。您可以在这里下载：[`static.packt-cdn.com/downloads/9781838648121_ColorImages.pdf`](https://static.packt-cdn.com/downloads/9781838648121_ColorImages.pdf)。

## 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄。这里有一个例子：“将下载的`WebStorm-10*.dmg`磁盘映像文件挂载为系统中的另一个磁盘。”

代码块设置如下：

```js
html, body, #map {
 height: 100%; 
 margin: 0;
 padding: 0
}
```

当我们希望引起您对代码块的特别关注时，相关的行或项目会以粗体显示：

```js
[default]
exten => s,1,Dial(Zap/1|30)
exten => s,2,Voicemail(u100)
exten => s,102,Voicemail(b100)
exten => i,1,Voicemail(s0)
```

任何命令行输入或输出都是这样写的：

```js
$ mkdir css
$ cd css
```

**粗体**：表示一个新术语、一个重要词或屏幕上看到的词。例如，菜单或对话框中的单词会在文本中出现。这里有一个例子：“从管理面板中选择系统信息。”

警告或重要说明会出现在这样的地方。提示和技巧会出现在这样的地方。
