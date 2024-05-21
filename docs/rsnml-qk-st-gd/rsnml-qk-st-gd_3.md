# 第三章：创建 ReasonReact 组件

现在我们已经设置好了开发环境，我们准备开始使用 ReasonReact——ReactJS 的未来。ReasonML 和 ReasonReact 都是由构建 ReactJS 的同一个人构建的。ReasonReact 就是 Reason，就像 ReactJS 就是 JavaScript 一样。在本书的其余部分，我们将使用在本章开始构建的应用程序。以下是本章结束时我们将构建的内容的截图：

![](img/b88262e1-bf4b-4508-aa96-84b090a76664.png)

要跟着做，克隆这本书的 GitHub 存储库，并从`Chapter03/start`开始。在本书的其余部分，每个目录都与我们在第二章结束时设置的开发环境相同。

```js
git clone https://github.com/PacktPublishing/ReasonML-Quick-Start-Guide.git
cd ReasonML-Quick-Start-Guide
cd Chapter03/start
npm install
```

我们将首先探索 ReasonReact，并且在本章的中间部分，我们将转移到`Chapter03/app-start`目录，在那里我们将开始使用 ReasonReact 的内置路由器构建应用程序。

在本章中，我们将做以下事情：

+   探索创建无状态和有状态的 ReasonReact 组件

+   创建一个包括导航和路由的应用程序

+   看看你已经熟悉的这么多 ReactJS 概念如何很好地映射到 ReasonReact

+   了解 ReasonReact 如何通过 Reason 的类型系统帮助我们创建更健壮的组件

# 组件创建基础知识

让我们从分析一个简单的无状态组件开始。在`App.re`中，让我们呈现一个带有一些文本的`<div />`元素：

```js
let component = ReasonReact.statelessComponent("App");

let make = _children => {
  ...component,
  render: _self => <div> {ReasonReact.string("hello world")} </div>,
};
```

并在`Index.re`中，将组件呈现到 ID 为`"root"`的 DOM 元素：

```js
ReactDOMRe.renderToElementWithId(<App />, "root");
```

由于 Reason 的模块系统，我们不需要在`Index.re`中使用`import`语句，也不需要在`App.re`中使用导出语句。每个 Reason 文件都是一个模块，每个 Reason 模块都是全局可用的。在本书的后面，我们将看到如何隐藏模块的实现细节，以便您组件的用户只能访问他们应该访问的内容。

# 组件模板

在 ReasonReact 中，所有组件都是使用以下四个函数之一创建的：

+   `ReasonReact.statelessComponent`

+   `ReasonReact.statelessComponentWithRetainedProps`

+   `ReasonReact.reducerComponent`

+   `ReasonReact.reducerComponentWithRetainedProps`

这四个函数中的每一个都接受一个`string`并返回与不同组件模板对应的`record`。`string`参数仅用于调试目的。组件的名称(`<App />`)来自其文件名(`App.re`)。返回的记录包含的字段取决于使用了哪个函数。在我们之前的例子中，我们可以覆盖以下字段：

+   `render`

+   `didMount`

+   `willReceiveProps`

+   `shouldUpdate`

+   `willUpdate`

+   `didUpdate`

+   `willUnmount`

除了`render`字段外，其余的都是熟悉的 ReactJS 生命周期事件。要覆盖一个字段，在`make`函数返回的`record`中添加该字段。在前面的例子中，组件模板的`render`字段被自定义的`render`函数替换了。

`make`函数接受`props`作为参数，并返回与四个组件创建函数之一最初创建的形状相同的`record`。`make`函数的最后一个参数必须是`children`属性。您可能已经注意到，在前面的例子中，`children`前缀为`_`。如果您的组件不需要引用 children 属性，则使用`_`前缀可以防止未使用绑定的编译器警告。

`make`函数的花括号属于返回的`record`文字。`...component`表达式将原始`record`的内容扩展到这个新的`record`中，以便可以覆盖单个字段，而无需显式设置每个字段。

# self

`render`字段保存一个接受名为`self`的参数的回调函数，并返回类型为`ReasonReact.reactElement`的值。`self`记录的三个字段如下：

+   `state`

+   `handle`

+   `send`

ReasonReact 不具有 JavaScript 的`this`的概念。相反，`self`保存必要的信息，并提供给需要它的回调函数。在使用有状态组件时，我们将看到更多关于`self`的内容。

# 事件处理程序

在我们的渲染函数中，我们可以以与 ReactJS 相同的方式将事件侦听器附加到 DOM 元素上。例如，要监听点击事件，我们添加一个`onClick`属性并将其值设置为事件处理程序：

```js
let component = ReasonReact.statelessComponent("App");

let make = _children => {
  ...component,
  render: _self =>
    <div onClick={_event => Js.log("clicked")}>
      {ReasonReact.string("hello world")}
    </div>,
};
```

但是，这个回调函数必须接受一个参数（对应于 JavaScript DOM 事件）并且必须返回一个名为`unit`的类型。

# unit

在 Reason 中，`unit`是一个表示"nothing"的类型。返回类型为`unit`的函数除了`unit`之外不能返回任何其他值。`unit`类型有一个值：`()`（即一对空括号，也称为`unit`）。

相比之下，`bool`类型有两个值：`true`和`false`。`int`类型有无限多个值。

在第一章中讨论了*ReasonML 简介*，在 Reason 中表示可空值的习惯方式是使用`option`类型。`option`类型和`unit`类型之间的主要区别在于`option`类型的值可以是空，也可以是某个值，而`unit`类型的值始终是`()`。

接受和/或返回`unit`的函数可能会引起副作用。例如，`Js.log`是一个返回`unit`的函数。`onClick`事件处理程序也是一个返回`unit`的函数。

`Random.bool`是一个接受`unit`作为参数并返回`bool`的函数的示例。调用带有`unit`的函数的语法非常熟悉：

```js
Random.bool()
```

由于`onClick`需要一个返回`unit`的函数，以下内容将导致类型错误：

```js
let component = ReasonReact.statelessComponent("App");

let make = _children => {
  ...component,
  render: _self =>
    <div onClick={_event => 42}> {ReasonReact.string("hello world")} </div>,
};
```

类型错误显示在这里：

```js
Error: This expression has type int but an expression was expected of type
  unit
```

在错误消息中，`This expression`指的是`42`。

