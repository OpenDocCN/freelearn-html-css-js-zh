# 前言

欢迎来到*高级 Node.js 开发*。本书充满了大量的内容、项目、挑战和真实世界的例子，所有这些都旨在通过*实践*来教您 Node。这意味着在接下来的章节中，您将很早就开始动手写一些代码，并且您将为每个项目编写代码。您将编写支持我们应用程序的每一行代码。现在，我们需要一个文本编辑器。

本书中的所有项目都很有趣，旨在教会您启动自己的 Node 应用程序所需的一切，从规划到开发，再到测试和部署。现在，当您启动这些不同的 Node 应用程序并在书中前进时，您将遇到错误，这是不可避免的。也许某些东西没有按预期安装，或者您尝试运行一个应用程序，而不是获得预期的输出，您得到了一个非常长的晦涩错误消息。别担心，我会帮助您。我将在各章节中向您展示通过这些错误的技巧和诀窍。让我们继续并开始吧。

# 这本书适合谁

本书面向任何希望启动自己的 Node 应用程序、转行或作为 Node 开发人员自由职业的人。您应该对 JavaScript 有基本的了解才能跟上本书。

# 本书涵盖内容

第一章，*设置*，将是您本地环境的非常基本的设置。我们将学习安装 MongoDB 和 Robomongo。

第二章，*MongoDB，Mongoose 和 REST API-第一部分*，将帮助您学习如何将您的 Node 应用程序连接到您在本地计算机上运行的 MongoDB 数据库。

第三章，*MongoDB，Mongoose 和 REST API-第二部分*，将帮助您开始使用 Mongoose 并连接到我们的 MongoDB 数据库。

第四章，*MongoDB，Mongoose 和 REST API-第三部分*，在与 Mongoose 玩耍后，将解决查询和 ID 验证问题。

第五章，*使用 Socket.io 创建实时 Web 应用程序*，将帮助您详细了解 Socket.io 和 WebSockets，帮助您创建实时 Web 应用程序。

第六章，*生成 newMessage 和 newLocationMessage*，讨论如何生成文本和地理位置消息。

第七章，*将我们的聊天页面样式化为 Web 应用程序*，继续我们关于样式化我们的聊天页面的讨论，并使其看起来更像一个真正的 Web 应用程序。

第八章，*加入页面和传递房间数据*，继续我们关于聊天页面的讨论，并研究加入页面和传递房间数据。

第九章，*ES7 类*，将帮助您学习 ES6 类语法，并使用它创建用户类和其他一些方法。

第十章，*Async/Await 项目设置*，将带您了解 async/await 的工作过程。

# 为了充分利用本书

要运行本书中的项目，您将需要以下内容：

+   Node.js 的最新版本（在撰写本书时为 9.x.x）

+   Express

+   MongoDB

+   Mongoose

+   Atom

我们将在书的过程中看到其他要求。

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便直接通过电子邮件接收文件。

您可以按照以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)登录或注册。

1.  选择 SUPPORT 选项卡。

1.  单击“代码下载和勘误”。

1.  在搜索框中输入书名，然后按照屏幕上的说明操作。

文件下载后，请确保使用最新版本的解压缩或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

该书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Advanced-Node.js-Development`](https://github.com/PacktPublishing/Advanced-Node.js-Development)。我们还有其他书籍和视频的代码包可供下载，网址为**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。快去看看吧！

# 下载彩色图片

我们还提供了一个 PDF 文件，其中包含了本书中使用的屏幕截图/图表的彩色图片。您可以在这里下载：[`www.packtpub.com/sites/default/files/downloads/AdvancedNode.jsDevelopment_ColorImages.pdf`](http://www.packtpub.com/sites/default/files/downloads/Bookname_ColorImages.pdf)。

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名。这里有一个例子：“将下载的`WebStorm-10*.dmg`磁盘映像文件挂载为系统中的另一个磁盘。”

代码块设置如下：

```js
html, body, #map {
 height: 100%; 
 margin: 0;
 padding: 0
}
```

当我们希望引起您对代码块的特定部分的注意时，相关的行或项目会以粗体显示：

```js
[default]
exten => s,1,Dial(Zap/1|30)
exten => s,2,Voicemail(u100)
exten => s,102,Voicemail(b100)
exten => i,1,Voicemail(s0)
```

任何命令行输入或输出都是这样写的：

```js
$ cd css
```

**粗体**：表示一个新术语，一个重要单词，或者您在屏幕上看到的单词。例如，菜单或对话框中的单词会以这种形式出现在文本中。这里有一个例子：“从管理面板中选择系统信息。”

警告或重要提示会显示在这样的形式下。

提示和技巧会显示在这样的形式下。
