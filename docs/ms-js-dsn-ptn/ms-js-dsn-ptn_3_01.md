# 第六章：函数式编程

函数式编程是一种与我们迄今为止专注的重度面向对象方法不同的开发方法。面向对象编程是解决许多问题的绝佳工具，但也存在一些问题。在面向对象的上下文中进行并行编程是困难的，因为状态可能会被不同的线程改变，产生未知的副作用。函数式编程不允许状态或可变变量。函数在函数式编程中充当主要的构建块。在过去可能使用变量的地方现在将使用函数。

即使在单线程程序中，函数也可能具有改变全局状态的副作用。这意味着，当调用一个未知的函数时，它可能改变程序的整个流程。这使得调试程序变得非常困难。

JavaScript 并不是一种函数式编程语言，但我们仍然可以将一些函数式原则应用到我们的代码中。我们将研究函数式空间中的许多模式：

+   函数传递

+   过滤器和管道

+   累加器

+   备忘录

+   不可变性

+   延迟实例化

# 函数式函数是无副作用的

函数式编程的核心原则之一是函数不应改变状态。函数内部的局部值可以被设置，但函数外部的任何东西都不可以改变。这种方法对于使代码更易维护非常有用。不再需要担心将数组传递给函数会对其内容造成混乱。特别是在使用不受控制的库时，这是一个问题。

JavaScript 内部没有机制可以阻止您改变全局状态。相反，您必须依赖开发人员编写无副作用的函数。这可能很困难，也可能不是，这取决于团队的成熟度。

也许并不希望将应用程序中的所有代码都放入函数中，但尽可能地分离是可取的。有一种称为命令查询分离的模式建议方法应该分为两类。要么是读取值的函数，要么是设置值的命令。二者不可兼得。保持方法按此分类有助于调试和代码重用。

无副作用函数的一个结果是，它们可以使用相同的输入被调用任意次数，结果都将是相同的。此外，由于没有状态的改变，多次调用函数不会产生任何不良副作用，除了使其运行速度变慢。

# 函数传递

在函数式编程语言中，函数是一等公民。函数可以赋值给变量并像处理其他变量一样传递。这并不是完全陌生的概念。即使像 C 这样的语言也有可以像其他变量一样处理的函数指针。C#有委托，在更近期的版本中有 lambda。最新版本的 Java 也添加了对 lambda 的支持，因为它们被证明非常有用。

JavaScript 允许将函数视为变量，甚至作为对象和字符串。这样，JavaScript 在本质上是函数式的。

由于 JavaScript 的单线程特性，回调是一种常见的约定，你几乎可以在任何地方找到它们。考虑在网页上的稍后时间调用一个函数。这是通过在 window 对象上设置超时来实现的，就像这样：

```js
setTimeout(function(){alert("Hello from the past")}, 5 * 1000);
```

设置超时函数的参数是要调用的函数和以毫秒为单位的延迟时间。

无论您在哪种 JavaScript 环境中工作，几乎不可能避免以回调函数的形式使用函数。Node.js 的异步处理模型高度依赖于能够调用函数并传递一些内容以便在以后的某个日期完成。在浏览器中调用外部资源也依赖于回调来通知调用者某些异步操作已完成。在基本的 JavaScript 中，这看起来像这样：

```js
let xmlhttp = new XMLHttpRequest()
xmlhttp.onreadystatechange = function()
if (xmlhttp.readyState==4 && xmlhttp.status==200){
  //process returned data
}
};
xmlhttp.open("GET", http://some.external.resource, true);
xmlhttp.send();
```

您可能会注意到我们在发送请求之前就分配了`onreadystatechange`函数。这是因为稍后分配可能会导致服务器在函数附加到准备状态更改之前做出响应的竞争条件。在这种情况下，我们使用内联函数来处理返回的数据。因为函数是一等公民，我们可以将其更改为以下形式：

