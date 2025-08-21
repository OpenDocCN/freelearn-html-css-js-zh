# 第九章：使用 Socket.io 添加实时功能

在之前的章节中，您学习了如何构建您的 MEAN 应用程序以及如何创建 CRUD 模块。这些章节涵盖了 Web 应用程序的基本功能；然而，越来越多的应用程序需要服务器和浏览器之间的实时通信。在本章中，您将学习如何使用`Socket.io`模块实时连接您的 Express 和 Angular 应用程序。Socket.io 使 Node.js 开发人员能够在现代浏览器中支持使用`WebSockets`进行实时通信，并在旧版浏览器中支持回退协议。在本章中，我们将涵盖以下主题：

+   设置 Socket.io 模块

+   配置 Express 应用程序

+   设置 Socket.io/Passport 会话

+   连接 Socket.io 路由

+   使用 Socket.io 客户端对象

+   构建一个简单的聊天室

# 介绍 WebSockets

现代 Web 应用程序，如 Facebook、Twitter 和 Gmail，正在整合实时功能，使应用程序能够持续向用户呈现最新更新的信息。与传统应用程序不同，在实时应用程序中，浏览器和服务器的常见角色可以颠倒，因为服务器需要更新浏览器的新数据，而不管浏览器请求的状态如何。这意味着与常见的 HTTP 行为不同，服务器不会等待浏览器的请求。相反，它将在可用新数据时立即将新数据发送到浏览器。

这种反向方法通常被称为**Comet**，这个术语是由一位名叫 Alex Russel 的网页开发者在 2006 年创造的（这个术语是对 AJAX 术语的一个双关语；Comet 和 AJAX 都是美国常见的家用清洁剂）。过去，有几种方法可以使用 HTTP 协议实现 Comet 功能。

第一种最简单的方法是**XMLHttpRequest**（**XHR**）轮询。在 XHR 轮询中，浏览器定期向服务器发出请求。然后服务器返回一个空响应，除非它有新数据要发送回来。在新事件发生时，服务器将新事件数据返回给下一个轮询请求。虽然这对大多数浏览器来说效果很好，但这种方法有两个问题。最明显的问题是，使用这种方法会产生大量没有特定原因的请求，因为很多请求都是空的。第二个问题是更新时间取决于请求周期。这意味着新数据只会在下一个请求时推送到浏览器，导致更新客户端状态的延迟。为了解决这些问题，引入了更好的方法：XHR 长轮询。

在 XHR 长轮询中，浏览器向服务器发出 XHR 请求，但除非服务器有新数据，否则不会发送响应。在事件发生时，服务器将以事件数据响应，并且浏览器会发出新的长轮询请求。这个循环可以更好地管理请求，因为每个会话只有一个请求。此外，服务器可以立即使用新信息更新浏览器，而无需等待浏览器的下一个请求。由于其稳定性和可用性，XHR 长轮询已成为实时应用程序的标准方法，并以各种方式实现，包括 Forever iFrame、多部分 XHR 和使用脚本标签的 JSONP 长轮询（用于跨域实时支持），以及常见的长期 XHR。

然而，所有这些方法实际上都是使用 HTTP 和 XHR 协议的黑客方法，这并不是它们本来的用途。随着现代浏览器的快速发展和新 HTML5 规范的广泛采用，出现了一种新的协议，用于实现实时通信：全双工`WebSockets`协议。

在支持`WebSockets`协议的浏览器中，服务器和浏览器之间的初始连接是通过 HTTP 进行的，称为 HTTP 握手。一旦建立了初始连接，浏览器和服务器将在 TCP 套接字上打开一个持续的通信通道。一旦套接字连接建立，它就可以实现浏览器和服务器之间的双向通信。这使双方能够通过单一通信通道发送和检索消息。这也有助于降低服务器负载，减少消息延迟，并统一使用独立连接进行 PUSH 通信。

然而，`WebSockets`仍然存在两个主要问题。首先是浏览器兼容性。`WebSockets`规范相当新，因此旧版浏览器不支持它，尽管大多数现代浏览器现在实现了该协议，但仍有大量用户在使用这些旧版浏览器。第二个问题是 HTTP 代理、防火墙和托管提供商。由于`WebSockets`使用与 HTTP 不同的通信协议，许多中介不支持它，因此阻止任何套接字通信。就像 Web 一直存在的问题一样，开发人员面临着碎片化问题，只能通过使用抽象库来解决，该库通过根据可用资源切换协议来优化可用性。幸运的是，一个名为 Socket.io 的流行库已经为此目的开发，并且可以免费提供给 Node.js 开发人员社区。

# 介绍 Socket.io

Socket.io 由 JavaScript 开发人员 Guillermo Rauch 于 2010 年创建，旨在抽象化 Node.js 实时应用程序开发。从那时起，它已经发展迅速，并在最新版本中分为两个不同的模块：`engine.io`和`socket.io`之前发布了九个主要版本。

Socket.io 的早期版本因首先尝试建立最先进的连接机制，然后回退到更原始的协议而受到批评。这导致在生产环境中使用 Socket.io 出现严重问题，并对将 Socket.io 作为实时库的采用构成威胁。为了解决这个问题，Socket.io 团队对其进行了重新设计，并将核心功能包装在一个名为 Engine.io 的基础模块中。

Engine.io 的理念是创建一个更稳定的实时模块，首先打开长轮询 XHR 通信，然后尝试升级连接到`WebSockets`通道。新版本的 Socket.io 使用 Engine.io 模块，并为开发人员提供各种功能，如事件、房间和自动连接恢复，否则您将自行实现。在本章的示例中，我们将使用新的 Socket.io 1.0，这是第一个使用 Engine.io 模块的版本。

### 注意

在 1.x 版本之前的旧版 Socket.io 不使用新的 Engine.io 模块，因此在生产环境中不太稳定。

当您包含`socket.io`模块时，它会为您提供两个对象：一个负责服务器功能的 socket 服务器对象，以及一个处理浏览器功能的 socket 客户端对象。我们将从检查服务器对象开始。

