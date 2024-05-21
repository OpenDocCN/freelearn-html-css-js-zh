# 第八章。类和模块

在本章中，我们将探讨 ES6 中引入的一些最有趣的特性。JavaScript 是一种基于原型的语言，支持原型继承。在上一章中，我们讨论了对象的原型属性以及原型继承在 JavaScript 中的工作原理。ES6 引入了类。如果你来自 Java 等传统的面向对象语言，你会立即理解类的这些熟悉概念。然而，在 JavaScript 中它们并不相同。JavaScript 中的类是对我们在上一章中讨论的原型继承的一种语法糖。

在本章中，我们将详细了解 ES6 类和模块-这些是 JavaScript 这一版本中的受欢迎的变化，使**面向对象编程**（**OOP**）和继承变得更加容易。

如果你来自传统的面向对象语言，原型继承可能会让你感到有些不适。ES6 类为你提供了更传统的语法，让你熟悉 JavaScript 中的原型继承。

在我们尝试深入了解类之前，让我向你展示为什么你应该使用 ES6 类语法而不是 ES5 的原型继承语法。

在这个片段中，我正在创建一个`Person`、`Employee`和`Engineer`的类层次结构，非常简单。首先，我们将看到 ES5 原型继承，写法如下：

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

现在让我们看一下使用 ES6 类语法的等效代码：

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

如果你观察前面的两个代码片段，你会发现第二个例子非常整洁。如果你已经了解 Java 或 C#，你会感到非常熟悉。然而，要记住的一点是，类并没有引入任何新的面向对象的继承模型到语言中，而是引入了一种更好的创建对象和处理继承的方式。

# 定义类

在底层，类是特殊的函数。就像你可以使用函数表达式和声明来定义函数一样，你也可以定义类。定义类的一种方式是使用类声明。

你可以使用`class`关键字和类的名称。这个语法非常类似于 Java 或 C#：

```js
    class Car { 
      constructor(model, year){ 
        this.model = model; 
        this.year = year; 
      } 
    } 
    console.log(typeof Car); //"function" 

```

为了证明类是一个特殊的函数，如果我们使用`typeof`操作符来获取`Car`类，我们会得到一个函数。

类和普通函数之间有一个重要的区别。普通函数会被提升，而类不会。普通函数在进入声明它的作用域时立即可用；这被称为**提升**，这意味着普通函数可以在作用域中的任何地方声明，并且它将可用。然而，类不会被提升；它们只有在声明后才可用。对于普通函数，你可以说：

```js
    normalFunction();   //use first 
    function normalFunction() {}  //declare later 

```

然而，你不能在声明之前使用类，例如：

```js
    var ford = new Car(); //Reference Error 
    class Car {} 

```

定义类的另一种方式是使用类表达式。类表达式，就像函数表达式一样，可能有也可能没有名称。

以下示例显示了一个匿名类表达式：

```js
    const Car = class { 
      constructor(model, year){ 
        this.model = model; 
        this.year = year; 
      } 
    } 

```

如果你给类表达式命名，这个名称只在类的主体内部可用，而在外部是不可用的。

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

如你所见，在这里，我们给`Car`类命名。这个名称在类的主体内是可用的，但当我们尝试在类外部访问它时，会得到一个引用错误。

在类的成员之间不能使用逗号。但分号是有效的。这很有趣，因为 ES6 忽略了分号，关于在 ES6 中使用分号有激烈的争论。考虑以下代码片段作为例子：

```js
    class NoCommas { 
      method1(){} 
      member1;  //This is ignored and can be used to 
        separate class members 
      member2,  //This is an error 
      method2(){} 
    } 

```

一旦定义，我们可以通过`new`关键字而不是函数调用来使用类；以下是例子：

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

到目前为止，我们在示例中使用了`constructor`函数。构造函数是用于创建和初始化使用类创建的对象的特殊方法。一个类中只能有一个构造函数。构造函数与普通构造函数有些不同。与普通构造函数不同，类构造函数可以通过`super()`调用其父类构造函数。当我们讨论继承时，我们将详细讨论这一点。

## 原型方法

原型方法是类的原型属性，并且被类的实例继承。

原型方法也可以有`getter`和`setter`方法。获取器和设置器的语法与 ES5 相同：

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

