# 第五章：运行时 —— 闭包和原型

在上一章中，我们学习了 TypeScript/JavaScript 运行时的某些方面，这将帮助我们理解在后续章节中实现某些函数式编程技术的方法。在本章中，我们将探索 TypeScript/JavaScript 运行的另外两个方面：

+   原型

+   闭包

探索了这两个概念之后，我们最终将准备好深入探讨主要函数式编程技术的实现和应用。

# 原型

当我们编译 TypeScript 程序时，所有类和对象都成为 JavaScript 对象。然而，即使编译没有错误，我们有时也可能在运行时遇到意外的行为。为了能够识别和理解这种行为的起因，我们需要对 JavaScript 运行时有一个很好的理解。我们需要理解的主要概念之一是类和继承在运行时的如何工作。

运行时继承系统使用原型继承模型。在原型继承模型中，对象从对象继承，没有类可用。然而，我们可以使用原型来模拟类。让我们看看它是如何工作的。

在运行时，对象有一个内部属性称为 `prototype`。`prototype` 属性的值是一个包含一些属性（数据）和方法（行为）的对象。

在 TypeScript 中，我们可以使用基于类的继承系统：

```js
class Person {

    public name: string;
    public surname: string;
    public age: number = 0;

    public constructor(name: string, surname: string) {
        this.name = name;
        this.surname = surname;
    }

    public greet() {
        let msg = 'Hi! my name is ${this.name} ${this.surname}';
        msg += 'I'm ${this.age}';
    }

}
```

我们定义了一个名为 `Person` 的类。在运行时，这个类使用原型而不是类来声明：

```js
var Person = (function() {

    function Person(name, surname) {
        this.age = 0;
        this.name = name;
        this.surname = surname;
    }

    Person.prototype.greet = function() {
        let msg = "Hi! my name is " + this.name +
                    " " + this.surname;
        msg += "I'm " + this.age;
    };

    return Person;

})();
```

上述代码是 TypeScript 在针对 ES5 时输出的。`class` 关键字在 ES6 运行时得到支持，但它只是语法糖。

语法糖是编程语言内部的语法，旨在使事物更容易阅读或表达。这意味着 `class` 关键字只是一个帮助我们作为软件工程师更容易生活的辅助工具；内部，始终使用原型。

TypeScript 编译器使用一个立即执行的函数表达式（**IIFE**）包装对象定义（我们不会将其称为类定义，因为技术上它不是一个类）。在 IIFE 内部，我们可以找到一个名为 `Person` 的函数。如果我们检查这个函数并与 TypeScript 类进行比较，我们会注意到它接受与 TypeScript 类构造函数相同的参数。这个函数用于创建 `Person` 类的新实例。

在构造函数之后，我们可以看到 `greet` 方法的定义。正如我们所见，`prototype` 属性被用来将 `greet` 方法附加到 `Person` 类上。

# 实例属性与类属性

由于 JavaScript 是一种动态编程语言，我们可以在运行时向对象实例添加属性和方法；它们不需要是对象（类）本身的一部分：

```js
const name = "Remo";
const surname = "Jansen";

function Person(name, surname) {
      // instance properties
      this.name = name;
      this.surname = surname;
}

const person1 = new Person(name, surname);
person1.age = 27;
```

我们为名为`Person`的对象定义了一个构造函数，它接受两个变量（`name`和`surname`）作为参数。然后我们创建了一个`Person`对象的实例，并给它添加了一个名为`age`的新属性。我们可以使用`for…in`语句在运行时检查`person1`的属性：

```js
for(let property in person1) {
  console.log("property: " + property + ", value: '" +
  person1[property] + "'");
}
```

控制台输出将显示以下内容：

```js
property: name, value: 'Remo'
property: surname, value: 'Jansen'
property: age, value: 27
property: greet, value: 'function (city, country) {
    let msg = "Hi, my name is " + this.name + " " + this.surname;
    msg += "\nI'm from " + city + " " + country;
    console.log(msg);
}'
```

如我们所见，`age`已被添加为一个属性。所有这些属性都是实例属性，因为它们为每个新实例持有值。例如，如果我们创建`Person`的新实例，这两个实例都将持有它们自己的值：

