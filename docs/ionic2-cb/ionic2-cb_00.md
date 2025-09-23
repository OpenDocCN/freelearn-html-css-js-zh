# 前言

移动开发的世界是碎片化的，有许多平台、框架和技术。Ionic 旨在填补这一空白，它是一个开源的 HTML5 移动应用框架，允许开发者使用 HTML、CSS 和 AngularJS 等 Web 技术构建具有原生感觉的应用。Ionic 使前端开发者能够轻松成为应用开发者。该框架通过深度 Cordova 集成和一套全面的工具（用于原型设计、后端支持和部署）提供了卓越的性能。

这本书将带你通过使用基于 Ionic 2 的 HTML5 和 JavaScript 开发跨平台移动应用的过程，Ionic 2 是使用 Angular 2 框架的最新版本。你将从熟悉 CLI 和了解如何构建和运行应用开始。你将接触到现实世界移动应用的常见功能，例如使用 Firebase 或本地存储进行用户认证、获取数据和保存数据。接下来，本书将解释 Ionic 如何通过 ngCordova 与 Cordova 集成以支持原生设备功能，并利用其生态系统中的现有模块。你还将探索扩展 Ionic 以创建新组件的高级主题。最后，本书将展示如何自定义 Ionic 主题并为所有平台构建应用。

# 本书涵盖的内容

第一章，*使用 Ionic 2 创建我们的第一个应用*，介绍了 Ionic 2 框架，并提供了如何设置开发环境以及快速创建和运行第一个应用的说明。

第二章，*添加 Ionic 2 组件*，通过一些示例指导你如何管理应用中的页面、状态以及整体导航。

第三章，*使用 Angular 2 构建块扩展 Ionic 2*，深入探讨了 AngularJS 组件、指令以及管道的自定义。你将学习如何利用 Ionic 2 模块架构创建共享服务。

第四章，*验证表单和发送 HTTP 请求*，解释了如何创建一个具有输入验证的复杂表单，通过 REST API 调用检索数据，并与 Stripe 集成进行在线支付。

第五章，*添加动画*，提供了如何将视频嵌入为背景、创建基于物理的 CSS 动画以及将手势绑定到动画状态的说明。

第六章，*使用 Ionic Cloud 进行用户认证和推送通知*，深入探讨了使用 Ionic Cloud 进行用户注册和认证以及发送和接收推送通知。

第七章，*使用 Ionic Native 支持设备功能*，解释了如何使用 Ionic Native 访问原生设备功能，例如相机、社交分享、InAppBrowser 和地图。

第八章，*为应用主题化*，提供了使用 Sass 变量定制不同平台应用的说明。

第九章，*为不同平台发布应用*，探讨了执行应用发布的最终步骤的过程。

# 您需要为本本书准备的内容

+   需要配备 Mac OS X El Capitan 和 root 权限的 Mac 电脑

+   iPhone 5 或更高版本

+   任何运行 Android 5.x 或更高版本的 Android 设备（可选）

+   任何 Windows Phone 设备（可选）

# 本书面向的对象

*Ionic 2 食谱*旨在帮助前端开发者利用现有技能开发跨平台移动应用。本书将帮助您通过深入探讨 AngularJS、Cordova 和 Sass，成为中级或高级 Ionic Framework 开发者。由于 Ionic 是开源的，因此有一个庞大的社区支持这个框架，帮助您继续学习之旅。

# 部分

在本书中，您将找到一些频繁出现的标题（准备就绪、如何操作、工作原理、更多信息、相关内容）。

为了清楚地说明如何完成食谱，我们使用以下这些部分：

## 准备就绪

本节将向您介绍在食谱中可以期待的内容，并描述如何设置任何软件或任何为食谱所需的初步设置。

## 如何操作...

本节包含遵循食谱所需的步骤。

## 工作原理...

本节通常包含对前一个章节发生情况的详细解释。

## 更多信息…

