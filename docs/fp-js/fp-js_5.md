# 第五章。范畴论

托马斯·沃森曾经著名地说过：“我认为世界市场上可能只需要五台计算机。”那是在 1948 年。当时，每个人都知道计算机只会用于两件事：数学和工程。甚至科技界最伟大的头脑也无法预测，有一天，计算机将能够将西班牙语翻译成英语，或者模拟整个天气系统。当时，最快的机器是 IBM 的 SSEC，每秒进行 50 次乘法运算，显示终端要等 15 年才会出现，多处理意味着多个用户终端共享一个处理器。晶体管改变了一切，但科技的远见者仍然未能抓住要点。肯·奥尔森在 1977 年又做了一个著名的愚蠢预测，他说：“没有理由让任何人在家里放一台计算机。”

现在对我们来说很明显，计算机不仅仅是为科学家和工程师准备的，但这是事后诸葛亮。70 年前，机器不仅仅能做数学这个想法一点都不直观。沃森不仅没有意识到计算机如何改变社会，他也没有意识到数学的变革和发展力量。

但是，计算机和数学的潜力并没有被所有人忽视。约翰·麦卡锡在 1958 年发明了**Lisp**，这是一种革命性的基于算法的语言，开启了计算机发展的新时代。自诞生以来，Lisp 在使用抽象层（编译器、解释器、虚拟化）推动计算机从严格的数学机器发展到今天的样子方面发挥了重要作用。

从 Lisp 出现了**Scheme**，它是 JavaScript 的直接祖先。现在我们又回到了原点。如果计算机在本质上只是做数学，那么基于数学的编程范式就会表现出色是理所当然的。

这里使用的“数学”一词并不是用来描述计算机显然可以做的“数字计算”，而是用来描述*离散数学*：研究离散数学结构的学科，比如逻辑陈述或计算机语言的指令。通过将代码视为离散数学结构，我们可以将数学中的概念和思想应用到其中。这就是为什么函数式编程在人工智能、图搜索、模式识别和计算机科学中的其他重大挑战中如此重要。

在本章中，我们将尝试一些这些概念及其在日常编程挑战中的应用。它们将包括：

+   范畴论

+   态射

+   函子

+   可能性

+   承诺

+   镜头

+   函数组合

有了这些概念，我们将能够非常轻松和安全地编写整个库和 API。我们将从解释范畴论到在 JavaScript 中正式实现它。

# 范畴论

范畴论是赋予函数组合力量的理论概念。范畴论和函数组合就像发动机排量和马力，像 NASA 和航天飞机，像好啤酒和杯子一样紧密相连。基本上，一个离不开另一个。

## 范畴论简介

范畴论实际上并不是一个太难的概念。它在数学中的地位足以填满整个研究生课程，但它在计算机编程中的地位可以很容易地总结起来。

爱因斯坦曾说过：“如果你不能向一个 6 岁的孩子解释清楚，那么你自己也不懂。”因此，在向一个 6 岁的孩子解释的精神下，*范畴论就是连接点*。虽然这可能严重简化了范畴论，但它确实很好地以直接的方式解释了我们需要知道的内容。

首先，您需要了解一些术语。**范畴**只是具有相同类型的集合。在 JavaScript 中，它们是包含明确定义为数字、字符串、布尔值、日期、节点等的变量的数组或对象。**态射**是纯函数，当给定特定的输入集时，总是返回相同的输出。**同态操作**限于单个范畴，而**多态操作**可以在多个范畴上操作。例如，同态函数*乘法*只对数字起作用，但多态函数加法也可以对字符串起作用。

![范畴论简介](img/00004.jpeg)

下图显示了三个范畴——A、B 和 C——和两个态射——*ƒ*和*ɡ*。

范畴论告诉我们，当我们有两个态射，其中第一个的范畴是另一个的预期输入时，它们可以*组合*成以下内容：

![范畴论简介](img/00005.jpeg)

