# 第二章：Node.js 入门

在 Node.js 网站 ([`nodejs.org/`](http://nodejs.org/)) 上给出的 Node.js 定义如下：

> Node.js 是一个基于 Chrome 的 JavaScript 运行时构建的平台，用于轻松构建快速、可扩展的网络应用程序。Node.js 使用事件驱动的、非阻塞的 I/O 模型，使其轻量级且高效，非常适合运行在分布式设备上的数据密集型实时应用程序。

对我们来说重要的是，作为平台的一部分，Node.js 提供了一个可扩展且高性能的 Web 应用程序开发框架，允许使用 JavaScript 进行编程。

我们中的许多人是在构建网站或 Web 应用程序以进行 DOM 操作、AJAX 和相关内容时接触到 JavaScript 的。但 JavaScript 远不止于此。就像 C、Java、Python 等一样，JavaScript 也是一个完整的编程语言。在所有浏览器中，JavaScript 都在浏览器的上下文中在虚拟机 (VM) 中执行。但它也可以在另一个上下文中执行——就像 Node.js 后端的情况一样——而不需要浏览器。

Node.js 使用 Google Chrome 的 JavaScript VM 在浏览器之外执行 JavaScript 应用程序，在服务器上。除了这个运行时环境之外，Node.js 还提供了一组模块库，为构建网络应用程序提供了一个框架。Node.js 不是一个像 Apache HTTP 服务器那样的网络服务器，也不是像 Tomcat 那样的应用程序服务器；但作为其模块库的一部分，Node.js 也提供了一个 HTTP 服务器，可以用来构建 Web 应用程序。

除了将 JavaScript 作为应用程序的编程语言之外，Node.js（以及大多数 Node.js 模块和应用）与传统服务器和应用程序区分开来的一点是异步事件驱动开发模型，我们将在后面的章节中看到这一点。

# Node.js 的起源

这并不是 JavaScript 首次被用于服务器端编程。Netscape 在 1996 年推出了 Netscape Enterprise Server，允许使用 JavaScript 进行服务器端编程。从那时起，许多服务器，如 **RingoJS** ([`ringojs.org/`](http://ringojs.org/))、**Persevere** ([`www.persvr.org/`](http://www.persvr.org/))、基于 Mozilla Rhino 的服务器以及其他服务器，都尝试效仿。

这些服务器没有被认真对待的主要原因之一是它们使用的 JavaScript 虚拟机 (VM) 的性能可怜。浏览器中的 JavaScript 性能也不是很好。直到 Google 推出了其 Chrome 网络浏览器，这种情况才有所改变。

在其发布时，Chrome 的 JavaScript VM，称为 **V8**，几乎比其他任何 JavaScript VM 快 10-20 倍，并且从那时起一直是最快的。

Node.js 是基于这个虚拟机在 2008 年由 Ryan Dahl 开发的。他想要构建一个能够支持并赋能实时交互式网络应用，如 Gmail 的服务器。但 Node.js 并不是他第一个构建的服务器。Ryan 构建了基于 Ruby 和 C 的 Ebb 服务器，但意识到它并没有达到他期望的运行速度。随后，他进行了一系列构建多个小型服务器的实验。

借助从实验和各种平台研究中获得的知识，他决定开发一个事件驱动或异步服务器。2008 年 1 月，他提出了基于 JavaScript 构建小型服务器的想法。他倾向于选择 JavaScript，因为它独立于操作系统，并且没有 I/O API。他辞去了工作，用 6 个月的时间专注于 Node.js。2009 年 11 月，他在 JSConf 上展示了 Node.js，并从那时起为 Joyent 工作。最初，Node.js 只能在基于 Unix 的系统上运行；后来，它也增加了对 Windows 操作系统的支持。

# 为什么选择 Node.js

Node.js 是一个新兴的平台，仍在不断发展（甚至还没有发布 1.0 版本），但即使在其初期，它可能也是 Web 上最受欢迎的平台之一。它已经为大量流行的服务提供动力。让我们看看是什么让 Node.js 如此诱人和受欢迎。

## JavaScript 到处都是

Node.js 的首要优势是 JavaScript。如果你知道并经常用 JavaScript 编码，你已经了解了 Node.js 的大部分内容；剩下的学习内容可以看作是 API 和最佳实践。

基于 Google Chrome 的 V8 JavaScript 引擎构建的 Node.js 允许使用 JavaScript 编写整个应用程序。我们已经在用 JavaScript 编写前端；有了 Node.js，我们也可以用同样的语言编写后端，这是我们磨练技能并深深喜爱的语言。它让每个前端开发者都无需学习另一种语言或依赖其他开发者来暴露应用程序所需的 RESTful API。

## 事件驱动设计

Node.js 是围绕事件和回调进行设计的。作为一个 JavaScript 开发者，你早已熟悉监听事件和使用回调的概念。Node.js 将这种理念融入到平台的每一个方面。无论是服务器请求处理、I/O 还是数据库交互，Node.js 中的所有操作理想上都将由一个事件监听器附加的回调来处理。

这带我们来到了 Node.js 背后最重要的概念之一，那就是 *事件循环*。我喜欢 Dan York ([`code.danyork.com`](http://code.danyork.com)) 用快餐店类比来解释基于事件循环的系统。

考虑一家餐厅，你走到收银台，下单，然后等待你的食物准备好。在这种情况下，收银员在你下单之前不能服务其他顾客，队列被阻塞。如果餐厅有大量的顾客涌入并需要扩展，他们就必须投资雇佣更多的收银员，创建更多的收银台等等。这类似于传统的多线程模型。

相比之下，让我们看看许多其他餐厅使用的模型。在这种情况下，你走到收银台并下单（他/她将其交给厨房）；然后他/她接受你的付款并给你一个凭证。然后你走到一边，收银员继续服务下一个顾客。当你的订单准备好时，厨房服务器通过叫你的名字或闪烁你的凭证号码来宣布这一点，然后你走过去取你的订单。这种面向事件的方法优化了收银员的工作，并让你在旁边等待，直到你的工作完成，从而释放相关资源为其他人提供服务。

在 Node.js 中，服务器就是收银台，所有的处理程序都是厨房工作人员。服务器接受一个请求并将其分配给一个处理程序。然后它继续接受其他请求。当请求被处理并且结果就绪时，响应被排队在服务器上，并在到达队列前端时发送回客户端。

与传统的启动服务器线程或进程的方法（这类似于增加更多的收银员）相比，这种方法更高效，因为启动的工作者有专门的责任。这比复制整个服务器要轻便和便宜得多。

在接下来的章节中，我们将看到我们如何将处理程序或工作者注册到服务器上以处理某些请求，而服务器所做的只是将这些请求委派给这些工作者。

事件驱动设计的优势在于我们设计的所有内容都是非阻塞的。“你不等待我，我召唤你”这句箴言使我们免受等待请求完成的痛苦。它释放了本应花费在等待请求上的系统资源，以便它们可以被用于队列中的其他任务。这使得 Node.js 应用程序能够提供非常高的性能和非常高的负载处理能力。

Node.js 是一个模块化框架，从底层开始就拥有一个现代的模块系统。Node.js 中的所有内容都是构建在 V8 JavaScript 引擎中运行的模块。平台上的每个功能都是通过模块提供的。这使平台保持精简，只引入所需的内容。拥有本地的模块系统也有助于保持我们的应用程序模块化。

在过去几年中，JavaScript 已经成为最广泛使用的语言之一，并且拥有一个充满活力的社区。Node.js 为开发者提供了一个良好的平台，帮助他们使用 JavaScript 开发端到端应用程序。Node.js 还引入了许多革命性的概念，例如始终异步、非阻塞 I/O、面向事件的服务器等。这导致了一个非常活跃、庞大且活跃的社区。新的模块不断涌现，社区提供积极的支持，非常有帮助。为 Node.js 构建的大多数流行模块和框架通常来自社区，并且大多是开源的。

## 企业支持

在过去几年中，许多公司都在 Node.js 上投入了大量资金。从 Ryan Dahl 的雇主 Joyent 到 Mojito 框架的创造者（互联网巨头 Yahoo!），许多公司都围绕 Node.js 构建了产品、平台、框架和服务。这种企业承诺确保了稳定的未来。

# 如何获取 Node.js

由于 Node.js 的流行，它在任何操作系统上运行都非常容易。你可以访问[`nodejs.org/`](http://nodejs.org/)下载适合你操作系统的适当版本。

### 小贴士

尽管 Node.js 可以在任何操作系统上运行，但由于它来自*nix 背景，许多模块可能只能在 Linux 或其他 Unix 系统上运行；因此，如果你手头有这样一个系统，最好使用它。

如果你使用 Linux，在大多数情况下，你应该能够使用你发行版的包管理器安装 Node.js。由于这些信息不断变化，我只会指出位置。你可以在以下位置找到使用包管理器安装 Node.js 的说明：

[`github.com/joyent/node/wiki/Installing-Node.js-via-package-manager`](https://github.com/joyent/node/wiki/Installing-Node.js-via-package-manager)

如果你使用 Mac OS 或 Windows，你应该知道 Node.js 现在为这些平台提供了安装程序，这是推荐的安装方法。你也可以使用源代码进行安装。为了避免在这里重复该过程，这个过程也可能再次发生变化，我建议你遵循 Node.js 维基百科上的官方安装说明，GitHub（[`github.com/joyent/node/wiki/Installation`](https://github.com/joyent/node/wiki/Installation)）。

# Node.js 包管理器（npm）

如果你使用 Node.js 网站上的安装程序安装了 Node.js，那么你将已经安装了 npm。

此外，如果你按照说明从源代码构建，你可能会已经安装了 npm。如果是这样的话，非常好！如果没有，请现在就安装。为此，我建议你遵循 npm 安装文档中提到的说明（[`github.com/isaacs/npm/`](https://github.com/isaacs/npm/)）。

你可以通过输入以下命令来检查你是否已安装 npm：

```js
$ npm -v

```

这应该会显示已安装的 npm 版本。

对于那些还在疑惑 npm 是什么以及为什么你需要为 Node.js 安装包管理器的人来说，npm 正是其名字所描述的那样；它为 Node.js 提供了一个基础设施，用于分发和管理包。正如我之前所说的，Node.js 非常模块化。Node.js 应用程序使用许多模块和第三方包来构建，因为 npm 为我们提供了添加和管理第三方依赖项的简单方法。我们稍后会看到它的更多用途。

# Hello World with Node.js

在这里，必做的 Hello World 示例使用了 Node.js。在一个名为`helloworld.js`的文件中写下以下行，并保存：

```js
console.log("Hello World");
```

现在要运行它，请执行以下命令：

```js
node helloworld.js

```

这应该在控制台打印出**Hello World**。所有 JavaScript 开发者都会立即认出，这些是我们开发 Web 应用程序时在控制台上打印任何内容的步骤。

发生的事情是，Node.js 在 JavaScript 虚拟机中加载 JavaScript 文件，为其执行提供环境，虚拟机解释脚本。当它遇到`console.log`时，它会检查环境中的控制台，在这种情况下是`STDOUT`，并将**Hello World**写入其中。

但我们在这里是为了开发 Web 应用程序，对吧？所以让我们向 Web 问好！

# Hello Web

让我们创建一个非常简单的 Web 应用程序，它会对用户说“你好”。在文件中写下以下代码，并将其命名为`helloweb.js`：

```js
var http = require("http");

http.createServer(function(request, response) {
  response.writeHead(200, {"Content-Type": "text/html"});
  response.write("<html>");
  response.write("<head><title>Node.js</title></head>");
  response.write("<body>Hello Web</body>");
  response.write("</html>");
  response.end();
}).listen(9999);
```

要运行前面的代码片段，请在 Node.js 中执行`helloweb.js`：

```js
node helloweb.js

```

然后在浏览器中打开`http://localhost:9999/`。你应该会看到一个写着**Hello Web**的页面。这里有很多事情在进行中！所以让我们一步步地看代码，了解发生了什么。

代码的第一行向我们介绍了 Node.js 的一个基本构建块，即模块系统。Node.js 有一个基于 CommonJS 的非常简单的模块系统。那些熟悉使用 RequireJS 进行前端开发并使用**异步模块定义（AMD）**的人会立即联想到这一点。Node.js 中的所有功能都作为模块构建，你需要使用`require`在代码中导入它。Node.js 有几个以二进制形式编译的模块，称为核心模块，HTTP 就是其中之一。我们也可以使用`require`创建和包含我们自己的自定义或第三方模块。在文件模块的情况下，文件和模块之间存在一对一的映射；因此，我们将在自己的文件中编写每个模块。我们稍后会看到更多关于编写我们自己的模块的内容。

```js
var http = require("http");
```

通过这个语句，Node.js 将加载核心 HTTP 模块，并且它将在名为`http`的变量中可用。下一个任务是使用 HTTP 模块创建一个服务器。这是通过模块中的`createServer`方法完成的。`createServer`方法接受`requestListener`。

```js
http.createServer([requestListener]);
```

`requestListener` 是一个处理传入请求的函数。在我们的例子中，这个函数是直接传递的。就像浏览器中的 JavaScript 一样，Node.js 也运行在一个单独的进程和线程中。这与传统的应用服务器不同，传统的应用服务器会为新的请求创建一个新的线程或进程。因此，为了扩展和处理多个请求，Node.js 使用异步事件处理。所以每个传入的请求都会触发一个事件，然后由事件处理器异步处理。这就是前面章节中解释的事件循环机制。

```js
http.createServer(function(request, response) {
  response.writeHead(200, {"Content-Type": "text/html"});
  response.write("<html>");
  response.write("<head><title>Node.js JS</title></head>");
  response.write("<body>Hello Web</body>");
  response.write("</html>");
  response.end();
});
```

### 小贴士

**下载示例代码**

您可以从您在 [`www.packtpub.com`](http://www.packtpub.com) 的账户下载您购买的所有 Packt 书籍的示例代码文件。如果您在其他地方购买了这本书，您可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 并注册，以便将文件直接通过电子邮件发送给您。

`createServer` 的工作方式与 JavaScript 中的任何事件处理器类似。在这个例子中，事件是接收一个请求来服务。正如我们所看到的，`requestListener` 接受两个参数，`request` 和 `response`。`request` 对象是 `http.ServerRequest` 的一个实例，它将包含有关请求的所有信息，例如 URL、方法、头和数据。

`response` 对象是 `ServerResponse` 的一个实例，它实现了 `WritableStream` 接口。它暴露了各种方法来将响应写入客户端；目前我们最感兴趣的方法是 `writeHead`、`write` 和 `end`。让我们首先看看 `writeHead`：

```js
response.writeHead(statusCode, [reasonPhrase], [headers]);
```

在这里，`statusCode` 是 HTTP 响应码，`reasonPhrase` 是可选的供人类阅读的响应短语，`headers` 是包含要发送在响应中的头的对象。这个函数应该只调用一次，在调用 `response.end` 之前。如果我们在这个之前调用 `response.write` 或 `response.end`，隐式/可变的头将被计算，并且下面的函数将自动被调用：

```js
response.writeHead(200, {"Content-Type": "text/html"});
```

在这次调用中，我们将状态码设置为 `200`，即 HTTP OK，并且我们只设置了 `Content-Type` 头为 `text/html`。这里的方法是 `response.write`，它用于将响应内容写入客户端。对这个方法的调用如下：

```js
response.write(chunk, [encoding]);
```

在这次调用中，`chunk` 是要写入的内容，`encoding` 是要使用的编码。如果 `chunk` 是一个字符串并且没有指定编码，默认将使用 UTF-8。

```js
response.write("<html>");
response.write("<head><title>Node.js JS</title></head>");
response.write("<body>Hello Web</body>");
response.write("</html>");
```

在上述代码中，第一次调用 `write` 时，Node.js 会发送响应头和响应体的一个片段。但 `write` 方法可以被多次调用。Node.js 会假设我们正在流式传输数据，并且会在每次调用时发送数据块。最后对响应的调用是用来告诉 Node.js 我们已经完成。这正是 `response.end` 所做的。

```js
response.end([data], [encoding]);
```

`response.end` 向服务器发出信号，表示所有响应头和主体内容都已发送，服务器应认为这条消息已完整。我们*必须*为每条消息调用此方法。

```js
response.end();
```

在我们的情况下，我们调用 `response.end` 而不带任何可选参数。如果我们确实传递了参数，那么它等同于调用 `response.write` 并传递参数，然后调用 `response.end`。我更喜欢将它们分开，并且明确地这样做。

最后，我们需要告诉 HTTP 服务器它应该监听哪个端口。在这种情况下，我们告诉它监听端口 9999。

```js
listen(9999);
```

# 请求路由

几乎任何 Web 应用程序都服务于多个资源。因此，现在我们知道如何使用 HTTP 服务器提供内容；但我们如何处理多个资源呢？路由就是关键。我们需要理解传入的请求并将其映射到适当的请求处理器。这比之前的例子要复杂一些，所以我们将逐步构建它，每一步都进行改进。

为了演示请求的路由，让我们构建一个简单的应用程序，该应用程序在 `/start` 和 `/finish` 路径上提供两个资源，分别显示 **Hello** 和 **Goodbye**。为了简化代码，我们将提供纯文本。因此，在所有其他事情之前，让我们先看看代码：

```js
var http = require("http");
var url = require("url");

function onRequest(request, response) {
  var pathname = url.parse(request.url).pathname;
  console.log("Request for " + pathname + " received.");
  if(pathname === "/start"){
    response.writeHead(200, {"Content-Type": "text/plain"});
    response.write("Hello");
    response.end();
  }else if(pathname === "/finish"){
    response.writeHead(200, {"Content-Type": "text/plain"});
    response.write("Goodbye");
    response.end();
  }else{
    response.writeHead(404, {"Content-Type": "text/plain"});
    response.end("404 Not Found");
  }
}

http.createServer(onRequest).listen(9999);
console.log("Server has started.");
```

将之前的代码片段保存到名为 `routing.js` 的文件中，并按以下方式执行：

```js
node routing.js

```

现在，当我们访问 `http://localhost:9999/start` 时，我们将在浏览器中看到 **Hello**。同样，当我们访问 `http://localhost:9999/finish` 时，我们将看到一个显示 **Goodbye** 的消息。如果我们尝试访问任何其他路径，我们将得到一个 HTTP 404 或未找到错误。现在让我们尝试理解在这个例子中引入的新内容。

为了路由一个请求，我们首先需要解析 URL；为此，我们将引入另一个内置模块，称为 `url`。当使用 `url` 模块解析 URL 字符串时，它返回一个 URL 实例。在这种情况下，我们感兴趣的是路径名。

```js
var pathname = url.parse(request.url).pathname;
```

在上一行代码中，我们正在传递请求中的 `url` 字符串，并使用 `url` 模块对其进行解析，以获取 `pathname`。下一步是根据访问的路径发送适当的响应：

```js
  if(pathname === "/start"){
    response.writeHead(200, {"Content-Type": "text/plain"});
    response.write("Hello");
    response.end();
  }else if(pathname === "/finish"){
    response.writeHead(200, {"Content-Type": "text/plain"});
    response.write("Goodbye");
    response.end();
  }
```

在这里，我们正在将路径名与我们期望处理的路径名进行比较，并相应地发送适当的响应。那么，对于我们没有处理的请求会发生什么呢？这就是 if-else-if 阶梯的最后一部分所做的事情。它发送一个 HTTP 404 错误。

```js
    response.writeHead(404, {"Content-Type": "text/plain"});
    response.end("404 Not Found");
```

现在，让我们考虑扩展这个应用程序。要处理更多的路径，我们必须添加更多的 if-else 条件。但这看起来并不干净，难以阅读，并且在执行上非常低效。想想 if-else 层级结构的最后一步处理的路由；这个过程仍然必须通过整个层级结构，检查每个条件。此外，向其中添加新路由将需要我们遍历并编辑这个 if-else 层级结构，这至少会让人困惑，也可能导致错误、打字错误，以及无意中修改现有路由的高度可能性。因此，让我们通过将处理程序放入按路径映射的对象中，并也提供一个 API 来扩展它，使其变得更加干净。因此，让我们将我们的代码修改如下：

```js
var http = require("http");
var url = require("url");

var route = { 
  routes : {}, 
  for: function(path, handler){
    this.routes[path] = handler;
  }
};

  route.for("/start", function(request, response){
    response.writeHead(200, {"Content-Type": "text/plain"});
    response.write("Hello");
    response.end();
  });

  route.for("/finish", function(request, response){
    response.writeHead(200, {"Content-Type": "text/plain"});
    response.write("Goodbye");
    response.end();
  });

function onRequest(request, response) {
  var pathname = url.parse(request.url).pathname;
  console.log("Request for " + pathname + " received.");
  if(typeof route.routes[pathname] ==='function'){
    route.routespathname;
  }else{
    response.writeHead(404, {"Content-Type": "text/plain"});
    response.end("404 Not Found");
  }
}

http.createServer(onRequest).listen(9999);
console.log("Server has started.");
```

要运行此代码片段，请使用以下命令使用 Node.js 执行文件：

```js
node resources.js

```

应用程序的功能将与上一个示例的结果相同。当我们访问`/start`或`/finish`时，它将分别为前者响应**Hello**，为后者响应**Goodbye**。尝试访问任何其他路径时，我们将收到 HTTP 404 消息。

我们在这里所做的改变是，我们放弃了 if-else-if 的层级结构，转而采用了一种干净且高效的设计方法。在这种方法中，我们不需要与现有的路由打转，可以通过从任何模块中调用`route.for`方法来添加新的路由。路由有一个指向`handler`函数的路径映射，并且还有一个`on`方法来添加新的路由。

```js
var route = {
  routes : {},
  for: function(path, handler){
    this.routes[path] = handler;
  }
}

route.on("/start", function(request, response){
    response.writeHead(200, {"Content-Type": "text/plain"});
    response.write("Hello");
    response.end();
});

route.on("/finish", function(request, response){
    response.writeHead(200, {"Content-Type": "text/plain"});
    response.write("Goodbye");
    response.end();
});
```

在这里，我们为路径`/start`和`/finish`添加了两个新的处理程序。处理程序的签名与主请求处理程序类似。我们期望处理程序能够获取请求和响应，这样处理程序就有处理请求和发送响应所需的一切。

```js
if(typeof(route.routes[pathname])==='function')
```

在 if 条件中，我们检查路径名对应的路由是否存在，以及它是否是一个函数。如果我们找到了请求路径的处理程序，我们将执行`handler`函数，并将请求和响应传递给它。

```js
route.routespathname;
```

如果未找到，我们将以 HTTP 404 错误响应。现在，要添加新的路径，我们可以通过调用带有路径及其处理程序的`route.on`方法来注册它。

```js
route.on("/newpath", function(request, response){ 
    response.writeHead(200, {"Content-Type": "text/plain"});
    response.write("new response");
    response.end();
});
```

# HTTP 方法

HTTP 不仅关乎路径，我们还要考虑 HTTP 方法。在本节中，我们将增强我们的应用程序以处理不同的 HTTP 方法：`GET`、`POST`、`PUT`和`DEL` `TE`。

作为这一步骤的第一步，我们将添加为不同的方法添加不同处理程序的能力。我们将在 resources.js 中的映射中添加方法，这是一个小的改动。这在上面的代码片段中显示：

```js
var http = require("http");
var url = require("url");
var route = {
  routes : {}, 
  for: function(method, path, handler){
    this.routes[method + path] = handler;
  }
}

  route.for("GET", "/start", function(request, response){
    response.writeHead(200, {"Content-Type": "text/plain"});
    response.write("Hello");
    response.end();
  });

  route.for("GET", "/finish", function(request, response){
    response.writeHead(200, {"Content-Type": "text/plain"});
    response.write("Goodbye");
    response.end();
  });

function onRequest(request, response) {
  var pathname = url.parse(request.url).pathname;
  console.log("Request for " + request.method + pathname +" received.");
  if(typeof(route.routes[request.method +pathname])==='function'){
    route.routesrequest.method + pathname;
  }else{
    response.writeHead(404, {"Content-Type": "text/plain"});
    response.end("404 Not Found");
  }
}

http.createServer(onRequest).listen(9999);
console.log("Server has started.");
```

保存文件并使用 Node.js 执行。功能仍然保持不变，但我们将能够使用不同的处理程序处理不同的方法。让我们创建一个新的处理程序来在`POST`上回显传入的数据。

```js
  route.on("POST", "/echo", function(request, response){
    var incoming = "";
    request.on('data', function(chunk) {
      incoming += chunk.toString();
    });

    request.on('end', function(){
      response.writeHead(200, {"Content-Type": "text/plain"});
      response.write(incoming);
      response.end();
    });
  });
```

在这里，我们为`/echo`路径上的`POST`请求添加了一个新的处理器。我们再次看到了 Node.js 事件驱动方法的使用，这次是在处理通过`POST`传入的数据。由于`request`是一个事件发射器，我们为每个任务（处理传入的数据块和完成请求处理）都附加了一个事件处理器。

```js
    request.on('data', function(chunk) {
      incoming += chunk.toString();
    });
```

在之前的代码中，我们在请求上添加了一个监听器来处理传入的数据块。在这种情况下，我们只是累积传入的数据。

```js
    request.on('end', function(){
      response.writeHead(200, {"Content-Type": "text/plain"});
      response.write(incoming);
      response.end();
    });
```

在`end`事件处理器中，一旦接收到所有通过`POST`发送的数据，它将被调用。这是我们完成接收所有数据的时刻。为了构建一个回显服务，我们将发送回所有累积的数据。现在，我们将创建一个表单来向此处理器提交请求。

```js
  route.on("GET", "/echo", function(request, response){
    var body = '<html>' + 
    '<head><title>Node.js Echo</title></head>' + 
    '<body>' + 
    '<form method="POST">' + 
    '<input type="text" name="msg"/>' + 
    '<input type="submit" value="echo"/>' + 
    '</form>' + 
    '</body></html>';

    response.writeHead(200, {"Content-Type": "text/html"});
    response.write(body);
    response.end();
  });
```

我们将在相同的路径（`/echo`）上添加一个事件处理器，但这次是为了处理`GET`请求。在处理器中，我们将返回一个包含表单的 HTML 页面，该表单将提交到相同的路径。

将这两个处理器添加到我们的`route-handlers.js`中，并使用 Node.js 执行它。要打开我们的表单，请访问`http://localhost:9999/echo`；然后，要触发我们的处理器，请在表单的文本框中输入一条消息，并点击**回显**按钮。这将提交表单的内容，浏览器中我们将看到`msg=<your text>`。

# 创建我们自己的模块

模块是 Node.js 应用程序的基本构建块。随着我们所有的更改，我们的文件正变得有些杂乱；此外，我们将基础设施（服务器和路由器）与应用程序逻辑（处理器）放在了同一个地方。如前所述，Node.js 建立在 CommonJS 模块系统之上。在 Node.js 中，模块和文件有一对一的关系。让我们将服务器和路由器移动到它们自己的模块中。将以下内容保存到一个名为`server.js`的文件中：

```js
var http = require("http");
var url = require("url");

function onRequest(request, response) {
  var pathname = url.parse(request.url).pathname;
  console.log("Request for " + request.method + pathname +" received.");
  if(typeof(routes[request.method + pathname])==='function'){
    routesrequest.method + pathname;
  }
  else{
    response.writeHead(404, {"Content-Type": "text/plain"});
    response.end("404 Not Found");
  }
}

var routes = {};

exports.forRoute = function(method, path, handler){
  routes[method + path] = handler;
};

exports.start = function(){
  http.createServer(onRequest).listen(9999);
  console.log("Server has started.");
};
```

代码的逻辑方面大部分保持不变，但我们做了一些非常微妙的结构变化。第一个变化是我们将路由从`route`对象中移除。在文件中声明的任何变量都在模块内部可用，且不可从外部访问。

```js
  if(typeof(routes[request.method + pathname])==='function'){
    routesrequest.method + pathname;
  }
```

由于`route`对象已不存在，我们现在可以直接访问模块内的路由，而不是通过`route`对象。

另一个更明显的改变是`exports`。由于模块内部没有任何内容可供外部访问，我们必须将想要公开的方法/对象添加到隐式的`exports`对象中。理想情况下，你应该只公开与模块最终用户相关的那些方法。

```js
exports.forRoute = function(method, path, handler){
  routes[method + path] = handler;
};

exports.start = function(){
  http.createServer(onRequest).listen(9999);
  console.log("Server has started.");
}
```

我们从模块中公开了两个方法：`forRoute`方法（原本是`route`对象中的`on`方法），以及包装启动 HTTP 服务器的`start`方法。我们还把应用程序逻辑移动到名为`app.js`的单独模块中，如下代码片段所示：

```js
var server = require("./server.js");
server.forRoute("GET", "/start", function(request, response){
  response.writeHead(200, {"Content-Type": "text/plain"});
  response.write("Hello");
  response.end();
});

server.forRoute("GET", "/finish", function(request, response){
  response.writeHead(200, {"Content-Type": "text/plain"});
  response.write("Goodbye");
  response.end();
});

server.forRoute("POST", "/echo", function(request, response){

  var incoming = "";

  request.on('data', function(chunk) {
    incoming += chunk.toString();
  });

  request.on('end', function(){
    response.writeHead(200, {"Content-Type": "text/plain"});
    response.write(incoming);
    response.end();
  });
});

server.forRoute("GET", "/echo", function(request, response){
  var body = '<html>' + 
  '<head><title>Node.js Echo</title></head>' + 
  '<body>' + 
  '<form method="POST">' + 
  '<input type="text" name="msg"/>' + 
  '<input type="submit" value="echo"/>' + 
  '</form>' + 
  '</body></html>';

  response.writeHead(200, {"Content-Type": "text/html"});
  response.write(body);
  response.end();
});

server.start();
```

再次强调，逻辑方面保持不变；变化仅在于结构。

```js
var server = require("./server.js");
```

上一行，也是前一个代码片段中的第一行，展示了我们的服务器模块是如何被加载的。这类似于加载核心模块如 HTTP 或 URL，但在这里我们加载了模块并传递了其文件名。由这个`require`方法创建的对象，即`server`对象，将有两个将被公开的方法：`forRoute`和`start`。

接下来，我们将所有对`route.on`的调用替换为`server.forRoute`方法。最后，我们调用`server.start`方法来启动 HTTP 服务器。

```js
server.start();
```

# 文件服务

我们可以看到，使用我们当前的基础设施编写 HTML 页面并不直观或容易。将 HTML 以字符串形式写入 JS 代码并不有趣。我们希望从 HTML 文件中服务 HTML 内容。我们将从需要从磁盘中的文件读取的`app`模块中的两个模块开始：

```js
var path = require('path');
var fs = require('fs');
```

第一个，`path`，是我们用来处理路径的模块，而`fs`是用于与文件系统交互的模块。下一步是获取应用程序根路径。

```js
var root = __dirname;
```

`__dirname`是由 Node.js 管理的变量，包含 Node.js 应用程序脚本的绝对路径。现在，我们添加一个将执行读取文件并发送到浏览器的大量工作的方法。将此方法添加到`app.js`中：

```js
var serveStatic = function(response, file){
  var fileToServe = path.join(root, file);
  var stream = fs.createReadStream(fileToServe);

  stream.on('data', function(chunk){
    response.write(chunk);
  });

  stream.on('end', function(){
    response.end();
  });
}
```

我们创建的`serveStatic`方法接受两个参数，即 HTTP 响应对象和要服务的文件路径。

```js
  var fileToServe = path.join(root, file);
```

我们将文件路径追加到我们的根路径，以构建要服务的文件的绝对路径。这里我们隐式地假设所有要服务的文件都是相对于 Node.js 应用程序根目录的；这将防止错误地由应用程序之外的文件提供服务。Node.js 以流的形式处理 I/O。我们可以看到这类似于它处理传入的`POST`数据的方式。

```js
  var stream = fs.createReadStream(fileToServe);
```

我们使用`fs`模块中的`createReadStream`创建一个从文件读取的流。这个流再次展示了 Node.js 的非阻塞、异步和事件驱动方法。

```js
 stream.on('data', function(chunk){
    response.write(chunk);
  });
```

流作为一个事件发射器，当流上有新数据时触发`'data'`事件。这允许应用程序继续进行其他处理活动，而无需等待数据被读取。我们在这里所做的是，将每次`read`接收到的数据写入`response`流。

```js
  stream.on('end', function(){
    response.end();
  });
```

一旦读取了文件中的所有数据，并且流接收到 EOF（文件结束标志），它将触发`'end'`事件。我们也会在我们的响应上调用`end`。最后，我们将修改`GET`处理器`"/echo"`以从名为`echo.html`的文件中提供服务，并将要服务的 HTML 内容写入该文件。

```js
server.forRoute("GET", "/echo", function(request, response){
    serveStatic(response, "echo.html");
});
```

我们已经移除了所有构建`response`字符串的代码，用对`serveStatic`的调用替换了它，以服务`echo.html`。

```js
<html>
  <head>
    <title>Node.js Echo</title>
  </head>
  <body>
    <form method="POST">
      <input type="text" name="msg"/>
      <input type="submit" value="echo"/>
    </form>
  </body>
</html>
```

我们之前作为字符串构建的内容现在被写入这个文件。一旦我们做出这些更改并使用 Node.js 运行`app.js`，你应该能够在`http://localhost:9999/echo`上看到表单并保留其原始功能。

熟悉类 Unix 操作系统的用户会发现，我们刚刚实现的功能，即从一个流读取并写入另一个流，也可以通过使用管道（`|`）来实现。Node.js 提供了一个高级函数来完成完全相同的功能，这并不令人惊讶。

使用 Node 中`stream`提供的`pipe`方法，我们可以按如下方式修改`serveStatic`方法：

```js
var serveStatic = function(response, file){
  var fileToServe = path.join(root, file);
  var stream = fs.createReadStream(fileToServe);
  stream.pipe(response);
}
```

在这里，我们使用`stream.pipe(response)`替换了数据和结束事件处理器。

# 第三方模块和 Express JS

现在我们已经构建了自己的路由器并了解了 Node.js 的基础知识，是时候介绍 Node.js 最广泛使用的框架之一——Express ([`expressjs.com`](http://expressjs.com))了。Node.js 提供了构建 Web 应用程序的基础组件，但处理的东西太多。这就是 Web 框架的作用所在。有相当多的 Web 框架为 Node 提供了构建应用程序的高级抽象。您可以在以下列表中看到其中大部分：

[`github.com/joyent/node/wiki/Modules#wiki-web-frameworks-full`](https://github.com/joyent/node/wiki/Modules#wiki-web-frameworks-full)

Express 是一个基于 Connect 中间件的 Node Web 应用程序框架，它提供了许多辅助工具和结构化方面来构建我们的 Web 应用程序。

要开始使用 Express 框架，我们需要使用 npm，Node.js 包管理器，来安装它。

```js
sudo npm install -g express

```

之前的命令将 Express 作为一个全局（`-g`）模块安装，并使`express`作为一个命令可用。让我们创建一个 Express 应用：

```js
express hello-express

```

这将创建一个名为`hello-express`的文件夹，其中包含一些文件和文件夹。让我们看看并理解这些文件。

首先需要理解的是`package.json`文件。这是定义 Node.js 应用程序包的文件。它包含应用程序元数据，如名称、描述和版本。更重要的是，它列出了模块依赖项。依赖项列表被 npm 用于下载所需的模块。

```js
{
  "name": "application-name",
  "version": "0.0.1",
  "private": true,
  "scripts": {
    "start": "node app"
  },
  "dependencies": {
    "express": "3.1.0",
    "jade": "*"
  }
}
```

在你的`package.json`中最重要的东西是`name`和`version`字段。这些字段是必需的，并且它们共同构成了包特定版本的唯一标识符。首先，让我们将我们的包名称更改为`hello-express`。`version`字段由以下内容组成（按相同顺序）：

+   一个数字（主要版本）

+   一个点

+   一个数字（次要版本）

+   一个点

+   一个数字（补丁版本）

+   可选：一个连字符，后跟一个数字（构建）

+   可选：一大堆几乎任何非空白字符的集合（标签）

如果我们将 `private` 设置为 true，npm 将不会将其发布到仓库。这确保了您不会不小心将代码发布到公共仓库。

`scripts` 对象将命令映射到应用程序生命周期中它们应该运行的那些点。在这个包中，我们告诉 Node.js，当使用 npm 启动应用程序时，它应该运行 `node app` 命令。有一些预定义的生命周期命令，如 `start`、`stop`、`restart` 和 `test`，可以使用 `npm <command>` 运行，例如 `npm start`。

您也可以使用 `run-script` 运行任意命令。为此，您将命令添加到 `scripts` 对象中，然后运行它作为 `npm run-script <command>`。

最后——包中最有趣的部分，引入了魔法——`dependencies`。此对象是依赖包名称和版本的映射，将由 npm 用于拉取所有必需的依赖项。

在我们的包中，`express` 已经定义了对 Express 和 Jade 的依赖。要引入这些依赖项，请运行以下命令：

```js
npm install

```

输出将列出它下载的所有依赖项。

**jade@0.27.2 node_modules/jade**

**├── commander@0.6.1**

**└── mkdirp@0.3.0**

**express@3.0.0rc2 node_modules/express**

**├── methods@0.0.1**

**├── fresh@0.1.0**

**├── range-parser@0.0.4**

**├── cookie@0.0.4**

**├── crc@0.2.0**

**├── commander@0.6.1**

**├── debug@0.7.0**

**├── mkdirp@0.3.3**

**├── send@0.0.3 (mime@1.2.6)**

**└── connect@2.4.2 (pause@0.0.1, bytes@0.1.0, qs@0.4.2, formidable@1.0.11)**

依赖项将被放置在一个名为 `node_modules` 的文件夹中。

接下来，我们将查看应用程序文件 `app.js`：

```js
var express = require('express')
  , routes = require('./routes')
  , http = require('http')
  , path = require('path');

var app = express();

app.configure(function(){
  app.set('port', process.env.PORT || 3000);
  app.set('views', __dirname + '/views');
  app.set('view engine', 'jade');
  app.use(express.favicon());
  app.use(express.logger('dev'));
  app.use(express.bodyParser());
  app.use(express.methodOverride());
  app.use(app.router);
  app.use(express.static(path.join(__dirname, 'public')));
});

app.configure('development', function(){
  app.use(express.errorHandler());
});

app.get('/', routes.index);

http.createServer(app).listen(app.get('port'), function(){
  console.log("Express server listening on port " + app.get('port'));
});
```

在前几行中，Node.js 加载了我们工作所需的模块。我们已经熟悉 `http` 和 `path`。`express` 模块引入了 Express 框架。我们还在加载一个模块 `./routes`，它将加载本地 `routes` 文件夹中定义的模块，该文件夹由 `./routes/index.js` 定义。以下代码片段关注前一个代码片段的前几行：

```js
var express = require('express')
  , routes = require('./routes')
  , http = require('http')
  , path = require('path');
```

在下一行，它将 Express 框架实例化为一个应用：

```js
var app = express();
```

然后是应用程序配置：

```js
app.configure(function(){

});
```

在前面的几行中，我们正在定义一个将配置应用的函数。`app.configure` 的签名是 `app.configure([env], callback)`，其中 `env` 是运行时环境变量 *或* 生产环境变量，如由 `process.env.NODE_ENV` 定义。当我们不指定 `env` 时，它将为所有环境设置。

以下设置已提供，以改变 Express 的行为：

+   `env`: 环境模式，默认为 `process.env.NODE_ENV` 或 "development"

+   `trust proxy`: 启用反向代理支持，默认禁用

+   `jsonp callback`: 启用 JSONP 回调支持，默认启用

+   `jsonp callback name`: 改变默认的回调名称 `?callback=`

+   `json replacer`: JSON 替换回调，默认为 `null`

+   `json spaces`: 用于格式化的 JSON 响应空格；开发中默认为 `2`，生产中为 `0`

+   `case sensitive routing`: 启用大小写敏感的路由，默认禁用，将 `/Foo` 和 `/foo` 视为相同

+   `strict routing`: 启用严格路由，默认情况下 `/foo` 和 `/foo/` 由路由器视为相同

+   `view cache`: 启用视图模板编译缓存，默认在生产环境中启用

+   `view engine`: 当省略时使用的默认引擎扩展

+   `views`: 视图目录路径

以下代码片段演示了如何将设置分配给应用程序：

```js
  app.set('port', process.env.PORT || 3000);
  app.set('views', __dirname + '/views');
  app.set('view engine', 'jade');
```

这里应用程序使用的设置是 `port`、`views` 和 `view engine`，指定应用程序应在端口 `3000` 上运行，视图将放置在 `views` 文件夹中，使用的引擎是 Jade。我们将在后面看到更多关于视图的内容。也可以指定某些功能，如下面的代码片段所示：

```js
  app.use(express.favicon());
  app.use(express.logger('dev'));
  app.use(express.bodyParser());
  app.use(express.methodOverride());
  app.use(app.router);
  app.use(express.static(path.join(__dirname, 'public')));
```

由于 Express 基于 Connect 中间件构建，它带来了许多来自 Connect 的现有功能。Connect 的 `use` 方法配置应用程序以利用给定的中间件处理程序来处理给定的路由，其中路由默认为 `/`。你可以在 [`www.senchalabs.org/connect/`](http://www.senchalabs.org/connect/) 看到 Connect 提供的中间件列表。

让我们遍历这里使用的中间件组件。Favicon 中间件将提供应用程序的 favicon。Logger 中间件以自定义格式记录请求。

Body 解析器解析支持不同格式的请求体；这包括 `application/json`、`application/x-www-form-urlencoded` 和 `multipart/form-data`。

方法覆盖启用模拟 HTTP 方法支持。这意味着如果我们想刺激对应用程序的 `DELETE` 和 `PUT` 方法调用，我们可以通过在请求中添加 `_method` 参数来实现。

`app.router` 提供了 Connect 的 `router` 模块的增强版本。基本上，这是当我们使用 `app.get` 等路由方法时确定要做什么的组件。最后一个中间件，静态文件服务器，提供并配置了它，从 `public` 目录提供文件。

对于开发环境，以下两行代码显示了如何设置错误处理中间件，以便在响应中提供堆栈跟踪和错误消息。

```js
app.configure('development', function(){
  app.use(express.errorHandler());
});
```

下一行配置路由器将 `/` 路由处理为由 `routes` 模块中的 `index` 方法处理：

```js
app.get('/', routes.index);
```

最后，我们启动配置为使用我们刚刚配置的应用实例的 HTTP 服务器。

```js
http.createServer(app).listen(app.get('port'), function(){
  console.log("Express server listening on port " + app.get('port'));
});
```

在应用程序配置中，我们看到了一些文件夹开始发挥作用，即 `routes`、`views` 和 `public`。

`routes` 是我们将编写所有处理程序的模块。在 `/` 的情况下，我们将其映射到使用 `index` 方法从 `routes.index` 方法提供服务。如果你打开 `routes/index.js`，你会看到 `index` 是从这个模块公开的方法。

```js
exports.index = function(req, res){
  res.render('index', { title: 'Express' });
};
```

这个函数签名与我们编写的处理程序类似。它是一个接受请求和响应作为参数的函数。在这里，我们使用 Express 方法 `response.render`，它将使用第二个参数作为模型或数据来渲染提到的视图。

视图位于 `views` 文件夹中，并使用 Jade ([`jade-lang.com/`](http://jade-lang.com/)) 视图引擎。Jade 是一个高性能的模板引擎，深受 Haml ([`haml.info/`](http://haml.info/)) 的影响，并使用 JavaScript 为 Node.js 实现。像许多现代 HTML 生成引擎一样，Jade 试图使 UI 代码更简单、更干净、更简单，去除了内联代码和 HTML 标签的噪音。

现在，我们将看到在 Express 应用程序中定义的视图。这里有两大文件需要查看：`layout.jade` 和 `index.jade`。

如其名所示，`layout.jade` 是我们将由应用程序中的不同页面使用的布局模板。它可以用来放置应用程序页面骨架的公共代码，如下面的代码片段所示：

```js
doctype 5
html
  head
    title= title
    link(rel='stylesheet', href='/stylesheets/style.css')
  body
    block content
```

在 Jade 中，我们不需要 `start` 和 `end` 标签，因为它通过缩进的代码块来识别标签的开始和结束。因此，当我们渲染这个 Jade 文件时，它将生成以下 HTML 代码：

```js
<!DOCTYPE html>
<html>
  <head>
    <title>{TITLE}</title>
    <link rel="stylesheet" href="/stylesheets/style.css" />
  </head>
  <body>{CONTENT}</body>
</html>
```

在前面的代码片段中，有两个东西尚未定义，`{TITLE}` 和 `{CONTENT}`。在 Jade 模板中，我们如下定义标题：

```js
    title= title
```

我们告诉 Jade 使用传递给 `render` 的数据中的标题作为 `title`。第二件事，`{CONTENT}`，在 Jade 中定义为块。

```js
    block content
```

`block content` 是在布局模板中提供的插件点，它可以由任何扩展它的模板描述。

`index.jade` 继承自 `layout.jade`。在我们的索引处理程序中，我们使用数据 `{title: 'Express'}` 来渲染索引视图。请看以下代码片段：

```js
extends layout
block content
  h1= title
  p Welcome to #{title}
```

在上一个文件中，我们定义了内容块包含 `h1` 和 `p` 标签。因此，根据给定的输入并且因为它扩展了布局，Jade 引擎将生成以下 HTML：

```js
<!DOCTYPE html>
<html>
  <head>
    <title>Express</title>
    <link rel="stylesheet" href="/stylesheets/style.css" />
  </head>
  <body>
    <h1>Express</h1>
    <p>Welcome to Express</p>
  </body>
</html>
```

在下一章中，我们将通过我们的聊天应用程序来工作，届时我们将看到更多 Jade 的功能和语法。

在生成的 HTML 代码中，我们可以看到 `/stylesheets/style.css` 正在被引用；此文件由我们在应用程序中配置的静态文件服务器提供。我们可以在公共文件夹中找到此文件和其他文件。

要运行此应用程序，我们将使用 npm。在控制台运行以下命令：

```js
npm start

```

然后转到 `http://localhost:3000/`。

# 摘要

在本章中，我们介绍了 Node.js 和 Express 网络框架。如前所述，这绝对不是对 Node.js 或 Express 的完整介绍。要了解更多信息，请参考在线上可用的广泛文档或任何关于 Node.js 网络开发的书籍。
