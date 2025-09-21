# 第六章：函数式编程技术

在详细学习了如何处理函数、掌握了异步编程以及了解了 JavaScript 运行时的主要特性之后，我们现在完全准备好专注于函数式编程。在本章中，我们将关注最基础的函数式编程技术和模式。

我们将尽量避免使用外部库，并从头实现一些这些技术和模式。这将稍微有些繁琐，但将帮助我们完全理解这些技术是如何在内部工作的。请注意，以下实现已经简化，并不涵盖所有潜在的边缘情况。在实际的生产系统中，建议使用经过良好测试的函数式编程库。

在本章中，我们将学习以下函数式编程技术和模式：

+   组合

+   部分应用

+   柯里化

+   管道

+   无参数风格

+   递归

+   模式匹配

# 组合技术

在本节中，我们将学习一些与函数组合非常紧密相关的函数式编程技术。我们将学习组合、部分应用、柯里化和管道。

# 组成

**函数组合**是一种技术或模式，它允许我们将多个函数组合成一个更复杂的函数。

以下代码片段声明了两个简单函数：

```js
const trim = (s: string) => s.trim();
const capitalize = (s: string) => s.toUpperCase();
```

前面的代码片段中声明的两个简单函数如下：

+   用于修剪字符串的函数

+   用于将文本转换为上档的函数

我们可以通过以下方式组合前两个操作来创建一个执行这两个操作的函数：

```js
const trimAndCapitalize = (s: string) => capitalize(trim(s));
```

`trimAndCapitalize`是一个函数，它调用`trim`函数（使用`s`作为其参数）并将返回值传递给`capitalize`函数。我们可以这样调用`trimAndCapitalize`函数：

```js
trimAndCapitalize("   hello world   "); // "HELLO WORLD"
```

两个函数`f(x)`和`g(x)`的组合定义为`f(g(x))`，这正是我们在`trimAndCapitalize`函数实现中所做的。然而，这种行为可以使用高阶函数进行抽象：

```js
const compose = <T>(f: (x: T) => T, g: (x: T) => T) => (x: T) => f(g(x));
```

我们可以使用前面的函数来组合两个给定的函数：

```js
const trimAndCapitalize = compose(trim, capitalize);
```

我们可以按照以下方式调用`trimAndCapitalize`函数：

```js
trimAndCapitalize("   hello world   "); // "HELLO WORLD"
```

一个需要注意的重要事项是`g`函数的返回值被传递为`f`函数的参数。这意味着`f`只能接受一个参数（它必须是一元函数）。`f`的唯一参数的类型必须与`g`函数的返回类型相匹配。这些限制可以用`compose`函数的更正确的定义来表示：

```js
const compose = <T1, T2, T3>( f: (x: T2) => T3, g: (x: T1) => T2) => (x: T1) => f(g(x));
```

我们还可以使用`compose`函数生成的函数进行组合：

```js
const composed1 = compose(func1, func2);
const composed2 = compose(func1, func2);
const composed3 = compose(composed1, composed2);
```

请注意，整个示例包含在配套源代码中。

或者我们可以声明一个高阶函数，在单个调用中组合三个函数：

```js
const compose3 = <T1, T2, T3, T4>(
    f: (x: T3) => T4,
    g: (x: T2) => T3,
    h: (x: T1) => T2
) => (x: T1) => f(g(h(x)));
```

我们可以按照以下方式调用它：

```js
const composed1 = composeMany(func1, func2, func3);
```

我们还可以创建一个辅助函数，允许我们组合无限数量的函数：

```js
const composeMany = <T>(...functions: Array<(arg: T) => T>) =>
    (arg: any) =>
        functions.reduce((prev, curr) => {
            return curr(prev);
        }, arg);
```

我们可以按如下方式调用它：

```js
const composed1 = composeMany(func1, func2, func3, func4);
const composed2 = composeMany(func1, func2, func3, func4, func5);
```

