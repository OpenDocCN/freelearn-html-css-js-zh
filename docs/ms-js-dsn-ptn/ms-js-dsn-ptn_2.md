# 第一部分。经典设计模式

组织代码

创造模式

结构模式

行为模式

# 第二章：组织代码

在本章中，我们将看看如何将 JavaScript 代码组织成可重用、可理解的代码块。语言本身并不适合这种模块化，但多年来出现了许多组织 JavaScript 代码的方法。本章将论证需要拆分代码，然后逐步介绍创建 JavaScript 模块的方法。

我们将涵盖以下主题：

+   全局范围

+   对象

+   原型继承

+   ECMAScript 2015 类

# 代码块

任何人学习编程的第一件事就是无处不在的 Hello World 应用程序。这个简单的应用程序将“hello world”的某种变体打印到屏幕上。取决于你问的人，hello world 这个短语可以追溯到 20 世纪 70 年代初，当时它被用来演示 B 编程语言，甚至可以追溯到 1967 年，当时它出现在 BCL 编程指南中。在这样一个简单的应用程序中，无需担心代码的结构。事实上，在许多编程语言中，hello world 根本不需要结构。

对于 Ruby，情况如下：

```js
 **#!/usr/bin/ruby** 

 **puts "hello world"** 

```

对于 JavaScript（通过 Node.js），情况如下：

```js
 **#!/usr/local/bin/node** 

 **console.log("Hello world")** 

```

最初使用极其简单的技术来编程现代计算机。许多最初的计算机在解决问题时都被硬编码。它们不像我们今天拥有的通用计算机那样。相反，它们被构建为仅解决一个问题，例如解码加密文本。存储程序计算机最早是在 1940 年代末开发的。

最初用于编程这些计算机的语言通常非常复杂，通常与二进制密切相关。最终，创建了越来越高级的抽象，使编程更加易于访问。随着这些语言在 50 年代和 60 年代开始形成，很快就显而易见地需要一些方法来划分大块代码。

部分原因是为了保持程序员的理智，他们无法一次记住整个大型程序。然而，创建可重用的模块也允许在应用程序内部甚至在应用程序之间共享代码。最初的解决方案是利用语句，它们跳转程序的流程控制从一个地方到另一个地方。多年来，这些 GOTO 语句被大量依赖。对于一个不断受到有关使用 GOTO 语句的警告的现代程序员来说，这似乎是疯狂的。然而，直到第一批编程语言出现几年后，结构化编程才发展成为取代 GOTO 语法的形式。

结构化编程基于 Böhm-Jacopini 定理，该定理指出有一类相当大的问题，其答案可以使用三个非常简单的构造来计算：

+   子程序的顺序执行

+   两个子程序的条件执行

+   重复执行子程序，直到条件为真

敏锐的读者会认识到这些构造是正常的执行流程，一个分支或`if`语句和一个循环。

Fortran 是最早的语言之一，最初构建时没有支持结构化编程。然而，结构化编程很快被采纳，因为它有助于避免意大利面式代码。

Fortran 中的代码被组织成模块。模块是松散耦合的过程集合。对于那些来自现代面向对象语言的人来说，最接近的概念可能是模块就像一个只包含静态方法的类。

模块对于将代码分成逻辑分组非常有用。但是，它并没有为实际应用程序提供任何结构。面向对象语言的结构，即类和子类，可以追溯到 Ole-Johan Dahl 和 Kristen Nygaard 在 1967 年撰写的一篇论文。这篇论文将成为 Simula-67 的基础，这是第一种支持面向对象编程的语言。

虽然 Simula-67 是第一种具有类的语言，但与早期面向对象编程相关的最多讨论的语言是 Smalltalk。这种语言是在 20 世纪 70 年代在著名的施乐**帕洛阿尔托研究中心**（**PARC**）秘密开发的。它于 1980 年作为 Smalltalk-80 向公众发布（似乎所有具有历史意义的编程语言都以发布年份作为版本号的前缀）。Smalltalk 带来的是语言中的一切都是对象，甚至像 3 这样的文字数字也可以对它们执行操作。

几乎每种现代编程语言都有一些类的概念来组织代码。通常，这些类将属于一个称为命名空间或模块的更高级结构。通过使用这些结构，即使是非常大的程序也可以分成可管理和可理解的块。

尽管类和模块具有丰富的历史和明显的实用性，JavaScript 直到最近才支持它们作为一流构造。要理解为什么，只需简单地回顾一下 JavaScript 的历史，从第一章，*为了乐趣和利润而设计*，并意识到对于其最初的目的来说，拥有这样的构造将是多余的。类是注定要失败的 ECMAScript 4 标准的一部分，它们最终成为了 ECMAScript 2015 标准的一部分。

在本章中，我们将探讨一些在 JavaScript 中重新创建其他现代编程语言中的经典类结构的方法。

# 全局范围有什么问题？

在基于浏览器的 JavaScript 中，您创建的每个对象都分配给全局范围。对于浏览器，这个对象简单地称为**window**。通过在您喜欢的浏览器中打开开发控制台，可以很容易地看到这种行为。

### 提示

**打开开发控制台**

现代浏览器内置了一些非常先进的调试和审计工具。要访问它们，有一个菜单项，位于**工具** | **Chrome 开发者工具** | **工具** | **Firefox Web 开发者**下，以及直接在菜单下方的**F12 开发者工具**在 Internet Explorer 中。还存在用于访问工具的键盘快捷键。在 Windows 和 Linux 上，*F12*是标准的，在 OSX 上，使用`Option` + `Command` + `I`。

开发工具中有一个控制台窗口，可以直接访问当前页面的 JavaScript。这是一个非常方便的地方，可以测试小代码片段或访问页面的 JavaScript。

打开控制台后，输入以下代码：

```js
> var words = "hello world"
> console.log(window.words);
```

这将导致`hello world`打印到控制台。通过全局声明单词，它会自动附加到顶层容器：window。

在 Node.js 中，情况有些不同。以这种方式分配变量实际上会将其附加到当前模块。不包括`var`对象将会将变量附加到`global`对象上。

多年来，您可能听说过使用全局变量是一件坏事。这是因为全局变量很容易被其他代码污染。

考虑一个非常常见的变量名，比如`index`。在任何规模可观的应用程序中，这个变量名可能会在多个地方使用。当任一代码片段使用该变量时，它会导致另一代码片段出现意外结果。重用变量是可能的，甚至在内存非常有限的系统中也可能很有用，比如嵌入式系统，但在大多数应用程序中，在单个范围内重用变量以表示不同的含义是难以理解的，也是错误的根源。

使用全局作用域变量的应用程序也容易受到其他代码的攻击。从其他代码改变全局变量的状态是微不足道的，这可能会使攻击者暴露登录信息等机密信息。

最后，全局变量给应用程序增加了很多复杂性。将变量的范围减小到代码的一小部分可以让开发人员更容易理解变量的使用方式。当范围是全局时，对该变量的更改可能会影响到代码的其他部分。对变量的简单更改可能会影响整个应用程序。

一般来说，应该避免使用全局变量。

# JavaScript 中的对象

JavaScript 是一种面向对象的语言，但大多数人在使用它时并不会充分利用其面向对象的特性。JavaScript 使用混合对象模型，它既有原始值，也有对象。JavaScript 有五种原始类型：

+   未定义

+   空值

+   布尔值

+   字符串

+   数字

在这五个值中，只有两个是我们期望的对象。另外三个，布尔值、字符串和数字都有包装版本，它们是对象：Boolean、String 和 Number。它们以大写字母开头进行区分。这与 Java 使用的模型相同，是对象和原始值的混合。

JavaScript 还会根据需要对原始值进行装箱和未装箱。

在这段代码中，您可以看到 JavaScript 原始值的装箱和未装箱版本在工作：

```js
var numberOne = new Number(1);
var numberTwo = 2;
typeof numberOne; //returns 'object'
typeof numberTwo; //returns 'number'
var numberThree = numberOne + numberTwo;
typeof numberThree; //returns 'number'
```

在 JavaScript 中创建对象是微不足道的。可以在这段代码中看到在 JavaScript 中创建对象的过程：

```js
var objectOne = {};
typeof objectOne; //returns 'object'
var objectTwo = new Object();
typeof objectTwo; //returns 'object'
```

因为 JavaScript 是一种动态语言，向对象添加属性也非常容易。甚至可以在创建对象之后进行。这段代码创建了对象：

```js
var objectOne = { value: 7 };
var objectTwo = {};
objectTwo.value = 7;
```

对象包含数据和功能。到目前为止，我们只看到了数据部分。幸运的是，在 JavaScript 中，函数是一等对象。函数可以传递并且函数可以分配给变量。让我们尝试向我们在这段代码中创建的对象添加一些函数：

```js
var functionObject = {};
functionObject.doThings = function() {
  console.log("hello world");
}
functionObject.doThings(); //writes "hello world" to the console
```

这种语法有点痛苦，一次分配一个对象。让我们看看是否可以改进创建对象的语法：

```js
var functionObject = {
  doThings: function() {
    console.log("hello world");
  }
}
functionObject.doThings();//writes "hello world" to the console
```

这种语法看起来，至少对我来说，是一种更清晰、更传统的构建对象的方式。当然，可以以这种方式在对象中混合数据和功能：

```js
var functionObject = {
  greeting: "hello world",
  doThings: function() {
    console.log(this.greeting);
  }
}
functionObject.doThings();//prints hello world
```

在这段代码中有几点需要注意。首先，对象中的不同项使用逗号而不是分号分隔。那些来自其他语言如 C#或 Java 的人可能会犯这个错误。下一个值得注意的是，我们需要使用`this`限定符来从`doThings`函数内部访问`greeting`变量。如果我们在对象中有多个函数，情况也是如此，如下所示：

```js
var functionObject = {
  greeting: "hello world",
  doThings: function() {
    console.log(this.greeting);
    this.doOtherThings();
  },
  doOtherThings: function() {
    console.log(this.greeting.split("").reverse().join(""));
  }
}
functionObject.doThings();//prints hello world then dlrow olleh
```

`this`关键字在 JavaScript 中的行为与您从其他 C 语法语言中所期望的不同。`this`绑定到函数的所有者中。但是，函数的所有者有时并不是您所期望的。在前面的示例中，`this`绑定到`functionObject`对象，但是如果函数在对象之外声明，这将指向全局对象。在某些情况下，通常是事件处理程序，`this`会重新绑定到触发事件的对象。

让我们看一下以下代码：

```js
var target = document.getElementById("someId");
target.addEventListener("click", function() {
  console.log(this);
}, false);
```

`this`采用目标的值。熟悉`this`的值可能是 JavaScript 中最棘手的事情之一。

ECMAScript-2015 引入了`let`关键字，可以替代`var`关键字来声明变量。`let`使用块级作用域，这是大多数语言中常用的作用域。让我们看一个它们之间的例子：

```js
for(var varScoped =0; varScoped <10; varScoped++)
{
  console.log(varScoped);
}
console.log(varScoped +10);
for(let letScoped =0; letScoped<10; letScoped++)
{
  console.log(letScoped);
}
console.log(letScoped+10);
```

使用 var 作用域版本，您可以看到变量在块外继续存在。这是因为在幕后，`varScoped`的声明被提升到代码块的开头。在代码的`let`作用域版本中，`letScoped`仅在`for`循环内部作用域，因此一旦离开循环，`letScoped`就变为未定义。在使用`let`或`var`的选择时，我们倾向于始终使用`let`。有些情况下，您确实希望使用 var 作用域，但这些情况寥寥无几。

我们已经建立了一个相当完整的模型，来展示如何在 JavaScript 中构建对象。但是，对象并不等同于类。对象是类的实例。如果我们想要创建多个`functionObject`对象的实例，我们就没那么幸运了。尝试这样做将导致错误。在 Node.js 的情况下，错误将如下所示：

```js
let obj = new functionObject();
TypeError: object is not a function
  at repl:1:11
  at REPLServer.self.eval (repl.js:110:21)
  at repl.js:249:20
  at REPLServer.self.eval (repl.js:122:7)
  at Interface.<anonymous> (repl.js:239:12)
  at Interface.EventEmitter.emit (events.js:95:17)
  at Interface._onLine (readline.js:202:10)
  at Interface._line (readline.js:531:8)
  at Interface._ttyWrite (readline.js:760:14)
  at ReadStream.onkeypress (readline.js:99:10)
```

这里的堆栈跟踪显示了一个名为`repl`的模块中的错误。这是在启动 Node.js 时默认加载的读取-执行-打印循环。

每次需要一个新实例时，都必须重新构建对象。为了避免这种情况，我们可以使用函数来定义对象，就像这样：

```js
let ThingDoer = function(){
  this.greeting = "hello world";
  this.doThings = function() {
    console.log(this.greeting);
    this.doOtherThings();
  };
  this.doOtherThings = function() {
    console.log(this.greeting.split("").reverse().join(""));
  };
}
let instance = new ThingDoer();
instance.doThings(); //prints hello world then dlrow olleh
```

这种语法允许定义构造函数，并从该函数创建新对象。没有返回值的构造函数是在创建对象时调用的函数。在 JavaScript 中，构造函数实际上返回创建的对象。您甚至可以通过将它们作为初始函数的一部分来使用构造函数来分配内部属性，就像这样：

```js
let ThingDoer = function(greeting){
  this.greeting = greeting;
  this.doThings = function() {
    console.log(this.greeting);
  };
}
let instance = new ThingDoer("hello universe");
instance.doThings();
```

# 给我建立一个原型

如前所述，直到最近，JavaScript 没有支持创建真正的类。虽然 ECMAScript-2015 为类带来了一些语法糖，但底层的对象系统仍然与过去一样，因此看到我们如何在没有这些语法糖的情况下创建对象仍然具有指导意义。使用前一节中的结构创建的对象有一个相当大的缺点：创建多个对象不仅耗时，而且占用内存。以相同方式创建的每个对象都是完全独立的。这意味着用于保存函数定义的内存不会在所有实例之间共享。更有趣的是，您甚至可以重新定义类的单个实例，而不改变所有实例。这在这段代码中得到了证明：

```js
let Castle = function(name){
  this.name = name;
  this.build = function() {
    console.log(this.name);
  };
}
let instance1 = new Castle("Winterfell");
let instance2 = new Castle("Harrenhall");
instance1.build = function(){ console.log("Moat Cailin");}
instance1.build(); //prints "Moat Cailin"
instance2.build(); //prints "Harrenhall" to the console
```

以这种方式改变单个实例的功能，或者实际上是以任何已定义的对象的方式，被称为**monkey** **patching**。人们对这是否是一种良好的做法存在分歧。在处理库代码时，它肯定是有用的，但它会带来很大的混乱。通常认为更好的做法是扩展现有类。

在没有适当的类系统的情况下，JavaScript 当然没有继承的概念。但是，它确实有一个原型。在 JavaScript 中，对象在最基本的层面上是一个键和值的关联数组。对象上的每个属性或函数都简单地定义为这个数组的一部分。您甚至可以通过使用数组语法访问对象的成员来看到这一点，就像这里所示的那样：

```js
let thing = { a: 7};
console.log(thing["a"]);
```

### 提示

使用数组语法访问对象的成员可以是一种非常方便的方法，可以避免使用 eval 函数。例如，如果我有一个名为`funcName`的字符串，我想在对象`obj1`上调用它，那么我可以这样做`obj1[funcName]()`，而不是使用可能危险的 eval 调用。Eval 允许执行任意代码。在页面上允许这样做意味着攻击者可能能够在其他人的浏览器上输入恶意脚本。

当创建对象时，它的定义是从原型继承的。奇怪的是，每个原型也是一个对象，所以甚至原型也有原型。好吧，除了作为顶级原型的对象。将函数附加到原型的优势在于只创建一个函数的副本；节省内存。原型有一些复杂性，但您肯定可以在不了解它们的情况下生存。要使用原型，您只需将函数分配给它，如下所示：

```js
let Castle = function(name){
  this.name = name;
}
Castle.prototype.build = function(){ console.log(this.name);}
let instance1 = new Castle("Winterfell");
instance1.build();
```

需要注意的一件事是只有函数分配给原型。诸如`name`之类的实例变量仍然分配给实例。由于这些对每个实例都是唯一的，因此对内存使用没有真正的影响。

在许多方面，原型语言比基于类的继承模型更强大。

如果以后更改对象的原型，则共享该原型的所有对象都将使用新函数进行更新。这消除了关于猴子打字的一些担忧。此行为的示例如下：

