数据存储和检索

在前两章中，我们构建了一个小型且有些有用的存储笔记的应用程序，然后使其在移动设备上运行。虽然我们的应用程序运行得相当不错，但它并没有将这些笔记存储在长期基础上，这意味着当您停止服务器时，笔记会丢失，并且如果您运行多个`Notes`实例，每个实例都有自己的笔记集。我们的下一步是引入一个数据库层，将笔记持久化到长期存储中。

在本章中，我们将研究 Node.js 中的数据库支持，目标是获得对几种数据库的暴露。对于`Notes`应用程序，用户应该在访问任何`Notes`实例时看到相同的笔记集，并且用户应该能够随时可靠地访问笔记。

我们将从前一章中使用的`Notes`应用程序代码开始。我们从一个简单的内存数据模型开始，使用数组来存储笔记，然后使其适用于移动设备。在本章中，我们将涵盖以下主题：

+   数据库和异步代码之间的关系

+   配置操作和调试信息的记录

+   捕获重要的系统错误

+   使用`import()`来启用运行时选择要使用的数据库

+   使用多个数据库引擎为`Notes`对象实现数据持久化

+   设计简单的配置文件与 YAML

第一步是复制上一章的代码。例如，如果你在`chap06/notes`中工作，复制它并将其更改为`chap07/notes`。

让我们从回顾一下在 Node.js 中为什么数据库代码是异步的一些理论开始。

让我们开始吧！

# 第十章：记住数据存储需要异步代码

根据定义，外部数据存储系统需要异步编码技术，就像我们在前几章中讨论的那样。Node.js 架构的核心原则是，任何需要长时间执行的操作必须具有异步 API，以保持事件循环运行。从磁盘、另一个进程或数据库检索数据的访问时间总是需要足够的时间来要求延迟执行。

现有的`Notes`数据模型是一个内存数据存储。理论上，内存数据访问不需要异步代码，因此现有的模型模块可以使用常规函数，而不是`async`函数。

我们知道`Notes`应该使用数据库，并且需要一个异步 API 来访问`Notes`数据。因此，现有的`Notes`模型 API 使用`async`函数，所以在本章中，我们可以将 Notes 数据持久化到数据库中。

这是一个有用的复习。现在让我们谈谈生产应用程序所需的一个管理细节——使用日志系统来存储使用数据。

# 记录和捕获未捕获的错误

在我们进入数据库之前，我们必须解决高质量 Web 应用程序的一个属性——管理记录信息，包括正常系统活动、系统错误和调试信息。日志为开发人员提供了对系统行为的洞察。它们为开发人员回答以下问题：

+   应用程序的流量有多大？

+   如果是一个网站，人们最常访问哪些页面？

+   发生了多少错误，以及是什么类型的错误？是否发生了攻击？是否发送了格式不正确的请求？

日志管理也是一个问题。如果管理不当，日志文件很快就会填满磁盘空间。因此，在删除旧日志之前，处理旧日志变得非常重要，希望在删除旧日志之前提取有用的数据。通常，这包括**日志轮换**，即定期将现有日志文件移动到存档目录，然后开始一个新的日志文件。之后，可以进行处理以提取有用的数据，如错误或使用趋势。就像您的业务分析师每隔几周查看利润/损失报表一样，您的 DevOps 团队需要各种报告，以了解是否有足够的服务器来处理流量。此外，可以对日志文件进行筛查以查找安全漏洞。

当我们使用 Express 生成器最初创建`Notes`应用程序时，它使用以下代码配置了一个活动日志系统，使用了`morgan`：

```

This module is what prints messages about HTTP requests on the terminal window. We'll look at how to configure this in the next section.

Visit [`github.com/expressjs/morgan`](https://github.com/expressjs/morgan) for more information about `morgan`.

Another useful type of logging is debugging messages about an application. Debugging traces should be silent in most cases; they should only print information when debugging is turned on, and the level of detail should be configurable. 

The Express team uses the `debug` package for debugging logs. These are turned on using the `DEBUG` environment variable, which we've already seen in use. We will see how to configure this shortly and put it to use in the `Notes` application. For more information, refer to [`www.npmjs.com/package/debug`](https://www.npmjs.com/package/debug).

Finally, the application might generate uncaught exceptions or unhandled Promises. The `uncaughtException` and `unhandledRejection` errors must be captured, logged, and dealt with appropriately. We do not use the word *must* lightly; these errors *must* be handled.

Let's get started.

## Request logging with morgan

The `morgan` package generates log files from the HTTP traffic arriving on an Express application. It has two general areas for configuration:

*   Log format
*   Log location

As it stands, `Notes` uses the `dev` format, which is described as a concise status output for developers. This can be used to log web requests as a way to measure website activity and popularity. The Apache log format already has a large ecosystem of reporting tools and, sure enough, `morgan` can produce log files in this format. 

To enable changing the logging format, simply change the following line in `app.mjs`:

```

这是我们在整本书中遵循的模式；即将默认值嵌入应用程序，并使用环境变量来覆盖默认值。如果我们没有通过环境变量提供配置值，程序将使用`dev`格式。接下来，我们需要运行`Notes`，如下所示：

```

To revert to the previous logging output, simply do not set this environment variable. If you've looked at Apache access logs, this logging format will look familiar. The `::1` notation at the beginning of the line is IPV6 notation for `localhost`, which you may be more familiar with as `127.0.0.1`.

Looking at the documentation for `morgan`, we learn that it has several predefined logging formats available. We've seen two of them—the `dev` format is meant to provide developer-friendly information, while the `common` format is compatible with the Apache log format. In addition to these predefined formats, we can create a custom log format by using various tokens.

We could declare victory on request logging and move on to debugging messages. However, let's look at logging directly to a file. While it's possible to capture `stdout` through a separate process, `morgan` is already installed on `Notes` and it provides the capability to direct its output to a file.

The `morgan` documentation suggests the following:

```

然而，这存在一个问题；无法在不关闭和重新启动服务器的情况下执行日志轮换。术语“日志轮换”指的是 DevOps 实践，其中每个快照覆盖了几小时的活动。通常，应用服务器不会持续打开文件句柄到日志文件，DevOps 团队可以编写一个简单的脚本，每隔几个小时运行一次，并使用`mv`命令移动日志文件，使用`rm`命令删除旧文件。不幸的是，`morgan`在这里配置时，会持续打开文件句柄到日志文件。

相反，我们将使用`rotating-file-stream`包。这个包甚至自动化了日志轮换任务，这样 DevOps 团队就不必为此编写脚本。

