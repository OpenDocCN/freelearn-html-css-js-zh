# 第七章：Socket.IO

简单的 HTTP 非常适合不需要实时数据的情况，但是当我们需要在事件发生时得知情况时怎么办。例如，如果我们正在创建一个具有聊天界面或类似功能的网站呢？

这就是 Web sockets 发挥作用的时候。Web sockets 通常被称为 WebSockets，是全双工或双向低延迟通信通道。它们通常被用于消息应用程序和游戏，其中需要在服务器和客户端之间中继消息。有一个非常方便的`npm`模块叫做`socket.io`，它可以为任何 Node.js 应用程序添加 Web sockets。

要安装它，我们只需要运行：

```js
[~/examples/example-27] npm install socket.io

```

Socket.IO 可以非常简单地设置以监听连接。首先，我们希望能够提供一个静态的 html 页面来运行客户端代码：

```js
var Http = require( 'http' ),
    FS = require( 'fs' );

var server = Http.createServer( handler );

server.listen( 8080 );

function handler( request, response ) {
    var index = FS.readFileSync( 'index.html' );
    index = index.toString( );

    response.writeHead(200, {
        'Content-Type': 'text/html',
        'Content-Length': Buffer.byteLength( index )
    });
    response.end( index );
}
```

现在，让我们在同一目录中创建一个名为`index.html`的 HTML 文件：

```js
<html>
    <head>
        <title>WS Example</title>
    </head>
    <body>
        <h2>WS Example</h2>
        <p id="output"></p>
        <!-- SocketIO Client library -->
        <script src="img/socket.io.js"></script>
        <script type="application/javascript">
            /* Our client side code will go here */
        </script>
    </body>
</html>
```

让我们运行我们的示例，并确保我们得到我们的页面，我们应该能够在屏幕上看到**WS Example**。现在，要为我们的应用程序添加 socket 支持，我们只需要要求`socket.io`并指定要使用`IOServer`进行监听的`http`服务器：

```js
var IOServer = require( 'socket.io' );
var io = new IOServer( server );
```

现在，每当有一个新的 socket 连接在`8080`上，我们将在`io`上收到一个`connection`事件：

```js
io.on( 'connection', function( socket ) {
    console.log( 'New Connection' );
});
```

让我们向客户端添加一些代码。Socket.IO 为我们提供了一个客户端库，并通过端点`/socket.io/socket.io.js`公开了这一点。这已经包含在前面的`index.html`文件中。

### 提示

所有客户端代码都包含在`index.html`文件的第二个`script`标签中。

要与服务器建立连接，我们只需要调用`io.connect`并传递位置。这将为我们返回一个 socket，我们可以用它与服务器通信。

我们在这里使用了 Socket.IO 提供的客户端，因为它会检测 WebSockets 是否可用，如果可能的话会使用它们。否则，它将利用其他方法，如轮询，以确保它可以在任何地方工作，而不仅仅是在现代浏览器上：

```js
var socket = io.connect( 'http://localhost:8080' );
```

我们将使用一个`p`元素来将消息记录到屏幕上。我们可以使用这段代码来做到这一点，然后我们只需要调用`logScreen`：

```js
var output = document.getElementById( 'output' );

function logScreen( text ) {
    var date = new Date( ).toISOString( );
    line = date + " " + text + "<br/>";
    output.innerHTML =  line + output.innerHTML
}
```

一旦建立连接，就像在服务器端一样，会发出一个`connection`事件，我们可以使用`on`来监听这个事件：

```js
socket.on( 'connection', function( ){
    logScreen( 'Connection!' );
});
```

现在，一旦我们导航到`http://localhost:8080`，我们就可以运行我们的服务器。您应该能够看到**Connection!**显示出来：

![Socket.IO](img/B04729_07_01.jpg)

要在服务器端接收消息，我们只需要监听`message`事件。现在，我们将简单地将消息回显：

```js
socket.on( 'connection', function( ){
    socket.on( 'message', function ( message ) {
        socket.send( message );
    });
});
```

在客户端，我们只需要调用`send`来发送消息，我们希望在连接事件中执行此操作。双方的`api`非常相似，正如你所看到的：

