# 第六章：生成函数 - 高阶函数

在第五章中，*声明式编程 - 更好的风格*，我们使用了一些预定义的高阶函数，并且能够看到它们的使用方式让我们编写了声明式的代码，不仅在可理解性上有所提升，而且在紧凑性上也有所提升。在这一新章节中，我们将进一步探讨高阶函数的方向，并且我们将开发我们自己的高阶函数。我们可以将我们要进入的函数类型大致分类为三组：

+   **包装函数**，保持其原始功能，添加某种新功能。在这一组中，我们可以考虑*日志记录*（为任何函数添加日志记录功能）、*计时*（为给定函数生成时间和性能数据）和*记忆化*（缓存结果以避免未来的重新计算）。

+   **修改函数**，在某些关键点上与它们的原始版本不同。在这里，我们可以包括`once()`函数（我们在第二章中编写过，*函数式思维 - 第一个示例*），它改变了原始函数只运行一次，像`not()`或`invert()`这样改变函数返回值的函数，以及产生具有固定参数数量的新函数的 arity 相关转换。

+   **其他产物**，提供新的操作，将函数转换为 promises，提供增强的搜索功能，或允许将方法与对象解耦，以便我们可以在其他上下文中使用它们，就像它们是普通函数一样。

# 包装函数

在这一部分，让我们考虑一些提供对其他函数进行*包装*以某种方式增强其功能，但不改变其原始目的的高阶函数。在*设计模式*方面（我们将在第十一章中重新讨论），我们也可以谈论*装饰器*。这种模式基于向对象（在我们的情况下是函数）添加一些行为而不影响其他对象的概念。装饰器这个术语也很受欢迎，因为它在 Angular 等框架中的使用，或者（在实验模式下）用于 JS 的一般编程。

装饰器正在考虑在 JS 中进行一般采用，但目前（2017 年 8 月）处于 2 阶段，*草案*级别，可能要等一段时间才能进入 3 阶段（*候选*）和最终进入 4 阶段（*完成*，意味着正式采用）。你可以在[`tc39.github.io/proposal-decorators/`](https://tc39.github.io/proposal-decorators/)了解更多关于 JS 装饰器的信息，以及 JS 采用过程本身，称为 TC39，在[`tc39.github.io/process-document/`](https://tc39.github.io/process-document/)。在第十一章，*实现设计模式 - 函数式方法*的*问题*部分中查看更多信息。

至于*包装器*这个术语，它比你想象的更重要和普遍；事实上，JavaScript 广泛使用它。在哪里？你已经知道对象属性和方法是通过点表示法访问的。然而，你也知道你可以编写诸如`myString.length`或`22.9.toPrecision(5)`的代码--这些属性和方法是从哪里来的，因为字符串和数字都不是对象？JavaScript 实际上在你的原始值周围创建了一个*包装对象*。这个对象继承了适用于包装值的所有方法。一旦需要进行评估，JavaScript 就会丢弃刚刚创建的包装器。我们无法对这些瞬时包装器做任何事情，但有一个概念我们将会回来：包装器允许在不适当类型的东西上调用方法--这是一个有趣的想法；参见第十二章，*构建更好的容器 - 函数式数据类型*，了解更多应用。

# 日志

让我们从一个常见的问题开始。在调试代码时，通常需要添加某种日志信息，以查看函数是否被调用，使用了什么参数，返回了什么，等等。（是的，当然，您可以简单地使用调试器并设置断点，但请在这个例子中忍耐一下！）正常工作意味着您将不得不修改函数本身的代码，无论是在进入还是退出时。您将不得不编写如下的代码：

```js
function someFunction(param1, param2, param3) {
 // *do something*
 // *do something else*
 // *and a bit more,*
 // *and finally*
 return *some expression*;
}
```

到这样的程度：

```js
function someFunction(param1, param2, param3) {
 console.log("entering someFunction: ", param1, param2, param3);
 // *do something*
 // *do something else*
 // *and a bit more,*
 // *and finally*
 let auxValue = *some expression*;
 console.log("exiting someFunction: ", auxValue);
 return auxValue;
}
```

如果函数可以在多个地方返回，您将不得不修改所有的`return`语句，以记录要返回的值。当然，如果您只是在动态计算返回表达式，您将需要一个辅助变量来捕获该值。

# 以一种功能性的方式记录

这样做并不困难，但修改代码总是危险的，容易发生“意外”。因此，让我们戴上我们的 FP 帽子，想出一种新的方法来做这件事。我们有一个执行某种工作的函数，我们想知道它接收到的参数和它返回的值。

我们可以编写一个高阶函数，它将有一个参数，即原始函数，并返回一个新的函数，该函数将执行以下操作：

1.  记录接收到的参数。

1.  调用原始函数，捕获其返回的值。

1.  记录该值；最后。

1.  返回给调用者。

一个可能的解决方案如下：

```js
const addLogging = fn => (...args) => {
    console.log(`entering ${fn.name}: ${args})`);
 const valueToReturn = fn(...args);
    console.log(`exiting ${fn.name}: ${valueToReturn}`);
 return valueToReturn;
};
```

由`addLogging()`返回的函数的行为如下：

+   第一个`console.log()`行显示了原始函数的名称及其参数列表

+   然后调用原始函数`fn()`，并存储返回的值

+   第二个`console.log()`行显示函数名称（再次）及其返回值

+   最后，`fn()`计算的值被返回

如果您为 Node.js 应用程序执行此操作，您可能会选择更好的日志记录方式，比如使用 Winston、Morgan 或 Bunyan 等库--但我们的重点是展示如何包装原始函数，使用这些库所需的更改将很小。

例如，我们可以将其与即将到来的函数一起使用--我同意，以一种过于复杂的方式编写，只是为了有一个合适的例子！

```js
function subtract(a, b) {
 b = changeSign(b);
 return a + b;
}

function changeSign(a) {
 return -a;
}

subtract = addLogging(subtract);
changeSign = addLogging(changeSign);
let x = subtract(7, 5);
```

执行最后一行的结果将产生以下日志行：

```js
entering subtract: 7 5
entering changeSign: 5
exiting changeSign: -5
exiting subtract: 2
```

我们在代码中所做的所有更改都是重新分配`subtract()`和`changeSign()`，这实质上替换了它们的新的生成日志的包装版本。对这两个函数的任何调用都将产生此输出。

我们将会看到一个可能的错误，因为在下一节的*Memoizing*中没有重新分配包装的日志函数。

# 考虑异常情况

让我们稍微增强我们的日志函数，考虑到需要的调整。如果函数抛出错误，您的日志会发生什么？幸运的是，这很容易解决。我们只需要添加一些代码：

```js
const addLogging2 = fn => (...args) => {
 console.log(`entering ${fn.name}: ${args}`);
 try {
 const valueToReturn = fn(...args);
 console.log(`exiting ${fn.name}: ${valueToReturn}`);
 return valueToReturn;
 } catch (thrownError) {
        console.log(`exiting ${fn.name}: threw ${thrownError}`);
 throw thrownError;
 }
};
```

