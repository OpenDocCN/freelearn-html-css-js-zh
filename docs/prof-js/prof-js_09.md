# 第十章：事件驱动编程和内置模块

## 学习目标

在本章结束时，您将能够：

+   在 Node.js 中使用事件模块

+   创建事件发射器以增强现有代码的功能

+   构建自定义事件发射器

+   使用内置模块和实用工具

+   实现一个计时器模块，以获得调度计时器函数的 API

在本章中，我们将使用事件发射器和内置模块，以避免创建具有紧密耦合依赖关系的项目。

## 介绍

在上一章中，我们讨论了 Node.js 中如何使用事件驱动编程，以及如何修改正常的基于回调的异步操作以使用 async-await 和 promises。我们知道 Node.js 核心 API 是建立在异步驱动架构上的。Node.js 有一个事件循环，用于处理大多数异步和基于事件的操作。

在 JavaScript 中，事件循环不断地运行并从回调队列中消化消息，以确保执行正确的函数。没有事件，我们可以看到代码非常紧密耦合。对于一个简单的聊天室应用程序，我们需要编写类似这样的东西：

```js
class Room {
    constructor() {
        this.users = [];
    }
    addUser(user) {
        this.users.push(user);
    }
    sendMessage(message) {
        this.users.forEach(user => user.sendMessage(message));
    }
}
```

正如您所看到的，因为我们没有使用事件，我们需要保留房间中所有用户的列表。当我们将用户添加到房间时，我们还需要将用户添加到我们创建的列表中。在发送消息时，我们还需要遍历我们列表中的所有用户并调用`sendMessage`方法。我们的用户类将被定义如下：

```js
class User {
    constructor() {
        this.rooms = {}
    }
    joinRoom(roomName, room) {
        this.rooms[roomName] = room;
        room.addUser(this);
    }
    sendMessage(roomName, message) {
        this.rooms[roomName].sendMessage(message);
    }
}
```

您可以看到这变得过于复杂；为了加入聊天室，我们需要同时将房间和当前用户添加到房间中。当我们的应用程序最终变得非常复杂时，我们会发现这会引发传统方法的问题。如果此应用程序需要网络请求（异步操作），它将变得非常复杂，因为我们需要用异步操作包装我们希望执行的所有代码。我们可能能够将该逻辑提取出来，但是当我们处理由未知数量的随机事件驱动的应用程序时，使用事件驱动编程的好处在于使我们的代码更易于维护。

## 传统方法与事件驱动编程

正如我们在介绍中提到的，在传统的编程模式中，当我们希望它们进行通信时，我们喜欢在组件之间建立直接的联系。这在下图中有所体现：

![图 9.1：传统编程方法](img/C14587_09_01.jpg)

###### 图 9.1：传统编程方法

对于一个简单的应用程序，允许用户更新其个人资料并接收消息，我们可以看到我们有四个组件：

+   代理

+   个人资料

+   投票

+   消息

这些组件之间的交互方式是通过调用希望通信的组件中的适当方法来实现的。通过这样做，使得代码非常易于理解，但我们可能需要传递组件引用。以我们的`Agent`类为例：

```js
class Agent {
    constructor(id, agentInfo, voteObj, messageObj) {
        this.voteObj = voteObj;
        this.messageObj = messageObj;
    }
    checkMessage() {
        if (this.messageObj.hasMessage()) {
            const message = this.messageObj.nextMessate();
            return message;
        }
        return undefined;
    }
    checkVote() {
        if (this.voteObj.hasNewVote()) {
            return true;
        }
        return false;
    }
}
```

`Agent`类必须在未来存储与其希望通信的组件的引用。如果没有，我们的组件就无法与其他组件通信。在前面的示例中，我们创建的`Agent`对象与其他所有内容都非常紧密耦合。它在创建时需要所有这些对象的引用，这使得我们的代码在未来要更改某些内容时非常难以解耦。考虑前面的`Agent`代码。如果我们要为其添加更多功能，我们希望代理类与新功能进行通信，例如社交页面、直播页面等。只要我们在我们的`constructor`中添加对这些对象的引用，这在技术上是可行的。通过这样做，我们将冒着我们的代码在未来看起来像这样的风险：

```js
class Agent {
    constructor(id, agentInfo, voteObj, messageObj, socialPage, gamePage, liveStreamPage, managerPage, paymentPage...) {
        this.voteObj = voteObj;
        this.messageObj = messageObj;
        this.socialPage = socialPage;
        this.gamePage = gamePage;
        this.liveStreamPage = liveStreamPage;
        this.managerPage = managerPage;
        this.paymentPage = paymentPage;
        ...
    }
    ...
}
```

当我们的应用程序变得越来越复杂时，我们的`Agent`类也会变得越来越复杂。因为它在`constructor`中有所有的引用，所以我们容易因为错误地传递参数类型而引起问题。当我们试图一次性在多个组件之间进行通信时，这是一个常见的问题。

## 事件

我们之前的方法——即处理组件通信的方法——是直接的，而且非常静态。我们需要存储我们想要进行通信的组件引用，并且在想要向其发送消息时编写非常特定于组件的代码。在 JavaScript 中，有一种新的通信方式，它被称为**事件**。

让我们考虑这个例子；你朋友传递给你的光是你从朋友那里接收事件的一种方式。在 JavaScript 中，我们可以拥有能够发出事件的对象。通过发出事件，我们可以创建对象之间的新通信方式。这也被称为观察者模式。以下图表描述了观察者模式：

![图 9.2：观察者模式](img/C14587_09_02.jpg)

###### 图 9.2：观察者模式

在这种模式中，我们不是在组件中调用特定的方法，而是希望发起通信的组件只是发出一个事件。我们可以有多个观察者观察来自组件的事件。这样，我们把消费消息的责任完全放在了消费者身上。当观察者决定观察事件时，它将在组件发出事件时每次接收到事件。如果使用事件来实现前面复杂的例子，它会是这样的：

![图 9.3：使用事件的观察者模式](img/C14587_09_03.jpg)

###### 图 9.3：使用事件的观察者模式

在这里，我们可以看到每个组件都遵循我们的观察者模式，当我们将其转换为代码时，它会看起来像这样：

```js
class Agent {
    constructor(id, agentInfo, emitter) {
        this.messages = [];
        this.vote = 0;
        emitter.on('message', (message) => {
            this.messages.push(message);
        });
        emitter.on('vote', () => {
            this.vote += 1;
        })
    }
}
```

现在，我们不再需要获取所有我们想要进行通信的组件的引用，而是只传递一个事件发射器，它处理所有的消息。这使得我们的代码与其他组件的耦合度大大降低。这基本上就是我们在代码中实现事件观察者模式的方式。在现实生活中，这可能会变得更加复杂。在下一个练习中，我们将介绍一个简单的例子，演示如何使用 Node.js 中内置的事件系统来发出事件。

### 练习 67：一个简单的事件发射器

在介绍中，我们谈到了如何使用事件观察者模式来消除我们代码中想要进行通信的所有组件的引用。在这个练习中，我们将介绍 Node.js 中内置的事件模块，我们如何创建`EventEmitter`以及如何使用它。

执行以下步骤完成这个练习：

1.  导入`events`模块：

```js
const EventEmitter = require('events');
```

我们将导入 Node.js 中内置的`events`模块。它提供了一个构造函数，我们可以用它来创建自定义的事件发射器或创建一个继承自它的类。因为这是一个内置模块，所以不需要安装它。

1.  创建一个新的`EventEmitter`：

```js
const emitter = new EventEmitter();
```

1.  尝试发出一个事件：

```js
emitter.emit('my-event', { value: 'event value' });
```

1.  附加一个事件监听器：

```js
emitter.on('my-event', (value) => {
    console.log(value);
});
```

要向我们的发射器添加事件监听器，我们需要在发射器上调用`on`方法，传入事件名称和在发出事件时要调用的函数。当我们在发出事件后添加事件监听器时，我们会发现事件监听器没有被调用。原因是在我们之前发出事件时，并没有为该事件附加事件监听器，因此它没有被调用。

1.  再发出一个事件：

```js
emitter.emit('my-event', { value: 'another value' });
```

