

# 带有动作的定制事件

动作是 Svelte 最强大的功能之一。

它是封装逻辑和数据到可重用单元的组件的轻量级替代品。它们帮助我们在不同元素上重用相同的逻辑。

虽然组件具有生命周期方法，如 `onMount` 和 `onDestroy`，这些方法在组件内的所有元素添加到或从 DOM 中移除时运行，但动作被设计来处理单个元素的逻辑，仅在特定元素添加到或从 DOM 中移除时运行。

虽然组件可以接收并响应属性变化，但你也可以从父组件向子组件传递数据到动作。当数据发生变化时，动作会做出反应，并且你可以指定动作在数据变化时应该如何反应。

动作简单却非常灵活。你可以用它们做很多事情。在本章和下一章中，我们将探讨动作的一些用例。

动作的一个许多用途案例是管理元素的监听器。事件监听器在 Web 应用程序中非常常见，因为它们允许我们针对用户操作实现特定的行为。这使得我们的应用程序更加互动和动态。因此，将很有趣地看到 Svelte 动作如何帮助我们管理事件监听器。

在本章中，我们将首先探讨如何使用动作帮助管理元素的监听器。我们将通过示例和练习来加强这一概念。

本章包括以下内容：

+   使用动作管理事件监听器

+   使用动作创建自定义事件

# 技术要求

