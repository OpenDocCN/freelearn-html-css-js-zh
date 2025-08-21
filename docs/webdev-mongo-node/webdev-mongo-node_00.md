# 前言

随着 ECMAscript 6 的出现，Node.JS 的可用性在未来有很大的发展空间，并且已经在今天得到了实现。学习 es6 语法糖的需求以及包含大多数跨技术特性的需求，激励了不同的技术社区学习 JavaScript。

Node.js 的高性能和可扩展性以及名为 MongoDB 的开源 NoSQL 数据库解决方案适用于轻松构建快速、可扩展的网络应用程序。这种组合使得管理任何形式的数据变得简单，并确保其交付速度。

本书旨在提供使用 Node.JS 和 MongoDB 构建等同服务器端渲染 Web 应用程序的不同方面。本书还指导我们使用 hapi.js 创建可配置的 Node.JS 服务器，并学习使用 Angular 4 开发单页前端应用程序。

本书将首先介绍您建立开发环境所需的基础知识，并对新的 ECMAscript 与传统 JavaScript 的不同进行比较研究。一旦基础就绪，我们将快速浏览必要的步骤，使主要应用程序服务器运行起来，并学习 Node.JS 核心。

此外，我们将通过使用控制器和 ViewModels 来生成可重用代码，从而减少开发时间。开发以学习适当的测试概念以及如何自动化测试以实现可重用性和可维护性而结束。

在本书结束时，您将与 JavaScript 生态系统连接，并了解流行的 JavaScript 前端和后端框架。

# 本书涵盖内容

第一章，*欢迎来到全栈 JavaScript*，介绍了 Node.js 和 MongoDB。除此之外，它还将解释您将使用本书构建的应用程序的整体架构。

第二章，*启动和运行*，解释了如何为 Node.js 和 MongoDB 设置开发环境。您还将通过编写一个示例应用程序并运行它来验证一切是否设置正确。

第三章，*Node 和 MongoDB 基础*，是关于学习 JavaScript 的基础知识。此外，还介绍了 NodeJS 的需要了解的概念以及 MongoDB 上的基本 CRUD 操作。

第四章，*介绍 Express*，向您介绍了 Express 框架及其各个组件。它还指导您如何组织使用该框架构建的基本应用程序。它还将详细介绍 Express 的 MVC 组件。

第五章，*使用 Handlebars 进行模板化*，向您介绍了使用模板引擎和 handlebars 的概念。此外，它还向您展示了如何在应用程序中使用 handlebars 作为模板引擎。

第六章，*控制器和视图模型*，向您展示了如何将构建的示例应用程序的代码组织到 Express 框架的控制器和视图中。它将通过介绍将代码分离到各种模块并利用 Express 框架来间接介绍 MVS 概念。

第七章，*使用 MongoDB 持久化数据*，向您展示了如何从正在构建的 Node.js 应用程序连接到 MongoDB 服务器。它还将向您介绍 ODM 的概念，最流行的是 Mongoose。

第八章，*创建 RESTful API*，向您介绍了 RESTful API。它还向您展示了 RESTful 包装器对应用程序的重要性。然后，它将教您如何将当前应用程序更改为基于 REST API 的应用程序。

第九章，*测试您的代码*，向您展示为什么需要将测试与应用程序结合，并且还会提到您在本章编写的代码的可测试性需要注意的事项。

第十章，*使用基于云的服务部署*，讨论了托管您正在构建的 Node.js MongoDB 应用程序的选项。它还比较了市场上可用的各种 PaaS 解决方案。

第十一章，*流行的 Node.js Web 框架*，介绍了除了 Express 之外在 Node.js 上可用的各种 Web 框架，您将在本书中用于构建应用程序。您将分析各种 Web 框架，如 Meteor、Sails、Koa、Hapi 和 Flatiron。您还将通过创建 API 服务器更详细地学习一种独特类型的框架，即 hapi.js。

第十二章，*使用流行的前端框架创建单页应用程序*，提供了单页应用程序与流行的前端框架（如 backbone.js、ember.js、react.js 和 Angular）的比较研究。您将详细了解一种流行的框架--Angular4。此外，您还将分析流行的前端方面，如可用的自动化工具和转译器。

# 您需要为本书做好准备

本书只需要对 JavaScript 和 HTML 有基本的了解。然而，本书的设计也有助于具有基本编程知识和跨平台开发人员学习 JavaScript 及其框架的初学者。

# 本书适合对象

本书适用于具有以下标准的 JavaScript 开发人员：

+   那些想要学习后端 JavaScript 的人

+   那些了解 es5 并希望从新的 ECMAscript 开始的人

+   那些具有 JavaScript 中级知识并希望探索新框架，如 Angular 2、hapi 和 Express 的人

最后，本书适用于任何渴望学习 JavaScript 并希望在 Node.js 和 MongoDB 中构建交互式 Web 应用程序的跨平台开发人员。

# 约定

在本书中，您将找到许多文本样式，用以区分不同类型的信息。以下是一些示例以及它们的含义解释。

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名显示如下：

"在上述情况中，`setTimeout()`方法由 JavaScript（Node.js）API 提供。"

代码块设置如下：

```js
var http = require('http');
http.createServer(function(req, res) {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('Hello World\n');
}).listen(8080, 'localhost');
console.log('Server running at http://localhost:8080'); 
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目将以粗体显示：

```js
app.engine('Handlebars', exphbs.create({ 
    defaultLayout: 'main', 
    layoutsDir: app.get('views') + '/layouts', 
    partialsDir: [app.get('views') + '/partials'], 
 helpers: { 
        timeago: (timestamp)=> { 
            return moment(timestamp).startOf('minute').fromNow(); 
        } 
    } 
}).engine); 
```

任何命令行输入或输出都以以下方式书写：

```js
$ sudo apt-get install python-software-properties $ sudo curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash - $ sudo apt-get install nodejs
```

**新术语**和**重要单词**以粗体显示。您在屏幕上看到的单词，例如

例如，在菜单或对话框中出现的文本如下所示：

Mac 上一个很好的替代品是 iTerm2。

警告或重要提示以此框出现。

提示和技巧显示如下。
