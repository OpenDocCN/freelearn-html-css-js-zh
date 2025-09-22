# 与函数一起工作

在第一章 *介绍 TypeScript* 中，我们学习了函数的基础知识。函数是 TypeScript 中任何应用程序的基本构建块，它们足够强大，值得用整整一章来探索它们的潜力。

在本章中，我们将深入学习如何与函数一起工作。本章分为两个主要部分。第一部分首先快速回顾一些基本概念，然后转向函数的一些不太常见的特性和用例。第一部分涵盖了以下概念：

+   函数声明和函数表达式

+   函数类型

+   带有可选参数的函数

+   带有默认参数的函数

+   带有可变参数的函数

+   函数重载

+   特殊化的重载签名

+   函数作用域

+   立即调用的函数

+   标签函数和标签模板

第二部分专注于 TypeScript 的异步编程能力，包括以下概念：

+   回调和高级函数

+   箭头函数

+   回调地狱

+   承诺

+   生成器

+   异步函数（`async`和`await`）

# 在 TypeScript 中与函数一起工作

本节主要关注函数、参数和参数的声明和使用。

# 函数声明和函数表达式

在第一章中，我们介绍了显式声明带有（命名函数）或没有（未命名或匿名函数）名称的函数的可能性，但我们没有提到我们也在使用两种不同类型的函数。

在以下示例中，命名函数`greetNamed`是一个**函数声明**，而`greetUnnamed`是一个**函数表达式**。现在，请忽略前两行，它们包含两个`console.log`语句：

```js
console.log(greetNamed("John")); // OK 
console.log(greetUnnamed("John")); // Error 

function greetNamed(name: string): string { 
    return `Hi! ${name}`; 
} 

let greetUnnamed = function(name: string): string { 
    return `Hi! ${name}`; 
}; 
```

我们可能会认为前面的函数是相同的，但它们的运行方式不同。JavaScript 解释器可以在解析时评估函数声明。另一方面，函数表达式是赋值的一部分，只有在赋值完成后才会被评估。

这些函数行为不同的主要原因是一个称为**变量提升**的过程。我们将在本章后面更深入地了解变量提升过程。

如果我们将前面的 TypeScript 代码片段编译成 JavaScript 并在网页浏览器中尝试执行它，我们将观察到第一个`console.log`调用是有效的。这是因为 JavaScript 了解声明函数，可以在程序执行之前解析它。

然而，第二个警报语句将抛出异常，这表明`greetUnnamed`不是一个函数。异常抛出是因为`greetUnnamed`的赋值必须在函数可以评估之前完成。

# 函数类型

我们已经知道，可以通过使用可选类型注解显式地声明应用程序中元素的类型：

```js
function greetNamed(name: string): string { 
    return `Hi! ${name}`; 
} 
```

在前面的函数中，我们已经指定了参数名称的类型（字符串）及其返回类型（字符串）。有时，我们不仅需要指定函数元素的类型，还需要指定函数本身。让我们看一个例子：

```js
let greetUnnamed: (name: string) => string; 

greetUnnamed = function(name: string): string { 
    return `Hi! ${name}`; 
}; 
```

在前面的例子中，我们已经声明了`greetUnnamed`变量及其类型。`greetUnnamed`类型是一个函数类型，它接受一个名为`name`的字符串变量作为其唯一参数，并在调用后返回一个字符串。在声明变量之后，一个类型必须等于变量类型的函数被分配给它。

我们也可以在单行中声明`greetUnnamed`类型并将函数分配给它，而不是像上一个例子那样在两行中分别声明：

```js
let greetUnnamed: (name: string) => string = function(name: string): string { 
    return `Hi! ${name}`; 
}; 
```

就像上一个例子一样，前面的代码片段也声明了一个变量`greetUnnamed`及其类型。`greetUnnamed`类型是一个函数类型，它接受一个名为`name`的字符串变量作为其唯一参数，并在调用后返回一个字符串。我们将在声明变量的同一行中将函数分配给这个变量。分配的函数的类型必须与变量的类型匹配。

在前面的例子中，我们声明了`greetUnnamed`变量的类型，然后将其值分配给一个函数。函数的类型可以从分配的函数中推断出来，因此添加冗余的类型注解是不必要的。我们这样做是为了帮助您理解这一部分，但重要的是要提到，添加冗余的类型注解可能会使我们的代码更难阅读，并且被认为是不良的实践。

# 函数参数中的尾随逗号

尾随逗号是指在函数最后一个参数之后使用的逗号。在函数的最后一个参数后使用逗号可能很有用，因为程序员在通过添加额外的参数修改现有函数时，常常会忘记添加逗号。

例如，以下函数只接受一个参数，并且没有使用尾随逗号：

```js
function greetWithoutTralingCommas( 
    name: string 
): string { 
    return `Hi! ${name}`; 
} 
```

经过一段时间后，我们可能需要向前面的函数添加一个参数。一个常见的错误是在声明新参数时忘记在第一个参数后添加逗号：

```js
function updatedGreetWithoutTralingCommas( 
    name: string 
    surname: string, // Error 
): string { 
    return `Hi! ${name} ${surname}`; 
} 
```

在函数的第一个版本中使用尾随逗号可以帮助我们避免这个常见的错误：

```js
function greetWithTralingCommas( 
    name: string, 
): string { 
    return `Hi! ${name}`; 
} 
```

使用尾随逗号消除了在添加新参数时忘记添加逗号的可能性：

```js
function updatedGreetWithTralingCommas( 
    name: string, 
    surname: string, 
): string { 
    return `Hi! ${name} ${surname}`; 
} 
```

如果我们忘记添加逗号，TypeScript 将抛出错误，因此与使用 JavaScript 相比，尾随逗号不是那么必要。尾随逗号是可选的，但许多 JavaScript 和 TypeScript 工程师认为使用它们是一种良好的实践。

# 带有可选参数的函数

与 JavaScript 不同，如果尝试调用一个函数而没有提供其签名声明的确切参数数量和类型，TypeScript 编译器将抛出一个错误。让我们看一个代码示例来演示这一点：

