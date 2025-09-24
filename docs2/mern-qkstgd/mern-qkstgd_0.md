# 前言

MERN 栈可以看作是一组工具的集合，这些工具共享一个共同的基础，即 JavaScript 语言。本书以食谱的形式探讨了如何遵循 MVC 架构模式使用 MERN 栈构建 Web 客户端和服务器应用程序。

MVC 架构模式中的模型和控制器在关于使用 ExpressJS 和 Mongoose 构建 RESTful API 的章节中进行了介绍。这些章节涵盖了 HTTP 协议的核心概念、方法类型、状态码、URL、REST 和 CRUD 操作。随后，它转向了特定于 ExpressJS 的主题，例如请求处理器、中间件和安全，以及特定于 Mongoose 的主题，例如模式、模型和自定义验证。

MVC 架构模式中的视图在关于 ReactJS 的章节中进行了介绍。ReactJS 是一个基于组件的 UI 库，具有声明式 API。本书旨在提供构建 ReactJS Web 应用程序和组件的必要知识。作为 ReactJS 的补充，本书包含了一个关于 Redux 的完整章节，从核心概念和原则到高级功能，如 store enhancers、时间旅行和异步数据流。

此外，本书还涵盖了使用 ExpressJS 和 SocketIO 进行实时通信，以实时交付和交换数据。

在本书结束时，你将了解使用 MVC 架构模式构建全栈 Web 应用程序的核心概念和基本要素。

# 为了充分利用本书

本书是为对使用 MERN 栈开发 Web 应用程序感兴趣的开发者而编写的。为了能够理解这些章节，你应该已经具备 JavaScript 语言的一般知识和理解。

# 你需要这本书的内容

为了能够处理食谱，你需要以下内容：

+   你偏好的 IDE 或代码编辑器。在编写食谱代码时使用了 Visual Studio Code (vscode)，所以我建议你尝试一下。

+   能够运行 NodeJS 和 MongoDB 的操作系统（O.S），最好是以下之一：

    +   macOS X Yosemite/El Capitan/Sierra

    +   Linux

    +   Windows 7/8/10（如果在 Windows 7 中安装 VSCode，则需要.NET 框架 4.5）

+   建议至少有 1 GB 的 RAM 和 1.6 GHz 或更快的处理器。

# 下载示例代码文件

你可以从[www.packtpub.com](http://www.packtpub.com)的账户下载本书的示例代码文件。如果你在其他地方购买了本书，你可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给你。

你可以通过以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)上登录或注册。

1.  选择“支持”标签。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入本书的名称，并遵循屏幕上的说明。

下载文件后，请确保使用最新版本解压缩或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

书籍的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/MERN-Quick-Start-Guide`](https://github.com/PacktPublishing/MERN-Quick-Start-Guide)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有其他来自我们丰富的书籍和视频目录的代码包，可在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**上找到。查看它们吧！

# 下载彩色图像

我们还提供了一个包含本书中使用的截图/图表的彩色图像的 PDF 文件。您可以从这里下载：[`www.packtpub.com/sites/default/files/downloads/MERNQuickStartGuide_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/MERNQuickStartGuide_ColorImages.pdf)。

# 代码在行动

访问以下链接查看代码运行的视频：

[`goo.gl/ymdYBT`](https://goo.gl/ymdYBT)

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“将下载的`WebStorm-10*.dmg`磁盘映像文件作为系统中的另一个磁盘挂载。”

代码块设置如下：

```js
 { 
        "dependencies": { 
          "express": "4.16.3", 
          "node-fetch": "2.1.1", 
          "uuid": "3.2.1" 
        } 
      } 
```

任何命令行输入或输出都应如下编写：

```js
npm install
```

**粗体**：表示新术语、重要单词或您在屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“从管理面板中选择系统信息。”

警告或重要注意事项看起来像这样。

小技巧和技巧看起来像这样。

# 部分

在本书中，您会发现一些频繁出现的标题（*准备工作*，*如何操作*，*让我们测试一下*，*它是如何工作的*，*还有更多...*，和*也见*）。

为了清楚地说明如何完成食谱，请按以下方式使用这些部分：

# 准备工作

本节告诉您在食谱中可以期待什么，并描述了如何设置任何软件或任何为食谱所需的初步设置。

# 如何操作...

本节包含遵循食谱所需的步骤。

# 让我们测试一下...

本节包含关于如何在*如何操作*部分测试给定代码的详细步骤。

# 它是如何工作的...

本节通常包含对前节发生情况的详细解释。

# 还有更多...

本节包含有关食谱的附加信息，以便您对食谱有更多的了解。

# 也见

本节提供了对食谱其他有用信息的链接。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：请发送电子邮件至 `feedback@packtpub.com` 并在邮件主题中提及书名。如果您对本书的任何方面有疑问，请发送电子邮件至 `questions@packtpub.com`。

**勘误**：尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告这一点。请访问 [www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上发现我们作品的任何非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过发送链接至 `copyright@packtpub.com` 与我们联系。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且对撰写或参与一本书籍感兴趣，请访问 [authors.packtpub.com](http://authors.packtpub.com/).

# 评论

请留下评论。一旦您阅读并使用了这本书，为何不在您购买它的网站上留下评论呢？潜在读者可以查看并使用您的客观意见来做出购买决定，我们 Packt 可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

如需更多关于 Packt 的信息，请访问 [packtpub.com](https://www.packtpub.com/).
