# 第八章。类和模块

在本章中，我们将探讨 ES6 中引入的一些最有趣的功能。JavaScript 是一种基于原型的语言，支持原型继承。在前一章中，我们讨论了对象的原型属性以及 JavaScript 中的原型继承是如何工作的。ES6 引入了类。如果你来自传统的面向对象语言，如 Java，你会立即联想到类的一些众所周知的概念。然而，在 JavaScript 中它们并不相同。JavaScript 中的类是我们在上一章讨论的原型继承之上的语法糖。

在本章中，我们将详细探讨 ES6 类和模块——这些是 JavaScript 版本中的受欢迎的改进，使得**面向对象编程**（**OOP**）和继承变得显著更容易。

如果你来自传统的面向对象语言，原型继承可能对你来说有点格格不入。ES6 类为你提供了一个更传统的语法，以便你熟悉 JavaScript 中的原型继承。

在我们尝试深入探讨类之前，让我先展示一下为什么你应该使用 ES6 类语法而不是 ES5 的原型继承语法。

在这个片段中，我正在创建一个 `Person`、`Employee` 和 `Engineer` 的类层次结构，相当直接。首先，我们将看到 ES5 原型继承，其写法如下：

```js
    var Person = function(firstname) { 
        if (!(this instanceof Person)) { 
            throw new Error("Person is a constructor"); 
        } 
        this.firstname = firstname; 
    }; 

    Person.prototype.giveBirth = function() { 
        // ...we give birth to the person 
    }; 

    var Employee = function(firstname, lastname, job) { 
        if (!(this instanceof Employee)) { 
            throw new Error("Employee is a constructor"); 
        } 
        Person.call(this, firstname); 
        this.job = job; 
    };  
    Employee.prototype = Object.create(Person.prototype); 
    Employee.prototype.constructor = Employee; 
    Employee.prototype.startJob = function() { 
        // ...Employee starts job 
    }; 

    var Engineer = function(firstname, lastname, job, department) { 
        if (!(this instanceof Engineer)) { 
            throw new Error("Engineer is a constructor"); 
        } 
        Employee.call(this, firstname, lastname, job); 
        this.department = department; 
    }; 
    Engineer.prototype = Object.create(Employee.prototype); 
    Engineer.prototype.constructor = Engineer; 
    Engineer.prototype.startWorking = function() { 
        // ...Engineer starts working 
    }; 

```

现在，让我们看看使用 ES6 类语法等效的代码：

```js
    class Person { 
        constructor(firstname) { 
            this.firsnamet = firstname; 
        } 
        giveBirth() { 
            // ... a person is born 
        } 
    } 

    class Employee extends Person { 
        constructor(firstname, lastname, job) { 
            super(firstname); 
            this.lastname = lastname; 
            this.position = position; 
        } 

         startJob() { 
            // ...Employee starts job 
        } 
    } 

    class Engineer extends Employee { 
        constructor(firstname, lastname, job, department) { 
            super(firstname, lastname, job); 
            this.department = department; 
        } 

        startWorking() { 
            // ...Engineer starts working 
        } 
    } 

```

如果你观察前两个代码片段，你会明显感觉到第二个例子相当整洁。如果你已经熟悉 Java 或 C#，你会感到非常自在。然而，有一件重要的事情需要记住，类并没有向语言引入任何新的面向对象继承模型，而是带来了一种创建对象和处理继承的更优雅的方式。

# 定义类

在底层，类是特殊的函数。就像你可以使用函数表达式和声明来定义函数一样，你也可以定义类。定义类的一种方法就是使用类声明。

你可以使用 `class` 关键字和类的名称。这种语法与 Java 或 C# 非常相似：

```js
    class Car { 
      constructor(model, year){ 
        this.model = model; 
        this.year = year; 
      } 
    } 
    console.log(typeof Car); //"function" 

```

为了证明类是特殊的函数，如果我们获取 `Car` 类的 `typeof`，我们会得到一个函数。

类和普通函数之间存在一个重要的区别。虽然普通函数是提升的，但类不是。一个普通函数在你进入其声明的范围时立即可用；这被称为**提升**，这意味着普通函数可以在作用域的任何位置声明，并且它将是可用的。然而，类不是提升的；它们只有在声明之后才可用。对于一个普通函数，你可以这样说：

```js
    normalFunction();   //use first 
    function normalFunction() {}  //declare later 

```

然而，你不能在声明之前使用类，例如：

```js
    var ford = new Car(); //Reference Error 
    class Car {} 

```

定义类的另一种方式是使用类表达式。类表达式，就像函数表达式一样，可以有名字也可以没有名字。

以下示例展示了匿名类表达式：

