# 前言

在过去的几年里，Node.js 已经进入到了财富 500 强公司的技术栈、以移动为先的初创公司、成功的互联网企业以及其他企业的技术栈中。除了验证其作为平台的力量外，这种成功也暴露了管理、部署和监控 Node.js 应用程序的工具短缺。尽管 Node.js 社区是开放和协作的，但关于专业 Node.js 开发者如何设计、测试并将代码推送到生产环境中的全面信息仍然难以找到。

本书试图通过解释和记录正在工作的 Node.js 开发者可以使用的技术和工具来填补这一知识差距，以创建可扩展、智能、健壮和可维护的企业级软件。

在简要介绍 Node 及其设计背后的哲学之后，你将学习如何在本地服务器和云中安装和更新你的应用程序。然后，通过逐步进行负载均衡和其他扩展技术，解释如何处理流量量的突然变化和形状，实施版本控制，并设计内存高效、有状态、分布式应用程序。

一旦你完成了创建生产就绪系统的必要基础工作，你将需要测试和部署它们。本书充满了真实的代码示例，结构上像一个逐步的工作手册，解释了在测试、部署、监控和维护每个成功的软件运行马拉松的每个阶段应使用的成功策略。

当你完成这本书后，你将学会一套可直接应用于解决你今天所面临问题的可重用模式，并为如何构建和部署你的下一个项目奠定信心基础。

# 本书涵盖内容

第一章，*欣赏 Node*，深入探讨了 Node.js 背后的思考，帮助你清晰地思考 Node.js 擅长什么，不擅长什么，以及如何利用 Node.js 解决现实世界的挑战。

第二章，*安装和虚拟化 Node 服务器*，将教你如何创建一个基本的 Node.js 应用程序并在服务器上运行它。你还将学习如何在某些流行的云托管提供商上执行相同操作，例如 DigitalOcean 和 Heroku，此外，你还将学习如何使用 Docker 创建轻量级、易于复制的虚拟机。

第三章，*扩展 Node*，探讨了垂直和水平扩展技术。你将学习如何使用集群模块在单个服务器上最大化 Node 的效率，以及如何协调多个分布式 Node.js 服务器来处理增加的网络流量，了解 Nginx 负载均衡、使用消息队列设置代理以及在过程中协调进程间通信。

第四章，*管理内存和空间*，展示了良好的工程实践永远不会过时。我们首先讨论微服务，介绍设计由小型、专注、通信进程组成的系统的通用技术。接下来，深入探讨如何优化 JavaScript，特别是针对 V8 编译器。从几个在 Redis 中高效存储和检索事件数据的示例开始，我们研究缓存策略、会话管理和使用 CDN 来减少服务器负载。

第五章，*监控应用程序*，解释了在应用程序部署后如何有效地监控应用程序的策略。包括使用各种第三方监控工具的示例，以及概述如何构建自己的自定义系统采样和日志模块的示例。最后，我们探讨调试技术，检查几个帮助您找到和防止运行时瓶颈的工具和策略。

第六章，*构建和测试*，介绍了创建应用程序构建管道时的一些考虑因素。提供了使用 Gulp、Browserify 和 npm 创建构建工具的完整示例，以及使用 Mocha 进行测试、使用 Sinon 进行模拟和利用 PhantomJS 进行无头浏览器测试的信息。

第七章，*部署和维护*，指导您了解整个部署管道，从设置虚拟化开发环境到将持续集成集成到您的流程中。您将了解如何使用 GitHub webhooks 和 Vagrant，以及如何使用 Jenkins 自动化您的部署过程。此外，npm 软件包管理器将被彻底剖析，并讨论依赖关系管理策略。

# 你需要为此书准备的内容

你需要安装 Node v. 0.12.5 或更高版本，最好是在基于 Unix 的操作系统上，例如 Linux 或 Mac OS X。你还需要安装一些工具，主要是为了设置开发和部署示例：

