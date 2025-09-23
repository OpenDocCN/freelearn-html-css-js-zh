# 第七章. 带你走进类

ES6 引入了类，它提供了创建构造函数和处理继承的更简单、更清晰的语法。尽管 JavaScript 是一种面向对象的编程语言，但它从未有过类的概念。来自其他编程语言背景的程序员往往由于缺乏类而难以理解 JavaScript 的面向对象模型和继承。在本章中，我们将使用 ES6 类来学习面向对象的 JavaScript：

+   JavaScript 数据类型

+   以传统方式创建对象

+   原始类型构造函数

+   ES6 中的类是什么

+   使用类创建对象

+   类中的继承

+   类的特性

# 理解面向对象的 JavaScript

在我们继续学习 ES6 类之前，让我们回顾一下 JavaScript 数据类型、构造函数和继承的知识。在学习类时，我们将比较构造函数和基于原型的继承的语法与类的语法。因此，对这些主题有良好的掌握是非常重要的。

## JavaScript 数据类型

JavaScript 变量持有（或存储）数据（或值）。变量所持有的数据类型称为 **数据类型**。在 JavaScript 中，有七种不同的数据类型：**number**、**string**、**Boolean**、**null**、**undefined**、**symbol** 和 **object**。

当涉及到持有对象时，变量持有对象引用（即内存地址）而不是对象本身。

除了对象之外的所有数据类型都称为 **原始数据类型**。

### 注意

数组和函数实际上是 JavaScript 对象。

## 创建对象

在 JavaScript 中创建对象有两种方式，即使用对象字面量或使用构造函数。当我们需要创建固定对象时使用对象字面量，而当我们在运行时动态创建对象时使用构造函数。

让我们考虑一个可能需要使用构造函数而不是对象字面量的情况。以下是一个代码示例：

```js
var student = {
  name: "Eden",
  printName: function(){
    console.log(this.name);
  }
}

student.printName(); //Output "Eden"
```

在这里，我们使用对象字面量创建了 `student` 对象，即 `{}` 符号。当你只想创建单个 `student` 对象时，这很有效。

但是，当你想要创建多个 `student` 对象时，问题就出现了。显然，你不想多次编写之前的代码来创建多个 `student` 对象。这就是构造函数发挥作用的地方。

当使用 `new` 关键字调用时，函数表现得像一个构造函数。构造函数创建并返回一个对象。在函数内部，当作为构造函数调用时，`this` 关键字指向新的对象实例，一旦构造函数执行完成，新对象将自动返回。考虑以下示例：

```js
function Student(name)
{
  this.name = name;
}

Student.prototype.printName = function(){
  console.log(this.name);
}

var student1 = new Student("Eden");
var student2 = new Student("John");

student1.printName(); //Output "Eden"
student2.printName(); //Output "John"
```

在这里，为了创建多个学生对象，我们多次调用了构造函数，而不是使用对象字面量创建多个学生对象。

要向构造函数的实例添加方法，我们没有使用`this`关键字，而是使用了构造函数的`prototype`属性。我们将在下一节中了解更多关于为什么这样做以及`prototype`属性是什么。

实际上，每个对象都必须属于一个构造函数。每个对象都有一个名为`constructor`的继承属性，指向该对象的构造函数。当我们使用对象字面量创建对象时，`constructor`属性指向全局的`Object`构造函数。考虑以下示例来理解这种行为：

```js
var student = {}

console.log(student.constructor == Object); //Output "true"
```

## 理解继承

每个 JavaScript 对象都有一个内部名为`[[prototype]]`的属性，它指向另一个称为其原型的对象。这个原型对象有自己的原型，以此类推，直到找到一个其原型为`null`的对象。`null`没有原型，它作为原型链中的最后一个链接。

当尝试访问一个对象的属性时，如果该属性在对象中找不到，那么该属性将在对象的原型中搜索。如果仍然找不到，那么将在原型对象的原型中搜索。这个过程会一直持续到原型链中遇到`null`。这就是 JavaScript 中继承的工作方式。

由于 JavaScript 对象只能有一个原型，JavaScript 只支持单继承。

