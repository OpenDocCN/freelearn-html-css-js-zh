# 第六章：JavaScript 中的高级主题和陷阱

JavaScript 被称为 Web 的"汇编语言"。这个类比（它并不完美，但哪个类比是完美的？）源自于 JavaScipt 经常是编译的目标，主要来自**Clojure**和**CoffeeScript**，但也来自许多其他来源，比如**pyjamas**（python 到 JS）和 Google Web Kit（Java 到 JS）。

但这个类比也提到了一个愚蠢的想法，即 JavaScript 和 x86 汇编一样具有表现力和低级。也许这个想法源于 JavaScript 自从 1995 年首次与网景一起发布以来就一直因其设计缺陷和疏忽而受到抨击。它是在匆忙开发和发布的，还没有完全开发就发布了。正因为如此，一些有问题的设计选择进入了 JavaScript，这种语言很快成为了 Web 的事实脚本语言。分号是一个大错误。定义函数的模糊方法也是错误的。是`var foo = function();`还是`function foo();`？

函数式编程是规避一些这些错误的绝佳方式。通过专注于 JavaScript 实际上是一种函数式语言这一事实，可以清楚地看到，在前面关于不同声明函数的方式的示例中，最好将函数声明为变量。分号大多只是为了使 JavaScript 看起来更像 C 而已。

但是，始终记住你正在使用的语言。JavaScript，像任何其他语言一样，都有其缺陷。而且，在编写通常会绕过可能的边缘的风格时，这些小失误可能会变成不可恢复的陷阱。其中一些陷阱包括：

+   递归

+   变量作用域和闭包

+   函数声明与函数表达式

然而，这些问题可以通过一点注意来克服。

# 递归

在任何语言中，递归对于函数式编程非常重要。许多函数式语言甚至要求通过不提供`for`和`while`循环语句来进行迭代，这只有在语言保证尾调用消除时才可能，而 JavaScript 并非如此。在第二章*函数式编程基础*中简要介绍了递归。但在本节中，我们将深入探讨递归在 JavaScript 中的工作原理。

## 尾递归

JavaScript 处理递归的例程被称为*尾递归*，这是一种基于堆栈的递归实现。这意味着，对于每次递归调用，堆栈中都会有一个新的帧。

为了说明这种方法可能出现的问题，让我们使用经典的递归算法来计算阶乘。

```js
var factorial = function(n) {
  if (n == 0) {
    // base case
    return 1;
  }
  else {
    // recursive case
    return n * factorial(n-1);
  }
}
```

该算法将自己调用`n`次以获得答案。它实际上计算了`(1 x 1 x 2 x 3 x … x N)`。这意味着时间复杂度是`O(n)`。

### 注意

`O(n)`，读作"大 O 到 n"，意味着算法的复杂度将随着输入规模的增长而增长，这是更精简的增长。`O(n2)`是指数增长，`O(log(n))`是对数增长，等等。这种表示法既可以用于时间复杂度，也可以用于空间复杂度。

但是，由于每次迭代都会为内存堆栈分配一个新的帧，因此空间复杂度也是`O(n)`。这是一个问题。这意味着内存将以这样的速度被消耗，以至于很容易超出内存限制。在我的笔记本电脑上，`factorial(23456)`返回`Uncaught Error: RangeError: Maximum call stack size exceeded`。

虽然计算 23456 的阶乘是一种不必要的努力，但可以肯定的是，许多使用递归解决的问题将很容易增长到这样的规模。考虑数据树的情况。树可以是任何东西：搜索应用程序、文件系统、路由表等。下面是树遍历函数的一个非常简单的实现：

```js
var traverse = function(node) {
  node.doSomething(); // whatever work needs to be done
  node.childern.forEach(traverse); // many recursive calls
}
```

每个节点只有两个子节点时，时间复杂度和空间复杂度（在最坏的情况下，整个树必须被遍历以找到答案）都将是`O(n2)`，因为每次都会有两个递归调用。如果每个节点有许多子节点，复杂度将是`O(nm)`，其中`m`是子节点的数量。递归是树遍历的首选算法；`while`循环会更加复杂，并且需要维护一个堆栈。

