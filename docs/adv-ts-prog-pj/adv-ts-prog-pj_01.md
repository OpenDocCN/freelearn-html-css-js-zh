# 第一章：高级 TypeScript 功能

在本章中，我们将研究 TypeScript 的一些方面，超越了语言的基础知识。当适当使用时，这些功能提供了一种清晰直观的方式来使用 TypeScript，并将帮助您编写专业水平的代码。我们在这里涵盖的一些内容可能对您来说并不新鲜，但我包括它们是为了确保我们在后面的章节中有一个共同的知识基础，以及为什么我们将使用这些功能的理解。我们还将介绍为什么我们需要这些技术；仅仅知道如何应用某些东西是不够的，我们还需要知道在什么情况下应该使用它们以及在这样做时需要考虑什么。本章的重点不是创建一个枯燥的、详尽的功能列表，而是要介绍我们在本书的其余部分需要的信息。这些都是我们在日常开发中一遍又一遍应用的实用技术。

由于这是一本关于 Web 开发的书，我们还将创建许多 UI，因此我们将看看如何使用流行的 Bootstrap 框架创建吸引人的界面。

本章将涵盖以下主题：

+   使用联合类型的不同类型

+   使用交集类型组合类型

+   使用类型别名简化类型声明

+   使用 REST 属性解构对象

+   使用 REST 处理可变数量的参数

+   使用装饰器进行**面向方面的编程** (**AOP**)

+   使用混合类型组合类型

+   使用相同的代码和不同的类型，并使用泛型

+   使用映射映射值

+   使用承诺和 async/await 创建异步代码

+   使用 Bootstrap 创建 UI

# 技术要求

