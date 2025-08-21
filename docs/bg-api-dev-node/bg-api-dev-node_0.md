# 前言

使用相同的框架构建服务器端和客户端应用程序可以节省时间和金钱。本书教你如何使用 JavaScript 和 Node.js 构建高度可扩展的 API，以便与轻量级跨平台客户端应用程序良好配合。它从 Node.js 的基础知识开始，快速地引导您创建一个示例客户端，该客户端与完全经过身份验证的 API 实现配对。

本书平衡了理论和练习，并包含了多个开放式活动，使用真实的商业场景让您练习和应用您新获得的技能。

我们包含了超过 20 个实际活动和练习，涵盖了 9 个主题，以加强您的学习。通过本书，您将具备进行自己的 API 开发项目所需的技能和经验。

# 本书适合对象

本书适合已经了解 JavaScript 并寻求快速简洁的 Node.js API 开发介绍的开发人员。虽然具有其他服务器端技术（如 Python、PHP、ASP.NET、Ruby）的经验会有所帮助，但在开始之前并不一定需要具备后端开发的背景。

# 本书涵盖的内容

第一章，*Node.js 简介*，涵盖了 Node.js 的一些基本概念，基本的 Node.js 代码以及如何从终端运行它，模块系统，其类别以及作为 Node.js 工作核心的异步编程模型，以及实际使 Node.js 运行的原理。

第二章，*构建 API-第一部分*，涵盖了构建基本的 HTTP 服务器，设置 Hapi.js，使用 Hapi.js 框架构建基本 API 以及 Web 应用程序的基本概念。

第三章，*构建 API-第二部分*，涵盖了 Knex.js 的介绍以及如何使用它连接和使用数据库，基本的 CRUD 数据库方法，使用 JWT 机制进行 API 身份验证，CORS 机制，使用 Lab 库测试 API 以及使用 Gulp.js 进行测试自动化。

# 充分利用本书

1.  具有其他服务器端技术经验，如 Python、PHP、ASP.NET 和 Ruby 将有益，但不是必需的。

1.  本书需要计算机系统。最低硬件要求为 1.8 GHz 或更高的奔腾 4（或等效）处理器，4 GB RAM，10 GB 硬盘和稳定的互联网连接。

1.  所需软件包括 Visual Studio Code ([`code.visualstudio.com/`](https://code.visualstudio.com/))，Node.js (8.9.1) ([`nodejs.org/en/`](https://nodejs.org/en/))，MySQL Workbench 6.3 ([`www.mysql.com/products/workbench/`](https://www.mysql.com/products/workbench/))和 MySQL ([`dev.mysql.com/downloads/mysql/`](https://dev.mysql.com/downloads/mysql/))。

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的帐户下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便文件直接通过电子邮件发送给您。

您可以按照以下步骤下载代码文件：

1.  在[www.packtpub.com/support](http://www.packtpub.com/support)上登录或注册。

1.  选择 SUPPORT 选项卡。

1.  单击“代码下载和勘误”。

1.  在搜索框中输入书名并按照屏幕上的说明操作。

下载文件后，请确保您使用最新版本的解压缩或提取文件夹：

+   Windows 需要 WinRAR/7-Zip

+   Mac 需要 Zipeg/iZip/UnRarX

+   Linux 需要 7-Zip/PeaZip

该书的代码包也托管在 GitHub 上，网址是[`github.com/TrainingByPackt/BeginningAPIDevelopmentwithNode.js`](https://github.com/TrainingByPackt/BeginningAPIDevelopmentwithNode.js)。如果代码有更新，将会在现有的 GitHub 存储库上进行更新。

我们还有其他代码包，来自我们丰富的图书和视频目录，可以在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**上找到。去看看吧！

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄。这里有一个例子：“完成此设置后，我们使用`server.start`方法启动服务器。”

代码块设置如下：

```js
handler: (request, reply) => 
{
  return reply({ message: 'hello, world' });
}
```

任何命令行输入或输出都是这样写的：

```js
node server.js
```

**粗体**：表示一个新术语、一个重要词或者屏幕上看到的词。例如，菜单或对话框中的单词会以这种方式出现在文本中。这里有一个例子：“将请求类型更改为 POST。”

**活动**：这些是基于场景的活动，让您可以在整个章节学习过程中实际应用所学知识。它们通常是在真实世界问题或情况的背景下进行。

警告或重要说明会以这种方式出现。