# JSX

Reason 带有 JSX 语法。ReasonReact 版本的 JSX 的一个区别是我们不能在 ReasonReact 中执行以下操作：

```js
<div>"hello world"</div>
```

相反，我们需要使用`ReasonReact.string`函数将`string`转换为`ReasonReact.reactElement`：

```js
<div>ReasonReact.string("hello world")</div>
```

但是，这仍然不起作用。我们还需要用`{ }`来包装表达式，以帮助解析器区分多个可能的子元素：

```js
<div> {ReasonReact.string("hello world")} </div>
```

您可以自由创建一个更简洁的别名并使用它：

```js
let str = ReasonReact.string;
<div> {str("hello world")} </div>;
```

在 JSX 中调用自定义组件时，将调用其`make`函数。`<App />`语法解糖为以下内容：

```js
ReasonReact.element(App.make([||]))
```

当组件将接收新的 props 时，它的`make`函数将再次被调用，并将新的 props 作为参数。`make`函数就像 ReactJS 的`constructor`和 ReactJS 的`componentWillReceiveProps`的组合。

# Props

让我们给我们的`<App />`组件添加一些 props：

```js
let make = (~greeting, ~name, _children) => {
  ...component,
  render: _self => <div> {ReasonReact.string(greeting ++ " " ++ name)} </div>,
};
```

编译后，我们得到了一个编译器错误，因为在`Index.re`中我们没有提供所需的`greeting`和`name`属性：

```js
We've found a bug for you!

1 │ ReactDOMRe.renderToElementWithId(<App />, "root");

This call is missing arguments of type:
(~greeting: string),
(~name: string)
```

`greeting`和`name`是`make`函数的**标记参数**，这意味着它们可以以任何顺序提供。要将参数转换为标记参数，请使用波浪号(`~`)作为前缀。Reason 还支持可选参数以及带默认值的参数。让我们给`greeting`一个默认值并使`name`可选：

```js
let make = (~greeting="hello", ~name=?, _children) => {
  ...component,
  render: _self => {
    let name =
      switch (name) {
      | None => ""
      | Some(name) => name
      };
    <div> {ReasonReact.string(greeting ++ " " ++ name)} </div>;
  },
};
```

由于`name`是一个可选参数，它被包装在`option`类型中，然后我们可以对其值进行模式匹配。当然，这只是一种提供`name`默认参数为`""`的冗长方式。

现在，即使未为`<App />`提供任何 props，我们的示例也可以编译：

```js
ReactDOMRe.renderToElementWithId(<App />, "root");
/* hello */

ReactDOMRe.renderToElementWithId(
  <App greeting="welcome," name="reason" />,
  "root",
);
/* welcome, reason */

```

如果我们决定删除名称属性，编译器将告诉我们需要更新`<App />`的使用位置。这使我们可以自由地重构我们的组件，而不必担心忘记更新代码库中的某个区域。编译器支持我们！

# 子元素

`make`函数的最后一个参数始终是`children`属性-它是强制性的。与其他属性一样，子元素可以是任何数据结构。只要组件允许，我们就可以使用在 ReactJS 中流行的渲染属性模式。重要的是，ReasonReact 始终将子元素包装在数组中，因此如果我们不想要这种包装，就需要使用`...`语法来解包数组。

在`App.re`中，我们将删除除了必需的`children`属性之外的所有属性。在渲染函数中，我们使用我们硬编码的问候语调用子元素：

```js
/* App.re */
let component = ReasonReact.statelessComponent("App");

let make = children => {
  ...component,
  render: _self => children("hello"),
};
```

在`Index.re`中，我们添加一个作为`<App />`子元素的函数，该函数接受提供的问候并返回 JSX（类型为`ReasonReact.reactElement`）。请注意`...`语法用于解包所有 ReasonReact 子元素都包装在其中的数组：

```js
/* Index.re */
ReactDOMRe.renderToElementWithId(
  <App> ...{greeting => <div> {ReasonReact.string(greeting)} </div>} </App>,
  "root",
);
```

如果我们忘记了`...`，编译器会友好地提醒我们：

```js
We've found a bug for you!

1 │ ReactDOMRe.renderToElementWithId(
2 │ <App> {greeting => <div> {ReasonReact.string(greeting)} </div>} </App>,
3 │ "root",
4 │ );

This has type:
  array('a)
But somewhere wanted:
  string => ReasonReact.reactElement
```

如果我们不包含任何子元素（即只有`<App />`），甚至会收到类似的编译器消息，因为这会转换为空数组。这意味着我们保证组件的用户必须在`<App />`的子元素中提供类型为`string => ReasonReact.reactElement`的函数，如果它要进行类型检查的话。

我们还可以要求我们的组件接受其他类型的子元素，例如两个字符串的元组：

```js
/* App.re */
let component = ReasonReact.statelessComponent("App");

let make = children => {
  ...component,
  render: _self => {
    let (greeting, name) = children;
    <div> {ReasonReact.string(greeting ++ " " ++ name)} </div>;
  },
};
/* Index.re */
ReactDOMRe.renderToElementWithId(<App> ...("hello", "tuple") </App>, "root");
```

由于在`App.re`中使用了它，Reason 能够推断出子元素必须是类型为`(string, string)`的元组。例如，考虑以下用法：

```js
ReactDOMRe.renderToElementWithId(<App> ("hello") </App>, "root");
```

这将导致友好的编译器错误，因为`App`组件要求其子元素是一个元组，但`App`组件的子元素不是元组。

```js
We've found a bug for you!

1 │ ReactDOMRe.renderToElementWithId(<App> ("hello") </App>, "root");

This has type:
  array('a)
But somewhere wanted:
  (string, string)
```

这非常强大。由于我们在编译时获得了这些保证，因此我们不必担心组件子元素的形状是否符合运行时检查。同样，我们保证了属性在编译时进行类型检查。重构组件变得不那么紧张，因为编译器会指导我们。更重要的是，由于 Reason 的强大类型推断，到目前为止我们还没有必须明确注释任何类型。

# 生命周期

ReasonReact 支持熟悉的 ReactJS 生命周期事件。当我们构建我们的应用程序时，我们将更仔细地查看一些生命周期事件，但是现在，让我们看看如何为`<App />`实现 ReactJS 的`componentDidMount`生命周期挂钩：

