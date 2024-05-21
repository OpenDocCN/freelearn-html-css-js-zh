# 第五章：有效的 ML

到目前为止，我们已经学习了 Reason 的基础知识。我们已经看到，拥有健壮的类型系统可以使重构变得更加安全，减轻压力。在更改实现细节时，类型系统会有用地提醒我们需要更新代码库的其他部分。在本章中，我们将学习如何隐藏实现细节，使重构变得更加容易。通过隐藏实现细节，我们保证更改它们不会影响代码库的其他部分。

我们还将学习类型系统如何帮助我们在应用程序中强制执行业务规则。隐藏实现细节还为我们提供了一种通过保证模块不被用户滥用来强制执行业务规则的好方法。我们将通过本章中包含在本书的 GitHub 存储库中的简单代码示例来阐明这一点。

要跟着做，请从`Chapter05/app-start`开始。这些示例与我们一直在构建的应用程序隔离开来。

您可以使用以下方式转到本书的 GitHub 存储库：

```js
git clone https://github.com/PacktPublishing/ReasonML-Quick-Start-Guide.git
cd ReasonML-Quick-Start-Guide
cd Chapter05/app-start
npm install
```

记住，所有模块都是全局的，模块的所有类型和绑定默认情况下都是公开的。正如我们将很快看到的，模块签名可以用来隐藏模块的类型和/或绑定，使其对其他模块不可见。在本章中，我们还将学习高级类型系统功能，包括以下内容：

+   抽象类型

+   幻影类型

+   多态变体

# 模块签名

模块签名约束模块的方式类似于接口约束面向对象编程中的类。模块签名可以要求模块实现特定类型和绑定，还可以用于隐藏实现细节。假设我们有一个名为`Foo`的模块，在`Foo.re`中定义。它的签名可以在`Foo.rei`中定义。如果模块签名存在并且该类型或绑定不在模块签名中，则模块中列出的任何类型或绑定都将被隐藏。在`Foo.re`中有一个绑定`let foo = "foo";`，该绑定可以通过其模块签名要求和暴露，方法是在`Foo.rei`中包括`let foo: string;`：

```js
/* Foo.re */
let foo = "foo";

/* Foo.rei */
let foo: string;

/* Bar.re */
Js.log(Foo.foo);
```

在这里，`Foo.rei`要求`Foo.re`有一个名为`foo`的`string`类型的`let`绑定。

如果模块的`.rei`文件存在且为空，则模块中的所有内容都被隐藏，如下面的代码所示：

```js
/* Foo.rei */
/* this is intentionally empty */

/* Bar.re */
Js.log(Foo.foo); /* Compilation error: The value foo can't be found in Foo */
```

模块的签名要求模块包括签名中列出的任何类型和/或绑定，如下面的代码所示：

```js
/* Foo.re */
let foo = "foo";

/* Foo.rei */
let foo: string;
let bar: string;
```

这导致以下编译错误，因为模块签名要求`bar`绑定为`string`类型，而模块中未定义：

```js
The implementation src/Foo.re does not match the interface src/Foo.rei:
The value `bar' is required but not provided
```

# 模块类型

模块签名也可以使用`module type`关键字来定义，而不是使用单独的`.rei`文件。模块类型必须以大写字母开头。一旦定义，模块可以使用`module <Name> : <Type>`语法来受模块类型的约束，如下所示：

```js
module type FooT {
  let foo: (~a: int, ~b: int) => int;
};

module Foo: FooT {
  let foo = (~a, ~b) => a + b;
};
```

相同的模块类型可以用于多个模块，如下所示：

```js
module Bar: FooT {
  let bar = (~a, ~b) => a - b;
};
```

我们可以将模块签名视为面向对象意义上的接口。接口定义了模块必须定义的属性和方法。然而，在 Reason 中，模块签名还隐藏了绑定和类型。但模块签名最有用的功能之一可能是暴露抽象类型的能力。

# 抽象类型

抽象类型是没有定义的类型声明。让我们探讨一下为什么这会有用。除了绑定，模块签名还可以包括类型。在下面的代码中，您会注意到`Foo`的模块签名包括一个`person`类型，现在`Foo`必须包括这个`type`声明：

```js
/* Foo.re */
type person = {
  firstName: string,
  lastName: string
};