```js
let xmlhttp;
function requestData(){
  xmlhttp = new XMLHttpRequest()
  xmlhttp.onreadystatechange=processData;
  xmlhttp.open("GET", http://some.external.resource, true);
  xmlhttp.send();
}

function processData(){
  if (xmlhttp.readyState==4 &&xmlhttp.status==200){
    //process returned data
  }
}
```

这通常是一种更清晰的方法，避免在另一个函数中执行复杂的处理。

但是，您可能更熟悉 jQuery 版本，它看起来像这样：

```js
$.getJSON('http://some.external.resource', function(json){
  //process returned data
});
```

在这种情况下，处理准备状态变化的模板已经为您处理了。如果请求数据失败，甚至还为您提供了便利：

```js
$.ajax('http://some.external.resource',
  { success: function(json){
      //process returned data
    },
    error: function(){
      //process failure
    },
    dataType: "json"
});
```

在这种情况下，我们将一个对象传递给`ajax`调用，该对象定义了许多属性。在这些属性中，成功和失败的函数回调是其中之一。将多个函数传递到另一个函数中的这种方法表明了为类提供扩展点的一种很好的方式。

很可能您以前已经看到过这种模式的使用，甚至没有意识到。将函数作为选项对象的一部分传递给构造函数是 JavaScript 库中提供扩展挂钩的常用方法。在上一章中，第五章，*行为模式*中，我们看到了对函数的一些处理，当将函数传递给观察者时。

## 实施

在维斯特洛，旅游业几乎不存在。有很多困难，如强盗杀害游客和游客卷入地区冲突。尽管如此，一些有远见的人已经开始宣传维斯特洛斯的大巡回之旅，他们将带领有能力的人游览所有主要景点。从国王之地到艾利，再到多恩的巨大山脉-这次旅行将覆盖一切。事实上，旅游局中一个相当数学倾向的成员已经开始称其为哈密顿之旅，因为它到达每个地方一次。

`HamiltonianTour`类提供了一个选项对象，允许定义一个选项对象。该对象包含可以附加回调的各种位置。在我们的情况下，它的接口看起来可能是以下样子：

```js
export class HamiltonianTourOptions{
  onTourStart: Function;
  onEntryToAttraction: Function;
  onExitFromAttraction: Function;
  onTourCompletion: Function;
}
```

完整的`HamiltonianTour`类如下所示：

```js
class HamiltonianTour {
  constructor(options) {
    this.options = options;
  }
  StartTour() {
    if (this.options.onTourStart && typeof (this.options.onTourStart) === "function")
      this.options.onTourStart();
      this.VisitAttraction("King's Landing");
      this.VisitAttraction("Winterfell");
      this.VisitAttraction("Mountains of Dorne");
      this.VisitAttraction("Eyrie");
    if (this.options.onTourCompletion && typeof (this.options.onTourCompletion) === "function")
      this.options.onTourCompletion();
  }
  VisitAttraction(AttractionName) {
    if (this.options.onEntryToAttraction && typeof (this.options.onEntryToAttraction) === "function")
      this.options.onEntryToAttraction(AttractionName);
      //do whatever one does in a Attraction
    if (this.options.onExitFromAttraction && typeof (this.options.onExitFromAttraction) === "function")
      this.options.onExitFromAttraction(AttractionName);
  }
}
```

您可以在突出显示的代码中看到我们如何检查选项，然后根据需要执行回调。只需简单地执行以下操作即可使用：

```js
var tour = new HamiltonianTour({
  onEntryToAttraction: function(cityname){console.log("I'm delighted to be in " + cityname)}});
      tour.StartTour();
```

运行此代码的输出将如下所示：

```js
I'm delighted to be in King's Landing
I'm delighted to be in Winterfell
I'm delighted to be in Mountains of Dorne
I'm delighted to be in Eyrie
```

在 JavaScript 中传递函数是解决许多问题的好方法，并且在 jQuery 等库和 express 等框架中被广泛使用。它是如此普遍地被采用，以至于使用它会增加代码的可读性障碍。