```js
let make = _children => {
  ...component,
  didMount: _self => Js.log("mounted"),
  render: _self => <div> {ReasonReact.string("hello")} </div>,
};
```

我们使用`didMount`而不是`componentDidMount`。同样，`didMount`只是组件的`make`函数返回的记录中的一个字段。`didMount`的类型是`self => unit`，它是一个接受`self`并返回`unit`的函数。由于它返回`unit`，它很可能会导致副作用，在我们的示例中确实如此。在浏览器中运行结果会在控制台中记录`mounted`。

# 订阅助手

为了使编写清理代码更加方便和容易记忆，ReasonReact 提供了`self.onUnmount`，它可以直接在组件的`didMount`生命周期中使用（或者在任何可以访问`self`的地方）。这允许您将清理代码与其补充一起编写，而不是分开在`willUnmount`中：

```js
didMount: self => {
  let intervalId = Js.Global.setInterval(() => Js.log("hello!"), 1000);
  self.onUnmount(() => Js.Global.clearInterval(intervalId));
},
```

# 有状态组件

到目前为止，我们只使用了`ReasonReact.statelessComponent`模板。要创建一个有状态的组件，我们将组件模板切换为`ReasonReact.reducerComponent`，并覆盖其`make`函数返回的记录中的一些附加字段。很快我们将看到，我们还需要声明自定义类型定义以在这些附加字段中使用。它被称为`reducerComponent`，因为它具有状态、操作和内置的 reducer 的概念-就像 Redux 一样，只是状态、操作和 reducer 是局部的。

这里显示了一个简单的计数器组件，带有增加和减少当前计数的按钮：

```js
type state = int;

type action =
  | Increment
  | Decrement;

let component = ReasonReact.reducerComponent("App");

let make = _children => {
  ...component,
  initialState: () => 0,
  reducer: (action, state) =>
    switch (action) {
    | Increment => ReasonReact.Update(state + 1)
    | Decrement => ReasonReact.Update(state - 1)
    },
  render: self =>
    <>
      <button onClick={_event => self.send(Decrement)}>
        {ReasonReact.string("-")}
      </button>
      <span> {ReasonReact.string(string_of_int(self.state))} </span>
      <button onClick={_event => self.send(Increment)}>
        {ReasonReact.string("+")}
      </button>
    </>,
};
```

在这里使用了 ReactJS 片段语法（`<>`和`</>`）来包装`<button>`和`<span>`元素，而不添加不必要的 DOM 节点。

# 状态、动作和减速器

让我们来分解一下。在文件的顶部，我们看到了两个类型声明，一个是状态，一个是动作。`state`和`action`是一种约定，但您可以使用任何您喜欢的名称：

```js
type state = int;

type action =
  | Increment
  | Decrement;
```

就像在 Redux 中一样，事件触发动作，这些动作被发送到一个减速器，然后更新状态。接下来，按钮的点击事件触发一个“减量”动作，通过`self.send`发送到组件的减速器。记住，渲染函数将`self`作为其参数提供：

```js
<button onClick={_event => self.send(Increment)}>
  {ReasonReact.string("+")}
</button>
```

`state`类型声明定义了我们状态的形状。在这种情况下，我们的状态只是一个保存组件当前计数的整数。组件的初始状态是`0`：

```js
initialState: () => 0,
```

`initialState`需要一个类型为`unit => state`的函数。

当被动作触发时，减速器函数接受该动作以及当前状态，并返回一个新状态。在当前动作上使用模式匹配，并使用`ReasonReact.Update`返回一个新状态：

```js
reducer: (action, state) =>
  switch (action) {
  | Increment => ReasonReact.Update(state + 1)
  | Decrement => ReasonReact.Update(state - 1)
  },
```

为了帮助您的 ReasonReact 应用程序为即将到来的 ReactJS Fiber 发布做好准备，确保`减速器`中的一切都是纯的。间接触发副作用的一种方法是使用`ReasonReact.UpdateWithSideEffects`：

```js
reducer: (action, state) =>
  switch (action) {
  | Increment =>
    ReasonReact.UpdateWithSideEffects(
      state + 1,
      (_self => Js.log("incremented")),
    )
  | Decrement => ReasonReact.Update(state - 1)
  },
```

`减速器`的返回值必须是以下变体构造函数之一：

+   `ReasonReact.NoUpdate`

+   `ReasonReact.Update(state)`

+   `ReasonReact.SideEffects(self => unit)`

+   `ReasonReact.UpdateWithSideEffects(state, self => unit)`

我们可以从我们的副作用中触发新的动作，因为我们再次提供了`self`：

```js
reducer: (action, state) =>
  switch (action) {
  | Increment =>
    ReasonReact.UpdateWithSideEffects(
      state + 1,
      (
        self =>
          Js.Global.setTimeout(() => self.send(Decrement), 1000) |> ignore
      ),
    )
  | Decrement => ReasonReact.Update(state - 1)
  },
```

增加后，`减速器`触发一个副作用，在一秒后触发“减量”动作。

# 重构

现在，让我们想象我们现在需要我们的有状态组件在计数达到 10 时显示一条祝贺用户的消息，一旦消息显示出来，用户可以通过点击关闭按钮关闭消息。多亏了我们乐于助人的编译器，我们可以按照以下步骤进行操作：

1.  更新`state`的形状

1.  更新可用的`动作`

1.  通过编译器错误进行步骤

1.  更新`render`函数

编译器消息将提醒我们更新组件的初始状态和减速器。由于我们现在还需要跟踪是否显示消息，让我们将`state`的形状更改为这样：

```js
type state = {
  count: int,
  showMessage: bool
};
```

对于我们的动作，让我们将`增量`和`减量`合并为一个接受`int`的构造函数，我们将有一个新的构造函数来切换消息：

```js
type action =
  | UpdateCount(int)
  | ToggleMessage;
```

现在，我们不再有`增量`和`减量`，而是有`UpdateCount`，它包含一个表示当前计数变化量的整数。

编译后，我们看到一个友好的错误提示，告诉我们之前的“减量”动作找不到：

