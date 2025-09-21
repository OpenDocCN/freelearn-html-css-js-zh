# 13

# 使用过渡效果

过渡效果对于创建平滑且引人入胜的用户体验至关重要。通过定义元素在用户界面中的出现、消失或变化方式，过渡效果可以将普通的交互转变为令人难忘的体验，给用户留下深刻印象。

在接下来的三章中，我们将探讨 Svelte 中的过渡效果主题，首先从全面了解如何在 Svelte 中使用过渡效果开始。

在本章中，我们将首先学习如何在 Svelte 中为元素添加过渡效果。我们将探讨不同的过渡指令，并学习如何自定义过渡效果。

之后，我们将讨论何时以及如何播放过渡效果。我们将探讨不同的场景，例如元素中包含和不含过渡效果的混合，或者当元素位于嵌套的逻辑块中时。

要真正掌握过渡效果，了解 Svelte 过渡系统的内部工作原理至关重要。我们将通过检查底层机制并提供有助于你在项目中优化过渡效果使用的见解来结束本章。

到本章结束时，你将拥有 Svelte 过渡效果的坚实基础，让你能够轻松创建引人入胜且动态的用户界面。

本章包括以下内容：

+   如何为元素添加过渡效果

+   元素的过渡效果何时播放

+   过渡效果在底层是如何工作的

# 技术要求

