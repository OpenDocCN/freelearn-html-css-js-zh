# 第十四章：ECMAScript-2015/2016 今天的解决方案

在本书中，我无法计算提到 JavaScript 即将推出的版本的次数，可以放心，这个数字很大。令人有些沮丧的是，语言没有跟上应用程序开发人员的要求。我们讨论过的许多方法在 JavaScript 的新版本中变得不再必要。然而，有一些方法可以让下一个版本的 JavaScript 在今天就能运行。

在本章中，我们将重点讨论其中的一些：

+   TypeScript

+   BabelJS

# TypeScript

编译成 JavaScript 的语言并不少。CoffeeScript 可能是这些语言中最知名的一个例子，尽管将 Java 编译成 JavaScript 的 Google Web Toolkit 也曾经非常流行。微软在 2012 年发布了一种名为 TypeScript 的语言，以设计成 JavaScript 的超集，就像 C++是 C 的超集一样。这意味着所有语法上有效的 JavaScript 代码也是 TypeScript 代码。

微软自身在一些较大的网络属性中大量使用 TypeScript。Office 365 和 Visual Studio Online 都有大量用 TypeScript 编写的代码库。这些项目实际上早于 TypeScript 很长时间。据报道，从 JavaScript 过渡到 TypeScript 相当容易，因为它是 JavaScript 的超集。

TypeScript 的设计目标之一是尽可能与 ECMAScript-2015 和未来版本兼容。这意味着 TypeScript 支持 ECMAScript-2016 的一些特性，尽管当然不是全部，以及 ECMAScript-2015 的大部分特性。TypeScript 部分支持的 ECMAScript-2016 的两个重要特性是装饰器和 async/await。

## 装饰器

在早些章节中，我们探讨了**面向方面的编程**（**AOP**）。使用 AOP，我们用拦截器包装函数。装饰器提供了一种简单的方法来做到这一点。假设我们有一个在维斯特洛传递消息的类。显然，那里没有电话或互联网，因此消息是通过乌鸦传递的。如果我们能监视这些消息将会非常有帮助。我们的`CrowMessenger`类看起来像下面这样：

```js
class CrowMessenger {
  @spy
  public SendMessage(message: string) {
    console.log(`Send message is ${message}`);
  }
}
var c = new CrowMessenger();
var r = c.SendMessage("Attack at dawn");
```

您可能会注意到`SendMessage`方法上的`@spy`注释。这只是另一个拦截和包装函数的函数。在 spy 内部，我们可以访问函数描述符。正如您在以下代码中所看到的，我们获取描述符并操纵它以捕获发送到`CrowMessenger`类的参数：

```js
function spy(target: any, key: string, descriptor?: any) {
  if(descriptor === undefined) {
    descriptor = Object.getOwnPropertyDescriptor(target, key);
  }
  var originalMethod = descriptor.value;

  descriptor.value =  function (...args: any[]) {
    var arguments = args.map(a => JSON.stringify(a)).join();
    var result = originalMethod.apply(this, args);
    console.log(`Message sent was: ${arguments}`);
    return result;
  }
  return descriptor;
}
```

间谍显然对于测试函数非常有用。我们不仅可以在这里监视值，还可以替换函数的输入和输出。考虑以下内容：

```js
descriptor.value =  function (...args: any[]) {
  var arguments = args.map(a => JSON.stringify(a)).join();
  **var result = "Retreat at once";** 

  console.log(`Message sent was: ${arguments}`);
  return result;
}
```

装饰器可以用于除 AOP 之外的其他目的。例如，您可以将对象的属性注释为可序列化，并使用注释来控制自定义 JSON 序列化。我怀疑随着装饰器的支持，装饰器将变得更加有用和强大。已经有 Angular 2.0 在大量使用装饰器。

## 异步/等待

在第七章中，*反应式编程*，我们谈到了 JavaScript 编程的回调性质使代码非常混乱。尝试将一系列异步事件链接在一起时，这一点表现得更加明显。我们很快陷入了一个看起来像下面这样的代码陷阱：