在使用对象字面量创建对象时，我们可以使用特殊的`__proto__`属性或`Object.setPrototypeOf()`方法来指定对象的原型。JavaScript 还提供了一个`Object.create()`方法，我们可以用它来创建一个新的对象，其`__proto__`被指定为原型。由于浏览器不支持`__proto__`，而`Object.setPrototypeOf()`方法看起来有些奇怪。以下是一个代码示例，展示了在创建对象时使用对象字面量设置对象原型的不同方法：

```js
var object1 = {
  name: "Eden",
  __proto__: {age: 24}
}

var object2 = {name: "Eden"}
Object.setPrototypeOf(object2, {age: 24});

var object3 = Object.create({age: 24}, {name: {value: "Eden"}});

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

在这里，`{age:24}`对象被称为**基对象**、**超对象**或**父对象**，因为它正在继承。而`{name:"Eden"}`对象被称为**派生对象**、**子对象**或**子对象**，因为它继承了另一个对象。

如果你在使用对象字面量创建对象时没有为其指定原型，那么原型将指向`Object.prototype`属性。由于`Object.prototype`的原型是`null`，因此导致原型链的结束。以下是一个示例来演示这一点：

```js
var obj = {
  name: "Eden"
}

console.log(obj.__proto__ == Object.prototype); //Output "true"
```

在使用构造函数创建对象时，新对象的原型始终指向函数对象的`prototype`属性。默认情况下，`prototype`属性是一个具有一个名为`constructor`的属性的对象。`constructor`属性指向函数本身。考虑以下示例来理解这个模型：

```js
function Student()
{
  this.name = "Eden";
}

var obj = new Student();

console.log(obj.__proto__.constructor == Student); //Output "true"
console.log(obj.__proto__ == Student.prototype); //Output "true"
```

要向构造函数的实例添加新方法，我们应该将它们添加到构造函数的 `prototype` 属性中，就像我们之前做的那样。我们不应该在构造函数体中使用 `this` 关键字来添加方法，因为构造函数的每个实例都将有一个方法副本，这并不非常内存高效。通过将方法附加到构造函数的 `prototype` 属性，每个函数只有一个副本，所有实例共享。为了理解这一点，考虑以下示例：

```js
function Student(name)
{
    this.name = name;
}

Student.prototype.printName = function(){
    console.log(this.name);
}

var s1 = new Student("Eden");
var s2 = new Student("John");

function School(name)
{
  this.name = name;
  this.printName = function(){
    console.log(this.name);
  }
}

var s3 = new School("ABC");
var s4 = new School("XYZ");

console.log(s1.printName == s2.printName);
console.log(s3.printName == s4.printName);
```

输出如下：

```js
true
false
```

在这里，`s1` 和 `s2` 共享相同的 `printName` 函数，这减少了内存的使用，而 `s3` 和 `s4` 包含两个不同的名为 `printName` 的函数，这使程序使用更多的内存。这是不必要的，因为这两个函数执行相同的事情。因此，我们将实例的方法添加到构造函数的 `prototype` 属性中。

在构造函数中实现继承层次结构并不像我们对对象字面量所做的那样直接。因为子构造函数需要调用父构造函数以执行父构造函数的初始化逻辑，并且我们需要将父构造函数的 `prototype` 属性的方法添加到子构造函数的 `prototype` 属性中，以便我们可以使用它们与子构造函数的对象一起使用。没有预定义的方式来完成所有这些。开发人员和 JavaScript 库有自己的实现方式。我将向您展示最常见的方法。

这里有一个示例，演示如何在创建对象时使用构造函数实现继承：

```js
function School(schoolName)
{
  this.schoolName = schoolName;
}
School.prototype.printSchoolName = function(){
  console.log(this.schoolName);
}

function Student(studentName, schoolName)
{
  this.studentName = studentName;

  School.call(this, schoolName);
}
Student.prototype = new School();
Student.prototype.printStudentName = function(){
  console.log(this.studentName);
}

