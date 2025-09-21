# 类

JavaScript 有类，它们提供了创建构造函数和处理继承的更简单、更清晰的语法。直到现在，JavaScript 从来没有类的概念，尽管它是一种面向对象的编程语言。来自其他编程语言背景的程序员常常因为缺乏类而发现理解 JavaScript 的面向对象模型和继承很困难。

在本章中，我们将通过类学习面向对象的 JavaScript：

+   JavaScript 数据类型

+   以传统方式创建对象

+   原始类型的构造函数

+   什么是类？

+   使用类创建对象

+   类中的继承

+   类的特性

# 理解面向对象的 JavaScript

在我们继续学习 ES6 类之前，让我们回顾一下 JavaScript 数据类型、构造函数和继承的知识。在学习类的同时，我们将比较基于构造函数和原型继承的语法与类的语法。因此，对这些主题有良好的掌握是很重要的。

# JavaScript 数据类型

JavaScript 变量持有（或存储）数据（或值）。变量持有的数据类型称为数据类型。在 JavaScript 中，有七种不同的数据类型：number、string、Boolean、null、undefined、symbol 和 object。

当涉及到存储对象时，变量持有对象引用（即内存地址）而不是对象本身。如果你来自 C/C++ 背景，你可以将它们与指针联系起来，但并不完全一样。

除了对象之外的所有数据类型都称为原始数据类型。

数组和函数实际上是 JavaScript 对象。很多事物在底层都是对象。

# 创建对象

在 JavaScript 中创建对象有两种方式：使用 `object` 字面量，或使用 `constructor`。当我们需要创建固定对象时使用 `object` 字面量，而当我们想要在运行时动态创建对象时使用 `constructor`。

让我们考虑一个可能需要使用 `constructor` 而不是 `object` 字面量的情况。以下是一个代码示例：

```js
const student = {  
    name: "Eden",  
    printName() {  
        console.log(this.name);  
    } 
} 
student.printName(); //Output "Eden"
```

在这里，我们使用 `object` 字面量创建了一个 `student` 对象，即 `{}` 符号。当你只想创建单个 `student` 对象时，这很有效。

但问题出现在你想要创建多个 `student` 对象时。显然，你不想多次编写之前的代码来创建多个 `student` 对象。这就是 `constructor` 发挥作用的地方。

当使用 `new` 关键字调用时，`function` 作为一个 `constructor`。`constructor` 创建并返回一个对象。在作为 `constructor` 调用时的 `function` 内部的 `this` 关键字指向新的对象实例，一旦 `constructor` 执行完成，新的对象就会自动返回。考虑以下示例：

```js
function Student(name) {  
    this.name = name; 
} 

Student.prototype.printName = function(){  
    console.log(this.name); 
}

const student1 = new Student("Eden"); 
const student2 = new Student("John"); 
student1.printName(); //Output "Eden" 
student2.printName(); //Output "John"
```

在这里，为了创建多个`student`对象，我们多次调用了`constructor`，而不是使用`object`字面量创建多个`student`对象。

要向`constructor`的实例添加方法，我们没有使用`this`关键字；相反，我们使用了`constructor`的`prototype`属性。我们将在下一节中了解更多关于为什么这样做，以及`prototype`属性是什么。

实际上，每个对象都必须属于一个`constructor`。每个对象都有一个继承属性名为`constructor`，指向对象的`constructor`。当我们使用`object`字面量创建对象时，`constructor`属性指向全局的`Object`的`constructor`。考虑以下示例来理解这种行为：

```js
var student = {} 
console.log(student.constructor == Object); //Output "true"
```

# 理解原型继承模型

每个 JavaScript 对象都有一个指向另一个对象的内嵌`[[prototype]]`属性，这个对象被称为其原型。这个原型对象有自己的原型，以此类推，直到遇到原型为 null 的对象。null 没有原型，并且作为原型链中的最后一个链接。

