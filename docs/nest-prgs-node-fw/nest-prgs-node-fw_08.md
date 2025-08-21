# 第八章：Web 套接字

正如您所见，Nest.js 通过`@nestjs/websockets`包提供了一种在应用程序中使用 Web 套接字的方法。此外，在框架内使用`Adapter`允许您实现所需的套接字库。默认情况下，Nest.js 自带适配器，允许您使用`socket.io`，这是一个众所周知的 Web 套接字库。

您可以创建一个完整的 Web 套接字应用程序，还可以在您的 Rest API 中添加一些 Web 套接字功能。在本章中，我们将看到如何使用 Nest.js 提供的装饰器实现在 Rest API 上使用 Web 套接字，以及如何使用特定中间件验证经过身份验证的用户。

Web 套接字的优势在于能够根据您的需求在应用程序中具有一些实时功能。对于本章，您可以查看存储库中的`/src/gateways`文件，还有`/src/shared/adapters`和`/src/middlewares`。

想象一下以下`CommentGatewayModule`，它看起来像这样：

```js
@Module({
    imports: [UserModule, CommentModule],
    providers: [CommentGateway]
})
export class CommentGatewayModule { }

```

导入`UserModule`以便访问`UserService`，这将在以后很有用，以及`CommentModule`。当然，我们将创建`CommentGateway`，它将用作可注入服务。

# WebSocketGateway

要使用 Nest.js Web 套接字实现您的第一个模块，您必须使用`@WebSocketGateway`装饰器。此装饰器可以接受一个对象作为参数，以提供配置如何使用适配器的方法。

参数的实现遵守`GatewayMetadata`接口，允许您提供：

+   `port`，适配器必须使用的端口

+   `namespace`，属于处理程序

+   在访问处理程序之前必须应用的`middlewares`

所有参数都是可选的。

要使用它，您必须创建您的第一个网关类，因此想象一个`UserGateway`：

```js
@WebSocketGateway({
    middlewares: [AuthenticationGatewayMiddleware]
})  
export class UserGateway { /*....*/ }

```

默认情况下，没有任何参数，套接字将使用与您的 express 服务器相同的端口（通常为`3000`）。正如您所见，在前面的示例中，我们使用了`@WebSocketGateway`，它使用默认端口`3000`，没有命名空间，并且有一个稍后将看到的中间件。

# 网关

在先前看到的装饰器中使用的类中的网关包含您需要提供事件结果的所有处理程序。

Nest.js 带有一个装饰器，允许您访问服务器实例`@WebSocketServer`。您必须在类的属性上使用它。

```js
export class CommentGateway {  
    @WebSocketServer() server; 

    /* ... */
}

```

此外，在整个网关中，您可以访问可注入服务。因此，为了访问评论数据，注入由`CommentModule`导出的`CommentService`，该服务已被注入到此模块中。

```js
export class CommentGateway {
    /* ... */

    constructor(private readonly commentService: CommentService) { }

    /* ... */
}

```

评论服务允许您为下一个处理程序返回适当的结果。

```js
export class CommentGateway {
    /* ... */

    @SubscribeMessage('indexComment')
    async index(client, data): Promise<WsResponse<any>> {
        if (!data.entryId) throw new WsException('Missing entry id.');

        const comments = await this.commentService.findAll({
            where: {entryId: data.entryId}
        });

        return { event: 'indexComment', data: comments };
    }

    @SubscribeMessage('showComment')
    async show(client, data): Promise<WsResponse<any>> {
        if (!data.entryId) throw new WsException('Missing entry id.');
        if (!data.commentId) throw new WsException('Missing comment id.');

        const comment = await this.commentService.findOne({
            where: {
                id: data.commentId,
                entryId: data.entryId
            }
        });

        return { event: 'showComment', data: comment };
    }
}

```

现在我们有两个处理程序，`indexComment`和`showComment`。要使用`indexComment`处理程序，我们期望有一个`entryId`以提供适当的评论，而对于`showComment`，我们期望有一个`entryId`，当然还有一个`commentId`。

正如您所见，要创建事件处理程序，请使用框架提供的`@SubscribeMessage`装饰器。此装饰器将使用传递的字符串作为参数创建`socket.on(event)`，其中事件对应于事件。

# 认证

我们已经设置了我们的`CommentModule`，现在我们想使用令牌对用户进行身份验证（请查看认证章节）。在此示例中，我们使用一个共享服务器用于 REST API 和 Web 套接字事件处理程序。因此，我们将共享身份验证令牌，以查看如何验证用户登录应用程序后收到的令牌。

重要的是要保护 Web 套接字，以避免在未登录应用程序的情况下访问数据。

如前一部分所示，我们使用了名为`AuthenticationGatewayMiddleware`的中间件。此中间件的目的是从 Web 套接字`query`中获取令牌，该令牌带有`auth_token`属性。

如果未提供令牌，中间件将返回`WsException`，否则我们将使用`jsonwebtoken`库（请查看身份验证章节）来验证令牌。

