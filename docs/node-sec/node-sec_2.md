# 第二章：一般考虑

构建安全的 Node.js 应用程序将需要理解它所构建的许多不同层次。从底层开始，我们有定义 JavaScript 组成的语言规范。接下来，虚拟机执行你的代码，并且可能与规范有所不同。在此之后，Node.js 平台及其 API 在操作上有细节会影响你的应用程序。最后，第三方模块与我们自己的代码交互，并且需要进行安全编程实践的审计。

首先，JavaScript 的官方名称是 ECMAScript。国际**欧洲计算机制造商协会**（**ECMA**）在 1997 年首次将这种语言标准化为**ECMAScript**。这个 ECMA-262 规范定义了 JavaScript 作为一种语言的组成，包括它的特性，甚至一些它的错误。甚至一些它的一般古怪之处在规范中保持不变，以保持向后兼容性。虽然我不会说规范本身是必读的，但我会说它是值得考虑的。

其次，Node.js 使用 Google 的**V8**虚拟机来解释和执行你的源代码。在为浏览器开发时，你需要考虑所有其他虚拟机（更不用说版本了），以及可用的功能。在 Node.js 应用程序中，你的代码只在服务器上运行，因此你有更多的自由，并且可以使用 V8 中可用的所有功能。此外，你还可以专门为 V8 引擎进行优化。

接下来，Node.js 处理设置事件循环，并且它会接受你的代码来注册事件的回调并相应地执行它们。在开发应用程序时，你需要注意 Node.js 对异常和其他错误的响应的一些重要细节。

Node.js 之上是开发者 API。这个 API 主要用 JavaScript 编写，允许你作为 JavaScript 开发者自己阅读它，并理解它的工作原理。有许多提供的模块可能会被你使用，了解它们的工作原理对你来说很重要，这样你就可以进行防御性编码。

最后，npm 提供给你访问的第三方模块数量众多，这可能是一把双刃剑。一方面，你有很多选项可以满足你的需求。另一方面，拥有第三方代码可能是一个潜在的安全责任，因为你需要支持和审计每一个这些模块（以及它们自己的依赖项）以寻找安全漏洞。

# JavaScript 安全

JavaScript 本身最大的安全风险之一，无论是在客户端还是现在在服务器端，就是使用`eval()`函数。这个函数，以及类似它的其他函数，接受一个字符串参数，它可以表示一个表达式、语句或一系列语句，并且会像其他 JavaScript 源代码一样被执行。这在下面的代码中有所展示：

```js
// these variables are available to eval()'d code
// assume these variables are user input from a POST request
var a = req.body.a; // => 1
var b = req.body.b; // => 2
var sum = eval(a + "+" + b); // same as '1 + 2'
```

这段代码可以完全访问当前作用域，甚至可以影响全局对象，给它带来了令人担忧的控制权。让我们看看相同的代码，但想象一下如果有人恶意发送任意的 JavaScript 代码而不是一个简单的数字。结果如下所示：

```js
var a = req.body.a; // => 1
var b = req.body.b; // => 2; console.log("corrupted");
var sum = eval(a + "+" + b); // same as '1 + 2; console.log("corrupted");
```

由于这里`eval()`的滥用，我们正在目睹一次“远程代码执行”攻击！当直接在服务器上执行时，攻击者可能会访问服务器文件和数据库。`eval()`有一些情况下可能会有用，但如果用户输入涉及到任何步骤，那么最好尽量避免使用！

JavaScript 还有其他与`eval()`功能等效的功能，除非绝对必要，否则也应该避免使用。首先是`Function`构造函数，它允许你从字符串创建一个可调用的函数，如下面的代码所示：

```js
// creates a function that returns the sum of 2 arguments
var adder = new Function("a", "b", "return a + b");
adder(1, 2); // => 3
```

虽然与`eval()`函数非常相似，但并非完全相同。这是因为它无法访问当前范围。但是，它仍然可以访问全局对象，并且在涉及用户输入时应避免使用。

如果发现自己处于需要执行涉及用户输入的任意代码的情况下，确实有一个安全选项。Node.js 平台的 API 包括一个旨在让您能够在沙盒中编译和运行代码的**vm**模块，以防止操纵全局对象甚至当前范围。

应该注意，vm 模块存在许多已知问题和边缘情况。您应该阅读文档，并了解您所做的一切可能带来的影响，以确保您不会措手不及。

# ES5 功能

ECMAScript5 对 JavaScript 进行了广泛的更改，包括以下更改：

