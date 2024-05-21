# 第五章：切换上下文-没有 DOM，不同的 Vanilla

当我们把注意力从浏览器转向其他地方时，我们将进入大多数后端程序员熟悉的环境。Node.js 为我们提供了一个熟悉的语言，即 JavaScript，可以在系统环境中使用。虽然 Node.js 以用于编写服务器的语言而闻名，但它可以用于大多数其他语言所用的大多数功能。例如，如果我们想创建一个**命令行界面**（**CLI**）工具，我们就有能力做到。

Node.js 还为我们提供了类似于浏览器中看到的编程环境。我们得到了一个允许我们进行异步**输入和输出**（**I/O**）的事件循环。这是通过 libuv 库实现的。在本章的后面，我们将解释这个库以及它如何帮助我们提供我们习惯的常见事件循环。首先，我们将看看如何启动和运行 Node.js，以及编写一些简单的程序。

在本章中，我们将涵盖以下主题：

+   获取 Node.js

+   理解无 DOM 的世界

+   调试和检查代码

让我们开始吧。

# 技术要求

本章需要以下技术要求：

+   像 VS Code 这样的编辑器或 IDE

+   本章的代码可以在[`github.com/PacktPublishing/Hands-On-High-Performance-Web-Development-with-JavaScript/tree/master/Chapter05`](https://github.com/PacktPublishing/Hands-On-High-Performance-Web-Development-with-JavaScript/tree/master/Chapter05)找到。

# 获取 Node.js

之前的章节要求有一个 Node.js 运行时。在本章中，我们将看看如何在我们的系统上安装它。如果我们前往[`Node.js.org/en/`](https://nodejs.org/en/)，我们将能够下载**长期支持**（**LTS**）版本或当前版本。对于本书来说，建议获取当前版本，因为模块支持更好。

对于 Windows，我们只需要下载并运行可执行文件。对于 OS X 和 Linux，这也应该很简单。特别是对于 Linux 用户，可能在特定发行版的存储库中有一个版本，但这个版本可能很旧，或者与 LTS 版本一致。记住：我们希望运行最新版本的 Node.js。

一旦我们安装好了，我们应该能够从任何命令行调用`node`命令（Linux 用户可能需要调用`Node.js`命令，因为一些存储库已经在其存储库中包含了一个 node 包）。一旦调用，我们应该会看到一个**读取评估打印循环**（**REPL**）工具。这使我们能够在实际将代码写入文件之前测试一些代码。运行以下代码片段：

```js
1 + 2 //3
typeof("this") //'string'
const x = 'what' //undefined
x //'what'
x = 'this' //TypeError: Assignment to a constant variable
const fun = function() { console.log(x); } //undefined
fun() //'what' then undefined
fun = 'something' //TypeError: Assignment to a constant variable
```

从这些例子中，很明显我们正在一个类似于我们在浏览器中使用的环境中工作。我们可以访问大多数我们在浏览器中拥有的数据操作和功能概念。

我们无法访问许多特定于浏览器的库/ API，比如 DOM API。我们也无法访问任何浏览器外部资源访问库，比如`Fetch`或`XMLHttpRequest`。我们将稍后讨论它们的较低级版本，但应该注意的是，在某些方面，它并不像调用 fetch API 那样简单。

继续玩一下 REPL 系统。如果我们想退出，只需要在 Windows 上按两次*Ctrl* + *C*（Linux 应该是一样的；对于 OS X，我们需要使用*command* + *C*）。现在，要运行一个脚本，我们只需要把一些代码放在一个 JavaScript 文件中，然后调用**`node <filename>`**。这应该在立即模式下运行我们的脚本。这可以在以下的`example.js`文件中看到：

```js
const x = 'what';
console.log('this is the value of x', x); //'this is the value of x what'
const obj = {
    what : 'that',
    huh : 'this',
    eh : 'yeah'
}
console.log(obj); // { what : 'that', huh : 'this', eh : 'yeah' }
```

为了访问 Node.js 给我们的各种内置库，我们可以利用两种不同的方法。首先，我们可以使用旧的`require`系统。以下脚本展示了这种能力：

```js
const os = require('os');

console.log(os.arch()); //prints out x64 on 64-bit OS
console.log(os.cpus()); //prints out information on our CPU
```

这是当前引入内置/用户构建模块的方式。这是 Node 团队决定的风格，因为没有常见的引入模块的方式。我们有 RequireJS 或 CommonJS 等系统，Node.js 决定采用 CommonJS 风格引入模块。然而，正如我们所了解的，也有一种标准化的方式将模块引入浏览器。对于 Node.js 平台也是如此。

模块系统目前处于实验阶段，但如果需要，可以使用诸如 RollupJS 之类的系统将代码更改为通用识别的系统版本，例如**通用模块依赖**（**UDM**）系统。

这个系统看起来应该很熟悉。以下脚本显示了先前的示例，但是在模块导入系统中：

```js
import os from 'os';

console.log(os.arch());
console.log(os.cpus());
```

我们还需要一个`package.json`文件，在其清单中有`"type" : "module"`。

# package.json 文件概述

`package.json`文件包含了我们正在构建的包的所有信息。它甚至让我们能够将其与我们的版本控制系统联系起来，甚至可以将其与我们的构建系统联系起来。让我们现在来看一下。

首先，`package.json`文件应该填写以下字段：

+   `name`：这是包的名称。

+   `version`：这是我们软件包的当前版本。

+   `type`：这应该是`module`或`commonjs`。这将允许我们区分传统系统和新的 ECMAScript 模块系统。

+   `license`：这是我们想要许可我们的模块的方式。大多数情况下，只需放置 MIT 许可证。然而，如果我们想要更严格地限制它，我们可以随时使用 GPL 或 LGPL 许可证。

+   `author`：这是一个带有`name`、`email`和`url`字段的对象。这为软件提供了归属，并帮助人们知道是谁构建了它。

+   `main`：这是模块的主入口点。这将允许其他人使用我们的模块并要求/导入它。它还将让系统知道在哪里寻找我们模块的起始点。

还有许多其他可以使用的字段，如下：

+   `man`：这允许`man`命令找到我们希望为我们的文档提供的文件。

+   `description`：这允许我们提供关于我们的模块及其功能的更多信息。如果描述超过两到三句，建议附带一个`README`文件。

+   `repository`：这允许其他人找到存储库并为其做出贡献或提交错误报告/功能请求。

+   `config`：这是一个对象，可以被我们在`package.json`文件的脚本部分定义的脚本使用。脚本将很快会详细讨论。

+   `dependencies`：这是我们的模块依赖的模块列表。这可以是来自公共`npm`注册表、私有存储库、Git 存储库、tarballs，甚至是本地文件路径用于本地开发。

+   `devDependencies`：这是需要用于此软件包开发的依赖列表。

+   `peerDependencies`：这是我们的包可能需要的依赖列表，如果有人使用系统的一部分。这允许我们的用户下载核心系统，如果他们想要使用其他部分，他们可以下载这些其他子系统需要的对等依赖。

+   `OS`：这是我们运行的操作系统列表。这也可以是其否定版本，比如`!darwin`，意味着这个系统将在除 OS X 之外的所有操作系统上运行。

+   `engines`：我们运行的 Node.js 版本。当我们使用最近版本引入的功能（例如 ECMAScript 模块）时，我们将要使用这个。如果我们使用已被弃用的模块并希望将 Node.js 版本锁定到旧版本，我们也可能想要使用这个功能。

`package.json`文件中还有一些其他字段，但这些是最常见的。

我们想要查看的`package.json`文件的一个特定部分是脚本部分。如果我们去查看`npm`的网站关于脚本部分的信息，它陈述了以下内容：

`scripts`属性是一个包含在包的生命周期中的各个时间点运行的脚本命令的字典。键是生命周期事件，值是在该点运行的命令。

如果我们进入更多细节部分，我们将看到我们可以使用生命周期钩子，以便我们可以通过捆绑和分发过程运行各种脚本。

值得注意的是，这些信息特定于**Node Package Manager**（**npm**）。在学习 Node.js 的过程中，我们会经常遇到`npm`，因此学习 Node.js 也意味着学习`npm`。

我们感兴趣的一些具体点是打包生命周期的**prepare**和**install**部分。让我们看看这些部分涵盖了什么：

+   **Prepare**将在将包打包成 tarball 并发布到远程存储库之前运行脚本。这是运行编译器和捆绑器以准备部署我们的包的好方法。

+   **Install**将在安装完包后运行脚本。当我们拉取一个包并想要运行诸如`node-gyp`之类的东西，或者我们的包可能需要的特定于操作系统的东西时，这非常有用。

`scripts`部分的另一个好处是，我们可以在这里放任意字符串并运行`npm run <script>`。无论我们决定使用什么值，都将在运行命令时进行评估。让我们将以下内容添加到我们的`package.json`文件中：

```js
"config" : {
    "port" : 8080,
    "name" : "example",
    "testdata" : {
        "thing" : 5,
        "other" : 10
    }
},
"scripts" : {
    "example-script" : "node main.js"
}
```

这将使我们能够获取配置数据。除此之外，我们还添加了一个可以通过`npm run example-script`命令运行的脚本。如果我们创建一个`main.js`文件并向其中添加以下字段，我们应该会得到以下输出：

```js
console.log(process.env.npm_package_config_port); //8080
console.log(process.env.npm_package_config_name); //'example'
console.log(process.env.npm_package_config_testdata); //undefined
```

这表明我们可以在配置中放入原始值，但我们不能尝试访问复杂对象。我们可以这样做来获取`testdata`对象的属性：

```js
console.log(process.env.npm_package_config_testdata_thing) //5
console.log(process.env.npm_package_config_testdata_other) //10
```

现在我们对 Node.js 和`npm`生态系统有了一些了解，让我们来看看 Node.js 是如何组合在一起的，以及我们将在接下来的章节中使用的一些关键模块。

# 理解无 DOM 世界

正如我们在介绍中所述，Node.js 的出现是基于这样一个想法：如果我们在浏览器中编写代码，那么我们应该能够在服务器上运行它。在这里，我们有一个语言适用于两种情境，无论我们在哪个部分工作，都不必切换上下文。

Node.js 可以通过两个库的混合方式运行。这些库是 V8，我们应该已经熟悉了，以及 libuv，我们目前还不熟悉。libuv 库为我们提供了异步 I/O。每个操作系统都有不同的处理方式，所以 libuv 为我们提供了一个很好的 C 包装器来处理所有这些实例。

libuv 库将 I/O 请求排队到请求堆栈上。然后，它将它们分配给一定数量的线程（Node.js 默认使用四个）。一旦这些线程的响应返回，libuv 将它们放在响应堆栈上，并通知 V8 响应已准备好被消耗。一旦 V8 注意到这个通知，它将从中取出值并将其用于对我们发出的请求的响应。这就是 Node.js 运行时如何能够具有异步 I/O 并仍然保持单线程执行的方式（至少对用户来说是这样看的）。

有了这些基本理解，我们应该能够开始编写一些处理各种 I/O 操作并利用使 Node.js 特殊的想法之一的基本脚本：流系统。

# 流的第一印象

正如我们在 DOM 中看到的那样，流给了我们控制数据流的能力，并且能够以创建非阻塞系统的方式处理数据。通过创建一个简单的流，我们可以看到这一点。让我们继续利用 Node.js 提供的内置流之一，`readFileStream`。让我们编写以下脚本：

```js
import fs from 'fs';
import { PassThrough } from 'stream'

const str = fs.createReadStream('./example.txt');
const pt = new PassThrough();
str.pipe(pt);
pt.on('data', (chunk) => {
    console.log(chunk);
});
```

在这里，我们导入了`fs`库和`stream`库中的`PassThrough`流。然后，我们为`example.txt`文件创建了一个读取流，以及一个`PassThrough`流。

`PassThrough`流允许我们处理数据，而无需显式创建流。我们读取数据并将其传输到我们的`PassThrough`流。

从这里，我们能够获得数据事件的处理，这给了我们一块数据。除此之外，我们还确保在`pipe`方法之后放置了我们的数据事件监听器。通过这样做，我们确保在附加监听器之前没有`data`事件运行。

让我们创建以下`example.txt`文件：

```js
This is some data
it should be processed by our system
it should be coming in chunks that our system can handle
most likely this will all come in one chunk
```

现在，如果我们运行`node --experimental-modules read_file_stream.js`命令，我们将看到它打印出一个`Buffer`。除非我们明确将其设置为对象模式，否则所有数据处理都是以二进制块包装在`Buffer`对象中的。如果我们将控制台日志命令更改为打印以下内容，我们应该得到纯文本输出：

```js
console.log(chunk.toString('utf8'));
```

让我们创建一个程序，统计文本中单词`the`的使用次数。我们可以使用我们的`PassThrough`流来做到这一点，如下所示：

```js
import fs from 'fs';
import { PassThrough } from 'stream';

let numberOfThe = 0;
const chars = Buffer.from('the');
let currPos = 0;
const str = fs.createReadStream('./example.txt');
const pt = new PassThrough();
str.pipe(pt);
pt.on('data', (chunk) => {
    for(let i = 0; i < chunk.byteLength; i++) {
        const char = chunk[i];
        if( char === chars[currPos] ) {
            if( currPos === chars.byteLength - 1 ) // we are at the end so 
             reset
                numberOfThe += 1;
                currPos = 0;
            } else {
                currPos += 1;
            }
        } else {
            currPos += 1;
        }
    }
});
pt.on('end', () => {
    console.log('the number of THE in the text is: ', numberOfThe);
});
```

我们需要记录单词`the`出现的次数。我们还将创建一个`the`字符串的字节缓冲区。我们还需要跟踪我们当前的位置。通过这样做，每当我们获得数据时，我们可以运行并测试每个字节。如果字节与我们持有的当前位置匹配，那么我们需要进行另一个检查。如果它等于单词`the`的字符字节计数，那么我们更新`the`的数量并重置我们的当前位置。否则，我们将当前位置设置为下一个索引。如果我们没有找到匹配，我们需要重置当前位置；否则，我们将得到字符*t*、*h*和*e*的任意组合。

这是一个有趣的例子，展示了如何利用`PassThrough`流，但让我们继续创建我们自己的写`Transform`流。我们将应用与之前相同的操作，但我们将构建一个自定义流。正如文档中所述，我们必须编写`_transform`函数，并且可以选择实现`_flush`函数。我们将实现`_transform`和`_flush`函数。我们还将利用新的类语法，而不是利用旧的基于原型的系统。在构建我们自己的自定义流时要记住的一件事是，在流中做任何其他事情之前运行`super(options)`方法。这将允许用户传递各种流选项，而无需我们做任何事情。

考虑到所有这些，我们应该得到类似以下的东西：

```js
import { Transform } from 'stream';

class GetThe extends Transform {
    #currPos = 0;
    #numberOfThe = 0;

    static chars = Buffer.from('the');
    constructor(options) {
        super(options);
    }
    _transform(chunk, encoding, callback) {
        for(let i = 0; i < chunk.byteLength; i++) {
            const char = chunk[i];
            if( char === GetThe.chars[this.#currPos]) {
                if( this.#currPos === GetThe.chars.byteLength - 1 ) {
                    this.#numberOfThe += 1;
                    this.#currPos = 0;
                } else {
                    this.#currPos += 1;
                }
            } else {
                this.#currPos = 0;
            }
        }
        callback();
    }
    _flush(callback) {
        callback(null, this.#numberOfThe.toString());
    }
}

export default GetThe;

```

首先，我们从`stream`基础库中导入`Transform`流。我们扩展它并创建一些私有变量，即`the`缓冲区中的当前位置和流中`the`的当前计数。我们还为我们要进行比较的缓冲区创建了一个静态变量。然后，我们有我们的构造函数。这是我们将选项传递给`Transform`流构造函数的地方。

接下来，我们以与我们在`PassThrough`流的`data`事件上实现的方式实现`_transform`方法。唯一的新部分应该是在最后调用回调函数。这让我们的流知道我们已经准备好处理更多数据。如果我们需要出错，我们可以将其作为第一个参数传递。我们还可以传递第二个参数，就像在`_flush`函数中所示的那样。这允许我们将处理过的数据传递给可能正在监听的人。在我们的情况下，我们只想传递我们在文本中找到的`the`的数量。我们还可以只传递`Buffer`、`String`或`Uint8Array`，所以我们决定传递我们数字的字符串版本（我们本可以使用`Buffer`，这可能是更好的选择）。最后，我们从我们的模块中导出这个。

在我们的`read_file_stream`文件中，我们将使用以下命令导入此模块：

```js
import GetThe from './custom_transform.js';
```

然后，我们可以使用以下代码：

```js
const gt = new GetThe();
gt.on('data', (data) => {
    console.log('the number of THE produced by the custom stream is: ', 
     data.toString('utf8'));
});
const str2 = fs.createReadStream('./example.txt');
str2.pipe(gt);
```

通过这样做，我们将所有这些逻辑封装到一个单独的模块和可重用的流中，而不仅仅是在`PassThrough`的`data`事件中这样做。我们还可以将我们的流实现链接到另一个流（在这种情况下，除非我们要将其传递给套接字，否则可能没有意义）。

这是流接口的简短介绍，并概述了我们将在后面章节中详细讨论的内容。接下来，我们将看一下 Node.js 附带的一些模块以及它们如何帮助我们编写服务器应用程序。

# 模块的高级视图

有三个 I/O 模块允许我们的应用程序与文件系统和外部世界进行交互。它们分别是：

+   `fs`

+   `net`

+   `http`

这三个模块很可能是用户在开发 Node.js 应用程序时将使用的主要模块。让我们分别来看看它们。

# fs 模块

首先，让我们创建一个访问文件系统、打开文件、添加一些文本、关闭文件，然后再追加一些文本的基本示例。这看起来类似于以下内容：

```js
import { promises } from 'fs';

(async() => {
    await promises.writeFile('example2.txt', "Here is some text\n");
    const fd = await promises.open('example2.txt', 'a');
    await fd.appendFile("Here is some more text\n");
    await fd.close();
    console.log(await promises.readFile('example2.txt', 'utf8'));
})();
```

首先，我们正在获取基于 Promise 的库版本。大多数内置模块都有基于 Promise 的版本，这可以导致看起来很好的代码，特别是与回调系统相比。接下来，我们写入一个文件并给它一些文本。`writeFile`方法允许我们写入文件并在文件不存在时创建文件。之后，我们打开我们文件的`FileHandle`。

Node.js 采用了 POSIX 风格的 I/O。这意味着一切都像文件一样对待。在这种情况下，一切都被分配了一个**文件描述符**（**fd**）。这对我们来说看起来像是 C++等语言中的一个数字。之后，我们可以将这个数字传递给我们可用的各种文件函数。在 Promises API 中，Node.js 决定切换到`FileHandle`对象，这是我们得到的而不是这个文件描述符。这导致了更清晰的代码，并且有时需要在系统上提供一层抽象。

我们可以看到作为第二个参数的`a`表示我们将如何使用文件。在这种情况下，我们将追加到文件中。如果我们用`r`打开它，这意味着我们要从中读取，而如果我们用`w`打开它，这意味着我们要覆盖已经存在的内容。

了解 Unix 系统可以帮助我们更好地理解 Node.js 的工作原理，以及所有这些与我们试图编写的程序之间的对应关系。

然后，我们向文件追加一些文本并关闭它。最后，我们在控制台记录文件中的内容，并声明我们要以 UTF-8 文本而不是二进制形式读取它。

与文件系统相关的 API 还有很多，建议查看 Promise 文档以了解我们有哪些能力，但它们都归结为我们可以访问文件系统，并能够读取/写入/追加到各种文件和目录。现在，让我们继续讨论`net`模块。

# 网络模块

`net`模块为我们提供了对底层套接字系统甚至本地**进程间通信**（**IPC**）方案的访问权限。IPC 方案是允许我们在进程之间进行通信的通信策略。进程不共享内存，这意味着我们必须通过其他方式进行通信。在 Node.js 中，这通常意味着三种不同的策略，它们都取决于我们希望系统有多快速和紧密耦合。这三种策略如下：

+   无名管道

+   命名管道/本地域套接字

+   TCP/UDP 套接字

首先，我们有无名管道。这些是单向通信系统，不会出现在文件系统中，并且在`parent`和`child`进程之间共享。这意味着`parent`进程会生成一个`child`进程，并且`parent`会将管道一端的*位置*传递给`child`。通过这样做，它们可以通过这个通道进行通信。一个例子如下：

```js
import { fork } from 'child_process';

const child = fork('child.js');
child.on('message', (msg) => {
    switch(msg) {
        case 'CONNECT': {
            console.log('our child is connected to us. Tell it to dispose 
             of itself');
            child.send('DISCONNECT');
            break;
        }
        case 'DISCONNECT': { 
            console.log('our child is disposing of itself. Time for us to 
             do the same');
            process.exit();
            break;
        }
    }
});
```

我们的`child`文件将如下所示：

```js
process.on('message', (msg) => {
    switch(msg) {
        case 'DISCONNECT': {
            process.exit();
            break;
        }
    }
});
process.send('CONNECT');
```

我们从`child_process`模块中获取 fork 方法（这允许我们生成新的进程）。然后，我们从`child` JavaScript 文件中 fork 一个新的`child`，并获得对该`child`进程的处理程序。作为 fork 过程的一部分，Node.js 会自动为我们创建一个无名管道，以便我们可以在两个进程之间进行通信。然后，我们监听`child`进程上的事件，并根据接收到的消息执行各种操作。

在`child`端，我们可以自动监听来自生成我们的进程的事件，并且可以通过我们的进程接口发送消息（这在每个启动的 Node.js 文件中都是全局的）。如下面的代码所示，我们能够在两个独立的进程之间进行通信。如果我们想要真正看到这一点，我们需要在`parent`进程中添加一个超时，以便在`15`秒内不发送`DISCONNECT`消息，就像这样：

```js
setTimeout(() => {
    child.send('DISCONNECT');
}, 15000);
```

现在，如果我们打开任务管理器，我们会看到启动了两个 Node.js 进程。其中一个是`parent`，另一个是`child`。我们正在通过一个无名管道进行通信，因此它们被认为是紧密耦合的，因为它们是唯一共享它的进程。这对于我们希望有`parent`/`child`关系的系统非常有用，并且不希望以不同的方式生成它们。

与在两个进程之间创建紧密链接不同，我们可以使用称为命名管道的东西（在 OS X 和 Linux 上称为 Unix 域套接字）。它的工作方式类似于无名管道，但我们能够连接两个不相关的进程。为了实现这种类型的连接，我们可以利用`net`模块。它提供了一个低级 API，可以用来创建、连接和监听这些连接。我们还会得到一个低级套接字连接，因此它的行为类似于`http(s)`模块。

要建立连接，我们可以这样做：

```js
import net from 'net';
import path from 'path';
import os from 'os';

const pipeName = (os.platform() === 'win32' ?
    path.join('\\\\?\\pipe', process.cwd(), 'temp') :
    path.join(process.cwd(), 'temp');
const server = net.createServer().listen(pipeName);
server.on('connection', (socket) => {
    console.log('a socket has joined the party!');
    socket.write('DISCONNECT');
    socket.on('close', () => {
        console.log('socket has been closed!');
    });
});
```

在这里，我们导入了`net`、`path`和`os`模块。`path`模块帮助我们创建和解析文件系统路径，而无需为特定的操作系统编写路径表达式。正如我们之前看到的，`os`模块可以为我们提供有关当前所在的操作系统的信息。在创建管道名称时，Windows 需要在`\\?\pipe\<something>`。在其他操作系统上，它可以只是一个常规路径。还有一点需要注意的是，除了 Windows 之外的任何其他操作系统在我们使用完管道后都不会清理它。这意味着我们需要确保在退出程序之前删除文件。

在我们的情况下，我们根据平台创建一个管道名称。无论如何，我们确保它在我们当前的工作目录（`process.cwd()`）中，并且它被称为`temp`。从这里，我们可以创建一个服务器，并在这个文件上监听连接。当有人连接时，我们收到一个`Socket`对象。这是一个完整的双工流，这意味着我们可以从中读取和写入。我们还能够将信息传送到其中。在我们的情况下，我们想要记录到控制台`socket`加入，然后发送一个`DISCONNECT`消息。一旦我们收到关闭事件，我们就会记录`socket`关闭。

对于我们的客户端代码，我们应该有类似以下的东西：

```js
import net from 'net';
import path from 'path';
import os from 'os';

const pipeName = (os.platform() === 'win32') ? 
    path.join('\\\\?\\pipe', process.cwd(), 'temp') :
    path.join(process.cwd(), 'temp');
const socket = new net.Socket().connect(pipeName);
socket.on('connect', () => {
    console.log('we have connected');
});
socket.on('data', (data) => {
    if( data.toString('utf8') === 'DISCONNECT' ) {
        socket.destroy();
    }
});
```

这段代码与之前的代码非常相似，只是我们直接创建了一个`Socket`对象并尝试连接到相同的管道名称。一旦连接成功，我们就会记录下来。当我们收到数据时，我们会检查它是否等于我们的`DISCONNECT`消息，如果是，我们就会摆脱这个套接字。

IPC 机制的好处在于我们可以在不同语言编写的不同程序之间传递消息。它们唯一需要共同拥有的是某种形式的共同*语言*。有许多系统可以做到这一点。尽管这不是本书的重点，但请注意，如果我们需要连接到另一个程序，我们可以使用`net`模块相当容易地实现这一点。

# http 模块

我们要高层次地看一下的最后一个模块是`http`模块。这个模块允许我们轻松创建`http`服务器。以下是一个简单的`http`服务器示例：

```js
import http from 'http';

const server = http.createServer((req, res) => {
    res.writeHead(200, { 'Content-Type' : 'application/json'});
    res.end(JSON.stringify({here : 'is', some : 'data'}));
});
server.listen(8000, '127.0.0.1');
```

如果我们在浏览器中输入`localhost:8000`，我们应该能在浏览器中看到 JSON 对象。如果我们想变得更加花哨，我们可以发送一些基本的 HTML，比如下面这样：

```js
const server = https.createServer((req, res) => {
    res.writeHead(200, { 'Content-Type' : 'text/html' });
    res.end(`
        <html>
            <head></head>
            <body>
                <h1>Hello!</h1>
                <p>This is from our server!</p>
            </body>
        </html>
    `);
});
```

我们将内容类型设置为`text/html`，而不是`application/json`，以便浏览器知道如何解释这个请求。然后，我们用我们的基本 HTML 结束响应。如果我们的 HTML 请求 CSS 文件，我们将如何响应服务器？

我们需要解释请求并能够发送一些 CSS。我们可以使用以下方式来做到这一点：

```js
const server = http.createServer((req, res) => {
    if( req.method === 'GET' &&
        req.url === '/main.css' ) {
        res.writeHead(200, { 'Content-Type' : 'text/css' });
        res.end(`
            h1 {
                color : green;
            }
            p {
                color : purple;
            }
        `);
    } else {
        res.writeHead(200, { 'Content-Type' : 'text/html' });
        // same as above
    }
});
```

我们能够从接收到的请求中提取各种信息。在这种情况下，我们只关心这是否是一个`GET`请求，并且它是否在请求`main.css`资源。如果是，我们返回 CSS；否则，我们只返回我们的 HTML。值得注意的是，这段代码应该看起来与诸如 Express 之类的 Web 服务器框架相似。Express 添加了许多辅助方法和保护服务器的方法，但值得注意的是，我们可以通过利用 Node.js 内部的模块编写简单的服务器，减少依赖。

我们还可以使用`http`模块从各种资源中获取数据。如果我们使用内置在`http`模块中的`get`方法，甚至更通用的请求方法，我们可以从各种其他服务器获取资源。以下代码说明了这一点：

```js
import https from 'https';

https.get('https://en.wikipedia.org/wiki/Surprise_(emotion)', (res) => {
    if( res.statusCode === 200 ) {
        res.on('data', (data) => {
            console.log(data.toString('utf8'));
        });
        res.on('end', () => {
            console.log('no more information');
        });
    } else {
        console.error('bad error code!', res.statusCode);
    }
});
```

首先，我们可以看到我们必须利用`https`模块。由于这个网页位于一个使用**安全套接字层**（**SSL**）证书的服务器上，我们必须使用安全连接方法。然后，我们只需调用`get`方法，传入我们想要的 URL，并从响应中读取数据。如果出现某种原因，我们没有得到一个 200 响应（一个正常的消息），我们就会出错。

这三个模块应该展示了我们在 Node.js 生态系统中有相当大的能力，并且应该引发一些好奇心，让我们想知道如何在没有任何依赖的情况下使用 Node.js 来制作有用的系统。在下一节中，我们将看看如何在命令行调试器中调试我们的 Node.js 代码，以及我们习惯于在 Chrome 中使用的代码检查系统。

# 调试和检查代码

新的 Node.js 开发人员常常在调试代码方面遇到困难。与检查员不同，我们有一个系统，第一次崩溃会将一些信息转储到屏幕上，然后立即将我们踢到命令行。以下代码可以看到这一点：

```js
const thing = 'this';
console.log(thing);
thing = 10;
```

在这里，我们可以看到我们正在尝试重新分配一个常量，所以 Node.js 将抛出类似以下的错误：

```js
TypeError: Assignment to constant variable. 
 at Object.<anonymous> (C:\Code\Ch5\bad_code.js:3:7) 
 at Module._compile (internal/modules/cjs/loader.js:774:30) 
 at Object.Module._extensions..js (internal/modules/cjs/loader.js:785:10) 
 at Module.load (internal/modules/cjs/loader.js:641:32) 
 at Function.Module._load (internal/modules/cjs/loader.js:556:12) 
 at Function.Module.runMain (internal/modules/cjs/loader.js:837:10) 
 at internal/main/run_main_module.js:17:11
```

虽然这可能让人感到害怕，但它也向我们展示了错误的位置。这个堆栈跟踪中的第一行告诉我们它在*第 3 行*，*第 7 个字符*。

堆栈跟踪是系统向开发人员提供有关调用了什么函数的信息的一种方式。在我们的情况下，`Object.<anonymous>`被`Module.__compile`调用，依此类推。当堆栈的大部分是我们自己的时候，错误实际上发生在更远的地方时，这可能有所帮助。

有了这些信息，我们知道如何纠正问题，但如果我们想在特定语句或特定行上中断怎么办？这就是检查员系统发挥作用的地方。在这里，我们可以利用类似于我们在代码的 Web 版本中看到的语句。如果我们在代码的中间插入一个调试语句，我们的命令行将在那一点停止。

让我们创建一些基本的代码来展示这一点。以下代码应该足够展示检查员的使用：

```js
const fun = function() {
    const item = 10;
    for(let i = 0; i < item; i++) {
        const tempObj = {};
        tempObj[i] = "what " + i;
    }
    return function() {
        console.log('we will have access to other things');
        const alternative = 'what';
        debugger;
        return item;
    }
}

console.log('this is some code');
const x = 'what';
debugger;
fun()();
```

这段代码将允许我们玩弄检查员的各个部分。如果我们运行`npm inspect bad_code.js`命令，我们应该会在对`fun`的调用上中断。我们会看到一个终端界面，指出我们处于调试模式。现在我们在这里停止执行，我们可以设置一个监视器。这允许我们捕获各种变量和表达式，并查看它们在下一个中断时的结果。在这里，我们通过在调试器中执行`watch('x')`来设置一个监视器，监视`x`变量。从这里，如果我们输入`next`，我们将移动到下一行。如果我们这样做几次，我们会注意到一旦我们通过变量的赋值，监视器将把`x`变量从未定义变为 10。

当我们需要调试一个在相当多的对象之间共享状态的有状态系统时，这可能特别有帮助。当我们试图查看我们可以访问的内容时，这也可能有所帮助。让我们设置几个更多的监视器，以便在下一个调试语句被触发时看到它们的值。在以下变量上设置监视器：`item`、`tempObj`和`alternative`。现在，输入`cont`。这将把我们移动到下一个调试器语句。让我们看看我们的监视器打印出了什么。当我们移动到下一个点时，我们会看到`tempObj`和`x`未定义，但我们可以访问`item`和`alternative`。

这是我们所期望的，因为我们被限定在外部`fun`函数内部。我们可以用这个版本的检查员做更多事情，但我们也可以连接到我们习惯的检查员。

现在，如果我们使用以下命令运行我们的代码，我们将能够将调试工具附加到我们的脚本上：

```js
 > node --inspect bad_code.js 
```

有了这个，我们将得到一个我们可以连接的地址。让我们这样做。我们还需要有一些长时间运行的代码；否则，脚本将退出，我们将没有任何东西可以监听。让我们回到`named_pipe.js`示例。运行`node --inspect -–experimental-modules named_pipe.js`。

我们应该得到类似以下的东西：

```js
Debugger listening on ws://127.0.0.1:9229/6abd394d-d5e0-4bba-8b28-69069d2cb800
```

如果我们在 Chrome 浏览器中输入以下地址，我们应该会看到一个熟悉的界面：

```js
chrome-devtools://devtools/bundled/js_app.html?experiments=true&v8only=true&ws=<url>
```

现在，我们可以在 Node.js 代码中使用 Chrome 的检查器的全部功能。在这里，我们可以看到，如果我们用`named_pipe_child.js`文件连接到我们的命名管道服务器，我们将在调试器中看到控制台日志。现在，如果我们添加调试器语句，我们应该能够在检查器中得到断点。如果我们在套接字连接到我们时添加一个调试语句，当我们用子套接字连接时，我们将能够以与在浏览器中一样的方式运行我们的代码！这是调试和逐步执行我们的代码的好方法。

我们还可以进行内存分析。如果我们转到内存选项卡并创建堆快照，我们将得到一个漂亮的内存转储。它应该看起来非常熟悉，就像我们已经看到的那样。

有了这些知识，我们可以进入围绕 Node.js 的更复杂的主题。

# 总结

随着 Node.js 的出现，我们能够使用一种编程语言，可以在客户端和服务器上都使用。虽然 Node.js 给我们的 API 可能看起来不太熟悉，但我们可以用它们创建强大的服务器应用程序。

在本章中，我们介绍了流的基础知识以及一些允许我们创建强大服务器应用程序的 API。我们还看了一些工具，可以让我们在有 GUI 和没有 GUI 的情况下进行调试。

有了这些知识，下一章中，我们将更深入地探讨我们可以使用的机制，以在线程和进程之间传递数据。