让我们设置中间件：

```js
@Injectable()
export class AuthenticationGatewayMiddleware implements GatewayMiddleware {
    constructor(private readonly userService: UserService) { }
    resolve() {
        return (socket, next) => {
            if (!socket.handshake.query.auth_token) {
                throw new WsException('Missing token.');
            }

            return jwt.verify(socket.handshake.query.auth_token, 'secret', async (err, payload) => {
                if (err) throw new WsException(err);

                const user = await this.userService.findOne({ where: { email: payload.email }});
                socket.handshake.user = user;
                return next();
            });
        }
    }
}

```

用于 Web 套接字的中间件与 REST API 几乎相同。现在实现`GatewayMiddleware`接口与`resolve`函数几乎相同。不同之处在于，您必须返回一个函数，该函数以`socket`和`next`函数作为其参数。套接字包含客户端发送的`query`的`handshake`和所有提供的参数，我们的情况下是`auth_token`。

与经典的身份验证中间件类似（请查看身份验证章节），套接字将尝试使用给定的有效负载查找用户，其中包含电子邮件，然后在握手中注册用户，以便在网关处理程序中访问。这是一种灵活的方式，可以在不再在数据库中查找的情况下已经拥有用户。

# 适配器

正如本章开头所提到的，Nest.js 自带了自己的适配器，使用`socket.io`。但是框架需要灵活，可以与任何第三方库一起使用。为了提供实现另一个库的方法，您可以创建自己的适配器。

适配器必须实现`WebSocketAdapter`接口，以实现以下方法。例如，我们将在新的适配器中使用`ws`作为套接字库。为了使用它，我们将不得不将`app`注入到构造函数中，如下所示：

```js
export class WsAdapter implements WebSocketAdapter {
    constructor(private app: INestApplication) { }

    /* ... */
}

```

通过这样做，我们可以获取`httpServer`以便与`ws`一起使用。之后，我们必须实现`create`方法以创建套接字服务器。

```js
export class WsAdapter implements WebSocketAdapter {
    /* ... */

    create(port: number) {
        return new WebSocket.Server({
            server: this.app.getHttpServer(),
            verifyClient: ({ origin, secure, req }, next) => { 
                return (new WsAuthenticationGatewayMiddleware(this.app.select(UserModule).
                get(UserService))).resolve()(req, next);
            }
        });
    }   

    /* ... */
}

```

如您所见，我们实现了`verifyClient`属性，该属性接受一个带有`{ origin, secure, req }`和`next`值的方法。我们将使用`req`，即来自客户端的`IncomingMessage`和`next`方法，以便继续该过程。我们使用`WsAuthenticationGatewayMiddleware`来验证客户端的令牌，并注入适当的依赖项，选择正确的模块和正确的服务。

在这种情况下，中间件处理身份验证：

```js
@Injectable()
export class WsAuthenticationGatewayMiddleware implements GatewayMiddleware {
    constructor(private userService: UserService) { }
    resolve() {
        return (req, next) => {
            const matches = req.url.match(/token=([^&].*)/);
            req['token'] = matches && matches[1];

            if (!req.token) {
                throw new WsException('Missing token.');
            }

            return jwt.verify(req.token, 'secret', async (err, payload) => {
                if (err) throw new WsException(err);

                const user = await this.userService.findOne({ where: { email: payload.email }});
                req.user = user;
                return next(true);
            });
        }
    }
}

```

在这个中间件中，我们必须手动解析 URL 以获取令牌，并使用`jsonwebtoken`进行验证。之后，我们必须实现`bindClientConnect`方法，将连接事件绑定到 Nest.js 将使用的回调方法。这是一个简单的方法，它接受服务器的参数和回调方法。

```js
export class WsAdapter implements WebSocketAdapter {
    /* ... */

    bindClientConnect(server, callback: (...args: any[]) => void) {
        server.on('connection', callback);
    }

    /* ... */
}

```

要完成我们的新自定义适配器，实现`bindMessageHandlers`以将事件和数据重定向到网关的适当处理程序。该方法将使用`bindMessageHandler`来执行处理程序并将结果返回给`bindMessageHandlers`方法，后者将结果返回给客户端。

```js
export class WsAdapter implements WebSocketAdapter {
    /* ... */

        bindMessageHandlers(client: WebSocket, handlers: MessageMappingProperties[], process: (data) => Observable<any>) {
            Observable.fromEvent(client, 'message')
                .switchMap((buffer) => this.bindMessageHandler(buffer, handlers, process))
                .filter((result) => !!result)
                .subscribe((response) => client.send(JSON.stringify(response)));
        }

        bindMessageHandler(buffer, handlers: MessageMappingProperties[], process: (data) => Observable<any>): Observable<any> {
            const data = JSON.parse(buffer.data);
            const messageHandler = handlers.find((handler) => handler.message === data.type);
            if (!messageHandler) {
                return Observable.empty();
            }
            const { callback } = messageHandler;
            return process(callback(data));
        }

    /* ... */
}

```

