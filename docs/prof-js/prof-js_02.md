# 第三章：*第二章*

# Node.js 和 npm

## 学习目标

在本章结束时，您将能够：

+   安装和使用 Node.js 构建应用程序

+   使用 Node.js 执行环境运行 JavaScript 代码

+   使用 nvm 安装和管理多个 Node.js 版本

+   识别并使用其他开发人员开发的模块，使用 npm

+   创建和配置自己的 npm 包

在本章中，我们将转向后端世界，学习有关 Node.js 及其基本概念。我们将学习如何使用 nvm 安装和管理多个 Node.js 版本，然后我们将学习 npm 以及如何查找和使用外部模块。

## 介绍

在上一章中，我们了解了 HTML 如何成为 DOM 以及如何使用 JavaScript 来查询和操作页面内容。

在 JavaScript 出现之前，所有页面都是静态的。在 Netscape 将脚本环境引入其浏览器后，开发人员开始使用它来创建动态和响应式应用程序。应用程序变得越来越复杂，但 JavaScript 运行的唯一地方是在浏览器内部。然后，在 2009 年，Node.js 的原始开发人员 Ryan Dahl 决定创建一种在服务器端运行 JavaScript 的方式，通过允许他们构建应用程序而无需依赖其他语言，简化了 Web 开发人员的生活。

在本章中，您将学习 Node.js 的工作原理以及如何使用它来使用 JavaScript 创建脚本。您将了解 Node.js 核心 API 的基础知识，以及如何找到它们的文档，并如何使用它们的**read-eval-print loop** (**REPL**)命令行。

掌握构建 JavaScript 代码的技能后，您将学习如何管理多个 Node.js 版本，并了解 Node.js 的重要性。您还将学习 npm 是什么，以及如何导入和使用其他开发人员的软件包并构建 Node.js 应用程序。

## 什么是 Node.js？

Node.js 是在 V8 JavaScript 引擎之上运行的执行环境。它的基本前提是它是异步和事件驱动的。这意味着所有阻塞操作，例如从文件中读取数据，可以在后台处理，而应用程序的其他部分可以继续工作。当数据加载完成时，将发出事件，等待数据的人现在可以执行并进行工作。

从诞生之初，Node.js 就被设计为 Web 应用程序的高效后端。因此，它被各种规模和行业类型的公司广泛采用。Trello、LinkedIn、PayPal 和 NASA 是一些在其技术堆栈的多个部分中使用 Node.js 的公司。

但是什么是执行环境？执行环境为程序员编写应用程序提供基本功能，例如 API。例如，想象一下浏览器-它具有 DOM，诸如文档和窗口的对象，诸如`setTimeout`和`fetch`的函数，以及前端世界中可以做的许多其他事情。所有这些都是浏览器执行环境的一部分。由于该执行环境专注于浏览器，它提供了与 DOM 交互和与服务器通信的方式，这是它存在的全部。

Node.js 专注于为开发人员提供一种有效构建 Web 应用程序后端的环境。它提供 API 来创建 HTTP(S)服务器，读写文件，操作进程等。

正如我们之前提到的，Node.js 在底层使用 V8 JavaScript 引擎。这意味着为了将 JavaScript 文本转换为计算机处理的可执行代码，它使用了 V8，这是由 Google 构建的开源 JavaScript 引擎，用于驱动 Chromium 和 Chrome 浏览器。以下是这个过程的示例：

![图 2.1：Node.js 使用 V8 引擎将 JavaScript 源代码转换为可在处理器中运行的可执行代码](img/C14587_02_01.jpg)

###### 图 2.1：Node.js 使用 V8 引擎将 JavaScript 源代码转换为在处理器中运行的可执行代码

Node.js 提供的执行环境是单线程的。这意味着每次只有一段 JavaScript 代码可以执行。但是 Node.js 有一个叫做事件循环的东西，它可以将等待某些东西的代码（比如从文件中读取数据）放入队列，而另一段代码可以执行。

从文件中读取或写入数据以及通过网络发送或接收数据都是由系统内核处理的任务，在大多数现代系统中都是多线程的。因此，一些工作最终会分布在多个线程中。但对于在 Node.js 执行环境中工作的开发人员来说，这一切都隐藏在一个叫做异步编程的编程范式中。

异步编程意味着你将要求执行一些任务，当结果可用时，你的代码将被执行。让我们回到从文件中读取数据的例子。在大多数编程语言和范式中，你只需编写一些伪代码，如下所示：

```js
var file = // open file here
var data = file.read(); // do something with data here
```

采用异步编程模型，工作方式有所不同。你打开文件并告诉 Node.js 你想要读取它。你还给它一个回调函数，当数据对你可用时将被调用。伪代码如下：

```js
var file = // open file here
file.read((data) => {
  // do something with data here
});
```

在这个例子中，脚本将被加载，并开始执行。脚本将逐行执行并打开文件。当它到达读取操作时，它开始读取文件并安排稍后执行回调。之后，它到达脚本的末尾。

当 Node.js 到达脚本的末尾时，它开始处理事件循环。事件循环分为阶段。每个阶段都有一个队列，存储着计划在其中运行的代码。例如，I/O 操作被安排在轮询阶段。有六个阶段，它们按以下顺序执行：

1.  **计时器**：使用`setTimeout`或`setInterval`计划的代码

1.  **挂起** **回调**：上一个周期的 I/O 的延迟回调

1.  **空闲**，**准备**：仅内部

1.  **轮询**：计划进行 I/O 处理的代码

1.  **检查**：`setImmediate`回调在这里执行

1.  **关闭回调**：计划在关闭套接字等上执行的代码

每个阶段都会执行代码，直到发生两种情况之一：阶段队列耗尽，或者执行了最大数量的回调：

![图 2.2：事件循环阶段](img/C14587_02_02.jpg)

###### 图 2.2：事件循环阶段

要理解这是如何工作的，让我们看一些代码，将阶段映射到事件循环，并了解底层到底发生了什么：