/* Foo.rei */
type person = {
  firstName: string,
  lastName: string
};
```

`person`类型的暴露方式与没有定义模块签名时的方式相同。正如你所期望的，如果定义了签名并且类型未列出，那么该类型不会暴露给其他模块。还有将类型保持抽象的选项。我们只保留等号后面的部分。让我们看看下面的代码：

```js
/* Foo.rei */
type person;
```

现在，`person`类型对其他模块是可见的，但没有其他模块可以直接创建或操纵`person`类型的值。`person`类型需要在`Foo`中定义，但可以有任何定义。这意味着`person`类型可以随时间改变，而`Foo`之外的模块永远不会知道这一点。

让我们在下一节进一步探讨抽象类型。

# 使用模块签名

假设我们正在构建一个发票管理系统，我们有一个`Invoice`模块，定义了一个`invoice`类型以及其他模块可以使用的函数来创建该类型的值。这种安排如下所示：

```js
/* Invoice.re */
type t = {
  name: string,
  email: string,
  date: Js.Date.t,
  total: float
};

let make = (~name, ~email, ~date, ~total) => {
  name,
  email,
  date,
  total
};
```

假设我们还有另一个模块负责向客户发送电子邮件，如下面的代码所示：

```js
/* Email.re */
let send = invoice: Invoice.t => ...
let invoice =
  Invoice.make(
    ~name="Raphael",
    ~email="persianturtle@gmail.com",
    ~date=Js.Date.make(),
    ~total=15.0,
  );
send(invoice);
```

由于`Invoice.t`类型是公开的，所以发票可以被`Email`操纵，如下面的代码所示：

```js
/* Email.re */
let invoice =
  Invoice.make(
    ~name="Raphael",
    ~email="persianturtle@gmail.com",
    ~date=Js.Date.make(),
    ~total=15.0,
  );
let invoice = {...invoice, total: invoice.total *. 0.8};
Js.log(invoice);
```

尽管`Invoice.t`类型是不可变的，但没有阻止`Email`用一些改变的字段来遮蔽发票绑定。然而，如果我们将`Invoice.t`类型设为抽象，这将是不可能的，因为`Email`将无法操纵抽象类型。`Email`模块可以访问的任何函数都无法与`Invoice.t`类型一起使用。

```js
/* Invoice.rei */
type t;
let make:
 (~name: string, ~email: string, ~date: Js.Date.t, ~total: float) => t;
```

现在，编译给我们带来了以下错误：

```js
8 │ let invoice = {...invoice, total: invoice.total *. 0.8};
9 │ Js.log(invoice);

The record field total can't be found.
```

如果我们决定允许其他模块向发票添加折扣，我们需要创建一个函数并将其包含在`Invoice`的模块签名中。假设我们只想允许每张发票只有一个折扣，并且还限制折扣金额为十、十五或二十个百分比。我们可以以以下方式实现这一点：

```js
/* Invoice.re */
type t = {
 name: string,
 email: string,
 date: Js.Date.t,
 total: float,
 isDiscounted: bool,
};

type discount =
 | Ten
 | Fifteen
 | Twenty;

let make = (~name, ~email, ~date, ~total) => {
 name,
 email,
 date,
 total,
 isDiscounted: false,
};

let discount = (~invoice, ~discount) =>
 if (invoice.isDiscounted) {
 invoice;
 } else {
 {
 ...invoice,
 isDiscounted: true,
 total:
 invoice.total
 *. (
 switch (discount) {
 | Ten => 0.9
 | Fifteen => 0.85
 | Twenty => 0.8
 }
 ),
 };
 };

/* Invoice.rei */
type t;

