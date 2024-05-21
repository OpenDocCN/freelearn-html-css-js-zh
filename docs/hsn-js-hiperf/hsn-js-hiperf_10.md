# 第十章：工作者-学习专用和共享工作者

在过去的几章中，我们专注于 Node.js 以及如何利用与前端相同的语言编写后端应用程序。我们已经看到了创建服务器、卸载任务和流式传输的各种方法。在这一部分，我们将专注于浏览器的任务卸载方面。

最终，正如我们在 Node.js 中所看到的，我们需要将一些计算密集型任务从主线程转移到单独的线程或进程，以确保我们的应用程序保持响应。服务器不响应的影响可能相当令人震惊，而用户界面不工作的影响对大多数用户来说是非常令人反感的。因此，我们有了 Worker API。

在本章中，我们将专门研究两种工作方式，即专用和共享。总的来说，我们将做以下工作：

+   学会通过 Worker API 将繁重的处理任务转移到工作线程。

+   学习如何通过`postMessage`和`BroadcastChannel` API 与工作线程进行通信。

+   讨论`ArrayBuffer`和`Transferrable`属性，以便我们可以快速在工作者和主线程之间移动数据。

+   查看`SharedWorker`和 Atomics API，看看我们如何在应用程序的多个选项卡之间共享数据。

+   查看利用前几节知识的共享缓存的部分实现。

# 技术要求

完成本章需要以下项目：

+   文本编辑器或 IDE，最好是 VS Code

+   访问 Chrome 或 Firefox

+   计算机并行化知识