var s = new Student("Eden", "ABC School");
s.printStudentName();
s.printSchoolName();
```

输出如下：

```js
Eden
ABC School
```

在这里，我们使用函数对象的 `call` 方法调用了父构造函数。为了继承方法，我们创建了一个父构造函数的实例，并将其分配给子构造函数的 `prototype` 属性。

这不是在构造函数中实现继承的万无一失的方法，因为存在许多潜在问题。例如——如果父构造函数执行的操作不仅仅是初始化属性，例如 DOM 操作，那么将父构造函数的新实例分配给子构造函数的 `prototype` 属性可能会引起问题。

因此，ES6 类提供了更好的、更简单的方式来继承现有的构造函数和类。我们将在本章后面进一步了解这一点。

## 原始数据类型的构造函数

原始数据类型，如布尔值、字符串和数字，都有它们的构造函数对应物。这些对应构造函数的行为类似于这些原始类型的包装器。例如，`String` 构造函数用于创建包含内部 `[[PrimitiveValue]]` 属性的字符串对象，该属性持有实际的原始值。

在运行时，在必要时，原始值会被它们的构造函数对应物包装，同时对应对象也会被当作原始值处理，这样代码才能按预期工作。考虑以下示例代码来理解它是如何工作的：

```js
var s1 = "String";
var s2 = new String("String");

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

在这里，`s1`是一个原始类型，而`s2`是一个对象，尽管对它们应用`==`运算符会得到`true`的结果。`s1`是一个原始类型，但我们仍然能够访问长度属性，然而原始类型不应该有任何属性。

所有这些都是在运行时将之前的代码转换成以下形式发生的：

```js
var s1 = "String";
var s2 = new String("String");

console.log(typeof s1);
console.log(typeof s2);

console.log(s1 == s2.valueOf());
console.log((new String(s1)).length);
```

这里，我们可以看到原始值是如何被其构造函数对应物包装的，以及当需要时对象对应物是如何被当作原始值处理的。因此，代码按预期工作。

从 ES6 开始引入的原始类型不会允许它们的对应函数作为构造函数被调用，也就是说，我们不能显式地使用它们的对象对应物来包装它们。我们在学习符号时看到了这种行为。

原始类型`null`和`undefined`没有对应的构造函数。

# 使用类

我们看到 JavaScript 的面向对象模型基于构造函数和基于原型的继承。嗯，ES6 类只是现有模型的新语法。类并没有为 JavaScript 引入新的面向对象模型。

ES6 类的目的是为了提供一个更简单、更清晰的语法来处理构造函数和继承。

实际上，类是函数。类只是创建用作构造函数的函数的新语法。使用类创建不作为构造函数使用的函数没有任何意义，也不会带来任何好处。相反，它会使得你的代码难以阅读，因为它变得混乱。因此，只有当你想用它来构造对象时才使用类。让我们详细看看类。

## 定义类

正如定义函数有两种方式，函数声明和函数表达式，定义类也有两种方式：使用类声明和类表达式。

### 类声明

要使用类声明来定义一个类，你需要使用`class`关键字和类的名称。

这里有一个代码示例，演示了如何使用类声明来定义一个类：

```js
class Student
{
  constructor(name)
  {
    this.name = name;
  }
}

var s1 = new Student("Eden");
console.log(s1.name); //Output "Eden"
```

在这里，我们创建了一个名为`Student`的类。然后，我们在其中定义了一个`constructor`方法。最后，我们创建了该类的一个新实例——一个对象，并记录了该对象的`name`属性。

类的主体位于花括号内，即`{}`。这是我们定义方法的地方。方法定义时不使用`function`关键字，并且在方法之间不使用逗号。

类被当作函数处理，并且内部类名被当作函数名，`constructor`方法的主体被当作函数的主体。

在一个类中只能有一个`构造函数`方法。定义多个构造函数将抛出`SyntaxError`异常。

默认情况下，类体内的所有代码都在`strict`模式下执行。

当使用函数编写时，前面的代码与这段代码相同：

```js
function Student(name)
{
  this.name = name;
}

var s1 = new Student("Eden");
console.log(s1.name); //Output "Eden"
```

要证明类是一个函数，请考虑以下代码：

```js
class Student
{
  constructor(name)
  {
    this.name = name;
  }
}

function School(name)
{
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

这里，我们可以看到类是一个函数。它只是创建函数的新语法。

### 类表达式

类表达式与类声明的语法类似。然而，使用类表达式时，您可以省略类名。类体和行为在这两种方式中保持相同。

下面是一个使用类表达式定义类的代码示例：

```js
var Student = class {
  constructor(name)
  {
    this.name = name;
  }
}