type discount =
 | Ten
 | Fifteen
 | Twenty;

let make:
 (~name: string, ~email: string, ~date: Js.Date.t, ~total: float) => t;

let discount: (~invoice: t, ~discount: discount) => t;

/* Email.re */
let invoice =
 Invoice.make(
 ~name="Raphael",
 ~email="persianturtle@gmail.com",
 ~date=Js.Date.make(),
 ~total=15.0,
 );
Js.log(invoice);
```

现在，只要`Invoice`模块的公共 API（或模块签名）不改变，我们就可以自由地重构`Invoice`模块，而不需要担心在其他模块中破坏代码。为了证明这一点，让我们将`Invoice.t`重构为元组而不是记录，如下面的代码所示。只要我们不改变模块签名，`Email`模块就不需要做任何改变：

```js
/* Invoice.re */
type t = (string, string, Js.Date.t, float, bool);

type discount =
  | Ten
  | Fifteen
  | Twenty;

let make = (~name, ~email, ~date, ~total) => (
  name,
  email,
  date,
  total,
  false,
);

let discount = (~invoice, ~discount) => {
  let (name, email, date, total, isDiscounted) = invoice;
  if (isDiscounted) {
    invoice;
  } else {
    (
      name,
      email,
      date,
      total
      *. (
        switch (discount) {
        | Ten => 0.9
        | Fifteen => 0.85
        | Twenty => 0.8
        }
      ),
      true,
    );
  };
};

/* Invoice.rei */
type t;

type discount =
  | Ten
  | Fifteen
  | Twenty;

let make:
  (~name: string, ~email: string, ~date: Js.Date.t, ~total: float) => t;

let discount: (~invoice: t, ~discount: discount) => t;

/* Email.re */
let invoice =
  Invoice.make(
    ~name="Raphael",
    ~email="persianturtle@gmail.com",
    ~date=Js.Date.make(),
    ~total=15.0,
  );
