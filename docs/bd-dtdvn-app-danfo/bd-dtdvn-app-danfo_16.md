# 第十四章：：附录：基本的 JavaScript 概念

欢迎来到本书的最后一章。我们将这一章放在书的最后，以免让有经验的 JavaScript 开发人员对基础概念感到无聊。这一章应该由那些在尝试使用`Danfo.js`之前想要复习 JavaScript 基础知识的开发人员阅读。

通过本章结束时，您将了解 JavaScript 的基本概念，这些概念对于使用该语言构建应用程序至关重要。您将学习有关数据类型、条件分支和循环结构以及 JavaScript 函数的知识。

具体来说，我们将涵盖以下概念：

+   JavaScript 的快速概述

+   了解 JavaScript 的基本原理

# 技术要求

我们将在本章的所有代码示例中使用开发者控制台。要在默认浏览器中运行任何代码片段，您需要打开开发者控制台。各种浏览器打开控制台的命令如下所示：

+   **Google Chrome**：要在 Google Chrome 中打开开发者控制台，请从浏览器窗口右上角打开**Chrome**菜单，然后选择**更多工具** | **开发者工具**。您还可以在 macOS 上使用*Option* + *Command* + *J*快捷键，或在 Windows/Linux 上使用*Shift* + *Ctrl* + *J*。

+   **Microsoft Edge**：在 Microsoft Edge 中，打开浏览器窗口右上角的**Edge**菜单，然后选择**F12 开发者工具**，或者您也可以直接按*F12*打开它。

+   **Mozilla Firefox**：在 Mozilla Firefox 中，点击浏览器右上角的**Firefox**菜单，然后选择**Web 开发者** | **浏览器控制台**。您还可以在 macOS 上使用*Shift* + *⌘* + *J*快捷键，或在 Windows/Linux 上使用*Shift* + *Ctrl* + *J*。

+   **Apple Safari**：在 Safari 中，您首先需要在浏览器设置中启用**开发者**菜单。要做到这一点，打开 Safari 的偏好设置（*Safari 菜单* | **偏好设置**），选择**高级**选项卡，然后启用**开发者**菜单。一旦该菜单被启用，您可以通过点击**开发** | **显示 JavaScript 控制台**来找到开发者控制台。您还可以使用*Option* + *⌘* + *C*快捷键。

一旦您的开发者控制台打开，根据您的浏览器，您将看到一个类似于这里显示的控制台：

![图 13.1 – Chrome 浏览器开发者控制台](img/B17076_13_01.jpg)

图 13.1 – Chrome 浏览器开发者控制台

打开控制台后，您就可以开始编写和测试 JavaScript 代码了。在下一节中，我们将快速介绍 JavaScript 语言的一些重要概念。

# JavaScript 的快速概述

