# 前言

JavaScript 网络应用程序的受欢迎程度正在飙升，并在互联网上推动着令人兴奋的新应用程序可能性。引领这一潮流的最普遍的框架之一是 Backbone.js，它为组织 JavaScript 应用程序提供了一种现代和理性的方法。

同时，测试客户端 JavaScript 和 Backbone.js 应用程序仍然是一项困难且繁琐的任务。即使是经验丰富的开发者，在编写前端测试时也可能遇到与浏览器特性、复杂的 DOM 交互和异步应用程序行为相关的问题。

*Backbone.js 测试* 将合理的实践和当前技术带到了 Backbone.js 测试开发的挑战中。你将了解基本的测试概念、当代前端测试基础设施以及 Backbone.js 应用程序开发各个方面的实际练习。本书涵盖了从创建基本测试套件到使用测试双库来解决甚至最困难/最不易测试的 Backbone.js 应用程序组件等主题。

在本书的一些建议指导下，你可以轻松、快速、有信心地测试你的 Backbone.js 网络应用程序。

# 本书涵盖的内容

第一章, *设置测试基础设施*，从如何设置你的测试应用程序代码以及获取我们将在这本书中使用的测试库的基本知识开始。我们创建了一个基本的测试基础设施，编写了第一个测试，并回顾了测试报告结果。

第二章, *创建 Backbone.js 应用程序测试计划*，从 Backbone.js 基础知识的复习开始，介绍了一个本书的示例网络应用程序，并讨论了广泛的与测试和规划相关的概念。我们通过编写和运行我们的第一个 Backbone.js 应用程序测试来结束。

第三章, *测试断言、规范和套件*，涵盖了使用 Mocha 编写 Backbone.js 测试套件和规范以及使用 Chai 进行测试断言的基本知识。

第四章, *测试间谍*，介绍了 Sinon.JS 测试双库以及如何在 Backbone.js 测试中监视应用方法行为。

第五章, *测试存根和模拟*，深入探讨了 Sinon.JS，其中包括可以替换应用程序方法行为的存根和模拟。我们检查了存根和模拟如何减少测试中的应用程序依赖，并简化了 Backbone.js 应用程序测试。

第六章, *自动化网络测试*，增强了前几章中构建的测试基础设施，使其能够自动运行，例如，从命令行或持续集成服务器。

# 本书面向的对象

本书面向希望为 Backbone.js 网络应用创建和实现测试支持的 JavaScript 开发者。你应该熟悉 JavaScript 编程语言，并了解 Backbone.js 应用开发，包括模型、视图和路由等核心组件，尽管你可能在学习本书的测试主题时刚开始学习这个框架。对测试方法和技术的了解（任何语言）将有所帮助，但不是必需的。

# 习惯用法

在本书中，你会找到多种文本样式，用于区分不同类型的信息。以下是一些这些样式的示例，以及它们含义的解释。

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称将按照以下方式显示：“我们使用原生的 JavaScript 函数`setTimeout()`来模拟慢速测试。”

代码块按照以下方式设置：

```js
describe("Test failures", function () {
  it("should fail on assertion", function () {
    expect("hi").to.equal("goodbye");
  });
});
```

当我们希望引起你对代码块中特定部分的注意时，相关的行或项目将以粗体显示：

```js
describe("Test failures", function () {
  it("should fail on assertion", function () {
    expect("hi").to.equal("goodbye");
  });
});
```

任何命令行输入或输出都按照以下方式编写：

```js
$ mocha-phantomjs chapters/05/test/test.html

```

**新术语**和**重要词汇**将以粗体显示。屏幕上显示的词，例如在菜单或对话框中，在文本中将像这样显示：“点击**下一步**按钮将您带到下一屏幕”。

### 注意

警告或重要注意事项将显示在这个框中。

### 小贴士

小技巧和技巧将像这样显示。

# 读者反馈

我们欢迎读者的反馈。请告诉我们您对本书的看法——您喜欢什么或可能不喜欢什么。读者反馈对我们开发您真正能从中获得最大价值的标题非常重要。

要向我们发送一般反馈，只需发送电子邮件到 `<feedback@packtpub.com>`，并在邮件主题中提及书名。

如果您在某个主题上具有专业知识，并且对撰写或为本书做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在，您已经成为 Packt 图书的骄傲拥有者，我们有一些可以帮助您充分利用购买的东西。

## 下载示例代码

本书所有示例和文件的源代码都可在 GitHub 仓库([`github.com/ryan-roemer/backbone-testing/`](https://github.com/ryan-roemer/backbone-testing/))中找到，并在[`backbone-testing.com`](http://backbone-testing.com)网站上详细介绍。该[`backbone-testing.com`](http://backbone-testing.com)网站将始终包含获取和使用本书代码示例的最新和更新说明。

### 注意

代码示例库内部使用符号链接来引用某些库和文件。因此，Windows 用户可能需要从 Packt 下载示例存档（请参阅以下说明），而不是从 GitHub 下载。

由于这是一个开源项目，示例可能会定期更新以修复错误或澄清代码或概念。因此，书中的代码片段可能不会与在线代码样本完全匹配，但在实践中不应有太大差异。最终，您可以依赖 GitHub 仓库作为本书中代码的最正确版本。

由于 Chai 断言库的限制，运行示例的最小浏览器要求如下：

+   **Chrome**: 7+

+   **Safari**: 5+

+   **Firefox**: 4+

+   **Internet Explorer**: 9+

本书使用的供应商库版本包括以下内容：

+   **Backbone.js**: 1.0.0

+   **Underscore.js**: 1.4.4

+   **jQuery**: 2.0.2

+   **Mocha**: 1.9.0

+   **Chai**: 1.7.1

+   **Sinon.JS**: 1.7.3

GitHub 仓库将努力跟上这些库随时间演变的步伐。同时，书中大多数的应用和测试样本在可预见的未来应该仍然可以很好地与更新的库一起工作，除非本书或网站上特别指出。

每章的文件和代码都通过 `chapters/NUMBER` 目录结构提供，其中 `NUMBER` 是章节编号。示例 Backbone.js 网络应用——Notes——在 `notes` 目录中以 `localStorage` 版本提供，并在 `notes-rest` 中作为完整的 MongoDB 支持的 Node.js 服务器提供。

要检索示例代码，您可以从以下位置下载整个压缩存档：[`github.com/ryan-roemer/backbone-testing/archive/master.zip`](https://github.com/ryan-roemer/backbone-testing/archive/master.zip)。另一个选项是使用 `git` 直接检出源代码：

```js
$ git clone https://github.com/ryan-roemer/backbone-testing.git

```

最后，您可以从[`www.packtpub.com`](http://www.packtpub.com)的账户下载您购买的所有 Packt 书籍的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 错误清单

尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何错误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**错误提交**表单链接，并输入您的错误详情来报告它们。一旦您的错误得到验证，您的提交将被接受，错误将被上传到我们的网站，或添加到该标题的错误清单中。任何现有的错误都可以通过从[`www.packtpub.com/support`](http://www.packtpub.com/support)选择您的标题来查看。

## 盗版

互联网上对版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现我们作品的任何非法副本，无论形式如何，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过`<copyright@packtpub.com>`与我们联系，并提供涉嫌盗版材料的链接。

我们感谢您在保护我们作者以及为我们提供有价值内容的能力方面的帮助。

## 问题

如果您在本书的任何方面遇到问题，可以通过`<questions@packtpub.com>`联系我们，我们将尽力解决。
