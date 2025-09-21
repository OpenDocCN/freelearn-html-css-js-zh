# 第十一章：网络和服务工作者

假设你正在构建一个酷炫的 Web 应用，比如说，将一个数分解为两个质数。现在，这涉及到大量的 CPU 密集型工作，这将阻塞主 UI 线程。主 UI 线程是最终用户直接观察和感知的交通车道。如果它看起来拥堵（卡顿）或阻塞，即使只有几秒钟，也会破坏用户体验。

这就是 web 工作者发挥作用的地方。可以将 web 工作者想象为道路上的那些侧车道，你可以将重型和缓慢（CPU 密集型）的卡车转移到那里，这样你就不至于阻塞主路上的用户闪亮的兰博基尼（主 UI 线程）。

另一方面，服务工作者也非常酷。服务工作者是你自己的可编程网络代理，它位于用户互联网连接和你的网站之间。在*与服务工作者一起工作*部分将有更多关于这个话题的介绍。

在本章中，我们将涵盖：

+   线程的介绍

+   网络工作者的介绍

+   专用工作者的介绍

+   设置专用工作者

+   共享工作者的介绍

+   设置共享工作者

+   设置内联 web 工作者

+   与主线程的通信

+   服务工作者的介绍

+   设置服务工作者

# 线程概念的介绍

简而言之，线程是一个简单且独立的运行代码片段。它是你的任务被执行的容器。在 web 工作者之前，JavaScript 只提供了一个线程；即开发者的主线程，用于做所有事情。

这在技术进步方面造成了一些问题。假设你正在运行一个平滑的 CSS3 动画，突然你需要因为某种原因在 JavaScript 端进行一些重量级的计算。如果你在主线程上这样做，这会使动画变得迟缓。然而，如果你将其卸载到在其自己的线程中运行的 web worker，它将不会对用户体验产生影响。

由于 web workers 在其自己的线程中运行，它们无法访问以下内容：

+   `DOM`：从 web 工作者和主 UI 脚本访问它不是线程安全的

+   `parent`对象：基本上，这提供了访问一些 DOM API 的权限，正如上述原因，访问这些 API 也是线程不安全的

+   `window`对象：浏览器对象模型（BOM）；访问此对象也是线程不安全的

+   `document`对象：`DOM`对象；因此，线程不安全

Web 工作者无法访问上述所有内容，因为给予工作者访问它们不是线程安全的。让我们更深入地了解我所说的意思。

# 什么是线程安全？

当两个或更多线程访问一个公共数据源时，必须非常小心，因为数据损坏和线程安全条件（如死锁、先决条件、竞争条件等）的可能性很高。

JavaScript 从一开始就没有添加线程支持。随着网页工作者在 JavaScript 中引入了一种**类似**的线程环境，这将有助于理解与线程相关的一些条件。

# 什么是死锁？

死锁是一种情况，其中两个线程因为各种原因而相互等待，两个线程的原因相互依赖。以下图将解释什么是死锁：

![](img/2baf6d73-527e-4759-b8b3-762242b2d7c9.png)

显然，两个线程（持有枪支的人）都需要对方的资源才能继续进行，所以没有人能继续进行。

# 什么是竞争条件？

如果允许网页工作者访问 DOM，可能会发生**竞争条件**问题。竞争条件是指两个线程竞争或竞争读取/修改单个数据源的情况。这是危险的，因为当两个线程同时尝试修改数据时，无法确定哪个线程会先修改数据。考虑以下示例。

假设有两个线程正在内存中处理相同的变量：

```js
// thread 1 - pseudo program code
if variable == 5:
    asyncOperationWhichTakes200MS()
    // just here thread 2 modifies variable to 10
    res = variable * 5
    // res is now 50 instead of 25
    // unpredicatable behavior ahead
```

通过使用**信号量**可以避免竞争条件，这实际上就是锁定共享数据资源，直到一个线程完成并释放它。

**有趣的事实：**如果你在 Ubuntu 或任何支持`apt-get`作为包管理器的 Linux 发行版上使用`sudo apt-get update`，然后在另一个终端中尝试运行另一个`apt-get update`命令，你会得到这个错误：

`E: 无法锁定管理目录 (/var/lib/dpkg/)，另一个进程正在使用它？`

Linux 锁定目录是为了避免两个命令相互覆盖结果的潜在竞争条件。

