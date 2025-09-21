# 构建你的第一个 Elm 应用

欢迎来到第二章。本章的目标是理解以下主题：

+   不可变数据结构

+   Elm 架构基础——与模型、视图和更新一起工作

+   Elm 中的 *Messages*

+   单向数据流

+   理解 `beginnerProgram` 函数

+   加强 Elm 中 HTML 函数的使用

+   Elm 中的 `If-else` 表达式

+   Elm 中的 `Case` 表达式

+   Elm 数据结构的基础（列表、元组、记录、集合、数组和字典）

+   Elm 中的联合类型

+   在 Elm 中使用模运算进行计算

完成本章后，你将能够做到以下事情：

+   与 Elm 架构一起工作

+   在你的应用中使用 `if-else` 表达式和 `case` 表达式

+   在 Elm 中构建一个非常简单的 *Fruit* Cou*nter* 应用

+   在 Elm 中构建一个非常简单的 *FizzBuzz* 应用

# 让我们构建一个应用

在本节中，我们将讨论不可变数据结构和 Elm 架构的基础。为了使事情更加实用，我们将构建一个非常简单的应用，这将有助于我们强化这些重要概念。

# 不可变数据结构

Elm 是一种函数式编程语言。函数式编程语言的信条之一是数据结构是不可变的。一旦创建，它们就不能改变。在实践中，这意味着 Elm 中的函数将接受一个数据结构作为其参数，然后返回一个全新的数据结构。

如果你这么想，这完全合理。如果所有数据结构都是不可变的，我们如何处理变化？更具体地说，我们如何处理 Elm 函数对现有数据引入的变化？唯一的明显答案是创建全新的数据。

Elm 如何在实际中应用这一点？

假设我们想要在健康领域构建一个应用。这个应用是一个简单的倒计时应用，只显示一个按钮和一个数字。应用从数字 5 开始。应用的想法是用户每次吃健康小吃时（比如，一块水果）就按按钮。这样，用户将通过确保每天吃五块水果来改善他们的健康习惯。

# Elm 架构

我们对 Elm 架构需要了解的最基本的东西是它由四个部分组成：模型、视图、消息和更新。

需要注意的是，架构通常被描述为由仅三个部分组成：模型、视图和更新。然而，为了清晰和便于学习，在我们 Elm 之旅的这个阶段，我们可以将消息视为 Elm 架构的同等构建块。在后面的章节中，当我们深入理论和实践时，我们将澄清这些区别。然而，为了有一个清晰的思维模型，我们将 Elm 架构视为由四个组成部分构成。

# 模型

模型持有我们的应用程序状态。在我们的*水果计数器*应用程序中，模型非常简单：它只持有作为其唯一数据结构的一个整数。当我们首次运行我们的应用程序时，模型持有值为 5。

由于它是以数据结构的形式表示的，并且由于 Elm 中的数据结构是不可变的，因此我们的模型必须作为前一个数据的副本加上对其所做的更改来更新。模型在函数对其操作时更新。

函数何时会操作模型？

由于我们正在构建一个非常简单的应用程序，函数将仅在用户点击我们应用程序中的一个按钮时操作模型。点击此按钮将使模型中的当前值递减 1。由于 Elm 中的数据是不可变的，函数必须返回一个包含更新的新模型副本。

# 视图

视图可以被视为*在屏幕上查看模型的方式*。视图是一个函数，我们将其传递给模型。因此，视图将模型作为其参数，并返回 HTML，这些 HTML 将在浏览器中渲染。

另一种思考视图的方式是，它*是一种允许用户与模型交互的方式*。根据我们在上一章中制作的 API 参考，视图可以看作是模型的一个视觉 API。它是用户以结构化方式操作模型的一种方式。

当用户与视图交互时，他们会通过改变其状态来操作模型。由于我们的应用程序非常简单，改变状态的唯一方式是按下我们应用程序中的一个按钮，这将递减模型当前持有的值。

这一次按钮点击将使视图向更新发送一个消息。

# 消息

在用户与视图交互（按下按钮）之后，此操作的通告将以消息的形式发送到更新函数。由于只有一个可能发生的行为，该消息很简单：*递减*。

当然，在任何现实的应用中，消息将需要更多的逻辑，但为了简单起见，让我们将其保留为*递减*。

# 更新

更新函数接收消息。接下来，更新函数根据接收到的消息确定如何更新我们应用程序的状态，即模型。一旦更新完成，就会创建一个新的模型并渲染视图。然后用户与视图交互，这会导致再次发送消息。更新接收消息并更新模型，然后循环继续。

# 单向数据流

以数据流的方式看待我们的应用程序是一种有趣的方法。我们之所以有这个概念并在我们的讨论中使用它，是因为它是一种帮助我们观察 Elm 应用程序状态变化的有用方式。

由于 Elm 建立在许多限制之上，因此将其架构也应用于这种限制性的理念是有意义的。

如果你看看我们的简单应用中正在发生的事情，你会注意到数据总是单向移动：从模型到视图到消息到更新到模型。这就是单向数据流的基本原理。

这个概念使我们能够轻松跟踪状态变化，同时也使得对这些变化的推理变得容易得多。

# 我们的应用需求

在本书的这一部分，我们已准备好构建我们的第一个应用：

+   我们对函数式编程的工作方式有一个概念

+   我们理解了 Elm 架构运作的最基本概念（模型、视图、消息、更新）

+   我们知道我们想要我们的应用做什么

带着这些知识，我们现在可以构建我们的应用。

# 构建水果计数器应用