```js
We've found a bug for you!
24 | render: self =>
25 | <>
26 | <button onClick={_event => self.send(Decrement)}>
27 | {ReasonReact.string("-")}
28 | </button>
The variant constructor Decrement can't be found.
- If it's defined in another module or file, bring it into scope by:
- Annotating it with said module name: let food = MyModule.Apple
- Or specifying its type: let food: MyModule.fruit = Apple
- Constructors and modules are both capitalized. Did you want the latter?
Then instead of let foo = Bar, try module Foo = Bar.
```

在`render`函数中，用`UpdateCount(+1)`替换`增量`，用`UpdateCount(-1)`替换`减量`：

```js
render: self =>
  <>
    <button onClick={_event => self.send(UpdateCount(-1))}>
      {ReasonReact.string("-")}
    </button>
    <span> {ReasonReact.string(string_of_int(self.state))} </span>
    <button onClick={_event => self.send(UpdateCount(1))}>
      {ReasonReact.string("+")}
    </button>
  </>,
```

再次编译，我们被告知在我们的减速器中，`增量`不属于类型`动作`。让我们更新我们的减速器来处理`UpdateCount`和`ToggleMessage`。如果我们忘记了一个构造函数，编译器会让我们知道减速器中的 switch 表达式不是穷尽的：

```js
reducer: (action, state) =>
  switch (action) {
  | UpdateCount(delta) =>
    let count = state.count + delta;
    ReasonReact.UpdateWithSideEffects(
      {...state, count},
      (
        self =>
          if (count == 10) {
            self.send(ToggleMessage);
          }
      ),
    );
  | ToggleMessage =>
    ReasonReact.Update({...state, showMessage: !state.showMessage})
  },
```

关于前面的代码片段，有几件事情需要提到：

+   在`UpdateCount`中，我们声明了一个反映新计数的绑定`count`。

+   我们使用`...`来覆盖状态记录的一部分。

+   多亏了记录标点符号的支持，我们可以写`{...state, count}`而不是`{...state, count: count}`。

+   `UpdateCount`正在使用`UpdateWithSideEffects`触发一个`ToggleMessage`动作，当计数达到 10 时；我们也可以这样做：

```js
| UpdateCount(delta) =>
  let count = state.count + delta;
  ReasonReact.Update(
    if (count == 10) {
      {count, showMessage: true};
    } else {
      {...state, count};
    },
  );
```

我更喜欢使用`UpdateWithSideEffects`，这样`UpdateCount`只需要关心它的计数字段，如果需要更新其他字段，`UpdateCount`可以触发正确的操作，而不需要知道如何发生。

在这里编译后，我们得到一个有趣的编译器错误：

```js
We've found a bug for you!

16 | switch (action) {
17 | | UpdateCount(delta) =>
18 | let count = state.count + delta;
19 | ReasonReact.UpdateWithSideEffects(
20 | {...state, count},

This has type:
  int
But somewhere wanted:
  state
```

编译器在第 18 行（之前显示）的`state.count`中看到`state`，将其视为`int`类型而不是`state`类型。这是因为我们的渲染函数使用`string_of_int(self.state)`而不是`string_of_int(self.state.count)`。在更新我们的渲染函数以反映这一点后，我们得到另一个类似的消息，抱怨类型`int`和类型`state`不兼容。这是因为我们的初始状态仍然返回`0`而不是`state`类型的记录。

更新初始状态后，代码最终成功编译：

```js
initialState: () => {count: 0, showMessage: false},
```

现在，我们准备更新渲染函数，在计数达到 10 时显示消息：

```js
render: self =>
  <>
    <button onClick={_event => self.send(UpdateCount(-1))}>
      {ReasonReact.string("-")}
    </button>
    <span> {ReasonReact.string(string_of_int(self.state.count))} </span>
    <button onClick={_event => self.send(UpdateCount(1))}>
      {ReasonReact.string("+")}
    </button>
    {
      if (self.state.showMessage) {
        <>
          <p>
            {ReasonReact.string("Congratulations! You've reached ten!")}
          </p>
          <button onClick={_event => self.send(ToggleMessage)}>
            {ReasonReact.string("close")}
          </button>
        </>;
      } else {
        ReasonReact.null;
      }
    }
  </>,
```

由于`if/else`在 Reason 中是一个表达式，我们可以在 JSX 中使用它来渲染标记或`ReasonReact.null`（类型为`ReasonReact.reactElement`）。

# 实例变量

虽然我们的示例在第一次计数达到 10 时正确显示消息，但没有阻止我们的`ToggleMessage`操作在`reducer`中的`UpdateCount`情况下再次触发。如果用户达到 10，然后递减然后递增，消息将再次切换。为了确保`UpdateCount`只触发一次`ToggleMessage`操作，我们可以在状态中使用**实例变量**。

在 ReactJS 中，每当状态发生变化时，组件都会重新渲染。在 ReasonReact 中，实例变量永远不会触发重新渲染，并且可以正确地放置在组件的状态中。

让我们添加一个实例变量来跟踪用户是否已经看到消息：

```js
type state = {
  count: int,
  showMessage: bool,
  userHasSeenMessage: ref(bool)
};
```

# Ref 和可变记录

ReasonReact 实例变量和普通状态变量之间的区别在于使用`ref`。之前，我们看到`state.userHasSeenMessage`的类型是`ref(bool)`而不是`bool`。这使得`state.userHasSeenMessage`成为一个实例变量。

由于`ref`只是具有可变字段的记录类型的语法糖，让我们首先讨论可变记录字段。

要允许记录字段可变，需要在字段名称前加上`mutable`。然后，可以使用`=`运算符就地更新这些字段：

```js
type ref('a) = {
  mutable contents: 'a
};

let foo = {contents: 5};
Js.log(foo.contents); /* 5 */
foo.contents = 6;
Js.log(foo.contents); /* 6 */
```

然而，类型声明已经包含在 Reason 的标准库中，所以我们可以省略它，前面的代码的其余部分仍然可以工作，声明它会遮蔽原始类型声明。我们可以通过用不可变记录遮蔽`ref`类型来证明这一点：

```js
type ref('a) = {contents: 'a};

let foo = {contents: 5};
Js.log(foo.contents); /* 5 */
foo.contents = 6;
Js.log(foo.contents); /* 6 */
```

编译器出现以下错误：

```js
We've found a bug for you!

The record field contents is not mutable
```

除了具有内置的类型定义之外，`ref`还具有一些内置函数。即`ref`用于创建类型为`ref`的记录，`^`用于获取`ref`的内容，`:=`用于设置`ref`的内容：