函数组合是一个非常强大的技术，但在某些场景中可能很难实施，例如，当我们的函数不是一元函数时。然而，有其他技术，如函数式部分应用，可以帮助在这些场景中，正如我们将在下一节中看到的那样。

# 部分应用

**部分应用**是一种函数式编程技术，它允许我们在不同时间点传递函数所需的参数。

这种技术在第一眼看起来可能像是一个奇怪的想法，因为大多数软件工程师习惯于在唯一的时间点应用或调用一个函数（完整应用），而不是在多个时间点应用一个函数（部分应用）。

以下代码片段实现了一个不支持部分应用的函数，并在一个时间点调用它（提供所有所需的参数）：

```js
function add(a: number, b: number) {
    return a + b;
}

const result = add(5, 5); // All arguments are provided at the same time
console.log(result); // 10
```

以下代码片段使用高阶函数实现前面的函数，允许我们在不同时间点提供所需的参数：

```js
function add(a: number) {
 return (b: number) => {
 return a + b;
 };
}

const add5 = add(5); // The 1st argument is provided
const result = add5(5); // The 2nd argument is provided later
console.log(result); // 10
```

如前述代码片段所示，第一和第二个参数是在不同时间提供的。然而，前面的例子不能被视为函数式部分应用的例子，因为这两个函数是一元函数，我们一次提供了一个参数。

我们还可以编写一个允许其完整和部分应用的函数：

```js
function add(a: number, b?: number) {
    if (b !== undefined) {
        return a + b;
    } else {
        return (b2: number) => {
            return a + b2;
        };
    }
}

const result1 = add(5, 5); // All arguments are
console.log(result1); // 10
const add5 = add(5) as (b: number) => number; // The 1st passed
const result2 = add5(5); // The 2nd argument is passed later
console.log(result2); // 10
```

前面的例子可以被视为部分应用的例子，因为我们既可以应用带有所有参数的函数（完整应用），也可以只应用其中的一些（部分应用）。

现在我们已经了解了函数式部分应用是如何工作的，让我们关注一下为什么它是有用的。在前面的函数组合部分，我们学习了如何将名为`trim`和`capitalize`的两个函数组合成一个名为`trimAndCapitalize`的第三个函数：

```js
const trim = (s: string) => s.trim();
const capitalize = (s: string) => s.toUpperCase();
const trimAndCapitalize = compose(trim, capitalize);
```

函数组合与一元函数配合得非常好，但与二元、三元或可变参数函数配合得不是很好。我们将声明以下函数来演示这一点：

```js
const replace = (s: string, f: string, r: string) => s.split(f).join(r);
```

前面的函数可以用作替换给定字符串中的子串。不幸的是，由于它不是一元函数，因此该函数不能轻易与`compose`函数一起使用：

```js
const trimCapitalizeAndReplace = compose(trimAndCapitalize, replace); // Error
```

然而，我们可以以允许我们在不同时间点应用函数参数的方式实现该函数：

```js
const replace = (f: string, r: string) => (s: string) => s.split(f).join(r);
```

然后，我们可以无困难地使用`compose`函数：

```js
const trimCapitalizeAndReplace = compose(trimAndCapitalize, replace("/", "-"));
trimCapitalizeAndReplace(" 13/feb/1989 "); // "13-FEB-1989"
```

由于我们掌握了函数式部分应用的知识，我们可以轻松地使用 `compose` 函数，而无需担心函数的 arity（元数）。然而，启用部分应用需要大量的手动样板代码。在下一节中，我们将学习一种名为 **柯里化** 的函数式编程技术，它可以帮助我们解决这个问题。

# 柯里化

柯里化是一种函数式编程技术，它允许我们在不担心函数实现方式的情况下部分应用一个函数。柯里化是将接受多个参数的函数转换为一系列一元函数的过程。以下函数允许我们将接受两个参数 `a` 和 `b` 的函数 `fn` 转换为接受一个参数 `a` 的函数，并返回另一个接受一个参数 `b` 的函数：

