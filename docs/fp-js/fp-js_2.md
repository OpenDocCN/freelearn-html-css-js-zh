# 第二章：函数式编程基础

到目前为止，你已经看到了函数式编程可以做些什么的一小部分。但函数式编程到底是什么？什么使一种语言是函数式的而另一种不是？什么使一种编程风格是函数式的而另一种不是？

在本章中，我们将首先回答这些问题，然后介绍函数式编程的核心概念：

+   使用函数和数组进行控制流

+   编写纯函数、匿名函数、递归函数等

+   像对象一样传递函数

+   利用 `map()`、`filter()` 和 `reduce()` 函数

# 函数式编程语言

函数式编程语言是促进函数式编程范式的语言。冒昧地说，我们可以说，如果一种语言包括函数式编程所需的特性，那么它就是一种函数式语言——就是这么简单。在大多数情况下，真正决定一个程序是否是函数式的是编程风格。

## 什么使一种语言是函数式的？

C 语言无法进行函数式编程。Java 语言也无法进行函数式编程（没有大量繁琐的“几乎”函数式编程的变通方法）。这些以及许多其他语言根本就不包含支持函数式编程的结构。它们纯粹是面向对象的，严格来说不是函数式语言。

同时，面向对象编程无法在纯函数式语言中进行，比如 Scheme、Haskell 和 Lisp，仅举几例。

然而，有些语言支持两种模型。Python 就是一个著名的例子，但还有其他的：Ruby、Julia，还有我们感兴趣的 JavaScript。这些语言如何支持两种非常不同的设计模式？它们包含了两种编程范式所需的特性。然而，在 JavaScript 的情况下，函数式特性有些隐藏。

但实际上，情况要复杂一些。那么什么使一种语言是函数式的呢？

| 特征 | 命令式 | 函数式 |
| --- | --- | --- |
| 编程风格 | 执行逐步任务和管理状态变化 | 定义问题是什么以及需要哪些数据转换来实现解决方案 |
| 状态变化 | 重要 | 不存在 |
| 执行顺序 | 重要 | 不重要 |
| 主要流程控制 | 循环、条件和函数调用 | 函数调用和递归 |
| 主要操作单元 | 结构和类对象 | 函数作为一等对象和数据集 |

语言的语法必须允许某些设计模式，比如隐式类型系统和使用匿名函数的能力。基本上，语言必须实现 Lambda 演算。此外，解释器的评估策略应该是非严格的和按需调用（也称为延迟执行），这允许不可变的数据结构和非严格的惰性评估。

## 优势

你可以说，当你最终“领悟”时所经历的深刻启示将使学习函数式编程变得值得。这样的经历将使你成为终身更好的程序员，无论你是否真的成为全职的函数式程序员。

但我们不是在谈论学习冥想；我们在谈论学习一种极其有用的工具，这将使你成为一个更好的程序员。

从形式上讲，使用函数式编程的实际优势是什么？

### 更清晰的代码

函数式程序更清洁、更简单、更小。这简化了调试、测试和维护。

例如，假设我们需要一个将二维数组转换为一维数组的函数。只使用命令式技术，我们可以这样写：

```js
function merge2dArrayIntoOne(arrays) {
  var count = arrays.length;
  var merged = new Array(count); 
  var c = 0;
  for (var i = 0; i < count; ++i) {
    for (var j = 0, jlen = arrays[i].length; j < jlen; ++j) {
      merged[c++] = arrays[i][j];
    }
  }
  return merged
}
```

并且使用函数式技术，可以写成如下形式：

```js
varmerge2dArrayIntoOne2 = function(arrays) {
  return arrays.reduce( function(p,n){
    return p.concat(n);
  });
};
```

这两个函数都接受相同的输入并返回相同的输出。然而，函数示例要简洁清晰得多。

### 模块化

函数式编程迫使将大问题分解为要解决的相同问题的较小实例。这意味着代码更加模块化。模块化的程序具有明确定义，更容易调试，更简单维护。测试更容易，因为每个模块化代码片段都可以潜在地检查正确性。

### 可重用性

由于函数式编程的模块化，函数式程序共享各种常见的辅助函数。您会发现这些函数中的许多函数可以被重用于各种不同的应用程序。

本章后面将介绍许多常见的函数。然而，作为函数式程序员，您将不可避免地编写自己的小函数库，这些函数可以反复使用。例如，一个设计良好的函数，用于搜索配置文件的行，也可以用于搜索哈希表。

### 减少耦合

耦合是程序中模块之间的依赖程度。因为函数式程序员致力于编写独立于彼此的一流、高阶、纯函数，不对全局变量产生副作用，所以耦合大大减少。当然，函数将不可避免地相互依赖。但只要输入到输出的一对一映射保持正确，修改一个函数不会改变另一个函数。

