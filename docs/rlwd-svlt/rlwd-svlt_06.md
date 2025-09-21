

# 第六章：使用操作整合库

互联网上有许多 JavaScript UI 库。然而，在撰写本书时，Svelte 相对较新。并不是所有的 UI 库都是用 Svelte 编写的，也不是专门为 Svelte 编写的。但这并不意味着我们不能在我们的 Svelte 组件中使用它们。

有许多方法可以将第三方 JavaScript UI 库集成到 Svelte 中。在本章中，我们将探讨如何使用 Svelte 操作来实现这一点。

我们将首先整合一个假设的 UI 库，逐步构建我们的案例，说明为什么 Svelte 操作适合这项工作。在这个过程中，我将解释如何使用 Svelte 操作处理不同的场景，并展示 Svelte 操作在哪些方面不足。我将讨论我选择 Svelte 操作和选择其他选项的理由和个人的观点。

在此之后，我将向你展示一些真实的 UI 库示例。然后，我们将探索如何使用更多示例将用其他框架（如 React 和 Vue）编写的 UI 库集成到 Svelte 中。

到本章结束时，你将看到你不仅限于在你的 Svelte 应用程序中只使用用 Svelte 编写的 UI 库——你可以重用互联网上可用的任何 UI 库。

本章涵盖了以下主题：

+   将 JavaScript UI 库集成到 Svelte 中

+   为什么我们应该使用操作来整合 UI 库和其他替代方案

+   将用其他框架编写的 UI 库集成到 Svelte 中

# 技术要求