```js
type foo = ref(int);

let foo = ref(5);
Js.log(foo^); /* 5 */
foo := 6;
Js.log(foo^); /* 6 */
```

让我们回到我们的 ReasonReact 示例，让我们使用我们的新的`userHasSeenMessage`实例变量。在更新状态的形状之后，我们还需要更新组件的初始状态：

```js
initialState: () => {
  count: 0,
  showMessage: false,
  userHasSeenMessage: ref(false),
},
```

现在，我们的代码再次编译，我们可以更新`reducer`以使用这个实例变量：

```js
reducer: (action, state) =>
  switch (action) {
  | UpdateCount(delta) =>
    let count = state.count + delta;
    if (! state.userHasSeenMessage^ && count == 10) {
      state.userHasSeenMessage := true;
      ReasonReact.UpdateWithSideEffects(
        {...state, count},
        (self => self.send(ToggleMessage)),
      );
    } else {
      ReasonReact.Update({...state, count});
    };
  | ToggleMessage =>
    ReasonReact.Update({...state, showMessage: !state.showMessage})
  },
```

现在，消息被正确显示一次。

# 导航菜单

让我们将我们迄今为止学到的东西作为基础，创建一个具有导航菜单和客户端路由的应用程序。在触摸设备上，用户将能够滑动关闭菜单，并且菜单将实时响应用户的触摸。如果用户在菜单关闭超过 50%时滑动然后释放，菜单将关闭；否则，它将保持打开状态。唯一的例外是，如果用户以足够高的速度关闭菜单，它将始终关闭。

我们将在本书的其余部分中使用这个应用程序。要跟随，克隆 GitHub 存储库并导航到代表本章开头的目录：

```js
git clone https://github.com/PacktPublishing/ReasonML-Quick-Start-Guide.git
cd ReasonML-Quick-Start-Guide
cd Chapter03/app-start
npm install
```

让我们花点时间看看我们要处理的内容。您将看到以下目录结构：

```js
├── bsconfig.json
├── package-lock.json
├── package.json
├── src
│   ├── App.re
│   ├── App.scss
│   ├── Index.re
│   ├── Index.scss
│   ├── img
│   │   └── icon
│   │   ├── arrow.svg
│   │   ├── chevron.svg
│   │   └── hamburger.svg
│   └── index.html
└── webpack.config.js
```

我们的`bsconfig.json`设置为将编译后的`.bs.js`文件放在`lib/es6/src`中，并且我们已经配置 webpack 来查找`lib/es6/src/Index.bs.js`作为入口点。

运行`npm install`，然后运行`npm start`，以在监视模式下使用 bsb 和 webpack 为我们的应用提供服务，地址为`http://localhost:3000`。

目前，我们的应用程序显示一个带有汉堡图标的蓝色导航栏。单击图标会打开菜单，单击菜单外部会关闭菜单。

在`App.re`中，我们的状态目前是一个单字段记录，用于跟踪菜单的状态：

```js
type state = {isOpen: bool};
```

我们有一个动作：

```js
type action =
  | ToggleMenu(bool);
```

我们的 reducer 负责更新菜单的状态：

```js
reducer: (action, _state) =>
  switch (action) {
  | ToggleMenu(isOpen) => ReasonReact.Update({isOpen: isOpen})
  },
```

尽管 Reason 支持记录 pun，但对于单字段记录，它不起作用，因为 Reason 将`{isOpen}`视为块而不是记录。

我们的渲染函数渲染一个带有条件类名的`<div />`元素，具体取决于当前状态：

```js
<div
  className={"App" ++ (self.state.isOpen ? " overlay" : "")}
  onClick={
    _event =>
      if (self.state.isOpen) {
        self.send(ToggleMenu(false));
      }
  }>
```

`App.scss`使用`overlay`类来在导航菜单打开时只显示一个深色叠加层：

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
  ...
}
```

注意`transition`属性是如何为`.App:after`和`.App.overly:after`定义的，前者包括对`transform`属性的`450ms`延迟的过渡，而后者则移除了该过渡。这样做的效果是即使菜单关闭，也能实现平滑的过渡。

# 绑定

让我们检查`App.re`顶部对 JavaScript 的`require`函数的绑定。由于我们将在第四章中深入研究 BuckleScript，*BuckleScript，Belt 和互操作性*，让我们推迟讨论细节，只简要看一下这个绑定在做什么：

```js
[@bs.val] external require: string => string = "";

require("../../../src/App.scss");
```

`external`关键字创建一个新的绑定，类似于`let`关键字。绑定到 JavaScript 的`require`函数后，只要我们使用 BuckleScript 编译器，就可以在 Reason 中使用它。我们用它来要求`App.scss`以及一些图片。检查编译输出`lib/es6/src/App.bs.js`显示，前面的 Reason 代码编译为以下内容：

```js
require("../../../src/App.scss");
```

Webpack 会处理剩下的事情。

# 事件

由于顶层`<div />`元素有一个点击事件处理程序，总是关闭菜单，其子元素的任何点击也会触发该顶层点击事件处理程序。为了允许菜单打开（或保持打开），我们需要在某些子元素的点击事件上调用`event.stopPropagation()`。

在 ReasonReact 中，我们可以使用`ReactEvent`模块来实现这一点：

```js
onClick=(event => ReactEvent.Mouse.stopPropagation(event))
```

`ReactEvent`模块有子模块对应于 ReactJS 的合成事件的每一个：

+   剪贴板事件

+   组合事件

+   键盘事件

+   焦点事件

+   表单事件

+   鼠标事件

+   指针事件

+   选择事件

+   触摸事件

+   UI 事件

+   滚轮事件

+   媒体事件

+   图像事件

+   动画事件

+   过渡事件

有关 ReactJS 合成事件的更多信息，请访问[`reactjs.org/docs/events.html`](https://reactjs.org/docs/events.html)。

要从触摸事件中获取诸如`event.changedTouches.item(0).clientX`之类的值，我们使用 ReasonReact 和 BuckleScript 的组合。

# Js.t 对象

BuckleScript 允许我们使用`##`语法访问任意 JavaScript 对象字段。我们可以在任何`Js.t`类型上使用语法，这是一个用于任意 JavaScript 对象的 Reason 类型。我们将在第四章中了解更多关于这个和其他互操作特性的信息，*BuckleScript，Belt 和互操作性*。

