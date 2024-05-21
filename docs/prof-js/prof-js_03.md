# 第四章：*第三章*

# Node.js API 和 Web 抓取

## 学习目标

在本章结束时，您将能够：

+   使用全局对象实现 Node.js 应用程序

+   创建可读和可写流

+   使用异步和同步 API 读写文件

+   使用 http 模块创建静态和动态 Web 服务器

+   使用 http/https 模块从网站下载内容

+   查询和提取解析后的 HTML 内容中的数据

在本章中，我们将学习全局对象和函数。然后，我们将学习如何使用 http 模块编写高效的 Web 服务器，包括静态和动态的。最后，我们将使用 http 和 https 模块来抓取网页并从中提取数据。

## 介绍

从一开始，Node.js 就被创建为第一代 HTTP 服务器的每个请求模型的替代方案。Node.js 的事件循环和异步特性使其非常适合需要为大量并发客户端提供高吞吐量的 I/O 密集型服务器。因此，它配备了强大且易于使用的 API，可以直接构建 HTTP 服务器。

在上一章中，我们讨论了 Node.js 和 NPM 是什么以及它们是如何工作的。在本章中，您将了解 Node.js 中每个脚本都可以使用的基本全局对象。您将学习可读和可写流，以及如何使用它们来异步读写文件。您还将学习如何使用同步文件系统 API 来读写文件。

在最后几节中，您将学习如何使用 HTTP 模块来编写 Web 服务器和发起 HTTP 请求。您将构建一个静态和一个动态 Web 服务器。然后，您将学习 Web 抓取的基础知识，以及如何使用它来从网站中提取数据。

## 全局对象

Node.js 执行上下文包含一些**全局**变量和函数，可以在任何脚本中的任何地方使用。其中最常用的是`require`函数，因为它是帮助您加载其他模块并访问来自 Node.js API 的非全局函数、类和变量的函数。

您一定注意到了在上一章中使用了这个函数，当我们从您的应用程序中安装的包中加载`commander`模块时：

```js
const program = require('commander');
```

它接收一个参数，这个参数是一个表示您想要加载的模块的 ID 的字符串，并返回模块的内容。内部模块，比如我们将在本章讨论的模块，以及从 npm 安装的包中加载的模块，都可以直接通过它们的名称来识别，比如 commander、fs 和 http。在*第五章，模块化 JavaScript*中，您将学习如何创建自己的模块，以及如何使用这个函数来加载它们。

另一个重要且广泛使用的全局对象是控制台。就像在 Chrome 开发者工具中一样，控制台可以用来使用标准输出和标准错误将文本打印到终端。它也可以用来将文本打印到文件进行日志记录。

到目前为止，您已经多次使用了控制台，比如在上一章的最后一个练习中，您打印了以下操作过的 HTML：

```js
console.log(html);
```

控制台不仅仅只有`log`函数。让我们更深入地了解一些它的应用。

当您想要将一些文本打印到控制台时，您可以使用以下任何一个函数：`debug`、`error`、`info`和`warn`。它们之间的区别在于文本的输出位置。当您使用`debug`和`info`方法时，文本将被打印到标准输出。对于`warn`和`error`，消息将被打印到标准错误。

确保您在`index.js`中有以下代码：

```js
console.debug('This will go to Standard Output');
console.info('This will also go to Standard Output');
console.warn('This will go to standard error');
console.error('Same here');
```

现在，运行脚本并重定向到不同的文件，然后打印它们的内容：

```js
$ node index.js > std.out 2> err.out
$ cat std.out 
This will go to Standard Output
This will also go to Standard Output
$ cat err.out 
This will go to standard error
Same here
```

所有前面的函数以及 log 函数都可以根据需要格式化文本，方法是提供额外的参数和格式字符串。您可以在`util.format`函数文档中阅读更多关于格式字符串的信息：[`nodejs.org/dist/latest-v12.x/docs/api/util.html#util_util_format_format_args`](https://nodejs.org/dist/latest-v12.x/docs/api/util.html#util_util_format_format_args)。如果愿意，您也可以使用反引号：

```js
const whatILike = 'cheese';
console.log('I like %s', whatILike);
console.log(`I like ${whatILike}`);
```

输出将如下所示：

```js
I like cheese
I like cheese
```

如果需要有条件地打印一些文本，可以使用`assert`。Assert 可用于检查条件是否为真。如果为假，则它将使用`console.warn`打印文本，并解释断言失败的原因。如果为真，则不会打印任何内容。以下是一个示例：

```js
console.assert(1 == 1, 'One is equal to one');
console.assert(2 == 1, 'Oh no! One is not equal to two');
```

这将只输出以下内容：

```js
Assertion failed: Oh no! One is not equal to two
```

`trace`函数用于标识输出的源文件和行。它接收与 log 和其他函数相同的参数，但它还将打印日志语句的堆栈跟踪；也就是调用发生的文件名和行：

```js
console.trace('You can easily find me.');
```

这将打印以下内容：

```js
Trace: You can easily find me.
    at Object.<anonymous> (.../Lesson03/sample_globals/console.js:14:9)
    at Module._compile (internal/modules/cjs/loader.js:776:30)
    at Object.Module._extensions.js (internal/modules/cjs/loader.js:787:10)
    at Module.load (internal/modules/cjs/loader.js:653:32)
    at tryModuleLoad (internal/modules/cjs/loader.js:593:12)
    at Function.Module._load (internal/modules/cjs/loader.js:585:3)
    at Function.Module.runMain (internal/modules/cjs/loader.js:829:12)
    at startup (internal/bootstrap/node.js:283:19)
    at bootstrapNodeJSCore (internal/bootstrap/node.js:622:3)
```

如果您有一组数据并希望将其显示为表格，可以使用 table 方法。它接收两个参数：表格数据和您希望在表格中看到的属性。例如，考虑以下表格数据（对象数组）：

```js
const myTable = [
  { name: 'John Doe', age: 10 },
  { name: 'Jane Doe', age: 17 },
];
```

您可以通过将数据传递给`console.table`来打印所有列：

```js
console.table(myTable);
```

这将给我们以下输出：

![图 3.1：console.table 函数的输出](img/C14587_03_01.jpg)

###### 图 3.1：console.table 函数的输出

或者，您可以传递要显示的属性名称列表：

```js
console.table(myTable, ['name']);
```

以下是前面代码的输出：

![](img/C14587_03_02.jpg)

###### 图 3.2：当传递要打印的属性列表时，console.table 的输出

您还可以使用`console`来计算代码中特定部分运行所需的时间。为此，您可以使用`time`和`timeEnd`方法，如下例所示：

```js
console.time();
blockFor2Seconds();
console.timeEnd();
```

这将输出以下内容：

```js
default: 2000.911ms
```

您还可以为计时器命名，并同时使用多个计时器：

```js
console.time('Outer');
console.time('Inner');
blockFor2Seconds();
console.timeEnd('Inner');
console.timeEnd('Outer');
```

这将输出以下内容：

```js
Inner: 2000.304ms
Outer: 2000.540ms
```

有时，您想知道脚本是从哪里加载的，或者文件的完整路径是什么。为此，每个脚本都有两个全局变量：`__filename`和`__dirname`（两个下划线，然后是文件名/目录名）。示例如下：

```js
console.log(`This script is in: ${__dirname}`);
console.log(`The full path for this file is: ${__filename}`);
```

这将输出以下内容：

```js
This script is in: /.../Lesson03/sample_globals
The full path for this file is: /.../Lesson03/sample_globals/dir_and_filename.js
```

在浏览器中，当您想要在将来的某个时间执行特定函数或定期执行时，可以分别使用`setTimeout`和`setInterval`。这些函数也在 Node.js 执行上下文中可用，并且与在浏览器中的工作方式相同。

您可以通过将回调函数传递给它以及您希望它在未来的毫秒数中执行的时间量来安排代码在未来的某个时间执行：

```js
const start = Date.now();
setTimeout(() => {
  console.log('I'm ${Date.now() - start}ms late.');
}, 1000);
```

在浏览器中，`setTimeout`返回一个定时器 ID，这是一个整数，除了通过`clearTimeout`函数取消定时器外，不能做更多事情。在 Node.js 中，`setTimeout`返回一个`Timeout`对象，它本身具有一些方法。一个有趣的方法是`refresh`方法，它将定时器的开始时间重置为当前时间，并重新开始计时，就好像它是在那一刻被安排的一样。看看下面的示例代码：

```js
const secondTimer = setTimeout(() => {
  console.log(`I am ${Date.now() - start}ms late.');
}, 3000);
setTimeout(() => {
  console.log(`Refreshing second timer at ${Date.now() - start}ms`);
  secondTimer.refresh();
}, 2000);
```

这打印以下内容：

```js
Refreshing second timer at 2002ms
I am 5004ms late.
```

从输出中可以看出，即使`secondTimer`被安排在未来 3 秒运行，它实际上是在未来 5 秒运行。这是因为第二个`setTimeout`设置为 2 秒，刷新了它，重新从那个时间开始计时，将 2 秒添加到 3 秒计时器上。

如前所述，您可以使用`Timeout`实例使用`clearTimeout`函数取消定时器。以下代码是此示例：

```js
const thirdTimer = setTimeout(() => {
  console.log('I am never going to be executed.');
}, 5000);
setTimeout(() => {
  console.log('Cancelling third timer at ${Date.now() - start}ms');
  clearTimeout(thirdTimer);
}, 2000);
```

此代码的输出将如下所示：

```js
Cancelling third timer at 2007ms
```

`setTimeout`只执行一次。您可以使用`setInterval`每隔一段时间执行特定任务。`setInterval`还返回一个`Timeout`实例，可以使用`clearInterval`取消定时器。以下示例设置了一个定时器，每秒运行一次，并跟踪它运行的次数。在一定数量的执行之后，它会取消定时器：

```js
let counter = 0;
const MAX = 5;
const start = Date.now();
const timeout = setInterval(() => {
  console.log(`Executing ${Date.now() - start}ms in the future.`);
  counter++
  if (counter >= MAX) {
    console.log(`Ran for too long, cancelling it at ${Date.now() - start}ms`);
    clearInterval(timeout);
  }
}, 1000);
```

此代码的输出看起来像以下内容：

```js
Executing 1004ms in the future.
Executing 2009ms in the future.
Executing 3013ms in the future.
Executing 4018ms in the future.
Executing 5023ms in the future.
Ran for too long, cancelling it at 5023ms
```

在浏览器中，我们有一个称为 window 的全局对象，代表浏览器。在 Node.js 中，我们有 process，它代表当前运行的应用程序。通过它，我们可以访问传递给应用程序的参数，包括标准输入和输出以及有关进程的其他信息，例如版本或进程 ID。

要访问传递给进程的参数，可以使用全局变量 process 的`argv`属性。`argv`是一个包含每个参数的数组。它包括 Node.js 二进制文件的路径和脚本的完整路径作为前两个元素。之后，所有其他额外的参数都被传递进来。

以下代码将打印传入的所有参数，每个参数一行：

```js
console.log(`Arguments are:\n${process.argv.join('\n')}`);
```

让我们来看看这个单行应用程序的一些示例输出。

无额外参数：

```js
$ node argv.js 
Arguments are:
/usr/local/bin/node
/Users/visola/git/Professional-JavaScript/Lesson03/sample_globals/argv.js
```

许多参数一个接一个地分开：

```js
$ node argv.js this is a test
Arguments are:
/usr/local/bin/node
/Users/visola/git/Professional-JavaScript/Lesson03/sample_globals/argv.js
this
is
a
test
```

一个参数都在一个字符串中：

```js
$ node argv.js 'this is a test'
Arguments are:
/usr/local/bin/node
/Users/visola/git/Professional-JavaScript/Lesson03/sample_globals/argv.js
this is a test
```

在上一章中，我们使用了`commander`库来解析命令行参数。在配置`commander`时，对它的最后一次调用是`parse(process.argv)`，这使`commander`可以访问传入的所有选项：

```js
program.version('0.1.0')
  .option('-b, --add-bootstrap', 'Add Bootstrap 4 to the page.')
  .option('-c, --add-container', 'Adds a div with container id in the body.')
  .option('-t, --title [title]', 'Add a title to the page.')
  .parse(process.argv);
