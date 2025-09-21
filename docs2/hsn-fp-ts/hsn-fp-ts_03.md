# 第三章：掌握异步编程

在上一章中，我们学习了如何处理函数。在本章中，我们将探讨如何使用函数以及一些原生 API 来编写异步 TypeScript 代码。我们将重点关注 TypeScript 的异步编程能力，包括以下概念：

+   回调函数和高级函数

+   箭头函数

+   回调地狱

+   Promises

+   生成器

+   异步函数（`async` 和 `await`）

# 回调函数和高级函数

在 TypeScript 中，函数可以作为参数传递给另一个函数。函数也可以由另一个函数返回。将函数作为参数传递的函数称为 **回调**。接受函数作为参数（回调）或返回函数的函数称为 **高级函数**。

回调通常是匿名函数。它们可以在传递给高级函数之前声明，如下面的例子所示：

```js
var myCallback = function() { // callback
  console.log("foo");
}

function bar(cb: () => void) { // higher order function
  console.log("bar");
  cb();
}

bar(myCallback); // prints "bar" then prints "foo"
```

回调也可以在行内声明，即在将它们传递给高级函数的同一位置，如下面的例子所示：

```js
bar(() => {
  console.log("foo");
}); // prints "bar" then prints "foo"
```

之前的代码片段声明了一个匿名函数并将其传递给名为 `bar` 的函数。匿名函数是使用一种称为箭头函数的替代语法声明的。我们将在下一节中了解更多关于箭头函数的内容。

# 箭头函数

在 TypeScript 中，我们可以使用函数表达式或箭头函数声明一个函数。箭头函数表达式比函数表达式语法更短，并且按词法绑定 `this` 操作符的值。

在 TypeScript 和 JavaScript 中，`this` 操作符的行为与其他流行的编程语言略有不同。当我们使用 TypeScript 定义一个类时，我们可以使用 `this` 操作符来引用该类。让我们看一个例子：

```js
class Person {
    private _name: string;
    constructor(name: string) {
        this._name = name;
    }
    public greet() {
        console.log('Hi! My name is ${this._name}');
    }
}

let person = new Person("Remo");
person.greet(); // "Hi! My name is Remo"
```

我们定义了一个名为 `Person` 的类，该类包含一个名为 `name` 的 `string` 类型属性。该类有一个构造函数和一个名为 `greet` 的方法。我们创建了一个名为 `person` 的实例并调用了 `greet` 方法，该方法内部使用 `this` 操作符来访问 `_name` 属性。在 `greet` 方法内部，`this` 操作符指向包含 `greet` 方法（类实例）的对象。

使用 `this` 操作符时必须小心，因为在某些情况下，它可能指向错误值。让我们在之前的例子中添加一个额外的方法：

```js
class Person {
    private _name: string;
    constructor(name: string) {
        this._name = name;
    }

    public greet() {
        alert('Hi! My name is ${this._name}');
    }

    public greetDelay(time: number) {
        setTimeout(function() {
            alert('Hi! My name is ${this._name}'); // Error
        }, time);
    }

}

let person = new Person("Remo");
person.greetDelay(1000); // Error
```

在 `greetDelay` 方法中，我们执行的操作几乎与 `greet` 方法执行的操作相同。这次，函数接受一个名为 `time` 的参数，该参数用于延迟 `greet` 消息。

要延迟消息，我们使用 `setTimeout` 函数和回调。一旦我们定义了一个匿名函数（回调），`this` 关键字就会改变其值并开始指向匿名函数，这也解释了为什么 TypeScript 编译器会抛出错误。

如前所述，箭头函数表达式词法绑定 `this` 操作符的值。这意味着它允许我们添加一个函数而不改变该操作符的值。让我们将前一个示例中的函数表达式替换为箭头函数：

```js
class Person {

    private _name: string;

    constructor(name: string) {
        this._name = name;
    }

    public greet() {
        alert('Hi! My name is ${this._name}');
    }

    public greetDelay(time: number) {
        setTimeout(() => {
            alert('Hi! My name is ${this._name}'); // OK
        }, time);
    }

}

let person = new Person("Remo");
person.greet(); // "Hi! My name is Remo"
person.greetDelay(1000); // "Hi! My name is Remo"
```