由于`ReactEvent.Touch.changedTouches(event)`返回一个普通的 JavaScript 对象，我们可以使用以下方法访问其字段：

```js
/* App.re */
ReactEvent.Touch.changedTouches(event)##item(0)##clientX
```

查看编译输出，我们看到这就是我们想要的：

```js
/* App.bs.js */
event.changedTouches.item(0).clientX
```

我们将使用这个来为我们的菜单添加触摸功能，以便用户可以滑动菜单关闭并在滑动时看到菜单移动。

# 添加动作

让我们首先为`TouchStart`、`TouchMove`和`TouchEnd`添加操作：

```js
type action =
  | ToggleMenu(bool)
  | TouchStart(float)
  | TouchMove(float)
  | TouchEnd;
```

我们只需要`TouchStart`和`TouchMove`的触摸事件的`clientX`属性。

让我们在顶层`<div />`组件上添加事件监听器：

```js
render: self =>
  <div
    className={"App" ++ (self.state.isOpen ? " overlay" : "")}
    onClick={
      _event =>
        if (self.state.isOpen) {
          self.send(ToggleMenu(false));
        }
    }
    onTouchStart={
      event =>
        self.send(
          TouchStart(
            ReactEvent.Touch.changedTouches(event)##item(0)##clientX,
          ),
        )
    }
    onTouchMove={
      event =>
        self.send(
          TouchMove(
            ReactEvent.Touch.changedTouches(event)##item(0)##clientX,
          ),
        )
    }
    onTouchEnd={_event => self.send(TouchEnd)}>
```

在我们的 reducer 中，暂时只记录那些`clientX`值：

```js
reducer: (action, state) =>
  switch (action) {
  | ToggleMenu(isOpen) => ReasonReact.Update({isOpen: isOpen})
  | TouchStart(clientX) =>
    Js.log2("Start", clientX);
    ReasonReact.NoUpdate;
  | TouchMove(clientX) =>
    Js.log2("Move", clientX);
    ReasonReact.NoUpdate;
  | TouchEnd =>
    Js.log("End");
    ReasonReact.NoUpdate;
  },
```

为了找出用户滑动的整体方向，我们需要该滑动的第一个和最后一个`clientX`值。菜单应该按照第一个和最后一个`clientX`值的差值移动，但只有在用户滑动的方向会关闭菜单的情况下才移动。

我们的状态现在包括一个`touches`记录，其中包含第一个和最后一个`clientX`值的值：

```js
type touches = {
  first: option(float),
  last: option(float),
};

type state = {
  isOpen: bool,
  touches,
};
```

由于我们不能嵌套记录类型定义，我们单独定义`touches`类型，并将其包含在`state`中。您会注意到`state.touches.first`的类型是`option(float)`，因为用户可能没有使用触摸设备，或者用户尚未进行交互。

改变我们状态的形状需要我们同时改变初始状态：

```js
initialState: () => {
  isOpen: false,
  touches: {
    first: None,
    last: None,
  },
},
```

在 reducer 中，如果菜单是打开的，我们在`TouchStart`情况下使用一个新的记录更新`state.touches`，但在`TouchMove`情况下，我们只更新`state.touches.last`。如果菜单当前没有打开，将返回`ReasonReact.NoUpdate`：

```js
reducer: (action, state) =>
  switch (action) {
  | ToggleMenu(isOpen) => ReasonReact.Update({...state, isOpen})
  | TouchStart(clientX) =>
    if (state.isOpen) {
      ReasonReact.Update({
        ...state,
        touches: {
          first: Some(clientX),
          last: None,
        },
      });
    } else {
      ReasonReact.NoUpdate;
    }
  | TouchMove(clientX) =>
    if (state.isOpen) {
      ReasonReact.Update({
        ...state,
        touches: {
          ...state.touches,
          last: Some(clientX),
        },
      });
    } else {
      ReasonReact.NoUpdate;
    }
  | TouchEnd => ReasonReact.NoUpdate
  },
```

我们很快将使用这个状态来有条件地在`<nav />`元素上设置内联样式。

# 内联样式

在 ReasonReact 中，我们可以通过`ReactDOMRe.Style.make`添加内联样式，它接受 CSS 属性作为可选的标记参数。由于它们都是可选的，传递`unit`是调用该函数所必需的：

```js
style={ReactDOMRe.Style.make(~backgroundColor="yellow", ())}
```

将这个应用到我们的`<nav />`元素上，我们可以根据状态中是否有第一个和最后一个触摸来有条件地添加样式：

```js
style={
  switch (self.state.touches) {
  | {first: Some(x), last: Some(x')} =>
    ReactDOMRe.Style.make(
      ~transform=
        "translateX("
        ++ string_of_float(x' -. x > 0.0 ? 0.0 : x' -. x)
        ++ "0px)",
      ~transition="none",
      (),
    )
  | _ => ReactDOMRe.Style.make()
  }
}
```

在`transform`属性中，我们使用`"0px"`进行连接，而不仅仅是`"px"`，因为`float`类型总是包含小数点，但可能用户滑动的距离恰好是一百像素，`transform: translateX(100.px)`不是有效的 CSS，但`transform: translateX(100.0px)`是。

在触摸设备上运行这个程序，我们能够根据用户的滑动来改变菜单的位置。现在，让我们专注于 reducer 中的`TouchEnd`情况。暂时，如果用户将菜单滑动关闭不到一半，我们将设置菜单保持打开状态，否则关闭。如果`state.touches.last`是`None`，那么用户没有滑动，我们不更新`state`：

```js
| TouchEnd =>
  if (state.isOpen) {
    let x = Belt.Option.getWithDefault(state.touches.last, 0.0);
    if (x < 300.0 /. 2.0) {
      ReasonReact.UpdateWithSideEffects(
        {
          ...state,
          touches: {
            first: None,
            last: None,
          },
        },
        (self => self.send(ToggleMenu(false))),
      );
    } else {
      ReasonReact.Update({
        ...state,
        touches: {
          first: None,
          last: None,
        },
      });
    };
  } else {
    ReasonReact.NoUpdate;
  }
```

注意，我们将`state.touches`重置为一个新的记录，其中包含`{first: None, last: None}`，这将导致`<nav />`元素上的样式属性为空。