```js
let person2 = new Person("John", "Wick");
person2.name; // "John"
person1.name; // "Remo"
```

我们使用`this`运算符定义了这些实例属性，因为当一个函数用作构造函数（使用`new`关键字）时，`this`运算符绑定到所构造的对象实例。前面的内容也解释了为什么我们可以通过对象的原型来定义实例属性：

```js
Person.prototype.name = name; // instance property
Person.prototype.surname = surname; // instance property
```

我们还可以声明类属性和方法（也称为静态属性）。实例属性和类属性之间的主要区别是，类属性和方法的价值在对象的所有实例之间共享。类属性通常用于存储静态值：

```js
function MathHelper() {
  //...
}

// class property
MathHelper.PI = 3.14159265359;
```

类方法也常被用作执行参数计算并返回结果的实用函数：

```js
function MathHelper() {
    // ...
}

// class property
MathHelper.PI = 3.14159265359;

// class method
MathHelper.areaOfCircle = function(radius) {
  return radius * radius * MathHelper.PI;
}
```

请注意，前面的代码片段在 JavaScript 中是有效的，但在 TypeScript 中会抛出编译错误。

在前面的示例中，我们从类方法（`areaOfCircle`）中访问了一个类属性（`PI`）。我们可以从实例方法中访问类属性，但不能从类属性或方法中访问实例属性或方法。我们可以通过将`PI`声明为实例属性而不是类属性来演示这一点：

```js
function MathHelper() {
  // instance property
  this.PI = 3.14159265359;
}
```

如果我们尝试从类方法中访问`PI`，它将是未定义的：

```js
// class method
MathHelper.areaOfCircle = function(radius) {
  return radius * radius * this.PI; // this.PI is undefined
}

MathHelper.areaOfCircle(5); // NaN
```

我们不应该从实例方法中访问类方法或属性，但有一种方法可以做到。我们可以通过使用原型的`constructor`属性来实现，如下面的示例所示：

```js
function MathHelper () { /* ... */ }

// class property
MathHelper.PI = 3.14159265359;

// instance method
MathHelper.prototype.areaOfCircle = function(radius) {
 return radius * radius * this.constructor.PI;
}

const math = new MathHelper ();
console.log(MathHelper.areaOfCircle(5)); // 78.53981633975
```

我们可以通过原型对象的`constructor`属性从`areaOfCircle`实例方法访问`PI`类属性，因为这个属性返回一个指向对象构造函数的引用。

在`areaOfCircle`内部，`this`运算符返回对对象原型的引用：

```js
this === MathHelper.prototype; // true
```

我们可以推断出`this.constructor`等于`MathHelper.prototype.constructor`，因此`MathHelper.prototype.constructor`等于`MathHelper`。

在 TypeScript 中，我们可以使用`static`关键字定义类属性：

```js
class MathHelper {

    // class property
    public static PI = 3.14159265359;

    // class method
    public static areaOfCircle(radius: number) {
        return radius * radius * MathHelper.PI;
    }
}
```

# 原型继承

你可能想知道`extends`关键字是如何工作的。让我们创建一个新的 TypeScript 类，它从`Person`类继承，以理解它：

```js
class SuperHero extends Person {

    public superpower: string;

    public constructor(
        name: string,
        surname: string,
        superpower: string
    ) {
        super(name, surname);
        this.superpower = superpower;
    }

    public userSuperPower() {
        return 'I'm using my ${this.superpower}';
    }

}
```

前面的类名为`SuperHero`，它扩展了`Person`类。它有一个额外的属性（`superpower`）和一个方法（`useSuperPower`）。

我们需要将之前的代码片段编译成 JavaScript 代码，以便我们可以检查继承在运行时是如何实现的。编译器将生成一个名为`__extends`的 polyfill 函数，该函数旨在作为与旧版 JavaScript 兼容的`extends`关键字的替代品：

```js
var __extends = this.__extends || function (d, b) {
    for (var p in b) if (b.hasOwnProperty(p)) d[p] = b[p];
    function __() { this.constructor = d; }
    __.prototype = b.prototype;
    d.prototype = new __();
};
```

