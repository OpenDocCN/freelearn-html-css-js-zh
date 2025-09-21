# 13. TypeScript 中的 Async/Await

概述

`async`/`await` 关键字为开发者提供了一种更简洁的方式来编写异步、非阻塞程序。在本章中，我们将了解这种语法糖，一个用于更简洁和表达性更强的语法的术语，以及它是如何推动现代软件开发。我们将探讨 `async`/`await` 的常见用法，并讨论 TypeScript 中异步编程的格局。到本章结束时，你将能够将 `async/await` 关键字应用于 TypeScript，并使用它们来编写异步程序。

# 简介

上一章让你在 TypeScript 中开始了承诺的学习。虽然承诺提高了我们编写异步代码的能力，避免了嵌套回调的丑陋，但开发者仍然想要一种更好的方式来编写异步代码。承诺语法对于有 C 语言家族背景的程序员来说有时可能具有挑战性，因此提出了将 `async`/`await` 的“语法糖”添加到 ECMAScript 规范的建议。

在本章中，我们将了解新的异步编程范式被引入 ECMAScript 标准的介绍，检查其语法，并查看它们在 TypeScript 中的使用。我们还将介绍（截至编写时）新的顶级 `await` 功能，它允许在 `async` 函数之外进行异步编程。我们还将再次探讨异步编程中的错误处理，并检查使用 `async`/`await` 语法与承诺相比的优缺点。

阅读过上一章的读者会看到，在承诺中仍然有一些嵌套。虽然通过多个承诺比嵌套回调更容易管理流程，但我们仍然没有一种机制可以将控制权返回到顶层。

例如，考虑一个返回承诺的 `getData` 函数。调用此函数的代码将类似于以下内容：

```js
getData().then(data => {
  // do something with the data
});
```

我们没有方法将 `data` 值传播到外部作用域。我们无法在后续作用域中处理该值。一些程序员可能会尝试编写类似这样的代码：

```js
let myData;
getData().then(data => {
  myData = data
});
console.log(myData);
```

这段代码将始终输出 `undefined`。它看起来应该可以工作，但不会，因为承诺回调将在承诺返回之前被调用。这种异步编程可能会令人困惑，并导致许多错误。`async`/`await` 通过允许我们在承诺解决之前暂停代码的执行来解决此问题。我们可以使用 `async`/`await` 语法重写前面的代码：

```js
const myData = await getData();
console.log(myData);
```

我们已经从五行代码缩减到两行。`console.log` 的同步操作将等待承诺解决。代码更加易于理解，我们可以在顶层作用域中存储变量而不需要嵌套。

由于 TypeScript 在大多数情况下被转换为 JavaScript，我们需要确保我们选择正确的目标环境，以便我们的代码能够运行。这个主题将在本章后面更详细地讨论。

# 进化与动机

虽然承诺（promises）在异步编程范式方面取得了显著进展，但人们仍然渴望一种更轻量级的语法，它依赖于更少的显式声明承诺对象。将 `async`/`await` 关键字添加到 ECMAScript 规范中，将允许开发者减少样板代码并使用承诺（promises）。这一概念源自 C# 编程语言，而 C# 又从 F# 编程语言中借鉴了异步工作流的概念。

异步函数允许程序在函数调用尚未返回的情况下继续正常操作。程序不会等待异步函数调用完成，直到找到 `await` 关键字。更重要的是，使用 `await` 不会阻塞事件循环。即使我们暂停了程序的一部分以等待异步函数调用的结果，其他操作仍然可以完成。事件循环不会被阻塞。有关事件循环的更多信息，请参阅第十二章，TypeScript 中承诺的指南。

这些关键字的好处在于它们与承诺（promises）立即兼容。我们可以等待任何承诺（promise），从而避免使用 `then()` API。这种能力意味着，即使在集成较旧的库或模块时，我们也可以使用最新的语法，这与承诺化（promisification）的概念（见第十二章，TypeScript 中承诺的指南）相结合。为了演示这一点，让我们回到前一章的一个例子：

```js
import { promises } from "fs";
promises.readFile('text.txt').then(file => console.log(file.toString()));
```

此示例使用 Node.js 的 `fs`（文件系统）模块中的 `promises` API。该代码从本地文件系统中读取文件并将内容记录到控制台。我们可以使用 `await` 语法来使用此代码：

```js
import { promises } from "fs";
const text = (await promises.readFile('text.txt')).toString();
console.log(text);
```

注意，为了运行此代码，您必须能够使用顶层 `await`，这在撰写本文时需要一些额外的设置。请参阅本章后面的部分。从这个例子中我们可以得出的结论是，即使我们更喜欢 `async`/`await`，我们仍然可以使用 `fs` 模块中的 `promises` API。

# TypeScript 中的 async/await

TypeScript 的维护者从审查过程的第 1 和第 2 阶段开始支持 ECMAScript 功能，但只有在它们达到第 3 阶段时才会正式发布。

TypeScript 从 2015 年 9 月发布的 1.6 版本开始提供对 `async` 函数的实验性支持，并在 2015 年 11 月发布的 1.7 版本中提供完全支持。TypeScript 程序员可以在官方浏览器和 Node.js 支持之前整整一年使用这种语法。

在 TypeScript 中使用 `async`/`await` 关键字与 JavaScript 的使用方式没有太大差异，但我们确实在明确指定哪些函数应该返回承诺（promises）以及哪些应该返回已解析的值或抛出错误方面具有优势。

在 TypeScript 中编写现代语法时，需要注意的一点是，大多数 TypeScript 代码在运行时（如网页浏览器或 Node.js）执行时会被转译为 JavaScript。我们需要理解转译和 polyfill 之间的区别。`async`/`await` 代码会被转译到只支持 promise 语法的环境中。**polyfill** 添加缺失的语言特性。如果我们的目标环境甚至不支持 promises，那么将 async/await 转译为 promises 就无法解决问题。我们还需要一个 polyfill。

## 练习 13.01：转译目标

在这个练习中，我们将使用一个虚构的 "Hello World!" 示例来演示 TypeScript 如何处理 `async` /`await` 关键字的转译：

注意

