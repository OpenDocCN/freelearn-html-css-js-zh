# 第九章。JavaScript 表达式、运算符、语句和数组

JavaScript 是最常用的网络编程语言，并且在全世界开发者中非常受欢迎。本章将涵盖该语言中使用的几乎所有表达式、运算符、语句和数组。

# 表达式

用来解析值的有效代码单元被称为**表达式**。它是一组字面量、运算符、变量和表达式，用于评估一个值。这个值可以是字符串或任何逻辑值。表达式产生一个值，可以在需要值的地方写入。表达式有两种类型：

+   将值分配给变量的表达式

+   具有值的表达式

考虑以下示例：

```js
var A = 2;
```

在前面的例子中，一个值被分配给变量`A`，为了将这个值分配给变量，我们使用赋值运算符。现在考虑另一个例子：

```js
2+3;
```

在这个例子中，没有值被分配给变量；然而，它评估`2+3`的结果为`5`。在这个表达式中使用的运算符被称为**加法运算符**。

JavaScript 中的表达式可以大致分为三种类型：

+   **算术**：这些评估数字。这意味着这些表达式在值之间执行数学计算。

+   **逻辑**：这些用于评估并以`true`或`false`的形式给出结果。

+   **字符串**：这些用于评估字符串。

### 注意

另一种类型的表达式称为**条件表达式**。这些表达式通常用于`if-else`和`loop`条件中。这些条件表达式以`true`或`false`的形式评估结果，并且只有两个值`True`和`False`。如果第一个条件是`true`，则首先评估，否则将评估第二个，例如：

```js
int age = 20;
var flag;
if (age<18) {
  flag=true;
}
else {
  flag=false;
}
```

## 主要表达式

在 JavaScript 中，主要表达式是基本关键字，它们是特殊对象、标识符和字面量，不需要进一步评估即可解析它们的值。它们也可能是另一个表达式的结果，该表达式被括号包围。

例如，`this`和`function`是关键字，这些是 JavaScript 中的主要表达式。`this`关键字在进入执行上下文时始终返回一个值。标识符也是主要表达式，它们指代一个函数或对象。

有四种字面量类型，作为主要表达式：

+   **布尔字面量**：这些包含`1`/`0`或`true`/`false`

+   **空字面量**：这些包含一个`null`值

+   **未定义字面量**：这些包含任何类型的数据类型

+   **字符串字面量**：这些包含字符字符串

## 对象初始化器

在 JavaScript 中，对象初始化器用于初始化对象。它是一个称为**对象初始化器**的表达式。JavaScript 中的每个对象都是一个具有类型的实体。每个对象都与其一些属性相关联。与对象相关联的函数被称为**方法**。对象属性基本上是 JavaScript 变量。与对象相关联的属性会告诉你关于对象特性的信息。

对象属性可以属于原始数据类型，如`int`或其他数据类型，如`char`。对象也可以有一个空属性。我们可以创建一个具有空属性的对象，如下所示：

```js
var xyz={};
```

在这些大括号中，你可以轻松快速地创建对象。这些是逗号分隔的名称-值对。对象初始化器基本上用于创建一个新的对象。这些对象初始化器被称为**对象字面量**。

### 小贴士

在 JavaScript 中，属性名和对象名是区分大小写的，所以当你创建一个对象时，应该记住这一点。

对象初始化器使用字面量表示法创建一个对象，例如：

```js
var studentInfo =  {  age:24, Name:"Ali" };
```

这在第八章中有更详细的介绍，*JavaScript 面向对象编程*。

## 函数定义表达式

JavaScript 中的函数使用函数关键字定义。在这些函数上定义表达式。它们用于声明函数或函数表达式。你可以在脚本中的任何地方定义函数。函数使用表达式来定义它们的原型。

在 JavaScript 中定义函数有很多方法，例如，我们有以下几种：

+   函数声明

+   函数表达式

### 函数声明

当浏览器考虑执行脚本时，函数声明是一个预执行阶段。函数可以在你的代码的任何地方声明：

```js
function greetings() {
  alert("Happy New Year");
}
greetings();
```

### 函数表达式

当一个函数的形式是一个表达式时，它被称为**函数表达式**。这个表达式可以是一个大的表达式，这意味着函数声明需要多个表达式。使用函数表达式赋值的函数可以是有名字的，也可以是没有名字的。这通常被称为匿名函数。它是一个一等值，这意味着它允许将值作为参数传递给函数。

考虑以下示例：

```js
var abc=function() {
  return 1;
};

 var xyz=function foo() {
  return 2;
};
```

函数值也可以赋给另一个函数参数。这样，函数值作为值传递传递给另一个函数参数。

```js
var f = function f ( function foo() );
```

JavaScript 中的函数也是一个常规值。如果你不希望你的变量在全局作用域中设置，那么将这些变量放入一个函数中。我们这样做是因为设置全局变量并不总是一个好的选择，因为它可以被程序中的任何函数在任何地方访问，这可能会改变变量的值并覆盖原本应该使用的原始值。函数通常在括号内编写，因为 JavaScript 允许表达式全部在同一空间中。

## 属性访问表达式

在 JavaScript 中，要访问对象或数组元素的值，我们使用属性访问操作。从这些表达式访问值有两种编写属性访问表达式的写法：

+   中括号表示法

+   点表示法

### 中括号表示法

中括号表示法也称为数组表示法。编写中括号表示法的语法如下：

```js
get=abc[xyz];
abc[xyz]=set;
get = abc [1]; // get value at array abc index 1 
```

### 点表示法

点表示法也称为对象表示法。编写点表示法的语法如下：

```js
 document.write ("hello world");
```

## 调用表达式

在 JavaScript 中，调用表达式用于执行或调用函数或表达式。当函数调用开始时，它首先评估其表达式，然后评估其参数。以下是调用表达式上的两个目标：

+   调用目标

+   可选参数列表

### 调用目标

一个调用目标后跟一个开括号，然后是一个目标列表，最后是一个闭括号目标列表，必须被分类为方法或对象。有两种类型的参数列表：

+   位置参数

+   命名参数

位置参数是表达式，命名参数是标识符。以下是一个调用表达式的示例：

```js
Function.getvalue(5);
Math.max(2,3);
```

命名参数是带有其名称的参数。

### 可选参数列表

变量被分配一个作为函数参数传递的值。如果没有传递值，则变量将被分配一个默认值：

```js
function add(a, b) {
  b = typeof b !== 'undefined' ?  b : 0; // if b not defined set it to 0
  return a+b;
}

add(10); // returns 10
```

## 对象创建表达式

在 JavaScript 中，此表达式用于使用构造函数方法创建对象。此方法用于初始化对象的属性。此方法首先创建新对象，然后使用 JavaScript 中的对象初始化方法初始化对象。用于初始化对象的构造函数不返回任何值，此值成为对象的值。考虑以下示例：

```js
New myObj();
```

## 评估表达式

评估表达式用于评估一个字符串。这是 JavaScript 中全局对象的一个属性。当它评估一个字符串时，它会给出一个值作为输出。

