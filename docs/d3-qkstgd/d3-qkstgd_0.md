# 前言

欢迎阅读 *D3.js 快速入门指南*。在其中，我们将通过一系列大型构建来介绍 D3 的基础知识。到结束时，你应该对库有足够的了解，可以出去构建自己的交互式数据可视化。

# 本书面向的对象

本书面向对数据可视化感兴趣的初级到高级前端和全栈 Web 开发者。读者需要对 HTML、CSS、JavaScript、AJAX 以及服务器是什么有基本的了解，才能使用本书中给出的代码和概念。

# 本书涵盖的内容

第一章，*开始使用 D3.js*，提供了一个关于 D3 为何如此有趣的概述。我们探讨了 SVG 元素是什么，并设置我们的机器，使其准备好创建 D3 代码。我们还审视了本书的学习方法以及它如何应用于我们将要构建的应用程序。

第二章，*使用 SVG 通过代码创建图像*，涵盖了 SVG 的基础（基础标签、基本元素、定位和样式）。我们还探讨了贝塞尔曲线以及如何使用它们绘制有机形状。现在我们已经准备好学习如何使用 D3 来修改这些元素。

第三章，*构建交互式散点图*，解释了静态散点图，并展示了如何构建一个显示其数据的表格。

第四章，*制作一个基本的交互式散点图*，展示了尽可能多的有用模块，以及基于我们使用它们的经验提供的日常活动示例和个人评论。

第五章，*使用数据文件创建柱状图*，涵盖了任何系统管理员每天都需要运行的许多有趣用例。我们还可以通过自定义剧本展示许多其他任务。但并非每个脚本都被视为良好的自动化。重要的是，正确的节点能够无错误且在短时间内从状态 A 过渡到状态 B。

第六章，*通过动画 SVG 元素创建交互式饼图*，展示了当你从饼图中移除部分时，饼图是如何进行动画的。

第七章，*使用物理力创建力导向图*，展示了如何使用 D3 创建一个可视化数据中各个节点之间关系的图表。这在诸如绘制朋友网络、显示父子公司关系或显示公司员工层级等场景中非常有用。

第八章，*映射*，讨论了 GeoJSON，它的用途以及为什么它与更通用的 JSON 数据不同。我们还介绍了如何使用 D3 创建投影并将 GeoJSON 数据渲染为地图。

# 为了充分利用本书

本书假设您对 HTML、CSS、JavaScript、AJAX 以及服务器是什么有基本的了解，以便能够使用本书中给出的代码和概念。

对于本书，您实际上只需要下载并安装以下内容：

+   Chrome，可在[`www.google.com/chrome/`](https://www.google.com/chrome/)找到：一个网页浏览器，以便我们可以查看我们的可视化。

+   Node：[`nodejs.org/en/`](https://nodejs.org/en/)：这允许我们从终端运行 JavaScript。在第四章“制作基本散点图交互”中，我们将使用它来执行 AJAX 调用。

+   一个代码编辑器。如果您是编程新手，我建议使用 Atom：[`atom.io/`](https://atom.io/).

# 下载示例代码文件

您可以从[www.packt.com](http://www.packt.com)的账户下载本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[www.packt.com/support](http://www.packt.com/support)并注册，以便直接将文件通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在[www.packt.com](http://www.packt.com)登录或注册。

1.  选择“支持”标签。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入本书的名称，并遵循屏幕上的说明。

文件下载完成后，请确保使用最新版本的以下软件解压缩或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/D3.js-Quick-Start-Guide`](https://github.com/PacktPublishing/D3.js-Quick-Start-Guide)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还提供其他来自我们丰富图书和视频目录的代码包，可在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**找到。查看它们吧！

# 下载彩色图像

我们还提供了一份包含本书中使用的截图/图表彩色图像的 PDF 文件。您可以从这里下载：[`www.packtpub.com/sites/default/files/downloads/9781789342383_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/9781789342383_ColorImages.pdf)[.](https://www.packtpub.com/sites/default/files/downloads/9781789342383_ColorImages.pdf)

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“将下载的`WebStorm-10*.dmg`磁盘映像文件作为系统中的另一个磁盘挂载。”

代码块设置如下：

```js
<circle r=50 cx=50 cy=50 fill=red stroke=blue stroke-width=5></circle>
```

当我们希望引起您对代码块中特定部分的注意时，相关的行或项目将以粗体显示：

```js
<head>
       <link rel="stylesheet" href="app.css">
</head>
```

**粗体**：表示新术语、重要单词或您在屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。例如：“从管理面板中选择系统信息。”

警告或重要注意事项如下所示。

技巧和窍门如下所示。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果您对本书的任何方面有疑问，请在邮件主题中提及书名，并将邮件发送至 `customercare@packtpub.com`。

**勘误**：尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告，请访问 [www.packt.com/submit-errata](http://www.packt.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上发现任何形式的我们作品的非法副本，如果您能提供位置地址或网站名称，我们将不胜感激。请通过 `copyright@packt.com` 联系我们，并附上材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且有兴趣撰写或参与一本书籍，请访问 [authors.packtpub.com](http://authors.packtpub.com).

# 评论

请留下评论。一旦您阅读并使用了这本书，为何不在您购买它的网站上留下评论呢？潜在读者可以查看并使用您的客观意见来做出购买决定，Packt 可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

如需了解 Packt 的更多信息，请访问 [packtpub.com](https://www.packtpub.com/).
