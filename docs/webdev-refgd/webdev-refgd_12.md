# 第十二章. 服务器端 JavaScript – NodeJS

Node.js 是一个相对较新的基于 JavaScript 的服务器平台。Node 的一大特点是它是一个非阻塞服务器。这意味着资源密集型任务不会占用服务器。然后 Node.js 可以处理许多并发连接。它也比阻塞服务器更容易处理实时通信。

Node.js 的主要用途之一是作为 Web 服务器。这是一个完美的任务，因为服务网页通常涉及读取文件和连接到数据库。它能够在执行这两个动作中的任何一个时服务更多客户端。这与阻塞服务器非常不同。阻塞服务器必须等待所有资源返回，然后才能响应更多请求。

### Note

注意，这个参考将仅限于 Node.js 特定的；我们不会在本章中涵盖任何框架或库。

最受欢迎的框架是有用的，并且应该在可能的情况下使用，但我们的重点将仅限于 Node.js 内置的函数和模块。我们将不涉及 **Express**（据说是最常用的 Node.js 框架）。最重要的是，我们将涵盖 Express 及其依赖项构建的基础。

一本涵盖框架等内容的好书是 Packt Publishing 出版的 *使用 Redis 和 Node.js 构建可扩展应用程序*。

在本章中，将描述以下几组参考，并附上示例：

+   文件和进程管理：

    +   Modules

    +   OS (operating system)

    +   Process

    +   File

    +   Path

    +   REPL (Read Eval Print Loop)

    +   Errors

+   Utilities:

    +   Events

    +   Crypto

    +   Buffer

    +   Console

    +   Npm (Node Package Manager)

    +   Stream

+   Net 模块：

    +   createServer

    +   net.Server

+   HTTP:

    +   Server

    +   IncomingMessage

    +   ServerResponse

    +   clientRequest

# 文件和进程管理

我们将开始对 Node.js 的概述，从基础知识开始。这包括加载模块、管理进程、处理文件和路径，以及 **REPL** (**读取-评估-打印循环**)。这些都是几乎任何 Node.js 项目都需要的东西。

## Modules

Node.js 是一个模块化系统。Node.js 的许多功能都包含在必须在运行时加载的模块中。这使得了解加载和构建模块成为使用 Node 的核心要求。以下是你可以用于你的 Node 模块的参考：

+   `require()`

+   `modules.export`

### require()

这将在给定路径加载模块：

```js
require(path)
```

#### 返回值

返回值将是一个对象。这个对象的属性将根据加载的内容而变化。我们将在下一节介绍 `module.exports`，这是模块设计者和甚至你可以用来设置这个返回值值的。

#### Description

`require()` 函数用于在指定路径加载模块，其中路径可以是核心模块（与 Node.js 一起打包的模块）、目录、文件或模块（你或其他人构建的项目）。因此，它非常灵活。`require()` 函数将按照以下顺序尝试解析传入的路径：核心模块、如果路径以 "./"、"/" 或 "../" 开头，则目录或文件，然后是模块。有关更多信息，请参阅以下描述。你需要检查不同位置的 require 函数，以便所有这些都能正常工作：

+   你首先会查找核心模块。核心模块可以在 Node.js 的源代码中在 lib 目录下查看。在本章中，我们将介绍最常用的模块。

+   接下来，如果字符串以 "./"、"/" 或 "../" 开头，require 将会将其作为文件加载路径。这些分别是当前目录、根目录和父目录。

+   如果找不到文件名，require 将尝试添加 `.js`、`.json` 或 `.node` 扩展名。这意味着 `require (./data/models)` 函数和 `require (./data/models,js)` 将加载相同的文件。如果 require 找不到文件，它将尝试将路径作为目录加载。这意味着它会寻找 `index.js` 文件或 `package.json` 中的主属性。require 将使用 `index.js` 或 `package.json` 来加载文件。

+   最后，如果路径不以 "./"、"/" 或 "../" 开头，并且不是核心模块，require 将搜索 `node_modules` 文件夹。第一个目录将是 `current` 目录。之后，每个目录将是一个父目录，一直到 `root` 目录。此外，require 还会在 `NODE_PATH` 环境变量中搜索路径。

简要总结一下，require 将会使用的顺序是：

+   核心模块（无相对路径）

+   文件模块（相对路径）

+   文件夹模块（相对路径）

+   `node_modules` 模块（无相对路径）

这里有一些如何使用 require 函数加载核心模块、文件或模块的示例：

```js
var http = require('http'); //loads a core module
```

如注释所示，此示例将加载 HTTP 核心模块：

```js
var data = require('./data'); //loads a directory
```

此示例展示了如何加载一个目录：

```js
var dataModels = require('./data/models'); //loads the file models.js
```

这个将会加载一个文件：

```js
var redis = require('redis'); //loads a module
```

最后，这个将会加载一个模块。

每个 Node.js 项目都会多次使用 require，因此了解它如何以及在哪里加载是很重要的。

这里有一个简单的示例，演示了 require 以及它与 `module.exports` 的工作方式。

这里是将会用 require 加载的文件，`requireTest.js`，它将从相对路径加载：

```js
module.exports = 'This value is from requireTest';
```

这里是如何在 Node.js 脚本中加载这个值的：

```js
var requireTest = require('./requireTest');
console.log(requireTest);
```

### `module.exports`

`module.exports` 属性是一个便利属性，用于在模块中暴露模块的功能，使其超出当前文件的作用域。

#### 返回值

`module.exports` 属性是模块的返回值。这可能会让人产生误解。我们不会返回 `module.exports`。实际上，是设置在 `module.exports` 上的对象或属性可以从该模块中访问。

#### 描述

`module.exports` 属性是从当前文件链接到调用文件的作用域。在当前文件中定义的任何变量都不会在调用它的文件的作用域中。为了使变量可用，你必须将它们设置为 `module.exports` 的值或作为 `module.exports` 的属性。这种功能是 Node.js 使用 require 的机制。导出可以设置为任何类型：字符串、整数、函数、对象等。函数允许我们传递值或创建 `constructor()` 函数。对象可以创建并用于覆盖导出对象或将其作为属性添加到导出对象中。我们拥有 JavaScript 的所有函数和对象创建技巧。

### 注意

一个简短的说明是，require 会缓存输出。这意味着对同一模块的连续调用将返回相同实例的对象。

这里展示了 `constructor()` 函数的使用示例：

+   这允许传递一个对象：

    ```js
    module.exports = function Foo(bar) {do something with bar};
    ```

+   这允许导出函数被执行：

    ```js
    module.exports = function FooConstructor() {};
    ```

+   在以下两个示例中，将使用 `foo` 属性：

    ```js
    module.exports = {foo: foo()};
    module.exports.foo = foo();
    ```

## OS 模块

下面是 `os` 模块最重要的函数和属性。`os` 模块允许你从操作系统中获取信息。这很重要，因为我们可能需要知道主机名或当前有多少个 CPU 可用。

我们现在将要查看的所有函数都假设 `var os = require('os')` 已经在文件中。

### hostname()

这将返回设备的主机名：

```js
.hostname()
```

#### 描述

下面展示了如何使用 `os.hostname()` 函数的示例。在这种情况下，它将返回当前计算机使用的计算机名。它不会返回计算机的域名后缀：

```js
os.hostname();
```

### cpus()

这将返回当前设备的 CPU 数量：

```js
.cpus()
```

#### 描述

我们将得到一个映射到每个 CPU 的对象的数组。这些对象将包含型号、速度和 times：

+   模型和速度分别代表 CPU 的型号和速度。请注意，大多数情况下，型号和速度与我们的代码无关。

+   Times 展示了 CPU 在 `user`、`nice`、`sys`、`idle` 和 `irq` 中的耗时分布。

通常，这些信息可以用来找出机器有多少个 CPU。了解这些信息是有好处的，因为 Node.js 是单线程的，我们可以为每个 CPU 在集群中启动多个进程。

下面是一个获取计算机上 CPU 数量的示例：

```js
var cpus = os.cpus().length;
```

### networkInterfaces()

这将获取网络接口列表：

```js
.networkInterfaces()
```

#### 描述

这将返回一个包含所有网络接口的对象。每个属性映射到一个接口，并且它将有一个包含所有 IP 地址（IPv4 和 IPv6）的数组。

例如，这是获取计算机所有接口的对象的方法：

```js
var network = os.networkInterfaces();
```

## 进程模块

进程是一个对象，它允许我们访问进程上的事件。我们还可以访问`stdin`、`stdout`和`stderr`。这些是全局对象，在任何文件或上下文中都可用。

### stdout

`stdout`对象是一个可写流。

```js
stdout
```

#### 描述

这是连接到`stdout`的流。请注意，流将在本章的“实用工具”部分稍后介绍。如果我们以控制台应用程序的形式运行 Node.js，如果需要，我们可以在这里写入`stdout`。Node.js 中的大多数流都是非阻塞的，但写入 stdout 和 stderr 是阻塞的。

我们还可以使用`stdout`来找出 Node.js 是否在 TTY 中运行。我们可以访问`process.stdout.isTTY`。这是一个布尔值。

此处的示例将展示如何将消息写入`stdout`流。默认情况下，这将发送到`process.stdout.write`控制台。这将写入`stdout`。

### stderr

`stderr`对象是一个可写流。

```js
stderr
```

#### 描述

这与`stdout`类似，关键区别在于它写入`stderr`。这对于控制台应用程序非常有用，因为我们可以将正在发生的事情写入控制台。

例如，这会将内容写入`stderr`流。默认情况下，这将写入控制台。

