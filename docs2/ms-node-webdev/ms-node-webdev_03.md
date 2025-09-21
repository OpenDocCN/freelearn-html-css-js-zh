

# 第三章：JavaScript 和 TypeScript 入门

开发者通过多种途径进入网络应用程序开发的世界，并不总是基于网络应用程序所依赖的基本技术。在本章中，我介绍了 JavaScript 和 TypeScript 的基本功能。这并不是这两种语言的全面指南，但它涵盖了关键内容，并会为你提供开始所需的知识。

# 准备本章内容

为了准备本章，在方便的位置创建一个名为 `primer` 的文件夹。导航到 `primer` 文件夹并运行 *清单 3.1* 中显示的命令。

清单 3.1：准备项目文件夹

```js
npm init --yes 
```

在 `primer` 文件夹中运行 *清单 3.2* 中显示的命令以安装本章中使用的开发包。

清单 3.2：安装开发包

```js
npm install nodemon@2.0.20
npm install tsc-watch@6.0.4
npm install typescript@5.2.2
npm install @tsconfig/node20@20.1.4
npm install @types/node@20.6.1 
```

在本章开始时将使用 `nodemon` 包来监控和执行 JavaScript 文件。`tsc-watc1h` 包对 TypeScript 文件做同样的事情，而 `typescript` 包包含 TypeScript 编译器。`@tsconfig/node20` 包包含用于 Node.js 项目的 TypeScript 编译器的配置设置。

如 *清单 3.3* 所示，替换 `package.json` 文件中的 `scripts` 部分，这将使其更容易使用开发包。

清单 3.3：替换 primer 文件夹中 package.json 文件中的脚本部分

```js
{
  "name": "primer",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
   ** "use_js": "nodemon",**
 **"****use_ts": "tsc-watch --onSuccess \"node index.js\""**
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "@tsconfig/node20": "²⁰.1.4",
    "@types/node": "²⁰.6.1",
    "nodemon": "².0.20",
    "tsc-watch": "⁶.0.4",
    "typescript": "⁵.2.2"
  }
} 
```

在 `primer` 文件夹中添加一个名为 `tsconfig.json` 的文件，其内容如 *清单 3.4* 所示，这将为适用于 Node.js 项目的 TypeScript 编译器创建一个基本配置。

清单 3.4：primer 文件夹中 tsconfig.json 文件的内容

```js
{
    "extends": "@tsconfig/node20/tsconfig.json"
} 
```

在 `primer` 文件夹中添加一个名为 `index.js` 的文件，其内容如 *清单 3.5* 所示。

清单 3.5：primer 文件夹中 index.js 文件的内容

```js
console.log("Hello, World"); 
```

在 `primer` 文件夹中运行 *清单 3.6* 中显示的命令以开始监控和执行 JavaScript 文件。

清单 3.6：启动开发工具

```js
npm run use_js 
```

监视器将生成类似以下内容的输出，并将包括 *清单 3.5* 中语句编写的消息：

```js
[nodemon] 2.0.20
[nodemon] to restart at any time, enter `rs`
[nodemon] watching path(s): *.*
[nodemon] watching extensions: js,mjs,json
[nodemon] starting `node index.js`
**Hello, World**
[nodemon] clean exit - waiting for changes before restart 
```

任何对 `index.js` 文件的更改都将由 `nodemon` 包检测，并由 Node.js 运行时执行。

# 理解 JavaScript 的困惑

JavaScript 是一种令人难以置信的语言，它一直是网络应用程序开发的变革引擎。我喜欢 JavaScript，并且愿意向任何愚蠢到询问的人赞美它的优点；它是我使用过的最流畅和最具表现力的语言之一。

话虽如此，JavaScript 略显奇特，它会引起困惑。乍一看，JavaScript 看起来像任何其他编程语言，这给对语言新手程序员带来了一种自信感。但这种自信不会持续太久，并且不久之后，Stack Overflow 上的独立搜索就会开始。

JavaScript 与其他主流语言不同。要看到最令人困惑的特性，将 `index.js` 文件的内容替换为 *清单 3.7* 中显示的代码。

清单 3.7：替换 primer 文件夹中 index.js 文件的内容

```js
**function sum(first, second) {**
 **return first + second;**
**}**
**let result =** **sum(10, 10);**
**console.log(`Result value: ${result}, Result type: ${typeof result}`);**
**result = sum(****10, "10");**
**console.log(`Result value: ${result}, Result type: ${typeof result}`);** 
```

保存更改后，文件的内容将被执行，产生以下结果：

```js
Result value: 20, Result type: number
Result value: 1010, Result type: string 
```

有两个对名为 `sum` 的函数的调用，JavaScript 允许使用不同的类型作为函数参数。第一个调用使用两个数字值（`10` 和 `10`）。第二个调用使用一个数字值（`10`）和一个字符串值（`"10"`）。

JavaScript 是 *动态类型*，这意味着变量不受特定类型值的限制，任何类型的值都可以分配给任何变量，包括函数参数。

如果你查看 *清单 3.7* 生成的输出，你会看到函数结果异常不同，并且具有不同的类型：

```js
Result value: **20**, Result type: **number**
Result value: **1010**, Result type: **string** 
```

JavaScript 也是 *弱类型*，这意味着值将被隐式转换，以便它们可以一起使用，这个过程称为 *类型强制转换*。这可以是一个方便的特性，但它可能导致意外的结果，因为值根据执行的操作以不同的方式被强制转换。当 `+` 运算符应用于一对 `number` 值时，JavaScript 将两个值相加以产生一个 `number` 值。如果 `+` 运算符应用于 `string` 和 `number` 值，那么 JavaScript 将 `number` 值转换为 `string` 并连接值以产生一个 `string` 结果。这就是为什么 `"10" + 10` 产生 `string` 结果 `1010`，而 `10 + 10` 产生数字结果 `20`。

## 使用 JavaScript 特性表达类型预期

JavaScript 处理数据类型的方式可能会令人困惑，尤其是在初次使用该语言时，但一旦你理解了发生了什么，其行为就是一致和可预测的。

一个更大的问题是，可能很难传达编写 JavaScript 代码时使用的假设和预期。`sum` 函数非常简单，但更复杂的函数可能会很难确定预期的数据类型和返回的数据类型。

JavaScript 提供了检查类型的特性，可以用来强制类型预期，如 *清单 3.8* 所示。

清单 3.8：在 primer 文件夹中的 index.js 文件中检查类型

```js
function sum(first, second) {
   ** if (typeof first == "number" && typeof second == "number") {**
 **return** **first + second;**
 **}**
 **throw Error("Expected two numbers");**
}
let result = sum(10, 10);
console.log(`Result value: ${result}, Result type: ${typeof result}`);
result = sum(10, "10");
console.log(`Result value: ${result}, Result type: ${typeof result}`); 
```

使用 `typeof` 关键字检查两个参数都是 `number` 值，并在接收到任何其他类型时使用 `throw` 关键字创建错误。当代码执行时，对 `sum` 函数的第一个调用是成功的，但第二个调用失败了：

```js
Result value: 20, Result type: number
C:\primer\index.js:5
    throw Error("Expected two numbers");
    ^
Error: Expected two numbers at sum (C:\primer\index.js:5:11) 
```

这类类型检查是有效的，但它们仅在 JavaScript 代码执行时应用，这意味着需要进行彻底的测试，以确保 `sum` 函数没有被错误类型调用。

## 使用 JavaScript 检查类型预期

TypeScript 并没有改变 JavaScript 类型系统的工作方式，但它确实使表达和强制类型期望变得更加容易，因此可以更容易地找到并解决类型不匹配。将一个名为 `index.ts` 的文件添加到 `primer` 文件夹中，其内容如 *列表 3.9* 所示。

列表 3.9：primer 文件夹中 index.ts 文件的内容

