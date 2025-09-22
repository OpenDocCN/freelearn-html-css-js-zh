# 前言

由 Netscape 的 Brendan Eich 于 1995 年创建的 JavaScript，已经从仅在浏览器中使用的玩具脚本语言发展成为世界上用于全栈开发最受欢迎的语言之一。随着 Node.js 的引入，它建立在 Chrome 的 V8 JavaScript 引擎之上，开发者现在能够使用 JavaScript 构建强大且性能卓越的应用程序。随着现代 NoSQL 数据库 MongoDB 的加入，应用程序可以在每个层级上利用 JavaScript。

许多流行的 Web 应用程序部分或全部作为单页应用程序（SPA）实现。使用 SPA，用户只需加载一个网页，该网页会根据用户交互或传入的服务器数据进行动态更新。其优势是提供更平滑的应用程序体验，模仿原生应用程序交互，并且可能需要更少的网络流量和服务器负载。

MEAN 栈是一套从数据库到运行时环境、应用程序框架和前端的 JavaScript 工具集合，代表了一个全栈。本书详细介绍了使用 MEAN 栈和其他 JavaScript 生态系统中的工具构建 SPA 的背景。它涵盖了 SPA 架构和 JavaScript 工具的基础知识。本书随后扩展到更高级的主题，例如使用 MEAN 栈构建、保护和部署 SPA。

# 本书涵盖内容

第一章, *使用 NPM、Bower 和 Grunt 进行组织*，介绍了 JavaScript 前端包管理、构建和任务运行工具。这些工具构成了您设置理想开发环境的基础。

第二章, *模型-视图-任意*，超越了原始的 MVC 设计模式，并探讨了其在前端框架中的转换及其演变为模型-视图-*，或模型-视图-任意（MVW），其中控制器层更加开放且通常被抽象为更适合 SPA 环境的其他层。

第三章, *SPA 基础 - 创建理想的应用程序环境*，向您介绍了 SPA 的常见组件/层，这些组件的最佳实践和变体，以及如何将它们组合起来并为现代 SPA 打下基础。

第四章, *REST 是最佳选择 - 与应用程序后端交互*，进一步详细介绍了 SPA 架构的后端——特别是关于 REST（表示状态转移）架构模式。您将熟悉使用 JavaScript 和 AJAX 与 REST API 交互的不同方法，并伴有实际示例。

第五章, *一切关于视图*，专注于 SPA 架构中的视图概念以及如何在单页容器中初始化视图。它讨论了 JavaScript 模板，并提供了来自不同库的示例，深入探讨了 AngularJS 视图的实现方式。

第六章, *数据绑定，以及为什么你应该拥抱它*，教您关于数据绑定的知识，描述了一对一与双向数据绑定，并讨论了在 SPA 中数据绑定的实用性以及为什么您应该使用它。您还将涵盖 ECMAScript/JavaScript 标准的持续演变，以及它现在如何支持某些客户端的原生数据绑定。

第七章, *利用 MEAN 栈*，向您介绍 MEAN 栈（MongoDB、Express、AngularJS 和 Node.js）以及它们是如何协同工作的。您将安装和配置 MongoDB 和 Node.js，并在命令行中探索与每个组件的工作方式。您将为新的 SPA 创建数据库，并了解 AngularJS 和 Express，这是栈中的另外两个组件。

第八章, *使用 MongoDB 管理数据*，教您如何在 MongoDB 中创建和管理数据库。使用命令行，您将执行 CRUD（创建、读取、更新和删除）功能。

第九章, *使用 Express 处理 Web 请求*，使您熟悉 Express 路由中间件以及处理来自和发送到浏览器的请求。在配置 Express 路由器后，您将创建多个路由，当请求时，这些路由将返回动态生成数据给网络浏览器，逻辑地组织您的路由，并处理来自表单的 POST 请求。

第十章, *显示视图*，探讨了在 Express 中结合动态视图渲染与 AngularJS。您将配置 Express 应用程序使用 EJS（嵌入式 JavaScript）模板，并使用 Bootstrap 进行基本样式设计。

第十一章, *添加安全和身份验证*，教您如何通过防止常见的漏洞，如跨站请求伪造（CSRF），来保护基于 Express 的 SPA。您将为 Node.js 安装 passport-authentication 中间件，并对其进行本地身份验证配置，以及设置会话管理。

第十二章, *连接应用至社交媒体*，通过连接到多个社交媒体平台来扩展 SPA。您将使用 Facebook 和 Twitter 策略设置护照身份验证并共享 SPA 数据。

