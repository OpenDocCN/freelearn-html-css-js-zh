# 前言

Wijmo 是一个专注于用户界面小部件的新 JavaScript 库。它建立在 jQuery UI 的基础上，增强了现有小部件，并添加了新的小部件。在这本书中，我们探讨了对于 Web 开发至关重要的 Wijmo 小部件。涵盖了 15 个小部件的有用配置选项及其使用场景。每当你在 Web 开发中遇到之前实现过的组件或小部件时，Wijmo 小部件很可能已经为你提供了解决方案。本书中的章节旨在让你能够迅速开始使用这些小部件。另一方面，第六章，*使用 Wijmo Grid 的仪表板*，在构建应用程序和解释其工作原理方面采取了不同的方法。

如果你熟悉 Wijmo，就没有必要按照章节顺序阅读。然而，如果你是第一次体验 Wijmo，我建议你按照章节的顺序进行。

# 这本书涵盖什么

第一章，*Wijmo 入门*，介绍了 Wijmo、安装步骤和许可。

第二章，*对话框小部件*，解释了可以添加到 jQuery UI 对话框小部件中的 Wijmo 功能。

第三章，*表单组件*，探讨了 Wijmo 表单小部件。

第四章，*处理图像*，展示了画廊、相册和旋转木马小部件的常见用途。

第五章，*高级小部件*，涵盖了提示、上传、视频和编辑器小部件。

第六章，*使用 Wijmo Grid 的仪表板*，构建了一个结合 Knockout 和 Wijmo 的交互式应用程序。

第七章，*Wijmo 移动*，设置了移动开发环境并介绍了移动视图。

第八章，*扩展 Wijmo*，解释了如何修改小部件和更改主题。

# 你需要这本书什么

你需要一个具有 JavaScript、CSS 和 HTML 语法高亮的文本编辑器。Windows 上的 Notepad++或 Mac 上的 Textmate 就足够了。使用 Wijmo 进行开发不需要花哨的编辑器功能，如自动完成或 JavaScript 的警告。小部件简单易用。

除了文本编辑器，你还需要一个网络浏览器。Wijmo 支持版本高于 5 的 Internet Explorer、Firefox、Safari 或 Chrome。你可能已经安装了其中之一，并且可能更倾向于使用其中一个。

# 这本书面向谁

本书的主要读者是那些在项目中需要使用现成小部件的网页开发者。jQuery UI 缺少必要的组件或功能，而 Wijmo 提供了免费的开源小部件，以及更复杂小部件的授权选项。由于本书涵盖了这两个领域，从事开源项目开发的开发者也能从中受益。

由于 Wijmo 使用简单，许多简单示例都可以由 JavaScript 初学者理解。当满足先前条件时，这本书是 JavaScript 初学者在学会 jQuery 之后应该阅读的第一本书。学习如何使用 Wijmo 小部件将减少编写自定义 JavaScript 的不必要工作。

# 习惯用法

在本书中，您将找到许多不同风格的文本，以区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名如下所示：“`widget` 方法返回对话框 HTML 元素。”

代码块设置如下：

```js
$("#dialog").wijdialog({captionButtons: {
  pin: { visible: false },
  refresh: { visible: false },
  toggle: { visible: false },
  minimize: { visible: false },
  maximize: { visible: false }
  }
});
```

当我们希望将您的注意力引到代码块的一个特定部分时，相关的行或项目将以粗体显示：

```js
maximize: {visible: true, click: function () {
  alert('To enlarge text, click the zoom icon.')
}, iconClassOn: 'ui-icon-lightbulb'},
close: {visible: true, click: self.close, iconClassOn:'ui-icon-close'}
```

**新术语**和**重要词汇**以粗体显示。您在屏幕上、菜单或对话框中看到的单词，例如，在文本中显示如下：“点击**下一步**按钮将您带到下一屏幕”。

### 注意

警告或重要注意事项以如下方式显示在框中。

### 小贴士

小贴士和技巧看起来像这样。

# 读者反馈

我们读者提供的反馈总是受欢迎的。请告诉我们您对这本书的看法——您喜欢什么或可能不喜欢什么。读者反馈对我们开发您真正能从中获得最大收益的标题非常重要。

要向我们发送一般反馈，只需发送电子邮件到 `<feedback@packtpub.com>`，并在邮件主题中提及书名。

如果您在某个主题上具有专业知识，并且您对撰写或为书籍做出贡献感兴趣，请参阅我们在 [www.packtpub.com/authors](http://www.packtpub.com/authors) 上的作者指南。

# 客户支持

现在您已经是 Packt 书籍的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从您在 [`www.packtpub.com`](http://www.packtpub.com) 的账户下载您购买的所有 Packt 书籍的示例代码文件。如果您在其他地方购买了这本书，您可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 并注册，以便将文件直接通过电子邮件发送给您。

## 错误更正

尽管我们已经尽最大努力确保内容的准确性，错误仍然可能发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以避免其他读者感到沮丧，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站，或添加到该标题的勘误部分下的现有勘误列表中。您可以通过从[`www.packtpub.com/support`](http://www.packtpub.com/support)选择您的标题来查看任何现有的勘误。

## 盗版

互联网上版权材料的盗版是一个持续存在的问题，涉及所有媒体。在 Packt，我们非常重视保护我们的版权和许可证。如果您在互联网上发现我们作品的任何非法副本，无论形式如何，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过`<copyright@packtpub.com>`与我们联系，并提供疑似盗版材料的链接。

我们感谢您在保护我们的作者和我们为您提供有价值内容的能力方面的帮助。

## 问题

如果您在本书的任何方面遇到问题，可以通过`<questions@packtpub.com>`联系我们，我们将尽力解决。