让我们从我们的应用骨架开始。让我们将浏览器指向 [Ellie-app.com](http://Ellie-app.com)，我们将看到以下代码：

```js
module Main exposing (main)

import HTML exposing (HTML, text)

main : HTML msg
main =
    text "Hello, World!"
```

以下代码将是我们的应用起点。我们将通过逐渐添加功能来构建它，并在适当的时候解释底层概念，即当我们需要理解它们的时候。

# 展示我们所需要的一切

让我们从更新`module Main`开始，通过在括号内添加两个点来暴露一切，我们将对导入的`HTML`模块做同样的事情，因为我们想要能够使用所有可用的 HTML 函数。具体来说，我们需要访问`h1`、`p`和`button`函数。

接下来，读取为`main : HTML msg`的行是一个可选的类型注解，为了演示我们可以没有它工作，我们将通过在行首放置两个连字符和一个空格来注释掉它。

包含类型注解被认为是最佳实践，我们只是将其注释掉以显示我们的应用即使没有类型注解也可以运行（以及演示如何在 Elm 中添加单行注释）。

我们的应用现在看起来是这样的：

```js
module Main exposing (..)

import HTML exposing (..)

-- main : HTML msg
main =
    text "Hello, World!"
```

要预览这个阶段的 app，只需在 Ellie-app 中编译它。

# 模型

让我们添加我们的模型：

```js
-- MODEL

type alias Model = 
    Int
```

我们的模型只是一个简单的整数类型。

现在，我们将简单地忽略这个`type alias`的含义，因为它只会让我们在理解基本概念时分心。我们将在本书的后面部分回到类型。

# 视图

我们的`View`函数将接收当前模型并将返回以下 HTML：

```js
-- VIEW

view model =
    div [] [ h1 [] [ text ("Fruit to eat: " ++ (toString model)) ] ] 
```

在 第一章，*为什么现在是学习 Elm 的绝佳时机？*，我们探讨了使用 Elm 的 HTML 函数将一些基本的 HTML 渲染到屏幕上的情况。鉴于前面的代码，让我们快速讨论`view`函数将要渲染的 HTML。

我们分配给 `view` 函数的代码位于等号右侧（Elm 的赋值操作符）。在这段代码中，我们正在运行 `div` 函数。像 Elm 中的所有其他 HTML 函数一样，`div` 函数有两个方括号对。第一对方括号可选地列出 `div` 函数的 HTML 属性，第二对方括号列出实际的 `div` 内容。

我们留出了第一对方括号为空，这意味着我们没有给 `div` 函数指定任何属性。然后，在 `div` 函数的第二对方括号内，我们传递了 `h1` 函数。

像所有其他 HTML 函数一样，`h1` 函数也有两个方括号对。在上面的例子中，我们的 `h1` 函数没有指定任何属性（因为第一对方括号是空的——就像我们在 `div` 中做的那样，它的父函数/元素）。在 `h1` 函数的第二对方括号内，我们调用 `text` 函数。

`text` 函数将渲染一个文本节点。要输出到文本节点中的文本被括号包围。在括号内，我们使用一个字符串字面量，并将其与 `model` 的值（使用 `toString` 函数转换为字符串）连接起来。

我们刚刚学习了 Elm 的一个操作符，即 `++` 操作符。

在 Elm 中，`++` 是字符串连接操作符，用于将两个独立的字符串连接在一起。

由于我们将模型初始化为 5 的值，因此 `view` 函数的前置代码最初将返回以下 HTML 代码：

```js
<div>
    <h1>Fruit to eat: 5</h1>
</div>
```

我们的观点函数现在在最低级别上已经准备好了。接下来，我们将处理消息部分。

# 消息

让我们现在看看消息部分，我们将声明一个新的类型，我们将其称为 `Msg`：

```js
-- MESSAGE

type Msg = 
    Decrement
```

如本章前面所述，我们不会在此处解释类型。

# 更新

是时候添加我们的更新了。在上一个步骤中，我们已声明我们的特殊类型 `Msg`。我们将将其传递给 `update` 函数：

```js
-- UPDATE

update msg model =
    model - 1
```

到目前为止，你应该很容易猜到 `update` 函数将做什么：它将接受一个 `msg` 和一个 `model`，并将返回一个 `model` 的副本，其值减 1。

在我们 *Fruit Counter* 应用程序开发的这个阶段，我们只需要让模型（Model）、视图（View）和更新（Update）协同工作，为此，我们将使用 `beginnerProgram` 函数。

# 添加 beginnerProgram 函数

是时候添加 `beginnerProgram` 函数了，我们将将其分配给 `main` 函数。

我们当前的 `main` 函数看起来是这样的：

```js
-- main : HTML msg
main =
    text "Hello, World!"
```

让我们添加我们的更新后的 `main` 函数，现在它将分配给 `beginnerProgram` 函数：

```js
-- main : HTML msg
main =
  beginnerProgram { model = 5, view = view, update = update }
```

如您所见，我们只是调用了 `beginnerProgram` 函数。然后，我们传递 `model`、`view` 和 `update` 函数，并为每个函数分配一个值。

我们初始化模型为`5`的值。对于`update`，我们将其赋值给`update`函数的值。`view`同样设置为`view`函数的值。

在我们对`main`函数做出这些更改之后，我们应用的完整代码现在看起来是这样的：

```js
module Main exposing (..)
import HTML exposing (..)

-- main : HTML msg
main =
  beginnerProgram { model = 5, view = view, update = update }

-- MODEL
type alias Model = 
    Int

-- VIEW
view model =
    div [] [ h1 [] [ text ("Fruit to eat: " ++ (toString model)) ] ]   

-- MESSAGE
type Msg = 
    Decrement

-- UPDATE
update msg model =
    model - 1
```

如果我们现在运行应用，一切都会正常工作，屏幕上会出现以下输出：要吃的水果：5。

尽管我们的应用非常基础，但一切正常，我们没有遇到任何编译器错误，这真是太好了。然而，我们还没有做的一件事是，我们没有添加按钮，这是启动我们应用当前状态改变的一个入口点。

在我们添加这个按钮之前，请随意查看前面的代码，并稍微思考一下一个完全无状态的应用。目前，我们的应用模型永远不会改变，因为我们的代码中的更新部分永远不会被执行。

让我们通过添加一个按钮来纠正这个问题。

# 视图、按钮和事件

让我们从简单地给我们的应用添加一个静态按钮开始。我们将通过更新视图函数来实现这一点，以下是相应的代码：

```js
-- VIEW
view model =
    div [] 
        [ h1 [] [ text ("Fruit to eat: " ++ (toString model)) ] 
        , button [ onClick Decrement ] [ text "Eat fruit" ]
        ]   
```

如果你现在保存并运行应用，编译器会抛出以下错误：

```js
NAMING ERROR
Line 23, Column 20
Cannot find variable onClick
```

为什么我们会得到这个错误？因为我们还没有导入`onClick`函数。现在让我们通过在应用的第 3 行添加导入语句来解决这个问题。

看看我们代码的开头，更新后的前三条代码应该如下所示：

```js
module Main exposing (..)

import HTML exposing (..)
import HTML.Events exposing (onClick)
```

现在运行我们的应用会给我们一个简单、基础但功能齐全的应用，它是用 Elm 构建的！

当你点击“吃水果”按钮时，`view`函数会调用`button`函数的第一对方括号，并监听点击事件。我们为`onClick`函数提供了在按钮点击时发送的`Msg`。由于我们的应用中只有一个可能的消息，一旦点击事件被触发，`view`函数将向`update`函数发送`Decrement`消息。

一旦`update`函数收到消息，它将返回一个新的`model`，新的`model`将被`view`函数渲染。

然而，有一个问题。如果我们继续点击按钮，我们的应用最终会进入计数负数的状态，这是不可能的。用户不应该剩下负 2 个要吃的水果。

在接下来的部分，我们将修复这个问题。

# 限制减少消息

为了限制减少消息，让我们首先看看当前的更新函数：

```js
-- UPDATE
update msg model =
    model - 1
```

现在，让我们引入一个`if-else`语句来应对不同的可能情况：

```js
-- UPDATE
update msg model =
    if model > 0 then model - 1 else model == 5
```

不幸的是，前面的代码并没有产生预期的结果。相反，我们得到了以下编译器消息：

```js
TYPE MISMATCH
Line 39, Column 5
The branches of this if produce different types of values.

The then branch has type:

number
But the else branch is:

Bool
Hint: These need to match so that no matter which branch we take, we always get back the same type of value.
```

我们在 第一章 *为什么现在是学习 Elm 的好时机？* 中简要提到了 Elm 的约束。前面的问题是一个 Elm 约束在实践中的绝佳例子。由于 Elm 的设置方式，每个分支都必须返回相同的数据类型。在我们的情况下，我们可以为 `if-else` 表达式的任一分支返回布尔值，或者返回数字，但不能混合使用。

那么，我们如何纠正这个问题？为了使事情简单并仍然使用我们最初开始的 `if-else` 表达式，让我们考虑一种简洁的方式来避免类型不匹配。计数器的值永远不会低于零，所以我们只需这样做：

```js
-- UPDATE
update msg model =
    if model > 0 then model - 1 else model + 5
```

使用前面的代码，我们的 `if-else` 逻辑将始终返回一个数字。更新函数的消息值只要大于零就会增加 1。否则（如果它是零），它将增加 5。

为了总结这一部分，让我们看看我们还可以如何编写我们的代码中的主函数：

```js
main =
    HTML.beginnerProgram
        { model = 5
        , update = update
        , view = view
        }
```

在我们完成本章的这一部分之前，让我们看看完成的 *Fruit Counter* 应用程序，因为我们将在本章后面提到它：

```js
module Main exposing (..)

import HTML exposing (..)
import HTML.Events exposing (onClick)

-- main : HTML msg
main =
    HTML.beginnerProgram
        { model = 5
        , update = update
        , view = view
        }

-- MODEL
type alias Model = 
    Int

-- VIEW
view model =
    div [] 
        [ h1 [] [ text ("Fruit to eat: " ++ (toString model)) ] 
        , button [ onClick Decrement ] [ text "Eat fruit" ]
        ]   

-- MESSAGE
type Msg = 
    Decrement

-- UPDATE
update msg model =
    if model > 0 then model - 1 else model + 5
```

现在我们已经看到了一个简单应用程序的实际实现，让我们讨论一些背后的理论概念，即 Elm 中的值和类型、函数以及 `if` 表达式。我们将通过更深入地研究 Elm 消息来结束讨论。

# Elm 中的值、表达式、数据结构和类型

Elm 中没有语句。在我们本章前面看到的 `if-else` 示例中，我们使用的是表达式，而不是语句。

这种差异很重要，因为它告诉我们关于 Elm 语言行为的一些信息：表达式总是会返回一个值。实际上，Elm 中的每件事都是一个表达式，因此每件事都会返回一个值，甚至不需要显式地使用 `return` 关键字（例如，在 JavaScript 中我们必须这样做）。

那么，这个值可以是什么？值只是计算的结果。它是运行表达式的结果。换句话说，当表达式被评估时，它将产生一个值。为了测试这一点，我们可以看看 **Elm REPL**。为了使事情简单，我们将使用可在线访问的 elm-repl，网址为 [elmrepl.cuberoot.in](http://elmrepl.cuberoot.in)。

Elm REPL 是 Elm 语言的解释器。区分解释器和编译器很重要。

为什么我们在以下示例中使用 Elm REPL？我们使用它是因为这是一种直接获取我们给 REPL 的每个表达式评估到的值类型信息的方法。换句话说，我们将给 Elm REPL 提供一系列表达式，主要是简单值的形式，REPL 将返回它们的类型。我们将从评估为原始类型的表达式开始。

# Elm 中的原始类型

Elm 中的原始类型包括 `Char`、`String`、`Bool` 和 `number`（`Int` 和 `Float`）。

# 字符和字符串类型

让我们在在线 elm-repl 中输入以下内容：

```js
'a'
```

Elm-repl 会给我们返回运行前面表达式的结果，并跟随着该值的类型。值 'a' 的类型是 `Char`：

```js
'a' : Char
```

让我们再试一个，这次用双引号：

```js
"a"
```

我们得到的是这个：

```js
"a" : String
```

我们可以读作：值 `''a''` 的类型是 `String`。

显然，当我们使用单引号时，我们得到 `Chars`。

要从值中获取 `String` 类型，我们需要在该值周围加上双引号。

多行字符串通过将任意数量的行用三个连续的双引号括起来来编写。当使用 REPL 时，每一行也必须以反斜杠结尾。Elm REPL 会自动插入管道字符。输入的多行字符串值仍然会评估为 `String` 类型：

```js
""" \
| This 
| is 
| a 
| multi-line
| string
| """
" \n This \n is \n a \n multi-line  \n string \n " : String
```

让我们看看 Elm 中的其他原始类型。

# 数字类型

如果我们只输入一个数字，我们将得到相同的数字，后面跟着一个冒号和 `number` 类型：

```js
> 5
5 : number
```

如果我们测试一个十进制数，我们将得到 `Float` 类型的类型：

```js
> 3.6
3.6 : Float
```

为什么有些类型是大写的，而有些不是？

如果一个类型是大写的，这意味着它是一个显式类型。

基本上，`number` 类型用于 `Int` 和 `Float`。它最终会变成哪种类型（即最终会变成哪种显式类型），取决于这个数字是如何使用的。

换句话说，`number` 是一个隐式类型，因为它最终可以是显式的 `Int` 或显式的 `Float`。

要获取类型为 `Int` 的值，请运行以下命令：

```js
> truncate 3.14
3 : Int
```

# 布尔值

让我们看看布尔值：

```js
> True
True : Bool
> False
False : Bool
```

大写很重要！例如，输入 `true` 会引发错误：

```js
> true
-- NAMING ERROR ---------------------------------------------- repl-temp-000.elm

Cannot find variable `true`

3| true
 ^^^^
Maybe you want one of the following?

 List.take
 String.trim
```

通过这个，我们已经涵盖了 Elm 中的原始类型。接下来，我们将查看数据结构。由于 Elm 语言是静态类型的，并且 Elm 中的每件事都是表达式，因此我们可以推断出，无论我们放入 REPL 中的内容实际上都是一个表达式，其值将评估为某种类型。数据结构也是这种行为的组成部分。

# 数据结构：列表、元组、记录、集合、数组和字典

Elm 中的基本数据结构是列表、元组、记录、集合、数组和字典。在本节中，我们将使用 Elm REPL 和 Ellie-app 来查看这些数据结构的每个行为。

# 列表

Elm 中的列表类似于 JavaScript 中的数组。在我们的第一个例子中，让我们在 Elm REPL 中输入这个值：

```js
[ 1, 2, 3, 4 ]
```

这是我们从 REPL 获取到的：

```js
[1,2,3,4] : List number
```

太棒了，一个数字列表！

与浮点数进行对比：

```js
[ 0.1, 0.2, 0.3, 0.4 ]
```

REPL 的响应是：

```js
[0.1,0.2,0.3,0.4] : List Float
```

让我们尝试字符：

```js
[ 'a', 'b', 'c' ]
```

REPL 返回这个：

```js
['a','b','c'] : List Char
```

然而，在 Elm 的列表中混合值是不允许的：

```js
> [ 1, 2, 3, 'c' ]
-- TYPE MISMATCH --------------------------------------------- repl-temp-000.elm

The 3rd and 4th entries in this list are different types of values.

3| [ 1, 2, 3, 'c' ]
 ^^^
The 3rd entry has this type:

 number

But the 4th is:

 Char

Hint: Every entry in a list needs to be the same type of value. This way you
never run into unexpected values partway through. To mix different types in a
single list, create a "union type" as described in:
<http://guide.elm-lang.org/types/union_types.HTML>

```

明显地，我们无法在 Elm 列表中混合类型。换句话说，Elm 中的每个列表都是一个表达式组，这些表达式必须始终评估为相同的类型。

让我们尝试字符串：

```js
> [ "just", "a", "bunch", "of", "strings" ]
["just","a","bunch","of","strings"] : List String
```

那么，一个空列表呢？：

```js
> []
[] : List a
```

`List a` 意味着这个列表是空的，也就是说，它可以包含任何东西。这总结了我们对 Elm 中列表的简要概述。接下来，我们将查看元组。

# 元组

在 Elm 中，元组是一种可以持有各种类型值的 数据结构。

要在 Elm REPL 中创建一个元组，我们只需在括号内放入一个 `String` 和一个布尔值：

```js
( "abc", True )
```

REPL 将会响应：

```js
("abc",True) : ( String, Bool )
```

元组可以持有最多九个值。有趣的是，不同长度的元组被认为是不同类型的。例如，让我们使用 Elm REPL 创建一个包含两个元组的 List：

```js
[ ( 'a', 'b' ), ( 'c', 'd' ) ]
```

这个表达式将会计算为：

```js
[('a','b'),('c','d')] : List ( Char, Char )
```

REPL 告诉我们的是，我们放入的值是一个包含两个元组的 `List`，这些元组持有 `Char` 类型的值。

让我们尝试改变第二个元组中 Chars 的数量：

```js
[ ( 'a', 'b' ), ( 'c' ) ]
```

在 REPL 中运行前面的代码将会抛出以下错误：

```js
-- TYPE MISMATCH --------------------------------------------- repl-temp-000.elm

The 1st and 2nd entries in this list are different types of values.

3| [ ( 'a', 'b' ), ( 'c' ) ]
 ^^^
The 1st entry has this type:

 ( Char, Char )

But the 2nd is:

 Char
```

Elm 会查看我们给出的表达式，并返回类型不匹配错误。确实，为了使两个元组被认为是同一类型，它们必须持有相同数量的值，并且这些值 *也* 必须是同一类型的。

在前面的工作示例中，我们将两个包含两个 Chars 的元组添加到了一个 `List` 中，Elm REPL 返回了 `List ( Char, Char )`。

Elm 中元组能持有的最大值数是 9。如果你尝试向元组中添加 10 个或更多的值，Elm 将会抛出一个错误。让我们来试一试：

```js
('1','2','3','4','5','6','7','8','9','0')
```

我们得到的是以下错误：

```js
elm-make: Could not find `_Tuple10` when solving type constraints.
```

在接下来的章节中，我们将更深入地了解元组。现在，让我们只提一下一个常见的用例：由于元组作为数据结构可以持有各种值，因此它们作为存储计算结果的手段非常有用。

# 记录

Elm 中的记录使用花括号，并且每个值都必须有一个标签。记录也可以持有多个值。例如，我们可以在 Elm REPL 中输入以下记录：

```js
{ color="blue", quantity=17 }
```

REPL 将返回以下内容：

```js
{ color = "blue", quantity = 17 } : { color : String, quantity : number }
```

我们将在 Elm 程序中大量使用记录，因为记录允许我们在各种场景下建模数据。

在本章的代码中，我们已经使用了记录，在 *添加 beginnerProgram 函数* 部分中。让我们回顾一下我们使用的代码：

```js
-- main : HTML msg
main =
  beginnerProgram { model = 5, view = view, update = update }
```

要在 REPL 中测试代码，我们只需要使用记录，而不需要所有冗余的部分：

```js
{ model = 5, view = view, update = update }
```

REPL 将会返回以下错误：

```js
-- NAMING ERROR ---------------------------------------------- repl-temp-000.elm

Cannot find variable `update`

3| { model = 5, view = view, update = update }
 ^^^^
```

REPL 将会跟随前面的错误，并给出一个类似的错误，指出 `'update'` 变量也无法找到。为了纠正这个问题，作为一个练习，我们可以给记录中使用的变量赋值，如下所示：

```js
> view = "view info"
"view info" : String
> update = "update info"
"update info" : String
> { model = 5, view = view, update = update }
{ model = 5, view = "view info", update = "update info" }
 : { model : number, update : String, view : String }
```

我们已经将 `String` 类型的值赋给了 Elm REPL 中的 `view` 和 `update` 变量。然后我们输入了记录，REPL 返回了记录中使用的每个变量的类型。因此，在前面的例子中，`model` 是 `number` 类型，`update` 是 `String` 类型，而 `view` 也是 `String` 类型。

接下来，我们将探讨集合、数组和字典，虽然它们也需要导入，但它们同样内置在 Elm 中。

# 集合

集合是唯一值的集合。它们的唯一性由 Elm 编程语言保证。我们可以通过 `fromList` 函数实例化集合。创建一个空集合很简单：`set = Set.empty`。

让我们看看在 Elm 中创建集合的另一种方法，通过将浏览器指向：[ellie-app.com/new](http://ellie-app.com/new)。

打开的页面已经包含了一些 Elm 代码。让我们调整代码，使其看起来如下：

```js
module Main exposing (main)

import HTML exposing (HTML, text)
import Set

set = Set.fromList [1,1,1,2]

main : HTML msg
main =
    text (toString set)
```

在前面的代码中，我们在导入 `Set`（到我们命名为 `set` 的变量）之后，将评估表达式的返回值赋给了它：`Set.fromList [1,1,1,2]`。

接下来，我们将 `set` 变量传递给 `main` 函数，以将其渲染为文本节点。当然，在它被渲染出来之前，我们必须将其转换为 `String`。在 Ellie-app 中按下编译按钮后，我们应该看到以下结果：`Set.fromList [1,2]`。

当我们试图找出数据结构之间的差异时，集合非常有用。接下来，我们将探讨数组。

# 数组

Elm 中的数组是零基的，就像在 JavaScript 中一样。使用数组，我们可以根据它们的索引来处理元素。和集合一样，数组可以使用 `fromList` 函数创建。

或者，我们可以创建一个空数组，如下所示：`array = Array.empty`。仍然在 Ellie-app 中，让我们对我们的代码进行轻微的修改以测试数组：

```js
module Main exposing (main)

import HTML exposing (HTML, text)
import Array

array = Array.fromList [1,1,1,2]
array2 = Array.get 0 array

main : HTML msg
main =
    text ((toString array) ++ " " ++ (toString array2))
```

在前面的代码中，我们有一个小小的转折——我们将两个数组的连接和一个空格（都转换为 `Strings`）分组，然后对它们运行 `text` 函数，最后将表达式评估的返回值传递给主函数。

编译后的代码将显示以下结果：`Array.fromList [1,1,1,2] Just 1`。现在，让我们暂时忽略这个结果的意义，因为我们将在本书的后面部分再次回到它。接下来，我们将探讨 Elm 中的字典。

# 字典

字典也是使用 `fromList` 函数创建的。让我们回到 Ellie-app，我们的代码已按如下方式更改：

```js
module Main exposing (main)

import HTML exposing (HTML, text)
import Dict

dict = 
    Dict.fromList 
    [ ("keyOne", "valueOne")
    , ("keyTwo", "valueTwo") 
    ]

main : HTML msg
main =
    text (toString dict)
```

编译后，Ellie-app 将返回以下内容：

```js
Dict.fromList [("keyOne","valueOne"),("keyTwo","valueTwo")]
```

`Dict` 是用于存储键值对的数据结构。键必须是唯一的。要了解更多关于这个数据结构的信息，请访问以下网址：[`package.elm-lang.org/packages/elm-lang/core/latest/Dict`](http://package.elm-lang.org/packages/elm-lang/core/latest/Dict)。

接下来，我们将探讨 Elm 语言中类型如何作用于函数和 `if` 表达式。

# 函数、if 表达式和类型

让我们在 Elm REPL 中创建一个新的函数。我们将我们的函数命名为 `multiplyBy5`：

```js
multiplyBy5 num = 5 * num
```

REPL 将返回以下内容：

```js
<function> : number -> number
```

前面的行表示我们的 `multiplyBy5` 函数的类型为 `number -> number`。让我们看看处理字符串的函数将返回什么类型：

```js
appendSuffix n = n ++ "ing"
```

如我们所知，`++` 运算符是 Elm 中的连接运算符；它将两个 `Strings` 连接在一起。因此，预期 Elm REPL 将返回以下内容：

```js
<function> : String -> String
```

如我们所见，前面的函数类型为 `String -> String`。

但是，这个 `String -> String` 是什么意思？同样地，前一个例子中的 `Int -> Int` 又是什么意思？`String -> String` 简单来说就是该函数期望一个 `String` 类型的参数，并将返回一个 `String`。对于 `Int -> Int` 的例子，该函数期望一个 `Int` 类型的值，并将返回一个 `Int` 类型的值。

是时候看看 Elm 中 `if` 表达式的类型基础了。考虑以下代码片段和 REPL 给出的响应：

```js
> time = 24
24 : number
> if time < 12 then "morning" else "afternoon"
"afternoon" : String
```

在前面的代码中，我们使用变量 `time`（我们将其赋值为 `24`）运行一个 `if` 表达式。然后，我们运行比较。请注意，`if` 表达式实际上应该被称为 `if-else` 表达式，因为 `if` 表达式必须有 `else` 部分，否则在 Elm 中将无法工作。`if` 和 `else` 分支必须是同一类型。这就是为什么在前面的例子中，我们确保我们得到的任何结果都是 `String` 类型。

# 重新审视 Elm 消息

我们本章从 Elm 架构：模型、视图和更新开始。我们还提到了另一个重要组成部分：消息，**这是视图与更新进行通信的方式**。

我们使用了一个非常简单的示例应用，*水果计数器*。该应用确实很简单：我们的视图可以向更新函数发送的唯一消息是`Decrement`。我们创建的应用的简单性是我们理解架构的好方法，无需引入太多会妨碍学习的概念。

然而，现在我们已经对所有的组成部分及其如何组合有了基本的理解，我们可以讨论 Elm 架构中与消息相关的另一个复杂层次。

要做到这一点，我们可以查看官方文档中的一个完成示例。这将有两个目的：首先，它将使我们养成尽可能经常参考优秀官方文档的习惯，其次，它将为我们提供一个基准，我们最初会参考，在本书的学习过程中，随着我们学习更高级的概念，我们将逐渐超越。

要开始，请打开按钮示例的文档，网址为：[`guide.elm-lang.org/architecture/user_input/buttons.HTML`](https://guide.elm-lang.org/architecture/user_input/buttons.html)。

接下来，访问 Ellie 在线编辑器，网址为[`ellie-app.com/new`](https://ellie-app.com/new)，并将按钮示例中的代码粘贴进去：

```js
import HTML exposing (HTML, button, div, text)
import HTML.Events exposing (onClick)

main =
  HTML.beginnerProgram { model = model, view = view, update = update }

-- MODEL
type alias Model = Int
model : Model
model =
  0

-- UPDATE
type Msg = Increment | Decrement
update : Msg -> Model -> Model
update msg model =
  case msg of
    Increment ->
      model + 1

    Decrement ->
      model - 1

-- VIEW
view : Model -> HTML Msg
view model =
  div []
    [ button [ onClick Decrement ] [ text "-" ]
    , div [] [ text (toString model) ]
    , button [ onClick Increment ] [ text "+" ]
    ]
```

粘贴代码后，点击右侧面板中的编译按钮，应用将在那里编译并运行。该应用本身与我们的 *水果计数器* 非常相似，只是稍微复杂一些。让我们比较这些差异。

在 *水果计数器* 中，我们有以下消息：

```js
-- MESSAGE

type Msg = 
    Decrement
```

在按钮应用中，消息如下：

```js
-- UPDATE

type Msg = Increment | Decrement
```

首先要注意的是代码前面的注释。最初，我们使用`MESSAGE`作为注释文本。然而，在按钮示例中，他们使用`UPDATE`作为内联注释。原因是：传统上，Elm 架构被认为只由模型(Model)、视图(View)和更新(Update)组成。我们使用 Message 作为架构的独立部分只是为了更容易理解它。理解的最重要的一点是：**视图向更新函数发送消息**。从概念上讲，这两个例子是相同的。唯一的区别是现在*消息*被定义在我们应用的*更新*部分，正如它应该的那样。

第二点要注意的是，我们的消息只有`Decrement`的值。它*只能是*一个`Decrement`消息。在按钮应用中，我们有两种选择：消息可以是*要么*`Decrement`，*要么*`Increment`。

那么`type`关键字和管道字符是什么意思呢？这与一个称为**联合类型**（也称为**代数数据类型**或**标记联合**）的东西有关。

在 Elm 中，联合类型只是我们可以即时想出的自定义类型。在我们的*Fruit Counter*应用中，我们的`Msg`联合类型只有一个值：`Decrement`。在按钮应用中，`Msg`联合类型可以是两个值中的任何一个：`Increment`或`Decrement`。为了在联合类型中清楚地区分可能的值，我们使用管道字符。

让我们在 Elm REPL 中创建另一个自定义联合类型：

```js
type Vehicle = Car | Bike | Boat | Helicopter
```

要创建一个联合类型，我们首先使用`type`关键字。接下来，我们提供实际类型，`Vehicle`。我们即时创建了一个自定义类型，并将其命名为`Vehicle`！在赋值运算符（等号`=`）的右侧，我们提供`Vehicle`联合类型可以具有的值。**这些值被称为类型构造函数**，因为你可以使用它们来**构造**新的`Vehicle`实例。

让我们在 REPL 中创建一个新的`Vehicle`实例：

```js
> friendsRide = Helicopter
Helicopter : Repl.Vehicle
```

我们刚刚构建了一个新的`Vehicle`类型的实例。正如我们所见，REPL 以这条信息响应——值是`Helicopter`，其类型是`Vehicle`。

有时，用几种不同的方式解释相同的概念是有帮助的。看待联合类型的另一种方式是，它们是我们描述构造函数的方式，也就是说，定义它们。

在`update`函数中，我们有：

```js
type Msg = Increment | Decrement
```

上述代码意味着要创建一个`Msg`，必须调用`Increment`函数或`Decrement`函数。

联合类型的理论基础根植于数学逻辑，即集合论，这基本上是研究事物集合的研究。因此，我们可以将联合类型看作是任何数量的事物集合的组合。联合类型和集合论都可以相当抽象，但在这个学习阶段，我们只需说联合类型是组织 Elm 应用中消息的一种方式。

# 函数、模式匹配和情况表达式

本章的目标是构建一个简单的应用，并学习其背后的重要理论。我们通过将我们的应用与官方文档中的应用进行比较来扩展了这个目标。

在本节中，我们将查看 Buttons 应用的更新函数，并对其进行拆解，以便完全理解其功能和运作方式。

这很重要，因为我们一旦理解了 Buttons 应用中的`update`函数是如何工作的，我们就可以自信地实现一个类似的解决方案，并改进我们的*水果计数器*。

让我们从检查 Buttons 应用的`update`函数开始：

```js
update msg model =
  case msg of
    Increment ->
      model + 1

    Decrement ->
      model - 1
```

我们在这里看到了一个新的关键字：`case`。

通常，`case`语法的运作方式如下——一个变量具有某个值。根据其值，将执行特定的代码块。当值不同时，要执行的代码块也会不同。最后，在`case`表达式的末尾，有一个代码块将执行所有在`case`表达式中未指定的值。换句话说，对于任何未指定的场景，都有一个在底部的 case 块来处理它。这个`case`块被称为*通配符*，并用下划线字符`_`标记。正如我们可以在前面的例子中看到的那样，在某些情况下，不需要添加*通配符*case，因为我们已经*覆盖*了所有可能性。

Elm 的`case`表达式通过模式匹配来评估，也就是说，通过验证`case`是否符合模式。*if 表达式*和*case 表达式*非常相似。一个主要区别是`case`表达式*匹配模式*，而`if`表达式*检查真条件*，作为确定要运行哪个代码块的方式。

查看 case 表达式的语法，我们可以看到它们以`case`关键字开头，后面跟着 case 表达式的名称（在我们的例子中，是`msg`）。名称是完全任意的；我们本可以使用任何其他名称，而不仅仅是`msg`。例如：

```js
update whatever model =
  case whatever of
    Increment ->
      model + 1

    Decrement ->
      model - 1
```

正如你可以在前面的代码片段中看到的那样，`update`函数的第一个参数和`case 表达式`的名称必须相同。为了避免混淆，最好在这里坚持使用`msg`作为第一个参数，因为这是规范，你将在大多数 Elm 程序中看到这种方式的使用。

因此，在`case`关键字和 case 表达式的名称之后，我们还有一个关键字`of`。

接下来，我们列出我们的 case。代码的结构始终相同：

```js
Pattern -> Expression to evaluate
```

如果我们查看第一个 case，我们可以看到它被写成如下：

```js
Increment ->
      model + 1
```

在前面的代码片段中，要匹配的模式是`Increment`，要评估的表达式是`model + 1`。

# 改进水果计数器应用

基于我们迄今为止所学的一切，让我们改进*水果计数器*应用。以下是完整的代码：

```js
module Main exposing (..)

import HTML exposing (..)
import HTML.Events exposing (onClick)

-- main : HTML msg
main =
    HTML.beginnerProgram
        { model = 5
        , update = update
        , view = view
        }

-- MODEL
type alias Model = 
    Int

-- VIEW
view model =
    div [] 
        [ h1 [] [ text ("Fruit to eat: " ++ (toString model)) ] 
        , button [ onClick Decrement ] [ text "Eat fruit" ]
        , button [ onClick Reset ] [ text "Reset counter" ]
        ]   

-- MESSAGE
type Msg = Decrement | Reset

-- UPDATE
update msg model =
    case msg of
        Decrement -> 
            if model >= 1 then 
                model - 1
            else
                5
        Reset -> 
            5
```

让我们突出显示我们对应用所做的改进：

1.  我们在`view`函数中添加了另一个按钮。

1.  当新按钮被点击时，我们添加了一个新的消息`Reset`。

1.  我们向我们的 `Msg` 联合类型中添加了 `Reset` 构造函数。

1.  在我们的 `update` 函数中，我们给我们的模式表达式命名为 `msg`，并且我们还给它提供了两个匹配模式：`Decrement` 和 `Reset`。

1.  如果匹配到 `Decrement` 模式，将评估一个 `if` 表达式（以确定模型是否应该减一或其值应该是 `5`）。

1.  如果匹配到 `Reset` 模式，将评估一个 `5` 的表达式。

在前面的解释中，我们尽力按照本章学到的概念描述了所有的更新。理解前面的解释非常重要，因为它是我们在后续章节构建更复杂应用程序的基础。

# 在 Elm 中构建简单的 FizzBuzz 应用程序

FizzBuzz 是一种儿童文字游戏。其目的是教授数学，即除法。

游戏很简单：每个玩家从 1 开始报数。如果一个数可以被 3 整除，玩家需要报出 *Fizz* 而不是数字。此外，如果一个数可以被 5 整除，玩家需要报出 *Buzz* 而不是数字。

最后，如果一个数可以被 3 和 5 整除，玩家需要报出 *FizzBuzz*。

由于这是一个众所周知且相对简单的问题，因此它是测试程序员知识水平的好方法。

为了解决这个问题的方法，我们将引入一个新的运算符，即取模运算符 `%`。这个运算符返回除法的余数。

这种工作方式可以通过一个例子来最好地描述。打开在线 Elm REPL，并运行以下表达式：

```js
12 % 10
```

REPL 将返回：

```js
2 : Int
```

这意味着：如果我们用 `12` 除以 `10`，我们会得到 `1`，而剩下的就是 `2`。由于 `2` 小于 `10`，它不会被整除，因为在我们例子中的取模运算只会以 `10` 为增量进行除法，并返回余数。

那就是为什么我们说取模运算符返回除法的余数（也就是说，取模运算符右侧的内容决定了增量的大小）。

例如，如果我们运行这个：

```js
10 % 9
```

我们将得到以下返回：

```js
1 : Int
```

然而，如果我们运行这个：

```js
19 % 9
```

前面的表达式将评估为 `1 : Int`，因为在前面的例子中，将要计算的增量大小是 `9`。由于 `9 * 2 = 18`，除法的余数是 `1`。

如果我们将数字除以它自己会发生什么？让我们尝试数字 `3`：

```js
3 % 3
```

REPL 返回以下内容：

```js
0 : Int
```

我们得到返回值 `0`，类型为 `Int`。我们可以对 `5` 和 `15` 做同样的操作：

```js
> 5 % 5
0 : Int
> 15 % 15
0 : Int
```

再次，我们得到零，类型为 `Int`。

这在实践中意味着我们可以使用 `if-else` 表达式来检查所有除以 3 没有余数的数字。对于所有满足该条件的数字，我们将返回 *Fizz*。我们将根据游戏规则对数字 5 和 15 采用类似的方法。

这是我们创建简单的 *FizzBuzz* 应用所需的信息。将您的浏览器导航到 ;[`ellie-app.com/new`](https://ellie-app.com/new) 并输入以下代码：

```js
module Main exposing (main)

import HTML exposing (text)

fizzBuzz = "FizzBuzz"
fizz = "Fizz"
buzz = "Buzz"

fizzBuzzInput value = 
    if value % 15 == 0 then
        fizzBuzz
    else if value % 3 == 0 then
        fizz
    else if value % 5 == 0 then
        buzz
    else (toString value)

main =
    text (fizzBuzzInput 34567)
```

到目前为止，我们已经有足够的知识轻松理解前面的代码。查看前面的代码，需要提醒的一个重要概念是我们与 `if` 表达式一起工作时需要遵守的原则：它们应该始终返回相同类型的值。这就是为什么我们在 `if` 表达式的底部传递值到 `toString` 函数的原因。

作为对管道语法的快速提醒（在第一章 *Why Is This a Great Time to Learn Elm?* 中讨论），以下是另一种我们可以编写主函数的方式：

```js
main =
    fizzBuzzInput 34567
    |> text  
```

`|>` 被称为 **前向函数应用操作符**。

我们已经实现了在 Elm 中创建一个工作 *FizzBuzz* 应用的目标。在接下来的章节中，我们将探讨改进我们简单 *FizzBuzz* 应用的方法。

# 概述

在本章中，我们涵盖了多个重要主题，具体如下：

+   Elm 语法：值、类型、数据结构、`if-else` 表达式、`case` 表达式以及一些操作符

+   TEA：Elm 架构

+   单向数据流的概念

+   使用 Elm REPL 和 Ellie-app 进行工作

我们还构建了两个工作应用，虽然简单，但已经展示了我们所涵盖的理论概念的实际应用。

在下一章中，我们将使用 Elm 创建我们自己的个人投资组合。
