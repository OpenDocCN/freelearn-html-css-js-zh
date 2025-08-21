# 第四章：调试

在使用 Node.js 的过程中，不可避免地会遇到一些棘手的错误。因此，让我们预先期望它们并为那一天做好准备。

# 日志

我们可以使用一些方法来调试我们的软件；我们要看的第一种方法是日志记录。记录消息的最简单方法是使用`console`。在大多数先前的示例中，`console`已被用来描述正在发生的事情，而无需查看整个 HTTP 请求和响应，从而使事情变得更加可读和简单。

一个例子是：

```js
var Http = require( 'http' );

Http.createServer( function( request, response ) {
    console.log( 
        'Received request', 
        request.method,
        request.url 
    )

    console.log( 'Returning 200' );

    response.writeHead( 200, { 'Content-Type': 'text/plain' } );
    response.end( 'Hello World\n' );

}).listen( 8000 );

console.log( 'Server running on port 8000' );
```

运行此示例将在控制台上记录请求和响应：

```js
[~/examples/example-6]$ node server.js
Server running on port 8000
Received request GET /
Returning 200
Received request GET /favicon.ico
Returning 200
Received request GET /test
Returning 200
Received request GET /favicon.ico
Returning 200

```

如果我们使用接受中间件的框架，比如 express，我们可以使用一个简单的`npm`包叫做**morgan**；您可以在[`www.npmjs.com/package/morgan`](https://www.npmjs.com/package/morgan)找到该包：

```js
[~/examples/example-7]$ npm install morgan
[~/examples/example-7]$ npm install router

```

我们可以通过使用`require`将其引入我们的代码并将其添加为中间件来使用它：

```js
var Morgan = require( 'morgan' ),
    Router = require( 'router' ),
    Http = require( 'http' );

router = new Router( );

router.use( Morgan( 'tiny' ) ); 

/* Simple server */
Http.createServer( function( request, response ) {
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
}).listen( 8000 );

console.log( 'Server running on port 8000' );

function getInfo ( request, response ) {
    var info = process.versions;

    info = JSON.stringify( info );
    response.writeHead( 200, { 'Content-Type': 'application/json' } );
    response.end( info );
}
router.get( '/info', getInfo );
```

服务器运行时，我们可以在不必为每个处理程序添加日志的情况下查看每个请求和响应：

```js
[~/examples/example-7]$ node server.js
Server running on port 8000
GET /test 404 - - 4.492 ms
GET /favicon.ico 404 - - 2.281 ms
GET /info 200 - - 1.120 ms
GET /info 200 - - 1.120 ms
GET /test 404 - - 0.199 ms
GET /info 200 - - 0.494 ms
GET /test 404 - - 0.162 ms
```

这种类型的日志记录是查看服务器上正在使用的内容以及每个请求花费多长时间的简单方法。在这里，您可以看到第一个请求花费的时间最长，然后它们变得快得多。差异仅为 3 毫秒；如果时间更长，可能会成为一个大问题。

我们可以通过更改我们传递给 morgan 的格式来增加记录的信息，如下所示：

```js
router.use( Morgan( 'combined' ) );
```

通过运行服务器，您将看到更多信息，例如远程用户、请求的日期和时间、返回的内容量以及他们正在使用的客户端。

```js
[~/examples/example-7]$ node server.js 
Server running on port 8000
::1 - - [07/Jun/2015:11:09:03 +0000] "GET /info HTTP/1.1" 200 - "-" "--REMOVED---"

```

时间绝对是一个重要因素，因为在筛选您将获得的大量日志时，它可能会有所帮助。有些错误就像一个定时炸弹，等待在周六晚上 3 点爆炸。如果进程已经死亡并且日志已经消失，所有这些日志对我们来说都毫无意义。还有另一个流行且有用的包叫做`bunyan`，它将许多日志记录方法包装成一个。

Bunyan 带来了可写流的优势，可以将日志写入磁盘上的文件或`stdout`。这使我们能够保存日志以进行事后调试。您可以在[`www.npmjs.com/package/bunyan`](https://www.npmjs.com/package/bunyan)找到有关`bunyan`的更多详细信息。

现在，让我们安装该软件包。我们希望它在本地和全局都安装，以便我们还可以将其用作命令行工具：

```js
 [~/examples/example-8]$ npm install –g bunyan
 [~/examples/example-8]$ npm install bunyan 

```

现在，让我们做一些日志记录：

```js
var Bunyan = require( 'bunyan' ),
    logger;

logger = Bunyan.createLogger( {
    name: 'example-8'
});
logger.info( 'Hello logging' );
```

运行我们的示例：

```js
[~/examples/example-8]$ node index.js
{"name":"example-8","hostname":"macbook.local","pid":2483,"level":30,"msg":"Hello logging","time":"2015-06-07T11:35:13.973Z","v":0}

```

这看起来不太好看，对吧？Bunyan 使用简单的结构化 JSON 字符串保存消息；这使得它易于解析、扩展和阅读。Bunyan 配备了一个 CLI 实用程序，使一切变得美观。

如果我们使用实用程序运行示例，那么我们将看到输出格式很好：

```js
[~/examples/example-8]$ node index.js | bunyan
[2015-06-07T11:38:59.698Z]  INFO: example-8/2494 on macbook.local: Hello logging

```

如果我们添加了更多级别，您将在控制台上看到每个级别的颜色不同，以帮助我们识别它们：

```js
var Bunyan = require( 'bunyan' ),
    logger;
logger = Bunyan.createLogger( {
    name: 'example-8'
});
logger.trace( 'Trace' );
logger.debug( 'Debug' );
logger.info( 'Info' );
logger.warn( 'Warn' );
logger.error( 'Error' );
logger.fatal( 'Fatal' );

logger.fatal( 'We got a fatal, lets exit' );
process.exit( 1 );
```

让我们运行示例：

```js
[~/examples/example-8]$ node index.js | bunyan
[2015-06-07T11:39:55.801Z]  INFO: example-8/2512 on macbook.local: Info
[2015-06-07T11:39:55.811Z]  WARN: example-8/2512 on macbook.local: Warn
[2015-06-07T11:39:55.814Z] ERROR: example-8/2512 on macbook.local: Error
[2015-06-07T11:39:55.814Z] FATAL: example-8/2512 on macbook.local: Fatal
[2015-06-07T11:39:55.814Z] FATAL: example-8/2512 on macbook.local: We got a fatal, lets exit

```

如果注意到，跟踪和调试没有在控制台上输出。这是因为它们用于跟踪程序的流程而不是关键信息，通常非常嘈杂。

我们可以通过在创建记录器时将其作为选项传递来更改我们想要查看的日志级别：

```js
logger = Bunyan.createLogger( {
    name: 'example-8',
    level: Bunyan.TRACE 
});
```

现在，当我们运行示例时：

```js
[~/examples/example-8]$ node index.js | bunyan
[2015-06-07T11:55:40.175Z] TRACE: example-8/2621 on macbook.local: Trace
[2015-06-07T11:55:40.177Z] DEBUG: example-8/2621 on macbook.local: Debug
[2015-06-07T11:55:40.178Z]  INFO: example-8/2621 on macbook.local: Info
[2015-06-07T11:55:40.178Z]  WARN: example-8/2621 on macbook.local: Warn
[2015-06-07T11:55:40.178Z] ERROR: example-8/2621 on macbook.local: Error
[2015-06-07T11:55:40.178Z] FATAL: example-8/2621 on macbook.local: Fatal
[2015-06-07T11:55:40.178Z] FATAL: example-8/2621 on macbook.local: We got a fatal, lets exit

```

通常我们不希望看到低于信息级别的日志，因为任何有用于事后调试的信息都应该使用信息级别或更高级别进行记录。

Bunyan 的 API 非常适用于记录错误和对象的功能。它在其 JSON 输出中保存了正确的结构，可以直接显示：

```js
try {
    ref.go( );
} catch ( error ) {
    logger.error( error );
}
```

让我们运行示例：

```js
[~/examples/example-9]$ node index.js | bunyan
[2015-06-07T12:00:38.700Z] ERROR: example-9/2635 on macbook.local: ref is not defined
 ReferenceError: ref is not defined
 at Object.<anonymous> (~/examples/example-8/index.js:9:2)
 at Module._compile (module.js:460:26)
 at Object.Module._extensions..js (module.js:478:10)
 at Module.load (module.js:355:32)
 at Function.Module._load (module.js:310:12)
 at Function.Module.runMain (module.js:501:10)
 at startup (node.js:129:16)
 at node.js:814:3

```

如果我们查看示例并进行漂亮打印，我们将看到它们将其保存为错误：

```js
[~/examples/example-9]$ npm install -g prettyjson
[~/examples/example-9]$ node index.js | prettyjson
name:     example-9
hostname: macbook.local
pid:      2650
level:    50
err: 
 message: ref is not defined
 name:    ReferenceError
 stack: 
 """
 ReferenceError: ref is not defined
 at Object.<anonymous> (~/examples/example-8/index.js:9:2)
 at Module._compile (module.js:460:26)
 at Object.Module._extensions..js (module.js:478:10)
 at Module.load (module.js:355:32)
 at Function.Module._load (module.js:310:12)
 at Function.Module.runMain (module.js:501:10)
 at startup (node.js:129:16)
 at node.js:814:3
 """
msg:      ref is not defined
time:     2015-06-07T12:02:33.875Z
v:        0

```

这很有用，因为如果您只记录错误，如果您使用了`JSON.stringify`，则会得到一个空对象，或者如果您使用了`toString`，则只会得到消息：

```js
try {
    ref.go( );
} catch ( error ) {
    console.log( JSON.stringify( error ) );
    console.log( error );
    console.log( {
        message: error.message
        name: error.name
        stack: error.stack
    });
}
```

让我们运行示例：

```js
[~/examples/example-10]$ node index.js
{}
[ReferenceError: ref is not defined]
{ message: 'ref is not defined',
 name: 'ReferenceError',
 stack: '--REMOVED--' }

```

使用`logger.error( error )`比`logger.error( { message: error.message /*, ... */ } );`更简单和更清晰。

如前所述，`bunyan`使用流的概念，这意味着我们可以写入文件、`stdout`或任何其他我们希望扩展到的服务。

要写入文件，我们只需要将其添加到设置时传递给`bunyan`的选项中：

```js
var Bunyan = require( 'bunyan' ),
    logger;

logger = Bunyan.createLogger( {
    name: 'example-11',
    streams: [
        {
            level: Bunyan.INFO,
            path: './log.log'   
        }
    ]
});
logger.info( process.versions );
logger.info( 'Application started' );
```

通过运行示例，您将看不到任何日志输出到控制台，而是会写入文件：

```js
 [~/examples/example-11]$ node index.js

```

如果您列出目录中的内容，您会看到已创建了一个新文件：

```js
[~/examples/example-11]$ ls 
index.js     log.log      node_modules

```

如果您读取文件中的内容，您会看到日志已经被写入：

```js
[~/examples/example-11]$ cat log.log
{"name":"example-11","hostname":"macbook.local","pid":3614,"level":30,"http_parser":"2.3","node":"0.12.2","v8":"3.28.73","uv":"1.4.2-node1","zlib":"1.2.8","modules":"14","openssl":"1.0.1m","msg":"","time":"2015-06-07T12:29:46.606Z","v":0}
{"name":"example-11","hostname":"macbook.local","pid":3614,"level":30,"msg":"Application started","time":"2015-06-07T12:29:46.608Z","v":0}

```

我们可以通过`bunyan`运行它，以便将其打印出来：

```js
[~/examples/example-11]$ cat log.log | bunyan
[~/examples/example-11]$ cat log.log | bunyan
[2015-06-07T12:29:46.606Z]  INFO: example-11/3614 on macbook.local:  (http_parser=2.3, node=0.12.2, v8=3.28.73, uv=1.4.2-node1, zlib=1.2.8, modules=14, openssl=1.0.1m)
[2015-06-07T12:29:46.608Z]  INFO: example-11/3614 on macbook.local: Application started

```

现在我们可以记录到文件中，我们还希望能够在消息显示时看到它们。如果我们只是记录到文件中，我们可以使用：

```js
[~/examples/example-11]$ tail -f log.log | bunyan

```

这将记录到正在写入的文件`stdout`；或者我们可以向`bunyan`添加另一个流：

```js
logger = Bunyan.createLogger( {
    name: 'example-11',
    streams: [
        {
            level: Bunyan.INFO,
            path: './log.log'   
        },
        {
            level: Bunyan.INFO,
            stream: process.stdout
        }
    ]
});
```

运行示例将在控制台上显示日志：

```js
[~/examples/example-11]$ node index.js | bunyan
 [2015-06-07T12:37:19.857Z] INFO: example-11/3695 on macbook.local: (http_parser=2.3, node=0.12.2, v8=3.28.73, uv=1.4.2-node1, zlib=1.2.8, modules=14, openssl=1.0.1m) [2015-06-07T12:37:19.860Z] INFO: example-11/3695 on macbook.local: Application started

```

我们还可以看到日志已经附加到文件中：

```js
[~/examples/example-11]$ cat log.log | bunyan
 [2015-06-07T12:29:46.606Z]  INFO: example-11/3614 on macbook.local:  (http_parser=2.3, node=0.12.2, v8=3.28.73, uv=1.4.2-node1, zlib=1.2.8, modules=14, openssl=1.0.1m)
[2015-06-07T12:29:46.608Z]  INFO: example-11/3614 on macbook.local: Application started
[2015-06-07T12:37:19.857Z]  INFO: example-11/3695 on macbook.local:  (http_parser=2.3, node=0.12.2, v8=3.28.73, uv=1.4.2-node1, zlib=1.2.8, modules=14, openssl=1.0.1m)
[2015-06-07T12:37:19.860Z]  INFO: example-11/3695 on macbook.local: Application started

```

很好，现在我们已经记录下来了，我们应该怎么处理呢？

好吧，知道错误发生的地方是有帮助的，当您周围有很多匿名函数时，情况就会变得非常混乱。如果您注意到覆盖 HTTP 服务器的示例中，大多数函数都是命名的。当涉及到回调时，这对于跟踪错误非常有帮助。

让我们看看这个例子：

```js
try {
    a = function( callback ) {
        return function( ) {
            callback( );
        };
    };
    b = function( callback ) {
        return function( ) {
            callback( );
        }
    };
    c = function( callback ) {
        return function( ) {
            throw new Error( "I'm just messing with you" ); 
        };
    };
    a( b( c( ) ) )( );
} catch ( error ) {
    logger.error( error );
}
```

它可能看起来有点混乱，因为它确实如此。让我们运行以下示例：

```js
[~/examples/example-12]$ node index.js | bunyan
 [2015-06-07T12:51:11.665Z] ERROR: example-12/4158 on macbook.local: I'm just messing with you
 Error: I'm just messing with you
 at /Users/fabian/examples/example-12/index.js:19:10
 at /Users/fabian/examples/example-12/index.js:14:4
 at /Users/fabian/examples/example-12/index.js:9:4
 at Object.<anonymous> (/Users/fabian/examples/example-12/index.js:22:16)
 at Module._compile (module.js:460:26)
 at Object.Module._extensions..js (module.js:478:10)
 at Module.load (module.js:355:32)
 at Function.Module._load (module.js:310:12)
 at Function.Module.runMain (module.js:501:10)
 at startup (node.js:129:16)

```

您可以看到我们的代码中没有函数名称，堆栈跟踪也没有命名，这与前几个函数不同。在 Node.js 中，函数的命名将来自变量名或实际函数名。例如，如果您使用`Cls.prototype.func`，那么名称将是`Cls.func`，但如果您使用函数`func`，那么名称将是`func`。

您可以看到这里有一点好处，但是一旦您开始使用涉及`async`回调的模式，这将变得非常有用：

```js
[~/examples/example-13]$ npm install q

```

让我们在回调中抛出一个错误：

```js
var Q = require( 'q' );

Q( )
.then( function() {
    // Promised returned from another function
    return Q( )
    .then( function( ) {
        throw new Error( 'Hello errors' ); 
    });
})
.fail( function( error ) {
    logger.error( error );
});
```

运行我们的示例给我们：

```js
[~/examples/example-13]$ node index.js | bunyan
 [2015-06-07T13:03:57.047Z] ERROR: example-13/4598 on macbook.local: Hello errors
 Error: Hello errors
 at /Users/fabian/examples/example-13/index.js:12:9
 at _fulfilled (/Users/fabian/examples/example-13/node_modules/q/q.js:834:54)

```

这是开始变得难以阅读的地方；为我们的函数分配简单的名称可以帮助我们找到错误的来源：

```js
return Q( )
    .then( function resultFromOtherFunction( ) {
        throw new Error( 'Hello errors' ); 
    });
```

运行示例：

```js
[~/examples/example-13]$ node index.js | bunyan
 [2015-06-07T13:04:45.598Z] ERROR: example-13/4614 on macbook.local: Hello errors
 Error: Hello errors
 at resultFromOtherFunction (/Users/fabian/examples/example-13/index.js:12:9)
 at _fulfilled (/Users/fabian/examples/example-13/node_modules/q/q.js:834:54)

```

# 错误处理

调试的另一个方面是处理和预期错误。我们可以以三种方式处理我们的错误：

+   一个简单的`try`/`catch`

+   在进程级别捕获它们

+   在域级别捕获错误

如果我们期望发生错误并且我们能够在不知道正在执行的结果的情况下继续，那么`try`/`catch`函数就足够了，或者我们可以处理并返回错误，如下所示：

```js
function parseJSONAndUse( input ) {
    var json = null;
    try {
        json = JSON.parse( input );
    } catch ( error ) {
        return Q.reject( new Error( "Couldn't parse JSON" ) );
    }
    return Q( use( json ) );
}
```

另一种捕获错误的简单方法是向您的进程添加错误处理程序；在这个级别捕获的任何错误通常是致命的，应该视为这样处理。进程的退出应该跟随，您应该使用一个包，比如`forever`或`pm2`：

```js
process.on( 'uncaughtException', function errorProcessHandler( error ) {
    logger.fatal( error );
    logger.fatal( 'Fatal error encountered, exiting now' );
    process.exit( 1 );
});
```

在捕获到未捕获的错误后，您应该始终退出进程。未捕获的事实意味着您的应用程序处于未知状态，任何事情都可能发生。例如，您的 HTTP 路由器可能出现错误，无法将更多请求路由到正确的处理程序。您可以在[`nodejs.org/api/process.html#process_event_uncaughtexception`](https://nodejs.org/api/process.html#process_event_uncaughtexception)上阅读更多相关信息。

在全局级别处理错误的更好方法是使用`domain`。使用域，您几乎可以*沙箱*一组异步代码在一起。

让我们在请求服务器的情境下思考。我们发出请求，从数据库中读取数据，调用外部服务，写回数据库，进行一些日志记录，执行一些业务逻辑，并且我们期望来自代码周围所有外部来源的数据都是完美的。然而，在现实世界中并非总是如此，我们无法处理可能发生的每一个错误；此外，我们也不希望因为一个非常特定的请求出现错误而导致整个服务器崩溃。这就是我们需要域的地方。

让我们看下面的例子：

```js
var Domain = require( 'domain' ),
    domain;

domain = Domain.create( );

domain.on( 'error', function( error ) {
    console.log( 'Domain error', error.message );
});

domain.run( function( ) {
    // Run code inside domain
    console.log( process.domain === domain );
    throw new Error( 'Error happened' ); 
});
```

让我们运行这段代码：

```js
[~/examples/example-14]$ node index.js
true
Domain error Error happened

```

这段代码存在问题；然而，由于我们是同步运行的，我们仍然将进程置于一个破碎的状态。这是因为错误冒泡到了节点本身，然后传递给了活动域。

当我们在异步回调中创建域时，我们可以确保进程可以继续。我们可以通过使用`process.nextTick`来模拟这一点：

```js
process.nextTick( function( ) {
    domain.run( function( ) {
        throw new Error( 'Error happened' );
    });
    console.log( "I won't execute" );
}); 

process.nextTick( function( ) {
    console.log( 'Next tick happend!' );
});

console.log( 'I happened before everything else' );
```

运行示例应该显示正确的日志：

```js
[~/examples/example-15]$ node index.js
I happened before everything else
Domain error Error happened
Next tick happend!

```

# 摘要

在本章中，我们介绍了一些事后调试方法，帮助我们发现错误，包括日志记录、命名惯例和充分的错误处理。

在下一章中，我们将介绍如何配置我们的应用程序。

为 Bentham Chang 准备，Safari ID 为 bentham@gmail.com 用户编号：2843974 © 2015 Safari Books Online，LLC。此下载文件仅供个人使用，并受到服务条款的约束。任何其他使用都需要版权所有者的事先书面同意。未经授权的使用、复制和/或分发严格禁止并违反适用法律。保留所有权利。
