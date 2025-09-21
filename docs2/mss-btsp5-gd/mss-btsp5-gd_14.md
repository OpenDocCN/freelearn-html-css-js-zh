# 第十一章：*第十一章*: 使用 Bootstrap 5 的高级 JavaScript 特性

在本章中，我们将学习一些可以与 Bootstrap 5 交互式组件一起使用的 JavaScript 高级特性。首先，我们将快速概述交互式组件，并查看它们在依赖项和初始化方面的共同要求。然后，我们将看到如何使用数据属性或 JavaScript 初始化交互式组件。接着，我们将看到如何使用数据属性或 JavaScript 定义选项，最后，我们将更详细地探讨如何使用 JavaScript 进行方法和事件处理。

在本章中，我们将涵盖以下主要主题：

+   Bootstrap 5 的交互式组件

+   初始化交互式组件

+   定义交互式组件的选项

+   使用 JavaScript 方法

+   使用 JavaScript 事件

# 技术要求

为了预览示例，你需要一个代码编辑器和浏览器。所有代码示例的源代码都可以在这里找到：

https://github.com/PacktPublishing/The-Missing-Bootstrap-5-Guide

# Bootstrap 5 的交互式组件

我们将从这个章节开始，先对 Bootstrap 5 的交互式组件进行概述。对于这里列出的每个组件，我们将看到它使用了哪个 JavaScript 文件，是否需要自定义初始化，它有哪些选项、方法和事件，是否有任何依赖项，对于其中的一些，我们还会提供一些额外的信息。在接下来的章节中，我们将更详细地探讨初始化、选项、方法和事件，因此这应该只被视为一个概述。列表的最后，还有一个总结，以突出最重要的差异。

## 折叠面板

此组件是使用后面描述的折叠组件构建的。

## 警告框

`alert.js`

**自定义初始化**: 否

**选项**: 无

`close`, `dispose`, `getInstance`, `getOrCreateInstance`

`close.bs.alert`, `closed.bs.alert`

**依赖项**: 无

此组件可以使用关闭按钮组件关闭。

## 按钮

`button.js`

**自定义初始化**: 否

**选项**: 无

`toggle`, `dispose`, `getInstance`, `getOrCreateInstance`

**事件**: 无

**依赖项**: 无

## 轮播图

`carousel.js`

**自定义初始化**: 否

`interval`, `keyboard`, `pause`, `ride`, `wrap`, `touch`

`cycle`, `pause`, `prev`, `next`, `nextWhenVisible`, `to`, `dispose`, `getInstance`, `getOrCreateInstance`

`slide.bs.carousel`, `slid.bs.carousel`

**依赖项**: 无

## 折叠组件

`collapse.js`

**自定义初始化**: 否

`parent`, `toggle`

`toggle`, `show`, `hide`, `dispose`, `getInstance`, `getOrCreateInstance`

`show.bs.collapse`, `shown.bs.collapse`, `hide.bs.collapse`, `hidden.bs.collapse`

**依赖项**: 无

## 下拉菜单

`dropdown.js`

**自定义初始化**: 否

`boundary`, `reference`, `display`, `offset`, `autoClose`, `popperConfig`

`toggle`, `show`, `hide`, `update`, `dispose`, `getInstance`, `getOrCreateInstance`

`show.bs.dropdown`, `shown.bs.dropdown`, `hide.bs.dropdown`, `hidden.bs.dropdown`

**依赖**: Popper

此组件需要外部第三方库 Popper 进行定位。它必须在 `bootstrap.js` 之前包含，或者使用包含 Popper 的 `bootstrap.bundle.js`。

## 列表组

此组件使用 `tab.js` 基于列表组创建标签导航。请参考关于导航和标签的详细信息，它们使用相同的 JavaScript 文件。

## Modal

`modal.js`

**自定义初始化**: 否

`backdrop`, `keyboard`, `focus`

`toggle`, `show`, `hide`, `handleUpdate`, `dispose`, `getInstance`, `getOrCreateInstance`

`show.bs.modal`, `shown.bs.modal`, `hide.bs.modal`, `hidden.bs.modal`, `hidePrevented.bs.modal`