`Eval()`函数在 JavaScript 中用于获取值。在这个函数中，我们执行以下操作：

+   作为输入，我们传递一个参数

+   如果它没有返回值，那么意味着代码中存在错误，并且它会抛出异常

    ```js
    var x = 10;
    console.log(eval("x * 5")); //50
    ```

你有两种方式可以调用`eval()`函数：

+   直接：直接调用名为 'eval' 的函数。直接 eval() 在其局部作用域中执行代码。例如：

    ```js
    var abc = 'global scope';
        function directEval() {
            'use strict';
            var abc = 'local scope';
            console.log(eval('abc')); // local scope
        }
    ```

+   间接：通过调用 call() 函数、window 的方法或通过不同的名称存储并从那里调用等方式调用它。间接 eval() 在其全局作用域中执行代码。例如：

    ```js
    var abc = 'global scope';
        function indirectEval() {
            'use strict';
            var abc = 'local scope';
            // eval via call()
            console.log(eval.call(null, 'abc')); // global scope
            console.log(window.eval('abc')); // global scope
            console.log((1, eval)('abc')); // global scope
            // eval with a different name
            var abceval = eval;
            console.log(abceval('abc')); // global scope
            var obj = { eval: eval };
            console.log(obj.eval('abc')); // global scope
        }
    ```

# 运算符

在编程语言中，运算符是对操作数执行的操作。基本上，运算符是用于执行各种操作的符号。考虑以下示例：

```js
9 + 11 = 20
```

在前面的示例中，`9` 和 `11` 是操作数，`+` 是加法运算符。

## 概述

与其他语言一样，在 JavaScript 中，存在执行操作的各种运算符，例如加法、乘法、减法等。

在 JavaScript 中，存在不同的运算符，例如：

+   逻辑运算符

+   位运算符

+   条件运算符

+   算术运算符

+   赋值运算符

### 二元运算符

如其名所示，二元运算符需要两个操作数；例如：

```js
A + B
```

在这里，`A` 和 `B` 是操作数，`+` 是执行它们之间加法运算的运算符。

### 一元运算符

在编程语言中，一元运算符只有一个操作数和与之相关的运算符，例如：

```js
A++;
```

这也可以写成：

```js
++A;
```

### 注意

`A++` 是一个后递增运算符。它将首先评估 `A`，然后递增它；而 `++A` 是一个前递增运算符。它递增 `A` 的值，然后评估表达式。

### 三元运算符

三元运算符需要三个操作数来执行不同的操作；例如：

```js
<condition> ? <true>: <false>;
```

在前面的语句中，`?` 是三元运算符。

## 算术运算符

在编程语言中，算术运算符用于对值和变量执行算术运算。基本的算术运算如下：

+   `+`：加法

+   `-`：减法

+   `*`：乘法

+   `/`：除法

+   `%`：取模运算符

+   `++`：递增运算符

+   `--`：递减运算符

这些运算符对数值操作数执行操作，并返回一个数值结果。

在 JavaScript 中，运算符与其他编程语言中的运算符相同；例如，如果你想对变量执行除法运算，那么你应该写成以下形式：

```js
var a=2;
a=(2+a)*1/a;// a=2
console.log(a); // prints 2
```

### 注意

在 JavaScript 中，算术运算符是最有用且功能强大的运算符。

### `+` 运算符

`+` 运算符执行两个或更多数字的加法。

#### 返回值

`+` 运算符返回数字的加法。

#### 参数

`+` 运算符没有参数。

#### 描述

`+` 运算符用于将两个数字或两个存储数字的变量相加。

例如：

```js
var a = 3;
var b = 4;
var c = a + b;
Document.Write("The value of c is " + c);
Document.Write(3+1);
```

这个输出的结果将是：

```js
The value of c is 7
4
```

### 注意

此运算符还可以用于连接或连接两个字符串；例如：

```js
document.write("Apple "+"Banana");
```

输出结果如下：

```js
Apple Banana
```

### `-` 运算符

`-` 运算符在两个数字之间执行减法运算。

#### 返回值

`-` 运算符的结果返回两个数字之间的减法。

#### 参数

`-` 运算符没有参数。

#### 描述

`-` 运算符用于减去两个数字或存储数字的两个变量。

考虑以下示例：

```js
var a = 6;
var b = 4;
var c = a - b;
Document.Write("The value of c is " + c);
Document.Write(3-1);
```

此输出的结果如下：

```js
The value of c is 2
2
```

### `*` 运算符

`*` 运算符执行两个或更多数字之间的乘法。

#### 返回值

数字之间乘法的结果。

#### 参数

没有参数。

#### 描述

此运算符用于乘以两个或更多数字或存储数字的两个或更多变量。

考虑以下示例：

```js
var a = 3;
var b = 4;
var c = a * b;
Document.Write("The value of c is " + c);
Document.Write(3*1);
```

此输出的结果如下：

```js
The value of c is 12
3
```

### `/` 运算符

`/` 运算符在两个数字之间执行除法。

#### 返回值

数字之间除法的结果。

#### 参数

没有参数。

#### 描述

这也被称为 **余数运算符**，因为返回的值不是绝对值（即，它保留第一个操作数的符号，而不是第二个操作数的符号）。

考虑以下示例：

```js
var a = 5;
var b = 2;
var c = a % b;
Document.Write("The value of c is " + c);
Document.Write(10%3);
```

输出结果如下：

```js
The value of c is 1
1
```

### % 运算符

`%` 模数运算符用于计算余数。

#### 返回值

数字之间乘法的结果。

#### 参数

没有参数。

#### 描述

此运算符用于乘以两个或更多数字或存储数字的两个或更多变量。

考虑以下示例：

```js
var a = 3;
var b = 4;
var c = a * b;
Document.Write("The value of c is " + c);
Document.Write(3*1);
```

输出结果如下：

```js
The value of c is 12
3
```

### ++ 运算符

`++` 运算符用于增加一个值。

#### 返回值

结果是递增的值。

#### 参数

没有参数。

#### 描述

增量。增加值并将值存储在自身中。

考虑以下示例：

```js
I = 0; // I contains 0
I++; // this line is equivalent to I = I + 1 
```

此输出的结果如下：

```js
I = 1
```

### -- 运算符

`--` 运算符用于递减一个值。

#### 返回值

结果是递减的值。

#### 参数

没有参数。

#### 描述

`--` 运算符递减值并将值存储在自身中。

例如：

```js
I = 0; // I contains 0
I--; // this line is equivalent to I = I - 1 
```

输出结果如下：

```js
I = - 1
```

## 逻辑运算符

在编程语言中，逻辑运算符是布尔运算符。这些运算符在布尔逻辑上工作。像其他语言一样，JavaScript 在布尔评估上工作。这些布尔运算符总是返回布尔值。任何逻辑运算符都有两种可能的结果：

+   `True`

+   `False`

以下是在编程语言以及 JavaScript 中使用的逻辑运算符：

+   逻辑与 (`&&`)

+   逻辑或(`||`)

+   逻辑非(`!`)

### 注意

算术运算符比逻辑运算符有更高的优先级。

