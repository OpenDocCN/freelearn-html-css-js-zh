# 前言

Puppeteer 是由 Google 开发的一款多用途浏览器自动化工具。你将看到 Puppeteer 以多种不同的方式被使用。它被用于网页抓取、任务自动化、内容生成、网页监控和 UI 测试。

本书专注于 UI 测试，但不会止步于此。例如，我们有一章专注于内容生成，另一章则专注于网页抓取。如果你从头到尾阅读这本书，你将能够将 Puppeteer 应用于所有领域。

当我发现市场上没有关于 Puppeteer 的书籍时，这真的激励了我：我想这本书要涵盖整个 Puppeteer API。到本书结束时，你将了解整个 Puppeteer API。

# 本书面向的对象

如果你是一位寻求更好、更现代的工具来完成工作的质量保证专业人士，这本书就是为你准备的。

现在，UI 测试不仅限于 QA 团队。有一股新的前端开发者浪潮正在寻找测试他们 UI 组件的工具。如果你是一位在日常工作中使用 JavaScript 和 Node.js 的网页开发者，并且你想测试你的 UI 组件，这本书也适合你。

网页开发者也将从本书中学到如何使用浏览器自动化工具来自动化截图、生成 PDF 文件或进行网页抓取等任务。

# 本书涵盖的内容

*第一章*，*开始使用 Puppeteer*，为本书奠定了基础。它将通过介绍这个工具并让你熟悉基本知识来帮助你开始使用 Puppeteer。你还将学习如何在 JavaScript 中编写异步代码。

*第二章*，*自动化测试和测试运行器*，涵盖了端到端测试的基础知识和不同类型测试之间的区别。在章节的后半部分，我们将介绍创建和组织测试项目以及开始使用测试运行器。

*第三章*，*浏览网站*，将带你开始编写测试代码。你将学习如何启动浏览器，导航到页面，并进行一些断言。然后，你将了解如何将测试发布到云端进行测试。

*第四章*，*与页面交互*，全部关于交互。一旦你到达一个页面，你如何测试它？你如何模拟用户交互？本章将带你了解与页面交互的最常见方式。本章还涵盖了一些基本的 HTML 概念，这样你就可以充分利用 Puppeteer 提供的所有工具。

*第五章*, *等待元素和网络调用*，教你如何等待你在测试页面上的不同场景——等待页面加载并准备就绪，等待按钮启用，等待 Ajax 调用完成，等等。本章涵盖了 Puppeteer 提供的所有处理这些场景的工具。

*第六章*, *执行和注入 JavaScript*，展示了 Puppeteer 的最佳特性之一：轻松注入 JavaScript 代码。在这一章中，我们将暂时离开端到端测试的世界，深入使用通用工具进行网页自动化。

*第七章*, *使用 Puppeteer 生成内容*，扩展了 Puppeteer 的使用范围，展示了如何使用 Puppeteer 创建内容。我们将从学习截图的创建及其在回归测试中的应用开始。然后，我们将涵盖 PDF 生成，最后，我们将学习如何动态创建页面。

*第八章*, *环境模拟*，解释了 Puppeteer 提供的所有用于模拟不同场景的工具。它将向你展示如何模拟移动设备、不同的屏幕分辨率、各种网络速度、地理位置，甚至视力缺陷等问题。

*第九章*, *爬虫工具*，揭示了网络爬取的神秘面纱，展示了如何利用它来为你带来优势。你将学习如何使用 Puppeteer Cluster 创建爬虫并并行运行任务。

*第十章*, *评估和提升网站性能*，展示了 Puppeteer 如何帮助开发者评估和提升他们网站的性能。我们将学习如何提取和分析你在开发者工具中可以看到的所有性能指标。本章还提供了对 Google Lighthouse 的精彩介绍，以及如何自动化其报告并将其集成到你的测试中。

我还希望这本书能给你的工具箱增添更多工具，而不仅仅是 Puppeteer。在本书的学习过程中，你将了解其他工具，例如 GitHub Actions、Visual Studio Code、Checkly 及其 Puppeteer 录制器，以及其他许多工具。

