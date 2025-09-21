# 1

# 与创建型设计模式一起工作

JavaScript 设计模式是允许我们在 JavaScript 中编写更健壮、可扩展和可扩展的应用程序的技术。JavaScript 是一种非常流行的编程语言，部分原因是它作为在网页上提供交互功能的一种方式。其受欢迎的另一个原因是 JavaScript 的轻量级、动态、多范式特性，这意味着其他生态系统的设计模式可以适应以利用 JavaScript 的优势。JavaScript 的具体优势和劣势也可以为语言及其使用环境中的新模式提供信息。

创建型设计模式为对象创建提供结构，这使得开发不需要知道如何创建彼此实例的不同模块、类和对象的应用程序和系统成为可能。与 JavaScript 最相关的模式——原型、单例和工厂模式——将被探讨，以及它们何时有用以及如何以惯用的方式实现。

本章我们将涵盖以下主题：

+   创建型设计模式的全面定义以及原型、单例和工厂模式的定义

+   原型模式的多种实现及其用例

+   单例设计模式的实现，包括急切和延迟初始化，单例的用例以及现代 JavaScript 中的单例模式是什么样的

+   如何使用类、现代 JavaScript 的替代方案以及用例来实现工厂模式

到本章结束时，你将能够识别何时使用创建型设计模式是有用的，并就其多种实现中选择一个明智的决定，从更惯用的 JavaScript 形式到经典形式。

# 什么是创建型设计模式？

创建型设计模式处理对象创建。它们允许消费者创建对象实例，而无需了解如何实例化对象的具体细节。由于在面向对象的语言中，对象的实例化仅限于类的构造函数，因此允许不调用构造函数就创建对象实例可以减少消费者和被实例化的类之间的噪声和紧密耦合。

在 JavaScript 中，当我们讨论“对象创建”时存在歧义，因为 JavaScript 的多范式特性意味着我们可以不使用类或构造函数来创建对象。例如，在 JavaScript 中，这是一个使用对象字面量进行对象创建的例子 – `const config = { forceUpdate: true` `}`。实际上，现代惯用的 JavaScript 往往更倾向于过程和函数范式，而不是面向对象。这意味着创建型设计模式可能需要调整才能在 JavaScript 中完全有用。

总结来说，在面向对象的 JavaScript 中，创建型设计模式非常有用，因为它们隐藏了实例化细节，从而降低了耦合度，允许更好的模块分离。

在下一节中，我们将遇到第一个创建型设计模式——原型设计模式。

# 在 JavaScript 中实现原型模式

让我们先定义一下原型模式。

原型设计模式允许我们根据另一个现有实例（我们的原型）创建一个实例。

更正式地说，一个 `prototype` 类公开了一个 `clone()` 方法。消费代码，而不是调用 `new SomeClass`，将调用 `new SomeClassPrototype(someClassInstance).clone()`。这个方法调用将返回一个带有从 `someClassInstance` 复制的所有值的 `new SomeClass` 实例。

## 实现

让我们想象一个场景，我们正在构建一个棋盘。有两种关键的方块类型——白色和黑色。除了这些信息之外，每个方块还包含其行、列以及位于其上的棋子信息。

一个 `BoardSquare` 类构造函数可能看起来如下：

```js
class BoardSquare {
  constructor(color, row, file, startingPiece) {
    this.color = color;
    this.row = row;
    this.file = file;
  }
}
```

`BoardSquare` 上可能有一组有用的方法，例如 `occupySquare` 和 `clearSquare`，如下所示：

```js
class BoardSquare {
  // no change to the rest of the class
  occupySquare(piece) {
    this.piece = piece;
  }
  clearSquare() {
    this.piece = null;
  }
}
```

由于 `BoardSquare` 有很多属性，实例化它相当繁琐：

```js
const whiteSquare = new BoardSquare('white');
const whiteSquareTwo = new BoardSquare('white');
// ...
const whiteSquareLast = new BoardSquare('white');
```

注意传递给 `new BoardSquare` 的参数重复，如果我们想将所有棋盘方块变为黑色，我们需要逐个更改每个 `new BoardSquare` 调用的参数。这可能会非常容易出错；只需在 `color` 值中犯一个难以发现的错误就可能导致错误：

