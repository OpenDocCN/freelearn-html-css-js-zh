# 前言

《HTML5 for Flash Developers》专门为准备立即投入 HTML5 开发的 Flash 开发人员编写。我们将首先分析组成 HTML5 的每个元素，然后开始学习如何通过比较它们的特性与典型的 Flash 开发来利用它们。

# 本书涵盖的内容

第一章《为什么学习 HTML5？》开始回答了为什么学习如何在 HTML5 中开发可以成为一项非常重要的技能。我们将继续全面概述组成 HTML5 的所有不同技术以及它们的利用方式。

第二章《为战斗做准备》涵盖了为网络准备图像、音频和视频等资产的重要过程。本章还涵盖了 JavaScript 开发的许多重要方面，以及它们与 ActionScript 3 的不同之处。

第三章《可伸缩性、限制和效果》旨在告诉您 HTML5 开发人员在今天为网络开发时经常遇到的许多限制。我们还将首次详细了解最新的层叠样式表 CSS3 的添加。最后，我们将通过查看 HTML5 Web Workers、Web Sockets、Canvas 元素和最终的 WebGL 来概述 HTML 开发的一些最有趣的新添加。

第四章《使用 HTML5 构建强大的应用程序》继续深入研究 JavaScript 开发，旨在以面向对象的方式构建代码。还将进行与 ActionScript 3 类结构、用法和语法的比较，以及一些库和框架的概述，以帮助构建强大的 HTML5 应用程序。

第五章《一次编码，到处发布》探讨了使开发人员能够轻松地针对多个平台进行单个应用程序构建的应用程序和代码库，最大限度地减少开发时间并最大化公众使用。我们将花费大部分时间深入研究 CreateJS 框架及其所有包。最后，我们将介绍 CSS3 媒体查询如何允许在各种屏幕上进行有针对性的元素样式设置。

第六章《HTML5 框架和库》继续深入挖掘在开发下一个 HTML5 应用程序时可用的各种令人惊叹的框架和库。我们首先查看了当今最受欢迎的库之一，即 jQuery。除了 jQuery JavaScript 库，我们还将看看 jQuery Mobile 项目如何将简单的 HTML 文档转换为移动友好的用户界面。最后，我们将查看其他开源项目，如 Google V8、Node.js 和 Three.js。

第七章《选择开发方式》探讨了许多适用于 HTML5 开发人员的流行代码编辑平台。我们将概述大多数开发人员在为网络开发时从编码环境中需要的绝对必需品。我们还将花一些时间了解 Adobe Edge Animate 平台，该平台为创建 HTML5 动画提供了类似 Flash 的用户界面。

第八章《导出到 HTML5》继续查看更多允许您在平台上编写 HTML5 应用程序并直接编译为 HTML5 的软件。我们将概述许多流行的导出到 HTML5 的平台，如 Jangaroo、Haxe 和 Google 的 Dart。

第九章，*避免障碍*，试图展示和讨论许多开发人员在使用 HTML5 的许多新添加时所面临的典型问题。在本章中，我们将开发一个简单的 2D 横向滚动游戏，并检查常见问题发生的位置。

第十章，*发布准备*，通过讨论通常在将 HTML5 应用程序发布到互联网之前执行的许多常见任务，结束了本书。我们将讨论适当的浏览器测试方法以及利用“夜间”网络浏览器版本。我们将讨论许多方法，通过外部应用程序和浏览器插件对 HTML5 内容进行基准测试，以便您检查运行时问题。最后，我们将讨论通过使用 Grunt 等应用程序自动化 Web 开发人员反复执行的流程的方法。

# 您需要为本书准备什么

为了完全理解本书，需要以下软件：

+   符合 HTML5 标准的网络浏览器（Google Chrome，Firefox，Opera 等）。

+   HTML5 友好的文本编辑器（Sublime，Dreamweaver，Aptana 和 Adobe Brackets）

+   访问 Adobe Creative Cloud [`creative.adobe.com/`](https://creative.adobe.com/)

+   Adobe Flash

+   CreateJS 工具包[`www.adobe.com/ca/products/flash/flash-to-html5.html`](http://www.adobe.com/ca/products/flash/flash-to-html5.html)

访问互联网以下载最新版本的开源库和框架。

# 本书的受众

本书专门针对有 Adobe Flash 网页应用程序和游戏开发经验，准备将 HTML5 开发添加到其技能组合中的开发人员。

# 约定

在本书中，您会发现一些文本样式，用于区分不同类型的信息。以下是一些这些样式的示例，以及它们的含义解释。

文本中的代码词如下所示："我们可以通过使用`include`指令来包含其他上下文。"

代码块设置如下：

```html
  this.setX = function(x) { _xVal = x; }
  this.setY = function(y) { _yVal = y; }
  this.currentX = function() { return _xVal; }
  this.currentY = function() { return _yVal; }
  this.currentWidth = function() { return _widthVal; }
  this.currentHeight = function() { return _heightVal; }
```

当我们希望引起您对代码块的特别关注时，相关的行或项目将以粗体显示：

```html
  this.setX = function(x) { _xVal = x; }
  this.setY = function(y) { _yVal = y; }
 this.currentX = function() { return _xVal; }
 this.currentY = function() { return _yVal; }
 this.currentWidth = function() { return _widthVal; }
 this.currentHeight = function() { return _heightVal; }

```

**新术语**和**重要单词**以粗体显示。例如，屏幕上看到的单词，菜单或对话框中的单词等，会在文本中以这种方式出现："单击**下一步**按钮将您移至下一个屏幕"。

### 注意

警告或重要说明会以这样的方式出现在框中。

### 提示

提示和技巧会以这种方式出现。
