# 前言

当我被要求写这本培训书时，我意识到关于 Webpack 及其用途的了解很少。通常这是开发人员偶然发现并在工作中学习的东西，这可能是一个非常费力的过程。Webpack.js 网站上有一些文档，以及一些可靠的资源，比如 Medium。然而，这些资源往往从专家的角度来与读者交流，而我个人发现这并不理想。

作为一名网页开发讲师，我看到非常有技术和智慧的人可能存在盲点和知识空白。作为一名讲师，我被告知，并且也传达这个信息，那就是“没有愚蠢的问题”。许多没有教学背景的人可能会建议他们不想对你说教，宁愿你不问愚蠢的问题。我们发现，如果学生宁愿保持沉默而不问问题，这是有害的。

我打算尽可能保持简单。也许我已经失败了，使用了“定制”这样的词，尽管如此，前提是我们所有人都会有让人啪啪打脑袋的时刻，我们本应该做某事，后来意识到我们做错了。对吧？这种事发生在我们最聪明的人身上。此外，大多数讲师可能不愿意对你进行详尽的解释，因为他们担心冒犯你。问题在于，总会有一些平凡的细节，开发人员认为显而易见，但可能有多种解释。当我讲课时的规则是：“没有愚蠢的问题”，所以我希望证明这个理论的必要性。

# 这本书适合谁

这本书是为希望通过学习 Webpack 来开始他们的 Web 项目依赖管理的 Web 开发人员而写的。假定读者具有 JavaScript 的工作知识。

# 本书涵盖的内容

第一章，“Webpack 5 简介”，将向您介绍 Webpack——具体来说，是 Webpack 5 版本。它将概述围绕 Webpack 的核心概念以及它的使用方式。

第二章，“使用模块和代码拆分”，将详细介绍模块和代码拆分，以及 Webpack 5 的一些突出和有趣的方面，这些方面对于理解 Webpack 至关重要。

第三章，“使用配置和选项”，将探讨配置的世界，了解其局限性和能力，以及选项在其中的作用。

第四章，“API、插件和加载器”，将深入探讨 API、加载器和插件的世界。这些 Webpack 的特性阐述了平台的能力，从配置和选项出发。

第五章，“库和框架”，将讨论库和框架。我们对插件、API 和加载器的研究表明，有时我们不想使用诸如库之类的远程代码，但有时我们确实需要。Webpack 通常处理本地托管的代码，但有时我们可能需要使用库。这为我们引入了这个话题。

第六章，“生产、集成和联合模块”，将深入介绍这个主题，并希望解决开发人员可能存在的任何疑虑。

第七章，“调试和迁移”，将讨论热模块替换和实时编码，并深入了解一些严肃的教程。

第八章，*编写教程和实时编码技巧*，将向您展示 Webpack 5 的工作示例，特别是 Webpack 5 相对于早期版本的差异。将有纯 JavaScript 教程以及常见的框架，Vue.js 将是一个不错的选择。

# 为了充分利用本书

您可以在[`github.com/PacktPublishing/Webpack-5-Up-and-Running`](https://github.com/PacktPublishing/Webpack-5-Up-and-Running)找到本书所有章节中使用的代码。为了充分利用本书，您需要以下内容：

+   JavaScript 的基本知识。

+   确保您已安装最新版本的 Webpack 5。

+   您需要使用命令行界面，如命令提示符或其他您选择的命令行实用程序。

+   您将需要 Node.js，JavaScript 运行环境。

+   确保您已安装最新版本的 Node.js（webpack 5 至少需要 Node.js 10.13.0（LTS））；否则，您可能会遇到许多问题。

+   您需要在本地计算机上安装具有管理员级别权限的`npm`。Webpack 和 Webpack 5 在 Node.js 环境中运行，这就是为什么我们需要它的包管理器——NPM。

+   截至撰写本文时，最新版本是 Webpack 5。访问[`webpack.js.org`](https://webpack.js.org)查找适合您的最新版本。

# 下载示例代码文件

您可以从[www.packt.com](http://www.packt.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了本书，可以访问[www.packtpub.com/support](https://www.packtpub.com/support)并注册，以便文件直接发送到您的邮箱。

您可以按照以下步骤下载代码文件：

1.  登录或注册[www.packt.com](http://www.packt.com)。

1.  选择“支持”选项卡。

1.  点击“代码下载”。

1.  在搜索框中输入书名，并按照屏幕上的说明操作。

下载文件后，请确保使用以下最新版本的解压缩或提取文件夹：

+   Windows 使用 WinRAR/7-Zip

+   Mac 使用 Zipeg/iZip/UnRarX

+   Linux 使用 7-Zip/PeaZip

该书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Webpack-5-Up-and-Running`](https://github.com/PacktPublishing/Webpack-5-Up-and-Running)。如果代码有更新，将在现有的 GitHub 存储库中更新。

我们还有其他代码包来自我们丰富的图书和视频目录，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。快去看看吧！

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄。以下是一个示例：“以下行是`package.json`文件中的代码片段。”

代码块设置如下：

```js
"scripts": {
"build": "webpack --config webpack.config.js"
}
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目将以粗体显示：

```js
<!doctype html>
<html>
 <head>
 <title>Webpack - Test</title>
 <script src="img/lodash@4.16.6"></script>
 </head>
 <body>
 <script src="img/index.js"></script>
 </body>
</html>
```

任何命令行输入或输出都以以下形式编写：

```js
npm install --save-dev webpack-cli
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词会以这种形式出现在文本中。以下是一个示例：“从管理面板中选择系统信息。”

警告或重要说明会以这种形式出现。

提示和技巧会出现在这样的形式中。
