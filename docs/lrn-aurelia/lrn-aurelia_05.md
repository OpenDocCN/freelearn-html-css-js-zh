# 第五章：制作可重用组件

Aurelia 是为了可重用性和组合性而构建的。因此，其模板引擎不仅支持组件的组合，还支持自定义 HTTP 元素和属性，在 Aurelia 的术语中称为 HTML 行为。实际上，我们在模板中使用的资源，如`if`、`repeat`、`show`、`focus`、`compose`和`router-view`，并不是框架中内置的特殊构建，而是使用与我们编写自己的自定义 HTML 行为相同的 API 编写的实际 HTML 行为。

在本章中，我们将了解组合与自定义元素的区别，以及每种技术的优缺点，并探讨在哪种场景下一方比另一方更适合。接下来，我们将查看如何创建自定义属性和自定义元素，以及我们可以它们做什么。最后，我们将了解如何自定义 Aurelia 的视图位置约定。

# 组合

组合是 Aurelia 应用程序中组装组件的最简单方法，也是最有限的方法。其主要目的是在其他上下文中重用现有的组件和模板。组合只适用于简单的重用场景，其中情况与预期使用相差不大。与 HTML 行为相比，组合的灵活性非常有限。

在以下部分中，我们将通过重构我们的联系人管理应用程序来了解组合的各种可能性和局限性。我们将从我们的`contact-edition`组件中提取联系人创建行为，放入一个新的`contact-creation`组件中。这样做是为了实现更清晰的设计，使我们的新组件具有更专注的责任。然而，由于联系人表单本身在两种上下文中都是相同的，我们将看到各种提取这个公共模板和行为并在这两个组件中重用它们的方法。

## 拆分联系人编辑组件

首先，我们从`contact-edition`组件中移除所有与联系人创建相关的引用：

`src/contact-edition.js`

```js
//Omitted snippet... 
export class ContactEdition { 
  //Omitted snippet... 

  activate(params, config) { 
    return this.contactGateway.getById(params.id).then(contact => { 
      this.contact = contact; 
      config.navModel.setTitle(this.contact.fullName); 
    }); 
  } 

  save() { 
    return this.validationController.validate().then(errors => { 
      if (errors.length > 0) { 
        return; 
      } 

      return this.contactGateway.update(this.contact.id, this.contact) 
        .then(() => this.router.navigateToRoute('contact-details', { id: this.contact.id })); 
    }); 
  } 
} 

```

在这里，我们简单地移除了`isNew`属性，以及在`activate`和`save`方法中使用它的`if`语句和相关代码分支。

同样的事情也适用于模板：

`src/contact-edition.html`

```js
<template> 
  <section class="container"> 
    <h1>Contact #${contact.id}</h1> 
    <!-- Omitted snippet --> 
    <div class="form-group"> 
        <div class="col-sm-9 col-sm-offset-3"> 
          <button type="submit" class="btn btn-success">Save</button> 
          <a class="btn btn-danger" route-href="route: contact-details;
params.bind: { id: contact.id }">Cancel</a> 
        </div> 
      </div> 
    </form> 
  </section> 
</template> 

```

在这里，我们简单地移除了创建新组件时显示的静态标题和**取消**按钮，基本上就是当`isNew`为`true`时显示的所有模板部分。

接下来，让我们创建一个新的`contact-creation`组件：

`src/contact-creation.js`

```js
import {inject, NewInstance} from 'aurelia-framework'; 
import {ValidationController} from 'aurelia-validation'; 
import {Router} from 'aurelia-router'; 
import {ContactGateway} from './contact-gateway'; 
import {Contact} from './models'; 

@inject(ContactGateway, NewInstance.of(ValidationController), Router) 
export class ContactCreation { 

  contact = new Contact(); 

  constructor(contactGateway, validationController, router) { 
    this.contactGateway = contactGateway; 
    this.validationController = validationController; 
    this.router = router; 
  } 

  save() { 
    return this.validationController.validate().then(errors => { 
      if (errors.length > 0) { 
        return; 
      } 

      return this.contactGateway.create(this.contact) 
        .then(() => this.router.navigateToRoute('contacts')); 
    }); 
  } 
} 

```

在这个新组件的视图模型中，我们简单地将一个`contact`属性初始化为一个新的`Contact`实例。此外，我们定义了一个`save`方法，如果没有验证错误，它将委派给`ContactGateway`的`create`方法，当返回的`Promise`解决时，导航回到联系人列表。

对于模板，我们将从表单字段本身的框架开始：

`src/contact-creation.html`

```js
<template> 
  <section class="container"> 
    <h1>New contact</h1> 

    <form class="form-horizontal" validation-renderer="bootstrap-form"  
          submit.delegate="save()"> 
      <!-- The form will go here --> 

      <div class="form-group"> 
        <div class="col-sm-9 col-sm-offset-3"> 
          <button type="submit" class="btn btn-success">Save</button> 
          <a class="btn btn-danger" route-href="route: contacts">Cancel</a> 
        </div> 
      </div> 
    </form> 
  </section> 
</template> 

```

除了我们目前省略的表单字段外，这个模板与`contact-edition`模板几乎完全相同。主要区别是高亮显示的，标题是一个静态的`New contact`字符串，而**取消**按钮导航回到联系人列表而不是联系人的详细信息。

## 复用模板

组合性提供的一个可能性就是在一个多个上下文中复用模板。我们将通过从`contact-edition.html`模板中提取表单字段到其单独的模板，来说明这一点，这样我们就可以在`contact-edition.html`和`contact-creation.html`中都使用它：

`src/contact-form.html`

```js
<template> 
  <div class="form-group"> 
    <label class="col-sm-3 control-label">First name</label> 
    <div class="col-sm-9"> 
      <input type="text" class="form-control" value.bind="contact.firstName & validate"> 
    </div> 
  </div> 

  <div class="form-group"> 
    <label class="col-sm-3 control-label">Last name</label> 
    <div class="col-sm-9"> 
      <input type="text" class="form-control" value.bind="contact.lastName & validate"> 
    </div> 
  </div> 

  <!-- Omitted company, birthday and note fields --> 

  <hr> 
  <div class="form-group" repeat.for="phoneNumber of contact.phoneNumbers"> 
    <div class="col-sm-2 col-sm-offset-1"> 
      <select value.bind="phoneNumber.type & validate" class="form-control"> 
        <option value="Home">Home</option> 
        <option value="Office">Office</option> 
        <option value="Mobile">Mobile</option> 
        <option value="Other">Other</option> 
      </select> 
    </div> 
    <div class="col-sm-8"> 
      <input type="tel" class="form-control" placeholder="Phone number"  
             value.bind="phoneNumber.number & validate"> 
    </div> 
    <div class="col-sm-1"> 
      <button type="button" class="btn btn-danger"  
              click.delegate="contact.phoneNumbers.splice($index, 1)"> 
        <i class="fa fa-times"></i> Remove 
      </button> 
    </div> 
  </div> 
  <div class="form-group"> 
    <div class="col-sm-9 col-sm-offset-3"> 
      <button type="button" class="btn btn-primary"  
              click.delegate="contact.addPhoneNumber()"> 
        <i class="fa fa-plus-square-o"></i> Add a phone number 
      </button> 
    </div> 
  </div> 

  <!-- Omitted emailAddresses, addresses and socialProfiles list editors --> 
</template> 

```

在此，我们从`contact-edition.html`中提取了字段集和列表编辑器，放入了单独的模板中。现在我们可以使用组合方法在这段模板中渲染之前的位置：

`src/contact-edition.html`

```js
<template> 
  <section class="container"> 
    <h1>Contact #${contact.id}</h1> 

    <form class="form-horizontal" validation-renderer="bootstrap-form" submit.delegate="save()"> 
      <compose view="contact-form.html"></compose> 

      <!-- Omitted buttons snippet... --> 
    </form> 
  </section> 
</template> 

```

此外，`contact-form.html`模板必须在`contact-creation.html`模板中进行组合。我会让你将模板中的注释替换为与前一个片段相同的`compose`指令。

### 注意

不要忘记更新`src/app.js`中的`contact-creation`路由，通过将它的`moduleId`属性更改为`'contact-creation'`。

一旦完成，你可以运行应用程序并测试一切是否仍然如常工作。

当使用组合方法渲染模板时，这个模板将继承周围的绑定上下文。这意味着，为了组合`contact-form.html`，渲染它的模板必须在其上下文中存储一个名为`contact`的`contact`对象。这是因为`contact-form.html`模板期望存在一个名为`contact`的上下文属性。

关于组合性的整个要点是，一个组件应该与其周围上下文独立。这个例子违反了这条规则。我们需要一种方法将`contact`对象注入到组件中。

如果我们的组件只是一个模板并且没有视图模型，我们可以在一个未命名的视图模型中注入`contact`对象：

```js
<compose view="contact-form.html" view-model.bind="{ contact: contact }"></compose> 

```

在这里，模板引擎将创建一个对象，并将其`contact`属性分配为周围绑定上下文中的`contact`对象，然后组合引擎将这个动态视图模型与`contact-form.html`模板进行数据绑定。

## 复用组件

如果我们的组件有行为，这意味着它有一个视图模型类。因此，前一个技术不能工作，因为它将用一个匿名对象覆盖组件的视图模型，并且组件将失去其行为。

尽管现在它没有任何行为，让我们为我们的`contact-form`组件创建一个空的视图模型，以便我们可以说明这一点：

`src/contact-form.js`

```js
export class ContactForm { 
} 

```

这将允许我们在`contact-creation.html`和`contact-edition.html`中更改`compose`指令，以便使用`contact-form`组件而不是单独的模板。为此，我们将使用`compose`元素的`view-model`属性而不是它的`view`属性：

```js
<compose view-model="contact-form"></compose> 

```

注意`compose`元素现在有一个`view-model`属性，而不是`view`属性，路径中的`.html`文件扩展名已经被移除，所以现在它指的是整个组件，而不仅仅是模板。

然而，像这样组合后，我们的组件又回到了依赖周围上下文的`contact`属性的状态。我们需要将其注入。

组合引擎支持向组合组件传递模型。因此，`compose`元素支持`model`属性：

```js
<compose view-model="contact-form" model.bind="contact"></compose> 

```

为了让视图模型接收这个模型，它必须实现一个名为`activate`的回调方法，该方法将由组合引擎调用，并传递绑定到`model`属性的值：

`src/contact-form.js`

```js
export class ContactForm { 
  activate(contact) { 
    this.contact = contact; 
  } 
} 

```

此时，`contact-form.html`模板使用`ContactForm`视图模型的`contact`属性进行数据绑定，覆盖了周围上下文的`contact`。这使得更加灵活。例如，你可以注入一个与周围上下文中的`contact`名称不同的对象，或者在不破坏任何内容的情况下更改`contact-form`组件中的属性名称。这类似于在同一个函数中传递一个参数与使用来自外层作用域的变量的区别。

当然，在这种情况下，由于`model`属性绑定到周围上下文的`contact`属性，如果这个`contact`属性被分配了一个新值，组件将被重新组合。这意味着组件的`activate`方法将被用新的`contact`值重新调用。

# 使用模板作为自定义元素

由于我们的`contact-form.html`模板只有一个参数，即一个联系对象，因此组合是绝对足够的。然而，如果我们的组件需要有多个可以分别绑定的参数，就不能使用组合，除非我们将所有参数聚合在一个单一的参数对象中，这可能会很快变得难看。另一方面，自定义元素正是为这种情况设计的。

