# 第九章：动画

应用程序中的动画现在是常见的。动画视觉转换通常会给人一种流畅感，而且很好地使用动画可以是向用户传达某事最好的方式，比图标、图片或又是另一段文字。

Aurelia 的模板引擎已被设计来支持动画。它使用一个抽象层，允许可插拔的动画库，而 Aurelia 生态系统已经提供了多个实现。

在本章中，我们将首先了解动画师 API，并看看模板引擎是如何与其交互的。然后，我们将向我们的联系管理应用程序添加一些简单的基于 CSS 的动画，以了解它是如何工作的。

# 动画师 API

在`aurelia-templating`库中，`TemplatingEngine`类需要与动画服务一起工作以执行视图转换。默认情况下，它使用一个名为`Animator`的类，该类作为空对象，顺便说一下，描述了`Animator`期望的接口。

### 注意

**空对象**设计模式描述了一个作为接口的空实现的对象或类。这个对象可以用作 null 引用，并消除了在引用之前检查 null 的需要。您可以在[`sourcemaking.com/design_patterns/null_object`](https://sourcemaking.com/design_patterns/null_object)上获取有关此模式的更多信息。

以下是从动画师 API 中最常用的方法：

+   `enter(element: HTMLElement): Promise<boolean>`: 在 DOM 中添加元素的动画效果

+   `leave(element: HTMLElement): Promise<boolean>`: 将元素从 DOM 中移除的动画效果

+   `addClass(element: HTMLElement, className: string): Promise<boolean>`: 为元素添加 CSS 类，这可以根据实现方式触发动画

+   `removeClass(element: HTMLElement, className: string): Promise<boolean>`: 从元素中移除 CSS 类，这可以根据实现方式触发动画

+   `animate(element: HTMLElement|HTMLElement[], className: string): Promise<boolean>`: 在一个元素或元素数组上执行单个动画。`className`要么是触发动画的 CSS 类，要么是应用的效果名称，要么是动画的属性，这取决于动画师实现方式

+   `runSequence(animations: CssAnimation[]): Promise<boolean>`: 按顺序运行一系列动画。`CssAnimation`是一个由具有`element: HTMLElement`和`className: string`属性的对象实现的接口。对于每个动画，`className`要么是触发动画的 CSS 类，要么是应用的效果名称，要么是动画的属性，这取决于动画师实现方式

所有这些方法都返回一个`Promise`，其解决值为一个`boolean`值。这个值通常是`true`，当确实执行了动画时，以及`false`，当没有执行动画时。最后一个场景可能会发生，例如，尝试使用不定义任何动画的 CSS 类来动画化一个元素。

在撰写本文时，模板引擎对动画师的调用仅限于在将元素添加到 DOM 时调用其`enter`方法，然后在移除它时调用其`leave`方法。其他方法不被框架使用，但将由我们自己的代码使用。

最后，对元素的动画转换是可选的。模板引擎在渲染元素时调用`enter`方法，在从 DOM 中移除它时调用`leave`方法，但仅当元素具有`au-animate`CSS 类。这是出于性能原因；如果没有这个可选机制，每次渲染和卸载任何元素时都会执行大量无用的代码，而通常，只有少数选定的元素具有动画转换。

# CSS 动画师

`aurelia-animator-css`库是基于 CSS 的动画师实现。我们将安装它，并使用它为我们的联系人管理应用程序添加简单的基于 CSS 的动画。

## 安装插件

首先，在项目目录中打开一个控制台，并运行以下命令：

```js
> npm install aurelia-animator-css --save

```

像往常一样，它需要添加到供应商包中。在`aurelia_project/aurelia.json`文件中，在`build`下的`bundles`部分，添加到名为`vendor-bundle.js`的包的`dependencies`下列：

```js
"aurelia-animator-css", 

```

最后，我们需要加载插件，以便模板引擎使用它而不是默认的`Animator`：

`src/main.js`

```js
//Omitted snippet... 
export function configure(aurelia) { 
  aurelia.use 
    .standardConfiguration() 
    .plugin('aurelia-animator-css') 
    .feature('validation') 
  //Omitted snippet... 
} 

```

至此，一切就绪，可以处理 CSS 动画。

## 动画视图转换

然而，在动手之前，让我们快速了解一下高级算法，并看看基于 CSS 的动画师的`enter`和`leave`方法是如何工作的。

当模板引擎将渲染的元素添加到 DOM 时，以下过程会发生：

1.  模板引擎将元素添加到 DOM。

1.  模板引擎检查元素是否具有`au-animate`类。如果有，它调用动画师的`enter`方法。如果没有，动画师完全被绕过，过程到这里结束。

1.  动画师为元素添加`au-enter`类。这个类可以在描述元素在整个动画过程中将保持不变的样式的 CSS 规则中使用。

1.  动画师为元素添加`au-enter-active`类。这个类应该在触发动画的 CSS 规则中使用。

1.  动画师检查元素的计算样式是否包含动画。如果不包含，它会从其中移除`au-enter`和`au-enter-active`类，并使用`false`解决产生的`Promise`。到这里过程结束。如果包含，它开始监听浏览器上的`animationend`事件。

1.  当收到`animationend`事件时，动画师将元素的`au-enter`和`au-enter-active`类移除，并将产生的`Promise`解决为`true`。

从 DOM 中删除元素的流程非常相似，但顺序相反：

1.  模板引擎检查元素是否具有`au-animate`类。如果有，它调用动画师的`leave`方法。如果没有，动画师完全被绕过，过程直接跳到第 6 步。

1.  动画师给元素添加了`au-leave`类。这个类可以在描述元素在整个动画期间将保持不变的样式的 CSS 规则中使用。

1.  动画师给元素添加了`au-leave-active`类。这个类应该用在触发动画的 CSS 规则中。

1.  动画师检查元素的计算样式是否包含动画。如果有，它开始监听浏览器的一个`animationend`事件。如果没有，它将`au-leave`和`au-leave-active`类从其中移除，并将产生的`Promise`解决为`false`。过程直接跳到第 6 步。

1.  当收到`animationend`事件时，动画师将元素的`au-leave`和`au-leave-active`类移除，并将产生的`Promise`解决为`true`。

1.  模板引擎将元素从 DOM 中移除。

既然我们已经理解了基于 CSS 的动画师是如何处理事情的，让我们先来动画化`list-editor`组件。

### 列表编辑器动画

我们在第五章*制作可复用的组件*中编写的`list-editor`组件具有允许用户添加和删除项目的特性。让添加的项目出现，比如窗帘被拉下，和删除的项目消失，比如窗帘被拉上，应该不会很难。

为了这样做，我们首先需要为组件定义 CSS 动画：

`src/resources/elements/list-editor.css`

```js
list-editor .le-item.au-enter-active { 
  animation: blindDown 0.2s; 
  overflow: hidden; 
} 

@keyframes blindDown { 
  0% { max-height: 0px; } 
  100% { max-height: 80px; } 
} 

list-editor .le-item.au-leave-active { 
  animation: blindUp 0.2s; 
  overflow: hidden; 
} 

@keyframes blindUp { 
  0% { max-height: 80px; } 
  100% { max-height: 0px; } 
} 

```

在这里，我们首先定义了用于添加项目的 CSS 规则；项目的`max-height`将在 0.2 秒内从 0 动画到 80 像素，在此期间，其溢出内容将被隐藏。

然后，我们定义了用于删除项目的 CSS 规则。这与添加项目非常相似，但顺序相反；它的`max-height`在 0.2 秒内从 80 像素动画到 0。在这个动画期间，它的溢出内容也将被隐藏。

当然，我们需要用组件加载这个新的 CSS 文件：

`src/resources/elements/list-editor.html`

```js
<template> 
  <require from="./list-editor.css"></require> 
  <!-- Omitted snippet... --> 
</template> 

```

我们还需要提示模板引擎项目应该被动画化：

`src/resources/elements/list-editor.html`

```js
<!-- Omitted snippet... --> 
<div class="form-group le-item ${animated ? 'au-animate' : ''}"  
     repeat.for="item of items"> 
  <!-- Omitted snippet... --> 
</div> 
<!-- Omitted snippet... --> 

```

在这里，我们在`class`属性中添加了一个字符串插值表达式，只有在`animated`属性为真时，才会给项目的`div`元素添加`au-animate`类。

在视图模型中，`animated`属性将初始设置为`false`，因此在组件渲染时项目不会被动画化。只有在组件完全附加到 DOM 时，该属性才会被设置为`true`，因此添加新项目或移除现有项目的操作才能正确动画化：

`src/resources/elements/list-editor.js`

```js
// Omitted snippet... 
export class ListEditorCustomElement { 
  // Omitted snippet... 

  animated = false; 

  attached() { 
    setTimeout(() => { this.animated = true; }); 
  } 
} 

```

为什么我们不在`attached`回调方法中直接将`animated`设置为`true`？为什么要用`setTimeout`类？嗯，如果你记得前一部分中描述的动画过程，首先元素被附加到 DOM，这意味着`attached`回调在同一时间被调用，然后动画师检查`au-animate`CSS 类。如果在`attached`回调中同步地将`animated`设置为`true`，当动画师检查是否需要对元素进行动画化时，`au-animate`CSS 类将出现在元素上，在初始渲染期间项目将被动画化，这是我们想要防止的。相反，我们将`animated`设置为`true`推送到浏览器的事件队列中，这样当`au-animate`CSS 类添加到项目的`div`元素时，组件的渲染已完成。

至此，如果你运行应用程序，导航到联系人`creation`或`edition`组件，并尝试编辑列表编辑器；你应该看到动画播放。

## 手动触发动画

除了动画过渡效果，动画师还支持手动触发的动画。与动画过渡不同，手动动画没有`au-enter`或`au-leave`这样的 CSS 类。相反，动画是通过使用用户自定义的 CSS 类来手动触发的。

用于手动触发动画的基本方法是 addClass 和 removeClass。这些方法允许你向元素添加或移除 CSS 类，并在两个状态之间实现动画过渡。

例如，假设我们有一个名为`A`的 CSS 类。如果我们调用`animator.addClass('A')`，以下过程会发生：

1.  动画师将`A-add`类添加到元素上。

1.  动画师检查元素的计算样式是否包含动画。如果不包含，它将`A`类添加到元素上，然后将其上的`A-add`类移除，并以`false`解析结果`Promise`。在此处结束该过程。如果包含动画，它开始监听浏览器上的`animationend`事件。

1.  当接收到`animationend`事件时，动画师将`A`类添加到元素上，然后将其上的`A-add`类移除。

正如你所看到的，这个过程允许你向元素添加一个 CSS 类，并在没有该类的元素和具有该类的元素之间实现动画状态过渡，该过程应由带有`-add`后缀的中间类触发。

此外，当在同一元素上调用`animator.removeClass('A')`时，以下过程会发生：

1.  动画师将`A`类从元素中移除。

1.  动画师将`A-remove`类添加到元素上。

1.  动画师检查元素的计算样式是否包含动画。如果不包含，它会从其中移除`A-remove`类，并用`false`解析产生的`Promise`。流程在这里结束。如果包含，它开始监听浏览器上的`animationend`事件。

1.  当收到`animationend`事件时，动画师从元素中移除`A-remove`类，并用`true`解析产生的`Promise`。

这个过程允许您从一个元素中移除 CSS 类，在带有类和不带类的元素之间进行带有动画的状态转换，该状态转换应由带有`-remove`后缀的中间类触发。

最后，`animate`方法允许按顺序触发`addClass`和`removeClass`。在这种情况下，动画可以由`-add`类、`-remove`类或两者同时触发。

### 强调验证错误

让我们在我们的联系人管理应用程序中尝试这个功能，通过添加一个动画，当用户尝试保存一个联系人并且表单无效时，验证错误会闪烁几次。

首先，我们需要创建一个 CSS 动画：

`src/contacts/components/form.css`

```js
.blink-add { 
  animation: blink 0.5s; 
} 

@keyframes blink { 
  0% { opacity: 1; } 
  25% { opacity: 0; } 
  50% { opacity: 1; } 
  75% { opacity: 0; } 
  100% { opacity: 1; } 
} 

```

这里，我们简单地定义了一个 CSS 规则，使匹配的元素在半秒内闪烁两次。触发动画的类名为`blink-add`，所以我们可以通过调用`addClass`来触发它。然而，由于使错误信息闪烁不是一个状态转换，并且我们不想让我们的错误信息带有`blink`类，我们将通过调用`animate`来触发它，这样我们就可以确保`blink`在动画结束时被移除。

为了促进代码重用，让我们将当前仅作为模板的联系人`form`组件转换为完整的组件。为此，我们需要为表单创建一个视图模型。在这个视图模型中，我们将添加一个通过使它们闪烁来强调错误的方法：

`src/contacts/components/form.js`

```js
import {inject, bindable, DOM} from 'aurelia-framework'; 
import {Animator} from 'aurelia-templating'; 

@inject(DOM.Element, Animator) 
export class ContactFormCustomElement { 

  @bindable contact; 

  constructor(element, animator) { 
    this.element = element; 
    this.animator = animator; 
  } 

  emphasizeErrors() { 
    const errors = this.element 
      .querySelectorAll('.validation-message'); 
    return this.animator.animate(Array.from(errors), 'blink'); 
  } 
} 

```

首先，我们定义视图模型，在其中移动`bindable` `contact`属性的声明；然后我们注入组件的 DOM 元素和`animator`实例。接下来，我们定义一个`emphasizeErrors`方法，该方法检索元素内的所有验证错误并使用`blink`效果调用它们。

当调用`animate`时，`animator`将遍历向元素添加`blink-add`的过程，这将触发动画。动画完成后，它将移除`blink`，添加`blink-remove`，并且由于`blink-remove`不触发任何动画，它将立即移除它，使元素回到过程开始时的状态。

接下来，我们需要从模板中移除`bindable`属性，因为`contact`现在是由视图模型定义的，并且加载包含新动画的 CSS 文件：

`src/contacts/components/form.html`

```js
<template> 
  <require from="./form.css"></require> 
  <!-- Omitted snippet... --> 
</template> 

```

最后，让我们更新一下`creation`组件。我们首先需要更改`form`的`require`语句，去掉`.html`后缀，这样模板引擎就知道该组件不仅仅是一个模板，还包含一个视图模型。我们还需要在`creation`组件的模板中获取`form`视图模型的引用：

`src/contacts/components/creation.html`

```js
<template> 
  <require from="./form"></require> 
  <!-- Omitted snippet... --> 
  <contact-form contact.bind="contact"  
    view-model.ref="form"></contact-form> 
  <!-- Omitted snippet... --> 
</template> 

```

通过在`contact-form`自定义元素上添加`view-model.ref="form"`属性，将`form`视图模型的引用分配给`creation`视图模型作为一个新的`form`属性。

我们现在可以使用这个`form`属性在验证失败时调用`emphasizeErrors`方法：

`src/contacts/components/creation.js`

```js
//Omitted snippet... 
save() { 
  return this.validationController.validate().then(errors => { 
    if (errors.length > 0) { 
      this.form.emphasizeErrors(); 
      return; 
    } 
    //Omitted snippet... 
  } 
} 
//Omitted snippet... 

```

至此，如果您运行应用程序，点击`New`按钮，在**Birthday**字段中输入胡言乱语，然后点击**Save**，验证错误信息应该出现并闪烁两次。每次点击**Save**按钮时，它应该再次闪烁。

当然，`edition`组件也应该以同样的方式进行修改。我将留给读者作为练习。本章节的示例应用程序可以作为参考。

## 动画路由转换

另一个可能从动画转换中受益的区域是路由器。让我们给路由转换添加一个简单的淡入/淡出动画：

`src/app.css`

```js
/* Omitted snippet... */ 

section.au-enter-active { 
  animation: fadeIn 0.2s; 
} 

section.au-leave-active { 
  animation: fadeOut 0.2s; 
} 

@keyframes fadeIn { 
  0% { opacity: 0; } 
  100% { opacity: 1; } 
} 

@keyframes fadeOut { 
  0% { opacity: 1; } 
  100% { opacity: 0; } 
} 

```

在这里，我们创建 CSS 规则，使`section`元素在进入时淡入，在离开时淡出。

接下来，我们只需向每个路由组件的`section`元素添加`au-animate`类。

如果您在此时运行应用程序，路由更改应该使用新动画平滑过渡。

### 交换顺序

当执行路由转换时，`router-view`元素用新视图替换旧视图。默认情况下，这个交换过程首先动画化旧视图的移除，然后是新视图的插入。如果没有视图动画，过程是立即的。如果两个视图都有动画，动画一个接一个地运行。

`router-view`处理视图交换的方式称为交换策略，可以是以下之一：

+   `before`：首先添加新视图，然后移除旧视图。如果新视图有动画，则等待其`enter`动画完成后再动画化旧视图的移除。

+   `with`：新视图添加，旧视图移除同时进行。两个动画并行运行。

+   `after`：默认的交换策略。先移除旧视图，然后添加新视图。如果旧视图有动画，新视图的插入仅在旧视图的移除动画完成后进行一次动画化。

我们的淡入/淡出转换之所以正常工作，是因为它遵循了默认的交换策略：首先将旧视图动画化移出，然后将新视图动画化进入。然而，某些动画可能需要不同的交换策略。

例如，如果你在从一个路由跳转到另一个路由时希望看到新视图从右侧滑入，而旧视图向左滑出，那么你需要旧视图的移除动画和新视图的添加动画同时运行，因此你需要使用`with`交换策略。

因此，`router-view`元素的交换策略可以通过将其`swap-order`属性设置为适当策略的名称来更改：

```js
<router-view swap-order="with"></router-view> 

```

# 摘要

为 Aurelia 应用程序添加动画相当简单。基于 CSS 的实现允许您轻松快速地为现有应用程序添加动画。

当需要更复杂的动画时，如果它不存在，可以很容易地编写您最喜欢的动画库的适配器插件。在撰写本文时，官方 Aurelia 库包括`aurelia-velocity`，它是流行的`velocity.js`库的适配器插件。我确信社区最终会提出其他动画解决方案的适配器，所以我强烈建议您密切关注它。