指数增长意味着不需要一个非常大的树就能抛出`RangeError`异常。必须有更好的方法。

### 尾调用消除

我们需要一种方法来消除每次递归调用都分配新的堆栈帧。这就是所谓的*尾调用消除*。

通过尾调用消除，当一个函数返回调用自身的结果时，语言实际上不执行另一个函数调用。它为您将整个过程转换为循环。

好的，我们该怎么做呢？使用惰性求值。如果我们可以将其重写为对惰性序列进行折叠，使得函数返回一个值或者返回调用另一个函数的结果而不对该结果进行任何操作，那么就不需要分配新的堆栈帧。

为了将其转换为“尾递归形式”，阶乘函数必须被重写，使得内部过程`fact`在控制流中最后调用自身，如下面的代码片段所示：

```js
var factorial = function(n) {
  var _fact = function(x, n) {
    if (n == 0) {
      // base case
      return x;
    }
    else {
      // recursive case
      return _fact(n*x, n-1);
    }
  }
  return fact(1, n);
}
```

### 注意

与其让递归尾部产生结果（比如`n * factorial(n-1)`），不如让结果在递归尾部进行计算（通过调用`_fact(r*n, n-1)`），并由该尾部中的最后一个函数产生结果（通过`return r;`）。计算只朝一个方向进行，而不是向上。对解释器来说，将其处理为迭代相对容易。

然而，*尾调用消除在 JavaScript 中不起作用*。将上述代码放入您喜欢的 JavaScript 引擎中，`factorial(24567)`仍然会返回`Uncaught Error: RangeError: Maximum call stack size exceeded`异常。尾调用消除被列为要包含在下一个 ECMAScript 版本中的新功能，但在所有浏览器实现它之前还需要一些时间。

JavaScript 无法优化转换为尾递归形式的函数。这是语言规范和运行时解释器的特性，简单明了。这与解释器如何获取堆栈帧的资源有关。有些语言在不需要记住任何新信息时会重用相同的堆栈帧，就像在前面的函数中一样。这就是尾调用消除如何减少时间和空间复杂度。

不幸的是，JavaScript 不会这样做。但如果它这样做了，它将重新组织堆栈帧，从这样：

```js
call factorial (3)
  call fact (3 1)
    call fact (2 3)
      call fact (1 6)
        call fact (0 6)
        return 6
      return 6
    return 6
  return 6
return 6
```

转换为以下形式：

```js
call factorial (3)
  call fact (3 1)
  call fact (2 3)
  call fact (1 6)
  call fact (0 6)
  return 6
return 6
```

## trampolining

解决方案？一种称为**trampolining**的过程。这是一种通过使用**thunks**来“黑客”尾调用消除概念的方法。

### 注意

为此，thunks 是带有参数的表达式，用于包装没有自己参数的匿名函数。例如：`function(str){return function(){console.log(str)}}`。这可以防止表达式在接收函数调用匿名函数之前被评估。

trampoline 是一个接受函数作为输入并重复执行其返回值直到返回的不再是函数的函数。以下是一个简单的实现代码片段：

```js
var trampoline = function(f) {
  while (f && f instanceof Function) {
    f = f.apply(f.context, f.args);
  }
  return f;
}
```

要实际实现尾调用消除，我们需要使用 thunks。为此，我们可以使用`bind()`函数，它允许我们将一个方法应用于具有分配给另一个对象的`this`关键字的对象。在内部，它与`call`关键字相同，但它链接到方法并返回一个新的绑定函数。`bind()`函数实际上进行了部分应用，尽管方式非常有限。

```js
var factorial = function(n) {
  var _fact = function(x, n) {
    if (n == 0) {
      // base case
      return x;
    }
    else {
      // recursive case
      return _fact.bind(null, n*x, n-1);
    }
  }
  return trampoline(_fact.bind(null, 1, n));
}
```

但是编写 `fact.bind(null, ...)` 方法很麻烦，会让任何阅读代码的人感到困惑。相反，让我们编写自己的函数来创建 thunks。`thunk()` 函数必须做一些事情：