为了举例，让我们将我们的`contact-form`组件转换为自定义 HTML 元素。由于 Aurelia 的模板引擎支持仅包含模板的自定义元素，因此我们可以删除`contact-form.js`视图模型，因为当前的`contact-form`除了渲染模板外没有其他行为。

接下来，我们只需要告诉模板哪些参数应该作为属性暴露在元素上：

`src/contact-form.html`

```js
<template bindable="contact"> 
  <!-- Omitted snippet... --> 
</template> 

```

在这里，我们在`template`元素上使用`bindable`属性，告诉 Aurelia 的模板引擎，当这个模板作为自定义元素使用时，暴露一个`contact`属性，模板可以使用这个自定义元素进行绑定。

要定义多个绑定属性，只需用逗号将它们分开。例如，`bindable="title, contact"`将定义两个名为`title`和`contact`的绑定属性。

然后，在`contact-creation.html`和`contact-edition.html`两个文件中，我们首先以资源的形式加载模板：

```js
<template> 
  <require from="contact-form.html"></require> 
  <!-- Omitted snippet... --> 
</template> 

```

`require`语句告诉模板引擎`contact-form`元素仅使用`contact-form.html`模板渲染，不带任何视图模型。

接下来，我们可以用我们新的`contact-form`元素替换`compose`指令：

```js
<contact-form contact.bind="contact"></contact-form> 

```

在这里，我们将`contact-creation`或`contact-edition`组件的`contact`属性绑定到我们`contact-form`自定义元素的`contact`属性上。这将把`contact`注入到模板中。另外，属性也将被绑定，这意味着如果周围上下文的`contact`被分配为新的值，`contact-form.html`上下文中的`contact`属性将被同步，所有依赖它的绑定也将被更新。

这允许我们在元素没有行为时，仅使用模板来创建自定义元素。在这种情况下，我们需要编写的代码量严格限制在最小值。

# 理解 HTML 行为

HTML 行为使我们能够用自定义元素和属性丰富标准 HTML。与组合相比，它们不仅提供了更多可能性，而且更加灵活，而且它们还具有比模板内的`compose`指令更具有语义意义的特点。

一个 HTML 行为至少由一个视图模型 JS 类组成。另外，自定义元素可以声明一个模板为其视图。当然，属性不能声明一个视图，因为它的意图只是简单地增强或改变元素的 behavior。

无论是 HTML 行为中的元素还是属性，它们都使用相同的基本概念并遵循相同的通用规则。

## 注入 DOM 元素

HTML 行为通常需要使用对其 DOM 元素的引用，特别是自定义属性。模板引擎了解这一点。当在模板中评估 HTML 元素时，它会在当前 DI 容器中暴露这个元素。

由于这一点，如果元素是一个 Aurelia 自定义元素，它的视图模型可以在构造函数中声明对`Element`类的依赖，并在其中看到 DOM 元素被注入。

同样，声明对`Element`类依赖的自定义属性在其构造函数中被实例化时，会看到它所声明的 DOM 元素被注入。

### 注意

应避免依赖浏览器全局变量。因此，`Element`类应从`aurelia-pal`库暴露的`DOM`接口中获取。这样，如果您的应用程序需要被使 为同质，它可以通过使用 PAL 的不同实现来在服务器上运行。

## 声明绑定属性

一个 HTML 行为可以声明绑定属性。这样的属性通过模板引擎暴露给外部世界，因此自定义元素或属性的实例可以绑定到这些属性。`bindable`装饰器允许我们将属性标识为如此。

例如，让我们想象一个名为`text-block`的自定义元素，它将暴露一个名为`text`的可绑定属性，并像这样使用：

```js
<text-block text.bind="someText "></text-block> 

```

为了将`text`属性作为属性暴露出来，元素的视图模型需要用`bindable`装饰它：

```js
import {bindable} from 'aurelia-framework'; 

export class TextBlockCustomElement { 
  @bindable text = 'Some default text'; 
} 

```

`bindable`装饰器可以接受一个选项对象，该对象可以有以下属性：

+   `defaultBindingMode`：当用在属性上时`.bind`命令所选择的绑定模式。应使用`bindingMode`枚举来设置此值。如果省略，则默认使用单向绑定。

+   `changeHandler`：更改处理方法的名称。如果省略，将使用属性名称后跟`Changed`的默认名称。例如，名为`title`的属性的更改处理方法名称为`titleChanged`。

+   `attribute`: 用于向外部暴露属性的属性名称。如果省略，将使用属性名称转换为连字符-格式的名称。例如，`defaultText`属性将作为`default-text`属性暴露出来。

    ### 注意

    Dash-case 是一种所有单词均为小写且由连字符分隔的命名模式。尽管社区对此名称没有明确的共识（它也被称为*kebab-case*），但为了使用一致的词汇，我将在整本书中坚持使用这种命名方式。

例如，让我们假设我们想要上一个示例中`text-block`自定义元素的`text`属性默认使用双向绑定，并且更改处理方法的名称为`onTextChanged`而不是默认的`textChanged`：

```js
import {bindable, bindingMode} from 'aurelia-framework'; 

export class TextBlockCustomElement { 
  @bindable({  
    defaultBindingMode: bindingMode.twoWay, 
    changeHandler: 'onTextChanged' 
  }) text = 'Some default text'; 
} 

```

如果你出于某种原因不能或不想在类内部声明绑定属性，可以直接在类上使用`bindable`装饰器。在这种情况下，传递给`bindable`的选项对象应该有一个`name`属性，这个属性将作为可绑定属性的名称，正如你所猜测的那样。在这种情况下，你还可以通过在选项对象上使用额外的`defaultValue`属性来指定属性的默认值。

为了说明这一点，让我们将之前的示例重构为通过在类本身上放置`bindable`装饰器来声明属性：

```js
import {bindable, bindingMode} from 'aurelia-framework'; 

@bindable({ 
  name: 'text', 
  defaultValue: 'Some default text', 
  defaultBindingMode: bindingMode.twoWay, 
  changeHandler: 'onTextChanged' 
}) 
export class TextBlockCustomElement { 
} 

```

在这里，我们可以清楚地看到`TextBlockCustomElement`类中`text`属性已经完全消失。其整个声明都由类上的`bindable`装饰器处理。

## 更改处理方法

一个 HTML 行为可以为它的任何可绑定属性提供一个更改处理方法。当属性的值发生变化时，更改处理方法会自动被模板引擎调用。

除非使用`bindable`的`changeHandler`选项显式指定方法名，否则给定属性的更改处理方法的名称为属性名称后跟`Changed`。例如，名为`firstName`的属性的默认更改处理方法名称为`firstNameChanged`。

更改处理方法带有两个参数，第一个是属性的新值，第二个是之前的值。当然，由于处理程序在其属性发生变化后调用，更改处理程序内部可以直接使用属性而不是第一个参数：

```js
export class TextBlockCustomElement { 
  @bindable text; 

  textChanged(newValue, oldValue) { 
    //Here, newValue is equal to this.text 
  } 
} 

```

## 生命周期

所有 HTML 行为都遵循相同的生命周期。行为的视图模型可以实现以下任意回调方法，这些方法将由模板引擎在行为生命周期的特定时刻调用：

+   `created(owningView: View, view?: View)`：在行为创建后立即调用。`owningView`，即行为声明其中的`View`实例，作为第一个参数传递。另外，如果行为是一个具有视图的自定义元素，行为的`View`实例作为第二个参数传递。

+   `bind(bindingContext: Object, overrideContext: Object)`：在视图和视图模型绑定在一起后立即调用。周围的绑定上下文将作为第一个参数传递。一个暴露祖先上下文并且可以用来由视图模型添加上下文属性的覆盖上下文，作为第二个参数传递。

    ### 注意

    如果行为没有声明`bind`回调方法，那么在此阶段将调用视图模型的所有可绑定属性的更改处理程序，以便允许视图模型根据实例的绑定说明初始化其状态。然而，如果实现了`bind`，模板引擎在绑定过程中不会自动调用更改处理程序，在这种情况下，`bind`方法被认为负责初始化行为的状态。

+   `attached()`：在绑定视图已附加到 DOM 后立即调用。

+   `detached()`：在绑定视图从 DOM 中解除后立即调用。这发生在开始处理行为释放过程时。

+   `unbind()`：在视图模型从其视图中解绑后立即调用此方法。这标志着行为生命的结束。通常，如果`unbind`正确地完成其工作，并且没有遗漏任何引用和资源，那么在返回此方法后，视图模型实例可以被垃圾回收。

除了那些生命周期回调方法，任何可绑定属性的更改处理方法都可以实现。在行为实例的生命周期内，每当可绑定属性的值发生变化时，模板引擎都会调用相应的更改处理方法，如果已实现的话。

实际上，那些生命周期回调方法并不仅限于 HTML 行为。它们可以添加到任何 Aurelia 组件中，例如路由组件或组合组件。

# 自定义属性

自定义属性是可以通过向任何 HTML 元素（无论是原生元素还是自定义元素）添加相应的 HTML 属性来附加到 HTML 行为上的。Aurelia 的标准模板资源包括许多我们已经介绍过的自定义属性，如`focus`、`show`或`hide`。

自定义属性完全是行为性的，意味着它们没有视图。

通常有四种类型的自定义属性：

+   具有单值属性的属性

+   具有多个属性的属性

+   具有动态属性的属性

我们将在接下来的章节中详细介绍这些类型的属性。

## 声明一个自定义属性

标识一个类为自定义属性有两种方法。第一种是遵守命名约定，使自定义属性的类名以`CustomAttribute`结尾。

在这种情况下，类名的其余部分将被转换为破折号形式，并在模板中作为属性的名称。例如，一个名为`MySuperAttributeCustomAttribute`的类将在模板中作为`my-super-attribute`属性提供。

作为命名规则的替代方案，`customAttribute`装饰器可以应用于一个类，使其被模板引擎识别为自定义属性。在这种情况下，必须将属性在模板中提供的名称作为装饰器的第一个参数传递。例如，下面的属性将在模板中作为`file-drop-target`属性提供：

```js
import {customAttribute} from 'aurelia-framework'; 

@customAttribute('file-drop-target') 
export class WhateverNameYouWant { 
  //Omitted snippet...  
} 

```

当使用`customAttribute`装饰器并传递一个显式的属性名称时，社区中公认的好做法是坚持使用破折号模式，并将所有 HTML 行为的名称前缀为一个在应用程序、插件、框架或公司中通用的两位字母标识符。

例如，原本打算作为更大 Aurelia 界面项目一部分的`aurelia-dialog`插件，现在重新定义并范围缩减为 Aurelia UX，使用`ai-`前缀。我们在第四章，*表单，以及如何验证它们*中已经看到过，元素如`ai-dialog`和`ai-dialog-body`。

## 具有单值属性的属性

默认情况下，自定义属性有一个隐式的`value`属性，该属性是分配属性的值的地方。当然，可以实现一个名为`valueChanged`的变更处理方法，以便响应`value`属性的变化。

显然，一个属性可以没有任何值使用：

```js
<div my-attribute></div> 

```

在这种情况下，`value`属性将被分配一个空字符串。

最后，在声明一个单值属性时，`customAttribute`装饰器可以接受第二个参数，即属性的默认绑定模式。默认情况下，自定义属性是单向绑定的。然而，使用装饰器，可以覆盖这个约定。

例如，设想一个`file-drop-target`属性，它默认会进行双向绑定：

```js
import {customAttribute, bindingMode} from 'aurelia-framework'; 

@customAttribute('file-drop-target', bindingMode.twoWay) 
export class FileDropTarget { 
  //Omitted snippet...  
} 

```

