# 3. 函数

概述

函数是任何应用程序的基本构建块。本章将教你如何利用 TypeScript 的多功能函数释放其力量，这些功能可能在其他编程语言中找不到。我们将讨论 `this` 关键字，并查看函数表达式、成员函数和箭头函数。本章还讨论了函数参数，包括剩余参数和默认参数。我们还将查看 `import` 和 `export` 关键字。

本章还将教你如何编写通过不同参数组合的测试用例，并将预期的输出与实际输出进行比较。我们将通过设计一个原型应用程序并完成单元测试来结束本章。

# 简介

到目前为止，我们已经学习了 TypeScript 的一些基础知识，如何设置项目以及定义文件的使用。现在我们将深入探讨函数的主题，这些函数将成为你工具箱中最重要的工具。甚至面向对象编程范式也严重依赖于函数作为业务逻辑的基本构建块。

函数，有时也称为例程或方法，是每种高级编程语言的一部分。重用代码段的能力至关重要，但函数提供的作用比这更为重要，因为它们可以接受不同的参数或变量来执行并产生不同的结果。编写好的函数是优秀程序与卓越程序之间的区别。你首先需要从学习语法开始，然后再考虑通过考虑它应该接受什么参数以及它应该产生什么来编写一个好的函数。

在本章中，我们将介绍创建函数的三种不同方法。我们将描述使用 `this` 关键字的陷阱和正确用法。我们将探讨强大的编程技术，包括柯里化、函数式编程和闭包的使用。我们将探讨 TypeScript 模块系统以及如何通过 `import` 和 `export` 关键字在模块之间共享代码。我们将看到函数如何组织到类中，以及如何将 JavaScript 代码重构为 TypeScript。然后我们将学习如何使用流行的 Jest 测试框架。

将这些技能付诸实践，我们将设计、构建和测试一个原型航班预订系统。

# TypeScript 中的函数

函数的简单定义是一组可以被调用的语句；然而，函数的使用和约定并不能如此简单地总结。TypeScript 中的函数比某些其他语言具有更大的实用性。除了正常调用外，函数还可以作为其他函数的参数，并可以从函数中返回。实际上，函数是一种可以调用的特殊对象。这意味着除了参数外，函数实际上还可以有自己的属性和方法，尽管这很少发生。

只有极小的程序才会避免大量使用函数。大多数程序将由许多`.ts`文件组成。这些文件通常导出函数、类或对象。程序的其它部分将通过调用函数与导出的代码进行交互。函数创建了你应用程序逻辑的重用模式，并允许你编写**DRY**（**不要重复自己**）的代码。

在深入研究函数之前，让我们进行一个练习，以了解函数在一般情况下是如何有用的。如果你不理解练习中的一些与函数相关的语法，请不要担心。你将在本章的后面学习所有这些内容。以下练习的目的是帮助你理解函数的重要性。

## 练习 3.01：在 TypeScript 中开始使用函数

为了举例说明函数的有用性，你将创建一个计算平均值的程序。这个练习首先创建一个不使用任何函数的程序。然后，将使用函数执行相同的计算平均值任务。

让我们开始吧：

注意

这个练习的代码文件可以在 https://packt.link/ZHrsh 找到。

1.  打开 VS Code 并创建一个名为 `Exericse01.ts` 的新文件。编写以下不使用除`console.log`语句以外的任何函数的代码：

    ```js
    const values = [8, 42, 99, 161];
    let total = 0;
    for(let i = 0; i < values.length; i++) {
        total += values[i];
    }
    const average = total/values.length;
    console.log(average);
    ```

1.  通过在终端中执行`npx ts-node Exercise 01.ts`来运行文件。你会得到以下输出：

    ```js
    77.5\. 
    ```

1.  现在，使用内置函数和我们的自定义函数`calcAverage`重写相同的代码：

    ```js
    const calcAverage = (values: number[]): number =>     (values.reduce((prev, curr) =>     prev + curr, 0) / values.length);
    const values = [8, 42, 99, 161];
    const average = calcAverage(values);
    console.log(average);
    ```

1.  运行文件并观察输出：

    ```js
    77.5\. 
    ```

    输出相同，但这段代码更简洁、更易于理解。我们编写了自己的函数，同时也使用了内置的`array.reduce`函数。理解函数的工作原理将使我们能够编写自己的有用函数，并利用强大的内置函数。

让我们继续在此基础上进行练习。除了获取平均值之外，考虑一个计算标准差的程序。这可以写成没有函数的过程化代码：

```js
Example01_std_dev.ts
1  const values = [8, 42, 99, 161];
2  let total = 0;
3  for (let i = 0; i < values.length; i++) {
4      total += values[i];
5  }
6  const average = total / values.length;
7  const squareDiffs = [];
8  for (let i = 0; i < values.length; i++) {
9      const diff = values[i] - average;
10    squareDiffs.push(diff * diff)
11 }
12 total = 0;
13 for (let i = 0; i < squareDiffs.length; i++) {
14     total += squareDiffs[i];
15 }
16 const standardDeviation = Math.sqrt(total / squareDiffs.length);
17 console.log(standardDeviation);
Link to the preceding example: https://packt.link/YdTYD
```

运行文件后，你会得到以下输出：

```js
58.148516748065035
```

虽然我们得到了正确的结果，但这段代码非常低效，因为实现细节（在循环中求和数组，然后除以它的长度）被重复了。此外，由于没有使用函数，代码调试起来会很困难，因为程序的不同部分不能单独运行。如果我们得到一个错误的结果，整个程序必须反复运行，进行微小的修正，直到我们确信得到正确的结果。这对于包含数千或数百万行代码的程序来说是不适用的，因为许多主要的网络应用程序就是这样。现在考虑以下程序：

```js
Example02_std_dev.ts
1  const calcAverage = (values: number[]): number =>
2  (values.reduce((prev, curr) => prev + curr, 0) / values.length);
3  const calcStandardDeviation = (values: number[]): number => {
4    const average = calcAverage(values);
5    const squareDiffs = values.map((value: number): number => {
6      const diff = value - average;
7      return diff * diff;
8    });
9    return Math.sqrt(calcAverage(squareDiffs));
10 }
11 const values = [8, 42, 99, 161];
12 console.log(calcStandardDeviation(values));
Link to the preceding example: https://packt.link/smsxT
```

输出如下：

```js
58.148516748065035
```

再次强调，输出是正确的，并且在这个程序中我们两次重用了`calcAverage`函数，证明了编写该函数的价值。即使所有函数和语法现在还不明白，大多数程序员都会同意更简洁、更具表达力的代码比没有重用模式的代码块更可取。

## 关键字函数

创建函数的最简单方法是使用`function`关键字的功能声明。关键字位于函数名之前，之后是一个参数列表，函数体用大括号括起来。函数的参数列表始终用括号括起来，即使没有参数也是如此。在 TypeScript 中，括号始终是必需的，与一些其他语言（如 Ruby）不同：

```js
function myFunction() {
  console.log('Hello world!');
}
```

一个成功完成的函数总是会返回一个或零个值。如果没有返回任何内容，可以使用`void`标识符来表示没有返回任何内容。一个函数不能返回超过一个值，但许多开发者通过返回一个包含多个值的数组或对象来绕过这个限制，这些值可以被重新分配到单独的变量中。函数可以返回 TypeScript 中的任何内置类型或我们编写的类型。函数还可以返回复杂或内联类型（在后面的章节中描述）。如果一个函数可能返回的类型不能通过函数体和`return`语句轻松推断，那么给函数添加一个返回类型是一个好主意。它看起来像这样。`void`的返回类型表示这个函数不返回任何内容：

```js
function myFunction(): void {
  console.log('Hello world!');
}
```

## 函数参数

参数是传递给函数的值的占位符。可以为函数指定任意数量的参数。由于我们正在编写 TypeScript，参数应该有它们的类型注解。让我们改变我们的函数，使其需要一个参数并返回一些内容：

```js
Example03.ts
1 function myFunction(name: string): string {
2   return `Hello ${name}!`;
3 }
```

与前面的例子相比，这个函数期望一个名为`name`的单个参数，其类型已被定义为`string` - `(name: string)`。函数体已更改，现在使用字符串模板将问候消息作为模板字符串返回。我们可以这样调用函数：

```js
4 const message = myFunction('world');
5 console.log(message);
Link to the preceding example: https://packt.link/ITlEU
```

当你运行文件后，你会得到以下输出：

```js
Hello world!
```

这段代码使用参数`'world'`调用`myFunction`，并将函数调用的结果赋值给一个新的常量`message`。`console`对象是一个内置对象，它公开了一个`log`函数（有时称为对象成员的方法）将给定的字符串打印到控制台。由于`myFunction`将给定的参数连接到一个模板字符串，因此`Hello world!`被打印到控制台。

当然，在将函数结果记录之前，没有必要将其存储在常量中。我们可以简单地写下以下代码：

```js
console.log(myFunction('world'));
```

这段代码将调用函数并将结果记录到控制台，如下面的输出所示：

```js
Hello world!
```

本章中的许多示例都将采用这种形式，因为这是一种验证函数输出的简单方法。更复杂的应用程序使用单元测试和更健壮的日志解决方案来验证函数，因此读者应小心不要在应用程序中填充 `console.log` 语句。

## 参数与参数

许多开发者将参数和参数互换使用；然而，参数一词指的是传递给函数的值，而参数指的是函数中的占位符。在 `myFunction('world');` 的例子中，`'world'` 字符串是一个参数，而不是参数。函数声明中具有指定类型的 `name` 占位符是一个参数。

## 可选参数

与 JavaScript 相比，TypeScript 函数参数只有在后缀加上 `?` 时才是可选的。前一个示例中的函数 `myFunction` 期望一个参数。考虑以下情况，我们未指定任何参数：

```js
const message = myFunction();
```

这段代码将导致编译错误：`期望 1 个参数，但得到 0`。这意味着代码甚至无法编译，更不用说运行了。同样，考虑以下代码片段，其中我们提供了一个错误类型的参数：

```js
const message = myFunction(5);
```

现在的错误信息显示：`类型 '5' 无法分配给类型 'string' 的参数`。

这个错误信息给出了我们尝试传递的值的可能最窄的类型。而不是说 `参数类型为 'number'`，编译器将类型视为简单的数字 `5`。这给我们一个提示，即类型可以比原始的 `number` 类型窄得多。

TypeScript 通过强制类型来防止我们犯这样的错误。但如果我们确实想使参数可选怎么办？一个选项是，如前所述，在参数后缀加上 `?`，如下面的代码片段所示：

```js
Example04.ts
1 function myFunction(name?: string): string {
2   return `Hello ${name}!`;
3 }
```

现在我们可以成功调用它：

```js
4 const message = myFunction();
5 console.log(message);
Link to the preceding example: https://packt.link/cnW4c
```

运行此命令将显示以下输出：

```js
Hello undefined!
```

在 TypeScript 中，任何尚未分配的变量都将具有 `undefined` 的值。当函数执行时，`undefined` 值在运行时被转换为 `undefined` 字符串，因此 `Hello undefined!` 被打印到控制台。

## 默认参数

