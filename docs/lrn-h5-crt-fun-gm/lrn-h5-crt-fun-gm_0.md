# 前言

如果您想编写一款可以触及全球数十亿人的软件，那么这本书将帮助您开始这段旅程。如今，人们每天使用的大多数设备（计算机，笔记本电脑，平板电脑，智能手机等）都能够运行 HTML5 代码。而且，随着现代 Web 浏览器继续变得越来越强大，您基于 HTML5 的游戏和应用程序可以以本机应用程序性能水平或非常接近本机应用程序性能水平运行。

本书将帮助您了解 HTML5 的所有内容，包括语义标记元素，CSS3 样式和最新的支持 JavaScript API。有了这些知识和技能，我们将能够创建可以由连接到互联网的设备上的任何人玩的有趣游戏。

# 本书内容

第一章，*HTML5 概述*，解释了 HTML5 是什么，以及它如何适应开放 Web 平台范例。它还介绍了 HTML5 的三大支柱，即新的 HTML 元素，CSS3 和新的 JavaScript API。

第二章，*HTML5 排版*，介绍了本书中的第一个游戏，即基于 DOM 的排版游戏。本章描述的主要 HTML5 功能包括 Web 表单，元数据，Web 字体，过渡，动画，文本阴影，框阴影，window.JSON 和 querySelector。

第三章，*理解 HTML5 的引力*，构建了一个基本的果冻摇摆引力游戏。本章包括跨浏览器支持，polyfill 的讨论，以及如何解决不同浏览器之间的 API 实现差异。本章描述的主要 HTML5 功能包括 Web 音频，SVG 图形和拖放。

第四章，*使用 HTML5 捕捉蛇*，使用新的 HTML5 画布元素创建了一个传统的贪吃蛇游戏，以及其伴随的 2D 渲染上下文。本章描述的其他 HTML5 功能包括 Web Workers，离线存储和 RequestAnimationFrame。

第五章，*改进贪吃蛇游戏*，在上一章中创建的相同游戏基础上添加了窗口消息传递，Web 存储，本地存储，会话存储和 IndexedDB 等功能。

第六章，*为您的游戏添加功能*，重点讨论了高级 HTML5 概念，以及最新功能。尽管本章没有构建游戏，但所描述的 JavaScript 和 CSS API 代表了 HTML5 和 Web 开发的最新技术。本章描述的主要功能包括 WebGL，Web 套接字，视频，地理位置，CSS 着色器，CSS 列和 CSS 区域和排除。

第七章，*HTML5 和移动游戏开发*，通过构建一个完全针对移动游戏玩法进行优化的 2D 太空射击游戏来结束本书。本章的重点是 Web 开发中移动特定的考虑因素，包括讨论桌面和移动平台之间的差异。本章描述的主要 HTML5 功能包括媒体查询和触摸事件。

*设置环境*，介绍了本地 Web 开发环境的设置，包括安装开源 Apache 服务器。除了设置开发环境外，它还演示了如何使用新的 HTML5 元素构建 Web 门户，通过该门户我们可以访问本书中开发的游戏。该章节可在线获取：[`www.packtpub.com/sites/default/files/downloads/Setting_up_the_Environment.pdf`](http://www.packtpub.com/sites/default/files/downloads/Setting_up_the_Environment.pdf)。

# 本书所需内容

您需要最新版本的现代网络浏览器，目前包括 Google Chrome，Mozilla Firefox，Safari，Opera 和 Internet Explorer（至少版本 10）。您还需要选择的基本文本编辑器，尽管您熟悉的任何代码编辑软件也可以。具有 HTML、CSS 和 JavaScript 的先验知识或经验是有帮助的，但不是必需的。

# 这本书是为谁写的

这本书主要是为有游戏开发经验的开发人员编写的，他们现在正在转向 HTML5。本书的重点不是游戏开发的复杂性和理论，而是帮助读者学习 HTML5，以及开放网络平台如何成为触达全球数十亿用户的手段。

# 约定

在本书中，您会发现一些文本样式，用于区分不同类型的信息。以下是一些这些样式的示例，以及它们的含义解释。

文本中的代码单词显示如下：“我们可以通过使用`include`指令来包含其他上下文。”

代码块设置如下：

```js
[<div id="wrapper">
  <div id="header"></div>
  <div id="body">
    <div id="main_content">
      <p>Lorem Ipsum...</p>
    </div>
    <div id="sidebar"></div>
  </div>
  <div id="footer"></div>
</div>
```

当我们希望引起您对代码块的特定部分的注意时，相关的行或项目会以粗体显示：

```js
<input type="text" name="firstName" value="First Name" class="hint-on"
 onblur="if (this.value == '') {

```

**新术语**和**重要单词**以粗体显示。例如，屏幕上看到的单词，菜单或对话框中的单词会在文本中显示为：“点击**下一步**按钮会将您移动到下一个屏幕”。

### 注意

警告或重要提示会以这样的方式出现在框中。

### 提示

提示和技巧会以这样的方式出现。