**依赖**: 无

## 导航和标签

`tab.js`

**自定义初始化**: 否

**选项**: 无

`show`, `dispose`, `getInstance`, `getOrCreateInstance`

`show.bs.tab`, `shown.bs.tab`, `hide.bs.tab`, `hidden.bs.tab`

**依赖**: 无

此组件使用的 JavaScript 文件也用于列表组组件。

## Offcanvas

`offcanvas.js`

**自定义初始化**: 否

`backdrop`, `keyboard`, `scroll`

`toggle`, `show`, `hide`, `getInstance`, `getOrCreateInstance`

`show.bs.offcanvas`, `shown.bs.offcanvas`, `hide.bs.offcanvas`, `hidden.bs.offcanvas`

**依赖**: 无

## Popover

`popover.js`

**自定义初始化**: 是

`animation`, `container`, `content`, `delay`, `html`, `placement`, `selector`, `template`, `title`, `trigger`, `fallbackPlacements`, `boundary`, `customClass`, `sanitize`, `allowList`, `sanitizeFn`, `offset`, `popperConfig`

`show`, `hide`, `toggle`, `dispose`, `enable`, `disable`, `toggleEnabled`, `update`, `getInstance`, `getOrCreateInstance`

`show.bs.popover`, `shown.bs.popover`, `hide.bs.popover`, `hidden.bs.popover`, `inserted.bs.popover`

`tooltip.js`, Popover

此组件需要 `tooltip.js` 作为依赖。它还需要外部第三方库 Popper 进行定位。后者必须在 `bootstrap.js` 之前包含，或者使用包含 Popper 的 `bootstrap.bundle.js`。

## Scrollspy

`scrollspy.js`

**自定义初始化**: 否

`rootMargin`, `smoothScroll`, `target`

`refresh`, `dispose`, `getInstance`, `getOrCreateInstance`

`activate.bs.scrollspy`

**依赖**: 无

## Toast

`toast.js`

**自定义初始化**: 是

`animation`, `autohide`, `delay`

`show`, `hide`, `dispose`, `getInstance`, `getOrCreateInstance`

`show.bs.toast`, `shown.bs.toast`, `hide.bs.toast`, `hidden.bs.toast`

**依赖**: 无

## Tooltip

`tooltip.js`

**自定义初始化**: 是

`animation`, `container`, `delay`, `html`, `placement`, `selector`, `template`, `title`, `trigger`, `fallbackPlacements`, `boundary`, `customClass`, `sanitize`, `allowList`, `sanitizeFn`, `offset`, `popperConfig`

`show`, `hide`, `toggle`, `dispose`, `enable`, `disable`, `toggleEnabled`, `update`, `getInstance`, `getOrCreateInstance`

`show.bs.tooltip`, `shown.bs.tooltip`, `hide.bs.tooltip`, `hidden.bs.tooltip`, `inserted.bs.tooltip`

**依赖**: 弹出

此组件需要外部第三方库 Popper 来定位。它必须在 `bootstrap.js` 之前包含，或者通过使用包含 Popper 的 `bootstrap.bundle.js`。

弹出组件需要 `tooltip.js` 文件作为依赖。

我们刚刚列出并概述了所有 15 个以某种方式使用 JavaScript 的组件。现在，我们将总结这些内容，并突出组件之间的最重要的差异。因此，在接下来的四个标题下，我们将看到哪些组件脱颖而出。

### 需要初始化

**组件**: 弹出、通知、工具提示

由于性能原因，这些组件不能使用数据属性初始化。因此，它们必须使用自定义 JavaScript 来初始化。我们将在 *初始化交互式组件* 部分看到如何操作。

### 有依赖

**组件**: 下拉菜单、弹出、工具提示

这些组件都需要外部第三方库 Popper 来定位。它必须在 `bootstrap.js` 之前包含，或者通过使用包含 Popper 的 `bootstrap.bundle.js`。

此外，弹出组件还需要工具提示组件的 `tooltip.js` JavaScript 文件作为依赖。

