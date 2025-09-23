# 前言

API 是每个严肃 Web 应用程序的核心。Node.js 是一个特别令人兴奋的工具，易于使用，允许你构建 API，并在 JavaScript 中开发后端代码。它为包括 PayPal、Netflix 和 Zenhub 在内的 Web 应用程序的后端提供动力。

Express.js 是 Node.js 上最流行的框架之一，可以用来构建强大的 Web 应用程序——它提供了开发健壮 Web 应用程序的基本抽象级别。随着这个最小化和灵活的 Node.js Web 应用程序框架的出现，创建 Node.js 应用程序变得更加简单、快速，并且需要最小的努力。

本书采用实用主义方法，充分利用 Express.js 所能提供的功能，介绍了关键库，并全面装备你从零开始构建可扩展 API 所需的所有技能和工具，同时提供多年经验积累的微妙细节和智慧结晶。

# 本书涵盖的内容

第一章, *构建基本的 Express 网站*，将提供一个基本的应用程序（脚手架），我们将用它来演示接下来的示例。你将了解 Express 应用程序的外观。

第二章, *强大的电影 API*，将指导你构建一个电影 API，允许你将演员和电影信息添加到数据库中，并将演员与电影以及反之连接起来。

第三章, *多人游戏 API – 连接 4*，将围绕构建多人游戏 API 展开。我们还将使用测试驱动开发（TDD）和最大代码覆盖率来构建应用程序。

第四章, *多人在线文字游戏*，将教你如何使用 Express 和 SocketIO 构建实时应用程序，执行 socket 握手的身份验证，并使用 MongoDB 的原子更新处理竞争条件。

第五章, *与陌生人共进咖啡*，将使你能够编写一个允许用户去喝咖啡的 API！它将包含一个简单且可扩展的用户匹配系统。

第六章, *Koa.js 上的 Hacker News API*，将带你构建一个用于发布链接和点赞的 CRUD 后端，我们将使用 thunks 来集中处理错误，避免回调地狱。

附录, *连接 4 – 游戏逻辑*，展示了我们在第三章“多人游戏 API – 连接 4”中省略的配套游戏逻辑。

# 你需要为本书准备的内容

要开始使用本书中的示例，你需要以下内容：

+   Nvm: [`github.com/creationix/nvm`](https://github.com/creationix/nvm)

+   MongoDB: [`www.mongodb.org/downloads`](https://www.mongodb.org/downloads)

+   RoboMongo: [`robomongo.org/`](http://robomongo.org/)

+   Mocha：使用`npm i -g mocha`命令下载

Mac OS 是首选，但不是必需的。

# 本书面向对象

本书适合 Node.js 初学者，也适合技术先进的读者。本书结束时，每位开发者都将具备使用 Express 构建 Web 应用程序的专业知识。

# 惯例

在这本书中，您将找到许多不同的文本样式，以区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称如下所示："如果它是，那么我们使用`req.user`作为数据渲染`users/profile.jade`模板"。

代码块应如下设置：

```js
var express = require('express');
var app = express();

app.get('/', function(req, res, next) {
 res.send('Hello, World!');
});

app.listen(3000);
console.log('Express started on port 3000');
```

任何命令行输入或输出都应如下编写：

```js
$ npm install --save express

```

**新术语**和**重要词汇**以粗体显示。您在屏幕上看到的单词，例如在菜单或对话框中，在文本中如下所示："您还可以右键单击页面，并选择**检查元素**"。

### 注意

警告或重要提示将以如下框的形式出现。

### 小贴士

小技巧和窍门如下所示。

# 读者反馈

我们欢迎读者的反馈。请告诉我们您对这本书的看法——您喜欢或不喜欢的地方。读者反馈对我们很重要，因为它帮助我们开发出您真正能从中受益的书籍。

要向我们发送一般反馈，请简单地发送电子邮件至`<feedback@packtpub.com>`，并在邮件主题中提及书籍的标题。

如果您在某个主题上具有专业知识，并且您对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您已经是 Packt 图书的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)的账户下载示例代码文件，以获取您购买的所有 Packt Publishing 书籍。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 错误清单

尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在我们的某本书中发现了错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何错误清单，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**错误提交表单**链接，并输入您的错误详细信息。一旦您的错误清单得到验证，您的提交将被接受，错误清单将被上传到我们的网站或添加到该标题的错误部分下的现有错误清单中。

要查看之前提交的勘误表，请访问 [`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将出现在**勘误**部分。

## 海盗行为

互联网上对版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现我们作品的任何非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过 `<copyright@packtpub.com>` 联系我们，并提供涉嫌盗版材料的链接。

我们感谢您在保护我们作者和我们为您提供有价值内容的能力方面的帮助。

## 询问

如果您对本书的任何方面有问题，请通过 `<questions@packtpub.com>` 联系我们，我们将尽力解决问题。
