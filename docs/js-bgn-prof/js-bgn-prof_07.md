

# 第七章：类

在本章中，我们将讨论 JavaScript 类。我们已经看到了 JavaScript 对象，而类是对象创建的蓝图或模板。因此，这里讨论的许多内容不应该听起来太陌生或革命性。

类使面向对象编程成为可能，这是软件开发中最重要的设计进步之一。这种发展大大降低了应用程序的复杂性，并极大地提高了可维护性。

因此，面向对象编程和类对于计算机科学来说非常重要。然而，当我们将其应用于 JavaScript 时，情况并不一定如此。JavaScript 类与其他编程语言相比有一些特殊之处。在表面之下，类被某种特殊函数所包装。这意味着它们实际上是一种使用构造函数定义对象的替代语法。在本章中，我们将学习类是什么，以及我们如何创建和使用它们。在这个过程中，我们将涵盖以下主题：

+   面向对象编程

+   类和对象

+   类

+   继承

+   原型

    注意：练习、项目和自我检查测验的答案可以在*附录*中找到。

# 面向对象编程

在我们深入探讨类的乐趣之前，让我们简要地谈谈**面向对象编程**（**OOP**）。OOP 是一种非常重要的编程范式，其中代码以对象的形式组织，从而产生更易于维护和重用的代码。使用 OOP 可以教会你真正尝试以对象的形式思考各种主题，通过将属性捆绑在一起，使它们可以包含在一个称为类的蓝图内。这反过来又可能从父类继承属性。

例如，如果我们考虑一个动物，我们可以想出一些属性：名称、重量、高度、最大速度、颜色等等。然后如果我们考虑一种特定的鱼类，我们可以重用“动物”的所有属性，并在其中添加一些特定的鱼类属性。对狗也是如此；如果我们考虑狗，我们可以重用“动物”的所有属性，并添加一些特定的狗属性。这样我们就有了可重用的动物类代码。当我们意识到我们忘记了一个非常重要的属性，这个属性对于我们的应用程序中的许多动物都很重要时，我们只需要将其添加到动物类中。

这对于 Java、.NET 和其他经典面向对象编程方式来说非常重要。JavaScript 并不一定是围绕对象构建的。我们需要它们并会使用它们，但它们并不是我们代码的明星，换句话说。

# 类和对象

作为快速回顾，对象是一系列属性和方法集合。我们在*第三章*，*JavaScript 多值*中看到了它们。对象的属性应该有合理的名称。例如，如果我们有一个`person`对象，这个对象可以拥有名为`age`和`lastName`的属性，它们包含值。以下是一个对象的示例：

```js
let dog = { dogName: "JavaScript", 
            weight: 2.4, 
            color: "brown", 
            breed: "chihuahua"
          }; 
```

JavaScript 中的类封装了属于该类的数据和函数。如果你创建了一个类，你可以稍后使用以下语法使用该类创建对象：

```js
class ClassName {
  constructor(prop1, prop2) {
    this.prop1 = prop1;
    this.prop2 = prop2;
  }
}

let obj = new ClassName("arg1", "arg2"); 
```

这段代码定义了一个名为`ClassName`的类，声明了一个`obj`变量，并用这个对象的新实例初始化它。提供了两个参数。这些参数将由构造函数用于初始化属性。正如你所看到的，构造函数的参数和类的属性（`prop1`和`prop2`）具有相同的名称。类的属性可以通过它们前面的`this`关键字来识别。`this`关键字指的是它所属的对象，因此它是`ClassName`实例的第一个属性。

记得我们说过，类只是表面下的一些特殊函数。我们可以使用如下特殊函数来创建对象：

```js
function Dog(dogName, weight, color, breed) {  
  this.dogName = dogName; 
  this.weight = weight; 
  this.color = color; 
  this.breed = breed; 
}  
let dog = new Dog("Jacky", 30, "brown", "labrador"); 
```

狗的例子也可以使用类语法来创建。它看起来像这样：

```js
class Dog {
  constructor(dogName, weight, color, breed) {
    this.dogName = dogName;
    this.weight = weight;
    this.color = color;
    this.breed = breed;
  }
}
let dog = new Dog("JavaScript", 2.4, "brown", "chihuahua"); 
```

这将产生具有相同属性的对象。如果我们像下面这样进行日志记录，我们就能看到它：

```js
console.log(dog.dogName, "is a", dog.breed, "and weighs", dog.weight, "kg."); 
```

这将输出：

```js
JavaScript is a chihuahua and weighs 2.4 kg. 
```

在下一节中，我们将深入探讨类的所有部分。

# 类