```js
function sum(first: number, second: number) {
    return first + second;
}
let result = sum(10, "10");
console.log(`Result value: ${result}, Result type: ${typeof result}`);
result = sum(10, 10);
console.log(`Result value: ${result}, Result type: ${typeof result}`); 
```

使用 `Control+C` 停止执行 JavaScript 代码的 `npm` 命令，并在 `primer` 文件夹中运行 *列表 3.10* 中显示的命令以启动运行 TypeScript 的命令。

列表 3.10：启动 TypeScript 工具

```js
npm run use_ts 
```

TypeScript 编译器处理 `index.ts` 文件的内容并生成以下错误：

```js
index.ts(5,22): error TS2345: Argument of type 'string' is not assignable to parameter of type 'number'. 
```

`sum` 函数的参数被添加了 *类型注解*，这告诉 TypeScript 编译器 `sum` 函数期望只接收 `number` 类型的值。编译器检查函数调用时使用的参数值，并报告错误，因为其中一个参数不是 `number` 类型。

**注意**

如果你正在使用 Visual Studio Code，你可能会在编辑器窗口中看到一个错误，显示消息 *Cannot redeclare block-scoped variable*。这发生在 TypeScript 和 JavaScript 文件都打开进行编辑时。如果你关闭 JavaScript 文件，错误将会消失。

### 使用类型联合

在注解中使用单个类型，如 `number`，会使 JavaScript 的行为更像其他编程语言，但会限制动态 JavaScript 类型系统的某些灵活性。JavaScript 代码可以编写来有意支持多种类型，如 *列表 3.11* 所示。

列表 3.11\. 在 primer 文件夹中的 index.ts 文件中支持多种类型

```js
**function sum(first: number, second: number) {**
 **if** **(typeof second == "string") {**
 **return first + Number.parseInt(second);**
 **} else {**
 **return first + second;**
 **}**
**}**
let result = sum(10, "10");
console.log(`Result value: ${result}, Result type: ${typeof result}`);
result = sum(10, 10);
console.log(`Result value: ${result}, Result type: ${typeof result}`); 
```

`sum` 函数会检查第二个参数是否为字符串值，如果是，则使用内置的 `Number.parseInt` 函数将其转换为 `number` 值。

这导致了函数的功能与应用于参数的类型注解之间的不匹配，因此编译器产生与 *列表 3.10* 相同的错误。这种不匹配可以使用 *类型联合* 来解决，如 *列表 3.12* 所示。

列表 3.12：在 primer 文件夹中的 index.ts 文件中使用类型联合

```js
**function sum(first: number, second: number | string) {**
    if (typeof second == "string") {
        return first + Number.parseInt(second);
    } else {
        return first + second;       
    }
}
let result = sum(10, "10");
console.log(`Result value: ${result}, Result type: ${typeof result}`);
result = sum(10, 10);
console.log(`Result value: ${result}, Result type: ${typeof result}`); 
```

使用竖线（`|` 字符）来组合类型，因此 `number | string` 告诉编译器第二个参数可以是 `number` 值或 `string` 值。TypeScript 检查 `sum` 函数的所有使用情况，并发现所有用作参数的类型都与类型注解匹配。代码执行时产生以下输出：

```js
Result value: 20, Result type: number
Result value: 20, Result type: number 
```

TypeScript 编译器非常聪明，它使用 `typeof` 关键字等 JavaScript 特性来了解类型的使用情况。*列表 3.13* 修改了 `sum` 函数的实现，使得 `string` 值不再与 `number` 值分开处理。

列表 3.13：在 primer 文件夹中的 index.ts 文件中更改函数

```js
function sum(first: number, second: number | string) {
   ** return first + second;**
}
let result = sum(10, "10");
console.log(`Result value: ${result}, Result type: ${typeof result}`);
result = sum(10, 10);
console.log(`Result value: ${result}, Result type: ${typeof result}`); 
```

TypeScript 编译器知道当 JavaScript 将加法运算符应用于两个 `number` 值或 `string` 和 `number` 时，JavaScript 会执行不同的操作，这意味着这个语句会产生一个歧义的结果：

```js
...
return first + second;       
... 
```

TypeScript 的设计目的是为了避免歧义，当编译代码时，编译器将生成以下错误：

```js
index.ts(2,12): error TS2365: Operator '+' cannot be applied to types 'number' and 'string | number'. 
```

TypeScript 的目的仅仅是突出潜在的问题，而不是强制执行任何特定的解决方案。*列表 3.13* 中的代码是合法的 JavaScript，但 TypeScript 编译器生成了错误，因为在 `sum` 函数内部使用参数值的方式与应用于参数的类型注解之间存在不匹配。

解决这个问题的方法之一是回到 *列表 3.12* 中的代码，如果 `sum` 函数想要在不进行类型强制转换的情况下处理 `number` 和 `string` 值，这是合理的。另一种方法是告诉编译器歧义是故意的，如 *列表 3.14* 所示，如果需要类型强制转换，这是合理的。

列表 3.14：在 primer 文件夹中的 index.ts 文件中解决歧义

```js
function sum(first: number, second: number | string) {
    **return first + (second as any);**
}
let result = sum(10, "10");
console.log(`Result value: ${result}, Result type: ${typeof result}`);
result = sum(10, 10);
console.log(`Result value: ${result}, Result type: ${typeof result}`); 
```

`as` 关键字告诉 TypeScript 编译器其对 `second` 值的了解不完整，并且应该将其视为我指定的类型。在这种情况下，我指定了 `any` 类型，这相当于告诉 TypeScript 预期存在歧义，并阻止它产生错误。此代码产生以下输出：

```js
Result value: 1010, Result type: string
Result value: 20, Result type: number 
```

应该谨慎使用 `as` 关键字，因为 TypeScript 编译器非常复杂，通常对数据类型的使用有很好的理解。同样，使用 `any` 类型可能是危险的，因为它本质上阻止 TypeScript 编译器检查类型。当你告诉 TypeScript 编译器你对代码了解得更多时，你需要确保你是正确的；否则，你将回到导致 TypeScript 最初引入的运行时错误问题。

# 理解基本的 TypeScript/JavaScript 功能

现在你已经理解了 TypeScript 和 JavaScript 之间的关系，是时候描述你将需要遵循本书示例的基本语言特性了。这不是 TypeScript 或 JavaScript 的全面指南，但应该足够让你开始。

## 定义变量和常量

使用 `let` 关键字定义变量，使用 `const` 关键字定义一个不会改变的常量值，如 *列表 3.15* 所示。

列表 3.15：在 primer 文件夹中的 index.ts 文件中定义变量和常量

```js
let condition = true;
let person = "Bob";
const age = 40; 
```

TypeScript 编译器从分配给每个变量或常量的值推断其类型，如果分配了不同类型的值，将生成错误。类型可以显式指定，如 *列表 3.16* 所示。

列表 3.16\. 在 primer 文件夹中的 index.ts 文件中指定类型

```js
**let condition: boolean = true;**
**let person: string = "Bob";**
**const age: number = 40****;** 
```

## 处理未分配和 null 值

在 JavaScript 中，已定义但未赋值的变量将被赋予特殊值 `undefined`，其类型为 `undefined`，如 *清单 3.17* 所示。

*清单 3.17*：在 primer 文件夹中的 index.ts 文件中定义无值的变量

```js
let condition: boolean = true;
let person: string = "Bob";
const age: number = 40;
let place;
console.log("Place value: " + place + " Type: " + typeof(place));
place = "London";
console.log("Place value: " + place + " Type: " + typeof(place)); 
```

此代码产生以下输出：

```js
Place value: undefined Type: undefined
Place value: London Type: string 
```

