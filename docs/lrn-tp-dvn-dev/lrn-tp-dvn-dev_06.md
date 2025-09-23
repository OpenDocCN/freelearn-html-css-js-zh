# 第六章：创建可以插入任何其他类型的类型

在上一章中，我们看到了如何表达具有在运行时可能成为几种不同事物之一的潜在值的类型。在上一章的某些时候，以及到目前为止整本书中，我们遇到了 Reason 标记为*稍后填充*的类型。在本章中，我们将更详细地介绍这些类型，特别是以下主题：

+   Reason 的泛型类型推断

+   什么是类型参数？

+   常见的参数化类型，如列表、选项和数组

+   向求和和乘积类型添加参数

+   参数化可变类型的类型推断限制

# 类型推断和泛型类型

让我们看看 Reason 的类型推断的一些有趣示例以及它是如何决定需要稍后*填充*哪些类型的，如下面的代码片段所示：

```js
/* src/Ch06/Ch06_GenericInference.re */
let *triple*(*x*) = (*x*, *x*, *x*); /* (1) */
let *wrap*(*x*) = `*wrap*(*x*); /* (2) */
let *makeObj*(*x*) = {as _; pub *x* = *x*}; /* (3) */
let *greet*(*x*) = *print_endline*({j|Hello, $*x*!|j}); /* (4) */
```

这些例子都有一个共同点：编译器并没有足够的信息来推断它们的*具体*类型。相反，它推断出它们的一般形状，但将一些部分留为*泛型*。我们可以观察到以下内容：

1.  在（1）中，类型被推断为 `'a => ('a, 'a, 'a)'。

1.  在（2）中，类型被推断为 ``'a => [> `wrap('a)]``。

1.  在（3）中，类型被推断为`'a => {. x: 'a}`。

1.  在（4）中，类型被推断为 `'a => unit'。

在这些情况下，编译器推断出一些类型为 `'a'，换句话说，*我还不知道*。让我们看看第一个案例，`triple`函数，以尝试理解原因。

在`triple`中，函数参数是`x`，函数主体是`(x, x, x)`。考虑到这两个事实，编译器试图通过从函数参数和主体的每一部分向上工作来推断（即缩小）`triple`的类型。让我们看看我们可以从`triple`的每一部分推断出什么：

+   从`x`参数：没有，因此我们将其类型标记为（一个泛型类型）`'a`。

+   从主体`(x, x, x)`：我们已经将`x`的类型标记为`'a`，因此我们将主体的类型标记为`('a, 'a, 'a)`，即由三个相同类型的元素组成的元组类型

