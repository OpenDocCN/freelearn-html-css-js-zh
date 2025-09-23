# 第六章。部署和扩展

在本地服务器上运行我们的应用程序是可以的，但要使一个网络应用程序真正有用，需要将其部署到公共服务器并使其对他人可访问。要在 Node.js 上运行我们的聊天服务器应用程序，除了使用 WebSocket 等协议外，还需要考虑一些特殊因素。在本章中，我们将探讨以下内容：

+   部署我们的应用程序时需要考虑的事项

+   生产就绪部署的建议

+   为什么 socket.io 应用程序的扩展与其他网络应用程序不同

+   我们如何扩展我们的聊天应用程序

# 生产环境

在生产服务器上运行应用程序之前，我们应该做的第一件事是将环境设置为 `production`。每个现代服务器或框架都有开发和生产模式，node 也不例外。实际上，在 node 中，你可以将环境设置为任何名称，然后在代码中为该名称设置不同的配置。要设置我们的 node 服务器运行的环境，我们设置一个环境变量 `NODE_ENV` 为我们想要运行 node 的环境。因此，要运行 node 在 `production` 环境中，我们使用以下行：

```js
$ export NODE_ENV=production

```

然后运行你的 node 应用程序。在 第二章 中，*入门 Node.js*，我们看到了 `app.configure` 中的第一个参数是我们需要配置的环境变量：

```js
app.configure('development', function(){
  app.use(express.errorHandler());
});
```

在这个片段中，我们正在将应用程序设置为在 `development` 环境中激活 `express.errorHandler`，这是默认环境。如果我们已将 `NODE_ENV` 设置为 `production`，则不会使用 `express.errorHandler`。

# 运行应用程序

使用 node 在命令行上运行应用程序，就像我们到目前为止所做的那样，在开发期间是可行的；但在我们远程连接的生产服务器上，通常不切实际或建议保持控制台运行。有两种处理方式，要么我们将 node 作为后台进程运行，将所有控制台输出重定向到文件，要么我们在持久控制台中运行它，我们可以使用 `screen` 或 `byobu` 重新连接到该控制台。

要将节点作为后台进程运行，就像在 Linux 上的任何其他进程一样，我们将使用 `&` 操作符，并且为了确保我们在登出后它仍然继续运行，我们将使用 `nohup`：

```js
$ nohup npm start 2>&1 >> npmout.log &

```

之前的命令将重定向 `stdout` 和 `stderr` 命令到 `npmout.log`，并将 npm 进程置于后台。

另一个选项是使用 `screen` 或 `byobu` 等实用程序在持久控制台上运行 node。要使用此方法，启动 `screen`，然后运行你的应用程序，如下所示：

```js
$ screen
$ npm start

```

现在，我们可以通过使用 *Ctrl* +*a* 然后按 *d* 来断开与此屏幕的连接。这将使我们回到默认的 shell。然后我们可以断开连接。当我们再次连接到服务器时，为了查看服务器输出，我们可以使用以下命令重新连接到屏幕：

```js
$ screen -r

```

# 保持运行

我们不仅希望应用程序在我们登出时运行，我们还希望我们的应用程序能够可靠地持续运行。生产服务器不经常重启，通常我们希望它们在发生崩溃、故障或错误时尽快恢复。对于 node，通常意味着在失败时立即重启进程。有许多方法可以保持 node 服务器运行。在本节中，我们将看到其中两种：

+   Monit

+   Forever

