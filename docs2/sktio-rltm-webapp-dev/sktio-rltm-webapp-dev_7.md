# 附录 A. Socket.IO 快速参考

在本附录中，我们将查看 socket.io 提供的 API。目的是快速浏览所有 API，以便我们知道在工作的过程中是否有函数可以帮助我们。socket.io 正在积极开发中，API 本身也可能发生变化。尽管文档化的方法可能不会改变，但 socket.io 中总是会有新的函数和功能被添加。所以，请始终检查 socket.io 网站和 wiki，以确认所需功能的可用性。

# 服务器

如我们所知，socket.io 提供了用于服务器和客户端的库。让我们首先看看为服务器提供的 API。

## 实例化套接字

与其他 node 模块一样，使用`require`导入模块来实例化`socket.io`模块：

```js
var io = require('socket.io');
```

## 启动 Socket.IO

通过使用`listen`方法启动 socket.io 服务器组件，该方法将 socket.io 附加到 node HTTP 服务器：

```js
var sio = io.listen(<server>)
```

在这里，`server`是 node HTTP 服务器的实例。

## 监听事件

使用`on`方法将事件处理程序附加到套接字上。`on`方法接受事件名称和回调/处理函数作为参数：

```js
sio.on(<event>, function(eventData){
  //DO SOMETHING
});
```

在这里，`event`是事件的名称，而`eventData`代表在调用处理程序时传递给处理程序的事件特定数据。

## 发射事件

我们使用`emit`方法来触发一个事件。这个事件将在客户端被处理：

```js
socket.emit(<event>, <event_data>, ack_callback);
```

在这里，`event`是要触发的事件的名称，`event_data`是作为 JSON 对象的事件数据，而`ack_callback`是在客户端成功接收到事件时调用的可选回调函数。

## 发送消息

使用`send`方法向客户端发送消息：

```js
socket.send(<message>, ack_callback);
```

其中`message`是要发送给客户端的消息，而`ack_callback`是在客户端成功接收到消息时调用的可选回调函数。

## 发送 JSON 消息

可以通过在`send`方法之前使用`json`标志来发送 JSON 消息：

```js
socket.json.send(<message>, ack_callback);
```

在这里，`message`是要发送给客户端的消息，而`ack_callback`是在客户端成功接收到消息时调用的可选回调函数。

## 广播消息/事件

可以使用`broadcast`标志将消息或事件广播到所有已连接的套接字：

```js
socket.broadcast.emit(<event>, <event_data>);
```

在这里，`event`是要发射的事件的名称，而`event_data`是与事件一起发送的 JSON 数据。以下代码行显示了如何广播消息：

```js
socket.broadcast.send(<message>);
```

在这里，`message`是要发送给客户端的消息，而`ack_callback`是在客户端成功接收到消息时调用的可选回调函数。

## 发送一个易失性消息

有时发送的消息并不重要，如果未送达，则可以忽略。因此，这些方法不需要排队或尝试重新发送。这是通过`volatile`标志完成的：

```js
socket.volatile.send(<message>);
```

在这里，`message` 是将发送给客户端的消息，`ack_callback` 是在客户端成功接收到消息时调用的可选回调函数。

## 存储套接字数据

我们可以在套接字上调用 `set` 方法来存储一些数据在套接字上。这是一个异步方法调用，它需要一个键、值和一个回调函数：

```js
socket.set(<key>, <value>, function(){
  //DO SOMETHING
});
```

这里，`key` 是此数据的键名，`value` 是要存储的值。

## 获取套接字数据

我们使用 `get` 方法从套接字获取存储的值。这是一个异步方法，它需要一个键和一个回调函数，该回调函数将获取值：

```js
socket.get(<key>, function(value){
  //DO SOMETHING
});
```

在这里，`key` 是要获取的数据的键，`value` 是如果与套接字一起存储的值。如果没有存储值，这将是一个 `null`。

## 限制到命名空间

我们可以通过使用 `of` 方法来复用套接字并将消息/事件限制到命名空间，从而实现限制。此方法返回一个套接字，它可以像任何其他套接字一样使用，但消息将被限制为仅连接到此命名空间的客户端：

```js
var namespace_socket = socket.of(<namespace>);
```

这里，`namespace` 是我们想要限制套接字的命名空间。

## 加入房间

我们使用 `socket` 的 `join` 方法加入一个房间。如果房间不存在，它将创建一个新的房间：