通过使用箭头函数，我们可以确保 `this` 操作符仍然指向 `Person` 实例，而不是 `setTimeout` 回调。如果我们执行 `greetDelay` 函数，名称属性将显示为预期的那样。

下面的代码片段是由 TypeScript 编译器生成的。当编译一个箭头函数时，TypeScript 编译器将为 `this` 操作符生成一个名为 `_this` 的别名。这个别名用于确保 `this` 操作符指向正确的对象：

```js
Person.prototype.greetDelay = function (time) {
  var _this = this;
  setTimeout(function () {
    alert("Hi! My name is " + _this._name);
  }, time);
};
```

我们将在第四章 《运行时 – 事件循环和 this 操作符》 中深入探讨 `this` 操作符，*运行时 – 事件循环和 this 操作符*。

# 回调地狱

我们已经了解到回调函数和高级函数是 JavaScript 和 TypeScript 中的两个强大且灵活的特性。然而，回调函数的使用可能会导致一个称为回调地狱的可维护性问题。

现在我们将编写一个示例来展示回调地狱。我们将编写三个具有相同行为的函数。

第一个函数被命名为 `doSomethingAsync`。该函数接受一个数字数组作为其参数之一，并向其中添加一个新数字。该函数使用 `setTimeout` 来模拟一些 I/O 操作，例如从数据库读取，以及使用 `Math.random` 来模拟潜在的异常，例如请求超时：

```js
function doSomethingAsync(
    arr: number[],
    success: (arr: number[]) => void,
    error: (e: Error) => void
) {
    setTimeout(() => {
        try {
            let n = Math.ceil(Math.random() * 100 + 1);
            if (n < 25) {
                throw new Error("n is < 25");
            }
            success([...arr, n]);
        } catch (e) {
            error(e);
        }
    }, 1000);
}
```

第二个函数被命名为 `doSomethingElseAsync`，第三个也是最后一个函数被命名为 `doSomethingMoreAsync`。我们将在以下代码片段中跳过这两个函数的实现，因为这两个函数与我们在 `doSomethingAsync` 函数中使用的实现完全相同：

```js
function doSomethingElseAsync(
    arr: number[],
    success: (arr: number[]) => void,
    error: (e: Error) => void
) {
    // ... Same implementation here...
}

function doSomethingMoreAsync(
    arr: number[],
    success: (arr: number[]) => void,
    error: (e: Error) => void
) {
    // Same imlementation here...
}
```

前面的函数通过使用 `setTimeout` 函数来模拟异步操作。每个函数都接受一个 `success` 回调，如果操作成功则调用该回调，以及一个 `error` 回调，如果发生错误则调用该回调。

在实际应用中，异步操作通常涉及与硬件的交互（例如，文件系统、网络等）。这些交互被称为 **输入**/**输出** （**I**/**O**）操作。I/O 操作可能因许多不同的原因而失败；例如，当我们尝试与文件系统交互以保存新文件时，硬盘上没有足够的空间，我们会得到一个错误。

前面的函数生成一个随机数，如果该数字小于 `25`，则抛出错误；我们这样做是为了模拟潜在的 I/O 错误。然后，它们将随机数添加到作为每个函数参数传递的数组中。如果没有发生错误，最终函数（`doSomethingMoreAsync`）的结果应该是一个包含三个随机数的数组。

现在已经声明了三个函数，我们可以尝试按顺序调用它们。我们将使用回调函数来确保在`doSomethingElseAsync`之后调用`doSomethingMoreAsync`，并且在`doSomethingAsync`之后调用`doSomethingElseAsync`：

```js
doSomethingAsync([], (arr1) => {
    doSomethingElseAsync(arr1, (arr2) => {
        doSomethingMoreAsync(arr2, (arr3) => {
            console.log(
                '
                doSomethingAsync: ${arr3[0]}
                doSomethingElseAsync: ${arr3[1]}
                doSomethingMoreAsync: ${arr3[2]}
                '
            );
        }, (e) => console.log(e));
    }, (e) => console.log(e));
}, (e) => console.log(e));
```

上述示例使用了几个嵌套的回调函数。这类嵌套回调函数被称为回调地狱，因为它们可能导致一些可维护性问题，如下所示：

+   它们使代码更难以跟踪和理解

+   它们使代码更难以维护（重构、重用等）

+   它们使异常处理更困难

# 承诺

