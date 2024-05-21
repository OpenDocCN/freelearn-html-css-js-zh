# 第六章：CSS-in-JS（在 Reason 中）

React 的一个很棒的特性是它让我们将组件的标记、行为和样式放在一个文件中。这种集合对开发人员的体验、版本控制和代码质量有着连锁反应（无意冒犯）。在本章中，我们将简要探讨 CSS-in-JS 是什么，以及我们如何在 Reason 中处理 CSS-in-JS。当然，如果您喜欢的话，可以完全将组件分开放在不同的文件中，或者使用更传统的 CSS 解决方案。

在本章中，我们将讨论以下主题：

+   什么是 CSS-in-JS？

+   使用`styled-components`

+   使用`bs-css`

# 什么是 CSS-in-JS？

定义 CSS-in-JS 目前是 JavaScript 社区中一个极具争议的话题。CSS-in-JS 诞生于组件时代。现代 Web 主要是基于组件模型构建的。几乎所有的 JavaScript 框架都已经接受了它。随着它的采用增加，越来越多的团队开始同时在同一个项目的各个组件上工作。想象一下，您正在一个分布式团队中开发一个大型应用程序，每个团队都在并行开发一个组件。如果没有团队统一 CSS 约定，您将遇到 CSS 作用域问题。如果没有某种类型的标准化的 CSS 样式指南，多个团队很容易会为一个类名设置样式，从而影响其他意外的组件。随着时间的推移，出现了许多解决这个问题和其他与规模有关的 CSS 问题的解决方案。

# 简史

一些流行的 CSS 约定包括 BEM、SMACSS 和 OOCSS。这些解决方案都要求开发人员学习约定并正确应用它们；否则，仍然可能出现令人沮丧的作用域问题。

CSS 模块成为了一个更安全的选择，开发人员可以将 CSS 导入到 JavaScript 模块中，构建步骤会自动将 CSS 局部范围限制在该 JavaScript 模块中。CSS 本身仍然是在普通的 CSS（或 SASS）文件中编写的。

CSS-in-JS 更进一步，允许您直接在 JavaScript 模块中编写 CSS，并自动将 CSS 局部范围限制在组件中。这对许多开发人员来说是正确的；其他人从一开始就不喜欢它。一些 CSS-in-JS 解决方案，如`styled-components`，允许开发人员直接将 CSS 与组件耦合在一起。您可以使用`<Header />`而不是`<header className="..." />`，其中`Header`组件是使用`styled-components`定义的，以及其 CSS，如下面的代码所示：

```js
import React from 'react';
import styled from 'styled-components';

const Header = styled.header`
  font-size: 1.5em;
  text-align: center;
  color: dodgerblue;
`;
```

曾经`styled-components`存在性能问题，因为 JavaScript 包必须在库能够在 DOM 中动态创建样式表之前下载、编译和执行。这些问题现在在很大程度上得到了解决，这要归功于服务器端渲染的支持。那么，在 Reason 中我们能做到这一点吗？让我们来看看！

# 使用 styled-components

`styled-components`最受欢迎的功能之一是根据组件的 props 动态创建 CSS 的能力。使用此功能的一个原因是创建组件的备用版本。然后这些备用版本将被封装在样式化组件本身内。以下是一个`<Title />`的示例，其中文本可以居中或左对齐，也可以选择是否加下划线。

```js
const Title = styled.h1`
  text-align: ${props => props.center ? "center" : "left"};
  text-decoration: ${props => props.underline ? "underline" : "none"};
  color: white;
  background-color: coral;
`;

render(
  <div>
    <Title>I'm Left Aligned</Title>
    <Title center>I'm Centered!</Title>
    <Title center underline>I'm Centered & Underlined!</Title>
  </div>
);
```

在 Reason 的背景下，挑战在于通过`style-components`API 创建一个可以动态处理 props 的组件。考虑`styled.h1`函数的以下绑定和我们的`<Title />`组件。

```js
/* StyledComponents.re */
[@bs.module "styled-components"] [@bs.scope "default"] [@bs.variadic]
external h1: (array(string), array('a)) => ReasonReact.reactClass = "h1";

module Title = {
  let title =
    h1(
      [|
        "text-align: ",
        "; text-decoration: ",
        "; color: white; background-color: coral;",
      |],
      [|
        props => props##center ? "center" : "left",
        props => props##underline ? "underline" : "none",
      |],
    );

  [@bs.deriving abstract]
  type jsProps = {
    center: bool,
    underline: bool,
  };

  let make = (~center=false, ~underline=false, children) =>
    ReasonReact.wrapJsForReason(
      ~reactClass=title,
      ~props=jsProps(~center, ~underline),
      children,
    );
};
```

`h1`函数接受一个字符串数组作为其第一个参数，以及一个表达式数组作为其第二个参数。这是因为这是 ES6 标记模板字面量的 ES5 表示。在`h1`函数的情况下，表达式数组是传递给 React 组件的 props 的函数。

