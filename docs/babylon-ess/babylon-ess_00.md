# 前言

3D 开发一直是一件神秘的事情。从开始讲起，这本书介绍了 3D 开发所需的基本知识以及使用 Babylon.js 框架所需的知识。它侧重于 Babylon.js 提供的简洁性，并结合理论和实践。所有章节都提供了可运行的示例文件；每个示例文件都提供了之前学到的框架功能。最后，开发者将能够轻松理解未来添加到框架中的新功能，并仅使用文档来使用更高级的功能。

# 本书涵盖内容

第一章, *Babylon.js 和 TypeScript 语言*，快速介绍了 Babylon.js 的故事，并开设了一门关于 TypeScript 语言的基础课程。

第二章, *Babylon.js 和可用工具的基础知识*，从 Babylon.js 框架开始，创建第一个 3D 场景，展示了框架的简洁性以及理论。

第三章, *在屏幕上创建、加载和绘制 3D 对象*，从 第二章 的概念开始，介绍创建 3D 场景和与 3D 艺术家合作的正确方法。

第四章, *使用材质自定义 3D 对象的外观*，解释了 3D 引擎中材质的概念。换句话说，让我们释放 Babylon.js 标准材质的潜力。

第五章, *在对象上创建碰撞*，通过管理场景中的碰撞，包括物理模拟，专注于游戏玩法本身。

第六章, *在 Babylon.js 中管理音频*，解释了 Babylon.js 框架的一个增值功能。让我们在场景中添加和管理声音，包括空间化声音。

第七章, *在对象上定义动作*，介绍了一种智能的方法，可以在不写太多代码的情况下触发 3D 对象上的动作，因为一些任务对开发者来说可能很痛苦。

第八章, *使用内置后处理添加渲染效果*，展示了大多数 3D 开发者的首选部分。这展示了如何使用后处理效果轻松美化 3D 场景，结合 Babylon.js 提供的简洁性。

第九章, *创建并播放动画*，让我们能够使用 Babylon.js 提供的动画系统进行操作。本章提供了你准备构建自己的专业 3D 应用程序所需的最终技能！

# 您需要为这本书准备什么

本书针对希望在其浏览器上构建完整 3D 视频游戏或应用程序的 HTML5 开发者。读者应熟悉 JavaScript 语言和基本 3D 表示（向量和维度的概念）。Babylon.js 框架设计得易于使用，因此，不需要 3D 开发背景，欢迎初学者。

# 这本书面向的对象

Babylon.js Essentials 旨在为希望进入 Web 3D 开发世界或将其 Babylon.js 框架添加到技能集的开发者设计。理解面向对象编程的概念将有助于理解 Babylon.js 框架的架构。此外，熟悉 Web 开发将有助于理解所使用的原则。

# 约定

在这本书中，您将找到许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称如下所示：“使用 TS，输入新的`Array()`相当于新的`Array<any>()`。”

代码块设置如下：

```js
var skybox = BABYLON.Mesh.CreateBox("skybox", 300, scene); 
var skyboxMaterial = new BABYLON.StandardMaterial("skyboxMaterial", scene); 
skyboxMaterial.backFaceCulling = false; 
skyboxMaterial.reflectionTexture = 
  new BABYLON.CubeTexture("skybox/TropicalSunnyDay", scene);
```

任何命令行输入或输出都应如下所示：

```js
var myVar: FileAccess = FileAccess.Read; // Equivalent to 0
```

新术语和重要单词以粗体显示。您在屏幕上看到的单词，例如在菜单或对话框中，在文本中如下所示：“选择文件位置，就像您为 Blender 保存 Babylon.js 场景时所做的，然后点击**导出**。”

### 注意

警告或重要注意事项以如下框中的形式出现。

### 小贴士

小贴士和技巧看起来像这样。

# 读者反馈

我们欢迎读者的反馈。请告诉我们您对这本书的看法——您喜欢什么或不喜欢什么。读者反馈对我们很重要，因为它帮助我们开发出您真正能从中获得最大价值的标题。

要向我们发送一般反馈，只需发送电子邮件至`<feedback@packtpub.com>`，并在邮件主题中提及书的标题。

如果您在某个主题上具有专业知识，并且您对撰写或为本书做出贡献感兴趣，请参阅我们的作者指南[`www.packtpub.com/authors`](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 图书的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大价值。

## 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)下载示例代码文件，这是您购买的所有 Packt Publishing 书籍的账户。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 下载本书中的彩色图像

我们还为您提供了一个包含本书中使用的截图/图表的彩色图像的 PDF 文件。彩色图像将帮助您更好地理解输出中的变化。您可以从[`www.packtpub.com/sites/default/files/downloads/BabylonJSEssentials_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/BabylonJSEssentials_ColorImages.pdf)下载此文件。

## 错误清单

尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在我们的某本书中找到错误——可能是文本或代码中的错误——如果您能向我们报告，我们将不胜感激。这样做可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何错误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**错误提交表单**链接，并输入您的错误详细信息来报告它们。一旦您的错误得到验证，您的提交将被接受，错误将被上传到我们的网站或添加到该标题的错误清单部分。

要查看之前提交的错误清单，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将出现在“错误清单”部分下。

## 盗版

互联网上版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现我们作品的任何非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过`<copyright@packtpub.com>`与我们联系，并提供指向疑似盗版材料的链接。

我们感谢您在保护我们作者和我们为您提供有价值内容的能力方面提供的帮助。

## 问题

如果您对本书的任何方面有问题，您可以通过`<questions@packtpub.com>`与我们联系，我们将尽力解决问题。
