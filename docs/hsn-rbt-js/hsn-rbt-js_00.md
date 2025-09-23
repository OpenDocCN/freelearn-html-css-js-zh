# 前言

在硬件和嵌入式设备编程中使用 JavaScript 的使用量急剧增加。JavaScript 有一套有效的框架和库，支持机器人生态系统。

使用 JavaScript 进行动手机器人编程从设置一个用于用 JavaScript 编程机器人的环境开始。然后，您将深入构建基本级别的项目，如跟随线机器人。您将完成一系列项目，这些项目将教会您关于 Johnny-Five 库的知识，并在每个项目中提高您的技能。随着您通过章节的进展，您将创建一个闪烁的 LED，然后转向传感器和其他更高级的概念。然后，您将构建一个高级级别的 AI 机器人，将他们的 NodeBots 连接到互联网，创建一个 NodeBots 集群，并探索 MQTT。

在本书结束时，您将通过使用 JavaScript 构建机器人获得实际操作经验。

# 本书面向的对象

使用 JavaScript 进行动手机器人编程适合那些有树莓派 3 使用经验并且喜欢用 JavaScript 编写草图的人。对 JavaScript 和 Node.js 的基本了解将帮助您充分利用本书。

# 本书涵盖的内容

第一章，*设置您的开发环境*，本章涵盖了树莓派及其使用和设置方法。这包括在 SD 卡上设置 Raspbian，安装 Node.js，安装 Johnny-Five 库，以及安装 Raspi-IO 库。本章还将解释 Johnny-Five 和 Raspi-IO 的总体概念，以及使用 JavaScript 进行机器人编程带来的好处。

第二章，*创建您的第一个 Johnny-Five 项目*，在本章中，读者将构建他们用于 Johnny-Five 项目的开发环境，并创建他们的第一个 Johnny-Five 项目：一个闪烁的 LED

第三章，*使用 RGB LED 构建交互式项目*，在本章中，读者将通过 LED 介绍数字和 PWM IO 引脚—they will create a couple of projects with multiple LEDs and an RGB LED and explore fully the Johnny-Five LED API. 我们还将探讨通过包含颜色库来控制 RGB LED 的颜色，从而包含外部 Node.js 库。

第四章，*通过按钮引入输入*，在本章中，我们将向用户展示如何通过按钮将基本输入集成到他们的项目中。读者将学习通过使用 Johnny-Five 按钮事件来跟踪按钮。

第五章，*使用光传感器创建夜灯*，在本章中，我们将添加一个传感器并创建我们的第一个实际项目：夜灯！夜灯将通过读取光传感器来工作，如果房间明亮，则关闭 LED 灯，但如果房间昏暗，则点亮它！我们还将讨论传感器在机器人项目生态系统中的作用。

第六章，*使用电机移动你的项目*，在本章中，我们将讨论如何使用电机使你的项目移动。这包括你将需要的额外硬件，以确保你的项目能够正确供电，如何将电机连接到你的项目，johnny-five 电机 API，以及在使用 Raspberry pi 时使用电机时常见问题的故障排除。

第七章，*使用伺服器进行精确运动*，在本章中，我们将讨论使用伺服器在机器人项目中实现精确运动，并构建一个对传感器做出响应的伺服器。读者将了解伺服器和 Johnny-Five 伺服器 API，并构建一个包含伺服器的项目。他们还将了解在制作移动机器人项目时伺服器和电机的区别。

第八章，*动画库*，动画库是通过对速度、加速度曲线以及伺服器运动的起始点和结束点进行控制，以精细调整对 johnny-five 伺服器项目的控制的一种好方法。在本章中，我们将研究动画库，并介绍如何控制精确的伺服器运动。

第九章，*获取所需信息*，在本章中，我们将探讨为什么你想将你的 NodeBots 项目连接到互联网。我们将从查看你可以使用 GET 请求从网站获取信息的方式开始；比如你所在地区的天气预报。我们将使用天气数据和 RGB LED 构建我们的第一个互联网连接机器人。