我们使用 `[@bs.variadic]` 装饰器来允许任意数量的参数。在 Reason 中，我们使用数组，在 JavaScript 中，该数组会被扩展为任意数量的参数。

# 使用 [@bs.variadic]

让我们稍微偏离一下，进一步探索 `[@bs.variadic]`。假设你想要绑定 `Math.max()`，它可以接受一个或多个参数：

```js
/* JavaScript */
Math.max(1, 2);
Math.max(1, 2, 3, 4);
```

这是 `[@bs.variadic]` 的一个完美案例。我们在 Reason 中使用数组来保存参数，并且该数组将会被扩展以匹配 JavaScript 中的上述语法。

```js
/* Reason */
[@bs.scope "Math"][@bs.val][@bs.variadic] external max: array('a) => unit = "";
max([|1, 2|]);
max([|1, 2, 3, 4|]);
```

好的，我们回到了 `styled-components` 的例子。我们可以像下面这样使用 `<Title />` 组件：

```js
/* Home.re */
let component = ReasonReact.statelessComponent("Home");

let make = _children => {
  ...component,
  render: _self =>
    <StyledComponents.Title center=true underline=true>
 {ReasonReact.string("Page1")}
 </StyledComponents.Title>,
};
```

上面的代码是一个带有样式的 ReasonReact 组件，它渲染了一个带有一些 CSS 的 `h1`。CSS 在之前在 `StyledComponents.Title` 模块中定义。`<Title />` 组件有两个属性——center 和 underline——默认值都是 `false`。

当然，这不是编写样式组件的优雅方式，但在功能上与 JavaScript 版本相似。另一个选择是回到原始的 JavaScript 中，以利用熟悉的标记模板文字语法。让我们在 `Title.re` 中举个例子。

```js
/* Title.re */
%bs.raw
{|const styled = require("styled-components").default|};

let title = [%bs.raw
  {|
     styled.h1`
       text-align: ${props => props.center ? "center" : "left"};
       text-decoration: ${props => props.underline ? "underline" : "none"};
       color: white;
       background-color: coral;
     `
   |}
];

[@bs.deriving abstract]
type jsProps = {
  center: bool,
  underline: bool,
};

let make = (~center=false, ~underline=false, children) =>
  ReasonReact.wrapJsForReason(
    ~reactClass=title,
    ~props=jsProps(~center, ~underline),
    children,
  );
```

使用方式类似，只是现在 `<Title />` 组件不再是 `StyledComponents` 的子模块。

```js
/* Home.re */
let component = ReasonReact.statelessComponent("Home");

let make = _children => {
  ...component,
  render: _self =>
    <Title center=true underline=true> {ReasonReact.string("Page1")} </Title>,
};
```

就我个人而言，我喜欢使用 `[%bs.raw]` 版本时的开发体验。我想要为 Adam Coll（`@acoll1`）提供的 `styled-components` 绑定的两个版本鼓掌。我也很期待看看社区会有什么新的东西。

现在让我们来探索社区中最受欢迎的 CSS-in-JS 解决方案：`bs-css`。

# 使用 bs-css

虽然 Reason 团队没有对 CSS-in-JS 解决方案做出官方推荐，但目前许多人正在使用一个名为 `bs-css` 的库，它包装了 emotion CSS-in-JS 库（版本 9）。`bs-css` 库为在 Reason 中使用提供了类型安全的 API。通过这种方式，我们可以让编译器检查我们的 CSS。我们将通过转换我们在第三章中创建的 `App.scss` 来感受一下这个库。

要跟着做，克隆本书的 GitHub 仓库，并从 `Chapter06/app-start` 开始使用以下代码：

```js
git clone https://github.com/PacktPublishing/ReasonML-Quick-Start-Guide.git
cd ReasonML-Quick-Start-Guide
cd Chapter06/app-start
npm install
```

要开始使用 `bs-css`，我们将在 `package.json` 和 `bsconfig.json` 中将其包含为依赖项，如下所示：

```js
/* bsconfig.json */
...
"bs-dependencies": ["reason-react", "bs-css"],
...
```

通过 npm 安装 `bs-css` 并配置 `bsconfig.json` 后，我们将可以访问库提供的 `Css` 模块。通常的做法是定义自己的子模块叫做 `Styles`，在那里我们打开 `Css` 模块并编写所有的 CSS-in-Reason。由于我们将要转换 `App.scss`，我们将在 `App.re` 中声明一个 `Styles` 子模块，如下所示：

```js
/* App.re */

...
let component = ReasonReact.reducerComponent("App");

module Styles = {
  open Css;
};
...
```

现在，让我们转换以下的 Sass：

```js
.App {
  min-height: 100vh;

  &:after {
    content: "";
    transition: opacity 450ms cubic-bezier(0.23, 1, 0.32, 1),
      transform 0ms cubic-bezier(0.23, 1, 0.32, 1) 450ms;
    position: fixed;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    background-color: rgba(0, 0, 0, 0.33);
    transform: translateX(-100%);
    opacity: 0;
    z-index: 1;
  }

  &.overlay {
    &:after {
      transition: opacity 450ms cubic-bezier(0.23, 1, 0.32, 1);
      transform: translateX(0%);
      opacity: 1;
    }
  }
}
```