当我们这次发出事件时，我们会看到我们的事件监听器被正确调用，并且我们的事件值被正确打印出来，就像这样：

![图 9.4：使用正确的事件值发出的事件](img/C14587_09_04.jpg)

###### 图 9.4：使用正确的事件值发出的事件

1.  为`my-event`附加另一个事件监听器：

```js
emitter.on('my-event', (value) => {
    console.log('i am handling it again');
});
```

我们不仅限于每个事件只有一个监听器 - 我们可以附加尽可能多的事件监听器。当发射事件时，它将调用所有监听器。

1.  发射另一个事件：

```js
emitter.emit('my-event', { value: 'new value' });
```

以下是上述代码的输出：

![图 9.5：多次发射事件后的输出](img/C14587_09_05.jpg)

###### 图 9.5：多次发射事件后的输出

当我们再次发射事件时，我们将看到我们发射的第一个事件。我们还将看到它成功地打印出我们的消息。请注意，它保持了与我们附加监听器时相同的顺序。当我们发射错误时，发射器会遍历数组并依次调用每个监听器。

1.  创建`handleEvent`函数：

```js
function handleEvent(event) {
    console.log('i am handling event type: ', event.type);
}
```

当我们设置我们的事件监听器时，我们使用了匿名函数。虽然这很容易和简单，但它并没有为我们提供`EventEmitters`提供的所有功能：

1.  将新的`handleEvent`附加到新类型的事件上：

```js
emitter.on('event-with-type', handleEvent);
```

1.  发射新的事件类型：

```js
emitter.emit('event-with-type', { type: 'sync' });
```

以下是上述代码的输出：

![图 9.6：发射新的事件类型](img/C14587_09_06.jpg)

###### 图 9.6：发射新的事件类型

1.  移除事件监听器：

```js
emitter.removeListener('event-with-type', handleEvent);
```

因为我们使用了命名函数，所以我们可以使用这个函数引用来移除监听器，一旦我们不再需要将事件传递给该监听器。

1.  在移除监听器后发射事件：

```js
emitter.emit('event-with-type', { type: 'sync2' });
```

以下是上述代码的输出：

![图 9.7：移除监听器后发射事件的输出](img/C14587_09_07.jpg)

###### 图 9.7：移除监听器后发射事件的输出

因为我们刚刚移除了对`event-with-type`的监听器，当我们再次发射事件时，它将不会被调用。

在这个练习中，我们构建了一个非常简单的事件发射器，并测试了添加和移除监听器。现在，我们知道如何使用事件将消息从一个组件传递到另一个组件。接下来，我们将深入研究事件监听器方法，并看看通过调用它们我们可以实现什么。

### 事件发射器方法

在上一个练习中，我们讨论了一些可以调用的方法来发射事件和附加监听器。我们还使用了`removeListener`来移除我们附加的监听器。现在，我们将讨论我们可以在事件监听器上调用的各种方法。这将帮助我们更轻松地管理事件发射器。

### 移除监听器

有些情况下，我们希望从我们的发射器中移除监听器。就像我们在上一个练习中所做的那样，我们可以通过调用`removeListener`来简单地移除一个监听器：

```js
emitter.removeListener('event-with-type', handleEvent);
```

当我们调用`removeListener`方法时，我们必须为其提供事件名称和函数引用。当我们调用该方法时，无论事件监听器是否已设置都无关紧要；如果监听器一开始就没有设置，什么也不会发生。如果设置了，它将遍历我们的事件发射器中监听器的数组，并移除该监听器的第一次出现，就像这样：

```js
const emitter = new EventEmitter();
function handleEvent(event) {
    console.log('i am handling event type: ', event.type);
}
emitter.on('event-with-type', handleEvent);
emitter.on('event-with-type', handleEvent);
emitter.on('event-with-type', handleEvent);
emitter.emit('event-with-type', { type: 'sync' });
emitter.removeListener('event-with-type', handleEvent);
```

在这段代码中，我们三次附加了相同的监听器。在事件发射器中，当我们附加事件监听器时，允许这样做，它只是简单地追加到该事件的事件监听器数组中。当我们在`removeListener`之前发射我们的事件时，我们将看到我们的监听器被调用三次：

![图 9.8：在移除监听器之前使用 emit 事件调用三次监听器](img/C14587_09_08.jpg)

###### 图 9.8：在移除监听器之前使用 emit 事件调用三次监听器

在这种情况下，因为我们有三个相同的监听器附加到我们的事件上，当我们调用`removeListener`时，它只会移除我们的`listener`数组中的第一个监听器。当我们再次发射相同的事件时，我们会看到它只运行两次：

![图 9.9：使用 removeListener 后，第一个监听器被移除](img/C14587_09_09.jpg)

###### 图 9.9：使用 removeListener 后，第一个监听器被移除

### 移除所有监听器

我们可以从我们的事件发射器中删除特定的侦听器。但通常，当我们在发射器上处理多个侦听器时，有时我们希望删除所有侦听器。`EventEmitter`类为我们提供了一个方法，我们可以使用它来删除特定事件的所有侦听器。考虑我们之前使用的相同示例：

```js
const emitter = new EventEmitter();
function handleEvent(event) {
    console.log('i am handling event type: ', event.type);
}
emitter.on('event-with-type', handleEvent);
emitter.on('event-with-type', handleEvent);
emitter.on('event-with-type', handleEvent);
```

如果我们想要删除`event-with-type`事件的所有侦听器，我们将不得不多次调用`removeListener`。有时，当我们确定所有事件侦听器都是由我们添加的，没有其他组件或模块时，我们可以使用单个方法调用来删除该事件的所有侦听器：

```js
emitter.removeAllListeners('event-with-type');
```

当我们调用`removeAllListeners`时，我们只需要提供事件名称。这将删除附加到事件的所有侦听器。调用后，事件将没有处理程序。确保您不要删除由另一个组件附加的侦听器，如果您使用此功能：

```js
emitter.emit('event-with-type', { type: 'sync' });
```

当我们在调用`removeAllListeners`后再次发出相同的事件时，我们将看到我们的程序不会输出任何内容：

![图 9.10：使用`removeAllListeners`将不会输出任何内容](img/C14587_09_10.jpg)

###### 图 9.10：使用`removeAllListeners`将不会输出任何内容

### 附加一次性侦听器

有时，我们希望我们的组件只接收特定事件一次。我们可以通过使用`removeListener`来确保在调用后删除侦听器：

```js
const EventEmitter = require('events');
const emitter = new EventEmitter();
function handleEvent(event) {
    console.log('i am handling event type once : ', event.type);
    emitter.removeListener('event-with-type', handleEvent);
}
emitter.on('event-with-type', handleEvent);
emitter.emit('event-with-type', { type: 'sync' });
emitter.emit('event-with-type', { type: 'sync' });
emitter.emit('event-with-type', { type: 'sync' });
```

在这里，我们可以看到，在我们的`handleEvent`侦听器中，执行后我们还删除了侦听器。这样，我们可以确保我们的事件侦听器只会被调用一次。当我们运行上述代码时，我们将看到以下输出：

![图 9.11：使用`handleEvent`侦听器后的输出](img/C14587_09_11.jpg)

###### 图 9.11：使用`handleEvent`侦听器后的输出

这做到了我们想要的，但还不够好。它要求我们在事件侦听器中保留发射器的引用。此外，它还不够健壮，因为我们无法将侦听器逻辑分离到不同的文件中。`EventEmitter`类为我们提供了一个非常简单的方法，可以用来附加一次性侦听器：

```js
...
emitter.once('event-with-type', handleEvent);
emitter.emit('event-with-type', { type: 'sync' });
emitter.emit('event-with-type', { type: 'sync' });
emitter.emit('event-with-type', { type: 'sync' });
```

在这里，当我们附加事件侦听器时，我们使用了`.once`方法。这告诉我们的发射器，我们传递的函数应该只被调用一次，并且在被调用后将从事件侦听器列表中删除。当我们运行它时，它将为我们提供与以前相同的输出：

图 9.12：使用`.once`方法获取一次性侦听器

](Images/C14587_09_12.jpg)

###### 图 9.12：使用`.once`方法获取一次性侦听器

