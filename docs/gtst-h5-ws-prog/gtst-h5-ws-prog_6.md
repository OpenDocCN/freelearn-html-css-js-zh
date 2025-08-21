# 第六章：错误处理和备用方案

到目前为止，您一定熟悉了 WebSocket 的功能，并且对全双工通信的强大有了一定的了解。然而，WebSocket 的好处是建立在 HTML5 之上的，并且在很大程度上依赖于浏览器的全面支持。当您想要实现的功能不受受众使用的手段支持时会发生什么？您会让您的客户离开吗？这听起来不是一个好主意。幸运的是，通过一点额外的努力，您可以实现、模仿并大部分模拟 WebSocket 的行为。

WebSocket 是未来友好的方式，但为了支持尽可能广泛的受众，您需要一些备用技术。

# 错误处理

在处理错误时，您必须考虑内部和外部参数。内部参数包括由于代码中的错误或意外的用户行为而产生的错误。外部错误与应用程序无关，而是与您无法控制的参数相关。最重要的是网络连接。任何交互式的双向网络应用程序都需要一个活动的互联网连接。

## 检查网络可用性

想象一下，您的用户正在享受您的 Web 应用程序，突然在他们的任务中间，网络连接变得无响应。在现代原生桌面和移动应用程序中，检查网络可用性是一项常见的任务。最常见的方法是简单地向一个应该处于活动状态的网站（例如[`www.google.com`](http://www.google.com)）发出 HTTP 请求。如果请求成功，桌面或移动设备就知道有活动的连接。

同样，HTML 有`XMLHttpRequest`用于确定网络可用性。不过，HTML5 使得这更加容易，并引入了一种检查浏览器是否能接受 Web 响应的方法。这是通过 navigator 对象实现的：

```js
if (navigator.onLine) { 
  alert("You are Online");
}
else {

  alert("You are Offline");
}
```

离线模式意味着设备未连接或用户从浏览器工具栏中选择了离线模式。

以下是在 WebSocket 关闭事件发生时通知用户网络不可用并尝试重新连接的方法：

```js
socket.onclose = function (event) {
  // Connection closed.
  // Firstly, check the reason.
  if (event.code != 1000) {
    // Error code 1000 means that the connection was closed normally.
    // Try to reconnect.
    if (!navigator.onLine) {
      alert("You are offline. Please connect to the Internet and try   again.");
    }
  }
}
```

前面的代码非常简单。它检查错误代码以确定 WebSocket 连接是否已成功关闭。错误代码 1000 将确定这一点。如果关闭事件是由于错误而引发的，代码将不是 1000。在这种情况下，代码会检查连接性并适当地通知用户。

您可能会注意到这是一个 HTML5 功能。稍后，我们将讨论 polyfills，以下是两个用于检查网络连接性的 polyfills：

+   [`github.com/remy/polyfills/blob/master/offline-events.js`](https://github.com/remy/polyfills/blob/master/offline-events.js)

+   [`nouincolor.com/heyoffline.js/`](http://nouincolor.com/heyoffline.js/)

第一个方法是使用`XMLHttpRequest`，类似于智能手机 API 所做的事情。

# 备用解决方案

在现实生活中，人们更喜欢进行身体接触，因为这样更直接和高效，但这不应该是认识某人的唯一方式。有很多情况下你无法握手，所以你需要找到其他的沟通方式。

HTML5 的悲哀现实是，并非每个浏览器都能完全支持它。特别是考虑到新的 JavaScript API，不同浏览器之间仍然存在主要或次要的差异。然而，即使浏览器供应商决定为其当前版本提供完全相同的功能，仍然会有人无法或不想更新。根据 StatCounter 和 W3Counter 的数据，截至 2013 年 3 月，桌面浏览的大部分份额属于 Google Chrome，其次是 Microsoft Internet Explorer 和 Mozilla Firefox。

Internet Explorer 8 仍占 7％，Internet Explorer 7 占 5％，Safari 5.1 占 3％。总共 15％的份额转化为您可能不想错过的客户数量。

这里是备用解决方案，可以处理这种情况，并为旧版浏览器的用户提供一个优雅的缩减体验。如今有两种流行的备用方案，**插件**（如 Flash 或 Silverlight）和**JavaScript hacks**，正式称为**polyfills**。

## JavaScript polyfills

我们首先来研究 polyfills，因为它们更接近原生网络。JavaScript polyfills 是模拟未来功能的解决方案和库，通过为旧版浏览器提供支持来实现。目前，几乎所有 HTML5 特定功能（如 canvas、storage、geolocation、WebSockets、CSS3 等）都有 polyfill 解决方案。

应该并行使用 polyfill 解决方案和基于标准的、符合 HTML5 的 API。

如果您需要同时实现 HTML5 和 polyfill 解决方案，为什么不只实现第二个并节省时间和金钱呢？好吧，以下是您应该同时使用两者的四个原因：

1.  更好的用户体验：使用 HTML5 时，您为访问者提供了最佳和最顺畅的体验。一切都由浏览器处理，您只需专注于应用程序的需求。使用 polyfill 来解决特定问题时，最终产品无法达到相同的质量。当然，提供一些东西总比什么都不提供要好，但是 polyfill 只是为那些运行较差的浏览器打补丁。

1.  性能：原生 HTML5 解决方案和 polyfill 插件之间最显著的优势是性能。当您请求 JavaScript 文件时，您需要额外的资源，这会增加加载时间。此外，JavaScript 插件的运行速度远远慢于原生浏览器实现的方法。关于 WebSockets，该协议旨在提供双向全双工通信。这是您可以实现这种类型的最快方式。而 polyfill 所能做的就是简单地模拟全双工通信，使用传统的 AJAX 轮询。我们已经看到 AJAX 轮询比 WebSockets 慢得多。

1.  面向未来：现在使用 HTML5 可以让您的网站或应用程序在任何未来的浏览器更新中自动增强。例如，三年前使用 canvas 的人在 Internet Explorer 更新到第 9 版时自动受益。

1.  符合标准：尽管内容而不是网络标准应该是我们的首要任务，但了解我们当前的实现是否符合正式的技术规范也是好的。此外，网络标准提出了所谓的“最佳实践”。尽管 polyfills 通常由有效的 JavaScript 代码组成，但大多数时候它们需要通过插入必要的非标准代码来解决特定浏览器的错误和不一致性。

### 流行的 polyfills

**Modernizr**，一个用于检测 HTML5 和 CSS3 功能的知名库，提供了一系列 HTML5 polyfills 的列表，可以在支持旧版浏览器时让您的生活更加轻松。无论您使用哪种 HTML5 功能，都可以在[`github.com/Modernizr/Modernizr/wiki/HTML5-Cross-Browser-Polyfills`](https://github.com/Modernizr/Modernizr/wiki/HTML5-Cross-Browser-Polyfills)找到相应的 polyfill。

关于 WebSockets，以下是一些模拟 WebSocket 行为的库：

| 名称 | 超链接 |
| --- | --- |
| SockJS | [`github.com/sockjs/sockjs-client`](https://github.com/sockjs/sockjs-client) |
| socket.io | [`socket.io/`](http://socket.io/) |
| Kaazing WebSocket Gateway | [`kaazing.com/products/kaazing-websocket-gateway.html`](http://kaazing.com/products/kaazing-websocket-gateway.html) |
| web-socket-js | [`github.com/gimite/web-socket-js/`](http://github.com/gimite/web-socket-js/) |
| Atmosphere | [`jfarcand.wordpress.com/2010/06/15/using-atmospheres-jquery-plug-in-to-build-applicationsupporting-both-websocket-and-comet/`](http://jfarcand.wordpress.com/2010/06/15/using-atmospheres-jquery-plug-in-to-build-applicationsupporting-both-websocket-and-comet/) |
| Graceful WebSocket | [`github.com/ffdead/jquery-graceful-websocket`](https://github.com/ffdead/jquery-graceful-websocket) |
| Portal | [`github.com/flowersinthesand/portal`](https://github.com/flowersinthesand/portal) |
| DataChannel | [`github.com/piranna/DataChannel-polyfill`](https://github.com/piranna/DataChannel-polyfill) |

除了 Kaazing，以上所有库都是开源的，可以免费使用。其中一些库使用 AJAX 方法，而其他一些依赖 Flash 来模拟 WebSocket 的行为。

以下是使用 Graceful WebSocket 库的示例。我们选择了 Graceful WebSocket，因为它简单，轻量级，不使用 Flash，并且暴露了类似于 WebSocket API 的功能。

首先，下载该库以及 jQuery，并将它们包含在您的项目中：

```js
<script src="img/ jquery-1.9.1.min.js"></script>
<script src="img/jquery.gracefulWebSocket.js"></script>
```

像平常一样构建您的文档，并只需一次将对 WebSocket 原生类的任何引用替换为`gracefulWebSocket`即可。

替换这个：

```js
var socket = new WebSocket("ws://localhost:8181");
```

使用这个：

```js
var socket = $.gracefulWebSocket("ws://localhost:8181");
```

就是这么简单！其余的 WebSocket 事件和方法保持不变：

```js
socket.onopen = function (event) {
  // Handle the open event as previously.   
};

socket.onclose = function (event) {
  // Handle the close event as previously.   
};

socket.onmessage = function (event) {
  // Handle the message event as previously.   
};

socket.onerror = function (event) {
  // Handle the error event as previously.   
};
```

发送数据同样简单，可以按以下方式进行：

```js
socket.send("Hello server! I'm a WebSocket polyfill.");
```

在正常模式下，上述代码的前几行简单地包装了 WebSocket 对象并执行原生方法。在回退模式下，该库将协议从 WS 更改为 HTTP，通过进行 HTTP GET 请求来监听消息，并使用 HTTP POST 请求发送消息。

### 注意

特定的 polyfill 解决方案只需要对我们的代码进行轻微修改。其他解决方案可能需要您进行大量修改，或者只能与特定的服务器后端一起使用。在将其用于生产之前，您需要密切关注每个插件的要求、使用方法和文档。

## 浏览器插件

在 HTML5 之前的富互联网应用程序中，浏览器插件是一个非常有用的解决方案。举几个例子，开发人员过去通常利用 Flash（主要是）、Silverlight 或 Java 的功能，在其网站上提供桌面丰富的功能。几年前，基本的 UX 效果、过渡和动画无法使用纯 HTML、CSS 或 JavaScript 来实现。

为了填补这一空白，浏览器插件为开发人员提供了一个可以安装在客户端浏览器中的框架，并允许更丰富的内容。

浏览器插件有一些缺点，使它们日益被淘汰。它们占用资源多，用户需要等待更长时间才能完全加载页面，而且它们大多基于专有技术。因此，越来越多的公司（包括苹果和微软）正在摒弃浏览器插件，转而支持 HTML5。

然而，如果您的用户使用旧版浏览器，他们很可能在旧的台式电脑上安装了一个或多个这样的浏览器插件。一些出色的 WebSocket 实现使用 Flash 来实现双向通信，之前提到的一些 polyfill 也是如此。

**websocket-as**，可在[`github.com/y8/websocket-as`](https://github.com/y8/websocket-as)找到，是一个流行的实用程序，用 ActionScript 编写，实现了类似 HTML5 方法的 WebSocket API。Microsoft 的 Silverlight 和 WCF 技术也有类似的例子([`www.codeproject.com/Articles/220350/Super-WebSockets-WCF-Silverlight-5`](http://www.codeproject.com/Articles/220350/Super-WebSockets-WCF-Silverlight-5))。

如果您熟悉 Flash 或 Silverlight，那么您可以基于您喜欢的浏览器插件实现一个回退解决方案。否则，您可以坚持使用 JavaScript 实现。

# 摘要

并非所有浏览器都原生支持 WebSocket 协议。因此，您需要为那些无法感知 HTML5 好处的用户提供一些备用解决方案。幸运的是，开源社区为我们提供了各种技术，使用纯 HTTP 或 Flash 内部模拟 WebSockets 的功能。实现 HTML5 和备用方案对于您的 Web 应用程序至关重要，并且与您想要触及的受众范围密切相关。在本章中，我们研究了一些流行的备用技术，并了解了如何处理 WebSocket 应用程序中的常见连接错误。这就是您需要了解的关于 WebSocket 和 HTML 部分的内容。在最后一章中，我们将从本地移动体验的角度来研究 WebSocket 协议。