var s1 = new Student("Eden");
console.log(s1.name); //Output "Eden"
```

这里，我们将类的引用存储在一个变量中，并使用它来构造对象。

当使用函数编写时，前面的代码与这段代码相同：

```js
var Student = function(name) {
  this.name = name;
}

var s1 = new Student("Eden");
console.log(s1.name); //Output "Eden"
```

## 原型方法

类体中的所有方法都被添加到类的`prototype`属性中。`prototype`属性是使用类创建的对象的原型。

下面是一个示例，展示如何向类的`prototype`属性中添加方法：

```js
class Person
{
  constructor(name, age)
  {
    this.name = name;
    this.age = age;
  }

  printProfile()
  {
    console.log("Name is: " + this.name + " and Age is: " + this.age);
  }
}

var p = new Person("Eden", 12)
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

这里，我们可以看到`printProfile`方法被添加到了类的`prototype`属性中。

当使用函数编写时，前面的代码与这段代码相同：

```js
function Person(name, age)
{
  this.name = name;
  this.age = age;
}

Person.prototype.printProfile = function()
{
  console.log("Name is: " + this.name + " and Age is: " + this.age);
}

var p = new Person("Eden", 12)
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

### 获取和设置方法

在 ES5 中，为了向对象添加访问器属性，我们必须使用`Object.defineProperty()`方法。ES6 引入了方法的`get`和`set`前缀。这些方法可以添加到对象字面量和类中，以定义访问器属性的`get`和`set`属性。

当在类体中使用`get`和`set`方法时，它们被添加到类的`prototype`属性中。

下面是一个示例，演示如何在类中定义`get`和`set`方法：

```js
class Person
{
  constructor(name)
  {
    this._name_ = name;
  }

  get name(){
    return this._name_;
  }

  set name(name){
    this._name_ = name;
  }
}

var p = new Person("Eden");
console.log(p.name);
p.name = "John";
console.log(p.name);

console.log("name" in p.__proto__);
console.log("name" in Person.prototype);
console.log(Object.getOwnPropertyDescriptor(p.__proto__, "name").set);
console.log(Object.getOwnPropertyDescriptor(Person.prototype, "name").get);
console.log(Object.getOwnPropertyDescriptor(p, "_name_").value);
```

输出如下：

```js
Eden
John
true
true
function name(name) { this._name_ = name; }
function name() { return this._name_; }
John
```

这里，我们创建了一个访问器属性来封装`_name_`属性。我们还记录了一些其他信息来证明`name`是一个访问器属性，它被添加到了类的`prototype`属性中。

### 生成器方法

要将对象字面量的简洁方法视为生成器方法，或将类的方法视为生成器方法，我们只需在它前面加上`*`字符。

类的生成器方法被添加到类的`prototype`属性中。

下面是一个示例，演示如何在类中定义一个生成器方法：

```js
class myClass
{
  * generator_function()
  {
    yield 1;
    yield 2;
    yield 3;
    yield 4;
    yield 5;
  }

}

var obj = new myClass();

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

## 静态方法

使用`static`前缀添加到类体中的方法被称为静态方法。静态方法是类的自有方法，也就是说，它们不是添加到类的`prototype`属性中，而是直接添加到类本身。例如，`String.fromCharCode()`方法是`String`构造函数的静态方法，即`fromCharCode`是`String`函数本身的自有属性。

静态方法通常用于为应用程序创建实用函数。

这里有一个示例来展示如何在类中定义和使用静态方法：

```js
class Student
{
  constructor(name)
  {
    this.name = name;
  }

  static findName(student)
  {
    return student.name;
  }
}

var s = new Student("Eden");
var name = Student.findName(s);

console.log(name); //Output "Eden"
```

在这里，`findName`是`Student`类的静态方法。

之前的代码与以下使用函数编写的代码相同：

```js
function Student(name)
{
    this.name = name;
}

Student.findName = function(student){
  return student.name;
}

var s = new Student("Eden");
var name = Student.findName(s);

console.log(name); //Output "Eden"
```

## 在类中实现继承

