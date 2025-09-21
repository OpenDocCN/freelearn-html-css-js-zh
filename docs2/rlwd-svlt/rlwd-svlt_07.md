

# 带有操作的渐进增强

渐进增强是网络开发中的一种设计理念，强调为所有人提供内容和核心功能，同时为负担得起更好浏览器、更强大的硬件和更高互联网带宽的用户提供增强体验。

在本章中，我们将从更深入的角度讨论什么是渐进增强。在您的应用中实现渐进增强有许多方法；我们将通过使用 Svelte 操作来探索其中的一种。我将解释我为什么认为 Svelte 操作是为这种用例设计的。

在本章的末尾，我们将通过几个示例来展示如何使用 Svelte 操作来渐进增强我们的应用程序。

到本章结束时，您将能够构建一个遵循渐进增强原则并支持尽可能多用户设备的应用程序。

本章涵盖了以下主题：

+   什么是渐进增强？

+   为什么使用 Svelte 操作进行渐进增强？

+   使用 Svelte 操作进行渐进增强的示例

# 技术要求

您可以在此处找到本章的示例和代码：[`github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter07`](https://github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter07)。

# 什么是渐进增强？

在“渐进增强”这个短语中，最重要的事情却缺失了。这里隐含的是我们从哪里开始渐进增强。

渐进增强的主要思想是为每个人提供一个优秀的基线，包括基本内容和核心功能，无论浏览器软件、设备硬件还是互联网连接的质量如何。较旧的浏览器软件可能不支持较新的 JavaScript 语法和 CSS 功能；较旧的设备硬件可能需要更多时间来处理和渲染您的网页；较慢的互联网连接可能需要更长的时间来加载显示网页所需的资源。

我们如何确保我们的网页尽可能多地为用户保持可用性？思考一下这个问题——我稍后会回到它。

对于能够负担得起更好浏览器、更强大硬件和更高互联网带宽的用户，我们为他们提供增强体验。我们利用 JavaScript 和 CSS 的力量来惊喜和取悦我们的用户。

我们如何区分用户并决定何时提供增强体验？

有另一个术语经常与渐进增强进行比较，那就是*优雅降级*。优雅降级从功能丰富的基线开始，优雅地处理用户浏览器无法再支持功能的情况，通过替换为更简单的替代体验来处理。很多时候，这些功能从更复杂的假设开始，因此在执行上，要优雅地降级到各种用户要困难得多。

逐步增强，另一方面，从大多数用户都能使用的基线开始，通过添加更多功能逐步提升。因此，我们可以确保当新功能没有加载或无法工作时，用户仍然有一个基本可用的网页。

那么，让我们回到我们的问题：*我们如何确保我们的网页对所有用户都是可用的？* 我们将在下一节中揭开这个秘密。

## 逐步增强网络体验

其中一种方法是我们确保遵循标准。HTML、CSS 和 JavaScript 是网络的主要语言。我们确保我们只使用标准规范中的语言特性。那些已经包含在规范中较长时间的特性更有可能被所有浏览器实现。最新和最热门的特性在所有浏览器中可能不太可用。

因此，使用标准的 HTML、CSS 和 JavaScript 构建你的网页。

这就引出了下一个问题：*我们如何根据用户的浏览器、设备和* *网络能力提供差异化的用户体验？*

有许多种方法可以做到这一点。

对于大多数方法来说，一个关键的想法是分层构建你的应用程序。从核心功能的第一层开始，确保一切正常工作。然后，添加随后的层来增强体验。

## 分层构建网页

分层构建网页的一个例子是，首先在 HTML 中构建基础内容和功能作为基础层，然后使用 CSS 作为下一层添加样式、过渡和动画。最后，使用 JavaScript 作为最终层添加复杂的交互性。

这与浏览器加载你的网站的方式相一致。

每当用户访问你的网站时，浏览器首先从你的网站下载的是 HTML。HTML 描述了你的内容。例如，`<p>`、`<div>`和`<table>`标签描述了内容在屏幕上的布局方式。而`<form>`、`<input>`和`<a>`标签描述了用户如何提交数据和与内容交互。

使用 HTML，你的网站应该已经为用户提供基本的内容和功能。

如果你 HTML 中有`<link rel="stylesheet">`，并引用外部 CSS 文件，浏览器将会分别发出请求下载 CSS 资源，然后解析并应用 CSS 样式到你的文档上。这将反过来增强默认浏览器样式的视觉效果。HTML 可以提供基本的布局，但有了 CSS，你可以拥有更高级的布局，例如弹性布局、网格布局等。

另一方面，如果你的 HTML 中有`<script>`标签，浏览器将会寻找并加载引用的 JavaScript 文件，一旦 JavaScript 文件下载完成，浏览器将会解析并执行它们。JavaScript 可以用来动态地更改 DOM，执行计算，并为网站添加交互性。

没有 JavaScript，仅使用 HTML 表单即可让用户提交数据；然而，在提交后，浏览器将根据表单操作导航到新的位置。使用 JavaScript，您可以在用户继续在同一页面上浏览的同时，异步发送 HTTP 请求将数据发送到服务器。

因此，正如您所看到的，将 HTML 作为基础体验层，并在其上添加 CSS 和 JavaScript 以提供增强体验，这是一种渐进式增强。

拥有较旧浏览器、较慢硬件和较低互联网带宽的用户仍然可以在等待 CSS 和 JavaScript 下载和执行以获得更丰富体验的同时，仅使用 HTML 查看和与您的网站互动。

希望您现在已经对第一种 HTML 方法深信不疑。但假设您的网站内容是动态的？您如何为用户生成动态 HTML？您需要编写单独的代码来生成动态 HTML 吗？

不，您不需要。

Svelte 支持 **服务器端渲染**（**SSR**）。这意味着相同的 Svelte 组件可以用于在浏览器上渲染内容，同时在服务器端生成 HTML。

您可以自己设置它（然而，这超出了本书的范围），或者您可以使用像 SvelteKit 这样的元框架，它全面概述了所有应该如何工作。

从这里可以吸取的一点是，无论您的设置如何，您都可以像这样编写您的 Svelte 组件，并且您编写的相同 Svelte 组件代码可以用于在服务器端生成 HTML 和在浏览器端渲染内容。

这引发了一个问题：所有代码在服务器端和浏览器端是否都以相同的方式工作？是否有只在服务器端运行而不在浏览器端运行的代码，或者相反？

好吧，并非所有代码都在服务器端和浏览器端运行。Svelte 动作，以及 `bind:` 指令和 `on:` 事件监听器，是 Svelte 功能，它们不会在服务器端和浏览器端同时运行。Svelte 动作仅在浏览器端运行，不在服务器端运行。这是因为 Svelte 动作在元素添加到 DOM 之后运行。由于在服务器端生成 HTML 字符串时没有 DOM，因此 Svelte 动作不会在服务器端运行。

这使得 Svelte 动作成为渐进式增强的完美候选者。

## Svelte 动作用于渐进式增强

在上一节中，我们学习了渐进式增强以及以分层构建网页以实现渐进式增强的概念。

在本节中，我们将更深入地探讨 Svelte 动作的作用，这些动作使我们能够为现有的 HTML 元素添加额外的交互层，使它们非常适合创建渐进式增强。

让我们从检查一个 Svelte 代码示例开始，以了解 Svelte 动作如何融入这种方法。

现在想象以下代码：

```js
<button use:enhance />
```

当从服务器端渲染时，您得到的 HTML 看起来像这样：

```js
<button></button>
```

仅凭 HTML `button` 元素本身应该能够完成按钮元素应有的功能：可点击并能提交表单。

但随着 JavaScript 的加载和 Svelte 组件代码的执行，`enhance` 动作就会在 `button` 元素上运行，允许动作代码增强 `button` 元素。

这里有一些动作可能执行的操作示例：在悬停时显示有用的工具提示，在按下时提供加载指示器，等等。

对于那些在客户端运行 Svelte 组件代码时遇到困难的旧版浏览器用户，他们仍然可以使用和交互默认的 HTML 按钮元素，并体验一个未增强的网页版本。

从本节中我们可以了解到，Svelte 动作使我们能够为现有的 HTML 元素添加另一层交互性。它们是设计渐进增强的理想候选者。

因此，让我们看看一些使用动作来逐步增强 HTML 元素的示例。

我们将要查看的第一个示例是逐步增强 `<a>` 元素。

# 示例 - 使用 <a> 元素预览链接

在我们的第一个示例中，我们将探讨如何逐步增强 `<a>` 元素，以便在悬停时显示预览。

在这里，浏览器接收到的 HTML 包含一个具有 `href` 属性的 `<a>` 标签，如下所示：

```js
<a href="..." />
```

然后创建一个超链接。当你点击超链接时，浏览器将导航到 `href` 属性中指定的目标。

这是 `<a>` 元素的默认行为。

当用户加载 JavaScript 时，我们希望 `<a>` 元素不仅仅能导航到新位置。

我们将通过使元素在悬停时显示目标位置来增强 `<a>` 元素。

为了做到这一点，我们将创建一个 `preview` 动作并在 `<a>` 元素上使用它：

```js
<script>
  function preview(element) {
  }
</script>
<a href="..." use:preview>Hello</a>
```

无论在服务器端渲染前面的代码时 `preview` 动作是如何实现的，Svelte 都会生成以下 HTML：

```js
<a href="..." />Hello</a>
```

这是因为 Svelte 动作永远不会在服务器端运行。

一旦用户收到 HTML 响应，他们就可以开始点击链接并导航到新位置。现在你有一个仅使用 HTML 就能工作的应用程序。

根据用户的网络条件，从你的 Svelte 组件编译的 JavaScript 代码可能需要更长的时间才能到达。但这并不会阻止用户使用超链接进行导航。

只有当 JavaScript 加载并执行时，Svelte 才会在 DOM 上的 `<a>` 元素上运行 `preview` 动作，并增强 `<a>` 元素的行为。

这里的关键是，尽可能多地使我们的应用程序的核心功能仅通过 HTML 就能工作，并通过 JavaScript 添加一个增强层，这取决于用户的网络条件，可能要晚得多。

足够了关于渐进增强哲学。让我们看看我们如何实现这个 `preview` 动作。

我们希望 `preview` 动作在将鼠标光标移至链接时显示包含链接内容的浮动弹出窗口，并在我们将鼠标光标移开时隐藏它。

我们可以通过 `'mouseover'` 和 `'mouseout'` 事件来实现这一点。下面是如何操作的：

```js
function preview(element) {
  element.addEventListener('mouseover', onMouseOver);
  element.addEventListener('mouseout', onMouseOut);
  return {
    destroy() {
          element.removeEventListener('mouseover', onMouseOver);
          element.removeEventListener('mouseout', onMouseOut);
        }
  };
}
```

在前面的代码片段中，我们在 `preview` 动作开始时添加了 `'mouseover'` 和 `'mouseout'` 事件监听器。此外，我们通过在 `destroy` 方法中移除这两个事件监听器来确保适当的清理。在我们弄清楚如何实现 `onMouseOver` 和 `onMouseOut` 之前，我们首先需要决定浮动弹出窗口的外观以及如何在 DOM 中布局它。

为了显示链接目标的内联内容，我们将使用 `<iframe>` 元素，它允许我们将另一个 HTML 页面嵌入到当前页面中：

```js
<iframe src="img/..." />
```

为了使 `<iframe>` 元素浮在其他内容之上而不是成为文档流的一部分，我们需要通过使用 `position: fixed` 或 `position: absolute` 来修改 `<iframe>` 元素的 CSS `position` 属性。

如果我们在 `<iframe>` 元素上使用 `position: fixed`，那么 `<iframe>` 元素将相对于视口进行定位。为了将 `<iframe>` 元素放置在 `<a>` 元素旁边，我们需要确定 `<a>` 元素相对于视口的定位，并计算顶部和左侧的值以放置我们的 `<iframe>` 元素。

另一方面，如果我们使用 `position: absolute`，那么 `<iframe>` 元素将相对于最近的定位父元素进行定位。我们可以将 `<iframe>` 元素放在 `<a>` 元素内部，并通过在 `<a>` 元素上指定 `position: relative` 来使其成为定位父元素（`position: relative` CSS 属性是相对于其当前位置进行定位）。然后 `<iframe>` 元素将相对于其父元素，即 `<a>` 元素进行定位。

两种方法都有其优缺点。在这里，我将使用第二种方法，即使用 `position: absolute`。我需要修改内容和 `<a>` 元素的 `position` CSS 属性，但如果我使用 `position: absolute` 而不是 `position: fixed`，我就可以避免进行计算。

下面是我们将 `<iframe>` 元素放入 `<a>` 元素内部后 DOM 的样子：

```js
<a href="..." style="position: relative">Hello<iframe src="img/..." style="position: absolute"/></a>
```

我们现在的任务是使用程序在 `onMouseOver` 函数中创建并插入 `<iframe>` 元素。以下代码片段展示了如何实现这一点：

```js
function preview(element) {
  // make the <a> element position relative
  element.style.position = 'relative';
  let iframe;
  function onMouseOver() {
    iframe = document.createElement('iframe');
    iframe.src = element.getAttribute('href');
    iframe.style.position = 'absolute';
    iframe.style.left = 0;
    iframe.style.top = '100%';
    element.appendChild(iframe);
  }
  // ...
}
```

在前面的代码片段中，我们将 `<a>` 元素的 CSS `position` 属性设置为 `'relative'`。在鼠标悬停在 `<a>` 元素上时将被调用的 `onMouseOver` 函数中，我们将程序化创建一个 `<iframe>` 元素，设置其样式，并将其插入到 `<a>` 元素中。

在前面的代码中，我们使用了 DOM API，如 `document.createElement()` 和 `element.appendChild()`。由于 `<iframe>` 元素是程序创建的，我们也在程序上修改了它的 `style` 属性。

幸运的是，在这个例子中，我们只创建了一个元素，但你可以想象，如果我们创建更多元素，这会多么容易变得混乱。

由于我们在这里学习 Svelte，而 Svelte 是设计用来将这些命令式 DOM 指令抽象成声明式 Svelte 语法，为什么不在我们的操作中利用 Svelte 呢？

我们可以用 Svelte 组件替换之前的命令式代码，如下所示：

```js
<!-- IframePopup.svelte -->
<script>
  export let src;
</script>
<iframe {src} />
<style>
  iframe {
    position: absolute;
    left: 0;
    top: 100%;
  }
</style>
```

前面的代码片段显示了一个包含 `<iframe>` 元素的 Svelte 组件。这个 `<iframe>` 元素与我们之前代码中程序创建的元素等效。它应用了相同的 CSS 样式。

Svelte 组件暴露了一个名为 `src` 的属性，`src` 属性的值将被用来设置 `<iframe>` 元素的 `src` 属性的值。现在，我们不再需要调用 DOM API 来创建 `<iframe>` 元素，而是可以实例化我们的 Svelte 组件，并将期望的 `src` 值作为 `src` 属性传递给组件。在 Svelte 中，你可以通过使用 `new` 关键字和组件的构造函数来实例化一个组件，将任何所需的属性作为构造函数参数的一部分传递。然后，Svelte 组件将根据传递的属性渲染具有指定 `src` 属性值的 `<iframe>` 元素。这简化了在 Svelte 应用程序中创建和管理 `<iframe>` 元素的过程：

```js
import IframePopup from './IframePopup.svelte';
function preview(element) {
  // ...
  function onMouseOver() {
    iframe = new IframePopup({
      // target specifies where we want the component
      // to be inserted into
      target: element,
      // we are passing the href value into the
      // component through props
      props: { src: element.getAttribute('href') },
    });
  }
  // ...
}
```

在前面的代码片段中，我们将 `onMouseOver` 函数中的 DOM 操作替换为实例化 `IframePopup` Svelte 组件。

当我们移动鼠标离开链接时，我们需要记住移除弹出窗口。以下是我们可以这样做的方法：

```js
function onMouseOut() {
  iframe.$destroy();
}
```

虽然我们只通过 Svelte 动作将一个 HTML 元素插入 DOM，但我们已经看到，将其封装到一个 Svelte 组件中并实例化该组件，而不是手动创建 HTML 元素，要容易管理得多。

然后，我们可以利用 Svelte 创建作用域样式，以及向我们将要创建的元素添加其他交互逻辑。

我们还可以向元素添加过渡效果；而不是在悬停时突然出现，我们可以使用 `fade` 过渡效果使弹出窗口淡入，如下所示：

```js
<!-- IframePopup.svelte -->
<script>
  import { fade } from 'svelte/transition';
  // ...
</script>
<iframe {src} transition:fade />
```

默认情况下，当组件首次创建时，不会播放过渡效果。因此，我们需要传递 `intro: true` 选项来播放过渡效果：

```js
iframe = new IframePopup({
  target: element,
  props: { src: element.getAttribute('href') },
  // play transition when created
  intro: true,
});
```

在前面的代码片段中，我们将 `intro: true` 传递给 `IframePopup` 构造函数。

我们现在有一个显示在弹出窗口中的链接，悬停时会淡入。尝试模拟使用慢速网络加载页面。大多数浏览器都提供开发者工具来模拟网络速度。例如，如果你使用 Google Chrome，那么你可以打开**开发者工具**，找到**网络条件**选项卡，查找**网络限制**部分，并选择**慢速** **3G**预设。

尝试重新加载你的页面，你会发现一旦你看到链接（尽管 JavaScript 文件仍在加载），链接就会立即生效；点击它将带你到目的地。随着 JavaScript 最终加载到浏览器中，你的链接现在得到了增强，你现在能够悬停在链接上并看到预览。

我们的链接预览动作完成后，让我们看看网络应用中的另一个常见组件，表单，并看看我们如何逐步增强表单元素。

# 示例 - 逐步增强表单

一个`<form>`元素是文档的一部分，可以包含用于提交信息的输入。

默认情况下，当你提交表单时，浏览器将导航到 URL 以处理表单提交。这意味着当用户提交表单并离开当前页面时，他们会丢失他们所在的状态。

然而，通过浏览器`fetch` API 的异步请求能力，我们现在可以通过 API 请求提交数据，而无需离开当前页面，并保持在原地。

这意味着如果网站正在播放音乐、视频或动画，它们在我们进行异步 API 调用时仍然会播放。

我们现在的任务是创建一个动作来增强表单元素，使得增强后的表单不会导航到新位置，而是异步提交表单数据。

由于没有更好的名字，我将把这个增强动作称为`enhance`。

在我们继续实施`增强`动作之前，让我们回顾一下默认表单行为。

## 默认表单行为

当你有一个`<form>`元素时，默认情况下，当你点击`action`属性时，它会携带`<form>`元素内填充的`<input>`元素的值。

例如，想象你有一个以下表单：

```js
<form action="/foo">
  <input name="name" />
  <input name="address" />
  <button>Submit</button>
</form>
```

当你输入`/foo?name=xxx&address=yyy`，通过查询参数携带表单数据。

一个`<form>`元素可以通过指定`method`属性来定义用于提交表单的 HTTP 方法。

例如，以下表单将通过`POST`请求导航到`/foo`：

```js
<form action="/foo" method="post">...</form>
```

表单数据将以请求体形式发送到`POST`请求。

根据服务器对`/foo`端点的实现，服务器可以选择如何处理数据以及要在`/foo`页面上显示什么。有时，服务器可能会决定在处理数据后重定向回当前页面。在这种情况下，有一个可以替换默认表单动作并异步提交表单数据的动作将非常有用，因为我们最终会回到同一个页面。

现在我们知道了默认的表单行为，让我们弄清楚我们需要实现`enhance`动作所需的内容。

## 实现增强动作

让我们分解这个问题。

首先，我们需要弄清楚何时用户提交表单。然后，我们需要防止默认的表单行为，然后异步调用 API 提交表单，最后将表单重置到初始状态，类似于服务器重定向回同一页面后的情况。

为了确定用户何时提交表单，我们可以监听`<form>`元素上的`'submit'`事件：

```js
form.addEventListener('submit', handleSubmit);
```

为了防止默认的表单行为，我们在`'submit'`事件监听器上调用`event.preventDefault()`以防止默认提交行为：

```js
function handleSubmit(event) {
  event.preventDefault();
}
```

为了异步提交表单的 API 调用，我们首先需要找出我们正在提交表单的位置。我们可以从读取`form`实例的`action`属性来获取这个信息：

```js
form.action; // "https://domain/foo"
```

我们还可以从`form`实例中确定首选的 HTTP 请求方法：

```js
form.method // "post"
```

要获取提交的表单数据，我们可以使用`FormData`接口：

```js
const data = new FormData(form);
```

在有了 URL、请求方法和数据后，我们可以使用`fetch` API 提交表单：

```js
fetch(form.action, {
  method: form.method,
  body: new FormData(form),
});
```

最后，为了重置表单，我们可以使用`reset()`方法：

```js
form.reset()
```

将所有内容整合起来，我们得到以下内容：

```js
<script>
  function enhance(form) {
    async function handleSubmit(event) {
      event.preventDefault();
      const response = await fetch(form.action, {
        method: form.method,
        body: new FormData(form),
      });
      form.reset();
    }
    form.addEventListener('submit', handleSubmit);
    return {
      destroy() {
        form.removeEventListener('submit', handleSubmit);
      }
    };
  }
</script>
<form action="/foo" method="post" use:enhance>...</form>
```

现在，尝试在 JavaScript 加载后提交表单。你会发现有一个网络请求被用来提交表单，而你仍然停留在同一页面上，没有导航离开。

尝试禁用 JavaScript 或模拟缓慢的网络速度。你会发现你仍然可以提交表单，而 JavaScript 仍在加载。然而，这一次，你通过默认的浏览器行为提交，这会将你从页面上导航离开。

这里就是它——默认可用的可工作表单，但通过渐进增强可以在 JavaScript 加载时无需离开页面提交表单数据。

`enhance`动作有很多可以改进的地方。我将把它留作练习：

+   修改`enhance`动作以允许传递一个回调函数，该函数将在表单提交成功后调用。

+   如果表单提交失败会发生什么？你应该如何处理这种情况？

+   目前，`enhance`动作通过请求体提交表单数据；然而，当表单方法为`"get"`时，表单数据应通过查询参数传递。修改`enhance`动作以处理`"get"`表单方法。

# 摘要

在本章中，我们解释了渐进增强是什么以及为什么它很重要。在此基础上，我们学习了如何使用 Svelte 动作来渐进增强我们的元素。

我们已经探讨了两种不同的渐进增强示例——增强一个链接以显示预览弹出窗口，以及增强表单元素以异步提交表单。

在过去的三章中，我们看到了 Svelte actions 的三种不同模式和用例，包括创建自定义事件、集成 UI 库和渐进式增强。Svelte actions 可以做的不仅仅限于我们之前讨论的三个不同用例，但希望这些模式已经激发了你的想象力，让你看到了 Svelte actions 的可能性。

在此基础上，我们将继续探索本书的下一部分。我们将从 *第八章* 到 *第十二章* 探索 Svelte 上下文和 Stores 的各种用例，例如在状态管理、创建无渲染组件以及用于动画。我们将在下一章中定义并比较 Svelte 上下文和 Svelte Stores。

# 第三部分：上下文和 Stores

在本部分，我们将深入探讨 Svelte 的两个核心特性：Svelte 上下文和 Svelte Stores。在接下来的五章中，我们将探索使用 Svelte 上下文和 Stores 的不同场景。我们的探索将从定义 Svelte 上下文和 Stores 开始。随后，我们将深入研究实现自定义 Stores 和使用 Svelte Stores 管理应用程序状态的战略。接着，我们将学习如何使用 Svelte 上下文创建无渲染组件。最后，我们将学习如何使用 Svelte Stores 创建动画。

本部分包含以下章节：

+   *第八章**，上下文与 Stores 对比*

+   *第九章**，实现自定义 Stores*

+   *第十章**，使用 Svelte Stores 进行状态管理*

+   *第十一章**，无渲染组件*

+   *第十二章**，Stores 和动画*
