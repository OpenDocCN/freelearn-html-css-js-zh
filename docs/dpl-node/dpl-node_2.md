# 第二章。安装和虚拟化 Node 服务器

回想一下来自第一章的故事，*欣赏 Node*，讲述了沃尔玛如何将所有*黑色星期五*的移动流量通过 Node 处理，Node 部署在*相当于 2 个 CPU 和 30GB RAM*的配置上。这证明了 Node 处理 I/O 的效率非常高，即使是*黑色星期五*的沃尔玛级流量也可以仅用几个服务器来处理。这意味着，对于许多人来说，在单个服务器上运行 Node 应用程序就是他们需要做的全部。

然而，通常最好有多个服务器可供使用，例如冗余服务器以确保故障恢复，一个独立的数据库服务器，专门的媒体服务器，一个托管消息队列的，等等。按照将关注点分离到许多独立进程的想法，基于 Node 的应用程序通常由许多轻量级服务器组成，这些服务器分布在数据中心，甚至可能分布在几个数据中心。

在本章中，我们将具体探讨如何设置单个 Node 服务器，无论是实际还是虚拟的。目标是探索在响应扩展需求时大规模生产服务器的选项，并了解如何将这些服务器连接起来。你将学习如何自己设置 HTTP/S 服务器，以及如何使用 Node 进行隧道和代理。然后，我们将探讨一些流行的云托管解决方案以及如何在那些解决方案上设置 Node 服务器。最后，我们将讨论**Docker**，这是一种创建轻量级虚拟服务的新兴技术。

# 启动基本的 Node 服务器

HTTP 是在请求/响应模型上构建的数据传输协议。通常，客户端向服务器发出请求，收到响应，然后再次发出请求，依此类推。HTTP 是无状态的，这仅仅意味着每个请求或响应都不会保留有关先前请求或响应的信息。促进这种快速模式网络通信的是 Node 设计用来擅长的 I/O 类型。虽然 Node 代表了一个更有趣的整个技术栈，但它确实帮助工程师创建网络协议服务器。在本节中，我们将概述如何设置基本的 HTTP 服务器，然后探讨该协议的一些更专业的用途。

## Hello world

HTTP 服务器响应连接尝试，并管理到达和发送的数据。Node 服务器通常使用 HTTP 模块的`createServer`方法创建：

```js
var http = require('http');

var server = http.createServer(function(request, response) {
  console.log('Got Request Headers: ');
  console.log(request.headers);
  response.writeHead(200, {
    'Content-Type': 'text/plain'
  });
  response.write('PONG');
  response.end();
}).listen(8080);
```

`http.createServer`返回的对象是`http.Server`的一个实例，它扩展了`EventEmitter`，并在事件发生时广播网络事件，例如客户端连接或请求。大多数使用 Node 的服务器实现都使用这种方法进行实例化。然而，通过`http.Server`实例监听事件广播可以是在 Node 程序中组织服务器/客户端交互的更有效、甚至更自然的方式。

在这里，我们创建了一个基本的服务器，它简单地报告连接建立和终止的情况：

```js
var http = require('http');
var server = new http.Server();

server.on("connection", function(socket) {
  console.log("Client arrived: " + new Date());
  socket.on("end", function() {
    console.log("Client left: " + new Date());
  });
})

server.listen(8080);
```

在构建多用户系统，尤其是经过身份验证的多用户系统时，服务器-客户端事务的这个点是一个进行客户端验证和跟踪代码的绝佳位置。可以设置和读取 cookies，以及其他会话变量。可以将客户端到达事件广播给其他在实时应用程序中交互的并发客户端。

通过添加请求监听器，我们达到了更常见的请求/响应模式，它被处理为一个`Readable`流。当客户端发送数据时，我们可以捕获这些数据，如下所示：

```js
server.on("request", function(request, response) {
  request.setEncoding("utf8");
  request.on("readable", function() {
    console.log(request.read())
  });
});
```

使用**curl**向这个服务器发送一些数据：

```js
curl http://localhost:8080 -d "Here is some data"
```

使用连接事件，我们可以很好地将我们的连接处理代码分离，将其分组到清晰定义的功能域中，这些域被正确地描述为响应特定事件而执行。

例如，我们可以在服务器连接上设置定时器。在这里，我们可以在大约 2 秒的窗口内未能发送新数据的客户端连接上进行终止：

```js
server.setTimeout(2000, function(socket) {
  socket.write("Too Slow!", "utf8");
  socket.end();
});
```

## 发送 HTTP 请求

HTTP 服务器通常被要求为发起请求的客户端执行 HTTP 服务。最常见的情况是，这种代理服务是在浏览器中运行的受跨域请求限制的 Web 应用 behalf 上进行的。Node 提供了一个简单的接口来执行外部 HTTP 调用。

