# 9

# 实现自定义存储

在上一章中，我们了解到 Svelte 存储是任何遵循 Svelte 存储合约的对象，将数据封装在 Svelte 存储中允许数据在多个 Svelte 组件之间反应性地共享和使用。Svelte 组件会根据数据更新 DOM，即使数据是在 Svelte 组件外部被修改的。

我们学习了 Svelte 创建 Svelte 存储的两种内置方法——即 `readable()` 和 `writable()`，它们创建一个可读和可写的存储。这两种方法遵循 Svelte 合约并创建一个非常基础的 Svelte 存储。然而，除了使用 Svelte 存储封装数据外，我们还可以封装与数据相关的逻辑，使 Svelte 存储模块化且高度可重用。

在本章中，我们将创建自定义 Svelte 存储——封装自定义数据逻辑的 Svelte 存储。我们将通过三个不同的示例进行讲解；每个示例都将作为指南，充满创建您自己的自定义 Svelte 存储的技巧和窍门。

首先，我们将探讨如何将用户事件转换为存储值，特别是将点击次数转换为 Svelte 存储。

其次，我们将探讨一个超出基本 `set` 方法以修改其值的自定义存储。我们将查看一个撤销/重做存储，它包含用于撤销或重做对其值更改的额外方法。

最后，我们将关注高级存储。虽然高级存储本身不是一个自定义存储，但它是一个接受 Svelte 存储作为输入并返回其增强版本的函数。本章包括以下主题的章节：

+   从用户事件创建 Svelte 存储

+   创建一个撤销/重做存储

+   创建一个防抖的高级 Svelte 存储

因此，无需多言，让我们深入探讨创建我们的第一个自定义 Svelte 存储。

# 技术要求