这种行为单独看起来可能有些不合逻辑，但它与 JavaScript 的其余部分保持一致，其中值有类型，任何值都可以赋给变量。JavaScript 还定义了一个单独的特殊值 `null`，可以赋给变量以表示无值或结果，如 *清单 3.18* 所示。

*清单 3.18*：在 primer 文件夹中的 index.ts 文件中赋值 null

```js
let condition: boolean = true;
let person: string = "Bob";
const age: number = 40;
let place;
console.log("Place value: " + place + " Type: " + typeof(place));
place = "London";
console.log("Place value: " + place + " Type: " + typeof(place));
**place = null;**
**console.log("Place value: " + place + " Type: " + typeof(place));** 
```

我通常可以提供对 JavaScript 功能工作方式的稳健辩护，但 `null` 值的奇怪之处使得输出显得有些说不通，这可以从此代码产生的输出中看到：

```js
Place value: undefined Type: undefined
Place value: London Type: string
Place value: null Type: object 
```

奇怪的是，特殊 `null` 值的类型是 `object`。这个 JavaScript 的怪癖可以追溯到 JavaScript 的第一个版本，并且由于大量代码依赖于它，所以一直没有得到解决。抛开这种不一致性，当 TypeScript 编译器处理代码时，它会确定不同类型的值被分配给 `place` 变量，并推断变量的类型为 `any`。

`any` 类型允许使用任何类型的值，这实际上禁用了 TypeScript 编译器的类型检查。可以使用类型联合来限制可以使用的值，同时仍然允许使用 `undefined` 和 `null`，如 *清单 3.19* 所示。

*清单 3.19*：在 primer 文件夹中的 index.ts 文件中使用类型联合

```js
let condition: boolean = true;
let person: string = "Bob";
const age: number = 40;
**let place: string | undefined** **| null;**
console.log("Place value: " + place + " Type: " + typeof(place));
place = "London";
console.log("Place value: " + place + " Type: " + typeof(place));
place = null;
console.log("Place value: " + place + " Type: " + typeof(place)); 
```

这种类型联合允许 `place` 变量被赋予 `string` 值或 `undefined` 或 `null`。注意，类型联合中指定了 `null` 的值。此清单产生的输出与 *清单 3.18* 相同。

## 使用 JavaScript 原始类型

JavaScript 定义了一组常用原始类型：`string`、`number`、`boolean`、`undefined` 和 `null`。这似乎是一个简短的列表，但 JavaScript 成功地将大量灵活性融入到这些类型中。（还有 `symbol` 和 `bigint` 类型，但这些是 JavaScript 中相对较新的添加，并且使用得不太广泛，本书中也没有使用。）

### 处理布尔值

`boolean` 类型有两个值：`true` 和 `false`。*清单 3.20* 展示了这两个值的使用，但此类型在条件语句（如 `if` 语句）中使用时最有用。此清单没有输出。

*清单 3.20*：在 primer 文件夹中的 index.ts 文件中定义 boolean 值

```js
let firstBool = true;
let secondBool = false; 
```

### 处理字符串

您可以使用双引号或单引号字符定义 `string` 值，如 *清单 3.21* 所示。

*清单 3.21*：在 primer 文件夹中的 index.ts 文件中定义字符串变量

```js
let firstString = "This is a string";
let secondString = 'And so is this'; 
```

您使用的引号必须匹配。例如，您不能以单引号开始字符串并以双引号结束。此清单没有输出。

### 使用模板字符串

常见的编程任务是结合静态内容和数据值以生成可以呈现给用户的字符串。JavaScript 支持 *模板字符串*，允许在静态内容中指定数据值，如 *清单 3.22* 所示。

*清单 3.22*：在 primer 文件夹中的 index.ts 文件中使用模板字符串

```js
let place: string | undefined | null;
console.log(`Place value: ${place} Type: ${typeof(place)}`); 
```