你可以在 GitHub 上找到本章使用的代码：[`github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter13`](https://github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter13)。

# 为元素添加过渡效果

Svelte 提供了一种简单而强大的方法来为你的应用元素添加过渡效果。该框架提供了内置的过渡函数，可以轻松应用于元素，从而实现平滑的动画和无缝的用户体验。你还可以定义自己的自定义过渡效果，我们将在下一章中学习。

Svelte 中的过渡效果是在元素挂载或从 DOM 卸载时应用的。这确保了元素的出现和消失都是优雅的，而不是突然出现在视野中或消失。

要在 Svelte 中为元素添加过渡效果，你可以使用`transition:`指令与所需的过渡函数。以下是一个为元素添加过渡效果的示例：

```js
<script>
  import { fade } from 'svelte/transition';
</script>
<div transition:fade>some text here</div>
```

在前面的代码片段中，我们从`svelte/transition`中导入了`fade`并将其应用于`<div>`元素。

你会看到，在前面代码中，当`<div>`元素挂载到 DOM 上时，`<div>`元素将平滑地淡入。当`<div>`元素从 DOM 卸载时，`<div>`元素将平滑地淡出。

`transition:` 指令设置元素挂载到 DOM 和从 DOM 卸载时播放的转换。如果你想更精细地控制元素挂载或卸载时播放的转换，可以使用 `in:` 和 `out:` 指令：

```js
<script>
  import { fade, blur } from 'svelte/transition';
</script>
<div in:fade out:blur>some text here</div>
```

在前面的代码片段中，我们应用了 `fade` 作为进入转换，`blur` 作为退出转换。当 `<div>` 元素挂载到 DOM 上时，`<div>` 元素将平滑地淡入。当 `<div>` 元素从 DOM 中卸载时，`<div>` 元素将平滑地模糊退出。

因此，`transition:` 指令本质上是对 `in:` 和 `out:` 转换的简写。换句话说，以下片段中应用于两个元素的转换在功能上是相同的：

```js
<div transition:blur>some text here</div>
```

因此，前面的代码片段类似于以下代码片段：

```js
<div in:blur out:blur>some text here</div>
```

从前面的示例中，我们已经看到了 Svelte 的两个内置转换，`fade` 和 `blur` - 让我们看看更多！

## Svelte 的内置转换

Svelte 的内置转换是从 `svelte/transition` 模块导出的。

以下列表提供了 Svelte 内置转换的概述：

+   `fade`: 这个转换平滑地使元素淡入或淡出，随时间调整其不透明度

+   `blur`: `blur` 转换逐渐在元素上应用或移除模糊效果

+   `slide`: `slide` 转换使元素平滑地进入或退出视图

+   `fly`: `fly` 转换使元素平滑地从指定的 `x` 和 `y` 偏移量平移

+   `scale`: 这个转换使元素在出现或消失时大小增长或缩小

+   `draw`: `draw` 转换在 SVG 路径上创建绘制或擦除效果

在浏览内置转换列表时，你可能注意到其中一些转换依赖于用户指定的值。例如，`fly` 转换依赖于元素在转换进入时应该飞出的指定 `x` 和 `y` 偏移量。

## 自定义转换

要使用这些转换及其所需的值，你可以传递一个包含必要属性的配置对象到转换指令中：

```js
<script>
  import { fly } from 'svelte/transition';
</script>
<div transition:fly={{ x: 200, y: 100 }}>Content goes here</div>
```

在前面的代码片段中，我们应用了带有指定 `x` 和 `y` 偏移量的 `fly` 转换，这表示元素将从右侧 200 像素和下方 100 像素的位置飞入。通过提供适当的值，你可以在你的 Svelte 组件中实现一系列定制的转换效果。

当你想要元素在转换退出时飞到不同的位置，而不是转换进入时飞入的位置时，这种方法尤其有用。

你可以不使用 `transition:` 指令，并为进入和退出转换只提供一个配置，而是将其分开为 `in:` 和 `out:` 指令，并将不同的配置对象传递给每个指令。

以下代码片段展示了这一点的示例：

```js
<script>
  import { fly } from 'svelte/transition';
</script>
<div in:fly={{ x: 200, y: 100 }} out:fly={{ x: -200, y: 50 }}>Content goes here</div>
```

`<div>` 元素从右侧 200 像素和下方 100 像素飞入，同时从左侧 200 像素和下方 50 像素飞出。通过将 `transition:` 指令分为 `in:` 和 `out:` 指令，您可以使用不同的配置对象控制进入和退出转换，在 Svelte 组件中实现更复杂的转换效果。

除了每个转换的特定自定义配置外，Svelte 的所有内置转换都接受 `delay`、`duration` 和 `easing` 作为转换配置的一部分。这些参数允许您控制动画的时间，为设计用户界面提供更大的灵活性。

`delay` 参数确定转换开始前的等待时间，而 `duration` 参数指定转换持续的时间。通过修改这些值，您可以协调转换的开始时间和每个转换的持续时间，创建更复杂和吸引人的动画。

这是一个调整 `fade` 转换的 `delay` 和 `duration` 值的示例：

```js
<script>
  import { fade } from 'svelte/transition';
</script>
<div transition:fade=fade transition will start with a 500 ms delay after it is mounted onto the DOM, and the transition will last for 1000 ms.
On the other hand, `easing` is a function that is responsible for controlling the pacing of the animation. By adjusting the `easing` function, you can create animations that start slow and end fast, start fast and end slow, or follow a custom pattern, giving you even more control over the look and feel of your animations.
Svelte comes with a collection of built-in `easing` functions, which can be imported from `svelte/easing`. These `easing` functions can then be applied to transitions, as seen in the following code:

```

<script>

import { fade } from 'svelte/transition';

import { quadInOut } from 'svelte/easing';

</script>

<div transition:fade=fade transition 使用了 quadInOut 缓动函数，这使得动画开始缓慢，加速，然后缓慢结束。通过将不同的缓动函数融入您的转换中，您可以为您应用程序创建各种动画。

练习 - 发现 Svelte 的内置转换

作为练习，尝试访问官方 Svelte 文档，并识别每个内置转换的可配置属性列表。

为了让您开始，这里有一份 Svelte 内置转换的列表：

+   `fade`

+   `blur`

+   `fly`

+   `slide`

+   `scale`

+   `draw`

我们知道转换在元素挂载或从 DOM 中卸载时播放，但转换究竟何时以及如何播放？

让我们在下一节中探讨转换播放的时间和方式。

转换何时播放？

Svelte 中的转换在元素被添加或从 DOM 中移除时播放。

`in:` 转换在元素被添加到 DOM 时执行。这通常发生在组件初始化时或当控制元素渲染的条件变为 `true` 时。

例如，在一个 `{#if}` 块中，当 `if` 条件从假变为真时，该 `{#if}` 块内的元素会被添加到 DOM 中。所有应用于这些元素的 `in:` 转换将在元素被插入 DOM 后立即 **同时播放**：

```js
{#if condition}
  <div in:fade>some content</div>
  <div transition:blur>more content</div>
{/if}
```

在前面的代码片段中，当 `condition` 变为 `true` 时，两个 `<div>` 元素将被插入到 DOM 中。一旦两个 `<div>` 元素都被插入，`fade` 和 `blur` 转换将立即同时开始播放。`fade` 和 `blur` 转换是否同时结束取决于每个转换指定的持续时间。

相反，当元素从 DOM 中移除时，会执行`out:`过渡。这可以发生在组件被销毁时，或者当控制元素渲染的条件变为`false`时。

过渡在元素被安排从 DOM 中移除时立即开始。过渡完成后，元素将从 DOM 中移除。

让我们用一个例子来说明这一点：

```js
{#if condition}
  <div out:fade>some content</div>
  <div transition:blur>more content</div>
{/if}
```

在前面的代码片段中，当`condition`变为`false`时，两个`<div>`元素仍然保留在 DOM 中，尽管条件不再为真。这是因为需要在从 DOM 中移除这两个`<div>`元素之前，对它们执行`out:`过渡。如果立即从 DOM 中移除`<div>`元素，它们将不再对用户可见，使得任何后续的`out:`过渡无效且不可见。

`fade`和`blur`过渡将同时应用于两个`<div>`元素作为`out:`过渡。与`in:`过渡类似，每个过渡的持续时间取决于每个过渡指定的持续时间。

一旦所有`out:`过渡都播放完毕，两个`<div>`元素将一起从 DOM 中移除，使 DOM 状态与`condition`的更新值保持一致。

在之前解释何时播放`in:`和`out:`过渡的例子中，`{#if}`块内的所有元素都应用了过渡，导致`{#if}`块中的所有元素同时播放过渡。但是，如果`{#if}`块内不是所有元素都应用了过渡会发生什么？让我们接下来讨论这个问题。

处理混合过渡和静态元素

当一个`{#if}`块内的某些元素应用了过渡而其他没有时，Svelte 会根据指定的过渡对每个元素进行不同的处理。

让我们考虑一个例子：

```js
{#if condition}
  <div in:fade>Element with fade transition</div>
  <div>Static element without transition</div>
  <div transition:slide>Element with slide transition</div>
{/if}
```

在这个例子中，当`condition`变为`true`时，应用了过渡的元素将随着它们被插入 DOM 而动画化，而没有任何过渡的静态元素将简单地出现，没有任何动画。

根据前面的代码片段，第二个`<div>`元素将立即插入并显示在 DOM 上，因为第一个和第三个`<div>`元素分别淡入和滑动进入。

同样，当`condition`变为`false`时，具有`out``:`过渡的元素（在这种情况下，只有第三个`<div>`元素，因为`transition:`指令暗示了`in:`和`out:`过渡）将播放它们各自的退出过渡。

根据前面的代码片段，你会看到第一个和第二个`<div>`元素保持不变，并且第三个`<div>`元素上播放了幻灯片过渡。在`{#if}`块内的所有元素只有在所有`out:`过渡完成后才会一起从 DOM 中移除。

总结来说，当你在同一个逻辑块内混合具有和没有过渡效果的元素时，所有元素将同时被添加到和从 DOM 中移除。Svelte 只对应用了过渡效果的元素进行动画处理，而没有过渡效果的静态元素将无动画地被插入或移除。

到目前为止，我们只看到了使用`{#if}`块作为添加或删除元素手段的例子，但 Svelte 中还有其他逻辑块也可以使用。

让我们看看它们是什么。

其他 Svelte 逻辑块用于过渡

`{#if}`块根据`if`条件添加或删除元素。除了`{#if}`块之外，Svelte 中还有其他逻辑块提供了在添加或删除元素时应用过渡的机会，例如`{#each}`、`{#await}`和`{#key}`。这些块也可以对它们包含的元素应用过渡，为你的用户界面动画提供了广泛的可能性。

例如，`{#each}`块用于遍历项目列表并为每个项目渲染元素。你可以以与`{#if}`块类似的方式在`{#each}`块内对元素应用过渡。让我们看看一个例子：

```js
{#each items as item (item.id)}
  <div in:fade out:slide>{item.name}</div>
{/each}
```

在这个例子中，当向`items`数组中添加或删除新项目时，`{#each}`块内的元素将执行各自的`in:`和`out:`过渡。当`items`数组中有新项目时，新的`<div>`元素将淡入列表的末尾。当从`items`数组中删除元素时，相应的`<div>`元素将从列表中滑动出去。在列表中使用过渡可以创建动态且引人入胜的用户体验，当向列表中添加或删除项目时，提供清晰的视觉提示。

类似地，你可以使用`{#await}`和`{#key}`块与过渡结合，在各种场景中管理元素的添加和删除，同时创建视觉上吸引人的动画。

`transition:`, `in:`, 和 `out:` 指令可以应用于任何元素，并且同一个逻辑块内的元素将同时被添加或移除。这同样适用于嵌套的逻辑块。

例如，让我们考虑以下代码片段：

```js
{#if condition}
  <p transition:blur>paragraph 1</p>
  {#each items as item (item.id)}
    <div transition:fade>{item.name}</div>
  {/each}
  <p>paragraph 2</p>
{/if}
```

当`condition`从`false`变为`true`时，以下情况会发生：

+   第一个具有`blur`过渡效果的`<p>`元素将随着其被插入到 DOM 中进行动画处理

+   同时，对于`items`数组中的每个项目，当它们被插入到 DOM 中时，具有`fade`过渡效果的`<div>`元素将进行动画处理

+   最后一个没有过渡效果的`<p>`元素将简单地出现在 DOM 中，没有动画

相反，当`condition`从`true`变为`false`时，以下情况会发生：

+   第一个具有`blur`过渡效果的`<p>`元素将进行动画

+   同时，对于`items`数组中的每个项目，具有`fade`过渡效果的`<div>`元素将进行动画处理

+   最后一个没有过渡的`<p>`元素将保持不变

+   一旦`<p>`元素中所有过渡和`{#each}`块中所有`<div>`元素中的过渡都已完成，`<p>`和`<div>`元素将一起从 DOM 中删除

通过结合使用过渡和嵌套逻辑块，你可以创建复杂的动画，从而提升用户体验。

默认情况下，过渡只有在最近的逻辑块导致元素添加或删除时才会播放。然而，我们可以通过`global`修饰符来改变这种行为。

全局修饰符

只有当最近的逻辑块导致元素添加或删除时才会播放过渡，这有助于限制同时动画的数量，使用户体验更加专注且不那么令人不知所措。这被称为*本地*模式；也就是说，过渡仅应用于本地更改。

要改变这种行为，我们可以应用`global`修饰符。当`global`修饰符应用于`transition:`, `in:`, 和 `out:`指令时，确保动画在元素被添加或删除时播放。

要应用`global`修饰符，只需在指令后缀加上`|global`，如下所示：

```js
{#if condition}
  <div in:fade|global>some text here</div>
{/if}
```

根据前面的例子，在应用`global`修饰符之前，`fade`动画只有在最近的逻辑块，即`{#if}`块触发`<div>`元素的插入或删除时才会播放。这意味着如果另一个父逻辑块导致了元素的添加或删除，动画将不会播放。有了`global`修饰符，过渡将在`<div>`元素被添加或删除时播放，无论哪个逻辑块导致它。

为了进一步阐述，让我们看看以下嵌套的`{#if}`块示例：

```js
{#if condition1}
  <div transition:fade>first div</div>
  {#if condition2}
    <div transition:fade>second div</div>
    <div transition:fade|global>third div</div>
  {/if}
{/if}
```

让我们从`condition1`为`false`和`condition2`为`true`开始。

当`condition1`变为`true`时，三个`<div>`元素将一起插入到 DOM 中。由于`condition2`始终为`true`，此时，导致所有`<div>`元素插入的`{#if}`块是带有`condition1`的那个。

第一个`<div>`元素将淡入，因为其最近的逻辑块，即`{#if condition1}`，负责插入`<div>`元素。

第二个`<div>`元素将立即在屏幕上可见，而不会播放`fade`过渡。这是因为，默认情况下，过渡处于*本地*模式，其最近的逻辑块，即`{#if condition2}`，不是导致在此点插入`<div>`元素的原因。

第三个`<div>`元素将与第一个`<div>`元素同时淡入。因为`<div>`元素对其过渡应用了`|global`修饰符，所以负责其插入的逻辑块无关紧要。过渡将播放，无论哪个具体的逻辑块导致`<div>`元素被插入。

现在假设`condition1`从`true`变为`false`呢？

同样的逻辑适用；因此，第二个`<div>`元素将保持不变，只有第一个和第三个`<div>`元素会渐隐。一旦渐隐过渡完成，所有三个`<div>`元素都将从 DOM 中移除。

在我们已走过的场景中，第二个`<div>`元素的`fade`过渡尚未播放。那么，第二个`<div>`元素的`fade`过渡将在何时播放？

要理解第二个`<div>`元素的`fade`过渡将在何时播放，让我们考虑`condition1`保持`true`且`condition2`从`false`变为`true`的情况。

当`condition1`为`true`且`condition2`从`false`变为`true`时，第二个`<div>`元素将被插入到 DOM 中。由于它最近的逻辑块`{#if condition2}`现在负责插入，`transition:fade`过渡将被播放。

如您所见，使用`global`修饰符，我们可以改变过渡的播放时机以响应变化。我们不仅可以只在元素受特定条件影响时播放过渡，还可以将其改为始终播放。

Svelte 3 和 Svelte 4 之间的区别

如我们之前所解释的，Svelte 过渡默认是*本地*模式。然而，这仅在 Svelte 4 中有所改变。在 Svelte 3 中，情况正好相反。在 Svelte 3 中，过渡默认是*全局*模式，你需要将`local`修饰符应用到过渡上才能将其改为*本地*模式。

到目前为止，我们已经介绍了如何使用`transition:`, `in:`, 和 `out:`指令给元素添加过渡，我们学习了何时以及如何播放过渡。在我们结束这一章之前，让我们深入了解 Svelte 中过渡的内部工作原理，以便更好地理解其机制。

Svelte 过渡的内部工作原理

在我们深入探讨 Svelte 过渡的内部工作原理之前，让我们首先简要讨论在网页上创建动画的一般方法。理解这些基本概念为掌握 Svelte 过渡的工作原理提供了坚实的基础。

通常，你可以使用 CSS 或 JavaScript 来创建动画。

使用 CSS 创建动画

要使用 CSS 创建动画，你可以使用 CSS 的`animation`属性以及`@keyframes`规则。`@keyframes`规则用于定义一系列样式，指定动画过程中每个关键帧（从 0%到 100%）的 CSS 样式。

例如，看看这个：

```js
@keyframes example {
  0% {
    opacity: 1;
    transform: scale(1);
  }
  100% {
    opacity: 0;
    transform: scale(1.75);
  }
}
```

在前面的代码片段中，我们定义了一个名为`example`的动画关键帧，它同时将不透明度从 100%变为 0%，并将缩放从 1 变为 1.75。

要将`example`动画应用到元素上，我们使用 CSS 的`animation`属性：

```js
<div style="animation: example 4s 1s 1;">Animated element</div>
```

在前面的代码片段中，我们将动画设置为名为`example`的动画关键帧，持续时间为四秒，延迟为一秒，并且只播放动画一次。

`@keyframes` 规则非常灵活。我们可以通过 `@keyframes` 声明对动画序列的中间步骤进行精细控制。结合 `animation` 属性，我们可以控制动画的外观，以及动画何时开始和持续多长时间。

使用 CSS 创建动画的优点是它不涉及 JavaScript，浏览器可以自行优化 CSS 动画。这节省了 JavaScript 带宽，因此即使有密集的 JavaScript 任务同时运行，动画也可以平滑运行。这确保了更好的性能和用户体验，因为即使在重处理负载下，动画也能保持响应和流畅。

使用 JavaScript 创建动画

使用 JavaScript 创建动画涉及动态操作 DOM 元素的样式和属性。

例如，让我们用 JavaScript 编写一个淡入动画。

要实现这一点，我们需要逐渐将元素的透明度从 `0` 变更到 `1`。要在 JavaScript 中将 `<div>` 元素的透明度设置为 `0`，我们直接通过元素的 `style.opacity` 属性来设置：

```js
div.style.opacity = 0;
```

在前面的代码片段中，我们假设我们已经获得了对 `<div>` 元素的引用，并将其存储在名为 `div` 的变量中。然后，我们通过 `div` 变量将 `<div>` 元素的透明度设置为 `0`。

要将元素的透明度从一个值动画过渡到另一个值，您需要在指定的时间间隔内定期更新样式。

与通过 `setInterval` 设置固定间隔不同，更新样式的间隔通常是通过使用 `requestAnimationFrame` 方法来实现的。`requestAnimationFrame` 是一个浏览器方法，它通过在下次重绘之前调用指定的函数来优化动画性能。`requestAnimationFrame` 通过允许浏览器确定最佳时间来更新样式，避免了不必要的操作或重复的重绘，从而帮助确保动画运行得既平滑又高效。

这里是一个使用 `requestAnimationFrame` 创建动画的例子：

```js
let start;
const duration = 4000; // 4 seconds
function loop(timestamp) {
  if (!start) start = timestamp;
  // get the progress in percentage
  const progress = (timestamp – start) / duration;
  // Update the DOM element's styles based on progress
  if (progress > 1) {
    div.style.opacity = 0;
    div.style.transform = 'scale(1.75)';
  } else {
    div.style.opacity = 1 – progress;
    const scale = 1 + progress * 0.75;
    div.style.transform = `scale(${scale})`;
    // continue animating, schedule the next loop
    requestAnimationFrame(loop);
  }
}
// Start the animation
requestAnimationFrame(loop);
```

在前面的代码片段中，我们安排在下一个动画帧中调用 `loop` 函数，直到进度完成。我们计算 `progress` 为动画总持续时间内经过的时间百分比。有了 `progress` 的值，我们计算 `<div>` 元素的透明度和缩放比例。

在这个例子中，使用 JavaScript 和 `requestAnimationFrame` 创建动画的最终结果与上一节中使用 CSS 动画的例子所达到的最终结果相同。

`<div>` 元素在动画开始时的透明度为 `1`，缩放为 `1`，在动画结束时透明度为 `0`，缩放为 `1.75`。

使用 JavaScript 进行动画提供了对动画逻辑的更多控制，使您能够创建复杂的交互式动画，这些动画可以响应用户输入或其他事件。

然而，使用 JavaScript 进行动画的一个缺点是，随着动画的依赖，它可能需要更多的资源，因为动画依赖于浏览器的 JavaScript 引擎来处理和执行动画逻辑。

现在我们已经了解了在网络上创建动画的两种不同方法，Svelte 过渡使用的是哪一种？

答案是两者都是。

在 Svelte 中动画化过渡

尽管 Svelte 的所有内置过渡都使用 CSS 进行动画处理，但 Svelte 能够使用 CSS 和 JavaScript 两种方式来动画化过渡。

要通过 CSS 动画化过渡，Svelte 将为每个元素生成一个一次性的`@keyframes`规则，基于过渡和指定的配置对象。

让我们以`fly`过渡为例：

```js
<script>
  import { fly } from 'svelte/transition';
</script>
<div in:fly={{ x: 50, y: 30 }}>Some text here</div>
```

在前面的代码片段中，`fly`过渡被应用于一个`<div>`元素。作为回应，Svelte 生成一个类似下面的`@keyframes`规则：

```js
@keyframes fly-in-unique-id {
  0% {
    transform: translate(50px, 30px);
    opacity: 0;
  }
  100% {
    transform: translate(0, 0);
    opacity: 1;
  }
}
```

生成的`@keyframes`规则将在过渡期间应用于元素。关键帧名称中的`unique-id`部分确保每个生成的关键帧都是唯一的，并且不会与其他元素冲突。

根据过渡的指定`duration`和`delay`，Svelte 将计算动画的适当时间，并将生成的`@keyframes`规则应用于元素，使用 CSS 的`animation`属性。然后，元素将根据指定的`transition`、`duration`和`delay`进行动画化。

例如，在下面的代码片段中，我们有一个`fly`过渡被应用于一个持续时间为 500 毫秒、延迟时间为 200 毫秒的`<div>`元素：

```js
<script>
  import { fly } from 'svelte/transition';
</script>
<div in:fly={{ x: 50, y: 30, duration: 500, delay: 200 }}>Some text here</div>
```

要在前面代码片段中动画化`fly`过渡，Svelte 将生成相应的关键帧动画，并将生成的关键帧动画应用于具有指定持续时间和延迟的元素：

```js
div.style.animation = 'fly-in-unique-id 500ms 200ms 1';
```

同样，也可以使用 JavaScript 来动画化过渡。

Svelte 将通过`requestAnimationFrame`调度一个循环，在整个指定持续期间运行动画。

我们现在不会深入探讨`requestAnimationFrame`循环与动画的具体工作原理。在下一章中，我们将探讨如何使用 JavaScript 创建自定义过渡，这将更深入地了解`requestAnimationFrame`循环如何与动画交互，以及如何有效地利用它来实现平滑、引人入胜的过渡。敬请期待，了解更多关于如何使用 Svelte 制作独特动画的知识。

摘要

在本章中，我们学习了如何将过渡添加到元素上。我们探讨了`transition:`, `in:`, 和 `out:`指令，以及如何自定义它们。

随后，我们探讨了何时以及如何播放过渡。我们讨论了当元素既有过渡又有无过渡时，过渡是如何播放的，以及当过渡用于嵌套逻辑块内的元素时，过渡是如何播放的。

最后但同样重要的是，我们深入探讨了 Svelte 如何播放过渡动画。

借助这些知识，你现在可以自信地将过渡应用到 Svelte 中的元素上。这将使你能够增强应用程序的交互性和视觉吸引力，从而提供更吸引人的用户体验。

在下一章中，我们将超越内置的过渡，并探讨自定义过渡的创建。

```js

```