```js
console.log('First');
setTimeout(() => {
  console.log('Last');
}, 100);
console.log('Second');
```

在这段简短的代码中，我们向控制台打印一些内容（在 Node.js 中，默认情况下会输出到标准输出），然后我们设置一个函数在`100`毫秒后调用，并向控制台打印一些其他文本。

当 Node.js 启动你的应用程序时，它会解析 JavaScript 并执行脚本直到结束。当结束时，它开始事件循环。这意味着，直接打印到控制台时，它会立即执行。计划的函数被推送到计时器队列，并等待脚本完成（以及**100**毫秒过去）才会执行。当事件循环没有任务可执行时，应用程序结束。以下图表说明了这个过程：

![图 2.3：Node.js 应用程序的执行流程](img/C14587_02_03.jpg)

###### 图 2.3：Node.js 应用程序的执行流程

由于执行顺序，应用程序的输出如下：

```js
First
Second
Last
```

这里发生了两件重要的事情。首先，传递给`setTimeout`函数的代码在脚本执行完成后执行。其次，应用程序在脚本执行到最后不会立即退出；相反，它会等待事件循环耗尽要执行的任务。

Node.js 有两种执行方法。最常用的是当您传递文件的路径时，JavaScript 代码将从那里加载和执行。第二种方法是在 REPL 中。如果您执行 Node.js 命令而不给出任何参数，它将以 REPL 模式启动，这类似于我们在上一章中看到的 Dev Tools 中的控制台。让我们在下一个练习中详细探讨这一点。

### 练习 8：运行您的第一个 Node.js 命令

在这个练习中，您将在计算机上下载和安装 Node.js，创建您的第一个脚本并运行它。然后，我们将使用 Node.js 附带的 REPL 工具，并在其中运行一些命令。

#### 注意

要能够运行 Node.js 应用程序，您需要在计算机上安装它。为此，您可以转到`nodejs.org`并下载 Node.js 软件包。建议下载最新的**长期支持**（**LTS**）版本，这将为您提供最稳定和最长的安全和错误修补支持时间。在撰写本文时，该版本为`10.16.0`。

执行以下步骤以完成此练习：

1.  下载并安装 Node.js 后，转到命令行并检查您已安装的版本：

```js
$ node –version
v10.16.0
```

1.  现在，创建一个名为`event_loop.js`的新文本文件，并添加代码的扩展版本（事件循环示例），如前所示。它看起来像这样：

```js
console.log('First');
const start = Date.now();
setTimeout(() => {
  console.log(`Last, after: ${Date.now() - start}ms`);
}, 100);
console.log('Second');
```

1.  要使用 Node.js 运行 JavaScript，调用`node`并传递要执行的文件的路径。要运行刚刚创建的文件，请在命令行中执行以下代码，从您创建文件的目录中执行：

```js
$ node event_loop.js
```

您将看到以下输出：

```js
$ node event_loop.js
First
Second
Last, after: 106ms
```

最后看到的时间将在每次运行时都有所不同。这是因为`setTimeout`只能确保代码将在指定的时间之后运行，但不能保证它会准确地在您要求的时间执行。

1.  运行`node`命令而不带任何参数；您将进入 REPL 模式：

```js
$ node
>
```

`>`表示您现在在 Node.js 执行环境中。

1.  在 REPL 命令行中，键入命令并按*Enter*执行。让我们尝试第一个：

```js
> console.log('First');
First
Undefined
```

你可以看到它打印出你传递给`console.log`调用的字符串。它还打印出`Undefined`。这是最后执行语句的返回值。由于`console.log`没有返回任何东西，它打印了 undefined。

1.  创建存储当前时间的常量：

```js
> const start = Date.now()
undefined
```

1.  声明变量也不会返回任何东西，所以它再次打印`undefined`：

```js
> start
1564326469948
```

如果要知道变量的值是多少，只需键入变量名称并按*Enter*。变量名称的返回语句是变量值，因此它会打印出该值。

1.  现在，键入`setTimeout`调用，就像在您的文件中一样。如果您按*Enter*并且您的语句不完整，因为您正在启动一个函数或打开括号，Node.js 将打印省略号，表示它正在等待命令的其余部分：

```js
> setTimeout(() => {
... 
```

1.  您可以继续键入，直到所有命令都被键入。`setTimeout`函数返回一个`Timeout`对象，您可以在控制台中看到它。您还可以看到在执行回调时打印的文本：

```js
> setTimeout(() => {
...   console.log('Last, after: ${Date.now() - start}ms');
... }, 100);
```

以下是前述代码的输出：

```js
Timeout {
  _called: false,
  _idleTimeout: 100,
  _idlePrev: [TimersList],
  _idleNext: [TimersList],
  _idleStart: 490704,
  _onTimeout: [Function],
  _timerArgs: undefined,
  _repeat: null,
  _destroyed: false,
  domain: [Domain],
  [Symbol(unrefed)]: false,
  [Symbol(asyncId)]: 492,
  [Symbol(triggerId)]: 5 }
> Last, after: 13252ms
```

您可以看到打印出的时间远远超过了`100`毫秒。这是因为`start`变量是一段时间前声明的，它正在从初始值中减去当前时间。因此，该时间表示`100`毫秒，再加上您键入和执行命令所花费的时间。

1.  尝试更改`start`的值。您会观察到 Node.js 不会让您这样做，因为我们将其声明为常量：

```js
> start = Date.now();
Thrown:
TypeError: Assignment to constant variable.
```

我们可以尝试将其重新声明为一个变量，但是 Node.js 不会让我们这样做，因为它已经在当前环境中声明过了：

```js
> let start = Date.now()
Thrown:
SyntaxError: Identifier 'start' has already been declared
```

1.  在另一个函数中声明超时的整个调度，以便每次执行函数时都获得一个新的作用域：