当尝试访问一个对象属性时，如果该属性在对象中找不到，那么就会在对象的原型中搜索该属性。如果仍然找不到，那么就会在原型对象的原型中搜索。这个过程会一直持续到原型链中遇到 null。这就是 JavaScript 中继承的工作方式。

由于 JavaScript 对象只能有一个原型，JavaScript 只支持单继承。

在使用`object`字面量创建对象时，我们可以使用特殊的`__proto__`属性或`Object.setPrototypeOf()`方法来为对象分配一个原型。JavaScript 还提供了一个`Object.create()`方法，我们可以使用它来创建一个具有指定原型的新的对象，因为`__proto__`在浏览器中不受支持，而`Object.setPrototypeOf()`方法看起来有点奇怪。

下面是一个代码示例，演示了在创建特定对象时使用`Object`字面量设置对象原型的不同方法：

```js
const object1 = { name: "Eden", __proto__: {age: 24} } 
const object2 = {name: "Eden" } 

Object.setPrototypeOf(object2, {age: 24}); 

const object3 = Object.create({age: 24}, {
    name: {value: "Eden"}
}); 

console.log(object1.name + " " + object1.age); 
console.log(object2.name + " " + object2.age); 
console.log(object3.name + " " + object3.age);
```

输出如下：

```js
Eden 24
Eden 24
Eden 24
```

在这里，`{age:24}`对象被称为基对象、超对象或父对象，因为它正在被继承。而`{name:"Eden"}`对象被称为派生对象、子对象或子对象，因为它继承另一个对象。

如果你在使用`object`字面量创建对象时没有为其分配原型，那么原型将指向`Object.prototype`属性。`Object.prototype`的原型是 null，因此导致原型链的结束。以下是一个示例来演示这一点：

```js
const obj = { name: "Eden" } 
console.log(obj.__proto__ == Object.prototype); //Output "true"
```

当使用`constructor`创建对象时，新对象的原型始终指向一个名为`prototype`的属性，该属性属于`function`对象。默认情况下，`prototype`属性是一个具有一个名为`constructor`的属性的对象。`constructor`属性指向该函数本身。考虑以下示例以理解此模型：

```js
function Student() { 
    this.name = "Eden"; 
} 
const obj = new Student(); 
console.log(obj.__proto__.constructor == Student); //Output "true" 
console.log(obj.__proto__ == Student.prototype); //Output "true"
```

要向`constructor`的实例添加新方法，我们应该像之前做的那样，将它们添加到`constructor`的`prototype`属性中。

我们之前没有使用`this`将方法添加到`constructor`中的原因是因为每个`constructor`的实例都将有一个方法副本，这并不非常节省内存。通过将方法附加到`constructor`的`prototype`属性，每个函数只有一个副本，所有实例都共享。为了理解这一点，请考虑以下示例：

```js
function Student(name) { 
    this.name = name; 
} 
Student.prototype.printName = function() { 
    console.log(this.name); 
} 

const s1 = new Student("Eden"); 
const s2 = new Student("John"); 

function School(name) { 
    this.name = name; 
    this.printName = function() { 
        console.log(this.name); 
    } 
} 

const s3 = new School("ABC");
const s4 = new School("XYZ"); 
console.log(s1.printName == s2.printName); 
console.log(s3.printName == s4.printName);
```

输出如下：

```js
true
false
```

在这里，`s1`和`s2`共享相同的`printName`函数，这减少了内存的使用，而`s3`和`s4`包含两个不同的`printName`函数，使得程序使用更多的内存。这是不必要的，因为这两个函数执行的是相同的事情。因此，我们将实例的方法添加到`constructor`的`prototype`属性中。

在`constructor`中实现继承层次结构并不像在`object`字面量中那样直接。这是因为子`constructor`需要调用父`constructor`以执行父构造函数的初始化逻辑，并且我们需要将父`constructor`的`prototype`属性的函数添加到子`constructor`的`prototype`属性中，以便我们可以使用它们与子`constructor`的对象一起使用。没有预定义的方法来完成所有这些。开发人员和 JavaScript 库有他们自己的方法来做这件事。我将向您展示最常见的方法。

