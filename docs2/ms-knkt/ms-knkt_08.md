# 第八章. 插件和其他 Knockout 库

在任何软件领域的有效工作中，熟悉社区使用的工具是一个很大的部分。通常，依靠已经使用并经过实际测试的现有库和插件，比在每个新项目中重新发明轮子要好。在本章中，我们将探讨一些最受欢迎的 Knockout 插件：

+   Knockout Validation

+   Knockout Mapping

+   Knockout Kendo

+   KoGrid

+   Knockout Bootstrap

+   Knockout Switch-Case

+   Knockout Projections

+   Knockout-ES5

# Knockout Validation

用户输入的验证是一个常见的任务，几乎每个网络应用都将至少有一些需求。到目前为止，GitHub 上最受欢迎的 Knockout 插件，比下一个与 Knockout 相关的项目多 50% 的星标，Knockout Validation 创建了几个扩展器和绑定处理程序，旨在简化 HTML 表单验证。

插件的用法从将验证逻辑应用于可观察对象而不替换它们开始：

```js
var requiredValue = ko.observable().extend({ required: true });
var multipleValidationValue = ko.observable().extend({
                     required: true,
                     minLength: 3,
                     pattern: {
                          message: 'Hey this doesnt match my pattern',
                          params: '^[A-Z0-9].$'
                     }
                 });
```

对验证扩展值进行绑定使用正常的 `value` 绑定：

```js
<input data-bind="value: requiredValue" />
```

Knockout Validation 修改了标准的值和已检查绑定，以便它们显示无效值警告。默认的显示行为将在值绑定输入元素之后放置一个包含任何错误的 `span` 元素。当值有效时，错误消息 `span` 将被隐藏，当无效时将包含错误消息文本。

如果您想手动将验证消息放置在视图中，可以禁用此自动错误插入。为此，请使用 `validationMessage` 绑定，它具有与插入的 `span` 相同的行为。

## 默认验证规则

Knockout Validation 默认提供几个验证扩展器，它称之为**规则**。像正常的扩展器一样，可以在单个调用中传递多个验证规则以进行扩展，或者它们可以串联：

```js
var myObj = ko.observable().extend({ required: true });
var myObj = ko.observable().extend({ number: true, min: 10, max: 30 });
var myObj = ko.observable().extend({ number: true})
    .extend({ min: 10, max: 30 });
```

默认规则涵盖了检查值的大部分标准情况，包括但不限于数值的最小和最大值、字符串长度的最小和最大值、正则表达式模式、日期和值相等性。

## 配置验证选项

Knockout Validation 的行为非常可配置。一些更有用的选项包括：

+   `insertMessages` (默认: `true`): 如果为真，将在绑定到已验证可观察对象的输入之后插入一个 `span`。

+   `errorElementClass` (默认: `validationElement`): 这是一个在已验证的可观察对象无效时应用于元素的类。

+   `messagesOnModified` (默认: `true`): 如果为真，验证消息将不会显示，直到已验证的值被修改，这样它们就会在用户与表单交互之前隐藏。

+   `messageTemplate` (默认: `null`): 这是一个脚本元素的 ID，将用作验证消息模板，而不是将消息插入到 `span` 中。

可以通过传递一个对象到 `ko.validation.init` 来全局设置配置选项：

```js
ko.validation.init({
   insertMessages: false,
   errorElementClass: 'text-danger'
});
```

可以使用 `validationOption` 绑定（参见下一节）或通过传递配置对象到 `ko.applyBindingsWithValidation` 来上下文设置选项：

```js
ko.applyBindingsWithValidation(viewModel, rootNode, {
   insertMessages: false,
   errorElementClass: 'text-danger'
});
```

## 验证绑定处理程序

Knockout 验证添加了一些绑定处理程序来帮助显示验证错误。

`validationMessage` 绑定在验证的观察者无效时显示错误信息。当值有效时，元素将被隐藏：

```js
<div>
   <input type="text" data-bind="value: someValue"/>
   <p data-bind="validationMessage: someValue"></p>
</div>
```

