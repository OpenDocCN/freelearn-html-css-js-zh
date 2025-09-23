# 第一章. 玩转语法

**JavaScript** 在与声明常量变量、声明块作用域变量、从数组中提取数据、函数声明更简短的语法等各种语法形式相比时，落后于一些其他编程语言。**ES6** 为 JavaScript 添加了许多基于新语法的特性，这有助于开发者写出更少但功能更强大的代码。ES6 还防止程序员使用各种为了达到各种目标而采取的“黑客”手段，这些手段会对性能产生负面影响，并使代码更难以阅读。在本章中，我们将探讨 ES6 引入的新语法特性。

在本章中，我们将涵盖：

+   使用 `let` 关键字创建块作用域变量

+   使用 `const` 关键字创建常量变量

+   扩展运算符和剩余参数

+   使用解构赋值从可迭代对象和对象中提取数据

+   箭头函数

+   创建对象属性的新语法

# 使用 `let` 关键字

ES6 的 `let` 关键字用于声明一个块作用域变量，可选地初始化它为一个值。来自其他编程语言背景但新接触 JavaScript 的程序员，常常会编写出容易出错的 JavaScript 程序，认为 JavaScript 变量是块作用域的。几乎每种流行的编程语言在变量作用域方面都有相同的规则，但 JavaScript 由于缺乏块作用域变量而略有不同。由于 JavaScript 变量不是块作用域的，存在内存泄漏的风险，而且 JavaScript 程序也更难以阅读和调试。

## 声明函数作用域变量

使用 `var` 关键字声明的 JavaScript 变量被称为 **函数作用域** 变量。函数作用域变量在整个脚本中全局可访问，即如果在外部声明，则在整个脚本中可访问。同样，如果函数作用域变量在函数内部声明，则它们在整个函数中可访问，但在函数外部不可访问。

这里有一个示例，展示了如何创建函数作用域变量：

```js
var a = 12; //accessible globally

function myFunction()
{
  console.log(a);

  var b = 13; //accessible throughout function

  if(true)
  {
    var c = 14; //accessible throughout function
    console.log(b);
  }

  console.log(c);
}

myFunction();
```

代码的输出如下：

```js
12
13
14
```

### 小贴士

**下载示例代码**