```js
function add(foo: number, bar: number, foobar: number): number { 
    return foo + bar + foobar; 
} 
```

前面的函数名为 `add`，它将接受三个数字作为参数，参数名称分别为 `foo`、`bar` 和 `foobar`。如果我们尝试调用此函数而不提供恰好三个数字，我们将得到一个编译错误，表明提供的参数与函数的签名不匹配：

```js
add(); // Error, expected 3 arguments, but got 0\. 
add(2, 2); // Error, expected 3 arguments, but got 2\. 
add(2, 2, 2); // OK, returns 6 
```

在某些情况下，我们可能希望能够在不提供所有参数的情况下调用函数。TypeScript 在函数中提供了可选参数，以帮助我们增加函数的灵活性并克服这些情况。

我们可以通过在参数名称后附加 `?` 字符来向 TypeScript 编译器指示我们希望函数的参数是可选的。让我们更新前面的函数，将必需的 `foobar` 参数转换为可选参数：

```js
function add(foo: number, bar: number, foobar?: number): number { 
    let result = foo + bar; 
    if (foobar !== undefined) { 
        result += foobar; 
    } 
    return result; 
} 
```

注意我们如何将 `foobar` 参数名称更改为 `foobar?`，以及我们如何在函数内部检查 `foobar` 的类型，以确定该参数是否作为参数传递给了函数。在实施这些更改后，TypeScript 编译器将允许我们在提供两个或三个参数时调用该函数而不会出错：

```js
add(); // Error, expected 2-3 arguments, but got 0\. 
add(2, 2); // OK, returns 4 
add(2, 2, 2); // OK, returns 6 
```

需要注意的是，可选参数必须始终位于函数参数列表中的必选参数之后。

# 带有默认参数的函数

当一个函数有一些可选参数时，我们必须检查是否向函数传递了参数（就像我们在前面的例子中所做的那样），以防止潜在的错误。

在某些情况下，为参数提供一个默认值（而不是将其作为可选参数）可能更有用。让我们重写 `add` 函数（来自上一节），使用内联 `if` 结构：

```js
function add(foo: number, bar: number, foobar?: number): number { 
    return foo + bar + (foobar !== undefined ? foobar : 0); 
} 
```

前面的函数本身没有问题，但我们可以通过为 `foobar` 参数提供一个默认值来提高其可读性，而不是使用可选参数：

```js
function add(foo: number, bar: number, foobar: number = 0): number { 
    return foo + bar + foobar; 
} 
```

要表示一个函数参数是可选的，我们需要在声明函数签名时使用 `=` 运算符提供默认值。在编译前面的代码示例后，TypeScript 编译器将在 JavaScript 输出中生成一个 `if` 结构，以设置 `foobar` 参数的默认值，如果它没有被作为函数的参数传递：

```js
function add(foo, bar, foobar) { 
    if (foobar === void 0) { foobar = 0; } 
    return foo + bar + foobar; 
} 
```

这很棒，因为 TypeScript 编译器为我们生成了防止潜在运行时错误的代码。

`void 0`参数由 TypeScript 编译器用于检查变量是否未定义。虽然大多数开发者使用`undefined`变量来执行这种检查，但大多数编译器使用`void 0`，因为它始终评估为`undefined`。与未定义变量进行比较的安全性较低，因为其值可能已被修改，如下面的代码片段所示：

`function test() {`

`    var undefined = 2; // 2`

`    console.log(undefined === 2); // true`

`}`

就像可选参数一样，默认参数必须始终位于函数参数列表中的任何必需参数之后。

# 带有 REST 参数的函数

我们已经学会了如何使用可选和默认参数来增加调用函数的方式。让我们再次回到之前的例子：

```js
function add(foo: number, bar: number, foobar: number = 0): number { 
    return foo + bar + foobar; 
} 
```

我们已经学会了如何使用两个或三个参数调用`add`函数，但如果我们想允许其他开发者向我们的函数传递四个或五个参数呢？我们就必须添加两个额外的默认或可选参数。如果我们想允许他们传递他们需要的任意数量的参数呢？解决这种可能场景的方法是使用 REST 参数。REST 参数语法允许我们将不定数量的参数表示为一个数组：

```js
function add(...foo: number[]): number { 
    let result = 0; 
    for (let i = 0; i < foo.length; i++) { 
        result += foo[i]; 
    } 
    return result; 
} 
```

正如前一个代码片段所示，我们将函数参数`foo`、`bar`和`foobar`替换为只有一个参数名为`foo`。请注意，参数`foo`的名称前面有一个省略号（一组三个点，而不是实际的省略号字符）。REST 参数必须是数组类型，否则我们将得到编译错误。现在我们可以使用我们需要的任意数量的参数来调用`add`函数：

```js
add(); // 0 
add(2); // 2 
add(2, 2); // 4 
add(2, 2, 2); // 6 
add(2, 2, 2, 2); // 8 
add(2, 2, 2, 2, 2); // 10 
add(2, 2, 2, 2, 2, 2); // 12 
```

虽然函数可以接受的理论最大参数数量没有具体限制，但当然存在实际限制。这些限制完全取决于实现，并且很可能会也取决于我们调用函数的确切方式。

JavaScript 函数有一个内置对象，称为`arguments`对象。该对象作为名为`arguments`的局部变量可用。`arguments`变量包含一个类似于数组的对象，其中包含函数被调用时使用的参数。

`arguments`对象暴露了一些标准数组提供的方法和属性，但并非全部。有关其特性的更多信息，请参阅[`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/arguments`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/arguments)文档。

如果我们检查 JavaScript 的输出，我们会注意到 TypeScript 会迭代`arguments`对象，将值添加到`foo`变量中：

```js
function add() { 
    var foo = []; 
    for (var _i = 0; _i < arguments.length; _i++) { 
        foo[_i - 0] = arguments[_i]; 
    } 
    var result = 0; 
    for (var i = 0; i < foo.length; i++) { 
        result += foo[i]; 
    } 
    return result; 
} 
```

