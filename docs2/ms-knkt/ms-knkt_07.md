# 第七章. 最佳实践

到目前为止，所有的编码建议都穿插着 Knockout 技术介绍。为了更详细地介绍这些模式以及它们为什么有用，并提供一个综合参考，我们将在这章中回顾它们。由于 JavaScript 是一个非常灵活的语言，拥有最大的在线开发者社区之一，并且在业余爱好者到企业级开发的各个层面都有使用，因此不带有偏见地谈论好的或有用的模式是困难的。这些实践应被视为建议，而不应被视为教条。许多这些建议适用于一般的编程，而不仅仅是 Knockout 开发。

# 坚持 MVVM

Knockout 是按照 **模型-视图-视图模型**（**MVVM**）模式设计的。虽然使用 Knockout 和其他设计模式开发应用程序是可能的，但坚持 MVVM 将会在 Knockout 和您的代码之间产生自然的对齐。

## 视图和视图模型

关注点的分离是关键。不要在您的视图模型中引入视图概念，如 DOM 元素或 CSS 类；这些属于 HTML。限制或避免在您的视图中使用业务逻辑和内联绑定函数；这些应作为属性或函数存在于视图模型中。保持这两者分离可以使工作得以分割和并行化，使视图模型可重用，并使单元测试视图模型成为可能。

### 视图模型杂乱

动画处理程序是视图逻辑的一个很好的例子，这些逻辑通常最终会出现在视图模型中。`foreach` 绑定处理程序有几个后处理钩子（如 `afteradd`、`afterrender` 和 `beforeremove`），目的是允许使用动画。使用视图模型函数似乎很自然，因为它们在绑定中指定，通常绑定视图模型属性：

```js
<div data-bind='template: { foreach: planetsToShow,
                            beforeRemove: hidePlanetElement,
                            afterAdd: showPlanetElement }'>
    <div data-bind='attr: { "class": "planet " + type }, text: name'></div>
</div>

var PlanetsModel = function() {
    //Viewmodel properties
    this.planets = ko.observableArray();

    // Animation callbacks for the planets list
    this.showPlanetElement = function(elem) {
        if (elem.nodeType === 1) {
$(elem).hide().slideDown() ;
}
    }
    this.hidePlanetElement = function(elem) {
    	if (elem.nodeType === 1) {
$(elem).slideUp(function() { $(elem).remove(); });
}
    }
};
```

不幸的是，这紧密地将视图模型与视图耦合在一起，使得视图模型和动画都变得不可重用。更好的解决方案是将动画存储在全局可访问的位置，例如 `ko.animations`，并在绑定中引用它们：

```js
<div data-bind='template: { foreach: planetsToShow,
                            beforeRemove: ko.animations.slideHide,
                            afterAdd: ko.animations.slideShow }'>
    <div data-bind='attr: { "class": "planet " + type }, text: name'></div>
</div>

ko.animations = {};
ko.animations.slideShow = function(elem) {
    if (elem.nodeType === 1) {
$(elem).hide().slideDown();
}
};
ko.animations.slideHide =  function(elem) {
    if (elem.nodeType === 1) {
$(elem).slideUp(function() { $(elem).remove(); });
   }
};

var PlanetsModel = function() {
    //Viewmodel properties
    this.planets = ko.observableArray();
};
```

现在，相同的动画可以在其他列表中重用，并且视图模型不包含控制 DOM 的逻辑。

### 视图杂乱

虽然保持视图模型对视图的无知通常是明确的（不要引用 HTML 类型），但将内联代码排除在视图之外通常需要更多的考虑。这部分是因为与呈现相关的逻辑可能属于视图，或者至少是一个绑定处理程序，部分是因为当需要许多小而独特属性时，存在一种平衡行为。

不属于视图的内联逻辑的例子是一个按钮禁用表达式：

```js
<button data-bind="disable: items().length > 3, click: submitOrder">Submit</button>
```

考虑这种情况，这个值需要改变：你真的想在整个 HTML 中寻找控制这个规则的规则吗？当这个值是变量并由其他因素确定时呢？这绝对应该是视图模型中计算出的`canSubmit`（或类似名称），因为项目数量的最大值是业务逻辑，这不是视图的领域。

一个不那么明确的例子是基于类似逻辑的警告显示。假设禁用按钮不足以提供视觉提示，你还希望按钮变成红色：

```js
<form data-bind="submit: submitOrder, css: { 'invalid-form': items().length > maxItems }">
    //Irrelevant form code…
</form>
```