1.  严格模式用于从语言中删除不安全的功能。

1.  属性描述符可控制对象和属性访问。

1.  更改对象可变性的功能。

## 严格模式

严格模式改变了 JavaScript 代码在某些情况下的运行方式。首先，它会在以前是静默的情况下抛出错误。其次，它会删除和/或更改使 JavaScript 引擎优化变得困难或不可能的功能。最后，它禁止了一些可能出现在未来版本 JavaScript 中的语法。

此外，严格模式仅适用于选择加入，并且可以全局应用或应用于单个函数范围。对于 Node.js 应用程序，要全局启用严格模式，请在执行程序时添加`-use_strict`命令行标志。

### 提示

在处理可能使用严格模式的第三方模块时，这可能会对整个应用程序产生负面影响。话虽如此，您可能会要求第三方模块的审核符合严格模式的要求。

通过在函数开头添加`"use strict"`指示符，可以启用严格模式，在任何其他表达式之前，如下面的代码所示：

```js
function sayHello(name) {
    "use strict"; // enables strict mode for this function scope
    console.log("hello", name);
}
```

在 Node.js 中，所有所需的文件都包装在一个处理`CommonJS`模块 API 的函数表达式中。因此，您可以通过简单地将指令放在文件顶部来为整个文件启用严格模式。这不会像在浏览器等环境中那样全局启用严格模式。

严格模式对语法和运行时行为进行了许多更改，但为了简洁起见，我们只讨论与应用程序安全相关的更改。

首先，在严格模式下，通过`eval()`运行的脚本无法向封闭范围引入新变量。这可以防止在运行`eval()`时泄漏新的可能会与现有变量冲突的变量，如下面的代码所示：

```js
"use strict";
eval("var a = true");
console.log(a); // ReferenceError thrown – a does not exist
```

此外，通过`eval()`运行的代码无法通过其上下文访问全局对象。这与其他函数范围的更改类似，不过稍后将对此进行解释，但对于`eval()`来说，这是特别重要的，因为它不能再使用全局对象执行其他黑魔法。

事实证明，`eval()`函数可以在 JavaScript 中被覆盖。可以通过创建一个名为`eval`的新全局变量，并为其分配其他内容来实现。严格模式禁止了这种操作。它更像是一个语言关键字而不是一个变量，尝试修改它将导致语法错误，如下面的代码所示：

```js
// all of the examples below are syntax errors
"use strict";
eval = 1;
++eval;
var eval;
function eval() { }
```

接下来，函数对象更加安全。ECMAScript 的一些常见扩展为每个函数添加了 `function.caller` 和 `function.arguments` 引用，这些引用在函数调用后出现。实际上，您可以通过遍历这些特殊引用来“遍历”特定函数的调用堆栈。这可能会暴露通常看起来超出范围的信息。严格模式只是在尝试读取或写入这些属性时抛出 `TypeError` 备注，如下面的代码所示：

```js
"use strict";
function restricted() {
    restricted.caller;    // TypeError thrown
    restricted.arguments; // TypeError thrown
}
```

接下来，在严格模式下移除了 `arguments.callee`（例如前面显示的 `function.caller` 和 `function.arguments`）。通常，`arguments.callee` 指的是当前函数，但这个神奇的引用也暴露了一种“遍历”调用堆栈的方式，可能会揭示以前隐藏或超出范围的信息。此外，这个对象使得某些优化对 JavaScript 引擎来说变得困难或不可能。因此，当尝试访问时，它也会抛出 `TypeError` 异常，如下面的代码所示：

```js
"use strict";
function fun() {
    arguments.callee; // TypeError thrown
}
```

最后，使用 `null` 或 `undefined` 作为上下文执行的函数不再将全局对象强制转换为上下文。这适用于之前看到的 `eval()`，但更进一步地阻止了在其他函数调用中对全局对象的任意访问，如下面的代码所示：

```js
"use strict";
(function () {
    console.log(this); // => null
}).call(null);
```

严格模式可以帮助使代码比以前更加安全，但 ECMAScript 5 也通过属性描述符 API 包括了访问控制。JavaScript 引擎一直具有定义属性访问的能力，但 ES5 包括了这些 API，将同样的权力赋予应用程序开发人员。

## 对象属性描述符

对象属性具有以下三个隐藏属性，确定对它们可以进行哪些变化：

+   `writable`：如果这是 `false`，意味着属性值不能被更改（换句话说，只读）

+   `enumerable`：如果这是 `false`，意味着该属性在 for in 循环中不会出现