```js
function curry2<T1, T2, T3>(fn: (a: T1, b: T2) => T3) {
 return (a: T1) => (b: T2) => fn(a, b);
}
```

上述函数是一个高阶函数，它允许我们的函数在部分应用的同时，保持其实现与这一关注点无关。

```js
function add(a: number, b: number) {
 return a + b;
}

const curriedAdd = curry2(add);
const add5 = curriedAdd(5);
const addResult = add5(5);
console.log(addResult); // 10
```

`curry2` 函数允许我们将二元函数转换为两个一元函数。`curry2` 函数是一个高阶函数，可以与任何二元函数一起使用。例如，在前面的代码片段中，我们将 `add` 函数传递给 `curry2` 函数，但以下示例将 `multiply` 函数传递给 `curry2` 函数：

```js
function multiply(a: number, b: number) {
 return a * b;
}

const curriedMultiply = curry2(multiply);
const multiplyBy5 = curriedMultiply(5);
const multiplyResult = multiplyBy5(5);
console.log(multiplyResult); // 25
```

在前面关于函数式部分应用的章节中，我们学习了如何使用部分应用来与不是一元的函数一起使用 `compose`。我们声明了一个名为 `replace` 的函数，并将其传递给 `compose` 函数：

```js
const replace = (f: string, r: string) => (s: string) => s.split(f).join(r);

const trimCapitalizeAndReplace = compose(
    trimAndCapitalize,
    replace("/", "-")
);
```

我们可以声明一个名为 `curry3` 的函数，它将三元函数转换为一系列三个一元函数：

```js
function curry3<T1, T2, T3, T4>(fn: (a: T1, b: T2, c: T3) => T4) {
    return (a: T1) => (b: T2) => (c: T3) => fn(a, b, c);
}
```

然后，我们可以使用 `curry3` 函数以与函数式部分应用实现细节无关的方式重写 `replace` 函数：

```js
const replace = (s: string, f: string, r: string) => s.split(f).join(r);

const curriedReplace = curry3(replace);

const trimCapitalizeAndReplace = compose(
    trimAndCapitalize,
    curriedReplace("/")("-")
);
```

请注意，整个示例包含在配套源代码中。

# strictBindCallApply

在本章前面，我们探索了几种实现部分应用的方法。然而，我们避免了一种使用 `Function.prototype.bind` 方法的替代实现。我们这样做是因为在 TypeScript 3.2 版本之前的 TypeScript 中，`bind` 方法是不安全的。如果我们安装 TypeScript 3.2 或更高版本，并在 `tsconfig.json` 文件中启用 `strictBindCallApply` 编译标志，我们就可以像下面这样使用 `bind`：

```js
const replace = (s: string, f: string, r: string) => s.split(f).join(r);
const replaceForwardSlash = replace.bind(replace, "/");
const replaceForwardSlashWithDash = replaceForwardSlash.bind(replaceForwardSlash, "-");
replaceForwardSlashWithDash("13/feb/1989");
```

如我们所见，`bind` 方法允许我们部分应用函数。我们可以重写本章前面实现的柯里化示例，并用 `bind` 方法代替柯里化函数：

```js
const compose = <T1, T2, T3>( f: (x: T2) => T3, g: (x: T1) => T2) => (x: T1) => f(g(x));
const trim = (s: string) => s.trim();
const capitalize = (s: string) => s.toUpperCase();
const trimAndCapitalize = compose(trim, capitalize);
const replace = (s: string, f: string, r: string) => s.split(f).join(r);
const replaceForwardSlashWithDash = replace.bind(replace, "/", "-");
const trimCapitalizeAndReplace = compose(trimAndCapitalize, replaceForwardSlashWithDash);
const result = trimCapitalizeAndReplace(" 13/feb/1989 ");
console.log(result); // "13-FEB-1989"
```

