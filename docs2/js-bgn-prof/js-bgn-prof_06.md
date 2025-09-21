# 6

# 函数

你已经看到了很多 JavaScript，现在你准备好学习函数了。很快你就会发现你已经在使用函数了，但现在是你学习如何开始编写自己的函数的时候了。函数是一个很好的构建块，它将减少你应用中所需的代码量。你需要函数时就可以调用它，你可以将其编写为一种带有变量的模板。所以，根据你如何编写它，你可以在许多情况下重用它。

它确实要求你以不同的方式思考代码的结构，这可能会很困难，尤其是在开始的时候。一旦你习惯了这种思维方式，函数将真正帮助你编写结构良好、可重用和易于维护的代码。让我们深入这个新的抽象层！

在此过程中，我们将涵盖以下主题：

+   基本函数

+   函数参数

+   返回

+   函数中的变量作用域

+   递归函数

+   嵌套函数

+   匿名函数

+   函数回调

    注意：练习、项目和自我检查测验的答案可以在*附录*中找到。

# 基本函数

我们已经调用函数有一段时间了。还记得`prompt()`、`console.log()`、`push()`和`sort()`数组函数吗？这些都是函数。函数是一组语句、变量声明、循环等捆绑在一起的内容。调用函数意味着整个语句组将被执行。

首先，我们将看看如何调用函数，然后我们将看看如何编写我们自己的函数。

## 调用函数

我们可以通过末尾的括号来识别函数。我们可以这样调用函数：

```js
nameOfTheFunction();
functionThatTakesInput("the input", 5, true); 
```

这是在没有参数的情况下调用一个名为`nameOfTheFunction`的函数，以及一个名为`functionThatTakesInput`的函数，它需要三个必需的参数。让我们看看当我们开始编写函数时，函数可以看起来像什么。

## 编写函数

使用`function`关键字可以编写函数。以下是编写函数的模板语法：

```js
function nameOfTheFunction() {
    //content of the function
} 
```

上面的函数可以这样调用：

```js
nameOfTheFunction(); 
```

让我们编写一个函数，询问你的名字，然后问候你：

```js
function sayHello() {
  let you = prompt("What's your name? ");
  console.log("Hello", you + "!");
} 
```

我们在问号后添加一个空格，以确保用户在问号后一个空格开始输入答案，而不是直接在其后。我们这样调用这个函数：

```js
sayHello(); 
```

它将提示：

```js
What's your name? > 
```

让我们继续输入我们的名字。输出将是：

```js
Hello Maaike! 
```

花点时间考虑函数和变量之间的关系。正如你所看到的，函数可以包含变量，这些变量决定了它们的操作方式。相反的情况也是正确的：变量可以包含函数。你还在吗？这里你可以看到一个包含函数的变量（`varContainingFunction`）和一个函数内部的变量（`varInFunction`）的例子：

```js
let varContainingFunction = function() {
    let varInFunction = "I'm in a function.";
    console.log("hi there!", varInFunction);
};
varContainingFunction(); 
```

变量包含一定的值并且*是*某物；它们不*做*任何事情。函数是动作。它们是一组可以在被调用时执行的语句。JavaScript 不会在函数未被调用时运行这些语句。我们将在*匿名函数*部分回到将函数存储在变量中的想法，并考虑一些好处，但现在让我们继续看看命名函数的最佳方式。

## 函数命名

给你的函数命名可能看起来是一个微不足道的事情，但这里有一些最佳实践需要记住。为了保持简洁：

+   使用驼峰式命名法为你的函数命名：这意味着第一个单词以小写字母开头，新单词以大写字母开头。这使得阅读更容易，并保持你的代码一致性。

+   确保名称描述了函数正在做什么：将数字加法函数命名为`addNumbers`比`myFunc`更好。

+   使用一个动词来描述函数正在做什么：使其成为一个动作。所以，而不是`hiThere`，可以将其命名为`sayHi`。

## 练习第 6.1 节习题

看看你能否为自己编写一个函数。我们希望编写一个可以添加两个数字的函数。

1.  创建一个函数，该函数接受两个参数，将参数相加，并返回结果。

1.  设置两个不同的变量，并赋予它们不同的值。

1.  使用你的函数对两个变量进行操作，并使用`console.log`输出结果。

1.  使用两个更多的数字作为参数调用该函数的第二个调用。

## 练习第 6.2 节习题

我们将创建一个程序，该程序将随机描述输入的名字。

1.  创建一个描述性单词的数组。

1.  创建一个包含提示用户输入名字的函数。

1.  使用`Math.random`从数组中选择一个随机值。

1.  在控制台输出提示值和随机选择的数组值。

1.  调用该函数。

# 参数和参数

你可能已经注意到我们在谈论参数和参数。这两个术语通常用来表示传递给函数的信息：

```js
function tester(para1, para2){
    return para1 + " " + para2;
}
const arg1 = "argument 1";
const arg2 = "argument 2";
tester(arg1, arg2); 
```

参数定义为函数定义中括号内的变量，它定义了函数的作用域。它们是这样声明的：

```js
function myFunc(param1, param2) {
  // code of the function;
} 
```

