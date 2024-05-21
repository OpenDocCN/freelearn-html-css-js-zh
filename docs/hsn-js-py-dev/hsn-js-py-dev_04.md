细枝末节的语法

当比较两种编程语言时，必然会有结构和语法上的差异。好消息是，Python 和 JavaScript 都是非常易读的语言，所以从 Python 切换到 JavaScript 和 Node.js 的上下文转换不应该太费力。

风格是一个很好的问题：制表符还是空格？分号还是不用？在任何编程语言中写作时出现的许多风格问题都已经在 Python 的 PEP-8 风格指南中得到了回答。虽然 JavaScript 没有官方的风格指南，但不用担心——外面并不是西部荒野。

在我们能够编写 JavaScript 之前，我们必须知道它是什么，才能够阅读和理解它。所有编程语言都有所不同，利用你的 Python 知识来学习一门新语言将需要一些思维的重新构建。例如，当我们想要声明变量时，JavaScript 是什么样子的？它是如何构建的，以便计算机能够理解？在我们进展时，我们需要注意什么？

本章是解锁 JavaScript 能做什么以及如何做的关键。

本章将涵盖以下主题：

+   风格的历史

+   语法规则

+   标点和可读性

+   房间里的大象-空白

+   现有标准-使用 linting 来拯救！

# 第五章：技术要求

要跟着本章的示例编码，你有几种选择：

+   直接在浏览器的 JavaScript 控制台中编码

+   在 Node 命令行中编码

