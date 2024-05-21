# 第四章：BuckleScript，Belt 和互操作性

在本章中，我们将更仔细地了解 BuckleScript 特定的功能。我们还将学习递归和递归数据结构。到本章结束时，我们将在 Reason 及其生态系统的介绍中完成一个完整的循环。在这样做的过程中，我们将完成以下工作：

+   更多了解 Reason 的模块系统

+   探索了 Reason 的原始数据结构（数组和列表）

+   看到各种管道运算符如何使代码更易读

+   熟悉 Reason 和 Belt 标准库

+   为在 Reason 中使用而创建了对 JavaScript 模块的绑定

+   通过绑定到 React Transition Group 组件为我们的应用程序添加路由转换

要跟进，请使用您希望的任何环境。我们将要做的大部分工作与 ReasonReact 无关。在本章末尾，我们将继续构建我们的 ReasonReact 应用程序。

# 模块范围

正如您现在所知，所有`.re`文件都是模块，所有模块都是全局可用的，包括嵌套的模块。默认情况下，可以通过提供命名空间从任何地方访问所有类型和绑定。然而，一遍又一遍地这样做很快变得乏味。幸运的是，我们有几种方法可以使这更加愉快：

```js
/* Foo.re */
type fromFoo =
  | Add(int, int)
  | Multiply(int, int);

let a = 1;
let b = 2;
```

接下来，我们将以不同的方式在另一个模块中使用`Foo`模块的`fromFoo`类型以及它的绑定：

+   **选项 1**：不使用任何语法糖：

```js
/* Bar.re */
let fromFoo = Foo.Add(Foo.a, Foo.b);
```

+   **选项 2**：将模块别名为更短的名称。例如，我们可以声明一个新模块`F`并将其绑定到现有模块`Foo`：

```js
/* Bar.re */
module F = Foo;
let fromFoo = F.Add(F.a, F.b);
```

+   **选项 3**：使用`Module.()`语法在本地打开模块。此语法仅适用于单个表达式：

```js
/* Bar.re */
let fromFoo = Foo.(Add(a, b));
```

+   **选项 4**：在面向对象编程意义上，使用`include`使`Bar`扩展`Foo`：

```js
/* Bar.re */
include Foo;
let a = 4; /* override Foo.a */
let fromFoo = Add(a, b);
```

+   **选项 5**：全局`open`模块。在大范围内谨慎使用`open`，因为很难知道哪些类型和绑定属于哪些模块：

```js
/* Bar.re */
open Foo;
let fromFoo = Add(a, b);
```

在本地范围内最好使用`open`：

```js
/* Bar.re */
let fromFoo = {
  open Foo;
  Add(a, b);
};
```

前面的语法将通过`refmt`重新格式化为选项 3 的语法，但请记住，选项 3 的语法仅适用于单个表达式。例如，以下内容无法转换为选项 3 的语法：

```js
/* Bar.re */
let fromFoo = {
  open Foo;
  Js.log("foo");
  let result = Add(a, b);
};
```

Reason 标准库包含在我们已经可以使用的各种模块中。例如，Reason 的标准库包括一个`Array`模块，我们可以使用点表示法（即`Array.length`）访问其函数。

在第五章中，*Effective ML*，我们将学习如何隐藏模块的类型和绑定，以便在不希望它们全局可用时不让它们全局可用。

# 数据结构

我们已经看到了 Reason 的几种原始数据结构，包括字符串、整数、浮点数、元组、记录和变体。让我们再探索一些。

# 数组

Reason 数组编译为常规的 JavaScript 数组。Reason 数组如下：

+   同种（所有元素必须是相同类型）

+   可变的

+   快速随机访问和更新

它们看起来像这样：

```js
let array = [|"first", "second", "third"|];
```

访问和更新数组的元素与 JavaScript 中的操作相同：

```js
array[0] = "updated";
```

在 JavaScript 中，我们对数组进行映射，如下所示：

```js
/* JavaScript */
array.map(e => e + "-mapped")
```

在 Reason 中执行相同操作时，我们有几种不同的选择。

# 使用 Reason 标准库

Reason 标准库的`Array`模块包含几个函数，但并非您从 JavaScript 中期望的所有函数。但它确实有一个`map`函数：

```js
/* Reason standard library */
let array = [|"first", "second", "third"|];
Array.map(e => e ++ "-mapped", array);
```

`Array.map`的类型如下：