一个实际例子可以是以下内容，它将`x`和`y`作为参数：

```js
function addTwoNumbers(x, y) {
  console.log(x + y);
} 
```

当被调用时，这个函数将简单地添加参数并记录结果。然而，为了做到这一点，我们可以用参数调用函数：

```js
myFunc("arg1", "arg2"); 
```

我们已经看到了各种参数的例子；例如：

```js
console.log("this is an argument");
prompt("argument here too");
let arr = [];
arr.push("argument"); 
```

根据你调用函数时使用的参数，函数的结果可以改变，这使得函数成为一个非常强大和灵活的构建块。使用我们的`addTwoNumbers()`函数的一个实际例子如下所示：

```js
addTwoNumbers(3, 4);
addTwoNumbers(12,-90); 
```

这将输出：

```js
7
-78 
```

如你所见，函数对两次调用都有不同的结果。这是因为我们用不同的参数调用它，这些参数取代了发送到函数中并在函数作用域内使用的`x`和`y`。

## 练习第 6.3 题

创建一个基本的计算器，它接受两个数字和一个表示操作的字符串值。如果操作等于加法，则应将两个数字相加。如果操作等于减法，则应从其中一个数字中减去另一个数字。如果没有指定选项，则选项的值应为`add`。

此函数的结果需要被记录。通过使用不同的运算符和未指定运算符来调用该函数来测试你的函数。

1.  设置两个包含数字值的变量。

1.  设置一个变量来保存一个运算符，可以是加号（+）或减号（-）。

1.  创建一个函数，它在其参数中检索两个值和运算符字符串值。使用这些值通过条件检查运算符是加号（+）还是减号（-），并相应地添加或减去值（记住如果没有提供有效的运算符，函数应默认为加法）。

1.  在`console.log()`中，使用你的变量调用该函数，并将响应输出到控制台。

1.  将运算符值更新为另一种运算符类型——加号或减号——然后再次使用新更新的参数调用该函数。

## 默认或不合适的参数

如果我们不传递任何参数就调用我们的`addTwoNumbers()`函数，会发生什么？花点时间决定你认为它应该做什么：

```js
addTwoNumbers(); 
```

一些语言可能会崩溃并哭泣，但 JavaScript 不会。JavaScript 只是给变量赋予一个默认类型，即未定义。而`undefined` + `undefined`等于：

```js
NaN 
```

相反，我们可以告诉 JavaScript 使用不同的默认参数。这可以这样做：

```js
function addTwoNumbers(x = 2, y = 3) {
  console.log(x + y);
} 
```

如果你现在调用该函数而不传递任何参数，它将自动将`2`赋值给`x`，将`3`赋值给`y`，除非你通过传递参数来覆盖它们。用于调用的值优先于硬编码的参数。因此，给定上述函数，以下函数调用的输出将是什么？

```js
addTwoNumbers();
addTwoNumbers(6, 6);
addTwoNumbers(10); 
```

输出将是：

```js
5
12
13 
```

第一个具有默认值，因此`x`是`2`，`y`是`3`。第二个将`6`赋值给`x`和`y`。最后一个有点不明显。我们只提供了一个参数，所以哪个参数会被赋予这个值？嗯，JavaScript 不喜欢过于复杂化。它只是将值赋给第一个参数，`x`。因此，`x`变为`10`，`y`得到其默认值`3`，两者相加为`13`。

如果你调用一个函数的参数多于参数，则不会发生任何事情。JavaScript 将仅使用可以映射到参数的第一个参数来执行函数。就像这样：

```js
addTwoNumbers(1,2,3,4); 
```

这将输出：

```js
3 
```

这只是将 1 和 2 相加，并忽略最后两个参数（`3`和`4`）。

# 特殊函数和运算符

函数的编写方式有一些特殊的方法，以及一些有用的特殊运算符。我们在这里讨论的是箭头函数和扩展运算符以及剩余运算符。箭头函数非常适合作为参数传递函数和使用更短的表示法。扩展运算符和剩余运算符使我们的工作更加容易，并且在传递参数和处理数组时更加灵活。

## 箭头函数

箭头函数是一种特殊的函数编写方式，一开始可能会让人感到困惑。它们的用法如下：

```js
(param1, param2) => body of the function; 
```

或者对于没有参数的情况：

```js
() => body of the function; 
```

或者对于单个参数（这里不需要括号）：

```js
param => body of the function; 
```

或者对于具有两个参数的多行函数：

```js
(param1, param2) => {
  // line 1;
  // any number of lines;
}; 
```

箭头函数在你想要即时编写实现的地方非常有用，比如在另一个函数作为参数时。这是因为它们是编写函数的简写表示法。它们最常用于只包含一个语句的函数。让我们从一个简单的函数开始，我们将将其重写为箭头函数：

```js
function doingStuff(x) {
  console.log(x);
} 
```

要将这段代码重写为箭头函数，你必须将其存储在变量中，或者作为参数传入，以便能够使用它。我们使用变量的名称来执行箭头函数。在这种情况下，我们只有一个参数，所以没有必要将其括在括号中。我们可以这样写：

```js
let doingArrowStuff = x => console.log(x); 
```