你可能会想，如果类和简单地定义一个对象做的是完全相同的事情，我们为什么还需要类呢？答案是，类本质上是为对象创建的蓝图。这意味着如果我们有一个`dog`类，需要创建 20 只狗时，我们就不需要输入很多代码。如果我们必须创建对象，我们每次都必须指定所有属性的名称。而且很容易出错，拼错属性名。在这些情况下，类非常有用。

如前节所示，我们使用`class`关键字告诉 JavaScript 我们想要创建一个类。接下来，我们给类起一个名字。按照惯例，类名应该以大写字母开头。

让我们来看看类的所有不同元素。

## 构造函数

构造函数是一个特殊的方法，我们用它来使用我们的类蓝图初始化对象。一个类中只能有一个构造函数。这个构造函数包含在初始化类时将被设置的属性。

这里你可以看到一个`Person`类中构造函数的例子：

```js
class Person {
  constructor(firstname, lastname) {
    this.firstname = firstname;
    this.lastname = lastname;
  }
} 
```

在表面之下，JavaScript 基于这个构造函数创建了一个特殊函数。这个函数获取类名，并使用给定的属性创建一个对象。通过这个特殊函数，你可以创建类的实例（对象）。

这就是如何从`Person`类创建一个新对象的方法：

```js
let p = new Person("Maaike", "van Putten"); 
```

`new`这个词告诉 JavaScript 在`Person`类中查找特殊的构造函数并创建一个新对象。构造函数被调用并返回一个具有指定属性的 person 对象实例。这个对象被存储在`p`变量中。

如果我们在日志语句中使用我们的新`p`变量，你可以看到属性确实被设置了：

```js
console.log("Hi", p.firstname); 
```

这将输出：

```js
Hi Maaike 
```

你认为当我们创建一个没有所有属性的类时会发生什么？让我们找出答案：

```js
let p = new Person("Maaike"); 
```

许多语言会崩溃，但 JavaScript 不会。它只是将剩余的属性设置为`undefined`。你可以通过记录它来看到会发生什么：

```js
console.log("Hi", p.firstname, p.lastname); 
```

这会导致：

```js
Hi Maaike undefined 
```

你可以在`constructor`中指定默认值。你可以这样做：

```js
constructor(firstname, lastname = "Doe") {
    this.firstname = firstname;
    this.lastname = lastname;
  } 
```

这样，它就不会打印`Hi Maaike undefined`，而是`Hi Maaike Doe`。

### 练习 7.1

按以下步骤创建一个`Person`类，并打印朋友的姓名实例：

1.  为`Person`创建一个类，包括`firstname`和`lastname`的构造函数。

1.  创建一个变量，并使用你朋友的第一个和最后一个名字将新`Person`对象赋值给它。

1.  现在添加第二个变量，使用另一个朋友的第一个和最后一个名字。

1.  将两个朋友的姓名输出到控制台，问候语为`hello`。

## 方法

在类中，我们可以指定函数。这意味着我们的对象可以使用对象的自身属性开始做一些事情——例如，打印一个名字。类上的函数被称为方法。在定义这些方法时，我们不使用`function`关键字。我们直接从名称开始：

```js
class Person {
  constructor(firstname, lastname) {
    this.firstname = firstname;
    this.lastname = lastname;
  }
  greet() {
    console.log("Hi there! I'm", this.firstname);
  }
} 
```

我们可以这样在`Person`对象上调用`greet`方法：

```js
let p = new Person("Maaike", "van Putten");
p.greet(); 
```

它将输出：

```js
Hi there! I'm Maaike 
```

你可以在类上指定任意多的方法。在这个例子中，我们使用了`firstname`属性。我们这样做是通过说`this.property`。如果我们有一个`firstname`属性值不同的个人，例如`Rob`，它将打印：

```js
Hi there! I'm Rob 
```

就像函数一样，方法也可以接受参数并返回结果：

```js
class Person {
  constructor(firstname, lastname) {
    this.firstname = firstname;
    this.lastname = lastname;
  }
  greet() {
    console.log("Hi there!");
  }
  compliment(name, object) {
    return "That's a wonderful " + object + ", " + name;
  }
} 
```

`compliment`函数本身不输出任何内容，所以我们记录它

```js
let compliment = p.compliment("Harry", "hat");
console.log(compliment); 
```

输出将是：

```js
That's a wonderful hat, Harry 
```

在这种情况下，我们向我们的方法发送参数，因为你通常不会赞美你自己的属性（这是一个好句子，Maaike！）。然而，每当方法不需要外部输入而只需要对象的属性时，就不需要任何参数，方法可以使用其对象的属性。让我们做一个练习，然后继续使用类外部的属性。

