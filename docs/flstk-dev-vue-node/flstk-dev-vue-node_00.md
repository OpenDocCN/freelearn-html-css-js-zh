# 前言

JavaScript 已成为当今和未来最重要的语言之一。

过去几年中 JavaScript 的崛起是如此迅猛，以至于它已成为开发现代 Web 应用程序的强大语言。

MEVN 是用于开发现代 Web 应用程序的堆栈之一，除了 MEAN 和 MERN。本书提供了使用 MEVN 技术逐步构建全栈 Web 应用程序的方法，其中包括 MongoDB、Express.js、Vue.js 和 Node.js。

本书将介绍 Node.js 和 MongoDB 的基本概念，继续构建 Express.js 应用程序并实现 Vue.js。

在本书中，我们将涵盖以下内容：

+   学习技术堆栈- MongoDB、Node.js、Express.js 和 Vue.js

+   构建 Express.js 应用程序

+   学习什么是 REST API 以及如何实现它们

+   学习在 Express.js 应用程序中使用 Vue.js 作为前端层

+   在应用程序中添加身份验证层

+   添加自动化脚本和测试

# 本书适合对象

本书旨在帮助对使用 Mongo DB、Express.js、Vue.js 和 Node.js 技术堆栈构建全栈应用程序感兴趣的 Web 开发人员学习。

本书适合具有 HTML、CSS 和 JavaScript 基本知识的初学者和中级开发人员。如果您是 Web 或全栈 JavaScript 开发人员，并且已经尝试过传统的堆栈，如 LAMP、MEAN 或 MERN，并希望探索具有现代 Web 技术的新堆栈，那么本书适合您。

# 本书涵盖的内容

第一章，“MEVN 简介”，介绍了 MEVN 堆栈以及构建应用程序所需的不同工具的安装。

第二章，“构建 Express 应用程序”，介绍了 Express.js，MVC 结构的概念，并向您展示如何使用 Express.js 和 MVC 结构设置应用程序。

第三章，“MongoDB 简介”，重点介绍了 Mongo 和其查询，介绍了 Mongoose 以及使用 Mongoose 执行 CRUD 操作的性能。

第四章，“REST API”，介绍了 REST 架构以及 RESTful API 是什么。本章还介绍了不同的 HTTP 动词和开发 REST API 的方法。

第五章，“构建真实应用程序”，介绍了 Vue.js，并向您展示如何使用 MEVN 中的所有技术构建一个完全工作的动态应用程序。

第六章，“使用 Passport.js 进行身份验证”，介绍了 Passport.js 是什么，并描述了如何实现 JWT 和本地策略以在应用程序中添加身份验证层。

第七章，“Passport.js OAuth 策略”，介绍了 OAuth 策略是什么，并指导您实现 Facebook、Twitter、Google 和 LinkedIn 的 Passport.js 策略。

第八章，“Vuex 简介”，介绍了 Vuex 的核心概念-状态、获取器、突变和操作。它还描述了如何在应用程序中实现它们。

第九章，“测试 MEVN 应用程序”，解释了单元测试和端到端测试是什么，并指导您编写应用程序不同方面的单元测试和自动化测试。

第十章，*Go Live*，解释了什么是持续集成，指导您如何设置一个持续集成服务，并在 Heroku 上部署应用程序。

# 充分利用本书

如果您具备以下技能，本书将对您最有益处：

+   了解 HTML、CSS 和 JavaScript

+   了解 Vue.js 和 Node.js 是一个加分项

+   了解如何使用 MEAN 和 MERN 堆栈构建 Web 应用程序是一个加分项

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了本书，可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便文件直接发送到您的邮箱。

您可以按照以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)上登录或注册。

1.  选择“支持”选项卡。

1.  单击“代码下载和勘误”。

1.  在搜索框中输入书名，然后按照屏幕上的说明操作。

下载文件后，请确保使用最新版本的解压缩或提取文件夹：

+   WinRAR/7-Zip 适用于 Windows

+   Zipeg/iZip/UnRarX 适用于 Mac

+   7-Zip/PeaZip 适用于 Linux

该书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Full-Stack-Web-Development-with-Vue.js-and-Node`](https://github.com/PacktPublishing/Full-Stack-Web-Development-with-Vue.js-and-Node)。如果代码有更新，将在现有的 GitHub 存储库上进行更新。

我们还有来自丰富图书和视频目录的其他代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)上找到。去看看吧！

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄。以下是一个例子：“模块是可以通过 Node.js 的`require`命令加载并具有命名空间的东西。模块有一个与之关联的`package.json`文件。”

代码块设置如下：

```js
extends layout

block content
  h1= title
  p Welcome to #{title}
```

当我们希望引起您对代码块的特定部分的注意时，相关的行或项目将以粗体显示：

```js
var index = require('./routes/index');
var users = require('./routes/users');

var app = express();

// Require file system module
var fs = require('file-system');
```

任何命令行输入或输出都将按照以下方式编写：

```js
$ mkdir css
$ cd css
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词会在文本中以这种方式出现。以下是一个例子：“只需点击“继续”，直到安装完成。”

警告或重要说明会出现在这样。

技巧和窍门会出现在这样。