我们可以争辩说，这是对函数参数的额外、不必要的迭代。尽管很难想象这种额外的迭代会成为性能问题，但如果您认为这可能会影响您应用程序的性能，您可能希望考虑避免使用 REST 参数，而是使用数组作为函数的唯一参数：

```js
function add(foo: number[]): number { 
    let result = 0; 
    for (let i = 0; i < foo.length; i++) { 
        result += foo[i]; 
    } 
    return result; 
} 
```

之前的功能只接受一个数字数组作为其唯一参数。调用 API 将与 REST 参数略有不同，但我们将有效地避免在函数参数列表上额外的迭代：

```js
add(); // Error, expected 1 arguments, but got 0\. 
add(2); // Error, '2' is not assignable to parameter of type 'number[]'. 
add(2, 2); // Error, expected 1 arguments, but got 2\. 
add(2, 2, 2); // Error, expected 1 arguments, but got 3\. 

add([]); // returns 0 
add([2]); // returns 2 
add([2, 2]); // returns 4 
add([2, 2, 2]); // returns 6 
```

# 函数重载

函数重载，或方法重载，是创建具有相同名称但参数数量或类型不同的多个方法的能力。在 TypeScript 中，我们可以通过指定一个函数的所有函数签名（称为**重载签名**），然后是一个签名（称为**实现签名**）来重载一个函数。让我们看一个例子：

```js
function test(name: string): string; // overloaded signature 
function test(age: number): string; // overloaded signature 
function test(single: boolean): string; // overloaded signature 
function test(value: (string|number|boolean)): string { // implementation signature 
    switch (typeof value) { 
        case "string": 
            return `My name is ${value}.`; 
        case "number": 
            return `I'm ${value} years old.`; 
        case "boolean": 
            return value ? "I'm single." : "I'm not single."; 
        default: 
            throw new Error("Invalid Operation!"); 
    } 
} 
```

正如我们可以在前面的例子中看到的那样，我们通过添加一个只接受字符串作为唯一参数的签名、另一个接受数字的函数和一个最终接受布尔值作为唯一参数的签名，三次重载了函数 test。重要的是要注意，所有函数签名都必须兼容；因此，如果例如，一个签名试图返回一个数字，而另一个试图返回一个字符串，我们将得到一个编译错误：

```js
function test(name: string): string; 
function test(age: number): number; // Error 
function test(single: boolean): string; 
function test(value: (string|number|boolean)): string { 
    switch (typeof value) { 
        case "string": 
            return `My name is ${value}.`; 
        case "number": 
            return `I'm ${value} years old.`; 
        case "boolean": 
            return value ? "I'm single." : "I'm not single."; 
        default: 
            throw new Error("Invalid Operation!"); 
    } 
} 
```

实现签名必须与所有重载签名兼容，始终是列表中的最后一个，并且其参数的类型必须是`any`类型或联合类型。

调用函数并提供不匹配重载签名中声明的任何类型的参数将导致我们得到一个编译错误：

```js
test("Remo"); // returns "My name is Remo." 
test(26); // returns "I'm 26 years old."; 
test(false); // returns "I'm not single."; 
test({ custom: "custom" }); // Error 
```

# 专用重载签名

我们可以使用专用签名创建具有相同名称和参数数量但不同返回类型的多个方法。要创建专用签名，我们必须使用字符串指示函数参数的类型。字符串字面量用于识别调用了哪个函数重载：

```js
interface Document { 
    createElement(tagName: "div"): HTMLDivElement; // specialized 
    createElement(tagName: "span"): HTMLSpanElement; // specialized 
    createElement(tagName: "canvas"): HTMLCanvasElement; // specialized 
    createElement(tagName: string): HTMLElement; // non-specialized 
} 
```

在前面的例子中，我们为名为`createElement`的函数声明了三个**专用重载签名**和一个**非专用签名**。

当我们在对象中声明一个专用签名时，它必须可以分配给该对象中至少一个非专用签名。这可以从前面的例子中观察到，因为`createElement`属性属于一个包含三个专用签名的类型，所有这些都可以分配给该类型的非专用签名。

当编写重载声明时，我们必须将非专用签名放在最后。

# 函数作用域

低级语言，如 C 语言，具有低级内存管理功能。在具有更高抽象级别的编程语言中，如 TypeScript，变量在创建时分配内存，当不再使用时自动从内存中清除。清理内存的过程称为 **垃圾回收**，由 JavaScript 运行时的垃圾回收器执行。

垃圾回收器做得很好，但认为它总能防止我们遇到内存泄漏是错误的。垃圾回收器会在变量超出作用域时从内存中清除变量。理解 TypeScript 作用域的工作方式很重要，因此我们现在将查看变量的生命周期。

一些编程语言使用程序源代码的结构来确定我们引用的是哪个变量（**词法作用域**），而另一些则使用程序调用栈的运行时状态来确定我们引用的是哪个变量（**动态作用域**）。大多数现代编程语言使用词法作用域（包括 TypeScript）。词法作用域通常比动态作用域更容易被人类和分析工具理解。

在大多数词法作用域编程语言中，变量的作用域限定在代码块（由花括号 `{}` 分隔的代码部分）内，而在 TypeScript（和 JavaScript）中，变量的作用域限定在函数内，如下面的代码片段所示：

```js
function foo(): void { 
    if (true) { 
        var bar: number = 0; 
    } 
    console.log(bar); 
} 

foo(); // 0 
```

前面的名为 `foo` 的函数包含一个 `if` 结构。我们在 `if` 结构内部声明了一个名为 `bar` 的数值变量，后来我们尝试使用 `log` 函数显示 `bar` 变量的值。

我们可能会认为前面的代码示例在第五行会抛出错误，因为当调用 `log` 函数时 `var` 变量应该超出作用域。然而，如果我们调用 `foo` 函数，`log` 函数将能够无错误地显示 `bar` 变量，因为函数内部的所有变量都将处于整个函数体的作用域内，即使它们位于另一个代码块内（除了函数块）。

这可能看起来有些令人困惑，但一旦我们知道在运行时，所有变量声明都会在函数执行之前移动到函数顶部，这种行为就很容易理解了。这种行为被称为 **提升**。

TypeScript 编译成 JavaScript 然后执行——这意味着 TypeScript 应用程序在运行时是一个 JavaScript 应用程序，因此，当我们提到 TypeScript 运行时，我们实际上是在谈论 JavaScript 运行时。我们将在第六章 理解运行时中深入探讨运行时。

在执行前面的代码片段之前，运行时会将 `bar` 变量的声明移动到我们的函数顶部：

```js
function foo() { 
    var bar; 
    if (true) { 
        bar = 0; 
    } 
    console.log(bar); 
} 