然后这样调用它：

```js
doingArrowStuff("Great!"); 
```

这将在控制台记录`Great!`。如果有多个参数，我们必须使用括号，如下所示：

```js
let addTwoNumbers = (x, y) => console.log(x + y); 
```

我们可以这样称呼它：

```js
addTwoNumbers(5, 3); 
```

然后它会在控制台记录`8`。如果没有参数，你必须使用括号，如下所示：

```js
let sayHi = () => console.log("hi"); 
```

如果我们调用`sayHi()`，它将在控制台记录`hi`。

作为最后的例子，我们可以将箭头函数与某些内置方法结合起来。例如，我们可以在数组上使用`forEach()`方法。这个方法对数组中的每个元素执行一个特定的函数。看看这个例子：

```js
const arr = ["squirrel", "alpaca", "buddy"];
arr.forEach(e => console.log(e)); 
```

它会输出：

```js
squirrel
alpaca
buddy 
```

对于数组中的每个元素，它都会将其作为输入，并对其执行箭头函数。在这种情况下，函数是记录元素。因此，输出是数组中的每个单独元素。

将箭头函数与内置函数结合使用非常强大。我们可以对数组中的每个元素执行某些操作，而无需计数或编写复杂的循环。我们将在稍后看到箭头函数的更多有用示例。

## 扩展运算符

扩展运算符是一个特殊运算符。它由在引用表达式或字符串之前使用的三个点组成，它将参数或数组的元素展开。

这可能听起来非常复杂，所以让我们看看一个简单的例子：

```js
let spread = ["so", "much", "fun"];
let message = ["JavaScript", "is", ...spread, "and", "very","powerful"]; 
```

这个数组的值变为：

```js
['JavaScript', 'is', 'so', 'much', 'fun', 'and', 'very', 'powerful'] 
```

如您所见，扩展运算符的元素成为数组中的单独元素。扩展运算符将数组扩展为新数组中的单独元素。它也可以用来向函数传递多个参数，如下所示：

```js
function addTwoNumbers(x, y) {
  console.log(x + y); 
} 
let arr = [5, 9];
addTwoNumbers(...arr); 
```

这将在控制台输出`14`，因为它等同于调用函数的方式：

```js
addTwoNumbers(5, 9); 
```

这个运算符避免了需要将长数组或字符串复制到函数中，这节省了时间并减少了代码复杂性。你可以用多个展开运算符调用一个函数。它将使用数组的所有元素作为输入。这里有一个例子：

```js
function addFourNumbers(x, y, z, a) { 
  console.log(x + y + z + a); 
} 
let arr = [5, 9];
let arr2 = [6, 7];
addFourNumbers(...arr, ...arr2); 
```

这将在控制台输出`27`，调用函数的方式如下：

```js
addFourNumbers(5, 9, 6, 7); 
```

## 剩余参数

与展开运算符类似，我们还有剩余参数。它具有与展开运算符相同的符号，但它用于函数参数列表中。记住如果我们像这里一样发送过多的参数会发生什么：

```js
function someFunction(param1, param2) {
  console.log(param1, param2);
}
someFunction("hi", "there!", "How are you?"); 
```

没错。实际上什么也没有：它只是假装我们只传入了两个参数并记录了`hi there!`。如果我们使用剩余参数，它允许我们传入任何数量的参数并将它们转换为参数数组。这里有一个例子：

```js
function someFunction(param1, ...param2) {
  console.log(param1, param2);
}
someFunction("hi", "there!", "How are you?"); 
```

这将记录：

```js
hi [ 'there!', 'How are you?' ] 
```

如您所见，第二个参数已变为一个数组，包含我们的第二个和第三个参数。这在你不确定将获得多少参数时非常有用。使用剩余参数允许你处理这个可变数量的参数，例如，使用循环。

# 返回函数值

我们还缺少一个非常重要的部分，使函数像它们一样有用：返回值。当我们指定返回值时，函数可以返回一个结果。返回值可以存储在变量中。我们已经这样做过了——还记得`prompt()`吗？

```js
let favoriteSubject = prompt("What is your favorite subject?"); 
```

我们将`prompt()`函数的结果存储在变量`favoriteSubject`中，在这个例子中，它将是用户指定的任何内容。让我们看看如果我们存储`addTwoNumbers()`函数的结果并将其记录下来会发生什么：

```js
let result = addTwoNumbers(4, 5);
console.log(result); 
```

你可能已经猜到了——这将记录以下内容：

```js
9
undefined 
```

值`9`被写入控制台，因为`addTwoNumbers()`包含一个`console.log()`语句。`console.log(result)`行输出`undefined`，因为没有将任何内容插入函数以存储结果，这意味着我们的函数`addTwoNumbers()`没有返回任何内容。由于 JavaScript 不喜欢引起麻烦和崩溃，它会分配`undefined`。为了解决这个问题，我们可以重新编写我们的`addTwoNumbers()`函数，使其真正返回值而不是记录它。这要强大得多，因为我们可以在代码的其余部分存储这个函数的结果并继续使用它：

```js
function addTwoNumbers(x, y) {
  return x + y;
} 
```