大多数语言只有一个线程与 UI 交互并更新 UI，其他线程只能向主线程发送消息以更新 UI。

# 网页工作者的简介

网页工作者本质上是一段不在你的主应用程序线程中运行的 JS 代码。而且，我说的线程字面意思是不同的线程。网页工作者真正使 JavaScript 能够在多线程模式下工作。这里可能会出现的一个问题是，异步操作和网页工作者之间有什么区别？

如果你仔细想想，它们基本上是同一件事。网页工作者会暂时从主线程中卸载一些负载，然后带着结果回来。然而，要理解的是，`async`函数在 UI 线程上运行，而网页工作者则不是。此外，网页工作者是长期存在的，它们存在于一个单独的线程中，而异步操作符，如我们在*第四章，异步编程*中讨论的，遵循*事件循环*。

在性能方面，网页工作者也比传统的异步操作快得多。这里有一个测试，它将随机生成的长度为**10K**和**1M**的数组作为异步操作和网页工作者进行排序：

![](img/0fd49ce5-f3e9-4e78-9db0-669da6798119.png)

注意，**2,493 ops/sec** 意味着 JS 能够在 1 秒内对**2,493**个长度为**10K**的数组进行排序！另一方面，异步 JS 在 1 秒内能够对大约 67 个长度为**10K**的数组进行排序，这仍然非常快，但比其竞争对手慢得多。

# 检查工作支持是否可用

尽管 Web 工作器已经存在很长时间，并且支持非常强大，但你仍然可能想要检查客户端浏览器中是否支持 Web 工作器（例如，Opera Mini 不支持它）。如果不支持，那么只需在主脚本中也加载 Web 工作器文件，让你的用户感受到这种热感。

Web 工作器作为 `window` 对象可用，因此你只需检查这一点就可以开始：

```js
if(typeof window.Worker !== "function") {
    // worker not available
} else {
    // good to go
}
```

# 使用专用 Web 工作器

**专用工作器**是专门为单个主脚本服务的工作器。这意味着工作器不能与任何其他脚本交互，除了页面上或任何其他域的主脚本。

让我们通过设置一个专用工作器来尝试理解专用工作器。

# 设置专用工作器

使用构造函数中的文件名调用 `new Worker()` 是创建专用工作器所需做的全部事情：

```js
// script.js loaded on index.html

const awesomeWorker = new Worker('myworker.js');
```

使用 `new Worker` 构造函数，我们创建了一个 `Worker` 实例。这将使浏览器下载 `myworker.js` 文件并为它启动一个新的操作系统线程。

我们可以在 `myworker.js` 文件中放置以下内容：

```js
// myworker.js
console.log('Hello world!');
```

这将在控制台内打印 `Hello world`。

工作器可以自己创建子工作器，以下所有内容也适用于该子工作器。

# 使用专用工作器

专用工作器可以与其启动脚本通信，监听某些事件，这些事件在任一脚本发送/接收消息时触发。

这些事件可以通过某些事件处理器在两个脚本（工作器和主脚本）中处理。让我们学习如何实现这一点。

# 在主脚本上监听消息

我们可以通过 `onmessage` 事件监听一个工作器发送到主脚本的任何内容。这将是它的样子：

```js
// script.js

const awesomeworker = new Worker('myworker.js');
awesomeworker.addEventListener('message', e => {
    console.log(e.data); // data sent by worker
});
```

在这里，我们的脚本正在监听工作器发送的消息。每次工作器发送消息（我们将在下一节“从主脚本发送消息”中看到如何做），都会触发前一个事件，并且我们在控制台中打印数据。

# 在工作脚本上监听消息

工作者本身可以访问 `self` 对象，你可以附加类似的事件监听器，如之前讨论的那样。让我们看看结果：

```js
// myworker.js

self.addEventListener('message', e => {
    console.log(e.data); // data sent by main script
});
```

在这里，每当主脚本向这个特定的 Web 工作器发送消息时，消息事件监听器就会被触发。我们简单地使用 `console.log(e.data)` 打印主脚本发送的内容。

如果你愿意，可以省略 `self` 关键字。默认情况下，在工作者中，事件将附加到 `self`。

# 从主脚本发送消息

好的！一旦你正确设置了监听事件，你将想要向工作器发送一些任务以便它执行。这是如何实现这一点的：