请注意，先前的代码片段在 TypeScript 的最新版本中稍微复杂一些。在这里我们将使用之前版本的代码，因为它包含的条件较少，更容易理解。

这段代码是由 TypeScript 生成的。尽管它是一小段代码，但它展示了本章几乎包含的每一个概念，理解它可能相当具有挑战性。

在函数表达式第一次评估之前，`this`操作符指向全局对象，该对象不包含名为`__extends`的方法。这意味着`__extends`变量是未定义的。

当函数表达式第一次评估时，函数表达式的值（一个匿名函数）被分配给全局作用域中的`__extends`属性。

TypeScript 为包含`extends`关键字的每个 TypeScript 文件生成一个函数表达式。然而，函数表达式只被评估一次（当`__extends`变量未定义时）。这种行为是通过第一行的条件语句实现的：

```js
var __extends = this.__extends || function (d, b) { // ...
```

当执行上一行代码时，函数表达式将被评估。函数表达式的值是一个匿名函数，它被分配给全局作用域中的`__extends`变量。因为我们在全局作用域中，`var __extends`和`this. __extends`在此点指向相同的变量。

当一个新文件被执行时，`__extends`变量已经在全局作用域中可用，函数表达式不会被评估。这意味着函数表达式的值只被分配给`__extends`变量一次，即使代码片段被多次执行。

现在我们来关注一下`__extends`变量（匿名函数）：

```js
function (d, b) {
    for (var p in b) if (b.hasOwnProperty(p)) d[p] = b[p];
    function __() { this.constructor = d; }
    __.prototype = b.prototype;
    d.prototype = new __();
}
```

此函数接受两个名为`d`和`b`的参数。当我们调用它时，我们应该传递一个派生对象构造函数（`d`）和一个基对象构造函数（`b`）。

匿名函数内的第一行代码迭代基类中的每个类属性和方法，并在派生类中创建它们的副本：

```js
for (var p in b) if (b.hasOwnProperty(p)) d[p] = b[p];
```

当我们使用`for…in`语句迭代对象的实例到`a`时，它将迭代对象的实例属性。然而，如果我们使用`for…in`语句迭代对象构造函数的属性，该语句将迭代其类属性。在先前的例子中，`for…in`语句用于继承对象的类属性和方法。要继承实例属性，我们将复制对象的原型。

第二行声明了一个新的构造函数 `__`，在其中，使用 `this` 操作符来访问其原型。

```js
function __() { this.constructor = d; }
```

原型包含一个名为 `constructor` 的特殊属性，它返回对对象构造函数的引用。在这个点上，名为 `__` 和 `this.constructor` 的函数指向相同的变量。然后，派生对象构造函数（`d`）的值被分配给 `__` 构造函数。

在第三行，将基对象构造函数的原型对象的值分配给 `__` 对象构造函数的原型：

```js
__.prototype = b.prototype;
```

在最后一行，`__` 函数作为构造函数使用 `new` 关键字调用，其结果被分配给派生类（`d`）的原型。通过执行所有这些步骤，我们已经完成了调用以下内容所需的所有操作：

```js
var instance = new d():
```

这样做之后，我们将得到一个包含派生类（`d`）和基类（`b`）所有属性的对象。此外，由派生构造函数（`d`）构建的任何实例对象都将成为派生类的实例，同时继承自基类（`b`）的类和实例属性和方法。

我们可以通过检查定义 `SuperHero` 类的运行时代码来看到函数的作用：

```js
var SuperHero = (function (_super) {

    __extends(SuperHero, _super);

    function SuperHero(name, surname, superpower) {
        _super.call(this, name, surname);
        this.superpower = superpower;
    }

    SuperHero.prototype.userSuperPower = function () {
        return "I'm using my " + superpower;
    };

    return SuperHero;

})(Person);
```

我们在这里再次看到了一个立即执行函数表达式（IIFE）。这次，IIFE 将 `Person` 对象构造函数作为参数。在函数内部，我们将使用名称 `_super` 来引用这个参数。在 IIFE 内部，调用 `__extends` 函数，并将 `SuperHero`（派生类）和 `_super`（基类）参数传递给它。

