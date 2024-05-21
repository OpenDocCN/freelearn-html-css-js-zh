# 第三章：设置函数式编程环境

# 介绍

我们是否需要了解高级数学——范畴论、Lambda 演算、多态——才能使用函数式编程编写应用程序？我们是否需要重新发明轮子？对这两个问题的简短回答都是*不*。

在本章中，我们将尽力调查一切可能影响我们在 JavaScript 中编写函数式应用程序的方式。

+   库

+   工具包

+   开发环境

+   编译为 JavaScript 的函数式语言

+   等等

请理解，JavaScript 的函数库当前的格局是非常不确定的。就像计算机编程的所有方面一样，社区可能会在一瞬间发生变化；新的库可能会被采用，旧的库可能会被抛弃。例如，在撰写本书的过程中，流行且稳定的`Node.js`平台已被其开源社区分叉。它的未来是模糊的。

因此，从本章中获得的最重要的概念不是如何使用当前的库进行函数式编程，而是如何使用任何增强 JavaScript 函数式编程方法的库。本章不会专注于只有一两个库，而是将尽可能多地探索所有存在于 JavaScript 中的函数式编程风格。

# JavaScript 的函数库

据说每个函数式程序员都会编写自己的函数库，函数式 JavaScript 程序员也不例外。如今的开源代码共享平台，如 GitHub、Bower 和 NPM，使得分享、合作和发展这些库变得更加容易。存在许多用于 JavaScript 函数式编程的库，从微小的工具包到庞大的模块库不等。

每个库都推广其自己的函数式编程风格。从严格的、基于数学的风格到轻松的、非正式的风格，每个库都不同，但它们都有一个共同的特点：它们都具有抽象的 JavaScript 函数功能，以增加代码重用、可读性和健壮性。

然而，在撰写本文时，尚未有一个库确立为事实上的标准。有人可能会认为`underscore.js`是其中一个，但正如你将在下一节中看到的，最好避免使用`underscore.js`。

## Underscore.js

在许多人眼中，Underscore 已成为标准的函数式 JavaScript 库。它成熟、稳定，并由`Jeremy Ashkenas`创建，他是`Backbone.js`和`CoffeeScript`库背后的人物。Underscore 实际上是 Ruby 的`Enumerable`模块的重新实现，这也解释了为什么 CoffeeScript 也受到 Ruby 的影响。

与 jQuery 类似，Underscore 不修改原生 JavaScript 对象，而是使用一个符号来定义自己的对象：下划线字符"`_`"。因此，使用 Underscore 的方式如下：

```js
var x = _.map([1,2,3], Math.sqrt); // Underscore's map function
console.log(x.toString());
```

我们已经看到了 JavaScript 原生的`Array`对象的`map()`方法，它的工作方式如下：

```js
var x = [1,2,3].map(Math.sqrt);
```

不同之处在于，在 Underscore 中，`Array`对象和`callback()`函数都作为参数传递给 Underscore 对象的`map()`方法(`_.map`)，而不是仅将回调传递给数组的原生`map()`方法(`Array.prototype.map`)。

但 Underscore 不仅仅是`map()`和其他内置函数。它充满了非常方便的函数，比如`find()`、`invoke()`、`pluck()`、`sortyBy()`、`groupBy()`等等。

```js
var greetings = [{origin: 'spanish', value: 'hola'}, 
{origin: 'english', value: 'hello'}];
console.log(_.pluck(greetings, 'value')  );
// Grabs an object's property.
// Returns: ['hola', 'hello']
console.log(_.find(greetings, function(s) {return s.origin == 
'spanish';}));
// Looks for the first obj that passes the truth test
// Returns: {origin: 'spanish', value: 'hola'}
greetings = greetings.concat(_.object(['origin','value'],
['french','bonjour']));
console.log(greetings);
// _.object creates an object literal from two merged arrays
// Returns: [{origin: 'spanish', value: 'hola'},
//{origin: 'english', value: 'hello'},
//{origin: 'french', value: 'bonjour'}]
```

它提供了一种将方法链接在一起的方式：