在看到回调的使用可能导致一些可维护性问题之后，我们现在将学习关于承诺以及如何使用它们来编写更好的异步代码。承诺背后的核心思想是，承诺表示异步操作的结果。承诺必须处于以下三种状态之一：

+   **挂起**：承诺的初始状态。

+   **实现**：也称为已解决，这是表示成功操作的承诺的状态。术语*实现*和*已解决*都常用来指代此状态。

+   **已拒绝**：表示操作失败的承诺的状态。

一旦承诺被实现或拒绝，其状态将无法再改变。让我们看看承诺的基本语法：

```js
function foo() {
    return new Promise<string>((fulfill, reject) => {
        try {
            // do something
            fulfill("SomeValue");
        } catch (e) {
            reject(e);
        }
    });
}

foo().then((value) => {
    console.log(value);
}).catch((e) => {
    console.log(e);
});
```

请注意，这里使用`try…catch`语句来展示如何显式实现或拒绝承诺。对于`Promise`函数，不需要`try…catch`语句，因为当在承诺内部抛出错误时，承诺将自动被拒绝。

上述代码片段声明了一个名为`foo`的函数，它返回一个承诺。这个承诺包含一个名为`then`的方法，该方法接受一个回调函数作为参数。当承诺被实现时，将调用回调函数。承诺还提供了一个名为`catch`的方法，当承诺被拒绝时调用。

如果我们针对 ES5，TypeScript 编译器将不会识别承诺，因为`Promise` API 是 ES6 的一部分。我们可以通过在`tsconfig.json`文件中使用`lib`选项启用`es2015.promise`类型来解决此问题。请注意，启用此选项将禁用默认包含的一些类型，从而破坏一些示例。您可以通过在`tsconfig.json`文件中使用`lib`选项包括`dom`和`es5`类型来解决这些问题：

```js
"lib": [
     "es2015.promise",
     "dom",
     "es5",
     "es2015.generator", // new
     "es2015.iterable" // new
 ]
```

现在，我们将重写我们在*回调地狱*部分编写的`doSomethingAsync`、`doSomethingElseAsync`和`doSomethingMoreAsync`函数，但这次我们将使用承诺而不是回调：

```js
function doSomethingAsync(arr: number[]) {
    return new Promise<number[]>((resolve, reject) => {
        setTimeout(() => {
            try {
                let n = Math.ceil(Math.random() * 100 + 1);
                if (n < 25) {
                    throw new Error("n is < 25");
                }
                resolve([...arr, n]);
            } catch (e) {
                reject(e);
            }
        }, 1000);
    });
}
```

再次，我们将跳过`doSomethingElseAsync`和`doSomethingMoreAsync`函数的实现细节，因为它们应该与`doSomethingAsync`函数的实现相同：

```js
function doSomethingElseAsync(arr: number[]) {
    // Same implementation here...
}

function doSomethingMoreAsync(arr: number[]) {
    // Same implementation here...
}
```

我们可以使用 `Promise` API 将每个先前函数返回的承诺链式连接起来：

```js
doSomethingAsync([]).then((arr1) => {
    doSomethingElseAsync(arr1).then((arr2) => {
        doSomethingMoreAsync(arr2).then((arr3) => {
            console.log(
                '
                doSomethingAsync: ${arr3[0]}
                doSomethingElseAsync: ${arr3[1]}
                doSomethingMoreAsync: ${arr3[2]}
                '
            );
        });
    });
}).catch((e) => console.log(e));
```

上述代码片段比回调示例中使用的代码略好，因为我们只需要声明一个而不是三个异常处理程序。这是可能的，因为错误是通过承诺链传播的。然而，我们可以进一步改进代码，因为 `Promise` API 允许我们以更简洁的方式链式连接承诺：

```js
doSomethingAsync([])
    .then(doSomethingElseAsync)
    .then(doSomethingMoreAsync)
    .then((arr3) => {
        console.log(
            '
            doSomethingAsync: ${arr3[0]}
            doSomethingElseAsync: ${arr3[1]}
            doSomethingMoreAsync: ${arr3[2]}
            '
        );
    }).catch((e) => console.log(e));
```

之前的代码比回调示例中使用的代码更容易阅读和跟踪，但这并不是唯一青睐承诺而不是回调的原因。使用承诺也让我们能够更好地控制操作的执行流程。让我们看看几个例子。