```js
let Castle = function(name){
  this.name = name;
}
Castle.prototype.build = function(){
  console.log(this.name);
}
let instance1 = new Castle("Winterfell");
Castle.prototype.build = function(){
  console.log(this.name.replace("Winterfell", "Moat Cailin"));
}
instance1.build();//prints "Moat Cailin" to the console
```

在构建对象时，您应该确保尽可能利用原型对象。

现在我们知道了原型，JavaScript 中构建对象的另一种方法是使用`Object.create`函数。这是 ECMAScript 5 中引入的新语法。语法如下：

```js
Object.create(prototype [, propertiesObject ] )
```

创建语法将基于给定的原型构建一个新对象。您还可以传递一个`propertiesObject`对象，该对象描述了创建的对象上的附加字段。这些描述符包括许多可选字段：

+   `可写`：这决定了字段是否可写

+   `可配置`：这决定了文件是否应该从对象中移除或在创建后支持进一步配置

+   `可枚举`：这决定了属性在对象属性枚举期间是否可以被列出

+   `值`：这决定了字段的默认值

还可以在描述符中分配`get`和`set`函数，这些函数充当其他内部属性的 getter 和 setter。

使用`object.create`为我们的城堡，我们可以像这样使用`Object.create`构建一个实例：

```js
let instance3 = Object.create(Castle.prototype, {name: { value: "Winterfell", writable: false}});
instance3.build();
instance3.name="Highgarden";
instance3.build();
```

您会注意到我们明确定义了`name`字段。`Object.create`绕过了构造函数，因此我们在前面的代码中描述的初始赋值不会被调用。您可能还注意到`writeable`设置为`false`。其结果是对`name`的重新分配为`Highgarden`没有效果。输出如下：

```js
Winterfell
Winterfell
```

# 继承

对象的一个好处是可以构建更复杂的对象。这是一个常见的模式，用于任何数量的事情。JavaScript 中没有继承，因为它是原型的性质。但是，您可以将一个原型中的函数组合到另一个原型中。

假设我们有一个名为`Castle`的基类，并且我们想将其定制为一个更具体的类`Winterfell`。我们可以通过首先将所有属性从`Castle`原型复制到`Winterfell`原型来实现。可以这样做：

```js
let Castle = function(){};
Castle.prototype.build = function(){console.log("Castle built");}

let Winterfell = function(){};
Winterfell.prototype.build = Castle.prototype.build;
Winterfell.prototype.addGodsWood = function(){}
let winterfell = new Winterfell();
winterfell.build(); //prints "Castle built" to the console
```

当然，这是一种非常痛苦的构建对象的方式。您被迫确切地知道基类有哪些函数来复制它们。可以像这样天真地抽象化：

```js
function clone(source, destination) {
  for(var attr in source.prototype){ destination.prototype[attr] = source.prototype[attr];}
}
```

如果你对对象图表感兴趣，这显示了**Winterfell**在这个图表中如何扩展**Castle**：

![继承](img/Image00004.jpg)

这可以很简单地使用如下：

```js
let Castle = function(){};
Castle.prototype.build = function(){console.log("Castle built");}

let Winterfell = function(){};
clone(Castle, Winterfell);
let winterfell = new Winterfell();
winterfell.build();
```

我们说这是天真的，因为它没有考虑到许多潜在的失败条件。一个完整的实现是相当广泛的。jQuery 库提供了一个名为`extend`的函数，它以健壮的方式实现了原型继承。它大约有 50 行代码，处理深层复制和空值。这个函数在 jQuery 内部被广泛使用，但它也可以成为你自己代码中非常有用的函数。我们提到原型继承比传统的继承方法更强大。这是因为可以从许多基类中混合和匹配位来创建一个新的类。大多数现代语言只支持单一继承：一个类只能有一个直接的父类。有一些语言支持多重继承，然而，这是一种在运行时决定调用哪个版本的方法时增加了很多复杂性的做法。原型继承通过在组装时强制选择方法来避免许多这些问题。

以这种方式组合对象允许从两个或更多不同的基类中获取属性。有许多时候这是很有用的。例如，代表狼的类可能从描述狗的类和描述四足动物的另一个类中获取一些属性。

通过使用以这种方式构建的类，我们可以满足几乎所有构建类系统包括继承的要求。然而，继承是一种非常强的耦合形式。在几乎所有情况下，最好避免继承，而选择一种更松散的耦合形式。这将允许类在对系统的其余部分影响最小的情况下被替换或更改。

# 模块

现在我们有了一个完整的类系统，很好地解决了之前讨论过的全局命名空间问题。同样，JavaScript 没有对命名空间的一流支持，但我们可以很容易地将功能隔离到等同于命名空间的东西中。在 JavaScript 中有许多不同的创建模块的方法。我们将从最简单的开始，并随着进展逐渐添加一些功能。

首先，我们只需要将一个对象附加到全局命名空间。这个对象将包含我们的根命名空间。我们将命名我们的命名空间为`Westeros`；代码看起来就像这样：

```js
Westeros = {}
```

这个对象默认附加到顶层对象，所以我们不需要做更多的事情。一个典型的用法是首先检查对象是否已经存在，然后使用该版本而不是重新分配变量。这允许你将你的定义分散在许多文件中。理论上，你可以在每个文件中定义一个单一的类，然后在交付给客户或在应用程序中使用之前，在构建过程的一部分将它们全部汇集在一起。这个简短的形式是：

```js
Westeros = Westeros || {}
```

一旦我们有了这个对象，只需要将我们的类分配为该对象的属性。如果我们继续使用`Castle`对象，那么它看起来会像这样：

```js
let Westeros = Westeros || {};
Westeros.Castle = function(name){this.name = name}; //constructor
Westeros.Castle.prototype.Build = function(){console.log("Castle built: " +  this.name)};
```

如果我们想要构建一个多于单层深度的命名空间层次结构，也很容易实现，就像这段代码中所示的那样：

```js
let Westeros = Westeros || {};
Westeros.Structures = Westeros.Structures || {};
Westeros.Structures.Castle = function(name){ this.name = name}; //constructor
Westeros.Structures.Castle.prototype.Build = function(){console.log("Castle built: " +  this.name)};
```

这个类可以被实例化并且以类似于之前例子的方式使用：

```js
let winterfell = new Westeros.Structures.Castle("Winterfell");
winterfell.Build();
```

当然，使用 JavaScript 有多种构建相同代码结构的方法。构建前面的代码的一种简单方法是利用创建并立即执行函数的能力：

```js
let Castle = (function () {
  function Castle(name) {
    this.name = name;
  }
  Castle.prototype.Build = function () {
    console.log("Castle built: " + this.name);
  };
  return Castle;
})();
Westros.Structures.Castle = Castle;
```

这段代码似乎比之前的代码示例要长一些，但由于其分层性质，我觉得更容易理解。我们可以像前面的代码中所示的那样，在相同的结构中使用它们来创建一个新的城堡：

```js
let winterfell = new Westeros.Structures.Castle("Winterfell");
winterfell.Build();
```

使用这种结构进行继承也相对容易。如果我们定义了一个`BaseStructure`类，它是所有结构的祖先，那么使用它会像这样：

```js
let BaseStructure = (function () {
  function BaseStructure() {
  }
  return BaseStructure;
})();
Structures.BaseStructure = BaseStructure;
let Castle = (function (_super) {
  **__extends(Castle, _super);** 

  function Castle(name) {
    this.name = name;
    _super.call(this);
  }
  Castle.prototype.Build = function () {
    console.log("Castle built: " + this.name);
  };
  return Castle;
})(BaseStructure);
```

您会注意到，当闭包被评估时，基本结构被传递到`Castle`对象中。代码中的高亮行使用了一个叫做`__extends`的辅助方法。这个方法负责将函数从基本原型复制到派生类中。这段特定的代码是由 TypeScript 编译器生成的，它还生成了一个看起来像这样的`extends`方法：

```js
let __extends = this.__extends || function (d, b) {
  for (var p in b) if (b.hasOwnProperty(p)) d[p] = b[p];
  function __() { this.constructor = d; }
  __.prototype = b.prototype;
  d.prototype = new __();
};
```

我们可以继续使用我们采用的相当巧妙的闭包语法来实现整个模块。如下所示：

```js
var Westeros;
(function (Westeros) {
  (function (Structures) {
    let Castle = (function () {
      function Castle(name) {
        this.name = name;
      }
       Castle.prototype.Build = function () {
         console.log("Castle built " + this.name);
       };
       return Castle;
     })();
     Structures.Castle = Castle;
  })(Westeros.Structures || (Westeros.Structures = {}));
  var Structures = Westeros.Structures;
})(Westeros || (Westeros = {}));
```

在这个结构中，您可以看到我们之前探讨过的创建模块的相同代码。在单个模块中定义多个类也相对容易。这可以在这段代码中看到：

```js
var Westeros;
(function (Westeros) {
  (function (Structures) {
    let Castle = (function () {
      function Castle(name) {
        this.name = name;
      }
      Castle.prototype.Build = function () {
        console.log("Castle built: " + this.name);
        var w = new Wall();
      };
      return Castle;
    })();
    Structures.Castle = Castle;
 **var Wall = (function () {** 

 **function Wall() {** 

 **console.log("Wall constructed");** 

 **}** 

 **return Wall;** 

 **})();** 

 **Structures.Wall = Wall;** 

  })(Westeros.Structures || (Westeros.Structures = {}));
  var Structures = Westeros.Structures;
})(Westeros || (Westeros = {}));
```

高亮代码在模块内创建了第二个类。在每个文件中定义一个类也是完全允许的。因为代码在盲目重新分配之前检查`Westeros`的当前值，所以我们可以安全地将模块定义分割成多个文件。

高亮部分的最后一行显示了在闭包之外暴露类。如果我们想要创建只在模块内部可用的私有类，那么我们只需要排除那一行。这实际上被称为揭示模块模式。我们只暴露需要全局可用的类。尽可能将功能保持在全局命名空间之外是一个很好的做法。

# ECMAScript 2015 类和模块

到目前为止，我们已经看到在 ECMAScript-2015 之前的 JavaScript 中完全可以构建类甚至模块。显然，这种语法比如 C#或 Java 等语言更复杂。幸运的是，ECMAScript-2015 为创建类提供了一些语法糖的支持：

```js
class Castle extends Westeros.Structures.BaseStructure {
  constructor(name, allegience) {
    super(name);
    ...
  }
  Build() {
    ...
    super.Build();
  }
}
```

ECMAScript-2015 还为 JavaScript 带来了一个经过深思熟虑的模块系统。还有一些用于创建模块的语法糖，看起来像这样：

```js
module 'Westeros' {
  export function Rule(rulerName, house) {
    ...
    return "Long live " + rulerName + " of house " + house;
  }
}
```

由于模块可以包含函数，当然也可以包含类。ECMAScript-2015 还定义了模块导入语法和支持从远程位置检索模块。导入模块看起来像这样：

```js
import westeros from 'Westeros';
module JSON from 'http://json.org/modules/json2.js';
westeros.Rule("Rob Stark", "Stark");
```

一些这种语法糖在任何支持完整 ECMAScript-2015 的环境中都是可用的。在撰写本文时，所有主要浏览器供应商对 ECMAScript-2015 的类部分都有很好的支持，所以几乎没有理由不使用它，除非你不得不支持古老的浏览器。

# 最佳实践和故障排除

在理想的世界中，每个人都可以在从一开始就制定标准的绿地项目上工作。然而，情况并非如此。经常情况下，你可能会发现自己处于一个遗留系统的一部分，其中有一堆非模块化的 JavaScript 代码。

在这些情况下，简单地忽略非模块化的代码直到真正需要升级它可能是有利的。尽管 JavaScript 很受欢迎，但 JavaScript 的许多工具仍然不够成熟，这使得很难依赖编译器来找出 JavaScript 重构引入的错误。自动重构工具也受到 JavaScript 动态特性的复杂性的影响。然而，对于新代码，正确使用模块化的 JavaScript 可以非常有助于避免命名空间冲突并提高可测试性。

如何安排 JavaScript 是一个有趣的问题。从网页的角度来看，我采取了将 JavaScript 与网页保持一致的方法。因此，每个页面都有一个关联的 JavaScript 文件，负责该页面的功能。此外，页面之间共同的组件，比如网格控件，被放置在一个单独的文件中。在编译时，所有文件都被合并成一个单独的 JavaScript 文件。这有助于在保持小型代码文件的同时减少浏览器向服务器发出的请求次数。

# 总结

有人说计算机科学中只有两件真正困难的事情。这些问题的具体内容因说话者而异。经常是一些与缓存失效和命名有关的变体。如何组织代码是其中很大一部分的命名问题。

作为一个团体，我们似乎已经坚定地接受了命名空间和类的概念。正如我们所见，JavaScript 中没有直接支持这两个概念。然而，有无数种方法可以解决这个问题，其中一些方法实际上提供的功能比传统的命名空间/类系统更强大。

JavaScript 的主要问题是要避免用大量名称相似但不相关的对象污染全局命名空间。将 JavaScript 封装成模块是朝着编写可维护和可重用代码的关键步骤。

随着我们的前进，我们会看到许多复杂的接口排列在 JavaScript 的世界中变得更加简单。原型继承，起初似乎很困难，但它是简化设计模式的巨大工具。

# 第三章：创建模式

在上一章中，我们详细研究了如何构建一个类。在本章中，我们将研究如何创建类的实例。表面上看，这似乎是一个简单的问题，但我们如何创建类的实例可能非常重要。

我们非常努力地创建我们的代码，使其尽可能解耦。确保类对其他类的依赖最小是构建一个可以随着使用软件的人的需求变化而流畅变化的系统的关键。允许类之间关系过于紧密意味着变化会像涟漪一样在它们之间传播。

一个涟漪并不是一个巨大的问题，但随着你不断引入更多的变化，涟漪会累积并产生干涉模式。很快，曾经平静的表面就变成了无法辨认的添加和破坏节点的混乱。我们的应用程序中也会出现同样的问题：变化会放大并以意想不到的方式相互作用。我们经常忽视耦合的一个地方就是在对象的创建中：

```js
let Westeros;
(function (Westeros) {
  let Ruler = (function () {
    function Ruler() {
      this.house = new Westeros.Houses.Targaryen();
    }
    return Ruler;
  })();
  Westeros.Ruler = Ruler;
})(Westeros || (Westeros = {}));
```

在这个类中，你可以看到统治者的家与`Targaryen`类紧密耦合。如果这种情况发生改变，那么这种紧密耦合就必须在很多地方进行改变。本章讨论了一些模式，这些模式最初是在《设计模式：可复用面向对象软件的元素》一书中提出的。这些模式的目标是改善应用程序中的耦合程度，并增加代码重用的机会。这些模式如下：

+   抽象工厂

+   建造者

+   工厂方法

+   单例

+   原型

当然，并非所有这些都适用于 JavaScript，但随着我们逐步了解创建模式，我们会了解到这一切。

# 抽象工厂

这里介绍的第一个模式是一种创建对象套件的方法，而不需要知道对象的具体类型。让我们继续使用前一节中介绍的统治王国的系统。

对于所讨论的王国，统治家族的更换频率相当高。很可能在更换家族时会有一定程度的战斗和斗争，但我们暂且不予理会。每个家族都会以不同的方式统治王国。有些人看重和平与宁静，以仁慈的领导者统治，而另一些则以铁腕统治。一个王国的统治对于一个人来说太大了，所以国王会将一些决定交给一个叫做国王之手的副手。国王也会在一些事务上得到一个由一些精明的领主和贵妇组成的议会的建议。

我们描述的类的图表如下：

![抽象工厂](img/Image00005.jpg)

### 提示

**统一建模语言**（**UML**）是由对象管理组开发的标准化语言，用于描述计算机系统。该语言中有用于创建用户交互图、序列图和状态机等的词汇。对于本书的目的，我们最感兴趣的是类图，它描述了一组类之间的关系。