### 数学上的正确性

最后一个是在更理论的层面上。由于其源自 Lambda 演算，函数式程序可以在数学上被证明是正确的。这对需要证明程序的增长率、时间复杂度和数学正确性的研究人员来说是一个巨大优势。

让我们来看看斐波那契数列。尽管它很少用于除了概念验证之外的任何其他用途，但它很好地说明了这个概念。评估斐波那契数列的标准方法是创建一个递归函数，表达式为`fibonnaci(n) = fibonnaci(n-2) + fibonnaci(n–1)`，并且有一个基本情况，即`当 n < 2 时返回 1`，这样可以停止递归并开始在递归调用堆栈的每一步返回的值上进行求和。

这描述了计算序列所涉及的中间步骤。

```js
var fibonacci = function(n) {
  if (n < 2) {
    return 1;
  }
  else {
    return fibonacci(n - 2) + fibonacci(n - 1);
  }
}
console.log( fibonacci(8) );
// Output: 34
```

然而，借助实现惰性执行策略的库，可以生成一个无限序列，该序列陈述了定义整个数字序列的*数学方程*。只有需要计算的数字才会被计算。

```js
var fibonacci2 = Lazy.generate(function() {
  var x = 1,
  y = 1;
  return function() {
    var prev = x;
    x = y;
    y += prev;
    return prev;
  };
}());

console.log(fibonacci2.length());// Output: undefined

console.log(fibonacci2.take(12).toArray());// Output: [1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144] 

var fibonacci3 = Lazy.generate(function() {
  var x = 1,
  y = 1;
  return function() {
    var prev = x;
    x = y;
    y += prev;
    return prev;
  };
}());

console.log(fibonacci3.take(9).reverse().first(1).toArray());// Output: [34]
```

第二个例子显然更具数学上的合理性。它依赖于 JavaScript 的`Lazy.js`库。还有其他可以帮助的库，比如`Sloth.js`和`wu.js`。这些将在第三章中进行介绍，*设置函数式编程环境*。

## 非函数式世界中的函数式编程

函数式和非函数式编程可以混合在一起吗？尽管这是第七章的主题，*JavaScript 中的函数式和面向对象编程*，但在我们继续之前，有几件事情需要搞清楚。

本书的目的不是教您如何实现严格遵循纯函数式编程严格要求的整个应用程序。这样的应用程序在学术界之外很少合适。相反，本书将教您如何在应用程序中使用函数式编程设计策略，以补充必要的命令式代码。

例如，如果您需要从某个文本中提取出的前四个只包含字母的单词，可以天真地写成这样：

```js
var words = [], count = 0;
text = myString.split(' ');
for (i=0; count<4, i<text.length; i++) {
  if (!text[i].match(/[0-9]/)) {
    words = words.concat(text[i]);
    count++;
  }
}
console.log(words);
```

相比之下，函数式程序员可能会这样写：

```js
var words = [];
var words = myString.split(' ').filter(function(x){
  return (! x.match(/[1-9]+/));
}).slice(0,4);
console.log(words);
```

或者，通过一个函数式编程工具库，它们甚至可以更简化：

```js
var words = toSequence(myString).match(/[a-zA-Z]+/).first(4);
```

识别可以以更函数式方式编写的函数的关键是查找循环和临时变量，例如前面示例中的`words`和`count`实例。通常我们可以通过用高阶函数替换它们来摆脱临时变量和循环，我们将在本章后面探讨这一点。

### JavaScript 是一种函数式编程语言吗？

我们必须问自己最后一个问题。JavaScript 是一种函数式语言还是非函数式语言？

JavaScript 可以说是世界上最流行但最不被理解的函数式编程语言。JavaScript 是一种穿着 C 样式衣服的函数式编程语言。它的语法无可否认地类似于 C，意味着它使用 C 的块语法和中缀顺序。而且它是存在的最糟糕的命名语言之一。毫不费力地可以想象为什么这么多人会混淆 JavaScript 与 Java 有关；不知何故，它的名字暗示着它应该有关联！但实际上它与 Java 几乎没有共同之处。而且，为了真正巩固 JavaScript 是一种面向对象的语言的想法，诸如 Dojo 和**ease.js**等库和框架一直在努力将其抽象化，并使其适用于面向对象的编程。JavaScript 在 20 世纪 90 年代成熟起来，当时面向对象编程是所有人都在谈论的话题，我们被告知 JavaScript 是面向对象的，因为我们非常希望它是这样。但它并不是。

它的真正身份更与其祖先相一致：Scheme 和 Lisp，两种经典的函数式语言。JavaScript 是一种纯函数式语言。它的函数是一等公民，可以嵌套，具有闭包和组合，并且允许柯里化和单子。所有这些都是函数式编程的关键。以下是 JavaScript 是函数式语言的几个原因：