```js
$.post("someurl", function(){
  $.post("someotherurl", function(){
    $.get("yetanotherurl", function(){
      navigator.geolocation.getCurrentPosition(function(location){
        ...
      })
    })
  })
})
```

这段代码不仅难以阅读，而且几乎不可能理解。从 C#借鉴的异步/等待语法允许以更简洁的方式编写代码。在幕后，使用（或滥用，如果您愿意）生成器来创建真正的异步/等待的印象。让我们看一个例子。在前面的代码中，我们使用了返回客户端位置的地理位置 API。它是异步的，因为它与用户的机器进行一些 IO 以获取真实世界的位置。我们的规范要求我们获取用户的位置，将其发送回服务器，然后获取图像：

```js
navigator.geolocation.getCurrentPosition(function(location){
  $.post("/post/url", function(result){
    $.get("/get/url", function(){
   });
  });
});
```

如果我们现在引入异步/等待，代码可以变成以下形式：

```js
async function getPosition(){
  return await navigator.geolocation.getCurrentPosition();
}
async function postUrl(geoLocationResult){
  return await $.post("/post/url");
}
async function getUrl(postResult){
  return await $.get("/get/url");
}
async function performAction(){
  var position = await getPosition();
  var postResult = await postUrl(position);
  var getResult = await getUrl(postResult);
}
```

这段代码假设所有`async`响应都返回包含状态和结果的 promise 构造。事实上，大多数`async`操作并不返回 promise，但有库和工具可以将回调转换为 promise。正如您所看到的，这种语法比回调混乱要清晰得多，更容易理解。

## 类型

除了我们在前一节中提到的 ECMAScript-2016 功能之外，TypeScript 还具有一个非常有趣的类型系统。JavaScript 最好的部分之一是它是一种动态类型语言。我们反复看到，不受类型负担的好处节省了我们的时间和代码。TypeScript 中的类型系统允许您根据需要使用尽可能多或尽可能少的类型。您可以使用以下语法声明变量的类型：

```js
var a_number: number;
var a_string: string;
var an_html_element: HTMLElement;
```

一旦变量被分配了一个类型，TypeScript 编译器将使用它不仅来检查该变量的使用情况，还将推断出可能从该类派生的其他类型。例如，考虑以下代码：

```js
var numbers: Array<number> = [];
numbers.push(7);
numbers.push(9);
var unknown = numbers.pop();
```

在这里，TypeScript 编译器将知道`unknown`是一个数字。如果您尝试将其用作其他类型，比如以下字符串：

```js
console.log(unknown.substr(0,1));
```

然后编译器会抛出一个错误。然而，你不需要为任何变量分配类型。这意味着你可以调整类型检查的程度。虽然听起来很奇怪，但实际上这是一个很好的解决方案，可以在不失去 JavaScript 的灵活性的情况下引入类型检查的严谨性。类型只在编译期间强制执行，一旦代码编译成 JavaScript，与字段相关的类型信息的任何提示都会消失。因此，生成的 JavaScript 实际上非常干净。

如果你对类型系统感兴趣，知道逆变等词汇，并且可以讨论逐渐类型的各个层次，那么 TypeScript 的类型系统可能值得你花时间去研究。

本书中的所有示例最初都是用 TypeScript 编写的，然后编译成 JavaScript。这样做是为了提高代码的准确性，通常也是为了让我不那么频繁地搞砸。我非常偏袒，但我认为 TypeScript 做得非常好，肯定比纯 JavaScript 写得好。

未来版本的 JavaScript 中不支持类型。因此，即使未来版本的 JavaScript 带来了许多变化，我仍然相信 TypeScript 在提供编译时类型检查方面有其存在的价值。每当我写 TypeScript 时，类型检查器总是让我惊讶，因为它多次帮我避免了愚蠢的错误。

# BabelJS