`strictBindCallApply` 编译标志确保调用 `bind` 方法返回的结果将具有正确的类型。在 TypeScript 3.2 版本之前的版本中，`bind` 方法的返回类型是 `any`。

# 管道

管道是一个函数或操作符，它允许我们将一个函数的输出作为另一个函数的输入。JavaScript 和 TypeScript 本身不支持管道操作符（作为一个操作符），但我们可以使用以下函数来实现我们的管道：

```js
const pipe = <T>(...fns: Array<(arg: T) => T>) =>
    (value: T) =>
        fns.reduce((acc, fn) => fn(acc), value);
```

我们将使用在本章中先前声明的`curry3`、`trim`、`capitalize`和`replace`函数：

```js
const trim = (s: string) => s.trim();
const capitalize = (s: string) => s.toUpperCase();

const replace = curry3(
 (s: string, f: string, r: string) => s.split(f).join(r)
);
```

我们可以使用`pipe`函数来声明一个新的函数：

```js
const trimCapitalizeAndReplace = pipe(
    trim,
    capitalize,
    replace("/")("-")
);

trimCapitalizeAndReplace(" 13/feb/1989 "); // "13-FEB-1989"
```

`pipe`函数确保`trim`函数的输出被传递给`capitalize`函数。然后，`capitalize`函数的返回值被传递给`replace`函数，该函数已经部分应用。

已有官方提案建议为 JavaScript 添加一个新操作符，称为管道操作符（`|>`）。此操作符将允许我们以如下方式原生化地使用管道：

```js
const result = " 13/feb/1989 "
  |> trim
  |> capitalize
  |> replace("/")("-");
```

