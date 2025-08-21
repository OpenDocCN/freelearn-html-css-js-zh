# 第三章：显示数据

为了渲染视图，Aurelia 依赖于两个核心库：`aurelia-templating`，它提供了一个丰富且可扩展的模板引擎，以及`aurelia-binding`，它是一个现代且适应性强的数据绑定库。由于模板引擎依赖于数据绑定的抽象，这意味着可以使用 Aurelia 之外的数据绑定库，`aurelia-templating-binding`库充当了两者之间的桥梁。此外，`aurelia-templating-resources`建立在模板引擎之上，定义了一组标准行为和组件。

在本章中，我们将介绍数据绑定和模板的基础知识。我们将了解 Aurelia 提供的标准行为以及如何在视图中使用它们。

在渲染任何数据之前，首先必须获取它。大多数时候，单页网络应用程序依赖于某种类型的网络服务。因此，我们将了解 Fetch API 是什么，如何使用 Aurelia 的 Fetch 客户端，以及如何配置它。

最后，在关闭本章之前，我们将把我们新学到的知识应用到我们的联系人管理应用程序中，通过添加视图来显示联系人列表和联系人的详细信息。

# 模板基础

模板是一个根元素为`template`元素的 HTML 文件。它必须是有效的 HTML，因为模板引擎依赖于浏览器解析该文件并从中构建一个 DOM 树，然后引擎将遍历、分析和丰富行为。

这意味着适用于 HTML 文件的限制也适用于任何 Aurelia 模板。例如，`table`元素只能作为子元素包含某些类型的元素，如`thead`、`tbody`或`tr`。因此，以下模板在大多数浏览器中是非法的：

```js
<template> 
  <table> 
    <compose view="table-head.html"></compose>  
  </table> 
</template> 

```

在这里，我们想要使用在后面小节中介绍的`compose`元素，以插入包含表头的视图。由于`compose`不是`table`的有效子元素，大多数浏览器在解析 HTML 文件时会忽略它，因此模板引擎无法看到它。

为了克服这些限制，Aurelia 寻找一个特殊的`as-element`属性。这个属性作为元素名称的别名供模板引擎使用：

```js
<template> 
  <table> 
    <thead as-element="compose " view="table-head.html"></thead> 
  </table> 
</template> 

```

在这里，将元素名称从`compose`更改为`thead`使其成为一个合法的 HTML 片段，并添加`as-element="compose"`属性告诉 Aurelia 的模板引擎将这个`thead`元素视为一个`compose`元素。

## 视图资源

视图资源是可供模板引擎使用的工件，因此它们可以被模板使用。例如，自定义元素或值转换器是资源。

正如我们在前一章中看到的那样，资源可以全局加载，例如通过应用程序的`configure`方法、通过插件或通过特性。这样的资源对应用程序中的每个模板都可用。

### 本地加载资源

除了全局资源外，每个模板都有自己的资源集。一个需要使用一个在全球范围内不可用的资源的模板必须首先加载它。这通过使用`require`元素来实现：

`src/some-module/some-template.html`

```js
<template> 
  <require from="some-resource"></require> 
  <!-- at this point, some-resource is available to the template --> 
</template> 

```

`from`属性必须是要加载的资源的路径。在前一个示例中，路径是相对于代码根目录的，通常是指向`src`目录。这意味着`some-resource`预期直接位于`src`中。然而，路径也可以通过使用`.`前缀来使其相对于当前模板文件所在的目录：

`src/some-module/some-template.html`

```js
<template> 
  <require from="./some-resource"></require> 
</template> 

```

在这个例子中，`some-resource`预期位于`src/some-module`目录中。

此外，可以指定`as`属性。它用于更改资源的本地名称，以解决与其他资源的名字冲突，例如：

```js
<template> 
  <require from="some-resource" as="another-resource"></require> 
</template> 

```

在这个例子中，`some-resource`作为`another-resource`在模板中可用。

### 资源类型

默认情况下，预期资源是一个 JS 文件，在这种情况下，路径应该排除`.js`扩展名。例如，要加载从`sort.js`文件导出的值转换器，模板只需要求`sort`。无论资源类型是什么，值转换器、绑定行为、自定义元素等等，除了用作自定义元素的模板之外，都是正确的。

稍后我们将看到如何创建自定义元素。我们还将看到在没有视图模型的情况下如何创建仅包含模板的组件，当一个组件没有行为时。在这种情况下，作为资源加载时，仅包含模板的组件必须使用其完整文件名（包括其扩展名）来引用。例如，要加载一个名为`menu.html`的仅包含模板的组件，我们需要要求`menu.html`，而不仅仅是`menu`。否则，模板引擎将不知道它在寻找一个 HTML 文件而不是一个 JS 文件，并尝试加载`menu.js`。当我们开始将应用程序拆分为组件时，我们将看到这个的真实示例。

## 加载 CSS

除了本地加载模板资源外，`require`元素还可以用来加载样式表：

`src/my-component.html`

```js
<template> 
  <require from="./my-component.css"></require> 
</template> 

```

在这个例子中，`my-component.css`样式表将被加载并添加到文档的头部。

此外，可以使用`as="scoped"`属性将样式表的作用域限定在组件内：

`src/my-component.html`

```js
<template> 
  <require from="./my-component.css" as="scoped"></require> 
</template> 

```

在这个第二个例子中，如果`my-component`使用 ShadowDOM，并且浏览器支持它，样式表将被注入到 ShadowDOM 根部。否则，它将被注入到组件的视图中，并将`scoped`属性设置到`style`元素。

### 注意

影子 DOM 是一个 API，它允许我们在 DOM 中创建孤立的子树。这样的子树可以加载它们自己的样式表和 JavaScript，并与周围文档的冲突风险无关。这项技术对于无痛开发 Web 组件至关重要，但在撰写本文时，它仍然没有得到广泛浏览器的支持。

在`style`元素上的`scoped`属性告诉浏览器将样式表的作用域限制在包含元素及其后代元素上。这防止样式与其他文档部分发生冲突，而无需使用 ShadowDOM 根。它是 ShadowDOM 的有用替代品，但仍然没有得到广泛浏览器的支持。

# 数据绑定

数据绑定是将模板元素使用表达式与数据模型链接起来的动作，数据模型是一个 JS 对象。这个数据模型称为绑定上下文。这个上下文由 Aurelia 用于例如，暴露组件视图模型的属性和方法给其模板。此外，以下部分描述的一些行为会在其绑定上下文中添加信息。

## 绑定模式

数据绑定支持三种不同的模式：

+   **单向**：该表达式最初被评估，并且应用了说明并在视图中渲染。该表达式被观察，因此，无论其值如何变化，都可以重新评估，说明可以更新视图。它的变化只流向一个方向，从模型流向视图。

+   **双向**：与单向类似，但更新既可以从模型流向视图，也可以从视图流向模型：如果模板元素（如`input`）通过用户交互发生变化，模型就会被更新。它的变化是双向的，从模型流向视图，以及从视图流向模型。

    ### 注意

    当然，双向绑定限制了可以绑定的表达式的种类。只有可赋值表达式（通常是可以在 JavaScript 赋值指令的等号（`=`）操作符左侧使用的表达式）可以用于双向绑定。例如，你不能双向绑定到一个条件三元表达式或一个方法调用。

+   **一次性**：该表达式最初被评估，并且应用了说明，但该表达式不会被观察，因此任何在初始渲染后发生的模型变化都不会在视图中反映出来。绑定只会在视图渲染时从模型流向视图，只有一次。

## 字符串插值

构建模板时最基本的需求是显示文本。这可以通过使用字符串插值来实现：

```js
<template> 
  <h1>Welcome ${user.name}!</h1> 
</template> 

```

与 ES2015 的字符串插值类似，Aurelia 模板中的此类说明在`${`和`}`之间评估表达式，并将结果作为文本插入到 DOM 中。

字符串插值可以与更复杂的表达式一起使用：

```js
<template> 
  <h1>Welcome ${user ? user.name : 'anonymous user'}!</h1> 
</template> 

```

这里，我们使用三元表达式在绑定上下文中定义用户时显示用户的名字，否则显示通用信息。

它也可以用在属性内部：

```js
<template> 
  <h1 class="${isFirstTime ? ' emphasis' : ''}">Welcome!</h1> 
</template> 

```

在这个例子中，我们使用三元表达式在`model`的`isFirstTime`属性为真时，有条件地将`emphasis` CSS 类分配给`h1`元素。

默认情况下，字符串插值指令被绑定单向。这意味着，无论表达式的值如何变化，它都将被重新评估并在文档中更新。

## 数据绑定命令

当分析模板中的一个元素时，模板引擎会寻找带有数据绑定命令的属性。数据绑定命令是附加在属性后面，由点分隔的。它指导引擎对这个属性执行某种数据绑定。它有以下形式：`attribute.command="expression"`。

让我们来看看 Aurelia 提供的各种绑定命令。

### 绑定（bind）

`bind`命令将属性的值解释为表达式，并将这个表达式绑定到属性本身：

```js
<template> 
  <a href.bind="url">Go</a> 
</template> 

```

在这个例子中，绑定上下文中`url`属性的值将被绑定到`a`元素的`href`属性上。

`bind`命令是自适应的。它根据其目标元素和属性选择其绑定模式。默认情况下，它使用单向绑定，除非目标属性可以通过用户交互更改：例如`input`的`value`。在这种情况下，`bind`执行双向绑定，因此用户引起的变化会在模型上得到反映。

### 单向（One-way）

类似于`bind`，这个命令执行数据绑定，但不适应其上下文；绑定被强制为单向，无论目标类型如何。

### 双向（Two-way）