# 过滤器和管道

如果您对 Unix 命令行或者在较小程度上对 Windows 命令行有所了解，那么您可能已经使用过管道。管道由`|`字符表示，它是“获取程序 A 的输出并将其放入程序 B”的简写。这个相对简单的想法使得 Unix 命令行非常强大。例如，如果您想要列出目录中的所有文件，然后对它们进行排序并过滤出以字母`b`或`g`开头并以`f`结尾的文件，那么命令可能如下所示：

```js
ls|sort|grep "^[gb].*f$"
```

`ls`命令列出所有文件和目录，`sort`命令对它们进行排序，`grep`命令匹配文件名与正则表达式。在 Ubuntu 的`/etc`目录中运行这个命令会得到类似以下的结果：

```js
 **stimms@ubuntu1:/etc$ ls|sort|grep "^[gb].*f$"** 

blkid.conf
bogofilter.cf
brltty.conf
gai.conf
gconf
groff
gssapi_mech.conf
```

一些函数式编程语言，如 F#，提供了在函数之间进行管道传递的特殊语法。在 F#中，可以通过以下方式对列表进行偶数过滤：

```js
[1..10] |>List.filter (fun n -> n% 2 = 0);;
```

这种语法看起来非常漂亮，特别是在长链式函数中使用时。例如，将一个数字转换为浮点数，然后对其进行平方根运算，最后四舍五入，看起来会像下面这样：

```js
10.5 |> float |>Math.Sqrt |>Math.Round
```

这比 C 风格的语法更清晰，后者看起来会像下面这样：

```js
Math.Round(Math.Sqrt((float)10.5))
```

不幸的是，JavaScript 没有使用巧妙的 F#风格语法编写管道的能力，但是我们仍然可以通过方法链接来改进前面代码中显示的普通方法。

JavaScript 中的所有内容都是对象，这意味着我们可以通过向现有对象添加功能来改进它们的外观。对对象集合进行操作是函数式编程提供一些强大功能的领域。让我们首先向数组对象添加一个简单的过滤方法。您可以将这些查询视为以函数式方式编写的 SQL 数据库查询。

## 实现

我们希望提供一个对数组的每个成员进行匹配并返回一组结果的函数：

```js
Array.prototype.where = function (inclusionTest) {
  let results = [];
  for (let i = 0; i<this.length; i++) {
    if (inclusionTest(this[i]))
      results.push(this[i]);
  }
  return results;
};
```

这个看起来相当简单的函数允许我们快速过滤一个数组：

```js
var items = [1,2,3,4,5,6,7,8,9,10];
items.where(function(thing){ return thing % 2 ==0;});
```

我们返回的也是一个对象，这种情况下是一个数组对象。我们可以继续像下面这样链式调用方法：

```js
items.where(function(thing){ return thing % 2 ==0;})
  .where(function(thing){ return thing % 3 == 0;});
```

结果是一个只包含数字 6 的数组，因为它是 1 到 10 之间唯一既是偶数又可被三整除的数字。返回原始对象的修改版本而不改变原始对象的方法称为流畅接口。通过不改变原始的项目数组，我们为变量引入了一定程度的不可变性。

如果我们向数组扩展库添加另一个函数，我们就可以开始看到这些管道有多么有用：

```js
Array.prototype.select=function(projection){
  let results = [];
  for(let i = 0; i<this.length;i++){
    results.push(projection(this[i]));
  }
  return results;
};
```

这个扩展允许根据任意投影函数对原始项目进行投影。给定一组包含 ID 和名称的对象，我们可以使用我们的流畅扩展到数组来执行复杂的操作：

```js
let children = [{ id: 1, Name: "Rob" },
{ id: 2, Name: "Sansa" },
{ id: 3, Name: "Arya" },
{ id: 4, Name: "Brandon" },
{ id: 5, Name: "Rickon" }];
let filteredChildren = children.where(function (x) {
  return x.id % 2 == 0;
}).select(function (x) {
  return x.Name;
});
```