假设，如果有一个表达式，并且其中包含逻辑和算术运算符，那么由于它们的优先级较高，算术运算符将首先被评估。逻辑运算符从左到右工作。

逻辑运算适用于所有内容。一个非布尔值在评估后可以产生一个布尔值。

### && 运算符

`&&` 运算符是英语单词 "and" 的翻译。例如，如果我们必须写苹果和橙子，我们写 *apples && oranges*。

#### 返回值

这返回操作数上最具体的价值；如果使用非布尔值，则返回非布尔值。

#### 参数

我们通常用 `&&` 符号分隔要评估的表达式或条件。

#### 描述

这被称为 **AND** 操作符。

这里是 `&&` 操作符的一个示例：

```js
var a= true && true; // it will be true
var b=true && false;//it will be false
```

### `||` 操作符

这个操作符是英语单词 "or" 的翻译。例如，如果我们必须写苹果或橙子，我们写 *apples || oranges*。

#### 返回

这返回操作数上最具体的价值；如果使用非布尔值，则返回非布尔值。

#### 参数

我们通常用 `||` 符号分隔要评估的表达式或条件。

#### 描述

这被称为 **OR** 操作符。

这里是 `||` 操作符的一个示例：

```js
var a= true  ||  true; // it will be true
var b=true  ||  false;// it will be true
```

### `!` 操作符

这是 `NOT` 操作符。`NOT` 操作符评估值是否不相等。例如：

```js
if(abc != 0){
 //if abc is not equal to 0 run this block
}else{
 //if abc = 0 then run this block
}
```

#### 返回

布尔值。

#### 参数

`null`

#### 描述

这被称为 **NOT** 操作符。

这里是 `!` 操作符的一个示例：

```js
var a=!false //it will print true
var b=!true //it will print false
```

### 注意

在 JavaScript 中，这些逻辑操作符用于条件语句和 `while` 循环中。

## 赋值操作符

在 JavaScript 中，赋值操作符用于将值赋给变量。赋值操作符 `=` 将其右侧操作数的值赋给其左侧的操作数。例如，`abc = xyz`，在这里 `xyz` 是右侧操作数，其值被赋给 `abc`，即左侧操作数。赋值操作符可以将单个变量的值赋给多个变量。在以下各节中，我们将讨论 JavaScript 中的赋值操作符。

### `=` 操作符

`=` 操作符用于将值赋给变量。

#### 返回

没有参数，但它用于两个或多个操作数之间。

#### 参数

没有参数。

#### 描述

这是所有赋值操作符中最简单的一个，并且被广泛使用。它将右侧操作数的值赋给左侧操作数。

考虑以下示例：

```js
var x=2;
console.log("The value of x is " + x);
```

这个输出的结果将是：

```js
The value of x is 2
```

### `+=` 操作符

这个操作符被称为加法赋值操作符。

#### 返回

这将值加到变量上，并返回加法的结果。

#### 参数

没有参数。

#### 描述

加法赋值操作符用于在单个操作中将值加到变量上，并将值赋给另一个或相同的变量。

考虑以下示例：

```js
var abc = 2;
abc += 10;
console.log(abc); //12
```

这是 `abc = abc + 10` 的简写。

### `-=` 操作符

这个操作符被称为减法赋值操作符。

#### 返回

这个操作符从变量中减去给定的值，并返回减法的结果。

#### 参数

没有参数。

#### 描述

`-=` 操作符用于从变量中减去一个值，并在同一操作中将给定的值赋给另一个或相同的变量。

考虑以下示例：

```js
var abc = 10;
abc -= 2;
console.log(abc); //8
```

这是 `abc = abc - 2` 的简写。

### `*=` 操作符

这个操作符被称为乘法赋值操作符。

#### 返回

这将变量和值相乘，并返回乘法的结果。

#### 参数

没有参数。

#### 描述

此操作符用于将一个值与变量相乘，并在同一操作中将结果赋给另一个或相同的变量。

考虑以下示例：

```js
var abc = 5;
abc *= 5;
console.log(abc); //25
```

这是一种简写形式，相当于 `abc = abc * 5`。

### /= 操作符

这个操作符被称为除法赋值操作符。

#### 返回值

它将变量除以右操作数的值，并将结果赋给变量。

#### 参数

没有参数。

#### 描述

此操作符用于将变量除以值，并将结果赋给变量。

考虑以下示例：

```js
var abc = 5;
abc /= 5;
console.log(abc); //1
```

这是一种简写形式，相当于 `abc = abc / 5`。

### %= 操作符

`%=` 操作符也称为余数赋值操作符。

#### 参数

%= 操作符接受一个空参数。

#### 返回值

%= 操作符返回余数。

#### 描述

它将变量除以右操作数的值，并将余数赋给变量。例如：

```js
var abc = 10;
abc %= 3;
console.log(abc); //1
```

这是一种简写形式，相当于 `abc = abc % 3`。

### 注意

这是一个实验性技术，是 ECMAScript 2016（ES7）提案的一部分。

## 幂赋值（**=**）

此操作符返回左操作数幂的值，其幂为第二个操作数的值。

```js
var abc = 5;
abc **= 2;
console.log(abc); //25
```

这是一种简写形式，相当于 `abc = abc ** 2`。

## 关系操作符

在编程语言中，关系操作符也称为比较操作符。这些操作符显示两个值的相对顺序。以下各节将介绍这些比较操作符。

### < 操作符

`<`（小于）操作符用于比较表达式两边的值，并检查操作符左边的值是否小于操作符右边的值。

#### 返回值

如果前面的条件为真，则表达式返回 `true`，否则返回 `false`。

#### 参数

在这里，两个操作数必须是数字或两者都必须是字符串。

#### 描述

这也被称为**小于**操作符。

下面是一个使用此操作符的简单示例：

```js
if(2<3) { //returns true
  //block of code
}
```

### <= 操作符

`<=` 操作符用于比较表达式两边的值，并检查操作符左边的值是否小于或等于操作符右边的值。

#### 返回值

如果前面的条件为真，则它返回布尔值 `true`，否则返回 `false`。

##### 参数

在这里，两个操作数必须是数字或两者都必须是字符串。

#### 描述

这也被称为**小于等于**操作符。

下面是一个使用此操作符的简单示例：

```js
if(2 <= 3) { //returns true
  //block of code
}
```

### > 操作符

`>` 操作符用于比较表达式两边的值，并检查操作符左边的值是否大于操作符右边的值。

#### 返回值

如果前面的条件为真，则它返回布尔值 `true`，否则返回 `false`。

#### 参数

在这里，两个操作数必须是数字或两者都必须是字符串。

#### 描述

这也被称为**大于**操作符。

使用此操作符的一个简单示例如下：

```js
if(2>3) { //returns false
  //block of code
}
```

### >= 操作符

`>=` 操作符用于比较表达式两边的值，并检查操作符左边的值是否大于或等于操作符右边的值。

#### 返回值

如果前面的条件为真，则返回布尔值 `true`，否则返回 `false`。

#### 参数

这里，两个操作数必须是数字或两者都必须是字符串。

