# 第一章：介绍 HTML5 游戏

> 超文本标记语言 HTML 在过去几十年中一直在塑造互联网。它定义了内容在网页中的结构以及相关页面之间的链接。HTML 从 2 版到 HTML 4.1 版再到 XHTML 1.1 版不断发展。由于这些网络应用程序和社交网络应用程序，HTML 现在正在向 HTML5 迈进。
> 
> **层叠样式表**（**CSS**）定义了网页的视觉呈现方式。它为所有 HTML 元素和它们的状态（如悬停和激活）定义样式。
> 
> JavaScript 是网页的逻辑控制器。它使网页动态化，并在页面和用户之间提供客户端交互。它通过**文档对象模型**（**DOM**）访问 HTML。它通过应用不同的 CSS 样式来重新设计 HTML 元素。

这三个功能为我们带来了新的游戏市场，HTML5 游戏。有了它们的新力量，我们可以使用 HTML5 元素、CSS3 属性和 JavaScript 设计游戏在浏览器中玩耍。

在本章中，我们将：

+   发现 HTML5 中的新功能

+   讨论让我们对 HTML5 和 CSS3 如此兴奋的原因

+   看看其他人如何使用 HTML5 进行游戏设计

+   预览我们将在后面章节中构建的游戏

所以让我们开始吧。

# 在 HTML5 中发现新功能

HTML5 和 CSS3 中引入了许多新功能。在我们开始创建游戏之前，让我们概览一下这些新功能，看看我们如何使用它们来创建游戏。

## 画布

**Canvas**是 HTML5 元素，提供低级别的绘制形状和位图操作功能。我们可以将 Canvas 元素想象成一个动态图像标签。传统的`<img>`标签显示静态图像。无论图像是动态生成的还是静态加载自服务器，图像都是静态的，不会改变。我们可以将`<img>`标签更改为另一个图像源，或者对图像应用样式，但我们无法修改图像的位图上下文本身。

另一方面，Canvas 就像是一个客户端动态的`<img>`标签。我们可以在其中加载图像，在其中绘制形状，并通过 JavaScript 与之交互。

Canvas 在 HTML5 游戏开发中扮演着重要角色。这是本书的主要关注点之一。

## 音频

背景音乐和音效通常是游戏设计中的重要元素。HTML5 通过`audio`标签提供了本地音频支持。由于这一功能，我们不需要专有的 Flash Player 来播放 HTML5 游戏中的音效。我们将在第六章讨论`audio`标签的用法，*使用 HTML5 音频元素构建音乐游戏*。

## 地理位置

**地理位置**让网页获取用户计算机的纬度和经度。多年前，当每个人都在使用台式电脑上网时，这个功能可能并不那么有用。我们并不需要用户的道路级别的位置精度。我们可以通过分析 IP 地址获得大致位置。

如今，越来越多的用户使用强大的智能手机上网。Webkit 和其他现代移动浏览器都在每个人的口袋里。地理位置让我们设计移动应用程序和游戏，以便使用位置信息。