本练习的代码文件可以在此处找到：[`packt.link/NS8gY`](https://packt.link/NS8gY)。

1.  导航到 `Exercise01` 文件夹，并使用 `npm install` 安装依赖项：

    ```js
    npm install
    ```

1.  这将安装 TypeScript 和 TS Node 执行环境。现在，通过输入 `npx ts-node target.ts` 来执行程序。结果如下：

    ```js
    World!
    Hello
    ```

    在 `Hello` 之前打印了 `World!`。

1.  打开 `target.ts` 并检查导致这种情况的原因。这个程序创建了一个 `sayHello` 函数，该函数内部创建了一个在 1 毫秒后解决的 promise。你可能注意到，即使我们移除了 `await` 关键字，程序仍然会做完全相同的事情。这是可以的。这里有趣的不是 `await` 关键字，而是不同的转译目标。当我们使用 TS Node 运行这个程序时，这将针对我们正在运行的当前 Node.js 版本。假设这是一个较新的版本，`async`/`await` 将被支持。而不是这样做，让我们尝试使用 TypeScript 将代码转译为 JavaScript，看看会发生什么。

1.  现在，打开 `tsconfig.json` 文件并查看它：

    ```js
    {
      "compilerOptions": {
        "target": "es5",
        "module": "commonjs",
        "strict": true,
        "esModuleInterop": true,
        "skipLibCheck": true,
        "forceConsistentCasingInFileNames": true
      }
    }
    ```

1.  将 `target` 选项设置为 `es5` 意味着 TypeScript 将尝试生成符合 ECMAScript5 规范的代码。所以让我们试一试：

    ```js
    npx tsc
    ```

    没有输出意味着它执行成功。

1.  查看由 TypeScript 生成的 `target.js` 文件。这个文件的大小可能因你的 TypeScript 版本而异，但转译后的代码模块可能超过 50 行：

    ```js
    "use strict";
    var __awaiter = (this && this.__awaiter) || function (thisArg, _arguments, P, generator) {
        function adopt(value) { return value instanceof P ? value : new P(function (resolve) { resolve(value); }); }
        return new (P || (P = Promise))(function (resolve, reject) {
    //….
    sayHello();
    console.log('World!');
    ```

    注意

    完整的代码可以在此处找到：[`packt.link/HSmyX`](https://packt.link/HSmyX)。

    我们可以通过在命令提示符中输入 `node target.js` 来执行转译后的代码，我们会看到与之前相同的输出。

    Promises 不是 ECMAScript5 规范的一部分，因此为了生成在 ECMAScript5 环境中可以工作的代码，转译器必须创建 `__awaiter` 和 `__generator` 函数以支持类似 promise 的功能。

1.  让我们把目标切换到 es6。打开 `tsconfig.json` 并将目标属性更改为 `es6`：

    ```js
    {
      "compilerOptions": {
        "target": "es6",
        "module": "commonjs",
        "strict": true,
        "esModuleInterop": true,
        "skipLibCheck": true,
        "forceConsistentCasingInFileNames": true
      }
    }
    ```

1.  使用 `node target.js` 调用函数，我们得到与之前完全相同的输出。现在让我们看看 TypeScript 在转译我们的源代码时做了什么：

    ```js
    "use strict";
    var __awaiter = (this && this.__awaiter) || function (thisArg, _arguments, P, generator) {
        function adopt(value) { return value instanceof P ? value : new P(function (resolve) { resolve(value); }); }
        return new (P || (P = Promise))(function (resolve, reject) {
            function fulfilled(value) { try { step(generator.next(value)); } catch (e) { reject(e); } }
            function rejected(value) { try { step(generator"throw"); } catch (e) { reject(e); } }
            function step(result) { result.done ? resolve(result.value) : adopt(result.value).then(fulfilled, rejected); }
            step((generator = generator.apply(thisArg, _arguments || [])).next());
        });
    };
    const sayHello = () => __awaiter(void 0, void 0, void 0, function* () {
        yield new Promise((resolve) => setTimeout(() => resolve(console.log('Hello')), 1));
    });
    sayHello();
    console.log('World!');
    ```

    转译后的代码现在是 15 行，而不是超过 50 行，因为 ECMAScript6 比 es5 更接近支持我们需要的所有功能。`async`/`await` 关键字在 ECMAScript6 中不受支持，但 promises 是支持的，所以 TypeScript 正在利用 promises 来使输出的代码更加简洁。

1.  现在，让我们将目标改为 `esnext`，再次运行 `npx tsc`，看看输出结果是什么：

    ```js
    "use strict";
    const sayHello = async () => {
        await new Promise((resolve) => setTimeout(() => resolve(console.log('Hello')), 1));
    };
    sayHello();
    console.log('World!');
    ```

    这非常类似于我们的源代码！由于 `async`/`await` 在最新的 ECMAScript 规范中是支持的，因此不需要转换。

1.  旧版本的 TypeScript 没有完全 polyfill promises 和 async/await。使用 `npm i -D typescript@2` 降级 TypeScript 版本，将编译目标恢复到 es5，然后尝试转译：

    ```js
    npx tsc
    target.ts:1:18 - error TS2705: An async function or method in ES5/ES3 requires the 'Promise' constructor.  Make sure you have a declaration for the 'Promise' constructor or include 'ES2015' in your `--lib` option.
    1 const sayHello = async () => {
                       ~~~~~~~~~~~~~
    target.ts:2:13 - error TS2693: 'Promise' only refers to a type, but is being used as a value here.
    2   await new Promise((resolve) =>
                  ~~~~~~~
    target.ts:2:22 - error TS7006: Parameter 'resolve' implicitly has an 'any' type.
    2   await new Promise((resolve) =>
    ```

    这不起作用。

1.  如果你将目标提升到 `es6`，它仍然会失败：

    ```js
    % npx tsc
    target.ts:3:30 - error TS2345: Argument of type 'void' is not assignable to parameter of type '{} | PromiseLike<{}> | undefined'.
    3     setTimeout(() => resolve(console.log('Hello')))
    ```

1.  使用 `npm i -D typescript@latest` 安装 TypeScript 的最新版本，然后一切应该和以前一样工作。

对于初学者来说，TypeScript 的这个方面可能会令人困惑。TypeScript 不会为缺失的 promises 提供 polyfill，但它会提供对功能等效的语法的转换。

## 选择目标

那么，我们如何选择编译目标呢？通常情况下，使用 ES2017 或更高版本是安全的，除非你需要支持过时的浏览器，例如 Internet Explorer，或者废弃的 Node.js 版本。有时，由于客户需求，我们别无选择，只能支持过时的浏览器，但如果我们对 Node.js 运行时环境有任何控制权，建议更新到当前支持的版本。这样做应该允许我们使用最新的 TypeScript 功能。

# 语法

这两个新关键字 `async`/`await` 通常会一起出现，但并不总是如此。让我们分别看看每个关键字的语法。

## async

`async` 关键字修改了一个函数。如果使用函数声明或函数表达式，它放在 `function` 关键字之前。如果使用箭头函数，`async` 关键字放在参数列表之前。给函数添加 `async` 关键字将导致函数返回一个 promise。

例如：

```js
function addAsync(num1: number, num2: number) {
  return num1 + num2;
}
```

只需在这个简单函数中添加 `async` 关键字，这个函数就会返回一个 promise，现在它是可等待的并且是 thenable 的。由于函数中没有异步操作，这个 promise 将立即解析。

这个箭头函数版本可以写成如下：

```js
const addAsync = async (num1: number, num2: number) => num1 + num2;
```

## 练习 13.02：`async` 关键字

这个练习说明了给函数添加 `async` 关键字是如何使其返回一个 promise 的：

注意

这个练习的代码文件可以在以下位置找到：[`packt.link/BgujE`](https://packt.link/BgujE)。

1.  检查 `async.ts` 文件：

    ```js
    export const fn = async () => {
      return 'A Promise';
    };
    const result = fn();
    console.log(result);
    ```

    你可能期望这个程序输出 `A Promise`，但让我们看看当我们运行它时实际上会发生什么：

    ```js
    npx ts-node async.ts
    Promise { 'A Promise' }
    ```

1.  `async` 关键字将响应封装在一个 promise 中。我们可以通过移除该关键字并再次运行程序来确认这一点：

    ```js
    npx ts-node async.ts
    A Promise
    ```

1.  使用 `async` 修改我们的函数与将其包裹在一个承诺中完全等价。如果我们想使用承诺语法，我们可以将程序写成这样：

    ```js
    export const fn = () => {
      return Promise.resolve('A Promise');
    };
    const result = fn();
    console.log(result);
    ```

1.  再次，以这种方式编写的程序将输出未解决的承诺：

    ```js
    npx ts-node async.ts
    Promise { 'A Promise' }
    ```

由于我们使用 TypeScript 并且返回类型可以被推断，使用 `async` 修改函数可以保证 TypeScript 总是将其视为返回一个承诺。

`async` 关键字使得它修改的函数被包裹在一个承诺中。你选择通过声明一个承诺或使用 `async` 关键字来显式地这样做，通常是一个品味和风格的问题。

我们如何解决一个 `async` 函数？我们稍后会谈到 `await`，但使用 `then` 和我们在 *第十二章*，*TypeScript 中承诺指南*中学到的承诺链式操作呢？是的，这也是可能的。

## 练习 13.03：使用 then 解决 async 函数

本练习将教你如何使用 `then` 解决一个 `async` 函数：

注意

本练习的代码文件可以在这里找到：[`packt.link/4Bo4c`](https://packt.link/4Bo4c)。

1.  创建一个名为 `resolve.async.ts` 的新文件，并输入以下代码：

    ```js
    export const fn = async () => {
      return 'A Promise';
    };
    const result = fn();
    result.then((message) => console.log(message));
    ```

1.  通过在控制台中输入 `npx ts-node resolve.async.ts` 执行此代码，你会看到预期的文本消息被记录，而不是未解决的承诺：

    ```js
    A Promise
    ```

尽管我们从未显式地声明承诺对象，但使用 `async` 确保我们的函数始终返回一个承诺。

## await

这个组合的第二部分可能更有价值。`await` 关键字将尝试解决任何承诺，然后再继续。这将使我们摆脱 `then` 链式操作，并允许我们编写看起来是同步的代码。使用 `await` 的一个巨大好处是，如果我们想将异步调用的结果分配给某个值，然后对这个值进行一些操作。让我们看看在承诺中是如何做到这一点的：

```js
asyncFunc().then(result => {
  // do something with the result
});
```

这可以正常工作，实际上，这种语法被广泛使用，但如果我们需要进行一些复杂的链式操作，它可能会有些问题：

```js
asyncFuncOne().then(resultOne => {
  asyncFuncTwo(resultOne).then(resultTwo => {
    asyncFuncThree(resultTwo).then(resultThree => {
      // do something with resultThree
    });
  });
});
```

但等等。我以为承诺是用来消除回调地狱的！实际上，对于这种链式操作来说，这并不理想。让我们尝试使用 `await` 代替：

```js
const resultOne = await asyncFuncOne();
const resultTwo = await asyncFuncTwo(resultOne);
const resultThree = await asyncFuncThree(resultTwo);
// do something with resultThree
```

大多数程序员都会同意这种语法更干净，实际上，这也是为什么在语言中添加 `async`/`await` 的主要原因之一。

## 练习 13.04：await 关键字

本练习将展示如何使用 `await` 解决一个承诺：

注意

本练习的代码文件可以在这里找到：[`packt.link/mUzGI`](https://packt.link/mUzGI)。

1.  创建一个名为 `await.ts` 的文件，并输入以下代码：

    ```js
    export const fn = async () => {
      return 'A Promise';
    };
    const resolveIt = async () => {
      const result = await fn();
      console.log(result);
    };
    resolveIt();
    ```

    在这里我们声明了两个 `async` 函数。其中一个使用 `await` 调用另一个，以解决承诺，并且应该打印出字符串，而不是未解决的承诺。

1.  使用 `npx` `ts-node` `await.ts` 运行文件，你应该会看到以下输出：

    ```js
    A Promise
    ```

为什么我们需要在第二个函数中包装 `await`？这是因为通常，`await` 不能在 `async` 函数之外使用。我们将在本章后面讨论顶级 `await` 功能，这是这个规则的例外。关于将 `await` 与承诺混合呢？这当然可以做到。

## 练习 13.05：等待承诺

这个练习教你如何使用 `await` 与承诺：

注意

这个练习的代码文件可以在以下位置找到：[`packt.link/mMDiw`](https://packt.link/mMDiw)。

1.  创建一个名为 `await-promise.ts` 的新文件，并输入以下代码：

    ```js
    export const resolveIt = async () => {
      const result = await Promise.resolve('A Promise');
      console.log(result);
    };
    resolveIt();
    ```

1.  通过输入 `npx ts-node await-promise.ts` 执行代码，你会看到文本输出：

    ```js
    A Promise
    ```

1.  使用更明确的承诺声明来编写相同代码的另一种更长的写法是：

    ```js
    export const resolveIt = async () => {
      const p = new Promise((resolve) => resolve('A Promise'));
      const result = await p;
      console.log(result);
    };
    resolveIt();
    ```

    这段代码的功能完全相同：

1.  输入 `npx ts-node src/await-promise.ts` 以验证你是否得到以下输出：

    ```js
    A Promise
    ```

## 语法糖

之前关于 `async` 函数和承诺的练习只是两种不同的方式，在 TypeScript 中表达完全相同的操作。同样，使用 `await` 和使用 `then` 解决承诺是等效的。`async`/`await` 关键字被称为“语法糖”，或者代码结构，它允许更具有表现力的语法，而不改变程序的行为。

这意味着有可能，有时甚至建议将 `async`/`await` 语法与承诺混合。这样做的一个非常常见的原因是，你正在使用一个编写为使用承诺的库，但你更喜欢 `async`/`await` 语法。混合两种语法的另一个原因可能是更明确地处理异常。我们将在本章后面详细讨论异常处理。

# 异常处理

我们已经讨论了如何将 `then` 链转换为 `await`，但关于 `catch` 呢？如果一个承诺被拒绝，错误将会冒泡并必须以某种方式捕获。在 `async`/`await` 世界中未能捕获异常与未能捕获承诺拒绝一样有害。事实上，它们是完全相同的，`async`/`await` 只是承诺之上的语法糖。

未处理被拒绝的承诺可能导致系统故障，其中在网页浏览器中运行的程序崩溃，导致空白页面或功能损坏，从而驱使用户远离你的网站。在服务器端未能处理被拒绝的承诺可能会导致 Node.js 进程退出和服务器崩溃。即使你有自我修复系统尝试将服务器恢复在线，你试图完成的任何工作都将失败，频繁的重启会使你的基础设施运行成本更高。

处理这些错误的最直接方法是使用 `try` 和 `catch` 块。这种语法不仅对 `async`/`await` 是独特的，自 ECMAScript3 以来一直是 ECMAScript 规范的一部分。它非常简单直接，易于使用：

```js
try {
  await someAsync();
} catch (e) {
  console.error(e);
}
```

就像您可以从多个链式承诺中捕获抛出的错误一样，您也可以在这里实现类似的模式：

```js
try {
  await someAsync();
  await anotherAsync();
  await oneMoreAsync();
} catch (e) {
  console.error(e);
}
```

可能会有需要更精细异常处理的情况。这些结构可以嵌套：

```js
try {
  await someAsync();
  try {
    await anotherAsync();
  } catch (e) {
    // specific handling of this error
  }
  await oneMoreAsync();
} catch (e) {
  console.error(e);
}
```

然而，编写这样的代码会抵消 `async`/`await` 语法的大部分好处。更好的解决方案是抛出特定的错误消息并对其进行测试：

```js
try {
  await someAsync();
  await anotherAsync();
  await oneMoreAsync();
} catch (e) {
  if(e instanceOf MyCustomError) {
    // some custom handling
  } else {
    console.error(e);
  }
}
```

使用这种技术，我们可以将所有内容都在同一个块中处理，避免嵌套和看起来杂乱的代码结构。

## 练习 13.06：异常处理

让我们看看我们如何在简单示例中实现错误处理。在这个练习中，我们将故意并明确地从 `async` 函数中抛出一个错误，并看看它是如何实现我们的程序操作的：

注意

这个练习的代码文件可以在以下位置找到：[`packt.link/wbA8E`](https://packt.link/wbA8E)。

1.  首先创建一个名为 `error.ts` 的新文件，并输入以下代码：

    ```js
    export const errorFn = async () => {
      throw new Error('An error has occurred!');
    };
    const asyncFn = async () => {
      await errorFn();
    };
    asyncFn();
    ```

1.  当然，这个程序总是会抛出错误。当我们通过在控制台中输入 `npx ts-node error.ts` 来执行它时，我们可以清楚地看到错误没有被正确处理：

    ```js
    (node:29053) UnhandledPromiseRejectionWarning: Error: An error has occurred!
        at Object.exports.errorFn (/workshop/async-chapter/src/error.ts:2:9)
        at asyncFn (/workshop/async-chapter/src/error.ts:6:9)
        at Object.<anonymous> (/workshop/async-chapter/src/error.ts:9:1)
        at Module._compile (internal/modules/cjs/loader.js:1138:30)
        at Module.m._compile (/workshop/async-chapter/node_modules/ts-node/src/index.ts:858:23)
        at Module._extensions..js (internal/modules/cjs/loader.js:1158:10)
        at Object.require.extensions.<computed> [as .ts] (/workshop/async-chapter/node_modules/ts-node/src/index.ts:861:12)
        at Module.load (internal/modules/cjs/loader.js:986:32)
        at Function.Module._load (internal/modules/cjs/loader.js:879:14)
        at Function.executeUserEntryPoint [as runMain] (internal/modules/run_main.js:71:12)
    (node:29053) UnhandledPromiseRejectionWarning: Unhandled promise rejection. This error originated either by throwing inside of an async function without a catch block, or by rejecting a promise which was not handled with .catch(). To terminate the node process on unhandled promise rejection, use the CLI flag `--unhandled-rejections=strict` (see https://nodejs.org/api/cli.html#cli_unhandled_rejections_mode). (rejection id: 2)
    (node:29053) [DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process with a non-zero exit code.
    ```

    注意到弃用警告。这不仅是一个难看的堆栈跟踪，在未来的某个时刻，这样的异常将导致 Node.js 进程退出。我们显然需要处理这个异常！

1.  幸运的是，我们可以通过简单地用 `try` 和 `catch` 包围调用来实现这一点：

    ```js
    export const errorFn = async () => {
      throw new Error('An error has occurred!');
    };
    const asyncFn = async () => {
      try {
        await errorFn();
      } catch (e) {
        console.error(e);
      }
    };
    asyncFn();
    ```

1.  现在，当我们执行程序时，我们得到一个更有序的异常和堆栈跟踪记录：

    ```js
    Error: An error has occurred!
        at Object.exports.errorFn (/workshop/async-chapter/src/error.ts:2:9)
        at asyncFn (/workshop/async-chapter/src/error.ts:7:11)
        at Object.<anonymous> (/workshop/async-chapter/src/error.ts:13:1)
        at Module._compile (internal/modules/cjs/loader.js:1138:30)
        at Module.m._compile (/workshop/node_modules/ts-node/src/index.ts:858:23)
        at Module._extensions..js (internal/modules/cjs/loader.js:1158:10)
        at Object.require.extensions.<computed> [as .ts] (/workshop/node_modules/ts-node/src/index.ts:861:12)
        at Module.load (internal/modules/cjs/loader.js:986:32)
        at Function.Module._load (internal/modules/cjs/loader.js:879:14)
        at Function.executeUserEntryPoint [as runMain] (internal/modules/run_main.js:71:12)
    ```

    当然，这个消息只出现是因为我们明确地记录了它。我们也可以选择抛出一个默认值或执行其他一些操作而不是记录错误。

1.  如果系统行为不正确，记录一个错误总是一个好主意，但根据您的系统要求，您可能更愿意写一些像这样的事情：

    ```js
    const primaryFn = async () => {
      throw new Error('Primary System Offline!');
    };
    const secondaryFn = async () => {
      console.log('Aye aye!');
    };
    const asyncFn = async () => {
      try {
        await primaryFn();
      } catch (e) {
        console.warn(e);
        secondaryFn();
      }
    };
    asyncFn();
    ```

    在这种情况下，我们只是抛出一个警告并回退到二级系统，因为这个程序被设计成容错的。仍然记录这个警告以便我们可以追踪我们的系统行为是个好主意。现在就先这样变体一次。

1.  让我们把我们的 `try` 和 `catch` 块放在顶层，并像这样重写我们的程序：

    ```js
    export const errorFN = async () => {
      throw new Error('An error has occurred!');
    };
    const asyncFn = async () => {
      await errorFN();
    };
    try {
      asyncFn();
    } catch (e) {
      console.error(e);
    }
    ```

1.  这是您得到的输出：

    ```js
    Error: Primary System Offline!
        at primaryFn (C:\Users\Mahesh\Documents\Chapter13_TypeScript\Exercise13.06\error-secondary.ts:2:9)
        at asyncFn (C:\Users\Mahesh\Documents\Chapter13_TypeScript\Exercise13.06\error-secondary.ts:11:11)
        at Object.<anonymous> (C:\Users\Mahesh\Documents\Chapter13_TypeScript\Exercise13.06\error-secondary.ts:18:1)
        at Module._compile (internal/modules/cjs/loader.js:1063:30)
        at Module.m._compile (C:\Users\Mahesh\AppData\Roaming\npm-cache\_npx\13000\node_modules\ts-node\src\index.ts:1056:23)       
        at Module._extensions..js (internal/modules/cjs/loader.js:1092:10)
        at Object.require.extensions.<computed> [as .ts] (C:\Users\Mahesh\AppData\Roaming\npm-cache\_npx\13000\node_modules\ts-node\src\index.ts:1059:12)
        at Module.load (internal/modules/cjs/loader.js:928:32)
        at Function.Module._load (internal/modules/cjs/loader.js:769:14)
        at Function.executeUserEntryPoint [as runMain] (internal/modules/run_main.js:72:12)
    Aye aye!
    ```

    您可以假设程序可能的工作方式与在 `asyncFn` 内部放置 `try` 和 `catch` 相同，但实际上，它将表现得与没有任何错误处理一样。这是因为我们没有在 `try` 块内等待函数。

# 顶层 await

顶层 `await` 是一个允许在模块级别（任何函数之外）使用 `await` 关键字的功能。这允许一些有趣的模式，例如在尝试使用它之前，通过调用异步函数等待依赖项完全加载。总有一天，顶层 `await` 可能会支持一些非常令人兴奋的函数式编程范式，但在写作时，它仍然处于技术预览模式，因此尚未准备好广泛使用。您可能是在顶层 `await` 广泛可用和支持的时候阅读这本书，如果是这样，您绝对应该看看它！

使用顶层 `await` 编写代码非常简单。以下是一个非常简短的程序，它试图利用它：

```js
export const fn = async () => {
  return 'awaited!';
};
console.log(await fn());
```

这看起来没问题。现在让我们看看当我们尝试执行它时会发生什么：

```js
⨯ Unable to compile TypeScript:
src/top-level-await.ts:5:13 - error TS1378: Top-level 'await' expressions are only allowed when the 'module' option is set to 'esnext' or 'system', and the 'target' option is set to 'es2017' or higher.
5 console.log(await fn());
              ~~~~~
```

这不被支持，但它给了我一些提示。我们如何才能让它工作？

顶层 `await` 需要 NodeJS 14.8 或更高版本。这个版本的 NodeJS 在 2020 年 10 月进入了 LTS（长期服务）阶段，因此在写作时仍然很新。您可以在命令行中使用 `node -v` 检查您的 NodeJS 版本。如果您没有运行 14.8 或更高版本，有一些很好的工具如 `nvm` 和 `n` 可以让您轻松切换版本。

然而，这并没有解决问题。看起来我需要将我的 `tsconfig.json` 的 `target` 属性更改为 `es2017` 或更高版本，并将 `module` 属性设置为 `esnext`。添加 `module` 属性意味着我想使用 ES 模块，这是一种相对较新的处理模块的方式，超出了本书的范围。为了启用 ES 模块，我需要在 `package.json` 文件中设置 `type` 属性为 `module`。

现在，我已经更新了几个 JSON 文件，准备再次尝试：

```js
TypeError [ERR_UNKNOWN_FILE_EXTENSION]: Unknown file extension ".ts" for /workshop/async-chapter/src/top-level-await.ts
    at Loader.defaultGetFormat [as _getFormat] (internal/modules/esm/get_format.js:65:15)
    at Loader.getFormat (internal/modules/esm/loader.js:113:42)
    at Loader.getModuleJob (internal/modules/esm/loader.js:243:31)
    at Loader.import (internal/modules/esm/loader.js:177:17)
```

这仍然不起作用。我还需要做一件事才能让它工作，那就是在 Node.js 中启用实验性功能，并指导 TS Node 允许 **ES 模块**（**esm**）。这需要更长的命令：

```js
node --loader ts-node/esm.mjs top.ts
(node:91445) ExperimentalWarning: --experimental-loader is an experimental feature. This feature could change at any time
(Use `node --trace-warnings ...` to show where the warning was created)
awaited! 
```

但它确实工作了。顶层 `await` 很可能在接下来的几个月和几年中变得更加容易和直观，所以请确保检查你运行时的最新文档。

# 承诺方法

除了承诺（promises）暴露的常规 `next` 和 `catch` 方法之外，还有一些其他便利方法，例如 `all`、`allSettled`、`any` 和 `race`，这些方法使得与承诺（promises）一起工作更加愉快。它们如何在 `async`/`await` 世界中使用？实际上，它们可以非常和谐地一起工作。例如，这里是一个使用 `Promise.all` 的例子，它使用了 `then` 和 `catch`。给定三个承诺，`p1`、`p2` 和 `p3`：

```js
Promise.all([p1, p2, p3])
  .then(values => console.log(values))
  .catch(e => console.error(e));
```

没有任何 `awaitAll` 操作符，所以如果我们想并行执行我们的承诺，我们仍然需要使用 `Promise.all`，但如果我们选择的话，可以避免链式使用 `then` 和 `catch`：

```js
try {
  const values = await Promise.all([p1, p2, p3]);
  console.log(values);
} catch (e) {
  console.error(e);
}
```

在这种情况下，我们可能会觉得添加 `await` 并没有提高代码的质量，因为我们实际上已经将代码从三行扩展到了六行。有些人可能会觉得这种形式更易于阅读。像往常一样，这取决于个人或团队的偏好。

## 练习 13.07：Express.js 中的 async/await

在这个练习中，我们将使用流行的 Express 框架构建一个小型 Web 应用程序。虽然 Express 是为 JavaScript 语言编写的，但已经为其发布了类型定义，因此它可以完全与 TypeScript 一起使用。Express 是一个无意见、极简主义的 Web 应用程序构建框架。它是目前使用最久和最受欢迎的框架之一。

对于我们的简单应用程序，我们将在端口 `8888` 上启动一个 Web 服务器并接受 `GET` 请求。如果该请求的查询字符串中有一个 `name` 参数，我们将将其记录在名为 `names.txt` 的文件中。然后我们将问候用户。如果没有查询字符串中的名称，我们将不记录任何内容并打印出 `Hello World!`：

注意

这个练习的代码文件可以在以下链接找到：[`packt.link/cG4r8`](https://packt.link/cG4r8)。

让我们从安装 Express 框架和类型定义开始吧。

1.  输入 `npm i express` 安装 Express 作为依赖项，并输入 `npm i -D @types/express @types/node` 安装我们需要的类型定义。

    记住 `-D` 标志表示这是一个 `devDependency`，它可以与生产依赖项不同地管理，尽管它的使用是可选的。

1.  在安装了依赖项之后，让我们创建一个名为 `express.ts` 的文件。首先，我们需要导入 `express`，创建应用程序，添加一个简单的处理程序，并监听端口 `8888`：

    ```js
    import express, { Request, Response } from 'express';
    const app = express();
    app.get('/', (req: Request, res: Response) => {
      res.send('OK');
    });
    app.listen(8888);
    ```

    这看起来非常像你的标准 Express 入门应用程序，除了我们为 `Request` 和 `Response` 对象提供了类型。这已经非常有用了，因为我们能够使用 IntelliSense 并确定可以调用这些对象上的哪些方法，而无需查找它们。

    我们的要求是，我们需要监听查询字符串中的 `name` 参数。我们可能会看到一个看起来像 `http://localhost:8888/?name=Matt` 的请求，对此我们应该响应 `Hello Matt!`。

    查询字符串位于 `Request` 对象中。如果我们深入研究类型定义，它被标记如下：

    ```js
    interface ParsedQs { [key: string]: undefined | string | string[] | ParsedQs | ParsedQs[] }
    ```

    这基本上意味着它是一个键/值对的哈希表和嵌套键/值对的哈希表。在我们的情况下，我们预计会看到一个查询对象，其外观类似于 `{ name: 'Matt' }`。因此，我们可以通过使用 `const { name } = req.query;` 来获取 `name` 属性。然后我们可以用类似 `res.send(`Hello ${name ?? 'World'}!`);` 的方式响应请求。在这种情况下，我们使用了空值合并运算符 (`??`) 来表示如果 `name` 变量具有空值（null 或 undefined），我们将回退到 `World` 字符串。我们也可以使用回退或逻辑 OR 运算符 `||`。

1.  更新后的代码现在看起来是这样的：

    ```js
    import express, { Request, Response } from 'express';
    const app = express();
    app.get('/', (req: Request, res: Response) => {
      const { name } = req.query;
      res.send(`Hello ${name ?? 'World'}!`);
    });
    app.listen(8888);
    ```

1.  还有一个要求尚未满足。如果存在，我们需要将名称记录到文件中。为此，我们需要使用 Node.js 的 `fs` 库。我们还将使用 `path` 库来解析要写入的文件的路径。首先，添加新的导入：

    ```js
    import { promises } from 'fs';
    import { resolve } from 'path';
    ```

1.  现在，我们将使用 `fs` 的 `promises` API 以异步方式将内容写入我们的日志文件。由于这是一个日志，我们希望每次请求时都追加到它，而不是覆盖它。我们将使用 `appendFile` 并写入名称以及换行符。我们希望在返回之前重复此操作：

    ```js
      if (name) {
        await promises.appendFile(resolve(__dirname, 'names.txt'), `${name}\n`);
      }
    ```

    这几乎就完成了，但现在我们应该已经意识到我们的处理函数没有正确地使用异步。我们只需要将其添加到 `async` 关键字。

1.  完成的代码如下所示：

    ```js
    import express, { Request, Response } from 'express';
    import { promises } from 'fs';
    import { resolve } from 'path';
    const app = express();
    app.get('/', async (req: Request, res: Response) => {
      const { name } = req.query;
      if (name) {
        await promises.appendFile(resolve(__dirname, 'names.txt'), `${name}\n`);
      }
      res.send(`Hello ${name ?? 'World'}!`);
    });
    app.listen(8888);
    ```

1.  使用 `npx ts-node express.ts` 运行程序，并尝试多次点击 `http://localhost:8888?name=your_name` 的 URL。尝试使用不同的名称点击该 URL，并观察你的日志文件递增。以下是一些示例。

1.  以下是为 your_name 生成的浏览器输出：![图 13.1：名称为 your_name 的浏览器消息    ![图 B14508_13_01.jpg](img/B14508_13_01.jpg)

    图 13.1：名称为 your_name 的浏览器消息

1.  以下是为 Matt 生成的浏览器输出：![图 13.2：名称为 Matt 的浏览器消息    ![图 B14508_13_02.jpg](img/B14508_13_02.jpg)

    图 13.2：名称为 Matt 的浏览器消息

1.  以下是为阿尔伯特·爱因斯坦生成的浏览器输出：

![图 13.3：名称为 Albert Einstein 的浏览器消息![图 B14508_13_03.jpg](img/B14508_13_03.jpg)

图 13.3：名称为 Albert Einstein 的浏览器消息

`names.txt` 文件将按以下方式递增：

![图 13.4：日志文件![图 B14508_13_04.jpg](img/B14508_13_04.jpg)

图 13.4：日志文件

## 练习 13.08：NestJS

与 Express 相比，NestJS 是一个高度意见化且功能齐全的框架，用于构建 TypeScript 应用程序。NestJS 可以快速启动应用程序。它提供了中间件、GraphQL 和 Websockets 的开箱即用支持。它附带 ESLint、依赖注入框架、测试框架以及许多其他有用的东西。一些开发者非常喜欢使用这样一个功能齐全的框架，而另一些开发者则觉得所有样板代码都令人压抑，更愿意使用更裸骨的工具，如 Express：

注意

该练习的代码文件可以在以下位置找到：[`packt.link/blRq3`](https://packt.link/blRq3)。

让我们启动一个新的 NestJS 应用程序，并对其进行更详细的了解。

1.  NestJS 应用程序可以通过 `npm` 生成。全局安装该包：

    ```js
    npm i -g @nestjs/cli
    ```

1.  当我们使用 CLI 时，它会在我们输入命令的目录内创建一个新的目录来生成项目，因此你可能想要将目录更改为你存储项目的地方。然后，生成项目：

    ```js
    nest new async-nest
    ```

    这里项目被命名为 `async-nest`。你可以将其命名为其他名称。NestJS 将自动安装所有依赖项并启动一个裸骨应用。

1.  切换到你的新应用程序目录，并开始查看代码。如果你打开 `main.ts`，你会看到已经使用了 `async`/`await`。该模块看起来可能像这样：

    ```js
    import { NestFactory } from '@nestjs/core';
    import { AppModule } from './app.module';
    async function bootstrap() {
      const app = await NestFactory.create(AppModule);
      await app.listen(3000);
    }
    bootstrap();
    ```

    NestJS 是建立在 Express 之上的。这段代码将创建一个新的 Express 应用程序。Express 的内部结构在编写 NestJS 代码时不会暴露给你，但如果你需要 NestJS 不支持的功能，你始终可以选择降级到它们。

    让我们回顾一些你可以立即开始使用的有用命令。如果你输入 `npm test`（或 `npm t`），Jest 框架将启动一个测试运行。这个测试将启动你的应用程序的一个实例，调用它，并在验证预期的响应被接收后关闭它。NestJS 随带提供了一些固定值，允许测试你的应用程序的轻量级版本。

    在你开发应用程序的过程中，继续添加单元和集成测试是一个很好的主意。TypeScript 可以帮助你确保代码的正确性，但只有测试才能保证你的应用程序按预期运行。

    另一个有用的命令是 `npm run lint`。这将检查你的代码风格，并通过使用流行的 ESLint 库通知你任何与之相关的问题。

1.  最后，你可以输入 `npm run start:dev` 来以监视模式运行开发服务器，这意味着每次你更改文件时，服务器都会重新启动。

1.  现在尝试运行它，并导航到 `http://localhost:3000`，你将看到`Hello World`消息。如果你打开名为 `app.service.ts` 的文件并更改那里返回的消息，你只需刷新浏览器，就应该能看到消息已更改。

    现在我们已经看到了在两个非常不同的框架中完成的简单的 Hello World 应用程序，让我们添加与在 *练习 13.07：Express.js 中的 async/await* 中所做的相同的问候和日志记录功能。

1.  为了根据查询参数添加自定义问候语，让我们打开两个文件，`app.controller.ts` 和 `app.service.ts`。注意 `app.service.ts` 实现了一个返回字符串 "Hello World!" 的 `getHello` 函数。我们需要将这个函数更改为接受一个 `name` 参数。

1.  将 `name` 参数以 `string` 类型添加到函数的参数列表中，然后将返回值更改为字符串模板并说“Hello”。你将得到类似这样的结果：

    ```js
    export class AppService {
      getHello(name: string): string {
        return `Hello ${name}!`;
      }
    }
    ```

    这是一个简单的重构。如果我们检查 `app.controller.ts`，我们会看到我们的 IDE 现在告诉我们 `getHello` 需要一个参数，我们还没有完成。

    在 Express 应用程序中，我们在内置的 `Request` 对象上找到了我们的查询参数。你同样可以在 NestJS 中做同样的事情，但更常见且更受欢迎的做法是使用装饰器。装饰器是特殊的函数，它们封装其他函数。它们有时被称为高阶函数，类似于 Java 等语言的一些特性。

    我们想要使用的装饰器是 `@Query`，它接受一个参数，即查询参数的名称，然后将该参数绑定到我们的函数参数之一。

1.  我们可以从 `@nestjs/common` 中导入那个装饰器。然后我们将函数参数添加到 `getHello` 中，并将其传递给服务调用。还有一件事值得注意，那就是设置一个默认值，这样我们就可以保持向后兼容性，并且在我们未能提供参数时不会打印出 `Hello undefined`。添加默认值可能会提示你不再需要类型注解，因为它可以从默认类型中轻易推断出来。如果你喜欢，可以将其删除：

    ```js
    import { Controller, Get, Query } from '@nestjs/common';
    import { AppService } from './app.service';
    @Controller()
    export class AppController {
      constructor(private readonly appService: AppService) {}
      @Get()
      getHello(@Query('name') name: string = 'World'): string {
        return this.appService.getHello(name);
      }
    }
    ```

1.  开发服务器应该会重新启动，现在，如果我们浏览到 `http://localhost:3000/?name=Matt`，我们会看到 `Hello Matt!`：![图 13.5：name = Matt 的浏览器消息

    ![图片 B14508_13_05.jpg]

    ![图 13.5：name = Matt 的浏览器消息

1.  现在，让我们添加我们在 Express 中实现的相同日志功能。

    在一个完整规模的应用程序中，我们可能想要构建一个单独的日志服务类。为了我们的目的，我们可以将其实现为一个单独的 `async` 方法。将方法添加到 `app.service.ts` 中，并从 `getHello` 中使用 `await` 调用它。测试以确保它正确工作。

    这里有几个需要注意的问题。一个是 NestJS 会自动将代码从名为 `dist` 的文件夹中转换并提供服务，所以一旦你开始记录名称，你会在那里找到你的 `names.txt` 文件。但更大的技巧是，为了等待记录，我们需要将 `app.service.ts` 中的 `getHello` 方法改为 `async` 方法。这反过来又意味着 `app.controller.ts` 中的 `getHello` 也必须是 `async`。将这些方法改为 `async` 会对我们应用程序产生什么影响？没有！NestJS 已经知道如何在返回请求之前解决承诺。

1.  在这个练习中，还有一件事需要检查，那就是单元测试。由于我们已为 `name` 属性设置了一个默认值，测试应该仍然有效，对吧？实际上，它不起作用。尝试运行 `npm test`，你会看到问题。问题是测试没有期望 `getHello` 是异步的。没关系。我们可以通过使测试回调 `async` 来修复它，如下所示：

    ```js
      describe('root', () => {
        it('should return "Hello World!"', async () => {
          expect(await appController.getHello()).toBe('Hello World!');
        });
      });
    ```

    测试现在应该通过。尝试添加另一个带有参数的测试。

## 练习 13.09：TypeORM

TypeORM 是一个用 TypeScript 编写并针对 TypeScript 设计的对象关系映射器。TypeORM 支持许多流行的数据库，例如 MySQL、Postgres、SQL Server、SQLite，甚至 MongoDB 和 Oracle。TypeORM 经常用于 NestJS 应用程序中，因此在这个练习中，我们将添加一个本地的内存 SQLite 数据库来与我们的 NestJS 应用程序一起工作。

在这个练习中，你将构建另一个 REST 服务，帮助我们跟踪我们做出的承诺。由于 `Promise` 是 TypeScript 中内置对象的名称，让我们使用“pledge”这个术语，这样我们就可以区分领域概念和语言抽象：

注意

这个练习的代码文件可以在以下位置找到：[`packt.link/ZywYh`](https://packt.link/ZywYh)。

1.  要开始，让我们启动一个新的 NestJS 项目：

    ```js
    nest new typeorm-nest
    ```

1.  NestJS 有一个强大的模块系统，允许我们将应用程序的不同功能区域构建成连贯的块。让我们为承诺创建一个新的模块：

    ```js
    nest g module pledge
    ```

    这个命令将在 `/pledge` 子目录下生成一个新的模块。

1.  我们还需要为承诺 API 创建一个控制器和服务，所以让我们使用 NestJS CLI 生成它们：

    ```js
    nest g controller pledge
    nest g service pledge
    ```

1.  最后，我们需要安装 `typeorm` 库、SQLite3 和 NestJS 集成：

    ```js
    npm i @nestjs/typeorm sqlite3 typeorm
    ```

    TypeORM 通过在普通对象上的装饰器将数据库表映射到 TypeScript 实体。

1.  让我们在 `/pledge` 下创建 `pledge.entity.ts` 并创建我们的第一个实体：

    ```js
    import { Entity, PrimaryGeneratedColumn, Column } from 'typeorm';
    @Entity()
    export class Pledge {
      @PrimaryGeneratedColumn()
      id: number;
      @Column()
      desc: string;
      @Column()
      kept: boolean;
    }
    ```

    对于这个实体，我们使用了几个专门的装饰器，例如 `PrimaryGeneratedColumn`。这些装饰器可能非常强大，但通常依赖于底层数据库功能。因为 SQLite 可以为我们生成表 ID，TypeORM 能够通过装饰器以声明式的方式暴露这一点，但如果它不能这样做，那么这就不起作用了。在开始新的实现之前检查文档总是好的。

    现在我们有一个实体，我们需要向 TypeORM 提供配置，说明我们的数据库是什么以及在哪里可以找到它，以及我们想要映射哪些实体。对于 MySQL 和 Postgres 等数据库，这可能包括 URI 以及数据库凭据。由于 SQLite 是基于文件的数据库，我们只需提供我们想要写入的文件名。

    注意，生产数据库凭据应始终安全处理，而这样做最佳实践超出了本书的范围，但可以说，它们不应该被提交到版本控制中。

1.  让我们配置我们的应用程序以使用 SQLite。我们希望在应用程序的根目录下配置 TypeORM，所以让我们将模块导入到 `app.module.ts`：

    ```js
        TypeOrmModule.forRoot({
          type: 'sqlite',
          database: 'db',
          entities: [Pledge],
          synchronize: true,
        }),
    ```

1.  做这件事需要在模块顶部进行几个额外的导入：

    ```js
    import { TypeOrmModule } from '@nestjs/typeorm';
    import { Pledge } from './pledge/pledge.entity';
    ```

    我们让 NestJS 知道我们的应用程序将使用 SQLite 数据库并管理 `Pledge` 实体。通过设置 `synchronize: true`，我们告诉 TypeORM 在应用程序启动时自动创建数据库中尚不存在的任何实体。这个设置不应该在生产环境中使用，因为它可能会导致数据丢失。TypeORM 支持迁移，用于在生产环境中管理数据库，这是本书范围之外的另一个主题。

1.  如果我们现在使用 `npm run start:dev` 启动应用程序，它将启动，我们会得到一个名为 `db` 的新二进制文件（SQLite 数据库）。

1.  在我们可以在新模块中使用 `Pledge` 实体之前，我们需要做一些额外的样板代码。打开 `pledge.module.ts` 并添加一个导入，使模块看起来像这样：

    ```js
    import { Module } from '@nestjs/common';
    import { TypeOrmModule } from '@nestjs/typeorm';
    import { PledgeController } from './pledge.controller';
    import { Pledge } from './pledge.entity';
    import { PledgeService } from './pledge.service';
    @Module({
      controllers: [PledgeController],
      imports: [TypeOrmModule.forFeature([Pledge])],
      providers: [PledgeService],
    })
    export class PledgeModule {}
    ```

    这将允许 `Pledge` 实体被 `pledge.service.ts` 使用。再次强调，NestJS 有很多样板代码，这可能会让习惯于无意见的 ExpressJS 工作流程的开发者感到不适。这个模块系统可以帮助我们将应用程序隔离到功能区域。在决定是否将 NestJS 这样的框架用于您的应用程序或团队之前，了解结构化应用程序的好处是一个好主意。

    我们现在可以开始构建我们的 `Pledge` 服务了。TypeORM 支持两种模式：Active Record，其中实体本身具有读取和更新的方法，以及 Data Mapper，其中此类功能委托给 `Repository` 对象。在这个练习中，我们将遵循 Data Mapper 模式。

1.  首先，我们将在 `Pledge` 服务中添加一个构造函数并注入存储库，将其暴露为类的私有成员。一旦我们这样做，我们就可以开始访问一些存储库方法了：

    ```js
    import { Injectable } from '@nestjs/common';
    import { Pledge } from './pledge.entity';
    import { InjectRepository } from '@nestjs/typeorm';
    import { Repository } from 'typeorm';
    @Injectable()
    export class PledgeService {
      constructor(
        @InjectRepository(Pledge)
        private pledgeRepository: Repository<Pledge>,
      ) {}
      findAll(): Promise<Pledge[]> {
        return this.pledgeRepository.find();
      }
    }
    ```

    我们现在公开了一个 `findAll` 方法，它将查询数据库中的所有 `Pledge` 实体并将它们作为一个数组返回。

1.  在生产应用程序中，实现分页通常是一个好主意，但这对我们的目的来说已经足够了。让我们实现一些其他方法：

    ```js
    import { Injectable } from '@nestjs/common';
    import { InjectRepository } from '@nestjs/typeorm';
    import { DeleteResult, Repository } from 'typeorm';
    import { Pledge } from './pledge.entity';
    @Injectable()
    export class PledgeService {
      constructor(
        @InjectRepository(Pledge)
        private pledgeRepository: Repository<Pledge>,
      ) {}
      delete(id: number): Promise<DeleteResult> {
        return this.pledgeRepository.delete(id);
      }
      findAll(): Promise<Pledge[]> {
        return this.pledgeRepository.find();
      }
      findOne(id: number): Promise<Pledge> {
        return this.pledgeRepository.findOne(id);
      }
      save(pledge: Pledge): Promise<Pledge> {
        return this.pledgeRepository.save(pledge);
      }
    }
    ```

    我们可以使用存储库方法走得很远，这些方法会为我们生成 SQL 查询，但也可以使用 TypeORM 的 SQL 或查询构建器。

1.  在服务中实现这些方法不会将它们暴露给我们的 API，因此我们需要在 `pledge.controller.ts` 中添加匹配的控制器方法。每个控制器方法将委托给服务方法，NestJS 将负责将这些部分粘合在一起：

    ```js
    import { Body, Controller, Delete, Get, Param, Post } from '@nestjs/common';
    import { DeleteResult } from 'typeorm';
    import { Pledge } from './pledge.entity';
    import { PledgeService } from './pledge.service';
    @Controller('pledge')
    export class PledgeController {
      constructor(private readonly pledgeService: PledgeService) {}
      @Delete(':id')
      deletePledge(@Param('id') id: number): Promise<DeleteResult> {
        return this.pledgeService.delete(id);
      }
      @Get()
      getPledges(): Promise<Pledge[]> {
        return this.pledgeService.findAll();
      }
      @Get(':id')
      getPledge(@Param('id') id: number): Promise<Pledge> {
        return this.pledgeService.findOne(id);
      }
      @Post()
      savePledge(@Body() pledge: Pledge): Promise<Pledge> {
        return this.pledgeService.save(pledge);
      }
    }
    ```

    这个控制器将自动注入服务，然后可以使用装饰器和依赖注入轻松地将服务方法映射到 API 端点。

1.  由于我们使用 `npm run start:dev` 运行应用程序，它应该会通过所有这些更改进行热重载。

1.  检查控制台，确保没有错误。如果我们的代码是正确的，我们可以使用 Postman 这样的 REST 客户端开始向我们的服务发送请求。如果我们向 `http://localhost:3000/pledge` 发送包含如下负载的 `POST` 请求：`{"desc":"Always lint your code", "kept": true}`，我们将得到一个 `201 Created` 的 HTTP 响应。然后我们可以发出 `GET` 请求到 `http://localhost:3000/pledge` 和 `http://localhost:3000/pledge/1`，以查看存储在 SQLite 中的记录。

在这个练习中，我们使用了 NestJS 和 TypeORM 来构建一个真实的网络 API，该 API 可以从 SQLite 数据库中创建和检索记录。这样做与使用像 MySQL 或 PostgreSQL 这样的真实生产级数据库并没有太大的区别。

## 活动 13.01：将链式承诺重构为使用 await

在这个活动中，我们将重构一个将承诺链在一起的函数，以使用 `await`。你将得到一个入门程序，旨在模拟为网站创建 DOM 元素并依次渲染它们。在现实中，大多数网站都希望并行渲染，但可能一个组件的信息可能会影响另一个组件的渲染。在任何情况下，这对于示例来说都是足够的：

注意

这个活动的代码文件可以在以下链接找到：[`packt.link/L5r76`](https://packt.link/L5r76)。

1.  首先，使用 `npx ts-node src/refactor.ts` 运行程序。你会按顺序得到每条消息。

1.  现在，将 `renderAll` 函数重构为使用 `async`/`await`。你不需要修改代码的其他部分来实现这一点。当你完成重构后，再次运行程序并验证输出是否已更改。

入门程序的代码（`refactor.ts`）如下：

```js
export class El {
  constructor(private name: string) {}
  render = () => {
    return new Promise((resolve) =>
      setTimeout(
        () => resolve(`${this.name} is resolved`),
        Math.random() * 1000
      )
    );
  };
}
const e1 = new El('header');
const e2 = new El('body');
const e3 = new El('footer');
const renderAll = () => {
  e1.render().then((msg1) => {
    console.log(msg1);
    e2.render().then((msg2) => {
      console.log(msg2);
      e3.render().then((msg3) => {
        console.log(msg3);
      });
    });
  });
};
renderAll();
```

运行程序后，你应该得到以下输出：

```js
header is resolved
body is resolved
footer is resolved
```

注意

这个活动的解决方案可以通过这个链接找到。

# 摘要

在过去的 10 年里，异步编程已经取得了长足的进步，而 `async`/`await` 的引入继续推动其发展。尽管它并不适合每个用例，但这种语法糖在 TypeScript 社区中非常受欢迎，并在流行的库和框架中得到了广泛接受。

在本章中，我们介绍了 `async`/`await` 语法，它是如何成为语言的一部分，以及这种语法的使用实际上是如何与承诺（promises）相辅相成的。然后，我们游览了几个 TypeScript 开发者常用的流行框架，以了解应用开发者如何使用承诺和异步编程来开发强大的 Web 应用程序。

这本书对语言特性的研究到此结束。下一章将探讨使用 TypeScript 构建用户界面的 React。