```js
socket.send( 'Hello' );
```

在客户端，我们还希望监听消息并将其记录到屏幕上：

```js
socket.on( 'message', logScreen );
```

一旦我们重新启动服务器并刷新页面，我们应该能够看到屏幕上出现一个额外的**Hello**消息。

```js
[~/examples/example-27]$ node index.js
Hello
```

这是因为服务器现在可以向客户端发送数据包。这也意味着我们可以随时更新客户端。例如，我们可以每秒向客户端发送一个更新：

```js
socket.on( 'connection', function( ){
    function onTimeout( ) {
        socket.send( 'Update' );
    }
    setInterval( onTimeout, 1000 );
});
```

现在，当我们重新启动服务器时，我们应该能够每秒看到一个更新消息。

您可能已经注意到，您无需刷新网页即可重新打开连接。这是因为`socket.io`会透明地保持我们的连接“活动”，并在需要时重新连接。这消除了使用 sockets 的所有痛苦，因为我们没有这些麻烦。

# 房间

Socket.IO 还有房间的概念，多个客户端可以被分组到不同的房间中。要模拟这一点，您只需要在多个选项卡中导航到`http://localhost:8080`。

一旦客户端连接，我们需要调用`join`方法告诉 socket 要加入哪个房间。如果我们希望做一些特定用户的群聊之类的事情，我们需要在数据库中有一个房间标识符或创建一个。现在我们只是让每个人加入同一个房间：

```js
socket.on( 'connection', function( ){
    console.log( 'New Connection' );
    var room = 'our room';
    socket.join( room, function( error ) {
        if ( error ) return console.log( error );

        console.log( 'Joined room!' );
    });
});
```

每次我们打开一个标签页，我们都应该看到一个消息，告诉我们已经加入了一个房间：

```js
[~/examples/example-27]$ node index.js
New Connection
Joined room!
New Connection
Joined room!
New Connection
Joined room

```

有了这个，我们可以向整个房间广播消息。每次有人加入时让我们这样做。在加入回调中：

```js
socket
    .to( room )
    .emit(
        'message',
        socket.id + ' joined the room!'
    );
```

如果你在浏览器中查看，每次连接时其他客户端都会收到通知，有人加入了：

```js
x3OwYOkOCSsa6Qt5AAAF joined the room!
mlx-Cy1k3szq8W8tAAAE joined the room!
Connection!
Connecting
```

这很棒，我们现在几乎可以直接在浏览器之间通信了！

如果我们想离开一个房间，我们只需要调用`leave`，在调用该函数之前我们将进行广播：

```js
socket
    .to( room )
    .emit(
        'message',
        socket.id + ' is leaving the room'
    );
socket.leave( room );
```

在运行时，您不会看到来自另一个客户端的任何消息，因为您立即离开了：但是，如果您对此进行延迟，您可能会看到另一个客户端进入和离开：

```js
leave = function( ) {
    socket
        .to( room )
        .emit(
            'message',
            socket.id + ' is leaving the room'
        );
    socket.leave( room );
};

setTimeout( leave, 2000 );
```

# 认证

对于认证，我们可以使用与 HTTP 服务器相同的方法，并且我们可以接受 JSON Web Token

在这些示例中，为了简单起见，我们将只有一个单一的 HTTP 路由来登录。我们将签署一个 JWT，稍后我们将通过检查签名来进行身份验证

我们需要安装一些额外的`npm`模块；我们将包括`chance`，以便我们可以生成一些随机数据。

```js
[~/examples/example-27] npm install socketio-jwt jsonwebtoken chance

```

首先，我们需要一个到`login`的路由。我们将修改我们的处理程序以监视`/login`的 URL：

```js
if ( request.url === '/login' ) {
    return generateToken( response )
}
```

我们的新函数`generateToken`将使用`chance`创建一个 JSON Web Token，并且我们还需要一个令牌的密钥：