你可以在这里找到本章的示例和代码：[`github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter06`](https://github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter06)。

# 将纯 JavaScript UI 库集成到 Svelte 中

首先，我们将探索用纯 JavaScript 编写的 UI 库。当我们使用“纯 JavaScript”这个短语时，我们指的是普通的 JavaScript，或者在没有框架或库的情况下使用的 JavaScript。

有许多原因使得 UI 库是用纯 JavaScript 编写的：

+   性能原因——没有 Web 框架的抽象，优化会更容易

+   库作者的个性化偏好，希望库不依赖于任何框架

+   该库是在任何现代 Web 框架之前创建的

对于我们来说，纯 JavaScript UI 库很棒，因为它们不依赖于任何特定的框架运行时，这给 UI 库本身带来了额外的开销。

例如，如果我们使用一个用 React 实现的日历组件库，那么除了安装日历组件库之外，我们还需要安装 React 的框架。

这个额外的依赖导致包的大小增加，并且可能与 Svelte 发生冲突。因此，当在 Svelte 中使用组件库时，通常更倾向于选择不依赖于任何特定框架的库。

既然我们已经了解了为什么纯 JavaScript UI 库很棒，让我们讨论如何将它们集成到 Svelte 中。在本章中，我们将探索使用 Svelte 动作在 Svelte 中集成库，这引发了一个问题，为什么我们选择使用 Svelte 动作来集成 UI 库？

## 为什么使用 Svelte 动作来集成 UI 库？

在上一章中，我们探讨了 Svelte 动作在添加自定义事件处理程序方面的有用性。同时，Svelte 动作作为元素级别的生命周期函数，这使得它们在与第三方库交互时非常有用。现在，让我们探讨为什么这是这种情况。

让我们以一个日历组件库为例。为了简化起见，并且避免被实现细节所困扰，让我们想象这个库是一个虚构的库，而不是使用任何真实的日历组件库。这使我们能够专注于一般问题本身，而不是特定库的实现细节。

我们将在之后查看一些真实的 UI 库。

为了决定日历组件将被添加到 DOM 中的位置，组件库通常要求我们指定一个容器元素来容纳库组件。

例如，在这里，`ImaginaryCalendar`要求我们将容器元素作为构造函数参数的一部分传递：

```js
new ImaginaryCalendar({ container: containerElement })
```

要获取 Svelte 中元素的引用，我们可以使用`bind:this`：

```js
<script>
  let containerElement;
</script>
<div bind:this={containerElement} />
```

`containerElement`变量仅在元素挂载后更新为元素的引用，因此它只能在`onMount`中引用：

```js
<script>
  import { onMount } from 'svelte';
  let containerElement;
  let calendar;
  onMount(() => {
    calendar = new ImaginaryCalendar({ container: containerElement });
    return () => calendar.cleanup();
  });
</script>
```

注意，我们保留了对`calendar`实例的引用，因为我们可以使用它来调用日历方法以获取或设置值：

```js
calendar.setDate(date);
```

此外，当组件卸载时，我们也调用`calendar.cleanup()`进行清理。

注意

日历库只是一个虚构的例子。然而，大多数 UI 库都会提供类似的 API 或方法来检索或修改组件实例的内部状态，并在不再使用时进行清理。

在这里使用`calendar`实例时，我们需要格外小心。我们希望避免在初始化之前引用`calendar`实例，以防止在`calendar`实例仅在`onMount`之后声明和初始化时遇到引用错误：

为了安全起见，我们应该在调用`calendar`实例的任何方法之前检查它是否已定义。在下面的代码示例中，我们在调用`calendar.setDate()`方法之前验证日历实例是否已定义。

```js
if (calendar) calendar.setDate(date).
```

当日历条件性地创建时，这种额外谨慎的需求更为明显：

```js
<script>
  import { onMount } from 'svelte';
  let containerElement;
  let calendar;
  onMount(() => {
    if (containerElement) {
      calendar = new ImaginaryCalendar({ container: containerElement });
      return () => calendar.cleanup();
    }
  });
</script>
{#if someCondition}
  <div bind:this={containerElement} />
{/if}
```

在前面的代码中，你可以看到 `<div>` 是基于 `someCondition` 条件有条件地创建的。这就是为什么在 `onMount` 中，我们需要在用 `containerElement` 作为容器创建 `ImaginaryCalendar` 之前检查 `containerElement` 是否可用。并且，只有当 `ImaginaryCalendar` 被创建时，`calendar` 实例才可用，因此只有在 `someCondition` 为 `true` 时才可用。

前面的代码说明了 `calendar` 实例可能未定义的许多可能性之一。

关于这段代码，有一点需要注意，它的行为是不正确的，因为它没有尝试在 `someCondition` 的值从 `false` 变为 `true` 时创建 `ImaginaryCalendar`，也没有在它变回 `false` 时进行清理。

这正是 Svelte 操作大放异彩的地方。

## 使用 Svelte 操作

通过修改前面的代码，使其使用操作，你会发现我们不需要额外的检查来确保在实例化 `ImaginaryCalendar` 之前 `containerElement` 是可用的。

以下代码展示了如何实现这样的操作。在这里，我们的 Svelte 操作名为 `calendar`：

```js
<script>
  function calendar(containerElement) {
    const calendar = new ImaginaryCalendar({ container: containerElement });
    return {
      destroy() {
        calendar.cleanup();
      }
    };
  }
</script>
{#if someCondition}
  <div use:calendar />
{/if}
```

这是因为，当使用 Svelte 操作时，操作函数只有在元素被创建并挂载到 DOM 上时才会被调用，并且只传递元素的引用。

当条件改变到 `<div>` 元素从 DOM 中移除时，操作的 `destroy` 方法将被调用以清理资源。

使用 Svelte 操作，我们现在可以在一个组件内创建任意多的 `ImaginaryCalendar` 实例，只需将操作添加到不同的 HTML 元素中：

为了证明我的观点，在下面的代码片段中，除了在之前的例子中看到的原始 `<div>` 元素外，我还添加了另一个 `<div>` 元素和另外三个 `<div>` 元素，使用 `{#each}` 块来实现。然后，我将日历操作应用于所有四个 `<div>` 元素，以创建四个更多的日历，并且一次创建多个日历时我们没有遇到任何错误。

```js
{#if someCondition}
  <div use:calendar />
{/if}
<!-- Look we can have as many calendars as we want -->
<div use:calendar />
{#each [1, 2, 3] as item}
  <div use:calendar />
{/each}
```

如果我们要使用 `bind:this` 和 `onMount`，我们就必须多次重复自己，通过多次声明多个 `containerElement` 变量，并使用每个 `containerElement` 变量实例化 `ImaginaryCalendar`。

现在，随着 `calendar` 实例被封装在操作中，我们应该如何从外部调用 `calendar` 实例方法来更新 `calendar` 状态？

那就是操作数据的作用！

## 向 Svelte 操作添加数据

在前面的章节中，我们创建了一个 `calendar` 操作并在其中实例化了 `ImaginaryCalendar` 实例。如果我们想在 `calendar` 操作外部调用 `ImaginaryCalendar` 实例的方法，例如调用 `calendar.setDate(date)` 来设置日历的日期，我们应该怎么做？

由于 `calendar` 实例是在 `calendar` 动作中定义的，因此无法在 `calendar` 动作之外调用 `calendar.setDate(date)`。一种解决方案是通过动作数据传递 `date` – 也就是说，我们可以将 `date` 作为动作数据提供，并使用传入的日期调用 `calendar.setDate(date)`。

例如，在下面的代码片段中，我们将 `date` 传递给 `calendar` 动作：

```js
<div use:calendar={date} />
```

在 `calendar` 动作中，我们使用传入的日期调用 `calendar.setDate(date)`。除此之外，我们在动作中定义了一个 `update` 方法，以便每当传递给 `calendar` 动作的日期更改时，Svelte 将调用 `calendar.setDate(date)`：

```js
<script>
  function calendar(containerElement, date) {
    const calendar = new ImaginaryCalendar({ container: containerElement });
    calendar.setDate(date);
    return {
      update(newDate) {
        calendar.setDate(newDate);
      },
      destroy() {
        calendar.cleanup();
      }
    };
  }
</script>
<div use:calendar={new Date(2022, 10, 5)} />
```

在这里，我们可以向不同的 `calendar` 实例传递不同的日期：

```js
{#each dates as date}
  <div use:calendar={date} />
{/each}
```

那太好了！

现在，如果你想在模式更改时调用不同的 `calendar` 实例方法，比如 `calendar.setMode()`，会怎么样？

你可以将 `date` 和 `mode` 都传递给动作：

```js
<div use:calendar={{ date, mode }} />
```

在这种情况下，`calendar` 动作需要同时处理 `date` 和 `mode`：

```js
function calendar(node, { date, mode }) {
  const calendar = new ImaginaryCalendar({ container: containerElement });
  calendar.setDate(date);
  calendar.setMode(mode);
  return {
    update({ date: newDate, mode: newMode }) {
      calendar.setDate(newDate);
      calendar.setMode(newMode);
    },
    destroy() { ... }
  };
}
```

当 `date` 或 `mode` 中的任何一个更改时，`calendar` 动作的 `update` 方法将被调用。这意味着在先前的代码中，每当 `date` 或 `mode` 发生更改时，我们都会调用 `calendar.setDate()` 和 `calendar.setMode()`。这可能没有明显的后果，但我们可能正在进行不必要的操作。

解决这个问题的方法是跟踪并始终在 `update` 方法中检查 `date` 或 `mode` 是否已更改。这就是我们如何做到这一点：

```js
function calendar(node, { date, mode }) {
  const calendar = new ImaginaryCalendar({ container: containerElement });
  calendar.setDate(date);
  calendar.setMode(mode);
  return {
    update({ date: newDate, mode: newMode }) {
      if (date !== newDate) {
        calendar.setDate(newDate);
        date = newDate;
      }
      if (mode !== newMode) {
        calendar.setMode(newMode);
        mode = newMode;
      }
    },
    destroy() { ... }
  };
}
```

在先前的代码中，我们正在检查 `newDate` 是否与当前的 `date` 不同，如果是的话，我们就调用 `calendar.setDate()` 方法并更新我们的当前 `date` 引用。我们对 `mode` 做了类似的事情。

这有效。然而，正如你所看到的，这比我们最初创建 `calendar` 动作时设置的代码更多，也更复杂。

那么如果你想要调用一个与任何数据无关的 `calendar` 实例方法，比如 `calendar.refreshDates()`，会怎么样？

这就是使用动作的不足之处。

## Svelte 动作的一个替代方案

记得之前的例子，我们使用了 `bind:this` 和 `onMount` 来初始化 `ImaginaryCalendar` 吗？

我们说过，这种方法不够灵活，如果需要执行以下操作，就会不足：

+   条件渲染容器并创建 `ImaginaryCalendar`

+   在同一组件内拥有多个日历

这些缺点都是真实的，但有一个用例，使用 `bind:this` 和 `onMount` 初始化 `ImaginaryCalendar` 是完全可行的。这是当之前提到的条件从未为真时：

+   我们不需要条件渲染容器

+   我们不需要在同一个组件内拥有多个日历实例（这并不完全正确，但我们会回到这一点）

我不确定你现在是否在想同样的事情，但让我来打破这种悬念。

这时我们想要将 `ImaginaryCalendar` 作为 Svelte 组件使用。

在 `ImaginaryCalendar` Svelte 组件内部，我们只有一个容器元素，它始终可用：

```js
<!-- ImaginaryCalendarComponent.svelte -->
<script>
  import { onMount } from 'svelte';
  let containerElement;
  let calendar;
  onMount(() => { ... });
</script>
<div bind:this={containerElement} />
```

然后，你可以有条件地使用这个组件：

```js
<script>
  import ImaginaryCalendarComponent from './ ImaginaryCalendarComponent.svelte';
</script>
{#if someCondition}
  <ImaginaryCalendarComponent />
{/if}
```

或者，你可以根据需要多次使用它：

```js
{#if someCondition}
  <ImaginaryCalendarComponent />
{/if}
<!-- Look we can have as many calendars as we want -->
<ImaginaryCalendarComponent />
{#each array as item}
  <ImaginaryCalendarComponent />
{/each}
```

在这里，我们使用动作替换了元素，`<div use:calendar />`，用 Svelte 组件，`<``ImaginaryCalendarComponent />`。

这很正常。

在上一章中，我们考虑了通过组件或动作进行逻辑抽象。

在这个场景中，我们正在考虑将使用元素作为容器实例化 UI 库的逻辑进行抽象，我们可以将其抽象为一个 Svelte 动作或一个 Svelte 组件。

这两个选项都同样合适。

这两个选项都是为了这个目的而设计的。

那么，你应该选择哪些选项呢？让我们来了解一下。

## 在 Svelte 动作和 Svelte 组件之间进行选择

当面临选择这两个选项之一时，我的个人偏好如下。

当你在寻找提供以下方面的选项时，选择 Svelte 动作与 UI 库集成： 

+   更轻量级。与 Svelte 动作相比，Svelte 组件有稍微多一点的开销。

+   只传递零到一份数据到 UI 库组件实例。如果你要将两份或更多数据传递到动作中，那么每当数据中的任何部分发生变化时，动作的更新方法将被调用。

如果你正在寻找提供以下功能的选项，你应该选择 Svelte 组件：

+   允许更多的优化空间和更精细的控制。

+   允许你直接调用 UI 库组件实例方法。

+   允许你将子内容传递到 UI 组件库中。

我们并没有过多讨论这一点，但将 UI 库作为组件集成可以打开通过插槽向 UI 组件库传递额外内容的可能性：

```js
<ImaginaryCalendarComponent>
  <!—Customize how each cell of the calendar looks ––>
  <svelte:fragment sl"t="c"ll" let:date>
    {date}
  </svelte:fragment>
</ImaginaryCalendarComponent>
```

如果你想要了解更多关于插槽和如何在 Svelte 中组合组件的信息，请阅读*第四章*，在那里我们广泛地探讨了这一主题。现在我们已经介绍了如何使用 Svelte 动作集成 UI 库，为什么我们应该使用 Svelte 动作，以及替代方案和考虑因素，让我们来看一个真实世界的例子，Tippy.js。

# 示例 - 集成 Tippy.js

Tippy.js 是一个提示框、弹出框、下拉菜单和菜单库。

我与 Tippy.js 库没有任何关联，我选择 Tippy.js 作为示例纯粹是偶然的。不过，Tippy.js 有一个简洁的 API，使其成为示例的好候选。

首先，让我们看看 Tippy.js 的文档：[`atomiks.github.io/tippyjs/`](https://atomiks.github.io/tippyjs/)。

在使用我们选择的包管理器安装了 `tippy.js` 库之后，我们就可以将 Tippy.js 导入到我们的代码中：

```js
import tippy from 'tippy.js';
import 'tippy.js/dist/tippy.css';
```

现在，我们可以使用以下构造函数初始化 `tippy`：

```js
tippy('#id');
tippy(document.getElementById('my-element'));
```

在这里，我们传递了 Tippy.js 应该提供提示的元素。

你可以通过元素的数据属性指定提示内容的任何自定义设置，Tippy.js 将在初始化时拾取这些设置：

```js
<button data-tippy-content="hello" />
```

或者，你可以在构造函数中传递这个方法：

```js
tippy(element, { content: 'hello' });
```

要在初始化后更新内容，请调用 Tippy.js 的`setContent`方法：

```js
tooltipInstance.setContent("bye");
```

要永久销毁并清理提示实例，Tippy.js 提供了`destroy`方法：

```js
tooltipInstance.destroy();
```

这里，我们已经有了一切创建`tippy`动作所需的东西。我们有以下方法：

+   创建`tippy`提示——`tippy(…)`

+   清理`tippy`提示——`tooltipInstance.destroy()`

+   更新`tippy`提示——`tooltipInstance.setContent(…)`

让我们看看`tippy`动作应该是什么样子。

这是我希望它看起来像的样子：

```js
<div use:tippy={tooltipContent} />
```

在前面的代码片段中，我们将我们的`tippy`动作应用于一个`<div>`元素。由 Tippy.js 创建的提示内容应该作为动作数据传递给`tippy`动作，表示为`tooltipContent`变量。每当`tooltipContent`更改时，动作应该对其做出反应并更新提示。

因此，让我们编写我们的`tippy` Svelte 动作。以下是动作的脚手架：

```js
function tippy(element, content) {
  // TODO #1: initialize the library
  return {
    update(newContent) {
      // TODO #2: do something when action data changes
    },
    destroy() {
      // TODO #3: clean up
    }
  };
}
```

如您所见，我们基于 Svelte 动作合约创建了`tippy`动作：一个返回包含`destroy`和`update`方法的对象的函数。

我在代码中留下了三个`TODO`，每个都标记了 Svelte 动作的不同阶段。让我们逐一过一遍并填补它们。第一个`TODO`是动作在元素创建并挂载到 DOM 后被调用的地方。在这里，我们得到了动作应用到的元素和动作数据，我们应该使用它来初始化 Tippy.js 提示：

```js
// TODO #1: initialize the library
const tooltipInstance = tippy(element, { content });
```

第二个`TODO`位于`update`方法中。这个方法将在动作数据每次更改时被调用。在这里，我们需要调用 Tippy.js 提示实例以反映 Svelte 组件中的数据集：

```js
// TODO #2: do something when action data changes
tooltipInstance.setContent(newContent);
```

第三个`TODO`位于`destroy`方法中。这个方法将在元素从 DOM 中移除后被调用。在这里，我们需要清理我们在动作中创建的 Tippy.js 提示实例：

```js
// TODO #3: clean up
tooltipInstance.destroy();
```

就这样——我们现在有一个工作的`tippy`动作，它集成了 Tippy.js 提示，并且每当我们将鼠标悬停在元素上时，都会显示一个具有自定义内容的提示。

你可以在 GitHub 上找到完整的代码：[`github.com/PacktPublishing/Practical-Svelte/tree/main/Chapter06/01-tippy`](https://github.com/PacktPublishing/Practical-Svelte/tree/main/Chapter06/01-tippy).

让我们再看一个例子，通过这个例子我想向你展示在集成 UI 库时可以使用动作做的一些其他事情。

我们接下来要查看的 UI 库是 CodeMirror。

# 示例——集成 CodeMirror

CodeMirror 是一个具有许多出色编辑功能的代码编辑组件，例如语法高亮、代码折叠等。

你可以在[`codemirror.net/`](https://codemirror.net/)找到 CodeMirror 的文档。

在撰写本文时，CodeMirror 目前处于版本 5.65.9。

在使用我们选择的包管理器安装了`codemirror`库之后，我们可以在代码中导入`codemirror`：

```js
import CodeMirror from 'codemirror';
import 'codemirror/lib/codemirror.css';
```

现在，我们可以使用以下构造函数初始化 CodeMirror：

```js
const myCodeMirror = CodeMirror(document.body);
```

在这里，我们传递了我们要放置 CodeMirror 代码编辑器的元素。

在我继续之前，请注意，我们现在从 CodeMirror 中寻找的是同一组东西：

+   初始化 CodeMirror 的方法

+   清理 CodeMirror 实例所需的任何方法

+   任何更新 CodeMirror 实例的方法

我将把这个清单的完成和解决方法留给你。

然而，请允许我特别提醒您注意 CodeMirror 实例中的一个特定 API：

```js
myCodeMirror.on('change', () => { ... });
```

CodeMirror 的`on`方法允许 CodeMirror 实例监听事件并对它们做出反应。

所以，如果我们想从动作外部向 CodeMirror 实例添加事件监听器，我们应该怎么做？

在上一章中，我们看到了我们可以在元素上使用操作来创建自定义事件。

这意味着我们可以允许用户监听使用`codemirror`动作的元素的`'change'`事件：

```js
<div use:codemirror on:change={onChangeHandler} />
```

要实现这一点，你可以在动作内部触发一个事件：

```js
function codemirror(element) {
  const editor = CodeMirror(element);
  editor.on('change', () => {
    // trigger 'change' event on the element
    // whenever the editor changes
    element.dispatchEvent(new CustomEvent('change'));
  });
}
```

请记住，在`destroy`方法中检查是否需要清理或取消监听任何事件，以免引起任何不期望的行为。

就这样！

其余的操作留给你作为练习。

你可以在 GitHub 上找到完整的代码：[`github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter06/02-codemirror`](https://github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter06/02-codemirror)。

在本节中，我们学习了如何将纯 UI 库集成到 Svelte 中。然而，并非所有 UI 库都是独立实现的，没有任何框架。有时，你可能需要的库可能是在不同的框架中实现的，比如 React 或 Vue。在这种情况下，如何将它们集成到 Svelte 应用程序中？这就是我们接下来要探讨的。

# 使用其他框架编写的 UI 库

在 Svelte 中使用其他框架的组件并非不可能。

然而，这样做将会引入框架的运行时和其他与框架相关的开销。运行时通常包括处理响应性和标准化浏览器 API 和事件的代码。每个框架通常为其逻辑提供自己的代码，并且不会与其他框架共享。React 版本 18.2.0 的运行时在压缩后重量为 6.4 kB，这是当你想在 Svelte 中使用 React 组件时需要包含的额外代码。

因此，除非必要，否则不建议这样做。

这一节被包含在这本书中更多的是出于教育目的，以及为了展示这是可能的，以及为了实现它需要做什么。

## 在各种框架中创建组件

每个框架通常提供一个 API，它接受一个容器元素和框架组件作为应用程序的根。

在本节中，我们将探讨 React 和 Vue，这两个在撰写本文时最受欢迎的 JavaScript 框架。

例如，在 React 18 中，我们使用 `createRoot` API：

```js
import ReactDOM from 'react-dom';
const root = ReactDOM.createRoot(container);
root.render(<h1>Hello, world</h1>);
```

前面的代码使用了 JSX 语法，这不是标准 JavaScript 语言的语法。它是 `jsx` 的语法糖：

```js
import { jsx } from 'react/jsx-runtime';
const root = ReactDOM.createRoot(container)
root.render(jsx('h1', { children: 'Hello, world' }));
```

如果你没有在代码中配置任何将 JSX 语法转换为有效 JavaScript 的转换过程，你将不得不编写前面的代码。

另一方面，在 Vue 3 中，有 `createApp` API：

```js
import { createApp } from 'vue'
import App from './App.vue'
const app = createApp(App);
app.mount(container);
```

Vue 框架在文档中使用“application”这个词，提到 `createApp` 方法用于创建一个新的应用程序实例。这个词“application”用得恰到好处，因为将其他框架编写的组件库集成到我们的 Svelte 应用程序中，就像是在我们的 Svelte 应用程序中启动一个新的子应用程序一样。

你可能也已经注意到了这些框架的 API 与我们之前看到的其他 UI 库之间的相似性——它们都接受一个容器元素，以便知道在哪里渲染内容或应用更改。

与使用动作集成 UI 库类似，在弄清楚我们可以使用哪些 API 在容器元素内渲染组件之后，接下来我们必须检查是否有任何 API 在不再需要时进行清理。

## 在各种框架中清理组件

根据组件库的底层框架，提供了不同的 API 来清理不再需要的组件。

在 React 中，有一个名为 `unmount` 的方法来做这件事：

```js
root.unmount();
```

在 Vue 中，这也被称为 `unmount`：

```js
app.unmount();
```

我们接下来需要检查的是是否有任何 API 可以将数据传递到我们的组件和 API 中，以便它们可以在以后进行更新。

## 在各种框架中更新组件

与不同框架提供不同的清理 API 类似，框架提供了不同的 API 来用新数据更新组件。

如果你熟悉 React，你可以通过 props 将数据传递给 React 组件：

```js
<Component prop_name={value} />
```

这类似于 Svelte 组件中的 props。

前面的代码简化为以下内容：

```js
jsx(Component, { prop_name: value });
```

要更新组件的 props，React 允许我们再次调用 `root.render`，使用相同的组件但不同的 prop 值：

```js
root.render(jsx(App, { prop_name: 123 }));
// some time later
root.render(jsx(App, { prop_name: 456 }));
```

React 将内部进行协调并找出如何更新 DOM。

另一方面，在 Vue 中，你可以在 `createApp` API 中通过 props 传递数据：

```js
const app = createApp(Component, { prop_name: value });
```

然而，据我所知，没有直接更新 props 值的简单方法。

然而，你可以使用 Vue 的组合式 API，例如 `ref()`，来创建一个可响应和可变的 ref。有了这个，你可以修改 ref 而不是直接更新 props：

```js
const value = ref(123);
const app = createApp(Component, { prop_name: ref });
// some time later
value.value = 456;
```

如果你不太熟悉 React 和 Vue 的工作原理，那也无所谓。这本书是针对 Svelte 的，而不是针对 React 或 Vue。

从这个例子中我们学到最重要的一点是，在集成 UI 库时，无论是使用纯 JavaScript、React 还是 Vue，我们都在寻找三件事：

+   一种创建带有容器元素组件的方法

+   一种清理组件实例的方法

+   一种传递数据和更新数据的方法

如果你熟悉某个框架，你将能够找到一种方法来完成所有这些事情。

现在我们已经解决了这个问题，让我们看看一个现实世界的例子，我们将集成一个 React 日历库，`react-calendar`。

# 将 react-calendar 集成到 Svelte 中

`react-calendar`库是一个用 React 编写的日历组件库。

你可以在这里了解更多信息：[`projects.wojtekmaj.pl/react-calendar/`](https://projects.wojtekmaj.pl/react-calendar/).

`react-calendar`库接受各种属性以进行定制。但为了演示目的，我们只关注两个属性，`value`和`onChange`，这两个属性允许我们控制库选择的日期。

我们通过一个名为`value`的属性传递选定的日期。另一方面，`onChange`属性用于传递一个事件处理器，当值在日历组件内部发生变化时将被调用。我们在上一节讨论 CodeMirror 时看到了如何处理事件处理器。

因此，这就是我认为使用`calendar`动作的样子：

```js
<div
  use:calendar={selectedDate}
  on:change={(event) => selectedDate = event.detail}
/>
```

这里，`event.detail`是附加到自定义`'change'`事件的 数据，这将是从`react-calendar`组件通过`onChange`属性发送的日期值。

现在我们知道了我们的`calendar`动作会是什么样子，让我们把它写出来。

再次，这是动作的框架：

```js
function calendar(element, date) {
  // TODO #1: render the react-calendar into the element
  // TODO #2: the onChange handler to dispatch a new custom event
  return {
    update(newDate) {
      // TODO #3: re-render the calendar again when there's a new date value
    },
    destroy() {
      // TODO #4: clean up
    }
  };
}
```

在这里，我创建了一个 Svelte 动作的基本代码结构，并在代码中留下了一些 TODOs。前两个 TODOs 是设置动作应用的元素上的`calendar`实例。第三个`TODO`是处理当新日期传递到动作中时的情况，最后一个`TODO`是当元素从 DOM 中移除时进行清理。

因此，让我们填写 TODOs。

对于第一个`TODO`，让我们创建一个 React 根实例并渲染我们的`react-calendar`组件：

```js
import { jsx } from 'react/jsx-runtime';
import ReactDOM from 'react-dom';
function calendar(element, date) {
  # TODO #1: render the react-calendar into the element
  const app = ReactDOM.createRoot(element);
  app.render(jsx(Calendar, { value: date, onChange }));
  // ...
}
```

这里，我们传递了`onChange`，但我们还没有定义它。

让我们来做这件事：

```js
# TODO #2: the onChange handler to dispatch a new custom event
function onChange(value) {
  element.dispatchEvent(
  new CustomEvent('change', { detail: value })
  );
}
```

在前面的代码片段中，每当调用`onChange`时，我们将分派一个新的自定义事件，并将`value`作为自定义事件的详细信息传递。

第三个`TODO`是`update`方法的内容。每当从动作传递新的日期值时，我们将重新渲染`Calendar`组件：

```js
// TODO #3: re-render the calendar again when there's a new date value
app.render(jsx(Calendar, { value: newDate, onChange }));
```

在最后一个`TODO`中，在`destroy`方法中，我们卸载了`Calendar`组件：

```js
// TODO #4: clean up
app.unmount();
```

就这样了。

你可以在 GitHub 上找到完整的代码：[`github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter06/03-react-calendar`](https://github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter06/03-react-calendar).

这样，你就已经编写了一个 Svelte 动作，该动作将来自不同框架的组件库（React）集成到 Svelte 中，并且以受控的方式设置和更新了组件的值。

# 摘要

在本章中，我们学习了如何使用动作将 UI 库集成到 Svelte 中，无论是用纯 JavaScript 编写的还是任何其他框架。我们通过两个真实世界的例子进行了演示——使用 Svelte 动作将 Tippy.js 和 `react-calendar` 集成到 Svelte 中。在两个例子中，我们都经历了一个逐步的过程来编写 Svelte 动作。我们首先创建了 Svelte 动作的结构，然后在动作中填充了步骤，包括当 Svelte 动作初始化为元素创建时、数据变化时以及元素从 DOM 中移除时的步骤。我们还讨论了为什么选择使用 Svelte 动作，以及集成 UI 库时的其他替代方案和考虑因素。

在下一章中，我们将探讨动作的下一个常见模式，即逐步增强你的元素。
