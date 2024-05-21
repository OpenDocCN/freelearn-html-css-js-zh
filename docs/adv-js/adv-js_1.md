# 第一章：介绍 ECMAScript 6

## 学习目标

在本章结束时，您将能够：

+   定义 JavaScript 中的不同作用域并表征变量声明

+   简化 JavaScript 对象定义

+   解构对象和数组，并构建类和模块

+   为了兼容性转译 JavaScript

+   组合迭代器和生成器

在本章中，您将学习如何使用 ECMAScript 的新语法和概念。

## 介绍

JavaScript，通常缩写为 JS，是一种旨在允许程序员构建交互式 Web 应用程序的编程语言。JavaScript 是 Web 开发的支柱之一，与 HTML 和 CSS 一起。几乎每个主要的网站，包括 Google、Facebook 和 Netflix，都大量使用 JavaScript。JS 最初是为 Netscape Web 浏览器于 1995 年创建的。JavaScript 的第一个原型是由 Brendan Eich 在短短的 10 天内编写的。自创建以来，JavaScript 已成为当今最常用的编程语言之一。

在本书中，我们将加深您对 JavaScript 核心及其高级功能的理解。我们将涵盖 ECMAScript 标准中引入的新功能，JavaScript 的异步编程特性，DOM 和 HTML 事件与 JavaScript 的交互，JavaScript 的函数式编程范式，测试 JavaScript 代码以及 JavaScript 开发环境。通过本书所获得的知识，您将准备好在专业环境中使用 JavaScript 构建强大的 Web 应用程序。

## 从 ECMAScript 开始

**ECMAScript**是由**ECMA International**标准化的脚本语言规范。它旨在标准化 JavaScript，以允许独立和兼容的实现。**ECMAScript 6**，或**ES6**，最初于 2015 年发布，并自那时以来经历了几次次要更新。

#### 注意

您可以参考以下链接了解更多关于 ECMA 规范的信息：[`developer.mozilla.org/en-US/docs/Web/JavaScript/Language_Resources`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Language_Resources)。

## 理解作用域

在计算机科学中，**作用域**是计算机程序中名称与实体（如变量或函数）的绑定或关联有效的区域。JavaScript 具有以下两种不同类型的作用域：

+   **函数作用域**

+   **块作用域**

在 ES6 之前，函数作用域是 JavaScript 中唯一的作用域形式；所有变量和函数声明都遵循函数作用域规则。ES6 引入了块作用域，仅由使用新的变量声明关键字`let`和`const`声明的变量使用。这些关键字在*声明变量*部分中有详细讨论。

### 函数作用域

JavaScript 中的**函数作用域**是在函数内部创建的。当声明一个函数时，在该函数的主体内部创建一个新的作用域块。在新函数作用域内声明的变量无法从父作用域访问；但是，函数作用域可以访问父作用域中的变量。

要创建具有函数作用域的变量，必须使用`var`关键字声明变量。例如：

`var example = 5;`

以下代码段提供了函数作用域的示例：

```php
var example = 5;
function test() {
  var testVariable = 10;
  console.log( example ); // Expect output: 5
  console.log( testVariable ); // Expect output: 10
}
test();
console.log( testVariable ); // Expect reference error
```

###### 代码段 1.1：函数作用域

**父作用域**只是函数定义的代码段的作用域。这通常是全局作用域；但是，在某些情况下，在函数内部定义函数可能很有用。在这种情况下，嵌套函数的父作用域将是其定义的函数。在前面的代码段中，函数作用域是在函数 test 内创建的作用域。父作用域是全局作用域，即函数定义的地方。

#### 注意

父作用域是定义函数的代码块。它不是调用函数的代码块。

### 函数作用域提升

当使用函数作用域创建变量时，其声明会自动提升到作用域的顶部。**提升**意味着解释器将实体的实例化移动到其声明的作用域顶部，而不管它在作用域块中的定义位置。在 JavaScript 中，使用`var`声明的函数和变量会被提升；也就是说，函数或变量可以在其声明之前使用。以下代码演示了这一点：

```php
example = 5; // Assign value
console.log( example ); // Expect output: 5
var example; // Declare variable
```

###### 片段 1.2：函数作用域提升

#### 注意

由于使用`var`声明的提升变量可以在声明之前使用，因此我们必须小心在变量被赋值之前不要使用该变量。如果在变量被赋值之前访问变量，它将返回`undefined`，这可能会导致问题，特别是如果变量在全局作用域中使用。

### 块作用域

在 JavaScript 中，使用花括号（`{}`）创建一个新的块作用域。一对**花括号**可以放置在代码的任何位置以定义一个新的作用域块。if 语句、循环、函数和任何其他花括号对都将有自己的块作用域。这包括与关键字（if、for 等）无关的浮动花括号对。以下片段中的代码是块作用域规则的示例：

```php
// Top level scope
function scopeExample() {
  // Scope block 1
  for ( let i = 0; i < 10; i++ ){ /* Scope block 2 */ }
  if ( true ) { /* Scope block 3 */ } else {  /* Scope block 4 */ }
  // Braces without keywords create scope blocks
  { /* Scope block 5 */ } 
  // Scope block 1
}
// Top level scope
```

###### 片段 1.3：块作用域

使用关键字`let`和`const`声明的变量具有**块作用域**。当使用块作用域声明变量时，它不具有与在函数作用域中创建的变量相同的变量提升。块作用域变量不会被提升到作用域的顶部，因此在声明之前无法访问。这意味着使用块作用域创建的变量受到**暂时性死区**（**TDZ**）的影响。TDZ 是指进入作用域和声明变量之间的时间段。它在变量被声明而不是赋值时结束。以下示例演示了 TDZ：

```php
// console.log( example ); // Would throw ReferenceError
let example;
console.log( example ); // Expected output: undefined
example = 5;
console.log( example ); // Expected output: 5
```

###### 片段 1.4：暂时性死区

#### 注意

如果在暂时性死区内访问变量，则会抛出运行时错误。这很重要，因为它可以使我们的代码更加健壮，减少由于变量声明而产生的语义错误。

要更好地理解作用域块，请参考以下表格：

![图 1.1：函数作用域与块作用域](img/Figure_1.1.jpg)

###### 图 1.1：函数作用域与块作用域

总之，作用域为我们提供了一种在代码块之间分离变量并限制访问的方式。变量标识符名称可以在作用域块之间重复使用。所有创建的新作用域块都可以访问父作用域，或者它们被创建或定义的作用域。JavaScript 有两种作用域。为每个定义的函数创建一个新的函数作用域。变量可以使用`var`关键字添加到函数作用域，并且这些变量会被提升到作用域的顶部。块作用域是 ES6 的一个新特性。为每组花括号创建一个新的块作用域。使用`let`和`const`关键字将变量添加到块作用域。添加的变量不会被提升，并且受到 TDZ 的影响。

### 练习 1：实现块作用域

要使用变量实现块作用域原则，请执行以下步骤：

1.  创建一个名为`fn1`的函数（`function fn1()`）。

1.  记录字符串为`scope 1`。

1.  创建一个名为`scope`的变量，其值为 5。

1.  记录名为`scope`的变量的值。

1.  在函数内部使用花括号（`{}`）创建一个新的作用域块。

1.  在新的作用域块内，记录名为`scope 2`的字符串。

1.  在作用域块内创建一个名为`scope`的新变量，并赋值为`different scope`。

1.  记录块作用域内变量`scope`的值（scope 2）。

1.  在步骤 5 中定义的作用域块之外（scope 2），创建一个新的作用域块（使用花括号）。

1.  记录名为`scope 3`的字符串。

1.  在作用域块（作用域 3）内创建一个同名的变量（称为 `scope`）并将其赋值为 `第三个作用域`。

1.  记录新变量的值。

1.  调用 `fn1` 并观察其输出

**代码**

index.js：

```php
function fn1(){
 console.log('Scope 1');
 let scope = 5;
 console.log(scope);
 {
   console.log('Scope 2');
   let scope = 'different scope';
   console.log(scope);
 }
  {
   console.log('Scope 3');
   let scope = 'a third scope';
   console.log(scope);
 }
}
fn1();
```