你可以在这里找到本章的项目：[`github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter05`](https://github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter05)

# 定义动作

在我们开始讨论如何使用 Svelte 动作创建自定义事件之前，让我们快速回顾一下如何在 Svelte 中定义动作。

在 Svelte 中，动作不过是一个遵循动作合约的函数。这意味着如果一个函数遵循特定的函数签名，它就被认为是动作。以下是动作的函数签名：

```js
function action(node) {
  return {
    destroy() {}
  };
}
```

这是一个可选返回具有 `destroy` 方法的对象的函数。

在这种情况下，由于 `action` 函数遵循动作合约，它是一个 Svelte 动作。

要在元素上使用 Svelte 动作，你可以使用 `use:` 指令：

```js
<div use:action />
```

在这里，我们在 `div` 元素上使用了名为 `action` 的 Svelte 动作。

那么，带有 Svelte 动作的 `div` 元素会发生什么？

当 `<div>` 元素挂载到 DOM 上时，Svelte 将使用 `<div>` 元素的引用调用 `action` 函数：

```js
const action_obj = action(div);
```

当元素从 DOM 中移除时，Svelte 将调用从调用 `action` 函数返回的对象中的 `destroy` 方法：

```js
if (action_obj && action_obj.destroy) action_obj.destroy();
```

我们可以通过使用参数来自定义 Svelte 动作的行为。我们可以向 `action` 函数传递一个额外的参数：

```js
function action(node, parameter) {
  return {
   update(parameter) {},
   destroy() {},
  };
}
```

此外，你还可以向返回的对象添加另一个方法，`update`，当参数的值发生变化时将被调用：

```js
action_obj.update(new_value);
```

要传递一个额外的参数，你可以使用与 `value` 属性类似的语法来指定它：

```js
<div use:action={value} />
```

现在我们已经知道了如何在 Svelte 中使用和定义动作，让我们看看动作的第一个用例。

# 使用自定义事件重用 DOM 事件逻辑

在我们直接讨论动作之前，让我们看看如何使用 `mousedown` 和 `mouseup` 事件来创建长按行为的一个例子。我们将看到这个简单的例子将如何引导我们进入 Svelte 动作：

```js
<script>
  let timer;
  function handleMousedown() {
    timer = setTimeout(() => {
      console.log('long press!');
    }, 2000);
  }
  function handleMouseup() {
    clearTimeout(timer);
  }
</script>
<button
  on:mousedown={handleMousedown}
  on:mouseup={handleMouseup}
/>
```

在前面的例子中，我们尝试在按钮中实现长按行为。想法是按住按钮超过两秒钟，然后执行某些操作。当我们检测到是长按时，我们在控制台中打印 `'long press!'`。

要实现长按行为，我附加了两个事件监听器：`mousedown` 和 `mouseup`。这两个事件监听器协同工作。`mousedown` 使用 `setTimeout` 开始计时两秒钟，而 `mouseup` 使用 `clearTimeout` 清除计时。如果用户没有足够长时间地按住按钮，超时就不会被触发，就不会被视为长按。请注意，为了在两个事件监听器之间协调计时器，`timer` 变量在事件监听器之间是共享的。

如你所见，要实现长按行为，你需要两个事件监听器和共享的一个变量。

现在，如果你需要在另一个按钮上实现这个长按行为怎么办？

你实际上不能共享相同的事件监听器和变量，因为你可能希望将不同的持续时间视为超时，或者在长按发生时有不同的行为。

因此，你必须再次声明它们，并记住将正确的事件与正确的事件监听器配对。

## 将逻辑封装到组件中

重新创建不同长按按钮的一种方法是将其封装为一个组件，将所有长按按钮逻辑放入组件中，并作为重用逻辑的手段重用组件。

当你定义一个组件时，你也在定义组件中的逻辑以及元素。这意味着如果我们把长按逻辑和 `button` 元素都放入组件中，我们必须与 `button` 元素一起使用长按逻辑，而不能使用其他元素。

如果你想要自定义元素，比如使用不同的元素、不同的样式、显示不同的文本内容或添加更多的事件监听器，你必须定义样式、文本内容或事件监听器作为组件的属性，并将它们传递到组件中的 `button` 元素：

```js
<!-- LongPressButton.svelte -->
<button
  // besides the longpress behavior
  on:mousedown={handleMousedown}
  on:mouseup={handleMouseup}
  // you need to pass down props as attributes
  {...$$props}
  // and also forward events up
  on:click
  on:dblclick
>
  <slot />
</button>
```

在前面的代码中，我们将从 props 中传递的额外属性以及从按钮元素中转发两个事件（`click`和`dblclick`）到组件中。

我在这里试图说明的是，如果你希望通过组件重用事件监听器逻辑，你将发现自己必须处理与元素一起在组件中出现的其他属性。

我们可以用组件做更多的事情，但如果我们只是尝试重用长按行为，那么通过在组件中定义它来重用它就有点过度了，而且它可能会很快变得难以管理。

那么，我们还有其他什么选择？

## 将逻辑封装到动作中

使用动作封装长按行为是一个更好的选择。

让我们就这样做，然后我会解释为什么使用动作是一个更好的方法：

```js
function longPress(node) {
  let timer;
  function handleMousedown() {
    timer = setTimeout(() => {
      console.log('long press!');
    }, 2000);
  }
  function handleMouseup() {
    clearTimeout(timer);
  }
  node.addEventListener('mousedown', handleMousedown);
  node.addEventListener('mouseup', handleMouseup);
  return {
    destroy() {
     node.removeEventListener('mousedown', handleMousedown);
     node.removeEventListener('mouseup', handleMouseup);
    }
  }
}
```

定义了动作（如前述代码所示），我们可以在多个元素上使用这个动作：

```js
<button use:longPress>Button one</button>
<button use:longPress>Button two</button>
```

你可以将动作应用于不同类型的元素：

```js
<span use:longPress>Hold on to me</span>
```

你也可以将它与其他属性或事件监听器一起使用：

```js
<button use:longPress class="..." on:click={...} />
```

我希望你能看到，`longPress`动作只封装了长按行为。与`LongPressButton`组件不同，`longPress`动作可以很容易地在任何元素中重用。

所以，作为一个经验法则，当你从元素中抽象逻辑时，如果你需要与元素一起抽象它，使用组件是可以的。但如果你只需要从元素中抽象逻辑行为，请使用动作。

动作是抽象元素级逻辑的强大工具。但仍然缺少一个拼图的一部分：我们应该如何为不同的元素添加不同的长按处理程序？我们将在下一部分看到。

## 向动作传递参数

那么，我们如何自定义`longPress`动作的行为呢？

由于你在前面的部分已经看到了它，你可能已经猜到了答案。我们可以通过向动作传递参数来自定义动作的行为。

例如，如果我们想在不同的`button`元素上处理长按动作，我们可以通过动作参数传递一个不同的函数：

```js
<button use:longPress={doSomething1} />
<button use:longPress={doSomething2} />
```

然后，我们将在动作函数的第二个参数中接收到这个函数：

```js
function longPress(node, fn) {
  // ...
}
```

当被视为长按时，我们调用传递的函数：

```js
fn();
```

我们可以类似地传递其他参数，例如，将持续时间考虑为长按。

在 Svelte 动作中，你只能传递一个参数。要传递多个参数，你必须将它们转换成一个对象，并将它们作为一个参数传递。

在我们的`longPress`动作中，我们想要传递当检测到`longPress`动作时要调用的函数，以及被视为长按的持续时间，到`longPress`动作中。为了传递这两个值，我们创建一个包含它们作为对象值的对象，并将该对象作为动作参数传递：

```js
<button
  use:longPress={{
    onLongPress: doSomething1,
    duration: 5000
  }} />
```

对象中的某些参数可能是可选的，因此在操作中读取它们时，我们可能需要提供默认值：

```js
function longPress(node, { onLongPress, duration = 1000 }) {
  // if not specified, the default duration is 1s
}
```

在这个阶段，你可能想知道，`onLongPress` 是否也可以是可选的？

在我们的情况下，这没有太多意义，因为我们的操作的主要目标是检测长按并调用 `onLongPress` 函数。

然而，这是一个好问题。

在其他一些操作中，你可能会有可选的函数处理程序。

例如，如果我们有一个可以检测元素上不同手势的 `gesture` 操作，那么每个 `gesture` 回调函数都是可选的，因为你可能只对其中的一种手势感兴趣：

```js
function gesture(node, { onSwipe, onDrag, onPinch }) { }
```

在这个 `gesture` 操作中，`onSwipe`、`onDrag` 和 `onPinch` 函数处理程序是可选的，可能是未定义的。

我们不应该创建一个空函数作为后备，而应该在使用之前检查函数是否已定义：

```js
if (typeof onSwipe === 'function') onSwipe();
```

这样，我们就不必创建不必要的函数。

然而，当你有多个回调函数并且需要检查它们是否已定义后再调用它们时，这会变得复杂。

有没有更好的处理方式？

是的，确实有。

实际上，在 Svelte 中，有一个更符合习惯的方式，当发生某些操作时，可以通过派发一个自定义事件来通知或调用一个函数。

例如，要确定用户是否长按了 `<button>` 元素，自然是在 `<button>` 元素上监听 `'longpress'` 自定义事件：

```js
<button on:longpress={doSomething1} />
```

然而，没有名为 `'longpress'` 的原生事件。

但别担心，我们可以创建一个自定义的 `'longpress'` 事件。

幸运的是，我们有 `longPress` 操作来确定用户何时长按按钮。要创建自定义的 `'longpress'` 事件，我们需要在 `longPress` 操作中确定用户长按按钮后触发 `'longpress'` 事件。因此，在确定用户正在长按按钮的代码行中，我们可以从按钮中派发一个自定义事件：

```js
node.dispatchEvent(new CustomEvent('longpress'));
```

让我们监听 `'longpress'` 自定义事件，并在我们的操作中创建自定义的 `'longpress'` 事件。以下是最终的代码：

```js
<script>
  function longPress(node, { duration = 1000 } = {}) {
    let timer;
    function handleMousedown() {
      timer = setTimeout(() => {
        node.dispatchEvent(new CustomEvent('longpress'));
      }, duration);
    }
    function handleMouseup() {
      clearTimeout(timer);
    }
    node.addEventListener('mousedown', handleMousedown);
    node.addEventListener('mouseup', handleMouseup);
    return {
      destroy() {
        node.removeEventListener('mousedown', handleMousedown);
        node.removeEventListener('mouseup', handleMouseup);
     }
    }
  }
</script>
<button longPress action to the button element, which adds logic to determine whether the button is being long-pressed. When the button is long-pressed, the longPress action dispatches a custom event called 'longpress' on the element. To react to and trigger specific behaviors when the custom 'longpress' event is dispatched on the element, we can listen to the event by using on:longpress with an event handler.
It may feel like a roundabout way to call a function from an action by dispatching an event, but doing it this way has a few pros:

*   Whether we listen to the `'longpress'` event on the button or not, the action could still dispatch the `'longpress'` custom event. So, with this approach, we don’t need to check whether the handler is defined or not.
*   Listening to the `'longpress'` event using `on:` instead of passing the function directly into the action would mean that you could use other Svelte features that come with Svelte’s `on:` directive. For example, to only listen to the `'longpress'` event once, you can use the `|once` event modifier, for example, `on:longpress|once`.

Another way of describing what we have done with the `longPress` action is that the `longPress` action enhances the button element and provides a new event, `'longpress'`, that can be listened to on the element.
Now that we’ve learned how we can define Svelte actions, and how we can use actions to create new events that we can listen to on an element, let’s look at a few more examples that use this technique.
Example – validating form inputs with custom events
The example that we are going to explore is using actions to validate form inputs.
When you add an input element to your form, you can add attributes such as `required`, `minlength`, and `min` to indicate that the input value has to pass the constraint validation or else would be considered invalid.
However, by default, such a validation check is only done during form submission. There’s no real-time feedback on whether your input is valid as you type.
To make the input element validate as you type, we need to add an `'input'` event listener (which will be called on every keystroke as we type in the input element) and call `input.checkValidity()` to validate the input. Now, let’s do just that:

```

<input on:input={(event) => event.target.checkValidity()} />

```js

 As you call the `checkValidity()` method, if the input is indeed invalid, then it will trigger the `'``invalid'` event:

```

<input

on:input={(event) => event.target.checkValidity()}

on:invalid={(event) => console.log(event.target.validity)}

/>

```js

 Unfortunately, there’s no `'valid'` event. So, there’s no way to tell whether the input has passed the validation.
It would be great if there were an event called `'validate'` in which within the event details, we can tell whether the input is valid or not. If it isn’t, we could get an error message about why the input is invalid.
Here’s an example of how we could use the `'``validate'` event:

```

<input on:validate={(event) => {

if (event.detail.isValid) {

errorMessage = '';

} else {

errorMessage = event.detail.errorMessage;

}

}} />

```js

 There isn’t an event called `'validate'`, but we can create this custom event ourselves. So, why not create an action to create this custom event for us?
This logic is well suited to be written as an action for the following reasons:

*   It can be reused in other input elements.
*   This logic itself is not involved in creating or updating elements. If it were, it would probably be better suited to be a component.

Let us write this action:

1.  Firstly, this action involves listening to the `'input'` event listener. So, in the code, as shown, we are going to add an event listener at the start of the action and remove the `'input'` event listener in the `destroy` method. This means that whenever an element that uses this action is added to the DOM, it will listen to the `'input'` event, and when it is removed from the DOM, the event listener will be automatically removed:

    ```

    function validateOnType(input) {

    input.addEventListener('input', onInput);

    return {

    destroy() {

    input.removeEventListener('input', onInput);

    }

    };

    }

    ```js

     2.  Next, within the input handler, we are going to call `checkValidity()` to check whether the input is valid. If the input is invalid, then we will read `input.validity` and determine the error message:

    ```

    function validateOnType(input) {

    function onInput() {

    const isValid = input.checkValidity();

    const errorMessage = isValid ? '' : getErrorMessage(input.validity);

    }

    // ...

    }

    ```js

     3.  Finally, we will dispatch the custom `'validate'` event with `isValid` and `errorMessage` as event details:

    ```

    function validateOnType(input) {

    function onInput() {

    // ...

    input.dispatchEvent(

    new CustomEvent(

    'validate',

    { detail: { isValid, errorMessage } }

    )

    );

    }

    // ...

    }

    ```js

     4.  Now, with this action completed, we can enhance the `<input>` element by adding a new `'validate'` event, which will be called `as-you-type`, letting you know whether the input is currently valid or invalid. It will also show the corresponding error message:

    ```

    <input

    use:validateOnType

    当：validate={(event) => console.log(event.detail)}

    />

    ```js

    To make the validation results more useful to the user, you can use the result from the `'validate'` event to modify element styles, such as setting the input border color to red when the validation result is invalid, as shown in the following snippet:

    ```

    <script>

    let isValid = false;

    </script>

    <input

    类：invalid={!isValid}

    使用：validateOnType

    当：validate={(event) => isValid = event.detail.isValid}

    />

    <style>

    .invalid { border: red 1px solid; }

    </style>

    ```js

Are you getting the hang of writing actions that create custom events?
Let’s try the next one as an exercise. This time, we’ll tackle one of the most common user interactions, drag and drop.
Exercise – creating a drag-and-drop event
A drag-and-drop behavior means clicking on an element, moving the mouse while holding down the click to drag the element across the screen to the desired location, and then releasing the mouse click to drop the element in the new location.
A drag-and-drop behavior thus involves coordination between multiple events, namely, `'mousedown'`, `'mousemove'`, and `'mouseup'`.
As we perform the drag-and-drop motion, what we are interested in knowing is when the dragging starts, how far the element is dragged, and when the dragging ends.
These three events can be translated into three custom events: `'dragStart'`, `'dragMove'`, and `'dragEnd'`.
Let us try to implement the drag-and-drop behavior as an action that will create these three custom events:

```

<div

使用：dragAndDrop

当：dragStart={...}

当：dragMove={...}

当：dragEnd={...}

/>

```js

 You can check the answer to this exercise here: [`github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter05/03-drag-and-drop`](https://github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter05/03-drag-and-drop).
Summary
In this chapter, we saw how to define an action. We talked about one of the common patterns of actions, which is to create custom events. This allows us to encapsulate DOM event logic into custom events and reuse them across elements.
In the next chapter, we will look at the next common pattern of actions, which is integrating third-party libraries.

```