### 没有选项

**组件**: 警告、按钮、导航和标签页

这些组件没有任何选项。

### 没有事件

**组件**: 按钮

此组件没有任何事件。

# 初始化交互式组件

在本节中，我们将看到我们可以以两种不同的方式初始化交互式组件。首先，我们将看到如何在 HTML 中使用数据属性来初始化，这是默认方式，你可能已经在使用了。然后，我们将看到如何使用 JavaScript 来初始化。我们可以使用数据属性初始化大多数与 Bootstrap 5 一起提供的交互式组件，而无需添加任何额外的 JavaScript 代码，而对于其中的一些组件，我们必须使用自己的 JavaScript 来初始化。

## 使用数据属性初始化交互式组件

我们可以使用数据属性初始化大多数与 Bootstrap 5 一起提供的交互式组件，而无需添加任何额外的 JavaScript 代码。对于大多数组件，我们使用 `data-bs-toggle` 数据属性，其值是组件的名称，如下所示：

HTML

```js
<button type="button" class="btn btn-primary" data-bs-toggle="button">Button</button>
```

然而，对于轮播组件，我们必须使用 `data-bs-ride="carousel"`，对于滚动监听组件，我们必须使用 `data-bs-spy="scroll"`，并且，要关闭警告或偏移画布组件，我们必须使用 `data-bs-dismiss="[alert/offcanvas]"`。

其他所需的数据属性

一些组件需要各种其他数据属性才能正常工作。上述提到的数据属性只是用于初始化它们的属性。

## 使用 JavaScript 初始化交互式组件

正如我们在本章开头的列表中所看到的，我们必须使用自己的 JavaScript 代码来初始化一些交互式组件。

例如，我们想要为以下徽章组件创建一个提示组件：

HTML

```js
<div class="badge bg-secondary" title="Tooltip for badge">Badge with tooltip</div>
```

`<div>` 标签具有 `title` 属性，该属性被提示组件使用。

### 语法

我们可以通过在 JavaScript 中使用构造函数创建并初始化一个类的对象实例来初始化一个交互式组件，使用 `new` 关键字。

该语法的语法如下：

JavaScript

```js
var tooltip = new bootstrap.Tooltip(element, options)
```

在这里，`element` 可以是一个 DOM 元素或 CSS 选择器，而 `options` 是一个对象，是可选的。即使我们使用 JavaScript 初始化组件，也可以使用数据属性定义选项。我们将在 *为交互式组件定义选项* 部分了解更多关于选项的内容。

### 使用 DOM 元素

要使用 DOM 元素初始化一个交互式组件，我们必须首先将其存储在一个变量中，然后使用该变量作为 DOM 元素的引用。以下是一个示例：

part-3/chapter-11/examples/initialization/index.xhtml 行 14

```js
<div class="badge bg-secondary" title="Tooltip for badge" id="tooltipDOM">Badge with tooltip</div>
```

part-3/chapter-11/examples/initialization/js/script.js 行 1-2

```js
var tooltipDOMelement = document.getElementById('tooltipDOM');
```

```js
var tooltipDOM = new bootstrap.Tooltip(tooltipDOMelement);
```

在前面的代码中，我们将具有 `id="tooltipDOM"` 的元素存储在 `element` 变量中，然后将其作为参数传递给构造函数。我们没有传递任何选项，因为我们将在此章的稍后部分更详细地介绍这一点。

### 使用 CSS 选择器

要使用 CSS 选择器初始化一个交互式组件，我们可以简单地将它作为构造函数的参数传递。然后使用 `querySelector()` 方法找到该元素。以下是一个使用 `<div>` 元素的 ID 作为选择器的示例：

part-3/chapter-11/examples/initialization/index.xhtml 行 19

```js
<div class="badge bg-secondary" title="Tooltip for badge" id="tooltipCSS">Badge with tooltip</div>
```

part-3/chapter-11/examples/initialization/js/script.js 行 4

```js
var tooltip = new bootstrap.Tooltip('#tooltipCSS');
```

## 初始化多个组件

