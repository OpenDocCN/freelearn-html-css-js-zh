

# 第十四章：Node.js 中的错误处理

Node.js 应用程序需要对错误进行稳固和一致的控制。大多数应用程序都是使用许多依赖项构建的，或者高度依赖于异步操作（网络、磁盘等），这使得错误管理变得更加复杂。

在本章中，我们将学习在 Node.js 应用程序中可能遇到的不同类型的错误以及如何正确处理它们。我们还将学习如何抛出自定义错误，以及如何捕获和从任何类型的错误中恢复应用程序，包括在 Express 应用程序中发生的错误。

我们还将学习如何在服务崩溃时进行优雅的关闭，如何根据情况使用退出代码，以及如何防止僵尸进程。

总结一下，以下是本章我们将探讨的主要主题：

+   如何抛出自定义错误

+   如何捕获和从任何类型的错误中恢复

+   如何在 Express 中管理应用程序和用户错误

+   如何在服务崩溃时进行优雅的关闭

+   如何防止僵尸进程

+   如何使用退出代码来指示应用程序关闭的原因

# 技术要求

本章的代码文件可以在[`github.com/PacktPublishing/NodeJS-for-Beginners`](https://github.com/PacktPublishing/NodeJS-for-Beginners)找到。

请查看本章关于代码执行的视频：[`youtu.be/VPXV1L1epIk`](https://youtu.be/VPXV1L1epIk)

# 探索错误类型

正如我们在第一章中学到的，Node.js 是一个单线程应用程序。这意味着如果发生错误而我们没有正确处理它，应用程序将会崩溃。这就是为什么正确处理错误很重要的原因。

Node.js 中主要有两种错误类型：**语法**错误和**运行时**错误。

## 语法错误

当代码解析时，如果它无效，就会抛出语法错误。这些错误由 JavaScript 引擎抛出，通常很容易修复。许多 IDE 和代码编辑器可以检测这些错误并在代码编辑器中突出显示，这样你就可以在运行应用程序之前修复它们。在我们的案例中，我们已经在之前的章节中使用 StandardJS 作为代码检查器（这是一种帮助我们检测语法错误并强制执行一致代码风格的工具）。

这是一个语法错误的例子：

```js
executeThisFunction()
```

之前的代码将抛出`ReferenceError`错误，因为`executeThisFunction`函数未定义。这个错误可以通过定义函数来轻松修复：

```js
executeThisFunction()
ReferenceError: executeThisFunction is not defined
    at file:///file.js:1:1
    at ModuleJob.run (node:internal/modules/esm/module_job:192:25)
    at async DefaultModuleLoader.import (node:internal/modules/esm/loader:228:24)
    at async loadESM (node:internal/process/esm_loader:40:7)
    at async handleMainPromise (node:internal/modules/run_main:66:12)
```

## 运行时错误

运行时错误也被称为**操作**错误。这些错误在应用程序运行时抛出，与代码的语法无关。这些错误可以由应用程序本身或应用程序使用的依赖项抛出。

有许多方式可以生成运行时错误，例如通过访问未定义对象的属性、调用不存在的函数、尝试读取不存在的文件、尝试连接不可用的数据库以及尝试访问不可用的网络资源。

如您所见，有多种方式可以生成运行时错误。这就是为什么正确处理它们很重要。如果我们不处理它们，应用程序将会崩溃并停止工作。因此，在编写应用程序代码时，牢记可能抛出的运行时错误以及如何处理它们是非常重要的。

一些错误可以恢复，而另一些则不能，这取决于错误的类型。例如，如果我们有一个 REST API 应用程序，并且数据库不可用，我们可以通过返回一个`503` HTTP 状态码和一条消息给客户端来从该错误中恢复。您将始终负责决定错误是否可以恢复以及如何处理它。

现在我们已经了解了在 Node.js 应用程序中可能抛出的错误类型，让我们在下一节中看看如何抛出有意义的错误。

# 抛出有意义的错误

当发生错误时，重要的是它是有意义的。这意味着错误应该包含足够的信息来了解发生了什么，以及可能如何修复它。

## 错误对象

错误对象是`Error`类的一个实例。这个类有一个接受消息作为参数的构造函数。这个消息将用来描述错误。以下是一个示例：

```js
const myError = new Error('This is an error message')
throw myError
```

以下是上一段代码的输出：

```js
file:///file.js:1
const myError = new Error('This is an error message')
                ^
Error: This is an error message
    at file:///file.js:1:17
    at ModuleJob.run (node:internal/modules/esm/module_job:192:25)
    at async DefaultModuleLoader.import (node:internal/modules/esm/loader:228:24)
    at async loadESM (node:internal/process/esm_loader:40:7)
    at async handleMainPromise (node:internal/modules/run_main:66:12)
Node.js v20.11.0
```

如您所见，错误信息在输出中显示。这是我们传递给`Error`类构造函数的消息。如果您将其与`ReferenceError: executeThisFunction is not defined`进行比较，我们可以看到错误信息描述不够详细，并且我们正在使用一个通用的错误类。

## 自定义错误

您可以通过扩展`Error`类来创建自己的自定义错误。当您想创建自己的`Error`类并向错误对象添加更多信息时，这非常有用。以下是一个示例：

```js
class NotEnoughSleep extends Error {
  constructor (message) {
    super(message)
    this.requireSleep = true
    this.isRecoverable = true
  }
}
throw new NotEnoughSleep('Looks like you need more sleep')
```

如果我们运行上一段代码，我们将得到以下输出：

```js
file:///file.js:9
  throw new NotEnoughSleep('Looks like you need more sleep')
NotEnoughSleep [Error]: Looks like you need more sleep
    at file:///file.js:9:9
    at ModuleJob.run (node:internal/modules/esm/module_job:192:25)
    at async DefaultModuleLoader.import (node:internal/modules/esm/loader:228:24)
    at async loadESM (node:internal/process/esm_loader:40:7)
    at async handleMainPromise (node:internal/modules/run_main:66:12) {
  requireSleep: true,
  isRecoverable: true
}
```

如您所见，输出中显示了错误信息`看起来你需要更多的睡眠`，以及类名`NotEnoughSleep`。此外，我们还在错误对象中添加了两个属性：`requireSleep`和`isRecoverable`。这些属性是我们自己创建的，我们可以根据需要创建任意多个，并且可以尽可能具体。这些属性可以用来向错误对象添加更多信息，这样我们就可以使用这些属性在`try`/`catch`块中正确地处理错误：

```js
try {
  throw new NotEnoughSleep("Looks like you need more sleep");
} catch (error) {
  if (error.isRecoverable) {
    console.log("You are lucky, because you can recover from this error");
  }
  if (error.requireSleep) {
    console.log("Please, go to sleep!");
  }
}
```

以下是上一段代码的输出：

```js
You are lucky, because you can recover from this error
Please, go to sleep!
```

如您所见，我们使用了`isRecoverable`和`requireSleep`属性来处理错误。这是一个非常简单的例子，但您可以向错误对象添加更多属性来正确处理错误。

在下一节中，我们将学习如何在使用 Express 时捕获和恢复任何类型的错误。

# 在 Express 中管理错误

在前面的章节中，我们学习了如何使用 Express 创建 REST API 应用程序，并看到了如何在 Express 应用程序中处理错误，但在这个章节中，我们将刷新这些概念并对其进行扩展。

## 错误处理中间件

Express 有一个内置的错误处理中间件，可以用来集中处理应用程序中的错误。当应用程序中发生错误时，将执行此中间件。此中间件在所有其他中间件和路由执行之后执行。它仅在发生错误时执行，因此非常重要，需要将其添加到中间件链的末尾，如下所示：

```js
import express from 'express'
const app = express()
// Other middlewares...
app.use((err, req, res, next) => {
  console.error(err.stack)
  res.status(500).send('Something broke!')
})
// Route handler...
```

## 自定义错误

如果你正在构建 REST API 应用程序，你可以在错误对象中添加一个属性来指示应返回给客户端的 HTTP 状态码。这样，你可以在错误处理中间件中正确处理错误，并将适当的 HTTP 状态码返回给客户端。以下是一个示例：

```js
class NotFoundError extends Error {
  constructor (message) {
    super(message)
    this.statusCode = 404
  }
}
try {
  throw new NotFoundError('The resource was not found')
} catch (error) {
  console.log(error.statusCode)
  res.status(error.statusCode).send(error.message)
}
```

如你所见，我们可以使用 `statusCode` 属性返回适当的 HTTP 状态码给客户端。这是一个非常简单的示例，但你可以在错误对象中添加更多属性来正确处理错误。

现在我们已经知道了如何处理错误，是时候学习如何在应用程序无法从错误中恢复时优雅地关闭应用程序了。

# 优雅地关闭应用程序

在整本书中，我们学习了如何使用 `try`/`catch` 块、错误优先回调、`catch` 用于承诺以及事件来处理错误，但有时我们需要全局处理错误。

Node.js 提供了一种处理错误的全局方法，并在发生错误时优雅地关闭应用程序：使用 `process.on()`。你还可以使用 `process.exit()` 以特定的退出代码退出应用程序。这在 CI/CD 管道中非常有用，可以指示应用程序是否因为错误而关闭，以及在生产环境中。

## 事件

有许多事件可以用来处理全局错误：

+   `uncaughtException`：当发生未捕获的异常时，此事件被触发

+   `unhandledRejection`：当发生未处理的拒绝时，此事件被触发

+   `exit`：当 Node.js 进程即将退出时，此事件被触发

+   `SIGINT` 和 `SIGTERM`：当 Node.js 进程接收到这些信号时，这些事件被触发

可以使用许多其他事件来处理全局错误，但这些都是最常见的。在以下示例中，我们将结合一些场景来处理全局错误：

```js
const events = ['uncaughtException','unhandledRejection', 'exit', 'SIGINT'];
events.forEach(event => {
  process.on(event, (error) => {
    console.log(`This is an ${event} that we track!`)
  })
})
setTimeout(() => {
  throw new Error('Exception!')
}, 10000)
setTimeout(() => {
  Promise.reject(new Error('Rejection!'))
}, 20000)
```

如果你运行前面的代码，你会看到由于未处理的拒绝，应用程序将在 20 秒后关闭，但未捕获的异常最终被捕获，进程继续运行。此外，如果你在任何时候按下 *Ctrl* + *C*，应用程序将因为 `SIGINT` 信号而关闭。

在以下示例中，我们可以看到，当我们关闭 Node.js 应用程序时，`exit`事件总是被触发。因此，在应用程序关闭之前执行一些操作是很常见的：

```js
This is an uncaughtException that we track!
This is an unhandledRejection that we track!
This is an exit that we track!
```

请注意，`exit`事件不仅在发生错误时触发，而且当应用程序优雅地关闭且不支持异步操作时也会触发。

在下一节中，我们将学习如何使用退出码来指示应用程序关闭的原因。这在 CI/CD 管道中非常有用，可以指示应用程序是否是因为错误而关闭的。

## 退出码

退出码用于指示应用程序关闭的原因，以及应用程序是否因为错误而关闭，以及应用程序是否是优雅关闭的。

如果退出码是`0`，则表示应用程序是优雅关闭的。如果退出码不同于`0`，则表示应用程序是因为错误而关闭的。默认情况下，当应用程序中没有其他事情要做时，Node.js 将以退出码`0`退出。

通过使用`process.exit()`，我们可以指定我们想要使用的退出码。例如，如果我们想表示应用程序是因为错误而关闭的，我们可以使用`process.exit(1)`。如果我们想表示应用程序是优雅关闭的，我们可以使用`process.exit(0)`。

一些进程可能在执行上正确完成，但使用错误码。例如，当我们运行并完成应用程序测试时，如果任何测试失败，退出码将不同于`0`。这样，执行输出将是一个错误，可以防止持续执行 CI 管道。

在下一节中，我们将学习如何在处理错误时使用 process 库来防止僵尸进程。

## 避免僵尸进程

我喜欢僵尸电影，但我不喜欢僵尸进程。僵尸进程是指在后台运行且不做任何事情的进程。这类进程会消耗主机机的资源，在某些场景下（如低功耗设备）可能成为大问题。

使用`process.on()`可能很危险，因为它可能会阻止 Node.js 进程退出。这就是为什么在需要时使用`process.exit()`以特定的退出码退出应用程序很重要的原因。

让我们看看一个例子。如果我们不使用`process.exit()`，应用程序将不会退出，并且它将永远运行，即使执行一个未定义的函数时发生错误：

```js
process.on('uncaughtException', (error) => {
  console.log('We are not going to exit the application!')
})
setInterval(() => {
    executeThisFunction()
}, 1000)
```

这在以下输出中显示：

```js
We are not going to exit the application!
We are not going to exit the application!
We are not going to exit the application!
We are not going to exit the application!
We are not going to exit the application!
```

我们可以通过添加`process.exit()`来防止这种情况，以特定的退出码退出应用程序：

```js
process.on("uncaughtException", (error) => {
  console.log("Now, exit the application!");
  process.exit(1);
});
setInterval(() => {
  executeThisFunction();
}, 1000);
```

输出将如下所示：

```js
Now, exit the application!
```

如您所见，应用程序被关闭是因为我们使用了 `process.exit()` 来使用特定的退出代码退出应用程序。如果我们不使用 `process.exit()`，应用程序将永远运行，使其成为一个僵尸进程。

# 摘要

在本章中，我们学习了在 Node.js 应用程序中可能抛出的错误类型。我们看到了如何抛出自定义错误以及如何捕获和从任何类型的错误中恢复。

此外，我们还回顾了如何在 Express 中管理应用程序和用户错误。我们还学习了如何在服务崩溃时管理优雅关闭。

最后，我们学习了如何防止僵尸进程。

在下一章中，我们将学习更多关于安全性的内容，包括如何通过应用最佳实践来保护我们的应用程序，以及如何评估 CVE 和安全漏洞。

# 进一步阅读

+   *Express* | *健康检查和优雅* *关闭*: [`expressjs.com/en/advanced/healthcheck-graceful-shutdown.html`](https://expressjs.com/en/advanced/healthcheck-graceful-shutdown.html)

+   *Express* | *错误处理*: [`expressjs.com/en/guide/error-handling.html`](https://expressjs.com/en/guide/error-handling.html)

+   Node.js 文档 | *错误* *API*: [`nodejs.org/dist/latest-v20.x/docs/api/errors.html`](https://nodejs.org/dist/latest-v20.x/docs/api/errors.html)

+   *Bash 命令行退出代码* *揭秘*: [`www.redhat.com/sysadmin/exit-codes-demystified`](https://www.redhat.com/sysadmin/exit-codes-demystified)