下面是一个示例，演示如何在创建对象时实现继承：

```js
function School(schoolName) { 
    this.schoolName = schoolName; 
} 

School.prototype.printSchoolName = function(){ 
    console.log(this.schoolName); 
} 

function Student(studentName, schoolName) { 
    this.studentName = studentName; 
    School.call(this, schoolName); 
} 

Student.prototype = new School(); 
Student.prototype.printStudentName = function() { 
    console.log(this.studentName); 
}

const s = new Student("Eden", "ABC School"); 
s.printStudentName(); 
s.printSchoolName();
```

输出如下：

```js
Eden
ABC School
```

在这里，我们使用函数对象的`call`方法调用了父`constructor`。为了继承方法，我们创建了一个父`constructor`的实例并将其赋值给子构造函数的`prototype`属性。

这并不是在构造函数中实现继承的万无一失的方法，因为这里存在许多潜在的问题。例如，如果父`constructor`执行的操作不仅仅是初始化属性，比如 DOM 操作，那么将父`constructor`的新实例赋值给子`constructor`的`prototype`属性可能会导致问题。

因此，类提供了更好的、更简单的方式来继承现有的构造函数和类。我们将在本章后面看到更多关于这一点的内容。

# 原始数据类型的构造函数

原始数据类型，如布尔值、字符串和数字，都有它们的构造函数对应物。这些对应构造函数的行为就像这些原始类型的包装器。例如，`String` 构造函数用于创建一个包含内部 `[[PrimitiveValue]]` 属性的字符串对象，该属性持有实际的原始值。

在运行时， wherever necessary，原始值会被它们的 `constructor` 对应物所包裹，而对应对象则被当作原始值来处理，这样代码才能按预期工作。考虑以下示例代码来理解它是如何工作的：

```js
const s1 = "String"; 
const s2 = new String("String"); 
console.log(typeof s1); 
console.log(typeof s2); 
console.log(s1 == s2); 
console.log(s1.length);
```

输出如下：

```js
string
object
true
6
```

在这里，`s1` 是一个原始类型，而 `s2` 是一个对象，尽管对它们应用 `==` 操作符会得到一个真值。`s1` 是一个原始类型，但我们仍然能够访问长度属性，尽管原始类型不应该有任何属性。

所有这些都是在运行时将之前的代码转换为以下内容发生的：

```js
const s1 = "String"; 
const s2 = new String("String");

console.log(typeof s1); 
console.log(typeof s2); 
console.log(s1 == s2.valueOf()); 
console.log((new String(s1)).length);
```

在这里，我们可以看到原始值是如何被包裹在其 `constructor` 对应物中的，以及对象对应物在必要时是如何被当作原始值处理的。因此，代码按预期工作。

从 ES6 开始引入的原始类型不会允许它们的对应函数作为构造函数被调用，也就是说，我们不能使用它们的对象对应物显式地包裹它们。我们在学习符号时看到了这种行为。

`null` 和 `undefined` 原始类型没有对应的构造函数。

# 使用类

我们看到，JavaScript 的面向对象模型是基于构造函数和基于原型的继承。嗯，ES6 类只是现有模型的一种新语法。类并没有为 JavaScript 引入一个新的面向对象模型。

ES6 类旨在提供一种更简单、更清晰的语法来处理构造函数和继承。

实际上，类是函数。类只是创建用作构造函数的函数的一种新语法。使用类创建不作为构造函数使用的函数没有任何意义，也不提供任何好处。

相反，这会让你的代码难以阅读，因为它变得混乱。因此，只有当你想使用类来构建对象时才使用类。让我们详细看看类。

# 定义一个类