```js
const blackSquare = new BoardSquare('black');
const blackSquareTwo = new BoardSquare('black');
// ...
const blackSquareLast = new BoardSquare('black');
```

使用经典的原型实现我们的实例化逻辑如下。我们需要一个 `BoardSquarePrototype` 类；其构造函数接受一个 `prototype` 属性，并将其存储在实例上。`BoardSquarePrototype` 公开了一个 `clone()` 方法，该方法不接受任何参数，并返回一个带有 `prototype` 上所有属性的 `BoardSquare` 实例：

```js
class BoardSquarePrototype {
  constructor(prototype) {
    this.prototype = prototype;
  }
  clone() {
    const boardSquare = new BoardSquare();
    boardSquare.color = this.prototype.color;
    boardSquare.row = this.prototype.row;
    boardSquare.file = this.prototype.file;
    return boardSquare;
  }
}
```

使用 `BoardSquarePrototype` 需要以下步骤：

1.  首先，我们想要一个 **BoardSquare** 的实例来初始化——在这种情况下，使用 **'white'**。然后，它将在调用 **BoardSquarePrototype** 构造函数时作为 **prototype** 属性传递：

    ```js
    const whiteSquare = new BoardSquare('white');
    const whiteSquarePrototype = new BoardSquarePrototype
      (whiteSquare);
    ```

1.  然后，我们可以使用 **whiteSquarePrototype** 与 **.clone()** 方法来创建 **whiteSquare** 的副本。注意，**color** 被复制，但每次调用 **clone()** 都会返回一个新的实例。

    ```js
    const whiteSquareTwo = whiteSquarePrototype.clone();
    // ...
    const whiteSquareLast = whiteSquarePrototype.clone();
    console.assert(
      whiteSquare.color === whiteSquareTwo.color &&
        whiteSquareTwo.color === whiteSquareLast.color,
      'Prototype.clone()-ed instances have the same color
         as the prototype'
    );
    console.assert(
      whiteSquare !== whiteSquareTwo &&
        whiteSquare !== whiteSquareLast &&
        whiteSquareTwo !== whiteSquareLast,
      'each Prototype.clone() call outputs a different
         instances'
    );
    ```

根据代码中的断言，克隆的实例具有相同的 `color` 值，但它们是 `Square` 对象的不同实例。

## 一个用例

为了说明从白色方块变为黑色方块需要改变什么，让我们看看一些示例代码，其中变量名中没有引用 `'white'`：

```js
const boardSquare = new BoardSquare('white');
const boardSquarePrototype = new BoardSquarePrototype(boardSquare);
const boardSquareTwo = boardSquarePrototype.clone();
// ...
const boardSquareLast = boardSquarePrototype.clone();
console.assert(
  boardSquareTwo.color === 'white' &&
    boardSquare.color === boardSquareTwo.color &&
    boardSquareTwo.color === boardSquareLast.color,
  'Prototype.clone()-ed instances have the same color as
     the prototype'
);
console.assert(
  boardSquare !== boardSquareTwo &&
    boardSquare !== boardSquareLast &&
    boardSquareTwo !== boardSquareLast,
  'each Prototype.clone() call outputs a different
    instances'
);
```

在这种情况下，我们只需更改传递给 `BoardSquare` 的 `color` 值，就可以改变从原型克隆的所有实例的颜色：

```js
const boardSquare = new BoardSquare('black');
// rest of the code stays the same
console.assert(
  boardSquareTwo.color === 'black' &&
    boardSquare.color === boardSquareTwo.color &&
    boardSquareTwo.color === boardSquareLast.color,
  'Prototype.clone()-ed instances have the same color as
     the prototype'
);
console.assert(
  boardSquare !== boardSquareTwo &&
    boardSquare !== boardSquareLast &&
    boardSquareTwo !== boardSquareLast,
  'each Prototype.clone() call outputs a different
     instances'
);
```

原型模式在需要对象实例的“模板”的情况下很有用。这是一个创建“默认对象”但带有自定义值的好模式。它允许更快、更轻松地进行更改，因为它们在模板对象上只实现一次，但应用于所有 `clone()`-ed 实例。

### 使用现代 JavaScript 增强对原型实例变量变化的鲁棒性

我们可以在 JavaScript 的原型实现中做出一些改进。

