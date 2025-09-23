# 前言

实时网络应用传统上是一个具有挑战性的目标，依赖于黑客手段和幻觉。许多人因为认为其复杂性而避免实现实时性。本书将向您展示如何使用 Socket.IO 构建现代实时网络应用，并介绍 Socket.IO 的各种功能，并指导您进行聊天服务器的开发、托管和扩展。

# 本书涵盖的内容

第一章, *在网络上实现实时性*，介绍了实时网络应用及其历史。

第二章, *开始使用 Node.js*，介绍了 Node.js 及其相关技术。Node.js 是许多现代网络应用的平台，这些应用都是用 JavaScript 编写的。

第三章, *让我们聊天*，使我们能够启动我们的第一个单页聊天系统，并介绍了 Socket.IO API 用于实时通信。

第四章, *让它更有趣！*，为我们的聊天应用添加了更多功能，例如给我们的用户提供一个名字，拥有多个聊天室，以及将即时消息与 Socket.IO 会话集成。

第五章, *Socket.IO 协议,* 解释了 Socket.IO 协议、其机制和工作原理。

第六章, *部署和扩展*，解释了将我们的聊天系统投入生产并扩展其规模所涉及的复杂性。

附录 A, *Socket.IO 快速参考*，是 Socket.IO API 的参考指南。

附录 B, *Socket.IO 后端*，列出了针对不同语言和平台的几种替代后端实现。

# 你需要这本书的内容

使用本书时，我们不假设任何特殊的软件要求。您需要一个装有 Linux 或 Windows OS 或 Mac 的 PC。您可以使用任何文本编辑器进行编码，但拥有如 Vi、Emacs、Notepad++、Sublime Text 或您选择的任何 IDE 的程序员编辑器将有所帮助。我们将随着本书的进展和需要时安装剩余的软件，如 Node.js 和 npm。

# 本书面向的对象

本书面向希望开始开发高度交互性和实时网络应用（如聊天系统、在线多人游戏）或希望在现有应用中引入实时更新或服务器推送机制的开发者。预期读者具备 JavaScript 开发和一般网络应用的知识。尽管本书有一章介绍 Node.js，但具备 Node.js 的先验知识将是一个加分项。读者需要能够运行 Node.js 的计算机系统，一个测试或代码编辑器，以及访问互联网以下载所需的软件和组件。

# 习惯用法

在这本书中，您将找到多种文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词如下所示：“为了设置我们的节点服务器运行的环境，我们将环境变量`NODE_ENV`设置为我们要在其中运行节点的环境。”

代码块设置如下：

```js
<!DOCTYPE html>
<html>
<head>
<title>{TITLE}</title>
<link rel="stylesheet" href="/stylesheets/style.css" />
</head>
<body>
<header id="banner">
<h1>Awesome Chat</h1>
</header>
    {CONTENT}
<footer>
      Hope you enjoy your stay here
</footer>
</body>
</html>
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```js
doctype 5 
html 
 block head 
    title= title 
    link(rel='stylesheet', href='/stylesheets/style.css') 
  body 
header#banner
h1 Awesome Chat 
    block content 
    footer Hope you enjoy your stay here
```

任何命令行输入或输出都应如下编写：

```js
$ express awesome-chat
$ cd awesome-chat
$ npm install

```

**新术语**和**重要词汇**将以粗体显示。您在屏幕上看到的单词，例如在菜单或对话框中，将以如下方式显示：“现在您可以在浏览器中的一个消息框中输入您的消息，然后点击**发送**。您将看到它在两个浏览器的消息区域中显示。”

### 注意

警告或重要注意事项将以如下框显示。

### 小贴士

技巧和窍门将以如下方式显示。

# 读者反馈

我们欢迎读者的反馈。请告诉我们您对这本书的看法——您喜欢什么或可能不喜欢什么。读者反馈对我们来说非常重要，以便我们开发出您真正能从中获得最大收益的标题。

要向我们发送一般反馈，只需发送一封电子邮件到 `<feedback@packtpub.com>`，并在邮件主题中提及书名。

如果您在某个主题领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请参阅我们的作者指南，网址为 [www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 书籍的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从您在 [`www.packtpub.com`](http://www.packtpub.com) 的账户下载您购买的所有 Packt 书籍的示例代码文件。如果您在其他地方购买了这本书，您可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 并注册，以便将文件直接通过电子邮件发送给您。

## 错误清单

尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以避免其他读者感到沮丧，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站，或添加到该标题的勘误部分下的现有勘误列表中。您可以通过从[`www.packtpub.com/support`](http://www.packtpub.com/support)选择您的标题来查看任何现有勘误。

## 盗版

互联网上版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现我们作品的任何非法副本，无论形式如何，请立即向我们提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过发送链接到疑似盗版材料至`<copyright@packtpub.com>`与我们联系。

我们感谢您在保护我们的作者以及为我们提供有价值内容的能力方面的帮助。

## 问题

如果您在本书的任何方面遇到问题，可以通过`<questions@packtpub.com>`与我们联系，我们将尽力解决。