+   `configurable`：如果这是 `false`，意味着该属性不能被删除

在使用对象字面量或赋值定义对象属性时，这是最常见的方法，这三个隐藏属性的默认值都是 `true`。这使得属性在各个方面完全开放修改。然而，有一些新函数允许应用程序开发人员自行设置这些属性，限制对某些对象属性的访问。属性描述符 API 是完全自选的，即使在 ES5 中，对象属性的默认行为也不会改变。

首先，`Object.defineProperty()` 函数允许您在指定的对象上指定单个属性及其访问器描述符。它接受三个参数：目标对象、新属性的名称和前面提到的描述符对象。访问器描述符只是一个包含指定属性的对象，这些属性对应于前面列出的属性。

### 提示

访问器描述符告诉 JavaScript 引擎，要给我们的新属性赋予的访问级别。在使用 `Object.defineProperty()` 及其相关函数时，重要的是要注意，所有描述符属性值默认设置为 `false`。这与基本赋值相比产生了相反的效果。

```js
var o = {};

// the next 2 statements are completely identical in result

o.a = "A";

Object.defineProperty(o, "a", {
    writable: true,
    enumerable: true,
    configurable: true,
    value: "A"
});
```

这两个语句具有相同的结果，后者更加冗长。然而，传统赋值不能影响任何描述符，与后者不同。让我们看看创建“锁定”属性需要什么：

```js
var o = {};

Object.defineProperty(o, "a", {
    value: "A"
});
```

我们刚刚做的是创建了一个不能被写入、删除或枚举的属性，使其不可变。这允许应用程序开发人员控制数据访问，即使在各种代码边界之间共享对象。

访问器描述符提供的最后一个功能是允许开发人员为特定属性创建 getter 和 setter 函数。getter 是一个在访问属性时返回数据的函数，setter 存储通过赋值发送的数据。以下是示例代码：

```js
var person = {
    firstName: "Dominic",
    lastName: "Barnes"
};

Object.defineProperty(person, "name", {
    enumerable: true,
    get: function () {
        return this.firstName + " " + this.lastName;
    },
    set: function (input) {
        var names = input.split(" ");
        this.firstName = names[0];
        this.lastName = names[1];
    }
});

console.log(person.name); // => "Dominic Barnes"
```

这段代码创建了一个包含来自同一对象上的两个其他属性的数据的属性，并且是动态计算的。在许多情况下，可以使用函数来实现相同的效果，但这样可以更好地分离这两个操作，而不需要在对象本身上使用两个单独的函数。

下一个函数`Object.defineProperties()`类似。然而，这个函数只接受两个参数，宿主对象和另一个对象，该对象是多个属性的哈希，其中属性值都是访问器描述符。以下是示例代码：

```js
var letters = {};

Object.defineProperties(letters, {
    a: {
        enumerable: true,
        value: "A"
    },
    b: {
        enumerable: true,
        value: "B"
    }
});

console.log(letters.a); // => "A"
console.log(letters.b); // => "B"
```

这使我们可以将多个属性定义压缩成一个函数调用，这更多的是为了方便。接下来是其中最强大的函数：`Object.create()`函数。这个函数从头开始创建一个全新的对象，并为其分配一个原型。这反映了 JavaScript 的原型性质，我们不会花时间进一步讨论这一点，因为它与本讨论并不特别相关。

这个函数只接受两个参数，新对象的原型（或`null`以完全不继承）和一个属性对象，就像我们在`Object.defineProperties()`中使用的那样，如下面的代码所示：

```js
var constants = Object.create(null, {
    PI: {
        enumerable: true,
        value: 3.14
    },
    e: {
        enumerable: true,
        value: 2.72
    }
});
```

通过将原型设置为`null`，而不是其他对象，我们创建了一个完全普通的对象，它不继承任何东西，甚至不继承自`Object.prototype`对象。这是可取的，因为即使对`Object.prototype`的修改（这本来就是一个坏主意）也不会对使用这种方法创建的对象产生不利影响。

还有一些其他特殊的函数用于改变对象的可访问性。首先是`Object.preventExtensions()`函数，它防止向指定的对象添加新属性，如下面的代码所示：

```js
var o = {
    a: "A",
    b: "B",
    c: "C"
};

o.d = "D"; // works as expected

Object.preventExtensions(o);

o.e = "E"; // will not work
```

正如你所看到的，这允许你配置一个对象，以便其他人无法在你的对象上创建额外的属性。如果在混合中包括严格模式，最后的赋值将抛出错误，而不是悄无声息地失败。另外，应该注意的是，这个操作一旦发生就无法逆转。