+   `thunk()` 函数必须模拟 `_fact.bind(null, n*x, n-1)` 方法，返回一个未评估的函数

+   `thunk()` 函数应该包含另外两个函数：

+   用于处理给定函数，以及

+   用于处理函数参数，这些参数将在调用给定函数时使用

有了这些，我们就可以开始编写函数了。我们只需要几行代码就可以写出来。

```js
var thunk = function (fn) {
  return function() {
    var args = Array.prototype.slice.apply(arguments);
    return function() { return fn.apply(this, args); };
  };
};
```

现在我们可以在阶乘算法中使用 `thunk()` 函数，就像这样：

```js
var factorial = function(n) {
  var fact = function(x, n) {
    if (n == 0) {
      return x;
    }
    else {
      return thunk(fact)(n * x, n - 1);
    }
  }
  return trampoline(thunk(fact)(1, n));
}
```

但是，我们可以通过将 `_fact()` 函数定义为 `thunk()` 函数来进一步简化。通过将内部函数定义为 `thunk()` 函数，我们无需在内部函数定义中和返回语句中都使用 `thunk()` 函数。

```js
var factorial = function(n) {
  var _fact = thunk(function(x, n) {
    if (n == 0) {
      // base case
      return x;
    }
    else {
      // recursive case
      return _fact(n * x, n - 1);
    }
  });
  return trampoline(_fact(1, n));
}
```

结果是美丽的。看起来像 `_fact()` 函数被递归调用以实现无尾递归，实际上几乎透明地被处理为迭代！

最后，让我们看看 `trampoline()` 和 `thunk()` 函数如何与我们更有意义的树遍历示例一起工作。以下是使用 trampolining 和 thunks 遍历数据树的一个简单示例：

```js
var treeTraverse = function(trunk) {
  var _traverse = thunk(function(node) {
    node.doSomething();
    node.children.forEach(_traverse);
  }
  trampoline(_traverse(trunk));
}
```

我们已经解决了尾递归的问题。但是有没有更好的方法？如果我们能够简单地将递归函数转换为非递归函数呢？接下来，我们将看看如何做到这一点。

## Y 组合子

Y 组合子是计算机科学中令人惊叹的事物之一，即使是最熟练的编程大师也会感到惊讶。它自动将递归函数转换为非递归函数的能力是为什么 Douglas Crockford 称其为 "计算机科学中最奇怪和奇妙的产物"，而 Sussman 和 Steele 曾经说过，"这个方法能够工作真是了不起"。

因此，一个真正令人惊叹的、奇妙的计算机科学产物，能够让递归函数屈服，一定是庞大而复杂的，对吗？不完全是这样。它在 JavaScript 中的实现只有九行非常奇怪的代码。它们如下：

```js
var Y = function(F) {
  return (function (f) {
    return f(f);
  } (function (f) {
    return F(function (x) {
      return f(f)(x);
    });
  }));
}
```

它的工作原理是：找到作为参数传入的函数的 "不动点"。不动点提供了另一种思考函数的方式，而不是在计算机编程理论中的递归和迭代。它只使用匿名函数表达式、函数应用和变量引用来实现。请注意，`Y` 并没有引用自身。事实上，所有这些函数都是匿名的。

正如你可能已经猜到的，Y 组合子源自 λ 演算。它实际上是借助另一个称为 U 组合子的组合子推导出来的。组合子是特殊的高阶函数，它们只使用函数应用和早期定义的组合子来从输入中定义结果。

为了演示 Y 组合子，我们将再次转向阶乘问题，但我们需要以稍微不同的方式定义阶乘函数。我们不再写一个递归函数，而是写一个返回数学定义阶乘的函数。然后我们可以将这个函数传递给 Y 组合子。

```js
var FactorialGen = function(factorial) {
  return (function(n) {
    if (n == 0) {
      // base case
      return 1;
    }
    else {
      // recursive case
      return n * factorial(n – 1);
    }
  });
};
Factorial = Y(FactorialGen);
Factorial(10); // 3628800
```