交互式组件的各种 JavaScript 插件仅支持单个元素。因此，要初始化多个元素，我们可以为列表或数组中的每个元素调用构造函数。现在我们将看到两个不同的示例，说明我们如何做到这一点。

### 使用 for 循环

对于第一个示例，我们将使用以下 HTML，其中包含三个具有相同 `tooltipLoop` 类的相同徽章组件：

part-3/chapter-11/examples/initialization/index.xhtml 行 24-26

```js
<div class="badge bg-secondary tooltipLoop" title="Tooltip for badge">Badge with tooltip</div>
```

```js
<div class="badge bg-secondary tooltipLoop" title="Tooltip for badge">Badge with tooltip</div>
```

```js
<div class="badge bg-secondary tooltipLoop" title="Tooltip for badge">Badge with tooltip</div>
```

当使用 `for` 循环时，我们首先将所有 `<div>` 元素存储在 `tooltipLoopElements` 变量中。然后，我们遍历这个 NodeList 并为每个元素调用构造函数：

part-3/chapter-11/examples/initialization/js/script.js 行 6-9

```js
var tooltipLoopElements = document.querySelectorAll('.tooltipLoop');
```

```js
for (var i = 0; i < tooltipLoopElements.length; i++) {
```

```js
  new bootstrap.Tooltip(tooltipLoopElements[i])
```

```js
}
```

### 使用 map 方法

对于第二个示例，我们将使用以下 HTML，其中包含三个具有相同 `tooltipMap` 类的相同徽章组件：

part-3/chapter-11/examples/initialization/index.xhtml 行 31-33

```js
<div class="badge bg-secondary tooltipMap" title="Tooltip for badge">Badge with tooltip</div>
```

```js
<div class="badge bg-secondary tooltipMap" title="Tooltip for badge">Badge with tooltip</div>
```

```js
<div class="badge bg-secondary tooltipMap" title="Tooltip for badge">Badge with tooltip</div>
```

当使用`map()`方法时，我们首先将所有`<div>`元素存储在`tooltipMapElements`变量中。然而，这次我们使用`slice()`和`call()`方法将 NodeList 转换为数组。然后，我们使用`map()`方法创建一个新的数组并将其存储在`tooltipMap`变量中。对于数组中的每个元素，我们像之前一样调用构造函数：

part-3/chapter-11/examples/initialization/js/script.js 行 11-14

```js
var tooltipMapElements = [].slice.call(document.querySelectorAll('.tooltipMap'));
```

```js
var tooltipMap = tooltipMapElements.map(function(tooltipMapElement) {
```

```js
  return new bootstrap.Tooltip(tooltipMapElement)
```

```js
});
```

下拉组件所需的数据属性

即使您选择使用 JavaScript 初始化下拉组件，触发元素的`data-bs-toggle="dropdown"`数据属性仍然是必需的。

我们现在已经学会了如何以各种方式初始化交互式组件。我们将继续学习如何为交互式组件定义选项。

# 定义交互式组件的选项

