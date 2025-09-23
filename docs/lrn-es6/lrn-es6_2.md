# 第二章：熟悉你的库

**ES6** 向内置的 JavaScript 对象添加了许多新的属性和方法，以便程序员可以轻松地完成繁琐的任务。这些新功能旨在帮助开发者摆脱使用黑客和易出错的技巧来完成与数字、字符串和数组相关的各种操作。在本章中，我们将查看 ES6 添加到原生对象的所有新功能。

在本章中，我们将介绍：

+   `Number` 对象的新属性和方法

+   将数字常量表示为二进制或八进制

+   `Math` 对象的新属性和方法

+   创建多行字符串和 `String` 对象的新方法

+   `Array` 对象的新属性和方法

+   什么是 Map 和 Set？

+   使用数组缓冲区和类型化数组

+   `Object` 对象的新属性和方法

# 与数字一起工作

ES6 增加了创建数字的新方法以及 `Number` 对象的新属性，以便更容易地处理数字。ES6 通过增强 `Number` 对象来简化创建数学丰富应用的过程，并防止导致错误的常见误解。ES6 还提供了在 ES5 中已经可能做到的事情的新方法，例如将数字常量表示为八进制。

### 注意

JavaScript 将数字表示为十进制。默认情况下，数字常量被解释为十进制。

## 二进制表示法

在 ES5 中，没有原生的方法来表示二进制的数字常量。但在 ES6 中，你可以使用 `0b` 符号来前缀数字常量，使 JavaScript 将它们解释为二进制。

这里有一个例子：

```js
let a = 0b00001111;
let b = 15;

console.log(a === b);
console.log(a);
```

输出如下：

```js
true
15
```

这里，`0b00001111` 是 15 的二进制表示，基于十进制的十进制。

## 八进制表示法

在 ES5 中，要表示八进制的数字常量，我们需要使用 `0` 前缀。例如，看看下面的例子：

```js
var a = 017;
var b = 15;

console.log(a === b);
console.log(a);
```

输出如下：

```js
true
15
```

但是，对于初学者来说，八进制表示法可能会造成混淆，因为他们认为以 `0` 开头的十进制数与 `17` 相同。例如，他们认为 `017` 与 `17` 相同。因此，为了消除这种混淆，ES6 允许我们使用 `0o` 前缀来使 JavaScript 将这些数字常量解释为八进制。

这里有一个例子来演示这一点：

```js
let a = 0o17;
let b = 15;

console.log(a === b);
console.log(a);
```

输出如下：

```js
true
15
```

## `Number.isInteger(number)` 方法

JavaScript 中的数字以 64 位浮点数的形式存储。因此，JavaScript 中的整数是没有小数部分的浮点数，或者小数部分全部为 0 的浮点数。

在 ES5 中，没有内置的方法来检查一个数字是否为整数。ES6 向 `Number` 对象添加了一个新方法，称为 `isInteger()`，它接受一个数字并返回 `true` 或 `false`，这取决于该数字是否为整数。

这里是一个示例代码：

```js
let a = 17.0;
let b = 1.2;

console.log(Number.isInteger(a));
console.log(Number.isInteger(b));
```

输出如下：

```js
true
false
```

## `Number.isNaN(value)` 方法

在 ES5 中，没有方法可以检查一个变量是否包含 `NaN` 值。

### 注意

全局 `isNaN()` 函数用于检查一个值是否是数字。如果值不是数字，则返回 `true`，否则返回 `false`。

因此，ES6 引入了一个新的 `Number` 对象方法，称为 `isNaN()`，用于检查一个值是否是 `NaN`。以下是一个示例，它演示了 `Number.isNaN()` 并解释了它与全局 `isNaN()` 函数的不同之处：

```js
let a = "NaN";
let b = NaN;
let c = "hello";
let d = 12;

console.log(Number.isNaN(a));
console.log(Number.isNaN(b));
console.log(Number.isNaN(c));
console.log(Number.isNaN(d));

console.log(isNaN(a));
console.log(isNaN(b));
console.log(isNaN(c));
console.log(isNaN(d));
```

输出如下：

```js
false
true
false
false
true
true
true
false
```

这里你可以看到，`Number.isNaN()` 方法仅在传入的值正好是 `NaN` 时返回 `true`。

### 提示

你可能会问，为什么不用 `==` 或 `===` 运算符来代替 `Number.isNaN(value)` 方法？`NaN` 值是唯一一个不等于自身的值，也就是说，表达式 `NaN==NaN` 或 `NaN===NaN` 将返回 `false`。

## Number.isFinite(number) 方法

在 ES5 中，没有内置的方法来检查一个值是否是有限数。

### 注意

全局 `isFinite()` 函数接受一个值并检查它是否是一个有限数。但不幸的是，它也会对转换为 `Number` 类型的值返回 `true`。

因此，ES6 引入了 `Number.isFinite()` 方法，解决了 `window.isFinite()` 函数的问题。以下是一个演示这个特性的例子：

```js
console.log(isFinite(10));
console.log(isFinite(NaN));
console.log(isFinite(null));
console.log(isFinite([]));

console.log(Number.isFinite(10));
console.log(Number.isFinite(NaN));
console.log(Number.isFinite(null));
console.log(Number.isFinite([]));
```

输出如下：

```js
true
false
true
true
true
false
false
false
```

## Number.isSafeInteger(number) 方法