+   在[`github.com/PacktPublishing/Hands-On-High-Performance-Web-Development-with-JavaScript/tree/master/Chapter10`](https://github.com/PacktPublishing/Hands-On-High-Performance-Web-Development-with-JavaScript/tree/master/Chapter10)找到的代码。

# 将工作转移到专用工作者

工作者使我们能够将长时间运行的计算密集型任务转移到后台。我们不必再担心我们的事件循环是否被某种繁重的任务填满，我们可以将该任务转移到后台线程。

在其他语言/环境中，这可能看起来像以下内容（这只是伪代码，实际上与任何语言都没有真正联系）：

```js
Thread::runAsync((data) -> {
   for(d : data) { //do some computation }
});
```

虽然这在这些环境中运行良好，但我们必须开始考虑诸如死锁、僵尸线程、写后读等主题。所有这些都可能非常难以理解，通常是可以遇到的最困难的错误之一。JavaScript 没有给我们提供利用类似前述的能力，而是给了我们工作者，这给了我们另一个上下文来工作，我们在那里不会遇到相同的问题。

对于那些感兴趣的人，操作系统或 Unix 编程的书籍可以帮助解决上述问题。这些主题超出了本书的范围，但它们非常有趣，甚至有一些语言正在尝试通过将解决方案构建到语言中来解决这些问题。其中一些例子是 Go（[`golang.org/`](https://golang.org/)），它使用消息传递技术，以及 Rust（[`www.rust-lang.org/`](https://www.rust-lang.org/)），它利用借用检查等概念来最小化这些问题。

首先，让我们以在后台进行工作的示例开始，我们将生成一个`Worker`并让它计算 100 万个数字的总和。为此：

1.  我们在 HTML 文件中添加以下`script`部分：

```js
<script type="text/javascript">
    const worker = new Worker('worker.js');
    console.log('this is on the main thread');
</script>
```

1.  我们为我们的`Worker`创建一个 JavaScript 文件，并添加以下内容：

```js
let num = 0;
for(let i = 0; i < 1000000; i++) {
    num += i;
}
```

如果我们启动 Chrome，我们应该看到打印出两条消息-一条说它在主线程上运行，另一条显示值为 499999500000。我们还应该看到其中一条是由 HTML 文件记录的，另一条是由工作者记录的。我们刚刚生成了一个工作者，并让它为我们做了一些工作！

请记住，如果我们想从我们的文件系统运行 JavaScript 文件而不是服务器，我们需要关闭所有 Chrome 的实例，然后从命令行重新启动它，使用`chrome.exe –-allow-file-access-from-files`。这将使我们能够从文件系统启动我们的外部 JavaScript 文件，而不需要服务器。

让我们继续做一些用户可能想做的更复杂的事情。一个有趣的数学问题是得到一个数字的质因数分解。这意味着，当给定一个数字时，我们将尝试找到组成该数字的所有质数（只能被 1 和它自己整除的数字）。一个例子是 12 的质因数分解，即 2、2 和 3。

这个问题导致了密码学的有趣领域以及公钥/私钥的工作原理。基本的理解是，给定两个相对较大的质数，将它们相乘很容易，但根据时间限制，从它们的乘积中找到这两个数字是不可行的。

回到手头的任务，我们将在用户将数字输入到输入框后生成一个`worker`。我们将计算该数字并将其记录到控制台。所以让我们开始：

1.  我们在 HTML 文件中添加一个输入，并更改代码以在输入框的更改事件上生成一个`worker`：

```js
<input id="in" type="number" />
<script type="text/javascript">
document.querySelector("#in").addEventListener('change', (ev) => {
    const worker = new Worker('worker.js', {name : 
     ev.target.value});
});
</script>
```

1.  接下来，我们将在`worker`中获取我们的名字，并将其用作输入。从那里，我们将运行在[`www.geeksforgeeks.org/print-all-prime-factors-of-a-given-number/`](https://www.geeksforgeeks.org/print-all-prime-factors-of-a-given-number/)找到的质因数分解算法，但转换为 JavaScript。完成后，我们将关闭`worker`：

```js
let numForPrimes = parseInt(self.name);
const primes = [];
console.log('we are looking for the prime factorization of: ', numForPrimes);
while( numForPrimes % 2 === 0 ) {
    primes.push(2);
    numForPrimes /= 2;
}
for(let i = 3; i <= Math.sqrt(numForPrimes); i+=2) {
    while( numForPrimes % i === 0 ) {
        primes.push(i);
        numForPrimes /= i;
    }
}
if( numForPrimes > 2 ) {
    primes.push(numForPrimes);
}
console.log('prime factorization is: ', primes.join(" "));
self.close();
```

如果我们现在在浏览器中运行这个应用程序，我们会看到在每次输入后，我们会在控制台中得到控制台日志消息。请注意，数字 1 没有因子。这是一个数学原因，但请注意数字 1 没有质因数分解。

我们可以对一堆输入运行这个，但如果我们输入一个相对较大的数字，比如`123,456,789`，它仍然会在后台计算，因为我们在主线程上做事情。现在，我们目前通过 worker 的名称向 worker 传递数据。必须有一种方法在 worker 和主线程之间传递数据。这就是`postMessage`和`BroadcastChannel`API 发挥作用的地方！

# 在我们的应用程序中移动数据

正如我们在 Node.js 的`worker_thread`模块中看到的，有一种方法可以与我们的 worker 通信。这是通过`postMessage`系统。如果我们看一下方法签名，我们会发现它需要一个消息，可以是任何 JavaScript 对象，甚至带有循环引用的对象。我们还看到另一个名为 transfer 的参数。我们稍后会深入讨论这一点，但正如其名称所示，它允许我们实际传输数据，而不是将数据复制到 worker。这是一个更快的数据传输机制，但在利用它时有一些注意事项，我们稍后会讨论。

让我们以我们一直在构建的例子为例，并回应从前端发送的消息：

1.  我们将在每次更改事件发生时创建一个新的`worker`并立即创建一个。然后，在更改事件上，我们将通过`postMessage`将数据发送到`worker`：*

```js
const dedicated_worker = new Worker('worker.js', {name : 'heavy lifter'});
document.querySelector("#in").addEventListener('change', (ev) => {
    dedicated_worker.postMessage(parseInt(ev.target.value));
});
```

1.  如果我们现在尝试这个例子，我们将不会从主线程收到任何东西。我们必须响应 worker 的全局描述符`self`上的`onmessage`事件。让我们继续添加我们的处理程序，并删除`self.close()`方法，因为我们想保留它：

```js
function calculatePrimes(val) {
    let numForPrimes = val;
    const primes = [];
    while( numForPrimes % 2 === 0 ) {
        primes.push(2);
        numForPrimes /= 2;
    }
    for(let i = 3; i <= Math.sqrt(numForPrimes); i+=2) {
        while( numForPrimes % i === 0 ) {
            primes.push(i);
            numForPrimes /= i;
        }
    }
    if( numForPrimes > 2 ) {
        primes.push(numForPrimes);
    }
    return primes;
}
self.onmessage = function(ev) {
    console.log('our primes are: ', calculatePrimes(ev.data).join(' '));
}
```

从这个例子中可以看出，我们已经将素数的计算移到了一个单独的函数中，当我们收到消息时，我们获取数据并将其传递给`calculatePrimes`方法。现在，我们正在使用消息系统。让我们继续为我们的示例添加另一个功能。不要打印到控制台，让用户根据他们的输入得到一些反馈：

1.  我们将在输入框下面添加一个段落标签来保存我们的答案：

```js
<p>The primes for the number is: <span id="answer"></span></p>
<script type="text/javascript">
    const answer = document.querySelector('#answer');
    // previous code here
</script>
```

1.  现在，我们将在`worker`的`onmessage`处理程序中添加一些内容，就像我们在`worker`内部所做的那样，以监听来自`worker`的事件。当我们收到一些数据时，我们将用返回的值填充答案：

```js
dedicated_worker.onmessage = function(ev) {
    answer.innerText = ev.data;
}
```

1.  最后，我们将更改我们的`worker`代码，利用`postMessage`方法将数据发送回主线程：

```js
self.onmessage = function(ev) {
    postMessage(calculatePrimes(ev.data).join(' '));
}
```

这也展示了我们不需要添加`self`来调用全局范围的方法。就像窗口是主线程的全局范围一样，`self`是工作线程的全局范围。

通过这个例子，我们已经探讨了`postMessage`方法，并看到了如何在工作线程和生成它的线程之间发送数据，但如果我们有多个选项卡想要进行通信怎么办？如果我们有多个工作线程想要发送消息怎么办？

处理这个问题的一种方法是跟踪所有的工作线程，并循环遍历它们，像下面这样发送数据：

```js
const workers = [];
for(let i = 0; i < 5; i++) {
    const worker = new Worker('test.js', {name : `worker${i}`});
    workers.push(worker);
}
document.querySelector("#in").addEventListener('change', (ev) => {
    for(let i = 0; i < workers.length; i++) {
        workers[i].postMessage(ev.target.value);
    }
});
```

在`test.js`文件中，我们只是控制台记录消息，并说明我们正在引用的工作线程的名称。这可能很快失控，因为我们需要跟踪哪些工作线程仍然存活，哪些已经被移除。处理这个问题的另一种方法是在一个通道上广播数据。幸运的是，我们有一个名为`BroadcastChannel`的 API 可以做到这一点。

正如 MDN 网站上的文档所述（[`developer.mozilla.org/en-US/docs/Web/API/Broadcast_Channel_API`](https://developer.mozilla.org/en-US/docs/Web/API/Broadcast_Channel_API)），我们只需要通过将单个参数传递给它的构造函数来创建一个`BroadcastChannel`对象，即通道的名称。谁先调用它就创建了通道，然后任何人都可以监听它。发送和接收数据就像我们的`postMessage`和`onmessage`示例一样简单。以下是我们先前用于测试界面的代码，而不需要跟踪所有工作线程，只需广播数据出去：

```js
const channel = new BroadcastChannel('workers');
document.querySelector("#in").addEventListener('change', (ev) => {
    channel.postMessage(ev.target.value);
});
```

然后，在我们的`workers`中，我们只需要监听`BroadcastChannel`，而不是监听我们自己的消息处理程序：

```js
const channel = new BroadcastChannel('workers');
channel.onmessage = function(ev) {
    console.log(ev.data, 'was received by', name);
}
```

现在，我们已经简化了在多个工作线程和甚至多个具有相同主机的选项卡之间发送和接收消息的过程。这个系统的优点在于，我们可以根据一些标准让一些工作线程监听一个通道，而让其他工作线程监听另一个通道。然后，我们可以有一个全局通道发送命令，任何工作线程都可以响应。让我们继续对我们的素数程序进行简单的调整。我们将不再将数据发送到单独的工作线程，而是将有四个工作线程；其中两个将处理偶数，另外两个将处理奇数：

1.  我们更新我们的主要代码以启动四个工作线程。我们将根据数字是偶数还是奇数来命名它们：

```js
for(let i = 0; i < 4; i++) {
    const worker = new Worker('worker.js', 
        {name : `worker ${i % 2 === 0 ? 'even' : 'odd'}`}
    );
}
```

1.  我们更改了输入后发生的事情，将偶数发送到偶数通道，将奇数发送到奇数通道：

```js
document.querySelector("#in").addEventListener('change', (ev) => {
    const value = parseInt(ev.target.value);
    if( value % 2 === 0 ) {
        even_channel.postMessage(value);
    } else {
        odd_channel.postMessage(value);
    }
});
```

1.  我们创建三个通道：一个用于偶数，一个用于奇数，一个用于全局发送给所有工作线程：

```js
const even_channel = new BroadcastChannel('even');
const odd_channel = new BroadcastChannel('odd');
const global = new BroadcastChannel('global');
```

1.  我们添加一个新按钮来终止所有工作线程，并将其连接到全局通道上广播：

```js
<button id="quit">Stop Workers</button>
<script type="text/javascript">
document.querySelector('#quit').addEventListener('click', (ev) => {
     global.postMessage('quit');
});
</script>
```

1.  我们更改我们的工作线程以根据其名称处理消息：

```js
const mainChannelName = name.includes("odd") ? "odd" : "even";
const mainChannel = new BroadcastChannel(mainChannelName);
```

1.  当我们在这些通道中的一个上收到消息时，我们会像以前一样做出响应：

```js
mainChannel.onmessage = function(ev) {
    if( typeof ev.data === 'number' )
        this.postMessage(calculatePrimes(ev.data));
}
```

1.  如果我们在全局通道上收到消息，我们检查它是否是`quit`消息。如果是，就终止工作线程：

```js
const globalChannel = new BroadcastChannel('global');
globalChannel.onmessage = function(ev) {
    if( ev.data === 'quit' ) {
        close();
    }
}
```

1.  现在，回到主线程，我们将监听奇数和偶数通道上的数据。当有数据时，我们几乎与以前处理它的方式完全相同：

```js
even_channel.onmessage = function(ev) {
    if( typeof ev.data === 'object' ) {
        answer.innerText = ev.data.join(' ');
    }
}
odd_channel.onmessage= function(ev) {
    if( typeof ev.data === 'object' ) {
        answer.innerText = ev.data.join(' ');
    }
}
```

需要注意的一点是我们的工作线程和主线程如何处理奇数和偶数通道上的数据。由于我们是广播，我们需要确保它是我们想要的数据。在工作线程的情况下，我们只想要数字，在主线程的情况下，我们只想要看到数组。

`BroadcastChannel` API 只能与相同的源一起使用。这意味着我们不能在两个不同的站点之间通信，只能在同一域下的页面之间通信。

虽然这是`BroadcastChannel`机制的一个过于复杂的例子，但它应该展示了我们如何可以轻松地将工作线程与其父级解耦，并使它们易于发送数据而无需循环遍历它们。现在，我们将回到`postMessage`方法，并查看`transferrable`属性以及它对发送和接收数据的意义。

# 在浏览器中发送二进制数据

虽然消息传递是发送数据的一种很好的方式，但在通过通道发送非常大的对象时会出现一些问题。例如，假设我们有一个专用的工作线程代表我们发出请求，并且还从缓存中向工作线程添加一些数据。它可能会有数千条记录。虽然工作线程已经占用了相当多的内存，但一旦我们使用`postMessage`，我们会看到两件事：

+   移动对象所需的时间会很长。

+   我们的内存将大幅增加

这是因为浏览器使用结构化克隆算法来发送数据。基本上，它不仅仅是将数据移动到通道上，而是将对象进行序列化和反序列化，从根本上创建多个副本。除此之外，我们不知道垃圾回收器何时运行，因为我们知道它是不确定的。

我们实际上可以在浏览器中看到复制过程。如果我们创建一个名为`largeObject.js`的工作线程并移动一个巨大的有效负载，我们可以通过利用`Date.now()`方法来测量所需的时间。除此之外，我们还可以利用开发者工具中的记录系统，就像我们在第一章中学到的那样，*网络高性能工具*，来分析我们使用的内存量。让我们设置这个测试案例：

1.  创建一个新的工作线程并分配一个大对象。在这种情况下，我们将使用一个存储对象的 100,000 元素数组：

```js
const dataToSend = new Array(100000);
const baseObj = {prop1 : 1, prop2 : 'one'};
for(let i = 0; i < dataToSend.length; i++) {
    dataToSend[i] = Object.assign({}, baseObj);
    dataToSend[i].prop1 = i;
    dataToSend[i].prop2 = `Data for ${i}`;
}
console.log('send at', Date.now());
postMessage(dataToSend);
```

1.  现在我们在 HTML 文件中添加一些代码来启动这个工作线程并监听消息。我们将标记消息到达的时间，然后对代码进行分析以查看内存增加情况：

```js
const largeWorker = new Worker('largeObject.js');
largeWorker.onmessage = function(ev) {
    console.log('the time is', Date.now());
    const obj = ev.data;
}
```

如果我们现在将其加载到浏览器中并对代码进行分析，我们应该会看到类似以下的结果。消息的时间在 800 毫秒到 1.7 秒之间，堆大小在 80MB 到 100MB 之间。虽然这种情况绝对超出了大多数人的范围，但它展示了这种消息传递方式的一些问题。

解决这个问题的方法是使用`postMessage`方法的可传递部分。这允许我们*发送*一个二进制数据类型通过通道，而不是复制它，通道实际上只是转移对象。这意味着发送方不再能够访问它，但接收方可以。可以这样理解，发送方将数据放在一个保持位置，并告诉接收方它在哪里。此时，发送方不再能够访问它。接收方接收所有数据，并注意到它有一个位置来查找数据。它去到这个位置并获取数据，从而实现数据传输机制。

让我们继续编写一个简单的例子。让我们使用大量数据填充我们的重型工作线程，比如从 1 到 1,000,000 的数字列表：

1.  我们创建一个包含 1,000,000 个元素的`Int32Array`。然后我们在其中添加从 1 到 1,000,000 的所有数字：

```js
const viewOfData = new Int32Array(1000000);
for(let i = 1; i <= viewOfData.length; i++) {
    viewOfData[i-1] = i;
}
```

1.  然后，我们将利用`postMessage`的可传递部分发送这些数据。请注意，我们必须获取基础的`ArrayBuffer`。我们很快会讨论这一点：

```js
postMessage(viewOfData, [viewOfData.buffer]);
```

1.  我们将在主线程上接收数据并输出该数据的长度：

```js
const obj = ev.data;
console.log('data length', obj.byteLength);
```

我们会注意到传输这一大块数据所花费的时间几乎是不可察觉的。这是因为前面的理论，它只是将数据打包并将其放到接收端。

对于类型化数组和`ArrayBuffers`需要额外说明。`ArrayBuffers`可以被视为 Node.js 中的缓冲区。它们是存储数据的最低形式，并直接保存一些数据的字节。但是，为了真正利用它们，我们需要在`ArrayBuffer`上放置一个*视图*。这意味着我们需要赋予`ArrayBuffer`意义。在我们的例子中，我们说它存储有符号的 32 位整数。我们可以在`ArrayBuffer`上放置各种视图，就像我们可以以不同的方式解释 Node.js 中的缓冲区一样。最好的思考方式是，`ArrayBuffer`是我们真正不想使用的低级系统，而视图是赋予底层数据意义的系统。

考虑到这一点，如果我们在工作线程端检查`Int32Array`的字节长度，我们会发现它是零。我们不再可以访问那些数据，正如我们所说的。在继续讨论`SharedWorkers`和`SharedArrayBuffers`之前，我们将修改我们的因式分解程序，利用这个可传递属性发送因子：

1.  我们将几乎使用完全相同的逻辑，只是不再发送我们拥有的数组，而是发送`Int32Array`：

```js
if( typeof ev.data === 'number' ) {
    const result = calculatePrimes(ev.data);
    const send = new Int32Array(result);
    this.postMessage(result, [result.buffer]);
}
```

1.  现在我们将更新接收端代码，以处理发送的`ArrayBuffers`而不仅仅是一个数组：

```js
if( typeof ev.data === 'object' ) {
    const data = new Int32Array(ev.data);
    answer.innerText = data.join(' ');                  
}
```

如果我们测试这段代码，我们会发现它的工作方式是一样的，但我们不再复制数据，而是将其交给主线程，从而使消息传递更快，利用的内存更少。

主要思想是，如果我们只是发送结果或需要尽快完成，我们应该尝试利用可传递系统发送数据。如果我们在发送数据后需要在工作线程中使用数据，或者没有简单的方法发送数据（我们没有序列化技术），我们可以利用正常的`postMessage`系统。

仅仅因为我们可以使用可传递系统来减少内存占用，这可能会导致基于需要应用的数据转换量而增加时间。如果我们已经有二进制数据，这很好，但如果我们有需要移动的 JSON 数据，可能最好的方法是以该形式传输它，而不是经过许多中间转换。

有了所有这些想法，让我们来看看`SharedWorker`系统和`SharedArrayBuffer`系统。这两个系统，特别是`SharedArrayBuffer`，在过去引起了一些问题（我们将在下一节讨论），但如果我们小心使用它们，我们将能够利用它们作为良好的消息传递和数据共享机制的能力。

# 共享数据和工作线程

虽然大多数时候我们希望保持工作线程和应用程序选项卡之间的边界，但有时我们希望只是共享数据，甚至是工作线程。在这种情况下，我们可以利用两个系统，`SharedWorker`和`SharedArrayBuffer`。

`SharedWorker`就像它的名字一样，当一个启动时，就像`BroadcastChannel`一样，当其他人调用创建`SharedWorker`时，它将连接到已经创建的实例。让我们继续做这件事：

1.  我们将为`SharedWorker` JavaScript 代码创建一个新文件。在这里面，放一些通用的计算函数，比如加法和减法：

```js
const add = function(a, b) {
    return a + b;
}
const mult = function(a, b) {
    return a * b;
}
const divide = function(a, b) {
    return a / b;
}
const remainder = function(a, b) {
    return a % b;
}
```

1.  在我们当前某个工作线程的代码中，启动`SharedWorker`：

```js
const shared = new SharedWorker('shared.js');
shared.port.onmessage = function(ev) {
    console.log('message', ev);
}
```

我们已经看到了一个问题。我们的系统显示找不到`SharedWorker`。要使用`SharedWorker`，我们必须在一个窗口中启动它。所以现在，我们将不得不将启动代码移动到我们的主页面。

1.  将启动代码移动到主页面，然后将端口传递给其中一个工作线程：

```js
const shared = new SharedWorker('shared.js');
shared.port.start();
for(let i = 0; i < 4; i++) {
    const worker = new Worker('worker.js', 
        {name : `worker ${i % 2 === 0 ? 'even' : 'odd'}`}
    );
    worker.postMessage(shared.port, [shared.port]);
}
```

我们现在遇到另一个问题。由于我们想要将端口传递给工作线程，并且不希望在主窗口中访问它，所以我们利用了可传递的系统。然而，由于那时我们只有一个引用，一旦我们将它发送给一个工作线程，就无法再次发送。相反，让我们启动一个工作线程，并关闭我们的`BroadcastChannel`系统。

1.  注释掉我们的`BroadcastChannels`和所有的循环代码。让我们只在这个窗口中启动一个工作线程：

```js
const shared = new SharedWorker('shared.js');
shared.port.start();
const worker = new Worker('worker.js');
document.querySelector("#in").addEventListener('change', (ev) => {
    const value = parseInt(ev.target.value);
    worker.postMessage(value);
});
document.querySelector('#quit').addEventListener('click', (ev) => {
    worker.postMesasge('quit');
});
```

1.  有了这些改变，我们将不得不简化我们的专用工作线程。我们将只是像以前一样响应我们消息通道上的事件：

```js
let sharedPort = null;
onmessage = function(ev) {
    const data = ev.data;
    if( typeof data === 'string' ) {
        return close();
    }
    if( typeof data === 'number' ) {
        const result = calculatePrimes(data);
        const send = new Int32Array(result);
        return postMessage(send, [send.buffer]);
    }
    // handle the port
    sharedPort = data;
}
```

1.  现在我们在一个单一的工作线程中有了`SharedWorker`端口，但是这对我们解决了什么问题呢？现在，我们可以同时打开多个选项卡，并将数据发送到每一个选项卡。为了看到这一点，让我们将一个处理程序连接到`sharedPort`：

```js
sharedPort.onmessage = function(ev) {
    console.log('data', ev.data);
}
```

1.  最后，我们可以更新我们的`SharedWorker`，一旦连接发生，就做出响应，如下所示：

```js
onconnect = function(e) {
    let port = e.ports[0];
    console.log('port', port);
    port.onmessage = function(e) {
        port.postMessage('you sent data');
    }
    port.postMessage('you connected');
}
```

有了这个，我们将看到一个消息回到我们的工作线程。我们现在的`SharedWorker`已经运行起来，并且直接与我们的`DedicatedWorker`进行通信！然而，仍然有一个问题：为什么我们没有看到来自我们的`SharedWorker`的日志？嗯，我们的`SharedWorker`存在于与我们的`DedicatedWorker`和主线程不同的上下文中。要访问我们的`SharedWorker`，我们可以转到 URL`chrome://inspect/#workers`，然后定位它。现在，我们没有给它起名字，所以它应该叫做`untitled`，但是当我们点击它下面的`inspect`选项时，我们现在有了一个工作线程的调试上下文。

我们已经将我们的`SharedWorker`连接到 DOM 上下文，并且已经将每个`DedicatedWorker`连接到该`SharedWorker`，但是我们需要能够向每个`DedicatedWorker`发送消息。让我们继续添加这段代码：

1.  首先，我们需要跟踪所有通过`SharedWorker`连接到我们的工作线程。将以下代码添加到我们`onconnect`监听器的底部：

```js
ports.push(port);
```

1.  现在，我们将在我们的文档中添加一些 HTML，这样我们就可以发送`add`、`multiply`、`divide`和`subtract`请求，以及两个新的数字输入：

```js
<input id="in1" type="number" />
<input id="in2" type="number" />
<button id="add">Add</button>
<button id="subtract">Subtract</button>
<button id="multiply">Multiply</button>
<button id="divide">Divide</button>
```

1.  接下来，我们将通过`DedicatedWorker`将这些信息传递给`SharedWorker`：

```js
if( typeof data === 'string' ) {
    if( data === 'quit' ) {
        close();
    } else {
        sharedPort.postMessage(data);
    }
}
```

1.  最后，我们的`SharedWorker`将运行相应的操作，并将其传递回`DedicatedWorker`，后者将数据记录到控制台：

```js
port.onmessage = function(e) {
    const _d = e.data.split(' ');
    const in1 = parseInt(_d[1]);
    const in2 = parseInt(_d[2]);
    switch(_d[0]) {
        case 'add': {
            port.postMessage(add(in1, in2));
            break;
        }
        // other operations removed since they are the same thing
    }
}
```

有了这一切，我们现在可以打开多个应用程序选项卡，它们都共享相同的前置数学系统！对于这种类型的应用程序来说，这有点过度，但是当我们需要在我们的应用程序中执行跨多个窗口或选项卡的复杂操作时，这可能是有用的。这可能是利用 GPU 的东西，我们只想做一次。让我们通过概述`SharedArrayBuffer`来结束本节。然而，要记住的一件事是，`SharedWorker`是所有选项卡持有的单个线程，而`DedicatedWorker`是每个选项卡/窗口的一个线程。虽然共享一个工作线程对于前面解释的一些任务可能是有益的，但如果多个选项卡同时使用它，也可能会减慢其他任务的速度。

`SharedArrayBuffer`允许我们的所有实例共享相同的内存块。就像可传递的对象可以根据将内存传递给另一个工作线程而有不同的所有者一样，`SharedArrayBuffer`允许不同的上下文共享相同的部分。这允许更新在我们的所有实例中传播，并且对于某些类型的数据几乎立即更新，但它也有许多与之相关的缺点。

这是我们在其他语言中最有可能接近`SharedMemory`的方式。要正确使用`SharedArrayBuffer`，我们需要使用 Atomics API。再次强调，不直接深入 Atomics API 背后的细节，它确保操作按正确顺序进行，并且保证在更新时能够更新需要更新的内容，而不会被其他人在更新过程中覆盖。

我们开始进入细节，这些细节可能很难完全理解发生了什么。一个好的理解 Atomics API 的方式是将其想象成一个许多人共享一张纸的系统。他们轮流在上面写字和阅读其他人写下的内容。

然而，其中一个缺点是他们一次只能写一个字符。因此，当他们仍在尝试完成写入单词时，其他人可能会在他们的位置上写入内容，或者有人可能会读取他们的不完整短语。我们需要一个机制，让人们能够在开始写入之前写入他们想要的整个单词，或者在开始写入之前读取整个部分。这就是 Atomics API 的工作。

`SharedArrayBuffer`确实存在一些问题，与浏览器不支持它有关（目前，只有 Chrome 支持它而无需标志），以及我们可能希望使用 Atomics API（由于安全问题，`SharedWorker`无法将其发送到主线程或专用 worker）。

为了设置`SharedArrayBuffer`的基本示例，我们将在主线程和 worker 之间共享一个缓冲区。当我们向 worker 发送请求时，我们将更新 worker 中的数字。更新这个数字应该对主线程可见，因为它们共享缓冲区。

1.  创建一个简单的 worker，并使用`onmessage`处理程序检查是否收到了一个数字。如果是，我们将增加`SharedArrayBuffer`中的数据。否则，数据是来自主线程的`SharedArrayBuffer`。

```js
let sharedPort = null;
let buf = null;
onmessage = function(ev) {
    const data = ev.data;
    if( typeof data === 'number' ) {
        Atomics.add(buf, 0, 1);
    } else {
        buf = new Int32Array(ev.data);
    }
}
```

1.  接下来，在我们的主线程上，我们将添加一个新的按钮，上面写着“增加”。当点击它时，它将向专用 worker 发送一条消息，以增加当前数字。

```js
// HTML
<button id="increment">Increment</button>
<p id="num"></p>

// JavaScript
document.querySelector('#increment').addEventListener('click', () => {
    worker.postMessage(1);
});
```

1.  现在，当 worker 在其端更新缓冲区时，我们将不断检查`SharedArrayBuffer`是否有更新。我们将始终将数字放在前面代码片段中显示的数字段落元素中。

```js
setInterval(() => {
    document.querySelector('#num').innerText = shared;
}, 100);
```

1.  最后，为了开始所有这些，我们将在主线程上创建一个`SharedArrayBuffer`，并在启动后将其发送给 worker：

```js
let shared = new SharedArrayBuffer(4);
const worker = new Worker('worker_to_shared.js');
worker.postMessage(shared);
shared = new Int32Array(shared);
```

通过这样，我们可以看到我们的值现在正在增加，即使我们没有从 worker 发送任何数据到主线程！这就是共享内存的力量。现在，正如之前所述，由于我们无法在主线程上使用`wait`和`notify`系统，也无法在`SharedWorker`中使用`SharedArrayBuffer`，因此我们在 Atomics API 方面受到相当大的限制，但它对于只读取数据的系统可能是有用的。

在这些情况下，我们可能会更新`SharedArrayBuffer`，然后向主线程发送一条消息，告诉它我们已经更新了它，或者它可能已经是一个接受`SharedArrayBuffers`的 Web API，比如 WebGL 渲染上下文。虽然前面的例子并不是很有用，但它展示了如果再次可以在`SharedWorker`中生成和使用`SharedArrayBuffer`的能力，我们可能如何在未来使用共享系统。接下来，我们将专注于构建一个所有 worker 都可以共享的单一缓存。

# 构建一个简单的共享缓存

通过我们学到的一切，我们将专注于一个在报告系统和大多数类型的操作 GUI 中非常普遍的用例——需要添加其他数据的大块数据（有些人称之为装饰数据，其他人称之为属性）。一个例子是我们有一组客户的买入和卖出订单。

这些数据可能以以下方式返回：

```js
{
    customerId : "<guid>",
    buy : 1000000,
    sell : 1000000
}
```

有了这些数据，我们可能想要添加一些与客户 ID 相关联的上下文。我们可以通过两种方式来做到这一点：

+   首先，我们可以在数据库中执行联接操作，为用户添加所需的信息。

+   其次，我们将在此处进行说明的是，在我们获得基本查询时在前端添加这些数据。这意味着当我们的应用程序启动时，我们将获取所有这些归因数据并将其存储在某个后台缓存中。接下来，当我们发出请求时，我们还将向缓存请求相应的数据。

为了实现第二个选项，我们将实现我们之前学到的两种技术，`SharedWorker`和`postMessage`接口：

1.  我们创建一个基本级别的 HTML 文件，其中包含每一行数据的模板。我们不会深入创建 Web 组件，就像我们在第三章中所做的那样，但我们将使用它来根据需要创建我们的表行：

```js
<body>
    <template id="row">
        <tr>
            <td class="name"></td>
            <td class="zip"></td>
            <td class="phone"></td>
            <td class="email"></td>
            <td class="buy"></td>
            <td class="sell"></td>
        </tr>
    </template>
   <table id="buysellorders">
   <thead>
       <tr>
           <th>Customer Name</th>
           <th>Zipcode</th>
           <th>Phone Number</th>
           <th>Email</th>
           <th>Buy Order Amount</th>
           <th>Sell Order Amount</th>
       </tr>
   </thead>
   <tbody>
   </tbody>
   </table>
</body>
```

1.  我们设置了一些指向我们模板和表的指针，以便我们可以快速插入。除此之外，我们可以为即将创建的`SharedWorker`创建一个占位符：

```js
const tableBody = document.querySelector('#buysellorders > tbody');
const rowTemplate = document.querySelector('#row');
const worker = new SharedWorker('<fill in>', {name : 'cache'});
```

1.  有了这个基本设置，我们可以创建我们的`SharedWorker`并为其提供一些基本数据。为此，我们将使用网站[`www.mockaroo.com/`](https://www.mockaroo.com/)。这将允许我们创建大量随机数据，而无需自己考虑。我们可以将数据更改为我们想要的任何内容，但在我们的情况下，我们将选择以下选项：

+   `id`：行号

+   `full_name`：全名

+   `email`：电子邮件地址

+   `phone`：电话

+   `zipcode`：数字序列：`######`

1.  填写了这些选项后，我们可以将格式更改为 JSON，并通过单击“下载数据”进行保存。完成后，我们可以构建我们的`SharedWorker`。与我们的其他`SharedWorker`类似，我们将使用`onconnect`处理程序，并为传入的端口添加一个`onmessage`处理程序：

```js
onconnect = function(e) {
    let port = e.ports[0];
    port.onmessage = function(e) {
        // do something
    }
}
```

1.  接下来，在我们的 HTML 文件中启动我们的`SharedWorker`：

```js
const worker = new SharedWorker('cache_shared.js', 'cache');
```

1.  现在，当我们启动我们的`SharedWorker`时，我们将使用`importScripts`加载文件。这允许我们加载外部 JavaScript 文件，就像我们在 HTML 中使用`script`标签一样。为此，我们需要修改 JSON 文件，将对象指向一个变量并将其重命名为 JavaScript 文件：

```js
let cache = [{"id":1,"full_name":"Binky Bibey","email":"bbibey0@furl.net","phone":"370-576-9587","zipcode":"640069"}, //rest of the data];

// SharedWorker.js
importScripts('./mock_customer_data.js');
```

1.  现在我们已经将数据缓存进来，我们将回应从端口发送来的消息。我们只期望数字数组。这些将对应于与用户关联的 ID。现在，我们将循环遍历字典中的所有项目，看看我们是否有它们。如果有，我们将将它们添加到一个数组中，然后进行响应：

```js
const handleReq = function(arr) {
    const res = new Array(arr.length)
    for(let i = 0; i < arr.length; i++) {
        const num = arr[i];
        for(let j = 0; j < cache.length; j++) {
            if( num === cache[j].id ) {
                res[i] = cache[j];
               break;
            }
        }
    }
    return res;
}
onconnect = function(e) {
    let port = e.ports[0];
    port.onmessage = function(e) {
        const request = e.data;
        if( Array.isArray(request) ) {
            const response = handleReq(request);
            port.postMessage(response);
        }
    }
}
```

1.  因此，我们需要在我们的 HTML 文件中添加相应的代码。我们将添加一个按钮，该按钮将向我们的`SharedWorker`发送 100 个随机 ID。这将模拟当我们发出请求并获得与数据关联的 ID 时的情况。模拟函数如下：

```js
// developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/
// Global_Objects/Math/random

const getRandomIntInclusive = function(min, max) {
    return Math.floor(Math.random() * (max - min + 1)) + min;
}
const simulateRequest = function() {
    const MAX_BUY_SELL = 1000000;
    const MIN_BUY_SELL = -1000000;
    const ids = [];
    const createdIds = [];
    for(let i = 0; i < 100; i++) {
        const id = getRandomIntInclusive(1, 1000);
        if(!createdIds.includes(id)) {
            const obj = {
                id,
                buy : getRandomIntInclusive(MIN_BUY_SELL,  
                 MAX_BUY_SELL),
                sell : getRandomIntInclusive(MIN_BUY_SELL, 
                 MAX_BUY_SELL)
            };
            ids.push(obj);
        }
    }
    return ids;
}
```

1.  通过上述模拟，我们现在可以添加我们的请求输入，然后将其发送到我们的`SharedWorker`：

```js
requestButton.addEventListener('click', (ev) => {
    const res = simulateRequest();
    worker.port.postMessage(res);
});
```

1.  现在，我们目前正在向我们的`SharedWorker`发布错误的数据。我们只想发布 ID，但是我们如何将我们的请求与我们的`SharedWorker`的响应联系起来呢？我们需要稍微修改我们的`request`和`response`方法的结构。我们现在将 ID 绑定到我们的消息，这样我们就可以让`SharedWorker`将其发送回给我们。这样，我们就可以在前端拥有请求和与之关联的 ID 的映射。进行以下更改：

```js
// HTML file
const requestMap = new Map();
let reqCounter = 0;
requestButton.addEventListener('click', (ev) => {
    const res = simulateRequest();
    const reqId = reqCounter;
    reqCounter += 1;
    worker.port.postMessage({
        id : reqId,
        data : res
    });
});

// Shared worker
port.onmessage = function(e) {
    const request = e.data;
    if( request.id &&
        Array.isArray(request.data) ) {
        const response = handleReq(request.data);
        port.postMessage({
            id : request.id,
            data : response
        });
    }
}
```

1.  通过这些更改，我们仍然需要确保我们只将 ID 传递给`SharedWorker`。在发送请求之前，我们可以从请求中取出这些 ID：

```js
requestButton.addEventListener('click', (ev) => {
    const res = simulateRequest();
    const reqId = reqCounter;
    reqCounter += 1;
    requestMap.set(reqId, res);
    const attribute = [];
    for(let i = 0; i < res.length; i++) {
        attribute.push(res[i].id);
    }
    worker.port.postMessage({
        id : reqId,
        data : attribute
    });
});
```

1.  现在我们需要处理返回到我们的 HTML 文件中的数据。首先，我们将一个`onmessage`处理程序附加到端口上：

```js
worker.port.onmessage = function(ev) {
    console.log('data', ev.data);
}
```

1.  最后，我们从地图中获取相关的买卖订单，并用返回的缓存数据填充它。完成这些后，我们只需克隆我们的行模板并填写相应的字段：

```js
worker.port.onmessage = function(ev) {
    const data = ev.data;
    const baseData = requestMap.get(data.id);
    requestMap.delete(data.id);
    const attribution = data.data;
    tableBody.innerHTML = '';
    for(let i = 0; i < baseData.length; i++) {
        const _d = baseData[i];
        for(let j = 0; j < attribution.length; j++) {
            if( _d.id === attribution[j].id ) {
                const final = {..._d, ...attribution[j]};
                const newRow = rowTemplate.content.cloneNode(true);
                newRow.querySelector('.name').innerText =  
                 final.full_name;
                newRow.querySelector('.zip').innerText = 
                 final.zipcode;
                newRow.querySelector('.phone').innerText = 
                 final.phone;
                newRow.querySelector('.email').innerText = 
                 final.email;
                newRow.querySelector('.buy').innerText = 
                 final.buy;
                newRow.querySelector('.sell').innerText = 
                 final.sell;
                tableBody.appendChild(newRow);
            }
        }
    }
}
```

通过上面的例子，我们创建了一个任何具有相同域的页面都可以使用的共享缓存。虽然有一些优化（我们可以将数据存储为地图，并将 ID 作为键），但我们仍然会比潜在地等待数据库连接要快一些（特别是当我们在带宽有限的地方时）。

# 总结

整个章节都集中在将任务从主线程转移到其他工作线程上。我们看了只有单个页面才有的专用工作线程。然后我们看了如何在多个工作线程之间广播消息，而不必循环遍历各自的端口。

然后我们看到了如何在同一域上利用`SharedWorker`共享工作线程，还看了如何利用`SharedArrayBuffer`共享数据源。最后，我们实际看了一下如何创建一个任何人都可以访问的共享缓存。

在下一章中，我们将通过利用`ServiceWorker`将缓存和处理请求的概念推进一步。
