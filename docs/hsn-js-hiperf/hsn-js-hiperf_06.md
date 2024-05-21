# 第六章：消息传递 - 了解不同类型

在上一章中，我们看了 Node.js 和我们需要创建服务器端应用程序的基本环境。现在，我们将看看如何利用我们之前学习的通信技术来编写可扩展的系统。消息传递是应用程序解耦但仍然能够共同工作的一种很好的方式。这意味着我们可以创建相互独立工作的模块，无论是通过进程还是线程，仍然可以实现共同的目标。

在本章中，我们将涵盖以下主题：

+   使用 net 模块进行本地通信

+   利用网络

+   快速浏览 HTTP/3

我们还将在查看正在开发的 HTTP/3 标准时，了解客户端/服务器通信的未来。然后，我们将查看 QUIC 协议的实现，这是由 Google 开发的协议，HTTP/3 从中汲取了一些想法。

让我们开始吧！

# 技术要求

对于本章，您将需要以下技术要求：

+   一个 IDE 或代码编辑器（首选 VS Code）

+   运行中的 Node.js 环境

+   OpenSSL 或安装 Cygwin 的能力

+   本章的代码可以在[`github.com/PacktPublishing/Hands-On-High-Performance-Web-Development-with-JavaScript/tree/master/Chapter06`](https://github.com/PacktPublishing/Hands-On-High-Performance-Web-Development-with-JavaScript/tree/master/Chapter06)找到。

# 使用 net 模块进行本地通信

虽然许多应用程序可以在单个线程上运行并利用事件循环来运行，但当我们编写服务器应用程序时，我们将希望尽量利用我们可用的所有核心。我们可以通过使用**进程**或**线程**来实现这一点。在大多数情况下，我们将希望使用线程，因为它们更轻量级且启动速度更快。

我们可以根据我们是否需要在主系统死机后仍然运行子系统来确定我们是否需要进程还是线程。如果我们不在乎，我们应该利用线程，但如果我们需要在主进程死机后仍然运行该子系统，我们应该利用一个解耦的进程。这只是考虑何时使用进程或线程的一种方式，但它是一个很好的指标。

在浏览器和 Node.js 中，我们有 Web Workers 来代替传统系统中的线程。虽然它们与其他语言的线程有许多相同的概念，但它们无法共享状态（在我们的情况下，这是首选）。

有一种方法可以在 worker 之间共享状态。这可以通过`SharedArrayBuffer`来实现。虽然我们可以利用这一点来共享状态，但我们要强调事件系统和 IPC 几乎总是足够快，可以在不同的部分之间移动状态和协调。此外，我们不必处理锁等概念。

要启动一个 worker，我们需要调用`new Worker(<script here>)`。让我们来看看这个概念：

1.  创建一个名为`Main_Worker.js`的文件，并将以下代码添加到其中：

```js
import { Worker } from 'worker_threads';

const data = {what: 'this', huh: 'yeah'};
const worker = new Worker('./worker.js');
worker.postMessage(data);
worker.on('message', (data) => {
    worker.terminate();
});
worker.on('exit', (code) => {
    console.log('our worker stopped with the following code: ', 
     code);
});
```

1.  创建一个名为`worker.js`的文件，并将以下代码添加到其中：

```js
import { parentPort } from 'worker_threads'

parentPort.on('message', (msg) => {
    console.log('we received a message from our parent: ', msg);
    parentPort.postMessage({RECEIVED: true});
});
```

正如我们所看到的，这个系统与浏览器中的系统类似。首先，我们从`worker_threads`模块中导入 worker。然后，我们启动它。线程将启动，这意味着我们可以向其发送消息并监听事件，就像我们在上一章中能够与进程一样。

在`worker.js`文件中，我们从`worker_threads`模块中导入`parentPort`消息通道。我们监听并传递消息的方式与父级相同。一旦我们收到消息，我们就会声明我们收到了消息。然后父级终止我们，我们打印出我们已经被终止。

现在，如果我们想要紧密耦合所有子系统，这种形式的消息传递是完全可以的。但是，如果我们希望不同的线程有不同的工作，该怎么办？我们可以有一个只为我们缓存数据的线程。另一个可能为我们发出请求。最后，我们的主线程（起始进程）可以移动所有这些数据并从命令行中接收数据。

要做到所有这些，我们可以简单地使用内置系统。或者，我们可以利用我们在上一章中看到的机制。这不仅使我们拥有高度可扩展的系统，还允许我们将这些各个子系统从线程转换为进程，如果需要的话。这也允许我们在需要时用另一种语言编写这些单独的子系统。现在让我们来看一下：

1.  让我们继续制作这个系统。我们将创建四个文件：`main.js`，`cache.js`，`send.js`和`package.json`。我们的`package.json`文件应该看起来像这样：

```js
{
    "name" : "Chapter6_Local",
    "version" : "0.0.1",
    "type" : "module",
    "main" : "main.js"
}
```

1.  接下来，将以下代码添加到`cache.js`文件中：

```js
import net from 'net';
import pipeName from './helper.js';

let count = 0;
let cacheTable = new Map();
// these correspond to !!!BEGIN!!!, !!!END!!!, !!!GET!!!, and 
// !!!DELETE!!! respectively
const begin, end, get, del; //shortened for readability they will use the Buffer.from() methods
let currData = [];

const socket = new net.Socket().connect(pipeName());
socket.on('data', (data) => {
    if( data.toString('utf8') === 'WHOIS' ) {
        return socket.write('cache');
    }
    if( data.includes(get) ) {
        const loc = parseInt(data.slice(get.byteLength).toString('utf8'));
        const d = cacheTable.get(loc);
        if( typeof d !== 'undefined' ) {
            socket.write(begin.toString('utf8') + d + 
             end.toString('utf8'));
        }
    }
    if( data.includes(del) ) {
        if( data.byteLength === del.byteLength ) {
            cacheTable.clear();
        } else {
            const loc = parseInt(data.slice(del.byteLength).toString('utf8'));
            cacheTable.delete(loc);
        }
    }
    if( data.includes(begin) ) {
        currData.push(data.slice(begin.byteLength).toString('utf8'));
    }
    if( currData.length ) {
        currData.push(data.toString('utf8'));
    }
    if( data.includes(end) ) {
        currData.push(data.slice(0, data.byteLength - 
         end.byteLength).toString('utf8'));
        cacheTable.set(count, currData.join(''));
        currData = [];
    }
});
```

这绝对不是处理流数据的万无一失的机制。`!!!BEGIN!!!`和其他命令消息可能会被分块，我们永远看不到它们。虽然我们保持简单，但要记住，生产级别的流处理需要处理所有这些类型的问题。

`cache`子模块检查消息上的不同标头。根据每种类型，我们将执行该类型的操作。这可以被视为一种简单的远程过程调用。以下列表描述了我们根据每个事件所做的操作：

+   `!!!BEGIN!!!`：我们需要开始监听线路上的更多数据，因为这意味着我们将存储数据。

+   `!!!END!!!`：一旦我们看到这条消息，我们就可以将所有这些数据放在一起并根据缓存中的计数存储它。

+   `!!!GET!!!`：我们将尝试获取由服务器提供给我们的编号位置存储的文件。

+   `!!!DELETE!!!`：如果消息的长度与此字符串一样长，这意味着我们想要从缓存中删除所有内容。否则，我们将尝试删除稍后在消息中指定的位置的数据。

1.  将以下代码添加到`send.js`文件中：

```js
import net from 'net'
import https from 'https'
import pipeName from './helpers.js'

const socket = new net.Socket().connect(pipeName());
socket.on('data', (data) => {
    if( data.toString('utf8') === 'WHOIS' ) {
        return socket.write('send');
    }
    const all = [];
    https.get(data.toString('utf8'), (res) => {
        res.on('data', (data) => {
            all.push(data.toString('utf8'));
        });
        res.on('end', () => {
            socket.write('!!!BEGIN!!!');
            socket.write(all.join(''));
            socket.write('!!!END!!!');
        });
    }).on('error', (err) => {
        socket.write('!!!FALSE!!!');
    });
    console.log('we received data from the main application',  
     data.toString('utf8'));
});
```

对于我们拥有的每个子模块，我们处理可能通过网络传输的特定命令。正如`send`子模块所示，我们处理除了`WHOIS`命令之外的任何网络传输，该命令告诉主应用程序谁连接到它。我们尝试从指定的地址获取文件并将其写回主应用程序，以便将其存储在缓存中。

我们还添加了我们自己的*协议*来发送数据。虽然这个系统并非万无一失，我们应该添加某种类型的锁定（比如一个布尔值，这样我们在完全发送当前数据之前不会尝试接收更多数据），但它展示了我们如何在系统中发送数据。在第七章中，*流-理解流和非阻塞 I/O*，我们将看到一个类似的概念，但我们将利用流，这样我们就不会在每个线程中使用太多内存。

正如我们所看到的，我们只导入了`https`模块。这意味着我们只能向通过 HTTPS 提供的地址发出请求。如果我们想要支持 HTTP，我们将不得不导入`http`模块，然后检查用户输入的地址。在我们的情况下，我们尽可能地简化了它。

当我们想要发送数据时，我们发送`!!!BEGIN!!!`消息，以让接收方知道我们将发送无法适应单个帧的数据。然后，我们用`!!!END!!!`消息结束我们的消息。

如果我们无法读取我们尝试抓取的端点或者我们的连接超时（这两种情况都会进入错误条件），我们将发送`!!!FALSE!!!`消息，以让接收方知道我们无法完全传输数据。

在几乎所有数据传输系统中，都使用了将我们的数据包装在*帧*中的概念。没有帧，我们将不得不发送一个标头，说明数据传输的大小。然而，这意味着我们需要在发送之前知道内容的大小。帧给了我们选择不发送消息的长度的选项，因此我们可以处理无限大的消息。

在任何地方都会对数据进行包装或装箱。例如，如果我们看一下如何创建数据包，这个概念仍然适用。理解这个概念是理解通信堆栈的较低层次的关键。另一个需要了解的概念是，并非所有数据都一次性发送。它是分批发送的。一次可以发送的数量通常在操作系统级别设置。我们可以设置的唯一属性之一是流的`highWaterMark`属性。该属性允许我们说出在停止读取/写入之前我们将在内存中保存多少数据。

缓存应用程序类似于发送子模块，只是它响应更多的命令。如果我们收到一个`get`命令，我们可以尝试从缓存中获取该项并将其发送回主模块；否则，我们只是发送回`null`。如果我们收到一个`delete`命令，如果没有其他参数，我们将删除整个缓存；否则，我们将删除特定位置的项目。最后，如果我们收到开始或结束包装，我们将处理数据并将其缓存。

目前，我们的缓存是无限增加的。我们可以很容易地添加一个允许数据在缓存中停留的一定时间阈值（**生存时间**或**TTL**），或者只保留一定数量的记录，通常通过利用**最近最少使用**（**LRU**）销毁系统。我们将看看如何在第九章中实现缓存策略，*实际示例 - 构建静态服务器*。只需注意，这些概念在缓存和缓存策略中是非常普遍的。

回到代码中，创建`main.js`并添加以下代码：

1.  为我们的状态变量创建占位符。这些对应于我们的消息可能处于的各种状态以及通过套接字传递的数据：

```js
// import required modules and methods
const table = new Map();
let currData = [];
// These three items correspond to the buffers for: !!!FALSE!!!, 
// !!!BEGIN!!!, and !!!END!!! respectively
const failure, begin, end;
const methodTable = new WeakMap();
```

1.  创建处理通过我们的缓存传入的数据的方法：

```js

const cacheHandler = function(data) {
    if( data.includes(begin) || currData.length ) {
        currData.push(data.toString('utf8'));
    }
    if( data.includes(end) ) {
        currData.push(data.toString('utf8'));
        const final = currData.join('');
        console.log(final.substring(begin.byteLength, 
         final.byteLength - end.byteLength));
        currData = [];
    }
}
```

1.  接下来，添加一个方法来处理我们的`send`工作进程发送的消息：

```js

const sendHandler = function(data) {
    if( data.equals(failure) ) { //failure }
    if( data.includes(begin) ) { 
     currData.push(data.toString('utf8')); }
    if( currData.length ) { currData.push(data.toString('utf8')); }
    if( data.includes(end) ) { 
        table.get('cache').write(currData.join(''));
        currData = [];
    }
}
```

1.  创建两个最终的辅助方法。这些方法将测试我们拥有的工作进程数量，以便知道何时准备就绪，另一个将向每个工作进程套接字添加方法处理程序：

```js

const testConnections = function() {
    return table.size === 2;
}
const setupHandler = function() {
    table.forEach((value, key) => {
        value.on('data', methodTable.get(value.bind(value));
    });
}
```

1.  最终的大型方法将处理我们通过命令行接收到的所有消息：

```js
const startCLIMode = function() {
    process.stdin.on('data', function(data) {
        const d = data.toString('utf8');
        const instruction = d.trim().split(/\s/ig);
        switch(instruction[0]) {
            case 'delete': {
                table.get('cache').write(`!!!DELETE!!!${instruction[1] || ''}`);
                break; }
            case 'get': {
                if( typeof instruction[1] === 'undefined' ) {
                    return console.log('get needs a number 
                     associated with it!');
                }
                table.get('cache').write(`!!!GET!!!${instruction[1]}`);
                break; }
            case 'grab': {
                table.get('send').write(instruction[1]);
                break; }
            case 'stop': {
                table.forEach((value, key) => value.end());
                process.exit();
                break; }
    }});
}
```

1.  最后，创建服务器并启动工作进程：

```js
const server = net.createServer().listen(pipeName());
server.on('connection', (socket) => {
    socket.once('data', (data) => {
        const type = data.toString('utf8');
        table.set(type, socket);
        if( testConnections() ) {
            setupHandlers();
            startCLIMode();
        }
    });
    socket.once('close', () => {
        table.delete(type);
    });
    socket.write('WHOIS');
});

const cache = new Worker('./cache.js');
const send = new Worker('./send.js');
```

为了缩短本书中的代码量，主文件的某些部分已被删除。完整的示例可以在本书的 GitHub 存储库中找到。

在这里，我们有一堆辅助程序，将处理来自缓存和发送子系统的消息。我们还将套接字映射到我们的处理程序。使用`WeakMap`的好处是，如果这些子系统崩溃或以某种方式被移除，我们就不需要清理。我们还将子系统的名称映射到套接字，以便我们可以轻松地向正确的子系统发送消息。最后，我们创建一个服务器并处理传入的连接。在我们的情况下，我们只想检查两个子系统。一旦我们看到两个，我们就启动我们的程序。

我们包装消息的方式存在一些缺陷，测试连接数量以查看我们是否准备就绪也不是处理程序的最佳方式。然而，这确实使我们能够创建一个相当复杂的应用程序，以便我们可以快速测试这里所见的想法。有了这个应用程序，我们现在能够从远程资源缓存各种文件，并在需要时获取它们。这是一种类似于某些静态服务器工作方式的系统。

通过查看前面的应用程序，很容易看出我们可以利用本地连接来创建一个只使用核心 Node.js 系统的消息传递系统。同样有趣的是，我们可以将`listen`方法的参数从管道名称替换为端口号，这样我们就可以将这个应用程序从使用命名管道/Unix 域套接字转换为使用 TCP 套接字。

在 Node.js 中有这些工作线程之前，我们必须用进程将所有东西分开。起初，我们只有 fork 系统。当我们开始创建更多的进程时，这使得一些系统变得非常复杂。为了帮助我们理解这个概念，创建了`cluster`模块。使用`cluster`模块，更容易管理主/从架构中的进程。

# 了解 cluster 模块

虽然`cluster`模块可能不像过去那样经常使用，因为我们在 Node.js 中有工作线程，但仍有一个概念使其强大。我们能够在应用程序中的各个工作线程之间共享服务器连接。我们的主进程将使用一种策略，以便我们只向其中一个从进程发送请求。这使我们能够处理许多同时运行在完全相同的地址和端口上的连接。

有了这个概念，让我们利用`cluster`模块来实现前面的程序。现在，我们将确保发送和缓存子系统与主进程绑定。我们的子进程将负责处理通过我们的服务器传入的请求。要记住的一件事是，如果父进程死亡，我们的子进程也会死亡。如果我们不希望出现这种行为，在我们的主进程内调用 fork 时，我们可以传递`detached : true`选项。这将允许工作线程继续运行。这通常不是我们在使用`cluster`模块时想要的行为，但知道它是可用的是很好的。

我们已将以下程序分成更易管理的块。要查看完整的程序，请转到本章的代码存储库。

现在，我们应该能够编写一个类似于我们的 IPC 程序的程序。让我们来看一下：

1.  首先，我们将导入在`cluster`模式下实现我们之前示例所需的所有 Node 模块：

```js
import cluster from 'cluster';
import https from 'https';
import http from 'http';
import { URL } from 'url';
```

1.  接下来，我们设置可以在各个进程中使用的常量：

```js
const numWorkers = 2;
const CACHE = 0;
const SEND = 1;
const server = '127.0.0.1';
const port = 3000;
```

1.  然后，我们添加一个`if/else`检查，以查看我们是主进程还是从进程。同一文件用于两种类型的进程，因此我们需要一种区分两者的方法：

```js
if( cluster.isMaster ) {
    // do master work
} else {
    // handle incoming connections
}
```

1.  现在，编写主代码。这将进入`if/else`语句的第一个块中。我们的主节点需要启动从节点，并初始化我们的缓存：

```js
let count = 1; //where our current record is at. We start at 1
const cache = new Map();
for(let i = 0; i < numWorkers; i++ ) {
    const worker = cluster.fork();
    worker.on('message', (msg) => {
        // handle incoming cache request
    });
}
```

1.  添加一些代码来处理每个请求，就像我们在之前的例子中所做的那样。记住，如果我们停止主进程，它将销毁所有从进程。如果我们收到`STOP`命令，我们将只杀死主进程：

```js
// inside of the worker message handler
switch(msg.cmd) {
    case 'STOP': {
        process.exit();
        break;
    }
    case 'DELETE': {
        if( msg.opt != 'all' ) {
            cache.delete(parseInt(msg.opt);
        } else {
            cache.clear();
        }
        worker.send({cmd : 'GOOD' });
        break;
    }
    case 'GET': {
        worker.send(cache.get(parseInt(msg.opt));
        break;
    }
    case 'GRAB': {
        // grab the information
        break;
    }
}
```

1.  编写`GRAB` case 语句。为此，利用`https`模块请求资源：

```js
// inside the GRAB case statement
const buf = [];
https.get(msg.opt, (res) => {
    res.on('data', (data) => {
        buf.push(data.toString('utf8'));
    });
    res.on('end', () => {
        const final = buf.join('');
        cache.set(count, final);
        count += 1;
        worker.send({cmd : 'GOOD' });
    });
});
```

现在，我们将编写从节点代码。所有这些将保存在`else`块中。记住我们可以在从节点之间共享相同的服务器位置和端口。我们还将通过传递给我们的 URL 的搜索参数来处理所有传入的请求。这就是为什么我们从`url`模块导入了`URL`类。让我们开始吧：

1.  通过启动一个`HTTP`服务器来启动从节点代码。记住它们将共享相同的位置和端口：

```js
// inside of the else block
http.Server((req, res) => {
    const search = new URL(`${location}${req.url}`).searchParams;
    const command = search.get('command');
    const params = search.get('options');
    // handle the command
    handleCommand(res, command, params);
}).listen(port);
```

1.  现在，我们可以处理传递给我们的命令。这将类似于我们之前的例子，只是我们将通过**进程间通信**（**IPC**）与主进程交谈，并通过 HTTP/2 服务器处理请求。这里只显示了`get`命令；其余内容可以在本章的 GitHub 存储库中找到：

```js
const handleCommand = function(res, command, params=null) {
    switch(command) {
        case 'get': {
            process.send({cmd: 'GET', opt : params});
            process.once('message', (msg) => {
                res.writeHead(200, { 'Content-Type' : 'text/plain' });
                res.end(msg);
            });
            break;
        }
    }
}
```

在这里，我们可以看到两个工作进程都创建了一个`HTTP`服务器。虽然它们都创建了独立的对象，但它们共享底层端口。这对我们来说完全是隐藏的，但这是通过`cluster`模块完成的。如果我们尝试使用自己的版本来做类似的事情，同时使用`child_process`的 fork 方法，我们将会收到一个错误，指出`EADDRINUSE`。

如果我们请求以 HTML 格式存储的数据，我们将看到它以纯文本形式返回。这涉及到`writeHead`方法。我们告诉浏览器我们正在写`text/plain`。浏览器接收这些信息并利用它来查看它需要如何解析数据。由于它被告知数据是纯文本，它将只是在屏幕上显示它。如果我们在获取 HTML 数据时将其更改为`text/html`，它将解析并尝试呈现它。

通过这两种方法，我们能够编写能够充分利用系统上所有核心的程序，同时仍然能够协同工作。第一种架构为我们提供了一个良好的解耦系统，是大多数应用程序应该编写的方式，但`cluster`模块为我们提供了一个处理服务器的好方法。通过结合这两种方法，我们可以创建一个高吞吐量的服务器。在 Node.js 中构建这些客户端/服务器应用程序可能很容易，但也有一些需要注意的事项。

# 新开发人员常见的陷阱

在使用 Unix 域套接字/Windows 命名管道时，这两个系统之间存在一些差异。Node.js 试图隐藏这些细节，以便我们可以专注于我们想要编写的应用程序，但它们仍然会显现出来。新开发人员可能会遇到的两个最常见的问题是：

+   Windows 命名管道在应用程序退出时会自动销毁。Unix 域套接字则不会。这意味着当我们退出应用程序时，我们应该尝试使用`fs`模块，并通过`unlink`或`unlinkSync`方法取消链接文件。我们还应该在启动时检查它是否存在，以防我们不能正常退出。

+   Windows 的数据帧可能比 Unix 域套接字大。这意味着一个应用程序在 Windows 上可能正常工作，但在 Unix 系统上会失败。这就是我们创建我们所做的数据帧系统的原因。特别是当我们可能想要使用外部库来处理构建 IPC 系统的部分时，要牢记这一点是很重要的，因为一些系统并没有考虑到这一点，因此可能会很容易出现错误。

Node.js 的目标是完全跨操作系统兼容，但这些系统在实际跨系统操作时总是有一些小问题。如果我们想要确保它能够正常工作，就像我们必须在不能保证我们的最终用户将使用什么浏览器一样，那么我们需要在所有系统上进行测试。

虽然开发跨越单台计算机的服务器应用程序很常见，但我们仍然需要连接所有这些应用程序。当我们不能再使用单台计算机时，我们需要通过网络进行通信。接下来我们将看看这些协议。

# 利用网络

构建能够在同一台机器上相互通信的应用程序可能很酷，但最终我们需要与外部系统进行通信。在我们的情况下，大多数这些系统将是浏览器，但它们也可能是其他服务器。由于我们无法通过这些通道使用命名管道/Unix 域套接字，我们需要使用各种网络协议。

从技术上讲，我们仍然可以通过使用共享驱动器/文件系统共享来跨服务器使用前面两个概念，但这不是一个好主意。我们已经表明我们可以将`listen`方法从指向文件更改为指向端口。在最坏的情况下，我们可以使用共享文件系统，但这远非最佳选择，应该转换为使用我们将在这里介绍的协议之一。

我们将重点关注两种低级协议，即传输控制协议（TCP）和用户数据报协议（UDP）。我们还将研究网络的高级协议：超文本传输协议版本 2（HTTP/2）。通过这些协议，我们将能够创建高度可用的应用程序，可以通过网络访问。

# TCP/UDP

TCP 和 UDP 是 Node.js 中我们可以访问的两种低级网络协议。这两种协议都允许我们发送和接收消息，但它们在一些关键领域有所不同。首先，TCP 需要连接的接收方和发送方。因此，我们不能只在一个通道上广播，而不关心是否有人在听。

其次，除了 TCP 需要握手过程外，它还为我们提供了可靠的传输。这意味着我们知道当我们发送数据时，它应该到达另一端（显然，这也有失败的可能，但我们不打算讨论这个）。最后，TCP 保证了传递的顺序。如果我们在一个通道上向接收方发送数据，它将按照我们发送的顺序接收数据。因为这些原因，当我们需要保证传递和顺序时，我们使用 TCP。

实际上，TCP 并不一定需要按顺序发送数据。所有数据都是以数据包的形式发送的。它们实际上可以发送到不同的服务器，路由逻辑可能意味着后续数据包会比后来的更早到达接收方。然而，我们接收方的网络卡会为我们重新排序它们，使得看起来我们是按顺序接收它们的。TCP 还有许多其他很酷的方面，包括数据的传输，这些都超出了本书的范围，但任何人都可以查阅网络并了解更多这些概念以及它们是如何实现的。

话虽如此，TCP 似乎是我们总是想要使用的东西。为什么我们不使用能够保证传递的东西呢？此外，如果我们可以遍历所有当前的连接并将数据发送给每个人，我们就不需要广播。然而，由于所有这些保证，这使得 TCP 更加沉重和缓慢。这对于我们需要尽快发送数据的系统来说并不好。对于这种类型的数据传输，我们可以利用 UDP。UDP 给我们提供了一种称为无状态传输的东西。无状态传输意味着我们可以在一个通道上发送数据，它会将数据发送出去然后忘记。我们不需要连接到一个地址；相反，我们可以直接发送数据（只要没有其他人绑定到该地址和端口）。我们甚至可以建立一个多播系统，任何人都可以收听该地址，它可能会接收到数据。

这种类型的传输希望/需要的一些领域如下：

+   发送股票交易的买卖订单。由于数据传输速度很快，我们只关心最新的信息。因此，如果我们没有收到一些买卖订单，也并不重要。

+   视频游戏中的玩家位置数据。我们只能以有限的速度更新游戏。如果我们已经知道玩家移动的方向和速度，我们还可以插值或推断玩家在屏幕上的位置。因此，我们可以以任何速率接收玩家位置，并计算出他们应该在哪里（这有时被称为服务器的 tick 率）。

+   电信数据并不一定在乎我们发送的所有数据，只要我们发送了大部分数据即可。我们不需要保证完整视频/音频信号的传递，因为我们仍然可以用大部分数据获得很好的画面。

这只是 UDP 发挥作用的一些领域。通过对这两种系统的理解，我们将通过构建一个高度简化和不切实际的股票应用程序来研究它们。行为将如下所示：

1.  服务器将发布新的股票代码和可用股票数量。然后，它将在已知端口上通过 UDP 向所有人广播信息。

1.  服务器将存储与客户持仓相关的所有信息。这样，客户端就无法操纵他们可能拥有的股票数量。

1.  客户端将向服务器发送买入或卖出订单。服务器将确定它是否能处理该请求。所有这些流量都将通过 TCP 进行，因为我们需要确保知道服务器收到了我们的消息。

1.  服务器将以错误或成功的消息作出回应，告诉客户端他们的订单已更新。

1.  服务器将通过 UDP 通道广播股票的买入或卖出发生了。

这个应用程序看起来如下：

```js
import dgram from 'dgram';
import { Socket } from 'net';
const multicastAddress = '239.192.0.0';
const sendMessageBadOutput = 'message needs to be formatted as follows: BUY|SELL <SYMBOL> <NUMBER>';
const recvClient = dgram.createSocket({type : 'udp4', reuseAddr: true }); //1.
const sendClient = new Socket().connect(3000, "127.0.0.1");
// receiving client code seen below
process.stdin.setEncoding('utf8');
process.stdin.on('data', (msg) => {
    const input = msg.split(' ');
    if( input.length !== 3 ) {
        console.log(sendMessageBadOutput);
        return;
    }
    const num = parseInt(input[2]);
    if( num.toString() === 'NaN' ) {
        console.log(sendMessageBadOutput);
        return;
    }
    sendClient.write(msg);
});
sendClient.on('data', (data) => {
    console.log(data.toString('utf8'));
});
```

前面的大部分程序应该是熟悉的，除了我们正在使用的新模块：`dgram`模块。这个模块允许我们在使用 UDP 时发送数据。

在这里，我们创建了一个使用 UDP4（IPv4 上的 UDP，或者我们通常知道的 IP 地址）的套接字。我们还声明我们正在重用地址和端口。我们这样做是为了在本地测试。在其他情况下我们不希望这样做：

```js
recvClient.on('connect', () => {
    console.log('client is connected to the server');
});
recvClient.on('message', (msg) => {
    console.log('client received message', msg.toString('utf8'));
});
recvClient.bind(3000, () => {
    recvClient.addMembership(multicastAddress);
});
```

我们绑定到端口`3000`，因为服务器将在那里发送数据。然后，我们声明我们要将自己添加到多播地址。为了使多播工作，服务器需要通过多播地址发送数据。这些地址通常是操作系统设置的特定地址。每个操作系统都可以决定使用哪些地址，但我们选择的地址应该在任何操作系统上都可以使用。

一旦我们收到消息，我们就打印出来。再次，这应该看起来很熟悉。Node.js 是基于事件和流的，它们通常以相同的名称命名以保持一致性。

这个程序的其他部分处理用户输入，然后通过我们创建新套接字时打开的 TCP 通道发送数据（这应该类似于我们之前的 IPC 程序，只是我们传递了一个端口和一个 IP 地址）。

这个应用程序的服务器涉及的内容更多，因为它包含了股票应用程序的所有逻辑。我们将把这个过程分解为几个步骤：

1.  创建一个名为`main.js`的文件，并将`dgram`和`net`模块导入其中：

```js
import dgram from 'dgram';
import net from 'net';
```

1.  为我们的多播地址、错误消息和股票代码和客户端的`Maps`添加一些常量：

```js
const multicastAddress = '239.192.0.0';
const badListingNumMessage = 'to list a new ticker the following format needs to be followed <SYMBOL>
<NUMBER>';
const symbolTable = new Map();
const clientTable = new Map();
```

1.  接下来，我们创建两个服务器。第一个用于监听 UDP 消息，而第二个用于接收 TCP 消息。我们将利用 TCP 服务器来处理客户端请求。TCP 是可靠的，而 UDP 不是：

```js
const server = dgram.createSocket({type : 'udp4', reuseAddr : true}).bind(3000);
const recvServer = net.createServer().listen(3000, '127.0.0.1');
```

1.  然后，我们需要在 TCP 服务器上设置一个监听器以接受任何连接。一旦有客户端连接，我们将为他们设置一个临时表，以便我们可以存储他们的投资组合：

```js
recvServer.on('connection', (socket) => {
    const temp = new Map();
    clientTable.set(socket, temp);
});
```

1.  现在，为客户端设置一个数据监听器。当我们收到数据时，我们将根据以下格式解析消息，`SELL/BUY <Ticker> <Number>`：

```js
// inside of the connection callback for recvServer
socket.on('data', (msg) => {
    const input = msg.toString('utf8').split(' ');
    const buyOrSell = input[0];
    const tickerSymbol = input[1];
    const num = parseInt(input[2]);
});
```

1.  根据这个解析，我们检查客户端是否能执行这个操作。如果可以，我们将更改他们的投资组合，并发送一条消息告诉他们更改成功了：

```js
// inside the socket 'data' handler
const numHeld = symbolTable.get(input[1]);
if( buyOrSell === "BUY" && (num <= 0 || numHeld - num <= 0) ) {
    socket.write("ERROR!");
    return;
} 
const clientBook = clientTable.get(socket);
const clientAmount = clientBook.get(tickerSymbol);
if( buyOrSell === "SELL" && clientAmount - num < 0 ) {
    socket.write("ERROR!");
    return;
}
if( buyOrSell === "BUY" ) {
    clientBook.set(tickerSymbol, clientAmount + num);
    symbolTable.set(tickerSymbol, numHeld - num);
} else if( buyOrSell === "SELL" ) {
    clientBook.set(tickerSymbol, clientAmount - num);
    symbolTable.set(tickerSymbol, numHeld + num);
}
socket.write(`successfully processed request. You now hold ${clientBook.get(tickerSymbol)}` of ${tickerSymbol}`);
```

1.  一旦我们告诉客户端我们已处理他们的请求，我们可以通过 UDP 服务器向所有客户端写入：

```js
// after the socket.write from above
const msg = Buffer.from(`${tickerSymbol} ${symbolTable.get(tickerSymbol)}`);
server.send(msg, 0, msg.byteLength, 3000, multicastAddress);
```

1.  最后，我们需要通过标准输入处理来自服务器的新股票代码。一旦我们处理了请求，我们就通过 UDP 服务器发送数据，以便每个客户端都知道新股票的情况。

```js
process.stdin.setEncoding('utf8');
process.stdin.on('data', (data) => {
    const input = data.split(' ');
    const num = parseInt(input[1]);
    symbolTable.set(input[0], num);
    for(const client of clientTable) {
        client[1].set(input[0], 0);
    }

    server.send(Buffer.from(data), 0, data.length, 3000, multicastAddress);
});
```

为了清晰起见，几乎所有的错误逻辑都已被移除，但你可以在本书的 GitHub 存储库中找到它们。正如前面的例子所示，利用所有接口向其他点发送数据非常简单，无论是我们应用程序的其他部分还是监听数据的远程客户端。它们几乎都使用相同的接口，只在细微的实现细节上有所不同。只需记住，如果需要保证交付，应使用 TCP；否则，UDP 也是一个不错的选择。

接下来，我们将看一下 HTTP/2 标准以及与`net`、`dgram`和`http`/`https`模块相比，Node.js 中的服务器系统有些不同。

# HTTP/2

虽然它是在 2015 年引入的，但技术的采用速度很慢。HTTP/2 建立在 HTTP/1.1 协议的基础上，允许各种功能，这些功能在以前的系统中引起了问题。这使我们能够使用单个 TCP 连接接收不同的请求。这在 HTTP/1.1 中是不可能的，它引起了一个叫做头部阻塞的问题。这意味着我们实际上只能处理那么多的 TCP 连接，如果我们有一个长时间运行的 TCP 连接，它可能会阻塞之后的所有请求。

HTTP/2 还赋予了我们推送服务器端资源的能力。这意味着如果服务器知道浏览器将需要一个资源，比如一个 CSS 文件，它可以在需要之前将其推送到服务器。最后，HTTP/2 赋予了我们内置的流式传输能力。这意味着我们能够使用连接并将数据作为流发送，而不需要一次性发送所有数据。

HTTP/2 还给我们带来了其他好处，但这些是主要的好处。虽然`http`和`https`模块可能还会在未来一段时间内使用，但 Node.js 中的`http2`模块应该用于任何新的应用程序。

Node.js 中的`http2`模块与`http`和`https`模块有一些不同之处。虽然它不遵循许多其他 IPC/网络模块给我们的标准，但它确实为我们提供了一些很好的方法来通过 HTTP/2 发送数据。其中一个允许我们直接从文件系统流式传输文件，而不需要为文件创建管道并将其发送给发送方。以下代码中可以看到其中一些差异：

```js
import http2 from 'http2';
import fs from 'fs';
const server = http2.createSecureServer({
    key : fs.readFileSync('server.key.pem'),
    cert : fs.readFileSync('server.crt.pem')
});
server.on('error', (err) => console.error(err));
server.on('stream', (stream, headers) => {
    stream.respond({
        'content-type': 'text/plain',
        ':status' : 200
    });
    stream.end('Hello from Http2 server');
});
server.listen(8081, '127.0.0.1');
```

首先，注意服务器需要一个私钥和一个公共证书。这些用于确保建立的连接是安全的，这意味着没有人可以看到正在发送的内容。为了能够做到这一点，我们需要一个工具，比如`openssl`来创建这些密钥和证书。在 Windows 10 和其他 Unix 操作系统中，我们可以免费获得这个工具。否则，我们需要下载 Cygwin（[`www.cygwin.com/`](http://www.cygwin.com/)）。使用`openssl`，我们可以运行以下命令：

```js
> openssl req -x509 -newkey rsa:4096 -keyout server.key.pem -out server.crt.pem -days 365
```

这个命令生成了服务器和客户端进行安全通信所需的私钥和公共证书。我们不会在这里详细介绍它是如何实现的，但关于如何使用 SSL/TLS 实现这一点的信息可以在这里找到：[`www.cloudflare.com/learning/ssl/transport-layer-security-tls/`](https://www.cloudflare.com/learning/ssl/transport-layer-security-tls/)。

生成了我们的证书和密钥后，我们可以读取它们，以便我们的服务器可以开始运行。我们还会注意到，与响应消息事件或请求事件不同，我们响应流事件。HTTP/2 使用流而不是尝试一次性发送所有数据。虽然 Node.js 为我们封装了流的请求和响应，但这并不是操作系统层面可能处理的方式。HTTP/2 立即使用流。这就是为什么事件被称为流的原因。

接下来，我们不是调用`writeHead`方法，而是响应流。当我们想要发送信息时，我们利用`respond`方法并以这种方式发送头部。我们还会注意到一些头部是以冒号为前缀的。这是`http2`模块特有的，如果在发送特定头部时发现问题，加上冒号可能会解决问题。

除了我们在这里讨论的内容之外，这应该看起来与我们在 Node.js 中编写的普通 HTTP(s)服务器非常相似。然而，`http2`模块还有一些其他好处，其中之一是响应文件而不是必须读取文件并以这种方式发送。这可以在以下代码中看到：

```js
import http2 from 'http2';
import fs from 'fs';
import path from 'path';