正如定义函数有两种方式，函数声明和函数表达式一样，定义类也有两种方式：使用类声明和类表达式。

# 类声明

要使用 `class` 声明来定义一个类，你需要使用 `class` 关键字并为 `class` 提供一个名称。

这里有一个代码示例，演示如何使用 `class` 声明来定义一个类：

```js
class Student { 
    constructor(name) { 
        this.name = name; 
    } 
} 

const s1 = new Student("Eden"); 
console.log(s1.name); //Output "Eden" 
```

在这里，我们创建了一个名为 `Student` 的 `class`。然后，我们在其中定义了一个 `constructor` 方法。最后，我们创建了该类的一个新实例——一个对象，并记录了该对象的名字属性。

类的主体在花括号 `{}` 中。这是我们定义方法的地方。方法定义时不使用 `function` 关键字，方法之间不用逗号分隔。

类被视为函数；内部上，类名被视为函数名，`constructor` 方法的主体被视为函数的主体。

一个 `class` 中只能有一个 `constructor` 方法。定义多个 `constructor` 将会抛出 `SyntaxError` 异常。

默认情况下，类主体内的所有代码都在严格模式下执行。

当使用 `function` 编写时，前面的代码与以下代码相同：

```js
function Student(name) { 
    this.name = name; 
} 

const s1 = new Student("Eden"); 
console.log(s1.name); //Output "Eden"
```

要证明一个 `class` 是一个 `function`，考虑以下代码：

```js
class Student { 
    constructor(name) { 
        this.name = name; 
    } 
} 

function School(name) { 
    this.name = name; 
} 

console.log(typeof Student); 
console.log(typeof School == typeof Student);
```

输出如下：

```js
function
true
```

这里，我们可以看到 `class` 是一个 `function`。它只是创建函数的新语法。

# 类表达式

类表达式具有与类声明类似的语法。然而，使用类表达式时，您可以省略类名。两种方式下的主体和行为都保持不变。

下面是一个代码示例，演示如何使用类表达式定义一个 `class`：

```js
const Student = class { 
    constructor(name) { 
        this.name = name; 
    } 
} 

const s1 = new Student("Eden"); 
console.log(s1.name); //Output "Eden"
```

这里，我们将 `class` 的引用存储在一个变量中，并使用它来构造对象。

当使用 `function` 编写时，前面的代码与以下代码相同：

```js
const Student = function(name) { 
    this.name = name; 
}; 
const s1 = new Student("Eden"); 
console.log(s1.name); //Output "Eden"
```

# 原型方法

类主体中的所有方法都被添加到类的 `prototype` 属性中。`prototype` 属性是使用 `class` 创建的对象的原型。

下面是一个示例，展示如何向 `class` 的 `prototype` 属性添加方法：

```js
class Person { 
    constructor(name, age) { 
        this.name = name; 
        this.age = age; 
    } 
    printProfile() { 
        console.log("Name is: " + this.name + " and Age is: " + this.age); 
    } 
} 

const p = new Person("Eden", 12);
p.printProfile(); 
console.log("printProfile" in p.__proto__); 
console.log("printProfile" in Person.prototype);
```

输出如下：

```js
Name is: Eden and Age is: 12
true
true
```

这里，我们可以看到 `printProfile` 方法被添加到了 `class` 的 `prototype` 属性中。

当使用 `function` 编写时，前面的代码与以下代码相同：

```js
function Person(name, age) { 
    this.name = name; 
    this.age = age; 
} 

Person.prototype.printProfile = function() { 
    console.log("Name is: " + this.name + " and Age is: " + this.age); 
} 

const p = new Person("Eden", 12);
p.printProfile(); 
console.log("printProfile" in p.__proto__); 
console.log("printProfile" in Person.prototype);
```

输出如下：

```js
Name is: Eden and Age is: 12
true
true
```

# 获取器和设置器