例如，以下代码将获取[google.com](http://google.com)的首页：

```js
var http = require('http');

http.request({
  host: 'www.google.com',
  method: 'GET',
  path: "/"
}, function(response) {
  response.setEncoding('utf8');
  response.on('readable', function() {
    console.log(response.read())
  });
}).end();
```

在这里，我们只是简单地将一个`Readable`流输出到终端，但这个流可以很容易地被管道传输到一个`Writable`流，可能是一个文件句柄。请注意，您必须始终使用`request.end`方法来表示您已完成请求。

### 小贴士

一个流行的 Node 模块来管理 HTTP 请求是 Mikeal Rogers 的**request**：

[`github.com/mikeal/request`](https://github.com/mikeal/request)

由于通常使用`HTTP.request`来获取外部页面，Node 提供了一个快捷方式：

```js
http.get("http://www.google.com/", function(response) {
  console.log("Status: " + response.statusCode);
}).on('error', function(err) {
  console.log("Error: " + err.message);
});
```

现在我们来看一些更高级的 HTTP 服务器实现，在这些实现中，我们为客户端执行通用网络服务。

## 代理和隧道

有时，提供一个让一个服务器作为其他服务器的代理或经纪人的手段是有用的。这将允许一个服务器将请求分发到其他服务器，例如。另一个用途是提供给无法直接连接到该服务器的用户访问受保护服务器的权限——这在限制互联网访问的国家很常见。通常，一个服务器通过代理回答多个 URL；那个服务器可以将请求转发给正确的接收者。

由于 Node 具有一致的网络接口，这些接口作为事件流实现，我们可以用几行代码就构建一个简单的 HTTP 代理。例如，以下程序将在端口`8080`上设置一个 HTTP 服务器，该服务器将对任何请求做出响应，通过获取谷歌的首页并将其管道传输回客户端：

```js
var http = require('http');
var server = new http.Server();

server.on("request", function(request, socket) {
  http.request({
    host: 'www.google.com',
    method: 'GET',
    path: "/",
    port: 80
  }, function(response) {
    response.pipe(socket);
  }).end();
});

server.listen(8080);
```

一旦这个服务器收到客户端套接字，它就可以自由地将任何可读流的内容推回到客户端。在这里，对 [www.google.com](http://www.google.com) 的 GET 请求的结果就是这样流式传输的。人们可以很容易地看到，一个管理着应用程序缓存层的远程内容服务器可能成为一个代理端点。

使用类似的想法，我们可以创建一个使用 Node 的原生 CONNECT 支持的隧道服务：

```js
var http = require('http');
var net = require('net');
var url = require('url');
var proxy = new http.Server();

proxy.on('connect', function(request, clientSocket, head) {
  var reqData = url.parse('http://' + request.url);
  var remoteSocket = net.connect(reqData.port, reqData.hostname, function() {
    clientSocket.write('HTTP/1.1 200 \r\n\r\n');
    remoteSocket.write(head);

    // The bi-directional tunnel
    remoteSocket.pipe(clientSocket);
    clientSocket.pipe(remoteSocket);
  });
}).listen(8080, function() {
```

我们已经设置了一个代理服务器，它会对请求 HTTP CONNECT 方法（`on("connect")`）的客户端做出响应，该方法包含请求对象、网络套接字绑定客户端和服务器以及隧道的 'head'（第一个数据包）。当从客户端收到 CONNECT 请求时，我们解析出 `request.url`，获取请求的主机信息，并打开请求的网络套接字。通过将远程数据管道到客户端并将客户端数据管道到远程连接，建立了一个双向数据隧道。现在我们只需要向我们的代理发送 CONNECT 请求，如下所示：

```js
  var request = http.request({
    port: 8080,
    hostname: 'localhost',
    method: 'CONNECT',
    path: 'www.google.com:80'
  });
  request.end();
```

一旦收到我们 CONNECT 请求的状态 200 确认，我们就可以将请求数据包推送到这个隧道中，捕获响应并将这些内容输出到 `stdout`：

```js
  request.on('connect', function(res, socket, head) {
    socket.setEncoding("utf8");
    socket.write('GET / HTTP/1.1\r\nHost: www.google.com:80\r\nConnection: close\r\n\r\n');
    socket.on('readable', function() {
      console.log(socket.read());
    });
    socket.on('end', function() {
      proxy.close();
    });
  });
});
```

## HTTPS、TLS（SSL）和保障你的服务器安全

Web 应用程序在规模、重要性和复杂性方面都得到了增长。因此，Web 应用程序的安全性已经成为一个重要的话题。由于一个或多个原因，早期的 Web 应用程序被允许进入客户端业务逻辑的实验世界，未加密的密码传输和开放的网络服务，而仅仅是被一层薄薄的帘子所保护。对于关注其信息安全的用户来说，这种情况变得越来越难找到。

由于 Node 通常被部署为 Web 服务器，社区开始对这些服务器的安全性负责是至关重要的。HTTPS 是一种安全的传输协议——本质上，是在 SSL/TLS 协议之上叠加 HTTP 协议的加密 HTTP。让我们学习如何确保我们的 Node 部署的安全性。

## 为开发创建自签名证书

为了支持 SSL 连接，服务器需要一个正确签名的证书。在开发过程中，简单地创建一个自签名证书会容易得多，这允许我们使用 Node 的 HTTPS 模块。

这些是创建开发证书所需的步骤。请记住，这个过程不会创建真正的证书，生成的证书 **不安全**——它只是允许我们在终端中从 HTTPS 环境中进行开发：

```js
openssl genrsa -out server-key.pem 2048
openssl req -new -key server-key.pem -out server-csr.pem
openssl x509 -req -in server-csr.pem -signkey server-key.pem -out server-cert.pem

```

这些密钥现在可以用来开发 HTTPS 服务器。这些文件的内容只需作为选项传递给运行在（默认）SSL 端口 `443` 上的 Node 服务器：

```js
var https = require('https');
var fs = require('fs');

https.createServer({
  key: fs.readFileSync('server-key.pem'),
  cert: fs.readFileSync('server-cert.pem')
}, function(req,res) {
   ...
}).listen(443)
```

### 注意

对于在开发过程中自签名证书不是理想选择的情况，可以从 [`www.startssl.com/`](http://www.startssl.com/) 获取免费的 **低保证** SSL 证书。

## 安装真实 SSL 证书

为了将安全的应用程序从开发环境迁移到互联网暴露的环境，需要购买真实的证书。这些证书的价格逐年下降，应该很容易找到提供价格合理且安全级别足够的证书的供应商。一些供应商甚至提供免费的个人使用证书。

设置专业证书只需更改我们之前介绍的 HTTPS 选项。不同的供应商会有不同的流程和文件名。通常，您需要从您的供应商那里下载或接收一个私有的 `#key` 文件，您的签名域名证书 `#crt` 文件，以及一个描述证书链的通用 `#ca` 文件包：

```js
var options = {
  key  : fs.readFileSync('mysite.key'),
  cert  : fs.readFileSync('mysite.com.crt'),
  ca  : [ fs.readFileSync('gd_bundle.crt') ]
};
```

重要的一点是，即使证书包已被连接成一个文件，`#ca` 参数也必须以 *数组* 的形式发送。

这里是本节的关键要点：

+   HTTP 套接字被抽象为事件流。这对于 Node 提供的所有网络接口都是正确的。这些流可以轻松地相互连接。

+   由于流活动是事件驱动的，因此可以记录这些事件。可以在事件处理器中记录关于系统行为的非常精确的日志信息，或者通过将流通过一个可能监听并记录事件的 `PassThrough Stream` 参数进行管道传输来记录。

+   Node 在 I/O 服务方面表现出色。Node 服务器可以作为调度器，仅关注在客户端与任何数量的远程服务或本地操作系统上运行的专用进程之间进行通信的经纪人。

现在您已经知道了如何在 Node 中设置 HTTP 服务器并处理协议，那么就继续实验吧。在您的本地机器上创建一个小应用程序，允许用户读取 Twitter 流或连接到公共数据 API。习惯通过网络验证远程服务，并通过它们的 API 或作为它们数据的代理与之交互。习惯通过使用 Node 作为经纪人，通过集成远程网络服务来构建网络应用程序。

在生产环境中运行自己的服务器可能既昂贵又耗时，尤其是如果您不熟悉系统管理。因此，许多云托管公司应运而生，其中许多是为 Node 开发者专门设计的。

让我们来看看其中的一些。为了比较，相同的 Node 应用程序将部署在每个平台上——一个存储在 **MongoDB** 中的可编辑 JSON 文档，绑定到一个简单的基于浏览器的 **用户界面**（**UI**）。鼓励您按顺序尝试这些服务，尽管这不是必需的。

# 在 Heroku 上安装应用程序

Heroku 是一个成熟的 PaaS 云托管解决方案，支持 Node 应用程序的开发。要开始，请访问[`www.heroku.com`](http://www.heroku.com)并提交一个电子邮件地址。Heroku 开始使用是免费的。在您确认了账户后，您就可以立即开始部署应用程序了。

扩展 Heroku 应用程序涉及增加您支付的费用中的*dynos*数量。每个**dyno**都是一个运行您应用程序的独立容器，您可以轻松地增加或减少应用程序使用的 dynos 数量。这样，您不需要购买任何托管*套餐*——您只需根据需要请求更多或更少的 dynos 即可进行扩展。

Heroku 允许您在许多平台和语言上部署应用程序——它不是以 Node 为中心的。如果您预计需要将非 Node 编写的服务添加到您的应用程序中，请记住这一点。

要控制 Heroku 远程实例，您将使用一个本地的*工具包*应用程序。一旦您加入 Heroku 并确认了您的注册，登录并转到仪表板上的**应用程序**部分。那里应该有关于安装 Heroku Toolbelt 的说明（[`toolbelt.heroku.com/`](https://toolbelt.heroku.com/))。

### 注意

`heroku`命令行客户端将被安装在`/usr/local/heroku`，并且`/usr/local/heroku/bin`将被添加到您的路径中。

一旦安装了 Toolbelt，打开终端并使用`heroku login`登录 Heroku。由于这是您第一次登录，您很可能会被要求生成一个公钥。一旦这个密钥生成并上传，您就安全了，并且从现在开始，您可以通过 Toolbelt 和命令行来管理您的 Heroku 部署。

如果 Heroku 在您的应用程序文件夹的根目录中找到一个`package.json`文件，它将识别您的应用程序为 Node 应用程序。我们的示例应用程序已经包含了一个，因此不需要再创建一个。然而，由于 Heroku 不是 Node 的专用主机，它不会自动在包文件的`start`属性中找到我们的-- `server.js` --应用程序的启动脚本：

```js
"scripts": {
  "start": "node server.js"
}
```

相反，Heroku 需要所谓的**Procfile**。在我们的示例应用程序的根目录中创建一个`Procfile`文件，并将以下文本插入其中：

```js
web: node server.js
```

它略有不同，但我们可以看出最终效果是相同的。Procfile 声明我们想要一个"web"进程——在执行`node server.js`命令后启动的进程将期望有 HTTP 流量路由到它。

### 注意

当您安装 Heroku Toolbelt 时，还会安装另一个应用程序：**Foreman**。Foreman 帮助您管理基于 Procfile 的应用程序。对我们来说，它最重要的作用是允许您在本地启动 Heroku 应用程序。虽然您可以直接通过 Node 更新 Node 包的`scripts`属性并直接运行应用程序，但这确实节省了一步。尝试`foreman start`并访问`localhost:8080`。

在以下章节中，我们将探讨如何在 Heroku 上安装和管理存储库，以及如何将 MongoDB 添加到我们的应用程序中，并且我们将部署一个 JSON 编辑应用程序。

## 添加组件

在 Heroku 上，数据库被视为许多附加组件之一。从日志工具到缓存层，再到数据库，Heroku 提供了数十种附加组件。由于我们需要一个 MongoDB 实例来运行我们的应用程序，让我们安装一个。

注意，虽然来自 MongoLab 的开发者（沙盒）MongoDB 实例是免费的，但 Heroku 需要您使用信用卡验证您的账户。如果您没有信用卡，仍然可以通过其他服务获得免费的 MongoDB 云账户，并使用这些凭据为您的 Heroku 应用程序提供服务。最后，我们只需要一个可以连接的 MongoDB 端点。

要添加 MongoDB 账户，请运行 `heroku addons:add mongolab` 命令：

```js
Adding mongolab on mighty-hamlet-7855... done, v14 (free)
Welcome to MongoLab. Your new subscription is being created and will be available shortly. Please consult the MongoLab Add-on Admin UI to check on its progress.

```

使用 `heroku addons:docs mongolab` 在浏览器中查看文档。

您刚刚为您的 Heroku 实例添加了一个配置选项。不出所料，您可以通过 `heroku config` 查看此信息，它将返回类似以下内容：

```js
MONGOLAB_URI: mongodb://heroku_app2485743:ie02k3nnic3l0tjfgi3135inq@ds035488.mongolab.com:35488/heroku_app2487483

```

在数据库建立之后，现在让我们将我们的应用程序推送到 Heroku 并使其运行。

## Git

在 Heroku 上部署应用程序涉及将您的本地版本推送到您刚刚配置的远程应用程序仓库。没有 `heroku deploy` 命令；您所做的是推送到 **Git**，从而在 Heroku 端触发 post-receive 钩子。这些钩子会部署您的应用程序。

### 注意

如果您不熟悉 Git，请访问 [`git-scm.com/book/en/Getting-Started-Git-Basics`](http://git-scm.com/book/en/Getting-Started-Git-Basics)。

让我们试试。在您的代码包中，存在一个 `json-editor` 文件夹。首先，进入该文件夹，并在 `server.js` 中更新 MongoDB 连接和身份验证代码，以便我们可以使用之前定义的数据库连接：

```js
var mongodb = require('mongodb');
var db = new mongodb.Db('your_db_identifier',
  new mongodb.Server('dt019963.mongolab.com', 29960, {})
);
db.open(function (err, db_p) {
  if (err) { throw err; }
  db.authenticate('your_username', '6i490i5d3teoen62524vqkccgu', function (err, replies) {
    // You are now connected and authenticated.
  });
});
```

接下来，在您的终端中运行以下命令：

```js
git init
git add .
git commit -m "initial commit"

```

这将初始化我们的应用程序为一个合适的 Git 仓库。现在，我们需要通知 Heroku 我们的新应用程序和新的 Git 仓库。让我们部署。

在您的代码包的 `json-editor` 文件夹中，使用 Heroku Toolbelt 创建您的第一个 Heroku 应用程序：

```js
heroku create

```

如果一切顺利，您应该在终端中看到类似以下内容：

```js
Creating mighty-hamlet-7855... done, stack is cedar
http://mighty-hamlet-7855.herokuapp.com/ | git@heroku.com:mighty-hamlet-7855.git
Git remote heroku added

```

如果你立即访问该 URL，你会收到一个错误消息。因为我们还没有推送我们的存储库，所以没有部署任何内容，这意味着没有可以展示的内容。要将应用程序部署到 Heroku，请推送您的本地 Git 仓库：

```js
git push heroku master

```

这应该会产生大量的构建输出，清楚地告知您正在发生的事情：

```js
-----> Node.js app detected
-----> Requested node range: 0.10.x
...
-----> Building runtime environment
-----> Discovering process types
 Procfile declares types -> web
...
-----> Launching... done, v3
 http://mighty-hamlet-7855.herokuapp.com/ deployed to Heroku

```

因此，将应用程序部署到 Heroku 自然地将实际的容器部署与通过 Git 进行应用程序版本管理结合起来。更重要的是，将您的 Git 仓库上的更改推送到 Heroku 将自动更新运行中的应用程序，允许“热”代码刷新。在某些情况下，能够持续部署您的应用程序可能非常有用，正如我们将在后面的章节中看到的。

在我们开始之前，请注意，你部署的应用程序的 URL 没有端口号。Heroku 会自动分配一个端口号，通过该端口号 Web 进程与应用程序通信——这不在我们的控制范围内。然而，它通过`process.env.PORT`提供给 Node 进程。因此，你需要将`server.js`中的`}).listen(8081);`行更改为`}).listen(process.env.PORT || 8081);`。

我们现在可以启动我们的应用程序了。记住，我们正在部署一个基于 Procfile 的应用程序——进程被定义为某种类型。在我们的例子中，这种类型是“web”。我们还需要为我们的部署分配 dynos——我们需要从 Heroku 中请求一个进程来运行我们的应用程序。启动此类应用程序的命令如下：

```js
heroku ps:scale web=1

```

这告诉 Heroku 给我们一个（`1`）个`web`类型的 dyno（也称为进程）。你也可以根据需要请求两个或更多。

运行那个命令。你应该会看到以下类似的内容：

```js
Scaling dynos... done, now running web at 1:1X.

```

这告诉我们一切运行正常，我们有一个`1`个大小为`1x`的 dyno 正在处理我们的应用程序。你可以使用`heroku ps`命令来检查你的进程是否正在运行：

```js
=== web (1X): `node server.js`
web.1: up 2014/04/04 17:40:34 (~ 27m ago)

```

我们的应用程序正在运行！访问你之前收到的 Heroku URL。你应该会看到一个 JSON 编辑器和我们的 MongoDB 文档：

![Git](img/1403OS_02_04.jpg)

这是一个 JSON 编辑器，正在读取我们服务器上创建的 MongoDB 文档。它除了让你更改`for`属性的值之外，没有做太多。如果你查看`index.html`中的 JavaScript 代码，你会看到我们已经将客户端结构化为在文档中的值更改时通过`/update`路径向服务器发送更新：

```js
var editor = new jsoneditor.JSONEditor(container, {
  change : function() {
    var json = editor.get();

    var xhr = new XMLHttpRequest();
    xhr.open('POST', '/update', true);
    xhr.onload = function () {
      console.log("POST RESPONSE: ", this.responseText);
    };
    xhr.send('data=' + JSON.stringify(json));
  },
  mode : "form"
});
```

尝试一下。使用编辑器将`Deploying NodeJS`更改为其他内容。如果你打开浏览器控制台，你应该会在更改此值时看到**POST 响应：OK**。在做出更改后，重新加载浏览器。你会看到新值——你做出的更改正在通过我们的 Heroku 实例持久化到 MongoDB。

## 管理配置变量

对于应用程序的某些方面可以进行配置是很正常的。例如，用于生产的部署的应用程序可能配置得与在开发环境中构建的应用程序不同。此外，认证凭证（例如我们用于 MongoDB 连接的凭证）将包含在环境变量中。

由于许多配置变量是敏感的，将它们包含在应用程序仓库或公开文件中是一个坏主意。如何以安全的方式在多个进程之间共享变量？一个解决方案是在通过命令行启动 Node 进程时传递环境变量。如果我们想通知 Node 进程它应该作为一个生产服务器执行，例如，我们可以这样做：

```js
NODE_ENV=PRODUCTION node myprogram.js

```

在那个脚本中，我们可以通过`process.env`访问值：

```js
console.log(process.env.NODE_ENV);
// production
```

虽然以这种方式传递配置变量在隐私方面非常有效，但每次为每个进程重复此操作可能会很繁琐，尤其是如果有许多变量。

Heroku 提供了一个界面来帮助管理环境变量。如果你登录到你的 Heroku 实例并访问**设置**部分，你会看到如下内容：

![管理配置变量](img/1403OS_02_05.jpg)

这些环境变量将在应用程序启动和/或重启时自动传递给应用程序。使用**编辑**按钮，你可以添加或删除额外的设置。

## 管理你的部署

如果你的应用程序因任何原因崩溃，`heroku ps`将会显示这一点。你也可以通过`heroku logs`访问你的进程日志。就像当你启动你的进程一样，停止你的进程涉及到将 dynos 的规模缩小到零：

```js
heroku ps:scale web=0
```

Heroku 允许你非常精确地扩展和配置你的进程，扩展到多个 dynos，添加各种工作者，并改变 dynos 本身的大小。在我们的示例中，我们使用基本的**1x** dyno，它具有最小的内存和计算能力，并且是最便宜的。更多信息，请访问[`devcenter.heroku.com/articles/dyno-size`](https://devcenter.heroku.com/articles/dyno-size)和[`devcenter.heroku.com/articles/process-model`](https://devcenter.heroku.com/articles/process-model)。

有时，你可能会提交一个错误的变化或想要重新部署之前的版本。不用担心！Toolbelt 允许你管理你的版本。

要列出版本，使用`heroku releases`：

```js
v11 Deploy 310fe56  nataxia@gmail.com 2014/05/04 18:19:45 (~ 6m ago)
v10 Deploy a0c6005  nataxia@gmail.com 2014/05/04 18:15:17 (~ 10m ago)
...

```

你可以获取有关特定版本的详细信息：

```js
> heroku releases:info v11
=== Release v11
By:   spasquali@gmail.com
Change: Deploy 310fe56
When:  2014/04/04 18:19:45 (~ 8m ago)

```

通过简单的 Heroku 回滚可以回滚到立即之前的版本。你也可以回滚到特定的版本：

```js
> heroku rollback v11
Rolling back mighty-hamlet-7855... done, v11

```

就像在推送更改时一样，回滚到的版本将自动“上线”。

### 小贴士

你可以使用`heroku open`从命令行直接打开你的应用程序。

# 在 OpenShift 上安装应用程序

红帽公司，一家企业级 Linux 公司，运营着 OpenShift，这是一个云托管解决方案。OpenShift 提供了多种部署应用程序的选项——通过基于 Web 的界面、通过命令行或通过在线 IDE。由于我们在其他部署示例中使用了命令行，我们将以相同的方式使用 OpenShift。

一旦你加入并确认了你的账户，你需要安装 OpenShift 客户端工具——`rhc`。为了本节的目的，我将使用 Mac OS X 客户端。无论你选择哪个包，命令集都是相同的：

```js
sudo gem install rhc
gem update rhc

```

这将安装客户端并将其更新到最新版本。

安装完成后，你需要设置你的 SSH 密钥并通过运行一个`rhc`设置来与系统进行认证。只需输入你的认证信息，确认密钥的安装，并确认凭证的上传。

然后，您将被要求输入一个命名空间。这将在系统中作为您的标识符，其中还包括形成您部署实例的子域等。

OpenShift 的工作原理是**齿轮**和**卡式件**。

齿轮大致上是有一定计算单元、内存、磁盘、带宽等分配的容器，具有给定的卡式件容量。更大的齿轮性能更好，并且（通常）可以支持更多的卡式件。您可以将您的安装视为一组管理的运行时（卡式件），完全隔离并部署到一个或多个齿轮中。随着您的应用程序需要增长，您将添加齿轮和卡式件。当您添加卡式件时，OpenShift 系统将您的卡式件部署到您的部署中的正确齿轮——某些卡式件只能访问它们自己的齿轮，而其他卡式件可以访问所有齿轮。定价取决于使用的齿轮数量，以及这些齿轮的特性，隐含的卡式件插槽数量。

OpenShift 支持许多类型的开发环境、开源仓库、Web 框架、数据库等等——一个非常丰富的工具生态系统，比我们迄今为止查看的提供商提供的工具要多得多。您甚至可以开发自己的卡式件或使用社区卡式件。

系统使您能够动态地根据齿轮、卡式件或两者同时进行部署扩展。我们将使用的免费层提供三个小型齿轮。

## 安装 Node 应用程序和 MongoDB

在 OpenShift 生态系统中，Node 不是一个特殊的公民（就像 NodeJitsu 一样）或一组固定的进程类型之一（就像 Heroku 一样）。由于这种齿轮和卡式件概念提供的模块化，创建一个具有访问 MongoDB 实例的示例 Node 应用程序只需一行即可完成：

```js
rhc app create MyApp nodejs-0.10 mongodb-2.4
Application Options
-------------------
Domain:   <your namespace>
Cartridges: nodejs-0.10, mongodb-2.4
Gear Size: default
Scaling:  no

Creating application 'MyApp' ...
...
Your application 'myapp' is now available.

URL:    http://yoursub.rhcloud.com/
SSH to:   5366e4cc500446d15300022d@yoursub.rhcloud.com
Git remote: ssh://5366e4cc500446d15300022d@yoursub.rhcloud.com/~/git/myapp.git/
Cloned to: /json_editor/myapp

```

如您所见，您的部署配置强大，允许 SSH 访问和 HTTP 访问，并且作为一个 Git 仓库已准备就绪——如果您查看您的 json-editor 文件夹，一个名为 `myapp/` 的新文件夹已被创建。请继续访问您的 URL。同时提供了如何使用 Git 的完整说明以及如何通过其他方式访问您的应用程序。

我们现在想用我们自己的 `json-editor` 应用程序替换这个示例 Node 应用程序。

## 部署您的应用

当然，我们不希望使用 OpenShift 提供的示例应用程序。而不是重新配置，让我们保持 `myapp/` 中的 `.git` 远程配置，并将以下文件和文件夹复制到我们的 `json-editor/` 文件夹中的 `myapp` 文件夹：

```js
index.html
jsoneditor.css
jsoneditor.js
package.json
server.js
/img
```

这些将覆盖 OpenShift 创建的任何类似文件，同时保留其他文件。确保您已更改目录到 myapp/，因为我们将从那里开始工作。

正如我们在 Heroku 上安装时做的那样，我们需要在启动 Node 服务器时咨询 `process.env` 对象。打开 `server.js` 并转到此行：

```js
}).listen(8081);
```

现在，将行更改为以下内容：

```js
}).listen(process.env.OPENSHIFT_NODEJS_PORT || 8081, process.env.OPENSHIFT_NODEJS_IP || "127.0.0.1");
```

我们现在准备部署我们的应用。更新 Git 中的所有本地文件，提交它们，并推送到 OpenShift：

```js
git add .
git commit -m "first"
git push

```

如果一切顺利，您应该在输出结果的末尾看到以下内容：

```js
remote: Starting MongoDB cartridge
remote: Starting NodeJS cartridge
remote: Starting application 'myapp' ...
remote: -------------------------
remote: Git Post-Receive Result: success
remote: Activation status: success
remote: Deployment completed with status: success

```

我们可以看到 Node 和 MongoDB 都是卡式（不是特殊进程或附加组件），以及一个成功的 post-receive 钩子将自动部署和激活我们的应用程序（这与我们在部署到 Heroku 时看到的情况类似）。

如果有任何问题，我们可以直接访问我们的部署日志。要通过 SSH 连接到您的应用程序（`myapp`），请使用`rhc`工具：

```js
rhc ssh myapp

```

连接后，使用`cd $OPENSHIFT_LOG_DIR`跳转到您的日志目录。您应该看到两个日志：

```js
mongodb.log
nodejs.log

```

这些是标准的 Linux 日志文件，您可以读取或以其他方式操作它们，例如，通过跟踪它们。

您也可以通过`rhc`跟踪日志：

```js
rhc tail

```

### 小贴士

当您远程登录到您的虚拟容器时，您可以通过`cd $OPENSHIFT_REPO_DIR`跳转到应用程序的根目录。

通过`rhc`轻松控制您的应用程序。`rhc`提供了几个命令，通过`rhc app <command>`可用。以下是一些常用命令：

+   **delete**: 这将从服务器删除应用程序

+   **force-stop**: 这将停止所有应用程序进程

+   **reload**: 这将重新加载应用程序的配置

+   **restart**: 这将重新启动应用程序

+   **show**: 这显示了有关应用程序的信息

+   **start**: 这将启动应用程序

+   **stop**: 这将停止应用程序

+   **tidy**: 这将清理日志和`tmp`目录，并整理服务器上的`git`仓库

OpenShift 为那些想要对其部署的应用程序有更多控制权的人提供了一个灵活的选项——为高级用户提供的强大工具。

# 使用 Docker 创建轻量级虚拟容器

来自 Docker 网站的此图像（[`www.docker.com/`](http://www.docker.com/））提供了关于 Docker 团队认为他们的技术如何适应应用程序开发未来的信息和原因：

![使用 Docker 创建轻量级虚拟容器](img/1403OS_02_01.jpg)

上述图像简要描述了我们现在正在经历的应用程序架构的代际转变，同样可以用来描述 Node 的设计方式和原因。

根据网站上的描述，Docker *…是一个开源引擎，它自动化了任何应用程序的部署，作为一个轻量级、便携、自给自足的容器，几乎可以在任何地方运行*。一旦你创建了应用程序的 Docker 镜像，该镜像的运行实例可以在毫秒内启动。是的，就是几毫秒。Docker 让你在几秒钟内创建甚至数百个应用程序部署。

Docker 生态系统有三个主要组件。以下是文档中关于组件的一些信息：

+   **Docker 容器**：Docker 容器就像目录。一个 Docker 容器包含运行应用程序所需的一切。每个容器都是从一个 Docker 镜像创建的。Docker 容器可以运行、启动、停止、移动和删除。每个容器都是一个隔离且安全的应用程序平台。你可以将 Docker 容器视为 Docker 框架的`run`部分。

+   **Docker 镜像**：Docker 镜像是一个模板，例如，一个安装了 Apache 和你的 Web 应用的 Ubuntu 操作系统。Docker 容器是从镜像启动的。Docker 提供了一种简单的方式来构建新的镜像或更新现有的镜像。你可以将 Docker 镜像视为 Docker 框架的`build`部分。

+   **Docker 仓库**：Docker 仓库存储镜像。这些是公共（或私有）存储库，您可以上传或下载镜像。这些镜像可以是您自己创建的，或者您可以使用其他人之前创建的镜像。Docker 仓库允许您构建简单而强大的开发和部署工作流程。你可以将 Docker 仓库视为 Docker 框架的`share`部分。

你可以创建在任何数量的隔离容器中运行的应用程序镜像，如果你愿意，还可以与他人共享这些镜像。将 Node 应用程序组合成许多独立进程的概念与 Docker 背后的哲学自然吻合。Docker 容器是沙箱化的，拥有自己的文件系统等，并且在没有你知识的情况下无法在其主机上执行指令。然而，它们可以向其宿主操作系统暴露一个端口。在本章的后面部分，我们将学习如何使用 Node 将许多独立的虚拟容器链接成一个更大的应用程序。

## 首先，一些 Unix 命令

Docker 是一种新技术，在撰写本文时，它尚未在所有 Unix 版本上可用（尽管团队正在努力在不久的将来实现这一点）。我将安装 CentOS 上的 Docker。Docker 网站([`www.docker.io/`](https://www.docker.io/))定期更新有关如何在您喜欢的 Unix 版本上安装的信息。

了解你的操作系统细节很重要。要找出你的操作系统名称和版本，请使用`cat /etc/*-release`，它应该返回类似以下内容：

```js
CentOS release 6.5 (Final)

```

或者你可以尝试`cat /proc/version`：

```js
Linux version 2.6.32-279.14.1.el6.x86_64 (mockbuild@cb79.bsys.dev.centos.org) (gcc version 4.4.6 20120305 (Red Hat 4.4.6-4) (GCC) ) #1 SMP Tue Nov 6 23:43:09 UTC 2012

```

当你开始创建虚拟机并绑定端口时，有时需要检查你的网络状态。你绝对应该安装一个好的进程查看器，例如**HTOP** ([`hisham.hm/htop/`](http://hisham.hm/htop/))，这样你可以快速扫描/搜索你的打开进程列表。

要获取关于你的盒子网络连接的快速统计列表，请使用`netstat`，它将返回一个类似这样的列表：

![首先，一些 Unix](img/1403OS_02_02.jpg)

你可以看到端口 `8080` 绑定到了 Node 进程 `31878`。你也可以直接查询与端口关联的进程 ID：

```js
> fuser 8080/tcp
8080/tcp:      31878

```

要获取有关进程的更多信息，请输入 `ls -l /proc/31878/exe`：

```js
lrwxrwxrwx 1 root root 0 Oct 9 2013 /proc/31878/exe -> /root/nvm/v0.10.20/bin/node

```

要获取端口使用者的更多信息，尝试使用 `lsof`：

```js
> lsof -i :8080
COMMAND  PID USER  FD  TYPE  DEVICE NODE NAME
node  31878 root  10u IPv4 22570201 TCP *:webcache (LISTEN)

```

紧跟谁在听什么，这将有助于你在阅读本书的过程中。

## 开始使用 Docker

首先，你需要安装 Docker。所有支持的 Linux 发行版的安装说明可以在 [`docs.docker.io/installation/`](http://docs.docker.io/installation/) 找到。

一旦安装了 Docker 服务，你需要启动它：

```js
service docker start

```

然后，停止 Docker 服务：

```js
service docker stop

```

如果一切正常，这个命令应该会告诉你有关你的 Docker 安装的一些信息：

```js
docker info

```

Docker 容器运行着你的应用程序镜像。当然，你也可以自己创建这些镜像，但确实存在一个庞大的现有镜像生态系统。让我们创建一个运行 Express 的 Node 服务器镜像。

### 注意

要搜索 Docker 镜像仓库，请访问 [`index.docker.io/`](https://index.docker.io/)。

首先，我们需要构建一个要运行的应用程序。创建一个文件夹来存放你的应用程序文件。就像所有的 Node 应用程序一样，我们需要创建一个 `package.json` 文件供 npm 解析：

```js
{
  "name": "docker-example",
  "private": true,
  "version": "0.0.0",
  "description": "Example of running a Node app within a CENTOS container",
  "author": "Sandro Pasquali <spasquali@gmail.com>",
  "dependencies": {
    "express": "4.1.1"
  }
}
```

接下来，我们需要一个程序来启动 Express HTTP 服务器。创建以下文件，并将其命名为 `server.js`：

```js
var express = require('express');

var port = 8087;

var app = express();
app.get('/', function (req, res) {
  res.send('You just deployed some Node!\n');
});

app.listen(port);
console.log('Running on http://localhost:' + port);
```

现在，安装并启动你的应用程序：

```js
npm install;
node app.js
// Running on http://localhost:8087

```

现在，你可以将你的浏览器指向端口 `8087` 的主机，并看到 **You just deployed some Node!** 的显示。

现在，我们将探讨如何使用 Docker 将这些文件构建成一个虚拟容器。

## 创建 Dockerfile

我们的目标是描述应用程序执行的环境，以便 Docker 能够在容器中重现该环境。此外，我们还想将应用程序的源文件添加到这个新虚拟化的环境中。Docker 可以作为一个构建器，按照你提供的指令构建应用程序的镜像。

首先，你应该有一个包含应用程序文件的文件夹。这是你的源代码仓库。在这个仓库中，创建一个 `./src` 文件夹。我们很快就会了解到为什么创建这个文件夹。如果这个文件夹是你测试应用程序构建的地方，请删除 `node_modules` 文件夹。

Dockerfile 是构建应用程序的一系列指令。当然，你可以手动构建 Docker 镜像，但很可能会多次重复这些操作。Dockerfile 描述了一个构建过程。你通常会在 Dockerfile 中声明容器将运行的 Linux 版本以及你可能需要安装的任何操作系统——例如 Node 和 npm。此外，你将指出应用程序源代码的位置：位于之前创建的 `./src` 文件夹中。

Dockerfile 始终基于另一个 Docker 镜像构建。通常，您会基于操作系统镜像构建。在这个例子中，我们将使用 CentOS 6.4。我的 Dockerfile 以关于我构建的 Docker 版本和这个镜像将基于哪个镜像构建的注释开始：

```js
# DOCKER-VERSION 0.9.0
FROM  centos:6.4

```

我们现在已经建立了一个在容器中运行的操作系统。现在，我们将简单地列出设置构建环境的典型 Unix 命令。首先，我们需要 Node 和 npm：

```js
# Enable EPEL for Node.js
RUN   rpm -Uvh http://download.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm
# Install Node.js and npm
RUN   yum install -y npm

```

太好了！现在我们的容器知道如何构建 Node 和 npm。现在让我们使用 `ADD` 指令将我们的应用程序打包到容器中的 `./src` 目录：

```js
# Bundle app source
ADD . /src

```

现在应用程序文件已打包到 `./src`，让我们进入该目录并安装应用程序包：

```js
# Install app
RUN cd /src; npm install

```

我们的应用程序现在已安装。注意，在 `app.js` 中，我们正在在端口 `8087` 上公开一个 Express 服务器。容器无法知道这一点，所以我们必须告诉容器在主机系统上设置端口重定向：

```js
EXPOSE 8087

```

最后，容器被指示启动 Node 应用程序：

```js
CMD ["node", "/src/app.js"]

```

就这些了。现在，创建一个名为（确切地）`Dockerfile`的文件，包含前面的指令。我们现在可以使用这个 Dockerfile 来构建一个 Docker 镜像。

## 构建和运行 Docker 镜像

构建 Docker 镜像的命令是 `docker build`。Docker 会在当前目录中查找 Dockerfile，并根据其中包含的指令构建一个镜像。由于我们很可能会重用这个镜像，所以给它一个特殊名称是一个好主意。要给镜像命名，请使用 `–t` 指令，后跟您选择的标签，然后是 Dockerfile 的路径（在这里，是当前目录）：

```js
docker build -t docker/example .

```

当您运行该命令时，您将在终端看到大量输出，因为请求的包正在下载和安装。这可能需要一些时间。幸运的是，Docker 缓存了这些安装——使用此 Dockerfile 或包含相同安装指令的其他 Dockerfile 的下一个构建将会快得多。

如果构建成功，您可以使用 `docker images` 命令列出镜像，输出可能如下所示：

```js
REPOSITORY   TAG   IMAGE ID   CREATED    VIRTUAL SIZE
docker/example latest d8bb295407f1 20 minutes ago   667.8 MB
centos     6.4  539c0211cd76 2 months ago    300.6 MB

```

要删除一个镜像，请使用 `docker rmi <image id>`。

我们的应用程序现在已容器化。我们可以使用以下命令运行它：

```js
docker run -p 49001:8087 -d docker/example

```

`–d` 指令指示 Docker 以分离模式运行此镜像——在后台运行。`49001:8087` 这一部分是必要的，因为它将容器内 Express 服务器监听的 *虚拟* 端口（`8087`）映射到主机机器上的实际端口。

打开您的浏览器，将其指向主机机的端口 `49001`。您应该会看到 **您刚刚部署了一些 Node**！显示。我们之前创建的 Node 应用程序现在正在容器中运行。

为了演示 Docker 的作用，执行之前给出的相同 `run` 指令，但更改端口映射到类似 `49002:8087` 的值。通过更改端口，在另一个浏览器窗口中打开您的应用程序。现在，您有两个相同的应用程序副本在同一主机上以隔离的容器中运行。

### 注意

更多关于运行指令的详细信息可以在 [`docs.docker.io/reference/run/`](http://docs.docker.io/reference/run/) 找到。

要了解更多关于端口重定向的信息，请访问 [`docs.docker.io/use/port_redirection/#port-redirection`](http://docs.docker.io/use/port_redirection/#port-redirection)。

你将想要能够检查正在运行的 Docker 实例。执行此操作的命令是 `docker ps`，它将显示类似以下的信息：

![构建和运行 Docker 镜像](img/1403OS_02_03.jpg)

在这里，我们可以看到我们正在运行的两个容器，包括它们正在运行的信息以及它们的映射方式。要停止一个正在运行的容器，请使用 `docker stop <容器 ID>`。你可以使用 `docker start <容器 ID>` 来重新启动一个已停止的容器，或者当然，启动一个新的容器。这意味着停止一个容器并不会销毁它。要销毁容器，请使用 `docker rm <容器 ID>`。

### 小贴士

要获取 Docker 命令的完整列表，只需在终端中输入 `docker`。

# 摘要

在本章中，你学习了如何在本地和 *云中* 创建 Node 服务器和应用程序。通过使用 Node 和 MongoDB 在三个不同的 PaaS 提供商上部署了一个简单的文档编辑应用程序，你对寻求扩展其应用程序的 Node 开发者可用的资源有了初步的了解。你被介绍到了 Docker，它提供了一种强大的新容器化技术，使我们能够制作许多便宜的应用程序副本；只要有 Linux，就有 Docker 的部署目标。

在下一章中，我们将通过更详细地探讨 Node 可以如何垂直和水平扩展——跨核心和跨多台机器，将这些关于扩展的简单想法推向更远和更深。