这样，我们就不需要在侦听器中保留对事件发射器的引用。这使我们的代码更灵活，更容易模块化。

### 从事件发射器中读取

到目前为止，我们一直在设置和删除事件发射器的侦听器。`EventEmitter`类还为我们提供了几种读取方法，我们可以从中获取有关事件发射器的更多信息。考虑以下示例：

```js
const EventEmitter = require('events');
const emitter = new EventEmitter();
emitter.on('event 1', () => {});
emitter.on('event 2', () => {});
emitter.on('event 2', () => {});
emitter.on('event 3', () => {});
```

在这里，我们向我们的发射器添加了三种类型的事件侦听器。对于`event 2`，我们为其设置了两个侦听器。要获取我们的发射器中特定事件的事件侦听器数量，我们可以调用`listenerCount`。对于上面的示例，如果我们想要知道附加到`event 1`的事件侦听器的数量，我们可以执行以下命令：

```js
emitter.listenerCount('event 1');
```

以下是上述代码的输出：

图 9.13：输出显示附加到事件 1 的事件数量

](Images/C14587_09_13.jpg)

###### 图 9.13：输出显示附加到事件 1 的事件数量

同样，我们可以通过执行以下命令来检查附加到`event 2`的事件侦听器的数量：

```js
emitter.listenerCount('event 2');
```

以下是上述代码的输出：

![图 9.14：输出显示附加到事件 2 的事件数量](img/C14587_09_14.jpg)

###### 图 9.14：输出显示附加到事件 2 的事件数量

有时我们想要知道已经附加到事件的事件监听器列表，以便我们可以确定某个处理程序是否已经附加，就像这样：

```js
function anotherHandler() {}
emitter.on('event 4', () => {});
emitter.on('event 4', anotherHandler);
```

在这里，我们附加了一个匿名函数到`event 4`，并使用一个命名函数附加了另一个监听器。如果我们想知道`anotherHandler`是否已经附加到`event 4`，我们可以附加一个监听器列表到该事件。`EventEmitter`类为我们提供了一个非常简单的方法来调用这个：

```js
const event4Listeners = emitter.listeners('event 4');
```

以下是前面代码的输出：

![图 9.15：使用 EventEmitter 类获取附加到事件的监听器列表](img/C14587_09_15.jpg)

###### 图 9.15：使用 EventEmitter 类获取附加到事件的监听器列表

在这里，我们可以看到我们已经附加到我们的发射器的两个监听器：一个是我们的匿名函数，另一个是我们的命名函数`anotherHandler`。要检查我们的处理程序是否已经附加到发射器，我们可以检查`event4Listeners`数组中是否有`anotherHandler`：

```js
event4Listeners.includes(anotherHandler);
```

以下是前面代码的输出：

![图 9.16：检查处理程序是否附加到发射器](img/C14587_09_16.jpg)

###### 图 9.16：检查处理程序是否附加到发射器

通过使用这个方法和数组包含一个方法，我们可以确定一个函数是否已经附加到我们的事件。

### 获取已注册监听器的事件列表

有时我们需要获取已注册监听器的事件列表。这可以用于确定我们是否已经为事件附加了监听器，或者查看事件名称是否已经被使用。继续前面的例子，我们可以通过调用`EventEmitter`类中的另一个内部方法来获取这些信息：

```js
emitter.eventNames();
```

以下是前面代码的输出：

![图 9.17：使用 EventEmitter 类获取事件名称的信息](img/C14587_09_17.jpg)

###### 图 9.17：使用 EventEmitter 类获取事件名称的信息

在这里，我们可以看到我们的事件发射器已经附加到四种不同的事件类型的监听器；即事件 1-4。

### 最大监听器

默认情况下，每个事件发射器只能为任何单个事件注册最多 10 个监听器。当我们附加超过最大数量时，我们将收到类似这样的警告：

![图 9.18：为单个事件附加超过 10 个监听器时的警告](img/C14587_09_18.jpg)

###### 图 9.18：为单个事件附加超过 10 个监听器时的警告

这是为了确保我们不会泄漏内存而设置的预防措施，但也有时我们需要为一个事件设置超过 10 个监听器。如果我们确定了，我们可以通过调用`setMaxListeners`来更新默认的最大值：

```js
emitter.setMaxListeners(20)
```

在这里，我们将最大监听器默认设置为`20`。我们也可以将其设置为`0`或无穷大，以允许无限数量的监听器。

### 在事件之前添加监听器

当我们添加监听器时，它们被附加到监听器数组的末尾。当事件被发出时，发射器将按照它们被分配的顺序调用每个分配的监听器。在某些情况下，我们需要我们的监听器首先被调用，我们可以使用事件发射器提供的内置方法来实现这一点：

```js
const EventEmitter = require('events');
const emitter = new EventEmitter();
function handleEventSecond() {
    console.log('I should be called second');
}
function handleEventFirst() {
    console.log('I should be called first');
}
emitter.on('event', handleEventSecond);
emitter.on('event', handleEventFirst);
emitter.emit('event');
```

在这里，我们在`handleEventFirst`之前附加了`handleEventSecond`。当我们发出事件时，我们将看到以下输出：

![图 9.19：在第一个事件之前附加第二个事件后发出事件](img/C14587_09_19.jpg)

###### 图 9.19：在第一个事件之前附加第二个事件后发出事件

因为事件监听器是按照它们附加的顺序调用的，我们可以看到当我们发出事件时，`handleEventSecond`首先被调用，然后是`handleEventFirst`。如果我们希望`handleEventFirst`在使用`emitter.on()`附加它们的顺序不变的情况下首先被调用，我们可以调用`prependListener`：

```js
...
emitter.on('event', handleEventSecond);
emitter.prependListener('event', handleEventFirst);
emitter.emit('event');
```

前面的代码将产生以下输出：

![图 9.20：使用 prependListener 对事件进行排序](img/C14587_09_20.jpg)

###### 图 9.20：使用 prependListener 对事件进行排序

这可以帮助我们保持监听器的顺序，并确保优先级较高的监听器始终首先被调用。接下来我们将讨论监听器中的并发性。

### 监听器中的并发性

在之前的章节中，我们提到了如何将多个监听器附加到我们的发射器上，以及在事件被触发时这些监听器是如何工作的。之后，我们还谈到了如何在事件被触发时添加监听器，使得它们首先被调用。我们可能想要添加监听器的原因是，当监听器被调用时，它们是同步一个接一个被调用的。考虑以下例子：

```js
const EventEmitter = require('events');
const emitter = new EventEmitter();
function slowHandle() {
    console.log('doing calculation');
    for(let i = 0; i < 10000000; i++) {
        Math.random();
    }
}
function quickHandle() {
    console.log('i am called finally.');
}
emitter.on('event', slowHandle);
emitter.on('event', quickHandle);
emitter.emit('event');
```

在这里，我们有两个附加到`event`类型的监听器。当事件被触发时，它将首先调用`slowHandle`，然后调用`quickHandle`。在`slowHandle`中，我们有一个非常大的循环，模拟一个在事件监听器中可以执行的非常耗时的操作。当我们运行前面的代码时，我们首先会看到`doing calculation`被打印出来，然后会有一个很长的等待，直到`I am called finally`被调用。我们可以看到，当发射器调用事件监听器时，它是同步进行的。这可能会给我们带来问题，因为在大多数情况下，我们不希望等待一个监听器完成后再触发另一个监听器。不过，有一种简单的解决方法：我们可以用`setImmediate`函数包装我们的耗时逻辑。`setImmediate`函数将我们的逻辑包装成一个立即执行的异步块，这意味着耗时的循环是非阻塞的。我们将在本书的后面介绍`setImmediate`函数：

```js
...
function slowHandle() {
    console.log('doing calculation');
    setImmediate(() => {
        for(let i = 0; i < 10000000; i++) {
            Math.random();
        }
    });
}
```

当我们用`setImmediate()`包装我们的耗时逻辑时，代码几乎同时输出**doing calculation**和**I am called finally**。通过用`setImmediate`包装所有逻辑，我们可以确保它是异步调用的。

### 构建自定义事件发射器

