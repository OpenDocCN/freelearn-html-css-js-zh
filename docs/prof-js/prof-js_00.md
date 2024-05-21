# 第一章：*前言*

## 关于

本节简要介绍了作者、本书的内容、开始所需的技术技能，以及完成所有包含的活动和练习所需的硬件和软件要求。

## 关于本书

深入了解 JavaScript，更容易学习其他框架，包括 React、Angular 和相关工具和库。本书旨在帮助您掌握构建现代应用程序所需的核心 JavaScript 概念。

您将首先学习如何在文档对象模型（DOM）中表示 HTML 文档。然后，您将结合对 DOM 和 Node.js 的知识，为实际情况创建一个网络爬虫。随着您阅读更多章节，您将使用 Express 库为 Node.js 创建基于 Node.js 的 RESTful API。您还将了解如何使用模块化设计来实现更好的可重用性，并与多个开发人员在单个项目上进行协作。后面的章节将指导您构建单元测试，以确保程序的核心功能不会随时间而受到影响。本书还将演示构造函数、async/await 和事件如何快速高效地加载您的应用程序。最后，您将获得有关不可变性、纯函数和高阶函数等函数式编程概念的有用见解。

通过本书，您将掌握使用现代 JavaScript 方法解决客户端和服务器端的任何真实世界 JavaScript 开发问题所需的技能。

### 关于作者

**雨果·迪弗朗西斯科（Hugo Di Francesco）**是一名软件工程师，他在 JavaScript 方面有丰富的经验。他拥有伦敦大学学院（UCL）的数学计算工程学士学位。他曾在佳能和 Elsevier 等公司使用 JavaScript 创建可扩展和高性能的平台。他目前正在使用 Node.js、React 和 Kubernetes 解决零售运营领域的问题，同时运营着同名的 Code with Hugo 网站。工作之外，他是一名国际击剑运动员，他在全球范围内进行训练和比赛。

**高思远（Siyuan Gao）**是艺电公司的软件工程师。他拥有普渡大学的计算机科学学士学位。他已经使用 JavaScript 和 Node.js 超过 4 年，主要为高可用性系统构建高效的后端解决方案。他还是 Node.js 核心项目的贡献者，并且已经发布了许多 npm 模块。在业余时间，他喜欢学习视频游戏设计和机器学习。

**Vinicius Isola**于 1999 年开始使用 Macromedia Flash 和 ActionScript 进行编程。2005 年，他获得了 Java 认证，并专门从事构建 Web 和企业应用程序。JavaScript 和 Web 技术一直在他的许多工作角色和所在公司中发挥作用。在业余时间，他喜欢参与开源项目并指导新开发人员。

菲利普·柯克布赖德（Philip Kirkbride）在蒙特利尔拥有超过 5 年的 JavaScript 经验。他于 2011 年从技术学院毕业，自那时起一直在不同的角色中使用 Web 技术。他曾与 2Klic 合作，这是一家由主要电暖公司 Convectair 承包的物联网公司，用 Z-Wave 技术创建智能加热器。他的角色包括在 Node.js 和 Bash 中编写微服务。他还有机会为开源项目 SteemIt（基于区块链的博客平台）和 DuckDuckGo（基于隐私的搜索引擎）做出一些贡献。

### 学习目标

通过本书，您将能够：

+   应用函数式编程的核心概念

+   构建一个使用 Express.js 库托管 API 的 Node.js 项目

+   为 Node.js 项目创建单元测试以验证其有效性

+   使用 Cheerio 库与 Node.js 创建基本网络爬虫

+   开发一个 React 界面来构建处理流程

+   使用回调作为将控制权带回的基本方法

### 受众

如果您想从前端开发人员转变为全栈开发人员，并学习 Node.js 如何用于托管全栈应用程序，那么这本书非常适合您。阅读本书后，您将能够编写更好的 JavaScript 代码，并了解语言中的最新趋势。为了轻松掌握这里解释的概念，您应该了解 JavaScript 的基本语法，并且应该使用过流行的前端库，如 jQuery。您还应该使用过 JavaScript 与 HTML 和 CSS，但不一定是 Node.js。

### 方法

