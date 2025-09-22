# 前言

在过去的十年中，平均 Web 应用程序的 JavaScript 代码库呈指数级增长。然而，当前版本的 JavaScript 是几年前设计的，缺乏一些应对现代 JavaScript 应用程序可能遇到的复杂程度所必需的特性。由于这些缺失的特性，维护性问题已经出现。

**ECMAScript 2015**规范旨在解决 JavaScript 的一些维护性问题，但其实现仍在进行中，许多不兼容的 Web 浏览器仍在使用。因此，广泛采用**ECMAScript 2015**规范预计将是一个缓慢的过程。

为了解决 JavaScript 的维护性和可扩展性问题，TypeScript 在 2012 年 10 月公开发布，这是在微软内部开发两年之后：

“我们设计 TypeScript 是为了满足构建和维护大型 JavaScript 程序的 JavaScript 编程团队的需求。TypeScript 帮助编程团队定义软件组件之间的接口，并深入了解现有 JavaScript 库的行为。TypeScript 还使团队能够通过将代码组织成动态可加载的模块来减少命名冲突。TypeScript 的可选类型系统使 JavaScript 程序员能够使用高度生产力的开发工具和实践：静态检查、基于符号的导航、语句完成和代码重构。”

——TypeScript 语言规范 1.0

一些在 Web 开发方面有多年经验的开发者可能会发现定义大型 JavaScript 应用程序具有挑战性。当我们提到这个术语时，我们将避免考虑应用程序中的代码行数。考虑应用程序中的模块和实体数量以及它们之间的依赖关系作为衡量应用程序规模的标准会更好。我们将大型应用程序定义为非平凡的应用程序，这些应用程序需要大量的开发者努力来维护。

《TypeScript 2.x 第二版学习指南》以简单易懂的格式介绍了 TypeScript 的许多特性。本书将教你如何使用 TypeScript 实现大型 JavaScript 应用程序所需的一切知识。它不仅教授 TypeScript 的核心特性，这些特性对于实现 Web 应用程序至关重要，而且还探讨了某些强大的开发工具、设计原则和良好实践，并展示了如何将它们应用于实际应用中。

第二版已经升级和扩展，增加了五个额外的章节，涵盖了诸如函数式编程、高级类型系统特性、使用 React 和 Angular 进行前端开发的介绍、Node.js 开发的介绍以及 TypeScript 编译器内部 API 的介绍。

新版包含总共 15 章。其中 7 章是完全新的，并未包含在第一版中。

# 这本书面向的对象

如果你是一名开发者，旨在学习 TypeScript 来构建吸引人的 Web 应用程序，这本书适合你。不需要先前的 TypeScript 知识。然而，对 JavaScript 的基本理解将是一个额外的优势。

# 要充分利用这本书

本书不需要任何外部资源。你只需遵循本书中提到的内容，就能充分利用本书。

# 下载示例代码文件

你可以从[www.packtpub.com](http://www.packtpub.com)的账户下载这本书的示例代码文件。如果你在其他地方购买了这本书，你可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给你。

你可以通过以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)上登录或注册。

1.  选择 SUPPORT 标签。

1.  点击代码下载和勘误表。

1.  在搜索框中输入书籍名称，并遵循屏幕上的说明。

下载文件后，请确保使用最新版本的软件解压缩或提取文件夹：

+   Windows 上的 WinRAR/7-Zip

+   Mac 上的 Zipeg/iZip/UnRarX

+   Linux 上的 7-Zip/PeaZip

书籍的代码包也托管在 GitHub 上，网址为[`github.com/remojansen/LearningTypeScript`](https://github.com/remojansen/LearningTypeScript)。如果代码有更新，它将在现有的 GitHub 仓库中更新。GitHub 仓库将包括有关如何运行每个示例的详细说明。

我们还有其他来自我们丰富图书和视频目录的代码包可供选择，网址为**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。查看它们吧！

# 下载彩色图像

我们还提供了一份包含本书中使用的截图/图表彩色图像的 PDF 文件。您可以从这里下载：[`www.packtpub.com/sites/default/files/downloads/LearningTypeScript2xSecondEdition_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/LearningTypeScript2xSecondEdition_ColorImages.pdf)。

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“如果我们尝试预测调用`isIndexPage`的结果，我们需要了解当前状态。”

代码块设置如下：

```js
function addMany(...numbers: number[]) { 
    numbers.reduce((p, c) => p + c, 0); 
} 
```

任何命令行输入或输出都应如下编写：

```js
 npm install typescript -g
```

**粗体**：表示新术语、重要单词或你在屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“从管理面板中选择系统信息。”

警告或重要注意事项如下显示。

小贴士和技巧看起来像这样。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**: 发送电子邮件至`feedback@packtpub.com`，并在邮件主题中提及书籍标题。如果您对本书的任何方面有疑问，请通过`questions@packtpub.com`给我们发送电子邮件。

**勘误**: 尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，如果您能向我们报告这一点，我们将不胜感激。请访问[www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**: 如果您在互联网上以任何形式发现了我们作品的非法副本，如果您能向我们提供位置地址或网站名称，我们将不胜感激。请通过`copyright@packtpub.com`与我们联系，并附上材料的链接。

**如果您有兴趣成为作者**: 如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。一旦您阅读并使用了这本书，为何不在您购买它的网站上留下评论呢？潜在读者可以看到并使用您的客观意见来做出购买决定，我们 Packt 可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

想了解更多关于 Packt 的信息，请访问 [packtpub.com](https://www.packtpub.com/)。
