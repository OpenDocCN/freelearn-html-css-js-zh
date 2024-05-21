`# 使用 WebRTC 进行实时视频通话应用

嘿！只是想告诉你，JS Meetup 在找到后端开发人员完成应用程序的服务器端之后取得了巨大成功。但是你很棒，完成了整个应用程序的前端。你创建了一个完整的活动注册网站，让用户报名参加活动，同时学习了一些非常重要的概念，比如构建可重用的 ES6 模块，使用 Promises 处理异步代码进行 AJAX 请求，从数据创建美丽的图表，当然还有经典的表单验证与验证服务。

后端代码也是用 JavaScript（Node.js）编写的，所以你可能真的对编写服务器端代码感兴趣。但遗憾的是，正如我之前提到的，Node.js 超出了本书的范围。实际上，你可以用纯 JavaScript 做一些非常酷的事情，尽管很多人认为，“*它需要大量的服务器端代码！*”因为你已经读过本章的标题 - 是的！我们将在本章中构建一个真正的视频通话应用程序，几乎没有服务器端代码。最好的部分是，就像我们的其他应用程序一样，这个应用程序也将是响应式的，并且将与大多数移动浏览器兼容。

让我们首先看一下我们将在本章学习的概念清单：

+   WebRTC 介绍

+   JavaScript 中的 WebRTC API

+   使用 SimpleWebRTC 框架进行工作

+   构建视频通话应用程序

除了这些主要概念，本章还有很多东西要学习。因此，在我们开始之前，请确保你有以下硬件：

+   带有网络摄像头和麦克风的台式机或笔记本电脑（你可能想使用另一台计算机来体验视频通话的实际效果）

+   安卓或 iPhone 设备（可选）

+   局域网连接，以便所有设备都在同一局域网上进行开发应用程序的测试（可以是 Wi-Fi 或有线以太网）

这个项目中使用的一个依赖项要求你的系统中安装了 Python 2.7.x。Linux 和 Mac 用户已经预装了 Python。Windows 用户可以从[`www.python.org/downloads/`](https://www.python.org/downloads/)下载 Python 2.7.x 版本。

# 第四章：WebRTC 介绍

在我们开始构建应用程序之前，最好先了解一些关于 WebRTC 的知识，以便你对应用程序的工作原理有一个很好的了解。

# WebRTC 的历史

实时通信能力已经成为我们现在使用的许多应用程序的常见功能。比如你想和朋友聊天或者观看现场足球比赛。这些应用程序必须具备实时通信功能。然而，在过去在浏览器上进行实时视频通话对用户来说是一项相当困难的任务，因为他们必须为不同的应用程序在 Web 浏览器上使用视频通话安装插件，而插件会带来漏洞，因此需要定期更新。

为了解决这个问题，谷歌于 2011 年 5 月发布了一个开源项目，用于基于浏览器的实时通信标准，名为 WebRTC。WebRTC 的概念很简单。它定义了一套标准，应该在所有应用程序中使用，以便应用程序可以直接相互通信（点对点通信）。通过实现 WebRTC，将不再需要插件，因为通信平台是标准化的。

目前，WebRTC 正在由**万维网联盟**（**W3C**）和**互联网工程任务组**（**IETF**）进行标准化。WebRTC 正在被大多数浏览器供应商积极实施，并且它也将与原生的 Android 和 iOS 应用程序一起工作。如果你想知道你的浏览器是否准备支持 WebRTC，你可以访问：[`iswebrtcreadyyet.com/`](http://iswebrtcreadyyet.com/)。

在撰写本书时，浏览器支持状态如下：

![](img/00022.jpeg)

尽管大多数常用的浏览器都支持 WebRTC，除了 Safari，但实现中仍然存在许多问题和错误，因此建议使用适配器（如`adapter.js`）（[`github.com/webrtc/adapter`](https://github.com/webrtc/adapter)），以便应用程序在规范或供应商前缀发生变化时不会遇到任何问题。当我们研究 WebRTC 的 JavaScript API 时，我们将更多地了解这一点。

WebRTC 也支持 Chrome 和 Firefox 的移动版本；因此，即使在没有插件的移动浏览器中，你也可以进行视频通话。

对于 iPhone 用户，iPhone 上的 Safari 移动浏览器或 Chrome 尚不支持 WebRTC。因此，你必须安装 Firefox 或来自应用商店的 Bowser 应用。Bowser 的链接：[`itunes.apple.com/app/bowser/id560478358?mt=8`](https://itunes.apple.com/app/bowser/id560478358?mt=8)。

# JavaScript WebAPIs

到目前为止，我们已经使用了一些 WebAPIs，比如`FileReader`，文档（在`document.querySelector()`方法中使用），`HTMLImageElement`（我们在 Meme Creator 中使用的`new Image()`构造函数），等等。它们不是 JavaScript 语言的一部分，但它们是 WebAPIs 的一部分。在浏览器中运行 JavaScript 时，将提供一个包含所有 WebAPIs 方法的`window`对象。`window`对象的范围是全局的，`window`对象的属性和方法也是全局的。这意味着，如果你想使用 navigator WebAPI，你可以这样做：

```js
window.navigator.getUserMedia()
```

或者，你可以简单地这样做：

```js
navigator.getUserMedia();
```

两者都可以正常工作并实现相同的方法。但是请注意，WebAPI（`window`对象）仅在浏览器中运行 JavaScript 时才可用。如果你在其他平台上使用 JavaScript，比如 Node.js 或 React Native，你将无法使用 WebAPIs。

现在 WebAPIs 变得越来越强大，为 JavaScript 提供了更多的功能，比如直接从浏览器录制视频和音频。渐进式 Web 应用程序就是这样的一个例子，由`ServiceWorker` WebAPI 提供支持。

本章和接下来的章节中，我们将使用大量的 WebAPIs。有关 JavaScript 可用的 WebAPIs 的完整列表，请访问以下 MDN 页面：[`developer.mozilla.org/en-US/docs/Web/API`](https://developer.mozilla.org/en-US/docs/Web/API)。

# JavaScript WebRTC API

由于浏览器原生支持 WebRTC，因此浏览器供应商创建了 JavaScript WebAPIs，以便开发人员可以轻松构建应用程序。目前，WebRTC 实现了以下三个 JavaScript 使用的 API：

+   MediaStream

+   RTCPeerConnection

+   RTCDataChannel

# MediaStream

MediaStream API 用于获取用户的视频和音频设备的访问权限。通常，浏览器会提示用户是否允许网站访问他/她设备的摄像头和麦克风。尽管 MediaStream API 的基本概念是相同的，但不同的浏览器供应商对 API 的实现方式有所不同。

在使用`getUserMedia()`方法时，使用`{audio: true}`来访问你自己的麦克风时，*要么将扬声器静音，要么将 HTML 视频元素静音*。否则，*可能会导致反馈，损坏你的扬声器*。

例如，在 Chrome 中，要使用 MediaStream API，你需要使用`navigator.getUserMedia()`方法。此外，Chrome 只允许 MediaStream 在 localhost 或 HTTPS URL 中工作。

`navigator.getUserMedia()`接受三个参数。第一个是配置对象，告诉浏览器网站需要访问什么。另外两个是成功或失败响应的回调函数。

创建一个简单的 HTML 文件，比如`chrome.html`，放在一个空目录中。在 HTML 文件中，添加以下代码：

```js
<video></video>
<script>
const $video = document.querySelector('video');
if (navigator.getUserMedia) {
  navigator.getUserMedia(
    {audio: true, video: true},
    stream => {
      $video.srcObject = stream;
      $video.muted = true; // Video muted to avoid feedback
      $video.onloadedmetadata = () => {
        $video.play();
      };
    },
    error => console.error(error)
  );
}
</script>
```

这段代码做了以下几件事：

+   它将在`$video`对象中创建对`<video>`元素的引用。

+   然后，它检查`navigator.getUserMedia`是否可用。这样做是为了避免在使用不兼容 WebRTC 的浏览器时出现错误。

+   然后，它使用以下三个参数调用`navigator.getUserMedia()`方法：

+   第一个参数指定网站对浏览器的需求。在我们的例子中，需要音频和视频。因此，我们应该传递`{audio: true, video: true}`。

+   第二个参数是成功的回调函数。用户接收的视频和音频流在传递给此函数的`stream`对象中可用。它将`srcObject`属性添加到`<video>`元素，其值为从用户输入设备接收的视频和音频的`stream`对象。当流加载时，将调用`$video.onloadedmetadata`，并且它将开始播放视频，因为我们在其回调函数中添加了`$video.play()`。

+   第三个参数是当用户拒绝网站访问摄像头或麦克风，或者发生其他错误且无法检索媒体流时调用的函数。此函数的参数是一个`error`对象，其中包含错误详细信息。

现在，使用`http-server`在本地主机中的 Chrome 中打开文件。首先，Chrome 将提示您允许访问设备的摄像头和麦克风。它应该如下所示：

![](img/00023.jpeg)

如果您点击允许，您应该看到通过前置摄像头传输的视频。我已经在以下网址设置了一个 JS fiddle：[`jsfiddle.net/1odpck45/`](https://jsfiddle.net/1odpck45/)，您可以在其中玩弄视频流。

一旦您点击允许或阻止，Chrome 将记住网站的偏好设置。要更改网站的权限，您必须点击地址栏左侧的锁定或信息图标，它将显示一个菜单，如下所示，您可以再次更改权限：

![](img/00024.jpeg)由于我们使用 http-server 或 Webpack 开发服务器进行开发，这些服务器在本地主机上运行，因此我们可以在 Chrome 中开发 WebRTC 应用程序。但是，如果要在生产环境中部署应用程序，则需要使用 HTTPS URL 进行部署。否则，应用程序将无法在 Chrome 上运行。

我们在 Chrome 上创建的视频在 Chrome 上运行得很好，但是如果您尝试在不同的浏览器 Firefox 上运行此代码，它将无法运行。这是因为 Firefox 对 MediaStream API 有不同的实现。

在 Firefox 中，您需要使用`navigator.mediaDevices.getUserMedia()`方法，该方法返回一个 Promise。可以使用`.then().catch()`链使用`stream`对象。

Firefox 的代码如下：

```js
<video></video>
<script>
const $video = document.querySelector('video');
navigator.mediaDevices.getUserMedia({audio: true, video: true})
.then(stream => {
  $video.srcObject = stream;
  $video.muted = true;
  $video.onloadedmetadata = function(e) {
    $video.play();
  };
})
.catch(console.error);
</script>
```

您可以在 Firefox 中运行此代码，方法是在与您创建`chrome.html`文件相同的目录中创建一个`firefox.html`文件，或者在您的 Firefox 浏览器中打开以下 JS fiddle：[`jsfiddle.net/hc39mL5g/`](https://jsfiddle.net/hc39mL5g/)。

为了生产环境设置 HTTPS 服务器超出了本书的范围。但是，根据您想要使用的服务器类型，可以很容易地在互联网上找到说明。

# 使用 Adapter.js 库

由于 WebRTC 在不同浏览器之间的实现不同，建议使用适配器（例如`adapter.js`库([`github.com/webrtc/adapter`](https://github.com/webrtc/adapter)））来隔离代码与浏览器实现的差异。通过包含`adapter.js`库，您可以在 Firefox 浏览器中运行为 Chrome 编写的 WebRTC 代码。尝试在 Firefox 中运行以下 JS fiddle，其中包含适用于 Chrome 的 WebRTC 代码，但包括`adapter.js`：[`jsfiddle.net/1ydwr4tt/`](https://jsfiddle.net/1ydwr4tt/)。

如果您想了解`<video>`元素，它是在 HTML5 中引入的。要了解有关使用视频元素的更多信息，请访问 w3schools 页面：[`www.w3schools.com/html/html5_video.asp`](https://www.w3schools.com/html/html5_video.asp)或 MDN 页面：[`developer.mozilla.org/en/docs/Web/HTML/Element/video`](https://developer.mozilla.org/en/docs/Web/HTML/Element/video)。

# RTCPeerConnection 和 RTCDataChannel

虽然 MediaStream API 用于从用户设备检索视频和音频流，但 RTCPeerConnection 和 RTCDataChannel API 用于建立对等连接并在它们之间传输数据。在我们的视频通话应用程序中，我们将使用 SimpleWebRTC 框架，它将抽象这些 API 并为我们提供一个更简单的对象来与其他设备建立连接。因此，我们不打算深入研究这两个 API。

然而，在使用 WebRTC 时有一件重要的事情要知道。尽管 WebRTC 是为了使设备直接连接而无需任何服务器而创建的，但目前不可能实现这一点，因为要连接到设备，您需要知道设备在互联网上的位置，即设备在互联网上的 IP 地址。但是，一般来说，设备只会知道它们的本地 IP 地址（类似于 192.168.1.x）。公共 IP 地址由防火墙或路由器管理。为了克服这个问题并将确切的 IP 地址发送给其他设备，我们需要信令服务器，例如**STUN**或**TURN**。

设备将向 STUN 服务器发送请求，以检索其公共 IP 地址，并将该信息发送给其他设备。这是广泛使用的，并适用于大多数情况。但是，如果路由器或防火墙的 NAT 服务为设备的每个连接分配不同的端口号，或者设备的本地地址不断变化，那么从 STUN 服务器接收的数据可能不足，因此必须使用 TURN 服务器。TURN 服务器充当两个设备之间的中继，即设备将数据发送到 TURN 服务器，然后 TURN 服务器将数据中继到其他设备。但是，TURN 服务器不像 STUN 服务器那样高效，因为它消耗了大量服务器端资源。

通常会使用**ICE**实现，它确定两台设备之间是否需要 STUN 或 TURN 服务器（在大多数情况下会选择 STUN，而使用 TURN 作为最后的手段），从而保持连接更有效和稳定。

使用 WebRTC 进行实时通信是一个很大的主题，但如果您有兴趣了解更多关于 WebRTC 的信息，可以访问 WebRTC 的官方网站[`webrtc.org/`](https://webrtc.org/)，查看一些可用于开始使用 WebRTC 的各种资源。

# 构建视频通话应用程序

我们将在本章中构建的应用程序是一个简单的视频会议应用程序，您可以在其中创建一个房间，然后将房间 URL 分享给其他人。谁点击 URL 将能够加入通话。对于 UI 部分，我们可以将参与者的视频排列在小框中，当您点击参与者时，我们可以放大视频。这种类型的视频通话应用程序现在广泛使用。以下是应用程序在桌面浏览器上的外观：

![](img/00025.jpeg)

蓝色框将显示您的视频，而其他框应显示其他参与者的视频。当参与者数量增加时，行将自动换行到新行（flex-wrap）。在移动设备上，我们可以将视频显示为列而不是行，因为对于较小的屏幕来说，这样会更有效。因此，对于手机，应用程序应如下所示：

![](img/00026.jpeg)

这些框只是占位符。对于真实的视频，我们可以使用 margin/padding 在每个视频之间留出间距。此外，为了分享链接，我们可以使用一个点击复制按钮，这将非常用户友好。现在你已经很好地理解了我们要构建的内容，让我们开始吧！

# 初始项目设置

初始设置与我们在之前的活动注册应用程序中所做的并没有太大的不同。在 VSCode 的`Chapter04`文件夹中打开起始文件并创建一个`.env`文件。从`.env.example`文件中，你应该知道，对于这个应用程序，我们只需要一个环境变量`NODE_ENV`，其值只在生产环境下为`production`。对于开发，我们可以简单地为其分配其他值，比如`dev`。

创建了`.env`文件后，在 VSCode 的终端或本机终端（导航到项目根文件夹）中运行`npm install`来安装项目的所有依赖项。之后，在终端中运行`npm run webpack`，这应该会启动 Webpack 开发服务器。

# 为页面添加样式

你知道如何使用 Webpack 开发服务器。所以，让我们继续添加样式到我们的页面。首先，浏览`index.html`文件，了解页面的基本结构。

页面的主体分为两个部分：

+   导航栏

+   容器

容器进一步分为三个部分：

1.  首先是`create-room-area`，其中包含创建具有房间名称的新房间所需的输入字段。

1.  其次是`info-area`，其中包含有关房间的信息（房间名称和房间 URL）。它还有两个按钮，用于复制房间 URL（当前使用`.hidden` Bootstrap 样式类进行隐藏）。

1.  最后是`video-area`，用于显示所有参与者的视频。

首先，在`src/css/styles.css`文件中添加以下代码，以防止容器部分与导航栏重叠：

```js
body {
  padding-top: 65px;
}
```

启用 Webpack 热重载后，你应该立即看到 CSS 的更改。`create-room-area`使用默认的 Bootstrap 样式看起来很好。所以，让我们继续进行第二部分，info-area。要处理`info-area`，暂时从 HTML 中删除`.hidden`类。还要从两个按钮中删除`.hidden`，并在段落元素中添加一些文本，其中包含房间 URL。如果房间 URL 和按钮在同一行对齐会很好。为了对齐它们，在`styles.css`文件中添加以下 CSS：

```js
.room-text {
  display: flex;
  flex-direction: row;
  padding: 10px;
  justify-content: flex-start;
  align-items: center;
  align-content: center;
}
.room-url {
  padding: 10px;
}
.copy {
  margin-left: 10px;
}
.copied {
  margin-left: 10px;
}
```

对于`video-area`，视频需要在移动设备上以列的形式排列，而在桌面上应以行的形式排列。因此，我们可以使用媒体查询为其分配不同的样式。此外，对于视频元素（`.video-player`）的大小，我们可以将`max-width`和`max-height`设置为 25 视口宽度，以使其在所有设备上具有响应性的尺寸。在你的`styles.css`文件中，添加以下样式：

```js
@media only screen and (max-width: 736px) {
  .video-area {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
  }
}
@media only screen and (min-width: 736px) {
  .video-area {
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
  }
}
.video-player {
  max-height: 25vh;
  max-width: 25vh;
  margin: 20px;
}
```

现在所需的样式就是这些。所以，让我们开始编写应用程序的 JavaScript。

# 构建视频通话应用程序

一切就绪，让我们开始编码。像以前的应用程序一样，打开你的`home.js`文件并创建你的`Home`类和构造函数：

```js
class Home {
  constructor() {

  }
}
```

之后，创建`Home`类的一个实例，并将其分配给一个对象`home`，如下所示：

```js
const home = new Home();
```

我们以后会用到 home 对象。现在，通过在项目根文件夹的终端中运行以下命令，将`SimpleWebRTC`包添加到我们的项目中：

```js
npm install -S simplewebrtc
```

并在你的`home.js`文件顶部添加以下导入语句：

```js
import SimpleWebRTC from 'simplewebrtc';
```

根据`SimpleWebRTC`文档，我们需要创建一个`SimpleWebRTC`类的实例，并进行一些配置以在我们的应用程序中使用它。在你的`home.js`文件中，在`Home`类之前，添加以下代码：

```js
const webrtc = new SimpleWebRTC({
  localVideoEl: 'localVideo',
  remoteVideosEl: '',
  autoRequestMedia: true,
  debug: false,
});
```

您的应用程序现在应该请求权限来访问摄像头和麦克风。这是因为在幕后，`SimpleWebRTC`已经开始设置一切需要启动视频通话的工作。如果您点击“允许”，您应该会看到您的视频出现在一个小矩形框中。这就是您在之前的代码中添加的对象中的配置所做的事情：

+   `localVideoEl`：包含应该包含您本地视频的元素的 ID。在这里，我们的`index.html`文件中的`video#localVideo`元素将显示我们自己的视频，因此选择它作为其值。