请参阅管道操作符提案（[`github.com/tc39/proposal-pipeline-operator`](https://github.com/tc39/proposal-pipeline-operator)）以了解更多信息。

请注意，整个示例都包含在配套源代码中。

# 其他技术

在本节中，我们将探讨一些与函数组合不直接相关的其他函数式编程技术。

# 无参数风格

**无参数风格**，也称为**隐式编程**，是一种编程风格，其中函数声明不声明它们操作的参数（或*点*）。

以下代码片段声明了一些用于确定一个人是否有资格参加选举的函数：

```js
interface Person {
 age: number;
 birthCountry: string;
 naturalizationDate: Date;
}

const OUR_COUNTRY = "Ireland";
const wasBornInCountry = (person: Person) => person.birthCountry === OUR_COUNTRY;
const wasNaturalized = (person: Person) => Boolean(person.naturalizationDate);
const isOver18 = (person: Person) => person.age >= 18;
const isCitizen = (person: Person) => wasBornInCountry(person) || wasNaturalized(person);
const isEligibleToVote = (person: Person) => isOver18(person) && isCitizen(person);

isEligibleToVote({
    age: 27,
    birthCountry: "Ireland",
    naturalizationDate: new Date(),
});
```

之前的代码片段没有使用本章我们已经学到的任何函数式编程技术。以下代码片段使用部分应用等技术为相同问题实现了一个替代解决方案。此代码片段声明了两个函数，分别命名为`both`和`either`，可用于确定变量是否匹配由提供给这些函数的某些或所有函数指定的要求：

`either`和`both`函数是某些实际代数数据类型的简化实现。我们将在下一章中了解更多关于代数数据类型和范畴论的内容。

```js
const either = <T1>(
 funcA: (a: T1) => boolean,
 funcB: (a: T1) => boolean
) => (arg: T1) => funcA(arg) || funcB(arg);

const both = <T1>(
 funcA: (a: T1) => boolean,
 funcB: (a: T1) => boolean
) => (arg: T1) => funcA(arg) && funcB(arg);

interface Person {
 age: number;
 birthCountry: string;
 naturalizationDate: Date;
}

const OUR_COUNTRY = "Ireland";
const wasBornInCountry = (person: Person) => person.birthCountry === OUR_COUNTRY;
const wasNaturalized = (person: Person) => Boolean(person.naturalizationDate);
const isOver18 = (person: Person) => person.age >= 18;

// Point-free style

const isCitizen = either(wasBornInCountry, wasNaturalized);
const isEligibleToVote = both(isOver18, isCitizen);

isEligibleToVote({
 age: 27,
 birthCountry: "Ireland",
 naturalizationDate: new Date(),
});
```

如我们所见，`isCitizen`和`isElegibleToVote`函数接受一些函数作为参数，但它们没有说明期望的参数数据类型。例如，我们不必写以下内容：

```js
const isCitizen = (person: Person) => wasBornInCountry(person) || wasNaturalized(person);
```

我们可以写出以下内容：

```js
const isCitizen = either(wasBornInCountry, wasNaturalized);
```

这种避免引用函数参数的风格被称为无参数风格，它比更传统的函数声明风格具有许多优点：

+   它使程序更简单、更简洁。这并不总是好事，但有时是。

+   它通过仅关注组合的函数来简化算法的理解。我们可以在数据参数干扰的情况下更好地理解正在发生的事情。

+   它迫使我们更多地思考数据的使用方式，而不是使用的数据类型。

+   它帮助我们以通用构建块的方式思考我们的函数，这些构建块可以与不同类型的数据一起工作，而不是将它们视为对一种数据类型的操作。

请注意，整个示例包含在配套源代码中。

# 递归

调用自身的函数被称为递归函数。以下是一个递归函数，它允许我们计算给定数字`n`的阶乘。阶乘是所有小于或等于给定数字`n`的正整数的乘积：

```js
const factorial = (n: number): number => (n === 0) ? 1 : (n * factorial(n - 1));
```

我们可以如下调用前面的函数：

```js
factorial(5); // 120
```

通常情况下，你应该尝试不使用递归来实现函数。使用递归应该仔细考虑，因为 JavaScript 运行时在处理递归时并不非常高效，因为在递归函数调用中，每次函数调用都会在栈上添加一个帧。

# 模式匹配

模式匹配允许你将一个值（或一个对象）与一些模式进行匹配，以选择代码的某个分支。在函数式语言中，模式匹配可以用来匹配标准原始值，如字符串。TypeScript 允许我们使用字面量类型和控制流分析来实现模式匹配。

例如，我们可以定义三个类型，分别命名为`Circle`、`Square`和`Rectangle`。然后，我们可以定义一个新的类型，命名为`Shape`，它是`Circle`、`Square`和`Rectangle`类型的并集：

```js
const enum ShapeKind {
    circle = "circle",
    square = "square",
    rectangle = "rectangle",
}

type Circle = { kind: ShapeKind.circle, radius: number };
type Square = { kind: ShapeKind.square, size: number };
type Rectangle = { kind: ShapeKind.rectangle, w: number, h: number };
type Shape = Circle | Square | Rectangle;
```

然后，我们可以实现接受`Shape`类型参数的函数，并使用模式匹配来识别`Shape`是`Circle`、`Square`还是`Rectangle`：

```js
function area(shape: Shape) {
    switch(shape.kind) {
        case ShapeKind.circle:
            return shape.radius ** 2;
        case ShapeKind.square:
            return shape.size ** 2;
        case ShapeKind.rectangle:
            return shape.w * shape.h;
        default:
            throw new Error("Invalid shape!"); 
    }
}
```

在 TypeScript 2.0 之前的版本中，模式匹配是不可能的，因为控制流分析和字面量类型不可用。

# 摘要

在本章中，我们学习了一些主要的函数式编程技术和模式，包括函数式组合、函数式部分应用和柯里化。

在下一章中，我们将学习关于范畴论的内容。我们将学习如何处理一些代数数据类型以及它们如何帮助我们使 TypeScript 应用程序更加健壮。