### 添加图片预览

为了说明单值自定义属性是如何工作的，让我们创建一个。

在我们联系应用程序中，我们将在联系照片上传组件中添加所选图像的预览。为此，我们将利用浏览器提供的`URL.createObjectURL`函数，该函数接受一个`Blob`对象作为参数，并返回一个特殊的 URL，该 URL 指向这个资源。我们的自定义属性将主要用于`img`元素，将其与`Blob`对象绑定，从它生成一个对象 URL，并将这个 URL 分配给`img`元素的`src`属性。

### 注意

`URL.createObjectURL`函数被大多数主流浏览器支持，但仍然是对 File API 的实验性特性。Mozilla 开发者网络有一个很好的文档关于它，可以在[`developer.mozilla.org/en-US/docs/Web/API/URL/createObjectURL`](https://developer.mozilla.org/en-US/docs/Web/API/URL/createObjectURL)找到。

你可以说值转换器会更适合这种特性，我完全同意。值转换器可以接受一个`Blob`对象作为输入，并返回该对象的 URL。然后它可以在`img`元素的`src`属性和包含`Blob`对象的属性之间使用。

然而，在这种情况下，每个对象 URL 在使用后都必须释放，以防止内存泄漏，而值转换器没有任何机制在值不再使用时通知。相反，HTML 行为提供了更丰富的流程和更广泛的扩展点。这就是我们创建自定义属性的原因：

`src/resources/attributes/blob-src.js`

```js
import {inject, PLATFORM, DOM} from 'aurelia-framework'; 

const URL = PLATFORM.global.URL; 
const Blob = PLATFORM.global.Blob; 

@inject(DOM.Element) 
export class BlobSrcCustomAttribute { 

  constructor(element) { 
    this.element = element; 
  } 

  disposeObjectUrl() { 
    if (this.objectUrl && URL) { 
      this.element.src = ''; 
      URL.revokeObjectURL(this.objectUrl); 
      this.objectUrl = null; 
    } 
  } 

  valueChanged(value) { 
    this.disposeObjectUrl(); 

    if (Blob && URL && value instanceof Blob) { 
      this.objectUrl = URL.createObjectURL(value); 
      this.element.src = this.objectUrl; 
    } 
  } 

  unbind() { 
    this.disposeObjectUrl(); 
  } 
} 

```

在这里，我们依靠命名约定来识别我们的类作为自定义属性，并在构造函数中注入所属性。

我们为`PLATFORM`常量的`global`值检索`URL`对象和`Blob`类。当在浏览器中运行并使用`aurelia-pal-browser`实现时，这个`global`属性将引用`window`对象。它允许我们在调用其方法之前检查这些值是否可用。这样，如果应用程序在服务器端执行以渲染其 HTML，并且服务器上使用的 PAL 实现不提供这些 API，则自定义属性不会引发任何错误，并将`src`属性保持不变。

我们还使用`valueChanged`来释放任何先前的对象 URL，然后创建一个新的并分配给自定义属性所在的元素的`src`属性。

最后，`unbind`方法将在模板引擎解绑我们的自定义属性时调用，它只是释放当前的对象 URL，如果有的话。

### 注意

不要忘记在`resources`特性的`configure`函数中加载这个新属性，或者在下一个模板中添加一个`require`语句，在使用它之前加载属性。

接下来，让我们在我们的联系照片上传组件中使用这个自定义属性。首先，我们希望在选择有效的图像文件时只显示预览。这将防止显示损坏的图片。为此，我们将使用`aurelia-validation`库的`validation-errors`属性将当前验证错误分配给新的`errors`属性：

`src/contact-photo.html`

```js
<template> 
  <!-- Omitted snippet... --> 
  <input type="file" id="photo" accept="image/*" files.bind="photo & validate"  
    validation-errors.bind="photoErrors"> 
  <!-- Omitted snippet... --> 
</template> 

```

接下来，我们在视图模型上添加计算属性以获取预览的`File`对象：

`src/contact-photo.js`

```js
import {inject, NewInstance} from 'aurelia-framework'; 

//Omitted snippet... 
export class ContactPhoto { 
  //Omitted snippet... 

  get areFilesValid() { 
    return !this.errors || this.errors.length === 0; 
  }
get preview() { 
    return this.photo && this.photo.length > 0 && this.areFilesValid 
      ? this.photo.item(0) : null; 
  } 
  //Omitted snippet... 
} 

```

我们首先创建了一个`areFilesValid`属性，该属性使用了新的`photoErrors`属性，并确保`photo`属性没有验证错误。接下来，我们添加了一个`preview`属性，只有当`photo`包含至少一个项目并且有效时，它才会返回`photo`中的第一个文件。否则，它返回`null`。

现在，利用新的`preview`属性和我们的`blog-src`属性，我们可以显示所选图像文件的前缀：

`src/contact-photo.html`

```js
<template> 
  <!-- Omitted snippet... --> 
  <div class="col-sm-9"> 
    <input type="file" id="photo" accept="image/*"  
           files.bind="photo & validate"> 
    <div class="thumbnail" show.bind="preview"> 
      <img blob-src.bind="preview" alt="Preview"> 
    </div> 
  </div> 
  <!-- Omitted snippet... --> 
</template> 

```

在这里，我们简单地添加了一个`div`元素，当有预览时它才会显示。在这个`div`内部，我们添加了一个带有我们的`blob-src`自定义属性的`img`元素，该属性绑定到`preview`属性。

如果您在这个时候进行测试，您应该能够在选择有效的图像文件后看到预览。此外，当没有选择图像或选择无效时，预览应该自动隐藏。

### 添加文件拖放目标

在接下来的部分中，我们将添加一个自定义元素，它将作为一个文件选择器，支持使用对话框选择文件和拖放图像文件。为了为此做准备，让我们创建一个第二个自定义属性，它将监听其元素上的拖放事件，并将任何被拖放的文件分配给它的值：

`src/resources/attributes/file-drop-target.js`

```js
import {customAttribute, bindingMode, inject, DOM} from 'aurelia-framework'; 

@customAttribute('file-drop-target', bindingMode.twoWay) 
@inject(DOM.Element) 
export class FileDropTarget { 
  constructor(element) { 
    this.element = element; 
    this._onDragOver = this.onDragOver.bind(this); 
    this._onDrop = this.onDrop.bind(this); 
    this._onDragEnd = this.onDragEnd.bind(this); 
  } 

  attached() { 
    this.element.addEventListener('dragover', this._onDragOver); 
    this.element.addEventListener('drop', this._onDrop); 
    this.element.addEventListener('dragend', this._onDragEnd); 
  } 

  onDragOver(e) { 
    e.preventDefault(); 
  } 

  onDrop(e) { 
    e.preventDefault(); 
    this.value = e.dataTransfer.files; 
  } 

  onDragEnd(e) { 
    e.dataTransfer.clearData(); 
  } 

  detached() { 
    this.element.removeEventListener('dragend', this._onDragEnd); 
    this.element.removeEventListener('drop', this._onDrop); 
    this.element.removeEventListener('dragover', this._onDragOver); 
  } 
} 

```

在这里，我们首先声明了我们自定义的属性，使其默认使用双向绑定。这样，当用户将文件拖放到带有我们属性的元素上时，分配给值的字段列表也将分配给与之绑定的表达式。

### 注意

说这个属性使用双向绑定有点夸张。实际上，这个属性从未真正读取它所绑定的值；它只是写入它。但由于 Aurelia 不支持这种仅向外绑定的模式，我们必须使用双向绑定。您可能已经注意到`aurelia-validation`插件的`validation-errors`属性也是以同样的方式工作的。

我们还需要声明一个对其 DOM 元素的依赖，并通过构造函数来获取它。

当我们的属性被附加到文档时，我们在其元素上添加适当的事件监听器。当发生`drop`事件时，我们将被拖动的`files`分配给属性的`value`属性。

最后，当我们的属性从文档中分离时，我们将其元素上的事件监听器移除。

## 具有多个属性的属性

自定义属性可以声明可绑定的属性。在这种情况下，属性不再有一个单一的`value`属性，而是有任意数量明确命名的属性。

当然，这样的属性可以定义值变化处理方法，当它们各自的属性值发生变化时，模板引擎会调用这些方法。

例如，`aurelia-router`库导出的`route-href`属性可能定义如下：

```js
import {bindable} from 'aurelia-framework'; 

export class RouteHrefCustomAttribute { 
  @bindable route; 
  @bindable params; 
} 

```

### 使用带有多个属性的自定义属性

当使用带有多个属性的自定义属性时，语法与`style`属性类似，属性名后面跟着冒号和它的值，属性之间用分号分隔：

```js
<a route-href="route: my-route; params.bind: { id: 1 }">Link</a> 

```

在这里，属性实例的`route`属性将被赋值为`'my-route'`字符串，其`params`属性将被绑定到一个具有`id`属性等于 1 的对象上。

显然，在这种属性上，绑定并不应用于属性本身，而是应用于其属性。我们可以在前一个示例中看到这一点，其中`params`属性被绑定到一个对象上。

## 具有动态属性的属性

当属性需要具有动态属性，其名称在静态情况下不可知时，属性类应该用`dynamicOptions`进行装饰。

动态属性的值变化不使用标准的值变化处理方法进行通知。相反，属性必须实现一个`propertyChanged`方法，每当动态属性之一的值发生变化时，此方法将被调用，并将传递三个参数：属性的名称、其新值和旧值：

```js
import {dynamicOptions} from 'aurelia-framework'; 

@dynamicOptions 
export class BookCustomAttribute { 
  propertyChanged(name, newValue, oldValue) { 
    //React to the property change 
  } 
} 

```

### 使用带有动态属性的自定义属性

使用带有动态属性的自定义属性与使用具有多个静态属性的属性相同：

```js
<div book="title: Learning Aurelia; last-updated.bind: now"></div> 

```

在这里，`Book`属性实例将有一个`title`属性，其将被赋值为`'Learning Aurelia'`字符串，以及一个`lastUpdated`属性，其将被绑定到外部上下文的`now`属性。

# 自定义元素

自定义元素比自定义属性要复杂得多。一个自定义 HTML 元素具有以下属性：

+   它可以有可绑定的属性

+   它可以有自己的模板来控制它的渲染方式

+   它可以支持内容投射，因此用户可以将绑定的视图片段或自定义模板注入其中

+   它可以定义自己的行为

+   它可以与原生 DOM API 进行接口

此外，自定义元素可以通过多种方式进行自定义，主要是使用`aurelia-templating`提供的各种装饰器。我们将在接下来的章节中介绍这些可能性和扩展点。

需要理解的一个重要点是，自定义元素并不是通过模板技巧来替换它们的渲染模板来处理的。一个自定义元素是一个真实的 DOM 元素，这意味着它继承了所有 DOM 元素的属性和行为，并且可以用任何针对 DOM 元素的 API 来使用。

## 声明一个自定义元素

自定义元素的声明与自定义属性非常相似。按照约定，以`CustomElement`结尾的类被认为是自定义元素，其余的名称将被转换为短横线命名法，并用作模板中的元素名称。

例如，名为`TextBlockCustomElement`的类将在模板中作为`text-block`元素提供。

与自定义属性类似，`customElement`装饰器可以应用于一个类，作为命名规则的替代。在这种情况下，元素将在模板中以装饰器的第一个参数的形式提供名称。

例如，下面的元素将在模板中作为`text-block`元素可用：

```js
import {customElement} from 'aurelia-framework'; 

@customElement('text-block') 
export class WhateverNameYouWant { 
  //Omitted snippet...  
} 

```

### 注意

尽管 Aurelia 支持自定义元素使用单字名称，但建议坚持使用短横线命名法，并在自定义元素名称中使用至少两个单词。这是因为 Web 组件规范为所有单字名称保留了原生浏览器元素，所以将来不可能将这样的 Aurelia 自定义元素作为标准 Web 组件导出。此外，社区公认的良好实践是在你所有的 HTML 行为名称前加上一个两个字母的标识符，这个标识符在你的应用程序、插件、框架或公司中是共同的。

然而，这些都是只是约定，因为任何没有装饰器标识其类型的资源，如`valueConverter`、`bindingBehavior`或`customAttribute`，并且不匹配任何资源命名规则，如类名以`ValueConverter`、`BindingBehavior`或`CustomAttribute`结尾，将被模板引擎视为自定义元素。在这种情况下，类的全名将被转换为短横线命名法，并用作模板中的元素名称。

例如，作为一个资源的类被命名为`TextBlock`，它将作为`text-block`元素提供给模板。然而，遵循命名规则或使用装饰器被认为是最佳实践。

## 创建文件选择器

让我们通过在我们的联系人管理应用程序中创建名为`file-picker`的第一个自定义元素来直接进入。这个元素将封装一个`file input`，并将使用我们在上一节中创建的`file-drop-target`自定义属性，以便用户可以打开文件选择对话框或将文件拖放到元素上。

### 声明自定义元素

我们将从为我们的自定义元素添加一些 CSS 开始：

`src/resources/elements/file-picker.css`

```js
file-picker > label { 
  width: 100%; 
  height: 100%; 
  cursor: pointer; 
} 

file-picker > input[type=file] { 
  visibility: hidden; 
  width: 0; 
  height: 0; 
}  

```

在这里，我们简单地隐藏了`file input`并样式化了我们元素内的`label`。`label`将通过`for`属性链接到隐藏的`file input`，所以点击它将会打开输入的文件选择对话框，即使`input`不可见。这允许我们用更时尚的 UI 替换浏览器的`file input`。

接下来，让我们创建 JS 类：

`src/resources/elements/file-picker.js`

```js
import {bindable, bindingMode} from 'aurelia-framework'; 

export class FilePickerCustomElement { 

  @bindable inputId = ''; 
  @bindable accept = ''; 
  @bindable multiple = false; 
  @bindable({ defaultBindingMode: bindingMode.twoWay }) files; 
} 

```

这个类简单地定义了一些可绑定的属性，这些属性将在模板中使用。另外，由于`files`属性用于收集用户输入，它将默认双向绑定。这就是这个类存在的主要原因。实际上，如果没有需要默认使`files`使用双向绑定的需求，这可以是一个仅包含模板的定制元素，没有 JS 类，类似于我们在这个章节开始时所做的那样，使用我们的`contact-form`。

最后，我们需要构建模板：

`src/resources/elements/file-picker.html`

```js
<template> 
  <require from="./file-picker.css"></require> 

  <input type="file" id="${inputId}" accept="${accept}" multiple.bind="multiple" files.bind="files"> 
  <label for="${inputId}" file-drop-target.bind="files"> 
    <slot></slot> 
  </label> 
</template> 

```

在这里，我们首先`require`元素的 CSS 文件。接下来，我们添加一个`file input`，它将把一些属性绑定到视图模型的属性。这允许我们元素的使用者指定`input`的`id`以及它的`accept`和`multiple`属性。最重要的是，由于`files`属性绑定到`input`的`files`属性，用户选择的文件将与绑定到`files`属性的外层作用域的表达式同步。

### 注意

instead of leaving the `inputId` property empty by default, this element could implement some ID generation algorithm. This would make using the element a little simpler for other developers.

接下来，我们在`label`中添加一个`for`属性，也绑定到`inputId`属性。这将把`label`和`input`链接在一起，所以点击`label`会打开`input`的文件选择对话框。此外，我们把`file-drop-target`属性添加到这个`label`中，并绑定到`files`属性，所以拖放到这个`label`上的文件将被分配给`files`属性。

最后，我们在`label`内部添加一个`slot`。`slot`是影子 DOM 规范的一部分，它允许内容投射。我们将在后面的章节中详细介绍内容投射；现在需要记住的是，这个`slot`元素将被替换为`file-picker`元素实例的内容。

### 使用自定义元素

我们新的`file-picker`元素现在准备使用。当然，它需要在`resources`特性的`configure`函数中全局加载，或者在模板中使用时`require`：

`src/contact-photo.html`

```js
<template> 
  <!-- Omitted snippet... --> 
  <div class="form-group"> 
    <label class="col-sm-3 control-label" for="photo">Photo</label> 
    <div class="col-sm-6"> 
      <file-picker input-id="photo" accept="image/*" files.bind="photo & validate" class="thumbnail"> 
        <strong hide.bind="preview"> 
          Click to select a file or drag and drop one here 
        </strong> 
        <img show.bind="preview" blob-src.bind="preview" alt="Preview"> 
      </file-picker> 
    </div> 
  </div> 
  <!-- Omitted snippet... --> 
</template> 

```

在这里，我们用新的`file-picker`替换了原来的`file input`。我们指定`input-id`为`photo`，这使得我们的`file-picker`中的`file input`和上面的两行中的`label`链接在一起。

我们还指定`file-picker`的文件选择对话框应只显示图片文件，使用`accept`属性，并将`files`属性绑定到`photo`属性上。此外，这个绑定说明被`validate`绑定行为装饰，所以选中或拖放的文件将会得到适当的验证。

最后，我们利用内容投射，在`file-picker`内部注入一条`strong`文本，当没有`preview`可用时显示，以及一个`img`元素，当有`preview`可用时可见，并使用我们的`blob-src`自定义属性显示预览。这部分内容将在`file-picker`的 DOM 树中的`slot`位置投射。

如果你在这个时候运行应用程序，你应该能够点击`file-picker`来使用选择对话框选择文件，或者将图片文件拖放到元素上，如果文件有效，选中或拖放的文件应该会显示在预览区域。

## 验证自定义元素

由于`aurelia-validation`库验证任何双向绑定，`validate`绑定行为可以不用任何问题地用在自定义元素绑定上。实际上，我们在上一节中就是用它来验证我们的`file-picker`的，如果你没有注意到的话。然而，`file-picker`的验证正确工作的原因是因为我们的`contact-photo`组件使用了`change`作为`validateTrigger`。

为了使自定义元素与`blur``validateTrigger`无缝工作，自定义元素必须发布`blur`事件。另外，为了尊重所有原生表单相关元素实现的 API，如果您的元素的用户输入目的是表单相关元素，实现一个`focus`方法被认为是一个好习惯，它将委派焦点到包含的任何表单相关元素。

为了说明这一点，让我们想象一个`my-widget`自定义元素，它包含一个`input`元素，像这样：

```js
<template> 
  <input value.bind="value"> 
</template> 

```

我们也想象一个非常基础的视图模型：

```js
import {bindable} from 'aurelia-framework'; 

export class MyWidgetCustomElement { 
  @bindable value; 
} 

```

为了符合`aurelia-validation`的要求，这个模板必须进行修改：

```js
<template> 
  <input value.bind="value" ref="input" blur.delegate="blur()"> 
</template> 

```

在这里，我们在视图模型上创建了一个名为`input`的新属性，它将包含`input`元素的引用。接下来，我们为`blur`事件添加了一个委派事件处理程序，当触发时会调用`blur`方法。

接下来，让我们修改视图模型以实现新要求：

```js
import {inject, DOM, bindable} from 'aurelia-framework'; 

@inject(DOM.Element) 
export class MyWidgetCustomElement { 
  @bindable value; 

  constructor(element) { 
    this.element = element; 
    element.focus = () => this.input.focus(); 
  }
blur() { 
    this.element.dispatchEvent(DOM.createCustomEvent('blur')); 
  } 
} 

```

在这里，我们首先声明了对 DOM 元素本身的依赖，并在构造函数中注入。另外，我们在`my-widget`元素上定义了一个`focus`方法，当调用时它会调用`input`的`focus`方法。最后，我们创建了一个`blur`方法，当调用时会在`my-widget`元素上创建并分发一个`blur`事件。

现在`my-widget`元素可以使用默认的`blur``validateTrigger`。

至于我们的`file-picker`，为了使其与`blur``validateTrigger`一起工作，应该修改它，使其在选择文件或拖放文件时发布一个`blur`事件。尽管该元素没有可聚焦的内容，因为`file input`是隐藏的，但每次其值发生变化时发布此类事件将基本上强制它使用`change validateTrigger`进行验证，即使验证控制器的触发器是`blur`。通过实现一个`filesChanged`变更处理方法，分派一个`blur`事件，可以轻松实现这一点。

实现一个`focus`方法不那么直接。因为它不包含任何可聚焦的元素，它应该做什么呢？一种可能性是在`file-picker`获得焦点时打开文件选择对话框，尽管这从用户的角度来看可能有点侵扰性。这样做只是调用`file input`的`click`方法。

一旦完成这个操作，可以将验证控制器分配的`validateTrigger`从`contact-photo`组件中移除，使其恢复到默认的`blur`触发器。

由于除了保持一致性和提高`file-picker`的重用性外，没有增加太多内容，我将留给读者作为练习来应用这些更改。本书完成的应用程序示例可以作为参考。

## 代理行为

代理行为允许自定义元素在其自身上声明属性、事件处理程序和绑定。这是通过将这些代理行为添加到自定义元素的`template`元素来实现的，模板引擎将把这些行为投射到元素本身。它特别有用，可以在元素上定义`aria`属性，以添加可访问性。

### 注意

以下代码片段摘自`chapter-5/samples/surrogate-behaviors`示例。

例如，设想一个名为`tree-view`的自定义元素，它渲染一个树形结构。在其模板中，我们可以定义一个代理`role`属性，例如：

```js
<template role="tree"> 
  <!-- Omitted snippet... --> 
</template> 

```

当在模板中使用`tree-view`元素时，此`role="tree"`属性将添加到元素的每个实例：

```js
<tree-view></tree-view> 

```

如前例所示使用时，该元素在 DOM 中渲染后看起来像以下样子：

```js
<tree-view role="tree"></tree-view> 

```

代理行为也可以是事件处理程序。例如，`tree-view`可以声明一个这样的代理`click`处理程序：

```js
<template role="tree" click.delegate="click()"> 
  <!-- Omitted snippet... --> 
</template> 

```

在这种情况下，当点击`tree-view`元素本身时，会调用`click`处理函数。

正如这个例子所暗示的，代理属性也可以使用数据绑定：

```js
<template role="${role}" click.delegate="click()"> 
  <!-- Omitted snippet... --> 
</template> 

```

在这里，投影在`tree-view`元素上的`role`属性将绑定到自定义元素绑定上下文上的`role`属性。

## 内容投射

内容投影是将内容注入自定义元素的动作。通过定义投影点，自定义元素允许实例将外部的 DOM 子树注入到它自己的 DOM 中。这一机制是 Shadow DOM 1.0 规范的一部分，也是大型 web 组件 growing standard 的一部分。

### 默认插槽

自定义元素中的一个投影点被称为插槽。插槽使用`slot`元素定义。我们已经用了一个，当我们之前构建`file-picker`元素在我们的联系人管理应用程序中时：

`src/resources/elements/file-picker.html`

```js
<template> 
  <require from="./file-picker.css"></require> 

  <input type="file" id="${inputId}" accept="${accept}" multiple.bind="multiple" files.bind="files"> 
  <label for="${inputId}" file-drop-target.bind="files"> 
    <slot></slot> 
  </label> 
</template> 

```

自定义元素可以定义一个单一的、未命名的插槽，作为默认插槽。使用这个元素时，元素的内容将被投影到这个默认插槽上。

我们在`contact-photo`组件中像这样使用了`file-picker`：

```js
<file-picker input-id="photo" accept="image/*" files.bind="photo & validate" class="thumbnail"> 
  <strong hide.bind="preview"> 
    Click to select a file or drag and drop one here 
  </strong> 
  <img show.bind="preview" blob-src.bind="preview" alt="Preview"> 
</file-picker> 

```

投影后的 DOM 结果看起来像这样：

```js
<file-picker input-id="photo" accept="image/*" files.bind="photo & validate" class="thumbnail"> 
  <input type="file" id="${inputId}" accept="${accept}" multiple.bind="multiple" files.bind="files"> 
  <label for="${inputId}" file-drop-target.bind="files"> 
    <strong hide.bind="preview"> 
      Click to select a file or drag and drop one here 
    </strong> 
    <img show.bind="preview" blob-src.bind="preview" alt="Preview"> 
  </label> 
</file-picker> 

```

在这里，我们可以清楚地看到，`file-picker`实例的内容，`strong`和`img`元素已经注入到元素的 DOM 中，并替换了`slot`元素。

### 命名插槽

自定义元素可以声明多个投影点，通过定义多个不同名称的`slot`元素。

例如，让我们想象一下，我们想要创建一个`submit-button`自定义元素，其模板看起来像这样：

```js
<template> 
  <button type="submit" class="btn btn-primary"> 
    <slot name="icon"></slot> 
    <slot name="label"></slot> 
  </button> 
</template> 

```

使用这个元素时，我们现在将有两个插槽，分别命名为`icon`和`label`，我们可以将内容投影到它们中：

```js
<submit-button> 
  <i slot="icon" class="fa fa-floppy-o" aria-hidden="true"></i> 
  <span slot="label">Update ${contact.fullName}</span> 
</submit-button> 

```

要将内容投影到命名插槽中，你只需要给想要投影的元素添加一个`slot`属性，其值为插槽的名称。在这里，我们将一个`i`元素投影到`icon`插槽上，并将一个包含按钮标签的`span`元素投影到`label`插槽上。

此外，如果多个内容元素使用`slot`属性的相同值，它们都将被投影到这个插槽中，按照它们在自定义元素实例中声明的顺序：

```js
<submit-button> 
  <i slot="icon" class="fa fa-floppy-o" aria-hidden="true"></i> 
  <span slot="label">Update</span> 
  <span slot="label">${contact.fullName}</span> 
</submit-button> 

```

在这里，这两个`span`元素都将被投影到`label`插槽中，顺序相同。

### 数据绑定投影内容

模板引擎将首先处理投影前的内容，所以使用字符串插值或绑定命令在或投影元素内部是完全合法的。前一个示例说明了这一点，通过使用字符串插值来渲染`contact`的`fullName`，在`label`插槽上的`span`之前。

在投影发生之前，内容是数据绑定的。这意味着内容是使用元素实例周围的上下文进行绑定的。它无法访问自定义元素的内层上下文。

在前一个示例中，`submit-button`的视图模型不知道任何`contact`属性。这个属性只存在于外层上下文中，在该上下文中声明了`submit-button`实例。

### 默认内容

当定义一个插槽时，自定义元素可以为其提供默认内容。这样，如果没有内容被投影到插槽上，它就不会留空。

为了说明这一点，让我们来改变前一个部分中的`submit-button`自定义元素：

```js
<template> 
  <button type="submit" class="btn btn-primary"> 
    <slot name="icon"> 
      <i class="fa fa-check-circle-o" aria-hidden="true"></i> 
    </slot> 
    <slot name="label">Submit</slot> 
  </button> 
</template> 

```

在这里，我们只是在`icon`插槽上添加了一个检查图标，在`label`插槽上添加了`Submit`文本。这样，如果`submit-button`实例没有在任何插槽上投影内容，按钮将显示默认图标和标签。

只有在没有内容投影到插槽时，默认插槽的内容才会显示。这意味着，为了覆盖默认内容并强制一个空插槽，你只需在插槽上投影一个空元素：

```js
<submit-button> 
  <span slot="icon"></span> 
</submit-button> 

```

在这里，一个空`span`将被投影到`icon`插槽上，这将覆盖默认图标。

### 插槽中套插槽

一个有趣的可能性是在另一个插槽的默认内容中定义插槽。在这种情况下，可以选择完全覆盖第一个插槽的内容，或者仅覆盖第二个插槽的内容并保留第一个插槽的其余默认内容。

让我们通过修改前一个示例中的`submit-button`元素来说明这一点：

```js
<template> 
  <button type="submit" class="btn btn-primary"> 
    <slot name="content"> 
      <slot name="icon"> 
        <i class="fa fa-check-circle-o" aria-hidden="true"></i> 
      </slot> 
      <slot name="label">Submit</slot> 
    </slot> 
  </button> 
</template> 

```

在这里，我们将之前定义的插槽包裹在一个名为`content`的新插槽中。所有之前的使用示例仍然有效；然而，现在可以通过`content`插槽完全覆盖`submit-button`的内容：

```js
<submit-button> 
  <span slot="content">Save</span> 
</submit-button> 

```

在这里，我们简单地将整个内容替换为一个包含文本`Save`的`span`元素。

### 命名插槽与默认插槽的混合

在一个给定的自定义元素内部，也可以定义命名插槽和默认未命名插槽。在这种情况下，所有在命名插槽之外的内容都将被投影到默认插槽中。

让我们通过让`label`插槽成为`submit-button`的默认未命名插槽来说明这一点：

```js
<template> 
  <button type="submit" class="btn btn-primary"> 
    <slot name="content"> 
      <slot name="icon"> 
        <i class="fa fa-check-circle-o" aria-hidden="true"></i> 
      </slot> 
      <slot>Submit</slot> 
    </slot> 
  </button> 
</template> 

```

经过这个更改，我们仍然可以像以前一样覆盖`content`或`icon`插槽。然而，要覆盖`label`，现在只需在元素实例中投影内容，而不需要任何插槽名称：

```js
<submit-button>Save</submit-button> 

```

这个`submit-button`实例覆盖了按钮的标签，该标签由默认的未命名插槽定义。

可以在命名插槽和默认插槽上混合投影：

```js
<submit-button> 
  <i slot="icon" class="fa fa-check-square-o" aria-hidden="true"></i> 
  Save 
</submit-button> 

```

在这里，我们在`icon`插槽上投影了一个带有不同图标的`I`元素，并在默认插槽上投影了文本`Save`。

### 插槽 ception

那么一个声明插槽的自定义元素，它在另一个声明自己插槽的自定义元素中被投影呢？这是完全可能的。让我们想象一个`form-button-bar`组件，它将封装一个`submit-button`按钮，还有一个**取消**按钮：

```js
<template bindable="cancelUrl"> 
  <div class="form-group"> 
    <div class="col-sm-9 col-sm-offset-3"> 
      <submit-button> 
        <slot name="submit-label" slot="label">Save</slot> 
      </submit-button> 
      <a class="btn btn-danger" href.bind="cancelUrl"> 
        <slot name="cancel-label">Cancel</slot> 
      </a> 
    </div> 
  </div> 
</template> 

```

在这里，`form-button-bar`元素声明了两个插槽，分别命名为`submit-label`和`cancel-label`，它们的默认内容分别是`Save`和`Cancel`。此外，`submit-label`插槽又被投影到了`submit-button`的`label`插槽上。当使用时，如果`form-button-bar`实例没有在`submit-label`插槽上投影任何内容，它的默认内容将被投影到`submit-button`的`label`插槽上。

这意味着`submit-button`的`label`插槽的默认内容将总是被覆盖，要么是被`form-button-bar`的`submit-label`插槽的默认内容覆盖，要么是被其投射的内容覆盖。

这也意味着，在使用`form-button-bar`元素时，没有办法在`submit-button`的`icon`插槽中投射内容，因为它没有被`form-button-bar`的插槽暴露出来。

### 限制

插槽机制的实现有几个重要的限制。插槽声明上的`name`属性不能绑定，元素实例上的`slot`属性也不能绑定。这包括字符串插值。这些属性的值必须是静态的。

此外，`slot`定义不能被模板控制器修改，例如`if`和`repeat`属性。`if`属性的限制可以通过在另一个元素上放置一个`show`属性来某种程度上绕过，这个元素包围着`slot`。然而，`repeat`属性就是不能工作，因为，由于插槽名称不可绑定且必须是静态的，重复插槽意味着有多个具有相同名称的插槽，这是不支持的。

奥雷利亚团队宣布，他们打算在未来至少解除一些这些限制，但在撰写本文时，这些限制仍然存在。

## 模板注入

还有一种方法可以扩展自定义元素的渲染。除了内容投射之外，自定义元素还可以在其模板中声明可替换的模板部分。这些可替换的部分随后可以被实例覆盖。

这种技术与其他插槽的主要区别在于绑定方式。虽然插槽上注入的内容在投射之前绑定，因此使用外部上下文进行绑定，但注入的模板在注入后绑定。这意味着注入的模板使用自定义元素的内部上下文进行绑定。因此，注入的模板可以无需任何问题地重复。

### 创建分组列表

让我们通过从联系人列表中提取可重用组件来说明这一点。我们将创建一个`group-list`自定义元素，它将对其绑定的项目进行分组和排序，以渲染项目组。它将定义一个可替换的部分，该部分将用于渲染组内的单个项目：

`src/resources/elements/group-list.html`

```js
<template bindable="items, groupBy, orderBy"> 
  <div repeat.for="group of items | groupBy:groupBy | orderBy:'key'" class="panel panel-default"> 
    <div class="panel-heading">${group.key}</div> 
    <ul class="list-group"> 
      <li repeat.for="item of group.items | orderBy:orderBy" class="list-group-item"> 
        <template replaceable part="item"></template> 
      </li> 
    </ul> 
  </div> 
</template> 

```

在这里，我们首先在`template`元素上定义可绑定的属性。这意味着`group-list`元素将只由这个模板组成；它将没有任何视图模型。

可绑定的属性如下：

+   `items`：要渲染的项目

+   `groupBy`：用于分组项目的属性名称

+   `orderBy`：用于对分组项目进行排序的属性名称

接下来，我们简单地重用来自`contact-list`组件的相同模板来渲染项目组。主要区别是，我们没有硬编码传递给`groupBy`和`orderBy`值转换器的属性，而是使用适当的可绑定属性。

最后，在模板中渲染联系人的位置，我们放置一个名为`item`的可替换模板部分。当使用这个自定义元素时，我们将能够注入一个模板，以替换这个部分。这个注入的模板将有权访问周围上下文，这意味着它将能够使用当前的`item`。

### 使用分组列表

让我们看看如何通过重构`contact-list`组件以使用这个新的`group-list`元素来使用具有可替换部分的自定义元素：

`src/contact-list.html`

```js
<template> 
  <!-- Omitted snippet... --> 
  <group-list items.bind="contacts | filterBy:filter:'firstName':'lastName':'company'" 
              group-by="firstLetter" order-by="fullName"> 
    <template replace-part="item"> 
      <a route-href="route: contact-details; params.bind: { id: item.id }"> 
        <span if.bind="item.isPerson"> 
          ${item.firstName} <strong>${item.lastName}</strong> 
        </span> 
        <span if.bind="!item.isPerson"> 
          <strong>${item.company}</strong> 
        </span> 
      </a> 
    </template> 
  </group-list> 
</template> 

```

在这里，我们首先使用`group-list`自定义元素。不要忘记将其作为资源加载并将其`items`属性与根据用户搜索过滤的`contacts`数组绑定。我们还需要使用`group-by`和`order-by`属性指定用于分组的属性和排序属性。

接下来，我们定义一个模板来替换名为`item`的部分。在这个模板中，我们保留了用于渲染单个联系人项的视图。正如你所看到的，模板部分能够使用来自自定义元素自身模板中的`repeat.for`属性的`item`属性。

### 默认模板部分

在这个阶段，`group-list`的使用者需要替换`item`部分，否则项目根本不会被渲染。在声明可替换的模板部分时，可以定义其默认内容，当该部分没有被替换时将使用该默认内容。

让我们将`group-list`模板更改为默认将每个项目渲染为字符串：

`src/resources/elements/group-list.html`

```js
<template bindable="items, groupBy, orderBy"> 
  <!-- Omitted snippet... --> 
  <template replaceable part="item">${item}</template> 
  <!-- Omitted snippet... --> 
</template> 

```

在这里，我们定义了可替换的`item`部分的默认内容，它将简单地使用其`toString`方法渲染当前的`item`。这样，如果没有注入`item`部分，用户至少会看到一些东西，即使它只是`[object Object]`，这也是一个没有覆盖`toString`方法的对象的默认结果。

你可以尝试通过在`Contact`类中添加一个返回`fullName`属性的`toString`方法，并在`contact-list`组件中的`group-list`元素内注释掉注入的`item`模板部分来实现。分组列表现在应该简单地渲染每个联系人的`fullName`，而不带任何链接。

### 重新作用域绑定上下文

如目前所示，我们`group-list`自定义元素的使用者需要知道当前项在绑定上下文中名为`item`。使事情变得更容易的一个可能性是使用`with`属性并在`repeat.for`内重新作用域绑定上下文到当前的`item`：

`src/resources/elements/group-list.html`

```js
<template bindable="items, groupBy, orderBy"> 
  <!-- Omitted snippet... --> 
  <li repeat.for="item of group.items | orderBy:orderBy" class="list-group-item"> 
    <template with.bind="item"> 
      <template replaceable part="item">${$this}</template> 
    </template> 
  </li> 
  <!-- Omitted snippet... --> 
</template> 

```

我们不能在`li`上放置`with`属性，因为它已经包含了一个`repeat`属性。确实，单个元素不能包含多个模板控制器。

我们需要用另一个`template`元素包围可替换的模板，该元素包含`with`属性，我们将其绑定到当前`item`。在可替换的`item`模板内，我们将字符串插值中的`item`引用替换为对`$this`的引用。`$this`关键字指的是当前上下文本身，得益于`with`，这是当前`item`。这部分是可选的，因为当前上下文仍然从父上下文继承，这意味着通过上下文继承`item`仍然可用。实际上，`$this`和`item`都指的是当前项目。除非当前项目有其自己的`item`属性。在这种情况下，`$this`将指的是当前项目，而`item`将指的是当前项目的`item`属性。

由于`item`仍然在绑定上下文中可用，而`Contact`对象没有`item`属性，所以我们不需要更改`contact-list`模板中的任何内容。它仍然有效。然而，在重复的`li`上使用`with`意味着我们现在可以在`contact-list`中删除所有对`item`的引用：

`src/contact-list.html`

```js
<template> 
  <!-- Omitted snippet... --> 
  <template replace-part="item"> 
    <a route-href="route: contact-details; params.bind: { id: id }"> 
      <span if.bind="isPerson"> 
        ${firstName} <strong>${lastName}</strong> 
      </span> 
      <span if.bind="!isPerson"> 
        <strong>${company}</strong> 
      </span> 
    </a> 
  </template> 
  <!-- Omitted snippet... --> 
</template> 

```

现在，我们的`group-list`自定义元素更易于使用。使用它的开发者不需要知道上下文有一个`item`属性。他们可以简单地假设可替换模板部分的绑定上下文就是当前项目。

当然，这主要是关于品味的问题，但也涉及到一致性。如果你在一个应用程序或插件中开始在这种场景下使用`with`，你应该保持一致，并在所有其他类似情况下继续使用它。

### 创建列表编辑器

让我们看看另一个例子。我们将创建一个可重用的列表编辑器，我们可以在`contact-form`组件中使用它来编辑电话号码、电子邮件地址、地址和社会资料：

`src/resources/elements/list-editor.js`

```js
import {bindable} from 'aurelia-framework'; 

export class ListEditorCustomElement { 

  @bindable items = []; 
  @bindable addItem; 
} 

```

在这里，我们首先创建一个视图模型，在该模型上定义两个可绑定的属性：

+   `items`：要编辑的项目数组

+   `addItem`：用于将新项目添加到数组中的函数

### 注意

在重构`contact-form`之后，我们将能够从`Contact`类中删除`removePhoneNumber`、`removeEmailAddress`、`removeAddress`和`removeSocialProfile`方法，因为它们将被`list-editor`中的`removeItem`方法替换。

`list-editor`的模板如下所示：

`src/resources/elements/list-editor.html`

```js
<template> 
  <div class="form-group" repeat.for="item of items" > 
    <template with.bind="item"> 
      <template replaceable part="item"> 
        <div class="col-sm-2 col-sm-offset-1"> 
          <template replaceable part="label"></template> 
        </div> 
        <div class="col-sm-8"> 
          <template replaceable part="value">${$this}</template> 
        </div> 
        <div class="col-sm-1"> 
          <template replaceable part="remove-btn"> 
            <button type="button" class="btn btn-danger"  click.delegate="items.splice($index, 1)"> 
              <i class="fa fa-times"></i> 
            </button> 
          </template> 
        </div> 
      </template> 
    </template> 
  </div> 
  <div class="form-group" show.bind="addItem"> 
    <div class="col-sm-9 col-sm-offset-3"> 
      <button type="button" class="btn btn-primary" click.delegate="addItem()"> 
        <slot name="add-button-content"> 
          <i class="fa fa-plus-square-o"></i> 
          <slot name="add-button-label">Add</slot> 
        </slot> 
      </button> 
    </div> 
  </div> 
</template> 

```

此模板的全球布局与我们在第四章*表单及其验证方式*中创建的所有列表编辑器使用了相同的框架。我们先为每个`item`重复一个块，并使用`with`将上下文范围限制在此块内的当前`item`上。

在重复项目块内，我们首先声明一个可替换的`item`部分，它封装了单个项目的整个模板。作为这个部分的默认内容，我们使用了与联系表单其余部分相同的列设置。第一列包含一个空的 replaceable 部分，名为`label`。第二列包含一个名为`value`的可替换部分，如果它没有被替换，则将当前项目渲染为字符串。第三列包含一个名为`remove-btn`的可替换部分，其默认内容是一个**移除**按钮，当点击时从数组中删除项目。

在这里我们可以看到，就像插槽可以定义子插槽一样，可替换部分也可以定义其他可替换部分作为它们的默认内容。在`list-editor`中，它允许我们替换整个项目模板，或者只替换其中的一部分。这是一个非常强大的功能。

我们甚至可以在同一个自定义元素中使用可替换部分和插槽。这就是我们在这里所做的，最后一个块，在重复的项目之外，包含一个**添加**按钮，当点击时调用`addItem`函数。这个按钮包含一个名为`add-button-content`的第一个插槽。其默认内容是一个图标，还有一个名为`add-button-label`的插槽，其默认内容是文本`Add`。这允许我们投影内容以自定义整个**添加**按钮的内容，或仅其标签。

最后，只有当`addItem`属性绑定到某个东西时（我们预期是一个函数），我们才显示包含**添加**按钮的整个块。这意味着，如果`list-editor`实例没有绑定`add-item`属性，**添加**按钮将不可见。

### 使用列表编辑器

我们现在可以在我们的`contact-form`组件中使用这个`list-editor`元素：

`src/contact-form.html`

```js
<template> 
  <!-- Omitted snippet... --> 
  <hr> 
  <list-editor items.bind="contact.phoneNumbers" add-item.call="contact.addPhoneNumber()"> 
    <template replace-part="label"> 
      <select value.bind="type & validate" class="form-control"> 
        <option value="Home">Home</option> 
        <option value="Office">Office</option> 
        <option value="Mobile">Mobile</option> 
        <option value="Other">Other</option> 
      </select> 
    </template> 
    <template replace-part="value"> 
      <input type="tel" class="form-control" placeholder="Phone number" value.bind="number & validate"> 
    </template> 
    <span slot="add-button-label">Add a phone number</span> 
  </list-editor> 
  <!-- Omitted snippet... --> 
</template> 

```

在这里，我们首先将电话号码编辑器重构为使用我们新的`list-editor`元素。我们将它的`items`属性绑定到`contact`的`phoneNumbers`属性。我们还绑定`add-item`属性以调用`contact`的`addPhoneNumber`方法。

接下来，我们将`label`模板部分替换为绑定到项目`type`的`select`元素。当然，这个绑定被`validate`装饰器装饰，所以项目的`type`将被正确验证。

我们还将`value`模板部分替换为绑定到项目`number`的`tel input`。同样，这个绑定被`validate`装饰器装饰，所以项目的`number`将被验证。

最后，我们在`add-button-label`插槽上投影文本`Add a phone number`。

此时，如果您运行应用程序，电话号码编辑器应该和之前一样的外观和行为。我将留给读者一个练习，即使用`list-editor`重构电子邮件地址、地址和社会资料的编辑器。本章的完整应用程序示例可以作为参考。

## 使用自定义装饰器

`aurelia-templating`库提供了许多装饰器，可以用来定制自定义元素的行为以及它们被模板引擎处理的方式。

### 注意

以下的大部分代码片段都来自`chapter-5/samples/element-decorators`示例。

### viewResources

`viewResources`装饰器可以用来声明视图依赖项。它就像`require`元素一样，但作用于组件的视图模型而不是其模板。

例如，我们可以通过从其模板中移除`require`语句来重构我们联系管理应用程序中的`contact-edition`组件：

`src/contact-edition.html`

```js
<template> 
  <!-- Comment out the require statement --> 
  <!-- <require from="contact-form.html"></require> --> 
  <!-- Omitted snippet... --> 
</template> 

```

然后，我们还需要用`viewResources`来装饰`ContactEdition`类：

`src/contact-edition.js`

```js
import {inject, NewInstance, viewResources} from 'aurelia-framework'; 

//Omitted snippet... 
@viewResources(['contact-form.html']) 
export class ContactEdition { 
  //Omitted snippet... 
} 

```

`contact-edition`组件仍然会像以前一样工作。

`viewResource`装饰器期望一个依赖项数组。每个依赖项可以是以下之一：

+   一个字符串，它必须是加载资源的路径

+   一个对象，有一个`src`属性，它必须包含要加载的资源的路径，和一个可选的`as`属性，如果存在，它将在模板中作为资源的别名，就像`require`元素的`as`属性一样

+   一个函数，它必须是加载资源的类

我实在想不出这个装饰器除了想要把所有依赖项放在视图模型中而不是视图里之外还有哪个好的用例。然而，由于加载依赖项是与模板相关的事务，对我来说，在视图中使用`require`语句来做这件事感觉更加自然。

### useView

`useView`装饰器可以用来明确指定自定义元素模板的路径。

例如，让我们更新我们联系管理应用程序中的`file-picker`元素，使其使用这个装饰器：

`src/resources/elements/file-picker.js`

```js
import {bindable, bindingMode, inject, DOM, useView} from 'aurelia-framework'; 

@inject(DOM.Element) 
@useView('./file-picker.html') 
export class FilePickerCustomElement { 
  //Omitted snippet... 
} 

```

如果多个元素共享同一个视图，这将会非常有用。另外，当一个元素打算在一个可重用的库或插件中分发时，明确指定元素的模板被认为是一个好的实践。确实，正如我们将在本章末尾看到的，使用你元素的开发者可以改变视图位置的约定。在这种情况下，依赖标准约定的元素将会被破坏。

### inlineView

`inlineView`装饰器允许我们完全用内联模板替换组件的模板文件：

```js
import {inlineView} from 'aurelia-framework'; 

@inlineView('<template><button type="submit">Submit</button></template>') 
export class SubmitButtonCustomElement { 
} 

```

这个自定义元素将没有`.html`文件，因为它的模板是声明内联的，位于 JS 类旁边。

这对于只是作为容器存在的自定义元素来说非常有用，它们主要依赖内容投射，因为它消除了需要一个包含很少几行的单独模板文件。

例如，这是来自`aurelia-dialog`库的`ai-dialog`元素的代码：

```js
import {customElement, inlineView} from 'aurelia-templating'; 

@customElement('ai-dialog') 
@inlineView('<template><slot></slot></template>') 
export class AiDialog { 
} 

```

这个元素的唯一目的是作为`ai-header`、`ai-body`和`ai-footer`元素的容器，因此当模板与视图模型相邻时，它会更简单。

### noView

`noView`装饰器告诉模板引擎给定的自定义元素没有模板。在这种情况下，模板引擎将简单地绑定元素本身，然后处理其内容（如果有），除非还使用了`processContent`装饰器并禁用了内容处理。

没有视图的自定义元素有用的情况相当罕见。对于我所能想到的绝大多数用例，例如封装 UI 库中 JS 小部件的行为，自定义属性更为合适。然而，可能存在一些场景，你希望将某些行为封装在一个完整的元素中，而不是在另一个元素的属性中。

为了举例说明，让我们想象一个作为 UI 库中 JS 小部件适配器的自定义元素。这个小部件是通过调用一个函数并将其 DOM 元素传递给它作为小部件视觉根来创建的：

```js
import {noView, inject, DOM} from 'aurelia-framework'; 

@noView 
@inject(DOM.Element) 
export class MyWidget { 
  constructor(element) { 
    this.element = element; 
  } 

  attached() { 
    SomeWidgetApi.create(this.element); 
  } 
} 

```

这样的元素不需要任何模板，因为元素的视图由外部库渲染。

与`viewResource`装饰器类似，`noView`装饰器可以作为其第一个参数传递一个依赖项数组。这些依赖项将与组件一起加载。

此外，第二个参数可以指定依赖项相对于的路径。在这种情况下，将使用此路径而不是视图模型的路径来定位依赖项。

### useViewStrategy

`useViewStrategy`装饰器告诉模板引擎使用给定的`ViewStrategy`实例来加载组件的视图。它实际上是由`useView`、`inlineView`和`noView`装饰器在幕后使用的。

它只是将提供的视图策略作为元数据附加到类上。在视图定位过程中，这个元数据然后由视图定位器检查并用于定位组件的视图。

它主要与自定义`ViewStrategy`实现一起使用，这是本书范围之外的高级主题。然而，了解一下它的存在是好的，以防你将来需要去那里。

### processAttributes

`processAttribute`装饰器可以用来提供一个函数，可以在模板引擎处理元素属性之前处理元素的属性。处理函数必须作为装饰器的参数传递：

```js
import {processAttributes} from 'aurelia-framework'; 

@processAttributes((compiler, resources, node, attributes, instruction) => { 
  //Omitted snippet... 
}) 
export class MyCustomElementCustomElement { 
  //Omitted snippet... 
} 

```

处理函数将被传递一堆参数：

+   `compiler`: 用于编译当前模板的`ViewCompiler`实例。

+   `resources`: 包含元素模板可用的资源集的`ViewResources`实例。

+   `node`: 自定义元素的 DOM 元素本身。

+   `attributes`: 一个`NamedNodeMap`实例，它是`node`参数的`attributes`属性。

+   `instruction`：包含模板引擎用来处理、数据绑定并显示自定义元素的所有信息的`BehaviorInstruction`实例。

### processContent

`processContent`装饰器可以用来控制模板引擎将如何以及是否处理自定义元素的內容。这取决于传递给装饰器的参数是什么。

如果装饰器被传递`false`，模板引擎将不会处理元素的內容。在这种情况下，元素负责处理自己的内容：

```js
import {noView, processContent} from 'aurelia-framework'; 

@noView 
@processContent(false) 
export class ProcessNoContentSampleCustomElement { 
  //Omitted snippet... 
} 

```

这样的元素不会看到其内容被模板引擎处理：

```js
<template> 
  <process-no-content-sample>${someProperty}</process-no-content-sample> 
</template> 

```

当渲染时，之前的模板将会精确地显示出来。字符串插值指令不会被解释，因为`process-no-content-sample`的内容不是由模板引擎处理的。`${someProperty}`文本将会保持不变。

另一个可能性是将一个处理函数传递给装饰器。在这种情况下，处理函数可以处理元素的內容，并且被期望返回`true`或`false`，告诉模板引擎在处理函数返回后是否应该处理内容：

```js
import {noView, processContent} from 'aurelia-framework'; 

@noView 
@processContent((compiler, resources, node, instruction) => { 
  //Omitted snippet... 
}) 
export class ProcessContentSampleCustomElement { 
  //Omitted snippet... 
} 

```

处理函数将被传递一堆参数：

+   `compiler`：用于编译当前模板的`ViewCompiler`实例。

+   `resources`：包含元素模板可用的资源集合的`ViewResources`实例。

+   `node`：自定义元素自身的 DOM 元素。

+   `instruction`：包含模板引擎用来处理、数据绑定并显示自定义元素的所有信息的`BehaviorInstruction`实例。

这个装饰器可以被用来，例如，创建一个作为 Aurelia 应用中集成点的自定义元素，以封装必须使用不同模板引擎的应用程序的部分。

### containerless

`containerless`装饰器向模板引擎指示，自定义元素的视图必须被注入到元素本身的位置，而不是在其内部：

```js
import {containerless} from 'aurelia-framework'; 

@containerless 
export class ContainerlessSample { 
  //Omitted snippet... 
} 

```

让我们假设这个`containerless-sample`元素有以下模板：

```js
<template> 
  <p>This is a containerless element example.</p> 
</template> 

```

这个元素会被这样使用：

```js
<div class="example"> 
  <containerless-sample></containerless-sample> 
</div> 

```

没有`containerless`装饰器，它会被渲染到 DOM 中，如下所示：

```js
<div class="example"> 
  <containerless-sample> 
    <p>This is a containerless element example.</p> 
  </containerless-sample> 
</div> 

```

然而，因为它被装饰了`containerless`，周围的`containerless-sample`元素不会被渲染：

```js
<div class="example"> 
  <p>This is a containerless element example.</p> 
</div> 

```

即使元素本身没有被渲染，自定义元素仍然可以声明可绑定的属性，并通过属性进行绑定。即使元素和它的属性没有被渲染到 DOM 上，这也会起作用。

当然，这意味着代理行为不能用于`containerless`自定义元素，因为代理行为应该被投影到的元素没有渲染。

这个装饰器在必须尊重特定 DOM 结构时非常有用，例如使用 SVG 元素时。

### useShadowDOM

`useShadowDOM` 装饰器将使自定义元素在其影子 DOM 中渲染其视图。这对于将自定义元素的 DOM 子树与文档的其余部分隔离很有用，以防止 CSS 或 DOM 查询在元素的 DOM 子树与外部世界之间产生不希望的交互。

为了说明这一点，让我们考虑我们联系管理应用程序中的`file-picker`自定义元素。这个元素有一个 CSS 文件，通过它的模板加载。没有影子 DOM，CSS 文件将被附加到文档的`head`中，这意味着 CSS 将全局应用于整个文档。可能发生冲突。

为了防止这一点，让我们让我们的`file-picker`元素在其影子 DOM 中渲染其视图。这样，其 CSS 文件将在其自己的影子根中加载，并且只在此有限的范围内应用：

`src/resources/elements/file-picker.js`

```js
import {bindable, bindingMode, inject, DOM, useView, useShadowDOM} from 'aurelia-framework'; 

@inject(DOM.Element) 
@useView('./file-picker.html') 
@useShadowDOM 
export class FilePickerCustomElement { 
  //Omitted snippet... 
} 

```

通过向我们的元素的类中添加`shadowDOM`装饰器，我们告诉模板引擎这个元素的内容应该在其自己的影子根内渲染。

为了让 CSS 文件在元素的影子根中渲染，我们需要将`require`语句标记为`scoped`：

`src/resources/elements/file-picker.html`

```js
<template> 
  <require from="./file-picker.css" as="scoped"></require> 
  <!-- Omitted snippet... --> 
</template> 

```

最后，由于 CSS 文件将在元素的影子根内加载，该影子根将在`file-picker`元素内但围绕其视图，我们需要从 CSS 选择器中删除`file-picker`元素，以便它们保持匹配：

`src/resources/elements/file-picker.css`：

```js
label { 
  width: 100%; 
  height: 100%; 
  cursor: pointer; 
} 

input[type=file] { 
  visibility: hidden; 
  width: 0; 
  height: 0; 
}  

```

注意`file-picker > label`选择器如何被`label`选择器替换。其他 CSS 规则的选择器也同理。

现在，如果您运行应用程序并检查围绕此元素的 DOM，您应该看到一个影子根，它封装了元素本身和一个包含 CSS 的`style`元素。您可能会注意到，在这里，`file-picker`内的`strong`和`img`元素投影到的内容位于影子根的外部。这很重要。这意味着元素的视图只受 CSS 文件的影响。其投影内容不受影响。如果您向投影内容添加了`label`，它将不会与`file-picker.css`中定义的规则匹配，因为它不会在相同的影子根上。

### `children`

`children`装饰器旨在用于自定义元素的属性上。它选择所有匹配提供的查询选择器的直接子元素并将它们作为数组分配给作为数组分配给装饰属性的数组。

为了说明这一点，让我们想象以下自定义元素：

```js
import {inlineView, children} from 'aurelia-framework'; 

@inlineView('<template><slot></slot></template>') 
export class ChildChildrenSampleCustomElement { 
  @children('item') items; 
} 

```

假设元素像这样使用：

```js
<child-children-sample> 
  <item repeat.for="value of values">${value}</item> 
</child-children-sample> 

```

在这里，`child-children-sample`实例将看到重复的`item`元素分配为其`items`属性数组。

此外，`items`的值将与匹配的元素集同步。这意味着，如果插入一个新的`item`元素或移除一个现有的元素，由于`repeat.for`绑定，`items`属性将被同步。

与可绑定属性类似，一个`children`属性也可以实现一个使用相同命名规则的变化处理方法，以响应变化。在这个例子中，如果存在`itemsChanged`方法，那么在渲染时，在属性初始化期间，以及每次`items`数组发生变化时，该方法将被调用。

### 孩子

`child`装饰器与`children`非常相似，只不过它针对的是一个单一的元素。

为了说明这个问题，让我们调整一下之前的例子：

```js
import {inlineView, child} from 'aurelia-framework'; 

@inlineView('<template><slot></slot></template>') 
export class ChildChildrenSampleCustomElement { 
  @child('header') header; 
} 

```

假设元素像这样使用：

```js
<child-children-sample> 
  <header>Some title</header> 
</child-children-sample> 

```

在这里，`child-children-sample`实例会将`header`元素分配给它的`header`属性。另外，如果存在`headerChanged`方法，那么在渲染时，在属性初始化期间，以及每次元素被移除、添加或替换时，该方法将被调用。

# 奖励 - 防止多次提交

目前，我们的联系人管理应用程序处理表单提交并不好。实际上，在`contact-creation`、`contact-edition`和`contact-photo`组件中，如果**保存**按钮点击一次，然后再次点击，在底层`Fetch`调用完成之前，路由导航离开表单之前，将同时执行后端的多项调用。有时候，这并不重要。然而，在许多场景下，这也可能是一个问题。

## 创建提交任务属性

为了解决这个问题，我们将创建一个名为`submit-task`的自定义属性，它将替代`form`元素的`submit`处理程序。它将使用`call`命令绑定到一个方法，该方法预期返回一个`Promise`。当`form`提交时，属性将切换一个标志，当返回的`Promise`完成时，它将再次切换回来。这个标志将指示表单是否正在等待一个提交任务完成：

`src/resources/attributes/submit-task.js`

```js
import {inject, DOM} from 'aurelia-framework'; 

@inject(DOM.Element) 
export class SubmitTaskCustomAttribute { 

  constructor(element) { 
    this.element = element; 
    this.onSubmit = this.trySubmit.bind(this); 
  } 

  attached() { 
    this.element.addEventListener('submit', this.onSubmit); 
    this.element.isSubmitTaskExecuting = false; 
  } 

  trySubmit(e) { 
    e.preventDefault(); 
    if (this.task) { 
      return; 
    } 

    this.element.isSubmitTaskExecuting = true; 
    this.task = Promise.resolve(this.value()).then( 
      () => this.completeTask(), 
      () => this.completeTask()); 
  } 

  completeTask() { 
    this.task = null; 
    this.element.isSubmitTaskExecuting = false; 
  } 

  detached() { 
    this.element.removeEventListener('submit', this.onSubmit); 
  } 
} 

```

在这里，我们首先使用命名约定来识别这个类作为一个自定义属性。我们还声明了对属性所在的 DOM 元素的依赖，我们在构造函数中注入这个元素。

在这里，当我们的自定义属性被`attached`到文档时，我们在元素的`submit`事件上添加一个监听器，当被触发时将调用`trySubmit`方法。另外，在元素上创建了一个新的`isSubmitTaskExecuting`属性，并初始化为`false`。

当元素发布一个`submit`事件时，我们首先要确保当前没有正在运行的提交`task`。如果已经有了，我们就直接返回。如果没有，元素的`isSubmitTaskExecuting`属性被设置为`true`，然后调用与自定义属性`value`绑定的函数。保证得到的结果是一个`Promise`，并且在这个`Promise`上附上一个回调，无论`Promise`成功还是失败，当`Promise`完成时，`isSubmitTaskExecuting`都会被设置回`false`。

最后，当属性从文档中`detached`时，我们简单地移除了`submit`事件监听器。

## 使用提交任务属性

现在我们可以进入带有`form`元素的各个组件，并将`submit`事件处理程序替换为新的`submit-task`属性，使用`call`命令将其绑定到`save`方法：

`src/contact-creation.html`

```js
<template> 
  <!-- Omitted snippet... --> 
  <form class="form-horizontal" validation-renderer="bootstrap-form" submit-task.call="save()"> 
    <!-- Omitted snippet... --> 
  </form> 
  <!-- Omitted snippet... --> 
</template> 

```

当然，为了让这正常工作，我们需要修改`save`方法，使其返回跟踪 Fetch 调用的`Promise`：

`src/contact-creation.js`

```js
//Omitted snippet... 
save() { 
  //Omitted snippet... 

  return this.contactGateway.create(this.contact) 
    .then(() => this.router.navigateToRoute('contacts')); 
} 
//Omitted snippet... 

```

我将留给读者作为练习，也将这些更改应用于`contact-edition`和`contact-photo`组件。

此时，如果你运行应用程序，当你已经进行一个提交任务时，你不应该能够触发多个提交。

## 创建提交按钮

另一个很好的功能是向用户显示一个视觉指示器，表明一个提交任务正在进行。现在我们已经有了一个自定义属性，用于创建和管理适当的标志，让我们创建一个`submit-button`自定义元素，当其表单正在提交时，显示一个旋转的动画图标：

`src/resources/elements/submit-button.html`

```js
<template bindable="disabled"> 
  <button type="submit" ref="button" disabled.bind="disabled" class="btn btn-success"> 
    <span hide.bind="button.form.isSubmitTaskExecuting"> 
      <slot name="icon"> 
        <i class="fa fa-check-circle-o" aria-hidden="true"></i> 
      </slot> 
    </span> 
    <i class="fa fa-spinner fa-spin" aria-hidden="true" show.bind="button.form.isSubmitTaskExecuting"></i> 
    <slot>Submit</slot> 
  </button> 
</template> 

```

在这里，我们首先在模板元素上声明了一个`disabled`可绑定属性。这意味着这个元素将由这个模板组成；它将没有视图模型。

接下来，我们声明了一个`button`元素，具有`submit`类型。我们还使用`ref`属性将这个按钮的引用分配给绑定上下文的`button`属性，并将按钮的`disabled`属性绑定到`disabled`可绑定属性。

在按钮内部，我们添加一个`span`，当按钮的`form`元素的`isSubmitTaskExecuting`属性为`true`时，将其隐藏。在这个`span`内部，我们定义一个`icon`插槽，其默认内容是一个复选标记。

我们还在按钮内部添加一个旋转图标，只有在按钮的`form`元素的`isSubmitTaskExecuting`属性为`true`时，才会显示。

最后，我们定义一个默认插槽，其中包含`Submit`文本作为其默认内容。

这个自定义元素将在没有提交任务进行时简单地显示一个复选标记，并在任何提交任务期间替换这个复选标记为一个旋转图标。然后在提交任务完成后，切换回复选标记。

此外，`icon`插槽将允许实例覆盖默认的复选标记，而未命名插槽将允许实例覆盖`Submit`标签。

## 使用提交按钮

现在我们可以进入带有`form`元素的各个组件，并将**Save**按钮替换为新的`submit-button`元素：

`src/contact-creation.html`

```js
<template> 
  <!-- Omitted snippet... --> 
  <submit-button>Save</submit-button> 
  <!-- Omitted snippet... --> 
</template> 

```

在这里，我们简单地定义了一个`submit-button`元素，并在默认插槽上投影了`Save`文本，这覆盖了其默认标签。

我将留给读者作为练习，也将这些更改应用于`contact-edition`和`contact-photo`组件。

此时，如果你运行应用程序，你应该看到各种**Save**按钮的复选标记被旋转图标替换，当一个提交任务进行时。

# 自定义视图位置策略

视图位置是定位给定组件的模板或视图的过程。按照约定，模板应该是一个位于视图模型相同的目录中的文件，除了扩展名应该是`.html`。

我们已经看到了一种自定义自定义元素视图位置过程的方法，使用诸如`useView`、`inlineView`和`noView`的装饰器。需要注意的是，使用这些装饰器并不仅限于自定义元素。它们可以用于任何 Aurelia 组件，如路由组件，或使用`compose`指令显示的组件。

然而，还有两种其他方式可以自定义视图位置策略。让我们逐一了解它们。

## 改变约定本身

整个应用程序的常规视图位置策略可以通过重写`ViewLocator`的`convertOriginToViewUrl`方法来改变。这意味着，默认情况下，应用程序中所有组件和自定义元素的观点将使用这个新策略来定位。

让我们想象我们想要改变这个约定。这应该在`main`模块的`configure`函数中完成：

`src/main.js`

```js
import {ViewLocator} from 'aurelia-framework'; 
//Omitted snippet... 

export function configure(aurelia) { 
  //Omitted snippet... 
  ViewLocator.prototype.convertOriginToViewUrl = origin => { 
    let moduleId = origin.moduleId; 
    let id = (moduleId.endsWith('.js') || moduleId.endsWith('.ts')) 
      ? moduleId.substring(0, moduleId.length - 3) 
      : moduleId; 
    return id + '.html'; 
  }; 
  //Omitted snippet... 
} 

```

在此，我们精确地重新实现了`aurelia-templating`中的`convertOriginToViewUrl`方法。这里的约定不会改变。然而，这给你一个很好的启示，了解你如何可以实现自己的视图位置逻辑。

### 注意

`convertOriginToViewUrl`方法接收一个`Origin`实例作为其参数。`Origin`类有一个`moduleId`属性，它包含导出组件视图模型类的 JS 文件的路径，还有一个`moduleMember`属性，它包含视图模型类从其 JS 文件导出的名称。

## 改变单个组件的策略

改变约定的替代方法是在组件或自定义元素级别指定视图位置策略。这可以通过我们之前看到的视图位置装饰器来实现，如`useView`、`inlineView`和`noView`。

然而，如果你不想依赖 Aurelia 导入给定组件或自定义元素，或者如果你不能使用装饰器，你也可以在视图模型上实现`getViewStrategy`方法。这个方法预期返回模板文件路径的字符串，或者一个`ViewStrategy`实例。

`aurelia-templating`库自带了几种视图策略实现，所有这些都在其对应的视图位置装饰器的背后使用：

+   `RelativeViewStrategy`：由`useView`装饰器使用。其构造函数期望与`useView`相同的参数。

+   `InlineViewStrategy`：由`inlineView`装饰器使用。其构造函数期望与`inlineView`相同的参数。

+   `NoViewStrategy`：由`noView`装饰器使用。其构造函数期望与`noView`相同的参数。

例如，我们可以在我们的联系人管理应用程序的`file-picker`自定义元素中移除`useView`装饰器，并使用`getViewStrategy`方法代替：

`src/resources/elements/file-picker.js`

```js
import {bindable, bindingMode, inject, DOM, useShadowDOM} from 'aurelia-framework'; 

@inject(DOM.Element) 
@useShadowDOM 
export class FilePickerCustomElement { 
  //Omitted snippet... 

  getViewStrategy() { 
    return './file-picker.html'; 
  } 
} 

```

在这里，我们成功地将`useView`从导入语句中移除。此外，我们用`getViewStrategy`方法替换了装饰器的使用，返回模板文件的路径。

# 总结

HTML 行为非常强大且灵活。它们为创建复杂且灵活的组件、专门针对单一应用程序的专用组件或可重用、完全可自定义的组件开辟了广阔的可能性，旨在作为第三方插件或框架分发。

它们还提供了一种很好的方法将第三方库集成到 Aurelia 中。我们将在第十一章，*与其他库集成*中看到如何做到这一点。由于 Aurelia 的模板 API 是开放的且易于使用，我们将能够定制和插入这些集成组件的渲染过程，以完成一些令人惊叹的事情。

但我们还没有达到这个阶段。在下一章中，我们将退后一步，好好看看我们的联系人管理应用程序。我们将思考我们所做的设计选择以及我们没有做出的选择，并看看我们如何可以使事情变得更好。我们还将讨论不同的方法来组织 Aurelia 应用程序，使其更加模块化、可测试和易于维护。