#### 描述

这也被称为**大于或等于**操作符。

使用此操作符的一个简单示例如下：

```js
if(3 >= 3) { //returns true
  //block of code
}
```

### != 操作符

`!=` 操作符用于比较表达式两边的值，并检查操作符左边的值是否不等于操作符右边的值。

#### 返回值

如果前面的条件为真，则返回布尔值 `true`，否则返回 `false`。

#### 参数

这里，两个操作数必须是数字或两者都必须是字符串。

#### 描述

这也被称为**不等于**操作符。

使用此操作符的一个简单示例如下：

```js
if(2!=3) { //returns true
  //block of code
}
```

### == 操作符

`==` 操作符用于比较表达式两边的值，并检查操作符左边的值是否等于操作符右边的值。

#### 返回值

如果前面的条件为真，则返回布尔值 `true`，否则返回 `false`。

#### 参数

这里，两个操作数必须是数字或两者都必须是字符串。

#### 描述

这也被称为**等于**操作符。

使用此操作符的一个简单示例如下：

```js
if(2==3) { //returns false
  //block of code
}
```

### === 操作符

`===` 操作符用于比较表达式两边的值，并查看操作符左边的值是否等于操作符右边的值，同时也要检查两个操作数是否为同一数据类型。

#### 返回值

如果前面的条件为真，则返回布尔值 `true`，否则返回 `false`。

#### 参数

`===` 操作符接受参数 `null`。

#### 描述

这也被称为**值相等且类型相等**操作符。

使用此操作符的一个简单示例如下：

```js
if(3 === 3) { //returns true
  //block of code
}
if(3 === "3") { //returns false
  //block of code
}
```

# 语句

JavaScript 在一组语句上工作。这些语句具有适当的语法，语法取决于语句包含的内容。一个语句可以包含多行。基本上，一个语句是一组给计算机浏览器执行 JavaScript 代码的指令。一个语句可以用分号结束，并返回一个值。一个语句可以是一个对特定动作的调用，例如：

```js
Document.Write("hello world");
```

前面的语句调用了内置的 `Write()` 函数，并在屏幕上打印了消息 **hello world**。

我们可以在一行中写多个语句，用分号分隔；例如：

```js
var a=20; var b=2;var c=a*b; document.write(c);
```

语句也可以用换行符结束，例如：

```js
var a=20;
Document.Write(a);
```

JavaScript 中的每个语句都按照 JavaScript 程序中给出的指令顺序依次执行。

## 表达式语句

在 JavaScript 中，语句和表达式之间有一个区别。每当在脚本或代码中期望一个值时，表达式通过语句产生这个值。

表达式由字面量、变量、运算符和方法组成。返回值的数据类型取决于表达式中使用的变量。我们可以根据数据类型从一组较小的表达式中创建一个复合表达式。

```js
x = Math.PI;
  cx = Math.cos(x);
  alert("cos(" + x + ") = " + cx);
```

语句基本上是在脚本中执行的动作，例如循环和`if-else`在 JavaScript 中都是语句。无论 JavaScript 期望一个语句还是表达式，都可以称为表达式语句。它既满足语句的功能，也满足表达式的功能。但这是不可逆的。我们不能在 JavaScript 期望表达式的地方写一个语句。例如，if 语句不能用作函数的参数。

为了防止这种情况发生，JavaScript 不允许我们将函数表达式和对象字面量用作语句，也就是说，**表达式语句**不应该以下列方式开始：

+   大括号

+   关键字`function`

## 复合空语句

复合语句是一组用大括号括起来的语句，所有这些语句都由分号分隔。它将程序中的多个语句组合成一个单一的语句。复合语句也称为**块语句**。

这种类型的语句通常与条件语句如`if`、`else`、`for`、`while`循环等一起使用。考虑以下例子：

```js
if(x<10) //  if x is less than 10 then x is incremented {
  x++;
}
```

空语句是复合语句的反面。空语句中没有语句，只有一个分号。这个分号意味着不会执行任何语句。通常，空语句用于需要在代码中添加注释的情况。考虑以下例子：

```js
for(i=0; i<10; i++)
{} /*  Empty  */
```

## 声明语句

这个语句用于在 JavaScript 中声明变量或函数。

### `function`

在 JavaScript 中，`function`关键字用于在程序中声明一个函数。

这里有一个简单的例子，其中声明了`myFunction`函数：

```js
function myFunction() {
  //block of code
}
```

### `var`

在 JavaScript 中，要声明一个变量，我们使用`var`关键字。在使用变量之前，我们需要声明它。

考虑以下例子：

```js
var a =5;
var b = 23.5;
var c = "Good morning";
```

下面是一个同时使用`var`和`function`的例子：

```js
var a=2;
var b=3;
function Addition(x,y) {
  return x+y;
}
var c= Addition(a,b)
Document.Write("The value of c is " + c)
```

前述代码的输出将如下所示：

```js
The value of c is 5
```

## 条件语句

在 JavaScript 中，当我们想要对不同的语句执行不同的操作时，我们使用条件语句。这些是一组根据不同条件执行不同操作的命令。在 JavaScript 中，有三种类型的条件语句，如下所示：

+   `if`

+   `else`

+   `switch`

基本上，这些语句显示了代码的流程。条件语句控制程序的流程，即根据哪个条件执行哪个操作。

考虑一个例子，假设你有一个电子商务网站，并且你想要在产品有特别优惠时显示一个优惠标签。在这种情况下，你可以在条件语句中放置适当的代码。

### If 语句

在编程语言中，`if` 语句是一个控制语句。`if` 语句有两个重要的部分：

+   条件

+   执行动作的代码

在 JavaScript 和其他语言中，`if` 语句根据变量和数据类型做出决策。例如，如果你编写一个脚本要求在生日时通知你，那么当时间到来时，将显示消息 "**今天是你的生日**"。

只有当条件为 `true` 时，`if` 语句才会执行。如果表达式是 `false`，则不会执行。

#### 语法

这里是 JavaScript 中 `if` 语句的语法：

```js
if(condition) {
  // block of code
}
```

#### 示例

例如，在 JavaScript 中，我们可以这样编写一个 `if` 语句代码：

```js
var cAge=25;
If (cAge>18) {
  document.write ("You must have CNIC");
}// if age is above 18 it displays that you must have a CNIC  
```

### Else if 语句

在 JavaScript 中，可能会有一些情况，我们需要根据不同的可能性做出决定。`else if` 语句只有在前面的语句为 `false` 且其条件为 `true` 时才会执行。`else if` 语句有两个主要点，如下：

+   在任何 `else if` 语句之前都应该有一个 `if` 语句

+   你可以使用任意数量的 `else if` 语句，但最后必须有一个 `else` 语句。

#### 语法

这里是 JavaScript 中简单 `if` 语句的语法：

```js
if(condition 1) {
  // block of code
}
else if(condition 2) {
  //block of code
}
else {
  //  block of code
}
```

在前面的语法中，可以检查两个条件是否有效。

这里是 JavaScript 中稍微复杂一点的 `else if` 语句的语法：