```js
var JWT = require( 'jsonwebtoken' ),
    Chance = require( 'chance' ).Chance( );

var jwtSecret = 'Our secret';

function generateToken( response ) {

    var payload = {
        email: Chance.email( ),
        name: Chance.first( ) + ' ' + Chance.last( )
    }

    var token = JWT.sign( payload, jwtSecret );

    response.writeHead(200, {
        'Content-Type': 'text/plain',
        'Content-Length': Buffer.byteLength( token )
    })
    response.end(token);
}
```

现在，每当我们请求`http://localhost:8080/login`时，我们将收到一个可以使用的令牌：

```js
[~]$ curl -X GET http://localhost:8080/login
eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJlbW
joiR2VuZSBGbGVtaW5nIiwiaWF0IjoxNDQxMjcyMjM0
e1Y

```

我们可以将其输入到[`jwt.io/`](http://jwt.io/)的调试器中并查看内容：

```js
{
  "email": "jefoconeh@ewojid.io",
  "name": "Gene Fleming",
  "iat": 1441272234
}
```

太棒了，我们有一个令牌和一个为我们生成的随机用户。现在，我们可以用这个来验证我们的用户。Socket.IO 在服务器上有一个方法来做到这一点，我们只需要向其传递一个处理程序类型函数。这就是`socketio-jwt`的作用，我们向其传递我们的密钥，它将确保它是一个真实的令牌，非常简单：

```js
var SocketIOJWT = require( 'socketio-jwt' );

io.use( SocketIOJWT.authorize({
    secret: jwtSecret,
    handshake: true }));
```

现在，当我们尝试从客户端连接到我们的服务器时，它永远不会发出`connect`事件，因为我们的客户端没有经过身份验证。这正是我们想要的。

我们首先想要包装我们的 Socket.IO 代码（稍后我们将调用它）；我们还想给它一个`token`参数：

```js
function socketIO ( token ) {

    var socket = io.connect( 'http://localhost:8080' );

    var output = document.getElementById( 'output' );

    function logScreen( text ) {
        var date = new Date( ).toISOString( );
        line = date + " " + text + "<br/>";
        output.innerHTML =  line + output.innerHTML
    }

    logScreen( 'Connecting' );

    socket.on( 'connect', function( ){
        logScreen( 'Connection!' );
        socket.send( 'Hello' );

    });
    socket.on( 'message', logScreen );

}
```

接下来，我们将创建一个`login`函数，这将请求登录 URL，然后将响应传递给`socketIO`函数，如下所示：

```js
function login( ) {
{
   var request = new XMLHttpRequest();
    request.onreadystatechange = function() {

            if (
            request.readyState !== 4 ||
            request.status !== 200
            ) return

           socketIO( request.responseText );
    }
    request.open( "GET", "/login", true );
    request.send( null );
}
```

然后我们想调用登录函数：

```js
login( );
```

我们可以通过更改`connect`调用以传递查询字符串来将令牌传递给服务器：

```js
var socket = io.connect( 'http://localhost:8080', {
    query: 'token=' + token
});
```

现在，当我们运行服务器并导航到我们的客户端时，我们应该能够连接 - 太棒了！由于我们已经经过身份验证，我们还可以针对每个用户响应个性化消息，在我们的服务器端`connection`事件处理程序内，我们将向客户端发出消息。

我们的 socket 将有一个名为`decoded_token`的新属性；使用这个属性，我们将能够查看我们令牌的内容：

```js
var payload = socket.decoded_token;
var name = payload.name;

socket.emit( 'message', 'Hello ' + name + '!' );
```

一旦我们加入房间，我们可以告诉其他也加入的客户端：

```js
socket
    .to( room )
    .emit(
        'message',
        name + ' joined the room!'
    );
```

# 总结

Socket.IO 为我们的应用程序带来了惊人的功能。我们现在可以立即与其他人通信，无论是个别通信还是在房间中广播。通过识别用户的能力，我们可以记录消息或该用户的历史，准备通过 RESTful API 提供。

我们现在已经准备好构建实时应用程序了！

为 Bentham Chang 准备，Safari ID bentham@gmail.com 用户编号：2843974 © 2015 Safari Books Online，LLC。此下载文件仅供个人使用，并受到服务条款的约束。任何其他使用都需要版权所有者的事先书面同意。未经授权的使用、复制和/或分发严格禁止并违反适用法律。保留所有权利。