有关此内容的文档，请参阅包页面[`www.npmjs.com/package/rotating-file-stream`](https://www.npmjs.com/package/rotating-file-stream)。

首先，安装包：

```

Then, add the following code to `app.mjs`:

```

在顶部的`import`部分，我们将`rotating-file-stream`加载为`rfs`。如果设置了`REQUEST_LOG_FILE`环境变量，我们将把它作为要记录的文件名。`morgan`的`stream`参数只需接受一个可写流。如果`REQUEST_LOG_FILE`没有设置，我们使用`?:`运算符将`process.stdout`的值作为可写流。如果设置了，我们使用`rfs.createStream`创建一个可写流，通过`rotating-file-stream`模块处理日志轮换。

在`rfs.createStream`中，第一个参数是日志文件的文件名，第二个是描述要使用的行为的`options`对象。这里提供了一套相当全面的选项。这里的配置在日志文件达到 10 兆字节大小（或 1 天后）时进行日志轮换，并使用`gzip`算法压缩旋转的日志文件。

可以设置多个日志。例如，如果我们想要将日志记录到控制台，除了记录到文件中，我们可以添加以下`logger`声明：

```

If the `REQUEST_LOG_FILE` variable is set, the other logger will direct logging to the file. Then, because the variable is set, this logger will be created and will direct logging to the console. Otherwise, if the variable is not set, the other logger will send logging to the console and this logger will not be created.

We use these variables as before, specifying them on the command line, as follows:

```

使用这个配置，将在`log.txt`中创建一个 Apache 格式的日志。在进行一些请求后，我们可以检查日志：

```

As expected, our log file has entries in Apache format. Feel free to add one or both of these environment variables to the script in `package.json` as well.

We've seen how to make a log of the HTTP requests and how to robustly record it in a file. Let's now discuss how to handle debugging messages.

## Debugging messages

How many of us debug our programs by inserting `console.log` statements? Most of us do. Yes, we're supposed to use a debugger, and yes, it is a pain to manage the `console.log` statements and make sure they're all turned off before committing our changes. The `debug` package provides a better way to handle debug tracing, which is quite powerful.

For the documentation on the `debug` package, refer to [`www.npmjs.com/package/debug`](https://www.npmjs.com/package/debug).

The Express team uses `DEBUG` internally, and we can generate quite a detailed trace of what Express does by running `Notes` this way:

```

如果要调试 Express，这非常有用。但是，我们也可以在我们自己的代码中使用这个。这类似于插入`console.log`语句，但无需记住注释掉调试代码。

要在我们的代码中使用这个，需要在任何想要调试输出的模块顶部添加以下声明：

```

This creates two functions—`debug` and `dbgerror`—which will generate debugging traces if enabled. The Debug package calls functions *debuggers*. The debugger named `debug` has a `notes:debug` specifier, while `dbgerror` has a `notes:error` specifier. We'll talk in more detail about specifiers shortly.

Using these functions is as simple as this:

```

当为当前模块启用调试时，这会导致消息被打印出来。如果当前模块未启用调试，则不会打印任何消息。再次强调，这类似于使用`console.log`，但您可以动态地打开和关闭它，而无需修改您的代码，只需适当设置`DEBUG`变量。

`DEBUG`环境变量包含描述哪些代码将启用调试的标识符。最简单的标识符是`*`，它是一个通配符，可以打开每个调试器。否则，调试标识符使用`identifer:identifier`格式。当我们说要使用`DEBUG=express:*`时，该标识符使用`express`作为第一个标识符，并使用`*`通配符作为第二个标识符。

按照惯例，第一个标识符应该是您的应用程序或库的名称。因此，我们之前使用`notes:debug`和`notes:error`作为标识符。但是，这只是一个惯例；您可以使用任何您喜欢的标识符格式。

要向`Notes`添加调试，让我们添加一些代码。将以下内容添加到`app.mjs`的底部：

```

This is adapted from the `httpsniffer.mjs` example from Chapter 4, *HTTP Servers and Clients*, and for every HTTP request, a little bit of information will be printed.

Then, in `appsupport.mjs`, let's make two changes. Add the following to the top of the `onError` function:

```

这将在 Express 捕获的任何错误上输出错误跟踪。

然后，将`onListening`更改为以下内容：

```

This changes the `console.log` call to a `debug` call so that a `Listening on` message is printed only if debugging is enabled.

If we run the application with the `DEBUG` variable set appropriately, we get the following output:

```

仔细看一下，你会发现输出既是来自`morgan`的日志输出，也是来自`debug`模块的调试输出。在这种情况下，调试输出以`notes:debug`开头。由于`REQUEST_LOG_FORMAT`变量，日志输出是以 Apache 格式的。

我们现在有一个准备好使用的调试跟踪系统。下一个任务是看看是否可能在文件中捕获这个或其他控制台输出。

## 捕获 stdout 和 stderr

重要消息可以打印到`process.stdout`或`process.stderr`，如果您不捕获输出，这些消息可能会丢失。最佳做法是捕获这些输出以供将来分析，因为其中可能包含有用的调试信息。更好的做法是使用系统设施来捕获这些输出流。

**系统设施**可以包括启动应用程序并将标准输出和标准错误流连接到文件的进程管理应用程序。

尽管它缺乏这种设施，但事实证明，在 Node.js 中运行的 JavaScript 代码可以拦截`process.stdout`和`process.stderr`流。在可用的包中，让我们看看`capture-console`。对于可写流，该包将调用您提供的回调函数来处理任何输出。

请参考`capture-console`包页面，了解相关文档：[`www.npmjs.com/package/capture-console`](https://www.npmjs.com/package/capture-console)。

最后一个行政事项是确保我们捕获其他未捕获的错误。

## 捕获未捕获的异常和未处理的拒绝的 Promises

未捕获的异常和未处理的拒绝的 Promises 是其他重要信息可能丢失的地方。由于我们的代码应该捕获所有错误，任何未捕获的错误都是我们的错误。如果我们不捕获这些错误，我们的失败分析可能会缺少重要信息。

Node.js 通过进程对象发送的事件指示这些条件，`uncaughtException`和`unhandledRejection`。在这些事件的文档中，Node.js 团队严厉地表示，在任何一种情况下，应用程序都处于未知状态，因为某些事情失败了，可能不安全继续运行应用程序。

要实现这些处理程序，请将以下内容添加到`appsupport.mjs`中：

```

Because these are events that are emitted from the `process` object, the way to handle them is to attach an event listener to these events. That's what we've done here.

The names of these events describe their meaning well. An `uncaughtException` event means an error was thrown but was not caught by a `try/catch` construct. Similarly, an `unhandledRejection` event means a Promise ended in a rejected state, but there was no `.catch` handler.

Our DevOps team will be happier now that we've handled these administrative chores. We've seen how to generate useful log files for HTTP requests, how to implement debug tracing, and even how to capture it to a file. We wrapped up this section by learning how to capture otherwise-uncaught errors.

We're now ready to move on to the real purpose of this chapter—storing notes in persistent storage, such as in a database. We'll implement support for several database systems, starting with a simple system using files on a disk.

# Storing notes in a filesystem

Filesystems are an often-overlooked database engine. While filesystems don't have the sort of query features supported by database engines, they are still a reliable place to store files. The Notes schema is simple enough, so the filesystem can easily serve as its data storage layer.

Let's start by adding two functions to the `Note` class in `models/Notes.mjs`:

```

我们将使用这个将`Note`对象转换为 JSON 格式的文本，以及从 JSON 格式的文本转换为`Note`对象。

`JSON`方法是一个 getter，这意味着它检索对象的值。在这种情况下，`note.JSON`属性/getter（没有括号）将简单地给我们提供笔记的 JSON 表示。我们稍后将使用它来写入 JSON 文件。

`fromJSON` 是一个静态函数，或者工厂方法，用于帮助构造 `Note` 对象，如果我们有一个 JSON 字符串。由于我们可能会得到任何东西，我们需要仔细测试输入。首先，如果字符串不是 JSON 格式，`JSON.parse` 将失败并抛出异常。其次，我们有 TypeScript 社区所谓的**类型保护**，或者 `if` 语句，来测试对象是否符合 `Note` 对象所需的条件。这检查它是否是一个带有 `key`、`title` 和 `body` 字段的对象，这些字段都必须是字符串。如果对象通过了这些测试，我们使用数据来构造一个 `Note` 实例。

这两个函数可以如下使用：

```

This example code snippet produces a simple `Note` instance and then generates the JSON version of the note. Then, a new note is instantiated from that JSON string using `from JSON()`.

Now, let's create a new module, `models/notes-fs.mjs`, to implement the filesystem datastore:

```

这导入了所需的模块；一个额外的添加是使用 `fs-extra` 模块。这个模块被用来实现与核心 `fs` 模块相同的 API，同时添加了一些有用的额外函数。在我们的情况下，我们对 `fs.ensureDir` 感兴趣，它验证指定的目录结构是否存在，如果不存在，则创建一个目录路径。如果我们不需要 `fs.ensureDir`，我们将简单地使用 `fs.promises`，因为它也提供了在 `async` 函数中有用的文件系统函数。

有关 `fs-extra` 的文档，请参考 [`www.npmjs.com/package/fs-extra`](https://www.npmjs.com/package/fs-extra)。

现在，将以下内容添加到 `models/notes-fs.mjs` 中：

```

The `FSNotesStore` class is an implementation of `AbstractNotesStore`, with a focus on storing the `Note` instances as JSON in a directory. These methods implement the API that we defined in Chapter 5, *Your First Express Application*. This implementation is incomplete since a couple of helper functions still need to be written, but you can see that it relies on files in the filesystem. For example, the `destroy` method simply uses `fs.unlink` to delete the note from the disk. In `keylist`, we use `fs.readdir` to read each `Note` object and construct an array of keys for the notes.

Let's add the helper functions:

```

`crupdate` 函数用于支持 `update` 和 `create` 方法。对于这个 `Notes` 存储，这两种方法都是相同的，它们将内容写入磁盘作为一个 JSON 文件。

代码中，笔记存储在由 `notesDir` 函数确定的目录中。这个目录可以在 `NOTES_FS_DIR` 环境变量中指定，也可以在 `Notes` 根目录中的 `notes-fs-data` 中指定（从 `approotdir` 变量中得知）。无论哪种方式，我们都使用 `fs.ensureDir` 来确保目录存在。

`Notes` 的路径名是由 `filePath` 函数计算的。

由于路径名是 `${notesDir}/${key}.json`，因此键不能使用文件名中不能使用的字符。因此，如果键包含 `/` 字符，`crupdate` 将抛出错误。

`readJSON` 函数的功能与其名称所示的一样——它从磁盘中读取一个 `Note` 对象作为 JSON 文件。

我们还添加了另一个依赖项：

```

We're now almost ready to run the `Notes` application, but there's an issue that first needs to be resolved with the `import()` function.

## Dynamically importing ES6 modules

Before we start modifying the router functions, we have to consider how to account for multiple `AbstractNotesStore` implementations. By the end of this chapter, we will have several of them, and we want an easy way to configure `Notes` to use any of them. For example, an environment variable, `NOTES_MODEL`, could be used to specify the `Notes` data model to use, and the `Notes` application would dynamically load the correct module.

In `Notes`, we refer to the `Notes` datastore module from several places. To change from one datastore to another requires changing the source in each of these places. It would be better to locate that selection in one place, and further, to make it dynamically configurable at runtime.

There are several possible ways to do this. For example, in a CommonJS module, it's possible to compute the pathname to the module for a `require` statement. It would consult the environment variable, `NOTES_MODEL`, to calculate the pathname for the datastore module, as follows:

```

然而，我们的意图是使用 ES6 模块，因此让我们看看在这种情况下它是如何工作的。因为在常规的 `import` 语句中，模块名不能像这样是一个表达式，所以我们需要使用 `动态导入` 来加载模块。`动态导入` 功能——即 `import()` 函数——允许我们动态计算要加载的模块名。

为了实现这个想法，让我们创建一个新文件 `models/notes-store.mjs`，其中包含以下内容：

```

This is what we might call a factory function. It uses `import()` to load a module whose filename is calculated from the `model` parameter. We saw in `notes-fs.mjs` that the `FSNotesStore` class is the default export. Therefore, the `NotesStoreClass` variable gets that class, then we call the constructor to create an instance, and then we stash that instance in a global scope variable. That global scope variable is then exported as `NotesStore`.

We need to make one small change in `models/notes-memory.mjs`:

```

任何实现 `AbstractNotesStore` 的模块都将默认导出定义的类。

在 `app.mjs` 中，我们需要对调用这个 `useModel` 函数进行另一个更改。在第五章中，*你的第一个 Express 应用程序*，我们让 `app.mjs` 导入 `models/notes-memory.mjs`，然后设置 `NotesStore` 包含 `InMemoryNotesStore` 的一个实例。具体来说，我们有以下内容：

```

We need to remove these two lines of code from `app.mjs` and then add the following:

```

我们导入 `useModel`，将其重命名为 `useNotesModel`，然后通过传入 `NOTES_MODEL` 环境变量来调用它。如果 `NOTES_MODEL` 变量未设置，我们将默认使用“memory” `NotesStore`。由于 `useNotesModel` 是一个 `async` 函数，我们需要处理生成的 Promise。`.then` 处理成功的情况，但由于没有需要执行的操作，所以我们提供了一个空函数。重要的是任何错误都会关闭应用程序，因此我们添加了 `.catch`，它调用 `onError` 来处理错误。

为了支持这个错误指示器，我们需要在 `appsupport.mjs` 的 `onError` 函数中添加以下内容：

```

This added error handler will also cause the application to exit.

These changes also require us to make another change. The `NotesStore` variable is no longer in `app.mjs`, but is instead in `models/notes-store.mjs`. This means we need to go to `routes/index.mjs` and `routes/notes.mjs`, where we make the following change to the imports:

```

我们从`notes-store.mjs`中导入`NotesStore`导出，并将其重命名为`notes`。因此，在两个路由模块中，我们将进行诸如`notes.keylist()`的调用，以访问动态选择的`AbstractNotesStore`实例。

这种抽象层提供了期望的结果——设置一个环境变量，让我们在运行时决定使用哪个数据存储。

现在我们已经拥有了所有的部件，让我们运行`Notes`应用程序并看看它的行为。

## 使用文件系统存储运行 Notes 应用程序

在`package.json`中，将以下内容添加到`scripts`部分：

```

When you add these entries to `package.json`, make sure you use the correct JSON syntax. In particular, if you leave a comma at the end of the `scripts` section, it will fail to parse and `npm` will throw an error message.

With this code in place, we can now run the `Notes` application, as follows:

```

我们可以像以前一样在`http://localhost:3000`上使用应用程序。因为我们没有更改任何模板或 CSS 文件，所以应用程序看起来与您在第六章结束时一样。

因为`notes:*`的调试已打开，我们将看到`Notes`应用程序正在执行的任何操作的日志。通过简单地不设置`DEBUG`变量，可以轻松关闭此功能。

您现在可以关闭并重新启动`Notes`应用程序，并查看完全相同的注释。您还可以使用常规文本编辑器（如**vi**）在命令行中编辑注释。您现在可以在不同端口上启动多个服务器，使用`fs-server1`和`fs-server2`脚本，并查看完全相同的注释。

就像我们在第五章结束时所做的那样，*您的第一个 Express 应用程序*，我们可以在两个单独的命令窗口中启动两个服务器。这将在不同的端口上运行两个应用程序实例。然后，在不同的浏览器窗口中访问这两个服务器，您会发现两个浏览器窗口显示相同的注释。

另一个尝试的事情是指定`NOTES_FS_DIR`以定义一个不同的目录来存储注释。

最后的检查是创建一个带有`/`字符的键的注释。请记住，键用于生成我们存储注释的文件名，因此键不能包含`/`字符。在浏览器打开的情况下，单击“添加注释”，并输入一条注释，确保在“键”字段中使用`/`字符。单击提交按钮后，您将看到一个错误，指出这是不允许的。

我们现在已经演示了向`Notes`添加持久数据存储。但是，这种存储机制并不是最好的，还有其他几种数据库类型可以探索。我们列表中的下一个数据库服务是 LevelDB。

# 使用 LevelDB 数据存储存储注释

要开始使用实际数据库，让我们看一下一个极其轻量级、占用空间小的数据库引擎：`level`。这是一个 Node.js 友好的包装器，它包装了 LevelDB 引擎，并由 Google 开发。它通常用于 Web 浏览器进行本地数据持久化，并且是一个非索引的 NoSQL 数据存储，最初是为在浏览器中使用而设计的。Level Node.js 模块使用 LevelDB API，并支持多个后端，包括 leveldown，它将 C++ LevelDB 数据库集成到 Node.js 中。

访问[`www.npmjs.com/package/level`](https://www.npmjs.com/package/level)了解有关此模块的信息。

要安装数据库引擎，请运行以下命令：

```

This installs the version of `level` that the following code was written against.

Then, create the `models/notes-level.mjs` module, which will contain the `AbstractNotesStore` implementation:

```

我们从`import`语句和一些声明开始模块。`connectDB`函数用于连接数据库，`createIfMissing`选项也是如其名所示，如果不存在具有所使用名称的数据库，则创建一个数据库。从模块`level`导入的是一个构造函数，用于创建与第一个参数指定的数据库连接的`level`实例。这个第一个参数是文件系统中的位置，换句话说，是数据库将被存储的目录。

`level`构造函数通过返回一个`db`对象来与数据库进行交互。我们将`db`作为模块中的全局变量存储，以便于使用。在`connectDB`中，如果`db`对象已经设置，我们立即返回它；否则，我们使用构造函数打开数据库，就像刚才描述的那样。

数据库的位置默认为当前目录中的`notes.level`。`LEVELDB_LOCATION`环境变量可以设置，如其名称所示，以指定数据库位置。

现在，让我们添加这个模块的其余部分：

```

As expected, we're creating a `LevelNotesStore` class to hold the functions.  

In this case, we have code in the `close` function that calls `db.close` to close down the connection. The `level` documentation suggests that it is important to close the connection, so we'll have to add something to `app.mjs` to ensure that the database closes when the server shuts down. The documentation also says that `level` does not support concurrent connections to the same database from multiple clients, meaning if we want multiple `Notes` instances to use the database, we should only have the connection open when necessary.

Once again, there is no difference between the `create` and `update` operations, and so we use a `crupdate` function again. Notice that the pattern in all the functions is to first call `connectDB` to get `db`, and then to call a function on the `db` object. In this case, we use `db.put` to store the `Note` object in the database.

In the `read` function, `db.get` is used to read the note. Since the `Note` data was stored as JSON, we use `Note.fromJSON` to decode and instantiate the `Note` instance.

The `destroy` function deletes a record from the database using the `db.del` function.

Both `keylist` and `count` use the `createKeyStream` function. This function uses an event-oriented interface to stream through every database entry, emitting events as it goes. A `data` event is emitted for each key in the database, while the `end` event is emitted at the end of the database, and the `error` event is emitted on errors. Since there is no simple way to present this as a simple `async` function, we have wrapped it with a Promise so that we can use `await`. We then invoke `createKeyStream`, letting it run its course and collect data as it goes. For `keylist`, in the `data` events, we add the data (in this case, the key to a database entry) to an array. 

For `count`, we use a similar process, and in this case, we simply increment a counter. Since we have this wrapped in a Promise, in an `error` event, we call `reject`, and in an `end` event, we call `resolve`.

Then, we add the following to `package.json` in the `scripts` section:

```

最后，您可以运行`Notes`应用程序：

```

The printout in the console will be the same, and the application will also look the same. You can put it through its paces to check whether everything works correctly.

Since `level` does not support simultaneous access to a database from multiple instances, you won't be able to use the multiple `Notes` application scenario. You will, however, be able to stop and restart the application whenever you want to without losing any notes.

Before we move on to looking at the next database, let's deal with a issue mentioned earlier—closing the database connection when the process exits.

## Closing database connections when closing the process

The `level` documentation says that we should close the database connection with `db.close`. Other database servers may well have the same requirement. Therefore, we should make sure we close the database connection before the process exits, and perhaps also on other conditions.

Node.js provides a mechanism to catch signals sent by the operating system. What we'll do is configure listeners for these events, then close `NotesStore` in response.

 Add the following code to `appsupport.mjs`:

```

我们导入`NotesStore`以便可以调用其方法，`server`已经在其他地方导入。

前三个`process.on`调用监听操作系统信号。如果您熟悉 Unix 进程信号，这些术语会很熟悉。在每种情况下，事件调用`catchProcessDeath`函数，然后调用`NotesStore`和`server`上的`close`函数，以确保关闭。

然后，为了确认一些事情，我们附加了一个`exit`监听器，这样当进程退出时我们可以打印一条消息。Node.js 文档表示，`exit`监听器被禁止执行需要进一步事件处理的任何操作，因此我们不能在此处理程序中关闭数据库连接。

让我们试一下运行`Notes`应用程序，然后立即按下*Ctrl* + *C*：

```

Sure enough, upon pressing *Ctrl* + *C*, the `exit` and `catchProcessDeath` listeners are called.

That covers the `level` database, and we also have the beginning of a handler to gracefully shut down the application. The next database to cover is an embedded SQL database that requires no server processes.

# Storing notes in SQL with SQLite3

To get started with more normal databases, let's see how we can use SQL from Node.js. First, we'll use SQLite3, which is a lightweight, simple-to-set-up database engine eminently suitable for many applications.

To learn more about this database engine, visit [`www.sqlite.org/`](http://www.sqlite.org/).

To learn more about the Node.js module, visit [`github.com/mapbox/node-sqlite3/wiki/API`](https://github.com/mapbox/node-sqlite3/wiki/API) or [`www.npmjs.com/package/sqlite3`](https://www.npmjs.com/package/sqlite3).

The primary advantage of SQLite3 is that it doesn't require a server; it is a self-contained, no-set-up-required SQL database. The SQLite3 team also claims that it is very fast and that large, high-throughput applications have been built with it. The downside to the SQLite3 package is that its API requires callbacks, so we'll have to use the Promise wrapper pattern.

The first step is to install the module:

```

当然，这会安装`sqlite3`包。

要管理 SQLite3 数据库，您还需要安装 SQLite3 命令行工具。该项目网站为大多数操作系统提供了预编译的二进制文件。您还会发现这些工具在大多数软件包管理系统中都是可用的。

我们可以使用的一个管理任务是设置数据库表，我们将在下一节中看到。

## SQLite3 数据库模式

接下来，我们需要确保我们的数据库配置了适合`Notes`应用程序的数据库表。这是上一节末尾提到的一个示例数据库管理员任务。为此，我们将使用`sqlite3`命令行工具。`sqlite3.org`网站有预编译的二进制文件，或者该工具可以通过您的操作系统的软件包管理系统安装——例如，您可以在 Ubuntu/Debian 上使用`apt-get`，在 macOS 上使用 MacPorts。

对于 Windows，请确保已经安装了 Chocolatey 软件包管理工具，然后以管理员权限启动 PowerShell，并运行"`choco install sqlite`"。这将安装 SQLite3 的 DLL 和其命令行工具，让您可以运行以下指令。

我们将使用以下的 SQL 表定义作为模式（将其保存为`models/schema-sqlite3.sql`）：

```

To initialize the database table, we run the following command:

```

虽然我们可以这样做，但最佳实践是自动化所有管理过程。为此，我们应该编写一小段脚本来初始化数据库。

幸运的是，`sqlite3`命令为我们提供了一种方法来做到这一点。将以下内容添加到`package.json`的`scripts`部分：

```

Run the setup script:

```

这并不是完全自动化，因为我们必须在`sqlite`提示符下按*Ctrl* + *D*，但至少我们不必费心去记住如何做。我们本可以轻松地编写一个小的 Node.js 脚本来做到这一点；然而，通过使用软件包提供的工具，我们在自己的项目中需要维护的代码更少。

有了数据库表的设置，让我们继续编写与 SQLite3 交互的代码。

## SQLite3 模型代码

我们现在准备为 SQLite3 实现一个`AbstractNotesStore`实现。

创建`models/notes-sqlite3.mjs`文件：

```

This imports the required packages and makes the required declarations. The `connectDB` function has a similar purpose to the one in `notes-level.mjs`: to manage the database connection. If the database is not open, it'll go ahead and open it, and it will even make sure that the database file is created (if it doesn't exist). If the database is already open, it'll simply be returned.

Since the API used in the `sqlite3` package requires callbacks, we will have to wrap every function call in a Promise wrapper, as shown here.

Now, add the following to `models/notes-sqlite3.mjs`:

```

由于有许多成员函数，让我们逐个讨论它们：

```

In `close`, the task is to close the database. There's a little dance done here to make sure the global `db` variable is unset while making sure we can close the database by saving `db` as `_db`. The `sqlite3` package will report errors from `db.close`, so we're making sure we report any errors:

```

我们现在有理由定义`Notes`模型的`create`和`update`操作是分开的，因为每个函数的 SQL 语句是不同的。`create`函数当然需要一个`INSERT INTO`语句，而`update`函数当然需要一个`UPDATE`语句。

`db.run`函数在这里使用了多次，它执行一个 SQL 查询，同时给我们机会在查询字符串中插入参数。

这遵循了 SQL 编程接口中常见的参数替换范式。程序员将 SQL 查询放在一个字符串中，然后在查询字符串中的任何位置放置一个问号，以便在查询字符串中插入一个值。查询字符串中的每个问号都必须与程序员提供的数组中的一个值匹配。该模块负责正确编码这些值，以便查询字符串格式正确，同时防止 SQL 注入攻击。

`db.run`函数只是运行它所给出的 SQL 查询，并不检索任何数据。

```

To retrieve data using the `sqlite3` module, you use the `db.get`, `db.all`, or `db.each` functions. Since our `read` method only returns one item, we use the `db.get` function to retrieve just the first row of the result set. By contrast, the `db.all` function returns all of the rows of the result set at once, and the `db.each` function retrieves one row at a time, while still allowing the entire result set to be processed.

By the way, this `read` function has a bug in it—see whether you can spot the error. We'll read more about this in Chapter 13, *Unit Testing and Functional Testing*, when our testing efforts uncover the bug:

```

在我们的`destroy`方法中，我们只需使用`db.run`执行`DELETE FROM`语句来删除相关笔记的数据库条目：

```

In `keylist`, the task is to collect the keys for all of the `Note` instances. As we said, `db.get` returns only the first entry of the result set, while the `db.all` function retrieves all the rows of the result set. Therefore, we use `db.all`, although `db.each` would have been a good alternative.

The contract for this function is to return an array of note keys. The `rows` object from `db.all` is an array of results from the database that contains the data we are to return, but we use the `map` function to convert the array into the format required by this function:

```

在`count`中，任务类似，但我们只需要表中行的计数。SQL 提供了一个`count()`函数来实现这个目的，我们已经使用了，然后因为这个结果只有一行，我们可以再次使用`db.get`。

这使我们能够使用`NOTES_MODEL`设置为`sqlite3`运行`Notes`。现在我们的代码已经设置好，我们可以继续使用这个数据库运行`Notes`。

## 使用 SQLite3 运行 Notes

我们现在准备使用 SQLite3 运行`Notes`应用程序。将以下代码添加到`package.json`的`scripts`部分：

```

This sets up the commands that we'll use to test `Notes` on SQLite3.

We can run the server as follows:

```

现在你可以在`http://localhost:3000`上浏览应用程序，并像以前一样运行它。

因为我们还没有对`View`模板或 CSS 文件进行任何更改，所以应用程序看起来和以前一样。

当然，你可以使用`sqlite`命令，或其他 SQLite3 客户端应用程序来检查数据库：

```

The advantage of installing the SQLite3 command-line tools is that we can perform any database administration tasks without having to write any code.  

We have seen how to use SQLite3 with Node.js. It is a worthy database for many sorts of applications, plus it lets us use a SQL database without having to set up a server.

The next package that we will cover is an **Object Relations Management** (**ORM**) system that can run on top of several SQL databases.

# Storing notes the ORM way with Sequelize

There are several popular SQL database engines, such as PostgreSQL, MySQL, and MariaDB. Corresponding to each are Node.js client modules that are similar in nature to the `sqlite3` module that we just used. The programmer is close to SQL, which can be good in the same way that driving a stick shift car is fun. But what if we want a higher-level view of the database so that we can think in terms of objects, rather than rows of a database table? **ORM** systems provide a suitable higher-level interface, and even offer the ability to use the same data model with several databases. Just as driving an electric car provides lots of benefits at the expense of losing out on the fun of stick-shift driving, ORM produces lots of benefits, while also distancing ourselves from the SQL.

The **Sequelize** package ([`www.sequelizejs.com/`](http://www.sequelizejs.com/)) is Promise-based, offers strong, well-developed ORM features, and can connect to SQLite3, MySQL, PostgreSQL, MariaDB, and MSSQL databases. Because Sequelize is Promise-based, it will fit naturally with the Promise-based application code we're writing.

A prerequisite to most SQL database engines is having access to a database server. In the previous section, we skirted around this issue by using SQLite3, which requires no database server setup. While it's possible to install a database server on your laptop, right now, we want to avoid the complexity of doing so, and so we will use Sequelize to manage a SQLite3 database. We'll also see that it's simply a matter of using a configuration file to run the same Sequelize code against a hosted database such as MySQL. In Chapter 11, *Deploying Node.js Microservices with Docker*, we'll learn how to use Docker to easily set up a service, including database servers, on our laptop and deploy the exact same configuration to a live server. Most web-hosting providers offer MySQL or PostgreSQL as part of their service.

Before we start on the code, let's install two modules:

```

第一个安装了 Sequelize 包。第二个`js-yaml`是安装的，以便我们可以实现一个以 YAML 格式存储 Sequelize 连接配置的文件。YAML 是一种人类可读的**数据序列化语言**，这意味着它是一种易于使用的文本文件格式，用于描述数据对象。

也许最好了解 YAML 的地方是它的维基百科页面，可以在[`en.wikipedia.org/wiki/YAML`](https://en.wikipedia.org/wiki/YAML)找到。

让我们从学习如何配置 Sequelize 开始，然后我们将为 Sequelize 创建一个`AbstractNotesStore`实例，最后，我们将使用 Sequelize 测试`Notes`。

## 配置 Sequelize 并连接到数据库

我们将以与以前不同的方式组织 Sequelize 支持的代码。我们预见到`Notes`表不是`Notes`应用程序将使用的唯一数据模型。我们可以支持其他功能，比如上传笔记的图片或允许用户评论笔记。这意味着需要额外的数据库表，并建立数据库条目之间的关系。例如，我们可能会有一个名为`AbstractCommentStore`的类来存储评论，它将有自己的数据库表和自己的模块来管理评论数据。`Notes`和`Comments`存储区域都应该在同一个数据库中，因此它们应该共享一个数据库连接。

有了这个想法，让我们创建一个文件`models/sequlz.mjs`，来保存管理 Sequelize 连接的代码：

```

As with the SQLite3 module, the `connectDB` function manages the connection through Sequelize to a database server. Since the configuration of the Sequelize connection is fairly complex and flexible, we're not using environment variables for the whole configuration, but instead we use a YAML-formatted configuration file that will be specified in an environment variable. Sequelize uses four items of data—the database name, the username, the password, and a parameters object.

When we read in a YAML file, its structure directly corresponds to the object structure that's created. Therefore, with a YAML configuration file, we don't need to use up any brain cells developing a configuration file format. The YAML structure is dictated by the Sequelize `params` object, and our configuration file simply has to use the same structure.

We also allow overriding any of the fields in this file using environment variables. This will be useful when we deploy `Notes` using Docker so that we can configure database connections without having to rebuild the Docker container.

For a simple SQLite3-based database, we can use the following YAML file for configuration and name it `models/sequelize-sqlite.yaml`:

```

`params.dialect`的值决定了要使用的数据库类型；在这种情况下，我们使用的是 SQLite3。根据方言的不同，`params`对象可以采用不同的形式，比如连接到数据库的连接 URL。在这种情况下，我们只需要一个文件名，就像这样给出的。

`authenticate` 调用是为了测试数据库是否正确连接。

`close` 函数做你期望的事情——关闭数据库连接。

有了这个设计，我们可以很容易地通过添加一个运行时配置文件来更改数据库以使用其他数据库服务器。例如，很容易设置一个 MySQL 连接；我们只需创建一个新文件，比如 `models/sequelize-mysql.yaml`，其中包含类似以下代码的内容：

```

This is straightforward. The `username` and `password` fields must correspond to the database credentials, while `host` and `port` will specify where the database is hosted. Set the database's `dialect` parameter and other connection information and you're good to go.

To use MySQL, you will need to install the base MySQL driver so that Sequelize can use MySQL:

```

运行 Sequelize 对其支持的其他数据库，如 PostgreSQL，同样简单。只需创建一个配置文件，安装 Node.js 驱动程序，并安装/配置数据库引擎。

从 `connectDB` 返回的对象是一个数据库连接，正如我们将看到的，它被 Sequelize 使用。因此，让我们开始这一部分的真正目标——定义 `SequelizeNotesStore` 类。

## 为 Notes 应用程序创建一个 Sequelize 模型

与我们使用的其他数据存储引擎一样，我们需要为 Sequelize 创建一个 `AbstractNotesStore` 的子类。这个类将使用 Sequelize `Model` 类来管理一组注释。

让我们创建一个新文件，`models/notes-sequelize.mjs`：

```

The database connection is stored in the `sequelize` object, which is established by the `connectDB` function that we just looked at (which we renamed `connectSequlz`) to instantiate a Sequelize instance. We immediately return if the database is already connected.

In Sequelize, the `Model` class is where we define the data model for a given object. Each `Model` class corresponds to a database table. The `Model` class is a normal ES6 class, and we start by subclassing it to define the `SQNote` class. Why do we call it `SQNote`? That's because we already defined a `Note` class, so we had to use a different name in order to use both classes.

By calling `SQNote.init`, we initialize the `SQNote` model with the fields—that is, the schema—that we want it to store. The first argument to this function is the schema description and the second argument is the administrative data required by Sequelize.

As you would expect, the schema has three fields: `notekey`, `title`, and `body`. Sequelize supports a long list of data types, so consult the documentation for more on that. We are using `STRING` as the type for `notekey` and `title` since both handle a short text string up to 255 bytes long. The `body` field is defined as `TEXT` since it does not need a length limit. In the `notekey` field, you see it is an object with other parameters; in this case, it is described as the primary key and the `notekey` values must be unique.

Online documentation can be found at the following locations:
Sequelize class: [`docs.sequelizejs.com/en/latest/api/sequelize/`](http://docs.sequelizejs.com/en/latest/api/sequelize/) [](http://docs.sequelizejs.com/en/latest/api/sequelize/) Defining models: [`docs.sequelizejs.com/en/latest/api/model/`](http://docs.sequelizejs.com/en/latest/api/model/)

That manages the database connection and sets up the schema. Now, let's add the `SequelizeNotesStore` class to `models/notes-sequelize.mjs`:

```

首先要注意的是，在每个函数中，我们调用在 `SQNote` 类中定义的静态方法来执行数据库操作。Sequelize 模型类就是这样工作的，它的文档中有一个全面的这些静态方法的列表。

在创建 Sequelize 模型类的新实例时——在本例中是 `SQNote`——有两种模式可供选择。一种是调用 `build` 方法，然后创建对象和 `save` 方法将其保存到数据库。或者，我们可以像这样使用 `create` 方法，它执行这两个步骤。此函数返回一个 `SQNote` 实例，在这里称为 `sqnote`，如果您查阅 Sequelize 文档，您将看到这些实例有一长串可用的方法。我们的 `create` 方法的约定是返回一个注释，因此我们构造一个 `Note` 对象来返回。

在这个和其他一些方法中，我们不想向调用者返回一个 Sequelize 对象。因此，我们构造了我们自己的 `Note` 类的实例，以返回一个干净的对象。

我们的 `update` 方法首先调用 `SQNote.findOne`。这是为了确保数据库中存在与我们给定的键对应的条目。此函数查找第一个数据库条目，其中 `notekey` 匹配提供的键。在快乐路径下，如果存在数据库条目，我们然后使用 `SQNote.update` 来更新 `title` 和 `body` 值，并通过使用相同的 `where` 子句，确保 `update` 操作针对相同的数据库条目。

Sequelize 的 `where` 子句提供了一个全面的匹配操作符列表。如果您仔细考虑这一点，很明显它大致对应于以下 SQL：

```

That's what Sequelize and other ORM libraries do—convert the high-level API into database operations such as SQL queries.

To read a note, we use the `findOne` operation again. There is the possibility of it returning an empty result, and so we have to throw an error to match. The contract for this function is to return a `Note` object, so we take the fields retrieved using Sequelize to create a clean `Note` instance.

To destroy a note, we use the `destroy` operation with the same `where` clause to specify which entry to delete. This means that, as in the equivalent SQL statement (`DELETE FROM SQNotes WHERE notekey = ?`), if there is no matching note, no error will be thrown.

Because the `keylist` function acts on all `Note` objects, we use the `findAll` operation. The difference between `findOne` and `findAll` is obvious from the names. While `findOne` returns the first matching database entry, `findAll` returns all of them. The `attributes` specifier limits the result set to include the named field—namely, the `notekey` field. This gives us an array of objects with a field named `notekey`. We then use a `.map` function to convert this into an array of note keys.

For the `count` function, we can just use the `count()` method to calculate the required result.

This allows us to use Sequelize by setting `NOTES_MODEL` to `sequelize`.

Having set up the functions to manage the database connection and defined the `SequelizeNotesStore` class, we're now ready to test the `Notes` application.

## Running the Notes application with Sequelize

Now, we can get ready to run the `Notes` application using Sequelize. We can run it against any database server, but let's start with SQLite3\. Add the following declarations to the `scripts` entry in `package.json`:

```

这设置了命令以运行单个服务器实例（或两个）。

然后，按以下方式运行它：

```

As before, the application looks exactly the same because we haven't changed the `View` templates or CSS files. Put it through its paces and everything should work.

You will be able to start two instances; use separate browser windows to visit both instances and see whether they show the same set of notes.

To reiterate, to use the Sequelize-based model on a given database server, do the following:

1.  Install and provision the database server instance; otherwise, get the connection parameters for an already-provisioned database server.
2.  Install the corresponding Node.js driver.
3.  Write a YAML configuration file corresponding to the connection parameters.
4.  Create new `scripts` entries in `package.json` to automate starting `Notes` against the database.

By using Sequelize, we have dipped our toes into a powerful library for managing data in a database. Sequelize is one of several ORM libraries available for Node.js. We've already used the word *comprehensive* several times in this section as it's definitely the best word to describe Sequelize. 

An alternative that is worthy of exploration is not an ORM library but is what's called a query builder. `knex` supports several SQL databases, and its role is to simplify creating SQL queries by using a high-level API.

In the meantime, we have one last database to cover before wrapping up this chapter: MongoDB, the leading NoSQL database.

# Storing notes in MongoDB

MongoDB is widely used with Node.js applications, a sign of which is the popular **MEAN** acronym: **MongoDB (or MySQL), Express, Angular, and Node.js**. MongoDB is one of the leading NoSQL databases, meaning it is a database engine that does not use SQL queries. It is described as a *scalable, high-performance, open source, document-oriented database*. It uses JSON-style documents with no predefined, rigid schema and a large number of advanced features. You can visit their website for more information and documentation at [`www.mongodb.org`](http://www.mongodb.org).

Documentation on the Node.js driver for MongoDB can be found at [`www.npmjs.com/package/mongodb`](https://www.npmjs.com/package/mongodb) and [`mongodb.github.io/node-mongodb-native/`](http://mongodb.github.io/node-mongodb-native/).

Mongoose is a popular ORM for MongoDB ([`mongoosejs.com/`](http://mongoosejs.com/)). In this section, we'll use the native MongoDB driver instead, but Mongoose is a worthy alternative.

First, you will need a running MongoDB instance. The Compose- ([`www.compose.io/`](https://www.compose.io/)) and ScaleGrid- ([`scalegrid.io/`](https://scalegrid.io/)) hosted service providers offer hosted MongoDB services. Nowadays, it is straightforward to host MongoDB as a Docker container as part of a system built of other Docker containers. We'll do this in Chapter 13, *Unit Testing and Functional Testing*.

It's possible to set up a temporary MongoDB instance for testing on, say, your laptop. It is available in all the operating system package management systems, or you can download a compiled package from [mongodb.com](https://www.mongodb.com). The MongoDB website also has instructions ([`docs.mongodb.org/manual/installation/`](https://docs.mongodb.org/manual/installation/)).

For Windows, it may be most expedient to use a cloud-hosted MongoDB instance.

Once installed, it's not necessary to set up MongoDB as a background service. Instead, you can run a couple of simple commands to get a MongoDB instance running in the foreground of a command window, which you can kill and restart any time you like.

In a command window, run the following:

```

这将创建一个数据目录，然后运行 MongoDB 守护程序来对该目录进行操作。

在另一个命令窗口中，您可以按以下方式进行测试：

```

This runs the Mongo client program with which you can run commands. The command language used here is JavaScript, which is comfortable for us.

This saves a *document* in the collection named `foo`. The second command finds all documents in `foo`, printing them out for you. There is only one document, the one we just inserted, so that's what gets printed. The `_id` field is added by MongoDB and serves as a document identifier.

This setup is useful for testing and debugging. For a real deployment, your MongoDB server must be properly installed on a server. See the MongoDB documentation for these instructions.

With a working MongoDB installation in our hands, let's get started with implementing the `MongoNotesStore` class.

## A MongoDB model for the Notes application

The official Node.js MongoDB driver ([`www.npmjs.com/package/mongodb`](https://www.npmjs.com/package/mongodb)) is created by the MongoDB team. It is very easy to use, as we will see, and its installation is as simple as running the following command:

```

这为我们设置了驱动程序包，并将其添加到 `package.json`。

现在，创建一个新文件，`models/notes-mongodb.mjs`：

```

This sets up the required imports, as well as the functions to manage a connection with the MongoDB database.

The `MongoClient` class is used to connect with a MongoDB instance. The required URL, which will be specified through an environment variable, uses a straightforward format: `mongodb://localhost/`. The database name is specified via another environment variable.

The documentation for the MongoDB Node.js driver can be found at [`mongodb.github.io/node-mongodb-native/`](http://mongodb.github.io/node-mongodb-native/).

There are both reference and API documentation available. In the *API* section, the `MongoClient` and `Db` classes are the ones that most relate to the code we are writing ([`mongodb.github.io/node-mongodb-native/`](http://mongodb.github.io/node-mongodb-native/)).

The `connectDB` function creates the database client object. This object is only created as needed. The connection URL is provided through the `MONGO_URL` environment variable.

The `db` function is a simple wrapper around the client object to access the database that is used for the `Notes` application, which we specify via the `MONGO_DBNAME` environment variable. Therefore, to access the database, the code will have to call `db().mongoDbFunction()`.

Now, we can implement the `MongoDBNotesStore` class:

```

MongoDB 将所有文档存储在集合中。*集合* 是一组相关文档，类似于关系数据库中的表。这意味着创建一个新文档或更新现有文档始于将其构造为 JavaScript 对象，然后要求 MongoDB 将对象保存到数据库中。MongoDB 自动将对象编码为其内部表示形式。

`db().collection` 方法为我们提供了一个 `Collection` 对象，我们可以使用它来操作命名集合。在这种情况下，我们使用 `db().collection('notes')` 访问 `notes` 集合。

有关 `Collection` 类的文档，请参阅之前引用的 MongoDB Node.js 驱动程序文档。

在`create`方法中，我们使用`insertOne`；顾名思义，它将一个文档插入到集合中。这个文档用于`Note`类的字段。同样，在`update`方法中，`updateOne`方法首先找到一个文档（在这种情况下，通过查找具有匹配`notekey`字段的文档），然后根据指定的内容更改文档中的字段，然后将修改后的文档保存回数据库。

`read`方法使用`db().findOne`来搜索笔记。

`findOne`方法采用所谓的*查询选择器*。在这种情况下，我们要求与`notekey`字段匹配。MongoDB 支持一套全面的查询选择器操作符。

另一方面，`updateOne`方法采用所谓的*查询过滤器*。作为一个`update`操作，它在数据库中搜索与过滤器匹配的记录，根据更新描述符更新其字段，然后将其保存回数据库。

关于 MongoDB CRUD 操作的概述，包括插入文档、更新文档、查询文档和删除文档，请参阅[`docs.mongodb.com/manual/crud/`](https://docs.mongodb.com/manual/crud/)。

有关查询选择器的文档，请参阅[`docs.mongodb.com/manual/reference/operator/query/#query-selectors`](https://docs.mongodb.com/manual/reference/operator/query/#query-selectors)。

有关查询过滤器的文档，请参阅[`docs.mongodb.com/manual/core/document/#query-filter-documents`](https://docs.mongodb.com/manual/core/document/#query-filter-documents)。

有关更新描述符的文档，请参阅[`docs.mongodb.com/manual/reference/operator/update/`](https://docs.mongodb.com/manual/reference/operator/update/)。

MongoDB 有许多基本操作的变体。例如，`findOne`是基本`find`方法的一个变体。

在我们的`destroy`方法中，我们看到另一个`find`变体，`findOneAndDelete`。顾名思义，它查找与查询描述符匹配的文档，然后删除该文档。

在`keylist`方法中，我们需要处理集合中的每个文档，因此`find`查询选择器为空。`find`操作返回一个`Cursor`，这是一个用于导航查询结果的对象。`Cursor.forEach`方法采用两个回调函数，不是一个 Promise 友好的操作，因此我们必须使用一个 Promise 包装器。第一个回调函数对查询结果中的每个文档都会调用，而在这种情况下，我们只是将`notekey`字段推送到一个数组中。第二个回调函数在操作完成时调用，并且我们通知 Promise 它是成功还是失败。这给我们了我们的键数组，它返回给调用者。

有关`Cursor`类的文档，请参阅[`mongodb.github.io/node-mongodb-native/3.1/api/Cursor.html`](http://mongodb.github.io/node-mongodb-native/3.1/api/Cursor.html)。

在我们的`count`方法中，我们简单地调用 MongoDB 的`count`方法。`count`方法采用查询描述符，并且顾名思义，计算与查询匹配的文档数量。由于我们给出了一个空的查询选择器，它最终计算整个集合。

这使我们可以将`NOTES_MODEL`设置为`mongodb`来使用 MongoDB 数据库运行 Notes。

现在我们已经为 MongoDB 编写了所有的代码，我们可以继续测试`Notes`。

## 使用 MongoDB 运行 Notes 应用程序

我们准备使用 MongoDB 数据库测试`Notes`。到目前为止，你知道该怎么做；将以下内容添加到`package.json`的`scripts`部分：

```

The `MONGO_URL` environment variable is the URL to connect with your MongoDB database. This URL is the one that you need to use to run MongoDB on your laptop, as outlined at the top of this section. If you have a MongoDB server somewhere else, you'll be provided with the relevant URL to use.

You can start the `Notes` application as follows:

```

`MONGO_URL`环境变量应包含与您的 MongoDB 数据库连接的 URL。这里显示的 URL 对于在本地机器上启动 MongoDB 服务器是正确的，就像您在本节开始时在命令行上启动 MongoDB 一样。否则，如果您在其他地方提供了 MongoDB 服务器，您将被告知访问 URL 是什么，您的`MONGO_URL`变量应该有该 URL。

您可以启动两个`Notes`应用程序实例，并查看它们都共享相同的笔记集。

我们可以验证 MongoDB 数据库最终是否具有正确的值。首先，这样启动 MongoDB 客户端程序：

```

再次强调，这是基于迄今为止所呈现的 MongoDB 配置，如果您的配置不同，请在命令行上添加 URL。这将启动与 Notes 配置的数据库连接的交互式 MongoDB shell。要检查数据库的内容，只需输入命令：`db.notes.find()`。这将打印出每个数据库条目。

有了这一点，我们不仅完成了对`Notes`应用程序中 MongoDB 的支持，还支持了其他几种数据库，因此我们现在准备结束本章。

# 总结

在本章中，我们经历了不同的数据库技术的真正风暴。虽然我们一遍又一遍地看了同样的七个函数，但接触到各种数据存储模型和完成任务的方式是有用的。即便如此，在 Node.js 中访问数据库和数据存储引擎的选项只是触及了表面。

通过正确抽象模型实现，我们能够轻松地在不改变应用程序其余部分的情况下切换数据存储引擎。这种技术让我们探索了 JavaScript 中子类化的工作原理，以及创建相同 API 的不同实现的概念。此外，我们还对`import()`函数进行了实际介绍，并看到它可以用于动态选择要加载的模块。

在现实生活中的应用程序中，我们经常为类似的目的创建抽象。它们帮助我们隐藏细节或允许我们更改实现，同时使应用程序的其余部分与更改隔离。我们用于我们的应用程序的动态导入对于动态拼接应用程序非常有用；例如，加载给定目录中的每个模块。

我们避免了设置数据库服务器的复杂性。正如承诺的那样，当我们探索将 Node.js 应用程序部署到 Linux 服务器时，我们将在第十章中进行讨论，*将 Node.js 应用程序部署到 Linux 服务器*。

通过将我们的模型代码专注于存储数据，模型和应用程序应该更容易测试。我们将在第十三章中更深入地研究这一点，*单元测试和功能测试*。

在下一章中，我们将专注于支持多个用户，允许他们登录和退出，并使用 OAuth 2 对用户进行身份验证。