```js
    const Car = class { 
      constructor(model, year){ 
        this.model = model; 
        this.year = year; 
      } 
    } 

```

如果给类表达式命名，该名字是局部于类的主体，且在类外部不可用：

```js
    const NamedCar = class Car{ 
      constructor(model, year){ 
        this.model = model; 
        this.year = year; 
      } 
      getName() { 
          return Car.name; 
      } 
    } 
    const ford = new NamedCar(); 
    console.log(ford.getName()); // Car 
    console.log(ford.name); // ReferenceError: name is not defined 

```

如您所见，这里我们将给 `Car` 类起一个名字。这个名字在类的主体内部是可用的，但当我们尝试在类外部访问它时，会得到一个引用错误。

在分隔类的成员时，你不能使用逗号。虽然分号是有效的。这很有趣，因为 ES6 忽略分号，并且关于在 ES6 中使用分号存在激烈的争论。以下代码片段可以作为示例：

```js
    class NoCommas { 
      method1(){} 
      member1;  //This is ignored and can be used to 
        separate class members 
      member2,  //This is an error 
      method2(){} 
    } 

```

一旦定义，我们可以通过 `new` 关键字使用类，而不是函数调用；以下是一个示例：

```js
    class Car { 
      constructor(model, year){ 
        this.model = model; 
        this.year = year; 
      } 
    } 
    const fiesta = new Car('Fiesta','2010'); 

```

## 构造函数

到目前为止，我们在示例中使用了 `constructor` 函数。构造函数是一种特殊的方法，用于使用类创建和初始化对象。一个类中只能有一个构造函数。构造函数与普通的构造函数函数有点不同。与普通构造函数不同，类构造函数可以通过 `super()` 调用其父类构造函数。我们将在讨论继承时详细讨论这一点。

## 原型方法

原型方法是类的原型属性，并且它们被类的实例继承。

原型方法也可以有 `getter` 和 `setter` 方法。getter 和 setter 的语法与 ES5 相同：

```js
    class Car { 
      constructor(model, year){ 
        this.model = model; 
        this.year = year; 
      } 
      get model(){ 
        return this.model 
      } 

      calculateCurrentValue(){ 
        return "7000" 
      } 
    } 
    const fiesta = new Car('Fiesta','2010') 
    console.log(fiesta.model) 

```

同样，计算属性也得到了支持。您可以使用表达式来定义方法的名字。表达式需要放在方括号内。我们已经在前面的章节中讨论了这种简写语法。以下都是等效的：

```js
    class CarOne { 
        driveCar() {} 
    } 
    class CarTwo { 
        ['drive'+'Car']() {} 
    } 
    const methodName = 'driveCar'; 
    class CarThree { 
        [methodName]() {} 
    } 

```

## 静态方法

静态方法与类相关联，而不是与该类的实例（对象）相关联。换句话说，你只能通过类的名字来访问静态方法。静态方法在实例化类之前被调用，并且不能在类的实例上调用。静态方法在创建实用或辅助方法时很受欢迎。以下是一段代码示例：

```js
    class Logger { 
      static log(level, message) { 
        console.log(`${level} : ${message}`) 
      } 
    } 
    //Invoke static methods on the Class 
    Logger.log("ERROR","The end is near") //"ERROR : The end is near" 

    //Not on instance 
    const logger = new Logger("ERROR") 
    logger.log("The end is near")     //logger.log is not a function 

```

## 静态属性

您可能会问——我们已经有静态方法了，静态属性怎么办？在匆忙准备 ES6 的过程中，他们没有添加静态属性。它们将在语言的未来迭代中添加。

## 生成器方法

我们在几章之前讨论了非常有用的生成器函数。您可以将生成器函数作为类的一部分添加，它们被称为生成器方法。生成器方法很有用，因为您可以将它们的键定义为 `Symbol.iterator`。以下示例展示了如何在类内部定义生成器方法：

```js
    class iterableArg { 
        constructor(...args) { 
            this.args = args; 
        } 
        * [Symbol.iterator]() { 
            for (const arg of this.args) { 
                yield arg; 
            } 
        } 
    } 

    for (const x of new iterableArg('ES6', 'wins')) { 
        console.log(x); 
    } 

    //ES6 
    //wins 

```

# 子类化

到目前为止，我们讨论了如何声明类以及类可以支持成员类型。类的主要用途是作为一个模板来创建其他子类。当你从一个类创建子类时，你继承了父类的属性，并通过添加更多自己的特性来扩展父类。

让我们看看以下关于继承的实际情况示例：

```js
    class Animal {  
      constructor(name) { 
        this.name = name; 
      } 
        speak() { 
        console.log(this.name + ' generic noise'); 
      } 
    } 
    class Cat extends Animal { 
      speak() { 
        console.log(this.name + ' says Meow.'); 
      } 
    } 
    var c = new Cat('Grace');  
    c.speak();//"Grace says Meow." 

```