`return`结束函数并返回`return`之后的所有值。如果它是一个表达式，就像上面的一样，它将计算表达式的结果并将其返回到调用它的地方（在这个例子中是`result`变量）：

```js
let result = addTwoNumbers(4, 5);
console.log(result); 
```

```js
9 to the terminal.
```

你认为这段代码做什么？

```js
let resultsArr = [];
for(let i = 0; i < 10; i ++){
  let result = addTwoNumbers(i, 2*i);
  resultsArr.push(result);
}
console.log(resultsArr); 
```

它将所有结果记录到屏幕上。该函数正在循环中被调用。第一次迭代，`i`等于`0`。因此，结果是`0`。最后一次迭代，`i`等于`9`，因此数组的最后一个值等于`27`。以下是结果：

```js
[
   0,  3,  6,  9, 12,
  15, 18, 21, 24, 27
] 
```

## 练习题 6.4

将你在*练习题 6.2*中制作的计算器修改为返回加法值而不是打印它们。然后在循环中调用该函数 10 次或更多次，并将结果存储在数组中。一旦循环结束，将最终数组输出到控制台。

1.  设置一个空数组来存储循环中要计算的所有值。

1.  创建一个循环，运行 10 次，每次递增 1，每次迭代创建两个值。对于第一个值，将循环计数值乘以 5。对于第二个值，将循环计数器的值自乘。

1.  创建一个函数，当调用该函数时返回传入的两个参数的值。将值相加，返回结果。

1.  在循环中调用计算函数，将两个值作为参数传递给函数，并将返回的结果存储在响应变量中。

1.  仍然在循环中，随着循环的迭代，将结果值推入数组。

1.  循环完成后，将数组的值输出到控制台。

1.  你应该在控制台看到数组`[0, 6, 14, 24, 36, 50, 66, 84, 104, 126]`的值。

## 使用箭头函数返回

如果我们有一个单行箭头函数，我们可以不使用关键字`return`来返回。所以如果我们想重写这个函数，我们可以这样写，将其转换为箭头函数：

```js
let addTwoNumbers = (x, y) => x + y; 
```

我们可以这样调用并存储结果：

```js
let result = addTwoNumbers(12, 15);
console.log(result); 
```

这将在控制台输出`27`。如果是一个多行函数，你将不得不使用关键字`return`，就像在上一节中演示的那样。例如：

```js
let addTwoNumbers = (x, y) => {
  console.log("Adding...");
  return x + y;
} 
```

# 函数中的变量作用域

在本节中，我们将讨论一个经常被认为具有挑战性的主题。我们将讨论作用域。作用域定义了你可以访问某个变量的地方。当变量在作用域内时，你可以访问它。当变量不在作用域内时，你不能访问该变量。我们将对局部变量和全局变量进行讨论。

## 函数中的局部变量

局部变量仅在其定义的函数中有效。这对于`let`变量和`var`变量都适用。它们之间有一个区别，我们也会在这里简要提及。函数参数（它们不使用`let`或`var`）也是局部变量。这听起来可能很模糊，但下一个代码片段将演示这是什么意思：

```js
function testAvailability(x) {
  console.log("Available here:", x);
}
testAvailability("Hi!");
console.log("Not available here:", x); 
```

这将输出：

```js
Available here: Hi!
ReferenceError: x is not defined 
```

当在函数内部调用时，`x`将被记录。函数外的语句失败，因为`x`是`testAvailability()`函数的局部变量。这表明函数参数在函数外部是不可访问的。

它们在函数外部不可用，在函数内部可用。让我们看看函数内部定义的变量：

```js
function testAvailability() {
  let y = "Local variable!";
  console.log("Available here:", y);
}
testAvailability();
console.log("Not available here:", y); 
```

控制台显示了以下内容：

```js
Available here: Local variable!
ReferenceError: y is not defined 
```

在函数内部定义的变量在函数外部也是不可用的。

对于初学者来说，将局部变量和 `return` 结合起来可能会令人困惑。现在，我们告诉你函数内部声明的局部变量在函数外部不可用，但通过 `return`，你可以使它们的值在函数外部可用。所以如果你需要在函数外部使用它们的值，你可以返回这些值。这里的关键词是 *值*！你不能返回变量本身。相反，一个值可以被捕获并存储在不同的变量中，如下所示：

```js
function testAvailability() {
  let y = "I'll return";
  console.log("Available here:", y);
  return y;
}
let z = testAvailability();
console.log("Outside the function:", z);
console.log("Not available here:", y); 
```

因此，分配给局部变量 `y` 的返回值 `I'll return` 被返回并存储在变量 `z` 中。

这个变量 `z` 实际上也可以被称为 `y`，但那样会让人困惑，因为它仍然是一个不同的变量。

这段代码的输出如下：

```js
Available here: I'll return
Outside the function: I'll return
ReferenceError: y is not defined 
```

### `let` 与 `var` 变量

`let` 和 `var` 的区别在于 `var` 是函数作用域，这是我们上面描述的概念。实际上 `let` 不是函数作用域，而是块作用域。一个块由两个大括号 `{}` 定义。那些大括号内的代码是 `let` 仍然可用的地方。