+   JavaScript 的词法语法包括将函数作为参数传递的能力，具有推断类型系统，并允许匿名函数、高阶函数、闭包等。这些事实对于实现函数式编程的结构和行为至关重要。

+   它不是一种纯粹的面向对象语言，大多数面向对象的设计模式是通过复制原型对象来实现的，这是一种较弱的面向对象编程模型。**欧洲计算机制造商协会脚本**（**ECMAScript**），JavaScript 的正式和标准化实现规范，在规范 4.2.1 中陈述了以下内容：

> *“ECMAScript 不包含像 C++、Smalltalk 或 Java 中那样的适当类，而是支持创建对象的构造函数。在基于类的面向对象语言中，一般来说，状态由实例承载，方法由类承载，继承仅涉及结构和行为。在 ECMAScript 中，状态和方法由对象承载，结构、行为和状态都是继承的。”*

+   它是一种解释性语言。有时被称为“引擎”，JavaScript 解释器通常与 Scheme 解释器非常相似。两者都是动态的，都具有灵活的数据类型，可以轻松组合和转换，都将代码评估为表达式块，并且都类似地处理函数。

也就是说，JavaScript 并不是一种纯函数式语言。缺少的是惰性求值和内置的不可变数据。这是因为大多数解释器是按名称调用而不是按需调用。由于它处理尾调用的方式，JavaScript 也不太擅长递归。然而，所有这些问题都可以通过一点注意来缓解。非严格求值，用于无限序列和惰性求值，可以通过一个名为`Lazy.js`的库来实现。不可变数据可以通过编程技术简单实现，但这需要更多的程序员纪律，而不是依赖语言来处理。递归尾调用消除可以通过一种称为**Trampolining**的方法来实现。这些问题将在第六章中得到解决，*JavaScript 中的高级主题和陷阱*。

关于 JavaScript 是一种函数式语言、面向对象语言、两者都是还是两者都不是，已经进行了许多争论。而且这不会是最后一次辩论。

最终，函数式编程是通过巧妙地改变、组合和使用函数的方式来编写更清晰的代码。JavaScript 为这种方法提供了一个出色的媒介。如果您真的想要充分发挥 JavaScript 的潜力，您必须学会将其用作一种函数式语言。

# 使用函数

|   | *有时，优雅的实现是一个函数。不是一个方法。不是一个类。不是一个框架。只是一个函数。* |   |
| --- | --- | --- |
|   | --*约翰·卡马克，末日游戏的首席程序员* |

函数式编程是将问题分解为一组函数的过程。通常，函数被链接在一起，嵌套在彼此内部，传递并被视为一等公民。如果您使用过 jQuery 和 Node.js 等框架，您可能已经使用了其中一些技术，只是您没有意识到！

让我们从一个小 JavaScript 困境开始。

我们需要编译一个分配给通用对象的值列表。对象可以是任何东西：日期、HTML 对象等等。

```js
var
  obj1 = {value: 1},
  obj2 = {value: 2},
  obj3 = {value: 3};

var values = [];
function accumulate(obj) {
  values.push(obj.value);
}
accumulate(obj1);
accumulate(obj2);
console.log(values); // Output: [obj1.value, obj2.value]
```

它能工作，但它是不稳定的。任何代码都可以修改`values`对象，而不调用`accumulate()`函数。如果我们忘记将空集`[]`分配给`values`实例，那么代码将根本无法工作。

但是，如果变量在函数内声明，它就不能被任何不受控制的代码改变。

```js
function accumulate2(obj) {
  var values = [];
  values.push(obj.value);
  return values;
}
console.log(accumulate2(obj1)); // Returns: [obj1.value]
console.log(accumulate2(obj2)); // Returns: [obj2.value]
console.log(accumulate2(obj3)); // Returns: [obj3.value]
```

它不起作用！只返回最后传入的对象的值。

我们可能可以通过在第一个函数内部嵌套一个函数来解决这个问题。

```js
var ValueAccumulator = function(obj) {
  var values = []
  var accumulate = function() {
    values.push(obj.value);   
  };
  accumulate();
  return values;
};
```

但问题是一样的，现在我们无法访问`accumulate`函数或`values`变量。

我们需要的是一个自调用函数。

## 自调用函数和闭包

如果我们能返回一个函数表达式，该表达式反过来返回`values`数组呢？在函数中声明的变量对函数内的任何代码都是可用的，包括自调用函数。

通过使用自调用函数，我们解决了我们的困境。

```js
var ValueAccumulator = function() {
  var values = [];
  var accumulate = function(obj) {
    if (obj) {
      values.push(obj.value);
      return values;
    }
    else {
      return values;
    }
  };
  return accumulate;
};

//This allows us to do this:
var accumulator = ValueAccumulator();
accumulator(obj1); 
accumulator(obj2); 
console.log(accumulator()); 
// Output: [obj1.value, obj2.value]
```