### 练习 7.2

获取你朋友的完整姓名：

1.  在*练习 7.1*中使用的`Person`类中添加一个名为`fullname`的方法，当调用时返回`firstname`和`lastname`的连接值。

1.  使用两个朋友的第一个和最后一个名字为`person1`和`person2`创建值。

1.  在类中使用`fullname`方法，返回一个人的或两个人的全名。

## 属性

属性，有时也称为字段，存储类的数据。我们已经在构造函数中创建它们时看到了一种属性：

```js
class Person {
  constructor(firstname, lastname) {
    this.firstname = firstname;
    this.lastname = lastname;
  }
} 
```

在这里，`Person`类从构造函数中获取两个属性：`firstname`和`lastname`。属性可以像我们对对象所做的那样添加或删除。这些属性可以从类外部访问，就像我们在类外部通过访问实例来记录它们时看到的那样：

```js
let p = new Person("Maaike", "van Putten");
console.log("Hi", p.firstname); 
```

通常，我们不希望直接提供对属性的访问。我们希望我们的类控制属性的值，有多个原因——也许我们想在属性上执行验证以确保它具有某个值。例如，想象一下想要验证年龄不低于 18 岁。我们可以通过使从类外部直接访问属性变得不可能来实现这一点。

这是如何添加外部不可访问的属性的示例。我们用`#`符号作为前缀：

```js
class Person {
  #firstname;
  #lastname;
  constructor(firstname, lastname) {
    this.#firstname = firstname;
    this.#lastname = lastname;
  }
} 
```

目前，`firstname`和`lastname`属性不能从类外部访问。这是通过在属性前添加`#`符号来实现的。如果我们尝试这样做：

```js
let p = new Person("Maria", "Saga");
console.log(p.firstname); 
```

我们将得到：

```js
undefined 
```

如果我们想要确保只能创建以"M"开头的名字的对象，我们可以稍微修改一下构造函数：

```js
constructor(firstname, lastname) { 
    if(firstname.startsWith("M")){
      this.#firstname = firstname; 
    } else {
      this.#firstname = "M" + firstname; 
    }
    this.#lastname = lastname; 
  } 
```

现在当你尝试创建一个`firstname`值不以"M"开头的个人时，它会在前面添加一个"M"。例如，以下名字的值是`Mkay`：

```js
let p = new Person("kay", "Moon"); 
```

这是一个非常愚蠢的验证示例。在这个阶段，在构造函数之后，我们根本不能从类外部访问它。我们只能从类内部访问它。这就是获取器和设置器发挥作用的地方。

### 获取器和设置器

获取器和设置器是特殊的属性，我们可以使用它们从类中获取数据，并在类上设置数据字段。获取器和设置器是计算属性。因此，它们更像属性而不是函数。我们称它们为访问器。它们看起来有点像函数，因为它们后面有`()`，但它们不是！

这些访问器以`get`和`set`关键字开头。尽可能将字段设置为私有，并使用获取器和设置器来访问它们被认为是良好的实践。这样，属性就不能从外部设置，除非对象本身控制。这个原则被称为**封装**。类封装了数据，对象控制自己的字段。

下面是如何做到这一点的示例：

```js
class Person {
  #firstname;
  #lastname;
  constructor(firstname, lastname) {
    this.#firstname = firstname;
    this.#lastname = lastname;
  }
  get firstname() {
    return this.#firstname;
  }
  set firstname(firstname) {
    this.#firstname = firstname;
  }
  get lastname() {
    return this.#lastname;
  }
  set lastname(lastname) {
    this.#lastname = lastname;
  }
} 
```

获取器用于获取属性。因此，它不需要任何参数，只需返回属性即可。设置器则相反：它接受一个参数，将这个新值赋给属性，然后不返回任何内容。设置器可以包含更多的逻辑，例如，一些验证，正如我们下面将要看到的。获取器可以在对象外部使用，就像它是一个属性一样。属性不再可以从外部直接访问，但可以通过获取器获取值，通过设置器更新值。以下是如何在类实例外部使用它的示例：

```js
let p = new Person("Maria", "Saga");
console.log(p.firstname); 
```

这将输出：

```js
Maria 
```

我们创建了一个新的`Person`对象，名字为`Maria`，姓氏为`Saga`。输出显示了名字，这仅因为我们可以使用获取器访问器。我们也可以将值设置为其他内容，因为存在设置器。以下是更新名字的示例，这样名字就不再是`Maria`，而是`Adnane`。

```js
p.firstname = "Adnane"; 
```

