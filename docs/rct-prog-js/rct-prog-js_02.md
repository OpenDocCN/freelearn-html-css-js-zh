# 第二章。核心 JavaScript

JavaScript 是一个好坏参半的语言。JavaScript 有一些真正糟糕的部分和一些优秀的部分。总的来说，JavaScript 是一种动态的、弱类型的、解释性的脚本语言。对整个语言的处理是重点探索*好的部分*，正如 Douglas Crockford 所描述的那样，因为 JavaScript 的坏部分有多糟糕：它们确实是雷区。Crockford 对 JavaScript 的现在的使用产生了深远的影响；足以让题为*AngularJS：坏部分*的博客文章立即、清楚、完全地传达了将会被痛苦细致地剖析的事物。

本章涵盖的主题包括：

+   **严格模式**：隐式用于包括 ECMAScript 6 模块的代码

+   **变量和赋值**：任何语言中编程的基本构建块之一

+   **注释**：有两种风格；我们更偏向于一种，因为通过在程序员意图之外的地方开始或结束注释，很容易产生意外行为

+   **流程控制**：基本的 if-then、if-then-else 和 switch 语句的简要介绍

+   **关于值和 NaN 的注释**：关于真值和有毒的*非数字*值的注释

+   **函数**：JavaScript 函数的示例，是语言中最好的部分之一

+   **关于 ECMAScript 6 的一些简要注释**：多年来核心 JavaScript 的第一个真正的根本性变化

# 严格模式

“use strict”;作为文件或函数的第一行将导致某些可能引起无数问题的事情（例如在没有声明的情况下对变量进行赋值）以清晰的错误行号报告错误，而不是让您从可能的多种后果中去猜测线索。 “use strict”;也可以是函数的第一行，在这种情况下，函数处于严格模式。

Perl 用户将了解`-w`标志，可能是与该语言相关的最著名的标志，以及它的后继者 Perl 的使用警告的惯例。文档中说了一些事情，比如，打开已知错误列表，“使用警告所暗示的行为不是强制性的”。

JavaScript 的严格模式本身可能与 Perl 的使用警告的惯例不相上下，但至少可以让您养成使用的习惯。

# 变量和赋值

变量应该使用`var`关键字声明。在函数外声明的任何变量，或者在没有声明的情况下使用的任何变量，都是全局变量，全局变量是一个大问题；它们在*默认 JavaScript*中的位置是主要的设计缺陷之一。

当 Java 进行了一次重大的公共关系推动时，JavaScript 被命名为在 Java 的影响下运行，并且做出了某些决定，使 JavaScript 代码看起来像 Java。这些决定是不幸的。JavaScript 在形式上是一种类似 C 的语言，它与 Java 或 C#的最近共同祖先是 C，而不是 C++或 Java。JavaScript 被描述为穿着 C 的外衣的 Scheme。Lisp 是与包括 Scheme、Common Lisp、Clojure 和 ClojureScript 在内的一系列语言相关的语法，可以说，追求最佳 JavaScript 的日益功能性的重点来自 ClojureScript。在 JavaScript 中，没有单独的整数和浮点类型；数字是 64 位浮点值，在一定的长范围内使用时表现得像整数。然而，它们有时会给新程序员带来惊喜；例如，0.1 + 0.2 并不等于 0.3，出于历史原因，这也困扰着其他无数语言。

基本变量赋值看起来像 C 语言：

```js
var x;
var y = 12 + 2;
```

如果变量声明但未赋值，如前面示例中的`x`，其值将为 undefined。

以下是等效的：

```js
y = y + 1;
y += 1;
++y;
```

我们将避免在前面示例中列出的最后一个选项的使用，因为它不被认为是好的部分之一。道格拉斯·克罗克福德在其中的一个视频中讲述了一个故事，他在一个小时的辩护中对`++y`的使用进行了精彩的辩护，然后进行了一场漫长的调试会话，在这场调试会话中，`++y`的微妙之处已经咬了他一口。与前面示例中的前两个选项不同，可以为其分配一个值，并且`++y`和`y++`不相同。克罗克福德随后慷慨地放弃了他先前的立场。

## 注释

大多数语言都支持某种形式的注释。除了对代码的解释外，它们还用于临时停用代码。JavaScript 具有与 Java 相似的语言所期望的注释；但是，Javadoc 注释并不是本地特殊的（已经制定了各种解决方案，如 JSDoc 来填补这一空白）。

JavaScript 支持 C++注释的两类。前三行只包含注释而没有可执行代码，最后一行有一行代码，然后一直到行尾都是注释：

```js
/*
 * Multiline comments are legal.
 */

if (x) // In this case, we ...
```