```js
// script.js

const awesomeworker = new Worker('myworker.js');
awesomeworker.addEventListener('message', e => {
    console.log(e.data); // data sent by worker
});

const data = {task: "add", nums: [5, 10, 15, 20]};

// lets send this data
awesomeworker.postMessage(data);
```

好的。在这里，我们给工作器分配了一个加两个数字的任务。请注意，我们能够通过 `postMessage` 方法传递对象/数组，这实际上用于向生成的工作器发送/传递消息。

通过 `postMessage` 发送的对象是复制的而不是引用。这意味着，如果工作器脚本以任何方式修改此对象，它将不会反映在主 `script` 对象中。这对于消息传递的一致性很重要。

现在，我们可以在另一端（即工作器）接收此对象，并按以下方式处理它：

```js
// myworker.js

self.addEventListener('message', e => {
    if(e.data.task == "add") {
        const res = e.data.nums.reduce((sum, num) => sum+num, 0);
        // do something with res
    }
});
```

在这里，在接收到消息后，我们检查主脚本是否希望工作器添加数字。如果是这种情况，我们使用内置的 `reduce` 方法将传递的数组中的所有数字减少到一个单一值。

# 从工作器脚本发送消息

与主脚本类似，`postMessage` 在工作器脚本中也用于与主脚本通信。让我们看看如何将前面的结果发送到主脚本：

```js
// myworker.js

addEventListener('message', e => {
    if(e.data.task == "add") {
        const res = e.data.nums.reduce((sum, num) => sum+num, 0);
        postMessage({task: "add", result: res}); // self.postMessage will also work
    }
});
```

在这里，就像在先前的代码中一样，我们将数组值减少到 `sum`，然后实际上通过 `postMessage` 函数将我们执行的操作发送回主 UI 脚本。传递的对象可以通过调用其自身的监听方法来接收。

`script.js` 将如下所示：

```js
// script.js

const awesomeworker = new Worker('myworker.js');
awesomeworker.addEventListener('message', e => {
    if(e.data.task == "add") { 
    // task completed. do something with result
        document.write(e.data.result);
    }
});

const data = {task: "add", nums: [5, 10, 15, 20, 25, 30, 35, 40, 45, 50]};
awesomeworker.postMessage(data);
```

在这里，你可以看到我们将 `task` 以对象的形式发送到工作器，工作器很好地执行了计算并将结果发送回主脚本，这进一步由附加到 `awesomeworker` 的消息事件监听器处理，该监听器简单地将结果写入文档。

# 工作器中的错误处理

有可能你的工作器可能会因为主脚本发送的格式不正确的数据而抛出错误。在这种情况下，主脚本中的工作器 `onerror` 方法被调用：

```js
// script.js

const awesomeworker = new Worker('myworker.js');
awesomeworker.postMessage({task: "divide", num1: 5, num2: 0})

awesomeworker.addEventListener('error', e => {
    console.log(e); // information of ErrorEvent
});
```

在这里，我们附加了一个错误事件监听器，目前我们只是将其记录到控制台。你可能希望将其发送到服务器以在生产应用程序中进行进一步分析。

工作器如下所示：

```js
// myworker.js

self.addEventListener('message', e => {
    if(e.data.num2 == 0) {
        throw "Cannot divide by 0";
    } else {
        postMessage({task: "divide", result: e.data.num1/e.data.num2 });
    }
});
```

在前一个例子中，工作器抛出一个错误，这在主脚本中表现为一个 `ErrorEvent` 对象。从那里，你可以处理这个错误。

从 Web Worker 抛出的错误不会永久停止其工作。除非终止，否则它仍然可用。

# 终止工作器

当你认为工作器不再需要时，你可以终止工作器。你可以从工作器本身或父脚本中终止工作器。让我们在下一节中看看如何操作。

# 从工作器脚本终止

有时候，当工作器在进行某种可能持续不同时间段的异步任务时，可能需要在工作器内部终止工作器。为此，工作器内部提供了一个名为 `close()` 的方法：

```js
// myworker.js

addEventListener('message', e => {
    if(e.data.message == "doAjaxAndDie") {
        fetch(...).then(data => {
            postMessage(data);
            close(); // or self.close();
        });
    }
});
```

# 从主脚本终止

同样，如果你愿意，你也可以从主脚本中终止一个工作线程。终止后，你的工作线程实例将无法再用于发送消息。它还会终止你工作线程中正在执行的所有进程：