现在，我们已经创建了我们的第一个自定义适配器。为了使用它，我们必须在`main.ts`文件中调用`app: INestApplication`提供的`useWebSocketAdapter`，而不是 Nest.js 的`IoAdapter`，如下所示：

```js
app.useWebSocketAdapter(new WsAdapter(app));

```

我们将适配器传递给`app`实例，以便像前面的示例中所示使用它。

# 客户端

在上一节中，我们介绍了如何在服务器端设置 Web 套接字以及如何处理来自客户端的事件。

现在我们将看到如何设置客户端，以便使用 Nest.js 的`IoAdapter`或我们自定义的`WsAdapter`。为了使用`IoAdapter`，我们必须获取`socket.io-client`库并设置我们的第一个 HTML 文件。

该文件将定义一个简单的脚本，将套接字连接到具有已登录用户令牌的服务器。这个令牌将用于确定用户是否连接良好。

检查以下代码：

```js
<script>
    const socket = io('http://localhost:3000',  {
        query: 'auth_token=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.
 eyJlbWFpbCI6InRlc3QzQHRlc3QuZnIiLCJpYXQiOjE1MjQ5NDk3NTgs
 ImV4cCI6MTUyNDk1MzM1OH0.QH_jhOWKockuV-w-vIKMgT_eLJb3dp6a
 ByDbMvEY5xc'
    });
</script>

```

正如您所看到的，我们在套接字连接中传递了一个名为`auth_token`的令牌到查询参数中。我们可以从套接字握手中获取它，然后验证套接字。

发出事件也很容易，参见以下示例：

```js
socket.on('connect', function () {
    socket.emit('showUser', { userId: 4 });
    socket.emit('indexComment', { entryId: 2 });
    socket.emit('showComment', { entryId: 2, commentId: 1 });
});

```

在这个例子中，我们正在等待`connect`事件，以便在连接完成时得知。然后我们发送三个事件：一个是获取用户，然后是一个条目，以及条目的评论。

通过以下`on`事件，我们能够获取服务器作为响应我们之前发出的事件而发送的数据。

```js
socket.on('indexComment', function (data) {
    console.log('indexComment', data);
});
socket.on('showComment', function (data) {
    console.log('showComment', data);
});
socket.on('showUser', function (data) {
    console.log('showUser', data);
});
socket.on('exception', function (data) {
    console.log('exception', data);
});

```

在这里，我们在控制台中显示服务器响应的所有数据，并且我们还实现了一个名为`exception`的事件，以便捕获服务器可能返回的所有异常。

当然，正如我们在身份验证章节中所见，用户无法访问另一个用户的数据。

在我们想要使用自定义适配器的情况下，流程是类似的。我们将使用以下方式打开到服务器的连接：

```js
const ws = new WebSocket("ws://localhost:3000?token=eyJhbGciO
iJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6InRlc3QzQHRlc3QuZnIiL
CJpYXQiOjE1MjUwMDc2NjksImV4cCI6MTUyNTAxMTI2OX0.GQjWzdKXAFTAtO
kpLjId7tPliIpKy5Ru50evMzf15YE");

```

我们在本地主机上使用与我们的 HTTP 服务器相同的端口打开连接。我们还将令牌作为查询参数传递，以便通过`verifyClient`方法，这是我们在`WsAuthenticationGatewayMiddleware`中看到的。

接下来，我们将等待服务器的返回，以确保连接成功并可用。

```js
ws.onopen = function() {
    console.log('open');
    ws.send(JSON.stringify({ type: 'showComment', entryId: 2, commentId: 1 }));
};

```

当连接可用时，使用`send`方法发送我们想要处理的事件类型，这里是使用`showComment`，并传递适当的参数，就像我们在使用 socket.io 时所做的一样。

我们将使用`onmessage`来获取服务器为我们之前发送的事件返回的数据。当`WebSocket`接收到事件时，将发送一个`message`事件给我们可以使用以下示例捕获的管理器。

```js
ws.onmessage = function(ev) {
    const _data = JSON.parse(ev.data);
    console.log(_data);
};

```

您现在可以根据自己的喜好在客户端应用程序的其余部分中使用这些数据。

# 总结

在本章中，您学会了如何设置服务器端，以便使用：

+   由 Nest.js 的`IoAdapter`提供的`socket.io`库

+   具有自定义适配器的`ws`库

您还需要设置一个网关来处理客户端发送的事件。

您已经学会了如何设置客户端以使用`socket.io-client`或`WebSocket`客户端来连接服务器的套接字。这是在与 HTTP 服务器相同的端口上完成的，并且您学会了如何发送和捕获服务器返回的数据或在出现错误时捕获异常。

最后，您学会了如何设置身份验证中间件，以便检查提供的套接字令牌并确定用户是否经过身份验证，以便能够在`IoAdapter`或自定义适配器的情况下访问处理程序。

下一章将涵盖 Nest.js 的微服务。