我们将避免多行注释。星号和斜杠在正则表达式中经常出现，多行注释可能会引起问题，并且可能需要上下文或由其他人编写的代码中仍然会引起意外效果。内联注释本质上不太容易受到意外效果的影响。

## 流程控制

如果-然后和如果-然后-否则按照以下示例代码的描述工作，其中一个在数字非零时执行某些操作，另一个在数字非零时执行一个动作，如果为假则执行另一个动作。两者都使用了真值，其中 0（和 NaN）为假，而任何其他值为真：

```js
if (books_remaining) {
  print_title();
}

var c = 1;
if (c) {
  c += 2;
} else {
  c -= 1;
}
```

# 关于值和 NaN 的注释

所有值都可以在布尔上下文中使用并进行真值测试。值`undefined`、`null`、`''`、`0`、`[]`、`false`和`NaN`（不是数字）都是`假值`，所有其他值都是`真值`。

`NaN`特别是一个特殊情况，它的行为不像其他*真实*数值。`NaN`是有毒的；包含`NaN`的任何计算都将得到`NaN`的结果。此外，尽管`NaN`是`假值`，但它不等于任何东西，包括`NaN`本身。检查`NaN`的通常方法是通过`isNaN()`函数。如果您发现`NaN`潜伏在某个意想不到的地方，您可能会为代码提供调试日志语句，指导您检测到`NaN`的位置；在生成`NaN`的地方和观察到它破坏结果的地方之间可能存在一定的距离。

## 函数

在函数中，默认情况下，控制从开始到结束，函数返回`undefined`。可选地，在结束之前可以有一个返回语句，函数将停止执行并返回相应的值。以下示例说明了一个带有返回值的函数：

```js
var add = function(first, second) {
  return first + second;
}

console.log(add(1, 2));

// 3
```

前面的函数接受两个参数，尽管函数可以给出（没有错误或错误消息）少于或多于声明指定的参数。如果它们被声明为具有值，这些值将作为局部变量存在（在前面的示例中，`first`和`second`）。无论如何，这些参数也可以在一个类似数组的对象`arguments`中使用，该对象具有`.length`方法（数组具有`.length`方法，比项的最高位置大 1），但不具有数组的其他特性。在这里，我们创建一个函数，该函数可以接受任意数量的*数字参数*，并通过使用`arguments`伪数组返回它们的（算术）平均值。

```js
var average_arbitrarily_many() {
  if (!arguments.length) {
    return 0;
  }
  var count = 0;
  var total = 0;
  for(var index = 0; index < arguments.length; index += 1) {
    total += arguments[i];
  }
  return total / arguments.length;
}
```

基本数据类型包括数字、字符串、布尔值、符号、对象、null 和 undefined。对象包括函数、数组、日期和正则表达式。

对象包括函数意味着函数是值，可以作为值传递。这有助于高阶函数，或者将函数作为值传递的函数。

作为高阶函数的一个例子，我们将包括一个 `sort` 函数，它对数组进行排序并可选择接受一个比较函数。这建立在函数定义上，实际上包含了一个函数定义在另一个函数定义中（这与其他任何地方一样合法），然后是一个 QuickSort 的实现，其中值被分为 *比第一个元素小*、*等于第一个元素* 和 *比第一个元素大* 进行比较，并且这三个中的第一个和最后一个被递归排序。我们在排序之前检查非空长度，以避免无限递归。作为高级函数实现的经典 QuickSort 算法如下：

```js
var sort = function(array, comparator) {
  if (typeof comparator === 'undefined') {
    comparator = function(first, second) {
      if (first < second) {
        return -1;
      } else if (second < first) {
        return 1;
      } else {
        return 0;
      }
    }
  }
  var before = [];
  var same = [];
  var after = [];
  if (array.length) {
    same.push(array[0]);
  }
  for(var i = 1; i < array.length; ++i) {
    if (comparator(array[i], array[0]) < 0) {
      before.push(array[i]);
    } else if (comparator(array[i], array[0]) > 0) {
      after.push(array[i]);
    } else {
      same.push(array[i]);
    }
  }
  var result = [];
  if (before.length) {
    result = result.concat(sort(before, comparator));
  }
  result = result.concat(same);
  if (after.length) {
    result = result.concat(sort(after, comparator));
  }
  return result;
}
```

### 注释

有几个关于这个函数的基本特性和观察需要注意，这并不是要突破界限，而是要展示如何很好地覆盖标准基础：

1.  我们使用 `var sort = function()...` 而不是允许的 `function sort()...`。当在函数内部使用时，这将函数存储在一个变量中，而不是在全局定义某些东西。请注意，在调试时，为函数包括一个名称可能会有所帮助，`var sort = function sort()...`，只能通过变量访问函数，并让调试器捕捉到第二个。例如：`sort` 而不是匿名地引用函数。请注意，使用 `var sort = function()`，变量声明被提升，而不是初始化；使用 `function sort()`，函数值被提升，在当前范围内任何地方都可用。

