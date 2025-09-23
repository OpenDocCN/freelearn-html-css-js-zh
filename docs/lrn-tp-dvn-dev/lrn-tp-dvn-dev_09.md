# 第九章：使用新行为扩展类型

在上一章中，我们看到了 Reason 提供的工具和技术，这些工具和技术为通用编程开辟了可能性。一个是使用类型变量的参数多态性。另一个是函子，它可以接受一个或多个模块作为参数并返回一个模块。

在本章中，我们将探讨扩展类型本身以添加行为的方法。Reason 中可用的一种技术称为 **子类型化**。其思想是类型之间存在层次关系，具体类型是更通用类型的子类型。例如，一只 *猫* 可以是 *哺乳动物* 的子类型，而 *哺乳动物* 本身又是 *脊椎动物* 的子类型。

我们可以在 OCaml 文档中找到一个关于子类型化方法的定义：

子类型化控制着一个类型 A 的对象能否被用于期望另一个类型 B 的对象的表达式中。当这是真的时，我们说 A 是 B 的子类型。更具体地说，子类型化限制了何时可以应用转换操作符 `e :> t`。这种转换仅在 e 的类型是 t 的子类型时才有效。

除了子类型化之外，还有一些类似于我们在面向对象编程（OOP）风格的继承中所做的技术，这些技术可能很有用。

根据具体情况和使用案例，你可以利用这些技术中的一个或多个来改进你的代码结构，使其易于扩展功能，同时保持代码的类型安全性。

在本章中，我们将涵盖以下主题：

+   使用多态变体进行子类型化

+   使用 OOP 风格的继承进行代码复用

# 使用多态变体进行子类型化

正如我们在 第五章 中所看到的，*在类型中放置替代值*，Reason 有变体类型的概念，这可以在模式匹配和完备性检查中发挥作用。变体类型有它们更复杂和强大的版本，称为 **多态变体***。它们比常规变体提供了更多的灵活性。例如，它们使用特殊的语法定义，如 `type color = [Red | Orange | Yellow | Green | Blue ];`，并且它们的构造函数是独立存在的。

让我们看看它们如何通过子类型化来扩展类型行为。

# 重复使用不同类型的构造函数

由于构造函数是独立存在的，我们可以多次使用同一个构造函数。因此，我们可以定义一个名为 `rgb` 的类型，它使用与 `color` 类型定义一起提供的 `Red`、`Green` 和 `Blue` 构造函数。

在这里，我们首先定义了类型，然后是几个绑定来使用这些类型（`onegreen` 和 `othergreen`）：