同样，计算属性也是支持的。您可以使用表达式定义方法的名称。表达式需要放在方括号内。我们在之前的章节中讨论了这种简写语法。以下都是等价的：

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

静态方法与类相关联，而不是与该类的实例（对象）相关联。换句话说，您只能使用类的名称来调用静态方法。静态方法在不实例化类的情况下被调用，它们不能在类的实例上调用。静态方法在创建实用程序或辅助方法时很受欢迎。考虑以下代码片段：

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

您可能会问，我们有静态方法，那静态属性呢？在忙于准备 ES6 的过程中，他们没有添加静态属性。它们将在语言的未来迭代中添加。

## 生成器方法

我们在前几章中讨论了非常有用的生成器函数。您可以将生成器函数作为类的一部分添加，并将它们称为生成器方法。生成器方法很有用，因为您可以将它们的键定义为`Symbol.iterator`。以下示例显示了如何在类内部定义生成器方法：

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

到目前为止，我们讨论了如何声明类以及类可以支持的成员类型。类的一个主要用途是作为创建其他子类的模板。当您从一个类创建子类时，您会继承父类的属性，并通过添加自己的特性来扩展父类。

让我们看看继承的以下事实例子：

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

在这里，`Animal`是基类，`Cat`类是从类`Animal`派生出来的。扩展子句允许您创建现有类的子类。此示例演示了子类化的语法。让我们通过编写以下代码来进一步增强此示例：

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

在这里，我们使用`super`关键字来调用父类的函数。以下是`super`关键字的三种用法：

+   您可以使用`super (<params>)`作为函数调用来调用父类的构造函数

+   您可以使用`super.<parentClassMethod>`来访问父类方法

+   您可以使用`super.<parentClassProp>`来访问父类属性

在派生类构造函数中，您必须在使用`this`关键字之前调用`super()`方法；例如，以下代码将失败：

```js
    class Base {} 
    class Derive extends Base { 
      constructor(name){ 
        this.name = name; //'this' is not allowed before super() 
      } 
    } 

```

您不能将派生构造函数隐式地使用`super()`方法作为错误：

```js
    class Base {} 
    class Derive extends Base { 
      constructor(){  //missing super() call in constructor 
      } 
    } 

```

如果您没有为基类提供构造函数，则将使用以下构造函数：

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

JavaScript 仅支持单一继承。最多，一个类可以有一个超类。当您想要创建类层次结构，但又想要从不同来源继承工具方法时，这是有限的。

假设我们有一个场景，我们有一个`Person`类，并且我们创建一个子类`Employee`：

```js
    class Person {} 
    class Employee extends Person{} 

```

我们还希望从两个实用类`BackgroundCheck`（此类执行员工背景检查）和`Onboard`（此类处理员工入职流程，例如打印工牌等）中继承函数：

```js
    class BackgroundCheck { 
      check() {} 
    } 
    class Onboard { 
      printBadge() { } 
    } 

```

`BackgroundCheck`和`Onboard`类都是模板，它们的功能将被多次使用。这种模板（抽象子类）称为 mixin。

由于 JavaScript 不支持多重继承，我们将采用不同的技术来实现这一点。在 ES6 中实现 mixin 的一种流行方式是编写一个以超类为输入的函数，并将子类扩展为输出，例如：

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

这基本上意味着`Employee`是`BackgroundCheck`的子类，而`BackgroundCheck`又是`Onboard`的子类，而`Onboard`又是`Person`的子类。

# 模块

JavaScript 模块并不新鲜。事实上，有一段时间支持模块的库。然而，ES6 提供了内置模块。传统上，JavaScript 主要用于浏览器，大部分 JavaScript 代码要么是嵌入式的，要么足够小，可以轻松管理。事情已经改变。JavaScript 项目现在规模庞大。如果没有一个有效的系统将代码分散到文件和目录中，管理代码将变成一场噩梦。

ES6 模块是文件。一个文件对应一个模块。没有模块关键字。您在模块文件中编写的任何代码都是局部的，除非您导出它。您可能在一个模块中有一堆函数，但您只想导出其中的一些。您可以以几种方式导出模块功能。

第一种方法是使用`export`关键字。您可以导出任何顶级`function`，`class`，`var`，`let`或`const`。

以下示例显示了`server.js`中的一个模块，我们在其中导出了一个`function`，一个`class`和一个`const`。我们不导出`processConfig()`函数，任何导入此模块的文件都无法访问未导出的函数：

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

