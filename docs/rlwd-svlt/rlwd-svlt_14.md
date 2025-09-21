

# 第十四章：探索自定义过渡

在本章中，我们将深入探讨 Svelte 中的自定义过渡。到目前为止，我们已经探讨了 Svelte 的内置过渡以及如何使用它们创建引人入胜和动态的用户界面。然而，可能存在内置过渡无法完全满足你的要求，而你希望创建更独特的东西的情况。这就是自定义过渡发挥作用的地方。

自定义过渡允许你完全控制你希望在 Svelte 应用程序中实现的动画和效果。本章将指导你创建自己的自定义过渡，无论它们是基于 CSS 还是 JavaScript。我们将探讨过渡合约，它是创建自定义过渡的基础，并提供实际示例以帮助你入门。

到本章结束时，你将牢固地理解如何在 Svelte 中创建自定义过渡，并且你将具备在项目中实现它们的知识，将你的用户界面提升到新的水平。

本章包括以下主题的章节：

+   过渡合约

+   使用`css`函数编写自定义 CSS 过渡

+   使用`tick`函数编写自定义 JavaScript 过渡

# 技术要求

本章将包含大量的代码，但请放心——你可以在 GitHub 上找到本章中使用的所有代码示例，网址为[`github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter14`](https://github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter14)。

# 过渡合约

在我们深入创建自定义过渡之前，了解它们建立的基础是至关重要的：过渡合约。

如果你已经阅读了*第九章*，你将熟悉存储合约的概念。正如存储是一个遵循特定存储合约的对象一样，过渡是一个遵循过渡合约的函数。通过理解和遵守这些合约，你可以创建与 Svelte 内置过渡系统无缝集成的自定义过渡。

过渡合约由一个负责过渡的单个函数组成。这个函数接受两个参数：

+   `节点`：应用过渡的目标 DOM 元素

+   `params`：包含配置选项的对象

函数应返回一个描述如何执行过渡的对象。我们将在本节稍后深入探讨这个返回对象的细节。

这里是一个遵循过渡合约的自定义过渡示例：

```js
function customTransition(node, params) {
  const config = { ... };
  return config;
}
```

在前面的代码片段中，我们创建了一个名为`customTransition`的自定义过渡。我们通过声明一个接受两个参数的`customTransition`函数来实现这一点：`node`和`params`。然后，这个函数返回一个对象——我们将称之为`config`——它描述了过渡。

为了将我们刚刚创建的自定义过渡与 Svelte 中过渡的使用联系起来，这里我们看到 `customTransition` 函数是如何应用于 `<div>` 元素的：

```js
<div transition:customTransition={{ duration: 500 }}>some text</div>
```

当 `<div>` 元素被插入或即将从 DOM 中移除时，Svelte 将尝试播放过渡。Svelte 通过调用 `customTransition` 函数并传递 `<div>` 元素和传递给过渡的 `config` 对象来实现这一点：

```js
const config = customTransition(div, { duration: 500 });
```

由 `customTransition` 返回的此 `config` 对象将决定过渡如何播放。

现在，让我们关注自定义过渡函数返回的 `config` 对象的要求。

从自定义过渡返回的 `config` 对象应至少包含以下属性或方法之一：

+   `delay`: 以毫秒为单位的数字。这指定了在过渡开始之前需要等待多长时间。

+   `duration`: 以毫秒为单位的数字。指定过渡播放的持续时间。这决定了动画对用户来说看起来有多快或多慢。

+   `easing`: 用于缓动的函数。此函数确定过渡进度随时间变化的速率。

+   `css`: 一个带有两个参数的函数：`progress` 和 `remaining`。在这里，`progress` 是一个介于 `0` 和 `1` 之间的值，表示过渡的进度，而 `remaining` 参数的值等于 `1 - progress`。

    此函数应返回一个包含要应用于目标 DOM 元素的 CSS 样式的字符串。

+   `tick`: 一个在过渡期间重复调用的函数，带有两个参数：`progress` 和 `remaining`。在这里，`progress` 是一个介于 `0` 和 `1` 之间的值，表示过渡的进度，而 `remaining` 参数的值等于 `1 - progress`。

    此函数可用于根据当前进度更新 DOM 元素的样式。

这是一个遵循过渡契约的更完整的自定义过渡示例：

```js
import { cubicInOut } from 'svelte/easing';
function customTransition(node, params) {
  return {
    duration: 1000,
    delay: 500,
    easing: cubicInOut,
    css: (progress) => `opacity: ${progress}`,
  };
}
```

在前面的代码片段中，我们的自定义过渡（命名为 `customTransition`）返回一个对象，描述了过渡的 `duration`、`delay`、`easing` 和 `css` 样式。

在上一章中，当我们将过渡应用于元素时，我们看到了 `delay`、`duration` 和 `easing`。由于这些属性的行为在此上下文中保持不变，让我们关注一些新的内容：`css` 和 `tick` 函数。

## `css` 函数

如您可能记得，在上一章的最后部分，*Svelte 过渡在底层是如何工作的*，Svelte 使用 CSS 和 JavaScript 的组合来动画化过渡。它利用 CSS `@keyframe` 规则以及 CSS 动画的 `animation` 属性和 JavaScript 动画的 `requestAnimationFrame` 函数。

`css`函数用于生成自定义过渡的 CSS `@keyframe`规则。如果`css`函数定义在自定义过渡返回的对象中，Svelte 将在元素被插入到 DOM 或即将从 DOM 中移除时调用此函数。Svelte 将根据过渡的持续时间和缓动效果，多次调用`css`函数，以生成适当的`@keyframe`规则。

`css`函数的第一个参数与过渡的进度相关。`progress`是一个介于`0`和`1`之间的数字，其中`0`表示元素不可见，`1`表示元素在屏幕上的最终位置。

例如，当元素被插入到 DOM 中后进行过渡时，`progress`的值从`0`开始，逐渐移动到`1`。相反，当元素在从 DOM 中移除之前进行过渡时，`progress`的值从`1`开始，逐渐移动到`0`。

您可以使用`progress`来计算创建自定义过渡所需的 CSS 样式。

例如，如果我们想创建一个将元素从透明渐变到完全可见的过渡效果，我们可以使用`progress`来计算过渡过程中的透明度值：

+   当元素不可见（`progress`的值为`0`）时，我们希望元素是透明的（`opacity`的值应为`0`）

+   当元素可见（`progress`的值为`1`）时，我们希望元素完全可见（`opacity`的值应为`1`）

`progress`与`opacity`之间的关系可以用*图 14**.1*中的图表表示：

图 14.1：进度与透明度的关系

我们将在`css`函数中从`progress`的值推导出`opacity`的值，如下所示：

```js
function customTransition(node, params) {
  return {
    css: (progress) => `opacity: ${progress}`,
  };
}
```

在前面的代码中应用过渡将使元素在插入 DOM 时从透明渐变到可见，并在从 DOM 中移除时渐变回透明。

让我们再举一个例子。让我们创建一个将元素从右侧飞到最终位置的过渡效果。在这里，元素的平移在整个过渡过程中发生变化，我们可以使用`progress`来计算平移：

+   当元素不可见（`progress`的值为`0`）时，我们希望元素位于右侧（`translateX`的值为`100px`）

+   当元素可见（`progress`的值为`1`）时，我们希望元素处于其最终位置（`translateX`的值为`0px`）

这里有一个描述`progress`与`translateX`之间关系的图表：

图 14.2：进度与 translateX 的关系

与前面的例子不同，`translateX`的值是`progress`的倒数：当`progress`为`0`时，`translateX`有一个非零值；当`progress`为`1`时，`translateX`变为`0`。

因此，为了计算`translateX`的值，我们使用`1 – progress`乘以一个值，如以下代码片段所示：

```js
function customTransition(node, params) {
  return {
    css: (progress) => `transform: translateX(${(1 – progress) * 100}px)`,
  };
}
```

当应用前面的代码片段中的 `customTransition` 函数到元素上时，随着元素被添加到 DOM 中，该元素将从右侧飞入其最终位置。并且由于计算 `progress` 的倒数（`1 – progress`）是如此常见，这个值被作为 `css` 函数的第二个参数提供。

因此，这里再次是我们的自定义转换，但这次使用第二个参数来计算平移：

```js
function customTransition(node, params) {
  return {
    css: (progress, remaining) => `transform: translateX(${remaining * 100}px)`,
  };
}
```

`css` 函数返回一个可以包含多个 CSS 声明的 CSS 字符串。你用分号分隔每个 CSS 声明，就像在元素的 `style` 属性中做的那样。例如，让我们创建一个同时实现淡入和从右侧平移的转换：

```js
function customTransition(node, params) {
  return {
    css: (progress, remaining) => `opacity: ${progress}; transform: translateX(${remaining * 100}px); `,
  };
}
```

在前面的代码片段中，我们同时实现了淡入和从右侧平移。返回的 CSS 字符串包含多个用分号分隔的 CSS 声明，一个用于 `opacity`，另一个用于 `transform`，这些声明将在转换期间应用。

现在我们已经介绍了 `css` 函数，让我们来看看 `tick` 函数。

## `tick` 函数

`tick` 函数是创建自定义转换的 `css` 函数的替代品。与用于生成动画 CSS `@keyframe` 规则的 `css` 函数不同，`tick` 函数允许你使用 JavaScript 动画化转换。这可以提供对转换的更精细控制，从而创建出仅使用 CSS 难以实现的更复杂的动画。

`tick` 函数在通过 `requestAnimationFrame` 进行转换期间被反复调用。与 `css` 函数类似，`tick` 函数接受两个参数：`progress` 和 `remaining`。`progress` 参数是一个介于 `0` 和 `1` 之间的值，其中 `0` 表示元素处于不可见状态，而 `1` 表示元素位于屏幕上的最终位置，而 `remaining` 参数等于 `1 – progress`。这些参数可以根据转换的当前进度修改应用到的 DOM 元素。

例如，如果我们想使用 `tick` 函数创建淡入转换，可以根据进度值更新元素的透明度，如下所示：

```js
function customTransition(node, params) {
  return {
    tick: (progress) => {
      node.style.opacity = progress;
    },
  };
}
```

根据前面的代码片段，Svelte 在转换过程中的每一帧都会触发 `tick` 函数。

当元素开始出现时，`progress` 值为 `0`，我们使用这个 `progress` 值将元素的初始 `opacity` 值设置为 `0`。

随着转换的进行，`tick` 函数会使用介于 `0` 和 `1` 之间的 `progress` 值被调用，并且我们会根据 `progress` 值更新元素的 `opacity` 值。

在转换结束时，`tick` 函数最后一次使用 `progress` 值为 `1` 被调用。此时，我们将元素的 `opacity` 值设置为最终的 `1`。

代码片段中的`tick`函数与使用`css`函数创建的自定义淡入过渡操作类似。两种方法都在整个过渡过程中修改元素的`opacity`值。关键区别在于它们的执行方式。

Svelte 在过渡开始时多次调用`css`函数，并使用不同的进度值来构建 CSS 的`@keyframe`规则。一旦完成，`css`函数在过渡期间不再被调用。新创建的 CSS`@keyframe`规则随后通过 CSS 的`animation`属性应用于元素。然后通过 CSS 更新元素的`opacity`值。

另一方面，`tick`函数在过渡的每个动画帧中由 Svelte 多次调用。在每次`tick`调用中，元素的`opacity`值通过 JavaScript 进行修改。

现在我们已经了解了过渡合约，让我们利用这些知识来创建更多自定义过渡。

# 使用`css`函数编写自定义 CSS 过渡

我们将要一起尝试编写的第一个自定义过渡是一种在演示中经常看到的效果，通常被称为“颜色滑动”。这种效果因其动态的颜色扫动而突出，这种扫动在屏幕上流动，创造出一种吸引观众注意力的能量感。

如其名所示，颜色滑动过渡涉及在对象上发生的颜色扫动变化。

想象一下：你正在查看一个静态屏幕，可能是网站的一部分。突然，一种新颜色从屏幕的一边开始显现。就像波浪一样，这种颜色在屏幕上扩散，包围了整个屏幕。一旦颜色完全覆盖了屏幕，它开始从起始边缘退去，揭示新的内容。当颜色完全消失后，新的内容完全展现出来：

图 14.3：颜色滑动过渡

*滑动*可以从任何方向移动——它可以水平地从左到右移动，垂直地从上到下移动，甚至可以斜向移动。

我们将修改颜色滑动过渡，使其应用于段落（`<p>`）元素。当`<p>`元素被添加到 DOM 中时，一道颜色波将扫过它，在过渡完成后揭示`<p>`元素内的文本。当`<p>`元素从 DOM 中移除时，过渡将反向播放，在过渡完成后隐藏文本。

过渡的视觉演示可以在这里看到：

图 14.4：段落上的颜色滑动过渡

在本节中，我们将逐步介绍如何使用 Svelte 创建这种迷人的颜色滑动过渡。

由于当`<p>`元素从 DOM 中移除时隐藏文本的过渡与当`<p>`元素被添加时显示文本的过渡相同，但播放方向相反，因此我们将关注当`<p>`元素被添加到 DOM 时播放的过渡。这是因为当过渡应用于一个元素时，Svelte 会在元素被移除时播放相同的过渡，但方向相反。因此，通过关注元素被添加到 DOM 时播放的过渡，我们实际上覆盖了两种情况。

因此，让我们开始创建一个过渡。

首先，让我们创建我们自定义过渡的结构。回想一下过渡合约——一个过渡是一个返回描述过渡的对象的函数：

```js
<script>
  function colourSwipe(node) {
    // TODO: implement the transition here
    const config = {};
    return config;
  }
</script>
<p transition:colourSwipe>Some text here</p>
```

在前面的代码片段中，我们创建了一个`colourSwipe`过渡并将其应用于`<p>`元素。我们当前的任务是实现`colourSwipe`过渡，通过填充`config`对象来完成。

我们将要添加到`config`对象的前两个字段是`duration`和`delay`。如以下代码片段所示，我们将过渡的持续时间设置为 1 秒，并且过渡将没有延迟开始：

```js
function colourSwipe(node) {
  const config = {
    duration: 1000,
    delay: 0,
  };
  return config;
}
```

然而，在创建自定义过渡时，通常你可能希望允许过渡的使用者根据过渡应用的位置来自定义持续时间和延迟。

例如，一个过渡的使用者可能希望通过在`transition:`指令中指定它们来有一个 200 毫秒的延迟和 2 秒的持续时间，如下面的代码片段所示：

```js
<div transition:colourSwipe={{ delay: 200, duration: 2000 }} />
```

在`transition:`指令中指定的这些自定义延迟和持续时间将作为第二个参数传递给`colourSwipe`过渡，它将在`config`对象中使用它们：

```js
function colourSwipe(node, params) {
  const config = {
    duration: params?.duration ?? 1000,
    delay: params?.delay ?? 0,
  };
  return config;
}
```

在前面的代码片段中，我们使用`config`对象中的`params.duration`和`params.delay`的值，并在这些参数没有明确指定时提供默认值。

既然我们已经指定了过渡的`delay`和`duration`字段，让我们将注意力转向下一个字段——`easing`。

我们将使用`linear`（线性）缓动，使过渡以恒定速度移动，没有任何加速或减速。就像我们对`duration`或`delay`所做的那样，我们将使`easing`可由用户自定义。因此，在以下代码片段中，我们根据用户指定的缓动设置`easing`的值。如果它被省略，我们将回退到我们的默认缓动——`linear`缓动：

```js
import { linear } from 'svelte/easing';
function colourSwipe(node, params) {
  const config = {
    duration: params?.duration ?? 1000,
    delay: params?.delay ?? 0,
    easing: params?.easing ?? linear,
  };
  return config;
}
```

通常情况下，在创建自定义过渡的过程中，`duration`（持续时间）、`delay`（延迟）和`easing`（缓动函数）字段是最容易设置的。大多数情况下，我们会确定默认的`duration`、`delay`和`easing`值，然后允许用户根据自己的喜好调整这些值。

在确定了`duration`、`delay`和`easing`值之后，我们现在深入过渡的核心：为过渡元素编写 CSS。

如果你仔细观察过渡，你会注意到过渡可以分为两个不同的阶段：前半部分涉及颜色块扩展以包围整个`<p>`元素，而后半部分则对应颜色块收缩以揭示`<p>`元素内的文本：

图 14.5：颜色滑动过渡分为两个部分，由虚线分隔

让我们探索如何创建这些 CSS 规则，以有效地实现过渡的各个阶段。

在开始之前，让我们在我们的`colourSwipe`过渡中创建一个`css`函数：

```js
function colourSwipe(node, params) {
  const config = {
    duration: params?.duration ?? 1000,
    delay: params?.delay ?? 0,
    easing: params?.easing ?? linear,
    css: (progress) => {}
  };
  return config;
}
```

我们将在前面的代码片段中填充`css`函数。

需要注意的关键点是，`progress`值在过渡开始时从`0`开始，并在过渡结束时达到`1`。由于我们将过渡分为两个阶段，第一阶段将看到`progress`值从`0`移动到`0.5`，而第二阶段则从`0.5`推进到`1`。

因此，在我们的`css`函数中，我们将为过渡的不同阶段实现不同的 CSS 规则：

```js
css: (progress) => {
  if (progress <= 0.5) {
  } else {
  }
}
```

在前面的代码片段中，你可以看到我们在`css`函数中添加了一个`if`块，该块使用`progress`值来确定要应用哪些 CSS 规则集。对于过渡的前半部分（`progress` <= 0.5），实现了第一组 CSS 规则。对于后半部分（`progress` > 0.5），使用第二组规则。这样，我们可以在过渡的两个阶段以不同的方式定制元素的外观。

在我们的过渡的前半部分，我们需要创建一个增长的颜色块。为了创建这个，我们将在元素的背景上应用一个线性渐变。渐变将从实心颜色过渡到透明颜色。通过将实心颜色和透明颜色的颜色停止点对齐在相同的位置，我们可以在渐变过渡中创建一个尖锐的硬线。

例如，如果我们想要一个占据元素左侧 25%的实心红色颜色块，我们可以应用以下 CSS：

```js
background: linear-gradient(to right, red 0, 25%, transparent 25%);
```

在前面的代码片段中，我们有一个从左到右移动的线性渐变，红色和透明颜色在 25%的位置共享相同的颜色停止点。这就在渐变的左侧 25%处创建了一个实心红色块，而剩余的 75%是透明的。

我们选择使用线性渐变来实现这个颜色块，而不是叠加另一个元素，这显示了这种方法的高效性。它消除了创建额外元素的需要。

在背景上设置线性渐变而不是调整元素大小的好处是，可以在不实际调整元素大小的情况下产生调整颜色块的效果，这避免了在 DOM 中引起重新布局和布局偏移。这样，应用了 CSS 的元素在整个过渡过程中保持在其原始位置和大小不变。

因此，现在我们已经确定了要使用的 CSS，让我们将其纳入我们过渡的`css`函数中。

在此之前，我们需要做一些数学计算。我们打算使用 `progress` 的值来计算实色块应覆盖的元素百分比。

在过渡的第一半中，`progress` 的值从 `0` 到 `0.5`。

在这个过渡阶段，需要覆盖的元素百分比应从 0% 到 100%。

因此，通过进行算术计算，我们可以得出结论，百分比值是 `progress` 值的 200 倍：

```js
const percentage = progress * 200;
```

现在我们将这个整合到我们的 `css` 函数中：

```js
css: (progress) => {
  if (progress <= 0.5) {
    const percentage = progress * 200;
    return `background: linear-gradient(to right, red 0, ${percentage}%, transparent ${percentage}%);`;
  } else {
  }
}
```

在前面的代码片段中，我们使用计算出的百分比来控制实色块的大小，使其在过渡的第一半中从左侧的 0% 增长到 100%。

现在，让我们将注意力转向过渡的第二部分，其中实色块从元素的全宽向右边缘收缩。

另一种设想这个问题的方法是考虑从左侧边缘开始的透明部分的扩展，覆盖从 0% 到 100% 的元素。这反映了过渡的第一半，关键区别在于现在增长到完全包围元素的是透明颜色，而不是实色。

计算百分比值的公式保持不变，但由于 `progress` 值现在从 `0.5` 到 `1` 变化，我们需要在将其乘以 200 之前从 `progress` 值中减去 `0.5`。因此，方程变为这个：

```js
const percentage = (progress – 0.5) * 200
```

通过这个修改，我们的 `css` 函数现在变成这样：

```js
css: (progress) => {
  if (progress <= 0.5) {
    const percentage = progress * 200;
    return `background: linear-gradient(to right, red 0, ${percentage}%, transparent ${percentage}%);`;
  } else {
    const percentage = (progress – 0.5) * 200;
    return `background: linear-gradient(to right, transparent 0, ${percentage}%, red ${percentage}%);`;
  }
}
```

在这个更新的函数中，实色块和透明区域在过渡过程中根据计算出的百分比动态调整大小，有效地创造出颜色滑动的视觉错觉。

将过渡应用到元素上时，你可能会注意到，尽管我们有一个功能性的实色滑动过渡效果，但有一些元素可以进一步优化以获得更平滑的视觉体验。

一个突出的问题是，元素内的文本在整个过渡过程中都保持可见。理想情况下，它应该在过渡的第一半中保持隐藏，当实色块扩展时，只有在第二半中颜色块收缩时才被揭示。

以下截图说明了这个问题：

图 14.6：在过渡的第一半中，文本没有隐藏

为了解决这个问题，我们可以在过渡的第一半中将文本颜色设置为透明，如下面的代码片段所示：

```js
css: (progress) => {
  if (progress <= 0.5) {
    const percentage = progress * 200;
    return `background: linear-gradient(to right, red 0, ${percentage}%, transparent ${percentage}%); color: transparent;`;
  } else { /* ... */ }
}
```

另一个问题是在文本颜色的情况下，实色块仍然保持红色。因为我们使用 CSS 的 `background` 属性来创建滑动效果，所以文本保持在最前面，而实色块在背景中。

这影响了从颜色块中揭示文本的效果，因为整个文本内容在过渡的第二半部分完全可见。如果实色块与文本颜色相同，文本就会与背景融合。这将产生一种视觉错觉，给人一种文本随着颜色块收缩而被揭示的印象。

下面的截图说明了这个问题：

图 14.7：文本和块颜色不匹配

为了解决这个问题，我们需要找到一种方法来获取文本的颜色并将其融入我们的线性渐变背景中。

`window.getComputedStyle()` 函数允许我们获取应用到元素上的样式。我们可以使用这个函数来获取过渡开始时文本的颜色，并使用这个颜色作为我们的渐变背景：

```js
function colourSwipe(node, params) {
  const { color } = window.getComputedStyle(node);
  const config = {
    css: (progress) => {
      if (progress <= 0.5) {
        const percentage = progress * 200;
        return `background: linear-gradient(to right, ${color} 0, ${percentage}%, transparent ${percentage}%); color: transparent;`;
      } else {
        const percentage = (progress – 0.5) * 200;
        return `background: linear-gradient(to right, transparent 0, ${percentage}%, ${color} ${percentage}%);`;
      }
    }
  };
  return config;
}
```

在前面的修改后的代码片段中，我们将 `red` 替换为从 `node` 元素的计算样式中获取的文本颜色。

由此，我们就得到了一个定制的颜色滑动效果，作为 Svelte 过渡实现。完整的代码可以在 [`github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter14/01-css-transition`](https://github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter14/01-css-transition) 找到。

我们一步一步地通过使用 CSS 创建自定义 Svelte 过渡。在整个过程中，我们学习了如何将用户可定制的属性（如 `duration`、`delay` 和 `easing`）集成到我们的过渡中。在我们的颜色滑动过渡中，我们学习了如何构建多阶段过渡，以及如何将 `progress` 参数分割成各个阶段，利用其值来计算 CSS 规则，并将其应用到元素上。

希望你现在已经准备好使用 CSS 创建自己的自定义 Svelte 过渡了。

在本章的开头，我们了解到过渡合约可以包括不仅是一个 `css` 函数，还可以是一个 `tick` 函数。`tick` 函数允许我们在过渡期间修改元素。我们已经探讨了如何使用 `css` 函数创建颜色滑动过渡；在下一节中，我们将深入探讨创建另一个自定义过渡，这次使用 `tick` 函数。

# 使用 tick 函数编写自定义 JavaScript 过渡

我们在本节中尝试编写的自定义过渡是一个翻页过渡。这个过渡模仿了复古机场出发牌的机制。在这个过渡过程中，文本的每个字母都会翻转，循环遍历字符，直到最终落在正确的字母上。当所有字母都稳定在正确的字母上时，过渡结束。

下面的图示说明了翻页过渡如何揭示短语 *Hello Svelte*，其中垂直轴表示从上到下的时间流动：

图 14.8：翻页过渡可视化

在过渡开始时，字母从左到右开始出现，最初显示为破折号（`-`），然后通过随机字符翻转，最终定位到正确的字母。这种翻转动作从左到右继续，直到所有字母都与其对应的字符对齐，揭示出预期的短语。

整个过渡过程涉及修改元素内的字符，从空白状态过渡到混乱的字符，最后过渡到正确的文本。由于不需要样式或布局更改，我们没有使用`css`函数来实现这个过渡。`tick`函数是 Svelte 中实现这个过渡的完美候选者。

既然我们已经定义了翻页过渡的外观，让我们开始实现这个过渡。

建立在上一节中学习的颜色滑动过渡的基础上，翻页过渡以类似的方式开始。以下是我们的翻页过渡的基本代码结构：

```js
<script>
  function flipboard(node, params) {
    const config = {
      duration: params?.duration ?? 1000,
      delay: params?.delay ?? 0,
      easing: params?.easing ?? linear,
      tick: (progress) => {
        // TODO: implement the transition here
      },
    };
    return config;
  }
</script>
<p transition:flipboard>Hello Svelte.</p>
```

在前面的代码片段中，我们定义了一个`flipboard`函数，它遵循过渡契约。它接受两个参数，`node`和`params`，并返回一个过渡配置——一个描述过渡的对象。因此，我们能够将`flipboard`函数作为过渡应用于`<div>`元素上的`transition:`指令。

在翻页过渡中，我们已经设置了基本参数，例如定义`duration`、`delay`和`easing`值，同时为`tick`函数留出一个占位符，我们将在这里实现过渡。

要创建翻页过渡，我们首先获取将要过渡的元素的文本内容。然后，每次调用`tick`函数时，我们根据`progress`值确定要显示的文本，并相应地更新元素。

我们可以使用以下 API 检索元素的文本：

```js
const text = node.textContent;
```

同样，为了设置元素的文本内容，我们通过相同的属性赋值：

```js
node.textContent = text;
```

将这些整合到翻页过渡中，我们得到以下结果：

```js
function flipboard(node, params) {
  const text = node.textContent;
  const config = {
    // ...
    tick: (progress) => {
      let newText;
      // TODO: compute the newText based on `text` and progress value
      node.textContent = newText;
    },
  };
  return config;
}
```

在前面的代码片段中，我们在`flipboard`函数开始播放元素上的过渡之前，在元素的开头检索文本内容。`tick`函数在每个动画帧上被反复调用，根据原始`text`值和当前的`progress`值计算元素的新的`text`值。

`tick`函数的任务是根据`progress`值确定每个字母应该如何显示。一些字母可能显示为破折号，一些可能显示为随机字符，一些可能显示为它们的原始值，而其他字母可能被隐藏。

对于每个字母，其显示取决于其在整个文本长度中的位置和当前的`progress`值。例如，如果一个字母位于从左边的 30%，而当前的`progress`值是`0.5`（或 50%），那么该字母应该按原样显示。

我们如何确定这些规则？是什么让我们得出刚才的结论？

我们希望元素在过渡结束时显示其所有原始字符。这意味着当`progress`值达到 1（或 100%）时，所有字母都应该显示其原始字符。在中间点，当`progress`值为`0.5`（或 50%）时，左侧的 50%字母应该显示其原始字符，而右侧剩余的 50%应该显示破折号、随机字符或根本不显示。

为了概括，如果一个字母的左侧位置小于`progress`值，它应该显示其原始字符。否则，它可能显示破折号、随机字符或根本不显示。

以下图表说明了`progress`值和显示文本之间的关系：

图 14.9：红色框显示在各个进度值下哪些字母显示原始字符

以下图表说明了在不同`progress`值下原始字符的显示方式。随着`progress`值的增加，单词中的更多字母会显示其原始字符。

为了实现刚才描述的翻页效果，我们将遍历每个字符，确定其相对位置，然后决定是否显示。对于位置超过当前`progress`值的字符，我们将显示空白空间。

这里是更新后的代码：

```js
tick: (progress) => {
  let newText = '';
  for (let i = 0; i < text.length; i++) {
    const position = i / text.length;
    if (position < progress) {
      // display the original character
      newText += text[i];
    } else {
      // display a blank space instead
      newText += ' ';
    }
  }
  node.textContent = newText;
},
```

使用这段代码，翻页过渡现在根据进度值显示原始字符或空白空间。在播放过渡时，你会看到字符一个接一个地从左到右出现。

在确定了何时显示原始字符之后，让我们继续确定文本何时应该显示破折号或随机字符。

使用类似的想法，我们可以确定一个字母是否应该显示随机字符、破折号或根本不显示。我们将引入一个常数来管理这些变化的时机。如果一个字母的位置超过这个常数乘以`progress`值，它将显示为空白。我根据试错选择了 2 这个常数，它大于 1 但不是太大，以创建字符从左到右逐渐出现的效果。

同样，我们也可以引入另一个常数来管理破折号或随机字符的显示。如果一个字母的相对位置大于这个新常数乘以进度值，但小于 2，该字母将显示为破折号。否则，它将显示为随机字符。为此，我选择了 1.5，将其定位在 1 和 2 之间。

以下图表直观地表示了这两个常数及其对过渡的影响：

图 14.10：新文本与原始文本之间的关系

在前面的图中，你可以观察到字符在过渡过程中的变化。例如，当 `progress` 值为 `0.4` 时，位于 40% 位置的字母显示原始字符，位于 40% - 60%（`progress` * 1.5）的字母显示随机字符，位于 60% - 80%（`progress` * 2）的字母显示破折号，而超出这个范围的则不显示。

下面是我们的翻页过渡效果的更新代码：

```js
tick: (progress) => {
  let newText = '';
  for (let i = 0; i < text.length; i++) {
    const position = i / text.length;
    if (position < progress) {
      // display the original character
      newText += text[i];
    } else if (position < progress * 1.5) {
      // display random characters
      newText += randomCharacter()
    } else if (position < progress * 2) {
      // display dash
      newText += '-';
    } else {
      // display a blank space instead
      newText += ' ';
    }
  }
  node.textContent = newText;
},
```

在前面的代码片段中，我添加了两个额外的条件来确定何时显示破折号或随机字符。

`randomCharacter()` 函数返回一个随机选择的字符，实现方式如下：

```js
const chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz1234567890'
function randomCharacter() {
  return chars[Math.floor(Math.random() * chars.length)];
}
```

使用这段代码，你就有了一个翻页过渡！尝试将其应用于一个元素，并观察效果。字符会一个接一个地从左到右缓慢出现，最初是破折号，然后翻转通过字符，最终稳定在正确的字符上。

你可能会注意到一个小问题：不是所有字符的宽度都相同，因此文本的整体宽度会随之增长和缩小。由于每个字母都没有与它之前的位置对齐，翻转效果可能不会立即明显。

为了解决这个问题，你可以使用等宽字体。等宽字体，也称为固定宽度字体，确保每个字母占据相同的水平空间。将等宽字体应用于元素可以增强翻转效果，使其更加视觉上突出。

例如，你可以设置字体如下：

```js
font-family: monospace;
```

在本节中，我们探讨了如何创建翻页过渡，模拟了复古机场出发牌的外观。我们学习了如何根据过渡的进度来控制字符的显示，使用随机字符、破折号和原始文本。通过修改元素的文本，我们创建了一个视觉上吸引人的过渡。

本节的完整代码可以在以下链接找到：[`github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter13`](https://github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter13)。

# 摘要

在本章中，我们探讨了如何在 Svelte 中创建自定义过渡。我们详细介绍了两个示例，这两个示例都使用了 `css` 函数和 `tick` 函数。

希望你现在已经准备好在 Svelte 中编写自己的自定义过渡，从而能够制作出更具吸引力和独特性的用户体验。

在我们接下来的最后一章中，我们将深入探讨过渡如何影响 Svelte 应用的可访问性，指导你创建一个对所有用户都具有吸引力和包容性的体验。