其他更改将由您决定--添加日期和时间数据，增强参数列表的方式等。然而，我们的实现仍然存在一个重要的缺陷；让我们改进一下。

# 以更纯粹的方式工作

当我们编写了`addLogging()`前面的函数时，我们放弃了第四章中看到的一些原则，*行为得体 - 纯函数*，因为我们在代码中包含了一个不纯的元素（`console.log()`）。这样做，我们不仅失去了灵活性（您能够选择替代的日志方式吗？），而且还使我们的测试变得更加复杂。当然，我们可以通过监听`console.log()`方法来测试它，但这并不是很干净：我们依赖于了解我们想要测试的函数的内部，而不是进行纯粹的黑盒测试：

```js
describe("a logging function", function() {
 it("should log twice with well behaved functions", () => {
 let something = (a, b) => `result=${a}:${b}`;
 something = addLogging(something);

 spyOn(window.console, "log");
 something(22, 9);
 expect(window.console.log).toHaveBeenCalledTimes(2);
 expect(window.console.log).toHaveBeenCalledWith(
 "entering something: 22,9"
 );
 expect(window.console.log).toHaveBeenCalledWith(
 "exiting something: result=22:9"
 );
 });

 it("should report a thrown exception", () => {
 let thrower = (a, b, c) => {
 throw "CRASH!";
 };
 spyOn(window.console, "log");
 expect(thrower).toThrow();

 thrower = addLogging(thrower);
 try {
 thrower(1, 2, 3);
 } catch (e) {
 expect(window.console.log).toHaveBeenCalledTimes(2);
 expect(window.console.log).toHaveBeenCalledWith(
 "entering thrower: 1,2,3"
 );
 expect(window.console.log).toHaveBeenCalledWith(
 "exiting thrower: threw CRASH!"
 );
 }
 });
});
```

运行这个测试表明`addLogging()`的行为符合预期，所以这是一个解决方案。

即使这样，以这种方式测试我们的函数并不能解决我们提到的灵活性不足。我们应该注意我们在*注入不纯函数*部分写的内容：日志函数应该作为参数传递给包装函数，这样我们就可以在需要时更改它：

```js
const addLogging3 = (fn, logger = console.log) => (...args) => {
    logger(`entering ${fn.name}: ${args}`);
 try {
 const valueToReturn = fn(...args);
        logger(`exiting ${fn.name}: ${valueToReturn}`);
 return valueToReturn;
 } catch (thrownError) {
        logger(`exiting ${fn.name}: threw ${thrownError}`);
 throw thrownError;
 }
};
```

如果我们什么都不做，日志包装器显然会产生与前一节相同的结果。然而，我们可以提供一个不同的记录器——例如，在 Node.js 中，我们可以使用*winston*，结果会相应地有所不同：