`Promise` API 包含一个名为 `all` 的方法，它允许我们并行执行一系列承诺，并一次性获取每个承诺的所有结果：

```js
Promise.all([
    new Promise<number>((resolve) => {
        setTimeout(() => resolve(1), 1000);
    }),
    new Promise<number>((resolve) => {
        setTimeout(() => resolve(2), 1000);
    }),
    new Promise<number>((resolve) => {
        setTimeout(() => resolve(3), 1000);
    })
]).then((values) => {
    console.log(values); // [ 1 ,2, 3]
});
```

`Promise` API 还包含一个名为 `race` 的方法，它允许我们并行执行一系列承诺，并获取第一个解决的承诺的结果：

```js
Promise.race([
    new Promise<number>((resolve) => {
        setTimeout(() => resolve(1), 3000);
    }),
    new Promise<number>((resolve) => {
        setTimeout(() => resolve(2), 2000);
    }),
    new Promise<number>((resolve) => {
        setTimeout(() => resolve(3), 1000);
    })
]).then((fastest) => {
    console.log(fastest); // 3
});
```

当与承诺一起工作时，我们可以使用许多不同类型的异步流程控制：

+   **并发**: 任务并行执行（如 `Promise.all` 示例所示）

+   **竞速**: 任务并行执行，只返回最快承诺的结果

+   **串联**: 一组任务按顺序执行，但前面的任务不会将参数传递给下一个任务

+   **瀑布**: 一组任务按顺序执行，每个任务将参数传递给下一个任务（如示例所示）

+   **复合**: 这是指之前并发、串联和瀑布方法的任何组合

# 回调参数中的协变检查

TypeScript 2.4 改变了类型系统内部的行为方式，以改善嵌套回调和承诺的错误检测：

+   TypeScript 对回调参数的检查现在与即时签名检查是协变的。之前它是双变的，偶尔会允许错误类型通过。

+   基本上，这意味着回调参数和包含回调的类将受到更仔细的检查，因此 TypeScript 在这个版本中将需要更严格的类型。这尤其适用于承诺和观察者，因为它们的 API 指定方式。

在 TypeScript 2.4 版本之前的 TypeScript 中，以下示例被认为是有效的，并且没有抛出错误：

```js
declare function someFunc(
    callback: (
    nestedCallback: (error: number, result: any) => void
    ) => void
): void;

someFunc(
    (
        nestedCallback: (e: number) => void // Error
    ) => {
        nestedCallback(1);
    }
);
```

在 TypeScript 2.4 版本之后的 TypeScript 中，我们将需要添加 `nestedCallback` 的完整签名来解决此错误：

```js
someFunc(
    (
        nestedCallback: (e: number, result: any) => void // OK
    ) => {
        nestedCallback(1, 1);
    }
);
```

多亏了 TypeScript 类型系统的内部变化，以下错误也被检测到：

```js
let p: Promise<number> = new Promise((res, rej) => {
    res("error"); // Error
});
```

在 TypeScript 2.4 之前，上述承诺会被推断为 `Promise<{}>`，因为我们创建 `Promise` 类实例时忘记添加泛型参数 `<number>`。字符串错误随后会被视为 `{}` 的有效实例。

这前面的例子清楚地说明了为什么建议您定期升级 TypeScript。TypeScript 的每个新版本都引入了新的功能，能够为我们检测到新的错误。

# 生成器

如果我们在 TypeScript 中调用一个函数，我们可以假设一旦函数开始运行，它将始终运行到完成，然后任何其他代码才能运行。然而，一种称为**生成器**的函数类型可以在执行过程中暂停——一次或多次——并在稍后恢复，允许在这些暂停期间运行其他代码。

生成器代表一系列值。生成器对象的接口只是一个**迭代器**。迭代器实现了以下接口：

```js
interface Iterator<T> {
  next(value?: any): IteratorResult<T>;
  return?(value?: any): IteratorResult<T>;
  throw?(e?: any): IteratorResult<T>;
}
```

`next`函数可以被调用，直到它耗尽值。我们可以通过使用`function`关键字，后跟一个星号（`*`），来定义一个生成器。`yield`关键字用于停止函数的执行并返回一个值。让我们看一个例子：