让我们看看这个区别在实际中的应用：

```js
function doingStuff() {
  if (true) {
    var x = "local";
  }
  console.log(x);
}
doingStuff(); 
```

这段代码的输出将是：

```js
local 
```

如果我们使用 `var`，变量将成为函数作用域，并在函数块中的任何地方（甚至在定义之前值为 `undefined` 的情况下）可用。因此，在 `if` 块结束后，`x` 仍然可以被访问。

下面是 `let` 发生的情况：

```js
function doingStuff() {
  if (true) {
    let x = "local";
  }
  console.log(x);
}
doingStuff(); 
```

这将产生以下输出：

```js
ReferenceError: x is not defined 
```

这里我们得到一个错误，提示 `x` 未定义。由于 `let` 只能是块作用域，当 `if` 块结束时 `x` 就会超出作用域，并且之后不能再访问。

`let` 和 `var` 之间的最后一个区别与脚本中声明的顺序有关。尝试在定义 `let` 之前使用 `x` 的值：

```js
function doingStuff() { 
  if (true) { 
    console.log(x);
    let x = "local"; 
  } 
}
doingStuff(); 
```

这将给出一个 `ReferenceError`，提示 `x` 未初始化。这是因为使用 `let` 声明的变量在定义之前不能被访问，即使在同一个块内部。你认为这样的 `var` 声明会发生什么？

```js
function doingStuff() { 
  if (true) { 
    console.log(x);
    var x = "local";
  }
}
doingStuff(); 
```

这次，我们不会得到错误。当我们使用 `var` 变量在 `define` 语句之前时，我们只是得到 `undefined`。这是由于一个称为提升的现象，这意味着在使用 `var` 变量之前声明它会导致变量变为 `undefined` 而不是抛出 `ReferenceError`。

提升和如何在不必要的情况下消除其影响是更复杂的话题，我们将在 *第十二章*，*中级 JavaScript* 中介绍。

### `const` 作用域

常量是块作用域的，就像 `let` 一样。这就是为什么这里的范围规则与 `let` 的规则相似。以下是一个示例：

```js
function doingStuff() {
  if (true) {
    const X = "local";
  }
  console.log(X);
}
doingStuff(); 
```

这将产生以下输出：

```js
ReferenceError: X is not defined 
```

在定义它之前使用一个`const`变量也会引发一个`ReferenceError`，就像对`let`变量所做的那样。

## 全局变量

如你所猜，全局变量是在函数外部声明的，不在其他代码块中。变量在其定义的作用域（函数或代码块）内是可访问的，以及任何“较低”的作用域。因此，在函数外部定义的变量在函数内部以及该函数内部任何其他函数或其他代码块中都是可用的。因此，在程序顶层定义的变量在程序中的任何地方都是可用的。这个概念被称为全局变量。你可以在下面看到示例：

```js
let globalVar = "Accessible everywhere!";
console.log("Outside function:", globalVar);
function creatingNewScope(x) {
  console.log("Access to global vars inside function." , globalVar);
}
creatingNewScope("some parameter");
console.log("Still available:", globalVar); 
```

这将输出：

```js
Outside function: Accessible everywhere!
Access to global vars inside function. Accessible everywhere!
Still available: Accessible everywhere! 
```

如你所见，全局变量因为不在代码块中声明，所以可以从任何地方访问。它们在定义后总是处于作用域内——无论你在哪里使用它们。然而，你可以在该作用域内通过指定一个具有相同名称的新变量来隐藏它们的可访问性；这可以用于`let`、`var`和`const`。（这不会改变`const`变量的值；你正在创建一个新的`const`变量，它将在内部作用域中覆盖第一个。）在同一个作用域中，你不能指定两个具有相同名称的`let`或两个`const`变量。你可以为`var`这样做，但你不应该这样做，以避免混淆。

如果你在一个函数内部创建了一个具有相同名称的变量，那么在特定函数的作用域内引用该变量名称时，将使用该变量的值。这里有一个示例：

```js
let x = "global";
function doingStuff() {
  let x = "local";
  console.log(x);
}
doingStuff();
console.log(x); 
```

这将输出：

```js
local
global 
```

如你所见，`doingStuff()`函数内部的`x`的值是`局部`的。然而，在函数外部，值仍然是`全局`的。这意味着你必须非常小心，不要在局部和全局作用域中混淆名称。通常最好避免这样做。

参数名称也是如此。如果你有一个与全局变量相同的参数名称，将使用参数的值：

```js
let x = "global";

function doingStuff(x) {
  console.log(x); 
} 

doingStuff("param"); 
```

这将记录`param`。

过度依赖全局变量存在风险。当你的应用程序增长时，你很快就会遇到这个问题。正如我们刚才看到的，局部变量会覆盖全局变量的值。最好在函数中使用局部变量；这样，你就能更好地控制你所处理的内容。现在这可能会有些模糊，但当你在实际编码中遇到更大、更多行和文件代码时，它将变得清晰。

目前关于作用域还有一个非常重要的点需要说明。让我们从一个例子开始，看看你是否能弄清楚应该记录什么：

