# 前言

*《学习 JavaScriptMVC》* 将引导您了解所有框架方面，并展示如何构建小型到中型、结构良好且文档齐全的客户端应用程序，您将喜欢在它们上工作。

# 本书涵盖的内容

第一章，*JavaScriptMVC 入门*，提供了 JavaScriptMVC 框架的概述。安装它，了解其架构，并学习以最佳方式完成它——通过构建一个简单的应用程序。

第二章，*DocumentJS*，展示了尽管功能强大，但 DocumentJS 是一个简单的工具，旨在轻松创建任何 JavaScript 代码库的可搜索文档。

第三章，*FuncUnit*，解释了 FuncUnit 是一个具有类似 jQuery 语法的功能测试框架。使用 FuncUnit，我们可以在所有现代网络浏览器中运行测试。编写测试既简单又快捷。

第四章，*jQueryMX*，展示了 jQueryMX 是一个提供实现和组织大型 JavaScript 应用程序所需功能的 jQuery 库集合。它提供了经典继承模拟和模型-视图-控制器层，以提供逻辑上分离的代码库。

第五章，*StealJS*，展示了 StealJS 是一个独立的代码管理和构建工具。

第六章，*构建应用程序*，展示了如何从概念到设计、实现、文档和测试构建实际应用程序。

# 本书所需的条件

要运行本书中的示例，需要以下软件：

+   **JavaScriptMVC 启动器**：[`github.com/wbednarski/JavaScriptMVC_kick-starter`](https://github.com/wbednarski/JavaScriptMVC_kick-starter)

+   **Oracle VM VirtualBox**：[`www.virtualbox.org/`](https://www.virtualbox.org/)

+   **Vagrant**：[`downloads.vagrantup.com/`](http://downloads.vagrantup.com/)

# 本书面向的对象

本书面向任何对使用基于最流行的 JavaScript 库——jQuery 的 JavaScriptMVC 框架开发小型到中型网络应用程序感兴趣的人。

读者应熟悉 JavaScript、浏览器 API、jQuery、HTML5 和 CSS。

# 习惯用法

在本书中，您将找到不同样式的文本，以区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码词如下所示："通过`checkout`标签轻松切换到另一个版本。"

代码块如下设置：

```js
<!doctype html>

<html>
    <head>
        <title>Todo List</title>
        <meta charset="UTF-8" />
    </head>
    <body>
        <ul id="todos">
            <li>all done!</li>
        </ul>

        <script src="img/steal.js?todo"></script>
    </body>
</html>
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```js
steal(
    'jquery/class',
    'jquery/model',
 'jquery/dom/fixture',
 'jquery/view/ejs',
    'jquery/controller',
    'jquery/controller/route',

    function ($) {

    }
);
```

任何命令行输入或输出都如下所示：

```js
$ git submodule add git://github.com/bitovi/steal.git
$ git submodule add git://github.com/bitovi/documentjs.git
$ git submodule add git://github.com/bitovi/funcunit.git
$ git submodule add git://github.com/jupiterjs/jquerymx jquery

```

**新术语**和**重要词汇**以粗体显示。您在屏幕上看到的单词，例如在菜单或对话框中，在文本中显示如下：“**存档**按钮在任务悬停时可见。”

### 注意

警告或重要提示以这样的框出现。

### 小贴士

小贴士和技巧看起来像这样。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们您对这本书的看法——您喜欢什么或可能不喜欢什么。读者反馈对我们开发您真正能从中受益的标题非常重要。

要发送一般反馈，只需发送电子邮件到<mailto:feedback@packtpub.com>，并在邮件主题中提及书籍标题。

如果您在某个主题上具有专业知识，并且您对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您已经是 Packt 图书的骄傲拥有者了，我们有许多事情可以帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)的账户下载您购买的所有 Packt 图书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)，并注册以将文件直接通过电子邮件发送给您。

## 勘误

尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站，或添加到该标题的勘误部分下的现有勘误列表中。任何现有勘误都可以通过从[`www.packtpub.com/support`](http://www.packtpub.com/support)选择您的标题来查看。

## 侵权

互联网上版权材料的侵权是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在网上遇到我们作品的任何非法副本，无论形式如何，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过链接<mailto:copyright@packtpub.com>与我们联系，并提供涉嫌侵权材料的链接。

我们感谢您在保护我们的作者和我们为您提供有价值内容的能力方面的帮助。

## 问题

如果你在本书的任何方面遇到问题，可以通过 `<questions@packtpub.com>` 联系我们，我们将尽力解决。