在这个阶段，setter 中并没有发生任何特殊的事情。我们可以像在构造函数中那样进行类似的验证，如下所示：

```js
set firstname(firstname) { 
    if(firstname.startsWith("M")){
      this.#firstname = firstname; 
    } else {
      this.#firstname = "M" + firstname; 
    }
  } 
```

这将检查`firstname`是否以`M`开头，如果是，它将更新值为`firstname`参数的值。如果不是，它将在参数前面连接一个`M`。

请注意，我们不是像函数一样访问`firstname`。如果你在它后面放两个括号`()`，实际上你会得到一个错误，告诉你它不是一个函数。

# 继承

继承是面向对象编程中的一个关键概念。它是这样的概念：类可以有子类，这些子类继承父类的属性和方法。例如，如果你在应用程序中需要各种车辆对象，你可以在一个名为`Vehicle`的类中指定一些车辆的共享属性和方法。然后，基于这个`Vehicle`类，你可以继续创建具体的子类，例如`boat`、`car`、`bicycle`和`motorcycle`。

这可能是`Vehicle`类的一个非常简单的版本：

```js
class Vehicle {
  constructor(color, currentSpeed, maxSpeed) {
    this.color = color;
    this.currentSpeed = currentSpeed;
    this.maxSpeed = maxSpeed;
  }
  move() {
    console.log("moving at", this.currentSpeed);
  }
  accelerate(amount) {
    this.currentSpeed += amount;
  }
} 
```

在我们的`Vehicle`类中，有两种方法：`move`和`accelerate`。这可以是一个使用`extends`关键字从该类继承的`Motorcyle`类：

```js
class Motorcycle extends Vehicle {
  constructor(color, currentSpeed, maxSpeed, fuel) {
    super(color, currentSpeed, maxSpeed);
    this.fuel = fuel;
  }
  doWheelie() {
    console.log("Driving on one wheel!");
   }
} 
```

使用`extends`关键字，我们指定一个类是另一个类的子类。在这种情况下，`Motorcycle`是`Vehicle`的子类。这意味着我们将在我们的`Motorcycle`类中访问`Vehicle`的属性和方法。我们添加了一个特殊的`doWheelie()`方法。这不是可以添加到`Vehicle`类中的东西，因为这是一种特定于某些车辆的行为。

构造函数中的`super`关键字是在调用父类的构造函数，在这个例子中是`Vehicle`构造函数。这确保了父类的字段也被设置，并且方法可用，而无需做任何其他事情：它们是自动继承的。调用`super()`是必须的，当你在一个继承自另一个类的类中时，否则你会得到一个`ReferenceError`。

因为我们在`Motorcycle`中可以访问`Vehicle`的字段，所以这会起作用：

```js
let motor = new Motorcycle("Black", 0, 250, "gasoline");
console.log(motor.color);
motor.accelerate(50);
motor.move(); 
```

这就是它将输出的内容：

```js
Black
moving at 50 
```

我们不能在我们的`Vehicle`类中访问任何`Motorcycle`特定的属性或方法。这是因为并不是所有的车辆都是摩托车，所以我们不能确定我们会有来自子类的属性或方法。

目前，我们没有在这里使用任何 getter 和 setter，但我们当然可以。如果父类中有 getter 和 setter，它们也会被子类继承。这样我们就可以影响哪些属性可以被获取和修改（以及如何修改）在我们的类外部。这通常是一个好的实践。

# 原型

原型是 JavaScript 中使对象成为可能的一种机制。在创建类时没有指定任何内容时，对象会从 `Object.prototype` 原型继承。这是一个相当复杂的内置 JavaScript 类，我们可以使用它。我们不需要查看它在 JavaScript 中的实现方式，因为我们把它看作是始终位于继承树顶部的基对象，因此始终存在于我们的对象中。

所有类都有一个名为 "prototype" 的 `prototype` 属性，我们可以这样访问它：

```js
ClassName.prototype 
```

让我们通过 `prototype` 属性向一个类添加函数的例子。为了做到这一点，我们将使用这个 `Person` 类：

```js
class Person {
  constructor(firstname, lastname) {
    this.firstname = firstname;
    this.lastname = lastname;
  }
  greet() {
    console.log("Hi there!");
  }
} 
```

如此，这是如何使用 `prototype` 向此类添加函数的方法：

```js
Person.prototype.introduce = function () {
  console.log("Hi, I'm", this.firstname);
}; 
```

`prototype` 是一个包含对象所有属性和方法属性的属性。因此，向 `prototype` 添加函数就是向类添加函数。你可以使用 `prototype` 向对象添加属性或方法，就像我们在上面的例子中用 `introduce` 函数所做的那样。你也可以为属性做同样的事情：