这段代码将构建一个新数组，其中只包含具有偶数 ID 的子项，而不是完整的对象，数组将只包含它们的名称：`Sansa`和`Brandon`。对于熟悉.Net 的人来说，这些函数可能看起来非常熟悉。.Net 上的**语言集成查询**（**LINQ**）库提供了类似命名的受函数启发的函数，用于操作集合。

以这种方式链接函数既更容易理解，也更容易构建，因为避免了临时变量，代码更加简洁。考虑使用循环和临时变量重新实现前面的示例：

```js
let children = [{ id: 1, Name: "Rob" },
{ id: 2, Name: "Sansa" },
{ id: 3, Name: "Arya" },
{ id: 4, Name: "Brandon" },
{ id: 5, Name: "Rickon" }];
let evenIds = [];
for(let i=0; i<children.length;i++)
{
  if(children[i].id%2==0)
    evenIds.push(children[i]);
}
let names = [];
for(let i=0; i< evenIds.length;i++)
{
  names.push(evenIds[i].name);
}
```

许多 JavaScript 库，比如 d3，都是为了鼓励这种编程方式而构建的。起初，遵循这种约定创建的代码似乎很糟糕，因为行长非常长。我认为这是行长不是一个很好的衡量复杂性的工具，而不是这种方法的实际问题。

# 累加器

我们已经研究了一些简单的数组函数，它们为数组添加了过滤和管道。另一个有用的工具是累加器。累加器通过对集合进行迭代来帮助构建单个结果。许多常见的操作，比如对数组元素求和，都可以使用累加器来实现，而不是使用循环。

递归在函数式编程语言中很受欢迎，其中许多语言实际上提供了一种称为“尾递归优化”的优化。支持这一点的语言为使用递归的函数提供了优化，其中堆栈帧被重用。这是非常高效的，可以轻松地替代大多数循环。关于 JavaScript 解释器是否支持尾递归优化的细节还不清楚。在大多数情况下，似乎并不支持，但我们仍然可以利用递归。

`for`循环的问题在于循环中的控制流是可变的。考虑这个相当容易犯的错误：

```js
let result = "";
let multiArray = [[1,2,3], ["a", "b", "c"]];
for(vari=0; i<multiArray.length; i++)
  for(var j=0; i<multiArray[i].length; j++)
    result += multiArray[i][j];
```

你发现错误了吗？我尝试了几次才得到一个可行的版本，我才发现了问题。问题在于第二个循环中的循环计数器，它应该是这样的：

```js
let result = "";
let multiArray = [[1,2,3], ["a", "b", "c"]];
for(let i=0; i<multiArray.length; i++)
  for(let j=0; j<multiArray[i].length; j++)
    result +=multiArray[i][j];
```

显然，通过更好的变量命名可以在一定程度上缓解这个问题，但我们希望完全避免这个问题。

相反，我们可以利用累加器，这是一个将集合中的多个值组合成单个值的工具。我们错过了 Westeros 的一些模式，所以让我们回到我们的神话般的例子。战争花费了大量的金钱，但幸运的是有大量的农民来交税，为领主们的王位之争提供资金。

## 实施

我们的农民由一个简单的模型代表，看起来像下面这样：

```js
let peasants = [
  {name: "Jory Cassel", taxesOwed: 11, bankBalance: 50},
  {name: "VardisEgen", taxesOwed: 15, bankBalance: 20}];
```

在这组农民中，我们有一个看起来像下面这样的累加器：

```js
TaxCollector.prototype.collect = function (items, value, projection) {
  if (items.length> 1)
    return projection(items[0]) + this.collect(items.slice(1), value, projection);
  return projection(items[0]);
};
```

这段代码接受一个项目列表，一个累加器值，以及一个将值投影到累加中的函数。

