# 第五章. 实时双向通信

我们一直使用 COMET 技术来实现浏览器和 Web 服务器之间的双向通信。长轮询是实现浏览器和 Web 服务器之间双向通信的最流行技术，因为它在不影响用户体验和无需额外服务器配置的情况下工作，并且适用于所有支持 AJAX 的 Web 浏览器。长轮询可以轻松地集成到任何现有的 HTTP 服务器中。但是，长轮询和其他 COMET 技术的缺点是，由于 HTTP 开销，它们都不适合构建实时应用程序。这意味着每次发起 HTTP 请求时，都会将大量头部和 cookie 数据传输到服务器，这反过来又增加了延迟，因此不适合创建需要实时双向通信的应用程序，如多人游戏、聊天应用、社交网络和实时比分网站。因此，引入了一种名为**WebSocket**的新协议，该协议旨在实现浏览器和 WebSocket 服务器之间实时双向通信。

在本章中，我们将涵盖以下内容：

+   WebSocket 概述

+   WebSocket 与 HTTP 的关系

+   WebSocket 与代理服务器和防火墙的交互

+   使用 Socket.IO 实现 WebSocket

+   深入了解 Socket.IO API

+   与 WebSocket 和 Socket.IO 相关的许多其他重要事项

# 介绍 WebSocket

**WebSocket** 是一种应用层协议，旨在促进浏览器和 WebSocket 服务器之间实时双向（客户端或服务器可以在有消息可用时向对方发送消息）和全双工通信（客户端和服务器可以同时向对方发送消息）。

WebSocket 是一种二进制协议；因此，它比基于文本的 HTTP 协议更快。

由于 COMET 技术造成的开销，WebSocket 因其实时和全双工特性而受到欢迎，并且已经被许多网站使用。由于 COMET 技术造成的开销，它不适合实时双向消息传输，并且使用 COMET 也无法在浏览器和 Web 服务器之间建立全双工通信系统。也就是说，COMET 技术只能实现半双工通信系统（在给定时间内，客户端或服务器只能向对方发送消息）。

WebSocket 旨在促进浏览器和 WebSocket 服务器之间的双向通信，但它可以被任何客户端使用。在本章中，我们将仅关注它在浏览器中的实现方式。

### 注意

什么是 WebSocket API？

Web 浏览器提供了一个 API，用于创建和管理与 WebSocket 服务器的 WebSocket 连接，以及在该连接上发送和接收数据。我们不会使用此 API 来实现 WebSocket；相反，我们将使用 Socket.IO 库。

## WebSocket 和 HTTP 之间的关系

WebSocket 和 HTTP 之间的唯一关系是，Web 浏览器和 WebSocket 服务器之间的 WebSocket 握手是通过 HTTP 实现的。因此，WebSocket 服务器也是一个 HTTP 服务器。一旦握手成功，相同的 TCP 连接就会用于 WebSocket 通信，即通信切换到双向二进制协议，该协议不符合 HTTP 协议。WebSocket 的默认端口号是 80，与 HTTP 相同。

### 注意

为什么 WebSocket 的默认端口号是 80？

将 HTTP 和 WebSocket 集成得如此紧密，并让 WebSocket 共享 HTTP 端口的主要原因是为了防止防火墙阻止非网页内容。

虽然如果你在 Web 浏览器环境之外使用 WebSocket，你可以实现自己的 WebSocket 握手机制，但官方 WebSocket 文档仅说明了 HTTP 握手机制，因为 WebSocket 是设计用来在 Web 浏览器和 WebSocket 服务器之间实现双向通信的。

你可以将 WebSocket 服务器集成到为主 HTML 页面提供服务的 Web 服务器中，或者你可以为 WebSocket 通信使用单独的 Web 服务器。

## 在 WebSocket 连接上发送和接收数据

数据通过 WebSocket 连接作为消息传输，每个消息包含一个或多个帧，这些帧包含你要发送的数据（称为有效载荷）。为了确保消息在到达对方时可以正确重建，每个帧都带有 4-12 字节的有效载荷信息。使用基于帧的消息系统有助于减少非有效载荷数据的传输量，从而显著降低延迟，这使得构建实时组件成为可能。

我们不会深入探讨 WebSocket 握手、数据帧定界以及发送和接收数据的精确数据格式和其他细节，因为这仅在你计划创建自己的 WebSocket 服务器时才是必需的。我们将使用 Socket.IO JavaScript 库在我们的应用程序中实现 WebSocket，该库处理所有 WebSocket 的内部细节，并提供一个易于使用的 API。