```js
function confuseReader() {
  x = "Guess my scope...";
  console.log("Inside the function:", x);
}
confuseReader();
console.log("Outside of function:", x); 
```

准备好了吗？以下是输出结果：

```js
Inside the function: Guess my scope...
Outside of function: Guess my scope... 
```

不要关闭这本书——我们将解释正在发生的事情。如果你仔细观察，函数中的 `x` 在没有 `let` 或 `var` 关键字的情况下被定义。在代码上方没有 `x` 的声明；这是整个程序的代码。JavaScript 不看到 `let` 或 `var`，然后决定，“这必须是一个全局变量。”即使它是在函数内部定义的，函数内部 `x` 的声明具有全局作用域，并且仍然可以在函数外部访问。

我们真的想强调，这是一个糟糕的做法。如果你需要一个全局变量，请在文件顶部声明它。

## 立即调用的函数表达式

**立即调用的函数表达式**（**IIFE**）是一种表达函数的方式，使其立即被调用。它是匿名的，没有名字，并且是自我执行的。

当你想使用此函数初始化某些内容时，这很有用。它也被用于许多设计模式中，例如，创建私有和公共变量和函数。

这与函数和变量可访问的位置有关。如果你在顶层作用域中有一个 IIFE，即使它位于顶层，其中的内容也无法从外部访问。

这就是如何定义它的：

```js
(function () {
  console.log("IIFE!");
})(); 
```

函数本身被括号包围，这使得它创建一个函数实例。如果没有这些括号，它会抛出一个错误，因为我们的函数没有名字（尽管可以通过将函数赋给一个变量来解决这个问题，这样输出就可以返回到变量中）。

`();` 执行一个未命名的函数——这必须在函数声明之后立即完成。如果你的函数需要参数，你可以在这些最后的括号内传递它。

你还可以将 IIFE 与其他函数模式结合使用。例如，你可以在这里使用箭头函数来使函数更加简洁：

```js
(()=>{
    console.log("run right away");
})(); 
```

再次，我们使用 `();` 来调用你创建的函数。

### 练习 6.5

使用 IIFE 创建几个立即调用的函数，并观察作用域是如何受到影响的。

1.  使用 `let` 创建一个变量值，并将其赋值为字符串 1000。

1.  创建一个 IIFE 函数，并在该函数作用域内将新值赋给同名变量。在函数内部，将局部值打印到控制台。

1.  创建一个 IIFE 表达式，将其赋给新的 `result` 变量，并在该作用域内将新值赋给同名变量。将此局部值返回到 `result` 变量，并调用该函数。打印 `result` 变量，以及你一直在使用的变量名：现在它包含什么值？

1.  最后，创建一个带有参数的匿名函数。添加逻辑，将传递的值赋给与其他步骤相同的变量名，并将其作为字符串句子的一部分打印出来。调用该函数，并在圆括号内传递你希望的价值。

# 递归函数

在某些情况下，你可能需要在函数内部调用同一个函数。这可能是解决相对复杂问题的美丽解决方案。但是也有一些需要注意的事情。你认为这会做什么？

```js
function getRecursive(nr) {
  console.log(nr);
  getRecursive(--nr);
}
getRecursive(3); 
```

它打印`3`然后开始倒计时并且永远不会停止。为什么它不会停止呢？好吧，我们并没有说它应该在什么时候停止。看看我们改进的版本：

```js
function getRecursive(nr) {
  console.log(nr);
  if (nr > 0) {
    getRecursive(--nr);
  }
}
getRecursive(3); 
```

这个函数将会一直调用自己，直到参数的值不再大于`0`。然后它才会停止。

当我们递归地调用一个函数时会发生什么？每次都会进入一个函数的更深层次。第一次函数调用是最后完成的。对于这个函数，它的过程是这样的：

+   `getRecursive(3)`

    +   `getRecursive(2)`

        +   `getRecursive(1)`

            +   `getRecursive(0)`

            +   `getRecursive(0)`执行完成

        +   `getRecursive(1)`执行完成

    +   `getRecursive(2)`执行完成

+   `getRecursive(3)`执行完成

下一个递归函数将演示：

```js
function logRecursive(nr) {
  console.log("Started function:", nr);
  if (nr > 0) {
    logRecursive(nr - 1);
  } else {
      console.log("done with recursion");
  }
  console.log("Ended function:", nr);
}
logRecursive(3); 
```

它将输出：

```js
Started function: 3
Started function: 2
Started function: 1
Started function: 0
done with recursion
Ended function: 0
Ended function: 1
Ended function: 2
Ended function: 3 
```

在某些情况下，递归函数可以非常出色。当你觉得需要在循环中反复调用同一个函数时，你可能应该考虑使用递归。一个例子也可以是搜索某物。你不需要在同一个函数内部循环遍历所有内容，你可以在函数内部分割，并从内部反复调用函数。

然而，必须记住的是，通常情况下，递归的性能略低于使用循环的常规迭代的性能。所以如果这会导致瓶颈情况，真正地减慢你的应用程序，那么你可能需要考虑另一种方法。

看看以下练习中如何使用递归函数计算阶乘。

## 练习 6.6