这是 Monit 在其网站上如何描述的（[`mmonit.com/monit/`](http://mmonit.com/monit/))：

> Monit 是一个免费的开源实用程序，用于管理和监控 UNIX 系统上的进程、程序、文件、目录和文件系统。Monit 执行自动维护和修复，并在错误情况下执行有意义的因果操作。

让我们从安装 Monit 开始。在基于 RPM 或 Yum 的系统上，如 RHEL、Fedora 或 CentOS，你可以使用`yum`命令安装它，如下所示：

```js
$ sudo yum install monit

```

或者，在基于 Debian 或 apt-get 的系统上，你可以使用`apt-get`安装 Monit：

```js
$ apt-get install monit

```

对于其他系统，你可以在 Monit 网站上查看安装说明。

一旦安装了 Monit，我们就可以配置它来管理我们的 node 应用程序。为此，我们将在`/etc/monit.d/`或`/etc/monit/conf.d/`中创建一个配置文件（在我们的例子中我们将称之为`awesome-chat`），具体取决于你的 Monit 安装：

```js
check host objdump with address 127.0.0.1
 start program = "/bin/sh -c \
    'NODE_ENV=production \
    node /opt/node_apps/awesome-chat/app.js 2>&1 \
    >> /var/log/awesome-chat.log'"
        as uid nobody and gid nobody
 stop program  = "/usr/bin/pkill -f \
  'node /opt/node_apps/awesome-chat/app.js'"
    if failed port 3000 protocol HTTP
        request /
        with timeout 10 seconds
        then restart
```

在这个文件中，你应该注意到高亮的部分。我们强调的是程序，或者更重要的是，启动/停止我们的应用程序的命令，然后最终配置 Monit 在发生故障时重启应用程序。这是通过向端口`3000`发送 HTTP 请求来检测的。

就这样；我们可以使用以下命令启动我们的应用程序：

```js
$ monit start awesome-chat

```

使用以下代码停止它：

```js
$ monit stop awesome-chat

```

在发生崩溃的情况下，Monit 将负责重启应用程序。

Monit 可以用来运行和监控任何守护进程服务。它还有一个网页界面，如果你想要检查状态，默认情况下它运行在端口`2812`上。你可以在 Monit 的网站上及其在线手册中了解更多关于 Monit 的信息。

另一种更具体的 node 方法来确保我们的服务器持续运行是**Forever** ([`github.com/nodejitsu/forever`](https://github.com/nodejitsu/forever))。Forever 描述自己为：

> 一个简单的 CLI 工具，用于确保给定的脚本持续运行。

这就是它的作用。给定你的 node 应用程序脚本，Forever 将启动它并确保它持续运行。由于 Forever 本身也是一个 node 应用程序，我们将使用 npm 来安装它：

```js
 $ sudo npm install forever -g

```

现在，要使用 Forever 启动应用程序，只需执行`app.js`文件并使用`forever`。只需运行以下命令：

```js
$ forever start app.js

```

我们可以使用以下命令查看持续运行的应用程序列表：

```js
$ forever list
 0 app.js [ 24597, 24596 ]

```

要停止应用程序，请使用`forever stop`命令：

```js
$ forever stop 0

```

访问 Forever 的 GitHub 页面，了解更多关于 Forever 及其工作原理的信息。

在 *nix 系统上还有其他一些工具可以将 node 运行为守护进程。以下是一些：

+   [`upstart.ubuntu.com/`](http://upstart.ubuntu.com/))

+   Supervisord ([`supervisord.org/`](http://supervisord.org/))

+   Daemontools ([`cr.yp.to/daemontools.html`](http://cr.yp.to/daemontools.html))

# 扩展

现在我们已经确保了我们的应用程序将持续运行，并且可以从故障中恢复，是时候开始考虑如何处理成千上万用户涌入我们的聊天室了。首先，第一步是在服务器前设置一个负载均衡器代理。这里有多种选择，我们可以使用 Apache HTTP 服务器、Nginx 等。所有这些服务器在平衡传统 HTTP 流量方面都工作得很好，但还需要一些时间才能赶上处理 WebSocket。因此，我们将使用一个本身就在负载均衡 TCP/IP 上工作的服务器。这是 HAProxy 在其官方网站上的描述：

> HAProxy 是一个免费、非常快速且可靠的解决方案，提供高可用性、负载均衡和 TCP 及 HTTP 应用程序的代理功能。它特别适合在需要持久性或第 7 层处理的情况下，处理承受极高负载的网站。在今天硬件的支持下，显然可以支持数万个连接。

HAProxy 与前端和后端一起工作。这些是通过位于 `/etc/haproxy/haproxy.cfg` 的 HAProxy 配置文件配置的。以下文件在端口 `80` 创建了一个前端监听器，并将其转发到单个位于 `3000` 的服务器：

```js
global
  maxconn 4096

defaults
  environment http

frontend all 0.0.0.0:80
  default_backend www_Node.js

backend www_Node.js
  environment http
  option forwardfor
  server Node.js 127.0.0.1:3000 weight 1 maxconn 10000 check
```

在此文件中，我们正在定义一个位于 `0.0.0.0:80` 的前端监听器，默认的 `www_Node.js` 后端在相同的 `127.0.0.1` 服务器上监听 `3000`。

但此配置尚未准备好处理 WebSocket。要支持和处理 WebSocket，请参考以下代码块：

```js
global
  maxconn 4096

defaults
  environment http

frontend all 0.0.0.0:80
 timeout client 86400000
  default_backend www_Node.js
 acl is_websocket hdr(upgrade) -i websocket
 acl is_websocket hdr_beg(host) -i ws

 use_backend www_
Node.js if is_websocket

backend www_Node.js
  environment http
  option forwardfor
 timeout server 86400000
 timeout connect 4000
  server Node.js 127.0.0.1:3000 weight 1 maxconn 10000 check
```

我们首先做的事情是增加客户端超时值，这样当客户端长时间无活动时，客户端连接不会断开。`acl` 代码行指示 HAProxy 理解并检查我们何时收到 `websocket` 请求。

通过使用 `use_backend` 指令，我们配置 HAProxy 使用 `www_Node.js` 后端来处理 `websocket` 请求。当你想从任何服务器（如 Apache HTTP）提供静态页面，并希望仅使用 node 来处理 socket.io 时，这很有用。

现在我们来到想要请求由多个节点服务器/进程处理的环节。为此，首先我们将告诉代理通过添加以下指令到后端来实现请求的轮询：

```js
  balance roundrobin
```

然后我们将向后端添加更多服务器条目：

```js
  server Node.js 127.0.0.1:4000 weight 1 maxconn 10000 check
  server Node.js 192.168.1.101:3000 weight 1 maxconn 10000 check
```

在这里，我们添加了两个新的节点实例：一个是在同一服务器上监听端口 `4000` 的新进程，另一个运行在另一服务器上，该服务器可通过负载均衡器在 `192.168.1.101` 的端口 `3000` 上访问。

我们已完成服务器的配置，现在进入的请求将现在在配置的三个节点实例之间路由。

# 节点集群

Node 现在自带完全重写的集群模块。集群模块允许节点在集群前端启动多个进程，并监控和管理它们。我们将快速查看如何使用此模块创建应用程序集群，但请注意，这只是为了创建多个进程，我们仍然需要设置一个工具来监控集群主进程，以及一个代理来将请求转发到节点服务器。

让我们看看如何使用集群模块。集群模块最好的部分是，你实际上不需要更改你的应用程序。集群将运行一个主实例，我们可以启动我们应用程序的多个实例，它们都将监听一个共享端口。

这里是我们可以使用来集群化 `app.js` 文件的脚本：

```js
var cluster = require('cluster');

if (cluster.isMaster) {
  var noOfWorkers =
           process.env.NODE_WORKERS || require('os').cpus().length;
  while(workers.length < noOfWorkers) {
    cluster.fork();
  }
} else {
  require('./app.js');
}
```

那么，这里发生了什么？我们首先使用 `require` 引入 `cluster` 模块。在下一行，我们检查启动的实例是主进程还是工作进程。

如果是主进程，我们检查是否设置了 `NODE_WORKERS` 环境变量，否则获取服务器运行系统上的处理器数量。要设置 `NODE_WORKERS` 环境变量，你可以运行以下命令：

```js
$ export NODE_WORKERS=2

```

之前的命令将告诉集群启动两个节点。

现在，在循环中，我们在集群上调用 `fork`。这调用 `child_process.fork`，以便主进程和启动的工作进程可以通过 IPC 进行通信。

当从 `fork` 运行集群进程时，`cluster.isMaster` 为 false，因此我们的 `app.js` 脚本位于当前工作进程。

在我们的应用程序中，当我们调用 `server.listen(3000)` 时，工作进程将序列化此操作并将请求发送到服务器，服务器检查是否已经在该端口上监听，如果存在，则返回监听器的句柄。否则，服务器将开始在该端口上监听，并将句柄传递给新创建的监听器。

由于所有我们的工作进程都请求监听端口 `3000`，服务器将在第一个工作进程启动时开始监听该端口，并将相同的处理程序传递给所有工作进程。当请求到来时，它将由任何可以处理并处理该请求的工作进程处理。

由于我们的监控工具（Monit 或 Forever，或其他）现在将仅监控主进程，因此监控工作进程成为主进程的责任。这意味着集群应该重新启动任何意外死亡的工作进程。我们将通过在主进程中添加以下事件处理程序来完成此操作：

```js
cluster.on('exit', function (worker, code, signal){
  var exitCode = worker.process.exitCode;
  console.log('worker ' + worker.process.pid +
                               ' died ('+exitCode+'). restarting...');
  cluster.workers[worker.id].delete();
  cluster.fork();
});
```

通过监听套接字的 `exit` 事件来监控进程。这是任何工作进程死亡时将触发的事件。事件处理程序将获取工作进程、其退出代码以及导致进程被杀死的信号。在处理程序中，我们记录死亡情况，并使用 `cluster.fork()` 启动一个新的工作进程。

现在，我们可以启动新的集群应用程序；我们将运行 `cluster.js` 而不是 `app.js`。因此，将 `package.json` 中的 `start` 脚本更改为运行 `cluster.js`：

```js
 "scripts": {
    "start": "node cluster",
  }
```

然后使用 npm 运行应用程序。

```js
npm start

```

这将启动应用程序，一切看起来都和之前一样。但当你开始使用它时，你会注意到在尝试连接到房间或发送消息时出现了错误。这些错误是因为我们正在使用内存存储来存储 Express.js 会话，而 socket.io 使用内存存储来存储和传输所有消息。

# 应用程序扩展

在前面的部分中，我们看到了如何对 Node.js 应用程序进行集群化以及它如何由于我们的应用程序机制而受到限制。在其当前状态下，应用程序使用内存存储来保持会话数据。此存储是 Node.js 实例本地的，因此不会在任何其他集群实例中可用。此外，数据将在 Node.js 实例重启时丢失。因此，我们需要一种将会话存储在持久存储中的方法。此外，我们希望配置 socket.io，使其所有实例都使用共享的 pub-sub 和数据存储。Connect 框架有一个扩展机制，因此可以插入新的存储，并且有一个存储既持久又擅长 pub-sub。它是 **Redis** **会话存储**。

Redis ([`redis.io/`](http://redis.io/)) 是一个高性能、分布式、开源的键值存储，也可以用作队列。我们将使用 Redis 和相应的 Redis 存储来提供一个可靠、分布式和共享的存储和 pub-sub 队列。请查看在您的操作系统上安装 Redis 服务器并启动它的说明。

让我们对我们的聊天应用程序做一些更改，从 `package.json` 开始：

```js
    "connect-redis":"*",
    "redis":"*"
```

这将为 Connect/Express.js Redis 存储和 Redis 连接客户端添加支持。让我们首先让 Express.js 使用 Redis；为此，通过以下代码片段编辑 `app.js`：

```js
var express = require('express')
  , routes = require('./routes')
  , http = require('http')
  , path = require('path')
  , connect = require('connect')
 , RedisStore = require('connect-redis')(express);

var app = express();

var sessionStore = new RedisStore();

//Existing Code
```

因此，我们在这里做的两个更改是引入 Redis 会话存储，然后我们可以将会话存储替换为 `RedisStore` 的实例。这就是 Express 使用 Redis 存储运行所需的所有内容。

下一步，我们需要使用 Redis 来配置 socket.io。因此，让我们编辑 `socket.js`：

```js
var io = require('socket.io')
 , redis = require('redis')
 , RedisStore = require('socket.io/lib/stores/redis')
 , pub    = redis.createClient()
 , sub    = redis.createClient()
 , client = redis.createClient();

exports.initialize = function (server) {
  io = io.listen(server);

 io.set('store', new RedisStore({
 redisPub : pub
 , redisSub : sub
 , redisClient : client
 }));

  //Existing Code
}
```

在前面的代码片段中，我们首先执行的是 `require('redis')`，它提供了客户端和 socket.io 的 `redisStore`，后者为 socket.io 提供了 Redis 支持。然后我们创建了三个不同的 Redis 客户端，用于 pub-sub 和数据存储：

```js
 io.set('store', new RedisStore({
 redisPub : pub
 , redisSub : sub
 , redisClient : client
 }));

```

在之前的代码片段中，我们配置了 socket.io 使用 Redis 作为队列和数据存储。现在我们可以开始了！现在使用以下命令再次运行应用程序：

```js
npm start

```

# 生产中 node 的技巧

这里有一些帮助我们执行 node 在生产中的技巧：

1.  在 `生产` 环境中运行服务器。

1.  永远不要直接将 node 应用程序暴露在互联网上；总是使用代理。例如 Apache HTTP、Nginx 和 HAProxy 这样的服务器在多年的生产中已经得到了加固和增强，以使其能够抵御各种类型的攻击，特别是 DOS 和 DDOS。Node 是新的；它可能随着时间的推移而变得稳定，但今天不建议将其直接放在前端。

1.  永远不要以 root 身份运行 node。嗯，这是对任何应用服务器的建议，也适用于 node。如果我们以 root 身份运行 node，黑客可能会获得 root 访问权限或以 root 身份运行一些有害的代码。所以，永远不要以 root 身份运行它！

1.  总是运行多个节点进程。Node 是一个单线程、单进程的应用服务器。应用中的错误可能会导致服务器崩溃。因此，为了可靠性，总是要有多个进程。此外，以 1+ 进程的方式思考，使我们能够在需要时扩展。

1.  总是使用监控工具。Monit、Forever、Upstart 选择你喜欢的，但总是使用它。安全比抱歉更好。

1.  永远不要在 `生产` 环境中使用 `MemoryStore`；`MemoryStore` 是为 `开发` 环境准备的；我建议即使在 `开发` 环境中也使用 `RedisStore`。

1.  记录所有错误。一切运行顺利，直到它不顺利！当出现问题的时候，日志是你的最佳朋友。尽量在尽可能接近原因的地方捕获异常，并在上下文中记录所有相关信息。不要只记录一些错误消息，要记录所有相关的对象。

1.  除非没有其他选择，否则永远不要阻塞。Node 运行在事件循环上，对单个请求的阻塞将导致不必要的开销，并降低所有请求的性能。

1.  总是保持你的服务器、node 和所有依赖模块的最新状态。

# 摘要

在本节中，我们看到了将我们的应用程序部署到生产环境所涉及的工作。我们必须记住，这些并不是唯一的方法。对于我们所做的每一项任务，都有许多其他的方法可以完成，没有一种解决方案适合所有场景。但既然我们已经知道了 `生产` 环境的期望，我们就可以研究选项，并根据我们的需求选择一个。
