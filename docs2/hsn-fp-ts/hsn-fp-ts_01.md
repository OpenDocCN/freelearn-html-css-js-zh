# 第一章：函数式编程基础

自从 1995 年 JavaScript 诞生以来，JavaScript 一直是一种多范式编程语言。它允许我们利用面向对象编程（**OOP**）风格以及函数式编程风格。同样，TypeScript 也是如此。然而，对于函数式编程来说，TypeScript 比 JavaScript 更适合，因为正如我们将在本章中学习的，静态类型系统和类型推断是函数式编程语言（例如 ML 编程语言家族）中非常重要的特性。

在过去几年中，JavaScript 和 TypeScript 生态系统对函数式编程的兴趣显著增加。我相信这种兴趣的增加可以归因于 React 的成功。React 是由 Facebook 开发的一个用于构建用户界面的库，它深受一些核心函数式编程概念的影响。

在本章中，我们将专注于学习一些最基础的函数式编程概念和原则。

本章中，你将学习以下内容：

+   函数式编程的主要特征

+   函数式编程的主要好处

+   纯函数

+   副作用

+   不可变性

+   函数的阶数

+   高阶函数

+   惰性

# TypeScript 是不是函数式编程语言？

这个问题的答案是肯定的，但只是部分。TypeScript 是一种多范式编程语言，因此它包含了来自面向对象语言和函数式编程范式的许多影响。

然而，如果我们把 TypeScript 视为一种函数式编程语言，我们会发现它并不是一种纯粹的函数式编程语言，因为例如，TypeScript 编译器并不强制我们的代码无副作用。

不是纯粹的函数式编程语言不应被视为负面因素。TypeScript 为我们提供了一套广泛的功能，使我们能够利用面向对象编程语言世界和函数式编程语言世界的最佳特性。这使得 TypeScript 类型系统在生产力与形式之间达到了一个非常好的平衡。

# 函数式编程的好处

使用函数式编程风格编写 TypeScript 代码有许多好处，其中我们可以突出以下内容：

+   **我们的代码是可测试的**：如果我们尝试将我们的函数编写为纯函数，我们将能够非常容易地编写单元测试。我们将在本章后面了解更多关于纯函数的内容。

+   **我们的代码易于推理：** 对于缺乏函数式编程经验的开发者来说，函数式编程可能难以理解。然而，当应用程序正确地使用函数式编程范式实现时，结果是非常小的函数（通常是单行函数）和非常声明性的 API，可以轻松地进行推理。此外，纯函数只与它们的参数一起工作，这意味着当我们想要了解一个函数的功能时，我们只需要检查该函数本身，而无需担心任何其他外部变量。

+   **并发：** 我们的大多数函数都是无状态的，我们的代码主要是无状态的。我们将状态从应用程序的核心中推出去，这使得我们的应用程序更有可能支持许多并发操作，并且将具有更好的可扩展性。我们将在本章的后面部分学习更多关于无状态代码的内容。

+   **更简单的缓存：** 当我们能根据函数的参数预测其输出时，缓存结果的缓存策略会变得简单得多。

# 介绍函数式编程

**函数式编程**（**FP**）是一种编程范式，其名称来源于我们使用它时构建应用程序的方式。在面向对象编程（OOP）这样的编程范式中，我们用来创建应用程序的主要构建块是对象（对象使用类声明）。然而，在 FP 中，我们将函数作为我们应用程序中的主要构建块**。**

每一种新的编程范式都会引入一系列与之相关的概念和思想。其中一些概念是通用的，在学习不同的编程范式时也很有兴趣。在面向对象编程（OOP）中，我们有诸如继承、封装和多态等概念。在函数式编程中，概念包括高阶函数、函数部分应用、不可变性和引用透明性。我们将在本章中探讨一些这些概念。

**迈克尔·费思**（Michael Feathers），SOLID 缩写词的作者以及许多其他著名的软件工程原则的作者，曾写下以下内容：

"面向对象编程通过封装移动部件使代码易于理解。函数式编程通过最小化移动部件使代码易于理解。"

– *迈克尔·费思*

前面的引用提到了移动部件。我们应该将这些移动部件理解为**状态变化**（也称为**状态突变**）。在面向对象编程（OOP）中，我们使用封装来防止对象意识到其他对象的状态突变。在函数式编程中，我们试图避免处理状态突变，而不是封装它们。

