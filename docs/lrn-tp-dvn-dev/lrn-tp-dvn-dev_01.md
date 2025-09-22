# 第一章：开始类型驱动开发

在这本书中，我们正在探索类型驱动开发中可用的技术和惯例。有些人也将类型驱动开发称为类型级编程。静态类型提供了几个好处，包括：

+   阻止错误代码有机会运行

+   记录当前的代码库

+   通过指出你可能遗漏的代码部分来帮助正确重构代码库

+   提供更丰富的 IDE 支持，例如自动完成

+   当编译器知道类型并相应地优化代码时，性能更好

类型驱动开发是使用静态类型来限制代码可以做什么的实践。通常，你的编程语言给你足够的权力来表示任何计算。通过类型驱动开发，你本质上是在尝试让你的代码不可能做你不希望做的事情。

在本章中，我们将对一段代码进行一些基本的批判性分析，并查看它可能包含的错误。我们还将介绍 ReasonML，这是我们用来学习类型驱动开发的语言，并将其与 JavaScript 进行比较。我们将从一个基本的 Reason 项目开始，然后介绍 Reason 以及其相关的社区和生态系统。

在本章中，我们将涵盖以下主题：

+   类型驱动开发的主要思想和好处

+   动态类型代码与其静态类型 ReasonML 等效物

+   Reason 语言、生态系统和相关项目

+   如何设置一个基本的 Reason 项目，我们将在此书中使用它

+   在线尝试 Reason 游戏场

# 分析代码中的隐藏错误

假设你有以下 JavaScript 代码：

```js
// src/Ch01/Ch01_Demo.js
function *makePerson*(*id*, *name*) { return {*id*, *name*}; }
```

前面的代码可能会出现很多问题；它们如下所示：

+   调用者可以将 null 或 undefined 值作为参数传递

+   调用者可以传递非预期的参数类型

+   调用者可以随意操作返回的 `person` 对象，例如，他们可以添加或删除属性

