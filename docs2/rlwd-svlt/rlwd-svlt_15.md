# 15

# 使用过渡实现无障碍

在过去两章中，我们学习了如何在 Svelte 中使用过渡。当正确使用时，过渡可以增强用户体验，引导用户的注意力，提供反馈，并为界面增添一层光泽。然而，对于患有前庭功能障碍的用户来说，这些动画可能会感到不适，甚至可能造成伤害。因此，在创造引人入胜的动画和确保它们不会对有特定需求的用户产生负面影响之间取得平衡是至关重要的。

在本章中，我们将深入探讨使网络过渡对患有前庭功能障碍的用户更加无障碍的技术，探索尊重用户对运动偏好的 CSS 和 JavaScript 方法。

到本章结束时，你将更好地理解网络无障碍以及如何创建更包容的 Web 应用程序，以满足所有用户的具体需求或偏好。

本章将涵盖以下内容：

+   什么是网络无障碍？

+   使用`prefers-reduced-motion`理解用户偏好

+   减少 Svelte 过渡的运动

+   为无法访问的用户提供替代过渡

让我们从探讨什么是网络无障碍开始。

# 技术要求

本章中的所有代码都可以在以下链接找到：[`github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter15/01-accessible-transition`](https://github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter15/01-accessible-transition)

# 什么是网络无障碍？

无障碍是指产品、设备、服务或环境的设计，以便尽可能多的人可以使用，无论他们可能有什么物理、感官或认知障碍。

确保网站对所有用户都无障碍至关重要。有许多残疾可能会影响用户在网站上的体验。确保网站的无障碍性使得所有人，无论他们的能力如何，都能平等地访问到其他人可用的相同服务和信息。

许多可能妨碍网站用户体验的残疾中，前庭功能障碍是其中之一。在本章中，我们将特别关注提高患有前庭功能障碍的人士的无障碍性。

前庭功能障碍是影响内耳和大脑的疾病，它们可能导致平衡、空间定位和运动感知困难。想象一下，你身体的自然平衡感不正常。这就像头晕或感觉不稳。你脚下的地面感觉不稳定，你周围看到的东西似乎在移动，即使你站着不动。

对于患有前庭功能障碍的人来说，某些视觉刺激，如网页上移动或闪烁的内容，可能会引发头晕、恶心或偏头痛等症状。

我们在 *第十三章* 和 *第十四章* 中学习了如何添加过渡，以使我们的应用程序对用户更具吸引力。然而，对于有前庭功能障碍的用户，这些过渡可能会无意中提供负面的体验。大多数操作系统提供可访问性设置，使用户能够减少或删除动画。这些可访问性偏好可以被网络应用程序用来创建包容性的用户体验。

因此，让我们探索一个网络应用程序如何访问用户的可访问性偏好。

# 使用 `prefers-reduced-motion` 理解用户偏好

大多数操作系统都提供可访问性设置，允许用户禁用动画效果。例如，在 Windows 11 中，您可以导航到 **设置** | **可访问性** | **视觉效果** | **动画效果**，取消选中 **动画效果** 选项以关闭动画。

图 15.1：Windows 11 中的动画效果选项

在网络应用程序中，您可以使用 `prefers-reduced-motion` CSS 媒体查询来确定用户是否在他们的设备上激活了减少或消除非必要运动的设置。

以下是如何使用 `prefers-reduced-motion` CSS 媒体查询的示例：

```js
@media (prefers-reduced-motion: reduce) {
  div {
    /* Removes animation */
    animation: none;
  }
}
```

在前面的代码片段中，如果用户表示偏好减少运动，我们将 CSS `animation` 属性设置为 `none`，以从 `<div>` 元素中移除动画。

或者，除了使用 CSS 之外，您还可以使用 JavaScript 来确定用户对减少运动的偏好。`window.matchMedia` 方法允许您检查网页是否匹配给定的媒体查询字符串。

在下面的代码片段中，我们使用 `window.matchMedia` 来测试 `prefers-reduced-motion` CSS 媒体查询是否匹配：

```js
const mediaQuery = window.matchMedia('(prefers-reduced-motion: reduce)');
const matches = mediaQuery.matches;
```

如果用户偏好减少运动，则前面代码片段中 `matches` 的值将为 `true`；否则，`matches` 的值将为 `false`。

用户在浏览网页时可能会更改他们的可访问性偏好。为了在用户更改对减少运动的偏好时得到通知，我们可以监听媒体查询的 `change` 事件。以下是方法：

```js
const mediaQuery = window.matchMedia('(prefers-reduced-motion: reduce)');
mediaQuery.addEventListener('change', () => {
  let matches = mediaQuery.matches;
});
```

在前面的代码片段中，每当用户更改他们对减少运动的偏好时，传递给 `change` 事件处理器的监听函数将被调用。它将通过 `mediaQuery.matches` 评估更新的用户偏好。

既然我们已经学会了如何通过 `prefers-reduced-motion` 确定用户对减少运动的偏好，让我们看看我们如何可以使用它来减少有前庭功能障碍用户的 Svelte 过渡。

# 减少 Svelte 过渡中的运动

在学习了如何获取用户对减少运动的偏好之后，现在让我们通过减少过渡中的不必要的动作来尊重这一偏好，这可能会引发前庭不适。

在下面的代码块中，有一个我们 Svelte 组件的示例，该组件将`fly`过渡应用于列表项：

```js
<script>
  import { fly } from 'svelte/transition';
  export let list = [];
</script>
<ul>
  {#each list as item}
    <li transition:fly={{ x: 40 }}>{item}</li>
  {/each}
</ul>
```

在前面的代码中，每当向列表中添加新项目时，一个新的`<li>`元素将从右侧飞入并插入到列表中。这种飞行运动可能成为前庭功能障碍用户的触发因素。

然而，飞行过渡并非必需，因为即使没有它，应用程序仍然可以正常工作。因此，如果用户在系统设置中表明了偏好减少运动，我们应该通过减少或移除飞行过渡来尊重这一偏好。

实现这一点的办法是将`fly`过渡的持续时间设置为`0`。这样，过渡将不会花费任何时间来播放和完成，实际上将不会播放。

这是之前 Svelte 组件的修改版本，如果用户偏好减少运动，则不会播放`fly`过渡：

```js
<script>
  import { fly } from 'svelte/transition';
  export let list = [];
  const mediaQuery = window.matchMedia('(prefers-reduced-motion: reduce)');
  let prefersReducedMotion = mediaQuery.matches;
  mediaQuery.addEventListener('change', () => {
    prefersReducedMotion = mediaQuery.matches;
  });
</script>
<ul>
  {#each list as item}
    <li transition:fly={{
      x: 40,
      duration: prefersReducedMotion ? 0 : 400,
    }}>{item}</li>
  {/each}
</ul>
```

在前面的代码片段中，我们通过检查 CSS 媒体查询`prefers-reduced-motion: reduce`是否匹配来确定用户是否偏好减少运动，并将此信息存储在名为`prefersReducedMotion`的变量中。如果`prefersReducedMotion`为`true`（表示用户偏好减少运动），则我们将`fly`过渡的持续时间设置为`0`。当向列表中添加新项目时，用户将看不到任何飞行运动。

另一方面，如果用户没有前庭功能障碍，并且没有表达对减少运动效果的偏好，那么`prefersReducedMotion`将为`false`。在这种情况下，`fly`过渡的持续时间将被设置为`400 ms`，并且每当向列表中添加新项目时，都会显示飞行过渡。

然而，并非所有过渡都会触发前庭运动障碍。例如，`fade`过渡作为一种更微妙的动画，对前庭功能障碍用户的影响较小。我们不必通过将它们的持续时间设置为`0`来完全消除过渡，而是可以选择用更温和的过渡来替换更强烈的过渡。我们将在下一节中深入探讨这种方法。

# 为无法访问的用户提供替代过渡

前庭功能障碍用户在接触到基于运动的动画，如缩放或平移大对象时可能会感到不适。然而，他们通常受更微妙的动画，如淡入，的影响较小。

将所有过渡设置为淡入效果以适应前庭功能障碍用户并非万能的解决方案。最好总是寻求用户自身的反馈。

我们将继续使用上一节中的相同示例，并探讨当用户偏好减少运动时，如何从`fly`过渡切换到`fade`过渡。

需要注意的一点是，在 Svelte 中，不允许对一个元素应用超过一个过渡。

例如，以下代码是无效的，会导致构建错误：

```js
<div transition:fade transition:fly />
```

这意味着我们不能将两个过渡应用于一个元素，然后决定使用哪一个。我们必须找到一种方法，在只对一个元素应用一个过渡的同时，在不同的过渡之间切换。

正如我们在*第十四章*中学习到的创建自定义过渡，Svelte 中的过渡是一个遵循过渡契约的函数。函数的返回值决定了过渡将如何执行。

因此，实现一个根据条件在两个过渡之间切换的过渡的一种方法，是创建一个自定义过渡，该过渡根据条件返回不同的过渡配置。

我们的自定义过渡可能看起来像以下代码，根据用户是否偏好减少运动返回不同的过渡配置：

```js
<script>
  function accessibleFly(node, params) {
    const mediaQuery = window.matchMedia('(prefers-reduced-motion: reduce)');
    const matches = mediaQuery.matches;
    if (matches) {
      // user prefers reduced motion
      // return a fade transition
    } else {
      // return a fly transition
    }
  }
</script>
<ul>
  {#each list as item}
    <li transition:accessibleFly={{ x: 40 }}>{item}</li>
  {/each}
</ul>
```

在前面的代码片段中，我们定义了一个 `accessibleFly` 过渡，这是一个更易于访问的 `fly` 过渡，如果用户偏好减少运动，它将切换到 `fade` 过渡。

现在，我们需要确定在我们的自定义 `accessibleFly` 过渡中的每个条件情况下应该返回什么。

重要的是要记住，在 Svelte 中，过渡是一个 JavaScript 函数。因此，我们可以将 `fly` 和 `fade` 过渡都作为函数来调用，并且返回值将是每个相应过渡的过渡配置。通过这样做，我们可以从我们的 `accessibleFly` 过渡中返回这些值，从而有效地使我们的过渡可以是 `fly` 或 `fade` 过渡，具体取决于用户的偏好。

下面是更新后的 `accessibleFly` 过渡：

```js
<script>
  function accessibleFly(node, params) {
    const mediaQuery = window.matchMedia('(prefers-reduced-motion: reduce)');
    const matches = mediaQuery.matches;
    if (matches) {
      // user prefers reduced motion
      return fade(node, params);
    } else {
      return fly(node, params);
    }
  }
</script>
```

在前面的代码片段中，我们根据用户的偏好从 `fade` 或 `fly` 过渡中返回值。我们将传递给我们的 `accessibleFly` 过渡的 `node` 和 `params` 值传递给 `fade` 和 `fly` 过渡。`node` 和 `params` 值指定了过渡应用于哪个元素，并提供了用户参数，例如 `duration` 和 `delay`。这些对于 `fade` 和 `fly` 过渡来说很有用，可以确定过渡应该如何执行。

通过前面的代码更改，我们现在有一个名为 `accessibleFly` 的可访问 `fly` 过渡，默认情况下，它将使元素在过渡过程中飞出。然而，如果用户表示偏好减少运动，`accessibleFly` 过渡将使元素淡出。

由此，我们得到了一个既吸引大多数用户又考虑那些偏好减少运动的人的过渡。

您可以在[`github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter15/01-accessible-transition`](https://github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter15/01-accessible-transition)找到 `accessibleFly` 过渡的代码。

# 摘要

在本章中，我们探讨了无障碍访问在网页设计中的重要性以及如何实现考虑前庭功能障碍用户偏好的过渡效果。通过了解基于运动的动画对前庭功能障碍用户的影响，我们可以创建更包容和用户友好的网络应用程序。

我们学习了关于`prefers-reduced-motion`媒体查询的知识，它允许我们检测用户是否在其系统设置中表明了对减少运动的需求。使用这个媒体查询，我们可以调整我们的过渡效果以减少运动量，或者完全移除对偏好减少运动的用户。

我们还讨论了如何在 Svelte 中创建自定义过渡以实现无障碍访问。我们查看了一个名为`accessibleFly`的自定义过渡示例，它根据用户的减少运动偏好在`fly`和`fade`过渡之间切换。这个自定义过渡考虑到了患有前庭功能障碍的用户，同时为其他用户提供吸引人和有趣的过渡效果。

总结来说，无障碍访问在网页设计中至关重要，过渡效果也不例外。通过考虑所有用户的偏好和需求，包括前庭功能障碍用户，我们可以创建更包容和用户友好的网络应用程序。
