# 使用多种不同类型重用代码

在前面的章节中，我们介绍了很多特定的技术。其中，我们看到了如何使用模块来封装定义类型和值的代码。我们还看到了函数和函数类型，包括它们的用法技术，如柯里化和部分应用。

在本章中，我们将基于我们迄今为止所看到的，并涵盖以下主题：

+   Reason 中的多态技术

+   使用模块和函子编写泛型代码

# Reason 中的多态性

多态性，是许多编程语言中使用的技巧类别，允许编写适用于不同类型或对象的代码（例如，在 C++或 Java 等语言中）。精确地看待事物可以显示存在几种多态技术或类型。

我们将在这里讨论两种实现多态性的方法：

+   参数多态性

+   **特设多态性**

# 具有参数多态性的通用函数

参数多态性允许以通用方式编写函数或数据类型，这意味着它可以以相同的方式处理不同类型的值。这既有趣又强大，因为它意味着使用参数多态性编写的函数可以在不同的数据类型上工作。

在 C++中，参数多态性通常被称为**泛型编程**或**编译时多态性**。

我们可以在 ReasonML 中使用类型进行参数多态性。我们有一个特殊的功能来实现这一点，**类型变量**，这在第七章中已经遇到过，在*表示操作的类型*的例子中，与*函数字面量*相关。我们不是使用`int`或`string`等具体类型作为参数或结果，而是使用`类型变量`。因此，使用`类型变量`作为参数的类型将确保接受任何类型的值。

使用`类型变量`的函数被称为**通用函数**。

让我们用一个例子来解释。*恒等*函数是解释通用函数是什么的简单例子。恒等函数（让我们称它为`id()`）只是返回其输入参数。它定义如下：

```js
let *id* = x => x;
```

根据这个定义，ReasonML 使用`'a`来推断输入参数的类型，这意味着它使用类型变量来表示接受任何类型的值。函数的返回类型也是以相同的方式推断的，即其参数的类型，即`'a`类型变量。这就是通用函数的行为。

每当我们遇到以`'`开头的类型名时，例如`'a`（表示*任何类型*），这定义了一个类型变量。

这里，是此类函数的另一个示例。我们可以考虑一个返回列表最后一个元素的函数`(lastElem)`。关键是列表的元素可以是任何类型。此外，由于我们必须考虑空列表的情况，我们将使用带有类型变量的选项类型。

我们可以为该函数编写一个接口（参见`src/Ch08/Ch08_GenericFunctionLastElementOfList.rei`文件），如下所示：

```js
let *lastElem*: *list('a)* => option('a);
```

此外，基于 Reason 中列表的工作方式，我们可以定义一个**递归**函数（使用`rec`关键字），如下所示：

```js
let *rec* *lastElem* = aList =>
  switch aList {
  | [] => None
  | [x] => Some(x)
  | [_, ...l] => lastElem(l)
};
```

让我们用以下`Js.log`调用进行测试：

```js
Js.log(*lastElem*([1, 3, 2, 5, 4]));
Js.log(*lastElem*(["a", "b", "c", "d"]));
```

如预期的那样，输出（参见`src/Ch08/Ch08_GenericFunctionLastElementOfList.re`文件）显示结果值为`4`和`d`。

# 特设多态或重载

特设多态是我们接下来要讨论的另一种技术。它也被称为**重载**，它为相同的操作提供不同的实现，例如`+`操作。为了继续这个例子，我们可能会在某些编程语言中，例如 Python，发现`+`操作有用于数字加法的实现，另一个用于字符串连接，还有一个用于列表或数组连接。

几乎所有编程语言都支持内置操作的多态，例如`+`、`-`和`*`。

ReasonML 目前不支持特设多态。例如，我们有用于整数加法的不同`+`操作符，用于浮点数加法的`+.`操作符，以及用于字符串连接的`++`操作符。如果需要，我们必须在应用给定操作符之前手动将值转换为正确的类型。

ReasonML 最终可能通过目前正在开发的模块隐式来实现特设多态。

# 使用函数式编程的通用代码