foo(); // 0 
```

这就是为什么可以在变量声明之前使用变量的原因。让我们来看一个例子：

```js
function foo(): void { 
    bar = 0; 
    var bar: number; 
    console.log(bar); 
} 

foo(); // 0 
```

在前面的代码片段中，我们声明了一个`foo`函数，并在其函数体内将值`0`赋给名为`bar`的变量。此时，该变量尚未声明。在第二行，我们声明了变量`bar`及其类型。在最后一行，我们使用 alert 函数显示`bar`的值。

因为在函数内部（除了另一个函数）的任何地方声明变量（等同于在函数顶部声明），`foo`函数在运行时会转换为以下形式：

```js
function foo(): void { 
    var bar: number; 
    bar = 0; 
    console.log(bar); 
} 

foo(); // 0 
```

由于具有像 Java 或 C#这样的具有块作用域的编程语言背景的开发者不习惯函数作用域，这是 JavaScript 最被批评的特性之一。ECMAScript 6 规范的开发负责人对此有所了解，因此他们引入了`let`和`const`关键字。

`let`关键字允许我们将变量的作用域设置为块（`if`、`while`、`for`等），而不是函数。我们可以更新本节中的第一个示例来展示`let`是如何工作的：

```js
function foo(): void { 
    if (true) { 
        let bar: number = 0; 
        bar = 1; 
    } 
    console.log(bar); // Error 
} 
```

现在，`bar`变量使用`let`关键字声明，因此它只能在`if`块内部访问。变量不会被提升到`foo`函数的顶部，并且不能在`if`语句外部通过`alert`函数访问。

虽然`const`声明的变量遵循与`let`声明的变量相同的范围规则，但它们不能被重新赋值：

```js
function foo(): void { 
    if (true) { 
        const bar: number = 0; 
        bar = 1; // Error 
    } 
    alert(bar); // Error 
} 
```

如果我们尝试编译前面的代码片段，我们会得到一个错误，因为`bar`变量在`if`语句外部不可访问（就像我们使用`let`关键字时一样），并且当我们尝试给`bar`变量赋新值时，会出现新的错误。第二个错误发生是因为一旦变量已经被初始化，就无法给常量变量赋新值。

使用`const`关键字声明的变量不能重新赋值，但不是不可变的。当我们说一个变量是不可变的，我们的意思是它不能被修改。我们将在第七章中了解更多关于不可变性的内容，*使用 TypeScript 进行函数式编程*。

# 立即调用的函数

立即调用的函数表达式（**IIFE**）是一种设计模式，它使用函数作用域创建词法作用域。IIFE 可以用来避免在块内部提升变量，或者防止我们污染全局作用域——例如：

```js
let bar = 0; // global 

(function() { 
    let foo: number = 0; // In scope of this function 
    bar = 1; // Access global scope 
    console.log(bar); // 1 
    console.log(foo); // 0 
})(); 

console.log(bar); // 1 
console.log(foo); // Error 
```

在前面的例子中，我们用立即执行函数表达式（IIFE）包裹了变量的声明（`foo`）。`foo`变量的作用域限定在 IIFE 函数内部，并且不在全局作用域中，这解释了当我们尝试在最后一行访问它时抛出的错误。

`bar` 变量是全局的。因此，它可以从 IIFE 函数的内部和外部访问。

我们还可以将变量传递给 IIFE，以更好地控制其作用域外变量的创建：

```js
let bar = 0; // global 
let topScope = window; 

(function(global: any) { 
    let foo: number = 0; // In scope of this function 
    console.log(global.bar); // 0 
    global.bar = 1; // Access global scope 
    console.log(global.bar); // 1 
    console.log(foo); // 0 
})(topScope); 

console.log(bar); // 1 
console.log(foo); // Error 
```

此外，IIFE 可以帮助我们同时允许对方法进行公共访问，同时保留函数内部定义的变量的隐私。让我们来看一个例子：

```js
class Counter { 
    private _i: number; 
    public constructor() { 
        this._i = 0; 
    } 
    public get(): number { 
        return this._i; 
    } 
    public set(val: number): void { 
        this._i = val; 
    } 
    public increment(): void { 
        this._i++; 
    } 
} 

