# 前言

单页、客户端 JavaScript 框架可能是 Web 的未来。在过去几年中，客户端框架已经发展得非常成熟。它们使得创建 Web 应用程序变得更加容易和响应。Ember.js 就是推动这一运动的主要框架之一。

本书深入解释了如何使用 Ember.js 框架，帮助您从初学者成长为专家。您将从基础知识开始，到本书结束时，您将在 Ember 中创建实时 Web 应用程序方面拥有坚实的基础。

我们将首先解释如何使用 Ember.js 框架及其相关工具的关键要点。您将学习如何有效地使用 Ember CLI 以及如何创建和部署您的应用程序。我们将通过观察绑定和观察者来仔细研究 Ember 对象模型和模板。接下来，我们将探讨 Ember 组件和模型以及 Ember Data。然后，我们将查看使用 QUnit 进行集成和验收测试。之后，我们将探讨身份验证和服务以及与 Ember 扩展件一起工作。我们将探索高级主题，例如服务和初始化器以及如何将它们一起使用来构建实时应用程序。

# 本书涵盖的内容

第一章, *Ember CLI 基础*，向您展示如何使用 Ember CLI 命令行工具以及如何处理项目的升级和部署。

第二章, *Ember.Object 模型*，展示了如何创建 Ember 对象和实例以及使用绑定、混合和可枚举。

第三章, *Ember 模板*，告诉您如何使用模板和模板助手。

第四章, *Ember 路由器*，展示了如何设置模型以及如何处理加载状态和重定向。

第五章, *Ember 控制器*，解释了如何使用属性并管理控制器之间的依赖关系。

第六章, *Ember 组件*，涵盖了传递属性、事件和动作。

第七章, *Ember 模型和 Ember Data*，解释了如何操作记录和自定义适配器。

第八章, *日志记录、调试和测试*，展示了如何创建验收和单元测试以及 Ember 检查器。

第九章, *使用 Ember.js 的实际任务*，讨论了如何使用服务、身份验证、Bootstrap 和 Liquid Fire。

第十章, *使用 Ember 的出色任务*，解释了如何使用 Ember 验证、Firebase、WebSockets 和服务器端渲染。

第十一章, *实时网络应用*，讨论了如何使用依赖注入、应用程序初始化器、运行循环和创建插件。

# 你需要这本书什么

+   Windows、Mac OS X 或 Linux

+   NPM 2.x 或更高版本

+   Node 4.0 或更高版本，可从 [`www.nodejs.org`](http://www.nodejs.org) 免费获取

+   Bower 1.4 或更高版本

# 这本书面向谁

任何想要探索 Ember.js 并希望通过更少的编码来制作复杂网络应用的人会发现这本书很有用。建议有编码经验并熟悉 JavaScript。如果你听说过 Ember.js 或只是对单页应用框架的工作原理感到好奇，那么这本书适合你。

# 部分

在这本书中，你会发现一些经常出现的标题（*准备就绪*，*如何操作...*，*它是如何工作的...*，*还有更多...* 和 *也见*）。

为了清楚地说明如何完成食谱，我们使用以下这些部分：

## 准备就绪

这个部分告诉你在食谱中可以期待什么，并描述了如何设置任何软件或任何为食谱所需的初步设置。

## 如何操作...

这个部分包含遵循食谱所需的步骤。

## 它是如何工作的...

这个部分通常包含对前一个部分发生事件的详细解释。

## 还有更多…

这个部分包含有关食谱的附加信息，以便让读者对食谱有更多的了解。

## 也见

这个部分提供了对其他有用信息的有用链接。

# 习惯用法

在这本书中，你会发现许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 处理方式如下所示："我们可以通过使用 `include` 指令来包含其他上下文。"

代码块设置如下：

```js
{
  "usePods": true
}
```

当我们希望将你的注意力引到代码块的一个特定部分时，相关的行或项目将以粗体显示：

```js
const Light = Ember.Object.extend({
  isOn: false,
  color: 'yellow',
  age: null,
  description: Ember.computed('isOn','color',function() {
    return 'The ' + this.get('color') + ' light is set to ' + this.get('isOn');
  }),
  fullDescription: Ember.computed('description','age',function() {
    return this.get('description') + ' and the age is ' + this.get('age')
  }),
 aliasDescription: Ember.computed.alias('fullDescription')
});

const bulb = Light.create({age: 22});
bulb.get('aliasDescription');//The yellow light is set to false and the age is 22.
```

任何命令行输入或输出都如下所示：

```js
$ ember server --port 1234

```

**新术语**和**重要词汇**将以粗体显示。屏幕上显示的单词，例如在菜单或对话框中，将以这种方式显示：“屏幕上的消息将显示**Hello World! My name is John Smith**。**Hello World! My name is Erik Hanchett**。”

### 注意

警告或重要提示将以这样的框显示。

### 小贴士

技巧和窍门将以这种方式显示。

# 读者反馈

我们始终欢迎读者的反馈。告诉我们您对这本书的看法——您喜欢或不喜欢什么。读者反馈对我们很重要，因为它帮助我们开发出您真正能从中获得最大收益的标题。

要向我们发送一般反馈，只需发送电子邮件至`<feedback@packtpub.com>`，并在邮件主题中提及书籍的标题。

如果您在某方面有专业知识，并且您对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 书籍的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)的账户中下载示例代码文件，适用于您购买的所有 Packt Publishing 书籍。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)，并注册以将文件直接通过电子邮件发送给您。

## 勘误

尽管我们已经尽最大努力确保内容的准确性，错误仍然可能发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以避免其他读者感到沮丧，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情来报告。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。

要查看之前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将显示在**勘误**部分下。

## 盗版

互联网上对版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现任何形式的非法复制我们的作品，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过 `<copyright@packtpub.com>` 联系我们，并提供涉嫌盗版材料的链接。

我们感谢您在保护我们作者和我们为您提供有价值内容的能力方面的帮助。

## 问题和建议

如果您对本书的任何方面有问题，您可以联系我们的 `<questions@packtpub.com>`，我们将尽力解决问题。