const basePath = process.env.npm_package_config_static; //1.
const supportedTypes = new Set(['.ico', '.html', '.css', '.js']);
const server = http2.createSecureServer({
    key : fs.readFileSync(process.env.npm_package_config_key),
    cert : fs.readFileSync(process.env.npm_package_config_cert),
    allowHTTP1 : true //2.
});
server.on('error', (err) => console.error(err));
server.on('stream', (stream, header) => {
    const fileLoc = header[':path'];
    const extension = path.extname(fileLoc); //3.
    if(!supportedTypes.has(extension)) {
        stream.respond({
            ':status' : 400,
            'content-type' : 'application/json'
        });
        stream.end(JSON.stringify({
            error : 'unsupported data type!',
            extension
        }));
        return;
    }
    stream.respondWithFile( //4.
        path.join(process.cwd(), basePath, fileLoc),
        {
            ':status' : 200,
            'content-type' :
                extension === ".html" ?
                'text/html' :
                extension === ".css" ?
                'text/css' :
                'text/javascript'
        },
        {
            onError : (err) => { //5.
                if( err.code === 'ENOENT') {
                    stream.respond({ ':status' : 404 });
                } else {
                    stream.respond({ ':status' : 500 });
                }
                stream.end();
            }
        }
    )
});
server.listen(80, '127.0.0.1');
```

程序编号是关键的兴趣点，它们的工作方式如下：

1.  我们正在从`package.json`文件中读取信息，就像我们在上一章中所做的那样。我们还通过`npm run <script>`命令运行这个。查看上一章，了解如何做到这一点，以及我们如何在程序中使用`package.json`文件中的配置数据。

1.  我们为服务器设置了特定的配置选项。如果连接到我们的客户端无法使用 HTTP/2，那么我们将自动将一切转换回协商的协议，例如 HTTP/1.1。

1.  我们从 URL 中获取扩展名。这样，我们可以看到我们是否支持该文件类型，并发送适当的文件；否则，我们将返回一个 400 错误消息，并声明这是一个错误的请求。

1.  这种方法允许我们传入一个路径。然后，核心系统将帮助我们发送文件。我们所需要做的就是确保正确设置内容类型，以便浏览器可以解释数据。

1.  如果在任何时候出现错误，比如文件不存在，我们将以正确的状态做出响应，比如 404 或 500 错误。

虽然我们在这里呈现的只是`http2`模块的一小部分，但这展示了`http2`模块的不同之处，以及我们如何可以快速设置一个。如果需要，可以参考[`Node.js.org/dist/latest-v12.x/docs/api/http2.html`](https://nodejs.org/dist/latest-v12.x/docs/api/http2.html)来了解`http2`模块与`http`的不同之处以及它带来的所有功能。现在，我们将看一下网络的未来状态，并了解 Node.js 中的 HTTP/3。

# 快速浏览 HTTP/3

虽然我们所讨论的是进程、线程和其他计算机之间通信的现状，但有一种新的信息传递方式。新标准称为 HTTP/3，与前两个版本有很大不同。

# QUIC 协议

**Quick UDP Internet Connections** (**QUIC**)是由 Google 于 2012 年推出的。它是一种类似于 TCP、**传输层安全**（**TLS**）和 HTTP/2 协议的协议，但它全部通过 UDP 传输。这意味着 TCP 中内置的许多开销已经被移除，并用一种新的发送数据的方法替代。除此之外，由于 TLS 内置到协议中，这意味着在已定义的协议中添加安全性的开销已经被移除。

QUIC 目前被 Google 用于诸如 YouTube 之类的事物。虽然 QUIC 从未获得大规模的吸引力，但它帮助产生了将创建 HTTP/3 标准委员会的团体，并帮助指导委员会利用 UDP 作为协议的基础层。它还展示了安全性可以内置到协议中，并已经使 HTTP/3 具备了这一特性。

其他公司已经开始实施 QUIC 协议，而 HTTP/3 正在开发中。这个名单中一个显著的包括 Cloudflare。他们关于实施 QUIC 的博客文章可以在这里找到：[`blog.cloudflare.com/the-road-to-quic/`](https://blog.cloudflare.com/the-road-to-quic/)。

虽然 HTTP/3 尚未添加到 Node.js 中，但有一些包实现了 QUIC 协议。

# 对 node-quic 的一瞥

虽然 QUIC 目前不是最容易使用的，而且唯一的官方实现是在 Chromium 源代码中编写的，但已经有其他实现允许我们玩弄这个协议。`node-quic`模块已经被弃用，而 QUIC 实现正在尝试直接构建到 Node.js 中，但我们仍然可以使用它来看看我们将来如何利用 QUIC 甚至 HTTP/3。

首先，我们需要通过运行`npm install node-quic`命令来安装模块。有了这个，我们就能够编写一个简单的客户端-服务器应用程序。客户端应该看起来像下面这样：

```js
import quic from 'node-quic'

