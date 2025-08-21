# 第四章：使用 Socket.IO 和 ExpressJS 进行实时通信

在本章中，我们将涵盖以下配方：

+   理解 NodeJS 事件

+   理解 Socket.IO 事件

+   使用 Socket.IO 命名空间

+   定义并加入 Socket.IO 房间

+   为 Socket.IO 编写中间件

+   将 Socket.IO 与 ExpressJS 集成

+   在 Socket.IO 中使用 ExpressJS 中间件

# 技术要求

您需要一个 IDE，Visual Studio Code，Node.js 和 MongoDB。 您还需要安装 Git，以便使用本书的 Git 存储库。

本章的代码文件可以在 GitHub 上找到：

[`github.com/PacktPublishing/MERN-Quick-Start-Guide/tree/master/Chapter04`](https://github.com/PacktPublishing/MERN-Quick-Start-Guide/tree/master/Chapter04)

查看以下视频以查看代码的实际操作：

[`goo.gl/xfyDBn`](https://goo.gl/xfyDBn)

# 介绍

现代 Web 应用程序通常需要实时通信，其中数据不断从客户端流向服务器，反之亦然，几乎没有延迟。

HTML5 WebSocket 协议是为了满足这一要求而创建的。 WebSocket 使用单个 TCP 连接，即使服务器或客户端不发送任何数据，该连接也会保持打开。 这意味着，在客户端和服务器之间存在连接时，可以随时发送数据，而无需打开到服务器的新连接。

实时通信有多个应用场景，从构建聊天应用程序到多用户游戏，响应时间非常重要。

在本章中，我们将专注于学习如何使用 Socket.IO（[`socket.io`](https://socket.io)）构建实时 Web 应用程序，并理解 Node.js 的事件驱动架构。

Socket.IO 是实现实时通信最常用的库之一。 Socket.IO 在可能的情况下使用 WebSocket，但在特定 Web 浏览器不支持 WebSocket 时会退回到其他方法。 因为您可能希望使您的应用程序可以从任何 Web 浏览器访问，所以必须直接使用 WebSocket 可能看起来不是一个好主意。

# 理解 Node.js 事件

Node.js 具有事件驱动的架构。 Node.js 的大部分核心 API 都是围绕`EventEmitter`构建的。 这是一个允许`侦听器`订阅特定命名事件的 Node.js 模块，稍后可以由**发射器**触发。

您可以通过包含事件 Node.js 模块并创建`EventEmitter`的新实例来轻松定义自己的事件发射器：

```js
const EventEmitter = require('events') 
const emitter = new EventEmitter() 
emitter.on('welcome', () => { 
    console.log('Welcome!') 
}) 
```

然后，您可以使用`emit`方法触发`welcome`事件：

```js
emitter.emit('welcome') 
```

这实际上相当简单。 其中一个优点是您可以订阅多个侦听器到同一个事件，并且当使用`emit`方法时它们将被触发：

```js
emitter.on('welcome', () => { 
    console.log('Welcome') 
}) 
emitter.on('welcome', () => { 
    console.log('There!') 
}) 
emitter.emit('welcome') 
```

`EventEmitter` API 提供了几种有用的方法，可以让您更好地控制处理事件。 请查看官方 Node.js 文档，以查看有关 API 的所有信息：[`nodejs.org/api/events.html`](https://nodejs.org/api/events.html)。

# 准备工作

在这个配方中，您将创建一个类，它将扩展`EventEmitter`，并且将包含其自己的实例方法来触发附加到特定事件的侦听器。 首先，通过打开终端并运行以下命令来创建一个新项目：

```js
npm init
```

# 如何做...

创建一个类，它扩展`EventEmitter`并定义两个名为`start`和`stop`的实例方法。 当调用`start`方法时，它将触发附加到`start`事件的所有侦听器。 它将使用`process.hrtime`保持起始时间。 然后，当调用`stop`方法时，它将触发附加到`stop`事件的所有侦听器，并将自`start`方法调用以来的时间差作为参数传递：

1.  创建一个名为`timer.js`的新文件

1.  包括事件 NodeJS 模块：

```js
      const EventEmitter = require('events') 
```

1.  定义两个常量，我们将使用它们将`process.hrtime`的返回值从秒转换为纳秒，然后转换为毫秒：

```js
      const NS_PER_SEC = 1e9 
      const NS_PER_MS = 1e6 
```

1.  定义一个名为`Timer`的类，其中包含两个实例方法：

```js
      class Timer extends EventEmitter { 
          start() { 
              this.startTime = process.hrtime() 
              this.emit('start') 
          } 
          stop() { 
              const diff = process.hrtime(this.startTime) 
              this.emit( 
                  'stop', 
                  (diff[0] * NS_PER_SEC + diff[1]) / NS_PER_MS, 
              ) 
          } 
      } 
```

1.  创建先前定义的类的新实例：

```js
      const tasks = new Timer() 
```

1.  将一个事件监听器附加到`start`事件，它将有一个循环执行乘法。之后，它将调用`stop`方法：

```js
      tasks.on('start', () => { 
          let res = 1 
          for (let i = 1; i < 100000; i++) { 
              res *= i 
          } 
          tasks.stop() 
      }) 
```

1.  将一个事件监听器附加到`stop`事件，它将打印事件`start`执行所有附加监听器所花费的时间：

```js
      tasks.on('stop', (time) => { 
          console.log(`Task completed in ${time}ms`) 
      }) 
```

1.  调用`start`方法来触发所有`start`事件监听器：

```js
      tasks.start() 
```

1.  保存文件

1.  打开一个新的终端并运行：

```js
 node timer.js
```

# 它是如何工作的...

当执行`start`方法时，它使用`process.hrtime`来保留开始时间，该方法返回一个包含两个项目的数组，第一个项目是表示秒的数字，而第二个项目是表示纳秒的另一个数字。然后，它触发所有附加到`start`事件的事件监听器。

另一方面，当执行`stop`方法时，它使用之前调用`process.hrtime`的结果作为相同函数的参数，该函数返回时间差。这对于测量从调用`start`方法到调用`stop`方法的时间非常有用。

# 还有更多...

一个常见的错误是假设事件是异步调用的。确实，定义的事件可以在任何时候被调用。然而，它们仍然是同步执行的。看下面的例子：

```js
const EventEmitter = require('events') 
const events = new EventEmitter() 
events.on('print', () => console.log('1')) 
events.on('print', () => console.log('2')) 
events.on('print', () => console.log('3')) 
events.emit('print') 
```

上述代码的输出将如下所示：

```js
1 
2 
3 
```

如果你的事件中有一个循环在运行，下一个事件将不会被调用直到前一个完成执行。

事件可以通过简单地将`async`函数添加为事件监听器来变成异步的。这样做，每个函数仍然会按照从第一个定义的`listener`到最后一个的顺序被调用。然而，发射器不会等待第一个`listener`完成执行才调用下一个 listener。这意味着你不能保证输出总是按照相同的顺序，例如：

```js
events.on('print', () => console.log('1')) 
events.on('print', async () => console.log( 
    await Promise.resolve('2')) 
) 
events.on('print', () => console.log('3')) 
events.emit('print')  
```

上述代码的输出将如下所示：

```js
1 
3 
2 
```

异步函数允许我们编写非阻塞的应用程序。如果实现正确，您不会遇到上面的问题。

`EventEmitter`实例有一个名为`listeners`的方法，当执行时，提供一个事件名称作为参数，返回附加到该特定事件的监听器数组。我们可以使用这种方法以允许`async`函数按照它们被附加的顺序执行，例如：

```js
const EventEmitter = require('events') 
class MyEvents extends EventEmitter { 
    start() { 
        return this.listeners('logme').reduce( 
            (promise, nextEvt) => promise.then(nextEvt), 
            Promise.resolve(), 
        ) 
    } 
} 
const event = new MyEvents() 
event.on('logme', () => console.log(1)) 
event.on('logme', async () => console.log( 
    await Promise.resolve(2) 
)) 
event.on('logme', () => console.log(3)) 
event.start() 
```

这将按照它们被附加的顺序执行并显示输出：

```js
1 
2 
3 
```

# 理解 Socket.IO 事件

Socket.IO 是一个基于`EventEmitter`的事件驱动模块或库，正如您可能猜到的那样。Socket.IO 中的一切都与事件有关。当新连接建立到 Socket.IO 服务器时，将触发一个事件，并且可以发出事件以向客户端发送数据。

Socket.IO 服务器 API 与 Socket.IO 客户端 API 不同。然而，两者都使用事件来从客户端向服务器发送数据，反之亦然。

# Socket.IO 服务器事件

Socket.IO 使用单个 TCP 连接到单个路径。这意味着，默认情况下，连接是建立到 URL`http[s]://host:port/socket.io`。然而，在 Socket.IO 中，它允许您定义**命名空间**。这意味着不同的终点，但连接仍然保持单一 URL。

默认情况下，Socket.IO 服务器使用`"/"`或根命名空间

当然，您可以定义多个实例并监听不同的 URL。然而，为了本教程的目的，我们将假设只创建一个连接。

Socket.IO 命名空间具有以下事件，您的应用程序可以订阅：

+   `connect`或`connection`：当建立新连接时，将触发此事件。它将**socket 对象**作为第一个参数提供给监听器，表示与客户端的新连接

```js
      io.on('connection', (socket) => { 
          console.log('A new client is connected') 
      }) 
      // Which is the same as:
       io.of('/').on('connection', (socket) => { 
          console.log('A new client is connected') 
      }) 
```

Socket.IO 套接字对象具有以下事件：

+   `disconnecting`：当客户端即将从服务器断开连接时发出此事件。它向监听器提供一个指定断开连接原因的参数

```js
      socket.on('disconnecting', (reason) => { 
          console.log('Disconnecting because', reason) 
      }) 
```

+   `disconnected`：类似于断开连接事件。但是，此事件在客户端从服务器断开连接后触发：

```js
      socket.on('disconnect', (reason) => { 
          console.log('Disconnected because', reason) 
      }) 
```

+   `error`：当事件发生错误时触发此事件

```js
      socket.on('error', (error) => { 
          console.log('Oh no!', error.message) 
      }) 
```

+   `[eventName]`：一个用户定义的事件，当客户端发出具有相同名称的事件时将被触发。客户端可以发出一个提供参数中的数据的事件。在服务器上，事件将被触发，并且将接收客户端发送的数据

# Socket.IO 客户端事件

客户端不一定需要是一个网络浏览器。我们也可以编写一个 Node.js Socket.IO 客户端应用程序。

Socket.IO 客户端事件非常广泛，可以对应用程序进行很好的控制：

+   `connect`：当成功连接到服务器时触发此事件

```js
      clientSocket.on('connect', () => { 
          console.log('Successfully connected to server') 
      }) 
```

+   `connect_error`：当尝试连接或重新连接到服务器时出现错误时，会触发此事件

```js
      clientSocket.on('connect_error', (error) => { 
          console.log('Connection error:', error) 
      }) 
```

+   `connect_timeout:` 默认情况下，在发出`connect_error`和`connect_timeout`之前设置的超时时间为 20 秒。之后，Socket.IO 客户端可能会再次尝试重新连接到服务器：

```js
      clientSocket.on('connect_timeout', (timeout) => { 
          console.log('Connect attempt timed out after', timeout) 
      }) 
```

+   `disconnect`：当客户端从服务器断开连接时触发此事件。提供一个参数，指定断开连接的原因：

```js
      clientSocket.on('disconnect', (reason) => { 
          console.log('Disconnected because', reason) 
      }) 
```

+   `reconnect`：在成功重新连接尝试后触发。提供一个参数，指定在连接成功之前发生了多少次尝试：

```js
      clientSocket.on('reconnect', (n) => { 
          console.log('Reconnected after', n, 'attempt(s)') 
      }) 
```

+   `reconnect_attempt`或`reconnecting`：当尝试重新连接到服务器时会触发此事件。提供一个参数，指定当前尝试连接到服务器的次数：

```js
      clientSocket.on('reconnect_attempt', (n) => { 
          console.log('Trying to reconnect again', n, 'time(s)') 
      })  
```

+   `reconnect_error`：类似于`connect_error`事件。但是，只有在尝试重新连接到服务器时出现错误时才会触发：

```js
      clientSocket.on('reconnect_error', (error) => { 
          console.log('Oh no, couldn't reconnect!', error) 
      })  
```

+   `reconnect_failed:` 默认情况下，尝试的最大次数设置为`Infinity`。这意味着，这个事件很可能永远不会被触发。但是，我们可以指定一个选项来限制最大连接尝试次数。稍后我们会看到：

```js
      clientSocket.on('reconnect_failed', (n) => { 
    console.log('Couldn'nt reconnected after', n, 'times') 
      }) 
```

+   `ping`：简而言之，此事件被触发以检查与服务器的连接是否仍然存在：

```js
      clientSocket.on('ping', () => { 
          console.log('Checking if server is alive') 
      }) 
```

+   `pong`：在从服务器接收到`ping`事件后触发。提供一个参数，指定延迟或响应时间：

```js
      clientSocket.on('pong', (latency) => { 
          console.log('Server responded after', latency, 'ms') 
      }) 
```

+   `error`：当事件发生错误时触发此事件：

```js
      clientSocket.on('error', (error) => { 
          console.log('Oh no!', error.message) 
      }) 
```

+   `[eventName]`：当在服务器中发出事件时触发的用户定义的事件。服务器提供的参数将被客户端接收。

# 准备工作

在这个示例中，您将使用刚刚学到的有关事件的知识构建一个 Socket.IO 服务器和一个 Socket.IO 客户端。在开始之前，请创建一个新的`package.json`文件，内容如下：

```js
{ 
  "dependencies": { 
    "socket.io": "2.1.0" 
  } 
} 
```

然后，通过打开终端并运行以下命令来安装依赖项：

```js
npm install 
```

# 如何做...

将构建一个 Socket.IO 服务器来响应一个名为`time`的单个事件。当事件被触发时，它将获取服务器的当前时间，并发出另一个名为`"got time?"`的事件，提供两个参数，当前的`time`和一个指定请求次数的`counter`。

1.  创建一个名为`simple-io-server.js`的新文件

1.  包括 Socket.IO 模块并初始化一个新服务器：

```js
      const io = require('socket.io')() 
```

1.  定义连接将被建立的 URL 路径：

```js
      io.path('/socket.io') 
```

1.  使用根目录或`"/"`命名空间：

```js
      const root = io.of('/') 
```

1.  当建立新连接时，将`counter`变量初始化为`0`。然后，添加一个新的监听器到`time`事件，每次有新的请求时，将`counter`增加一次，并发出后来在客户端定义的`"got time?"`事件：

```js
      root.on('connection', socket => { 
          let counter = 0 
          socket.on('time', () => { 
              const currentTime = new Date().toTimeString() 
              counter += 1 
              socket.emit('got time?', currentTime, counter) 
          }) 
      }) 
```

1.  监听端口`1337`以获取新连接：

```js
      io.listen(1337) 
```

1.  保存文件

接下来，构建一个连接到我们服务器的 Socket.IO 客户端：

1.  创建一个名为`simple-io-client.js`的新文件

1.  包括 Socket.IO 客户端模块：

```js
      const io = require('socket.io-client') 
```

1.  初始化一个新的 Socket.IO 客户端，提供服务器 URL 和一个选项对象，在该对象中我们将定义 URL 中使用的路径，连接将在该路径上进行：

```js
      const clientSocket = io('http://localhost:1337', { 
          path: '/socket.io', 
      }) 
```

1.  为`connect`事件添加一个事件监听器。然后，当建立连接时，使用`for`循环，发出`time`事件 5 次：

```js
      clientSocket.on('connect', () => { 
          for (let i = 1; i <= 5; i++) { 
              clientSocket.emit('time') 
          } 
      }) 
```

1.  在`"got time?"`事件上添加一个事件监听器，该事件将期望接收两个参数，时间和一个指定了向服务器发出了多少次请求的计数器，然后在控制台上打印：

```js
      clientSocket.on('got time?', (time, counter) => { 
          console.log(counter, time) 
      }) 
```

1.  保存文件

1.  打开终端并首先运行 Socket.IO 服务器：

```js
    node simple-io-server.js
```

1.  打开另一个终端并运行 Socket.IO 客户端：

```js
    node simple-io-client.js
```

# 工作原理...

一切都与事件有关。Socket.IO 允许在服务器端定义客户端可以发出的事件。另一方面，它还允许在客户端端定义服务器可以发出的事件。

当服务器端发出用户定义的事件时，数据被发送到客户端。Socket.IO 客户端首先检查是否有该事件的监听器。然后，如果有监听器，它将被触发。当客户端端发出用户定义的事件时，同样的事情也会发生：

1.  在我们的 Socket.IO 服务器的**socket 对象**中添加了一个事件监听器`time`，可以由客户端发出

1.  在我们的 Socket.IO 客户端中添加了一个事件监听器`"got time?"`，可以由服务器端发出

1.  在连接时，客户端首先发出`time`事件

1.  随后，在服务器端触发`time`事件，该事件将提供两个参数，当前服务器的`time`和一个指定了请求次数的`counter`

1.  然后，在客户端端触发`"got time?"`事件，接收服务器提供的两个参数，`time`和`counter`。

# 使用 Socket.IO 命名空间

命名空间是一种分隔应用程序业务逻辑的方式，同时重用相同的 TCP 连接或最小化创建新 TCP 连接的需求，以实现服务器和客户端之间的实时通信。

命名空间看起来与 ExpressJS 的路由路径非常相似：

```js
/home 
/users 
/users/profile 
```

然而，正如前面的配方中提到的，这些与 URL 无关。默认情况下，在此 URL`http[s]://host:port/socket.io`创建单个 TCP 连接

在使用命名空间时，重用相同的事件名称是一个很好的做法。例如，假设我们有一个 Socket.IO 服务器，当客户端发出`getWelcomeMsg`事件时，我们用来发出`setWelcomeMsg`事件：

```js
io.of('/en').on('connection', (socket) => { 
    socket.on('getWelcomeMsg', () => { 
        socket.emit('setWelcomeMsg', 'Hello World!') 
    }) 
}) 
io.of('/es').on('connection', (socket) => { 
    socket.on('getWelcomeMsg', () => { 
        socket.emit('setWelcomeMsg', 'Hola Mundo!') 
    }) 
}) 
```

正如您所看到的，我们在两个不同的命名空间中为事件`getWelcomeMsg`定义了监听器：

+   如果客户端连接到英语或`/en`命名空间，当`setWelcomeMsg`事件被触发时，客户端将收到`"Hello World!"`

+   另一方面，如果客户端连接到西班牙语或`/es`命名空间，当`setWelcomeMsg`事件被触发时，客户端将收到`"Hola Mundo!"`

# 准备工作

在本配方中，您将看到如何使用包含相同事件名称的两个不同命名空间。在开始之前，请创建一个新的`package.json`文件，其中包含以下内容：

```js
{ 
  "dependencies": { 
    "socket.io": "2.1.0" 
  } 
} 
```

然后，通过打开终端并运行以下命令来安装依赖项：

```js
npm install
```

# 如何做...

构建一个 Socket.IO 服务器，该服务器将触发一个`data`事件，并发送一个包含两个属性`title`和`msg`的对象，该对象将用于填充所选语言的 HTML 内容。使用命名空间来分隔并根据客户端选择的语言（英语或西班牙语）发送不同的数据。

1.  创建一个名为`nsp-server.js`的新文件

1.  包括 Socket.IO npm 模块和创建 HTTP 服务器所需的模块：

```js
      const http = require('http') 
      const fs = require('fs') 
      const path = require('path') 
      const io = require('socket.io')() 
```

1.  使用`http`模块创建一个新的 HTTP 服务器，该服务器将作为 Socket.IO 客户端提供的 HTML 文件的服务：

```js
     const app = http.createServer((req, res) => { 
      if (req.url === '/') { 
               fs.readFile( 
               path.resolve(__dirname, 'nsp-client.html'), 
              (err, data) => { 
                  if (err) { 
                    res.writeHead(500) 
                    return void res.end() 
                   } 
                    res.writeHead(200) 
                    res.end(data) 
                } 
              ) 
          } else { 
              res.writeHead(403) 
             res.end() 
         } 
    }) 
```

1.  指定新连接将要进行的路径：

```js
      io.path('/socket.io') 
```

1.  对于`"/en"`命名空间，添加一个新的事件监听器`getData`，当触发时将在客户端发出一个`data`事件，并发送一个包含`title`和`msg`属性的对象，使用英语语言：

```js
     io.of('/en').on('connection', (socket) => { 
        socket.on('getData', () => { 
            socket.emit('data', { 
               title: 'English Page', 
               msg: 'Welcome to my Website', 
           }) 
        }) 
   }) 
```

1.  对于`"/es"`命名空间，做同样的事情。但是，发送到客户端的对象将包含西班牙语言中的`title`和`msg`属性：

```js
      io.of('/es').on('connection', (socket) => { 
          socket.on('getData', () => { 
              socket.emit('data', { 
                  title: 'Página en Español', 
                  msg: 'Bienvenido a mi sitio Web', 
              }) 
          }) 
      }) 
```

1.  监听端口`1337`以获取新连接，并将 Socket.IO 附加到底层 HTTP 服务器：

```js
      io.attach(app.listen(1337, () => { 
          console.log( 
              'HTTP Server and Socket.IO running on port 1337' 
          ) 
      })) 
```

1.  保存文件。

之后，创建一个 Socket.IO 客户端，将连接到我们的服务器，并根据从服务器接收到的数据填充 HTML 内容。

1.  创建一个名为`nsp-client.html`的新文件

1.  首先，将文档类型指定为 HTML5。在其旁边，添加一个`html`标签，并将语言设置为英语。在`html`标签内，还包括`head`和`body`标签：

```js
      <!DOCTYPE html> 
      <html lang="en"> 
      <head> 
          <meta charset="UTF-8"> 
          <title>Socket.IO Client</title> 
      </head> 
      <body> 
          <!-- code here --> 
      </body> 
      </html> 
```

1.  在`body`标签内，添加前三个元素：一个包含内容标题的标题（`h1`），一个包含来自服务器的消息的`p`标签，以及一个用于切换到不同命名空间的`button`。还包括 Socket.IO 客户端库。Socket.IO 服务器将在此 URL 上提供库文件：http[s]://host:port/socket.io/socket.io.js。然后，还包括`babel`独立库，它将把下一步的代码转换为可以在所有浏览器中运行的 JavaScript 代码：

```js
      <h1 id="title"></h1> 
      <section id="msg"></section> 
      <button id="toggleLang">Get Content in Spanish</button> 
       <script src="img/socket.io.js">  
       </script> 
        <script src="img/babel.min.js">
      </script> 
```

1.  在`body`内，在最后的`script`标签之后，添加另一个`script`标签，并将其类型设置为`"text/babel"`：

```js
      <script type="text/babel"> 
          // code here! 
      </script> 
```

1.  之后，在`script`标签内，添加以下 JavaScript 代码

1.  定义三个常量，它们将包含对`body`中创建的元素的引用：

```js
      const title = document.getElementById('title') 
      const msg = document.getElementById('msg') 
      const btn = document.getElementById('toggleLang') 
```

1.  定义一个 Socket.IO 客户端管理器。它将帮助我们使用提供的配置创建套接字：

```js
      const manager = new io.Manager( 
          'http://localhost:1337', 
          { path: '/socket.io' }, 
      ) 
```

1.  创建一个新的套接字，将连接到`"/en"`命名空间。我们将假设这是默认连接：

```js
      const socket = manager.socket('/en') 
```

1.  为`"/en"`和`"/es"`命名空间保留两个连接。保留连接将允许我们在不需要创建新的 TCP 连接的情况下切换到不同的命名空间：

```js
      manager.socket('/en') 
      manager.socket('/es') 
```

1.  添加一个事件监听器，一旦套接字连接，就会发出一个`getData`事件来请求服务器的数据：

```js
      socket.on('connect', () => { 
          socket.emit('getData') 
      }) 
```

1.  添加一个`data`事件的事件监听器，当客户端从服务器接收到数据时将被触发：

```js
      socket.on('data', (data) => { 
          title.textContent = data.title 
          msg.textContent = data.msg 
      }) 
```

1.  为`button`添加一个事件监听器。当单击时，切换到不同的命名空间：

```js
      btn.addEventListener('click', (event) => { 
          socket.nsp = socket.nsp === '/en' 
              ? '/es' 
              : '/en' 
          btn.textContent = socket.nsp === '/en' 
              ? 'Get Content in Spanish' 
              : 'Get Content in English' 
          socket.close() 
          socket.open() 
      }) 
```

1.  保存文件

1.  打开一个新的终端并运行：

```js
 node nsp-server.js
```

1.  在 Web 浏览器中，导航到：

```js
 http://localhost:1337/
```

# 让我们来测试一下...

要查看之前的工作效果，请按照以下步骤操作：

1.  一旦在 Web 浏览器中导航到`http://localhost:1337/`，单击`"Get Content in Spanish"`按钮，切换到西班牙语命名空间

1.  单击`"Get Content in English"`按钮，切换回英语命名空间

# 工作原理...

这是服务器端发生的事情：

1.  我们定义了两个命名空间，`"/en"`和`"/es"`，然后向**套接字对象**添加了一个新的事件监听器`getData`。

1.  当在任何两个定义的命名空间中触发`getData`事件时，它将发出一个数据事件，并向客户端发送一个包含标题和消息属性的对象

在客户端，在我们的 HTML 文档的`script`标签内：

1.  最初，为命名空间`"/en"`创建一个新的套接字：

```js
      const socket = manager.socket('/en')
```

1.  同时，我们为`"/en"`和`"/es"`命名空间创建了两个新的**套接字**。它们将充当保留连接：

```js
      manager.socket('/en')
      manager.socket('/es')
```

1.  之后，添加了一个事件监听器`connect`，在连接时向服务器发送请求

1.  然后，添加了另一个`data`事件的事件监听器，当从服务器接收到数据时触发

1.  在处理按钮的`onclick`事件的事件监听器内部，我们将`nsp`属性更改为切换到不同的命名空间。但是，为了实现这一点，我们必须首先断开**套接字**，然后调用`open`方法，再次使用新的命名空间建立新的连接

让我们看看关于保留连接的一个令人困惑的部分。当您在同一个命名空间中创建一个或多个**sockets**时，第一个连接会被重用，例如：

```js
const first = manager.socket('/home')
const second = manager.socket('/home') // <- reuses first connection
```

在客户端，如果没有保留连接，那么切换到以前未使用过的命名空间将导致创建一个新连接。

如果您感到好奇，请从`nsp-client.html`文件中删除这两行：

```js
manager.socket('/en')
manager.socket('/es')
```

之后，重新启动或再次运行 Socket.IO 服务器。您会注意到切换到不同命名空间时会有一个缓慢的响应，因为会创建一个新连接而不是重用。

有一种替代方法可以实现相同的目标。我们可以创建两个指向两个不同命名空间`"/en"`和`"/es"`的 socket。然后，我们可以为每个 socket 添加两个事件监听器 connect 和 data。然而，因为第一个和第二个 socket 将包含相同的事件名称，并且以相同的格式从服务器接收数据，我们将得到重复的代码。想象一下，如果我们必须为五个具有相同事件名称并以相同格式从服务器接收数据的不同命名空间做同样的事情，那将会有太多重复的代码行。这就是切换命名空间并重用相同的 socket 对象有帮助的地方。然而，可能存在两个或更多不同的命名空间具有不同事件名称的情况，对于不同类型的事件，最好为每个命名空间单独添加事件监听器。例如：

```js
const englishNamespace = manager.socket('/en')
const spanishNamespace = manager.socket('/es')
// They listen to different events
englishNamespace.on('showMessage', (data) => {})
spanishNamespace.on('mostrarMensaje', (data) => {})
```

# 还有更多...

在客户端，您可能已经注意到了一个我们以前没有使用过的东西，`io.Manager`。

# io.Manager

这使我们能够预定义或配置新连接将如何创建。在`Manager`中定义的选项，如 URL，将在初始化时传递给 socket。

在我们的 HTML 文件中，在`script`标签内，我们创建了`io.Manager`的一个新实例，并传递了两个参数；服务器 URL 和一个包含`path`属性的选项对象，该属性指示新连接将被创建的位置：

```js
const manager = new io.Manager( 
    'http://localhost:1337', 
    { path: '/socket.io' }, 
) 
```

要了解有关`io.Manager`API 的更多信息，请访问官方文档网站提供的 Socket.IO [`socket.io/docs/client-api/#manager`](https://socket.io/docs/client-api/#manager)。

稍后，我们使用了`socket`方法来初始化并创建一个提供的命名空间的新 Socket：

```js
const socket = manager.socket('/en') 
```

这样，就可以更容易地同时处理多个命名空间，而无需为每个命名空间配置相同的选项。

# 定义和加入 Socket.IO 房间

在命名空间内，您可以定义一个 socket 可以加入和离开的房间或通道。

默认情况下，房间会使用一个随机的不可猜测的 ID 来创建与连接的**socket**：

```js
io.on('connection', (socket) => { 
    console.log(socket.id) // Outputs socket ID 
}) 
```

在连接时，例如发出一个事件时：

```js
io.on('connection', (socket) => { 
    socket.emit('say', 'hello') 
}) 
```

底层发生的情况类似于这样：

```js
io.on('connection', (socket) => { 
    socket.join(socket.id, (err) => { 
        if (err) { 
            return socket.emit('error', err) 
        } 
        io.to(socket.id).emit('say', 'hello') 
    }) 
}) 
```

`join`方法用于将 socket 包含在房间内。在这种情况下，socket ID 是联合房间，连接到该房间的唯一客户端就是 socket 本身。

因为 socket ID 代表与客户端的唯一连接，并且默认情况下会创建具有相同 ID 的房间；服务器发送到该房间的所有数据将只被该客户端接收。然而，如果几个客户端或 socket ID 加入具有相同名称的房间，并且服务器发送数据；所有客户端都可以接收到。

# 准备工作

在这个示例中，您将看到如何加入一个房间并向连接到该特定房间的所有客户端广播消息。在开始之前，创建一个新的`package.json`文件，内容如下：

```js
{ 
  "dependencies": { 
    "socket.io": "2.1.0" 
  } 
} 
```

然后，通过打开终端并运行以下命令来安装依赖项：

```js
npm install
```

# 如何做...

构建一个 Socket.IO 服务器，当新的 socket 连接时，它将通知所有连接的客户端加入`"commonRoom"`房间。

1.  创建一个名为`rooms-server.js`的新文件

1.  包括 Socket.IO NPM 模块并初始化一个新的 HTTP 服务器：

```js
      const http = require('http') 
      const fs = require('fs') 
      const path = require('path') 
      const io = require('socket.io')() 
      const app = http.createServer((req, res) => { 
          if (req.url === '/') { 
              fs.readFile( 
                  path.resolve(__dirname, 'rooms-client.html'), 
                  (err, data) => { 
                     if (err) { 
                          res.writeHead(500) 
                          return void res.end() 
                      } 
                      res.writeHead(200) 
                      res.end(data) 
                  } 
              ) 
          } else { 
              res.writeHead(403) 
              res.end() 
          } 
      }) 
```

1.  指定新连接将被创建的路径：

```js
      io.path('/socket.io') 
```

1.  使用根命名空间来监听事件：

```js
      const root = io.of('/') 
```

1.  定义一个方法，用于向连接到`"commonRoom"`的所有套接字客户端发出`updateClientCount`事件，并提供连接的客户端数量作为参数：

```js
      const notifyClients = () => { 
          root.clients((error, clients) => { 
              if (error) throw error 
              root.to('commonRoom').emit( 
                  'updateClientCount', 
                  clients.length, 
              ) 
          }) 
      } 
```

1.  连接后，所有新连接的 Socket 客户端都将加入`commonRoom`。然后，服务器将发出`welcome`事件。之后，通知所有连接的套接字更新连接客户端的数量，并在客户端断开连接时执行相同的操作：

```js
      root.on('connection', socket => { 
          socket.join('commonRoom') 
          socket.emit('welcome', `Welcome client: ${socket.id}`) 
          socket.on('disconnect', notifyClients) 
          notifyClients() 
      }) 
```

1.  监听端口`1337`以进行新连接，并将 Socket.IO 附加到 HTTP 服务器：

```js
      io.attach(app.listen(1337, () => { 
          console.log( 
              'HTTP Server and Socket.IO running on port 1337' 
          ) 
      })) 
```

1.  保存文件。

之后，构建一个 Socket.IO 客户端，该客户端将连接到 Socket.IO 服务器并使用接收到的数据填充 HTML 内容：

1.  创建一个名为`rooms-client.html`的新文件

1.  添加以下代码：

```js
      <!DOCTYPE html> 
      <html lang="en"> 
      <head> 
          <meta charset="UTF-8"> 
          <title>Socket.IO Client</title> 
      </head> 
      <body> 
          <h1 id="title"> 
              Connected clients: 
              <span id="n"></span> 
          </h1> 
          <p id="welcome"></p> 
          <script src="img/socket.io.js">
          </script> 
          <script 
          src="img/babel.min.js">
          </script> 
          <script type="text/babel"> 
      // Code here 
          </script> 
      </body> 
      </html> 
```

1.  在脚本标签中，按以下步骤添加代码，从第 4 步开始

1.  定义两个常量，它们将引用两个 HTML 元素，我们将根据 Socket.IO 服务器发送的数据进行更新：

```js
      const welcome = document.getElementById('welcome') 
      const n = document.getElementById('n') 
```

1.  定义一个 Socket.IO 客户端管理器：

```js
      const manager = new io.Manager( 
          'http://localhost:1337', 
          { path: '/socket.io' }, 
      ) 
```

1.  使用 Socket.IO 服务器中使用的根命名空间：

```js
      const socket = manager.socket('/') 
```

1.  为`welcome`事件添加事件侦听器，该事件预期包含服务器发送的欢迎消息作为参数：

```js
      socket.on('welcome', msg => { 
          welcome.textContent = msg 
      }) 
```

1.  为`updateClientCount`事件添加事件侦听器，该事件预期包含一个参数，该参数将包含连接的客户端数量：

```js
      socket.on('updateClientCount', clientsCount => { 
          n.textContent = clientsCount 
      }) 
```

1.  保存文件

1.  打开一个新的终端并运行：

```js
 node rooms-server.js
```

1.  在 Web 浏览器中，导航到：

```js
http://localhost:1337/
```

1.  在不关闭上一个选项卡或窗口的情况下，再次在 Web 浏览器中导航到：

```js
http://localhost:1337/
```

1.  两个选项卡或窗口中连接的客户端数量应该增加到`2`

# 还有更多...

向多个客户端发送相同的消息或数据称为广播。我们已经看到的方法向所有客户端广播消息，包括生成请求的客户端。

还有其他几种广播消息的方法。例如：

```js
socket.to('commonRoom').emit('updateClientCount', data) 
```

这将向`commonRoom`中的所有客户端发出`updateClientCount`事件，但不包括发出请求的发送方或套接字。

有关完整列表，请查看 Socket.IO 发射速查表的官方文档：[`socket.io/docs/emit-cheatsheet/`](https://socket.io/docs/emit-cheatsheet/)

# 为 Socket.IO 编写中间件

Socket.IO 允许我们在服务器端定义两种类型的中间件函数：

+   **命名空间中间件**：注册一个函数，该函数将在每个新连接的 Socket 上执行，并具有以下签名：

```js
      namespace.use((socket, next) => { ... }) 
```

+   **Socket 中间件**：注册一个函数，该函数将在每个传入的数据包上执行，并具有以下签名：

```js
      socket.use((packet, next) => { ... }) 
```

它的工作方式类似于 ExpressJS 中间件函数。我们可以向`socket`或`packet`对象添加新属性。然后，我们可以调用`next`将控制传递给链中的下一个中间件。如果未调用`next`，则不会连接`socket`，或者接收到的`packet`。

# 准备工作

在这个示例中，您将构建一个 Socket.IO 服务器应用程序，在其中定义中间件函数以限制对某个命名空间的访问，以及根据某些条件限制对某个套接字的访问。在开始之前，请创建一个包含以下内容的新的`package.json`文件：

```js
{ 
  "dependencies": { 
    "socket.io": "2.1.0" 
  } 
} 
```

然后，通过打开终端并运行以下命令来安装依赖项：

```js
    npm install
```

# 如何做...

Socket.IO 服务器应用程序将期望用户已登录，以便他们能够连接到`/home`命名空间。使用 socket 中间件，我们还将限制对`/home`命名空间的访问权限：

1.  创建一个名为`middleware-server.js`的新文件

1.  包括 Socket.IO 库并初始化一个新的 HTTP 服务器：

```js
      const http = require('http') 
      const fs = require('fs') 
      const path = require('path') 
      const io = require('socket.io')() 
      const app = http.createServer((req, res) => { 
          if (req.url === '/') { 
              fs.readFile( 
                  path.resolve(__dirname, 'middleware-cli.html'), 
                  (err, data) => { 
                      if (err) { 
                          res.writeHead(500) 
                          return void res.end() 
                      } 
                      res.writeHead(200) 
                      res.end(data) 
                  } 
              ) 
          } else { 
              res.writeHead(403) 
              res.end() 
          } 
      }) 
```

1.  指定新连接将建立的路径：

```js
      io.path('/socket.io') 
```

1.  定义一个用户数组，我们将将其用作内存数据库：

```js
      const users = [ 
          { username: 'huangjx', password: 'cfgybhji' }, 
          { username: 'johnstm', password: 'mkonjiuh' }, 
          { username: 'jackson', password: 'qscwdvb' }, 
      ] 
```

1.  定义一个方法来验证提供的用户名和密码是否存在于用户数组中：

```js
      const userMatch = (username, password) => ( 
          users.find(user => ( 
              user.username === username && 
              user.password === password 
          )) 
      ) 
```

1.  定义一个命名空间中间件函数，该函数将检查用户是否已经登录。如果用户未登录，客户端将无法使用此中间件连接到特定命名空间：

```js
      const isUserLoggedIn = (socket, next) => { 
          const { session } = socket.request 
          if (session && session.isLogged) { 
              next() 
          } 
      } 
```

1.  定义两个命名空间，一个用于`/login`，另一个用于`/home`。`/home`命名空间将使用我们之前定义的中间件函数来检查用户是否已登录：

```js
      const namespace = { 
          home: io.of('/home').use(isUserLoggedIn), 
          login: io.of('/login'), 
      } 
```

1.  当一个新的 socket 连接到`/login`命名空间时，首先我们将为检查所有传入的数据包定义一个 socket 中间件函数，并禁止`johntm`用户名的访问。然后，我们将为输入事件添加一个事件监听器，该事件将期望接收一个包含用户名和密码的纯对象，如果它们存在于用户数组中，那么我们将设置一个会话对象，告诉用户是否已登录。否则，我们将向客户端发送一个带有错误消息的`loginError`事件：

```js
      namespace.login.on('connection', socket => { 
          socket.use((packet, next) => { 
              const [evtName, data] = packet 
              const user = data 
              if (evtName === 'tryLogin' 
                  && user.username === 'johnstm') { 
                  socket.emit('loginError', { 
                      message: 'Banned user!', 
                  }) 
              } else { 
                  next() 
              } 
          }) 
          socket.on('tryLogin', userData => { 
              const { username, password } = userData 
              const request = socket.request 
              if (userMatch(username, password)) { 
                  request.session = { 
                      isLogged: true, 
                      username, 
                  } 
                  socket.emit('loginSuccess') 
              } else { 
                  socket.emit('loginError', { 
                      message: 'invalid credentials', 
                  }) 
              } 
          }) 
      }) 
```

1.  监听端口 1337 以获取新连接并将 Socket.IO 附加到 HTTP 服务器：

```js
      io.attach(app.listen(1337, () => { 
          console.log( 
              'HTTP Server and Socket.IO running on port 1337' 
          ) 
      })) 
```

1.  保存文件

之后，构建一个 Socket.IO 客户端应用程序，它将连接到我们的 Socket.IO 服务器，并允许我们尝试登录和测试：

1.  创建一个名为`middleware-cli.html`的新文件

1.  添加以下代码：

```js
      <!DOCTYPE html> 
      <html lang="en"> 
      <head> 
          <meta charset="UTF-8"> 
          <title>Socket.IO Client</title> 
          <script src="img/socket.io.js">
          </script> 
          <script 
          src="img/babel.min.js">
          </script> 
      </head> 
      <body> 
          <h1 id="title"></h1> 
          <form id="loginFrm" disabled> 
            <input type="text" name="username" placeholder="username"/> 
              <input type="password" name="password" 
                placeholder="password" /> 
              <input type="submit" value="LogIn" /> 
              <output name="logs"></output> 
          </form> 
          <script type="text/babel"> 
              // Code here 
          </script> 
      </body> 
      </html> 
```

1.  在脚本标签内，从步骤 4 开始，添加以下代码

1.  定义三个常量，它们将引用我们将用于获取输入或显示输出的 HTML 元素：

```js
      const title = document.getElementById('home') 
      const error = document.getElementsByName('logErrors')[0] 
      const loginForm = document.getElementById('loginForm') 
```

1.  定义一个 Socket.IO 管理器：

```js
      const manager = new io.Manager( 
          'http://localhost:1337', 
          { path: '/socket.io' }, 
      ) 
```

1.  让我们定义一个命名空间常量，其中包含一个包含 Socket.IO 命名空间`/home`和`/login`的对象：

```js
      const namespace = { 
          home: manager.socket('/home'), 
          login: manager.socket('/login'), 
      } 
```

1.  为`/home`命名空间添加一个`connect`事件的事件监听器。只有当`/home`命名空间成功连接到服务器时才会触发：

```js
      namespace.home.on('connect', () => { 
          title.textContent = 'Great! you are connected to /home' 
          error.textContent = '' 
      }) 
```

1.  为`/login`命名空间添加一个`loginSuccess`事件的事件监听器。它将要求`/home`命名空间再次连接到服务器。如果用户已登录，则服务器将允许此连接：

```js
      namespace.login.on('loginSuccess', () => { 
          namespace.home.connect() 
      }) 
```

1.  为`/login`命名空间添加一个`loginError`事件的事件监听器。它将显示服务器发送的错误消息：

```js
      namespace.login.on('loginError', (err) => { 
          logs.textContent = err.message 
      }) 
```

1.  为登录表单的提交事件添加事件监听器。它将发出输入事件，提供一个包含在表单中填写的用户名和密码的对象：

```js
      form.addEventListener('submit', (event) => { 
          const body = new FormData(form) 
          namespace.login.emit('tryLogin', { 
              username: body.get('username'), 
              password: body.get('password'), 
          }) 
          event.preventDefault() 
      }) 
```

1.  保存文件

# 让我们来测试一下...

查看我们之前的工作的效果： 

1.  首先运行 Socket.IO 服务器。打开一个新的终端并运行：

```js
 node middleware-server.js
```

1.  在您的网络浏览器中，导航到：

```js
 http://localhost:1337
```

1.  您将看到一个带有两个字段`username`和`password`的登录表单

1.  尝试使用随机无效的凭据登录。将显示以下错误：

```js
      invalid credentials 
```

1.  接下来，尝试使用`johntm`作为`username`和任何`password`登录。将显示以下错误：

```js
      Banned user! 
```

1.  之后，使用另外两个有效凭据之一登录。例如，使用`jingxuan`作为用户名和`qscwdvb`作为密码。将显示以下标题：

```js
      Connected to /home 
```

# 将 Socket.IO 与 ExpressJS 集成

Socket.IO 与 ExpressJS 配合良好。事实上，可以在同一端口或 HTTP 服务器上运行 ExpressJS 应用程序和 Socket.IO 服务器。

# 准备工作

在这个示例中，我们将看到如何将 Socket.IO 与 ExpressJS 集成。您将构建一个 ExpressJS 应用程序，该应用程序将提供包含 Socket.IO 客户端应用程序的 HTML 文件。在开始之前，创建一个新的`package.json`文件，内容如下：

```js
{ 
  "dependencies": { 
    "express": "4.16.3", 
    "socket.io": "2.1.0" 
  } 
} 
```

然后，通过打开终端并运行来安装依赖项：

```js
npm install
```

# 如何做...

创建一个 Socket.IO 客户端应用程序，它将连接到您将要构建的 Socket.IO 服务器，并显示服务器发送的欢迎消息。

1.  创建一个名为`io-express-view.html`的新文件

1.  添加以下代码：

```js
      <!DOCTYPE html> 
      <html lang="en"> 
      <head> 
          <meta charset="UTF-8"> 
          <title>Socket.IO Client</title> 
          <script src="img/socket.io.js">
          </script> 
          <script 
           src="img/babel.min.js">
          </script> 
      </head> 
      <body> 
          <h1 id="welcome"></h1> 
          <script type="text/babel"> 
              const welcome = document.getElementById('welcome') 
              const manager = new io.Manager( 
                  'http://localhost:1337', 
                  { path: '/socket.io' }, 
              ) 
              const root = manager.socket('/') 
              root.on('welcome', (msg) => { 
                  welcome.textContent = msg 
              }) 
          </script> 
      </body> 
      </html> 
```

1.  保存文件

接下来，构建一个 ExpressJS 应用程序和一个 Socket.IO 服务器。ExpressJS 应用程序将在根路径`"/"`上提供先前创建的 HTML 文件：

1.  创建一个名为`io-express-server.js`的新文件

1.  初始化一个新的 Socket.IO 服务器应用程序和一个 ExpressJS 应用程序：

```js
      const path = require('path') 
      const express = require('express') 
      const io = require('socket.io')() 
      const app = express() 
```

1.  定义新连接将连接到 Socket.IO 服务器的 URL 路径：

```js
      io.path('/socket.io') 
```

1.  定义一个路由方法来提供包含我们的 Socket.IO 客户端应用程序的 HTML 文件：

```js
      app.get('/', (req, res) => { 
          res.sendFile(path.resolve( 
              __dirname, 
              'io-express-view.html', 
          )) 
      }) 
```

1.  定义一个命名空间`"/"`并发出一个带有欢迎消息的`welcome`事件：

```js
      io.of('/').on('connection', (socket) => { 
          socket.emit('welcome', 'Hello from Server!') 
      }) 
```

1.  将 Socket.IO 附加到 ExpressJS 服务器：

```js
      io.attach(app.listen(1337, () => { 
          console.log( 
              'HTTP Server and Socket.IO running on port 1337' 
          ) 
      })) 
```

1.  保存文件

1.  打开终端并运行：

```js
 node io-express-server.js
```

1.  在您的浏览器中访问：

```js
http://localhost:1337/
```

# 它是如何工作的...

Socket.IO 的`attach`方法期望接收一个 HTTP 服务器作为参数，以便将 Socket.IO 服务器应用程序附加到它上面。我们之所以能够将 Socket.IO 附加到 ExpressJS 服务器应用程序上，是因为`listen`方法返回 ExpressJS 连接的基础 HTTP 服务器。

总之，`listen`方法返回基础 HTTP 服务器。然后，它作为参数传递给`attach`方法。这样，我们可以与 ExpressJS 共享相同的连接。

# 还有更多...

到目前为止，我们已经看到我们可以在 ExpressJS 和 Socket.IO 之间共享相同的基础 HTTP 服务器。然而，这还不是全部。

我们定义 Socket.IO 路径的原因实际上在与 ExpressJS 一起工作时非常有用。看以下示例：

```js
const express = require('express') 
const io = require('socket.io')() 
const app = express() 
io.path('/socket.io')
 app.get('/socket.io', (req, res) => { 
    res.status(200).send('Hey there!') 
}) 
io.of('/').on('connection', socket => { 
    socket.emit('someEvent', 'Data from Server!') 
}) 
io.attach(app.listen(1337)) 
```

正如您所看到的，我们在 Socket.IO 和 ExpressJS 中使用相同的 URL 路径。我们接受新连接到`/socket.io`路径上的 Socket.IO 服务器，但我们也使用 GET 路由方法发送内容到`/socket.io`。

尽管上述示例实际上不会破坏您的应用程序，但请确保永远不要同时使用相同的 URL 路径来从 ExpressJS 提供内容并接受 Socket.IO 的新连接。例如，将上一个代码更改为以下内容：

```js
io.path('/socket.io')
 app.get('/socket.io/:msg', (req, res) => { 
    res.status(200).send(req.params.msg) 
}) 
```

当您访问`http://localhost:1337/socket.io/message`时，您可能期望您的浏览器显示`message`，但事实并非如此，您将看到以下内容：

```js
{"code":0,"message":"Transport unknown"} 
```

这是因为 Socket.IO 将首先解释传入的数据，它不会理解您刚刚发送的数据。此外，您的路由处理程序将永远不会被执行。

除此之外，Socket.IO 服务器还默认提供其自己的 Socket.IO 客户端，位于定义的 URL 路径下。例如，尝试访问[`localhost:1337/socket.io/socket.io.js`](http://localhost:1337/socket.io/socket.io.js)，您将能够看到 Socket.IO 客户端的最小化 JavaScript 代码。

如果您希望提供自己版本的 Socket.IO 客户端，或者如果它包含在您的应用程序的捆绑包中，您可以使用`serveClient`方法在 Socket.IO 服务器应用程序中禁用默认行为。

```js
io.serveClient(false) 
```

# 另请参阅

+   第二章，*使用 Express.js 内置中间件函数为静态资源提供服务*

# 在 Socket.IO 中使用 ExpressJS 中间件

Socket.IO 命名空间中间件的工作方式与 ExpressJS 中间件非常相似。事实上，Socket 对象还包含一个`request`和一个`response`对象，我们可以使用它们以与 ExpressJS 中间件函数相同的方式存储其他属性：

```js
namespace.use((socket, next) => { 
    const req = socket.request 
    const res = socket.request.res 
    next() 
}) 
```

因为 ExpressJS 中间件函数具有以下签名：

```js
const expressMiddleware = (request, response, next) => { 
    next() 
} 
```

我们可以安全地在 Socket.IO 命名空间中间件中执行相同的函数，传递必要的参数：

```js
root.use((socket, next) => { 
    const req = socket.request 
    const res = socket.request.res 
    expressMiddleware(req, res, next) 
}) 
```

然而，这并不意味着所有 ExpressJS 中间件函数都能直接使用。例如，如果 ExpressJS 中间件函数仅使用 ExpressJS 中可用的方法，它可能会失败或产生意外行为。

# 准备工作

在这个示例中，我们将看到如何将 ExpressJS 的`express-session`中间件集成到 Socket.IO 和 ExpressJS 之间共享会话对象。在开始之前，创建一个新的`package.json`文件，内容如下：

```js
{ 
  "dependencies": { 
    "express": "4.16.3", 
    "express-session": "1.15.6", 
    "socket.io": "2.1.0" 
  } 
} 
```

然后，通过打开终端并运行以下命令来安装依赖项：

```js
npm install
```

# 如何做...

构建一个 Socket.IO 客户端应用程序，它将连接到接下来您将构建的 Socket.IO 服务器。包括一个表单，用户可以在其中输入用户名和密码尝试登录。只有在用户登录后，Socket.IO 客户端才能连接到`/home`命名空间：

1.  创建一个名为`io-express-cli.html`的新文件

1.  添加以下 HTML 内容：

```js
      <!DOCTYPE html> 
      <html lang="en"> 
      <head> 
          <meta charset="UTF-8"> 
          <title>Socket.IO Client</title> 
          <script src="img/socket.io.js">  
          </script> 
          <script 
           src="img/babel.min.js">
          </script> 
      </head> 
      <body> 
          <h1 id="title"></h1> 
          <form id="loginForm"> 
            <input type="text" name="username" placeholder="username"/> 
              <input type="password" name="password" 
                placeholder="password" /> 
              <input type="submit" value="LogIn" /> 
              <output name="logErrors"></output> 
          </form> 
          <script type="text/babel"> 
              // Code here 
          </script> 
      </body> 
      </html> 
```

1.  在脚本标签中添加从第 4 步开始的下一步中的代码

1.  定义引用我们将使用的 HTML 元素的常量：

```js
      const title = document.getElementById('title') 
      const error = document.getElementsByName('logErrors')[0] 
      const loginForm = document.getElementById('loginForm') 
```

1.  定义一个 Socket.IO 管理器：

```js
      const manager = new io.Manager( 
          'http://localhost:1337', 
          { path: '/socket.io' }, 
      ) 
```

1.  定义两个命名空间，一个用于`/login`，另一个用于`/home`：

```js
      const namespace = { 
          home: manager.socket('/home'), 
          login: manager.socket('/login'), 
      } 
```

1.  为`welcome`事件添加一个事件监听器，该事件将在允许连接到`/home`命名空间时由服务器端触发：

```js
      namespace.home.on('welcome', (msg) => { 
          title.textContent = msg 
          error.textContent = '' 
      }) 
```

1.  为`loginSuccess`事件添加一个事件监听器，当触发时，将要求`/home`命名空间尝试重新连接到 Socket.IO 服务器：

```js
      namespace.login.on('loginSuccess', () => { 
          namespace.home.connect() 
      }) 
```

1.  为`loginError`事件添加一个事件监听器，当提供无效凭据时将显示错误：

```js
      namespace.login.on('loginError', err => { 
          error.textContent = err.message 
      }) 
```

1.  为`submit`事件添加一个事件监听器，当提交表单时将触发该事件。它将发出一个带有包含提供的`username`和`password`的数据的`enter`事件：

```js
      loginForm.addEventListener('submit', event => { 
          const body = new FormData(loginForm) 
          namespace.login.emit('enter', { 
              username: body.get('username'), 
              password: body.get('password'), 
          }) 
          event.preventDefault() 
      }) 
```

1.  保存文件。

在此之后，构建一个 ExpressJS 应用程序，该应用程序将在根路径`"/"`上提供 Socket.IO 客户端，并包含用于记录用户的逻辑的 Socket.IO 服务器：

1.  创建一个名为`io-express-srv.js`的新文件

1.  初始化一个新的 ExpressJS 应用程序和一个 Socket.IO 服务器应用程序。还包括`express-session` NPM 模块：

```js
      const path = require('path') 
      const express = require('express') 
      const io = require('socket.io')() 
      const expressSession = require('express-session') 
      const app = express() 
```

1.  定义新连接到 Socket.IO 服务器的路径：

```js
      io.path('/socket.io') 
```

1.  使用给定选项定义一个 ExpressJS 会话中间件函数：

```js
      const session = expressSession({ 
          secret: 'MERN Cookbook Secret', 
          resave: true, 
          saveUninitialized: true, 
      }) 
```

1.  定义一个 Socket.IO 命名空间中间件，该中间件将使用先前创建的会话中间件生成会话对象：

```js
      const ioSession = (socket, next) => { 
          const req = socket.request 
          const res = socket.request.res 
          session(req, res, (err) => { 
              next(err) 
              req.session.save() 
          }) 
      } 
```

1.  定义两个命名空间，一个用于`/home`，另一个用于`/login`：

```js
      const home = io.of('/home') 
      const login = io.of('/login') 
```

1.  定义一个内存数据库或包含`username`和`password`属性的对象数组。这些属性定义了允许登录的用户：

```js
      const users = [ 
          { username: 'huangjx', password: 'cfgybhji' }, 
          { username: 'johnstm', password: 'mkonjiuh' }, 
          { username: 'jackson', password: 'qscwdvb' }, 
      ] 
```

1.  在 ExpressJS 中包含会话中间件：

```js
      app.use(session) 
```

1.  为`/home`路径添加一个路由方法，用于提供我们之前创建的包含 Socket.IO 客户端的 HTML 文档：

```js
      app.get('/home', (req, res) => { 
          res.sendFile(path.resolve( 
              __dirname, 
              'io-express-cli.html', 
          )) 
      }) 
```

1.  在`/home` Socket.IO 命名空间中使用会话中间件。然后，检查每个新的 socket 是否已登录。如果没有，禁止用户连接到此命名空间：

```js
      home.use(ioSession) 
      home.use((socket, next) => { 
          const { session } = socket.request 
          if (session.isLogged) { 
              next() 
          } 
      }) 
```

1.  一旦连接到`/home`命名空间，也就是用户可以登录，就会发出一个带有欢迎消息的`welcome`事件，该消息将显示给用户：

```js
      home.on('connection', (socket) => { 
          const { username } = socket.request.session 
          socket.emit( 
              'welcome', 
              `Welcome ${username}!, you are logged in!`, 
          ) 
      }) 
```

1.  在`/login` Socket.IO 命名空间中使用会话中间件。然后，当客户端发出带有提供的用户名和密码的`enter`事件时，它会验证`users`数组中是否存在该配置文件。如果用户存在，则将`isLogged`属性设置为`true`，并将`username`属性设置为当前已登录的用户：

```js
      login.use(ioSession) 
      login.on('connection', (socket) => { 
          socket.on('enter', (data) => { 
              const { username, password } = data 
              const { session } = socket.request 
              const found = users.find((user) => ( 
                  user.username === username && 
                  user.password === password 
              )) 
              if (found) { 
                  session.isLogged = true 
                  session.username = username 
                  socket.emit('loginSuccess') 
              } else { 
                  socket.emit('loginError', { 
                      message: 'Invalid Credentials', 
                  }) 
              } 
          }) 
      }) 
```

1.  监听端口`1337`以获取新连接，并将 Socket.IO 服务器附加到该端口：

```js
      io.attach(app.listen(1337, () => { 
          console.log( 
              'HTTP Server and Socket.IO running on port 1337' 
          ) 
      })) 
```

1.  保存文件

1.  打开一个新的终端并运行：

```js
 node io-express-srv.js  
```

1.  在浏览器中访问：

```js
 http://localhost:1337/home
```

1.  使用有效的凭据登录。例如：

```js
      * Username: johntm
      * Password: mkonjiuh
```

1.  如果您成功登录，刷新页面后，您的 Socket.IO 客户端应用程序仍然能够连接到`/home`，并且每次都会看到欢迎消息

# 工作原理...

当在 ExpressJS 中使用会话中间件时，在修改会话对象后，响应结束时会自动调用`save`方法。然而，在 Socket.IO 命名空间中使用会话中间件时并非如此，这就是为什么我们需要手动调用`save`方法将会话保存回存储中的原因。在我们的情况下，存储是内存，会话会一直保存在那里直到服务器停止。

根据特定条件禁止访问某些命名空间是可能的，这要归功于 Socket.IO 命名空间中间件。如果控制权没有传递给`next`处理程序，那么连接就不会建立。这就是为什么在登录成功后，我们要求`/home`命名空间再次尝试连接。

# 另请参阅

+   第二章，*使用 ExpressJS 构建 Web 服务器*，*编写中间件函数*部分