+   从整体函数：我们将参数标记为类型`'a'，并将主体标记为类型`('a, 'a, 'a)`，因此我们推断整个函数的类型为`'a => ('a, 'a, 'a)`。

注意，这似乎更像是一个排除过程，或者解决数独谜题的过程。我们尽可能多地根据我们所知道的信息缩小类型，直到我们不能再缩小为止。这正是我们从之前的章节中熟悉的推断和统一过程，但在泛型类型中，我们看到类型系统的新维度。

Reason 的类型推断过程是著名的，称为 **Hindley-Milner (H-M**) 类型推断*；* 它是一种数学特定的方法，用于检查任何给定的表达式，以尝试推断最适合该表达式的最一般类型。我们不会深入理论，但我们将涵盖使用 H-M 类型推断的实际操作。

通过检查 `triple` 的类型推断过程，我们可以看到 `wrap`、`makeObj` 和 `greet` 的推断是如何工作的。理解的关键点是这些函数中的每一个都有一个不使用其输入参数任何特定属性的体表达式；相反，体表达式在一个更大的表达式中组合输入参数，以通用的方式使用值，而不暴露任何关于类型本身的信息。例如，想象一下被给了一个某些对象（你不知道是什么），然后立即把它放在一个盒子里；你仍然不知道这个对象是什么，你只是知道你现在有一个包含某个对象的盒子。这就是结构化类型通用类型推断的工作方式。

# 特殊情况插值

在我们提到的函数中，`greet` 稍微特殊，因为类型推断并不完全像其他函数那样工作。对于前三个函数，由于参数在结构化类型表达式中使用，推断达到了一个通用类型。然而，对于 `greet`，`x` 参数被插入到一个字符串中；然而，字符串并不是结构化类型！正在发生的事情是，字符串插值是 BuckleScript 编译器提供的一个特殊逃生门，允许你将任何值转换为字符串，但仅当针对 JavaScript 时。

这里关键点是任何值转换为字符串。我们可以将这种行为视为一个函数，`'a => string`。由于我们随后使用 `print_endline` 打印字符串，最终的结果类型是 `unit`；因此，整体类型是 `'a => unit`。

# 类型参数

我们已经看到类型检查通过一个算法过程来确定任何给定表达式的最一般类型，当表达式是结构化类型，例如 `let pair(x) = (x, x)`，它可以推断出参数化类型（例如 `'a => ('a, 'a)`)，因为它不知道，或者不需要知道它们的确切内容。

**类型参数** 是一个尚未指定的类型，将在稍后使用时指定。Reason 支持在所有类型上使用类型参数，包括命名类型。这为记录和变体类型提供了新的数据建模维度（字面上）。现在让我们探索一些最基本但也很重要的参数化数据类型。

# 列表——建模多个

我们已经在第五章中看到了如何建模人员记录的列表，*在类型中放置替代值*，但那个数据结构仅限于存储人员记录的值。理想情况下，我们希望有一个可以存储任何类型值的结构，这样我们就不必为每个可能的元素类型重新实现类型及其操作。我们可以通过将列表类型参数化为元素类型来实现这一点，如下所示：

```js
/* src/Ch06/Ch06_List.re */
type list('a) = *Cons*('a, list('a)) | *Empty*; /* (1) */

/* (2) */
let *people* = *Ch04_RecordLiterals*.(*Cons*(*bob*, *Cons*(*jim*, *Cons*(*tom*, *Empty*))));

/* (3) */
let *greetOne*({*Ch04_RecordLiterals.id*, *name*}) = *print_endline*(
 {j|Hello, $*name* with ID $*id*!|j});

let rec *greetAll*(*people*) = switch (*people*) {
| *Cons*(*person*, *people*) => { /* (4) */
 *greetOne*(*person*);
 *greetAll*(*people*)
 }
| *Empty* => () /* (5) */
};
```

之前的例子展示了如何泛型地持有任何给定类型的对象，以及如何对它们进行特定操作。解释如下：

1.  在这里，我们使用显式类型参数`'a`（发音为'alpha'）对命名类型（确切地说，是一个变体类型）`list`进行参数化。这里我们只有一个，但类型可以有多个类型参数。声明类型参数的语法是`type typeName('param1, 'param2, ..., 'paramN)`。请注意，类型参数必须始终以撇号（`'`）字符开头，以区分常规类型。类型参数也被称为类型变量。

1.  我们构建一个由我们在之前模块中定义的`person`记录组成的列表。编译器可以推断其类型为`list(Ch04_RecordLiterals.person)`，因为我们正好在声明类型参数`'a`的地方插入了人员类型。

1.  我们定义了如何问候一个单独的人。这个操作在某种程度上没有使用类型参数，但它是一个构建后续操作的基础。

1.  在`greetAll`内部，我们对`people`进行了一个有趣的模式匹配。通过`Cons(person, people)`和`Empty`分支，编译器推断出`people`的类型为`list('a)`，通过`greetOne(person)`，它推断出`'a = person`；总体上，该函数的类型为`list(person) => unit`。

1.  当我们到达列表的末尾时，我们不想做任何事情，所以我们只返回`()`。

现在我们已经看到了如何构建和操作多态数据类型，让我们看看 Reason 内置的`list`类型的实现。内置实现的工作方式与前面提到的类似，但 Reason 提供了一些语法糖，使得与列表一起工作更加容易。让我们看看下面的代码片段：

```js
/* src/Ch06/Ch06_ReasonList.re */
let *people* = *Ch04_RecordLiterals*.[*bob*, *jim*, *tom*]; /* (1) */

let rec *greetAll*(*people*) = switch (*people*) {
| [*person*, ...*people*] => { /* (2) */
    *Ch06_List.greetOne*(*person*); /* (3) */
    *greetAll*(*people*)
  }
| [] => () /* (4) */
};
```

首先，请注意，我们已经去掉了类型声明，因为 Reason 的`list('a)`类型已经内置并且可以从每个模块中访问。更准确地说，它在`Pervasives`模块中定义，其内容默认可以从每个模块中访问。

1.  我们使用列表构造的语法糖，即`[elem1, elem2, ... elemN]`，来构造一个本质上类似于我们之前的列表`Cons(elem1, Cons(elem2, ... Cons(elemN, Empty) ... ))`。括号和逗号语法更简洁，更容易理解。

1.  在列表上进行模式匹配现在看起来像`[elem, ...restElems]`，以绑定第一个元素和剩余元素列表。`...`被称为**扩展运算符**，它被设计成类似于 JavaScript 的数组扩展功能，其工作方式类似。

1.  我们重用了已经定义的`greetOne`函数，因为它不依赖于特定的列表类型——它只是问候一个人。

1.  空列表模式现在看起来像`[]`而不是`Empty`。

注意列表语法的简洁性。它是为日常使用设计的，因为列表是 Reason 中最重要的一种数据类型，在函数式编程中也是如此。

# 选项 - 模拟无或一个

与`list('a')`类似，`Pervasives`模块也提供了一个数据类型，`option('a')`。这次让我们看看它的实际定义，因为它没有语法糖，如下所示：

```js
type option('a) = *Some*('a) | *None*;
```

在某些方面，这比列表是一个更简单的数据类型。它的真正效用来自于我们对变体情况赋予的意义：

+   `Some('a')`: 表示一个存在且已知的值

+   `None`: 表示一个不存在且未知的值

在 Reason 和一些其他语言中，没有 null 值的概念，所以这种选项类型用来表示一个值是存在还是不存在。当我们使用 null 时，我们可以使用选项，其好处是可选性（某个值可能存在或不存在）被捕获在类型系统中，而不是在幕后。由于`option`类型是一个变体，编译器帮助我们通过穷尽性检查来处理可能缺失的值。没有忘记处理 null 值并在运行时崩溃的风险。

存在和不存在可能仍然是一些模糊的概念，所以下面的代码是一个更具体的例子，其中我们尝试在列表中找到一个匹配的值：

```js
/* src/Ch06/Ch06_Option.re */
let rec *tryFind*(*needle*, *haystack*) = switch (*haystack*) { /* (1) */
| [*item*, ...*_items*] when *needle*(*item*) => *Some*(*item*) /* (2) */
| [*_item*, ...*items*] => *tryFind*(*needle*, *items*) /* (3) */
| [] => *None* /* (4) */
};

let *optionallyGreet*(*person*) = switch (*person*) { /* (5) */
| *Some*(*person*) => *Ch06_List.greetOne*(*person*)
| *None* => *print_endline*("No such person!")
};

let *idEq1*({*Ch04_RecordLiterals.id*}) = *id* == 1; /* (6) */
let *idEq4*({*Ch04_RecordLiterals.id*}) = *id* == 4;

*optionallyGreet*(*tryFind*(*idEq1*, *Ch06_ReasonList.people*)); /* (7) */
*optionallyGreet*(*tryFind*(*idEq4*, *Ch06_ReasonList.people*));
```

在这个例子中，我们定义了如何在列表中查找一个项目，并安全地处理缺失项的情况：

1.  我们传递一个列表去搜索（称为`haystack`）和一个测试函数（称为`needle`），该函数告诉我们是否找到了我们正在寻找的值。由于`haystack`是一个列表，我们可以在它上面进行模式匹配。

1.  第一个模式检查列表的第一个元素是否是我们想要的，这是由`needle`决定的。我们不想绑定其余的元素，所以我们在`_items`名称前加上下划线来告诉 Reason 忽略它。注意，我们在这里使用的是`when`子句，正如在第五章中介绍的，*在类型中放置替代值*。这相当于在分支体中使用`if`表达式，但更简洁。如果第一个元素与针匹配，我们将其放入`Some`构造函数中并对其进行评估。

1.  在第二种模式中，我们匹配忽略第一个项目后的剩余项目列表，并递归地尝试在该列表中找到我们想要的元素。但只有当第一个分支不匹配时，才会到达这个分支，这意味着元素不是列表中的第一个项目。这个分支本身评估为 `tryFind` 的结果，意味着 `Some(person)` 或 `None`。

1.  在最终分支中，我们必须处理 `haystack` 列表的另一种可能状态：为空。如果是这样，我们可能开始时就是一个空列表，或者通过递归最终变成了一个空列表。在任何情况下，我们都没有在列表中找到我们想要的元素，所以我们返回 `None`，在这种情况下意味着未找到*。

1.  这里，我们描述了如何问候可能或可能不在那里的人。编译器强制处理 `Some(person)` 和 `None` 两种情况——我们不能忘记处理缺失的值。

1.  我们定义了两个不同的 `needle` 函数，它们测试给定的人的记录是否有 ID 1 或 4。

1.  这里是回报：我们可以选择性地问候一个人，但仅在我们找到了具有给定 ID 的人在我们的 `people` 列表中。我们可以运行以下输出脚本来查看会发生什么：

```js
$ node src/Ch06/Ch06_Option.bs.js 
Hello, Bob with ID 1!
No such person!
```

如预期的那样，我们找到了并问候了 `ID 1` 的 (`Bob`) 人，但没有问候 `ID 4` 的人，因为没有这样的人。

在输出 JavaScript 中，BuckleScript 再次将尾递归的 `tryFind` 函数转换为一个简单的命令式循环。

这里，我们看到使用 `option` 类型的两个方面：我们可能需要在操作过程中表示值的出现或缺失，如果我们使用像 `option` 这样的变体类型，我们就能得到其详尽性检查的好处。

# 可变参数化类型 – ref 和 array

Reason 还提供了两种重要的参数化类型，允许它们的值在原地被修改。这种可变性为某些类型的算法带来了效率提升，但通常需要谨慎使用，因为，正如我们将看到的，它可能是一个错误来源。

# 管理对值的引用

我们已经在第四章的 *Mutable record fields* 部分，*Grouping Values Together in Types* 中看到了一个可变性的例子。有时，我们只需要管理一个或两个可变值，我们可能不想通过声明一个新的具有可变字段的记录类型来进行仪式。在这些情况下，我们可以利用内置的 `ref` 类型。`ref` 类型本质上给我们一个盒子，一个 ref，它让我们可以交换值。值本身不需要是可变的，只需要盒子本身：

```js
type ref('a) = {mutable *contents*: 'a}; /* (1) */
let *ref*: 'a => ref('a); /* (2) */
let (:=): (ref('a), 'a) => unit; /* (3) */
let (^): ref('a) => 'a; /* (4) */
let *incr*: ref(int) => unit; /* (5) */
let *decr*: ref(int) => unit;
```

之前的代码示例显示了 `ref` 类型的完整 API，解释如下：

1.  它被实现为一个只有一个可变记录字段的记录类型，但这个字段由类型参数 `'a'` 参数化，让我们可以为其任何类型重用它。

1.  我们可以使用 `ref` 函数将任何类型的值装箱，并将其放入一个 ref 中：`let count = ref(0);`。

1.  赋值运算符 `(:=)` 实现为一个函数，并且可以用作中缀位置：`count := 1;`。

1.  取消引用运算符 `(^)` 也实现为一个函数，但 Reason 允许我们在后缀位置使用它：`let countVal = count^;`。

1.  `incr` 和 `decr` 是用于增加和减少整数的便利函数，因为我们经常需要更新计数。

使用 `ref` 类型来管理突变的好处是可变性在类型级别上被捕获。我们可以从查看任何包含 `ref(something)` 的类型签名中看出，`something` 类型的某个东西正在改变。相比之下，当我们直接使用可变记录字段时，我们没有可变性的类型级别指示，只有记录定义本身。

让我们看看使用 `ref` 的一个示例：重新定义 `tryGreet` 函数以使用更命令式的风格遍历其输入列表并尝试找到所需的项目。我们需要三个 `ref`：列表的剩余部分、是否应该停止搜索，以及一个可选的找到的项目。只要我们还没有找到项目，我们就会继续搜索，但一旦找到，我们就会返回它：

```js
/* src/Ch06/Ch06_Ref.re */
let *tryFind*(*needle*, *haystack*) = {
  let *currHaystack* = *ref*(*haystack*);
  let *stop* = *ref*(*false*);
  let *currItem* = *ref*(*None*);

  while (!(*stop*^)) { /* (1) */
    switch (*currHaystack*^) { /* (2) */
    | [*item*, ...*_items*] when *needle*(*item*) => { /* (3) */
        *stop* := *true*;
        *currItem* := *Some*(*item*)
      }
    | [*_item*, ...*items*] => *currHaystack* := *items* /* (4) */
    | [] => *stop* := *true* /* (5) */
    };
  };

  *currItem*^ /* (6) */
};
```

在上一个示例中，我们使用了 Reason 的命令式特性（突变和循环），如下所述：

1.  我们会持续循环，直到我们明确表示应该停止。注意，命令式的 `while` 循环看起来就像我们从其他命令式语言（如 JavaScript）中期望的那样。

1.  在输入 `haystack list` 上的模式匹配仍然是遍历它的最方便的方式，所以我们在这里继续这样做。

1.  如果我们找到了我们正在寻找的项目，我们需要确保我们将停止指示符设置为 `true`，并且设置找到的项目，以便我们稍后可以返回它。

1.  如果我们还没有找到它，但列表不为空，将当前 `haystack` 设置为列表的剩余部分。

1.  如果列表为空，显然我们没有找到项目，因此我们需要停止。

1.  无论我们迄今为止找到了什么（或者没有找到什么），我们最终都需要取消引用并返回它。

这个命令式版本首先要注意的是它更冗长。跟踪可变状态在代码层面上涉及一些仪式。这并不是说 Reason 的命令式风格特别繁琐；在任何命令式语言中看起来都差不多。只是函数式风格，特别是递归风格，通常更简洁，因为递归调用实际上跟踪了当前状态，所以我们不需要。

第二个要注意的是，我们保留了相同的函数签名（`('a => bool, list('a)) => option('a)`）来实现这个版本，只是没有使用递归。如果我们需要，我们可以用递归实现替换这个实现，而我们的客户端代码不需要重新编译就可以使用它。这是许多静态类型系统的特性：如果我们保留类型签名，我们可以自由地替换实现。特别是 Reason 的类型系统使用签名来决定整个模块是否兼容。这有助于我们在构建时早期定位兼容性问题，而不是在运行代码时发现不兼容性。

# 管理值数组

有时，我们需要高效地管理和更改相同类型的大量值。Reason 提供了 `array` 类型来帮助处理这个问题。我们可以将数组想象成一条由相同大小的盒子组成的连续线，每个盒子都可以存储相同类型的值。形式上，它是一个多态类型，如下所示：

```js
type array('a);
```

`array` 数据结构在特征上与其他语言中可能看到的结构相似，其中：

+   它允许随机访问其元素。

+   它不允许像 `list` 使用展开操作（`[item, ...items]`）那样递归遍历元素。

然而，它确实允许对其元素进行基本的模式匹配。以下是其使用的一个示例：

```js
/* src/Ch06/Ch06_Array.re */
let *empty* = [||]; /* (1) */
let *singleton* = [|1|];
let *multi* = [|*false*, *true*, *true*|];

*multi*[1] = *false*; /* (2) */
Js.log(*multi*[1]); /* (3) */
```

在前面的例子中，我们看到以下简单的数组使用：

1.  我们可以创建一个空数组，以及包含一个或多个元素的数组。数组分隔符是 `[|` 和 `|]`，并且用于区分它们与在 Reason 中更频繁使用的列表。

1.  我们可以分配到数组中的任何有效索引（分配到越界索引会导致运行时异常）。

1.  我们可以读取任何有效索引处的值（从越界索引读取也会导致运行时异常）。

对于数组，索引的概念很重要，正如我们在这里所看到的。基于索引的随机访问是常数时间的，但作为交换，我们必须注意只访问有效的索引。

为了更好地理解数组的有用性，让我们尝试使用一个数组和几个更新和检查棋盘的函数来实现井字棋棋盘，如下所示：

```js
/* src/Ch06/Ch06_TicTacToe.re */

/* Each slot on the board can be taken by X or O, or it can be empty. */
type slot = *X* | *O* | *Empty*;

let *newBoard*() = *Array.make*(9, *Empty*); /* (1) */

/* Coords are as follows on the board:
   1 2 3
   4 5 6
   7 8 9 */
let *play*(*player*, *coord*, *board*) = *board*[*coord* - 1] = *player*; /* (2) */

let *xWon*(*board*) = switch (*board*) {
| [|*X*, *X*, *X*, /* (3) */
    _, _, _,
    _, _, _|]
| [|_, _, _, /* (4) */
    *X*, *X*, *X*,
    _, _, _|]
| [|_, _, _,
    _, _, _,
    *X*, *X*, *X*|]
| [|*X*, _, _,
    *X*, _, _,
    *X*, _, _|]
| [|_, *X*, _,
    _, *X*, _,
    _, *X*, _|]
| [|_, _, *X*,
    _, _, *X*,
    _, _, *X*|]
| [|*X*, _, _,
    _, *X*, _,
    _, _, *X*|]
| [|_, _, *X*,
    _, *X*, _,
    *X*, _, _|] => *true* /* (5) */
| _ => *false* /* (6) */
};
```

这是一个设计数组的有趣示例，以便我们可以对其进行字面意义上的模式匹配，如下所述：

1.  我们可以通过创建一个新的数组并使用 `Array.make` 库函数填充 `Empty` 槽来创建一个新的游戏棋盘。这个函数为我们提供了一个所需长度的数组，并且所有索引都填充了单个值。

1.  由于我们接受一个一维的棋盘坐标，我们只需简单地减去一将其转换为零索引的数组索引。

1.  我们可以针对数组的精确结构进行模式匹配。在这种情况下，我们的九元素数组如果将其分成三行纯粹是为了展示，可以看起来就像一个井字棋棋盘。

1.  我们可以使用 `or` 模式来捕获玩家 X 获胜的所有情况。

1.  对于所有这些返回 `true`。

1.  否则，我们返回`false`。

在这种情况下，当玩家移动时，我们可以轻松地设置棋盘上的任何位置，这符合数组的随机访问能力。相比之下，在列表中设置随机位置将是一个低效的操作，因为唯一的方法是遍历列表的每个元素，然后执行多次列表拆分和连接操作，直到创建最终的输出列表。

通常，当我们需要对多个元素进行频繁更新时，数组非常有用。常见场景是像素缓冲区和手动管理的内存区域。幸运的是，Reason 使这些场景相对容易实现。

# 变异和类型推断限制

通常，编译器会为我们推断各种类型，但它确实有一些限制。有时，我们需要给它一点提示，以帮助它得到正确的推断结果，例如。我们需要注意的主要情况被称为**值限制**。值限制基本上意味着可变值不能是泛型的，编译器必须完全知道它们的类型。以下是在文件`src/Ch06/Ch06_ValueRestrictionError.re`中取消注释代码时你会得到的错误示例：

```js
(Output from bsb -w)
  We've found a bug for you!
  src/Ch06/Ch06_ValueRestrictionError.re 2:17-24

  1 │ /* src/Ch06/Ch06_ValueRestrictionError.re */
  2 │ let optionArr = [|None|];
  3 │ let optionRef = ref(None);

  This expression's type contains type variables that can't be generalized:
  array(option('_a))

  This happens when the type system senses there's a mutation/side-effect, in combination with a polymorphic value.
  Using or annotating that value usually solves it. More info:
  https://realworldocaml.org/v1/en/html/imperative-programming-1.html#side-effects-and-weak-polymorphism
```

注意`optionArr: array(option('_a'))`的类型推断。类型变量名前的下划线前缀是编译器表示遇到了值限制的方式。名为`'_a'`的类型变量被称为**弱类型变量**（在这个意义上，它的实际类型可能会以后改变）。

类型在编译时推断后可能会改变，这是一个坏主意。例如，让我们考虑如果编译器推断类型为`array(option('a'))`会发生什么。在代码的后续部分，我们可以将该索引设置为`Some(1)`，然后稍后将其设置为`Some(false)`。这将完全破坏类型系统，并使我们无法在任何代码点确定确切的类型。编译器设计者决定防止这种情况发生。这只是他们多年来为防止运行时类型错误渗入程序而做出的类型安全性决策之一。

作为有效代码，我们可以使用`array(option(string))`精确类型，正如你在替代代码文件（`src/Ch06/Ch06_ValueRestrictionErrorFixed.re`）中看到的那样，该文件可以正确编译。其代码如下：

```js
/* src/Ch06/Ch06_ValueRestrictionErrorFixed.re */
let optionArr: array(option(string)) = [|None|];
let optionRef: ref(option(string)) = ref(None);
```

让我们看看另一个值限制错误。再次，取消注释文件`src/Ch06/Ch06_ValueRestrictionOtherError.re`中的代码，你将得到以下编译错误：

```js
(Output from bsb -w)
We've found a bug for you!
  src/Ch06/Ch06_ValueRestrictionOtherError.re 5:15-28

  1 │ /* src/Ch06/Ch06_ValueRestrictionOtherError.re */
  2 │ let pair(x) = (x, x);
  3 │ let pairAll = List.map(pair);

  This expression's type contains type variables that can't be generalized:
  list('_a) => list(('_a, '_a))

  This happens when the type system senses there's a mutation/side-effect, in combination with a polymorphic value.
  Using or annotating that value usually solves it. More info:
  https://realworldocaml.org/v1/en/html/imperative-programming-1.html#side-effects-and-weak-polymorphism
```

这个问题稍微有点棘手，因为没有明显的突变。`pairAll` 函数的目的是将项目列表转换为这些项目的对（二元组）列表。问题是 `pair` 函数是通用的；编译器无法确定它是否可能会修改任何内容。如果我们有一个单态（即非通用）函数，比如 `let pair(x) = (x + 1, x - 1);`，那么编译器就能确定输入和输出只是 `int`，并且没有涉及任何突变。

然而，还有另一种解决这个特定错误的方法；记住，它被称为*值限制*。换句话说，只有值受到这种限制。如果我们将 `pairAll` 扩展为一个显式的函数，那么错误就会消失，如下所示：

```js
let *pairAll*(*list*) = *List.map*(*pair*, *list*);
```

我们固定的代码可以在 `src/Ch06/Ch06_ValueRestrictionErrorFixed.re` 中找到。

这个有趣的方法让编译器相信，是的，这确实是一个函数，因此值限制不适用。

# 强制与幻影类型的不同

因为我们可以声明可以插入任何类型参数的类型，包括类型实际上没有使用的类型参数。这些被称为幻影类型参数，或者更非正式地称为**幻影类型**。

幻影类型的一个常见用途是在一种类型安全的**构建器模式**中。（构建器模式是一段代码，帮助我们根据特定规则构建对象。）例如，我们可能想要构建语法上有效的 SQL 语句。做到这一点的一种方法是有这样一个验证函数，它接受一个输入 SQL 语句，并在运行时决定它是否遵循 SQL 语法规则。这个函数可能会尝试解析输入语句并构建一个表达式树。如果树可以构建，则语句有效。否则，它无效。

另一种方法是提供一组函数，这些函数静态地强制只创建语法上有效的语句。这个方法的神奇之处在于，我们可以在类型体实际上没有使用类型参数时，告诉编译器类型参数应该是什么。类型定义中没有与我们的说法相矛盾的内容，因此编译器必须接受它。

以下是一个简化的示例：

```js
/* src/Ch06/Ch06_PhantomTypes.re */

module *Sql*: {
  type column = string; /* (1) */
  type table = string;
  type t('a); /* (2) */

  let *select*: list(column) => t([`*select*]); /* (3) */
  let *from*: (table, t([`*select*])) => t([`*ok*]);
  let *print*: t([`*ok*]) => string;
} = {
  type column = string;
  type table = string;
  type t('a) = string;

  let *select*(*columns*) = { /* (4) */
    let *commalist* = *String.concat*(", ", *columns*);
    {j|select $*commalist*|j}
  };

  let *from*(*table*, *t*) = {j|$*t* from $*table*|j};
  let *print*(*t*) = *t*; /* (5) */
};

let *sql* = *Sql*.(*select*(["name"]) |> *from*("employees") |> *print*); /* (6) */
Js.log(*sql*);
```

为了简化，我们在这个 SQL 构建模块中只处理 `select` 和 `from` 子句，具体解释如下：

1.  我们将几个类型别名为文档用途。

1.  这个类型具有幻影类型参数；对于模块消费者来说，它看起来像是一个正常的参数化类型。内部，它不包含或以其他方式使用其参数类型的任何值。

1.  这个函数是构建的入口点：它接受一个列列表，并返回一个部分构建的 SQL 语句。我们无法对这个返回值做任何事情，除了将其输入到下一个函数 `from` 中。注意，类型参数实际上是适当命名的多态变体的类型；它们只是作为标签。

1.  `select` 和 `from` 的实现非常简单：它们只是构建形式为语法有效 SQL 语句的正常字符串。最有趣的是，它们的类型被强制接受参数，这样它们就只能以特定的顺序调用：`select`，`from`，`print`。

1.  `print` 函数非常简单，它只是返回构建的字符串。我们可以检查它，然后将其传递给 SQL 引擎运行。

1.  我们通过按正确的顺序调用函数，并由类型系统强制执行，构建一个语法有效的 SQL 语句，然后将它们输出到终端。请注意，`|>` 操作符被称为 **pipe-forward**，它用于将一个函数的输出作为下一个函数的输入。我们将在下一章中介绍常见的操作符。

以下代码是如果我们尝试打印一个无效的 SQL 语句时可能会得到的错误：

```js
(Output from bsb -w)
We've found a bug for you!
  src/Ch06/Ch06_PhantomTypes.re 25:36-40

  23 │ };
  24 │ 
  25 │ let sql = Sql.(select(["name"]) |> print); /* (6) */
  26 │ Js.log(sql);

  This has type:
    Sql.t([ `ok ]) => string
  But somewhere wanted:
    Sql.t([ `select ]) => 'a
  These two variant types have no intersection
```

这种类型错误表示 `print` 函数期望一个完整的 `ok` SQL 语句，但只收到了一个 `select` 子句。类型参数与模块的函数一起工作，确保 SQL 以正确的方式构建。

# 摘要

在本章中，我们深入探讨了 Reason 的参数化类型，学习了类型参数以及它们如何扩展类型以成为泛型，我们在 Reason 中使用的常见参数化类型，编译器对使用参数化类型和突变的同时的限制，以及如何使用幻影类型参数强制相同的底层类型对编译器看起来不同。

我们还看到了一些将函数作为参数传递给其他函数的例子，例如 `tryFind` 和 `List.map`。在下一章中，我们将彻底介绍函数以及 Reason 如何让我们将它们作为一等对象处理，这样我们就可以在代码中传递它们，使代码的行为更加灵活。