换句话说，这段代码并不能阻止许多潜在的错误。在 JavaScript 中，我们有像 ESLint ([`eslint.org/`](https://eslint.org/)) 这样的 linter，它可以检查许多可能出现的错误，但你必须记得去找到它们，启用它们，然后解决它们的限制。linter 可以以各种其他方式提供帮助，例如指出编码风格中的推荐最佳实践。然而，JavaScript 中的 linter 往往被重新用于执行静态类型检查任务；因为它们提供了很大的灵活性并且需要配置（实际上，人们通常上传他们为不同编程风格首选的配置集），因此在不同代码库之间可能存在很大的差异。

# 添加类型

使用静态类型系统，我们可以以多种方式限制我们的 `makePerson` 函数。以下是一个使用 ReasonML 的例子，这是我们在这本书中用来学习类型驱动开发的语言：

```js
/* src/Ch01/Ch01_Demo.re */
type person = {*id*: int, *name*: string};
let *makePerson*(*id*, *name*) = {*id*, *name*};
```

在这里，我们定义一个新的数据类型`person`和一个根据所需参数创建该类型值的函数。与前面的 JavaScript 代码相比，我们多了一行代码，但作为交换，我们得到了以下保证：

+   函数的调用者不能传入 null 或 undefined 参数

+   函数的调用者不能传入错误的参数类型

+   函数的调用者不能修改函数的结果值

注意在先前的例子中，我们不需要为`makePerson`函数声明参数或类型。这是因为 ReasonML 具有出色的类型推断，可以自动理解`int`、`string`和`person`必须是函数这些部分允许的唯一可能类型。

ReasonML 将前面的代码编译成以下 JavaScript：

```js
// src/Ch01/Ch01_Demo.bs.js
function *makePerson*(*id*, *name*) { return [*id*, *name*]; }
```

如您所见，前面的代码几乎与之前我们编写的 JavaScript 完全一样——主要区别在于 Reason 的 JavaScript 编译器将记录（我们将在后面探讨）转换为 JavaScript 数组，以利用它们的速度。

这只是静态类型对代码库能做的事情的一个缩影。在接下来的章节中，我们将探讨更多实际的应用。

# ReasonML

我们将探索使用 ReasonML（[`reasonml.github.io/`](https://reasonml.github.io/)）进行类型驱动开发。Reason 是一种类似于 JavaScript 的语法，同时也是一套用于 OCaml（[`ocaml.org/`](https://ocaml.org/)）的工具。OCaml 是一种成熟的静态类型函数式编程语言，它对面向对象和模块化编程提供了出色的支持。

我们将使用 BuckleScript 编译器（[`bucklescript.github.io/`](https://bucklescript.github.io/)）编写 Reason 代码并将其编译成 JavaScript。BuckleScript 从 Reason 代码中获取输入，并输出基本上是 ES5 的一个简单子集（即没有 ES2015 风格的类，没有箭头函数等）。这将使我们能够编写强静态类型代码，并看到所有类型被剥离后的输出 JavaScript 的样子。

BuckleScript 默认输出扩展名为`.bs.js`的 JavaScript 文件，以区分您的其他 JS 文件。您可以在示例输出文件中看到这一点，`src/Ch01/Ch01_Demo.bs.js`。

Reason 工具包目前包括：

+   代码格式化和语法转换工具，`refmt`

+   交互式代码评估环境，`rtop`

+   用于原生编译项目的构建管理器（我们在这本书中不需要这个），`rebuild`

+   为编辑器提供智能感知能力的工具，`ocamlmerlin-reason`

这些工具协同工作，提供了一种最小化但强大的开发体验。与一个好的编辑器（我们推荐 Visual Studio Code）一起，它们覆盖了您日常开发的大部分需求。

# 为什么选择 ReasonML？

那么，我们为什么选择 ReasonML 而不是其他语言呢？例如，TypeScript 和 Flow 是目前针对 JavaScript（以及其他许多语言）的流行语言，但我们选择 Reason 是因为：

+   它有一个强大而优雅的类型系统，它巧妙地结合了许多类型驱动开发概念

+   它的 JavaScript 编译器（BuckleScript）具有极快的编译、优化和高质量的死代码消除；如果你正在进行类型驱动开发，快速编译是非常有用的，而在任何系统中，性能良好的代码都是非常受欢迎的

+   它有一个非常有帮助且热情的社区，非常易于接触

+   它为你提供了访问成熟的 OCaml 社区和其聚合的知识库

我们将利用两种语言之间的差异来理解静态类型 Reason 代码是如何转换为动态类型 JavaScript 代码，同时仍然通过设计安全地运行。

# ReasonML 入门

Reason 网站还有一个很好的快速入门指南以及设置编辑器支持的教程。首先，安装 NodeJS 以获取 **node 包管理器**（**npm**）。然后，运行以下命令：

```js
npm install -g bs-platform
cd <your-projects-folder>
bsb –init learning-tydd-reason –theme basic-reason
cd learning-tydd-reason
```

现在，我们可以使用以下命令进行初始编译：

```js
bsb -make-world
```

上述命令递归地构建你的整个项目和其依赖项。它将几乎是瞬间的。

值得注意的是，我们实际上推荐运行前面的 shell 命令（当然，替换为你的实际项目文件夹），因为在这本书中，我们将把代码示例组织成单个项目，`learning-tydd-reason`，而你输入到各种给定文件名中的代码示例将组合在一起形成该项目。

你几乎肯定想在 Reason 中设置编辑器支持，以便你可以获得诸如自动完成和转到定义等功能。ReasonML 网站上可用的指南（[`reasonml.github.io/docs/en/global-installation.html`](https://reasonml.github.io/docs/en/global-installation.html)）对此非常有帮助。目前，Visual Studio Code ([`code.visualstudio.com/`](http://code.visualstudio.com/)) 是支持最好的编辑器；你可能会从使用它中获得最佳结果。

如果你正在尝试决定安装方法，我们个人推荐 OPAM 方法（**OPAM** 是 **OCaml 包管理器**的缩写）。

# 使用 Try Reason

Reason 为学习者提供了一个极好的资源：一个在线 Reason 到 JavaScript 编译器和评估器。要访问它，请访问 Reason 网站，并在顶部导航栏中点击 Try。你可以用它来快速尝试不同的想法。

让我们通过一个快速示例来使用 Try Reason 了解我们的方向。将 `src/Ch01/Ch01_Demo.re` 中的示例代码输入到 Try Reason 网页应用中的 Reason 部分。现在在那之后添加以下行：

```js
let *bob* = *makePerson*(1, "Bob");
```

现在如果你检查输出的 JS，你应该能看到以下更改：

+   类型已被移除

+   记录已被转换为没有字段名称的数组（记录大致类似于 C 结构体或 JavaScript 对象）

+   每个声明的值都明确导出（公开）

注意，我们在这章中故意引入了非常少的实际 Reason 语法。如果你对探索语法（其核心与 JavaScript 非常相似）感兴趣，最好的方式是探索出色的 Reason 网站文档。由于本书的重点是类型驱动开发，在接下来的章节中，我们将介绍我们将需要的所有语法，并讨论它对我们理解代码的影响。

# 深入学习

ReasonML 社区是一个有帮助的、快速发展的社区。如果你需要任何帮助，不要害怕提问。你只会初学者一次，一旦你感到舒适，你就能帮助其他初学者。请访问社区页面 [`reasonml.github.io/docs/en/community.html`](https://reasonml.github.io/docs/en/community.html)，并在 discord 聊天中作为首次接触点。

# 摘要

在本章中，我们介绍了类型驱动开发的初步概念，并批判性地分析了一块动态类型代码，以探索其潜在的错误条件，这些错误条件可以通过添加静态类型来防止。我们还介绍了 ReasonML 语言及其生态系统，设置了我们的 Reason 项目，并瞥见了它如何将静态类型代码编译成 JavaScript。

下一章将非常重要——我们将更深入地探讨类型、值和在 Reason 中的工作方式。那里见！
