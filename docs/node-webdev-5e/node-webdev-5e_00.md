前言

Node.js 是一个服务器端的 JavaScript 平台，允许开发人员在网页浏览器之外使用 JavaScript 构建快速可扩展的应用程序。它在软件开发世界中扮演着越来越重要的角色，最初作为服务器应用程序的平台，但现在在命令行开发工具甚至 GUI 应用程序中得到广泛应用，这要归功于 Electron 等工具包。Node.js 已经将 JavaScript 从浏览器中解放出来。

它运行在谷歌 Chrome 浏览器核心的超快 JavaScript 引擎 V8 之上。Node.js 运行时遵循一个巧妙的事件驱动模型，尽管使用单线程模型，但在并发处理能力方面被广泛使用。

Node.js 的主要重点是高性能、高可扩展性的 Web 应用程序，但它也在其他领域得到了应用。例如，基于 Node.js 的 Electron 包装了 Chrome 引擎，让 Node.js 开发人员可以创建桌面 GUI 应用程序，并成为许多热门应用程序的基础，包括 Atom 和 Visual Studio Code 编辑器、GitKraken、Postman、Etcher 和桌面版 Slack 客户端。Node.js 在物联网设备上很受欢迎。它的架构特别适合微服务开发，并经常帮助构建全栈应用程序的服务器端。

在单线程系统上提供高吞吐量的关键是 Node.js 的异步执行模型。这与依赖线程进行并发编程的平台非常不同，因为这些系统通常具有很高的开销和复杂性。相比之下，Node.js 使用一个简单的事件分发模型，最初依赖回调函数，但今天依赖 JavaScript Promise 对象和 async 函数。

由于 Node.js 建立在 Chrome 的 V8 引擎之上，该平台能够快速采用 JavaScript 语言的最新进展。Node.js 核心团队与 V8 团队密切合作，让它能够快速采用 V8 中实现的新 JavaScript 语言特性。Node.js 14.x 是当前版本，本书是针对该版本编写的。

# 第一章：这本书适合谁

服务器端工程师可能会发现 JavaScript 是一种优秀的替代编程语言。由于语言的进步，JavaScript 早就不再是一种只适用于在浏览器中为按钮添加动画效果的简单玩具语言。我们现在可以使用这种语言构建大型系统，而 Node.js 具有许多内置功能，比如一流的模块系统，可以帮助开发更大的项目。

有经验的浏览器端 JavaScript 开发人员可能会发现通过本书扩展视野，包括使用服务器端开发。

# 本书内容

《第一章》《关于 Node.js》介绍了 Node.js 平台。它涵盖了 Node.js 的用途、技术架构选择、历史、服务器端 JavaScript 的历史、JavaScript 应该从浏览器中解放出来以及 JavaScript 领域的重要最新进展。

《第二章》《设置 Node.js》介绍了如何设置 Node.js 开发环境。这包括在 Windows、macOS 和 Linux 上安装 Node.js。还介绍了一些重要的工具，包括`npm`和`yarn`包管理系统，以及用于将现代 JavaScript 转译为在旧 JavaScript 实现上可运行形式的 Babel。

《第三章》《探索 Node.js 模块》深入探讨了模块作为 Node.js 应用程序中的模块化单元。我们将深入了解和开发 Node.js 模块，并使用`npm`来维护依赖关系。我们将了解新的模块格式 ES6 模块，以及如何在 Node.js 中使用它，因为它现在得到了原生支持。

第四章，“HTTP 服务器和客户端”，开始探索 Node.js 的 Web 开发。我们将在 Node.js 中开发几个小型 Web 服务器和客户端应用程序。我们将使用斐波那契算法来探索重型、长时间运行计算对 Node.js 应用程序的影响。我们还将学习几种缓解策略，并获得我们开发 REST 服务的第一次经验。

第五章，“你的第一个 Express 应用程序”，开始了本书的主要旅程，即开发一个用于创建和编辑笔记的应用程序。在本章中，我们运行了一个基本的笔记应用程序，并开始使用 Express 框架。

第六章，“实现移动优先范式”，使用 Bootstrap V4 框架在笔记应用程序中实现响应式 Web 设计。这包括集成流行的图标集以及自定义 Bootstrap 所需的步骤。

第七章，“数据存储和检索”，探索了几种数据库引擎和一种可以轻松切换数据库的方法。目标是将数据稳健地持久化到磁盘。

第八章，“使用微服务对用户进行身份验证”，为笔记应用程序添加了用户身份验证。我们将学习使用 PassportJS 处理登录和注销。身份验证既支持本地存储的用户凭据，也支持使用 Twitter 的 OAuth。

