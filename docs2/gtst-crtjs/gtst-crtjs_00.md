# 前言

CreateJS 是一个功能齐全的工具，网络开发者可以使用它来在网页浏览器中使用 HTML5 创建和维护动画和绘图。它由一些模块组成，每个模块执行不同的任务来管理基于 Web 的应用程序。

在本书中，我们将通过许多交互式示例和截图讨论 CreateJS 的不同部分。

# 本书涵盖的内容

第一章, *安装 CreateJS*，为初学者提供了快速安装参考。

第二章, *开始使用 CreateJS*，介绍了如何开始使用 CreateJS 以及其他组件，使用 API 以及配置模块。

第三章, *使用拖放交互*，讨论了 CreateJS 的拖放功能以及如何在项目中自定义这些功能或扩展它们。

第四章, *执行动画和变换功能*，介绍了如何使用 CreateJS 在页面上使用动画和变换对象。

第五章, *在 EaselJS 中使用缓存*，介绍了如何在 CreateJS 中使用缓存以实现动画中的更好性能。

第六章, *在 EaselJS 中使用滤镜*，讨论了在 CreateJS 中的图像和对象上使用滤镜。

第七章, *开发绘画应用程序*，讨论了如何使用图形在画布上开发一个简单的绘画应用程序。

第八章, *利用矢量蒙版*，讨论了将形状用作任何显示对象的剪切路径。

第九章, *开发您的第一个 CreateJS 应用程序*，介绍了如何使用 CreateJS 从零开始构建和创建 UI。

# 您需要为本书准备什么

本书中包含的所有示例和源代码都在现代网络浏览器中运行。为了所有示例都能完美运行，您需要安装最新版本的 Google Chrome 或 Mozilla Firefox。

# 本书面向对象

如果您是一位有 JavaScript 开发经验的网络开发者，并希望进入使用 CreateJS 的功能丰富的互联网应用迷人世界，那么这本书非常适合您。

# 术语约定

在本书中，您将找到多种文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码词显示如下："在 EaselJS 中，当您有一个形状，或者更好的是，一个不经常改变的 `DisplayObject` 实例时，最好使用 `cache` 函数将其缓存到不同的 `Canvas` 元素中"。

代码块设置如下：

```js
var circle = new createjs.Shape();
circle.graphics.beginFill("red").drawCircle(0, 0, 50);
circle.x = 100; 
circle.y = 100; 
```

任何命令行输入或输出都应如下所示：

```js
# Install the grunt command line utility globally 
npm install grunt-cli -g 

# Install all the dependencies from package.json
npm install

```

**新术语**和**重要词汇**将以粗体显示。你在屏幕上看到的，例如在菜单或对话框中的文字，将以如下方式显示：“使用下拉菜单，用户可以更改**画笔样式**、**画笔大小**、**背景颜色**和**画笔颜色**字段。”

### 注意

警告或重要注意事项将以如下框中的方式显示。

### 小贴士

小贴士和技巧看起来像这样。

# 读者反馈

我们始终欢迎读者的反馈。告诉我们您对这本书的看法——您喜欢什么或可能不喜欢什么。读者反馈对我们开发您真正从中获得最大收益的标题非常重要。

要向我们发送一般反馈，只需发送电子邮件到`<feedback@packtpub.com>`，并在邮件主题中提及书籍标题。

如果您在某个主题上具有专业知识，并且您对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您已经是 Packt 书籍的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)的账户中下载您购买的所有 Packt 书籍的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 下载本书的彩色图像

我们还为您提供了一个包含本书中使用的截图/图表彩色图像的 PDF 文件。彩色图像将帮助您更好地理解输出中的变化。您可以从以下链接下载此文件：[`www.packtpub.com/sites/default/files/downloads/0260OS_coloredimages.pdf`](https://www.packtpub.com/sites/default/files/downloads/0260OS_coloredimages.pdf)。

## 勘误

尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情来报告。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站，或添加到该标题的勘误部分下的现有勘误列表中。您可以通过从[`www.packtpub.com/support`](http://www.packtpub.com/support)选择您的标题来查看任何现有勘误。

## 侵权

互联网上对版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现我们作品的任何非法副本，无论形式如何，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过`<copyright@packtpub.com>`与我们联系，并提供疑似盗版材料的链接。

我们感谢您在保护我们作者以及为我们提供有价值内容的能力方面的帮助。

## 问题

如果您在本书的任何方面遇到问题，可以通过`<questions@packtpub.com>`联系我们，我们将尽力解决。
