# 前言

随着移动设备上的互联网用户数量每时每刻都在增长，你的网站不再只为桌面机器构建。移动优先的哲学要求网站能够完全兼容所有现有和未来的移动设备。Bootstrap 允许并轻松地使你能够为所有设备包括各种屏幕阅读器设计和开发你的网站。

到目前为止，我们使用所有这些手动 CSS 类和相当数量的各种 JavaScript 库来开发网站。交付期望的结果和升级你的网站传统上一直是一个相当大的挑战。随着移动设备的出现，这项任务变得更加困难。Bootstrap 在这里为你提供援助——提供你需要的所有东西，包括 CSS 类和 JavaScript 组件，全部包含在一个包中。

本书涵盖了 Bootstrap 的所有理论和实践方面，使你成为移动世界的熟练网络开发者。你将能够下载、包含并在你的网络项目中配置 Bootstrap。你将理解 Bootstrap 的内部架构和结构，并完全熟悉 Bootstrap CSS 和组件的使用，以及 Bootstrap 提供的 JavaScript 对象。你还将能够从源代码构建和编译 Bootstrap，并最终根据你的需求进行定制和扩展。

# 本书涵盖内容

第一章, *CSS 和 Bootstrap 的演变*，向你介绍了 CSS 的简要历史和 CSS3 的诞生。它介绍了移动优先的哲学，并阐述了 Bootstrap 的基本概念和需求。

第二章, *Bootstrap 入门*，使你能够下载并将 Bootstrap 包含到你的项目中。你将了解不同 Bootstrap 发行版的文件和文件夹结构。你还将开始使用 Bootstrap 框架构建一个示例应用程序。

第三章, *使用 Bootstrap CSS 创建响应式布局*，介绍了 Bootstrap 提供的所有 CSS 类的主要方面，包括基本 HTML 元素、响应式类、处理图片、Bootstrap 网格系统、数据输入表单等。

第四章, *Bootstrap 中的内置组件*，探讨了 Bootstrap 框架中所有内置的组件。例如，图标字体、导航栏、工具栏、面板、井和分页。你将在示例应用程序中使用所有这些组件来理解其实际应用。

第五章，*Bootstrap 中的 JavaScript 扩展*，使你精通 Bootstrap 提供的所有重要 JavaScript 扩展。你将能够使用模态窗口、标签页、药丸、手风琴、工具提示、下拉菜单、弹出框、警报、轮播图等等。

第六章，*编译和构建 Bootstrap*，介绍了 Bootstrap 的高级用法以及如何建立 Bootstrap 的开发环境，如何编译 Bootstrap 源代码，以及如何构建框架的可分发版本。

第七章，*定制 Bootstrap*，使你能够使用开发环境并定制 Bootstrap 的默认行为以满足你的需求。

第八章，*扩展 Bootstrap*，介绍了 Bootstrap 社区市场，在那里你可以找到不同的资源，通过这些资源你可以扩展和扩展 Bootstrap 的默认功能和特性。例如，你将使用树形视图控件和所见即所得编辑器，这些都不是 Bootstrap 默认提供的。最后，你将能够使用几乎所有的 Bootstrap 主要类和组件以及社区控件来创建示例应用。

# 你需要这本书的内容

本书是开发全设备友好网站的分步详细指南。它从 CSS、CSS3 和 Bootstrap 的基本概念开始。它包括 Bootstrap 提供的所有主要功能和功能的实际知识。它还展示了如何与 Bootstrap 的主要源代码一起工作，以及如何定制和扩展框架。

每一步都伴随着一个易于跟随的实践练习，并展示了 Bootstrap 功能的用法。

为了作为读者理解和处理这本书，你需要了解 HTML、CSS 和 JavaScript。了解 HTML5、CSS3 和 jQuery 将是一个额外的优势；然而，这本书并不本质上依赖于这些先决条件。

# 这本书是为谁而写的

*Bootstrap Essentials* 旨在为所有那些使用 HTML、CSS 和 JavaScript 设计和开发网站和页面的网络开发者提供指导——无论你是否拥有移动网站开发的经验，这本书都将指导你成为一名熟练的 Bootstrap 网络开发者。为了更好地理解 Bootstrap 和移动网站开发，具备 HTML、CSS 和 JavaScript 的先前经验将有所帮助。进一步了解 jQuery 将是一个额外的优势。关于构建和定制 Bootstrap 的先进信息是本书的一部分；因此，想要拥有自己 Bootstrap 版本的读者将能够建立一个完整的 Bootstrap 开发环境。

# 习惯用法

在这本书中，你会发现许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称都按照以下方式显示：“例如，这就是在 Bootstrap CSS 中配置`HTML`全局元素的方式。”

代码块设置如下：

```js
<link rel="stylesheet" media="print" href="print.css">
<link rel="stylesheet" media="(max-width: 800px)" href="max800.css">
<link rel="stylesheet" media="(max-width: 320px)" href="max320.css">
<link rel="stylesheet" media="(orientation: portrait)" href="prt.css">
<link rel="stylesheet" media="(orientation: landscape)" href="lnd.css">
```

任何命令行输入或输出都按照以下方式编写：

```js
npm install --prefix D:\Bootstrap grunt-cli

```

**新术语**和**重要词汇**以粗体显示。你在屏幕上看到的单词，例如在菜单或对话框中，在文本中显示如下：“你所需要做的只是更新这个界面中这些变量的值，最后在页面底部按下**编译**和**下载**按钮。”

### 注意

警告或重要注意事项以如下框的形式出现。

### 技巧

技巧和窍门以如下方式显示。

# 读者反馈

我们欢迎读者的反馈。告诉我们你对这本书的看法——你喜欢什么或不喜欢什么。读者反馈对我们很重要，因为它帮助我们开发出你真正能从中获得最大价值的标题。

要发送给我们一般性的反馈，只需发送电子邮件至`<feedback@packtpub.com>`，并在邮件主题中提及本书的标题。

如果你在某个领域有专业知识，并且你对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南，链接为[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在你已经是 Packt 图书的骄傲拥有者，我们有一些事情可以帮助你从你的购买中获得最大价值。

## 下载示例代码

你可以从你购买的所有 Packt Publishing 书籍的账户中下载示例代码文件。[`www.packtpub.com`](http://www.packtpub.com)。如果你在其他地方购买了这本书，你可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给你。

## 错误清单

尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果你在我们的某本书中发现错误——可能是文本或代码中的错误——如果你能向我们报告这一点，我们将不胜感激。通过这样做，你可以帮助其他读者避免挫败感，并帮助我们改进本书的后续版本。如果你发现任何错误清单，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择你的书籍，点击**错误提交表单**链接，并输入你的错误详细信息。一旦你的错误清单得到验证，你的提交将被接受，错误清单将被上传到我们的网站或添加到该标题的错误部分下的现有错误清单中。

要查看之前提交的勘误表，请访问 [`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将出现在**勘误**部分。

## 盗版

互联网上对版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现任何形式的我们作品的非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过 `<copyright@packtpub.com>` 联系我们，并提供疑似盗版材料的链接。

我们感谢您在保护我们的作者和我们为您提供有价值内容的能力方面的帮助。

## 询问

如果您在这本书的任何方面遇到问题，您可以通过 `<questions@packtpub.com>` 联系我们，我们将尽力解决问题。