第九章，“使用 Socket.IO 进行动态客户端/服务器交互”，让我们看看如何让用户实时交流。我们将使用 Socket.IO 这个流行的框架来支持内容的动态更新和简单的评论系统。所有内容都是由用户在伪实时中动态更新的，这给了我们学习实时动态更新的机会。

第十章，“将 Node.js 应用部署到 Linux 服务器”，是我们开始部署旅程的地方。在本章中，我们将使用传统的方法在 Ubuntu 上使用 Systemd 部署后台服务。

第十一章，“使用 Docker 部署 Node.js 微服务”，让我们开始探索使用 Docker 进行基于云的部署，将笔记应用程序视为一组微服务的集群。

第十二章，“使用 Terraform 在 AWS EC2 上部署 Docker Swarm”，让我们看看如何构建一个使用 AWS EC2 系统的云托管系统。我们将使用流行的工具 Terraform 来创建和管理 EC2 集群，并学习如何几乎完全自动化使用 Terraform 功能部署 Docker Swarm 集群。

第十三章，“单元测试和功能测试”，让我们探索三种测试模式：单元测试、REST 测试和功能测试。我们将使用流行的测试框架 Mocha 和 Chai 来驱动这三种模式的测试用例。对于功能测试，我们将使用 Puppeteer，这是一个在 Chrome 实例中自动化测试执行的流行框架。

第十四章，“Node.js 应用程序中的安全性”，是我们集成安全技术和工具以减轻安全入侵的地方。我们将首先在 AWS EC2 部署中使用 Let's Encrypt 实现 HTTPS。然后，我们将讨论 Node.js 中的几种工具来实现安全设置，并讨论 Docker 和 AWS 环境的最佳安全实践。

# 为了充分利用本书

基本要求是安装 Node.js 并拥有面向程序员的文本编辑器。编辑器不必太复杂；即使是 vi/vim 也可以。我们将向您展示如何安装所需的一切，而且这些都是开源的，因此没有任何准入障碍。

最重要的工具是您的大脑，我们指的不是耳屎。

| **书中涵盖的软件/硬件** **操作系统要求** |
| --- |
| Node.js 及相关框架，如 Express、Sequelize 和 Socket.IO |
| 使用`npm`/`yarn`软件包管理工具 |
| Python 和 C/C++编译器 |
| MySQL、SQLite3 和 MongoDB 数据库 |
| Docker |
| Multipass |
| Terraform |
| Mocha 和 Chai |

每个涉及的软件都是 readily available。对于 Windows 和 macOS 上的 C/C++编译器，您需要获取 Visual Studio（Windows）或 Xcode（macOS），但两者都是免费提供的。

如果您已经有一些 JavaScript 编程经验，这将会很有帮助。如果您已经有其他编程语言的经验，学习它会相当容易。

## 下载示例代码文件

尽管我们希望书中和存储库中的代码片段是相同的，但在某些地方可能会有细微差异。存储库中可能包含书中未显示的注释、调试语句或替代实现（已注释掉）。

您可以从[www.packt.com](http://www.packt.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[www.packtpub.com/support](https://www.packtpub.com/support)并注册，文件将直接发送到您的邮箱。

您可以按照以下步骤下载代码文件：

1.  在[www.packt.com](http://www.packt.com)登录或注册。

1.  选择支持选项卡。

1.  单击代码下载。

1.  在搜索框中输入书名，然后按照屏幕上的说明操作。

下载文件后，请确保使用以下最新版本的软件解压或提取文件夹：

+   Windows 上的 WinRAR/7-Zip

+   Mac 上的 Zipeg/iZip/UnRarX

+   Linux 上的 7-Zip/PeaZip

该书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Node.js-Web-Development-Fifth-Edition`](https://github.com/PacktPublishing/Node.js-Web-Development-Fifth-Edition)。如果代码有更新，将在现有的 GitHub 存储库上进行更新。

我们还有来自丰富书籍和视频目录的其他代码包，可在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**上找到。快去看看吧！

## 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码字、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名。例如："首先更改`package.json`，使其具有以下`scripts`部分。"

代码块设置如下：

```

When we wish to draw your attention to a particular part of a code block, the relevant lines or items are set in bold:

```

任何命令行输入或输出都以以下形式编写：

```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词会在文本中以这种方式出现。例如："单击提交按钮。"

警告或重要说明看起来像这样。

提示和技巧看起来像这样。