本书的每一部分都经过明确设计，旨在吸引和激发您，以便您可以在实际环境中保留和应用所学知识，产生最大的影响。您将学习如何应对具有智力挑战的编程问题，这将通过函数式编程和测试驱动开发实践为您准备真实世界的主题。每一章都经过明确设计，以 JavaScript 作为核心语言进行构建。

### 硬件要求

为了获得最佳体验，我们建议以下硬件配置：

+   处理器：Intel Core i5 或同等级处理器

+   内存：4GB RAM

+   存储：5GB 可用空间

### 软件要求

我们还建议您提前安装以下软件：

+   Git 最新版本

+   Node.js 10.16.3 LTS ([`nodejs.org/en/`](https://nodejs.org/en/))

### 约定

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄显示如下：

"ES6 的`import`函数还允许您导入模块的子部分，而不是整个模块。这是 ES6 的`import`比 Node.js 的`require`函数更强大的功能。SUSE"

代码块设置如下：

```js
let myString = "hello";
console.log(myString.toUpperCase()); // returns HELLO
console.log(myString.length); // returns 5
```

### 安装和设置

在我们可以使用数据做出了不起的事情之前，我们需要准备好最高效的环境。在这个简短的部分中，我们将看到如何做到这一点。

### 安装 Node.js 和 npm

Node.js 的安装包中包含 npm（Node.js 的默认包管理器）。

**在 Windows 上安装 Node.js**：

1.  在[`nodejs.org/en/download/current/`](https://nodejs.org/en/download/current/)官方安装页面上找到您想要的 Node.js 版本。

1.  确保选择 Node.js 12（当前版本）。

1.  确保您为计算机系统安装了正确的架构；即 32 位或 64 位。您可以在操作系统的**系统属性**窗口中找到这些信息。

1.  下载安装程序后，只需双击文件，然后按照屏幕上的用户友好提示操作即可。

**在 Linux 上安装 Node.js 和 npm**：

在 Linux 上安装 Node.js，您有几个不错的选择：

+   要在未详细介绍的系统上通过 Linux 软件包管理器安装 Node.js，请参阅[`nodejs.org/en/download/package-manager/`](https://nodejs.org/en/download/package-manager)。

+   要在 Ubuntu 上安装 Node.js，请运行此命令（更多信息和手动安装说明可在[`github.com/nodesource/distributions/blob/master/README.md#installation-instructions`](https://github.com/nodesource/distributions/blob/master/README.md#installation-instructions)找到）：

```js
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt-get install -y nodejs
```

+   要在基于 Debian 的发行版上安装 Node.js（更多信息和手动安装说明可在[`github.com/nodesource/distributions/blob/master/README.md#installation-instructions`](https://github.com/nodesource/distributions/blob/master/README.md#installation-instructions)找到）：

```js
# As root
curl -sL https://deb.nodesource.com/setup_12.x | bash -
apt-get install -y nodejs
```

+   官方 Node.js 安装页面还提供了一些 Linux 系统的其他安装选项：[`nodejs.org/en/download/current/`](https://nodejs.org/en/download/current)。

**在 macOS 上安装 Node.js 和 npm**：

与 Linux 类似，Mac 上安装 Node.js 和 npm 有几种方法。要在 macOS X 上安装 Node.js 和 npm，请执行以下操作：

1.  按下*cmd + Spacebar*打开 Mac 的终端，输入`terminal`并按下*Enter*。

1.  通过运行`xcode-select --install`命令行来安装 Xcode。

1.  安装 Node.js 和 npm 的最简单方法是使用 Homebrew，通过运行`ruby -e "$(curl -fsSL` ([`raw.githubusercontent.com/Homebrew/install/master/install`](https://raw.githubusercontent.com/Homebrew/install/master/install))来安装 Homebrew。

1.  最后一步是安装 Node.js 和 npm。在命令行上运行`brew install node`。

1.  同样，您也可以通过[`nodejs.org/en/download/current/`](https://nodejs.org/en/download/current/)提供的安装程序安装 Node.js 和 npm。

**安装 Git**

安装 git，请前往[`git-scm.com/downloads`](https://git-scm.com/downloads)，并按照针对您平台的说明进行操作。

### 其他资源

本书的代码包也托管在 GitHub 上，网址为[`github.com/TrainingByPackt/Professional-JavaScript`](https://github.com/TrainingByPackt/Professional-JavaScript)。我们还有来自丰富书籍和视频目录的其他代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing)找到。快去看看吧！