```

process 变量扮演的另一个重要角色是访问标准输入和输出。如果要向控制台打印内容，可以使用`stdout`和`stderr`。这两个属性是控制台中的`console.log`和所有其他方法在内部使用的。不同之处在于`stdout`和`stderr`在每次调用时不会在末尾添加新行，因此如果希望每个输出都进入自己的行，您必须自己添加新行：

```js
process.stdout.write(`You typed: '${text}'\n`);
process.stderr.write('Exiting your application now.\n');
```

这是两个示例，打印出以换行结束的内容。在大多数情况下，建议使用控制台，因为它可以提供一些额外的东西，例如日志级别和格式化。

如果要从命令行读取输入，可以使用`process.stdin`。`stdin`是一个流，我们将在下一节中更多地讨论。现在，您只需要知道流是基于事件的。这意味着当输入进来时，它将以数据事件的形式到达。要从用户那里接收输入，您需要监听该事件：

```js
process.stdin.addListener('data', (data) => {
  ...
});
```

当没有更多的代码需要执行时，事件循环将阻塞，等待标准输入的输入。当读取输入时，它将作为字节缓冲传递到回调函数中。您可以通过调用其`toString`方法将其转换为字符串，如下面的代码所示：

```js
const text = data.toString().trim();
```

然后，您可以像普通字符串一样使用它。以下示例应用程序演示了如何使用`stdout`、`stderr`和`stdin`从命令行请求用户输入： 

```js
process.stdout.write('Type something then press [ENTER]\n');
process.stdout.write('> ');
process.stdin.addListener('data', (data) => {
  const text = data.toString().trim();
  process.stdout.write('You typed: '${text}'\n');
  if (text == 'exit') {
    process.stderr.write('Exiting your application now.\n');
    process.exit(0);
  } else {
    process.stdout.write('> ');
  }
});
```

以下代码显示了在运行应用程序并输入一些单词，按*Enter*，然后输入“exit”以退出应用程序后的样子：

```js
$ node read_input.js 
Type something then press [ENTER]
> test
You typed: 'test'
> something
You typed: 'something'
> exit
You typed: 'exit'
Exiting your application now.
```

在前面的代码中，您可以看到当用户输入“exit”时，它执行应用程序代码的特殊分支，调用`process.exit`，这是一个退出整个进程并返回指定退出代码的函数。

### 练习 11：创建任务提醒应用程序

在这个练习中，我们将创建一个任务提醒应用程序。现在我们已经学会了如何使用全局变量 process 与用户进行交互，还学会了如何创建定时器，让我们编写一个应用程序，利用这些新技能来管理命令行中的提醒。

应用程序将接收用户输入并收集信息以构建提醒。它将使用消息、时间单位和一定的时间。应用程序的输入将分阶段提供。每个阶段都会要求用户输入一些内容，收集它，验证它，然后设置一个变量的值以进入下一个阶段。

执行以下步骤完成此练习：

1.  在一个空文件夹中，使用`npm init`创建一个新的包，并创建一个名为`index.js`的文件。在`index.js`文件中，我们将首先添加一些常量和变量，用于存储创建计时器的状态：

```js
// Constants to calculate the interval based on time unit
const timeUnits = ['Seconds', 'Minutes', 'Hours'];
const multipliers = [1000, 60 * 1000, 3600 * 1000];
// Variables that will store the application state
let amount = null;
let message = null;
let timeUnit = null;
// Alias to print to console
const write = process.stdout.write.bind(process.stdout);
```

1.  接下来，我们将添加应用程序的核心函数。该函数如下所示：

```js
function processInput(input) {
  // Phase 1 - Collect message
  if (message == null) {
    askForMessage(input);
    input = null;
  }
  // Phase 2 - Collect time unit
  if (message != null && timeUnit == null) {
    askForTimeUnit(input);
    input = null;
  }
  // Phase 3 - Collect amount of time
  if (timeUnit != null && amount == null) {
    askForAmount(input);
  }
}
```

该函数处理用户的所有输入，根据当前状态的一组条件进行处理，根据已经可用的变量。处理输入后，将其设置为 null，以便可以执行下一个阶段。

前面的函数调用了一些尚不存在的函数：`askForMessage`，`askForTimeUnit`和`askForAmount`。这些函数负责验证输入并根据每个阶段设置变量，以便代码可以进入下一个阶段。

1.  添加一些细节到`askForMessage`函数。该函数首先检查输入是否为 null，这意味着它正在首次更改阶段。这意味着它需要为用户打印输入提示。

代码如下所示：

```js
function askForMessage(input) {
  if (input == null) {
    write('What do you want to be reminded of? > ');
    return;
  }
  if (input.length == 0) {
    write('Message cannot be empty. Please try again. > ');
    return;
  }
  message = input;
}
```

如果输入不是`null`，这意味着用户已经为当前状态输入了信息，需要进行验证。如果验证失败，打印更多信息并等待下一个输入。

如果输入有效，则设置当前状态的变量，这种情况下是`message`，这将使代码进入下一个阶段。

1.  接下来，我们创建`askForTimeUnit`函数，这是处理代码的下一个阶段的函数。该函数使用第一步列出的常量来打印支持的时间单位，并让用户选择一个。它的工作方式类似于`askForMessage`函数：`prompt`，`validate`和`set value`：

```js
function askForTimeUnit(input) {
  if (input == null) {
    console.log('What unit?');
    timeUnits.forEach((unit, index) => console.log('${index + 1} - ${unit}') );
    write('> ');
    return;
  }
  const index = parseInt(input, 10);
  if (isNaN(index) || index <= 0 || index > timeUnits.length) {
    write(`Sorry, '${input}' is not valid. Please try again. > `);
    return;
  }
 timeUnit = index - 1;
  console.log(`Picked: ${timeUnits[timeUnit]}`);
}
```

1.  最后，我们创建`askForAmount`函数，处理最后一个阶段。该函数提示用户输入一定的时间来创建计时器。与之前一样，它有三个部分：`prompt`，`validate`和`set value`：

```js
function askForAmount(input) {
  if (input == null) {
    write(`In how many ${timeUnits[timeUnit]}? > `);
    return;
  }
  const number = parseInt(input, 10);
  if (isNaN(number)) {
    write(`Sorry, '${input}' is not valid. Try again. > `);
    return;
  }
  amount = number;
  setTimerAndRestart();
}
```

1.  在`askForAmount`函数的末尾，它调用`setTimerAndRestart`函数。让我们创建该函数，它创建计时器并重置所有状态，以便循环可以重新开始，并且用户可以创建新的计时器。该`setTimerAndRestart`函数如下所示：

```js
function setTimerAndRestart() {
  const currentMessage = message;
  write(`Setting reminder: '${message}' in ${amount} ${unit} from now.\n`);
  let timerMessage = `\n\x07Time to '${currentMessage}'\n> `;
  setTimeout(() => write(timerMessage), amount * multipliers[timeUnit]);
  amount = message = timeUnit = null;
  askForMessage();
}
```

这里的一个重要部分是特殊字符`\x07`。这将导致您的终端发出哔哔声，然后打印消息中设置的文本。此外，文本经过特殊格式化，在开头和结尾都有换行，以便不会太大干扰工具的使用，因为计时器将在用户继续使用应用程序的同时打印。

1.  应用程序的最后一部分需要在标准输入中注册数据事件的监听器，并通过询问用户消息来启动循环：

```js
process.stdin.on('data', (data) => processInput(data.toString().trim()));
askForMessage();
```

1.  现在，您可以从终端运行应用程序，设置一些提醒，并在计时器到期时听到它发出哔哔声：

![](img/C14587_03_03.jpg)

###### 图 3.3：运行应用程序后的输出

您会注意到退出应用程序的唯一方法是同时按下*Ctrl+C*键发送中断信号。作为额外的挑战，尝试添加一些代码，以创建一个退出点，使用户可以以更友好的方式退出。

处理用户输入对于每个命令行应用程序都是至关重要的。在这个练习中，您学会了如何掌握 Node.js 的异步特性，以便处理一组复杂的输入，引导用户在创建提醒的决策过程中。

## 文件系统 API

在上一节中，我们了解了在 Node.js 执行上下文中可用的全局变量。在本节中，我们将了解文件系统 API，这些 API 用于访问文件和目录，读取和写入文件等等。

但在我们深入研究文件系统 API 之前，我们需要了解流。在 Node.js 中，流是表示流数据的抽象接口。在上一节中，我们使用了标准 I/O，并简要提到它们是流，所以让我们详细了解它们。

流可以是可读的、可写的，或者两者兼有。它们是事件发射器，这意味着要接收数据，你需要注册事件监听器，就像我们在上一节中对标准输入所做的那样：

```js
process.stdin.addListener('data', (data) => {
  ...
});
```

在下一节中，我们将继续建立对前几节的理解，并看到流被用作抽象来表示数据可以流经的所有东西，包括标准输入和输出、文件和网络套接字。

为了开始理解这是如何工作的，我们将编写一个应用程序，通过使用文件系统包中的`createReadStream`来读取自己的代码。要使用文件系统 API，我们需要导入它们，因为它们不是全局可用的：

```js
const fs = require('fs');
```

然后，我们可以创建一个指向脚本文件本身的可读流：

```js
const readStream = fs.createReadStream(__filename);
```

最后，我们注册流的事件，以便了解发生了什么。读取流有四个你应该关心的事件：ready、data、close 和 error。

Ready 会告诉你文件何时准备好开始读取，尽管当你创建一个指向文件的可读流时，它会在文件可用时立即开始读取文件。

数据，正如我们在标准输入中看到的，将通过传递从流中读取的数据作为字节缓冲区来调用。缓冲区需要通过调用它的`toString`方法或与另一个字符串连接来转换为字符串。

当所有字节都被读取完毕，流不再可读时，将调用 close。

如果在从流中读取时发生错误，将调用`Error`。

以下代码演示了我们如何通过在控制台打印内容来注册事件：

```js
readStream.on('data', (data) => console.log(`--data--\n${data}`));
readStream.on('ready', () => console.log(`--ready--`));
readStream.on('close', () => console.log(`--close--`));
```

该应用程序的输出如下：

```js
$ node read_stream.js 
--ready--
--data--
const fs = require('fs');
const readStream = fs.createReadStream(__filename);
readStream.on('data', (data) => console.log(`--data--\n${data}`));
readStream.on('ready', () => console.log(`--ready--`));
readStream.on('close', () => console.log(`--close--`));
--close--
```

现在你知道如何读取文件和使用读取流，让我们更详细地了解可写流。你在上一节中看到了一些它们的用法，因为标准输出是一个可写流：

```js
process.stdout.write('You typed: '${text}'\n');
process.stderr.write('Exiting your application now.\n');
```

`write`方法是可写流中最常用的方法。如果你想创建一个写入文件的可写流，你只需要传递文件名即可：

```js
const fs = require('fs');
const writable = fs.createWriteStream('todo.txt');
```

然后，你可以开始写入它：

```js
writable.write('- Buy milk\n');
writable.write('- Buy eggs\n');
writable.write('- Buy cheese\n');
```

不要忘记在最后添加换行符，否则所有内容将打印在同一行。

在写入文件完成后，调用`end`方法来关闭它：

```js
writable.end();
```

可写流也有你可以监听的事件。最重要的两个事件是`error`和`close`。当写入流时发生错误时，将触发`error`事件。当流关闭时，将调用`close`事件。还有`finish`事件，当调用`end`方法时将触发。以下代码是可以在 GitHub 上找到的示例代码的最后部分：[`github.com/TrainingByPackt/Professional-JavaScript/blob/master/Lesson03/sample_filesystem/write_stream.js`](https://github.com/TrainingByPackt/Professional-JavaScript/blob/master/Lesson03/sample_filesystem/write_stream.js)：

```js
writable.on('finish', () => console.log("-- finish --"));
writable.on('close', () => console.log("-- close --"));
```

运行应用程序后，你会看到它会创建`todo.txt`文件，并在其中包含预期的内容：

```js
$ node write_stream.js 
-- finish --
-- close --
$ cat todo.txt 
- Buy milk
- Buy eggs
- Buy cheese
```

#### 注意

创建一个指向文件的流将默认创建一个覆盖文件内容的流。要创建一个追加到文件的流，你需要传递一个带有"a"标志的选项对象，如追加一样：

```js
const writable = fs.createWriteStream('todo.txt', { flags: 'a'});
```

关于流的另一个有趣的事情是你可以将它们连接起来。这意味着你可以将读取流中的所有字节发送到写入流中。你可以使用以下代码轻松地将一个文件的内容复制到另一个文件中：

```js
const fs = require('fs');
fs.createReadStream('somefile.txt')
  .pipe(fs.createWriteStream('copy.txt'));