根据*Stack Overflow 2020 开发者调查*（[`stackoverflow.blog/2020/05/27/2020-stack-overflow-developer-survey-results/`](https://stackoverflow.blog/2020/05/27/2020-stack-overflow-developer-survey-results/)），**JavaScript**（也称为**JS**）是世界上最常见的编程语言，大约有 70%的开发人员在某种任务中使用它。这一统计数据并不令人意外，因为在调查进行之前的许多年里，JavaScript 一直是最受欢迎的语言。为什么会这样有很多原因，我们将在这里列出其中一些：

+   它在最常见和最常用的平台上运行——浏览器。

+   许多有用的框架如 Node.js、React 和 Angular 都是围绕它构建的。

+   它是多才多艺的——即它可以用于前端和后端应用。例如，您可以使用 JavaScript 库如 React、Vue 和 Angular 来构建出色的**用户界面**（**UI**），而您可以使用服务器端包如 Node.jS 和 Deno 来构建高效的后端/服务器端应用。

+   它可以用于**物联网**（**IoT**）和跨平台移动应用。

JavaScript 最初被创建为仅限浏览器的语言，但很快发展成几乎可以在任何地方使用，从前端应用到具有 Node.js 的后端应用，再到物联网应用，以及最近的数据科学/**机器学习**（**ML**）领域。

注意

不要将 JavaScript 与 Java 编程语言混淆（[`en.wikipedia.org/wiki/Java_(programming_language)`](https://en.wikipedia.org/wiki/Java_(programming_language))）。尽管名称可能相似，但它们在用途、语法甚至语义上都有很大的不同。

JavaScript 是一种动态和事件驱动的语言，因此是许多关注的焦点，特别是对于来自其他语言的程序员。这导致了创建可以直接转译为 JavaScript 的语言。其中一些语言是**TypeScript**（由微软开发的具有严格数据类型的语言）、**Dart**（由谷歌开发的独立语言）、**CoffeeScript**等等。

# 理解 JavaScript 的基本原理

在本节中，作为记忆提醒，我们将快速介绍现代 JavaScript 的基本概念。如果您熟悉 JavaScript，那么您可以跳过本节。

正如我们反复提到的，JavaScript 可以用于前端和服务器端脚本编写，因此每个环境都有特定的语法或特性，例如，浏览器端 JavaScript 由于安全原因无法访问 Node.js 等文件系统。因此，在本节中，我们将介绍可以在两个环境/任何环境中工作的概念。

## 声明变量

`const`和`let`，尽管在旧脚本中，您会发现用于声明变量的`var`关键字。通常不鼓励使用`var`来声明变量，应该很少使用。在后面的部分中，我们将讨论为什么在现代 JavaScript 中鼓励使用`let`而不是`var`的一些原因。

以下语句使用`let`关键字声明变量：

```js
let name;
let price = 200;
let message, progress, id;

message = "This is a new message";
progress = 80;
id = "gh12345";
```

接下来的关键字`const`可以用于声明常量变量，即在应用程序运行时其引用不能被更改的变量。

以下代码片段显示了一些`const`声明的示例：

```js
const DB_NAME;
const LAST_NAME = "Williams";
const TEXT_COLOR = "#B00";
const TITLE_COLOR = "#F00";
```

重要提示

在声明常量时使用大写变量名是一种常见且被广泛鼓励的做法，不仅在 JavaScript 中，在许多编程语言中也是如此。

## 数据类型

JavaScript 支持八种基本数据类型。声明变量的数据类型会在 JavaScript 编译器在运行时自动推断，因为 JavaScript 是一种动态类型语言。支持的八种数据类型是**number**、**string**、**Boolean**、**object**、**BigInt**、**undefined**、**null**和**symbol**。

### 数字

`number`类型可以表示`整数`、`浮点数`、`infinity`、`-infinity`和`number`：

```js
let num1 = 10; //integer
let num2 = 20.234; //floating point number
console.log(num1 + num2); //outputs 30.234
```

重要提示

`infinity`和`-infinity`代表数学上的正无穷和负无穷（∞）。这些值大于任何数字。您可以在这里看到一个示例：

`console.log(1 / 0); // Infinity`

`NaN`子类型表示数字的数学运算中的错误，例如，尝试用数字除以字符串，如下所示：

`console.log("girl" / 2); // NaN`

### 字符串

JavaScript 中的字符串代表文本，必须用单引号（`' '`）、双引号（`" "`）或反引号（`' '`)引号括起来。您可以在以下代码片段中看到一些字符串的示例：

```js
let name = "John Doe";
let msg = 'I am on my way';
let address = '10 Slow ave, NY';
```

双引号或单引号的选择基于个人偏好，因为它们都具有相同的功能。另一方面，反引号比基本引号具有更多的功能。它们允许您轻松地将变量和表达式嵌入字符串中，使用`${...}`模板文字语法。例如，在以下代码片段中，我们展示了如何轻松地将`name`、`address`和`dog_counts`变量嵌入到一个新变量消息中：

```js
let name = "John Doe";
let address = '10 Slow ave, NY';
let dog_counts = 10;
let msg = '${name} lives in ${address} and has ${dog_counts} dogs';
console.log(msg)

//outputs
John Doe lives in 10 Slow ave, NY and has 10 dogs
```

提示

字符串数据类型有许多内置函数用于操作它。您可以在**Mozilla 开发者网络**（**MDN**）文档页面上找到许多这些函数（[`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String)）。

### 布尔

布尔数据类型是一种逻辑类型。它只有两个值：`true`和`false`。布尔值主要用于比较操作和存储*是*/*否*值。让我们看下面的例子：

```js
let pageClicked = false; // No, the page was not clicked
let pageOpened = true; // Yes, the page was opened

if (5 > 1) { //Evaluates to the Boolean value true
  console.log("Yes! It is.")
}
```

接下来，让我们谈谈对象类型。

### 对象

JavaScript 中的对象是一种非常特殊的类型。它可能是 JavaScript 中最重要的类型，几乎在语言的各个方面都有所应用。对象数据类型是 JavaScript 中唯一的非原始类型。它可以存储不同类型的数据的键控集合，包括它自己。

提示

编程语言中的原始类型是最基本的类型，通常只存储一种类型的值。另一方面，非原始类型可以存储多种类型的值，并且还可以扩展以执行其他功能。

有两种创建对象的方式，如下所述：

+   第一种最常见的方法是使用花括号和可选的键-值属性列表，如下所示：

`let page = {}`

+   还有一种不太常见的方法是使用对象构造函数语法，如下所示：

`let page = new Object()`

对象可以在同一步骤中创建和初始化，如下代码片段所示：

```js
let page = {
  title: 'Home page',
  Id: 23422,
  text: 'This is the first page in my website',
  rank: 32.09,
  "closed": false,
  owner: {
    name: 'John Doe',
    level: 'admin',
    lastLoggedIn: 'Aug 5th',
  },
}
```

请注意，对象页面可以存储多种类型的数据，甚至其他对象（作为所有者）。这表明对象可以嵌套以包含更多对象。

要访问对象值，可以使用方括号，如下所示：

```js
console.log(page["closed"])
//outputs false
```

或者，您可以使用点`page.rank`表示法，如下所示：

```js
console.log(page.rank)
```

这将产生以下输出：

```js
//outputs 32.09
```

在*第一章*中，*现代 JavaScript 概述*，您了解了使对象非常强大的一些重要属性。

## 条件分支和循环

条件分支和循环是任何编程语言的重要方面，JavaScript 也提供了它们。为了根据某些条件执行不同的操作，您可以使用`if`、`if..else`、条件/三元运算符和`switch`语法。除了条件分支，您可能希望根据特定次数执行重复操作。这就是循环发挥作用的地方。JavaScript 提供了诸如`while`、`do...while`、`for..of`、`for..in`和传统`for`循环等循环结构。我们将在下一节简要介绍这些语句。

### if 语句

`if`语句评估括号中的表达式、条件或代码片段，只有在为`true`时才执行该语句。这是一个单向条件，只有在为真时才运行。您可以在以下代码片段中看到这个例子：

```js
let age = 25
if (age >= 18){
  console.log("You're an adult!")  //will run since age is greater than 18
}
//outputs
You're an adult!
```

### if...else 语句

`if...else`语句在初始条件变为`false`时提供一个额外的块运行。以下是一个例子：

```js
let age = 15
if (age > 18){
  console.log("You're an adult!")
}else{
  console.log("You're still a teenager!")
}
//outputs
You're still a teenager!
```

### 条件/三元运算符

三元运算符是一种更短、更简洁的写法，用于根据两个条件分配变量。三元运算符的语法如下所示：

```js
let result = condition ? val1 : val2;
```

如果条件为真，则执行`val1`并将其分配给`result`，否则将分配`val2`。以下代码片段显示了使用此功能的示例：

```js
let age = 20
let isAdult = age > 18 ? true : false; //age is greater than 18, so true is returned
console.log(isAdult) 
//outputs
true 
```

### switch 语句

`switch`语句可以以更简洁和可读的方式替代多个`if...else`语句。语法如下所示：

```js
switch(value) {
  case 'option1':  // if (value === 'option1')
    ...
    [break]
  case 'option2':  // if (value === 'option2')
    ...
    [break]
 case 'option3':  // if (value === 'option3')
    ...
    [break]
  default:
    ...
    [break]
}
```

`switch`括号中的变量或表达式将与每个`case`语句进行比较，如果条件为真，则执行相应的代码。当其他条件都失败时，将执行默认语句。以下代码片段显示了使用`switch`语句的示例：

```js
let value = 4
switch (value) {
  case 1:
    console.log( 'You selected 1' );
    break;
  case 2:
    console.log( 'You selected 2' );
    break;
  case 4:
    console.log( 'You selected 4!' ); //gets executed
    break;
  default:
    console.log( "No such value" );
}

//outputs
"You selected 4!"
```

在每个 case 之后使用`break`语句很重要，以停止执行后续的 case。如果忽略`break`语句，那么后续的 case 也将被执行。

### while 循环

`while`循环将重复执行一段代码，只要条件为真。语法如下所示：

```js
while (condition) {
  // loop body
}
```

让我们来看一个使用`while`循环的例子：

```js
let year = 2015;
while (year <= 2020) {
  console.log( year );
  year++
}
//outputs
2015
2016
2017
2018
2019
2020
```

上面的代码将不断打印年份，只要它小于或等于`2020`。注意`year++`部分。这是为了在某个时刻打破`while`循环的重要部分。

### do...while 循环

`do...while`循环与`while`循环非常相似，只有一个小差别——在测试条件之前至少执行一次循环体。语法如下所示：

```js
do {
  // loop body
} while (condition);
```

让我们来看一个`do...while`循环的例子：

```js
let year = 2015;
do {
  console.log( year );
  year++;
} while (year <= 2020);
```

如果需要在进行条件检查之前至少执行一次代码，则`do...while`循环很重要。

重要提示

始终记得设置一个在某个时刻打破循环的条件，否则你的循环将永远执行——理论上，不过浏览器和服务器端会在一定时间后停止这样的循环。

### for 循环

`for`循环是 JavaScript 中最流行的循环结构。语法如下所示：

```js
for (initialization; condition; step) {
  // loop body
}
```

语法有三个重要部分，如下所示：

+   `let i = 0`）。

+   **条件部分**：在每次循环交互之前检查此代码。如果为假，则停止循环。

+   **步骤部分**：步骤很重要，因为它通常会增加一个将被测试的计数器或变量。

这里给出了使用`for`循环的例子：

```js
for (let i=0; i <=5; i++){
  console.log(i)
}
//outputs
0
1
2
3
4
5
```

注意

注意我们在`for`循环的步骤部分使用了后增量运算符（`i++`）？这是非常标准的，这只是`i = i + 1`的简写。

### for...of 和 for...in 循环

`for...of`和`for...in`循环是用于遍历可迭代对象的`for`循环的变体。**可迭代对象**是特殊的数据类型，如对象、数组、映射和字符串，具有可迭代属性——换句话说，可以循环遍历属性或值。

`for...of`循环主要用于遍历数组和字符串等对象。语法如下所示：

```js
for (let val of iterable) {
  //loop body
}
```

让我们来看一个使用`for...of`循环的例子：

```js
let animals = [ "bear", "lion", "giraffe", "jaguar" ];
// Print out each type of animal
for (let animal of animals) {
    console.log(animal);
}
//outputs
"bear"
"lion"
"giraffe"
"jaguar"
```

`for...in`循环用于遍历对象。语法与`for...of`循环非常相似，如下面的代码片段所示：

```js
for (let val in iterable) {
  //loop body
}
```

`for...in`循环应该只用于枚举对象，而不是用于可迭代对象如数组。

以下是使用`for...in`循环的例子：

```js
let user = {
  name: "Hannah Montana",
  age: 24,
  level: 2,
};
for (key in user) {
    console.log(user[key]);
}
//outputs
"Hannah Montana"
24
2
```

进一步阅读

`for...in`和`for...of`是用于遍历数据结构的循环结构。它们之间的主要区别是它们遍历的数据结构。`for...in`遍历对象的所有可枚举属性键，而`for...of`遍历可迭代对象的值。

## JavaScript 函数

在本节中，我们将学习 JavaScript 函数。每当我们需要向控制台打印文本时，就会使用`console.log`内置函数。这显示了函数可以有多重要，因为一旦定义，单个函数可以以任何方式和任何次数调用以执行相同的操作。

JavaScript 中的函数通常如下所示：

```js
function sayHi() {
  console.log('Hello World!')
}
sayHi()
//output
"Hello World!"
```

函数还可以在括号内接受一系列逗号分隔的参数，用于执行计算。例如，具有一些参数的函数通常看起来像这样：

```js
function sayHi(name, message) {
  console.log('This is a message from ${name}. The message read thus: ${message}')
}

sayHi("Jenny", "How are you doing?")
//output
"This is a message from Jenny. The message read thus: How are you doing?"
```

以下是关于函数的一些重要事项：

+   函数内声明的变量只在该函数内部有效，只能在函数内部访问。以下是一个例子：

```js
function sayHi() {
  let name = 'Jane'
  console.log(name)
}

sayHi()
//output
"Jane"
console.log(name) //throws error: ReferenceError: name is not defined 
```

+   函数可以访问全局变量。全局变量是在函数范围之外声明的变量，并且可以在每个代码块中使用。你可以在这里看到一个例子：

```js
let name = 'Jenny'
function sayHi() {
  console.log(name)
}
sayHi()
//output
"Jenny"
```

+   函数可以返回值。当函数执行某些计算并且需要在其他地方使用结果时，这一点非常重要。到目前为止，我们一直在使用的函数大多返回空/未定义，并且被称为无效函数。让我们看下面的一个函数返回值的例子：

```js
function add(num1, num2) {
  let sum = num1 + num2
  return sum
}
let result = add(25, 30)
console.log(result)

//output
55
```

提示

函数中的`return`语句一次只能返回一个值。为了返回多个值，可以将结果作为 JavaScript 对象返回。您可以在这里看到一个示例：

`function add(num1, num2) {`

`let sum = num1 + num2`

`return {sum: sum, funcName: "add"}}`

`let result = add(25, 30)`

`console.log(result)`

`//输出`

`Object { funcName: "add", sum: 55}`

### 回调函数

回调在 JavaScript 中非常重要。回调是作为参数传递给另一个函数的函数。根据外部函数中的一些条件，调用此参数来执行某些操作。让我们看下面的例子来更好地理解回调：

```js
function showValue(value) {
  console.log(value)
}
function err() {
  console.log("Wrong value specified!!")
}
```

在上面的代码示例中，我们创建了两个回调函数，`showValue`和`error`。您可以在下面的代码片段中看到它们的使用：

```js
function printValues(value, showValue, error) {
  if (value > 0){
    showValue(value)
  }else{
    error()
  }
}
```

`showValue`回调函数将传递给它的参数打印到控制台，而`error`回调函数则打印错误消息。可以像下面的代码片段中所示那样使用回调函数。

```js
printValues(20, showValue, err);
//outputs 20

printValues(0, showValue, err);
//outputs "Wrong value specified!!"
```

`printValues`函数首先测试值是否大于`0`，然后调用`showValue`或`error`回调函数。

回调不仅接受命名函数。它们还可以接受匿名函数；通过这种方式，可以创建*嵌套回调*。

假设我们有一个`doSomething`函数执行特定任务，并且我们希望在任务完成之前执行不同的操作。这意味着我们可以将回调函数传递给另一个回调函数，从而创建嵌套回调，如下面的代码块所示：

```js
doSomething(((((()=>{
//dosomething A
         })=>{
    //dosomething B
       })=>{
   //dosomething C
     })=>{
  //dosomething D
   })=>{
 //dosomething E
})
```

上面的代码是我们所谓的*回调地狱*。这种方法有很多问题，变得难以管理。在*第一章*中，我们介绍了一种现代和更有效的方法来使用**应用程序编程接口**（**API**）来处理回调。这样做更清晰，可以消除与回调地狱相关的许多问题。

重要提示

JavaScript 默认是异步的，因此长时间执行的函数会被排队，可能在你需要它们之前永远不会被执行。回调主要用于继续代码执行并确保返回正确的结果。

# 总结

在本章中，我们介绍了 JavaScript 编程语言的基本方面。我们从一些基本背景开始，解释了为什么 JavaScript 是当今世界上最流行的语言，以及 JavaScript 的各种用途。接下来，我们讨论了语言的基本概念，讨论了声明变量以及 JavaScript 中可用的八种数据类型。之后，我们讨论了 JavaScript 中的分支和循环结构，并展示了一些使用它们的示例，最后，我们简要讨论了 JavaScript 中的函数和类。