let counter = new Counter(); 
console.log(counter.get()); // 0 
counter.set(2); 
console.log(counter.get()); // 2 
counter.increment(); 
console.log(counter.get()); // 3 
console.log(counter._i); // Error: Property '_i' is private 
```

我们定义了一个名为 `Counter` 的类，该类有一个名为 `_i` 的私有数值属性。该类还具有获取和设置私有 `_i` 属性值的函数。

按照惯例，TypeScript 和 JavaScript 开发者通常使用以下划线（`_`）开头命名的名称来命名私有变量。

我们还创建了一个 `Counter` 类的实例，并调用了 `set`、`get` 和 `increment` 方法来观察一切是否按预期工作。如果我们尝试访问 `Counter` 实例中的 `_i` 属性，我们将得到一个错误，因为该变量是 `private`。

如果我们将前面的 TypeScript 代码（仅类定义）编译并检查生成的 JavaScript 代码，我们将看到以下内容：

```js
var Counter = (function() { 
    function Counter() { 
        this._i = 0; 
    } 
    Counter.prototype.get = function() { 
        return this._i; 
    }; 
    Counter.prototype.set = function(val) { 
        this._i = val; 
    }; 
    Counter.prototype.increment = function() { 
        this._i++; 
    }; 
    return Counter; 
})(); 
```

此生成的 JavaScript 代码在大多数场景下将完美运行，但如果我们在浏览器中执行它并尝试创建 `Counter` 的实例并访问其 `_i` 属性，我们将不会得到任何错误，因为 TypeScript 不会为我们生成运行时私有属性。有时我们需要以某种方式编写我们的类，以便某些属性在运行时是私有的——例如，如果我们发布一个将被 JavaScript 开发者使用的库。

我们也可以使用 IIFE 同时允许对方法进行公共访问，同时保留函数内部定义的变量的隐私：

```js
var Counter = (function() { 
    var _i: number = 0; 
    function Counter() { 
        // 
    } 
    Counter.prototype.get = function() { 
        return _i; 
    }; 
    Counter.prototype.set = function(val: number) { 
        _i = val; 
    }; 
    Counter.prototype.increment = function() { 
        _i++; 
    }; 
    return Counter; 
})(); 
```

在前面的例子中，一切几乎与 TypeScript 生成的 JavaScript 相同，但现在 `_i` 变量是 `Counter` 闭包中的一个对象，而不是 `Counter` 类的一个属性。

闭包是引用独立（自由）变量的函数。换句话说，闭包中定义的函数*记住*了其创建时的环境（作用域内的变量）。我们将在第六章，*理解运行时*中了解更多关于闭包的内容。

如果我们在浏览器中运行生成的 JavaScript 输出并尝试直接调用 `_i` 属性，我们会注意到该属性现在在运行时是私有的：

```js
let counter = new Counter(); 
console.log(counter.get()); // 0 
counter.set(2); 
console.log(counter.get()); // 2 
counter.increment(); 
console.log(counter.get()); // 3 
console.log(counter._i); // undefined 
```

在某些情况下，我们需要对作用域和闭包有精确的控制，我们的代码最终会看起来更像 JavaScript。只要我们将应用程序组件（类、模块等）编写为供其他 TypeScript 组件使用，我们就很少需要担心实现运行时私有属性。我们将在第六章，*理解运行时*中深入探讨 TypeScript 的运行时。

# 标签函数和标签模板

在 TypeScript 中，我们可以使用如下模板字符串：

```js
let name = "remo"; 
let surname = "jansen"; 
let html = `<h1>${name} ${surname}</h1>`; 
```

我们可以使用模板字符串创建一种特殊类型的函数，称为**标签函数**。

我们可以使用标签函数来扩展或修改模板字符串的标准行为。当我们将标签函数应用于模板字符串时，模板字符串就变成了一个标签模板。

我们将实现一个名为 `htmlEscape` 的标签函数。要使用标签函数，我们必须使用函数名后跟一个模板字符串：

```js

    let html = htmlEscape `<h1>${name} ${surname}</h1>`;