第十章，*使用 MQTT 与互联网上的事物进行通信*，在本章中，我们将讨论 MQTT，这是一种常见的物联网通信协议。我们将研究使用 MQTT 代理，用我们的 NodeBot 订阅它，并使用这些数据实时做出反应。

第十一章，*构建 NodeBots 集群*，在最后一章中，我们将探讨继续你的 NodeBots 之旅的可能路径。我们将研究如何将前几章中开发的技能应用于构建多个连接的 NodeBots，以及探索 Johnny-Five 库中留下的探索途径。

# 为了充分利用这本书

为了构建书中包含的所有项目，你需要以下基本物品：

+   带有任意操作系统的笔记本电脑

+   Raspberry Pi3

+   Micro SD 卡（至少 8 GB）

+   面板和电线

+   Adafruit DC 和步进电机 HAT for Raspberry Pi - 迷你套件

+   Raspberry Pi 3 T-Cobbler

+   GPIO 扩展板

+   将 LCD 连接到您的 Pi

+   直流玩具/业余电机 - 130 尺寸

+   4 x AA 电池盒带开关

关于要求的更多细节，硬件和软件的设置在每个章节的“技术要求”部分都有解释。

# 下载示例代码文件

您可以从 [www.packtpub.com](http://www.packtpub.com) 的账户下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问 [www.packtpub.com/support](http://www.packtpub.com/support) 并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在 [www.packtpub.com](http://www.packtpub.com/support) 上登录或注册。

1.  选择“支持”选项卡。

1.  点击代码下载和勘误。

1.  在搜索框中输入书籍名称，并遵循屏幕上的说明。

文件下载完成后，请确保使用最新版本的以下软件解压缩或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书代码包也托管在 GitHub 上，网址为 [`github.com/PacktPublishing/Hands-On-Robotics-with-JavaScript`](https://github.com/PacktPublishing/Hands-On-Robotics-with-JavaScript)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还提供其他代码包，这些代码包来自我们丰富的书籍和视频目录，可在 **[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)** 上找到。查看它们！

# 下载彩色图像

我们还提供了一份包含本书中使用的截图/图表彩色图像的 PDF 文件。您可以从这里下载：[`www.packtpub.com/sites/default/files/downloads/HandsOnRoboticswithJavaScript_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/HandsOnRoboticswithJavaScript_ColorImages.pdf)。

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 账号。以下是一个示例：“将下载的 `WebStorm-10*.dmg` 磁盘镜像文件挂载为系统中的另一个磁盘。”

代码块设置如下：

```js
board.on("ready", function() {
  // Everything else goes in here!
});
```

任何命令行输入或输出都应如下所示：

```js
sudo node implicit-animations.js
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“要设置一个源，请在左侧菜单中选择“源”。”

警告或重要注意事项看起来像这样。

小贴士和技巧看起来像这样。

# 联系我们

欢迎读者反馈。

**一般反馈**：请发送电子邮件至 `feedback@packtpub.com`，并在邮件主题中提及书籍名称。如果您对本书的任何方面有疑问，请发送电子邮件至 `questions@packtpub.com`。

**勘误**：尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将不胜感激，如果您能向我们报告这一情况。请访问 [www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上遇到任何形式的我们作品的非法副本，我们将不胜感激，如果您能向我们提供位置地址或网站名称。请通过发送链接至 `copyright@packtpub.com` 与我们联系。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且对撰写或参与一本书籍感兴趣，请访问 [authors.packtpub.com](http://authors.packtpub.com/).

# 评论

请留下评论。一旦您阅读并使用了这本书，为何不在您购买它的网站上留下评论呢？潜在读者可以查看并使用您的客观意见来做出购买决定，我们 Packt 可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

如需了解更多关于 Packt 的信息，请访问 [packtpub.com](https://www.packtpub.com/).
