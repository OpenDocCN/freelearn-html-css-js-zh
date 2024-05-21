# 前言

## 关于

本节简要介绍了作者、本书的内容、开始所需的技术技能，以及完成所有包含的活动和练习所需的硬件和软件。

## 关于本书

JavaScript 是 Web 技术的核心编程语言，可用于修改 HTML 和 CSS。它经常被缩写为 JS。JavaScript 经常用于大多数 Web 浏览器的用户界面中进行的处理，如 Internet Explorer，Google Chrome 和 Mozilla Firefox。由于其使浏览器能够完成工作的能力，它是当今最广泛使用的客户端脚本语言。

在本书中，您将深入了解 JavaScript。您将学习如何在 ES6 中使用新的 JavaScript 语法在专业环境中编写 JavaScript，如何利用 JavaScript 的异步特性使用回调和承诺，以及如何设置测试套件并测试您的代码。您将了解 JavaScript 的函数式编程风格，并将所学的一切应用于使用各种 JavaScript 框架和库构建简单应用程序的后端和前端开发。

### 关于作者

**Zachary Shute**在 RPI 学习计算机和系统工程。他现在是位于加利福尼亚州旧金山的一家机器学习初创公司的首席全栈工程师。对于他的公司 Simple Emotion，他管理和部署 Node.js 服务器，MongoDB 数据库以及 JavaScript 和 HTML 网站。

### 目标

+   检查 ES6 中的主要功能，并实现这些功能来构建应用程序

+   创建承诺和回调处理程序以处理异步进程

+   使用 Promise 链和 async/await 语法开发异步流

+   使用 JavaScript 操作 DOM

+   处理 JavaScript 浏览器事件

+   探索测试驱动开发，并使用 JavaScript 代码测试框架构建代码测试。

+   列出函数式编程与其他风格相比的优缺点

+   使用 Node.js 后端框架和 React 前端框架构建应用程序

### 受众

本书旨在针对任何希望在专业环境中编写 JavaScript 的人群。我们期望受众在某种程度上使用过 JavaScript，并熟悉基本语法。本书适合技术爱好者，想知道何时使用生成器或如何有效地使用承诺和回调，或者想加深对 JavaScript 的了解和理解 TDD 的初学开发人员。

### 方法

这本书以易于理解的方式全面解释了技术，同时完美地平衡了理论和练习。每一章都设计为在前一章所学内容的基础上构建。本书包含多个活动，使用真实的商业场景让您练习并应用新技能，使之具有高度相关性。

### 最低硬件要求

为了获得最佳的学生体验，我们建议以下硬件配置：

+   处理器：Intel Core i5 或同等处理器

+   内存：4 GB RAM

+   存储：35 GB 可用空间

+   互联网连接

### 软件要求

您还需要提前安装以下软件：

+   操作系统：Windows 7 SP1 64 位，Windows 8.1 64 位或 Windows 10 64 位

+   Google Chrome ([`www.google.com/chrome/`](https://www.google.com/chrome/))

+   Atom IDE ([`atom.io/`](https://atom.io/))

+   Babel ([`www.npmjs.com/package/babel-install`](https://www.npmjs.com/package/babel-install))

+   Node.js 和 Node Package Manager（npm）([`nodejs.org/en/`](https://nodejs.org/en/))

安装说明可以单独提供给大型培训中心和组织。所有源代码都可以在 GitHub 上公开获取，并在培训材料中得到完全引用。

### 安装代码包

将课程的代码包复制到`C:/Code`文件夹中。

### 额外资源

本书的代码包也托管在 GitHub 上，网址为[`github.com/TrainingByPackt/Advanced-JavaScript`](https://github.com/TrainingByPackt/Advanced-JavaScript)。

我们还有来自丰富书籍和视频目录的其他代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)上找到。去看看吧！

### 约定

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄都显示为以下方式："JavaScript 中声明变量的三种方式：`var`、`let`和`const`。"

代码块设置如下：

```php
var example; // Declare variable
example = 5; // Assign value
console.log( example ); // Expect output: 5
```

任何命令行输入或输出都以以下方式编写：

```php
npm install babel --save-dev
```

新术语和重要单词以粗体显示。您在屏幕上看到的单词，例如菜单或对话框中的单词，会以这种方式出现在文本中："这意味着使用块作用域创建的变量受到**时间死区（TDZ）**的影响。"

### 安装 Atom IDE

1.  要安装 Atom IDE，请在浏览器中转到[`atom.io/`](https://atom.io/)。

1.  点击**下载 Windows 安装程序**以下载名为**AtomSetup-x64.exe**的设置文件。

1.  运行可执行文件。

1.  将`atom`和`apm`命令添加到您的路径中。

1.  在桌面和开始菜单上创建快捷方式。

Babel 会安装到每个代码项目的本地。要在 NodeJs 项目中安装 Babel，请完成以下步骤：

1.  打开命令行界面并导航到项目文件夹。

1.  运行命令`npm init`。

1.  填写所有必填问题。如果您不确定任何提示的含义，可以按“enter”键跳过问题并使用默认值。

1.  运行`npm install --save-dev babel-cli`命令。

1.  运行命令`install --save-dev babel-preset-es2015`。

1.  验证`package.json`中的`devDependencies`字段是否包含`babel-cli`和`babel-presets-es2015`。

1.  创建一个名为`.babelrc`的文件。

1.  在文本编辑器中打开此文件并添加代码`{ "presets": ["es2015"] }`。

### 安装 Node.js 和 npm

1.  要安装 Node.js，请在浏览器中转到[`nodejs.org/en/`](https://nodejs.org/en/)。

1.  点击**下载 Windows（x64）**，以下载推荐给大多数用户的 LTS 设置文件，名为`node-v10.14.1-x64.msi`。

1.  运行可执行文件。

1.  确保在安装过程中选择 npm 软件包管理器捆绑包。

1.  接受许可证和默认安装设置。

1.  重新启动计算机以使更改生效。