第一个是在 `clone()` 方法中。为了使我们的原型类对原型构造函数/实例变量的变化具有鲁棒性，我们应该避免逐个复制属性。

例如，如果我们添加一个新的 `startingPiece` 参数，该参数由 `BoardSquare` 构造函数接受并将 `piece` 实例变量设置为，我们当前的 `BoardSquarePrototype` 实现将无法复制它，因为它只复制 `color`、`row` 和 `file`：

```js
class BoardSquare {
  constructor(color, row, file, startingPiece) {
    this.color = color;
    this.row = row;
    this.file = file;
    this.piece = startingPiece;
  }
  // same rest of the class
}
const boardSquare = new BoardSquare('white', 1, 'A',
  'king');
const boardSquarePrototype = new BoardSquarePrototype
  (boardSquare);
const otherBoardSquare = boardSquarePrototype.clone();
console.assert(
  otherBoardSquare.piece === undefined,
  'prototype.piece was not copied over'
);
```

注意

**Object.assign** 的参考：[`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)。

如果我们将 `BoardSquarePrototype` 类修改为使用 `Object.assign(new BoardSquare(), this.prototype)`，它将复制 `prototype` 的所有可枚举属性：

```js
class BoardSquarePrototype {
  constructor(prototype) {
    this.prototype = prototype;
  }
  clone() {
    return Object.assign(new BoardSquare(), this.prototype);
  }
}
const boardSquare = new BoardSquare('white', 1, 'A',
  'king');
const boardSquarePrototype = new BoardSquarePrototype
  (boardSquare);
const otherBoardSquare = boardSquarePrototype.clone();
console.assert(
  otherBoardSquare.piece === 'king' &&
    otherBoardSquare.piece === boardSquare.piece,
  'prototype.piece was copied over'
);
```

### JavaScript 中没有类的原型模式

由于历史原因，JavaScript 在语言中深深嵌入了一个原型概念。实际上，类是在 ECMAScript 标准中后来引入的，ECMAScript 6 在 2015 年发布（参考，ECMAScript 1 在 1997 年发布）。

这就是为什么很多 JavaScript 完全放弃了使用类的使用。JavaScript 的“对象原型”可以用来使对象相互继承方法和变量。

克隆对象的一种方法是通过使用 `Object.create` 来克隆具有其方法的对象。这依赖于 JavaScript 原型系统：

```js
const square = {
  color: 'white',
  occupySquare(piece) {
    this.piece = piece;
  },
  clearSquare() {
    this.piece = null;
  },
};
const otherSquare = Object.create(square);
```

这里的一个细微差别是 `Object.create` 实际上并没有复制任何东西；它只是创建了一个新对象，并将其原型设置为 `square`。这意味着如果 `otherSquare` 上没有找到属性，它们将在 `square` 上访问：

```js
console.assert(otherSquare.__proto__ === square, 'uses JS
  prototype');
console.assert(
  otherSquare.occupySquare === square.occupySquare &&
    otherSquare.clearSquare === square.clearSquare,
  "methods are not copied, they're 'inherited' using the
     prototype"
);
delete otherSquare.color;
console.assert(
  otherSquare.color === 'white' && otherSquare.color ===
    square.color,
  'data fields are also inherited'
);
```

关于 JavaScript 原型的进一步说明，以及它在类成为 JavaScript 部分之前就存在，是 JavaScript 中的子类化是设置对象原型的另一种语法。看看下面的 `extends` 示例。`BlackSquare extends Square` 将 `BlackSquare` 的 `prototype.__proto__` 属性设置为 `Square.prototype`：

```js
class Square {
  constructor() {}
  occupySquare(piece) {
    this.piece = piece;
  }
  clearSquare() {
    this.piece = null;
  }
}
class BlackSquare extends Square {
  constructor() {
    super();
    this.color = 'black';
  }
}
console.assert(
  BlackSquare.prototype.__proto__ === Square.prototype,
  'subclass prototype has prototype of superclass'
);
```

在本节中，我们学习了如何使用具有公开 `clone()` 方法的原型类实现原型模式，原型模式可以帮助的代码情况，以及如何使用现代 JavaScript 功能进一步改进我们的原型实现。我们还涵盖了 JavaScript 的“原型”，为什么它存在，以及它与原型设计模式的关系。

在本章的下一部分，我们将探讨另一种创建型设计模式，即单例设计模式，以及一些在 JavaScript 中的实现方法和用例。

# JavaScript 中的单例模式，具有 eager 和 lazy 初始化

首先，让我们定义单例设计模式。

单例模式允许一个对象只被实例化一次，向消费者公开这个单一实例，并控制单一实例的实例化。

单例是另一种在不使用构造函数的情况下获取对象实例的方法，尽管对象必须设计为单例。

## 实现

单例的一个经典例子是日志记录器。在应用程序中很少需要（并且通常，这是一个问题）实例化多个日志记录器。拥有单例意味着初始化位置受到控制，并且日志记录器配置将在整个应用程序中保持一致——例如，日志级别不会根据我们在应用程序中从哪里调用日志记录器而改变。

一个简单的日志记录器看起来如下，它有一个接受 `logLevel` 和 `transport` 的构造函数，以及一个私有的 `isLevelEnabled` 方法，它允许我们丢弃日志记录器未配置保留的日志（例如，当级别是 `warn` 时，我们丢弃 `info` 消息）。日志记录器最终实现了 `info`、`warn` 和 `error` 方法，它们的行为如前所述；它们只有在级别“启用”时（即，“高于”配置的日志级别）才会调用相关的 `transport` 方法。

支持 `isLevelEnabled` 的可能 `logLevel` 值存储在 `Logger` 的静态字段上：

```js
class Logger {
  static logLevels = ['info', 'warn', 'error'];
  constructor(logLevel = 'info', transport = console) {
    if (Logger.#loggerInstance) {
      throw new TypeError(
        'Logger is not constructable, use getInstance()
           instead'
      );
    }
    this.logLevel = logLevel;
    this.transport = transport;
  }
  isLevelEnabled(targetLevel) {
    return (
      Logger.logLevels.indexOf(targetLevel) >=
      Logger.logLevels.indexOf(this.logLevel)
    );
  }
  info(message) {
    if (this.isLevelEnabled('info')) {
      return this.transport.info(message);
    }
  }
  warn(message) {
    if (this.isLevelEnabled('warn')) {
      this.transport.warn(message);
    }
  }
  error(message) {
    if (this.isLevelEnabled('error')) {
      this.transport.error(message);
    }
  }
}
```

为了使 `Logger` 成为单例，我们需要实现一个返回缓存实例的 `getInstance` 静态方法。为了做到这一点，我们将在 `Logger` 上使用一个静态的 `loggerInstance`。`getInstance` 将检查 `Logger.loggerInstance` 是否存在，如果存在则返回它；否则，它将创建一个新的 `Logger` 实例，将其设置为 `loggerInstance`，然后返回它：

```js
class Logger {
  static loggerInstance = null;
  // rest of the class
  static getInstance() {
    if (!Logger.loggerInstance) {
      Logger.loggerInstance = new Logger('warn', console);
    }
    return Logger.loggerInstance;
  }
}
```

在另一个模块中使用它就像调用 `Logger.getInstance()` 一样简单。所有的 `getInstance` 调用都将返回同一个 `Logger` 实例：

```js
const a = Logger.getInstance();
const b = Logger.getInstance();
console.assert(a === b, 'Logger.getInstance() returns the
  same reference');
```

我们实现了一个具有“lazy”初始化的单例。初始化发生在第一次调用 `getInstance` 时。在下一节中，我们将看到如何扩展我们的代码以实现 `loggerInstance` 的“eager”初始化，其中 `loggerInstance` 将在 `Logger` 代码评估时初始化。

### 确保只构建一个单例实例

单例的一个特点是“单一实例”概念。我们希望“强制”消费者使用 `getInstance` 方法。

为了做到这一点，我们可以在构造函数被调用时检查 `loggerInstance` 的存在：

```js
class Logger {
  // rest of the class
  constructor(logLevel = 'info', transport = console) {
    if (Logger.loggerInstance) {
      throw new TypeError(
        'Logger is not constructable, use getInstance()
          instead'
      );
    }
    this.logLevel = logLevel;
    this.transport = transport;
  }
  // rest of the class
}
```

在我们调用 `getInstance`（因此，`Logger.loggerInstance` 被填充）的情况下，构造函数现在将抛出一个错误：

```js
Logger.getInstance();
new Logger('info', console); // new TypeError('Logger is
  not constructable, use getInstance() instead');
```

这种行为有助于确保消费者不会实例化自己的记录器，而是使用`getInstance`。所有使用`getInstance`的消费者意味着设置记录器的配置被`Logger`类封装。

实现中仍然存在一个差距，因为如以下示例所示，在调用`getInstance()`之前构造`new Logger()`将成功：

```js
new Logger('info', console); // Logger { logLevel: 'info',
  transport: ... }
new Logger('info', console); // Logger { logLevel: 'info',
  transport: ... }
Logger.getInstance();
new Logger('info', console); // new TypeError('Logger is
  not constructable, use getInstance() instead');
```

在多线程语言中，我们的实现也可能存在潜在的竞争条件——多个消费者并发调用`Logger.getInstance()`可能会导致存在多个实例。然而，由于流行的 JavaScript 运行时是单线程的，我们不必担心这种竞争条件——`getInstance`是一个“同步”方法，所以对它的多次调用会被依次解释。作为参考，Node.js、Deno 以及主流浏览器 Chrome、Safari、Edge 和 Firefox 都提供了一个单线程的 JavaScript 运行时。

### eager-initialization 的单例

eager-initialization 可以用来确保单例已准备好使用，并且像在实例存在时禁用构造函数这样的特性对所有情况都有效。

我们可以通过在`Logger`构造函数中设置`Logger.loggerInstance`来 eager-initialize：

```js
class Logger {
  // rest of the class unchanged
  constructor(logLevel = 'info', transport = console) {
    // rest of the constructor unchanged
    Logger.loggerInstance = this;
  }
}
```

这种方法的一个缺点是构造函数执行全局状态突变，这在“单一职责原则”的角度来看并不理想；构造函数现在除了设置对象实例的责任之外，还有某种副作用（突变全局状态）。

通过在记录器的模块中运行`Logger.getInstance()`来 eager-initialize 的另一种方法是，它与一个`export` `default`语句配对很有用：

```js
export class Logger {
  // no changes to the Logger class
}
export default Logger.getInstance();
```

在添加了前面的导出之后，现在有两种方式可以访问记录器实例。第一种是通过按名称导入`Logger`并调用`Logger.getInstance()`：

```js
import { Logger } from './logger.js';
const logger = Logger.getInstance();
logger.warn('testing testing 12'); // testing testing 12
```

使用记录器的第二种方式是通过导入默认导出：

```js
import logger from './logger.js';
logger.warn('testing testing 12'); // testing testing 12
```

任何现在导入`Logger`的代码都将获得一个预先确定的记录器单例实例。

## 用例

单例模式在应用程序中仅应有一个对象实例时特别出色——例如，一个不需要在每次请求中设置/拆除的记录器。

由于单例类控制其实例化方式，因此它也适合难以配置的对象（再次以记录器为例，以及度量导出器和 API 客户端都是很好的例子）。如果像我们的例子一样“禁用”构造函数，实例化过程将完全封装。

限制应用程序使用单个对象实例在内存占用方面具有性能优势。

单例的主要缺点是其对全局状态的依赖（在我们的例子中，是静态的`loggerInstance`）。测试单例很困难，尤其是在构造函数被“禁用”的情况下（就像我们的例子一样），因为我们的测试总是希望有一个单例实例。

在某种程度上，单例也可以被认为是“全局状态”，这带来了所有缺点。全局状态有时可能是设计不佳的标志，更新/消费全局状态容易出错（例如，如果消费者正在读取状态，但随后状态被更新而没有再次读取）。

## “类单例”模式的改进

在我们的单例日志实现中，从外部修改单例的内部状态是可能的。这并不是我们单例特有的；这是 JavaScript 的本质。默认情况下，它的字段和方法是公开的。

然而，在我们的单例场景中，这是一个更大的问题，因为消费者可以使用诸如`Logger.loggerInstance = null`或`delete Logger.loggerInstance`之类的语句重置`loggerInstance`。请看以下示例：

```js
const logger = Logger.getInstance();
Logger.loggerInstance = null;
const logger = new Logger('info', console); // should throw but creates a new instance
```

为了阻止消费者修改`loggerInstance`静态字段，我们可以将其设为私有字段。JavaScript 中的私有字段是 ECMAScript 2023 规范（第 13 版 ECMAScript）的一部分。

要定义私有字段，我们使用字段名称前的`#`前缀——在这种情况下，`loggerInstance`变为`#loggerInstance`。`isLevelEnabled`方法变为`#isLevelEnabled`，我们还声明`logLevel`和`transport`分别为`#logLevel`和`#transport`：

```js
export class Logger {
  // other static fields are unchanged
  static #loggerInstance = null;
  #logLevel;
  #transport;
  constructor(logLevel = 'info', transport = console) {
    if (Logger.#loggerInstance) {
      throw new TypeError(
        'Logger is not constructable, use getInstance()
          instead'
      );
    }
    this.#logLevel = logLevel;
    this.#transport = transport;
  }
  #isLevelEnabled(targetLevel) {
    // implementation unchanged
  }
  info(message) {
    if (this.#isLevelEnabled('info')) {
      return this.#transport.info(message);
    }
  }
  warn(message) {
    if (this.#isLevelEnabled('warn')) {
      this.#transport.warn(message);
    }
  }
  error(message) {
    if (this.#isLevelEnabled('error')) {
      this.#transport.error(message);
    }
  }
  getInstance() {
    if (!Logger.#loggerInstance) {
      Logger.#loggerInstance = new Logger('warn', console);
    }
    return Logger.#loggerInstance;
  }
}
```

无法删除`loggerInstace`或将其设置为`null`，因为尝试访问`Logger.#loggerInstance`是一个语法错误：

```js
  Logger.#loggerInstance = null;
        ^
SyntaxError: Private field '#loggerInstance' must be
  declared in an enclosing class
```

另一种有用的技术是不允许修改对象上的字段。为了禁止修改，我们可以在实例创建后使用`Object.freeze`将其冻结。

```js
class Logger {
  // no changes to the logger class
}
export default Object.freeze(new Logger('warn', console));
```

现在，当有人尝试更改`Logger`实例的字段时，他们会收到`TypeError`：

```js
import logger from './logger.js';
logger.transport = {}; // new TypeError('Cannot add
  property transport, object is not extensible')
```

我们现在已经重构了单例实现，通过使用私有字段和`Object.freeze`来禁止外部对其进行修改。接下来，我们将看到如何使用**EcmaScript**（**ES**）模块来提供单例功能。

## 不使用类字段且使用 ES 模块行为的单例

JavaScript 模块系统具有以下缓存行为——如果加载了一个模块，任何进一步的导入该模块的导出都将被缓存为导出的实例。

因此，在 JavaScript 中可以如下创建单例。

```js
class MySingleton {
  constructor(value) {
    this.value = value;
  }
}
export default new MySingleton('my-value');
```

多次导入默认导出将只存在一个`MySingleton`对象的实例。此外，如果我们不导出类，那么构造函数不需要是“受保护的”。

```js
import('./my-singleton.js') result in the same object. They both return the same object because the output of the import for a given module is a singleton:
```

```js
await Promise.all([
  import('./my-singleton.js'),
  import('./my-singleton.js'),
]).then(([import1, import2]) => {
  console.assert(
    import1.default.value === 'my-value' &&
      import2.default.value === 'my-value',
    'instance variable is equal'
  );
  console.assert(
    import1.default === import2.default,
    'multiple imports of a module yield the same default
      object value, a single MySingleton instance'
  );
  console.assert(import1 === import2, 'import objects are a
    single reference');
});
```

对于我们的日志记录器，这意味着我们可以在 JavaScript 中实现一个急切初始化的单例，而不需要任何对构造函数或`getInstance`方法的严格保护。注意`logLevel`和`isLevelEnabled`分别作为公共实例属性和公共方法的使用（因为可能需要从消费者那里访问它们）。同时，`#transport`保持私有，我们已经删除了`loggerInstance`和`getInstance`。我们保留了`Object.freeze()`，这意味着尽管`logLevel`可以从消费者那里读取，但它不可修改：

```js
class Logger {
  static logLevels = ['info', 'warn', 'error'];
  #transport;
  constructor(logLevel = 'info', transport = console) {
    this.logLevel = logLevel;
    this.#transport = transport;
  }
  isLevelEnabled(targetLevel) {
    return (
      Logger.logLevels.indexOf(targetLevel) >=
      Logger.logLevels.indexOf(this.logLevel)
    );
  }
  info(message) {
    if (this.isLevelEnabled('info')) {
      return this.#transport.info(message);
    }
  }
  warn(message) {
    if (this.isLevelEnabled('warn')) {
      this.#transport.warn(message);
    }
  }
  error(message) {
    if (this.isLevelEnabled('error')) {
      this.#transport.error(message);
    }
  }
}
export default Object.freeze(new Logger('warn', console));
```

在本章的这一部分，我们学习了如何使用公开`getInstance()`方法的类来实现单例模式，以及单例的急切初始化和懒加载初始化之间的区别。我们介绍了一些 JavaScript 特性，如私有类字段和`Object.freeze`，这些特性在实现单例模式时可能很有用。最后，我们探讨了 JavaScript/ECMAScript 模块具有单例类似的行为，并且可以依赖它们为类实例提供这种行为。

在下一节中，我们将探讨本章涵盖的最后一个创建型设计模式——工厂设计模式。

# JavaScript 中的工厂模式

类似于关于 JavaScript“原型”与原型创建型设计模式的讨论，“工厂”在一般程序设计讨论和设计模式中指的是相关但不同的概念。

在一般编程意义上，“工厂”是一个旨在创建其他对象的实体。这一点从其名称中可以体现出来，该名称指的是一个将物品从一个形状加工成另一个形状（或从一种类型的物品加工成另一种类型的物品）的设施。这种工厂的命名意味着函数或方法的输出是一个新对象。在 JavaScript 中，这意味着像返回一个对象字面量的函数这样简单的东西也是一个工厂函数：

```js
const simpleFactoryFunction = () => ({}); // returns an object, therefore it's a factory.
```

这种工厂的定义很有用，但本章的这一节是关于工厂设计模式的，它确实符合这个整体的“工厂”定义。

工厂或工厂方法设计模式解决了一个类继承问题。一个基类或超类被扩展（扩展的类是一个子类）。基类的角色是为子类实现的方法提供协调，因为我们希望子类控制用哪些其他对象填充实例。

## 实现

一个工厂示例如下。我们有一个实现`generateBuilding()`方法的`Building`基类。目前，它将使用`makeTopFloor`实例方法创建顶层。在基类（`Building`）中，`makeTopFloor`被实现，主要是因为 JavaScript 没有提供定义抽象方法的方式。`makeTopFloor`的实现会抛出错误，因为子类应该重写它；在这种情况下，`makeTopFloor`是“工厂方法”。这是基类如何将对象的实例化推迟到子类的方法：

```js
class Building {
  generateBuilding() {
    this.topFloor = this.makeTopFloor();
  }
  makeTopFloor() {
    throw new Error('not implemented, left for subclasses
      to implement');
  }
}
```

如果我们想要实现一个单层房屋，我们会扩展`Building`并重写`makeTopFloor`；在这种情况下，`topFloor`将具有`level: 1`。

```js
class House extends Building {
  makeTopFloor() {
    return {
      level: 1,
    };
  }
}
```

当我们实例化`House`，它是`Building`的子类时，我们可以访问`generateBuilding`方法；当调用它时，它会正确设置`topFloor`（为`{ level: `1` }`）。

```js
const house = new House();
house.generateBuilding();
console.assert(house.topFloor.level === 1, 'topFloor works
  in House');
```

现在，如果我们想要创建一个具有非常不同顶层的新型建筑，我们仍然可以扩展`Building`；我们只需重写`makeTopFloor`以返回不同的楼层。在摩天大楼的情况下，我们希望顶层非常高，所以我们将会这样做：

```js
class SkyScraper extends Building {
  makeTopFloor() {
    return {
      level: 125,
    };
  }
}
```

定义了我们的`SkyScraper`，它是`Building`的子类后，我们可以实例化它并调用`generateBuilding`。正如前面的`House`案例一样，`generateBuilding`方法将使用`SkyScraper`的`makeTopFloor`方法来填充`topFloor`实例属性：

```js
const skyScraper = new SkyScraper();
skyScraper.generateBuilding();
console.assert(skyScraper.topFloor.level > 100, 'topFloor
  works in SkyScraper');
```

在这种情况下，“工厂方法”是`makeTopFloor`。在基类中，`makeTopFloor`方法是“未实现”的，也就是说，它以一种强制子类在希望使用`generateBuilding`时定义`makeTopFloor`重写的方式实现。

注意，在我们的示例中，`makeTopFloor`返回了对象字面量，如本章前面提到的；这是 JavaScript 中不是所有面向对象语言都有的特性（JavaScript 是多范式的）。我们将在本节后面看到实现工厂模式的不同方法。

## 用例

使用工厂方法的好处是，我们可以在不修改基类的情况下创建各种子类。这是“开/闭原则”在起作用——我们例子中的`Building`类是“开放”的，可以扩展（即可以无限地子类化以创建不同类型的建筑），但“封闭”的，不允许修改（即我们不需要为每个子类在`Building`中进行更改，只有在我们要添加新行为时才需要）。

## 现代 JavaScript 的改进

我们可以用 JavaScript 实现的关键改进是由其对函数的第一级支持以及使用字面量定义对象的能力（而不是类实例化）所启用的。

JavaScript 有“第一级函数”意味着函数就像任何其他类型一样——它们可以作为参数传递，设置为变量值，并从其他函数返回。

这种模式的更自然实现可能涉及一个 `generateBuilding` 独立函数，而不是 `Building` 类。`generateBuilding` 将 `makeTopFloor` 作为参数，或者接受一个包含 `makeTopFloor` 键的对象参数。`generateBuilding` 的输出将是一个使用对象字面量创建的对象，它将 `makeTopFloor()` 的输出设置为 `topFloor` 键的值：

```js
function generateBuilding({ makeTopFloor }) {
  return {
    topFloor: makeTopFloor(),
  };
}
```

为了创建我们的房屋和摩天大楼，我们将使用相关的 `makeTopFloor` 函数调用 `generateBuilding`。在房屋的情况下，我们想要一个位于第 1 层的顶层；在摩天大楼的情况下，我们想要一个位于第 125 层的顶层。

```js
const house = generateBuilding({
  makeTopFloor() {
    return {
      level: 1,
    };
  },
});
console.assert(house.topFloor.level === 1, 'topFloor works
  in house');
const skyScraper = generateBuilding({
  makeTopFloor() {
    return {
      level: 125,
    };
  },
});
console.assert(skyScraper.topFloor.level > 100, 'topFloor works in skyScraper');
```

使用函数直接在 JavaScript 中工作得更好的一个原因是，我们不必实现一个“抛出错误以提醒消费者覆盖我”的 `makeFloor` 方法，就像我们在 `Building` 类中做的那样。

在支持抽象方法的除 JavaScript 之外的语言中，这种模式比在具有一等函数的 JavaScript 中更有用和自然实现。

你还必须记住，JavaScript/ECMAScript 的原始版本并没有包含 `class` 构造。

在本章的最后部分，我们学习了工厂方法模式是什么，以及它与工厂编程概念的对比。然后我们实现了一个基于类的工厂模式场景，以及一个更自然的 JavaScript 版本。在这一节中，我们涵盖了工厂方法模式在 JavaScript 中的用例、优点和缺点。

# 摘要

在本章中，我们讨论了如何使用创建型设计模式在 JavaScript 中构建更可扩展和可维护的系统。

原型设计模式在创建包含相同值的多个对象实例时特别出色。这种设计模式允许我们更改原型的初始值，并影响所有克隆实例。

单例设计模式对于完全隐藏应该只实例化一次的类的初始化细节非常有用。我们看到了 JavaScript 的模块系统如何生成单例，以及如何利用这一点来简化单例实现。

工厂方法设计模式允许基类将一些对象创建的实现推迟到子类。我们看到了哪些特性会使这种模式在 JavaScript 中更有用，以及一个使用工厂函数的替代的 JavaScript 习惯用法。

我们现在可以利用创建型设计模式来构建可组合的类，这些类可以根据需要进化以覆盖不同的用例。

现在我们已经了解了如何使用创建型设计模式高效地创建对象，在下一章中，我们将介绍如何使用结构型设计模式来组织不同对象和类之间的关系。