JavaScript 数字以 64 位浮点数的形式存储，遵循国际 IEEE 754 标准。这种格式使用 64 位存储数字，其中数字（分数）存储在 0 到 51 位，指数在 52 到 62 位，符号在最后一位。

因此，在 JavaScript 中，安全的整数是那些不需要四舍五入到其他整数以适应 IEEE 754 表示的数。从数学上讲，从 -(2⁵³-1) 到 (2⁵³-1) 的数被认为是安全的整数。

下面是一个演示这个特性的例子：

```js
console.log(Number.isSafeInteger(156));
console.log(Number.isSafeInteger('1212'));
console.log(Number.isSafeInteger(Number.MAX_SAFE_INTEGER));
console.log(Number.isSafeInteger(Number.MAX_SAFE_INTEGER + 1));
console.log(Number.isSafeInteger(Number.MIN_SAFE_INTEGER));
console.log(Number.isSafeInteger(Number.MIN_SAFE_INTEGER - 1));
```

输出如下：

```js
true
false
true
false
true
false
```

这里，`Number.MAX_SAFE_INTEGER` 和 `Number.MIN_SAFE_INTEGER` 是 ES6 中引入的常量值，分别代表 (2⁵³-1) 和 -(2⁵³-1)。

## Number.EPSILON 属性

JavaScript 使用的是计算机无法准确表示的二进制浮点数表示法，例如 0.1、0.2、0.3 等数字。当你的代码执行时，像 0.1 这样的数字会被四舍五入到该格式中最接近的数字，这会导致小的舍入误差。

考虑以下示例：

```js
console.log(0.1 + 0.2 == 0.3);
console.log(0.9 - 0.8 == 0.1);
console.log(0.1 + 0.2);
console.log(0.9 - 0.8);
```

输出如下：

```js
false
false
0.30000000000000004
0.09999999999999998
```

`Number.EPSILON` 属性是在 ES6 中引入的，其值大约为 2^(-52)。这个值表示在比较浮点数时一个合理的误差范围。使用这个数字，我们可以创建一个自定义函数来比较浮点数，同时忽略最小的舍入误差。

下面是一个示例代码：

```js
functionepsilonEqual(a, b)
{
  return Math.abs(a - b) <Number.EPSILON;
}

console.log(epsilonEqual(0.1 + 0.2, 0.3));
console.log(epsilonEqual(0.9 - 0.8, 0.1));
```

输出如下：

```js
true
true
```

在这里，`epsilonEqual()` 是我们构建的用于比较两个值是否相等的自定义函数。现在，输出结果符合预期。

### 注意