# 为了最大限度地利用这本书

本书假设你有一些 JavaScript 的知识，但不需要成为专家。所有示例都将运行在 Node.js 上。如果你有这个框架的经验，你会发现开始更容易。如果你没有，不要担心；我们不会深入探讨 Node.js*.*

我们将使用的所有工具都是跨平台的。您将能够使用任何流行的操作系统（如 Windows、macOS 或 Linux）跟随本书的代码。在*第一章*“使用 Puppeteer 入门”中，我们将看到如何安装 Node.js 和 Visual Studio Code。如果您已经安装了这些工具，请注意 Puppeteer 依赖于 Node 10.18.1+。

**如果您正在使用这本书的数字版，我们建议您亲自输入代码或通过 GitHub 仓库（下一节中提供链接）访问代码。这样做将帮助您避免与代码复制粘贴相关的任何潜在错误。**

# 下载示例代码文件

您可以从 GitHub 下载本书的示例代码文件：[`github.com/PacktPublishing/UI-Testing-with-Puppeteer`](https://github.com/PacktPublishing/UI-Testing-with-Puppeteer)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还提供其他丰富的代码包，这些代码包来自我们的书籍和视频目录：[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)。请查看它们！

# 免责声明

本书中的许多示例将基于我们大多数人每天都会使用的真实网站。我们将测试 GitHub.com、StackOverlow.com、天气频道和 PacktPub 等网站。这意味着您将学习如何使用最新技术自动化真实网站。然而，这也意味着一些代码示例最终可能会失败。这可能是由于网络条件、服务器问题或网站重新设计。然而，我们的主要目标是您能够学习这些概念并将它们应用到您自己的解决方案中，而不仅仅是这些变化。

# 下载颜色图像

我们还提供了一个包含本书中使用的截图/图表的颜色图像的 PDF 文件。您可以从这里下载：[`static.packt-cdn.com/downloads/9781800206786_ColorImages.pdf`](https://static.packt-cdn.com/downloads/9781800206786_ColorImages.pdf)。

或者，您也可以在以下位置找到本书的颜色图像：[`github.com/PacktPublishing/UI-Testing-with-Puppeteer/blob/master/ColorImages.pdf`](https://github.com/PacktPublishing/UI-Testing-with-Puppeteer/blob/master/ColorImages.pdf)

# 使用的约定

本书使用了多种文本约定。

`文本中的代码`：表示文本中的代码单词、函数、变量、文件夹名称、文件名、文件扩展名、路径名、虚拟 URL 和用户输入。以下是一个示例：“每个 HTML 文档都将包含在`<html>`元素中。”

代码块设置如下：

```js
const productId = config.productToTestId;
const productDiv = await this.page.$(`[data-test-product-id="${productId}"]`);
const stockElement = await productDiv.$('[data-test-stock]');
const priceElement = await productDiv.$('[data-test-price]');
```

任何命令行输入或输出都应如下编写：

```js
~ % /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --headless --remote-debugging-port=9222 --crash-dumps-dir=/tmp
```

`page.click`，该函数将返回一个**Promise**。

提示或重要注意事项

看起来是这样的。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果您对这本书的任何方面有疑问，请在邮件主题中提及书名，并通过 customercare@packtpub.com 给我们发送邮件。

**勘误表**：尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们非常感谢您能向我们报告。请访问[www.packtpub.com/support/errata](http://www.packtpub.com/support/errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上发现任何形式的我们作品的非法副本，我们非常感谢您能提供位置地址或网站名称。请通过版权@packt.com 与我们联系，并附上材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com)。

# 评价

请留下您的评价。一旦您阅读并使用过这本书，为何不在购买它的网站上留下评价呢？潜在读者可以查看并使用您的客观意见来做出购买决定，我们 Packt 公司可以了解您对我们产品的看法，并且我们的作者可以查看他们对书籍的反馈。谢谢！

如需了解更多关于 Packt 的信息，请访问[packt.com](http://packt.com)。