*ƒ o g*符号是态射*ƒ*和*g*的组合。现在我们可以连接点了。

![范畴论简介](img/00006.jpeg)

这就是它的全部内容，只是连接点。

## 类型安全

让我们连接一些点。范畴包含两个东西：

1.  对象（在 JavaScript 中，类型）。

1.  态射（在 JavaScript 中，只对类型起作用的纯函数）。

这些是数学家给范畴论的术语，所以我们的 JavaScript 术语中存在一些不幸的命名重载。范畴论中的**对象**更像是具有显式数据类型的变量，而不是 JavaScript 对象定义中的属性和值的集合。**态射**只是使用这些类型的纯函数。

因此，将范畴论的思想应用到 JavaScript 中非常容易。在 JavaScript 中使用范畴论意味着每个范畴都使用一种特定的数据类型。数据类型包括数字、字符串、数组、日期、对象、布尔值等。但是，在 JavaScript 中没有严格的类型系统，事情可能会出错。因此，我们将不得不实现自己的方法来确保数据是正确的。

JavaScript 中有四种原始数据类型：数字、字符串、布尔值和函数。我们可以创建*类型安全函数*，它们要么返回变量，要么抛出错误。*这满足了范畴的对象公理*。

```js
var str = function(s) {
  if (typeof s === "string") {
    return s;
  }
  else {
    throw new TypeError("Error: String expected, " + typeof s + " given.");   
  }
}
var num = function(n) {
  if (typeof n === "number") {
    return n;
  }
  else {
    throw new TypeError("Error: Number expected, " + typeof n + " given.");   
  }
}
var bool = function(b) {
  if (typeof b === "boolean") {
    return b;
  }
  else {
    throw new TypeError("Error: Boolean expected, " + typeof b + " given.");   
  }
}
var func = function(f) {
  if (typeof f === "function") {
    return f;
  }
  else {
    throw new TypeError("Error: Function expected, " + typeof f + " given.");   
  }
}
```

然而，这里有很多重复的代码，这并不是很实用。相反，我们可以创建一个返回另一个函数的函数，这个函数是类型安全函数。

```js
var typeOf = function(type) {
  return function(x) {
    if (typeof x === type) {
      return x;
    }
    else {
      throw new TypeError("Error: "+type+" expected, "+typeof x+" given.");
    }
  }
}
var str = typeOf('string'),
  num = typeOf('number'),
  func = typeOf('function'),
  bool = typeOf('boolean');
```

现在，我们可以使用它们来确保我们的函数表现如预期。

```js
// unprotected method:
var x = '24';
x + 1; // will return '241', not 25

// protected method
// plusplus :: Int -> Int
function plusplus(n) {
  return num(n) + 1;
}
plusplus(x); // throws error, preferred over unexpected output
```

让我们看一个更有意思的例子。如果我们想要检查由 JavaScript 函数`Date.parse()`返回的 Unix 时间戳的长度，而不是作为字符串而是作为数字，那么我们将不得不使用我们的`str()`函数。

```js
// timestampLength :: String -> Int
function timestampLength(t) { return num(**str(t)**.length); }
timestampLength(Date.parse('12/31/1999')); // throws error
timestampLength(Date.parse('12/31/1999')
  .toString()); // returns 12
```

像这样明确地将一种类型转换为另一种类型（或相同类型）的函数被称为*态射*。*这满足了范畴论的态射公理*。通过类型安全函数和使用它们的态射强制类型声明，这些都是我们在 JavaScript 中表示范畴概念所需要的一切。

### 对象标识

还有一个重要的数据类型：对象。

```js
var obj = typeOf('object');
obj(123); // throws error
obj({x:'a'}); // returns {x:'a'}
```

然而，对象是不同的。它们可以被继承。除了原始的数字、字符串、布尔值和函数之外，一切都是对象，包括数组、日期、元素等。

没有办法知道某个对象是什么类型，比如从`typeof`关键字知道 JavaScript 的一个子类型是什么，所以我们将不得不 improvisation。对象有一个`toString()`函数，我们可以利用它来实现这个目的。