## Socket.io 服务器对象

Socket.io 服务器对象是一切的开始。您首先需要引入`socket.io`模块，然后使用它创建一个新的 Socket.io 服务器实例，该实例将与 socket 客户端进行交互。服务器对象支持独立实现和与 Express 框架一起使用的能力。然后，服务器实例公开一组方法，允许您管理 Socket.io 服务器操作。一旦服务器对象被初始化，它还将负责为浏览器提供 socket 客户端 JavaScript 文件。

独立 Socket.io 服务器的简单实现如下所示：

```js
const io = require('socket.io')();
io.on('connection', function(socket){ /* ... */ });
io.listen(3000);
```

这将在`3000`端口上打开一个 Socket.io，并在`http://localhost:3000/socket.io/socket.io.js`上提供套接字客户端文件。与 Express 应用程序一起实现 Socket.io 服务器将会有一些不同，如下面的代码所示：

```js
const app = require('express')();
const server = require('http').Server(app);
const io = require('socket.io')(server);
io.on('connection', (socket) =>  { /* ... */ });
server.listen(3000);
```

这次，你首先使用 Node.js 的`http`模块创建一个服务器并包装 Express 应用程序。然后将服务器对象传递给`socket.io`模块，同时提供 Express 应用程序和 Socket.io 服务器。一旦服务器运行起来，它将可供套接字客户端连接。试图与 Socket.io 服务器建立连接的客户端将通过启动握手过程开始。

### Socket.io 握手

当客户端想要连接到 Socket.io 服务器时，它首先会发送一个握手 HTTP 请求。服务器将分析请求以收集进行中通信所需的必要信息。然后它会查找在服务器上注册的配置中间件，并在触发连接事件之前执行它。当客户端成功连接到服务器时，连接事件监听器将被执行，暴露一个新的套接字实例。

一旦握手过程结束，客户端就连接到服务器了，并且所有与它的通信都通过套接字实例对象处理。例如，处理客户端的断开连接事件将如下所示：

```js
const app = require('express')();
const server = require('http').Server(app);
const io = require('socket.io')(server);
io.on('connection', (socket) => { 
 socket.on('disconnect', () => {
 console.log('user has disconnected');
 });
});
server.listen(3000); 
```

注意`socket.on()`方法如何向断开连接事件添加事件处理程序。尽管断开连接事件是一个预定义事件，但这种方法对自定义事件也适用，后面你会看到。

虽然握手机制是完全自动的，Socket.io 提供了一种使用配置中间件拦截握手过程的方法。

### Socket.io 配置中间件

虽然 Socket.io 配置中间件在以前的版本中存在，但在新版本中，它更简单，允许你在握手实际发生之前操纵套接字通信。要创建一个配置中间件，你需要使用服务器的`use()`方法，这与 Express 应用程序的`use()`方法非常相似：

```js
const app = require('express')();
const server = require('http').Server(app);
const io = require('socket.io')(server);
io.use((socket, next) => {
 /* ... */
 next(null, true);
});
io.on('connection', (socket) => { 
  socket.on('disconnect', () => {
    console.log('user has disconnected');
  });
});
server.listen(3000); 
```

如你所见，`io.use()`方法的回调接受两个参数：`socket`对象和`next`回调。`socket`对象是将用于连接的相同套接字对象，它保存了一些连接属性。一个重要的属性是`socket.request`属性，它代表握手 HTTP 请求。在接下来的部分，你将使用握手请求来将 Passport 会话与 Socket.io 连接结合起来。

`next`参数是一个回调方法，接受两个参数：一个错误对象和一个布尔值。`next`回调告诉 Socket.io 是否继续握手过程，因此如果你向`next`方法传递一个错误对象或 false 值，Socket.io 将不会初始化套接字连接。现在你已经基本了解了握手的工作原理，是时候讨论 Socket.io 客户端对象了。

## Socket.io 客户端对象

Socket.io 客户端对象负责实现浏览器与 Socket.io 服务器的套接字通信。首先要包含由 Socket.io 服务器提供的 Socket.io 客户端 JavaScript 文件。Socket.io JavaScript 文件公开了一个`io()`方法，用于连接到 Socket.io 服务器并创建客户端`socket`对象。套接字客户端的简单实现如下：

```js
<script src="img/socket.io.js"></script>
<script>
  var socket = io();
  socket.on('connect', function() {
      /* ... */
  });
</script>
```

注意 Socket.io 客户端对象的默认 URL。虽然它可以被改变，但通常你可以保持默认的 Socket.io 路径并包含文件。另一件你应该注意的事情是，当没有参数执行`io()`方法时，它将自动尝试连接到默认的基本路径；但是，你也可以传递不同的服务器 URL 作为参数。

正如您所看到的，socket 客户端的实现要简单得多，因此我们可以继续讨论 Socket.io 如何使用事件处理实时通信。

## Socket.io 事件

为了处理客户端和服务器之间的通信，Socket.io 使用了一种模仿`WebSockets`协议的结构，并在服务器和客户端对象之间触发事件消息。有两种类型的事件：系统事件，指示 socket 连接状态，和自定义事件，您将使用它们来实现业务逻辑。

socket 服务器上的系统事件如下：

+   `io.on('connection', ...)`: 当一个新的 socket 连接时触发

+   `socket.on('message', ...)`: 当使用`socket.send()`方法发送消息时触发

+   `socket.on('disconnect', ...)`: 当 socket 断开连接时触发

客户端上的系统事件如下：

+   `socket.io.on('open', ...)`: 当 socket 客户端与服务器建立连接时触发

+   `socket.io.on('connect', ...)`: 当 socket 客户端连接到服务器时触发

+   `socket.io.on('connect_timeout', ...)`: 当 socket 客户端与服务器的连接超时时触发

+   `socket.io.on('connect_error', ...)`: 当 socket 客户端无法连接到服务器时触发