任何可以访问`server.js`的代码都可以导入导出的功能：

```js
    //--------------app.js---------------------------- 
    import {Config, startServer} from 'server' 
    startServer(port); 

```

在这种情况下，另一个 JavaScript 文件正在从`server`模块（对应的 JavaScript 文件`server.js`）中导入`Config`和`startServer`（我们省略了文件扩展名）。

您还可以导入模块中导出的所有内容：

```js
    import * from 'server' 

```

如果只有一件事要导出，可以使用默认导出语法。考虑以下代码片段作为示例：

```js
    //----------------server.js--------------------- 
    export default class { 
      //... 
    } 
    //--------------app.js---------------------------- 
    import Server from 'server'; 
    const s = new Server(); 

```

在这个例子中，我们将保持类匿名，因为我们可以在外部使用模块名称本身作为引用。

在 ES6 模块之前，外部库支持了几种模块方法。它们为 ES6 制定了相当不错的指南/风格。ES6 遵循以下风格：

+   模块是单例的。模块只被导入一次，即使您在代码中尝试多次导入它。

+   变量、函数和其他类型的声明都是局部的。只有用`export`标记的声明才能在模块外部进行`import`。

+   模块可以从其他模块导入。以下是引用其他模块的三种选项：

+   您可以使用相对路径`("../lib/server");`这些路径是相对于导入模块的文件解析的。例如，如果您从`<project_path>/src/app.js`导入模块，并且模块文件位于`<project_path>/lib/server.js`，您需要提供相对于`app.js`的路径 - 在这种情况下是`../lib/server`。

+   绝对路径也可以直接指向模块文件。

+   在导入模块时，可以省略文件的`.js`扩展名。

在我们深入了解 ES6 模块系统的更多细节之前，我们需要了解 ES5 是如何通过外部库支持它们的。ES5 有两种不兼容的模块系统，分别是：

+   **CommonJS**：这是主导标准，因为 Node.js 采用了它

+   **AMD**（**异步模块定义**）：这比 CommonJS 稍微复杂一些，设计用于异步模块加载，并针对浏览器

ES6 模块的目标是让来自任何这些系统的工程师都能轻松使用。

## 导出列表

与使用`export`关键字为模块中的每个导出函数或类打标签不同，您可以编写一个包含要从模块导出的所有内容的单个列表，如下所示：

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

模块的第一行是导出的列表。您可以在模块文件中有多个`export`列表，并且列表可以出现在文件的任何位置。您还可以在同一个模块文件中同时拥有`export`列表和`export`声明的混合，但是您只能`export`一个名称一次。

在一个大型项目中，有时会遇到名称冲突的情况。假设您导入了两个模块，并且它们都导出了相同名称的函数。在这种情况下，您可以按如下方式重命名导入：

```js
    import {trunc as StringLib} from "../lib/string.js" 
    import {trunc as MathLib} from "../lib/math.js" 

```

在这里，导入的两个模块都导出了一个名为`trunc`的名称，因此创建了名称冲突。我们可以为它们取别名以解决这个冲突。

您也可以在导出时进行重命名，如下所示：

```js
    function v() {} 
    function v2() {} 
    export { 
      v as functionV(), 
      v2 as functionV2(), 
      v2 as functionLatest() 
    } 

```

如果您已经在使用 ES5 模块系统，ES6 模块可能看起来多余。然而，对于语言来说，支持这样一个重要的功能非常重要。ES6 模块语法也是标准化的，比其他替代方案更紧凑。

# 总结

在本章中，我们专注于理解 ES6 类。ES6 类正式支持了使用函数和原型模拟类似继承层次结构的常见 JavaScript 模式。它们是基于原型的 OO 的语法糖，为鼓励互操作性的类模式提供了方便的声明形式。ES6 类提供了一个更好、更清晰的语法，用于创建这些对象并处理继承。ES6 类支持构造函数、实例和静态方法、（基于原型的）继承和 super 调用。

到目前为止，JavaScript 缺少最基本的功能之一 - 模块。在 ES6 之前，我们使用 CommonJS 或 AMD 编写模块。ES6 正式将模块引入 JavaScript。在本章中，我们详细了解了 ES6 中模块的使用。

下一章将重点介绍 ES6 的另一个有趣的新增功能 - 代理和承诺。