```js
var obj = function(o) {
  if (Object.prototype.toString.call(o)==="[object Object]") {
    return o;
  }
  else {
    throw new TypeError("Error: Object expected, something else given."); 
  }
}
```

再次，有了所有这些对象，我们应该实现一些代码重用。

```js
var objectTypeOf = function(name) {
  return function(o) {
    if (Object.prototype.toString.call(o) === "[object "+name+"]") {
      return o;
    }
    else {
      throw new TypeError("Error: '+name+' expected, something else given.");
    }
  }
}
var obj = objectTypeOf('Object');
var arr = objectTypeOf('Array');
var date = objectTypeOf('Date');
var div = objectTypeOf('HTMLDivElement');
```

这些将对我们接下来的主题非常有用：函子。

# 函子

虽然态射是类型之间的映射，*函数器*是范畴之间的映射。它们可以被看作是将值从容器中提取出来，对其进行态射，然后将其放入新的容器中的函数。第一个输入是类型的态射，第二个输入是容器。

### 注意

函数器的类型签名如下：

```js
// myFunctor :: (a -> b) -> f a -> f b
```

这意味着，“给我一个接受`a`并返回`b`的函数和一个包含`a`的盒子，我会返回一个包含`b`的盒子”。

## 创建函数器

事实证明，我们已经有一个函数器：`map()`。它从容器中（数组）获取值，并对其应用函数。

```js
[1, 4, 9].map(Math.sqrt); // Returns: [1, 2, 3]
```

但是，我们需要将其编写为全局函数，而不是数组对象的方法。这将使我们能够以后编写更清洁、更安全的代码。

```js
// map :: (a -> b) -> [a] -> [b]
var map = function(f, a) {
  return arr(a).map(func(f));
}
```

这个例子看起来像一个人为的包装，因为我们只是依赖`map()`函数。但它有一个目的。它为其他类型的映射提供了一个模板。

```js
// strmap :: (str -> str) -> str -> str
var strmap = function(f, s) {
  return str(s).split('').map(func(f)).join('');
}

// MyObject#map :: (myValue -> a) -> a
MyObject.prototype.map(f{
  return func(f)(this.myValue);
}
```

## 数组和函数器

在函数式 JavaScript 中，数组是处理数据的首选方式。

有没有一种更简单的方法来创建已经分配给态射的函数器？是的，它被称为`arrayOf`。当你传入一个期望整数并返回一个数组的态射时，你会得到一个期望整数数组并返回一个数组的态射。

它本身不是一个函数器，但它允许我们从态射创建函数器。

```js
// arrayOf :: (a -> b) -> ([a] -> [b])
var arrayOf = function(f) {
  return function(a) {
    return map(func(f), arr(a));
  }
}
```

以下是如何使用态射创建函数器：

```js
var plusplusall = arrayOf(plusplus); // plusplus is our morphism
console.log( plusplusall([1,2,3]) ); // returns [2,3,4]
console.log( plusplusall([1,'2',3]) ); // error is thrown
```

`arrayOf`函数器的有趣属性是它也适用于类型安全。当你传入字符串的类型安全函数时，你会得到一个字符串数组的类型安全函数。类型安全被视为*恒等函数*态射。这对于确保数组包含所有正确的类型非常有用。

```js
var strs = arrayOf(str);
console.log( strs(['a','b','c']) ); // returns ['a','b','c']
console.log( strs(['a',2,'c']) ); // throws error
```

## 重新审视函数组合

函数是我们可以为其创建一个函数器的另一种原始类型。这个函数器被称为`fcompose`。我们将函数器定义为从容器中取出一个值并对其应用函数的东西。当容器是一个函数时，我们只需调用它以获取其内部值。

我们已经知道函数组合是什么，但让我们看看它们在范畴论驱动的环境中能做什么。