在下一行中，我们可以找到 `SuperHero` 对象构造函数和 `useSuperPower` 函数的声明。在声明之前，我们可以将 `SuperHero` 作为 `__extend` 的参数使用，因为函数声明会被提升到作用域的顶部。

函数表达式不会被提升。当我们在一个函数表达式中将函数赋值给变量时，变量会被提升，但它的值（函数本身）不会被提升。

在 `SuperHero` 构造函数内部，使用 `call` 方法调用了基类（`Person`）的构造函数：

```js
_super.call(this, name, surname);
```

正如我们在上一章中发现的，我们可以使用 `call` 方法在函数上下文中设置 `this` 操作符的值。在这种情况下，我们传递了 `this` 操作符，它指向正在创建的 `SuperHero` 实例：

```js
function Person(name, surname) {
    // this points to the instance of SuperHero being created
    this.name = name;
    this.surname = surname;
}
```

# 原型链和属性遮蔽

当我们尝试访问一个对象的属性或方法时，运行时会搜索该属性或方法在对象自己的属性和方法中。如果找不到，运行时会继续通过遍历整个继承树来搜索对象的继承属性。因为派生对象通过 `prototype` 属性与基对象链接，所以我们称这个继承树为 **原型链**。

让我们来看一个例子。我们将声明两个简单的 TypeScript 类，分别命名为 `Base` 和 `Derived`：

```js
class Base {
    public method1() { return 1; }
    public method2() { return 2; }
}

class Derived extends Base {
    public method2() { return 3; }
    public method3() { return 4; }
}
```

现在我们将检查 TypeScript 生成的 JavaScript 代码：

```js
var Base = (function () {
    function Base() {
    }
    Base.prototype.method1 = function () { return 1; };
    Base.prototype.method2 = function () { return 2; };
    return Base;
})();

var Derived = (function (_super) {
    __extends(Derived, _super);
    function Derived() {
        _super.apply(this, arguments);
    }
    Derived.prototype.method2 = function () { return 3; };
    Derived.prototype.method3 = function () { return 4; };
    return Derived;
})(Base);
```

我们可以创建`Derived`类的实例：

```js
const derived = new Derived();
```

`new`运算符创建了一个从`Base`类继承的对象实例。

如果我们尝试访问名为`method1`的方法，运行时会从实例属性中找到它：

```js
console.log(derived.method1()); // 1
```

实例还有一个名为`method2`的属性（值为`2`），但还有一个继承的属性名为`method2`（值为`3`）。对象的属性（值为`3`的`method2`）阻止了对原型属性（值为`2`的`method2`）的访问。这被称为**属性遮蔽**：

```js
console.log(derived.method2()); // 3
```

实例没有自己的名为`method3`的属性，但在其原型链中有一个名为`method3`的属性：

```js
console.log(derived.method3()); // 4
```

实例以及原型链中的对象（`Base`类）都没有名为`method4`的属性：

```js
console.log(derived.method4()); // error
```

# 访问对象的原型

原型可以通过三种不同的方式访问：

+   `Person.prototype`

+   `Object.getPrototypeOf(person)`

+   `person.__proto__`

使用`__proto__`是有争议的，并且被许多经验丰富的软件工程师所不推荐。它从未最初包含在 ECMAScript 语言规范中，但现代浏览器还是决定无论如何实现它。如今，`__proto__`属性已经被标准化在 ECMAScript 6 语言规范中，并将在未来得到支持，但它仍然是一个应该避免的慢操作，如果性能是一个考虑因素的话。

# 闭包

闭包是 JavaScript 和 TypeScript 中最强大的特性之一，但它们也是最容易误解的特性之一。*Mozilla 开发者网络*将闭包定义为如下：

闭包是引用独立（自由）变量的函数。换句话说，闭包中定义的函数“记住”了它被创建的环境。

我们将独立（自由）变量理解为在它们被创建的词法作用域之外持续存在的变量。让我们看一个例子：