在本章开头列出的列表中，我们看到了交互式组件可用的选项。在 Bootstrap 5 的官方文档中，您将找到一个表格，描述了每个组件的所有可用选项（如果有）。在这个表格中，您将找到每个选项的名称、类型、默认值和描述。您可以通过以下链接访问官方文档中的组件：[getbootstrap.com/docs/5.1/components](http://getbootstrap.com/docs/5.1/components)。

可以通过 JavaScript 设置同一类型的所有组件的默认选项和一组组件的常见选项，而单个组件的选项可以通过数据属性或 JavaScript 传递。现在让我们看看这是如何完成的。

## 定义默认选项

您可以使用以下语法为组件的选项更改默认值：

JavaScript

```js
bootstrap.[component name].Default.[option name] = [option value];
```

例如，我们可以将我们的提示框组件设置为默认延迟为 1,000 毫秒，如下所示：

JavaScript

```js
bootstrap.Tooltip.Default.delay = 1000;
```

## 使用数据属性定义选项

要使用数据属性定义选项，我们将选项名称附加到`data-bs-`，并且选项名称的案例类型应从*camelCase*更改为*kebab-case*。

现在，让我们看看如何使用触发选项初始化提示框组件，该选项接受字符串类型的值。默认值为`‘hover focus’`，可能的值有`‘click | hover | focus | manual’`。我们将使用以下代码仅使用点击来触发提示框：

HTML

```js
<a href="#" id="tooltip" title="Tooltip for link" data-bs-trigger="click">Link</a>
```

现在提示框将仅在点击时触发以显示和隐藏——就像弹出组件一样。请注意，仅凭之前的代码本身不会初始化提示框组件。正如我们刚刚学到的，这必须通过我们自己的 JavaScript 代码来完成：

无法使用数据属性设置的选项

由于安全原因，以下由弹出框和提示组件使用的选项不能使用数据属性设置：`sanitize`、`sanitizeFn`和`allowList`。这些都与在提示或弹出框的内容中使用 HTML 有关。

## 使用 JavaScript 定义选项

如本章前面所述，初始化交互式组件时，可以通过构造函数传递一个可选的`options`对象。再次强调，语法如下：

```js
var tooltip = new bootstrap.Tooltip(element, options)
```

要使用对象定义一个选项，我们只需将选项名称作为对象的属性，然后根据选项类型给出一个有效的值。初始化与之前相同的提示组件，将`trigger`选项设置为`click`，将看起来像这样：

HTML

```js
<a href="#" id="tooltip" title="Tooltip for link">Link</a>
```

JavaScript

```js
var tooltip = new bootstrap.Tooltip('#tooltip', { trigger: 'click' })
```

现在提示框将仅在点击时触发显示和隐藏。

## 定义默认、通用和个体选项

如前所述，我们可以使用 JavaScript 和数据属性为组件定义默认、通用和个体选项。如果你想要为同一类型的所有组件定义一些默认选项，为组件组定义具有相同值的选项，以及为每个组件实例定义具有不同值的个体选项，这将非常有用。

我们现在将看到一个示例，我们将利用所有这些。在我们的 HTML 中，我们有四个徽章组件，我们想要为它们添加一个提示组件。在每个`<div>`元素上，我们使用`data-bs-placement`数据属性定义提示的位置：

part-3/chapter-11/examples/options/index.xhtml

```js
<div class="badge bg-secondary tooltipOnClick" title="Tooltip on top" data-bs-placement="top">Badge with tooltip on top</div>
```

```js
<div class="badge bg-secondary tooltipOnClick" title="Tooltip on bottom" data-bs-placement="bottom">Badge with tooltip on bottom</div>
```

```js
<div class="badge bg-secondary tooltipOnClick" title="Tooltip on left" data-bs-placement="left">Badge with tooltip on left</div>
```

```js
<div class="badge bg-secondary tooltipOnClick" title="Tooltip on right" data-bs-placement="right">Badge with tooltip on right</div>
```

然后，在我们的 JavaScript 中，我们首先定义`delay`的默认选项并将其设置为`1000`毫秒。之后，我们使用`trigger`选项设置为`‘click’`来初始化组件：

part-3/chapter-11/examples/options/js/script.js

```js
bootstrap.Tooltip.Default.delay = 1000;
```

```js
var tooltipElementList = document.querySelectorAll('.tooltipOnClick');
```

```js
for (var i = 0; i < tooltipElementList.length; i++) {
```

```js
    new bootstrap.Tooltip(tooltipElementList[i], { trigger: 
```

```js
      'click' })
```

```js
}
```

现在四个徽章组件将各自拥有一个提示组件，点击时延迟 1,000 毫秒触发。提示框将分别放置在顶部、底部、左侧和右侧。

使用数据属性和 JavaScript 设置相同选项

如果你同时使用数据属性和 JavaScript 指定相同的选项，JavaScript 设置将优先。

# 使用 JavaScript 方法

在本章开头列表中，我们看到了所有交互式组件都有方法。在 Bootstrap 5 的官方文档中，你可以找到一个表格，描述了每个组件的所有可用方法。在这个表格中，你可以找到每个方法的名称和描述。你可以通过以下链接访问官方文档中的组件：[getbootstrap.com/docs/5.2/components](http://getbootstrap.com/docs/5.2/components)。

现在，我们将看到一些如何使用特定方法的示例。在我们的示例页面上，我们首先有一个默认的提示组件用于比较。其 HTML 代码如下：

part-3/chapter-11/examples/methods/index.xhtml 行 15

```js
<div class="badge bg-secondary" id="tooltipDefault" title="Default tooltip">Badge with default tooltip</div>
```

提示具有`id="tooltipDefault"`属性，我们使用以下 JavaScript 对其进行初始化：

part-3/chapter-11/examples/methods/js/script.js 行 1

```js
var tooltipDefault = new bootstrap.Tooltip('#tooltipDefault', { placement: "bottom" });
```

如我们所见，提示使用`placement`选项放置在底部。这只是为了使示例页面在视觉上更好。

对于示例页面上的下一个提示组件，我们希望它在页面加载时显示，同时禁用悬停和聚焦触发。HTML 代码如下：

part-3/chapter-11/examples/methods/index.xhtml 行 21

```js
<div class="badge bg-secondary" id="tooltipOnLoad" title="Tooltip triggered on load">Badge with tooltip triggered on load</div>
```

提示具有`id="tooltipOnLoad"`属性，我们将使用它进行初始化。JavaScript 代码如下：

part-3/chapter-11/examples/methods/js/script.js 行 3-4

```js
var tooltipOnLoad = new bootstrap.Tooltip('#tooltipOnLoad', { trigger: "manual", placement: "bottom" });
```

```js
tooltipOnLoad.show();
```

提示使用`placement`选项放置在底部。这只是为了使示例页面在视觉上更好。

对于示例页面上的最终提示组件，我们希望当我们点击页面上的另一个按钮时切换其可见性。这个 HTML 代码如下：

part-3/chapter-11/examples/methods/index.xhtml 行 27-28

```js
<button type="button" id="triggerForTooltip" class="btn btn-secondary mb-4">Trigger for tooltip</button><br>
```

```js
<div class="badge bg-secondary" id="tooltipOnButtonClick" title="Tooltip toggled on button click">Badge with tooltip toggled on button click</div>
```

按钮具有`id="triggerForTooltip"`属性，我们将使用它来附加事件监听器，提示具有`id="tooltipOnButtonClick"`属性，我们将使用它进行初始化。JavaScript 代码如下：

part-3/chapter-11/examples/methods/js/script.js 行 6-10

```js
var tooltipOnButtonClick = new bootstrap.Tooltip('#tooltipOnButtonClick', { trigger: "manual", placement: "bottom" });
```

```js
var triggerForTooltip = document.getElementById('triggerForTooltip');
```

```js
triggerForTooltip.addEventListener('click', function() {
```

```js
  tooltipOnButtonClick.toggle();
```

```js
})
```

第一行与上一个示例类似：使用`placement`选项将提示放置在底部，并将触发提示的方法设置为`manual`。下面，我们首先获取按钮元素，然后将其附加一个`click`事件，并在按钮被点击时执行`toggle()`方法。这将显示和隐藏提示。

## 异步方法和转换

重要的是要知道所有方法都是异步的，并且开始一个转换过程。一旦转换开始，它们就会返回调用者。这意味着在组件正在转换时对其进行的调用将被忽略。以下是一个示例，它将显示模态框，但不会隐藏它，因为`hide()`方法是在`show()`方法之后立即调用的，因此是在模态框显示转换完成之前：

JavaScript

```js
var modalElement = document.getElementById('modal');
```

```js
var modal = bootstrap.Modal.getOrCreateInstance(modalElement);
```

```js
modal.show();
```

```js
modal.hide();
```

相反，如果我们想在转换完成后执行一些代码，我们需要监听相应的事件，如下所示：

JavaScript

```js
var modalElement = document.getElementById('modal');
```

```js
var modal = bootstrap.Modal.getOrCreateInstance(modalElement);
```

```js
modal.show();
```

```js
modalElement.addEventListener('shown.bs.modal', function (event) {
```

```js
  modal.hide();
```

```js
})
```

现在，模态框将首先显示，然后在模态框的`shown`事件被触发时再次隐藏。在下一节中，我们将看到另一个如何使用 Bootstrap 5 事件的示例。

# 使用 JavaScript 事件

在本章开头的列表中，我们看到了大多数交互式组件都有事件。在 Bootstrap 5 的官方文档中，你可以找到一个表格，描述了每个组件的所有可用事件。在这个表格中，你可以找到每个方法的类型和描述。你可以通过此链接访问官方文档中的组件：[getbootstrap.com/docs/5.2/components](http://getbootstrap.com/docs/5.2/components)。

现在，我们将看到一个如何使用特定事件的示例，用于提示框组件。在我们的示例页面上，我们有一个按钮，它触发一个模态组件。在模态框的页脚中，我们有一个**确认**按钮，我们希望在模态框显示且过渡完成时显示提示框。

这里是 HTML 的样式：

part-3/chapter-11/examples/events/index.xhtml

```js
<button type="button" class="btn btn-primary" data-bs-toggle="modal" data-bs-target="#modal">Open modal</button>
```

```js
<div class="modal fade" id="modal" tabindex="-1" aria-labelledby="modalTitle" aria-hidden="true">
```

```js
  <div class="modal-dialog">
```

```js
    <div class="modal-content">
```

```js
      <div class="modal-header">
```

```js
        <h5 class="modal-title" id="modalTitle">Modal 
```

```js
          title</h5>
```

```js
        <button type="button" class="btn-close" data-bs-
```

```js
          dismiss="modal" aria-label="Close"></button>
```

```js
      </div>
```

```js
      <div class="modal-body">
```

```js
        <p>Lorem ipsum dolor sit amet, consectetur 
```

```js
           adipiscing elit. In laoreet pellentesque lorem 
```

```js
           sed elementum. Suspendisse maximus convallis ex. 
```

```js
           Etiam eleifend velit leo.</p>
```

```js
      </div>
```

```js
      <div class="modal-footer">
```

```js
        <button type="button" class="btn btn-primary" 
```

```js
          id="tooltip" title="Click here to 
```

```js
          confirm">Confirm</button>
```

```js
        <button type="button" class="btn btn-secondary" 
```

```js
          data-bs-dismiss="modal">Close</button>
```

```js
      </div>
```

```js
    </div>
```

```js
  </div>
```

```js
</div>
```

提示框具有 `id="tooltip"`，我们将用它进行初始化，并且提示框的内容放置在 `title` 属性中。在 JavaScript 代码中，我们首先将模态框的 DOM 元素存储在一个变量中，并初始化提示框组件。然后，在附加到模态框的事件监听器中，我们监听 `shown.bs.modal` 事件，并在该事件触发时调用提示框组件的 `show()` 方法。此代码如下：

part-3/chapter-11/examples/events/js/script.js

```js
var modal = document.getElementById('modal');
```

```js
var tooltip = new bootstrap.Tooltip('#tooltip');
```

```js
modal.addEventListener('shown.bs.modal', function (event) {
```

```js
  tooltip.show();
```

```js
})
```

此示例将显示提示框，正好在模态框的过渡完成时。

带有属性的事件

旋转木马组件的两个事件具有以下附加属性：`direction` (`"left"` 或 `"right"`), `relatedTarget` (DOM 元素), `from` (索引号), 和 `to` (索引号)。

# 摘要

在本章中，我们学习了可以与 Bootstrap 5 的交互式组件一起使用的某些高级 JavaScript 特性。首先，我们快速概述了交互式组件，并看到了它们在依赖性和初始化方面的共同要求。然后，我们看到了如何使用数据属性或 JavaScript 初始化交互式组件。接着，我们看到了如何使用数据属性或 JavaScript 定义选项，最后，我们更详细地探讨了如何使用 JavaScript 进行方法和事件的处理。

在本书的下一章和最后一章中，我们将学习如何优化编译后的 Bootstrap 5 CSS 和 JavaScript 代码。
