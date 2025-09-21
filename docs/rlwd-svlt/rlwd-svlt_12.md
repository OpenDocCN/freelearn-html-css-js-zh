

# 第十二章：存储和动画

在本章中，我们将深入探讨 Svelte 动画的世界，重点关注 `tweened` 和 `spring` 存储的强大功能和多功能性。`tweened` 和 `spring` 存储是可写存储，当调用 `set` 或 `update` 方法时，它们的存储值会随时间变化，使我们能够开发更复杂且视觉上吸引人的动画。通过有效地利用这些存储，您可以提升用户体验，并创建既动态又吸引人的应用程序。

我们从深入了解 `tweened` 和 `spring` 存储开始本章，学习如何使用这些存储创建动画。随后，我们探索插值和自定义插值的使用。在整个章节中，我们检查了各种示例，例如动画图表和图像灯箱，以说明概念。到本章结束时，您将掌握有效利用 `tweened` 和 `spring` 存储所需的技能，使您能够在 Svelte 项目中创建复杂且引人入胜的动画。

本章涵盖了以下主题：

+   `tweened` 和 `spring` 存储简介

+   自定义插值及其使用

+   使用 `tweened` 和 `spring` 存储进行动画

# 技术要求

您可以在 GitHub 上找到本章使用的代码：[`github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter12`](https://github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter12)

# 介绍 `tweened` 和 `spring` 存储

让我们通过理解 `tweened` 和 `spring` 存储的概念，开始我们的 Svelte 动画之旅。

`tweened` 和 `spring` 存储是通常包含数字值的可写存储。为了了解它们提供的功能，让我们将它们与一个常规的数字变量进行比较。

如果您不熟悉可写存储，您可以查看*第八章*，在那里我们详细解释了 Svelte 存储，以及如何使用内置的 `writable()` 函数创建可写 Svelte 存储。

通常，当您有一个数字变量并且更新该变量时，变量的值会立即改变。在以下示例中，我们有一个初始值为 `10` 的数字变量 `height`。当我们将该变量的新值设置为 `20` 时，变量的值会立即变为 `20`：

```js
let height = 10;
height = 20;
```

如果我们使用这个数字变量来表示元素的高度或进度条中的进度，一旦赋值，高度或进度就会跳转到新值。这些突然的变化可能会让人感到震惊。

那么，我们如何确保在更新目标值时有一个平滑的过渡？

Svelte 提供了两个内置存储，`tweened` 和 `spring`，专门设计用于存储数字值，并允许在指定的时间内平滑过渡到新值。

让我们通过一个示例来获得更清晰的概念。

在示例中，我们创建了一个初始值为 `10` 的 `tweened` 存储：

```js
import { tweened } from 'svelte/motion';
const height = tweened(10, {
  duration: 1000 /* 1 second */
});
```

然后，我们将存储库的新值设置为`20`：

```js
$height = 20;
```

当完成这些操作后，存储库的值在一秒钟内逐渐从`10`增加到`20`。如果我们将这个存储库值用作元素的宽度，那么当我们将新值分配给存储库时，元素的宽度将平滑地增长或缩小到目标值，从而实现视觉上吸引人且流畅的过渡。

让我们尝试将高度从`tweened`存储库更改为`spring`存储库。请不要担心`spring`函数中的选项，因为我们将它们在下一节中解释：

```js
import { spring } from 'svelte/motion';
const height = spring(10, {
  stiffness: 0.1,
  damping: 0.25
});
```

现在，当你为存储库分配新值时，你会注意到，与使用`tweened`存储库时类似，存储库的值随时间变化到新值，但变化速率不同。

如您所见，`tweened`和`spring`存储库是 Svelte 中的强大功能，使您能够在应用程序中创建平滑的动画和过渡。这些存储库允许进行平滑的值变化。当用作组件状态时，它们允许以更自然和流畅的方式更新状态。

`tweened`存储库和`spring`存储库之间的区别在于，`tweened`存储库提供了一种使用缓动函数在指定的时间内平滑过渡两个值的方法，而`spring`存储库是为基于物理的动画设计的，其中元素表现得就像它们被连接到一个弹簧上。

让我们深入探讨如何使用`tweened`和`spring`存储库。

## 使用`tweened`和`spring`存储库

`tweened`是一个存储库，它使用选择的缓动函数在指定的时间内平滑地过渡到数值。

这是创建一个`tweened`存储库的方法：

```js
import { tweened } from 'svelte/motion';
import { cubicOut } from 'svelte/easing';
const progress = tweened(0, {
  duration: 1000,
  easing: cubicOut,
  delay: 500,
});
```

在前面的代码片段中，我们创建了一个名为`progress`的`tweened`存储库，其初始存储库值为`0`。当你为`progress`存储库设置新值时，`progress`存储库的存储库值在 0.5 秒内保持不变，然后使用`cubicOut`缓动函数在 1 秒内过渡到新值。

`tweened`存储库的函数签名如下：

```js
import { tweened } from 'svelte/motion';
const value = tweened(initialValue, options);
```

`initialValue`是存储库的初始数值。

`options`是一个包含以下属性的对象：

+   `duration`（默认：`400`）：过渡的持续时间（以毫秒为单位）。

+   `easing`（默认：`linear`）：用于过渡的缓动函数。

    Svelte 在`svelte/easing`模块中提供了各种缓动函数，例如`linear`、`quadIn`和`expoOut`。您还可以创建自定义缓动函数。

+   `delay`（默认：`0`）：在过渡开始之前的延迟（以毫秒为单位）。

另一方面，`spring`是一个存储库，它使用基于弹簧的物理模拟平滑地过渡到数值。

以下是创建一个`spring`存储库的方法：

```js
import { spring } from 'svelte/motion';
const position = spring(0, {
  stiffness: 0.2,
  damping: 0.5,
  precision: 0.001,
});
```

在前面的代码片段中，我们创建了一个名为`position`的`spring`存储，其初始存储值为`0`。当您为`position`存储设置新值时，此值将弹向目标值，并在目标值周围振荡一段时间，直到稳定下来。振荡的幅度和持续时间取决于配置的`stiffness`、`damping`和`precision`值。

`spring`存储的函数签名如下：

```js
import { spring } from 'svelte/motion';
const value = spring(initialValue, options);
```

`initialValue`是存储的初始数值。

`options`是一个包含以下属性的对象：

+   `stiffness`（默认：`0.15`）：弹簧的刚度。较高的值会导致更刚性的弹簧，从而引起更快、更有力的转换。

+   `damping`（默认：`0.8`）：弹簧的阻尼系数。较高的值会导致更多的阻尼，使弹簧更快地稳定下来。

+   `precision`（默认：`0.01`）：当弹簧被认为处于静止状态时的阈值。较小的值会导致更精确的模拟，但可能需要更长的时间才能稳定。

## 使用数组和对象与`tweened`和`spring`存储

`tweened`和`spring`存储不仅可以处理单个数值，还可以处理数字数组和具有数值属性的对象。这使得创建涉及多个值的复杂动画变得容易。

当您传递一个数字数组作为初始值时，存储将独立平滑地过渡数组的每个元素。以下是一个使用两个数字数组的示例：

```js
import { tweened } from 'svelte/motion';
const coordinates = tweened([0, 0], { duration: 1000 });
// Updating the coordinates
$coordinates = [100, 200];
```

类似地，当您传递具有数值属性的对象作为初始值时，存储将独立平滑地过渡每个属性。以下是一个使用具有两个数值属性的对象的示例：

```js
import { tweened } from 'svelte/motion';
const position = tweened({ x: 0, y: 0 }, { duration: 1000 });
// Updating the position
$position = { x: 100, y: 200 };
```

当使用数组或对象时，您可以在 Svelte 组件中如下访问和使用单个值：

```js
<script>
  import { tweened } from 'svelte/motion';
  const position = tweened({ x: 0, y: 0 }, { duration: 1000 });
</script>
<div style="transform: translate({$position.x}px, {$position.y}px)"></div>
```

处理数组和对象的能力使得`tweened`和`spring`存储更加灵活和强大，让您能够轻松创建复杂的动画。

现在我们已经了解了如何使用`tweened`和`spring`存储，让我们用它们来创建一个动画图表。

# 示例 - 使用`tweened`和`spring`存储创建动画图表

在本节中，我们将探讨一个示例，展示`tweened`和`spring`存储的强大功能。我们将创建一个动画柱状图，其中柱子会动态调整大小以反映更新的数据值。通过向柱状图添加动画，我们可以有效地突出数据变化，并深入了解复杂的数据集。

首先，让我们创建一个柱状图组件：

```js
<script>
  let data = generateData(10);
  function generateData(length) {
    const result = new Array(length);
    for (let i = 0; i < length; i ++) {
      result[i] = Math.random() * 300;
    }
    return result;
  }
</script>
<style>
  .bar {
    background-color: steelblue;
    height: 50px;
  }
</style>
<div>
  {#each data as value}
    <div class="bar" style="width: {value}px"></div>
  {/each}
</div>
```

在提供的代码片段中，我们使用`generateData`函数初始化`data`变量，该函数使用`length`参数创建一个指定长度的随机生成数据数组。

使用 `data` 数组，我们通过 `{#each}` 块为数组中的每个项目创建一个 `<div>` 元素，并将 `<div>` 元素的宽度设置为数组中相应项目的值。

因此，我们得到了一个显示 10 个条形的水平条形图，它们的宽度是随机生成的。

为了使事情更有趣，我们将以固定间隔更新条形图的值：

```js
import { onMount } from 'svelte';
onMount(() => {
  const intervalId = setInterval(() => {
    data = generateData(10);
  }, 1000);
  return () => clearInterval(intervalId);
});
```

组件挂载后，我们立即使用 `setInterval` 启动一个 1 秒的间隔。在每次间隔中，我们通过使用 `generateData(10)` 重新生成数据来更新数据。

通过添加这段新代码，你会观察到水平条在每次间隔期间改变它们的宽度。水平条的宽度突然调整，产生一种令人不适的视觉效果。

现在，让我们利用 `tweened` 存储在数据更改时使条形图平滑地增长或缩小：

1.  首先，让我们从 `svelte/motion` 中导入 `tweened` 存储。然后，我们将使用 `tweened` 存储包装 `data` 数组。如前一小节所示，我们可以将数字数组传递给 `tweened` 函数以创建一个 `tweened` 存储。当更新时，此存储将独立平滑地过渡数组中的每个数字：

    ```js
    import { tweened } from 'svelte/motion';
    const data = tweened(generateData(10));
    ```

1.  现在，由于 `data` 是一个 Svelte 存储，在更改时我们需要使用 `$data` 变量而不是直接更新其值：

    ```js
    data in the {#each} block, we need to use the $data variable as well:

    ```

    {#each $data as value}

    ```js

    ```

将所有这些更改放在一起，你现在将观察到水平条形图平滑地增长和缩小，大大提高了条形图的视觉吸引力和用户体验。

如你所见，使用 `tweened` 存储创建平滑动画的图表相当简单。在我们进入下一节之前，我们鼓励你亲自尝试：将 `tweened` 函数替换为 `spring` 并观察动画的变化。

## 练习 – 创建一个动态的折线图

现在你已经看到了如何创建动态条形图，现在是时候尝试创建一个动态折线图了。

你可以使用 `d3-shape` 库 ([`github.com/d3/d3-shape`](https://github.com/d3/d3-shape))，它提供了一个方便的 `line` 方法，该方法基于项目数组生成 SVG 路径。以下是如何使用 `line` 方法的示例：

```js
const pathGenerator = line().x((d, i) => i).y((d) => d);
const path = pathGenerator(data);
```

在前面的代码片段中，我们使用 `line` 方法创建了一个 `pathGenerator` 函数，该函数通过将数组的值映射到 *y* 坐标来生成 SVG 路径。你可以通过在 SVG 中使用返回的 SVG 路径和 `<path>` 元素来创建折线图。

一旦你完成了你的实现，请随意将你的结果与我们的示例进行比较，示例在 [`github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter12/02-line-chart`](https://github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter12/02-line-chart)。祝你好运，享受实验你的动态折线图吧！

在熟悉了使用 `tweened` 和 `spring` 存储器处理数字、数组和对象之后，现在是时候将非数值值纳入其中了。这将通过创建自定义缓动插值器来实现，我们将在下一节中探讨。

# 创建自定义缓动插值器

有时，你可能想要在非数值值之间进行转换，例如颜色。幸运的是，这并不妨碍我们使用 `tweened` 存储器。`tweened` 函数提供了一个选项来定义两个值之间的自定义插值。

在本节中，我们将探讨如何创建自定义插值，以在两种颜色之间实现平滑过渡。

但什么是插值？

当在两个值之间进行转换时，插值意味着在两个值之间生成中间值，以创建平滑的过渡。

例如，考虑一个初始化为 `0` 的 `tweened` 存储器，并将其设置为 `100`。`tweened` 存储器生成 `0` 和 `100` 之间的中间值，例如 `20`、`40`、`60` 等等，同时使用这些中间值更新存储器值。因此，在从 `0` 到 `100` 的转换过程中，存储器值平滑地变化，提供了一个从 `0` 到 `100` 的视觉上吸引人的进度。

生成中间值的过程被称为插值。

默认的 `tweened` 存储器能够插值两个数字、两个数字数组以及具有数值属性值的两个对象。然而，它不知道如何插值两个颜色。在这种情况下，我们可以在创建 `tweened` 存储器时传递一个自定义插值函数。

`tweened` 存储器插值的函数签名如下：

```js
function interpolate(a, b) {
  return function (t) {
    // calculate the intermediate value between 'a' and 'b' based on 't'
  };
}
```

在这个函数中，`a` 和 `b` 代表起始值和结束值，而 `t` 是一个介于 `0` 和 `1` 之间的值，表示转换的进度。`interpolate` 函数应该返回另一个函数，该函数根据 `t`（转换的进度）计算并返回中间值。

例如，一个线性插值两个数字的 `interpolate` 函数看起来像这样：

```js
function interpolate(a, b) {
  return function (t) {
    return a + t * (b – a);
  };
}
```

`interpolate` 函数返回另一个函数，该函数接受一个进度值 `t`，并根据 `t` 值在 `a` 和 `b` 之间计算线性插值。当 `t` 为 `0` 时，函数返回 `a`，当 `t` 为 `1` 时，它返回 `b`。对于 `t` 在 0 和 1 之间的值，结果是 `a` 和 `b` 之间的比例值。要为颜色创建插值函数，我们可以将颜色分解成单个 **红色、绿色和蓝色**（**RGB**）分量，并分别对每个分量进行插值。在插值每个分量之后，我们可以将它们重新组合以形成中间颜色。

或者，我们可以使用已经实现了此类插值的库。此类库的一个好例子是 `d3-interpolate` ([`github.com/d3/d3-interpolate`](https://github.com/d3/d3-interpolate))。通过使用经过良好测试的库如 `d3-interpolate`，我们可以节省时间并确保我们的颜色插值既准确又高效。

下面是一个使用 `d3-interpolate` 的示例：

```js
import { interpolateRgb } from 'd3-interpolate';
function interpolate(a, b) {
  const interpolateColor = interpolateRgb(a, b);
  return function (t) {
    return interpolateColor(t);
  };
}
```

在前面的代码片段中，我们从 `d3-interpolate` 导入了 `interpolateRgb` 函数，该函数返回用于颜色 `a` 和 `b` 的插值函数。然后我们创建了一个自定义的插值函数 `interpolateColor`，它返回一个基于进度 `t` 计算中间颜色的函数。

当创建 `tweened` 存储库时，我们可以通过将 `interpolate` 函数作为第二个参数传递来使用我们的自定义 `interpolate` 函数：

```js
tweened(color, { interpolate: interpolate });
```

就这样；现在您可以为颜色创建一个可以平滑过渡的 `tweened` 存储库。

您可以在 GitHub 上找到使用颜色插值的代码示例：[`github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter12/04-interpolation`](https://github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter12/04-interpolation)。

到现在为止，您已经学会了如何使用 `tweened` 存储库创建动画图表，以及如何使用自定义 `interpolate` 函数在非数值之间进行转换。

让我们探索更多使用 `tweened` 和 `spring` 存储库创建流畅用户界面的示例。

# 示例 - 创建动画图像预览

在这个例子中，我们将创建一个图像预览功能，允许用户在点击缩略图时查看更大、更详细的版本，从而增强用户的视觉体验，并使他们能够更仔细地检查图像。

在构建此功能时，您将看到我们如何利用 `spring` 存储库创建更流畅、更自然的用户体验，使图像及其更大预览之间的转换感觉平滑且引人入胜。

首先，让我们创建一个将在页面上显示的图像列表：

```js
<script>
  const images = [
    "path/to/image1.jpg",
    "path/to/image2.jpg",
    // ...more image paths
  ];
  const imgElements = [];
</script>
<div class="image-container">
  {#each images as image, index}
    <div>
      <img src={image} bind:this={imgElements[index]} />
    </div>
  {/each}
</div>
```

在这个例子中，我们创建了一个包含图像文件路径的图像数组。我们使用 `{#each}` 块遍历图像，并为每个图像创建一个包含 `<img>` 元素的 `<div>` 元素。

在前面的代码片段中，省略了 `<style>` 部分，因为它对于理解代码功能不是必需的。如果您想了解样式的外观，可以在 [`github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter12/03-image-preview`](https://github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter12/03-image-preview) 找到它们。

我们将 `<img>` 元素的引用保存在 `imgElements` 变量中。这将在以后很有用。

为了预览图像并关闭预览，我们需要实现两个函数 `openPreview` 和 `closePreview`，以及一个变量 `selectedImageIndex` 来跟踪当前预览的图像：

```js
<script>
  let selectedImageIndex = -1;
  function openPreview(index) {
    selectedImageIndex = index;
  }
  function closePreview() {
    selectedImageIndex = -1;
  }
</script>
...
      <img
        src={image}
        bind:this={imgElements[index]}
        on:click={() => openPreview(index)}
      />
```

在前面的代码片段中，我们将 `selectedImageIndex` 初始化为 `-1`，表示没有选择任何图片。`openPreview` 函数设置 `selectedImageIndex`，而 `closePreview` 函数取消设置。最后，我们添加一个点击事件监听器，以便在点击的图片上调用 `openPreview` 函数。

为了为我们的图片预览创建一个黑色背景，我们添加了一个 `<div>` 元素，该元素仅在图片被选中时具有 `.backdrop` 类。点击背景将关闭预览：

```js
<div
  class:backdrop={selectedImageIndex !== -1}
  on:click={closePreview}
/>
```

为了显示图片预览，我们的目标是强调、放大并在屏幕上居中显示图片。为了实现这一点，我们必须确定放大图片的目标宽度和高度，并计算所需的 *x* 和 *y* 位置以使其居中。

为了简单起见，让我们假设图片的宽高比是 1:1。我们将目标宽度和高度设置为窗口高度和窗口宽度之间较小值的 80%。确定了目标高度和宽度后，我们可以使用这些值来计算在屏幕上居中图片所需的 *x* 和 *y* 位置。让我们看看如何：

```js
<script>
  let selectedImageIndex = -1;
  let width, height, left, top;
  function openPreview(index) {
    selectedImageIndex = index;
    width = Math.min(window.innerWidth, window.innerHeight) * 0.8;
    height = width; // same as width, assuming 1:1 ratio
    left = (window.innerWidth - width) / 2;
    top = (window.innerHeight - height) / 2;
  }
</script>
...
      <img
        src={image}
        bind:this={imgElements[index]}
        on:click={() => openPreview(index)}
        style={selectedImageIndex === index ? `
          position: fixed;
          left: ${left}px;
          top: ${top}px;
          width: ${width}px;
          height: ${height}px;
        ` : ''}
      />
```

在代码片段中，我们设置了选中图片的样式。当图片被选中时，我们使用 `position: fixed` 来定位它，允许我们设置图片相对于视口的 `left` 和 `top` 位置。到此为止，我们就有一个简单的图片预览组件了。

现在，让我们转向一个有趣的问题：我们如何使用 `spring` 存储使预览更加流畅，而不是突然将图片放置在屏幕中心？

一个想法是使用 `transform: translate` 而不是直接设置左和顶位置来居中图片。我们可以保持左和顶位置不变，并使用 `transform: translate` 将图片移动到屏幕中心。平移偏移量的值将来自 `spring` 存储。

使用 `transform: translate` 而不是更新左和顶位置的原因是，它允许更平滑、更高效的动画，因为它不像更新位置属性（如 `left` 和 `top`）那样频繁地触发布局重新计算和重绘。此外，使用 `transform: translate` 使得只需将平移偏移量重置为 `0`，就可以轻松地将图片重置回原始位置。

同样，我们可以将这个想法应用到图片的宽度和高度上。我们可以保持原始图片的尺寸，并使用 `transform: scale` 来调整图片大小。

在有了这个想法之后，让我们来看代码。在这里，我将转换初始化为一个 `spring` 存储：

```js
const transform = spring(
  { translate: { x: 0, y: 0 }, scale: { x: 1, y: 1 } },
  { stiffness: 0.1, damping: 0.25 }
);
```

`transform` 的 `spring` 存储的默认值是一个 `0` 平移偏移量和 `1` 的缩放比例。

在将图片设置为使用 `position: fixed` 后，为了保持图片在原始位置，我们需要获取 `<img>` 元素的当前位置和尺寸，这可以通过 `getBoundingClientRect` 来实现：

```js
function openPreview(index) {
  // ...
  const rect = imgElements[index].getBoundingClientRect();
  left = rect.left;
  top = rect.top;
  width = rect.width;
  height = rect.height;
}
```

我们之前计算用于`left`、`top`、`width`和`height`值的公式，以实现图像居中和放大，将成为我们的目标`left`、`top`、`width`和`height`。它们将被用来计算位移偏移量和缩放：

```js
  const targetWidth = Math.min(window.innerWidth, window.innerHeight) * 0.8;
  const targetHeight = targetWidth;
  const targetLeft = (window.innerWidth - targetWidth) / 2;
  const targetTop = (window.innerHeight - targetHeight) / 2;
  $transform = {
    translate: {
      x: targetLeft - left,
      y: targetTop - top
    },
    scale: {
      x: targetWidth / width,
      y: targetHeight / height
    },
  };
```

位移偏移量是通过目标位置与实际位置之间的差值来计算的，而缩放则是目标宽度和高度与实际宽度和高度之间的比率。

将变换值合并到`<img>`样式中的方法如下：

```js
<img
  style={selectedImageIndex === index ? `
  ...
  transform: translate(${$transform.translate.x}px, ${$transform.translate.y}px) scale(${$transform.scale.x}, ${$transform.scale.y});
  ` : ''}
```

通过这些更改，当点击图像时，图像将平滑地弹回到中心，从而创造一个更加流畅和自然的用户体验。

现在，轮到你自己尝试了。通过使用`spring`存储库创建一个不透明度值，并使用这个值来调整图像预览背景的暗度。这将进一步增强图像预览组件的流畅性和视觉吸引力。

# 摘要

在本章中，我们探讨了 Svelte 的`tweened`和`spring`存储库。

我们探讨了如何使用`tweened`和`spring`存储库来创建平滑的动画和过渡，增强了视觉吸引力和用户体验。通过使用自定义插值函数并将它们应用于非数值，例如颜色，我们扩展了创建动态和引人入胜的用户界面元素的可能性。

在本章中，我们看到了多个`tweened`和`spring`存储库在实际应用中的例子，了解了如何轻松地使用`tweened`和`spring`存储库来创建动画。希望你现在在使用 Svelte 项目中使用`tweened`和`spring`存储库时更加得心应手。

这是本章最后讨论 Svelte 上下文和 Svelte 存储库的部分。在下一章中，我们将探讨过渡，特别是如何在 Svelte 组件中使用过渡。

# 第四部分：过渡

在本节的最后部分，我们将深入探讨 Svelte 的过渡。我们将首先了解如何将内置过渡集成到我们的 Svelte 组件中。随后，我们将指导你创建自己的自定义过渡。为了结束本节，我们将强调可访问性以及如何通过过渡创建一个满足所有用户需求的应用程序。

本部分包含以下章节：

+   *第十三章**，使用过渡*

+   *第十四章**，探索自定义过渡*

+   *第十五章**，过渡中的可访问性*
