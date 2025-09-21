# 前言

*学习 Chart.js*将使数据密集型网站的视觉化变得简单且吸引人。我将解释如何将复杂的数据简化、易于访问和直观，以便您的用户能够更好地理解您的网站。

本书是创建和发布您自己的交互式数据可视化项目的实用指南。

阅读本书后，您将能够使用 Chart.js 创建美丽的网络图表。

# 本书面向对象

本书是为希望为网络创建交互式数据可视化的设计师和艺术家而编写的。

基本的 HTML、CSS 和 JavaScript 知识将大有裨益。

# 要充分利用这本书

我建议您阅读第一章，以确保您对 Chart.js 的基本概念和基础原理有所了解。在接下来的章节中，我们将看到 Chart.js 所需的设置以及本书将教授的各种视觉表示技术及其用途。将会有关于使用不同图表类型的教程，并且我们将探索它们的交互性。

# 下载示例代码文件

您可以从[www.packt.com](http://www.packt.com)的账户下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[www.packt.com/support](http://www.packt.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在[www.packt.com](http://www.packt.com)登录或注册。

1.  选择支持选项卡。

1.  点击代码下载与勘误。

1.  在搜索框中输入书名，并遵循屏幕上的说明。

文件下载后，请确保使用最新版本的以下软件解压缩或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Learn-Charts.js`](https://github.com/PacktPublishing/Learn-Charts.js)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有其他来自我们丰富的图书和视频目录的代码包，可在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**上找到。查看它们吧！

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“这设置了在`fill()`命令中使用的颜色。”

代码块设置如下：

```js
const chartObj = {…}; // the chart data is here
const context = canvas.getContext("2d");
new Chart(context, chartObj); // this will display the chart in the canvas
```

任何命令行输入或输出都应如下编写：

```js
npm install chart.js --save
bower install chart.js --save
```

**粗体**：表示新术语、重要词汇或屏幕上看到的词汇。例如，菜单或对话框中的文字会像这样显示。以下是一个示例：“您只需在“资源”选项卡中添加 Chart.js CDN。”

警告或重要注意事项如下所示。

小贴士和技巧看起来像这样。

# 联系我们

我们欢迎读者的反馈。

**一般反馈**：如果您对本书的任何方面有疑问，请在邮件主题中提及书名，并电子邮件我们至 `customercare@packtpub.com`。

**勘误**：尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告，我们将不胜感激。请访问 [www.packt.com/submit-errata](http://www.packt.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上以任何形式遇到我们作品的非法副本，我们将不胜感激，如果您能提供位置地址或网站名称，我们将不胜感激。请通过 `copyright@packt.com` 联系我们，并提供材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问 [authors.packtpub.com](http://authors.packtpub.com/).

# 评论

请留下评论。一旦您阅读并使用了这本书，为什么不在这本书购买的网站上留下评论呢？潜在读者可以看到并使用您的客观意见来做出购买决定，Packt 的我们能够了解您对我们产品的看法，而我们的作者可以查看他们对书籍的反馈。谢谢！

如需更多关于 Packt 的信息，请访问 [packt.com](http://www.packt.com/).