第十三章, *使用 Mocha、Karma 等测试*，教您在服务器端和视图中进行测试。

第十四章, *部署和扩展 SPA*，将指导您在 MongoLab 上设置生产数据库并将您的 SPA 部署到 Heroku。最后，您将探索在云中扩展您的 SPA。

# 您需要为这本书准备什么

除了您的网络浏览器和操作系统命令行或终端之外，这本书只需要很少的软件。您需要一个编辑器来编写代码。任何文本编辑器都可以，从记事本到像 Jetbrains WebStorm 这样的 IDE。

# 本书面向对象

这本书非常适合想要用 JavaScript 构建复杂单页应用（SPA）的 JavaScript 开发者。对 SPA 概念的基本了解将有所帮助，但不是必需的。

# 惯例

在这本书中，您将找到许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称如下所示："我们可以通过使用 include 指令来包含其他上下文。"

代码块如下所示：

```js
module.exports = function(grunt) {

    grunt.initConfig({
        pkg: grunt.file.readJSON('package.json'),
```

当我们希望将您的注意力引到代码块的一个特定部分时，相关的行或项目将以粗体显示：

```js
    grunt.initConfig({
        pkg: grunt.file.readJSON('package.json'),
        clean: ['dist/**'],
        copy: {
            main: {
        files: [

```

任何命令行输入或输出都如下所示：

```js
$ npm install -g grunt-cli
grunt-cli@0.1.13 /usr/local/lib/node_modules/grunt-cli
├── resolve@0.3.1
├── nopt@1.0.10 (abbrev@1.0.7)
└── findup-sync@0.1.3 (lodash@2.4.2, glob@3.2.11)

```

新术语和重要词汇以粗体显示。你在屏幕上看到的单词，例如在菜单或对话框中，在文本中显示如下："点击**下一步**按钮将您带到下一屏幕。"

### 注意

警告或重要注意事项如下所示。

### 小贴士

小贴士和技巧如下所示。

# 读者反馈

我们欢迎读者的反馈。告诉我们您对这本书的看法——您喜欢或不喜欢什么。读者反馈对我们很重要，因为它帮助我们开发您真正能从中获得最大收益的标题。要发送一般反馈，请简单地发送电子邮件至 feedback@packtpub.com，并在邮件主题中提及书的标题。如果您在某个主题上有专业知识，并且您有兴趣撰写或为书籍做出贡献，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您已经是 Packt 图书的骄傲拥有者，我们有一些可以帮助您充分利用购买的东西。

## 下载示例代码

您可以从您的账户下载这本书的示例代码文件[`www.packtpub.com`](http://www.packtpub.com)。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的**支持**选项卡上。

1.  点击**代码下载与勘误**。

1.  在**搜索**框中输入书的名称。

1.  选择您想要下载代码文件的书籍。

1.  从下拉菜单中选择您购买此书的来源。

1.  点击**代码下载**。

文件下载完成后，请确保您使用最新版本的软件解压缩或提取文件夹：

+   WinRAR / 7-Zip for Windows

+   Zipeg / iZip / UnRarX for Mac

+   7-Zip / PeaZip for Linux

该书的代码包也托管在 GitHub 上，网址为 [`github.com/PacktPublishing/Mastering-JavaScript-Single-Page-Application-Development`](https://github.com/PacktPublishing/Mastering-JavaScript-Single-Page-Application-Development)。我们还有其他来自我们丰富图书和视频目录的代码包，可在 [`github.com/PacktPublishing/`](https://github.com/PacktPublishing/) 找到。查看它们吧！

## 错误清单

尽管我们已经尽一切努力确保我们内容的准确性，但错误仍然可能发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——如果您能向我们报告，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何错误清单，请通过访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**错误提交表单**链接，并输入您的错误详细信息来报告它们。一旦您的错误清单得到验证，您的提交将被接受，错误清单将被上传到我们的网站或添加到该标题的错误清单部分。

要查看之前提交的错误清单，请访问 [`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将出现在**错误清单**部分。

## 盗版

互联网上对版权材料的盗版是一个持续存在的问题，涉及所有媒体。在 Packt，我们非常重视保护我们的版权和许可证。如果您在互联网上发现我们作品的任何非法副本，请立即提供位置地址或网站名称，以便我们可以追究补救措施。

请通过 copyright@packtpub.com 联系我们，并提供疑似盗版材料的链接。

我们感谢您在保护我们作者和我们为您提供有价值内容的能力方面的帮助。

## 询问

如果您在这本书的任何方面遇到问题，您可以通过 questions@packtpub.com 联系我们，我们将尽力解决问题。