整个 UML 类图词汇量很大，超出了本书的范围。然而，维基百科上的文章[`en.wikipedia.org/wiki/Class_diagram`](https://en.wikipedia.org/wiki/Class_diagram)以及 Derek Banas 的优秀视频教程[`www.youtube.com/watch?v=3cmzqZzwNDM`](https://www.youtube.com/watch?v=3cmzqZzwNDM)都是很好的介绍。

问题在于，由于统治家族甚至统治家族成员经常变动，与 Targaryen 或 Lannister 等具体家族耦合会使我们的应用程序变得脆弱。脆弱的应用程序在不断变化的世界中表现不佳。

解决这个问题的方法是利用抽象工厂模式。抽象工厂声明了一个接口，用于创建与统治家族相关的各种类。

这种模式的类图相当令人生畏：

![抽象工厂](img/Image00006.jpg)

抽象工厂类可能对统治家族的各个实现有多个。这些被称为具体工厂，它们每个都将实现抽象工厂提供的接口。具体工厂将返回各种统治类的具体实现。这些具体类被称为产品。

让我们首先看一下抽象工厂接口的代码。

没有代码？实际上确实如此。JavaScript 的动态特性消除了描述类所需的接口的需要。我们将直接创建类，而不是使用接口：

![抽象工厂](img/Image00007.jpg)

JavaScript 不使用接口，而是相信您提供的类实现了所有适当的方法。在运行时，解释器将尝试调用您请求的方法，并且如果找到该方法，就会调用它。解释器只是假设如果您的类实现了该方法，那么它就是该类。这就是所谓的**鸭子类型**。

### 注意

**鸭子类型**

鸭子类型的名称来源于 Alex Martelli 在 2000 年发布的一篇文章，他在*comp.lang.python*新闻组中写道：

*换句话说，不要检查它是否是一只鸭子：检查它是否像一只鸭子一样嘎嘎叫，像一只鸭子一样走路，等等，具体取决于您需要用来玩语言游戏的鸭子行为的子集。*

我喜欢 Martelli 可能是从*Monty Python and the Holy Grail*的巫师狩猎片段中借用了这个术语。虽然我找不到任何证据，但我认为这很可能，因为 Python 编程语言的名称就来自于 Monty Python。

鸭子类型是动态语言中的强大工具，可以大大减少实现类层次结构的开销。然而，它确实引入了一些不确定性。如果两个类实现了具有根本不同含义的同名方法，那么就无法知道调用的是正确的方法。例如，考虑以下代码：

```js
class Boxer{
  function punch(){}
}
class TicketMachine{
  function punch(){}
}
```

这两个类都有一个`punch()`方法，但显然意义不同。JavaScript 解释器不知道它们是不同的类，并且会愉快地在任何一个类上调用 punch，即使其中一个没有意义。

一些动态语言支持一种通用方法，当调用未定义的方法时会调用该方法。例如，Ruby 有`missing_method`，在许多情况下都被证明非常有用。截至目前，JavaScript 目前不支持`missing_method`。然而，ECMAScript 2016，即 ECMAScript 2015 的后续版本，定义了一个称为`Proxy`的新构造，它将支持动态包装对象，借助它可以实现一个等价的`missing_method`。

## 实现

为了演示抽象工厂的实现，我们首先需要一个`King`类的实现。这段代码提供了该实现：

```js
let KingJoffery= (function () {
  function KingJoffery() {
  }
  KingJoffery.prototype.makeDecision = function () {
    …
  };
  KingJoffery.prototype.marry = function () {
    …
  };
  return KingJoffery;
})();
```

### 注意

这段代码不包括第二章*组织代码*中建议的模块结构。在每个示例中包含模块代码是乏味的，你们都是聪明的人，所以如果你们要真正使用它，就知道把它放在模块中。完全模块化的代码可以在分发的源代码中找到。

这只是一个普通的具体类，实际上可以包含任何实现细节。我们还需要一个同样不起眼的`HandOfTheKing`类的实现：

```js
let LordTywin = (function () {
  function LordTywin() {
  }
  LordTywin.prototype.makeDecision = function () {
  };
  return LordTywin;
})();
```

具体的工厂方法如下：

```js
let LannisterFactory = (function () {
  function LannisterFactory() {
  }
  LannisterFactory.prototype.getKing = function () {
    return new KingJoffery();
  };
  LannisterFactory.prototype.getHandOfTheKing = function ()
  {
    return new LordTywin();
  };
  return LannisterFactory;
})();
```

这段代码只是实例化所需类的新实例并返回它们。不同统治家族的另一种实现将遵循相同的一般形式，可能如下所示：

```js
let TargaryenFactory = (function () {
  function TargaryenFactory() {
  }
  TargaryenFactory.prototype.getKing = function () {
    return new KingAerys();
  };
  TargaryenFactory.prototype.getHandOfTheKing = function () {
    return new LordConnington();
  };
  return TargaryenFactory;
})();
```

在 JavaScript 中实现抽象工厂比其他语言要容易得多。然而，这样做的代价是失去了编译器检查，它强制要求对工厂或产品进行完整的实现。随着我们继续学习其他模式，你会注意到这是一个常见的主题。在静态类型语言中有很多管道的模式要简单得多，但会增加运行时失败的风险。适当的单元测试或 JavaScript 编译器可以缓解这种情况。

要使用抽象工厂，我们首先需要一个需要使用某个统治家族的类：

```js
let CourtSession = (function () {
  function CourtSession(abstractFactory) {
    this.abstractFactory = abstractFactory;
    this.COMPLAINT_THRESHOLD = 10;
  }
  CourtSession.prototype.complaintPresented = function (complaint) {
    if (complaint.severity < this.COMPLAINT_THRESHOLD) {
      this.abstractFactory.getHandOfTheKing().makeDecision();
    } else
    this.abstractFactory.getKing().makeDecision();
  };
  return CourtSession;
})();
```

现在我们可以调用这个`CourtSession`类，并根据传入的工厂注入不同的功能：

```js
let courtSession1 = new CourtSession(new TargaryenFactory());
courtSession1.complaintPresented({ severity: 8 });
courtSession1.complaintPresented({ severity: 12 });

let courtSession2 = new CourtSession(new LannisterFactory());
courtSession2.complaintPresented({ severity: 8 });
courtSession2.complaintPresented({ severity: 12 });
```

尽管静态语言和 JavaScript 之间存在差异，但这种模式在 JavaScript 应用程序中仍然适用且有用。创建一组共同工作的对象对于许多情况都是有用的；每当一组对象需要协作提供功能但可能需要整体替换时。当试图确保一组对象一起使用而不进行替换时，这也可能是一个有用的模式。

# 建造者

在我们的虚构世界中，有时需要构建一些相当复杂的类。这些类包含了根据构建方式不同的接口实现。为了简化这些类的构建并将构建类的知识封装在消费者之外，可以使用建造者。多个具体建造者降低了实现中构造函数的复杂性。当需要新的建造者时，不需要添加构造函数，只需要插入一个新的建造者。

锦标赛是一个复杂类的例子。每个锦标赛都有一个复杂的设置，涉及事件、参与者和奖品。这些锦标赛的大部分设置都是相似的：每一个都有比武、射箭和混战。从代码中的多个位置创建锦标赛意味着构建锦标赛的责任被分散。如果需要更改初始化代码，那么必须在许多不同的地方进行更改。

通过使用构建器模式，可以避免这个问题，因为它集中了构建对象所需的逻辑。不同的具体构建器可以插入到构建器中，以构建不同的复杂对象。构建器模式中各个类之间的关系如下所示：

![构建器](img/Image00008.jpg)

## 实施

让我们进去看一些代码。首先，我们将创建一些实用类，它们将表示比赛的各个部分，如下面的代码所示：

```js
let Event = (function () {
  function Event(name) {
    this.name = name;
  }
  return Event;
})();
Westeros.Event = Event;

let Prize = (function () {
  function Prize(name) {
    this.name = name;
  }
  return Prize;
})();
Westeros.Prize = Prize;

let Attendee = (function () {
  function Attendee(name) {
    this.name = name;
  }
  return Attendee;
})();
Westeros.Attendee = Attendee;
```

比赛本身是一个非常简单的类，因为我们不需要显式地分配任何公共属性：

```js
let Tournament = (function () {
  this.Events = [];
  function Tournament() {
  }
  return Tournament;
})();
Westeros.Tournament = Tournament;
```

我们将实现两个创建不同比赛的构建器。下面的代码中可以看到：

```js
let LannisterTournamentBuilder = (function () {
  function LannisterTournamentBuilder() {
  }
  LannisterTournamentBuilder.prototype.build = function () {
    var tournament = new Tournament();
    tournament.events.push(new Event("Joust"));
    tournament.events.push(new Event("Melee"));
    tournament.attendees.push(new Attendee("Jamie"));
    tournament.prizes.push(new Prize("Gold"));
    tournament.prizes.push(new Prize("More Gold"));
    return tournament;
  };
  return LannisterTournamentBuilder;
})();
Westeros.LannisterTournamentBuilder = LannisterTournamentBuilder;

let BaratheonTournamentBuilder = (function () {
  function BaratheonTournamentBuilder() {
  }
  BaratheonTournamentBuilder.prototype.build = function () {
    let tournament = new Tournament();
    tournament.events.push(new Event("Joust"));
    tournament.events.push(new Event("Melee"));
    tournament.attendees.push(new Attendee("Stannis"));
    tournament.attendees.push(new Attendee("Robert"));
    return tournament;
  };
  return BaratheonTournamentBuilder;
})();
Westeros.BaratheonTournamentBuilder = BaratheonTournamentBuilder;
```

最后，导演，或者我们称之为`TournamentBuilder`，只需拿起一个构建器并执行它：

```js
let TournamentBuilder = (function () {
  function TournamentBuilder() {
  }
  TournamentBuilder.prototype.build = function (builder) {
    return builder.build();
  };
  return TournamentBuilder;
})();
Westeros.TournamentBuilder = TournamentBuilder;
```

再次，您会看到 JavaScript 的实现比传统的实现要简单得多，因为不需要接口。

构建器不需要返回一个完全实现的对象。这意味着您可以创建一个部分填充对象的构建器，然后允许对象传递给另一个构建器来完成。一个很好的现实世界类比可能是汽车的制造过程。在装配线上的每个工位都只组装汽车的一部分，然后将其传递给下一个工位组装另一部分。这种方法允许将构建对象的工作分配给几个具有有限责任的类。在我们上面的例子中，我们可以有一个负责填充事件的构建器，另一个负责填充参与者的构建器。

在 JavaScript 的原型扩展模型中，构建器模式是否仍然有意义？我认为是的。仍然存在需要根据不同的方法创建复杂对象的情况。

# 工厂方法

我们已经看过抽象工厂和构建器。抽象工厂构建了一组相关的类，而构建器使用不同的策略创建复杂对象。工厂方法模式允许类请求接口的新实例，而不是类决定使用接口的哪个实现。工厂可能使用某种策略来选择要返回的实现：

![工厂方法](img/Image00009.jpg)

有时，这种策略只是接受一个字符串参数或检查一些全局设置来充当开关。

## 实施

在我们的 Westworld 示例世界中，有很多时候我们希望将实现的选择推迟到工厂。就像现实世界一样，Westworld 拥有丰富多彩的宗教文化，有数十种不同的宗教崇拜各种各样的神。在每种宗教中祈祷时，必须遵循不同的规则。有些宗教要求献祭，而其他宗教只要求给予礼物。祈祷类不想知道所有不同的宗教以及如何构建它们。

让我们开始创建一些不同的神，可以向他们献祷。这段代码创建了三个神，包括一个默认的神，如果没有指定其他神，祷告就会落在他身上：

```js
let WateryGod = (function () {
  function WateryGod() {
  }
  WateryGod.prototype.prayTo = function () {
  };
  return WateryGod;
})();
Religion.WateryGod = WateryGod;
let AncientGods = (function () {
  function AncientGods() {
  }
  AncientGods.prototype.prayTo = function () {
  };
  return AncientGods;
})();
Religion.AncientGods = AncientGods;

let DefaultGod = (function () {
  function DefaultGod() {
  }
  DefaultGod.prototype.prayTo = function () {
  };
  return DefaultGod;
})();
Religion.DefaultGod = DefaultGod;
```

我避免了为每个神明的任何实现细节。您可以想象任何您想要填充`prayTo`方法的传统。也没有必要确保每个神都实现了`IGod`接口。接下来，我们需要一个工厂，负责构建不同的神：

```js
let GodFactory = (function () {
  function GodFactory() {
  }
  GodFactory.Build = function (godName) {
    if (godName === "watery")
      return new WateryGod();
    if (godName === "ancient")
      return new AncientGods();
    return new DefaultGod();
  };
  return GodFactory;
})();
```

您可以看到，在这个例子中，我们接受一个简单的字符串来决定如何创建一个神。它可以通过全局或更复杂的对象来完成。在 Westeros 的一些多神教中，神明有明确定的角色，如勇气之神、美丽之神或其他方面的神。必须祈祷的神不仅由宗教决定，还由祈祷的目的决定。我们可以用`GodDeterminant`类来表示这一点，如下所示：

```js
let GodDeterminant = (function () {
  function GodDeterminant(religionName, prayerPurpose) {
    this.religionName = religionName;
    this.prayerPurpose = prayerPurpose;
  }
  return GodDeterminant;
})();
```

工厂将被更新以接受这个类，而不是简单的字符串。

最后，最后一步是看看这个工厂将如何被使用。这很简单，我们只需要传入一个表示我们希望观察的宗教的字符串，工厂将构造正确的神并返回它。这段代码演示了如何调用工厂：

```js
let Prayer = (function () {
  function Prayer() {
  }
  Prayer.prototype.pray = function (godName) {
  GodFactory.Build(godName).prayTo();
  };
  return Prayer;
})();
```

再次，JavaScript 中肯定需要这样的模式。有很多时候，将实例化与使用分开是有用的。由于关注点的分离和注入假工厂以允许测试`Prayer`也很容易，测试实例化也非常简单。

继续创建不带接口的更简单模式的趋势，我们可以忽略模式的接口部分，直接使用类型，这要归功于鸭子类型。

工厂方法是一种非常有用的模式：它允许类将实例化的实现选择推迟到另一个类。当存在多个类似的实现时，这种模式非常有用，比如策略模式（参见第五章 ，*行为模式*），并且通常与抽象工厂模式一起使用。工厂方法用于在抽象工厂的具体实现中构建具体对象。抽象工厂模式可能包含多个工厂方法。工厂方法无疑是一种在 JavaScript 领域仍然适用的模式。

# 单例

单例模式可能是最常被滥用的模式。它也是近年来不受青睐的模式。为了看到为什么人们开始建议不要使用单例模式，让我们看看这个模式是如何工作的。

当需要全局变量时使用单例是可取的，但单例提供了防止意外创建复杂对象的保护。它还允许推迟对象实例化直到第一次使用。

单例的 UML 图如下所示：

![单例](img/Image00010.jpg)

这显然是一种非常简单的模式。单例充当类的实例的包装器，单例本身作为全局变量存在。在访问实例时，我们只需向单例请求包装类的当前实例。如果类在单例中尚不存在，通常会在那时创建一个新实例。

## 实施

在我们在维斯特洛大陆的持续示例中，我们需要找到一个只能有一个东西的情况。不幸的是，这是一个经常发生冲突和敌对的土地，所以我最初想到的将国王作为单例的想法根本行不通。这也意味着我们不能利用其他明显的候选人（首都，王后，将军等）。然而，在维斯特洛大陆的最北部，有一堵巨大的墙，用来阻挡一位古老的敌人。这样的墙只有一堵，将其放在全局范围内应该没有问题。

让我们继续在 JavaScript 中创建一个单例：

```js
let Westros;
(function (Westeros) {
  var Wall = (function () {
 **function Wall() {** 

 **this.height = 0;** 

 **if (Wall._instance)** 

 **return Wall._instance;** 

 **Wall._instance = this;** 

 **}** 

    Wall.prototype.setHeight = function (height) {
      this.height = height;
    };
    Wall.prototype.getStatus = function () {
      console.log("Wall is " + this.height + " meters tall");
    };
 **Wall.getInstance = function () {** 

 **if (!Wall._instance) {** 

 **Wall._instance = new Wall();** 

 **}** 

 **return Wall._instance;** 

 **};** 

    Wall._instance = null;
    return Wall;
  })();
  Westeros.Wall = Wall;
})(Westeros || (Westeros = {}));
```

该代码创建了墙的轻量级表示。单例在两个突出显示的部分中进行了演示。在像 C#或 Java 这样的语言中，我们通常会将构造函数设置为私有，以便只能通过静态方法`getInstance`来调用它。然而，在 JavaScript 中我们没有这个能力：构造函数不能是私有的。因此，我们尽力而为，从构造函数中返回当前实例。这可能看起来很奇怪，但在我们构造类的方式中，构造函数与任何其他方法没有区别，因此可以从中返回一些东西。

在第二个突出部分中，我们将静态变量`_instance`设置为 Wall 的新实例（如果还没有）。如果`_instance`已经存在，我们将返回它。在 C#和 Java 中，这个函数需要一些复杂的锁定逻辑，以避免两个不同的线程同时尝试访问实例时出现竞争条件。幸运的是，在 JavaScript 中不需要担心这个问题，因为多线程的情况不同。

## 缺点

单例在过去几年中声名狼藉。它们实际上是被吹捧的全局变量。正如我们讨论过的，全局变量是不合理的，可能导致许多错误。它们也很难通过单元测试进行测试，因为实例的创建不能轻易被覆盖，测试运行器中的任何并行性都可能引入难以诊断的竞争条件。我对它们最大的担忧是单例承担了太多的责任。它们不仅控制自己，还控制它们的实例化。这是对单一责任原则的明显违反。几乎每一个可以使用单例解决的问题，都可以通过其他机制更好地解决。

JavaScript 使问题变得更糟。由于构造函数的限制，无法创建单例的清晰实现。这与单例的一般问题结合在一起，使我建议在 JavaScript 中应避免使用单例模式。

# 原型

本章中的最后一个创建模式是原型模式。也许这个名字听起来很熟悉。它确实应该：这是 JavaScript 支持继承的机制。

我们研究了用于继承的原型，但原型的适用性不一定局限于继承。复制现有对象可以是一个非常有用的模式。有许多情况下，能够复制构造对象是很方便的。例如，通过保存利用某种克隆创建的先前实例，很容易地维护对象状态的历史。

## 实施

在维斯特洛，我们发现家庭成员经常非常相似；正如谚语所说：“有其父必有其子”。随着每一代的诞生，通过复制和修改现有家庭成员来创建新一代比从头开始建造要容易得多。

在第二章中，*组织代码*，我们看了如何复制现有对象，并介绍了一个非常简单的克隆代码：

```js
function clone(source, destination) {
  for(var attr in source.prototype){
    destination.prototype[attr] = source.prototype[attr];}
}
```

这段代码可以很容易地改变，以便在类内部使用，返回自身的副本：

```js
var Westeros;
(function (Westeros) {
  (function (Families) {
    var Lannister = (function () {
      function Lannister() {
      }
      **Lannister.prototype.clone = function () {** 

 **var clone = new Lannister();** 

 **for (var attr in this) {** 

 **clone[attr] = this[attr];** 

 **}** 

 **return clone;** 

 **};** 

      return Lannister;
    })();
    Families.Lannister = Lannister;
  })(Westeros.Families || (Westeros.Families = {}));
  var Families = Westeros.Families;
})(Westeros || (Westeros = {}));
```

代码的突出部分是修改后的克隆方法。它可以这样使用：

```js
let jamie = new Westeros.Families.Lannister();
jamie.swordSkills = 9;
jamie.charm = 6;
jamie.wealth = 10;

let tyrion = jamie.clone();
tyrion.charm = 10;
//tyrion.wealth == 10
//tyrion.swordSkill == 9
```

原型模式允许只构造一次复杂对象，然后克隆成任意数量的仅略有不同的对象。如果源对象不复杂，那么采用克隆方法就没有太多好处。在使用原型方法时，必须注意依赖对象。克隆是否应该是深层的？

原型显然是一个有用的模式，也是 JavaScript 从一开始就形成的一个组成部分。因此，它肯定是任何规模可观的 JavaScript 应用程序中会看到一些使用的模式。

# 提示和技巧

创建模式允许在创建对象时实现特定行为。在许多情况下，比如工厂，它们提供了可以放置横切逻辑的扩展点。也就是说，适用于许多不同类型对象的逻辑。如果你想要在整个应用程序中注入日志，那么能够连接到工厂是非常有用的。

尽管这些创建模式非常有用，但不应该经常使用。您的大部分对象实例化仍应该是改进对象的正常方法。虽然当你有了新的工具时，把一切都视为钉子是很诱人的，但事实是每种情况都需要有一个具体的策略。所有这些模式都比简单使用`new`更复杂，而复杂的代码更容易出现错误。尽量使用`new`。

# 总结

本章介绍了创建对象的多种不同策略。这些方法提供了对创建对象的典型方法的抽象。抽象工厂提供了构建可互换的工具包或相关对象集合的方法。建造者模式提供了解决参数问题的解决方案。它使得构建大型复杂对象变得更加容易。工厂方法是抽象工厂的有用补充，允许通过静态工厂创建不同的实现。单例是一种提供整个解决方案可用的类的单个副本的模式。这是迄今为止我们所见过的唯一一个在现代软件中存在一些适用性问题的模式。原型模式是 JavaScript 中常用的一种模式，用于基于其他现有对象构建对象。

我们将在下一章继续对经典设计模式进行考察，重点关注结构模式。

# 第四章：结构模式

在上一章中，我们探讨了多种创建对象的方法，以便优化重用。在本章中，我们将研究结构模式；这些模式关注于通过描述对象可以相互交互的简单方式来简化设计。

再次，我们将限制自己只研究 GoF 书中描述的模式。自 GoF 出版以来，已经确定了许多其他有趣的结构模式，我们将在本书的第二部分中进行研究。

我们将在这里研究的模式有：

+   适配器

+   桥接

+   组合

+   装饰器

+   外观

+   享元

+   代理

我们将再次讨论多年前描述的模式是否仍然适用于不同的语言和不同的时代。

# 适配器

有时需要将圆销子放入方孔中。如果你曾经玩过儿童的形状分类玩具，你可能会发现实际上可以把圆销子放入方孔中。孔并没有完全填满，把销子放进去可能会很困难：

![适配器](img/Image00011.jpg)

为了改善销子的适配，可以使用适配器。这个适配器完全填满了孔，结果非常完美：

![适配器](img/Image00012.jpg)

在软件中经常需要类似的方法。我们可能需要使用一个不完全符合所需接口的类。该类可能缺少方法，或者可能有我们希望隐藏的额外方法。在处理第三方代码时经常会出现这种情况。为了使其符合您代码中所需的接口，可能需要使用适配器。

适配器的类图非常简单，如下所示：

![适配器](img/Image00013.jpg)

实现的接口看起来不符合我们在代码中想要的样子。通常的解决方法是简单地重构实现，使其看起来符合我们的要求。然而，有一些可能的原因无法这样做。也许实现存在于我们无法访问的第三方代码中。还有可能实现在应用程序的其他地方使用，接口正好符合我们的要求。

适配器类是一个薄薄的代码片段，实现所需的接口。它通常包装实现类的私有副本，并通过代理调用它。适配器模式经常用于改变代码的抽象级别。让我们来看一个快速的例子。

## 实施

在 Westeros 的土地上，许多贸易和旅行都是通过船只进行的。乘船旅行不仅比步行或骑马更危险，而且由于风暴和海盗的不断出现，也更加危险。这些船只不是皇家加勒比公司用来在加勒比海周游的那种船只；它们是粗糙的东西，看起来更像是 15 世纪欧洲探险家所驾驶的。

虽然我知道船只存在，但我对它们的工作原理或如何操纵船只几乎一无所知。我想很多人和我一样。如果我们看看 Westeros 的船只接口，它看起来很吓人：

```js
interface Ship{
  SetRudderAngleTo(angle: number);
  SetSailConfiguration(configuration: SailConfiguration);
  SetSailAngle(sailId: number, sailAngle: number);
  GetCurrentBearing(): number;
  GetCurrentSpeedEstimate(): number;
  ShiftCrewWeightTo(weightToShift: number, locationId: number);
}
```

我真的希望有一个更简单的接口，可以抽象掉所有繁琐的细节。理想情况下是这样的：

```js
interface SimpleShip{
  TurnLeft();
  TurnRight();
  GoForward();
}
```

这看起来像是我可能会弄清楚的东西，即使我住在离最近的海洋有 1000 公里远的城市。简而言之，我想要的是对船只进行更高级别的抽象。为了将船只转换为 SimpleShip，我们需要一个适配器。

适配器将具有 SimpleShip 的接口，但它将在 Ship 的包装实例上执行操作。代码可能看起来像这样：

```js
let ShipAdapter = (function () {
  function ShipAdapter() {
    this._ship = new Ship();
  }
  ShipAdapter.prototype.TurnLeft = function () {
    this._ship.SetRudderAngleTo(-30);
    this._ship.SetSailAngle(3, 12);
  };
  ShipAdapter.prototype.TurnRight = function () {
    this._ship.SetRudderAngleTo(30);
    this._ship.SetSailAngle(5, -9);
  };
  ShipAdapter.prototype.GoForward = function () {
    //do something else to the _ship
  };
  return ShipAdapter;
})();
```

实际上，这些功能会更加复杂，但这并不重要，因为我们有一个简单的接口来展示给世界。所呈现的接口也可以设置为限制对基础类型的某些方法的访问。在构建库代码时，适配器可用于隐藏内部方法，只向最终用户呈现所需的有限功能。

使用这种模式，代码可能看起来像这样：

```js
var ship = new ShipAdapter();
ship.GoForward();
ship.TurnLeft();
```

你可能不想在客户端类的名称中使用适配器，因为它泄露了一些关于底层实现的信息。客户端不应该知道它们正在与适配器交谈。

适配器本身可能会变得非常复杂，以调整一个接口到另一个接口。为了避免创建非常复杂的适配器，必须小心。构建几个适配器是完全可以想象的，一个在另一个之上。如果发现适配器变得太大，那么最好停下来检查适配器是否遵循单一责任原则。也就是说，确保每个类只负责一件事。一个从数据库中查找用户的类不应该包含向这些用户发送电子邮件的功能。这责任太大了。复杂的适配器可以被复合对象替换，这将在本章后面探讨。

从测试的角度来看，适配器可以用来完全包装第三方依赖。在这种情况下，它们提供了一个可以挂接测试的地方。单元测试应该避免测试库，但它们可以确保适配器代理了正确的调用。

适配器是简化代码接口的非常强大的模式。调整接口以更好地满足需求在无数地方都是有用的。这种模式在 JavaScript 中肯定很有用。用 JavaScript 编写的应用程序往往会使用大量的小型库。通过将这些库封装在适配器中，我能够限制我直接与库交互的地方的数量；这意味着可以轻松替换这些库。

适配器模式可以稍微修改，以在许多不同的实现上提供一致的接口。这通常被称为桥接模式。

# 桥接

桥梁模式将适配器模式提升到一个新的水平。给定一个接口，我们可以构建多个适配器，每个适配器都充当到不同实现的中介。

我遇到的一个很好的例子是，处理两个提供几乎相同功能并且在故障转移配置中使用的不同服务。两个服务都没有提供应用程序所需的确切接口，并且两个服务提供不同的 API。为了简化代码，编写适配器以提供一致的接口。适配器实现一致的接口并提供填充，以便可以一致地调用每个 API。再举一个形状分类器的比喻，我们可以想象我们有各种不同的销子，我们想用它们来填充方形孔。每个适配器填补了缺失的部分，并帮助我们得到一个良好的适配：

![Bridge](img/Image00014.jpg)

桥梁是一个非常有用的模式。让我们来看看如何实现它：

![Bridge](img/Image00015.jpg)

在前面的图表中显示的适配器位于实现和所需接口之间。它们修改实现以适应所需的接口。

## 实现

我们已经讨论过，在维斯特洛大陆上，人们信仰多种不同的宗教。每个宗教都有不同的祈祷和献祭方式。在正确的时间进行正确的祈祷有很多复杂性，我们希望避免暴露这种复杂性。相反，我们将编写一系列可以简化祈祷的适配器。

我们需要的第一件事是一些不同的神，我们可以向他们祈祷：

```js
class OldGods {
  prayTo(sacrifice) {
    console.log("We Old Gods hear your prayer");
  }
}
Religion.OldGods = OldGods;
class DrownedGod {
  prayTo(humanSacrifice) {
    console.log("*BUBBLE* GURGLE");
  }
}
Religion.DrownedGod = DrownedGod;
class SevenGods {
  prayTo(prayerPurpose) {
    console.log("Sorry there are a lot of us, it gets confusing here. Did you pray for something?");
  }
}
Religion.SevenGods = SevenGods;
```

这些类应该看起来很熟悉，因为它们基本上是在上一章中找到的相同类，它们被用作工厂方法的示例。但是，您可能会注意到，每种宗教的`prayTo`方法的签名略有不同。当构建像这里的伪代码中所示的一致接口时，这可能会成为一个问题：

```js
interface God
{
  prayTo():void;
}
```

那么让我们插入一些适配器，作为我们拥有的类和我们想要的签名之间的桥梁：

```js
class OldGodsAdapter {
  constructor() {
    this._oldGods = new OldGods();
  }
  prayTo() {
    let sacrifice = new Sacrifice();
    this._oldGods.prayTo(sacrifice);
  }
}
Religion.OldGodsAdapter = OldGodsAdapter;
class DrownedGodAdapter {
  constructor() {
    this._drownedGod = new DrownedGod();
  }
  prayTo() {
    let sacrifice = new HumanSacrifice();
    this._drownedGod.prayTo(sacrifice);
  }
}
Religion.DrownedGodAdapter = DrownedGodAdapter;
class SevenGodsAdapter {
  constructor() {
    this.prayerPurposeProvider = new PrayerPurposeProvider();
    this._sevenGods = new SevenGods();
  }
  prayTo() {
    this._sevenGods.prayTo(this.prayerPurposeProvider.GetPurpose());
  }
}
Religion.SevenGodsAdapter = SevenGodsAdapter;
class PrayerPurposeProvider {
  GetPurpose() { }
  }
Religion.PrayerPurposeProvider = PrayerPurposeProvider;
```

这些适配器中的每一个都实现了我们想要的`God`接口，并抽象了处理三种不同接口的复杂性，每种接口对应一个神。

要使用桥梁模式，我们可以编写如下代码：

```js
let god1 = new Religion.SevenGodsAdapter();
let god2 = new Religion.DrownedGodAdapter();
let god3 = new Religion.OldGodsAdapter();

let gods = [god1, god2, god3];
for(let i =0; i<gods.length; i++){
  gods[i].praryTo();
}
```

这段代码使用桥梁为众神提供一致的接口，以便它们可以被视为平等的。

在这种情况下，我们只是包装了单个神并通过代理方法调用它们。适配器可以包装多个对象，这是另一个有用的地方可以使用适配器。如果需要编排一系列复杂的对象，那么适配器可以承担一些责任，为其他类提供更简单的接口。

你可以想象桥梁模式是多么有用。它可以与上一章介绍的工厂方法模式很好地结合使用。

这种模式在 JavaScript 中仍然非常有用。正如我在本节开始时提到的，它对于以一致的方式处理不同的 API 非常有用。我已经用它来交换不同的第三方组件，比如不同的图形库或电话系统集成点。如果您正在使用 JavaScript 在移动平台上构建应用程序，那么桥梁模式将成为您的好朋友，可以帮助您清晰地分离通用代码和特定于平台的代码。因为 JavaScript 中没有接口，所以桥梁模式比其他语言中的适配器更接近 JavaScript。实际上，它基本上是一样的。

桥梁还可以使测试变得更容易。我们可以实现一个虚拟桥梁，并使用它来确保对桥梁的调用是正确的。

# 组合

在上一章中，我提到我们希望避免对象之间的紧密耦合。继承是一种非常强的耦合形式，我建议使用组合代替。组合模式是这种情况的一个特例，其中组合被视为可与组件互换。让我们探讨一下组合模式的工作原理。

以下类图包含了构建复合组件的两种不同方式：

![Composite](img/Image00016.jpg)

在第一个中，复合组件由各种组件的固定数量构建。第二个组件是由一个不确定长度的集合构建的。在这两种情况下，父组合中包含的组件可以与组合的类型相同。因此，一个组合可以包含其自身类型的实例。

组合模式的关键特征是组件与其子组件的可互换性。因此，如果我们有一个实现了`IComponent`的组合，那么组合的所有组件也将实现`IComponent`。这可能最好通过一个例子来说明。

## 例子

树结构在计算中非常有用。事实证明，分层树可以表示许多事物。树由一系列节点和边组成，是循环的。在二叉树中，每个节点包含左右子节点，直到我们到达称为叶子的终端节点。

虽然维斯特洛的生活很艰难，但也有机会享受宗教节日或婚礼等事物。在这些活动中，通常会大量享用美味食物。这些食物的食谱与您自己的食谱一样。像烤苹果这样的简单菜肴包含一系列成分：

+   烘焙苹果

+   蜂蜜

+   黄油

+   坚果

这些成分中的每一个都实现了一个我们称之为`IIngredient`的接口。更复杂的食谱包含更多的成分，但除此之外，更复杂的食谱可能包含复杂的成分，这些成分本身是由其他成分制成的。

在维斯特洛南部，一道受欢迎的菜肴是一种甜点，与我们所说的提拉米苏非常相似。这是一个复杂的食谱，其中包含以下成分：

+   奶油

+   蛋糕

+   打发奶油

+   咖啡

当然，奶油本身是由以下成分制成的：

+   牛奶

+   糖

+   鸡蛋

+   香草

奶油是一个复合成分，咖啡和蛋糕也是。

组合对象上的操作通常会通过代理传递给所有包含的对象。

## 实现

这段代码展示了一个简单的成分，即叶子节点：

```js
class SimpleIngredient {
  constructor(name, calories, ironContent, vitaminCContent) {
    this.name = name;
    this.calories = calories;
    this.ironContent = ironContent;
    this.vitaminCContent = vitaminCContent;
  }
  GetName() {
    return this.name;
  }
  GetCalories() {
    return this.calories;
  }
  GetIronContent() {
    return this.ironContent;
  }
  GetVitaminCContent() {
    return this.vitaminCContent;
  }
}
```

它可以与具有成分列表的复合成分互换使用：

```js
class CompoundIngredient {
  constructor(name) {
    this.name = name;
    this.ingredients = new Array();
  }
  AddIngredient(ingredient) {
    this.ingredients.push(ingredient);
  }
  GetName() {
    return this.name;
  }
  GetCalories() {
    let total = 0;
    for (let i = 0; i < this.ingredients.length; i++) {
      total += this.ingredients[i].GetCalories();
    }
    return total;
  }
  GetIronContent() {
    let total = 0;
    for (let i = 0; i < this.ingredients.length; i++) {
      total += this.ingredients[i].GetIronContent();
    }
    return total;
  }
  GetVitaminCContent() {     let total = 0;
    for (let i = 0; i < this.ingredients.length; i++) {
      total += this.ingredients[i].GetVitaminCContent();
    }
    return total;
  }
}
```

复合成分循环遍历其内部成分，并对每个成分执行相同的操作。当然，由于原型模型，无需定义接口。

要使用这种复合成分，我们可以这样做：

```js
let egg = new SimpleIngredient("Egg", 155, 6, 0);
let milk = new SimpleIngredient("Milk", 42, 0, 0);
let sugar = new SimpleIngredient("Sugar", 387, 0,0);
let rice = new SimpleIngredient("Rice", 370, 8, 0);

let ricePudding = new CompoundIngredient("Rice Pudding");
ricePudding.AddIngredient(egg);
ricePudding.AddIngredient(rice);
ricePudding.AddIngredient(milk);
ricePudding.AddIngredient(sugar);

console.log("A serving of rice pudding contains:");
console.log(ricePudding.GetCalories() + " calories");
```

当然，这只显示了模式的一部分。我们可以将米布丁用作更复杂食谱的成分：米布丁馅饼（在维斯特洛有一些奇怪的食物）。由于简单和复合版本的成分具有相同的接口，调用者不需要知道两种成分类型之间有任何区别。

组合是 JavaScript 代码中广泛使用的模式，用于处理 HTML 元素，因为它们是树结构。例如，jQuery 库提供了一个通用接口，如果您选择了单个元素或一组元素。当调用函数时，实际上是在所有子元素上调用，例如：

```js
$("a").hide()
```

这将隐藏页面上的所有链接，而不管调用`$("a")`实际找到多少元素。组合是 JavaScript 开发中非常有用的模式。

# 装饰者

装饰器模式用于包装和增强现有类。使用装饰器模式是对现有组件进行子类化的替代方法。子类化通常是一个编译时操作，是一种紧密耦合。这意味着一旦子类化完成，就无法在运行时进行更改。在存在许多可能的子类化可以组合的情况下，子类化的组合数量会激增。让我们看一个例子。

Westeros 骑士所穿的盔甲可以是非常可配置的。盔甲可以以多种不同的风格制作：鳞甲、板甲、锁子甲等等。除了盔甲的风格之外，还有各种不同的面罩、膝盖和肘部关节，当然还有颜色。由板甲和面罩组成的盔甲的行为与带有面罩的锁子甲是不同的。然而，你可以看到，存在大量可能的组合；明显太多的组合无法显式编码。

我们所做的是使用装饰器模式实现不同风格的盔甲。装饰器使用与适配器和桥接模式类似的理论，它包装另一个实例并通过代理调用。然而，装饰器模式通过将要包装的实例传递给它来在运行时执行重定向。通常，装饰器将作为一些方法的简单传递，对于其他方法，它将进行一些修改。这些修改可能仅限于在将调用传递给包装实例之前执行附加操作，也可能会改变传入的参数。装饰器模式的 UML 表示如下图所示：

![Decorator](img/Image00017.jpg)

这允许对装饰器修改哪些方法进行非常精细的控制，哪些方法保持为简单的传递。让我们来看一下 JavaScript 中该模式的实现。

## 实施

在这段代码中，我们有一个基类`BasicArmor`，然后由`ChainMail`类进行装饰：

```js
class BasicArmor {
  CalculateDamageFromHit(hit) {
    return hit.Strength * .2;
  }
  GetArmorIntegrity() {
    return 1;
  }
}

class ChainMail {
  constructor(decoratedArmor) {
    this.decoratedArmor = decoratedArmor;
  }
  CalculateDamageFromHit(hit) {
    hit.Strength = hit.Strength * .8;
    return this.decoratedArmor.CalculateDamageFromHit(hit);
  }
  GetArmorIntegrity() {
    return .9 * this.decoratedArmor.GetArmorIntegrity();
  }
}
```

`ChainMail`装甲接受符合接口的装甲实例，例如：

```js
export interface IArmor{
  CalculateDamageFromHit(hit: Hit):number;
  GetArmorIntegrity():number;
}
```

该实例被包装并通过代理调用。`GetArmorIntegiry`方法修改了基础类的结果，而`CalculateDamageFromHit`修改了传递给装饰类的参数。这个`ChainMail`类本身可以被装饰多层装饰器，直到实际为每个方法调用调用了一长串方法。当然，这种行为对外部调用者来说是不可见的。

要使用这个装甲装饰器，请看下面的代码：

```js
let armor = new ChainMail(new Westeros.Armor.BasicArmor());
console.log(armor.CalculateDamageFromHit({Location: "head", Weapon: "Sock filled with pennies", Strength: 12}));
```

利用 JavaScript 重写类的单个方法来实现这种模式是很诱人的。事实上，在本节的早期草案中，我本打算建议这样做。然而，这样做在语法上很混乱，不是一种常见的做法。编程时最重要的事情之一是要记住代码必须是可维护的，不仅是对你自己，也是对其他人。复杂性会导致混乱，混乱会导致错误。

装饰器模式是一种对继承过于限制的情况非常有价值的模式。这些情况在 JavaScript 中仍然存在，因此该模式仍然有用。

# Façade

Façade 模式是适配器模式的一种特殊情况，它在一组类上提供了简化的接口。我在适配器模式的部分提到过这样的情景，但只在`SimpleShip`类的上下文中。这个想法可以扩展到提供一个抽象，围绕一组类或整个子系统。Façade 模式在 UML 形式上看起来像下面的图表：

![Façade](img/Image00018.jpg)

## 实施

如果我们将之前的`SimpleShip`扩展为整个舰队，我们就有了一个创建外观的绝佳示例。如果操纵一艘单独的船很困难，那么指挥整个舰队将更加困难。需要大量微妙的操作，必须对单独的船只下达命令。除了单独的船只外，还必须有一位舰队上将，并且需要在船只之间协调以分发补给。所有这些都可以被抽象化。如果我们有一系列代表舰队方面的类，比如这些：

```js
let Ship = (function () {
  function Ship() {
  }
  Ship.prototype.TurnLeft = function () {
  };
  Ship.prototype.TurnRight = function () {
  };
  Ship.prototype.GoForward = function () {
  };
  return Ship;
})();
Transportation.Ship = Ship;

let Admiral = (function () {
  function Admiral() {
  }
  return Admiral;
})();
Transportation.Admiral = Admiral;

let SupplyCoordinator = (function () {
  function SupplyCoordinator() {
  }
  return SupplyCoordinator;
})();
Transportation.SupplyCoordinator = SupplyCoordinator;
```

那么我们可以构建一个外观，如下所示：

```js
let Fleet = (function () {
   function Fleet() {
  }
  Fleet.prototype.setDestination = function (destination) {
    **//pass commands to a series of ships, admirals and whoever else needs it** 

  };

  Fleet.prototype.resupply = function () {
  };

  Fleet.prototype.attack = function (destination) {
    **//attack a city** 

  };
  return Fleet;
})();
```

外观在处理 API 时非常有用。在细粒度 API 周围使用外观可以创建一个更简单的接口。API 的抽象级别可以提高，使其更符合应用程序的工作方式。例如，如果您正在与 Azure blob 存储 API 交互，您可以将抽象级别从处理单个文件提高到处理文件集。而不是编写以下内容：

```js
$.ajax({method: "PUT",
url: "https://settings.blob.core.windows.net/container/set1",
data: "setting data 1"});

$.ajax({method: "PUT",
url: "https://settings.blob.core.windows.net/container/set2",
data: "setting data 2"});

$.ajax({method: "PUT",
url: "https://settings.blob.core.windows.net/container/set3",
data: "setting data 3"});
```

可以编写一个外观，封装所有这些调用并提供一个接口，如下所示：

```js
public interface SettingSaver{
  Save(settings: Settings); //preceding code in this method
  Retrieve():Settings;
}
```

如您所见，外观在 JavaScript 中仍然很有用，并且应该是您工具箱中保留的模式。

# 蝇量级

拳击中有一个 49-52 公斤之间的轻量级级别，被称为蝇量级。这是最后一个建立的级别之一，我想它之所以被命名为蝇量级，是因为其中的拳击手很小，就像苍蝇一样。

蝇量级模式用于对象实例非常多，而这些实例之间只有轻微差异的情况。在这种情况下，大量通常指的是大约 10,000 个对象，而不是 50 个对象。然而，实例数量的截止点高度依赖于创建对象的成本。

在某些情况下，对象可能非常昂贵，系统在超载之前只需要少数对象。在这种情况下，引入蝇量级在较小数量上将是有益的。为每个对象维护一个完整的对象会消耗大量内存。似乎大部分内存也被浪费地消耗掉了，因为大多数实例的字段具有相同的值。蝇量级提供了一种通过仅跟踪与每个实例中的某个原型不同的值来压缩这些数据的方法。

JavaScript 的原型模型非常适合这种情况。我们可以简单地将最常见的值分配给原型，并在需要时覆盖各个实例。让我们看一个例子。

## 实施

再次回到维斯特洛（你是否为我选择了一个单一的主要问题领域感到高兴？），我们发现军队中充满了装备不足的战斗人员。在这些人中，从将军的角度来看，实际上没有太大的区别。当然，每个人都有自己的生活、抱负和梦想，但在将军眼中，他们都已经成为简单的战斗自动机。将军只关心士兵们打得多好，他们是否健康，是否吃饱。我们可以在这段代码中看到简单的字段集：

```js
let Soldier = (function () {
  function Soldier() {
    this.Health = 10;
    this.FightingAbility = 5;
    this.Hunger = 0;
  }
  return Soldier;
})();
```

当然，对于一支由 10,000 名士兵组成的军队，跟踪所有这些需要相当多的内存。让我们采用另一种方法并使用一个类：

```js
class Soldier {
  constructor() {
    this.Health = 10;
    this.FightingAbility = 5;
    this.Hunger = 0;
  }
}
```

使用这种方法，我们可以将对士兵健康的所有请求推迟到原型。设置值也很容易：

```js
let soldier1 = new Soldier();
let soldier2 = new Soldier();
console.log(soldier1.Health); //10
soldier1.Health = 7;
console.log(soldier1.Health); //7
console.log(soldier2.Health); //10
delete soldier1.Health;
console.log(soldier1.Health); //10
```

您会注意到我们调用删除来删除属性覆盖，并将值返回到父值。

# 代理

本章介绍的最后一个模式是代理。在前一节中，我提到创建对象是昂贵的，我们希望避免创建过多的对象。代理模式提供了一种控制昂贵对象的创建和使用的方法。代理模式的 UML 如下图所示：

![代理](img/Image00019.jpg)

正如你所看到的，代理模式反映了实际实例的接口。它被替换为所有客户端中的实例，并且通常包装类的私有实例。代理模式可以在许多地方发挥作用：

+   昂贵对象的延迟实例化

+   保护秘密数据

+   远程方法调用的存根

+   在方法调用之前或之后插入额外的操作

通常一个对象实例化是昂贵的，我们不希望在实际使用之前就创建实例。在这种情况下，代理可以检查它的内部实例，并且如果尚未初始化，则在传递方法调用之前创建它。这被称为延迟实例化。

如果一个类在设计时没有考虑到安全性，但现在需要一些安全性，可以通过使用代理来提供。代理将检查调用，并且只有在安全检查通过的情况下才会传递方法调用。

代理可以简单地提供一个接口，用于调用其他地方调用的方法。事实上，这正是许多网络套接字库的功能，将调用代理回到 Web 服务器。

最后，可能有些情况下，将一些功能插入到方法调用中是有用的。这可能是参数日志记录，参数验证，结果更改，或者其他任何事情。

## 实现

让我们看一个需要方法拦截的 Westeros 示例。通常情况下，液体的计量单位在这片土地的一边和另一边差异很大。在北方，人们可能会买一品脱啤酒，而在南方，人们会用龙来购买。这导致了很多混乱和代码重复，但可以通过包装关心计量的类来解决。

例如，这段代码是用于估算运输液体所需的桶数的桶计算器：

```js
class BarrelCalculator {
  calculateNumberNeeded(volume) {
    return Math.ceil(volume / 157);
  }
}
```

尽管它没有很好的文档记录，但这个版本以品脱作为体积参数。创建一个代理来处理转换：

```js
class DragonBarrelCalculator {
  calculateNumberNeeded(volume) {
    if (this._barrelCalculator == null)
      this._barrelCalculator = new BarrelCalculator();
    return this._barrelCalculator.calculateNumberNeeded(volume * .77);
  }
}
```

同样，我们可能为基于品脱的桶计算器创建另一个代理：

```js
class PintBarrelCalculator {
  calculateNumberNeeded(volume) {
    if (this._barrelCalculator == null)
      this._barrelCalculator = new BarrelCalculator();
    return this._barrelCalculator.calculateNumberNeeded(volume * 1.2);
  }
}
```

这个代理类为我们做了单位转换，并帮助减轻了一些关于单位的混乱。一些语言，比如 F#，支持单位的概念。实际上，它是一种类型系统，覆盖在简单的数据类型上，如整数，防止程序员犯错，比如将表示品脱的数字加到表示升的数字上。在 JavaScript 中，没有这样的能力。然而，使用 JS-Quantities（[`gentooboontoo.github.io/js-quantities/`](http://gentooboontoo.github.io/js-quantities/)）这样的库是一个选择。如果你看一下，你会发现语法非常痛苦。这是因为 JavaScript 不允许操作符重载。看到像将一个空数组添加到另一个空数组一样奇怪的事情（结果是一个空字符串），也许我们可以感谢 JavaScript 不支持操作符重载。

如果我们想要防止在有品脱而认为有龙时意外使用错误类型的计算器，那么我们可以停止使用原始类型，并为数量使用一种类型，一种类似于贫穷人的计量单位：

```js
class PintUnit {
  constructor(unit, quantity) {
    this.quanity = quantity;
  }
}
```

这可以作为代理中的一个保护使用：

```js
class PintBarrelCalculator {
  calculateNumberNeeded(volume) {
    if(PintUnit.prototype == Object.getPrototypeOf(volume))
      //throw some sort of error or compensate
    if (this._barrelCalculator == null)
      this._barrelCalculator = new BarrelCalculator();
    return this._barrelCalculator.calculateNumberNeeded(volume * 1.2);
  }
}
```

正如你所看到的，我们最终得到了基本上与 JS-Quantities 相同的东西，但是以更 ES6 的形式。

代理模式在 JavaScript 中绝对是一个有用的模式。我已经提到它在生成存根时被 Web 套接字库使用，但它在无数其他位置也很有用。

# 提示和技巧

本章介绍的许多模式提供了抽象功能和塑造接口的方法。请记住，每一层抽象都会引入成本。函数调用会变慢，但对于需要理解您的代码的人来说，这也更加令人困惑。工具可以帮助一点，但跟踪一个函数调用穿过九层抽象从来都不是一件有趣的事情。

同时要小心在外观模式中做得太多。很容易将外观转化为一个完全成熟的管理类，这很容易变成一个负责协调和执行一切的上帝对象。

# 总结

在本章中，我们已经看了一些用于构造对象之间交互的模式。它们中的一些模式相互之间相当相似，但它们在 JavaScript 中都很有用，尽管桥接模式实际上被简化为适配器。在下一章中，我们将通过查看行为模式来完成对原始 GoF 模式的考察。

# 第五章：行为模式

在上一章中，我们看了描述对象如何构建以便简化交互的结构模式。

在本章中，我们将看一下 GoF 模式的最后一个，也是最大的分组：行为模式。这些模式提供了关于对象如何共享数据或者从不同的角度来看，数据如何在对象之间流动的指导。

我们将要看的模式如下：

+   责任链

+   命令

+   解释器

+   迭代器

+   中介者

+   备忘录

+   观察者

+   状态

+   策略

+   模板方法

+   访问者

再次，有许多最近确定的模式可能很好地被归类为行为模式。我们将推迟到以后的章节再来看这些模式，而是继续使用 GoF 模式。

# 责任链

我们可以将对象上的函数调用看作是向该对象发送消息。事实上，这种消息传递的思维方式可以追溯到 Smalltalk 的时代。责任链模式描述了一种方法，其中消息从一个类传递到另一个类。一个类可以对消息进行操作，也可以将其传递给链中的下一个成员。根据实现，可以对消息传递应用一些不同的规则。在某些情况下，只允许链中的第一个匹配链接对消息进行操作。在其他情况下，每个匹配的链接都对消息进行操作。有时允许链接停止处理，甚至在消息继续传递下去时改变消息：

![责任链](img/Image00020.jpg)

让我们看看我们常用的例子中是否能找到这种模式的一个很好的例子：维斯特洛大陆。

## 实现

在维斯特洛大陆，法律制度几乎不存在。当然有法律，甚至有执行它们的城市警卫，但司法系统很少。这片土地的法律实际上是由国王和他的顾问决定的。有时间和金钱的人可以向国王请愿，国王会听取他们的投诉并作出裁决。这个裁决就是法律。当然，任何整天听农民的投诉的国王都会发疯。因此，许多案件在传到国王耳朵之前就被他的顾问们抓住并解决了。

为了在代码中表示这一点，我们需要首先考虑责任链将如何工作。投诉进来，从能够解决问题的最低可能的人开始。如果那个人不能或不愿解决问题，它就会上升到统治阶级的更高级成员。最终问题会达到国王，他是争端的最终仲裁者。我们可以把他看作是默认的争端解决者，当一切都失败时才会被召唤。责任链在下图中可见：

![实施](img/Image00021.jpg)

我们将从一个描述可能听取投诉的接口开始：

```js
export interface ComplaintListener{
  IsAbleToResolveComplaint(complaint: Complaint): boolean;
  ListenToComplaint(complaint: Complaint): string;
}
```

接口需要两个方法。第一个是一个简单的检查，看看类是否能够解决给定的投诉。第二个是听取和解决投诉。接下来，我们需要描述什么构成了投诉：

```js
var Complaint = (function () {
  function Complaint() {
    this.ComplainingParty = "";
    this.ComplaintAbout = "";
    this.Complaint = "";
  }
  return Complaint;
})();
```

接下来，我们需要一些不同的类来实现`ComplaintListener`，并且能够解决投诉：

```js
class ClerkOfTheCourt {
  IsInterestedInComplaint(complaint) {
    //decide if this is a complaint which can be solved by the clerk
    if(isInterested())
      return true;
    return false;
  }
  ListenToComplaint(complaint) {
    //perform some operation
    //return solution to the complaint
    return "";
  }
}
JudicialSystem.ClerkOfTheCourt = ClerkOfTheCourt;
class King {
  IsInterestedInComplaint(complaint) {
    return true;//king is the final member in the chain so must return true
  }
  ListenToComplaint(complaint) {
    //perform some operation
    //return solution to the complaint
    return "";
  }
}
JudicialSystem.King = King;
```

这些类中的每一个都实现了解决投诉的不同方法。我们需要将它们链接在一起，确保国王处于默认位置。这可以在这段代码中看到：

```js
class ComplaintResolver {
  constructor() {
    this.complaintListeners = new Array();
     this.complaintListeners.push(new ClerkOfTheCourt());
     this.complaintListeners.push(new King());
  }
  ResolveComplaint(complaint) {
    for (var i = 0; i < this.complaintListeners.length; i++) {
      if         (this.complaintListeners[i].IsInterestedInComplaint(complaint)) {
        return this.complaintListeners[i].ListenToComplaint(complaint);
      }
    }
  }
}
```

这段代码将逐个遍历每个监听器，直到找到一个对听取投诉感兴趣的监听器。在这个版本中，结果会立即返回，停止任何进一步的处理。这种模式有多种变体，其中多个监听器可以触发，甚至允许监听器改变下一个监听器的参数。以下图表显示了多个配置的监听器：

![实施](img/Image00022.jpg)

责任链在 JavaScript 中是一个非常有用的模式。在基于浏览器的 JavaScript 中，触发的事件会经过一条责任链。例如，您可以将多个监听器附加到链接的单击事件上，每个监听器都会触发，最后是默认的导航监听器。很可能您在大部分代码中都在使用责任链，甚至自己都不知道。

# 命令

命令模式是一种封装方法参数、当前对象状态以及要调用的方法的方法。实际上，命令模式将调用方法所需的一切打包到一个很好的包中，可以在以后的某个日期调用。使用这种方法，可以发出命令，并等到以后再决定哪段代码将执行该命令。然后可以将此包排队或甚至序列化以供以后执行。具有单一的命令执行点还允许轻松添加功能，如撤消或命令记录。

这种模式可能有点难以想象，所以让我们把它分解成其组成部分：

![命令](img/Image00023.jpg)

## 命令消息

命令模式的第一个组件是，可预测地，命令本身。正如我提到的，命令封装了调用方法所需的一切。这包括方法名、参数和任何全局状态。可以想象，在每个命令中跟踪全局状态是非常困难的。如果全局状态在命令创建后发生变化会发生什么？这个困境是使用全局状态的另一个原因，它是有问题的，应该避免使用。

设置命令有几种选择。在简单的一端，只需要跟踪一个函数和一组参数。因为 JavaScript 中函数是一等对象，它们可以很容易地保存到对象中。我们还可以将函数的参数保存到一个简单的数组中。让我们使用这种非常简单的方法构建一个命令。

命令的延迟性质在维斯特洛大陆中有一个明显的隐喻。在维斯特洛大陆中没有快速通信的方法。最好的方法是将小消息附加到鸟上并释放它们。这些鸟倾向于想要返回自己的家，因此每个领主在自己的家中饲养一些鸟，当它们成年时，将它们发送给其他可能希望与他们交流的领主。领主们保留一群鸟并记录哪只鸟将飞往哪个其他领主。维斯特洛国王通过这种方法向他忠诚的领主发送了许多命令。

国王发送的命令包含了领主所需的所有指令。命令可能是像带领你的部队这样的东西，而该命令的参数可能是部队的数量、位置和命令必须执行的日期。

在 JavaScript 中，最简单的表示方法是通过数组：

```js
var simpleCommand = new Array();
simpleCommand.push(new LordInstructions().BringTroops);
simpleCommand.push("King's Landing");
simpleCommand.push(500);
simpleCommand.push(new Date());
```

这个数组可以随意传递和调用。要调用它，可以使用一个通用函数：

```js
simpleCommand0;
```

如您所见，这个函数只适用于具有三个参数的命令。当然，您可以将其扩展到任意数量：

```js
simpleCommand0;
```

附加参数是未定义的，但函数不使用它们，因此没有任何不良影响。当然，这绝不是一个优雅的解决方案。

为每种类型的命令构建一个类是可取的。这样可以确保正确的参数已被提供，并且可以轻松区分集合中的不同类型的命令。通常，命令使用祈使句命名，因为它们是指令。例如，BringTroops、Surrender、SendSupplies 等。

让我们将我们丑陋的简单命令转换成一个合适的类：

```js
class BringTroopsCommand {
  constructor(location, numberOfTroops, when) {
    this._location = location;
    this._numberOfTroops = numberOfTroops;
    this._when = when;
  }
  Execute() {
    var receiver = new LordInstructions();
    receiver.BringTroops(this._location, this._numberOfTroops, this._when);
  }
}
```

我们可能希望实现一些逻辑来确保传递给构造函数的参数是正确的。这将确保命令在创建时失败，而不是在执行时失败。在执行期间可能会延迟，甚至可能延迟几天。验证可能不完美，但即使它只能捕捉到一小部分错误，也是有帮助的。

正如前面提到的，这些命令可以保存在内存中以供以后使用，甚至可以写入磁盘。

## 调用者

调用者是命令模式的一部分，指示命令执行其指令。调用者实际上可以是任何东西：定时事件，用户交互，或者只是流程中的下一步都可能触发调用。在前面的部分中执行`simpleCommand`命令时，我们在扮演调用者的角色。在更严格的命令中，调用者可能看起来像下面这样：

```js
command.Execute()
```

如您所见，调用命令非常容易。命令可以立即调用，也可以在以后的某个时间调用。一种流行的方法是将命令的执行推迟到事件循环的末尾。这可以在节点中完成：

```js
process.nextTick(function(){command.Execute();});
```

函数`process.nextTick`将命令的执行推迟到事件循环的末尾，以便在下次进程没有事情可做时执行。

## 接收者

命令模式中的最后一个组件是接收者。这是命令执行的目标。在我们的例子中，我们创建了一个名为`LordInstructions`的接收者：

```js
class LordInstructions {
  BringTroops(location, numberOfTroops, when) {
    console.log(`You have been instructed to bring ${numberOfTroops} troops to ${location} by ${when}`);
  }
}
```

接收者知道如何执行命令推迟的操作。实际上，接收者可能是任何类，而不必有任何特殊之处。

这些组件共同构成了命令模式。客户端将生成一个命令，将其传递给一个调用者，该调用者可以延迟命令的执行或立即执行，然后命令将作用于接收者。

在构建撤销堆栈的情况下，命令是特殊的，因为它们既有`Execute`方法，也有`Undo`方法。一个将应用程序状态推进，另一个将其推回。要执行撤销，只需从撤销堆栈中弹出命令，执行`Undo`函数，并将其推到重做堆栈上。对于重做，从重做中弹出，执行`Execute`，并推到撤销堆栈上。就是这么简单，尽管必须确保所有状态变化都是通过命令执行的。

《设计模式》一书概述了命令模式的一组稍微复杂的玩家。这在很大程度上是由于我们在 JavaScript 中避免了接口的依赖。由于 JavaScript 中的原型继承模型，该模式变得简单得多。

命令模式是一个非常有用的模式，用于推迟执行某段代码。我们将在《第十章 消息模式》中实际探讨命令模式和一些有用的伴生模式。

# 解释器

解释器模式是一种有趣的模式，因为它允许你创建自己的语言。这可能听起来有点疯狂，我们已经在写 JavaScript 了，为什么还要创建一个新的语言？自《设计模式》一书以来，领域特定语言（DSL）已经有了一些复兴。有些情况下，创建一个特定于某一需求的语言是非常有用的。例如，结构化查询语言（SQL）非常擅长描述对关系数据库的查询。同样，正则表达式已被证明在解析和操作文本方面非常有效。

有许多情况下，能够创建一个简单的语言是有用的。这才是关键：一个简单的语言。一旦语言变得更加复杂，优势很快就会因为创建实际上是一个编译器的困难而丧失。

这种模式与我们到目前为止看到的模式不同，因为它没有真正由模式定义的类结构。你可以按照自己的意愿设计你的语言解释器。

## 示例

对于我们的示例，让我们定义一种语言，用于描述维斯特洛大陆上的历史战斗。这种语言必须简单易懂，便于文职人员编写。我们将从创建一个简单的语法开始：

```js
(aggressor -> battle ground <- defender) -> victor
```

在这里，你可以看到我们只是写出了一个相当不错的语法，让人们描述战斗。罗伯特·拜拉席恩和雷加·坦格利安在三叉戟河之间的战斗将如下所示：

```js
(Robert Baratheon -> River Trident <- RhaegarTargaryen) -> Robert Baratheon
```

使用这种语法，我们希望构建一些能够查询战斗列表的代码。为了做到这一点，我们将依赖于正则表达式。对于大多数语言来说，这不是一个好的方法，因为语法太复杂。在这种情况下，人们可能希望创建一个词法分析器和一个解析器，并构建语法树，然而，到了那个时候，你可能会希望重新审视一下是否创建 DSL 真的是一个好主意。对于我们的语言，语法非常简单，所以我们可以使用正则表达式。

## 实现

我们首先为战斗建立一个 JavaScript 数据模型，如下所示：

```js
class Battle {
  constructor(battleGround, agressor, defender, victor) {
    this.battleGround = battleGround;
    this.agressor = agressor;
    this.defender = defender;
    this.victor = victor;
  }
}
```

接下来我们需要一个解析器：

```js
class Parser {
  constructor(battleText) {
    this.battleText = battleText;
    this.currentIndex = 0;
    this.battleList = battleText.split("\n");
  }
  nextBattle() {
   if (!this.battleList[0])
     return null;
    var segments = this.battleList[0].match(/\((.+?)\s?->\s?(.+?)\s?<-\s?(.+?)\s?->\s?(.+)/);
    return new Battle(segments[2], segments[1], segments[3], segments[4]);
  }
}
```

最好不要太在意那个正则表达式。然而，这个类确实接受一系列战斗（每行一个），并使用`next Battle`，允许解析它们。要使用这个类，我们只需要做以下操作：

```js
var text = "(Robert Baratheon -> River Trident <- RhaegarTargaryen) -> Robert Baratheon";
var p = new Parser(text);
p.nextBattle()
```

这将是输出：

```js
{
  battleGround: 'River Trident',
  agressor: 'Robert Baratheon',
  defender: 'RhaegarTargaryen)',
  victor: 'Robert Baratheon'
}
```

现在可以像查询 JavaScript 中的任何其他结构一样查询这个数据结构了。

正如我之前提到的，实现这种模式没有固定的方式，因此在前面的代码中所做的实现只是提供了一个例子。你的实现很可能会看起来非常不同，这也是可以的。

解释器在 JavaScript 中可能是一个有用的模式。然而，在大多数情况下，这是一个相当少用的模式。JavaScript 中解释的最佳示例是编译为 CSS 的语言。

# 迭代器

遍历对象集合是一个非常常见的问题。以至于许多语言都提供了专门的构造来遍历集合。例如，C#有`foreach`循环，Python 有`for x in`。这些循环构造经常建立在迭代器之上。迭代器是一种模式，提供了一种简单的方法，按顺序选择集合中的下一个项目。

迭代器的接口如下：

```js
interface Iterator{
  next();
}
```

## 实现

在维斯特洛大陆，有一个众所周知的人们排队等候王位的序列，以备国王不幸去世的情况。我们可以在这个集合上设置一个方便的迭代器，如果统治者去世，只需简单地调用`next`：

```js
class KingSuccession {
  constructor(inLineForThrone) {
    this.inLineForThrone = inLineForThrone;
    this.pointer = 0;
  }
  next() {
    return this.inLineForThrone[this.pointer++];
  }
}
```

这是用一个数组初始化的，然后我们可以调用它：

```js
var king = new KingSuccession(["Robert Baratheon" ,"JofferyBaratheon", "TommenBaratheon"]);
king.next() //'Robert Baratheon'
king.next() //'JofferyBaratheon'
king.next() //'TommenBaratheon'
```

迭代器的一个有趣的应用是不仅仅迭代一个固定的集合。例如，迭代器可以用来生成无限集合的顺序成员，比如斐波那契序列：

```js
class FibonacciIterator {
  constructor() {
    this.previous = 1;
    this.beforePrevious = 1;
  }
  next() {
    var current = this.previous + this.beforePrevious;
    this.beforePrevious = this.previous;
    this.previous = current;
    return current;
  }
}
```

这样使用：

```js
var fib = new FibonacciIterator()
fib.next() //2
fib.next() //3
fib.next() //5
fib.next() //8
fib.next() //13
fib.next() //21
```

迭代器是方便的构造，允许探索不仅仅是数组，而且是任何集合，甚至是任何生成的列表。有很多地方可以大量使用这个。

## ECMAScript 2015 迭代器

迭代器是如此有用，以至于它们实际上是 JavaScript 下一代的一部分。ECMAScript 2015 中使用的迭代器模式是一个返回包含`done`和`value`的对象的单个方法。当迭代器在集合的末尾时，`done`为`true`。ECMAScript 2015 迭代器的好处是 JavaScript 中的数组集合将支持迭代器。这开辟了一种新的语法，可以在很大程度上取代`for`循环：

```js
var kings = new KingSuccession(["Robert Baratheon" ,"JofferyBaratheon", "TommenBaratheon"]);
for(var king of kings){
  //act on members of kings
}
```

迭代器是 JavaScript 长期以来一直缺少的一种语法上的美好。ECMAScript-2015 的另一个很棒的特性是生成器。这实际上是一个内置的迭代器工厂。我们的斐波那契序列可以重写如下：

```js
function* FibonacciGenerator (){
  var previous = 1;
  var beforePrevious = 1;
  while(true){
    var current = previous + beforePrevious;
    beforePrevious = previous;
    previous = current;
    yield current;
  }
}
```

这样使用：

```js
var fib = new FibonacciGenerator()
fib.next().value //2
fib.next().value //3
fib.next().value //5
fib.next().value //8
fib.next().value //13
fib.next().value //21
```

# 中介者

在类中管理多对多关系可能是一个复杂的前景。让我们考虑一个包含多个控件的表单，每个控件都想在执行操作之前知道页面上的其他控件是否有效。不幸的是，让每个控件都知道其他控件会创建一个维护噩梦。每次添加一个新控件，都需要修改每个其他控件。

中介者将坐在各种组件之间，并作为一个单一的地方，可以进行消息路由的更改。通过这样做，中介者简化了维护代码所需的复杂工作。在表单控件的情况下，中介者很可能是表单本身。中介者的作用很像现实生活中的中介者，澄清和路由各方之间的信息交流：

![中介者](img/Image00024.jpg)

## 实现

在维斯特洛大陆，经常需要中介者。中介者经常会死去，但我相信这不会发生在我们的例子中。

在维斯特洛大陆有许多伟大的家族拥有大城堡和广阔的土地。次要领主们向大家族宣誓效忠，形成联盟，经常通过婚姻得到支持。

在协调各家族的时候，大领主将充当中介者，来回传递信息并解决他们之间可能发生的任何争端。

在这个例子中，我们将大大简化各家之间的通信，并说所有消息都通过大领主传递。在这种情况下，我们将使用史塔克家作为我们的大领主。他们有许多其他家族与他们交谈。每个家族看起来大致如下：

```js
class Karstark {
  constructor(greatLord) {
    this.greatLord = greatLord;
  }
  receiveMessage(message) {
  }
  sendMessage(message) {
    this.greatLord.routeMessage(message);
  }
}
```

它们有两个函数，一个接收来自第三方的消息，一个发送消息给他们的大领主，这是在实例化时设置的。`HouseStark`类如下所示：

```js
class HouseStark {
  constructor() {
    this.karstark = new Karstark(this);
    this.bolton = new Bolton(this);
    this.frey = new Frey(this);
    this.umber = new Umber(this);
  }
  routeMessage(message) {
  }
}
```

通过`HouseStark`类传递所有消息，其他各个家族不需要关心它们的消息是如何路由的。这个责任被交给了`HouseStark`，它充当了中介。

中介者最适合用于通信既复杂又明确定义的情况。如果通信不复杂，那么中介者会增加额外的复杂性。如果通信不明确定义，那么在一个地方对通信规则进行编码就变得困难。

在 JavaScript 中，简化多对多对象之间的通信肯定是有用的。我实际上认为在许多方面，jQuery 充当了中介者。在页面上操作一组项目时，它通过抽象掉代码需要准确知道页面上哪些对象正在被更改来简化通信。例如：

```js
$(".error").slideToggle();
```

jQuery 是切换页面上所有具有`error`类的元素的可见性的简写吗？

# 备忘录

在命令模式的部分，我们简要讨论了撤销操作的能力。创建可逆命令并非总是可能的。对于许多操作，没有明显的逆向操作可以恢复原始状态。例如，想象一下对一个数字进行平方的代码：

```js
class SquareCommand {
  constructor(numberToSquare) {
    this.numberToSquare = numberToSquare;
  }
  Execute() {
    this.numberToSquare *= this.numberToSquare;
  }
}
```

给这段代码-9 将得到 81，但给它 9 也将得到 81。没有办法在没有额外信息的情况下撤销这个命令。

备忘录模式提供了一种恢复对象状态到先前状态的方法。备忘录记录了变量先前的值，并提供了恢复它们的功能。为每个命令保留一个备忘录可以轻松恢复不可逆转的命令。

除了撤销堆栈之外，还有许多情况下，具有回滚对象状态的能力是有用的。例如，进行假设分析需要对状态进行一些假设性的更改，然后观察事物如何变化。这些更改通常不是永久性的，因此可以使用备忘录模式进行回滚，或者如果项目是可取的，可以保留下来。备忘录模式的图表可以在这里看到：

![备忘录](img/Image00025.jpg)

典型的备忘录实现涉及三个角色：

+   **原始对象**：原始对象保存某种状态并提供生成新备忘录的接口。

+   **看护者**：这是模式的客户端，它请求获取新备忘录并管理何时进行恢复。

+   **备忘录**：这是原始对象保存状态的表示。这可以持久化到存储中以便进行回滚。

将备忘录模式的成员想象成老板和秘书做笔记可能会有所帮助。老板（看护者）向秘书（原始对象）口述备忘录，秘书在记事本（备忘录）上写下笔记。偶尔老板可能会要求秘书划掉他刚刚写的内容。

与备忘录模式相关的看护者的参与可以有所不同。在某些实现中，原始对象在其状态发生变化时会生成一个新的备忘录。这通常被称为写时复制，因为会创建状态的新副本并应用变化。旧版本可以保存到备忘录中。

## 实施

在维斯特洛大陆上有许多预言者，他们是未来的预言者。他们通过使用魔法来窥视未来，并检查当前的某些变化将如何在未来发挥作用。通常需要进行许多略有不同起始条件的预测。在设置起始条件时，备忘录模式是非常宝贵的。

我们从一个世界状态开始，它提供了某个特定起点的世界状态信息：

```js
class WorldState {
  constructor(numberOfKings, currentKingInKingsLanding, season) {
    this.numberOfKings = numberOfKings;
    this.currentKingInKingsLanding = currentKingInKingsLanding;
    this.season = season;
  }
}
```

这个`WorldState`类负责跟踪构成世界的所有条件。每当对起始条件进行更改时，应用程序都会修改它。因为这个世界状态包含了应用程序的所有状态，所以它可以被用作备忘录。我们可以将这个对象序列化并保存到磁盘上，或者发送回某个历史服务器。

接下来我们需要一个类，它提供与备忘录相同的状态，并允许创建和恢复备忘录。在我们的示例中，我们将其称为`WorldStateProvider`：

```js
class WorldStateProvider {
  saveMemento() {
    return new WorldState(this.numberOfKings, this.currentKingInKingsLanding, this.season);
  }
  restoreMemento(memento) {
    this.numberOfKings = memento.numberOfKings;
    this.currentKingInKingsLanding = memento.currentKingInKingsLanding;
    this.season = memento.season;
  }
}
```

最后，我们需要一个预言者的客户端，我们将称之为`Soothsayer`：

```js
class Soothsayer {
  constructor() {
    this.startingPoints = [];
    this.currentState = new WorldStateProvider();
  }
  setInitialConditions(numberOfKings, currentKingInKingsLanding, season) {
    this.currentState.numberOfKings = numberOfKings;
    this.currentState.currentKingInKingsLanding = currentKingInKingsLanding;
    this.currentState.season = season;
  }
  alterNumberOfKingsAndForetell(numberOfKings) {
    this.startingPoints.push(this.currentState.saveMemento());
    this.currentState.numberOfKings = numberOfKings;
  }
  alterSeasonAndForetell(season) {
    this.startingPoints.push(this.currentState.saveMemento());
    this.currentState.season = season;
  }
  alterCurrentKingInKingsLandingAndForetell(currentKingInKingsLanding) {
    this.startingPoints.push(this.currentState.saveMemento());
    this.currentState.currentKingInKingsLanding = currentKingInKingsLanding;
    //run some sort of prediction
  }
  tryADifferentChange() {
    this.currentState.restoreMemento(this.startingPoints.pop());
  }
}
```

这个类提供了一些方便的方法，它们改变了世界的状态，然后运行了一个预言。这些方法中的每一个都将先前的状态推入历史数组`startingPoints`。还有一个方法`tryADifferentChange`，它撤销了先前的状态更改，准备运行另一个预言。撤销是通过加载存储在数组中的备忘录来执行的。

尽管客户端 JavaScript 应用有很高的血统，但提供撤销功能却非常罕见。我相信这其中有各种原因，但大部分原因可能是人们并不期望有这样的功能。然而，在大多数桌面应用程序中，撤销功能是被期望的。我想，随着客户端应用程序在功能上不断增强，撤销功能将变得更加重要。当这种情况发生时，备忘录模式是实现撤销堆栈的一种绝妙方式。

# 观察者

观察者模式可能是 JavaScript 世界中使用最多的模式。这种模式特别在现代单页应用程序中使用；它是提供**模型视图视图模型**（**MVVM**）功能的各种库的重要组成部分。我们将在第七章中详细探讨这些模式，*响应式编程*。

经常有必要知道对象的值何时发生了变化。为了做到这一点，您可以用 getter 和 setter 包装感兴趣的属性：

```js
class GetterSetter {
  GetProperty() {
    return this._property;
  }
  SetProperty(value) {
    this._property = value;
  }
}
```

setter 函数现在可以增加对其他对值发生变化感兴趣的对象的调用：

```js
SetProperty(value) {
  var temp = this._property;
  this._property = value;
  this._listener.Event(value, temp);
}
```

现在，这个 setter 将通知监听器属性已发生变化。在这种情况下，旧值和新值都已包括在内。这并不是必要的，因为监听器可以负责跟踪先前的值。

观察者模式概括和规范了这个想法。观察者模式允许感兴趣的各方订阅变化通知，而不是只有一个调用监听器的单个调用。多个订阅者可以在下图中看到：

![Observer](img/Image00026.jpg)

## 实施

维斯特洛的法庭是一个充满阴谋和诡计的地方。控制谁坐在王位上，以及他们的行动，是一个复杂的游戏。权力的游戏中的许多玩家雇佣了许多间谍来发现其他人的行动。这些间谍经常被多个玩家雇佣，并必须向所有玩家报告他们所发现的情况。

间谍是使用观察者模式的理想场所。在我们的特定示例中，被雇佣的间谍是国王的官方医生，玩家们非常关心给这位患病的国王开了多少止痛药。知道这一点可以让玩家提前知道国王可能何时去世 - 这是一个非常有用的信息。

间谍看起来像下面这样：

```js
class Spy {
  constructor() {
    this._partiesToNotify = [];
  }
  Subscribe(subscriber) {
    this._partiesToNotify.push(subscriber);
  }
  Unsubscribe(subscriber) {
    this._partiesToNotify.remove(subscriber);
  }
  SetPainKillers(painKillers) {
    this._painKillers = painKillers;
    for (var i = 0; i < this._partiesToNotify.length; i++) {
      this._partiesToNotifyi;
    }
  }
}
```

在其他语言中，订阅者通常必须遵守某个接口，观察者只会调用接口方法。这种负担在 JavaScript 中不存在，事实上，我们只给`Spy`类一个函数。这意味着订阅者不需要严格的接口。这是一个例子：

```js
class Player {
  OnKingPainKillerChange(newPainKillerAmount) {
    //perform some action
  }
}
```

可以这样使用：

```js
let s = new Spy();
let p = new Player();
s.Subscribe(p.OnKingPainKillerChange); //p is now a subscriber
s.SetPainKillers(12); //s will notify all subscribers
```

这提供了一种非常简单和高效的构建观察者的方法。订阅者使订阅者与可观察对象解耦。

观察者模式也可以应用于方法和属性。通过这样做，可以提供用于发生附加行为的钩子。这是为 JavaScript 库提供插件基础设施的常见方法。

在浏览器中，DOM 中各种项目上的所有事件监听器都是使用观察者模式实现的。例如，使用流行的 jQuery 库，可以通过以下方式订阅页面上所有按钮的`click`事件：

```js
$("body").on("click", "button", function(){/*do something*/})
```

即使在纯 JavaScript 中，相同的模式也适用：

```js
let buttons = document.getElementsByTagName("button");
for(let i =0; i< buttons.length; i++)
{
  buttons[i].onclick = function(){/*do something*/}
}
```

显然，观察者模式在处理 JavaScript 时非常有用。没有必要以任何重大方式改变模式。

# 状态

状态机在计算机编程中是一个非常有用的设备。不幸的是，大多数程序员并不经常使用它们。我相信对状态机的一些反对意见至少部分是因为许多人将它们实现为一个巨大的`if`语句，如下所示：

```js
function (action, amount) {
  if (this.state == "overdrawn" && action == "withdraw") {
    this.state = "on hold";
  }
  if (this.state == "on hold" && action != "deposit") {
    this.state = "on hold";
  }
  if (this.state == "good standing" && action == "withdraw" && amount <= this.balance) {
    this.balance -= amount;
  }
  if (this.state == "good standing" && action == "withdraw" && amount >this.balance) {
    this.balance -= amount;
    this.state = "overdrawn";
  }
};
```

这只是一个可能更长的示例。这样长的`if`语句很难调试，而且容易出错。只需翻转一个大于号就足以大大改变`if`语句的工作方式。

不要使用单个巨大的`if`语句块，我们可以利用状态模式。状态模式的特点是有一个状态管理器，它抽象了内部状态，并将消息代理到适当的状态，该状态实现为一个类。所有状态内部的逻辑和状态转换的控制都由各个状态类管理。状态管理器模式可以在以下图表中看到：

![State](img/Image00027.jpg)

将状态分为每个状态一个类允许更小的代码块进行调试，并且使测试变得更容易。

状态管理器的接口非常简单，通常只提供与各个状态通信所需的方法。管理器还可以包含一些共享状态变量。

## 实施

正如在`if`语句示例中所暗示的，维斯特洛有一个银行系统。其中大部分集中在布拉沃斯岛上。那里的银行业务与这里的银行业务基本相同，包括账户、存款和取款。管理银行账户的状态涉及监视所有交易并根据交易改变银行账户的状态。

让我们来看看管理布拉沃斯银行账户所需的一些代码。首先是状态管理器：

```js
class BankAccountManager {
  constructor() {
    this.currentState = new GoodStandingState(this);
  }
  Deposit(amount) {
    this.currentState.Deposit(amount);
  }
  Withdraw(amount) {
    this.currentState.Withdraw(amount);
  }
  addToBalance(amount) {
    this.balance += amount;
  }
  getBalance() {
    return this.balance;
  }
  moveToState(newState) {
    this.currentState = newState;
  }
}
```

`BankAccountManager`类提供了当前余额和当前状态的状态。为了保护余额，它提供了一个用于读取余额的辅助工具，另一个用于增加余额。在真实的银行应用程序中，我更希望设置余额的功能比这个更有保护性。在这个`BankManager`版本中，操作当前状态的能力对状态是可访问的。它们有责任改变状态。这个功能可以集中在管理器中，但这会增加添加新状态的复杂性。

我们已经为银行账户确定了三种简单的状态：`Overdrawn`，`OnHold`和`GoodStanding`。每个状态在该状态下负责处理取款和存款。`GoodStandingstate`类如下所示：

```js
class GoodStandingState {
  constructor(manager) {
    this.manager = manager;
  }
  Deposit(amount) {
    this.manager.addToBalance(amount);
  }
  Withdraw(amount) {
    if (this.manager.getBalance() < amount) {
      this.manager.moveToState(new OverdrawnState(this.manager));
    }
    this.manager.addToBalance(-1 * amount);
  }
}
```

`OverdrawnState`类如下所示：

```js
class OverdrawnState {
  constructor(manager) {
    this.manager = manager;
  }
  Deposit(amount) {
    this.manager.addToBalance(amount);
    if (this.manager.getBalance() > 0) {
      this.manager.moveToState(new GoodStandingState(this.manager));
    }
  }
  Withdraw(amount) {
    this.manager.moveToState(new OnHold(this.manager));
    throw "Cannot withdraw money from an already overdrawn bank account";
  }
}
```

最后，`OnHold`状态如下所示：

```js
class OnHold {
  constructor(manager) {
    this.manager = manager;
  }
  Deposit(amount) {
    this.manager.addToBalance(amount);
    throw "Your account is on hold and you must attend the bank to resolve the issue";
  }
  Withdraw(amount) {
    throw "Your account is on hold and you must attend the bank to resolve the issue";
  }
}
```

您可以看到，我们已经成功地将混乱的`if`语句的所有逻辑重现在一些简单的类中。这里的代码量看起来比`if`语句要多得多，但从长远来看，将代码封装到单独的类中将会得到回报。

在 JavaScript 中有很多机会可以利用这种模式。跟踪状态是大多数应用程序中的典型问题。当状态之间的转换很复杂时，将其封装在状态模式中是简化事情的一种方法。还可以通过按顺序注册事件来构建简单的工作流程。这样做的一个好接口可能是流畅的，这样你就可以注册以下状态：

```js
goodStandingState
.on("withdraw")
.when(function(manager){return manager.balance > 0;})
  .transitionTo("goodStanding")
.when(function(manager){return mangaer.balance <=0;})
  .transitionTo("overdrawn");
```

# 策略

有人说过有很多种方法可以剥猫皮。我明智地从未研究过有多少种方法。在计算机编程中，算法也经常如此。通常有许多版本的算法，它们在内存使用和 CPU 使用之间进行权衡。有时会有不同的方法提供不同级别的保真度。例如，在智能手机上执行地理定位通常使用三种不同的数据来源之一：

+   GPS 芯片

+   手机三角定位

+   附近的 WiFi 点

使用 GPS 芯片提供了最高级别的保真度，但也是最慢的，需要最多的电池。查看附近的 WiFi 点需要非常少的能量，速度非常快，但提供的保真度较低。

策略模式提供了一种以透明方式交换这些策略的方法。在传统的继承模型中，每个策略都会实现相同的接口，这将允许任何策略进行交换。下图显示了可以进行交换的多个策略：

![策略](img/Image00028.jpg)

选择正确的策略可以通过多种不同的方式来完成。最简单的方法是静态选择策略。这可以通过配置变量或甚至硬编码来完成。这种方法最适合策略变化不频繁或特定于单个客户或用户的情况。

或者可以对要运行策略的数据集进行分析，然后选择合适的策略。如果已知策略 A 在数据传入时比策略 B 更好，那么可以首先运行一个快速的分析传播的算法，然后选择适当的策略。

如果特定算法在某种类型的数据上失败，这也可以在选择策略时考虑进去。在 Web 应用程序中，这可以用于根据数据的形状调用不同的 API。它还可以用于在 API 端点之一宕机时提供备用机制。

另一种有趣的方法是使用渐进增强。首先运行最快且最不准确的算法以提供快速的用户反馈。同时也运行一个较慢的算法，当它完成时，优越的结果将用于替换现有的结果。这种方法经常用于上面概述的 GPS 情况。您可能会注意到，在移动设备上使用地图时，地图加载后一会儿您的位置会更新；这是渐进增强的一个例子。

最后，策略可以完全随机选择。这听起来像是一种奇怪的方法，但在比较两种不同策略的性能时可能会有用。在这种情况下，将收集关于每种方法的表现如何的统计数据，并进行分析以选择最佳策略。策略模式可以成为 A/B 测试的基础。

选择要使用的策略可以是应用工厂模式的绝佳地方。

## 实施

在维斯特洛大陆，没有飞机、火车或汽车，但仍然有各种不同的旅行方式。人们可以步行、骑马、乘船航行，甚至可以坐船沿河而下。每种方式都有不同的优点和缺点，但最终它们都能把一个人从 A 点带到 B 点。接口可能看起来像下面这样：

```js
export interface ITravelMethod{
  Travel(source: string, destination: string) : TravelResult;
}
```

旅行结果向调用者传达了一些关于旅行方式的信息。在我们的情况下，我们追踪旅行需要多长时间，风险是什么，以及费用是多少：

```js
class TravelResult {
  constructor(durationInDays, probabilityOfDeath, cost) {
    this.durationInDays = durationInDays;
    this.probabilityOfDeath = probabilityOfDeath;
    this.cost = cost;
  }
}
```

在这种情况下，我们可能希望有一个额外的方法来预测一些风险，以便自动选择策略。

实现策略就像下面这样简单：

```js
class SeaGoingVessel {
  Travel(source, destination) {
    return new TravelResult(15, .25, 500);
  }
}

class Horse {
  Travel(source, destination) {
    return new TravelResult(30, .25, 50);
  }
}

class Walk {
  Travel(source, destination) {
    return new TravelResult(150, .55, 0);
  }
}
```

在策略模式的传统实现中，每个策略的方法签名应该相同。在 JavaScript 中，函数的多余参数会被忽略，缺少的参数可以给出默认值，因此有更多的灵活性。

显然，实际实现中风险、成本和持续时间的实际计算不会硬编码。要使用这些方法，只需要做以下操作：

```js
var currentMoney = getCurrentMoney();
var strat;
if (currentMoney> 500)
  strat = new SeaGoingVessel();
else if (currentMoney> 50)
  strat = new Horse();
else
  strat = new Walk();
var travelResult = strat.Travel();
```

为了提高这种策略的抽象级别，我们可以用更一般的名称替换具体的策略，描述我们要优化的内容：

```js
var currentMoney = getCurrentMoney();
var strat;
if (currentMoney> 500)
  strat = new FavorFastestAndSafestStrategy();
else
  strat = new FavorCheapest();
var travelResult = strat.Travel();
```

策略模式在 JavaScript 中是一个非常有用的模式。我们能够使这种方法比在不使用原型继承的语言中更简单：不需要接口。我们不需要从不同的策略中返回相同形状的对象。只要调用者有点意识到返回的对象可能有额外的字段，这是一个完全合理的，虽然难以维护的方法。

# 模板方法

策略模式允许用一个互补的算法替换整个算法。经常替换整个算法是过度的：绝大部分算法在每个策略中仍然保持相同，只有特定部分有轻微的变化。

模板方法模式是一种方法，允许共享算法的一些部分，并使用不同的方法实现其他部分。这些外包部分可以由方法家族中的任何一个方法来实现：

![模板方法](img/Image00029.jpg)

模板类实现了算法的部分，并将其他部分留作抽象，以便稍后由扩展它的类来覆盖。继承层次结构可以有几层深，每个级别都实现了模板类的更多部分。

### 提示

抽象类是包含抽象方法的类。抽象方法只是没有方法体的方法。抽象类不能直接使用，必须由另一个实现抽象方法的类来扩展。抽象类可以扩展另一个抽象类，以便不需要所有方法都由扩展类实现。

这种方法将渐进增强的原则应用到算法中。我们越来越接近一个完全实现的算法，同时建立一个有趣的继承树。模板方法有助于将相同的代码保持在一个位置，同时允许一些偏差。部分实现的链可以在下图中看到：

![模板方法](img/Image00030.jpg)

重写留作抽象的方法是面向对象编程的一个典型部分。很可能你经常使用这种模式，甚至没有意识到它有一个名字。

## 实现

我已经被知情人告知，有许多不同的酿造啤酒的方法。这些啤酒在选择原料和生产方法上有所不同。事实上，啤酒甚至不需要含有啤酒花 - 它可以由任意数量的谷物制成。然而，所有啤酒之间都存在相似之处。它们都是通过发酵过程制作的，所有合格的啤酒都含有一定的酒精含量。

在维斯特洛有许多自豪地制作顶级啤酒的工匠。我们想将他们的工艺描述为一组类，每个类描述一种不同的酿造啤酒的方法。我们从一个简化的酿造啤酒的实现开始：

```js
class BasicBeer {
  Create() {
    this.AddIngredients();
    this.Stir();
    this.Ferment();
    this.Test();
    if (this.TestingPassed()) {
      this.Distribute();
    }
  }
  AddIngredients() {
    throw "Add ingredients needs to be implemented";
  }
  Stir() {
    //stir 15 times with a wooden spoon
  }
  Ferment() {
    //let stand for 30 days
  }
  Test() {
    //draw off a cup of beer and taste it
  }
  TestingPassed() {
    throw "Conditions to pass a test must be implemented";
  }
  Distribute() {
    //place beer in 50L casks
  }
}
```

由于 JavaScript 中没有抽象的概念，我们已经为必须被覆盖的各种方法添加了异常。剩下的方法可以更改，但不是必须的。树莓啤酒的实现如下所示：

```js
class RaspberryBeer extends BasicBeer {
  AddIngredients() {
    **//add ingredients, probably including raspberries** 

  }
  TestingPassed() {
    **//beer must be reddish and taste of raspberries** 

  }
}
```

在这个阶段可能会进行更具体的树莓啤酒的子类化。

在 JavaScript 中，模板方法仍然是一个相当有用的模式。在创建类时有一些额外的语法糖，但这并不是我们在之前章节中没有见过的。我唯一要提醒的是，模板方法使用继承，因此将继承类与父类紧密耦合。这通常不是一种理想的状态。

# 访问者

本节中的最后一个模式是访问者模式。访问者提供了一种将算法与其操作的对象结构解耦的方法。如果我们想对不同类型的对象集合执行某些操作，并且根据对象类型执行不同的操作，通常需要使用大量的`if`语句。

让我们立刻在维斯特洛进行一个示例。一个军队由几个不同类别的战斗人员组成（重要的是我们要政治正确，因为维斯特洛有许多著名的女战士）。然而，军队的每个成员都实现了一个名为`IMemberOfArmy`的假设接口：

```js
interface IMemberOfArmy{
  printName();
}
```

这个的简单实现可能是这样的：

```js
class Knight {
  constructor() {
    this._type = "Westeros.Army.Knight";
  }
  printName() {
    console.log("Knight");
  }
  visit(visitor) {
    visitor.visit(this);
  }
}
```

现在我们有了这些不同类型的集合，我们可以使用`if`语句只在骑士上调用`printName`函数：

```js
var collection = [];
collection.push(new Knight());
collection.push(new FootSoldier());
collection.push(new Lord());
collection.push(new Archer());

for (let i = 0; i<collection.length; i++) {
  if (typeof (collection[i]) == 'Knight')
    collection[i].printName();
  else
    console.log("Not a knight");
}
```

除非你运行这段代码，你实际上会发现我们得到的只是以下内容：

```js
Not a knight
Not a knight
Not a knight
Not a knight
```

这是因为，尽管一个对象是骑士，但它仍然是一个对象，`typeof`在所有情况下都会返回对象。

另一种方法是使用`instanceof`而不是`typeof`：

```js
var collection = [];
collection.push(new Knight());
collection.push(new FootSoldier());
collection.push(new Lord());
collection.push(new Archer());

for (var i = 0; i < collection.length; i++) {
  if (collection[i] instanceof Knight)
    collection[i].printName();
  else
    console.log("No match");
}
```

实例方法的方法在遇到使用`Object.create`语法的人时效果很好：

```js
collection.push(Object.create(Knight));
```

尽管是骑士，当被问及是否是`Knight`的实例时，它将返回`false`。

这对我们来说是一个问题。访问者模式使问题变得更加严重，因为它要求语言支持方法重载。JavaScript 实际上并不支持这一点。可以使用各种技巧来使 JavaScript 在某种程度上意识到重载的方法，但通常的建议是根本不要费心，而是创建具有不同名称的方法。

然而，我们还不要放弃这种模式；它是一个有用的模式。我们需要一种可靠地区分一种类型和另一种类型的方法。最简单的方法是在类上定义一个表示其类型的变量：

```js
var Knight = (function () {
  function Knight() {
    this._type = "Knight";
  }
  Knight.prototype.printName = function () {
    console.log("Knight");
  };
  return Knight;
})();
```

有了新的`_type`变量，我们现在可以伪造真正的方法覆盖：

```js
var collection = [];
collection.push(new Knight());
collection.push(new FootSoldier());
collection.push(new Lord());
collection.push(new Archer());

for (vari = 0; i<collection.length; i++) {
  if (collection[i]._type == 'Knight')
    collection[i].printName();
  else
    console.log("No match");
}
```

有了这种方法，我们现在可以实现一个访问者。第一步是扩展我们军队的各种成员，使其具有一个接受访问者并应用它的通用方法：

```js
var Knight = (function () {
  function Knight() {
    this._type = "Knight";
  }
  Knight.prototype.printName = function () {
    console.log("Knight");
  };
  **Knight.prototype.visit = function (visitor) {** 

 **visitor.visit(this);** 

 **};** 

  return Knight;
})();
```

现在我们需要构建一个访问者。这段代码近似于我们在前面的代码中的`if`语句：

```js
varSelectiveNamePrinterVisitor = (function () {
  function SelectiveNamePrinterVisitor() {
  }
  SelectiveNamePrinterVisitor.prototype.Visit = function (memberOfArmy) {
    if (memberOfArmy._type == "Knight") {
      this.VisitKnight(memberOfArmy);
    } else {
      console.log("Not a knight");
    }
  };

  SelectiveNamePrinterVisitor.prototype.VisitKnight = function (memberOfArmy) {
    memberOfArmy.printName();
  };
  return SelectiveNamePrinterVisitor;
})();
```

这个访问者将被用作下面这样：

```js
var collection = [];
collection.push(new Knight());
collection.push(new FootSoldier());
collection.push(new Lord());
collection.push(new Archer());
var visitor = new SelectiveNamePrinterVisitor();
for (vari = 0; i<collection.length; i++) {
  collection[i].visit(visitor);
}
```

正如您所看到的，我们已经将集合中项目的类型的决定推迟到了访问者。这将项目本身与访问者解耦，如下图所示：

![Visitor](img/Image00031.jpg)

如果我们允许访问者决定对访问对象调用哪些方法，那么就需要一些技巧。如果我们可以为访问对象提供一个恒定的接口，那么访问者只需要调用接口方法。然而，这将逻辑从访问者移到被访问的对象中，这与对象不应该知道自己是访问者的一部分的想法相矛盾。

是否值得忍受这种欺诈行为，这实际上是一个练习。就我个人而言，我倾向于避免在 JavaScript 中使用访问者模式，因为使其工作的要求很复杂且不明显。

# 提示和技巧

以下是一些关于本章中一些模式的简短提示：

+   在实现解释器模式时，您可能会被诱惑使用 JavaScript 本身作为您的 DSL，然后使用`eval`函数来执行代码。这实际上是一个非常危险的想法，因为`eval`会带来整个安全问题的世界。在 JavaScript 中使用`eval`通常被认为是非常不好的做法。

+   如果您发现自己需要审计项目中的数据更改，则可以轻松地修改备忘录模式以适应。您不仅可以跟踪状态更改，还可以跟踪更改的时间和更改者。将这些备忘录保存到磁盘的某个地方，可以让您回溯并快速构建指向更改对象的审计日志。

+   观察者模式因为监听器没有正确注销而导致内存泄漏而臭名昭著。即使在 JavaScript 这样的内存管理环境中，这种情况也可能发生。要警惕未能取消观察者。

# 总结

在本章中，我们已经看过了一堆行为模式。其中一些模式，比如观察者和迭代器，几乎每天都会用到，而另一些模式，比如解释器，你可能在整个职业生涯中只会用到几次。了解这些模式应该有助于您找到常见问题的明确定义解决方案。

大多数模式都直接适用于 JavaScript，其中一些模式，比如策略模式，在动态语言中变得更加强大。我们发现的唯一有一些限制的模式是访问者模式。缺乏静态类和多态性使得这个模式难以实现，而不破坏适当的关注点分离。

这些并不是存在的所有行为模式。编程社区在过去的二十年里一直在基于 GoF 书中的思想并识别新的模式。本书的其余部分致力于这些新识别的模式。解决方案可能是非常古老的，但直到最近才被普遍认为是常见解决方案。就我而言，这是书开始变得非常有趣的地方，因为我们开始研究不太知名和更具 JavaScript 特色的模式。