const port = 3000;
const address = '127.0.0.1';
process.stdin.setEncoding('utf8');
process.stdin.on('data', (data) => {
    quic.send(port, address, data.trim())
        .onData((data) => {
            console.log('we received the following back: ', data);
        });
});
```

我们会注意到，发送数据类似于我们在 UDP 系统中所做的方式；也就是说，我们可以发送数据而不需要绑定到端口和地址。除此之外，该系统运行方式类似于使用`http`或`http2`模块编写的其他应用程序。这里值得注意的一件事是，当我们从`quic`流中接收数据时，数据会自动转换为字符串。

上一个客户端的服务器将如下所示：

```js
import quic from 'node-quic'

const port = 3000;
const address = '127.0.0.1';
quic.listen(port, address)
    .then(() => {})
    .onError((err) => console.error(err))
    .onData((data, stream, buffer) => {
        console.log('we received data:', data);
        if( data === 'quit' ) {
            console.log('we are going to stop listening for data');
            quic.stopListening();
        } else {
            stream.write("Thank you for the data!");
        }
    });
```

再次，这应该看起来与我们编写的其他应用程序类似。这里的一个主要区别是，这个模块是针对 promise 编写的。除此之外，我们接收的数据是一个字符串，所以如果我们接收到`quit`，我们通过运行`stopListening`方法关闭自己。否则，我们将要发送的数据写入流中，类似于我们在 HTTP/2 协议中所做的。

为了了解 HTTP/3 的实现状态，建议您查看以下链接并定期检查：[`quicwg.org/`](https://quicwg.org/)。

正如我们所看到的，使用这个模块来利用 QUIC 协议是相当简单的。这对内部应用程序也可能很有用。只要注意，QUC 协议和 HTTP/3 标准都还没有完全完成，可能还需要几年的时间。这并不意味着你不应该利用它们，只是意味着在标准不稳定的时候事情可能会发生很快。

# 摘要

在不同系统之间发送数据，无论是线程、进程，甚至其他计算机，这是我们作为开发人员所做的。我们可以使用许多工具来做到这一点，我们已经看过大部分。只要记住，虽然一个选项可能使应用程序变得简单，但这并不总是意味着它是最好的选择。当我们需要拆分系统时，通常希望将特定的工作分配给一个单元，并使用某种形式的 IPC，比如命名管道，进行通信。如果我们需要将该任务移动到另一台计算机，我们总是可以切换到 TCP。

有了这些 IPC 和 Web 协议的基础，我们将能够轻松解决 Node.js 中的大多数问题，并在涉及 Web 应用程序时编写客户端和服务器端代码。然而，Node.js 并不仅仅是为 Web 应用程序而构建的。我们几乎可以做任何其他语言可以做的事情，甚至拥有大多数其他语言拥有的工具。本章应该有助于澄清这一点，并帮助巩固 Node.js 如何构建到已经开发的应用程序生态系统中。

考虑到所有这些，我们将研究流和如何在 Node.js 中实现我们自己的流。
