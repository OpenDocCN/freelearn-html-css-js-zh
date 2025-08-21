# 序言

这是一本关于 JavaScript 编程语言的书，面向希望使用流行的基于客户端的 JavaScript 框架开发响应式网络应用程序的.NET 开发者，以及希望创建丰富用户体验的程序员。它也适合具有 JavaScript 编程语言基础知识并希望了解一些核心和高级概念以及一些业界最佳实践和模式的程序员，以结构和设计网络应用程序。

这本书从 JavaScript 的基础知识开始，帮助读者了解核心概念，然后逐步深入到一些高级主题。其中一章主要关注 jQuery 库，这个库在整个网络应用开发中广泛使用，之后是一章关于 Ajax 技术，帮助开发者理解如何进行异步请求。接着是使用原生 JavaScript 的 XHR 对象或 jQuery 库进行请求的选项。还有一章通过使用 Angular 2 和 ASP.NET Core 开发完整的应用程序来介绍 TypeScript，这是一种支持 ECMAScript 2015 最新和 evolving 特性的 JavaScript 的超集。我们还将探索 Windows JavaScript (WinJS)库，使用 JavaScript 和 HTML 开发 Windows 应用程序，并使用这个库将 Windows 行为、外观和感觉带到 ASP.NET 网络应用程序中。有一个完整的章节介绍了 Node.js，帮助开发者了解 JavaScript 语言在服务器端的强大之处，之后的一章则讨论了在大型项目中使用 JavaScript 的方法。最后，这本书将以测试和调试章节结束，讨论哪些测试套件和调试技术可用于故障排除并使应用程序健壮。

这本书有一些非常密集的主题，需要全神贯注，因此非常适合有一些先前知识的人。所有章节都与 JavaScript 相关，围绕 JavaScript 框架和库构建丰富的网络应用程序。通过这本书，读者将获得关于 JavaScript 语言及其构建在其上的框架和库的端到端知识，以及测试和调试 JavaScript 代码的技术。

# 本书涵盖内容

第一章，*现代网络应用程序的 JavaScript*，关注 JavaScript 的基本概念，包括变量的声明、数据类型、实现数组、表达式、运算符和函数。我们将使用 Visual Studio 2015 编写简单的 JavaScript 程序，并了解这个 IDE 为编写 JavaScript 程序提供了什么。我们还将研究如何编写 JavaScript 代码，并比较.NET 运行时与 JavaScript 运行时的区别，以阐明代码编译过程的执行周期。

第二章，*高级 JavaScript 概念*，涵盖了 JavaScript 的高级概念，并向开发者展示了 JavaScript 语言的洞察。它将展示 JavaScript 语言在功能方面可以被使用的程度。我们将讨论变量提升及其作用域、属性描述符、面向对象编程、闭包、类型数组和异常处理。

第三章，*在 ASP.NET 中使用 jQuery*，讨论了 jQuery 及其在 ASP.NET Core 开发的网络应用程序中的使用。我们将讨论 jQuery 提供的选项及其与普通原生的 JavaScript 在操作 DOM 元素、附加事件和执行复杂操作方面的优势。

第四章，*Ajax 技术*，讨论了被称为 Ajax 请求的异步请求技术。我们将探讨使用 XMLHttpRequest（XHR）对象的核心概念，并研究 Ajax 请求的基本处理架构以及它提供的事件和方法。另一方面，我们还将探讨 jQuery 库与普通的 XHR 对象相比提供的内容。

