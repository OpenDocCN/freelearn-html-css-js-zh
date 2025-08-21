# 第二章：Promise API 及其兼容性

Promises 对于 JavaScript 世界来说是相当新的，但绕过方法已经存在一段时间了。正如我们在上一章所看到的，有方法可以解决 JavaScript 中的异步编程问题，无论是通过事件还是回调。你还了解到为什么 promise 与传统技术不同。

接下来，我们将详细介绍 Promise API。你还将了解 promises 标准的当前浏览器支持，并查看实现 promises 和 promise-like 特性的 JavaScript 库。在本章中，我们将涵盖以下主题：

+   Promise API 及其详细内容

+   浏览器兼容性

+   Promise 实施情况

+   具有 promise 类似特性的库

# 了解 API

在整本书中，我们将主要讨论并使用在 Promises/A+ 规范中定义的 promise。（*[`promisesaplus.com/`](http://promisesaplus.com/)*）Promises/A+ 组织制定了 Promises/A+ 规范，目的是将初始的 Promises/A 规范阐述得更为清晰、测试更为充分。以下是从他们网站引用的一段话：

|   | *Promises/A+ 是基于 CommonJS Promises/A 提案中提出的概念和 `then` API 构建的.* |   |
| --- | --- | --- |
|   | --*[`promisesaplus.com/differences-from-promises-a`](http://promisesaplus.com/differences-from-promises-a)* |

这些差异体现在三个层面：省略、添加和澄清。在省略层面，Promises/A+ 从原始版本中移除了以下功能：

+   **进度处理**：此功能包括一个在操作/promise 仍在进行中时处理的回调函数，即尚未完成或拒绝。它被移除是因为实施者认为，在实践中，这些功能证明是规格不足的，目前在 promise 实施者社区中对其行为没有达成完全一致。

+   **交互式承诺**：这个特性在之前的 Promises/A 提案中是一个扩展的承诺，它基本上为承诺方法支持了两个额外的函数；`get(propertyName)`，从 promise 的目标请求给定的属性，和 `call(functionName, arg1, arg2, ...)`，在 promise 的目标的参数上调用给定的方法/函数。在新的 A+ 规范中，这个特性以及两个函数 `call` 和 `get`，在实现 interoperable promises 所需的基本 API 时被认为是超出范围的。

+   `promise!== resultPromise`：这个特性在旧提案中是一个要求，它指出 promise 的结果不应该等于 promise，例如，`var resultPromise = promise.then(onFulfilled, onRejected)`。实际上，任何实现都可能允许 `resultPromise === promise`，只要实现满足所有要求。

在添加层面上，Promises/A+ 规格书在现有的 Promises/A 提案中增加了以下特性和要求：

+   在 `onFulfilled` 或 `onRejected` 返回一个 thenable 的情况下，包括解决过程的详细规格。

+   传递给 `onRejected` 处理程序的原因，必须是那种情况下抛出的异常。

+   必须异步调用两个处理程序 `onFulfilled` 和 `onRejected`。

+   必须调用两个处理程序 `onFulfilled` 和 `onRejected`。

+   实现必须遵守在相同承诺上连续调用 `then` 方法时处理程序 `onFulfilled` 和 `onRejected` 的确切调用顺序。用更通俗的话来说，这意味着如果像 `promise.then().then()` 这样在同一个承诺上多次调用 `then` 方法，所有这些 `then` 调用中使用的 `onFulfilled` 处理程序必须按照原始调用 `then` 的顺序执行。因此，第一个 `then` 函数中的 `onFulfilled` 回调将首先执行，接着是第二个 `then` 中的 `onFulfilled` 回调，依此类推。在這種情況下，`onRejected` 回调的执行也是如此。是否非常复杂？也许下面的例子可以解释得更好：

    ```js
    var p = [[promise]];
    p.then();
    p.then();
    ```

    前面的代码不同于下面的代码行：

    ```js
    promise.then().then();
    ```

    区别在于 `promise.then()` 可能返回一个不同的承诺。

最后，在澄清的层面上，Promises/A+ 提案对 Promises/A 使用了不同的命名，因为新规格的作者希望反映在承诺实现中已经传播的词汇。这些变化包括以下内容：

+   承诺状态被称为挂起、满足和拒绝，代替未满足、满足和失败。

+   当一个承诺被满足时，承诺有一个 *值*；同样，当一个承诺被拒绝时，它有一个 *原因*。

`then` 方法是 API 中的主要角色。如果一个对象没有指定 `then` 方法来检索和访问其当前或最终值或原因，那么它就不被视为一个承诺，正如我们在上一章所看到的。这个方法需要两个参数，都必须是函数，如下面的例子所示：

```js
promise.then(onFulfilled, onRejected);
```

让我们深入探讨 `then` 的细节和其参数的规格，考虑到之前代码示例中的简单 `then` 方法：

+   `onFulfilled` 和 `onRejected` 两个参数都是可选的。

+   两个参数都必须是函数；否则，它必须被忽略。

+   在同一个 `then` 调用中，两个参数不得调用超过一次。

+   `onFulfilled` 参数必须在承诺被满足后调用，以承诺的值为第一个参数。

+   `onRejected` 参数必须在承诺被拒绝后调用，以拒绝的原因作为其第一个参数。

+   `onFulfilled` 和 `onRejected` 参数不能作为 `this` 值传递，因为如果我们对 JavaScript 代码应用严格模式，这将在处理程序内部被当作未定义处理；在怪异模式中，它将被当作那个 JavaScript 代码的全球对象处理。

+   `then` 方法可以在同一个承诺上被调用多次。

+   当一个承诺被满足时，所有相应的 `onFulfilled` 处理程序必须按照它们发起的 `then` 调用的顺序执行。同样的规则适用于 `onRejected` 回调。

+   `then` 方法必须返回一个承诺，如下所示：

    ```js
    promiseReturned = promise.then(onFulfilled, onRejected);
    ```

+   如果 `onFulfilled` 或 `onRejected` 返回一个值 `x`，承诺解决程序必须被调用以解决值 `x`，如下面的代码所示：

    ```js
    promiseReturned = promise1.then(onFulfilled, onRejected);
    [[Resolve]](promiseReturned, x).
    ```

+   如果 `onFulfilled` 或 `onRejected` 处理程序抛出异常 `e`，`promiseReturned` 必须以 `e` 为拒绝或失败的原因被拒绝。

+   如果 `onFulfilled` 不是一个函数且承诺被满足，`promiseReturned` 必须以相同的值被满足。

+   如果 `onRejected` 不是一个函数且 `promise1` 被拒绝，`promiseReturned` 必须用相同的原因拒绝。

前面列表是对在 Promises/A+ 开放标准中定义和指定的承诺和 `then` 方法的详细规范。我们之前谈到了承诺解决程序，但我们还不知道它是什么。好吧，承诺解决程序基本上是一个抽象操作，它接受一个承诺和一个值作为参数，如下所示：

```js
[[Resolve]](promise, x)]
```

如果 `x` 是一个 thenable，意味着它是一个定义了 `then` 方法的的对象或函数，`resolve` 方法将尝试强制一个承诺假设 `x` 的状态，假设 `x` 至少有点像一个承诺。否则，它将以值 `x` 满足承诺。

承诺解决程序使用的处理 thenables 的技术使得只要承诺暴露出一个符合 Promises/A+ 标准的 `then` 方法，承诺实现就可以可靠地相互工作。此外，它还允许实现*整合*具有合理 `then` 方法的的非标准实现。

| ``` |
| --- |
| ``` |

承诺解决过程允许我们有一个正确的`promise.resolve`实现。它也是保证`then`正确实现所必需的。你可能会注意到承诺解决过程中没有返回值，因为它是一个抽象过程，可以以任何作者认为合适的方式实现。因此，只要能达到最终目标，即把承诺置于与*x*相同的状态，返回值就留给实现者来决定。所以，从概念上讲，它影响承诺的状态转换。

尽管承诺解决过程的实现留给实现者，但它有一些我们想要遵守的规则，如果我们想要在需要运行它时符合该提案。这些规则如下：

1.  如果一个承诺和*x*引用同一个对象，那么在`onRejected`处理程序中，承诺应该以一个`TypeError`作为拒绝的原因。

1.  如果*x*是一个承诺，我们应该采用其当前状态。这个规则允许使用特定于实现的的行为来实际采用已知符合规范的承诺的状态。以下是一些条件：

    +   如果*x*处于待定状态，承诺必须保持待定状态，直到*x*被满足或被拒绝。

    +   如果/当*x*被满足时，承诺应该用*x*具有的相同值被满足。

    +   如果/当*x*被拒绝时，承诺应该用*x*被拒绝的相同原因被拒绝。

1.  如果*x*是一个对象或函数，且不是承诺，则执行以下操作：

    +   当我们想要调用`then`时，方法应该是`x.then`。这是一个必要的防御措施，可以确保在`accessor`属性面前的一致性。这个属性值在我们每次获取它时都可能发生变化。

    +   如果获取`x.then`属性最终抛出了异常`e`，那么该承诺应该用`e`作为原因被拒绝。

    +   如果`then`是一个函数，用*x*调用它，`this`的值为值。第一个参数应该是`resolvePromise`，第二个参数应该是`rejectPromise`。

    +   如果`then`不符合函数的要求，直接用*x*满足承诺。

1.  如果*x*既不是对象也不是函数，承诺应该用*x*来满足。

让我们看看第三条规则。我们发现，如果`then`是函数，第一个参数应该是`resolvePromise`，第二个参数应该是`rejectPromise`，其中以下规则适用：

1.  如果/当用值*z*调用`resolvePromise`时，实现必须运行`[[Resolve]](promise, z)`。

1.  如果/当`rejectPromise`用理由*j*被调用时，实现必须用理由 j 拒绝该承诺。

1.  如果同时调用了处理程序`resolvePromise`和`rejectPromise`，或者在同一参数上多次调用，第一次调用应优先考虑，其他后续调用均忽略。

1.  如果调用`then`导致抛出异常 e，我们有两个条件：

    +   如果`resolvePromise`或`rejectPromise`处理程序已经被调用，我们应该忽略`then`

    +   如果不然，实现应当拒绝该承诺，并以`e`作为返回的原因。

之前那长长的规则列表作为实现者的指导。所以，如果你在自己的公共 API 中实现`then`，这些规则应当适用于你的算法，以符合 Promises/A+标准规范。我向 Brian Cavalier 询问了关于 PRP 的需求，他添加了以下内容：

> **PRP 最重要的方面之一是，它被精心设计，以允许不同的承诺实现以可靠的方式互操作。**

此外，承诺解决程序甚至允许在非符合（略微危险）的 thenables 面前保持正确性。一个例子就是使用`resolve`函数将 jQuery 的承诺版本（不符合 A+标准）转换为非常简单的符合标准的承诺。以下代码说明了这种实现：

```js
// an ajax call that returns jquery promisevar jQueryPromise = $.ajax('/sample.json'); //correct it and convert it to a standard conforming promisevar standardPromise = Promise.resolve(jQueryPromise); 
```

归根结底，Promises/A+的核心目标是提供尽可能简单、最小的规范，以允许不同承诺实现之间的可靠互操作，即使面临危险。

### 注意

为了消除可能产生的任何混淆，承诺解决程序并不完全等同于某些实现在其公共 API 中提供的`promise.resolve`方法。

与 Promises/A+标准的核心目标保持一致，Promises/A+组织创建了一个符合性测试套件，以测试承诺库或 API 实现是否符合 Promises/A+规范。这些测试，可以在[`github.com/promises-aplus/promises-tests`](https://github.com/promises-aplus/promises-tests)找到，通过测试`then`来检查承诺解决程序的正确性。这些测试也旨在为实现是否满足要求并提供更具体的指导和证据，符合标准。

# 浏览器支持和兼容性

JavaScript 与浏览器紧密耦合，承诺也是如此，因为承诺在之前的 ECMAScript 版本中不是一个标准，并且将成为新的 ECMAScript 6 版本的组成部分；它们不会在所有浏览器上得到支持。此外，承诺可以被实现，我们将看到几个库提供类似承诺的功能或暴露承诺能力。在本章剩余的部分，我们将涵盖这两个对于使用承诺至关重要的要点。

## 检查浏览器兼容性

与任何客户端技术一样，JavaScript 是为了与 HTML 页面一起在网页浏览器中使用而专门开发的。它利用浏览器来完成工作，这就是为什么它是一种脚本语言。一旦脚本发送到浏览器，接下来如何处理就取决于浏览器了。这里有很大的依赖性；因此，浏览器兼容性至关重要。

一些浏览器已经有了承诺的实现；在撰写本书时，支持承诺的浏览器选择很少，正如 Kangax 所示的以下 ECMAScript 6 兼容性表所示：

![检查浏览器兼容性](img/00002.jpeg)

来源：[`kangax.github.io/compat-table/es6/#Promise`](http://kangax.github.io/compat-table/es6/#Promise)

### 注意

**兼容性表格中使用的缩写**

IE 代表 Internet Explorer，FF 代表 Firefox，CH 代表 Chrome，SF 代表 Safari，WK 代表 Webkit，OP 代表 Opera。

正如前表所示，只有最新三个版本的 Firefox（截至版本 29）和 Chrome（截至 32 版本）默认启用承诺。不必担心，因为有一个 polyfill 可以将承诺功能添加到尚不支持它的浏览器中。

### 注意

补丁是一个相对较新的术语，由 Remy Sharp 提出，并在网页开发者社区中流行起来。它代表了一段代码，提供了我们期望浏览器原生提供的技术和行为。我们可以把它当作计算机领域的补丁来思考。

这个施展魔法的 polyfill 并为我们提供承诺支持的功能可以从这个链接下载：[`www.promisejs.org/polyfills/promise-4.0.0.js`](https://www.promisejs.org/polyfills/promise-4.0.0.js)。它基本上为尚未原生实现承诺的浏览器添加了承诺支持。它还可以用于为 Node.js 提供承诺支持。以下代码示例展示了如何在我们的代码文件中包含它：

```js
<script src="img/promise-4.0.0.js"></script>
```

我们展示 ECMAScript 6 兼容性表是因为承诺是 ECMAScript 6 规范的一部分，该规范将承诺作为一等语言特性提供，并且实现基于 Promises/A+ 提案。

## 具有 promise-like 功能的库

承诺的概念在网页开发和 JavaScript 的世界中并不新鲜。开发者可能已经在通过库以非标准化方式在 JavaScript 中遇到或使用过承诺。这些库是承诺概念的实现；其中一些是符合规范的实现，并开始采用承诺模式，而许多则不是。此外，其中一些库不符合 Promises/A+ 标准，这在选择在我们的项目中使用哪些 JavaScript 库时是一个非常重要的要求。

### 提示

开发者可以通过使用合规性测试套件来测试他们实现的库和 API 是否符合 Promises/A+标准。

以下是完全符合 Promises/A+规范的一些库，因此我毫不犹豫地推荐它们：

+   **Q.js**：由 Kris Kowal 和 Domenic Denicola 开发，它是一个功能完备的承诺库，包括适用于 Node.js 的适配器和支持进度处理器的支持。您可以从[`github.com/kriskowal/q`](https://github.com/kriskowal/q)下载。

+   **RSVP.js**：由 Yehuda Katz 开发，它具有非常小巧轻量的承诺库。您可以从[`github.com/tildeio/rsvp.js`](https://github.com/tildeio/rsvp.js)下载。

+   **when.js**：由 Brian Cavalier 开发，它是一个中介库，包括管理预期操作集合的功能。它还具有暴露承诺的进度和取消处理器的功能。您可以从[`github.com/cujojs/when`](https://github.com/cujojs/when)下载。

此外，我们还有`then`([`github.com/then`](https://github.com/then))，这是一组简单的 Promises/A+实现库，符合规范并扩展了一些功能，例如在承诺被满足或拒绝时进行进度处理。

另外，著名的 jQuery 有一个称为 Deferred 的 API——位于[`api.jquery.com/jquery.deferred/`](http://api.jquery.com/jquery.deferred/)，声称与承诺相似。直到版本 1.8，jQuery 的 Deferred 没有从`then`返回新的承诺对象，如规范所要求；因此，依赖 jQuery 的开发人员没有得到承诺模式的全功率和能力。此外，使用此实现编写的许多代码与其他确实符合规范的承诺实现不完全兼容。Deferred 不符合 Promise/A+规范，至少不符合规范的第二部分，该部分指出在执行处理器之一时`then`不会返回新的承诺对象。因此，我们无法实现`then`函数的组合和链式调用，以及由于链断裂而导致的错误冒泡，这两个规范中最重要的点。这使得 jQuery 与众不同且某种程度上不那么有用。尽管如此，如果我们需要使用 jQuery 或其他不遵循规范的库暴露的`promise`对象，我们可以使用前面提到的库之一将非符合规范的承诺转换为符合 A+提案的真实承诺。例如，使用 Q，我们可以有以下代码将 jQuery 承诺转换为标准承诺：

```js
var SpecPromise = Q.when($.get("http://example.com/json"));
```

另一个例子是使用承诺多填充库（[`www.promisejs.org/polyfills/promise-4.0.0.js`](https://www.promisejs.org/polyfills/promise-4.0.0.js)），如下代码所示：

```js
var specPromise = Promise.resolve($.ajax(' http://example.com/json););
```

尽管这些承诺实现遵循标准化的行为，但它们的整体 API 存在差异。

# 摘要

正如我们所看到的，承诺（promises）的概念并不非常新，并且在 JavaScript 中已经存在，通过不同的库实现了不同的实现方式，无论是符合标准的还是其他方式。然而，现在，所有这些努力都在 Promises/A+社区规范中得到了总结，大多数库都符合这个规范。因此，我们现在可以通过包含在 ECMAScript 6 下一个版本中的标准`Promise`类，在 JavaScript 中得到对承诺的内置支持，使得网络平台 API 能够为其异步操作返回承诺。此外，我们深入讲解了承诺 API 和`then`方法，并了解了新标准在当前浏览器中的兼容性。最后，我们简要介绍了几个实现承诺并符合 Promises/A+规范的库。

在下一章中，我们将讲解承诺的链式调用以及如何使用`then`方法来实现它，以启用多个异步操作。