let invoice = Invoice.(discount(~invoice, ~discount=Ten));
Js.log(invoice);
```

另外，由于`Invoice.t`抽象类型，我们保证发票只能打折一次，并且只能按指定的百分比打折。我们可以通过要求对发票的所有更改都进行日志记录来进一步举例。传统上，这种要求会通过在数据库事务之后添加副作用来解决，因为在 JavaScript 中，我们无法确定是否会记录所有对发票的更改。使用模块签名，我们可以选择在应用层解决这些要求。

# 幻影类型

看看我们之前的实现，如果我们不必在运行时检查发票是否已经打折，那将是很好的。有没有一种方法可以在编译时检查发票是否已经打折？使用幻影类型，我们可以。

幻影类型是具有类型变量的类型，但这个类型变量在其定义中没有被使用。为了更好地理解，让我们再次看看`option`类型，如下面的代码所示：

```js
type option('a) =
  | None
  | Some('a);
```

`option`类型有一个类型变量`'a`，并且类型变量在其定义中被使用。正如我们已经学到的，`option`是一种多态类型，因为它有一个类型变量。另一方面，幻影类型在其定义中不使用类型变量。让我们看看这在我们的发票管理示例中是如何有用的。

让我们将`Invoice`模块的签名更改为使用幻影类型，如下所示：

```js
/* Invoice.rei */
type t('a);

type discounted;
type undiscounted;

type discount =
  | Ten
  | Fifteen
  | Twenty;

let make:
  (~name: string, ~email: string, ~date: Js.Date.t, ~total: float) =>
  t(undiscounted);

let discount:
  (~invoice: t(undiscounted), ~discount: discount) => t(discounted);
```

抽象类型`t`现在是`type t('a)`。我们还有两个更多的抽象类型，如下面的代码所示：

```js
type discounted;
type undiscounted;
```

还要注意，`make`函数现在返回`t(undiscounted)`（而不仅仅是`t`），`discount`函数现在接受`t(undiscounted)`并返回`t(discounted)`。记住，抽象`t('a)`接受一个`type`变量，而`type`变量恰好是`discounted`类型或`undiscounted`类型。

在实现中，我们现在可以摆脱之前的运行时检查，如下面的代码所示：

```js
if (isDiscounted) {
  ...
} else {
  ...
}
```

现在，这个检查是在编译时进行的，因为`discount`函数只接受`undiscounted`发票，如下面的代码所示：

```js
/* Invoice.re */
type t('a) = {
  name: string,
  email: string,
  date: Js.Date.t,
  total: float,
};

type discount =
  | Ten
  | Fifteen
  | Twenty;

let make = (~name, ~email, ~date, ~total) => {name, email, date, total};

let discount = (~invoice, ~discount) => {
  ...invoice,
  total:
    invoice.total
    *. (
      switch (discount) {
      | Ten => 0.9
      | Fifteen => 0.85
      | Twenty => 0.8
      }
    ),
};
```

这只是类型系统可以帮助我们更多地关注逻辑而不是错误处理的另一种方式。以前，尝试两次打折发票只会返回原始发票。现在，让我们尝试在`Email.re`中两次打折发票，使用以下代码：

```js
/* Email.re */
let invoice =
  Invoice.make(
    ~name="Raphael",
    ~email="persianturtle@gmail.com",
    ~date=Js.Date.make(),
    ~total=15.0,
  );
let invoice = Invoice.(discount(~invoice, ~discount=Ten));
let invoice = Invoice.(discount(~invoice, ~discount=Ten)); /* discounted twice */
Js.log(invoice);
```

现在，尝试两次打折发票将导致一个可爱的编译时错误，如下所示：

```js
We've found a bug for you!

   7 │ );
   8 │ let invoice = Invoice.(discount(~invoice, ~discount=Ten));
   9 │ let invoice = Invoice.(discount(~invoice, ~discount=Ten));
  10 │ Js.log(invoice);

  This has type:
    Invoice.t(Invoice.discounted)
  But somewhere wanted:
    Invoice.t(Invoice.undiscounted)
```

这绝对美丽。然而，假设你想能够给任何发票发送电子邮件，无论是否打折。我们使用幻影类型会导致问题吗？我们如何编写一个接受任何发票类型的函数？我们的发票类型是`Invoice.t('a)`，如果我们想接受任何发票，我们保留类型参数，如下面的代码所示：

```js
/* Email.re */
let invoice =
  Invoice.make(
    ~name="Raphael",
    ~email="persianturtle@gmail.com",
    ~date=Js.Date.make(),
    ~total=15.0,
  );

let send: Invoice.t('a) => unit = invoice => {
 /* send invoice email */
 Js.log(invoice);
};

send(invoice);
```

所以我们可以两全其美。

# 多态变体

我们已经在上一章简要地看过多态变体。简而言之，我们在使用`[@bs.unwrap]`装饰器绑定到一些现有的 JavaScript 时学到了它们。这个想法是`[@bs.unwrap]`可以用于绑定到现有的 JavaScript 函数，其中它的参数可以是不同的类型。例如，假设我们想绑定到以下函数：

```js
function dynamic(a) {
  switch (typeof a) {
    case "string":
      return "String: " + a;
    case "number":
      return "Number: " + a;
  }
}
```

假设这个函数只接受`string`类型或`int`类型的参数，不接受其他类型。我们可以这样绑定这个示例函数：

```js
[@bs.val] external dynamic : 'a => string = "";
```

然而，我们的绑定将允许无效的参数类型（如`bool`）。如果我们的编译器能够通过阻止无效的参数类型来帮助我们，那将更好。其中一种方法是使用多态变体与`[@bs.unwrap]`。我们的绑定将如下所示：

```js
[@bs.val] external dynamic : ([@bs.unwrap] [
  | `Str(string)
  | `Int(int)
]) => string = "";
```

我们会这样使用绑定：

```js
dynamic(`Int(42));
dynamic(`Str("foo"));
```

现在，如果我们尝试传递无效的参数类型，编译器会让我们知道，如下面的代码所示：

```js
dynamic(42);

/*
We've found a bug for you!

This has type:
  int
But somewhere wanted:
  [ `Int of int | `Str of string ]
*/
```

这里的折衷是我们需要通过将参数包装在多态变体构造函数中而不是直接传递参数。

一开始，你会注意到普通变体和多态变体之间的以下两个不同之处：

1.  我们不需要显式声明多态变体的类型

1.  多态变体以反引号字符（`` ` ``）

每当您看到一个以反勾号字符为前缀的构造函数时，您就知道它是一个多态变体构造函数。可能有也可能没有与多态变体构造函数相关联的类型声明。

# 这对正常变体有效吗？

让我们试着用普通变体来做这件事，看看会发生什么:

```js
type validArgs = 
  | Int(int)
  | Str(string);

[@bs.val] external dynamic : validArgs => string = "";

dynamic(Int(1));
```

前面实现的问题是`Int(1)`不会编译为 JavaScript 数字。普通变体编译为`array`，我们的`dynamic`函数返回`undefined`而不是`"Number: 42"`。函数返回`undefined`是因为在 switch 语句上没有匹配到任何情况。

使用多态变体，BuckleScript 将`dynamic(`Int(42))`编译为`dynamic(42)`，函数按预期工作。

# 高级类型系统特性

Reason 的类型系统非常全面，并在过去的几十年中得到了完善。到目前为止，我们所看到的只是对 Reason 类型系统的介绍。在我看来，你应该在继续学习更高级的类型系统功能之前熟悉基础知识。没有经历过合理的类型系统本应阻止的错误，很难欣赏诸如类型安全之类的东西。没有对到目前为止在本书中学到的内容感到略微沮丧，很难欣赏高级类型系统功能。本书的范围不包括对高级类型系统功能进行过多详细讨论，但我想确保那些正在评估 Reason 作为一个选项的人知道它的类型系统还有更多内容。

除了幻影类型和多态变体之外，Reason 还具有**广义代数数据类型**（**GADTs**）。模块可以使用函数器（即，在编译时和运行时之间操作的模块函数）动态创建。Reason 还具有类和对象——OCaml 中的 O 代表 objective。OCaml 的前身是一种称为 Caml 的语言，最早出现在 20 世纪 80 年代中期。到目前为止，在本书中学到的东西在典型的 React 应用程序的上下文中特别有用。就我个人而言，我喜欢 Reason 是一种我可以在其中不断成长并保持高效的语言。

如果你发现自己对类型系统感到沮丧，那么可以在 Discord 频道上寻求专家的帮助，有人很可能会帮助你解决问题。我对社区的乐于助人感到不断惊讶。而且不要忘记，如果你只是想继续前进，你总是可以转到原始的 JavaScript，如果需要的话，等你准备好了再回来解决问题。

你可以在这里找到 Reason 的 Discord 频道：

[`discord.gg/reasonml`](https://discord.gg/reasonml)

不使用 Reason 类型系统的更高级功能也是完全有效的。到目前为止，我们所学到的内容在为我们的 React 应用程序添加类型安全方面提供了很大的价值。

# 总结

到目前为止，我们已经看到 Reason 如何帮助我们使用其类型系统构建更安全、更易维护的代码库。变体允许我们使无效状态不可表示。类型系统有助于使重构过程变得不那么可怕、不那么痛苦。模块签名可以帮助我们强制执行应用程序中的业务规则。模块签名还可以作为基本文档，列出模块公开的内容，并根据公开的函数名称和其参数类型以及公开的类型，给出模块的基本使用方式的概念。

在第六章中，*CSS-in-JS（在 Reason 中）*，我们将看看如何使用 Reason 的类型系统来强制执行有效的 CSS，使用一个包装 Emotion（[`emotion.sh`](https://emotion.sh)）的 CSS-in-Reason 库，名为`bs-css`。