`validationElement` 绑定对于应用属性和类到元素很有用。它将标题属性设置为验证消息，这对于显示工具提示很有用，并且当 `decorateElement` 配置选项为 true 时，它将 `errorElementClass`（默认为 `validationElement`）设置为元素的类属性：

```js
<div>
   <label data-bind="validationElement: someValue">
     <input type="text" data-bind="value: someValue"/>
   </label>
</div>
```

`validationOptions` 绑定与控制流绑定类似，因为它将指定的配置选项应用于所有子 DOM 节点。它可以采用与配置选项相同的对象格式：

```js
<div data-bind="validationOptions: { insertMessages: false } ">
   <input type="text" data-bind="value: someValue"/>
   <p data-bind="validationMessage: someValue"></p>
</div>
```

## 创建自定义规则

自定义规则可以全局创建，以便多个扩展器可以重用，或者内联创建，以便在单个扩展器中使用。添加全局验证规则是通过将规则对象添加到 `ko.validation.rules` 对象来完成的。规则有两个组成部分，即 **验证函数** 和 **默认消息**：

```js
ko.validation.rules['contains'] = {
    validator: function (val, substring) {
        return val.indexof(substring) !== -1;
    },
    message: 'The field must contain {0}'
};
```

验证函数接收两个参数：观察者的值和传递给验证扩展器的值。验证扩展器可以接受任何有效的 JavaScript 值，包括对象和函数。

一旦添加了验证规则，其扩展器将通过以下调用创建：

```js
ko.validation.registerExtenders();
```

然后，它可以用来扩展观察者：

```js
var title = ko.observable().extend({ contains: 'Sr.' });
```

内联验证规则通过传递相同的验证规则对象到验证扩展器来工作：

```js
var title = ko.observable().extend({
   validation: {
       validator: function (val, substring) {
           return val.indexof(substring) !== -1;
       },
       message: 'The field must contain {0}',
       params: 'Sr.'
   }
});
```

当使用内联验证规则时，`validator` 函数的第二个参数由验证规则的 `params` 属性定义。

### 注意

