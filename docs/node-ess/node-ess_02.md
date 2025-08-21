# 第二章：简单的 HTTP

现在我们已经了解了基础知识，我们可以继续学习一些更有用的东西。在本章中，我们将学习如何创建一个 HTTP 服务器和路由请求。在使用 Node.js 时，你会经常遇到 HTTP，因为服务器端脚本是 Node.js 的常见用途之一。

Node.js 自带一个内置的 HTTP 服务器；你所需要做的就是要求包含的`http`包并创建一个服务器。你可以在[`nodejs.org/api/http.html`](https://nodejs.org/api/http.html)上阅读更多关于该包的信息。

```js
var Http = require( 'http' );

var server = Http.createServer( );
```

这将创建一个属于你自己的 HTTP 服务器，准备就绪。然而，在这种状态下，它不会监听任何请求。我们可以在任何可用的端口或套接字上开始监听，如下所示：

```js
var Http = require( 'http' );

var server = Http.createServer( );
server.listen( 8080, function( ) {
    console.log( 'Listening on port 8080' ); 
});
```

让我们把前面的代码保存为`server.js`并运行它：

```js
[~/examples/example-4]$ node server.js
Listening on port 8080

```

通过在浏览器中导航到`http://localhost:8080/`，你会看到请求已被接受，但服务器没有响应；这是因为我们还没有处理这些请求，我们只是在监听它们。

当我们创建服务器时，我们可以传递一个回调函数，每次有请求时都会调用它。传递的参数将是：`request`，`response`。

```js
function requestHandler( request, response ) {
}
var server = Http.createServer( requestHandler );
```

现在每次收到请求时，我们都可以做一些事情：

```js
var count = 0;
function requestHandler( request, response ) {
    var message;
    count += 1;
    response.writeHead( 201, {
        'Content-Type': 'text/plain'
    });

    message = 'Visitor count: ' + count;
    console.log( message );
    response.end( message );
}
```

让我们运行脚本并从浏览器请求页面；你应该看到`访客计数：1`返回到浏览器：

```js
[~/examples/example-4]$ node server.js
Listening on port 8080
Visitor count: 1
Visitor count: 2
```

然而，出现了一些奇怪的事情：多生成了一个请求。谁是访客 2？

`http.IncomingMessage`（参数`request`）*暴露*了一些属性，可以用来弄清楚这一点。我们现在最感兴趣的属性是`url`。我们期望只有`/`被请求，所以让我们把这个添加到我们的消息中：

```js
message = 'Visitor count: ' + count + ', path: ' + request.url;
```

现在你可以运行代码，看看发生了什么。你会注意到`/favicon.ico`也被请求了。如果你没有看到这个，那么你一定在想我在说什么，或者你的浏览器最近是否访问过`http://localhost:8080`并且已经缓存了图标。如果是这种情况，你可以手动请求图标，例如从`http://localhost:8080/favicon.ico`：

```js
[~/examples/example-4]$ node server.js
Listening on port 8080
Visitor count: 1, path: /
Visitor count: 2, path: /favicon.ico

```

我们还可以看到，如果我们请求任何其他页面，我们将得到正确的路径，如下所示：

```js
[~/examples/example-4]$ node server.js
Listening on port 8080
Visitor count: 1, path: /
Visitor count: 2, path: /favicon.ico
Visitor count: 3, path: /test
Visitor count: 4, path: /favicon.ico
Visitor count: 5, path: /foo
Visitor count: 6, path: /favicon.ico
Visitor count: 7, path: /bar
Visitor count: 8, path: /favicon.ico
Visitor count: 9, path: /foo/bar/baz/qux/norf
Visitor count: 10, path: /favicon.ico

```

然而，这并不是我们想要的结果，除了少数路由之外，我们希望返回`404: Not Found`。

# 介绍路由

路由对于几乎所有的 Node.js 服务器都是必不可少的。首先，我们将实现我们自己的简单版本，然后再转向更复杂的路由。

我们可以使用`switch`语句来实现我们自己的简单路由器，例如：

```js
function requestHandler( request, response ) {
    var message,
        status = 200;

    count += 1;

    switch( request.url ) {
        case '/count':
            message = count.toString( );
            break;
        case '/hello':
            message = 'World';
            break;
        default: 
            status = 404;
            message = 'Not Found';
            break;
    }

    response.writeHead( 201, {
        'Content-Type': 'text/plain'
    });
    console.log( request.url, status, message );
    response.end( message ); 
}
```

让我们运行以下示例：

```js
[~/examples/example-4]$ node server.js
Listening on port 8080
/foo 404 Not Found
/bar 404 Not Found
/world 404 Not Found
/count 200 4
/hello 200 World
/count 200 6

```

你可以看到每次请求时计数都在增加；然而，它并不是每次都返回。如果我们没有为该路由定义一个特定的情况，我们将返回`404: Not Found`。

对于实现 RESTful 接口的服务，我们希望能够根据 HTTP 方法路由请求。请求对象使用`method`属性来暴露这一点。

将这个添加到日志中，我们可以看到这个：

```js
console.log( request.method, request.url, status, message );
```

运行示例并执行你的请求，你可以使用一个 REST 客户端来调用一个 POST 请求：

```js
[~/examples/example-4]$ node server.js
Listening on port 8080
GET /count 200 1
POST /count 200 2
PUT /count 200 3
DELETE /count 200 4

```

我们可以实现一个路由器来根据方法路由，但是已经有一些包可以为我们做到这一点。现在我们将使用一个叫做`router`的简单包：

```js
[~/examples/example-5]$ npm install router

```

现在，我们可以对我们的请求进行一些更复杂的路由：

让我们创建一个简单的 RESTful 接口。

首先，我们需要创建服务器，如下所示：

```js
/* server.js */
var Http = require( 'http' ),
    Router = require( 'router' ), 
    server,
    router; 

router = new Router( );

server = Http.createServer( function( request, response ) {
    router( request, response, function( error ) {
        if( !error ) {
            response.writeHead( 404 );
        } else {
            //Handle errors
            console.log( error.message, error.stack );
            response.writeHead( 400 );
        }       
        response.end( '\n' );
    });
});

server.listen( 8080, function( ) {
    console.log( 'Listening on port 8080' );
});
```

运行服务器应该显示服务器正在监听。

```js
[~/examples/example-5]$ node server.js
Listening on port 8080

```

我们想要定义一个简单的接口来读取、保存和删除消息。我们可能还想要读取单个消息以及消息列表；这本质上定义了一组 RESTful 端点。

REST 代表**R**epresentational **S**tate **T**ransfer；这是许多 HTTP 编程接口使用的一种非常简单和常见的风格。

我们想要定义的端点是：

| HTTP 方法 | 端点 | 用途 |
| --- | --- | --- |
| `POST` | `/message` | 创建消息 |
| `GET` | `/message/:id` | 读取消息 |
| `DELETE` | `/message/:id` | 删除消息 |
| `GET` | `/message` | 读取多条消息 |

对于每种 HTTP 方法，路由器都有一种用于映射路由的方法。这个接口的形式是：

```js
router.<HTTP method>( <path>, [ ... <handler> ] )
```

我们可以为每个路由定义多个处理程序，但我们稍后会回到这一点。

我们将逐个路由进行实现，并将代码追加到`server.js`的末尾。

我们想要把我们的消息存储在某个地方，在现实世界中我们会把它们存储在数据库中；然而，为了简单起见，我们将使用一个带有简单计数器的数组，如下所示：

```js
var counter = 0,
    messages = { };
```

我们的第一个路由将用于创建消息：

```js
function createMessage( request, response ) {
    var id = counter += 1;
    console.log( 'Create message', id );
    response.writeHead( 201, {
        'Content-Type': 'text/plain'
    });
    response.end( 'Message ' + id );
}
router.post( '/message', createMessage );
```

我们可以通过运行服务器并向`http://localhost:8000/message`发送 POST 请求来确保这个路由工作。

```js
[~/examples/example-5]$ node server.js
Listening on port 8080
Create message 1
Create message 2
Create message 3

```

我们还可以确认计数器正在递增，因为每次我们发出请求时 id 都会增加。我们将这样做来跟踪消息的数量并为每条消息赋予一个*唯一*的 id。

现在这个工作了，我们需要能够读取消息文本，为此我们需要能够读取客户端发送的请求正文。这就是多个处理程序发挥作用的地方。我们可以以两种不同的方式来解决这个问题，如果我们只在一个路由中读取正文，或者如果我们正在执行与路由特定的其他操作，例如授权，我们将在路由中添加一个额外的处理程序，例如：

```js
router.post( '/message', parseBody, createMessage ) 
```

我们可以通过为所有方法和路由添加一个处理程序来完成另一种方式；这将在路由处理程序之前首先执行，这些通常被称为中间件。您可以将处理程序视为一系列函数，其中每个函数都在完成其任务后调用下一个函数。有了这个想法，您应该注意添加处理程序的顺序，无论是中间件还是路由，都将决定操作的顺序。这意味着，如果我们注册一个对所有方法执行的处理程序，我们必须首先执行这个处理程序。

路由器*公开*了一个函数来添加以下处理程序：

```js
router.use( function( request, response, next ) {
    console.log( 'middleware executed' );
    // Null as there were no errors
    // If there was an error then we could call `next( error );`
    next( null );
});
```

您可以将此代码添加到`createMessage`的实现之前：

完成后，运行服务器并进行以下请求：

```js
[~/examples/example-5]$ node server.js
Listening on port 8080
middleware executed
Create message 1

```

中间件在路由处理程序之前执行。

现在我们知道了中间件的工作原理，我们可以按照以下方式使用它们：

```js
[~/examples/example-5]$ npm install body-parser

```

用以下内容替换我们的自定义中间件：

```js
var BodyParser = require( 'body-parser' );
router.use( BodyParser.text( ) );
```

在这个阶段，我们只想将所有请求读取为纯文本。

现在我们可以在`createMessage`中检索消息。

```js
function createMessage( request, response ) {
    var id = counter += 1,
        message = request.body;

    console.log( 'Create message', id, message );
    messages[ id ] = message;
    response.writeHead( 201, {
        'Content-Type': 'text/plain',
        'Location': '/message/' + id 
    });
    response.end( message );
}
```

运行`server.js`并向`http://localhost:8080/message`发送`POST`请求；你会看到类似以下消息：

```js
[~/examples/example-5]$ node server.js
Listening on port 8080
Create message 1 Hello foo
Create message 2 Hello bar

```

如果你注意到，你会发现一个标题返回了消息的新位置和它的 id，如果我们请求`http://localhost:8080/message/1`，应该返回第一条消息的内容。

然而，这个路由有一些不同之处；每次创建消息时都会生成一个密钥。我们不想为每条新消息设置一个新的路由，因为这样效率非常低。相反，我们创建一个与模式匹配的路由，比如`/message/:id`。这是在 Node.js 中定义动态路由的常见方式。

路由的`id`部分称为参数。我们可以在我们的路由中定义任意数量的这些参数，并使用请求引用它们；例如，我们可以有一个类似于`/user/:id/profile/:attribute`的路由。

有了这个想法，我们可以创建我们的`readMessage`处理程序，如下所示：

```js
function readMessage( request, response ) {
    var id = request.params.id,
        message = messages[ id ];
    console.log( 'Read message', id, message );

    response.writeHead( 200, {
        'Content-Type': 'text/plain'
    });
    response.end( message );
}
router.get( '/message/:id', readMessage );
```

现在让我们把前面的代码保存在`server.js`文件中并运行服务器：

```js
[~/examples/example-5]$ node server.js
Listening on port 8080
Create message 1 Hello foo
Read message 1 Hello foo
Create message 2 Hello bar
Read message 2 Hello bar
Read message 1 Hello foo

```

通过向服务器发送一些请求，我们可以看到它正在工作。

删除消息几乎与读取消息相同；但我们不返回任何内容并将原始消息值设置为 null：

```js
function deleteMessage( request, response ) {
    var id = request.params.id;

    console.log( 'Delete message', id );

    messages[ id ] = undefined;

    response.writeHead( 204, { } );

    response.end( '' );
}

router.delete( '/message/:id', deleteMessage )
```

首先运行服务器，然后按照以下方式创建、读取和删除消息：

```js
[~/examples/example-5]$ node server.js
Listening on port 8080
Delete message 1
Create message 1 Hello
Read message 1 Hello
Delete message 1
Read message 1 undefined

```

看起来不错；然而，我们遇到了一个问题。我们不应该在删除消息后再次读取消息；如果我们找不到消息，我们将在读取和删除处理程序中返回`404`。我们可以通过向我们的读取和删除处理程序添加以下代码来实现这一点：

```js
    var id = request.params.id,
        message = messages[ id ];

    if( typeof message !== 'string' ) {
        console.log( 'Message not found', id );

        response.writeHead( 404 );
        response.end( '\n' );
        return;
    } 
```

现在让我们把前面的代码保存在`server.js`文件中并运行服务器：

```js
[~/examples/example-5]$ node server.js
Listening on port 8080
Message not found 1
Create message 1 Hello
Read message 1 Hello

```

最后，我们希望能够阅读所有消息并返回所有消息值的列表：

```js
function readMessages( request, response ) {
    var id,
        message,
        messageList = [ ],
        messageString;

    for( id in messages ) {
        if( !messages.hasOwnProperty( id ) ) {
            continue;
        }
        message = messages[ id ];
        // Handle deleted messages
        if( typeof message !== 'string' ) {
            continue;
        }
        messageList.push( message );
    }

    console.log( 'Read messages', JSON.stringify( 
        messageList, 
        null, 
        '  ' 
    ));

    messageString = messageList.join( '\n' );

    response.writeHead( 200, {
        'Content-Type': 'text/plain'
    });

    response.end( messageString );
}
router.get( '/message', readMessages );
```

现在让我们把前面的代码保存在`server.js`文件中并运行服务器：

```js
[~/examples/example-5]$ node server.js
Listening on port 8080
Create message 1 Hello 1
Create message 2 Hello 2
Create message 3 Hello 3
Create message 4 Hello 4
Create message 5 Hello 5
Read messages [
 "Hello 1",
 "Hello 2",
 "Hello 3",
 "Hello 4",
 "Hello 5"
]

```

太棒了；现在我们有了一个完整的 RESTful 接口来读写消息。但是，我们不希望每个人都能读取我们的消息；它们应该是安全的，我们还想知道谁创建了这些消息，我们将在下一章中介绍这个问题。

# 总结

现在我们拥有了制作一些非常酷的服务所需的一切。我们现在可以从头开始创建一个 HTTP，路由我们的请求，并创建一个 RESTful 接口。

这将帮助您创建完整的 Node.JS 服务。在下一章中，我们将介绍身份验证。

为 Bentham Chang 准备，Safari ID 为 bentham@gmail.com 用户编号：2843974 © 2015 Safari Books Online，LLC。此下载文件仅供个人使用，并受到服务条款的约束。任何其他使用均需版权所有者的事先书面同意。未经授权的使用、复制和/或分发严格禁止并违反适用法律。保留所有权利。
