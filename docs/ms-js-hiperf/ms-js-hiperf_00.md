# 序言

欢迎来到*精通 JavaScript 高性能*。在这本书中，我们已经以帮助任何 JavaScript 开发者，无论他们是新手上路还是经验丰富的老手的方式，覆盖了 JavaScript 性能。本书涵盖常见的性能瓶颈、如何在代码中寻找性能问题以及如何轻松纠正它们。

我们还回顾了优化我们的 JavaScript 代码的现代方法，不仅依靠对 JavaScript 的深入了解，还使用工具帮助我们优化代码。这些工具包括 Gulp 和 Node.js，它们有助于创建性能出色的构建，还有 Jasmine，一个 JavaScript 单元测试框架，有助于发现 JavaScript 中的应用程序流程问题。我们还使用 Apple Xcode 调试工具对 HTML 和 JavaScript 的混合应用程序进行调试。

# 本书内容概览

第一章，*追求速度*，解释了为什么需要更快的 JavaScript，讨论了为什么 JavaScript 代码传统上运行较慢，并展示了可以帮助我们编写更快 JavaScript 的代码编辑器类型，而无需改变我们的编码风格。

第二章，*使用 JSLint 提高代码性能*，探讨了 JavaScript 中的性能修复，并介绍了 JSLint，一个非常好的 JavaScript 验证和优化工具。

第三章，*理解 JavaScript 构建系统*，教你 JavaScript 构建系统及其在 JavaScript 性能测试和部署中的优势。

第四章，*检测性能*，介绍了 Google 的开发工具选项，并包含使用网络检查器改进我们的 JavaScript 代码性能的复习。

第五章，*操作符、循环和定时器*，解释了 JavaScript 语言中的操作符、循环和定时器，并展示了它们对性能的影响。

第六章，*构造函数、原型和数组*，涵盖了 JavaScript 语言中的构造函数、原型和数组，并展示了它们对性能的影响。

第七章，*避免操作 DOM*，包含有关编写高性能 JavaScript 的 DOM 复习，并展示了如何优化我们的 JavaScript 以使我们的网络应用程序渲染得更快。我们还将查看 JavaScript 动画并测试性能与现代 CSS3 动画相比。

第八章，*Web Workers 和 Promises*，展示了 Web Workers 和 Promises。这一章还教你如何使用它们，包括它们的局限性。

在第九章，*为 iOS 混合应用优化 JavaScript*中，涵盖了为移动 iOS 网络应用（也称为混合应用）优化 JavaScript。另外，我们查看了 Apple Web Inspector，并了解如何在 iOS 开发中使用它。

在第十章，*应用性能测试*中，我们介绍了 Jasmine，一个允许我们单元测试 JavaScript 代码的 JavaScript 测试框架。

# 本书你需要什么

对于这本书，你需要对 JavaScript 有基本的了解，知道如何在 JavaScript 中编写函数和变量，知道如何使用 HTML 和 CSS 等基本网络技术，以及使用 Chrome 开发者工具或 Firebug 等 Web 检查器进行一些基本的调试技能。

你需要一个文本编辑器，最好是能用于 HTML 和 JavaScript 编码的；可用的选择在第一章，*速度的必要性*中有所覆盖。选择编辑器和你在工作的系统中的管理权限由你自己决定，这也取决于你的预算。另外，第九章，*为 iOS 混合应用优化 JavaScript*，严格涵盖了 iOS 开发中的 JavaScript；为此，你需要一份 Xcode 和基于 Intel 的 Mac。如果你没有这些，你仍然可以阅读，但理想情况下，完成这项工作大多数需要使用 Mac。

# 本书面向谁

这本书是为中级 JavaScript 开发者编写的。如果你对用 JavaScript 进行单元测试和编写自己的框架有经验，并能够理解 JavaScript 中的基于实例与基于静态的区别，那么这本书可能不适合你。另外，如果你对 JavaScript 非常陌生——比如，“我怎么使用函数？”——我建议你也寻找一本 JavaScript 初学者的书。

然而，如果你已经对 JavaScript 有所了解，但新手于 node 风格性能测试、grunt 或 gulp 项目部署，以及 JavaScript 中的单元测试，或者如果你想知道如何更快地编写 JavaScript，或者如果你只是想阻止你的代码库落后，而无需重新工作你的编码风格，那么你读对了书。

# 约定

在这本书中，你会发现有许多种文本样式，用以区分不同类型的信息。以下是一些这些样式的例子，以及它们含义的解释。

文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、假 URL、用户输入和 Twitter 处理显示如下："为了解决这个问题，现代浏览器已经实现了新的控制台函数，称为`console.time`和`console.timeEnd`。"

代码块如下所示：

```js
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>Jasmine Spec Runner v2.1.3</title>
```

**新术语**和**重要词汇**以粗体显示。例如，您在屏幕上、菜单或对话框中看到的词汇，在文本中以这种方式出现：“点击**下一页**按钮将您带到下一页。”

### 注意

警告或重要说明以这样的盒子形式出现。

### 提示

技巧和窍门就像这样出现。

# 读者反馈

来自我们读者的反馈总是受欢迎的。让我们知道您对这本书的看法——您喜欢什么或者可能不喜欢什么。读者反馈对我们来说非常重要，以开发您真正能从中获得最大收益的标题。

要向我们提供一般性反馈，只需发送电子邮件到`<feedback@packtpub.com>`，并在您消息的主题中提及书籍的标题。

如果您在某个主题上具有专业知识，并且有兴趣撰写或贡献一本书，请查看我们在[www.packtpub.com/authors](http://www.packtpub.com/authors)上的作者指南。

# 客户支持

现在您已经成为 Packt 书籍的骄傲拥有者，我们有很多东西可以帮助您充分利用您的购买。

## 下载示例代码

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)账户下载所有您购买的 Packt 书籍的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册以便将文件直接通过电子邮件发送给您。

## 下载本书的彩色图像

我们还为您提供了一个包含本书中使用的屏幕截图/图表彩色图像的 PDF 文件。彩色图像将帮助您更好地理解输出的变化。您可以从以下链接下载此文件：[`www.packtpub.com/sites/default/files/downloads/7296OS_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/7296OS_ColorImages.pdf)。

## 错误

虽然我们已经尽一切努力确保我们内容的准确性，但是错误确实会发生。如果您在我们的某本书中发现了一个错误——也许是在文本或代码中——我们将非常感谢您能向我们报告。通过这样做，您可以节省其他读者的时间并帮助我们改进本书的后续版本。如果您发现任何错误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**错误报告****表单**链接，并输入您错误的详细信息。一旦您的错误得到验证，您的提交将被接受，错误将被上传到我们的网站，或添加到该标题的错误部分现有的错误列表中。任何现有的错误可以通过选择您的标题从[`www.packtpub.com/support`](http://www.packtpub.com/support)查看。

## 盗版

互联网上的版权材料侵权是一个持续存在的问题，所有媒体都受到影响。 Packt 出版社非常重视版权和许可的保护。如果您在互联网上发现我们作品的任何非法副本，无论以何种形式，请立即提供位置地址或网站名称，以便我们可以采取补救措施。

如果您发现任何疑似侵权的材料，请通过`<copyright@packtpub.com>`联系我们，并提供材料的链接。

我们感谢您在保护我们的作者和我们为您提供有价值内容的能力方面所提供的帮助。

## 疑问

如果您在阅读本书的过程中遇到任何问题，可以通过`<questions@packtpub.com>`联系我们，我们会尽最大努力解决问题。