接下来是`Object.seal()`函数，它接受一个对象，并防止属性被删除，除了`Object.preventExtensions()`函数的效果。换句话说，这将获取所有现有属性，并将它们的可配置属性设置为`false`。

```js
var o = {
    a: "A",
    b: "B",
    c: "C"
};

delete o.c; // works as expected

Object.seal(o);

delete o.b; // will not work
```

这很强大，因为我们可以保留对象的结构，但仍然允许属性值发生变化。与之前一样，这个操作是不可逆的。此外，添加严格模式会导致抛出异常，而不是允许操作悄无声息地失败。

最后是其中最强大的`Object.freeze()`函数。这个函数应用了与`Object.seal()`相同的效果，并完全锁定了所有属性。没有值可以被改变（即所有可写属性都设置为`false`），并且属性描述符都是不可修改的。这使得对象实际上是不可变的，并阻止所有其他尝试改变对象的任何操作，如下面的代码所示：

```js
var o = {
    a: "A",
    b: "B",
    c: "C"
};

// works as expected
o.a = 1;
delete o.c;

Object.freeze(o);

// will not work
o.a = "A";
delete o.b;
```

冻结对象与其他操作一样，是不可逆转的。在严格模式下，任何尝试写入或更改对象的操作都会引发错误。

# 静态程序分析

跟踪我们在这里讨论的所有事情可能会让人不知所措。当一个团队的人在同一个项目上工作时，问题会变得更加复杂。执行静态分析的工具会获取你的源代码（而不是执行它），并检查你可以配置的特定代码模式。

例如，你可以配置**JSHint**禁止使用`eval()`并要求所有函数使用严格模式。通过让它检查你的源代码，当违反这些规则时，它会提醒你。这可以与版本控制结合使用，以防止不安全的代码被添加到项目的代码库中。此外，它也可以在发布之前使用，以确保所有代码在进入生产环境之前都是安全的。

JSHint 是**JSLint**项目的社区驱动分支。JSLint 持有主观意见，不像许多人所期望的那样可配置，因此创建了 JSHint 来填补这一空白。两者都是很好的工具，我强烈建议你为你的 JS 项目采用其中之一。虽然静态分析不会捕捉一切，但它将通过自动化帮助确保代码的更高质量。

# Node.js 的注意事项

JavaScript 语言内置了异常作为错误处理的构造。当抛出异常时，需要一些代码来检测错误并适当处理。然而，如果异常未被捕获，它将触发一个致命的错误。

在浏览器中，未捕获的异常会立即停止任何执行。这不会导致网页崩溃，但有可能使应用程序处于不稳定的状态。

在 Node.js 中，未捕获的异常将终止应用程序线程。这与其他服务器端编程语言（如 PHP）非常不同，那里类似的错误只会导致单个请求失败。现在，你必须应对整个服务器和应用程序被突然停止的情况。

## 回调错误

你可以采取的第一步是确保以一种预期和可预测的方式抛出错误，以便以后能够有效地捕获。在 Node.js 中，使用回调进行异步操作的惯例是将一个`Error`对象作为第一个参数发送给回调函数。这是 Node.js 核心使用的标准惯例，并且已被社区广泛采用。

```js
var fs = require("fs");

fs.readFile("/some/file", "utf8", function (err, contents) {
    // err will be...
    // null if no error has occurred … or
    // an Error object with information about the error
});
```

上述代码只是将一个文件读取为字符串。这个操作有一个回调，接受两个参数。第一个是一个`Error`对象，但只有在这个 I/O 操作期间发生错误时才会有，比如文件不存在。通过简单地将错误对象作为函数参数传递，这在技术上并不会"抛出"异常。你的应用程序仍然应该处理这些错误，如果可能的话进行纠正。如果发生意外错误，或者无法直接纠正，你应该自己抛出错误，而不是悄悄地吞噬错误，为自己以后创建难以调试的场景。

## EventEmitter 错误处理

Node.js 核心有一个广泛使用的实用对象叫做`EventEmitter`。这是一个可以实例化或继承的对象，允许绑定和发出异步操作的事件。当`EventEmitter`对象遇到错误时，惯例是使用`Error`对象作为参数发出一个错误事件。

```js
var http = require("http");

http.get("http://nodejs.org/", function (res) {
    // res is an EventEmitter that represents the HTTP response

    res.on("data", function (chunk) {
        // this event occurs many times
        // each with a small chunk of the response data
    });

    res.on("error", function (err) {
        // this event occurs if an error occurs during transmission
    });
});
```