```js
process.stderr.write("Something seems to have gone wrong.");
```

### stdin

这是一个映射到`stdin`的可读流。

```js
stdin
```

#### 描述

这与标准流`stdin`相对应，就像`stdout`和`stderr`映射到`stderr`流一样。区别在于`stdin`对象是可读的，而其他的是可写的。任何通过管道输入此进程的内容都可以使用`process.stdin`读取出来。要从可读流中读取，我们需要监听两个可能的事件，这些事件会告诉我们如何检索数据。第一个是可读事件，另一个是数据事件。当我们到达流部分时，我们将更深入地介绍这一点。

例如，这个例子将`stdin`中发送的数据通过可读事件发送到`stdout`：

```js
process.stdin.on('readable', function() {
  var data = process.stdin.read();
  if(data !== null) process.stdout.write(data);
});
```

### argv

这就是所有传递的命令行参数：

```js
argv

```

#### 描述

`argv`命令将是一个包含传递给此脚本的全部参数的数组。参数通过空格分隔。这意味着`debug=true`（没有空格）将是一个参数，而`debug = true`（每个单词之间有空格）将是三个。如果你想在参数中传递值，这是一个需要记住的重要区别。参数`0`和`1`将始终是节点和当前脚本的路径和文件名。

这里是一个例子：

```js
process.argv.forEach(function(val){ if(val === 'debug'){ debug(); } });
```

### 信号事件

信号事件允许你监听标准的 POSIX 信号：

```js
process.on({signal, callback});
```

#### 描述

除了`SIGKILL`和`SIGSTOP`之外的所有`POSIX`信号都可以监听。除了这些信号之外，你还可以监听一些 Windows 信号。以下是信号的列表：

+   `SIGUSR1`：这是一个用户定义的信号。在 Node.js 中，它通常用于启动调试器。

+   `SIGTERM`：这是一个终止信号。

+   `SIGINT`：这是中断的信号。通常通过按下 *Ctrl* + *C* 发送。

+   `SIGPIPE`：这会让进程知道它正在尝试向一个不存在的进程进行管道操作。

+   `SIGHUP`：这是告诉进程它运行的终端已经挂起的信号。

+   `SIGBREAK`：这是一个非 POSIX 信号，由 Windows 使用。它应该以类似于`SIGINT`的方式使用。

+   `SIGWINCH`：这是告诉进程窗口已更改大小的信号。

这里是一个监听 `SIGINT` 的示例，它识别出按下了 *Ctrl* + *C*：

```js
process.on('SIGINT', function(){
  //ctrl+c was pressed
});
```

### process.env

此对象包含环境变量：

```js
process.env
```

#### 描述

`process.env` 对象是一个简单的 JavaScript 对象，包含所有环境变量。每个属性都将是一个字符串。

在构建可扩展应用程序时，将配置设置放在环境变量中是一个很好的地方。然后可以使用 `process.env` 对象来检查当前环境是否为生产环境，甚至只是存储配置设置。

这里是一个检查环境和从环境中设置值的示例：

```js
if(process.env.NODE_ENV !== 'production')
  //do test stuff
var redisHost = process.env.REDIS_HOST;
```

### kill

这将向进程发送一个信号事件：

```js
process.kill(pid, [signal])
```

#### 描述

这将发送信号事件部分中定义的任何信号事件，我们在前面已经讨论过。这本质上是在运行 `POSIX` kill 命令。

我们可以使用这个 kill 命令向当前进程发送信号，使用 `process.pid`，或者向另一个我们有引用的进程发送 kill 信号。

这里是一个杀死特定进程 ID `4634` 的示例：

```js
process.kill(4634, 'SIGTERM');
```

### pid

这将获取当前的 `pid`：

```js
process.pid
```

#### 描述

这是进程的 `pid`（进程 ID）。

这是一个 `process.kill` 和 `process.pid` 一起使用的示例：

```js
process.kill(process.pid, 'SIGTERM');
```

### cwd

这将获取当前工作目录：

```js
process.cwd()
```

这里是一个将 `cwd` 变量设置为进程当前工作目录的示例：

```js
var cwd = process.cwd();
```

## 文件函数

本节不是一个真正的模块，就像之前关于进程的章节。本节将关注任何允许我们读取、写入或查找文件和目录的函数或模块。

### __filename

这将返回当前文件名的字符串：

```js
__filename

```

#### 描述

`__filename` 命令返回当前文件名。这可能会根据正在执行的文件而有所不同。正在执行的模块将返回其文件名而不是主入口点。

这里是一个将当前文件名记录到控制台的示例：

```js
console.log(__filename);

```

### __dirname

这是一个当前目录的字符串：

```js
__dirname

```

#### 描述

与 `__filename` 类似，这将返回当前正在执行的文件的目录。`__dirname` 命令在需要创建相对路径时经常被使用。

### 注意

`__filename` 和 `__dirname` 变量在 Node.js 执行的任何文件中都是可用的。它们按文件确定，因此对于每个执行的文件都是正确的。

此示例假设 Express 已经被加载为 express：

```js
//express is loaded
app.use(express.static(__dirname + '/static'));
```

### 文件模块

我们现在将查看文件模块中的关键函数列表。所有这些函数只是运行底层的 `POSIX` 命令。

我们将要介绍的每个命令都会有每个函数的异步和同步版本。同步函数将使用相同的名称，并在后面添加 Sync，例如 `open` 和 `openSync`。从某种意义上说，异步版本更像是 Node.js，因为它们允许事件循环处理而不阻塞执行。

每个异步函数都需要定义一个作为最后一个参数的回调函数。这个函数的形式将是 `function(err, result)`。我们将在处理错误时介绍这一点。快速总结是，如果没有发生错误，第一个参数 `err` 将是 `null`，或者它将包含错误。

### 注意

在当前版本的 Node.js 中，您不需要使用回调函数，但在 v0.12 版本中这将成为一个例外。

同步函数将在函数返回之前不在事件循环中处理任何内容。虽然这似乎与 Node.js 的非阻塞范式相矛盾，但在某些情况下是有意义的，例如，在处理其他任何内容之前必须打开特定文件时。这意味着需要使用经典的 `try/catch` 来处理任何异常或错误。

在几乎所有情况下，都应该使用异步版本。

每个示例都基于 `fs` 变量已经存在的假设。以下是设置 `fs` 变量为 `fs` 模块的代码：

```js
var fs = require('fs');
```

### stat

这将返回 `fs.Stats`，这是一个包含查询的文件或目录信息的对象：

```js
fs.statstat(path, callback)
fs.statSync(path)
```

#### 描述

`stat` 变量获取在操作系统上运行 `stat()` 后返回的数据。`stat` 对象是 `fs.Stats` 的一个实例。该对象提供了诸如 `isFile()` 和 `isDirectory()` 等有用的函数。此外，`fs.Stats` 还有许多属性。我们可以看到修改时间、访问时间和文件大小。

还有更多函数和属性，但这些都是最可能使用的。

`fs.lstat()` 和 `fs.fstat()` 函数都会在指向文件时返回一个 `fs.Stats` 对象。区别在于 `lstat` 将针对链接运行而不是跟随链接到文件或目录。`fstat` 将针对文件描述符而不是路径运行。

这里有一个示例，它加载与代码文件相同的目录下的名为 `test.txt` 的文件，并记录 `fs.Stats` 对象：

```js
fs.stat(__dirname + '\test.txt', function(err, stats){
  console.log(stats);
});
```

### open

这将返回传入路径的文件描述符：

```js
fs.open(path, flags, [mode], callback)
fs.openSync(path, flags, [mode])
```

#### 描述

这将在传入的路径中打开一个文件。可以传入许多标志。它们在这里进行了描述：

+   `r`: 只读，当文件不存在时出错

+   `r+`: 读写，当文件不存在时出错

+   `rs`: 只读同步，这里的“同步”仅指文件系统缓存数据，而不是像 `openSync` 这样的同步

+   `rs+`: 读写同步

+   `w`: 写入

+   `wx`: 写入，当文件存在时出错

+   `w+`: 读写，文件被创建或截断

+   `a`: 追加，文件被创建

+   `ax`: 追加，当文件存在时出错

+   `a+`: 读取和追加，文件被创建

+   `ax+`: 读取和追加，当文件存在时出错

标志允许你打开、读取和写入文件，无论它是否存在。这意味着大多数时候，不需要调用 `fs.exists` 作为标志之一，因为其中一个标志会给你需要的答案。

模式参数是在创建文件时使用的权限。默认为 `0666`。

最后，回调返回一个文件描述符。有了这个，可以使用 `fs.read` 或 `fs.write` 来读取或写入数据。

这是一个打开文件 `test.txt` 的示例：

```js
fs.open(__dirname + '\file.txt', 'r', function(err, fd){
  //fd is available to be read from
});
```

### read

从文件描述符读取：

```js
fs.read(fd, buffer, offset, length, position, callback) 
fs.readSync(fd, buffer, offset, length, position)
```

#### 描述

`read()` 函数运行需要很多参数。我们需要一个文件描述符，这是从打开中获得的，一个缓冲区，以及文件的大小。在示例中，我们使用了 `fs.stat` 来获取文件大小。

这是一个打开文件并从中读取的完整示例：

```js
fs.stat(__dirname + '/test.txt', function(error, stats) {
  fs.open(__dirname + '/test.txt', 'r', function(err, fd){
    var buffer = new Buffer(stats.size);
    fs.read(fd, buffer, 0, stats.size, null, function(err, bytesRead, buffer){
      console.log(buffer.toString('utf8'));
    });
  });
});
```