要了解更多关于 JavaScript 的这种行为和浮点算术的信息，请访问 [`floating-point-gui.de/`](http://floating-point-gui.de/)。

# 进行数学运算

ES6 向`Math`对象添加了许多新方法，涉及三角学、算术和杂项。这使得开发者可以使用原生方法而不是外部数学库。原生方法针对性能进行了优化，并且具有更好的十进制精度。

## 与三角学相关的操作

下面是一个示例代码，展示了添加到`Math`对象中的所有与三角学相关的函数：

```js
console.log(Math.sinh(0));    //hyberbolic sine of a value
console.log(Math.cosh(0));    //hyberbolic cosine of a value
console.log(Math.tanh(0));    //hyberbolic tangent of a value
console.log(Math.asinh(0));  //inverse hyperbolic sine of a value
console.log(Math.acosh(1));  //inverse hyperbolic cosine of a value
console.log(Math.atanh(0));  //inverse hyperbolic tangent of a value
console.log(Math.hypot(2, 2, 1));//Pythagoras theorem

```

输出如下：

```js
0
1
0
0
0
0
3
```

## 与算术相关的操作

下面是一个示例代码，展示了添加到`Math`对象中的所有与算术相关的函数：

```js
console.log(Math.log2(16));    //log base 2
console.log(Math.log10(1000)); //log base 10
console.log(Math.log1p(0));    //same as log(1 + value)
console.log(Math.expm1(0));    //inverse of Math.log1p()
console.log(Math.cbrt(8));     //cube root of a value
```

输出如下：

```js
4
3
0
0
2
```

## 杂项方法

ES6 向`Math`对象添加了一些杂项方法。这些方法用于转换和从数字中提取信息。

### `Math.imul(number1, number2)` 函数

`Math.imul()` 函数将两个数字作为 32 位整数相乘，并返回结果的下 32 位。这是 JavaScript 中进行 32 位整数乘法的唯一原生方法。

下面有一个示例来演示这一点：

```js
console.log(Math.imul(590, 5000000)); //32-bit integer multiplication
console.log(590 * 5000000); //64-bit floating-point multiplication
```

输出如下：

```js
-1344967296
2950000000
```

在这里，当进行乘法运算时，产生了一个如此大的数字，以至于它无法存储在 32 位中，因此丢失了低位。

### `Math.clz32(number)` 函数

`Math.clz32()` 函数返回一个数字在 32 位表示中的前导零位数。

下面是一个演示此功能的示例：

```js
console.log(Math.clz32(7));
console.log(Math.clz32(1000));
console.log(Math.clz32(295000000));
```

输出如下：

```js
29
22
3
```

### `Math.sign(number)` 函数

`Math.sign()` 函数返回一个数字的符号，表示该数字是负数、正数还是零。

这里有一个示例来演示这一点：

```js
console.log(Math.sign(11));
console.log(Math.sign(-11));
console.log(Math.sign(0));
```

输出如下：

```js
1
-1
0
```

从前面的代码中，我们可以看到，如果数字是正数，`Math.sign()` 函数返回 `1`；如果是负数，返回 `-1`；如果是零，返回 `0`。

### `Math.trunc(number)` 函数

`Math.trunc()` 函数通过移除任何小数位来返回一个数字的整数部分。以下是一个演示此功能的示例：

```js
console.log(Math.trunc(11.17));
console.log(Math.trunc(-1.112));
```

输出如下：

```js
11
-1
```

### `Math.fround(number)` 函数

`Math.fround()` 函数将一个数字四舍五入到 32 位浮点值。以下是一个演示此功能的示例：

```js
console.log(Math.fround(0));
console.log(Math.fround(1));
console.log(Math.fround(1.137));
console.log(Math.fround(1.5));
```

输出如下：

```js
0
1
1.1369999647140503
1.5
```

# 字符串操作

ES6 提供了创建字符串的新方法，并为全局 String 对象及其实例添加了新属性，以使字符串操作更容易。与 Python 和**Ruby**等编程语言相比，JavaScript 中的字符串缺乏功能和能力，因此 ES6 增强了字符串以改变这一点。

在我们深入了解新的字符串功能之前，让我们回顾一下 JavaScript 的内部字符编码和转义序列。在 Unicode 字符集中，每个字符都由一个称为**代码点**的十进制基数表示。**代码单元**是在内存中存储代码点的固定位数。编码方案决定了代码单元的长度。如果使用**UTF-8**编码方案，则代码单元是 8 位，如果使用**UTF-16**编码方案，则代码单元是 16 位。如果一个代码点不适合代码单元，它将被分割成多个代码单元，即多个字符按顺序表示一个字符。

JavaScript 解释器默认将 JavaScript 源代码解释为 UTF-16 代码单元的序列。如果源代码是用 UTF-8 编码方案编写的，那么有各种方法可以告诉 JavaScript 解释器将其解释为 UTF-8 代码单元的序列。JavaScript 字符串始终是 UTF-16 代码点的序列。

任何小于 65536 的代码点的 Unicode 字符都可以使用其代码点的十六进制值来转义，并在其前面加上`\u`。转义序列由六个字符组成。它们需要紧跟`\u`后的正好四个字符。如果十六进制字符代码只有一位、两位或三位，则需要用前导零填充。以下是一个演示此点的例子：

```js
var \u0061 = "\u0061\u0062\u0063";
console.log(a); //Output is "abc"
```

## 转义更大的代码点

在 ES5 中，对于需要超过 16 位存储的字符，我们需要两个 Unicode 转义序列。例如，要将`\u1F691`添加到字符串中，我们必须这样转义：

```js
console.log("\uD83D\uDE91");
```

这里`\uD83D`和`\uDE91`被称为**代理对**。代理对是两个 Unicode 字符，当它们按顺序书写时，代表另一个字符。

在 ES6 中，我们可以不使用代理对来编写它：

```js
console.log("\u{1F691}");
```

字符串将`\u1F691`存储为`\uD83D\uDE91`，因此上述字符串的长度仍然是`2`。

## `codePointAt(index)`方法

字符串的`codePointAt()`方法返回一个非负整数，表示给定索引处的字符的代码点。

以下是一个演示此点的例子：

```js
console.log("\uD83D\uDE91".codePointAt(1));
console.log("\u{1F691}".codePointAt(1));
console.log("hello".codePointAt(2));
```

输出如下：

```js
56977
56977
1080
```

## `String.fromCodePoint(number1, …, number 2)`方法

`String`对象的`fromCodePoint()`方法接受一系列代码点，并返回一个字符串。以下是一个演示此点的例子：

```js
console.log(String.fromCodePoint(0x61, 0x62, 0x63));
console.log("\u0061\u0062 " == String.fromCodePoint(0x61, 0x62));
```

输出如下：

```js
abc
true
```

## `repeat(count)`方法

字符串的`repeat()`方法构建并返回一个新的字符串，该字符串包含调用它的指定次数的副本，并将它们连接在一起。以下是一个演示此点的例子：

```js
console.log("a".repeat(6));      //Output "aaaaaa"
```

## `includes(string, index)`方法

`includes()`方法用于确定一个字符串是否可以在另一个字符串中找到，根据情况返回`true`或`false`。以下是一个演示此点的例子：

```js
var str = "Hi, I am a JS Developer";
console.log(str.includes("JS")); //Output "true"
```

它有一个可选的第二个参数，表示在字符串中开始搜索的位置。以下是一个演示此点的例子：

```js
var str = "Hi, I am a JS Developer";
console.log(str.includes("JS", 13)); // Output "false"
```

## `startsWith(string, index)`方法

`startsWith()` 方法用于确定一个字符串是否以另一个字符串的字符开头，根据情况返回 `true` 或 `false`。以下是一个演示此功能的示例：

```js
var str = "Hi, I am a JS Developer";
console.log(str.startsWith('Hi, I am')); //Output "true"
```

它接受一个可选的第二个参数，表示在字符串中开始搜索的位置。以下是一个演示此功能的示例：

```js
var str = "Hi, I am a JS Developer";
console.log(str.startsWith('JS Developer', 11)); //Output "true"
```

## `endsWith(string, index)` 函数

`endsWith()` 方法用于确定一个字符串是否以另一个字符串的字符结尾，根据情况返回 true 或 false。它还接受一个可选的第二个参数，表示假设为字符串结尾的字符串中的位置。以下是一个演示此功能的示例：

```js
var str = "Hi, I am a JS Developer";
console.log(str.endsWith("JS Developer"));  //Output "true"
console.log(str.endsWith("JS", 13));        //Output "true"
```

## 规范化

**规范化**简单地说，是一个在改变字符串意义的情况下搜索和标准化码点的过程。

还有不同的规范化形式：NFC、NFD、NFKC 和 NFKD。

让我们通过一个示例用例来理解 Unicode 字符串规范化。

### 案例研究

有许多 Unicode 字符可以用 16 位存储，也可以使用代理对表示。例如，"`é`" 字符可以用两种方式转义：

```js
console.log("\u00E9");  //output 'é'
console.log("e\u0301"); //output 'é'
```

问题在于当应用 `==` 运算符、迭代或查找长度时，你会得到一个意外的结果。以下是一个演示此功能的示例：

```js
var a = "\u00E9";
var b = "e\u0301";

console.log(a == b);
console.log(a.length);
console.log(b.length);

for(let i = 0; i<a.length; i++)
{
  console.log(a[i]);
}

for(let i = 0; i<b.length; i++)
{
  console.log(b[i]);
}
```

输出如下：

```js
false
1
2
é
é
```

在这里，这两个字符串显示方式相同，但当我们对它们进行各种字符串操作时，我们会得到不同的结果。

`length` 属性忽略代理对，并假设每个 16 位是一个单独的字符。`==` 运算符匹配二进制位，因此它也忽略了代理对。`[]` 运算符也假设每个 16 位是一个索引，因此忽略了代理对。

在这种情况下，为了解决问题，我们需要将代理对转换为 16 位字符表示。这个过程被称为**规范化**。为此，ES6 提供了一个 `normalize()` 函数。以下是一个演示此功能的示例：

```js
var a = "\u00E9".normalize();
var b = "e\u0301".normalize();

console.log(a == b);
console.log(a.length);
console.log(b.length);

for(let i = 0; i<a.length; i++)
{
  console.log(a[i]);
}

for(let i = 0; i<b.length; i++)
{
  console.log(b[i]);
}
```

输出如下：

```js
true
1
1
é
é
```

这里输出符合预期。`normalize()` 返回字符串的规范化版本。`normalize()` 默认使用 NFC 格式。

规范化不仅用于代理对的情况；还有许多其他情况。

### 注意

字符串的规范化版本不是用于向用户显示的；它用于字符串的比较和搜索。

要了解更多关于 Unicode 字符串规范化和规范化形式的信息，请访问 [`www.unicode.org/reports/tr15/`](http://www.unicode.org/reports/tr15/).

## 模板字符串

模板字符串只是创建字符串的新字面量，这使得许多事情变得更容易。它们提供嵌入表达式、多行字符串、字符串插值、字符串格式化、字符串标记等功能。它们总是在运行时被处理和转换为正常的 JavaScript 字符串，因此可以在使用正常字符串的任何地方使用。

模板字符串使用反引号而不是单引号或双引号来编写。以下是一个简单模板字符串的示例：

```js
let str1 = `hello!!!`; //template string
let str2 = "hello!!!";

console.log(str1 === str2); //output "true"
```

### 表达式

在 ES5 中，要在普通字符串中嵌入表达式，你会这样做：

```js
Var a = 20;
Var b = 10;
Var c = "JavaScript";
Var str = "My age is " + (a + b) + " and I love " + c;

console.log(str);
```

输出是：

```js
My age is 30 and I love JavaScript
```

在 ES6 中，模板字符串使得在字符串中嵌入表达式变得更加容易。模板字符串可以包含表达式。这些表达式放置在由美元符号和大括号指示的占位符中，即 `${expressions}`。占位符中的表达式解析值和它们之间的文本被传递给一个函数以解析模板字符串为普通字符串。默认函数只是将部分连接成一个字符串。如果我们使用自定义函数处理字符串部分，则模板字符串被称为 **标签模板字符串**，自定义函数被称为 **标签函数**。

以下是一个示例，展示了如何在模板字符串中嵌入表达式：

```js
let a = 20;
let b = 10;
let c = "JavaScript";
letstr = `My age is ${a+b} and I love ${c}`;

console.log(str);
```

输出是：

```js
My age is 30 and I love JavaScript
```

让我们创建一个标签模板字符串，即使用标签函数处理字符串。让我们实现标签函数以执行与默认函数相同的功能。以下是一个演示此功能的示例：

```js
let tag = function(strings, ...values)
{
  let result = "";

  for(let i = 0; i<strings.length; i++)
  {
    result += strings[i];

    if(i<values.length)
    {
      result += values[i];
    }
  }

  return result;
};

return result;
};

let a = 20;
let b = 10;
let c = "JavaScript";
let str = tag `My age is ${a+b} and I love ${c}`;

console.log(str);
```

输出是：

```js
My age is 30 and I love JavaScript
```

在这里，我们的标签函数的名称是 `tag`，但你可以将其命名为任何其他名称。自定义函数接受两个参数，即第一个参数是模板字符串的字符串字面量数组，第二个参数是表达式的解析值数组。第二个参数作为多个参数传递，因此我们使用剩余参数。

### 多行字符串

模板字符串提供了一种创建包含多行文本的新字符串的方法。

在 ES5 中，我们需要使用 `\n` 换行符来添加换行符。以下是一个演示此功能的示例：

```js
console.log("1\n2\n3");
```

输出是：

```js
1
2
3
```

在 ES6 中，使用 **多行** 字符串我们可以简单地写：

```js
console.log(`1
2
3`);
```

输出是：

```js
1
2
3
```

在上面的代码中，我们简单地包含了需要放置 `\n` 的换行符。在将模板字符串转换为普通字符串时，换行符被转换为 `\n`。

### 原始字符串

原始字符串是一个普通的字符串，其中转义字符不会被解释。

我们可以使用模板字符串创建原始字符串。我们可以使用 `String.raw` 标签函数获取模板字符串的原始版本。以下是一个演示此功能的示例：

```js
let s = String.raw `xy\n${ 1 + 1 }z`;
console.log(s);
```

输出是：

```js
xy\n2z
```

这里 `\n` 不会被解释为换行符，而是其两个字符，即 `\` 和 `n`。变量 `s` 的长度将是 `6`。

如果你创建了一个标签函数并想返回原始字符串，则使用第一个参数的 `raw` 属性。`raw` 属性是一个数组，它包含第一个参数的字符串的原始版本。以下是一个演示此功能的示例：

```js
let tag = function(strings, ...values)
{
    return strings.raw[0]
};

letstr = tag `Hello \n World!!!`;

console.log(str);
```

输出是：

```js
Hello \n World!!!
```

# 数组

ES6 向全局 Array 对象及其实例添加了新属性，以使处理数组更容易。与 Python 和 Ruby 等编程语言相比，JavaScript 中的数组缺乏功能和能力，因此 ES6 增强了数组以改变这一点。

## Array.from(iterable, mapFunc, this) 方法

`Array.from()` 方法从一个可迭代对象创建一个新的数组实例。第一个参数是可迭代对象的引用。第二个参数是可选的，是一个回调函数（称为 **映射函数**），它会对可迭代对象的每个元素进行调用。第三个参数也是可选的，是映射函数内部 `this` 的值。

这里有一个示例来演示这一点：

```js
letstr = "0123";
letobj = {number: 1};
letarr = Array.from(str, function(value){
    return parseInt(value) + this.number;
}, obj);

console.log(arr);
```

输出是：

```js
1, 2, 3, 4
```

## `Array.of(values…)` 方法

`Array.of()` 方法是创建数组的 `Array` 构造函数的替代方案。当使用 `Array` 构造函数时，如果我们只传递一个参数，而且这个参数是一个数字，那么 `Array` 构造函数将创建一个空数组，其 `length` 属性等于传递的数字，而不是创建一个包含该数字的单元素数组。因此，引入了 `Array.of()` 方法来解决这个问题。

这里有一个示例来演示这一点：

```js
let arr1 = new Array(2);
let arr2 = new Array.of(2);

console.log(arr1[0], arr1.length);
console.log(arr2[0], arr2.length);
```

输出是：

```js
undefined 2
2 1
```

当你动态地构造一个新的数组实例时，应该使用 `Array.of()` 而不是 `Array` 构造函数，即当你不知道值的类型和元素数量时。

## `fill(value, startIndex, endIndex)` 方法

数组的 `fill()` 方法使用给定的值填充数组从 `startIndex` 到 `endIndex`（不包括 `endIndex`）的所有元素。记住，`startIndex` 和 `endIndex` 参数是可选的；因此，如果它们没有提供，则整个数组将用给定的值填充。如果只提供了 `startIndex`，则 `endIndex` 默认为数组的长度减 1。

如果 `startIndex` 是负数，则它被视为数组长度加上 `startIndex`。如果 `endIndex` 是负数，则它被视为数组长度加上 `endIndex`。

这里有一个示例来演示这一点：

```js
let arr1 = [1, 2, 3, 4];
let arr2 = [1, 2, 3, 4];
let arr3 = [1, 2, 3, 4];
let arr4 = [1, 2, 3, 4];
let arr5 = [1, 2, 3, 4];

arr1.fill(5);
arr2.fill(5, 1, 2);
arr3.fill(5, 1, 3);
arr4.fill(5, -3, 2);
arr5.fill(5, 0, -2);

console.log(arr1);
console.log(arr2);
console.log(arr3);
console.log(arr4);
console.log(arr5);
```

输出是：

```js
5,5,5,5
1,5,3,4
1,5,5,4
1,5,3,4
5,5,3,4
```

## `find(testingFunc, this)` 方法

数组的 `find()` 方法返回一个数组元素，如果它满足提供的测试函数。否则它返回 `undefined`。

`find()` 方法接受两个参数，即第一个参数是测试函数，第二个参数是测试函数中 `this` 的值。第二个参数是可选的。

测试函数有三个参数：第一个参数是正在处理的数组元素，第二个参数是正在处理的当前元素的索引，第三个参数是调用 `find()` 的数组。

测试函数需要返回 `true` 来满足一个值。`find()` 方法返回第一个满足提供的测试函数的元素。

这里有一个示例来演示 `find()` 方法：

```js
var x = 12;
var arr = [11, 12, 13];
var result = arr.find(function(value, index, array){
    if(value == this)
    {
        return true;
    }
}, x);

console.log(result); //Output "12"
```

## `findIndex(testingFunc, this)` 方法

`findIndex()` 方法与 `find()` 方法类似。`findIndex()` 方法返回满足条件的数组元素的索引，而不是元素本身。

```js
let x = 12;
let arr = [11, 12, 13];
let result = arr.findIndex(function(value, index, array){
    if(value == this)
    {
        return true;
    }
}, x);

console.log(result); Output "1"
```

## `copyWithin(targetIndex, startIndex, endIndex)` 函数

数组的 `copyWithin()` 方法用于将数组的值序列复制到数组中的不同位置。

`copyWithin()`方法接受三个参数：第一个参数表示目标索引，即要复制元素的位置，第二个参数表示开始复制的索引位置，第三个参数表示索引，即实际结束复制元素的位置。

第三个参数是可选的，如果没有提供，则默认为`length-1`，其中`length`是数组的长度。如果`startIndex`是负数，则计算为`length+startIndex`。同样，如果`endIndex`是负数，则计算为`length+endIndex`。

这里有一个示例来演示这一点：

```js
let arr1 = [1, 2, 3, 4, 5];
let arr2 = [1, 2, 3, 4, 5];
let arr3 = [1, 2, 3, 4, 5];
let arr4 = [1, 2, 3, 4, 5];

arr1.copyWithin(1, 2, 4);
arr2.copyWithin(0, 1);
arr3.copyWithin(1, -2);
arr4.copyWithin(1, -2, -1);

console.log(arr1);
console.log(arr2);
console.log(arr3);
console.log(arr4);
```

输出是：

```js
1,3,4,4,5
2,3,4,5,5
1,4,5,4,5
1,4,3,4,5
```

## `entries()`、`keys()`和`values()`方法

数组的`entries()`方法返回一个包含数组每个索引的键/值对的可迭代对象。同样，数组的`keys()`方法返回一个包含数组中每个索引的键的可迭代对象。同样，数组的`values()`方法返回一个包含数组值的可迭代对象。

`entries()`方法返回的可迭代对象以数组的形式存储键/值对。

这些函数返回的可迭代对象不是一个数组。

这里有一个示例来演示这一点：

```js
let arr = ['a', 'b', 'c'];
let entries = arr.entries();
let keys = arr.keys();
let values = arr.values();

console.log(...entries);
console.log(...keys);
console.log(...values);
```

输出是：

```js
0,a 1,b 2,c
0 1 2
a b c
```

# 集合

集合是指任何将多个元素存储为单一单元的对象。ES6 引入了各种新的集合对象，以提供更好的存储和组织数据的方式。

在 ES5 中，数组是唯一的集合对象。ES6 引入了数组缓冲区、类型化数组、集合（Sets）和映射（Maps），这些都是内置的集合对象。

让我们看看 ES6 提供的不同集合对象。

## 数组缓冲区

数组的元素可以是任何类型，如字符串、数字、对象等。数组可以动态增长。数组的问题在于它们在执行时间上较慢，并且占用更多内存。这导致在开发需要大量计算和大量处理数字的应用程序时出现问题。因此，引入数组缓冲区来解决这个问题。

数组缓冲区是内存中 8 位块的集合。每个块都是一个数组缓冲区元素。数组缓冲区的大小在创建时需要确定，因此它不能动态增长。数组缓冲区只能存储数字。所有块在创建数组缓冲区时都初始化为数字 0。

使用`ArrayBuffer`构造函数创建数组缓冲区对象。

```js
let buffer = new ArrayBuffer(80); //80 bytes size
```

可以使用`DataView`对象从数组缓冲区对象中读取值或写入值。不强制使用 8 位来表示数字。我们可以使用 8 位、16 位、32 位和 64 位来表示数字。以下是一个示例，展示了如何创建`DataView`对象并读取/写入`ArrayBuffer`对象：

```js
let buffer = new ArrayBuffer(80);
let view = new DataView(buffer);

view.setInt32(8,22,false);

var number = view.getInt32(8,false);

console.log(number); //Output "22"
```

在这里，我们使用`DataView`构造函数创建了一个`DataView`对象。`DataView`对象提供了几种方法来将数字读入或写入数组缓冲区对象。这里我们使用了`setInt32()`方法，它使用 32 位来存储提供的数字。

所有用于将数据写入数组缓冲区对象的 DataView 对象的方法都接受三个参数。第一个参数表示偏移量，即我们想要写入数字的字节。第二个参数是要存储的数字。第三个参数是一个布尔类型，它表示数字的端序，例如`false`表示大端序。

类似地，所有用于读取数组缓冲区对象数据的 DataView 对象的方法都接受两个参数。第一个参数是偏移量，第二个参数表示使用的端序。

这里是 DataView 对象提供的其他用于存储数字的函数：

+   **setInt8**: 使用 8 位存储一个数字。它接受一个有符号整数（负或正）。

+   **setUint8**: 使用 8 位存储一个数字。它接受一个无符号整数（正）。

+   **setInt16**: 使用 16 位存储一个数字。它接受一个有符号整数。

+   **setUint16**: 使用 16 位存储一个数字。它接受一个无符号整数。

+   **setInt32**: 使用 32 位存储一个数字。它接受一个有符号整数。

+   **setUint32**: 使用 32 位存储一个数字。它接受一个无符号整数。

+   **setFloat32**: 使用 32 位存储一个数字。它接受一个有符号十进制数。

+   **setFloat64**: 使用 64 位存储一个数字。它接受一个有符号十进制数。

这里是 DataView 对象提供的其他用于检索存储数字的函数：

+   **getInt8**: 读取 8 位。返回有符号整数。

+   **getUint8**: 读取 8 位。返回无符号整数。

+   **getInt16**: 读取 16 位。返回有符号整数。

+   **getUint16**: 读取 16 位。返回无符号整数。

+   **getInt32**: 读取 32 位。返回有符号整数。

+   **getUint32**: 读取 32 位。返回无符号整数。

+   **getFloat32**: 读取 32 位。返回有符号十进制数。

+   **getFloat64**: 读取 64 位。返回有符号十进制数。

## 类型化数组

我们看到了如何在数组缓冲区中读取和写入数字。但是方法非常繁琐，因为每次都必须调用一个函数。**类型化数组**允许我们像处理普通数组一样读取和写入数组缓冲区对象。

类型化数组像一个数组缓冲区对象的包装器，并将数组缓冲区对象的数据视为*n*-位数字的序列。`n`值取决于我们如何创建类型化数组。

下面是一个代码示例，演示了如何创建一个数组缓冲区对象，并使用类型化数组对其进行读写：

```js
var buffer = new ArrayBuffer(80);
vartyped_array = new Float64Array(buffer);
typed_array[4] = 11;

console.log(typed_array.length);
console.log(typed_array[4]);
```

输出是：

```js
10
11
```

在这里，我们使用`Float64Array`构造函数创建了类型化数组，因此它将数组缓冲区中的数据视为 64 位有符号十进制数的序列。这里数组缓冲区对象的大小是 640 位，因此只能存储 10 个 64 位数字。

类似地，还有其他类型化数组构造函数，用于将数组缓冲区中的数据表示为不同位数的序列。以下是列表：

+   **Int8Array**: 将其视为 8 位有符号整数

+   **Uint8Array**: 将其视为 8 位无符号整数

+   **Int16Array**: 将其视为 16 位有符号整数

+   **Uint16Array**：被视为 16 位无符号整数

+   **Int32Array**：被视为 32 位有符号整数

+   **Uint32Array**：被视为 32 位无符号整数

+   **Float32Array**：被视为 32 位有符号十进制数

+   **Float64Array**：被视为 64 位有符号十进制数

类型化数组提供了正常 JavaScript 数组提供的所有方法。它们也实现了可迭代协议，因此可以用作可迭代对象。

## Set

**Set** 是任何数据类型唯一值的集合。集合中的值按插入顺序排列。集合是通过 `Set` 构造函数创建的。以下是一个示例：

```js
let set1 = new Set();
let set2 = new Set("Hello!!!");
```

在这里 `set1` 是一个空集合。而 `set2` 是使用可迭代对象的值创建的，即字符串的字符，并且字符串不为空，因此 `set2` 不是空的。

这里是示例代码，它演示了可以在集合上执行的各种操作：

```js
let set = new Set("Hello!!!");

set.add(12); //add 12

console.log(set.has("!")); //check if value exists
console.log(set.size);

set.delete(12); //delete 12

console.log(...set);

set.clear(); //delete all values
```

输出是：

```js
true
6
H e l o !
```

我们向 `set` 对象中添加了九个项，但大小只有六，因为集合会自动删除重复的值。字符 `l` 和 `!` 被重复多次。

集合对象也实现了可迭代协议，因此它们可以用作可迭代对象。

当你想维护一个值的集合并检查一个值是否存在而不是检索一个值时，会使用集合。例如：如果你在代码中只使用数组的 `indexOf()` 方法来检查值是否存在，那么集合可以用作数组的替代。

## WeakSet

这里是 Set 和 WeakSet 对象之间的区别：

+   集合可以存储原始类型和对象引用，而 WeakSet 对象只能存储对象引用。

+   WeakSet 对象的一个重要特性是，如果没有其他引用指向存储在 WeakSet 对象中的对象，则它们会被垃圾回收。

+   最后，WeakSet 对象是不可枚举的，也就是说，你不能找到它的大小；它也没有实现可迭代协议。

除了这三个区别之外，它的行为与 Set 完全相同。除了这三个区别之外，Set 和 WeakSet 对象之间的一切都是相同的。

WeakSet 对象是通过 `WeakSet` 构造函数创建的。你不能将可迭代对象作为参数传递给 WeakSet 对象。

这里是一个演示 WeakSet 的示例：

```js
letweakset = new WeakSet();

(function(){
  let a = {};
  weakset.add(a);
})()

//here 'a' is garbage collected from weakset
console.log(weakset.size); //output "undefined"
console.log(...weakset); //Exception is thrown

weakset.clear(); //Exception, no such function
```

## Map

Map 是键/值对的集合。Map 的键和值可以是任何数据类型。键/值对按插入顺序排列。Map 对象是通过 `Map` 构造函数创建的。

这里是一个示例，演示了如何创建 Map 对象并在其上执行各种操作：

```js
let map = new Map();
let o = {n: 1};

map.set(o, "A"); //add
map.set("2", 9);

console.log(map.has("2")); //check if key exists
console.log(map.get(o)); //retrieve value associated with key
console.log(...map);

map.delete("2"); //delete key and associated value
map.clear(); //delete everything

//create a map from iterable object
let map_1 = new Map([[1, 2], [4, 5]]);

console.log(map_1.size); //number of keys
```

输出是：

```js
true
A
[object Object],A 2,9
2
```

在从可迭代对象创建 Map 对象时，我们需要确保可迭代对象返回的值是长度为 `2` 的数组，即索引 0 是键，索引 1 是值。

如果我们尝试添加一个已经存在的键，那么它将被覆盖。Map 对象也实现了可迭代协议，因此也可以用作可迭代对象。在通过可迭代协议迭代 Map 时，它们返回包含键/值对的数组，正如前一个示例所示。

## WeakMap

以下是 Map 和 WeakMap 对象之间的差异：

+   Map 对象的键可以是原始类型或对象引用，但 WeakMap 对象中的键只能是对象引用

+   `WeakMap`对象的一个重要特性是，如果没有其他引用指向由键引用的对象，则该键将被垃圾回收。

+   最后，WeakMap 对象不是可枚举的，也就是说，你不能找到它的大小，它也不实现可迭代协议。

除了这三个差异之外，Map 和 WeakMap 对象之间的一切都是相似的。

使用`WeakMap`构造函数创建`WeakMap`。以下是一个演示其使用的示例：

```js
let weakmap = new WeakMap();

(function(){
  let o = {n: 1};
  weakmap.set(o, "A");
})()

//here 'o' key is garbage collected
let s = {m: 1};

weakmap.set(s, "B");

console.log(weakmap.get(s));
console.log(...weakmap); //exception thrown

weakmap.delete(s);
weakmap.clear(); //Exception, no such function

let weakmap_1 = new WeakMap([[{}, 2], [{}, 5]]);   //this works

console.log(weakmap_1.size); //undefined
```

# Object

ES6 标准化了对象的`__proto__`属性，并向全局`Object`对象添加了新属性。

## `__proto__`属性

JavaScript 对象有一个内部`[[prototype]]`属性，它引用对象的原型，即它继承的对象。为了读取属性，我们必须使用`Object.getPrototypeOf()`，为了创建具有给定原型的新的对象，我们必须使用`Object.create()`方法。`[[prototype]]`属性不能直接读取或修改。

由于`[[prototype]]`属性的性质，继承变得繁琐，因此一些浏览器在对象中添加了一个特殊的`__proto__`属性，这是一个访问器属性，它公开了内部的`[[prototype]]`属性，使得与原型的工作更加容易。`__proto__`属性在 ES5 中并未标准化，但由于其流行，它在 ES6 中被标准化。

以下是一个演示此点的示例：

```js
//In ES5
var x = {x: 12};
var y = Object.create(x, {y: {value: 13}});

console.log(y.x); //Output "12"
console.log(y.y); //Output "13"

//In ES6
let a = {a: 12, __proto__: {b: 13}};
console.log(a.a); //Output "12"
console.log(a.b); //Output "13"
```

### `Object.is(value1, value2)`方法

`Object.is()`方法确定两个值是否相等。它与`===`运算符类似，但`Object.is()`方法有一些特殊情况。以下是一个演示这些特殊情况的示例：

```js
console.log(Object.is(0, -0));
console.log(0 === -0);
console.log(Object.is(NaN, 0/0));
console.log(NaN === 0/0);
console.log(Object.is(NaN, NaN));
console.log(NaN ===NaN);
```

输出是：

```js
false
true
true
false
true
false
```

### `Object.setPrototypeOf(object, prototype)`方法

`Object.setPrototypeOf()`方法只是分配对象`[[prototype]]`属性的另一种方式。以下是一个演示此点的示例：

```js
let x = {x: 12};
let y = {y: 13};

Object.setPrototypeOf(y, x)

console.log(y.x); //Output "12"
console.log(y.y); //Output "13"
```

### `Object.assign(targetObj, sourceObjs...)`方法

`Object.assign()`方法用于将一个或多个源对象的所有可枚举自有属性的值复制到目标对象。此方法将返回`targetObj`。

以下是一个演示此点的示例：

```js
let x = {x: 12};
let y = {y: 13, __proto__: x};
let z = {z: 14, get b() {return 2;}, q: {}};

Object.defineProperty(z, "z", {enumerable: false});

let m = {};

Object.assign(m, y, z);

console.log(m.y);
console.log(m.z);
console.log(m.b);
console.log(m.x);
console.log(m.q == z.q);
```

输出是：

```js
13
undefined
2
undefined
true
```

在使用`Object.assign()`方法时，以下是一些需要记住的重要事项：

+   它在来源上调用 getter，在目标上调用 setter。

+   它只是将源属性的值分配给目标的新或现有属性。

+   它不会复制来源的`[[prototype]]`属性。

+   JavaScript 属性名可以是字符串或符号。`Object.assign()` 会复制两者。

+   属性定义不会从源中复制，因此你需要使用 `Object.getOwnPropertyDescriptor()` 和 `Object.defineProperty()`。

+   它会忽略带有 `null` 和 `undefined` 值的键的复制。

# 摘要

在本章中，我们学习了 ES6 为处理数字、字符串、数组和对象添加的新特性。我们看到了数组如何在数学密集型应用中影响性能，以及如何使用数组缓冲区来替代。我们还了解了 ES6 提供的新集合对象。

在下一章中，我们将学习关于符号和迭代协议的内容，并且我们会发现 `yield` 关键字和生成器。