正如我们所见，模块在 OCaml 和 ReasonML 中非常重要，有助于将代码组织成具有指定接口的单位。除此之外，我们现在将看到它们可以用所谓的**函数式**来构建通用代码。

# 什么是函数式？

函数式是一种函数，其参数是模块，其结果是另一个模块。

函数式允许我们通过新功能扩展现有模块，而无需为不同类型编写大量重复代码。

函数式编程的语法如下所示：

```js
module *F* = (M1: I1, ···): ResultI => {
   ...
 };
```

基于这些特性，请注意以下内容：

+   `F`函数式编程的参数是一个或多个`M1`模块等

+   每个参数模块必须通过接口进行类型化（`I1`用于`M1`等）

+   结果类型（`ResultI`）的接口是可选的

一些示例将帮助我们理解。

# 示例 1 – 在标准库中查找

最好的例子是标准库中的`Set`模块。

`Set`类型有一个排序（例如，在整数集合中，*1 < 2*和*2 > 1*），并且元素是唯一的。注意，在其他语言，如 Python 中，也有这种情况。

要在 ReasonML 和 OCaml 中使用集合，首先必须创建一个。你可以通过调用`Set.Make`来实现，这是一个函数式编程，它接受作为输入的另一个模块，该模块必须在其内部实现一个`compare()`函数，并返回我们的`Set`类型模块。

例如，我们可以为整数集编写如下代码：

```js
module *IntSet* =
   *Set.Make*(
     {
       let *compare* = Pervasives.compare;
       type *t* = int;
     }
 );
```

我们得到一个新的模块，它提供了方便地处理整数集合的函数，例如 `IntSet.of_list()`：

```js
let *myIntSet* = IntSet.of_list([1,2,3]);
```

让我们在控制台中显示结果：

```js
Js.log(myIntSet)
```

当我们使用 `node` 命令运行编译该程序文件（`src/Ch08/Ch08_FunctorsExample1.re`）生成的 JS 文件时，我们得到输出，显示了创建的 `IntSet`：

```js
[ 0, 1, [ 0, 2, [ 0, 3, 0, 1 ], 2 ], 3 ]
```

通过这个第一个示例，我们了解到了函子的作用。

# 示例 2

这里还有一个从 ReasonML 文档中摘取的另一个示例，也是关于集合的，我们将对其进行讨论。我们将展示一个 `MakeSet` 函子，它接受一个 `Comparable` 类型的模块并返回一个可以包含此类可比较项的新集合。

我们首先定义 `Comparable` 类型，如下所示：

```js
module type *Comparable* = {
   type *t*;
   let *equal*: (t, t) => bool;
 };
```

现在，我们定义函子，如下所示：

```js
module *MakeSet* = (Item: Comparable) => {
   /* 1 */
   type backingType = list(Item.t);
   let *empty* = [];
   let *add* = (currentSet: backingType, newItem: Item.t) : backingType =>
     if (*List.exists*((*x*)=> *Item.equal*(*x, newItem*)*, currentSet*)) {
       currentSet /* 2 */
     } else {
       [
         *newItem*,
         ...currentSet /* 3 */
       ]
     };
 };
```

下面是这个代码块的解释：

(1) 我们使用列表作为我们的数据结构。

(2) 如果项目存在，则返回相同的集合。

(3) 否则，将元素添加到集合的前面并返回它。

现在，让我们记住我们想要创建一个集合，其元素是整数对。我们创建输入模块 `IntPair`，它遵循 `MakeSet` 所需的 `Comparable` 签名，如下所示：

```js
module *IntPair* = {
   type *t* = (int, int);
   let *equal* = ((x1, y1), (x2, y2)) => x1 == x2 && y1 == y2;
   let *create* = (x, y) => (x, y);
 };
```

这意味着我们可以使用函子编写以下内容：

```js
module *SetOfIntPairs* = *MakeSet*(IntPair);
```

最后，让我们添加一些代码来使用生成的模块：

```js
let *aSetOfPairItems*: SetOfIntPairs.backingType = SetOfIntPairs.empty;
Js.log(aSetOfPairItems);
let *otherSetOfPairItems* = SetOfIntPairs.add(aSetOfPairItems, (1, 2));
Js.log(*otherSetOfPairItems*);
let *thirdSetOfPairItems* = SetOfIntPairs.add(otherSetOfPairItems, (2, 3));
Js.log(thirdSetOfPairItems);
```