+   `remoteVideosEl`：包含需要添加远程视频的容器的 ID。我们还没有创建该元素，最好稍后再添加视频，所以将其留空。

+   `autoRequestMedia`：用于提示用户允许访问摄像头和麦克风的权限，需要设置为`true`。

+   `debug`：如果为 true，它将在控制台中打印所有的`webrtc`事件。我已将其设置为`false`，但在您的系统上将其设置为 true 以查看事件发生。

默认情况下，`SimpleWebRTC`使用由 Google 提供的免费 STUN 服务器，即`stun.l.google.com:19302`。在大多数情况下，这个 STUN 服务器就足够了，除非您身处一些具有复杂路由协议的企业防火墙之后。否则，您可以设置自己的 ICE 配置，包括 STUN 和 TURN 服务器。为此，您需要安装 signalmaster（[`github.com/andyet/signalmaster`](https://github.com/andyet/signalmaster)），并将 ICE 配置详细信息添加到前面提到的构造函数中。然而，这超出了本书的范围。我们将简单地继续使用默认配置。

对于我们的第一步，我们将在构造函数中创建类变量和对 DOM 元素的引用：

```js
constructor() {
  this.roomName = '';

  this.$createRoomSection = document.querySelector('#createRoomSection');
  this.$createRoomButton = document.querySelector('#createRoom');
  this.$roomNameInput = document.querySelector('#roomNameInput');

  this.$infoSection = document.querySelector('#infoSection');
  this.$roomName = document.querySelector('#roomNameText');
  this.$roomUrl = document.querySelector('#roomUrl');
  this.$buttonArea = document.querySelector('.room-text');
  this.$copy = document.querySelector('.copy');
  this.$copied = document.querySelector('.copied');

  this.$remotes = document.querySelector('.video-area');
  this.$localVideo = document.querySelector('#localVideo');
}
```

这很多，但它们都是我们应用程序不同步骤所需的。我们在这里创建的唯一变量是`roomName`，正如其名称所示，它包含了房间的名称。其他的都是对 DOM 元素的引用。

# 创建房间

该应用的第一步是创建一个房间，以便其他成员可以加入房间进行通话。根据当前的 UI 设计，当用户点击“创建房间”按钮时，我们需要创建房间。因此，让我们在该按钮上注册一个点击事件处理程序。

到目前为止，我们一直在使用不同的方法来处理事件：

+   在我们的待办事项列表应用程序中，我们在 HTML 中添加了`onclick`属性来调用 JavaScript 函数的`onclick`事件。

+   在 Meme Creator 中，我们为每个元素附加了事件侦听器，我们希望监听特定事件的发生（keyup、change 和 click 事件）。在事件注册表单中也是如此，我们添加了一个事件侦听器来监听表单提交操作。

+   还有另一种方法，即将回调函数添加到 DOM 元素的引用的事件属性中。在我们的情况下，我们需要检测“创建房间”按钮的点击事件。我们可以这样处理：

```js
this.$createRoomButton.onclick  = () => { }
```

因此，每当单击“创建房间”按钮时，它将执行前述函数中编写的代码。完全取决于您和您的要求来决定使用哪种事件处理程序。通常，第一种方法会被避免，因为它会将您的 JavaScript 代码暴露在 HTML 中，并且在大型项目中难以跟踪 HTML 中调用的所有 JavaScript 函数。

如果您有大量的元素，比如表格中的 100 行，为每一行附加 100 个事件侦听器是低效的。您可以使用第三种方法，通过将函数附加到每行 DOM 元素的引用的`onclick`方法，或者您可以将单个事件侦听器附加到行的父元素，并使用该事件侦听器来监听其子元素的事件。

有关所有 DOM 事件的列表，请访问 W3Schools 页面：[`www.w3schools.com/jsref/dom_obj_event.asp`](https://www.w3schools.com/jsref/dom_obj_event.asp)。

在我们的应用程序中，我们需要处理很多点击事件。因此，让我们在`Home`类中创建一个方法来注册所有的点击事件：

```js
registerClicks() {
}
```

并在构造函数中调用此方法：

```js
constructor() {
  ...
  this.registerClicks();
}
```

在`registerClicks()`方法中，添加以下代码：

```js
this.$createRoomButton.onclick  = () => { }
```

当用户单击“创建房间”按钮时，需要执行一些操作：

+   获取房间名称。但是房间名称不能包含任何会导致 URL 出现问题的特殊字符

+   使用`SimpleWebRTC`创建一个房间

+   将用户重定向到为房间创建的 URL（带有房间名称作为查询字符串的 URL）

+   显示他/她可以与其他需要参与通话的人分享的 URL

您应该在您在上述代码中创建的`onclick`方法中编写以下代码：

```js
this.roomName  =  this.$roomNameInput.value.toLowerCase().replace(/\s/g, '-').replace(/[^A-Za-z0-9_\-]/g, '');
```

这将获取在输入字段中键入的房间名称，并使用正则表达式将其转换为 URL 友好的字符。如果房间名称不为空，我们可以继续在`SimpleWebRTC`中创建房间：

```js
if(this.roomName) {  webrtc.createRoom(this.roomName, (err, name) => {
    if(!err) {
      // room created
    } else {
      // unable to create room
      console.error(err);
    }
  });
}
```

上述代码执行以下操作：

+   `if`条件将检查房间名称是否不为空（空字符串为假）。

+   `webrtc.createRoom()`将创建房间。它接受两个参数：第一个是房间名称字符串，第二个是在创建房间时执行的回调函数。

+   回调函数具有参数`err`和`name`。通常，我们应该检查过程是否成功。因此，`if(!err) {}`将包含在过程成功时执行的代码。`name`是由`SimpleWebRTC`创建的房间名称。

在`if(!err)`条件中，添加以下代码：

```js
const  newUrl  =  location.pathname  +  '?'  +  name; history.replaceState({}, '', newUrl);
this.roomName = name; this.roomCreated();
```

`location`对象包含有关当前 URL 的信息。`location.pathname`用于设置或获取网页的当前 URL。因此，我们可以通过将房间名称附加到其中来构造 URL。因此，如果您当前的 URL 是`http://localhost:8080/`，那么在创建房间后，您的 URL 应该变为`http://localhost:8080/?roomName`。

要替换 URL 而不影响当前页面，我们可以使用 History Web API 提供的`history`对象。`history`对象用于操作浏览器的历史记录。如果要执行用户单击浏览器后退按钮时发生的后退操作，可以按照以下步骤进行：

```js
history.back();
```

同样，要前进，可以按照以下步骤进行：

```js
history.forward();
```

但是我们在应用程序中需要做的是在不影响浏览器历史记录的情况下更改当前的 URL。也就是说，我们需要将 URL 从`http://localhost:8080/`更改为`http://localhost:8080/?roomName`，而不影响浏览器的后退或前进按钮。

对于这样复杂的操作，您可以使用 HTML5 中引入的`pushState()`和`replaceState()`方法来处理历史对象。`pushState()`在浏览器上创建一个新的历史记录条目，并更改页面的 URL，而不影响当前页面。`replaceState()`也是一样，但是它替换当前条目，非常适合我们的目的。

`pushState()`和`replaceState()`方法都接受三个参数。第一个是`state`（一个 JSON 对象），第二个是`title`（字符串），第三个是新的 URL。这就是`pushState()`和`replaceState()`的工作原理：

+   每次调用`pushState()`或`replaceState()`时，都会触发`window`对象中的`popstate`事件。第一个参数，状态对象，由该事件的回调函数使用。我们现在用不到它，所以将其设置为空对象。

+   目前，大多数浏览器都会忽略第二个参数，所以我们将其设置为空字符串。

+   第三个参数 URL 是我们真正需要的。它将浏览器的 URL 更改为提供的 URL 字符串。

由于房间已创建并且 URL 已更改，我们需要隐藏`.create-room-area` div 并显示`.info-area` div。这就是为什么我添加了`this.roomCreated()`方法。在`Home`类中，创建新方法：

```js
roomCreated() {
  this.$infoSection.classList.remove('hidden');
  this.$createRoomSection.classList.add('hidden');
  this.$roomName.textContent = `Room Name: ${this.roomName}`;
  this.$roomUrl.textContent = window.location.href;
}
```

这个方法将显示信息部分，同时隐藏创建房间部分。此外，它将使用`textContent()`方法更改房间名称和 URL，该方法更改了相应 DOM 元素中的文本。

有关位置对象的更多信息可以在 w3schools 页面上找到：[`www.w3schools.com/jsref/obj_location.asp`](https://www.w3schools.com/jsref/obj_location.asp)。有关历史对象的更多信息可以在 MDN 页面上找到：[`developer.mozilla.org/en-US/docs/Web/API/History`](https://developer.mozilla.org/en-US/docs/Web/API/History)。此外，如果你想学习如何操纵浏览器历史记录，可以访问[`developer.mozilla.org/en-US/docs/Web/API/History_API`](https://developer.mozilla.org/en-US/docs/Web/API/History_API)。

# 向你的房间添加参与者

你有一个活跃的房间和房间 URL，你需要邀请其他人。但是如果有一个点击复制功能来复制 URL，那不是更方便吗？这实际上是一个非常好的功能。因此，在我们向房间添加参与者之前，让我们构建一个点击复制功能。

# 点击复制文本

目前，信息区的外观是这样的：

![](img/00027.jpeg)

对于点击复制功能，如果你将鼠标悬停在房间 URL 上，它应该显示一个复制按钮：

![](img/00028.jpeg)

如果你点击复制按钮，它应该复制文本并变成已复制按钮：

![](img/00029.jpeg)

对于这个功能，我们需要添加一些事件监听器。因此，在你的 home 类中，创建一个新的方法`addEventListeners()`，并在构造函数中调用它：

```js
class Home {
  constructor() {
    ...
    this.addEventListeners();
  }

  addEventListeners() {
  }
}
```

包含复制按钮的 div 的引用存储在`this.$buttonArea`变量中。每当鼠标进入 div 时，它将触发一个`mouseenter`事件。当这个事件发生在`$buttonArea`中时，我们需要从复制按钮中移除`.hidden`类。

在你的`addEventListeners()`方法中，添加以下代码：

```js
this.$buttonArea.addEventListener('mouseenter', () => {
  this.$copy.classList.remove('hidden');
});
```

页面将重新加载，你将不得不再次创建一个房间。如果你现在将鼠标指针悬停在房间 URL 上，它应该会显示复制按钮。当指针离开`div`时，我们还需要隐藏按钮。类似于`mouseenter`，当指针离开`div`时，`div`将触发一个`mouseout`事件。因此，再次在前面的代码旁边添加以下代码：

```js
this.$buttonArea.addEventListener('mouseout', event => {
  this.$copy.classList.add('hidden');
  this.$copied.classList.add('hidden');
});
```

现在，再次尝试将鼠标指针悬停在房间 URL 上。令人惊讶的是，它并没有按预期工作。它应该有作用，但它没有。这是因为`mouseout`事件，当你的指针进入`$buttonArea`的子元素时，它也会触发。它将子元素视为`div`的`外部`。为了解决这个问题，我们需要过滤传递给回调函数的`event`对象，这样如果指针通过进入子元素而移动到`外部`，就不会发生任何操作。

这个有点棘手，但是如果你在控制台中打印事件对象，你会看到有很多属性和方法包含了事件的所有细节。`toElement`属性或`relatedTarget`属性将包含指针移动到的元素，具体取决于浏览器。因此，我们需要检查该元素的父元素是否是`$buttonArea`。如果是，我们应该阻止任何操作发生。为了做到这一点，将前面的代码更改为以下内容：

```js
this.$buttonArea.addEventListener('mouseout', event => {
  const e = event.toElement || event.relatedTarget;
  if(e) {
    if (e.parentNode == this.$buttonArea || e == this.$buttonArea) {
      return;
    }
  }
  this.$copy.classList.add('hidden');
  this.$copied.classList.add('hidden');
});
```

注意这一行：

```js
const e = event.toElement || event.relatedTarget;
```

这是一个短路评估。它的作用是，如果第一个值为真，它将把它赋给常量`e`。如果它为假，或运算符将评估第二个值，并将其值赋给`e`。你可以声明任意数量的值。比如：

```js
const fun = false || '' || true || 'test';
```

在这里，fun 的值将是列表中第一个真值语句，因此它的值将为 true。`'test'`也是一个真值，但它不会被评估，因为在它之前有一个真值。这种类型的赋值通常被使用，对于某些任务来说非常方便。

现在，`e`对象包含目标元素。所以，我们只需要检查`e`是否存在（以防止异常），如果存在，是否其父元素或元素本身是`$buttonArea`。如果是真的，我们只需返回。这样，回调函数在不隐藏复制和已复制按钮的情况下停止执行。我们也隐藏已复制按钮，因为当用户点击复制按钮时，我们将使其可见。

尝试在应用程序中再次悬停在房间 URL 上，应该按预期工作。最后一步是在用户点击复制按钮时复制 URL。因此，让我们在我们的`Home`类中早期创建的`registerClicks()`方法中注册点击。在`registerClicks()`方法中，添加处理点击复制和已复制按钮的代码，并在`Home`类中创建一个新方法`copyUrl()`来执行复制操作：

```js
registerClicks() {
  ...
  this.$copy.onclick = () => {
    this.copyUrl();
  };
  this.$copied.onclick = () => {
    this.copyUrl();
  };
}

copyUrl() {

}
```

在前面的代码中，在`registerClicks()`方法中，点击复制按钮和已复制按钮都将调用类的`copyUrl()`方法。我们需要在`copyUrl()`方法中添加复制文本的代码。

要复制文本，首先，我们需要从中复制文本的节点（DOM 元素）的范围。为此，创建一个范围对象并选择包含房间 URL 文本的`this.$roomUrl`节点。在`copyUrl()`方法中，添加以下代码：

```js
const range = document.createRange();
range.selectNode(this.$roomUrl);
```

现在，范围对象包含元素`$roomUrl`作为所选节点。然后，我们需要选择节点中的文本，就像用户通常使用光标选择文本一样。`window`对象有`getSelection()`方法，我们可以用于此目的。我们必须删除所有范围以清除先前的选择，然后选择一个新范围（即我们之前创建的范围对象）。在前面的代码中添加以下代码：

```js
window.getSelection().removeAllRanges();
window.getSelection().addRange(range);
```

最后，我们不知道用户的浏览器是否支持执行复制命令，所以我们在`try{} catch(err){}`语句中进行复制，以便如果发生任何错误，可以在 catch 语句中处理。`document.execCommand('copy')`方法将复制所选范围内的文本并将其作为字符串返回。此外，我们需要在复制成功时隐藏复制按钮并显示已复制按钮。复制的代码如下：

```js
try {
  const successful = document.execCommand('copy');
  const msg = successful ? 'successful' : 'unsuccessful';
  console.log('Copying text command was ' + msg);
  this.$copy.classList.add('hidden');
  this.$copied.classList.remove('hidden');
} catch(err) {
  console.error(err);
}
```

在添加了前面的代码之后，在应用程序中创建一个房间，然后尝试再次点击复制。它应该变成已复制按钮，并且房间 URL 文本将被突出显示，因为我们选择了文本，就像我们用 JavaScript 和光标选择文本一样。但是，一旦复制完成，清除选择会更好。因此，在`copyUrl()`方法的末尾添加这行：

```js
window.getSelection().removeAllRanges();
```

这将清除选择，所以下次点击复制时，房间 URL 文本将不会被突出显示。然后，您可以简单地粘贴所选的 URL 到任何您想要分享的地方。

# 加入房间

现在您有了一个链接，我们需要让用户使用该链接加入房间。这个过程很简单：当用户打开链接时，他会加入房间，并且所有参与者的视频都会显示给他。要让用户加入房间，`SimpleWebRTC`有`joinRoom('roomName')`方法，其中房间名称字符串作为参数传递。一旦用户在房间里，它将寻找房间中连接的其他用户的视频，并为它找到的每个视频触发`videoAdded`事件，以及一个回调函数，其中包含视频对象和该用户的对等对象。

让我们制定一下过程应该如何工作：

+   首先，我们需要检查用户输入的 URL 是否在其查询字符串中包含房间名称。也就是说，如果以`'?roomName'`结尾。

+   如果房间名称存在，那么我们应该让用户加入房间，同时隐藏`.create-room-area` div，并显示`.info-area` div 以显示房间详情。

+   然后，我们需要监听`videoAdded`事件，如果触发了事件，我们将视频添加到`.video-area div`中。

`SimpleWebRTC`在加载完成后会触发`readyToCall`事件。它还有`on()`方法来监听触发的事件。我们可以使用`readyToCall`事件来检查 URL 中的房间名称。这段代码应该在`Home`类之外。因此，在调用`Home`类构造函数的那一行之后，添加以下代码：

```js
webrtc.on('readyToCall', () => {
  if(location.search) {
    const locationArray = location.search.split('?');
    const room = locationArray[1];
  }
});
```

我们使用 location 对象来获取 URL。首先，我们需要检查 URL 是否包含查询字符串，使用`location.search`。因此，我们在 if 条件中使用它，如果它包含查询字符串，我们可以继续进行处理。

`split()`方法将字符串拆分为由传递给它的值分隔的子字符串数组。URL 将如下所示：

```js
http://localhost:8080/?myRoom
```

`location.search`将返回 URL 的查询字符串部分：

```js
'?myRoom'
```

因此，`location.search.split('?')`将把字符串转换为以下数组：

```js
[ '', 'myRoom']
```

我们在数组的索引 1 处有房间名称。像这样写是可以的，但是在这里我们可以使用短路评估。我们之前使用了 OR 运算符进行评估，它将获取第一个真值。在这种情况下，我们可以使用 AND 运算符，它将获取第一个假值，或者如果没有假值，则获取最后一个真值。上述代码将简化为以下形式：

```js
webrtc.on('readyToCall', () => {
  const room = location.search && location.search.split('?')[1];
});
```

如果 URL 不包含查询字符串，`location.search`将是一个空字符串（`""`），这是一个假值。因此，房间的值将是一个空字符串。

如果 URL 包含带有房间名称的查询字符串，那么`location.search`将返回`'?roomName'`，这是一个真值，所以下一个语句`location.search.split('?')[1]`将被评估，它执行分割并返回数组中的第一个索引（房间名称）。由于它是最后一个真值，room 常量现在将包含房间名称字符串！我们使用短路评估将三行代码简化为一行代码。

关于短路评估的详细信息可以在以下网址找到：[`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_Operators#Short-circuit_evaluation`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Logical_Operators#Short-circuit_evaluation)。

# 设置器和获取器

我们只需要添加一行代码来让用户加入房间：

```js
webrtc.joinRoom(room);
```

这将使用户加入房间，但是一旦用户进入房间，我们需要隐藏`.create-room-area` div 并显示`.info-area` div。这些都在`Home`类的`roomCreated()`方法中。但是该方法依赖于`this.roomName`类变量，该变量应该包含房间名称。因此，我们需要更新一个类变量并从类外部调用`class`方法。

尽管我们可以使用之前创建的`home`对象来做到这一点，但如果我们只能更新类的 room 属性并且它将自动执行操作，那将更有意义。为此，我们可以使用设置器。设置器是用于为对象的属性分配新值的特殊方法。我们以前已经多次使用过获取器和设置器。还记得我们如何获取输入字段的值吗？

```js
const inputValue = this.$roomNameInput.value
```

在这里，值属性是一个获取器。它从`$roomNameInput`对象返回一个值。但是，如果我们这样做：

```js
$roomNameInput.value = 'New Room Name'
```

然后，它将把输入字段的值更改为`'New Room Name'`。这是因为值现在充当设置器，并更新了`$roomNameInput`对象内的属性。

我们将为我们的`Home`类创建一个设置器来加入一个房间。创建一个设置器很简单；我们只需创建一个以`set`关键字为前缀的方法，该方法应该有*正好一个参数*。在您的`Home`类中，添加以下代码：

```js
set room(room) {
  webrtc.joinRoom(room);
  this.roomName = room;
  this.roomCreated();
}
```

现在，在您的`readyToCall`事件处理程序中使用设置器（仅当房间不是空字符串时）：

```js
const home = new Home();

webrtc.on('readyToCall', () => {
  const room = location.search && location.search.split('?')[1];
  if(room) home.room = room;
});
```

添加代码后，在视频通话应用中创建一个房间，然后复制 URL 并粘贴到新标签中。它应该会自动从 URL 获取房间名称并加入房间。如果您能看到房间信息，那么您就可以开始了。我们正接近应用程序的最后阶段--添加和删除视频。

如果和 else 条件后面只有一个语句，不需要`{}`大括号。也就是说，`if (true) console.log('true'); else console.log('false');`将正常工作！但应该避免这样做，因为最好始终使用带有`{}`大括号的`if else`条件。

要创建一个 getter，您只需在方法前面加上`get`而不是`set`，但是该方法不应包含*参数*，并且应*返回一个值*。比如，在您的`Home`类中，您需要使用 getter 知道房间名称。然后，您可以添加以下方法：

```js
get room() {
  return this.roomName;
}
```

如果您尝试在类外部使用`console.log(home.room)`，您应该会得到存储在`roomName`类变量中的值。

# 添加和删除视频

类似于`readyToCall`事件，`SimpleWebRTC`将为在房间中找到的每个视频触发`videoAdded`事件，具有具有视频对象和包含 ID（该用户的唯一 ID）的对等对象的回调函数。

为了测试多个视频，我们将在同一系统的同一浏览器中打开两个标签。这可能会导致反馈损坏您的音频设备，所以保持音量静音！

在`Home`类中创建一个新方法`addRemoteVideo($video, peer)`，如下所示：

```js
class Home {
  ...
  addRemoteVideo($video, peer) {
  }
}
```

让我们为`videoAdded`事件添加另一个事件处理程序，就像我们为`readToCall`事件所做的那样：

```js
webrtc.on('videoAdded', ($video, peer) =>  home.addRemoteVideo($video, peer)); 
```

每当添加视频时，它将调用`Home`类的`addRemoteVideo`方法，并传入视频对象和对等对象。我们有一个`.video-area`的 div，它应该包含所有的视频。因此，我们需要构建一个类似于用于本地视频的新 div 元素，例如：

```js
<div class="video-container" id="container_peerid">
  <video class="video-player"></video>
</div>
```

然后，我们应该将此元素附加到`.video-area` div，它当前由`this.$remotes`变量引用。这很简单，就像我们在上一章中添加`script`元素一样。在您的`addRemoteVideo()`方法中，添加以下代码：

```js
addRemoteVideo($video, peer) {
  const $container = document.createElement('div');
  $container.className = 'video-container';
  $container.id = 'container_' + webrtc.getDomId(peer);

  $video.className = 'video-player';

  $container.appendChild($video);

  this.$remotes.appendChild($container);
}
```

上述代码执行以下操作：

+   首先，我们使用`document.createElement('div')`方法创建一个`div`元素，并将其分配给`$container`对象。

+   然后，我们将`$container`的类名设置为`'video-container'`，ID 设置为`'container_peerid'`。我们可以使用`webrtc.getDomId()`方法从我们收到的对等对象中获取对等 ID。

+   我们收到的`$video`对象是一个 HTML 元素，就像`$container`一样。因此，我们将其分配为类名`'video-player'`。

+   然后，作为最后一步，我们将`$video`作为子元素附加到`$container`中，最后将`$container`作为子元素附加到`this.$remotes`中。

这将使用类和 ID 构造我们需要的 HTML。当用户离开房间时，将触发`videoRemoved`事件，这类似于`videoAdded`事件。每当用户离开房间时，我们需要使用对等 ID 来删除包含 ID`'container_peerid'`的 div，其中`peerid`是离开的用户的 ID。为此，请添加以下代码：

```js
class Home {
  ...

  removeRemoteVideo(peer) {
    const $removedVideo = document.getElementById(peer ? 'container_' + webrtc.getDomId(peer) : 'no-video-found');
    if ($removedVideo) {
      this.$remotes.removeChild($removedVideo);
    }
  }

}
...

webrtc.on('videoRemoved', ($video, peer) => home.removeRemoteVideo(peer));
```

`removeRemoteVideo()`方法将使用对等 ID 查找包含远程视频的 div，并使用`removeChild()`方法从`this.$remotes`对象中删除它。

是时候测试我们的视频通话应用了。在 Chrome 中打开应用程序并创建房间。复制房间 URL 并粘贴到新标签中（保持音量静音！）。可能需要几秒钟，但除非 STUN 对您不起作用，否则您应该在每个标签中看到两个视频。您正在在标签之间传输视频。

第一个视频是你的视频。如果你关闭其中一个标签，你会看到第二个视频会从另一个标签中移除。在我们在其他设备上测试这个应用之前，还有一个功能会让这个应用看起来更棒。那就是增加所选视频的大小。

# 选择视频

目前，所有的视频都很小。因此，我们需要一个功能来放大视频，比如：

+   在桌面上，点击视频将增加视频的大小，并将其移动到视频列表的第一个位置

+   在手机上，点击视频只会增加视频的大小

这听起来不错。为了实现这一点，让我们在我们的`styles.css`文件中添加一些样式：

```js
@media only screen and (max-width: 736px) {
  .video-selected {
    max-height: 70vw;
    max-width: 70vw;
  }
}
@media only screen and (min-width: 736px) {
  .video-selected {
    max-height: 50vh;
    max-width: 50vh;
  }
  .container-selected {
    order: -1;
  }
}
```

我们使用媒体查询添加了两组样式。一组用于手机（`max-width: 736px`），另一组用于桌面（`min-width: 736px`）。

对于每次点击视频，我们应该为该视频添加`.video-selected`类，并为该视频的父 div 添加`.container-selected`类：

+   在手机上，它将把视频的大小增加到视口宽度的 70%。

+   在桌面上，它将把大小增加到视口宽度的 50%，并且还会给其父 div 分配`order: -1`。这样，由于父 div 是 flex 的一部分，它将成为 flex 元素的第一个项目（但其他元素不应该在其样式中包含 order）。

在你的`Home`类中，添加以下方法：

```js
clearSelected() {
  let $selectedVideo = document.querySelector('.video-selected');
  if($selectedVideo) {
    $selectedVideo.classList.remove('video-selected');
    $selectedVideo.parentElement.classList.remove('container-selected');
  }
}
```

这将找到包含`.video-selected`类的视频，并从该视频和该视频的父 div 中移除`.video-selected`类和`.container-selected`类。这很有用，因为我们可以在选择另一个视频之前调用它来清除已选择的视频。

我们可以在`registerClicks()`方法中为本地视频注册点击事件。在`registerClicks()`方法中，添加以下代码：

```js
this.$localVideo.onclick = () => {
  this.clearSelected();
  this.$localVideo.parentElement.classList.add('container-selected');
  this.$localVideo.classList.add('video-selected');
};
```

这将为视频元素及其父级 div 添加所需的类。对于远程视频，我们不能在这里注册点击，因为我们动态创建这些元素。因此，我们要么创建一个事件监听器，要么在创建远程视频元素时注册点击事件。

在这里为每个视频创建一个事件监听器并不太有效，因为当用户离开时，视频将被移除，所以我们将有不需要的事件监听器运行在每个视频上。我们将不得不使用`removeEventListener()`方法来移除这些事件监听器，或者通过在父 div`.video-area`上创建一个事件监听器来避免这种情况。不过，这意味着我们需要筛选`.video-area`内的每次点击，以检查该点击是否是在视频上进行的。

显然，当视频元素被创建时，使用`onclick()`方法注册点击更简单。这样可以避免处理事件监听器的麻烦。在你的`addRemoteVideo()`方法中，在现有代码之后添加以下代码：

```js
$video.onclick = () => {
  this.clearSelected();
  $container.classList.add('container-selected');
  $video.classList.add('video-selected');
};
```

现在尝试在 Chrome 中点击视频。你应该看到视频会增大并移动到列表的第一个位置。恭喜！你已经成功构建了你的视频通话应用！是时候测试视频通话了。

# 视频通话

你已经准备好应用程序了，所以让我们在本地测试一下。首先，为你的应用生成生产构建。你之前在事件注册应用中已经做过这个了。你需要在你的`.env`文件中设置`NODE_ENV=production`。

之后，在你的项目根目录中，关闭 Webpack 开发服务器，运行`npm run webpack`命令。它应该会为你的 JS 和 CSS 文件生成生产构建。文件名将在`dist/manifest.json`文件中。在你的`index.html`页面中包含这些 CSS 和 JS 文件。

现在，在您的项目根文件夹中运行`http-server`。它应该打印出两个 IP 地址。在浏览器中打开以 192 开头的那个。这个 IP 地址对您局域网中的所有设备都是可访问的，除非您使用防火墙阻止了端口。然而，Chrome 将无法显示您的视频！这是因为`getUserMedia()`方法只能在本地主机和 HTTPS URL 中工作。由于我们的本地地址只使用 HTTP，视频将无法工作。

我们可以通过在公共服务器上部署我们的 WebRTC 应用程序并使用来自证书颁发机构的 SSL 证书来添加 HTTPS。然而，对于我们的本地开发环境，我们可以使用自签名证书。来自证书颁发机构的 SSL 证书将受到所有浏览器的信任，但自签名证书将不受信任，因此会显示警告，我们应该在浏览器上手动选择信任该网站的选项。因此，自签名证书不适用于生产，只应用于开发目的。

创建自签名证书是一个复杂的过程，但幸运的是，有一个`npm`包可以在一行命令中完成这个过程。我们需要全局安装这个包，因为它像`http-server`一样是一个命令行工具。在您的终端中运行以下命令：

```js
npm install -g local-ssl-proxy
```

Linux 用户可能需要在他们的命令中添加`sudo`以全局安装软件包。默认情况下，`http-server`将从端口号`8080`提供您的文件。比如，如果您当前的 URL 如下：

```js
http://192.168.1.8:8080
```

然后，打开另一个终端并运行以下命令：

```js
local-ssl-proxy --source 8081 --target 8080
```

在这里，源是新的端口号，目标是 http-server 正在运行的端口号。然后，您应该在 Chrome 中使用新的端口号和`https://`前缀打开相同的 IP 地址，如下面的代码块所示：

```js
https://192.168.1.8:8081
```

如果您在 Chrome 中打开此页面，您应该会收到类似以下截图的警告。在这种情况下，请选择高级，如下图所示：

![](img/00030.jpeg)

点击高级后，您将看到一个类似以下图像的页面，您应该点击继续链接：

![](img/00031.jpeg)

您现在可以使用这个 HTTPS URL 在连接到您的局域网的任何设备上打开应用程序。确保设备之间有足够的距离，以免造成反馈。

# 总结

希望您在构建视频通话应用程序时度过了愉快的时光。在本章中，我们使用 JavaScript 做了一些新的事情，并学习了一些新概念，如 JavaScript WebRTC API 和 SimpleWebRTC 框架。在这个过程中，我们做了很多很酷的事情，比如操纵浏览器历史记录，使用 JavaScript 选择文本，以及处理 URL。此外，我们使用短路评估缩短了一些代码，并学习了在 JavaScript 中操纵类变量的设置器和获取器。

`SimpleWebRTC`还带有许多其他事件和操作，允许您在应用程序中执行更多操作，例如静音麦克风，静音其他人的音频等。如果您感兴趣，可以查看 SimpleWebRTC 主页获取更多示例。

我们知道如何创建可重用的 JavaScript 模块，这是我们在上一章中做的。在下一章中，我们将进一步迈出一步，使用 Web 组件构建我们自己的可重用的 HTML 元素。