```js
type color = [`Red | `Orange | `Yellow | `Green | `Blue ];
type rgb = [`Red | `Green | `Blue];

/* Bindings using the variants we defined */
let *onegreen*: color = `Green;
let *othergreen*: rgb = `Green;
```

让我们再添加一些代码来展示值：

```js
/* Console log */
Js.log(onegreen);
Js.log(othergreen);
```

在这一点上，如果你尝试执行编译此 Reason 代码生成的 JavaScript，你会得到类似以下输出的结果：

```js
756711075
756711075
```

如我们所见，打印的标识符对两个变量都是相同的，这我们可能已经有所预感，因为我们使用了相同的构造函数（` `Green ``）。

让我们通过添加将值转换为字符串的函数来使这个例子更直观。

我们添加一个函数，为每个颜色提供一个字符串值，如下所示：

```js
let *stringOfColor* = (c: color) : string => {
 switch (c) {
 | *`Red* => "red"
 | *`Orange* => "orange"
 | *`Yellow* => "yellow"
 | *`Green* => "green"
 | *`Blue* => "blue"
 }
};
```

我们添加一个函数，为每个 *RGB 颜色* 提供一个字符串值，如下所示：

```js
let *stringOfRgb* = (c: rgb) : string => {
 switch (c) {
 | *`Red* => "RGB red"
 | *`Green* => "RGB green"
 | *`Blue* => "RGB blue"
 }
};
```

我们还添加了相同的代码，用于记录值，如下所示：

```js
Js.log(stringOfColor(onegreen));
Js.log(stringOfRgb(othergreen));
```

那么，这个新实验的结果是什么？完整版本的代码（在 `src/Ch09/Ch09_Example1.re` 文件中）编译后生成一个 JS 代码文件，使用 `node` 命令执行时，输出结果类似于以下内容：

```js
756711075
756711075
green
RGB green
```

如你所料，使用两个不同的函数，我们能够根据输入类型（以及涉及的构造函数，例如多态构造函数 `Green`）来区分输出。

作为留给读者的练习，将 `Js.log(stringOfRgb(onegreen));` 添加到 ReasonML 代码中，并查看该调用输出的结果。

# 多态变体类型扩展的示例

代码复用的一个更有趣的技巧是，可以将现有的多态变体类型扩展以创建一个新的类型。让我们通过另一个例子来看一下。

假设我们有一个网站类型定义，我们想将其扩展到所有网络应用（包括个人生产力工具、社交网络，甚至 API）。我们首先定义了特定于女性的类别，如下所示：

```js
type onlyWomanShoe = [`Slingbacks | `HighHeels];
```

我们可以通过重用该类型定义并扩展它来为所有（女性和男性）定义多态变体，如下所示：

```js
type shoe = [onlyWomanShoe | `Moccasins | `Boots | `Sneakers | `Wingtips];
```

我们可以通过代码文件 `src/Ch09/Ch09_Example2.re` 来查看这些类型的结果。像往常一样，我们添加一些控制台日志（使用 `Js.log()`）使事情更有趣。完整的代码如下：

```js
type onlyWomanShoe = [`Slingbacks | `HighHeels];
type shoe = [onlyWomanShoe | `Moccasins | `Boots | `Sneakers | `Wingtips];

let *johndoe_shoe*: shoe = `Moccasins;
let *janedoe_shoe*: shoe = `Slingbacks;

Js.log(johndoe_shoe);
Js.log(janedoe_shoe);

let *infoAboutShoe* = (s: shoe) : string => {
 switch (s) {
 | *`Slingbacks* => "Slingbacks - Specific woman shoe"
 | *`HighHeels* => "High Heels - Specific woman shoe"
 | *`Moccasins* => "Moccasins"
 | *`Boots* => "Boots"
 | *`Sneakers* => "Sneakers"
 | *`Wingtips* => "Wingtips"
 }
};

Js.log(infoAboutShoe(johndoe_shoe));
Js.log(infoAboutShoe(janedoe_shoe)); 
```

执行编译结果生成的 JS 代码会输出类似于以下内容：

```js
265261402
-594895036
Moccasins
Slingbacks - Specific woman shoe
```

如果你在将构造函数绑定到 `janedoe_shoe` 变量时出错，例如，写下以下内容，编译时会发生什么很有趣：

```js
let *janedoe_shoe*: onlyWomanShoe = `Slingbacks;
```

尝试一下，在重新编译期间，你会立即看到类似于以下错误的错误：

```js
  This has type:
    onlyWomanShoe
  But somewhere wanted:
    shoe
  The first variant type does not allow tag(s)
  `Boots, `Moccasins, `Sneakers, `Wingtips
```

现在，回到我们的正常代码！我们能否在这里改进代码以减少代码量？我们能否在 `infoAboutShoe()` 函数中利用变体类型 `onlyWomanShoe`，以避免 `switch` 块中的 `Slingbacks` 和 `HighHeels` 的两行代码，并尝试使其真正强大？

有一个技巧可以在函数的 `switch` 块中使用语法糖来实现这一点：`#onlyWomanShoe`。

我们将在稍后详细解释这一点，但基本上我们可以编写几乎等效的代码（`src/Ch09/Ch09_Example2bis.re`），我们只更改 `infoAboutShoe` 函数的主体，如下所示：

```js
let *infoAboutShoe* = (s: shoe) : string => {
    switch (s) {
        | *#onlyWomanShoe* => "Woman shoe such as Sandals or High Heels"
        | *`Moccasins* => "Moccasins"
        | *`Boots* => "Boots"
        | *`Sneakers* => "Sneakers"
        | *`Wingtips* => "Wingtips"
    }
};
```

执行编译生成的 JS 代码会输出如下内容：

```js
265261402
-594895036
Moccasins
Woman shoe such as Slingbacks or High Heels
```

# 更多关于扩展多态变体类型的内容

假设我们有一个网站类型定义，我们想将其扩展到所有网络应用（包括个人生产力工具、社交网络，甚至 API）。我们首先定义了描述网站的主要特征，如下所示：

+   域名

+   是否通过登录进行访问（对于主要内容），即私有或公共访问

要开始，我们首先定义两个实用类型，`domain`和`accessType`，如下所示：

```js
type domain = [ `Domain(string) ];
type accessType = [`Private | `Public];
```

然后，我们添加一个辅助函数来将`accessType`值转换为字符串：

```js
let *accessTypeName* = (a: accessType) : string => {
 switch (a) {
 | *`Private* => "private"
 | *`Public* => "public"
 }
 };
```

现在，让我们定义网站的变体类型：

```js
type website = [
 | *`CorporateSite*(domain)
 | *`CommerceSite*(domain, accessType)
 | *`Blog*(domain, accessType)
 ];
```

然后，我们添加一个函数，该函数将返回包含每个网站信息的简短摘要文本：

```js
let *siteSummary* = (app: website) : string => {
 switch (app) {
 | *`CorporateSite*(`Domain(s)) => s ++ " - corporate site (public)"
 | *`CommerceSite*(`Domain(s), a) => s ++ " - commerce site (" ++ accessTypeName(a) ++ ")"
 | *`Blog*(`Domain(s), a) => {
 switch (a) {
 | *`Private* => s ++ " - " ++ "corporate blog (" ++ accessTypeName(a) ++ ")"
 | *`Public* => s ++ " - blog (public - login-based access for authors)"
 }
 }
 }
};
```

如同往常，为了测试目的，我们添加了一些绑定，并使用`Js.log()`在控制台显示结果：

```js
let *mysite* = `CorporateSite(`Domain("www.acme.com"))
Js.log(siteSummary(mysite))

let *myblog* = `Blog(`Domain("www.contentgardening.com"), `Public)
Js.log(siteSummary(myblog))

let *corpinternalblog* = `Blog(`Domain("internalblog.acme.com"), `Private)
Js.log(siteSummary(corpinternalblog))
```

我们可以看到这部分正在工作。但是，我们还没有完成。我们还想将这段代码扩展到任何 webapp。因此，我们定义了一个新的类型，用于*web apps*（`webapp`，正如你所猜到的），目的是作为*websites*类型的扩展。类型定义如下：

```js
type webapp = [
 | *`CorporateSite*(domain)
 | *`CommerceSite*(domain, accessType)
 | *`Blog*(domain, accessType)
 | *`SocialApp*(domain)
 ];
```

之后，我们添加了一个新的函数（`appSummary()`），它将扩展之前的函数（`siteSummary()`）。我们在这里使用的一个技术是`as`关键字，它允许我们在函数的`switch`部分将结果与`website`变体类型的构造函数相匹配。例如，我们可以写`` `CorporateSite(`Domain(s)) as ws ```。

我们可以定义函数如下：

```js
let *appSummary* = (app: webapp) : string => {
 switch (app) {
 | *`CorporateSite*(`Domain(s)) as ws => siteSummary(ws)
 | *`CommerceSite*(`Domain(s), a) as ws => siteSummary(ws)
 | *`Blog*(`Domain(s), a) as ws => siteSummary(ws)
 | *`SocialApp*(`Domain(s)) => s ++ " - social app"
 }
};
```

注意，我们实际上可以稍微改进一下；为了避免编译器对`switch`块的前三条线中的`s`和`a`变量未使用而抱怨，我们可以用`_`替换这些变量。改进后的函数如下：

```js
/* 1) the extended function **/
let *appSummary* = (app: webapp) : string => {
 switch (app) {
 | *`CorporateSite*(`Domain(_)) as ws => siteSummary(ws)
 | *`CommerceSite*(`Domain(_), _) as ws => siteSummary(ws)
 | *`Blog*(`Domain(_), _) as ws => siteSummary(ws)
 | *`SocialApp*(`Domain(s)) => s ++ " - social app"
 }
};
```

我们可以通过添加一些测试代码来完成，如下所示：

```js
Js.log("---")
let *fb* = `SocialApp(`Domain("facebook.com"))
Js.log(appSummary(fb))
```

但是等等，还有更多的改进空间！

另一种技术是通过`#website as ws`语法重用第一个类型，这允许我们完全替换与`website`变体类型构造函数相关的模式匹配的行。