之前，为了向对象添加访问器属性，我们必须使用 `Object.defineProperty()` 方法。从 ES6 开始，有 `get` 和 `set` 前缀用于方法。这些方法可以被添加到对象字面量和类中，以定义访问器属性的获取和设置属性。

当在类主体中使用 `get` 和 `set` 方法时，它们会被添加到类的 `prototype` 属性中。

下面是一个示例，演示如何在 `class` 中定义 `get` 和 `set` 方法：

```js
class Person { 
    constructor(name) { 
        this._name_ = name; 
    } 
    get name() { 
        return this._name_; 
    } 
    set name(name) {
        this.someOtherCustomProp = true;
        this._name_ = name; 
    } 
} 

const p = new Person("Eden"); 
console.log(p.name); // Outputs: "Eden"
p.name = "John"; 
console.log(p.name); // Outputs: "John"
console.log(p.someOtherCustomProp); // Outputs: "true"
```

在这里，我们创建了一个访问器属性来封装 `_name_` 属性。我们还记录了一些其他信息来证明 `name` 是一个添加到 `class` 的 `prototype` 属性的访问器属性。

# `generator` 方法

要将一个对象的简洁方法视为 `generator` 方法，或者将一个类的方法视为 `generator` 方法，我们只需在它前面加上 `*` 字符。

类的 `generator` 方法被添加到类的 `prototype` 属性中。

下面是一个示例，演示如何在 `class` 中定义 `generator` 方法：

```js
class myClass {  
    * generator_function()  {  
        yield 1;  
        yield 2;  
        yield 3;  
        yield 4;  
        yield 5;  
    } 
} 

const obj = new myClass(); 
let generator = obj.generator_function(); 
console.log(generator.next().value); 
console.log(generator.next().value); 
console.log(generator.next().value); 
console.log(generator.next().value); 
console.log(generator.next().value); 
console.log(generator.next().done); 
console.log("generator_function" in myClass.prototype);
```

输出如下：

```js
 1
 2
 3
 4
 5
 true
 true
```

# 静态方法

使用 `static` 前缀添加到 `class` 体内的方法称为 `static` 方法。`static` 方法是类的自身方法；也就是说，它们是添加到类本身而不是类的 `prototype` 属性。例如，`String.fromCharCode()` 方法是字符串构造函数的 `static` 方法，即 `fromCharCode` 是 `String` 函数本身的属性。

`static` 方法通常用于创建应用程序的实用函数。

下面是一个示例，演示如何在 `class` 中定义和使用 `static` 方法：

```js
class Student {  
    constructor(name)  {  
        this.name = name;  
    }  
    static findName(student)  {  
        return student.name;  
    } 
} 

const s = new Student("Eden"); 
const name = Student.findName(s); 
console.log(name); //Output "Eden"
```

# 在类中实现继承

在本章的早期部分，我们看到了在函数中实现继承层次结构的难度。因此，ES6 通过引入 `extends` 子句和 `class` 的 `super` 关键字来简化这一点。

通过使用 `extends` 子句，一个 `class` 可以从另一个 `constructor`（可能使用类定义，也可能不是）继承静态和非静态属性。

`super` 关键字有两种用法：

+   它用于类 `constructor` 方法中调用父 `constructor`

+   当在 `class` 的方法中使用时，它引用父 `constructor` 的静态和非静态方法

下面是一个示例，演示如何使用 `extends` 子句和 `super` 关键字在构造函数中实现继承层次结构：

```js
function A(a) { 
    this.a = a; 
} 

A.prototype.printA = function(){ 
    console.log(this.a); 
}

class B extends A {  
    constructor(a, b)  {  
        super(a);  
        this.b = b;  
    }  

    printB()  {  
        console.log(this.b);  
    }  

    static sayHello()  {  
        console.log("Hello");  
    } 
} 

class C extends B {  
    constructor(a, b, c)  {  
        super(a, b);  
        this.c = c;  
    }  

    printC()  {  
        console.log(this.c);  
    }  

    printAll()  {  
        this.printC();  
        super.printB();  
        super.printA();  
    } 
} 

const obj = new C(1, 2, 3); 
obj.printAll(); 
C.sayHello();
```