我们可以用递归解决的一个常见问题是计算阶乘。

关于阶乘的快速数学复习：

一个数的阶乘是所有大于 0 的正整数的乘积，直到这个数本身。例如，7 的阶乘是 7 * 6 * 5 * 4 * 3 * 2 * 1。你可以写成 7!。

递归函数如何帮助我们计算阶乘？我们将用较小的数字调用函数，直到达到 0。在这个练习中，我们将使用递归来计算函数参数设置的数值的阶乘结果。

1.  创建一个函数，在其中包含一个条件，检查参数值是否为`0`。

1.  如果参数等于`0`，它应该返回`1`的值。否则，它应该返回参数值乘以函数自身返回的值，并从提供的参数值中减去`1`。这将导致代码块运行，直到值达到`0`。

1.  调用函数，提供一个你想要找到阶乘的数字作为参数。代码应该运行最初传递到函数中的任何数字，一直减少到`0`，并将计算结果输出到控制台。它也可以包含一个`console.log()`调用，以打印函数被调用时的当前参数值。

1.  更改并更新数字，看看它如何影响结果。

# 嵌套函数

就像循环、`if`语句以及实际上所有其他构建块一样，我们可以在函数内部有函数。这种现象称为嵌套函数：

```js
function doOuterFunctionStuff(nr) {
  console.log("Outer function");
  doInnerFunctionStuff(nr);
  function doInnerFunctionStuff(x) {
    console.log(x + 7);
    console.log("I can access outer variables:", nr);
  }
}
doOuterFunctionStuff(2); 
```

这将输出：

```js
Outer function
9
I can access outer variables: 2 
```

如您所见，外部函数正在调用其嵌套函数。这个嵌套函数可以访问父函数的变量。反过来，情况并非如此。在内部函数内部定义的变量具有函数作用域。这意味着它们可以在定义它们的函数内部访问，在这种情况下是内部函数。因此，这将抛出一个`ReferenceError`：

```js
function doOuterFunctionStuff(nr) {
  doInnerFunctionStuff(nr);
  function doInnerFunctionStuff(x) {
    let z = 10;
  }
  console.log("Not accessible:", z);
}
doOuterFunctionStuff(2); 
```

你认为这会做什么？

```js
function doOuterFunctionStuff(nr) {
  doInnerFunctionStuff(nr);
  function doInnerFunctionStuff(x) {
    let z = 10;
  }
}
doInnerFunctionStuff(3); 
```

这也会抛出一个`ReferenceError`。现在，`doInnerFunctionStuff()`是在外部函数中定义的，这意味着它只在外部函数`doOuterFunctionStuff()`的作用域内有效。在这个函数外部，它就不再有效。

## 练习题 6.7

从一个动态值`10`开始创建一个倒计时循环。

1.  将`start`变量设置为`10`，这将是循环的起始值。

1.  创建一个函数，它接受一个参数，即倒计时值。

1.  在函数内部，将倒计时的当前值输出到控制台。

1.  添加一个条件来检查值是否小于 1；如果是，则返回该函数。

1.  添加一个条件来检查倒计时的值是否不小于 1，然后通过在函数内部调用函数来继续循环。

1.  确保你在倒计时上添加一个递减运算符，这样前面的条件最终会变为真以结束循环。每次循环时，值都会减少，直到达到`0`。

1.  如果值大于 0，则使用条件更新和创建第二个倒计时。如果是，则将倒计时的值减 1。

1.  使用`return`返回函数，然后再次调用它，直到条件不再为真。

1.  确保当你将新的倒计时值作为参数发送到函数中时，有一个方法通过使用`return`关键字和一个条件来退出循环，如果满足条件则继续循环。

# 匿名函数

到目前为止，我们一直在给我们的函数命名。如果我们将它们存储在变量中，我们也可以创建没有名称的函数。我们称这些函数为匿名函数。以下是一个非匿名函数：

```js
function doingStuffAnonymously() {
  console.log("Not so secret though.");
} 
```

这是如何将前面的函数转换为匿名函数的方法：

```js
function () {
  console.log("Not so secret though.");
}; 
```

如您所见，我们的函数没有名称。它是匿名的。所以你可能想知道你如何调用这个函数。实际上，你不能这样调用！

我们必须将其存储在变量中才能调用匿名函数；我们可以这样存储：

```js
let functionVariable = function () {
  console.log("Not so secret though.");
}; 
```

匿名函数可以通过变量名来调用，如下所示：

```js
functionVariable(); 
```

它将简单地输出 `Not so secret though.`。

这可能看起来有点无用，但它是一个非常强大的 JavaScript 构造。将函数存储在变量中使我们能够做非常酷的事情，比如将函数作为参数传递。这个概念为编程添加了另一个抽象层。这个概念被称为回调，我们将在下一节中讨论它。

## 练习 6.8

1.  设置一个变量名并将一个函数分配给它。创建一个带有一个参数的函数表达式，该参数将提供的参数输出到控制台。

1.  将参数传递给函数。

1.  将相同的函数创建为一个正常的函数声明。

## 函数回调

这里是一个将函数作为参数传递给另一个函数的例子：