然而，当我们给它一个非常大的数字时，堆栈会溢出，就像使用尾递归而没有 trampolining 一样。

```js
Factorial(23456); // RangeError: Maximum call stack size exceeded
```

但是我们可以像下面这样在 Y 组合子中使用 trampolining：

```js
var FactorialGen2 = function (factorial) {
  return function(n) {
    var factorial = thunk(function (x, n) {
      if (n == 0) {
        return x;
      }
      else {
        return factorial(n * x, n - 1);
      }
    });
    return trampoline(factorial(1, n));
  }
};

var Factorial2 = Y(FactorialGen2)
Factorial2(10); // 3628800
Factorial2(23456); // Infinity
```

我们还可以重新排列 Y 组合子以执行称为 memoization 的操作。

### Memoization

Memoization 是一种存储昂贵函数调用结果的技术。当以相同的参数再次调用函数时，将返回存储的结果，而不是重新计算结果。

尽管 Y 组合子比递归快得多，但它仍然相对较慢。为了加快速度，我们可以创建一个记忆化的不动点组合子：一个类似 Y 的组合子，它缓存中间函数调用的结果。

```js
var Ymem = function(F, cache) {
  if (!cache) {
    cache = {} ; // Create a new cache.
  }
  return function(arg) {
    if (cache[arg]) {
      // Answer in cache
      return cache[arg] ; 
    }
    // else compute the answer
    var answer = (F(function(n){
      return (Ymem(F,cache))(n);
    }))(arg); // Compute the answer.
    cache[arg] = answer; // Cache the answer.
    return answer;
  };
}
```