在前面的示例中，`name` 参数已被设置为可选，并且由于它从未获得过值，所以我们打印了 `Hello undefined!`。更好的方法是给 `name` 赋予一个默认值，如下所示：

```js
Example05.ts
1 function myFunction(name: string = 'world'): string {
2   return `Hello ${name}!`;
3 }
Link to the preceding example: https://packt.link/zS5Ej
```

现在，如果我们不提供参数，函数将给出默认值：

```js
4 const message = myFunction();
5 console.log(message);
```

输出如下：

```js
Hello world!
```

如果我们使用以下代码提供参数，它将给出我们传递的值：

```js
const message = myFunction('reader');
console.log(message);
```

这将显示以下输出：

```js
Hello reader!
```

这很简单。现在，让我们尝试处理多个参数。

## 多个参数

函数可以有任意数量或类型的参数。参数列表由逗号分隔。尽管您的编译器设置可以允许您省略参数类型，但启用`noImplicitAny`选项是一种最佳实践。这将导致编译器错误，如果您意外省略了类型。此外，尽可能避免使用广泛的`any`类型，正如在*第一章*，*TypeScript 基础知识与类型概述*中所述。*第六章*，*高级类型*将更深入地探讨高级类型，特别是交集和联合类型，这将帮助我们确保所有变量都有良好、描述性的类型。

## 剩余参数

扩展运算符（`…`）可以用作函数的最后一个参数。这将接受传递给函数的所有参数并将它们放入一个数组中。让我们看看这个是如何工作的：

```js
Example06.ts
1 function readBook(title: string, ...chapters: number[]): void {
2   console.log(`Starting to read ${title}...`);
3   chapters.forEach(chapter => {
4     console.log(`Reading chapter ${chapter}.`);
5   });
6   console.log('Done reading.');
7 }
Link to the preceding example: https://packt.link/Fw2iC
```

现在，函数可以用可变参数列表来调用：

```js
readBook('The TypeScript Workshop', 1, 2, 3);
```

第一个参数是必需的。其余的是可选的。我们完全可以不指定任何章节来阅读。然而，如果我们提供了额外的参数，它们必须是`number`类型，因为这是我们用作类型（`number[]`）的剩余参数的类型。

运行前面的代码后，您将获得以下输出：

```js
Starting to read The TypeScript Book...
Reading chapter 1.
Reading chapter 2.
Reading chapter 3.
Done reading.
```

注意，这个语法特别要求单个参数为`number`类型。我们可以不使用剩余参数来实现该函数，而是期望一个数组作为单个参数：

```js
Example07.ts
1 function readBook(title: string, chapters: number[]): void {
2   console.log(`Starting to read ${title}...`);
3   chapters.forEach(chapter => {
4     console.log(`Reading chapter ${chapter}.`);
5   });
6   console.log('Done reading.');
7 }
Link to the preceding example: https://packt.link/AvInF
```

函数现在将精确需要两个参数：

```js
readBook('The TypeScript Book', [1, 2, 3]);
```

输出如下：

```js
Starting to read The TypeScript Book...
Reading chapter 1.
Reading chapter 2.
Reading chapter 3.
Done reading.
```

哪个更好？这需要你自己决定。在这种情况下，我们想要阅读的章节已经以数组形式存在，然后将其传递给函数可能最有意义。

注意，`readBook`函数中包含一个箭头函数。我们将在下一节中介绍箭头函数。

## 解构返回类型

有时，一个函数返回多个值可能是有用的。采用函数式编程范式的程序员通常希望有一个返回元组或包含不同类型两个元素的数组函数。回到我们之前的例子，如果我们想计算数字数组的平均值和标准差，可能有一个同时处理这两个操作的单一函数会更方便，而不是需要多次调用同一个数字数组。

TypeScript 中的函数只能返回一个值。然而，我们可以通过解构来模拟返回多个参数。解构是将对象或数组的一部分分配给不同变量的实践。这允许我们将返回值的各个部分分配给变量，给人一种返回多个值的感觉。让我们看一个例子：

```js
Example08.ts
1 function paritySort(...numbers: number[]): { evens: number[], odds: 2 number[] } {
3   return {
4     evens: numbers.filter(n => n % 2 === 0),
5     odds: numbers.filter(n => n % 2 === 1)
6   };
7 }
Link to the preceding example: https://packt.link/SHkuW
```

此代码使用内置数组对象的 `filter` 方法遍历数组中的每个值并对其进行测试。如果测试返回 `true` 布尔值，则该值将被推入一个新数组，该数组将被返回。使用模运算符来测试余数将我们的数字数组过滤成两个单独的数组。然后函数将返回这些数组作为对象属性。我们可以利用这一点进行解构。考虑以下代码：

```js
const { evens, odds } = paritySort(1, 2, 3, 4);
console.log(evens); 
console.log(odds);
```

在这里，我们给函数传递参数 `1, 2, 3, 4`，它返回以下输出：

```js
[2, 4]
[1, 3]
```

## 函数构造函数

注意，TypeScript 语言包含一个大写的 `Function` 关键字。这不同于小写的 `function` 关键字，不应使用，因为它被认为不安全，因为它能够解析和执行任意代码字符串。`Function` 关键字仅在 TypeScript 中存在，因为 TypeScript 是 JavaScript 的超集。

## 练习 3.02：比较数字数组

TypeScript 比较运算符如 `===` 或 `>` 仅适用于原始类型。如果我们想比较更复杂的数据类型，如数组，我们需要使用库或实现自己的比较。让我们编写一个函数，可以比较一对未排序的数字数组，并告诉我们值是否相等。

注意