### readFile

这是一个读取文件的简化版本：

```js
fs.readFile(filename, [options], callback)
fs.readFileSync(filename, [options])
```

#### 描述

`readFile()` 函数极大地简化了在 Node.js 中读取文件的过程。在之前的 `fs.read` 函数中，我们需要在尝试读取文件之前设置好一切。`readFile()` 函数允许我们只传递一个文件名，然后返回文件中的缓冲区。

`optional options` 对象接受一个标志和一个编码属性。标志属性与 `fs.open` 中的标志相同，因此可以使用任何一个。编码用于回调中返回的缓冲区。

这里有一个示例，展示了使用 `readFile` 加载文件有多容易。该示例还使用了 `optional options` 参数：

```js
var filename = __dirname + '/test.txt';

fs.readFile(filename, {flag: 'r', encoding: 'utf8'}, function(err, data){
  console.log(data.toString('utf8'));
});
```

### close

将关闭一个文件描述符：

```js
fs.close(fd, callback)
fs.closeSync(fd)
```

#### 描述

在我们的脚本中打开的任何文件都很重要。如果我们从 `fs.open` 获取文件描述符，我们将在某个时候调用 `fs.close` 来关闭该文件描述符。

这是一个关闭文件描述符的示例：

```js
fs.close(fd, function(err){
  //handle err here
});
```

### write

这是写入文件描述符的示例：

```js
fs.write(fd, buffer, offset, length, position, callback)
fs.writeSync(fd, buffer, offset, length, position)
```

#### 描述

`write()` 函数接受一个文件描述符和一个要写入文件的缓冲区。与读取类似，我们必须传递相当多的参数才能使其工作。

在下面的示例中，我们将文件读入缓冲区，然后将该缓冲区写回另一个文件。偏移量和长度都是需要传递的整数。位置可以是整数，也可以是 `null`。`Null` 将从文件当前位置开始写入。

就像读取一样，写入也可以很复杂。这里有一个从文件读取并写入另一个文件的完整示例：

```js
var fs = require('fs');
var filename = __dirname + '/test.txt';
var writeFile = __dirname + '/test2.txt';

fs.stat(filename, function(error, stats) {
  fs.open(filename, 'r', function(err, fd) {
    var buffer = new Buffer(stats.size);
    fs.read(fd, buffer, 0, stats.size, null, function(err, bytesRead, buffer) {
      fs.open(writeFile, 'w', function(err, writeFD) {
        //will create a file named test2.txt with the contents from test.txt
        fs.write(writeFD, buffer, 0, buffer.length, null, function(err, bytesWritten, writeBuffer) {
          console.log(writeBuffer.toString('utf8'));
        });
      });
    });
  });
});
```

### writeFile

这是一个写入文件的简化函数：

```js
fs.writeFile(filename, data, [options], callback)
fs.writeFileSync(filename, data, [options])
```

#### 描述

正如读取和 `readFile` 之间的区别一样，`writeFile` 是写入的简化版本。我们只需传递一个文件名和一个字符串或缓冲区，它就会写入文件。

当发生错误时，回调只返回错误。

在大多数情况下，`readFile`和`writeFile`似乎是最佳选择，实际上，这很可能就是真的。使用这些函数所放弃的主要是控制权。你只能读取整个文件或写入整个文件。

### 注意

使用读取或写入，你可以读取文件中的任何部分，也可以写入文件中的任何部分。这一点很重要，需要记住。

这里是`writeFile`的一个例子：

```js
var filename = __dirname + '/write.txt';
var buffer = new Buffer('Write this to a file.', 'utf8');
fs.writeFile(filename, buffer, {encoding: 'utf8', flag: 'w'}, function(err){
  if(null !== null){
    //do something
  }
});
```

### appendFile

这个函数允许我们追加文件：

```js
fs.appendFile(filename, data, [options], callback) 
fs.appendFileSync(filename, data, [options])
```

#### 描述

`appendFile`存在的原因是`writeFile`只会写入整个文件。因此，我们需要另一个函数来向文件中添加内容。这就是为了方便而放弃一些控制权的地方。如果我们使用文件描述符和写入操作，那么我们可以选择从文件的末尾开始写入。这实际上就是追加文件：

这里是`appendFile`的一个例子：

```js
var filename = __dirname + '/write.txt';
var buffer = new Buffer('Append this to a file.', 'utf8');
fs.appendFile(filename, buffer, {encoding: 'utf8', flag: 'w'}, function(err){
  if(null !== null){
    //do something
  }
});
```

## `path`模块

`path`模块与文件模块是分开的。`path`模块关注的是路径的修正，而文件模块关注的是文件和目录的操作。很多时候，这两个模块会一起使用。

涵盖的函数是最常用且最有用的。你很可能需要定位到当前项目相对的路径，这正是`path`模块的作用所在：

### 注意

注意，`path`模块不会检查路径修改的存在性。它本质上只对路径的字符串值进行修改。

就像其他模块一样，这个例子假设`path`模块已经被加载：

```js
var path = require('path');
```

### normalize

这将返回一个经过修正的路径字符串：

```js
path.normalize(pathString)
```

#### 描述

`normalize`函数将修复与读取路径相关联的许多问题。一个很好的例子是 Windows 和 Unix 路径之间的差异。Unix 使用正斜杠，而 Windows 使用反斜杠。`normalize`将根据执行的环境返回正确的斜杠。

此外，`normalize`还会移除当前目录和父目录的快捷方式（分别是`.`和`..`），也会从目录中移除多余的斜杠。它会规范化传入的所有路径。

这里是一个在 Windows 系统上使用 Unix 斜杠的例子。这个例子会将所有的斜杠都转换为反斜杠：

```js
var pathString = "/unix/style/path/separators";
console.log(path.normalize(pathString));
```

### join

这将返回所有路径连接在一起形成的字符串：

```js
path.join([pathString1],[…])
```

#### 描述

`join`函数使得从部分路径创建完整路径变得简单。这看起来像是可以通过连接操作完成的，但很容易忘记路径分隔符。`join`函数会确保路径分隔符都位于正确的位置。

这里是`join`的一个例子。这个例子将接受所有参数，并使用系统正确的斜杠将它们连接起来：

```js
console.log(path.join('path', 'separators', 'added'));
```

### resolve

这将根据参数返回路径的字符串：

```js
path.resolve([pathString], […])
```

#### 描述

这可以看作是对每个参数执行的 cd 命令。传入的路径可以是相对路径或完整路径。相对路径将添加到返回的路径中，而完整路径将被整个使用。

这里有两个例子：

+   相对路径将返回`/home/josh/test`：

    ```js
    console.log(path.resolve('/home/josh/node', '..', 'test'));
    ```

+   完整路径将返回`/home/brian/node`：

    ```js
    console.log(path.resolve('/home/josh/node', '/home/brian/node'));
    ```

### relative

这返回`to`和`from`路径之间的差异：

```js
path.relative(from, to)
```

#### 描述

返回的路径可以用`cd`来更改到新目录。

以下是一个示例，将返回`../../brian/node`，因为您必须向上移动两个父目录，然后转到`brian/node`才能从一个移动到另一个：

```js
var from = '/home/josh/node';
var to = '/home/brian/node';
console.log(path.relative(from, to));
```

### dirname

这返回目录名字符串：

```js
path.dirname(pathString)
```

此示例将返回当前文件所在的目录。如果文件位于`/home/users/jjohanan/node`目录中，则示例将返回`/home/users/jjohanan/node`：

```js
console.log(path.dirname(__filename));
```

### basename

这返回路径的最后一部分字符串：

```js
path.basename(pathString, [ext])
```

#### 描述

这将根据传入的路径返回文件名或目录。如果存在，可选的`ext`参数将从返回值中移除该部分。

这里有两个示例，一个涉及文件，另一个涉及目录：

+   这将返回`test`：

    ```js
    console.log(path.basename('/home/josh/test.js', '.js'));
    ```

+   这将返回`josh`：

    ```js
    console.log(path.basename('/home/josh'));
    ```

### extname

这将返回路径的扩展名字符串。如果没有扩展名，则返回一个空字符串：

```js
path.extname(pathString)
```

这里有两个示例：

+   此示例将返回`.js`：

    ```js
    console.log(path.extname('/home/josh/test.js'));
    ```

+   此示例将返回一个空字符串：

    ```js
    console.log(path.extname('/home/josh'));
    ```

## REPL

REPL 代表**读取-评估-打印循环**。这意味着它是一个交互式控制台。我们可以输入一个命令（**读取**）。命令将被执行（**评估**）。命令的输出将被打印到控制台（**打印**）。最后，我们可以多次这样做（**循环**）。REPL 非常适合运行几行代码以查看它们将做什么。

### node

这将在 REPL 模式下启动 Node.js：

```js
node
```

#### 描述

REPL 的主要使用方式是通过直接调用它。这可以通过运行不带文件服务的 Node.js 来实现，只需运行`node`。一旦`node`返回提示符，我们就可以添加命令并查看运行它们时会发生什么。这对于测试几行代码非常完美。

这里是一个快速示例，说明如何在控制台中登录；首先运行`node`以获取提示符，然后运行命令：

```js
node
//wait for >
console.log('hey this REPL!');
```

## 处理错误

错误是任何开发项目的一部分。我们必须能够优雅地处理错误。如果我们不这样做，那么我们正在为我们的用户提供糟糕的体验。更糟糕的是，我们可能会打开攻击的途径。发送给最终用户的堆栈跟踪可能会泄露许多细节。