## WebSocket 方案

WebSocket 协议规范引入了两种新的 URL 方案，称为 **ws** 和 **wss**。

`ws` 代表一个未加密的连接，而 `wss` 代表一个加密的连接。加密连接使用 TLS 加密消息。

因此，当使用 HTTP 进行 WebSocket 握手请求时，我们需要分别使用 `ws` 或 `wss` 而不是 `http` 或 `https`。

### 注意

为什么使用 `ws` 和 `wss` 而不是 `http` 和 `https`？

你可能想知道引入新方案而不是仅仅使用 `http` 的目的是什么。背后的原因是 WebSocket 也可以在 Web 浏览器环境之外使用，握手可以通过非 HTTP 服务器进行协商。因此，在不使用 HTTP 进行握手时，需要不同的方案。

## WebSocket 与代理服务器、负载均衡器和防火墙的交互

WebSocket 协议本身并不了解代理服务器。当 WebSocket 连接在代理服务器后面建立时，WebSocket 连接可能会失败或正常工作，这取决于代理服务器是否透明或明确，以及我们是否建立了安全或不安全的连接。

如果浏览器配置为使用明确的代理服务器，那么在建立 WebSocket 连接时，它将首先向该代理服务器发出一个 `HTTP CONNECT` 方法。`CONNECT` 方法用于告诉代理服务器连接到另一个主机，并简单地回复内容，而不尝试解析或缓存它。浏览器无论连接是加密的还是未加密的，都会发出 `HTTP CONNECT` 方法。

如果我们使用的是透明代理服务器（即浏览器不知道的代理服务器）且连接加密，那么浏览器不会发出 `HTTP CONNECT` 方法，因为它不知道代理服务器。但是，由于连接加密，代理服务器很可能会让所有加密数据通过，因此不会对 WebSocket 连接造成问题。

如果我们使用的是透明代理服务器且连接未加密，那么浏览器不会发出 `HTTP CONNECT` 方法，因为它不知道代理服务器。但是，由于连接未加密，代理服务器可能会尝试缓存、解析或阻止数据，因此可能会对 WebSocket 连接造成问题。在这种情况下，代理服务器应该升级或显式配置以支持 WebSocket 连接。

WebSocket 协议本身并不了解负载均衡器。如果你使用的是 TCP 负载均衡器，它不太可能对 WebSocket 连接造成任何问题。但是，如果你使用的是 HTTP 负载均衡器，它很可能会引起问题；因此，需要升级或显式配置以处理 WebSocket 连接。

WebSocket 协议本身并不了解防火墙。防火墙不太可能对 WebSocket 连接造成任何问题。

## WebSocket 的同源策略

浏览器以及 WebSocket 实例都可以执行跨域通信，即它们不受任何同源策略的限制。

在进行握手请求的 HTTP 请求时，浏览器会发送一个分配给网页源的 `Origin` 标头。

如果 WebSocket 服务器想要限制通信到特定域名，它可以读取握手 HTTP 请求的 `Origin` HTTP 标头，并根据情况阻止或允许握手。

# Socket.IO 简介

Socket.IO 是一个客户端 JavaScript 库和 Node.js 库的组合，用于在浏览器和 Node.js 后端之间集成双向通信。

Socket.IO 客户端库用于创建 Socket.IO 客户端，而 Socket.IO Node.js 库用于创建 Socket.IO 服务器。Socket.IO 客户端和服务器可以双向通信。Socket.IO 主要使用 WebSocket 来实现双向通信。

使用 Socket.IO 客户端库而不是使用 WebSocket API 的主要原因在于，WebSocket 在撰写本文时是一个相对较新的协议，并且并非所有浏览器都支持该 API。如果 Socket.IO 发现浏览器不支持 WebSocket，那么它会跳转到其他机制之一，例如 Flash sockets、长轮询、多部分流、iframe 或 JSONP 轮询，以实现浏览器和服务器之间的双向通信。因此，我们可以断言 Socket.IO 在每个浏览器上都能保证工作。Socket.IO 后端库提供了创建命名空间和房间、广播消息等 API，这在某些情况下非常有用。因此，Socket.IO 是在浏览器和 Node.js 服务器之间实现双向通信的最佳方式。

## 设置你的项目