```js
socket.join(<room>);
```

在这里，`room` 是要加入的房间的名称。

## 在房间中广播消息/事件

我们可以通过使用 `broadcast` 中的 `in` 标志向房间中的所有已连接客户端发送消息：

```js
socket.broadcast.in(<room>).send(<message>);
socket.broadcast.in(<room>).emit(<event>, <event_data>);
```

在这里，`room` 是要发送消息的房间，`message` 是要发送的消息，`event` 是要发出的事件，`event_data` 是与事件一起发送的数据。

## 离开房间

使用 `leave` 方法离开房间。如果套接字正在退出，我们不需要显式地这样做。此外，空房间将自动被销毁：

```js
socket.leave(<room>);
```

这里，`room` 是要退出的房间。

## 更改配置

Socket.io 使用 `configure` 方法回调处理器的 `set` 方法进行配置：

```js
io.configure('environment', function () {
  io.set(<property>, <value>);
});
```

在这里，`environment` 是此配置将使用的可选环境，`property` 是要设置的属性，`value` 是属性的值。

# 服务器事件

我们将在本节中讨论一些与服务器相关的事件。

## 连接

当与客户端建立初始连接时，会触发此事件：

```js
io.sockets.on('connection', function(socket) {})
```

在这里，`socket` 用于与客户端的进一步通信。

## 消息

当使用 `socket.send` 发送的消息被接收时，会发出 `message` 事件：

```js
socket.on('message', function(<message>, <ack_callback>) {})
```

在这里，`message` 是发送的消息，`ack_callback` 是一个可选的确认函数。

## 断开连接

当套接字断开连接时，会触发此事件：

```js
socket.on('disconnect', function() {})
```

# 客户端

在本节中，我们将了解客户端 API。

## 连接到套接字

我们通过客户端 `io` 对象上的 `connect` 方法连接到套接字：

```js
var socket = io.connect(<uri>);
```

在这里，`uri` 是要连接的服务器 URI。它可以绝对或相对。如果不是 `/` 或其绝对等效项，它将连接到命名空间。

## 监听事件

我们可以使用`on`方法将事件处理器附加到套接字上：

```js
socket.on(<event>, function(event_data, ack_callback){});
```

在这里，`event`是要监听的事件，`event_data`是事件的数据，`ack_callback`是在成功接收事件时调用的可选回调方法。

## 触发事件

我们使用`emit`方法来触发一个事件。这个事件将在服务器上处理：

```js
socket.on(<event>, <event_data>, ack_callback);
```

在这里，`event`是要触发的事件的名称，`event_data`是事件的 JSON 对象数据，`ack_callback`是在服务器成功接收消息时调用的可选回调函数。

## 发送消息

使用`send`方法向服务器发送消息：

```js
socket.send(<message>, ack_callback);
```

在这里，`message`是要发送到服务器的消息，`ack_callback`是在服务器成功接收消息的可选回调函数。

# 客户端事件

在本节中，我们将了解一些客户端事件。

## connect

当套接字成功连接时，会触发`connect`事件：

```js
socket.on('connect', function () {})
```

## connecting

当套接字尝试与服务器连接时，会触发`connecting`事件：

```js
socket.on('connecting', function () {})
```

## disconnect

当套接字断开连接时，会触发`disconnect`事件：

```js
socket.on('disconnect', function () {})
```

## connect_failed

当 socket.io 由于传输失败或授权失败等原因无法与服务器建立连接时，会触发`connect_failed`事件：

```js
socket.on('connect_failed', function () {})
```

## error

当发生错误且无法由其他事件类型处理时，会触发`error`事件：

```js
socket.on('error', function () {})
```

## message

当使用`socket.send`发送的消息被接收时，会触发`message`事件：

```js
socket.on('message', function (<message>, <ack_callback>) {})
```

在这里，`message`是发送的消息，`ack_callback`是一个可选的确认函数。

## reconnect

当 socket.io 成功重新连接到服务器时，会触发`reconnect`事件：

```js
socket.on('reconnect', function () {})
```

## reconnecting

当套接字尝试与服务器重新连接时，会触发`reconnecting`事件：

```js
socket.on('reconnecting', function () {})
```

## reconnect_failed

当 socket.io 在连接断开后无法重新建立有效连接时，会触发`reconnect_failed`事件：

```js
socket.on('reconnect_failed', function () {})
```