```js
function makeArmy() {
    const shooters = [];
    for (let i = 0; i < 10; i++) {
        const shooter = () => { // a shooter is a function
            console.log(i); // which should display it's number
        };
        shooters.push(shooter);
    }
    return shooters;
}
```

请注意，前面的例子是一个 JavaScript 示例。

我们已经声明了一个名为`makeArmy`的函数。在函数内部，我们创建了一个名为`shooters`的函数数组。`shooters`数组中的每个函数都将显示一个数字，其值是从`for`语句内部的变量`i`设置的。现在我们将调用`makeArmy`函数：

```js
const army = makeArmy();
```

`army`变量现在应该包含函数的`shooters`数组。然而，如果我们执行以下代码，我们会注意到一个问题：

```js
army[0](); // 10 (expected 0)
army[5](); // 10 (expected 5)
```

前面的代码片段没有按预期工作，因为我们犯了与闭包相关的一个最常见的错误。当我们把`shooter`函数声明在`makeArmy`函数内部时，我们无意中创建了一个闭包。

这是因为分配给 `shooter` 的函数是闭包。闭包可以访问包围它们的（`makeArmy` 函数的作用域）环境中的变量。已经创建了十个闭包函数，但每个函数都共享同一个单一的环境。当 `shooter` 函数执行时，循环已经完成，共享的 `i` 变量（所有闭包函数共享）已经指向最后一个条目（`10`）。

在这个情况下，一个解决方案是使用更多的闭包：

```js
function makeArmy() {
    const shooters = [];
    for (let i = 0; i < 10; i++) {
        ((index: number) => {
            const shooter = () => {
                console.log(index);
            };
            shooters.push(shooter);
        })(i);
    }
    return shooters;
}

const army = makeArmy();
army[0](); // 0
army[5](); // 5
```

请注意，前面的示例是一个 TypeScript 示例。

这正如预期的那样工作。不是 `shooter` 函数共享一个单一的环境，立即调用的函数为每个函数创建了一个新的环境，其中 `i` 指向相应的值。

# 由闭包驱动的静态变量

在上一节中，我们了解到，当闭包上下文中的变量可以在类的多个实例之间共享时，这意味着该变量表现得像静态变量。现在我们将看到如何使用闭包创建表现得像静态变量的变量和方法。让我们首先声明一个名为 `Counter` 的 TypeScript 类：

```js
class Counter {

 private static _COUNTER = 0;

 public increment() {
 this._changeBy(1);
 }

 public decrement() {
 this._changeBy(-1);
 }

 public value() {
 return Counter._COUNTER;
 }

 private _changeBy(val: number) {
 Counter._COUNTER += val;
 }

}
```

请注意，前面的示例是一个 TypeScript 示例。

前面的类包含一个名为 `_COUNTER` 的静态成员。TypeScript 编译器将其转换为以下代码：

```js
var Counter = (function () {

    function Counter() {
    }

    Counter.prototype._changeBy = function (val) {
        Counter._COUNTER += val;
    };

    Counter.prototype.increment = function () {
        this._changeBy(1);
    };

    Counter.prototype.decrement = function () {
        this._changeBy(-1);
    };

    Counter.prototype.value = function () {
        return Counter._COUNTER;
    };

    Counter._COUNTER = 0;
    return Counter;

})();
```

请注意，前面的代码片段是 TypeScript 编译器生成的编译输出。

如我们所观察到的，TypeScript 编译器将静态变量声明为类属性（而不是实例属性）。编译器使用类属性，因为类属性在类的所有实例之间共享。问题是私有变量在运行时并不是私有的。

或者，我们可以编写一些 JavaScript（记住，所有有效的 JavaScript 都是有效的 TypeScript）代码来使用闭包模拟静态属性：

```js
var Counter = (function() {

    // closure context

    let _COUNTER = 0;

    function changeBy(val: number) {
        _COUNTER += val;
    }

    interface Counter {
        increment: () => void;
        decrement: () => void;
        value: () => number;
    }

    interface CounterConstructor {
        new(): Counter;
    }

    function Counter() {};

    // closure functions

    Counter.prototype.increment = function() {
      changeBy(1);
    };

    Counter.prototype.decrement = function() {
      changeBy(-1);
    };

    Counter.prototype.value = function() {
      return _COUNTER;
    };

    return (Counter as unknown) as CounterConstructor;

})();
```