```js
function doFlexibleStuff(executeStuff) {
  executeStuff();
  console.log("Inside doFlexibleStuffFunction.");
} 
```

如果我们用之前创建的匿名函数 `functionVariable` 调用这个新函数，如下所示：

```js
doFlexibleStuff(functionVariable); 
```

它将输出：

```js
Not so secret though.
Inside doFlexibleStuffFunction. 
```

但我们也可以用另一个函数来调用它，然后我们的 `doFlexibleStuff` 函数将执行这个其他函数。这有多酷？

```js
let anotherFunctionVariable = function() {
  console.log("Another anonymous function implementation.");
}
doFlexibleStuff(anotherFunctionVariable); 
```

这将产生以下输出：

```js
Another anonymous function implementation.
Inside doFlexibleStuffFunction. 
```

那么，发生了什么？我们创建了一个函数，并将其存储在 `anotherFunctionVariable` 变量中。然后我们将它作为函数参数传递给我们的 `doFlexibleStuff()` 函数。这个函数只是简单地执行被传递进来的任何函数。

在这一点上，你可能想知道为什么作者对这种回调概念如此兴奋。在你迄今为止看到的例子中，这可能看起来相当平淡无奇。一旦我们稍后讨论异步函数，这个概念将会非常有帮助。为了满足你对更具体例子的需求，我们将给你一个例子。

如你所知，JavaScript 中有很多内置函数。其中之一是 `setTimeout()` 函数。这是一个非常特殊的函数，它会在等待一段指定的时间后执行某个函数。它似乎也负责很多性能极差的网页，但这绝对不是这个可怜的、被误解和误用的函数的错。

这段代码真的是你应该尝试理解的东西：

```js
let youGotThis = function () {
  console.log("You're doing really well, keep coding!");
};
setTimeout(youGotThis, 1000); 
```

它将等待 `1000`ms（一秒）然后打印：

```js
You're doing really well, keep coding! 
```

如果你需要更多的鼓励，你可以使用 `setInterval()` 函数。它的工作方式非常相似，但与只执行一次指定的函数不同，它会在指定的间隔内持续执行：

```js
setInterval(youGotThis, 1000); 
```

在这种情况下，它将每秒打印一次鼓励信息，直到你终止程序。

这种在函数被调用后执行函数的概念对于管理异步程序执行非常有用。

# 章节项目

## 创建一个递归函数

创建一个递归函数，该函数从 10 开始计数。用不同的起始数字作为传递给函数的参数来调用该函数。函数应运行到值大于 10 为止。

## 设置超时顺序

使用箭头格式创建函数，将值`one`和`two`输出到控制台。创建一个第三个函数，将值`three`输出到控制台，然后调用前两个函数。

创建一个第四个函数，该函数将单词`four`输出到控制台，并使用`setTimeout()`立即调用第一个函数，然后调用第三个函数。

你的输出在控制台看起来是什么样子？尝试让控制台输出：

```js
Four
Three
One
Two
One 
```

# 自我检查测验

1.  输出到控制台的是什么值？

    ```js
    let val = 10;
    function tester(val){
        val += 10;
        if(val < 100){
            return tester(val);
        }
        return val;
    }
    tester(val);
    console.log(val); 
    ```

1.  以下代码将输出到控制台的内容是什么？

    ```js
    let testFunction = function(){
        console.log("Hello");
    }(); 
    ```

1.  控制台将输出什么？

    ```js
    (function () {
        console.log("Welcome");
    })();
    (function () {
        let firstName = "Laurence";
    })();
    let result = (function () {
        let firstName = "Laurence";
        return firstName;
    })();
    console.log(result);
    (function (firstName) {
        console.log("My Name is " + firstName);
    })("Laurence"); 
    ```

1.  控制台将输出什么？

    ```js
    let test2 = (num) => num + 5;
    console.log(test2(14)); 
    ```

1.  控制台将输出什么？

    ```js
    var addFive1 = function addFive1(num) {
    return num + 2;
    };
    let addFive2 = (num) => num + 2;
    console.log(addFive1(14)); 
    ```

# 概述

在本章中，我们介绍了函数。函数是 JavaScript 的一个强大构建块，我们可以用它来重用代码行。我们可以给函数传递参数，这样我们就可以根据函数被调用的参数来改变代码。函数可以返回一个结果；我们使用`return`关键字来这样做。我们还可以在调用函数的地方使用`return`。我们可以将结果存储在变量中或在另一个函数中使用，例如。

然后，我们遇到了变量作用域。作用域包括变量可访问的地方。默认的`let`和`const`变量可以在定义它们的块（及其内部块）内访问，而`var`则仅可以从定义它的行访问。

我们还可以使用递归函数优雅地解决那些本质上可以递归解决的问题，例如计算阶乘。嵌套函数是我们接下来研究的下一个主题。它们并不是什么大问题，只是函数内部的函数。函数内部的简单函数并不被认为很漂亮，但匿名函数和箭头函数并不少见。匿名函数是没有名称的函数，箭头函数是匿名函数的一种特殊情况，其中我们使用箭头分隔参数和主体。

在下一章中，我们将考虑类，这是另一种强大的编程结构！