```js
('a => 'b, array('a)) => array('b);
```

类型签名表示`map`接受类型为`'a => 'b`的函数，类型为`'a`的数组，并返回类型为`'b`的数组。请注意，`'a`和`'b`是**类型变量**。类型变量类似于普通变量，只不过是类型。在前面的示例中，`map`的类型为：

```js
(string => string, array(string)) => array(string);
```

这是因为`'a`和`'b`类型变量都被一致地替换为具体的`string`类型。

请注意，使用`Array.map`时，编译输出不会编译为 JavaScript 的`Array.prototype.map`——它有自己的实现：

```js
/* in the compiled output */
...
require("./stdlib/array.js");
...
```

Reason 标准库文档可以在这里找到：

[`reasonml.github.io/api`](https://reasonml.github.io/api)

# 使用 Belt 标准库

Reason 标准库实际上是 OCaml 标准库。它并不是为 JavaScript 而创建的。Belt 标准库是由创建 BuckleScript 的同一个人——张宏波——创建的，并且随 BuckleScript 一起发布。Belt 是为 JavaScript 而创建的，尤其以其性能而闻名。Belt 标准库通过`Belt`模块访问：

```js
/* Belt standard library */
let array = [|"first", "second", "third"|];
Belt.Array.map(array, e => e ++ "-mapped");
```

Belt 标准库文档可以在这里找到：

[`bucklescript.github.io/bucklescript/api/Belt.html`](https://bucklescript.github.io/bucklescript/api/Belt.html)

# 使用 BuckleScript 内置的 JavaScript 绑定

另一个很好的选择是使用 BuckleScript 内置的 JavaScript 绑定，可以在`Js`模块中找到：

```js
/* BuckleScript's JavaScript bindings */
let array = [|"first", "second", "third"|];
Js.Array.map(e => e ++ "-mapped", array);
```

这个选项的优势是在编译输出中不需要任何依赖项。它还具有非常熟悉的 API。但是，由于并非所有 Reason 数据结构都存在于 JavaScript 中，您可能会使用标准库。如果是这样，请优先选择 Belt。

BuckleScript 的绑定文档可以在这里找到：

[`bucklescript.github.io/bucklescript/api/Js.html`](https://bucklescript.github.io/bucklescript/api/Js.html)

# 使用自定义绑定

你可以自己编写自定义绑定：

```js
[@bs.send] external map: (array('a), 'a => 'b) => array('b) = "";
let array = [|"first", "second", "third"|];
map(array, e => e ++ "-mapped")
```

当然，你应该更倾向于使用`Js`模块中的内置绑定。我们将在本章后面探讨更多自定义绑定。

# 使用原始 JavaScript

最后的选择是在 Reason 中使用实际的 JavaScript：

```js
let array = [|"first", "second", "third"|];
let map = [%raw {|
  function(f, array) {
    return array.map(f)
  }
|}];
map(e => e ++ "-mapped", array)
```

BuckleScript 让我们以原始 JavaScript 的方式保持高效学习。当然，这样做时，我们放弃了 Reason 提供的安全性。因此，一旦准备好，将任何原始 JavaScript 代码转换回更符合惯例的 Reason。

在使用原始 JavaScript 时，对于表达式使用`%`，对于语句使用`%%`。记住，`{| |}`是 Reason 的多行字符串语法：

```js
let array = [%raw "['first', 'second', 'third']"];
[%%raw {|
  array = array.map(e => e + "-mapped");
|}];
```

使用原始表达式语法，我们还可以注释类型：

```js
let array: array(string) = [%raw "['first', 'second', 'third']"];
```

我们甚至可以注释函数类型：

```js
let random: unit => float = [%raw
  {|
    function() {
     return Math.random();
    }
  |}
];
```

尽管从 JavaScript 中来时数组很熟悉，但您可能会发现自己使用列表，因为它们在函数式编程中是无处不在的。列表既是**不可变的**又是**递归的**。现在让我们看看如何使用这种递归数据结构。

# 列表

Reason 列表如下：

+   同质的

+   不可变的

+   在列表的头部快速添加和访问

它们看起来像这样：

```js
let list = ["first", "second", "third"];
```

列表的头，在这种情况下，是`"first"`。到目前为止，我们已经看到使用不可变数据结构并不困难。我们不是进行突变，而是创建更新后的副本。

在处理列表时，我们不能直接使用 JavaScript 绑定，因为列表在 JavaScript 中并不作为原始数据结构存在。但是，我们可以将列表转换为数组，反之亦然：

```js
/* Belt standard library */
let list = ["first", "second", "third"];
let array = Belt.List.toArray(list);

let array = [|"first", "second", "third"|];
let list = Belt.List.fromArray(array);

/* Reason standard library */
let list = ["first", "second", "third"];
let array = Array.of_list(list);

let array = [|"first", "second", "third"|];
let list = Array.to_list(array);
```

但我们也可以直接映射列表：

```js
/* Belt standard library */
let list = ["first", "second", "third"];
Belt.List.map(list, e => e ++ "-mapped");

/* Reason standard library */
let list = ["first", "second", "third"];
List.map(e => e ++ "-mapped", list);
```

将`list`记录到控制台显示，列表在 JavaScript 中表示为嵌套数组，其中每个数组始终有两个元素：

```js
["first", ["second", ["third", 0]]]
```

在理解列表是一个递归数据结构之后，这是有意义的。Reason 列表是**单向链表**。列表中的每个元素要么是**空**（在 JavaScript 中表示为`0`），要么是值和另一个列表的**组合**。

`list`的示例类型定义显示`list`是一个变体：

```js
type list('a) = Empty | Head('a, list('a));
```

注意：类型定义可以是递归的。

Reason 提供了一些语法糖，简化了更冗长的版本：

```js
Head("first", Head("second", Head("third", Empty)));
```

# 递归

由于列表是一个递归数据结构，我们通常在处理它时使用递归。

为了热身，让我们编写一个（天真的）函数，对整数列表求和：

```js
let rec sum = list => switch(list) {
  | [] => 0
  | [hd, ...tl] => hd + sum(tl)
};
```

+   这是一个递归函数，因此需要`rec`关键字（即`let rec`而不仅仅是`let`）

+   我们可以对列表进行模式匹配（就像任何其他变体和许多其他数据结构一样）

+   从示例类型定义中，`Empty`表示为`[]`，`Head`表示为`[hd, ...tl]`，其中`hd`是列表的**头部**，`tl`是剩余部分（即列表的**尾部**）

+   `tl`可能是`[]`（即`Empty`），当它是这样时，递归停止

传入`sum`函数的列表`[1, 2, 3]`会产生以下步骤：

```js
sum([1, 2, 3])
1 + sum([2, 3])
1 + 2 + sum([3])
1 + 2 + 3
6
```

让我们通过分析另一个（朴素的）反转列表的函数，更加熟悉列表和递归：

```js
let rec reverse = list => switch(list) {
  | [] => []
  | [hd, ...tl] => reverse(tl) @ [hd]
};
```

+   同样，我们使用`rec`来定义一个递归函数

+   同样，我们在列表上使用模式匹配——如果它为空，则停止递归；否则，继续使用较小的列表

+   `@`操作符将第二个列表附加到第一个列表的末尾

传入先前定义的列表(`["first", "second", "third"]`)会产生以下步骤：

```js
reverse(["first", "second", "third"])
reverse(["second", "third"]) @ ["first"]
reverse(["third"]) @ ["second"] @ ["first"]
reverse([]) @ ["third"] @ ["second"] @ ["first"]
[] @ ["third"] @ ["second"] @ ["first"]
["third", "second", "first"]
```

这个 reverse 的实现方法有两个问题：

+   它不是尾调用优化的（我们的`sum`函数也不是）

+   它使用`append`（`@`），这比`prepend`慢

更好的实现方法是使用一个带有累加器的本地辅助函数：

```js
let reverse = list => {
  let rec aux = (list, acc) => switch(list) {
    | [] => acc
    | [hd, ...tl] => aux(tl, [hd, ...acc])
  };
  aux(list, []);
};
```

现在，它的尾调用已经优化，并且它使用 prepend 而不是 append。在 Reason 中，您可以使用`...`语法向列表前置：

```js
let list = ["first", "second", "third"];
let list = ["prepended", ...list];
```

传入列表(`["first", "second", "third"]`)大致会产生以下步骤：

```js
reverse(["first", "second", "third"])
aux(["first", "second", "third"], [])
aux(["second", "third"], ["first"])
aux(["third"], ["second", "first"])
aux([], ["third", "second", "first"])
["third", "second", "first"]
```

请注意，在非尾递归版本中，Reason 无法创建列表直到递归完成。在尾递归版本中，累加器（即`aux`的第二个参数）在每次迭代后更新。

尾递归（即尾调用优化）函数的好处在于能够重用当前的堆栈帧。因此，尾递归函数永远不会发生堆栈溢出，但非尾递归函数在足够的迭代次数后可能会发生堆栈溢出。

# 管道操作符

Reason 有两个管道操作符：

```js
|> (pipe)
-> (fast pipe)
```

两个管道操作符都将参数传递给函数。`|>`管道操作符将参数传递给函数的最后一个参数，而`->`快速管道操作符将参数传递给函数的第一个参数。

看一下这些：

```js
three |> f(one, two)
one -> f(two, three)
```

它们等价于这个：

```js
f(one, two, three)
```

如果函数只接受一个参数，那么两个管道的工作方式是相同的，因为函数的第一个参数也是函数的最后一个参数。

使用这些管道操作符非常流行，因为一旦你掌握了它，代码会变得更加可读。

我们不需要使用这个：

```js
Belt.List.(reduce(map([1, 2, 3], e => e + 1), 0, (+)))
```

我们可以以一种不需要读者从内到外阅读的方式来编写它：

```js
Belt.List.(
 [1, 2, 3]
 ->map(e => e + 1)
 ->reduce(0, (+))
);
```

正如你所看到的，使用快速管道看起来类似于 JavaScript 中的链式调用。与 JavaScript 不同的是，我们可以传递`+`函数，因为它只是一个接受两个参数并将它们相加的普通函数。括号是必要的，告诉 Reason 将中缀操作符`（+）`视为标识符。

# 使用 Belt

让我们利用本章学到的知识来编写一个小程序，创建一副牌，洗牌，并从牌堆顶部抽取五张牌。为此，我们将使用 Belt 的`Option`和`List`模块，以及快速管道操作符。

# Option 模块

Belt 的`Option`模块是用于处理`option`类型的实用函数集合。例如，要解包一个选项，并在选项的值为`None`时抛出运行时异常，我们可以使用`getExn`：

```js
let foo = Some(3)->Belt.Option.getExn;
Js.log(foo); /* 3 */

let foo = None->Belt.Option.getExn;
Js.log(foo); /* raises getExn exception */
```

能够抛出运行时异常的 Belt 函数总是带有`Exn`后缀。

另一个解包选项的替代函数是`getWithDefault`，它不能抛出运行时异常：

```js
let foo = None->Belt.Option.getWithDefault(0);
Js.log(foo); /* 0 */
```

`Option`模块还提供了其他几个函数，如`isSome`、`isNone`、`map`、`mapWithDefault`等。查看文档以获取详细信息。

Belt Option 模块的文档可以在这里找到：

[`bucklescript.github.io/bucklescript/api/Belt.Option.html`](https://bucklescript.github.io/bucklescript/api/Belt.Option.html)

# List 模块

List 模块是用于列表数据类型的实用程序。要查看 Belt 提供的用于处理列表的函数，请检查 Belt 的`List`模块文档。

Belt List 模块的文档可以在这里找到：

[`bucklescript.github.io/bucklescript/api/Belt.List.html`](https://bucklescript.github.io/bucklescript/api/Belt.List.html)

让我们专注于其中的一些。

# make

`make` 函数用于创建一个填充列表。它接受一个整数作为列表的长度，以及列表中每个项目的值。它的类型如下：

```js
(int, 'a) => Belt.List.t('a)
```

`Belt.List.t` 被公开为 `list` 类型的别名，因此我们可以说 `Belt.List.make` 的类型如下：

```js
(int, 'a) => list('a)
```

我们可以用它来创建一个包含十个字符串的列表，就像这样：

```js
let list = Belt.List.make(10, "string");
```

在第五章 *Effective ML* 中，我们将学习如何显式地从模块中公开或隐藏类型和绑定。

# makeBy

`makeBy` 函数类似于 `make` 函数，但它接受一个用于确定每个项目的值的函数，给定项目的索引。

`makeBy` 函数的类型如下：

```js
(int, int => 'a) => Belt.List.t('a)
```

我们可以用它来创建一个包含十个项目的列表，其中每个项目都等于它的索引：

```js
let list = Belt.List.makeBy(10, i => i);
```

# shuffle

`shuffle` 函数会随机洗牌一个列表。它的类型是：

```js
Belt.List.t('a) => Belt.List.t('a)
```

它接受一个列表并返回一个新列表。让我们用它来洗牌我们的整数列表：

```js
let list = Belt.List.(makeBy(10, i => i)->shuffle);
```

# take

`take` 函数接受一个列表和一个长度，并返回从列表头部开始的长度等于请求长度的子集。由于子集的请求长度可能超过原始列表的长度，结果被包装在一个选项中。它的类型如下：

```js
(Belt.List.t('a), int) => option(Belt.List.t('a))
```

我们可以从洗牌后的列表中取出前两个项目，就像这样：

```js
let list = Belt.List.(makeBy(10, i => i)->shuffle->take(2));
```

# 卡牌组示例

现在，我们准备将这个与我们从前几章学到的内容结合起来。你会如何编写一个创建一副卡牌、洗牌并抽取前五张卡的程序？在看下面的例子之前，自己试一试。

```js
type suit =
  | Hearts
  | Diamonds
  | Spades
  | Clubs;

type card = {
  suit,
  rank: int,
};

Belt.List.(
  makeBy(52, i =>
    switch (i / 13, i mod 13) {
    | (0, rank) => {suit: Hearts, rank: rank + 1}
    | (1, rank) => {suit: Diamonds, rank: rank + 1}
    | (2, rank) => {suit: Spades, rank: rank + 1}
    | (3, rank) => {suit: Clubs, rank: rank + 1}
    | _ => assert(false)
    }
  )
  ->shuffle
  ->take(5)
  ->Belt.Option.getExn
  ->(
      cards => {
        let rankToString = rank =>
          switch (rank) {
          | 1 => "Ace"
          | 13 => "King"
          | 12 => "Queen"
          | 11 => "Jack"
          | rank => string_of_int(rank)
          };

        let suitToString = suit =>
          switch (suit) {
          | Hearts => "Hearts"
          | Diamonds => "Diamonds"
          | Spades => "Spades"
          | Clubs => "Clubs"
          };

        map(cards, ({rank, suit}) =>
          rankToString(rank) ++ " of " ++ suitToString(suit)
        );
      }
    )
  ->toArray
  ->Js.log
);
```

这会以字符串格式随机产生五张卡牌的数组：

```js
[
  "Queen of Clubs",
  "4 of Clubs",
  "King of Spades",
  "Ace of Hearts",
  "9 of Spades"
]
```

# 柯里化

Belt 标准库的一些函数带有 *U* 后缀，比如这个：

```js
Belt.List.makeBy
```

你可以在这里看到后缀：

```js
Belt.List.makeByU
```

*U* 后缀代表 *uncurried*。在继续之前，让我们定义一下柯里化。

在 Reason 中，每个函数都只接受一个参数。这似乎与我们之前的许多例子相矛盾：

```js
let add = (a, b) => a + b;
```

前述的 `add` 函数看起来好像接受两个参数，但实际上只是以下的语法糖：

```js
let add = a => b => a + b;
```

`add` 函数接受一个参数 `a`，返回一个接受一个参数 `b` 的函数，然后返回 `a + b` 的结果。

在 Reason 中，两个版本都是有效的，并且具有相同的编译输出。在 JavaScript 中，前述两个版本都是有效的，但它们并不相同；它们需要以不同的方式使用才能获得相同的结果。第二个需要这样调用：

```js
add(2)(3);
```

这是因为 `add` 返回一个需要再次调用的函数，因此有两组括号。Reason 可以接受任何一种用法：

```js
add(2, 3);
add(2)(3);
```

柯里化的好处在于它使得组合函数更容易。你可以轻松地创建一个部分应用的函数 `addOne`：

```js
let addOne = add(1);
```

然后可以将这个 `addOne` 函数传递给其他函数，比如 `map`。也许你想使用这个功能将一个函数部分应用到 ReasonReact 子组件，而父组件的 `self` 部分应用。

令人困惑的是，`add` 的任一版本的编译输出如下：

```js
function add(a, b) {
  return a + b | 0;
}
```

中间函数在哪里？在可能的情况下，BuckleScript 优化编译输出，以避免不必要的函数分配，从而提高性能。

请记住，由于 Reason 的中缀运算符只是普通函数，我们可以这样做：

```js
let addOne = (+)(1);
```

# 柯里化的函数

由于 JavaScript 的动态特性，BuckleScript 不能总是优化编译输出以删除中间函数。但是，你可以告诉 BuckleScript 使用以下语法对函数进行 uncurry：

```js
let add = (. a, b) => a + b;
```

uncurry 语法是参数列表中的点。它需要在声明和调用站点都存在：

```js
let result = add(. 2, 3); /* 5 */
```

如果调用站点没有使用 uncurry 语法，BuckleScript 将抛出编译时错误：

```js
let result = add(2, 3);

We've found a bug for you!

This is an uncurried BuckleScript function. It must be applied with a dot.

Like this: foo(. a, b)
Not like this: foo(a, b)
```

此外，如果在调用站点缺少某些函数的参数，则会抛出编译时错误：

```js
let result = add(. 2);

We've found a bug for you!

Found uncurried application [@bs] with arity 2, where arity 1 was expected.
```

术语`arity`指的是函数接受的参数数量。

# makeByU

如果我们取消`makeBy`的第二个参数的柯里化，可以用`makeByU`替换它。这将提高性能（在我们的示例中可以忽略不计）：

```js
...
makeByU(52, (. i) =>
  switch (i / 13, i mod 13) {
  | (0, rank) => {suit: Hearts, rank: rank + 1}
  | (1, rank) => {suit: Diamonds, rank: rank + 1}
  | (2, rank) => {suit: Spades, rank: rank + 1}
  | (3, rank) => {suit: Clubs, rank: rank + 1}
  | _ => assert(false)
  }
)
...
```

点语法需要在`i`周围加括号。

# JavaScript 互操作性

术语**互操作性**指的是 Reason 程序在 Reason 中使用现有 JavaScript 的能力。BuckleScript 提供了一个出色的系统，用于在 Reason 中使用现有的 JavaScript 代码，并且还可以轻松地在 JavaScript 中使用 Reason 代码。

# 在 Reason 中使用 JavaScript

我们已经看到了如何在 Reason 中使用原始 JavaScript。现在让我们专注于如何绑定到现有的 JavaScript。要将值绑定到命名引用，通常使用`let`。然后可以在后续代码中使用该绑定。当我们要绑定的值位于 JavaScript 中时，我们使用`external`。`external`绑定类似于`let`，因为它可以在后续代码中使用。与`let`不同，`external`通常伴有 BuckleScript 装饰器，如`[@bs.val]`。

# 理解[@bs.val]装饰器

我们可以使用`[@bs.val]`绑定全局值和函数。一般来说，语法如下：

```js
[@bs.val] external alert: string => unit = "alert";
```

+   BuckleScript 的一个或多个装饰器（即`[@bs.val]`）

+   `external`关键字

+   绑定的命名引用

+   类型声明

+   等号

+   一个字符串

`external`关键字将`alert`绑定到类型为`string => unit`的值，并绑定到字符串`alert`。字符串`alert`是上述外部声明的值，也是编译输出中要使用的值。当外部绑定的名称等于其字符串值时，字符串可以留空：

```js
[@bs.val] external alert: string => unit = "";
```

使用绑定就像使用任何其他绑定一样：

```js
alert("hi!");
```

# 理解[@bs.scope]装饰器

要绑定到`window.location.pathname`，我们使用`[@bs.scope]`添加一个作用域。这为`[@bs.val]`定义了作用域。例如，如果要绑定到`window.location`的`pathname`属性，可以指定作用域为`[@bs.scope ("window", "location")]`：

```js
[@bs.val] [@bs.scope ("window", "location")] external pathname: string = "";
```

或者，我们可以只使用`[@bs.val]`在字符串中包含作用域：

```js
[@bs.val] external pathname: string = "window.location.pathname";
```

# 理解[@bs.send]装饰器

`[@bs.send]`装饰器用于绑定对象的方法和属性。使用`[@bs.send]`时，第一个参数始终是对象。如果有剩余的参数，它们将被应用于对象的方法：

```js
[@bs.val] external document: Dom.document = "";
[@bs.send] external getElementById: (Dom.document, string) => Dom.element = "";
let element = getElementById(document, "root");
```

`Dom`模块也由 BuckleScript 提供，并为 DOM 提供类型声明。

Dom 模块文档可以在这里找到：

[`bucklescript.github.io/bucklescript/api/Dom.html`](https://bucklescript.github.io/bucklescript/api/Dom.html)

还有一个用于 Node.js 的 Node 模块：

[`bucklescript.github.io/bucklescript/api/Node.html`](https://bucklescript.github.io/bucklescript/api/Node.html)

在编写外部声明时要小心，因为您可能会意外地欺骗类型系统，这可能导致运行时类型错误。例如，我们告诉 Reason 我们的`getElementById`绑定总是返回`Dom.element`，但是当 DOM 找不到提供的 ID 的元素时，它返回`undefined`。更正确的绑定应该是这样的：

```js
[@bs.send] external getElementById: (Dom.document, string) => option(Dom.element) = "";
```

# 理解[@bs.module]装饰器

要导入一个节点模块，使用`[@bs.module]`。编译输出取决于`bsconfig.json`中使用的`package-specs`配置。我们使用`es6`作为模块格式。

```js
[@bs.module] external leftPad: (string, int) => string = "left-pad";
let result = leftPad("foo", 6);
```

这编译成以下内容：

```js
import * as LeftPad from "left-pad";

var result = LeftPad("foo", 6);

export {
  result ,
}
```

将模块格式设置为`commonjs`会产生以下编译输出：

```js
var LeftPad = require("left-pad");

var result = LeftPad("foo", 6);

exports.result = result;
```

当`[@bs.module]`没有字符串参数时，默认值被导入。

# 合理的 API

在绑定到现有的 JavaScript API 时，考虑一下你想在 Reason 中如何使用 API。即使是依赖于 JavaScript 动态类型的现有 JavaScript API 也可以在 Reason 中使用。BuckleScript 利用了高级类型系统技术，让我们能够利用 Reason 的类型系统来使用这样的 API。

从 BuckleScript 文档中，看一下以下 JavaScript 函数：

```js
function padLeft(value, padding) {
  if (typeof padding === "number") {
    return Array(padding + 1).join(" ") + value;
  }
  if (typeof padding === "string") {
    return padding + value;
  }
  throw new Error(`Expected string or number, got '${padding}'.`);
}
```

如果我们要在 Reason 中绑定到这个函数，最好使用`padding`作为一个变体。这是我们将如何做到这一点：

```js
[@bs.val]
external padLeft: (
  string,
  [@bs.unwrap] [
    | `Str(string)
    | `Int(int)
  ])
  => string = "";

padLeft("Hello World", `Int(4));
padLeft("Hello World", `Str("Message: "));
```

这编译成了以下内容：

```js
padLeft("Hello World", 4);
padLeft("Hello World", "Message: ");
```

`padLeft`的类型是`(string, some_variant) => string`，其中`some_variant`使用了一个称为**多态变体**的高级类型系统特性，它使用`[@bs.unwrap]`来转换为 JavaScript 可以理解的内容。我们将在第五章中了解更多关于多态变体的知识，*Effective ML*。

# BuckleScript 文档

虽然这只是一个简短的介绍，但你可以看到 BuckleScript 有很多工具可以帮助我们与惯用的 JavaScript 进行交流。我强烈建议你阅读 BuckleScript 文档，以了解更多关于 JavaScript 互操作性的知识。

BuckleScript 文档可以在这里找到：

[`bucklescript.github.io/docs/interop-overview`](https://bucklescript.github.io/docs/interop-overview)

# 绑定到现有的 ReactJS 组件

ReactJS 组件不是 Reason 组件。要使用现有的 ReactJS 组件，我们使用`[@bs.module]`来导入节点模块，然后使用`ReasonReact.wrapJsForReason`辅助函数将 ReactJS 组件转换为 Reason 组件。还有一个`ReasonReact.wrapReasonForJs`辅助函数用于在 ReactJS 中使用 Reason。

让我们从第三章离开的地方继续构建我们的应用程序，*创建 ReasonReact 组件*：

```js
git clone https://github.com/PacktPublishing/ReasonML-Quick-Start-Guide.git
cd ReasonML-Quick-Start-Guide
cd Chapter03/app-end
npm install
```

在这里，我们通过绑定到现有的 React Transition Group 组件来添加路由转换：

React Transition Group 文档可以在这里找到：

[`reactcommunity.org/react-transition-group/`](https://reactcommunity.org/react-transition-group/)

# 导入依赖项

运行`npm install --save react-transition-group`来安装依赖。

让我们创建一个名为`ReactTransitionGroup.re`的新文件来存放这些绑定。在这个文件中，我们将绑定到`TransitionGroup`和`CSSTransition`组件：

```js
[@bs.module "react-transition-group"]
external transitionGroup: ReasonReact.reactClass = "TransitionGroup";

[@bs.module "react-transition-group"]
external cssTransition: ReasonReact.reactClass = "CSSTransition";
```

# 创建 make 函数

接下来，我们创建组件所需的`make`函数。这是我们使用`ReasonReact.wrapJsForReason`辅助函数的地方。

对于`TransitionGroup`，我们不需要任何 props。由于`~props`参数是必需的，我们传递`Js.Obj.empty()`。`~reactClass`参数传递了我们在上一步中创建的外部绑定：

```js
module TransitionGroup = {
  let make = children =>
    ReasonReact.wrapJsForReason(
      ~reactClass=transitionGroup,
      ~props=Js.Obj.empty(),
      children,
    );
};
```

现在，`ReactTransitionGroup.TransitionGroup`是一个可以在我们的应用程序中使用的 ReasonReact 组件。

# 使用[@bs.deriving abstract]

`CSSTransitionGroup`将需要以下 props：

+   `_in`

+   `timeout`

+   `classNames`

由于`in`是 Reason 中的保留字，惯例是在 Reason 中使用`_in`，并让 BuckleScript 将其编译为 JavaScript 中的`in`，使用`[@bs.as "in"]`。

BuckleScript 提供了`[@bs.deriving abstract]`，可以轻松地处理某些类型的 JavaScript 对象。我们可以直接使用 BuckleScript 创建对象，而不是在 JavaScript 中创建对象并绑定到该对象：

```js
[@bs.deriving abstract]
type cssTransitionProps = {
  [@bs.as "in"] _in: bool,
  timeout: int,
  classNames: string,
};
```

注意：`cssTransitionProps`不是一个记录类型，它只是看起来像一个。

当使用`[@bs.deriving abstract]`时，会自动提供一个辅助函数来创建具有该形状的 JavaScript 对象。这个辅助函数也被命名为`cssTransitionProps`。我们在组件的`make`函数中使用这个辅助函数来创建组件的 props：

```js
module CSSTransition = {
  let make = (~_in: bool, ~timeout: int, ~classNames: string, children) =>
    ReasonReact.wrapJsForReason(
      ~reactClass=cssTransition,
      ~props=cssTransitionProps(~_in, ~timeout, ~classNames),
      children,
    );
};
```

# 使用组件

现在，在`App.re`中，我们可以改变渲染函数来使用这些组件。我们将改变这个：

```js
<main> {currentRoute.component} </main>
```

现在它看起来是这样的：

```js
<main>
  ReactTransitionGroup.(
    <TransitionGroup>
      <CSSTransition
        key={currentRoute.title} _in=true timeout=900 classNames="routeTransition">
        {currentRoute.component}
      </CSSTransition>
    </TransitionGroup>
  )
</main>
```

注意：key 属性是一个特殊的 ReactJS 属性，不应该是组件 props 参数的一部分在`ReasonReact.wrapJsForReason`中。对于特殊的 ReactJS ref 属性也是如此。

为了完整起见，以下是相应的 CSS，可以在`ReactTransitionGroup.scss`中找到：

```js
@keyframes enter {
  from {
    opacity: 0;
    transform: translateY(50px);
  }
}

@keyframes exit {
  to {
    opacity: 0;
    transform: translateY(50px);
  }
}

.routeTransition-enter.routeTransition-enter-active {
  animation: enter 500ms ease 400ms both;
}

.routeTransition-exit.routeTransition-exit-active {
  animation: exit 400ms ease both;
}
```

请确保在`ReactTransitionGroup.re`中要求前述内容：

```js
/* ReactTransitionGroup.re */
[@bs.val] external require: string => string = "";
require("../../../src/ReactTransitionGroup.scss");
```

现在，当改变路由时，旧路由的内容会向下动画并淡出，然后新路由的内容会向上动画并淡入。

# 摘要

BuckleScript 非常强大，因为它让我们以一种非常愉快的方式与惯用的 JavaScript 进行交互。它还提供了 Belt 标准库，这是为 JavaScript 而创建的。我们学习了数组和列表，看到了在 Reason 中如何轻松地使用现有的 ReactJS 组件。

在第五章 *Effective ML*中，我们将学习如何使用模块签名来隐藏组件的实现细节，同时构建一个自动完成输入组件。我们将首先使用硬编码数据，然后在第六章 *CSS-in-JS (in Reason)*中，我们将把数据移到`localStorage`（客户端 Web 存储）。