当前的实现假设导航的宽度为`300px`。我们可以使用 React ref 来获取对 DOM 节点的引用，然后获取它的`clientWidth`，而不是假设宽度。

# React ref

React ref 只是`state`的一个实例变量：

```js
type state = {
  isOpen: bool,
  touches,
  width: ref(float),
};
```

我们通过将`ref`属性设置为`self.handle((ref, self) => ...)`的结果来在`<nav />`元素上附加 React ref：

```js
ref={
  self.handle((ref, self) =>
    self.state.width :=
      (
        switch (Js.Nullable.toOption(ref)) {
        | None => 0.0
        | Some(r) => ReactDOMRe.domElementToObj(r)##clientWidth
        }
      )
  )
}
```

由于在 JavaScript 中，React ref 可能为`null`，我们将其转换为一个选项，并对其值进行模式匹配。

React ref 的类型取决于它是 DOM 元素还是 React 组件。前者的类型是`Dom.element`，后者的类型是`ReasonReact.reactRef`。要将`ReasonReact.reactRef`转换为 JavaScript 对象，使用`ReasonReact.refToJsObj`而不是`ReactDOMRe.domElementToObj`。

然后，在 reducer 中，我们可以使用`state.width`代替`300.0`作为菜单的宽度。由于`TouchStart`和`TouchMove`操作总是在菜单打开时更新状态，`<App />`组件总是重新渲染，这导致我们的 React ref 函数重新运行，我们可以合理地确定菜单的宽度始终是正确的。

# 速度

为了获得用户滑动的速度，我们还需要存储当前时间以及触摸事件的`clientX`。让我们绑定到浏览器的`performance.now()`方法：

```js
[@bs.val] [@bs.scope "performance"] external now: unit => float = "";
```

我们还将在`touches`类型中为触摸的当前时间腾出一些空间：

```js
type touches = {
  first: option((float, float)),
  last: option((float, float)),
};
```

在减速器中，我们将`Some(clientX)`更改为`Some((clientX, now()))`。

现在，我们可以计算用户在`TouchEnd`情况下的滑动速度：

```js
| TouchEnd =>
  if (state.isOpen) {
    let (x, t) =
      Belt.Option.getWithDefault(state.touches.first, (0.0, 0.0));
    let (x', t') =
      Belt.Option.getWithDefault(state.touches.last, (0.0, 0.0));
    let velocity = (x' -. x) /. (t' -. t);
    let state = {
      ...state,
      touches: {
        first: None,
        last: None,
      },
    };
    if (velocity < (-0.3) || x' < state.width^ /. 2.0) {
      ReasonReact.UpdateWithSideEffects(
        state,
        (self => self.send(ToggleMenu(false))),
      );
    } else {
      ReasonReact.Update(state);
    };
  } else {
    ReasonReact.NoUpdate;
  }
```

我觉得每毫秒-0.3 像素的速度对我来说感觉不错，但是随意使用任何对你来说感觉正确的值。

请注意，我们可以使用模式匹配来解构`(x, t)`，这会在作用域中创建两个绑定。此外，`x'`是 Reason 中有效的绑定名称，通常发音为*x prime*。最后，请注意我们的状态被遮蔽以防止编写重复的代码。

为了完成速度功能，我们在渲染函数中更新`style`属性，以将`state.touches.first`和`state.touches.last`都视为元组：

```js
style=(
  switch (self.state.touches) {
  | {first: Some((x, _)), last: Some((x', _))} =>
    ReactDOMRe.Style.make(
      ~transform=
        "translateX("
        ++ string_of_float(x' -. x > 0.0 ? 0.0 : x' -. x)
        ++ "0px)",
      ~transition="none",
      (),
    )
  | _ => ReactDOMRe.Style.make()
  }
)
```

现在，打开菜单时，菜单对触摸作出了很好的响应-非常酷！

# 客户端路由

ReasonReact 附带了一个内置路由器，位于`ReasonReact.Router`模块中。它非常不具有偏见，因此非常灵活。公共 API 只有四个函数：

+   `ReasonReact.Router.watchUrl: (url => unit) => watcherID`

+   `ReasonReact.Router.unwatchUrl: watcherID => unit`

+   `ReasonReact.Router.push: string => unit`

+   `ReasonReact.Router.dangerouslyGetInitialUrl: unit => url`

`watchUrl`函数开始监视 URL 的更改。更改后，将调用`url => unit`回调函数。`unwatchUrl`函数停止监视 URL。

`push`函数设置 URL，`dangerouslyGetInitialUrl`函数获取`url`类型的记录。`dangerouslyGetInitialUrl`函数仅在`didMount`生命周期钩子中使用，与`watchUrl`一起使用，以防止陈旧信息的问题。

`url`类型定义如下：

```js
type url = {
  path: list(string),
  hash: string,
  search: string,
};
```

我们将在第四章中学习更多关于`list`类型构造函数的知识，*BuckleScript，Belt 和互操作性*。`url`记录中的`path`字段是`list(string)`类型。如果`window.location.pathname`的值是`"/book/title/edit"`，那么`url.path`的值将是`["book", "title", "edit"]`，这是一个字符串列表。语法使它看起来像 JavaScript 数组，但有一些区别。简而言之，Reason 列表是不可变的同构单链表，意味着所有元素必须是相同类型的。

`watcherID`类型是一个**抽象类型**。我们将在第六章中学习更多关于抽象类型的知识，*CSS-in-JS（在 Reason 中）*。获取`watcherID`类型的值的唯一方法是作为`ReasonReact.Router.watchUrl`的返回值。

让我们创建一个路由器组件，它包装我们的`<App />`组件并为其提供`currentRoute`属性。以下内容受到了 Khoa Nguyen（`@thangngoc89`）示例的启发。

首先，让我们为`<Home />`，`<Page1 />`，`<Page2 />`和`<Page3 />`创建占位符组件。然后，在`Router.re`中，让我们创建一个表示路由的类型以及路由列表：

```js
type route = {
  href: string,
  title: string,
  component: ReasonReact.reactElement,
};

let routes = [
  {href: "/", title: "Home", component: <Home />},
  {href: "/page1", title: "Page1", component: <Page1 />},
  {href: "/page2", title: "Page2", component: <Page2 />},
  {href: "/page3", title: "Page3", component: <Page3 />},
];
```