```

标签模板必须返回一个字符串并接受以下参数：

+   包含模板字符串中所有静态字面量（在先前的示例中为 `<h1>` 和 `</h1>`）的 `TemplateStringsArray` 作为第一个参数传递。

`TemplateStringsArray` 类型由 `lib.d.ts` 文件声明。我们将在第九章*自动化您的开发工作流程*中了解更多关于 `lib.d.ts` 文件的内容。

+   可变参数作为第二个参数传递。可变参数包含模板字符串中的所有值（在先前的示例中为 `name` 和 `surname`）。

标签函数的签名如下所示：

```js
tag(literals: TemplateStringsArray, ...placeholders: any[]): string; 
```

让我们实现 `htmlEscape` 标签函数：

```js
function htmlEscape(literals: TemplateStringsArray, ...placeholders: any[]) { 
    let result = ""; 
    for (let i = 0; i < placeholders.length; i++) { 
        result += literals[i]; 
        result += placeholders[i] 
            .replace(/&/g, "&amp;") 
            .replace(/"/g, "&quot;") 
            .replace(/"/g, "'") 
            .replace(/</g, "&lt;") 
            .replace(/>/g, "&gt;"); 
    } 
    result += literals[literals.length - 1]; 
    return result; 
} 
```

然后，我们可以按如下方式调用该函数：

```js
let html = htmlEscape `<h1>${name} ${surname}</h1>`; 
```

模板字符串包含值和字面量。`htmlEscape` 函数遍历它们，并确保在值中转义 HTML 代码，以避免可能的代码注入攻击。

使用标签函数的主要好处是它允许我们创建自定义模板字符串处理器。

# TypeScript 中的异步编程

现在我们已经了解了如何使用函数，我们将探讨如何结合一些原生 API 使用它们来编写异步应用程序。

# 回调与高阶函数

在 TypeScript 中，函数可以作为参数传递给另一个函数。函数也可以由另一个函数返回。传递给另一个函数作为参数的函数称为**回调**。接受函数作为参数（回调）或返回函数的函数称为**高阶函数**。

回调通常是匿名函数。它们可以在传递给高阶函数之前声明，如下面的示例所示：

```js
var foo = function() { // callback 
  console.log("foo"); 
} 

function bar(cb: () => void) { // higher order function 
  console.log("bar"); 
  cb(); 
} 

bar(foo); // prints "bar" then prints "foo" 
```

然而，回调是在行内声明的，与传递给高阶函数的点相同，如下面的示例所示：

```js
bar(() => { 
  console.log("foo"); 
}); // prints "bar" then prints "foo" 
```

# 箭头函数

在 TypeScript 中，我们可以使用 `function` 表达式或箭头函数声明一个函数。箭头函数的语法比函数表达式更短，并且词法绑定 `this` 操作符的值。

在 TypeScript 和 JavaScript 中，`this` 操作符的行为与其他流行编程语言略有不同。当我们使用 TypeScript 定义一个类时，我们可以使用 `this` 操作符来引用该类。让我们看一个例子：

```js
class Person { 
    private _name: string; 
    constructor(name: string) { 
        this._name = name; 
    } 
    public greet() { 
        console.log(`Hi! My name is ${this._name}`); 
    } 
} 

let person = new Person("Remo"); 
person.greet(); // "Hi! My name is Remo" 
```

我们已经定义了一个包含一个名为 `name` 的字符串类型属性的 `Person` 类。该类有一个构造函数和一个 `greet` 方法。我们创建了一个名为 `person` 的实例并调用了 `greet` 方法，该方法内部使用 `this` 操作符来访问 `_name` 属性。在 `greet` 方法内部，`this` 操作符指向包含 `greet` 方法的对象。

在使用 `this` 操作符时，我们必须小心，因为在某些场景中，它可能指向错误值。让我们向之前的示例添加一个额外的方法：

```js
class Person { 
    private _name: string; 
    constructor(name: string) { 
        this._name = name; 
    } 
    public greet() { 
        alert(`Hi! My name is ${this._name}`); 
    } 
    public greetDelay(time: number) { 
        setTimeout(function() { 
            alert(`Hi! My name is ${this._name}`); // Error 
        }, time); 
    } 
} 
```

在 `greetDelay` 方法中，我们执行的操作几乎与 `greet` 方法执行的操作相同。这次，函数接受一个名为 `time` 的参数，该参数用于延迟 `greet` 消息。

要延迟一个消息，我们使用 `setTimeout` 函数和一个回调。一旦我们定义了一个匿名函数（回调），`this` 关键字就会改变其值并开始指向匿名函数，这也解释了为什么 TypeScript 编译器会抛出错误。

如前所述，箭头函数表达式按词法绑定 `this` 操作符的值。这意味着它 `允许` 我们在不改变 `this` 操作符的值的情况下添加一个函数。让我们将之前示例中的函数表达式替换为箭头函数：

```js
class Person { 
    private _name: string; 
    constructor(name: string) { 
        this._name = name; 
    } 
    public greet() { 
        alert(`Hi! My name is ${this._name}`); 
    } 
    public greetDelay(time: number) { 
        setTimeout(() => { 
            alert(`Hi! My name is ${this._name}`); // OK 
        }, time); 
    } 
} 

let person = new Person("Remo"); 
person.greet(); // "Hi! My name is Remo" 
person.greetDelay(1000); // "Hi! My name is Remo" 
```

通过使用箭头函数，我们可以确保 `this` 操作符仍然指向 `Person` 实例，而不是 `setTimeout` 回调。如果我们执行 `greetDelay` 函数，将按预期显示名称属性。

以下代码片段是由 TypeScript 编译器生成的。当编译箭头函数时，TypeScript 编译器将为 `this` 操作符生成一个名为 `_this` 的别名。该别名用于确保 `this` 操作符指向正确的对象：

```js
Person.prototype.greetDelay = function (time) { 
  var _this = this; 
  setTimeout(function () { 
    alert("Hi! My name is " + _this._name); 
  }, time); 
}; 
```

我们将在 第六章 中深入探讨 `this` 操作符，*理解运行时*。

# 回调地狱

我们已经了解到回调和高级函数是 JavaScript 和 TypeScript 中的两个强大且灵活的功能。然而，回调的使用可能导致一个称为 **回调地狱** 的可维护性问题。

我们现在将编写一个示例来展示回调地狱。我们将编写三个具有相同行为的函数，分别命名为 `doSomethingAsync`、`doSomethingElseAsync` 和 `doSomethingMoreAsync`：

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

function doSomethingElseAsync( 
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

function doSomethingMoreAsync( 
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

之前的功能通过使用 `setTimeout` 函数模拟异步操作。每个函数都接受一个成功回调，如果操作成功则调用该回调，以及一个错误回调，如果发生错误则调用该回调。

在现实世界的应用中，异步操作通常涉及与硬件的一些交互（例如，文件系统、网络等）。这些交互被称为**输入/输出（I/O）**操作。I/O 操作可能因许多不同的原因而失败（例如，当我们尝试与文件系统交互以保存新文件时，硬盘上没有足够的空间）。

前面的函数生成一个随机数，如果数字小于 25 则抛出错误；我们这样做是为了模拟潜在的 I/O 错误。

前面的函数将随机数添加到一个数组中，该数组作为参数传递给每个函数。如果没有发生错误，最终函数（`doSomethingMoreAsync`）的结果应该是一个包含三个随机数的数组。

现在三个函数已经声明，我们可以尝试按顺序调用它们。我们将使用回调来确保`doSomethingMoreAsync`在`doSomethingElseAsync`之后被调用，而`doSomethingElseAsync`在`doSomethingAsync`之后被调用：

```js
doSomethingAsync([], (arr1) => { 
    doSomethingElseAsync(arr1, (arr2) => { 
        doSomethingMoreAsync(arr2, (arr3) => { 
            console.log( 
                ` 
                doSomethingAsync: ${arr3[0]} 
                doSomethingElseAsync: ${arr3[1]} 
                doSomethingMoreAsync: ${arr3[2]} 
                ` 
            ); 
        }, (e) => console.log(e)); 
    }, (e) => console.log(e)); 
}, (e) => console.log(e)); 
```

前面的例子使用了几个嵌套的回调。这类嵌套回调的使用被称为**回调地狱（callback hell）**，因为它们可能导致以下可维护性问题：

+   使代码更难以跟踪和理解

+   使代码更难以维护（重构、重用等）

+   使异常处理更加困难

# 承诺

在看到回调的使用如何导致一些可维护性问题之后，我们现在将学习关于承诺（promises）以及如何使用它们来编写更好的异步代码。承诺背后的核心思想是，承诺代表异步操作的结果。承诺必须处于以下三种状态之一：

+   **挂起（Pending）**：承诺的初始状态。

+   **已实现/已解决（Fulfilled/resolved）**：表示成功操作的承诺状态。术语“已实现”和“已解决”都常用来指代此状态。

+   **已拒绝（Rejected）**：表示失败操作的承诺状态。

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

这里使用`try...catch`语句来展示我们如何显式地实现或拒绝一个承诺。对于`Promise`函数来说，`try...catch`语句是不必要的，因为当在承诺回调中抛出错误时，承诺将自动被拒绝。

前面的代码片段声明了一个名为`foo`的函数，该函数返回一个承诺。承诺包含一个名为`then`的方法，该方法接受一个在承诺实现时将被调用的函数。承诺还提供了一个名为`catch`的方法，该方法在承诺被拒绝时被调用。

如果我们针对 ES5，TypeScript 编译器将不会识别承诺，因为承诺 API 是 ES6 的一部分。我们可以通过在 `tsconfig.json` 文件中使用 `lib` 选项启用 `es2015.promise` 类型来解决此问题。请注意，启用此选项将禁用默认包含的一些类型，因此会破坏一些示例。您可以通过包含 `dom` 和 `es5` 类型，以及使用 `tsconfig.json` 文件中的 `lib` 选项来解决这些问题：

`"lib": [`

`    "es2015.promise",`

`    "dom",`

`    "es5",`

`    "es2015.generator", // new`

`    "es2015.iterable" // new`

`]`

我们将在第九章 自动化您的开发工作流程中了解更多关于 `lib` 设置的信息。

现在，我们将重写我们在回调地狱示例中编写的 `doSomethingAsync`、`doSomethingElseAsync` 和 `doSomethingMoreAsync` 函数，使用承诺而不是回调：

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

function doSomethingElseAsync(arr: number[]) { 
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

function doSomethingMoreAsync(arr: number[]) { 
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

我们可以使用承诺 API 链式调用每个前面函数返回的承诺：

```js
doSomethingAsync([]).then((arr1) => { 
    doSomethingElseAsync(arr1).then((arr2) => { 
        doSomethingMoreAsync(arr2).then((arr3) => { 
            console.log( 
                ` 
                doSomethingAsync: ${arr3[0]} 
                doSomethingElseAsync: ${arr3[1]} 
                doSomethingMoreAsync: ${arr3[2]} 
                ` 
            ); 
        }); 
    }); 
}).catch((e) => console.log(e)); 
```

前面的代码片段比回调示例中使用的代码要好一些，因为我们只需要声明一个异常处理程序而不是三个异常处理程序。这是可能的，因为错误是通过承诺链传播的。

前面的例子介绍了一些改进。然而，承诺 API 允许我们以更简洁的方式链式调用承诺：

```js
doSomethingAsync([]) 
    .then(doSomethingElseAsync) 
    .then(doSomethingMoreAsync) 
    .then((arr3) => { 
        console.log( 
            ` 
            doSomethingAsync: ${arr3[0]} 
            doSomethingElseAsync: ${arr3[1]} 
            doSomethingMoreAsync: ${arr3[2]} 
            ` 
        ); 
    }).catch((e) => console.log(e)); 
```

前面的代码比回调示例中使用的代码更容易阅读和跟踪，但这并不是唯一选择承诺而不是回调的原因。使用承诺还使我们能够更好地控制操作的执行流程。让我们看看几个例子。

承诺 API 包含一个名为 `all` 的方法，它允许我们并行执行一系列承诺，并一次性获取每个承诺的所有结果：

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

承诺 API 还包含一个名为 `race` 的方法，它允许我们并行执行一系列承诺，并获取第一个解决的承诺的结果：

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

+   **并发**：任务并行执行（就像在 `Promise.all` 示例中）

+   **竞赛**：任务并行执行，只返回最快承诺的结果

+   **序列**：一组任务按顺序执行，但前面的任务不会将参数传递给下一个任务

+   **瀑布流**：一组任务按顺序执行，每个任务将参数传递给下一个任务（就像在 `Promise.all` 和 `Promise.race` 示例之前给出的例子）

+   **组合**：这是前面并发、序列和瀑布方法的任何组合

# 回调参数中的协变检查

TypeScript 2.4 改变了类型系统内部的行为方式，以改善嵌套回调和承诺中的错误检测：

TypeScript 对回调参数的检查现在与立即签名检查是协变的。之前它是双变的，有时可能会让错误类型通过。基本上，这意味着回调参数和包含回调的类将更加仔细地检查，因此 TypeScript 在这个版本中将要求更严格的类型。这尤其适用于 Promises 和 Observables，因为它们的 API 指定方式。

在 TypeScript 2.4 之前的版本中，以下示例被认为是有效的，并且没有抛出错误：

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

在 TypeScript 2.4 版本之后的版本中，我们需要添加`nestedCallback`的完整签名来解决这个错误：

```js
someFunc( 
    ( 
        nestedCallback: (e: number, result: any) => void // OK 
    ) => { 
        nestedCallback(1, 1); 
    } 
); 
```

多亏了 TypeScript 类型系统的内部变化，以下错误也被检测到了：

```js
let p: Promise<number> = new Promise((res, rej) => { 
    res("error"); // Error 
}); 
```

在 TypeScript 2.4 之前，前面的 promise 会被推断为`Promise<{}>`，因为我们创建`Promise`类的实例时忘记了添加泛型`<number>`参数。然后`error`字符串会被认为是`{}`的有效实例。

这是一个清晰的例子，说明了为什么建议您定期升级 TypeScript。TypeScript 的每个新版本都引入了新的功能，能够为我们检测到新的错误。

# 生成器

如果我们在 TypeScript 中调用一个函数，我们可以假设一旦函数开始运行，它将始终运行到完成，然后其他代码才能运行。然而，一种新的函数类型最近被添加到 ECMAScript 规范中，这种函数可以在执行过程中暂停——一次或多次——并在稍后恢复，允许在这些暂停期间运行其他代码。这些新类型的函数被称为**生成器**。

生成器代表一系列值。`generator`对象的接口只是一个**迭代器**。迭代器实现了以下接口：

```js
interface Iterator<T> { 
  next(value?: any): IteratorResult<T>; 
  return?(value?: any): IteratorResult<T>; 
  throw?(e?: any): IteratorResult<T>; 
} 
```

可以调用`next()`函数，直到它耗尽值。

我们可以通过使用`function`关键字后跟一个星号（`*`）来定义一个生成器。`yield`关键字用于停止函数的执行并返回一个值。让我们看一个例子：

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

注意，如果您针对 ES5，生成器需要一些额外的类型。您需要将`es2015.generator`和`es2015.iterable`添加到您的`tsconfig.json`文件中：

```js
"lib": [ 
      "es2015.promise", 
      "dom",  
      "es5", 
      "es2015.generator", // new 
      "es2015.iterable" // new 
] 
```

我们将在第九章自动化您的开发工作流程中了解更多关于`lib`设置的内容。

如我们所见，前面的迭代器有五个步骤。第一次调用`next`时，函数将执行直到遇到第一个`yield`语句，然后它将返回值`1`并停止函数的执行，直到我们再次调用生成器的`next`方法。如我们所见，我们现在能够在给定点停止函数的执行。这允许我们编写不会导致栈溢出的无限循环，如下面的例子所示：

```js
function* foo() { 
let i = 1; 
    while (true) { 
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

生成器将打开同步的可能性，因为我们可以在异步事件发生后调用生成器的`next`方法。

# 异步函数 – async 和 await

**异步函数**是 TypeScript 1.6 版本引入的一个特性。开发者可以使用`await`关键字等待函数结果，而不会阻塞程序的正常执行。

使用异步函数有助于提高代码的可读性，与使用 promises 或回调函数相比，但技术上我们可以使用两者都实现相同的功能。

让我们看看一个基本的`async`/`await`示例：

```js
let p = Promise.resolve(3); 

async function fn(): Promise<number> { 
    let i = await p; // 3 
    return 1 + i; // 4 
} 

fn().then((r) => console.log(r)); // 4 
```

上述代码片段声明了一个名为`p`的 promise。这个 promise 是我们将要等待的代码片段的执行。在等待的过程中，程序执行不会被阻塞，因为 JavaScript 允许我们在不阻塞的情况下等待名为`fn`的异步函数。正如我们所见，`fn`函数前面有`async`关键字，它用来告诉编译器这是一个异步函数。

在函数内部，`await`关键字用于挂起执行，直到 promise `p`被实现或拒绝。正如我们所见，语法比使用 promises API 或回调函数时的语法更简洁。

`fn`函数在运行时返回一个 promise，因为它是一个`async`函数。这应该解释了为什么我们需要在代码片段的末尾使用`then`回调来调用它。

以下代码片段展示了我们如何声明一个名为`invokeTaskAsync`的异步函数。这个异步函数使用`await`关键字等待我们在 promise 示例中声明的`doSomethingAsync`、`doSomethingElseAsync`和`doSomethingMoreAsync`函数的结果：

```js
async function invokeTaskAsync() { 
    let arr1 = await doSomethingAsync([]); 
    let arr2 = await doSomethingElseAsync(arr1); 
    return await doSomethingMoreAsync(arr2); 
} 
```

`invokeTaskAsync`函数是异步的，因此它在运行时会返回一个 promise。这意味着我们可以使用 promises API 来等待结果或分别捕获潜在的错误：

```js
invokeTaskAsync().then((result) => { 
    console.log( 
        ` 
        doSomethingAsync: ${result[0]} 
        doSomethingElseAsync: ${result[1]} 
        doSomethingMoreAsync: ${result[2]} 
        ` 
    ); 
}).catch((e) => { 
    console.log(e); 
}); 
```

我们还可以将异步 IIFE 定义为使用`async`和`await`关键字的一种方便方式：

```js
(async () => { 
    try { 
        let arr1 = await doSomethingAsync([]); 
        let arr2 = await doSomethingElseAsync(arr1); 
        let arr3 = await doSomethingMoreAsync(arr2); 
        console.log( 
            ` 
            doSomethingAsync: ${arr3[0]} 
            doSomethingElseAsync: ${arr3[1]} 
            doSomethingMoreAsync: ${arr3[2]} 
            ` 
        ); 
    } catch (e) { 
        console.log(e); 
    } 
})(); 
```

使用异步 IIFE 非常有用，因为通常我们无法在函数之外使用`await`关键字——例如，在应用程序的入口点。我们可以使用异步 IIFE 来克服这个限制：

```js
(async () => { 
    await main(); 
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

然而，我们还没有学习到所有异步迭代器实现的接口：

```js
interface AsyncIterator<T> { 
  next(value?: any): Promise<IteratorResult<T>>; 
  return?(value?: any): Promise<IteratorResult<T>>; 
  throw?(e?: any): Promise<IteratorResult<T>>; 
} 
```

每次我们调用`next`方法时，异步迭代器都会返回一个 promise。以下代码片段演示了异步迭代器如何与异步函数结合使用时非常有用：

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

let i = g1(); 
i.next().then((n) => console.log(n)); // 1 
i.next().then((n) => console.log(n)); // 2 
i.next().then((n) => console.log(n)); // 3 
```

如果我们针对 ES5，异步迭代器需要一些额外的类型。您需要在`tsconfig.json`文件中添加`esnext.asynciterable`。我们还需要在`tsconfig.json`中启用一个额外的设置，以提供对可迭代的全面支持（例如，使用`for...of`控制流语句、扩展运算符或对象解构）当针对 ES3 或 ES5 时：

`"lib": [`

`"es2015.promise",`

`"dom",`

`"es5",`

`"es2015.generator",`

`"es2015.iterable",`

`"esnext.asynciterable" // new`

`] `

我们将在第九章自动化你的开发工作流程中了解更多关于`lib`设置的内容。

# 异步迭代（for await...of）

我们可以使用新的`for...await...of`表达式来迭代并等待异步迭代器返回的每个承诺：

```js
async function func() { 
    for await (const x of g1()) { 
        console.log(x); 
    } 
} 
```

# 将迭代委托给另一个生成器（yield*）

我们可以使用`yield*`表达式将一个生成器委托给另一个生成器。以下代码片段定义了两个名为`g1`和`g2`的生成器函数。`g2`生成器使用`yield*`表达式将迭代委托给由`g1`创建的迭代器：

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

`yield*`表达式也可以用来将迭代委托给可迭代的对象，例如数组：

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

# 摘要

在本章中，我们深入学习了如何使用函数。我们从一个基本概念快速回顾开始，然后转向一些不太为人所知的函数特性和用例。

一旦我们学会了如何使用函数，我们就专注于回调函数、承诺和生成器在 TypeScript 异步编程能力中的应用。

在下一章中，我们将探讨如何使用 TypeScript 编程语言中的类、接口和其他面向对象编程特性。