投影函数看起来像下面这样：

```js
function (item) {
  return Math.min(item.moneyOwed, item.bankBalance);
}
```

为了激活这个函数，我们只需要传入一个累加器的初始值以及数组和投影。激活值会有所不同，但往往是一个身份；在字符串累加器的情况下是一个空字符串，在数学累加器的情况下是 0 或 1。

每次通过累加器，我们都会缩小我们操作的数组的大小。所有这些都是在没有一个可变变量的情况下完成的。

内部累积可以是任何你喜欢的函数：字符串追加，加法，或者更复杂的东西。累加器有点像访问者模式，只是在累加器内部修改集合中的值是不被赞同的。记住，函数式编程是无副作用的。

# 记忆化

不要与记忆混淆，记忆化是一个特定术语，用于保留函数中先前计算的值。

正如我们之前看到的，无副作用的函数可以被多次调用而不会引起问题。与此相对的是，函数也可以被调用的次数少于需要的次数。考虑一个复杂或者至少耗时的数学运算的昂贵函数。我们知道函数的结果完全取决于函数的输入。因此，相同的输入将始终产生相同的输出。那么，为什么我们需要多次调用函数呢？如果我们保存函数的输出，我们可以检索到它，而不是重新进行耗时的数学运算。

以空间换时间是一个经典的计算科学问题。通过缓存结果，我们可以使应用程序更快，但会消耗更多的内存。决定何时进行缓存，何时简单地重新计算结果是一个困难的问题。

## 实施

在维斯特洛大陆，被称为大师的学者们长期以来对一个数字序列产生了浓厚的兴趣，这个序列似乎在自然界中频繁出现。一个奇怪的巧合是，他们称这个序列为斐波那契数列。它的定义是将序列中的前两个项相加以得到下一个项。这个序列的起始项被定义为 0、1、1。所以要得到下一个项，我们只需将 1 和 1 相加得到 2。下一个项将 2 和 1 相加得到 3，依此类推。找到序列的任意成员需要找到前两个成员，因此可能需要进行一些计算。

在我们的世界中，我们已经发现了一个避免大部分计算的封闭形式，但在维斯特洛还没有做出这样的发现。

一个朴素的方法是简单地计算每个项，如下所示：

```js
let Fibonacci = (function () {
  function Fibonacci() {
  }
  Fibonacci.prototype.NaieveFib = function (n) {
    if (n == 0)
      return 0;
    if (n <= 2)
      return 1;
    return this.NaieveFib(n - 1) + this.NaieveFib(n - 2);
  };
  return Fibonacci;
})();
```

这个解决方案对于小数字（比如 10）非常快。然而，对于更大的数字，比如大于 40，速度会明显变慢。这是因为基本情况被调用了 102,334,155 次。

让我们看看是否可以通过备忘录一些值来改善情况：

```js
let Fibonacci = (function () {
  function Fibonacci() {
    this.memoizedValues = [];
  }

  Fibonacci.prototype.MemetoFib = function (n) {
    if (n == 0)
      return 0;
    if (n <= 2)
      return 1;
    if (!this. memoizedValues[n])
      this. memoizedValues[n] = this.MemetoFib(n - 1) + this.MemetoFib(n - 2);
    return this. memoizedValues[n];
  };
  return Fibonacci;
})();
```

我们刚刚对我们遇到的每个项目进行了备忘录。事实证明，对于这个算法，我们存储了*n+1*个项目，这是一个相当不错的折衷。没有备忘录，计算第 40 个斐波那契数需要 963 毫秒，而备忘录版本只需要 11 毫秒。当函数变得更复杂时，差异会更加明显。备忘录版本的斐波那契数列 140 只需要 12 毫秒，而朴素版本……嗯，已经过了一天，它还在运行。

备忘录的最大优点是，对具有相同参数的函数的后续调用将非常快，因为结果已经计算过了。

