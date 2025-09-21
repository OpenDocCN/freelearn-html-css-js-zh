# 7

# 事件驱动架构

事件是使用 Node.js 最强大的方式之一。Node.js 从头开始设计就是为了构建事件驱动的模块。许多核心库提供了易于使用和扩展的事件接口。此外，Node.js 还提供了一个强大的事件库，可以用来构建事件驱动的模块。

在本章中，我们将深入研究 Node.js 中的事件。我们将学习如何从核心库中使用事件，从事件监听器注册到事件发射，以及处理同一事件的多个监听器。

我们将使用事件构建我们的第一个 HTTP 服务器，并讨论事件监听器的组织和清理。

总结一下，以下是本章我们将探讨的主要主题：

+   介绍事件

+   监视文件更改

+   Node.js 事件发射器库

+   你的第一个 HTTP 服务器

+   为你的模块添加事件层

到本章结束时，你将知道如何使用事件，甚至如何在你的模块中包含事件接口。

# 技术要求

本章的代码文件可以在[`github.com/PacktPublishing/NodeJS-for-Beginners`](https://github.com/PacktPublishing/NodeJS-for-Beginners)找到

查看本章动作视频中的代码，视频链接为[`youtu.be/opZER2MY1Yc`](https://youtu.be/opZER2MY1Yc)

# 介绍事件

在现实世界中，事件是发生的事情。例如，当你点击一个按钮时，会触发一个点击事件。当你收到一条消息时，会触发一个消息接收事件。当你保存一个文件时，会触发一个文件保存事件。事件无处不在。在 Node.js 中，事件同样无处不在。

因此，当我们谈论 Node.js 中的事件时，我们谈论的是与现实世界相同的概念。事件是发生的事情，我们产生它们或消费它们。在某些情况下，一个实体产生一个事件，另一个实体消费它。在其他情况下，同一个实体既产生又消费事件。这可以非常灵活；甚至可能许多实体消费同一个事件，或者许多实体产生同一个事件。

如果你熟悉前端世界，你可能已经实现了当按钮被点击时的处理器，类似于这样：

```js
document.getElementById('my-button').addEventListener('click', () => {
    console.log('Button clicked');
});
```

在这种情况下，`addEventListener`方法接收两个参数，事件名称和回调函数。当事件被触发时，将调用回调函数。在这种情况下，事件名称是`click`，但你也可以订阅许多其他事件，例如`mouseover`、`mouseout`、`keydown`、`keyup`、`change`和`submit`。

如果你与其他编程语言有过合作，你可能听说过观察者、发布/订阅和中介模式。在本章中，我们将探讨如何使用 Node.js 事件库构建事件驱动的模块，并探讨核心库是如何使用这种架构的。

熟悉事件的最佳方式之一是通过使用 Node.js 核心 API 来处理文件。我们可以订阅事件，并在文件被修改时做出反应。因此，在下一节中，我们将详细探讨这个主题。

# 监视文件更改

由于我们已经熟悉 Node.js 文件系统库，让我们构建一个简单的脚本，用于监视文件更改。我们将使用`fs.watch`方法来监视文件更改。此方法接收两个参数，要监视的文件的路径和当文件更改时将被调用的回调函数。回调函数接收两个参数，事件类型和文件名。事件类型可以是`rename`或`change`。当文件被重命名或删除时，会触发`rename`事件。当文件被修改时，会触发`change`事件。

现在，我们将创建一个简单的程序来检测文件更改：

1.  让我们创建一个名为`watch.mjs`的文件，并添加以下代码：

    ```js
    import { watch } from 'node:fs';
    console.log('Watching for file changes...');
    watch('./watch.txt', (eventType, filename) => {
        console.log('-----------------------------');
        console.log(`Event type is: ${eventType}`);
        if (filename) {
            console.log(`Filename provided: ${filename}`);
        }
    });
    ```

1.  创建一个名为`watch.txt`的文件，并使用以下命令运行脚本：

    ```js
    node watch.js
    ```

1.  打开`watch.txt`文件，添加一些文本，并保存更改。您将看到脚本打印以下输出：

    ```js
    Watching for file changes...
    -----------------------------
    Event type is: change
    Filename provided: watch.txt
    ```

如您所见，`change`事件被触发，并提供了文件名。现在，重命名文件并保存更改。您将看到脚本打印以下输出：

```js
-----------------------------
Event type is: rename
Filename provided: watch2.txt
```

在下一节中，我们将学习如何在我们的应用程序中实现自定义事件，以及如何触发和消费它们。

# Node.js 事件发射器库

现在我们已经知道了如何监视文件更改，让我们探索 Node.js 事件库。这个库提供了一个`EventEmitter`类，可以用来构建简单的接口来注册和注销事件监听器以及触发事件。

让我们创建一个名为`event-emitter.mjs`的文件，并添加以下代码：

```js
import { EventEmitter } from 'node:events';
const emitter = new EventEmitter();
emitter.on('message', (message) => {
    console.log(`Message received: ${message}`);
});
emitter.emit('message', 'Hello world!');
```

在这个例子中，我们创建了一个`EventEmitter`类的实例，并为`message`事件注册了一个事件监听器。然后，我们使用消息`Hello world!`触发`message`事件。如果您运行脚本，您将看到消息在控制台中被打印出来。

您还可以为同一事件注册多个事件监听器和发射器；当您想要模块化代码以及/或者您希望从同一事件触发多个操作时，这是一种常见的做法。假设您收到一个传入的请求，并且您想存储该消息的副本并通知最终用户；通过使用事件，您可以并行处理这两个操作。让我们通过添加以下代码修改之前的例子：

```js
setInterval(() => {
    emitter.emit('message', `Interval (${Date.now()})`);
}, 1000);
emitter.on('message', (message) => {
    console.log(`Additional listener received: ${message}`);
});
emitter.once('message', (message) => {
    console.log(`Once listener received: ${message}`);
});
setTimeout(() => {
    emitter.emit('message', 'Timeout says hello!');
}, 2500);
```

让我们分析一下代码。首先，我们使用`setInterval`方法每秒触发一次`message`事件。然后，我们为`message`事件注册了一个额外的监听器。每当`message`事件被触发时，都会调用此监听器。然后，我们使用`once`方法注册了一个事件监听器。此监听器只会被调用一次，但如果您想监听多个消息，可以使用`on` – 例如，在 HTTP 服务器应用程序中监听传入请求时。最后，我们使用`setTimeout`方法在 2.5 秒后触发`message`事件。如果您运行脚本，您将看到以下输出：

```js
Message received: Hello world!
Message received: Interval (1691771547260)
Additional listener received: Interval (1691771547260)
Once listener received: Interval (1691771547260)
Message received: Interval (1691771548258)
Additional listener received: Interval (1691771548258)
Message received: Timeout says hello!
Additional listener received: Timeout says hello!
```

## 通过组织监听器来防止混乱

需要注意的一个重要事项是事件监听器是同步调用的。这意味着事件监听器是按照它们注册的顺序被调用的。此外，请记住，您可以使用更多通道在进程之间进行通信。在我们的示例中，我们使用了`message`，但您可以使用任何您想要的名称，或者有多个通道以更好地分段通信。

## 在不需要时移除监听器

`EventEmitter`类提供了`removeListener`和`off`方法，可以用来移除事件监听器，以及一个`removeAllListeners`方法，可以用来移除给定事件的全部事件监听器。您可以在官方文档中找到更多关于它的信息：[`nodejs.org/docs/latest-v20.x/api/events.html`](https://nodejs.org/docs/latest-v20.x/api/events.html)。

在下一节中，我们将使用 Node.js 创建我们的第一个 HTTP 服务器，这是在 Node.js 中进行 Web 应用程序开发时使用事件的最常见方式之一。

# 您的第一个 HTTP 服务器

现在我们已经知道了如何使用`EventEmitter`类，让我们构建一个简单的 HTTP 服务器。我们将使用`http`核心库创建服务器，并使用`EventEmitter`类来处理请求。在*第九章*中，我们将更详细地探讨如何构建 HTTP 服务器和客户端，但到目前为止，让我们专注于构建我们的第一个 HTTP 服务器。

让我们创建一个名为`server.mjs`的文件，并添加以下代码：

```js
import { createServer } from 'node:http';
const port = 3000;
const server = createServer();
server.on('request', (request, res) => {
  res.writeHead(200, { 'Content-Type': 'text/html' });
  res.end('<h1>This is my first HTTP server in Node.js. Yay</h1>!');
});
server.listen(port, () => {
  console.log(`Server running at http://localhost:${port}/`);
});
```

在这个例子中，我们创建了一个`http.Server`类的实例，并为`request`事件注册了一个事件监听器。每当收到请求时，都会调用此事件监听器。然后，我们使用`writeHead`方法设置响应的状态码和内容类型。最后，我们使用`end`方法发送响应。如果您运行脚本，您将看到以下输出：

```js
Server running at http://localhost:3000/
```

如果您在任何浏览器中打开此 URL，您将看到您的第一个 HTTP 服务器正在运行：

![图 7.1 – 应用程序运行的截图](img/B21678_07_1.jpg)

图 7.1 – 应用程序运行的截图

在下一节中，我们将学习如何在我们的模块中封装事件以及许多其他组件，以便轻松地发射和消费这些事件。这种技术相当流行，并且可以扩展到许多库。

# 为你的模块添加事件层

现在我们已经知道了如何使用`EventEmitter`类，让我们给我们的模块添加一个事件层。在这个例子中，我们将创建一个模块，它将被用来保存文件并在每次文件更改保存时触发一个事件。

让我们创建一个名为`utils.mjs`的文件，并添加以下代码：

```js
import { writeFile } from 'node:fs/promises';
import { EventEmitter } from 'node:events';
const emitter = new EventEmitter();
const on = emitter.on.bind(emitter);
const save = async (location, data) => {
  await writeFile(location, data);
  emitter.emit('file:saved', { location, data });
};
export { save, on };
```

在这个例子中，我们创建了一个`EventEmitter`类的实例，并导出了`save`函数。这个函数将被用来保存文件并触发`file:saved`事件。然后，我们导出了`EventEmitter`类的`on`方法。这个方法将被用来为`file:saved`事件注册事件监听器。

重要信息

在这个例子中，我们使用了`bind`来检查`this`值的正确性。你可以在官方文档中找到更多关于它的信息：[`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)。`bind`的使用相当高级，所以你现在可以跳过它。