[`bit.ly/2RoOotW`](https://bit.ly/2RoOotW)

###### 代码片段 1.5：块实现输出

**结果**

![图 1.2：作用域输出](img/Figure_1.2.jpg)

###### 图 1.2：作用域输出

您已成功在 JavaScript 中实现了块作用域。

在本节中，我们介绍了 JavaScript 作用域的两种类型，函数作用域和块作用域，以及它们之间的区别。我们演示了如何在每个函数内部创建一个新的函数作用域实例，以及如何在每组花括号内创建块作用域。我们讨论了每种作用域类型的变量声明关键字，`var` 用于函数作用域，`let/const` 用于块作用域。最后，我们介绍了函数作用域和块作用域的变量提升的基础知识。

## 声明变量

基本 JavaScript 使用关键字 `var` 进行**变量声明**。ECMAScript 6 引入了两个新关键字来声明变量；它们是 `let` 和 `const`。在专业 JavaScript 变量声明的世界中，`var` 现在是最薄弱的环节。在本主题中，我们将介绍新关键字 `let` 和 `const`，并解释它们为什么比 `var` 更好。

在 JavaScript 中声明变量的三种方式是使用 `var`、`let` 和 `const`。它们的功能略有不同。这三种变量声明关键字之间的关键区别在于它们处理变量重新分配、变量作用域和变量提升的方式。这三个特性可以简要解释如下：

**变量重新赋值：** 在任何时候改变或重新分配变量的值的能力。

**变量作用域：** 变量可以被访问的代码范围或区域。

**变量提升：** 变量实例化和赋值时间与变量声明的关系。有些变量可以在它们被声明之前使用。

`var` 关键字是在 JavaScript 中用于声明变量的较旧的关键字。所有使用 `var` 创建的变量都可以重新分配，具有函数作用域，并且具有变量提升。这意味着使用 `var` 创建的变量被提升到作用域块的顶部，在那里它们被定义并且可以在声明之前访问。以下代码片段演示了这一点，如下所示：

```php
// Referenced before declaration
console.log( example ); // Expect output: undefined
var example = 'example';
```

###### 代码片段 1.6：使用 var 创建的变量被提升

由关键字 `var` 创建的变量不是常量，因此可以随意创建、分配和重新分配值。以下代码演示了 `var` 功能的这一方面：

```php
// Declared and assigned
var example = { prop1: 'test' };
console.log( 'example:', example );
// Expect output: example: {prop1: "test"}
// Value reassigned
example = 5;
console.log( example ); // Expect output: 5
```

###### 代码片段 1.7：使用 var 创建的变量不是常量

使用 `var` 创建的变量可以在任何时候重新分配，并且一旦变量被创建，即可在函数中的任何地方访问，甚至是在原始声明点之前。

`let` 关键字与关键字 `var` 类似。如预期的那样，关键字 `let` 允许我们声明一个可以在任何时候重新分配的变量。以下代码中展示了这一点：

```php
// Declared and initialized
let example = { prop1: 'test' };
console.log( 'example:', example );
// Expect output: example: {prop1: 'test"}
// Value reassigned
example = 5;
console.log( example ); // Expect output: 5
```

###### 代码片段 1.8：使用 let 创建的变量不是常量

`let` 和 `var` 之间有两个重要的区别。`let` 和 `var` 的区别在于它们的作用域和变量提升属性。使用 `let` 声明的变量的作用域是块级的；也就是说，它们只在匹配的一对花括号（`{}`）内的代码块中定义。

使用 `let` 声明的变量不受变量提升的影响。这意味着在赋值之前访问使用 `let` 声明的变量将引发运行时错误。正如前面讨论的那样，这就是暂时性死区。以下代码示例说明了这一点：

```php
// Referenced before declaration
console.log( example );
// Expect ReferenceError because example is not defined
let example = 'example';
```

###### 代码片段 1.9：使用 let 创建的变量不会被提升

最后一个变量声明关键字是`const`。`const`关键字具有与`let`关键字相同的作用域和变量提升规则；使用`const`声明的变量具有块作用域，并且不会被提升到作用域的顶部。这在以下代码中显示：

```php
// Referenced before declaration
console.log( example );
// Expect ReferenceError because example is not defined
const example = 'example';
```

###### 片段 1.10：使用 const 创建的变量不会被提升

`const`和`let`之间的关键区别在于`const`表示标识符不会被重新分配。`const`标识符表示对值的只读引用。换句话说，不能更改`const`变量中写入的值。如果更改了使用`const`初始化的变量的值，将抛出`TypeError`。

即使使用`const`创建的变量不能被重新分配，这并不意味着它们是不可变的。如果数组或对象存储在使用`const`声明的变量中，则无法覆盖变量的值。但是，数组内容或对象属性可以更改。可以使用`push()`、`pop()`或`map()`等函数修改数组的内容，并且可以添加、删除或更新对象属性。这在以下代码中显示：

```php
// Declared and initialized
const example = { prop1: 'test' };
// Variable reassigned
example = 5;
// Expect TypeError error because variable was declared with const
// Object property updated
example.prop1 = 5;
// Expect no error because subproperty was modified
```

###### 片段 1.11：使用 const 创建的变量是常量但不是不可变的

要更详细地了解不同的关键字，请参考以下表格：

![图 1.3：var、let 和 const 之间的差异](img/Figure_1.3.jpg)

###### 图 1.3：var、let 和 const 之间的差异

现在我们了解了`var`、`let`和`const`之间的细微差别，我们可以决定使用哪一个。在专业世界中，我们应该始终使用`let`和`const`，因为它们提供了`var`的所有功能，并允许程序员对变量的范围和用法进行具体和限制性的定义。

总之，`var`、`let`和`const`都有类似的功能。关键区别在于`const`的性质、作用域和提升。`var`是函数作用域的，不是常量，并且被提升到作用域块的顶部。`let`和`const`都是块作用域的，不会被提升。`let`不是常量，而`const`是常量但不可变的。

### 练习 2：利用变量

为了利用`var`、`const`和`let`变量声明关键字的变量提升和重新分配属性，执行以下步骤：

1.  记录字符串`赋值前提升：`和`hoisted`变量的值。

1.  使用关键字`var`定义一个名为`hoisted`的变量，并将其赋值为`this got hoisted`。

1.  记录字符串`赋值后提升：`和`hoisted`变量的值。

1.  创建一个 try-catch 块。

1.  在`try`块内，记录名为`notHoisted1`的变量的值。

1.  在`catch`块内，给 catch 块`err`参数，然后记录字符串`带错误的未提升 1：`和`err.message`的值。

1.  在 try-catch 块之后，使用关键字`let`创建`notHoisted1`变量，并赋值为`5`。

1.  记录字符串`赋值后 notHoisted1`和`notHoisted1`的值。

1.  创建另一个 try-catch 块。

1.  在`try`块内，记录`notHoisted2`变量的值。

1.  在 catch 块内，给 catch 块`err`参数，然后记录字符串`带错误的未提升 2：`和`err.message`的值。

1.  在第二个 try-catch 块之后，使用关键字`const`创建`notHoisted2`变量，并赋值[`1`,`2`,`3`]。

1.  记录字符串`赋值后 notHoisted2`和`notHoisted2`的值。

1.  定义一个最终的 try catch 块。

1.  在`try`块内，将`notHoisted2`重新分配为`new value`字符串。

1.  在 catch 块内，给 catch 块`err`参数，然后记录字符串`未提升 2 无法更改`。

1.  在 try-catch 块之后，将值`5`推送到`notHoisted2`中的数组中。

1.  记录字符串`notHoisted2 已更新。现在是：`和`notHoisted2`的值。

**代码**

##### index.js:

```php
var hoisted = 'this got hoisted';
try{
 console.log(notHoisted1);
} catch(err){}
let notHoisted1 = 5;
try{
 console.log(notHoisted2);
} catch(err){}
const notHoisted2 = [1,2,3];
try{
 notHoisted2 = 'new value';
} catch(err){}
notHoisted2.push(5);
```

###### 片段 1.12：更新对象的内容

[`bit.ly/2RDEynv`](https://bit.ly/2RDEynv)

**结果**

![图 1.4：提升变量](img/Figure_1.4.jpg)

###### 图 1.4：提升变量

您已成功地利用关键字声明变量。

在本节中，我们讨论了 ES6 中的变量声明以及使用`let`和`const`变量声明关键字相对于`var`变量声明关键字的好处。我们讨论了每个关键字的变量重新赋值属性，变量作用域和变量提升属性。关键字`let`和`const`都在块作用域中`创建`变量，而`var`在函数作用域中创建变量。使用`var`和`let`创建的变量可以随意重新赋值。然而，使用`const`创建的变量不能被重新赋值。最后，使用关键字`var`创建的变量被提升到它们被定义的作用域块的顶部。使用`let`和`const`创建的变量不会被提升。

## 引入箭头函数

**箭头函数**，或**Fat 箭头函数**，是在 ECMAScript 6 中创建函数的新方法。箭头函数简化了函数语法。它们被称为**fat 箭头函数**，因为它们用字符=>表示，这样放在一起看起来像一个粗箭头。JavaScript 中的箭头函数经常在回调链，承诺链，数组方法中使用，在任何需要未注册函数的情况下都会很有用。

JavaScript 中箭头函数和普通函数之间的关键区别在于箭头函数是**匿名**的。箭头函数没有名称，也没有绑定到标识符。这意味着箭头函数是动态创建的，不像普通函数那样有名称。然而，箭头函数可以分配给一个变量以便重用。

创建箭头函数时，我们只需要删除函数关键字，并在函数参数和函数体之间放置一个箭头。箭头函数用以下语法表示：

```php
( arg1, arg2, ..., argn ) => { /* Do function stuff here */ }
```

###### 片段 1.13：箭头函数语法

从前面的语法中可以看出，箭头函数是 JavaScript 中更简洁的编写函数的方式。它们可以使我们的代码更简洁，更易读。

箭头函数语法也可能有所不同，取决于几个因素。语法可能会略有不同，具体取决于传递给函数的参数数量以及函数体中的代码行数。特殊的语法条件在以下列表中简要概述：

+   单个输入参数

+   无输入参数

+   单行函数体

+   单个表达式跨多行

+   对象字面量返回值

### 练习 3：转换箭头函数

为了演示通过将标准函数转换为箭头函数来简化语法，执行以下步骤：

1.  创建一个接受参数并返回两个参数之和的函数。将函数保存到名为`fn1`的变量中。

1.  将刚刚创建的函数转换为箭头函数，并保存到另一个名为`fn2`的变量中。

要转换函数，删除`function`关键字。接下来，在函数参数和函数体之间放置一个箭头。

1.  调用两个函数并比较输出。

**代码**

##### index.js:

```php
const fn1 = function( a, b ) { return a + b; };
const fn2 = ( a, b ) => { return a + b; };
console.log( fn1( 3 ,5 ), fn2( 3, 5 ) );
```

###### 片段 1.14：调用函数

[`bit.ly/2M6uKwN`](https://bit.ly/2M6uKwN)

**结果**

![图 1.5：比较函数的输出](img/Figure_1.5.jpg)

###### 图 1.5：比较函数的输出

您已成功将普通函数转换为箭头函数。

### 箭头函数语法

如果有多个参数传递给函数，那么我们使用括号来创建函数，括号包围参数就像平常一样。如果我们只有一个参数要传递给函数，我们就不需要在参数周围加括号。

这个规则有一个例外，那就是参数不是简单的标识符。如果我们在函数参数中包含默认值或执行操作，那么我们必须包含括号。例如，如果我们包含默认参数，那么我们将需要在参数周围加上括号。这两条规则如下面的代码所示：

```php
// Single argument arrow function
arg1 => { /* Do function stuff here */ }
// Non simple identifier function argument
( arg1 = 10 ) => { /* Do function stuff here */ }
```

###### 片段 1.15：单参数箭头函数

如果我们创建一个没有参数的箭头函数，那么我们需要包括括号，但括号将是空的。如下面的代码所示：

```php
// No arguments passed into the function
( ) => { /* Do function stuff here */ }
```

###### 片段 1.16：无参数

箭头函数的语法也可以有所不同，取决于函数的主体。如预期的那样，如果函数的主体是多行的，那么我们必须用花括号括起来。但是，如果函数的主体是单行的，那么我们不需要在函数的主体周围包含花括号。这如下面的代码所示：

```php
// Multiple line body arrow function
( arg1, arg2 ) => { 
  console.log( `This is arg1: ${arg1}` );
  console.log( `This is arg2: ${arg2}` );
  /* Many more lines of code can go here */
}
// Single line body arrow function
( arg1, arg2 ) => console.log( `This is arg1: ${arg1}` )
```

###### 片段 1.17：单行体

在使用箭头函数时，如果函数是单行的，我们也可以省略 return 关键字。箭头函数会自动返回该行表达式的解析值。这种语法如下面的代码所示：

```php
// With return keyword - not necessary
( num1, num2 ) => { return ( num1 + num2 ) }
// If called with arguments num1 = 5 and num2 = 5, expected output is 10
// Without return keyword or braces
( num1, num2 ) => num1 + num2
// If called with arguments num1 = 5 and num2 = 5, expected output is 10
```

###### 片段 1.18：返回值为单行体

由于单行表达式体的箭头函数可以在没有花括号的情况下定义，我们需要特殊的语法来允许我们将单个表达式分成多行。为此，我们可以将多行表达式放在括号中。JavaScript 解释器会看到括号中的行，并将其视为单行代码。这如下面的代码所示：

```php
// Arrow function with a single line body
// Assume numArray is an array of numbers
( numArray ) => numArray.filter( n => n > 5).map( n => n - 1 ).every( n => n < 10 )
// Arrow function with a single line body broken into multiple lines
// Assume numArray is an array of numbers
( numArray ) => (
  numArray.filter( n => n > 5)
          .map( n => n - 1 )
          .every( n => n < 10 )
) 
```

###### 片段 1.19：将单行表达式分成多行

如果我们有一个返回对象字面量的单行箭头函数，我们将需要特殊的语法。在 ES6 中，作用域块、函数主体和对象字面量都是用花括号定义的。由于单行箭头函数不需要花括号，我们必须使用特殊的语法来防止对象字面量的花括号被解释为函数主体花括号或作用域块花括号。为此，我们用括号括起返回的对象字面量。这指示 JavaScript 引擎将括号内的花括号解释为表达式，而不是函数主体或作用域块声明。这如下面的代码所示：

```php
// Arrow function with an object literal in the body
( num1, num2 ) => ( { prop1: num1, prop2: num2 } ) // Returns an object
```

###### 片段 1.20：对象字面量返回值

在使用箭头函数时，我们必须注意这些函数被调用的作用域。箭头函数遵循 JavaScript 中的正常作用域规则，但`this`作用域除外。回想一下，在基本的 JavaScript 中，每个函数都被分配一个作用域，即`this`作用域。箭头函数没有被分配一个`this`作用域。它们继承其父级的`this`作用域，并且不能将新的`this`作用域绑定到它们。这意味着，如预期的那样，箭头函数可以访问父函数的作用域，随后访问该作用域中的变量，但`this`的作用域不能在箭头函数中改变。使用`.apply()`、`.call()`或`.bind()`函数修改器都不会改变箭头函数的`this`属性的作用域。如果你处于必须将`this`绑定到另一个作用域的情况，那么你必须使用普通的 JavaScript 函数。

总之，箭头函数为我们提供了简化匿名函数语法的方法。要编写箭头函数，只需省略 function 关键字，并在参数和函数体之间添加一个箭头。

然后可以应用特殊语法来简化箭头函数。如果函数有一个输入参数，那么我们可以省略括号。如果函数主体是单行的，我们可以省略`return`关键字和花括号。然而，返回对象字面量的单行函数必须用括号括起来。

我们还可以在函数体周围使用括号，以便将单行函数体分成多行以提高可读性。

### 练习 4：升级箭头函数

要利用 ES6 箭头函数语法编写函数，请执行以下步骤：

1.  参考`exercises/exercise4/exercise.js`文件并在此文件中执行更新。

1.  使用基本的 ES6 语法转换`fn1`。

在函数参数之前删除函数关键字。在函数参数和函数体之间添加箭头。

1.  使用单语句函数体语法转换`fn2`。

在函数参数之前删除函数关键字。在函数参数和函数体之间添加箭头。

删除函数体周围的花括号`({})`。删除 return 关键字。

1.  使用单个输入参数语法转换`fn3`。

在函数参数之前删除函数关键字。在函数参数和函数体之间添加箭头。

删除函数输入参数周围的括号。

1.  使用无输入参数语法转换`fn4`。

在函数参数之前删除函数关键字。在函数参数和函数体之间添加箭头。

1.  使用对象文字语法转换`fn5`。

在函数参数之前删除函数关键字。在函数参数和函数体之间添加箭头。

删除函数体周围的花括号`({})`。删除 return 关键字。

用括号括起返回的对象。

**代码**

##### index.js：

```php
let fn1 = ( a, b ) => { … };
let fn2 = ( a, b ) => a * b;
let fn3 = a => { … };
let fn4 = () => { … };
let fn5 = ( a ) => ( …  );
```

###### 代码段 1.21：箭头函数转换

[`bit.ly/2M6qSfg`](https://bit.ly/2M6qSfg)

**结果**

![图 1.6：转换函数的输出](img/Figure_1.6.jpg)

###### 图 1.6：转换函数的输出

您已成功利用 ES6 箭头函数语法编写函数。

在本节中，我们介绍了箭头函数，并演示了它们如何在 JavaScript 中大大简化函数声明。首先，我们介绍了箭头函数的基本语法：`( arg1, arg2, argn ) => { /* function body */ }`。然后，我们继续介绍了高级箭头函数的五种特殊语法情况，如下列表所述：

+   单个输入参数：`arg1 => { /* function body */ }`

+   无输入参数：`( ) => { /* function body */ }`

+   单行函数体：`( arg1, arg2, argn ) => /* single line */`

+   单个表达式分成多行：`( arg1, arg2, argn ) => ( /* multi line single expression */ )`

+   对象文字返回值：`( arg1, arg2, argn ) => ( { /* object literal */ } )`

## 学习模板文字

**模板文字**是 ECMAScript 6 中引入的一种新形式的字符串。它们由**反引号**符号(```php`) instead of the usual single or double quotes. Template literals allow you to embed expressions in the string that are evaluated at runtime. Thus, we can easily create dynamic strings from variables and variable expressions. These expressions are denoted with the dollar sign and curly braces (`${ expression }`). The template literal syntax is shown in the following code:

```

const example = "pretty";

console.log( `模板文字非常 ${ example } 有用！！！` );

// 期望输出：模板文字非常有用！！！

```php

###### Snippet 1.22: Template literal basic syntax

Template literals are escaped like other strings in JavaScript. To escape a template literal, simply use a backslash (`\`) character. For example, the following equalities evaluate to true: ``\`` === "`",`\t` === "\t"`, and ``\n\r` === "\n\r".`

Template literals allow for multiline strings. Any newline characters that are inserted into the source are part of the template literal and will result in a line break in the output. In simpler terms, inside a template literal, we can press the **Enter** key on the keyboard and split it on to two lines. This newline character in the source code will be parsed as part of the template literal and will result in a newline in the output. To replicate this with normal strings, we would have to use the `\n` character to generate a new line. With template literals, we can break the line in the template literal source and achieve the same expected output. An example of this is shown in the following code:

```

// 使用普通字符串

console.log( 'This is line 1\nThis is line 2' );

// 期望输出：这是第 1 行

// 这是第 2 行

// 使用模板文字

console.log( `This is line 1

这是第 2 行` );

// 期望输出：这是第 1 行

// 这是第 2 行

```php

###### Snippet 1.23: Template literal multi-line syntax

### Exercise 5: Converting to Template Literals

To convert standard string objects to template literals to demonstrate the power of template literal expressions, perform the following steps:

1.  Create two variables, `a` and `b`, and save numbers into them.
2.  Log the sum of `a` and `b` in the format `a + b` is equal to `<result>` using normal strings.
3.  Log the sum of `a` and `b` in the format `a + b` is equal to `<result>` using a single template literal.

**Code**

##### index.js:

```

let a = 5, b = 10;

console.log( a + ' + ' + b + ' is equal to ' + ( a + b ) );

console.log( `${a} + ${b} is equal to ${a + b}` );

```php

###### Snippet 1.24: Template literal and string comparison

[`bit.ly/2RD5jbC`](https://bit.ly/2RD5jbC)

**Outcome**

![Figure 1.7: Logging the sum of the variable's output](img/Figure_1.7.jpg)

###### Figure 1.7: Logging the sum of the variable's output

You have successfully converted standard string objects to template literals.

Template literals allow for expression nesting, that is, new template literals can be put inside the expression of a template literal. Since the nested template literal is part of the expression, it will be parsed as a new template literal and will not interfere with the external template literal. In some cases, nesting a template literal is the easiest and most readable way to create a string. An example of template literal nesting is shown in the following code:

```

function javascriptOrCPlusPlus() { return 'JavaScript'; }

const outputLiteral = `我们正在学习关于 ${ `专业 ${ javascriptOrCPlusPlus() }` }`

```php

###### Snippet 1.25: Template literal nesting

A more advanced form of template literals are **tagged template literals**. Tagged template literals can be parsed with a special function called **tag functions**, and can return a manipulated string or any other value. The first input argument of a tag function is an array containing string values. The string values represent the parts of the input string, broken at each template expression. The remaining arguments are the values of the template expressions in the string. Tag functions are not called like normal functions. To call a tag function, we omit the parentheses and any whitespace around the template literal argument. This syntax is shown in the following code:

```

// 定义标签函数

function tagFunction( strings, numExp, fruitExp ) {

const str0 = strings[0]; // "We have"

const str1 = strings[1]; // " of "

const quantity = numExp < 10 ? 'very few' : 'a lot';

return str0 + quantity + str1 + fruitExp + str2;

}

定义水果为'apple'，数字为 8。

// 注意：调用标签函数时缺少括号或空格

const output = tagFunction`We have ${num} of ${fruit}. Exciting!`

console.log( output )

// 期望输出：We have very few of apples. Exciting!!

```php

###### Snippet 1.26: Tagged template literal example

A special property named `raw` is available for the first argument of a tagged template. This property returns an array that contains the raw, unescaped, versions of each part of the split template literal. This is shown in the following code:

```

function tagFunction( strings ){ console.log( strings.raw[0] ); }

tagFunction`This is line 1\. \n This is line 2.`

//预期输出：“这是第 1 行。 \n 这是第 2 行。”字符//'\'和'n'不会解析为换行符

```php

###### Snippet 1.27: Tagged template raw property

In summary, template literals allow for the simplification of complicated string expressions. Template literals allow you to embed variables and complicated expressions into strings. Template literals can even be nested into the expression fields of other template literals. If a template literal is broken into multiple lines in the source code, the interpreter will interpret that as a new line in the string and insert one accordingly. Template literals also provide a new way to parse and manipulate strings with the tagged template function. These functions give you a way to perform complex string manipulation via a special function. The tagged template functions also give access to the raw strings as they were entered, ignoring any escape sequences.

### Exercise 6: Template Literal Conversion

You are building a website for a real estate company. You must build a function that takes in an object with property information and returns a formatted string that states the property owner, where the property is located (`address`), and how much they are selling it for (`price`). Consider the following object as input:

```

{

地址：'123 Main St，San Francisco CA，USA'，

楼层：2，

价格：5000000，

owner：“John Doe”

}

```php

###### Snippet 1.28: Object Input

To utilize a template literal to pretty-print an object, perform the following steps:

1.  Create a function called `parseHouse` that takes in an object.
2.  Return a template literal from the function. Using expressions, embed the owner, address, and price in the format `<Owner> is selling the property at <address> for <price>`.
3.  Create a variable called `house` and save the following object into it: `{ address: "123 Main St, San Francisco CA, USA", floors: 2, price: 5000000, owner: "John Doe" }`
4.  Call the `parseHouse` function and pass in the `house` variable.
5.  Log the output.

**Code**

##### index.js:

```

function parseHouse（property）{

return `${property.owner}正在以${property.address}的价格出售该物业${property.price}美元`

}

const house = {

地址：“123 Main St，San Francisco CA，USA”，

floors：2，

price：5000000，

owner：“John Doe”

};

console.log（parseHouse（house））;

```php

###### Snippet 1.29: Template literal using expressions

[`bit.ly/2RklKKH`](https://bit.ly/2RklKKH)

**Outcome**

![Figure 1.8: Template literal output](img/Figure_1.8.jpg)

###### Figure 1.8: Template literal output

You have successfully utilized a template literal to pretty-print an object.

In this section, we covered template literals. Template literals upgrade strings by allowing us to nest expressions inside them that are parsed at runtime. Expressions are inserted with the following syntax: ``${ expression }``. We then showed you how to escape special characters in template literals and discussed how in-editor newline characters in template literals are parsed as newline characters in the output. Finally, we covered template literal tagging and tagging functions, which allow us to perform more complex template literal parsing and creation.

## Enhanced Object Properties

ECMAScript 6 added several enhancements to object literals as part of the **ES6 syntactic sugar**. ES6 added three ways to simplify the creation of object literals. These simplifications include a more concise syntax for initializing object properties from variables, a more concise syntax for defining function methods, and computed object property names.

#### Note

Syntactic sugar is a syntax that is designed to make expressions easier to read and express. It makes the syntax "sweeter" because code can be expressed concisely.

### Object Properties

The shorthand for initializing object properties allows you to make more concise objects. In ES5, we needed to define the object properties with a key name and a value, as shown in the following code:

```

function getPersionES5（name，age，height）{

return {

name：name，

年龄：年龄，

height：height

};

}

getPersionES5（'Zachary'，23，195）

//预期输出：{name：'Zachary'，age：23，height：195}

```php

###### Snippet 1.30: ES5 object properties

Notice the repetition in the object literal returned by the function. We name the property in the object after variable name causing duplication (`<code>name: name</code>`). In ES6, we can shorthand each property and remove the repetition. In ES6, we can simply state the variable in the object literal declaration and it will create a property with a key that matches the variable name and a value that matches the variable value. This is shown in the following code:

```

function getPersionES6（name，age，height）{

return {

name，

age，

height

};

}

getPersionES6（'Zachary'，23，195）

//预期输出：{name：'Zachary'，age：23，height：195}

```php

###### Snippet 1.31: ES6 object properties

As you can see, both the ES5 and ES6 examples output the exact same object. However, in a large object literal declaration, we can save a lot of space and repetition by using this new shorthand.

### Function Declarations

ES6 also added a shorthand for declaring function methods inside objects. In ES5, we had to state the property name, then define it as a function. This is shown in the following example:

```

function getPersonES5（name，age，height）{

return {

name：name，

height：height，

getAge：function（）{ return age; }

};

}

getPersonES5（'Zachary'，23，195）.getAge（）

//预期输出：23

```php

###### Snippet 1.32: ES5 function properties

In ES6, we can define a function but with much less work. As with the property declaration, we don't need a key and value pair to create the function. The function name becomes the key name. This is shown in the following code:

```

function getPersionES6（name，age，height）{

return {

name，

height，

getAge（）{ return age; }

};

}

getPersionES6（'Zachary'，23，195）.getAge（）

//预期输出：23

```php

###### Snippet 1.33: ES6 function properties

Notice the difference in the function declaration. We omit the function keyword and the colon after the property key name. Once again, this saves us a bit of space and simplifies things a little.

### Computed Properties

ES6 also added a new, efficient way to create property names from variables. This is through computed property notation. As we already know, in ES5, there is only one way to create a dynamic property whose name is specified by a variable; this is through bracket notation, that is, `: obj[ expression ] = 'value'` . In ES6, we can use this same type of notation during the object literal's declaration. This is shown in the following example:

```

const varName = 'firstName';

const person = {

[varName] = 'John'，

lastName：'Smith'

};

console.log（person.firstName）; //预期输出：John

```php

###### Snippet 1.34: ES6 Computed property

As we can see from the preceding snippet, the property name of `varName` was computed to be `firstName`. When accessing the property, we simply reference it as `person.firstName`. When creating computed properties in object literals, the value that's computed in the brackets does not need to be a variable; it can be almost any expression, even a function. An example of this is shown in the following code:

```

const varName = 'first';

function computeNameType（type）{

return type + 'Name';

}

const person = {

[ varName + 'Name' ] = 'John'，

[ computeNameType（'last'）]：'Smith'

};

console.log（person.firstName）; //预期输出：John

console.log（person.lastName）; //预期输出：Smith

```php

###### Snippet 1.35: Computed property from function

In the example shown in the preceding snippet, we created two variables. The first contains the string `first` and the second contains a function that returns a string. We then created an object and used computed property notation to create dynamic object key names. The first key name is equal to `firstName`. When `person.firstName` is accessed, the value that was saved will be returned. The second key name is equal to `lastName`. When `person.lastName` is accessed, the value that was saved will be returned.

In summary, ES6 added three ways to simplify the declaration of object literals, that is, property notation, function notation, and computed properties. To simplify property creation in objects, when properties are created from variables, we can omit the key name and the colon. The name property that's created is set to the variable name and the value is set to the value of the variable. To add a function as a property to an object, we can omit the colon and function keyword. The name of the property that's created is set to the function name and the value of the property is the function itself. Finally, we can create property names from computed expressions during the declaration of the object literal. We simply replace the key name with the expression in brackets. These three simplifications can save us space in our code and make object literal creation easier to read.

### Exercise 7: Implementing Enhanced Object Properties

You are building a simple JavaScript math package to publish to **Node Package Manager (NPM)**. Your module will export an object that contains several constants and functions. Using ES6 syntax, create the export object with the following functions and values: the value of pi, the ratio to convert inches to feet, a function that sums two arguments, and a function that subtracts two arguments. Log the object after it has been created.

To create objects using ES6 enhanced object properties and demonstrate the simplified syntax, perform the following steps:

1.  Create an object and save it into the `exportObject` variable.
2.  Create a variable called `PI` that contains the value of pi (3.1415).
3.  Create a variable called `INCHES_TO_FEET` and save the value of the inches to feet conversion ratio (0.083333).

    Using ES6 enhanced property notation, add a property called `PI` from the variable PI. Add a property called `INCHES_TO_FEET` from the `INCHES_TO_FEET` variable, which contains the inches to feet conversion ratio.

    Add a function property called `sum` that takes in two input arguments and returns the sum of the two input arguments.

    Add a function property called `subtract` that takes in two input arguments and returns the subtraction of the two input arguments.

4.  Log the object `exportObject`.

**Code**

##### index.js:

```

const PI = 3.1415;

const INCHES_TO_FEET = 0.083333;

const exportObject = {

PI，

INCHES_TO_FEET，

sum（n1，n2）{

return n1 + n2;

}，

subtract（n1，n2）{

return n1 - n2;

}

};

console.log（exportObject）;

```php

###### Snippet 1.36: Enhanced object properties

[`bit.ly/2RLdHWk`](https://bit.ly/2RLdHWk)

**Outcome**

![Figure 1.9: Enhanced object properties output](img/Figure_1.9.jpg)

###### Figure 1.9: Enhanced object properties output

You have successfully created objects using ES6 enhanced object properties.

In this section, we showed you enhanced object properties, a syntactic sugar to help condense object property creation into fewer characters. We covered the shorthand for initializing object properties from variables and functions, and we covered the advanced features of computed object properties, that is, a way to create an object property name from a computed value, inline, while defining the object.

## Destructuring Assignment

**Destructuring assignment** is syntax in JavaScript that allows you to unpack values from arrays or properties from objects, and save them into variables. It is a very handy feature because we can extract data directly from arrays and objects to save into variables, all on a single line of code. It is powerful because it enables us to extract multiple array elements or object properties in the same expression.

### Array Destructuring

**Array destructuring** allows us to extract multiple array elements and save them into variables. In ES5, we do this by defining each variable with its array value, one variable at a time. This makes the code lengthy and increases the time required to write it.

In ES6, to destructure an array, we simply create an array containing the variable to assign data into, and set it equal to the data array being destructured. The values in the array are unpacked and assigned to the variables in the left-hand side array from left to right, one variable per array value. An example of basic array destructuring is shown in the following code:

```

let names = ['John'，'Michael'];

let [ name1，name2 ] = names;

console.log（name1）; //预期输出：'John'

console.log（name2）; //预期输出：'Michael'

```php

###### Snippet 1.37: Basic array destructuring

As can be seen in this example, we have an array of names and we want to destructure it into two variables, `name1` and `name2`. We simply surround the variables `name1` and `name2` with brackets and set that expression equal to the data array names, and then JavaScript will destructure the `names` array, saving data into each of the variables.

The data is destructured from the input array into the variables from left to right, in the order of array items. The first index variable will always be assigned the first index array item. This leads to the question, what do we do if we have more array items than variables? If there are more array items than variables, then the remaining array items will be discarded and will not be destructured into variables. The destructuring is a one to one mapping in array order.

What about if there are more variables than array items? If we attempt to destructure an array into an array that contains more variables than the total number of array elements in the data array, some of the variables will be set to undefined. The array is destructured from left to right. Accessing a non-existent element in a JavaScript array results in an undefined value to be returned. This undefined value is saved to the leftover variables in the variable array. An example of this is shown in the following code:

```

let names = ['John'，'Michael'];

let [ name1 ] = names

let [name2，name3，name4] = names;

console.log（name1）; //预期输出：'John'

console.log（name2）; //预期输出：'John'

console.log（name3）; //预期输出：'Michael'

console.log（name4）; //预期输出：未定义

```php

###### Snippet 1.38: Array destructuring with mismatched variable and array items

#### Note

We must be careful when destructuring arrays to make sure that we don't unintentionally assume that a variable will contain a value. The value of the variable could be set to undefined if the array is not long enough.

ES6 array destructuring allows for skipping array elements. If we have an array of values and we only care about the first and third values, we can still destructure the array. To ignore a value, simply omit the variable identifier for that array index in the left-hand side of the expression. This syntax can be used to ignore a single item, multiple items, or even all the items in an array. Two examples of this are shown in the following snippet:

```

let names = ['John'，'Michael'，'Jessica'，'Susan'];

let [name1，，name3] = names;

//注意第二个数组项的缺少变量名

let [ ,,, ] = names; //忽略数组中的所有项目

console.log（name1）; //预期输出：'John'

console.log（name3）; //预期输出：'Jessica'

```php

###### Snippet 1.39: Array destructuring with skipped values

Another very useful feature of array destructuring is the ability to set default values for variables that are created with destructuring. When we want to add a default value, we simply need to set the variable equal to the desired default value in the left-hand side of the destructuring expression. If what we are destructuring does not contain an index to assign to the variable, then the default value will be used instead. An example of this is shown in the following code:

```

let [a = 1，b = 2，c = 3] = ['cat'，null];

console.log（a）; //预期输出：'cat'

console.log（b）; //预期输出：null

console.log（c）; //预期输出：3

```php

###### Snippet 1.40: Array destructuring with skipped values

Finally, array destructuring can be used to easily swap values of variables. If we wish to swap the value of two variables, we can simply destructure an array into the reversed array. We can create an array containing the variables we want to reverse and set it equal to the same array, but with the variable order changed. This will cause the references to be swapped. This is shown in the following code:

```

let a = 10;

let b = 5;

[a，b] = [b，a];

console.log（a）; //预期输出：5

console.log（b）; //预期输出：10

```php

###### Snippet 1.41: Array destructuring with skipped values

### Exercise 8: Array Destructuring

To extract values from an array using array destructuring assignment, perform the following steps:

1.  Create an array with three values, `1`, `2`, and `3`, and save it into a variable called `data`.
2.  Destructure the array created with a single expression.

    Destructure the first array value into a variable called `a`. Skip the second value of the array.

    Destructure the third value into a variable called `b`. Attempt to destructure a fourth value into a variable called `c` and provide a default value of `4`.

3.  Log the value of all of the variables.

**Code**

##### index.js:

```

const data = [1, 2, 3];

const [ a，，b，c = 4 ] = data;

console.log（a，b，c）;

```php

###### Snippet 1.42: Array destructuring

[`bit.ly/2D2Hm5g`](https://bit.ly/2D2Hm5g)

**Outcome**

![Figure 1.10: Destructured variable's output](img/Figure_1.51.jpg)

###### Figure 1.10: Destructured variable's output

You have successfully applied an array destructuring assignment to extract values from an array.

In summary, array destructuring allows us to quickly extract values from arrays and save them into variables. Variables are assigned to array values, item by item, from left to right. If the number of variables exceeds the number of array items, then the variables are set to undefined, or the default value if specified. We can skip an array index in the destructuring by leaving a hole in the variables array. Finally, we can use destructuring assignment to quickly swap the values of two or more variables in a single line of code.

### Rest and Spread Operators

ES6 also introduces two new operators for arrays called **rest**and **spread**. The rest and spread operators are both denoted with three ellipses or periods before an identifier ( `...array1` ). The rest operator is used to represent an infinite number of arguments as an array. The spread operator is used to allow an iterable object to be expanded into multiple arguments. To identify which is being used, we must look at the item that the argument is being applied to. If the operator is applied to an iterable object (array, object, and so on), then it is the spread operator. If the operator is applied to function arguments, then it is the rest operator.

#### Note

In JavaScript, something considered iterable if something (generally values or key/value pairs) can be stepped through one at a time. For example, an array is iterable because the items in the array can be stepped through one at a time. Objects are considered iterable because the key/value pairs can be stepped through one at a time.

The **rest operator** is used to represent an indefinite number of arguments as an array. When the last parameter of a function is prefixed with the three ellipses, it becomes an array. The array elements are supplied by the actual arguments that are passed into the function, excluding the arguments that already have been given a separate name in the formal declaration of the function. An example of rest destructuring is shown in the following code:

```

function fn（num1，num2，...args）{

//将不确定数量的函数参数解构为

//数组 args，不包括传入的前两个参数。

console.log（num1）;

console.log（num2）;

console.log（args）;

}

fn（1, 2, 3, 4, 5, 6）;

//预期输出

// 1

// 2

// [3, 4, 5, 6]

```php

###### Snippet 1.43: Array destructuring with skipped values

Similar to the **arguments object** of a JavaScript function, the rest operator contains a list of function arguments. However, the rest operator has three distinct differences from the arguments object. As we already know, the arguments object is an array-like object that contains each argument that's passed into the function. The differences are as follows. First, the rest operator contains only the input parameters that have not been given a separate formal declaration in the function expression.

Second, the arguments object is not an instance of an **Array** object. The rest parameter is an instance of an array, which means that array functions like `sort()`, `map()`, and `forEach()` can be applied to them directly.

Lastly, the arguments object has special functionality that the rest parameter does not have. For example, the caller property exists on the arguments object.

The rest parameter can be destructured similar to how we destructure an array. Instead of putting a single variable name inside before the ellipses, we can replace it with an array of variables we want to fill. The arguments passed into the function will be destructured as expected for an array. This is shown in the following code:

```

function fn（...[n1，n2，n3]）{

//将不确定数量的函数参数解构为

//数组 args，解构为 3 个变量

console.log（n1，n2，n3）;

}

fn（1, 2）; //预期输出：1, 2, 未定义

```php

###### Snippet 1.44: Destructured rest operator

The spread operator allows an iterable object such as an array or string to be expanded into multiple arguments (for function calls), array elements (for array literals), or key-value pairs (for object expressions). This essentially means that we can expand an array into arguments for creating another array, object, or calling a function. An example of spread syntax is shown in the following code:

```

function fn（n1，n2，n3）{

console.log（n1，n2，n3）;

}

const values = [1, 2, 3];

fn（...values）; //预期输出：1, 2, 3

```php

###### Snippet 1.45: Spread operator

In the preceding example, we created a simple function that takes in three inputs and logs them to the console. We created an array with three values, then called the function using the `spread` operator to destructure the array of values into three input parameters for the function.

The rest operator can be used in destructuring objects and arrays. When destructuring an array, if we have more array elements than variables, we can use the rest operator to capture, or catch, all of the additional array elements during destructuring. When using the rest operator, it must be the last parameter in the array destructuring or function arguments list. This is shown in the following code:

```

const [n1，n2，n3，...remaining] = [1, 2, 3, 4, 5, 6];

console.log（n1）; //预期输出：1

console.log（n2）; //预期输出：2

console.log（n3）; //预期输出：3

console.log（remaining）; //预期输出：[4, 5, 6]

```php

###### Snippet 1.46: Spread operator

In the preceding snippet, we destructured the first three array elements into three variables, `n1`, `n2`, and `n3`. We then captured the remaining array elements with the rest operator and destructured them into the variable that remained.

In summary, the rest and spread operators allow iterable entities to be expanded into many arguments. They are denoted with three ellipses before the identifier name. This allows us to capture arrays of arguments in functions or unused items when destructuring entities. When we use the rest and spread operators, they must be the last arguments that are passed into the expression they are being used in.

### Object Destructuring

**Object destructuring** is used in a very similar way to array destructuring. Object destructuring is used to extract data from an object and assign the values to new variables. In ES6, we can do this in a single JavaScript expression. To destructure an object, we surround the variables we want to destructure with curly braces (`{}`), and set that expression equal to the object we are destructuring. A basic example of object destructuring is shown in the following code:

```

const obj = {firstName：'Bob'，lastName：'Smith'};

const { firstName, lastName } = obj;

console.log( firstName ); // 期望输出：'Bob'

console.log( lastName ); // 期望输出：'Smith'

```php

###### Snippet 1.47: Object destructuring

In the preceding example, we created an object with the keys `firstName` and `lastName`. We then destructured this object into the variables `firstName` and `lastName`. Notice that the names of the variables and the object parameters match. This is shown in the following example:

#### Note

When doing basic object destructuring, the name of the parameter in the object and the name of the variable we are assigning it to must match. If there is no matching parameter for a variable we are trying to destructure, then the variable will be set to undefined.

```

const obj = { firstName: 'Bob', lastName: 'Smith' };

const { firstName, middleName } = obj;

console.log( firstName ); // 期望输出：'Bob'

console.log( middleName ); // 期望输出：未定义

```php

###### Snippet 1.48: Object destructuring with no defined key

As we saw, the `middleName` key does not exist in the object. When we try to destructure the key and save it into the variable, it is unable to find a value and the variable is set to undefined.

With advanced object destructuring syntax, we can save the key that's extracted into a variable with a different name. This is done by adding a colon and the new variable name after the key name in the destructuring notation. This is shown in the following code:

```

const obj = { firstName: 'Bob', lastName: 'Smith' };

const { firstName: first, lastName } = obj;

console.log( first ); // 期望输出：'Bob'

console.log( lastName ); // 期望输出：'Smith'

```php

###### Snippet 1.49: Object destructuring into new variable

In the preceding example, we could clearly see that we are destructuring the `firstname` key from the object and saving it into the new variable, called first. The `lastName` key is being destructured normally and is saved into a variable called `lastName`.

Much like with array destructuring, we can destructure an object and provide a default value. If a default value is provided and the key we are attempting to destructure does not exist in the object, then the variable will be set to the default value instead of undefined. This is shown in the following code:

```

const obj = { firstName: 'Bob', lastName: 'Smith' };

const { firstName = 'Samantha', middleName = 'Chris' } = obj;

console.log( firstName ); // 期望输出：'Bob'

console.log( middleName ); // 期望输出：'Chris'

```php

###### Snippet 1.50: Object destructuring with default values

In the preceding example, we set the default values for both of the variables we are trying to destructure from the object. The default value for `firstName` is specified, but the `firstName` key exists in the object. This means that the value stored in the `firstName` key is destructured and the default value is ignored. The `middleName` key does not exist in the object and we have specified a default value to use when destructuring. Instead of using the undefined value of the `firstName` key, the destructuring assignment sets the destructured variable to the default value of `Chris`.

When we are providing a default value and assigning the key to a new variable name, we must put the default value assignment after the new variable name. This is shown in the following example:

```

const obj = { firstName: 'Bob', lastName: 'Smith' };

const { firstName: first = 'Samantha', middleName: middle = 'Chris' } = obj;

console.log( first ); // 期望输出：'Bob'

console.log( middle); // 期望输出：'Chris'

```php

###### Snippet 1.51: Object destructuring into new variables with default values

The `firstName` key exists. The value of `obj.firstName` is saved into the new variable named `first`. The `middleName` key does not exist. This means that the new variable middle is created and set to the default value of `Chris`.

### Exercise 9: Object Destructuring

To extract data from an object by using object destructuring concepts, perform the following steps:

1.  Create an object with the fields `f1`, `f2`, and `f3`. Set the values to `v1`, `v2`, and `v3`, respectively. Save the object into the `data` variable.
2.  Destructure this object into variables with a single statement, as follows:

    Destructure the `f1` property into a variable named `f1.` Destructure the `f2` property into a variable named `field2.` Destructure the property `f4` into a variable named `f4` and provide a default value of `v4`.

3.  Log the variables that are created.

**Code**

##### index.js:

```

const data = { f1: 'v1', f2: '2', f3: 'v3' };

const { f1, f2: field2, f4 = 'v4' } = data;

console.log( f1, field2, f4 );

```php

###### Snippet 1.52: Object destructuring

[`bit.ly/2SJUba9`](https://bit.ly/2SJUba9)

**Outcome**

![Figure 1.11: Created variable's output](img/Figure_1.11.jpg)

###### Figure 1.11: Created variable's output

You have successfully applied object destructuring concepts to extract data from an object.

JavaScript requires special syntax if we declare the variables before the object destructuring expression. We must surround the entire object destructuring expression with parentheses. This syntax is not required for array destructuring. This is shown in the following code:

```

const obj = { firstName: 'Bob', lastName: 'Smith' };

let firstName, lastName;

（{ firstName: first, lastName } = obj）;

// 注意表达式周围的括号

console.log( firstName ); // 期望输出：'Bob'

console.log( lastName ); // 期望输出：'Smith'

```php

###### Snippet 1.53: Object destructuring into predefined variables

#### Note

Make sure that object destructuring done in this way is preceded by a semicolon on the same or previous line. This prevents the JavaScript interpreter from interpreting the parentheses as a function call.

The **rest operator** can also be used to destructure objects. Since object keys are iterable, we can use the rest operator to catch the remaining keys that were uncaught in the original destructuring expression. This is done similar to arrays. We destructure the keys that we want to capture, and then we can add the rest operator to a variable and catch the remaining key/value pairs that have not been destructured out of the object. This is shown in the following example:

```

const obj = { firstName: 'Bob', middleName: 'Chris', lastName: 'Smith' };

const { firstName, ...otherNames } = obj;

console.log( firstName ); // 期望输出：'Bob'

console.log( otherNames );

// 期望输出：{ middleName: 'Chris', lastName: 'Smith' }

```php

###### Snippet 1.54: Object destructuring with the rest operator

In summary, object destructuring allows us to quickly extract values from objects and save them into variables. The key name must match the variable name in simple object destructuring, however we can use more advanced syntax to save the key's value into a new object. If a key is not defined in the object, then the variable will be set to `false`, that is, unless we provide it with a default value. We can save this into predefined variables, but we must surround the destructuring expression with parentheses. Finally, the rest operator can be used to capture the remaining key value pairs and save them in a new object.

Object and array destructuring support nesting. Nesting destructuring can be a little confusing, but it is a powerful tool because it allows us to condense several lines of destructuring code into a single line.

### Exercise 10: Nested Destructuring

To destructure values from an array that's nested inside an object using the concept of nested destructuring, perform the following steps:

1.  Create an object with a property, `arr`, that is, set to an array containing the values `1`, `2`, and `3`. Save the object into the `data` variable.
2.  Destructure the second value of the array into a variable by doing the following:

    Destructure the `arr` property from the object and save it into a new variable called `v2`, which is the array. Replace `v2` with array destructuring.

    In the array destructuring, skip the first element. Save the second element into a variable called `v2`.

3.  Log the variable.

**Code**

##### index.js:

```

const data = { arr: [ 1, 2, 3 ] };

const { arr: [ , v2 ] } = data;

console.log( v2 );

```php

###### Snippet 1.55: Nested array and object destructuring

[`bit.ly/2SJUba9`](https://bit.ly/2SJUba9)

**Outcome**

![Figure 1.12: Nested destructuring output](img/Figure_1.12.jpg)

###### Figure 1.12: Nested destructuring output

You have successfully destructured values from an array inside an object.

In summary, object and array destructuring was introduced into ES6 to cut down code and allow for the quick creation of variables from objects and arrays. Array destructuring is denoted by setting an array of variables equal to an array of items. Object destructuring is denoted by setting an object of variables equal to an object of key value pairs. Destructuring statements can be nested for even greater effect.

### Exercise 11: Implementing Destructuring

You have registered for university courses and need to buy the texts required for the classes. You are building a program to scrape data from the book list and obtain the ISBN numbers for each text book that's required. Use object and array nested destructuring to obtain the ISBN value of the first text of the first book in the courses array. The courses array follows the following format:

```

[

{

title: 'Linear Algebra II',

description: 'Advanced linear algebra.',

texts: [ {

author: 'James Smith',

price: 120,

ISBN: '912-6-44-578441-0'

} ]

},

{ ... },

{ ... }

]

```php

###### Snippet 1.56: Course array format

To obtain data from complicated array and object nesting by using nested destructuring, perform the following steps:

1.  Save the provided data structure into the `courseCatalogMetadata` variable.
2.  Destructure the first array element into a variable called `course`:

```

[ course ] = [ … ]

```php

3.  Replace the `course` variable with object destructuring to save the texts field into a variable called `textbooks`:

```

[ { texts: textbooks} ] = [ … ]

```php

4.  Replace the `textbooks` variable with array destructuring to get the first element of the texts array and save it into the variable called `textbook`:

```

[ { texts: [ textbook ] } ] = [ … ]

```php

5.  Replace the `textbook` variable with object destructuring to get the `ISBN` field and save it into the `ISBN` variable:

```

[ { texts: [ { ISBN } ] } ] = [ … ]

```php

6.  Log the value of the `ISBN`.

**Code**

##### index.js:

```

const courseCatalogMetadata = [

{

title: 'Linear Algebra II',

description: 'Advanced linear algebra.',

texts: [ {

author: 'James Smith',

price: 120,

ISBN: '912-6-44-578441-0'

} ]

}

];

const [ course ] = courseCatalogMetadata;

const [ { texts: textbooks } ] = courseCatalogMetadata;

const [ { texts: [ textbook ] } ] = courseCatalogMetadata;

const [ { texts: [ { ISBN } ] } ] = courseCatalogMetadata;

console.log( course );

console.log( textbooks );

console.log( textbook );

console.log( ISBN );

```php

###### Snippet 1.57: Implementing destructuring into code

[`bit.ly/2TMlgtz`](https://bit.ly/2TMlgtz)

**Outcome**

![Figure 1.13: Array destructuring output](img/Figure_1.13.jpg)

###### Figure 1.13: Array destructuring output

You have successfully obtained data from arrays and objects using destructuring and nested destructuring.

In this section, we discussed destructuring assignment for arrays and objects. We demonstrated how array and object destructuring simplifies code and allows us to quickly extract values from objects and arrays. Destructuring assignment allows us to unpack values from objects and arrays, provide default values, and rename object properties as variables when destructuring. We also introduced two new operators— the rest and spread operators. The rest operator was used to represent an indefinite number of arguments as an array. The spread operator was used to break an iterable object into multiple arguments.

## Classes and Modules

Classes and Modules were added to ES6\. Classes were introduced as a way to expand on prototype-based inheritance by adding some object oriented concepts. Modules were introduced as a way to organize multiple code files in JavaScript and expand on code reusability and scoping among files.

### Classes

**Classes** were added to ECMAScript 6 primarily as syntactic sugar to expand on the existing prototype-based inheritance structure. Class syntax does not introduce object oriented inheritance to JavaScript. Class inheritance in JavaScript do not work like classes in object oriented languages.

In JavaScript, a class can be defined with the keyword class. A class is created by calling the keyword class, followed by the class name and curly braces. Inside the curly braces, we define all of the functions and logic for the class. The syntax is as follows:

```

class name { /* class stuff goes here */ }

```php

###### Snippet 1.58: Class syntax

A class can be created with the **optional function constructor**. The constructor, if not necessary for a JavaScript class, but there can only be one method with the name constructor in a class. The constructor is called when an instance of the class in initialized and can be used to set up all of the default internal values. An example of a class declaration is shown in the following code:

```

class House{

constructor(address, floors = 1, garage = false) {

this.address = address;

this.floors = floors;

this.garage = garage;

}

}

```php

###### Snippet 1.59: Basic class creation

In the example, we create a class called `House`. Our `House` class has a `constructor` method. When we instantiate the class, it calls the constructor. Our `constructor` method takes in three parameters, two of them with default values. The constructor saves these values to variables in the `this` scope.

The keyword this is mapped to each class instantiation. It is a global scope class object. It is used to scope all functions and variables globally inside a class. Every function that is added at the root of the class will be added to the `this` scope. All the variables that is added to the `this` scope will be accessible inside any function inside the class. Additionally, anything added to the `this` scope is accessible publicly outside of the class.

### Exercise 12: Creating Your Own Class

To create a simple class and demonstrate internal class variables, perform the following steps:

1.  Declare a class called `Vehicle`.
2.  Add a constructor function to the class. Have the constructor take in two variables, `wheels` and `topSpeed`.
3.  In the constructor, save the input variables to two variables in the `this` scope, that is, `this.wheels` and `this.topSpeed`.
4.  Instantiate the class with `wheels = 3` and `topSpeed = 20` and save it into the `tricycle` variable.
5.  Log the value for wheels and `topSpeed` from the class that was saved in `tricycle`.

**Code**

##### index.js:

```

class Vehicle {

constructor( wheels, topSpeed ) {

this.wheels = wheels;

this.topSpeed = topSpeed;

}

}

const tricycle = new Vehicle( 3, 20 );

console.log( tricycle.wheels, tricycle.topSpeed );

```php

###### Snippet 1.60: Creating a class

[`bit.ly/2FrpL8X`](https://bit.ly/2FrpL8X)

**Outcome**

![Figure 1.14: Creating classes output](img/Figure_1.14.jpg)

###### Figure 1.14: Creating classes output

You have successfully created a simple class with values.

We instantiated a new instance of a class with the new keyword. To create a new class, simply declare a variable and set it equal to the expression `new className()`. When we instantiate a new class, the parameters that are passed into the class call are passed into the constructor function, if one exists. An example of a class instantiation is shown in the following code:

```

class House{

constructor(address, floors = 1) {

this.address = address;

this.floors = floors;

}

}

// 实例化类

let myHouse = new House( '1100 Fake St., San Francisco CA, USA', 2, false );

```php

###### Snippet 1.61: Class instantiation

In this example, the class instantiation happens on the line with the new keyword. This line of code creates a new instance of the `House` class and saves it into the `myHouse` variable. When we instantiate the class, we are providing the parameters for `address`, `floors`, and `garage`. These value are passed into the constructor and then saved into the instantiated class object.

To add functions to a class, we declare them with the new ES6 object function declaration. As a quick reminder, when using the new ES6 object function declaration, we can omit the function keyword and object key name. When a function is added to an object, it is automatically attached to the `this` scope. Additionally, all functions that are added to the class have access to the `this` scope and will be able to call any function and access any variable attached to the `this` scope. An example of this is shown in the following code:

```

class House{

constructor( address, floors = 1) {

this.address = address;

this.floors = floors;

}

getFloors() {

return this.floors;

}

}

let myHouse = new House( '1100 Fake St., San Francisco CA, USA', 2 );

console.log( myHouse.getFloors() ); // 期望输出：2

```php

###### Snippet 1.62: Creating a class with functions

As we can see from this example, the two functions `getFloors` and `setFloors` were added with the new ES6 enhanced object property syntax for function declarations. Both functions have access to the `this` scope. They can get and set variables in that scope, as well as call functions that have been attached to the `this` scope.

In ES6, we can also create subclasses using the `extends` keyword. **Subclasses** inherit properties and methods from the parent class. A subclass is defined by following the class name with the keyword `extends` and the name of the parent class. An example of a subclass declaration is shown in the following code:

```

class House {}

class Mansion extends House {}

```php

###### Snippet 1.63: Extending a class

### Classes – Subclasses

In this example, we will create a class called `House`, and then we will create a subclass called `Mansion` that extends the class `House`. When we create a subclass, we need to take note of the behavior of the constructor method. If we provide a constructor method, then we must call the `super()` function. `super` is a function that calls the constructor of the parent object. If we try to access the `this` scope without a call to call `super`, then we will get a runtime error and our code will crash. Any parameters that are required by the parent constructor can be passed in through the `super` method. If we do not specify a constructor for the subclass, the default constructor behavior will automatically call the super constructor. An example of this is shown in the following code:

```

class House {

constructor( address = 'somewhere' ) {

this.address = address;

}

}

class Mansion extends House {

constructor( address, floors ) {

super( address );

this.floors = floors;

}

}

let mansion = new Mansion( 'Hollywood CA, USA', 6, 'Brad Pitt' );

console.log( mansion.floors ); // 期望输出：6

```php

###### Snippet 1.64: Extending a class with and without a constructor

In this example, we created a subclass that extended our `House` class. The `Mansion` subclass has a defined constructor, so we must call super before we can access the `this` scope. When we call `super`, we pass the address parameter to the parent constructor, which adds it to the `this` scope. The constructor for `Mansion` then continues execution and adds the floors variable to the `this` scope. As we can see from the output logging at the end of this example, the subclass's `this` scope also includes all variables and functions that were created in the parent class. If a variable or function is redefined in the subclass, it will overwrite the inherited value or function from the parent class.

In summary, classes allow us to expand on the prototype-based inheritance of JavaScript by introducing some object oriented concepts. Classes are defined with the keyword `class` and initialized with the keyword `new`. When a class is defined, a special scope called `this` is created for it. All items in the `this` scope are publicly accessible outside the class. We can add functions and variables to the `this` scope to give our class functionality. When a class is instantiated, the constructor is called. We can also extend classes to create subclasses with the `extends` keyword. If an extended class has a constructor, we must call the super function to call its parent-class constructor. Subclasses have access to the parent class methods and variables.

### Modules

Almost every coding language has a concept of modules. **Modules** are features that allow the programmer to break code into smaller independent parts that can be imported and reused. Modules are critical for the design of programs and are used to prevent code duplication and reduce file size. Modules did not exist in vanilla JavaScript until ES6\. Moreover, not all JavaScript interpreters support this feature.

Modules are a way to reference other code files from the current file. Code can be broken into multiple parts, called **modules**. Modules allow us to keep unrelated code separate so that we can have smaller and simpler files in our large JavaScript projects.

Modules also allow the contained code to be quickly and easily shared without any code duplication. Modules in ES6 introduced two new keywords, `export` and `import`. These keywords allow us to make certain classes and variables publicly available when a file is loaded.

#### Note

JavaScript modules do not have full support across all platforms. At the time of writing this book, not all JavaScript frameworks could support modules. Make sure that the platforms you are releasing your code on can support the code you have written.

### Export Keyword

Modules use the `export` keyword to expose variables and functions contained in the file. Everything inside an ES6 module is private by default. The only way to make anything public is to use the export keyword. Modules can export properties in two ways, via **named exports** or **default exports**. Named exports allow for multiple exports per module. Multiple exports may be useful if you are building a math module that exports many functions and constants. Default exports allow for just a single export per model. A single export may be useful if you are building a module that contains a single class.

There are two ways to expose the named contents of a module with the `export` keyword. We can export each item individually by preceding the variable or function declaration with the `export` keyword, or we can export an object containing the key value pairs that reference each variable and function we want exported. These two export methods are shown in the following example:

```

// math-module-1.js

export const PI = 3.1415;

export const DEGREES_IN_CIRCLE = 360;

export function convertDegToRad( degrees ) {

return degrees * PI / ( DEGREES_IN_CIRCLE /2 );

}

// math-module-2.js

const PI = 3.1415;

const DEGREES_IN_CIRCLE = 360;

function convertDegToRad( degrees ) {

return degrees * PI / ( DEGREES_IN_CIRCLE /2 );

}

export { PI, DEGREES_IN_CIRCLE, convertDegToRad };

```php

###### Snippet 1.65: Named Exports

Both of the modules outlined in the preceding example export three constant variables and one function. The first module, `math-module-1.js`, exports each item, one at a time. The second module, `math-module-2.js`, exports all of the exports at once via an object.

To export the contents of a module as a default export, we must use the **default** **keyword**. The `default` keyword comes after the `export` keyword. When we default export a module, we can also omit the identifier name of the class, function, or variable we are exporting. An example of this is shown in the following code:

```

// HouseClass.js

export default class() { /* Class body goes here */ }

// myFunction.js

export default function() { /* Function body goes here */ }

```php

###### Snippet 1.66: Default exports

In the preceding example, we created two modules. One exports a class and the other exports a function. Notice how we include the `default` keyword after the `export` keyword, and how we omit the name of the class/function. When we export a default class, the `export` is not named. When we are importing default export modules, the name of the object we are importing is derived via the module's name. This will be shown in the next section, where we will talk about the `import` keyword.

### Import Keyword

The `import` keyword allows you to import a JavaScript module. Importing a module allows you to pull any items from that module into the current code file. When we import a module, we start the expression with the `import` keyword. Then, we identify what parts of the module we are going to import. Then, we follow that with the `from` keyword, and finally we finish with the path to the module file. The `from` keyword and file path tell the interpreter where to find the module we are importing.

#### Note

ES6 modules may not have full support from all browsers versions or versions of Node.js. You may have to make use of a transpiler such as Babel to run your code on certain platforms.

There are four ways we can use the `import` keyword, all of which are shown in the following code:

```

// math-module.js

export const PI = 3.1415;

export const DEGREES_IN_CIRCLE = 360;

// index1.js

import { PI } from 'math-module.js'

// index2.js

import { PI, DEGREES_IN_CIRCLE } from 'math-module.js'

// index3.js

import { PI as pi, DEGREES_IN_CIRCLE as degInCircle } from 'math-module.js'

// index4.js

import * as MathModule from 'math-module.js'

```php

###### Snippet 1.67: Different ways to import a module

In the code shown in preceding snippet, we have created a simple module that exports a few constants and four import example files. In the first `import` example, we are importing a single value from the module exports and making it accessible in the variable API. In the second `import` example, we are importing multiple properties from the module. In the third example, we are importing properties and renaming them to new variable names. The properties can then be accessed from the new variables. In the fourth example, we are using a slightly different syntax. The asterisk signifies that we want to import all exported properties from the module. When we use the asterisk, we must also use the `as` keyword to give the imported object a variable name.

The process of importing and using modules is better explained through the following snippet:

```

// email-callback-api.js

export function authenticate( … ){ … }

export function sendEmail( … ){ … }

export function listEmails( … ){ … }

// app.js

import * as EmailAPI from 'email-callback-api.js';

const credentials = { password: '****', user: 'Zach' };

EmailAPI.authenticate( credentials, () => {

EmailAPI.send( { to: 'ceo@google.com', subject: 'promotion', body: 'Please promote me' }, () => {} );'

} );

```php

###### Snippet 1.68: Importing a module

To use an import in the browser, we must use the `script` tag. The module import can be done inline or via a source file. To import a module, we need to create a `script` tag and set the type property to `module`. If we are importing via a source file, we must set the `src` property to the file path. This is shown in the following syntax:

```

<script type="module" src="./path/to/module.js"></script>

```php

###### Snippet 1.69: Browser import inline

#### Note

The script tag is an HTML tag that allows us to run JavaScript code in the browser.

We can also import modules inline. To do this, we must omit the `src` property and code the import directly in the body of the script tag. This is shown in the following code:

```

<script type="module">

import * as ModuleExample from './path/to/module.js';

</script>

```php

###### Snippet 1.70: Browser import in script body

#### Note

When importing modules in browsers, browser versions that do not support ES6 modules will not run scripts with type="module".

If the browser does not support ES6 modules, we can provide a fallback option with the `nomodule` attribute. Module compatible browsers will ignore script tags with the `nomodule` attribute so that we can use it to provide fallback support. This is shown in the following code:

```

<script type="module" src="es6-module-supported.js"></script>

<script nomodule src="es6-module-NOT-supported.js"></script>

```php

###### Snippet 1.71: Browser import with compatibility option

In the preceding example, if the browser supports modules, then the first script tag will be run and the second will not. If the browser does not support modules, then the first script tag will be ignored, and the second will be run.

One final consideration for modules: be careful that any modules you build do not have circular dependencies. Because of the load order of modules, circular dependencies in JavaScript can cause lots of logic errors when ES6 is transpiled to ES5\. If there is a circular dependency in your modules, you should restructure your dependency tree so that all dependencies are linear. For example, consider the dependency chain: Module A depends on B, module B depends on C, and module C depends on A. This is a circular module chain because through the dependency chain, A depends on C, which depends on A. The code should be restructured so that the circular dependency chain is broken.

### Exercise 13: Implementing Classes

You have been hired by a car sales company to design their sales website. You must create a vehicle class to store car information. The class must take in the car make, model, year, and color. The car should have a method to change the color. To test the class, create an instance that is a grey (color) 2005 (year) Subaru (make) Outback (model). Log the car's variables, change the car's color, and log the new color.

To build a functional class to demonstrate the capabilities of a class, perform the following steps:

1.  Create a `car` class.

    Add a constructor that takes in the `make`, `model`, `year`, and `color`. Save the `make`, `model`, `year`, and `color` in internal variables (`this` scope) in the constructor function.

    Add a function called `setColor` that takes in a single parameter, color, and updates the internal variable `color` to the provided color.

2.  Instantiate the class with the parameters `Subaru`, `Outback`, `2005`, and `Grey`. Save the class into the `Subaru` variable.
3.  Log the internal variables, that is, `make`, `model`, `year`, and `color`, of the class stored in `Subaru.`
4.  Change the color with the `setColor` of the class stored in `Subaru` class method. Set the color to `Red`.
5.  Log the new color.

**Code**

##### index.js:

```

class Car {

constructor( make, model, year, color ) {

this.make = make;

this.model = model;

this.year = year;

this.color = color;

}

setColor( color ) {

this.color = color;

}

}

let subaru = new Car( 'Subaru', 'Outback', 2005, 'Grey' );

subaru.setColor( 'Red' );

```php

###### Snippet 1.72: Full class implementation

[`bit.ly/2FmaVRS`](https://bit.ly/2FmaVRS)

**Outcome**

![Figure 1.15: Implementing classes output](img/Figure_1.15.jpg)

###### Figure 1.15: Implementing classes output

You have successfully built a functional class.

In this section, we introduced JavaScript classes and ES6 modules. We discussed the prototype-based inheritance structure and demonstrated the basics of class creation and JavaScript class inheritance. When discussing modules, we first showed how to create a module and export the functions and variables stored within them. Then, we showed you how to load a module and import the data contained within. We ended this topic by discussing browser compatibility and providing HTML script tag options for supporting browsers that do not yet support ES6 modules.

## Transpilation

**Transpilation** is defined as source-to-source compilation. Tools have been written to do this and they are called transpilers. **Transpilers** take the source code and convert it into another language. Transpilers are important for two reasons. First, not every browser supports every new syntax in ES6, and second, many developers use programming languages based off of JavaScript, such as CoffeeScript or TypeScript.

#### Note

The ES6 compatibility table can be found at [`kangax.github.io/compat-table/es6/`](https://kangax.github.io/compat-table/es6/).

Looking at the ES6 browser compatibility table clearly shows us that there are some holes in support. A transpiler allows us to write our code in ES6 and translate it into vanilla ES5, which works in every browser. It is critical to ensure that our code works on as many web platforms as possible. Transpilers can be an invaluable tool for ensuring compatibility.

Transpilers also allow us to develop web or server side applications in other programming languages. Languages such as TypeScript and CoffeeScript may not run natively in the browser; however, with a transpiler, we are able to build a full application in these languages and translate them into JavaScript for server or browser execution.

One of the most popular transpilers for JavaScript is **Babel**. Babel is a tool that was created to aid in the transpilation between different versions of JavaScript. Babel can be installed through the node package manager (npm). First, open your terminal and path to the folder containing your JavaScript project.

If there is no `package.json` file in this directory, we must create it. This can be done with the `npm init` command. The command-line interface will ask you for several entries so that you can fill out the defaults of the `package.json` file. You can enter the values or simply press the return key and accept the default values.

To install the Babel command-line interface, use the following command: `npm install --save-dev babel-cli`. After that has concluded, the `babel-cli` field will have been added to the `devDependencies` object in the `package.json` file:

```

{

"devDependencies": {

"babel-cli": "⁶.26.0"

}

}

```php

###### Snippet 1.73: Adding the first dependency

This command only installed the base Babel with no plugins for transpiling between versions of JavaScript. To install the plugin to transpile to ECMAScript 2015, use the `npm install --save-dev babel-preset-es2015` command. Once the command finishes running, our `package.json` file will contain another dependency:

```

"devDependencies": {

"babel-cli": "⁶.26.0",

"babel-preset-es2015": "⁶.24.1"

}

```php

###### Snippet 1.74: Adding the second dependency

This installs the ES6 presets. To use these presets, we must tell Babel to configure itself with these presets. Create a file called `.babelrc`. Note the leading period in the name. The `.babelrc` file is Babel's configuration file. This is where we tell Babel what presets, plugins, and so on, we are going to use. Once created, add the following contents to the file:

```

{

"presets": ["es2015"]

}

```php

###### Snippet 1.75: Installing the ES6 presets

### Babel- Transpiling

Now that Babel has been configured, we must create the code file to transpile. In the root folder of your project, create a file called `app.js`. In this file, paste the following ES6 code:

```

const sum5 = inputNumber  => inputNumber + 5;

console.log( `The sum of 5 and 5 is ${sum5(5)}!`);

```php

###### Snippet 1.76: Pasting the code

Now that Babel has been configured and we have a file that we wish to transpile, we need to update our `package.json` file to add a transpile script for npm. Add the following lines to your `package.json` file:

```

"scripts": {

"transpile": "babel app.js --out-file app.transpiled.js --source-maps"

}

```php

###### Snippet 1.77: Update the package.json file

The scripts object allows us to run these commands from npm. We are going to name the npm script `transpile` and it will run the command chain `babel app.js --out-file app.transpiled.js --source-maps`. `App.js` is our input file. The `--out-file` command specifies the output file for compilation. `App.transpiled.js` is our output file. Lastly, `--source-maps` creates a source map file. This file tells the browser which line of transpiled code corresponds to which lines of the original source. This allows us to debug directly in the original source file, that is, `app.js`.

Now that we have everything set up, we can run our transpile script by typing `npm run transpile` into the terminal window. This will transpile our code from `app.js` into `app.transpiled.js`, creating or updating the file as needed. Upon examination, we can see that the code in `app.transpiled.js` has been converted into ES5 format. You can run the code in both files and see that the output is the same.

Babel has many plugins and presets for different modules and JavaScript releases. There are enough ways to set up and run Babel that we could write an entire book about it. This was just a small preview for converting ES6 code to ES5\. For full documentation and information on Babel and the uses of each plugin, visit the documentation.

#### Note

Take a look at Babel's home page at [`babeljs.io`](https://babeljs.io).

In summary, transpilers allow you to do source to source compiling. This is very useful because it allows us to compile ES6 code to ES5 when we need to deploy on a platform that does not yet support ES6\. The most popular and most powerful JavaScript transpiler is Babel. Babel can be set up on the command line to allow us to build entire projects in different versions of JavaScript.

### Exercise 14: Transpiling ES6 Code

Your office team has written your website code in ES6, but some devices that users are using do not support ES6\. This means that you must either rewrite your entire code base in ES5 or use a transpiler to convert it to ES5\. Take the ES6 code written in the *Upgrading Arrow Functions* section and transpile it into ES5 with Babel. Run the original and transpiled code and compare the output.

To demonstrate Babel's ability to transpile code from ES6 to ES5, perform the following steps:

Ensure that Node.js is already installed before you start.

1.  Install Node.js if it is not already installed.
2.  Set up a Node.js project with the command line command `npm init`.
3.  Put the code from the *Upgrading Arrow Functions* section into the `app.js` file.
4.  Install Babel and the Babel ES6 plugin with `npm install`.
5.  Configure Babel by adding a `.babelrc` file with the es2015 preset.
6.  Add a transpile script to `package.json` that calls Babel and transpiles from `app.js` to `app.transpiled.js`.
7.  Run the transpile script.
8.  Run the code in `app.transpiled.js`.

**Code**

##### package.json:

```

// File 1: package.json

{

"scripts": {

"transpile": "babel ./app.js --out-file app.transpiled.js --source-maps"

},

"devDependencies": {

"babel-cli": "⁶.26.0",

"babel-preset-es2015": "⁶.24.1"

}

}

```php

###### Snippet 1.78: Package.json config file

[`bit.ly/2FsjzgD`](https://bit.ly/2FsjzgD)

##### .babelrc:

```

// File 2: .babelrc

{ "presets": ["es2015"] }

```php

###### Snippet 1.79: Babel config file

[`bit.ly/2RMYWSW`](https://bit.ly/2RMYWSW)

##### app.transpiled.js:

```

// File 3: app.transpiled.js

var fn1 = function fn1(a, b) { … };

var fn2 = function fn2(a, b) { … };

var fn3 = function fn3(a) { … };

var fn4 = function fn4() { … };

var fn5 = function fn5(a) { … };

```php

###### Snippet 1.80: Fully transpiled code

[`bit.ly/2TLhuR7`](https://bit.ly/2TLhuR7)

**Outcome**

![Figure 1.16: Transpiled script output](img/Figure_1.16.jpg)

###### Figure 1.16: Transpiled script output

You have successfully implemented Babel's ability to transpile code from ES6 to ES5.

In this section, we discussed the concept of transpilation. We introduced the transpiler Babel and walked through how to install Babel. We discussed the basic steps to set up Babel to transpile ES6 into ES5 compatible code and, in the activity, built a simple Node.js project with ES6 code to test Babel.

## Iterators and Generators

In their simplest forms, **iterators** and **generators** are two ways to process a collection of data incrementally. They gain efficiency over loops by keeping track of the state of the collection instead of all of the items in the collection.

### Iterators

An **iterator** is a way to traverse through data in a collection. To iterate over a data structure means to step through each of its elements in order. For example, the `for/in` loop is a method that's used to iterate over the keys in a JavaScript object. An object is an iterator when it knows how to access its items from a collection one at a time, while tracking position and finality. An iterator can be used to traverse custom complicated data structure or for traversing chunks of large data that may not be practical to load all at once.

To create an iterator, we must define a function that takes a collection in as the parameter and returns an object. The return object must have a function property called next. When next is called, the iterator steps to the next value in the collection and returns an object with the value and the done status of the iteration. An example iterator is shown in the following code:

```

function createIterator( array ){

let currentIndex = 0;

return {

next(){

return currentIndex < array.length ?

{ value: array[ currentIndex++ ], done: false} :

{ done: true };

}

};

}

```php

###### Snippet 1.81: Iterator declaration

This iterator takes in an array and returns an object with the single function property next. Internally, the iterator keeps track of the array and the index we are currently looking at. To use the iterator, we simply call the next function. Calling next will cause the iterator to return an object and increment the internal index by one. The object returned by the iterator must have, at a minimum, the properties value and done. Value will contain the value at the index we are currently viewing. `Done` will contain a Boolean. If the Boolean equals true, then we have finished the **traversion on** the input collection. If it is **falsy**, then we can keep calling the next function:

```

// Using an iterator

let it = createIterator( [ 'Hello', 'World' ] );

console.log( it.next() );

// Expected output: { value: 'Hello', done: false }

console.log( it.next() );

// Expected output: { value: 'World' , done: false }

console.log( it.next() );

// Expected output: { value: undefined, done: true }

```php

###### Snippet 1.82: Iterator use

#### Note

When an iterator's finality property is truthy, it should not return any new data. To demonstrate the use of `iterator.next()`, you can provide the example shown in the preceding snippet.

In summary, iterators provide us with a way to traverse potentially complex collections of data. An iterator tracks its current state and each time the next function is called, it provides an object with a value and a finality Boolean. When the iterator reaches the end of the collection, calls to `iterator.next()` will return a truthy finality parameter and no new values will be received.

### Generators

**A** **generator** provides an iterative way to build a collection of data. A generator can return values one at a time while pausing execution until the next value is requested. A generator keeps track of the internal state and each time it is requested, it returns a new number in the sequence.

To create a `generator`, we must define a function with an asterisk in front of the function name and the `yield` keyword in the body. For example, to create a generator called `testGenerator`, we would initialize it as follows:

```

function *testGen( data ) { yield 0; }.

```php

The asterisk designates that this is a `generator function`. The `yield` keyword designates a break in the normal function flow until the generator function is called again. An example of a generator is shown in the following snippet:

```

function *gen() {

let i = 0;

while (true){

yield i++;

}

}

```php

###### Snippet 1.83: Generator creation

This `generator` function that we created in the preceding snippet, called `gen`, has an internal state variable called `i`. When the `generator` is created, it is automatically initialized with an internal next function. When the `next` function is called for the first time, the execution starts, the loop begins, and when the `yield` keyword is reached, the execution of the function is stopped until the next function is called again. When the `next` function is called, the program returns an object containing a value and `done`.

### Exercise 15: Creating a Generator

To create a generator function that generates the values of the sequence of 2n to show how generators can build a set of sequential data, perform the following steps:

1.  Create a `generator` called `gen`.

    Place an asterisk before the identifier name.

2.  Inside the generator body, do the following:

    Create a variable called `i` and set the initial value to 1\. Then, create an infinite while loop.

    In the body of the while loop, `yield` `i` and set `i` to `i * 2.`

3.  Initialize `gen` and save it into a variable called `generator`
4.  Call your `generator` several times and log the output to see the values change.

**Code**

##### index.js:

```

function *gen() {

let i = 1;

while (true){

yield i;

i = i * 2;

}

}

const generator = gen();

console.log( generator.next(), generator.next(), generator.next() );

```php

###### Snippet 1.84: Simple generator

[`bit.ly/2VK7M3d`](https://bit.ly/2VK7M3d)

**Outcome**

![Figure 1.17: Calling the generator output](img/Figure_1.17.jpg)

###### Figure 1.17: Calling the generator output

You have successfully created a generator function.

Similar to iterators, the `done` value contains the completion status of the generator. If the `done` value is set to `true`, then the generator has finished execution and will no longer return new values. The value parameter contains the result of the expression contained on the line with the `yield` keyword. In this case, it will return the current value of `i`, before the increment. This is shown in the following code:

```

let sequence = gen();

console.log(sequence.next());

//Expected output: { value: 0, done: false }

console.log(sequence.next());

//Expected output: { value: 1, done: false }

console.log(sequence.next());

//Expected output: { value: 2, done: false }

```php

###### Snippet 1.85: Generator use

Generators pause execution when they reach the `yield` keyword. This means that loops will pause execution. Another powerful tool of a generator is the ability to pass in data via the next function and `yield` keyword. When a value is passed into the next function, the return value of the `yield` expression will be set to the value that's passed into next. An example of this is shown in the following code:

```

function *gen() {

let i = 0;

while (true){

let inData = yield i++;

console.log( inData );

}

}

let sequence = gen();

sequence.next()

sequence.next( 'test1' )

sequence.next()

sequence.next( 'test2' )

// Expected output:

// 'test1'

// undefined

// 'test2'

```

###### Snippet 1.86 Yield keyword

总之，生成器是构建数据集的迭代方式。它们一次返回一个值，同时跟踪内部状态。当达到`yield`关键字时，内部执行停止并返回一个值。当调用`next`函数时，执行恢复，直到达到`yield`。数据可以通过`next`函数传递给生成器。通过`yield`表达式返回传入的数据。当生成器发出一个值对象，并将`done`参数设置为 true 时，对`generator.next()`的调用不应产生任何新的值。

在最后一个主题 I 中，我们介绍了迭代器和生成器。迭代器遍历数据集合中的数据，并在每一步返回请求的值。一旦它们到达集合的末尾，`done`标志将设置为 true，并且不会再迭代新的项目。生成器是一种生成数据集合的方法。在每一步中，生成器根据其内部状态产生一个新值。迭代器和生成器都在它们的生命周期中跟踪它们的内部状态。

### 活动 1：实现生成器

您被要求构建一个简单的应用程序，根据请求生成斐波那契数列中的数字。该应用程序为每个请求生成序列中的下一个数字，并在给定输入时重置序列。使用生成器生成斐波那契数列。如果将一个值传递给生成器，则重置序列。

使用生成器构建复杂的迭代数据集，执行以下步骤：

1.  查找斐波那契数列。

1.  创建一个生成器，提供斐波那契数列中的值。

1.  如果生成器的`yield`语句返回一个值，则重置序列。

**结果**

![图 1.18：实现生成器输出](img/Figure_1.18.jpg)

###### 图 1.18：实现生成器输出

您已成功创建了一个可以用来基于斐波那契数列构建迭代数据集的生成器。

#### 注意

此活动的解决方案可在第 280 页找到。

## 总结

在本章中，我们看到 ECMAScript 是现代 JavaScript 的脚本语言规范。ECMAScript 6，或 ES6，于 2015 年发布。通过本章，我们涵盖了 ES6 的一些关键点及其与以前版本 JavaScript 的区别。我们强调了变量作用域的规则，声明变量的关键字，箭头函数语法，模板文字，增强的对象属性表示法，解构赋值，类和模块，转译和迭代器和生成器。您已经准备好将这些知识应用于您的专业 JavaScript 项目。

在下一章中，我们将学习什么是异步编程语言，以及如何编写和理解异步代码。