```js
var g = _.chain(greetings)
  .sortBy(function(x) {return x.value.length})
  .pluck('origin')
  .map(function(x){return x.charAt(0).toUpperCase()+x.slice(1)})
  .reduce(function(x, y){return x + ' ' + y}, '')
  .value();
// Applies the functions 
// Returns: 'Spanish English French'
console.log(g);
```

### 注意

`_.chain()`方法返回一个包装对象，其中包含所有 Underscore 函数。然后使用`_.value`方法来提取包装对象的值。包装对象对于将 Underscore 与面向对象编程混合使用非常有用。

尽管它易于使用并被社区所接受，但`underscore.js`库因迫使你编写过于冗长的代码和鼓励错误的模式而受到批评。Underscore 的结构可能不是理想的，甚至不起作用！

直到版本 1.7.0 发布后不久，Brian Lonsdorf 的演讲*嘿，Underscore，你做错了！*在 YouTube 上发布，Underscore 明确阻止我们扩展`map()`、`reduce()`、`filter()`等函数。

```js
_.prototype.map = function(obj, iterate, [context]) {
  if (Array.prototype.map && obj.map === Array.prototype.map) return obj.map(iterate, context);
  // ...
};
```

### 注意

您可以在[www.youtube.com/watch?v=m3svKOdZij](http://www.youtube.com/watch?v=m3svKOdZij)观看 Brian Lonsdorf 的演讲视频。

在范畴论的术语中，映射是一个同态函子接口（在第五章*范畴论*中有更多介绍）。我们应该能够根据需要为`map`定义一个函子。所以 Underscore 并不是非常函数式的。

由于 JavaScript 没有内置的不可变数据，一个函数库应该小心不要让它的辅助函数改变传递给它的对象。这个问题的一个很好的例子如下所示。片段的意图是返回一个新的`selected`列表，并将一个选项设置为默认值。但实际发生的是`selected`列表被就地改变。

```js
function getSelectedOptions(id, value) {
  options = document.querySelectorAll('#' + id + ' option');
  var newOptions = _.map(options, function(opt){
    if (opt.text == value) {
      opt.selected = true;
      opt.text += ' (this is the default)';
    }
    else {
      opt.selected = false;
    }
    return opt;
  });
  return newOptions;
}
var optionsHelp = getSelectedOptions('timezones', 'Chicago');
```

我们必须在`callback()`函数中插入`opt = opt.cloneNode();`这一行，以便复制传递给函数的列表中的每个对象。Underscore 的`map()`函数作弊以提高性能，但这是以牺牲函数式的*风水*为代价。本地的`Array.prototype.map()`函数不需要这样做，因为它会复制，但它也不能用于`nodelist`集合。

Underscore 可能不太适合数学上正确的函数式编程，但它从来没有打算将 JavaScript 扩展或转换为纯函数式语言。它定义自己为*一个提供大量有用的函数式编程辅助函数的 JavaScript 库*。它可能不仅仅是一堆类似函数式的辅助函数，但它也不是一个严肃的函数库。

是否有更好的库？也许是基于数学的库？

## Fantasy Land

有时，事实比小说更离奇。

**Fantasy Land**是一个功能基础库的集合，也是 JavaScript 中如何实现“代数结构”的正式规范。更具体地说，Fantasy Land 指定了常见代数结构或简称代数的互操作性：单子、幺半群、集合、函子、链等等。它们的名字听起来可能很可怕，但它们只是一组值、一组运算符和一些必须遵守的定律。换句话说，它们只是对象。

它是如何工作的。每个代数都是一个单独的 Fantasy Land 规范，并且可能依赖于需要实现的其他代数。

![Fantasy Land](img/00002.jpeg)

一些代数规范是：

+   集合：

+   实现反射、对称和传递定律

+   定义`equals()`方法

+   半群

+   实现结合定律

+   定义`concat()`方法

+   幺半群

+   实现右恒等和左恒等

+   定义`empty()`方法

+   函子

+   实现恒等和组合定律

+   定义`map()`方法

清单不断延续。

我们不一定需要确切知道每个代数的用途，但这肯定有所帮助，特别是如果你正在编写符合规范的自己的库。这不仅仅是抽象的胡言乱语，它概述了一种实现称为范畴论的高级抽象的方法。范畴论的完整解释可以在第五章*范畴论*中找到。

Fantasy Land 不仅告诉我们如何实现函数式编程，还为 JavaScript 提供了一组函数模块。然而，许多模块是不完整的，文档也相当稀少。但 Fantasy Land 并不是唯一一个实现其开源规范的库。其他库也有，比如：**Bilby.js**。

## Bilby.js

到底什么是 bilby？不，它不是一个可能存在于 Fantasy Land 中的神话生物。它存在于地球上，是一种奇怪/可爱的鼠和兔的混合物。尽管如此，`bibly.js`库符合 Fantasy Land 的规范。

事实上，`bilby.js`是一个严肃的函数库。正如其文档所述，它是*严肃的，意味着它应用范畴论来实现高度抽象的代码。功能性的，意味着它实现了引用透明的程序*。哇，这真的很严肃。文档位于[`bilby.brianmckenna.org/`](http://bilby.brianmckenna.org/)，并且提供了以下内容：

+   用于特定多态性的不可变多方法

+   函数式数据结构

+   用于函数式语法的运算符重载

+   自动化规范测试（**ScalaCheck**，**QuickCheck**）

迄今为止，符合 Fantasy Land 规范的最成熟的库是`Bilby.js`，它是致力于函数式风格的重要资源。

让我们试一个例子：

```js
// environments in bilby are immutable structure for multimethods
var shapes1 = bilby.environment()
  // can define methods
  .method(
    'area', // methods take a name
    function(a){return typeof(a) == 'rect'}, // a predicate
    function(a){return a.x * a.y} // and an implementation
  )
  // and properties, like methods with predicates that always
  // return true
  .property(
     'name',   // takes a name
     'shape'); // and a function
// now we can overload it
var shapes2 = shapes1
  .method(
    'area', function(a){return typeof(a) == 'circle'},
    function(a){return a.r * a.r * Math.PI} );
var shapes3 = shapes2
  .method(
    'area', function(a){return typeof(a) == 'triangle'},
    function(a){return a.height * a.base / 2} );

// and now we can do something like this
var objs = [{type:'circle', r:5}, {type:'rect', x:2, y:3}];
var areas = objs.map(shapes3.area);

// and this
var totalArea = objs.map(shapes3.area).reduce(add);
```

这是范畴论和特定多态性的实践。再次强调，范畴论将在第五章中全面介绍，*范畴论*。

### 注意

范畴论是函数式程序员最近振奋的数学分支，用于最大程度地抽象和提高代码的实用性。*但有一个主要缺点：很难理解并快速上手。*

事实上，Bilby 和 Fantasy Land 确实在 JavaScript 中推动了函数式编程的可能性。尽管看到计算机科学的发展是令人兴奋的，但世界可能还没有准备好接受 Bibly 和 Fantasy Land 所推动的那种硬核函数式风格。

也许这样一个位于函数式 JavaScript 前沿的宏伟库并不适合我们。毕竟，我们的目标是探索与 JavaScript 相辅相成的函数式技术，而不是建立函数式编程教条。让我们把注意力转向另一个新库，`Lazy.js`。

## Lazy.js

Lazy 是一个实用库，更接近于`underscore.js`库，但采用了惰性求值策略。因此，Lazy 通过函数式计算结果的方式实现了不会立即得到解释的系列。它还拥有显著的性能提升。

`Lazy.js`库仍然非常年轻。但它有很大的动力和社区热情支持。

Lazy 中的一切都是我们可以迭代的序列。由于库控制方法应用的顺序，可以实现许多非常酷的事情：异步迭代（并行编程），无限序列，函数式反应式编程等。

以下示例展示了一些内容：

```js
// Get the first eight lines of a song's lyrics
var lyrics = "Lorem ipsum dolor sit amet, consectetur adipiscing eli
// Without Lazy, the entire string is first split into lines
console.log(lyrics.split('\n').slice(0,3)); 

// With Lazy, the text is only split into the first 8 lines
// The lyrics can even be infinitely long!
console.log(Lazy(lyrics).split('\n').take(3));

//First 10 squares that are evenly divisible by 3
var oneTo1000 = Lazy.range(1, 1000).toArray(); 
var sequence = Lazy(oneTo1000)
  .map(function(x) { return x * x; })
  .filter(function(x) { return x % 3 === 0; })
  .take(10)
  .each(function(x) { console.log(x); });

// asynchronous iteration over an infinite sequence
var asyncSequence = Lazy.generate(function(x){return x++})
  .async(100) // 0.100s intervals between elements
  .take(20) // only compute the first 20  
  .each(function(e) { // begin iterating over the sequence
    console.log(new Date().getMilliseconds() + ": " + e);
  });
```

更多示例和用例在第四章中有详细介绍，*在 JavaScript 中实现函数式编程技术*。

但完全归功于`Lazy.js`库这个想法并不完全正确。它的前身之一，`Bacon.js`库，也是以类似的方式工作。

## Bacon.js

`Bacon.js`库的标志如下：

![Bacon.js](img/00003.jpeg)

函数式编程库的有胡子的嬉皮士，`Bacon.js`本身是一个*函数式响应式编程*库。函数式响应式编程意味着使用函数式设计模式来表示具有反应性和不断变化的值，比如屏幕上鼠标的位置或公司股票的价格。就像 Lazy 可以通过在需要时不计算值来创建无限序列一样，Bacon 可以避免在最后一刻之前计算不断变化的值。

在 Lazy 中称为序列的东西在 Bacon 中称为 EventStreams 和 Properties，因为它们更适合处理事件（`onmouseover`，`onkeydown`等）和响应属性（滚动位置，鼠标位置，切换等）。

```js
Bacon.fromEventTarget(document.body, "click")
  .onValue(function() { alert("Bacon!") });
```

Bacon 比 Lazy 要老一点，但它的功能集大约是一半大小，社区的热情也差不多。

## 荣誉提及

在这本书的范围内，有太多的库，无法对它们进行公正的评价。让我们再看看 JavaScript 中的一些函数式编程库。

+   `Functional`

+   可能是 JavaScript 中第一个函数式编程库，`Functional`是一个包括全面的高阶函数支持以及`string` lambda 的库。

+   `wu.js`

+   特别受欢迎的`curryable()`函数，`wu.js`库是一个非常好的函数式编程库。它是第一个（我知道的）实现惰性评估的库，为`Bacon.js`，`Lazy.js`和其他库打下了基础

+   是的，它是以臭名昭著的说唱组合*Wu Tang Clan*命名的

+   `sloth.js`

+   与`Lazy.js`库非常相似，但比它小得多

+   `stream.js`

+   `stream.js`库支持无限流，除此之外没有太多功能

+   绝对微小

+   `Lo-Dash.js`

+   顾名思义，`lo-dash.js`库受到了`underscore.js`库的启发

+   高度优化

+   `Sugar`

+   `Sugar`是 JavaScript 中函数式编程技术的支持库，类似于 Underscore，但在实现方式上有一些关键的不同。

+   在 Underscore 中执行`_.pluck(myObjs, 'value')`，在 Sugar 中只需`myObjs.map('value')`。这意味着它修改了原生 JavaScript 对象，因此有一定风险，可能无法与其他执行相同操作的库（如 Prototype）很好地配合。

+   非常好的文档，单元测试，分析器等。

+   `from.js`

+   一个新的函数库和 JavaScript 的**LINQ**（**语言集成查询**）引擎，支持大部分.NET 提供的相同 LINQ 函数

+   100%的惰性评估和支持 lambda 表达式

+   非常年轻，但文档非常好

+   JSLINQ

+   JavaScript 的另一个函数式 LINQ 引擎

+   比`from.js`库更老，更成熟

+   `Boiler.js`

+   另一个实用库，将 JavaScript 的函数方法扩展到更多的原语：字符串，数字，对象，集合和数组

+   **Folktale**

+   像`Bilby.js`库一样，Folktale 是另一个实现 Fantasy Land 规范的新库。和它的前辈一样，Folktale 也是一个用于 JavaScript 中的函数式编程的库集合。它还很年轻，但可能会有一个光明的未来。

+   **jQuery**

+   看到 jQuery 被提到感到惊讶？尽管 jQuery 不是用于执行函数式编程的工具，但它本身也是函数式的。jQuery 可能是最广泛使用的库之一，它的根源是函数式编程。

+   jQuery 对象实际上是一个单子。jQuery 使用单子定律来实现方法链式调用：

```js
$('#mydiv').fadeIn().css('left': 50).alert('hi!');
```

关于这一点的详细解释可以在第七章中找到，*JavaScript 中的函数式和面向对象编程*。

+   它的一些方法是高阶的：

```js
$('li').css('left': function(index){return index*50});
```

+   从 jQuery 1.8 开始，`deferred.then`参数实现了一种称为 Promises 的函数概念。

+   jQuery 是一个抽象层，主要用于 DOM。它不是一个框架或工具包，只是一种利用抽象来增加代码重用和减少丑陋代码的方法。这难道不正是函数式编程的全部意义吗？

# 开发和生产环境

从编程风格的角度来看，应用程序是在哪种环境中开发和部署的并不重要。但对于库来说却很重要。

## 浏览器

大多数 JavaScript 应用程序都设计为在客户端运行，也就是在客户端的浏览器中。基于浏览器的环境非常适合开发，因为浏览器无处不在，你可以在本地机器上直接编写代码，解释器是浏览器的 JavaScript 引擎，所有浏览器都有开发者控制台。Firefox 的 FireBug 提供非常有用的错误消息，并允许设置断点等，但在 Chrome 和 Safari 中运行相同的代码以交叉参考错误输出通常也很有帮助。即使是 Internet Explorer 也包含开发者工具。

浏览器的问题在于它们以不同的方式评估 JavaScript！虽然不常见，但有可能编写的代码在不同的浏览器中返回非常不同的结果。但通常差异在于它们处理文档对象模型的方式，而不是原型和函数的工作方式。显然，`Math.sqrt(4)`方法对所有浏览器和 shell 都返回`2`。但`scrollLeft`方法取决于浏览器的布局策略。

编写特定于浏览器的代码是浪费时间，这也是为什么应该使用库的另一个原因。

## 服务器端 JavaScript

`Node.js`库已成为创建服务器端和基于网络的应用程序的标准平台。函数式编程可以用于服务器端应用程序编程吗？可以！好吧，但是否存在专为这种性能关键环境设计的函数式库？答案也是：是的。

本章中概述的所有函数式库都可以在`Node.js`库中工作，并且许多依赖于`browserify.js`模块来处理浏览器元素。

### 服务器端环境中的函数式用例

在我们这个充满网络系统的新世界中，服务器端应用程序开发人员经常关注并且理所当然地关注并发性。经典的例子是允许多个用户修改同一个文件的应用程序。但如果他们同时尝试修改它，就会陷入一团糟。这是困扰程序员几十年的*状态维护*问题。

假设以下情景：

1.  一天早晨，亚当打开一个报告进行编辑，但在离开吃午饭前没有保存。

1.  比利打开同样的报告，添加了他的笔记，然后保存了。

1.  亚当从午饭回来，添加了他的笔记到报告中，然后保存了，无意中覆盖了比利的笔记。

1.  第二天，比利发现他的笔记不见了。他的老板对他大喊大叫；每个人都生气了，他们联合起来对那个误入歧途的应用程序开发人员进行了不公正的解雇。

很长一段时间，解决这个问题的方法是创建一个关于文件的状态。当有人开始编辑时，切换锁定状态为*on*，这样其他人就无法编辑它，然后在保存后切换为*off*。在我们的情景中，比利在亚当回来吃午饭之前无法完成工作。如果从未保存（比如说，亚当决定在午饭休息时辞职），那么就永远无法编辑它。

这就是函数式编程关于不可变数据和状态（或缺乏状态）的想法真正可以发挥作用的地方。与其让用户直接修改文件，采用函数式方法，他们会修改文件的副本，也就是一个新的版本。如果他们试图保存该版本，而新版本已经存在，那么我们就知道其他人已经修改了旧版本。危机得以避免。

现在之前的情景会是这样展开的：

1.  一天早晨，亚当打开一个报告进行编辑。但他在午餐前没有保存它。

1.  比利打开相同的报告，添加他的笔记，并将其保存为新的修订版本。

1.  亚当从午餐回来添加他的笔记。当他试图保存新的修订版本时，应用程序告诉他现在存在一个更新的修订版本。

1.  亚当打开新的修订版本，添加了他的笔记，并保存了另一个新的修订版本。

1.  通过查看修订历史，老板发现一切都运行顺利。每个人都很高兴，应用程序开发人员得到了晋升和加薪。

这被称为*事件溯源*。没有明确的状态需要维护，只有事件。这个过程更加清洁，有一个可以审查的明确事件历史。

这个想法和许多其他想法是为什么服务器端环境中的功能性编程正在兴起。

## CLI

尽管 Web 和`node.js`库是两个主要的 JavaScript 环境，一些务实和冒险的用户正在寻找方法在命令行中使用 JavaScript。

将 JavaScript 用作命令行界面（CLI）脚本语言可能是应用函数编程的最佳机会之一。想象一下，当搜索本地文件或将整个 bash 脚本重写为功能性 JavaScript 一行时，能够使用惰性评估。

## 与其他 JavaScript 模块一起使用功能库

Web 应用程序由各种组件组成：框架、库、API 等。它们可以作为依赖项、插件或并存对象一起工作。

+   `Backbone.js`

+   一个具有 RESTful JSON 接口的 MVP（模型-视图-提供者）框架

+   需要`underscore.js`库，Backbone 的唯一硬依赖

+   jQuery

+   `Bacon.js`库具有与 jQuery 混合的绑定

+   Underscore 和 jQuery 非常好地互补了彼此

+   原型 JavaScript 框架

+   提供 JavaScript 与 Ruby 的 Enumerable 最接近的集合函数

+   `Sugar.js`

+   修改本地对象及其方法

+   在与其他库混合使用时必须小心，特别是 Prototype

## 编译为 JavaScript 的功能语言

有时，JavaScript 的内部功能上的 C 样式厚重外观足以让你想切换到另一种功能性语言。好吧，你可以！

+   Clojure 和 ClojureScript

+   闭包是现代 Lisp 实现和功能齐全的功能语言

+   ClojureScript 将 Clojure 转译为 JavaScript

+   CoffeeScript

+   CoffeeScript 既是一种功能性语言的名称，也是一种将该语言转译为 JavaScript 的编译器。

+   CoffeeScript 中的表达式与 JavaScript 中的表达式之间存在一对一的映射

还有许多其他选择，包括 Pyjs，Roy，TypeScript，UHC 等。

# 总结

你选择使用哪个库取决于你的需求。需要功能性反应式编程来处理事件和动态值吗？使用`Bacon.js`库。只需要无限流而不需要其他东西吗？使用`stream.js`库。想要用功能性助手补充 jQuery 吗？试试`underscore.js`库。需要一个结构化环境来进行严肃的特定多态性吗？看看`bilby.js`库。需要一个全面的功能性编程工具吗？使用`Lazy.js`库。对这些选项都不满意吗？自己写一个！

任何库的好坏取决于它的使用方式。尽管本章概述的一些库存在一些缺陷，但大多数故障发生在键盘和椅子之间的某个地方。你需要正确使用库来满足你的需求。

如果我们将代码库导入 JavaScript 环境，也许我们也可以导入想法和原则。也许我们可以借鉴*Python 之禅*，由*Tim Peter*：

> *美丽胜于丑陋*
> 
> *显式胜于隐式*
> 
> *简单胜于复杂*
> 
> *复杂胜于复杂*
> 
> *平面胜于嵌套*
> 
> *稀疏胜于密集*
> 
> *可读性很重要。*
> 
> *特殊情况并不特别到足以打破规则。*
> 
> *尽管实用性胜过纯粹。*
> 
> *错误不应该悄悄地通过。*
> 
> *除非明确要求保持沉默。*
> 
> *面对模棱两可，拒绝猜测的诱惑。*
> 
> *应该有一种——最好只有一种——明显的方法来做到这一点。*
> 
> *尽管这种方式一开始可能不明显，除非你是荷兰人。*
> 
> *现在总比永远好。*
> 
> *尽管永远往往比“现在”更好。*
> 
> *如果实现很难解释，那是个坏主意。*
> 
> *如果实现很容易解释，那可能是个好主意。*
> 
> *命名空间是一个非常好的主意——让我们做更多这样的事情！*