有关*winston*日志工具的更多信息，请参见[`github.com/winstonjs/winston`](https://github.com/winstonjs/winston)。

```js
const winston = require("winston");
const myLogger = **t => winston.log("debug", "Logging by winston: %s", t)**;
winston.level = "debug";

subtract = addLogging3(subtract, myLogger);
changeSign = addLogging3(changeSign, myLogger);
let x = subtract(7, 5);

// *debug: Logging by winston: entering subtract: 7,5*
// *debug: Logging by winston: entering changeSign: 5*
// *debug: Logging by winston: exiting changeSign: -5*
// *debug: Logging by winston: exiting subtract: 2*
```

现在我们已经遵循了我们之前的建议，我们可以利用存根。测试代码几乎与以前相同，但我们使用了一个没有提供功能或副作用的存根`dummy.logger()`，所以在各方面都更安全。确实：在这种情况下，最初被调用的真实函数`console.log()`不会造成任何伤害，但并非总是如此，因此建议使用存根：

```js
describe("after addLogging2()", function() {
 let dummy;

 beforeEach(() => {
 dummy = {logger() {}};
 spyOn(dummy, "logger");
 });

 it("should call the provided logger", () => {
 let something = (a, b) => `result=${a}:${b}`;
 something = addLogging2(something, dummy.logger);

 something(22, 9);
 expect(dummy.logger).toHaveBeenCalledTimes(2);
 expect(dummy.logger).toHaveBeenCalledWith(
 "entering something: 22,9"
 );
 expect(dummy.logger).toHaveBeenCalledWith(
 "exiting something: result=22:9"
 );
 });

 it("a throwing function should be reported", () => {
 let thrower = (a, b, c) => {
 throw "CRASH!";
 };
 thrower = addLogging2(thrower, dummy.logger);

 try {
 thrower(1, 2, 3);
 } catch (e) {
 expect(dummy.logger).toHaveBeenCalledTimes(2);
 expect(dummy.logger).toHaveBeenCalledWith(
 "entering thrower: 1,2,3"
 );
 expect(dummy.logger).toHaveBeenCalledWith(
 "exiting thrower: threw CRASH!"
 );
 }
 });
});
```

在应用 FP 技术时，一定要记住，如果你在某种程度上使自己的工作复杂化——例如，使测试任何一个函数变得困难——那么你一定是在做错事。在我们的案例中，`addLogging()`的输出是一个不纯的函数，这一事实本应引起警惕。当然，鉴于代码的简单性，在这种特殊情况下，你可能会决定不值得修复，你可以不测试，你也不需要能够更改日志生成的方式。然而，长期的软件开发经验表明，迟早你会后悔这样的决定，所以尽量选择更清洁的解决方案。

# 时间

包装函数的另一个可能的应用是以完全透明的方式记录和记录每个函数调用的时间。

如果你计划优化你的代码，请记住以下规则：*不要这样做*，然后*还不要这样做*，最后*不要在没有测量的情况下这样做*。经常提到，很多糟糕的代码都是由早期的优化尝试产生的，所以不要试图写出最佳的代码，不要试图优化，直到你意识到需要优化，不要随意地进行优化，而是通过测量应用程序的所有部分来确定减速的原因。

在前面的例子的基础上，我们可以编写一个`addTiming()`函数，给定任何函数，它将生成一个包装版本，该版本将在控制台上写出时间数据，但在其他方面的工作方式完全相同：

```js
const myPut = (text, name, tStart, tEnd) =>
 console.log(`${name} - ${text} ${tEnd - tStart} ms`);

const myGet = () => performance.now();

const addTiming = (fn, getTime = myGet, output = myPut) => (...args) => {
 let tStart = getTime();
 try {
 const valueToReturn = fn(...args);
        output("normal exit", fn.name, tStart, getTime());
 return valueToReturn;
 } catch (thrownError) {
        output("exception thrown", fn.name, tStart, getTime());
 throw thrownError;
 }
};
```

请注意，与我们在前一节对日志函数应用的增强相一致，我们提供了单独的记录器和时间访问函数。编写我们的`addTiming()`函数的测试应该很容易，因为我们可以注入两个不纯函数。

使用`performance.now()`提供了最高的精度。如果你不需要这个函数提供的精度（它可能是过度的），你可以简单地用`Date.now()`替代。有关这些替代方案的更多信息，请参见[`developer.mozilla.org/en-US/docs/Web/API/Performance/now`](https://developer.mozilla.org/en-US/docs/Web/API/Performance/now)和[`developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Date/now`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Date/now)。你也可以考虑使用`console.time()`和`console.timeEnd()`；请参见[`developer.mozilla.org/en-US/docs/Web/API/Console/time`](https://developer.mozilla.org/en-US/docs/Web/API/Console/time)。

为了能够充分尝试日志功能，我修改了`subtract()`函数，这样如果你尝试减去零，它会抛出一个错误。如果需要，你也可以列出输入参数，以获取更多信息：

```js
subtract = **addTiming(subtract)**;

let x = subtract(7, 5);
// subtract - normal exit 0.10500000000001819 ms

let y = subtract(4, 0);
// subtract - exception thrown 0.0949999999999136 ms
```

这段代码与之前的`addLogging()`函数非常相似，这是合理的--在这两种情况下，我们都在实际函数调用之前添加了一些代码，然后在函数返回后添加了一些新代码。您甚至可以考虑编写一个*更高级*的高阶函数，它将接收三个函数，并且会产生一个高阶函数作为输出（例如`addLogging()`或`addTiming()`），该函数将在开始时调用第一个函数，然后在包装函数返回值时调用第二个函数，或者在抛出错误时调用第三个函数！怎么样？

# 记忆化

在第四章中，*行为良好-纯函数*，我们考虑了斐波那契函数的情况，并看到了如何通过手工将其转换为更高效的版本，通过*记忆化*：缓存计算的值，以避免重新计算。为简单起见，现在让我们只考虑具有单个非结构化参数的函数，并留待以后处理具有更复杂参数（对象、数组）或多个参数的函数。

我们可以轻松处理的值的类型是 JS 的原始值：不是对象且没有方法的数据。JS 有六种原始值：`boolean`、`null`、`number`、`string`、`symbol`和`undefined`。很可能我们只会看到前四个作为实际参数。在[`developer.mozilla.org/en-US/docs/Glossary/Primitive`](https://developer.mozilla.org/en-US/docs/Glossary/Primitive)中了解更多。

# 简单的记忆化

我们将使用我们提到的斐波那契函数，这是一个简单的情况：它接收一个数字参数。我们看到的函数如下：

```js
function fib(n) {
 if (n == 0) {
 return 0;
 } else if (n == 1) {
 return 1;
 } else {
 return fib(n - 2) + fib(n - 1);
 }
}
```

我们在那里做的解决方案在概念上是通用的，但在实现上特别是：我们必须直接修改函数的代码，以便利用所述的记忆化。现在我们应该研究一种自动执行相同方式的方法，就像对其他包装函数一样。解决方案将是一个`memoize()`函数，它包装任何其他函数，以应用记忆化：

```js
const memoize = fn => {
 let cache = {};
 return x => (x in cache ? cache[x] : (cache[x] = fn(x)));
};
```

这是如何工作的？对于任何给定的参数，返回的函数首先检查参数是否已经接收到；也就是说，它是否可以在缓存对象中找到。如果是这样，就不需要计算，直接返回缓存的值。否则，我们计算缺失的值并将其存储在缓存中。（我们使用闭包来隐藏缓存，防止外部访问。）我们在这里假设记忆化函数只接收一个参数（`x`），并且它是一个原始值，然后可以直接用作缓存对象的键值；我们以后会考虑其他情况。

这个方法有效吗？我们需要计时--我们碰巧有一个有用的`addTiming()`函数来做这个！首先，我们对原始的`fib()`函数进行一些计时。我们想要计时完整的计算过程，而不是每个递归调用，所以我们编写了一个辅助的`testFib()`函数，这是我们将计时的函数。我们应该重复计时操作并取平均值，但是，由于我们只是想确认记忆化是否有效，我们将容忍差异：

```js
const testFib = n => fib(n);
addTiming(testFib)(45); // 15,382.255 ms
addTiming(testFib)(40); //  1,600.600 ms
addTiming(testFib)(35); //    146.900 ms
```

当然，您的时间可能会有所不同，但结果似乎是合乎逻辑的：我们在第四章中提到的指数增长似乎是存在的，时间增长迅速。现在，让我们对`fib()`进行记忆化，我们应该得到更短的时间--或者不应该吗？

```js
const testMemoFib = memoize(n => fib(n));
addTiming(testMemoFib)(45); // 15,537.575 ms
addTiming(testMemoFib)(45); //      0.005 ms... *good!*
addTiming(testMemoFib)(40); //  1,368.880 ms... *recalculating?*
addTiming(testMemoFib)(35); //    123.970 ms... *here too?*
```

出了些问题！时间应该下降了——但它们几乎一样。这是因为一个常见的错误，我甚至在一些文章和网页中看到过。我们正在计时`memofib()`——但除了计时之外，没有人调用那个函数，而且那只会发生一次！在内部，所有的递归调用都是`fib()`，它没有被记忆化。如果我们再次调用`testMemoFib(45)`，*那个*调用会被缓存，它会几乎立即返回，但这种优化不适用于内部的`fib()`调用。这也是为什么`testMemoFib(40)`和`testMemoFib(35)`的调用没有被优化的原因——当我们计算`testMemoFib(45)`时，那是唯一被缓存的值。

正确的解决方案如下：

```js
fib = memoize(fib);
addTiming(testFib)(45); // 0.080 ms
addTiming(testFib)(40); // 0.025 ms
addTiming(testFib)(35); // 0.009 ms
```

现在，当计算`fib(45)`时，实际上所有中间的斐波那契值（从`fib(0)`到`fib(45)`本身）都被存储了，所以即将到来的调用几乎没有什么工作要做。

# 更复杂的记忆化

如果我们必须处理接收两个或更多参数的函数，或者可以接收数组或对象作为参数的函数，我们该怎么办？当然，就像我们在第二章中看到的问题一样，*函数式思维 - 第一个例子*，关于让函数只执行一次，我们可以简单地忽略这个问题：如果要进行记忆化的函数是一元的，我们就进行记忆化；否则，如果函数的 arity 不同，我们就什么都不做！

函数的参数个数称为函数的*arity*，或者它的*valence*。你可以用三种不同的方式来说：你可以说一个函数的 arity 是 1、2、3 等，或者你可以说一个函数是一元的、二元的、三元的等，或者你也可以说它是单元的、二元的、三元的等：随你挑！

```js
const memoize2 = fn => {
 if (fn.length === 1) {
 let cache = {};
 return x => (x in cache ? cache[x] : (cache[x] = fn(x)));
 } else {
        return fn;
 }
};
```

更认真地工作，如果我们想要能够记忆化任何函数，我们必须找到一种生成缓存键的方法。为此，我们必须找到一种将任何类型的参数转换为字符串的方法。我们不能直接使用非原始值作为缓存键。我们可以尝试将值转换为字符串，比如`strX = String(x)`，但会遇到问题。对于数组，似乎可以工作，但看看这三种情况：

```js
var a = [1, 5, 3, 8, 7, 4, 6];
String(a); // "1,5,3,8,7,4,6"

var b = [[1, 5], [3, 8, 7, 4, 6]];
String(b); // "1,5,3,8,7,4,6"

var c = [[1, 5, 3], [8, 7, 4, 6]];
String(c); // "1,5,3,8,7,4,6"
```

这三种情况产生相同的结果。如果我们只考虑单个数组参数，我们可能能够应付，但当不同的数组产生相同的键时，那就是个问题。

如果我们必须接收对象作为参数，情况会变得更糟，因为任何对象的`String()`表示都是`"[object Object]"`：

```js
var d = {a: "fk"};
String(d); // "[object Object]"

var e = [{p: 1, q: 3}, {p: 2, q: 6}];
String(e); // "[object Object],[object Object]"
```

最简单的解决方案是使用`JSON.stringify()`将我们收到的任何参数转换为有用的、不同的字符串：

```js
var a = [1, 5, 3, 8, 7, 4, 6];
JSON.stringify(a); // "[1,5,3,8,7,4,6]"

var b = [[1, 5], [3, 8, 7, 4, 6]];
JSON.stringify(b); // "[[1,5],[3,8,7,4,6]]"

var c = [[1, 5, 3], [8, 7, 4, 6]];
JSON.stringify(c); // "[[1,5,3],[8,7,4,6]]"

var d = {a: "fk"};
JSON.stringify(d); // "{"a":"fk"}"

var e = [{p: 1, q: 3}, {p: 2, q: 6}];
JSON.stringify(e); // "[{"p":1,"q":3},{"p":2,"q":6}]"
```

为了性能，我们的逻辑应该是这样的：如果我们要进行记忆化的函数接收一个单一的原始值作为参数，直接使用该参数作为缓存键；在其他情况下，使用`JSON.stringify()`应用于参数数组的结果作为缓存键。我们增强的记忆化高阶函数可以如下：

```js
const memoize3 = fn => {
 let cache = {};
 const PRIMITIVES = ["number", "string", "boolean"];
 return (...args) => {
 let strX =
 args.length === 1 && PRIMITIVES.includes(typeof args[0])
 ? args[0]
 : JSON.stringify(args);
 return strX in cache ? cache[strX] : (cache[strX] = fn(...args));
 };
};
```

就普遍性而言，这是最安全的版本。如果你确定要处理的函数的参数类型，可以说我们的第一个版本更快。另一方面，如果你想要更容易理解的代码，即使牺牲一些 CPU 周期，你可以选择一个更简单的版本：

```js
const memoize4 = fn => {
 let cache = {};
 return (...args) => {
 let strX = JSON.stringify(args);
 return strX in cache ? cache[strX] : (cache[strX] = fn(...args));
 };
};
```

如果你想了解一个性能最佳的记忆化函数的开发情况，可以阅读 Caio Gondim 的文章*How I wrote the world's fastest JavaScript memoization library*，在线可供阅读[`community.risingstack.com/the-worlds-fastest-javascript-memoization-library/`](https://community.risingstack.com/the-worlds-fastest-javascript-memoization-library/)。

# 记忆化测试

测试记忆化高阶函数提出了一个有趣的问题--你会怎么做？第一个想法是查看缓存--但那是私有的，不可见的。当然，我们可以改变`memoize()`来使用全局缓存，或者以某种方式允许外部访问缓存，但这种内部检查是不受欢迎的：你应该尝试仅基于外部属性进行测试。

接受我们应该省略尝试检查缓存，我们可以进行时间控制：调用一个函数，比如`fib()`，对于一个很大的 n 值，如果函数没有进行记忆化，应该需要更长的时间。这当然是可能的，但也容易出现可能的失败：你的测试之外的某些东西可能会在恰好的时候运行，可能你的记忆化运行时间会比原始运行时间更长。好吧，这是可能的，但不太可能--但你的测试并不完全可靠。

然后，让我们更直接地分析记忆化函数的实际调用次数。使用非记忆化的原始`fib()`，我们可以首先测试函数是否正常工作，并检查它调用了多少次：

```js
var fib = null;

beforeEach(() => {
 fib = n => {
 if (n == 0) {
 return 0;
 } else if (n == 1) {
 return 1;
 } else {
 return fib(n - 2) + fib(n - 1);
 }
 };
});

describe("the original fib", function() {
 it("should produce correct results", () => {
 expect(fib(0)).toBe(0);
 expect(fib(1)).toBe(1);
 expect(fib(5)).toBe(5);
 expect(fib(8)).toBe(21);
 expect(fib(10)).toBe(55);
 });

 it("should repeat calculations", () => {
 spyOn(window, "fib").and.callThrough();
 expect(fib(6)).toBe(8);
 expect(fib).toHaveBeenCalledTimes(25);
 });
});
```

`fib(6)`等于 8 这一事实很容易验证，但你怎么知道函数被调用了 25 次？为了回答这个问题，让我们重新看一下之前在第四章中看到的图表，*行为得体-纯函数*：

图 6.1。计算 fib(6)所需的所有递归调用。

每个节点都是一个调用；仅仅计数，我们得到为了计算`fib(6)`，实际上有 25 次对`fib()`的调用。现在，让我们转向函数的记忆版本。测试它是否仍然产生相同的结果很容易：

```js
describe("the memoized fib", function() {
 beforeEach(() => {
 fib = memoize(fib);
 });

 it("should produce same results", () => {
 expect(fib(0)).toBe(0);
 expect(fib(1)).toBe(1);
 expect(fib(5)).toBe(5);
 expect(fib(8)).toBe(21);
 expect(fib(10)).toBe(55);
 });

 it("shouldn't repeat calculations", () => {
 spyOn(window, "fib").and.callThrough();

 expect(fib(6)).toBe(8); // 11 calls
 expect(fib).toHaveBeenCalledTimes(11);

 expect(fib(5)).toBe(5); // 1 call
 expect(fib(4)).toBe(3); // 1 call
 expect(fib(3)).toBe(2); // 1 call
 expect(fib).toHaveBeenCalledTimes(14);
 });
});
```

但为什么在计算`fib(6)`时被调用了 11 次，然后在计算`fib(5)`，`fib(4)`和`fib(3)`之后又被调用了三次？为了回答问题的第一部分，让我们分析一下之前看到的图：

+   首先，我们调用`fib(6)`，它调用了`fib(4)`和`fib(5)`：三次调用

+   在计算`fib(4)`时，调用了`fib(2)`和`fib(3)`；计数增加到了五

+   在计算`fib(5)`时，调用了`fib(3)`和`fib(4)`；计数上升到 11

+   最后，计算并缓存了`fib(6)`

+   `fib(3)`和`fib(4)`都被缓存了，所以不再进行调用

+   `fib(5)`被计算并缓存

+   在计算`fib(2)`时，调用了`fib(0)`和`fib(1)`；现在我们有了七次调用

+   在计算`fib(3)`时，调用了`fib(1)`和`fib(2)`；计数增加到了九

+   `fib(4)`被计算并缓存

+   `fib(1)`和`fib(2)`都已经被缓存了，所以不会再进行进一步的调用

+   `fib(3)`被计算并缓存

+   在计算`fib(0)`和`fib(1)`时，不会进行额外的调用，两者都被缓存了

+   `fib(2)`被计算并缓存

哇！所以`fib(6)`的调用次数是 11--现在，鉴于所有`fib(n)`的值都已经被缓存，对于 n 从 0 到 6，很容易看出计算`fib(5)`，`fib(4)`和`fib(3)`只会增加三次调用：所有其他所需的值都已经被缓存。

# 改变函数

在前一节中，我们考虑了一些包装函数的方法，使它们保持其原始功能，尽管在某些方面得到了增强。现在我们将转而实际修改函数的功能，使新的结果实际上与原始函数的结果不同。

# 重新做一次事情

回到第二章，*思考功能性-第一个例子*，我们通过一个简单的问题的 FP 风格解决方案的例子：修复一个给定函数只能工作一次的问题：

```js
const once = func => {
 let done = false;
 return (...args) => {
 if (!done) {
 done = true;
 func(...args);
 }
 };
};
```

这是一个完全合理的解决方案，我们没有任何异议。然而，我们可以考虑一种变体。我们可以观察到给定的函数被调用一次，但其返回值被丢失了。然而，这很容易解决；我们只需要添加一个`return`语句。然而，这还不够；如果调用更多次，函数会返回什么呢？我们可以借鉴记忆化解决方案，并为将来的调用存储函数的返回值：

```js
const once2 = func => {
 let done = false;
    let result;
 return (...args) => {
 if (!done) {
 done = true;
            result = func(...args);
 }
        return result;
 };
};
```

你也可以考虑使函数仅对每组参数起作用一次...但是你不必为此做任何工作：`memoize()`就足够了！

回到提到的第二章，*函数式思维 - 第一个例子*，我们考虑了`once()`的一个可能替代品：另一个高阶函数，它以两个函数作为参数，并且只允许调用第一个函数一次，从那时起调用第二个函数。添加一个`return`语句，它将如下所示：

```js
const onceAndAfter = (f, g) => {
 let done = false;
 return (...args) => {
 if (!done) {
 done = true;
 return f(...args);
 } else {
 return g(...args);
 }
 };
};
```

如果我们记得函数是一级对象，我们可以重写这个过程。我们可以使用一个变量（`toCall`）直接存储需要调用的函数，而不是使用标志来记住要调用哪个函数。从逻辑上讲，该变量将被初始化为第一个函数，但随后将更改为第二个函数：

```js
const onceAndAfter2 = (f, g) => {
    let toCall = f;
 return (...args) => {
 let result = toCall(...args);
        toCall = g;
 return result;
 };
};
```

我们之前看到的完全相同的例子仍然可以工作：

```js
const squeak = (x) => console.log(x, "squeak!!");
const creak = (x) => console.log(x, "creak!!");
const makeSound = onceAndAfter2(squeak, creak);

makeSound("door"); // *"door squeak!!"*
makeSound("door"); // *"door creak!!"*
makeSound("door"); // *"door creak!!"*
makeSound("door"); // *"door creak!!"*
```

在性能方面，差异可能微乎其微。展示这种进一步变化的原因只是为了记住，通过存储函数，你通常可以以更简单的方式产生结果。在过程式编程中，使用标志存储状态是一种常见的技术，随处可见。然而，在这里，我们设法跳过了这种用法，但却产生了相同的结果。

# 逻辑否定一个函数

让我们考虑一下来自第五章的`.filter()`方法，*声明式编程 - 更好的风格*。给定一个谓词，我们可以过滤数组，只包括谓词为真的元素。但是如何进行反向过滤并*排除*谓词为真的元素呢？

第一个解决方案应该是相当明显的：重新设计谓词，使其返回与原始返回值相反的值。在前面提到的章节中，我们看到了这个例子：

```js
const delinquent = serviceResult.accountsData.filter(v => v.balance < 0);
```

因此，我们可以以另一种方式写出它，以这两种等效方式之一：

```js
const notDelinquent = serviceResult.accountsData.filter(
    v => v.balance >= 0
);

const notDelinquent2 = serviceResult.accountsData.filter(
    v => !(v.balance < 0)
);
```

这是完全可以的，但我们也可以有类似以下的东西：

```js
const isNegativeBalance = v => v.balance < 0;

// ...*many lines later..*.

const delinquent2 = serviceResult.accountsData.filter(isNegativeBalance);
```

在这种情况下，重写原始函数是不可能的。然而，在函数式编程中，我们可以编写一个高阶函数，它将接受任何谓词，评估它，然后否定其结果。由于 ES8 的语法，可能的实现会非常简单：

```js
const not = fn => (...args) => !fn(...args);
```

以这种方式工作，我们可以将前面的过滤重写为以下形式：

```js
const isNegativeBalance = v => v.balance < 0;

// ...*many lines later...* 
const notDelinquent3 = serviceResult.accountsData.filter(
    not(isNegativeBalance)
);
```

我们可能想要尝试的另一个解决方案是--而不是颠倒条件（如我们所做的），我们可以编写一个新的过滤方法（可能是`filterNot()`？），它将以与`filter()`相反的方式工作：

```js
const filterNot = arr => fn => arr.filter(not(fn));
```

这个解决方案与`.filter()`并不完全匹配，因为你不能将其用作方法，但我们可以将其添加到`Array.prototype`中，或者应用一些我们将在第八章中看到的方法，*连接函数 - 管道和组合*。然而，更有趣的是，我们使用了否定的函数，因此`not()`对于反向过滤问题的两种解决方案都是必要的。在即将到来的去方法化部分中，我们将看到另一个解决方案，因为我们将能够将诸如`.filter()`之类的方法与它们适用的对象分离开来，将它们变成普通函数。

至于否定函数*与*使用新的`filterNot()`，尽管两种可能性同样有效，但我认为使用`not()`更清晰；如果你已经理解了过滤的工作原理，那么你几乎可以大声朗读它，它就会被理解：我们想要那些没有负余额的，对吧？

# 反转结果

与前面的过滤问题类似，现在让我们重新讨论第三章中的*注入-排序*部分中的排序问题，*从函数开始-核心概念*。我们想要使用特定的方法对数组进行排序，并且我们使用了`.sort()`，提供了一个比较函数，基本上指出了哪个字符串应该先进行排序。为了提醒你，给定两个字符串，函数应该执行以下操作：

+   如果第一个字符串应该在第二个字符串之前，则返回一个负数

+   如果两个字符串相同，则返回零

+   返回一个正数，如果第一个字符串应该跟在第二个字符串后面

让我们回到我们之前在西班牙语排序中看到的代码。我们必须编写一个特殊的比较函数，以便排序能够考虑西班牙语的特殊字符顺序规则，比如在*n*和*o*之间包括字母*ñ*，等等。

```js
const spanishComparison = (a, b) => a.localeCompare(b, "es");

palabras.sort(spanishComparison); // *sorts the* palabras *array according to Spanish rules*
```

我们面临着类似的问题：我们如何能够以*降序*的方式进行排序？根据我们在前一节中看到的内容，应该立即想到两种替代方案：

+   编写一个函数，它将反转比较函数的结果。这将反转所有关于哪个字符串应该在前面的决定，最终结果将是一个完全相反排序的数组。

+   编写一个`sortDescending()`函数或方法，以与`sort()`相反的方式进行工作。

让我们编写一个`invert()`函数，它将改变比较的结果。代码本身与前面的`not()`非常相似：

```js
const invert = fn => (...args) => -fn(...args);
```

有了这个高阶函数，我们现在可以通过提供一个适当反转的比较函数来进行降序排序：

```js
const spanishComparison = (a, b) => a.localeCompare(b, "es");

var palabras = ["ñandú", "oasis", "mano", "natural", "mítico", "musical"];

palabras.sort(spanishComparison);
// ["mano", "mítico", "musical", "natural", "ñandú", "oasis"]

palabras.sort(**invert(spanishComparison)**);
// ["oasis", "ñandú", "natural", "musical", "mítico", "mano"]
```

输出与预期相符：当我们`invert()`比较函数时，结果是相反的顺序。顺便说一句，编写单元测试将非常容易，因为我们已经有了一些测试用例和它们的预期结果，不是吗？

# 改变参数数量

回到第五章中*隐式地*解析数字的部分，我们看到使用`parseInt()`与`.reduce()`会产生问题，因为该函数的参数数量是意外的，它需要多于一个参数：

```js
["123.45", "-67.8", "90"].map(parseInt); // *problem: parseInt isn't monadic!*
// [123, NaN, NaN]
```

我们有多种解决方法。在提到的章节中，我们选择了箭头函数，这是一个简单的解决方案，而且具有清晰易懂的优势。在第七章中，*转换函数-柯里化和部分应用*，我们将看到另一种方法，基于部分应用。但是，在这里，让我们使用一个高阶函数。我们需要的是一个函数，它将另一个函数作为参数，并将其转换为一元函数。使用 JS 的展开运算符和箭头函数，这很容易管理：

```js
const unary = fn => (...args) => fn(args[0]);
```

使用这个函数，我们的数字解析问题就解决了：

```js
["123.45", "-67.8", "90"].map(unary(parseInt));
// *[123, -67, 90]*
```

不用说，同样简单地定义进一步的`binary()`、`ternary()`等函数，可以将任何函数转换为等效的、限定数量参数的版本。

你可能会认为没有多少情况需要应用这种解决方案，但事实上，情况比你想象的要多得多。通过查看所有 JavaScript 的函数和方法，你可以轻松地列出一个以`.apply()`、`.assign()`、`.bind()`、`.concat()`、`.copyWithin()`...等等开头的列表！如果你想以一种心照不宣的方式使用其中任何一个，你可能需要修复它的参数数量，这样它就可以使用固定的、非可变的参数数量。

如果你想要一个漂亮的 JavaScript 函数和方法列表，请查看[`developer.mozilla.org/en/docs/Web/JavaScript/Guide/Functions`](https://developer.mozilla.org/en/docs/Web/JavaScript/Guide/Functions)和[`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Methods_Index`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Methods_Index)上的页面。至于暗示（或点自由风格）编程，我们将在第八章中回到它，*连接函数 - 管道和组合*。

# 其他高阶函数

让我们在本章结束时考虑其他杂项函数，提供诸如新查找器、将方法与对象解耦等结果。

# 将操作转换为函数

我们已经看到了几种情况，我们需要编写一个函数来添加或乘以一对数字。例如，在第五章的*求和数组*部分，*声明式编程 - 更好的风格*，我们不得不编写等效于以下代码的代码：

```js
const mySum = myArray.reduce((x, y) => x + y, 0);
```

在同一章节中，在*使用范围*部分，为了计算阶乘，我们需要这样：

```js
const factorialByRange = n => range(1, n + 1).reduce((x, y) => x * y, 1);
```

如果我们能够将二元运算符转换为计算相同结果的函数，那将会更容易。前面的两个例子可以更简洁地写成如下所示：

```js
const mySum = myArray.reduce(binaryOp("+"), 0);
const factorialByRange = n => range(1, n + 1).reduce(binaryOp("*"), 1);
```

# 实施操作

我们如何编写这个`binaryOp()`函数？至少有两种方法：一种安全但冗长，一种更冒险但更短的替代方法。第一种方法需要列出每个可能的运算符：

```js
const binaryOp1 = op => {
 switch (op) {
 case "+":
 return (x, y) => x + y;
 case "-":
 return (x, y) => x - y;
 case "*":
 return (x, y) => x * y;
 //
 // etc.
 //
 }
};
```

这个解决方案完全没问题，但需要太多的工作。第二个更危险，但更短。请将其仅视为一个示例，用于学习目的；出于安全原因，不建议使用`eval()`！

```js
const binaryOp2 = op => new Function("x", "y", `return x ${op} y;`);
```

如果你遵循这种思路，你也可以定义一个`unaryOp()`函数，尽管它的应用更少。 （我把这个实现留给你；它与我们已经写的内容非常相似。）在即将到来的第七章中，*转换函数 - 柯里化和部分应用*，我们将看到创建这个一元函数的另一种方法，即使用部分应用。

# 更方便的实现

让我们超前一步。进行 FP 并不意味着总是要回到非常基本、最简单的函数。例如，在第八章的*转换为自由点风格*部分，*连接函数 - 管道和组合*，我们将需要一个函数来检查一个数字是否为负数，并考虑使用`binaryOp2()`来编写它：

```js
const isNegative = curry(binaryOp2(">"))(0);
```

现在不要担心`curry()`函数（我们很快会在第七章中讨论它，*转换函数 - 柯里化和部分应用*），但其思想是将第一个参数固定为零，因此我们的函数将检查给定数字*n*是否*0>n*。这里的重点是，我们刚刚编写的函数并不是很清晰。如果我们定义一个二元操作函数，还可以让我们指定其参数之一，左边的参数或右边的参数，以及要使用的运算符，我们可以做得更好：

```js
const binaryLeftOp = (x, op) => 
 (y) => binaryOp2(op)(x,y);

const binaryOpRight = (op, y) => 
 (x) => binaryOp2(op)(x,y);
```

或者，你可以回到`new Function()`风格的代码：

```js
const binaryLeftOp2 = (x, op) => y => binaryOp2(op)(x, y);

const binaryOpRight2 = (op, y) => x => binaryOp2(op)(x, y);
```

有了这些新函数，我们可以简单地写出以下任一代码--尽管我认为第二个更清晰：我宁愿测试一个数字是否小于零，而不是零是否大于该数字：

```js
const isNegative1 = binaryLeftOp(0, ">");

const isNegative2 = binaryOpRight("<", 0);
```

这有什么意义？不要追求某种*基本简单*或*回归基础*的代码。我们可以将运算符转换为函数，没错--但如果你能做得更好，并通过允许指定操作的两个参数之一来简化编码，那就去做吧！FP 的理念是帮助编写更好的代码，而创造人为限制对任何人都没有好处。

当然，对于一个简单的函数，比如检查一个数字是否为负数，我绝对不想用柯里化、二元运算符或点自由风格或其他任何东西来复杂化事情，我只会毫不犹豫地写出以下内容：

```js
const isNegative3 = x => x < 0;
```

# 将函数转换为 promises

在 Node 中，大多数异步函数需要一个回调，比如`(err,data)=>{...}`：如果`err`是`null`，函数成功，`data`是其结果，如果`err`有一些值，函数失败，`err`给出了原因。（有关更多信息，请参见[`nodejs.org/api/errors.html#errors_node_js_style_callbacks`](https://nodejs.org/api/errors.html#errors_node_js_style_callbacks)。）

但是，您可能更喜欢使用 promises。因此，我们可以考虑编写一个高阶函数，将需要回调的函数转换为一个 promise，让您使用`.then()`和`.catch()`方法。（在第十二章中，*构建更好的容器-功能数据类型*，我们将看到 promises 实际上是 monads，因此这种转换在另一个方面也很有趣。）

我们如何管理这个？转换相当简单。给定一个函数，我们生成一个新的函数：这将返回一个 promise，当使用一些参数调用原始函数时，将适当地`reject()`或`resolve()`promise：

```js
const promisify = fn => (...args) =>
 new Promise((resolve, reject) =>
 fn(...args, (err, data) => (err ? reject(err) : resolve(data)))
 );
```

有了这个函数，我们可以这样写代码：

```js
const fs = require("fs");

const cb = (err, data) =>
 err ? console.log("ERROR", err) : console.log("SUCCESS", data);

fs.readFile("./exists.txt", cb); // *success, list the data*
fs.readFile("./doesnt_exist.txt", cb); // *failure, show exception*
```

相反，您可以使用 promises：

```js
const fspromise = promisify(fs.readFile.bind(fs));

const goodRead = data => console.log("SUCCESSFUL PROMISE", data);
const badRead = err => console.log("UNSUCCESSFUL PROMISE", err);

fspromise("./readme.txt") *// success*
 .then(goodRead)
 .catch(badRead);

fspromise("./readmenot.txt") // *failure*
 .then(goodRead)
 .catch(badRead);
```

现在您可以使用`fspromise()`而不是原始方法。我们必须绑定`fs.readFile`，正如我们在第三章的*一个不必要的错误*部分中提到的那样，*从函数开始-核心概念*。

# 从对象中获取属性

有一个简单但经常使用的函数，我们也可以生成。从对象中提取属性是一个常见的操作。例如，在第五章中，*以声明方式编程-更好的风格*，我们需要获取纬度和经度以便计算平均值：

```js
markers = [
 {name: "UY", lat: -34.9, lon: -56.2},
 {name: "AR", lat: -34.6, lon: -58.4},
 {name: "BR", lat: -15.8, lon: -47.9},
 ...
 {name: "BO", lat: -16.5, lon: -68.1}
];

let averageLat = average(markers.map(x => x.lat));
let averageLon = average(markers.map(x => x.lon));
```

当我们看到如何过滤数组时，我们有另一个例子；在我们的例子中，我们想要获取所有余额为负的帐户的 ID，并在过滤掉所有其他帐户后，我们仍然需要提取 ID 字段：

```js
const delinquent = serviceResult.accountsData.filter(v => v.balance < 0);
const delinquentIds = delinquent.map(v => v.id);
```

我们本可以将这两行合并，并用一行代码产生所需的结果，但这里并不重要。事实上，除非`delinquent`中间结果出于某种原因是必需的，大多数 FP 程序员都会选择一行解决方案。

我们需要什么？我们需要一个高阶函数，它将接收一个属性的名称，并产生一个新的函数作为其结果，这个函数将能够从对象中提取所述属性。使用 ES8 语法，这个函数很容易编写：

```js
const getField = attr => obj => obj[attr];
```

在第十章的*获取器和设置器*部分，*确保纯度-不可变性*，我们将编写这个函数的更通用版本，能够“深入”到对象中，获取对象的任何属性，无论其在对象中的位置如何。

有了这个函数，坐标提取可以这样写：

```js
let averageLat = average(markers.map(getField("lat")));
let averageLon = average(markers.map(getField("lon")));
```

为了多样化，我们可以使用辅助变量来获取拖欠的 ID。

```js
const getId = getField("id");
const delinquent = serviceResult.accountsData.filter(v => v.balance < 0);
const delinquentIds = delinquent.map(getId);
```

一定要完全理解这里发生了什么。`getField()`调用的结果是一个函数，将在进一步的表达式中使用。`map()`方法需要一个映射函数，这就是`getField()`产生的东西。

# 去方法化-将方法转换为函数

`.filter()`或`.map()`等方法仅适用于数组--但实际上，你可能希望将它们应用于`NodeList`或`String`，但你可能会碰壁。此外，我们正在关注字符串，因此必须将这些函数用作方法并不是我们想要的。最后，每当我们创建一个新函数（比如`none()`，我们在第五章 *以更好的方式编程 - 声明式编程* 的*检查否定*部分中看到的），它不能像它的同行（在这种情况下是`.some()`和`.every()`）那样应用，除非你做一些原型的把戏--这是被严厉谴责的，也完全不推荐...但是请看第十二章 *构建更好的容器 - 函数数据类型* 的*扩展当前数据类型*部分，我们将使`.map()`适用于大多数基本类型！

那么...我们能做什么呢？我们可以应用古话*如果山不来，穆罕默德就去山*，而不是担心无法创建新的方法，我们将现有的方法转换为函数。如果我们将每个方法转换为一个函数，该函数将作为其第一个参数接收它将要操作的对象。

解耦方法和对象可以帮助你，因为一旦你实现了这种分离，一切都变成了一个函数，你的代码会更简单。（还记得我们在*逻辑否定一个函数*中写的内容吗，关于可能的`filterNot()`函数与`.filter()`方法的比较？）解耦的方法在某种程度上类似于其他语言中所谓的*通用*函数，因为它们可以应用于不同的数据类型。

在 ES8 中，有三种不同但相似的实现方式。列表中的第一个参数将对应于对象；其他参数将对应于被调用方法的实际参数。

请参阅[`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function)以了解`apply()`、`call()`和`bind()`的解释。顺便说一句，在第一章 *成为函数 - 几个问题* 中，我们看到了在使用展开运算符时`.apply()`和`.call()`之间的等价性。

```js
const demethodize1 = fn => (arg0, ...args) => fn.apply(arg0, args);

const demethodize2 = fn => (arg0, ...args) => fn.call(arg0, ...args);

const demethodize3 = fn => (...args) => fn.bind(...args)();
```

还有另一种方法：`demethodize = Function.prototype.bind.bind(Function.prototype.call)`。如果你想了解这是如何工作的，请阅读 Leland Richardson 的*Clever way to* demethodize *Native JS Methods*，网址为[`www.intelligiblebabble.com/clever-way-to-demethodize-native-js-methods`](http://www.intelligiblebabble.com/clever-way-to-demethodize-native-js-methods)。

让我们看一些应用！从一个简单的例子开始，我们可以使用`.map()`来循环遍历一个字符串，而不必先将其转换为字符数组。假设你想将一个字符串分隔成单个字母并将它们转换为大写：

```js
const name = "FUNCTIONAL";
const result = name.split("").map(x => x.toUpperCase());
// *["F", "U", "N", "C", "T", "I", "O", "N", "A", "L"]*
```

然而，如果我们解除了`.map()`和`.toUpperCase()`，我们可以简单地写成以下形式：

```js
const map = demethodize3(Array.prototype.map);
const toUpperCase = demethodize3(String.prototype.toUpperCase);

const result2 = map(name, toUpperCase);
// *["F", "U", "N", "C", "T", "I", "O", "N", "A", "L"]*
```

是的，对于这种特殊情况，我们可以先将字符串转换为大写，然后将其拆分为单独的字母，如`name.toUpperCase().split("")` -- 但这不会是一个很好的例子，毕竟有两个解除方法的用法，对吧？

类似地，我们可以将一个十进制金额数组转换为格式正确的字符串，带有千位分隔符和小数点：

```js
const toLocaleString = demethodize3(Number.prototype.toLocaleString);

const numbers = [2209.6, 124.56, 1048576];
const strings = numbers.map(toLocaleString);
// *["2,209.6", "124.56", "1,048,576"]*
```

或者，给定前面的 map 函数，这也可以工作：

```js
const strings2 = map(numbers, toLocaleString);
```

将方法解除为函数的想法在不同的情况下将会非常有用。我们已经看到了一些例子，我们可以应用它，并且在本书的其余部分还会有更多这样的情况。

# 找到最佳解决方案

让我们通过创建`.find()`方法的扩展来结束本节。假设我们想要找到数组中的最优值--假设它是最大值--：

```js
const findOptimum = arr => Math.max(...arr);

const myArray = [22, 9, 60, 12, 4, 56];
findOptimum(myArray); // 60
```

现在，这是否足够通用？这种方法至少存在一对问题。首先，你确定集合的最优值总是最大值吗？如果你考虑了几种抵押贷款，那么利率最低的那个可能是最好的，不是吗？假设我们总是想要集合的*最大值*太过于局限了。

你可以绕个弯：如果你改变数组中所有数字的符号，找到它的最大值，然后改变它的符号，那么你实际上得到了数组的最小值。在我们的例子中，`-findOptimum(myArray.map((x) => -x))`将产生 4--但这不是容易理解的代码。

其次，找到最大值的这种方式取决于每个选项都有一个数值。但如果这样的值不存在，你该如何找到最优值？通常的方法依赖于将元素相互比较，并选择在比较中排在前面的元素：将第一个元素与第二个元素进行比较，并保留其中较好的那个；然后将该值与第三个元素进行比较，并保留最好的；依此类推，直到你完成了所有元素的遍历。

以更一般的方式解决这个问题的方法是假设存在一个`comparator()`函数，它以两个元素作为参数，并返回最好的那个。如果你能为每个元素关联一个数值，那么比较函数可以简单地比较这些值。在其他情况下，它可以根据需要执行任何逻辑，以便决定哪个元素排在前面。

让我们尝试创建一个合适的高阶函数：

```js
const findOptimum2 = fn => arr => arr.reduce(fn);
```

有了这个，我们可以轻松地复制最大值和最小值查找函数。

```js
const findMaximum = findOptimum2((x, y) => (x > y ? x : y));
const findMinimum = findOptimum2((x, y) => (x < y ? x : y));

findMaximum(myArray); // 60
findMinimum(myArray); // 4
```

让我们更上一层楼，比较非数值值。假设有一款超级英雄卡牌游戏：每张卡代表一个英雄，具有几个数值属性，如力量、能力和科技。当两个英雄互相对抗时，具有更多类别的英雄，其数值高于另一个英雄，将成为赢家。让我们为此实现一个比较器：

```js
const compareHeroes = (card1, card2) => {
 const oneIfBigger = (x, y) => (x > y ? 1 : 0);

 const wins1 =
 oneIfBigger(card1.strength, card2.strength) +
 oneIfBigger(card1.powers, card2.powers) +
 oneIfBigger(card1.tech, card2.tech);

 const wins2 =
 oneIfBigger(card2.strength, card1.strength) +
 oneIfBigger(card2.powers, card1.powers) +
 oneIfBigger(card2.tech, card1.tech);

 return wins1 > wins2 ? card1 : card2;
};
```

然后，我们可以将这应用到我们的英雄“比赛”中：

```js
function Hero(n, s, p, t) {
 this.name = n;
 this.strength = s;
 this.powers = p;
 this.tech = t;
}

const codingLeagueOfAmerica = [
 new Hero("Forceful", 20, 15, 2),
 new Hero("Electrico", 12, 21, 8),
 new Hero("Speediest", 8, 11, 4),
 new Hero("TechWiz", 6, 16, 30)
];

const findBestHero = findOptimum2(compareHeroes);
findBestHero(codingLeagueOfAmerica); // Electrico is the top hero!
```

当你根据一对一比较对元素进行排名时，可能会产生意想不到的结果。例如，根据我们的超级英雄比较规则，你可能会找到三个英雄，第一个击败第二个，第二个击败第三个，但第三个击败第一个！在数学术语中，这意味着比较函数不是传递的，你没有集合的*完全排序*。

# 问题

6.1\. **一个边界情况**。如果我们将`getField()`函数应用于一个空对象，会发生什么？它应该是什么行为？如果需要，修改该函数。

6.2\. **多少次？** 要计算`fib(50)`需要多少次调用而不使用记忆化？例如，计算`fib(0)`或`fib(1)`，只需要一次调用，不需要进一步递归，而对于`fib(6)`，我们看到需要 25 次调用。你能找到一个公式来做这个计算吗？

6.3\. **一个随机平衡器**。编写一个高阶函数`randomizer(fn1, fn2, ...)`，它将接收可变数量的函数作为参数，并返回一个新的函数，该函数在每次调用时将随机调用`fn1`、`fn2`等。如果每个函数都能执行 Ajax 调用，你可能会用到这个函数来平衡对服务器上不同服务的调用。为了加分，确保连续两次不会调用同一个函数。

6.4\. **只说不！** 在本章中，我们编写了一个与布尔函数一起工作的`not()`函数和一个与数值函数一起工作的`negate()`函数。你能更上一层楼，只编写一个`opposite()`函数，根据需要表现为`not()`或`negate()`吗？

# 总结

在本章中，我们已经看到如何编写我们自己的高阶函数，它可以包装另一个函数以提供一些新功能，改变函数的目标以便做其他事情，甚至是全新的功能，比如将方法与对象解耦或创建更好的查找器。

在第七章中，*函数转换-柯里化和部分应用*，我们将继续使用高阶函数，并且我们将看到如何通过柯里化和部分应用来生成现有函数的专门版本，带有预定义的参数。
