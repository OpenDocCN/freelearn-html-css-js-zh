# 第三章。JavaScript 入门

理解 JavaScript 对于编写 Node.js 应用程序非常重要。JavaScript 不是一个庞大或复杂的语言，但它可能看起来有些不寻常，并且有一些需要注意的怪癖和陷阱。

ECMAScript 2015（之前称为 ES6）的最新发布引入了许多新的语言特性，使 JavaScript 编程更加容易和安全。然而，并非所有 ES2015 特性都已在所有实现中可用。但是，我们将在本章中提到的所有 ES2015 特性都在 Node.js 以及大多数其他 JavaScript 环境中可用。

在本章中，我们将熟悉 JavaScript，以便我们可以自信地编写 Node.js 应用程序。我们将涵盖以下主题：

+   JavaScript 的类型系统

+   JavaScript 作为一种函数式编程语言

+   JavaScript 中的面向对象编程

+   JavaScript 的基于原型的继承

# 介绍 JavaScript 类型

JavaScript 是一种动态类型语言。这意味着类型是在运行时检查的，当你尝试对变量进行操作时，而不是由编译器检查。例如，以下代码是有效的 JavaScript 代码：

```js
var myVariable = 0; 
console.log(typeof myVariable); // Prints "number"
myVariable = "1";
console.log(typeof myVariable); // Prints "string"
```

虽然变量确实有类型，但这种类型可能会在变量的整个生命周期中发生变化。

JavaScript 还试图在可能的情况下隐式转换类型，例如，使用相等运算符：

```js
console.log(2 == "2"); // Prints "true"
```

虽然这可能在前端 JavaScript 中有意义（例如，与表单输入的值进行比较），但通常来说，它更可能是错误或混淆的来源。因此，建议始终使用严格的相等和不相等运算符：

```js
console.log(2 === "2"); // Prints "false"
console.log(2 !== "2"); // Prints "true"
```

## JavaScript 原始类型

JavaScript 有少量原始类型，类似于 C# 和 Java。这些是字符串、数字和布尔值，以及特殊的单值类型 null 和 undefined。ES2015 还添加了 symbol 类型，但在这里我们不会介绍它，因为它的用例更为高级。

**字符串**是不可变的，就像在 C# 和 Java 中一样。连接字符串会创建一个新的字符串实例。字符串字面量可以用双引号（如在 C# 或 Java 中）或单引号定义。这些可以互换使用（通常使用起来更简单，可以避免转义）。

ES2015 还引入了对模板字符串的支持，这些字符串使用反引号定义，并可以包含插值表达式。

这里有一些定义相同字符串的方法：

```js
var singleQuoted = '"Hey", I said, "I\'m a string"';
var doubleQuoted = "\"Hey\", I said, \"I'm a string\"";
console.log(doubleQuoted === singleQuoted); // Prints "true"

var expression = 'Hey';
var templated = `"${expression}", I said, "I'm a string"`;
console.log(templated === singleQuoted); // Prints "true"
```

**数字**是 JavaScript 唯一的内置数值类型。它是一个双精度 64 位浮点数，类似于 C# 或 Java 中的 `double`。它有特殊的值 `NaN`（不是一个数字）和 `Infinity`，用于表示无法用其他方式表示的值：

```js
console.log(1 / 0); // Prints "Infinity"
console.log(Infinity + 1); // Prints "Infinity" 
console.log((1 / 0) === (2 / 0)); // Prints "true"

var notANumber = parseInt("foo");
console.log(notANumber); // Prints "NaN"
console.log(notANumber === NaN); // Prints "false"
console.log(isNaN(notANumber)); // Prints "true"
```

### 注意

注意，尽管只有一个 `NaN` 值，但它不会被视为等于自身。JavaScript 提供了特殊的 `isNaN` 函数来测试变量是否包含 `NaN` 值。

**null**类型有一个实例，由字面量`null`表示，就像在 C#或 Java 中一样。JavaScript 也有**undefined**类型。从未被分配的变量或参数将具有`undefined`值：

```js
var declared;
console.log(typeof declared); // Prints "undefined"
console.log(declared === undefined); // Prints "true"

console.log(typeof undeclared); // Prints "undefined"
console.log(undeclared === undefined); // throws ReferenceError
```

注意，我们的`undeclared`标识符在正常代码中不能作为变量访问，因为它没有被声明。然而，我们可以将它传递给`typeof`运算符，它评估为`undefined`类型。