```js
// script.js

const awesomeworker = new Worker('myworker.js');

awesomeworker.addEventListener('message', e => {
    if(e.data.message == "killme") {
        awesomeworker.terminate(); // bye bye
        console.log("Worker terminated");
    }
});
```

为此，相应的 `myworker.js` 文件将是：

```js
// myworker.js
// .. some work
postMessage({message: "killme"});
```

在工作线程内部终止涉及调用 `close()`，而从父脚本中终止则涉及调用 `terminate()` 方法。

# 通过 `postMessage` 传输（而非复制）数据

实际上，你可以使用 `postMessage` 函数传输大量数据。这意味着什么，它与到目前为止我们使用 `postMessage` 所做的事情有什么不同？

好吧，`postMessage` 的实际语法是：`postMessage(aMessage, transferList)`。

这意味着，你传递给 `transferList` 的任何内容在发送它的那个工作线程中似乎都丢失了。实际上，你赋予了其他脚本拥有那些数据的权限。你将那些数据的所有权转让给了那个其他脚本。记住，这与通常发生的情况不同（即，你仍然可以访问发送到 Web 工作线程/主脚本的同一脚本中的对象），因为在这种情况下，数据并没有被复制。其所有权已被转让。

这使得在 Web 工作线程之间传输大量数据变得非常快。可传输的对象包括 `ArrayBuffer` 等内容。以下是如何处理它的一个示例：

```js
const ab = new ArrayBuffer(100);

// add data to this arraybuffer
console.log(ab.byteLength); // 100
worker.postMessage(ab, [ab]);
console.log(ab.byteLength); // 0 - ownership lost
```

你可以看到，我们的 `ArrayBuffer` 的大小从 100 变为 0。这是因为你不再可以访问 `ArrayBuffer` 内存，因为你已经将其传输给了其他脚本。

# 与共享工作线程一起工作

如前所述，共享工作线程是多个脚本可以访问的工作线程，前提是它们遵循相同的源策略（更多内容将在名为 *Same origin policy* 的后续部分中介绍）。

与专用工作线程相比，API 略有不同，因为这些工作线程可以被任何脚本访问，因此需要通过 `SharedWorker` 对象中嵌入的不同端口来管理所有连接。

# 设置共享工作线程

可以通过调用 `SharedWorker` 构造函数并传入文件名作为参数来创建一个共享工作线程：

```js
const awesomeworker = new SharedWorker('myworker.js');
```

在这里，我们使用了 `SharedWorker` 构造函数来创建一个 `sharedworker` 对象的实例。与专用工作线程不同，你将无法在浏览器中看到对 `myworker.js` 文件进行的 HTTP 网络请求。这很重要，因为浏览器只需要维护多个脚本调用此 Web 工作线程时该文件的一个实例：

```js
// myworker.js
console.log('Hello world!');
```

与专用工作线程不同，共享工作线程不会在主网站的控制台中输出 `Hello World!`。这是因为共享工作线程不会只加载到那个页面。共享工作线程为每个访问它的文件加载一次。因此，它有自己的控制台。

在 Google Chrome 中，要调试共享工作线程，请在打开负责启动共享工作线程的页面后，打开 `chrome://inspect/#workers`。在那里，你可以选择调试它：（“检查”链接）

![](img/57bd90a5-58ae-4621-b2c6-872208f41713.png)

完成这些后，让我们继续设置共享工作者监听器的指南。

# 与共享工作者一起工作

共享工作者可以与其启动脚本通信，监听某些事件，这些事件在任一脚本发送/接收消息时触发。然而，与专用工作者不同，在这里我们必须在每个连接上显式注册 `onmessage` 事件。

# 在主脚本上监听消息

在这里，与专用工作者不同，我们必须在共享工作者对象上可用的端口属性上添加 `onmessage` 事件：

```js
// script.js

const awesomeworker = new SharedWorker('myworker.js');
awesomeworker.port.start(); // important
awesomeworker.port.addEventListener('message', e => { // notice the .port
    console.log('Shared worker says .. ', e.data);
});
```

当我们的 `SharedWorker` 对这个特定的脚本做出回应时，会触发此事件。

注意到 `awesomeworker.port.start();` 这一行，它指示共享工作者与这个脚本进行交互。当使用 `addEventListener` 时，从两个文件（工作者和脚本）中都必须使用 `port.start()` 行来启动双向通信。

# 在工作者脚本上监听消息