本练习的代码文件可在 [`packt.link/A0IxN`](https://packt.link/A0IxN) 找到。

1.  在 VS Code 中创建一个新文件，并将其命名为 `array-equal.ts`。

1.  从以下代码开始，该代码声明了三个不同的数组并输出，无论它们是否相等：

    ```js
    const arrayone = [7, 6, 8, 9, 2, 25];
    const arraytwo = [6, 8, 9, 2, 25];
    const arraythree = [6, 8, 9, 2, 25, 7];
    function arrayCompare(a1: number[], a2: number[]): boolean {
      return true;
    }
    console.log(
      `Are ${arrayone} and ${arraytwo} equal?`,
      arrayCompare(arrayone, arraytwo)
    );
    console.log(
      `Are ${arrayone} and ${arraythree} equal?`,
      arrayCompare(arrayone, arraythree)
    );
    console.log(
      `Are ${arraytwo} and ${arraythree} equal?`,
      arrayCompare(arraytwo, arraythree)
    );
    ```

    由于函数尚未实现，它仅返回 `true`，因此所有三个比较的输出都将为 true。

    我们的功能 `arrayCompare` 接受两个数组作为参数，并返回一个布尔值，表示它们是否相等。我们的业务规则是，如果所有值在排序后匹配，则未排序的数组将被视为相等。

1.  使用以下代码更新 `arrayCompare`：

    ```js
    function arrayCompare(a1: number[], a2: number[]): boolean {
      if(a1.length !== a2.length) {
        return false;
      }
      return true;
    }
    ```

    在前面的代码中，我们正在测试传入的两个数组是否相等。我们首先应该检查数组长度是否相等。如果它们长度不相等，那么值就不可能相等，因此我们将从函数中返回 `false`。如果在执行过程中遇到返回语句，则函数的其余部分将不会执行。

    在这一点上，该函数将只能告诉我们数组是否长度相等。为了完成挑战，我们需要比较数组中的每个值。在尝试比较之前对值进行排序将使这项任务容易得多。幸运的是，数组对象原型包括一个 `sort()` 方法，这将为我们处理这一切。使用内置函数可以节省大量开发时间。

1.  实现排序 `sort()` 方法以排序数组值：

    ```js
    function arrayCompare(a1: number[], a2: number[]): boolean {
      if(a1.length !== a2.length) {
        return false;
      }
      a1.sort();
      a2.sort();
      return true;
    }
    ```

    `sort()` 方法就地排序数组元素，因此没有必要将结果分配给新变量。

    最后，我们需要遍历其中一个数组，比较相同索引处的每个元素。我们使用 `for` 循环遍历第一个数组，并将每个索引处的值与第二个数组中相同索引处的值进行比较。由于我们的数组使用原始值，`!==` 比较运算符将起作用。

1.  使用以下 `for` 循环遍历数组：

    ```js
    function arrayCompare(a1: number[], a2: number[]): boolean {
      if(a1.length !== a2.length) {
        return false;
      }
      a1.sort();
      a2.sort();
      for (let i = 0; i < a1.length; i++) {
        if (a1[i] !== a2[i]) {
          return false;
        }
      }
      return true;
    }
    ```

    再次，如果任何比较失败，我们将返回 `false` 并退出函数。

1.  使用 `ts-node` 执行程序：

    ```js
    npx ts-node array-equal.ts
    ```

    程序将产生以下输出：

    ```js
    Are 7,6,8,9,2,25 and 6,8,9,2,25,8 equal? false
    Are 2,25,6,7,8,9 and 6,8,9,2,25,7 equal? true
    Are 2,25,6,8,8,9 and 2,25,6,7,8,9 equal? False
    ```

1.  尝试不同的数组组合，并验证程序是否正确运行。

一个好的函数接受一个参数列表并返回一个单一值。你现在既有编写函数的经验，也有利用内置函数解决问题的经验。

# 函数表达式

函数表达式与函数声明不同之处在于它们可以被分配给变量、内联使用或立即调用——立即调用的函数表达式或 IIFE。函数表达式可以是命名的或匿名的。让我们看看几个例子：

```js
Example09.ts
1 const myFunction = function(name: string): string {
2   return `Hello ${name}!`;
3 };
4 console.log(myFunction('function expression'));
Link to the preceding example: https://packt.link/2JeGQ
```

你将得到以下输出：

```js
Hello function expression!
```

这看起来与之前我们看过的例子非常相似，并且几乎以相同的方式工作。以下是用于比较的函数声明：

```js
function myFunction(name: string = 'world'): string {
  return `Hello ${name}!`;
}
```

唯一的一点不同是函数声明是 *提升的*，这意味着它们被加载到内存中（连同任何声明的变量一起），因此可以在代码中声明之前使用。通常认为依赖提升是一种不好的做法，因此现在许多代码检查工具不允许这样做。过度使用提升的程序可能存在难以追踪的 bug，甚至可能在不同的系统中表现出不同的行为。函数表达式之所以受欢迎，其中一个原因就是它们不允许提升，因此避免了这些问题。

函数表达式可以用来创建匿名函数，即没有名字的函数。这与函数声明不同。匿名函数通常用作原生函数的回调。例如，考虑以下使用 `Array.filter` 函数的代码片段：

```js
Example10.ts
1 const numbers = [1, 3, 2];
2 const filtered = numbers.filter(function(val) {return val < 3});
3 console.log(filtered);
Link to the preceding example: https://packt.link/aJyhj
```

输出如下：

```js
[1, 2]
```

记住，在 TypeScript（以及 JavaScript）中，函数可以作为参数传递给其他函数，或者从其他函数返回。这意味着我们可以将匿名函数 `function(val) { return val < 3 }` 作为 `Array.filter` 函数的参数。这个函数没有名字，不能被其他代码引用或调用。这在大多数情况下是可以的。如果我们想，我们可以给它起一个名字：

```js
const filtered = numbers.filter(function myFilterFunc(val) {return val < 3});
```

在大多数情况下，这样做没有太大意义，但如果函数需要自引用，例如递归函数，这可能会很有用。

注意

更多关于回调的信息，请参阅 *第十一章*，*TypeScript 中的高阶函数和回调*。

立即调用的函数表达式看起来像这样：

```js
Example11.ts
1 (function () {
2    console.log('Immediately invoked!');
3 })();
Link to the preceding example: https://packt.link/iQoSX
```

函数输出以下内容：

```js
 "Immediately invoked!"
```

函数是内联声明的，然后末尾的额外`()`括号调用函数。TypeScript 中 IIFE 的主要用例涉及另一个称为闭包的概念，这将在本章后面讨论。现在，只需学会识别这种立即声明和调用的函数语法即可。

# 箭头函数

箭头函数提供了一种更紧凑的语法，同时也为围绕`this`关键字令人困惑且不一致的规则提供了一个替代方案。让我们首先看看语法。

箭头函数移除了`function`关键字，并在参数列表和函数体之间放置了一个“胖箭头”或`=>`。箭头函数从不命名。让我们重写一个打印`Hello`的函数：

```js
const myFunction = (name: string): string => {
  return `Hello ${name}!`;
};
```

这个函数可以变得更加紧凑。如果函数只是返回一个值，则可以省略大括号和`return`关键字。我们的函数现在看起来是这样的。

```js
const myFunction = (name: string): string => `Hello ${name}!`;
```

箭头函数在回调函数中非常常用。前一个过滤器函数的回调可以使用箭头函数重写。再次强调，回调将在第十一章“TypeScript 中的高阶函数和回调”中更详细地讨论。下面是箭头函数的另一个示例：

```js
Example12.ts
1 const numbers = [1, 3, 2];
2 const filtered = numbers.filter((val) => val < 3);
2 console.log(filtered);
Link to the preceding example: https://packt.link/lUTCm
```

输出如下：

```js
[1, 2]
```

这种简洁的语法一开始可能看起来有些令人困惑，所以让我们来分解一下。`filter`函数是 TypeScript 中数组对象的一个内置方法。它将返回一个新数组，包含所有在回调函数中符合标准的数组项。因此，我们说对于每个`val`，如果`val`小于`3`，就将其添加到新数组中。

箭头函数不仅仅是语法上的不同。虽然函数声明和函数表达式创建一个新的执行作用域，但箭头函数不会。这在使用`this`（见下文）和`new`（见第四章，“类和对象”）关键字时具有影响。

# 类型推断

让我们考虑以下代码：

```js
const myFunction = (name: string): string => `Hello ${name}!`;
const numbers = [1, 3, 2];
const filtered = numbers.filter((val) => val < 3);
console.log(filtered);
```

输出如下：

```js
[1, 2]
```

注意，在上述代码中，我们没有为`numbers`常量指定类型。但是等等，这不是一本关于 TypeScript 的书吗？是的，现在我们来到了 TypeScript 的一个最佳特性：类型推断。TypeScript 有在省略类型的情况下为变量分配类型的能力。当我们声明`const numbers = [1, 2, 3];`时，TypeScript 会直观地理解我们正在声明一个数字数组。如果我们想的话，我们可以写`const numbers: number[] = [1, 2, 3];`，但 TypeScript 会认为这些声明是相等的。

上述代码是 100%有效的 ES6 JavaScript。这很好，因为任何 JavaScript 开发者都能阅读和理解它，即使他们没有 TypeScript 的经验。然而，与 JavaScript 不同，TypeScript 会通过将错误类型的值放入`numbers`数组来防止你出错。

因为 TypeScript 推断出了我们的`numbers`数组类型，所以我们无法向其中添加除了数字以外的值；例如，`numbers.push('hello');`将导致编译器错误。如果我们想声明一个允许其他类型的数组，我们需要明确声明这一点——`const numbers: (number | string)[] = [1, 3, 2];`。现在，我们可以稍后向这个数组赋值一个字符串。或者，声明为`const numbers = [1, 2, 3, 'abc'];`的数组已经具有这种类型。

回到我们的`filter`函数，这个函数也没有指定参数或返回类型。为什么允许这样做？又是我们的朋友，类型推断。因为我们正在遍历一个数字数组，该数组中的每个项目都必须是数字。因此，`val`将始终是数字，无需指定类型。同样，表达式`val < 3`是一个布尔表达式，所以返回类型将始终是布尔类型。记住，可选意味着你总是可以选择提供所需类型，如果你认为这可以提高代码的清晰度或可读性，那么你绝对应该这样做。

当一个箭头函数只有一个参数且类型可以推断时，我们可以通过省略参数列表周围的括号来使我们的代码稍微简洁一些。最后，我们的`filter`函数可能看起来像这样：

```js
Example13.ts
1 const numbers = [1, 3, 2];
2 const filtered = numbers.filter(val => val < 3);
3 console.log(filtered);
Link to the preceding example: https://packt.link/hvbsc
```

输出结果如下：

```js
[1, 2]
```

你选择的语法完全是个人品味的问题，但许多经验丰富的程序员倾向于更简洁的语法，因此至少能够阅读和理解它。

## 练习 3.03：编写箭头函数

现在，让我们编写一些箭头函数，熟悉这种语法，并开始构建我们的实用库。实用库的一个好候选函数是可能被调用的函数。在这个练习中，我们将编写一个函数，它接受一个主语、谓语和对象列表，并返回一个语法正确的句子。

注意

这个练习的代码文件可以在[`packt.link/yIQnz`](https://packt.link/yIQnz)找到。

1.  在 VS Code 中创建一个新文件，并将其保存为`arrow-cat.ts`。

1.  从我们需要实现的函数的模式开始，以及一些对其的调用：

    ```js
    export const sentence = (
      subject: string,
      verb: string,
      ...objects: string[]
    ): string => {
      return 'Meow, implement me!';
    };
    console.log(sentence('the cat', 'ate', 'apples', 'cheese', 'pancakes'));
    console.log(sentence('the cat', 'slept', 'all day'));
    console.log(sentence('the cat', 'sneezed'));
    ```

    我们的`sentence`函数显然没有做我们需要的。我们可以修改实现以使用模板字符串来输出主语、谓语和宾语。

1.  使用以下代码实现一个模板字符串来输出主语、谓语和宾语：

    ```js
    export const sentence = (
      subject: string,
      verb: string,
      ...objects: string[]
    ): string => {
      return `${subject} ${verb} ${objects}.`;
    };
    ```

    现在，当我们执行我们的程序时，我们得到以下输出：

    ```js
    the cat ate apples,cheese,pancakes.
    the cat slept all day.
    the cat sneezed .
    ```

    这看起来是可读的，但我们有关于大小写和单词间距的几个问题。我们可以添加一些额外的函数来帮助解决这些问题。思考这些情况应该逻辑上发生什么，如果有多个对象，我们希望在它们之间添加逗号，并在最后一个对象之前使用“and”。如果只有一个对象，则不应有逗号或“and”，如果没有对象，则不应有空格，就像这里一样。

1.  实现一个新的函数，将此逻辑添加到我们的程序中：

    ```js
    export const arrayToObjectSegment = (words: string[]): string => {
      if (words.length < 1) {
        return '';
      }
      if (words.length === 1) {
        return ` ${words[0]}`;
      }
      ...
    };
    ```

    在这里，我们实现了较简单的情况。如果没有对象，我们希望返回一个空字符串。如果只有一个对象，我们返回该对象，前面加一个空格。现在，让我们来处理多个对象的情况。

    我们需要将对象添加到一个以逗号分隔的列表中，如果已经到达最后一个对象，则用“and”连接。

1.  要做到这一点，我们将初始化一个空字符串，并遍历对象的数组：

    ```js
    export const arrayToObjectSegment = (words: string[]): string => {
      if (words.length < 1) {
        return '';
      }
      if (words.length === 1) {
        return ` ${words[0]}`;
      }
      let segment = '';
      for (let i = 0; i < words.length; i++) {
        if (i === words.length - 1) {
          segment += ` and ${words[i]}`;
        } else {
          segment += ` ${words[i]},`;
        }
      }
      return segment;
    };
    ```

    通过将问题分解成小的组件，我们提出了一个解决所有用例的函数。现在，我们的`sentence`函数的`return`语句可以写成`return `${subject} ${verb}${arrayToObjectSegment(objects)}.`;`。

    注意，返回字符串的函数可以完美地嵌入我们的字符串模板中。运行这个程序，我们得到以下输出：

    ```js
    the cat ate apples, cheese, and pancakes.
    the cat slept all day.
    the cat sneezed.
    ```

    这几乎是对的，但句子的第一个字母应该大写。

1.  使用另一个函数来处理大小写，并用它来包裹整个字符串模板：

    ```js
    export const capitalize = (sentence: string): string => {
      return `${sentence.charAt(0).toUpperCase()}${sentence
        .slice(1)
        .toLowerCase()}`;
    };
    ```

1.  这个函数使用了几个内置函数：`charAt`、`toUpperCase`、`slice`和`toLowerCase`，所有这些都在字符串模板内部。这些函数从我们的句子中获取第一个字符，将其转换为大写，然后将其与句子的其余部分连接起来，所有内容都转换为小写。

    现在，当我们执行程序时，我们得到期望的结果：

    ```js
    The cat ate apples, cheese, and pancakes.
    The cat slept all day.
    The cat sneezed.
    ```

为了完成这个练习，我们编写了三个不同的函数，每个函数都服务于单一目的。我们本可以将所有功能都塞进一个函数中，但那样会使生成的代码的可重用性降低，并且更难以阅读和测试。从简单的单一目的函数构建软件仍然是编写干净、可维护代码的最好方法之一。

## 理解这一点

许多开发者都曾因`this`关键字而感到沮丧。`this`名义上指向当前函数的运行时。例如，如果调用一个对象的成员函数，`this`通常指向该对象。在其他上下文中使用`this`可能看起来不一致，其使用可能导致许多异常错误。部分问题在于，在 C++或 Java 等语言中，关键字的使用相对简单，那些有经验的程序员可能会期望 TypeScript 中的`this`有类似的行为。

让我们看看`this`的一个非常简单的用例：

```js
const person = {
    name: 'Ahmed',
    sayHello: function () {
        return `Hello, ${this.name}!`
    }
}
console.log(person.sayHello());
```

在这里，我们声明了一个具有属性`name`和方法`sayHello`的对象。为了让`sayHello`读取`name`属性，我们使用`this`来引用对象本身。这段代码没有问题，许多程序员会认为它非常直观。

当我们需要声明另一个内联函数时，问题就会出现，这可能是作为之前查看的`filter`函数等回调函数的一部分。

让我们设想我们想要将`arrayFilter`函数封装在一个对象中，该对象可以有一个属性来指定允许的最大数量。这个对象将与之前的对象有些相似，我们可能会期望能够使用`this`来获取那个最大值。让我们看看当我们尝试时会发生什么：

```js
const arrayFilter = {
    max: 3,
    filter: function (...numbers: number[]) {
        return numbers.filter(function (val) {
            return val <= this.max;
        });
    }
}
console.log(arrayFilter.filter(1, 2, 3, 4));
```

TypeScript 不喜欢我的代码。根据我的编辑器，`this`下面会出现一条红色的波浪线，我将无法执行我的程序。即使程序执行了，你也不会得到预期的输出。

这里的问题是，我使用`function`关键字创建了一个新的作用域，`this`不再具有我想要的值。事实上，它没有任何值。它是`undefined`。

原因在于，与 C++和 Java 这样的面向对象语言不同，`this`的值将在运行时确定，并将其设置为调用范围。在这种情况下，我们的回调函数不属于任何设置上下文或对象，因此`this`是`undefined`。实际上，`undefined`在这里并不重要。重要的是它不是我们想要的。

这些年来，已经出现了许多解决这个问题的方法。其中之一是我们将`this`引用缓存到另一个变量中，并在我们的回调函数中使该变量可用。另一种方法是使用`Function`原型的`bind`成员函数来设置`this`引用。你可能会遇到类似这样的代码。

一个更好的解决方案是简单地使用箭头函数而不是函数表达式。不仅语法更简洁、更现代，而且箭头函数不会创建新的`this`上下文。你将得到你想要的`this`引用，即顶层对象的引用。让我们使用箭头函数重写代码：

```js
Example14.ts
1 const arrayFilter = {
2     max: 3,
3     filter: function(...numbers: number[]) {
4         return numbers.filter(val => {
5             return val <= this.max;
6         });
7     }
8 }
9 console.log(arrayFilter.filter(1, 2, 3, 4));
Link to the preceding example: https://packt.link/90JSJ
```

该函数产生以下输出：

```js
[1, 2, 3]
```

TypeScript 不再对`this`提出抱怨，代码运行正确。

但是等等，为什么我们为`filter`函数使用函数表达式，而为回调使用箭头函数？这是因为我们实际上需要`function`的作用域创建能力，以便`this`具有值。如果我们把`filter`函数重写为箭头函数，`this`将永远不会设置，我们就无法访问`max`属性。

这确实令人困惑，这也是为什么在 TypeScript 和 JavaScript 中，`this`比其他语言更让人讨厌的原因。重要的是要记住，当你使用`this`编程时，你希望任何对象或类方法都是函数表达式，任何回调都是箭头函数。这样，你将始终拥有正确的`this`实例。

*第四章*，*类和对象*，将更深入地探讨类并探索其他模式。现在让我们在下面的练习中使用对象中的`this`。

## 练习 3.04：在对象中使用 this

对于这个练习，我们将想象我们必须实现一些会计软件。在这个软件中，每个账户对象将跟踪应付款总额，以及已付款金额，并将有几个实用方法来获取账户的当前状态和需要支付的余额。

让我们先创建一个对象，其方法尚未实现。这个例子将演示一个简化的工作流程，其中我们打印出账户，尝试支付超过应付款额（收到错误），然后支付应付款额，最后支付全额应付款额：

注意

这个练习的代码文件可以在[`packt.link/P6YIf`](https://packt.link/P6YIf)找到。

1.  编写以下代码，这是我们程序的起点：

    ```js
    export const account = {
      due: 1000,
      paid: 0,
      status: 'OPEN',
      payAccount: function (amount: number): string {
        return 'unimplemented!';
      },
      printStatus: function (): string {
        return 'unimplemented!';
      },
    };
    console.log(account.printStatus());
    console.log(account.payAccount(1500));
    console.log(account.payAccount(500));
    console.log(account.payAccount(500));
    ```

    我们需要实现两种方法。`printStatus`方法将只输出应付款总额、已付款金额以及账户是开放还是关闭（或已全额支付）。

1.  使用字符串模板来输出状态，但为了访问`account`对象上的属性，请使用`this`关键字：

    ```js
      printStatus: function (): string {
        return `$${this.paid} has been paid and $${
          this.due - this.paid
        } is outstanding. This account is ${this.status}.`;
      },
    ```

    我们将`printStatus`函数表达式实现为一个使用`this`来访问同一对象属性的字符串模板。作为提醒，我们必须在这里使用函数表达式，而不能使用箭头函数，即使我们可能更喜欢那种语法，因为箭头函数不会创建一个新的执行上下文。

    如果有任何疑问，这里没有双美元符号运算符。第一个是表示货币的文本，第二个是模板字符串的一部分。

    现在让我们处理付款。我们的要求是，如果支付的金额超过应付款额，我们应该抛出一个错误，并且不应用任何付款。否则，我们跟踪额外的付款。如果余额达到$0，则关闭账户。我们应在每次交易后打印当前状态。

1.  编写处理付款的代码：

    ```js
      payAccount: function (amount: number): string {
        if (amount > this.due - this.paid) {
          return `$${amount} is more than the outstanding balance of $${
            this.due - this.paid
          }.`;
        }
        this.paid += amount;
        if (this.paid === this.due) {
          this.status = 'CLOSED';
        }
        return this.printStatus();
      },
    ```

1.  执行程序并检查输出：

    ```js
    $0 has been paid and $1000 is outstanding. This account is OPEN.
    $1500 is more than the outstanding balance of $1000.
    $500 has been paid and $500 is outstanding. This account is OPEN.
    $1000 has been paid and $0 is outstanding. This account is CLOSED
    ```

在这个练习中，我们使用了函数表达式作为对象方法来访问对象上的属性。方法不仅可以读取对象上的属性，还可以更新它们。在面向对象编程中，有一个常见的模式是对象既包含数据，又具有访问和修改这些数据的方法。有时，这些方法会被设置为私有，并且只能通过`get`和`set`等访问器来访问。关于这个主题的更多内容将在*第四章*，*类和对象*中介绍。

正如我们在这次练习中看到的，在实现面向对象模式时，了解和掌握函数表达式仍然很重要。

## 闭包和作用域

除了我们之前讨论的所有内容之外，在 TypeScript 中函数还有一些特殊的行为。当一个函数被声明时（无论是函数声明、表达式还是箭头函数），它会封闭任何更高作用域中的变量。这被称为闭包。任何函数都可以是闭包。闭包简单地是一个封闭了变量的函数。

范围的概念简单地说就是每个函数创建一个新的作用域。正如我们所看到的，函数可以在其他函数内部声明。`内部`函数可以读取在`外部`函数中声明的任何变量，但`外部`函数无法看到在`内部`函数中声明的变量。这就是作用域。以下代码通过在`外部`函数内部声明第二个函数来建立外部作用域和内部作用域。`内部`函数能够访问`外部`作用域中的变量，但`内部`作用域中声明的`world`变量在该函数外部不可见：

```js
Example15.ts
1 const outer = (): void => {
2     const hello = 'Hello';
3     const inner = (): void => {
4         const world = 'world!';
5         console.log(`${hello} ${world}`);
6     }
7     inner();
8 
9     console.log(`${hello} ${world}`);
10 }
11 outer();
Link to the preceding example: https://packt.link/USZ74
```

该函数产生以下输出：

```js
Hello world!
ReferenceError: world is not defined
```

当这个函数被调用时，会到达内部日志语句并记录`"Hello world!"`，然后到达外部日志语句，我们得到`ReferenceError`。我们可以通过在`外部`函数中添加`let world;`来修复`ReferenceError`：

```js
Example16.ts
1  const outer = (): void => {
2      const hello = 'Hello';
3      let world;
4      const inner = (): void => {
5          const world = 'world!';
6          console.log(`${hello} ${world}`);
7      }
8      inner();
8 
9      console.log(`${hello} ${world}`);
10  }
11  outer();
Link to the preceding example: https://packt.link/yC0Zq
```

该函数产生以下输出：

```js
Hello world!
Hello undefined!
```

这是因为`内部`函数声明了一个新的`world`变量，而`外部`函数无法访问它。我们可以从`内部`声明中删除`const`：

```js
Example17.ts
1  const outer = (): void => {
2      const hello = 'Hello';
3      let world;
4      const inner = (): void => {
5          world = 'world!';
6          console.log(`${hello} ${world}`);
7      }
8      inner();
9  
10     console.log(`${hello} ${world}`);
11 }
12 
13 outer();
Link to the preceding example: https://packt.link/fCsaY
```

该函数产生以下输出：

```js
Hello world!
Hello world!
```

函数最终可以工作，因为`内部`函数是在`外部`函数的作用域中声明的变量上操作的。在退出内部作用域后，它仍然可见，因此可以打印出来。

让我们看看一个更有用的例子。斐波那契数列是一个数集，其中下一个数是前两个数的和：`[0, 1, 1, 2, 3, 5, 8, 13, 21, …]`。斐波那契数列通常用于帮助解释递归函数。在这种情况下，我们将用它来通过编写一个每次调用都返回序列中下一个值的函数来演示闭包。

我们程序的逻辑将是，我们将跟踪函数返回的当前数字，下一个应该是什么数字，以及增加数字的数量。每次调用时，这三个数字都将更新以准备下一次调用。实现这一点的一种方法是将这些值定义为全局作用域变量，并编写一个简单的函数来更新和跟踪它们。这可能看起来像这样：

```js
Example_Fibbonacci_1.ts
1 let next = 0;
2 let inc = 1;
3 let current = 0; 
4 
5 for (let i = 0; i < 10; i++) {
6     [current, next, inc] = [next, inc, next + inc];
7     console.log(current);
8 }
Link to the preceding example: https://packt.link/17Hda
```

该函数产生以下输出：

```js
0
1
1
2
3
5
8
13
21
34
```

这个程序可以运行并返回期望的结果，但由于它不是一个函数，程序将只执行一次然后停止。如果你想在其他过程中获取下一个斐波那契数，你将无法做到。如果你只是将它包裹在一个函数中，这也不会起作用：

```js
Example_Fibbonacci_2.ts
1  const getNext = (): number => {
2      let next = 0;
3      let inc = 1;
4      let current = 0;
5      [current, next, inc] = [next, inc, next + inc];
6      return current;
7  };
8  
9  for (let i = 0; i < 10; i++) {
10     console.log(getNext());
11 }
Link to the preceding example: https://packt.link/rfDuz
```

该函数产生以下输出：

```js
0
0
//...
```

这个函数每次被调用时都会返回`0`，因为所有变量在调用时都会被重新声明。我们可以通过将变量移出函数来修复这个问题。这样，它们只声明一次，并由被调用的函数修改。

我们的功能现在设置了下一个要返回的值，增量值以及最近返回的值。在循环中的每次函数调用，它将当前值替换为下一个值，下一个值替换为增量值，增量值替换为下一个值加上前一个增量值的总和。然后它记录当前值：

```js
Example_Fibbonacci_3.ts
1  let next = 0;
2  let inc = 1;
3  let current = 0;
4  
5  const getNext = (): number => {
6      [current, next, inc] = [next, inc, next + inc];
7      return current;
8  };
9  
10 for (let i = 0; i < 10; i++) {
11     console.log(getNext());
12 }
Link to the preceding example: https://packt.link/mAEds
```

函数产生了以下输出：

```js
0
1
1
2
3
5
8
13
21
34
```

这成功了！它之所以能成功，是因为`getNext`函数能够访问更高作用域中的变量。这个函数是一个闭包。这可能会显得很标准且预期，但可能意想不到的是，即使函数被导出并由程序的其他部分调用，这也能工作。这可以通过创建另一个函数来更好地说明：

```js
Example_Fibbonacci_4.ts
1  const fibonacci = () => {
2      let next = 0;
3      let inc = 1;
4      let current = 0;
5      return () => {
6          [current, next, inc] = [next, inc, next + inc];
7          return current;
8      };
9  };
10 const getNext = fibonacci();
11 for (let i = 0; i < 10; i++) {
12     console.log(getNext());
13 }
Link to the preceding example: https://packt.link/CdKte
```

输出没有变化：

```js
0
1
1
2
3
//...
```

调用`fibonacci`函数将返回一个新的函数，该函数可以访问`fibonacci`中声明的变量。如果我们想运行另一个斐波那契数列，我们可以再次调用`fibonacci()`以获得一个新的作用域和初始化的变量：

```js
Example_Fibbonacci_5.ts
1  const fibonacci = () => {
2     let next = 0;
3     let inc = 1;
4      let current = 0;
5      return () => {
6          [current, next, inc] = [next, inc, next + inc];
7          return current;
8      };
9  };
10 const getNext = fibonacci();
11 const getMoreFib = fibonacci();
12 for (let i = 0; i < 10; i++) {
13     console.log(getNext());
14 }
15 for (let i = 0; i < 10; i++) {
16     console.log(getMoreFib());
17 }
Link to the preceding example: https://packt.link/0nGph
```

我们将再次看到相同的输出，但这次是两次：

```js
0
1
1
2
//…
21
34
0
1
1
2
//…
```

注意

为了便于展示，这里只显示实际输出的部分。

在这两种情况下，闭包都覆盖了更高作用域中的变量，并且在函数调用中仍然可用。这是一个强大的技术，正如所展示的，但如果使用不当可能会导致内存泄漏。在这种闭包中声明的变量，在存在对其的引用时无法被垃圾回收。

## 练习 3.05：使用闭包创建订单工厂

闭包可能难以处理，但一个真正体现其有用性的常见模式有时被称为工厂模式。简单来说，这是一个返回另一个已经设置好并准备好使用的函数的函数。在这个模式中，闭包被用来确保变量可以在函数调用之间持久化。我们将在本练习中探讨这个模式。

让我们从几乎实现我们想要的功能的代码开始。我们正在为某种服装的订单系统工作。每个订单将指定相同颜色和大小的服装的数量。我们只需要为每件服装生成一个具有唯一 ID 的记录以进行跟踪：

注意

本练习的代码文件可以在[`packt.link/fsqdd`](https://packt.link/fsqdd)找到。

1.  在 VS Code 中创建一个新文件，并将其保存为`order.ts`。从以下代码和一些示例调用开始：

    ```js
    interface Order {
      id: number;
      color: string;
      size: string;
    }
    export const createOrder = (
      color: string,
      size: string,
      quantity: number
    ): Order[] => {
      let id = 0;
      const orders = [];
      for (let i = 0; i < quantity; i++) {
        orders.push({ id: id++, color, size });
      }
      return orders;
    };
    const orderOne = createOrder('red', 'M', 4);
    console.log(orderOne);
    const orderTwo = createOrder('blue', 'S', 7);
    console.log(orderTwo);
    ```

    代码看起来没问题。让我们运行它看看效果如何。你会得到以下输出：

    ```js
    [
      { id: 0, color: 'red', size: 'M' },
      { id: 1, color: 'red', size: 'M' },
      { id: 2, color: 'red', size: 'M' },
      { id: 3, color: 'red', size: 'M' }
    ]
    [
      { id: 0, color: 'blue', size: 'S' },
      { id: 1, color: 'blue', size: 'S' },
      { id: 2, color: 'blue', size: 'S' },
      { id: 3, color: 'blue', size: 'S' },
      { id: 4, color: 'blue', size: 'S' },
      { id: 5, color: 'blue', size: 'S' },
      { id: 6, color: 'blue', size: 'S' }
    ]
    ```

    这是不正确的。我们不能每次都从零开始重新开始 ID 号码。我们该如何解决这个问题？

    有几种方法可以解决这个问题。最简单的方法是在 `orderFactory` 之外声明 ID 号码。然而，这样做可能会随着系统复杂性的增长而导致错误。位于最高层或甚至全局作用域中的变量可以由系统的任何部分访问，并且可能被某些边缘情况修改。

1.  使用闭包来解决这个问题。创建一个 `orderFactory` 函数，它返回 `createOrder` 的一个实例，将 ID 号码放在 `createOrder` 上方的范围中。这样，ID 就会在 `createOrder` 的多次调用之间跟踪。

    ```js
    export const orderFactory = (): ((
      color: string,
      size: string,
      qty: number
    ) => Order[]) => {
      let id = 0;
      return (color: string, size: string, qty: number): Order[] => {
        const orders = [];
        for (let i = 0; i < qty; i++) {
          orders.push({ id: id++, color, size });
        }
        return orders;
      };
    };
    ```

    这个工厂函数返回另一个函数，该函数作为箭头函数内联定义。在返回该函数之前，`id` 变量在其上方的作用域中声明。每次调用返回的函数都会看到相同的 `id` 实例，因此它将在调用之间保留其值。

1.  为了使用工厂，请调用一次函数：

    ```js
    const createOrder = orderFactory();
    ```

    调用一次 `orderFactory` 将初始化 ID 变量，并使其在现在分配给 `createOrder` 的返回函数中可用。该变量现在是封装的。没有其他代码能够访问它，或者更重要的是，修改它。

1.  运行程序并观察我们现在得到了正确的输出：

    ```js
    [
      { id: 0, color: 'red', size: 'M' },
      { id: 1, color: 'red', size: 'M' },
      { id: 2, color: 'red', size: 'M' },
      { id: 3, color: 'red', size: 'M' }
    ]
    [
      { id: 4, color: 'blue', size: 'S' },
      { id: 5, color: 'blue', size: 'S' },
      { id: 6, color: 'blue', size: 'S' },
      { id: 7, color: 'blue', size: 'S' },
      { id: 8, color: 'blue', size: 'S' },
      { id: 9, color: 'blue', size: 'S' },
      { id: 10, color: 'blue', size: 'S' }
    ]
    ```

没有实践，闭包可能很难理解。TypeScript 初学者不应该担心立即掌握它们，但认识到工厂模式和封装变量的行为非常重要。

## 柯里化

柯里化（以数学家 Haskell Brooks Curry 命名，Haskell、Brooks 和 Curry 编程语言也是以他的名字命名的）是将一个函数（或数学中的公式）分解成具有单个参数的单独函数的行为。

注意

更多关于柯里化的信息，请参考以下网址：[`javascript.info/currying-partials`](https://javascript.info/currying-partials)。

由于 TypeScript 中的函数可以返回函数，箭头语法为我们提供了一个特殊的简洁语法，使得柯里化成为一种流行的实践。让我们从一个简单的函数开始：

```js
Example_Currying_1.ts
1 const addTwoNumbers = (a: number, b: number): number => a + b;
2 console.log(addTwoNumbers(3, 4));
Link to the preceding example: https://packt.link/InDVT
```

输出如下：

```js
7
```

这里，我们使用了箭头语法来描述一个没有花括号或 `return` 关键字的函数体。该函数返回体中的单个表达式的结果。这个函数期望两个参数，可以重写为每个参数只有一个参数的柯里化函数：

```js
Example_Currying_2.ts
1 const addTwoNumbers = (a: number): ((b: number) => number) => (b: 
2 number): number => a + b;
3 console.log(addTwoNumbers(3)(4));
Link to the preceding example: https://packt.link/975cf
```

输出如下：

```js
7
```

实际上这是两个函数声明。第一个函数返回另一个函数，实际上执行计算。由于闭包，`a` 参数在第二个函数内部可用，以及它自己的参数 `b`。两组括号意味着第一个返回一个新的函数，然后立即被第二个函数调用。前面的代码可以重写为更长的形式：

```js
Example_Currying_3.ts
1 const addTwoNumbers = (a: number): ((b: number) => number) => {
2     return (b: number): number => {
3         return a + b;
4     }
5 }
6 
7 const addFunction = addTwoNumbers(3);
8 
9 console.log(addFunction(4));
Link to the preceding example: https://packt.link/TgC17
```

输出如下：

```js
7
```

当这样写的时候看起来有点傻，但它们确实做了完全相同的事情。

那么 Currying 有什么用途呢？

高阶函数是 Curried 函数的一种形式。高阶函数既接受一个函数作为参数，又返回一个新的函数。这些函数通常封装或修改一些现有的功能。我们如何将我们的 REST 客户端封装在一个高阶函数中，以确保所有响应，无论是成功还是错误，都以统一的方式处理？这将是下一个练习的重点。

## 练习 3.06：将代码重构为 Curried 函数

Currying 技术利用闭包，并且与上一个练习紧密相关，因此让我们回到它，并将上一个练习的解决方案作为本练习的起点。我们的 `orderFactory` 函数正在正常工作并正确跟踪 ID，但每种服装类型的初始化太慢了。当第一次收到红色中号的订单时，我们预计启动这个特定配方需要一些时间，但随后的红色中号订单也遭受了相同的延迟。我们的系统在处理热门商品的需求方面效率不足。我们需要一种方法来减少每次类似订单到来时的设置时间：

注意

本练习的代码文件可以在 [`packt.link/jSKic`](https://packt.link/jSKic) 找到。

1.  查看来自 *练习 3.05，使用闭包创建订单工厂* (`order-solution.ts`) 的代码：

    ```js
    interface Order {
      id: number;
      color: string;
      size: string;
    }
    export const orderFactory = (): ((
      color: string,
      size: string,
      qty: number
    ) => Order[]) => {
      let id = 0;
      return (color: string, size: string, qty: number): Order[] => {
        const orders = [];
        for (let i = 0; i < qty; i++) {
          orders.push({ id: id++, color, size });
        }
        return orders;
      };
    };
    const createOrder = orderFactory();
    const orderOne = createOrder('red', 'M', 4);
    console.log(orderOne);
    const orderTwo = createOrder('blue', 'S', 7);
    console.log(orderTwo);
    ```

    我们如何使用 Currying 来提高效率？你需要将代码重构为 Curried 函数。

1.  将 `orderFactory` 重构为返回 Curried 函数，通过将返回的函数分解为三个单独的函数，每个函数都返回下一个函数：

    ```js
    export const orderFactory = () => {
      let id = 0;
      return (color: string) => (size: string) => (qty: number) => {
        const orders = [];
        for (let i = 0; i < qty; i++) {
          orders.push({ id: id++, color, size });
        }
        return orders;
      };
    };
    ```

    在这种情况下，我们的重构就像在每个参数之间放置一个箭头一样简单。请注意，此代码省略了函数的返回类型。有两个原因。一个是类型可以从代码中合理推断，并且非常清晰。另一个是添加所有返回类型将显著使代码杂乱。

    如果我们将所有返回类型相加，代码将看起来像这样：

    ```js
    export const orderFactory = (): ((
      color: string
    ) => (size: string) => (qty: number) => Order[]) => {
      let id = 0;
      return (color: string): ((size: string) => (qty: number) => Order[]) => (
        size: string
      ) => (qty: number): Order[] => {
        const orders = [];
        for (let i = 0; i < qty; i++) {
          orders.push({ id: id++, color, size });
        }
        return orders;
      };
    };
    ```

    TypeScript 给我们提供了在显式声明类型和允许类型推断之间进行选择的灵活性，当类型清晰时，可以提供正确的类型。

    现在 `orderFactory` 返回 Curried 函数，我们可以利用它。

1.  而不是将每个参数传递给 `createOrder`，我们可以只使用第一个参数来调用 `createOrder`，以建立我们的红色服装线：

    ```js
    const redLine = createOrder('red');
    ```

1.  然后，进一步分解可用的单个项目：

    ```js
    const redSmall = redLine('S');
    const redMedium = redLine('M');
    ```

1.  当需要或适当的时候，在一行中创建一个项目：

    ```js
    const blueSmall = createOrder('blue')('S')
    ```

1.  尝试创建许多不同的订单组合并打印出结果：

    ```js
    const orderOne = redMedium(4);
    console.log(orderOne);
    const orderTwo = blueSmall(7);
    console.log(orderTwo);
    const orderThree = redSmall(11);
    console.log(orderThree);
    ```

1.  当你运行程序时，你会看到以下输出：

    ```js
    [
      { id: 0, color: 'red', size: 'M' },
      { id: 1, color: 'red', size: 'M' },
      { id: 2, color: 'red', size: 'M' },
      { id: 3, color: 'red', size: 'M' }
    ]
    //...
    ```

    注意

    为了便于展示，这里只显示了实际输出的部分。

Currying 是一种强大的技术，用于缓存变量和部分函数结果。到目前为止，我们已经探讨了闭包、高阶函数和 Currying，这些都展示了 TypeScript 中函数的强大功能和多功能性。

# 函数式编程

函数式编程是一个深奥的话题，本身就是一个许多书籍的主题。本书只能触及这个话题的皮毛。函数式编程的一个基础概念是使用具有输入和输出且不修改其作用域之外变量的简单函数：

```js
Example_Functional_1.ts
1 let importantNumber = 3;
2
3 const addFive = (): void => {
4     importantNumber += 5;
5 };
6 
7 addFive();
8 
9 console.log(importantNumber);
Link to the preceding example: https://packt.link/CTn1X
```

函数产生以下输出：

```js
8
```

该程序的输出是正确的。我们确实将`5`加到了初始值`3`上，但`addFive`方法访问了更高作用域中的变量并对其进行了修改。在函数式编程范式下，更倾向于返回新值并允许外部作用域控制在其中声明的变量。我们可以修改`addFive`，使其不再操作其作用域之外的变量，而是仅对其参数操作并返回正确的值：

```js
Example_Functional_2.ts
1 let importantNumber = 3;
2
3 const addFive = (num: number): number => {
4     return num + 5;
5 };
6 
7 importantNumber = addFive(importantNumber);
8 
9 console.log(importantNumber);
Link to the preceding example: https://packt.link/6fWcF.
```

函数产生以下输出：

```js
8
```

函数现在更加便携。由于它不依赖于更高作用域中的任何东西，因此更容易测试或重用。函数式编程范式鼓励使用更小的函数。有时，程序员可能会编写执行太多不同功能的函数，这些函数难以阅读和维护。这通常是错误或对团队速度产生负面影响的原因。通过保持函数小而简单，我们可以以支持维护和可重用的方式链接逻辑。

函数式编程中的一个流行概念是不可变性。这就是一旦声明了变量，其值就不应该改变的概念。为了理解这为什么会是一个理想的特性，考虑一个需要打印客户 ID 的程序，客户名称之后：

```js
Example_Functional_3.ts
1 const customer = {id: 1234, name: 'Amalgamated Materials'}
2 
3 const formatForPrint = ()=> {
4   customer.name = `${customer.name} id: ${customer.id}`;
5 };
6 
7 formatForPrint();
8 
9 console.log(customer.name);
Link to the preceding example: https://packt.link/TX81Z
```

该程序按预期运行。当打印客户名称时，它后面有 ID；然而，我们实际上已经更改了客户对象中的名称：

```js
Amalgamated Materials id: 1234
```

如果`formatForPrint`被反复调用会发生什么？通过微小的重构，我们的代码更加安全和一致：

```js
const customer = {id: 1234, name: 'Amalgamated Materials'}
const formatForPrint = ()=> {
  return `${customer.name} id: ${customer.id}`;
};
console.log(formatForPrint());
```

输出如下：

```js
Amalgamated Materials id: 1234
```

如果将客户对象传递进去而不是让`formatForPrint`在更高作用域中访问它，会更好。

TypeScript 支持函数式编程和面向对象范式。许多应用程序都借鉴了两者。

# 将函数组织成对象和类

有时候，将函数组织成对象和类的成员函数是有意义的。这些概念将在第四章“类和对象”中更详细地介绍，但就目前而言，我们可以考察如何将一个函数声明添加到对象或类中。

让我们考虑一个简单的函数：

```js
Example_OrganizingFuncs_1.ts
1 function addTwoNumbers(a: number, b: number) { return a + b; }
```

如果我们想要一个包含多个数学函数的对象，我们可以简单地添加以下函数到其中：

```js
2 const mathUtils = {
3     addTwoNumbers
4 };
5 
6 console.log(mathUtils.addTwoNumbers(3, 4));
Link to the preceding example: https://packt.link/qX1QO
```

输出如下：

```js
7
```

注意在`mathUtils`对象中使用的语法是简写，意味着赋值语句的左右两侧是相同的。这也可以写成这样：

```js
Example_OrganizingFuncs_2.ts
5 const mathUtils = {
6     addTwoNumbers: addTwoNumbers
7 };
```

我们还可以选择使用函数表达式定义方法：

```js
5 const mathUtils = {
6     addTwoNumbers: function(a: number, b: number) { return a + b; }
7 };
Link to the preceding example: https://packt.link/Ew4vi
```

无论是哪种情况，输出结果如下：

```js
7
```

记住，函数表达式通常是在对象中使用最好的东西，因为它们将具有正确的`this`引用。在我们的`mathUtils`对象中，我们没有使用`this`关键字，所以可以使用箭头函数，但请注意，如果稍后其他开发者重构此对象，他们可能不会想到将箭头函数更改为函数表达式，你可能会得到有错误的代码。

向类中添加函数可以完全以相同的方式进行，实际上语法也非常相似。假设我们想使用类而不是普通对象，并希望内联定义`addTwoNumbers`。`MathUtils`类可能看起来像这样：

```js
class MathUtils {
    addTwoNumbers(a: number, b: number) { return a + b; }
};
```

现在我们正在使用类，为了调用函数，我们需要实例化一个对象：

```js
const mathUtils = new MathUtils();
console.log(mathUtils.addTwoNumbers(3, 4));
```

输出结果如下：

```js
7
```

有关类的更多信息，请参阅*第四章*，*类和对象*。

## 练习 3.07：将 JavaScript 重构为 TypeScript

将较旧的 JavaScript 代码更新为 TypeScript 并不困难。如果原始代码编写得很好，我们可以保留大部分结构，并通过接口和类型来增强它。在这个练习中，我们将使用一个示例遗留 JavaScript 代码，该代码根据给定的尺寸打印各种形状的面积：

注意

这个练习的代码文件可以在[`packt.link/gRVxx`](https://packt.link/gRVxx)找到。

1.  从以下遗留代码开始，并决定我们希望通过将其转换为 TypeScript 来改进什么：

    ```js
    var PI = 3.14;
    function getCircleArea(radius) {
      return radius * radius * PI;
    }
    //...
    ```

    注意

    这里只展示了实际代码的一部分。完整的代码可以在[`packt.link/pahq2`](https://packt.link/pahq2)找到。

    其中一些更改很简单。我们将用`const`替换`var`。确定面积的函数相当不错，但`getArea`会修改形状对象。最好是只返回面积。我们的所有形状都定义得相当好，但通过接口可以进一步改进。

1.  让我们创建一些接口。在 VS Code 中创建一个新文件，并将其保存为`refactor-shapes-solution.ts`。

1.  首先，创建一个包含枚举类型和面积属性的`Shape`接口。我们可以从这个接口扩展我们的`Circle`、`Square`、`Rectangle`和`RightTriangle`接口：

    ```js
    const PI = 3.14;
    interface Shape {
      area?: number;
      type: 'circle' | 'rectangle' | 'rightTriangle' | 'square';
    }
    interface Circle extends Shape {
      radius: number;
      type: 'circle';
    }
    interface Rectangle extends Shape {
      length: number;
      type: 'rectangle';
      width: number;
    }
    interface RightTriangle extends Shape {
      base: number;
      height: number;
      type: 'rightTriangle';
    }
    interface Square extends Shape {
      type: 'square';
      width: number;
    }
    ```

1.  现在，让我们改进并简化`getArea`。而不是访问每个形状上的属性，`getArea`可以简单地传递形状到正确的函数以确定面积，然后返回计算值：

    ```js
    const getArea = (shape: Shape) => {
      switch (shape.type) {
        case 'circle':
          return getCircleArea(shape as Circle);
        case 'rectangle':
          return getRectangleArea(shape as Rectangle);
        case 'rightTriangle':
          return getRightTriangleArea(shape as RightTriangle);
        case 'square':
          return getSquareArea(shape as Square);
      }
    };
    ```

    这个更改要求我们对所有计算面积的函数进行一些小的修改。

1.  现在不是每个单独的属性都传递进来，而是传递形状，然后在函数内部获取属性：

    ```js
    const getCircleArea = (circle: Circle): number => {
      const { radius } = circle;
      return radius * radius * PI;
    };
    const getRectangleArea = (rectangle: Rectangle): number => {
      const { length, width } = rectangle;
      return length * width;
    };
    const getSquareArea = (square: Square): number => {
      const { width } = square;
      return getRectangleArea({ length: width, type: 'rectangle', width });
    };
    const getRightTriangleArea = (rightTriangle: RightTriangle): number => {
      const { base, height } = rightTriangle;
      return (base * height) / 2;
    };
    ```

    这种模式在现代 Web 应用开发中非常常见，在 TypeScript 开发中也非常有效。

1.  在我们的对象声明中添加一些类型提示：

    ```js
    const circle: Circle = { radius: 4, type: 'circle' };
    console.log({ ...circle, area: getArea(circle) });
    const rectangle: Rectangle = { type: 'rectangle', length: 7, width: 4 };
    console.log({ ...rectangle, area: getArea(rectangle) });
    const square: Square = { type: 'square', width: 5 };
    console.log({ ...square, area: getArea(square) });
    const rightTriangle: RightTriangle = {
      type: 'rightTriangle',
      base: 9,
      height: 4,
    };
    console.log({ ...rightTriangle, area: getArea(rightTriangle) });
    ```

1.  运行程序会得到正确的输出：

    ```js
    { radius: 4, type: 'circle', area: 50.24 }
    { type: 'rectangle', length: 7, width: 4, area: 28 }
    { type: 'square', width: 5, area: 25 }
    { type: 'rightTriangle', base: 9, height: 4, area: 18 }
    ```

这个练习为我们提供了将遗留 JavaScript 代码重构为 TypeScript 的实际经验。这些技能可以帮助我们识别原始 JavaScript 代码中的代码质量问题，并在将代码迁移到 TypeScript 时进行改进。

# 导入、导出和需求

非常小的程序，如编程书籍中经常找到的那种，可以与单个文件中的所有代码一起工作得很好。大多数时候，应用程序将由多个文件组成，通常被称为模块。一些模块可能是从 Node 包管理器（`npm`）安装的依赖项，而另一些可能是你或你的团队编写的模块。当你查看其他项目时，你可能会看到用于链接不同模块的关键字`import`、`export`、`module`和`require`。`import`和`require`都服务于相同的目的。它们允许你在当前正在工作的模块（文件）中使用另一个模块。`export`和`module`则相反。它们允许你使你的模块的部分或全部可供其他模块使用。

我们将在下面介绍不同的语法选项。之所以有这么多做事情的方法，通常是因为语言和运行时的发展方式。Node.js 是目前最流行的服务器端 JavaScript 运行时，大多数编译的服务器端 TypeScript 代码都将在这里运行。Node.js 于 2009 年发布，当时 JavaScript 还没有标准模块系统。当时许多 JavaScript 网络应用程序会简单地将函数和对象附加到全局 window 对象上。对于网络应用程序来说，这可以工作得很好，因为 window 对象在页面加载时会刷新，存在于网络浏览器中，所以它只被单个用户使用。

尽管 Node.js 中存在全局对象，但这并不是将模块链接在一起的实际方法。这样做可能会风险一个模块覆盖另一个模块，内存泄漏，暴露客户数据，以及各种其他灾难。模块系统的好处在于，你可以只共享你打算共享的模块部分。

由于需要更健壮的解决方案，Node.js 采用了 CommonJS 规范以及`module`和`require`关键字。`module`用于共享你的模块的全部或部分，而`require`用于消费另一个模块。这些关键字在 Node.js 中是标准的，直到 ECMAScript 6 引入了`import`和`export`语法。后者在 TypeScript 中已经支持多年，并且是首选的，尽管`require`语法仍然是有效的，并且可以继续使用。

本书将使用`import`和`export`语法，因为这是标准的。下面的示例将使用这种语法，但也将包含`require`语法的注释，以便读者可以进行比较。

任何包含`import`或`export`关键字的文件都被视为模块。模块可以导出它们声明的任何变量或函数，无论是作为声明的一部分还是通过显式地这样做：

```js
// utils.ts
export const PI = 3.14;
export const addTwoNumbers = (a: number, b: number): number => a + b;
```

这与显式导出等效。以下是 `utils.ts` 的完整代码：

```js
Example_Import_Exports/utils.ts
1 // utils.ts
2 const PI = 3.14;
3 
4 const addTwoNumbers = (a: number, b: number): number => a + b;
5
6 export { PI, addTwoNumbers };
7 // module syntax:
8 // module.exports = { PI, addTwoNumbers };
Link to the preceding example: https://packt.link/3FEbm
```

现在，我们可以将我们的导出导入到另一个模块中（另一个 `.ts` 文件 - `app.ts`）：

```js
Example_Import_Exports/app.ts
1 // app.ts
2 import { PI, addTwoNumbers } from './utils';
3 // require syntax:
4 // const { PI, addTwoNumbers } = require('./utils');
5 console.log(PI);
6 console.log(addTwoNumbers(3, 4));
Link to the preceding example: https://packt.link/ozz9N
```

一旦运行 `app.ts`，你将获得以下输出：

```js
3.14
7
```

注意

前例的代码文件可以在以下位置找到：[`packt.link/zsCDe`](https://packt.link/zsCDe)

我们应用程序的一部分模块通过从项目根目录的相对路径导入。从我们安装的依赖项导入的模块通过名称导入。注意，文件扩展名不是必需路径的一部分，只是文件名。

模块也可以有默认导出，使用 `default` 关键字。默认导出不需要括号。考虑以下示例：

```js
Example_Import_Export_2/utils.ts
1 // utils.ts
2 const PI = 3.14;
3 const addTwo = (a: number, b: number): number => {
4   return a + b;
5 };
6 const fetcher = () => {
7   console.log('it is fetched!');
8 };
9 export default { addTwo, fetcher, PI };
Link to the preceding example: https://packt.link/h3R4r
```

`app.ts` 的代码如下：

```js
1 // app.ts
2 import utils from './utils';
3 console.log(utils.addTwo(3, 4));
Link to the preceding example: https://packt.link/oamFn
```

一旦运行 `app.ts` 文件，你将得到以下输出：

```js
7
```

## 练习 3.08：导入和导出

回顾上一个练习，我们有一个包含许多实用函数的单个文件，然后我们有过程式代码创建一些对象，调用函数并输出日志。让我们重构练习 3.07，将 JavaScript 重构为 TypeScript，使用 `import` 和 `export` 关键字，并将这些函数移动到单独的模块中：

注意

本练习的代码文件可以在 [`packt.link/2K4ds`](https://packt.link/2K4ds) 找到。本练习的第一步要求你将一些代码行复制粘贴到练习文件中。因此，我们建议你在开始此练习之前，要么从本存储库下载代码文件，要么将其迁移到您的桌面。

1.  将 `shapes.ts` 的前 61 行剪切粘贴到 `shapes-lib.ts` 中。您的 IDE 应该开始警告您它无法再找到相关的函数。

1.  查看代码在 `shapes-lib.ts` 中的情况。哪些函数和接口需要导出？正方形、圆形以及其他直接在 `shapes.ts` 中使用，但形状接口没有使用，所以只需要这四个导出。同样，PI 常量仅在 `shapes-lib.ts` 中使用，因此不需要导出该常量：

    ```js
    const PI = 3.14;
    interface Shape {
      area?: number;
      type: 'circle' | 'rectangle' | 'rightTriangle' | 'square';
    }
    export interface Circle extends Shape {
      radius: number;
      type: 'circle';
    }
    export interface Rectangle extends Shape {
      length: number;
      type: 'rectangle';
      width: number;
    }
    export interface RightTriangle extends Shape {
      base: number;
      height: number;
      type: 'rightTriangle';
    }
    export interface Square extends Shape {
      type: 'square';
      width: number;
    }
    ```

1.  只需要导出的函数是 `getArea`，因为它是 `shapes.ts` 中唯一引用的：

    ```js
    export const getArea = (shape: Shape) => {
      switch (shape.type) {
        case 'circle':
          return getCircleArea(shape as Circle);
        case 'rectangle':
          return getRectangleArea(shape as Rectangle);
        case 'rightTriangle':
          return getRightTriangleArea(shape as RightTriangle);
        case 'square':
          return getSquareArea(shape as Square);
      }
    };
    ```

1.  现在，让我们将导出的接口和函数导入到 `shapes.ts` 中。您的 IDE 可能会协助您完成此任务。例如，在 VS Code 中，如果将鼠标悬停在可以导入的模块上，它应该询问您是否要添加导入：

    ```js
    import {
      Circle,
      getArea,
      Rectangle,
      RightTriangle,
      Square,
    } from './shapes-lib-solution';
    ```

1.  在设置好所有导入和导出后，再次运行程序。你应该得到正确的结果：

    ```js
    { radius: 4, type: 'circle', area: 50.24 }
    { type: 'rectangle', length: 7, width: 4, area: 28 }
    { type: 'square', width: 5, area: 25 }
    { type: 'rightTriangle', base: 9, height: 4, area: 18 }
    ```

学习一门新编程语言更具挑战性的事情之一是如何构建模块。一个好的经验法则是，如果模块变得太大，总是准备好将它们拆分成更小的块。这个练习帮助我们理解如何将应用程序逻辑与实用程序或可重用函数分开，这种做法将导致代码整洁、易于维护。

## 活动 3.01：使用函数构建航班预订系统

作为一家在线预订初创公司的开发者，你需要实现一个管理航空公司预订的系统。这个系统的架构已经确定。将有一个用于管理航班和座位可用性的系统，以及一个用于管理预订的系统。用户将直接与预订系统交互，而该系统将搜索并更新航班信息。

为了使这项活动保持可管理的规模，我们将抽象出许多事物，例如客户信息、支付、航班日期，甚至出发城市。在理解我们需要解决的问题时，创建一个描述我们需要实现的流程的图表非常有帮助。以下图表显示了我们的用户预期的流程：

备注

这个活动的代码文件可以在以下位置找到：[`packt.link/o5n0t`](https://packt.link/o5n0t)。

![图 3.1：需要在航班预订系统中实现的流程]

![备注](img/B14508_03_01.jpg)

图 3.1：需要在航班预订系统中实现的流程

程序的流程如下：

1.  获取可供选择的航班列表。

1.  从这些航班中选择一个开始预订。

1.  为航班支付费用。

1.  在航班上预留座位后完成预订。

如图表所示，用户将与两个不同的系统交互，即预订系统和航班系统。在大多数情况下，用户与预订系统交互，但他们直接访问航班系统来搜索航班。

在这个活动中，这些系统可以用一个`bookings.ts`文件和一个`flights.ts`文件来表示，这两个文件是两个 TypeScript 模块。为了完成这个活动，请使用 TypeScript 实现这两个模块。以下是一些步骤来帮助你：

1.  由于用户和预订系统都依赖于航班系统，因此我们应该从航班系统开始，即`flights.ts`。由于活动被简化了，当用户想要访问航班时，我们可以简单地返回一个目的地列表。为了允许访问`bookings.ts`模块，我们将在一个函数上使用`export`关键字。

1.  尽管用户已经获取了航班信息，但在开始预订之前，我们需要检查可用性。这是因为我们的系统将有多个用户，可用性可能会每分钟变化。公开一个用于检查可用性的函数，另一个用于在交易完成前保留座位。

1.  支付流程实际上暗示了支付系统的第三个系统，但我们将不会在这个活动中包含该系统，所以当用户到达支付步骤时，只需将预订标记为已支付即可。航班系统不需要知道支付状态，因为这是由预订系统管理的。

1.  当我们完成预订时，保留的座位将转换为预留座位。我们的预订已经完成，座位不再在航班上可用。

1.  这种活动的典型输出将看起来像这样：

    ```js
    Booked to Lagos {
      bookingNumber: 1,
      flight: {
        destination: 'Lagos',
        flightNumber: 1,
        seatsHeld: 0,
        seatsRemaining: 29,
        time: '5:30'
      },
      paid: true,
      seatsHeld: 0,
      seatsReserved: 1
    //...
    ```

    备注

    为了便于展示，这里只显示了实际输出的部分。此活动的解决方案可以通过此链接找到。

在这里还有许多其他场景可以探索。尝试保留所有剩余的座位，未能为该航班启动新的预订，然后完成原始预订。这应该会与我们在这里实现的逻辑兼容！这个练习使用了几个函数来创建一个连贯的程序。它使用了闭包、柯里化、函数式编程概念以及`import`和`export`关键字在模块之间共享函数。

# 使用 ts-jest 进行单元测试

大型系统需要持续测试以确保它们是正确的。这就是单元测试发挥作用的地方。世界上一些最大的软件项目有数亿行代码和数千个特性和视图。手动测试每个特性是不可能的。这就是单元测试发挥作用的地方。单元测试测试代码的最小单元，通常是单个语句或函数，并给我们提供快速反馈，如果我们做了什么来改变应用程序的行为。短的反馈周期是开发者的好朋友，而单元测试是实现它们的强大工具之一。

有许多测试框架可以帮助我们进行单元测试我们的代码。Jest 是来自 Facebook 的一个流行的测试框架。你也可能遇到其他框架，如 Jasmine、Mocha 或 Ava。Jest 是一个“包含电池”的框架，对于使用那些其他框架的用户来说会感觉熟悉，因为它试图整合它们中的所有最佳特性。

Jest、Mocha、Ava 以及其他都是 JavaScript 库，而不是 TypeScript 库，因此在使用它们之前需要做一些特殊准备。`ts-jest`是一个库，它帮助我们用 TypeScript 编写 TypeScript 测试，并使用 Jest 测试运行器和 Jest 的所有优点。

要开始，我们将安装`jest`、`ts-jest`和用于`jest`的`typings`（`@types/jest`）：

```js
npm install -D jest ts-jest @types/jest
```

一旦安装了库，我们就可以使用`npx`来使用默认配置初始化`ts-jest`，这将允许我们编写我们的第一个测试：

```js
npx ts-jest config:init
```

运行此命令将创建一个名为`jest.config.js`的配置文件。随着你越来越习惯于使用 Jest 编写测试，你可能希望修改此文件，但到目前为止，默认设置将完全适用。

一些开发者将单元测试放在测试目录中，而另一些则将测试直接放在源代码旁边。我们的默认 Jest 配置将找到这两种类型的测试。单元测试的惯例是测试模块的名称，后面跟着一个点，然后是单词`spec`或`test`，然后是文件扩展名，在我们的情况下将是`ts`。如果我们创建符合这种命名约定的文件，并且这些文件位于我们的项目根目录下，Jest 将能够找到并执行这些测试。

让我们添加一个简单的测试。创建一个名为 `example.spec.ts` 的文件。然后向该文件添加以下代码。这段代码只是测试的占位符，实际上并没有做任何事情，只是验证 Jest 是否正确工作：

```js
describe("test suite for `sentence`", () => {
  test("dummy test", () => {
    expect(true).toBeTruthy();
  });
});
```

我们可以通过在控制台输入 `npx jest` 来运行 Jest，或者我们可以添加一个 `npm` 脚本。尝试在控制台输入 `npm test`。如果你没有更改默认测试，你应该看到以下类似的内容：

```js
npm test
> ex1@1.0.0 test /Users/mattmorgan/typescript/function-chapter/exercises
> echo "Error: no test specified" && exit 1
Error: no test specified
npm ERR! Test failed.  See above for more details.
```

现在让我们更新 `package.json` 文件，使其运行 Jest 而不是直接失败。找到 `package.json` 文件，你会在其中看到以下配置：

```js
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
```

我们可以用简单的 `jest` 替换整个测试：

```js
  "scripts": {
    "test": "jest"
  },
```

现在，再次尝试 `npm test`：

```js
npm test
> ex1@1.0.0 test /Users/mattmorgan/typescript/function-chapter/exercises
> jest
 PASS  ./example.spec.ts
  test suite for `sentence`
    ✓ dummy test (1ms)
Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        1.449s
Ran all test suites
```

当然，这个测试没有做任何有用的事情。现在，让我们导入我们想要测试的函数，并编写一些真正有用的测试。首先，让我们清理一下 `arrow-cat-solution.ts` 文件（来自练习 3.03，编写箭头函数）。我们可以删除所有的控制台语句，因为我们将通过编写测试来验证我们的代码，而不是仅仅记录控制台。然后，让我们为每个函数添加 `export` 关键字，以便我们的测试可以导入它们。`arrow-cat-solution.ts` 现在看起来是这样的：

```js
export const arrayToAnd = (words: string[]) => {
  return words.reduce((prev, curr, index) => {
    if (words.length === 1) {
      return ` ${curr}`;
    }
    if (words.length - 1 === index) {
      return `${prev} and ${curr}`;
    }
    return `${prev} ${curr},`;
  }, "");
};
export const capitalize = (sentence: string) => {
  return `${sentence.charAt(0).toUpperCase()}${sentence
    .slice(1)
    .toLowerCase()}`;
};
export const sentence = (
  subject: string,
  verb: string,
  ...objects: string[]
): string => {
  return capitalize(`${subject} ${verb}${arrayToAnd(objects)}.`);
};
```

让我们尝试编写一个针对 `capitalize` 函数的测试。我们只需要调用该函数，并将结果与预期结果进行比较。首先，在一个新文件中导入该函数 (`arrow-cat-solution.spec.ts`)：

```js
import { capitalize } from './arrow-cat-solution'; 
```

然后，编写一个期望。我们期望我们的函数将全大写的 "HELLO" 转换为 "Hello"。现在让我们编写这个测试并执行它：

```js
describe("test suite for `sentence`", () => {
  test("capitalize", () => {
    expect(capitalize("HELLO")).toBe("Hello");
  });
});
```

它工作了吗？

```js
npm test
> ex1@1.0.0 test /Users/mattmorgan/typescript/function-chapter/exercises
> jest
 PASS  ./example.spec.ts
  test suite for `sentence`
    ✓ capitalize (1ms)
Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        0.502s, estimated 2s
Ran all test suites.
```

`describe` 关键字用于分组测试，它的唯一目的是影响测试报告的输出。`test` 关键字应该包装实际的测试。除了 `test`，你还可以写 `it`。使用 `it` 的测试通常写成带有 `should` 的断言：

```js
  it("should capitalize the string", () => {
    expect(capitalize("HELLO")).toBe("Hello");
  });
```

现在，为其他函数编写测试。

## 活动三.02：编写单元测试

在上一个活动中，我们为航空公司构建了一个预订系统，并将 TypeScript 函数应用于确保航班预订涉及的场景。我们从单个 `index.ts` 文件中执行这些场景，代表用户交互。这种方法在我们学习时足够好，但它有点杂乱，实际上并没有断言任何场景是正确的。换句话说，它几乎是一个单元测试，但并不如单元测试那样好。

我们已经学习了如何安装 Jest，所以让我们使用它来对 *活动 3.01，使用函数构建航班预订系统* 进行单元测试。对于每个我们编写的函数，我们将编写一个测试来调用该函数并测试输出：

注意

本活动的代码文件可以在 [`packt.link/XMOZO`](https://packt.link/XMOZO) 找到。

1.  为本活动提供的代码占位符包括 `bookings.test.ts` 和 `flights.test.ts`，其中包含一些未实现的测试。实现这些测试以完成本活动。

    你可以通过运行 `npm test` 来执行测试。你也可以只运行解决方案，使用 `npm run test:solution`。

1.  要测试一个函数，你需要将其 `import` 到你的测试文件中。

1.  使用示例输入调用函数，然后使用 Jest 的 `expect` 断言来测试输出，例如，`expect(value).toBe(5);`。

1.  可以使用 `try/catch` 块来测试错误场景，捕获函数抛出的错误，然后测试错误条件。在单元测试中使用 `catch` 时，使用 `expect.assertions` 来指示你想要测试多少断言是一个最佳实践。否则，你的测试可能在没有调用 `catch` 块的情况下完成。

1.  尝试在覆盖率报告中达到 100% 的行覆盖率（已经使用 `--coverage` 配置）。

    注意

    此活动的解决方案可以通过此链接找到。

在这个活动中，我们使用了一些好的单元测试来应用最佳实践，我们编写的一个程序。现在，我们知道现有代码已经过测试，添加额外的功能和场景会容易得多。我们不再需要编写索引文件来调用各种函数，现在我们有逻辑上分组、排序和测试的东西。我们有一个机制来跟踪行覆盖率，并了解我们的代码中有多少是经过测试的。

# 错误处理

当我们编写函数时，我们需要牢记并非所有事情总是完美无缺。如果函数接收到意外的输入，我们会怎么做？如果我们需要调用的某个函数工作不完美，我们的程序会如何反应？始终验证函数输入是一个好主意。是的，我们正在使用 TypeScript，我们可以合理地确信，如果我们期望一个字符串，我们不会得到一个对象，但有时，外部输入不符合我们的类型。有时，我们自己的逻辑可能是错误的。考虑这个函数：

```js
const divide = (numerator: number, denominator: number) => {
    return numerator / denominator;
}
```

看起来不错，但如果我们传入数字 `0` 作为分母会怎样？我们不能除以零，所以结果将是常数，`NaN`。`NaN` 在任何数学方程中使用时，总会返回 `NaN`。这可能会在我们的系统中引入一个严重的错误，这需要避免。

为了解决这个问题，我们需要弄清楚如果得到无效输入会发生什么。记录它？抛出错误？只是返回零？退出程序？一旦决定，我们就可以在我们的函数中添加一些验证：

```js
const divide = (numerator: number, denominator: number) => {
    if(denominator === 0) {
        throw 'Cannot divide by zero!'
    }
    return numerator / denominator;
}
```

至少我们现在不会默默失败，因为我们已经在屏幕上显示了一个警告，`Cannot divide by zero!`。抛出异常总比函数失败而无人察觉要好。

# 摘要

到现在为止，你知道如何创建任何 TypeScript 程序最重要的构建块——函数。我们已经探讨了函数表达式和箭头函数之间的区别，以及何时使用哪一个。我们研究了立即执行的函数表达式、闭包、柯里化和其他强大的 TypeScript 技术。

我们讨论了函数式编程范式，并探讨了如何在对象和类中包含函数。我们还研究了如何将遗留的 JavaScript 代码转换为现代 TypeScript，以及通过这样做如何改进我们的软件。

我们概述了 TypeScript 模块系统和至关重要的`import`和`export`关键字。我们编写了大量的 TypeScript 代码，并学习了如何使用`ts-jest`对其进行测试。

最后，我们通过讨论错误处理来完善了本章内容。在异步编程方面，我们将更深入地探讨更高级的错误处理技术，包括在第十二章“TypeScript 中的 Promise 指南”和第十三章“TypeScript 中的 Async Await”。

本章涵盖了相当多的主题，大多数读者可能不会立即记住所有内容。这没关系！你已经在本章中编写了许多函数，在接下来的章节中你还将编写更多。编写好的函数是一种随着实践而提高的技能，随着你在 TypeScript 掌握方面的进步，你将能够回过头来参考本章以检查你的学习情况。

在下一章中，我们将通过研究`class`关键字以及如何构建类型安全的对象，进一步探索面向对象编程范式。