# 函数式面向对象编程

JavaScript 是一种函数式面向对象编程语言。然而，它与 C#或 Java 等其他面向对象编程语言相当不同。尽管语法相似，但有一些重要的区别。

## JavaScript 中的函数式编程

在 JavaScript 中，函数是一等对象。这意味着函数可以像任何其他对象一样被对待：它们可以动态创建，分配给变量，或作为参数传递给方法。

这使得指定事件回调或使用**高阶函数**进行更函数式编程变得非常容易。高阶函数是接受其他函数作为参数，或返回另一个函数的函数。以下是一个简单的例子，首先以命令式风格然后以函数式风格过滤数字数组。注意，此示例还展示了 JavaScript 的**数组字面量表示法**，用于创建数组，使用方括号。它还演示了 JavaScript 的条件构造和其中一个循环构造，这些在其他语言中应该是熟悉的：

```js
var numbers = [1,2,3,4,5,6,7,8];

var filteredImperatively = [];
for (var i = 0; i < numbers.length; ++i) {
    var number = numbers[i];
    if (number % 2 === 0) {
        filteredImperatively.push(number);
    }
}
console.log(filteredImperatively); // Prints [2, 4, 6, 8]

var filteredFunctionally =
    numbers.filter(function(x) { return x % 2 === 0; });
console.log(filteredFunctionally); // Prints [2, 4, 6, 8]
```

示例中的第二种方法使用函数表达式在行内定义一个新的匿名函数。通常，这被称为 lambda 表达式（数学中的 lambda 演算之后）。这个函数被传递给 JavaScript 数组上可用的内置`filter`表达式。

在 C#中，最初只能使用委托来执行赋值和传递行为。自 C# 3.0 以来，对 lambda 表达式的支持使得使用函数的方式更加简单。这允许更函数式的编程风格，例如，使用 C#的**语言集成查询**（**LINQ**）功能。

在 Java 中，长期以来没有一种让函数独立存在的方法。你必须在一个（可能是匿名）类上定义一个方法，并将这个方法传递出去，这增加了大量的样板代码。Java 8 以类似于 C#的方式引入了对 lambda 表达式的支持。

虽然 C# 和 Java 可能需要一段时间才能赶上，但你可能会想，JavaScript 现在可能落后了。与 C# 和 Java 中的 lambda 语法相比，JavaScript 定义新函数的语法相当笨拙。

这尤其不幸，因为 JavaScript 使用类似于 C 语言的语法，以便与其他语言如 Java 兼容！在 ES2015 中，**箭头函数**解决了这个问题，允许我们将前面的示例重写如下：

```js
var numbers = [1,2,3,4,5,6,7,8];
var filteredFunctionally = numbers.filter(x => x % 2 === 0);
console.log(filteredFunctionally); // Prints [2, 4, 6, 8]
```

这是一个带有单个参数和单个表达式的简单箭头函数。在这种情况下，表达式是隐式返回的。

### 注意

在箭头函数中，可以将 `=>` 符号读作 *goes to*。

箭头函数可以有多个（或零个）参数，在这种情况下，它们必须被括号包围。如果函数体被大括号包围，它可能包含多个语句，在这种情况下没有隐式返回。这些语法规则与 C# 中的 lambda 表达式语法完全相同。

这里是一个更复杂的箭头函数表达式，它返回两个参数中的最大值：

```js
var max = (a, b) => {
    if (a > b) {
        return a;
    } else {
        return b;
    }
};
```

## 理解 JavaScript 中的作用域

传统上，在 JavaScript 中，只有两种可能的变量作用域：全局和函数。也就是说，一个标识符（变量名）是在全局定义的，或者是对整个函数定义的。这可能会导致一些令人惊讶的行为，例如：

```js
function scopeDemo() {
    for (var i = 0; i < 10; ++i) {
        var j = i * 2;
    }
    console.log(i, j);
}
scopeDemo();
```

在大多数其他语言中，您会期望 `i` 在 `for` 循环的整个过程中存在，而 `j` 在每次循环迭代中存在。因此，您会期望这个函数记录 `undefined undefined`。实际上，它记录的是 `10 18`。这是因为变量不是作用域到 `for` 循环的块中，而是作用域到整个函数。因此，前面的代码等同于以下代码：