函数组合是可结合的。如果你的高中代数老师像我的一样，她教你这个性质*是*什么，但没有教你它能*做*什么。在实践中，组合是可结合性能做的事情。

![重新审视函数组合](img/00007.jpeg)![重新审视函数组合](img/00008.jpeg)

我们可以进行任何内部组合，不管它是如何分组的。这与交换律属性不同。*ƒ o g*并不总是等于*g o ƒ*。换句话说，字符串的第一个单词的反向不同于字符串反向的第一个单词。

这一切意味着，不管应用了哪些函数以及顺序如何，只要每个函数的输入来自前一个函数的输出，就没有关系。但是，等等，如果右边的函数依赖于左边的函数，那么难道只能有一种评估顺序吗？从左到右？是的，但如果它被封装起来，那么我们可以根据自己的意愿来控制它。这就是 JavaScript 中懒惰评估的强大之处。

![重新审视函数组合](img/00009.jpeg)

让我们重新编写函数组合，不是作为函数原型的扩展，而是作为一个独立的函数，这将允许我们更充分地利用它。基本形式如下：

```js
var fcompose = function(f, g) {
  return function() {
    return f.call(this, g.apply(this, arguments));
  };
};
```

但是我们需要它能够处理任意数量的输入。

```js
var fcompose = function() {
  // first make sure all arguments are functions
  var funcs = arrayOf(func)(arguments);

  // return a function that applies all the functions
  return function() {
    var argsOfFuncs = arguments;
    for (var i = funcs.length; i > 0; i -= 1) {
      argsOfFuncs  = [funcs[i].apply(this, args)];
    }
    return args[0];
  };
};

// example:
var f = fcompose(negate, square, mult2, add1);
f(2); // Returns: -36
```

现在我们已经封装了这些函数，我们对它们有了控制。我们可以重写 compose 函数，使得*每个函数都接受另一个函数作为输入，存储它，并返回一个做同样事情的对象*。我们可以接受一个单一数组作为输入，对源中的每个元素执行所有操作（每个`map()`、`filter()`等等，组合在一起），最后将结果存储在一个新数组中。这是通过函数组合实现的惰性评估。这里没有理由重新发明轮子。许多库都有这个概念的很好的实现，包括`Lazy.js`、`Bacon.js`和`wu.js`库。

这种不同的模式使我们能够做更多的事情：异步迭代，异步事件处理，惰性评估，甚至自动并行化。

### 注意

自动并行化？在计算机科学行业中有一个词：不可能。但它真的不可能吗？摩尔定律的下一个进化飞跃可能是一个为我们的代码并行化的编译器，而函数组合可能就是这样的编译器？

不，事情并不完全是这样的。JavaScript 引擎才是真正进行并行化的，不是自动的，而是通过深思熟虑的代码。Compose 只是给引擎一个机会将其拆分成并行进程。但这本身就很酷。

# 单子

**单子**是帮助您组合函数的工具。

像原始类型一样，单子是可以用作函子“触及”的容器的结构。函子抓取数据，对其进行处理，将其放入一个新的单子中，并返回它。

我们将专注于三个单子：

+   Maybes

+   承诺

+   Lenses

所以除了数组（map）和函数（compose）之外，我们还有五个函子（map、compose、maybe、promise 和 lens）。这些只是许多其他函子和单子中的一些。

## Maybes

Maybes 允许我们优雅地处理可能为空的数据，并设置默认值。也就是说，maybe 是一个变量，它要么有一些值，要么没有。而对调用者来说这并不重要。

单独看起来，这似乎并不是什么大不了的事。每个人都知道，使用`if-else`语句很容易实现空检查：

```js
if (getUsername() == null ) {
  username = 'Anonymous') {
else {
  username = getUsername();
}
```

但是通过函数式编程，我们正在摆脱逐行进行程序的程序化方式，而是使用函数和数据的管道。如果我们必须在中间打断链条来检查值是否存在，我们将不得不创建临时变量并编写更多的代码。Maybes 只是帮助我们保持逻辑在管道中流动的工具。

