# 前言

这本书是关于构建、理解和维护 JavaScript 框架的——这是许多现代网络应用的关键构建块。在过去 10 多年中，软件框架的生态系统，尤其是 JavaScript 框架，对许多开发者来说都极具吸引力。此外，JavaScript 可以说是最有影响力和最普遍的编程语言，它使数百万关键网络应用和服务成为可能。得益于充满活力的开发者社区和创新公司，JavaScript 框架的发展已经变成一个充满活力和令人兴奋的空间，充满了创新和探索的机会。每个新的框架都带来了自己的哲学和方法，代表了对常见的网络应用和服务的挑战的独特解决方案集。

重要的是要注意，这本书不仅仅是为那些计划从头开始开发他们的 JavaScript 框架的人准备的。虽然它提供了创建良好维护系统的宝贵智慧，但其主要目标超越了这一点。它还旨在提高你对底层框架的理解，提高你的 JavaScript 或软件开发技能。它鼓励你从框架的角度思考，从而培养对可重用软件架构的更深入理解。无论你是寻求深入了解工具集的经验丰富的开发者，还是寻求了解框架景观的新手，这本书都将对你的职业生涯大有裨益。

# 这本书面向谁

如果你是一个 JavaScript 新手或想要深入探索 JavaScript 框架世界的专家，我们为你提供了全面的支持。这本书介绍了前端框架的历史，并指导你创建自己的框架。为了从这本书中学习，你应该对 JavaScript 编程语言有良好的理解，并对现有的软件框架有一些经验。

# 这本书涵盖的内容

*第一章*，*不同 JavaScript 框架的优势*，详细介绍了 JavaScript 框架的世界及其在过去十年中的发展。

*第二章*，*框架组织*，提供了关于框架结构和组织模式的见解。

*第三章*，*内部框架架构*，探讨了各种现有框架的核心技术架构模式。

*第四章*，*确保框架可用性和质量*，解释了一系列技术和工具，使开发者能够构建专注于可用性和易用性的高质量框架。

*第五章*，*框架考虑因素*，探讨了围绕项目目标、技术问题空间和架构设计决策的考虑因素。

*第六章*, *通过示例构建框架*，解释了如何根据模式和最佳技术开发一个简单的 JavaScript 测试框架。

*第七章*, *创建全栈框架*,提供了开发新应用程序框架的实践，该框架将使开发者能够构建大型和小型 Web 应用程序。

*第八章*, *构建前端框架*,特别关注全栈框架的前端组件。

*第九章*, *框架维护*，讨论了与框架发布流程、持续开发周期以及大型框架项目的长期维护相关的话题。

*第十章*, *最佳实践*，解释了围绕通用 JavaScript 框架开发的几个重要主题，并展望了该生态系统框架的未来。

# 要充分利用本书

您需要对 JavaScript 编程语言有良好的理解，并有一些使用现有软件系统（如 React）的经验。

| **本书涵盖的软件/硬件** | **操作系统要求** |
| --- | --- |
| ECMAScript 2023 | Windows, macOS, 或 Linux |
| TypeScript |  |
| Node.js 20 |  |

按照指南在您的机器上安装 Node.js 版本 20+，以便能够与代码文件交互。Node.js 运行时的安装程序可以在[`nodejs.org/en/download`](https://nodejs.org/en/download)找到。

**如果您使用的是本书的数字版，我们建议您亲自输入代码或从本书的 GitHub 仓库（下一节中提供链接）获取代码。这样做将帮助您避免与代码复制和粘贴相关的任何潜在错误。**

# 下载示例代码文件

您可以从 GitHub（[`github.com/PacktPublishing/Building-Your-Own-JavaScript-Framework`](https://github.com/PacktPublishing/Building-Your-Own-JavaScript-Framework)）下载本书的示例代码文件。如果代码有更新，它将在 GitHub 仓库中更新。

我们还有其他来自我们丰富图书和视频目录的代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们！

# 使用约定

本书使用了多种文本约定。

`文本中的代码`: 表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“例如，如果您计划在 Next.js 项目中添加 API 路由来构建 API，它们必须映射到`/api/`端点。”

代码块设置如下：

```js
pages/index.vue
<template>
  <NuxtLink to="/">Index page</NuxtLink>
  <NuxtLink href="https://www.packtpub.com/" target="_blank">Packt</NuxtLink>
</template>
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```js
<root framework directory>
  | <main framework packages>
    + <core framework interfaces...>
    + <compiler / bundler>
  | <tests>
    + <unit tests>
    + <integration and end-to-end tests>
    + <benchmarks>
  | <static / dynamic typings>
  | <documentation>
  | <examples / samples>
  | <framework scripts>
  | LICENSE
  | README documentation
  | package.json (package configuration)
  | <.continuous integration>
  | <.source control add-ons>
  | <.editor and formatting configurations>
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词以**粗体**显示。以下是一个示例：“按下**调试**按钮后，您将看到一个包含项目中所有可用脚本的上下文菜单。”

小贴士或重要注意事项

如此显示。

# 联系我们

我们始终欢迎读者的反馈。

`customercare@packtpub.com`并在您的邮件主题中提及书名。

**勘误表**：尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告，我们将不胜感激。请访问[www.packtpub.com/support/errata](http://www.packtpub.com/support/errata)并填写表格。

`copyright@packt.com`并提供链接到该材料。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com)。

# 分享您的想法

一旦您阅读了《构建您自己的 JavaScript 框架》，我们很乐意听到您的想法！请点击[`packt.link/r/1804617407`](https://packt.link/r/1804617407)为此书提供反馈。

您的评论对我们和科技社区都至关重要，并将帮助我们确保我们提供高质量的内容。

# 下载此书的免费 PDF 副本

感谢您购买这本书！

您喜欢在路上阅读，但无法携带您的印刷书籍到处走？

您的电子书购买是否与您选择的设备不兼容？

请放心，现在，每购买一本 Packt 书籍，您都可以免费获得该书的 DRM 免费 PDF 版本。

在任何地方、任何设备上阅读。直接从您喜欢的技术书籍中搜索、复制和粘贴代码到您的应用程序中。

优惠不会就此停止，您还可以获得独家折扣、时事通讯和丰富的免费内容，每天直接发送到您的收件箱。

按照以下简单步骤获取福利：

1.  扫描下面的二维码或访问以下链接

![二维码图片](img/B19014_QR_Free_PDF.jpg)

[`packt.link/free-ebook/9781804617403`](https://packt.link/free-ebook/9781804617403)

1.  提交您的购买证明

1.  就这样！我们将直接将您的免费 PDF 和其他福利发送到您的邮箱

# 第一部分：JavaScript 框架的景观

前四章深入探讨了现有框架的生态系统。这种方法为 JavaScript 框架当前的技术景观提供了独特且多样化的视角。这些章节强调了结构、设计和保证项目卓越性和开发者友好性的方法论。在这里积累的知识使开发者能够理解框架开发的更广泛背景和基本原理，目标是建立一个坚实的基础。

在本部分，我们涵盖了以下章节：

+   *第一章*, *不同 JavaScript 框架的优势*

+   *第二章*, *框架组织*

+   *第三章*, *内部框架架构*

+   *第四章*, *确保框架可用性和质量*