```js
function *foo() {
    yield 1;
    yield 2;
    yield 3;
    yield 4;
    return 5;
}

let bar = foo();
bar.next(); // Object {value: 1, done: false}
bar.next(); // Object {value: 2, done: false}
bar.next(); // Object {value: 3, done: false}
bar.next(); // Object {value: 4, done: false}
bar.next(); // Object {value: 5, done: true}
bar.next(); // Object { done: true }
```

注意，如果您针对 ES5，生成器需要一些额外的类型。您需要在`tsconfig.json`文件中添加`es2015.generator`和`es2015.iterable`，并启用`downlevelIteration`：

```js
"lib": [
     "es2015.promise",
     "dom",
     "es5",
     "es2015.generator", // new
     "es2015.iterable" // new
 ]
```

正如我们所见，前面的迭代器有五个步骤。第一次我们调用`next`方法时，函数将执行，直到它达到第一个`yield`语句，然后它将返回值`1`并停止函数的执行，直到我们再次调用生成器的`next`方法。正如我们所见，我们现在能够在一个给定的点上停止函数的执行。这允许我们编写无限循环而不会导致堆栈溢出异常，如下面的示例所示：

```js
function* foo() {
    let i = 1;
    while (true) { // Infinite loop!
        yield i++;
    }
}

let bar = foo();
bar.next(); // Object {value: 1, done: false}
bar.next(); // Object {value: 2, done: false}
bar.next(); // Object {value: 3, done: false}
bar.next(); // Object {value: 4, done: false}
bar.next(); // Object {value: 5, done: false}
bar.next(); // Object {value: 6, done: false}
bar.next(); // Object {value: 7, done: false}
```

生成器的 API 通过同步性打开了可能性，因为异步事件发生后，我们可以调用生成器的`next`方法。

# 异步函数 – async 和 await

异步函数是 TypeScript 1.6 版本中引入的功能。开发者可以使用`await`关键字等待异步操作完成，而不会阻塞程序的正常执行。

使用异步函数可以帮助提高代码的可读性，与使用承诺或回调相比，但技术上，我们可以使用两者都实现相同的功能。

让我们看看一个基本的`async`/`await`示例：

```js
let p = Promise.resolve(3);

async function fn(): Promise<number> {
    var i = await p; // 3
    return 1 + i; // 4
}

fn().then((r) => console.log(r)); // 4
```

上述代码片段声明了一个名为`p`的承诺。这个承诺代表一个未来的结果。正如我们所见，`fn`函数前面有`async`关键字，它用于指示编译器这是一个异步函数。

在函数内部，`await`关键字用于暂停执行，直到承诺`p`得到解决或拒绝。正如我们所见，语法比使用`Promise` API 或回调要简洁得多。

一个异步函数，如 `fn`，在运行时会返回一个承诺。这应该解释了为什么我们需要在代码片段的末尾使用 `then` 方法的原因。

以下代码片段展示了如何声明一个名为 `invokeTaskAsync` 的异步函数。这个异步函数使用 `await` 关键字等待我们在承诺示例中声明的 `doSomethingAsync`、`doSomethingElseAsync` 和 `doSomethingMoreAsync` 函数的结果：

```js
async function invokeTaskAsync() {
    let arr1 = await doSomethingAsync([]);
    let arr2 = await doSomethingElseAsync(arr1);
    return await doSomethingMoreAsync(arr2);
}
```

`invokeTaskAsync` 函数是异步的。因此，它在运行时会返回一个承诺。这意味着我们可以使用 `Promise` API 来等待结果或捕获潜在的错误：

```js
invokeTaskAsync().then((result) => {
    console.log(
        '
        doSomethingAsync: ${result[0]}
        doSomethingElseAsync: ${result[1]}
        doSomethingMoreAsync: ${result[2]}
        '
    );
}).catch((e) => {
    console.log(e);
});
```

我们还可以将异步 IIFE 定义为使用 `async` 和 `await` 关键字的便捷方式：

```js
(async () => {
    try {
        let arr1 = await doSomethingAsync([]);
        let arr2 = await doSomethingElseAsync(arr1);
        let arr3 = await doSomethingMoreAsync(arr2);
        console.log(
            '
            doSomethingAsync: ${arr3[0]}
            doSomethingElseAsync: ${arr3[1]}
            doSomethingMoreAsync: ${arr3[2]}
            '
        );
    } catch (e) {
        console.log(e);
    }
})();
```