函数式编程（FP）减少了应用程序中状态变化发生的地方的数量，并试图将这些地方移动到应用程序的边界内，以尝试保持应用程序的核心无状态。

可变状态是坏的，因为它使我们的代码的行为更难以预测。以下函数为例：

```js
function isIndexPage() {
  return window.location.pathname === "/";
}
```

前面的代码片段声明了一个名为`isIndexPage`的函数。这个函数可以用来检查当前页面是否是 Web 应用程序中的根页面，基于当前路径。

路径是不断变化的数据，因此我们可以将其视为状态的一部分。如果我们尝试预测调用`isIndexPage`的结果，我们需要知道当前状态。问题是，我们可能会错误地假设状态自上次已知状态以来没有发生变化。我们可以通过将前面的函数转换为在 FP 中称为**纯函数**的形式来解决此问题，正如我们将在下一节中学习的那样。

# 纯函数

函数式编程引入了许多概念和原则，这些将帮助我们提高代码的可预测性。在本节中，我们将学习这些核心概念之一——纯函数。

当一个函数仅使用传递给它的参数来计算返回值时，我们可以认为它是纯函数。此外，纯函数避免修改其参数或任何其他外部变量。因此，纯函数在给定相同的参数时，总是返回相同的值，无论何时被调用。

在前一个部分中声明的`isIndexPage`函数不是一个纯函数，因为它访问了`pathname`变量，而这个变量并没有作为参数传递给函数。我们可以通过以下方式重写前面的函数，将其转换为纯函数：

```js
function isIndexPage(pathname: string) {
  return pathname === "/";
}
```

尽管这是一个基本的例子，但我们很容易感知到新版本更容易预测。纯函数帮助我们使代码更容易理解、维护和测试。

想象一下，如果我们想要为`isIndexPage`函数的不纯版本编写单元测试。在尝试编写测试时，我们会遇到一些问题，因为该函数使用了`window.location`对象。我们可以通过使用模拟框架来克服这个问题，但这会为我们的单元测试增加很多复杂性，仅仅因为我们没有使用纯函数。

另一方面，测试`isIndexPage`函数的纯版本将会非常直接，如下所示：

```js
function shouldReturnTrueWhenPathIsIndex(){
    let expected = true;
    let result = isIndexPage("/");
    if (expected !== result) {
        throw new Error('Expected ${expected} to equals ${result}');
    }
}

function shouldReturnFalseWhenPathIsNotIndex() {
    let expected = false;
    let result = isIndexPage("/someotherpage");
    if (expected !== result) {
        throw new Error('Expected ${expected} to equals ${result}');
    }
}
```

现在我们已经了解了函数式编程如何通过避免状态突变来帮助我们编写更好的代码，我们可以学习副作用和引用透明度。

# 副作用

在前一个部分中，我们了解到纯函数返回的值可以使用仅传递给它的参数来计算。纯函数还避免修改其参数或任何其他未作为参数传递给函数的外部变量。在函数式编程术语中，通常说纯函数是没有副作用的函数，这意味着当我们调用纯函数时，我们可以期望该函数不会干扰（通过状态突变）我们应用程序中的任何其他组件。

某些编程语言，如 Haskell，可以使用它们的类型系统确保应用程序没有副作用。TypeScript 与 JavaScript 有出色的互操作性，但与像 Haskell 这样的更隔离的语言相比，其缺点是类型系统无法保证我们的应用程序没有副作用。然而，我们可以使用一些 FP 技术来提高我们的 TypeScript 应用程序的类型安全性。让我们来看一个例子：

```js
interface User {
    ageInMonths: number;
    name: string;
}

function findUserAgeByName(users: User[], name: string): number {
    if (users.length == 0) {
        throw new Error("There are no users!");
    }
    const user = users.find(u => u.name === name);
    if (!user) {
        throw new Error("User not found!");
    } else {
        return user.ageInMonths;
    }
}
```

前面的函数返回一个`number`。代码编译没有问题。问题是这个函数并不总是返回一个`number`。因此，我们可以这样使用该函数，并且代码会编译，但在运行时抛出异常：

```js
const users = [
    { ageInMonths: 1, name: "Remo" },
    { ageInMonths: 2, name: "Leo" }
];

// The variable userAge1 is as number
const userAge1 = findUserAgeByName(users, "Remo"); 
console.log('Remo is ${userAge1 / 12} years old!');

// The variable userAge2 is a number but the function throws!
const userAge2 = findUserAgeByName([], "Leo"); // Error
console.log('Leo is ${userAge2 / 12} years old!');
```