+   使用 Node 版本管理器 (nvm) 安装 Node.js (以及 npm): [`github.com/creationix/nvm`](https://github.com/creationix/nvm)

+   Git: [`git-scm.com/book/en/Getting-Started-Installing-Git`](http://git-scm.com/book/en/Getting-Started-Installing-Git)

+   Redis: [`redis.io/topics/quickstart`](http://redis.io/topics/quickstart)

+   MongoDB: [`docs.mongodb.org/manual/installation/`](http://docs.mongodb.org/manual/installation/)

+   Nginx: [`wiki.nginx.org/Install`](http://wiki.nginx.org/Install)

+   Docker: [`docs.docker.com/installation/`](https://docs.docker.com/installation/)

+   PhantomJS: [`phantomjs.org/download.html`](http://phantomjs.org/download.html)

+   Jenkins: [`wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins`](https://wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins)

+   Vagrant: [`docs.vagrantup.com/v2/installation/`](http://docs.vagrantup.com/v2/installation/)

+   此外，以下 npm 包必须全局安装（npm install <packagename> -g）：

    +   gulp

    +   pm2

    +   mocha

在您阅读本书的过程中，如有必要，将提供这些和其他包的进一步安装和配置说明。

# 本书面向的对象

这本书是为那些准备在生产环境中部署大型 Node.js 应用程序的 Node.js 开发者设计的。它旨在通过将示例置于现实情境中，关注模块化设计，并使用广泛的测试、主动监控和以团队为中心的维护策略，更详细地向中级 Node.js 开发者介绍该平台。那些希望提高他们编写的 JavaScript/Node 程序的质量和效率，并交付能够承受企业级流量的稳健系统的人，会喜欢这本书。没有 Node.js 平台经验的 DevOps 工程师也将从 Node.js 社区如何实施他们已知的技术的宝贵信息中受益。

# 约定

在本书中，您将找到许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称如下所示：“因此，一旦我们加载了页面，我们就将按键（`sendKeys`）发送到`#source`输入框中，输入意大利语单词`"Ciao"`。”

代码块设置如下：

```js
variable = produceAValue()
print variable
// some value is output when #produceAValue is finished.
```

任何命令行输入或输出都应如下编写：

```js
> This happens first
> Then the contents are available, [file contents shown]

```

**新术语**和**重要词汇**以粗体显示。屏幕上显示的单词，例如在菜单或对话框中，在文本中显示如下：“你应该看到**您刚刚部署了一些 Node!**显示。”

### 注意

警告或重要注意事项以如下方式显示：

### 提示

技巧和窍门看起来像这样。

# 读者反馈

我们欢迎读者的反馈。让我们知道您对这本书的看法——您喜欢或不喜欢什么。读者反馈对我们很重要，因为它帮助我们开发出您真正能从中获得最大收益的标题。

要向我们发送一般反馈，只需发送电子邮件至 `<feedback@packtpub.com>`，并在邮件主题中提及本书的标题。

如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请参阅我们的作者指南，链接为[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您已经成为 Packt 书籍的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)的账户下载所有已购买 Packt 出版物的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 错误清单

尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何错误清单，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**错误提交表单**链接，并输入您的错误详细信息来报告它们。一旦您的错误得到验证，您的提交将被接受，错误将被上传到我们的网站或添加到该标题的错误清单部分。

要查看之前提交的错误清单，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将出现在**错误清单**部分。

## 盗版

在互联网上盗版受版权保护的材料是一个跨所有媒体的持续问题。在 Packt，我们非常重视保护我们的版权和许可证。如果您在互联网上发现任何形式的我们作品的非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过发送电子邮件到`<copyright@packtpub.com>`并附上疑似盗版材料的链接联系我们。

我们感谢您在保护我们作者和我们为您提供有价值内容的能力方面所提供的帮助。

## 询问

如果您在这本书的任何方面遇到问题，您可以通过发送电子邮件到`<questions@packtpub.com>`联系我们，我们将尽力解决问题。