+   `socket.io.on('reconnect_attempt', ...)`: 当 socket 客户端尝试重新连接到服务器时触发

+   `socket.io.on('reconnect', ...)`: 当 socket 客户端重新连接到服务器时触发

+   `socket.io.on('reconnect_error', ...)`: 当 socket 客户端无法重新连接到服务器时触发

+   `socket.io.on('reconnect_failed', ...)`: 当 socket 客户端无法重新连接到服务器时触发

+   `socket.io.on('close', ...)`: 当 socket 客户端关闭与服务器的连接时触发

### 处理事件

虽然系统事件帮助我们进行连接管理，但 Socket.io 的真正魔力在于使用自定义事件。为了做到这一点，Socket.io 在客户端和服务器对象上都公开了两种方法。第一种方法是`on()`方法，它将事件处理程序与事件绑定在一起，第二种方法是`emit()`方法，用于在服务器和客户端对象之间触发事件。

在 socket 服务器中实现`on()`方法非常简单：

```js
const app = require('express')();
const server = require('http').Server(app);
const io = require('socket.io')(server);
io.on('connection', function(socket){ 
 socket.on('customEvent', (customEventData) => {
 /* ... */
 });
});
server.listen(3000); 
```

在上面的代码中，您绑定了一个事件监听器到`customEvent`事件。当 socket 客户端对象发出`customEvent`事件时，事件处理程序被调用。注意事件处理程序如何接受从 socket 客户端对象传递给事件处理程序的`customEventData`参数。

在 socket 客户端中实现`on()`方法也很简单：

```js
<script src="img/socket.io.js"></script>
<script>
  var socket = io();
 socket.on('customEvent', function(customEventData) {
 /* ... */
 });
</script>
```

这次，事件处理程序在 socket 服务器发出`customEvent`事件并将`customEventData`发送到 socket 客户端事件处理程序时被调用。

一旦设置了事件处理程序，您就可以使用`emit()`方法从 socket 服务器向 socket 客户端发送事件，反之亦然。

### 发送事件

在 socket 服务器上，`emit()`方法用于向单个 socket 客户端或一组连接的 socket 客户端发送事件。`emit()`方法可以从连接的`socket`对象中调用，这将向单个 socket 客户端发送事件，如下所示：

```js
io.on('connection', (socket) => { 
  socket.emit('customEvent', customEventData);
});
```

`emit()`方法也可以从`io`对象中调用，这将向所有连接的 socket 客户端发送事件，如下所示：

```js
io.on('connection', (socket) => { 
  io.emit('customEvent', customEventData);
});
```

另一个选项是使用`broadcast`属性将事件发送给除发送方以外的所有连接的 socket 客户端，如下面的代码所示：

```js
io.on('connection', (socket) => { 
  socket.broadcast.emit('customEvent', customEventData);
});
```

在 socket 客户端上，情况要简单得多。由于 socket 客户端只连接到 socket 服务器，`emit()`方法只会将事件发送到 socket 服务器：

```js
const socket = io();
socket.emit('customEvent', customEventData);
```

尽管这些方法允许您在个人和全局事件之间切换，但它们仍然缺乏向一组连接的套接字客户端发送事件的能力。Socket.io 提供了两种选项来将套接字分组：命名空间和房间。

## Socket.io 命名空间

为了更容易地控制套接字管理，Socket.io 允许开发人员根据其目的将套接字连接分割成不同的命名空间。因此，您可以使用相同的服务器来创建不同的连接端点，而不是为不同的连接创建不同的套接字服务器。这意味着套接字通信可以被分成组，然后分别处理。

### Socket.io 服务器命名空间

要创建套接字服务器命名空间，您需要使用套接字服务器的`of()`方法，该方法返回一个套接字命名空间。一旦保留了套接字命名空间，您可以像使用套接字服务器对象一样使用它：

```js
const app = require('express')();
const server = require('http').Server(app);
const io = require('socket.io')(server);

io.of('/someNamespace').on('connection', (socket) => { 
  socket.on('customEvent', (customEventData) => {
    /* ... */
  });
});

io.of('/someOtherNamespace').on('connection', (socket) => { 
  socket.on('customEvent', (customEventData) => {
    /* ... */
  });
});
server.listen(3000);
```

实际上，当您使用`io`对象时，Socket.io 实际上使用一个默认的空命名空间，如下所示：

```js
io.on('connection', (socket) => { 
/* ... */
});
```

前面的代码行实际上等同于这个：

```js
io.of('').on('connection', (socket) => { 
/* ... */
});
```

### Socket.io 客户端命名空间

在套接字客户端上，实现略有不同：

```js
<script src="img/socket.io.js"></script>
<script>
  var someSocket = io('/someNamespace');
  someSocket.on('customEvent', function(customEventData) {
    /* ... */
  });
 var someOtherSocket = io('/someOtherNamespace');
  someOtherSocket.on('customEvent', function(customEventData) {
    /* ... */
  });
</script>
```

如您所见，您可以在同一个应用程序中轻松使用多个命名空间。然而，一旦套接字连接到不同的命名空间，您将无法同时向所有这些命名空间发送事件。这意味着命名空间对于更动态的分组逻辑并不是很好。为此，Socket.io 提供了一个称为 rooms 的不同功能。

## Socket.io 房间

Socket.io 房间允许您以动态方式将连接的套接字分成不同的组。连接的套接字可以加入和离开房间，Socket.io 为您提供了一个清晰的界面来管理房间并向房间中的套接字子集发出事件。房间功能完全由套接字服务器处理，但可以轻松地暴露给套接字客户端。

### 加入和离开房间

使用套接字`join()`方法处理加入房间，而使用`leave()`方法处理离开房间。因此，可以实现一个简单的订阅机制，如下所示：

```js
io.on('connection', (socket) => {
    socket.on('join', (roomData) => {
 socket.join(roomData.roomName);
    })
    socket.on('leave', (roomData) => {
 socket.leave(roomData.roomName);
    })
});
```

请注意，`join()`和`leave()`方法都将房间名称作为第一个参数。