类似于`bind`，这个命令执行数据绑定，但不适应其上下文，绑定被强制为双向，无论目标类型如何。当然，将这个命令应用于自身无法更新的属性是毫无意义的。

### 一次性（One-time）

类似于`bind`，这个命令执行数据绑定，但强制进行一次性绑定，意味着在初始渲染之后模型中的任何变化都不会在视图中反映出来。

### 注意（Note）

你可能已经推断出一次性绑定比提供的实时绑定（单向和双向绑定）要轻量得多。确实，因为实时绑定需要观察，所以它更消耗 CPU 和内存。在一个大型应用程序中，如果有很多数据绑定指令，尽可能使用一次性绑定会在性能上产生巨大的不同。这就是为什么尽可能坚持使用一次性绑定，并在必要时才使用实时绑定被认为是一个好习惯。

### 触发器（trigger）

`trigger`命令将事件绑定到表达式，每次事件被触发时该表达式将被评估。`Event`对象作为`$event`变量可供表达式使用：

```js
<template> 
  <button click.trigger="open($event)">Open</button> 
</template> 

```

在这个例子中，`button` 的 `click` 事件将触发对绑定上下文的 `open` 方法的调用，并将 `Event` 对象传递给它。当然，使用 `$event` 是完全可选的；在这里，点击处理器可以是 `open()`，在这种情况下，`Event` 对象将被简单忽略。

请注意，事件名称不带 `on` 前缀：属性名称为 `click`，而不是 `onclick`。

### delegate

与直接在目标元素上附加事件处理器的 `trigger` 命令不同，`delegate` 利用事件委托，通过将一个处理程序附加到文档或最近的 ShadowDOM 根元素上来实现。这个处理程序会将事件分派到它们正确的目标，以便评估绑定的表达式。

与 `trigger` 一样，`Event` 对象作为 `$event` 变量 available 给表达式，属性名中必须省略 `on` 前缀。

### 注意

与直接附加到目标元素的事件处理程序相比，事件委托消耗的内存要少得多。就像一次性绑定与实时绑定一样，在小型应用程序中使用委托几乎不会注意到任何差异，但随着应用程序的大小增长，它可能会对内存足迹产生影响。另一方面，直接将事件处理程序附加到元素上是某些场景所必需的，尤其是当禁用冒泡时要触发自定义事件。

### call

`call` 命令用于将一个包含表达式的函数绑定到自定义属性或自定义元素的结构。当发生特定事件或满足给定条件时，这些自定义行为可以调用该函数来评估包装的表达式。

此外，自定义行为可以传递一个参数对象，此对象上的每个属性都将在此表达式的上下文中作为变量可用：

```js
<template> 
  <person-form save.call="createPerson(person)"></person-form> 
</template> 

```

在这里，我们可以想象有一个带有 `save` 属性的 `person-form` 自定义元素。在这个模板中，我们将 `person-form` 的 `save` 属性绑定到一个包含对模型 `createPerson` 方法的调用的函数，并向其传递表达式作用域上 `person` 变量的值。

然后 `person-form` 视图模型会在某个时刻调用这个函数。传递给这个函数的参数对象将在此表达式的上下文中可用：

```js
this.save({ person: this.somePersonData }); 

```

在这里，`person-form` 视图模型调用绑定在 `save` 属性上的函数，并向其传递一个 `person` 参数。

显然，这个命令在与原生 HTML 元素一起使用时是没有用的。

当我们覆盖自定义元素的制作时，我们会看到这个命令更具体的例子。

### ref

`ref` 命令可用于将 HTML 元素或组件部分的引用分配给绑定上下文。如果模板或视图模型需要访问模板中使用的 HTML 元素或组件的某部分，这可能很有用。

在以下示例中，我们首先使用 `ref` 将模型上的 `input` 元素分配为 `nameInput`，然后使用字符串插值实时显示这个 `input` 的 `value`：

```js
<template> 
  <input type="text" ref="nameInput"> 
  <p>Is your name really ${nameInput.value}?</p> 
</template> 

```

`ref`命令必须用于一组特定的属性：

+   `element.ref="someProperty"`（或`ref="someProperty"`的简写）将在绑定上下文中创建一个名为`someProperty`的属性，引用一个 HTML 元素

+   当放在具有`some-attribute`自定义属性的元素上时，`some-attribute.ref="someProperty"`将在绑定上下文中创建一个属性，名为`someProperty`，引用这个自定义属性的视图模型

+   当放在自定义元素上时，`view-model.ref="someProperty"`将在绑定上下文中创建一个属性，名为`someProperty`，引用自定义元素的视图模型

+   当放在自定义元素上时，`view.ref="someProperty"`将在绑定上下文中创建一个属性，名为`someProperty`，引用自定义元素的`view`实例

+   当放在自定义元素上时，`controller.ref="someProperty"`将在绑定上下文中创建一个属性，名为`someProperty`，引用自定义元素的两个`Controller`实例

## 绑定字面量

模板引擎将所有没有命令的属性的值解释为字符串。例如，一个`value="12"`属性将被解释为一个`'12'`字符串。

一些组件可能具有需要特定值类型的属性，例如布尔值、数字，甚至是数组或对象。在这种情况下，您应该使用数据绑定强制模板引擎将表达式解释为适当的类型，即使该表达式是一个字面值，且永远不会改变。例如，一个`value.bind="12"`属性将被解释为数字`12`。

类似地，一个`options="{ value: 12 }"`属性将被解释为一个`'{ value: 12 }'`字符串，而`options.bind="{ value: 12 }"`属性将被解释为一个包含`value`属性的数字`12`的对象。

当然，当数据绑定到字面值时，最好使用`one-time`而不是`bind`，以减少应用程序的内存占用。

## 使用内置绑定上下文属性

每个绑定上下文都公开了两个可能在某些场景中有用的属性：

+   `$this`: 一个自引用的属性。它包含对上下文本身的引用。它可能很有用，例如，将整个上下文传递给一个方法，或者在组合时将其注入到组件中。

+   `$parent`: 一个引用父级绑定上下文的属性。它可能很有用，例如，在`repeat.for`属性的作用域内访问被子上下文覆盖的父上下文的一个属性。它可以通过链式调用向上追溯到绑定上下文树更高层。例如，调用`$parent.$parent.$parent.name`将尝试访问曾祖上下文的`name`属性。

## 绑定到 DOM 属性

一些标准 DOM 属性通过 Aurelia 暴露为属性，因此它们可以进行数据绑定。

### `innerhtml`

`innerhtml`属性可用于数据绑定到元素的`innerHTML`属性：

```js
<template> 
  <div innerhtml.bind="htmlContent"></div> 
</template> 

```

在这个例子中，我们可以想象模型的`htmlContent`属性将包含 HTML 代码，这些代码与`div`的`innerHTML`属性数据绑定，将在`div`内部显示。

然而，这 HTML 不被认为是模板，所以它不会被模板引擎解释。如果它包含绑定表达式或需要指令，例如，它们不会被评估。

显示用户生成的 HTML 是一个众所周知的安全风险，因为它可能包含恶意脚本。强烈建议在向任何用户显示之前对这种 HTML 进行消毒。