输出如下：

```js
 3
 2
 1
 Hello
```

这里，`A` 是一个函数构造函数；`B` 是继承 `A` 的 `class`；`C` 是继承 `B` 的 `class`；由于 `B` 继承了 `A`，因此 `C` 也继承了 `A`。

由于类可以继承函数构造函数，我们也可以通过类继承预构建的函数构造函数，例如字符串和数组，以及使用类而不是我们以前使用的替代性笨拙方法来创建自定义函数构造函数。

之前的示例还展示了如何以及在哪里使用 `super` 关键字。记住，在 `constructor` 方法内部，在使用 `this` 关键字之前需要使用 `super` 关键字。否则，会抛出异常。

如果子类没有 `constructor` 方法，则默认行为将调用父类的 `constructor` 方法。

# 计算方法名称

你也可以在运行时决定 `class` 的静态和非静态方法以及对象字面量的简洁方法名称；也就是说，你可以通过表达式定义方法名称。下面是一个示例来演示这一点：

```js
class myClass { 
    static ["my" + "Method"]() { 
        console.log("Hello"); 
    } 
} 

myClass["my" + "Method"](); //Output "Hello"
```

计算属性名称还允许你使用符号作为方法的键。下面是一个示例来演示这一点：

```js
var s = Symbol("Sample"); 

class myClass { 
    static [s]() { 
        console.log("Hello"); 
    } 
} 

myClass[s](); //Output "Hello"
```

# 属性的属性特征

当使用 `class` 时，静态和非静态属性的属性与使用函数声明的属性不同：

+   `static` 方法是可写的和可配置的，但不可枚举

+   `class` 的 `prototype` 属性和 `prototype.constructor` 属性是不可写的、不可枚举的或不可配置的

+   `prototype` 属性的属性是可写和可配置的，但不是可枚举的

# 类不会被提升！

你可以在定义函数之前调用它；也就是说，可以在函数定义之前调用函数调用。但是，你无法在定义之前使用 `class`。在类中尝试这样做将抛出 `ReferenceError` 异常。

以下是一个示例来演示这一点：

```js
myFunc(); // fine
function myFunc(){} 
var obj = new myClass(); // throws error
class myClass {}
```

# 覆盖构造函数方法的返回结果

默认情况下，`constructor` 方法如果没有返回语句，则返回新实例。如果有返回语句，则返回语句中的任何值。如果你来自像 C++ 这样的语言，这可能会显得有些奇怪，因为在那里通常不能从 `constructor` 返回任何值。

以下是一个示例来演示这一点：

```js
class myClass { 
    constructor() { 
        return Object.create(null); 
    } 
} 

console.log(new myClass() instanceof myClass); //Output "false"
```

# Symbol.species 静态访问器属性

`@@species` 静态访问器属性可选地添加到子 `constructor` 中，以便通知父 `constructor` 的方法，如果父构造函数的方法返回新的实例，则构造函数应该使用什么。如果子 `constructor` 上未定义 `@@species` 静态访问器属性，则父 `constructor` 的方法可以使用默认的 `constructor`。

考虑以下示例以了解 `@@species` 的使用——数组对象的 `map()` 方法返回一个新的数组实例。如果我们调用继承自 `Array` 对象的对象的 `map()` 方法，那么 `map()` 方法将返回子 `constructor` 的新实例而不是 `Array` 构造函数，这通常不是我们想要的。提供此类函数信号方式的 `@@species` 属性使用不同的 `constructor` 而不是默认的 `constructor`。

以下是一个代码示例来演示如何使用 `@@species` 静态访问器属性：