这都与变量作用域有关。`values`变量对内部的`accumulate()`函数是可用的，即使在作用域外部调用函数时也是如此。这就是闭包。

### 注意

JavaScript 中的闭包是指即使父函数已关闭，也可以访问父作用域的函数。

闭包是所有函数式语言的特性。传统的命令式语言不允许它们。

## 高阶函数

自调用函数实际上是一种高阶函数形式。高阶函数是要么将另一个函数作为输入，要么将一个函数作为输出的函数。

高阶函数在传统编程中并不常见。在命令式编程中，程序员可能会使用循环来迭代数组，而函数式编程者则会采取完全不同的方法。通过使用高阶函数，可以通过将该函数应用于数组中的每个项来对数组进行操作，从而创建一个新数组。

这是函数式编程范式的核心思想。高阶函数允许将逻辑传递给其他函数，就像对象一样。

在 JavaScript 中，函数被视为一等公民，这是 JavaScript 与 Scheme、Haskell 和其他经典函数式语言共享的特点。这听起来很奇怪，但这实际上意味着函数被视为原语，就像数字和对象一样。如果数字和对象可以被传递，那么函数也可以被传递。

为了看到这一点，让我们在前面部分的`ValueAccumulator()`函数中使用一个高阶函数：

```js
// using forEach() to iterate through an array and call a 
// callback function, accumulator, for each item
var accumulator2 = ValueAccumulator();
var objects = [obj1, obj2, obj3]; // could be huge array of objects
objects.forEach(accumulator2);
console.log(accumulator2());
```

## 纯函数

纯函数返回仅使用传递给它的输入计算的值。不能使用外部变量和全局状态，也不能产生副作用。换句话说，它不能改变传递给它的输入变量。因此，纯函数只能用于它们的返回值。

一个简单的例子是数学函数。`Math.sqrt(4)`函数将始终返回`2`，不使用任何隐藏信息，如设置或状态，并且永远不会产生任何副作用。

纯函数是对数学术语“函数”的真正解释，它是输入和输出之间的关系。它们很容易理解，并且可以被很容易地重复使用。因为它们是完全独立的，所以纯函数更有可能被反复使用。

为了说明这一点，比较以下非纯函数和纯函数。

```js
// function that prints a message to the center of the screen
var printCenter = function(str) {
  var elem = document.createElement("div");
  elem.textContent = str;
  elem.style.position = 'absolute';
  elem.style.top = window.innerHeight/2+"px";
  elem.style.left = window.innerWidth/2+"px";
  document.body.appendChild(elem);
};
printCenter('hello world');
// pure function that accomplishes the same thing
var printSomewhere = function(str, height, width) {
  var elem = document.createElement("div");
  elem.textContent = str;
  elem.style.position = 'absolute';
  elem.style.top = height;
  elem.style.left = width;
  return elem;
};
document.body.appendChild(printSomewhere('hello world', window.innerHeight/2)+10+"px",window.innerWidth/2)+10+"px")
);
```

非纯函数依赖于窗口对象的状态来计算高度和宽度，而纯自给自足的函数则要求传入这些值。实际上，这样做可以让消息在任何地方打印，这使得函数更加灵活。

虽然非纯函数可能看起来更容易，因为它自己执行附加而不是返回一个元素，但纯函数`printSomewhere()`及其返回值与其他函数式编程设计技术更配合。

```js
var messages = ['Hi', 'Hello', 'Sup', 'Hey', 'Hola'];
messages.map(function(s,i){
  return printSomewhere(s, 100*i*10, 100*i*10);
}).forEach(function(element) {
  document.body.appendChild(element);
});
```

### 注意

当函数是纯的并且不依赖于状态或环境时，我们不关心它们实际上何时何地被计算。稍后我们将在惰性求值中看到这一点。

## 匿名函数

将函数视为一等公民的另一个好处是匿名函数的出现。

顾名思义，匿名函数是没有名称的函数。但它们不仅仅是这样。它们允许根据需要定义临时逻辑。通常是为了方便起见；如果函数只被引用一次，那么就不需要浪费一个变量名。

一些匿名函数的例子如下：

```js
// The standard way to write anonymous functions
function(){return "hello world"};

// Anonymous function assigned to variable
var anon = function(x,y){return x+y};

// Anonymous function used in place of a named callback function, 
// this is one of the more common uses of anonymous functions.
setInterval(function(){console.log(new Date().getTime())}, 1000);
// Output:  1413249010672, 1413249010673, 1413249010674, ...

// Without wrapping it in an anonymous function, it immediately // execute once and then return undefined as the callback:
setInterval(console.log(new Date().getTime()), 1000)
// Output:  1413249010671
```