在我们的例子中，只需要一个非常小的缓存。在更复杂的例子中，很难知道缓存应该有多大，或者一个值需要重新计算的频率。理想情况下，您的缓存应该足够大，以至于总是有足够的空间来放更多的结果。然而，这可能并不现实，需要做出艰难的决定，即哪些缓存成员应该被移除以节省空间。有很多方法可以执行缓存失效。有人说，缓存失效是计算科学中最棘手的问题之一，原因是我们实际上在试图预测未来。如果有人完善了一种预测未来的方法，那么他们很可能会将自己的技能应用于比缓存失效更重要的领域。两个选择是依赖于最近最少使用的缓存成员或最不经常使用的成员。问题的形状可能决定了更好的策略。

备忘录是加速需要多次执行的计算或者有共同子计算的计算的一个奇妙工具。人们可以将备忘录视为缓存的一种特殊情况，这是在构建网络服务器或浏览器时常用的技术。在更复杂的 JavaScript 应用程序中探索备忘录是值得的。

# 不变性

函数式编程的基石之一是所谓的变量只能被赋值一次。这就是不变性。ECMAScript 2015 支持一个新关键字，`const`。`const`关键字可以像`var`一样使用，只是用`const`赋值的变量将是不可变的。例如，以下代码显示了一个变量和一个常量，它们都以相同的方式被操作：

```js
let numberOfQueens = 1;
const numberOfKings = 1;
numberOfQueens++;
numberOfKings++;
console.log(numberOfQueens);
console.log(numberOfKings);
```

运行的输出如下：

```js
2
1
```

正如你所看到的，常数和变量的结果是不同的。

如果您使用的是不支持`const`的旧浏览器，那么`const`对您来说将不可用。一个可能的解决方法是使用更广泛采用的`Object.freeze`功能：

```js
let consts = Object.freeze({ pi : 3.141});
consts.pi = 7;
console.log(consts.pi);//outputs 3.141
```

正如您所看到的，这里的语法并不是很用户友好。另一个问题是，尝试对已分配的`const`进行赋值只会静默失败，而不是抛出错误。以这种方式静默失败绝对不是一种理想的行为；应该抛出完整的异常。如果启用了严格模式，ECMAScript 5 中添加了更严格的解析模式，并且实际上会抛出异常：

```js
"use strict";
var consts = Object.freeze({ pi : 3.141});
consts.pi = 7;
```

前面的代码将抛出以下错误：

```js
consts.pi = 7;
          ^
TypeError: Cannot assign to read only property 'pi' of #<Object>
```

另一种方法是我们之前提到的`object.Create`语法。在创建对象的属性时，可以指定`writable: false`来使属性不可变：

```js
var t = Object.create(Object.prototype,
{ value: { writable: false,
  value: 10}
});
t.value = 7;
console.log(t.value);//prints 10
```

然而，即使在严格模式下，当尝试写入不可写属性时也不会抛出异常。因此，我认为`const`关键字并不完美地实现了不可变对象。最好使用 freeze。

# 延迟实例化

如果您进入一个高端咖啡店并点一杯过于复杂的饮料（大杯奶茶拿铁，3 泵，脱脂牛奶，少水，无泡沫，超热，有人吗？），那么这种饮料将是临时制作的，而不是提前制作的。即使咖啡店知道当天会有哪些订单，他们也不会提前制作所有的饮料。首先，因为这会导致大量的毁坏、冷却的饮料，其次，如果他们必须等待当天所有订单完成，第一个顾客要等很长时间才能拿到他们的订单。

咖啡店遵循按需制作饮料的方法。他们在点单时制作饮料。我们可以通过使用一种称为延迟实例化或延迟初始化的技术来将类似的方法应用到我们的代码中。

考虑一个昂贵的创建对象；也就是说，创建对象需要很长时间。如果我们不确定对象的值是否需要，我们可以推迟到以后再完全创建它。

## 实施