Knockout 验证是一个具有许多功能和选项的大型库，这些功能和选项在本节中未讨论。Knockout 验证库的完整文档可以在其 GitHub 仓库中找到：[`github.com/Knockout-Contrib/Knockout-Validation`](https://github.com/Knockout-Contrib/Knockout-Validation)。

# Knockout 映射

Knockout 映射插件是针对那些想要绑定到其服务器 AJAX 响应，而不需要手动编写 JavaScript 类以将它们转换为观察者的项目的解决方案。映射插件将 JavaScript 对象或 JSON 字符串转换为具有可观察属性的观察者对象：

```js
var mappedViewmodel = ko.mapping.fromJS({
   name: 'Timothy Moran',
   age: 24
});
ko.applyBindings(mappedViewmodel);
```

对于 JSON，请查看以下代码：

```js
var serverResponse = "{"name":"Timothy Moran","age":24}";
var mappedViewmodel = ko.mapping.fromJSON(serverResponse);
ko.applyBindings(mappedViewmodel);
```

映射插件通过将它们转换为 `observableArrays` 来处理数组。它还创建了对象的副本，允许从服务器到客户端的完整对象图被转换为可观察对象。

可以通过将 viewmodel 作为 `fromJS` 或 `fromJSON` 的第二个参数传递来执行针对使用映射插件创建的 viewmodel 的更新：

```js
ko.mapping.fromJS(data, viewModel);
```

你可以在 `cp8-mapping` 分支中看到一个映射插件作用的简单示例。

## 更新 viewmodel

`fromJS` 和 `fromJSON` 方法也可以用来更新整个 viewmodel，以便通过传递 viewmodel 作为第三个参数来处理未来的服务器更新响应：

```js
ko.mapping.fromJS(data, {}, viewModel);
```

## 解包

通常，在将数据发送回服务器时，你会使用 `ko.toJS` 或 `ko.toJSON` 将 viewmodel 解包为一个具有正常 JavaScript 属性的对象，而不是可观察属性。因为映射插件为你的 viewmodel 添加了几个用于内部使用的属性，`ko.toJS` 将产生一个杂乱的副本。相反，你可以使用 `ko.mapping.toJS` 和 `ko.mapping.toJSON` 来获取一个不带附加映射属性的解包 viewmodel。

## 映射选项

要控制映射插件如何创建或更新对象，可以在 viewmodel 首次创建时传递选项。映射插件将使用这些选项来构建 viewmodel，然后存储这些选项，以便它们可以用于所有未来的更新：

```js
var mapping = {
   // options
};

var vm = ko.mapping.fromJS(data, mapping);
```

### 使用键进行数组更新

更新数组的默认行为是用新值替换任何不完美匹配的元素。当处理对象的数组时，通常期望元素将在原地更新其值。为了告诉映射插件如何确定要更新的值中的元素与旧值相同，可以定义一个键：

```js
var mapping = {
   people: {
      key: function(person) {
         return ko.unwrap(person.id);
      }
   }
};
var vm = ko.mapping.fromJS(data, mapping);
```

### 使用 create 进行对象构造

你可以提供一个回调来控制单个属性的创建。一个常见的用例是提供一个对象的构造函数：

```js
var mapping = {
   people: {
      key: function(person) { /* same as before */ },
      create: function(options) {
         return new Person(options.data);
      }
   }
};
var vm = ko.mapping.fromJS(data, mapping);
```

### 控制更新

与创建类似，可以提供一个更新回调。返回值将用作属性的值：

```js
var mapping = {
   price: {
      update: function (options) {
         return parseMoney(options.data);
      }
   }
};
var vm = ko.mapping.fromJS(data, mapping);
```

### 选择哪些属性被映射

映射选项可以指定一个属性名称数组，以控制映射的各个方面：

+   `ignore`: 映射将不包括这些属性在生成的 viewmodel 中。

+   `copy`: 映射将直接复制这些属性的值，而不是将它们转换为可观察属性。

+   `observe`: 如果存在，只有此数组中的属性将被转换为 viewmodel 上的可观察属性。这是之前选项的逆操作。

+   `include`: 通常在使用 `ko.mapping.toJS` 时，只有原始映射中的属性会出现在输出中。`include` 数组中的任何属性也会被复制到输出中，即使它们不在原始的 viewmodel 中。

所有这些数组都将与 `ko.mapping.defaultOptions` 对象中的默认值合并。默认情况下，所有默认值都是空的，但它们可以被修改：

```js
ko.mapping.defaultOptions().ignore = ["alwaysIgnoreThis"];
ko.mapping.defaultOptions().copy = ["alwaysCopyThis"];
```

### 挑战

当服务器响应驱动应用程序的工作时，Knockout 映射插件非常有用。当应用程序需要与模型一起工作时，在模型被服务器发送之前，映射插件将无法创建视图模型。这种情况在填写表单以创建新模型时经常发生。视图模型的属性也只讲了一半的故事；对于大多数视图模型，仍然需要编写业务逻辑。由于映射插件填充的属性不在将作为参考的类中，因此针对这些属性编写函数或计算属性可能会很具挑战性。在非常复杂的情况下，某些对象的映射逻辑可能超过了用正常 JavaScript 定义的相同逻辑。虽然这可以在具有许多服务器响应的中等到大型应用程序中节省时间，但它可能并不总是适合项目。

尽管它很受欢迎，但它已不再在 GitHub 上维护。然而，截至版本 3.2，它仍然与 Knockout 兼容。

### 备注

映射插件的文档位于官方 Knockout 网站上，链接为 [`knockoutjs.com/documentation/plugins-mapping.html`](http://knockoutjs.com/documentation/plugins-mapping.html)。

# 击倒剑道

Kendo UI ([`www.telerik.com/kendo-ui`](http://www.telerik.com/kendo-ui)) 是 Telerik 提供的一个流行的 HTML5 小部件库，它提供了大量外观专业的控件。Knockout Kendo 是一个绑定库，允许 Knockout 视图模型使用 Kendo 控件。Knockout Kendo 有超过 30 个绑定，每个绑定都有多种选项，这里无法一一介绍。虽然 Knockout Kendo 是免费的，但 Kendo UI 本身不是免费的，并且需要您购买许可证才能使用。

大多数绑定都是围绕 Kendo 小部件的简单包装，提供了一个带有一些惊喜的 API。例如，这里有 **autocomplete** 绑定，它接受一个选项数组和一个绑定选择的观察：

```js
<input data-bind="kendoAutoComplete: { data: autocompleteOptions, value: autocompleteValue }" /><br>
```

`DateTimePicker` 创建了两个独立的日期和时间选择控件，绑定到一个单一的观察 `Date` 对象：

```js
<input data-bind="kendoDateTimePicker: startDate" />
```

如果您之前使用过 Kendo，您将熟悉可用的控件，Knockout Kendo 甚至还有非免费的专业 UI 小部件的绑定。您可以在 `cp8-kendo` 分支中看到一些 Kendo 控件的示例。

### 备注

您可以在 Knockout Kendo 的 GitHub 网站上找到完整的文档，链接为 [`rniemeyer.github.io/knockout-kendo/`](http://rniemeyer.github.io/knockout-kendo/)。

# KoGrid

KoGrid 是一个插件，它创建了一个渲染表格数据的绑定。正如其 GitHub 页面所述，它是“ng-grid 的直接 Knockout 版本，ng-grid 最初受到 KoGrid 的启发，而 KoGrid 又受到 SlickGrid 的启发。”其历史可能受到了祖父悖论的影响。

在其最基本的工作模式下，KoGrid 可以绑定到一个对象数组，将它们的属性转换为列，将它们的值转换为单元格：

```js
var vm = {
      people: ko.observableArray([{name: "Moroni", age: 50},
                                      {name: "Tiancum", age: 43},
                                      {name: "Jacob", age: 27},
                                      {name: "Nephi", age: 29},
                                      {name: "Enos", age: 34}])
   }
<div class="gridStyle" data-bind="koGrid: { data: people }"></div>
```

您可以在`cp8-kogrid`分支中看到这个示例。除了需要手动通过 CSS 样式指定网格本身的尺寸外，其他所有内容都是自动的。您可以通过点击列进行行排序，切换列的可见性，使用滚动条处理溢出，显示项目计数，并且可以通过拖动重新排序列。最大的缺点是数据不是使用真正的表格元素渲染的，而是使用一堆`div`元素渲染的。

当然，这只是一个基本的工作模式。KoGrid 自带了您从完整的网格小部件中期望的大多数功能：

+   **列定义**：这指定了哪些行属性被显示为列。

+   **分组**：这允许用户选择一个列来旋转表格，通过匹配所选列的值将所有行分组。

+   **选中行**：使用此功能，可以将可观察数组绑定到表格的选中行。当`multiSelect`选项为 false 时，这可以用来创建一个主/详细视图，其中包含选中的行。

+   **模板**：这为网格提供行和单元格模板。

+   **主题**：这指定了每个网格的绑定选项，从而指定主题。

+   **服务器端分页**：这提供了回调函数，允许网格从外部源异步获取数据。

### 注意

如果您想要一个输出真实表格元素的绑定，并且不需要 KoGrid 提供的所有功能，请查看 knockout-table 插件[`github.com/mbest/knockout-table`](https://github.com/mbest/knockout-table)。

在这些功能中，模板可能是最重要的。虽然它们的示例页面将模板直接放在视图模型代码中，但这不是推荐的做法，除非您是从外部源（如 AJAX 或 RequireJS 文本加载器）加载字符串。KoGrid 还支持通过引用其 ID 使用脚本元素作为模板，例如 Knockout 的模板系统。然而，最简单的方法是使用 URL 字符串来引用 HTML 部分文件作为模板：

```js
<div data-bind="koGrid: { data: people,
      canSelectRows: false,
      displaySelectionCheckbox: false,
      columnDefs: [
         { field: 'name', displayName: 'Name', width: '*' },
         { field: 'age', displayName: 'Age', width: '*' },	
         { field: '', displayName: ' ', 
            cellTemplate: 'app/deleteButtonCell.html', 
            width: '**' 
         }]}" class="gridStyle"></div>
```

前面的示例向您展示了几个网格选项以及指定要显示哪些列的列定义。请注意，最后一列没有属性，但有一个模板，会显示一个删除按钮。

可以在视图模型中定义这些选项，并将单个对象传递给 KoGrid 绑定；然而，这会导致视图模型与其使用紧密耦合，这是违反 MVVM 模式的。在视图中定义网格选项可以保持视图模型与显示方式无关。

删除按钮模板将由 KoGrid 在单元格的绑定上下文中渲染：

```js
<div data-bind="attr: { 'class': 'kgCellText colt' + $index()}">
   <button class="btn btn-xs btn-danger" data-bind="click: function() { $parent.$userViewModel.remove($parent.entity) }">Delete</button>
</div>
```

这里不会涵盖单元格和行模板的完整文档，但前面的模板展示了几个重要组件。

为了正确控制其宽度和位置，单元格需要包含`kgCellText`类以及代表该列的 0 索引类。由于单元格将在列循环中使用，它能够访问特殊的绑定上下文属性`$index()`来获取这个值。

在 Knockout 中，点击绑定的默认值是当前绑定上下文。在单元格模板内部，这将是指单元格对象而不是数据数组中的项目。可以通过`$parent.entity`访问绑定项。要访问视图模型，网格绑定到`$parent.$userViewModel`。在这两种情况下，`$parent`是行的绑定上下文；在创建行模板时，可以使用`$data.entity`和`$userViewModel`来访问相同的属性。

你可以在`cp8-kogrid-template`分支中看到这个自定义模板的例子。

### 注意

KoGrid 的完整文档可以在其 GitHub Wiki 页面上找到，网址为[`github.com/Knockout-Contrib/KoGrid/wiki`](https://github.com/Knockout-Contrib/KoGrid/wiki)。

# Knockout Bootstrap

Twitter Bootstrap 有几个依赖于 jQuery 的美丽小部件，可以从 JavaScript 中使用，或者在某些情况下，使用它们的`data-*`属性。如果你使用 Knockout，则需要做一些工作才能使其与可观察对象一起工作，并从绑定处理程序中初始化它。Knockout Bootstrap 是一个流行的插件，它解决了这个问题。不幸的是，在撰写本文时，它还没有更新以支持 Bootstrap 3，因此，其中一些功能无法正常工作。当与 Knockout 3 和 Bootstrap 3 一起工作时，**ToolTip**、**Popover**和**Alerts**绑定可以正常工作，但**Progress Bar**和**Typeahead**绑定则无法工作。

如同 Knockout Kendo，如果你使用过 Bootstrap 小部件，Knockout Bootstrap 中的绑定应该会立即熟悉。绑定以它们的部件命名，并接受一个具有与 jQuery 插件初始化器相同属性的对象。当合理时，这些属性可以绑定到：

```js
//Tooltip
<p>This is a paragraph with a <span data-bind="tooltip: { title: tooltipText, placement: 'bottom' }"> tooltip span</span> inside.
</p>

//Popover
<button class="btn btn-primary" data-bind="popover: {template: 'popoverTemplate', title: 'Oh Yea'}">
    Launch Simple Popover
</button>

//Alerts
<div data-bind="foreach: alerts">
    <div data-bind="alert: $data"></div>
</div>
```

这些都可以在`cp8-knockout-bootstrap`分支中看到。需要注意的是，当在 UI 上关闭警报时，警报绑定不会从绑定数组中删除警报，尽管它会根据添加或删除数组元素来显示或隐藏数组元素。

### 注意

Knockout Bootstrap 的完整文档可在[`billpull.com/knockout-bootstrap`](http://billpull.com/knockout-bootstrap)找到。

# Knockout Switch-Case

尽管 Knockout Switch-Case 插件针对的是单一、特定的用例，但它在 GitHub 上的流行程度证明了 switch/case 控制流绑定是一个非常实用的工具。与其编写一系列`if`/`ifnot`绑定，不如使用单个 case-switch 绑定：

```js
<div data-bind="switch: orderStatus">
    <div data-bind="case: 'shipped'">
        Your order has been shipped. Your tracking number is <span data-bind="text: trackingNumber"></span>.
    </div>
    <div data-bind="case: 'pending'">
        Your order is being processed. Please be patient.
    </div>
    <div data-bind="case: 'incomplete'">
        Your order could not be processed. Please go back and complete the missing data.
    </div>
    <div data-bind="case: $default">
        Please call customer service to determine the status of your order.
    </div>
</div>
```

之前提到的例子可以在`cp8-case-switch`分支中看到。

Switch 绑定也可以作用于真值。这可以通过在一系列中查找第一个匹配值来完成：

```js
<div data-bind="switch: true">
    <div data-bind="case: trackingNumber">
        Your order has been shipped.
    </div>
    <div data-bind="case: isReady">
        Your order is being processed.
    </div>
    <div data-bind="casenot: isComplete">
        Your order has been processed.
    </div>
    <div data-bind="case: $else">
        Your order could not be processed.
    </div>
</div>
```

或者，它也可以作为一对 `if/ifnot` 绑定的缩写来完成：

```js
<div data-bind="switch: isReady">
    <div data-bind="case: true">You are ready!</div>
    <div data-bind="case: false">You are not ready!</div>
</div>
```

Switch-case 绑定也可以用作前面情况组合中的无容器绑定。

如您可能已经注意到的，还有特殊的 `$default` 和 `$else` 选项，可以在找不到匹配值时使用。

### 注意

Knockout Switch-Case 的源代码可在 GitHub 上找到，链接为 [`github.com/mbest/knockout-switch-case`](https://github.com/mbest/knockout-switch-case)。

# Knockout 投影

使用计算观察者来过滤或投影观察者数组是一个极其常见的操作；我认为我从未见过一个没有至少一次执行此操作的 Knockout 投影。Knockout Projections 是一个插件，它向观察者数组添加了映射和过滤功能，这创建了一个仅在其依赖元素发生变化时才重新计算其回调的计算观察者，而不是重新评估每个依赖元素。

### 注意

Steven Sanderson 通过他的博客 [`blog.stevensanderson.com/2013/12/03`](http://blog.stevensanderson.com/2013/12/03) 介绍了这个插件。

为了更好地理解此插件解决的问题，我们将查看 Sanderson 在他的博客上使用的示例，以说明正常计算观察者数组和使用 Knockout Projections 制成的数组之间的区别。

考虑以下模型：

```js
function Product(data) {
    this.name = ko.observable(data.name);
    this.isSelected = ko.observable(false);
}
function PageViewModel() {
    // Some data, perhaps loaded via an Ajax call
    this.products = ko.observableArray([ /* several Products /* ]);
    this.selectedProducts = ko.computed(function() {
        return this.products().filter(function(product) {
            return product.isSelected();
        });
    }, this);
}
```

这个 `selectedProducts` 计算是通过标准的 ES5 数组的 `filter` 函数定义的，调用 `products()` 返回底层的 JavaScript 数组。每次运行时，它都会遍历所有产品，并返回一个包含所有 `isSelected() === true` 的元素的数组。这里的问题是计算观察者总是在其依赖项的任何一项发生变化时重新运行；计算只能通过运行其回调来执行重新评估，并且每次运行时都必须重新检查每个产品。这扩展性不好；它在 O(N) 时间内运行。

当使用 Knockout Projections 时，您将使用观察者数组上的 `filter` 函数创建相同的计算：

```js
this.selectedProducts = this.products.filter(function(product) {
    return product.isSelected();
});
```

这创建了一个只读的观察者数组，它为每个产品的 `isSelected` 观察者创建单独的依赖。当产品发生变化时，回调将仅针对该产品运行，并且 `selectedProducts` 数组将根据变化进行更新。性能现在有一个固定的成本：无论数组有多大，回调都将在每个依赖项变化时只运行一次。声明代码也更短，更容易阅读！

Knockout Projections 还在观察者上创建了一个映射函数，该函数运行一个产生数组转换的回调而不是过滤。例如，您可以创建一个只接收单个名称更改时更新的产品名称的观察者数组：

```js
this.productNames = this.products.map(function(product) {
    return product.name();
});
```

由于由 filter 和 map 创建的只读数组也是可观察数组，因此可以将这些方法链接在一起：

```js
this.selectedNames = this.selectedProducts.map(function(product) {
    return product.name();
});
```

使用 Knockout Projections 的性能提升在小数组中微乎其微，但在大数组中则非常显著。如果您正在处理即使是中等规模的数据集，使用 Knockout Projections 是不言而喻的。

# Knockout-ES5

Knockout-ES5 是 Knockout 的一个插件，它使用 JavaScript 的 getter 和 setter 来隐藏在对象属性背后的可观察对象，允许您的应用程序代码使用标准语法与它们一起工作。基本上，它消除了可观察对象的括号：

查看以下代码：

```js
var latestOrder = this.orders()[this.orders().length - 1];
latestOrder.isShipped(true);
```

之前的代码变为这样：

```js
var latestOrder = this.orders[this.orders.length - 1];
latestOrder.isShipped = true;
```

如果您还记得 Durandal 的可观察插件，它们非常相似；它们甚至几乎同时出现。这两个插件之间最大的区别是，Durandal 的可观察插件执行深度对象转换，而 Knockout ES5 执行浅度转换。

将视图模型的属性转换为可观察对象，请调用 `ko.track`:

```js
function Person(init) {
   var self = this,
      data = init || {};

   self.name = data.name || '';
   self.age = data.age || '';
self.alive = data.alive !== undefined ? data.alive : true;
   self.job = data.job || '';

   ko.track(self);
}
```

要可选地指定要转换的属性，以便传递一个属性名称数组，请查看以下代码：

```js
ko.track(self, ['name', 'age']);
```

已经存在于模型上的可观察对象，例如使用 `ko.observable` 或 `ko.computed` 创建的，也会被 `ko.track` 转换为 ES5 属性。您还可以使用 `ko.defineProperty` 定义计算可观察对象：

```js
ko.defineProperty(self, 'canRemove', function() {
   return !self.alive;
});
```

第三个参数遵循与发送给 `ko.computed` 的第一个参数相同的规则；一个函数将用于创建只读计算，或者一个对象可以用来提供读写函数。

创建了可观察对象后，您可以使用 `ko.getObservable` 来访问它们：

```js
ko.getObservable(self, 'age').subscribe(function(newValue) {
   console.log(self.name + ' age was changed to ' + newValue);
});
```

这在应用扩展器或添加订阅时非常有用。扩展器也可以通过在调用 `ko.track` 之前使用 `ko.observable` 创建可观察对象来应用。

所有这些技术的示例可以在 `cp8-es5` 分支中看到。

## 浏览器支持

由于 Knockout-ES5 使用 JavaScript 的 getter 和 setter，它将不支持此功能的浏览器中无法工作。这不是可以通过脚本模拟或填充的功能。

由于 Knockout 在创建可观察对象函数时的决策导致其语法受到很多批评。根据 Stack Overflow 上问题的流行程度，这无疑是新用户最困惑的方面之一。做出这个决定的目的是为了支持像 Internet Explorer 6 这样的旧浏览器，这些浏览器不支持 JavaScript 的 getter 和 setter。现在，随着 Internet Explorer 6 终于开始失去对浏览器市场份额的控制，这个问题对网络开发者来说变得越来越不重要。不幸的是，直到 IE 9，Internet Explorer 才添加了对 ES5 getter 和 setter 的支持，这对大多数项目来说仍然是一个很高的门槛。

实际上，由于使用 Knockout ES5 对应用语法有如此大的影响，因此在项目已经开始的情况下切换到它通常是不切实际的。Knockout ES5 应仅考虑用于没有旧浏览器支持要求的新项目。

# 摘要

确定要包含在本章中的插件和库是一个难题。Knockout 在 GitHub 上的 Wiki 页面包含了一个长长的插件列表([`github.com/knockout/knockout/wiki/Plugins`](https://github.com/knockout/knockout/wiki/Plugins))——这里讨论的太多，难以一一列举。如果你在使用 Knockout，我们鼓励你查看社区提供的资源，因为这可能会为你节省大量工作。并非所有这些插件都对每个人或每个项目都有用，但希望它们能给你一些关于 Knockout 可以做什么的灵感，并激励你与社区分享你的一些工作。

在下一章中，我们将深入探讨 Knockout 的内部机制，以了解其工作原理。
