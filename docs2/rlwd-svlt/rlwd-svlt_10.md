

# 使用 Svelte 存储进行状态管理

每个用户界面控件都有一个状态。复选框有一个选中/未选中的状态。文本框的状态是其当前的输入值。表格的状态是显示的数据和当前排序的列。有时，当屏幕上同时存在多个用户界面控件时，你需要同步它们的状态——这就是状态管理的作用所在。

在本章中，我们将讨论使用 Svelte 存储进行状态管理。我们将从为什么应该使用 Svelte 存储开始，然后讨论使用 Svelte 存储进行状态管理时的一些技巧。

接下来，我们将探讨使用状态管理库的主题。我们将讨论为什么以及如何使用它们。在此基础上，我们将通过几个示例展示如何通过 Svelte 存储将第三方状态管理库集成到 Svelte 中。

本章包括以下主题的章节：

+   使用 Svelte 存储管理状态

+   在 Svelte 中使用状态管理库

# 技术要求

本章中的代码可以在 GitHub 上找到：[`github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter10`](https://github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter10)。

# 使用 Svelte 存储管理状态

当构建交互式用户界面时，我们首先考虑的是确定表示各种组件和交互所需的状态。

例如，在下面的代码片段中，我有一个包含几个组件的登录表单，包括两个输入框、一个复选框和一个按钮：

```js
<Input name="username" />
<Input name="password" />
<Checkbox label="Show Password" />
<Button>Submit</Button>
```

每个 Svelte 组件都有多个状态，如下所述：

+   `<Input />` 组件有一个输入值状态和验证期间设置的错误状态

+   `<Checkbox />` 组件有一个选中/未选中的状态，选中时会在输入框中显示密码

+   密码 `<Input />` 组件有一个额外的状态来显示/隐藏密码

+   `<Button />` 组件有一个启用/禁用状态，当表单不完整时禁用

这些状态中的每一个都可以用一个变量来表示。这些变量本身可以是相互关联的。

让我们看看这些变量如何相互关联。例如，当复选框被选中时，密码输入需要显示密码。当姓名和密码输入被填写时，按钮需要变为启用状态。当按钮被按下时，执行验证，并相应地更新输入的错误状态。

在组件之间管理多个状态以形成一个连贯体验的技艺被称为状态管理。

将所有这些状态分组在一起并作为一个整体来管理，会比分别管理单个状态更清晰、更容易。在下面的代码片段中，我将上述所有变量组合成一个变量对象：

```js
const state = {
  nameValue: '',
  nameError: null,
  passwordValue: '',
  passwordError: null,
  revealPassword: false,
  submitDisabled: true,
}
```

在这里，你可以看到相关状态集中在一个地方，这使得更容易查看并判断是否有错误发生。

为了说明这一点，让我们假设`nameValue`和`submitDisabled`变量被保留为两个单独的组件中的两个单独的变量；在`Input`组件中，我们有`nameValue`变量：

```js
<!-- Input.svelte -->
<script>
  export let nameValue = '';
</script>
<input bind:value={nameValue} />
```

在`Button`组件中，我们有一个`submitDisabled`变量：

```js
<!-- Button.svelte -->
<script>
  export let submitDisabled = false;
</script>
<button disabled={submitDisabled}>Submit</button>
```

从代码中，你可以看到很难直接判断`nameValue`和`submitDisabled`变量是否相关。如果某个状态没有按预期更新，你没有一个方便的地方可以同时检查这两个状态；你将不得不在每个单独的组件中检查它们。

例如，如果`<Button />`组件中的`submitDisabled`状态在输入名称后没有改变为`false`以启用按钮，你将需要在一个单独的组件（`<Input />`组件）中找到并检查`nameValue`变量。

如果，相反，状态被组合成一个`state`对象，那么你可以检查`state`对象并查看`nameValue`和`submitDisabled`属性是否设置正确。看看下面的例子：

```js
const state = {
  nameValue: '',
  submitDisabled: true,
};
```

既然我们已经确定应该将多个相关状态组合成一个状态对象，接下来的问题是：如果我们可以使用简单的 JavaScript 对象来组合状态，为什么还需要使用 Svelte 存储来组合状态呢？

好吧，事实是，这并不总是必要的。在某些情况下，使用简单的 JavaScript 对象来组合状态可以同样有效。然而，Svelte 存储提供了额外的功能和好处，可以增强你的状态管理体验。

正如我们在*第八章*中探讨的那样，Svelte 存储非常有用，因为它可以在组件之间传递，并且其更新可以在组件之间传播。然后在*第九章*中，我们看到了 Svelte 存储是多么有用，它能够封装逻辑在 Svelte 存储中，以及能够定义数据逻辑紧挨着数据本身。

因此，为了本章的目的，我们将继续使用 Svelte 存储将多个相关状态组合成一个状态对象。以下是如何看起来的一段简短代码片段：

```js
<script>
  import { writable } from 'svelte/store';
  const state = writable({
    nameValue: '',
    nameError: null,
    passwordValue: '',
    passwordError: null,
    revealPassword: false,
    submitDisabled: true,
  });
</script>
```

这里，我们使用`writable()`存储来创建状态对象，因为我们将会修改状态对象。

状态对象可以在 Svelte 组件内部或外部定义。由于我们的状态对象是一个 Svelte 存储，我们可以将状态对象导入到任何组件中，并且任何对状态对象的更新都将传播到所有使用状态对象的 Svelte 组件——所以，状态对象在哪个文件中定义并不重要。

如果我们在 Svelte 组件中定义我们的状态对象，那么我们可以通过 props 将状态对象传递给组件的子组件。以下是一个这样的例子：

```js
<script>
  const state = writable({ name: 'Svelte' });
</script>
<Input {state} />
```

另一方面，如果我们在一个 JavaScript 模块中定义我们的状态对象，那么我们可以将我们的状态对象导入任何将使用状态对象的 Svelte 组件。以下是将 Svelte 存储状态对象导入 Svelte 组件的代码片段：

```js
<script>
  // import state object defined in './state.js'
  import { state } from './state.js';
</script>
```

在 *第八章* 中，我们学习了如何读取、订阅和更新 Svelte 存储的内容。无论存储在 Svelte 存储中的数据是状态对象还是其他任何东西，在读取、订阅和更新 Svelte 存储状态对象的操作上都没有区别，就像任何其他 Svelte 存储一样。

但是，随着状态在增长的项目中使用频率越来越高，你将在 `state` 存储对象内部拥有更多的状态值。随着 `state` 存储对象变得更大和更复杂，你的应用程序可能会变慢，你可能会有优化/改进它的需求。因此，我将以下小节专门用于分享我关于使用 Svelte 存储进行状态管理的技巧和观点。

当然，你不应该盲目地应用所有这些方法。正如著名计算机科学家唐纳德·克努特所说，“*过早优化是万恶之源*”——你应该衡量和评估以下任何一条建议是否适用。随着我介绍这些技巧，我会解释它们是什么，为什么以及何时它们是有用的。

## 小贴士 1 – 使用单向数据流简化复杂的状态更新

当你的应用程序变得更加复杂，有多个组件和多个更新状态的地方时，管理状态更新变得更加具有挑战性。

你如何知道管理状态更新变得难以控制？你可能会遇到以下情况：

+   应用程序的状态发生变化，而你发现很难追踪状态变化的原因

+   你更改了某个状态值，但无意中，这导致其他看似无关的状态值也进行了更新，而你很难找出原因

这些迹象表明状态更新已经变得复杂且难以管理。在这种情况下，拥有单向数据流可以帮助简化状态管理。

使用单向数据流，状态更新以单一方向进行管理，数据流可以轻松追踪和调试。

有一些状态管理库，如 Redux、MobX 和 XState，可以帮助强制执行单一数据流并使你能够对状态变化进行推理。你可能对如何使用这些状态管理库与 Svelte 存储感兴趣；我将在本章的后面部分介绍它们，并将使用其中一个库作为示例。

## 小贴士 2 – 相比于一个大型的状态对象，更倾向于使用较小的自包含状态对象

In Svelte, when a state object changes, all components that use that state object are updated, even if the specific state value that a component uses did not change. This means that if a state object becomes larger and more complex, with many unrelated state values, updating that state object can trigger unnecessary updates to many components in the application.

Here, I am going to use a product details page of an e-commerce web application as an example. In a typical e-commerce web app, you have a page that shows the details of a product. You will see other information, such as the shopping cart and product reviews, on the same page as well.

If I use one state object for all the information on the page, it may look like this:

```js
<script>
  let state = {…}
</script>
<ShoppingCart cart={$state.cart} />
<ProductDetails product={$state.product} />
<ProductRatings ratings={$state.ratings} />
```

In the preceding code, we use the same state object, `$state`, across three components: `ShoppingCart`, `ProductDetails`, and `ProductRatings`. When any part of the `$state` state object changes, such as changing `$state.cart`, all three components will be triggered to update.

This is undesirable—multiple components updating unnecessarily could lead to slower performance and a less responsive user interface.

So, it is recommended to split big state objects into smaller state objects. Using the same example, that would mean splitting the `$state` state object into three smaller state objects, like so:

```js
<script>
  let cartState = {…};
  let productState = {…};
  let ratingState = {…};
</script>
<ShoppingCart cart={$cartState} />
<ProductDetails product={$productState} />
<ProductRatings ratings={$cartState, would not trigger an update on the <ProductDetails /> and <ProductRatings /> components.
So, if you have big state objects and you find components update unnecessarily and the performance of your application is impacted by it, then consider breaking these big state objects down into smaller state objects.
But what if the state object is still big, yet the different values in the state objects are closely related and you are unable to break it apart into smaller state objects? Well, there is still hope, which leads us to our third tip.
Tip 3 – derive a smaller state object from a bigger state object
If state values are related to each other and so you can’t break a big state object into smaller state objects, we can create smaller state objects that derive from the big state object. Let me show you some code to explain this clearly.
Let’s say that you have a state object, `userInfo`, that has two closely related state values, `$userInfo.personalDetails` and `$userInfo.socials`, as shown here:

```

import { writable } from 'svelte/store';

const userInfo = writable({

personalDetails: {...},

socials: {...},

});

```js

 You may realize that one part of the `userInfo` state object doesn’t change as often as the other. But whenever any part of `userInfo` changes, all the components that use either the `$userInfo.personalDetails` or `$userInfo.socials` state values will be updated.
To ensure that components using only `$userInfo.socials` are updated exclusively when `$userInfo.socials` changes, one way would be to break the state object into smaller, more focused state objects, like so:

```

import { writable } from 'svelte/store';

const userPersonalDetails = writable({...});

const userSocials = writable({...});

```js

 As you can see, you now have two separate state objects, `userPersonalDetails` and `userSocials`.
But this would mean that places where you previously updated the `userInfo` state object would have to change since `userInfo` is now split into two separate state objects.
Here is how you would change the code:

```

// previously

$userInfo = newUserInfo;

// now

$userPersonalDetails = newUserInfo.personalDetails;

$userSocials = newUserInfo.socials;

```js

 So, the question now is this: Is there an alternative to not having to change this, yet being able to update the components that use `$userInfo.socials` only when `$``userInfo.socials` changes?
I believe I’ve leaked the answer already. The alternative is to derive a new state object. In the following code snippet, I am deriving a new `userSocials` state object from the `userInfo` state object:

```

import { derived } from 'svelte/store';

const userSocials = derived(userInfo, $userInfo => $userInfo.socials);

```js

 The component that uses `$userSocials` will only update whenever the `userSocial` state changes. The `userSocial` state changes only when the `$userInfo.socials` changes. So, when the component uses `$userSocials` instead of `$userInfo.socials`, it will not update when any other part of the `userInfo` state object changes.
I believe that seeing is believing, and it will be much clearer to see and interact with an example to get this idea forward. So, I’ve prepared some demo examples at [`github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter10/01-user-social`](https://github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter10/01-user-social), and you can try them out and see what I mean.
Let’s quickly recap the three tips:

*   Whenever the state update logic is complex and convoluted, introduce some state management libraries to enforce simpler and unidirectional data flows
*   If the state object gets too big, and state changes update more components than needed, break the state object into smaller state objects
*   If you can’t split a Svelte store state object, consider deriving it into a smaller state object

So, we’ve gone through my three general tips for managing complex stores in a Svelte application; let’s now elaborate more on my first tip on how to use state management libraries with Svelte.
Using state management libraries with Svelte
If you google *State management library for frontend development*, at the time of writing, you will get list after list of libraries, such as Redux, XState, MobX, Valtio, Zustand, and many more.
These libraries have their own take on how states should be managed, each with different design considerations and design constraints. For the longevity of the content of this book, we are not going to compare and analyze each of them since these libraries will change and evolve over time and potentially be replaced by newer alternatives.
It is worth noting that some of the state management libraries are written for a specific web framework. For example, at the time of writing, the Jōtai library ([`jotai.org/`](https://jotai.org/)) is written specifically for React, which means you can only use Jōtai if you write your web application in React.
On the other hand, there are framework-agnostic state management libraries. An example is XState ([`xstate.js.org/`](https://xstate.js.org/)), which can be used by any web framework as the XState team has created packages such as `@xstate/svelte` to work with Svelte, `@xstate/react` for the React framework, `@xstate/vue` for Vue, and many more.
While the `@xstate/svelte` package is tailored for seamless integration of XState in Svelte, not all state management libraries offer this level of compatibility. Nevertheless, there are several other state management libraries that you can use in Svelte, and integrating them is straightforward. In fact, I will provide some examples to showcase how simple it is to integrate these libraries in Svelte and utilize Svelte’s first-class capabilities for working with stores.
One such state management library that works seamlessly with Svelte is Valtio ([`github.com/pmndrs/valtio`](https://github.com/pmndrs/valtio)), a minimalist library that turns objects into self-aware proxy states. We are going to explore how we can use Valtio in Svelte, by turning Valtio’s proxy state into a Svelte store and using the Svelte store’s `$`-prefixed syntax to subscribe to the proxy state changes and access the proxy state value.
Example – using Valtio as a Svelte store
Before we start talking about how to use Valtio in Svelte, let’s look at how to use Valtio on its own.
Using Valtio
Valtio turns the object you pass to it into a self-aware proxy state. In the following code snippet, we create a Valtio state named `state` through the `proxy()` method:

```

import { proxy } from 'valtio/vanilla';

const state = proxy({ count: 0, text: 'hello' });

```js

 To update the proxy state, you can make changes to it the same way you would to a normal JavaScript object. For example, we can increment the value of `state.count` by mutating the value directly, as follows:

```

setInterval(() => {

state.count++;

}, 1000);

```js

 To be notified of modifications in the proxy state, we can use Valtio’s `subscribe` method, as illustrated here:

```

import { subscribe } from 'valtio';

const unsubscribe = subscribe(state, () => {

console.log('state object changed');

});

```js

 In this case, every time we modify the state, the callback function passed into the `subscribe` method will be called, and we will see a new log in the console, printing `'state` `object changed'`.
Valtio also allows you to subscribe only to a portion of a state. For example, in the following snippet, we subscribe only to the changes made to `state.count`:

```

const unsubscribe = subscribe(state.count, () => {

console.log('state count changed');

});

```js

 Since in this case, we are subscribing only to the changes made to `state.count`, then modifying `state.text` would not see a `'state count changed'` log added to the console as the change to `state.text` is not subscribed.
The Valtio proxy state is meant for tracking update changes. To read the latest value of the proxy state, we should use the `snapshot()` method to create a snapshot object, as follows:

```

import { subscribe, snapshot } from 'valtio';

const unsubscribe = subscribe(state, () => {

const obj = snapshot(state);

});

```js

 Using Valtio as a Svelte store
Now that we’ve learned about basic Valtio operations and concepts, let’s create a simple counter application to see how we can use Valtio in Svelte. Firstly, we create a proxy state for our counter application:

```

// filename: data.js

import { proxy } from 'valtio';

export const state = proxy({ count1: 0, count2: 0 });

```js

 Here, I am creating two counters, `count1` and `count2`, which will later allow us to experiment with subscribing to a specific portion of the proxy state. This way, we can observe whether our application updates only when one of the counters changes.
Also, we are creating the proxy state in a separate file, `data.js`, rather than declaring it inside a Svelte component; this way, we can import the state separately in each Svelte component later on.
In addition, let’s create two functions to increment the counters:

```

export function increment1() {

state.count1++;

}

export function increment2() {

state.count2++;

}

```js

 Now, let’s import the proxy state into our Svelte component; I have created two buttons to increment each counter separately:

```

<script>

import { state, increment1, increment2 } from './data.js';

</script>

Count #1: {state.count1}

Count #2: {state.count2}

<button on:click={increment1}>Increment 1</button>

<button on:click={increment2}>Increment 2</button>

```js

 If you try clicking on the buttons, you’ll realize that they are not working as expected— the counters are not incrementing.
However, if you add the following code to `data.js` and click on the button, the console will print out the current value of the counters, indicating that the counters are incrementing successfully:

```

import { subscribe } from 'valtio';

subscribe(state, () => {

console.log(state);

});

```js

 As you can see, the counters are updating as expected. The issue, therefore, does not lie in the inability to increment the counters, but rather in the failure to display the changes on the screen. It is possible that Svelte is not recognizing the changes to the counters, and therefore, it is not updating the elements to show the latest value of the counters.
So, what can we do to make Svelte aware of the changes to Valtio’s proxy state?
One approach would be to use a Svelte store, as it provides a mechanism for Svelte components to react to changes in data. We can turn Valtio’s proxy state into a Svelte store. Then, by subscribing to the store, Svelte will be aware of the changes to the state and will update the elements accordingly.
This approach of converting states from other state management libraries into a Svelte store in order to take advantage of Svelte’s built-in update mechanism is very common. It allows developers to use their preferred state management solution while still taking advantage of Svelte’s reactive capabilities.
So, let’s see how we can turn Valtio’s proxy state into a Svelte store. To start off, I am creating a function called `valtioStateToSvelteStore`:

```

function valtioStateToSvelteStore(proxyState) {

return {};

}

```js

 Before creating the Svelte store from the Valtio proxy state, let’s have a quick recap on what the Svelte store contract is. The Svelte store contract dictates that a Svelte store should have a `subscribe` method and an optional `set` method. The `subscribe` method takes a callback function as its only parameter, which will be called whenever the store’s value changes, and returns an `unsubscribe` function to stop further notifications.
So, let’s define the `subscribe` method in the returned object of the `valtioStateToSvelteStore` function, like so:

```

function valtioStateToSvelteStore(proxyState) {

return {

subscribe(fn) {

},

};

}

```js

 The initial value of the Svelte store can be defined by calling the callback function synchronously with the value of the proxy state:

```

function valtioStateToSvelteStore(proxyState) {

return {

subscribe(fn) {

fn(snapshot(proxyState));

},

};

}

```js

 Our next step is to subscribe to changes in the Valtio proxy state:

```

function valtioStateToSvelteStore(proxyState) {

return {

subscribe(fn) {

fn(snapshot(proxyState));

return subscribe(proxyState, () => {

fn(snapshot(proxyState));

});

},

};

}

```js

 Based on the Svelte store contract, within the `subscribe` method, we need to return a function to unsubscribe the callback function from the Svelte store. In our code, the reason we return the return value of the `subscribe` function from Valtio in our `subscribe` method is that the return value of Valtio’s `subscribe` function is a function to unsubscribe from the proxy state.
A function to unsubscribe from changes is just what we need. Isn’t this convenient?
It’s no coincidence that most state management libraries provide methods for subscribing to state changes and unsubscribing from them, just like what we need to define a Svelte store. This is because both Svelte stores and state management libraries are designed based on the Observer pattern we discussed in *Chapter 8*. In summary, to turn a state management library’s state into a Svelte store, we need to understand how the library works and how its APIs translate into the Svelte store contract.
Now that we have a function to transform a Valtio’s proxy state into a Svelte store, let’s try to use it by running the following code:

```

<script>

import { state, increment1, increment2, valtioStateToSvelteStore } from './data.js';

const store = valtioStateToSvelteStore(state);

</script>

计数 #1: {$store.count1}

计数 #2: {$store.count2}

```js

 Clicking on any button in the component, you will see that the counter works perfectly fine now.
So, this is how we turn a Valtio proxy state into a Svelte store.
The next thing I would like to explore is creating a Svelte store that subscribes only to partial updates of the Valtio proxy state. By selecting only a specific portion of the state to monitor, we can ensure that the Svelte store updates only when a particular part of the state changes.
Before we do that, let’s add a few more lines to show you what I mean:

```

<script>

// ...

$: console.log('count1: ', $store.count1);

$: console.log('count2: ', $store.count2);

</script>

```js

 As you click on either of the increment buttons, you will notice that both reactive statements are called, indicating that `$store` is updated whenever either `count1` or `count2` is updated.
As discussed in the third tip earlier in the chapter, if state changes cause unnecessary code to run, we can derive a smaller state from the original state to only subscribe to partial updates. So, let’s do that:

```

<script>

import { state, increment1, increment2, valtioStateToSvelteStore } from './data.js';

const count1 = valtioStateToSvelteStore(state.count1);

const count2 = valtioStateToSvelteStore(state.count2);

$: console.log('count1: ', $count1);

$: console.log('count2: ', $count2);

</script>

```js

 Here, instead of turning `state` into a Svelte store, we are turning `state.count1` into a Svelte store. This allows us to create separate Svelte stores that only subscribe to a portion of the proxy state.
This should work, but unfortunately, it doesn’t. The reason for this has nothing to do with our code but with the data structure of our state. `state.count1` is a primitive value, which Valtio is unable to subscribe to.
To work around this, I’m going to change the data type of `state.count1` from a primitive value to an object:

```

export const state = proxy({

count1: { value: 0 },

count2: { value: 0 },

});

export function increment1() {

state.count1.value ++;

}

```js

 In the preceding code snippet, we changed `state.count1` to an object with a property called `value`. We still keep the code of deriving the `count1` Svelte store from the `state.count1` proxy state. So, now, the derived Svelte store of `count1` would be an object, and to get the value of the count, we will be referring to `$count1.value` instead of `$count1`:

```

$: console.log('count1: ', count1. 相反，当你点击另一个按钮，标记为 count2。只有当相应的计数器增加时，响应式语句才会运行。这是因为我们的 count1 和 count2 存储只在 Valtio 代理状态的一个特定部分发生变化时更新。

那么，让我们总结一下到目前为止我们所做的工作。

我们通过将 Valtio 代理状态转换为 Svelte 存储，探索了 Valtio 在 Svelte 中的集成。一步一步地，我们通过利用 Valtio 内置的订阅和取消订阅方法，实现了 Svelte 存储合约。在最后，我们探讨了如何订阅代理状态的局部更新，以最小化不必要的响应性。

我相信你现在一定迫不及待想要亲自尝试一下，那么我们就用下一个状态管理库来做练习。

练习 – 将 XState 状态机转换为 Svelte 存储

XState ([`xstate.js.org/`](https://xstate.js.org/)) 是一个用于使用状态机管理状态的 JavaScript 库。在这个练习中，你将把一个 XState 状态机转换成一个 Svelte 存储。

XState 提供了一个名为 `@xstate/svelte` 的包，其中包含专门为在 Svelte 中集成 XState 设计的实用工具。尽管这个包可以为你的任务提供灵感，但在这个练习中，使用 `@xstate/svelte` 是不允许的。这里的目的是挑战你实现一个将 XState 状态机转换为 Svelte 存储的功能，而不使用现有的集成实用工具。

要开始这个练习，请按照以下步骤操作：

1.  **理解 XState**：如果你还不熟悉 XState，花些时间阅读其文档并尝试其功能。了解状态机的工作原理以及 XState 如何实现它们。

1.  **创建状态机**：使用 XState 创建一个你想要转换为 Svelte 存储的状态机。这可能是一个具有几个状态和转换的简单机，或者如果你感到舒适，也可以是更复杂的机。

1.  **将状态机转换为 Svelte 存储**：现在到了练习的关键部分。你需要编写一个函数，将你的 XState 状态机转换为 Svelte 存储。这涉及到订阅机器中的状态变化并将它们转发到 Svelte 存储，以及将存储中的操作映射回状态机的转换。

1.  **测试你的实现**：在你完成转换后，确保你彻底测试它。更改存储中的状态，并观察是否在状态机中反映了相同的更改，反之亦然。

如果你遇到了困难，退一步参考 XState 和本章的文档，或者 `@xstate/svelte` 的源代码。

摘要

在本章中，我们学习了如何使用 Svelte 管理我们的应用程序状态。

在本章的开头，我们讨论了一些通过 Svelte 存储管理 Svelte 中状态的技巧。随着你的 Svelte 应用程序变得更大、更复杂，这些技巧将对你非常有用。

我们讨论的其中一个技巧是使用状态管理库来管理数据变化和数据流。这就是为什么我们在本章的后半部分探讨了如何在 Svelte 中使用状态管理库，通过将状态管理库的状态转换为 Svelte 存储。

在下一章中，我们将探讨如何结合使用 Svelte 上下文和 Svelte 存储来创建无渲染组件——不渲染任何内容的逻辑组件。

```js

```