请注意，前面的示例是一个 TypeScript 示例。

前面的代码片段声明了一个名为 `Counter` 的类。该类有一些用于增加、减少和读取名为 `_COUNTER` 的变量的方法。`_COUNTER` 变量本身不是对象原型的部分。

所有的 `Counter` 类实例将共享同一个上下文，这意味着上下文（`_COUNTER` 变量和 `changeBy` 函数）将表现得像一个单例。

单例模式需要一个对象被声明为静态变量以避免在需要时创建其实例。因此，对象实例由应用程序中的所有组件共享。单例模式在不需要类唯一实例的情况下经常被使用，从而在不必要的情况下引入了不必要的限制，并将全局状态引入了应用程序。

因此，我们现在知道可以使用闭包来模拟静态变量：

```js
let counter1 = new Counter();
let counter2 = new Counter();
console.log(counter1.value()); // 0
console.log(counter2.value()); // 0
counter1.increment();
counter1.increment();
console.log(counter1.value()); // 2
console.log(counter2.value()); // 2 (expected 0)
counter1.decrement();
console.log(counter1.value()); // 1
console.log(counter2.value()); // 1 (expected 0)
```

如我们所见，前面的例子并没有按预期工作，因为 `Counter` 的两个实例共享内部计数器。我们将在下一节中学习如何解决这个问题。

# 由闭包支持的私有成员

在上一节中，我们了解到闭包可以访问超出它们创建的词法作用域的持久变量。这些变量不是函数的原型或主体的部分，但它们是函数上下文的一部分。

由于我们无法直接调用函数的上下文，上下文变量和方法可以用来模拟私有成员。使用闭包来模拟私有成员（而不是 TypeScript 的私有访问修饰符）的主要优势是，闭包将在运行时防止对私有成员的访问。

TypeScript 避免在运行时模拟私有属性，因为如果我们尝试访问私有成员，编译器将在编译时抛出错误。TypeScript 避免使用闭包来模拟私有成员，以便提高应用程序性能。如果我们向我们的类之一添加或移除访问修饰符，生成的 JavaScript 代码将完全不会改变。这意味着类的私有成员在运行时变成了公共成员。

然而，使用闭包在运行时模拟私有属性是可能的。让我们看看一个例子：

```js
function makeCounter() {

    // closure context
    let _COUNTER = 0;

    function changeBy(val: number) {
        _COUNTER += val;
    }

    class Counter {

        public increment() {
            changeBy(1);
        }

        public decrement() {
            changeBy(-1);
        }

        public value() {
            return _COUNTER;
        }

    }

    return new Counter();

}
```

请注意，前面的例子是一个 TypeScript 示例。

上述类几乎与我们之前声明的类相同，目的是为了演示如何使用闭包在运行时模拟静态变量。

这次，每次我们调用 `makeCounter` 函数时，都会创建一个新的闭包上下文，因此每个新的 `Counter` 实例都会记住一个独立的环境（`_COUNTER` 和 `changeBy`）：

```js
let counter1 = makeCounter();
let counter2 = makeCounter();
console.log(counter1.value()); // 0
console.log(counter2.value()); // 0
counter1.increment();
counter1.increment();
console.log(counter1.value()); // 2
console.log(counter2.value()); // 0 (expected 0)
counter1.decrement();
console.log(counter1.value()); // 1
console.log(counter2.value()); // 0 (expected 0)
```

由于上下文无法直接访问，我们可以这样说，`_COUNTER` 变量和 `changeBy` 函数即使在运行时也是私有成员：

```js
console.log(counter1.counter); // Error
console.log(counter1.changeBy(2)); // Error
```

# 摘要

在本章中，我们更好地理解了运行时，这使我们不仅能够轻松解决运行时问题，而且还能编写更好的 TypeScript 代码。对闭包和原型的深入理解将使我们能够理解在后续章节中某些函数式编程技术的实现。

在下一章中，我们将学习如何实现许多基本的函数式编程技术。
