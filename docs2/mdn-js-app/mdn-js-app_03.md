实时浏览器间的通信

为了在网站中实现音频/视频聊天或其他需要实时点对点（浏览器到浏览器）数据传输的功能，或者需要从麦克风、摄像头或其他设备检索音频/视频流，我们不得不使用浏览器插件，如 Java 和 Flash。网站依赖浏览器插件存在各种问题，例如移动浏览器不支持插件，以及插件需要保持更新。因此，引入了 WebRTC 来解决这些问题，即支持 WebRTC 的浏览器提供 API，以实现浏览器之间的实时数据交换，并且无需使用插件即可从物理媒体源检索流。在本章中，我们将讨论 WebRTC 以及 PeerJS 库，该库封装了 WebRTC API，以提供一个易于使用的 API 来与 WebRTC 一起工作。

在本章中，我们将涵盖以下主题：

+   讨论 WebRTC 提供的各种 API

+   从物理媒体输入设备检索流

+   显示媒体流

+   讨论 WebRTC 使用的协议

+   使用 PeerJS 在节点之间交换媒体流和任意数据

+   讨论与 WebRTC 和 PeerJS 基础相关的主题

# 第三章：术语

在我们深入了解 WebRTC 和 PeerJS 之前，您需要了解我们将要使用的某些术语的含义。这些术语将在以下部分中进行讨论。

## 流

**流**是一系列随时间提供的任何类型的数据。流对象表示一个流。通常，将事件处理程序或回调函数附加到流对象上，每当有新数据可用时，就会调用该函数。

**媒体流**是一个数据为音频或视频的流。同样，**媒体源**是一个物理设备、文件或提供音频或视频数据的某种东西。**媒体消费者**也是一个物理设备、API 或使用媒体流的某种东西。

### 注意

WebRTC 允许我们从物理媒体源（如麦克风、摄像头、屏幕等）检索媒体流。我们将在本章后面进一步讨论这一点。

## 点对点网络模型

点对点模型是客户端-服务器模型的相反。在客户端-服务器模型中，服务器向客户端提供资源，而在点对点模型中，网络中的每个节点都充当服务器和客户端，即每个节点提供和消耗资源。点对点模型中的节点直接相互通信。

要建立点对点连接，我们需要一个信令服务器，它用于信令。**信令**是指用于建立点对点连接的节点之间交换的数据。例如，会话控制消息、网络配置等数据需要建立点对点连接。信令服务器实现信令协议，如 SIP、Jingle 或某些其他协议。

根据应用的需求和资源可用性选择一个模型。让我们考虑一些例子：

+   要构建一个视频聊天应用，我们应该使用点对点模型而不是客户端-服务器模型。因为在这种情况下，每个节点都将产生大量数据（或帧），并实时将数据发送到其他节点，服务器需要大量的网络和其他资源，这增加了服务器运行成本。因此，点对点模型是视频聊天应用的最佳选择。例如，Skype 视频聊天就是基于点对点模型的。

+   要构建一个将消息存储在集中式数据库中的文本聊天应用，我们应该使用客户端-服务器模型，因为客户端产生的数据量不是很高，而且你也会希望将消息存储在集中式数据库中。例如，Facebook 消息应用就是基于客户端-服务器模型的。

### 注意

要使用 WebRTC 建立点对点连接，你需要一个信令服务器、STUN 服务器和可选的 TURN 服务器。我们将在本章后面进一步讨论这个问题。

## 实时数据

**实时数据** 是需要无延迟处理和传输的数据。例如，视频聊天、实时分析、实时股价、实时流媒体、文本聊天、实时比分、在线多人游戏数据等等都是实时数据。

实时数据传输是一个难以实现的任务。用于实时数据传输的技术和技术取决于数据量以及数据传输过程中数据丢失是否可容忍。如果实时数据量很大，且数据丢失不可容忍，那么需要大量资源来实现实时数据传输，这使得实现实时数据传输在实际上变得不可能。例如，在视频聊天时，每个用户都会生成大量的帧。如果某些帧丢失，那么这是可容忍的，因此在这种情况下，我们可以使用 UDP 协议作为传输层协议，它不可靠且比 TCP 有更少的开销，这使得 UDP 非常适合视频聊天应用。

### 注意

WebRTC 允许我们使用 SRTP 协议传输它产生的实时媒体流。为了传输任意数据，它使用 SCTP 协议。我们将在本章后面进一步讨论这些协议。

# WebRTC 简介

**Web 实时通信（WebRTC）** 是一种浏览器技术，它能够检索物理媒体源的媒体流，并实时交换媒体流或任何其他数据。它包括三个 API：`MediaStream` 构造函数、`RTCPeerConnection` 构造函数和 `RTCDataChannel` 接口。