```js
class myCustomArray1 extends Array {  
    static get [Symbol.species]()  {  
        return Array;  
    } 
} 

class myCustomArray2 extends Array{} 

var arr1 = new myCustomArray1(0, 1, 2, 3, 4); 
var arr2 = new myCustomArray2(0, 1, 2, 3, 4); 

console.log(arr1 instanceof myCustomArray1); // Outputs "true"
console.log(arr2 instanceof myCustomArray2); // Outputs "true"
arr1 = arr1.map(value => value + 1); 
arr2 = arr2.map(value => value + 1); 

console.log(arr1 instanceof myCustomArray1);  // Outputs "false"
console.log(arr2 instanceof myCustomArray2);  // Outputs "true"
console.log(arr1 instanceof Array); // Outputs "true"
console.log(arr2 instanceof Array); // Outputs "true"
```

如果你在创建 JavaScript 库，建议你的库中的构造函数方法在返回新实例时始终查找 `@@species` 属性。以下是一个示例来演示这一点：

```js
//Assume myArray1 is part of library 
class myArray1 { 
//default @@species. Child class will inherit this property 
    static get [Symbol.species]() { 
        //default constructor 
        return this; 
    } 
    mapping() { 
        return new this.constructor[Symbol.species](); 
    } 
} 

class myArray2 extends myArray1 { 
    static get [Symbol.species]() { 
        return myArray1; 
    } 
}

let arr = new myArray2(); 
console.log(arr instanceof myArray2); //Output "true" 
arr = arr.mapping(); 
console.log(arr instanceof myArray1); //Output "true"
```

如果你不想在父构造函数中定义默认的 `@@species` 属性，则可以使用 `if…else` 条件语句检查 `@@species` 属性是否已定义，但首选的模式是之前的模式。内置的 `map()` 方法也使用之前的模式。

从 ES6 开始，JavaScript 构造函数的所有内置方法在返回新实例时都会查找 `@@species` 属性。例如，`Array`、`Map`、`ArrayBuffer`、`Promise` 和其他此类构造函数的方法在返回新实例时会查找 `@@species` 属性。

# `new.target` 隐式参数

`new.target` 的默认值是 `undefined`，但当一个函数作为构造函数被调用时，`new.target` 参数的值取决于以下条件：

+   如果使用 `new` 运算符调用构造函数，则 `new.target` 指向此构造函数

+   如果通过 super 关键字调用构造函数，那么其中的 new.target 的值与被调用 super 的构造函数的 new.target 的值相同。

在箭头函数内部，new.target 的值与周围非箭头函数的`new.target`的值相同。

下面是演示此功能的示例代码：

```js
function myConstructor() { 
    console.log(new.target.name); 
} 

class myClass extends myConstructor { 
    constructor() { 
        super(); 
    } 
} 

const obj1 = new myClass(); 
const obj2 = new myConstructor();
```

输出如下：

```js
myClass
myConstructor
```

# 在对象字面量中使用 super

super 关键字也可以在对象字面量的简洁方法中使用。对象字面量简洁方法中的 super 关键字具有与该对象字面量定义的对象的`[[prototype]]`属性相同的值。

在对象字面量中，super 用于通过子对象访问重写的属性。

这里有一个示例来展示如何在对象字面量中使用 super：

```js
const obj1 = { 
    print() { 
        console.log("Hello"); 
    } 
} 

const obj2 = { 
    print() {
        super.print(); 
    } 
} 

Object.setPrototypeOf(obj2, obj1); 
obj2.print(); //Output "Hello"
```

ES.next 提案包括通过使用 hash（#）符号在类中添加对真正私有属性的支持。类内部的#myProp 将是该类的私有属性。

# 概述

在本章中，我们首先使用传统函数方法学习了面向对象编程的基础。然后，我们转向类，学习了它们如何使我们更容易阅读和编写面向对象的 JavaScript 代码。我们还了解了一些其他特性，如`new.target`和存取器方法。现在，让我们继续前进到网络，一个我们可以实现所学内容的地方！