在 `Styles` 中，我们声明了一个叫做 `app` 的绑定，它将在 `<App />` 组件的 `className` 属性中使用。我们将绑定到一个叫做 `style` 的 `bs-css` 函数的结果。`style` 函数接受一系列 CSS 规则。让我们使用以下代码来探索语法：

```js
module Styles = {
  open Css;

  let app = style([
    minHeight(vh(100.)),
  ]);
};
```

一开始有点奇怪，但你使用得越多，它就会感觉越好。所有的 CSS 属性和单位都是函数。这些函数有类型。如果类型不匹配，编译器会报错。考虑以下无效的 CSS：

```js
min-height: red;
```

这在 CSS、Sass 甚至 `styled-components` 中都会悄悄失败。使用 `bs-css`，我们至少可以防止大量无效的 CSS。编译器还会通知我们任何未使用的绑定，这有助于我们维护 CSS 样式表，而且通常我们还有完整的智能感知，这有助于我们在学习 API 的过程中。

就我个人而言，我非常喜欢通过 Sass 嵌套 CSS，并且我很高兴我们可以用`bs-css`做同样的事情。为了嵌套`:after`伪选择器，我们使用`after`函数。为了嵌套`.overlay`选择器，我们使用`selector`函数。就像在 Sass 中一样，我们使用`&`符号来引用父元素，如下面的代码所示：

```js
module Styles = {
  open Css;

  let app =
    style([
      minHeight(vh(100.)),

      after([
 contentRule(""),
 transitions([
 `transition("opacity 450ms cubic-bezier(0.23, 1, 0.32, 1)"),
 `transition("transform 0ms cubic-bezier(0.23, 1, 0.32, 1) 450ms"),
 ]),
        position(fixed),
        top(zero),
        right(zero),
        bottom(zero),
        left(zero),
        backgroundColor(rgba(0, 0, 0, 0.33)),
        transform(translateX(pct(-100.))),
        opacity(0.),
        zIndex(1),
      ]),

      selector(
        "&.overlay",
        [ 
          after([
            `transition("opacity 450ms cubic-bezier(0.23, 1, 0.32, 1)"),
            transform(translateX(zero))),
            opacity(1.),
          ]),
        ],
      )
    ]);
};
```

请注意，我们正在使用多态变体``transition`来表示过渡字符串。否则过渡是无效的。

您可以在 GitHub 存储库的`Chapter06/app-end/src/App.re`文件中找到其余的转换。现在剩下的就是将样式应用到`<App />`组件的`className`属性，如下面的代码所示：

```js
/* App.re */
...
render: self =>
  <div
    className={"App " ++ Styles.app ++ (self.state.isOpen ? " overlay" : "")}
...
```

删除`App.scss`后，一切看起来基本相同。太棒了！唯一的例外是`nav > ul > li:after`选择器。在以前的章节中，我们使用内容属性来渲染图像，就像这样：

```js
content: url(./img/icon/chevron.svg);
```

根据`Css.rei`，`contentRule`函数接受一个字符串。因此，使用`url`函数不会通过类型检查，如下面的代码所示：

```js
contentRule(url("./img/icon/chevron.svg")) /* type error */
```

作为一种逃逸路线，`bs-css`提供了`unsafe`函数（如下面的代码所示），可以绕过这个问题：

```js
unsafe("content", "url('./img/icon/chevron.svg')")
```

然而，尽管我们的 webpack 配置以前将前面的图像作为依赖项引入，但在使用`bs-css`时不再这样做。

# 权衡

在 Reason 中使用 CSS-in-JS 显然是一种权衡。一方面，我们可以获得类型安全的、本地范围的 CSS，并且可以将我们的 CSS 与组件一起放置。另一方面，语法有点冗长，可能会有一些奇怪的边缘情况。选择 Sass 而不是 CSS-in-JS 解决方案是完全合理的，因为在这里没有明显的赢家。

# 其他库

我鼓励您尝试其他 CSS-in-JS Reason 库。每当您寻找 Reason 库时，您的第一站应该是 Redex (**Re**ason Package In**dex**)。

您可以在 Redex (**Re**ason Package In**dex**)找到：

[`redex.github.io/`](https://redex.github.io/)

另一个有用的资源是 Reason Discord 频道。这是一个很好的地方，可以询问各种 CSS-in-JS 解决方案及其权衡。

您可以在 Reason Discord 频道找到：

[`discord.gg/reasonml`](https://discord.gg/reasonml)

# 摘要

CSS-in-JS 仍然是相当新的，在不久的将来 Reason 社区将对其进行大量实验。在本章中，我们了解了 CSS-in-JS（在 Reason 中）的一些好处和挑战。你站在哪一边？

在第七章中，*Reason 中的 JSON*，我们将学习如何在 Reason 中处理 JSON，并了解 GraphQL 如何帮助减少样板代码，同时实现一些非常引人注目的保证。