这将是一个专门处理设计模式而不是实际代码引用的特殊部分。Node.js 是一个异步事件驱动平台。对于大多数人来说，他们没有在类似这样的平台上工作过，处理错误时可能会犯错。我们认为处理错误非常重要。

这些信息的核心来自 Joyent，它是今天`Node.js`背后的主要力量之一。您可以在[`www.joyent.com/developers/node/design/errors`](https://www.joyent.com/developers/node/design/errors)上找到有关 Joyent 的更多信息。

### 错误类型

错误可以分为两种类型：**操作**错误和**程序员**错误。操作错误是在应用程序操作期间发生的错误。例如，数据库服务器不可访问就是一个例子。这些错误可以计划并优雅地处理。

接下来是程序员错误，这在字面上是错误。例如，一段代码格式不正确或出现了意外的条件。这些很难计划（如果我们已经计划了它们，那么它们就不会是错误了！）。这些几乎总是会破坏服务器，因此我们可以通过日志回溯来找出出了什么问题。

### 错误设计模式

现在我们已经知道了两种不同类型的错误，让我们看看三种通知应用程序有错误的方式。这三种方式是抛出错误、异步回调和发出错误事件：

+   抛出错误的第一个模式是内置于 JavaScript 中的。这种模式非常适合任何同步代码。如果我们进行任何同步操作并且发生错误，我们应该抛出它。在处理同步调用时，我们应该在函数调用周围包裹一个 `try/catch` 块。以下是一个使用 `JSON.parse` 的示例，它同步运行，并在传递非 JSON 字符串时抛出错误：

    ```js
    try{
        JSON.parse(jsonObject);
    } catch (ex) {
        //do something with this error
    }
    ```

+   下一个模式是使用异步回调。许多内置的 Node 函数已经这样做了。模式是使用一个具有签名 `function(error, result)` 的回调函数。第一个参数将要么包含一个错误，要么是 null 或 undefined。我们可以在编写任何异步函数时自行实现这一点。如果有错误，请在回调中将它作为第一个参数返回。

    ### 注意

    当处理这些错误时，我们必须在每个回调函数中放置一个错误检查。这是非常重要的，因为不这样做可能会默默地吞下错误。

这的一个好例子是异步和同步的 `filesystem` 模块调用。例如，读取操作需要一个回调，而 `readSync` 应该被包裹在一个 `try/catch` 块中。

这里有一个示例回调和错误检查：

```js
fs.read(path, function (err, data) {
    if(err !== null)
        //handle error
})
```

最后，我们可以发出一个错误事件。这也用于异步函数。我们实现回调或事件是个人选择，但应该清楚使用的是哪一个。这也是一个最佳实践，只实现一个。很多时候，当存在长时间运行的异步过程时，会使用事件。从网络套接字读取数据就是一个例子。套接字并不总是简单地在一次传递中提供数据，因此会设置事件。其中之一就是错误事件。要处理这个问题，我们只需要监听那个事件。以下是一个监听套接字错误事件的示例：

```js
socket.on('error', function(error){
   //handle error here
})
```

# 工具

在下一组模块中，我们将查看工具。这里选择的功能被用于许多不同类型的应用程序。我们将涵盖从事件和密码学到缓冲区和 npm 的所有内容。

## 事件

事件被用于许多内置的 Node 对象中。这是因为触发和监听事件是让另一个函数知道何时开始执行的最佳方式。这在 Node.js 的异步世界中尤其如此。任何时候我们使用对象的 `on` 函数，就意味着它已经继承了 `EventEmitter`。所有示例都将假设事件变量已经按照以下方式创建：

```js
var events = require('events');
```

### EventEmitter

这是可以继承以创建新 `EventEmitter` 的父类：

```js
events.EventEmitter
```

#### 描述

Node.js 拥有一个功能齐全的事件系统，我们可以轻松地继承并实现。我们不需要任何额外的框架或自定义代码。`EventEmitter` 是要继承的类，我们将获得本节其余部分的所有函数。

这里是设置自定义 `EventEmitter` 参数的示例：

```js
var util = require('util');
var events = require('events');

function MyEventEmitter(){
    events.EventEmitter.call(this);
    this.test = function (emitThis) {
        this.emit('testEvent', emitThis);
    }
}

util.inherits(MyEventEmitter, events.EventEmitter);

var myEE = new MyEventEmitter();

myEE.on('testEvent', function (data) { console.log(data) });

myEE.test('test');
```

### on

这个函数为特定事件添加监听器：

```js
emitter.on(event, listenerFunction)
emitter.addListener(event, listenerFunction)
```

#### 描述

`on` 函数已成为添加事件监听器的首选命名约定。例如，jQuery 使用完全相同的函数名作为它们的事件监听器。`event` 处理器是即将触发的事件的字符串名称。`listenerFunction` 参数是事件触发时将要执行的函数。

`listenerFunction` 参数可以是一个匿名函数或对函数的引用。添加监听器的首选方式是对函数的引用。这将允许我们在稍后时间移除这个特定的监听器。

这里是一个基于我们新的 `MyEventEmitter` 类的示例：

```js
var quickLog = function (data) {
    console.log('quickLog: ' + data);
}
myEE.on('testEvent', quickLog);
```

### once

这与 `on` 函数的工作方式相同，但它只执行一次，然后作为监听器移除自己：

```js
emitter.once(event, listenerFunction)
```

### removeListener

这是一个用于从事件中移除监听器的函数：

```js
emitter.removeListener(event, function)
```

#### 描述

当我们完成对事件的监听后，我们希望移除我们的监听器。这将有助于防止内存泄漏。如果我们添加了一个匿名函数作为监听器，那么我们无法像之前那样移除它，因为我们没有对这个函数的引用。使用我们之前的 `on` 示例，我们将移除监听器：

```js
myEE.removeListener('testEvent', quickLog);
```

### removeAllListeners

这个函数将移除所有事件或特定事件的监听器：

```js
emitter.removeAllListeners([event])
```

#### 描述

这实际上是一种移除监听器的核选项。它对监听器没有选择性。`removeAllListeners` 参数甚至会移除我们没有添加的监听器。请将此作为最后的手段使用。

这里展示了一个移除此事件所有监听器的示例。如果事件留空，它将移除所有事件的监听器：

```js
myEE.removeAllListeners('testEvent');
```

### setMaxListeners

这个函数设置了在 Node.js 之前监听器的数量，并警告可能存在的内存泄漏：

```js
emitter.setMaxListeners(numberOfListeners)
```

当监听器的数量超过阈值时，Node.js 会提供一个有用的警告。默认值是 `10`，所以当你添加第十一个监听器时，Node.js 将会警告：

```js
(node) warning: possible EventEmitter memory leak detected. 11 listeners added. Use emitter.setMaxListeners() to increase limit.
```

作为一般规则，这是正确的。如果我们继续向事件添加监听器，就有很大的内存泄漏风险。然而，有时我们可能需要在事件上添加超过 `10` 个监听器。这就是我们使用 `setMaxListeners` 的地方。如果我们把最大监听器数设置为零，那么我们可以添加任意多个。

这里有一个将最大监听器数设置为 `50` 的示例：

```js
myEE.setMaxListeners(50);
```

### emit

这就是如何触发一个事件的方式：

```js
emitter.emit(eventName, [argument], […])
```

如果我们扩展了一个对象使其成为事件发射器，那么我们可能想要发射一些事件！这是执行此操作的函数。它将根据 `eventName` 执行已附加到该事件的全部事件监听器。

这里有一个示例，展示了添加监听器然后触发事件的操作：

```js
myEE.on('testEvent', function (data) { console.log(data) });
myEE.emit('testEvent', 'Emit This!', 'Another Argument!');
```

## Crypto

每个现代应用程序都需要加密。Node.js 使用加密的一个很好的例子是与 HTTPS 一起使用。我们不会探索 HTTPS 的内部工作原理，因为有一个模块（https 模块）为我们做了这件事。我们将查看用于哈希、存储和检查密码的加密函数。

与其他模块一样，我们将需要 `crypto` 模块，并在我们的示例中使用它。以下是我们需要的内容：

```js
var crypto = require('crypto');
```

### createHash

此函数将返回一个 `crypto.Hash` 对象：

```js
crypto.createHash(algorithm)
```

#### Description

可以使用的算法将因系统而异，因为它依赖于 OpenSSL。我们可以通过调用 `crypto.getHashes()` 来找出 crypto 可以使用的算法。这将返回一个字符串数组，然后可以将其传递给 `createHash` 作为算法。

此函数的返回对象是一个 `crypto.Hash` 对象，这在下一节中会介绍。

这里有一个创建 MD5 哈希的示例：

```js
var md5 = crypto.createHash('md5');
```

### 哈希对象

这是 `crypto.createHash` 返回的对象：

```js
hash.update(data, [encoding])
hash.digest([encoding])
```

#### Description

一旦我们获得了返回的哈希对象的引用，您会看到它有两个函数。第一个是 update，它允许我们添加到将被哈希的数据。这可以多次调用。如果我们想要哈希流输入，这很重要。

下一个函数是 digest。这将根据创建哈希对象时使用的算法返回摘要。此函数的字符串编码可以是十六进制、二进制或 BASE64\。

这里有一个从文件中读取数据然后计算文件 MD5 哈希的完整示例。

```js
var f = file.readFileSync(__dirname + '/test.txt');

var md5 = crypto.createHash('md5');
md5.update(f);
console.log(md5.digest('base64'));
```

### Tip

不要使用哈希来存储密码。我们将要介绍的下一个函数比简单的哈希更安全。摘要哈希非常适合检查数据是否已更改，因为即使是一个比特位不同，哈希值也会不同。此外，它可以用作密钥或标识符。一个很好的例子是将其用于内存缓存，并使用哈希作为密钥。如果数据相同，密钥也将相同。

### pbkdf2

此函数将多次使用 HMAC-SHA1 来创建派生密钥：

```js
crypto.pbkdf2(password, salt, iterations, keyLength, callback)
crypto.pbkdf2Sync(password, salt, iterations, keyLength)
```

#### 返回类型

`pbkdf2` 和 `pbkdf2Sync` 都会返回一个作为缓冲区的派生密钥。

#### Description

**Pbkdf2**（基于密码的派生函数 2）是为密码存储而设计的。它比哈希更安全，因为它更难。哈希是为了快速计算而设计的。这是不好的，因为现代 CPU 每秒可以计算数千个哈希值。这使得破解哈希密码变得容易。

Pbkdf2 通过在迭代中使用工作因子来解决这个问题。迭代次数越高，计算时间越长。现在，我们不再每秒计算数千次，而是可以将 CPU 速度降低到每秒仅几个。这是一个显著的降低。

这些是使用的参数：

+   `password`：这是我们想要创建派生密钥的字符串。

+   `salt`：这是一个与密码结合的字符串。这样做可以确保即使 `salt` 不同，相同的密码也不会有相同的哈希值。

+   `iterations`：这是工作因子，指示函数重复的次数。随着 CPU 的变快，这可以增加。目前，至少 10,000 次可以创建一个相当安全的派生密钥。

+   `keyLength`：这是返回的派生密钥的期望长度。

这是一个为字符串 `password` 和 `salt` 创建派生密钥的示例：

```js
crypto.pbkdf2('password', 'salt', 10000, 32, function (err, key) {
    console.log(key.toString('base64'));
});
```

### randomBytes

这将返回具有加密强度的伪随机数据：

```js
crypto.randomBytes(length, [callback])
```

#### 返回类型

返回一个缓冲区。

#### 描述

需要随机数据用于各种函数。虽然没有任何数据可以真正是随机的，但存在不同级别的随机性。`randomBytes` 参数的随机性足以用于加密函数。`randomBytes` 的一个完美用途是在 `pbkdf2` 中用作 `salt`。`salt` 变量与密码结合，即使密码相同，也会创建不同的哈希值。

此函数可以异步或同步执行。这取决于是否有回调，如果有，它将异步执行。

这是一个为 `pbkdf2` 创建随机 `salt` 的函数。如果您将此示例与上一个示例进行比较，您将看到此示例每次输出一个独特的字符串，而上一个示例则不是：

```js
var random = crypto.randomBytes(256);

crypto.pbkdf2('password', random.toString('base64'), 10000, 32, function (err, key) {
    console.log(key.toString('base64'));
});
```

### pseudoRandomBytes

这将返回伪随机数据：

```js
crypto.pseudoRandomBytes(length, [calback])
crypto.pseudoRandomBytes(length)
```

#### 返回类型

返回一个缓冲区。

#### 描述

这个函数与 `randomBytes` 完全一样，但它不是加密强度的。这意味着我们不应该将其与任何加密函数（如 `pbkdf2`）一起使用。

我们可以用它做任何需要随机性、文件名或缓存键等其他事情。

这是一个异步执行此函数的简单示例：

```js
crypto.pseudoRandomBytes(256, function (err, randomData) {
    console.log(randomData.toString('base64'));
});
```

## 缓冲区

Node.js 在许多方面都使用缓冲区。我们已经在许多函数返回缓冲区时看到了这一点。这是因为任何需要存储原始数据或二进制数据时，它都会存储在缓冲区中。

缓冲区有几个需要注意的特性。首先，缓冲区存储数据，但为了利用其中存储的数据，我们必须对其进行编码。本节将介绍编码方式以及如何进行编码。其次，缓冲区的大小不能调整。当创建缓冲区时，我们必须指定一个长度，这个长度将始终是缓冲区的长度。当然，我们可以创建一个新的更大的缓冲区，然后将数据复制过去。最后，缓冲区是全局的。我们不需要使用 `var buffer = require('buffer')`。此外，它可以在任何文件中的任何时间被使用。

### 缓冲区创建

缓冲区的初始化函数如下代码片段所示：

```js
new Buffer(size)
new Buffer(array)
new Buffer(str, [encoding])
```

#### 返回值

这将返回一个缓冲区对象。

#### 描述

有三种不同的方式可以初始化缓冲区。第一个函数使用给定大小的整数，下一个函数使用数组，最后一种方法可以使用字符串。编码是可选的，默认为 UFT8。

此示例演示了使用包含 ASCII 码中`hello`字符串的数组进行初始化：

```js
var hello = [72, 101, 108, 108, 111];
var buffer = new Buffer(hello);
console.log(buffer.toString('ascii'));
```

### index

这在缓冲区的特定索引处获取值：

```js
buffer[index]
```

#### 返回值

这将返回指定索引处的值。

#### 描述

这与数组的工作方式非常相似。以下是一个获取缓冲区第一个索引的示例：

```js
var buffer = new Buffer('Hello!');
console.log(buffer[0]);
```

### toString

这将根据编码返回缓冲区作为字符串：

```js
buffer.toString([encoding], [start], [end])
```

#### 返回值

这将返回一个字符串。

#### 描述

这很可能是您将用于从缓冲区获取数据的函数。第一个参数是编码，它可以是以下之一：

+   ASCII

+   UTF8

+   UTF16LE

+   BASE64

+   Binary

+   Hex

所有参数都是可选的，这意味着它们都有默认值。编码默认为 UTF8，`start`为`0`，`end`为缓冲区的末尾。

下面是一个创建缓冲区并从中检索数据的示例。它明确定义了每个参数：

```js
var buffer = new Buffer('Hello this is a buffer');
console.log(buffer.toString('utf8', 0, buffer.length));
```

### toJSON

这将返回缓冲区的内容作为 JavaScript 对象：

```js
buffer.toJSON()
```

#### 返回值

这将返回一个包含缓冲区内容的数组。

#### 描述

这将返回缓冲区内容映射到数组。此示例类似于上一个示例，但使用了`toJSON`：

```js
var buffer = new Buffer('Hello this is a buffer');
console.log(buffer.toJSON());
```

### isBuffer

这是一个类方法，用于确定一个对象是否是缓冲区：

```js
Buffer.isBuffer(objectToTest)
```

#### 返回值

这将返回一个布尔值。

#### 描述

记住这是一个类方法，因此可以在没有缓冲区新实例的情况下执行。以下是一个使用该函数的示例：

```js
var buffer = new Buffer('Hello this is a buffer');
console.log(Buffer.isBuffer(buffer));
```

### write

以下代码向缓冲区写入数据。

```js
buffer.write(stringToWrite, [offset], [length], [encoding])
```

#### 返回值

这将返回写入的字节数的整数。

#### 描述

此函数将写入传递到缓冲区的字符串。与其他缓冲区函数一样，有许多可选参数具有默认值。默认偏移量为`0`，长度为`buffer.length – offset`，编码为 UTF8。

此示例使用第一次写入的返回值向缓冲区写入两次，以附加第二个字符串：

```js
var buffer = new Buffer(12);
var written = buffer.write('Buffer ', 0, 7, 'utf8');
console.log(written);
buffer.write('time.', written);
console.log(buffer.toString());
```

### byteLength

此函数将根据编码获取字符串的字节长度：

```js
buffer.byteLength(string, [encoding])
```

#### 返回值

这返回一个整数，表示所需的字节数。

#### 描述

缓冲区初始化后不能调整大小，因此我们可能需要事先知道字符串的大小。这就是 `byteLength` 发挥作用的地方。

这里是一个确定字符串大小并将其写入缓冲区的示例：

```js
var byteLength = Buffer.byteLength('Buffer time.', 'utf8');
var buffer = new Buffer(byteLength);
var written = buffer.write('Buffer time.', 0, buffer.length, 'utf8');
console.log(buffer.toString());
```

### readUInt

这将在缓冲区的某个位置获取一个无符号整数：

```js
buffer.readUInt8(offset, [noAssert])
buffer.readUInt16LE(offset, [noAssert])
buffer.readUInt16BE(offset, [noAssert])
buffer.readUInt32LE(offset, [noAssert])
buffer.readUInt32BE(offset, [noAssert])
```

#### 返回值

这返回一个表示使用大小的无符号整数，`readUInt8` 为 `8`。

#### 描述

存储在缓冲区中的数据并不总是精确的字节。有时，数据需要以 16 位或 32 位块读取。此外，您还可以指定数据是低位字节序还是高位字节序，分别由函数名中的 `LE` 或 `BE` 表示。

偏移量是指从缓冲区的哪个位置开始。如果偏移量大小在缓冲区中，`noAssert` 参数将运行验证。默认情况下，它是 `false`。

这里是一个设置数据然后使用 `readUInt16` 读取数据的示例：

```js
var buffer = new Buffer(2);
buffer[0] = 0x1;
buffer[1] = 0x2;
console.log(buffer.readUInt16LE(0));
console.log(buffer.readUInt16BE(0));
```

### writeUInt

这将一个无符号整数写入缓冲区：

```js
buffer.writeUInt8(value, offset, [noAssert])
buffer.writeUInt16LE(value, offset, [noAssert])
buffer.writeUInt16BE(value, offset, [noAssert])
buffer.writeUInt32LE(value, offset, [noAssert])
buffer.writeUInt32BE(value, offset, [noAssert])
```

#### 描述

这与 `readUInt` 函数正好相反。有时，我们需要写入长度大于一个字节的 数据。这些函数使这变得简单。

`noAssert` 参数是可选的，默认为 `false`。这不会对值或偏移量执行任何验证。

这里是一个将 `UInt16` 写入缓冲区的示例：

```js
var buffer = new Buffer(4);
buffer.writeUInt16LE(0x0001, 0);
buffer.writeUInt16LE(0x0002, 2);
console.log(buffer);
```

### 小贴士

注意，缓冲区类还有更多此类读写函数。而不是创建一个非常冗余的部分，我将列出它们。请记住，它们与我们所讨论的函数以相同的方式工作：`readInt8`、`readInt16LE`、`readInt16BE`、`readInt32LE`、`readInt32BE`、`readDoubleLE`、`readDoubleBE`、`readFloatLE` 和 `readFloatBE`。还有一个 `write()` 函数映射到这些函数中的每一个。

## 控制台

这与大多数现代浏览器中存在的控制台非常相似。控制台本质上映射到 `stdout` 和 `stderr`。这不是一个模块，每个函数都不太复杂，所以让我们直接进入。

### log

这写入到 `stdout`：

```js
console.log(message, […])
```

#### 描述

这可能是最常用的控制台函数。它非常适合调试，输出可以与管道结合写入文件以保存历史记录。多个参数创建了一个类似字符串的函数。

这里是一个使用多个参数的示例：

```js
console.log('Multiple parameters in %s', 'console.log');
```

### dir

这是 `util.inspect` 的别名：

```js
console.dir(object)
```

#### 描述

许多时候，`console.log` 和 `console.dir` 的输出将是相似的。然而，当试图查看一个对象时，应该首选 `console.dir`。

### time 和 timeEnd

这两个函数一起使用来标记计时器的开始和结束：

```js
console.time(label)
console.timeEnd(label)
```

#### 描述

这两个函数总是一起使用。`console.time` 参数将启动一个计时器，可以通过传递相同的标签使用 `console.timeEnd` 来停止。当调用 `timeEnd` 时，它将在毫秒中记录开始和结束之间的经过时间。

这里是一个使用 `setTimeout` 的示例：

```js
console.time('simple-timer');
setTimeout(function () {
    console.timeEnd('simple-timer');
}, 500);
```

### trace

这会在控制台记录并包含堆栈跟踪：

```js
console.trace(message, […])
```

#### 描述

这与`console.log`的工作方式非常相似。第一个参数可以被视为格式化字符串，其他参数提供额外的输入。主要区别是`console.trace`在记录到控制台时会包含堆栈跟踪。

这里有一个简单的例子：

```js
console.trace('This should be the first line.');
```

## npm（Node 包管理器）

npm 不是一个 Node.js 模块。我们将探讨一些 npm 的功能和用法，因为几乎每个 Node.js 项目都会使用 npm。

大多数现代平台都有一种将执行特定功能或目的的代码分组在一起的方式，称为包。Node.js 使用 npm 来跟踪、更新、固定和安装这些包。

### init

这通过在当前目录中创建`package.json`来初始化一个 node 模块：

```js
npm init
```

#### 描述

一个交互式控制台会问你很多问题，并使用这些答案为你构建一个`package.json`文件。这是一个启动新模块的好方法。如果`package.json`文件存在，这将不会删除当前的`package.json`文件或任何当前属性。

### package.json

这是包含你项目所有信息的文件。

#### 描述

这不是一个函数或命令，但它是你项目中最重要的文件。它决定了关于你项目所有信息。虽然技术上只需要 name 和 version 这两个属性，但可以设置许多属性。如果你使用 npm 创建了这个文件，你将会有很多已经填写好的。列出所有可能性将会很繁琐。这里只列出一些最有用的：name、version、scripts、dependencies、devDependencies、authors 和 license。

npm 文档在[`www.npmjs.org/doc/files/package.json.html`](https://www.npmjs.org/doc/files/package.json.html)中详细介绍了所有设置及其用法。

### install

这是安装包的命令：

```js
npm install 
npm install [package] [@version] [--save | --save-dev]

```

#### 描述

这是安装新包及其依赖项的主要方式。如果没有参数调用`npm install`，它将使用`package.json`并安装所有依赖项。

这个命令还允许你通过执行包名来安装包。可以通过添加版本来增强这一点。如果省略版本，它将安装最新版本。此外，你可以使用 save 标志在 dependencies 或 devDependcies 中创建属性。

这不是`npm install`能做的全部列表，但这是最常用的列表。

### update

这将更新一个包到最新版本：

```js
npm update [package]
```

#### 描述

这个命令将更新所有包或特定包到最新版本。

### shrinkwrap

这将明确定义项目的所有依赖项：

```js
npm shrinkwrap
```

#### 描述

这与 `package.json` 中基本依赖列表不同。大多数包都有自己的要求。当安装一个包时，npm 将出去寻找与依赖指定的版本匹配的最新版本。这可能导致在运行不同时间时安装不同版本的包。这是大多数开发者想要避免的事情。

一种对抗这种情况的方法是运行 `npm shrinkwrap`。它将创建 `npm-shrinkwrap.json` 文件。这个文件将明确定义每个已安装的包递归安装的当前版本。这确保了当你再次运行 `npm install` 时，你会知道将安装哪些包版本。

### run

这将运行一个任意命令：

```js
npm run [script]

```

#### 描述

`package.json` 文件中有一个名为 scripts 的属性。这是一个对象，可以包含可以被 npm 运行的命令列表。这些脚本可以是任何可以通过命令行运行的命令。

在 npm 中有三个命令使用这些脚本对象。这些是 `npm test`、`npm start` 和 `npm stop`。这些命令分别映射到测试、启动和停止脚本对象。

这里是一个来自 `package.json` 的脚本对象示例以及从 `npm` 调用它的方法：

```js
package.json:
 "scripts": {
 "hey":  "echo hey"
}
npm run hey

```

## Stream

Stream 是许多内部对象使用的接口。每当需要读取或写入数据时，最有可能通过流来完成。这与 Node.js 的异步范式相符合。如果我们从文件系统中读取一个大文件，我们会创建一个监听器来告诉我们每个数据块何时准备好读取。如果这个文件来自网络、HTTP 请求或 `stdin`，这也不会改变。

在这本书中，我们只将涵盖使用流。流接口也可以用你自己的对象实现。

流可以是可读的、可写的，或者双工（两者都是）。我们将分别介绍可读流和可写流。

### Readable

这当然是一个我们可以从中获取数据的流。一个可读流可以在两种不同的模式中，流动模式或非流动模式。它处于哪种模式取决于监听哪些事件。

要将流置于流动模式，你只需监听数据事件。相反，要将流置于非流动模式，你必须监听可读事件，然后调用 `stream.read()` 函数来检索数据。理解这些模式的最简单方法是将数据视为许多块。在流动模式下，每次一个块准备好时，数据事件就会触发，你可以在数据事件的回调中读取该块。在非流动模式下，块会触发一个可读事件，然后你必须调用 read 来获取块。

这里是一个事件列表：

+   readable

+   data

+   end

+   close

+   错误

这里有两个例子，一个使用流动模式，一个使用非流动模式：

```js
var fs = require('fs');
var readable = fs.createReadStream('test.txt');

readable.on('data', function (chunk) {
    console.log(chunk.toString());
});
var fs = require('fs');
readable.on('readable', function () {
    var chunk;
   while (chunk = readable.read()) {
        console.log(chunk.toString());
    }
});
```

#### read

使用非流动的 readable 流：

```js
readable.read([size])
```

##### 返回值

这将返回字符串、缓冲区或 null。如果设置了编码，则返回字符串。如果没有设置编码，则返回缓冲区。最后，当流中没有数据时返回 null。

##### 描述

这从流中读取。大多数时候，可选参数大小是不需要的，应该避免使用。仅与非流动流一起使用。

这是一个简单的示例，用于读取文件：

```js
readable.on('readable', function () {
    var chunk;
   while (chunk = readable.read()) {
        console.log(chunk.toString());
    }
});
```

#### setEncoding

这设置了流的编码：

```js
stream.setEncoding(encoding)
```

##### 描述

默认情况下，可读流将输出一个缓冲区。这将设置缓冲区的编码，因此返回一个字符串。

这是一个使用`setEncoding`的打开示例：

```js
readable.setEncoding('utf8');
readable.on('readable', function () {
    var chunk;
   while (chunk = readable.read()) {
        console.log(chunk);
    }
});
```

#### 恢复和暂停

这些函数暂停和恢复流：

```js
stream.pause()
stream.resume()
```

##### 描述

暂停将停止流发出数据事件。如果在对非流动流调用此函数，它将被更改为流动流并暂停。恢复将使流再次开始发出数据事件。

这是一个示例，在读取之前暂停流几秒钟：

```js
readable.pause();
readable.on('data', function (chunk) {
    console.log(chunk.toString());
});
setTimeout(function () { readable.resume();}, 3000);
```

#### 管道

这允许你将可读流的输出发送到可写流的输入：

```js
readable.pipe(writable, [options])
```

##### 返回值

这返回可写流，以便可以链式管道。

##### 描述

这与在 shell 中管道输出完全相同。

一个很好的设计范例是从一个流管道到另一个转换流的流，然后再将其管道到输出。例如，你想通过网络发送文件。你会以可读流打开文件，将其传递给一个双工流，该流将压缩它，然后将压缩输出管道到套接字。

这里是一个简单的示例，将输出管道输出到`stdout`：

```js
var readable = fs.createReadStream('test.txt');
readable.pipe(process.stdout);
```

### 可写

这是数据流向的流。这有点简单，因为实际上只有两个重要的函数：写入和结束。

这里是它们触发时的事件和详细信息：

+   `drain`：当内部缓冲区已写入所有数据时触发

+   `finish`：当流已结束并且所有数据都已写入时触发

+   `error`：当发生错误时触发

### 注意

需要注意的是，流可以接收比它能及时写入更多的数据。这在写入网络流时尤其如此。正因为如此，`finish`和`drain`事件让程序知道数据已被发送。

#### write

这个函数写入流：

```js
writable.write(chunk, [encoding], [callback])
```

##### 返回值

如果流已完全写入，则返回一个布尔值。

##### 描述

这是可写流的主要功能。数据可以是缓冲区或字符串，编码默认为 UTF8，当当前数据块被写入时，将调用回调函数。

这是一个简单的示例：

```js
var fs = require('fs');
var writable = fs.createWriteStream('WriteStream.txt');

var hasWritten = writable.write('Write this!', 'utf8', function () {
    console.log('The buffer has written');
});
end
```

这将关闭流，并且无法再写入更多数据：

```js
writable.end([chunk], [encoding], [callback])
```

##### 描述

当你完成向流写入时，应该在它上调用 end 函数。所有参数都是可选的，块是在流结束之前可以写入的数据，编码默认为 UTF8，回调将附加到 finish 事件。

这里有一个结束可写流的例子：

```js
var fs = require('fs');
var writable = fs.createWriteStream('WriteStream.txt');

writable.end('Last data written.', 'utf8', function () {
    //this runs when everything has been written.
});
```

# `net`模块

Node.js 中的`net`模块允许我们创建网络连接。创建的连接将是我们可以写入和读取的流。本节将仅关注网络连接，而不是 HTTP。Node.js 有一个完整的 HTTP 模块，我们将在下一节中介绍。

所有这些函数都假设`net`模块已经以这种方式加载：

```js
var net = require('net');
```

## createServer

这个函数将创建一个 TCP 服务器：

```js
net.createServer([options], [listener]) 
```

### 返回值

这返回一个`net.Server`对象。

### 描述

这个函数允许我们监听连接。返回的对象将是一个`net.Server`对象，连接监听器将被传递给一个`net.Socket`对象。我们将在稍后介绍这两个对象。

这里有一个简单的例子，展示了我们如何监听并写入套接字。每个连接都必须手动关闭：

```js
var net = require('net');
var server = net.createServer(function (connection) {
    connection.write('You connected!');
});
server.listen(5000, function () { 
    console.log('Listening on port 5000');
});
```

## net.Server

我们现在将查看`net.Server`对象。下一节中的所有函数都需要通过`createServer`创建一个`Server`对象。

`net.Server`参数是一个`EventEmitter`，它将发出事件。以下是一个包含事件及其事件参数（如果有）的列表：

+   连接：`net.Socket`

+   `close`

+   错误：`Error`

+   `listening`

这里有一个使用所有事件的例子：

```js
var net = require('net');
var server = net.createServer();
server.on('listening', function () {
    console.log('I am listening');
});
server.on('connection', function (socket) {
    console.log(socket);
    socket.end();
    server.close();
});
server.on('error', function (err) {
    console.log(err);
});
server.on('close', function () {
    console.log('The server has stopped listening');
});
server.listen(5000);
```

### listen

这开始接受连接：

```js
server.listen(port, [host], [backlog], [callback])
```

#### 描述

创建服务器并不会让它开始监听请求。我们必须至少提供一个端口号给监听函数。主机是可选的，因为它将监听所有 IPv4 地址，而 backlog 将是连接队列。最后，当服务器开始监听时，回调函数将被调用。

你可能会得到`EADDRINUSE`错误。这仅仅意味着该端口已经被另一个进程使用。

这里有一个定义所有参数的例子：

```js
server.listen(5000, '127.0.0.1', 500, function () { 
    console.log('Listening on port 5000');
});
```

### close

这关闭当前服务器：

```js
server.close([callback])
```

#### 描述

这将停止服务器创建新的连接。这一点很重要，因为它不会关闭当前连接。

当没有更多连接时，回调将被调用。

### address

这获取端口号和地址：

```js
server.address()
```

#### 描述

这将给出服务器监听的端口号和 IP 地址。

### getConnections

这获取连接数量：

```js
server.getConnections(callback)
```

#### 返回值

这返回一个整数。

#### 描述

这不会给出每个连接的任何信息。它只返回数量。这是一个查看是否有连接的好方法。回调函数将需要是*function(err, connections)*的形式。

### connect

这可以轻松地创建到指定地址的连接：

```js
net.connect(port, [host], [connectListener])
net.createConnection(port, [host], [connectListener])
```

#### 返回值

这返回一个`net.Socket`对象。

#### 描述

这个函数并不做任何你不能用套接字做的事情。这是一个方便的函数，它返回一个`net.Socket`对象。

对于参数，需要一个端口号，主机将默认为 localhost，`connectListener`将被添加到新形成的`net.Socket`对象的连接事件中。

这里是一个示例，它将连接到我们刚刚创建的服务器，并且每秒发送数据：

```js
var net = require('net');
var server = net.createServer();
server.on('listening', function () {
    console.log('I am listening');
});
server.on('connection', function (socket) {
    socket.on('data', function (d) {
        console.log('from client: ' + d);
    });
});

server.listen(5000);

var client = net.connect({ port: 5000, host: 'localhost' }, function () {
    setInterval(function () {
        client.write('hey!');
    }, 1000);
});
```

### net.Socket

这一节将与`net.Server`部分类似。`net.Socket`参数是在建立连接时返回的对象。它也可以用来创建新的连接。

它是一个可读和可写流。这是从`net.Socket`发送和接收数据的唯一方式。

这里是事件列表以及它们触发时的详细信息：

+   `connect`：在连接时。

+   `data`：当接收到数据时。

+   `end`：当套接字结束时。

+   `timeout`：当套接字由于不活动而超时时。此时套接字仍然打开。

+   `drain`：当写入缓冲区中的所有数据都已发送时。

+   `Error`：在出错时。

+   `close`：当套接字关闭时。

由于`net.Socket`是一个可读流，没有读取函数。你必须监听数据事件以获取数据。

这里是一个使用套接字连接到本地服务器的示例：

```js
var server = net.createServer();
server.on('listening', function () {
    console.log('I am listening');
});
server.on('connection', function (socket) {
    socket.on('data', function (d) {
        console.log('from client: ' + d);
    });
});

server.listen(5000);

var client = new net.Socket();
client.on('connect', function () {
    setInterval(function () {
        client.write('Hey!');
    }, 1000);
});

client.connect(5000, 'localhost');
```

#### connect

这创建了一个连接：

```js
socket.connect(port, [host], [listener])
```

##### 描述

这是实际创建连接的函数。与`net.connection`类似，端口是必需的，主机默认为 localhost，监听器将函数映射到连接事件。

这里是一个本地连接并写入连接的示例：

```js
var client = new net.Socket();
client.on('connect', function () {
    setInterval(function () {
        client.write('Hey!');
    }, 1000);
});

client.connect(5000, 'localhost');
```

#### write

这是在套接字上发送数据：

```js
socket.write(data, [encoding], [callback])
```

##### 描述

套接字是缓冲的，因为很容易排队比可以通过网络发送的数据更多的数据。这是 Node.js 自动完成的，但你应该意识到这一点，因为缓冲会使用内存来保存要发送的数据。

编码参数默认为 UTF8，但可以使用我们讨论过的任何编码。数据写入套接字后，将调用回调函数。

这里是一个定义了所有参数的示例：

```js
var client = new net.Socket();
client.on('connect', function () {
    client.write('This is the data', 'utf8', function(){
        console.log('Data has been sent');
    });
});

client.connect(5000, 'localhost');
```

#### end

这开始关闭套接字的过程：

```js
socket.end([data], [encoding])
```

##### 描述

这实际上是关闭套接字。我们无法确定，但原因可能是服务器可能发送一些数据，尽管套接字将很快关闭。

在套接字关闭之前，你可以发送一些数据，这就是可选参数的作用：

```js
Here is an example that closes a socket.
var client = new net.Socket();
client.on('connect', function () {
    client.end('I am closing', 'utf8');
});

client.connect(5000, 'localhost');
```

# HTTP 模块

我们将介绍 HTTP 服务器模块。技术上，你可以使用`net`模块编写你的 HTTP 服务器，但你不必这样做。

其中一些函数与`net`模块的函数非常相似。这应该是有意义的，因为 HTTP 在本质上是一个网络服务器。

所有这些函数和对象也用于 HTTPS 模块。唯一的区别是，对于`createServer`和`https.request`的选项，你可以传递证书。

所有的以下示例都假设模块已经被加载：

```js
var http = require('http');
```

## createServer

这创建了一个 HTTP 服务器：

```js
http.createServer([requestListener])
```

### 返回值

这返回一个`http.Server`对象。

### 描述

与`net.createServer`类似，这是提供任何内容所必需的。`requestListener`参数附加到请求事件。

这里有一个简单的示例，它会在请求被制作时记录到控制台：

```js
var server = http.createServer(function (req, res) {
    console.log('Someone made a request!');
    res.end();
});
server.listen(8080);
```

## http.Server

这是`http.createServer`返回的服务器对象。这是将响应所有请求的对象。

我们将从函数开始，并单独查看每个事件，因为事件对于处理请求非常重要。

### 监听

这告诉服务器在提供的端口、路径或文件描述符上监听：

```js
server.listen(port, [host], [callback])
server.listen(path, [callback])
server.listen(fd, [callback])
```

#### 描述

虽然这个函数有三种不同的执行方式，但你最可能只会使用网络监听器。实际上，其他两个监听器在 Windows 上执行起来可能很困难，甚至不可能。让我们快速了解一下最后两个。

路径监听器将使用本地套接字服务器，并且文件描述符需要一个句柄。如果这听起来很陌生，这意味着你将使用另一种方法。

网络监听器要求使用端口。如果没有传入任何内容，主机将默认为 localhost。在所有函数中，都会将一个回调函数附加到监听事件。

这里是一个在定义了所有参数的网络端口上监听的示例：

```js
var server = http.createServer();
server.listen(8080, 'localhost', function(){
    console.log('The server is listening');
});
```

### 关闭

这将关闭服务器：

```js
server.close([callback])
```

#### 描述

这将停止服务器监听。回调函数将附加到关闭事件。

这里有一个简单的示例：

```js
var server = http.createServer();
server.listen(8080, 'localhost', function () {
    server.close(function () {
        console.log('Server has closed');
    });
});
```

### 事件

`http.Server`参数是一个`EventEmitter`对象。事件也是大多数工作将完成的地方。

#### 请求

当有请求进来时，此事件会被触发：

```js
server.on('request', function (req, res) { });
```

##### 描述

如果你只监听一个事件，这就是你要监听的事件。它包含`request`和服务器响应。`req`属性将是`http.IncomingMessage`，而`res`属性将是`http.ServerResponse`。在本节中，我们将查看这两个对象。此外，`req`实现了一个可读流接口，而`res`实现了一个可写流接口。

这里是一个监听请求的示例：

```js
server.on('request', function (req, res) {
    res.end();
    console.log('A request was received');
});
```

#### 关闭

当服务器关闭时，此事件会被触发：

```js
server.on('close', function () { });
```

#### 升级

当客户端发送 HTTP 升级时，此事件会被触发：

```js
server.on('upgrade', function (req, socket, buffer) { });
```

##### 描述

升级请求要求 Web 服务器更改协议。如果你正在实现 HTTP 之外的协议，你应该监听并处理此事件。一个很好的例子是`WebSocket`升级。

`req`属性是请求，套接字将是一个`net.Socket`，而`buffer`是一个 Buffer。

## IncomingMessage

这是监听请求事件或从`http.clientRequest`返回的请求对象。这是一个可读的流。

### 头部信息

请求的 HTTP 头信息：

```js
message.headers
```

#### 描述

有时，你可能需要根据头部信息中的信息做出决策。以下是一个使用头部信息检查基本身份验证的示例：

```js
server.on('request', function (req, res) {
    if (req.headers.authorization !== undefined)
        //do a check here
    res.end();
});
```

### 方法

这将获取请求的 HTTP 方法：

```js
message.method
```

#### 描述

这将返回一个字符串形式的函数，并且是大写的。以下是一个`GET`方法的示例：

```js
server.on('request', function (req, res) {
    if (req.method === 'GET')
        res.write(req.method);
    res.end();
});
```

### url

这是请求的 URL：

```js
message.url
```

#### 描述

这将是一个包含任何查询参数的 URL 字符串。你可以自己解析这个字符串，或者使用 Node 的查询字符串模块并使用`parse`函数。

下面是一个简单的示例，它将服务当前目录中的任何文件。请记住，这只是一个示例，并且没有进行错误检查：

```js
server.on('request', function (req, res) {
        var file = fs.createReadStream('.' + req.url);
        file.pipe(res);
});
```

### data

这是可读流接口的数据事件。

#### 描述

如果你有一个传入的消息，你很可能会想知道消息的内容。由于它是一个可读流，我们需要监听数据事件以获取所有数据。当数据耗尽时，将触发结束事件。

下面是一个创建数据事件和结束事件的监听器的示例：

```js
var data = '';
response.on('data', function (chunk) {
    console.log(chunk);
    data += chunk;
});

response.on('end', function () {
    console.log(data);
});
```

## ServerResponse

这是 HTTP 服务器为事件请求创建的响应。每个请求都需要一个响应，这就是它。这实现了可写接口。

### writeHead

这将写入 HTTP 响应头：

```js
response.WriteHead(statusCode, [headers])
```

#### 描述

这将为响应写入头。这个方法必须在`response.write`之前调用。如果不这样做，服务器将自动为你发送带有你设置的头的响应。

`statusCode`是响应的 HTTP 状态码。一个`header`是一个对象，其属性是头的名称，值是头的值。

下面是一个写入头的示例：

```js
server.on('request', function (req, res) {
    res.writeHead(200, { 'Content-Type': 'text/html' });
    res.end();
});
```

### statusCode

这设置响应的状态码：

```js
response.statusCode
```

#### 描述

这是在`response.writeHead`被调用后使用的方法。如果在这个方法被调用之后`writeHead`已经被执行，那么它将不会改变响应头。它必须替代`writeHead`来调用。

下面是一个使用`statusCode`的示例：

```js
server.on('request', function (req, res) {
    res.statusCode = 404;
    res.end();
});
```

### setHeader

这将写入特定的头：

```js
response.setHeader(name, value)
```

#### 描述

与`statusCode`必须替代`writeHead`一样，`setHeader`也必须替代`writeHead`来调用。这个方法可以多次调用以设置多个头。

下面是使用`statusCode`和`setHeader`一起使用的示例：

```js
server.on('request', function (req, res) {
    res.statusCode = 200;
    res.setHeader('Content-Type', 'text/html');
    res.setHeader('Custom-Header', 'Custom-Value');
    res.end();
});
```

### write

这是写入响应体的函数：

```js
response.write(chunk, [encoding])
```

#### 描述

`response.write`参数是一个可写流，因此可以使用这个接口。一个块可以是一个缓冲区或字符串。编码是可选的，因为它将默认为 UTF8。下面是一个写入简单 HTML 页面的示例：

```js
server.on('request', function (req, res) {
    res.statusCode = 200;
    res.setHeader('Content-Type', 'text/html');
    res.write('<html><body><h1>Hello!</h1></body></html>');
    res.end();
});
```

### end

这结束响应：

```js
response.end([data], [encoding])
```

#### 描述

`response`参数是一个可写流，因此当我们完成写入时，我们必须结束流。数据是需要写入的任何可选数据，编码将默认为 UTF8。所有的响应示例都使用了`res.end`。如果你不结束响应，浏览器将等待来自服务器的响应。

## http.request

这使用 HTTP 发起请求：

```js
http.request(options, [callback])
```

### 返回值

这返回一个`http.ClientRequest`对象。

### 描述

Node.js 允许你消费 HTTP 请求以及提供 HTTP 服务。选项对象有许多可以设置的属性。以下是一个列表：

+   `host`

+   `hostname`

+   `port`

+   `localAddress`

+   `socketPath`

+   `method`

+   `path`

+   `headers`

+   `auth`

+   `agent`

并非每个请求都需要每个选项。大多数时候，只需要 `hostname`、`port`、`path` 和 `method` 来进行请求。选项参数也可以是一个字符串形式的 URL。

回调将被附加到响应事件。请求对象是一个可写流，因此可以向服务器发送数据。以下是一个向 Packt Publishing 发送请求的示例：

```js
var request = http.request({
    host: 'www.packtpub.com',
    port: 80,
    path: '/',
    method: 'GET'
}, function (res) {
    console.log(res);
});

request.end();
```

## http.get

这是 `GET` 请求的便捷方法：

```js
http.get(options, [callback])
```

### 返回值

这返回一个 `http.ClientRequest` 对象。

### 描述

它的工作方式与 `http.request` 类似，但会自动发送一个空请求并调用结束函数。如果你只进行 `GET` 请求，这可以节省一些样板代码。

这里是一个请求 Packt Publishing 的示例：

```js
http.get('http://www.packtpub.com', function (res) {
    console.log(res);
});
```

## http.clientRequest

这是 `http.request` 返回的对象。

### write

这在请求中向服务器写入：

```js
request.write(data, [encoding])
```

#### 描述

`http.clientRequest` 属性是一个可写流。当你需要向远程服务器发送数据时，它可以被写入，例如，在执行 `POST` 请求时。

它与所有其他可写流的工作方式相同，因此数据可以是缓冲区或字符串，编码将默认为 UTF8。

### end

这结束了请求：

```js
request.end([data], [encoding])
```

#### 描述

当你向流写入数据时，你必须使用此函数结束流。如果不这样做，连接将保持并/或超时。数据是可选的，可以是缓冲区或字符串，而编码将默认为 UTF8。

### 响应

这是响应事件。它让你知道远程服务器已经响应：

```js
request.on('response', function(response){})
```

#### 描述

当远程服务器响应时，此事件会被触发。回调中的响应对象将是 `http.incomingMessage`。

如果没有添加响应处理程序，服务器响应将被丢弃。如果有响应处理程序，那么 Node.js 将开始将响应缓冲到内存中。如果你不将其读回，可能会导致服务器崩溃。

这里是一个读取响应数据的示例监听器：

```js
var request = http.request({host: 'www.google.com', path: '/', port: 80, method: 'GET'});

request.on('response', function (response) {
    var data = '';
    response.on('data', function (chunk) {
        console.log(chunk);
        data += chunk;
    });

    response.on('end', function () {
        console.log(data);
    });
});
```