以下示例展示了先前函数的新实现。这次，我们不再返回一个数字，而是明确地返回一个承诺。这个承诺迫使我们使用处理器。这个处理器只有在承诺得到履行时才会执行，这意味着如果函数返回错误，我们永远不会尝试将年龄转换为年：

```js
function safeFindUserAgeByName(users: User[], name: string): Promise<number> {
    if (users.length == 0) {
        return Promise.reject(new Error("There are no users!"));
    }
    const user = users.find(u => u.name === name);
    if (!user) {
        return Promise.reject(new Error("User not found!"));
    } else {
        return Promise.resolve(user.ageInMonths);
    }
}

safeFindUserAgeByName(users, "Remo")
    .then(userAge1 => console.log('Remo is ${userAge1 / 12} years old!'));

safeFindUserAgeByName([], "Leo") // Error
    .then(userAge1 => console.log('Leo is ${userAge1 / 12} years old!'));
```

`Promise`类型帮助我们防止错误，因为它以明确的方式表达潜在的错误。在像 Haskell 这样的编程语言中，这是类型系统的默认行为，但在像 TypeScript 这样的编程语言中，我们必须自己以更安全的方式使用类型。

我们将在第三章“掌握异步编程”中了解更多关于承诺的内容。我们还将了解如何在第八章“范畴论”中使用多个库来减少 TypeScript 应用程序中副作用的可能性。