```

除了读写文件外，文件系统 API 还提供了方法，可以列出目录中的文件，检查文件状态，监视目录或文件的更改，复制，删除，更改文件权限等。

在处理文件系统操作时，你必须记住这些操作是异步的。这意味着所有操作都会接收一个回调函数，在操作完成时调用。例如，当创建目录时，你可以编写以下代码：

```js
const firstDirectory = 'first';
fs.mkdir(firstDirectory, (error) => {
  if (error != null) {
    console.error(`Error: ${error.message}`, error);
    return;
  }
  console.log(`Directory created: ${firstDirectory}`);
});
```

如果尝试创建目录时出现问题，例如目录已经存在，回调函数会接收一个错误参数。第一次运行代码会成功：

```js
$ node directories_and_files.js
...
Directory created: first
```

但是当第二次运行时，它会失败，因为目录已经被创建了：

```js
$ node directories_and_files.js 
Error: EEXIST: file already exists, mkdir 'first' { [Error: EEXIST: file already exists, mkdir 'first'] errno: -17, code: 'EEXIST', syscall: 'mkdir', path: 'first' }
...
```

如果你想在刚刚创建的目录中创建一个文件，你需要在传递给`mkdir`的回调函数中创建文件。以下方式可能会失败：

```js
const firstDirectory = 'first';
fs.mkdir(firstDirectory, (error) => {
  ...
});
fs.writeFile(`${firstDirectory}/test.txt`, 'Some content', (error) => {
  console.assert(error == null, 'Error while creating file.', error);
});
```

当你尝试运行它时会发生这种情况：

```js
$ node directories_and_files.js 
Assertion failed: Error while creating file. { [Error: ENOENT: no such file or directory, open 'first/test.txt']
...
```

这是因为当调用`writeFile`时，目录可能还不存在。正确的做法是在传递给`mkdir`的回调函数中调用`writeFile`：

```js
const firstDirectory = 'first';
fs.mkdir(firstDirectory, (error) => {
  ...
  fs.writeFile(`${firstDirectory}/test.txt`, 'Some content', (error) => {
    console.assert(error == null, 'Error while creating file.', error);
  });
});
```

由于处理前面的异步调用很复杂，并且并非所有情况都需要高性能的异步操作，在文件系统模块中，几乎所有操作都包括相同 API 的同步版本。因此，如果你想在目录中创建一个文件并在其中创建一些内容，而在目录不存在时应用程序没有其他事情可做，你可以按照以下方式编写代码：

```js
const thirdDirectory = 'third';
fs.mkdirSync(thirdDirectory);
console.log(`Directory created: ${thirdDirectory}`);
const thirdFile = `${thirdDirectory}/test.txt`;
fs.writeFileSync(thirdFile, 'Some content');
console.log(`File created: ${thirdFile}`);
```

注意每个方法名称末尾的`Sync`单词。上述代码的输出如下：

```js
$ node directories_and_files.js 
Directory created: third
File created: third/test.txt
```

在 Node.js 10 中，文件系统模块还添加了基于 Promise 的 API。关于 Promise 和其他处理异步操作的技术将在后续章节中讨论，所以我们暂时跳过这部分。

现在你知道如何创建目录和读写文件数据，让我们继续下一个最常用的文件系统操作：列出目录。

要列出目录中的文件，可以使用`readdir`方法。传递给函数的回调函数如果在尝试读取目录时出现问题，将会接收到一个错误对象和一个文件名列表。以下代码将打印当前目录中所有文件的名称：

```js
fs.readdir('./', (error, files) => {
  if (error != null) {
    console.error('Error while reading directory.', error);
    return;
  }
  console.log('-- File names --');
  console.log(files.join('\n'));
});
```

这是一个示例输出：

```js
$ node list_dir.js 
-- File names --
.gitignore
copy_file.js
directories_and_files.js
first
list_dir.js
read_stream.js
second
third
write_stream.js
...
```

但有时，你不仅仅想要文件名。在这里，`readdir`函数接受一个选项对象，可以提供`withFileTypes`标志。如果传递了该标志，那么回调函数得到的不是文件名，而是一个包含有关文件的额外信息的`Dirents`数组，例如它是目录还是文件。以下示例将打印当前目录中的文件名，并根据它是目录还是文件分别添加(D)或(F)：

```js
fs.readdir('./', { withFileTypes: true }, (error, files) => {
  if (error != null) {
    console.error('Error while reading directory.', error);
    return;
  }
  console.log('-- File infos --');
  console.log(files.map(d => `(${d.isDirectory() ? 'D': 'F'}) ${d.name}`)
    .sort()
    .join('\n'));
});
```

示例输出如下：

```js
$ node list_dir.js 
...
-- File infos --
(D) first
(D) second
(D) third
(F) .gitignore
(F) copy_file.js
(F) directories_and_files.js
(F) list_dir.js
(F) read_stream.js
(F) write_stream.js
```

文件系统 API 的最后一个重要操作是如何检查文件状态。如果你只需要知道文件是否存在且可读，可以使用`access`函数，它接收文件路径和一组状态标志来检查。如果文件状态与指定的标志匹配，那么错误将不会传递给回调函数。让我们看一个例子：

```js
const fs = require('fs');
const filename = process.argv[2];
fs.access(filename, fs.constants.F_OK | fs.constants.R_OK, (error) => {
  if (error == null) {
    console.log('File exists and is readable');
  } else {
    console.log(error.message);
  }
});
```

在这个例子中，我们结合了两个标志，`F_OK`和`R_OK`。第一个检查文件是否存在，而第二个检查文件是否可读。你可以使用`|`（或）运算符组合多个标志。

执行上述代码后，如果文件存在，你会看到以下输出：

```js
$ node file_status.js test.txt 
File exists and is readable
```

如果文件不存在，那么你会看到以下输出：

```js
$ node file_status.js not.txt 
ENOENT: no such file or directory, access 'not.txt'
```

最后，如果文件存在但不可读，你将收到以下消息：

```js
$ node file_status.js not.txt 
EACCES: permission denied, access 'not.txt'
```

所有这些看起来很有趣，但如果你需要知道一个路径是文件还是目录，它是何时最后修改的等等，那么你需要使用`lstat`函数，它将返回一个 Stats 实例。Stats 包含了你需要了解的关于路径的一切。

以下示例检查路径是文件还是目录，它是何时创建和最后修改的，并将该信息打印到控制台：

```js
fs.lstat(filename, (statError, stat) => {
  if (statError != null) {
    console.error('Error while file status.', statError);
    return;
  }
  console.log(`Is file: ${stat.isFile()}`);
  console.log(`Is directory: ${stat.isDirectory()}`);
  console.log(`Created at: ${stat.birthtime}`);
  console.log(`Last modified at: ${stat.mtime}`);
});
```

这是一个示例输出：

```js
$ node file_status.js first/test.txt 
...
Is file: true
Is directory: false
Created at: Tue Aug 13 2019 20:39:37 GMT-0400 (Eastern Daylight Time)
Last modified at: Tue Aug 13 2019 21:26:53 GMT-0400 (Eastern Daylight Time)
```

Globs 是包含路径部分的字符串，通配符`*`代表。当你有两个`*`时，例如`**`，这意味着任何目录或子目录。一个简单的例子是在当前目录的任何子目录中搜索所有的`.txt`文件：

```js
$ search '**/*.txt'
```

### 练习 12：使用 Glob 模式通过目录搜索文件

在这个练习中，我们将创建一个应用程序，它将扫描目录树并根据 glob 搜索文件。为了实现这一点，我们将递归调用`readdir`函数的同步版本，并使用`commander`和`glob-to-regexp`模块来帮助我们处理用户的输入。

执行以下步骤完成这个练习：

1.  在一个空目录中，使用`npm` `init`开始一个新的应用程序，并添加一个`index.js`文件，这将是我们的入口点。

1.  安装我们将使用的两个外部模块：`commander`和`glob-to-regexp`。为此，执行`npm install`命令：

```js
$ npm install commander glob-to-regexp
npm notice created a lockfile as package-lock.json. You should commit this file.
+ glob-to-regexp@0.4.1
+ commander@3.0.0
added 2 packages from 2 contributors and audited 2 packages in 0.534s
found 0 vulnerabilities
```

1.  在`index.js`文件中，使用你喜欢的编辑器，在文件开头导入所有这个项目所需的模块：

```js
const fs = require('fs');
const globToRegExp = require('glob-to-regexp');
const join = require('path').join;
const program = require('commander');
```

我们已经知道了`fs`和 commander 模块。`globToRegExp`模块和`join`函数将在接下来的步骤中进行解释。

1.  初始化`counter`和`found`变量。这些将用于显示与正在执行的搜索相关的一些统计信息：

```js
let counter = 0;
let found = 0;
const start = Date.now();
```

1.  配置`commander`以接收 glob 作为参数，并为用户设置初始目录开始搜索的额外选项：

```js
     program.version('1.0.0')
  .arguments('<glob>')
  .option('-b, --base-dir <dir>', 'Base directory to start the search.', './')
  .parse(process.argv);
```

1.  在这个练习中，我们将使用递归函数来遍历目录树。`walkDirectory`函数调用`readdirSync`，并将`withFileTypes`标志设置为`true`。`walkDirectory`函数接收两个参数：要开始读取的路径和要为每个文件调用的回调函数。当找到一个目录时，它被传递给`walkDirectory`函数，以便递归继续：

```js
function walkDirectory(path, callback) {
  const dirents = fs.readdirSync(path, { withFileTypes: true });
  dirents.forEach(dirent => {
    if (dirent.isDirectory()) {
      walkDirectory(join(path, dirent.name), callback);
    } else {
      counter++;
      callback(join(path, dirent.name));
   }
  });
}
```

当找到一个文件时，路径被传递给回调函数，并且计数器被递增。在这里，我们使用`path.join`函数将文件名连接到父路径，以重建文件的整个路径。

1.  现在我们有了`walkDirectory`树函数，我们将验证传递给应用程序的参数：

```js
const glob = program.args[0];
if (typeof glob === 'undefined') {
  program.help();
  process.exit(-1);
}
```

1.  然后，我们使用`globToRegExp`模块将 glob 转换为`RegExp`，以便用于测试文件：

```js
const matcher = globToRegExp(program.args[0], { globstar: true });
```

1.  有了匹配器和遍历目录树函数，我们现在可以遍历目录树并测试我们找到的每个文件：

```js
walkDirectory(program.baseDir, (f) => {
  if (matcher.test(f)) {
    found++;
    console.log(`${found} - ${f}`);
  }
});
```

1.  最后，由于所有的代码都是同步执行的，在调用`walkDirectory`完成后，所有的目录和子目录都将被处理。现在，我们可以打印出我们找到的统计信息：

![图 3.4：找到的文件的统计信息](img/C14587_03_04.jpg)

###### 图 3.4：找到的文件的统计信息

你可以通过在父目录中开始执行搜索：

![图 3.5：在父目录中执行搜索](img/C14587_03_05.jpg)

###### 图 3.5：在父目录中执行搜索

在这个练习中，你学会了如何使用文件系统 API 来遍历目录树。你还使用了正则表达式来按名称过滤文件。

文件系统 API 为几乎每个应用程序提供了基础。学习如何同步和异步地使用它们对于后端世界中的任何事情都是至关重要的。在下一节中，我们将使用这些 API 来构建一个基本的 Web 服务器，以便向浏览器提供文件。

## HTTP API

起初，Node.js 的目标是取代使用传统的每个连接一个线程模型的旧 Web 服务器。在线程每请求模型中，服务器保持一个端口开放，当新连接进来时，它使用线程池中的一个线程或创建一个新线程来执行用户请求的工作。服务器端的所有操作都是同步进行的，这意味着当从磁盘读取文件或从数据库中读取记录时，线程会休眠。以下插图描述了这个模型：

![图 3.6：在线程每请求模型中，线程在 I/O 和其他阻塞操作发生时处于休眠状态](img/C14587_03_06.jpg)

###### 图 3.6：在线程每请求模型中，线程在 I/O 和其他阻塞操作发生时处于休眠状态

线程每请求模型的问题在于创建线程的成本很高，而当它们在有更多工作要做时处于休眠状态，这意味着资源的浪费。另一个问题是，当线程的数量高于 CPU 的数量时，它们开始失去并发的最宝贵的价值。

由于这些问题，使用线程每请求模型的 Web 服务器将拥有一个不够大的线程池，以便服务器仍然可以并行响应许多请求。并且因为线程数量是有限的，当并发用户发出请求的数量增加时，服务器会耗尽线程，用户现在必须等待：

![](img/C14587_03_07.jpg)

###### 图 3.7：当并发请求数量增加时，用户必须等待线程可用

Node.js 以其异步模型和事件循环，提出了这样一个观念：如果只有一个线程来执行工作并将阻塞和 I/O 操作移到后台，只有在数据可用于处理时才返回到它，那么您可以更加高效。当您需要进行数据密集型工作时，比如 Web 服务器，它主要从文件、磁盘和数据库中读取和写入记录时，异步模型变得更加高效。以下插图描述了这个模型：

![图 3.8：带有事件循环的异步模型](img/C14587_03_08.jpg)

###### 图 3.8：带有事件循环的异步模型

当然，这个模型并不是万能的，在高负载和大量并发用户的情况下，队列中的工作量会变得如此之大，以至于用户最终会开始相互阻塞。

现在您已经了解了异步模型的历史以及 Node.js 为什么实现它，让我们来构建一个简单的 hello world Web 服务器。在接下来的章节中，您将学习更多关于 REST API 以及如何使用一些库来帮助您做一些更高级的事情。现在，我们将使用 http 模块来构建一个返回"hello world"字符串的服务器。

要创建一个 HTTP 服务器，您可以使用 http 模块中的`createServer`函数。只需按照以下步骤即可：

```js
const http = require('http');
const server = http.createServer();
```

服务器由事件驱动，我们最感兴趣的事件是请求。当 HTTP 客户端连接到服务器并发起请求时，将触发此事件。我们可以使用一个接收两个参数的回调来监听此事件：

+   请求：客户端发送给服务器的请求。

+   响应：用于与客户端通信的响应对象。

响应是一个可写流，这意味着我们已经知道如何向其发送数据：通过调用`write`方法。但它还包含一个特殊的方法叫做`writeHead`，它将返回 HTTP 状态码和任何额外的标头。以下是将 hello world 字符串发送回客户端的示例：

```js
server.on('request', (request, response) => {
  console.log('Request received.', request.url);
  response.writeHead(200, { 'Content-type': 'text/plain' });
  response.write('Hello world!');
  response.end();
});
```

我们有了服务器和请求处理程序。现在，我们可以开始在特定端口上监听请求。为此，我们在服务器实例上调用`listen`方法：

```js
const port = 3000;
console.log('Starting server on port %d.', port);
console.log('Go to: http://localhost:%d', port);
server.listen(port);
```

此示例的代码可在 GitHub 上找到：[`github.com/TrainingByPackt/Professional-JavaScript/blob/master/Lesson03/sample_http/http_server.js`](https://github.com/TrainingByPackt/Professional-JavaScript/blob/master/Lesson03/sample_http/http_server.js)。

如果您通过运行此应用程序启动 hello world 服务器，您将在控制台中看到类似以下内容：

```js
$ node http_server.js 
Starting server on port 3000.
Go to: http://localhost:3000
```

如果您打开浏览器并转到指定路径，您将看到以下内容：

![图 3.9：Hello world web 服务器示例响应](img/C14587_03_09.jpg)

###### 图 3.9：Hello world web 服务器示例响应

您可以尝试访问其他路径，例如`http://localhost:3000/index.html`。结果将是相同的：

![图 3.10：Hello world 服务器始终以 Hello world 响应](img/C14587_03_10.jpg)

###### 图 3.10：Hello world 服务器始终以 Hello world 响应

如果您返回到运行服务器的控制台，您将看到类似以下内容：

```js
$ node http_server.js 
Starting server on port 3000.
Go to: http://localhost:3000
Request received. /
Request received. /favicon.ico
Request received. /index.html
Request received. /favicon.ico
```

您可以看到服务器正确地从浏览器接收到路径。但是，由于代码没有处理任何特殊情况，它只是返回 Hello world。客户端无论请求什么路径，始终会得到相同的结果。

### 练习 13：提供静态文件

我们已经学会了如何构建一个始终以相同字符串响应的 hello world web 服务器，无论客户端请求什么。在这个练习中，我们将创建一个 HTTP 服务器，从目录中提供文件。这种类型的服务器称为静态 HTTP 服务器，因为它只在目录中查找文件并将它们无修改地返回给客户端。

执行以下步骤完成此练习：

1.  在空目录中，使用`init`命令初始化一个新的 npm 应用程序，并向其添加一个`index.js`文件。还要使用`npm install`安装`mime`包。我们将使用此包确定我们将提供的文件的内容类型是什么：

```js
npm install mime
```

1.  让我们首先导入我们在这个项目中需要的所有模块：

```js
const fs = require('fs');
const http = require('http');
const mime = require('mime');
const path = require('path');
const url = require('url');
```

我们将使用`fs`模块从磁盘加载文件。http 模块将用于创建 HTTP 服务器和处理 HTTP 请求。`mime`模块是我们在上一步中安装的，将用于确定每个文件的内容类型。path 模块用于以平台无关的方式处理路径。最后，`url`模块用于解析 URL。

1.  为了知道我们将要提供哪些文件，我们将使用上一个练习中的`walkDirectory`函数扫描目录：

```js
function walkDirectory(dirPath, callback) {
  const dirents = fs.readdirSync(dirPath, { withFileTypes: true });
  dirents.forEach(dirent => {
    if (dirent.isDirectory()) {
      walkDirectory(path.join(dirPath, dirent.name), callback);
    } else {
      callback(path.join(dirPath, dirent.name));
    }
  });
}
```

1.  然后，我们将选择根目录，可以将其作为参数传递。否则，我们将假定它是我们运行脚本的目录：

```js
const rootDirectory = path.resolve(process.argv[2] || './');
```

1.  现在，我们可以扫描目录树并将所有文件的路径存储在`Set`中，这将使文件可用性检查的过程更快：

```js
const files = new Set();
walkDirectory(rootDirectory, (file) => {
 file = file.substr(rootDirectory.length);
  files.add(file);
});
console.log(`Found ${files.size} in '${rootDirectory}'...`);
```

1.  准备好提供文件列表后，我们将创建 HTTP 服务器实例：

```js
const server = http.createServer();
```

1.  启动请求处理程序函数：

```js
server.on('request', (request, response) => {
```

1.  在处理程序函数内部，将用户请求的内容解析为 URL。为此，我们将使用 url 模块，并从解析后的 URL 中获取指向客户端想要的文件的路径名：

```js
const requestUrl = url.parse(request.url);
const requestedPath = path.join(requestUrl.pathname);
```

1.  有了文件路径，我们将检查文件是否在之前收集的列表中，如果不在，则响应 404（未找到）错误消息，记录请求的结果并返回它：

```js
if (!files.has(requestedPath)) {
  console.log('404 %s', requestUrl.href);
  response.writeHead(404);
  response.end();
  return;
}
```

1.  如果文件在`Set`中，我们将使用 path 模块提取其扩展名，并使用`mime`模块解析内容类型。然后，我们将以 200（ok）错误消息响应，创建一个读取文件的流，并将其传输到响应中：

```js
  const contentType = mime.getType(path.extname(requestedPath));
  console.log('200 %s', requestUrl.href);
  response.writeHead(200, { 'Content-type': contentType });
  fs.createReadStream(path.join(rootDirectory, requestedPath))
    .pipe(response);
});
```

1.  处理程序函数到此为止。之后，我们可以通过选择一个端口来启动服务器，让用户知道那是什么，并调用 http 服务器中的监听方法：

```js
const port = 3000;
console.log('Starting server on port %d.', port);
console.log('Go to: http://localhost:%d', port);
server.listen(port);
```

1.  您可以通过运行以下命令来启动服务器：

```js
$ node .
Found 23 in '/Path/to/Folder'...
Starting server on port 3000.
o to: http://localhost:3000
```

1.  从另一个终端窗口，我们可以使用命令行 HTTP 客户端 curl 来调用我们的服务器并查看响应：

```js
$ curl -i localhost:3000/index.js
HTTP/1.1 200 OK
Content-type: application/javascript
Date: Fri, 16 Aug 2019 02:06:05 GMT
Connection: keep-alive
Transfer-Encoding: chunked
const fs = require('fs');
const http = require('http');
const mime = require('mime');
... rest of content here....
```

我们也可以从浏览器中进行相同操作：

![图 3.11：从浏览器中查看的静态 index.js 从我们的 HTTP 服务器提供的](img/C14587_03_11.jpg)

###### 图 3.11：从浏览器中查看的静态 index.js 从我们的 HTTP 服务器提供的

您也可以尝试使用一个不存在的文件来查看结果：

```js
$ curl -i localhost:3000/not_real.js
HTTP/1.1 404 Not Found
Date: Fri, 16 Aug 2019 02:07:14 GMT
Connection: keep-alive
Transfer-Encoding: chunked
```

从浏览器中，404 响应看起来像一个错误页面：

![图 3.12：当请求一个不存在的文件时，服务器会以 404 错误响应](img/C14587_03_12.jpg)

###### 图 3.12：当请求一个不存在的文件时，服务器会以 404 错误响应

在运行服务器的终端上，您可以看到它打印了有关正在提供的信息：

```js
$ node .
Found 23 in '/Path/to/Folder'...
Starting server on port 3000
Go to: http://localhost:3000
200 /index.js
404 /not_real.js
```

只需几行代码，您就能够构建一个提供静态内容的 HTTP 服务器。

HTTP 服务器是互联网的基本组件之一。Node.js 使构建强大的服务器变得简单。在这个练习中，只需几行代码，我们就建立了一个静态 HTTP 服务器。在本节的其余部分，我们将学习如何构建一个动态服务器，它可以使用模板和从请求中传递的数据生成 HTML，并且还可以从其他数据源加载，比如 JSON 文件。

在继续构建动态 HTTP 服务器之前，让我们看看 Node.js 中可用的 HTTP 客户端 API。为了测试 HTTP 客户端 API，我们将使用 HTTP Bin，这是一个免费的服务，可以用来测试 HTTP 请求。您可以在这里阅读更多信息：[`httpbin.org`](https://httpbin.org)。

在接下来的章节中，您将了解每个 HTTP 方法的含义，但现在，我们将只探索其中的两个：GET 和 POST。HTTP GET 是我们到目前为止一直在使用的。它告诉服务器：“为我获取这个 URL。” HTTP POST 的意思是：“将这个内容存储在这个 URL 上。”在我们之前构建的静态服务器中，它是磁盘上一个真实文件的真实路径。但它可以以服务器认为合适的任何方式使用。

让我们使用 Node.js 执行对`httpbin` API 的 GET 请求。HTTP 客户端模块与服务器位于同一模块中，因为它使用了许多相同的构造。因此，我们必须要求 http 模块：

```js
const http = require('http');
```

由于 GET 是一个广泛使用的 HTTP 方法，http 模块为其设置了别名。我们可以通过调用`get`函数来执行 GET：

```js
const request = http.get('http://httpbin.org/get', (response) => {
```

`get`函数接收 URL 和回调函数，一旦服务器开始发送它，回调就会被调用并传递给响应。传递给回调的响应是一个可读流，我们可以通过监听数据事件来从中获取数据：

```js
response.on('data', (data) => {
  console.log(data.toString());
});
```

这里的数据是响应的主体。如果我们只是将其打印到控制台，我们将在终端中看到响应。

`get`方法返回的请求实例是一个可写流。要告诉服务器我们已经完成了请求，我们需要调用`end`方法：

```js
request.end();
```

以下是前面代码的一些示例输出，可以在 GitHub 上找到[`github.com/TrainingByPackt/Professional-JavaScript/blob/master/Lesson03/sample_http/http_client_get.js`](https://github.com/TrainingByPackt/Professional-JavaScript/blob/master/Lesson03/sample_http/http_client_get.js)：

```js
$ node http_client_get.js 
{
  "args": {}, 
  "headers": {
    "Host": "httpbin.org"
  }, 
  "origin": "100.0.53.211, 100.0.53.211", 
  "url": "https://httpbin.org/get"
}
```

您可以看到它将响应主体打印到终端。

有时，您需要发送一些额外的标头或使用 HTTP 基本身份验证。为此，`get`方法接受一个`options`对象，您可以在其中设置标头、基本身份验证等。以下是一个示例选项对象，其中设置了自定义标头和基本身份验证：

```js
const options = {
  auth: 'myuser:mypass',
  headers: {
    Test: 'Some Value'
  }
};
```

然后，在回调函数之前传递选项对象：

```js
const request = http.get(url, options, (response) => {
```

以下片段是前述代码的输出，也可在 GitHub 上找到[`github.com/TrainingByPackt/Professional-JavaScript/blob/master/Lesson03/sample_http/http_client_get_with_headers.js`](https://github.com/TrainingByPackt/Professional-JavaScript/blob/master/Lesson03/sample_http/http_client_get_with_headers.js)：

```js
$ node http_client_get_with_headers.js 
{
  "args": {}, 
  "headers": {
    "Authorization": "Basic bXl1c2VyOm15cGFzcw==", 
    "Host": "httpbin.org", 
    "Test": "Some Value"
  }, 
  "origin": "100.0.53.211, 100.0.53.211", 
  "url": "https://httpbin.org/get"
}
```

`httpbin`响应我们在请求中传递的所有信息。您可以看到现在有两个额外的标头，Test 和 Authorization，其值与我们指定的相同。授权标头是 base64 编码的，如基本身份验证规范中指定的。

如前所述，get 方法只是一个别名。request 方法是其更灵活的版本，可用于执行 HTTP POST 请求。尽管它更灵活，但 request 方法接收相同的参数：`url`、`options`和`callback`。

要指定要执行的 HTTP 方法，我们在选项对象中设置它：

```js
const options = {
  method: 'POST',
};
```

然后，我们调用 request 函数，而不是 get 函数：

```js
const request = http.request(url, options, (response) => {
```

如果要向服务器发送数据，可以使用我们创建的请求对象。请记住，它是一个可写流，因此我们可以直接将内容写入其中：

```js
request.write('Hello world.');
```

在向请求写入数据后，调用`end`方法，请求就完成了：

```js
request.end();
```

使用我们之前解释过的 write 和`end`方法的一些示例代码可在 GitHub 上找到[`github.com/TrainingByPackt/Professional-JavaScript/blob/master/Lesson03/sample_http/http_client_post.js`](https://github.com/TrainingByPackt/Professional-JavaScript/blob/master/Lesson03/sample_http/http_client_post.js)。

以下是运行上述代码的输出：

```js
$ node http_client_post.js 
{
  "args": {}, 
  "data": "Hello world.", 
  "files": {}, 
  "form": {}, 
  "headers": {
    "Content-Length": "12", 
    "Host": "httpbin.org"
  }, 
  "json": null, 
  "origin": "100.0.53.211, 100.0.53.211", 
  "url": "https://httpbin.org/post"
}
```

您可以看到 http 模块会根据您发送的数据量自动设置 Content-Length 标头。您还可以看到响应中设置了数据属性，指示服务器接收到的数据。

### 练习 14：提供动态内容

在本练习中，我们将重写上一章的商店前端。但现在，内容将以动态方式提供，并且 HTML 将在服务器端生成。为此，我们将有一个存储在 JSON 文件中的产品数组，该数组将被加载并用于生成要返回给客户端的 HTML 文件。

有许多生成要发送给客户端的 HTML 的方法：连接字符串，搜索和替换，模板字符串，甚至可以使用诸如 cheerio 之类的库。模板化通常是最简单的，因为您可以将模板存储在一个单独的文件中，就像普通的 HTML 文件一样，但其中有一些占位符。在本练习中，我们将使用 handlebars 模板库来完成这项艰苦的工作。

执行以下步骤以完成此练习：

1.  创建一个新的 npm 包，其中包含一个`index.js`文件。安装我们在本练习中将使用的两个外部包：

```js
$ npm init
...
$ npm install handlebars mime
+ handlebars@4.1.2
+ mime@2.4.4
updated 2 packages and audited 10 packages in 1.075s
found 0 vulnerabilities
```

handlebars 包是一个模板引擎。它可用于渲染带有占位符和一些基本逻辑（如 for 循环和 if/else 语句）的模板文本。我们还将使用之前使用过的`mime`包来确定静态提供的文件的内容类型。

1.  在应用程序中需要所有将使用的模块：

```js
const fs = require('fs');
const handlebars = require('handlebars');
const http = require('http');
const mime = require('mime');
const path = require('path');
const url = require('url');
```

1.  使用基本目录检查静态文件的路径。该目录将是脚本加载的静态目录。我们将该路径存储在变量中，以便以后使用：

```js
const staticDir = path.resolve(`${__dirname}/static`);
console.log(`Static resources from ${staticDir}`);
```

1.  接下来，我们使用`readFileSync`从 JSON 文件中加载产品数组。我们使用内置的`JSON.parse`函数解析 JSON，然后将找到的产品数量打印到控制台：

```js
const data = fs.readFileSync(`products.json`);
const products = JSON.parse(data.toString());
console.log(`Loaded ${products.length} products...`);
```

Handlebars 有一个辅助函数的概念。这些是可以在模板内注册和使用的函数。要注册一个辅助函数，您调用`registerHelp`函数，将您的辅助函数的名称作为第一个参数传递，并将处理程序函数作为第二个参数传递。

1.  让我们添加一个辅助函数，用于格式化货币：

```js
handlebars.registerHelper('currency', (number) => `$${number.toFixed(2)}`);
```

1.  为了初始化 HTTP 处理程序并开始监听连接，我们将使用以下函数：

```js
function initializeServer() {
  const server = http.createServer();
  server.on('request', handleRequest);
  const port = 3000;
  console.log('Go to: http://localhost:%d', port);
  server.listen(port);
}
```

我们在 HTTP 服务器中注册了一个名为`handleRequest`的函数。这是根处理程序，所有请求都将通过它。对于这个应用程序，我们期望有两种类型的请求：第一种是指向 css、图像和其他静态文件的静态请求，而第二种是获取商店 HTML 的请求。这意味着我们的根处理程序只关心这两种情况。

1.  要请求商店，我们将假设当用户请求`/`或`/index.html`(`http://localhost:3000/`或`http://localhost:3000/index.html`)时，用户正在尝试访问商店，也就是应用程序的基本页面或根页面。其他一切都将被视为静态资源。为了处理这些请求，我们将解析 URL，检查路径名，并使用`if`语句：

```js
function handleRequest(request, response) {
  const requestUrl = url.parse(request.url);
  const pathname = requestUrl.pathname;
  if (pathname == '/' || pathname == '/index.html') {
    handleProductsPage(requestUrl, response);
    return;
  }
  handleStaticFile(pathname, response);
}
```

1.  为了处理静态文件，我们将在静态文件应该来自的目录前面添加路径，并将其用作完整路径。然后，我们将使用文件系统 API 中的`access`函数来检查文件是否存在并且可读。如果有错误，那么返回`404`错误；否则，只需创建一个可读流并将文件的内容传输到响应。我们还希望使用 mime 库来检查每个文件的内容类型，并向响应添加一个头部：

```js
function handleStaticFile(pathname, response) {
  // For security reasons, only serve files from static directory
  const fullPath = path.join(staticDir, pathname);
  // Check if file exists and is readable
  fs.access(fullPath, fs.constants.R_OK, (error) => {
    if (error) {
      console.error(`File is not readable: ${fullPath}`, error);
      response.writeHead(404);
      response.end();
      return;
    }
    const contentType = mime.getType(path.extname(fullPath));
   response.writeHead(200, { 'Content-type': contentType });
    fs.createReadStream(fullPath)
      .pipe(response);
  });
}
```

1.  现在我们有了用于提供静态文件的函数，让我们使用 handlebars 来提供动态内容。为此，我们需要使用`readFileSync`加载 HTML 模板，然后编译它。编译后的脚本被转换为一个函数，当调用时返回处理过的模板的字符串。

模板函数接收将用于呈现模板的上下文。上下文可以在模板中访问，这将在下一步中演示。对于这个应用程序，上下文将是一个带有一个名为`products`的属性的对象：

```js
const htmlString = fs.readFileSync(`html/index.html`).toString();
const template = handlebars.compile(htmlString);
function handleProductsPage(requestUrl, response) {
  response.writeHead(200);
 response.write(template({ products: products }));
  response.end();
}
```

1.  在模板处理就位后，我们需要一个模板。Handlebars 使用双花括号作为占位符（例如，`{{variable}}`），你可以使用双花括号和井号来执行 for 循环：`{{#arrayVariable}}`。在一个相对于`index.js`文件的`html/index.html`文件中，添加以下 HTML 模板：

```js
<html>
  <head>
    <link rel="stylesheet" type="text/css" href="css/semantic.min.css" />
    <link rel="stylesheet" type="text/css" href="css/store.css" />
  </head>
  <body>
    <section>
      <h1 class="title">Welcome to Fresh Products Store!</h1>
      <div class="ui items">
        {{#products}}
        <div class="item">
          <div class="image"><img src="{{image}}" /></div>
          <div class="content">
            <a class="header">{{name}}</a>
            <div class="meta">
              <span>{{currency price}} / {{unit}}</span>
            </div>
            <div class="description">{{description}}</div>
            <div class="extra">
              {{#tags}}
              <div class="ui label teal">{{this}}</div>
              {{/tags}}
            </div>
         </div>
        </div>
        {{/products}}
      </div>
    </section>
  </body>
</html>
```

注意辅助函数`currency`，它被调用来呈现价格：`{{currency price}}.`

1.  不要忘记在最后调用`initialize`函数以开始监听 HTTP 连接：

```js
initializeServer();
```

为了使商店正确加载和呈现，你还需要 css 文件和图像。只需将它们放在一个名为**static**的文件夹中。你可以在 GitHub 上找到这些文件：[`github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson03/Exercise14`](https://github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson03/Exercise14)。

1.  所有文件就位后，运行服务器：

```js
$ node .
Static resources from
.../Lesson03/Exercise14/static
Loaded 21 products...
Go to: http://localhost:3000
```

1.  打开浏览器窗口，转到`http://localhost:3000`。你应该看到商店：

![图 3.13：从动态网络服务器提供的商店](img/C14587_03_13.jpg)

###### 图 3.13：从动态网络服务器提供的商店

在这个练习中，我们将商店应用程序转换为一个动态的网络应用程序，它从一个 JSON 文件中读取数据，并在用户请求时呈现一个 HTML 请求。

动态网络服务器是所有在线应用程序的基础，从 Uber 到 Facebook。你可以总结这项工作为加载数据/处理数据以生成 HTML。在*第二章，Node.js 和 npm*中，我们在前端使用了一些简单的 HTML 并进行了处理。在这个练习中，你学会了如何在后端使用模板引擎来完成相同的工作。每种方法都有其优缺点，大多数应用程序最终会结合两者。

你可以将过滤选项添加到商店前端网页作为改进。比如说用户想要按标签或它们的组合来筛选产品。在你的`handleProductsPage`函数中，你可以使用查询参数来过滤你传递给模板渲染的产品列表。看看你是否可以自己做出这个改进。

## 什么是爬取？

在本章的其余部分，我们将讨论网络**爬取**。但网络爬取到底是什么？这是下载页面并处理其内容以执行一些重复的自动化任务的过程，否则这些任务将需要手动执行太长时间。

例如，如果你想要购买汽车保险，你需要去每家保险公司的网站获取报价。这个过程通常需要几个小时，因为你需要填写表单，提交表单，等待他们在每个网站给你发送电子邮件，比较价格，然后选择你想要的：

![图 3.14：用户下载内容，输入数据，提交数据，然后等待结果](img/C14587_03_14.jpg)

###### 图 3.14：用户下载内容，输入数据，提交数据，然后等待结果

那么为什么不制作一个可以为你做到这一点的程序呢？这就是网络爬取的全部内容。一个程序像人一样下载页面，从中提取信息，并根据某种算法做出决策，然后将必要的数据提交回网站。

当你为你的汽车购买保险时，似乎自动化不会带来太多价值。为不同的网站编写正确执行此操作的应用程序将花费很多时间——比手动操作自己做要多得多。但如果你是一家保险经纪公司呢？那么你每天可能要做这个动作数百次，甚至更多。

如果你是一个保险经纪公司，如果你花时间建立一个机器人（这些应用程序就是这样称呼的），你将开始变得更加高效。这是因为对于那个网站，你不需要花时间填写表单。通过建立第一个机器人获得的效率，你可以节省时间并能够建立第二个，然后是第三个，依此类推：

![图 3.15：机器人通过下载内容并根据算法做出决策自动执行任务](img/C14587_03_15.jpg)

###### 图 3.15：机器人通过下载内容并根据算法做出决策自动执行任务

网络爬虫始于互联网早期，当时雅虎！试图手动索引所有存在的网站。然后，一家初创公司，由两名大学生在车库里开始使用机器人来提取数据并索引一切。在很短的时间内，谷歌成为了第一大搜索网站，这个位置对竞争对手来说越来越难以挑战。

网络爬取是一种广泛使用的技术，用于从不提供 API 的网站提取数据，比如大多数保险公司和银行。搜索和索引也是另一个非常常见的情况。一些公司使用爬取来分析网站的性能并对其进行评分，比如 HubSpot（[`website.grader.com`](https://website.grader.com)）。

网络爬虫有许多技术，取决于你想要实现的目标。最基本的技术是从网站下载基本的 HTML 并从中读取内容。如果你只需要下载数据或填写表单，这可能已经足够了：

![图 3.16：基本的爬取技术涉及下载和处理基本的 HTML 文件](img/C14587_03_16.jpg)

###### 图 3.16：基本的爬取技术涉及下载和处理基本的 HTML 文件

但有时，网站使用 Ajax 在 HTML 渲染后动态加载内容。对于这些情况，仅下载 HTML 是不够的，因为它只是一个空模板。为了解决这个问题，您可以使用一个无头浏览器，它像浏览器一样工作，解析所有 HTML，下载和解析相关文件（CSS、JavaScript 等），将所有内容一起渲染，并执行动态代码。这样，您就可以等待数据可用：

![图 3.17：根据用例，抓取需要一个模拟或完全无头浏览器来更准确地下载和渲染页面](img/C14587_03_17.jpg)

###### 图 3.17：根据用例，抓取需要一个模拟或完全无头浏览器来更准确地下载和渲染页面

第二种技术要慢得多，因为它需要下载、解析和渲染整个页面。它也更加脆弱，因为执行额外的调用可能会失败，等待 JavaScript 完成处理数据可能很难预测。

### 下载和解析网页

让我们来看看更简单的网页抓取方法。假设我们想要关注 Medium 上关于 JavaScript 的最新帖子。我们可以编写一个应用程序来下载 JavaScript 主题页面，然后搜索锚点（链接），并使用它来导航。

首先，拥有一个通用的下载函数，它将对 HTTP 客户端进行一些基本的封装，是一个好主意。我们可以使用外部库，比如 request，但让我们看看如何封装这种逻辑。

我们将需要 http 模块，但在这种情况下，我们将使用它的 https 版本，因为大多数网站这些天会在你尝试访问普通 HTTP 版本时将你重定向到它们的安全版本。https 模块提供了相同的 API，只是它理解 HTTPS 协议，这是 HTTP 的安全版本。

```js
const http = require('https');
```

`downloadPage`函数接收要下载的 URL 和在页面内容下载完成后将被调用的回调函数：

```js
function downloadPage(urlToDownload, callback) {
}
```

在该函数内部，我们将首先发出一个请求，并确保我们调用 end 函数来完成请求：

```js
const request = http.get(urlToDownload, (response) => {
});
request.end();
```

在我们传递给 get 函数的回调中，我们首先要做的是检查响应状态，并在它不匹配 200 时打印错误消息，这是表示我们有一个成功请求的 HTTP 代码。如果发生这种情况，我们还会通过从回调中返回来停止一切，因为如果发生这种情况，body 可能不是我们所期望的。

```js
if (response.statusCode != 200) {
  console.error('Error while downloading page %s.', urlToDownload);
  console.error('Response was: %s %s', response.statusCode, response.statusMessage);
  return;
}
```

在那个`if`语句之后，我们可以使用数据事件在一个变量中累积页面的内容。当连接关闭时，在`close`事件中，我们调用回调函数，并将累积在 content 变量中的全部内容传递给它。

```js
let content = '';
response.on('data', (chunk) => content += chunk.toString());
response.on('close', () => callback(content));
```

这个示例的完整代码可以在 GitHub 上找到：[`github.com/TrainingByPackt/Professional-JavaScript/blob/master/Lesson03/sample_scraping/print_all_texts.js`](https://github.com/TrainingByPackt/Professional-JavaScript/blob/master/Lesson03/sample_scraping/print_all_texts.js)。

这个函数的一个简单用法如下：

```js
downloadPage('https://medium.com/topic/javascript', (content) => {
  console.log(content);
});
```

这将下载页面并将其打印到控制台。但我们想做更多的事情，所以我们将使用`jsdom`库来解析 HTML 并从中获取一些信息。`jsdom`是一个解析 HTML 并生成 DOM 表示的库，可以像浏览器中的 DOM 一样进行查询和操作。

使用`npm install`命令安装后，您可以在代码中引用它。该模块公开了一个接收字符串的构造函数。在被实例化后，`JSDOM`实例包含一个窗口对象，其工作方式与浏览器中的窗口对象完全相同。以下是使用它来获取所有锚点、过滤掉空的锚点并打印它们的文本的示例：

```js
const JSDOM = require('jsdom').JSDOM;
downloadPage('https://medium.com/topic/javascript', (content) => {
 const document = new JSDOM(content).window.document;
  Array.from(document.querySelectorAll('a'))
    .map((el) => el.text)
    .filter(s => s.trim() != '')
    .forEach((s) => console.log(s));
});
```

以下是前述代码的示例输出：

```js
$ node print_all_texts.js 
Javascript
Become a member
Sign in
14 Beneficial Tips to Write Cleaner Code in React Apps
Be a hygienic coder by writing cleaner
14 Beneficial Tips to Write Cleaner Code in React Apps
Be a hygienic coder by writing cleaner
...
```

### 练习 15：抓取 Medium 文章

在这个练习中，我们将使用爬虫在控制台上打印文章。让我们利用这些知识构建一个应用程序，该应用程序将从 Medium 下载主题页面，解析信息，并以可消化的方式打印出来。该应用程序将有一个硬编码的主题列表，并将下载每个页面的 HTML。然后，它将使用`jsdom`解析已下载的内容，获取有关每篇文章的信息，并以漂亮的格式在控制台上打印出来，使每篇文章都只是一个点击之遥。

执行以下步骤完成此练习：

1.  创建一个新文件夹，其中包含一个`index.js`文件。然后，运行`npm init`并使用`npm install`安装`jsdom`：

```js
$ npm init
...
$ npm install jsdom
+ jsdom@15.1.1
added 97 packages from 126 contributors and audited 140 packages in 12.278s
found 0 vulnerabilities
```

1.  在`index.js`文件中，使用 require 函数引入我们将使用的所有模块：

```js
const http = require('https');
const JSDOM = require('jsdom').JSDOM;
const url = require('url');
```

1.  创建一个包含我们将下载页面的所有主题的常量数组：

```js
const topics = [
  'artificial-intelligence',
  'data-science',
  'javascript',
  'programming',
  'software-engineering',
];
```

1.  复制我们在上一节中创建的`downloadPage`函数：

```js
function downloadPage(urlToDownload, callback) {
  const request = http.get(urlToDownload, (response) => {
    if (response.statusCode != 200) {
      console.error('Error while downloading page %s.', urlToDownload);
      console.error('Response was: %s %s', response.statusCode, response.statusMessage);
      return;
    }
    let content = '';
    response.on('data', (chunk) => content += chunk.toString());
    response.on('close', () => callback(content));
  });
  request.end();
}
```

1.  迭代每个主题，为每个主题调用`downloadPage`函数：

```js
topics.forEach(topic => {
  downloadPage(`https://medium.com/topic/${topic}`, (content) => {
    const articles = findArticles(new JSDOM(content).window.document);
    Object.values(articles)
     .forEach(printArticle);
  });
});
```

在上述代码中，我们调用了两个函数：`findArticles`和`printArticle`。第一个函数将遍历从页面解析的 DOM，并返回一个对象，其中键是文章标题，值是包含每篇文章信息的对象。

1.  接下来，我们编写`findArticles`函数。我们首先初始化对象，该对象将是函数的结果，然后查询传递的文档中所有 H1 和 H3 元素内的所有锚点元素，这些元素代表文章的标题：

```js
function findArticles(document) {
  const articles = {};
  Array.from(document.querySelectorAll('h1 a, h3 a'))
```

1.  根据 Medium 文章路径有两部分：`/author/articleId`，过滤锚点。这意味着我们可以将锚点的`href`解析为 URL，获取路径名，使用“/”作为分隔符拆分，并忽略那些不完全有两部分的锚点：

```js
.filter(el => {
  const parsedUrl = url.parse(el.href);
  const split = parsedUrl.pathname.split('/').filter((s) => s.trim() != '');
  return split.length == 2;
})
```

使用 Chrome 开发者工具在页面上，您可以看到文章的标题位于一个标题元素内，其下一个兄弟元素是一个包含以下简短描述的 DIV：

![图 3.18：父级的下一个兄弟元素包含文章的简短描述](img/C14587_03_18.jpg)

###### 图 3.18：父级的下一个兄弟元素包含文章的简短描述

这意味着对于每个锚元素，我们可以获取该 DIV，查询一个锚点，并获取其文本作为文章的描述。

1.  使用文章标题作为键，将文章信息设置在结果对象中。我们使用文章的标题作为键，因为这将自动去重结果中的文章：

```js
.forEach(el => {
  const description = el.parentNode.nextSibling.querySelector('p a').text;
  articles[el.text] = {
    description: description,
    link: url.parse(el.href).pathname,
    title: el.text,
 };
});
```

1.  最后，从`findArticles`函数中返回包含所有文章的数组：

```js
  return articles;
}
```

我们在传递给`downloadPage`的回调中调用的另一个函数是`printArticle`。这也是使该应用程序完整的最后一部分代码。

1.  让我们编写`printArticle`函数，它接收一个文章对象，并以漂亮的方式将其打印到控制台上：

```js
function printArticle(article) {
  console.log('-----');
  console.log(` ${article.title}`);
  console.log(` ${article.description}`);
  console.log(` https://medium.com${article.link}`);
}
```

运行应用程序，以漂亮的格式将文章打印到控制台上，附加额外信息：

![图 3.19：运行应用程序后在控制台上打印的文章](img/C14587_03_19.jpg)

###### 图 3.19：运行应用程序后在控制台上打印的文章

在这个练习中，我们编写了一个从 Medium 获取数据并将找到的文章摘要打印到控制台的应用程序。

网络爬虫是在没有 API 可用时获取数据的强大方式。许多公司使用爬虫在系统之间同步数据，分析网站的性能，并优化否则无法扩展的流程，从而阻碍了一些重要的业务需求。了解爬虫背后的概念使您能够构建否则不可能构建的系统。

### 活动 4：从商店前端爬取产品和价格

在*第二章，Node.js 和 npm*中，我们编写了一些代码，用于获取商店示例页面中产品的信息。当时，我们说网站不会经常更新，因此可以从 Chrome 开发者控制台手动执行。对于某些情况，这是可以接受的，但是当内容是动态生成的，就像我们在本章中编写的商店的新版本一样，我们可能需要消除所有手动干预。

在此活动中，您将编写一个应用程序，通过使用 http 模块下载商店网页并使用`jsdom`解析它来抓取商店网页。然后，您将从 DOM 中提取数据并生成一个带有数据的`CSV`文件。

您需要执行以下步骤才能完成此活动：

1.  使用您之前构建的代码或其副本来为`localhost:3000`提供商店前端网站。 代码可以在 GitHub 上找到[`github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson03/Activity04`](https://github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson03/Activity04)。

1.  创建一个新的`npm`包，安装`jsdom`库，并创建一个名为`index.js`的入口文件。

1.  在入口文件中，调用`require()`方法加载项目中所需的所有模块。

1.  向`localhost:3000`发出 HTTP 请求。

1.  确保成功响应并从主体中收集数据。

1.  使用`jsdom`解析 HTML。

1.  从 DOM 中提取产品数据； 您将需要名称，价格和单位。

1.  打开`CSV`文件，数据将被写入其中。

1.  将产品数据写入`CSV`文件，这是一个产品行。

1.  运行应用程序并检查结果。

输出应该看起来像这样：

```js
$ node .
Downloading http://localhost:3000...
Download finished.
Parsing product data...
.....................
Found 21 products.
Writing data to products.csv...
Done.
$ cat products.csv 
name,price,unit
Apples,3.99,lb
Avocados,4.99,lb
Blueberry Muffin,2.5,each
Butter,1.39,lb
Cherries,4.29,lb
Chocolate Chips Cookies,3.85,lb
Christmas Cookies,3.89,lb
Croissant,0.79,each
Dark Chocolate,3.49,lb
Eggs,2.99,lb
Grapes,2.99,lb
Milk Chocolate,3.29,lb
Nacho Chips,2.39,lb
Parmesan Cheese,8.99,lb
Pears,4.89,lb
Petit French Baguette,0.39,each
Smiling Cookies,2.79,lb
Strawberries,7.29,lb
Swiss Cheese,2.59,lb
White Chocolate,3.49,lb
Whole Wheat Bread,0.89,each
```

#### 注意

此活动的解决方案可以在第 591 页找到。

## 摘要

在本章中，我们学习了每个 Node.js 脚本都可以使用的全局变量。我们学习了如何设置定时器并从控制台读取和写入数据。之后，我们学习了有关流的知识以及如何使用它们从文件中读取和写入数据。我们还学习了如何使用同步文件系统 API。然后，我们学习了如何使用 HTTP 模块构建 Web 服务器并从 Web 页面中抓取内容。

现在您已经对 Web 抓取概念有了很好的了解，可以开始探索机会，构建自己的 Web 应用程序，并构建自动机器人来从其他 Web 应用程序中抓取内容。一个好主意是尝试构建一个简单的内容管理应用程序来为您的博客提供服务，您将在其中写有关您刚学到的所有新事物的内容。

在下一章中，您将学习有关 REST API，并使用一些框架来帮助您构建它们。在后续章节中，您将学习有关可以使用的技术，以管理异步操作，使您的 Node.js 应用程序功能强大，但代码易于编写和维护。