```js
Person.prototype.favoriteColor = "green"; 
```

然后你可以从 `Person` 的实例中调用它们：

```js
let p = new Person("Maria", "Saga");
console.log(p.favoriteColor);
p.introduce(); 
```

它将输出：

```js
green
Hi, I'm Maria 
```

就好像你定义了一个类，其中有一个喜欢的颜色并持有默认值，还有一个函数 `introduce`。它们已经被添加到类中，并且对所有实例和未来的实例都是可用的。

因此，通过 `prototype` 定义的属性和方法实际上就像它们是在类中定义的一样。这意味着对于特定实例的重写不会影响所有实例。例如，如果我们有一个第二个 `Person` 对象，这个人可以重写 `favoriteColor` 值，这不会改变 `firstname` 为 `Maria` 的对象中的值。

当你控制类代码并希望永久更改它时，你不应该使用这种方法。在这种情况下，只需更改类即可。然而，你可以像这样扩展现有对象，甚至有条件地扩展现有对象。重要的是要知道，JavaScript 内置对象有原型，并从 `Object.prototype` 继承。但是，请确保不要修改这个原型，因为它会影响我们的 JavaScript 的工作方式。

## 练习 7.3

创建一个包含不同动物物种及其叫声的属性的类，并创建两个（或更多）动物：

1.  创建一个打印给定动物及其叫声的方法。

1.  为动物添加一个具有另一个动作的原型。

1.  将整个动物对象输出到控制台。

# 章节项目

## 员工跟踪应用

创建一个类来跟踪公司的员工：

1.  在构造函数中使用名字、姓氏和工作年数作为值。

1.  创建两个或更多具有他们的名字、姓氏和在公司工作年数的人。将这些人添加到数组中。

1.  设置一个原型来返回人员的姓名细节以及他们在公司工作的时间。

1.  遍历数组的元素，将结果输出到控制台，并添加一些文本使输出成为一个完整的句子。

## 菜单项价格计算器

创建一个类，它将允许你计算出多个项目的组合价格，并与它交互以计算出不同订单的总成本。

1.  创建一个包含两个菜单项价格的私有字段声明。

1.  在类中使用构造函数来获取参数值（购买每种物品的数量）。

1.  创建一个方法来根据用户选择的每种物品的数量计算并返回总成本。

1.  使用 getter 属性来获取计算方法输出的值。

1.  创建两个或三个具有不同菜单选择组合的对象，并在控制台输出总成本。

# 自我检查测验

1.  用于创建类的关键字是什么？

1.  你会如何设置一个包含`first`和`last`作为初始属性的类来存储人的姓名？

1.  将一种事物获得另一种事物的属性和行为的概念称为什么？

1.  以下关于`constructor`方法的哪些说法是正确的？

    +   当创建新对象时自动执行。

    +   应该只在之后添加。

    +   它必须包含`constructor`关键字。

    +   它用于初始化对象属性。

    +   当你有多重值时可以使用它。

1.  修复以下代码，以便原型将`Person`的姓名输出到控制台。`Person`原型的正确语法是什么？

    ```js
    function Person(first,last) {
      this.first = first;
      this.last = last;
    }
    // What should go here: A, B, or C?
    const friend1 = new Person("Laurence", "Svekis");
    console.log(friend1.getName()); 
    ```

    A)

    ```js
    Person.prototype.getName = (first,last) {
      return this.first + " " + this.last;
    }; 
    ```

    B)

    ```js
    Person.prototype.getName = function getName() {
    return this.first + " " + this.last;
    }; 
    ```

    C)

    ```js
    Person.prototype = function getName() {
    return this.first + " " + this.last;
    }; 
    ```

# 概述

在本章中，我们向您介绍了面向对象编程（OOP）的概念。这意味着我们以这种方式组织代码，使得对象是逻辑中的核心角色。类是对象的蓝图。我们可以为对象制作一个模板，并通过使用`new`关键字轻松创建实例。

我们看到类可以通过使用`extends`关键字相互继承。从另一个类继承的类必须使用`super()`调用此类的构造函数，然后自动访问所有父类的属性和方法。这对于可重用和高度可维护的代码来说是非常好的。

最后，我们遇到了原型。这是 JavaScript 的内置概念，使得类成为可能。通过使用`prototype`向类添加属性和方法，我们可以修改该类的蓝图。

在下一章中，我们将考虑一些 JavaScript 的内置方法，这些方法可以用来操纵和增加代码的复杂性！