模板字符串以反引号（`` ` `` 字符）开始和结束，数据值由美元符号前缀的括号表示。例如，此字符串将 `place` 变量的值及其类型合并到模板字符串中：

```js
...
console.log(`Place value: ${place} Type: ${typeof(place)}`);
... 
```

此示例产生以下输出：

```js
Place value: undefined Type: undefined 
```

### 处理数字

`number` 类型用于表示整数和浮点数，如 *清单 3.23* 所示。

*清单 3.23*：在 primer 文件夹中的 index.ts 文件中定义数字值

```js
let daysInWeek = 7;
let pi = 3.14;
let hexValue = 0xFFFF; 
```

您无需指定使用哪种类型的数字。您只需表达所需的价值，JavaScript 将相应地处理。在清单中，我定义了一个整数值，定义了一个浮点数值，并在值前加上 `0x` 以表示十六进制值。*清单 3.23* 不会产生任何输出。

### 处理 null 和 undefined 值

`null` 和 `undefined` 值没有特性，例如属性或方法，但 JavaScript 采取的异常方法意味着您只能将这些值分配给类型为包含 `null` 或 `undefined` 的联合类型的变量，如 *清单 3.24* 所示。

*清单 3.24*：在 primer 文件夹中的 index.ts 文件中分配 null 和 undefined 值

```js
let person1 = "Alice";
let person2: string | undefined = "Bob"; 
```

TypeScript 编译器会将 `person1` 变量的类型推断为 `string`，因为分配给它的值的类型是 `string`。此变量不能分配 `null` 或 `undefined` 值。

`person2` 变量使用类型注解定义，指定 `string` 或 `undefined` 值。此变量可以分配 `undefined` 但不能分配 `null`，因为 `null` 不是类型联合的一部分。*清单 3.24* 不会产生任何输出。

## 使用 JavaScript 运算符

JavaScript 定义了一个相当标准的运算符集，其中最有用的将在以下章节中描述。

### 使用条件语句

许多 JavaScript 运算符都与条件语句一起使用。在这本书中，我倾向于使用 `if/else`，但 JavaScript 也支持 `switch` 语句，*清单 3.25* 展示了两种语句的使用，如果您几乎与任何编程语言都合作过，这将很熟悉。

*清单 3.25*：在 primer 文件夹中的 index.ts 文件中使用 if/else 和 switch 条件语句

```js
let firstName = "Adam";
if (firstName == "Adam") {
    console.log("firstName is Adam");
} else if (firstName == "Jacqui") {
    console.log("firstName is Jacqui");
} else {
    console.log("firstName is neither Adam or Jacqui");
}
switch (firstName) {
    case "Adam":
        console.log("firstName is Adam");
        break;
    case "Jacqui":
        console.log("firstName is Jacqui");
        break;
    default:
        console.log("firstName is neither Adam or Jacqui");
        break;
} 
```

清单的结果如下：

```js
firstName is Adam
firstName is Adam 
```

### 等式运算符与身份运算符

在 JavaScript 中，等价运算符（`==`）将尝试将操作数转换为相同的类型以评估相等性。这可以是一个有用的功能，但它被广泛误解，并且经常导致意外的结果。*清单 3.26*显示了等价运算符的作用。

清单 3.26：在入门文件夹中的 index.ts 文件中使用等价运算符

```js
let firstVal: any = 5;
let secondVal: any = "5";
if (firstVal == secondVal) {
    console.log("They are the same");
} else {
    console.log("They are NOT the same");
} 
```

这段代码的输出如下：

```js
They are the same 
```

JavaScript 会将两个操作数转换为相同的类型并比较它们。本质上，等价运算符测试值是否相同，而不管它们的类型如何。

如果你想要确保值和类型都相同，那么你需要使用身份运算符（`===`，三个等号，而不是等价运算符的两个等号），如*清单 3.27*所示。

清单 3.27：在入门文件夹中的 index.ts 文件中使用身份运算符

```js
let firstVal: any = 5;
let secondVal: any = "5";
**if (firstVal === secondVal) {**
    console.log("They are the same");
} else {
    console.log("They are NOT the same");
} 
```

在这个例子中，身份运算符将认为两个变量是不同的。这个运算符不会强制类型转换。这段代码的结果如下：

```js
They are NOT the same 
```

为了演示 JavaScript 的工作原理，我不得不在声明`firstVal`和`secondVal`变量时使用`any`类型，因为 TypeScript 限制了等价运算符的使用，使其只能用于相同类型的两个值。*清单 3.28*移除了变量类型注释，并允许 TypeScript 从分配的值中推断类型。

清单 3.28：在入门文件夹中的 index.ts 文件中移除类型注释

```js
**let firstVal = 5;**
**let** **secondVal = "5";**
if (firstVal === secondVal) {
    console.log("They are the same");
} else {
    console.log("They are NOT the same");
} 
```

TypeScript 编译器检测到变量类型不同，并生成以下错误：

```js
index.ts(4,5): error TS2367: This comparison appears to be unintentional because the types 'number' and 'string' have no overlap 
```

**理解真值和假值**

类型转换的一个重要后果是 JavaScript 的*真值*。一个*真值*是在转换为布尔值时评估为`true`的值，而一个*假值*是在转换为布尔值时评估为`false`的值。除了`false`、`0`、`-0`、`""`（空字符串）、`null`、`undefined`和`NaN`之外，每个值都是真值。

这个功能通常用于检查一个变量是否已分配了值，你将在后面的章节中看到许多例子，就像这个表达式一样：

```js
`...`
`if (customer) {`
`...` 
```

这是一种查看值是否已分配值的有用方法，尤其是在查询数据库或处理从用户接收到的数据时。不要被这种表达式所诱惑：

```js
`...`
`if (customer == true) {`
`...` 
```

在这个表达式中，类型转换应用于`true`值，而不是分配给`customer`的任何值，这不太可能产生预期的结果。

### 使用 null 和 nullish 合并运算符

逻辑或运算符（`||`）在 JavaScript 中传统上被用作空合并运算符，允许使用回退值代替`null`或`undefined`值，如*清单 3.29*所示。

清单 3.29：在入门文件夹中的 index.ts 文件中使用 null 合并运算符

```js
let val1: string | undefined;
let val2: string | undefined = "London";
let coalesced1 = val1 || "fallback value";
let coalesced2 = val2 || "fallback value";
console.log(`Result 1: ${coalesced1}`);
console.log(`Result 2: ${coalesced2}`); 
```

`||` 操作符如果左侧操作数评估为真值则返回左侧操作数，否则返回右侧操作数。当操作符应用于 `val1` 时，返回右侧操作数，因为没有为变量赋值，意味着它是 `undefined`。当操作符应用于 `val2` 时，返回左侧操作数，因为变量已被赋值为字符串 `London`，这评估为真值。此代码产生以下输出：

```js
Result 1: fallback value
Result 2: London 
```

以这种方式使用 `||` 操作符的问题在于，真值和假值可能会产生意外的结果，如 *列表 3.30* 所示。

列表 3.30：在 primer 文件夹中的 index.ts 文件中意外的空合并运算结果

```js
let val1: string | undefined;
let val2: string | undefined = "London";
**let val3: number |** **undefined = 0;**
let coalesced1 = val1 || "fallback value";
let coalesced2 = val2 || "fallback value";
**let coalesced3 = val3 || 100;**
console.log(`Result 1: ${coalesced1}`);
console.log(`Result 2: ${coalesced2}`);
**console.log(`Result 3: ${coalesced3}****`);** 
```

新的合并操作返回回退值，即使 `val3` 变量既不是 `null` 也不是 `undefined`，因为 `0` 被评估为假值。代码产生以下结果：

```js
Result 1: fallback value
Result 2: London
Result 3: 100 
```

空合并操作符（`??`）通过仅在左侧操作数为 `null` 或 `undefined` 时返回右侧操作数来解决此问题，如 *列表 3.31* 所示。

列表 3.31：在 primer 文件夹中的 index.ts 文件中使用空合并操作符

```js
let val1: string | undefined;
let val2: string | undefined = "London";
let val3: number | undefined = 0;
**let coalesced1 = val1 ?? "fallback value";**
**let coalesced2 = val2 ?? "fallback value";**
**let coalesced3 = val3 ?? 100;**
console.log(`Result 1: ${coalesced1}`);
console.log(`Result 2: ${coalesced2}`);
console.log(`Result 3: ${coalesced3}`); 
```

空操作符不考虑真值和假值结果，只查找 `null` 和 `undefined` 值。此代码产生以下输出：

```js
Result 1: fallback value
Result 2: London
Result 3: 0 
```

### 使用可选链操作符

如前所述，TypeScript 不会允许将 `null` 或 `undefined` 赋值给变量，除非它们已经被定义为合适的类型联合。此外，TypeScript 只允许使用联合中所有类型定义的方法和属性。

这种功能组合意味着在使用联合中任何其他类型提供的功能之前，您必须保护 `null` 或 `undefined` 值，如 *列表 3.32* 所示。

列表 3.32：在 primer 文件夹中的 index.ts 文件中防止 `null` 或 `undefined` 值

```js
let count: number | undefined | null = 100;
if (count != null && count != undefined) {
    let result1: string = count.toFixed(2);
    console.log(`Result 1: ${result1}`);
} 
```

要调用 `toFixed` 方法，我必须确保 `count` 变量没有被赋值为 `null` 或 `undefined`。TypeScript 编译器理解 `if` 语句中的表达式含义，并且知道排除 `null` 和 `undefined` 值意味着分配给 `count` 的值必须是 `number` 类型，这意味着可以安全地使用 `toFixed` 方法。此代码产生以下输出：

```js
Result 1: 100.00 
```

可选链操作符（`?` 字符）简化了保护过程，如 *列表 3.33* 所示。

列表 3.33：在 primer 文件夹中的 index.ts 文件中使用可选链操作符

```js
let count: number | undefined | null = 100;
if (count != null && count != undefined) {
    let result1: string = count.toFixed(2);
    console.log(`Result 1: ${result1}`);
}
**let result2: string | undefined = count?.toFixed(****2);**
**console.log(`Result 2: ${result2}`);** 
```

操作符应用于变量和方法调用之间，如果值是 `null` 或 `undefined`，则返回 `undefined`，从而防止调用方法：

```js
...
let result2: string | undefined = count?.toFixed(2);
... 
```

如果值不是`null`或`undefined`，则方法调用将按正常进行。包含可选链操作符的表达式的结果是`undefined`和方法的返回值的类型联合。在这种情况下，联合将是`string | undefined`，因为`toFixed`方法返回`string`。*清单 3.33*中的代码会产生以下输出：

```js
Result 1: 100.00
Result 2: 100.00 
```

# 定义和使用函数

当 Node.js 处理 JavaScript 文件时，它会按照定义的顺序执行语句。与大多数语言类似，JavaScript 允许将语句组合成一个函数，该函数不会执行，直到执行调用该函数的语句，如*清单 3.34*所示。

*清单 3.34*：在 primer 文件夹中的 index.ts 文件中定义一个函数

```js
function writeValue(val: string | null) {
    console.log(`Value: ${val ?? "Fallback value"}`)
}
writeValue("London");
writeValue(null); 
```

函数使用`function`关键字定义，并赋予一个名称。如果一个函数定义了参数，TypeScript 要求类型注解，这些注解用于强制函数使用的统一性。*清单 3.34*中的函数名为`writeValue`，它定义了一个可以接受`string`或`null`值的参数。函数内部的语句只有在函数执行时才会执行。*清单 3.34*中的代码会产生以下输出：

```js
Value: London
Value: Fallback value 
```

## 定义可选函数参数

默认情况下，TypeScript 允许函数仅在参数数量与函数定义的参数数量相匹配时被调用。如果你习惯了其他主流语言，这可能会显得很直观，但在 JavaScript 中，无论定义了多少个参数，函数都可以用任意数量的参数来调用。`?` 字符用于表示可选参数，如*清单 3.35*所示。

*清单 3.35*：在 primer 文件夹中的 index.ts 文件中定义一个可选参数

```js
**function writeValue****(val?: string) {**
    console.log(`Value: ${val ?? "Fallback value"}`)
}
writeValue("London");
**writeValue();** 
```

`?` 操作符已应用于`val`参数，这意味着函数可以不带参数或带一个参数被调用。在函数内部，参数类型是`string | undefined`，因为如果函数不带参数被调用，其值将是`undefined`。

**注意**

不要将`val?: string`（这是一个可选参数）与`val: string | undefined`（这是`string`和`undefined`的类型联合）混淆。类型联合要求函数调用时必须带有一个参数，这个参数可能是`undefined`的值，而可选参数允许函数不带参数被调用。

*清单 3.35*中的代码会产生以下输出：

```js
Value: London
Value: Fallback value 
```

## 定义默认参数值

可以给参数定义一个默认值，当函数不带相应参数被调用时，将使用这个默认值。这是一种避免处理`undefined`值的有用方法，如*清单 3.36*所示。

*清单 3.36*：在 primer 文件夹中的 index.ts 文件中定义一个默认参数值

```js
**function writeValue(val: string = "default value") {**
 **console.log****(`Value: ${val}`)**
**}**
writeValue("London");
writeValue(); 
```

当函数在没有参数的情况下被调用时，将使用默认值。这意味着示例中参数的类型始终是 `string`，因此我不必检查 `undefined` 值。*列表 3.36* 中的代码产生以下输出：

```js
Value: London
Value: default value 
```

## 定义剩余参数

*剩余参数* 用于在函数调用时捕获任何额外的参数，如*列表 3.37* 所示。

列表 3.37：在 primer 文件夹中的 index.ts 文件中使用剩余参数

```js
function writeValue(val: string, ...extraInfo: string[]) {
    console.log(`Value: ${val}, Extras: ${extraInfo}`)
}
writeValue("London", "Raining", "Cold");
writeValue("Paris", "Sunny");
writeValue("New York"); 
```

剩余参数必须是函数定义中的最后一个参数，并且其名称前缀为省略号（三个点，`...`）。剩余参数是一个数组，任何额外的参数都将被分配到这个数组中。在列表中，该函数将每个额外的参数打印到控制台，产生以下结果：

```js
Value: London, Extras: Raining,Cold
Value: Paris, Extras: Sunny
Value: New York, Extras: 
```

## 定义返回结果的函数

你可以通过声明返回数据类型并在函数体中使用 `return` 关键字从函数中返回结果，如*列表 3.38* 所示。

列表 3.38：在 primer 文件夹中的 index.ts 文件中返回结果

```js
function composeString(val: string) : string {
    return `Composed string: ${val}`;
}
function writeValue(val?: string) {
    console.log(composeString(val ?? "Fallback value"));
}
writeValue("London");
writeValue(); 
```

新函数定义了一个参数，即 `string`，并返回一个结果，也是一个 `string`。结果类型使用参数后的类型注解定义：

```js
...
function composeString(val: string) **: string** {
... 
```

TypeScript 将检查 `return` 关键字的用法，以确保函数返回一个结果，并且结果类型符合预期。此代码产生以下输出：

```js
Composed string: London
Composed string: Fallback value 
```

## 将函数作为其他函数的参数使用

JavaScript 函数是值，这意味着你可以将一个函数作为另一个函数的参数使用，如*列表 3.39* 所示。

列表 3.39：在 primer 文件夹中的 index.ts 文件中将函数作为另一个函数的参数使用

```js
function getUKCapital() : string {
    return "London";
}
function writeCity(f: () => string)  {
    console.log(`City: ${f()}`)
}
writeCity(getUKCapital); 
```

`writeCity` 函数定义了一个名为 `f` 的参数，这是一个它调用来获取要插入到它写入的字符串中的值的函数。TypeScript 要求函数参数必须这样描述，以便声明其参数和结果类型：

```js
...
function writeCity(**f: () => string**)  {
... 
```

这是 *箭头语法*，也称为 *胖箭头语法* 或 *lambda 表达式语法*。箭头函数有三个部分：括号内的输入参数，然后是一个等号和一个大于号（“箭头”），最后是函数结果。参数函数没有定义任何参数，因此括号是空的。这意味着参数 `f` 的类型是一个不接受任何参数并返回 `string` 结果的函数。参数函数在模板字符串中被调用：

```js
...
console.log(`City: ${**f()**}`)
... 
```

只有具有指定参数和结果组合的函数才能用作 `writeCity` 的参数。`getUKCapital` 函数具有正确的特征：

```js
...
writeCity(**getUKCapital**);
... 
```

注意，这里只使用了函数的名称作为参数。如果你在函数名称后跟括号，`writeCity(getUKCapital())`，那么你是在告诉 JavaScript 调用 `getUKCapital` 函数并将结果传递给 `writeCity` 函数。TypeScript 将检测 `getUKCapital` 函数的结果与 `writeCity` 函数定义的参数类型不匹配，并在代码编译时产生错误。*列表 3.39*中的代码产生以下输出：

```js
City: London 
```

### 使用箭头语法定义函数

箭头语法也可以用来定义函数，这是一种在行内定义函数的有用方法，如*列表 3.40*所示。

列表 3.40：在 primer 文件夹中的 index.ts 文件中定义箭头函数

```js
function getUKCapital() : string {
    return "London";
}
function writeCity(f: () => string)  {
    console.log(`City: ${f()}`)
}
writeCity(getUKCapital);
**writeCity(****() => "Paris");** 
```

此行内函数不接受任何参数，并返回字面字符串值 `Paris`，定义了一个可以作为 `writeCity` 函数参数使用的函数。*列表 3.40*中的代码产生以下输出：

```js
City: London
City: Paris 
```

### 理解值闭包

函数可以使用称为 *闭包* 的功能访问在周围代码中定义的值，如*列表 3.41*所示。

列表 3.41：在 primer 文件夹中的 index.ts 文件中使用闭包

```js
function getUKCapital() : string {
    return "London";
}
function writeCity(f: () => string)  {
    console.log(`City: ${f()}`)
}
writeCity(getUKCapital);
writeCity(() => "Paris");
**let myCity = "Rome";**
**writeCity(() => myCity);** 
```

新的箭头函数返回名为 `myCity` 的变量的值，该变量在周围的代码中定义。这是一个强大的功能，意味着你不需要在函数上定义参数来传递数据值，但需要注意，因为在使用像 `counter` 或 `index` 这样的常用变量名时，很容易得到意外的结果，你可能没有意识到你正在重用周围代码中的变量名。此示例产生以下输出：

```js
City: London
City: Paris
City: Rome 
```

# 操作数组

JavaScript 数组在大多数其他编程语言中的工作方式类似于数组。*列表 3.42*演示了如何创建和填充一个数组。

列表 3.42：在 primer 文件夹中的 index.ts 文件中创建和填充数组

```js
let myArray = [];
myArray[0] = 100;
myArray[1] = "Adam";
myArray[2] = true; 
```

我使用字面语法创建了一个新的空数组，使用方括号，并将其分配给名为 `myArray` 的变量。在后续语句中，我将值分配到数组的各个索引位置。（此列表没有输出。）

在此示例中，有两点需要注意。首先，我在创建数组时不需要声明项目数量。JavaScript 数组会自动调整大小以容纳任何数量的项目。第二点是，我无需声明数组将持有的数据类型。任何 JavaScript 数组都可以持有任何类型的数据混合。在示例中，我已将三个项目分配给数组：`number`、`string` 和 `boolean`。TypeScript 编译器推断数组的类型为 `any[]`，表示可以持有所有类型值的数组。此示例可以用*列表 3.43*中显示的类型注解来编写。

列表 3.43：在 primer 文件夹中的 index.ts 文件中使用类型注解

```js
**let** **myArray: any[] = [];**
myArray[0] = 100;
myArray[1] = "Adam";
myArray[2] = true; 
```

数组可以被限制为具有特定类型的值，如*清单 3.44*所示。

清单 3.44：在 primer 文件夹中的 index.ts 文件中限制数组值类型

```js
**let** **myArray: (number | string | boolean)[] = [];**
myArray[0] = 100;
myArray[1] = "Adam";
myArray[2] = true; 
```

类型联合限制了数组，使其只能包含`number`、`string`和`boolean`值。请注意，我将类型联合放在括号中，因为联合`number | string | boolean[]`表示一个可以分配`number`、`string`或布尔值数组的值，这并不是预期的。

数组可以在单个语句中定义和填充，如*清单 3.45*所示。

清单 3.45：在 primer 文件夹中的 index.ts 文件中填充新的数组

```js
let myArray: (number | string | boolean)[] = [100, "Adam", true]; 
```

如果你省略了类型注解，TypeScript 将从用于填充数组的值中推断数组类型。你应该谨慎地使用此功能，因为对于打算包含多种类型的数组，它要求在创建数组时使用完整的类型范围。

## 读取和修改数组的内 容

你使用方括号（`[`和`]`）读取给定索引的值，将所需的索引放在括号之间，如*清单 3.46*所示。

清单 3.46：在 primer 文件夹中的 index.ts 文件中读取数组索引的数据

```js
let myArray: (number | string | boolean)[] = [100, "Adam", true];
**let val = myArray[0];**
**console.log(`Value: ${val}`);** 
```

TypeScript 编译器推断数组中值的类型，因此*清单 3.46*中`val`变量的类型是`number | string | boolean`。这段代码产生了以下输出：

```js
Value: 100 
```

你可以通过简单地分配一个新的值到索引来修改 JavaScript 数组中任何位置的数据，如*清单 3.47*所示。TypeScript 编译器将检查你分配的值的类型是否与数组元素类型匹配。

清单 3.47：在 primer 文件夹中的 index.ts 文件中修改数组的内容

```js
let myArray: (number | string | boolean)[] = [100, "Adam", true];
**myArray[0****] = "Tuesday";**
let val = myArray[0];
console.log(`Value: ${val}`); 
```

在这个例子中，我将一个`string`赋值给了数组的`0`位置，这个位置之前被一个`number`占据。这段代码产生了以下输出：

```js
Value: Tuesday 
```

## 枚举数组的内 容

你可以使用`for`循环或`forEach`方法枚举数组的内 容，`forEach`方法接收一个函数，该函数被调用来处理数组中的每个元素。*清单 3.48*展示了这两种方法。

清单 3.48：在 primer 文件夹中的 index.ts 文件中枚举数组的内 容

```js
let myArray: (number | string | boolean)[] = [100, "Adam", true];
**for (let i = 0; i < myArray.length; i++) {**
 **console.log("Index " + i + ": " + myArray[i]);**
**}**
**console****.log("---");**
**myArray.forEach((value, index) =>**
 **console.log("Index " + index + ": " + value));** 
```

JavaScript 的`for`循环与其他许多语言中的循环工作方式相同。你使用其`length`属性确定数组中有多少个元素。

传递给`forEach`方法的函数提供了两个参数：要处理的当前项的值和该项在数组中的位置。在这个列表中，我使用了箭头函数作为`forEach`方法的参数，这是箭头函数擅长的用法（你将在本书中看到其使用）。列表的输出如下：

```js
Index 0: 100
Index 1: Adam
Index 2: true
---
Index 0: 100
Index 1: Adam
Index 2: true 
```

## 使用展开运算符

扩展运算符用于展开数组，以便其内容可以用作函数参数或与其他数组组合。在 *列表 3.49* 中，我使用了扩展运算符来展开数组，以便其项可以组合到另一个数组中。

列表 3.49：在 primer 文件夹中的 index.ts 文件中使用扩展运算符

```js
let myArray: (number | string | boolean)[] = [100, "Adam", true];
**let otherArray = [...myArray, 200, "Bob", false];**
**// for (let i = 0; i < myArray.length; i++) {**
**//     console.log("Index " + i + ": " + myArray[i]);**
**// }**
**// console.log("---");**
**otherArray.forEach(****(value, index) =>**
 **console.log("Index " + index + ": " + value));** 
```

扩展运算符是一个省略号（三个点的序列），它会导致数组被展开：

```js
...
let otherArray = [**...myArray**, 200, "Bob", false];
... 
```

使用扩展运算符，我可以在定义 `otherArray` 时将 `myArray` 指定为一个项，结果是将第一个数组的所有内容展开并添加到第二个数组中作为项。此示例产生以下结果：

```js
Index 0: 100
Index 1: Adam
Index 2: true
Index 3: 200
Index 4: Bob
Index 5: false 
```

# 对象的处理

JavaScript 对象是一组属性集合，每个属性都有一个名称和值。创建对象的最简单方法是使用文字符号，如 *列表 3.50* 所示。

列表 3.50：在 primer 文件夹中的 index.ts 文件中创建对象

```js
let hat = {
    name: "Hat",
    price: 100
};
let boots = {
    name: "Boots",
    price: 100
}
console.log(`Name: ${hat.name}, Price: ${hat.price}`);
console.log(`Name: ${boots.name}, Price: ${boots.price}`); 
```

文字符号语法使用大括号来包含属性名称和值的列表。名称与它们的值之间用冒号分隔，与其他属性之间用逗号分隔。在 *列表 3.50* 中定义了两个对象，分别命名为 `hat` 和 `boots`。通过变量名可以访问对象定义的属性，如下所示：

```js
...
console.log(`Name: ${hat.name}, Price: ${hat.price}`);
... 
```

*列表 3.50* 中的代码产生以下输出：

```js
Name: Hat, Price: 100
Name: Boots, Price: 100 
```

## 理解文字对象类型

当 TypeScript 编译器遇到文字对象时，它会推断其类型，使用属性名称及其分配的值组合。这个组合可以用在类型注解中，允许描述对象的形状，例如，作为函数参数，如 *列表 3.51* 所示。

列表 3.51：在 primer 文件夹中的 index.ts 文件中描述对象类型

```js
let hat = {
    name: "Hat",
    price: 100
};
let boots = {
    name: "Boots",
    price: 100
}
**function printDetails(product : { name: string, price: number}) {**
 **console.log(`Name: ${product.name}, Price: ${product.price}****`);** 
**}**
**printDetails(hat);**
**printDetails(boots);** 
```

类型注解指定 `product` 参数可以接受定义了名为 `name` 的 `string` 属性和名为 `price` 的 `number` 属性的对象。此示例产生的输出与 *列表 3.50* 相同。

类型注解描述属性名称和类型的组合，仅为对象设定了一个最小阈值，可以定义额外的属性，并且仍然符合类型，如 *列表 3.52* 所示。

列表 3.52\. 在 primer 文件夹中的 index.ts 文件中添加属性

```js
let hat = {
    name: "Hat",
    price: 100
};
let boots = {
    name: "Boots",
    price: 100,
   ** category: "Snow Gear"**
}
function printDetails(product : { name: string, price: number}) {
    console.log(`Name: ${product.name}, Price: ${product.price}`);   
}
printDetails(hat);
printDetails(boots); 
```

列表添加了一个新属性到分配给 `boots` 变量的对象中，但由于该对象定义了类型注解中描述的属性，因此该对象仍然可以用作 `printDetails` 函数的参数。此示例产生的输出与 *列表 3.50* 相同。

### 在类型注解中定义可选属性

可以使用问号来表示可选属性，如 *列表 3.53* 所示，允许不定义该属性的对象仍然符合类型。

列表 3.53 在 primer 文件夹中的 index.ts 文件中定义可选属性

```js
let hat = {
    name: "Hat",
    price: 100
};
let boots = {
    name: "Boots",
    price: 100,
    category: "Snow Gear"
}
**function printDetails(product : { name: string, price: number,**
 **category?: string}) {**
 **if (product.category != undefined) {**
 **console.log(****`Name: ${product.name}, Price: ${product.price}, `**
 **+ `Category: ${product.category}`);** 
 **} else {**
 **console.log(`Name: ${product.name}, Price: ${product.price}****`);** 
 **}**
**}**
printDetails(hat);
printDetails(boots); 
```

类型注解添加了一个可选的 `category` 属性，该属性被标记为可选。这意味着该属性的 类型是 `string | undefined`，函数可以检查是否提供了 `category` 值。此代码生成了以下输出：

```js
Name: Hat, Price: 100
Boots, Price: 100, Category: Snow Gear 
```

## 定义类

类是用于创建对象的模板，提供了一种替代字面量语法的方案。类对 JavaScript 规范的支持是最近添加的，目的是使使用 JavaScript 与其他主流编程语言更加一致。*列表 3.54* 定义了一个类，并使用它来创建对象。

列表 3.54：在 primer 文件夹中的 index.ts 文件中定义一个类

```js
**class** **Product {**
 **constructor(name: string, price: number, category?: string) {**
 **this.name = name;**
 **this.price = price;**
 **this.category** **= category;**
 **}**
 **name: string**
 **price: number**
 **category?: string**
**}**
**let hat = new Product("Hat", 100);**
**let boots = new Product****("Boots", 100, "Snow Gear");**
function printDetails(product : { name: string, price: number,
        category?: string}) {
    if (product.category != undefined) {
        console.log(`Name: ${product.name}, Price: ${product.price}, `
            + `Category: ${product.category}`);   
    } else {
        console.log(`Name: ${product.name}, Price: ${product.price}`);   
    }
}
printDetails(hat);
printDetails(boots); 
```

如果你使用过其他主流语言，如 Java 或 C#，那么 JavaScript 类将很熟悉。`class` 关键字用于声明一个类，后跟类的名称，在这个例子中是 `Product`。

当使用类创建新对象时，会调用 `constructor` 函数，这提供了一个接收数据值和执行类所需的任何初始设置的机会。在示例中，构造函数定义了 `name`、`price` 和 `category` 参数，这些参数用于将值分配给具有相同名称的属性。

`new` 关键字用于从类创建对象，如下所示：

```js
...
let hat = **new** Product("Hat", 100);
... 
```

这个语句使用 `Product` 类作为模板创建了一个新对象。在这种情况下，`Product` 被用作一个函数，传递给它的参数将由类定义的 `constructor` 函数接收。这个表达式的结果是分配给名为 `hat` 的变量的新对象。

注意，从类创建的对象仍然可以用作 `printDetails` 函数的参数。引入类改变了对象创建的方式，但这些对象具有相同的属性名称和类型组合，并且仍然符合函数参数的类型注解。*列表 3.54* 中的代码生成了以下输出：

```js
Name: Hat, Price: 100
Name: Boots, Price: 100, Category: Snow Gear 
```

### 将方法添加到类中

我可以通过将 `printDetails` 函数定义的功能移动到由 `Product` 类定义的方法中来简化示例中的代码，如 *列表 3.55* 所示。

列表 3.55：在 primer 文件夹中的 index.ts 文件中定义一个方法

```js
class Product {
    constructor(name: string, price: number, category?: string) {
        this.name = name;
        this.price = price;
        this.category = category;
    }
    name: string
    price: number
    category?: string
    **printDetails() {**
 **if (this.category != undefined) {**
 **console.log(`Name: ${this****.name}, Price: ${this.price}, `**
 **+ `Category: ${this.category}`);** 
 **} else {**
 **console.log(`Name:** **${this.name}, Price: ${this.price}`);** 
 **}** 
 **}**
}
let hat = new Product("Hat", 100);
let boots = new Product("Boots", 100, "Snow Gear");
**// function printDetails(product : { name: string, price: number,**
**//         category?: string}) {**
**//     if (product.category != undefined) {**
**//         console.log(`Name: ${product.name}, Price: ${product.price}, `**
**//             + `Category: ${product.category}`);** 
**//     } else {**
**//         console.log(`Name: ${product.name}, Price: ${product.price}`);** 
**//     }**
**// }**
**hat.printDetails();**
**boots.printDetails();** 
```

方法通过对象调用，如下所示：

```js
...
hat.**printDetails****();**
... 
```

方法通过 `this` 关键字访问对象定义的属性：

```js
...
console.log(`Name: ${**this.name**}, Price: ${**this.price**}`);   
... 
```

这个示例生成了以下输出：

```js
Name: Hat, Price: 100
Name: Boots, Price: 100, Category: Snow Gear 
```

### 访问控制和简化构造函数

TypeScript 通过使用`public`、`private`和`protected`关键字提供对访问控制的支撑。`public`类允许对由类定义的属性和方法无限制访问，这意味着它们可以被应用程序的任何其他部分访问。`private`关键字限制了特性的访问，使得它们只能在被定义的类内部访问。`protected`关键字限制了访问，使得特性可以在类或其子类内部访问。

默认情况下，由类定义的特性可以通过应用程序的任何部分访问，就像已经应用了`public`关键字一样。在这本书中，您不会看到对方法和属性应用了访问控制关键字，因为在 Web 应用程序中访问控制不是必需的。但有一个我经常使用的相关特性，它允许通过将访问控制关键字应用于构造函数参数来简化类，如*清单 3.56*所示。

*清单 3.56*：简化 primer 文件夹中 index.ts 文件中的类

```js
class Product {
   ** constructor(public name: string, public price: number,**
 **public category?: string) {**
 **// this.name = name;**
 **// this.price = price;**
 **// this.category = category;**
 **}**
 **// name: string**
 **// price: number**
 **// category?: string**
    printDetails() {
        if (this.category != undefined) {
            console.log(`Name: ${this.name}, Price: ${this.price}, `
                + `Category: ${this.category}`);   
        } else {
            console.log(`Name: ${this.name}, Price: ${this.price}`);   
        }       
    }
}
let hat = new Product("Hat", 100);
let boots = new Product("Boots", 100, "Snow Gear");
hat.printDetails();
boots.printDetails(); 
```

在构造函数参数中添加一个访问控制关键字的效果是创建一个具有相同名称、类型和访问级别的属性。例如，将`public`关键字添加到`price`参数，将创建一个名为`price`的`public`属性，它可以分配`number`类型的值。通过构造函数接收的值用于初始化属性。这是一个有用的特性，消除了复制参数值以初始化属性的需要。*清单 3.56*中的代码产生与*清单 3.53*相同的输出，只是`name`、`price`和`category`属性的定义方式发生了变化。

### 使用类继承

类可以使用`extends`关键字从其他类继承行为，如*清单 3.57*所示。

*清单 3.57*：在 primer 文件夹中 index.ts 文件中使用类继承

```js
class Product {
    constructor(public name: string, public price: number,
        public category?: string) {
    }
    printDetails() {
        if (this.category != undefined) {
            console.log(`Name: ${this.name}, Price: ${this.price}, `
                + `Category: ${this.category}`);   
        } else {
            console.log(`Name: ${this.name}, Price: ${this.price}`);   
        }       
    }
}
**class** **DiscountProduct extends Product {**
 **constructor(name: string, price: number, private discount: number) {**
 **super(name, price - discount);**
 **}**
**}**
**let hat = new** **DiscountProduct("Hat", 100, 10);**
let boots = new Product("Boots", 100, "Snow Gear");
hat.printDetails();
boots.printDetails(); 
```

`extends`关键字用于声明将要继承的类，称为*超类*或*基类*。在清单中，`DiscountProduct`从`Product`继承。`super`关键字用于调用超类的构造函数和方法。`DiscountProduct`基于`Product`的功能添加了对价格折扣的支持，产生以下结果：

```js
Name: Hat, Price: 90
Name: Boots, Price: 100, Category: Snow Gear 
```

## 检查对象类型

当应用于一个对象时，`typeof`函数将返回`object`。为了确定一个对象是否是从一个类派生出来的，可以使用`instanceof`关键字，如*清单 3.58*所示。

*清单 3.58*：在 primer 文件夹中 index.ts 文件中检查对象类型

```js
class Product {
    constructor(public name: string, public price: number,
         public category?: string) {
    }
    printDetails() {
        if (this.category != undefined) {
            console.log(`Name: ${this.name}, Price: ${this.price}, ` +
                `Category: ${this.category}`);   
        } else {
            console.log(`Name: ${this.name}, Price: ${this.price}`);   
        }       
    }
}
class DiscountProduct extends Product {
    constructor(name: string, price: number, private discount: number) {
        super(name, price - discount);
    }
}
let hat = new DiscountProduct("Hat", 100, 10);
let boots = new Product("Boots", 100, "Snow Gear");
**// hat.printDetails();**
**// boots.printDetails();**
**console.log(`Hat is a Product? ${hat instanceof Product}`);**
**console.****log(`Hat is a DiscountProduct? ${hat instanceof DiscountProduct}`);**
**console.log(`Boots is a Product? ${boots instanceof Product}`);**
**console.****log("Boots is a DiscountProduct? "**
 **+ (boots instanceof DiscountProduct));** 
```

`instanceof`关键字与一个对象值和一个类一起使用，如果对象是由该类或其超类创建的，表达式将返回`true`。*清单 3.58*中的代码产生以下输出：

```js
Hat is a Product? True
Hat is a DiscountProduct? True
Boots is a Product? True
Boots is a DiscountProduct? false 
```

# 使用 JavaScript 模块

JavaScript 模块用于将应用程序拆分为单独的文件。在运行时，模块之间的依赖关系被解析，包含模块的文件被加载，它们包含的代码被执行。

## 创建和使用模块

你添加到项目中的每个 TypeScript 或 JavaScript 文件都被视为一个模块。为了演示，我在 `primer` 文件夹中创建了一个名为 `modules` 的文件夹，并向其中添加了一个名为 `name.ts` 的文件，并在 *列表 3.59* 中展示了相应的代码。

列表 3.59：模块文件夹中 name.ts 文件的内容

```js
export class Name {
    constructor(public first: string, public second: string) {}
    get nameMessage() {
        return `Hello ${this.first} ${this.second}`;
    }
} 
```

在 JavaScript 或 TypeScript 文件中定义的类、函数和变量默认情况下只能在该文件内部访问。使用 `export` 关键字可以使功能在文件外部可用，以便其他应用程序部分可以使用它们。在 *列表 3.59* 中，我应用了 `export` 关键字到 `Name` 类，这意味着它可以在模块外部使用。

接下来，向 `modules` 文件夹中添加一个名为 `weather.ts` 的文件，其内容如 *列表 3.60* 所示。此模块导出一个名为 `WeatherLocation` 的类。

列表 3.60：模块文件夹中 weather.ts 文件的内容

```js
export class WeatherLocation {
    constructor(public weather: string, public city: string) {}
    get weatherMessage() {
        return `It is ${this.weather} in ${this.city}`;
    }
} 
```

使用 `import` 关键字来声明对模块提供的功能的依赖。在 *列表 3.61* 中，我在 `index.ts` 文件中使用了 `Name` 和 `WeatherLocation` 类，这意味着我必须使用 `import` 关键字来声明对这些类及其来源模块的依赖。

列表 3.61：在 primer 文件夹中的 index.ts 文件中导入特定的类型

```js
import { Name } from "./modules/name";
import { WeatherLocation } from "./modules/weather";
let name = new Name("Adam", "Freeman");
let loc = new WeatherLocation("raining", "London");
console.log(name.nameMessage);
console.log(loc.weatherMessage); 
```

这就是我在这本书的大部分示例中使用 `import` 关键字的方式。关键字后面跟着大括号，其中包含当前文件中代码所依赖的特性的逗号分隔列表，然后是 `from` 关键字，最后是模块名称。在这种情况下，我从 `modules` 文件夹中的模块中导入了 `Name` 和 `WeatherLocation` 类。请注意，在指定模块时，不包括文件扩展名。

当 `index.ts` 文件被编译时，TypeScript 编译器检测到对 `name.ts` 和 `weather.ts` 文件中代码的依赖，因此创建了模块的纯 JavaScript 版本。在执行过程中，Node.js 检测到 `index.js` 文件中的依赖，并使用编译器创建的 `name.js` 和 `weather.js` 文件来解析这些依赖，产生以下输出：

```js
Hello Adam Freeman
It is raining in London 
```

## 合并模块内容

在后面的示例中，特别是在 *第三部分* 中的 SportsStore 应用程序中，我将模块文件夹的内容合并起来，以便所有重要功能都可以在单个语句中导入，尽管它们定义在不同的代码文件中。要查看这是如何工作的，请向 `modules` 文件夹中添加一个名为 `index.ts` 的文件，其内容如 *列表 3.62* 所示。

列表 3.62：模块文件夹中 index.ts 文件的内容

```js
export { Name } from "./name";
export { WeatherLocation } from "./weather"; 
```

`index.ts` 文件包含每个代码文件中定义的特性的 `export` 语句。这允许通过指定包含文件夹的名称来导入这些特性，而不需要指定单个文件，如 *列表 3.63* 所示。

列表 3.63：在 primer 目录中的 `index.ts` 文件中导入模块文件夹

```js
**import { Name, WeatherLocation } from "./modules";**
let name = new Name("Adam", "Freeman");
let loc = new WeatherLocation("raining", "London");
console.log(name.nameMessage);
console.log(loc.weatherMessage); 
```

此列表产生的输出与 *列表 3.61* 相同。

**理解模块解析**

你将在本书中 `import` 语句中看到两种指定模块的方式。第一种是相对模块，其中模块的名称前缀为 `./`，如 *列表 3.60* 中的此示例所示：

```js
`...`
`import { Name, WeatherLocation } from "./modules";`
`...` 
```

此语句指定了一个相对于包含 `import` 语句的文件所在的模块。在这种情况下，由于没有指定文件名，所以将加载 `modules` 目录中的 `index.ts` 文件。另一种导入类型是非相对的。以下是在后续章节中你会看到的非相对 `import` 的示例：

```js
`...`
`import { Express } from "express";`
`...` 
```

此 `import` 语句中的模块不以 `./` 开头，依赖关系通过在 `node_modules` 文件夹中查找包来解决。在这种情况下，依赖关系依赖于 `express` 包提供的功能，该包在 *第五章* 中介绍。

# 摘要

在本章中，我描述了基本的 TypeScript 和 JavaScript 功能，为后续章节提供基础。

+   JavaScript 是一种动态类型和弱类型语言，这在现代编程语言中是不常见的组合。

+   任何类型的值都可以分配给变量、常量和函数参数。

+   JavaScript 会将值强制转换为其他类型以执行比较和其他操作。

+   TypeScript 是 JavaScript 的超集，允许开发者在编写代码时清晰地表达他们对数据类型的假设。

+   TypeScript 并不改变 JavaScript 的类型系统，TypeScript 文件被编译成纯 JavaScript。

在下一章中，我将描述一个对于理解 Node.js 及其在 Web 应用程序中的角色至关重要的基本概念：并发。