为了实现 maybes，我们首先需要创建一些构造函数。

```js
// the Maybe monad constructor, empty for now
var Maybe = function(){}; 

// the None instance, a wrapper for an object with no value
var None = function(){}; 
None.prototype = Object.create(Maybe.prototype);
None.prototype.toString = function(){return 'None';};

// now we can write the `none` function
// saves us from having to write `new None()` all the time
var none = function(){return new None()};

// and the Just instance, a wrapper for an object with a value
var Just = function(x){return this.x = x;};
Just.prototype = Object.create(Maybe.prototype);
Just.prototype.toString = function(){return "Just "+this.x;};
var just = function(x) {return new Just(x)};
```

最后，我们可以编写`maybe`函数。它返回一个新的函数，要么返回空，要么返回一个 maybe。*它是一个函子*。

```js
var maybe = function(m){
  if (m instanceof None) {
    return m;
  }
  else if (m instanceof Just) {
    return just(m.x);   
  }
  else {
    throw new TypeError("Error: Just or None expected, " + m.toString() + " given."); 
  }
}
```

我们也可以创建一个与数组类似的函子生成器。

```js
var maybeOf = function(f){
  return function(m) {
    if (m instanceof None) {
      return m;
    }
    else if (m instanceof Just) {
      return just(f(m.x));
    }
    else {
      throw new TypeError("Error: Just or None expected, " + m.toString() + " given."); 
    }
  }
}
```

所以`Maybe`是一个单子，`maybe`是一个函子，`maybeOf`返回一个已经分配给一个态射的函子。

在我们继续之前，我们还需要一件事。我们需要为`Maybe`单子对象添加一个帮助我们更直观地使用它的方法。

```js
Maybe.prototype.orElse = function(y) {
  if (this instanceof Just) {
    return this.x;
  }
  else {
    return y;
  }
}
```

在其原始形式中，maybes 可以直接使用。

```js
maybe(just(123)).x; // Returns 123
maybeOf(plusplus)(just(123)).x; // Returns 124
maybe(plusplus)(none()).orElse('none'); // returns 'none'
```

任何返回一个然后执行的方法都足够复杂，以至于容易出问题。所以我们可以通过调用我们的`curry()`函数来使它更加简洁。

```js
maybePlusPlus = maybeOf.curry()(plusplus);
maybePlusPlus(just(123)).x; // returns 123
maybePlusPlus(none()).orElse('none'); // returns none
```

但是当直接调用`none()`和`just()`函数的肮脏业务被抽象化时，maybes 的真正力量将变得清晰。我们将通过一个使用 maybes 的示例对象`User`来做到这一点。

```js
var User = function(){
  this.username = none(); // initially set to `none`
};
User.prototype.setUsername = function(name) {
  this.username = just(str(name)); // it's now a `just
};
User.prototype.getUsernameMaybe = function() {
  var usernameMaybe = maybeOf.curry()(str);
  return usernameMaybe(this.username).orElse('anonymous');
};

var user = new User();
user.getUsernameMaybe(); // Returns 'anonymous'

user.setUsername('Laura');
user.getUsernameMaybe(); // Returns 'Laura'
```

现在我们有了一个强大而安全的方法来定义默认值。记住这个`User`对象，因为我们将在本章后面使用它。

## 承诺

> *承诺的本质是它们对变化的情况保持免疫。*
> 
> *- 弗兰克·安德伍德，《纸牌屋》*

在函数式编程中，我们经常使用管道和数据流：一系列函数，其中每个函数产生一个数据类型，由下一个函数消耗。然而，许多这些函数是异步的：readFile、事件、AJAX 等。我们如何修改这些函数的返回类型来指示结果，而不是使用延续传递风格和深度嵌套的回调？通过将它们包装在*promises*中。

Promises 就像回调的函数式等价物。显然，回调并不都是函数式的，因为如果有多个函数对同一数据进行变异，就会出现竞争条件和错误。Promises 解决了这个问题。

你应该使用 promises 来完成这个：

```js
fs.readFile("file.json", function(err, val) {
  if( err ) {
    console.error("unable to read file");
  }
  else {
    try {
      val = JSON.parse(val);
      console.log(val.success);
    }
    catch( e ) {
      console.error("invalid json in file");
    }
  }
});
```

进入以下代码片段：

```js
fs.readFileAsync("file.json").then(JSON.parse)
  .then(function(val) {
    console.log(val.success);
  })
  .catch(SyntaxError, function(e) {
    console.error("invalid json in file");
  })
  .catch(function(e){
    console.error("unable to read file")
  });