让我们来看一个例子。Westeros 并不是很喜欢昂贵的咖啡店，但他们确实喜欢好的面包店。这家面包店提前接受不同种类的面包请求，然后一旦有订单，就会一次性烘烤所有面包。然而，创建面包对象是一个昂贵的操作，所以我们希望推迟到有人来取面包时再进行：

```js
class Bakery {
  constructor() {
    this.requiredBreads = [];
  }
  orderBreadType(breadType) {
    this.requiredBreads.push(breadType);
  }
}
```

首先，我们创建一个要根据需要创建的面包类型列表。通过订购面包类型，这个列表会被追加：

```js
var Bakery = (function () {
  function Bakery() {
    this.requiredBreads = [];
  }
  Bakery.prototype.orderBreadType = function (breadType) {
    this.requiredBreads.push(breadType);
  };
```

这样就可以快速地将面包添加到所需的面包列表中，而不必为每个面包的创建付出代价。

现在当调用`pickUpBread`时，我们将实际创建面包：

```js
pickUpBread(breadType) {
  console.log("Picup of bread " + breadType + " requested");
  if (!this.breads) {
    this.createBreads();
  }
  for (var i = 0; i < this.breads.length; i++) {
    if (this.breads[i].breadType == breadType)
      return this.breads[i];
  }
}
createBreads() {
  this.breads = [];
  for (var i = 0; i < this.requiredBreads.length; i++) {
    this.breads.push(new Bread(this.requiredBreads[i]));
  }
}
```

在这里，我们调用了一系列操作：

```js
let bakery = new Westeros.FoodSuppliers.Bakery();
bakery.orderBreadType("Brioche");
bakery.orderBreadType("Anadama bread");
bakery.orderBreadType("Chapati");
bakery.orderBreadType("Focaccia");

console.log(bakery.pickUpBread("Brioche").breadType + "picked up");
```

这将导致以下结果：

```js
Pickup of bread Brioche requested.
Bread Brioche created.
Bread Anadama bread created.
Bread Chapati created.
Bread Focaccia created.
Brioche picked up
```

您可以看到实际面包的收集是在取货后进行的。

延迟实例化可以用来简化异步编程。Promise 是简化 JavaScript 中常见的回调的一种方法。Promise 是一个包含状态和结果的对象。首次调用时，promise 处于未解决状态；一旦`async`操作完成，状态就会更新为完成，并填充结果。您可以将结果视为延迟实例化。我们将在第九章 *Web Patterns*中更详细地讨论 promise 和 promise 库。

懒惰可以节省大量时间，因为创建昂贵的对象最终可能根本不会被使用。

# 提示和技巧

尽管回调是处理 JavaScript 中异步方法的标准方式，但它们很容易变得混乱。有许多方法可以解决这种混乱的代码：promise 库提供了一种更流畅的处理回调的方式，未来版本的 JavaScript 可能会采用类似于 C# `async/await`语法的方法。

我真的很喜欢累加器，但它们在内存使用方面可能效率低下。缺乏尾递归意味着每次通过都会增加另一个堆栈帧，因此这种方法可能会导致内存压力。在这种情况下，所有事情都是在内存和代码可维护性之间进行权衡。

# 总结

JavaScript 不是一种函数式编程语言。这并不是说不可能将一些函数式编程的思想应用到它上面。这些方法可以使代码更清晰、更易于调试。有些人甚至可能会认为问题的数量会减少，尽管我从未见过任何令人信服的研究。

在本章中，我们研究了六种不同的模式。延迟实例化、记忆化和不可变性都是创建模式。函数传递既是结构模式，也是行为模式。累加器也是行为模式。过滤器和管道实际上并不属于 GoF 的任何一类，因此可以将它们视为一种样式模式。

在下一章中，我们将研究一些在应用程序中划分逻辑和呈现的模式。随着 JavaScript 应用程序的增长，这些模式变得更加重要。