这里，`Animal` 是基类，`Cat` 类是从 `Animal` 类派生出来的。扩展子句允许你创建现有类的子类。以下示例演示了子类化的语法。让我们通过以下代码进一步扩展这个示例：

```js
    class Animal {  
      constructor(name) { 
        this.name = name; 
      } 
      speak() { 
        console.log(this.name + ' generic noise'); 
      } 
    } 
    class Cat extends Animal { 
      speak() { 
        console.log(this.name + ' says Meow.'); 
      } 
   } 
    class Lion extends Cat { 
      speak() { 
        super.speak(); 
        console.log(this.name + ' Roars....'); 
      } 
    } 
    var l = new Lion('Lenny');  
    l.speak(); 
    //"Lenny says Meow." 
    //"Lenny Roar...." 

```

这里，我们使用 `super` 关键字来调用父类的函数。以下是通过 `super` 关键字可以使用的三种方式：

+   你可以使用 `super (<params>)` 作为函数调用来调用父类的构造函数

+   你可以使用 `super.<parentClassMethod>` 来访问父类方法

+   你可以使用 `super.<parentClassProp>` 来访问父类的属性

在派生类构造函数中，你必须在使用 `this` 关键字之前调用 `super()` 方法；例如，以下代码将失败：

```js
    class Base {} 
    class Derive extends Base { 
      constructor(name){ 
        this.name = name; //'this' is not allowed before super() 
      } 
    } 

```

你不能隐式地从一个带有 `super()` 方法的派生构造函数中退出，否则会出错：

```js
    class Base {} 
    class Derive extends Base { 
      constructor(){  //missing super() call in constructor 
      } 
    } 

```

如果你没有为基类提供构造函数，将使用以下构造函数：

```js
    constructor() {} 

```

对于派生类，默认构造函数如下：

```js
    constructor(...args){ 
      super(...args); 
    } 

```

## 混入

JavaScript 只支持单继承。一个类最多只能有一个超类。当你想要创建类层次结构并从不同来源继承工具方法时，这会有限制。

假设我们有一个场景，其中有一个 `Person` 类，我们创建一个子类 `Employee`：

```js
    class Person {} 
    class Employee extends Person{} 

```

我们还希望从两个实用类中继承函数，`BackgroundCheck` - 这个类负责员工的背景调查 - 和 `Onboard` - 这个类处理员工的入职流程，例如打印胸牌等：

```js
    class BackgroundCheck { 
      check() {} 
    } 
    class Onboard { 
      printBadge() { } 
    } 

```

`BackgroundCheck` 和 `Onboard` 类都是模板，它们的功能将被多次使用。这样的模板（抽象子类）被称为混入。

由于 JavaScript 中不支持多重继承，我们将采用不同的技术来实现这一点。在 ES6 中实现混入的一个流行方法是将一个函数作为输入，将一个扩展该超类的子类作为输出，例如：

```js
    class Person {} 
    const BackgroundCheck = Tools => class extends Tools { 
      check() {} 
    }; 
    const Onboard = Tools => class extends Tools { 
      printBadge() {} 
    }; 
    class Employee extends BackgroundCheck(Onboard(Person)){  
    } 

```

这实际上意味着 `Employee` 是 `BackgroundCheck` 的子类，而 `BackgroundCheck` 又是 `Onboard` 的子类，`Onboard` 又是 `Person` 的子类。

# 模块

JavaScript 模块并不新鲜。实际上，一些库已经支持模块一段时间了。然而，ES6 提供了内置的模块。传统上，JavaScript 的主要用途是在浏览器上，其中大部分 JavaScript 代码要么是内嵌的，要么足够小，不需要太多麻烦就可以管理。但现在情况变了。JavaScript 项目现在规模庞大。如果没有一个有效的系统将代码分散到文件和目录中，管理代码就会变得非常困难。

ES6 模块是文件。每个文件一个模块，每个模块一个文件。没有模块关键字。你可以在模块文件中编写的任何代码都是模块本地的，除非你导出它。你可能在模块中有一系列函数，但只想导出其中几个。你可以通过几种方式导出模块功能。

第一种方法是使用 `export` 关键字。你可以导出任何顶层 `function`、`class`、`var`、`let` 或 `const`。

以下示例展示了 `server.js` 内部的模块，其中我们导出了一个 `function`、一个 `class` 和一个 `const`。我们没有导出 `processConfig()` 函数，因此任何导入此模块的文件都无法访问未导出的函数：