简而言之，`MediaStream` 用于检索物理媒体源的流，`RTCPeerConnection` 用于在实时点对点之间交换 `MediaStream`，最后，`RTCDataChannel` 用于在点对点之间交换任意数据。

让我们看看这些 API 是如何工作的。

## MediaStream API

媒体流 API 的两个主要组件是`MediaStream`构造函数和`MediaStreamTrack`接口。

轨道代表媒体源的流。轨道实现了`MediaStreamTrack`接口。轨道可以是音频轨道或视频轨道。也就是说，连接到音频源的轨道是音频轨道，连接到视频源的轨道是视频轨道。可以有一个或多个轨道连接到特定的媒体源。我们还可以将约束附加到轨道上。例如，连接到网络摄像头的轨道可以具有最小视频分辨率和 FPS 等约束。每个轨道都有自己的约束。

您可以使用`MediaStreamTrack`接口的`applyConstraints()`方法在创建轨道后更改轨道的约束。您可以使用`MediaStreamTrack`接口的`getSettings()`方法随时检索应用于轨道的约束。要从一个媒体源断开轨道，即永久停止轨道，我们可以使用`MediaStreamTrack`接口的`stop()`方法。要暂停轨道，即暂时停止轨道，我们可以将`MediaStreamTrack`接口的`enabled`属性赋值为`false`。

### 注意