```js
> const scheduleTimeout = () => {
... const start = Date.now();
... setTimeout(() => {
..... console.log('Later, after: ${Date.now() - start}');
..... }, 100);
... };
```

每次调用该函数，它都会安排并在`100`毫秒后执行，就像在您的脚本中一样。这将输出以下内容：

```js
Undefined
> scheduleTimeout
[Function: scheduleTimeout]
> scheduleTimeout()
Undefined
> Later, after: 104
```

1.  要退出 REPL 工具，您可以按两次*Ctrl + C*，或者输入`.exit`然后按*Enter*：

```js
>
(To exit, press ^C again or type .exit)
>
```

安装 Node.js 并开始使用它非常容易。其 REPL 工具允许您快速原型设计和测试。了解如何使用这两者可以提高您的生产力，并在日常 JavaScript 应用程序开发中帮助您很多。

在这个练习中，您安装了 Node.js，编写了一个简单的程序，并学会了如何使用 Node.js 运行它。您还使用了 REPL 工具来探索 Node.js 执行环境并运行一些代码。

## Node 版本管理器（nvm）

Node.js 和 JavaScript 拥有一个庞大的社区和非常快速的开发周期。由于这种快速的发展和发布周期，很容易过时（查看 Node.js 的先前版本页面以获取更多信息：[`nodejs.org/en/download/releases/`](https://nodejs.org/en/download/releases/)）。

你能想象在一个使用 Node.js 且已经存在几年的项目上工作吗？当您回来修复一个错误时，您会注意到您安装的版本无法再运行代码，因为存在一些兼容性问题。或者，您会发现您无法使用当前版本更改代码，因为生产环境中运行的版本已经有几年了，没有 async/await 或其他您在最新版本中经常使用的功能。

这个问题发生在所有编程语言和开发环境中，但在 Node.js 中，由于其极快的发布周期，这一点尤为突出。

为了解决这个问题，通常会使用版本管理工具，这样您就可以快速在 Node.js 的不同版本之间切换。**Node 版本管理器**（**nvm**）是一个广泛使用的工具，用于管理安装的 Node.js 版本。您可以在[`github.com/nvm-sh/nvm`](https://github.com/nvm-sh/nvm)上找到有关如何下载和安装它的说明。

#### 注意

如果您使用 Windows，可以尝试 nvm-windows（[`github.com/coreybutler/nvm-windows`](https://github.com/coreybutler/nvm-windows)），它为 Linux 和 Mac 中的 nvm 提供了类似的功能。此外，在本章中，许多命令都是针对 Mac 和 Linux 的。对于 Windows，请参阅`nvm-windows`的帮助部分。

安装程序在您的系统中执行两件事：

1.  在您的主目录中创建一个`.nvm`目录，其中放置了所有与管理 Node.js 的所有托管版本相关的脚本

1.  添加一些配置以使 nvm 在所有终端会话中可用

nvm 非常简单易用，并且有很好的文档。其背后的想法是您的机器上将运行多个版本的 Node.js，您可以快速安装新版本并在它们之间切换。

在我的电脑上，我最初只安装了一段时间前下载的 Node.js 版本（10.16.0）。安装 nvm 后，我运行了列出所有版本的命令。以下是输出：

```js
$ nvm ls

->system
iojs -> N/A (default)
node -> stable (-> N/A) (default)
unstable -> N/A (default)
```

您可以看到我没有其他版本可用。我还有一个系统版本，这是您在系统中安装的任何版本。我可以通过运行 `node --version` 来检查当前的 Node.js 版本：

```js
$ node --version
v10.16.0
```

作为使用 nvm 的示例，假设您想要在最新版本上测试一些实验性功能。您需要做的第一件事是找出那个版本。因此，您运行`nvm ls-remote`命令（或者对于 Windows 系统，运行`nvm list`命令），这是列出远程版本的命令：

```js
$ nvm ls-remote
        v0.1.14
        v0.1.15
        v0.1.16
       ...
       v10.15.3   (LTS: Dubnium)
       v10.16.0   (Latest LTS: Dubnium)
       ...
        v12.6.0
        v12.7.0
```

这将打印出所有可用版本的长列表。在写作时，最新的版本是 12.7.0，所以让我们安装这个版本。要安装任何版本，请运行`nvm install` <`version`>命令。这将下载指定版本的 Node.js 二进制文件，验证包是否损坏，并将其设置为终端中的当前版本：

```js
$ nvm install 12.7.0
Downloading and installing node v12.7.0...
Downloading https://nodejs.org/dist/v12.7.0/node-v12.7.0-darwin-x64.tar.xz...
######################################################################## 100.0%
Computing checksum with shasum -a 256
Checksums matched!
Now using node v12.7.0 (npm v6.10.0)
```

现在，您可以验证您已经安装了最新版本，并准备在终端中使用：

```js
$ node --version
v12.7.0
```

或者，您可以直接使用别名`node`，这是最新版本的别名。但是对于 Windows，您需要提到需要安装的特定版本：

```js
$ nvm install node
v12.7.0 is already installed.
Now using node v12.7.0 (npm v6.10.0)
```

广泛使用的框架和语言（如 Node.js）通常会为特定版本提供 LTS。这些 LTS 版本被认为更稳定，并保证对错误和安全修复提供更长时间的支持，这对于无法像正常发布周期那样快速迁移到新版本的公司或团队来说非常重要。如果您想使用最新的 LTS 版本，可以使用`--lts`选项：

```js
$ nvm install --lts
Installing the latest LTS version.
Downloading and installing node v10.16.0...
Downloading https://nodejs.org/dist/v10.16.0/node-v10.16.0-darwin-x64.tar.xz...
######################################################################## 100.0%
Computing checksum with shasum -a 256
Checksums matched!
Now using node v10.16.0 (npm v6.9.0)
```

使用 nvm 安装多个版本的 Node.js 后，您可以使用`use`命令在它们之间切换：

```js
$ nvm use system --version
Now using system version of node: v10.16.0 (npm v6.9.0)
$ nvm use node
Now using node v12.7.0 (npm v6.10.0)
$ nvm use 7
Now using node v7.10.1 (npm v4.2.0)
```

当您有多个项目并经常在它们之间切换时，很难记住您为每个项目使用的 Node.js 版本。为了让我们的生活更轻松，nvm 支持项目目录中的配置文件。您只需在项目的根目录中添加一个`.nvmrc`文件，它将使用文件中的版本。您还可以在项目的任何父目录中添加一个`.nvmrc`文件。因此，如果您想在父目录中按 Node.js 版本对项目进行分组，可以在该父目录中添加配置文件。

例如，如果您在一个文件夹中有一个`.nvmrc`文件，版本为`12.7.0`，当您切换到该文件夹并运行`nvm use`时，它将自动选择该版本：

```js
$ cat .nvmrc 
12.7.0
$ nvm use
Found '.../Lesson02/Exercise09/.nvmrc' with version <12.7.0>
Now using node v12.7.0 (npm v6.10.0)
```

### 练习 9：使用 nvm 管理版本

正如我们之前提到的，Node.js 的发布周期非常短。例如，如果您寻找 URL 类（[`nodejs.org/dist/latest-v12.x/docs/api/url.html#url_class_url`](https://nodejs.org/dist/latest-v12.x/docs/api/url.html#url_class_url)），您会发现它最近才在全局范围内可用。这发生在 10.0.0 版本中，这个版本在写作时只有大约一年的历史。

在这个练习中，我们将编写一个`.nvmrc`文件，使用 nvm 安装多个版本的 Node.js，并尝试不同的版本，看看当您使用错误的 Node.js 版本时会得到什么类型的错误。

执行以下步骤完成这个练习：

1.  在您的项目中添加一个`.nvmrc`文件。在一个空文件夹中，创建一个名为`.nvmrc`的文件，并在其中添加数字 12.7.0。您可以使用`echo`命令一次完成这个操作，并将输出重定向到文件中：

```js
$ echo '12.7.0' > .nvmrc
```

1.  您可以使用`cat`命令检查文件是否包含您想要的内容：

```js
$ cat .nvmrc
12.7.0
```

1.  让我们使用`nvm use`命令，它将尝试使用`.nvmrc`文件中的版本：

```js
$ nvm use
Found '.../Lesson02/Exercise09/.nvmrc' with version <12.7.0>
N/A: version "12.7.0 -> N/A" is not yet installed.
```

在使用之前，您需要运行`nvm install 12.7.0`来安装它。如果您没有安装指定的版本，nvm 将给出清晰的消息。

1.  调用`nvm install`来安装项目需要的版本：

```js
$ nvm install
Found '.../Lesson02/Exercise09/.nvmrc' with version <12.7.0>
Downloading and installing node v12.7.0...
Downloading https://nodejs.org/dist/v12.7.0/node-v12.7.0-darwin-x64.tar.xz...
#################################################################### 100.0%
Computing checksum with shasum -a 256
Checksums matched!
Now using node v12.7.0 (npm v6.10.0)
```

请注意，您不必传递您想要的版本，因为 nvm 将从`.nvmrc`文件中获取这个版本。

1.  现在，创建一个名为`url_explorer.js`的文件。在其中，通过传递完整的 URL 来创建一个 URL 的实例。让我们还添加一些调用来探索 URL 的各个部分：

```js
const url = new URL('https://www.someserver.com/not/a/path?param1=value1&param2=value2`);
console.log(`URL is: ${url.href}`);
console.log(`Hostname: ${url.hostname}`);
console.log(`Path: ${url.pathname}`);
console.log(`Query string is: ${url.search}`);
console.log(`Query parameters:`)
Array.from(url.searchParams.entries())
  .forEach((entry) => console.log(`\t- ${entry[0]} = ${entry[1]}`));
```

1.  运行脚本。您会看到 URL 被正确解析，并且所有关于它的细节都正确地打印到控制台上：

```js
$ node url_explorer.js
URL is: https://www.someserver.com/not/a/path?param1=value1&param2=value2
Hostname: www.someserver.com
Path: /not/a/path
Query string is: ?param1=value1&param2=value2
Query parameters:
    - param1 = value1
    - param2 = value2
```

1.  现在，让我们尝试错误的 Node.js 版本。使用`nvm`安装版本`9.11.2`：

```js
$ nvm install 9.11.2
Downloading and installing node v9.11.2...
Downloading https://nodejs.org/dist/v9.11.2/node-v9.11.2-darwin-x64.tar.xz...
################################################################## 100.0%
Computing checksum with shasum -a 256
Checksums matched!
Now using node v9.11.2 (npm v5.6.0)
```

1.  现在，您可以再次运行`url_explorer.js`，看看会发生什么：

```js
$ node url_explorer.js
.../Exercise09/url_explorer.js:1 ... { const url = new URL('...);^
ReferenceError: URL is not defined
    at Object.<anonymous> (.../Exercise09/url_explorer.js:1:75)
    at Module._compile (internal/modules/cjs/loader.js:654:30)
    at Object.Module._extensions..js (internal/modules/cjs/loader.js:665:10)
    at Module.load (internal/modules/cjs/loader.js:566:32)
    at tryModuleLoad (internal/modules/cjs/loader.js:506:12)
    at Function.Module._load (internal/modules/cjs/loader.js:498:3)
    at Function.Module.runMain (internal/modules/cjs/loader.js:695:10)
    at startup (internal/bootstrap/node.js:201:19)
    at bootstrapNodeJSCore (internal/bootstrap/node.js:516:3)
```

您应该看到与前面代码中相似的错误。它告诉您 URL 未定义。这是因为，正如我们之前提到的，URL 类只在 10.0.0 版本中变为全局可用。

1.  修复 Node.js 的版本并再次运行脚本以查看正确的输出：

```js
$ nvm use
Found '.../Lesson02/Exercise09/.nvmrc' with version <12.7.0>
Now using node v12.7.0 (npm v6.10.0)
$ node url_explorer.js 
URL is: https://www.someserver.com/not/a/path?param1=value1&param2=value2
Hostname: www.someserver.com
Path: /not/a/path
Query string is: ?param1=value1&param2=value2
Query parameters:
    - param1 = value1
    - param2 = value2
```

第 7 步中的错误消息没有提及 Node.js 版本。它只是一些关于缺少类的神秘错误。这类错误很难识别，并需要大量的历史追踪。这就是为什么在项目的根目录中有`.nvmrc`是重要的原因。它使其他开发人员能够快速识别和使用正确的版本。

在这个练习中，您学会了如何安装和使用多个版本的 Node.js，还学会了为项目创建`.nvmrc`文件。最后，您还了解了在使用错误版本时会看到的错误类型，以及`.nvmrc`文件的重要性。

## Node 包管理器（npm）

当有人谈论**Node 包管理器**或简称 npm 时，他们可能指的是以下三种情况之一：

+   一个管理 Node.js 应用程序包的命令行应用程序

+   开发人员和公司发布他们的包供他人使用的存储库

+   管理个人资料和搜索包的网站

大多数编程语言至少提供一种开发人员之间共享包的方式：Java 有 Maven，C#有 NuGet，Python 有 PIP 等。Node.js 在初始发布几个月后开始使用自己的包管理器。

包可以包括开发人员认为对他人有用的任何类型的代码。有时，它们还包括帮助开发人员进行本地开发的工具。

由于打包的代码需要共享，因此需要一个存储所有包的存储库。为了发布他们的包，作者需要注册并注册自己和他们的包。这解释了存储库和网站部分。

第三部分，即命令行工具，是您应用程序的实际包管理器。它随 Node.js 一起提供，并可用于设置新项目、管理依赖项以及管理应用程序的脚本，如构建和测试脚本。

#### 注意

Node.js 项目或应用程序也被视为一个包，因为它包含一个`package.json`文件，代表了包中的内容。因此，通常可以互换使用以下术语：应用程序、包和项目。

每个 Node.js 包都有一个描述项目及其依赖关系的`package.json`文件。要为您的项目创建一个`package.json`文件，您可以使用`npm init`命令。只需在您想要项目存在的文件夹中运行它：

```js
$ cd sample_npm
$ npm init
This utility will walk you through creating a package.json file.
It only covers the most common items and tries to guess sensible defaults.
See 'npm help json' for definitive documentation on these fields and exactly what they do.
Use 'npm install <pkg>' afterwards to install a package and save it as a dependency in the package.json file.
Press ^C at any time to quit.
package name: (sample_npm) 
version: (1.0.0) 
description: Sample project for the Professional JavaScript.
entry point: (index.js) 
test command: 
git repository: https://github.com/TrainingByPackt/Professional-JavaScript/
keywords: 
author: 
license: (ISC) MIT
About to write to .../Lesson02/sample_npm/package.json:
{
  "name": "sample_npm",
  "version": "1.0.0",
  "description": "Sample project for the Professional JavaScript.",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/TrainingByPackt/Professional-JavaScript.git"
  },
  "author": "",
  "license": "MIT",
  "bugs": {
    "url": "https://github.com/TrainingByPackt/Professional-JavaScript/issues"
  },
  "homepage": "https://github.com/TrainingByPackt/Professional-JavaScript#readme"
}
Is this OK? (yes) yes
```

该命令将询问您一些问题，指导您创建`package.json`文件。最后，它将打印生成的文件并要求您确认。它包含关于项目的所有信息，包括代码的位置、使用的许可证以及作者是谁。

现在我们有了一个 npm 包，我们可以开始寻找可以使用的外部模块。让我们去[`npmjs.com`](https://npmjs.com)寻找一个帮助我们解析命令行参数的包。在搜索框中输入**command line**并按*Enter*键，我们会得到一个包选择列表：

![图 2.4：搜索一个包来帮助我们构建一个命令行应用程序](img/C14587_02_04.jpg)

###### 图 2.4：搜索一个包来帮助我们构建一个命令行应用程序

由于我们正在寻找一个工具来帮助我们解析命令行参数，**commander**听起来像是一个不错的解决方案。它的简短描述是**node.js 命令行程序的完整解决方案**。让我们在一个应用程序中安装它，并使用它来理解这个流程是如何工作的。

要将包添加为您的包的依赖项，您可以从命令行请求 npm 按名称安装它：

```js
$ npm install commander
npm notice created a lockfile as package-lock.json. You should commit this file.
+ commander@2.20.0
added 1 package from 1 contributor and audited 1 package in 1.964s
found 0 vulnerabilities
```

您可以看到 npm 找到了该包并下载了最新版本，截至本文撰写时为`2.20.0`。它还提到了关于`package-lock.json`文件的一些内容。我们将稍后更多地讨论这个问题，所以现在不用担心它。

最近添加到 npm 的另一个很酷的功能是漏洞检查。在`install`命令输出的末尾，您可以看到有关发现的漏洞的注释，或者更好的是，没有发现漏洞。npm 团队正在努力增加对其存储库中所有包的漏洞检查和安全扫描。

#### 注意

从 npm 使用包是如此简单，以至于很多人都在向那里推送恶意代码，以捕捉最不注意的开发人员。强烈建议您在从 npm 安装包时要非常注意。检查拼写、下载次数和漏洞报告，并确保您要安装的包确实是您想要的。您还需要确保它来自可信任的方。 

运行`npm install`后，您会注意到`package.json`文件中添加了一个新的部分。它是`dependencies`部分，包含您刚刚请求的包：

```js
"dependencies": {
  "commander": "².20.0"
}
```

这就是`install`命令输出中`commander`前面的+号的含义：该包已作为项目的依赖项添加。

`dependencies`部分用于自动检测和安装项目所需的所有包。当您在一个具有`package.json`文件的 Node.js 应用程序上工作时，您不必手动安装每个依赖项。您只需运行`npm install`，它将根据`package.json`文件的`dependencies`部分自动解决所有问题。这里是一个例子：

```js
$ npm install
added 1 package from 1 contributor and audited 1 package in 0.707s
found 0 vulnerabilities
```

尽管没有指定任何包，npm 假定您想要安装当前包的所有依赖项，这些依赖项来自`package.json`。

除了向`package.json`文件添加`dependencies`部分之外，它还创建了一个`node_modules`文件夹。那是它下载并保留项目所有包的地方。您可以使用列表命令（`ls`）检查`node_modules`中的内容：

```js
$ ls node_modules/
commander
$ ls node_modules/commander/
CHANGELOG.md  LICENSE   Readme.md   index.js    package.json  typings
```

如果您再次运行`npm install`来安装 commander，您会注意到 npm 不会再次安装该包。它只显示该包已更新和已审核：

```js
$ npm install commander
+ commander@2.20.0
updated 1 package and audited 1 package in 0.485s
found 0 vulnerabilities
```

在下一个练习中，我们将构建一个使用 commander 作为依赖项的 npm 包，然后创建一个命令行 HTML 生成器。

### 练习 10：创建一个命令行 HTML 生成器

现在您已经学会了使用 npm 创建包以及如何安装一些依赖项的基础知识，让我们把这些知识整合起来，构建一个可以为您的下一个网站项目生成 HTML 模板的命令行工具。

在这个练习中，您将创建一个 npm 包，该包使用 commander 作为处理命令行参数的依赖项。然后，您将探索您创建的工具，并生成一些 HTML 文件。

此练习的代码可以在 GitHub 上找到，网址为[`github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson02/Exercise10`](https://github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson02/Exercise10)。

执行以下步骤以完成此练习：

1.  创建一个新的文件夹，您将在其中放置此练习的所有文件。

1.  在命令行中，切换到新文件夹并运行`npm init`来初始化一个`package.json`文件。选择所有默认选项应该就足够了：

```js
$ npm init
This utility will walk you through creating a package.json file.
...
Press ^C at any time to quit.
package name: (Exercise10) 
version: (1.0.0) 
...
About to write to .../Lesson02/Exercise10/package.json:
{
  "name": "Exercise10",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
Is this OK? (yes)
```

1.  安装`commander`包作为依赖项：

```js
$ npm install commander
npm notice created a lockfile as package-lock.json. You should commit this file.
+ commander@2.20.0
added 1 package from 1 contributor and audited 1 package in 0.842s
found 0 vulnerabilities
```

在您的`package.json`中，添加以下内容：

```js
"main": "index.js"
```

这意味着我们应用程序的入口点是`index.js`文件。

1.  运行一个具有入口点的 npm 包，并使用`node`命令，传递包含`package.json`文件的目录。以下是一个在`Lesson02/sample_npm`中运行该包的示例，该示例可在[`github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson02/sample_npm`](https://github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson02/sample_npm)上找到：

```js
$ node sample_npm/
I'm an npm package running from sample_npm
```

1.  创建一个名为`index.js`的文件，在其中使用`require`函数加载`commander`包：

```js
const program = require('commander');
```

这就是您开始使用外部包所需要的全部内容。

Commander 解析传入 Node.js 应用程序的参数。您可以配置它告诉它您期望的参数类型。对于这个应用程序，我们将有三个选项：`-b`或`--add-bootstrap`，它将在生成的输出中添加 bootstrap 4；`-c`或`--add-container`，它将在 body 中添加一个带有 ID container 的`<div>`标签；以及`-t`或`--title`，它将在页面上添加一个接受标题文本的`<title>`。

1.  配置 commander，我们调用 version 方法，然后多次调用 option 方法来添加应用程序将支持的每个选项。最后，我们调用`parse`，它将验证传入的参数（`process.argv`将在下一章详细讨论）是否与预期的选项匹配：

```js
program.version('0.1.0')
  .option('-b, --add-bootstrap', 'Add Bootstrap 4 to the page.')
  .option('-c, --add-container', 'Adds a div with container id in the body.')
  .option('-t, --title [title]', 'Add a title to the page.')
  .parse(process.argv);
```

1.  现在，您可以运行您的应用程序并查看到目前为止的结果：

```js
$ node . –help
```

我们将收到以下输出：

```js
Usage: Exercise10 [options]
Options:
  -V, --version        output the version number
  -b, --add-bootstrap  Add Bootstrap 4 to the page.
  -c, --add-container  Adds a div with container id in the body.
  -t, --title [title]  Add a title to the page.
  -h, --help           output usage information
```

您可以看到 commander 为您提供了一个很好的帮助消息，解释了您的工具应该如何使用。

1.  现在，让我们使用这些选项来生成 HTML。我们需要做的第一件事是声明一个变量，用于保存所有的 HTML：

```js
let html = '<html><head>';
```

我们可以使用`<html>`和`<head>`开放标签来初始化它。

1.  然后，检查程序是否接收到`title`选项。如果是，就添加一个带有传入标签内容的`<title>`标签：

```js
if (program.title) {
  html += `<title>${program.title}</title>`;
}
```

1.  对于`Bootstrap`选项也是同样的操作。在这种情况下，选项只是一个布尔值，因此您只需检查并添加一个指向`Bootstrap.css`文件的`<link>`标签：

```js
if (program.addBootstrap) {
  html += '<link';
  html += ' rel="stylesheet"';
  html += ' href="https://stackpath.bootstrapcdn.com';
  html += '/bootstrap/4.3.1/css/bootstrap.min.css"';
  html += '/>';
}
```

1.  关闭`<head>`标签并打开`<body>`标签：

```js
html += '</head><body>';
```

1.  检查容器`<div>`选项，并在启用时添加它：

```js
if (program.addContainer) {
  html += '<div id="container"></div>';
}
```

1.  最后，关闭`<body>`和`<html>`标签，并将 HTML 打印到控制台：

```js
html += '</body></html>';
console.log(html);
```

1.  不带任何选项运行应用程序将给我们一个非常简单的 HTML：

```js
$ node .
<html><head></head><body></body></html>
```

1.  运行应用程序，启用所有选项：

```js
$ node . -b -t Title -c
This will return a more elaborate HTML:
<html><head><title>Title</title><link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css"/></head><body><div id="container"></div></body></html>
```

npm 使得在您的应用程序中使用包变得非常容易。像 commander 和 npm 存储库中的其他数以千计的包使得 Node.js 成为构建功能强大且复杂的应用程序的绝佳选择，而代码量却很少。探索和学习如何使用包可以为您节省大量时间和精力，这将决定一个项目是否能够成功应用于数百万用户。

在这个练习中，您创建了一个 npm 包，使用外部包来解析命令行参数，这通常是一项费力的任务。您已经配置了 commander 来将参数解析为一个很好的可用格式，并学会了如何使用解析后的参数来构建一个根据用户输入做出决策的应用程序。

### 依赖项

在上一节中，我们看到 npm 如何使用`package.json`文件的`dependencies`部分来跟踪您的包的依赖关系。依赖关系是一个复杂的话题，但您必须记住的是，npm 支持语义版本或 semver 格式的版本号，并且它可以使用区间和其他复杂的运算符来确定您的包可以接受其他包的哪些版本。

默认情况下，正如我们在上一个练习中看到的，npm 使用插入符号标记所有包版本，例如 2.20.0。该插入符号表示您的包可以使用与 2.20.0 兼容的任何版本。在语义版本的意义上，兼容性意味着新的次要或补丁版本被认为是有效的，因为它们是向后兼容的：

![图 2.5：将次要和补丁版本视为有效的语义格式](img/C14587_02_05.jpg)

###### 图 2.5：将次要和补丁版本视为有效的语义格式

与 2.20.0 兼容的版本是 2.21.0 或 2.21.5，甚至是 2.150.47！

偶尔，您可能希望更新您的软件包版本，以提高安全性或转移到具有解决某些依赖项中出现的问题的版本。这就是为什么 npm 为您安装的软件包版本添加了插入符号的原因。使用一个命令，您可以将所有依赖项更新为更新的兼容版本。

例如，很久以前启动的命令行应用程序使用的是 commander 的 2.0.0 版本。当开发人员运行`install`命令时，他们在`package.json`文件中得到了 2.0.0 版本。几年后，他们回过头来注意到 commander 中存在一些安全漏洞。他们只需运行`npm update`命令来解决这个问题：

```js
$ npm update
+ commander@2.20.0
added 1 package from 1 contributor and audited 1 package in 0.32s
found 0 vulnerabilities
```

大多数情况下，开发人员遵循语义版本控制规范，并不会在次要或补丁版本更改时进行破坏性更改。但是，随着项目的增长，依赖项的数量很快就会达到成千上万，破坏性更改或兼容性问题的概率呈指数级增长。

为了帮助您在出现复杂的依赖树时，npm 还会生成一个`package-lock.json`文件。该文件包含了您的`node_modules`目录中的软件包的表示，就像您上次更改依赖包时一样。当您使用`install`命令安装新依赖项或使用`update`命令更新版本时，就会发生这种情况。

`package-lock.json`文件应该与您的其他代码一起检查，因为它跟踪您的依赖树，并且对于调试复杂的兼容性问题非常有用。另一方面，`node_modules`应该始终添加到您的`.gitignore`文件中，因为 npm 可以使用来自您的`package.json`和`package-lock.json`文件的信息随时重新创建该文件夹，并从 npm 存储库下载包。

除了`dependencies`部分，您的`package.json`文件还可以包含一个`devDependencies`部分。这个部分是开发人员在构建或测试包时使用的依赖项，但其他人不需要。这可以包括诸如`babel`之类的工具来`转译`代码，或者诸如`jest`之类的测试框架。

`devDependencies`中的依赖项在其他包使用时不会被拉取。一些框架，如 Webpack 或`Parcel.js`，也有一个生产模型，将在创建最终捆绑包时忽略这些依赖项。

### npm 脚本

当您运行`npm init`命令时，创建的`package.json`文件中将包含一个`scripts`部分。默认情况下，会添加一个测试脚本。它看起来像这样：

```js
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1"
},
```

脚本可用于运行开发人员在处理软件包时可能需要的任何类型的命令。脚本的常见示例包括测试、linting 和其他代码分析工具。还可以有脚本来启动应用程序或从命令行执行其他任何操作。

要定义一个脚本，您需要在`scripts`部分添加一个属性，其中值是将要执行的脚本，如下所示：

```js
"scripts": {
  "myscript": "echo 'Hello world!'"
},
```

上述代码创建了一个名为`myscript`的脚本。当调用时，它将打印文本“Hello World!”。

要调用一个脚本，您可以使用`npm run`或 run-script 命令，传入脚本的名称：

```js
$ npm run myscript
> sample_scripts@1.0.0 myscript .../Lesson02/sample_scripts
> echo 'Hello World!'
Hello World!
```

npm 将输出正在执行的所有细节，以让您知道它在做什么。您可以使用`--silent`（或`-s`）选项要求它保持安静：

```js
$ npm run myscript --silent
Hello World!
$ npm run myscript -s
Hello World!
$ npm run-script myscript -s
Hello World!
```

关于脚本的一个有趣的事情是，您可以使用前缀“pre”和“post”在设置和/或清理任务之前和之后调用其他脚本。以下是这种用法的一个例子：

```js
"scripts": {
  "preexec": "echo 'John Doe' > name.txt",
  "exec": "node index.js",
  "postexec": "rm -v name.txt"
}
```

`index.js`是一个 Node.js 脚本，它从`name.txt`文件中读取名称并打印一个 hello 消息。`exec`脚本将执行`index.js`文件。在执行之前和之后，将自动调用预和后`exec`脚本，创建和删除`name.txt`文件（在 Windows 中，您可以使用`del`命令而不是`rm`）。运行 exec 脚本将产生以下输出：

```js
$ ls
index.js package.json
$ npm run exec
> sample_scripts@1.0.0 preexec ../Lesson02/sample_scripts
> echo 'John Doe' > name.txt
> sample_scripts@1.0.0 exec ../Lesson02/sample_scripts
> node index.js
Hello John Doe!
> sample_scripts@1.0.0 postexec ../Lesson02/sample_scripts
> rm -v name.txt
name.txt
$ ls
index.js        package.json
```

您可以看到，在调用 exec 脚本之前，`name.txt`文件不存在。调用`preexec`脚本，它将创建带有名称的文件。然后调用 JavaScript 并打印 hello 消息。最后，调用`postexec`脚本，它将删除文件。您可以看到，在 npm 执行完成后，`name.txt`文件不存在。

npm 还带有一些预定义的脚本名称。其中一些是 published，install，pack，test，stop 和 start。这些预定义名称的优势在于您不需要使用`run`或`run-script`命令；您可以直接按名称调用脚本。例如，要调用由`npm init`创建的默认测试脚本，只需调用`npm test`：

```js
$ npm test
> sample_scripts@1.0.0 test .../Lesson02/sample_scripts
> echo "Error: no test specified" && exit 1
Error: no test specified
npm ERR! Test failed.  See above for more details.
```

在这里，您可以看到它失败了，因为它有一个`exit 1`命令，这使得 npm 脚本的执行失败，因为任何以非零状态退出的命令都会立即使调用停止。

`start`是一个广泛使用的脚本，用于启动本地前端开发的 Web 服务器。前面代码中的 exec 示例可以重写为以下形式：

```js
"scripts": {
  "prestart": "echo 'John Doe' > name.txt",
  "start": "node index.js",
  "poststart": "rm -v name.txt"
}
```

然后，只需调用`npm start`即可运行：

```js
$ npm start
> sample_scripts@1.0.0 prestart .../Lesson02/sample_scripts
> echo 'John Doe' > name.txt
> sample_scripts@1.0.0 start .../Lesson02/sample_scripts
> node index.js
Hello John Doe!
> sample_scripts@1.0.0 poststart .../Lesson02/sample_scripts
> rm -v name.txt
name.txt
```

#### 注意

编写 npm 脚本时要牢记的一件重要事情是是否有必要使它们独立于平台。例如，如果您正在与一大群开发人员一起工作，其中一些人使用 Windows 机器，另一些人使用 Mac 和/或 Linux，那么在 Windows 中编写的脚本可能会在 Unix 世界中失败，反之亦然。JavaScript 是这种情况的完美用例，因为 Node.js 为您抽象了平台依赖性。

正如我们在上一章中看到的，有时我们想从网页中提取数据。在那一章中，我们使用了一些 JavaScript，它是从开发者工具控制台选项卡中注入到页面中的，这样就不需要为此编写应用程序。现在，您将编写一个 Node.js 应用程序来做类似的事情。

### 活动 3：创建一个用于解析 HTML 的 npm 包

在这个活动中，您将使用 npm 创建一个新的包。然后，您将编写一些 Node.js 代码来使用名为`cheerio`的库加载和解析 HTML 代码。有了加载的 HTML，您将查询和操作它。最后，您将打印操作后的 HTML 以查看结果。

执行的步骤如下：

1.  使用 npm 在新文件夹中创建一个新包。

1.  使用`npm install`（[`www.npmjs.com/package/cheerio`](https://www.npmjs.com/package/cheerio)）安装一个名为`cheerio`的库。

1.  创建一个名为`index.js`的新条目文件，并在其中加载`cheerio`库。

1.  创建一个变量，用于存储*第一章，JavaScript，HTML 和 DOM*中第一个示例的 HTML（文件可以在 GitHub 上找到：[`github.com/TrainingByPackt/Professional-JavaScript/blob/master/Lesson01/Example/sample_001/sample-page.html`](https://github.com/TrainingByPackt/Professional-JavaScript/blob/master/Lesson01/Example/sample_001/sample-page.html)）。

1.  使用 cheerio 加载和解析 HTML。

1.  在加载的 HTML 中的`div`中添加一个带有一些文本的段落元素。

1.  使用 cheerio，迭代当前页面中的所有段落，并将它们的内容打印到控制台。

1.  打印控制台的操作版本。

1.  运行您的应用程序。

输出应该看起来像下面这样：

![图 2.6：从 node.js 调用应用程序后的预期输出](img/C14587_02_06.jpg)

###### 图 2.6：从 Node.js 调用应用程序后的预期输出

#### 注意

此活动的解决方案可在第 588 页找到。

在本活动中，你使用 npm init 命令创建了一个 Node.js 应用程序。然后，你导入了一个 HTML 解析库，用它来操作和查询解析后的 HTML。在下一章中，我们将继续探索技术，帮助我们更快地抓取网页，并且我们将实际应用于一个网站。

## 总结

在本章中，我们了解了 Node.js 是什么，以及它的单线程、异步、事件驱动的编程模型如何用于构建简单高效的应用程序。我们还学习了 nvm 以及如何管理多个 Node.js 版本。然后，我们学习了 npm，并在我们的 Node.js 应用程序中使用了外部库。最后，我们学习了 npm 脚本以及与其相关的一些基本概念。

为了帮助你理解本章学到的内容，你可以去 npm 仓库，找一些项目，探索它们的代码库。了解 npm、Node.js 以及存在的包和库的最佳方法是探索其他人的代码，看看他们是如何构建的，以及他们使用了哪些库。

在下一章中，我们将探索 Node.js 的 API，并学习如何使用它们来构建一个真正的网页抓取应用程序。在未来的章节中，你将学习如何使用 npm 脚本和包来通过 linting 和自动化测试来提高代码质量。