有些情况下，我们希望将事件发射功能构建到我们自己的自定义类中。我们可以通过使用**JavaScript ES6**继承来实现这一点。这允许我们创建一个自定义类，同时扩展事件发射器的所有功能。例如，假设我们正在构建一个火警类：

```js
class FireAlarm {
    constructor(modelNumber, type, cost) {
        this.modelNumber = modelNumber;
        this.type = type;
        this.cost = cost;
        this.batteryLevel = 10;
    }
    getDetail() {
        return '${this.modelNumber}:[${this.type}] - $${this.cost}';
    }
    test() {
        if (this.batteryLevel > 0) {
            this.batteryLevel -= 0.1;
            return true;
        }
        return false;
    }
}
```

在这里，我们有一个`FireAlarm`类，它有一个存储有关这个火警的信息的构造函数。它还有一些自定义方法来测试警报，比如检查电池电量，以及一个`getDetail`方法来返回表示警报信息的字符串。在定义了这个类之后，我们可以像这样使用`FireAlarm`类：

```js
const livingRoomAlarm = new FireAlarm('AX-00101', 'BATT', '20');
console.log(livingRoomAlarm.getDetail());
```

以下是前面代码的输出：

![图 9.21：定义火警类](img/C14587_09_21.jpg)

###### 图 9.21：定义火警类

现在，我们想在刚刚创建的火警上设置事件。我们可以通过创建一个通用事件发射器并将其存储在我们的`FireAlarm`对象中来实现这一点：

```js
class FireAlarm {
    constructor(modelNumber, type, cost) {
        this.modelNumber = modelNumber;
        this.type = type;
        this.cost = cost;
        this.batteryLevel = 10;
        this.emitter = new EventEmitter();
    }
    ...
}
```

当我们想要监视警报上的事件时，我们必须这样做：

```js
livingRoomAlarm.emitter.on('low-battery', () => {
    console.log('battery low');
});
```

虽然这是完全可以的，并且对我们的用例有效，但这显然不是最健壮的解决方案。因为我们的火警是发出事件的一方，我们希望像这样：

```js
livingRoomAlarm.on('low-battery', () => {
    console.log('battery low');
});
```

通过直接在火警上使用`.on`，我们告诉未来的开发人员，将要在这上面工作，我们的火警也是一个事件发射器。但是现在，我们的类定义不允许使用。我们可以通过使用类继承来解决这个问题，在那里我们可以使我们的`FireAlarm`类扩展`EventEmitter`类。通过这样做，它将拥有`EventEmitter`的所有功能。我们可以修改我们的类如下：

```js
class FireAlarm extends EventEmitter {
    constructor(modelNumber, type, cost) {
        this.modelNumber = modelNumber;
        this.type = type;
        this.cost = cost;
        this.batteryLevel = 10;
    }
    ...
}
```

使用`extends`关键字后跟`EventEmitter`，我们告诉 JavaScript`FireAlarm`类是`EventEmitter`的子类。因此，它将继承父类的所有属性和方法。但这本身并不能解决所有问题。当我们运行更新后的`FireAlarm`代码时，我们会看到抛出一个错误：

![图 9.22：当我们运行更新后的 FireAlarm 代码时会抛出错误](img/C14587_09_22.jpg)

###### 图 9.22：当我们运行更新后的 FireAlarm 代码时会抛出错误

这是因为我们使用了一个非常定制的类，具有自定义的构造函数，并访问`this`（这用作对当前对象的引用）。在此之前，我们需要确保在此之前调用父构造函数。为了使此错误消失，我们只需在自己的构造函数中添加对父构造函数的调用：

```js
class FireAlarm extends EventEmitter {
    constructor(modelNumber, type, cost) {
        super();
        this.modelNumber = modelNumber;
        this.type = type;
        this.cost = cost;
        this.batteryLevel = 10;
    }
    ...
}
```

现在，让我们测试我们自己的自定义`EventEmitter`：

```js
livingRoomAlarm.on('low-battery', () => {
    console.log('battery low');
});
livingRoomAlarm.emit('low-battery');
```

以下是上述代码的输出：

![图 9.23：'low-battery'事件的事件监听器被正确触发](img/C14587_09_23.jpg)

###### 图 9.23：'low-battery'事件的事件监听器被正确触发

在这里，我们可以看到我们将`livingRoomAlarm`视为常规的`EventEmitter`，当我们发出*low-battery*事件时，我们看到该事件的事件监听器被正确触发。在下一个练习中，我们将使用我们对`EventEmitters`的所有了解制作一个非常简单的聊天室应用程序。

### 练习 68：构建聊天室应用程序

之前，我们讨论了如何在我们的事件发射器上附加事件监听器并发出事件。在这个练习中，我们将构建一个简单的聊天室管理软件，该软件使用事件进行通信。我们将创建多个组件，并查看如何使它们相互通信。

#### 注意：

