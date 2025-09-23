# 前言

遍布各处的机器人！对话和聊天功能的应用程序正处于成为下一个大平台的最佳位置，以及应用程序是如何开发的。机器学习的发展，自然对话 API 的兴起，许多高科技和软件巨头都在拥抱对话机器人，并提供 API，允许开发者创建无缝集成到这些对话平台中的应用程序，从而提升用户体验。本书以简单直观的方式探讨了这些平台，使开发者能够快速掌握它们。

# 本书涵盖的内容

*第一章*，*机器人的崛起 – 传达信息*，介绍了并解释了机器人今天世界中的日益重要性，并教您如何使用 Twilio 消息平台创建一个短信机器人应用程序。

*第二章*，*让 Skype 为您服务*，解释了如何使用新的 Microsoft Bot Framework 来创建一个 Skype 机器人。

*第三章*，*Twitter 作为航班信息代理*，教您如何创建一个 Twitter 机器人应用程序，该应用程序可以与 Air France KLM API 交互以检索航班详情。

*第四章*，*Slack 引言机器人*，解释了如何创建一个 Slack 机器人应用程序，该应用程序可以向用户发送励志名言。

*第五章*，*Telegram 驱动的机器人*，展示了如何开发一个机器人，该机器人将使用 Telegram API 为您提供 Telegram 上的消息情感。

*第六章*，*BotKit – Slack 的文档管理代理*，教您如何使用 BotKit 与 Slack API 结合，为在 Slack 中协作的团队成员提供触手可及的文档。

*第七章*，*Facebook Messenger 机器人，谁在休息 – 团队调度机器人*，展示了如何设置一个 Facebook Messenger 机器人，该机器人可以使用 Microsoft Azure 平台和服务来安排团队会议或查看谁在休息。

*第八章*，*团队缺陷跟踪代理*，教您如何使用 IRC 平台和 DocumentDB 来创建一个缺陷跟踪机器人。

*第九章*，*Kik Salesforce CRM 机器人*，探讨了如何使用 Force.com API 和 Kik 创建 Salesforce CRM 机器人。

# 您需要为本书准备的内容

以下要求有助于获得最佳体验：

+   良好的互联网连接

+   一台相当现代的计算机或笔记本电脑（最好是 Windows 系统，但不一定是）

+   需要相当程度的创造力、想象力和愿意学习和探索新概念的态度

本书提到的几乎所有软件和 API 都是免费的，并且可以从互联网上下载。

# 本书面向的对象

本书适合任何了解一些 Node.js 并希望探索如何使用今天可用的各种现有对话平台编写机器人的读者。本书以易于理解的方式编写，适合所有类型的开发者，从新手到经验丰富的开发者。建议具备一些 Node.js 知识。

# 约定

在本书中，您将找到许多文本样式，用于区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称将如下所示：“`session`对象包含 Skype 传递给机器人应用程序的信息，它描述了已接收的会话数据以及来自谁。请注意，会话对象包含`message`和`text`等属性。”

代码块将如下设置：

```js
bot.dialog('/', function (session) { 
  if (session.message.text.toLowerCase().indexOf('hi') >= 0){ 
    session.send('Hi ' + session.message.user.name +  
     ' thank you for your message: ' + session.message.text); 
  } else{ 
    session.send('Sorry I dont understand you...'); 
  } 
});
```

任何命令行输入或输出将如下所示：

```js
mkdir telegrambot
cd telegrambot

```

新术语和重要词汇以粗体显示。屏幕上看到的单词，例如在菜单或对话框中，在文本中会像这样显示：“点击**注册机器人**选项以创建和注册您的 Skype 机器人。完成此操作后，您将看到以下屏幕：”

### 注意

警告或重要提示会以这样的框出现。

### 小贴士

小贴士和技巧会像这样出现。

# 读者反馈

读者的反馈总是受欢迎的。请告诉我们您对这本书的看法——您喜欢或不喜欢的地方。读者反馈对我们来说很重要，因为它帮助我们开发出您真正能从中获得最大收益的标题。

要向我们发送一般反馈，只需通过 feedback@packtpub.com 发送电子邮件，并在邮件主题中提及书籍的标题。

如果您在某个主题上具有专业知识，并且您对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您已经是 Packt 图书的骄傲拥有者，我们有一些事情可以帮助您从购买中获得最大收益。

## 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)下载您购买的所有 Packt 出版物的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  使用您的电子邮件地址和密码登录或注册我们的网站。

1.  将鼠标指针悬停在顶部的**支持**标签上。

1.  点击**代码下载与勘误表**。

1.  在**搜索**框中输入书籍名称。

1.  选择您想要下载代码文件的书籍。

1.  从下拉菜单中选择您购买此书籍的来源。

1.  点击**代码下载**。

您也可以通过点击 Packt Publishing 网站上书籍网页上的**代码文件**按钮来下载代码文件。您可以通过在搜索框中输入书籍名称来访问此页面。请注意，您需要登录到您的 Packt 账户。

文件下载完成后，请确保使用最新版本的以下软件解压缩或提取文件夹：

+   WinRAR / 7-Zip（适用于 Windows）

+   Zipeg / iZip / UnRarX（适用于 Mac）

+   7-Zip / PeaZip（适用于 Linux）

本书的相关代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Building-Bots-with-Nodejs`](https://github.com/PacktPublishing/Building-Bots-with-Nodejs)。我们还有其他来自我们丰富图书和视频目录的代码包可供在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)找到。查看它们吧！

## 下载本书的颜色图像

我们还为您提供了一个包含本书中使用的截图/图表的颜色图像的 PDF 文件。这些颜色图像将帮助您更好地理解输出的变化。您可以从以下网址下载此文件：[`www.packtpub.com/sites/default/files/downloads/BuildingBotswithNodejs_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/BuildingBotswithNodejs_ColorImages.pdf)。

## 勘误

尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以避免其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。

要查看之前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将出现在勘误部分下。

## 海盗行为

互联网上对版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现我们作品的任何形式的非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过版权@packtpub.com 与我们联系，并提供涉嫌盗版材料的链接。

我们感谢您在保护我们作者和为我们提供有价值内容的能力方面提供的帮助。

## 问题

如果您在这本书的任何方面遇到问题，您可以联系 questions@packtpub.com，我们将尽力解决问题。