TypeScript 的另一种选择是使用 BabelJS 编译器。这是一个开源项目，用于将 ECMAScript-2015 及更高版本转换为等效的 ECMAScript 5 JavaScript。ECMAScript-2015 中的许多更改都是语法上的美化，因此它们实际上可以表示为 ECMAScript 5 JavaScript，尽管不像那么简洁或令人愉悦。我们已经看到在 ES 5 中使用类似类的结构。BabelJS 是用 JavaScript 编写的，这意味着可以直接在网页上从 ECMAScript-2015 编译到 ES 5。当然，与编译器的趋势一样，BabelJS 的源代码使用了 ES 6 构造，因此必须使用 BabelJS 来编译 BabelJS。

在撰写本文时，BabelJS 支持的 ES6 函数列表非常广泛：

+   箭头函数

+   类

+   计算属性名称

+   默认参数

+   解构赋值

+   迭代器和 for of

+   生成器理解

+   生成器

+   模块

+   数字文字

+   属性方法赋值

+   对象初始化程序简写

+   剩余参数

+   扩展

+   模板文字

+   承诺

BabelJS 是一个多用途的 JavaScript 编译器，因此编译 ES-2015 代码只是它可以做的许多事情之一。有许多插件提供各种有趣的功能。例如，“内联环境变量”插件插入编译时变量，允许根据环境进行条件编译。

已经有大量关于这些功能如何工作的文档可用，因此我们不会详细介绍它们。

如果您已经安装了 node 和 npm，那么设置 Babel JS 就是一个相当简单的练习：

```js
 **npm install –g babel-cli** 

```

这将创建一个 BabelJS 二进制文件，可以进行编译，如下所示：

```js
 **babel  input.js --o output.js** 

```

对于大多数用例，您将希望使用构建工具，如 Gulp 或 Grunt，它们可以一次编译多个文件，并执行任意数量的后编译步骤。

## 类

到目前为止，你应该已经厌倦了阅读关于在 JavaScript 中创建类的不同方法。不幸的是，你是我写这本书的人，所以让我们看一个最后的例子。我们将使用之前的城堡例子。

BabelJS 不支持文件内的模块。相反，文件被视为模块，这允许以一种类似于`require.js`的方式动态加载模块。因此，我们将从我们的堡垒中删除模块定义，只使用类。TypeScript 中存在但 ES 6 中不存在的另一个功能是使用`public`作为参数前缀，使其成为类的公共属性。相反，我们使用`export`指令。

一旦我们做出了这些更改，源 ES6 文件看起来像这样：

```js
export class BaseStructure {
  constructor() {
    console.log("Structure built");
  }
}

export class Castle extends BaseStructure {
  constructor(name){
    this.name = name;
    super();
  }
  Build(){
    console.log("Castle built: " + this.name);
  }
}
```

生成的 ES 5 JavaScript 看起来像这样：

```js
"use strict";

var _createClass = function () { function defineProperties(target, props) { for (var i = 0; i < props.length; i++) { var descriptor = props[i]; descriptor.enumerable = descriptor.enumerable || false; descriptor.configurable = true; if ("value" in descriptor) descriptor.writable = true; Object.defineProperty(target, descriptor.key, descriptor); } } return function (Constructor, protoProps, staticProps) { if (protoProps) defineProperties(Constructor.prototype, protoProps); if (staticProps) defineProperties(Constructor, staticProps); return Constructor; }; }();

Object.defineProperty(exports, "__esModule", {
  value: true
});

function _possibleConstructorReturn(self, call) { if (!self) { throw new ReferenceError("this hasn't been initialised - super() hasn't been called"); } return call && (typeof call === "object" || typeofcall === "function") ? call : self; }

function _inherits(subClass, superClass) { if (typeof superClass !== "function" && superClass !== null) { throw new TypeError("Super expression must either be null or a function, not " + typeof superClass); } subClass.prototype = Object.create(superClass && superClass.prototype, { constructor: { value: subClass, enumerable: false, writable: true, configurable: true } }); if (superClass) Object.setPrototypeOf ? Object.setPrototypeOf(subClass, superClass) : subClass.__proto__ = superClass; }

function _classCallCheck(instance, Constructor) { if (!(instance instanceof Constructor)) { throw new TypeError("Cannot call a class as a function"); } }

var BaseStructure = exports.BaseStructure = function BaseStructure() {
  _classCallCheck(this, BaseStructure);
  console.log("Structure built");
};

var Castle = exports.Castle = function (_BaseStructure) {
  _inherits(Castle, _BaseStructure);
  function Castle(name) {
    _classCallCheck(this, Castle);
    var _this = _possibleConstructorReturn(this, Object.getPrototypeOf(Castle).call(this));
    _this.name = name;
    return _this;
  }
  _createClass(Castle, [{
    key: "Build",
    value: function Build() {
      console.log("Castle built: " + this.name);
    }
  }]);
  return Castle;
}(BaseStructure);
```