如果你认为你的 JavaScript 应用程序没有副作用的想法很有吸引力，你可以尝试开源项目，例如[`github.com/bodil/eslint-config-cleanjs`](https://github.com/bodil/eslint-config-cleanjs)。该项目是一个 ESLint 配置，旨在限制你使用 JavaScript 的一个子集，使其尽可能接近理想化的纯函数式语言。不幸的是，在出版时，没有可用的类似工具是专门为 TypeScript 设计的。

# 引用透明性

引用透明性是与纯函数和副作用密切相关的一个概念。一个函数在它没有副作用时是纯的。当一个表达式可以被其对应值替换而不改变应用程序的行为时，它被称为引用透明的。例如，如果我们正在使用以下代码：

```js
let result = isIndexPage("/");
```

我们知道`isIndexPage`函数是引用透明的，因为它可以安全地用其返回类型替换它。在这种情况下，我们知道当我们用`/`作为参数调用`isIndexPage`函数时，该函数总是会返回`true`，这意味着我们可以安全地执行以下操作：

```js
let result = true;
```

纯函数是一个引用透明表达式。一个不是引用透明的表达式被称为引用不透明。

# 无状态与有状态

纯函数和引用透明表达式是无状态的。当代码的输出不受先前事件的影响时，该代码是无状态的。例如，`isIndexPage`函数的结果不会受到我们调用它的次数或调用它的具体时间的影响。

无状态代码的反面是有状态代码。无状态代码非常难以测试，当我们试图实现可扩展和具有弹性的系统时，它成为一个问题。具有弹性的系统是可以处理服务器故障的系统；通常有多个服务实例，如果一个实例崩溃，其他实例可以继续处理流量。此外，在某个实例崩溃后，会自动创建新的实例。如果我们的服务器是有状态的，这会变得非常困难，因为我们需要在崩溃前保存当前状态，并在启动新实例前恢复状态。当我们设计服务器为无状态时，整个过程会变得简单得多。

随着云计算革命的到来，这类系统变得更加普遍，这导致了对函数式编程语言和设计原则的兴趣，因为函数式编程鼓励我们编写无状态代码。对于 OOP 来说，情况则相反，因为类是 OOP 应用中的主要构造。类封装了状态属性，然后通过方法进行修改，这鼓励方法是有状态的，而不是纯的。

# 声明式与命令式编程

FP 范式的倡导者经常将声明式编程作为其主要好处之一。声明式编程不一定仅限于函数式编程，但 FP 确实鼓励或促进这种编程风格。在我们查看一些示例之前，我们将定义声明式编程和命令式编程：

+   **命令式编程**是一种使用语句改变程序状态的编程范式。与自然语言中的祈使语气表达命令的方式类似，命令式程序由计算机执行的一系列命令组成。命令式编程侧重于描述程序如何运行。

+   **声明式编程**是一种编程范式，它通过描述计算逻辑而不是描述其控制流来表达。许多采用这种风格的编程语言试图通过以问题域的术语描述程序必须完成的内容来最小化或消除副作用，而不是描述如何通过一系列步骤来完成它。

以下示例计算了一个包含学生 ID 和成绩列表的对象集合的考试平均结果。这个示例使用命令式编程风格，因为我们可以看到，它使用了控制流语句（`for`）。这个示例也很明显是命令式的，因为它改变了状态。`total`变量使用`let`关键字声明，因为它被修改了，就像`results`数组中的结果一样多次：

```js
interface Result {
 id: number;
 result:number;
}

const results: Result[] = [
 { id: 1, result: 64 },
 { id: 2, result: 87 },
 { id: 3, result: 89 }
];

function avg(arr: Result[]) {
 let total = 0;
 for (var i = 0; i < arr.length; i++) {
 total += arr[i].result;
 }
 return total / arr.length;
}

const resultsAvg = avg(results);
console.log(resultsAvg);
```

另一方面，以下示例是声明式的，因为没有控制流语句，也没有状态突变：

```js
interface Result {
    id: number;
    result:number;
}

const results: Result[] = [
    { id: 1, result: 64 },
    { id: 2, result: 87 },
    { id: 3, result: 89 }
];

const add = (a: number, b: number) => a + b;
const division = (a: number, b: number) => a / b;

const avg = (arr: Result[]) =>
    division(arr.map(a => a.result).reduce(add, 0), arr.length)

const resultsAvg = avg(results);
console.log(resultsAvg);
```

虽然前面的示例是声明式的，但它并不像可能的那样声明式。以下示例将声明式风格进一步推进，以便我们可以了解一段声明式代码可能的样子。如果你现在不理解这个示例中的所有内容，请不要担心。一旦我们在这本书的后面部分学习了更多关于函数式编程技术，我们就能理解它。注意，现在程序被定义为一组非常小的函数，这些函数不改变状态，也不使用控制流语句。这些函数是可重用的，因为它们独立于我们试图解决的问题。例如，`avg`函数可以计算平均值，但它不需要是结果的平均值：

```js
const add = (a: number, b: number) => a + b;
const addMany = (...args: number[]) => args.reduce(add, 0);
const div = (a: number, b: number) => a / b;
const mapProp = <T>(k: keyof T, arr: T[]) => arr.map(a => a[k]);
const avg = (arr: number[]) => div(addMany(...arr), arr.length);

interface Result {
    id: number;
    result:number;
}

const results: Result[] = [
    { id: 1, result: 64 },
    { id: 2, result: 87 },
    { id: 3, result: 89 }
];

const resultsAvg = avg(mapProp("result", results));
console.log(resultsAvg);
```

我们试图解决的问题的特定代码非常小：

```js
const resultsAvg = avg(mapProp("result", results));
```

这段代码不可重用，但`add`、`addMany`、`div`、`mapProp`和`avg`函数是可重用的。这展示了声明式编程如何比命令式编程产生更多可重用代码。

# 不可变性

不可变性指的是在将值赋给变量之后无法更改该变量的值。纯函数式编程语言包括常见数据结构的不可变实现。例如，当我们向数组添加一个元素时，我们正在修改原始数组。然而，如果我们使用不可变数组，并尝试向其中添加新元素，原始数组将不会被修改，我们将新项目添加到它的副本中。

以下代码片段声明了一个名为`ImmutableList`的类，展示了如何实现不可变数组：

```js
class ImmutableList<T> {
    private readonly _list: ReadonlyArray<T>;
    private _deepCloneItem(item: T) {
        return JSON.parse(JSON.stringify(item)) as T;
    }
    public constructor(initialValue?: Array<T>) {
        this._list = initialValue || [];
    }
    public add(newItem: T) {
        const clone = this._list.map(i => this._deepCloneItem(i));
        const newList = [...clone, newItem];
        const newInstance = new ImmutableList<T>(newList);
        return newInstance;
    }
    public remove(
        item: T,
        areEqual: (a: T, b: T) => boolean = (a, b) => a === b
    ) {
        const newList = this._list.filter(i => !areEqual(item, i))
                            .map(i => this._deepCloneItem(i));
        const newInstance = new ImmutableList<T>(newList);
        return newInstance;
    }
    public get(index: number): T | undefined {
        const item = this._list[index];
        return item ? this._deepCloneItem(item) : undefined;
    }
    public find(filter: (item: T) => boolean) {
        const item = this._list.find(filter);
        return item ? this._deepCloneItem(item) : undefined;
    }
}
```

每次我们向不可变数组中添加或删除一个项目时，我们都会创建一个不可变数组的新的实例。这种实现非常低效，但它展示了基本思想。我们将创建一个快速测试来演示前面类的工作方式。我们将使用一些关于超级英雄的数据：

```js
interface Hero { 
    name: string;
    powers: string[];
}

const heroes = [
    {
        name: "Spiderman",
        powers: [
            "wall-crawling",
            "enhanced strength",
            "enhanced speed",
            "spider-Sense"
        ]
    },
    {
        name: "Superman",
        powers: [
            "flight",
            "superhuman strength",
            "x-ray vision",
            "super-speed"
        ]
    }
];

const hulk = {
    name: "Hulk",
    powers: [
        "superhuman strength",
        "superhuman speed",
        "superhuman Stamina",
        "superhuman durability"
    ]
};
```

现在我们可以使用前面的数据来创建一个新的不可变列表实例。当我们向列表中添加一个新的超级英雄时，会创建一个新的不可变列表。如果我们尝试在两个不可变列表中搜索超级英雄“浩克”，我们会观察到只有第二个列表包含它。我们还可以比较这两个列表，以观察到它们是两个不同的对象，如下所示：

```js
const myList = new ImmutableList<Hero>(heroes);
const myList2 = myList.add(hulk);
const result1 = myList.find((h => h.name === "Hulk")); 
const result2 = myList2.find((h => h.name === "Hulk"));
const areEqual = myList2 === myList;

console.log(result1); // undefined
console.log(result2); // { name: "Hulk", powers: Array(4) }
console.log(areEqual); // false
```

在大多数情况下，创建我们自己的不可变数据结构是不必要的。在实际应用中，我们可以使用`Immutable.js`等库来享受不可变数据结构。

# 函数作为一等公民

在 FP 文献中经常提到函数作为**一等公民**。当我们说一个函数是一等公民时，意味着它可以做任何变量能做的事情，这意味着函数可以作为参数传递给其他函数。例如，以下函数将其第二个参数作为函数：

```js
function find<T>(arr: T[], filter: (i: T) => boolean) {
    return arr.filter(filter);
}

find(heroes, (h) => h.name === "Spiderman");
```

或者，它被另一个函数返回。例如，以下函数只接受一个函数作为其唯一参数，并返回一个函数：

```js
function find<T>(filter: (i: T) => boolean) {
    return (arr: T[]) => {
        return arr.filter(filter);
    }
}

const findSpiderman = find((h: Hero) => h.name === "Spiderman");
const spiderman = findSpiderman(heroes);
```

函数也可以赋值给变量。例如，在前面的代码片段中，我们将 find 函数返回的函数赋值给名为`findSpiderman`的变量：

```js
const findSpiderman = find((h: Hero) => h.name === "SPiderman");
```

JavaScript 和 TypeScript 都将函数视为一等公民。

# Lambda 表达式

Lambda 表达式只是可以用来声明匿名函数（无名的函数）的表达式。在 ES6 规范之前，将函数作为值赋给变量的唯一方法是使用函数表达式：

```js
const log = function(arg: any) { console.log(arg); };
```

ES6 规范引入了箭头函数语法：

```js
const log = (arg: any) => console.log(arg);
```

请参阅第二章“掌握函数”、第四章“运行时 – 事件循环和 this 操作符”以及第五章“运行时 – 闭包和原型”，以了解更多关于箭头函数和函数表达式的信息。

# 函数元数

函数的**元数**是指函数所接受的参数数量。一元函数是指只接受单个参数的函数：

```js
function isNull<T>(a: T|null) {
    return (a === null);
}
```

一元函数在函数式编程中非常重要，因为它们促进了函数组合模式的应用。

我们将在第六章“函数式编程技术”中更深入地了解函数组合模式。

二元函数是指接受两个参数的函数：

```js
function add(a: number, b: number) {
    return a + b;
}
```

具有两个或更多参数的函数也很重要，因为一些最常用的 FP 模式和技巧（例如，部分应用和柯里化）已经被设计成将接受多个参数的函数转换为单参数函数。

还有一些具有三个（**三元函数**）或更多参数的函数。然而，接受可变数量参数的函数，称为**可变参数函数**，在函数式编程中特别有趣，如下代码片段所示：

```js
function addMany(...numbers: number[]) {
    numbers.reduce((p, c) => p + c, 0);
}
```

# 高阶函数

高阶函数是指至少执行以下操作之一的函数：

+   接受一个或多个函数作为参数

+   返回一个函数作为其结果

高阶函数是我们可以使用的一些最强大的工具，可以帮助我们用函数式编程风格编写 JavaScript。让我们看看一些例子。

以下代码片段声明了一个名为`addDelay`的函数。该函数创建一个新的函数，该函数在控制台打印消息之前等待给定数量的毫秒。这个函数被认为是一个高阶函数，因为它返回一个函数：

```js
function addDelay(msg: string, ms: number) {
    return () => {
        setTimeout(() => {
            console.log(msg);
        }, ms);
    };
}

const delayedSayHello = addDelay("Hello world!", 500);
delayedSayHello(); // Prints "Hello world!" (after 500 ms)
```

以下代码片段声明了一个名为`addDelay`的函数。该函数创建一个新的函数，该函数将延迟（以毫秒为单位）添加到作为参数传递的另一个函数的执行中。这个函数被认为是一个高阶函数，因为它接受一个函数作为参数并返回一个函数：

```js
function addDelay(func: () => void, ms: number) {
    return () => {
        setTimeout(() => {
            func();
        }, ms);
    };
}

function sayHello() {
    console.log("Hello world!");
}

const delayedSayHello = addDelay(sayHello, 500);
delayedSayHello(); // Prints "Hello world!" (after 500 ms)
```

高阶函数是抽象常见问题解决方案的有效技术。前面的例子演示了我们可以如何使用高阶函数（`addDelay`）来给另一个函数（`sayHello`）添加延迟。这种技术允许我们抽象延迟功能，并保持`sayHello`函数或其他函数对延迟功能的实现细节无感知。

# 懒惰性

许多函数式编程语言都提供了懒加载的 API。懒加载背后的想法是，只有在无法再推迟操作时才会进行计算。以下示例声明了一个函数，允许我们在数组中查找元素。当函数被调用时，我们不会过滤数组。相反，我们声明了一个`proxy`和一个`handler`：

```js
function lazyFind<T>(arr: T[], filter: (i: T) => boolean): T {

    let hero: T | null = null;

    const proxy = new Proxy(
        {},
        {
            get: (obj, prop) => {
                console.log("Filtering...");
                if (!hero) {
                    hero = arr.find(filter) || null;
                }
                return hero ? (hero as any)[prop] : null;
            }
        }
    );

    return proxy as any;
}
```

只有在访问结果中的某个属性之后，才会调用`proxy handler`并执行过滤操作：

```js
const heroes = [
    {
        name: "Spiderman",
        powers: [
            "wall-crawling",
            "enhanced strength",
            "enhanced speed",
            "spider-Sense"
        ]
    },
    {
        name: "Superman",
        powers: [
            "flight",
            "superhuman strength",
            "x-ray vision",
            "super-speed"
        ]
    }
];

console.log("A");
const spiderman = lazyFind(heroes, (h) => h.name === "Spiderman");
console.log("B");
console.log(spiderman.name);
console.log("C");

/*
    A
    B
    Filtering...
    Spiderman
    C
*/
```

如果我们检查控制台输出，我们会看到，只有在访问`result`对象的`name`属性时，`Filtering...`消息才会被记录到控制台。前面的实现是一个非常基础的实现，但它可以帮助我们理解懒加载是如何工作的。有时，懒惰性可以提高我们应用程序的整体性能。

我们将在第九章函数式响应式编程中了解更多关于函数组合模式的内容。

# 摘要

在本章中，我们探讨了函数式编程范式的某些最基本的原则和概念。

在接下来的四章中，我们将稍微偏离函数式编程，因为我们将要深入探讨函数、异步编程以及 TypeScript/JavaScript 运行时的某些方面，例如闭包和原型。在我们学习更多关于函数式编程技术实现之前，我们需要探索这些主题。然而，如果你已经非常自信地使用函数、闭包、`this`运算符和原型，那么你应该能够跳过接下来的四章。