上述代码只是向[`nodejs.org/`](http://nodejs.org/)发出一个 HTTP 请求。结果对象是一个代表 HTTP 响应的`EventEmitter`对象。它会发出多个数据事件，当从服务器接收数据时，如果传输过程中发生错误（类似于网络断开连接），则会发出一个`error`事件。

应该注意，`EventEmitter`对象在处理`error`事件时有非常特定的行为。如果你有一个`EventEmitter`对象发出了一个`error`事件，但没有附加的监听器来响应这个事件，那么相应的`Error`对象会被抛出，并且很可能成为一个未捕获的异常。这意味着任何未处理的错误事件都会导致应用程序崩溃，所以在使用`EventEmitter`对象时，始终要绑定一个`error`事件处理程序。

## 未捕获的异常

当发生未捕获的异常时，Node.js 将打印当前堆栈跟踪，然后终止线程。所有 Node.js 应用程序都可以使用一个名为`process`的全局对象。它是一个带有特殊事件`"uncaughtException"`的`EventEmitter`对象，当未捕获的异常被带到主事件循环时会被触发。通过绑定到此事件，您可以设置自定义行为，例如发送电子邮件或写入特殊的日志文件。以下代码中可以看到这一点：

```js
process.on("uncaughtException", function (err) {
    // we're just executing the default behavior
    // but you can implement your own custom logic here instead
    console.error(err);
    console.trace();
    process.exit();
});
```

在前面的代码中，我只是简单地做了 Node.js 默认的事情。如我之前提到的，您可以实现自己的错误记录程序。如果您使用自定义处理程序，需要确保通过`process.exit()`函数自行终止进程。

虽然在发生未捕获的异常后继续应用是可能的，但不建议这样做！根据定义，未捕获的异常中断了应用程序的正常流程，使其处于不稳定和不可靠的状态。如果您简单地忽略错误并继续处理，那么您就会陷入危险的境地。Node.js 文档将此视为拔掉计算机的电源来关闭它。您可能可以做几次，但如果这种情况不断重复，系统将变得越来越不稳定和不可预测。

## 域

虽然`uncaughtException`事件允许我们处理错误，但它仍然相当粗糙。您会失去错误来源的大部分原始上下文，这使得以后调试变得更加困难。从 Node.js v0.8 开始，有一种新的错误处理机制可用，称为**域**。它们是一种将不同的 I/O 操作组合在一起的方式，以便在发生错误时，通过`uncaughtException`事件通知域对象而不是进程对象。这允许您保留错误本身的上下文，并帮助您为将来准备和纠正错误。

除了保留上下文，域还允许您在发生错误时优雅地关闭相关服务。如果您运行着一个 HTTP 服务器，并且一个用户发生了错误，简单地关闭服务器将立即中断当前正在同时使用服务器的其他用户。这对这些用户是不公平的，因此我们需要能够更优雅地关闭我们的服务器。我们应该停止服务器接受新连接，并在关闭服务器之前让当前请求得到满足。

```js
var http = require("http"),
    domain = require("domain"),
    server = http.createServer(),
    counter = 0;
server.on("request", function (req, res) {
    // this domain will cover this entire request/response cycle
    var d = domain.create();
    d.on("error", function (err) {
        // outputs all relevant context for this error
        console.error("Error:", err);

        res.writeHead(500, { "content-type": "text/plain" });
        res.end(err.message);

        // stops the server from accepting new connections/requests
        console.warn("closing server to new connections");
        server.close(function () {
            console.warn("terminating process");
            process.exit(1);
        });
    });

    // adding the req and res objects to the domain allows
    // errors they encounter to be handled by the domain
    // automatically
    d.add(req);
    d.add(res);

    d.run(function () {
        if (++counter === 4) {
            throw new Error("Unexpected Error");
        }

        res.writeHead(200, { "content-type": "text/plain" });
        res.end("Hello World\n");
    });
});

server.listen(3000);
```

前面的代码设置了一个简单的 HTTP 服务器，在发生错误之前会响应四次。对于每个请求，都会创建一个域，可以将其传递给请求处理程序的各个部分，并且可以在域的上下文中运行任何异步操作。在第 4 个请求时，我们将抛出一个`Error`对象。域有一个错误事件处理程序，它输出错误信息、堆栈跟踪，然后继续关闭服务器。首先，它发送当前请求一个错误消息，然后停止接受新请求，并完成服务其队列中的所有当前请求。完成后，进程本身被终止。

从技术上讲，我们可以使用`uncaughtException`事件来实现我在这里演示的内容。但是，如果您在应用程序中并行运行多个服务器（例如，一个 HTTP 服务器和一个 WebSocket 服务器），或者使用集群模块运行多个进程，那么该事件处理程序不一定会给您处理特定于遇到错误的服务器的上下文。事实上，您甚至无法区分`uncaughtException`事件中的不同请求，因为该上下文也会丢失。使用域，您可以更优雅地处理错误，而不会丢失上下文。

Node.js 有一个名为**cluster**的模块，它允许你利用多核环境。它通过生成多个工作进程来实现这一点，这些工作进程共享相同的服务器端口，而`cluster`模块会为你处理这些进程之间的消息传递。如果其中一个工作进程出现错误，域将允许你轻松关闭只有单个服务器和工作进程，同时让其他工作进程继续正常运行。一旦该进程完成清理并退出，你可以生成一个全新的进程来取代它，你的应用程序将因此经历零停机时间。

## 进程监视

说到这一点，事情可能会出错。你不应该忽略未捕获的异常，因为你的应用程序会变得不稳定，并且会泄漏引用和内存。处理未捕获异常的唯一安全方式是停止该进程。这意味着你的服务器将无法提供给其他用户使用。这意味着，如果一个恶意用户能够找到一种方法在你的服务器上触发未捕获的异常，他们实际上正在对其他用户发起拒绝服务攻击。

解决方案是拥有一个可以监视你的应用程序进程并在停止时自动重新启动它的进程监视器。有很多选择，包括一些特定于平台的选项。一些可用的进程监视器包括 forever、mon 和 upstart。关键是你应该实现某种进程监视，这样当出现问题时就不必手动重新启动你的应用程序。

一旦你有了一个进程监视器，一定要配置它将错误记录在某个地方，这样你就可以跟踪，以便纠正应用程序中的有害和致命错误。监视你的应用程序崩溃频率，并尽快纠正错误也是明智的。

# npm 模块（第三方代码）

正如之前提到的，Node.js 最大的特点之一是其充满活力的社区和快速增长的模块注册表。因为 Node.js 核心 API 故意保持小而集中，你可能会整合其他模块，这样你就不必从头开始编写很多东西。

就像你会努力审查你的代码以确保安全实践一样，你也应该积极参与监控你在项目中包含的 npm 模块。npm 上有许多完全开源的项目，通常可以在 GitHub 或其他类似的在线资源上找到。这使得手动查看源代码以寻找突出问题变得很容易。作为最后的手段，你可以检查 npm 在安装依赖时下载的本地包，尽管不能保证获得包的开发环境中的所有内容。

在选择要采用的模块时，寻找包含某种测试套件的模块。如果它们有运行测试，那么你就更容易确定功能是否按设计工作。其次，寻找那些包含一些静态分析的项目，这通常以 JSHint 或 JSLint 的形式出现。查看它们的样式指南或静态分析配置，了解它们遵守的规则。许多这类项目都有某种构建过程，其中可能包括运行自动化测试、静态分析和其他相关工具的方法。

Node.js 开发人员在他们的模块中注重的一个重点是使它们小巧、高度集中和可组合（即它们很容易与其他模块互操作）。因此，它们通常在代码行数和复杂性方面非常小，这使得编写安全和可测试的代码变得更加容易。这在涉及应用程序安全时对 Node.js 平台非常有利。

有一个正在崛起的项目叫做**Node Security Project**，可以在[`nodesecurity.io/`](http://nodesecurity.io/)找到。他们的目标是审计每一个 npm 模块，以查找安全漏洞。他们需要 Node.js 开发人员和安全研究人员来帮助他们，因为他们面临着一项艰巨的任务。如果你已经对保护自己的应用程序感兴趣，你可以将你用来审计你最终使用的模块的时间贡献给这个团队的注册表。这是实现你自己目标的一个很好的方式，同时也为整个 Node.js 社区做出贡献。

# 总结

在本章中，我们研究了适用于 JavaScript 语言本身的安全功能，包括如何使用静态代码分析来检查前面提到的许多问题。此外，我们还研究了 Node.js 应用程序的一些内部工作原理，以及在安全性方面与典型的浏览器开发有何不同。最后，我们简要讨论了 npm 模块生态系统和 Node Security Project，该项目旨在为安全目的审计每一个模块。在下一章中，我们将讨论应用程序的安全考虑。
