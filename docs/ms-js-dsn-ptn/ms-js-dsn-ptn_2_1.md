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