本节包含有关食谱的附加信息，以便使读者对食谱有更深入的了解。

## 相关内容

本节提供了对其他有用信息的链接，以帮助读者了解食谱。

# 习惯用法

在本书中，您将找到许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名如下所示："除非需要进行故障排除，否则无需手动修改`/platforms`或`/plugins`文件夹。"

代码块设置如下：

```js
<ion-tabs>
  <ion-tab [root]="tab1Root" tabTitle="One" tabIcon="water"></ion-tab>
  <ion-tab [root]="tab2Root" tabTitle="Two" tabIcon="leaf"></ion-tab>
  <ion-tab [root]="tab3Root" tabTitle="Three" tabIcon="flame"></ion-tab>
</ion-tabs>
```

任何命令行输入或输出如下所示：

```js
$ sudo npm install -g cordova ionic ios-sim

```

**新术语**和**重要词汇**以粗体显示。屏幕上显示的单词，例如在菜单或对话框中，以这种方式显示：“在本节中，您将学习如何创建一个属性指令，该指令可以防止某些字符在**用户名**中输入，并通过切换其可见性来显示另一个 DOM 节点（显示为**您正在输入用户名**）。”

### 注意

警告或重要注意事项以这种方式出现在框中。

### 小贴士

小贴士和技巧看起来像这样。

# 读者反馈

我们欢迎读者的反馈。告诉我们您对这本书的看法——您喜欢或不喜欢什么。读者反馈对我们来说很重要，因为它帮助我们开发出您真正能从中获得最大收益的标题。

要向我们发送一般反馈，只需发送电子邮件至`<feedback@packtpub.com>`，并在邮件主题中提及书籍标题。

如果你在某个领域有专业知识，并且对撰写或参与一本书籍感兴趣，请参阅我们的作者指南，网址为[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在你已经是 Packt 图书的骄傲拥有者了，我们有一些事情可以帮助你从购买中获得最大收益。

## 下载示例代码

您可以从您的账户中下载本书的示例代码文件，网址为[`www.packtpub.com`](http://www.packtpub.com)。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的**支持**标签上。

1.  点击**代码下载与勘误**。

1.  在**搜索**框中输入书籍名称。

1.  选择您想要下载代码文件的书籍。

1.  从下拉菜单中选择您购买此书的来源。

1.  点击**代码下载**。

你也可以通过点击 Packt Publishing 网站上书籍网页上的**代码文件**按钮来下载代码文件。您可以通过在**搜索**框中输入书籍名称来访问此页面。请注意，您需要登录您的 Packt 账户。

文件下载完成后，请确保您使用最新版本的软件解压缩或提取文件夹：

+   WinRAR / 7-Zip for Windows

+   Zipeg / iZip / UnRarX for Mac

+   7-Zip / PeaZip for Linux

该书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Ionic-2-Cookbook`](https://github.com/PacktPublishing/Ionic-2-Cookbook)。我们还有其他来自我们丰富图书和视频目录的代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。去看看吧！

## 下载本书的颜色图像

我们还为您提供了一个包含本书中使用的截图/图表彩色图像的 PDF 文件。这些彩色图像将帮助您更好地理解输出的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/Ionic2Cookbook_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/Ionic2Cookbook_ColorImages.pdf)下载此文件。

## 勘误

尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。

要查看之前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将出现在**勘误**部分下。

## 侵权

互联网上对版权材料的侵权是一个跨所有媒体的持续问题。在 Packt，我们非常重视保护我们的版权和许可证。如果您在互联网上发现我们作品的任何非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过链接<mailto:copyright@packtpub.com>与我们联系，并提供涉嫌侵权材料的链接。

我们感谢您的帮助，保护我们的作者以及我们为您提供有价值内容的能力。

## 问题

如果您对本书的任何方面有问题，您可以通过链接<mailto:questions@packtpub.com>与我们联系，我们将尽力解决问题。