现在，让我们创建一个名为`index.mjs`的文件，并添加以下代码：

```js
import { save, on } from './utils.mjs';
on('file:saved', ({ location, data }) => {
  console.log(`File saved at ${location}`);
});
console.log('Saving file...');
save('test.txt', 'Hello world!').catch('Error saving file');
console.log('The file is being saved but is not blocking the execution...');
```

如果你运行脚本，你将看到以下输出：

```js
Saving file...
The file is being saved but is not blocking the execution...
File saved at test.txt
```

正如你所见，`file:saved`事件在`save`函数完成后被触发。这意味着`save`函数不会阻塞脚本的执行。在本书的前几个例子中，我们使用了`then`来处理 promise 的结果；在这种情况下，我们提供了一个替代方案，使用事件可以更容易地解耦应用程序的逻辑。

# 摘要

在本章中，我们学习了如何在 Node.js 中使用事件。我们学习了`EventEmitter`类以及如何使用它来发射和监听事件。我们还学习了如何使用事件来解耦我们应用程序的逻辑。

此外，我们还构建了一个脚本来监视我们系统中文件的更改，并且我们还构建了我们的第一个 HTTP 服务器，并学习了如何使用事件来处理请求。

最后，我们构建了一个简单的库，它导出一个事件层来解耦我们应用程序的逻辑。这将允许我们在未来的章节中构建更健壮的应用程序。

在下一章中，我们将学习如何将测试添加到我们的应用程序中。这将帮助我们构建更健壮的应用程序，避免错误，并且总体上，在学习 Node.js 的同时巩固我们对 Node.js 的知识。

# 进一步阅读

+   *重构大师 – 中介* *模式*：[`refactoring.guru/design-patterns/mediator`](https://refactoring.guru/design-patterns/mediator)

+   *重构大师 – 观察者* *模式*：[`refactoring.guru/design-patterns/observer`](https://refactoring.guru/design-patterns/observer)

+   *NodeConf Remote 2020 - 安娜·亨宁森 - Node.js 以及作为* *EventTarget* *的挑战*: [`www.youtube.com/watch?v=SOPC3aLoD4U`](https://www.youtube.com/watch?v=SOPC3aLoD4U)

+   *Node.js 事件发射器*: [`nodejs.org/en/learn/asynchronous-work/the-nodejs-event-emitter`](https://nodejs.org/en/learn/asynchronous-work/the-nodejs-event-emitter)