此练习的代码文件可以在[`github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson09/Exercise68`](https://github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson09/Exercise68)找到。

执行以下步骤完成此练习：

1.  创建一个`User`类：

```js
class User {
    constructor(name) {
        this.name = name;
        this.messages = [];
        this.rooms = {};
    }
    joinRoom(room) {
        room.on('newMessage', (message) => {
            this.messages.push(message);
        });
        this.rooms[room.name] = room;
    }
    getMesssages(roomName) {
        return this.messages.filter((message) => {
            return message.roomName === roomName;
        })
    }
    printMessages(roomName) {
        this.getMesssages(roomName).forEach((message) => {
            console.log(`>> [${message.roomName}]:(${message.from}): ${message.message}`);
        });
    }
    sendMessage(roomName, message) {
        this.rooms[roomName].emit('newMessage', {
            message,
            roomName,
            from: this.name
        });
    }
}
```

在这里，我们为用户创建了一个`User`类。它有一个`joinRoom`方法，我们可以调用该方法将用户加入房间。它还有一个`sendMessage`方法，该方法将消息发送给房间中的所有人。当我们加入一个房间时，我们还会监听来自该房间的所有新消息事件，并在接收到消息时追加消息。

1.  创建一个扩展`EventEmitter`类的`Room`类：

```js
class Room extends EventEmitter {
    constructor(name) {
        super();
        this.name = name;
    }
}
```

在这里，我们通过扩展现有的`EventEmitter`类创建了一个新的`Room`类。我们这样做的原因是我们希望在我们的`room`对象上拥有自定义属性，并且这样可以增加代码的灵活性。

1.  创建两个用户，`bob`和`kevin`：

```js
const bob = new User('Bob');
const kevin = new User('Kevin');
```

1.  使用我们的`Room`类创建一个房间：

```js
const lobby = new Room('Lobby');
```

1.  将`bob`和`kevin`加入`lobby`：

```js
bob.joinRoom(lobby);
kevin.joinRoom(lobby);
```

1.  从`bob`发送几条消息：

```js
bob.sendMessage('Lobby', 'Hi all');
bob.sendMessage('Lobby', 'I am new to this room.');
```

1.  打印`bob`的消息日志：

```js
bob.printMessages('Lobby');
```

以下是上述代码的输出：

![图 9.24：打印 bob 的消息日志](img/C14587_09_24.jpg)

###### 图 9.24：打印 bob 的消息日志

在这里，您可以看到我们所有的消息都正确添加到了`bob`的日志中。接下来，我们将检查`kevin`的日志。

1.  打印`kevin`的消息日志：

```js
kevin.printMessage('Lobby');
```

以下是上述代码的输出：

![图 9.25：打印 kevin 的消息日志](img/C14587_09_25.jpg)

###### 图 9.25：打印 kevin 的消息日志

即使我们从未明确对`kevin`做过任何事情，他也会收到所有消息，因为他正在监听房间中的新消息事件。

1.  从`kevin`和`bob`发送消息：

```js
kevin.sendMessage('Lobby', 'Hi bob');
bob.sendMessage('Lobby', 'Hey kevin');
kevin.sendMessage('Lobby', 'Welcome!');
```

1.  检查`kevin`的消息日志：

```js
kevin.printMessages('Lobby');
```

以下是上述代码的输出：

![图 9.26：检查 kevin 的消息日志](img/C14587_09_26.jpg)

###### 图 9.26：检查 kevin 的消息日志

在这里，我们可以看到所有我们的消息都正确地添加到我们的`user`对象中。因为我们使用事件发射器，所以我们避免了在我们的接收者周围传递引用。此外，因为我们在我们的房间上发出了消息事件，而我们的用户只是监听该事件，所以我们不需要手动遍历房间中的所有用户并传递消息。

1.  让我们修改`joinRoom`和`constructor`，以便稍后可以移除监听器：

```js
class User {
    constructor(name) {
        this.name = name;
        this.messages = [];
        this.rooms = {};
        this.messageListener = (message) => {
            this.messages.push(message);
        }
    }
    joinRoom(room) {
        this.messageListener = (message) => {
            this.messages.push(message);
        }
        room.on('newMessage', this.messageListener);
        this.rooms[room.name] = room;
    }
    ...
}
```

当我们移除监听器时，我们需要传递该监听器函数的引用，因此，我们需要将该引用存储在对象中，以便稍后可以使用它来移除我们的监听器。

1.  添加`leaveRoom`：

```js
class User {
    ...
    leaveRoom(roomName) {
        this.rooms[roomName].removeListener('newMessage', this.messageListener);
delete this.rooms[roomName];
    }
}
```

在这里，我们正在使用我们在构造函数中设置的函数引用，并将其传递给我们房间的`removeListener`。我们还从对象中移除了引用，以便稍后可以释放内存。

1.  从`room`中移除`bob`：

```js
bob.leaveRoom('Lobby');
```

1.  从`kevin`发送一条消息：

```js
kevin.sendMessage('Lobby', 'I got a good news for you guys');
```

1.  检查`bob`的消息列表：

```js
bob.printMessages('Lobby');
```

以下是上述代码的输出：

![图 9.27：再次检查鲍勃的消息列表](img/C14587_09_27.jpg)

###### 图 9.27：检查鲍勃的消息列表

因为`bob`离开了房间，并且我们移除了消息监听器，所以当发出新消息事件时，`newMessage`事件处理程序不会再被调用。

1.  检查`kevin`的消息列表：

```js
kevin.printMessages('Lobby');
```

以下是上述代码的输出：

![图 9.28：再次检查 kevin 的消息列表](img/C14587_09_28.jpg)

###### 图 9.28：再次检查 kevin 的消息列表

当我们检查`kevin`的消息列表时，我们应该仍然能够看到他仍然从房间中收到新消息。如果使用传统方法来完成这项工作，我们将需要编写更多的代码来完成相同的事情，这将非常容易出错。

在这个练习中，我们使用 Node.js 构建了一个带有事件的模拟聊天应用程序。我们可以看到在 Node.js 中传递事件是多么容易，以及我们如何正确使用它。事件驱动编程并不适用于每个应用程序，但是当我们需要将多个组件连接在一起时，使用事件来实现这种逻辑要容易得多。上述代码仍然可以改进-我们可以在用户离开房间时向房间添加通知，并且我们可以在添加和移除房间时添加检查，以确保我们不会添加重复的房间，并确保我们只移除我们所在的房间。请随意自行扩展此功能。

在本章中，我们讨论了如何使用事件来管理应用程序中组件之间的通信。在下一个活动中，我们将构建一个基于事件驱动的模块。

### 活动 13：构建一个基于事件驱动的模块

假设您正在为一家构建烟雾探测器模拟器的软件公司工作。您需要构建一个烟雾探测器模拟器，当探测器的电池电量低于一定水平时会引发警报。以下是要求：

+   探测器需要发出`警报事件`。

+   当电池低于 0.5 单位时，烟雾探测器需要发出*低电量*事件。

+   每个烟雾探测器在初始创建时都有 10 个单位的电池电量。

+   烟雾探测器上的测试函数如果电池电量高于 0 则返回 true，如果低于 0 则返回 false。每次运行测试函数时，电池电量将减少 0.1 个单位。

+   您需要修改提供的`House`类以添加`addDetector`和`demoveDetector`方法。

+   `addDetector`将接受一个探测器对象，并在打印出*低电量*和*警报事件*之前，为警报事件附加一个监听器。

+   `removeDetector`方法将接受一个**探测器**对象并移除监听器。

完成此活动，执行以下步骤：

1.  打开`event.js`文件并找到现有的代码。然后，修改并添加你自己的更改。

1.  导入`events`模块。

1.  创建`SmokeDetector`类，该类扩展`EventEmitter`并将`batteryLevel`设置为`10`。

1.  在`SmokeDetector`类内创建一个`test`方法来发出*低电量*消息。

1.  创建`House`类，它将存储我们警报的实例。

1.  在`House`类中创建一个`addDetector`方法，它将附加事件监听器。

1.  创建一个`removeDetector`方法，它将帮助我们移除之前附加的*警报事件*监听器。

1.  创建一个名为`myHouse`的`House`实例。

1.  创建一个名为`detector`的`SmokeDetector`实例。

1.  将探测器添加到`myHouse`中。

1.  创建一个循环来调用测试函数 96 次。

1.  在`detector`对象上发出警报。

1.  从`myHouse`对象中移除探测器。

1.  在探测器上测试发出警报。

#### 注意

此活动的解决方案可以在第 617 页找到。

在这个活动中，我们学习了如何使用事件驱动编程来建模烟雾探测器。通过使用这种方法，我们消除了在我们的`House`对象中存储多个实例的需要，并避免使用大量代码来进行它们的交互。

在本节中，我们介绍了如何充分利用事件系统来帮助我们管理应用程序中的复杂通信。在下一节中，我们将介绍一些处理事件发射器的最佳实践。

### 事件驱动编程最佳实践

在前一章中，我们提到了使用事件发射器和事件发射器继承创建事件驱动组件的方法。但通常，您的代码需要的不仅仅是能够正确工作。拥有更好管理的代码结构不仅可以使我们的代码看起来不那么凌乱，还可以帮助我们避免将来一些可避免的错误。在本节中，我们将介绍在处理代码中的事件时的一些最佳实践。

回顾一下我们在本章开头所讨论的内容，我们可以使用`EventEmitter`对象传递事件：

```js
const EventEmitter = require('events');
const emitter = new EventEmitter();
emitter.emit('event');
```

当我们想要使用我们创建的事件发射器时，我们需要有它的引用，这样我们才能在以后想要发出事件时附加监听器并调用发射器的`emit`函数。这可能会导致我们的源代码非常庞大，这将使未来的维护非常困难：

```js
const EventEmitter = require('events');
const userEmitter = new EventEmitter();
const registrationEmitter = new EventEmitter();
const votingEmitter = new EventEmitter();
const postEmitter = new EventEmitter();
const commentEmitter = new EventEmitter();
userEmitter.on('update', (diff) => {
    userProfile.update(diff);
});
registrationEmitter.on('user registered:activated', (user) => {
    database.add(user, true);
});
registrationEmitter.on('user registered: not activated', (user) => {
    database.add(user, false);
});
votingEmitter.on('upvote', () => {
    userProfile.addVote();
});
votingEmitter.on('downvote', () => {
    userProfile.removeVote();
});
postEmitter.on('new post', (post) => {
    database.addPost(post);
});
postEmitter.on('edit post', (post) => {
    database.upsertPost(post);
});
commentEmitter.on('new comment', (comment) => {
    database.addComment(comment.post, comment);
});
```

为了能够使用我们的发射器，我们需要确保我们的发射器在当前范围内是可访问的。做到这一点的一种方法是创建一个文件来保存所有我们的发射器和附加事件监听器的逻辑。虽然这样大大简化了我们的代码，但我们将创建非常庞大的源代码，这将使未来的开发人员困惑，甚至可能连我们自己也会困惑。为了使我们的代码更模块化，我们可以开始将所有的监听器函数放入它们各自的文件中。考虑以下庞大的源代码：

```js
// index.js
const EventEmitter = require('events');
const userEmitter = new EventEmitter();
const registrationEmitter = new EventEmitter();
const votingEmitter = new EventEmitter();
const postEmitter = new EventEmitter();
const commentEmitter = new EventEmitter();
// Listeners
const updateListener = () => {};
const activationListener = () => {};
const noActivationListener = () => {};
const upvoteListener = () => {};
const downVoteListener = () => {};
const newPostListener = () => {};
const editPostListener = () => {};
const newCommentListener = () => {};
userEmitter.on('update', updateListener);
registrationEmitter.on('user registered:activated', activationListener);
registrationEmitter.on('user registered: not activated', noActivationListener);
votingEmitter.on('upvote', upvoteListener);
votingEmitter.on('downvote', downVoteListener);
postEmitter.on('new post', newPostListener);
postEmitter.on('edit post', editPostListener);
commentEmitter.on('new comment', newCommentListener);
```

仅仅通过这样做，我们已经大大减少了我们代码的文件大小。但我们可以做得更多。保持我们代码有组织的一种方法是将所有的发射器放在一个文件中，然后在需要时导入它。我们可以通过创建一个名为`emitters.js`的文件，并将所有的发射器存储在该文件中来实现这一点：

```js
// emitters.js
const EventEmitter = require('events');
const userEmitter = new EventEmitter();
const registrationEmitter = new EventEmitter();
const votingEmitter = new EventEmitter();
const postEmitter = new EventEmitter();
const commentEmitter = new EventEmitter();
module.exports = {
    userEmitter,
    registrationEmitter,
    votingEmitter,
    postEmitter,
    commentEmitter
};
```

我们在这里所做的是在一个文件中创建所有的发射器，并将该`emitter`文件设置为导出模块。通过这样做，我们可以将所有的发射器放在一个地方，然后当我们使用发射器时，我们只需导入该文件。这将改变我们的代码如下：

```js
// index.js
// Emitters
const {
    userEmitter,
    registrationEmitter,
    votingEmitter,
    postEmitter,
    commentEmitter
} = require('./emitters.js');
... rest of index.js
```

现在，当我们导入`emitter.js`时，我们可以使用对象解构来只选择我们想要的发射器。我们可以在一个文件中拥有多个发射器，然后在需要时选择我们想要的发射器。当我们想要在`userEmitter`上发出事件时，我们只需将发射器导入我们的代码并发送该事件即可：

```js
const { userEmitter } = require('./emitters.js');
function userAPIHandler(request, response) {
    const payload = request.payload;
    const event = {
        diff: payload.diff
    };
    userEmitter.emit('update', event);
}
```

我们可以看到，每当我们想要使用`userEmitter`时，我们只需导入我们的`emitter`文件。当我们想要附加监听器时，也适用这一点：

```js
const { userEmitter } = require('./emitters.js');
userEmitter.on('update', (diff) => {
    database.update(diff);
})
```

当我们将我们的发射器分成不同的文件时，我们不仅使我们的代码更小，而且使其更模块化。通过将我们的发射器拉入一个单独的文件，如果我们将来想要访问我们的发射器，那么我们可以很容易地重用该文件。通过这样做，我们不需要在函数中传递我们的发射器，从而确保我们的函数声明不会混乱。

## Node.js 内置模块

在前一节中，我们广泛讨论了`events`模块，并学习了如何使用事件来实现应用程序内的简单通信。`events`模块是 Node.js 提供的内置模块，这意味着我们不需要使用`npm`来安装它。在这个模块中，我们将讨论如何使用`fs`、`path`和`util`模块。

### path

`path`模块是一个内置模块，提供了一些工具，可以帮助我们处理文件路径和文件名。

**path.join(…paths)**

`Path.join()`是一个非常有用的函数，当我们在应用程序中处理目录和文件时。它允许我们将路径连接在一起，并输出一个路径字符串，我们可以在**fs**模块中使用。要使用`join`路径，我们可以调用`join`方法，并为其提供一组路径。让我们看下面的例子：

```js
const currentDir = '/usr/home/me/Documents/project';
const dataDir = './data';
const assetDir = './assets';
```

如果我们想要访问我们当前目录中的数据目录，我们可以使用`path.join`函数将不同的路径组合成一个字符串：

```js
const absoluteDataDir = path.join(currentDir, dataDir);
```

以下是前面代码的输出：

![图 9.29：使用 path.join 函数组合不同的路径](img/C14587_09_29.jpg)

###### 图 9.29：使用 path.join 函数组合不同的路径

它还可以处理`..`和`.`，如果你熟悉 POSIX 系统如何表示当前目录和父目录。`..`表示父目录，而`.`表示当前目录。例如，以下代码可以给出我们当前目录的父目录的路径：

```js
const parentOfProject = path.join(currentDir, '..');
```

以下是前面代码的输出：

![图 9.30：显示我们当前目录的父目录](img/C14587_09_30.jpg)

###### 图 9.30：显示我们当前目录的父目录

**path.parse(path)**

当我们想要获取有关文件路径的信息时，我们可以使用`path.parse()`函数来获取其根目录、基本目录、文件名和扩展名。让我们看下面的例子：

```js
const myData = '/usr/home/me/Documents/project/data/data.json';
```

如果我们想要解析这个文件路径，我们可以使用`path.parse`调用`myData`字符串来获取不同的路径元素：

```js
path.parse(myData);
```

这将生成以下输出：

![图 9.31：使用 path.parse 函数解析的文件路径](img/C14587_09_31.jpg)

###### 图 9.31：使用 path.parse 函数解析的文件路径

在这里，我们可以看到我们的文件路径包括一个文件名，基本名称为`data.json`。扩展名是`.json`，文件名是`data`。它还解析出文件所在的目录。

**path.format(path)**

在前面的`parse`函数中，我们成功地将文件路径解析为其各自的组件。我们可以使用`path.format`将这些信息组合成一个单一的字符串路径。让我们来看一下：

```js
path.format({
    dir: '/usr/home/me/Pictures',
    name: 'me',
    ext: '.jpeg'
});
```

以下是前面代码的输出：

![图 9.32：使用 path.format 将信息组合成单个字符串路径](img/C14587_09_32.jpg)

###### 图 9.32：使用 path.format 将信息组合成单个字符串路径

这给我们提供了从我们提供给它的组件中生成的文件路径。

### fs

**fs**模块是一个内置模块，为您提供 API，以便您可以与主机文件系统进行交互。当我们需要在应用程序中处理文件时，它非常有用。在本节中，我们将讨论如何在我们的应用程序中使用**fs**模块与`async`和`await`。稍后，我们将介绍最近添加的`fs.promises`API，它提供相同的功能，但返回一个 promise 而不是使用回调。

#### 注意

在这一部分，我们将使用 POSIX 系统。如果你使用的是 Windows 系统，请确保将文件路径更新为 Windows 的等价物。要将 fs 模块导入到你的代码中，执行以下命令：

```js
const fs = require('fs');
```

**fs.createReadStream(path, options)**

当我们在 Node.js 中处理大文件时，建议始终使用`stream`。要创建一个读取流，我们可以调用`fs.createReadStream`方法。它将返回一个流对象，我们可以将其附加到事件处理程序，以便它们获取文件的内容：

```js
const stream = fs.createReadStream('file.txt', 'utf-8');
```

**fs.createWriteStream(path, options)**

这与`createReadStream`类似，但是创建了一个可写流，我们可以用它来流式传输内容：

```js
const stream = fs.createWriteStream('output', 'utf-8');
```

**fs.stat(path, callback)**

当我们需要关于我们正在访问的文件的详细信息时，`fs.stat`方法非常有用。我们还看到许多开发人员在调用、打开、读取或写入数据之前使用`fs.stat`来检查文件的存在。虽然使用`stat`检查文件的存在不会创建任何新问题，但不建议这样做。我们应该只使用从我们使用的函数返回的错误；这将消除任何额外的逻辑层，并可以减少 API 调用的数量。

考虑以下例子：

```js
const fs = require('fs');
fs.stat('index.js', (error, stat) => {
    console.log(stat);
});
```

这将给我们一个类似以下的输出：

![图 9.33：使用 fs.stat 方法后的输出](img/C14587_09_33.jpg)

###### 图 9.33：使用 fs.stat 方法后的输出

**fs.readFile(path, options, callback)**

这是大多数人熟悉的函数。当提供文件路径时，该方法将尝试以异步方式读取文件的整个内容。它将以异步方式执行，并且回调将被调用以获取文件的整个内容。当文件不存在时，回调将被调用以获取错误。

考虑以下例子：

```js
const fs = require('fs');
fs.readFile('index.js', (error, data) => {
    console.log(data);
});
```

这将给我们以下输出：

![图 9.34：使用 fs.readFile 函数读取文件的整个内容](img/C14587_09_34.jpg)

###### 图 9.34：使用 fs.readFile 函数读取文件的整个内容

这没有输出我们想要的结果。这是因为我们没有在选项中提供编码；要将内容读入字符串，我们需要提供编码选项。这将改变我们的代码为以下内容：

```js
fs.readFile('index.js', 'utf-8', (error, data) => {
    console.log(data);
});
```

现在，当我们运行上述代码时，它会给我们以下输出：

![图 9.35：使用 fs.readFile 函数读取文件的整个内容后编码](img/C14587_09_35.jpg)

###### 图 9.35：使用 fs.readFile 函数读取文件的整个内容后编码

我们刚刚做了一个输出自身的程序。

**fs.readFileSync(path, options)**

这个函数和`readFile`方法做的事情一样，但是以同步的方式执行`read`函数，这意味着它会阻塞执行。在程序启动期间，建议 - 并且期望 - 只调用一次。当需要多次调用时，不建议使用同步函数。

**fs.writeFile(file, data, options, callback)**

`writeFile`函数将数据写入我们指定的文件。它还将替换现有的文件，除非你在选项中传递一个附加的`flag`。

**fs.writeFileSync()**

就像`readFileSync`一样，它和它的非同步对应物做的事情一样。它们之间的区别在于这个是同步执行操作。

### 练习 69：Fs 模块的基本用法

在这个练习中，我们将使用`fs`模块来读取和写入应用程序中的文件。我们将使用我们在前一节中介绍的方法，并将它们与回调一起使用。然后，我们将对它们进行`promisify`，这样我们就可以用`async`和`await`来使用它们。

执行以下步骤完成这个练习：

1.  创建一个名为`test.txt`的新文件：

```js
fs.writeFile('test.txt', 'Hello world', (error) => {
    if (error) {
        console.error(error);
        return;
    }
    console.log('Write complete');
});
```

如果你做对了，你会看到以下输出：

![图 9.36：创建新的 test.txt 文件](img/C14587_09_36.jpg)

###### 图 9.36：创建新的 test.txt 文件

你应该能够在与源代码相同的目录中看到新文件：

![图 9.37：在与源代码相同的目录中创建新文件](img/C14587_09_37.jpg)

###### 图 9.37：在与源代码相同的目录中创建新文件

1.  读取其内容并在控制台中输出：

```js
fs.readFile('test.txt', 'utf-8', (error, data) => {
    if (error) {
        console.error(error);
    }
    console.log(data);
});
```

这只是简单地读取我们的文件；我们提供了一个编码，因为我们希望输出是一个字符串而不是一个缓冲区。这将给我们以下输出：

![图 9.38：使用 fs.readFile 读取文件内容](img/C14587_09_38.jpg)

###### 图 9.38：使用 fs.readFile 读取文件内容

1.  尝试读取一个不存在的文件：

```js
fs.readFile('nofile.txt', 'utf-8', (error, data) => {
    if (error) {
        console.error(error);
    }
    console.log(data);
});
```

当我们尝试打开一个不存在的文件时，我们的回调将会被调用并出现错误。建议我们在处理任何与文件相关的错误时，应该在处理程序内部处理，而不是创建一个单独的函数来检查它。当我们运行上述代码时，将会得到以下错误：

![图 9.39：尝试读取不存在的文件时抛出的错误](img/C14587_09_39.jpg)

###### 图 9.39：尝试读取不存在的文件时抛出的错误

1.  让我们创建自己的带有 promise 的`readFile`版本：

```js
function readFile(file, options) {
    return new Promise((resolve, reject) => {
        fs.readFile(file, options, (error, data) => {
            if (error) {
                return reject(error);
            }
            resolve(data);
        })
    })
}
```

这与我们可以使用任何基于回调的方法做的事情是一样的，如下所示：

```js
readFile('test.txt', 'utf-8').then(console.log);
```

这将生成以下输出：

![图 9.40：使用基于回调的方法创建 readFile](img/C14587_09_40.jpg)

###### 图 9.40：使用基于回调的方法创建 readFile

1.  让我们使用文件`stat`来获取有关我们文件的信息。在 Node.js 10.0.0 之后，引入了`fsPromises`，因此我们可以简单地导入`fsPromise`并调用 promise 的对应项，而不是手动将它们转换为 promise 并手动返回函数：

```js
const fsPromises = require('fs').promises;
fsPromises.stat('test.txt').then(console.log);
```

这将生成以下输出：

![图 9.41：通过导入 fspromise 调用 promise 的对应项](img/C14587_09_41.jpg)

###### 图 9.41：通过导入 fspromise 调用 promise 的对应项

在这里，你可以获取有关我们文件的大小、创建时间、修改时间和权限信息。

在这个练习中，我们介绍了**fs**模块的一些基本用法。这是 Node.js 中一个非常有用的模块。接下来，我们将讨论如何在 Node.js 中处理大文件。

## 在 Node.js 中处理大文件

在上一个练习中，我们讨论了如何使用`fs`模块在 Node.js 中读取文件内容。当处理小于 100MB 的小文件时，这很有效。当处理大文件（> 2GB）时，有时使用`fs.readFile`无法读取整个文件。考虑以下情况。

你得到了一个 20GB 的文本文件，你需要逐行处理文件中的数据，并将输出写入输出文件。你的计算机只有 8GB 的内存。

当你使用`fs.readFile`时，它会尝试将文件的整个内容读入计算机的内存中。在我们的情况下，这是不可能的，因为我们的计算机没有足够的内存来容纳我们正在处理的文件的整个内容。在这里，我们需要一个单独的方法来解决这个问题。为了处理大文件，我们需要使用流。

流是编程中一个有趣的概念。它将数据视为不是单一的内存块，而是来自源的数据流，每次一个数据块。这样，我们就不需要将所有数据都放入内存中。要创建一个文件流，我们只需使用`fs`模块中提供的方法：

```js
const fs = require('fs');
const stream = fs.createReadStream('file.txt', 'utf-8');
```

通过使用`fs.createReadStream`，我们创建了一个文件流，以便稍后可以获取文件的内容。我们像使用`fs.readFile`一样调用这个函数，传入文件路径和编码。与`fs.readFile`的区别在于，这不需要提供回调，因为它只是返回一个`stream`对象。要从流中获取文件内容，我们需要将事件处理程序附加到`stream`对象上：

```js
stream.on('data', data => {
    // Data will be the content of our file
    Console.log(data);
    // Or
    Data = data + data;
});
```

在`data`事件的事件处理程序中，我们将获得文件的内容，并且当流读取文件时，此处理程序将被多次调用。当我们完成读取文件时，流对象还会发出一个事件来处理此事件：

```js
stream.on('close', () => {
    // Process clean up process
});
```

### Util

**Util**是一个包含许多有助于 Node.js 内部 API 的函数的模块。这些也可以在我们自己的开发中很有用。

**util.callbackify(function)**

这在我们处理现有的基于回调的遗留代码时非常有用。要将我们的`async`函数用作基于回调的函数，我们可以调用`util.callbackify`函数。让我们考虑以下示例：

```js
async function outputSomething() {
    return 'Something';
}
outputSomething().then(console.log);
```

以下是前面代码的输出：

![图 9.42：将 async 函数用作基于回调的函数](img/C14587_09_42.jpg)

###### 图 9.42：将 async 函数用作基于回调的函数

要将此`async`函数与回调一起使用，只需调用`callbackify`：

```js
const callbackOutputSomething = util.callbackify(outputSomething);
```

然后，我们可以这样使用它：

```js
callbackOutputSomething((err, result) => {
    if (err) throw err;
    console.log('got result', result);
})
```

这将生成以下输出：

![图 9.43：通过调用 callbackify 函数使用 async 函数](img/C14587_09_43.jpg)

###### 图 9.43：通过调用 callbackify 函数使用 async 函数

我们已成功将`async`函数转换为使用回调的遗留函数。当我们需要保持向后兼容性时，这非常有用。

**util.promisify(function)**

`util`模块中还有一个非常有用的方法，可以帮助我们将基于回调的函数转换为`promisify`函数。该方法以一个函数作为参数，并将返回一个返回 promise 的新函数，如下所示：

```js
function callbackFunction(param, callback) {
    callback(null, 'I am calling back with: ${param}');
}
```

`callbackFunction`接受一个参数，并将使用我们提供的新字符串调用回调函数。要将此函数转换为使用 promises，我们可以使用`promisify`函数：

```js
const promisifiedFunction = util.promisify(callbackFunction);
```

这将返回一个新函数。稍后，我们可以将其用作返回 promise 的函数：

```js
promisifiedFunction('hello world').then(console.log);
```

以下是前面代码的输出：

![图 9.44：使用 promisify 函数进行回调](img/C14587_09_44.jpg)

###### 图 9.44：使用 promisify 函数进行回调

`util`模块中还有许多类型检查方法，在我们尝试确定应用程序中变量类型时非常有用。

### Timer

计时器模块为我们提供了一个用于调度计时器函数的 API。我们可以使用它在代码的某些部分设置延迟，或者在所需的间隔执行我们的代码。与之前的模块不同，不需要在使用之前导入`timer`模块。让我们看看 Node.js 提供的所有计时器函数以及如何在我们的应用程序中使用它们。

**setInterval(callback, delay)**

当我们想要设置一个在 Node.js 中重复执行的函数时，我们可以使用`setInterval`函数，并提供一个回调和延迟。要使用它，我们调用`setInterval`函数，并提供我们想要运行的函数以及以毫秒为单位的延迟。例如，如果我们想每秒打印相同的消息，我们可以这样实现：

```js
setInterval(() => {
    console.log('I am running every second');
}, 1000);
```

当我们运行前面的代码时，将看到以下输出：

![图 9.45：使用 setInterval 函数设置重复执行的函数](img/C14587_09_45.jpg)

###### 图 9.45：使用 setInterval 函数设置重复执行的函数

在这里，我们可以看到消息每秒打印一次。

**setTimeout(callback, delay)**

使用此函数，我们可以设置一次性延迟调用函数。当我们想要在运行函数之前等待一定的时间时，我们可以使用`setTimeout`来实现这一点。在前面的部分中，我们还使用`setTimeout`来模拟测试中的网络和磁盘请求。要使用它，我们需要传递我们想要运行的函数和以毫秒为单位的延迟整数。如果我们想在 3 秒后打印一条消息，我们可以使用以下代码：

```js
setTimeout(() => {
    console.log('I waited 3 seconds to run');
}, 3000);
```

这将生成以下输出：

![图 9.46：使用 setTimeout 函数设置一次延迟调用函数](img/C14587_09_46.jpg)

###### 图 9.46：使用 setTimeout 函数设置一次延迟调用函数

您将看到消息在 3 秒后打印出。当我们需要延迟调用函数或只想在测试中使用它模拟 API 调用时，这非常有用。

**setImmediate(callback)**

通过使用这种方法，我们可以推送一个函数，在事件循环结束时执行。如果您想在当前事件循环中的所有内容完成运行后调用某段代码，可以使用`setImmediate`来实现这一点。看一下以下示例：

```js
setImmediate(() => {
    console.log('I will be printed out second');
});
console.log('I am printed out first');
```

在这里，我们创建了一个函数，打印出`I will be printed out second`，它将在事件循环结束时执行。当我们执行这个函数时，我们将看到以下输出：

![图 9.47：使用 setimmediate 推送到事件循环结束时执行的函数](img/C14587_09_47.jpg)

###### 图 9.47：使用 setimmediate 推送到事件循环结束时执行的函数

我们也可以通过使用`setTimeout`并将`0`作为延迟参数来实现相同的效果：

```js
setTimeout(() => {
    console.log('I will be printed out second');
}, 0);
console.log('I am printed out first');
```

**clearInterval(timeout)**

当我们使用`setInterval`创建一个重复的函数时，该函数还会返回表示计时器的对象。当我们想要停止间隔运行时，我们可以使用`clearInterval`来清除计时器：

```js
const myInterval = setInterval(() => {
    console.log('I am being printed out');
}, 1000);
clearInterval(myInterval);
```

当我们运行上述代码时，我们将看到没有输出产生，因为我们清除了刚刚创建的间隔，并且它从未有机会运行：

![图 9.48：使用 clearInterval 函数停止间隔运行](img/C14587_09_48.jpg)

###### 图 9.48：使用 clearInterval 函数停止间隔运行

如果我们想要运行这个间隔 3 秒，我们可以将`clearInterval`包装在`setTimeout`内，这样它将在`3.1`秒后清除我们的间隔。我们额外给了 100 毫秒，因为我们希望第三次调用发生在我们清除间隔之前：

```js
setTimeout(() => {
    clearInterval(myInterval);
}, 3100);
```

当我们运行上述代码时，我们将看到我们的输出打印出 3 次：

![图 9.49：使用 setTimeout 在指定的秒数内包装 clearInterval](img/C14587_09_49.jpg)

###### 图 9.49：使用 setTimeout 在指定的秒数内包装 clearInterval

当我们处理多个预定计时器时，这非常有用。通过清除它们，我们可以避免内存泄漏和应用程序中的意外问题。

### 活动 14：构建文件监视器

在这个活动中，我们将使用定时器函数创建一个文件监视器，该监视器将指示文件中的任何修改。这些定时器函数将在文件上设置监视，并在文件发生更改时生成输出。让我们开始吧：

+   我们需要创建一个`fileWatcher`类。

+   将创建一个带有要监视的文件的文件监视器。如果没有文件存在，它将抛出异常。

+   文件监视器将需要另一个参数来存储检查之间的时间。

+   文件监视器需要允许我们移除对文件的监视。

+   文件监视器需要在文件更改时发出文件更改事件。

+   当文件更改时，文件监视器将发出带有文件新内容的事件。

打开`filewatcher.js`文件，并在该文件中进行您的工作。执行以下步骤以完成此活动：

1.  导入我们的库；即`fs`和`events`。

1.  创建一个文件监视器类，该类扩展了`EventEmitter`类。使用`modify`时间戳来跟踪文件更改。

1.  创建`startWatch`方法以开始监视文件的更改。

1.  创建`stopWatch`方法以停止监视文件的更改。

1.  在与`filewatch.js`相同的目录中创建一个`test.txt`文件。

1.  创建一个`FileWatcher`实例并开始监视文件。

1.  修改`test.txt`中的一些内容并保存。

1.  修改`startWatch`以便还检索新内容。

1.  修改`startWatch`，使其在文件被修改时发出事件，并在遇到错误时发出错误。

1.  在`fileWatcher`中附加事件处理程序以处理错误和更改。

1.  运行代码并修改`test.txt`以查看结果。

#### 注意

这个活动的解决方案可以在第 620 页找到。

如果您看到前面的输出，这意味着您的事件系统和文件读取完全正常。请随意扩展这个功能。您也可以尝试启用监视整个文件夹或多个文件。在这个活动中，我们只是使用文件系统模块和事件驱动编程创建了一个简单的`fileWatcher`类。使用这个帮助我们创建了一个更小的代码库，并在直接阅读代码时给了我们更多的清晰度。

## 总结

在本章中，我们讨论了 JavaScript 中的事件系统，以及如何使用内置的`events`模块来创建我们自己的事件发射器。后来，我们介绍了一些有用的内置模块及其示例用法。使用事件驱动编程可以帮助我们避免在编写需要多个组件相互通信的程序时出现交织的逻辑。此外，通过使用内置模块，我们可以避免添加提供相同功能的模块，并避免创建具有巨大依赖关系的项目。我们还提到了如何使用定时器来控制程序执行，使用`fs`来操作文件，以及使用`path`来组合和获取有关我们文件路径的有用信息。这些都是非常有用的模块，可以在构建应用程序时帮助我们。在下一章中，我们将讨论如何在 JavaScript 中使用函数式编程。