```

上述代码来自*bluebird*的 README：一个功能齐全的*Promises/A+*实现，性能异常出色。*Promises/A+*是 JavaScript 中实现 promises 的规范。鉴于它在 JavaScript 社区内的当前辩论，我们将把实现留给*Promises/A+*团队，因为它比可能更复杂得多。

但这是一个部分实现：

```js
// the Promise monad
var Promise = require('bluebird');

// the promise functor
var promise = function(fn, receiver) {
  return function() {
    var slice = Array.prototype.slice,
    args = slice.call(arguments, 0, fn.length - 1),
    promise = new Promise();
    args.push(function() {
      var results = slice.call(arguments),
      error = results.shift();
      if (error) promise.reject(error);
      else promise.resolve.apply(promise, results);
    });
    fn.apply(receiver, args);
    return promise;
  };
};
```

现在我们可以使用`promise()`函子将接受回调的函数转换为返回 promises 的函数。

```js
var files = ['a.json', 'b.json', 'c.json'];
readFileAsync = promise(fs.readFile);
var data = files
  .map(function(f){
    readFileAsync(f).then(JSON.parse)
  })
  .reduce(function(a,b){
    return $.extend({}, a, b)
  });
```

## 镜头

程序员真正喜欢单子的另一个原因是，它们使编写库变得非常容易。为了探索这一点，让我们扩展我们的`User`对象，增加更多用于获取和设置值的函数，但是，我们将使用*lenses*而不是使用 getter 和 setter。

镜头是一流的获取器和设置器。它们不仅允许我们获取和设置变量，还允许我们在其上运行函数。但是，它们不是对数据进行变异，而是克隆并返回由函数修改的新数据。它们强制数据是不可变的，这对于安全性和一致性以及库来说非常好。无论应用程序如何，它们都非常适合优雅的代码，只要引入额外的数组副本不会对性能造成重大影响。

在编写`lens()`函数之前，让我们看看它是如何工作的。

```js
var first = lens(
  function (a) { return arr(a)[0]; }, // get
  function (a, b) { return [b].concat(arr(a).slice(1)); } // set
);
first([1, 2, 3]); // outputs 1
first.set([1, 2, 3], 5); // outputs [5, 2, 3]
function tenTimes(x) { return x * 10 }
first.modify(tenTimes, [1,2,3]); // outputs [10,2,3]
```

这就是`lens()`函数的工作原理。它返回一个具有 get、set 和 mod 定义的函数。`lens()`函数本身是一个函子。

```js
var lens = fuction(get, set) {
  var f = function (a) {return get(a)};
  f.get = function (a) {return get(a)}; 
  f.set = set;
  f.mod = function (f, a) {return set(a, f(get(a)))};
  return f;
};
```

让我们来试一个例子。我们将扩展我们之前例子中的`User`对象。

```js
// userName :: User -> str
var userName = lens(
  function (u) {return u.getUsernameMaybe()}, // get
  function (u, v) { // set
    u.setUsername(v);  
    return u.getUsernameMaybe(); 
  }
);