在高阶函数中使用匿名函数的更复杂的例子：

```js
function powersOf(x) {
  return function(y) {
    // this is an anonymous function!
    return Math.pow(x,y);
  };
}
powerOfTwo = powersOf(2);
console.log(powerOfTwo(1)); // 2
console.log(powerOfTwo(2)); // 4
console.log(powerOfTwo(3)); // 8

powerOfThree = powersOf(3);
console.log(powerOfThree(3));  // 9
console.log(powerOfThree(10)); // 59049
```

返回的函数不需要被命名；它不能在`powersOf()`函数之外的任何地方使用，因此它是一个匿名函数。

记得我们的累加器函数吗？可以使用匿名函数来重写它。

```js
var
  obj1 = {value: 1},
  obj2 = {value: 2},
  obj3 = {value: 3};

var values = (function() {
  // anonymous function
  var values = [];
  return function(obj) {
    // another anonymous function!
    if (obj) {
      values.push(obj.value);
      return values;
    }
    else {
      return values;
    }
  }
})(); // make it self-executing
console.log(values(obj1)); // Returns: [obj.value]
console.log(values(obj2)); // Returns: [obj.value, obj2.value]
```

太好了！一个纯的、高阶的、匿名的函数。我们怎么会这么幸运呢？实际上，它不仅仅是这样。它还是*自执行*的，如结构`(function(){...})();`所示。匿名函数后面的一对括号会立即调用函数。在上面的例子中，`values`实例被赋值为自执行函数调用的输出。

### 注意

匿名函数不仅仅是一种语法糖。它们是 Lambda 演算的体现。跟我一起来理解一下……Lambda 演算是在计算机或计算机语言出现之前发明的。它只是用于推理函数的数学概念。令人惊讶的是，尽管它只定义了三种表达式：变量引用、函数调用和*匿名函数*，它却是图灵完备的。今天，如果你知道如何找到它，Lambda 演算是所有函数式语言的核心，包括 JavaScript。

因此，匿名函数通常被称为 Lambda 表达式。

匿名函数的一个缺点是它们很难在调用堆栈中识别，这使得调试变得更加棘手。应该谨慎使用它们。

## 方法链

在 JavaScript 中链接方法是相当常见的。如果你使用过 jQuery，你可能已经使用过这种技术。有时被称为“构建器模式”。

这是一种用于简化代码的技术，其中多个函数依次应用于对象。

```js
// Instead of applying the functions one per line...
arr = [1,2,3,4];
arr1 = arr.reverse();
arr2 = arr1.concat([5,6]);
arr3 = arr2.map(Math.sqrt);
// ...they can be chained together into a one-liner
console.log([1,2,3,4].reverse().concat([5,6]).map(Math.sqrt));
// parentheses may be used to illustrate
console.log(((([1,2,3,4]).reverse()).concat([5,6])).map(Math.sqrt) );
```

这只有在函数是对象的方法时才有效。如果你创建了自己的函数，例如，接受两个数组并返回一个将这两个数组合并在一起的数组，你必须将它声明为`Array.prototype`对象的成员。看一下以下代码片段：

```js
Array.prototype.zip = function(arr2) {
  // ...
}
```

这将使我们能够做到以下几点：

```js
arr.zip([11,12,13,14).map(function(n){return n*2});
// Output: 2, 22, 4, 24, 6, 26, 8, 28
```

## 递归

递归很可能是最著名的函数式编程技术。如果你还不知道，递归函数是调用自身的函数。

当一个函数调用*它自己*时，会发生一些奇怪的事情。它既像一个循环，执行相同的代码多次，又像一个函数堆栈。

递归函数必须非常小心地避免无限循环（在这种情况下是无限递归）。因此，就像循环一样，必须使用条件来知道何时停止。这被称为基本情况。

一个例子如下：

```js
var foo = function(n) {
  if (n < 0) {
    // base case
    return 'hello';
  }
  else {
    // recursive case
    foo(n-1);
  }
}
console.log(foo(5));
```

可以将任何循环转换为递归算法，也可以将任何递归算法转换为循环。但是递归算法更适合，几乎是必要的，用于与适合使用循环的情况大不相同的情况。

一个很好的例子是树的遍历。虽然使用递归函数遍历树并不太难，但使用循环会更加复杂，并且需要维护一个堆栈。这与函数式编程的精神相违背。

```js
var getLeafs = function(node) {
  if (node.childNodes.length == 0) {
    // base case
    return node.innerText;
  }
  else {
    // recursive case: 
    return node.childNodes.map(getLeafs);
  }
}
```

### 分而治之

递归不仅仅是一种在没有`for`和`while`循环的情况下进行迭代的有趣方式。一种算法设计，称为分而治之，将问题递归地分解为相同问题的较小实例，直到它们足够小以便解决。

