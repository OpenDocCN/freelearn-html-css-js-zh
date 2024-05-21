# 第七章：流-理解流和非阻塞 I/O

我们已经涉及了几乎所有帮助我们使用 JavaScript 为服务器编写高性能代码的主题。应该讨论的最后两个主题是流和数据格式。虽然这两个主题可以并驾齐驱（因为大多数数据格式是通过读/写流实现的），但我们将在本章中重点关注流。

流使我们能够编写可以处理数据而不占用大量工作内存并且不阻塞事件队列的系统。对于那些一直按顺序阅读本书的人来说，这可能听起来很熟悉，这是正确的。我们将重点关注 Node.js 提供的四种不同类型的流，以及我们可以轻松扩展的流。从那里，我们将看看如何结合流和生成器来处理具有内置生成器概念的数据。

本章涵盖以下主题：

+   流基础知识

+   可读流

+   可写流

+   双工流

+   转换流

+   附注-生成器和流

# 技术要求

本章的先决条件如下：

+   一个代码编辑器或 IDE，最好是 VS Code

+   可以运行 Node.js 的操作系统

+   在[`github.com/PacktPublishing/Hands-On-High-Performance-Web-Development-with-JavaScript/tree/master/Chapter07`](https://github.com/PacktPublishing/Hands-On-High-Performance-Web-Development-with-JavaScript/tree/master/Chapter07)找到的代码。

# 开始使用流

流是处理无限数据集的行为。这并不意味着它是无限的，但是意味着我们有可能拥有无限的数据源。如果我们从传统的数据处理上下文来思考，通常会经历三个主要步骤：

1.  打开/获取对数据源的访问。

1.  一旦数据源完全加载，就处理数据源。

1.  将计算出的数据输出到另一个位置。

我们可以将其视为**输入和输出**（**I/O**）的基础。我们的大多数 I/O 概念涉及批处理或处理所有或几乎所有数据。这意味着我们提前知道数据的限制。我们可以确保我们有足够的内存、存储空间、计算能力等来处理这个过程。一旦我们完成了这个过程，我们就会终止程序或排队下一批数据。

一个简单的例子如下所示，我们计算文件的行数：

```js
import { readFileSync } from 'fs'
const count = readFileSync('./input.txt', {encoding : 'utf8'})
 .split(/\n|\r\n/g).length;
console.log('number of lines in our file is: ', count);
```

我们从`fs`模块中引入`readFileSync`方法，然后读取`input.txt`文件。从这里开始，我们在`\n`或`\r\n`上拆分，这给我们一个文件所有行的数组。从那里，我们得到长度并将其放在我们的标准输出通道上。这似乎非常简单，而且似乎运行得很好。对于小到中等长度的文件，这很好用，但是当文件变得异常大时会发生什么呢？让我们继续看下去。前往[`loremipsum.io`](https://loremipsum.io)并输入 100 段落。将其复制并粘贴几次到`input.txt`文件中。现在，当我们运行这个程序时，我们可以在任务管理器中看到内存使用量的飙升。

我们将一个大约 3MB 的文件加载到内存中，计算换行符的数量，然后打印出来。这应该仍然非常快，但我们现在开始利用大量内存。让我们用这个文件做一些更复杂的事情。我们将计算文本中单词`lorem`出现的次数。我们可以使用以下代码来实现：

```js
import { readFileSync } from 'fs'
const file = readFileSync('./input.txt', {encoding : 'utf8'});
const re = /\slorem\s/gi;
const matches = file.match(re);

console.log('the number of matches is: ', matches.length);
```

同样，这应该处理得很快，但在处理方式上可能会有一些滞后。虽然在这里使用正则表达式可能会给我们一些错误的结果，但它确实展示了我们在这个文件上进行批处理。在许多情况下，当我们在高速环境中工作时，我们处理的文件可能接近或超过 1GB。当我们处理这些类型的文件时，我们不希望将它们全部加载到内存中。这就是流的作用所在。

许多被认为是大数据的系统正在处理几 TB 的数据。虽然有一些内存应用程序会将大量数据存储在内存中，但这种类型的数据处理大部分使用文件流和使用内存数据源来处理数据的混合。

让我们拿第一个例子来说。我们正在从文件中读取，并尝试计算文件中的行数。嗯，与其考虑整个行数，我们可以寻找表示换行的字符。我们在正则表达式中寻找的字符是换行符（`\n`）或回车加换行（`\r\n`）字符。有了这个想法，我们应该能够构建一个流应用程序，它可以读取文件并计算行数，而不需要完全将文件加载到内存中。

这个例子介绍了利用流的 API。我们将讨论每个流 API 给我们的东西，以及我们如何利用它来实现我们的目的。现在，拿出代码示例并运行它们，看看这些类型的应用是如何工作的。

这可以在以下代码片段中看到：

```js
import { createReadStream } from 'fs';

const newLine = 0x0A;
const readStream = createReadStream('./input.txt');
let counter = 1;
readStream.on('data', (chunk) => {
    for(const byte of chunk) {
        if( newLine === byte ) counter += 1;
    }
}).on('end', () => {
    console.log('number of line in our file is: ', counter);
});
```

我们从`fs`模块中获取一个`Readable`流并创建一个。我们还为 HEX 格式中表示的换行符创建一个常量。然后，我们监听数据事件，以便在数据到达时处理数据。然后，我们处理每个字节，看它是否与换行符相同。如果是，那么我们有一个换行符，否则我们继续搜索。我们不需要明确寻找回车符，因为我们知道它应该后跟一个换行符。

虽然这比将整个文件加载到内存中要慢，但在处理数据时它确实节省了我们相当多的内存。这种方法的另一个好处是这些都是事件。在我们的完整处理示例中，我们占用整个事件循环，直到处理完成。而使用流，我们有事件来处理数据进来。这意味着我们可以在同一个线程上同时运行多个流，而不必太担心阻塞（只要我们在数据块的处理上不花费太多时间）。

通过前面的例子，我们可以看到如何以流的形式编写反例。为了更好地说明问题，让我们继续做到这一点。它应该看起来像下面这样：

```js
const stream = createReadStream('./input.txt');
const buf = Buffer.from('lorem');
let found = 0;
let count = 0;
stream.on('data', (chunk) => {
    for(const byte of chunk) {
        if( byte === buf[found] ) {
            found += 1;
        } else {
            found = 0;
        }
        if( found === buf.byteLength ) {
            count += 1;
            found = 0;
        }
    }
}).on('end', () => {
    console.log('the number of matches is: ', count)
});
```

首先，我们创建一个读取`stream`，就像以前一样。接下来，我们创建一个关键字的`Buffer`形式，我们正在寻找的关键字（在原始字节上工作可能比尝试将流转换为文本更快，即使 API 允许我们这样做）。接下来，我们维护一个`found`计数和一个`actual`计数。`found`计数将告诉我们是否找到了这个单词；另一个计数跟踪我们找到了多少个`lorem`实例。接下来，当数据事件上的一个块到来时，我们处理每个字节。如果我们发现下一个字节不是我们要找的字符，我们会自动将`found`计数返回为`0`（我们没有找到这个特定的文本字符串）。在这个检查之后，我们将看到我们是否找到了完整的字节长度。如果是，我们可以增加计数并将`found`移回`0`。我们将`found`计数器保留在数据事件之外，因为我们以块接收数据。由于它是分块的，`lorem`的一部分可能出现在一个块的末尾，而`lorem`的另一部分可能出现在下一个块的开头。一旦流结束，我们就输出计数。

现在，如果我们运行两个版本，我们会发现第一个实际上捕获了更多的`lorem`。我们为正则表达式添加了不区分大小写的标志。如果我们通过删除末尾的`i`来关闭它，并且我们删除字符序列周围的`\s`，我们将看到我们得到相同的结果。这个例子展示了写流可能比批处理版本更复杂一些，但通常会导致更低的内存使用和更快的代码。

虽然利用内置流（如`zlib`和`fs`模块中的流）可以让我们走得更远，但我们将看到如何成为我们自己自定义流的生产者。我们将每个流都写成一个扩展流类型，以处理我们在上一章中所做的数据框架。

对于那些忘记或跳到本章的人，我们正在通过套接字对所有消息进行框架处理，使用`!!!BEGIN!!!`和`!!!END!!!`标记来告诉我们何时将完整数据流式传输给我们。

# 构建自定义可读流

`Readable`流确切地做了它所声明的事情，它从流源中读取。它根据某些标准输出数据。我们的例子是对 Node.js 文档中显示的简单示例的一种理解。

我们将以计算文本文件中`lorem`的数量为例，但我们将输出在文件中找到`lorem`的位置：

1.  从各自的模块中导入`Readable`类和`createReadStream`方法：

```js
import { Readable } from 'stream'
import { createReadStream } from 'fs'
```

1.  创建一个扩展`Readable`类的类，并设置一些私有变量来跟踪内部状态：

```js
class LoremFinder extends Readable {
    #lorem = Buffer.from('lorem');
    #found = 0;
    #totalCount = 0;
    #startByteLoc = -1;
    #file = null;
}
```

1.  添加一个构造函数，将我们的`#file`变量初始化为`Readable`流：

```js
// inside our LoremFinder class
constructor(opts) {
    super(opts); 
    if(!opts.stream ) { 
        throw new Error("This stream needs a stream to be 
         provided!");
    }
    this.#file = opts.stream;
    this.#file.on('data', this.#data.bind(this)); // will add #data 
     method next
    this.#file.on('end', () => this.push(null)); 
}
```

1.  根据构造函数，我们将利用一个`#data`私有变量，它将是一个函数。我们将利用它来从我们的`#file`流中读取，并检查`lorem`的位置：

```js
// inside of the LoremFinder class
#data = function(chunk) {
    for(let i = 0; i < chunk.byteLength; i++) {
        const byte = chunk[i];
        if( byte === this.#lorem[this.#found] ) {
            if(!this.#found ) {
                this.#startByteLoc = this.#totalCount + i; 
            }
            this.#found += 1;
        } else {
            this.#found = 0;
        }
        if( this.#found === this.#lorem.byteLength ) {
            const buf = Buffer.alloc(4);
            buf.writeUInt32BE(this.#startByteLoc);
            this.push(buf);
            this.#found = 0;
        }
    }
    this.#totalCount += chunk.byteLength;
}
```

我们遍历每个字节，并检查我们当前是否拥有我们在`lorem`单词中寻找的字节。如果我们找到了，并且它是单词的`l`，那么我们设置我们的位置`#startByteLoc`变量。如果我们找到整个单词，我们输出`#startByteLoc`，否则，我们重置我们的查找变量并继续循环。一旦我们完成循环，我们将我们读取的字节数添加到我们的`#totalCount`中，并等待我们的`#data`函数再次被调用。为了结束我们的流并让其他人知道我们已完全消耗了资源，我们输出一个`null`值。

1.  我们添加的最后一部分是`_read`方法。

这将通过`Readable.read`方法或通过挂接数据事件来调用。这是我们如何确保*原始*流（如`FileStream`）被消耗：

```js
// inside of the LoremFinder class
_read(size) {
    this.#file.resume();
}
```

1.  现在我们可以添加一些测试代码来确保这个流正常工作：

```js
const locs = new Set();
const loremFinder = new LoremFinder({
    stream : createReadStream('./input.txt')
});
loremFinder.on('data', (chunk) => {
    const num = chunk.readUInt32BE();
    locs.add(num);
});
loremFinder.on('end', () => {
    console.log('here are all of the locations:');
    for(const val of locs) {
        console.log('location: ', val);
    }
    console.log('number of lorems found is', locs.size);
});
```

通过所有这些概念，我们可以看到我们如何能够消耗原始流并能够用超集流包装它们。现在我们有了这个流，我们可以随时使用管道接口并将其管道到`Writable`流中。让我们将索引写入文件。为此，我们可以做一些简单的事情，比如`loremFinder.pipe(writeable)`。

如果我们打开文件，我们会发现它只是一堆随机数据。原因是我们将所有索引编码到 32 位缓冲区中。如果我们想看到它们，我们可以稍微修改我们的流实现。修改可能如下所示：`this.push(this.#startByteLoc.toString() + "\r\n");`。

通过这种修改，我们现在可以查看`output.txt`文件并查看所有索引。如果我们只是不断地将它们通过各种阶段进行管道传输，代码变得多么可读。

# 理解可读流接口

`Readable`流有一些可用的属性。它们都在 Node.js 文档中有解释，但我们感兴趣的主要是`highWaterMark`和`objectMode`。

`highWaterMark`允许我们声明内部缓冲区在流声明无法再接收任何数据之前应该容纳多少数据。我们实现的一个问题是我们没有处理暂停。如果达到了这个`highWaterMark`，流就会暂停。虽然大多数情况下我们可能不担心这个问题，但它可能会引起问题，通常是流实现者会遇到问题的地方。通过设置更高的`highWaterMark`，我们可以防止这些问题。另一种处理方法是检查运行`this.push`的结果。如果返回`true`，那么我们可以向流写入更多数据，否则，我们应该暂停流，然后在从另一个流得到信号时恢复。流的默认`highWaterMark`大约为 16 KB。

`objectMode` 允许我们构建不基于`Buffer`的流。当我们想要遍历对象列表时，这非常有用。我们可以设置一个管道系统，通过流传递对象并对其执行某种操作，而不是使用`for`循环或`map`函数。我们不仅限于普通的对象，而是几乎可以使用除`Buffer`之外的任何数据类型。关于`objectMode`的一点需要注意的是它改变了`highWaterMark`的计数方式。它不再计算存储在内部缓冲区中的数据量，而是计算直到暂停流之前将存储的对象数量。默认值为`16`，但如果需要，我们可以随时更改它。

有了这两个属性的解释，我们应该讨论一下可用的各种内部方法。对于每种流类型，都有一个我们*需要*实现的方法和一些我们*可以*实现的方法。

对于`Readable`流，我们只需要实现`_read`方法。这个方法给我们一个`size`参数，表示从底层数据源中读取的字节数。我们不总是需要遵循这个数字，但如果需要，它是可用的。

除了`_read`方法，我们需要使用`push`方法。这是将数据推送到内部缓冲区并帮助发出数据事件的方法，正如我们之前所见。正如我们之前所述，`push`方法返回一个布尔值。如果这个值为`true`，我们可以继续使用`push`，否则，我们应该停止推送数据，直到我们的`_read`实现再次被调用。

正如之前所述，当首次实现`Readable`流时，返回值可以被忽略。但是，如果我们注意到数据没有流动或数据丢失，通常的罪魁祸首是`push`方法返回了`false`，而我们继续尝试向流中推送数据。一旦发生这种情况，我们应该通过停止使用`push`方法直到再次调用`_read`来实现暂停。

可读接口的另外两个部分是`_destroy`方法以及如何使我们的流在无法处理的情况下出错。如果有任何低级资源需要释放，应该实现`_destroy`方法。

这可以是使用`fs.open`命令打开的文件句柄，也可以是使用`net`模块创建的套接字。如果发生错误，我们也应该使用它来发出错误事件。

为了处理流可能出现的错误，我们应该通过`this.emit`系统发出错误。如果我们抛出错误，根据文档，可能会导致意外的结果。通过发出错误，我们让流的用户处理错误并根据他们的意愿处理它。

# 实现可读流

根据我们在这里学到的知识，让我们实现我们之前讨论过的帧系统。从我们之前的示例中，我们应该清楚地知道我们如何处理这个问题。我们将持有底层资源，即套接字。然后，我们将找到`!!!BEGIN!!!`缓冲区并让其通过。然后我们将开始存储所持有的数据。一旦我们到达`!!!END!!!`缓冲区，我们将推出数据块。

在这种情况下，我们持有相当多的数据，但它展示了我们如何处理帧。双工流将展示我们如何处理一个简单的协议。示例如下：

1.  导入`Readable`流并创建一个名为`ReadMessagePassStream`的类：

```js
import { Readable } from 'stream';

class ReadMessagePassStream extends Readable {
}
```

1.  添加一些私有变量来保存流的内部状态：

```js
// inside of the ReadMessagePassStream class
#socket = null;
#bufBegin = Buffer.from("!!!START!!!");
#bufEnd = Buffer.from("!!!END!!!");
#internalBuffer = [];
#size = 0;
```

1.  创建一个像之前那样的`#data`方法。我们现在将寻找之前设置的开始和结束帧缓冲区`#bufBegin`和`#bufEnd`：

```js
#data = function(chunk) {
    let i = -1 
    if((i = chunk.indexOf(this.#bufBegin)) !== -1) {
        const tempBuf = chunk.slice(i + this.#bufBegin.byteLength);
        this.#size += tempBuf.byteLength;            
        this.#internalBuffer.push(tempBuf);
    }
    else if((i = chunk.indexOf(this.#bufEnd)) !== -1) {
        const tempBuf = chunk.slice(0, i);
        this.#size += tempBuf.byteLength;
        this.#internalBuffer.push(tempBuf);
        const final = Buffer.concat(this.#internalBuffer);            
        this.#internalBuffer = [];
        if(!this.push(final)) { 
            this.#socket.pause();
        }
    } else {
        this.#size += chunk.byteLength;
        this.#internalBuffer.push(chunk);
    }
}
```

1.  创建类的构造函数以初始化我们的私有变量：

```js
constructor(options) {
    if( options.objectMode ) {
        options.objectMode = false //we don't want it on
    }
    super(options);
    if(!options.socket ) {
        throw "Need a socket to attach to!"
    }
    this.#socket = options.socket;
    this.#socket.on('data', this.#data.bind(this));
    this.#socket.on('end', () => this.push(null));
}
```

一个新的信息是`objectMode`属性，它可以传递到我们的流中。这允许我们的流读取对象而不是原始缓冲区。在我们的情况下，我们不希望发生这种情况；我们希望使用原始数据。

1.  为确保我们的流将启动，请添加`_read`方法：

```js
// inside the ReadMessagePassStream
_read(size) {
    this.#socket.resume();
}
```

有了这段代码，我们现在有了一种处理套接字的方法，而不必在主代码中监听数据事件；它现在包装在这个`Readable`流中。除此之外，我们现在有了将此流传输到另一个流的能力。以下是测试工具代码：

```js
import { createWriteStream } from 'fs';

const socket = createConnection(3333);
const write = createWriteStream('./output.txt');
const messageStream = new ReadMessagePassStream({ socket });
messageStream.pipe(write);
```

我们在本地主机的端口`3333`上托管了一个服务器。我们创建一个`write`流，并将任何数据从我们的`ReadMessagePassStream`传输到该文件。如果我们将其连接到测试工具中的服务器，我们会注意到创建了一个输出文件，其中只包含我们发送的数据，而不包含帧代码。

我们正在使用的帧技术并不总是有效。就像在`lorem`示例中展示的那样，我们的数据可能在任何时候被分块，我们的`!!!START!!!`和`!!!END!!!`可能会出现在其中一个块的边界上。如果发生这种情况，我们的流将失败。我们需要额外的代码来处理这些情况，但这些示例应该提供了实现流代码所需的所有必要思路。

接下来，我们将看一下`Writable`流接口。

# 构建可写流

`Writable`流是我们写入数据的流，它可以连接到`Readable`、`Duplex`或`Transform`流。我们可以使用这些流以分块的方式写入数据，以便消费流可以以分块而不是一次性处理数据。可写流的 API 与`Readable`流非常相似，除了可用的方法。

# 理解可写流接口

可写流为我们提供了几乎与`Readable`流相同的选项，因此我们不会深入讨论。相反，我们将看一下可用于我们的四种方法——一种我们*必须*实现的方法和其余我们*可以*实现的方法：

+   `_write`方法允许我们执行任何类型的转换或数据操作，并为我们提供使用回调的能力。这个回调是信号，表明写流能够接收更多数据。

虽然不是固有的真实情况，但它会从内部缓冲区中弹出数据。然而，对于我们的目的，最好将回调视为处理更多数据的一种方式。

我们可以利用这一点来包装一个更原始的流，并在主数据块之前或之后添加我们自己的数据。我们将在我们的`Readable`流的实际对应物中看到这一点。

+   `_final`方法允许我们在可写流关闭之前执行任何必要的操作。这可能是清理资源或发送我们可能一直保留的任何数据。除非我们保留了诸如文件描述符之类的东西，我们通常不会实现这个方法。

+   `_destroy`方法与`Readable`流相同，应该类似于`_final`方法，只是我们可能会在这个方法上出现错误。

+   `_writev`方法使我们能够同时处理多个块。如果我们对块有某种排序系统，或者我们不在乎块的顺序，我们可以实现这一点。虽然现在可能不明显，但我们将在实现双工流时实现这个方法。用例可能有些有限，但仍然可能有益。

# 实现可写流

以下`Writable`流实现展示了我们的帧方法以及我们如何使用它在我们的数据上放置`!!!START!!!`和`!!!END!!!`帧。虽然简单，但它展示了帧的强大和如何在原始流周围构建更复杂的流：

1.  从流模块导入`Writable`类，并为`WriteMessagePassStream`创建外壳。将其设置为此文件的默认导出：

```js
import { Writable } from 'stream';

export default class WriteMessagePassStream extends Writable {
}
```

1.  添加私有状态变量和构造函数。确保不允许`objectMode`通过，因为我们要处理原始数据：

```js
// inside the WriteMessagePassStream
#socket = null;
#writing = false;
constructor(options) {
  if( options.objectMode ) { 
        options.objectMode = false;
    }
    if(!options.socket ) {
        throw new Error("A socket is required to construct this 
         stream!");
    }
    super(options);
    this.#socket = options.socket;
}
```

1.  向我们的类添加`_write`方法。将如下解释：

```js
_write(chunk, encoding, callback) { 
    if(!this.#writing ) {
        this.#writing = true;
        this.#socket.write("!!!START!!!");
    }
    let i = -1;
    let prevI = 0;
    let numCount = 0;
    while((i = chunk.indexOf([0x00], i)) !== -1) {
        const buf = chunk.slice(prevI, i);
        this.#socket.write(buf);
        this.#socket.write("!!!END!!!");
        if( i !== chunk.byteLength - 1 ) {
            this.#socket.write("!!!START!!!");
        } else {
            return callback();
        }
        numCount += 1;
    }
    if(!numCount ) {
        this.#socket.write(chunk);
    }
    return callback();
}
```

有了这段代码，我们可以看到一些与我们处理可读端类似的地方。一些值得注意的例外包括以下项目：

+   我们实现`_write`方法。再次忽略这个函数的编码参数，但我们应该检查这一点，以防我们得到一个意料之外的编码。chunk 是正在写入的数据，回调是在我们完成对这个块的写入处理时调用的。

+   由于我们正在包装一个套接字，并且我们不希望在发送数据完成后关闭它，我们需要向我们的流发送某种停止信号。在我们的情况下，我们使用简单的`0x00`字节。在更健壮的实现中，我们会利用其他东西，但现在这应该可以工作。

+   无论如何，我们要么使用帧，要么直接写入底层套接字。

+   我们在处理完成后调用回调。在我们的情况下，如果我们设置了`writing`标志，这意味着我们仍然处于一个帧中，我们希望提前返回，否则，我们希望将我们的流置于写入模式，并写出`!!!START!!!`，然后是块。同样，如果我们从不使用回调，我们的流将被无限暂停。回调告诉内部机制从内部缓冲区中拉取更多数据供我们消耗。

有了这段代码，我们现在可以看一下测试工具和我们如何利用它来创建一个服务器并处理实现我们帧上下文的传入`Readable`流：

```js
import { createServer } from 'net'
import WrappedWritableStream from '../writable/main.js'
const server = createServer((con) => {
 console.log('client connected. sending test data');
 const wrapped = new WrappedWritableStream({ socket : con });
 for(let i = 0; i < 100000; i++) {
 wrapped.write(`data${i}\r\n`);
 }
 wrapped.write(Buffer.from([0x00]));
 wrapped.end();
 console.log('finished sending test data');
});
server.listen(3333);
```

我们创建一个服务器，并在本地端口`3333`上监听。每当我们接收到一个连接时，我们用我们的`Writable`流包装它。然后我们发送一堆测试数据，一旦完成，我们写出`0x00`信号告诉我们的流这个帧已经完成，然后我们调用`end`方法告诉我们已经完成了这个套接字。如果我们在第一次之后添加了另一个测试运行，我们可以看到我们的帧系统是如何工作的。让我们继续做这件事。在`wrapped.write(Buffer.from([0x00]))`之后添加以下代码：

```js
for(let i = 0; i < 100000; i++) {
    wrapped.write(`more_data${i}\r\n`);
}
wrapped.write(Buffer.from([0x00]));
```

如果我们达到流的`highWaterMark`，写入流将暂停，直到读取流开始从中消耗。

如果我们现在使用之前的`Readable`流运行测试工具，我们将看到我们正在处理所有这些数据并将其写入文件，而没有任何传输。有了这两种流实现，我们现在可以通过套接字传输数据，而不需要传输任何帧。我们现在可以使用这个系统来实现前一章中的数据传递系统。然而，我们将实现一个`Duplex`流，它将改进这个系统，并允许我们处理多个可写块，这将在下一节中看到。

# 实现双工流

双工流就是这样，可以双向工作。它将`Readable`和`Writable`流合并为一个单一的接口。有了这种类型的流，我们现在可以直接从套接字中导入到我们的自定义流中，而不是像以前那样包装流（尽管我们仍然将其实现为包装流）。

关于`Duplex`流没有更多可以谈论的了，除了一个让新手对流类型感到困惑的事实。有两个单独的缓冲区：一个用于`Readable`，一个用于`Writable`。我们需要确保将它们视为单独的实例。这意味着我们在`_read`方法中使用的变量，在`_write`和`_writev`方法的实现中不应该使用，否则我们可能会遇到严重的错误。

如前所述，以下代码实现了一个`Duplex`流，以及一个计数机制，这样我们就可以利用`_writev`方法。正如在*理解可写流接口*部分所述，`_writev`方法允许我们一次处理多个数据块：

1.  从`stream`模块导入`Duplex`类，并为我们的`MessageTranslator`类添加外壳。导出这个类：

```js
import { Duplex } from 'stream';

export default class MessageTranslator extends Duplex {
}
```

1.  添加所有内部状态变量。每个变量将在接下来的部分中解释：

```js
// inside the MessageTranslator class
#socket = null;
#internalWriteBuf = new Map();
#internalReadHoldBuf = [];
#internalPacketNum = 0;
#readSize = 0;
#writeCounter = 0;
```

1.  为我们的类添加构造函数。我们将在这个构造函数中处理我们的`#socket`的数据事件，而不是像以前那样创建另一个方法：

```js
// inside the MessageTranslator class
constructor(opts) {
    if(!opts.socket ) {
        throw new Error("MessageTranslator stream needs a 
         socket!");
    }
    super(opts);
    this.#socket = opts.socket;
    // we are assuming a single message for each chunk
    this.#socket.on('data', (chunk) => {
        if(!this.#readSize ) {
            this.#internalPacketNum = chunk.readInt32BE();
            this.#readSize = chunk.readInt32BE(4);
            this.#internalReadHoldBuf.push(chunk.slice(8));
            this.#readSize -= chunk.byteLength - 8
        } else {
            this.#internalReadHoldBuf.push(chunk);
            this.#readSize -= chunk.byteLength;
        }
        // reached end of message
        if(!this.#readSize ) {
            this.push(Buffer.concat(this.#internalReadHoldBuf));
            this.#internalReadHoldBuf = [];
        }
    });
}
```

我们将自动假设每个块中有一条消息。这样处理会更容易。当我们获取数据时，我们将读取数据包编号，这应该是数据的前四个字节。然后我们读取消息的大小，这是接下来的`4`个字节数据。最后，我们将剩余的数据推入我们的内部缓冲区。一旦我们完成读取整个消息，我们将把所有内部块放在一起并推送它们出去。最后，我们将重置我们的内部缓冲区。

1.  向我们的类添加`_writev`和`_write`方法。记住，`_writev`方法用于多个数据块，所以我们需要循环遍历它们并将每个写出去：

```js
// inside the MessageTranslator class
_writev(chunks, cb) { 
    for(const chunk of chunks) {
        this.#processChunkHelper(chunk); //shown next
    }
    this.#writeHelper(cb); //shown next
}
_write(chunk, encoding, cb) {
    this.#processChunkHelper(chunk); //shown next
    this.#writeHelper(cb); //shown next
}
```

1.  添加处理块和实际写出的辅助方法。我们将使用数字`-1`作为`4`字节消息，表示我们已经完成了这条消息。

```js
// inside the MessageTranslator class
#processChunkHelper = function(chunk) {
    if(chunk.readInt32BE() === -1) { 
        this.#internalWriteBuf.get(this.#writeCounter).done = true;
        this.#writeCounter += 1;
        this.#internalWriteBuf.set(this.#writeCounter, {buf : [], 
         done : false});
    } else {
        if(!this.#internalWriteBuf.has(this.#writeCounter)) {
            this.#internalWriteBuf.set(this.#writeCounter, {buf : 
             [], done : false}); }
            this.#internalWriteBuf.get(this.#writeCounter)
             .buf.push(chunk);
        }
    }
}
#writeHelper = function(cb) {
    const writeOut = [];
    for(const [key, val] of this.#internalWriteBuf) { 
        if( val.done ) {
            const cBuf = Buffer.allocUnsafe(4);
            const valBuf = Buffer.concat(val.buf);
            const sizeBuf = Buffer.allocUnsafe(4);
            cBuf.writeInt32BE(valBuf.readInt32BE());
            sizeBuf.writeInt32BE(valBuf.byteLength - 4);
            writeOut.push(Buffer.concat([cBuf, sizeBuf, 
             valBuf.slice(4)]));
            val.buf = [];
        }
    }
    if( writeOut.length ) {
        this.#socket.write(Buffer.concat(writeOut));
    }
    cb();
}
```

我们的`#processChunkHelper`方法检查我们是否达到了神奇的`-1` `4`字节消息，表示我们已经完成了消息的写入。如果没有，我们将继续向我们的内部缓冲区（数组）添加。一旦我们到达末尾，我们将把所有数据放在一起，然后转移到下一个数据包。

我们的`#writeHelper`方法将循环遍历所有这些数据包，并检查它们是否有任何一个已经完成。如果有，它将获取数据包编号、缓冲区的大小、数据本身，并将它们全部连接在一起。一旦完成这些操作，它将重置内部缓冲区，以确保我们不会泄漏内存。我们将把所有这些数据写入套接字，然后调用回调函数表示我们已经完成写入。

1.  通过实现我们之前的`_read`方法来完成`Duplex`流。`_final`方法应该只是调用回调函数，因为没有剩余的处理：

```js
// inside the MessageTranslator class
_read() {
    this.#socket.resume();
}
_final(cb) {
    cb(); // nothing to do since it all should be consumed at this 
          // point
}
```

当顺序不重要且我们只是处理数据并可能将其转换为其他形式时，应该真正使用`_writev`。这可能是一个哈希算法或类似的东西。在几乎所有情况下，应该使用`_write`方法。

虽然这个实现有一些缺陷（其中一个是如果我们达到`-1`数字时没有寻找可能的其他数据包），但它展示了我们如何构建一个`Duplex`流，以及处理消息的另一种方式。不建议自己设计在套接字之间传输数据的方案（正如我们将在下一章中看到的），但如果有一个新的规范出来，我们总是可以利用`Duplex`套接字来编写它。

如果我们用我们的测试工具测试这个实现，我们应该得到一个名为`output.txt`的文件，其中包含了双工加上数字消息被写入了 10 万次，以及一个尾随的换行符。再次强调，`Duplex`流只是一个单独的`Readable`和`Writable`流组合在一起，应该在实现数据传输协议时使用。

我们将要看的最后一个流是`Transform`流。

# 实现 Transform 流

在这四个流中，这可能是最有用的，也可能是最常用的流之一。`Transform`流连接了流的可读和可写部分，并允许我们操纵流中传输的数据。这听起来可能类似于`Duplex`。嗯，`Transform`流是`Duplex`流的一种特殊类型！

`Transform`流的内置实现包括`zlib`模块中实现的任何流。基本思想是我们不仅仅是试图将信息从一端传递到另一端；我们试图操纵这些数据并将其转换为其他形式。这就是`zlib`流给我们的。它们压缩和解压数据。`Transform`流将数据转换为另一种形式。这也意味着我们可以使一个转换流成为单向转换；从转换流输出的任何东西都无法被撤销。我们将在这里创建一个这样的`Transform`流，具体地创建一个字符串的哈希。

首先，让我们来看一下`Transform`流的接口。

# 理解 Transform 流接口

我们可以访问两种方法，几乎无论如何我们都想要实现。其中一个让我们可以访问底层数据块，并允许我们对其进行转换。我们使用`_transform`方法来实现这一点。它接受三个参数：我们正在处理的数据块，编码和一个回调，让底层系统知道我们已经准备好处理更多信息。

与`Writable`流的`_write`回调不同的是，回调函数的一个特殊之处是我们可以向其传递数据，以在`Transform`流的可读端发出数据，或者我们可以不传递任何数据，以表示我们想要处理更多数据。这使我们只在需要时发送数据事件，而不是几乎总是需要传递它们。

另一种方法是`_flush`方法。这允许我们完成可能仍在持有的任何数据的处理。或者，它将允许我们在流中发送的所有数据都输出一次。这就是我们将用字符串哈希函数实现的功能。

# 实现 Transform 流

我们的`Transform`流将接收字符串数据并继续运行哈希算法。一旦完成，它将输出计算出的最终哈希值。哈希函数是一种我们将某种形式的输入转换为唯一数据的函数。这个唯一的数据（在我们的例子中是一个数字）不应该容易发生碰撞。碰撞是两个不同值可能得到相同哈希值的概念。在我们的情况下，我们将字符串转换为 JavaScript 中的 32 位整数，因此我们很少发生碰撞，但并非不可能。

以下是示例：

```js
// implemented in stream form from 
// https://stackoverflow.com/questions/7616461/generate-a-hash-from-string-in-javascript
export default class StreamHashCreator extends Transform {
    #currHash = 0; 
    constructor(options={}) {
        if( options.objectMode ) {
            throw new Error("This stream does not support object mode!");
        }
        options.decodeStrings = true;
        super(options);
    }
    _transform(chunk, encoding, callback) {
        if( Buffer.isBuffer(chunk) ) { 
            const str = chunk.toString('utf8');
            for(let i = 0; i < str.length; i++) {
                const char = str.charCodeAt(i);
                this.#currHash = ((this.#currHash << 5) - this.#currHash ) 
                 + char;
                this.#currHash |= 0;
            }
        }
        callback(); 
    }
    _flush(callback) {
        const buf = Buffer.alloc(4);
        buf.writeInt32BE(this.#currHash);
        this.push(buf); 
        callback(null);
    }
}
```

前一个流的每个函数都在下面解释：

1.  我们需要持久化的唯一一件事是直到流被销毁的当前哈希码。这将允许哈希函数跟踪我们已经传递给它的内容，并在每次写入后处理数据。

1.  我们在这里进行检查，看我们收到的块是否是一个`Buffer`。由于我们确保打开了`decodeStrings`选项，这意味着我们应该总是得到缓冲区，但检查仍然有帮助。

1.  虽然哈希函数的内容可以在提供的 URL 中看到，但我们需要担心的唯一重要事情是，我们要调用我们的回调，就像我们在实现`Writable`流时所做的那样。

1.  一旦我们准备生成数据，我们就使用`push`方法，就像我们在`Readable`流中所做的那样。记住，`Transform`流只是允许我们操纵输入数据并将其转换为输出的特殊`Duplex`流。我们还可以将代码的最后两行更改为`callback(null, buf)`；这只是我们之前看到的简写。

现在，如果我们对前面的代码运行一些测试用例，我们会发现每个唯一字符串输入都会得到一个唯一的哈希码，但当我们输入完全相同的内容时，我们会得到相同的哈希码。这意味着我们的哈希函数很好，我们可以将其连接到流应用程序中。

# 使用流生成器

到目前为止，我们所看到的一切都展示了我们如何利用 Node.js 中的所有内置系统来创建流应用程序。然而，对于那些一直在按顺序阅读本书的人来说，我们已经讨论了生成器。那些一直在思考它们的人会注意到流和生成器之间有很强的相关性。事实上就是这样！我们可以利用生成器来连接到流 API。

有了这个概念，我们可以构建既可以在浏览器中工作又可以在 Node.js 中工作的生成器，而不需要太多的开销。我们甚至在第六章中看到了如何使用 Fetch API 获取底层流。现在，我们可以编写一个可以与这两个子系统一起工作的生成器。

现在，让我们只看一个`async`生成器的示例，以及我们如何将它们连接到 Node.js 流系统中。示例将是看看我们如何将生成器作为`Readable`流的输入：

1.  我们将建立一个`Readable`流来读取英语字母表的 26 个小写字符。我们可以通过编写以下生成器来轻松实现这一点：

```js
function* handleData() {
    let _char = 97;
    while(_char < 123 ) { //char code of 'z'
        yield String.fromCharCode(_char++);
    }
}
```

1.  当字符代码低于`123`时，我们继续发送数据。然后我们可以将其包装在`Readable`流中，如下所示：

```js
const readable = Readable.from(handleData());
readable.on('data', (chunk) => {
    console.log(chunk);
});
```

如果我们现在运行这段代码，我们会看到控制台中出现字符*a*到*z*。`Readable`流知道它已经结束，因为生成器生成了一个具有两个键的对象。`value`字段给出了`yield`表达式的值，`done`告诉我们生成器是否已经完成运行。

这让可读接口知道何时发送`data`事件（通过我们产生一个值）以及何时关闭流（通过将`done`键设置为`true`）。我们还可以将可读系统的输出管道到可写系统的输出，以链接整个过程。这可以很容易地通过以下代码看到：

```js
(async() => {
    const readable2 = Readable.from(grabData());
    const tempFile = createWriteStream('./temp.txt');
    readable2.pipe(tempFile);
    await once(tempFile, 'finish');
    console.log('all done');
})();
```

通过生成器和`async`/`await`实现流可能看起来是一个好主意，但只有在我们试图将一个已经是`async`/`await`的代码片段与流结合时，我们才应该利用它。始终要追求可读性；利用生成器或`async`/`await`方法很可能会导致代码难以阅读。

通过前面的例子，我们已经将生成器的可读性与利用管道机制发送到文件相结合。随着`async`/`await`和生成器成为 JavaScript 语言中的构造，流很快就会成为一个一流的概念。

# 总结

流是编写高性能 Node.js 代码的支柱之一。它允许我们不阻塞主线程，同时仍然能够处理数据。流 API 允许我们为我们的目的编写不同类型的流。虽然这些流大多数将是转换流的形式，但看到我们如何实现其他三种流也是很好的。

我们将在下一章中看到的最后一个主题是数据格式。处理除了 JSON 之外的不同数据格式将使我们能够与许多大数据提供商进行接口，并能够处理他们喜欢使用的数据格式。我们将看到他们如何利用流来实现所有的格式规范。