var bob = new User();
bob.setUsername('Bob');
userName.get(bob); // returns 'Bob'
userName.set(bob, 'Bobby'); //return 'Bobby'
userName.get(bob); // returns 'Bobby'
userName.mod(strToUpper, bob); // returns 'BOBBY'
strToUpper.compose(userName.set)(bob, 'robert'); // returns 'ROBERT'
userName.get(bob); // returns 'robert'
```

## jQuery 是一个单子

如果你认为所有这些关于范畴、函子和单子的抽象胡言没有真正的现实应用，那就再想想吧。流行的 JavaScript 库 jQuery 提供了一个增强的接口，用于处理 HTML，实际上是一个单子库。

`jQuery`对象是一个单子，它的方法是函子。实际上，它们是一种特殊类型的函子，称为*endofunctors*。**Endofunctors**是返回与输入相同类别的函子，即`F :: X -> X`。每个`jQuery`方法都接受一个`jQuery`对象并返回一个`jQuery`对象，这允许方法被链接，并且它们将具有类型签名`jFunc :: jquery-obj -> jquery-obj`。

```js
$('li').add('p.me-too').css('color', 'red').attr({id:'foo'});
```

这也是 jQuery 的插件框架的强大之处。如果插件以`jQuery`对象作为输入并返回一个作为输出，则可以将其插入到链中。

让我们看看 jQuery 是如何实现这一点的。

单子是函子“触及”以获取数据的容器。通过这种方式，数据可以受到库的保护和控制。jQuery 通过其许多方法提供对底层数据的访问，这些数据是一组包装的 HTML 元素。

`jQuery`对象本身是作为匿名函数调用的结果编写的。

```js
var jQuery = (function () {
  var j = function (selector, context) {
    var jq-obj = new j.fn.init(selector, context);
    return jq-obj;
  };

  j.fn = j.prototype = {
    init: function (selector, context) {
      if (!selector) {
        return this;
      }
    }
  };
  j.fn.init.prototype = j.fn;
  return j;
})();
```

在这个高度简化的 jQuery 版本中，它返回一个定义了`j`对象的函数，实际上只是一个增强的`init`构造函数。

```js
var $ = jQuery(); // the function is returned and assigned to `$`
var x = $('#select-me'); // jQuery object is returned
```

与函子将值提取出容器的方式相同，jQuery 包装了 HTML 元素并提供对它们的访问，而不是直接修改 HTML 元素。

jQuery 并不经常宣传，但它有自己的`map()`方法，用于将 HTML 元素对象从包装器中提取出来。就像`fmap()`方法一样，元素被提取出来，对它们进行处理，然后放回容器中。这就是 jQuery 的许多命令在后端工作的方式。

```js
$('li').map(function(index, element) {
  // do something to the element
  return element
});
```

另一个用于处理 HTML 元素的库 Prototype 不是这样工作的。Prototype 通过助手直接改变 HTML 元素。因此，它在 JavaScript 社区中的表现并不好。

# 实施类别

是时候我们正式将范畴论定义为 JavaScript 对象了。范畴是对象（类型）和态射（仅在这些类型上工作的函数）。这是一种非常高级的、完全声明式的编程方式，但它确保代码非常安全可靠——非常适合担心并发和类型安全的 API 和库。

首先，我们需要一个帮助我们创建同态的函数。我们将其称为`homoMorph()`，因为它们将是同态。它将返回一个函数，该函数期望传入一个函数，并根据输入生成其组合。输入是态射接受的输入和输出的类型。就像我们的类型签名一样，即`// morph :: num -> num -> [num]`，只有最后一个是输出。

```js
var homoMorph = function( /* input1, input2,..., inputN, output */ ) {
  var before = checkTypes(arrayOf(func)(Array.prototype.slice.call(arguments, 0, arguments.length-1)));
  var after = func(arguments[arguments.length-1])
  return function(middle) {
    return function(args) {
      return after(middle.apply(this, before([].slice.apply(arguments))));   
    }
  }
}

// now we don't need to add type signature comments
// because now they're built right into the function declaration
add = homoMorph(num, num, num)(function(a,b){return a+b})
add(12,24); // returns 36
add('a', 'b'); // throws error
homoMorph(num, num, num)(function(a,b){
  return a+b;
})(18, 24); // returns 42
```