```js
if(condition 1) {
  // block of code
}
else if(condition 2) {
  // block of code
}
else if(condition 3) {
  // block of code
}
else {
  // block of code
}
```

在前面的语法中，可以检查三个条件是否有效。

#### 示例

例如，如果你有一个网页，并且你想检查谁在访问该网页。你将在 `Else if` 语句的脚本中放置两个自定义消息。你将这样编写它们：

```js
var person="Manager";
If(person=="Admin") {
  Document.Write("I am visiting it");
}
Else if(person=="Manager") {
  Document.Write("I am accessing it");
}
Else {
  document.write("hello kitty");
}
```

`Else if` 语句是 JavaScript 中最先进的语句形式之一。它在检查所有 `if` 条件后做出决定。它是一种复杂的语句类型，其中每个 `if` 都是 `else` 语句的一部分。所有条件都必须在 `else if` 关键字后面的括号内为 `true`。如果任何条件为 `false`，则执行 `else` 部分。

### Switch 语句

在编程语言中，`switch`语句的执行仅取决于表达式的值。有多个情况列表，对这些情况进行检查。检查是顺序进行的；JavaScript 解释器检查第一个情况及其条件，以检查它是否为`true`。同样，对所有的`case`语句进行检查，以检查它们的条件是否为`true`，并找到`break`语句。当解释器遇到`break`语句时，它退出`switch`循环。`break`语句的目的是确保一旦执行了一个`case`，`switch`语句就返回控制权给调用函数，解释器不会将控制权传递给下一个适用的`case`。

#### 语法

这里是 JavaScript 中简单`switch`语句的语法：

```js
switch(condition/expression)
  case expression 1:
    //block of code
  break;
  case expression 2:
    //block of code
  break;

  case expression 3:
    //block of code
  break;

  default:
    //block of code
  break;
```

#### 示例

下面是一个有效的`switch`语句的示例：

```js
Switch (personName)
  case "Jack":
    alert("my name is Jack");
  Break;
  case "Ahmed":
    alert("my name is Ahmed");
  break;
  default:
    alert("my name is Talha");
  break;
```

如果前面的代码没有找到匹配的情况，则执行默认语句。基本上，`switch`语句是一个表达式，它根据某些条件评估不同的语句以执行某些代码。

## 循环

在编程语言中，循环用于根据条件执行代码块多次。例如，如果你有一个语句并且想要反复执行它，你只需将其放入循环中。循环基本上用于重复性任务。

另一个循环的特定例子是，如果你想遍历一个数组以找到特定的数字，你必须遍历数组元素。每次迭代将获取下一个数组索引。

JavaScript 中有四种类型的循环，如下所示：

+   `for`

+   `while`

+   `do-WHILE`

+   `for-in`

对于重复性任务，我们将一系列指令放入循环中。为了控制循环执行的次数，我们使用循环中的控制变量来增加或减少迭代器或计数器的值，这将重复执行代码块。你还可以在循环中使用`break`和`continue`语句。

### 循环

`for`循环用于循环遍历或执行代码块，直到条件返回`false`结果。在`for`循环中，我们说明脚本应该执行多少次。例如，你可能想要执行你的脚本 12 次。

#### 语法

JavaScript 中的`for`循环与其他语言（如 C、C++和 Python）中的`for`循环相同。`for`循环的语法有三个重要部分：

+   **初始化**：这是循环初始化计数器变量的地方。这是循环开始执行时的第一条语句。

+   **条件**：这是一个在每次循环迭代之前要评估的表达式。如果条件满足且为`true`，我们进入循环；否则，如果条件不满足且返回`false`，则退出`for`循环。

+   **增量/减量**：这用于迭代和增加或减少计数器变量的值。这是一个更新语句，其中执行增量或减量。

    ```js
    for (initialization; condition; increment / decrement) {
      //code block
    }
    ```

#### 示例

考虑以下 JavaScript 中基本计数器代码的示例：

```js
var i; // declares the variable i
for(i=1; i < 5 ; i++) {
  Document.Write("basic counter:"+i);
  Document.Write("<br/>");
}
```

输出将如下所示：

```js
Basic counter:1
Basic counter:2
Basic counter:3
Basic counter:4
```

### `while`循环

这也是 JavaScript 中常用的一个循环。`while`循环在条件为`true`的情况下重复执行一系列语句，直到条件变为`false`。当条件变为`false`时，循环退出。

#### 语法

下面是 JavaScript 中`while`循环的语法：

```js
while (condition expression) {
  // code block
}
```

`while`循环有两个部分：

+   条件语句

+   用大括号编写的`while`循环代码

为了使循环工作，必须满足`while`循环的条件。如果条件中断，则循环中断。

#### 示例

下面是`while`循环的一个示例，它将打印出从`1`到`4`的数字：

```js
var count= 0;
While(count<5) {
  Document.Write("count");
  Document.Write("<br/>");
  count++;
}
```

输出将如下所示：

```js
1
2
3
4
```

### `do while`循环

`do while`循环也称为**后测试循环**，因为它首先执行语句，然后检查条件。如果条件为`true`，则进入循环；如果条件为`false`，则跳出循环。

#### 语法

下面是 JavaScript 中`do while`循环的语法：

```js
Do {
  //statement
}
While(condition);
```

`do while`循环有两个重要部分：

+   语句

+   条件

`do while`循环与`while`循环类似，但它是在循环的末尾检查条件。`do while`循环至少执行一次指定的语句，然后再检查条件。

#### 示例

下面是一个`do while`循环的示例，它会打印出`1`到`5`的值：

```js
var i=1;
Do {
  document.write("The value of i is " + i + "</br>");
  i++;
}
While(i<6);
```

输出将如下所示：

```js
The value of i is 1
The value of i is 2
The value of i is 3
The value of i is 4
The value of i is 5
```

### 小贴士

`do while`循环可以在我们需要至少执行一次循环，无论条件是否满足的情况下使用。

### `for in`循环

在 JavaScript 中，还有一种循环类型，称为`for in`循环。这个循环用于 JavaScript 中的对象属性。在这个循环中，对象的一个属性被分配给一个变量名。

#### 语法

下面是 JavaScript 中`for in`循环的语法：

```js
For (property in object) {
  //code block
}
```

#### 示例

下面是`for in`循环的一个示例：

```js
var I;
For (I is navigator) {
  Document.Write(i);
}
Document.Write("The loop ends here");
```

## 跳转和标签语句

在 JavaScript 中，`jump`语句用于强制执行流程跳转到脚本中的另一个条件。基本上，它用于终止迭代语句。JavaScript 中有两种`jump`语句：

+   `Break`

+   `继续`

这些语句用于当你立即离开循环或因为代码中的条件而想要提前跳转到另一个条件时。

### `break`语句

在 JavaScript 中，`break`语句与`switch`语句一起使用。它用于提前退出循环或条件。`break`语句也用于`for`循环和`while`循环中。

#### 语法

下面是 JavaScript 中`break`语句的语法：

```js
Any loop(condition) {
  //block of code that will be executed
  break;
  //block of code that won't get executed
}
```