1.  这是一种标准的检查方式，用于查看两个参数中是否只有一个被指定，即是否提供了一个数组但没有提供排序函数。如果没有，将提供一个默认函数。我们运行了几次排序的试验：

```js
console.log(sort([1, 3, 2, 11, 9]));
console.log(sort(['a', 'c', 'b']));
console.log(sort(['a', 'c', 'b', 1, 3, 2, 11, 9]); 
```

这给了我们：

```js
[1, 2, 3, 9, 11]
["a", "b", "c"]
["a", 1, 3, 2, 11, 9, "b", "c"] 
```

这给了我们一个调试的机会。现在，假设我们添加以下内容：

```js
console.log('Before: ' + before);
console.log('Same: ' + same);
console.log('After: ' + after);
```

在结果声明之前，我们得到：

```js
[1, 2, 3, 9, 11]
Before:
Same: a
After: c,b
Before: b
Same: c
After: 
Before:
Same: b
After:
["a", "b", "c"]
Before:
Same: a,1,3,2,11,9
After: c,b
Before: b
Same: c
After: 
Before: 
Same: b
After:  
["a", 1, 3, 2, 11, 9, "b", "c"]
```

输出中说 `Same: a,1,3,2,11,9` 看起来可疑，一个 `Same` 桶应该有相同的值，因此一个合适的输出可能是 `Same: 2,2,2` 或者只是 `Same: 3`，其中 `Same` 列表有五个值，没有两个是相同的。这不可能是我们想要的行为。看起来整数被分类为与初始的 "a" 相同，其余部分被排序。一点调查证实了 '`"a" < 1`' 和 '`"a" > 1`' 都是假的，所以我们的比较器可以改进。

我们对它们的类型进行了字典排序。这在类型的字母顺序上进行了一些任意排序，然后按照类型默认的排序顺序进行排序，这可以用另一个比较函数覆盖。这是另一种可能用于对数组进行排序的许多种比较器中的一个例子：与前一个不同，这个比较器将不同种类的项目进行分段，例如按顺序排序的数字将出现在字符串之前，按顺序排序：

```js
        var comparator = function(first, second) {
          if (typeof first < typeof second) {
            return -1;
          } else if (typeof second < typeof first) {
            return -1;
          } else if (first < second) {
            return -1;
          } else if (second < first) {
            return 1;
          } else {
            return 0;
          }
        }
```

`typeof` 函数返回一个包含类型描述的字符串；因此 `typeof` 可以提供一个扩展类型。使用类似于这样的比较函数，可以有意义地比较诸如包含名字、姓氏和地址的记录之类的对象。

对象可以通过花括号表示法声明。代码块和对象都可以使用花括号，但这是两个不同的东西。下面的代码及其花括号，不是一个带有要执行的语句的代码块；它定义了一个具有键和值的字典：

```js
var sample = {
  'a': 12,
  'b': 2.5
};
```

除非键是保留字或包含特殊字符，如 `'strange.key'`（这里是一个句号），否则键周围的引号是可选的。为了有一个简单和一致的规则，JSON 格式要求在所有情况下都使用引号，特别是双引号，而不是单引号。

下面显示了一个记录具有名字、姓氏和电子邮件地址的示例，可能是通过 JSON 填充的。这个示例不是 JSON，因为它没有遵循 JSON 关于双引号引用所有键和所有字符串的规则，但它说明了一个记录数组，其中可能有其他字段，可能会更长。按距离或资历排序可能是有意义的（这里没有显示填充字段）：可能有一整套可能用于记录的比较器。

```js
var records = [{
    first_name: 'Alice',
    last_name: 'Spung',
    email: 'a.spung@yahoo.com'
  }, {
    first_name: 'Robert',
    last_name: 'Hendrickson',
    email: 'Bob.Hendrickson@gmail.com'
  }
];
```

请注意，尾随逗号不仅在 JavaScript 中是不合适的（在几乎任何由逗号分隔的东西的最后一个条目之后），而且它有一些奇怪和意想不到的行为，这可能极其难以调试。

JavaScript 被设计为在语句的末尾有分号，这可能是可选的。这是一个有争议的决定，与决定制作一种流行的语言，普通非程序员可以在不涉及 IT 人员的情况下使用有关，这也是 SQL 设计中涉及的因素。当适当时，我们应该始终提供分号。这样做的一个副作用是，单独一行的`return`将返回 undefined。这意味着以下代码将不会产生预期的效果，并且将返回 undefined：

```js
return
  {
  x: 12
  };
```

