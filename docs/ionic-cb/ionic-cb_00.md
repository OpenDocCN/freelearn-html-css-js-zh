# 前言

移动应用程序开发已经是一个热门话题有一段时间了。有多个平台和不同屏幕尺寸和形态的设备，以适应各种需求。这使得移动应用程序开发变得非常困难。幸运的是，Ionic 就是这样的工具之一，它通过允许我们为所有平台和设备编写一次代码来帮助我们减轻这个问题。

在这本书中，读者将学习如何使用 Ionic 创建移动应用程序。我们将从非常基础的事情开始，例如设置开发环境、在应用程序中使用导航、通过 REST API 与后端工作、动画、用户认证、接收推送通知、本地化应用程序、生成文档以及发布应用程序，仅举几例。读者还将了解有关 Angular 和 Ionic CLI 的内容。我希望这本书能够帮助新手开发者以及高级开发者，因为内容是易于理解和高级 Ionic 内容的混合体。

# 本书面向的对象

《Ionic 烹饪书》面向希望创建跨平台移动应用程序的前端开发者。这本书将帮助您通过深入讲解 Angular、Cordova 和 Sass，成为一个足够舒适地承担困难应用程序的移动应用程序开发者。本书的目的是通过解决现实世界的问题（如认证、推送通知、使用摄像头等）来教授 Ionic。尽管如此，如果您是前端开发的新手，您仍然能够跟随本书。

# 要充分利用本书

1.  在这本书中，我假设您对 Angular 有一些了解。大多数时候，您会遇到的问题将是关于 Angular 而不是 Ionic。[`angular.io`](https://angular.io) 在这种情况下将是您的最佳朋友。

1.  如果您想复习 Angular 的知识，我建议您阅读 Victor Savkin 和 Jeff Cross（前 Angular 团队成员）的这本书 [`www.packtpub.com/application-development/essential-angular`](https://www.packtpub.com/application-development/essential-angular)。

1.  即使您可以在不安装 Android 或 iOS 的平台 SDK 的情况下运行大多数示例，我还是建议您从一开始就做这件事，以便在实际设备上测试应用程序。请查看以下指南：

    +   **Android**: [`cordova.apache.org/docs/en/latest/guide/platforms/android/index.html#requirements-and-support`](https://cordova.apache.org/docs/en/latest/guide/platforms/android/index.html#requirements-and-support)

    +   **iOS**: [`cordova.apache.org/docs/en/latest/guide/platforms/ios/index.html`](https://cordova.apache.org/docs/en/latest/guide/platforms/ios/index.html)

# 下载示例代码文件

您可以从 [www.packtpub.com](http://www.packtpub.com) 的账户下载本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问 [www.packtpub.com/support](http://www.packtpub.com/support) 并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)登录或注册。

1.  选择支持选项卡。

1.  点击代码下载与勘误。

1.  在搜索框中输入书名，并遵循屏幕上的说明。

下载文件后，请确保使用最新版本解压缩或提取文件夹。

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Ionic-Cookbook-Third-Edition`](https://github.com/PacktPublishing/Ionic-Cookbook-Third-Edition)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有其他来自我们丰富的图书和视频目录的代码包。请访问**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。查看它们！

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“将下载的`WebStorm-10*.dmg`磁盘映像文件作为系统中的另一个磁盘挂载。”

代码块设置如下：

```js
<ion-tabs> 
  <ion-tab [root]="tab1Root" tabTitle="One" 
   tabIcon="water"></ion-tab>  <ion-tab [root]="tab2Root" 
   tabTitle="Two" 
   tabIcon="leaf"></ion-tab>  <ion-tab [root]="tab3Root" 
   tabTitle="Three" 
   tabIcon="flame"></ion-tab> 
</ion-tabs> 
```

任何命令行输入或输出都按以下方式编写：

```js
$ ionic start LeftRightMenu sidemenu
$ cd LeftRightMenu
```

**粗体**：表示新术语、重要单词或您在屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“从管理面板中选择系统信息。”

警告或重要注意事项看起来像这样。

小贴士和技巧看起来像这样。

# 部分

在本书中，您会发现一些频繁出现的标题（*准备就绪*，*如何操作...*，*它是如何工作的...*，*还有更多...*，以及*另请参阅*）。

为了清楚地说明如何完成食谱，请按照以下方式使用这些部分：

# 准备就绪

本节告诉您在食谱中可以期待什么，并描述如何设置任何软件或任何为食谱所需的初步设置。

# 如何操作...

本节包含遵循食谱所需的步骤。

# 如何操作...

本节通常包含对前一个部分发生情况的详细解释。

# 还有更多...

本节包含有关食谱的附加信息，以便您对食谱有更深入的了解。

# 另请参阅

本节提供了指向其他对食谱有用的信息的链接。

# 联系我们

我们欢迎读者的反馈。

**一般反馈**：请发送电子邮件至 `feedback@packtpub.com` 并在邮件主题中提及书名。如果您对本书的任何方面有疑问，请发送电子邮件至 `questions@packtpub.com`。

**勘误**: 尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果你在这本书中发现了错误，我们将不胜感激，如果你能向我们报告这一点。请访问[www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择你的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**: 如果你在互联网上遇到任何形式的我们作品的非法副本，如果你能提供其位置地址或网站名称，我们将不胜感激。请通过`copyright@packtpub.com`与我们联系，并附上材料的链接。

**如果您有兴趣成为作者**: 如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。一旦你阅读并使用了这本书，为何不在你购买它的网站上留下评论呢？潜在读者可以查看并使用你的客观意见来做出购买决定，Packt 公司可以了解你对我们的产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

如需了解更多关于 Packt 的信息，请访问[packtpub.com](https://www.packtpub.com/).