函数的最终版本如下：

```js
/* 2) the extended function improved! **/
let *appSummaryImproved* = (app: webapp) : string => {
 switch (app) {
 | #website as ws => siteSummary(ws)
 | *`SocialApp*(`Domain(s)) => s ++ " - social app"
 }
};
```

然后，在这个更新之后，为了使事情更有趣，以下是最终的测试和值显示代码：

```js
Js.log("------")
Js.log(appSummaryImproved(mysite))
Js.log(appSummaryImproved(myblog))
Js.log(appSummaryImproved(corpinternalblog))
Js.log(appSummaryImproved(fb))
```

我们可以看到，执行编译生成的 JavaScript 代码会产生以下输出：

```js
www.acme.com - corporate site (public)
www.contentgardening.com - blog (public - login-based access for authors)
internalblog.acme.com - corporate blog (private)
---
facebook.com - social app
------
www.acme.com - corporate site (public)
www.contentgardening.com - blog (public - login-based access for authors)
internalblog.acme.com - corporate blog (private)
facebook.com - social app
```

我们刚刚看到了 Reason 支持的一种很好的类型扩展技术，即多态类型变体。

# 使用面向对象风格的继承进行代码复用

继承，就像我们在面向对象编程语言中看到的那样，在 Reason 中并不常用。然而，我们可以找到一些类似于 OOP 风格继承的技术示例。

# 打开一个模块

在模块中不断引用某个东西会给开发者带来很多编写工作。这就是`open`派上用场的地方。

没有这个功能，假设你有一个`ColorExample`模块（基于我们在本章第一个示例中已经使用过的代码），定义如下：

```js
/* A module */
module ColorExample = { 
  type color = [`Red | `Orange | `Yellow | `Green | `Blue ];
  type rgb = [`Red | `Green | `Blue];

  let *onegreen*: color = `Green;
  let *othergreen*: rgb = `Green;

  let *stringOfColor* = (c: color) : string => {
    switch (c) {
        | *`Red* => "red"
        | *`Orange* => "orange"
        | *`Yellow* => "yellow"
        | *`Green* => "green"
        | *`Blue* => "blue"
    }
  };

  let *stringOfRgb* = (c: rgb) : string => {
    switch (c) {
        | *`Red* => "RGB red"
        | *`Green* => "RGB green"
        | *`Blue* => "RGB blue"
    }
  };
}
```

你可以通过使用`ColorExample.stringOfColor`引用来使用该函数，对于值也是类似。所以，一些值显示代码可能如下所示（如实际在`src/Ch09/Ch09_Open_module.re`文件中看到的那样）：

```js
/* Use the module the default way */
Js.log("1/ Use function and values inside the module...");
Js.log(ColorExample.stringOfColor(ColorExample.onegreen));
Js.log(ColorExample.stringOfRgb(ColorExample.othergreen));
```

但是，使用在作用域内打开模块的 `open` 解决方案，我们可以编写更紧凑的代码，如下（在同一个 `src/Ch09/Ch09_Open_module.re` 文件中）：

```js
/* Open the module and use its content */
Js.log("2/ Use function and values from the module after opening it...");
let colorString = {
  open ColorExample;
  let oneblue: color = `Blue;
  Js.log("String value of another color: " ++ stringOfColor(oneblue));
};
```

事情按预期进行！执行代码文件时给出以下输出：

```js
1/ Use function and values inside the module...
green
RGB green
2/ Use function and values from the module after opening it...
String value of another color: blue
```

发生的事情是，模块的所有内容都可以神奇地访问，无需添加通常在访问包含的类型、函数和变量时需要添加的前缀（在这种情况下是 `ColorExample.` 部分）。它将模块的内容引入当前作用域。

我们可以以另一种方式做事：将模块放在自己的文件中以更好地分离代码。在某些情况下，如果你认为这样做更适合你的代码组织方式，你可能想这样做。所以基本上，让我们将模块的代码移动到 `src/Ch09/Ch09_OpenModulebisPart1.re` 文件中，例如，如下所示：

```js
module ColorExample = { 
  type color = [`Red | `Orange | `Yellow | `Green | `Blue ];
  type rgb = [`Red | `Green | `Blue];

  let *onegreen*: color = `Green;
  let *othergreen*: rgb = `Green;

  let *stringOfColor* = (c: color) : string => {
    *switch* (c) {
        | *`Red* => "red"
        | *`Orange* => "orange"
        | *`Yellow* => "yellow"
        | *`Green* => "green"
        | *`Blue* => "blue"
    }
  };

  let *stringOfRgb* = (c: rgb) : string => {
    switch (c) {
        | *`Red* => "RGB red"
        | *`Green* => "RGB green"
        | *`Blue* => "RGB blue"
    }
  };
}
```

然后，在另一个代码文件（例如 `src/Ch09/Ch09_OpenModulebisPart2.re`），我们可以打开模块并访问其函数和值，如下所示：

```js
open Ch09_OpenModulebisPart1.ColorExample;

Js.log(stringOfColor(onegreen));
Js.log(stringOfRgb(othergreen));

let *colorString* = {
  let *oneblue*: color = `Blue;
 Js.log("String value of another color: " ++ stringOfColor(oneblue));
};
```

就这样！我们已经看到了如何在 Reason 中利用 `open` 打开模块。并且，在你可以找到的其他开发者的代码中，你可以发现它被大量使用。

# 包含一个模块

Reason 还提供了一种重用已定义的模块并像在面向对象中一样扩展它的方法：`include` 关键字。

关于这个特性，文档实际上是这样说的：

在模块中使用 `include`，可以将模块的内容静态地转移到新的模块中。因此，通常可以满足继承或混合的角色。

假设我们有一个基础模块如下：

```js
module *Site* = {
 let *siteEnvMarker* = "TESTING";
 let *protocol* = (~secured) => secured ? "https" : "http";
 let *getInfo* = domainName => protocol(~secured=false) ++ "://" ++ domainName ++ " (" ++ siteEnvMarker ++ ")";
};
```

现在，我们将在以下模块中使用它，使用 `include` 技术，如下所示：

```js
module ProductionSite = {
 include Site;
 let *siteEnvMarker* = "production!";
 let *getPublicInfo* = domainName => {
 let *additionalText* = " (" ++ String.uppercase(siteEnvMarker) ++ ")";
 let *result* = protocol(~secured=true) ++ "://" ++ domainName ++ additionalText;
 Js.log(result);
 }
};
```

然后，我们可以使用每个模块中的函数来显示信息，以便比较行为：

```js
Js.log(Site.getInfo("dev-acme.com"));
print_newline();
ProductionSite.getPublicInfo("acme.com");
```

执行编译我们的代码（在 `src/Ch09/Ch09_Include_module.re` 文件中）产生的 JS 代码给出以下输出：

```js
http://dev-acme.com (TESTING)

https://acme.com (PRODUCTION!)
```

有趣。你可能已经对这里发生的事情有了直觉，但 ReasonML 关于 `include` 的文档规定，这种技术会静态地复制模块的定义，然后也执行 `open`。

# 摘要

我们现在已经看到了如何利用两种技术，使用多态变体进行子类型化和使用模块的面向对象继承风格，来改进代码结构并使其易于向类型添加行为。

使用多态变体类型，我们可以因为它们的设计而重用不同类型的构造函数。此外，还可以扩展多态变体类型以创建新的类型。

使用模块，我们可以打开一个现有的模块来使用其中定义的函数和绑定，并且我们可以将其包含在新的模块中以扩展其行为，类似于面向对象的继承风格。

在下一章和最后一章中，我们将通过一个最终示例来回顾我们迄今为止学到的所有主要技术。