#### 示例

考虑以下简单示例：

```js
var i=1;
While(i<10) {
  if(i==3) {
    Document.Write("Breaking from if loop");
    Break;
  }
  Document.Write("The value of i is " + i);
  i+=2;
}
```

输出将如下所示：

```js
The value of i is 1
Breaking from if loop
The value of i is 3
```

### `continue`语句

关键字`continue`将跳过当前迭代。

#### 语法

下面是 JavaScript 中`continue`语句的语法：

```js
Any loop(condition) {
  //block of code that will be executed
  Any loop (condition) {
    //block of code that will be executed
    continue;
    block of code that wont be executed
  }
  //block of code  that will be executed
}
```

#### 示例

考虑以下简单示例：

```js
var i=1;
While(i<5) {
  if(i==3) //skip the iteration {
    Document.Write("Continue statement encountered");
    continue;
  }
  Document.Write("The value of i is " + i);
  i=i+2;
}
```

输出将如下所示：

```js
The value of i is 1
Continue statement encountered
```

### `return`语句

在 JavaScript 中，`return` 语句用于从函数中返回一个值。返回值后，它停止函数执行。每个函数和语句都返回一个值。如果没有提供 `return` 语句，则返回未定义值。

#### 语法

下面是 JavaScript 中 `return` 语句的语法：

```js
function myfunc(x, y) {
  //some block of code
  return x;
}
Document.Write("The result is " + myfunc(2, 3));
```

#### 示例

考虑以下简单示例，展示 `return` 语句的工作原理：

```js
Function multiply(a,b) {
  return a*B;
}
var mul=multiply(2,3);
Document.Write("The result is "+ mul);
```

输出结果如下：

```js
The result is 6
```

### 抛出语句

`throw` 语句用于为 `try` 和 `catch` 块创建用户定义的错误条件。这些错误也称为 **异常**。我们可以在代码脚本中创建自己的异常。

#### 语法

下面是 JavaScript 中 `throw` 语句的语法：

```js
try {
  //block of code
  throw "error";
  //block of code
}
catch(error) {
  //block of code
}
```

#### 示例

让我们看看以下代码片段：

```js
Try {
  document.write( "hello world);//the code will throw an error due to the missing "
}
Catch(error) {
  Error.messageTxt;
}
```

### Try catch finally 语句

在 `try` 语句中，我们编写在执行时需要错误测试的代码。在 `catch` 块中，你编写处理 `try` 块中发生的错误的代码。当你将代码写入 `try` 块中时，你必须编写一个 `catch` 块来处理 `try` 块中可能发生的任何错误。当 `try` 和 `catch` 语句成功执行时，`finally` 语句执行。

#### 语法

`try`、`catch` 和 `finally` 块的语法如下所示：

```js
Try {//generate exception}
Catch {//error handling code}
Finally {//block of code which runs after try and catch}
```

# 数组

数组用于存储有序数据集合。如果我们有多个项目，则可以使用数组来存储这些值。数组原型方法用于在 JavaScript 中执行变异操作。数组没有固定长度。数组可以包含多个值类型，并且可以在初始化后任何时刻修改项目数量。

## 数组类型

在 JavaScript 中有两种数组类型：

+   **名义类型**：这种类型的数组具有唯一的标识。

+   **结构类型**：这种数组类型看起来像接口；它也被称为鸭子类型。它使用特定行为的具体实现。

### 注意

**稀疏数组**

在编程语言中，稀疏数组表示具有相同值（如 `0` 或 `null`）的数组。在数组存储中存在大量零，这在稀疏数组中发生。在稀疏数组中，索引不从 `0` 开始。在稀疏数组中，元素的长度小于属性的长度。

在 JavaScript 中，`length` 属性不是数组中实际元素的数量，而是最后一个 *索引+1*。因此，在稀疏数组的情况下，这一点至关重要，需要处理空白索引。考虑以下示例：

```js
var person=[];
Person[1]="Ali";
Person[20]="Ahmed";
Alert(person.length);// it will be 21 length
```

## 数组类型对象

一些 JavaScript 对象看起来像数组，但实际上它们不是数组。它们被称为 **类似数组** 对象。

类似数组的对象：

+   **具有**：这告诉你对象有多少个元素

+   **不具有**：这些是数组方法，例如 `indexOf`、`push` 和 `forEach`

### 注意

**字符串作为数组**

在 JavaScript 中，有一些字符串表现得像数组。如果我们把一个字符串当作数组，那么它就只能被当作数组。数组中有些方法，如`push()`、`reverse()`等，在字符串上不起作用。

## 创建数组

在 JavaScript 中，有两种创建数组的方法：

+   使用数组初始化器

+   使用数组构造函数

您可以使用这两种方法中的任何一种。

数组初始化器中有一个用方括号括起来的逗号分隔的字符串；例如：

```js
var students= ["Ali","Ahmed","Amina"];
```

在 JavaScript 中，您也可以使用数组构造函数通过使用 new 关键字来创建一个数组，这会创建一个数组并将值赋给它；例如：

```js
var students=new Array ("Ali,"Ahmed","Amina");
```

### 提示

数组初始化器方法是一种快速且好的初始化数组的方法，因为它的执行速度比构造函数方法要高。如果您在数组构造函数中不传递任何参数，那么它将设置其长度为零。

### 数组初始化器

数组用于在单个变量中存储有序数据集合。这意味着当您想要存储一个项目列表时，应该使用数组。在您的程序中使用数组之前，您需要首先初始化它。

在数组初始化器中，有一个变量中存储的项目逗号分隔列表，例如：

```js
var a1=[];
var a2=[2];
var a3=[1,2,3];
var a4=['hi', [2]];
```

### 提示

当您使用数组初始化器初始化数组时，所有项的类型不必相同。此外，数组可以有零长度。

### 数组构造函数

您可以使用数组构造函数的以下三种不同方式。它们的语法如下：

```js
var a1=new array(); // new empty array
var a2=new array(2); // new array of size 2
var a3= new array(1,2,3); // new array containing 1, 2, 3
```

如果您不带参数调用数组构造函数，它将设置其长度为零。

如果数组构造函数只有一个参数，则此参数将初始化其新长度。

如果您调用一个包含两个或更多元素的数组，参数将初始化一个大小等于参数数量的数组。

## 读取和写入数组元素

要从数组中读取和写入元素，我们使用方括号`[]`。它就像访问对象属性一样工作。在括号内应该是一个非负数。在数组中读取和写入的语法是相同的。值也在数组中索引。您通过索引读取值。数组的引用应该在括号的左侧。

考虑以下示例：

```js
var obj=["abc"];
var start=obj[0]; //its is for reading

Document.Write(start);

var obj[1]="hello";
i="world";
Obj[i]="hello world";//writing
Document.Write(obj[i]);
```

此输出的结果如下：

```js
Hello world
```

## JavaScript 中的多维数组

与其他编程语言一样，您也可以在 JavaScript 中创建多维数组。它们的语法与其他语言相同。在多维数组中，您在嵌套数组循环中创建数组内的数组。