```js
    //----------------server.js--------------------- 
    export const port = 8080; 
    export function startServer() { 
      //...start server 
    } 
    export class Config { 
      //... 
    } 
    function processConfig() { 
      //... 
    } 

```

任何可以访问 `server.js` 的代码都可以导入导出的功能：

```js
    //--------------app.js---------------------------- 
    import {Config, startServer} from 'server' 
    startServer(port); 

```

在这种情况下，另一个 JavaScript 文件正在从 `server` 模块（对应 JavaScript 文件 `server.js`，我们省略了文件扩展名）导入 `Config` 和 `startServer`。

你也可以导入模块导出的所有内容：

```js
    import * from 'server' 

```

如果你只有一个要导出的东西，你可以使用默认导出语法。以下代码片段作为示例：

```js
    //----------------server.js--------------------- 
    export default class { 
      //... 
    } 
    //--------------app.js---------------------------- 
    import Server from 'server'; 
    const s = new Server(); 

```

在这个例子中，我们将保持类匿名，因为我们可以使用模块名本身作为外部的引用。

在 ES6 模块之前，外部库支持了多种模块方法。它们为 ES6 建立了相当好的指南/风格。ES6 遵循以下风格：

+   模块是单例的。模块只导入一次，即使你在代码中尝试多次导入它。

+   变量、函数和其他类型的声明是模块本地的。只有标记了 `export` 的声明才可以在模块外部通过 `import` 使用。

+   模块可以从其他模块导入。以下是指向其他模块的三个选项：

    +   你可以使用相对路径 `("../lib/server");` 这些路径相对于导入模块的文件进行解析。例如，如果你从 `<project_path>/src/app.js` 导入模块，而模块文件位于 `<project_path>/lib/server.js`，你需要提供相对于 `app.js` 的路径 - 在这种情况下是 `../lib/server`。

    +   绝对路径也可以直接指向模块文件。

    +   在导入模块时，你可以省略文件 `.js` 扩展名。

在我们深入了解 ES6 模块系统之前，我们需要了解 ES5 是如何通过外部库支持它们的。ES5 有两个不兼容的模块系统，如下所示：

+   **CommonJS**：这是 Node.js 采用的主导标准

+   **AMD**（**异步模块定义**）：这比 CommonJS 稍微复杂一些，旨在用于异步模块加载，并针对浏览器设计

ES6 模块旨在使来自任何这些系统的工程师都能轻松使用。

## 导出列表

你不必为模块中的每个导出的函数或类加上`export`关键字，你可以写一个包含所有你想从模块中导出的内容的单个列表，如下所示：

```js
    export {port, startServer, Config}; 
    const port = 8080; 
    function startServer() { 
      //...start server 
    } 
    class Config { 
      //... 
    } 
    function processConfig() { 
      //... 
    } 

```

模块的第一行是导出列表。你可以在模块文件中有多个`export`列表，并且列表可以出现在文件的任何位置。你还可以在同一个模块文件中混合`export`列表和`export`声明，但只能导出一个名称一次。

在一个大项目中，有时会遇到名称冲突的情况。假设你导入了两个模块，并且它们都导出了一个同名函数。在这种情况下，你可以按照以下方式重命名导入：

```js
    import {trunc as StringLib} from "../lib/string.js" 
    import {trunc as MathLib} from "../lib/math.js" 

```

在这里，导入的两个模块都导出了一个名为`trunc`的名称，因此产生了名称冲突。我们可以通过别名来解决这个问题。

你也可以在导出时进行重命名，如下所示：

```js
    function v() {} 
    function v2() {} 
    export { 
      v as functionV(), 
      v2 as functionV2(), 
      v2 as functionLatest() 
    } 

```

如果你已经在使用 ES5 模块系统，ES6 模块可能看起来是多余的。然而，对于语言来说，支持这样一个重要的特性是非常关键的。ES6 模块语法也是标准化的，并且比其他替代方案更紧凑。

# 摘要

在本章中，我们专注于理解 ES6 类。ES6 类为使用函数和原型模拟类继承层次结构的常见 JavaScript 模式提供了正式支持。它们是原型基于的面向对象编程的语法糖，提供了一种方便的声明性形式，鼓励互操作性。ES6 类为创建这些对象和处理继承提供了更优雅、更清晰、更简洁的语法。ES6 类提供了对构造函数、实例和静态方法、（基于原型的）继承和 super 调用的支持。

到目前为止，JavaScript 缺少了一个最基本的功能——模块。在 ES6 之前，我们使用 CommonJS 或 AMD 来编写模块。ES6 将模块正式引入 JavaScript。在本章中，我们详细探讨了如何在 ES6 中使用模块。

下一章将重点介绍 ES6 的另一个有趣的新增功能——代理和承诺。