立即就会发现，BabelJS 生成的代码不如 TypeScript 中的代码干净。您可能还注意到有一些辅助函数用于处理继承场景。还有许多提到`"use strict";`。这是对 JavaScript 引擎的指示，它应该以严格模式运行。

严格模式阻止了许多危险的 JavaScript 实践。例如，在一些 JavaScript 解释器中，可以在不事先声明变量的情况下使用它是合法的：

```js
x = 22;
```

如果`x`之前未声明，这将抛出错误：

```js
var x = 22;
```

不允许在对象中复制属性，也不允许重复声明参数。还有许多其他实践方法，`"use strict";`会将其视为错误。我认为`"use strict";`类似于将所有警告视为错误。它可能不像 GCC 中的`-Werror`那样完整，但在新的 JavaScript 代码库中使用严格模式仍然是一个好主意。BabelJS 只是为您强制执行这一点。

## 默认参数

ES 6 中一个不是很重要但确实很好的功能是默认参数的引入。在 JavaScript 中一直可以调用函数而不指定所有参数。参数只是从左到右填充，直到没有更多的值，并且所有剩余的参数都被赋予 undefined。

默认参数允许为未填充的参数设置一个值，而不是 undefined：

```js
function CreateFeast(meat, drink = "wine"){
  console.log("The meat is: " + meat);
  console.log("The drink is: " + drink);
}
CreateFeast("Boar", "Beer");
CreateFeast("Venison");
```

这将输出以下内容：

```js
The meat is: Boar
The drink is: Beer
The meat is: Venison
The drink is: wine
```

生成的 JavaScript 代码实际上非常简单：

```js
"use strict";
function CreateFeast(meat) {
  var drink = arguments.length <= 1 || arguments[1] === undefined ? "wine" : arguments[1];
  console.log("The meat is: " + meat);
  console.log("The drink is: " + drink);
}
CreateFeast("Boar", "Beer");
CreateFeast("Venison");
```

## 模板文字

表面上看，模板文字似乎是解决 JavaScript 中缺乏字符串插值的解决方案。在某些语言中，比如 Ruby 和 Python，您可以直接将周围代码中的替换插入到字符串中，而无需将它们传递给某种字符串格式化函数。例如，在 Ruby 中，您可以执行以下操作：

```js
name= "Stannis";
print "The one true king is ${name}"
```

这将把`${name}`参数绑定到周围范围内的名称。

ES6 支持模板文字，允许在 JavaScript 中实现类似的功能：

```js
var name = "Stannis";
console.log(`The one true king is ${name}`);
```

可能很难看到，但该字符串实际上是用反引号而不是引号括起来的。要绑定到作用域的标记由`${}`表示。在大括号内，您可以放置复杂的表达式，例如：

```js
var army1Size = 5000;
var army2Size = 3578;
console.log(`The surviving army will be ${army1Size > army2Size ? "Army 1": "Army 2"}`);
```

这段代码的 BabelJS 编译版本只是简单地用字符串拼接来替代字符串插值：

```js
var army1Size = 5000;
var army2Size = 3578;
console.log(("The surviving army will be " + (army1Size > army2Size ? "Army 1" : "Army 2")));
```

