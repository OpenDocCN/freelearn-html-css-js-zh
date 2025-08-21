# 前言

MERN 堆栈可以被视为共享一个共同点的一组工具，即语言 JavaScript。本书以食谱的形式探讨了如何使用 MERN 堆栈构建遵循 MVC 架构模式的 Web 客户端和服务器应用程序。

MVC 架构模式的模型和控制器由关于使用 ExpressJS 和 Mongoose 构建 RESTful API 的章节涵盖。这些章节涵盖了关于 HTTP 协议、方法类型、状态码、URL、REST 和 CRUD 操作的核心概念。之后，它转向了特定于 ExpressJS 的主题，如请求处理程序、中间件和安全性，以及关于 Mongoose 的特定主题，如模式、模型和自定义验证。

MVC 架构模式的视图由关于 ReactJS 的章节涵盖。ReactJS 是一个基于组件的 UI 库，具有声明性 API。本书旨在提供构建 ReactJS Web 应用程序和组件的基本知识。除了 ReactJS，本书还包含了一个关于 Redux 的整个章节，从核心概念和原则到存储增强器、时间旅行和异步数据流等高级功能的解释。

此外，本书还涵盖了使用 ExpressJS 和 SocketIO 进行实时通信，以实现实时交换数据。

通过本书，您将了解使用 MVC 架构模式构建全栈 Web 应用程序的核心概念和要点。

# 为了充分利用本书

本书适用于有兴趣开始使用 MERN 堆栈开发 Web 应用程序的开发人员。为了能够理解章节，您应该已经对 JavaScript 语言有一般的知识和理解。

# 本书所需的内容

为了能够使用这些食谱，您需要以下内容：

+   您喜欢的 IDE 或代码编辑器。在编写食谱代码时使用了 Visual Studio Code（vscode），所以建议您试一试

+   能够运行 NodeJS 和 MongoDB 的操作系统（O.S），最好是以下之一：

+   macOS X Yosemite/El Capitan/Sierra

+   Linux

+   Windows 7/8/10（如果在 Windows 7 中安装 VSCode，则需要.NET 框架 4.5）

+   最好至少 1GB RAM 和 1.6GHz 处理器或更快

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了本书，可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，文件将直接发送到您的邮箱。

您可以按照以下步骤下载代码文件：

1.  登录或注册[www.packtpub.com](http://www.packtpub.com/support)。

1.  选择“支持”选项卡。

1.  点击“代码下载和勘误”。

1.  在搜索框中输入书名，按照屏幕上的说明操作。

下载文件后，请确保使用最新版本的解压缩或提取文件夹：

+   Windows 的 WinRAR/7-Zip

+   Mac 的 Zipeg/iZip/UnRarX

+   Linux 的 7-Zip/PeaZip

该书的代码包也托管在 GitHub 上[`github.com/PacktPublishing/MERN-Quick-Start-Guide`](https://github.com/PacktPublishing/MERN-Quick-Start-Guide)。如果代码有更新，将在现有的 GitHub 存储库上更新。

我们还有来自我们丰富书籍和视频目录的其他代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)上找到。快去看看吧！

# 下载彩色图片

我们还提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图片。您可以在这里下载：[`www.packtpub.com/sites/default/files/downloads/MERNQuickStartGuide_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/MERNQuickStartGuide_ColorImages.pdf)。

# 实战代码

访问以下链接查看代码运行的视频：

[`goo.gl/ymdYBT`](https://goo.gl/ymdYBT)

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄。 例如："将下载的`WebStorm-10*.dmg`磁盘映像文件挂载为系统中的另一个磁盘。"

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

任何命令行输入或输出都以以下方式编写：

```js
npm install
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。 例如，菜单或对话框中的单词会以这种形式出现在文本中。 例如："从管理面板中选择系统信息。"

警告或重要说明会以这种形式出现。

提示和技巧会出现在这样的形式中。

# 章节

在本书中，您会发现一些经常出现的标题（*准备工作*、*如何做...*、*让我们测试一下...*、*它是如何工作的...*、*还有更多...*和*参见*）。

为了清晰地说明如何完成一个食谱，使用以下章节：

# 准备工作

本节告诉您在食谱中可以期待什么，并描述了如何设置食谱所需的任何软件或任何初步设置。

# 如何做...

本节包含了遵循该食谱所需的步骤。

# 让我们测试一下...

本节包括有关如何测试*如何做...*部分中给出的代码的详细步骤。

# 它是如何工作的...

本节通常包括对上一节中发生的事情的详细解释。

# 还有更多...

本节包括有关食谱的其他信息，以使您对食谱更加了解。

# 参见

本节为食谱提供了其他有用信息的链接。