第五章，*使用 Angular 2 和 Web API 开发 ASP.NET 应用程序*，介绍了 TypeScript 的基本概念并将其与 Angular 2 结合使用。我们将使用 Angular 2 作为前端的客户端框架、Web API 作为后端服务以及 Entity Framework Core 用于数据库持久化，在 ASP.NET Core 中开发一个简单的应用程序。在撰写本文时，Angular 2 处于测试版阶段，本章使用了测试版。随着 Angular 2 未来的发布，框架有可能发生一些变化，但基本概念几乎保持不变。对于未来的更新，您可以参考[`angular.io/`](http://angular.io/)。

第六章，*探索 WinJS 库*，探讨了由微软开发的 WinJS 库，这是一个不仅可以用 JavaScript 和 HTML 开发 Windows 应用程序，还可以与 ASP.NET 和其他网络框架一起使用的 JavaScript 库。我们将讨论定义类、命名空间、派生类、混合类（mixins）和承诺（promises）的核心概念。我们还将研究数据绑定技术以及如何使用 Windows 控件或 HTML 元素的特定属性来改变控件的行为、外观和感觉。此外，我们将使用 WinRT API 在我们的网络应用程序中访问设备的摄像头，并讨论通过宿主应用（Hosted app）的概念，任何网络应用程序都可以使用 Visual Studio 2015 中的通用窗口模板（Universal Window template）转换成 Windows 应用程序。

第七章，*JavaScript 设计模式*，表明设计模式为软件设计提供了高效的解决方案。我们将讨论一些业界广泛采用的最佳设计模式，这些模式分为创建型、结构型和行为型。每个类别将涵盖四种类型的设计模式，这些模式可以使用 JavaScript 来实现并解决特定的设计问题。

第八章，*Node.js 对 ASP.NET 开发者的应用*，专注于 Node.js 的基础知识以及如何使用它来使用 JavaScript 开发服务器端应用程序。在本章中，我们将讨论视图引擎，如 EJS 和 Jade，以及使用控制器和服务的 MVC 模式实现。此外，我们将在本章结束时通过一些示例来访问 Microsoft SQL Server 数据库，执行对数据库的读取、创建和检索操作。

第九章，*使用 JavaScript 进行大型项目开发*，提供了使用 JavaScript 进行大型应用开发的最佳实践。我们将讨论如何通过将项目拆分为模块来提高可扩展性和可维护性，从而结构化我们的基于 JavaScript 的项目。我们将了解如何有效地使用中介者模式（Mediator pattern）提供模块间的通信以及文档框架，以提高你的 JavaScript 代码的可维护性。最后，我们将讨论如何通过将 JavaScript 文件压缩和合并成压缩版本来优化应用程序，并提高性能。

第十章, *测试和调试 JavaScript*, 专注于 JavaScript 应用程序的测试和调试。我们将讨论最受欢迎的 JavaScript 代码测试套件 Jasmine，并使用 Karma 运行测试用例。至于调试，我们将讨论一些使用 Visual Studio 调试 JavaScript 的技巧和技术，以及 Microsoft Edge 为简化调试所提供的内容。最后，我们将研究 Microsoft Edge 如何使调试 TypeScript 文件变得简单的基本概念以及实现所需的配置。

# 您需要什么

全书我们将使用 Visual Studio 2015 来实践示例。对于服务器端技术，我们使用了 ASP.NET Core 进行网络应用开发，并在其上使用 JavaScript。在第八章, *Node.js 对 ASP.NET 开发者的使用*中，我们使用 Node.js 展示了 JavaScript 如何用于服务器端。对于 Node.js，我们需要在 Visual Studio 2015 中安装一些扩展，具体细节在章节中说明。

# 本书适合谁阅读

本书面向具有扎实 ASP.NET Core 编程经验的.NET 开发者。全书使用 ASP.NET Core 进行网络开发，并假定开发者有.NET Core 和 ASP.NET Core 的深入知识或实际经验。

# 约定

在本书中，你会看到多种文本样式，用以区分不同类型的信息。以下是这些样式的一些示例及其含义解释。

文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、假网址、用户输入和 Twitter 处理显示如下："JavaScript 可以放在 HTML 页面的`<head>`或`<body>`部分。"

代码块如下所示：

```js
<html>
  <head>
    <script>
      alert("This is a simple text");
    </script>
  </head>
</html>
```

任何命令行输入或输出如下所示：

```js
dotnet ef database update –verbose

```

**新术语**和**重要词汇**以粗体显示。例如，在菜单或对话框中出现的屏幕上的词汇，在文本中显示为："当页面加载时，它将显示弹出消息和文本**这是一个简单文本**。"

### 注意

警告或重要说明以框的形式出现，如下所示。

### 提示

技巧和小窍门如下所示。

# 读者反馈

我们的读者反馈始终受欢迎。让我们知道您对这本书的看法——您喜欢或不喜欢的地方。读者反馈对我们很重要，因为它有助于我们开发出您能真正从中获益的标题。

要发送一般性反馈，只需发送电子邮件至`<feedback@packtpub.com>`，并在消息主题中提到本书的标题。

如果你在某个主题上有专业知识，并且对撰写或贡献书籍感兴趣，请查看我们的作者指南，网址为[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

既然你已经拥有了一本 Packt 书籍，我们有很多东西可以帮助你充分利用你的购买。

## 下载示例代码

你可以从你账户中的[`www.packtpub.com`](http://www.packtpub.com)下载本书的示例代码文件。如果你在其他地方购买了这本书，你可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给你。

你可以按照以下步骤下载代码文件：

1.  使用你的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的**支持**选项卡上。

1.  点击**代码下载与勘误**。

1.  在**搜索**框中输入书籍名称。

1.  选择你要下载代码文件的书籍。

1.  从下拉菜单中选择你购买这本书的地方。

1.  点击**代码下载**。

你还可以通过在 Packt 出版社网站上点击书籍网页上的**代码文件**按钮来下载代码文件。通过在**搜索**框中输入书籍名称可以访问此页面。请注意，你需要登录到你的 Packt 账户。

下载文件后，请确保使用最新版本解压或提取文件夹：

+   适用于 Windows 的 WinRAR / 7-Zip

+   适用于 Mac 的 Zipeg / iZip / UnRarX

+   适用于 Linux 的 7-Zip / PeaZip

本书的代码包也托管在 GitHub 上，地址为[`github.com/PacktPublishing/JavaScript-For-.NET-Developers`](https://github.com/PacktPublishing/JavaScript-For-.NET-Developers)。我们还有其他来自我们丰富书籍和视频目录的代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。去看看吧！

## 下载本书的彩色图片

我们还为你提供了一个包含本书中使用的屏幕截图/图表彩色图片的 PDF 文件。彩色图片将帮助你更好地理解输出中的变化。你可以从[`www.packtpub.com/sites/default/files/downloads/JavaScriptForNETDevelopers_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/JavaScriptForNETDevelopers_ColorImages.pdf)下载这个文件。

## 勘误表

虽然我们已经竭尽全力确保内容的准确性，但错误仍然可能发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告，我们将非常感激。这样做可以避免其他读者感到沮丧，并帮助我们改进本书的后续版本。如果您发现任何错误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)报告，选择您的书籍，点击**错误提交表单**链接，并输入您的错误详情。一旦您的错误得到验证，您的提交将被接受，并且错误将被上传到我们的网站或添加到该标题下的错误部分现有的错误列表中。

要查看以前提交的错误，请前往[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)并在搜索框中输入书籍名称。所需信息将在**错误**部分出现。

## 盗版

互联网上侵犯版权材料的问题持续存在，所有媒体都受到影响。在 Packt，我们对保护我们的版权和许可证非常认真。如果您在互联网上以任何形式发现我们作品的非法副本，请立即提供给我们地址或网站名称，以便我们可以寻求补救措施。

如果您怀疑有被盗版的材料，请联系我们`<copyright@packtpub.com>`。

我们感激您在保护我们的作者和我们提供有价值内容的能力方面所提供的帮助。

## 问题

如果您对本书任何方面有问题，您可以联系`<questions@packtpub.com>`，我们将尽力解决问题。