每个路由都有一个`href`，`title`和一个相关的`component`，如果该路由是当前路由，则将在`<App />`中呈现。

# 当前路由

在`Index.re`中，让我们在路由器组件中包装`<App />`，并提供`currentRoute`属性：

```js
ReactDOMRe.renderToElementWithId(
  <Router.WithRouter>
    ...((~currentRoute) => <App currentRoute />)
  </Router.WithRouter>,
  "root",
);
```

在`Router.re`中，我们使用`module`语法定义了三个组件-`<WithRouter />`，`<Link />`和`<NavLink />`。由于每个文件也是一个模块，这三个组件嵌套在`Router`模块下，在`Index.re`中，我们需要告诉编译器在`Router`模块中查找`<WithRouter />`：

```js
module WithRouter = {
  type state = route;
  type action =
    | ChangeRoute(route);
  let component = ReasonReact.reducerComponent("WithRouter");
  let make = children => {
    ...component,
    didMount: self => {
      let watcherID =
        ReasonReact.Router.watchUrl(url =>
          self.send(ChangeRoute(urlToRoute(url)))
        );
      ();
      self.onUnmount(() => ReasonReact.Router.unwatchUrl(watcherID));
    },
    initialState: () =>
      urlToRoute(ReasonReact.Router.dangerouslyGetInitialUrl()),
    reducer: (action, _state) =>
      switch (action) {
      | ChangeRoute(route) => ReasonReact.Update(route)
      },
    render: self => children(~currentRoute=self.state),
  };
};
```

我们之前已经见过所有这些概念。`<WithRouter />`只是一个减速器组件。组件的状态是之前定义的相同路由类型，只有一个操作可以更改路由。一旦`<WithRouter />`被挂载，`ReasonReact.Router`开始监视 URL，每当 URL 更改时，就会触发`ChangeRoute`操作，这将调用减速器，然后更新状态，然后使用更新的`currentRoute`属性重新呈现`<App />`。

为了确保每当`<App />`接收到新的`currentRoute`属性时，我们都会关闭菜单，我们为`<App />`添加了一个`willReceiveProps`生命周期钩子：

```js
willReceiveProps: self => {...self.state, isOpen: false},
```

# 辅助函数

由于`ReasonReact.Router`的`url.path`是一个字符串列表，而我们的`Router.route.href`是一个字符串，我们需要一种将字符串转换为字符串列表的方法：

```js
let hrefToPath = href =>
  Js.String.replaceByRe([%bs.re "/(^\\/)|(\\/$)/"], "", href)
  |> Js.String.split("/")
  |> Belt.List.fromArray;
```

我们将在第四章中深入讨论 Reason 的管道运算符（`|>`）和 JavaScript 互操作性，*BuckleScript，Belt 和互操作性*。

我们还需要一种方法，将`url`转换为`route`，以便在初始状态和`watchUrl`的回调函数中使用：

```js
let urlToRoute = (url: ReasonReact.Router.url) =>
  switch (
    Belt.List.getBy(routes, route => url.path == hrefToPath(route.href))
  ) {
  | None => Belt.List.headExn(routes)
  | Some(route) => route
  };
```

在第四章中，*BuckleScript，Belt 和互操作性*，我们将更深入地了解 BuckleScript、Belt 和 JavaScript 互操作性。`urlToRoute`函数尝试在将其转换为字符串列表后，找到`routes`列表中`url.path`在结构上等于`route.href`的`route`。

如果不存在这样的`route`，它将返回`routes`列表中的第一个`route`，这是与`<Home />`组件相关联的`route`。否则，将返回匹配的`route`。

`<Link />`组件是一个简单的无状态组件，它呈现一个锚链接。请注意，单击处理程序会阻止默认的浏览器行为并更新 URL：

```js
module Link = {
  let component = ReasonReact.statelessComponent("Link");
  let make = (~href, ~className="", children) => {
    ...component,
    render: self =>
      <a
        href
        className
        onClick=(
          self.handle((event, _self) => {
            ReactEvent.Mouse.preventDefault(event);
            ReasonReact.Router.push(href);
          })
        )>
        ...children
      </a>,
  };
};
```

`<NavLink />`组件包装了`<Link />`组件，并提供了当前路由作为属性，它用于有条件地设置`active`类：

```js
module NavLink = {
  let component = ReasonReact.statelessComponent("NavLink");
  let make = (~href, children) => {
   ...component,
   render: _self =>
    <WithRouter>
      ...(
          (~currentRoute) =>
            <Link
              href className=(currentRoute.href == href ? "active" : "")>
              ...children
            </Link>
          )
    </WithRouter>,
  };
};
```

# 用法

现在我们已经定义了路由器，我们可以重写我们的导航菜单链接，使用`<NavLink />`组件而不是直接使用原始锚链接：

```js
<li>
  <Router.NavLink href="/">
    (ReasonReact.string("Home"))
  </Router.NavLink>
</li>
```

无论我们想要显示当前页面的标题，我们都可以简单地访问当前`route`上的`title`字段：

```js
<h1> (ReasonReact.string(currentRoute.title)) </h1>
```

而且，我们可以以类似的方式呈现路由的相关组件：

```js
<main> currentRoute.component </main>
```

重要的是要强调 ReasonReact 的路由器不会规定`watchUrl`的回调函数应该做什么。在我们的情况下，我们触发一个更新当前路由的动作，这只是一个任意的记录。路由类型完全可以是完全不同的东西。而且，并没有规定路由器应该是顶级组件的法律。在这里有很多创造性的空间，我个人很期待看到社区会有什么想法。

# 摘要

在本章中，我们看到 ReasonReact 是构建 React 组件的一种更简单、更安全的方式。在编译时，Reason 的类型系统强制执行正确的组件使用是一个巨大的胜利。此外，它使重构更安全、更便宜，也更愉快。ReasonReact *只是* Reason，就像 ReactJS *只是* JavaScript 一样。到目前为止，我们所做的一切都只是 Reason 和 ReasonReact，没有任何第三方库，比如 Redux 或 React Router。

正如我们将在第四章中看到的，*BuckleScript，Belt 和互操作性*，我们还可以选择在 Reason 中使用现有的 JavaScript（和 ReactJS）解决方案。熟悉了 BuckleScript、Belt 标准库和 JavaScript 互操作性后，我们将添加路由转换。