```js
function scopeDemo() {
    var i, j;
    for (i = 0; i < 10; ++i) {
        j = i * 2;
    }
    console.log(i, j);
}
scopeDemo();
```

JavaScript 将所有变量声明视为在函数顶部创建的。这被称为 **变量提升**。虽然一致，但这可能会令人困惑并导致微妙的错误。

ES2015 引入了 `let` 关键字用于声明变量。这与 `var` 的行为完全相同，只是变量是块作用域的。还有一个 `const` 关键字，它与 `let` 的行为相同，只是不允许重新赋值。建议您始终使用 `let` 而不是 `var`，并在可能的情况下使用 `const`。以下代码为例：

```js
function scopeDemo() {
    "use strict";
    for (let i = 0; i < 10; ++i) {
        let j = i * 2;
    }
    console.log(i, j); // Throws ReferenceError: i is not defined
}
scopeDemo();
```

注意前面示例中的 `"use strict"` 字符串。我们将在下一节中讨论这个问题。

### 严格模式

`"use strict"` 字符串是给 JavaScript 解释器的提示，以启用 **严格模式**。这使得语言更安全，因为它将某些语言用法视为错误。例如，在严格模式下，如果误输变量名，则会在全局级别定义一个新变量，而不是引发错误。

严格模式现在也被一些浏览器用于启用 JavaScript 最新版本中的功能，例如之前显示的 `let` 和 `const` 关键字。如果您在浏览器中运行这些示例，您可能会发现前面的列表在没有严格模式的情况下无法工作。

在任何情况下，您都应该在所有生产代码中始终启用严格模式。`"use strict"` 字符串影响当前作用域中的所有代码（即 JavaScript 的传统函数或全局作用域），因此通常应放置在函数的顶部（或 Node.js 中模块脚本文件的顶部）。

## JavaScript 中的面向对象编程

任何不是 JavaScript 内置原始类型（字符串、数字、null 等）的东西都是一个 **对象**。这包括函数，正如我们在上一节中看到的。函数只是可以带参数调用的特殊类型的对象。数组是具有类似列表行为的特殊类型的对象。所有对象（包括这两种特殊类型）都可以有属性，属性只是带有值的名称。你可以把 JavaScript 对象想象成一个具有字符串键和对象值的字典。

可以使用对象字面量表示法创建具有属性的对象，如下例所示：

```js
var myObject = {
    myProperty: "myValue",
    myMethod: function() {
        return `myProperty has value "${this.myProperty}"`;
    }
};
console.log(myObject.myMethod());
```

即使你没有写过任何 JavaScript，你也可能对这个表示法很熟悉，因为它是 JSON 的基础。请注意，方法只是一个恰好具有函数值的对象属性。还要注意，在方法内部，我们可以使用 `this` 关键字来引用包含的对象。

最后，请注意，我们不需要为我们的对象定义一个类。JavaScript 在面向对象的语言中很独特，因为它实际上没有类。

### 无类编程

在大多数面向对象的语言中，我们可以在类中声明方法，供所有对象实例使用。我们还可以通过继承在类之间共享行为。

假设我们有一个包含大量点的图。这些点可能是由动态创建的具有一些共同行为的对象表示。我们可以这样实现点：

```js
function createPoint(x, y) {
    return {
        x: x,
        y: y,
        isAboveDiagonal: function() {
            return this.y > this.x;
        }
    };
}

var myPoint = createPoint(1, 2);
console.log(myPoint.isAboveDiagonal()); // Prints "true"
```

这种方法的一个问题是，`isAboveDiagonal` 方法在我们的图上的每个点上都被重新定义，因此占用更多的内存空间。

我们可以使用 **原型继承** 来解决这个问题。虽然 JavaScript 没有类，但对象可以继承自其他对象。每个对象都有一个 **原型**。如果我们尝试访问一个对象的属性，而这个属性不存在，解释器将在对象的原型上查找具有相同名称的属性。如果那里也不存在，它将检查原型的原型，依此类推。原型链将以内置的 `Object.prototype` 结束。

我们可以这样为我们点的对象实现：

```js
var pointPrototype = {
    isAboveDiagonal: function() {
        return this.y > this.x;
    }
};

function createPoint(x, y) {
    var newPoint = Object.create(pointPrototype);
    newPoint.x = x;
    newPoint.y = y;
    return newPoint;
}

var myPoint = createPoint(1, 2); 
console.log(myPoint.isAboveDiagonal()); // Prints "true"
```