这方面的历史例子是欧几里得算法，用于找到两个数的最大公约数。

```js
function gcd(a, b) {
  if (b == 0) {
    // base case (conquer)
    return a;
  }
  else {
    // recursive case (divide)
    return gcd(b, a % b);
  }
}

console.log(gcd(12,8));
console.log(gcd(100,20));
```

所以从理论上讲，分而治之的工作方式非常优雅，但它在现实世界中有用吗？是的！JavaScript 中用于对数组进行排序的函数并不是很好。它不仅会就地对数组进行排序，这意味着数据是不可变的，而且它也不可靠和灵活。通过分而治之，我们可以做得更好。

归并排序算法使用分而治之的递归算法设计，通过递归地将数组分成较小的子数组，然后将它们合并在一起来高效地对数组进行排序。

在 JavaScript 中的完整实现大约有 40 行代码。然而，伪代码如下：

```js
var mergeSort = function(arr){
  if (arr.length < 2) {
    // base case: 0 or 1 item arrays don't need sorting
    return items;
  }
  else {
    // recursive case: divide the array, sort, then merge
    var middle = Math.floor(arr.length / 2);
    // divide
    var left = mergeSort(arr.slice(0, middle));
    var right = mergeSort(arr.slice(middle));
    // conquer
    // merge is a helper function that returns a new array
    // of the two arrays merged together
    return merge(left, right);
  }
}
```

## 惰性求值

惰性评估，也称为非严格评估、按需调用和延迟执行，是一种等到需要值时才计算函数结果的评估策略，对函数式编程非常有用。很明显，一行代码 `x = func()` 表示要将 `x` 赋值为 `func()` 返回的值。但是 `x` 实际上等于什么并不重要，直到需要它为止。等到需要 `x` 时再调用 `func()` 就是惰性评估。

这种策略可以大大提高性能，特别是在方法链和数组中使用时，这是函数式编程者最喜欢的程序流技术。

惰性评估的一个令人兴奋的好处是存在无限序列。因为直到不能再延迟才实际计算任何东西，所以这是可能的：

```js
// wishful JavaScript pseudocode:
var infinateNums = range(1 to infinity);
var tenPrimes = infinateNums.getPrimeNumbers().first(10);
```

这为许多可能性打开了大门：异步执行、并行化和组合，仅举几例。

然而，有一个问题：JavaScript 本身不执行惰性评估。也就是说，存在着一些用于 JavaScript 的库，可以非常好地模拟惰性评估。这就是《第三章》《设置函数式编程环境》的主题。

# 函数式编程者的工具包

如果你仔细看了到目前为止呈现的几个例子，你会注意到使用了一些你可能不熟悉的方法。它们是 `map()`、`filter()` 和 `reduce()` 函数，对任何语言的函数式程序都至关重要。它们使你能够消除循环和语句，从而使代码更加清晰。

`map()`、`filter()` 和 `reduce()` 函数构成了函数式编程者工具包的核心，这是一组纯的高阶函数，是函数式方法的工作马。事实上，它们是纯函数和高阶函数应该具有的典范；它们接受一个函数作为输入，并返回一个没有副作用的输出。

虽然它们是 ECMAScript 5.1 实现的浏览器的标准，但它们只适用于数组。每次调用时，都会创建并返回一个新数组。现有数组不会被修改。但还有更多，*它们接受函数作为输入*，通常以匿名函数的形式作为回调函数；它们遍历数组并将函数应用于数组中的每个项目！

```js
myArray = [1,2,3,4];
newArray = myArray.map(function(x) {return x*2});
console.log(myArray);  // Output: [1,2,3,4]
console.log(newArray); // Output: [2,4,6,8]
```

还有一点。因为它们只适用于数组，所以不能用于其他可迭代的数据结构，比如某些对象。不用担心，诸如 `underscore.js`、`Lazy.js`、`stream.js` 等库都实现了自己的 `map()`、`filter()` 和 `reduce()` 方法，更加灵活。

## 回调

如果你以前从未使用过回调函数，可能会觉得这个概念有点费解。特别是在 JavaScript 中，因为 JavaScript 允许以多种方式声明函数。

`callback()` 函数用于传递给其他函数以供其使用。这是一种传递逻辑的方式，就像传递对象一样：

```js
var myArray = [1,2,3];
function myCallback(x){return x+1};
console.log(myArray.map(myCallback));
```

为了简化简单的任务，可以使用匿名函数：

```js
console.log(myArray.map(function(x){return x+1}));
```

它们不仅用于函数式编程，还用于 JavaScript 中的许多其他事情。仅举一个例子，这是在使用 jQuery 进行 AJAX 调用时使用的 `callback()` 函数：

