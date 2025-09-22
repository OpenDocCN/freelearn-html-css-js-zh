# 前言

|   | "JavaScript 是唯一一种我知道人们在使用它之前感觉不需要学习的语言。" |   |
| --- | --- | --- |
|   | --道格拉斯·克罗克福德 |

本书首先介绍了一个采用微服务架构的企业级应用，使用 Node.js 构建 Web 服务。随着我们继续前进，本书将向您展示如何使用 WebRTC 构建浏览器-浏览器应用。然后，我们专注于构建一个使用 WebSockets 的实时 Web 应用。

当您对不同的架构有了扎实的掌握后，您将了解如何使用函数式响应式编程（FRP）编写更好的响应式代码。然后，我们将探讨 Bootstrap 4 的新特性以及它如何使构建响应式网站变得更加容易。随着本书接近尾声，您将看到如何构建一个基于创新组件化架构的现代单页应用，该架构使用 React 和 Angular 2。

阅读本书后，您将具备构建现代 Web 应用所需的最新 JavaScript 技术、工具和架构的扎实知识。

# 本书涵盖内容

第一章, *进入微服务架构*，教授微服务架构是什么以及为什么企业级应用会使用它。然后，我们将探讨 Seneca.js，这是一个 Node.js 的微服务工具包。

第二章, *构建优惠券网站*，展示了如何构建一个基本的优惠券网站来演示 Seneca.js 和微服务架构。

第三章, *实时浏览器间通信*，教授什么是 WebRTC 以及如何使用它来实现音频/视频聊天或网站中需要实时浏览器间数据传输或从麦克风、网络摄像头或其他设备检索音频/视频流的功能。我们将学习使用 PeerJS 编写基于 WebRTC 的应用程序，PeerJS 简化了基于 WebRTC 的应用程序开发。

第四章, *构建 Chatroulette*，展示了如何构建一个 Chatroulette 来演示 WebRTC 和 PeerJS。

第五章, *实时双向通信*，教授什么是 WebSockets 以及如何使用 WebSockets 实现实时双向通信。然后，我们将探讨 Socket.IO，它利用 WebSockets 实现实时双向通信。

第六章, *构建实时比分网站*，展示了如何使用 Socket.IO 构建一个简单的实时比分网站。

第七章, *函数式响应式编程*，教您响应式代码以及如何使用函数式响应式编程编写更好的响应式代码。然后我们将探索 Bacon.js，这是一个用于 JavaScript 的函数式响应式编程库。

第八章, *使用 Bacon.js 构建高级配置搜索小部件*，帮助您使用 Bacon.js 构建高级配置搜索小部件。

第九章, *Bootstrap 4 的新特性*，为您介绍 Bootstrap 4 的新功能和如何使其创建响应式网站变得更加容易。

第十章, *使用 React 构建用户界面*，教您 React.js 是什么以及它如何使编写响应式 UI 的代码更加容易，并处理渲染性能和可重用性。

第十一章, *使用 React 和 Flux 构建 RSS 阅读器*，展示了如何使用 React 和 Flux 架构构建简单的 RSS 阅读器。

第十二章, *Angular 2 的新特性*，教您如何使用 Angular 2 构建网站的客户端。在本章中，我们还将了解 Web 组件。

第十三章, *使用 AngularJS 2 构建搜索引擎模板*，展示了如何使用 Angular 2 构建搜索引擎模板。我们还将学习如何使用 Angular 2 构建单页应用（SPA）。

第十四章, *保护并扩展 Node.js 应用*，教您如何使 Node.js 应用更加安全，以及用于扩展 Node.js 应用的常见技术。

# 阅读本书所需的条件

您可以使用任何支持 Node.js 和 MongoDB 的操作系统。您需要一个浏览器，但建议您使用 Chrome 的最新版本，因为它在支持本书中涵盖的新技术方面领先。您还需要一个摄像头和麦克风。最后，您需要一个有效的互联网连接。

# 本书面向的对象

本书面向希望探索一些现代 JavaScript 特性、技术和架构的现有 JavaScript 开发者，以便开发前沿的 Web 应用。

# 习惯用法

在本书中，您会发现许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称如下所示：“在这里，我们首先调用`senena`对象的`make`方法。它用于获取实体的存储引用。例如，在 MySQL 的情况下，`make`方法获取到表的引用。”

代码块设置如下：

```js
var script_start_time = Bacon.constant(Date.now()).map(function(value){
  var date = new Date(value);
  return (date).getHours() + ":" + (date).getMinutes() + ":" + (date).getSeconds();
});
```

**新术语**和**重要词汇**以粗体显示。屏幕上显示的单词，例如在菜单或对话框中，在文本中如下所示：“现在点击**管理员**按钮访问管理员面板并接受优惠券。以下是管理员面板的外观。”

### 注意

警告或重要注意事项以如下方式显示在框中。

### 小贴士

技巧和窍门如下所示。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们您对这本书的看法——您喜欢或不喜欢的地方。读者反馈对我们非常重要，因为它帮助我们开发出您真正能从中获得最大收益的标题。

要发送一般反馈，请简单地发送电子邮件至`<feedback@packtpub.com>`，并在邮件的主题中提及书的标题。

如果您在某个主题上具有专业知识，并且您对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 图书的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)的账户下载本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的**支持**标签上。

1.  点击**代码下载与勘误**。

1.  在**搜索**框中输入书的名称。

1.  选择您想要下载代码文件的书籍。

1.  从下拉菜单中选择您购买这本书的地方。

1.  点击**代码下载**。

您还可以通过点击 Packt Publishing 网站书籍网页上的**代码文件**按钮下载代码文件。您可以通过在**搜索**框中输入书的名称来访问此页面。请注意，您需要登录到您的 Packt 账户。

文件下载完成后，请确保您使用最新版本的以下软件解压或提取文件夹：

+   Windows 上的 WinRAR / 7-Zip

+   Mac 上的 Zipeg / iZip / UnRarX

+   Linux 上的 7-Zip / PeaZip

本书的相关代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Modern-JavaScript-Applications`](https://github.com/PacktPublishing/Modern-JavaScript-Applications)。我们还有其他来自我们丰富图书和视频目录的代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们吧！

## 下载本书的彩色图像

我们还为您提供了一个包含本书中使用的截图/图表彩色图像的 PDF 文件。彩色图像将帮助您更好地理解输出中的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/ModernJavaScriptApplications_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/ModernJavaScriptApplications_ColorImages.pdf)下载此文件。

## 勘误

尽管我们已经尽一切努力确保我们内容的准确性，但错误仍然会发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。

要查看之前提交的勘误表，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将出现在**勘误**部分下。

## 盗版

在互联网上对版权材料的盗版是所有媒体中持续存在的问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现任何形式的我们作品的非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过`<copyright@packtpub.com>`与我们联系，并提供疑似盗版材料的链接。

我们感谢您在保护我们作者和我们提供有价值内容的能力方面所提供的帮助。

## 问题

如果您对本书的任何方面有问题，您可以通过`<questions@packtpub.com>`联系我们，我们将尽力解决问题。