`aurelia-templating-resources`附带一个简单的值转换器（我们将在本章后面看到值转换器是什么），名为`sanitizeHTML`，它用于这个目的。然而，强烈建议使用更完整的消毒器，如`sanitize-html`，可以在[`www.npmjs.com/package/sanitize-html`](https://www.npmjs.com/package/sanitize-html)找到。

### textcontent

`textcontent`属性可用于数据绑定到元素的`textContent`属性：

```js
<template> 
  <div textcontent.bind="text"></div> 
</template> 

```

在这个例子中，我们可以想象模型的`text`属性将包含一些文本，这些文本与`div`的`textContent`属性数据绑定，将在`div`内部显示。

与`innerhtml`类似，绑定到`textcontent`的文本不被认为是模板，所以它不会被模板引擎解释。

如前所述，`bind`命令试图检测它应该使用哪种绑定模式。因此，如果元素的`contenteditable`属性设置为`true`，则`textcontent`上的`bind`命令将使用双向绑定：

```js
<template> 
  <div textcontent.bind="text" contenteditable="true"></div> 
</template> 

```

在这个例子中，模型的`text`属性将被绑定到`div`的`textContent`属性并在`div`内部显示。另外，由于`div`的内容是可编辑的，用户对这部分内容所做的任何更改都将反映在模型的`text`属性上。

### style

`style`属性可用于数据绑定到元素的`style`属性。它可以绑定到一个字符串或一个对象：

```js
some-component.js 
export class ViewModel { 
  styleAsString = 'font-weight: bold; font-size: 20em;'; 
  styleAsObject = { 
    'font-weight': 'bold', 
    'font-size': '20em' 
  }; 
} 
some-component.html 
<template> 
  <div style.bind="styleAsString"></div> 
  <div style.bind="styleAsObject"></div> 
</template> 

```

另外，`style`属性可以使用字符串插值。然而，由于一些技术限制，它不支持 Internet Explorer。为了解决这个问题，并确保应用程序与 IE 兼容，在使用字符串插值时应使用`css`别名：

```js
<template> 
  <div css="color: ${color}; background-color: ${bgColor};"></div> 
</template> 

```

在这里，`div`将把其`color`和`background-color`样式与模型的`color`和`bgColor`属性数据绑定。

### scrolltop

`scrolltop`属性可用于绑定到元素的`scrollTop`属性。默认支持双向绑定，该属性可用于更改元素的的水平滚动位置，或者将其位置分配给上下文中的属性以便使用。

### scrollleft

`scrollleft`属性可以用来绑定到元素的`scrollLeft`属性。默认双向绑定，这个属性可以用来更改元素的垂直滚动位置，或者将其位置分配给上下文中的一个属性以便使用。

# 使用内置行为

核心库`aurelia-templating-resources`提供了一组标准行为，基于`aurelia-templating`构建，可以在 Aurelia 模板中使用。

## show

`show`属性根据它所绑定的表达式的值来控制元素的可见性：

```js
<template> 
  <p show.bind="hasError">An error occurred.</p> 
</template> 

```

在这个例子中，只有当模型的`hasError`属性为 truthy 时，`p`元素才会可见。

这个属性通过在文档头部或最近的 ShadowDOM 根中注入 CSS 类，并在元素应该隐藏时添加这个 CSS 类来工作。这个 CSS 类简单地将`display`属性设置为`none`。

## hide

这与`show`类似，但条件是倒置的：

```js
<template> 
  <p hide.bind="isValid">Form is invalid.</p> 
</template> 

```

在这个例子中，当模型的`isValid`属性为 truthy 时，`p`元素将隐藏。

除了倒置条件之外，这个属性的工作方式与`show`完全一样，并使用相同的 CSS 类。

## if

`if`属性与`show`非常相似。主要区别在于，当绑定的表达式评估为`false`值时，它不是简单地隐藏元素，而是完全将元素从 DOM 中移除。

```js
<template> 
  <p if.bind="hasError">An error occurred.</p> 
</template> 

```

由于`if`属性是一个模板控制器，因此可以直接放在嵌套的`template`元素上，以控制多个元素的可见性：

```js
<template> 
  <h1>Some title</h1> 
  <template if.bind="hasError"> 
    <i class="fa fa-exclamation-triangle"></i> 
    An error occurred. 
  </template> 
</template> 

```

在这个例子中，当`hasError`为`false`时，`i`元素及其后面的文本将从 DOM 中移除。

实际上，当条件为 falsey 时，它所附加的元素不仅会被从 DOM 中移除，它自己的行为和其子元素的行为也会被解绑。这是一个非常重要的区别，因为它有重大的性能影响。

以下示例中，假设`some-component`非常大，显示大量数据，有许多绑定，并且非常耗内存和 CPU。

```js
<template> 
  <some-component if.bind="isVisible"></some-component> 
</template> 

```

如果我们在这里用`show`替换`if`，整个组件层次结构的数据绑定仍然存在，即使它不可见也会消耗内存和 CPU。当使用`if`时，当`isVisible`变为`false`，组件将解绑，减少应用程序中的活动绑定数量。

另一方面，这意味着当条件变为 truthy 时，元素及其后代必须重新绑定。在条件经常开关的场景中，使用`show`或`hide`可能更好。选择`if`和`show`/`hide`之间的主要问题是平衡性能和用户体验的优先级，并且应该有真实的性能测试支持。

### 注意

模板控制器是一个属性，它将所作用的元素转换成模板，并控制这个模板的渲染方式。标准的属性`if`和`repeat`是模板控制器。

## repeat.for

`repeat`属性与特殊的`for`绑定命令一起使用时，可以用来为一系列值重复一个元素：

```js
<template> 
  <ul> 
    <li repeat.for="item of items">${item.title}</li> 
  </ul> 
</template> 

```

在这个例子中，`li`元素将被重复并绑定到`items`数组中的每个项目：

instead of an array, a `Set` object can also be data-bound too.

作为一个模板控制器，`repeat`实际上将所作用的元素转换成一个模板。然后为绑定序列中的每个项目渲染这个模板。对于每个项目，将创建一个子绑定上下文，在该上下文中，通过绑定表达式中`of`关键字左边的名称来使用项目本身。这意味着两件事：您可以随意命名项目变量，而且您可以在项目的上下文中使用它：

```js
<template> 
  <ul> 
    <li repeat.for="person of people"  
        class="${person.isImportant ? 'important' : ''}"> 
      ${person.fullName} 
    </li> 
  </ul> 
</template> 

```

在这个例子中，`li`元素将被插入到`ul`元素中，为`people`数组中的每个项目。对于每个`li`元素，将创建一个子上下文，将当前项目作为`person`属性暴露出来，如果对应的`person`的`isImportant`属性，那么`li`上会设置一个`important` CSS 类。每个`li`元素将包含其`person`的`fullName`，作为文本。

此外，由`repeat`创建的子上下文从周围上下文继承，所以`li`元素外的任何可用属性在内部都是可用的：

```js
<template> 
  <ul> 
    <li repeat.for="person of people"  
        class="${person === selectedPerson ? 'active' : ''}"> 
      ${person.fullName} 
    </li> 
  </ul> 
</template> 

```

在这里，根绑定上下文暴露了两个属性：一个`people`数组和`selectedPerson`。当每个`li`元素被渲染时，每个子上下文都可以访问当前的`person`以及父上下文。这就是`li`元素对于`selectedPerson`将具有`active` CSS 类的原因。

`repeat`属性默认使用单向绑定，这意味着绑定数组将被观察，对其进行的任何更改都将反映在视图中：

如果向数组中添加一个项目，模板将被渲染成一个额外的视图，并插入到 DOM 中的适当位置。

如果从数组中删除一个项目，相应的视图元素将被从 DOM 中删除。

### 绑定到地图

`repeat`属性能够与`map`对象一起使用，使用稍微不同的语法：

```js
<template> 
  <ul> 
    <li repeat.for="[key, value] of map">${key}: ${value}</li> 
  </ul> 
</template> 

```

在这里，`repeat`属性将为`map`中的每个条目创建一个分别具有`key`和`value`属性的子上下文，分别与`map`条目的`key`和`value`匹配。

重要的是要记住，这种语法只适用于`map`对象。在前一个示例中，如果`map`不是`Map`实例，那么在子绑定上下文中就不会定义`key`和`value`属性。

### 重复 n 次

`repeat`属性还可以在绑定到数值时使用标准语法重复一个模板给定次数：

```js
<template> 
  <ul class="pager"> 
    <li repeat.for="i of pageCount">${i + 1}</li> 
  </ul> 
</template> 

```

在这个例子中，假设 `pageCount` 是一个数字，`li` 元素将被重复多次，次数等于 `pageCount`，`i` 从 `0` 到 `pageCount - 1` 包括在内。

### 重复模板

如果需要重复的元素由多个没有每个项目单一容器的元素组成，可以在 `template` 元素上使用 `repeat`：

```js
<template> 
  <div> 
    <template repeat.for="item of items"> 
      <i class="icon"></i> 
      <p>${item}</p> 
    </template> 
  </div> 
</template> 

```

在这里，渲染后的 DOM 是一个包含交替 `i` 和 `p` 元素的 `div` 元素。

### 上下文变量

除了当前项目本身，`repeat` 还在子绑定上下文中添加了其他变量：

+   `$index`：项目在数组中的索引

+   `$first`：如果项目是数组的第一个，则为 `true`；否则为 `false`

+   `$last`：如果项目是数组的最后一个，则为 `true`；否则为 `false`

+   `$even`：如果项目的索引是偶数，则为 `true`；否则为 `false`

+   `$odd`：如果项目的索引是奇数，则为 `true`；否则为 `false`

## `with` 属性

`with` 属性通过它绑定的表达式创建一个子绑定上下文。它可以用来重新作用域模板的一部分，以防止访问路径过长。

例如，以下模板没有使用 `with`，在访问其属性时 `person` 被多次遍历：

```js
<template> 
  <div> 
    <h1>${person.firstName} ${person.lastName}</h1> 
    <h3>${person.company}</h3> 
  </div> 
</template> 

```

通过将顶层的 `div` 元素重新作用域为 `person`，可以简化对其属性的访问：

```js
<template> 
  <div with.bind="person"> 
    <h1>${firstName} ${lastName}</h1> 
    <h3>${company}</h3> 
  </div> 
</template> 

```

前面的例子很短，但你可以想象一个更大的模板如何从中受益。

此外，由于 `with` 创建了一个子上下文，外层作用域中所有可用的变量都将可在内层作用域中访问。

## 焦点属性

`focus` 属性可用于将元素的所有权与文档的焦点绑定到表达式。它默认使用双向绑定，这意味着当元素获得或失去 `focus` 时，它所绑定的变量将被更新。

以下代码片段摘自 `samples/chapter-3/binding-focus`：

```js
<template> 
  <input type="text" focus.bind="hasFocus"> 
</template> 

```

在 previous example，如果 `hasFocus` 是 `true`，则在渲染时 `input` 将会获得焦点。当 `hasFocus` 变为 `false` 值时，`input` 将会失去 `focus`。此外，如果用户将 `focus` 给予 `input`，`hasFocus` 将被设置为 `true`。类似地，如果用户将焦点从 `input` 移开，`hasFocus` 将被设置为 `false`。

## 组合元素

组合是将组件实例化并插入视图中的动作。`aurelia-templating-resources` 库导出一个 `compose` 元素，允许我们在视图中动态组合组件。

### 注意

以下各节中的代码片段摘自 `samples/chapter-3/composition`。在阅读本节时，你可以并行运行示例应用程序，这样你就可以查看组合的实时示例。

### 渲染视图模型

组件可以通过引用其视图模型的 JS 文件路径来组合：

```js
<template> 
  <compose view-model="some-component"></compose> 
</template> 

```

在这里，当渲染时，`compose` 元素将加载 `some-component` 的视图模型，实例化它，定位其模板，渲染视图，并将其插入到 DOM 中。

当然，`view-model`属性可以绑定或使用字符串插值：

```js
<template> 
  <compose view-model="widgets/${currentWidgetType}"></compose> 
</template> 

```

在这个例子中，`compose`元素将根据当前绑定上下文中的`currentWidgetType`属性的值，显示位于`widgets`目录中的组件。当然，这意味着当`currentWidgetType`发生变化时，compose 将交换组件（除非使用了一次性绑定）。

此外，`view-model`属性可以绑定到视图模型的实例：

`src/some-component.js`

```js
import {AnotherComponent} from 'another-component'; 

export class SomeComponent { 
  constructor() { 
    this.anotherComponent = new AnotherComponent(); 
  } 
} 

```

在这里，一个组件导入并实例化了另一个组件的视图模型。在其模板中，`compose`元素可以直接绑定到`AnotherComponent`的实例：

`src/some-component.html`

```js
<template> 
  <compose view-model.bind="anotherComponent"></compose> 
</template> 

```

当然，这意味着，如果`anotherComponent`被分配了一个新值，`compose`元素将相应地反应，并用新的一个替换掉之前的组件视图。

### 传递激活数据

当渲染组件时，组合引擎将尝试调用组件上存在的`activate`回调方法。与路由器的屏幕激活生命周期方法类似，这个方法可以被组件实现，以便在它们被渲染时可以行动。它还可以用来将激活数据注入组件。

`compose`元素也支持`model`属性。如果有的话，这个属性的值将被传递给组件的`activate`回调方法。

让我们想象一下以下的组件：

`src/some-component.js`

```js
export class SomeComponent { 
  activate(data) { 
    this.activationData = data || 'none'; 
  } 
} 
src/some-component.html 
<template> 
  <p>Activation data: ${activationData}</p> 
</template> 

```

当没有任何`model`属性时，这个组件将显示`<p>Activation data: none</p>`。然而，当像这样组成时，它会显示`<p>Activation data: Some parameter</p>`：

```js
<template> 
  <compose view-model="some-component" model="Some parameter"></compose> 
</template> 

```

当然，`model`可以使用字符串插值，也可以进行数据绑定，因此可以将复杂对象传递给组件的`activate`方法。

当与未实现`activate`方法的组件一起使用时，`model`属性的值将被直接忽略。

### 渲染模板

`compose`元素还可以简单地渲染一个模板，使用当前的绑定上下文：

```js
<template> 
  <compose view="some-template.html"></compose> 
</template> 

```

在这里，`some-template.html`将使用周围的绑定上下文渲染成一个视图。这意味着`compose`元素周围的任何变量也将对`some-template.html`可用。

当与`view-model`属性一起使用时，`view`属性将覆盖组件的默认模板。它可以在使用不同模板时复用视图模型的行为。

# 值转换器

在数据绑定的世界里，经常需要将数据在视图模型和视图之间进行转换，或者在双向绑定更新模型时将用户输入转换回来。

实现这种方法的一种方式是在视图模型中使用计算属性，以执行另一个属性值的来回转换。这种解决方案的缺点是，它不能跨视图模型复用。

在 Aurelia 中，值转换器解决了这个需求。值转换器是一个可以插入绑定表达式的对象。每次绑定需要评估表达式以渲染其结果，或者在双向绑定情况下更新模型时，转换器都作为拦截器，可以转换值。

## 使用值转换器

值转换器是视图资源。和 Aurelia 中的所有视图资源一样，为了在模板中使用，它必须被加载，要么通过一个`configure`函数全局加载，要么通过一个`require`元素局部加载。

### 注意

如果你不记得如何加载资源，请参阅*模板基础*部分。

在模板中，可以使用管道（`|`）操作符将值转换器包裹在一个数据绑定表达式周围：

```js
<template> 
  <div innerhtml.bind="htmlContent | sanitizeHTML"></div> 
</template> 

```

在这个示例中，我们使用了内置的`sanitizeHTML`值转换器来绑定`innerhtml`属性。这个值转换器会在绑定过程中被管道使用，并将会清除绑定值中的任何潜在危险元素。

值转换器实际上并不改变它们操作的绑定上下文值。它们仅仅作为拦截器，为绑定提供了一个用于渲染的替代值。

### 传递一个参数

值转换器可以接受参数，在这种情况下，它们必须在绑定表达式中使用冒号（`:`）分隔符指定。

让我们想象一个名为`truncate`的值转换器，它对字符串值起作用，同时期望一个`length`参数。在评估期间，它将提供的值截断到提供的长度（如果更长），并返回结果。这个转换器将如何使用：

```js
<template> 
  <h1>${title | truncate:20}</h1> 
</template> 

```

在这里，如果`title`超过 20 个字符，它将被截断到 20 个字符。否则，它将保持不变。

### 传递多个参数

可以向值转换器传递多个参数。只需继续使用冒号（`:`）分隔符。例如，如果`truncate`可以接受第二个参数，即在截断后的字符串后添加省略号，它将像这样传递：

```js
${title | truncate:20:'...'} 

```

### 传递上下文变量作为参数

绑定上下文中的变量也可以作为参数使用，在这种情况下，当这些变量中的任何一个发生变化时，绑定表达式将会重新评估。例如：

`some-component.js`

```js
export class ViewModel { 
  title = 'Some title'; 
  maxTitleLength = 2; 
} 
some-component.html 
<template> 
  <h1>${title | truncate:maxTitleLength}</h1> 
</template> 

```

在这里，字符串插值的价值取决于视图模型的`title`和`maxTitleLength`属性。每当它们中的一个发生变化时，表达式将会重新评估，`truncate`转换器将会重新执行，视图将会更新。

### 串联

值转换器可以被串联。在这种情况下，值通过转换器链进行管道，当评估表达式值时从左到右，当更新模型时从右到左：

```js
<template> 
  <h1>${title | truncate:20:'...' | capitalize}</h1> 
</template> 

```

在这个示例中，`title`首先会被截断，然后首字母大写后渲染。

## 实现一个值转换器

值转换器是一个必须实现至少以下方法之一的类：

+   `toView(value: any [, ...args]): any`：在评估绑定表达式后、渲染结果之前调用。`value`参数是绑定表达式的值。该方法必须返回转换后的值，它将传递给下一个转换器或渲染到视图中。

+   `fromView(value: any [, ...args]): any`：当更新模型的绑定目标值时调用，在将值分配给模型之前。`value`参数是绑定目标的值。该方法必须返回转换后的值，它将传递给下一个转换器或分配给模型。

如果值转换器使用参数，它们将以附加参数的形式传递给方法。例如，让我们想象一下值转换器的以下使用方式：

```js
${text | truncate:20:'...'} 

```

在这种情况下，`truncate`值转换器的`toView`方法预计会像这样：

```js
export TruncateValueConverter { 
  toView(value, length, ellipsis = '...') { 
    value = value || ''; 
    return value.length > length ? value.substring(0, length) + ellipsis : value; 
  } 
} 

```

在这里，`truncate`值转换器的`toView`方法除了它应用的`value`之外，还期望有一个`length`参数。它还接受一个名为`ellipsis`的第三个参数，有一个默认值。如果提供的`value`比提供的`length`长，该方法将截断它，附上`ellipsis`，然后返回这个新值。如果`value`不太长，它简单地返回它不变。

默认情况下，Aurelia 认为任何以`ValueConverter`结尾的作为资源加载的类都是一个值转换器。值转换器的名称将是类名，不包含`ValueConverter`后缀，驼峰命名。例如，一个名为`OrderByValueConverter`的类将作为`orderBy`值转换器提供给模板。

然而，当创建一个将包含在可重用插件或库中的转换器时，你不应依赖这个约定。在这种情况下，类应该用`valueConverter`装饰器装饰：

```js
import {valueConverter} from 'aurelia-framework'; 

@valueConverter('truncate') 
export Truncate { 
  // Omitted snippet... 
} 

```

这样，即使你的插件用户改变了默认的命名约定，你的类仍然会被 Aurelia 识别为值转换器。

# 绑定行为

绑定行为是视图资源，与值转换器相似，它们应用于表达式。然而，它们拦截绑定操作本身并访问整个绑定说明，因此可以修改它。这开辟了许多可能性。

## 使用绑定行为

要为一个绑定表达式添加绑定行为，它必须紧跟在表达式的末尾，使用`&`分隔符：

```js
${title & oneTime} 

```

当然，就像值转换器一样，绑定行为可以链接，在这种情况下，它们将从左到右执行：

```js
${title & oneWay & throttle} 

```

如果表达式还使用值转换器，绑定行为必须放在它们之后：

```js
${title | toLower | capitalize & oneWay & throttle} 

```

### 传递参数

就像值转换器一样，绑定行为也可以传递参数，使用相同的语法：

```js
${title & throttle:500} 

```

行为及其参数必须用冒号（:）分隔，参数之间也必须以同样的方式分隔：

```js
${title & someBehavior:p1:p2} 

```

## 内置绑定行为

`aurelia-templating-resources`库附带了许多绑定行为。让我们去发现它们。

### 注意

以下部分中的代码片段摘自`samples/chapter-3/binding-behaviors`。

### `oneTime`

`oneTime`行为使绑定变为单向 only。它可以用在字符串插值表达式上：

```js
<template> 
  <em>${quote & oneTime}</em> 
</template> 

```

在这里，视图模型的`quote`属性不会被观察，所以如果它发生变化，文本不会被更新。

此外，Aurelia 还附带了其他绑定模式的绑定行为：`oneWay`和`twoWay`。它们可以像`oneTime`一样使用。

### 节流

`throttle`绑定行为可用于限制视图模型更新的速率对于双向绑定或视图更新的速率对于单向绑定。换句话说，一个被 500 毫秒节流的绑定将至少在两个更新通知之间等待 500 毫秒。

```js
<template> 
  ${title & throttle} 
  <input value.bind="value & throttle"> 
</template> 

```

在这里，我们看到了这两个场景的例子。第一个`throttle`应用于字符串插值表达式，默认是单向的，当视图模型的`title`属性发生变化时，将节流视图中的文本更新。第二个应用于`input`元素的`value`属性的绑定，默认是双向的，当`input`的`value`发生变化时，将节流视图模型的`value`属性的更新。

`throttle`行为可以接受一个参数，表示更新之间的时差，以毫秒表示。然而，这个参数可以省略，默认使用 200 毫秒。

```js
<template> 
  ${title & throttle:800} 
  <input value.bind="value & throttle:800"> 
</template> 

```

在这里，我们有一个与之前相同的示例，但是绑定将被 800 毫秒节流。

事件也可以被节流。无论它是在`trigger`还是`delegate`绑定命令中使用，将事件分发到视图模型的节流将相应地节流：

```js
<template> 
  <div mousemove.delegate="position = $event & throttle:800"> 
    The mouse was last moved to (${position.clientX}, ${position.clientY}). 
  </div> 
</template> 

```

在这里，`div`元素的`mousemove`事件的处理程序将事件对象分配给视图模型的`position`属性。然而，这个处理程序将被节流，所以`position`将每 800 毫秒更新一次。

您可以在`samples/chapter-3/binding-behaviors`中看到`throttle`行为的一些示例。

### `debounce`

`debounce`绑定行为也是一种速率限制行为。它确保在给定延迟过去且没有更改的情况下不发送任何更新。

一个常见的用例是一个搜索输入，它会自动触发搜索 API 的调用。在用户每次输入后调用这样的 API 将是效率低下且消耗资源的。最好在用户停止输入后等待一段时间再调用搜索 API。这可以通过使用`debounce`来实现：

```js
<template> 
  <input value.bind="searchTerms & debounce"> 
</template> 

```

在这个例子中，视图模型将观察`searchTerms`属性，并在每次更改时触发搜索。`debounce`行为将确保在用户停止输入 200 毫秒后`searchTerms`才得到更新。

这意味着，当应用于双向绑定时，`debounce`限制了视图模型的更新速率。然而，当应用于单向绑定时，它限制了视图的更新速率：

```js
<template> 
  <input value.bind="text"> 
  ${text & debounce:500} 
</template> 

```

在这里，`debounce`应用于字符串插值表达式，所以只有当用户在输入中停止打字 500 毫秒后，显示的文本才会更新。这里的区别很重要。`text`属性仍然会实时更新。只有字符串插值绑定会被延迟。

就像`throttle`一样，`debounce`也可以应用于事件，使用触发器或委托绑定命令：

```js
<template> 
  <div mousemove.delegate="position = $event & debounce:800"> 
    The mouse was last moved to (${position.clientX}, ${position.clientY}). 
  </div> 
</template> 

```

在这里，`div`元素的`mousemove`事件的处理程序将事件对象分配给视图模型的`position`属性。然而，这个处理程序将被防抖，所以只有在鼠标在`div`上停止移动 800 毫秒后，`position`才会更新。

你可能会注意到，在之前的例子中，`throttle`和`debounce`都可以接受延迟，以毫秒表示，作为参数。省略时，延迟也默认为 200 毫秒。

### updateTrigger

`updateTrigger`绑定行为用于改变触发视图模型更新的事件。这意味着它只能与双向绑定一起使用，只能用于支持双向绑定的元素的属性，如`input`的`value`、`select`的`value`或具有`contenteditable="true"`的`div`的`textcontent`属性。

使用时，它期望事件名称作为参数，至少需要一个：

```js
<template> 
  <input value.bind="title & updateTrigger:'change':'input' "> 
</template> 

```

在这里，视图模型的`title`属性将在每次`input`触发`change`或`input`事件时更新。

实际上，`change`和`input`事件在 Aurelia 中是默认的触发器。除了这两个，`blur`、`keyup`和`paste`事件也可以用作触发器。

### signal

信号绑定行为允许程序化地触发绑定更新。这对于不可观察的绑定值或在特定时间间隔内必须刷新时特别有用。

让我们想象一个名为`timeInterval`的值转换器，它接收一个`Date`对象，计算输入日期和当前日期之间的时间间隔，并将这个时间间隔输出为用户友好的字符串，如`a minute ago`、`in 2 hours`或`3 years ago`。

由于结果取决于当前日期和时间，如果不定期刷新，它将很快过时。可以使用`signal`行为来实现这一点：

`src/some-component.html`

```js
<template> 
  Last updated ${lastUpdatedAt | timeInterval & signal:'now'} 
</template> 

```

在这个模板中，`lastUpdatedAt`使用`timeInterval`值转换器显示，其绑定被一个名为`now`的`signal`装饰。

`src/some-component.js`

```js
import {inject} from 'aurelia-framework'; 
import {BindingSignaler} from 'aurelia-templating-resources'; 

@inject(BindingSignaler) 
export class SomeComponent { 
  constructor(signaler) { 
    this.signaler = signaler; 
  } 

  activate() { 
    this.handle = setInterval(() => this.signaler.signal('now'), 5000); 
  } 

  deactivate() { 
    clearInterval(this.handle); 
  } 
} 

```

在视图模型中，在注入一个`BindingSignaler`实例并将其存储在实例变量中后，`activate`方法创建一个间隔循环，每 5 秒触发一个名为`now`的信号。每次触发信号时，模板中的字符串插值绑定都将更新，使得显示的时间间隔最多比当前时间晚 5 秒。当然，为了防止内存泄漏，间隔处理程序存储在实例变量中，并在组件停用时使用`clearInterval`函数销毁。

可以将多个信号名称作为参数传递给`signal`。在这种情况下，每次触发任何一个信号时，绑定都会刷新：

```js
<template> 
  <a href.bind="url & signal:'signal-1':'signal-2' ">Go</a> 
</template> 

```

此外，它只能用于字符串插值和属性绑定；信号一个`trigger`、`call`或`ref`表达式是没有意义的。

# 计算属性

高效的数据绑定是一个复杂的问题。Aurelia 的数据绑定库是适应性强的，并使用多种技术尽可能高效地观察视图模型和 DOM 元素。它在可能的情况下利用 DOM 事件和 Reflect API，在没有其他策略适用时才回退到脏检查。

### 注意

脏检查是一种使用超时循环反复评估表达式的观察机制，检查其值自上次评估以来是否发生变化，如果发生变化，则更新相关绑定。

计算属性是脏检查经常使用的一种场景。看这个例子：

```js
export class ViewModel { 
  get fullName() { 
    return `${this.firstName} ${this.lastName}`;
  } 
} 

```

当对`fullName`应用绑定时，Aurelia 无法知道其值是如何计算的，必须依赖脏检查来检测变化。在这个例子中，`fullName`的获取器评估速度很快，所以使用脏检查是绝对可以的。

然而，一些计算属性可能会最终执行重工作：例如从一个大型数组中搜索或聚合数据。在这种情况下，依赖脏检查意味着属性将每秒评估几次，这可能会使浏览器过载。

## 计算来自

`aurelia-binding`库导出一个`computedFrom`装饰器，可以用来解决这个问题。在装饰一个计算属性时，它通知绑定系统属性依赖于哪些依赖项来计算其结果。

```js
import {computedFrom} from 'aurelia-binding'; 

const items = [/* a static, huge list of items */]; 
export class ViewModel { 
  @computedFrom('searchTerm') 
  get matchCount() { 
    return items.filter(i => i.value.includes(this.searchTerm)).size; 
  } 
} 

```

在这里，为了观察`matchCount`，绑定系统会观察`searchTerm`。只有当它发生变化时，它才会重新评估`matchCount`。这比每秒多次评估属性的结果以检查其是否已更改要高效得多。

`computedFrom`装饰器接受访问路径作为依赖项，这些路径相对于它所在的类的实例是相对的：

```js
import {computedFrom} from 'aurelia-binding'; 

const items = [/* a static, huge list of items */]; 
export class ViewModel { 
  model = { 
    searchTerm: '...' 
  }; 

  @computedFrom('model.searchTerm') 
  get matchCount() { 
    return items.filter(i => i.value.includes(this.searchTerm)).size; 
  } 
} 

```

在这里，我们可以看到`matchCount`依赖于作为视图模型`model`属性存储的对象的`searchTerm`属性。

当然，它期望至少有一个依赖项作为参数传递。

`computedFrom`装饰器观察属性或路径。它无法观察数组的内容。这意味着以下示例将无法工作：

```js
import {computedFrom} from 'aurelia-binding'; 

export class ViewModel { 
  items = [/* a huge list of items, that can change during the lifetime of the component */]; 
  searchTerms = '...'; 

  @computedFrom('items', 'searchTerms') 
  get matchCount() { 
    return this.items.filter(i => i.value.includes(this.searchTerm)).size; 
  } 
} 

```

在这里，如果`items`得到一个项目的添加或删除，`computedFrom`不会检测到它，也不会重新评估`matchCount`。它能检测到的唯一情况是一个全新的数组是否被分配给`items`属性。

`computedFrom`装饰器在非常特定的情况下很有用。它不应该替代值转换器，因为那些是转换数据的首选方式。

# 从端点获取数据

## Fetch API

Fetch API 已被设计用于获取资源，包括通过网络。在撰写本文时，尽管其规范非常有前途，但仍未获得批准。然而，许多现代浏览器如 Chrome、Edge 和 Firefox 已经支持它。对于其他浏览器，需要一个 polyfill。

Fetch API 依赖于请求和响应的概念。这允许拦截管道在发送之前修改请求和接收时修改响应。它使得处理诸如认证和 CORS 之类的事情变得更容易。

在以下章节中，术语`Request`和`Response`指的是 Fetch API 的类。Mozilla 开发者网络有关于这个 API 的详尽文档：[`developer.mozilla.org/en-US/docs/Web/API/Fetch_API`](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)。

## 使用 Fetch 客户端

Aurelia 的 Fetch 客户端是一个围绕原生或 polyfilled Fetch API 的包装器。它支持默认请求配置，以及可插拔的拦截机制。它由一个名为`HttpClient`的类组成。这个类暴露了通过 HTTP 获取资源的方法。

## 配置

`HttpClient`类有一个`configure`方法。它期望的参数是一个回调函数，该函数接收一个配置对象，该对象暴露了可以用来配置客户端的方法：

+   `withBaseUrl(baseUrl: string)`：这为客户端设置了基础 URL。所有相对 URL 的请求都将相对于这个 URL 进行。

+   `withDefaults(defaults: RequestInit)`：这设置了传递给`Request`构造函数的默认属性。

+   `withInterceptor(interceptor: Interceptor)`：这为拦截管道添加了一个`Interceptor`对象。

+   `rejectErrorReponses()`：`fetch`方法返回一个`Response`对象的`Promise`。这个`Promise`只在发生网络错误或类似情况阻止请求完成时被拒绝。否则，无论服务器可能回答什么 HTTP 状态，`Promise`都会成功解决为`Response`。这个方法添加了一个拦截器，当响应的状态不是成功代码时拒绝`Promises`。HTTP 成功代码在`200`到`299`之间。

+   `useStandardConfiguration()`：标准配置包括`same-origin`凭据设置（有关此设置的更多信息，请参见官方 Fetch API 文档）和拒绝错误响应（请参阅前面的`rejectErrorResponses`方法）。

除了一个回调配置函数外，`configure`方法还可以直接传递一个`RequestInit`对象。在这种情况下，这个`RequestInit`对象将被用作所有请求的默认属性。

这意味着，如果我们有一个存储在`defaultProperties`变量中的`RequestInit`对象，下面的两行将执行完全相同的事情：

```js
client.configure(defaultProperties); 
client.configure(config => { config.withDefaults(defaultProperties); }); 

```

`RequestInit`对象对应于 Fetch API 的`Request`构造函数期望的第二个参数。它用于指定`Request`的各种属性。最常用的属性有：

+   `method`：HTTP 方法，例如 GET、POST

+   `headers`：包含请求的 HTTP 头的对象

+   `body`：请求体，例如一个`Blob`、`BufferSource`、`FormData`、`URLSearchParams`或`USVString`实例

    ### 注意

    我将让你查看官方文档以获取关于可用`Request`属性的更多详细信息。

正如你所看到的，一个`RequestInit`对象可以用来指定一个 HTTP 方法和请求体，因此我们将能够执行 POST 和 PUT 请求来创建和更新`person`对象。我们将在下一章看到这个例子，那时我们开始构建表单。

### 一个常见的陷阱

正如我们在第二章中看到的，*布局、菜单和熟悉*，DI 容器默认自动将所有类作为应用程序单例注册。这意味着，如果您的应用程序包含多个服务，它们依赖于应该是独立的`HttpClient`实例，并且各自配置其相应的`HttpClient`不同，您会遇到奇怪的问题。

让我们想象一下以下两个服务：

```js
import {inject} from 'aurelia-framework'; 
import {HttpClient} from 'aurelia-fetch-client'; 

@inject(HttpClient) 
export class ContactService { 
  constructor(http) { 
    this.http = http.configure(c => c.withBaseUrl('api/contacts')); 
  } 
} 

@inject(HttpClient) 
export class AddressService { 
  constructor(http) { 
    this.http = http.configure(c => c.withBaseUrl('api/addresses')); 
  } 
} 

```

在这里，我们有两个服务，分别名为`ContactService`和`AddressService`。它们在其构造函数中都作为`HttpClient`实例注入，并使用不同的基础 URL 配置自己的实例。

默认情况下，相同的`HttpClient`实例将被注入到两个服务中，因为 DI 容器默认认为它是应用程序的单例。您看到问题了吗？第二个服务创建后，将覆盖第一个服务的基础 URL，所以第一个服务最终将尝试对错误的 URL 执行 HTTP 调用。

这种场景有很多可能的解决方案。您可以使用`NewInstance`解析器在每个服务中强制注入一个新的实例：

```js
import {inject, NewInstance} from 'aurelia-framework'; 
import {HttpClient} from 'aurelia-fetch-client'; 

@inject(NewInstance.of(HttpClient)) 
export class ContactService { 
  constructor(http) { 
    this.http = http.configure(c => c.withBaseUrl('api/contacts')); 
  } 
} 

@inject(NewInstance.of(HttpClient)) 
export class AddressService { 
  constructor(http) { 
    this.http = http.configure(c => c.withBaseUrl('api/addresses')); 
  } 
} 

```

另一个解决方案可能是将`HttpClient`类作为您的应用程序主要`configure`方法中的瞬态注册：

```js
import {HttpClient} from 'aurelia-fetch-client'; 

export function configure(config) { 
  config.container.registerTransient(HttpClient); 
  //Omitted snippet... 
} 

```

## 拦截器

拦截器是在 HTTP 调用过程中的不同时间截取请求和响应的对象。一个`Interceptor`对象可以实现以下任意回调方法：

+   `request(request: Request): Request|Response|Promise<Request|Response>`：在请求被发送之前调用。它可以修改请求，或者返回一个新的请求。它还可以返回一个响应来短路剩余的过程。在这种情况下，下一个拦截器的`request`方法将被跳过，并且将使用响应，好像请求已经被发送一样。支持`Promise`。

+   `requestError(error: any): Request|Response|Promise<Request|Response>`：当一个拦截器的`request`方法抛出错误时调用。它可能重新抛出错误以传播它，或者返回一个新的请求或响应以从失败中恢复。支持`Promise`。

+   `response(response: Response, request?: Request): Response|Promise<Response>`：在响应被接收之后调用。它可以修改响应，或者返回一个新的响应。支持`Promise`。

+   `responseError(error: any, request?: Request): Response|Promise<Response>`：当一个拦截器的`response`方法抛出错误时调用。它可能重新抛出错误以传播它，或者返回一个新的响应以从失败中恢复。支持`Promise`。

例如，我们可以定义以下的拦截器类：

```js
export class BearerAuthorizationInterceptor { 
  constructor(token) { 
    this.token = token; 
  } 

  request(request) { 
    request.headers.set('Authorization', `Bearer ${this.token}`); 
  } 
} 

```

这个拦截器期望一个`Bearer`认证令牌在它的构造函数中被传递。当添加到一个 Fetch 客户端时，它会在每个请求中添加一个`Authorization`头，允许一个已经认证的用户访问一个受保护的端点。

# 我们的应用程序

至此，我们已经涵盖了我们将需要的应用程序的下一步：查询我们的 HTTP 端点、显示联系人列表以及允许导航到给定联系人的详细信息。

为了使我们的应用程序更具吸引力，我们将利用 Font Awesome，一个提供可缩放矢量图标的 CSS 库。让我们首先安装它：

```js
> npm install font-awesome --save

```

接下来，我们需要将其包含在我们的应用程序中：

`index.html`

```js
<head>  
  <!-- Omitted snippet --> 
  <link href="node_modules/font-awesome/css/font-awesome.min.css" rel="stylesheet"> 
</head> 

```

## 我们的联系人网关

我们本可以在我们的视图模型中直接进行 HTTP 调用。然而，这样做会模糊责任之间的界限。视图模型除了要负责数据展示（其主要任务）之外，还要负责调用、解析请求、处理错误以及最终缓存响应。

相反，我们将创建一个联系人网关类，它将负责从端点获取数据，将是可重用的，并且能够独立发展：

`src/contact-gateway.js`

```js
import {inject} from 'aurelia-framework'; 
import {HttpClient} from 'aurelia-fetch-client'; 
import {Contact} from './models'; 
import environment from './environment'; 

@inject(HttpClient) 
export class ContactGateway { 

  constructor(httpClient) { 
    this.httpClient = httpClient.configure(config => { 
      config 
        .useStandardConfiguration() 
        .withBaseUrl(environment.contactsUrl); 
    }); 
  } 

  getAll() {    
    return this.httpClient.fetch('contacts') 
      .then(response => response.json()) 
      .then(dto => dto.map(Contact.fromObject)); 
  } 

  getById(id) { 
    return this.httpClient.fetch(`contacts/${id}`) 
      .then(response => response.json()) 
      .then(Contact.fromObject); 
  } 
} 

```

在这里，我们首先声明一个构造函数期望一个`HttpClient`实例的类，这是 Aurelia 的 Fetch 客户端。在这个构造函数中，我们配置客户端，使其使用标准配置，我们在*配置*部分看到了这个配置，并使用`environment`对象的`contactsUrl`属性作为其基本 URL。这意味着所有带有相对 URL 的请求都将相对于这个 URL 进行。

我们的 contact gateway 暴露了两个方法：一个获取所有联系人，第二个通过其 ID 获取单个联系人。它们通过调用客户端的`fetch`方法来工作，该方法默认向提供的 URL 发送 GET 请求。在这里，由于 URL 是相对路径，它们将相对于在构造函数中配置的基 URL 进行转换。

当 HTTP 请求完成后，`fetch`返回的`Promise`被解决，然后在解决后的`Response`对象上调用`json`方法来反序列化响应体为 JSON。`json`方法也返回一个`Promise`，所以当这个第二个`Promise`解决时，我们将未类型的数据传输对象转换为稍后我们将编写的`Contact`类的实例。

这意味着，基于端点返回的内容，`getAll`返回一个`Contact`对象的数组`Promise`，而`getById`返回一个单个`Contact`对象的`Promise`。

### 先决条件

为了让这一切正常工作，我们需要做两件事。首先，我们将安装 Fetch 客户端，通过在移动到应用程序目录后，在控制台中运行以下命令：

```js
npm install aurelia-fetch-client --save

```

### 注意

本书中编写的所有代码都在 Google Chrome 上运行过。如果你使用其他浏览器，你可能需要为各种 API（如 Fetch）安装 polyfill。

此外，你还需要让 Aurelia 打包器知道这个库。在`aurelia_project/aurelia.json`中，在`build`下的`bundles`中，在名为`vendor-bundle.js`的包定义中，将`aurelia-fetch-client`添加到`dependencies`数组中：

`aurelia_project/aurelia.json`

```js
{ 
  //Omitted snippet... 
  "build": { 
    //Omitted snippet ... 
    "bundles": { 
      //Omitted snippet ... 
      { 
        "name": "vendor-bundle.js", 
        //Omitted snippet ... 
        "dependencies": [ 
          "aurelia-fetch-client", 
          //Omitted snippet ... 
        ] 
      } 
    } 
  } 
} 

```

这是为了让`aurelia-fetch-client`库与其他库一起被捆绑，以便我们的应用程序可以使用它。

最后，`contactsUrl`属性在`environment`配置对象中默认不存在。我们需要添加它：

`aurelia_project/environments/dev.js`

```js
export default { 
  debug: true, 
  testing: true, 
  contactsUrl: 'http://127.0.0.1:8000/', 
}; 

```

在这里，我们将默认在哪个 URL 上运行我们的端点的 URL 分配给`contactsUrl`属性。在现实世界中，我们还会将其设置在`stage.js`和`prod.js`中，因此我们的端点为所有环境配置。我将留下这个作为读者的练习。

## 显示联系人

现在让我们在我们的空`contact-list`组件中添加一些代码。我们将利用我们新的`ContactGateway`类来获取联系人列表并显示它。

`src/contact-list.js`

```js
import {inject} from 'aurelia-framework'; 
import {ContactGateway} from './contact-gateway'; 

@inject(ContactGateway) 
export class ContactList { 

  contacts = []; 

  constructor(contactGateway) { 
    this.contactGateway = contactGateway; 
  } 

  activate() { 
    return this.contactGateway.getAll() 
      .then(contacts => { 
        this.contacts.splice(0); 
        this.contacts.push.apply(this.contacts, contacts); 
      }); 
  } 
} 

```

在这里，我们首先在`contact-list`组件的视图模型中注入了一个`ContactGateway`实例。在`activate`方法中，我们使用`getAll`获取联系人，一旦`Promise`解决，我们确保清除联系人数组；然后我们将加载的联系人添加到其中，以便它们可供模板使用。

在这种情况下，更改数组被视为比覆盖整个`contacts`属性更好的做法，因为视图中的`repeat.for`绑定观察数组实例的更改，但不观察属性本身，所以如果`contacts`在视图渲染后覆盖，视图不会刷新。

您可能注意到了`getAll`返回的`Promise`是如何在`activate`中返回的。这使得对 HTTP 端点的调用作为屏幕激活生命周期的一部分运行。如果没有这

我们还需要定义`Contact`类。它在列表和详细视图中会有用的计算属性：

`src/models.js`

```js
export class Contact { 
  static fromObject(src) { 
    return Object.assign(new Contact(), src); 
  } 

  get isPerson() { 
    return this.firstName || this.lastName; 
  } 

  get fullName() { 
    const fullName = this.isPerson  
      ? `${this.firstName} ${this.lastName}`  
      : this.company; 
    return fullName || ''; 
  } 
} 

```

这个类有一个名为`fromObject`的静态方法，它作为一个工厂方法。它期望一个源对象作为其参数，创建一个`Contact`的新实例，并将源对象的所有属性分配给它。此外，它定义了一个`isPerson`属性，如果联系人至少有一个名字或姓氏，则返回`true`，并在模板中用来区分人和公司。它还定义了一个`fullName`属性，如果联系人代表一个人，它将返回名字和姓氏，如果联系人是公司，它将返回公司名称。

现在，唯一缺少的是`contact-list`模板：

`src/contact-list.html`

```js
<template> 
  <section class="container"> 
    <h1>Contacts</h1> 
    <ul> 
      <li repeat.for="contact of contacts">${contact.fullName}</li> 
    </ul> 
  </section> 
</template> 

```

这里我们简单地将联系人渲染为无序列表。

你现在可以测试它：

```js
> au run --watch

```

### 注意

不要忘记通过在`api`目录中运行`npm start`来启动 HTTP 端点。当然，如果你之前没有运行过，你首先需要运行`npm install`来安装其依赖项。

如果你没有省略任何步骤，当你导航到 http://localhost:9000/时，你应该看到联系人列表出现。

## 对联系人进行分组和排序

目前，联系人列表很无聊。联系人以子弹列表的形式显示，甚至没有排序。我们可以通过按联系人名字的第一个字母分组并按字母顺序对这些组进行排序，大大提高这个屏幕的可使用性。这将使浏览列表和查找联系人变得更容易。

要实现这一点，我们有两个选择：我们可以在视图模型中先分组然后排序联系人，或者我们可以将此逻辑隔离在值转换器中，以便我们以后可以重新使用它们。我们将选择后者，因为它符合单一责任原则，并使我们的代码更加简洁。

### 创建 orderBy 值转换器

我们的`orderBy`值转换器将应用于一个数组，并期望其第一个参数为用于排序项目的属性名称。

我们的值转换器还可以接受一个可选的第二个参数，这将是一个排序方向，作为一个`'asc'`或`'desc'`字符串。省略时，排序顺序将升序。

`src/resources/value-converters/order-by.js`

```js
export class OrderByValueConverter { 
  toView(array, property, direction = 'asc') { 
    array = array.slice(0); 
    const directionFactor = direction == 'desc' ? -1 : 1;  
    array.sort((item1, item2) => { 
      const value1 = item1[property]; 
      const value2 = item2[property]; 
      if (value1 > value2) { 
        return directionFactor; 
      } else if (value1 < value2) { 
        return -directionFactor; 
      } else { 
        return 0; 
      } 
    }); 
    return array; 
  } 
} 

```

### 注意

一个重要的部分是在调用`sort`之前调用`slice`。它确保对数组进行复制，因为`sort`方法会修改它调用的数组。如果没有`slice`调用，原始数组将被修改。这是不好的；值转换器绝不应该修改其源值。这不是预期行为，因此这样的转换器会让使用它的开发者感到非常惊讶。

在设计值转换器时，你确实应该密切关注以避免此类副作用。

为了使这个新的转换器对模板可用，而不是每次需要时都手动`require`它，让我们在`resources`特性中加载它：

`src/resources/index.js`

```js
export function configure(config) { 
  config.globalResources([ 
    './value-converters/order-by', 
  ]); 
} 

```

你可以通过将`contact of contacts | orderBy:'fullName'`的`repeat.for`指令更改为`contact-list`模板中的新`firstLetter`属性来测试它。

### 创建 groupBy 值转换器

接下来，我们的`groupBy`值转换器将以几乎相同的方式工作；它将应用于数组，并期望一个参数，这个参数将是用于分组项目的属性的名称。它将返回一个对象数组，每个对象都包含两个属性：用作`key`的分组值和作为`items`数组的分组项目：

`src/resources/value-converters/group-by.js`

```js
export class GroupByValueConverter { 
  toView(array, property) { 
    const groups = new Map(); 
    for (let item of array) { 
      let key = item[property]; 
      let group = groups.get(key); 
      if (!group) { 
        group = { key, items: [] }; 
        groups.set(key, group); 
      } 
      group.items.push(item); 
    } 
    return Array.from(groups.values()); 
  } 
} 

```

这个值转换器还需要在`resources`特性的`configure`函数中加载。这个你自己来吧。

### 更新联系人列表

为了利用我们的值转换器，我们首先需要在`Contact`类中添加一个新属性：

`src/models.js`

```js
//Omitted snippet... 
export class Contact { 
  //Omitted snippet... 
  get firstLetter() { 
    const name = this.lastName || this.firstName || this.company; 
    return name ? name[0].toUpperCase() : '?'; 
  } 
} 

```

这个新的`firstLetter`属性取联系人的姓氏、名字或公司名字的第一个字母。它将用于将联系人分组在一起。

接下来，让我们丢弃我们之前的联系人列表模板，重新开始：

`src/contact-list.html`

```js
<template> 
  <section class="container"> 
    <h1>Contacts</h1> 
    <div repeat.for="group of contacts|groupBy:'firstLetter'|orderBy:'key'" 
         class="panel panel-default"> 
      <div class="panel-heading">${group.key}</div> 
      <ul class="list-group"> 
        <li repeat.for="contact of group.items|orderBy:'fullName'"    
            class="list-group-item"> 
          <a route-href="route: contact-details;  
                         params.bind: { id: contact.id }"> 
            <span if.bind="contact.isPerson"> 
              ${contact.firstName} <strong>${contact.lastName}</strong> 
            </span> 
            <span if.bind="!contact.isPerson"> 
              <strong>${contact.company}</strong> 
            </span> 
          </a> 
        </li> 
      </ul> 
    </div> 
  </section> 
</template> 

```

在这里，我们首先按照它们的`firstLetter`属性的值将联系人分组。`groupBy`转换器返回一个组对象的数组，然后根据它们的`key`属性进行排序并重复到面板上。对于每个组，以该组分的字母显示在面板标题中，然后按`fullName`属性对组中的联系人进行排序并显示在列表组中。对于每个联系人，都会渲染一个到其详细视图的链接，显示其人员或公司名称。

## 筛选联系人

即使联系人被分组和排序，找到特定的联系人可能仍然很麻烦，特别是如果用户不知道联系人的全名。我们添加一个搜索框，用于实时过滤联系人列表。

我们首先需要创建另一个值转换器来过滤联系人数组：

`src/resources/value-converters/filter-by.js`

```js
export class FilterByValueConverter { 
  toView(array, value, ...properties) { 
    value = (value || '').trim().toLowerCase(); 
    if (!value) { 
      return array; 
    } 
    return array.filter(item =>  
      properties.some(property =>  
        (item[property] || '').toLowerCase().includes(value))); 
  } 
} 

```

我们的`filterBy`值转换器期望一个第一个参数，这是要搜索的值。此外，它考虑以下参数是要搜索的属性。任何不在指定属性中包含搜索值的联系人将被过滤出结果。

### 注意

不要忘记在`resources`特性的`configure`函数中加载`filterBy`值转换器。

接下来，我们需要在`contact-list`模板中添加搜索框并应用我们的值转换器：

`src/contact-list.html`

```js
<template> 
  <section class="container"> 
    <h1>Contacts</h1> 

    <div class="row"> 
      <div class="col-sm-2"> 
        <div class="input-group"> 
          <input type="text" class="form-control" placeholder="Filter"  
                 value.bind="filter & debounce"> 
          <span class="input-group-btn" if.bind="filter"> 
            <button class="btn btn-default" type="button"  
                    click.delegate="filter = ''"> 
              <i class="fa fa-times"></i> 
              <span class="sr-only">Clear</span> 
            </button> 
          </span> 
        </div> 
      </div> 
    </div> 

    <div repeat.for="group of contacts 
                     | filterBy:filter:'firstName':'lastName':'company' 
                     | groupBy:'firstLetter'  
                     | orderBy:'key'" 
         class="panel panel-default"> 
      <!-- Omitted snippet... --> 
    </div> 
  </section> 
</template> 

```

在这里，我们首先添加一个搜索框，形式为一个`input`元素，其`value`绑定到`filter`属性。这个绑定是去抖的，所以属性将在用户停止输入 200 毫秒后才会更新。

另外，当`filter`不为空时，`input`旁边会显示一个按钮。点击这个按钮，简单地将`filter`分配为一个空字符串。

最后，我们在`repeat.for`绑定中将对`contacts`应用`filterBy`，传递`filter`作为搜索值，随后是`firstName`、`lastName`和`company`属性的名称，这些属性将被搜索。

### 注意

这里有趣的一点是，我们甚至没有在视图模型中声明`filter`属性。它只在视图中使用。由于它绑定到输入元素的值属性，默认情况下绑定是双向的，绑定只会将其值分配给视图模型。视图模型不需要知道这个属性。

## 联系人详细视图

如果你点击一个联系人，你应该在浏览器控制台看到一个错误。原因很简单：应该显示联系人详情的路由指的是一个`contact-details`组件，而这个组件还不存在。让我们来纠正这个问题。

### 视图模型

视图模型将利用我们之前编写的某些类：

`src/contact-details.js`

```js
import {inject} from 'aurelia-framework'; 
import {ContactGateway} from './contact-gateway'; 

@inject(ContactGateway) 
export class ContactDetails { 
  constructor(contactGateway) { 
    this.contactGateway = contactGateway; 
  } 

  activate(params, config) { 
    return this.contactGateway.getById(params.id) 
      .then(contact => { 
        this.contact = contact; 
        config.navModel.setTitle(contact.fullName); 
      }); 
  } 
} 

```

这段代码相当直接。视图模型期望在其构造函数中注入`ContactGateway`的一个实例，并实现`activate`生命周期回调方法。这个方法使用`id`路由参数并向网关请求适当的联系人对象。它返回网关的`Promise`，所以导航只有在联系人加载完成后才会完成。当这个`Promise`解决时，联系人对象被分配给视图模型的`contact`属性。此外，路由`config`对象用于动态将文档标题分配给联系人的`fullName`。

### 模板

联系人详情的模板很大，所以让我们将其分解为部分。你可以按照这一节逐步构建模板。

首先，让我们添加一个头，将显示联系人的图片和姓名：

```js
<template> 
  <section class="container"> 
    <div class="row"> 
      <div class="col-sm-2"> 
        <img src.bind="contact.photoUrl" class="img-responsive" alt="Picture"> 
      </div> 
      <template if.bind="contact.isPerson"> 
        <h1 class="col-sm-10">${contact.fullName}</h1> 
        <h2 class="col-sm-10">${contact.company}</h2> 
      </template>  
      <template if.bind="!contact.isPerson"> 
        <h1 class="col-sm-10">${contact.company}</h1> 
      </template> 
    </div> 
  </section> 
</template> 

```

模板的其余部分，应该放在关闭`section`标签之前，被一个带有`form-horizontal`类的`div`元素包含：

```js
<div class="form-horizontal"> 
  <!-- the rest of the template goes here. --> 
</div> 

```

在这个元素内部，我们首先显示联系人在创建和最后修改时的日期和时间：

```js
<div class="form-group"> 
  <label class="col-sm-2 control-label">Created on</label> 
  <div class="col-sm-10"> 
    <p class="form-control-static">${contact.createdAt}</p> 
  </div> 
</div> 

<div class="form-group"> 
  <label class="col-sm-2 control-label">Modified on</label> 
  <div class="col-sm-10"> 
    <p class="form-control-static">${contact.modifiedAt}</p> 
  </div> 
</div> 

```

接下来，如果联系人有生日，我们将显示联系人的生日：

```js
<div class="form-group" if.bind="contact.birthday"> 
  <label class="col-sm-2 control-label">Birthday</label> 
  <div class="col-sm-10"> 
    <p class="form-control-static">${contact.birthday}</p> 
  </div> 
</div> 

```

之后，我们将显示联系人的电话号码：

```js
<template if.bind="contact.phoneNumbers.length > 0"> 
  <hr> 
  <div class="form-group"> 
    <h4 class="col-sm-2 control-label">Phone numbers</h4> 
  </div> 
  <div class="form-group" repeat.for="phoneNumber of contact.phoneNumbers"> 
    <label class="col-sm-2 control-label">${phoneNumber.type}</label> 
    <div class="col-sm-10"> 
      <p class="form-control-static"> 
        <a href="tel:${phoneNumber.number}">${phoneNumber.number}</a> 
      </p> 
    </div> 
  </div> 
</template> 

```

在这里，块被包含在一个模板中，该模板仅当联系人至少有一个电话号码时才渲染。每个电话号码都显示其类型：家庭、办公室或移动电话等。

接下来的部分都会遵循与电话号码相同的模式。它们将显示联系人的电子邮件地址、地理位置和社交媒体资料：

```js
<template if.bind="contact.emailAddresses.length > 0"> 
  <hr> 
  <div class="form-group"> 
    <h4 class="col-sm-2 control-label">Email addresses</h4> 
  </div> 
  <div class="form-group"  
       repeat.for="emailAddress of contact.emailAddresses"> 
    <label class="col-sm-2 control-label">${emailAddress.type}</label> 
    <div class="col-sm-10"> 
      <p class="form-control-static"> 
        <a href="mailto:${emailAddress.address}"  
           target="_blank">${emailAddress.address}</a> 
      </p> 
    </div> 
  </div> 
</template> 

<template if.bind="contact.addresses.length > 0"> 
  <hr> 
  <div class="form-group"> 
    <h4 class="col-sm-2 control-label">Addresses</h4> 
  </div> 
  <div class="form-group" repeat.for="address of contact.addresses"> 
    <label class="col-sm-2 control-label">${address.type}</label> 
    <div class="col-sm-10"> 
      <p class="form-control-static">${address.number} ${address.street}</p> 
      <p class="form-control-static">${address.postalCode} ${address.city}</p> 
      <p class="form-control-static">${address.state} ${address.country}</p> 
    </div> 
  </div> 
</template> 

<template if.bind="contact.socialProfiles.length > 0"> 
  <hr> 
  <div class="form-group"> 
    <h4 class="col-sm-2 control-label">Social Profiles</h4> 
  </div> 
  <div class="form-group" repeat.for="profile of contact.socialProfiles"> 
    <label class="col-sm-2 control-label">${profile.type}</label> 
    <div class="col-sm-10"> 
      <p class="form-control-static"> 
        <a if.bind="profile.type === 'GitHub'"  
           href="https://github.com/${profile.username}"  
           target="_blank">${profile.username}</a> 
        <a if.bind="profile.type === 'Twitter'"  
           href="https://twitter.com/${profile.username}"  
           target="_blank">${profile.username}</a> 
      </p> 
    </div> 
  </div> 
</template> 

```

最后，如果有的话，我们将显示联系人的备注：

```js
<template if.bind="contact.note"> 
  <hr> 
  <div class="form-group"> 
    <label class="col-sm-2 control-label">Note</label> 
    <div class="col-sm-10"> 
      <p class="form-control-static">${contact.note}</p> 
    </div> 
  </div> 
</template> 

```

由于在组件的生命周期中加载的联系人永远不会改变，可以通过将所有`bind`命令替换为`one-time`命令，并将所有字符串插值装饰为`oneTime`绑定行为来大大改进此模板。我将把这个作为读者的练习留给读者。

# 概要

正如你所见，Aurelia 的数据绑定语言清晰简洁。它相当容易理解，即使对于不熟悉 Aurelia 的开发人员来说，模板也很容易理解。此外，它是适应性强的，使得编写高性能应用程序尽可能简单。

除了 Fetch 客户端的便利性，这些特质结合了值转换器和绑定行为系统的灵活性与可重用性，使得编写数据展示组件变得非常简单。

构建用于创建和编辑数据的形式并不比这更复杂。我们将在下一章中看到这一点，其中包括表单验证。