在[`developer.mozilla.org/en-US/docs/Web/API/MediaStreamTrack`](https://developer.mozilla.org/en-US/docs/Web/API/MediaStreamTrack)了解更多关于`MediaStreamTrack`接口的信息。

轨道可以是本地轨道或远程轨道。本地轨道代表本地媒体源的流；而远程轨道代表远程媒体源的流。您不能对远程轨道应用约束。要确定轨道是本地还是远程，我们可以使用`MediaStreamTrack`接口的`remote`属性。

### 注意

在交换对等方之间的轨道时，我们会遇到远程轨道。当我们向对等方发送本地轨道时，另一个对等方会接收到该轨道的远程版本。

`MediaStream`将多个轨道组合在一起。技术上，它不做任何事情。它只是代表应该以同步方式一起播放、存储或传输的一组轨道。

### 注意

在[`developer.mozilla.org/en/docs/Web/API/MediaStream`](https://developer.mozilla.org/en/docs/Web/API/MediaStream)了解更多关于`MediaStream`构造函数的信息。

`MediaStreamTrack`对象的`getSources()`方法允许我们检索所有媒体设备的 ID，例如扬声器、麦克风、网络摄像头等。如果 ID 代表媒体输入设备，我们可以使用该 ID 创建一个轨道。以下是一个演示此功能的示例：

```js
MediaStreamTrack.getSources(function(sources){
  for(var count = 0; count < sources.length; count++)
  {
    console.log("Source " + (count + 1) + " info:");
    console.log("ID is: " + sources[count].id);

    if(sources[count].label == "")
    {
      console.log("Name of the source is: unknown");
    }
    else
    {
      console.log("Name of the source is: " + sources[count].label);
    }

    console.log("Kind of source: " + sources[count].kind);

    if(sources[count].facing == "")
    {
      console.log("Source facing: unknown");
    }
    else
    {
      console.log("Source facing: " + sources[count].facing);
    }
  }
})
```

每个人的输出结果都可能不同。以下是我得到的输出结果：

```js
Source 1 info:
ID is: 0c1cb4e9e97088d405bd65ea5a44a20dab2e9da0d298438f82bab57ff9787675
Name of the source is: unknown
Kind of source: audio
Source facing: unknown
Source 2 info:
ID is: 68fb69033c86a4baa4a03f60cac9ad1c29a70f208e392d3d445f3c2d6731f478
Name of the source is: unknown
Kind of source: audio
Source facing: unknown
Source 3 info:
ID is: c83fc025afe6c7841a1cbe9526a6a4cb61cdc7d211dd4c3f10405857af0776c5
Name of the source is: unknown
Kind of source: video
Source facing: unknown
```

## navigator.getUserMedia

有各种 API 返回包含轨道的`MediaStream`。其中一种方法是`navigator.getUserMedia()`。使用`navigator.getUserMedia()`，我们可以从媒体输入源检索流，例如麦克风、网络摄像头等。以下是一个演示此功能的示例：

```js
navigator.getUserMedia = navigator.getUserMedia || navigator.webkitGetUserMedia || navigator.mozGetUserMedia;

var constraints = {
  audio: true, 
  video: {
    mandatory: {
      minWidth: 640,
      minHeight: 360
    },
    optional: [{
      minWidth: 1280
    }, {
      minHeight: 720
    }]
  }
}

var av_stream = null;

navigator.getUserMedia(constraints, function(mediastream){
  av_stream = mediastream; //this is the MediaStream
}, function(err){
  console.log("Failed to get MediaStream", err);
});
```

当您运行前面的代码时，浏览器将显示一个弹出窗口，请求用户授权。用户必须授权代码访问媒体输入设备。

默认情况下，使用 `getUserMedia()` 时附加到轨道的媒体输入设备取决于浏览器。一些浏览器允许用户选择他们想要使用的音频和视频设备，而其他浏览器则使用操作系统中列出的默认音频和视频设备。

我们还可以在约束对象的 `audio` 或 `video` 属性的 `mandatory` 属性中提供 `sourceId` 属性，将其分配给媒体输入设备的 ID，以便 `getUserMedia()` 将轨道附加到这些设备。因此，如果有多个网络摄像头和麦克风，则可以使用 `MediaStreamTrack.getSources()` 允许用户选择媒体输入设备，并将此媒体输入设备 ID 提供给 `getUserMedia()`，而不是依赖于浏览器，因为浏览器不能保证它是否会允许用户选择媒体输入设备。

它接受的第一个参数是一个包含音频和视频轨道约束的约束对象。强制约束是必须应用的约束。可选表示它们不是非常重要，因此如果无法应用它们，则可以省略。

音频轨道的一些重要约束是 `volume`、`sampleRate`、`sampleSize` 和 `echoCancellation`。视频轨道的一些重要约束是 `aspectRatio`、`facingMode`、`frameRate`、`height` 和 `width`。如果没有提供约束，则使用其默认值。

如果您不想创建音频或视频轨道，可以简单地将 `audio` 或 `video` 属性设置为 `false`。

我们可以使用 `MediaStream` 的 `getTracks()` 方法检索 `MediaStream` 的轨道。同样，我们可以使用 `addTrack()` 和 `removeTrack()` 方法分别添加或删除轨道。每当添加一个轨道时，就会触发 `onaddtrack` 事件。同样，每当删除一个轨道时，就会触发 `onendtrack` 事件。

如果我们已经有了一些轨道，则可以直接使用 `MediaStream` 构造函数创建带有这些轨道的 `MediaStream`。`MediaStream` 构造函数接受一个轨道数组，并返回带有添加到其中的轨道引用的 `MediaStream`。

从 `MediaStream` 轨迹读取数据的 API 被称为 `MediaStream` 消费者。一些 `MediaStream` 消费者包括 `<audio>` 标签、`<video>` 标签、`RTCPeerConnection`、`Media Recorder` API、`Image Capture` API、`Web Audio` API 等。

下面是一个示例，演示如何在视频标签中显示 `MediaStream` 轨迹的数据：

```js
<!doctype html>
<html>
  <body>

    <video id="myVideo"></video>
    <br>
    <input value="Pause" onclick="pause()" type="button" />

    <script type="text/javascript">

      navigator.getUserMedia = navigator.getUserMedia || navigator.webkitGetUserMedia || navigator.mozGetUserMedia;

      var constraints = {
        audio: true, 
        video: true
      }

      var av_stream = null;

      navigator.getUserMedia(constraints, function(mediastream){

        av_stream = mediastream;

        document.getElementById("myVideo").setAttribute("src", URL.createObjectURL(mediastream));
        document.getElementById("myVideo").play();
      }, function(err){
        console.log("Failed to get MediaStream", err);
      });

      function pause()
      {
        av_stream.getTracks()[0].enabled = !av_stream.getTracks()[0].enabled;
        av_stream.getTracks()[1].enabled = !av_stream.getTracks()[1].enabled;
      }

    </script>
  </body>
</html>
```

在这里，我们有一个 `<video>` 标签和一个用于暂停它的按钮。视频标签接受一个 URL 并显示资源。

### 注意

在 HTML5 之前，HTML 标签和 CSS 属性只能从 `http://` 和 `file://` URLs 读取数据。然而，在 HTML5 中，它们可以读取 `blob://`、`data://`、`mediastream://` 和其他类似 URL。

要在 `<video>` 标签中显示 `MediaStream` 的输出，我们需要使用 `URL.createObjectURL()` 方法，该方法接受一个 blob、文件对象或 `MediaStream`，并提供一个 URL 来读取其数据。`URL.createObjectURL()` 会占用额外的内存和 CPU 时间来通过 URL 提供对传递给它的值的访问，因此，当我们不再需要 URL 时，明智的做法是使用 `URL.revokeObjectURL()` 来释放 URL。

如果 `MediaStream` 中有多个音频和视频轨道，那么 `<video>` 会读取第一个音频和视频轨道。

## RTCPeerConnection API

`RTCPeerConnection` 允许两个浏览器实时交换 `MediaStream`。`RTCPeerConnection` 是 `RTCPeerConnection` 构造函数的一个实例。

### 建立点对点连接

要建立点对点连接，需要一个信令服务器。通过信令服务器，端点交换建立点对点连接所需的数据。实际的数据传输是在端对端之间直接进行的。信令服务器仅用于交换建立点对点连接的先决条件。一旦建立了点对点连接，两个端点都可以从信令服务器断开连接。信令服务器不需要是一个高度配置的服务器，因为实际数据不是通过它传输的。单个点对点连接的数据传输量将在一些 KB 左右，因此可以使用一个相当不错的服务器来进行信令。

信令服务器通常使用信令协议，但如果它是一个能够传递两个端点之间消息的 HTTP 服务器，也是可以的。WebRTC 并不强制我们使用任何特定的信令协议。

例如，假设有两个用户，Alice 和 Bob，他们分别在不同的浏览器上。如果 Alice 想与 Bob 建立点对点连接进行聊天，那么他们之间建立点对点连接的方式如下：

1.  他们都将连接到信令服务器。

1.  然后 Alice 将通过信令服务器向 Bob 发送请求，请求聊天。

1.  信令服务器可以可选地检查 Alice 是否允许与 Bob 聊天，以及 Alice 和 Bob 是否已登录。如果是的话，信令服务器将消息传递给 Bob。

1.  Bob 收到请求并通过信令服务器向 Alice 发送消息，确认建立点对点连接。

1.  现在它们两个都需要交换与会话控制、网络配置和媒体能力相关的消息。所有这些消息都是通过 `RTCPeerConnection` 交换的。因此，它们两个都需要创建一个 `RTCPeerConnection`，初始化它，并将事件处理器附加到 `RTCPeerConnection` 上，当它想要通过信令服务器发送消息时，`RTCPeerConnection` 会将消息以 **会话描述协议** (**SDP**) 格式传递给事件处理器。从信令服务器接收到的 `RTCPeerConnection` 消息必须以 SDP 格式提供给 `RTCPeerConnection`，也就是说，`RTCPeerConnection` 只能理解 SDP 格式。你需要使用自己的编程逻辑来分割自定义消息和 `RTCPeerConnection` 的消息。

前面的步骤似乎没有问题；然而，存在一些主要问题。对等方可能位于 NAT 设备或防火墙后面，因此找到它们的公网 IP 地址是一项挑战性任务，有时实际上不可能找到它们的 IP 地址。那么，当对等方可能位于 NAT 设备或防火墙后面时，`RTCPeerConnection` 如何找到对等方的 IP 地址？

`RTCPeerConnection` 使用一种称为 **交互式连接建立** (**ICE**) 的技术来解决所有这些问题。

ICE 涉及 **NAT 会话遍历实用工具** (**STUN**) 和 **NAT 旁路使用中继** (**TURN**) 服务器来解决这些问题。STUN 服务器用于找到对等方的公网 IP 地址。如果对等方的 IP 地址找不到，或者由于其他原因无法建立点对点连接，则使用 TURN 服务器来重定向流量，也就是说，两个对等方通过 TURN 服务器进行通信。

我们只需要提供 STUN 和 TURN 服务器的地址，`RTCPeerConnection` 会处理其余部分。Google 提供了一个公共 STUN 服务器，被每个人使用。构建 TURN 服务器需要大量资源，因为实际数据流通过它。因此，WebRTC 使使用 TURN 服务器成为可选的。如果 `RTCPeerConnection` 无法在两个对等方之间建立直接通信，并且没有提供 TURN 服务器，对等方就没有其他通信方式，对等方连接建立失败。

### 注意

WebRTC 不提供任何使信令安全的方法。确保信令安全是你的工作。

### 传输 MediaStream

我们看到了 `RTCPeerConnection` 如何建立点对点连接。现在，为了传输 `MediaStream`，我们只需要将 `MediaStream` 的引用传递给 `RTCPeerConnection`，它就会将 `MediaStream` 传输给已连接的对等方。

### 注意

当我们说 `MediaStream` 被传输时，我们指的是单个轨道的流被传输。

以下是一些关于 `MediaStream` 传输你需要了解的事项：

+   `RTCPeerConnection` 使用 SRTP 作为应用层协议和 UDP 作为传输层协议来传输 `MediaStream`。SRTP 是为实时媒体流传输而设计的。

+   UDP 不保证数据包的顺序，但 SRTP 会处理帧的顺序。

+   **数据报传输层安全性**（**DTLS**）协议用于确保 `MediaStream` 传输的安全性。因此，在传输 `MediaStream` 时，你不必担心安全问题。

+   远程对等方接收的轨道约束可能与本地轨道的约束不同，因为 `RTCPeerConnection` 会根据带宽和其他网络因素自动修改流，以加快传输速度，实现实时数据传输。例如，在传输过程中，`RTCPeerConnection` 可能会降低视频流的分辨率和帧率。

+   如果你向正在发送的 `MediaStream` 中添加或移除轨道，那么 `RTCPeerConnection` 会通过与其他对等方通过信令服务器通信来更新另一对等方的 `MediaStream`。

+   如果你暂停正在发送的轨道，那么 `RTCPeerConnection` 会暂停轨道的传输。

+   如果你停止正在发送的轨道，`RTCPeerConnection` 会停止轨道的传输。

### 注意

你可以通过单个 `RTCPeerConnection` 发送和接收多个 `MediaStream` 实例，也就是说，你不需要创建多个 `RTCPeerConnection` 实例来发送和接收来自对等方的多个 `MediaStream` 实例。每次你向 `RTCPeerConnection` 添加或移除新的 `MediaStream` 时，对等方都会通过信令服务器交换与此相关的信息。

## RTCDataChannel API

`RTCDataChannel` 用于在对等方之间传输除 `MediaStream` 之外的数据，以传输任意数据。建立点对点连接以传输任意数据的机制与前面章节中解释的机制类似。

`RTCDataChannel` 是一个实现了 `RTCDataChannel` 接口的对象。

以下是一些关于 `RTCDataChannel` 你需要知道的事情：

+   `RTCDataChannel` 使用 SCTP over UDP 作为传输层协议来传输数据。它不使用无层的 SCTP 协议，因为 SCPT 协议不被许多操作系统支持。

+   SCTP 可以配置为可靠性和交付顺序，与 UDP 不同，UDP 不可靠且无序。

+   `RTCDataChannel` 也使用 DTLS 来确保数据传输的安全性。因此，在通过 `RTCDataChannel` 传输数据时，你不必担心安全问题。

### 注意

我们可以在浏览器之间打开多个点对点连接。例如，我们可以有三个点对点连接，即第一个用于网络摄像头流传输，第二个用于文本消息传输，第三个用于文件传输。

# 使用 PeerJS 的 WebRTC 应用程序

**PeerJS** 是一个客户端 JavaScript 库，它提供了一个易于使用的 API 来处理 WebRTC。它仅提供 API 用于在节点之间交换 `MediaStream` 和任意数据。它不提供用于处理 `MediaStream` 的 API。

## PeerServer

**PeerServer** 是一个开源的信令服务器，PeerJS 使用它来建立点对点连接。PeerServer 使用 Node.js 编写。如果您不想运行自己的 PeerServer 实例，则可以使用 PeerServer 云，该云为公共使用托管 PeerServer。PeerServer 云允许您免费建立最多 50 个并发连接。

一个唯一的 ID 识别连接到 PeerServer 的每个节点。PeerServer 本身可以生成 ID，或者节点可以提供自己的 ID。为了使一个节点与另一个节点建立点对点连接，它只需要知道另一个节点的 ID。

当您想向 PeerServer 添加更多功能或需要支持超过 50 个并发连接时，您可能想运行自己的 PeerServer 实例。例如，如果您想检查用户是否已登录到 PeerServer，那么您需要添加此功能并托管自己的定制 PeerServer。

在本章中，我们将使用 PeerServer 云，但在下一章中，我们将创建自己的 PeerServer 实例。因此，为了继续本章的内容，请在 PeerServer 云上创建一个账户并检索 API 密钥。每个应用程序都会获得一个 API 密钥来访问 PeerServer 云。如果您正在托管自己的 PeerServer，那么您不需要 API 密钥。API 密钥由 PeerServer 云用于跟踪应用程序建立的连接总数。要创建账户并检索 API 密钥，请访问 [`peerjs.com/peerserver`](http://peerjs.com/peerserver)。

## PeerJS API

让我们通过创建一个简单的应用程序来讨论 PeerJS API，该应用程序允许用户与任何他们拥有 ID 的用户交换视频和文本消息。

在您的 Web 服务器上创建一个名为 `peerjs-demo` 的目录，并在其中放置一个名为 `index.html` 的文件。

在 `index.html` 文件中，我们首先需要入队 `PeerJS` 库。从 [`peerjs.com/`](http://peerjs.com/) 下载 `PeerJS`。在撰写本文时，PeerJS 的最新版本是 0.3.14。我建议您在以下示例中坚持使用这个版本。将以下起始代码放置在 `index.html` 文件中：

```js
<!doctype html>
<html>
  <head>
    <title>PeerJS Demo</title>
  </head>
  <body>

    <!-- Place HTML code here -->

    <script src="img/peer.min.js"></script>
    <script>
      //place JavaScript code here
    </script>
  </body>
</html>
```

在这里，我入队了 PeerJS 的精简版本。

PeerJS API 包含三个主要构造函数，如下所示：

+   `Peer`：`Peer` 实例代表网络中的一个节点。节点连接到信令服务器和 STUN，并且可选地连接到 TURN。

+   `DataConnection`：DataConnection（即 `DataConnection` 的实例）代表一个点对点连接，用于交换任意数据。技术上，它封装了 `RTCDataChannel`。

+   `MediaConnection`：MediaConnection（即 `MediaConnection` 的实例）代表一个用于交换 `MediaStream` 的点对点连接。技术上，它封装了 `RTCPeerConnection`。

如果一个节点想要与另一个节点建立 `DataConnection` 或 `MediaConnection`，那么它只需要知道另一个节点的 ID。PeerJS 不会给另一个节点提供接受或拒绝 `DataConnection` 的选项。在 `MediaConnection` 的情况下，PeerJS 也不会给另一个节点提供接受或拒绝 `MediaConnection` 的选项，但 `MediaConnection` 将在另一个节点程序化激活以传输 `MediaStream` 之前保持不活跃，否则 `MediaStream` 将不会传输。因此，我们可以编写自己的逻辑来让其他用户接受或拒绝 `DataConnection` 或 `MediaConnecton`，也就是说，一旦 `DataConnection` 或 `MediaConnection` 建立，我们就可以通过询问用户的意见来取消它。

### 注意

目前，一个 `MediaConnection` 只能传输一个 `MediaStream`。在 PeerJS 的未来版本中，单个 `MediaConnection` 将支持传输多个 MediaStreams。

现在，我们需要创建一个 `<video>` 标签来显示视频，一个连接到节点的按钮，以及一个用于发送消息的文本框。以下是显示所有这些内容的 HTML 代码：

```js
<video id="remoteVideo"></video>
<br>
<button onclick="connect()">Connect</button>
<br>
<input type="text" id="message">
<button onclick="send_message()">Send Message</button>
```

现在，当页面加载时，我们需要连接到 `PeerServer` 和 ICE 服务器，以便其他节点可以与我们交谈，并且当用户点击连接按钮时，我们可以建立 `DataConnection` 和 `MediaConnection`。以下是这个功能的代码：

```js
var peer = null;

window.addEventListener("load", function(){
  var id = prompt("Please enter an unique name");

  peer = new Peer(id, {key: "io3esxy6y43zyqfr"}); 

  peer.on("open", function(id){
    alert("Connected to PeerServer successfully with ID: " + id);
  });

  peer.on("error", function(err){
    alert("An error occured. Error type: " + err.type);
  })

  peer.on("disconnected", function(){
    alert("Disconnected from signaling server. You ID is taken away. Peer-to-peer connections is still intact");
  })

  peer.on("close", function(){
    alert("Connection to signaling server and peer-to-peer connections have been killed. You ID is taken away. You have been destroyed");
  })

  peer.on("connection", function(dataConnection){
    setTimeout(function(){
      if(confirm(dataConnection.peer + " wants to send data to you. Do you want to accept?"))
      {
        acceptDataConnection(dataConnection);
      }
      else
      {
        dataConnection.close();
      }
    }, 100)
  })

  peer.on("call", function(mediaConnection){
    setTimeout(function(){
      if(confirm("Got a call from " + mediaConnection.peer + ". Do you want to pick the call?"))
      {
        acceptMediaConnection(mediaConnection);
      }
      else
      {
        mediaConnection.close();
      }
    }, 100);
  })
});
```

下面是如何使代码工作的：

+   首先，我们显示了一个提示框来获取输入 ID，这样每个节点都可以决定自己的 ID。

+   然后我们使用 ID 和 PeerServer 云密钥创建了一个 `Peer` 实例。在这里，我们没有提供信令和 ICE 服务器的 URL，因此，PeerJS 将使用 PeerServer 云作为信令服务器和 Google 的公共 STUN 服务器。它将不会使用任何 TURN 服务器。一旦创建了一个 `Peer` 实例，该实例就会连接到信令服务器并注册给定的 ID。

+   然后，我们向 `peer` 对象附加了五个事件处理器。

+   当连接到 `PeerServer` 成功时，会触发 `open` 事件。

+   当 `peer` 对象出现错误时，会触发 `error` 事件。

+   当与信令服务器的连接断开时，会触发 `disconnected` 事件。与信令服务器的连接可能会因为网络问题或手动调用 `peer.disconnect()` 方法而断开。一旦断开连接，你的 ID 可能会被其他人占用。你可以尝试使用 `peer.reconnect()` 方法使用相同的 ID 重新连接。你可以使用 `peer.disconnect` 布尔属性检查 `peer` 是否连接到信令服务器。

+   当 `peer` 被销毁时，即它不能再使用时，所有 `MediaConnections` 和 `DataConnections` 都会被终止，与信令服务器的连接会被终止，ID 会被移除，等等。当你不再需要它时，你可能想要手动销毁 `peer`。你可以使用 `peer.destroy()` 方法销毁一个节点。

+   当其他节点与你建立 `DataConnection` 时，会触发 `connection` 事件。正如我之前所说的，`DataConnection` 是无需进一步许可就建立的，但如果你想的话，你可以在建立后立即关闭它。在这里，我们让用户决定是否想要继续或关闭由其他节点建立的 `DataConnection`。事件处理程序通过表示当前建立的 `DataConnection` 的参数接收 `DataConnection` 的实例。

+   当其他节点与你建立 `MediaConnection` 时，会触发 `call` 事件。在这里，我们也让用户决定是否想要继续或关闭由其他节点建立的 `MediaConnection`。事件处理程序通过表示当前建立的 `MediaConnection` 的参数接收 `MediaConnection` 的实例。

+   在这里，在 `call` 和 `connection` 事件处理程序中，我们异步显示确认弹出框，以防止阻塞某些浏览器中导致问题的事件处理程序的执行，即阻塞它将无法建立 `DataConnection` 和 `MediaConnection`。

现在，让我们实现 `acceptDataConnection()` 和 `acceptMediaConnection()` 函数，这样我们就可以在与其他节点建立 `DataConnection` 或 `MediaConnection` 时显示文本消息和远程 `MediaStream`。以下是代码：

```js
navigator.getUserMedia = navigator.getUserMedia || navigator.webkitGetUserMedia || navigator.mozGetUserMedia;

var myDataConnection = null;
var myMediaConnection = null;

function acceptDataConnection(dataConnection)
{
  myDataConnection = dataConnection;

  dataConnection.on("data", function(data){
    alert("Message from " + dataConnection.peer + ".\n" + data)
  })

  dataConnection.on("close", function(data){
    alert("DataConnecion closed");
  })

  dataConnection.on("error", function(err){
    alert("Error occured on DataConnection. Error: " + err);
  })
}

function acceptMediaConnection(mediaConnection)
{
  myMediaConnection = mediaConnection;

  mediaConnection.on("stream", function(remoteStream){

    document.getElementById("remoteVideo").setAttribute("src", URL.createObjectURL(remoteStream));
    document.getElementById("remoteVideo").play();
  })

  mediaConnection.on("close", function(data){
    alert("MediaConnecion closed");
  })

  mediaConnection.on("error", function(err){
    alert("Error occured on MediaConnection. Error: " + err);
  })

  navigator.getUserMedia({video: true, audio: true}, function(mediaStream) {
    mediaConnection.answer(mediaStream);
  }, function(e){ alert("Error with MediaStream: " + e); });
}
```

这就是前面代码的工作原理：

+   在 `acceptDataConnection()` 函数中，我们将三个事件处理程序附加到 `DataConnection` 上。当其他节点向我们发送数据时，会触发 `data` 事件。当 `DataConnection` 关闭时，会触发 `close` 事件。最后，当 `DataConnection` 上发生错误时，会触发 `error` 事件。我们可以使用 `dataConnection.close()` 方法手动关闭 `DataConnection`。

+   在 `acceptMediaConnection()` 函数中，我们附加了三个事件处理程序并将我们的 `MediaStream` 转移到其他节点。当其他节点发送我们 `MediaStream` 时，会触发 `stream` 事件。当 `MediaConnection` 关闭时，会触发 `close` 事件。最后，我们通过传递我们的 `MediaStream` 使用 `mediaConnection.answer()` 方法激活 `MediaConnection`。在 `MediaConnection` 激活后，将触发 `stream` 事件。

我们已经完成了处理由其他节点与我们建立的 `MediaConnection` 或 `DataConnection` 的代码。现在我们需要编写一个代码来创建用户点击 **连接** 按钮时触发的 `MediaConnection` 和 `DataConnection`。以下是代码：

```js
function connect()
{
  var id = prompt("Please enter other peer ID");
  establishDataConnection(id);
  establishMediaConnection(id);
}

function establishDataConnection(id)

{
  var dataConnection = peer.connect(id, {reliable: true, ordered: true});

  myDataConnection = dataConnection;

  dataConnection.on("open", function(){
    alert("DataConnecion Established");
  });

  dataConnection.on("data", function(data){
    alert("Message from " + dataConnection.peer + ".\n" + data)
  })

  dataConnection.on("close", function(data){
    alert("DataConnecion closed");
  })

  dataConnection.on("error", function(err){
    alert("Error occured on DataConnection. Error: " + err);
  })
}

function establishMediaConnection(id)
{
  var mediaConnection = null;

  navigator.getUserMedia({video: true, audio: true}, function(mediaStream) {
    mediaConnection = peer.call(id, mediaStream);

    myMediaConnection = mediaConnection;

    mediaConnection.on("stream", function(remoteStream){
      document.getElementById("remoteVideo").setAttribute("src", URL.createObjectURL(remoteStream));
      document.getElementById("remoteVideo").play();
    })

    mediaConnection.on("error", function(err){
      alert("Error occured on MediaConnection. Error: " + err);
    })

    mediaConnection.on("close", function(data){
      alert("MediaConnecion closed");
    })
  }, function(e){ alert("Error with MediaStream: " + e); });
}
```

下面是如何实现代码的：

+   首先，我们要求用户输入另一个用户的 ID。

+   然后，我们建立了 `DataConnection`。要与另一个用户建立 `DataConnection`，我们需要调用 `Peer` 实例的 `connect()` 方法并传入其他用户的 ID。我们还使 `DataConnection` 可靠且有序。然后，我们附加了事件处理器。我们还了解了 `data`、`close` 和 `error` 事件的工作方式。当 `DataConnection` 建立时，会触发 `open` 事件。

+   在建立 `DataConnection` 之后，我们建立了 `MediaConnection`。要建立 `MediaConnection`，我们需要调用 `Peer` 实例的 `call()` 方法。我们需要将 `MediaStream` 传递给 `call()` 方法。最后，我们附加了事件处理器。当其他用户调用 `MediaConnection` 实例的 `answer()` 方法时，即当 `MediaConnection` 激活时，将触发 `stream` 事件。

现在我们最后需要做的是编写代码，以便当用户点击发送消息按钮时发送消息。以下是相应的代码：

```js
function send_message()
{
  var text = document.getElementById("message").value;

  myDataConnection.send(text);
}
```

要通过 `MediaConnection` 发送数据，我们需要调用 `MediaConnection` 实例的 `send()` 方法。在这里，我们发送一个字符串，但您可以传递任何类型的数据，包括 blob 和对象。

现在，为了测试应用程序，请在两个不同的浏览器、设备或标签页中打开 `index.html` 页面的 URL。我假设您已经在两个不同的设备上打开了 URL。在每个设备上，提供不同的 ID 以识别用户。然后在任何一台设备上点击连接按钮并输入另一个用户的 ID。现在在另一台设备上接受请求。一旦完成，这两台设备将能够显示彼此的摄像头视频和麦克风音频。您还可以在它们之间发送消息。

### 注意

您可以在 [`peerjs.com/docs/#api`](http://peerjs.com/docs/#api) 找到 PeerJS API 的官方文档。

# 杂项

在撰写本书时，WebRTC 规范尚未最终确定。WebRTC 的整体功能和运作方式已经确定，只是 API 仍在开发中。

例如，WebRTC 引入了对 `navigator.getUserMedia()` 方法的替代方案，即 `navigator.mediaDevices.getUserMedia()` 方法。在撰写本书时，`navigator.mediaDevices.getUserMedia()` 在任何浏览器中都不受支持。它们之间的区别在于，`navigator.mediaDevices.getUserMedia()` 方法基于承诺模式，而 `navigator.getUserMedia()` 基于回调模式。目前，由于向后兼容性的原因，没有计划废弃 `navigator.getUserMedia()`，但将来，由于 WebRTC 希望使用承诺模式实现所有 API，因此维护多个执行相同功能的 API 将变得困难。同样，`navigator.mediaDevices.enumerateDevice()` 是 `MediaStreamTrack.getSources()` 的替代方案，即 `navigator.mediaDevices.enumerateDevice()` 基于承诺模式。

### 注意

您可以在[`www.w3.org/TR/#tr_Web_Real_Time_Communication`](http://www.w3.org/TR/#tr_Web_Real_Time_Communication)找到 WebRTC 的官方规范。

由于同一功能存在多个 API，每个 API 都有不同的浏览器支持，WebRTC 提供了一个名为`adapter.js`的脚本，它是一个垫片，用于隔离网站免受规范变化和前缀差异的影响。您可以在[`github.com/webrtc/adapter`](https://github.com/webrtc/adapter)找到这个垫片。

WebRTC 有一个 GitHub 仓库，其中包含许多示例项目，展示了可以使用 WebRTC 构建的一些东西。您可以在[`github.com/webrtc/samples`](https://github.com/webrtc/samples)找到这个仓库。只需查看示例及其源代码，您就可以了解更多关于 WebRTC 的知识。

# 摘要

在本章中，我们通过创建一个简单的应用讨论了 WebRTC 和 PeerJS 的基础知识。我们讨论了 WebRTC 用于实现实时点对点通信和读取物理媒体源流的各种协议、技术和其他技术。我们还概述了 PeerServer。现在您应该已经熟悉了使用 PeerServer 云构建任何类型的 WebRTC 应用。

在下一章中，我们将使用自定义的 PeerServer 构建一个高级的 WebRTC 应用。