### 向房间发出事件

要向房间中的所有套接字发出事件，您需要使用`in()`方法。因此，向已加入房间的所有套接字客户端发送事件非常简单，并且可以通过以下代码片段的帮助实现：

```js
io.on('connection', (socket) => { 
  io.in('someRoom').emit('customEvent', customEventData);
});
```

另一个选项是使用`broadcast`属性和`to()`方法将事件发送到房间中除发送者之外的所有连接的套接字客户端：

```js
io.on('connection', (socket) => { 
  socket.broadcast.to('someRoom').emit('customEvent', customEventData);
});
```

这基本上涵盖了 Socket.io 的简单而强大的 room 功能。在下一节中，您将学习如何在 MEAN 应用程序中实现 Socket.io，更重要的是，如何使用 Passport 会话来识别 Socket.io 会话中的用户。本章中的示例将直接从前几章中的示例继续，因此请从第八章中复制最终示例，*创建一个 MEAN CRUD 模块*，然后从那里开始。

### 注意

虽然我们已经介绍了大部分 Socket.io 的功能，但您可以通过访问官方项目页面[`socket.io/`](http://socket.io/)来了解更多关于 Socket.io 的信息。

# 安装 Socket.io

在您可以使用`socket.io`模块之前，您需要使用`npm`进行安装。为此，请按照以下更改您的`package.json`文件：

```js
{
  "name": "MEAN",
  "version": "0.0.9",
  "scripts": {
    "tsc": "tsc",
    "tsc:w": "tsc -w",
    "app": "node server",
    "start": "concurrently \"npm run tsc:w\" \"npm run app\" ",
    "postinstall": "typings install"
  },
  "dependencies": {
    "@angular/common": "2.1.1",
    "@angular/compiler": "2.1.1",
    "@angular/core": "2.1.1",
    "@angular/forms": "2.1.1",
    "@angular/http": "2.1.1",
    "@angular/platform-browser": "2.1.1",
    "@angular/platform-browser-dynamic": "2.1.1",
    "@angular/router": "3.1.1",
    "body-parser": "1.15.2",
    "core-js": "2.4.1",
    "compression": "1.6.0",
    "connect-flash": "0.1.1",
    "ejs": "2.5.2",
    "express": "4.14.0",
    "express-session": "1.14.1",
    "method-override": "2.3.6",
    "mongoose": "4.6.5",
    "morgan": "1.7.0",
    "passport": "0.3.2",
    "passport-facebook": "2.1.1",
    "passport-google-oauth": "1.0.0",
    "passport-local": "1.0.0",
    "passport-twitter": "1.0.4",
    "reflect-metadata": "0.1.8",
    "rxjs": "5.0.0-beta.12",
 "socket.io": "1.4.5",
    "systemjs": "0.19.39",
    "zone.js": "0.6.26"
  },
  "devDependencies": {
    "concurrently": "3.1.0",
    "traceur": "0.0.111",
    "typescript": "2.0.3",
    "typings": "1.4.0"
  }
}
```

要安装`socket.io`模块，请转到应用程序的根文件夹，并在命令行工具中发出以下命令：

```js
$ npm install

```

像往常一样，这将在您的`node_modules`文件夹中安装指定版本的 Socket.io。安装过程成功完成后，您需要配置 Express 应用程序以与`socket.io`模块配合工作，并启动套接字服务器。

## 配置 Socket.io 服务器

在安装了`socket.io`模块之后，你需要启动 Socket 服务器以及 Express 应用程序。为此，你需要在你的`config/express.js`文件中进行以下更改：

```js
const path = require('path');
const config = require('./config');
const http = require('http');
const socketio = require('socket.io');
const express = require('express');
const morgan = require('morgan');
const compress = require('compression');
const bodyParser = require('body-parser');
const methodOverride = require('method-override');
const session = require('express-session');
const flash = require('connect-flash');
const passport = require('passport');

module.exports = function() {
  const app = express();
 const server = http.createServer(app);
 const io = socketio.listen(server);

  if (process.env.NODE_ENV === 'development') {
    app.use(morgan('dev'));
  } else if (process.env.NODE_ENV === 'production') {
    app.use(compress());
  }

  app.use(bodyParser.urlencoded({
    extended: true
  }));
  app.use(bodyParser.json());
  app.use(methodOverride());

  app.use(session({
    saveUninitialized: true,
    resave: true,
    secret: config.sessionSecret
  }));

  app.set('views', './app/views');
  app.set('view engine', 'ejs');

  app.use(flash());
  app.use(passport.initialize());
  app.use(passport.session());

  app.use('/', express.static(path.resolve('./public')));
  app.use('/lib', express.static(path.resolve('./node_modules')));

  require('../app/routes/users.server.routes.js')(app);
  require('../app/routes/articles.server.routes.js')(app);
  require('../app/routes/index.server.routes.js')(app);

 return server;
};
```

让我们来看看你对 Express 配置所做的更改。在包含新的依赖项之后，你使用了`http`核心模块来创建一个包装你的 Express `app`对象的`server`对象。然后你使用了`socket.io`模块及其`listen()`方法将 Socket.io 服务器附加到你的`server`对象上。最后，你返回了新的`server`对象而不是 Express 应用程序对象。当服务器启动时，它将运行你的 Socket.io 服务器以及 Express 应用程序。

虽然你已经可以开始使用 Socket.io，但是这个实现还存在一个主要问题。由于 Socket.io 是一个独立的模块，发送到它的请求与 Express 应用程序分离。这意味着 Express 会话信息在 socket 连接中不可用。在应用程序的 socket 层处理 Passport 身份验证时，这将带来严重的障碍。为了解决这个问题，你需要配置一个持久的会话存储，这将允许你在 Express 应用程序和 Socket.io 握手请求之间共享会话信息。

## 配置 Socket.io 会话

要配置你的 Socket.io 会话与 Express 会话一起工作，你必须找到一种方法在 Socket.io 和 Express 之间共享会话信息。由于 Express 会话信息目前存储在内存中，Socket.io 将无法正确访问它。因此，一个更好的解决方案是将会话信息存储在你的 MongoDB 中。幸运的是，有一个名为`connect-mongo`的节点模块，它几乎无缝地允许你将会话信息存储在 MongoDB 实例中。为了检索 Express 会话信息，你需要一种方法来解析已签名的会话 cookie 信息。为此，你还需要安装`cookie-parser`模块，它用于解析 cookie 头并将 HTTP 请求对象填充为与 cookie 相关的属性。

### 安装 connect-mongo 和 cookie-parser 模块

在你可以使用`connect-mongo`和`cookie-parser`模块之前，你需要使用`npm`安装它们。为此，请按照以下方式更改你的`package.json`文件：

```js
{
  "name": "MEAN",
  "version": "0.0.9",
  "scripts": {
    "tsc": "tsc",
    "tsc:w": "tsc -w",
    "app": "node server",
    "start": "concurrently \"npm run tsc:w\" \"npm run app\" ",
    "postinstall": "typings install"
  },
  "dependencies": {
    "@angular/common": "2.1.1",
    "@angular/compiler": "2.1.1",
    "@angular/core": "2.1.1",
    "@angular/forms": "2.1.1",
    "@angular/http": "2.1.1",
    "@angular/platform-browser": "2.1.1",
    "@angular/platform-browser-dynamic": "2.1.1",
    "@angular/router": "3.1.1",
    "body-parser": "1.15.2",
    "core-js": "2.4.1",
    "compression": "1.6.0",
    "connect-flash": "0.1.1",
 "connect-mongo": "1.3.2",
 "cookie-parser": "1.4.3",
    "ejs": "2.5.2",
    "express": "4.14.0",
    "express-session": "1.14.1",
    "method-override": "2.3.6",
    "mongoose": "4.6.5",
    "morgan": "1.7.0",
    "passport": "0.3.2",
    "passport-facebook": "2.1.1",
    "passport-google-oauth": "1.0.0",
    "passport-local": "1.0.0",
    "passport-twitter": "1.0.4",
    "reflect-metadata": "0.1.8",
    "rxjs": "5.0.0-beta.12",
    "socket.io": "1.4.5",
    "systemjs": "0.19.39",
    "zone.js": "0.6.26"
  },
  "devDependencies": {
    "concurrently": "3.1.0",
    "traceur": "0.0.111",
    "typescript": "2.0.3",
    "typings": "1.4.0"
  }
}
```

要安装新的模块，进入你的应用程序根目录，并在命令行工具中输入以下命令：

```js
$ npm install

```

像往常一样，这将在你的`node_modules`文件夹中安装指定版本的`connect-mongo`和`cookie-parser`模块。当安装过程顺利完成后，你的下一步将是配置你的 Express 应用程序使用`connect-mongo`作为会话存储。

### 配置 connect-mongo 模块

要配置你的 Express 应用程序使用`connect-mongo`模块存储会话信息，你需要做一些更改。首先，你需要更改你的`config/express.js`文件，如下所示：

```js
const path = require('path');
const config = require('./config');
const http = require('http');
const socketio = require('socket.io');
const express = require('express');
const morgan = require('morgan');
const compress = require('compression');
const bodyParser = require('body-parser');
const methodOverride = require('method-override');
const session = require('express-session');
const MongoStore = require('connect-mongo')(session);
const flash = require('connect-flash');
const passport = require('passport');

module.exports = function(db) {
  const app = express();
  const server = http.createServer(app);
  const io = socketio.listen(server);

  if (process.env.NODE_ENV === 'development') {
    app.use(morgan('dev'));
  } else if (process.env.NODE_ENV === 'production') {
    app.use(compress());
  }

  app.use(bodyParser.urlencoded({
    extended: true
  }));
  app.use(bodyParser.json());
  app.use(methodOverride());

  const mongoStore = new MongoStore({
 mongooseConnection: db.connection
 });

 app.use(session({
 saveUninitialized: true,
 resave: true,
 secret: config.sessionSecret,
 store: mongoStore
 }));

  app.set('views', './app/views');
  app.set('view engine', 'ejs');

  app.use(flash());
  app.use(passport.initialize());
  app.use(passport.session());

  app.use('/', express.static(path.resolve('./public')));
  app.use('/lib', express.static(path.resolve('./node_modules')));

  require('../app/routes/users.server.routes.js')(app);
  require('../app/routes/articles.server.routes.js')(app);
  require('../app/routes/index.server.routes.js')(app);

  return server;
};
```

在上述代码片段中，你配置了一些东西。首先，你加载了`connect-mongo`模块，然后将 Express 会话模块传递给它。然后，你创建了一个新的`connect-mongo`实例，并将你的 Mongoose 连接对象传递给它。最后，你使用了 Express 会话存储选项，让 Express 会话模块知道在哪里存储会话信息。

正如你所看到的，你的 Express 配置方法需要一个`db`参数。这个参数是 Mongoose 连接对象，它将从`server.js`文件传递给 Express 配置方法，当它需要`express.js`文件时。因此，去你的`server.js`文件并按照以下方式更改它：

```js
process.env.NODE_ENV = process.env.NODE_ENV || 'development';
const configureMongoose = require('./config/mongoose');
const configureExpress = require('./config/express');
const configurePassport = require('./config/passport');

const db = configureMongoose();
const app = configureExpress(db);
const passport = configurePassport();
app.listen(3000);

module.exports = app;

console.log('Server running at http://localhost:3000/'); 
```

一旦 Mongoose 连接创建完成，`server.js`文件将调用`express.js`模块方法并将 Mongoose 数据库属性传递给它。这样，Express 将会将会话信息持久存储在您的 MongoDB 数据库中，以便为 Socket.io 会话提供。接下来，您需要配置您的 Socket.io 握手中间件以使用`connect-mongo`模块并检索 Express 会话信息。

### 配置 Socket.io 会话

要配置 Socket.io 会话，您需要使用 Socket.io 配置中间件并检索您的会话用户。首先，在您的`config`文件夹中创建一个名为`socketio.js`的新文件，以存储所有与 Socket.io 相关的配置。在您的新文件中，添加以下代码行：

```js
const config = require('./config');
const cookieParser = require('cookie-parser');
const passport = require('passport');

module.exports = function(server, io, mongoStore) {
  io.use((socket, next) => {
    cookieParser(config.sessionSecret)(socket.request, {}, (err) => {
      const sessionId = socket.request.signedCookies['connect.sid'];

      mongoStore.get(sessionId, (err, session) => {
        socket.request.session = session;

        passport.initialize()(socket.request, {}, () => {
          passport.session()(socket.request, {}, () => {
            if (socket.request.user) {
              next(null, true);
            } else {
              next(new Error('User is not authenticated'), false);
            }
          })
        });
      });
    });
  });
  io.on('connection', (socket) => {
    /* ... */
  });
};
```

让我们来看一下新的 Socket.io 配置文件。首先，您需要引入必要的依赖项，然后使用`io.use()`配置方法来拦截握手过程。在配置函数中，您使用 Express 的`cookie-parser`模块来解析握手请求的 cookie 并检索 Express 的`sessionId`。然后，您使用`connect-mongo`实例从 MongoDB 存储中检索会话信息。

一旦检索到会话对象，您使用`passport.initialize()`和`passport.session()`中间件来根据会话信息填充会话的`user`对象。如果用户经过身份验证，握手中间件将调用`next()`回调并继续进行套接字初始化；否则，它将使用`next()`回调以一种方式通知 Socket.io 套接字连接无法打开。这意味着只有经过身份验证的用户才能与服务器建立套接字通信，并防止未经授权的连接到您的 Socket.io 服务器。

要完成您的 Socket.io 服务器配置，您需要从`express.js`文件中调用 Socket.io 配置模块。转到您的`config/express.js`文件并进行更改，如下所示：

```js
const path = require('path');
const config = require('./config');
const http = require('http');
const socketio = require('socket.io');
const express = require('express');
const morgan = require('morgan');
const compress = require('compression');
const bodyParser = require('body-parser');
const methodOverride = require('method-override');
const session = require('express-session');
const MongoStore = require('connect-mongo')(session);
const flash = require('connect-flash');
const passport = require('passport');
const configureSocket = require('./socketio');

module.exports = function(db) {
  const app = express();
  const server = http.createServer(app);
  const io = socketio.listen(server);

  if (process.env.NODE_ENV === 'development') {
    app.use(morgan('dev'));
  } else if (process.env.NODE_ENV === 'production') {
    app.use(compress());
  }

  app.use(bodyParser.urlencoded({
    extended: true
  }));
  app.use(bodyParser.json());
  app.use(methodOverride());

  const mongoStore = new MongoStore({
    mongooseConnection: db.connection
  });

  app.use(session({
    saveUninitialized: true,
    resave: true,
    secret: config.sessionSecret,
    store: mongoStore
  }));

  app.set('views', './app/views');
  app.set('view engine', 'ejs');

  app.use(flash());
  app.use(passport.initialize());
  app.use(passport.session());

  app.use('/', express.static(path.resolve('./public')));
  app.use('/lib', express.static(path.resolve('./node_modules')));

  require('../app/routes/users.server.routes.js')(app);
  require('../app/routes/articles.server.routes.js')(app);
  require('../app/routes/index.server.routes.js')(app);

 configureSocket(server, io, mongoStore);

  return server;
};
```

这将执行您的 Socket.io 配置方法，并负责设置 Socket.io 会话。现在您已经配置好了一切，让我们看看如何使用 Socket.io 和 MEAN 轻松构建一个简单的聊天。

# 构建一个 Socket.io 聊天

为了测试您的 Socket.io 实现，构建一个简单的聊天应用程序。您的聊天将由几个服务器事件处理程序构成，但大部分实现将在您的 Angular 应用程序中进行。我们将从设置服务器事件处理程序开始。

## 设置聊天服务器的事件处理程序

在您的 Angular 应用程序中实现聊天客户端之前，您首先需要创建一些服务器事件处理程序。您已经有了一个合适的应用程序结构，因此不会直接在配置文件中实现事件处理程序。相反，最好通过在`app/controllers`文件夹中创建一个名为`chat.server.controller.js`的新文件来实现聊天逻辑。在您的新文件中，粘贴以下代码行：

```js
module.exports = function(io, socket) {
  io.emit('chatMessage', {
    type: 'status',
    text: 'connected',
    created: Date.now(),
    username: socket.request.user.username
  });

  socket.on('chatMessage', (message) => {
    message.type = 'message';
    message.created = Date.now();
    message.username = socket.request.user.username;

    io.emit('chatMessage', message);
  });

  socket.on('disconnect', () => {
    io.emit('chatMessage', {
    type: 'status',
    text: 'disconnected',
    created: Date.now(),
    username: socket.request.user.username
    });
  });
};
```

在这个文件中，您实现了一些内容。首先，您使用`io.emit()`方法通知所有连接的套接字客户端有新连接的用户。这是通过发出`chatMessage`事件并传递带有用户信息和消息文本、时间和类型的聊天消息对象来完成的。由于您在套接字服务器配置中处理了用户身份验证，用户信息可以从`socket.request.user`对象中获取。

接下来，您实现了`chatMessage`事件处理程序，负责处理从套接字客户端发送的消息。事件处理程序将添加消息类型、时间和用户信息，并使用`io.emit()`方法将修改后的消息对象发送给所有连接的套接字客户端。

我们的最后一个事件处理程序将负责处理`disconnect`系统事件。当某个用户从服务器断开连接时，事件处理程序将使用`io.emit()`方法通知所有连接的 socket 客户端关于这个事件。这将允许聊天视图向其他用户呈现断开连接的信息。

您现在已经实现了服务器处理程序，但是如何配置 socket 服务器以包含这些处理程序呢？为此，您需要返回到您的`config/socketio.js`文件并稍微修改它：

```js
const config = require('./config');
const cookieParser = require('cookie-parser');
const passport = require('passport');
const configureChat = require('../app/controllers/chat.server.controller');

module.exports = function(server, io, mongoStore) {
  io.use((socket, next) => {
    cookieParser(config.sessionSecret)(socket.request, {}, (err) => {
      const sessionId = socket.request.signedCookies['connect.sid'];

      mongoStore.get(sessionId, (err, session) => {
        socket.request.session = session;

        passport.initialize()(socket.request, {}, () => {
          passport.session()(socket.request, {}, () => {
            if (socket.request.user) {
              next(null, true);
            } else {
              next(new Error('User is not authenticated'), false);
            }
          })
        });
      });
    });
  });

  io.on('connection', (socket) => {
 configureChat(io, socket);
  });
};
```

注意 socket 服务器`connection`事件是如何用来调用聊天控制器的。这将允许您直接将事件处理程序与连接的 socket 绑定。

恭喜！您已成功完成了服务器的实现！接下来，您将看到实现 Angular 聊天组件是多么容易。

# 创建聊天 Angular 模块

为了完成我们的聊天实现，我们将创建一个新的 Angular 聊天模块。我们的模块将包括我们的组件和模板、路由配置以及包装`socket.io`客户端功能的服务。Socket.io 为我们提供了一个客户端库来处理 socket 通信；然而，最佳实践是使用我们自己的 Angular 服务来混淆它。我们将从配置 Socket.io 客户端库开始。

## 设置 Socket.io 客户端库

为了设置 Socket.io 客户端库，我们需要在我们的`index.ejs`模板中包含库的 JavaScript 文件。为此，转到`app/views/index.ejs`文件并进行以下更改：

```js
<!DOCTYPE html>
<html>
<head>
  <title><%= title %></title>
  <base href="/">
</head>
<body>
  <mean-app>
    <h1>Loading...</h1>
  </mean-app>

  <script type="text/javascript">
    window.user = <%- user || 'null' %>;
  </script>

  <script src="img/socket.io.js"></script>
  <script src="img/shim.min.js"></script>
  <script src="img/zone.js"></script>
  <script src="img/Reflect.js"></script>
  <script src="img/system.js"></script>

  <script src="img/systemjs.config.js"></script>

  <script>
    System.import('app').catch(function(err){ console.error(err); });
  </script>
</body>
</html>
```

如您所见，我们在这里所做的就是在我们的主应用页面中添加脚本标签以包含 Socket.io 的客户端文件。接下来，我们需要创建我们的`Chat`模块。

## 创建聊天模块

完成了客户端 Socket.io 实现的基本声明设置后，我们可以继续进行聊天实现。首先，在`public/app`文件夹内创建一个名为`chat`的文件夹。然后，在这个文件夹内创建一个名为`chat.module.ts`的文件，其中包含以下代码：

```js
import { NgModule }       from '@angular/core';
import { CommonModule }   from '@angular/common';
import { FormsModule }    from '@angular/forms';
import { RouterModule } from '@angular/router';

import { ChatRoutes } from './chat.routes';
import { ChatService } from './chat.service';
import { ChatComponent } from './chat.component';

@NgModule({
  imports: [
    CommonModule,
    FormsModule,
    RouterModule.forChild(ChatRoutes),
  ],
  declarations: [
    ChatComponent,
  ],
  providers: [
    ChatService
  ]
})
export class ChatModule {}
```

正如您可能注意到的，我们的模块导入了一个新的聊天组件和路由配置，并注入了聊天服务。让我们继续创建我们的聊天服务。

## 创建聊天服务

为了混淆我们的组件与 Socket.io 客户端库的通信，我们需要创建一个 Angular 服务。为此，在`public/app/chat`文件夹内创建一个名为`chat.service.ts`的文件。在新文件中，粘贴以下代码：

```js
import 'rxjs/Rx';
import { Observable } from 'rxjs/Observable';
import { Injectable } from '@angular/core';
import { Router } from '@angular/router';

import { AuthenticationService } from '../authentication/authentication.service';

@Injectable()
export class ChatService {
  private socket: any;

  constructor(private _router:Router, private _authenticationService: AuthenticationService) {
    if (this._authenticationService.isLoggedIn()) {
      this.socket = io();
    } else {
      this._router.navigate(['Home']);
    }
  }

  on(eventName, callback) {
    if (this.socket) {
      this.socket.on(eventName, function(data) {
        callback(data);
      });
    }
  };

  emit(eventName, data) {
    if (this.socket) {
      this.socket.emit(eventName, data);
    }
  };

  removeListener(eventName) {
    if (this.socket) {
      this.socket.removeListener(eventName);
    }
  };
}
```

让我们暂停一下来审查我们的新代码。基本结构应该看起来很熟悉，因为它基本上是一个常规的 Angular 服务。在构造函数中注入了 Authentication 和 Router 服务后，您使用`Authentication`服务检查用户是否经过身份验证。如果用户未经过身份验证，则使用`Router`服务将请求重定向回主页。由于 Angular 服务是惰性加载的，Socket 服务只有在请求时才会加载。这将阻止未经身份验证的用户使用 Socket 服务。如果用户经过身份验证，则通过调用 Socket.io 的`io()`方法设置服务`socket`属性。

接下来，您使用兼容的服务方法包装了 socket 的`emit()`、`on()`和`removeListener()`方法。为了保持我们的示例简单，我们称之为`ChatService`服务。然而，正如您可能通过其结构注意到的那样，这个服务很容易成为我们应用程序不同组件中使用的通用 Socket 服务。现在聊天服务已经准备好了，我们所要做的就是实现聊天组件和模板。让我们开始定义聊天组件。

## 创建聊天组件

我们的聊天组件将包含基本的客户端聊天功能。要实现它，转到您的`public/app/chat`文件夹，并创建一个名为`char.component.ts`的文件。在新文件中，粘贴以下代码：

```js
import { Component } from '@angular/core';
import { ChatService } from './chat.service';

@Component({
  selector: 'chat',
  templateUrl: 'app/chat/chat.template.html',
  providers: [ChatService]
})
export class ChatComponent {
  messageText: string;
  messages: Array<any>;

  constructor(private _chatService: ChatService) {}

  ngOnInit() {
    this.messages = new Array();

    this._chatService.on('chatMessage', (msg) => {
 this.messages.push(msg);
 });
  }

  sendMessage() {
    const message = {
      text: this.messageText,
    };

 this._chatService.emit('chatMessage', message);
    this.messageText = ''
  }

  ngOnDestroy() {
 this._chatService.removeListener('chatMessage');
  }
}
```

在我们的组件中，您首先创建了一个消息数组，然后使用`ChatService on()`方法来实现`chatMessage`事件监听器，将检索到的消息添加到此数组中。接下来，您创建了一个`sendMessage()`方法，通过向套接字服务器发出`chatMessage`事件来发送新消息。最后，您使用内置的`ngOnInit`指令来从套接字客户端中删除`chatMessage`事件监听器。当控制器实例被拆除时，`ngOnDestroy`方法将被触发。这很重要，因为除非您将其删除，否则事件处理程序仍将被执行。

## 创建聊天模板

聊天模板将由一个简单的表单和一个聊天消息列表构成。要实现您的聊天模板，请转到您的`public/app/chat`文件夹，并创建一个名为`chat.template.html`的新文件，其中包含以下代码片段：

```js
<div *ngFor="let message of messages" [ngSwitch]="message.type">
    <strong *ngSwitchCase="'status'">
      <span>{{message.created}}</span>
      <span>{{message.username}}</span>
      <span>is</span>
      <span>{{message.text}}</span>
    </strong>
    <span *ngSwitchDefault>
      <span>{{message.created}}</span>
      <span>{{message.username}}:</span>
      <span>{{message.text}}</span>
    </span>
</div>
<form (ngSubmit)="sendMessage()">
    <input type="text" name= "messageText" [(ngModel)]="messageText">
    <input type="submit">
</form>
```

在您的模板中，您使用了`ngFor`指令来渲染消息列表，使用了`ngSwitch`指令来区分状态消息和常规消息。模板以一个简单的表单结束，该表单使用`ngSubmit`指令来调用`sendMessage()`方法。就是这样！您只需要通过将聊天模块添加到我们的应用程序模块中来完成您的实现。

## 添加聊天路由配置

要添加聊天组件路由，请返回到您的`public/app/chat`文件夹，并创建一个名为`chat.routes.ts`的新文件，其中包含以下代码片段：

```js
import { Routes } from '@angular/router';
import { ChatComponent } from './chat.component';

export const ChatRoutes: Routes = [{
  path: 'chat',
  component: ChatComponent
}];
```

正如您所看到的，我们为我们的聊天组件创建了一个简单的路由。我们要做的就是在我们的应用程序模块中包含我们的聊天模块。

## 使用聊天模块

要完成我们的聊天实现，我们需要将我们的模块包含在应用程序模块中。为此，请转到您的`public/app/app.module.ts`，如下所示：

```js
import { NgModule }       from '@angular/core';
import { BrowserModule }  from '@angular/platform-browser';
import { FormsModule }    from '@angular/forms';
import { RouterModule }   from '@angular/router';
import { HttpModule, RequestOptions } from '@angular/http';
import { LocationStrategy, HashLocationStrategy } from '@angular/common';

import { AppComponent } from './app.component';
import { AppRoutes } from './app.routes';

import { HomeModule } from './home/home.module';
import { AuthenticationService } from './authentication/authentication.service';
import { AuthenticationModule } from './authentication/authentication.module';
import { ArticlesModule } from './articles/articles.module';
import { ChatModule } from './chat/chat.module';

@NgModule({
  imports: [
    BrowserModule,
    HttpModule,
    FormsModule,
    AuthenticationModule,
    HomeModule,
    ArticlesModule,
 ChatModule,
    RouterModule.forRoot(AppRoutes),
  ],
  declarations: [
    AppComponent
  ],
  providers: [
    AuthenticationService
  ],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

现在，您需要在主页组件中添加一个链接到我们的聊天组件。为此，请转到您的`public/app/home/home.template.html`文件并进行以下更改：

```js
<div *ngIf="user">
  <h1>Hello {{user.firstName}}</h1>
  <a href="/api/auth/signout">Signout</a>
  <ul>
    <li><a [routerLink]="['/articles']">List Articles</a></li>
    <li><a [routerLink]="['/articles/create']">Create Article</a></li>
 <li><a [routerLink]="['/chat']">Chat</a></li>
  </ul>
</div>

<div *ngIf="!user">
  <a [routerLink]="['/authentication/signup']">Signup</a>
  <a [routerLink]="['/authentication/signin']">Signin</a>
</div>
```

完成这些更改后，您的新聊天组件应该已经准备好使用了！使用命令行工具并导航到 MEAN 应用程序的根文件夹。然后，通过输入以下命令来运行您的应用程序：

```js
$ npm start

```

一旦您的应用程序运行起来，打开两个不同的浏览器并使用两个不同的用户进行注册。然后，导航到`http://localhost:3000/`并点击您的新的聊天链接。尝试在两个客户端之间发送聊天消息，您将能够看到聊天消息是如何实时更新的。您的 MEAN 应用程序现在支持实时通信！

# 总结

在本章中，您学习了`socket.io`模块的工作原理。您了解了 Socket.io 的关键特性，并学习了服务器和客户端如何通信。您配置了 Socket.io 服务器，并学习了如何将其与 Express 应用程序集成。您还使用了 Socket.io 握手配置来集成 Passport 会话。最后，您构建了一个完全功能的聊天示例，并学习了如何使用 Angular 服务包装 Socket.io 客户端。在下一章中，您将学习如何编写和运行测试来覆盖您的应用程序代码。
