# 前言

浏览器的服务工作者功能将使你能够构建高度可用和性能良好的原生 Web 应用程序，这些应用程序可以无缝集成第三方 API。无论你想创建离线 Web 应用程序还是代理，这本书都会向你展示如何做到这一点。

# **本书涵盖的内容**

第一章，*学习服务工作者基础*，涵盖了在你的环境中设置服务工作者，以及如何使用服务工作者开发来启动和运行。本章包括注册服务工作者和调试。

第二章，*处理资源文件*，提供了关于如何使用服务工作者处理资源文件的几个食谱，包括加载 CSS 和字体。

第三章，*访问离线内容*，探讨了如何缓存资源并离线提供服务。

第四章，*使用高级技术访问离线内容*，探讨了处理离线内容时的高级技术，包括模板和 Google Analytics。

第五章，*超越离线缓存*，提供了超越离线缓存的食谱，并探讨了从获取离线网络响应到如何将服务工作者用作负载均衡器的主题。

第六章，*使用高级库*，讨论了 Google Analytics、断路器和死信队列。

第七章，*获取资源*，涵盖了从不同来源获取资源的各种技术。

第八章，*尝试 Web 推送*，讨论了实现推送通知的不同方法。

第九章，*查看一般用法*，提供了关于服务工作者一般用法（从慢响应到实时流程图）的各种食谱。

第十章，*提高性能*，讨论了如何优化你的服务工作者应用程序以高效和性能良好的方式运行。

# 你需要这本书的内容

这本书是在使用 Mac 和 Google Chrome 浏览器，运行 Node.js 的环境下编写的。然而，Node.js 也可以在 Windows 或 Linux 机器上运行，同时配合 Google Chrome。

本书使用的所有软件都是免费和开源的。你肯定需要运行 Node.js 和 Google Chrome 来执行大多数食谱。

# 术语约定

在这本书中，你会发现许多不同风格的文本，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、文件夹名称、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称按照以下方式显示：“`skipWaiting()`方法在活动事件处理程序中使用，而后者又使用`Clients.claim()`。”

代码块按照以下方式设置：

```js
self.oninstall = function(event) {
  event.waitUntil(
    fetch(zipURL)
      .then(function(res) {
        return res.arrayBuffer();
      })
      .then(getZipFileReader)
      .then(cacheFileContents)
      .then(self.skipWaiting.bind(self))
  );
};
```

任何命令行输入或输出都按照以下方式编写：

```js
$ git add –all
$ git commit -m "initial commit"
$ git push -u origin master

```

**新术语**和**重要词汇**以粗体显示。屏幕上、菜单或对话框中看到的单词，例如，在文本中显示如下：“最后，在左侧边栏中，选择**凭据**。”

# 读者反馈

我们欢迎读者的反馈。告诉我们您对这本书的看法——您喜欢或不喜欢什么。读者反馈对我们很重要，因为它帮助我们开发出您真正能从中获得最大价值的标题。

要向我们发送一般反馈，请简单地发送电子邮件至`<feedback@packtpub.com>`，并在邮件主题中提及书籍标题。

如果您在某个主题上具有专业知识，并且您有兴趣撰写或为书籍做出贡献，请参阅我们的作者指南，网址为[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您已经是 Packt 图书的骄傲拥有者，我们有一些事情可以帮助您充分利用您的购买。

## 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)上的账户下载此书的示例代码文件。如果您在其他地方购买了此书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的**支持**选项卡上。

1.  点击**代码下载与勘误**。

1.  在**搜索**框中输入书籍名称。

1.  选择您想要下载代码文件的书籍。

1.  从下拉菜单中选择您购买此书的来源。

1.  点击**代码下载**。

您也可以通过点击 Packt Publishing 网站上的书籍网页上的**代码文件**按钮下载代码文件。您可以通过在**搜索**框中输入书籍名称来访问此页面。请注意，您需要登录到您的 Packt 账户。

文件下载完成后，请确保您使用最新版本的以下软件解压缩或提取文件夹：

+   Windows 上的 WinRAR / 7-Zip

+   Mac 上的 Zipeg / iZip / UnRarX

+   Linux 上的 7-Zip / PeaZip

此书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Service-Worker-Development-Cookbook`](https://github.com/PacktPublishing/Service-Worker-Development-Cookbook)。我们还有其他来自我们丰富图书和视频目录的代码包可供使用，网址为[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)。查看它们吧！

## 勘误

尽管我们已经尽一切努力确保我们内容的准确性，但错误仍然可能发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。

要查看之前提交的勘误，请访问 [`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将出现在**勘误**部分下。

## 盗版

在互联网上对版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视保护我们的版权和许可证。如果您在互联网上发现任何形式的我们作品的非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过 `<copyright@packtpub.com>` 联系我们，并提供疑似盗版材料的链接。

我们感谢您在保护我们作者和我们提供有价值内容的能力方面的帮助。

## 问题和建议

如果您对本书的任何方面有问题，您可以联系我们的 `<questions@packtpub.com>`，我们将尽力解决问题。