您可以从您在 [`www.packtpub.com`](http://www.packtpub.com) 的账户中下载您购买的所有 Packt 书籍的示例代码文件。如果您在其他地方购买了这本书，您可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 并注册，以便将文件直接通过电子邮件发送给您。

在这里，您可以看到 `c` 变量在 `if` 语句外部是可访问的，但在其他编程语言中并非如此。因此，来自其他语言的程序员会期望 `c` 变量在 `if` 语句外部是未定义的，但事实并非如此。因此，ES6 引入了 `let` 关键字，它可以用来创建块作用域变量。

## 声明块作用域变量

使用`let`关键字声明的变量被称为块作用域变量。当在函数外部声明时，块作用域变量的行为与函数作用域变量相同，即它们是全局可访问的。但是，当块作用域变量在块内部声明时，它们只在其定义的块内部（以及任何子块）可访问，而不可在块外部访问。

### 注意

块用于组合零个或多个语句。一对花括号定义了块，即`{}`。

让我们拿之前的示例脚本，将`var`替换为`let`关键字，看看输出结果：

```js
let a = 12; //accessible globally

function myFunction()
{
  console.log(a);

  let b = 13; //accessible throughout function

  if(true)
  {
    let c = 14; //accessible throughout the "if" statement
    console.log(b);
  }

  console.log(c);
}

myFunction();
```

代码的输出是：

```js
12
13
Reference Error Exception
```

现在，输出结果符合习惯使用其他编程语言的程序员的预期。

## 重新声明变量

当你使用`var`关键字声明一个变量，而这个变量在同一作用域内已经使用`var`关键字声明时，它会被覆盖。考虑以下示例：

```js
var a = 0;
var a = 1;

console.log(a);

function myFunction()
{
  var b = 2;
  var b = 3;

  console.log(b);
}

myFunction();
```

代码的输出是：

```js
1
3
```

输出结果符合预期。但是使用`let`关键字创建的变量表现方式并不相同。

当你使用`let`关键字声明一个变量，而这个变量在同一作用域内已经使用`let`关键字声明时，它会抛出一个`TypeError`异常。考虑以下示例：

```js
let a = 0;
let a = 1; //TypeError

function myFunction()
{
  let b = 2;
  let b = 3; //TypeError

  if(true)
  {
    let c = 4;
    let c = 5; //TypeError
  }
}

myFunction();
```

当你使用一个在函数（或内部函数）中已经可访问的名称声明一个变量，或者使用`var`或`let`关键字分别创建一个子块时，那么它是一个不同的变量。这里有一个示例来展示这种行为：

```js
var a = 1;
let b = 2;

function myFunction()
{
  var a = 3; //different variable
  let b = 4; //different variable

  if(true)
  {
    var a = 5; //overwritten
    let b = 6; //different variable

    console.log(a);
    console.log(b);
  }

  console.log(a);
  console.log(b);
}

myFunction();

console.log(a);
console.log(b);
```

代码的输出是：

```js
5
6
5
4
1
2
```

### 注意

`var`与`let`，哪一个该用？

当编写 ES6 代码时，建议切换到使用`let`关键字，因为它使脚本更节省内存，防止作用域错误，防止意外错误，并使代码更容易阅读。但如果你已经习惯了`var`关键字并且使用起来很舒服，那么你仍然可以使用它。

你可能想知道为什么不直接使用`var`关键字来定义块作用域变量，而不是引入`let`关键字？之所以没有将`var`关键字做得足够用来定义块作用域变量，而是引入`let`关键字，是为了向后兼容。

# `const`关键字

ES6 的`const`关键字用于声明只读变量，即值不能重新分配的变量。在 ES6 之前，程序员通常会在应该为常量的变量前加上前缀。例如，看看以下代码：

```js
var const_pi = 3.141;
var r = 2;
console.log(const_pi * r * r); //Output "12.564"
```

`pi`的值应该始终保持不变。在这里，尽管我们已添加了前缀，但仍有可能在程序中的某个地方意外更改其值，因为它们没有对`pi`值的原生保护。添加前缀只是不足以追踪常量变量。

因此，引入了`const`关键字来为常量变量提供原生保护。所以，之前的程序在 ES6 中应该这样编写：

```js
const pi = 3.141;
var r = 2;

console.log(pi * r * r); //Output "12.564"

pi = 12; //throws read-only exception
```

在这里，当我们尝试改变 `pi` 的值时，抛出了一个只读异常。

## 常量变量的作用域

常量变量是块作用域变量，即它们遵循与使用 `let` 关键字声明的变量相同的作用域规则。以下是一个示例，展示了常量变量的作用域：

```js
const a = 12; //accessible globally

function myFunction()
{
  console.log(a);

  const b = 13; //accessible throughout function

  if(true)
  {
    const c = 14; //accessible throughout the "if" statement
    console.log(b);
  }

  console.log(c);
}

myFunction();
```

上述代码的输出是：

```js
12
13
ReferenceError Exception
```

在这里，我们可以看到，常量变量在作用域规则方面与块作用域变量表现相同。

## 使用常量变量引用对象

当我们将一个对象赋值给一个变量时，变量持有的是对象的引用而不是对象本身。因此，当将对象赋值给一个常量变量时，对象的引用就变成了对该变量的常量引用，而不是对对象本身的引用。因此，对象是可变的。

考虑这个例子：

```js
const a = {
  "name" : "John"
};

console.log(a.name);

a.name = "Eden";

console.log(a.name);

a = {}; //throws read-only exception
```

上述代码的输出是：

```js
John
Eden
a is read only: Exception
```

在这个例子中，`a` 变量存储了对象的地址（即引用）。因此，对象的地址是 `a` 变量的值，且不能被改变。但是对象是可变的。所以当我们尝试将另一个对象赋值给 `a` 变量时，我们得到了一个异常，因为我们试图改变 `a` 变量的值。

# 默认参数值

在 JavaScript 中，没有定义的方法来为未传递的函数参数分配默认值。因此，程序员通常检查具有 `undefined` 值的参数（因为它是缺失参数的默认值），并将默认值分配给它们。以下是一个示例，展示了如何做到这一点：

```js
function myFunction(x, y, z)
{
  x = x === undefined ? 1 : x;
  y = y === undefined ? 2 : y;
  z = z === undefined ? 3 : z;

  console.log(x, y, z); //Output "6 7 3"
}
myFunction(6, 7);
```

ES6 提供了一种新的语法，可以更简单地实现这一点。以下代码演示了如何在 ES6 中实现这一点：

```js
function myFunction(x = 1, y = 2, z = 3)
{
  console.log(x, y, z); // Output "6 7 3"
}

myFunction(6,7);
```

此外，传递 `undefined` 被视为缺少参数。以下是一个示例来演示这一点：

```js
function myFunction(x = 1, y = 2, z = 3)
{
  console.log(x, y, z); // Output "1 7 9"
}

myFunction(undefined,7,9);
```

默认值也可以是表达式。以下是一个示例来演示这一点：

```js
function myFunction(x = 1, y = 2, z = 3 + 5)
{
  console.log(x, y, z); // Output "6 7 8"
}

myFunction(6,7);
```

# 扩展运算符

**扩展运算符**由"`…`"符号表示。扩展运算符将可迭代对象拆分为单独的值。

### 注意

可迭代对象是包含一组值并实现 ES6 可迭代协议的对象，使我们能够遍历其值。数组是内置的可迭代对象的一个例子。

扩展运算符可以放置在代码中期望多个函数参数或多个元素（对于数组字面量）的任何位置。

扩展运算符通常用于将可迭代对象的值扩展到函数的参数中。让我们以一个数组为例，看看如何将其拆分为函数的参数。

在 ES6 之前，为了将数组的值作为函数参数提供，程序员使用函数的 `apply()` 方法。以下是一个示例：

```js
function myFunction(a, b)
{
  return a + b;
}

var data = [1, 4];
var result = myFunction.apply(null, data);

console.log(result); //Output "5"
```

在这里，`apply` 方法接受一个数组，提取其值，将它们作为单独的参数传递给函数，然后调用它。

ES6 提供了一种简单的方法来实现这一点，使用扩展运算符。以下是一个示例：

```js
function myFunction(a, b)
{
 return a + b;
}

let data = [1, 4];
let result = myFunction(...data);
console.log(result); //Output "5"
```

在运行时，在 JavaScript 解释器调用`myFunction`函数之前，它将`…data`替换为`1,4`表达式：

```js
let result = myFunction(...data);
```

之前的代码被替换为：

```js
let result = myFunction(1,4);
```

在此之后，函数被调用。

### 注意

扩展操作符不调用`apply()`方法。JavaScript 运行时引擎使用迭代协议来展开数组，这与`apply()`方法无关，但行为相同。

## 扩展操作符的其他用法

扩展操作符不仅限于将可迭代对象展开为函数参数，它还可以用于代码中期望多个元素（对于数组字面量）的任何地方。因此，它有很多用途。让我们看看扩展操作符在数组中的其他一些用例。

### 将数组值作为另一个数组的一部分

它也可以用来将数组值作为另一个数组的一部分。以下是一个示例代码，演示了如何在创建数组的同时将现有数组的值作为另一个数组的一部分。

```js
let array1 = [2,3,4];
let array2 = [1, ...array1, 5, 6, 7];

console.log(array2); //Output "1, 2, 3, 4, 5, 6, 7"
```

这里是以下行：

```js
let array2 = [1, ...array1, 5, 6, 7];
```

将被替换为以下行：

```js
let array2 = [1, 2, 3, 4, 5, 6, 7];
```

### 将数组值推入另一个数组

有时，我们可能需要将现有数组的值推入另一个现有数组的末尾。

在 ES6 之前，程序员通常这样做：

```js
var array1 = [2,3,4];
var array2 = [1];

Array.prototype.push.apply(array2, array1);

console.log(array2); //Output "1, 2, 3, 4"
```

但在 ES6 中，我们有一个更简洁的方式来处理，如下所示：

```js
let array1 = [2,3,4];
let array2 = [1];

array2.push(...array1);

console.log(array2); //Output "1, 2, 3, 4"
```

这里，push 方法接受一系列变量，并将它们添加到被调用数组的末尾。

这里是以下行：

```js
array2.push(...array1);
```

将被替换为以下行：

```js
array2.push(2, 3, 4);
```

## 扩展多个数组

多个数组可以被展开到一行表达式上。例如，以下代码：

```js
let array1 = [1];
let array2 = [2];
let array3 = [...array1, ...array2, ...[3, 4]];//multi array spread
let array4 = [5];

function myFunction(a, b, c, d, e)
{
  return a+b+c+d+e;
}

let result = myFunction(...array3, ...array4); //multi array spread

console.log(result); //Output "15"
```

# 剩余参数

剩余参数也由“`…`”标记表示。带有“`…`”前缀的函数的最后一个参数被称为剩余参数。剩余参数是一个数组类型，它包含当参数数量超过命名参数数量时函数的其余参数。

剩余参数用于在函数内部捕获可变数量的函数参数。

在 ES6 之前，程序员使用函数的`arguments`对象来检索传递给函数的额外参数。`arguments`对象不是一个数组，但它提供了一些类似于数组的接口。

这里有一个代码示例，展示了如何使用`arguments`对象来检索额外的参数：

```js
function myFunction(a, b)
{
  var args = Array.prototype.slice.call(arguments, myFunction.length);

  console.log(args);
}

myFunction(1, 2, 3, 4, 5); //Output "3, 4, 5"
```

在 ES6 中，这可以通过使用剩余参数以更简单、更干净的方式完成。以下是一个使用剩余参数的示例：

```js
function myFunction(a, b, ...args)
{
  console.log(args); //Output "3, 4, 5"
}

myFunction(1, 2, 3, 4, 5);
```

`arguments`对象不是一个数组对象。因此，要在`arguments`对象上执行数组操作，您需要首先将其转换为数组。由于 ES6 的剩余参数是数组类型，与之一起工作更容易。

### 注意

“…”标记被称为什么？

“…”标记被称为扩展操作符或剩余参数，具体取决于其使用方式和位置。

# 解构赋值

解构赋值是一个表达式，允许您使用类似于数组或对象构造字面量的语法将可迭代或对象的值或属性分配给变量。

解构赋值通过提供更短的语法，使得从可迭代对象或对象中提取数据变得容易。解构赋值已经在编程语言中存在，例如**Perl**和**Python**，并且它们的工作方式在所有地方都是相同的。

有两种解构赋值表达式——数组和对象解构赋值。让我们详细看看每一种。

## 数组解构赋值

数组解构赋值用于从可迭代对象中提取值并将它们分配给变量。它被称为*数组解构赋值*，因为表达式类似于数组构造字面量。

在 ES6 之前，程序员通常这样做来将数组值分配给变量：

```js
var myArray = [1, 2, 3];
var a = myArray[0];
var b = myArray[1];
var c = myArray[2];
```

在这里，我们正在提取数组的值并将它们分别分配给`a`、`b`、`c`变量。

在 ES6 中，我们可以通过一行语句使用数组解构赋值来完成此操作：

```js
let myArray = [1, 2, 3];
let a, b, c;

[a, b, c] = myArray; //array destructuring assignment syntax
```

如您所见，`[a, b, c]`是数组解构表达式。

在数组解构语句的左侧，我们需要放置想要分配数组值的变量，使用与数组字面量类似的语法。在右侧，我们需要放置一个数组（实际上可以是任何可迭代对象），从中提取我们想要的值。

之前的示例代码可以这样进一步缩短：

```js
let [a, b, c] = [1, 2, 3];
```

在这里，我们在同一语句中创建变量，而不是提供数组变量，而是提供具有构造字面量的数组。

如果数组中的变量数量少于项目数量，则只考虑前几个项目。

### 注意

如果在数组解构赋值语法的右侧放置一个非可迭代对象，则会抛出`TypeError`异常。

### 忽略值

我们也可以忽略可迭代对象中的一些值。以下是一个示例代码，展示了如何做到这一点：

```js
let [a, , b] = [1, 2, 3];

console.log(a);
console.log(b);
```

输出如下：

```js
1
3
```

### 在数组解构赋值中使用剩余操作符

我们可以使用“`…`”标记作为数组解构表达式最后一个变量的前缀。在这种情况下，如果其他变量的数量少于可迭代对象中的值，则该变量始终被转换为数组对象，该对象包含可迭代对象剩余的值。

考虑以下示例来理解它：

```js
let [a, ...b] = [1, 2, 3, 4, 5, 6];

console.log(a);
console.log(Array.isArray(b));
console.log(b);
```

输出如下：

```js
1
true
2,3,4,5,6
```

在之前的示例代码中，您可以看到`b`变量被转换为数组，并且它包含右侧数组中所有其他值。

这里，“`…`”标记被称为**剩余操作符**。

我们也可以在使用剩余操作符时忽略值。以下是一个示例来演示这一点：

```js
let [a, , ,...b] = [1, 2, 3, 4, 5, 6];

console.log(a);
console.log(b);
```

输出如下：

```js
1
4,5,6
```

这里，我们忽略了`2,` `3`值。

### 变量的默认值

在展开过程中，如果你提供一个数组索引是`undefined`，你也可以为变量提供默认值。以下是一个演示此功能的示例：

```js
let [a, b, c = 3] = [1, 2];
console.log(c); //Output "3"
```

### 嵌套数组展开

我们还可以从多维数组中提取值并将它们分配给变量。以下是一个演示此功能的示例：

```js
let [a, b, [c, d]] = [1, 2, [3, 4]];
```

### 将展开赋值用作参数

我们还可以使用数组展开表达式作为函数参数，以提取作为函数参数传递的可迭代对象的值。以下是一个演示此功能的示例：

```js
function myFunction([a, b, c = 3])
{
  console.log(a, b, c); //Output "1 2 3"
}

myFunction([1, 2]);
```

在本章前面，我们看到了如果我们将`undefined`作为参数传递给函数调用，那么 JavaScript 会检查默认参数值。因此，我们也可以在这里提供一个默认数组，如果参数是`undefined`，则将使用该数组。以下是一个演示此功能的示例：

```js
function myFunction([a, b, c = 3] = [1, 2, 3])
{
  console.log(a, b, c);  //Output "1 2 3"
}

myFunction(undefined);
```

这里，我们传递了`undefined`作为参数，因此使用了默认数组`[1, 2, 3]`来提取值。

## 对象展开赋值

对象展开赋值用于提取对象的属性值并将它们分配给变量。

在 ES6 之前，程序员通常以以下方式将对象的属性值赋给变量：

```js
var object = {"name" : "John", "age" : 23};
var name = object.name;
var age = object.age;
```

在 ES6 中，我们可以通过一行语句来完成这个操作，使用对象展开赋值：

```js
let object = {"name" : "John", "age" : 23};
let name, age;

({name, age} = object); //object destructuring assignment syntax
```

在对象展开语句的左侧，我们需要放置想要将对象属性值赋给变量的变量，使用与对象字面量类似的语法。在右侧，我们需要放置一个包含我们想要提取的属性值的对象，并最终使用`( )`标记来关闭语句。

在这里，变量名必须与对象属性名相同。如果你想分配不同的变量名，可以这样做：

```js
let object = {"name" : "John", "age" : 23};
let x, y;

({name: x, age: y} = object);
```

之前的代码可以通过这种方式进一步缩短：

```js
let {name: x, age: y} = {"name" : "John", "age" : 23};
```

在这里，我们是在同一行上创建变量和对象。由于我们是在同一语句上创建变量，因此不需要使用`( )`标记来关闭语句。

### 变量的默认值

如果在展开对象时对象属性是`undefined`，你也可以为变量提供默认值。以下是一个演示此功能的示例：

```js
let {a, b, c = 3} = {a: "1", b: "2"};
console.log(c); //Output "3"
```

### 展开计算属性名

一些属性名是使用表达式动态构建的。在这种情况下，为了提取属性值，我们可以使用`[ ]`标记来提供一个属性名表达式。以下是一个示例：

```js
let {["first"+"Name"]: x} = { firstName: "Eden" };
console.log(x); //Output "Eden"
```

### 展开嵌套对象

我们还可以从嵌套对象中提取属性值，即对象内的对象。以下是一个演示此功能的示例：

```js
var {name, otherInfo: {age}} = {name: "Eden", otherInfo: {age: 23}};
console.log(name, age); //Eden 23
```

### 将对象展开赋值用作参数

就像数组展开赋值一样，我们也可以将对象展开赋值用作函数参数。以下是一个演示此功能的示例：

```js
functionmyFunction({name = 'Eden', age = 23, profession = "Designer"} = {})
{
  console.log(name, age, profession); //Output "John 23 Designer"
}

myFunction({name: "John", age: 23});
```

这里，我们传递了一个空对象作为默认参数值，如果将 `undefined` 作为函数参数传递，它将用作默认对象。

# 箭头函数

ES6 提供了一种使用 `=>` 操作符创建函数的新方法。这些函数被称为 **箭头** 函数。这种方法具有更短的语法，箭头函数是无名函数。

这里有一个示例，展示了如何创建一个箭头函数：

```js
let circleArea = (pi, r) => {
  let area = pi * r * r;
  return area;
}

let result = circleArea(3.14, 3);

console.log(result); //Output "28.26"
```

在这里，`circleArea` 是一个变量，引用了匿名箭头函数。前面的代码与以下 ES5 代码类似：

```js
Var circleArea = function(pi, r) {
  var area = pi * r * r;
  return area;
}

var result = circleArea(3.14, 3);

console.log(result); //Output "28.26"
```

如果箭头函数只包含一个语句，那么你不需要使用 `{}` 括号来包裹代码。以下是一个示例：

```js
let circleArea = (pi, r) => pi * r * r;
let result = circleArea(3.14, 3);

console.log(result); //Output "28.26"
```

当没有使用 `{}` 括号时，则语句体的值会自动返回。

## 箭头函数中 "this" 的值

在箭头函数中，`this` 关键字的值与封闭作用域（全局或函数作用域，无论箭头函数在哪里定义）中 `this` 关键字的值相同，而不是指向上下文对象（即函数作为属性所在的那个对象），这是传统函数中 `this` 的值。

考虑以下示例，以了解传统函数和箭头函数的 `this` 值之间的差异：

```js
var object = {
  f1: function(){
    console.log(this);
    var f2 = function(){ console.log(this); }
    f2();
    setTimeout(f2, 1000);
  }
}

object.f1();
```

输出如下：

```js
Object
Window
Window
```

在这里，`f1` 函数内部的 `this` 指向 `object`，因为 `f1` 是它的属性。`f2` 函数内部的 `this` 指向 `window` 对象，因为 `f2` 是 `window` 对象的属性。

但是在箭头函数中，`this` 的行为有所不同。让我们将前面的代码中的传统函数替换为箭头函数，并查看 `this` 的值：

```js
var object = {
  f1: () => {
    console.log(this);
    var f2 = () => { console.log(this); }
    f2();
    setTimeout(f2, 1000);
  }
}

object.f1();
```

输出如下：

```js
Window
Window
Window
```

在这里，`f1` 函数内部的 `this` 复制了全局作用域的 `this` 值，因为 `f1` 位于全局作用域。`f2` 函数内部的 `this` 复制了 `f1` 的 `this` 值，因为 `f2` 位于 `f1` 作用域。

## 箭头函数与传统函数之间的其他差异

箭头函数不能用作对象构造函数，也就是说，不能对它们应用 `new` 操作符。

除了语法、`this` 值和 `new` 操作符之外，箭头函数与传统函数之间没有其他差异，也就是说，它们都是 `Function` 构造函数的实例。

# 增强的对象字面量

ES6 为 `{}` 对象字面量添加了一些基于语法的扩展，用于创建属性。让我们看看它们：

## 定义属性

ES6 提供了一种更短的语法来将对象属性分配给具有相同名称的变量值。

在 ES5 中，你一直是这样做：

```js
var x = 1, y = 2;
var object = {
  x: x,
  y: y
};

console.log(object.x); //output "1"
```

在 ES6 中，你可以这样操作：

```js
let x = 1, y = 2;
let object = { x, y };

console.log(object.x); //output "1"
```

## 定义方法

ES6 提供了一种新的语法来定义对象上的方法。以下是一个演示新语法的示例：

```js
let object = {
  myFunction(){
    console.log("Hello World!!!"); //Output "Hello World!!!"
  }
}

object.myFunction();
```

这种简洁的函数允许在其中使用 `super`，而对象的传统方法不允许使用 `super`。我们将在本书的后面了解更多关于它的内容。

## 计算属性名称

在运行时评估的属性名称被称为**计算属性名称**。通常，表达式会被解析以动态地找到属性名称。

在 ES5 中，计算属性是这样定义的：

```js
var object = {};

object["first"+"Name"] = "Eden";//"firstName" is the property name

//extract
console.log(object["first"+"Name"]); //Output "Eden"
```

在这里，在创建对象之后，我们将属性附加到对象上。但在 ES6 中，我们可以在创建对象的同时添加具有计算名称的属性。以下是一个示例：

```js
let object = {
  ["first" + "Name"]: "Eden",
};

//extract
console.log(object["first" + "Name"]); //Output "Eden"
```

# 摘要

在本章中，我们学习了变量的作用域、只读变量、将数组拆分为单个值、向函数传递不定参数、从对象和数组中提取数据、箭头函数以及创建对象属性的新语法。

在下一章中，我们将学习内置对象和符号，并发现 ES6 为字符串、数组和对象添加的新属性。