在我们开始学习 Socket.IO API 之前，让我们首先设置我们的项目目录和文件。创建一个名为 `SocketIO-Example` 的目录。在该目录内，创建名为 `package.json`、`app.js` 和 `socket.js` 的文件，以及一个名为 `public` 的目录。在 `public` 目录内，创建两个目录，`html` 和 `js`。在 `html` 目录内，创建一个名为 `index.html` 的文件。最后，在 `js` 目录内，下载并放置从 [`cdn.socket.io/socket.io-1.3.7.js`](https://cdn.socket.io/socket.io-1.3.7.js) 的 Socket.IO 库。在撰写本文时，Socket.IO 的最新版本是 1.3.7；因此，我们将使用该版本。

在 `app.js` 文件中，我们将编写用于网页服务器的代码，而在 `socket.js` 文件中，我们将编写用于 Socket.IO 服务器的代码。目前，我们将运行两个不同的服务器，即一个单独的网页服务器用于提供网站，另一个服务器用于双向通信。在下一章中，我们将学习如何将 Socket.IO 服务器与 Express 服务器集成。

在 `package.json` 文件中，放置以下代码：

```js
{
  "name": "SocketIO-Example",
  "dependencies": {
    "express": "4.13.3",
    "socket.io": "1.3.7"
  }
}
```

现在，在 `SocketIO-Example` 目录内运行 `npm install` 命令，以便下载并安装 Express 和 Socket.IO Node.js 库。

现在，在 `index.html` 文件中，放置以下 HTML 代码：

```js
<!doctype html>
<html>
  <head>
    <title>SocketIO-Example</title>
  </head>
  <body>
    <script src="img/socket.io-1.3.7.js"></script>
    <script>
      //place JavaScript code here
    </script>
  </body>
</html>
```

在第二个 `<script>` 标签内，你需要放置 Socket.IO 的客户端代码。

现在，将以下代码放置在 `app.js` 文件中以提供 `index.html` 文件：

```js
var express = require("express");
var app = express();

app.use(express.static(__dirname + "/public"));

app.get("/", function(httpRequest, httpResponse, next){
  httpResponse.sendFile(__dirname + "/public/html/index.html");
})

app.listen(8080);
```

在这里，我们监听端口 `8080`。运行 `app.js` 文件，并访问 `http://localhost:8080/` URL 来加载 `index.html` 页面。

我们已经完成了文件和目录的设置。现在，让我们学习 Socket.IO 的客户端和服务器端 API。

## 深入 Socket.IO API

让我们先看看 Socket.IO API 的概述。之后，我们将逐一介绍高级功能。

让我们先构建一个 Socket.IO 服务器。以下是在`socket.js`文件中创建 Socket.IO 服务器实例并监听新的 Socket.IO 客户端握手请求的代码：

```js
var Server = require("socket.io");
var io = new Server({path: "/websocket"});
io.listen(3000);
```

下面是代码的工作方式：

1.  首先，我们导入 Socket.IO Node.js 库。

1.  然后，我们使用`Server`构造函数创建 Socket.IO 服务器的新实例。

1.  然后，在创建新实例时，我们传递 Socket.IO 客户端将发起握手请求的 HTTP 路径。如果我们不传递路径，则默认为`/socket.io`。

1.  最后，我们在端口`3000`上监听。

我在代码中创建了一个单独的 Socket.IO 服务器，但我们有自由创建多个服务器，它们在不同的端口上监听。

现在，Socket.IO 客户端可以发送一个握手请求，并且 Socket.IO 服务器可以与其客户端建立 Socket.IO 连接。让我们编写一些代码，在 Socket.IO 连接建立后，在 Socket.IO 服务器上执行一些操作。将此代码放置在`socket.js`文件中：

```js
io.on("connection", function(socket){

  socket.send("Hi, from server");

  socket.on("message", function(message){
    console.log(message);
  });

  socket.on("disconnect", function(){ 
    console.log("User Disconnected");
  });

  socket.emit("custom-event", "parameter1", "parameter2");

  socket.on("custom-event", function (parameter1, parameter2) {
    console.log(parameter1, parameter2);
  });
});
```

让我们看看这段代码是如何工作的，以及`send()`、`on()`和`emit()`方法的作用：

+   `io`对象上的`on()`方法用于将事件处理器附加到由 Socket.IO 服务器本身触发的事件。

+   我们首先为`connection`事件附加一个事件处理器。一旦 Socket.IO 连接建立，就会触发`connection`事件。事件处理器有一个单一参数，它是一个表示 Socket.IO 客户端的对象。在这里，我们将该参数命名为`socket`。

+   `socket`对象上的`on()`方法用于将事件处理器附加到由 Socket.IO 客户端触发的、发送到服务器的事件。

+   `socket`对象上的`send()`方法用于向 Socket.IO 客户端发送消息。我们在这里发送一个字符串，但也可以发送`ArrayBuffer`、`Blob`、Node.js `Buffer`的实例，甚至`File`。你还可以发送一个简单的 JavaScript 对象。

+   然后，我们附加了一个事件处理器来处理`message`事件，该事件在 Socket.IO 客户端向 Socket.IO 服务器发送消息时触发。

+   之后，我们附加了一个事件处理器来处理`disconnect`事件，该事件在 Socket.IO 客户端从 Socket.IO 服务器断开连接时触发。

+   `socket`对象上的`emit`方法用于向 Socket.IO 客户端发送自定义事件。它可以接受无限数量的参数。它接受的第一个参数是事件名称，其余参数是触发在 Socket.IO 客户端上的事件处理器的参数。

+   最后，我们使用`socket`对象的`on()`方法注册一个名为`custom-event`的自定义事件的事件处理器。

因此，我们现在已经完成了一个非常简单的 Socket.IO 服务器，它允许 Socket.IO 客户端与其建立连接。它也在监听 `message` 和 `custom-event` 事件。一旦 Socket.IO 客户端连接，它就会立即向 Socket.IO 客户端发送消息并触发自定义事件。

### 注意

由于每个 Socket.IO 客户端在 Socket.IO 服务器上都会获得一个独立的 `socket` 对象，如果我们想让一个 `socket` 对象能够访问另一个 Socket.IO 客户端的 `socket` 对象，那么我们可以在全局数组中保留每个 Socket.IO 客户端的 `socket` 对象引用。如果我们正在创建一个聊天应用，其中 `socket` 对象需要访问另一个 `socket` 对象以向其发送消息，这可能会很有用。

你现在可以使用 `node socket.js` 命令运行 Socket.IO 服务器。

现在，让我们构建 Socket.IO 客户端。以下是在 `index.html` 文件的 `<script>` 标签中创建 Socket.IO 客户端实例并与 Socket.IO 服务器建立连接的代码：

```js
var socket = io("http://localhost:3000", {path: "/websocket"});
```

在这里，我们首先创建一个 Socket.IO 客户端实例，并使用 `io` 构造函数与 Socket.IO 服务器建立连接。第一个参数是 Socket.IO 服务器的基本 URL。第二个参数是一个可选对象，我们向其中传递了握手请求应该进行的 URL 路径。如果我们不传递路径，则默认路径为 `/socket.io`。

在这里，我们创建了一个单独的 Socket.IO 客户端实例，但如果我们需要连接到多个 Socket.IO 服务器，我们也有自由创建多个 Socket.IO 客户端实例。

我们在这里使用 `http` 方案而不是 `ws` 方案，因为 Socket.IO 可以使用除 WebSocket 之外的技术或协议来实现双向通信。如果 Socket.IO 选择使用 WebSocket，那么它将自动将 `http` 替换为 `ws`。

让我们编写一些代码，在建立 Socket.IO 连接后，在 Socket.IO 客户端上执行某些操作。将此代码放置在 `index.html` 文件的 `<script>` 标签中：

```js
socket.on("connect", function () {

  socket.send("Hi, from client");

  socket.on("message", function (msg) {
    console.log(msg)
  });

  socket.on("disconnect", function(){ 
    console.log("I am disconnected");
  });

  socket.on("custom-event", function (parameter1, parameter2) {
    console.log(parameter1, parameter2);
  });

  socket.emit("custom-event", "parameter1", "parameter2");
});
```

让我们了解这段代码的工作原理以及 `send()`、`on()` 和 `emit()` 方法的作用：

+   `socket` 对象的 `on()` 方法用于将事件处理器附加到由 Socket.IO 客户端本身触发的事件。

+   我们首先向 `connect` 事件附加了一个事件处理器。一旦建立了 Socket.IO 连接，就会触发 `connect` 事件。

+   `send()` 方法的 `socket` 对象用于向 Socket.IO 服务器发送消息。我们在这里发送一个字符串，但你也可以发送 `ArrayBuffer`、`Blob` 或甚至 `File` 的实例。你也可以发送一个简单的 JavaScript 对象。

+   然后，我们向 `message` 事件附加了一个事件处理器，该事件在 Socket.IO 服务器向 Socket.IO 客户端发送消息时触发。

+   然后，我们将事件处理器附加到`disconnect`事件，该事件在 Socket.IO 客户端从 Socket.IO 服务器断开连接时触发。一旦 Socket.IO 连接中断，Socket.IO 客户端会自动尝试重新连接。

+   之后，我们使用`socket`对象的`on()`方法为名为`custom-event`的自定义事件注册事件处理器。

+   `socket`对象的`emit`方法用于向 Socket.IO 服务器发送自定义事件。它可以接受无限数量的参数。它接受的第一个参数是事件名称，其余参数是事件处理器的参数，该处理器在 Socket.IO 客户端中被触发。

现在，在您的浏览器中打开 URL `http://localhost:8080/`，您应该看到以下控制台输出：

```js
Hi, from server
parameter1 parameter2

```

并且您将在运行 Socket.IO 服务器的 shell 中看到以下输出：

```js
Hi, from client
parameter1 parameter2

```

### 根据来源限制连接

默认情况下，Socket.IO 服务器允许来自任何来源的 Socket.IO 客户端与其建立 Socket.IO 连接。Socket.IO 提供了一种限制连接到特定来源的方法。

要限制连接到特定来源或一组来源，我们可以使用`Server`实例的`origins`方法。

将此代码放置在`socket.js`文件中，以仅允许运行在`localhost`域和端口号`8080`上的 Socket.IO 客户端连接到 Socket.IO 服务器：

```js
io.origins("localhost:8080");
```

我们不能简单地将任何`origin`传递给`origins`方法。以下是一些有效的`origins`示例：

+   `testsite.com:80`

+   `http://testsite.com:80`

+   `http://*:8080` (*是一个通配符)

+   `*:8080`

+   `testsite.com:* http://someotherdomain.com:8080` (通过空格分隔的多个来源)

+   `testsite.com:*/somepath` (Socket.IO 将忽略/somepath)

+   `*:*`

在之前的列表中，每个来源都与一个端口号相关联，因为提供端口号或*（表示任何端口号）是强制性的。

这里有一些无效来源的示例：

+   `testsite.com`

+   `http://testsite.com`

+   `http://testsite.com/somepath`

这些是无效的，因为它们没有与端口号相关联。

还请注意，如果您将`sub.testsite.com`指定为`origins`值，则`testsite.com`将是一个有效的来源。

### Socket.IO 中的命名空间

Socket.IO 服务器实际上被划分为称为**命名空间**的子服务器。Socket.IO 客户端始终连接到命名空间。每个命名空间都有一个名称，看起来像 HTTP 路径。

在之前的代码中，当我们创建一个 Socket.IO 服务器时，会创建一个默认的命名空间。默认命名空间由`/`路径标识。如果我们创建 Socket.IO 客户端时不提及命名空间，那么它将连接到默认命名空间。因此，`connection`事件是特定于特定命名空间的，也就是说，对于每个命名空间，我们必须注册不同的`connection`事件处理器。

### 小贴士

**命名空间的好处是什么？**

你可能想知道引入命名空间的意义是什么。好吧，命名空间使得编写复杂的代码变得更加容易。让我们通过一个例子来理解这一点。

假设你有一个包含多个实时更新的组件的网页。那么，你可能需要为每个组件创建多个 Socket.IO 服务器，或者使用单个 Socket.IO 服务器，并依赖于消息或自定义事件的数据格式来找到哪些数据属于哪个组件。这两种技术都有缺点，也就是说，创建多个 Socket.IO 服务器会占用多个端口，因此不适合大量组件，而依赖于消息和自定义事件的数据格式会使将前端组件移动到单独的应用程序变得困难，因为新的应用程序将接收到大量不必要的消息和事件，导致双方带宽问题。因此，引入了命名空间，它结合了两种技术的优点，同时避免了它们的缺点。

下面是如何创建自定义命名空间的方法。将此代码放置在`socket.js`文件中：

```js
var nsp = io.of("/custom-namespace");

nsp.on("connection", function(socket){
  socket.send("Hi, from custom-namespace");

  socket.on("message", function(message){
    console.log(message);
  });

  socket.on("disconnect", function(){ 
    console.log("User Disconnected");
  });

  socket.on("custom-event", function (parameter1, parameter2) {
    console.log(parameter1, parameter2);
  });

  socket.emit("custom-event", "parameter1", "parameter2");
});
```

将此代码添加到`socket.js`文件后，我们将有两个命名空间，即我们之前创建的默认命名空间，以及这个名为`/custom-namespace`的命名空间。在这里，你可以看到我们为这个命名空间注册了一个新的连接事件处理器。

现在，让我们创建另一个 Socket.IO 客户端，一个连接到`/custom-namespace`命名空间的客户端。将此代码放置在`index.html`文件的`<script>`标签中：

```js
var socket1 = io("http://localhost:3000/custom-namespace", {path: "/websocket"});

socket1.on("connect", function () {

  socket1.send("Hi, from client");

  socket1.on("message", function (msg) {
    console.log(msg)
  });

  socket1.on("disconnect", function(){ 
    console.log("I am disconnected");
  });

  socket1.on("custom-event", function (parameter1, parameter2) {
    console.log(parameter1, parameter2);
  });

  socket1.emit("custom-event", "parameter1", "parameter2");
});
```

在这里，我们正在创建另一个 Socket.IO 客户端；这个客户端连接到`/custom-namespace`命名空间。

现在，重新运行`socket.js`文件并访问`http://localhost:8080/`。这将是在浏览器控制台中的输出：

```js
Hi, from server
parameter1 parameter2
Hi, from custom-namespace
parameter1 parameter2

```

这将是新的 shell 输出：

```js
Hi, from client
parameter1 parameter2
Hi, from client
parameter1 parameter2

```

### 注意

当我们使用`origins()`方法根据来源限制访问时，它将应用于所有命名空间。

#### 指向所有连接的 Socket.IO 客户端

Socket.IO 服务器 API 还为我们提供了一种向命名空间中的所有人发送消息或自定义事件的方法。

让我们看看如何实现这个例子。将以下代码放置在`socket.js`文件中：

```js
setInterval(function(){
  //sending message and custom-event-2 to all clients of default namespace
  io.emit("custom-event-2");
  io.send("Hello Everyone. What's up!!!");

  //sending message and custom-event-2 to all clients of /custom-namespace namespace
  nsp.emit("custom-event-2");
  nsp.send("Hello Everyone. What's up!!!");
}, 5000)
```

在这里，要向连接到默认命名空间的 Socket.IO 客户端发送消息或自定义事件，我们使用`io`对象。要向连接到自定义命名空间的 Socket.IO 客户端发送，我们使用`of()`方法返回的对象。

在这里，我们只是每`5`秒向两个命名空间中的所有人发送一个消息和自定义事件。

### Socket.IO 中的房间

**房间**简单地表示连接到特定命名空间的一组 Socket.IO 客户端。房间属于特定的命名空间。

命名空间不能有两个同名房间，但不同的命名空间可以有同名房间。不同命名空间中同名房间是完全不同的房间。

每个连接到命名空间的 Socket.IO 客户端都必须属于一个或多个组。默认情况下，当 Socket.IO 客户端连接时，会创建一个新的组，并将客户端添加到该组。因此，每个 Socket.IO 客户端默认属于一个唯一的组。

这是打印 Socket.IO 客户端连接后的唯一组名的代码。将其放置在默认和`/custom-namespace`命名空间的`connection`事件处理器中：

```js
console.log(socket.id);
```

`socket`对象的`id`属性持有唯一的房间名称。

#### 加入和离开房间

要将 Socket.IO 客户端添加到自定义房间，我们需要使用`socket.use()`方法。要移除 Socket.IO 客户端从自定义房间，我们需要使用`socket.leave()`方法。

以下代码将连接到默认和`/custom-namespace`服务器的所有 Socket.IO 客户端添加到名为`my-custom-room`的房间中。将其放置在默认和`/custom-namespace`命名空间的`connection`事件处理器中：

```js
socket.join("my-custom-room");
```

同样，要移除`my-custom-room`中的用户，你可以使用以下代码：

```js
socket.leave("my-custom-room");
```

#### 引用房间中的所有连接 Socket.IO 客户端

Socket.IO 服务器 API 还为我们提供了一种向房间中的所有人发送消息或自定义事件的方法。

让我们看看如何实现这个例子。将以下代码放置在`socket.js`文件中：

```js
setInterval(function(){
  //sending message and custom-event-3 to all clients in my-custom-room room of default namespace
  io.to("my-custom-room").send("Hello to everyone in this group");
  io.to("my-custom-room").emit("custom-event-3");

  //sending message and custom-event-3 to all clients in my-custom-room room of /custom-namespace namespace
  nsp.to("my-custom-room").send("Hello to everyone in this group");
  nsp.to("my-custom-room").emit("custom-event-3");
}, 5000)
```

在这里，要向默认命名空间中`my-custom-room`房间的所有 Socket.IO 客户端发送消息或自定义事件，我们需要使用`io.to().send()`方法。而要向`/custom-namespace`命名空间中`my-custom-room`房间的所有 Socket.IO 客户端发送消息或自定义事件，我们需要使用`nsp.to().send()`方法。

### 向命名空间和房间广播消息和自定义事件

广播是 Socket.IO 服务器 API 的一个功能，允许`socket`对象向命名空间或房间中的所有人发送消息或自定义事件，但不能发送给自己。

#### 向命名空间广播

要向命名空间中的所有 Socket.IO 客户端广播消息，我们需要使用`socket.broadcast.send()`方法，而要广播自定义事件，我们需要使用`socket.broadcast.emit()`方法。

让我们看看一个例子。将以下代码放置在默认命名空间的`connection`事件处理器中，以便每次新的 Socket.IO 客户端加入时广播一条消息：

```js
socket.broadcast.send("A new user have joined");
```

现在，在两个不同的标签页中打开`http://localhost:8080/`。在第一个标签页的控制台中，你会看到以下输出：

```js
Hi, from server
parameter1 parameter2
Hi, from custom-namespace
parameter1 parameter2
A new user have joined

```

在第二个标签页的控制台中，你会看到以下输出：

```js
Hi, from server
parameter1 parameter2
Hi, from custom-namespace
parameter1 parameter2

```

#### 向房间广播

要向房间中的所有 Socket.IO 客户端广播消息，我们需要使用`socket.broadcast.to().send()`方法，而要广播自定义事件，我们需要使用`socket.broadcast.to.emit()`方法。

将以下代码放置在默认和`/custom-namespace`命名空间的`connection`事件处理器中：

```js
socket.broadcast.to("my-custom-room").send("Hi everyone. I just joined this group");
```

在这里，一旦 Socket.IO 客户端连接，它就会向房间中的其他人发送消息。

### 注意

记住，Socket.IO 客户端不必是房间的成员即可向其客户端广播消息。

### Socket.IO 中的中间件

Socket.IO 服务器中的中间件是一个在 Socket.IO 客户端发起握手请求时执行的回调，在 Socket.IO 服务器回复它之前。中间件允许我们允许或拒绝握手。

Socket.IO 的中间件概念与 Express 类似，但不同之处在于中间件无法访问 HTTP 响应对象；此外，`parameter`签名也不同。因此，Express 中间件不能在 Socket.IO 中使用。

中间件的一个实例附加到特定的命名空间。以下是一个基本示例，演示了如何将中间件实例注册到所有命名空间中。将此代码放置在`Socket.IO`文件中：

```js
io.use(function(socket, next) {  
  //request object
  //socket.request

  //to reject
  //next(new Error("Reason for reject"));

  //to continue
  next();
});
```

在这里，我们可以看到我们需要使用`io.use()`方法将中间件实例注册到所有命名空间中。要将中间件附加到`/custom-namespace`命名空间，我们可以使用`nsp.use()`方法。

### 手动断开连接

你还可以手动断开 Socket.IO 连接。要从客户端断开连接，你需要使用`io`实例的`disconnect()`方法。要从服务器端断开连接，你需要使用`socket.disconnect()`方法。

# 摘要

在本章中，我们学习了 WebSocket 协议的基础知识。我们了解了它与 HTTP 的关系以及它与代理、负载均衡器和防火墙的交互方式。然后，我们深入了解了 Socket.IO 库，该库主要使用 WebSocket 在实时中实现双向全双工通信。你应该能够舒适地实现浏览器和 Node.js 服务器之间的双向通信。

在下一章中，我们将使用 Socket.IO 构建一个真实世界的应用程序。你将学习更多高级的内容，例如将 Socket.IO 服务器与 Express 服务器集成，并在连接到 WebSocket 服务器之前检查身份验证。