```js
function myCallback(xhr){
  console.log(xhr.status); 
  return true;
}
$.ajax(myURI).done(myCallback);
```

注意只使用了函数的名称。因为我们没有调用回调，只是传递了它的名称，所以写成这样是错误的：

```js
$.ajax(myURI).fail(**myCallback(xhr)**);
// or
$.ajax(myURI).fail(**myCallback()**);
```

如果我们调用回调会发生什么？在这种情况下，`myCallback(xhr)` 方法将尝试执行——控制台将打印“undefined”，并返回 `True`。当 `ajax()` 调用完成时，它将以 'true' 作为要使用的回调函数的名称，这将引发错误。

这也意味着我们无法指定传递给回调函数的参数。如果我们需要与`ajax()`调用传递给它的参数不同的参数，我们可以将回调函数包装在匿名函数中：

```js
function myCallback(status){
  console.log(status); 
  return true;
}
$.ajax(myURI).done(function(xhr){myCallback(xhr.status)});
```

## Array.prototype.map()

`map()`函数是这一系列中的头目。它只是在数组中的每个项目上应用回调函数。

### 注意

语法：`arr.map(callback [, thisArg]);`

参数：

+   `回调()`: 此函数为新数组生成一个元素，接收以下参数：

+   `currentValue`：此参数给出正在处理的数组中的当前元素

+   `索引`：此参数给出数组中当前元素的索引

+   `数组`：此参数给出正在处理的数组

+   `thisArg()`: 此函数是可选的。在执行`回调`时，该值将被用作`this`。

示例：

```js
var
  integers = [1,-0,9,-8,3],
  numbers = [1,2,3,4],
  str = 'hello world how ya doing?';
// map integers to their absolute values
console.log(integers.map(Math.abs));

// multiply an array of numbers by their position in the array
console.log(numbers.map(function(x, i){return x*i}) );

// Capitalize every other word in a string.
console.log(str.split(' ').map(function(s, i){
  if (i%2 == 0) {
    return s.toUpperCase();
  }
  else {
    return s;
  }
}) );
```

### 注意

虽然`Array.prototype.map`方法是 JavaScript 中数组对象的标准方法，但它也可以很容易地扩展到您的自定义对象。

```js
MyObject.prototype.map = function(f) {
  return new MyObject(f(this.value));
};
```

## Array.prototype.filter()

`filter()`函数用于从数组中取出元素。回调必须返回`True`（以在新数组中包含该项）或`False`（以删除该项）。使用`map()`函数并返回要删除的项目的`null`值也可以实现类似的效果，但`filter()`函数将从新数组中删除该项，而不是在其位置插入`null`值。

### 注意

语法：`arr.filter(callback [, thisArg]);`

参数：

+   `回调()`: 此函数用于测试数组中的每个元素。返回`True`以保留该元素，否则返回`False`。具有以下参数：

+   `currentValue`：此参数给出正在处理的数组中的当前元素

+   `索引`：此参数给出数组中当前元素的索引

+   `数组`：此参数给出正在处理的数组。

+   `thisArg()`: 此函数是可选的。在执行`回调`时，该值将被用作`this`。

示例：

```js
var myarray = [1,2,3,4]
words = 'hello 123 world how 345 ya doing'.split(' ');
re = '[a-zA-Z]';
// remove all negative numbers
console.log([-2,-1,0,1,2].filter(function(x){return x>0}));
// remove null values after a map operation
console.log(words.filter(function(s){
  return s.match(re);
}) );
// remove random objects from an array
console.log(myarray.filter(function(){
  return Math.floor(Math.random()*2)})
);
```

## Array.prototype.reduce()

有时称为折叠，`reduce()`函数用于将数组的所有值累积为一个值。回调需要返回要执行的逻辑以组合对象。对于数字，它们通常相加以获得总和或相乘以获得乘积。对于字符串，通常将字符串追加在一起。

### 注意

语法：`arr.reduce(callback [, initialValue]);`

参数：

+   `回调()`: 此函数将两个对象合并为一个，并返回。具有以下参数：

+   `previousValue`：此参数给出上一次调用回调时返回的值，或者如果提供了`initialValue`，则给出`initialValue`

+   `currentValue`：此参数给出正在处理的数组中的当前元素

+   `索引`：此参数给出数组中当前元素的索引

+   `数组`：此参数给出正在处理的数组

+   `initialValue()`: 此函数是可选的。用作`回调`的第一个参数的对象。

示例：

```js
var numbers = [1,2,3,4];
// sum up all the values of an array
console.log([1,2,3,4,5].reduce(function(x,y){return x+y}, 0));
// sum up all the values of an array
console.log([1,2,3,4,5].reduce(function(x,y){return x+y}, 0));

// find the largest number
console.log(numbers.reduce(function(a,b){
  return Math.max(a,b)}) // max takes two arguments
);
```