使用 `async` IIFE 非常有用，因为通常无法在函数外部使用 `await` 关键字，例如，在应用程序的入口点。我们可以使用 `async` IIFE 来克服这种限制：

```js
(async () => {
    await someAsyncFunction();
})();
```

# 异步生成器

我们已经学习了所有迭代器实现的接口：

```js
interface Iterator<T> {
  next(value?: any): IteratorResult<T>;
  return?(value?: any): IteratorResult<T>;
  throw?(e?: any): IteratorResult<T>;
}
```

然而，我们还没有学习所有异步迭代器实现的接口：

```js
interface AsyncIterator<T> {
  next(value?: any): Promise<IteratorResult<T>>;
  return?(value?: any): Promise<IteratorResult<T>>;
  throw?(e?: any): Promise<IteratorResult<T>>;
}
```

每次我们调用 `next` 方法时，异步迭代器都会返回一个承诺。以下代码片段展示了异步迭代器如何与异步函数结合使用，从而变得非常有用：

```js
let counter = 0;

function doSomethingAsync() {
    return new Promise<number>((r) => {
        setTimeout(() => {
            counter += 1;
            r(counter);
        }, 1000);
    });
}

async function* g1() {
    yield await doSomethingAsync();
    yield await doSomethingAsync();
    yield await doSomethingAsync();
}

let i: AsyncIterableIterator<number> = g1();
i.next().then((n) => console.log(n)); // 1
i.next().then((n) => console.log(n)); // 2
i.next().then((n) => console.log(n)); // 3
```

如果我们针对 ES5，异步迭代器需要一些额外的类型。您需要在 `tsconfig.json` 文件中添加 `esnext.asynciterable` 并启用 `downlevelIteration`。我们还需要在 `tsconfig.json` 中启用一个额外的设置，以在针对 ES5 或 ES3 时提供对 `for-of`、展开和结构赋值的完整支持：

```js
"lib": [
     "es2015.promise",
     "dom",
     "es5",
     "es2015.generator",
     "es2015.iterable",
     "esnext.asynciterable" // new
 ]
```

# 异步迭代（for await…of）

我们可以使用新的 `await…of` 表达式来迭代并等待异步迭代器返回的每个承诺：

```js
function* g1() {
    yield 2;
    yield 3;
    yield 4;
}

async function func() {
    for await (const x of g1()) {
        console.log(x);
    }
}

(async () => {
    await func();
})();
```

# 将迭代委托给另一个生成器（yield*）

我们可以使用 `yield*` 表达式将一个生成器委托给另一个生成器。以下代码片段定义了两个生成器函数，分别命名为 `g1` 和 `g2`。`g2` 生成器使用 `yield*` 表达式将迭代委托给由 `g1` 创建的迭代器：

```js
function* g1() {
    yield 2;
    yield 3;
    yield 4;
}

function* g2() {
    yield 1;
    yield* g1();
    yield 5;
}

var iterator1 = g2();

console.log(iterator1.next()); // {value: 1, done: false}
console.log(iterator1.next()); // {value: 2, done: false}
console.log(iterator1.next()); // {value: 3, done: false}
console.log(iterator1.next()); // {value: 4, done: false}
console.log(iterator1.next()); // {value: 5, done: false}
console.log(iterator1.next()); // {value: undefined, done: true}
```

`yield*` 表达式也可以用来将迭代委托给某些可迭代对象，例如数组：

```js
function* g2() {
    yield 1;
    yield* [2, 3, 4];
    yield 5;
}

var iterator = g2();

console.log(iterator.next()); // {value: 1, done: false}
console.log(iterator.next()); // {value: 2, done: false}
console.log(iterator.next()); // {value: 3, done: false}
console.log(iterator.next()); // {value: 4, done: false}
console.log(iterator.next()); // {value: 5, done: false}
console.log(iterator.next()); // {value: undefined, done: true}
```

请注意，前面的示例需要在 `tsconfig.json` 文件中设置一系列特定的设置。请参考本章前面的说明，了解更多关于所需设置的信息。

# 概述

在本章中，我们专注于使用回调、承诺和生成器来利用 TypeScript 的异步编程能力。在下一章中，我们将探讨运行时，了解事件循环和 `this` 操作符的工作原理。这些概念将帮助我们理解本书后面将要探索的一些函数式编程技术的实现。