+   使用网络编辑器，如[jsfiddle.net](https://jsfiddle.net)或[codepen.io](https://codepen.io)

使用网络编辑器可能更可取，因为你可以轻松保存你的进度。无论如何，你应该熟悉如何在浏览器中打开 JavaScript 控制台，因为我们将用它来调试输出。这通常在浏览器的“查看”菜单中；如果不是很明显，一些浏览器可能需要在“偏好设置”中打开开发者模式，所以请查阅你的浏览器文档以找到它。

你可以在 GitHub 上找到本章的代码文件，网址为[`github.com/PacktPublishing/Hands-on-JavaScript-for-Python-Developers/tree/master/chapter-3/Linting`](https://github.com/PacktPublishing/Hands-on-JavaScript-for-Python-Developers/tree/master/chapter-3/Linting)。

# 风格的历史

每种编程语言都有自己的风格，旨在简化每行代码的可读性和理解性。有些语言比其他语言更严格；JavaScript 在其原始形式中是较为宽松的语言之一。Brian W. Kernighan 和 P. J. Plauger 在 1974 年首次出版的《编程风格的要素》中有许多格言，这些格言不仅帮助塑造了编码标准，也塑造了编程语言本身。

你可能熟悉*Python*的 PEP-20 格言：

+   美好胜过丑陋。

+   明确比隐式更好。

+   简单比复杂更好。

+   复杂比复杂更好。

+   平面比嵌套更好。

+   稀疏比密集好。

+   可读性很重要。

+   特殊情况并不足以打破规则。

+   尽管实用性胜过纯洁性。

+   错误不应该悄悄地传递。

+   除非明确被消除。

+   面对模棱两可，拒绝猜测的诱惑。

+   应该有一种——最好只有一种——明显的方法来做到这一点。

+   尽管这种方式一开始可能不明显，除非你是荷兰人。

+   现在总比永远好。

+   尽管从来比*现在*好。

+   如果实现难以解释，那是个坏主意。

+   如果实现容易解释，那可能是个好主意。

+   命名空间是一个很棒的想法——我们应该做更多这样的事情！

开玩笑的特质抛开不谈，这些格言中的许多都是在 Python 开发之前写下的原则和经验的启发。Python 于 1991 年首次发布，从一开始就强调代码的可读性，并制定了一些严格的指导方针，从 PEP-8 到 PEP-20。

让我们举个例子，比如两个格言：

| **编程风格的要素，1974** | **Python 之禅，1999** |
| --- | --- |
| 写得清晰——不要太聪明。 | 明确比含蓄更好。 |

这里表达了类似的观点。我认为大多数软件工程师都会同意这样一种说法，即清晰、明确和可读是你在开发程序时应该追求的良好品质。

然而，有一个观点需要牢记，随着你在 JavaScript 学习中不断进步：由于 JavaScript 的语法*设计*比其他一些语言更宽松，你可能会发现不同的公司对 JavaScript 代码有自己的内部风格。这并不是 JavaScript 独有的现象——许多语言在公司中也有风格指南，以确保员工之间的代码一致性。这也有助于语言的整体生态系统具有一致的可读性。然而，这会导致不同代码库在风格上存在差异。

与任何语言一样，我们需要了解语法，以知道我们将如何编写 JavaScript。与 Python 一样，机器在执行工作之前期望得到格式正确的代码，这是你的工作。接下来是语法。

# 语法规则

就像任何其他编程语言一样，JavaScript 有语法规则必须遵循，以便计算机理解我们的代码想要告诉它的内容。这些规则相当简单明了，从大写和标点符号到使用特定结构和避免混淆含义的常用词，都可以提高代码的可读性。JavaScript 语法规则包括以下内容：

+   大写

+   保留关键字

+   变量语法

+   数据类型

+   逻辑结构

+   函数

+   标点符号

## 大小写很重要

与大多数编程语言一样，大小写有所不同。`myNode`和`mynode`变量将被解释为完全不同的变量。也就是说，计算机会完全区分`myNode`和`mynode`，因为它们的大小写不同。

## 保留关键字

JavaScript 中有许多保留的关键字不能用作变量名。以下是其中大部分的列表：

| `abstract` `arguments`

`await`

`boolean`

`break`

`byte`

`case`

`catch`

`char`

`class`

`const`

`continue`

`debugger`

`default`

`delete`

`do` | `double` `else`

`enum`

`eval`

`export`

`extends`

`false`

`final`

`finally`

`float`

`for`

`function`

`goto`

`if`

`implements`

`import` | `in` `instanceof`

`int`

`interface`

`let`

`long`

`native`

`new`

`null`

`package`

`private`

`protected`

`public`

`return`

`short`

`static` | `super` `switch`

`synchronized`

`this`

`throw`

`throws`

`transient`

`true`

`try`

`typeof`

`var`

`void`

`volatile`

`while`

`with`

`yield` |

这些关键字始终以小写形式存在，如果尝试将其中一个关键字用作变量名，程序将显示错误。

## 声明变量

在 JavaScript 中，最好在使用变量之前声明变量。这个声明可以在赋值时进行，也可以定义一个没有值的变量。

与其他一些语言不同，JavaScript 是*弱类型*的，因此不需要声明正在创建的变量的类型。按照惯例，JavaScript 中的变量以小写字母开头，采用驼峰命名法，而不是蛇形命名法。因此，`myAge`比`my_age`或`MyAge`更可取。变量不能以数字开头。

在 JavaScript 中有三个关键字用于声明变量：`const`、`let`和`var`。

### const

**const**，即**constant**，是一个在程序运行过程中不会改变值的变量。它们对于强制执行不希望更改的值很有用。在 ECMAScript 的第六版 ES2015（通常称为 ES6）之前，任何变量的值都可以被改变，因此常见的错误，比如使用赋值运算符（`=`）而不是比较运算符（`==`或`===`）：

```js
const firstName = "Jean-Luc"
const lastName = "Picard"
```

当然，皮卡德船长*可能*会改变他的名字，但这似乎不太可能。

有时，我们想将变量声明为硬常量，比如π或 API 密钥。这些用例通常是对命名标准的唯一例外，通常全部大写，有时有下划线：

```js
const PI = 3.14159
const API_KEY = 'cnview8773jass'
```

到目前为止，我们已经有了两种数据类型的示例：**字符串**和**数字**。JavaScript 没有*float*与*int*与*long*的概念；它们都是数字。如果你注意到了，我们也可以用单引号或双引号声明字符串。一些库和框架更喜欢其中一种，但对于标准的 JavaScript 来说，使用任何一种都可以。然而，最好保持一致。

### let

使用`let`声明变量时，我们明确声明我们期望或允许变量的值在程序运行过程中发生变化：

```js
let ship = "Stargazer"
ship = "Enterprise" // ship now equals "Enterprise"
```

皮卡德船长随时可以被转移到另一艘船上，所以我们希望我们的程序允许值的变化。

### var

JavaScript 中定义变量的最古老的方法是使用`var`关键字。使用`var`声明不会对变量的值施加任何限制；它可以被更改。

`var`的使用仍然受支持，但被认为是遗留的，并在 ES6 中被弃用。然而，由于存在数十年的现有程序和示例，至少熟悉`var`是很重要的。

## 数据类型

尽管 JavaScript 是弱类型的，但了解可用的数据类型对我们很重要，因为我们需要了解它们以解决比较和重新赋值等问题。

以下是基本 Python 变量到基本 JavaScript 的粗略映射：

| **Python** | **JavaScript** |
| --- | --- |
| Number | Number |
| String | String |
| List | Array |
| Dictionary | Object |
| Set | Set |

这涵盖了你可能会使用的基本类型。让我们来看看其他更微妙的 JavaScript 数据类型。有些在 Python 中有对应的，有些没有：

| **Python** | **JavaScript 半等效** | **差异原因** |
| --- | --- | --- |
| `bool` | `boolean` | 虽然在实践中数据类型是相同的，但 Python 的`bool`数据类型继承自`int`。虽然在 JavaScript 中可以使用`1`和`0`表示`True`和`False`，但它们不会被识别为`boolean`类型。 |
| `None` | `null` | 从技术上讲，`None`本身就是一个对象，而`null`是一个假值。 |
|  | `undefined` | 在 JavaScript 中，一个没有用值声明的变量仍然有一个伪值：`undefined`的单例值。 |
|  | `object` | Python 和 JavaScript 都是面向对象的语言，但它们对对象的使用有些不同。JavaScript 中对象的基本用法是键值存储。对象不是原始类型，可以存储多种类型的数据。 |
|  | `symbol` | 符号是 ES6 中的一种新数据类型。虽然使用方法有微妙之处，但值得一提。它们用于为对象创建唯一标识符。 |

现在，我们需要更多地了解一些类型，包括如何比较它们和处理它们。

### typeof 和 equality

尽管变量类型是可变的，但了解变量在某一时刻是什么数据类型通常是有用的。`typeof`运算符帮助我们做到这一点：

```js
typeof(1) // returns "number"
typeof("hello") // returns "string"
```

请注意返回值是字符串。

在比较变量时，有两种相等运算符：宽松相等和严格相等。让我们看一些例子：

```js
let myAge = 38
const age = "38"
myAge == age
```

如果我们运行这个比较，将得到 `true` 的结果。然而，我们可以看到 `myAge` 是一个数字，而 `age` 是一个字符串。结果为 `true` 的原因是，当使用宽松相等运算符（双等号）时，JavaScript 使用*类型强制转换*来试图提供帮助。当比较不同类型的变量时，值会被宽松比较，因此虽然 `38` 和 `"38"` 是不同类型，但由于它们的值，比较的结果是真值。

正如你可以想象的那样，这可能会产生一些意想不到的行为。要求 JavaScript 在比较中包含类型，使用*严格相等*运算符：三个等号。

通过前面的例子，我们可以尝试 `myAge === age`，将得到 `false` 的结果，因为它们是不同的数据类型。通常认为最佳实践是使用严格相等来避免类型强制转换，除非您有特定需要使用宽松相等。

### 数组和对象

数组和对象不是原始类型，可以包含混合类型。以下是一些示例：

```js
const officers = ['Riker','Data','Worf']

const captain = {
  "name": "Jean-Luc Picard",
  "age": 62,
  "serialNumber": "SP 937-215",
  "command": "NCC 1701-D",
  "seniorStaff": officers
}
```

`officers` 是一个**数组**，我们可以通过方括号看到。关于数组的一个有趣的事实是，即使我们通常将它们声明为常量，数组中的值可以被更改。`.push()` 和 `.pop()` 是两个用于操作数组的有用方法：

```js
officers.push('Troi') // officers now equals ['Riker','Data','Worf', 'Troi']
```

请注意，数组中的值没有以任何方式排序；我们可以通过使用方括号表示法来获取 `Riker`。然而，如果我们尝试完全重新分配数组，当重新分配已声明的常量时，我们仍然会得到一个错误。数组可以容纳任何组合的数据类型。

我们将使用的一个非常方便的数组属性是 `.length`。由于它是一个属性，它不使用括号：

```js
officers.length // now equals 4
```

请注意，即使数组是从零开始索引的，`length` 属性却不是。数组中有四个元素，索引从 0 到 3。

我们将在本章中更详细地讨论方法和属性。

**对象**是 JavaScript 非常强大的基础组件。实际上，从技术上讲，JavaScript 中的几乎所有东西都是对象！我们可以通过点符号访问数组方法，因为数组从技术上讲是一种对象。但是，我们无法通过点符号访问数组的*值*。

如果我们看 `captain`，我们可以看到三种不同的数据类型：字符串、数字和数组。对象也可以有嵌套对象。作为键值存储的一部分，键应该是一个字符串。要访问一个值，我们使用点符号：

```js
captain.command // equals "NCC 1701-D"
```

我们可以使用点符号访问对象的部分，这类似于 Python 中的**dict**，但不完全相同！随着我们使用对象，细微差别将变得更加清晰，因为它们是 JavaScript 独特之处的基础。

## 条件语句

让我们看看在 Python 和 JavaScript 中以两种方式编写的 `if`/`else` 语句：

| **Python** | **JavaScript** |
| --- | --- |

|

```js
if a < b:
  min = a
else:
  min = b
```

|

```js
let min

if (a < b) {
  min = a
} else {
  min = b
}
```

|

|

```js
min = a if a < b else b
```

|

```js
let min = (a < b) ? a : b
```

|

在两列中，代码正在执行相同的操作：简单测试以查看 `a` 是否小于 `b`，然后将较小的值分配给 `min` 变量。第一行是完整的 `if`/`else` 语句，第二行使用三元结构。这些示例中有一些语法规则需要注意：

+   `min` 必须在使用之前声明，作为最佳实践。在严格模式下，这实际上会抛出错误。

+   我们的 `if` 子句被括号包围。

+   我们的 `if`/`else` 语句被大括号包围。

+   三元运算符中的关键字和操作符与 Python 中的显着不同（并且有点更加神秘）。

如果我们想要使用我们现在了解的 `typeof`，我们可以使用严格相等来更好地理解我们的变量：

```js
let myVar = 2

if (typeof(myVar) === "number") {
  myVar++; // myVar now equals 3
}
```

## 循环

JavaScript 中有四种主要类型的循环：`for`、`while`、`do`/`while` 和 `for..in`。（还有一些其他的循环结构方式，但这些是主要的。）它们的使用情况应该不会有太多意外。

### for 循环

使用迭代器执行指定次数的代码：

| **Python** | **JavaScript** |
| --- | --- |

|

```js
names = ["Alice","Bob","Carol"]
for x in names:
    print(x)
```

|

```js
const names = ["Alice","Bob","Carol"]

for (let i = 0; i < names.length; i++) {
  console.log(names[i])
}
```

|

现在，你可能会想，“如果 JavaScript 有`for..in`循环，为什么我们不使用它呢？”事实证明，Python 的`for/in`和 JavaScript 的`for..in`是*假朋友*：它们的名字看起来很像，但在使用上却非常不同。我们将很快讨论 JavaScript 的`for..in`循环。另外，注意我们需要在`for`循环中有三个子句：

![](img/a869732b-aee6-483a-80d5-041b081918d6.png)

图 3.1 - `for`循环的声明、条件和执行阶段

**声明**将定义一个迭代器或使用现有的可变变量。注意它应该是一个可变的数字！

我们的**条件**是我们要测试的内容。我们希望我们的循环在`i`小于`names.length`时运行。由于`name.length`是`3`，我们将运行我们的循环三次，或者直到`i`等于`4`，这不再满足我们的条件。

在我们的循环的每次迭代结束时，我们都会**执行**一些东西；通常是简单地递增我们的声明。现在注意一下我们的每个子句之间的分号…不像 JavaScript 的其他部分，这些*不*是可选的。在执行部分之后没有分号。

### while 循环

JavaScript 的`while`循环在使用上与 Python 的等效部分相同，只是语法上有一点不同：

| **Python** | **JavaScript** |
| --- | --- |

|

```js
i = 0
while i < 10:
    i += 1
```

|

```js
let i = 0
while (i < 10) {
   i++
}
```

|

### do/while 循环

正如名称所示，`do`/`while`循环在给定条件等于`true`时执行`do`代码。看一下 JavaScript：

```js
let i = 0

do {
  i++
} while (i < 10)
```

### for..in 循环

现在，我承诺要解释为什么 Python 的`for..in`与 JavaScript 的用法不同。不同之处在于 JavaScript 的`for..in`用于遍历对象中的键，而 Python 的`for..in`用作离散实体的循环。

让我们看一个例子：

```js
const officers = ['Riker','Data','Worf']

const captain = {
  "name": "Jean-Luc Picard",
  "age": 62,
  "serialNumber": "SP 937-215",
  "command": "NCC 1701-D",
  "seniorStaff": officers
}

let myString = ''

for (let x in captain) {
  myString += captain[x] + ' '
}
```

你认为`myString`现在等于多少？由于*JavaScript*中`for..in`的目的是遍历对象中的每个*键*，它是`Jean-Luc Picard 62 SP 937-215 NCC 1701-D Riker,Data,Worf`。

### for..of 循环

还有一个`for`循环：`for..of`，它与`for..in`不同。`for..of`循环遍历任何可迭代的值，比如数组、字符串、集合等。如果我们想遍历`officers`并记录每个名字，我们可以这样做：

```js
for (const officer of officers) {
  console.log(officer)
}
```

接下来，我们将讨论函数！

## 函数

啊，函数。我们喜欢它们，因为它们是模块化、**不重复自己**（**DRY**）程序的关键。JavaScript 和 Python 中的用例是一样的：代码块打算被调用多次，通常是带有不同参数。参数是函数将接受的变量，以便在可变数据集上执行其代码。参数是我们在调用函数时传递的东西。本质上它们是一样的，但根据它们在何时何地使用，有不同的词：它们是抽象，还是实际数据？让我们来看一个并排比较：

| **Python** | **JavaScript** |
| --- | --- |

|

```js
def add_one(x):
  x += 1
  return x

print(add_one(5))
// output is 6
```

|

```js
function addOne(val) {
  return ++val
}

console.log(addOne(5))
// output is 6
```

|

如果你还没有在浏览器中打开 JavaScript 控制台，现在应该打开来看看我们的输出`6`。

你可以看到结构相当相似，我们的参数被传递在括号中。如前所述，我们在 JavaScript 中更喜欢驼峰命名，并用大括号封装。使用参数`5`调用函数是一样的。为了简洁起见，我们可以在`return`执行之前使用`++`运算符在左边递增`val`。这样的快捷方式在 JavaScript 中很常见，但记住要明智地使用它们：“写得清晰—不要太聪明。”

然而，JavaScript 实际上有两种不同的声明函数的方式，还有 ES6 中引入的新语法。

### 函数声明

在前面的代码中，`addOne()`是*函数声明*的一个例子。它使用函数关键字来声明我们的功能。它的结构和看起来一样简单：

```js
function functionName(optionalParameters, separatedByCommas) {
  // do work
  // optionally return a value
}
```

### 函数表达式

这是`addOne()`的一个函数表达式的例子：

```js
const addOne = function(val) {
  return ++val
}
```

函数表达式应该在表达式中使用`const`，尽管使用`var`或`let`在语法上并不是错误的。

声明和表达式之间有什么区别？核心区别在于函数*声明*可以在程序中的任何地方使用，因为它被*hoisted*到顶部。由于 JavaScript 是自上而下解释的；这是对该范例的一个重大例外。因此，相反，使用*表达式*必须在表达式编写后发生。

### 箭头函数

ES6 引入了箭头语法来编写函数表达式：

```js
const addOne = (val) => { return ++val }
```

为了进一步复杂化问题，我们可以省略`val`周围的括号，因为只有一个参数：

```js
const addOne = val => { return ++val }
```

箭头函数和表达式之间的主要区别集中在*词法作用域*上。我们在*hoisting*中提到了作用域，并且我们将在下一章中更详细地讨论它。

## 注释

与任何语言一样，注释都很重要。在 JavaScript 中有两种声明注释的方法：

```js
const addOne = (val) => { return ++val } // I am an inline, single line comment

// I am a single comment

/*
 I am a multiline comment
*/
```

因此，我们可以用`//`开始注释，并写到行尾。我们可以用`//`进行全行注释，还可以用`/*`进行多行注释，以`*/`结束。此外，您可能会在 JSDoc 风格的注释中遇到注释，用于内联文档：

```js
/**
 * Returns the argument incremented by one
 * @example
 * // returns 6
 * addOne(5);
 * @returns {Number} Returns the value of the argument incremented by one.
 */    
```

有关 JSDoc 的更多信息包含在*进一步阅读*部分中。

## 方法和属性

到目前为止，我们已经看到`.push()`和`.pop()`作为数组实例的方法。在 JavaScript 中，**方法**只是一个固有于其数据类型的函数，它对变量的数据和属性进行操作。我之前提到过，几乎 JavaScript 中的一切都是对象，这并不夸张。从功能和语法到结构和用法，*对象*的原始数据类型与任何其他变量之间有许多相似之处。

我们对 JavaScript 语法的理解的下一部分是每个人都喜欢的：标点符号。虽然这可能看起来微不足道，但对于代码的解释，*无论是人还是计算机*，它都非常重要。

# 标点符号和可读性

与每种语言一样，JavaScript 对标点符号和空格如何影响可读性有一些约定。让我们看看一些想法：

+   **Python**：

```js
def add_one(x):
  x += 1
  return x
```

+   **Java**：

```js
int add_one(int val) {
  val += 1;
  return val;
}
```

+   **C++**：

```js
int add_one (int val)
{
  val += 1;
  return val;
}
```

+   **JavaScript**：

```js
function addOne(val) {
  return ++val
}
```

在 JavaScript 中，前面示例的约定如下：

+   函数名称和括号之间没有空格。

+   在左花括号之前有一个空格，它在同一行上。

+   右花括号单独一行，与`function`的开头语句对齐。

在这里，关于 JavaScript 和我们将在本书中使用的示例与您可能在现场和在线示例中遇到的示例之间还有一个现代观点：**分号**。

在现代 JavaScript 中，除了少数例外，语句末尾的分号是*可选的*。过去，始终用分号终止语句行是最佳实践，您会在现有代码中看到很多分号。这是一个从公司到公司、项目到项目和库到库的风格问题。有一些标准，我们将很快在 linting 中讨论，但在本书的目的上，我们将*不*使用分号来终止语句，除非在语法上需要（例如我们在循环中看到的）。

重要的是要注意，嵌套行应该缩进两个空格。两个空格和四个空格是一个风格问题，但在这本书中，我们将使用**两个空格**。帮助保持一致性的一种方法是配置您的代码编辑器将制表符转换为两个空格（或四个，根据需要）。这样，您只需按一下*Tab*，而不用担心按了空格键多少次。我不会详细阐述正确缩进的重要性，但请记住：您的代码遵循的风格和最佳实践越多，对于维护您的代码的人员以及您未来的自己来说，它就会更易读！

# 大象在房间里——空白

好的，好的，我们知道 Python 是基于空格的：制表符很重要！然而，在大多数情况下，JavaScript 真的*不在乎*空格。正如我们之前看到的，缩进和空格是*风格*而不是*语法*的问题。

所以问题是：当我第一次学习 Python 时，依赖空格的语言的想法令人憎恶。我想：“一个依赖于不正确的 IDE 设置就会崩溃的语言怎么能生存？”。撇开我的观点不谈，好消息是 Python 中的缩进与 JavaScript 中的缩进加大括号是平行的。

这里有一个例子：

| **Python** | **JavaScript** |
| --- | --- |

|

```js
def hello_world(x):
 if x > 3:
   x += 1
 else:
   x += 2
 return x
```

|

```js
function helloWorld(val) {
  if (val > 3) {
    return ++val
  } else {
    return val+2
  }
}
```

|

如果您注意到，我们 Python 函数中的`if`语句的缩进方式与此 JavaScript 示例的缩进方式相同，尽管没有大括号。所以耶！您对 Python 缩进规则的遵守实际上在 JavaScript 中*非常*有用！虽然不需要像 Python 那样包含空格，但它确实可以提高可读性。

归根结底，JavaScript 喜欢缩进就像 Python 一样，因为这样可以使代码更易读，尽管对于程序运行来说并不是必需的。

# 现有标准- linting 来拯救！

我们已经看过了 JavaScript 的约定和规范，但大多数规则都有一个“这可能会有所不同”的例外或“这在技术上并不是必需的”。那么，在一个可塑的、以意见为驱动的环境中，我们如何理解我们的代码呢？一个答案：*linting*。

简而言之，**linting**指的是通过预定义的规则运行代码的过程，以确保它不仅在语法上正确，而且还遵循适当的风格规则。这不仅限于 JavaScript 的实践；您可能也对 Python 代码进行了 linting。在现代 JavaScript 中，linting 已经被视为确保代码一致的最佳实践。社区中的两个主要风格指南是 AirBnB ([`github.com/airbnb/javascript`](https://github.com/airbnb/javascript))和 Google ([`google.github.io/styleguide/jsguide.html`](https://google.github.io/styleguide/jsguide.html))。您的代码编辑器可能支持使用 linter，但我们现在不会进入实际使用它们的细节，因为每个编辑器的设置都有所不同。以下是在 Atom 中的快速查看：

![](img/7dd21500-86c8-4ab5-84c7-58bf85c03d90.png)

图 3.2 - Atom 中的 Linting 错误

对于我们的目的，要知道标准是存在的，尽管它们可能会因风格指南而有所不同。您可以从[`github.com/PacktPublishing/Hands-on-JavaScript-for-Python-Developers/tree/master/chapter-3/Linting`](https://github.com/PacktPublishing/Hands-on-JavaScript-for-Python-Developers/tree/master/chapter-3/Linting)克隆一个演示 linting 的存储库。

有几种流行的 linting 工具可用，例如 ESLint 和 Prettier。您选择的工具可以根据您选择的风格指南进行自定义。

好了，这一章内容太多了！让我们结束吧。

# 总结

JavaScript 拥有丰富的语法和语法，经过多年的使用和完善。使用 ES6，我们有各种数据类型、声明函数的方法和代码规范。虽然编写 JavaScript 似乎是非常随意和快速的，但有最佳实践，而且语言的基本原理与其他语言一样强大。请记住，大小写是有影响的；不要将保留字用作变量名；使用`const`或`let`声明变量；尽管 JavaScript 是弱类型的，但数据类型很重要；条件、循环和函数都有助于构建代码的逻辑结构。

精通 JavaScript 的语法和语法对于理解如何使用这种强大的语言至关重要，所以花时间熟悉细节和复杂性。在向前迈进时，我们将假设您对 JavaScript 的风格流利，因为我们将涉及更困难的材料。

在下一章中，我们将亲自动手处理数据，并了解 JavaScript 如何处理和建模数据。

# 问题

尝试回答以下问题来测试你的知识：

1.  以下哪个不是有效的 JavaScript 变量声明？

1.  `var myVar = 'hello';`

1.  `const myVar = "hello"`

1.  `String myVar = "hello";`

1.  `let myVar = "hello"`

1.  以下哪个开始了函数声明？

1.  `function`

1.  `const`

1.  `func`

1.  `def`

1.  以下哪个不是基本循环类型？

1.  `for..in`

1.  `for`

1.  `while`

1.  `map`

1.  真或假 - JavaScript *需要*使用分号进行行分隔。

1.  真

1.  假

1.  真或假 - 空格在 JavaScript 中*从不*计数。

1.  真

1.  假

# 进一步阅读

+   B. W. Kernighan 和 P. J. Plauger，*编程风格的要素第二版*，McGraw Hill，纽约，1978 年。 ISBN 0-07-034207-5

+   PEP-8 - *Python 代码风格指南*：[`www.python.org/dev/peps/pep-0008/`](https://www.python.org/dev/peps/pep-0008/)

+   PEP-20 - *Python 之禅*：[`www.python.org/dev/peps/pep-0020/`](https://www.python.org/dev/peps/pep-0020/)

+   JSDoc：[`usejsdoc.org/`](http://usejsdoc.org/)
