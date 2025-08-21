# 第八章：国际化

当涉及到 JavaScript 国际化时，`i18next`是最知名、最广泛使用的库之一。它提供了一系列功能，如可插拔的翻译加载器、缓存、用户语言检测和复数形式。也许这就是 Aurelia 团队在它之上构建`aurelia-i18n`库的原因。

本章的目的并不是要详细解释`i18next`，而是更多地探索`aurelia-i18n`层本身。至于`i18next`的详细信息，官方网站有广泛的文档，如果你不熟悉它，我强烈建议你查阅： [`i18next.com/`](http://i18next.com/)。

# 设置事情

`aurelia-i18n`库和底层`i18next`库在使用之前都需要安装和配置。让我们看看这个过程如何进行。

## 安装库

首先，需要通过在项目目录中打开控制台并运行以下命令来安装`aurelia-i18n`和`i18next`：

```js
> npm install aurelia-i18n i18next --save

```

`i18next`库使用一个抽象层来加载翻译数据。在`i18next`术语中，这被称为后端。这个抽象层允许不同的翻译加载策略。

存储和检索翻译数据的最常见方法是在应用程序文件的某个地方使用 JSON 文件。因此，我们将安装`i18next-xhr-backend`实现，它使用`XMLHttpRequest`从服务器获取包含翻译的 JSON 文件：

```js
> npm install i18next-xhr-backend --save

```

当然，打包器需要知道这些新库。因此，在`aurelia_project/aurelia.json`文件中，在`build`部分，在`bundles`下的`vendor-bundle.js`的`dependencies`中，让我们添加以下条目：

```js
{ 
  "name": "aurelia-i18n", 
  "path": "../node_modules/aurelia-i18n/dist/amd", 
  "main": "aurelia-i18n" 
}, 
{ 
  "name": "i18next", 
  "path": "../node_modules/i18next/dist/umd", 
  "main": "i18next" 
}, 
{ 
  "name": "i18next-xhr-backend", 
  "path": "../node_modules/i18next-xhr-backend/dist/umd", 
  "main": "i18nextXHRBackend" 
}, 

```

## 配置插件

我们还需要在我们的主`configure`函数中加载和配置插件：

`src/main.js`

```js
import Backend from 'i18next-xhr-backend'; 
//Omitted snippet... 
export function configure(aurelia) { 
  aurelia.use 
    .standardConfiguration() 
    .feature('validation') 
    .feature('resources') 
    .feature('contacts') 
 .plugin('aurelia-i18n', (i18n) => { 
      i18n.i18next.use(Backend); 

      return i18n.setup({ 
        backend: { 
          loadPath: './locales/{{lng}}/{{ns}}.json',  
        }, 
        lng : 'en', 
        fallbackLng : 'en', 
        debug : environment.debug 
      }); 
    }); 
  //Omitted snippet... 
}); 

```

在此，我们首先从`i18next-xhr-backend`库中导入`Backend`类。然后，我们调用`plugin`函数来加载`aurelia-i18n`并对其进行配置。

配置函数接收`aurelia-i18n`类的单个实例`I18N`，作为外观，分组和标准化 API。它首先告诉`i18next`使用`i18next-xhr-backend`的`Backend`类，该类负责从服务器获取 JSON 翻译文件。然后，它调用`I18N`类的`setup`方法，带有一组选项。这些选项将用于配置插件，但也将用于后台配置`i18next`。这意味着您通常会传递给`i18next`的`init`方法的任何选项，都可以传递给这个`setup`方法。

以下是最重要的选项：

+   `backend.loadPath`：用于加载翻译文件的路径。`{{lng}}`占位符将被替换为必须加载翻译的语言，`{{ns}}`占位符将被替换为必须加载翻译的命名空间。

+   `lng`：默认语言。

+   `fallbackLng`：如果在当前语言中找不到给定键，则回退到该语言。

+   `debug`：设置为`true`时，浏览器控制台中的日志将更加详细。

## 创建翻译文件

`i18next`库允许我们将翻译按命名空间隔离，这些命名空间是逻辑翻译组。其默认命名空间名为`translation`。如果我们看看`backend.loadPath`选项，我们可以很容易地看出我们的翻译文件应该放在哪里：

`locales/en/translation.json`

```js
{} 

```

在这里，我们简单地创建一个包含空对象的 JSON 文件。我们稍后向其中添加翻译。

## 填充 Intl API

`aurelia-i18n`插件使用`i18next`进行翻译，但依赖原生 Intl API 进行一些其他任务，如数字和日期格式化。然而，一些浏览器（主要是移动浏览器）还不支持这个 API。因此，如果您想要支持这些浏览器，可能需要添加一个填充物。 [`github.com/andyearnshaw/Intl.js/`](https://github.com/andyearnshaw/Intl.js/) 是官方文档中推荐的一个。

# 获取和设置当前区域设置

除了各种视图资源，我们将在本章后面看到，`aurelia-i18n`还导出一个`I18N`类，它作为各种 API（如`i18next`和原生 Intl API）的门面。

让我们看看我们如何使用这个 API 来获取和设置当前区域设置，通过创建一个`locale-picker`自定义元素，用户可以更改当前区域设置：

`src/resources/elements/locale-picker.html`

```js
<template> 
  <select class="navbar-btn form-control"  
          value.bind="selectedLocale"  
          disabled.bind="isChangingLocale"> 
    <option repeat.for="locale of locales" value.bind="locale"> 
      ${locale} 
    </option> 
  </select> 
</template> 

```

在此模板中，我们首先添加一个`select`元素，其值将绑定到`selectedLocale`属性，当`isChangingLocale`属性为`true`时，该元素将被禁用。在`select`元素中，我们为`locales`数组中的每个值渲染一个`option`。每个`option`的`value`绑定到其`locale`值，每个选项的文本将是本身使用字符串插值表达式渲染的`locale`。

接下来，我们需要添加视图模型，这将使这个模板与`I18N` API 相连接：

`src/resources/elements/locale-picker.js`

```js
import {inject, bindable} from 'aurelia-framework'; 
import {I18N} from 'aurelia-i18n'; 

@inject(I18N) 
export class LocalePickerCustomElement { 

  @bindable selectedLocale; 
  @bindable locales = ['en', 'fr']; 

  constructor(i18n) { 
    this.i18n = i18n; 

    this.selectedLocale = this.i18n.getLocale(); 
    this.isChangingLocale = false; 
  } 

  selectedLocaleChanged() { 
    this.isChangingLocale = true; 
    this.i18n.setLocale(this.selectedLocale).then(() => { 
      this.isChangingLocale = false; 
    }); 
  } 
} 

```

首先，这个类的构造函数从接收`I18N`实例开始，然后使用其`getLocale`方法检索当前区域设置并初始化`selectedLocale`属性。由于这个属性是可绑定的，所以声明实例的模板可以对其默认值进行数据绑定。

接下来，属性更改处理程序`selectedLocaleChanged`将在`selectedLocale`属性发生变化时由模板引擎调用，将`isChangingLocale`设置为`true`，以便禁用`select`元素，然后调用`I18N`的`setLocale`方法。由于它可能需要从服务器加载新的翻译文件，所以这个方法是异步的，返回一个`Promise`，我们监听其完成以将`isChangingLocale`恢复为`false`，以便重新启用`select`元素。

由于我们的本地化选择器默认支持英语和法语，因此我们需要为法语添加另一个翻译文件，其中包含一个空对象：

`locales/fr/translation.json`

```js
{} 

```

我们现在可以使用这个自定义元素在`app`组件中：

`src/app.html`

```js
<!-- Omitted snippet...--> 
<form class="navbar-search pull-right"> 
  <locale-picker></locale-picker> 
</form> 
<ul class="nav navbar-nav navbar-right"> 
  <!-- Omitted snippet...--> 
</ul> 
<!-- Omitted snippet...--> 

```

当然，如果你在这个时候运行应用程序，当你改变当前的本地化设置时，什么也不会被翻译；必须首先向模板中添加文本翻译。

# 翻译

`aurelia-i18n`库提供了许多不同的翻译文本的方法。在本节中，我们将了解我们的选择有哪些。

## 使用属性

在模板中翻译文本的最简单方法是使用名为`t`的翻译属性。让我们通过翻译我们的**未找到**页面来说明这一点。

我们将从将文本移动到翻译文件开始：

`locales/en/translation.js`

```js
{ 
  "404": { 
    "explanation": "The page cannot be found.", 
    "title": "Something is broken..." 
  } 
} 

```

`locales/fr/translation.js`

```js
{ 
  "404": { 
    "explanation": "La page est introuvable.", 
    "title": "Quelque-chose ne fonctionne pas..." 
  } 
} 

```

正如你所见，由于翻译是 JSON 结构，我们完全可以没有任何问题地使用嵌套键。

要在元素内静态显示翻译后的文本，你只需要向元素添加`t`属性，并将其值设置为翻译键的路径：

`src/not-found.html`

```js
<template> 
  <h1 t="404.title"></h1> 
  <p t="404.explanation"></p> 
</template> 

```

渲染后，属性将在当前本地化的翻译文件中查找键，并将翻译值分配给元素的文本内容。如果当前本地化是英语，渲染后的 DOM 将看起来像这样：

```js
<h1 t="404.title">The page cannot be found.</h1> 
<p t="404.explanation">Something is broken...</p> 

```

也可以使用`t`来翻译属性的值：

```js
<input type="text" value.bind="contact.firstName"  
       t="[placeholder]contacts.firstName"> 

```

通过在方括号内加上属性的名称来前缀键，`t`属性将为这个属性分配翻译值，而不是元素的文本内容。在这里，翻译键`contacts.firstName`的值将被分配给`input`的`placeholder`属性。

此外，可以在单个元素上翻译多个目标，通过用分号分隔指令来实现：

```js
<label t="[title] help; text"> 

```

在这里，`help`键的值将被分配给`title`属性，`text`的值将被分配给元素的文本内容。当然，使用相同的技术翻译多个属性也是可能的。

最后，`t`属性监控当前的本地化设置。当它改变时，输出会自动使用新的本地化设置进行更新。

### 传递参数

由于`i18next`支持向翻译传递参数，你可以将对象绑定到`t-params`属性以传递翻译的参数。

让我们想象一下以下的翻译：

```js
{ "message": "Hi {{name}}, welcome back!" } 

```

使用属性将`name`参数传递给这个翻译看起来像这样：

```js
<p t="message" t-params.bind="{ name: 'Chuck' }"></p> 

```

渲染后，`p`元素将包含文本`Hi Chuck, welcome back!`。

## 使用值转换器

`t`属性的一种替代方案是`t`值转换器。它可以在任何绑定表达式中使用，包括字符串插值，所以在某些情况下它比属性更方便：

```js
<p>${'explanation' | t}</p> 

```

在这里，`t`值转换器将在翻译文件中查找`explanation`翻译键并输出其值。

它的使用不仅限于字符串插值。它还适用于其他绑定表达式：

```js
<p title.bind=" 'explanation' | t "></p> 

```

在这里，`title`属性将包含`explanation`键的翻译。

### 传递参数

值转换器接受一个包含翻译参数的对象作为其第一个参数。

让我们假设以下的翻译：

```js
{ "message": "Hi {{name}}, welcome back!" } 

```

使用这个翻译与值转换器的效果是这样的：

```js
<p>${'message' | t: { name: 'Chuck' } }</p> 

```

渲染后，`p`元素将包含文本`Hi Chuck, welcome back!`。

## 使用绑定行为

然而，如果你的应用程序允许你在其生命周期内更改语言，那么值转换器根本就没有用。由于值转换器的工作方式，`t`值转换器不知道它必须重新评估其值，因为它不能在当前区域更改时得到通知。

这就是`t`绑定行为发挥作用的地方。当应用`t`绑定行为时，它简单地将`t`值转换器装饰在其绑定指示上。那么，为什么不用值转换器呢？

记得我们在第三章中看到的`signal`绑定行为吗？*显示数据*？好吧，`I18N`的`setLocale`方法实际上触发了`aurelia-translation-signal`绑定信号，而`t`绑定行为监听它。当当前区域更改时，所有活动的`t`绑定行为强制其绑定表达式重新评估，所以每个绑定表达式的底层值转换器可以使用新的区域。

### 传递参数

传递给绑定行为的任何参数对象都将传递给底层的值转换器，所以值转换器示例也适用于绑定行为：

```js
<p ></p> 

```

## 使用代码

当然，翻译一个键的所有这些不同方式都依赖于同一个`I18N`方法：

```js
tr(key: string, parameters?: object): string 

```

例如，假设`i18n`是`I18N`的一个实例，在 JS 代码中翻译同一个`message`键就像这样：

```js
let message = i18n.tr('message', { name: 'Chuck' }); 

```

## 选择一种技术胜过另一种

我们刚刚看到了四种不同的做事方式。一开始可能很难决定在哪种情况下一种技术最适合胜过其他技术。

`t`属性是来自`i18next`的一个遗留问题。当独立使用，在 Aurelia 之外时，`i18next`使用这个属性在 DOM 树内翻译文本。`aurelia-i18n`库可能支持它，只是为了让有`i18next`经验的人可以像往常一样使用它。然而，在一个 Aurelia 应用内部，它并不能在每种情况下使用；例如，它在与自定义元素一起使用时表现不佳，因为它会覆盖元素的内容。

作为一个经验法则，在模板内翻译时，我总是选择绑定行为技术。由于`t`属性和`t`值转换器有如此重要的限制，这种技术是最灵活的，我可以通过在整个应用程序中使用相同的技术来保持一致性。

如果应用程序只有一种语言，或者如果用户在应用程序启动后不能更改当前语言，那么可以使用值转换器技术。然而，我看不出真正的益处。尽管它的内存占用可能比绑定行为略小一些，但收益不会很大，而且如果上下文发生变化，应用程序突然需要支持区域设置变化，每个值转换器实例都不得不被绑定行为替换，到处都是。因此，在大多数情况下，使用值转换器可能是一种相当鲁莽的赌博。

最后，当需要在 JS 代码中翻译文本时，我会直接使用 API，在这种情况下，`I18N`实例可以很容易地被注入到需要它的类中。

这些指南适用于翻译，也适用于以下各节中描述的格式化特性。

# 格式化数字

如前所述，`aurelia-i18n`也依赖于本地 Intl API 提供数字格式化功能。

### 注意

由于库使用了 Intl API，如果你不熟悉它，我强烈建议你查阅相关资料。Mozilla 开发者网络提供了关于该主题的详尽文档：[`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Intl)

## 使用值转换器

格式化数字的最简单方法是使用`nf`值转换器：

```js
${1234 | nf} 

```

它只是使用当前区域设置创建一个`Intl.NumberFormat`实例，并调用其`format`方法，将`1234`值传递给它。

它也可以直接传递一个`Intl.NumberFormat`实例：

```js
${1234 | nf: myNumberFormat} 

```

在这种情况下，直接使用传递的`Intl.NumberFormat`实例来`format`值。

最后，它可以传递一个选项对象，可选地传递一个区域设置或区域设置数组：

```js
${1234 | nf: { currency: 'EUR' }} 
${1234 | nf: { currency: 'EUR' }: 'fr'} 

```

在这种情况下，将创建一个`Intl.NumberFormat`实例，使用选项和区域设置来`format`值。如果没有传递区域设置，将使用当前区域设置。

## 使用绑定行为

`nf`值转换器有一个与`t`值转换器相同的问题：如果当前区域设置发生变化，它没有办法得到通知。因此，如果应用程序在其生命周期内允许您更改语言，应使用`nf`绑定行为：

```js
${1234 & nf} 

```

它的运作方式与`t`绑定行为完全相同，监听`aurelia-translation-signal`绑定信号，并在信号发出时强制重新评估其绑定表达式。

它也是通过在幕后用`nf`值转换器装饰其绑定指令，并将所有参数传递给它，因此它支持与值转换器相同的参数。

## 使用代码

在幕后，值转换器依赖于`I18N`的`nf`方法：

```js
nf(options?: object, locales?: string | string[]): Intl.NumberFormat 

```

这个方法简单地使用提供的选项和区域设置创建一个`Intl.NumberFormat`实例，并返回它。如果没有传递区域设置，将使用当前区域设置：

```js
let value = i18n.nf({ currency: 'EUR' }).format(1234); 

```

在这里，我们调用`nf`方法使用提供的选项和当前区域设置创建一个`Intl.NumberFormat`实例，然后我们调用结果`Intl.NumberFormat`对象的`format`方法。

# 格式化日期

国际化的 Intl API 还包括日期格式化功能。因此，`aurelia-i18n`封装了这些功能，使其更简单地与当前区域设置一起工作。

## 使用值转换器

`df`值转换器的工作方式与`nf`值转换器几乎相同：

```js
${contact.birthday | df} 

```

它应用的值预期要么是一个`Date`对象，要么是一个`string`，该`string`将使用`Date(string)`构造函数转换为`Date`对象。

`df`值转换器在幕后的工作方式与`nf`基本相同，不同之处在于它使用`Intl.DateTimeFormat`类。这意味着它可以接受一个`Intl.DateTimeFormat`实例作为参数：

```js
${contact.birthday | df: myDateTimeFormat} 

```

在这种情况下，`format`方法将直接在提供的`Intl.DateTimeFormat`实例上调用。

它还可以接受一个选项对象，以及可选的区域设置或区域设置数组：

```js
${contact.birthday | df: { timeZone: 'UTC' }} 
${contact.birthday | df: { timeZone: 'UTC' }: 'fr'} 

```

在这种情况下，将使用选项和区域设置创建一个`Intl.DateTimeFormat`实例来`format`值。如果没有传递区域设置，将使用当前区域设置。

## 使用绑定行为

`df`值转换器与`t`和`nf`值转换器有同样的问题：它无法知道当前区域设置何时发生变化，因此无法重新评估其输出。因此，当应用程序生命周期中区域设置可以发生变化时，应使用`df`绑定行为：

```js
${contact.birthday & df} 

```

它的工作方式与`t`和`nf`绑定行为相同，它用`df`值转换器装饰其绑定表达式，并在`aurelia-translation-signal`发出时强制它重新评估其值：

此外，它将其参数传递给其底层值转换器，因此它支持与`df`值转换器相同的签名。

## 使用代码

值转换器依赖于`I18N`类的`df`方法来格式化日期：

```js
df(options?: object, locales?: string | string[]): Intl.DateTimeFormat 

```

与`nf`方法类似，它简单地使用提供的选项和区域设置创建一个`Intl.DateTimeFormat`实例，并返回它。如果没有提供区域设置，将使用当前区域设置：

```js
let value = i18n.df({ timeZone: 'UTC' }).format(new Date()); 

```

在这里，我们调用`df`方法使用提供的选项和当前区域设置创建一个`Intl.DateTimeFormat`实例，然后我们调用结果`Intl.DateTimeFormat`对象的`format`方法。

# 格式化相对时间

`aurelia-i18n`库还提供了一个服务，用于将时间相对于当前系统时间格式化。它允许你输出类似于`now`、`5 seconds ago`、`2 days ago`等人友好的时间差。

## 使用值转换器

显示人类友好时间差的最简单方法是使用`rt`值转换器：

`src/contacts/components/details.html`

```js
//Omitted snippet... 
${contact.modifiedAt | rt} 
//Omitted snippet... 

```

在这里，输出可能是类似于`5 days ago`，这取决于`contact.modifiedAt`的值和当前系统时间。

转换器应用的值预期要么是一个`Date`对象，要么是一个`string`，它将使用`Date(string)`构造函数转换为`Date`对象。

### 周期性地刷新值

之前的例子有一个小问题：`rt`的输出相对于当前时间，但从不更新。如果永远显示`5 秒钟前`，用户可能会觉得有些奇怪。

通常，`rt`值转换器将与`signal`绑定行为一起使用：

`src/contacts/components/details.html`

```js
//Omitted snippet... 
${contact.modifiedAt | rt & signal: 'rt-update'} 
//Omitted snippet... 

```

当然，这意味着我们需要在某个地方发出`rt-update`信号，可能是在视图模型中：

`src/contacts/components/details.js`

```js
import {inject} from 'aurelia-framework';  
import {Router} from 'aurelia-router'; 
import {BindingSignaler} from 'aurelia-templating-resources';   
import {ContactGateway} from '../services/gateway'; 
import {Contact} from '../models/contact'; 

@inject(ContactGateway, Router, BindingSignaler) 
export class ContactDetails { 

  constructor(contactGateway, router, signaler) { 
    this.contactGateway = contactGateway; 
    this.router = router; 
    this.signaler = signaler; 
  } 

  activate(params, config) { 
    return this.contactGateway.getById(params.id) 
      .then(contact => { 
        this.contact = Contact.fromObject(contact); 
        config.navModel.setTitle(this.contact.fullName); 
        this.rtUpdater = setInterval( 
          () => this.signaler.signal('rt-update'), 1000); 
      }); 
  } 

  //Omitted snippet... 

  deactivate() { 
    if (this.rtUpdater) { 
      clearInterval(this.rtUpdater); 
      this.rtUpdater = null; 
    } 
  } 
} 

```

在这里，我们首先在视图模型中注入一个`BindingSignaler`实例。然后，一旦联系人加载完成，我们使用`setInterval`函数每秒发出一个`rt-update`信号。每次发出信号时，视图中的`signal`绑定行为将刷新绑定并重新应用`rt`值转换器到`contact.modifiedAt`。

我们通过使用`clearInterval`函数在组件停用时停止信号的发出，从而防止内存泄漏。

这段代码仍然有一个问题：如果当前区域更改，绑定将会有延迟地刷新。这个问题很容易解决：

`src/contacts/components/details.html`

```js
//Omitted snippet... 
${contact.modifiedAt | rt  
  & signal:'rt-update':'aurelia-translation-signal'} 
//Omitted snippet... 

```

我们只需要监听`aurelia-translation-signal`信号，以及`rt-update`信号。前者是由`I18N`在当前区域每次更改时发出的信号。

现在`contact.modifiedAt`显示的时间差将每秒刷新，并且在当前区域更改时也会更新。

## 使用代码

值转换器依赖于一个独特的类，名为`RelativeTime`，该类由`aurelia-i18n`导出，并提供以下方法：

```js
getRelativeTime(time: Date): string 

```

这个方法简单地计算提供的`time`和当前系统时间之间的差异，并使用内置的翻译集合，返回当前区域的人友好的文本。

如果你需要从一些 JS 代码中转换日期为人友好的相对时间，你可以在你的类中轻松注入`RelativeTime`的一个实例并使用其`getRelativeTime`方法。

# 翻译我们的联系人管理应用程序

在此阶段，您已经拥有完全国际化我们的联系人管理应用程序所需的所有工具，除了验证消息和文档标题，它们需要与`aurelia-validation`和`aurelia-router`集成，这部分内容将在接下来的章节中详细介绍。

展示如何国际化应用程序中的每个模板会花费太长时间并且相当繁琐，所以我会留给读者作为一个练习。像往常一样，本章的示例应用程序可以作为参考。

下面的章节假设您已经国际化了您工作副本中应用程序中可以国际化的所有内容。如果您跳过手动执行此操作，我强烈建议您从书籍资源中的`chapter-8/samples/app-translated`目录获取最新的代码副本。

# 与验证集成

如果您向使用`aurelia-validation`的应用程序添加国际化，您将希望翻译错误消息。本节解释了如何将这两个库结合起来实现这一点。

## 覆盖 ValidationMessageProvider

验证库使用一个`ValidationMessageProvider`类来获取错误消息。让我们扩展这个类，并使用`I18N`从翻译文件中获取消息：

`src/validation/i18n-validation-message-provider.js`

```js
import {inject} from 'aurelia-framework'; 
import {I18N} from 'aurelia-i18n'; 
import {ValidationParser, ValidationMessageProvider}  
  from 'aurelia-validation'; 

@inject(ValidationParser, I18N) 
export class I18nValidationMessageProvider  
  extends ValidationMessageProvider { 

  options = { 
    messageKeyPrefix: 'validation.messages.', 
    propertyNameKeyPrefix: 'validation.properties.' 
  }; 

  constructor(parser, i18n) { 
    super(parser); 
    this.i18n = i18n; 
  } 

  getMessage(key) { 
    let translationKey = key.includes('.') || key.includes(':')  
      ? key  
      : `${this.options.messageKeyPrefix}${key}`; 
    let translation = this.i18n.tr(translationKey); 
    if (translation !== translationKey) { 
      return this.parser.parseMessage(translation); 
    } 
    return super.getMessage(key); 
  } 

  getDisplayName(propertyName) { 
    let translationKey =  
      `${this.options.propertyNameKeyPrefix}${propertyName}`; 
    let translation = this.i18n.tr(translationKey); 
    if (translation !== translationKey) { 
      return translation; 
    } 
    return super.getDisplayName(propertyName); 
  } 
} 

```

在这里，我们首先创建一个`ValidationParser`实例，这是`ValidationMessageProvider`基类所需的，并在构造函数中注入一个`I18N`实例。我们还定义了`options`，在执行翻译前用于构建键的前缀。

接下来，我们覆盖了`getMessage`方法，在其中我们构建了一个翻译键，然后请求`I18N`实例对其进行翻译。由于`tr`方法最终如果没有找到对应的翻译，就会返回键，所以我们只有在找到翻译时才使用翻译，否则我们退回到`getMessage`的基础实现。

构建翻译键时，如果键不包含任何点或冒号，我们会在其前面加上`options`的默认前缀，因为我们认为这个键将是验证规则的名称，这是默认行为。然而，我们的`getMessage`实现允许验证规则定义一个自定义消息键，这可以是一个自定义的翻译路径，从翻译文件中的另一个区域或命名空间获取消息文本。

`getDisplayName`方法遵循一个类似的过程：我们在键前面加上`options`的默认前缀，翻译它，然后使用翻译（如果找到了的话），或者如果没有找到，就退回到基础实现。

默认情况下，我们会认为所有的验证翻译都会存放在一个共同的`validation`对象下，该对象将在一个`messages`对象下包含所有错误消息，在`properties`对象下包含所有属性显示名称。这些路径前缀是存储在`options`对象中的默认值。

这个`options`对象如果应用程序的某个部分需要在其翻译文件的不同部分查找验证键时可能很有用；在这种情况下，应用程序的这部分可以定义自己的、定制的`I18nValidationMessageProvider`实例，使用不同的`options`值。

下一步是告诉验证系统使用这个类而不是默认的`ValidationMessageProvider`。在`validation`特性的`configure`函数中执行这个操作最合适：

`src/validation/index.js`

```js
import {ValidationMessageProvider} from 'aurelia-validation'; 
import './rules'; 
import {BootstrapFormValidationRenderer}  
  from './bootstrap-form-validation-renderer'; 
import {I18nValidationMessageProvider}  
  from './i18n-validation-message-provider'; 

export function configure(config) { 
  config.plugin('aurelia-validation'); 
  config.container.registerHandler('bootstrap-form',  
    container => container.get(BootstrapFormValidationRenderer)); 

 config.container.registerSingleton( 
    ValidationMessageProvider, I18nValidationMessageProvider); 
} 

```

在这里，我们只需告诉 DI 容器使用`I18nValidationMessageProvider`实例代替`ValidationMessageProvider`。

## 添加翻译

现在验证系统已经知道去哪里获取翻译后的错误消息和属性显示名称，接下来让我们添加正确的翻译：

`locales/en/translation.json`

```js
{ 
  //Omitted snippet... 
  "validation": { 
    "default": "${$displayName} is invalid.", 
    "required": "${$displayName} is required.", 
    "matches": "${$displayName} is not correctly formatted.", 
    "email": "${$displayName} is not a valid email.", 
    "minLength": "${$displayName} must be at least ${$config.length} character${$config.length === 1 ? '' : 's'}.", 
    "maxLength": "${$displayName} cannot be longer than ${$config.length} character${$config.length === 1 ? '' : 's'}.", 
    "minItems": "${$displayName} must contain at least ${$config.count} item${$config.count === 1 ? '' : 's'}.", 
    "maxItems": "${$displayName} cannot contain more than ${$config.count} item${$config.count === 1 ? '' : 's'}.", 
    "equals": "${$displayName} must be ${$config.expectedValue}.", 
    "date": "${$displayName} must be a valid date.", 
    "notEmpty": "${$displayName} must contain at least one item.", 
    "maxFileSize": "${$displayName} must be smaller than ${$config.maxSize} bytes.", 
    "fileExtension": "${$displayName} must have one of the following extensions: ${$config.extensions.join(', ')}." 
  },  
  "properties": { 
    "address": "Address", 
    "birthday": "Birthday", 
    "city": "City", 
    "company": "Company", 
    "country": "Country", 
    "firstName": "First name", 
    "lastName": "Last name", 
    "note": "Note", 
    "number": "Number", 
    "postalCode": "Postal code",  
    "state": "State", 
    "street": "Street", 
    "username": "Username" 
  }, 
  //Omitted snippet... 
} 

```

`messages`下的键是`aurelia-validation`在撰写本文时支持的标准规则，以及我们在`validation`特性中定义的自定义规则的消息。那些在`properties`下的键是应用程序中使用的每个属性的显示名称。至于法语翻译，您可以从本章的示例应用程序中获得。

在此阶段，如果您运行应用程序，点击**新建**按钮，例如在**生日**文本框中输入胡言乱语然后尝试保存，您应该会看到一条翻译后的错误消息出现。然而，如果您使用视图区域右上角的地区选择器更改当前语言环境，验证错误将不会随新语言环境刷新。

为了实现这一点，`ValidationController`实例需要被告知在当前语言环境发生变化时重新验证。

## 刷新验证错误

为了刷新验证错误，联系人创建视图模型必须订阅一个名为`i18n:locale:changed`的事件，当当前语言环境发生变化时，通过应用程序的事件聚合器由`I18N`实例发布。

事件聚合器是 Aurelia 默认配置的一部分，已经安装并加载，因此在我们的应用程序中使用它时，我们不需要做任何事情。我们可以直接更新我们的`creation`视图模型：

`src/contacts/components/creation.js`

```js
import {EventAggregator} from 'aurelia-event-aggregator'; 
//Omitted snippet... 
@inject(ContactGateway, NewInstance.of(ValidationController),  
  Router, EventAggregator) 
export class ContactCreation { 

  contact = new Contact(); 

  constructor(contactGateway, validationController,  
              router, events) { 
    this.contactGateway = contactGateway; 
    this.validationController = validationController; 
    this.router = router; 
    this.events = events; 
  } 

  activate() { 
    this.i18nChangedSubscription = this.events.subscribe( 
      'i18n:locale:changed',  
      () => { this.validationController.validate(); }); 
  }
 deactivate() { 
    if (this.i18nChangedSubscription) { 
      this.i18nChangedSubscription.dispose(); 
      this.i18nChangedSubscription = null; 
    } 
  } 
  //Omitted snippet... 
} 

```

在这里，我们只需订阅正确的事件，在当前语言环境发生变化时触发验证。当然，当组件被停用时，我们也需要处理订阅，以防止内存泄漏。

如果您再次尝试保存带有无效数据的新联系人，然后在显示验证错误时更改语言环境，您应该会看到错误消息随着新语言环境实时刷新。

# 整合路由器

您可能注意到我们完全忽略了文档标题的翻译，即在浏览器顶部栏显示的标题。由于这个标题由`aurelia-router`库控制，我们需要找到一种将路由器与`I18N`服务集成的方法。

实际上，这样做相当简单。`Router`类提供了一个专门为此类场景设计的集成点：

`src/main.js`

```js
import {Router} from 'aurelia-router';  
import {EventAggregator} from 'aurelia-event-aggregator'; 
//Omitted snippet... 
export function configure(aurelia) { 
  aurelia.use 
    .standardConfiguration() 
    .feature('validation') 
    .feature('resources') 
    .feature('contacts') 
    .plugin('aurelia-i18n', (i18n) => { 
      i18n.i18next.use(Backend); 

      return i18n.setup({ 
        backend: { 
          loadPath: './locales/{{lng}}/{{ns}}.json',  
        }, 
        lng : 'en', 
        fallbackLng : 'en', 
        debug : environment.debug 
      }).then(() => { 
        const router = aurelia.container.get(Router);  
        const events = aurelia.container.get(EventAggregator); 
        router.transformTitle = title => i18n.tr(title);  
        events.subscribe('i18n:locale:changed', () => { 
          router.updateTitle(); 
        }); 
      }); 
    }); 
  //Omitted snippet... 
}); 

```

这里，我们首先从`aurelia-router`库导入`Router`类和从`aurelia-event-aggregator`库导入`EventAggregator`类。接下来，当`I18N`的`setup`方法返回的`Promise`解决时，我们检索应用程序的根路由器实例，并将其`transformTitle`属性设置为一个函数，该函数将接收一个路由的标题并使用`I18N`的`tr`方法对其进行翻译。我们还检索事件聚合器并订阅`i18n:locale:changed`事件。当这个事件发布时，我们调用路由器的`updateTitle`方法。

当然，我们需要将所有标题替换为翻译键，并将这些添加到翻译文件中。我将留这作为读者的练习；不过，这里有一个快速列表，列出了那些标题必须更改的地方：

+   应用程序的主标题，在`src/app.js`中的`app`组件的`configureRouter`方法中设置。

+   `contacts`功能的主路由的标题，在`src/contacts/index.js`中的联系人的`configure`函数中添加到路由器。

+   在`src/contacts/main.js`中定义的第一个两个路由的标题。

本章完成的示例可以作为参考。

如果您继续测试这个，文档标题应该被正确翻译。当更改当前区域设置时，它也应该相应地更新。

# 按功能分割翻译

从书的开头，我们的一个目标就是尽可能保持应用程序中的特性解耦。本章中我们国际化的方式完全违反了这条规则。

有方法通过使用命名空间来按照功能分割翻译文件，这是`i18next`的一个特性。然而，这为我们的应用程序增加了另一层复杂性。这应该让我们重新评估我们的架构选择。我们从拥有解耦特性的好处是否值得它们不断增加的复杂性？这个问题非常值得提出。

如果您对这个问题答案仍然是肯定的，并且您对如何做到这一点感到好奇，您可以查看本书资源中的`chapter-8/samples/app-translations-by-feature`下的示例应用程序，它实现了这种分割。

# 摘要

国际化和被认为是简单话题常常被忽视，但正如本章所看到，在应用程序中它在很多层面都有影响。如果在一个项目后期添加翻译，它可能会迫使一个团队重新思考一些架构决策。

然而，一个设计良好且功能强大的国际化库可以极大地帮助这些任务。建立在著名的翻译库`i18next`和新的网络标准 Intl API 之上，`aurelia-i18n`是这样的一个库。