这不是一个完美的例子，你可能会想在你的视图模型中添加一个`overMaxItemLimit`计算属性；而且它也没有直接表达业务逻辑。*如果表单中的项目太多，突出显示表单*是表示逻辑，如果你有足够的这些一次性计算属性，它们只是对单个可观察对象进行简单表达式，你的视图模型会很快变得杂乱。在这些情况下，强迫视图模型表示这种逻辑可能没有任何价值，你在决定将其放在哪里时应该谨慎行事。

# 使用服务模块

即使在小型应用程序中，视图模型也不应包含所有应用程序代码。当可能时，代码应拆分为非视图模型模块，这些模块封装了工作并可以重用。这些模块通常被称为服务。

例如，从服务器获取数据的视图模型不需要知道该操作是如何处理的，无论是使用 jQuery 的 AJAX 方法、WebSocket 还是其他检索方法。将此逻辑放入数据服务模块不仅使其对其他视图模型可重用，而且通过限制每个对象的作用域到其自身的工作，使单元测试更容易。这里的驱动哲学是单一职责原则。

# 创建小型模块

创建更小的模块可以使单元测试更简单，并减少其他人阅读代码时理解代码所需的努力。在决定是否向模块添加功能或将其拆分为新的模块时，请牢记单一职责原则。

这是一种平衡行为。如果你的 JavaScript 应用程序有一个 RESTful API，那么创建模块通过提供方法来抽象单个 URL 是个好主意。然而，包含整个应用程序所有 URL 的单个数据服务模块，即使在中等规模的应用程序中，也会导致模块非常大。另一方面，为每个单独的路由创建服务模块会产生更多的文件。这将使单元测试和维护更困难，而不是更容易。最好的做法是按功能将路由分组到模块中。在 REST URL 的情况下，按资源分组会产生非常自然的组织。

# 编写单元测试

如果你遵循所有之前的建议，那么你的代码将处于良好的单元测试状态。在编写可测试的单元测试代码时，主要考虑的是可模拟性：外部依赖松散耦合的代码。松散耦合的依赖可以在单元测试中用模拟、存根、mock、spy 或其他形式的替代品替换，其行为可以通过测试来控制。这个挑战通过将 DOM 和绑定代码从 viewmodel 中分离出来，保持模块小，并通过依赖注入等实践避免与其他 viewmodel 的紧密耦合来解决。

有几个 JavaScript 单元测试框架可供使用，它们都提供了类似的好处和工作流程。重要的是你使用什么工具进行单元测试，而不是你是否编写了单元测试。单元测试的价值真的无法过高估计。在像 JavaScript 这样的不提供编译时检查的动态语言中，它甚至更重要。

# 单例与实例

当你有一个实际被多次使用的 viewmodel 时，例如支持`foreach`循环的 viewmodel，使用实例是唯一的选择。当 viewmodel 只有一个实例时，例如支持表单输入或 SPA 中的页面的 viewmodel，选择可能并不简单。

一个好的经验法则是考虑对象的生命周期。如果对象的生命周期不会结束，例如始终存在的导航栏的 viewmodel，使用单例是合适的。如果对象的生命周期较短，例如 SPA 中的页面 viewmodel，那么使用单例意味着对象即使在不再被积极使用后也无法被垃圾回收。在这种情况下，建议使用可丢弃的实例。

另一个经验法则是考虑它是否有内部状态。如果没有需要管理的内部状态，那么多次使用对象或其方法导致错误的风险很小。如果一个对象没有内部状态，例如抽象 AJAX 请求或 cookie 访问的服务，即使对象有有限的生命周期，单例也是合适的。这不适用于状态重要的 viewmodel，例如支持表单输入的 viewmodel；这是因为每次使用时，它都应该有一个新的状态。即使对象有较长的生命周期，例如导航栏中的登录 viewmodel，也需要新的状态。在注销后重建 viewmodel 将确保之前使用的信息不会保留。

# 一次调用 ko.applyBindings（每个根）

我不知道有多少次在 Stack Overflow 上看到关于开发者多次调用`ko.applyBindings`导致的问题，他们认为这是同步 DOM 和可观察数据的原因。这与其说是一个**最佳实践**，不如说是一个**警告**，但如果完全忽略它，我会感到疏忽。对于 HTML 中的任何给定根元素，最多只能有一个`ko.applyBindings`调用。

# 性能问题

自从 Knockout 首次发布以来，其性能已经提高了好几次，但在包含大量操作或对象的应用中仍然可能遇到问题。虽然随着工作量的增加，性能的某些下降是可以预料的，但有一些方法可以减轻 CPU 的负担。

