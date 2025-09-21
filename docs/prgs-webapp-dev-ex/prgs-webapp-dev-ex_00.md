# 前言

渐进式网络应用（PWAs）标志着用户体验交付的新时代。现在，每个主要浏览器和平台都支持 PWAs，消除了许多以前仅限于原生应用的缺失功能。如果您是一位从事应用前端开发的工作者，您需要了解渐进式网络应用是什么，它们的优点以及如何有效地构建现代 Web 应用。

您将学习基本 PWA 要求，如 Web 清单文件以及 HTTPS 是如何通过高级服务工作者生命周期和缓存策略工作的。本书涵盖了 Web 性能优化实践和工具，以帮助您持续创建高质量的渐进式网络应用。

# 本书面向对象

如果您是一位希望创造最佳用户体验的 Web 开发者和前端设计师，那么这本书是为您准备的。如果您是一位了解 HTML、CSS 和 JavaScript 的应用开发者，那么这本书将帮助您利用您的技能开发渐进式网络应用，这是应用开发的未来。

# 本书涵盖内容

第一章, *渐进式网络应用简介*，解释了渐进式网络应用是什么以及它们提供的优势。

第二章, *使用 Web 清单创建主屏幕体验*，介绍了 Web 清单文件，并解释了浏览器如何使用它来设计 PWA 安装后的主屏幕和启动体验。

第三章, *使您的网站安全*，解释了为什么 HTTPS 是现代 Web 的要求以及它是如何工作的。

第四章, *服务工作者 - 通知、同步以及我们的播客应用*，介绍了服务工作者、Fetch API 以及实现推送通知。

第五章, *服务工作者生命周期*，展示了服务工作者是如何安装和更新的，以及如何管理这个过程。

第六章, *掌握缓存 API - 在播客应用程序中管理 Web 资源*，深入探讨了服务工作者缓存 API 的工作原理。

第七章, *服务工作者缓存模式*，回顾了您可以在应用程序中使用的一些常见缓存模式。

第八章, *应用高级服务工作者缓存策略*，应用了带有失效技术的缓存策略。

第九章, *优化性能*，解释了什么是 Web 性能优化，并介绍了一些您可以用来测量页面加载时间以改进渐进式网络应用的工具。

第十章，*服务工作者工具*，回顾了四个有助于使开发和管理渐进式 Web 应用更简单、更一致的实用工具。

# 为了充分利用这本书

+   任何开发或负责应用程序前端或用户体验的技术方面的人都会从这本书中受益。

+   假设您具备现代网络开发的中等背景。

+   了解常见的 JavaScript 语法很重要，因为服务工作者是 JavaScript 文件。

+   由于许多现代工具依赖于 Node.js，因此建议您具备基本了解。

+   源代码在 GitHub 上管理，这需要一些使用 Git 源代码控制的知识。

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的账户下载本书的示例代码文件。如果您在其他地方购买了此书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)上登录或注册。

1.  选择支持选项卡。

1.  点击代码下载和勘误。

1.  在搜索框中输入书名，并遵循屏幕上的说明。

下载文件后，请确保使用最新版本的以下软件解压缩或提取文件夹：

+   WinRAR/7-Zip（适用于 Windows）

+   Zipeg/iZip/UnRarX（适用于 Mac）

+   7-Zip/PeaZip（适用于 Linux）

本书代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Progressive-Web-Application-Development-by-Example`](https://github.com/PacktPublishing/Progressive-Web-Application-Development-by-Example)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有来自我们丰富的图书和视频目录的其他代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们吧！

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“将下载的`WebStorm-10*.dmg`磁盘映像文件挂载为系统中的另一个磁盘。”

代码块按照以下方式设置：

```js
 function renderResults(results) {

 var template = document.getElementById("search-results-
 template"),
 searchResults = document.querySelector('.search-results');
```

任何命令行输入或输出都按照以下方式编写：

```js
npm install -g pwabuilder 
```

**粗体**：表示新术语、重要词汇或您在屏幕上看到的词汇。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“从管理面板中选择系统信息。”

警告或重要提示看起来像这样。

小贴士和技巧看起来像这样。

# 联系我们

我们欢迎读者的反馈。

**一般反馈**：请发送电子邮件至 `feedback@packtpub.com`，并在邮件主题中提及书籍标题。如果您对本书的任何方面有疑问，请发送电子邮件至 `questions@packtpub.com`。

**勘误**：尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告这一情况。请访问 [www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上发现我们作品的任何非法副本，我们将不胜感激，如果您能提供位置地址或网站名称。请通过链接至材料内容与我们联系 `copyright@packtpub.com`。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且对撰写或参与一本书籍感兴趣，请访问 [authors.packtpub.com](http://authors.packtpub.com/).

# 评论

请留下评论。一旦您阅读并使用过这本书，为何不在您购买它的网站上留下评论呢？潜在读者可以查看并使用您的客观意见来做出购买决定，Packt 可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

想了解更多关于 Packt 的信息，请访问 [packtpub.com](https://www.packtpub.com/).
