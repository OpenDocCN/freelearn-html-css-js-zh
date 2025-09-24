# 前言

JavaScript 已经成为当今和未来最重要的语言之一。

在过去几年中，JavaScript 的崛起非常迅速，它已成为现代网络应用开发中的强大语言。

MEVN 是除了 MEAN 和 MERN 之外，用于开发现代网络应用的一种技术栈。本书提供了一种逐步构建全栈网络应用程序的方法，使用 MEVN 技术栈，即 MongoDB、Express.js、Vue.js 和 Node.js。

本书将提供 Node.js 和 MongoDB 的基本概念，接着是构建 Express.js 应用程序并实现 Vue.js。

在这本书中，我们将涵盖以下内容：

+   了解技术栈——MongoDB、Node.js、Express.js 和 Vue.js

+   构建 Express.js 应用程序

+   学习什么是 REST API 以及如何实现它们

+   学习如何在 Express.js 应用程序中将 Vue.js 作为前端层使用

+   在应用程序中添加一个认证层

+   添加自动化脚本和测试

# 这本书面向的对象

这本书是为对学习如何使用 JavaScript 作为唯一编程语言构建全栈应用程序感兴趣的网页开发者而设计的，使用的技术栈是：MongoDB、Express.js、Vue.js 和 Node.js。

这本书适合对 HTML、CSS 和 JavaScript 有基本知识的初学者和中级开发者。如果你是网页或全栈 JavaScript 开发者，并且已经尝试过传统的技术栈，如 LAMP、MEAN 或 MERN，并希望探索一个使用现代网络技术的全新技术栈，那么这本书适合你。

# 这本书涵盖的内容

第一章，*MEVN 简介*，介绍了 MEVN 栈以及构建应用程序基础所需的不同工具的安装。

第二章，*构建 Express 应用程序*，介绍了 Express.js，解释了 **模型**、**视图**、**控制器**（**MVC**）结构的概念，并展示了如何使用 Express.js 和 MVC 结构设置应用程序。

第三章，*MongoDB 简介*，重点介绍了 Mongo 及其查询，介绍了 Mongoose 以及使用 Mongoose 进行 **创建**、**读取**、**更新** 和 **删除**（**CRUD**）操作的性能。

第四章，*REST API*，介绍了 REST 架构以及 RESTful API 的概念。本章还介绍了不同的 HTTP 动词以及开发 REST API。

第五章，*构建真实应用程序*，介绍了 Vue.js 并展示了如何使用 MEVN 中的所有技术构建一个完全工作的动态应用程序。

第六章，*使用 Passport.js 进行身份验证*，讨论了 Passport.js，并描述了如何实现 JWT 和本地策略以在应用程序中添加身份验证层。

第七章，*Passport.js OAuth 策略*，介绍了 OAuth 策略，并指导您实现 Facebook、Twitter、Google 和 LinkedIn 的 Passport.js 策略。

第八章，*Vuex 简介*，介绍了 Vuex 的核心概念——状态、获取器、突变和动作。它还描述了您如何在应用程序中实现它们。

第九章，*测试 MEVN 应用程序*，解释了单元测试和端到端测试是什么，并指导您为应用程序的不同方面编写单元测试和自动化测试。

第十章，*Go Live*，解释了什么是持续集成，并指导您如何设置与应用程序的持续集成服务，并在 Heroku 上部署应用程序。

# 要充分利用本书

如果您具备以下技能，本书将更有益：

+   了解 HTML、CSS 和 JavaScript

+   了解 Vue.js 和 Node.js 是一个加分项

+   了解如何使用 MEAN 和 MERN 堆栈构建 Web 应用程序是一个加分项

# 下载示例代码文件

您可以从 [www.packtpub.com](http://www.packtpub.com) 的账户下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问 [www.packtpub.com/support](http://www.packtpub.com/support) 并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在 [www.packtpub.com](http://www.packtpub.com/support) 登录或注册。

1.  选择支持选项卡。

1.  点击代码下载与勘误表。

1.  在搜索框中输入本书的名称，并遵循屏幕上的说明。

文件下载后，请确保使用最新版本的以下软件解压缩或提取文件夹：

+   Windows 上的 WinRAR/7-Zip

+   Mac 上的 Zipeg/iZip/UnRarX

+   7-Zip/PeaZip for Linux

本书代码包也托管在 GitHub 上，网址为 [`github.com/PacktPublishing/Full-Stack-Web-Development-with-Vue.js-and-Node`](https://github.com/PacktPublishing/Full-Stack-Web-Development-with-Vue.js-and-Node)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有来自我们丰富的图书和视频目录的其他代码包，可在 [`github.com/PacktPublishing/`](https://github.com/PacktPublishing/) 找到。查看它们吧！

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“一个模块是可以由 Node.js 使用`require`命令加载的，并且有一个命名空间。模块与其关联的`package.json`文件。”

代码块设置如下：

```js
extends layout

block content
  h1= title
  p Welcome to #{title}
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```js
var index = require('./routes/index');
var users = require('./routes/users');

var app = express();

// Require file system module
var fs = require('file-system');
```

任何命令行输入或输出都应如下编写：

```js
$ mkdir css
$ cd css
```

**粗体**：表示新术语、重要单词或您在屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“只需点击“继续”，直到安装完成。”

警告或重要注意事项看起来像这样。

小贴士和技巧看起来像这样。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：请发送电子邮件至 `feedback@packtpub.com`，并在邮件主题中提及书籍标题。如果您对本书的任何方面有疑问，请发送电子邮件至 `questions@packtpub.com`。

**勘误**：尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告，我们将不胜感激。请访问 [www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上发现我们作品的任何非法副本，我们将不胜感激，如果您能提供位置地址或网站名称，我们将不胜感激。请通过链接至材料的方式与我们联系 `copyright@packtpub.com`。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。一旦您阅读并使用过这本书，为什么不在您购买它的网站上留下评论呢？潜在读者可以查看并使用您的客观意见来做出购买决定，Packt 可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

如需更多关于 Packt 的信息，请访问 [packtpub.com](https://www.packtpub.com/)。