## 可观察循环

在循环中更改可观察数组会导致它们多次发布更改通知。这些更改的成本与数组的大小成比例。你可能需要向数组添加多个项目，并使用循环来完成这项工作：

```js
var contacts = ko.observableArray();
for (var i = 0, j = newContacts.length; i < j; i++) {
    contacts.push(new Contact(newContacts[i]);
}
```

这里的问题是`push`被多次调用，导致数组发送出多次更改通知。如果所有更改一次性发送，对数组的订阅者来说会更容易。这可以通过在循环中收集所有更改，然后在最后使用`push.apply`将它们应用到可观察数组中来实现：

```js
var contacts = ko.observableArray();
for (var i = 0, j = newContacts.length, newItems = []; i < j; i++) {
    newItems.push(new Contact(newContacts[i]);
}
contacts.push.apply(contacts, newItems);
```

前述方法确保对于可观察数组只发生一次`valueHasMutated`调用。对于这个常见问题的一个流行解决方案是将此添加到`observableArray.fn`对象上的函数中，使其对所有可观察数组都可用：

```js
ko.observableArray.fn.pushAll = function (items) {
    this.push.apply(this, items);
};
```

以下方法可以用来添加一个项目数组：

```js
var contacts = ko.observableArray();
for (var i = 0, j = newContacts.length, newItems = []; i < j; i++) {
    newItems.push(new Contact(newContacts[i]);
}

contacts.pushAll(newitems);
```

## 限制活动绑定

大量的绑定，尤其是那些注册事件处理程序（如值和点击）的绑定，可能会迅速导致浏览器性能下降。管理这一点需要仔细考虑如何最好地减少同时需要发生的变化的数量。

一种方法是使用控制流绑定来移除不必要的绑定部分。限制屏幕上的内容量有助于性能，并且还有减少用户需要解析的杂乱信息的附带好处。例如，分页技术不仅适用于长列表，还可以用来将长表单或活动拆分成几个屏幕。当然，这种方法仅限于可以拆分的活动。

一个更广泛适用的方法是使用委派事件，也称为**非侵入式事件处理器**。

## 委派事件

无侵入式事件处理器，如 jQuery 的 `on`，可以使用单个事件处理器来响应注册元素内部任意数量的 DOM 元素上的事件。这在处理大型或递归列表时特别有用，因为为每个元素注册单个事件处理器可能会过于昂贵。Knockout 提供了两种实用方法来将这些处理器与绑定上下文中的适当数据连接起来，其方式类似于 Knockout 的点击绑定如何将上下文作为第一个参数提供：

+   `ko.dataFor` (元素): 这返回可用于元素的 数据

+   `ko.contextFor` (元素): 这返回元素的绑定上下文（包括绑定上下文属性，如 `$parent` 和 `$root`）

这可以与提供事件委托的绑定处理器结合使用：

```js
ko.bindingHandlers.on = {
   init: function(element, valueAccessor, allBindings, viewModel, bindingContext) {
      var options = valueAccessor();
      var handler = function() {
         options.method.call(bindingContext.$rawData, ko.dataFor(this));
      };

      $(element).on(options.event, options.selector, handler);

      ko.utils.domNodeDisposal.addDisposeCallback(element, function() {
         $(element).off(options.event, options.selector, handler);
      });
   }
};

<ul class="list-unstyled" data-bind="foreach: displayContacts, on: { event: 'click', selector: '.remove-btn', method: deleteContact }">
  <li data-bind="compose: { model: $data, mode: 'templated' }">
  <div data-part="btn-container" class="inline">
    <button class="btn btn-sm btn-default"  data-bind="click: edit">Edit</button>
    <button class="btn btn-sm btn-danger remove-btn">Delete</button>
  </div>
  </li>
</ul>
```

前面的示例可以在 `cp7-unobtrusive` 分支中看到。

前述技术并非在所有地方都必需，但在处理大量处理器时，它可能会导致性能上的明显影响。

# 摘要

再次强调，这些是指导原则而不是规则，其中一些是可能导致同事之间意见分歧的观点。有时，打破模式会产生更清晰、更简洁的代码，并且是使某些事情能够工作的唯一方法；有时，打破模式是唯一能与同事妥协的方法。如果它阻碍了你的工作而没有给你带来任何好处，就不要这样做。软件开发没有唯一正确的方法。

下一章将介绍一些由社区维护的流行 Knockout 插件。