在本章的早期部分，我们看到了在函数中实现继承层次结构的难度。因此，ES6 通过引入`extends`子句和类的`super`关键字来简化这一过程。

通过使用`extends`子句，一个类可以继承另一个构造函数的静态和非静态属性（这些属性可能或可能不是使用类定义的）。

`super`关键字有两种用法：

+   它在类`constructor`方法中用于调用父构造函数

+   当在类的内部方法中使用时，它引用父构造函数的静态和非静态方法

以下是一个示例，展示如何使用`extends`子句和`super`关键字在构造函数中实现继承层次结构：

```js
function A(a)
{
  this.a = a;
}

A.prototype.printA = function(){
  console.log(this.a);
}

class B extends A
{
  constructor(a, b)
  {
    super(a);
    this.b = b;
  }

  printB()
  {
    console.log(this.b);
  }

  static sayHello()
  {
    console.log("Hello");
  }
}

class C extends B
{
  constructor(a, b, c)
  {
    super(a, b);
    this.c = c;
  }

  printC()
  {
    console.log(this.c);
  }

  printAll()
  {
    this.printC();
    super.printB();
    super.printA();
  }
}

var obj = new C(1, 2, 3);
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

在这里，`A`是一个函数构造函数；`B`是一个继承自`A`的类；`C`是一个继承自`B`的类；由于`B`继承了`A`，因此`C`也继承了`A`。

由于类可以继承函数构造函数，我们也可以继承预构建的函数构造函数，例如`String`和`Array`，以及使用类而不是我们以前使用的其他*笨拙*方法来自定义函数构造函数。

之前的示例也展示了如何以及在哪里使用`super`关键字。记住，在`constructor`方法内部，在使用`this`关键字之前需要使用`super`。否则，会抛出异常。

### 注意

如果子类没有`constructor`方法，则默认行为将调用父类的`constructor`方法。

## 计算方法名

你还可以在运行时决定类的静态和非静态方法以及对象字面量的简洁方法的名字，也就是说，你可以通过表达式定义方法的名字。以下是一个示例来展示这一点：

```js
class myClass
{
  static ["my" + "Method"](){
    console.log("Hello");
  }
}

myClass["my" + "Method"](); //Output "Hello"
```

计算属性名还允许你使用符号作为方法的键。以下是一个示例来展示这一点：

```js
var s = Symbol("Sample");

class myClass
{
  static [s]()
  {
    console.log("Hello");
  }
}

myClass[s](); //Output "Hello"
```

## 属性的属性

当使用类时，构造函数的静态和非静态属性的属性与使用函数声明时不同：

+   静态方法是可写和可配置的，但不是可枚举的

+   类的 `prototype` 属性和 `prototype.constructor` 属性是不可写、不可枚举或不可配置的

+   `prototype` 属性的属性是可写和可配置的，但不是可枚举的

## 类不会被提升！

你可以在定义函数之前调用它，也就是说，函数调用可以在函数定义之上进行。但在类的情况下，你不能在定义之前使用类。在类中尝试这样做将会抛出 `ReferenceError` 异常。

以下是一个演示此点的示例：

```js
myFunc();
function myFunc(){}

var obj = new myClass(); //throws ReferenceError exception
class myClass{}
```

## 覆盖构造函数方法的返回结果

默认情况下，`constructor` 方法如果没有 `return` 语句，则返回新实例。如果有 `return` 语句，则返回 `return` 语句中的值。

以下是一个演示此点的示例：

```js
class myClass
{
  constructor()
  {
    return Object.create(null);
  }
}

console.log(new myClass() instanceof myClass); //Output "false"
```

## “Symbol.species” 静态访问器属性

`@@species` 静态访问器属性可选地添加到子构造函数中，以便向父构造函数的方法发出信号，告知构造函数在父构造函数的方法返回新实例时应使用什么。如果子构造函数上没有定义 `@@species` 静态访问器属性，则父构造函数的方法可以使用默认构造函数。

考虑以下示例以了解 `@@species` 的用法——数组对象的 `map()` 方法返回一个新的 `Array` 实例。如果我们调用继承数组对象的对象的 `map()` 方法，那么 `map()` 方法将返回子构造函数的新实例而不是 `Array` 构造函数，这通常不是我们想要的。因此，ES6 引入了 `@@species` 属性，它提供了一种向此类函数发出信号的方式，使其使用不同的构造函数而不是默认构造函数。

以下是一个代码示例，演示如何使用 `@@species` 静态访问器属性：

```js
class myCustomArray1 extends Array
{
   static get [Symbol.species]()
   {
     return Array;
   }
}