本章中所有的代码都可以在 GitHub 上找到：[`github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter09`](https://github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter09)

# 从用户事件创建 Svelte 存储

Svelte 存储存储数据，但数据从何而来？

它可能来自用户交互或用户输入，这会调用一个事件处理器，通过调用 `store.set()` 来更新存储值。

如果我们能够将用户事件和事件处理器逻辑封装到存储中，这样我们就不需要手动调用 `store.set()` 会怎样？

例如，我们将有一个 Svelte 存储来计算用户在屏幕上点击的次数。如果我们能够创建一个 Svelte 存储，并在每次有新的点击时更新它，而不是手动在文档上添加事件监听器，那会是什么样子？简而言之，有没有一个可以为我们做所有这些的自定义 Svelte 存储？

如果我们能够在下一次有类似需求时重用这个 Svelte 存储，而不是再次手动设置它，那将是非常棒的。

因此，让我们尝试实现这个点击计数自定义 Svelte store。

让我们首先构建 Svelte store：

```js
const subscribers = [];
let clickCount = 0;
const store = {
  subscribe: (fn) => {
    fn(clickCount);
    subscribers.push(fn);
    return () => {
      subscribers.splice(subscribers.indexOf(fn), 1);
    }
  },
}
```

在前面的代码片段中，我们创建了一个基于 Svelte store contract 的基本 Svelte store，它只有一个`subscribe()`方法。store 的值存储在一个名为`clickCount`的变量中，在`subscribe()`方法中，我们使用`subscribers`数组跟踪所有订阅者。

注意，我们需要在`subscribe()`方法中同步调用`fn`与 store 的值；这样可以让订阅者知道 store 的当前值。

如果你现在在 Svelte 组件中使用这个 store（如下面的代码片段所示），你会在屏幕上看到`0`。这就是此时 store 的当前值：

```js
<script>
  import { store } from './store.js';
</script>
{$store}
```

在前面的代码片段中，我们将`store`导入到 Svelte 组件中，并使用`$store`显示 store 的值。现在，让我们监听`click`事件以更新 store 的`count`值。我们不应该在程序开始时监听`click`事件，而应该只在存在订阅者时开始订阅。

下面是更新后的代码：

```js
const store = {
  subscribe: (fn) => {
    // ...
    document.addEventListener('click', () => {
      clickCount++;
      // notify subscribers
      subscribers.forEach(subscriber => subscriber(clickCount));
    });
  },
};
```

在前面的代码片段中，我们在`subscribe`方法内部添加了`click`事件监听器，每当点击`document`时，我们就将`clickCount`的值增加`1`，并通知所有订阅者`clickCount`的最新值。

如果你现在尝试在多个 Svelte 组件中使用这个 Svelte store，你会意识到每次点击，store 的值都会增加多于一个。

## 确保事件监听器只添加一次

为什么点击一次时 store 的值会增加多于一次？

如果你仔细查看当前实现，你会意识到我们在每次`subscribe`方法调用时都调用了`document.addEventListener`。当你使用 Svelte store 在多个 Svelte 组件中时，每个 Svelte 组件都会单独订阅 store 的变化。如果有五个组件订阅了 store，那么五个`click`事件监听器将被添加到`document`上。结果，`document`上的单次点击将触发五个事件监听器，导致`clickCount`的值每次增加五。这意味着每次点击，`store`的值都会增加多于一个。为了修复这种行为，我们仍然可以在`subscribe`方法中调用`addEventListener`，但我们需要确保只调用一次，即使`subscribe`方法可能被多次调用。

我们可以使用一个标志来指示是否已经调用了`addEventListener`，并确保当标志设置为`true`时不再调用`addEventListener`，如下所示：

```js
let called = false;
const store = {
  subscribe: (fn) => {
    // ...
    if (!called) {
      called = true;
      document.addEventListener('click', () => {... });
    }
  },
};
```

在前面的代码片段中，我们添加了一个名为`called`的变量，并使用它来防止`document`添加多于一次的点击事件监听器。

这可以工作，但还有更好的实现方式。

我们不需要设置一个新的标志来指示是否已经调用过 `addEventListener`，我们可以使用任何现有的变量来确定是否应该调用 `addEventListener`。我们知道一旦调用 `subscribe` 方法，就应该调用 `addEventListener`，而在我们添加更多订阅者时不应随后再次调用；我们可以使用 `subscribers` 数组的长度来确定是否应该调用 `addEventListener` 方法。

如果目前没有订阅者，那么我们知道这是第一次调用 `subscribe` 方法。在这种情况下，我们应该调用 `addEventListener`。另一方面，如果存在现有订阅者——即 `subscribers` 的长度大于零，我们知道 `subscribe` 方法之前已经被调用过，因此我们不应再次调用 `addEventListener`。

因此，以下是更新后的代码，使用 `subscribers` 数组的长度而不是变量来确定是否应该向文档添加点击事件监听器：

```js
const store = {
  subscribe: (fn) => {
    // ...
    if (subscribers.length === 0) {
      document.addEventListener('click', () => {... });
    }
  },
};
```

在前面的代码片段中，我们将 `!called` 条件替换为 `subscribers.length === 0`。

现在我们已经为订阅我们的 Svelte 存储时添加了点击事件监听器，当所有订阅者从 Svelte 存储中取消订阅时，我们需要进行清理。

为了进行清理，我们将调用 `removeEventListener` 来从 `document` 中移除 `click` 事件监听器。`unsubscribe` 函数可以被多次调用，但我们应该只在 `subscribers` 数组中没有更多订阅者时调用 `removeEventListener`。

```js
const store = {
  subscribe: (fn) => {
    return () => {
      subscribers.splice(subscribers.indexOf(fn), 1);
      if (subscribers.length === 0) {
        document.removeEventListener('click', () => {...});
      }
    };
  },
};
```

在前面的代码片段中，`subscribe` 方法的 `return` 函数用于取消存储的订阅。在函数中，我们添加了一个检查来查看 `subscribers` 的数量是否下降到零；如果是这样，我们将从 `document` 中移除 `click` 事件监听器。

当你创建一个 Svelte 存储时，通常需要跟踪 `subscribers` 列表，确保你只设置一次事件监听器，并且在没有更多订阅者后进行清理。

正如你在本节和前一节中看到的，我们已经经历了很多步骤来从用户事件创建 Svelte 存储管理 `subscribers` 数组，并决定何时向文档添加或移除点击事件监听器。在下一节中，我们将探索使用 Svelte 的内置方法和内置 `readable()` 函数以更简单的方式实现相同目标的方法。

Svelte 提供了内置方法，如 `readable()`，以使我们在创建 Svelte 存储时生活更加轻松。

由于存储值只由 `click` 事件更新，而不是其他地方，所以 `readable()` 在两个最重要的方法——`readable()` 和 `writable()` 中——对于我们的用例来说已经足够好了。

我们将使用`readable()`函数来创建我们的点击事件存储。`readable()`函数有两个参数。第一个参数是存储的初始值。`readable()`函数的第二个参数接受一个函数，该函数在第一个订阅者订阅时会被调用，而对于后续的订阅者则不会被调用。如果该函数返回另一个函数，则当最后一个订阅者从存储中取消订阅时，返回的函数会被调用。这对于我们在`document`上添加和移除`click`事件监听器来说非常完美。

让我们看看使用`readable`重写的更新后的代码：

```js
let clickCount = 0;
const store = readable(clickCount, (set) => {
  const onClick = () => set(++clickCount);
  document.addEventListener('click', onClick);
  return () => {
    document.removeEventListener('click', onClick);
  };
});
```

在前面的代码片段中，我们利用`readable()`来处理`click`事件监听器的创建和清理，使我们的代码更简洁、更高效。

与上一节中我们自行管理`subscribers`列表的实现相比，你可以看到使用`readable()`如何使我们能够从手动维护`subscribers`数组中解脱出来，并专注于实现逻辑。

了解 Svelte 合约以及如何从头开始实现 Svelte 存储是很好的。但在大多数实际场景中，从`readable()`或`writable()`创建 Svelte 存储更容易，并将细节留给内置函数。

现在你已经学会了如何从点击事件创建 Svelte 存储，让我们通过一个练习来练习一下。

## 练习

让我们开始我们的练习，我们将实现一个 Svelte 存储，其值来自文档的滚动位置：

```js
<script>
  const scrollPosition = createStore();
  function createStore() {
    // Your code here
  }
</script>
Scroll position {$scrollPosition}
```

在代码片段中，使用名为`createStore()`的函数设置了一个`scrollPosition`存储。你的任务是实现`createStore()`函数以实际创建一个`scrollPosition`存储。

你可以在以下链接找到解决方案：[`github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter09/02-exercise-scroll-position`](https://github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter09/02-exercise-scroll-position)

现在我们已经看到了如何创建一个存储值来自事件的 Svelte 存储，让我们看看另一种类型的 Svelte 存储。

# 创建撤销/重做存储

通常，我们使用`set`方法来更改存储的值。然而，我们接下来要探索的下一个自定义 Svelte 存储提供了额外的自定义方法来更新其存储值。我们将要查看的下一个自定义 Svelte 存储是一个撤销/重做存储。它类似于可写存储，你可以订阅并设置新的存储值。但撤销/重做存储还提供了两个额外的函数，`undo`和`redo`，它们根据存储值的历史记录将存储值向前或向后回退。

这里是一个如何使用撤销/重做存储的示例：

```js
<script>
  let value = createUndoRedoStore();
  $value = 123;
  $value = 456;
  $value = 789;
  value.undo(); // $value now goes back to 456
  value.undo(); // $value now goes back to 123
  value.redo(); // $value now turns to 456
</script>
Value: {$value}
```

在提供的代码片段中，`createUndoRedoStore()`函数生成一个撤销/重做存储。最初，我们将存储的值设置为`123`，然后更新为`456`，接着是`789`。当我们调用存储的`undo`方法时，值会回滚到`456`，然后是`123`。随后，使用`redo`方法将存储的值再次恢复到`456`。现在我们了解了撤销/重做存储的工作原理，那么在 Svelte 中如何创建一个这样的存储呢？

首先，撤销/重做存储将拥有四个方法：`subscribe`、`set`、`undo`和`redo`。`subscribe`和`set`方法基于 Svelte 存储合约，这也是为什么撤销/重做存储被认为是 Svelte 存储的原因。另一方面，`undo`和`redo`方法是两个我们定义的额外方法。

一个 JavaScript 对象可以包含不同的方法和属性，但只要它有`subscribe`和`set`方法，并且方法签名遵循 Svelte 存储合约，我们就认为该对象是 Svelte 存储。你可以使用以`$`前缀的变量来自动订阅 Svelte 存储并引用 Svelte 存储的值。

现在，为了实现这个撤销/重做存储，我们知道如果没有撤销/重做功能，存储的行为就像一个可写存储。因此，我们将基于可写存储实现撤销/重做存储，如下面的代码片段所示：

```js
function createUndoRedoStore() {
  const store = writable();
  return store;
}
```

在前面的代码片段中，我们为`createUndoRedoStore()`函数设定了场景。我们首先使用 Svelte 的`writable()`函数创建一个可写存储，这将是我们的撤销/重做存储的基础。

但由于我们正在将新值设置到撤销/重做存储中，我们需要跟踪存储值的记录，以便我们可以撤销或重做它们。

为了做到这一点，我们将拦截可写存储的`set`方法，如下所示：

```js
function createUndoRedoStore() {
  const store = writable();
  function set(value) {
    store.set(value);
  }
  return {
    subscribe: store.subscribe,
    set: set,
  };
}
```

在前面的代码中，我返回了一个新的对象。虽然`subscribe`方法与原始可写存储的`subscribe`方法相同，但`set`方法现在是一个新的函数。我们仍然在`set`函数中调用可写存储的`set`方法，所以行为变化不大。

但现在，当我们调用撤销/重做存储的`set`方法时，我们首先调用`set`函数，然后再将这个调用传递给底层可写存储的`set`方法。这允许我们在`set`函数中添加额外的逻辑，该逻辑将在我们设置撤销/重做存储的新值时运行。

在我们急于求成之前，别忘了我们还需要在撤销/重做存储中添加两个更多的方法，即`undo`和`redo`。下面是如何实现这一点的说明：

```js
function createUndoRedoStore() {
  const store = writable();
  function set(value) { store.set(value); }
  function undo() {}
  function redo() {}
  return {
    subscribe: store.subscribe,
    set: set,
    undo: undo,
    redo: redo,
  };
}
```

在前面的代码片段中，我们向`createUndoRedoStore()`返回的对象中添加了两个额外的方法，即`undo`和`redo`。我们将继续在接下来的步骤中实现这些方法。现在，我们已经拥有了撤销/重做存储的基本结构。

你可以将前面的代码视为创建自定义 Svelte 存储的模板。我们使用可写存储作为基础并返回一个新的对象。返回的对象被视为 Svelte 存储，因为通过拥有 `subscribe` 和 `set` 方法，它遵循 Svelte 存储契约。如果我们想向 `subscribe` 或 `set` 方法添加逻辑，我们可以基于可写存储的原始 `subscribe` 和 `set` 方法构建一个新的函数。除此之外，我们还可以向返回的对象添加更多方法。

## 实现撤销/重做逻辑

现在，为了实现撤销/重做逻辑，我们正在创建两个数组，`undoHistory` 和 `redoHistory`，以记录我们可以在我们调用 `undo()` 或 `redo()` 时回放的价值历史。

每当调用 `set` 函数时，我们将值作为一个新条目添加到 `undoHistory` 中，这样我们就可以在调用 `undo()` 时稍后回放它。当调用 `undo()` 时，我们将 `undoHistory` 中的最新条目推入 `redoHistory` 中，这样我们就可以重做我们刚刚撤销的操作。

让我们继续实现我们刚刚描述的逻辑：

```js
function createUndoRedoStore() {
  const store = writable();
  const undoHistory = [];
  const redoHistory = [];
  function set(value) {
    undoHistory.push(value);
    redoHistory.length = 0; // resets redoHistory
    store.set(value);
  }
  function undo() {
    if (undoHistory.length <= 1) return;
    redoHistory.push(undoHistory.pop());
    store.set(undoHistory[undoHistory.length – 1]);
  }
  function redo() {
    if (redoHistory.length === 0) return;
    const value = redoHistory.pop();
    undoHistory.push(value);
    store.set(value);
  }
  // ...
}
```

在前面的代码片段中，我们使用两个数组：`undoHistory` 和 `redoHistory` 实现了 `undo` 和 `redo` 函数。我们在执行撤销或重做操作之前还添加了检查，以查看这些数组中是否有任何值。这确保了我们不会在没有历史记录可以回滚或前进的情况下尝试撤销或重做。因此，你现在已经学会了如何创建一个从可写存储扩展并添加新行为到原始 `set` 方法，以及向存储添加新方法的自定义 Svelte 存储。

是时候进行练习了。

## 练习

现在我们已经学会了如何创建一个提供自定义方法来操作底层存储值的自定义存储，让我们通过一个练习来构建另一个这样的自定义存储，一个缓动存储。缓动存储是一个只能包含数值的 Svelte 存储。当你设置缓动 Svelte 存储的值时，缓动 Svelte 存储会花费一个固定的持续时间来更新其存储值到设置的值。

例如，让我们假设缓动 Svelte 存储最初设置为 `0`：

```js
const store = createTweenedStore(0); // $store = 0
```

然后，你将存储的值设置为 `10`：

```js
$store = 10;
```

存储值不是直接设置为 `10`，而是在可配置的固定持续时间内从 `0` 增加到 `10`——比如说，1 秒。

现在你已经学会了缓动存储的行为，让我们实现一个缓动 Svelte 存储。你可以在 GitHub 上找到缓动存储的代码，链接为 [`github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter09/04-exercise-tweened-store`](https://github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter09/04-exercise-tweened-store)。

现在我们已经学会了如何创建自定义存储，让我们将注意力转向另一个概念，一个高阶存储——一个接受 Svelte 存储作为输入并返回基于该输入的更专业、更自定义存储的函数。

# 创建一个防抖高阶 Svelte 存储

我们到目前为止看到的两个先前的部分各自创建了一个新的 Svelte 存储。在本节中，我们将探讨如何创建一个高阶存储。

高阶存储的概念灵感来源于高阶函数，其中函数被当作其他数据一样对待。这意味着你可以将函数作为参数传递给其他函数，或者将它们作为值返回。

在类似的概念下，我们将创建一个函数，将存储当作任何数据一样对待，接受一个 Svelte 存储作为参数，然后返回一个 Svelte 存储。

高阶 Svelte 存储的想法是创建一个函数来增强现有的 Svelte 存储。高阶 Svelte 存储是一个函数，它接受一个 Svelte 存储，并返回一个新的 Svelte 存储，这是输入 Svelte 存储的增强版本。

我们将要使用的示例将创建一个 `debounce` 高阶 Svelte 存储。

我将要创建的 `debounce` 函数将接受一个 Svelte 存储，并返回一个新的 Svelte 存储，其存储值基于输入 Svelte 存储值进行防抖：

```js
<script>
  function debounce(store) { ... }
  const store = writable();
  const debouncedStore = debounce(store);
</script>
Store value: {$store}
Debounced store value: {$debouncedStore}
```

在前面的代码片段中，第四行展示了我们将在此部分实现 `debounce` 函数的使用方法。这个 `debounce` 函数接受一个 Svelte 存储作为参数，并返回其增强版本，我们将其称为 `debouncedStore`。为了展示防抖功能，原始 `store` 和 `debouncedStore` 参数的值并排显示。

在开始实现 `debounce` 函数之前，让我们谈谈什么是防抖。

## 防抖存储值变化

在工程学中，防抖是从用户输入中移除不想要的输入噪声的过程。在 Web 开发中，当有太多用户事件时，我们会使用防抖，只在用户事件稳定后触发事件处理器或处理事件。

这里有一个防抖的例子。在实现自动完成搜索时，我们不想在用户输入每个字符时都触发自动完成结果的搜索；相反，我们只在用户停止输入后才开始搜索。这样可以节省资源，因为当用户输入下一个字符时，自动完成结果可能就不再可用。

将防抖的概念应用于 Svelte 存储，我们将创建一个新的基于输入 Svelte 存储的防抖 Svelte 存储。当输入 Svelte 存储的值更新时，防抖 Svelte 存储仅在 Svelte 存储更新稳定后才会更新。

如果我们要从头开始创建一个延迟的 Svelte 存储，我们可以基于可写存储构建它。我在上一节中展示了如何从可写存储创建自定义 Svelte 存储，所以我希望你知道如何做。请亲自尝试一下；当你完成时，将你的实现与我提供的实现进行比较，请参阅[`github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter09/05-debounce-store`](https://github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter09/05-debounce-store)。

但在本节中，我们将创建一个高阶存储。我们已经有了一个 Svelte 存储。它可能是一个可写存储或撤销/重做存储。我们的延迟函数将接受 Svelte 存储，并返回一个延迟版本的 Svelte 存储。

让我们从 `debounce` 高阶存储的基本结构开始。

`debounce` 函数将返回一个新的 Svelte 存储。基于可写存储构建一个新的 Svelte 存储比从头开始实现它、创建一个订阅函数和维护订阅者数组要容易得多。

下面是 `debounce` 函数的基本框架：

```js
function debounce(store) {
  const debounced = writable();
  return {
    subscribe: debounced.subscribe,
    set: store.set,
  };
}
```

在前面的代码片段中，返回的 Svelte 存储将基于可写存储，因此 `subscribe` 方法将是可写存储的 `subscribe` 方法。

`set` 方法将是原始存储的 `set` 方法，而不是一个新的 `set` 函数。我们不会创建一个单独的 `set` 函数，在其中设置原始存储并尝试使用延迟逻辑设置一个延迟的存储。

我们不会像这个片段中那样做：

```js
function debounce(store) {
  const debounced = writable();
  function set(value) {
    store.set(value);
    // some debounce logic and call debounce.set(value);
  }
  return {
    subscribe: debounced.subscribe,
    set: set,
  };
}
```

在前面的代码片段中，我们拦截了 `set` 方法并将其转发到原始存储和延迟存储。我们不会这样做，因为我们想保留原始 Svelte 存储的逻辑。当一个值传递给 `set` 方法时，它可能会经历转换，特别是如果存储是自定义 Svelte 存储。`set` 方法中的 `value` 参数可能不等于最终的存储值。因此，我们让原始存储处理这些潜在的转换，并订阅原始 Svelte 存储以获取最终的存储值。我们使用最终的存储值，并在原始存储值变化时更新延迟存储，如下面的代码片段所示：

```js
function debounce(store) {
  const debounced = writable();
  store.subscribe(value => {
    // some debounce logic and call debounce.set(value);
  });
  // ...
}
```

在前面的代码片段中，我展示了如何订阅原始 Svelte 存储而不是拦截其 `set` 方法。这种方法允许我们保持原始存储的逻辑完整，同时仍然受益于其功能。它还处理了原始 Svelte 存储值可能通过其他方法更改的情况。这种情况的一个例子是撤销/重做存储的 `undo()` 和 `redo()` 方法。如果我们只拦截 `set` 函数的值，就像前面的代码片段中那样，那么当原始撤销/重做存储正在撤销或重做时，延迟的 Svelte 存储不会改变。

为了实现去抖逻辑，我们将使用一个超时来更新去抖动的 Svelte 存储库。如果在超时期间原始 Svelte 存储库有新的变化，我们将取消之前的超时并设置一个新的超时。如果没有，那么我们假设变化已经稳定，我们将更新`debounced`存储库。

这是带有去抖逻辑的更新代码片段：

```js
function debounce(store) {
  const debounced = writable();
  let timeoutId = null;
  store.subscribe(value => {
    if (timeoutId !== null) clearTimeout(timeoutId);
    timeoutId = setTimeout(() => {
      timeoutId = null;
      debounced.set(value);
    }, 200);
  });
  // ...
}
```

在前面的代码片段中，我们在`subscribe`回调函数中使用`setTimeout`函数设置了一个 200 毫秒的超时。在这段时间内，如果原始存储库的值再次变化，现有的超时将被清除，并设置一个新的超时。但如果在 200 毫秒的时间段内有新值，我们将使用原始存储库的最新值更新`debounced`存储库。

尝试使用`debounced`存储库，并查看存储库的值现在已从变化中去抖动。

你可能会注意到，我们订阅了原始的 Svelte 存储库，但还没有取消订阅它。让我们接下来解决这个问题。

## 根据需要订阅和取消订阅原始存储库

在我们结束之前，我们实现到目前为止的`debounced`存储库有一个小问题。我们在创建`debounced` Svelte 存储库时订阅了原始的 Svelte 存储库，但我们从未取消订阅它，即使`debounced` Svelte 存储库可能不再被使用。我们应该只在`debounced`存储库已经有订阅者时才开始订阅原始的 Svelte 存储库，并在没有订阅者时取消订阅。

我们可以拦截`subscribe`方法，并在去抖动的 Svelte 存储库被取消订阅时尝试取消订阅原始的 Svelte 存储库，如下所示：

```js
function debounce(store) {
  const debounced = writable();
  function subscribe(fn) {
    const debouncedUnsubscribe = debounced.subscribe(fn);
    const unsubscribe = store.subscribe(...);
    return () => {
      debouncedUnsubscribe();
      unsubscribe();
    };
  }
  // ...
  return {
    subscribe: subscribe,
    set: store.set,
  };
}
```

在更新的代码片段中，`subscribe`方法现在同时订阅了`debounced`存储库和原始 Svelte 存储库。在用于取消订阅`debounced`存储库的`return`函数中，我们也确保取消订阅原始 Svelte 存储库。

然而，这并不完全正确。这样做意味着每次我们订阅去抖动存储库时，我们都会订阅原始存储库，每次我们取消订阅去抖动存储库时，我们都会取消订阅原始存储库。

我们应该旨在只订阅原始存储库一次，无论有多少订阅者。同样，只有当没有订阅者剩下时，我们才应该取消订阅它。这听起来熟悉吗？

在上一节中，当我们尝试实现一个点击计数器 Svelte 存储库时，我们遇到了一个类似的难题。在我们发现 Svelte 提供了一个内置的`writable()`函数可以很好地处理这个问题之前，我们试图维护一个订阅者列表。

那么，有没有一个内置函数允许我们在订阅另一个存储库时只订阅一次，并且只有当存储库中没有更多订阅者时才取消订阅该存储库？

是的——这正是`derived()`发挥作用的地方。

## 使用 `derived()` 函数派生新的 Svelte 存储对象

`derived()` 是 Svelte 中的一个内置函数，它接受一个或多个 Svelte 存储并返回一个新的 Svelte 存储对象，其存储值基于输入的 Svelte 存储对象。

例如，你可以使用 `derived()` 定义一个 Svelte 存储对象，其存储值总是另一个 Svelte 存储值的两倍，如下面的代码片段所示：

```js
import { writable, derived } from 'svelte/store';
const store = writable(1);
const double = derived(store, value => value * 2);
```

或者，这也可以是一个 Svelte 存储对象，其存储值是两个 Svelte 存储对象之和，如下所示：

```js
const store1 = writable(1);
const store2 = writable(2);
const sum = derived(
  [store1, store2],
  ([$store1, $store2]) => $store1 + $store2
);
```

注意

你可能会注意到，我们在 `derived()` 函数的回调函数中使用了一个以 `$` 前缀的变量；例如，`$store1` 和 `$store2`。然而，它的工作方式与脚本顶层的 `$` 前缀变量不同，后者会自动订阅存储并作为存储值引用。

在这种情况下，这仅仅是一些人用来表示变量用于引用存储值的一种约定。它与任何其他变量名一样，没有更多。

当使用 `derived()` 时，Svelte 只在返回的 Svelte 存储对象（例如，`sum`）被订阅时才会订阅输入的 Svelte 存储对象（例如，`store1` 和 `store2`），当没有更多订阅者时，将取消订阅输入的 Svelte 存储对象。

在我们使用 `derived()` 重新编写 `debounce` 高阶存储之前，让我们先更深入地了解 `derived()` 函数。

`derived()` 函数提供了两种确定返回的 Svelte 存储存储值的方法：同步或异步。

同步方法意味着只要输入 Svelte 存储对象的值发生变化，返回的 Svelte 存储对象的存储值就会同步确定。同样，异步方法意味着它是异步确定的，这意味着存储值可以在输入 Svelte 存储对象的值改变后稍后设置。

我在本节开头展示的两个例子使用了同步方法，其中返回的 Svelte 存储的存储值是在输入 Svelte 存储值改变后立即同步计算并设置的。

当返回的 Svelte 存储对象的存储值通过某些异步操作确定时，异步方法可能很有用。

要确定派生的存储对象是使用同步方法还是异步方法，`derived()` 方法会查看回调函数的函数签名。如果它接受一个参数，则被认为是同步方法。如果它接受两个参数，则被认为是异步方法。为了更好地理解，请查看以下代码片段：

```js
derived(store, ($store) => ...) // synchronous
derived(store, ($store, set) => ...) // asynchronous
```

在这两种情况下，回调函数将在任何输入 Svelte 存储对象的值发生变化时被调用。

如果派生的存储对象使用同步方法，回调函数的返回值将用作返回的 Svelte 存储对象的新存储值。

如果派生存储使用异步方法，则回调函数的第二个参数是一个 `set` 函数。`set` 函数可以用来在任何时候设置返回存储的值。然而，回调函数的返回值将被视为一个清理函数，它将在再次调用回调函数之前立即被调用。

现在我们对 `derived()` 函数更熟悉了，让我们使用 `derived()` 重新编写我们的 `debounce` 高阶存储。

## 使用派生方法

这是使用 `derived()` 方法重写的 `debounced` 存储的更新代码：

```js
function debounce(store) {
  const debounced = derived(store, (value, set) => {
    let timeoutId = setTimeout(() => {
      timeoutId = null;
      set(value);
    }, 200);
    return () => {
      if (timeoutId !== null) clearTimeout(timeoutId);
    };
  });
  return {
    subscribe: debounced.subscribe,
    set: store.set,
  };
}
```

我们不是使用 `writable()` 创建一个新的 `debounced` 存储，而是使用 `derived()` 方法从原始存储中派生它。我们将 `setTimeout` 函数移入 `derived()` 方法回调中。当原始 Svelte 存储有新值时，我们将清除超时并设置一个新的超时。

`derived()` 函数将在订阅 `debounced` 存储时仅订阅一次原始 Svelte 存储，并在没有更多订阅者时取消订阅原始 Svelte 存储。

由于我们将在超时后异步更新 `debounced` 存储，我们将带有两个参数的回调函数传递给 `derived()` 函数。我们调用 `set` 函数在超时后设置 `debounced` 存储的值。如果原始 Svelte 存储的值在超时之前发生变化，则回调函数返回的函数将在再次使用原始 Svelte 存储的更新存储值调用回调函数之前首先被调用。在返回的函数中，我们清除超时，因为我们不再需要它了。

在我们总结之前，还有最后一件事——有时，原始 Svelte 存储可能包含额外的函数，例如在撤销/重做存储中的 `undo()` 和 `redo()` 方法。这些方法也应该在我们的高阶函数返回的 `debounced` 存储中定义。这确保了增强的存储在添加防抖功能的同时，保持了所有相同的方法和行为。你可以在下面的代码片段中看到这个示例：

```js
const value = createUndoRedoStore();
const debouncedValue = debounce(value);
$debouncedValue = 123;
$debouncedValue = 456;
debouncedValue.undo(); // $debouncedValue reverts to 123
```

要返回原始 Svelte 存储的所有方法，我们使用扩展运算符 (`...`) 来扩展原始 Svelte 存储的所有方法：

```js
function debounce(store) {
  const debounced = derived(...);
  return {
    ...store,
    subscribe: debounced.subscribe,
  };
}
```

在这种情况下，我们甚至不需要 `set: store.set`，因为 `set` 方法也会扩展到返回的 Svelte 存储中！

就这些了！在本节中，你学到了另一个技巧——创建一个高阶存储，这是一个接受现有 Svelte 存储并返回一个具有新行为的新的 Svelte 存储的函数。我们不再需要构建一个包含所有逻辑的自定义 Svelte 存储，现在我们可以创建更小、封装良好的高阶存储，并将它们放入你需要的自定义 Svelte 存储中。

## 练习

现在，是时候进行练习了。

上一节中的撤销/重做存储被创建为一个自定义的 Svelte 存储。你能创建一个撤销/重做高阶存储，将任何 Svelte 存储转换为具有两个额外方法`undo()`和`redo()`的撤销/重做存储吗？

下面是一个撤销/重做高阶存储的使用示例：

```js
import { writable } from 'svelte/store';
const originalStore = writable(100);
const undoRedoStore = undoRedo(originalStore);
$undoRedoStore = 42;
undoRedoStore.undo(); // store value now goes back to 5
```

上述代码片段展示了我们如何使用撤销/重做高阶存储`undoRedo`。`undoRedo`函数接收一个存储，命名为`originalStore`，并返回一个基于`originalStore`的新存储，该存储具有撤销/重做功能。例如，如果我们设置了一个新值然后调用`undo`方法，存储的值将恢复到其原始状态，在这个例子中是`5`。

您可以在 GitHub 上找到这个练习的解决方案：

[`github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter09/07-exercise-undo-redo-higher-order-store`](https://github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter09/07-exercise-undo-redo-higher-order-store%0D)

在本节中，我们探讨了使用`derived()`方法创建`debounce`高阶 Svelte 存储的过程。我期待你在实际场景中应用这些知识。

# 摘要

在本章中，我们研究了三个不同的示例，并学习了使用 Svelte 存储的三个不同技术。

我们探讨了如何将任何用户事件转换为 Svelte 存储的数据源，学习了如何从头创建 Svelte 存储，还学习了如何使用内置的`readable()`函数使过程变得更加简单。

之后，我们探讨了创建一个带有额外方法的自定义 Svelte 存储，基于内置的可写存储构建一个新的 Svelte 存储。

最后，我们学习了如何创建一个高阶存储，这是一个接收 Svelte 存储并返回输入存储增强版本的函数。在示例中，我们看到如何将任何 Svelte 存储转换为自身的防抖版本。

通过理解这些技术，你现在可以更有效地在 Svelte 中管理状态，以构建更可扩展和可维护的应用程序。

在下一章中，我们将探讨 Svelte 中的状态管理——具体来说，如何使用 Svelte 存储进行状态管理。
