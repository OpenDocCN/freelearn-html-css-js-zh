# 前言

树莓派 Zero 是一款功能强大、价格低廉、信用卡大小的计算机，非常适合开始控制复杂的家庭自动化设备。使用可用的板载接口，树莓派 Zero 可以扩展，允许连接几乎无限数量的安全传感器和设备。

由于 Arduino 平台更加多功能和有用于制作项目，包括物联网的网络应用，这就是我们将在本书中看到的：连接到节点的设备的集成，使用令人惊叹和重要的 Arduino 板，以及如何集成树莓派 Zero 来控制和监控设备，从而形成一个作为中心界面工作的中心。通过软件编程，您将创建一个基于 JavaScript、HTML5 和 Node.js 等发展技术的物联网系统。

这正是我将在这本书中教给你的。您将学习如何在几个家庭自动化项目中使用树莓派 Zero 板，以便您自己建立。

这本书指导您，每一章的项目都从准备领域、硬件、传感器、通信和软件编程控制开始，以便拥有完整的控制和监控系统。

# 本书涵盖的内容

第一章，“开始使用树莓派 Zero”，描述了设置树莓派和 Arduino 板的过程，以及设备之间的通信。我们将安装和设置操作系统，将我们的 Pi 连接到网络，并远程访问它。我们还将保护我们的 Pi，并确保它能保持正确的时间。

第二章，“将东西连接到树莓派 Zero”，展示了如何将信号连接到树莓派 Zero 和 Arduino。它探讨了 GPIO 端口及其各种接口。我们将看看可以使用 GPIO 连接到树莓派的各种东西。

第三章，“连接传感器-测量真实的事物”，展示了如何实现用于检测不同类型信号的传感器，用于安全系统、能源消耗的流量电流、家庭中的一些风险检测、实现气体传感器、流量水传感器来测量水量，并且还将展示如何制作一个将使用指纹传感器控制家庭入口的安全系统。

第四章，“控制连接设备”，展示了如何使用通信模块在树莓派 Zero 的网络领域中控制您的 Arduino 板，以及如何在中央界面仪表板中显示。

第五章，“添加网络摄像头监控您的安全系统”，展示了如何配置连接到您的板的网络摄像头，以监控物联网安全系统。

第六章，“构建 Web 监视器并从仪表板控制设备”，展示了如何设置一个系统来监控您的安全系统使用网络服务。将树莓派 Zero 与 Arduino 集成，构建一个完整的系统连接设备和监控。

第七章，“使用物联网仪表板构建间谍警察”，展示了如何制作不同的迷你家庭自动化项目，以及如何使用物联网连接网络服务和监控您的安全系统。

Chapter 8, *Monitor and Control your devices from a Smart Phone*, explains how to develop an app for Smart Phone using Android Studio and APP inventor, and control your Arduino board and the Raspberry Pi Zero.

Chapter 9, *Putting It All Together*, shows how to put everything together, all the parts of the project, the electronics field, software configurations, and power supplies.

# What you need for this book

You’ll need the following software:

+   Win32 Disk Imager 0.9.5 PuTTY

+   i2C-tools

+   WiringPi2 for Python

+   Node.js 4.5 or later

+   Node.js for Windows V7.3.0 or later

+   Python 2.7.x or Python 3.x

+   PHP MyAdmin Database

+   MySQL module

+   Create and account in Gmail so that you can get in APP Inventor

+   Android Studio and SDK modules

+   Arduino software

In the first chapters, we explain all the basics so you will have everything configured and will be able to use the Raspberry Pi Zero without any problems, so you can use it for the projects in this book. We will use some basic components, such as sensors, and move to more complex components in the rest of the book.

On the software side, it is good if you actually have some existing programming skills, especially in JavaScript and in the Node.js framework. However, I will explain all the parts of each software piece of this book, so even if you don't have good programming skills in JavaScript you will be able to follow along.

# Who this book is for

This book is for all the people who want to automate their homes and make them smarter, while at the same time having complete control of what they are doing. If that's your case, you will learn everything there is to learn in this book about how to use the amazing Raspberry Pi Zero board to control your projects.

This book is also for makers who have played in the past with other development boards, such as Arduino. If that's the case, you will learn how to use the power of the Raspberry Pi platform to build smart homes. You will also learn how to create projects that can easily be done with other platforms, such as creating a wireless security camera with the Pi Zero.

# Conventions

In this book, you will find a number of text styles that distinguish between different kinds of information. Here are some examples of these styles and an explanation of their meaning.

Code words in text, database table names, folder names, filenames, file extensions, pathnames, dummy URLs, user input, and Twitter handles are shown as follows: "Extract `2015-09-24-raspbian-jessie.img` to your Home folder."

A block of code is set as follows:

```js
# passwd
root@raspberrypi:/home/pi# passwd
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
        root@raspberrypi:/home/pi#
```

When we wish to draw your attention to a particular part of a code block, the relevant lines or items are set in bold:

```js
[default]
exten => s,1,Dial(Zap/1|30)
exten => s,2,Voicemail(u100)
**exten => s,102,Voicemail(b100)**
exten => i,1,Voicemail(s0)
```

Any command-line input or output is written as follows:

```js
**        sudo npm install express request**

```

**New terms** and **important words** are shown in bold. Words that you see on the screen, for example, in menus or dialog boxes, appear in the text like this:

"You can now just click on **Stream** to access the live stream from the camera."

### Note

Warnings or important notes appear in a box like this.

### Tip

Tips and tricks appear like this.
