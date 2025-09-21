

# 第一章：Svelte 中的生命周期

Svelte 是一个前端框架。你可以使用 Svelte 来构建网站和 Web 应用程序。一个 Svelte 应用程序由组件组成。你可以在具有`.svelte`扩展名的文件中编写 Svelte 组件。每个`.svelte`文件都是一个 Svelte 组件。

当你创建和使用一个 Svelte 组件时，该组件会经历组件生命周期的各个阶段。Svelte 提供了生命周期函数，允许你钩入组件的不同阶段。

在本章中，我们将首先讨论 Svelte 中的各种生命周期和生命周期函数。在心中有了对生命周期的清晰认识后，你将学习使用生命周期函数的基本规则。这是非常重要的，因为你会发现这种理解将使我们能够以很多创造性的方式使用生命周期函数。

本章包含以下主题的章节：

+   Svelte 生命周期函数是什么？

+   调用生命周期函数的规则

+   如何重用和组合生命周期函数

# 技术要求

编写 Svelte 应用程序非常简单，不需要任何付费工具。尽管大多数付费工具都有附加价值，但我们决定只使用免费工具，以便您无限制地获取本书的内容。

你将需要以下内容：

+   Visual Studio Code 作为集成开发环境([`code.visualstudio.com/`](https://code.visualstudio.com/))

+   一个不错的网络浏览器（例如 Chrome、Firefox 或 Edge）

+   Node.js 作为 JavaScript 运行环境([`nodejs.org/`](https://nodejs.org/))

本章的所有代码示例都可以在 GitHub 上找到：[`github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter01`](https://github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter01)

所有章节的代码可以在[`github.com/PacktPublishing/Real-World-Svelte`](https://github.com/PacktPublishing/Real-World-Svelte)找到。

# 理解 Svelte 的生命周期函数

当使用 Svelte 组件时，它在其生命周期中会经历不同的阶段：挂载、更新和销毁。这类似于人类。我们一生中会经历各种阶段，如出生、成长、老年和死亡。我们称这些不同的阶段为生命周期。

在我们讨论 Svelte 中的生命周期之前，让我们先看看一个 Svelte 组件。

```js
<script>
  import { onMount, beforeUpdate, afterUpdate, onDestroy } from 'svelte';
  let count = 0;
  onMount(() => { console.log('onMount!'); });
  beforeUpdate(() => { console.log('beforeUpdate!'); });
  afterUpdate(() => { console.log('afterUpdate!'); });
  onDestroy(() => { console.log('onDestroy!'); });
</script>
<button on:click={() => { count ++; }}>
  Counter: {count}
</button>
```

你能告诉我代码的每个部分何时执行吗？

代码的并非所有部分都会同时执行；代码的不同部分会在组件生命周期的不同阶段执行。

Svelte 组件有四个不同的生命周期阶段：初始化、挂载、更新和销毁。

## 初始化组件

当你创建一个组件时，组件首先会进入初始化阶段。你可以将其视为设置阶段，此时组件会设置其内部状态。

这就是第 2-7 行正在执行的地方。

声明并初始化 `count` 变量。调用 `onMount`、`beforeUpdate`、`afterUpdate` 和 `onDestroy` 生命周期函数，并传入回调函数，以在组件生命周期的特定阶段注册它们。

在组件初始化后，Svelte 开始在模板中创建元素，在这种情况下，是一个 `<button>` 元素和用于 `"Counter: "` 和 `{count}` 的文本元素。

## 挂载组件

在所有元素创建完成后，Svelte 将按顺序将它们插入到 **文档对象模型** **(DOM)** 中。这被称为挂载阶段，其中元素被挂载到 DOM 上。

如果你向一个元素添加 Svelte 动作，那么这些动作将与元素一起被调用：

```js
<script>
  function action(node) {}
</script>
<div use:action>
```

我们将在 *第五章* 到 *7* 中更深入地探讨 Svelte 动作。

如果你向元素添加事件监听器，这就是 Svelte 将事件监听器附加到元素的时候。

在前一个示例中，Svelte 在将按钮插入 DOM 之后将其附加到 `click` 事件监听器上。

当我们向一个元素添加绑定时，绑定的变量会根据元素的值进行更新：

```js
<script>
  let element;
</script>
<div bind:this={element} />
```

这时，`element` 变量将更新为 Svelte 创建的 `<div>` 元素的引用。

如果你向一个元素添加转换，这就是转换被初始化并开始播放的时候。

以下是一个向元素添加转换的示例。你可以使用 `transition:`, `in:`, 和 `out:` 指令向元素添加转换。我们将在 *第十三章* 到 *15* 中更深入地探讨 Svelte 转换：

```js
<div in:fade />
```

在处理完所有指令，`use:`（动作），`on:`（事件监听器），`bind:` 绑定，`in:`，`transition:`（转换）之后，通过调用在 `onMount` 生命周期函数中注册的所有函数，挂载阶段结束。

这时，第 4 行的函数将被执行，你将在日志中看到打印出 `"onMount!"`。

## 更新组件

当你点击按钮时，`click` 事件监听器被调用。第 9 行的函数被执行。`count` 变量被增加。

在 Svelte 根据最新的 `count` 变量值修改 DOM 之前，将调用在 `beforeUpdate` 生命周期函数中注册的函数。

第 5 行的函数将被执行，你将在日志中看到打印出文本 `"beforeUpdate!"`。

在此阶段，如果你尝试检索按钮内的文本内容，它仍然是 `"``Counter: 0"`。

然后 Svelte 继续修改 DOM，将按钮的文本内容更新为 `"``Counter: 1"`。

在更新组件内的所有元素之后，Svelte 会调用在 `afterUpdate` 生命周期函数中注册的所有函数。

第 6 行的函数被执行，你将在日志中看到打印出文本 `"afterUpdate!"`。

如果你再次点击按钮，Svelte 将通过另一个周期的 `beforeUpdate`，然后更新 DOM 元素，然后 `afterUpdate`。

## 销毁组件

当条件成立时，向用户显示的条件性组件将保持不变；当条件不再成立时，Svelte 将继续销毁组件。

假设我们例子中的组件现在进入了销毁阶段。

Svelte 会调用在 `onDestroy` 生命周期函数中注册的所有函数。第 7 行的函数会被执行，你将在日志中看到打印出的文本 `"onDestroy!"`。

之后，Svelte 会从 DOM 中移除元素。

如果需要，Svelte 会清理指令，例如移除事件监听器和从动作中调用销毁方法。

就这样！如果你尝试再次创建组件，新的周期会再次开始。

Svelte 组件的生命周期从初始化、挂载、更新和销毁开始。Svelte 提供生命周期方法，允许你在组件的不同阶段运行函数。

由于生命周期函数只是从 `'svelte'` 导出的函数，你可以在任何地方导入和使用它们吗？导入和使用它们时有什么规则或约束吗？

让我们来看看。

# 调用生命周期函数的一条规则

调用组件生命周期函数的唯一规则是，你应该在组件初始化期间调用它们。如果没有组件正在初始化，Svelte 会通过抛出错误来抱怨。

让我们看看以下示例：

```js
<script>
  import { onMount } from 'svelte';
  function buttonClicked() {
    onMount(() => console.log('onMount!'));
  }
</script>
<button on:click={buttonClicked} />
```

当你点击按钮时，它会调用 `buttonClicked`，这将调用 `onMount`。由于在调用 `onMount` 时没有组件正在初始化（在你点击按钮时，上面的组件已经初始化并挂载），Svelte 会抛出错误：

```js
Error: Function called outside component initialization
```

是的，Svelte 不允许在组件初始化阶段之外调用生命周期函数。这条规则规定了你可以何时调用生命周期函数。它没有规定在哪里或如何调用生命周期函数。这允许我们重构生命周期函数并以其他方式调用它们。

## 重构生命周期函数

如果你仔细观察调用生命周期函数的规则，你会注意到它关乎何时调用它们，而不是在哪里调用它们。

在 `<script>` 标签的顶层调用生命周期函数并不是必需的。

在以下示例中，`setup` 函数在组件初始化期间被调用，然后反过来调用 `onMount` 函数：

```js
<script>
  import { onMount } from 'svelte';
  setup();
  function setup() {
    onMount(() => console.log('onMount!'));
  }
</script>
```

由于组件仍在初始化中，这是完全正常的。

同样，在组件内部导入 `onMount` 函数也不是必需的。正如你在以下示例中看到的，你可以在另一个文件中导入它；只要在组件初始化期间调用 `onMount` 函数，那就完全没问题：

```js
// file-a.js
import { onMount } from 'svelte';
export function setup() {
  onMount(() => console.log('onMount!'));
}
```

在前面的代码片段中，我们将之前定义的`setup`函数移动到一个名为`file-a.js`的新模块中。然后，在原始的 Svelte 组件中，我们不是定义`setup`函数，而是从`file-a.js`中导入它，如下面的代码片段所示：

```js
<script>
  import { setup } from './file-a.js';
  setup();
</script>
```

由于`setup`函数调用了`onMount`函数，同样的规则也适用于`setup`函数！您不能再在组件初始化之外调用`setup`函数。

## 应该注册哪个组件？

仅看`setup`函数，您可能会想知道，当您调用`onMount`函数时，Svelte 是如何知道您指的是哪个组件的生命周期的？

内部，Svelte 跟踪哪个组件正在初始化。当您调用生命周期函数时，它将您的函数注册到正在初始化的组件的生命周期中。

因此，相同的`setup`函数可以在不同的组件中调用，并为不同的组件注册`onMount`函数。

这解锁了本章的第一个模式：重用生命周期函数。

# 在 Svelte 组件中重用生命周期函数

在上一节中，我们了解到我们可以将生命周期函数的调用提取到一个函数中，并在其他组件中重用该函数。

让我们看看一个例子。在这个例子中，组件在屏幕上添加了 5 秒后，将调用`showPopup`函数。我想在其他组件中重用调用`showPopup`的逻辑：

```js
<script>
  import { onMount } from 'svelte';
  import { showPopup } from './popup';
  onMount(() => {
    const timeoutId = setTimeout(() => {
      showPopup();
    }, 5000);
    return () => clearTimeout(timeoutId);
  });
</script>
```

这里，我可以将逻辑提取到一个函数中，`showPopupOnMount`：

```js
// popup-on-mount.js
import { onMount } from 'svelte';
import { showPopup } from './popup';
export function showPopupOnMount() {
  onMount(() => {
    const timeoutId = setTimeout(() => {
      showPopup();
    }, 5000);
    return () => clearTimeout(timeoutId);
  });
}
```

现在，我可以导入这个函数并在任何组件中重用它：

```js
<script>
  import { showPopupOnMount } from './popup-on-mount';
  showPopupOnMount();
</script>
```

您可能想知道，为什么不只提取回调函数并重用它呢？

```js
// popup-on-mount.js
import { showPopup } from './popup';
export function showPopupOnMount() {
  const timeoutId = setTimeout(() => {
    showPopup();
  }, 5000);
  return () => clearTimeout(timeoutId);
}
```

在这里，我们将`setTimeout`和`clearTimeout`逻辑提取到`showPopupOnMount`中，并将该函数传递给`onMount`：

```js
<script>
  import { onMount } from 'svelte';
  import { showPopupOnMount } from './popup-on-mount';
  onMount(showPopupOnMount);
</script>
```

在我看来，重构和重用的第二种方法不如第一种方法好。将整个生命周期函数的调用提取到函数中有几个优点，因为它允许您做更多的事情：

+   *您可以将不同的输入参数传递给您的生命周期函数。*

    假设您希望允许不同的组件自定义显示弹出窗口前的持续时间。以这种方式传递要容易得多：

    ```js
    <script>
      import { showPopupOnMount } from './popup-on-mount';
      showPopupOnMount(2000); // change it to 2s
    </script>
    ```

+   *您可以从函数中返回值。*

    假设您想返回`onMount`函数中使用的`timeoutId`，这样您就可以在用户在组件中的任何按钮上点击时取消它。

    如果您只是重用回调函数，这将几乎不可能做到，因为回调函数返回的值将用于注册`onDestroy`生命周期函数：

    ```js
    <script>
      import { showPopupOnMount } from './popup-on-mount';
      const timeoutId = showPopupOnMount(2000);
    </script>
    <button on:click={() => clearTimeout(timeoutId)} />
    ```

    看看这样写，实现它以返回任何内容是多么容易：

    ```js
    // popup-on-mount.js
    export function showPopupOnMount(duration) {
      let timeoutId;
      onMount(() => {
        timeoutId = setTimeout(() => {
          showPopup();
        }, duration ?? 5000);
        return () => clearTimeout(timeoutId);
      });
      return timeoutId;
    }
    ```

+   *您可以将更多逻辑与生命周期函数一起封装。*

    有时，您在生命周期函数回调函数中的代码在隔离环境中不起作用；它会与其他变量交互并修改它们。为了重用此类生命周期函数，必须将这些变量和逻辑封装到一个可重用的函数中。

    为了说明这一点，让我们看看一个新的例子。

    在这里，我有一个计数器，当组件被添加到屏幕上时开始计数：

    ```js
    <script>
      import { onMount } from 'svelte';
      let counter = 0;
      onMount(() => {
        const intervalId = setInterval(() => counter++, 1000);
        return () => clearInterval(intervalId);
      });
    </script>
    <span>{counter}</span>
    ```

    `counter` 变量与 `onMount` 生命周期函数相关联；为了重用此逻辑，应将 `counter` 变量和 `onMount` 函数一起提取到一个可重用的函数中：

    ```js
    import { writable } from 'svelte/store';
    import { onMount } from 'svelte';
    export function startCounterOnMount() {
      const counter = writable(0);
      onMount(() => {
        const intervalId = setInterval(() => counter.update($counter => $counter + 1), 1000);
        return () => clearInterval(intervalId);
      });
      return counter;
    }
    ```

    在这个例子中，我们使用一个 `writable` Svelte 存储使 `counter` 变量变得响应式。我们将在本书的 *第三部分* 中更深入地探讨 Svelte 存储。

    目前，您需要理解的是，Svelte 存储允许 Svelte 跨模块跟踪变量的变化，并且您可以通过在 Svelte 存储变量前加上 `$` 来订阅和检索存储的值。例如，如果您有一个名为 `counter` 的 Svelte 存储，那么要获取 Svelte 存储的值，您需要使用 `$counter` 变量。

    现在，我们可以在任何 Svelte 组件中使用 `startCounterOnMount` 函数：

    ```js
    <script>
      import { startCounterOnMount } from './counter';
      const counter = startCounterOnMount();
    </script>
    <span>{$counter}</span>
    ```

希望我已经说服您提取生命周期函数调用的优点。让我们通过一个例子来试试。

## 练习 1 – 更新计数器

在下面的示例代码中，我想知道组件经过了多少次更新周期。

利用每次组件经过更新周期时都会调用 `afterUpdate` 回调函数的事实，我创建了一个计数器，每次 `afterUpdate` 回调函数被调用时都会递增。

为了帮助我们仅测量特定用户操作的更新计数，我们有开始测量和停止测量的函数，因此更新计数器仅在测量时递增：

```js
<script>
  import { afterUpdate } from 'svelte';
  let updateCount = 0;
  let measuring = false;
  afterUpdate(() => {
    if (measuring) {
      updateCount ++;
    }
  });
  function startMeasuring() {
    updateCount = 0;
    measuring = true;
  }
  function stopMeasuring() {
    measuring = false;
  }
</script>
<button on:click={startMeasuring}>Measure</button>
<button on:click={stopMeasuring}>Stop</button>
<span>Updated {updateCount} times</span>
```

为了重用 `counter:` 的所有逻辑——更新周期的计数以及测量开始和停止——我们应该将所有这些移动到一个函数中，最终看起来像这样：

```js
<script>
  import { createUpdateCounter } from './update-counter';
  const { updateCount, startMeasuring, stopMeasuring } = createUpdateCounter();
</script>
<button on:click={startMeasuring}>Measure</button>
<button on:click={stopMeasuring}>Stop</button>
<span>Updated {$updateCount} times</span>
```

更新计数器返回一个包含 `updateCount` 变量和 `startMeasuring` 以及 `stopMeasuring` 函数的对象。

`createUpdateCounter` 函数的实现留作练习，您可以在 [`github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter01/01-update-counter`](https://github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter01/01-update-counter) 查看答案。

我们已经学习了如何提取生命周期函数并重用它，所以让我们更进一步，在下一个模式中重用多个生命周期函数：组合生命周期函数。

# 将生命周期函数组合成可重用钩子

到目前为止，我们主要讨论了重用单个生命周期函数。然而，没有任何阻止我们将多个生命周期函数组合起来执行一个函数。

这里摘录了来自 [`svelte.dev/examples/update`](https://svelte.dev/examples/update) 的示例。该示例显示了一个消息列表。当新消息被添加到列表中时，容器会自动滚动到最底部以显示新消息。在代码片段中，我们看到这种自动滚动行为是通过结合使用 `beforeUpdate` 和 `afterUpdate` 实现的：

```js
<script>
  import { beforeUpdate, afterUpdate } from 'svelte';
  let div;
  let autoscroll;
  beforeUpdate(() => {
    autoscroll = div && (div.offsetHeight + div.scrollTop) > (div.scrollHeight - 20);
  });
  afterUpdate(() => {
    if (autoscroll) div.scrollTo(0, div.scrollHeight);
  });
</script>
<div bind:this={div} />
```

要在其他组件中重用这个 `autoscroll` 逻辑，我们可以将 `beforeUpdate` 和 `afterUpdate` 逻辑一起提取到一个新函数中：

```js
export function setupAutoscroll() {
  let div;
  let autoscroll;
  beforeUpdate(() => {
    autoscroll = div && (div.offsetHeight + div.scrollTop) > (div.scrollHeight - 20);
  });
  afterUpdate(() => {
    if (autoscroll) div.scrollTo(0, div.scrollHeight);
  });
  return {
  setDiv(_div) {
  div = _div;
    },
  };
}
```

然后，我们可以在任何组件中使用提取的函数，`setupAutoScroll`：

```js
<script>
  import { setupAutoscroll } from './autoscroll';
  const { setDiv } = setupAutoscroll();
  let div;
  $: setDiv(div);
</script>
<div bind:this={div} />
```

在重构后的 `setupAutoscroll` 函数中，我们返回一个 `setDiv` 函数，以便我们可以在 `setupAutoscroll` 函数内部更新 `div` 的引用。

正如你所见，通过遵循在组件初始化期间调用生命周期函数的一条规则，你可以将多个生命周期函数组合成可重用的钩子。到目前为止你所学的足以组合生命周期函数，但未来还有更多替代方案。在接下来的章节中，你将探索第 *5 章* 中的 Svelte 动作和第 *8 章* 中的 Svelte 存储，进一步扩展你的选择。以下是一些这些替代方案的预览。

另一种实现方式可能是将 `div` 设为一个可写存储，并在 `setupAutoscroll` 函数中返回它。这样，我们就可以直接绑定到 `div` 可写存储，而无需手动调用 `setDiv`。

或者，我们可以返回一个遵循 Svelte 动作合约的函数，并在 `div` 上使用该动作：

```js
export function setupAutoscroll() {
  let div;
  // ...
  return function (node) {
    div = node;
    return {
      destroy() {
        div = undefined;
      },
    };
  };
}
```

`setupAutoscroll` 现在返回一个动作，我们使用这个动作在我们的 `div` 容器上：

```js
<script>
  import { setupAutoscroll } from './autoscroll';
  const autoscroll = setupAutoscroll();
</script>
<div use:autoscroll />
```

我们将在本书的后面部分更详细地讨论 Svelte 动作合约。

我们已经看到如何将生命周期函数提取到单独的文件中，并在多个 Svelte 组件中重用它。目前，组件独立调用生命周期函数，并作为独立单元运行。是否有可能同步或协调使用相同生命周期函数的组件之间的动作？让我们来找出答案。

# 在组件间协调生命周期函数

由于我们在组件间重用相同的函数，我们可以全局跟踪使用相同生命周期函数的组件。

让我给你举一个例子。在这里，我想跟踪屏幕上有多少组件正在使用我们的生命周期函数。

要计算组件的数量，我们可以在模块级别定义一个变量，并在我们的生命周期函数中更新它：

```js
import { onMount, onDestroy } from 'svelte';
import { writable } from 'svelte/store';
let counter = writable(0);
export function setupGlobalCounter() {
  onMount(() => counter.update($counter => $counter + 1));
  onDestroy(() => counter.update($counter => $counter - 1));
  return counter;
}
```

由于 `counter` 变量是在 `setupGlobalCounter` 函数外部声明的，因此相同的 `counter` 变量实例被用于并共享在所有组件之间。

当任何组件被挂载时，它将增加 `counter`，任何引用 `counter` 的组件都将更新为最新的计数器值。

当你想要在组件之间设置一个共享的通信通道并在组件被销毁时在`onDestroy`中将其拆除时，这种模式非常有用。

让我们尝试在我们的下一个练习中使用这项技术。

## 练习 2 – 滚动阻止器

通常情况下，当你将一个弹出组件添加到屏幕上时，你希望文档不可滚动，这样用户就能专注于弹出窗口，并且只在该弹出窗口内滚动。

这可以通过将`body`的`overflow` CSS 属性设置为`"hidden"`来实现。

编写一个可重用的函数，该函数由弹出组件使用，在弹出组件安装时禁用滚动。当弹出组件被销毁时，恢复初始的`overflow`属性值。

请注意，屏幕上可以同时安装多个弹出组件，因此只有当所有弹出窗口都被销毁时，你才应该恢复`overflow`属性值。

你可以在[`github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter01/02-scroll-blocker`](https://github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter01/02-scroll-blocker)查看答案。

# 摘要

在本章中，我们探讨了 Svelte 组件的生命周期。我们看到了组件生命周期的不同阶段，并学习了生命周期函数回调将在何时被调用。

我们还介绍了调用生命周期函数的规则。这有助于我们实现生命周期函数的重用和组合的不同模式。

在下一章中，我们将开始探讨为 Svelte 组件进行样式化和主题化的不同模式。