类似地，`self` 在这里被定义；然而，`window` 并没有被定义。所以，你可以使用 `self.addEventListener` 或 `addEventListener`（或者直接 `onconnect = function()`）：

```js
// myworker.js
addEventListener('connect', e => {
    console.log(e.ports);
    const port = e.ports[0];
    port.start();
    port.addEventListener('message', event => {
        console.log('Some calling script says.. ', event.data);
    });
});
```

在这里，事件包含了关于我们的脚本连接到的端口的详细信息。我们获取连接端口并与之建立连接。

与我们的主脚本类似，我们必须在这里指定 `port.start()` 以确保两个文件之间通信成功。

# 从父脚本发送消息

注意，任何同源的脚本（基本上，同源意味着你从属于同一域的多个 URL 访问它；例如，[`www.google.co.in/`](https://www.google.co.in/) 访问 `google.com/script.js` 符合同源，但 [facebook.com](https://www.facebook.com/) 访问 `google.com/script.js` 则不符合。关于同源的内容将在稍后的名为 *同源策略* 的部分中详细介绍）可以以与我们之前 `script.js` 访问它相同的方式访问共享工作者，工作者对于每个访问它的文件都处于相同的状态。

这就是从脚本向工作者发送消息的方式：

```js
// script.js

const awesomeworker = new SharedWorker('myworker.js');
awesomeworker.port.start();
awesomeworker.port.postMessage("Hello from the other side!");
```

这与专用工作者类似，但我们必须在这里明确提到端口对象。

`myworker.js` 文件看起来像：

```js
// myworker.js
addEventListener('connect', e => {
    console.log(e.ports);
    const port = e.ports[0];
    port.start();
    port.addEventListener('message', event => {
        console.log('Some calling script says.. ', event.data); // logs
        "Hello from the other side!"
    });
});
```

如前所述，如果我们使用 `addEventListener` 添加回调，则需要执行 `port.start()` 来建立工作者和主脚本之间的通信。然后我们为这个特定的端口分配一个 `onmessage` 事件监听器。

最后，我们只需将调用脚本要告诉工作者的内容记录到控制台。

# 从工作者脚本发送消息

如果您已经认识到我们在专用工作者和共享工作者中调用方法的方式之间的区别，做得好！我们不是在`self`上调用方法，而是在端口对象上调用所有专用 Web 工作者方法，这是工作者区分许多（可能）与之通信的脚本的方式：

```js
// myworker.js
addEventListener('connect', e => {
    console.log(e.ports);
    const port = e.ports[0];
    port.start();
    port.addEventListener('message', event => {
        console.log('Some calling script says.. ', event.data); 
        // some work
        port.postMessage("Hello ;)");
    });
});
```

它与上面的代码完全一样，但这次我们的共享工作者回复了发送消息的人，并向它说“你好”。

如果您有两个实例的 HTML 页面正在运行，加载`script.js`（即新的`SharedWorker`），它们都与共享工作者有独立的端口连接。

# 错误处理

在这里，错误处理有点棘手。由于错误可能由任何端口（任何父文件）在任何脚本中发生，您必须手动将错误发送到每个端口。但为此，您还必须存储端口（当它们连接时）。以下是它应该看起来像什么：

```js
// myworker.js

const ports = [];

addEventListener('connect', e => {
    const port = e.ports[0];
    ports.push(port); // assemble all connections
    port.start();
    // .. other info
});

addEventListener('error', e => {
    console.log(e); // Info about error
    ports.forEach(port => port.postMessage({type: 'error', res: e}));
});
```

在这里，您可以看到我们正在手动将错误信息发送到每个父文件。之后，您可以在父文件本身中处理错误。

作为旁注，在您的共享工作者内部有一个数组访问所有连接是一个好习惯。在某些情况下可能会有所帮助，例如，当您想要不同的页面相互通信时！

# 断开共享工作者连接

您可以从共享工作者那里断开父进程的连接，或者完全关闭共享工作者。然而，后者只能由工作者的 JS 完成。以下章节将讨论您如何断开单个父进程与工作者的连接。

# 断开单个父-工作者连接

当调用此代码时，父进程与工作者之间的连接被关闭，您将无法再使用该工作者对象来发送消息：

```js
// script.js

const awesomeworker = new SharedWorker('myworker.js');
awesomeworker.port.start(); 

// some processing and some work

awesomeworker.port.close();
awesomeworker.port.postMessage("Are you still alive?"); // does not work | no effect
```

尽管工作者仍然存在，但它失去了调用`.port.close()`的脚本的连接。

连接关闭后，工作者将无法从主脚本发送/接收消息。然而，主脚本始终可以通过使用`new SharedWorker`构造函数创建一个新的实例来再次调用共享 Web 工作者。

# 完全断开共享工作者

您可以通过在其 JS 内部调用`self.close()`来永久终止共享工作者。您也可以从父脚本发送消息来杀死工作者：

```js
// script.js

const awesomeworker = new SharedWorker('myworker.js');
awesomeworker.port.start();

awesomeworker.port.postMessage({type: 'cmd', action: 'die'});
```

我们只是从我们的主脚本向我们的共享工作者发送了一条消息，并传递了共享工作者应该被永久终止的消息。

工作者文件看起来像：

```js
// myworker.js

addEventListener('connect', e => {
    const port = e.ports[0];
    port.start();
    port.addEventListener('message', event => {
        if(event.data.type == 'cmd' && event.data.action == 'die') {
            self.close(); // terminates worker
        }
    });
});
```

在验证主脚本确实想要终止所有实例的工作者之后，工作者在其自身上调用`close`方法，从而终止它。

# 内联 Web 工作者的介绍

从单个文件创建 Web Worker 而不实际为你的 Web Worker 创建一个单独的 JS 文件是可能的。然而，我仍然建议你为你的 Web Worker 创建一个不同的文件，为了代码的清晰性和模块化。在编程中，模块化总是首选。

我们可以利用`blob` URL 来实际上将内存中的数据指向一个 URL，然后加载`blob` URL 而不是实际的文件 URL。由于这个 URL 仅在用户的计算机上动态生成，因此你不需要为特定的 Web Worker 创建一个单独的文件。以下是我们将如何做到这一点：

```js
const blob = new Blob(['(',
function() {
    // web worker code here
}.toString(),
')()'], { type: 'application/javascript' }));

const url = URL.createObjectURL(blob); // gives a url of kind blob:http://....
const awesomeworker = new Worker(url);
```

有时快速启动一个小型 Web Worker 很容易。然而，这种方法不适用于共享 Web Worker。你需要为它们创建一个单独的文件。这是因为`SharedWorker`依赖于所有实例都从一个单独的文件加载的事实。然而，创建 blob 数据的 URL 每次都会创建不同的 URL。所以两个页面，即使它们有相同的 JS 代码，也会有不同的 URL，因此会有不同的`SharedWorker`实例。

# 同源策略

之前我说了几次，共享工作者将仅对与它们共享相同源的父母文件可用。这意味着什么？

让我们考虑 URL [`www.packtpub.com/all.`](http://www.packtpub.com/all)

下面是一个表格，展示了哪些 URL 将与该域名具有相同的源，哪些不会：

| **新 URL** | **同源** | **原因** |
| --- | --- | --- |
| [`www.packtpub.com/support`](http://www.packtpub.com/support) | 是 | - |
| `http://www.packtpub.com/account/abc/xyz` | 是 | - |
| [`www.packtpub.com/all`](https://www.packtpub.com/all) | 否 | 不同的协议 |
| [`username:password@www.packtpub.com/all`](http://username:password@www.packtpub.com/all) | 是 | - |
| `http://www.packtpub.com:8000/somepage` | 否 | 不同的端口号 |
| `http://packtpub.com/somepage` | 否 | 不同的主机 |
| `http://dev.packtpub.com/somepage` | 否 | 不同的主机 |

到目前为止，我相信你能够猜出什么使得某些内容具有相同的源，什么不具有。是的，你是对的！相同的主机、端口和协议使得两个 URL 处于相同的源。对于列表中的 URL，其答案为“是”的，只有那些 URL 才能访问由[`www.packtpub.com/all`](http://www.packtpub.com/all)启动的共享工作者。

# 与服务工作者一起工作

服务工作者！它们最终通过在 JavaScript 中创建网络代理，为开发者提供了对网络层的精确控制。使用服务工作者，你可以拦截和修改网络资源请求，处理缓存方式，并在用户网络断开时做出适当的响应。

让我们一步一步地展示如何设置服务工作者及其相关方法。

# 服务工作者的先决条件

服务工作者的先决条件是：

+   由于 service worker 功能强大（几乎像网络代理），为了避免某些攻击，它们仅适用于运行在 HTTPS 上的域名。然而，它们在`localhost`上也能正常运行。

+   它们大量依赖于我们已经在*第四章，异步编程*中深入讨论过的 promises。

# 检查浏览器支持

检查客户端的浏览器是否支持 service workers 很容易：

```js
if('serviceWorker' in navigator) {
    // service worker available
    // lets code
}
```

这里，我将假设用户的浏览器中已经可用了一个 service worker，以避免每次都进行不必要的代码缩进。

# service worker 的生命周期

下图展示了 service worker 的生命周期：

![](img/bae86247-0f7d-4967-accd-515a09c38a9c.png)

从图中可以看出，首先需要安装 service worker。然后它触发某些事件，我们可以在代码中捕获这些事件来处理不同的事情。现在让我们详细讨论如何实现所有这些步骤。

# 注册 service worker

首先，你的主脚本必须将 service worker 注册到浏览器中。以下是操作方法：

```js
navigator.serviceWorker.register('/sw.js')
.then(reg => console.log(reg))
.catch(err => console.log(err));
```

`reg`对象与你的 service worker 注册信息相关联。

service worker 注册代码可以安全地多次运行。如果已经注册，浏览器将不会重新注册它。

你可以在这里调试 service workers：Chrome-->检查-->Service Workers。

注意，service worker 脚本的作用域是它所在的作用域。例如，前面的文件位于根目录（`localhost/sw.js`），因此它可以拦截所有`localhost/*`请求。如果它位于，比如说，`localhost/directory/sw.js`，那么它只能拦截`localhost/directory/*`请求。

# 安装 service workers

一旦你的 worker 注册成功，你的 service worker 文件内部会触发一个安装事件。在这里，我们将设置资源的缓存。接下来会有很多新术语出现；请耐心等待：

```js
// sw.js

self.addEventListener('install', e => {
    e.waitUntil(async function() {
        const cache = await caches.open('cacheArea');
        await cache.addAll(['/', '/styles/main.css', '/styles/about.css']);
    }());
});
```

好的！发生了什么？

1.  我们在我们的`sw.js`文件中添加了一个安装事件监听器，当我们的 service worker 被注册时，它会触发。

1.  `e.waitUntil`接受一个 promise（我们确实提供了一个 promise；记得`async function`返回一个 promise，我们也调用了那个函数）。

1.  然后在浏览器中有一个名为**CacheStorage**的东西。我们通过命名缓存并使用`addAll`方法添加我们想要缓存的资源来将资源添加到缓存中。

1.  我们传递一个包含所有想要添加到缓存存储的文件/路径的数组。

1.  安装事件已经结束。

在安装事件本身中设置缓存是完全可选的。我们可以在安装事件之后，稍后进行设置。例如，在获取新资源时，可以边走边设置缓存。

# 使用 service workers 进行获取

一切准备就绪后，你应该能够通过`fetch`事件以以下方式拦截请求：

```js
self.addEventListener('fetch', e => {
    e.respondWith(async function() {
        const response = await caches.match(e.request);
        if(response) {
            return response;
        }
        return fetch(e.request);
  }());
});
```

等一下！让我们看看这里发生了什么：

1.  这个监听器将在浏览器在其注册范围内进行 fetch 请求时被触发（我们之前讨论过这一点）。

1.  `respondWith`也接受一个承诺，这是我们给它提供的。

1.  然后，我们检查请求的文件是否已经存在于我们的缓存中（使用`catches.match(e.request)`）。如果是，我们直接返回缓存的文件。如果不是，我们使用 fetch API（我们之前章节讨论过）来获取响应，然后继续执行。

你也可以在控制台中打印出`e.request`并稍作修改以修改请求。这给了网站开发者对其自己网站巨大的控制权，因此不应由其他人处理。这也是为什么 service workers 仅在 HTTPS 协议上可用，以避免中间人攻击。

Service workers 是一项相对较新的技术，它们的标准正在经历许多工作。查看[`bit.ly/serviceworkers`](http://bit.ly/serviceworkers)获取任何更新。

# 摘要

因此，最终，我们有机会查看 web workers 和 service workers 以及 JavaScript 提供的多线程环境中的精彩部分！虽然 service workers 是渐进式 Web 应用的未来，但 web workers 将支持它们处理任何高负载任务。

正确地结合这些技术，似乎一切皆有可能！在下一章中，我们将探讨 JavaScript 首次引入的非常有趣的概念：共享内存和原子操作。