## 荣誉提及

`map()`、`filter()`和`reduce()`函数并不是我们辅助函数工具箱中的唯一函数。还有许多其他函数可以插入到几乎任何功能应用程序中。

### Array.prototype.forEach

本质上是`map()`的非纯版本，`forEach()`遍历数组并在每个项目上应用`回调()`函数。但它不返回任何东西。这是执行`for`循环的更干净的方式。

### 注意

语法：`arr.forEach(callback [, thisArg]);`

参数：

+   `回调()`: 此函数用于对数组的每个值执行。具有以下参数：

+   `currentValue`：此参数给出正在处理的数组中的当前元素

+   `索引`：此参数给出数组中当前元素的索引

+   `数组`：此参数给出正在处理的数组

+   `thisArg`：此函数是可选的。在执行`回调`时，该值将被用作`this`。

示例：

```js
var arr = [1,2,3];
var nodes = arr.map(function(x) {
  var elem = document.createElement("div");
  elem.textContent = x;
  return elem;
});

// log the value of each item
arr.forEach(function(x){console.log(x)});

// append nodes to the DOM
nodes.forEach(function(x){document.body.appendChild(x)});
```

### Array.prototype.concat

在处理数组时，通常需要将多个数组连接在一起，而不是使用`for`和`while`循环。另一个内置的 JavaScript 函数`concat()`可以为我们处理这个问题。`concat()`函数返回一个新数组，不会改变旧数组。它可以连接你传递给它的任意多个数组。

```js
console.log([1, 2, 3].concat(['a','b','c']) // concatenate two arrays);
// Output: [1, 2, 3, 'a','b','c']
```

原始数组不受影响。它返回一个新数组，其中包含两个数组连接在一起。这也意味着`concat()`函数可以链接在一起。

```js
var arr1 = [1,2,3];
var arr2 = [4,5,6];
var arr3 = [7,8,9];
var x = arr1.concat(arr2, arr3);
var y = arr1.concat(arr2).concat(arr3));
var z = arr1.concat(arr2.concat(arr3)));
console.log(x);
console.log(y);
console.log(z);
```

变量`x`、`y`和`z`都包含`[1,2,3,4,5,6,7,8,9]`。

### Array.prototype.reverse

另一个原生 JavaScript 函数有助于数组转换。`reverse()`函数颠倒了一个数组，使得第一个元素现在是最后一个，最后一个是第一个。

然而，它不会返回一个新数组；而是就地改变数组。我们可以做得更好。下面是一个纯方法的实现，用于颠倒一个数组：

```js
var invert = function(arr) {
  return arr.map(function(x, i, a) {
    return a[a.length - (i+1)];
  });
};
var q = invert([1,2,3,4]);
console.log( q );
```

### Array.prototype.sort

与我们的`map()`、`filter()`和`reduce()`方法类似，`sort()`方法接受一个定义数组中对象应如何排序的`callback()`函数。但是，与`reverse()`函数一样，它会就地改变数组。这样做不好。

```js
arr = [200, 12, 56, 7, 344];
console.log(arr.sort(function(a,b){return a–b}) );
// arr is now: [7, 12, 56, 200, 344];
```

我们可以编写一个不会改变数组的纯`sort()`函数，但是排序算法是让人头疼的源泉。需要排序的大型数组应该被组织在专门设计用于此目的的数据结构中：quickStort、mergeSort、bubbleSort 等等。

### Array.prototype.every 和 Array.prototype.some

`Array.prototype.every()`和`Array.prototype.some()`函数都是纯函数和高阶函数，是`Array`对象的方法，用于对数组的元素进行测试，以便返回一个表示相应输入的布尔值的`callback()`函数。如果`callback()`函数对数组中的每个元素都返回`True`，则`every()`函数返回`True`，而`some()`函数返回`True`，如果数组中的一些元素为`True`。

示例：

```js
function isNumber(n) {
  return !isNaN(parseFloat(n)) && isFinite(n);
}

console.log([1, 2, 3, 4].every(isNumber)); // Return: true
console.log([1, 2, 'a'].every(isNumber)); // Return: false
console.log([1, 2, 'a'].some(isNumber)); // Return: true
```

# 摘要

为了理解函数式编程，本章涵盖了一系列广泛的主题。首先，我们分析了编程语言为函数式编程意味着什么，然后评估了 JavaScript 的函数式编程能力。接下来，我们使用 JavaScript 应用了函数式编程的核心概念，并展示了一些 JavaScript 的内置函数用于函数式编程。

尽管 JavaScript 确实有一些用于函数式编程的工具，但其函数式核心仍然大多隐藏，还有很多需要改进的地方。在下一章中，我们将探索几个用于 JavaScript 的库，这些库揭示了其函数式的本质。