为了完成本章，您需要安装 Node.js。您可以从[`nodejs.org/en/`](https://nodejs.org/en/)下载并安装 Node.js。

您还需要安装 TypeScript 编译器。有两种方法可以通过 Node.js 使用**Node Package Manager** (**NPM**)来完成这个任务。如果您希望所有应用程序都使用相同版本的 TypeScript，并且确信它们在更新时都能运行在相同的版本上，请使用以下命令：

```ts
npm install -g typescript
```

如果您希望 TypeScript 的版本局限于特定项目，请在项目文件夹中输入以下内容：

```ts
npm install typescript --save-dev
```

对于代码编辑器，您可以使用任何合适的编辑器，甚至是基本的文本编辑器。在本书中，我将使用 Visual Studio Code，这是一个免费的跨平台**集成开发环境** (**IDE**)，可在[`code.visualstudio.com/`](https://code.visualstudio.com/)上获得。

所有代码都可以在 GitHub 上找到[`github.com/PacktPublishing/Advanced-TypeScript-3-Programming-Projects/tree/master/Chapter01`](https://github.com/PacktPublishing/Advanced-TypeScript-3-Programming-Projects/tree/master/Chapter01)。

# 使用 tsconfig 构建未来的 TypeScript

随着 TypeScript 的流行，它受益于快速发展的开源架构。原始实现背后的设计目标意味着它已经成为开发人员的热门选择，无论是对于新手 JavaScript 开发者还是经验丰富的专业人士。这种流行意味着该语言迅速获得了新功能，有些功能简单直接，而其他功能则面向那些在 JavaScript 生态系统的前沿工作的开发人员。本章旨在介绍 TypeScript 引入的功能，以匹配当前或即将到来的 ECMAScript 实现，这些功能您可能之前没有遇到过。

随着我们在本章的进展，我会不时地指出需要较新 ECMAScript 标准的功能。在某些情况下，TypeScript 已经提供了一个与较早版本的 ECMAScript 兼容的功能的 poly-filled 实现。在其他情况下，我们编译的版本将具有一个功能，该功能无法在某一点之后进行回填，因此值得使用更更新的设置。

虽然可以完全使用命令行编译 TypeScript，但我更喜欢使用`tsconfig.json`。您可以手动创建此文件，也可以使用以下命令从命令行让 TypeScript 为您创建它：

```ts
tsc --init
```

如果您想复制我的设置，这些是我默认设置的设置。当我们需要更新引用时，我会指出需要添加的条目：

```ts
{
  "compilerOptions": {
    "target": "ES2015",
    "module": "commonjs",
    "lib": [ "ES2015", "dom" ],
    "sourceMap": true,
    "outDir": "./script", 
    "strict": true, 
    "strictNullChecks": true, 
    "strictFunctionTypes": true, 
    "noImplicitThis": true, 
    "alwaysStrict": true, 
    "noImplicitReturns": true, 
    "noFallthroughCasesInSwitch": true,
    "esModuleInterop": true,
    "experimentalDecorators": true, 
  }
}
```

# 介绍高级 TypeScript 功能

随着每个版本的发布，TypeScript 不断迈出重要的步伐，增加了功能和能力，这些功能和能力是建立在语言基础之上的，这些语言基础是在 1 版本中引入的。从那时起，JavaScript 已经发展，TypeScript 已经添加了一些功能，以便针对新兴标准，提供对旧版 JavaScript 的实现，或者在针对更新的 ECMA 标准时调用本地实现。在本章中，我们将看一些这些功能，这些功能将贯穿本书的整个内容。

# 使用联合类型与不同类型

我们要看的第一个功能是我最喜欢的功能之一，即使用联合类型的能力。当函数期望单个参数是一种类型或另一种类型时，就会使用这些类型。例如，假设我们有一个验证例程，需要检查值是否在特定范围内，这个验证可以从文本框中接收`string`值，也可以从计算中接收`number`值。由于解决这个问题的每种技术都有很多共同之处，我们将从一个简单的类开始，这个类允许我们指定形成我们范围的最小值和最大值，并且有一个实际执行验证的函数，如下所示：

```ts
class RangeValidationBase {
     constructor(private start : number, private end : number) { }
     protected RangeCheck(value : number) : boolean {
         return value >= this.start && value <= this.end;
     }
     protected GetNumber(value : string) : number {
        return new Number(value).valueOf();
     }
 }
```

如果您以前没有见过那样的`constructor`，那就相当于编写以下内容：

```ts
 private start : number = 0;
 private end : number = 0;
 constructor(start : number, end : number) {
     this.start = start;
     this.end = end;
 }
```

如果您需要检查参数或以某种方式操纵它们，您应该使用参数的扩展格式。如果您只是将值分配给私有字段，那么第一种格式是一种非常优雅的方式，可以节省代码的混乱。

有几种方法可以解决确保我们只使用`string`或`number`进行验证的问题。我们可以通过提供两个接受相关类型的单独方法来解决这个问题，如下所示：

```ts
class SeparateTypeRangeValidation extends RangeValidationBase {
     IsInRangeString(value : string) : boolean {
         return this.RangeCheck(this.GetNumber(value));
     }
     IsInRangeNumber(value : number) : boolean {
         return this.RangeCheck(value);
     }
 }
```

虽然这种技术可以工作，但它并不是非常优雅，而且它肯定没有充分利用 TypeScript 的强大功能。我们可以使用的第二种技术是允许我们传入值而不加以限制，如下所示：

```ts
class AnyRangeValidation extends RangeValidationBase {
     IsInRange(value : any) : boolean {
         if (typeof value === "number") {
             return this.RangeCheck(value);
         } else if (typeof value === "string") {
             return this.RangeCheck(this.GetNumber(value));
         }
         return false;
     }
 }
```

这绝对是对我们原始实现的改进，因为我们已经确定了函数的一个签名，这意味着调用代码更加一致。不幸的是，我们仍然可以将无效类型传递给方法，因此，如果我们传递`boolean`，这段代码将成功编译，但在运行时会失败。

如果我们想要限制我们的验证只接受字符串或数字，那么我们可以使用联合类型。它与上一个实现并没有太大的不同，但它确实给了我们编译时类型安全性，这正是我们想要的，如下所示：

```ts
class UnionRangeValidation extends RangeValidationBase {
     IsInRange(value : string | number) : boolean {
         if (typeof value === "number") {
             return this.RangeCheck(value);
         }
         return this.RangeCheck(this.GetNumber(value));
     }
 }
```

标识类型约束为联合的签名是函数名称中的`type | type`。这告诉编译器（和我们）这种方法的有效类型是什么。因为我们已经限制了输入为`number`或`string`，所以一旦我们排除了类型不是`number`，我们就不需要检查`typeof`来查看它是否是`string`，所以我们甚至进一步简化了代码。

我们可以在联合语句中链接尽可能多的类型。实际上没有实际限制，但我们必须确保联合列表中的每种类型都需要相应的`typeof`检查，如果我们要正确处理它。类型的顺序也不重要，所以`number | string`与`string | number`是相同的。但要记住的是，如果函数将许多类型组合在一起，那么它可能做得太多了，应该查看代码，看看是否可以将其分解成更小的部分。

我们可以进一步使用联合类型。在 TypeScript 中，我们有两种特殊类型，`null`和`undefined`。除非我们使用`-strictNullChecks`选项编译我们的代码，或者如果我们在`tsconfig.json`文件中将其设置为`strictNullChecks = true`，否则这些类型可以分配给任何东西。我喜欢设置这个值，这样我的代码只处理应该处理的空值情况，这是防止副作用潜入的好方法，只是因为一个函数接收了一个空值。如果我们想允许`null`（或`undefined`），我们只需要将它们添加为联合类型。

# 使用交集类型组合类型

有时，对我们来说很重要的是，我们有能力处理一种情况，即我们可以将多种类型合并在一起，并将它们视为一种类型。交集类型是正在合并的每种类型中都可用的所有属性的类型。我们可以通过以下简单的示例看到交集的样子。首先，我们将为`Grid`和`Margin`创建类，如下所示：

```ts
class Grid {
     Width : number = 0;
     Height : number = 0;
 }
 class Margin {
     Left : number = 0;
     Top : number = 0;
 }
```

我们要创建的是一个交集，最终会得到`Grid`属性的`Width`和`Height`，以及`Margin`的`Left`和`Top`。为此，我们将创建一个函数，该函数接受`Grid`和`Margin`，并返回一个包含所有这些属性的类型，如下所示：

```ts
function ConsolidatedGrid(grid : Grid, margin : Margin) : Grid & Margin {
     let consolidatedGrid = <Grid & Margin>{};
     consolidatedGrid.Width = grid.Width;
     consolidatedGrid.Height = grid.Height;
     consolidatedGrid.Left = margin.Left;
     consolidatedGrid.Top = margin.Top;
     return consolidatedGrid;
 }
```

请注意，当我们在本章后面查看对象扩展时，我们将回到这个函数，看看如何消除大量属性的样板复制。

使这项工作的*魔法*是我们如何定义`consolidatedGrid`。我们使用`&`来连接我们想要使用的类型，以创建我们的交集。因为我们想要将`Grid`和`Margin`合并在一起，所以我们使用`<Grid & Margin>`来告诉编译器我们的类型将是什么样子。我们可以看到，我们不必明确命名这种类型；编译器足够聪明，可以为我们处理这个问题。

如果我们在两种类型中都有相同的属性，会发生什么？TypeScript 是否会阻止我们混合这些类型？只要属性是相同类型，TypeScript 就可以完全允许我们使用相同的属性名称。为了看到这一点，我们将扩展我们的`Margin`类，以包括`Width`和`Height`属性，如下所示：

```ts
class Margin {
     Left : number = 0;
     Top : number = 0;
     Width : number = 10;
     Height : number = 20;
 }
```

我们如何处理这些额外的属性取决于我们想要做什么。在我们的示例中，我们将`Margin`的`Width`和`Height`添加到`Grid`的`Width`和`Height`中。这样，我们的函数看起来像这样：

```ts
function ConsolidatedGrid(grid : Grid, margin : Margin) : Grid & Margin {
     let consolidatedGrid = <Grid & Margin>{};
     consolidatedGrid.Width = grid.Width + margin.Width;
     consolidatedGrid.Height = grid.Height + margin.Height;
     consolidatedGrid.Left = margin.Left;
     consolidatedGrid.Top = margin.Top;
     return consolidatedGrid;
 }
```

然而，如果我们想要尝试并重用相同的属性名称，但这些属性的类型不同，如果这些类型有限制，我们可能会遇到问题。为了看到这种影响，我们将扩展我们的`Grid`和`Margin`类以包括`Weight`。我们的`Grid`类中的`Weight`是一个数字，而我们的`Margin`类中的`Weight`是一个字符串，如下所示：

```ts
class Grid {
     Width : number = 0;
     Height : number = 0;
     Weight : number = 0;
 }
 class Margin {
     Left : number = 0;
     Top : number = 0;
     Width : number = 10;
     Height : number = 20;
     Weight : string = "1";
 }
```

我们将尝试在我们的`ConsolidatedGrid`函数中将`Weight`类型相加：

```ts
consolidatedGrid.Weight = grid.Weight + new          
    Number(margin.Weight).valueOf();
```

此时，TypeScript 会对这行代码进行以下错误提示：

```ts
error TS2322: Type 'number' is not assignable to type 'number & string'.
   Type 'number' is not assignable to type 'string'.
```

虽然有解决这个问题的方法，比如在`Grid`中使用联合类型来解析输入的`Weight`，但通常不值得那么麻烦。如果类型不同，这通常是属性行为不同的一个很好的指示，所以我们真的应该考虑给它取一个不同的名字。

虽然我们在这里的示例中使用类，但值得指出的是，交集不仅限于类。交集也适用于接口、泛型和原始类型。

在处理交集时，还有一些其他规则需要考虑。如果我们有相同的属性名称，但只有一个属性是可选的，那么最终的属性将是必需的。我们将在`Grid`和`Margin`类中引入一个`padding`属性，并在`Margin`中将`Padding`设为可选，如下所示：

```ts
class Grid {
     Width : number = 0;
     Height : number = 0;
     Padding : number;
 }
 class Margin {
     Left : number = 0;
     Top : number = 0;
     Width : number = 10;
     Height : number = 20;
     Padding?: number;
 }
```

因为我们提供了一个强制的`Padding`变量，我们不能改变我们的交集，如下所示：

```ts
consolidatedGrid.Padding = margin.Padding;
```

由于不能保证边距填充会被分配，编译器会尽力阻止我们。为了解决这个问题，我们将改变我们的代码，如果设置了`margin`填充，则应用`margin`填充，如果没有，则回退到`grid`填充。为了做到这一点，我们将做一个简单的修复：

```ts
consolidatedGrid.Padding = margin.Padding ? margin.Padding : grid.Padding;
```

这种看起来奇怪的语法被称为三元运算符。这是一种简写的方式，相当于写成以下形式——如果`margin.Padding`有值，则让`consolidatedGrid.Padding`等于该值；否则，让它等于`grid.Padding`。这本可以写成 if/else 语句，但是，由于这是 TypeScript 和 JavaScript 等语言中的常见范例，值得熟悉。

# 使用类型别名简化类型声明

与交集类型和联合类型相辅相成的是类型别名。TypeScript 允许我们创建一个方便的别名，而不是在代码中引用`string | number | null`，这个别名会被编译器展开成相关的代码。

假设我们想创建一个代表`string | number`联合类型的类型别名，那么我们可以创建一个如下所示的别名：

```ts
type StringOrNumber = string | number;
```

如果我们重新审视我们的范围验证示例，我们可以更改函数的签名以使用这个别名，如下所示：

```ts
class UnionRangeValidationWithTypeAlias extends RangeValidationBase {
     IsInRange(value : StringOrNumber) : boolean {
         if (typeof value === "number") {
             return this.RangeCheck(value);
         }
         return this.RangeCheck(this.GetNumber(value));
     }
 }
```

在这段代码中需要注意的重要事情是，我们并没有真正创建任何新类型。类型别名只是一个语法技巧，我们可以用它来使我们的代码更易读，更重要的是，帮助我们创建更一致的代码，尤其是在大型团队中工作时。

我们还可以将类型别名与类型结合起来创建更复杂的类型别名。如果我们想要为之前的类型别名添加`null`支持，我们可以添加这个类型：

```ts
type NullableStringOrNumber = StringOrNumber | null;
```

由于编译器仍然看到了底层类型并使用它，我们可以使用以下语法来调用我们的`IsInRange`方法：

```ts
let total : string | number = 10;
if (new UnionRangeValidationWithTypeAlias(0,100).IsInRange(total)) {
    console.log(`This value is in range`);
}
```

显然，这样做不会给我们带来非常一致的代码，所以我们可以将`string | number`改为`StringOrNumber`。

# 使用对象展开分配属性

在*交集类型*部分的`ConsolidatedGrid`示例中，我们分别将每个属性分配给了我们的交集。根据我们试图实现的效果，我们还可以用另一种方式用更少的代码创建我们的`<Grid & Margin>`交集类型。使用展开运算符，我们可以自动从一个或多个输入类型中复制属性的浅层副本。

首先，让我们看看如何重写之前的例子，以便自动填充边距信息：

```ts
function ConsolidatedGrid(grid : Grid, margin : Margin) : Grid  & Margin {
    let consolidatedGrid = <Grid & Margin>{...margin};
    consolidatedGrid.Width += grid.Width;
    consolidatedGrid.Height += grid.Height;
    consolidatedGrid.Padding = margin.Padding ? margin.Padding : 
    grid.Padding;
    return consolidatedGrid;
}
```

当我们实例化我们的`consolidatedGrid`函数时，这段代码会复制`margin`的属性并填充它们。三个点(`...`)告诉编译器将其视为展开操作。由于我们已经填充了`Width`和`Height`，我们使用`+=`来简单地添加网格中的元素。

如果我们想要同时应用`grid`和`margin`的值呢？为了做到这一点，我们可以将我们的实例化更改为如下所示：

```ts
let consolidatedGrid = <Grid & Margin>{…grid, ...margin};
```

这将`Grid`的值填充到`grid`的值中，然后将`Margin`的值填充到`margin`的值中。这告诉我们两件事。第一，扩展操作将适当的属性映射到适当的属性。第二，这告诉我们它执行的顺序很重要。由于`margin`和`grid`都具有相同的属性，`grid`设置的值将被`margin`设置的值覆盖。为了设置属性，以便我们在`Width`和`Height`中看到`grid`的值，我们必须颠倒这行的顺序。当然，实际上，我们可以看到效果如下：

```ts
let consolidatedGrid = <Grid & Margin>{...margin, …grid };
```

在这个阶段，我们应该真正看一下 TypeScript 从中产生的 JavaScript。当我们使用 ES5 编译它时，代码看起来像这样：

```ts
var __assign = (this && this.__assign) || function () {
    __assign = Object.assign || function(t) {
        for (var s, i = 1, n = arguments.length; i < n; i++) {
            s = arguments[i];
            for (var p in s) if (Object.prototype.hasOwnProperty.call(s,
            p))
                t[p] = s[p];
        }
        return t;
    };
    return __assign.apply(this, arguments);
};
function ConsolidatedGrid(grid, margin) {
    var consolidatedGrid = __assign({}, margin, grid);
    consolidatedGrid.Width += grid.Width;
    consolidatedGrid.Height += grid.Height;
    consolidatedGrid.Padding = margin.Padding ? margin.Padding : 
    grid.Padding;
    return consolidatedGrid;
}
```

然而，如果我们使用 ES2015 或更高版本编译代码，`__assign`函数将被移除，我们的`ConsolidatedGrid` JavaScript 看起来如下：

```ts
function ConsolidatedGrid(grid, margin) {
    let consolidatedGrid = Object.assign({}, margin, grid);
    consolidatedGrid.Width += grid.Width;
    consolidatedGrid.Height += grid.Height;
    consolidatedGrid.Padding = margin.Padding ? margin.Padding : 
    grid.Padding;
    return consolidatedGrid;
}
```

我们在这里看到的是，TypeScript 努力确保它可以生成无论我们针对的 ECMAScript 版本是哪个都能工作的代码。我们不必担心该功能是否可用；我们把这个问题留给 TypeScript 来填补空白。

# 使用 REST 属性解构对象

在构建对象时，我们使用扩展运算符，我们也可以使用 REST 属性解构对象。解构简单地意味着我们要把一个复杂的*东西*分解成更简单的东西。换句话说，解构发生在我们将数组或对象的属性中的元素分配给单独的变量时。虽然我们一直能够将复杂的对象和数组分解为更简单的类型，但 TypeScript 提供了一种干净而优雅的方式，使用 REST 参数来分解这些类型，可以解构对象和数组。

为了理解 REST 属性是什么，我们首先需要了解如何解构对象或数组。我们将从解构以下对象文字开始，如下所示：

```ts
let guitar = { manufacturer: 'Ibanez', type : 'Jem 777', strings : 6 };
```

我们可以通过以下方式解构这个对象：

```ts
const manufacturer = guitar.manufacturer;
const type = guitar.type;
const strings = guitar.strings;
```

虽然这样可以工作，但不够优雅，而且有很多重复。幸运的是，TypeScript 采用了 JavaScript 的语法，用于像这样简单的解构，提供了一个更整洁的语法：

```ts
let {manufacturer, type, strings} = guitar;
```

从功能上讲，这导致与原始实现相同的单独项目。单个属性的名称必须与我们解构的对象中的属性的名称匹配——这就是语言知道哪个变量与对象上的哪个属性匹配的方式。如果我们因某种原因需要更改属性的名称，我们使用以下语法：

```ts
let {manufacturer : maker, type, strings} = guitar;
```

对象上的 REST 运算符的想法是，当你获取可变数量的项目时，它适用于对象，因此我们将这个对象解构为制造商，其他字段将被捆绑到 REST 变量中，如下所示：

```ts
let { manufacturer, ...details } = guitar;
```

REST 运算符必须出现在赋值列表的末尾；如果我们在它之后添加任何属性，TypeScript 编译器会抱怨。

在这个语句之后，`details`现在包含了类型和字符串值。有趣的地方在于我们看一下生成的 JavaScript。在前面的例子中，解构的形式在 JavaScript 中是相同的。在 JavaScript 中没有 REST 属性的等价物（至少在 ES2018 之前的版本中没有），因此 TypeScript 为我们生成了代码，让我们以一种一致的方式解构更复杂的类型：

```ts
// Compiled as ES5
var manufacturer = guitar.manufacturer, details = __rest(guitar, ["manufacturer"]);
var __rest = (this && this.__rest) || function (s, e) {
    var t = {};
    for (var p in s) if (Object.prototype.hasOwnProperty.call(s, p) && 
    e.indexOf(p) < 0)
        t[p] = s[p];
    if (s != null && typeof Object.getOwnPropertySymbols === "function")
        for (var i = 0, p = Object.getOwnPropertySymbols(s); i < p.length;
        i++) if (e.indexOf(p[i]) < 0)
            t[p[i]] = s[p[i]];
    return t;
};
```

数组解构与对象解构类似。语法与对象版本几乎相同；不同之处在于它使用`[]`来解构，而对象版本使用`{}`，以及变量的顺序是基于数组中项目的位置。

解构数组的原始方法依赖于将变量与数组中特定索引处的项目关联起来：

```ts
const instruments = [ 'Guitar', 'Violin', 'Oboe', 'Drums' ];
const gtr = instruments[0];
const violin = instruments[1];
const oboe = instruments[2];
const drums = instruments[3];
```

使用数组解构，我们可以将此语法更改为更简洁的形式，如下所示：

```ts
let [ gtr, violin, oboe, drums ] = instruments;
```

知道 TypeScript 团队擅长为我们提供一致和逻辑的体验，应该不会让人感到意外，我们也可以使用类似的语法将 REST 属性应用于数组：

```ts
let [gtr, ...instrumentslice] = instruments;
```

再次强调，没有直接的 JavaScript 等价物，但编译后的 TypeScript 显示 JavaScript 确实提供了基本原理，TypeScript 设计者能够优雅地使用`array.slice`进行整合。

```ts
// Compiled as ES5
var gtr = instruments[0], instrumentslice = instruments.slice(1);
```

# 使用 REST 处理可变数量的参数

关于 REST 我们需要看的最后一件事是函数具有 REST 参数的概念。这些与 REST 属性不同，但语法非常相似，我们应该很容易掌握。REST 参数解决的问题是处理传递给函数的可变数量的参数。在函数中识别 REST 参数的方法是它前面有省略号，并且它被定义为数组。

在这个例子中，我们将记录一个标题，然后是可变数量的`instruments`：

```ts
function PrintInstruments(log : string, ...instruments : string[]) : void {
    console.log(log);
    instruments.forEach(instrument => {
        console.log(instrument);
    });
}
PrintInstruments('Music Shop Inventory', 'Guitar', 'Drums', 'Clarinet', 'Clavinova');
```

由于 REST 参数是一个数组，这使我们可以直接从中执行`forEach`等操作。重要的是，REST 参数与 JavaScript 函数内的 arguments 对象不同，因为它们从参数列表中未命名的值开始，而 arguments 对象包含所有参数的列表。

由于 ES5 中没有 REST 参数，TypeScript 会提供必要的工作来提供模拟 REST 参数的 JavaScript。首先，我们将看到编译为 ES5 时的情况，如下所示：

```ts
function PrintInstruments(log) {
    var instruments = [];
    // As our rest parameter starts at the 1st position in the list of 
    // arguments,
    // our index starts at 1.
    for (var _i = 1; _i < arguments.length; _i++) {
        instruments[_i - 1] = arguments[_i];
    }
    console.log(log);
    instruments.forEach(function (instrument) {
        console.log(instrument);
    });
}
```

当我们查看从 ES2015 编译生成的 JavaScript 时（您需要在`tsconfig.json`文件中将目标更改为 ES2015），我们看到它看起来与我们的 TypeScript 代码完全相同：

```ts
function PrintInstruments(log, ...instruments) {
    console.log(log);
    instruments.forEach(instrument => {
        console.log(instrument);
    });
}
```

在这一点上，我无法再强调查看生成的 JavaScript 有多么重要。TypeScript 非常擅长隐藏复杂性，但我们确实应该熟悉生成的内容。我发现这是了解*底层*发生了什么的好方法，尽可能使用不同版本的 ECMAScript 标准进行编译，并查看生成的代码。

# 使用装饰器的 AOP

在 TypeScript 中我最喜欢的功能之一是使用装饰器。装饰器作为一项实验性功能被引入，它们是我们可以使用的代码片段，用于修改单个类的行为，而无需更改类的内部实现。通过这个概念，我们可以调整现有类的行为，而无需对其进行子类化。

如果您从 Java 或 C#等语言转到 TypeScript，您可能会注意到装饰器看起来很像一种称为 AOP 的技术。AOP 技术提供给我们的是通过跨越代码并将其分离到不同位置来提取重复代码的能力。这意味着我们不必在实现中散布大量基本代码，但这些代码在运行应用程序中必须存在。

解释装饰器的最简单方法是从一个例子开始。假设我们有一个类，只有特定角色的用户才能访问某些方法，如下所示：

```ts
interface IDecoratorExample {
    AnyoneCanRun(args:string) : void;
    AdminOnly(args:string) : void;
}
class NoRoleCheck implements IDecoratorExample {
    AnyoneCanRun(args: string): void {
        console.log(args);
    }   
    AdminOnly(args: string): void {
        console.log(args);
    }
}
```

现在，我们将创建一个具有`admin`和`user`角色的用户，这意味着在这个类中调用两种方法都没有问题：

```ts
let currentUser = {user: "peter", roles : [{role:"user"}, {role:"admin"}] };
function TestDecoratorExample(decoratorMethod : IDecoratorExample) {
    console.log(`Current user ${currentUser.user}`);
    decoratorMethod.AnyoneCanRun(`Running as user`);
    decoratorMethod.AdminOnly(`Running as admin`);       
}
TestDecoratorExample(new NoRoleCheck());
```

这给我们我们期望的输出，如下所示：

```ts
Current user Peter
Running as user
Running as admin
```

如果我们创建一个只有`user`角色的用户，我们期望他们不应该能够运行只有管理员才能运行的代码。由于我们的代码没有角色检查，无论用户分配了什么角色，`AdminOnly`方法都将被运行。修复这段代码的一种方法是添加代码来检查权限，然后将其添加到每个方法中。

首先，我们将创建一个简单的函数来检查当前用户是否属于特定角色：

```ts
function IsInRole(role : string) : boolean {
    return currentUser.roles.some(r => r.role === role);
}
```

重新审视我们现有的实现，我们将改变我们的函数来调用这个检查，并确定`user`是否被允许运行该方法：

```ts
AnyoneCanRun(args: string): void {
    if (!IsInRole("user")) {
        console.log(`${currentUser.user} is not in the user role`);
        return;
    };
    console.log(args);
}   
AdminOnly(args: string): void {
    if (!IsInRole("admin")) {
        console.log(`${currentUser.user} is not in the admin role`);
    };
    console.log(args);
}
```

当我们看这段代码时，我们可以看到这里有很多重复的代码。更糟糕的是，虽然我们有重复的代码，但在这个实现中有一个 bug。在`AdminOnly`代码中，在`IsInRole`块内没有返回语句，所以代码仍然会运行`AdminOnly`代码，但它会告诉我们用户不在`admin`角色中，然后无论如何输出消息。这突显了重复代码的一个问题：很容易引入微妙（或不那么微妙）的 bug 而不自知。最后，我们违反了良好的**面向对象**（**OO**）开发实践的基本原则之一。我们的类和方法正在做它们不应该做的事情；代码应该只做一件事，所以检查角色不属于那里。在第二章，*使用 TypeScript 创建 Markdown 编辑器*，当我们更深入地探讨面向对象开发思维方式时，我们将更深入地讨论这个问题。

让我们看看如何使用方法装饰器来消除样板代码并解决单一职责问题。

在编写代码之前，我们需要确保 TypeScript 知道我们将使用装饰器，这是一个实验性的 ES5 功能。我们可以通过在命令行中运行以下命令来做到这一点：

```ts
tsc --target ES5 --experimentalDecorators
```

或者，我们可以在我们的`tsconfig`文件中设置这一点：

```ts
"compilerOptions": {
        "target": "ES5",
// other parameters….
        "experimentalDecorators": true
    }
```

启用了装饰器构建功能后，我们现在可以编写我们的第一个装饰器，以确保用户属于`admin`角色：

```ts
function Admin(target: any, propertyKey : string | symbol, descriptor : PropertyDescriptor) {
        let originalMethod = descriptor.value;
        descriptor.value = function() {
            if (IsInRole(`admin`)) {
                originalMethod.apply(this, arguments);
                return;
            }
            console.log(`${currentUser.user} is not in the admin role`);
        }
        return descriptor;
    }
```

每当我们看到一个函数定义看起来类似于这样的，我们知道我们正在看一个方法装饰器。TypeScript 期望按照这个顺序精确地使用这些参数：

```ts
function …(target: any, propertyKey : string | symbol, descriptor : PropertyDescriptor)
```

第一个参数用于引用我们正在应用的元素。第二个参数是元素的名称，最后一个参数是我们要应用装饰器的方法的描述符；这允许我们改变方法的行为。我们必须有一个具有这个签名的函数作为我们的装饰器。

```ts
let originalMethod = descriptor.value;
descriptor.value = function() {
    ...
}
return descriptor;
```

装饰器方法的内部并不像它们看起来那么可怕。我们所做的是从描述符中复制原始方法，然后用我们自己的自定义实现替换该方法。这个包装的实现被返回，并且在我们遇到它时将被执行的代码：

```ts
if (IsInRole(`admin`)) {
    originalMethod.apply(this, arguments);
    return;
}
console.log(`${currentUser.user} is not in the admin role`);
```

在我们的包装实现中，我们正在执行相同的角色检查。如果检查通过，我们应用原始方法。通过使用这样的技术，我们已经添加了一些东西，可以以一致的方式避免调用我们的方法，如果不需要的话。

为了应用这个，我们在我们的装饰器工厂函数名字前面使用`@`，就在我们的类的方法之前。当我们添加我们的装饰器时，我们必须避免在它和方法之间加上分号，如下所示：

```ts
class DecoratedExampleMethodDecoration implements IDecoratorExample {
    AnyoneCanRun(args:string) : void {
        console.log(args);
    }
    @Admin
    AdminOnly(args:string) : void {
        console.log(args);
    }
}
```

虽然这段代码对于`AdminOnly`代码来说是有效的，但它并不特别灵活。随着我们添加更多的角色，我们将不得不添加越来越多几乎相同的函数。如果我们能有一种方法来创建一个通用函数，我们可以用它来返回一个接受设置我们想要允许的角色的参数的装饰器。幸运的是，我们可以使用一种叫做装饰器工厂的东西来做到这一点。

简而言之，TypeScript 装饰器工厂是一个可以接收参数并使用这些参数返回实际装饰器的函数。我们的代码只需要进行一些微小的调整，就可以得到一个可以指定我们想要保护的角色的工作工厂：

```ts
function Role(role : string) {
    return function(target: any, propertyKey : string | symbol, descriptor 
    : PropertyDescriptor) {
        let originalMethod = descriptor.value;
        descriptor.value = function() {
            if (IsInRole(role)) {
                originalMethod.apply(this, arguments);
                return;
            }
            console.log(`${currentUser.user} is not in the ${role} role`);
        }
        return descriptor;
    }
}
```

这里唯一的真正区别是我们有一个返回装饰器的函数，这个函数不再有名字，工厂函数参数被用在我们的装饰器内部。现在我们可以改变我们的类来使用这个工厂：

```ts
class DecoratedExampleMethodDecoration implements IDecoratorExample {
    @Role("user") // Note, no semi-colon
    AnyoneCanRun(args:string) : void {
        console.log(args);
    }
    @Role("admin")
    AdminOnly(args:string) : void {
        console.log(args);
    }
}
```

通过这种改变，当我们调用我们的方法时，只有管理员才能访问`AdminOnly`方法，而任何用户都可以调用`AnyoneCanRun`。一个重要的副作用是，我们的装饰器只适用于类内部。我们不能在独立的函数上使用它。

我们之所以称这种技术为装饰器，是因为它遵循了一种叫做**装饰器模式**的东西。这种模式认识到一种用于向单个对象添加行为而不影响同一类的其他对象并且不必创建子类的技术。模式只是对软件工程中常见问题的正式化解决方案，因此这些名称作为描述功能上发生的事情的有用缩写。也许不会讦知道还有一种工厂模式。当我们阅读本书时，我们将遇到其他模式的例子，因此当我们到达末尾时，我们将能够自如地使用它们。

我们也可以将装饰器应用到类中的其他项目上。例如，如果我们想要防止未经授权的用户甚至实例化我们的类，我们可以定义一个类装饰器。类装饰器被添加到类定义中，并期望接收构造函数作为函数。这是我们从工厂创建的构造函数装饰器的样子：

```ts
function Role(role : string) {
    return function(constructor : Function) {
        if (!IsInRole (role)) {
            throw new Error(`The user is not authorized to access this class`);
        }
    }
}
```

当我们应用这个时，我们遵循相同的格式，使用`@`前缀，所以当代码尝试为非管理员用户创建这个类的新实例时，应用程序会抛出错误，阻止这个类被创建：

```ts
@Role ("admin")
class RestrictedClass {
    constructor() {
        console.log(`Inside the constructor`);
    }
    Validate() {
        console.log(`Validating`);
    }
}
```

我们可以看到我们没有在类内声明任何装饰器。我们应该总是将它们创建为顶级函数，因为它们的用法不适合装饰一个类，所以我们不会看到诸如`@MyClass.Role("admin");`这样的语法。

除了构造函数和方法的装饰，我们还可以装饰属性、访问器等等。我们不会在这里详细介绍，但它们将在本书的后面出现。我们还将看看如何将装饰器链接在一起，以便我们有以下的语法：

```ts
@Role ("admin")
@Log(“Creating RestrictedClass”)
class RestrictedClass {
    constructor() {
        console.log(`Inside the constructor`);
    }
    Validate() {
        console.log(`Validating`);
    }
}
```

# 使用混合类型进行组合

当我们首次接触经典的面向对象理论时，我们会遇到类可以被继承的概念。这里的想法是我们可以从通用类创建更加专业化的类。其中一个更受欢迎的例子是我们有一个包含有关车辆基本细节的车辆类。我们从`vehicle`类继承，创建一个`car`类。然后我们从`car`类继承，创建一个`sports car`类。这里每一层继承都添加了在我们继承的类中不存在的特性。

总的来说，这对我们来说是一个简单的概念，但是当我们想要将两个或更多看似无关的事物结合起来编写我们的代码时会发生什么呢？让我们来看一个简单的例子。

数据库应用程序中常见的一件事是存储记录是否已被删除，而不实际删除记录，并记录记录上次更新的时间。乍一看，似乎我们希望在个人数据实体中跟踪这些信息。但我们可能不是将这些信息添加到每个数据实体中，而是创建一个包含这些信息的基类，然后从中继承：

```ts
class ActiveRecord {
    Deleted = false;
}
class Person extends ActiveRecord {
    constructor(firstName : string, lastName : string) {
        this.FirstName = firstName;
        this.LastName = lastName;
    }

    FirstName : string;
    LastName : string;
} 
```

这种方法的第一个问题是，它混合了有关记录状态的详细信息和实际记录本身。随着我们在接下来的几章中进一步深入 OO 设计，我们将不断强调这样混合物的想法并不是一个好主意，因为我们正在创建必须执行多个任务的类，这可能会使它们不够健壮。这种方法的另一个问题是，如果我们想要添加记录更新日期，我们要么必须将更新日期添加到`ActiveRecord`中，这意味着每个扩展`ActiveRecord`的类也将获得更新日期，要么我们必须创建一个新类，添加更新日期并将其添加到我们的层次结构链中，这意味着我们不能有没有删除字段的更新字段。

尽管继承确实有其用武之地，但近年来，将对象组合在一起以创建新对象的想法日益突出。这种方法的理念是我们构建不依赖于继承链的离散元素。如果我们重新审视我们的人员实现，我们将使用一种称为混合物的功能来构建相同的功能。

我们需要做的第一件事是定义一个类型，它将作为我们混合物的合适构造函数。我们可以给这种类型取任何名字，但在 TypeScript 中，围绕混合物演变出来的约定是使用以下类型：

```ts
type Constructor<T ={}> = new(...args: any[]) => T;
```

这种类型定义为我们提供了一些可以扩展以创建我们专门的混合物的东西。这种奇怪的语法有效地表示，给定任何特定类型，将使用任何适当的参数创建一个新实例。

这是我们的记录状态实现：

```ts
function RecordStatus<T extends Constructor>(base : T) {
    return class extends base {
        Deleted : boolean = false;
    }
}
```

`RecordStatus`函数通过返回一个扩展构造函数实现的新类来扩展`Constructor`类型。在这里，我们添加了我们的`Deleted`标志。

将这两种类型*合并*或混合在一起，我们只需执行以下操作：

```ts
const ActivePerson = RecordStatus(Person);
```

这已经创建了我们可以使用来创建具有`RecordStatus`属性的`Person`对象的东西。它实际上还没有实例化任何对象。为了做到这一点，我们以与任何其他类型相同的方式实例化信息：

```ts
let activePerson = new ActivePerson("Peter", "O'Hanlon");
activePerson.Deleted = true;
```

现在，我们还想添加有关记录上次更新时间的详细信息。我们创建另一个混合物，如下所示：

```ts
function Timestamp<T extends Constructor>(base : T) {
 return class extends base {
   Updated : Date = new Date();
 }
}
```

要将此添加到`ActivePerson`，我们更改定义以包括`Timestamp`。无论我们首先放置哪个混合物，无论是`Timestamp`还是`RecordStatus`：

```ts
const  ActivePerson  =  RecordStatus(Timestamp(Person));
```

除了属性，我们还可以向我们的混合物添加构造函数和方法。我们将把我们的`RecordStatus`函数更改为在记录被删除时记录日志。为此，我们将把我们的`Deleted`属性转换为一个 getter 方法，并添加一个新的方法来执行删除：

```ts
function RecordStatus<T extends Constructor>(base : T) {
    return class extends base {
        private deleted : boolean = false;
        get Deleted() : boolean {
            return this.deleted;
        }
        Delete() : void {
            this.deleted = true;
            console.log(`The record has been marked as deleted.`);
        }
    }
}
```

关于使用这种混合技术的警告。它们是一种很好的技术，可以整洁地做一些非常有用的事情，但除非我们放宽参数限制到任意，否则我们不能将它们作为参数传递。这意味着我们不能使用这样的代码：

```ts
function DeletePerson(person : ActivePerson) {
     person.Delete();
}
```

如果我们查看 TypeScript 文档中有关混合物的部分，我们会发现语法看起来非常不同。与处理这种方法的所有固有限制相比，我们将坚持这里的方法，这是我在[`basarat.gitbooks.io/typescript/docs/types/mixins.html`](https://basarat.gitbooks.io/typescript/docs/types/mixins.html)首次接触到的。

# 使用相同的代码和不同的类型以及使用泛型

当我们在 TypeScript 中首次开始开发类时，很常见的是我们反复编写相同的代码，只是改变我们依赖的类型。例如，如果我们想存储整数队列，我们可能会写以下类：

```ts
class QueueOfInt {
    private queue : number[]= [];

    public Push(value : number) : void {
        this.queue.push(value);
    }

    public Pop() : number | undefined {
        return this.queue.shift();
    }
}
```

调用这段代码就像这样简单：

```ts
const intQueue : QueueOfInt = new QueueOfInt();
intQueue.Push(10);
intQueue.Push(35);
console.log(intQueue.Pop()); // Prints 10
console.log(intQueue.Pop()); // Prints 35
```

后来，我们决定还需要创建一个字符串队列，所以我们也添加了相应的代码：

```ts
class QueueOfString {
    private queue : string[]= [];

    public Push(value : string) : void {
        this.queue.push(value);
    }

    public Pop() : string | undefined {
        return this.queue.shift();
    }
}
```

很容易看出，我们添加的这些代码越多，我们的工作就变得越繁琐，错误也就越多。假设我们忘记在其中一个实现中放置了 shift 操作。shift 操作允许我们从数组中删除第一个元素并返回它，这给了我们队列的核心行为（队列按照**先进先出**（或**FIFO**）的原则运行）。如果我们忘记了 shift 操作，我们实际上实现了一个堆栈操作（**后进先出**（或**LIFO**））。这可能导致代码中出现微妙且危险的错误。

通过泛型，TypeScript 为我们提供了创建所谓的泛型的能力，这是一种使用占位符来表示正在使用的类型的类型。调用泛型的代码负责确定它们接受的类型。我们可以通过在类名后面的`<>`内或在方法名后面出现的泛型来识别泛型。如果我们重写我们的队列以使用泛型，我们将看到这意味着什么：

```ts
class Queue<T> {
    private queue : T[]= [];

    public Push(value : T) : void {
        this.queue.push(value);
    }

    public Pop() : T | undefined {
        return this.queue.shift();
    }
}
```

让我们来分解一下：

```ts
class Queue<T> {
}
```

在这里，我们创建了一个名为`Queue`的类，它接受任何类型。`<T>`语法告诉 TypeScript，每当它在这个类内部看到`T`时，它指的是传递进来的类型：

```ts
private queue : T[]= [];
```

这是泛型类型首次出现的实例。编译器将使用泛型类型来创建数组，而不是将数组固定为特定类型：

```ts
public Push(value : T) : void {
    this.queue.push(value);
}

public Pop() : T | undefined {
    return this.queue.shift();
}
```

再次，我们用泛型替换了代码中的具体类型。请注意，TypeScript 很乐意在`Pop`方法中使用`undefined`关键字。

改变我们使用代码的方式，我们现在可以告诉我们的`Queue`对象我们想要应用的类型：

```ts
const queue : Queue<number> = new Queue<number>();
const stringQueue : Queue<string> = new Queue<string>();
queue.Push(10);
queue.Push(35);
console.log(queue.Pop());
console.log(queue.Pop());
stringQueue.Push(`Hello`);
stringQueue.Push(`Generics`);
console.log(stringQueue.Pop());
console.log(stringQueue.Pop());
```

特别有帮助的是，TypeScript 在引用的任何地方都强制执行我们分配的类型，因此，如果我们尝试向我们的`queue`变量添加一个字符串，TypeScript 将无法编译这个代码。

尽管 TypeScript 尽力保护我们，但我们必须记住它会转换为 JavaScript。这意味着它无法保护我们的代码免受滥用，因此，尽管 TypeScript 强制执行我们分配的类型，如果我们编写了调用我们泛型类型的外部 JavaScript，就没有任何东西可以阻止添加不受支持的值。泛型仅在编译时强制执行，因此，如果我们的代码将被外部调用，我们应该采取措施防止代码中出现不兼容的类型。

我们不仅限于在泛型列表中只有一个类型。只要它们具有唯一的名称，泛型允许我们在定义中指定任意数量的类型，如下所示：

```ts
function KeyValuePair<TKey, TValue>(key : TKey, value : TValue)
```

敏锐的读者会注意到我们已经遇到了泛型。当我们创建一个 mixin 时，我们在我们的`Constructor`类型中使用了泛型。

如果我们想从我们的泛型中调用特定的方法会发生什么？由于 TypeScript 希望知道类型的底层实现，它对我们可以做什么非常严格。这意味着以下代码是不可接受的：

```ts
interface IStream {
    ReadStream() : Int8Array; // Array of bytes
}
class Data<T> {
    ReadStream(stream : T) {
        let output = stream.ReadStream();
        console.log(output.byteLength);
    }
}
```

由于 TypeScript 无法猜测我们想在这里使用`IStream`接口，如果我们尝试编译这段代码，它会报错。幸运的是，我们可以使用泛型约束告诉 TypeScript 我们有一个特定的类型要在这里使用：

```ts
class Data<T extends IStream> {
    ReadStream(stream : T) {
        let output = stream.ReadStream();
        console.log(output.byteLength);
    }
}
```

`<T extends IStream>`部分告诉 TypeScript，我们将使用基于我们的`IStream`接口的*任何*类。

虽然我们可以将泛型限制为类型，但通常我们会希望将泛型限制为接口。这使我们在约束中使用的类具有很大的灵活性，并且不会强加我们只能使用从特定基类继承的类的限制。

要看到这个动作，我们将创建两个实现`IStream`的类：

```ts
class WebStream implements IStream {
    ReadStream(): Int8Array {
        let array : Int8Array = new Int8Array(8);
        for (let index : number = 0; index < array.length; index++){
            array[index] = index + 3; 
        }
        return array;
    }
}
class DiskStream implements IStream {
    ReadStream(): Int8Array {
        let array : Int8Array = new Int8Array(20); 
        for (let index : number = 0; index < array.length; index++){
            array[index] = index + 3;
        }
        return array;
    }
}
```

这些现在可以用作我们的通用`Data`实现中的类型约束：

```ts
const webStream = new Data<WebStream>();
const diskStream = new Data<DiskStream>();
```

我们刚刚告诉`webStream`和`diskStream`它们将可以访问我们的类。要使用它们，我们仍然必须传递一个实例，如下所示：

```ts
webStream.ReadStream(new WebStream());
diskStream.ReadStream(new DiskStream());
```

虽然我们在类级别声明了我们的泛型及其约束，但我们不必这样做。如果需要，我们可以在方法级别声明更精细的泛型。不过，在这种情况下，如果我们想要在代码中的多个地方引用该泛型类型，将其作为类级别泛型是有意义的。如果我们只想在一个或两个方法中应用特定的泛型，我们可以将我们的类签名更改为这样：

```ts
class Data {
    ReadStream<T extends IStream>(stream : T) {
        let output = stream.ReadStream();
        console.log(output.byteLength);
    }
}
```

# 使用地图映射值

经常出现的情况是需要使用一个容易查找的键存储多个项目。例如，假设我们有一个按流派分类的音乐收藏：

```ts
enum Genre {
    Rock,
    CountryAndWestern,
    Classical,
    Pop,
    HeavyMetal
}
```

对于这些流派中的每一个，我们将存储一些艺术家或作曲家的详细信息。我们可以采取的一种方法是创建一个代表每个流派的类。虽然我们可以这样做，但这将是对我们编码时间的浪费。我们解决这个问题的方式是使用一种叫做**map**的东西。地图是一个接受两种类型的通用类：用于地图的键的类型和存储在其中的对象的类型。

键是一个唯一的值，用于允许我们存储值或快速查找事物-这使得地图成为快速查找值的良好选择。我们可以将任何类型作为键，值可以是绝对任何东西。对于我们的音乐收藏，我们将创建一个使用流派作为键和字符串数组表示作曲家或艺术家的地图的类：

```ts
class MusicCollection {
    private readonly collection : Map<Genre, string[]>;
    constructor() {
        this.collection = new Map<Genre, string[]>();
    }
}
```

为了填充地图，我们调用`set`方法，如下所示：

```ts
public Add(genre : Genre, artist : string[]) : void {
    this.collection.set(genre, artist);
}
```

从地图中检索值就像调用`Get`与相关的键一样简单：

```ts
public Get(genre : Genre) : string[] | undefined {
    return this.collection.get(genre);
}
```

我们必须在这里添加`undefined`关键字到返回值，因为地图条目可能不存在。如果我们忘记考虑 undefined 的可能性，TypeScript 会友好地提醒我们。再一次，TypeScript 努力为我们的代码提供强大的安全保障。

我们现在可以填充我们的集合，如下所示：

```ts
let collection = new MusicCollection();
collection.Add(Genre.Classical, [`Debussy`, `Bach`, `Elgar`, `Beethoven`]);
collection.Add(Genre.CountryAndWestern, [`Dolly Parton`, `Toby Keith`, `Willie Nelson`]);
collection.Add(Genre.HeavyMetal, [`Tygers of Pan Tang`, `Saxon`, `Doro`]);
collection.Add(Genre.Pop, [`Michael Jackson`, `Abba`, `The Spice Girls`]);
collection.Add(Genre.Rock, [`Deep Purple`, `Led Zeppelin`, `The Dixie Dregs`]);
```

如果我们想添加一个单独的艺术家，我们的代码会变得稍微复杂。使用 set，我们要么在地图中添加一个新条目，要么用新条目替换先前的条目。由于情况如此，我们确实需要检查是否已经添加了特定的键。为此，我们调用`has`方法。如果我们还没有添加流派，我们将使用空数组调用 set。最后，我们将使用 get 从地图中获取数组，以便我们可以推入我们的值：

```ts
public AddArtist(genre: Genre, artist : string) : void {
    if (!this.collection.has(genre)) {
        this.collection.set(genre, []);
    }
    let artists = this.collection.get(genre);
    if (artists) {
        artists.push(artist);
    }
}
```

我们要对我们的代码做的另一件事是改变`Add`方法。现在，该实现会覆盖对特定流派的先前调用`Add`，这意味着调用`AddArtist`然后`Add`最终会覆盖我们单独添加的艺术家与`Add`调用中的艺术家：

```ts
collection.AddArtist(Genre.HeavyMetal, `Iron Maiden`);
// At this point, HeavyMetal just contains Iron Maiden
collection.Add(Genre.HeavyMetal, [`Tygers of Pan Tang`, `Saxon`, `Doro`]);
// Now HeavyMetal just contains Tygers of Pan Tang, Saxon and Doro
```

为了修复`Add`方法，只需简单地迭代我们的艺术家并调用`AddArtist`方法，如下所示：

```ts
public Add(genre : Genre, artist : string[]) : void {
    for (let individual of artist) {
        this.AddArtist(genre, individual);
    }
}
```

现在，当我们完成填充`HeavyMetal`流派时，我们的艺术家包括`Iron Maiden`，`Tygers of Pan Tang`，`Saxon`和`Doro`。

# 使用承诺和异步/等待创建异步代码

我们经常需要编写以异步方式行为的代码。这意味着我们需要启动一个任务并将其在后台运行，同时我们做其他事情。一个例子是当我们调用一个可能需要一段时间才能返回的 web 服务时。很长一段时间以来，在 JavaScript 中的标准方式是使用回调。这种方法的一个大问题是，我们需要的回调越多，我们的代码就变得越复杂，潜在的错误也就越多。这就是 promise 出现的地方。

Promise 告诉我们某事将以异步方式发生；在异步操作完成后，我们可以选择继续处理并处理 promise 的结果，或者捕获任何被异常抛出的异常。

以下是一个演示这一点的示例：

```ts
function ExpensiveWebCall(time : number) : Promise<void> {
    return new Promise((resolve, reject) => setTimeout(resolve, time));
}
class MyWebService {
    CallExpensiveWebOperation() : void {
        ExpensiveWebCall(4000).then(()=> console.log(`Finished web 
        service`))
            .catch(()=> console.log(`Expensive web call failure`));
    }
}
```

当我们写一个 promise 时，我们可以选择接受两个参数——一个`resolve`函数和一个`reject`函数，可以调用它们来触发错误处理。Promise 为我们提供了两个函数来处理这些值，所以`then()`将在成功完成操作时触发，另一个`catch`函数处理`reject`函数。

现在，我们将运行这段代码来看看它的效果：

```ts
console.log(`calling service`);
new MyWebService().CallExpensiveWebOperation();
console.log(`Processing continues until the web service returns`);
```

当我们运行这段代码时，我们得到以下输出：

```ts
calling service
Processing continues until the web service returns
Finished web service
```

在`处理继续直到 web 服务返回`和`完成 web 服务`之间，有四秒的延迟，这是我们预期的，因为应用程序在执行处理控制台日志时正在等待 promise 返回。这向我们展示的是，这段代码在这里是异步行为，因为它在执行处理控制台日志时并没有等待 web 服务调用返回。

我们可能会觉得这段代码有点冗长，而且散布`Promise<void>`并不是让其他人理解我们的代码是异步的最直观的方式。TypeScript 提供了一个语法等效的方法，使得我们的代码异步的地方更加明显。通过使用`async`和`await`关键字，我们可以轻松地将之前的示例变得更加优雅：

```ts
function ExpensiveWebCall(time : number) {
    return  new Promise((resolve, reject) => setTimeout(resolve, time));
}
class MyWebService {
    async CallExpensiveWebOperation() {
        await ExpensiveWebCall(4000);
        console.log(`Finished web service`);
    }
}
```

`async`关键字告诉我们，我们的函数正在返回`Promise`。它还告诉编译器我们希望以不同的方式处理这个函数。在`async`函数中找到`await`时，应用程序将在那一点暂停该函数，直到被等待的操作返回。在那一点，处理继续，模仿我们在`Promise`的`then()`函数中看到的行为。

为了捕获`async`/`await`中的错误，我们真的应该将函数内部的代码包装在 try...catch 块中。当错误被`catch()`函数明确捕获时，`async`/`await`没有处理错误的等效方式，所以我们需要处理问题：

```ts
class MyWebService {
    async CallExpensiveWebOperation() {
        try {
            await ExpensiveWebCall(4000);
            console.log(`Finished web service`); 
        } catch (error) {
            console.log(`Caught ${error}`);
        }
    }
}
```

无论你选择采取哪种方法都是个人选择。使用`async`/`await`只是意味着它包装了`Promise`方法，因此不同技术的运行时行为完全相同。不过我建议的是，一旦你在应用程序中决定了一种方法，就要保持一致。不要混合风格，因为这会让任何审查你的应用程序的人感到困难。

# 使用 Bootstrap 创建 UI。

在接下来的章节中，我们将在浏览器中做很多工作。创建一个吸引人的 UI 可能是一件困难的事情，特别是在一个我们可能还要针对不同布局模式的移动设备的时代。为了让事情对我们自己更容易些，我们将相当依赖 Bootstrap。Bootstrap 被设计为一个移动设备优先的 UI 框架，可以平稳地扩展到 PC 浏览器。在本节中，我们将布置包含标准 Bootstrap 元素的基本模板，然后看看如何使用诸如 Bootstrap 网格系统等功能来布置一个简单的页面。

我们将从 Bootstrap 的起始模板开始（[`getbootstrap.com/docs/4.1/getting-started/introduction/#starter-template`](https://getbootstrap.com/docs/4.1/getting-started/introduction/#starter-template)）。使用这个特定的模板，我们避免了下载和安装各种 CSS 样式表和 JavaScript 文件的需要；相反，我们依赖于众所周知的**内容交付网络**（**CDN**）来为我们获取这些文件。

在可能的情况下，我建议使用 CDN 来获取外部 JavaScript 和 CSS 文件。这提供了许多好处，包括不需要自己维护这些文件，并在浏览器在其他地方遇到这个 CDN 文件时获得浏览器缓存的好处。

起始模板如下所示：

```ts
<!doctype html>
<html lang="en">
   <head>
      <!-- Required meta tags -->
      <meta name="viewport" content="width=device-width, initial-scale=1, 
      shrink-to-fit=no">
      <link rel="stylesheet"href="https://stackpath.bootstrapcdn.com/bootstrap
      /4.1.3/css/bootstrap.min.css" integrity="sha384-
      MCw98/SFnGE8fJT3GXwEOngsV7Zt27NXFoaoApmYm81iuXoPkFOJwJ8ERdknLPMO"
      crossorigin="anonymous">
      <title>
         <
         <Template Bootstrap>
         >
      </title>
   </head>
   <body>
      <!-- 
         Content goes here...
         Start with the container.
         -->
      <script src="https://code.jquery.com/jquery-3.3.1.slim.min.js" 
         integrity="sha384-q8i/X+965DzO0rT7abK41JStQIAqVgRVzpbzo5smXKp4YfRvH+8abtTE1Pi6jizo" 
         crossorigin="anonymous"></script>
      <script 
         src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.14.3/umd/popper.min.js" 
         integrity="sha384-ZMP7rVo3mIykV+2+9J3UJ46jBk0WLaUAdn689aCwoqbBJiSnjAK/l8WvCWPIPm49" 
         crossorigin="anonymous"></script>
      <script src="https://stackpath.bootstrapcdn.com/bootstrap/4.1.3/js/bootstrap.min.js" 
         integrity="sha384-ChfqqxuZUCnJSK3+MXmPNIyE6ZbWh2IMqE241rYiqJxyMiZ6OW/JmZQ5stwEULTy" 
         crossorigin="anonymous"></script>
   </body>
</html>
```

布局内容的起点是容器。这是在前面的内容部分。以下代码显示了`div`部分：

```ts
<div class="container">

</div>
```

`container`类给了我们熟悉的 Twitter 外观，每个屏幕尺寸都有固定的大小。如果我们需要填满整个窗口，我们可以将其更改为`container-fluid`。

在容器内部，Bootstrap 尝试以网格模式布置项目。Bootstrap 操作一个系统，屏幕的每一行可以表示为最多 12 个离散的列。默认情况下，这些列均匀分布在页面上，因此我们可以通过选择适当数量的列来创建复杂的布局。幸运的是，Bootstrap 提供了一套广泛的预定义样式，帮助我们为不同类型的设备创建布局，无论是 PC、手机还是平板电脑。这些样式都遵循相同的命名约定`.col-<<size-identifier>>-<<number-of-columns>>`：

| **类型** | **超小设备** | **小设备** | **中等设备** | **大设备** |
| --- | --- | --- | --- | --- |
| **尺寸** | 手机 < 768px | 平板 >= 768px | 桌面 >= 992px | 桌面 >= 1200px |
| **前缀** | .col-xs- | .col-sm- | .col-md- | .col-lg- |

列数的工作方式是，每行理想情况下应该加起来为 12 列。因此，如果我们想要一行由三列、然后六列，最后又是三列的内容，我们会在容器内定义我们的行如下：

```ts
<div class="row">
  <div class="col-sm-3">Hello</div>
  <div class="col-sm-6">Hello</div>
  <div class="col-sm-3">Hello</div>
</div>
```

这种样式定义了在小设备上的显示方式。可以覆盖大设备的样式。例如，如果我们希望大设备使用五列、两列和五列，我们可以应用这种样式：

```ts
<div class="row">
  <div class="col-sm-3 col-lg-5">Hello</div>
  <div class="col-sm-6 col-lg-2">Hello</div>
  <div class="col-sm-3 col-lg-5">Hello</div>
</div>
```

这就是响应式布局系统的美妙之处。它允许我们生成适合我们设备的内容。

让我们看看如何向我们的页面添加一些内容。我们将在第一列中添加`jumbotron`，在第二列中添加一些文本，并在第三列中添加一个按钮：

```ts
<div class="row">
  <div class="col-md-3">
    <div class="jumbotron">
      <h2>
        Hello, world!
      </h2>
      <p>
        Lorem ipsum dolor sit amet, consectetur adipiscing elit. Phasellus 
        eget mi odio. Praesent a neque sed purus sodales interdum. In augue sapien, 
        molestie id lacus eleifend...
      </p>
      <p>
        <a class="btn btn-primary btn-large" href="#">Learn more</a>
      </p>
    </div>
  </div>
  <div class="col-md-6">
    <h2>
      Heading
    </h2>
    <p>
      Lorem ipsum dolor sit amet, consectetur adipiscing elit. Phasellus 
      eget mi odio. Praesent a neque sed purus sodales interdum. In augue sapien, 
      molestie id lacus eleifend...
    </p>
    <p>
      <a class="btn" href="#">View details</a>
    </p>
  </div>
  <div class="col-md-3">
    <button type="button" class="btn btn-primary btn-lg btn-block active">
      Button
    </button>
  </div>
</div>
```

同样，我们使用 CSS 样式来控制我们的显示样式。通过给`div`部分添加`jumbotron`样式，Bootstrap 立即为我们应用了该样式。我们通过选择将其设置为主按钮（`btn-primary`）等来精确控制我们的按钮的外观。

`jumbotron`通常横跨所有列的宽度。我们将其放在一个三列的`div`中，只是为了让我们看到宽度和样式是由网格布局系统控制的，`jumbotron`并没有一些特殊属性强制它横跨页面。

当我想要快速原型设计布局时，我总是遵循两个阶段的过程。第一步是在纸上画出我想要 UI 看起来的样子。我可以使用线框工具来做到这一点，但我喜欢能够快速画出东西的能力。一旦我大致知道我想要的布局是什么样子，我就会使用 Layoutit!（[`www.layoutit.com/`](https://www.layoutit.com/)）这样的工具将想法放到屏幕上；这也给了我导出布局的选项，这样我就可以手工进一步完善它。

# 总结

在本章中，我们看了 TypeScript 的一些特性，这些特性帮助我们构建未来的 TypeScript 代码。我们看了如何设置适当的 ES 级别来模拟或使用现代 ECMAScript 特性。我们看了如何使用联合和交集类型，以及如何创建类型别名。然后我们研究了对象扩展和 REST 属性，然后我们涵盖了装饰器的 AOP。我们还介绍了如何创建和使用映射类型，以及如何使用泛型和 promises。

为了准备本书其余部分中将要制作的 UI，我们简要介绍了使用 Bootstrap 来布局 UI，并介绍了 Bootstrap 网格布局系统的基础知识。

在下一章中，我们将使用一个简单的 Bootstrap 网页构建一个简单的 Markdown 编辑器，连接到我们的 TypeScript。我们将看到设计模式和单一职责类等技术如何帮助我们创建健壮的专业代码。

# 问题

1.  我们编写了一个应用程序，允许用户将华氏度转换为摄氏度，以及将摄氏度转换为华氏度。计算是在以下类中执行的：

```ts
class FahrenheitToCelsius {
    Convert(temperature : number) : number {
        return (temperature - 32) * 5 / 9;
    }
}

class CelsiusToFahrenheit {
    Convert(temperature : number) : number {
        return (temperature * 9/5) + 32;
    }
}
```

我们想要编写一个方法，该方法接受一个温度和这些类型的实例之一，然后执行相关的计算。我们将使用什么技术来编写这个方法？

1.  我们已经编写了以下类：

```ts
class Command {
    public constructor(public Name : string = "", public Action : Function = new Function()){}
}
```

我们想在另一个类中使用这个功能，我们将在其中添加多个命令。`Name`命令将作为键，我们可以在代码中稍后查找`Command`。我们将使用什么来提供这种键值功能，以及如何向其中添加记录？

1.  我们如何自动记录我们在*问题 2*中添加的命令的条目，而不在我们的`Add`方法中添加任何代码？

1.  我们创建了一个 Bootstrap 网页，我们想要显示一个包含六个中等大小列的行。我们该如何做？