### 提示

代码执行时的效果与它看起来的效果以及可能的意图不同，因此最好不要这样做。

为了获得期望的效果，开放的大括号应该与 return 语句在同一行，如下所示：

```js
return {
  x: 12
};
```

然而，JavaScript 确实具有面向对象编程，避免了面向对象编程中的一个经典困难：必须第一次就正确地获取本体论。对象通常最好是通过工厂而不是类的实例来构造。或者道格拉斯·克罗克福德已经被缩写。原型仍然是语言的一部分，就像许多好的和坏的特性一样，但是除了涉及奇特的用例，对象通常应该由允许“比本体论驱动的类”更好的面向对象编程方法的工厂制造。我们将避免伪经典的新`function()`，不是因为如果你忘记了新的话它可能会破坏全局变量，而是因为它传统面向对象编程的外观并没有真正帮助太多。

### 提示

你应该知道 JavaScript 中一个广受尊敬的惯例，即打算与`new`一起使用的构造函数以大写字母开头。如果函数名以大写字母开头，那么它打算与`new`关键字一起使用，如果在没有`new`关键字的情况下调用，可能会发生奇怪的事情。

在 JavaScript 中，一些其他语言中经典面向对象编程所服务的利益有时最好通过函数式编程来推进。

## 循环

循环包括`for`循环，`for in`循环，`while-do`循环和`do-while`循环。`for`循环的工作方式与 C 语言相同：

```js
var numbers = [1, 2, 3];
var total = 0;
for(var index = 0; index < numbers.length; ++index) {
  total += numbers[index];
}
```

`for in`循环将循环遍历对象中的所有内容。`hasOwnProperty()`方法可用于仅检查对象的字段。对于名为`obj`的对象，两个变体如下：

```js
var counter = 0;
for(var field in obj) {
  counter += 1;
}
```

这将包括原型链中的任何字段（此处未解释）。为了检查对象本身的直接属性，而不是原型链中潜在的嘈杂数据，我们使用对象的`hasOwnProperty()`方法：

```js
var counter = 0;
for(var field in obj) {
  if (obj.hasOwnProperty(field)) {
    counter += 1;
  }
}
```

顺序不被保证；如果你正在寻找任何特定的字段，值得考虑的是只迭代你想要的字段并在对象上检查它们。

## 看一下 ECMAScript 6

JavaScript 工具一直在蓬勃发展，一个工具被另一个工具取代，有一个极其丰富的生态系统，很少有开发人员可以自豪地广泛和深入地了解。然而，核心语言 ECMAScript 或 JavaScript 已经稳定了好几年。

ECMAScript 6，有一个介绍性的路线图可在[`tinyurl.com/reactjs-ecmascript-6`](http://tinyurl.com/reactjs-ecmascript-6)上找到，它为核心语言引入了深刻的新变化和扩展。一般来说，这些功能增强、加深或使 JavaScript 的功能方面更加一致。可以建议 ECMAScript 6 的功能不做这种工作，比如增强的面向类的语法糖，让 Java 程序员假装 JavaScript 意味着在 Java 中编程，可能会被忽略。

ECMAScript 6 的功能是不可忽视的，在撰写本文时，它们已经开始在主流浏览器中广泛应用。如果你想扩展和提高自己作为 JavaScript 开发人员的价值，不要局限于深入挖掘丰富的 JavaScript 生态系统，无论那有多重要。学习新的 JavaScript。学习一个有更多更好部分的 ECMAScript。

# 总结

在这场对 JavaScript 一些更好部分的风暴式之旅中，我们涵盖了可以在我们进一步推进 JavaScript 时有所帮助的基础构建模块。这些包括变量和赋值、注释、流程控制、值、NaN 函数和 ECMAScript 6。

在*变量和赋值*部分，我们揭示了大多数编程的一些基本构建模块，尽管函数式响应式编程的重点可能在其他地方。在*注释*部分，我们了解到注释在任何地方都是相同的，但这里的主要关注点是防止奇怪的意外。

*流程控制*部分涵盖了函数内的基本流程控制（或者可能在任何函数之外，尽管通常应该避免这样做）。

在*关于值和 NaN 的说明*部分，我们讨论了类似于 Perl，JavaScript 认为真理是不言自明的；也就是说，如果它们为 null、零、空、非数字等，则这些东西是虚假的，而不在列表上的任何东西都是真的。

在*函数*部分，我们看了一些包含有些复杂示例的函数。在*ECMAScript 6*部分，我们讨论了核心 JavaScript 语言的扩展。

这只是对亮点的简要介绍，而不是全面的介绍。如果你需要更全面的 JavaScript 基础，有多种选择可供选择。我们将在下一章继续讨论响应式编程的基本理论。