模板文字还解决了许多其他问题。模板文字内部的换行符是合法的，这意味着您可以使用模板文字来创建多行字符串。

考虑到多行字符串的想法，模板文字似乎对构建特定领域语言很有用：这是我们已经多次看到的一个主题。DSL 可以嵌入到模板文字中，然后从外部插入值。例如，可以使用它来保存 HTML 字符串（当然是 DSL）并从模型中插入值。这些可能取代今天使用的一些模板工具。

## 使用 let 进行块绑定

JavaScript 中的变量作用域很奇怪。如果在块内定义变量，比如在`if`语句内部，那么该变量仍然可以在块外部使用。例如，看下面的代码：

```js
if(true)
{
  var outside = 9;
}
console.log(outside);
```

这段代码将打印`9`，即使外部变量显然超出了范围。至少如果你假设 JavaScript 像其他 C 语法语言一样支持块级作用域，那么它就超出了范围。JavaScript 中的作用域实际上是函数级的。在`if`和`for`循环语句附加的代码块中声明的变量被提升到函数的开头。这意味着它们在整个函数的范围内保持有效。

ES 6 引入了一个新关键字`let`，它将变量的作用域限制在块级。这种类型的变量非常适合在循环中使用，或者在`if`语句中保持正确的变量值。Traceur 实现了对块级作用域变量的支持。然而，由于性能影响，目前该支持是实验性的。

考虑以下代码：

```js
if(true)
{
  var outside = 9;
  et inside = 7;
}
console.log(outside);
console.log(inside);
```

这将编译为以下内容：

```js
var inside$__0;
if (true) {
  var outside = 9;
  inside$__0 = 7;
}
console.log(outside);
console.log(inside);
```

您可以看到内部变量被替换为重命名的变量。一旦离开代码块，变量就不再被替换。运行这段代码时，当`console.log`方法发生时，内部变量将报告为未定义。

## 在生产中

BabelJS 是一个非常强大的工具，可以在今天复制下一个版本的 JavaScript 的许多结构和特性。然而，生成的代码永远不会像原生支持这些结构那样高效。值得对生成的代码进行基准测试，以确保它继续满足项目的性能要求。

# 技巧和窍门

JavaScript 中有两个优秀的库可以在集合功能上进行函数式操作：Underscore.js 和 Lo-Dash。与 TypeScript 或 BabelJS 结合使用时，它们具有非常愉快的语法，并提供了巨大的功能。

例如，使用 Underscore 查找满足条件的集合成员的所有成员看起来像下面这样：

```js
_.filter(collection, (item) => item.Id > 3);
```

这段代码将找到所有 ID 大于`3`的项目。

这两个库中的任何一个都是我在新项目中添加的第一件事。Underscore 实际上已经与 backbone.js 捆绑在一起，这是一个 MVVM 框架。

Grunt 和 Gulp 的任务用于编译用 TypeScript 或 BabelJS 编写的代码。当然，微软的开发工具链中也对 TypeScript 有很好的支持，尽管 BabelJS 目前没有直接支持。

# 总结

随着 JavaScript 功能的扩展，对第三方框架甚至转译器的需求开始减少。语言本身取代了许多这些工具。像 jQuery 这样的工具的最终目标是它们不再需要，因为它们已经被吸收到生态系统中。多年来，Web 浏览器的速度一直无法跟上人们愿望变化的速度。

AngularJS 的下一个版本背后有很大的努力，但正在努力使新组件与即将到来的 Web 组件标准保持一致。Web 组件不会完全取代 AngularJS，但 Angular 最终将简单地增强 Web 组件。

当然，认为不需要任何框架或工具的想法是荒谬的。总会有新的解决问题的方法和新的库和框架出现。人们对如何解决问题的看法也会有所不同。这就是为什么市场上存在各种各样的 MVVM 框架的原因。

如果您使用 ES6 构造来处理 JavaScript，那么工作将会更加愉快。有几种可能的方法来做到这一点，哪种方法最适合您的具体问题是需要更仔细调查的问题。