那么它有多快呢？通过使用[`jsperf.com/`](http://jsperf.com/)，我们可以比较性能。

以下结果是使用 1 到 100 之间的随机数。我们可以看到，记忆化的 Y 组合子要快得多。而且加上 trampolining 并不会使它变慢太多。您可以在此 URL 查看结果并运行测试：[`jsperf.com/memoizing-y-combinator-vs-tail-call-optimization/7`](http://jsperf.com/memoizing-y-combinator-vs-tail-call-optimization/7)。

![记忆化](img/00010.jpeg)

最重要的是：在 JavaScript 中执行递归的最有效和最安全的方法是使用记忆化的 Y 组合子，通过 trampolining 和 thunks 进行尾调用消除。

# 变量作用域

JavaScript 中变量的作用域并不是自然的。事实上，有时它甚至是违反直觉的。他们说 JavaScript 程序员可以通过他们对作用域的理解程度来判断。

## 作用域解析

首先，让我们来看一下 JavaScript 中不同的作用域解析。

JavaScript 使用作用域链来确定变量的作用域。在解析变量时，它从最内部的作用域开始，向外搜索。

### 全局作用域

在这个级别定义的变量、函数和对象对整个程序中的任何代码都是可用的。这是最外层的作用域。

```js
var x = 'hi';
function a() {
  console.log(x);
}
a(); // 'hi'
```

### 局部作用域

每个描述的函数都有自己的局部作用域。在另一个函数内定义的任何函数都有一个与外部函数相关联的嵌套局部作用域。几乎总是源代码中的位置定义了作用域。

```js
var x = 'hi';
function a() {
  console.log(x);
}
function b() {
  var x = 'hello';
  console.log(x);
}
b(); // hello
a(); // hi
```

局部作用域仅适用于函数，而不适用于任何表达式语句（`if`、`for`、`while`等），这与大多数语言处理作用域的方式不同。

```js
function c() {
  var y = 'greetings';
  if (true) {
    var y = 'guten tag';
  }
  console.log(y);
}

function d() {
  var y = 'greetings';
  function e() {
    var y = 'guten tag';
  }
  console.log(y)
}
c(); // 'guten tag'
d(); // 'greetings'
```

在函数式编程中，这不是太大的问题，因为函数更常用，表达式语句不太常用。例如：

```js
function e(){
  var z = 'namaste';
  [1,2,3].foreach(function(n) {
    var z = 'aloha';
  }
  isTrue(function(){
    var z = 'good morning';
  });
  console.log(z);
}
e(); // 'namaste'
```

### 对象属性

对象属性也有它们自己的作用域链。

```js
var x = 'hi';
var obj = function(){
  this.x = 'hola';
};
var foo = new obj();
console.log(foo.x); // 'hola'
foo.x = 'bonjour';
console.log(foo.x); // 'bonjour'
```

对象的原型在作用域链中更靠下。

```js
obj.prototype.x = 'greetings';
obj.prototype.y = 'konnichi ha';
var bar = new obj();
console.log(bar.x); // still prints 'hola'
console.log(bar.y); // 'konnichi ha'
```

这甚至不能算是全面的，但这三种作用域类型足以让我们开始。

## 闭包

这种作用域结构的一个问题是它不留下私有变量的空间。考虑以下代码片段：

```js
var name = 'Ford Focus';
var year = '2006';
var millage = 123456;
function getMillage(){
  return millage;
}
function updateMillage(n) {
  millage = n;
}
```

这些变量和函数是全局的，这意味着程序后面的代码很容易意外地覆盖它们。一个解决方法是将它们封装到一个函数中，并在定义后立即调用该函数。

```js
var car = function(){
  var name = 'Ford Focus';
  var year = '2006';
  var millage = 123456;
  function getMillage(){
    return Millage;
  }
  function updateMillage(n) {
    millage = n;
  }
}();
```

在函数外部没有发生任何事情，所以我们应该通过使其匿名来丢弃函数名。

```js
(function(){
  var name = 'Ford Focus';
  var year = '2006';
  var millage = 123456;
  function getMillage(){
    return millage;
  }
  function updateMillage(n) {
    millage = n;
  }
})();
```

为了使函数`getValue()`和`updateMillage()`在匿名函数外部可用，我们需要在对象字面量中返回它们，如下面的代码片段所示：

```js
var car = function(){
  var name = 'Ford Focus';
  var year = '2006';
  var millage = 123456;
  return {
    getMillage: function(){
      return millage;
    },
    updateMillage: function(n) {
      millage = n;
    }
  }
}();
console.log( car.getMillage() ); // works
console.log( car.updateMillage(n) ); // also works
console.log( car.millage ); // undefined
```

这给我们伪私有变量，但问题并不止于此。下一节将探讨 JavaScript 中变量作用域的更多问题。

## 陷阱

在 JavaScript 中可以找到许多变量作用域的微妙之处。以下绝不是一个全面的列表，但它涵盖了最常见的情况：

+   以下将输出 4，而不是人们所期望的'undefined'：

```js
for (var n = 4; false; ) { } console.log(n);
```

这是因为在 JavaScript 中，变量的定义发生在相应作用域的开头，而不仅仅是在声明时。

+   如果你在外部作用域中定义一个变量，然后在函数内部用相同的名称定义一个变量，即使那个`if`分支没有被执行，它也会被重新定义。例如：

```js
var x = 1;
function foo() {
  if (false) {
    var x = 2;
  }
  return x;
}
foo(); // Return value: 'undefined', expected return value:
2
```

同样，这是由于将变量定义移动到作用域的开头，使用`undefined`值引起的。

+   在浏览器中，全局变量实际上是存储在`window`对象中的。

```js
window.a = 19;
console.log(a); // Output: 19
```

全局作用域中的`a`表示当前上下文的属性，因此`a===this.a`，在浏览器中的`window`对象充当全局作用域中`this`关键字的等价物。

前两个示例是 JavaScript 的一个特性导致的，这个特性被称为提升，在下一节关于编写函数的内容中将是一个关键概念。

# 函数声明与函数表达式与函数构造函数

这三种声明之间有什么区别？

```js
function foo(n){ return n; }
var foo = function(n){ return n; };
var foo = new Function('n', 'return n');
```

乍一看，它们只是编写相同函数的不同方式。但这里还有更多的事情。如果我们要充分利用 JavaScript 中的函数以便将它们操纵成函数式编程风格，那么我们最好能够搞清楚这一点。如果在计算机编程中有更好的方法，那么那一种方法应该是唯一的方法。

## 函数声明

函数声明，有时称为函数语句，使用`function`关键字定义函数。

```js
function foo(n) {
  return n;
}
```

使用这种语法声明的函数会被*提升*到当前作用域的顶部。这实际上意味着，即使函数在几行下面定义，JavaScript 也知道它并且可以在作用域中较早地使用它。例如，以下内容将正确地将 6 打印到控制台：

```js
foo(2,3);
function foo(n, m) {
  console.log(n*m);
}
```

## 函数表达式

命名函数也可以通过定义匿名函数并将其赋值给变量来定义为表达式。

```js
var bar = function(n, m) {
  console.log(n*m);
};
```

它们不像函数声明那样被提升。这是因为，虽然函数声明被提升，但变量声明却没有。例如，这将无法工作并抛出错误：

```js
bar(2,3);
var bar = function(n, m) {
  console.log(n*m);
};
```

在函数式编程中，我们希望使用函数表达式，这样我们可以将函数视为变量，使它们可以用作回调和高阶函数的参数，例如`map()`函数。将函数定义为表达式使得它们更像是分配给函数的变量。此外，如果我们要以一种风格编写函数，那么为了一致性和清晰度，我们应该以该风格编写所有函数。

## 函数构造函数

JavaScript 实际上有第三种创建函数的方式：使用`Function()`构造函数。与函数表达式一样，使用`Function()`构造函数定义的函数也不会被提升。

```js
var func = new Function('n','m','return n+m');
func(2,3); // returns 5
```

但`Function()`构造函数不仅令人困惑，而且非常危险。无法进行语法纠正，也无法进行优化。以以下方式编写相同函数要容易得多、更安全、更清晰：

```js
var func = function(n,m){return n+m};
func(2,3); // returns 5
```

## 不可预测的行为

所以区别在于函数声明会被提升，而函数表达式不会。这可能会导致意想不到的事情发生。考虑以下情况：

```js
function foo() {
  return 'hi';
}
console.log(foo());
function foo() {
  return 'hello';
}
```

实际打印到控制台的是`hello`。这是因为`foo()`函数的第二个定义被提升到顶部，成为 JavaScript 解释器实际使用的定义。

虽然乍一看这可能不是一个关键的区别，在函数式编程中这可能会引起混乱。考虑以下代码片段：

```js
if (true) {
  function foo(){console.log('one')};
}
else {
  function foo(){console.log('two')};
}
foo();
```

当调用`foo()`函数时，控制台会打印`two`，而不是`one`！

最后，有一种方法可以结合函数表达式和声明。它的工作方式如下：

```js
var foo = function bar(){ console.log('hi'); };
foo(); // 'hi'
bar(); // Error: bar is not defined
```

使用这种方法几乎没有意义，因为在声明中使用的名称（在前面的示例中的`bar()`函数）在函数外部不可用，会引起混乱。只有在递归的情况下才适用，例如：

```js
var foo = function factorial(n) {
  if (n == 0) {
    return 1;
  }
else {
    return n * factorial(n-1);
  }
};
foo(5); 
```

# 总结

JavaScript 被称为“Web 的汇编语言”，因为它像 x86 汇编语言一样无处不在且不可避免。它是唯一在所有浏览器上运行的语言。它也有缺陷，但将其称为低级语言却不准确。

相反，把 JavaScript 看作是网络的生咖啡豆。当然，有些豆子是受损的，有些是腐烂的。但是如果选择好豆子，由熟练的咖啡师烘焙和冲泡，这些豆子就可以变成一杯绝妙的摩卡咖啡，一次就无法忘怀。它的消费变成了日常习惯，没有它的生活会变得单调，更难以进行，也不那么令人兴奋。一些人甚至喜欢用插件和附加组件来增强这种咖啡，比如奶油、糖和可可，这些都很好地补充了它。

JavaScript 最大的批评者之一道格拉斯·克劳福德曾说过：“肯定有很多人拒绝考虑 JavaScript 可能做对了什么。我曾经也是那些人之一。但现在我对其中的才华仍然感到惊讶。”

JavaScript 最终变得非常棒。