您可以通过为每个需要的维度添加更多数组来创建一个数组。当您需要用信息覆盖整个存储空间时，创建多维数组很有用。如果大部分数据是稀疏的，也就是说数组是空的，那么使用关联数组来存储信息是一种更好的技术。以下是在 JavaScript 中创建多维数组的示例：

```js
var a=5;
var b=2;
var c=new array();
For(i=0;i<5;i++) {
  C[i]=new array();
  for For(j=0;j<5;j++)
  c[i][j];  }}
}
```

## 数组中的属性

数组具有原型、长度和构造器作为其属性。

### 长度

在 JavaScript 中，数组长度属性返回数组中的元素数量。

#### 返回值

这返回一个 32 位整数值。

#### 描述

我们还可以在需要时使用数组长度属性来清空数组。当你增加任何数组的长度时，它不会增加其中的元素数量。我们可以使用数组长度属性设置或返回任何数组的长度。

考虑以下示例：

```js
var num=new Array[1,2,3];
Document. write (num.length);
```

输出将如下所示：

```js
3
```

### 构造器

数组构造器用于初始化一个数组。一个数组可以包含零个或多个元素，其语法如下，其中数组元素由方括号内的逗号分隔。

```js
var fruits = ["Apple", "Banana"];
```

### 原型

所有数组实例都从 `Array.prototype` 继承。数组的原型构造器允许向 `Array()` 对象添加新方法和属性。

```js
Array.prototype is an array itself.
Array.isArray(Array.prototype); // true
```

你可以在 MDN 上找到有关如何进行这些更改的详细信息（[`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/prototype`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/prototype)）。

## 数组方法

有几种方法可以对数组执行以获得不同的结果。其中一些方法在以下章节中定义。

### concat()

`concat()` 方法在两个数组之间执行连接操作。

#### 返回值

`concat()` 方法返回连接后的数组。

#### 参数

`concat()` 方法接受一个要连接的字符串。

#### 描述

这需要两个数组并将它们连接起来。

```js
var alpha = ['a', 'b', 'c'],
var numeric = [1, 2, 3];
var alphaNumeric = alpha.concat(numeric);
console.log(alphaNumeric); // Result: ['a', 'b', 'c', 1, 2, 3]
```

### every()

`every()` 方法对每个数组元素测试一个函数。

#### 返回值

`every()` 方法返回布尔值 `true` 或 `false`。

#### 参数

在数组的每个元素上应用的一个回调函数。

#### 描述

如果数组中的每个元素都提供了测试函数，则它返回 `true`。

```js
function isBigEnough(element, index, array) {
  return element >= 10;
}
[12, 5, 8, 130, 44].every(isBigEnough);   // false
[12, 54, 18, 130, 44].every(isBigEnough); // true
```

### foreach()

`foreach()` 方法对数组中的每个元素执行一个作为参数传递的函数。

#### 返回值

每个函数的输出。

#### 参数

一个回调函数，用于调用数组的每个元素。

#### 描述

`foreach()` 方法调用每个元素的函数：

```js
function logArrayElements(element, index, array) {
  console.log('a[' + index + '] = ' + element);
}

// Note elision, there is no member at 2 so it isn't visited
[2, 5, , 9].forEach(logArrayElements);
// logs:
// a[0] = 2
// a[1] = 5
// a[3] = 9
```

### join()

这将所有元素连接成一个字符串。

#### 返回值

一个连接的字符串。

#### 参数

`Null` 或一个分隔符，用于在数组元素之间放置。

#### 描述

它将所有元素连接成一个字符串。

```js
var a = ['One', 'Two', 'Three'];
var var1 = a.join();      // assigns 'One,Two,Three' to var1
var var2 = a.join(', ');  // assigns 'One, Two, Three' to var2
var var3 = a.join(' + '); // assigns 'One + Two + Three' to var3
var var4 = a.join('');    // assigns 'OneTwoThree' to var4
```

### pop()

`pop()` 方法移除数组的最后一个元素，就像一个栈。

#### 返回值

`pop()` 方法返回一个 `null` 参数。

#### 参数

`pop()` 方法接受 `null` 参数。

#### 描述

它移除数组的最后一个元素，就像一个栈。

```js
var myColours = ['Yellow', 'Black', 'Blue', 'Green'];
console.log(myColours); // ['Yellow', 'Black', 'Blue', 'Green']
var popped = myColours.pop();
console.log(myColours); // ['Yellow', 'Black', 'Blue' ] 
console.log(popped); // 'Green'
```

### push()

它在数组的最后一个索引上添加元素。

#### 返回值

`null`。

#### 参数

`null`。

#### 描述

这在数组的最后一个索引上添加元素。

```js
var sports = ['cricket', 'baseball'];
var total = sports.push('football');

console.log(sports); // ['cricket', 'baseball', 'football']
console.log(total);  // 3
```

### indexOf()

`indexOf()` 方法返回数组的第一个索引。

#### 返回值

`indexOf()` 方法返回一个索引。

#### 参数

`indexOf()` 方法接受一个数组元素作为参数。

#### 描述

它返回数组的第一个索引。

```js
var array = [2, 5, 9];
array.indexOf(9);     // 2
array.indexOf(7);     // -1 if not found
```

### lastIndexOf()

`lastIndexOf()` 方法返回给定元素在数组中最后一次出现的位置索引。

#### 返回值

它返回数组的最后一个索引。

#### 参数

数组元素。

#### 描述

```js
var array = [2, 5, 9, 2];
array.indexOf(2);     // 3
array.indexOf(7);     // -1 if not found
```

### reverse()

`reverse()` 方法反转数组中所有元素的顺序。

#### 返回值

数组。

#### 参数

`null`。

#### 描述

`reverse()` 方法反转数组中元素的顺序。第一个元素变为最后一个元素，依此类推。

```js
var myArray = ['one', 'two', 'three'];
myArray.reverse(); 

console.log(myArray) // ['three', 'two', 'one']
```

### shift()

在数组中，`shift()` 方法移除并返回第一个元素。

#### 返回值

`null`。

#### 参数

`null`。

#### 描述

在数组中，它移除并返回第一个元素。

```js
var myColours = ['Yellow', 'Black', 'Blue', 'Green'];
console.log(myColours); // ['Yellow', 'Black', 'Blue', 'Green']
var shifted = myColours.shift();
console.log(myColours); // ['Black', 'Blue', 'Green'] 
console.log(shifted); // 'Green'
```

### unshift()

此方法在数组的开头添加一个新元素，并返回新的数组长度。

#### 返回值

数组的新长度。

#### 参数

要添加到数组中的元素。

#### 描述

此方法在数组的开头添加一个新元素，并返回新的数组长度：

```js
var arr = [1, 2, 3];
arr.unshift(0); // arr is [0, 1, 2, 3]
```

### slice()

`slice()` 方法将数组切片到一个新数组。

#### 返回值

新的切片数组。

#### 参数

要切片的元素索引。

#### 描述

此方法将数组切片到一个新数组：

```js
var fruits = ['Banana', 'Orange', 'Lemon', 'Apple', 'Mango'];
var citrus = fruits.slice(1, 3);

// citrus contains ['Orange','Lemon']
```