`homoMorph()`函数相当复杂。它使用闭包（参见第二章，“函数式编程基础”）返回一个接受函数并检查其输入和输出值的类型安全性的函数。为此，它依赖于一个辅助函数：`checkTypes`，其定义如下：

```js
var checkTypes = function( typeSafeties ) {
  arrayOf(func)(arr(typeSafeties));
  var argLength = typeSafeties.length;
  return function(args) {
    arr(args);
    if (args.length != argLength) {
      throw new TypeError('Expected '+ argLength + ' arguments');
    }
    var results = [];
    for (var i=0; i<argLength; i++) {
      results[i] = typeSafetiesi;   
    }
    return results;
  }
}
```

现在让我们正式定义一些同态。

```js
var lensHM = homoMorph(func, func, func)(lens);
var userNameHM = lensHM(
  function (u) {return u.getUsernameMaybe()}, // get
  function (u, v) { // set
    u.setUsername(v);
    return u.getUsernameMaybe(); 
  }
)
var strToUpperCase = homoMorph(str, str)(function(s) {
  return s.toUpperCase();
});
var morphFirstLetter = homoMorph(func, str, str)(function(f, s) {
  return f(s[0]).concat(s.slice(1));
});
var capFirstLetter = homoMorph(str, str)(function(s) {
  return morphFirstLetter(strToUpperCase, s)
});
```

最后，我们可以把它带回家。以下示例包括函数组合、镜头、同态和其他内容。

```js
// homomorphic lenses
var bill = new User();
userNameHM.set(bill, 'William'); // Returns: 'William'
userNameHM.get(bill); // Returns: 'William'

// compose
var capatolizedUsername = fcompose(capFirstLetter,userNameHM.get);
capatolizedUsername(bill, 'bill'); // Returns: 'Bill'

// it's a good idea to use homoMorph on .set and .get too
var getUserName = homoMorph(obj, str)(userNameHM.get);
var setUserName = homoMorph(obj, str, str)(userNameHM.set);
getUserName(bill); // Returns: 'Bill'
setUserName(bill, 'Billy'); // Returns: 'Billy'

// now we can rewrite capatolizeUsername with the new setter
capatolizedUsername = fcompose(capFirstLetter, setUserName);
capatolizedUsername(bill, 'will'); // Returns: 'Will'
getUserName(bill); // Returns: 'will'
```

前面的代码非常声明式，安全，可靠和可信赖。

### 注

代码声明式是什么意思？在命令式编程中，我们编写一系列指令，告诉机器如何做我们想要的事情。在函数式编程中，我们描述值之间的关系，告诉机器我们想要它计算什么，机器会找出指令序列来实现它。函数式编程是声明式的。

整个库和 API 可以通过这种方式构建，允许程序员自由编写代码，而不必担心并发和类型安全，因为这些问题在后端处理。

# 总结

大约每 2000 人中就有一人患有一种称为共感觉的病症，这是一种神经现象，其中一种感官输入渗入另一种感官。最常见的形式涉及将颜色与字母相匹配。然而，还有一种更罕见的形式，即将句子和段落与味道和感觉联系起来。

对于这些人来说，他们不是逐字逐句地阅读。他们看整个页面/文档/程序，感受它的“味道”——不是口中的味道，而是“心灵”中的味道。然后他们像拼图一样把文本的部分放在一起。

这就是编写完全声明式代码的样子：描述值之间的关系，告诉机器我们想要它计算什么。程序的部分不是按照逐行指令。共感者可能自然而然地做到这一点，但只要稍加练习，任何人都可以学会如何将关系拼图一起放在一起。

在本章中，我们看了几个数学概念，这些概念适用于函数式编程，以及它们如何允许我们在数据之间建立关系。接下来，我们将探讨递归和 JavaScript 中的其他高级主题。