这应该足以得到一些有趣的结果。

当我们使用 `node` 命令运行由编译我们的 Reason 代码文件（`src/Ch08/Ch08_FunctorsExample2.re`）生成的 JS 文件时，我们得到以下输出：

```js
0
[ [ 1, 2 ], 0 ]
[ [ 2, 3 ], [ [ 1, 2 ], 0 ] ]
```

我们做了什么？使用函子，我们能够从一个模块创建一个新的模块 `SetOfIntPairs`。新模块具有 `add` 函数等特性。使用该模块，我们可以创建一个空集（输出中的 `0`），我们可以根据需要向其中添加 `int` 实例的成对（使用之前提到的 `add` 函数）。

# 示例 3

现在，我们将使用来自 Axel Rauschmayer 的一个示例，该示例可以在他的仓库中找到，网址为 [`github.com/rauschma/reasonml-demo-functors`](https://github.com/rauschma/reasonml-demo-functors)。

为了更精确，让我们使用带有微小调整的可打印对函子示例，以帮助我们更容易地理解这如何有用。

在定义函子之前，我们必须定义其参数的接口。在这里，我们将有一个单个参数，即 `PrintablePair` 模块。为此，我们将定义一个第一类型 `PrintableType`，它将被 `PrintablePair` 使用。我们定义它如下：

```js
module type *PrintableType* = {
   type *t*;
   let *print*: t => string;
 };
```

现在，我们添加 `PrintablePair` 类型的定义，如下所示：

```js
module type *PrintablePair* = (First: PrintableType, Second: PrintableType) => {
   type *t*;
   let *make*: (First.t, Second.t) => t;
   let *print*: (t) => string;
 };
```

然后，我们可以定义函子，如下所示：

```js
module *Make*: *PrintablePair* = (First: PrintableType, Second: PrintableType) => {
 type *t* = (First.t, Second.t);
 let *make* = (f: First.t, s: Second.t) => (f, s);
 let *print* = ((f, s): t) =>
   "(" ++ First.print(f) ++ ", " ++ Second.print(s) ++ ")";
 };
```

现在，我们有代码，我们将使用函子，从定义 `PrintableString` 和 `PrintableInt` 模块开始。

我们定义 `PrintableString` 如下：

```js
module *PrintableString* = {
   type *t*=string;
   let *print* = (s: t) => s;
 };
```

然后，我们定义 `PrintableInt` 如下：

```js
module *PrintableInt* = {
   type *t*=int;
   let *print* = (i: t) => string_of_int(i);
 };
```

现在，我们添加其余的代码，如下所示：

```js
module *PrintableSI* = Make(PrintableString, PrintableInt);
let () = *PrintableSI*.({
   let *pair* = *make*("Jane", 53);
   let *str* = *print*(pair);
   *print_string*(str);
 });
```

当我们使用 `node` 命令运行由 Reason 文件（`src/Ch08/Ch08_FunctorsExample3.re`）生成的 JS 代码时，我们得到以下输出：

```js
(Jane, 53)
```

我们完成了！

# 摘要

我们已经看到，ReasonML 支持使用 `类型变量` 进行参数多态性，这是语言特性之一。当使用 `类型变量` 作为函数参数的类型时，接受任何类型的值作为该参数。这种技术允许编写我们所说的泛型函数，并在 ReasonML 中的代码复用中扮演着重要的角色。

相比之下，在流行的编程语言中支持的另一种多态性——临时多态性，在 ReasonML 中尚不存在。但是，正在努力在未来版本中纠正这一不足。

模块在代码复用中也扮演着重要的角色。但，这并非全部。除了它们自身允许的功能之外，ReasonML 还有一个强大的特性，可以增强我们对模块的使用能力：*函子*。它们就像特殊的函数，接受一个或多个模块作为输入，并返回一个模块。这为编程泛型开辟了一些可能性。

在下一章中，我们将探讨 ReasonML 的技术，用于扩展类型以添加行为。