### splice()

`splice()` 方法也通过移除现有元素来向数组中添加新元素。

#### 返回值

新的切片数组。

#### 参数

要移除的元素索引和要添加的元素。

#### 描述

此方法也通过移除现有元素来向数组中添加新元素：

```js
var myCars = ['Audi', 'BMW', 'Ferrari', 'Volkswagen'];

// removes 0 elements from index 2, and inserts 'Toyota'
var removed = myCars.splice(2, 0, 'Toyota');
// myCars is ['Audi', 'BMW', 'Toyota', 'Volkswagen']
// removed is [], no elements removed
```

### sort()

此方法按字母顺序对数组进行排序。

#### 返回值

`null`。

#### 参数

`null`。

#### 描述

作为数组方法调用，此方法按字母顺序（Unicode 字符）对数组进行排序，并且与数字不兼容：

```js
var fruit = ['cherries', 'apples', 'bananas'];
fruit.sort(); // ['apples', 'bananas', 'cherries']
```

### toString()

`toString()` 方法将对象转换为字符串。

#### 返回值

`toString()` 方法返回一个字符串。

#### 参数

`toString()` 方法接受一个 `null` 参数。

#### 描述

将数组对象转换为以逗号分隔的字符串。

```js
var monthNames = ['Jan', 'Feb', 'Mar', 'Apr'];
var myVar = monthNames.toString(); // 'Jan,Feb,Mar,Apr' to myVar.
```

## ECMA5 数组方法

新增的数组方法被添加到 ECMA5 脚本中，也称为 **数组扩展**。这些是九个新方法。这些方法执行与数组相关的常见操作。这些新方法将在以下章节中介绍。

### array.prototype.map()

`map()` 方法根据每次迭代的返回值创建一个新数组。

#### 返回值

修改后的数组。

#### 参数

回调函数。

#### 描述

它遍历数组，运行一个函数，并根据每次迭代的返回值创建一个新数组。它接受与 `forEach()` 方法相同的参数。

这里有一个简单的例子：

```js
var numbers = [1, 4, 9];
var roots = numbers.map(Math.sqrt);
```

根现在如下所示：

```js
[1, 2, 3]
```

### array.prototype.filter()

`filter()` 函数创建一个新数组或修改后的数组，该数组包含通过函数处理的值。

#### 返回值

修改后的数组。

#### 参数

回调函数。

#### 描述

它创建一个新数组，仅包含那些回调返回 `true` 的元素。

以下是一个简单的示例：

```js
function isBigEnough(value) {
  return value >= 10;
}
var filtered = [12, 5, 8, 130, 44].filter(isBigEnough);
```

输出如下：

```js
[12, 130, 44]
```

### array.prototype.reduce()

`reduce()` 函数同时将一个函数应用于数组的两个值，以将它们减少到一个单一值。值的选取方向是从左到右。

#### 返回值

修改后的数组。

#### 参数

一个回调函数。

#### 描述

`array.prototype.reduce()` 方法通过在回调函数中执行的操作，将数组的所有值累积到一个单一值。

以下是一个简单的示例：

```js
[0, 1, 2, 3, 4].reduce(function(previousValue, currentValue, index, array) {
  return previousValue + currentValue;
});
```

回调的执行方式如下：

|   | previousValue | currentValue | index | array | return value |
| --- | --- | --- | --- | --- | --- |
| **第一次调用** | `0` | `1` | `1` | `[0, 1, 2, 3, 4]` | `1` |
| **第二次调用** | `1` | `2` | `2` | `[0, 1, 2, 3, 4]` | `3` |
| **第三次调用** | `3` | `3` | `3` | `[0, 1, 2, 3, 4]` | `6` |
| **第四次调用** | `6` | `4` | `4` | `[0, 1, 2, 3, 4]` | `10` |

### array.prototype.forEach()

`array.prototype.forEach()` 方法对数组中的每个元素执行一个作为参数传递的函数。

#### 返回值

每个函数的输出。

#### 参数

一个要为数组每个元素调用的回调函数。

#### 描述

`array.prototype.forEach()` 方法调用每个元素的函数。以下是一个简单的示例：

```js
function logArrayElements(element, index, array) {
  console.log('a[' + index + '] = ' + element);
}

// Note elision, there is no member at 2 so it isn't visited
[2, 5, , 9].forEach(logArrayElements);
// logs:
// a[0] = 2
// a[1] = 5
// a[3] = 9
```

### array.prototype.indexOf()

`array.prototype.indexOf()` 方法返回数组的第一个索引。

#### 返回值

一个索引。

#### 参数

一个数组元素。

#### 描述

`array.prototype.indexOf()` 方法返回数组的第一个索引。以下是一个简单的示例：

```js
var array = [2, 5, 9];
array.indexOf(9);     // 2
array.indexOf(7);     // -1 if not found
```

### array.prototype.lastIndexOf()

`array.prototype.lastIndexOf()` 方法返回给定元素在数组中找到的最后一个索引。

#### 返回值

它返回数组的最后一个索引。

#### 参数

一个数组元素。

#### 描述

如前所述，如果数组中存在指定的元素，`array.prototype.lastIndexOf()` 函数将返回该元素的最后一个索引。以下是一个简单的示例：

```js
var array = [2, 5, 9, 2];
array.indexOf(2);     // 3
array.indexOf(7);     // -1 if not found
```

### array.prototype.every()

`array.prototype.every()` 方法对每个数组元素进行函数测试。

#### 返回值

一个布尔值 `true` 或 `false`。

#### 参数

一个要应用于数组每个元素的回调函数。

#### 描述

如果数组中的每个元素都提供了测试函数，则返回 `true`。以下是一个简单的示例：

```js
function isBigEnough(element, index, array) {
  return element >= 10;
}
[12, 5, 8, 130, 44].every(isBigEnough);   // false
[12, 54, 18, 130, 44].every(isBigEnough); // true
```

### array.prototype.some()

此方法测试是否有任何元素通过了由提供的函数实现的测试。

#### 返回值

一个布尔值 `True` 或 `False`。

#### 参数

一个回调函数。

#### 描述

`array.prototype.some()` 方法与 `Array.prototype.every()` 类似，但这里的条件是至少有一个回调应该返回 `true`。

一个简单的示例如下：

```js
function isBiggerThan10(element, index, array) {
  return element > 10;
}
[2, 5, 8, 1, 4].some(isBiggerThan10);  // false
[12, 5, 8, 1, 4].some(isBiggerThan10); // true
```

### array.prototype.reduceRight()

`array.prototype.reduceRight()` 方法从右到左同时应用一个函数到数组的两个值（值），以便将它们减少到一个单一值。

#### 返回值

修改后的数组。

#### 参数

一个回调函数。

#### 描述

这与 reduce 操作完全相同，但它从右侧开始，向左侧移动，同时累积值。

这里有一个简单的例子：

```js
var total = [0, 1, 2, 3].reduceRight(function(a, b) {
  return a + b;
});
```

这返回总和为`6`。
