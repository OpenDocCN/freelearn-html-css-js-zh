# 前言

使用 Node.js 构建的 Web 应用程序越来越受到开发人员的欢迎和青睐。如今，随着 Node.js 的不断发展，我们可以看到许多公司都在使用这项技术开发他们的应用程序。其中，Netflix、Paypal 等公司都在生产环境中使用 Node.js。

托管公司也通过在其平台上支持 Node.js 取得了突破。此外，许多构建工具，如任务运行器、生成器和依赖管理器，也使用 Node.js 引擎出现，如 Grunt、Gulp、Bower 等。

本书将向您展示如何使用 Node.js 生态系统中的所有资源从头开始构建和部署 Node.js 应用程序，并探索云服务以进行测试、图像处理和部署。

处理所有这些工具并充分利用它们是一项非常有趣和激动人心的任务。

我们还将介绍 Docker 容器的概念，以及使用不同工具和服务进行持续集成。

在本书中，我们将学习如何充分利用这种开发方法，使用最新和最先进的技术从头到尾构建十个应用程序。

享受吧！

# 本书涵盖的内容

在整本书中，我们将探索构建 Node.js 应用程序的不同方式，并了解使用 MVC 设计模式构建基本博客页面的各个元素。我们将学习如何处理不同类型的视图模板，如 EJS 和 SWIG，以及使用命令行工具部署和运行应用程序等更复杂的内容。

我们将涵盖 Restful API 架构的基本概念，以及使用 jQuery、React.js 和 Angular.js 进行客户端通信。

尽管一些内容较为高级，但您将准备好理解 Node.js 应用程序的核心概念，以及如何处理不同类型的数据库，如 MongoDB、MySQL，以及 Express 和 Loopback 框架。

第一章 *使用 MVC 设计模式构建类似 Twitter 的应用程序*，展示了 MVC 模式应用于 Node.js 应用程序的主要概念，使用 Express 框架、mongoose ODM 中间件和 MongoDB 数据库。我们将了解如何使用 Passport 中间件处理用户会话和身份验证。

第二章 *使用 MySQL 数据库构建基本网站*，是一个真正深入了解使用关系数据库的 Node.js 应用程序。我们将了解如何使用 Sequelize（ORM）中间件与 Mysql 数据库，如何创建数据库关系，以及如何使用迁移文件。

第三章 *构建多媒体应用程序*，教会您如何处理文件存储和上传多媒体文件，如图像和视频。我们还将看到如何在 MongoDB 上保存文件名，并如何检索文件并在用户界面上显示。然后，我们将学习如何使用 Node.js 流 API 进行写入和读取。

第四章 *不要拍照，要创造 - 面向摄影师的应用程序*，涵盖了使用 Cloudnary 云服务上传、存储和处理图像的应用程序，并与 MongoDB 进行交互。此外，我们还将看到如何为用户界面实现 Materialize.css 框架，并介绍使用点文件加载配置变量的方法。

第五章 *使用 MongoDB 地理空间查询创建门店定位应用程序*，解释了使用 MongoDB 进行地理空间数据和地理位置的核心概念，以及支持 GEOJSON 数据格式的最有用的功能之一，即 2dspheres 索引。您将了解如何将 Google Maps API 与 Node.js 应用程序集成。

第六章*，使用 Restful API 和 Loopback.io 构建客户反馈应用程序*，探讨了 loopback.io 框架来构建 Restful API。我们将了解 Loopback CLI 的基础知识，以便使用命令行创建整个应用程序。您将学习如何处理使用 MongoDB 的模型之间的关系，以及如何在客户端使用 React.js 与 API 进行通信。

第七章*，使用 Socket.io 构建实时聊天应用程序*，展示了使用 Socket.io 事件构建聊天应用程序的基础知识，使用 Express 和 jQuery 进行用户界面。它涵盖了任务管理器的基本概念，以及如何使用 Gulp 和 livereload 插件。

第八章*，使用 Keystone CMS 创建博客*，讨论了完全由 Node.js 制作的 CMS，称为 Keystone。这是对 Keystone 应用程序结构的深入探讨，以及如何扩展框架以创建新模型和视图。此外，我们将看到如何自定义和创建新的 Keystone 主题。

第九章*，使用 Node.js 和 NPM 构建前端流程*，特别有趣，因为我们将使用 loopback.io 框架创建一个 Restful 应用程序，并使用 AngularJS 进行用户界面。此外，我们将使用不同的构建工具使用命令行和 Node Package Manager（NPM）来连接、缩小和优化图像。我们还将看到如何使用 Heroku toolbelt CLI 来创建和部署应用程序。

第十章*，使用持续集成和 Docker 创建和部署*，探讨了使用 Node.js 应用程序的持续交付开发过程。您将学习如何将工具集成到您的开发环境中，例如 Github，Codeship 和 Heroku，以处理单元测试和自动部署。本章还教您如何设置环境变量以保护您的数据库凭据，以及如何使用 Docker 容器的概念创建完整的应用程序。

# 您需要为本书准备什么

本书中的所有示例均使用开源解决方案，并可以从每章提供的链接免费下载。

本书的示例使用许多 Node.js 模块和一些 JavaScript 库，例如 jQuery，React.js 和 AngularJS。在撰写本书时，最新版本为 Node.js 5.6 和 6.1。

在第一章*，使用 MVC 设计模式构建类似 Twitter 的应用程序*，您可以按照逐步指南安装 Node 和 Node Package Manager（NPM）。

您可以使用您喜欢的 HTML 编辑器。

现代浏览器也会非常有帮助。我们使用 Chrome，但请随意使用您喜欢的浏览器。我们推荐以下之一：Safari，Firefox，Chrome，IE 或 Opera，均为最新版本。

# 本书的受众

您必须具备 JavaScript、HTML 和 CSS 的基本到中级知识，才能跟随本书中的示例，但在某些章节中可能需要更高级的 Web 开发/Restful API 和 Node.js 模块/中间件知识。不用担心；通过示例，我们将详细介绍所有代码，并为您提供许多有趣的链接。

# 惯例

在本书中，您将找到一些文本样式，用于区分不同类型的信息。以下是一些样式的示例，以及它们的含义解释。

文本中的代码词显示如下：

在继续之前，让我们将欢迎消息从：routes/index.js 文件更改为以下突出显示的代码。

代码块设置如下：

```js
/* GET home page. */
router.get('/', function(req, res, next) {
    res.render('index', { title: 'Express from server folder' });
});
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目会以粗体显示：

```js
/* GET home page. */
router.get('/', function(req, res, next) {
    res.render('index', { title: 'Express from server folder' });
});
```

新术语和重要单词以粗体显示。例如，屏幕上看到的单词，在菜单或对话框中出现的单词会以这样的方式出现在文本中：“点击“下一步”按钮会将您移动到下一个屏幕”。

警告或重要提示会出现在这样的框中。

提示和技巧会出现在这样。

### 注意

警告或重要提示会出现在这样的框中。

### 提示

提示和技巧会出现在这样。
