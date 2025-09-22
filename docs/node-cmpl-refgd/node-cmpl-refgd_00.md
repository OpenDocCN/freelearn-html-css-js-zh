# 前言

随着时间的推移，机器学习正变得越来越对企业具有变革性。使用 Node.js 的机器学习完整参考指南将您从 JavaScript 和服务器端开发的基础知识引导至创建、维护、部署和测试您自己的 Node.js 应用程序。

您将从学习如何使用 HTTP 服务器和客户端对象开始，使用 SQL 和 MongoDB 数据库存储数据，以及使用 Mocha 5.x 进行单元测试，并使用 Puppeteer 1.1.x 进行功能测试。然后，您将学习在 Node.js 平台上创建可扩展且丰富的 RESTful 应用，并编写一个具有自描述 URL 的简单 HTTP 请求处理器。您将学习设置准确的 HTTP 状态码，研究如何保持您的应用程序向后兼容，并探索一些认证技术以保护您的应用程序。然后，您将研究 Node.js 如何成为开发微服务的一个强有力的候选者。

通过本学习路径，您将能够使用最佳实践并创建高效的微服务。

本学习路径包括以下 Packt 产品的内容：

+   《使用 Node.js 10 设计 RESTful Web API，第三版》由瓦伦丁·博金诺夫著

+   《Node.js Web 开发，第四版》由大卫·赫伦著

+   《使用 Node.js 实践微服务》由迪戈·雷森德著

# 这本书面向谁

Node.js 完整参考指南是为那些对 JavaScript 和 Web 应用开发有基本了解的 Web 开发者设计的，他们渴望丰富他们的开发技能以创建 RESTful 应用，并希望利用他们的技能来构建微服务。

# 为了充分利用本书

本学习路径中提供的代码示例的执行需要运行在 POSIX-like 操作系统上的 Node.js，包括各种 UNIX 衍生品（例如 Solaris）或类似操作系统（Linux、macOS 等），以及 Microsoft Windows。它可以在大小不同的机器上运行，包括微小的 ARM 设备，例如用于 DIY 软件/硬件项目的 Raspberry Pi 微尺度嵌入式计算机。

它可以在大小不同的机器上运行，包括微小的 ARM 设备，例如用于 DIY 软件/硬件项目的 Raspberry Pi 微尺度嵌入式计算机。

由于许多 Node.js 包是用 C 或 C++编写的，您必须拥有 C 编译器（如 GCC）、Python 2.7（或更高版本）以及 node-gyp 包。如果您计划在网络代码中使用加密，您还需要 OpenSSL 加密库。现代 UNIX 衍生品几乎都包含这些，并且当从源代码安装时使用的 Node.js 的配置脚本将检测它们的存在。如果您需要安装它们，Python 可在[`python.org`](http://python.org)获取，OpenSSL 可在[`openssl.org`](http://openssl.org)获取[。](http://openssl.org)

# 下载示例代码文件

您可以从[www.packt.com](http://www.packt.com)的账户下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[www.packt.com/support](http://www.packt.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在 [www.packt.com](http://www.packt.com) 登录或注册。

1.  选择“支持”选项卡。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书籍名称，并遵循屏幕上的说明。

文件下载完成后，请确保您使用最新版本解压缩或提取文件夹。

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

该书的代码包也托管在 GitHub 上，网址为 [`github.com/PacktPublishing/Node.js-Complete-Reference-Guide`](https://github.com/PacktPublishing/Node.js-Complete-Reference-Guide)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有其他来自我们丰富图书和视频目录的代码包，可在 **[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)** 上找到。查看它们吧！

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 处理。以下是一个示例：“`EventEmitter` 对象定义在 Node.js 的 events 模块中。”

代码块设置如下：

```js
if (anotherNote instanceof Note) {
 ... it's a Note, so act on it as a Note
}
```

任何命令行输入或输出如下所示：

```js
$ npm update express
$ npm update
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“如果您需要不同的内容，请点击页眉中的**下载**链接以获取所有可能的下载：”

警告或重要说明如下所示。

技巧和窍门如下所示。

# 联系我们

我们欢迎读者的反馈。

**一般反馈**：如果您对本书的任何方面有疑问，请在邮件主题中提及书名，并给我们发送邮件至 `customercare@packtpub.com`。

**勘误**：尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告，我们将不胜感激。请访问 [www.packt.com/submit-errata](http://www.packt.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上以任何形式遇到我们作品的非法副本，我们将不胜感激，如果您能提供位置地址或网站名称，我们将不胜感激。请通过链接至材料的方式与我们联系至 `copyright@packt.com`。

**如果您有兴趣成为作者**：如果您在某个主题上具有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下您的评价。一旦您阅读并使用过这本书，为何不在购买它的网站上留下评价呢？潜在读者可以查看并使用您的客观意见来做出购买决定，我们 Packt 可以了解您对我们产品的看法，而我们的作者也可以看到他们对书籍的反馈。谢谢！

如需了解 Packt 的更多信息，请访问 [packt.com](http://www.packt.com/).