基于位置的服务已经在一些社交网络应用程序中使用，例如 foursquare ([`foursquare.com`](http://foursquare.com))和 Gowalla ([`gowalla.com`](http://gowalla.com))。这种基于位置的社交社区的成功创造了使用位置服务与我们的智能手机的趋势。

## WebGL

WebGL 通过在 Web 浏览器中提供一组 3D 图形 API 来扩展 Canvas 元素。该 API 遵循 OpenGL ES 2.0 的标准。WebGL 为 3D HTML5 游戏提供了一个真正的 3D 渲染场所。然而，在撰写本书时，并非所有浏览器都原生支持 WebGL。目前，只有 Mozilla Firefox 4、Google Chrome 和 WebKit 浏览器的夜间构建版本原生支持它。

为 WebGL 创建游戏的技术与通常的 HTML5 游戏开发有很大不同。在 WebGL 中创建游戏需要处理 3D 模型，并使用类似于 OpenGL 的 API。因此，本书不会讨论 WebGL 游戏开发。

来自 Google Body（[`bodybrowser.googlelabs.com`](http://bodybrowser.googlelabs.com)）的以下屏幕截图演示了他们如何使用 WebGL 显示一个响应用户输入的 3D 人体。

![WebGL](img/1260_01_01.jpg)

### 提示

LearningWebGL（[`learnwebgl.com`](http://learnwebgl.com)）提供了一系列关于开始使用 WebGL 的教程。如果您想要学习更多关于使用它的知识，这是一个很好的起点。

## WebSocket

WebSocket 是 HTML5 规范的一部分，用于将网页连接到套接字服务器。它为浏览器和服务器之间提供了事件驱动的连接。这意味着客户端不需要每隔一段时间轮询服务器以获取新数据。只要有数据更新，服务器就会将更新推送到浏览器。这个功能的一个好处是游戏玩家几乎可以实时互动。当一个玩家做了什么并将数据发送到服务器时，服务器将向每个其他连接的浏览器广播一个事件，以确认玩家刚刚做了什么。这创造了创建多人 HTML5 游戏的可能性。

### 注意

由于安全问题，Mozilla Firefox 和 Opera 现在暂时禁用了 WebSocket。Safari 和 Chrome 也可能在问题解决之前放弃对 WebSocket 的支持。您可以通过访问以下链接了解更多关于这个问题的信息：[`hacks.mozilla.org/2010/12/websockets-disabled-in-firefox-4/`](http://hacks.mozilla.org/2010/12/websockets-disabled-in-firefox-4/)。

## 本地存储

HTML5 为 Web 浏览器提供了持久的数据存储解决方案。

本地存储可以持久地存储键值对数据。即使浏览器终止，数据仍然存在。此外，数据不仅限于只能由创建它的浏览器访问。它对具有相同域的所有浏览器实例都是可用的。由于本地存储，我们可以在 Web 浏览器中轻松地本地保存游戏状态，如进度和获得成就。

HTML5 还提供了 Web SQL 数据库。这是一个客户端关系数据库，目前受 Safari、Chrome 和 Opera 支持。通过数据库存储，我们不仅可以存储键值对数据，还可以支持 SQL 查询的复杂关系结构。

本地存储和 Web SQL 数据库对我们在创建游戏时本地保存游戏状态非常有用。

除了本地存储，一些其他存储方法现在也得到了 Web 浏览器的支持。这些包括 Web SQL 数据库和 IndexedDB。这些方法支持根据条件查询存储的数据，因此更适合支持复杂的数据结构。

您可以在 Mozilla 的以下链接中找到更多关于使用 Web SQL 数据库和 IndexedDB 的信息：[`hacks.mozilla.org/2010/06/comparing-indexeddb-and-webdatabase/`](http://hacks.mozilla.org/2010/06/comparing-indexeddb-and-webdatabase/)。

## 离线应用程序

通常我们需要互联网连接来浏览网页。有时我们可以浏览缓存的离线网页。这些缓存的离线网页通常会很快过期。通过 HTML5 引入的下一个离线应用程序，我们可以声明我们的缓存清单。这是一个文件列表，将在没有互联网连接时存储以供以后访问。

通过缓存清单，我们可以将所有游戏图形、游戏控制 JavaScript 文件、CSS 样式表和 HTML 文件本地存储。我们可以将我们的 HTML5 游戏打包成桌面或移动设备上的离线游戏。玩家甚至可以在飞行模式下玩游戏。

来自 Pie Guy 游戏（[`mrgan.com/pieguy`](http://mrgan.com/pieguy)）的以下屏幕截图显示了 iPhone 上的 HTML5 游戏，没有互联网连接。请注意离线状态的小飞机符号：

![离线应用](img/1260_01_02.jpg)

# 发现 CSS3 中的新功能

CSS 是演示层，HTML 是内容层。它定义了 HTML 的外观。在使用 HTML5 创建游戏时，尤其是基于 DOM 的游戏，我们不能错过 CSS。我们可能纯粹使用 JavaScript 来创建和设计带有 Canvas 元素的游戏。但是在创建基于 DOM 的 HTML5 游戏时，我们需要 CSS。因此，让我们看看 CSS3 中有什么新内容，以及如何使用新属性来创建游戏。

新的 CSS3 属性让我们可以以不同的方式在 DOM 中进行动画，而不是直接在 Canvas 绘图板上绘制和交互。这使得可以制作更复杂的基于 DOM 的浏览器游戏。

## CSS3 过渡

传统上，当我们对元素应用新样式时，样式会立即更改。CSS3 过渡在目标元素的样式更改期间应用插值。

例如，我们这里有一个蓝色的框，当我们鼠标悬停时想要将其变为红色。我们将使用以下代码片段：

HTML：

```js
<a href="#" class="box"></a>

```

CSS：

```js
a.box {
display:block;
width: 100px;
height: 100px;
background: #00f; /* blue */
border: 1px solid #000;
}
a.box:hover {
background: #f00;
}

```

当我们鼠标悬停时，框立即变为红色。应用了 CSS3 过渡后，我们可以使用特定持续时间和缓动值来插值样式：

```js
a.box {
-webkit-transition: all 5s linear;
}

```

### 提示

**下载本书的示例代码**

您可以从您在[`www.PacktPub.com`](http://www.PacktPub.com)的帐户中下载您购买的所有 Packt 图书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.PacktPub.com/support`](http://www.PacktPub.com/support)并注册，以便直接将文件发送到您的电子邮件。

以下屏幕截图显示了应用过渡的框悬停效果：

![CSS3 过渡](img/1260_01_03.jpg)

### 注意

由于 CSS3 规范仍处于草案阶段，尚未确定，因此来自不同浏览器供应商的实现可能与 W3C 规范有一些细微差异。因此，浏览器供应商倾向于使用供应商前缀来实现其 CSS3 属性，以防止冲突。

Safari 和 Chrome 使用`-webkit-`前缀。Opera 使用`-o-`前缀。Firefox 使用`-moz-`前缀，IE 使用`-ms-`前缀。现在声明 CSS3 属性，例如 box-shadow，可能有点复杂，需要为几个浏览器编写几行相同的规则。我们可以期望在该属性规范确定后，前缀将被消除。

我将在大多数示例中只使用`-webkit-`前缀，以防止在书中放置太多相似的行。更重要的是理解概念，而不是在这里阅读带有不同供应商前缀的相同规则。

## CSS3 变换

CSS3 变换让我们可以缩放元素，旋转元素和平移它们的位置。CSS3 变换分为 2D 和 3D。

我们可以用 translate 重新定位一个元素：

```js
-webkit-transform: translate(x,y);

```

或者使用缩放变换来缩放元素：

```js
-webkit-transform: scale(1.1);

```

我们还可以使用 CSS3 变换来缩放和旋转元素，并结合其他变换：

```js
a.box {
-webkit-transition: all 0.5s linear;
-webkit-transform: translate(100px,50px);
}
a.box:hover {
-webkit-transform: translate(100px,50px) scale(1.1) rotate(30deg);
}

```

以下屏幕截图显示了当我们鼠标悬停时 CSS3 变换效果：

![CSS3 变换](img/1260_01_04.jpg)

CSS3 变换 3D 进一步将空间扩展到三个轴，目前仅在 Safari 和移动 Safari 上有效。来自[WebKit.org](http://WebKit.org)的以下屏幕截图显示了当我们鼠标悬停时 3D 卡片翻转效果：

![CSS3 变换](img/1260_01_05.jpg)

## CSS3 动画

CSS3 过渡是一种动画类型。它声明了元素的两种样式之间的插值动画。

CSS3 动画是更进一步的一步。我们可以定义动画的关键帧。每个关键帧包含应在该时刻更改的一组属性。这就像一组应用于目标元素的 CSS3 过渡的序列。

AT-AT Walker ([`anthonycalzadilla.com/css3-ATAT/index-bones.html`](http://anthonycalzadilla.com/css3-ATAT/index-bones.html)) 展示了使用 CSS3 动画关键帧、变换和过渡创建骨骼动画的演示：

![CSS3 动画](img/1260_01_06.jpg)

# 学习更多 HTML5 和 CSS3 新功能的细节

来自 Google 的 HTML5Rocks（[`html5rocks.com`](http://html5rocks.com)）提供了一个关于新 HTML5 元素和 CSS3 属性的坚实的快速入门指南。

苹果还展示了在其基于 WebKit 的浏览器中使用 HTML5 可以有多么吸引人（[`apple.com/html5`](http://apple.com/html5)）。

CSS3 Info（[`www.css3.info`](http://www.css3.info)）是一个提供最新 CSS3 新闻的博客。这是一个获取最新 CSS3 规范状态、兼容列表和基本 CSS3 代码的好地方。

# 创建 HTML5 游戏的好处

我们探索了 HTML5 和 CSS3 的一些关键新功能。有了这些功能，我们可以在浏览器上创建 HTML5 游戏。但是为什么我们需要这样做呢？创建 HTML5 游戏有什么好处呢？

## 不需要第三方插件

在现代浏览器中原生支持所有这些功能，我们不需要用户预先安装任何第三方插件才能进行游戏。这些插件不是标准的。它们是专有的，通常需要额外的插件安装，我们可能无法安装。

## 支持 iOS 设备而无需插件

全球数百万的苹果 iOS 设备不支持 Flash Player 等第三方插件。无论苹果出于什么原因不允许 Flash Player 在他们的移动 Safari 上运行，HTML5 和相关的 Web 标准是他们在浏览器中得到的。我们可以通过创建为移动设备优化的 HTML5 游戏来触及这一用户群体。

## 打破传统浏览器游戏的界限

在传统的游戏设计中，我们在一个边界框内构建游戏。我们在电视上玩视频游戏。我们在网页浏览器中玩 Flash 游戏，有一个矩形边界。

有了创意，我们不再受限于矩形游戏舞台。我们可以玩弄所有页面元素，甚至可以使用许多浏览器窗口来组成一个游戏。此外，我们甚至可以只使用 URL 栏来创建一个游戏（[`probablyinteractive.com/url-hunter`](http://probablyinteractive.com/url-hunter)）。这可能听起来有些混乱，但这是因为还没有多少网页这样做。

Photojojo（[`photojojo.com/store/awesomeness/cell-phone-lenses`](http://photojojo.com/store/awesomeness/cell-phone-lenses)）是一个在线摄影商店，在其商店页面上提供了一个有趣的彩蛋功能。页面上有一个带有标题“不要拉”的开关按钮。当用户点击它时，一个橙色的手臂从顶部出现，帧逐帧地进行动画处理。它像一块布一样抓住网页并将整个页面向上拉，创建一个有趣的向下滚动效果。这不是一个游戏，但足够有趣，可以展示我们如何打破界限。

![打破传统浏览器游戏的界限](img/1260_01_07.jpg)

这里有另一个例子，名为 Twitch（[`reas.com/twitch/`](http://reas.com/twitch/)），来自 Chrome 实验。这是一个迷你游戏集合，玩家必须将球从起点运送到终点。有趣的是，每个迷你游戏都是一个小型浏览器窗口。当球到达该迷你游戏的目的地时，它会被转移到新创建的迷你游戏浏览器中继续旅程。以下截图显示了 Twitch 的整个地图以及各个网页浏览器：

![打破传统浏览器游戏的界限](img/1260_01_08.jpg)

## 构建 HTML5 游戏

由于 HTML5 和 CSS3 的新功能，我们现在可以在浏览器中创建整个游戏。我们可以控制 DOM 中的每个元素。我们可以使用 CSS3 对每个文档对象进行动画处理。我们有 Canvas 来动态绘制和与之交互。我们有音频元素来处理背景音乐和音效。我们还有本地存储来保存游戏数据和 WebSocket 来创建实时多人游戏。大多数现代浏览器已经支持这些功能。现在是时候制作 HTML5 游戏了。

# 其他人正在玩的 HTML5 游戏

通过观察使用不同技术制作的其他 HTML5 游戏，我们有机会研究不同 HTML5 游戏的表现。

## 匹配游戏

匹配游戏 ([`10k.aneventapart.com/Uploads/300/`](http://10k.aneventapart.com/Uploads/300/)) 展示了一个美丽的匹配游戏，使用了 CSS3 动画和其他视觉增强效果。当您按下 3D 样式的 CSS 按钮时，游戏开始。卡片在背面和正面使用 3D 旋转翻转。正面的图案是从在线画廊动态获取的。

![匹配游戏](img/1260_01_09.jpg)

## Sinuous

Sinuous ([`10k.aneventapart.com/Uploads/83/`](http://10k.aneventapart.com/Uploads/83/))，10K Apart 的获胜者，向我们展示了一个简单的游戏想法如何通过适当的实现让人上瘾。玩家用鼠标控制空间中的大点。目标是移动点以避开飞来的彗星。听起来很简单，但绝对让人上瘾，是一个“再试一次”的游戏。这个游戏是用 Canvas 标签创建的。玩家还可以在他们的支持 webkit 的移动设备上玩这个游戏，比如 iPhone、iPad 和 Android。

![Sinuous](img/1260_01_10.jpg)

## 类似于小行星的书签

来自瑞典的网页设计师 Erik 创建了一个有趣的书签。它是一个适用于任何网页的类似小行星的游戏。是的，任何网页。它展示了与任何网页进行交互的一种异常方式。它在您正在阅读的网站上创建了一个飞机。然后您可以使用箭头键驾驶飞机，并使用空格键发射子弹。有趣的是，子弹会摧毁页面上的 HTML 元素。您的目标是摧毁您选择的网页上的所有东西。这个书签是打破通常浏览器游戏界限的又一个例子。它告诉我们，在设计 HTML5 游戏时，我们可以打破常规思维。

该书签可以在以下网址安装：[`erkie.github.com/`](http://erkie.github.com/)。

以下的截图显示了飞机摧毁网页内容的情况：

![类似于小行星的书签](img/1260_01_11.jpg)

## Quake 2

谷歌演示了第一人称射击游戏 Quake 2 的 WebGL HTML5 移植版。玩家可以使用 WSAD 键四处移动，并用鼠标射击敌人。玩家甚至可以通过 WebSocket 实时进行多人游戏。据谷歌称，HTML5 Quake 2 的每秒帧数可以达到 60 帧。

![Quake 2](img/1260_01_12.jpg)

Quake 2 移植版可以在 Google Code 上找到：[`code.google.com/p/quake2-gwt-port/`](http://code.google.com/p/quake2-gwt-port/)。

## RumpeTroll

RumpeTroll ([`rumpetroll.com/`](http://rumpetroll.com/)) 是 HTML5 社区的一个实验，每个人都可以通过 WebSocket 连接在一起。我们可以给我们的生物取名字，并通过鼠标点击四处移动。我们还可以输入任何内容开始聊天。此外，由于 WebSocket 插入，我们可以实时看到其他人在做什么。

![RumpeTroll](img/1260_01_13.jpg)

## Scrabb.ly

Scrabb.ly ([`scrabb.ly`](http://scrabb.ly)) 是一个多人游戏，赢得了 Node.js Knockout 比赛的人气奖。它使用 HTML5 WebSocket 将用户连接在一起。这个在线棋盘游戏是基于 DOM 的，由 JavaScript 驱动。

![Scrabb.ly](img/1260_01_14.jpg)

### 注意

Node.js (http://nodejs.org) 是一个事件驱动的服务器端 JavaScript。它可以用作连接并发 WebSocket 客户端的服务器。

## Aves Engine

Aves Engine 是由 dextrose 开发的 HTML5 游戏开发框架。它为游戏开发者提供了工具和 API，用于构建自己的等距浏览器游戏世界和地图编辑器。从官方演示视频中捕获的以下截图显示了它是如何创建等距世界的：

![Aves Engine](img/1260_01_15.jpg)

该引擎还负责 2.5 维等距坐标系统、碰撞检测和其他基本的虚拟世界功能。这个游戏引擎甚至在 iPad 和 iPhone 等移动设备上运行良好。Aves Engine 自首次亮相以来就引起了很多关注，现在已被大型社交游戏公司 Zynga Game Network Inc 收购。

Aves Engine 的视频演示可在 YouTube 上通过以下链接观看：

[`tinyurl.com/dextrose-aves-engine-sneak`](http://tinyurl.com/dextrose-aves-engine-sneak)

# 浏览更多 HTML5 游戏

这些例子只是其中的一部分。以下网站提供了由他人创建的 HTML5 游戏的更新：

+   Canvas Demo ([`canvasdemo.com`](http://canvasdemo.com)) 收集了一系列使用 HTML5 Canvas 标签的应用程序和游戏。它还提供了大量 Canvas 教程资源。这是学习 Canvas 的好地方。

+   HTML5 游戏 ([`html5games.com`](http://html5games.com)) 收集了许多 HTML5 游戏，并将它们组织成不同的类别。

+   Mozilla Labs 在 2011 年初举办了一个 HTML5 游戏设计比赛，许多优秀的游戏被提交到比赛中。比赛现在已经结束，所有参赛作品的列表在以下链接：[`gaming.mozillalabs.com/games/`](http://https://gaming.mozillalabs.com/games/)。

+   HTML5 Game Jam ([`www.html5gamejam.com/games`](http://www.html5gamejam.com/games)) 是一个 HTML5 活动，该网站列出了一系列有趣的 HTML5 游戏，还提供了一些有用的资源。

# 我们将在本书中创建的内容

在接下来的章节中，我们将构建六款游戏。我们将首先创建一个基于 DOM 的乒乓球游戏，可以由同一台机器上的两名玩家进行游戏。然后我们将创建一个带有 CSS3 动画的记忆匹配游戏。之后，我们将使用 Canvas 创建一个解开谜题的游戏。接下来，我们将使用音频元素创建一个音乐游戏。然后，我们将使用 WebSocket 创建一个多人绘画和猜谜游戏。最后，我们将使用 Box2D JavaScript 端口创建一个物理汽车游戏的原型。以下截图是我们将在第三章中构建的记忆匹配游戏的截图，*在 CSS3 中构建记忆匹配游戏*

![我们将在本书中创建的内容](img/1260_01_16.jpg)

# 摘要

在本章中，我们学到了关于 HTML5 游戏的基本信息。

具体来说，我们涵盖了：

+   来自 HTML5 和 CSS3 的新功能。我们已经初步了解了我们将在后续章节中使用的技术。Canvas、音频、CSS 动画等更多新功能被介绍了。我们将有许多新功能可以使用。

+   创建 HTML5 游戏的好处。我们讨论了为什么要创建 HTML5 游戏。我们想要满足网络标准，满足移动设备，并打破游戏的边界。

+   其他人正在玩的 HTML5 游戏。我们列出了使用我们将使用的不同技术创建的几款现有 HTML5 游戏。在创建我们自己的游戏之前，我们可以测试这些游戏。

+   我们还预览了本书中将要构建的游戏。

现在我们已经了解了一些关于 HTML5 游戏的背景信息，我们准备在下一章中创建我们的第一个基于 DOM 的 JavaScript 驱动游戏。