class myCustomArray2 extends Array{}

var arr1 = new myCustomArray1(0, 1, 2, 3, 4);
var arr2 = new myCustomArray2(0, 1, 2, 3, 4);

console.log(arr1 instanceof myCustomArray1);
console.log(arr2 instanceof myCustomArray2);

arr1 = arr1.map(function(value){ return value + 1; })
arr2 = arr2.map(function(value){ return value + 1; })

console.log(arr1 instanceof myCustomArray1);
console.log(arr2 instanceof myCustomArray2);

console.log(arr1 instanceof Array);
console.log(arr2 instanceof Array);
```

输出如下：

```js
true
true
false
true
true
false
```

如果你正在创建一个 JavaScript 库，建议你的库中的构造函数方法在返回新实例时始终查找 `@@species` 属性。以下是一个演示此点的示例：

```js
//Assume myArray1 is part of library
class myArray1
{

  //default @@species. Child class will inherit this property
  static get [Symbol.species]()
  {
    //default constructor
    return this;
  }

  mapping()
  {
    return new this.constructor[Symbol.species]();
  }
}

class myArray2 extends myArray1
{
  static get [Symbol.species]()
  {
    return myArray1;
  }
}

var arr = new myArray2();

console.log(arr instanceof myArray2); //Output "true"

arr = arr.mapping();

console.log(arr instanceof myArray1); //Output "true"
```

如果你不希望在父构造函数中定义默认的 `@@species` 属性，那么你可以使用 `if…else` 条件语句来检查 `@@species` 属性是否已定义。但首选的做法是使用之前的模式。内置的 `map()` 方法也使用之前的模式。

ES6 中 JavaScript 构造函数的所有内置方法在返回新实例时都会查找 `@@species` 属性。例如，`Array`、`Map`、`ArrayBuffer`、`Promise` 等构造函数的方法在返回新实例时会查找 `@@species` 属性。

## “new.target” 隐式参数

ES6 为所有函数添加了一个名为 `new.target` 的参数。参数名中的点号是参数名的一部分。

`new.target` 的默认值是 `undefined`。但当函数被作为构造函数调用时，`new.target` 参数的值取决于以下条件：

+   如果使用 `new` 操作符调用构造函数，则 `new.target` 指向此构造函数

+   如果通过 `super` 关键字调用构造函数，则其中的 `new.target` 的值与被调用 `super` 的构造函数的 `new.target` 的值相同。

在箭头函数内部，`new.target` 的值与周围的非箭头函数的 `new.target` 的值相同。

这里有一个示例代码来演示这一点：

```js
function myConstructor()
{
  console.log(new.target.name);
}

class myClass extends myConstructor
{
  constructor()
  {
    super();
  }
}

var obj1 = new myClass();
var obj2 = new myConstructor();
```

输出如下：

```js
myClass
myConstructor
```

# 在对象字面量中使用 "super"

`super` 关键字也可以用于对象字面量的简洁方法。对象字面量简洁方法中的 `super` 关键字，其值与由对象字面量定义的对象的 `[[prototype]]` 属性相同。

在对象字面量中，`super` 用于通过子对象访问被覆盖的属性。

这里有一个示例来展示如何在对象字面量中使用 `super`：

```js
var obj1 = {
  print(){
    console.log("Hello");
  }
}

var obj2 = {
  print(){
    super.print();
  }
}

Object.setPrototypeOf(obj2, obj1);
obj2.print(); //Output "Hello"
```

# 摘要

在本章中，我们首先使用 ES5 学习了面向对象编程的基础。然后，我们跳入了 ES6 类，并学习了它如何使我们更容易阅读和编写面向对象的 JavaScript 代码。我们还学习了其他特性，例如 `new.target` 和存取器方法。

在下一章中，我们将学习如何创建和使用 ES6 模块。