`isAboveDiagonal` 方法现在只在内存中的 `pointPrototype` 对象上存在一次。

当我们尝试在一个单独的点对象上调用 `isAboveDiagonal` 方法时，它不存在，而是在原型上找到。

注意，这告诉我们关于 `this` 关键字的重要信息。它实际上指的是当前函数被调用的对象，而不是定义它的对象。

#### 使用 new 关键字创建对象

我们可以以略微不同的形式重写前面的代码示例，如下所示：

```js
var pointPrototype = {
    isAboveDiagonal: function() {
        return this.y > this.x;
    }
}

function Point(x, y) {
    this.x = x;
    this.y = y;
}

function createPoint(x, y) {
    var newPoint = Object.create(pointPrototype);
 Point.apply(newPoint, arguments);
    return newPoint;
}

var myPoint = createPoint(1, 2);
```

这使用了特殊的 `arguments` 对象，它包含当前函数的参数数组。它还使用了 `apply` 方法（所有函数都可用）来以相同的参数在 `newPoint` 对象上调用 `Point` 函数。

目前，我们的 `pointPrototype` 对象与 `Point` 函数并没有特别紧密的联系。让我们通过使用 `Point` 函数的 `prototype` 属性来解决这个问题。这是一个默认情况下所有函数都有的内置属性。它只包含一个空对象，我们可以向其中添加额外的属性：

```js
function Point(x, y) {
    this.x = x;
    this.y = y;
}

Point.prototype.isAboveDiagonal = function() {
    return this.y > this.x;
}

function createPoint(x, y) {
 var newPoint = Object.create(Point.prototype);
    Point.apply(newPoint, arguments);
    return newPoint;
}

var myPoint = createPoint(1, 2);
```

这可能看起来是一种不必要的复杂做事方式。然而，JavaScript 有一个特殊的操作符，允许我们极大地简化之前的代码，如下所示：

```js
function Point(x, y) {
    this.x = x;
    this.y = y;
}

Point.prototype.isAboveDiagonal = function() {
    return this.y > this.x;
}

var myPoint = new Point(1, 2);
```

`new` 操作符的行为与上一个示例中的 `createPoint` 函数相同。有一个小的例外：如果 `Point` 函数实际上返回了一个值，那么就会使用这个值而不是 `newPoint`。在 JavaScript 中，如果函数打算与 `new` 操作符一起使用，通常以大写字母开头。

### 使用类进行编程

虽然 JavaScript 实际上并没有类，但 ES2015 引入了一个新的 `class` 关键字。这使得以可能比其他面向对象语言更熟悉的方式实现共享行为和继承成为可能。

我们之前代码的等价物看起来如下所示：

```js
class Point {
    constructor(x, y) {
        this.x = x;
        this.y = y;
    }

    isAboveDiagonal() {
        return this.y > this.x;
    }
}

var myPoint = new Point(1, 2);
```

注意，这实际上与我们的上一段代码是等价的。`class` 关键字只是设置已讨论的基于原型的继承的语法糖。

#### 基于类的继承

如前所述，一个对象的原型可能还有另一个原型，从而允许继承链。使用上一节中提到的基于原型的方法设置这样的链变得相当复杂。使用类关键字会更直观，如下面的示例（可能用于绘制带有误差棒的图表）所示：

```js
class UncertainPoint extends Point {
    constructor(x, y, uncertainty) {
        super(x, y);
        this.uncertainty = uncertainty;
    }

    upperLimit() {
        return this.y + this.uncertainty;
    }

    lowerLimit() {
        return this.y - this.uncertainty;
    }
}

var myUncertainPoint = new Point(1, 2, 0.5);
```

# 概述

在本章中，我们介绍了 JavaScript 的类型系统，理解了函数在 JavaScript 中的第一类对象，看到了 JavaScript 与其他面向对象语言的不同之处，使用了原型和类来实现继承，并学习了 ECMAScript 2015（ES6）的新特性，这些特性使语言更安全、更易于使用。

现在你已经对 JavaScript 有了一个坚实的基础，你可以自信地开始编写 Node.js 应用程序。在下一章中，我们将扩展我们的 Express 项目，并了解 Node.js 中的模块系统如何允许我们构建我们的代码库。
