# 第三章. 木偶视图类型及其用法

在上一章中，我们学习了帮助我们为应用程序提供结构的组件；然而，这些组件都没有与 DOM 交互。这个责任属于 Backbone 开发中的视图；然而，在视图内部，DOM 的交互和操作可能会迅速变得复杂。为了拥有更干净、更有意义的对象来操作，DOM Marionette 引入了一套强大的视图。以下是对官方文档中提供的每个视图的描述，官方文档位于[`github.com/marionettejs/backbone.marionette`](https://github.com/marionettejs/backbone.marionette)：

+   `Marionette.ItemView`：这是一个渲染单个模型的视图

+   `Marionette.CollectionView`：这是一个遍历集合并渲染每个模型的单个 `ItemView` 实例的视图

+   `Marionette.CompositeView`：这是一个用于渲染叶分支/组合模型层次结构的集合视图和项目视图

+   `Marionette.Layout`：这是一个渲染布局并创建区域管理器来管理其内部的区域的视图

+   `Marionette.View`：这是其他 Marionette 视图从中扩展的基础视图类型（不打算直接使用）

在本章中，我们将了解每个视图背后的意图以及如何开始使用它们。

# Marionette.View 和 Marionette.ItemView

`Marionette.View` 扩展了 `Backbone.View`，这一点很重要，因为我们在创建视图方面已经掌握的所有知识，在处理 Marionette 的新视图集时都将非常有用。

每个视图都旨在提供特定的开箱即用功能，这样你就可以花更少的时间关注使事物工作所需的粘合代码，更多的时间关注与你的应用程序需求相关的事情。这让你可以集中所有注意力在应用程序的具体逻辑上。

我们将从描述 Marionette 的 `Marionette.View` 部分开始，因为所有其他视图都从此扩展；我们这样做的原因是，这个视图提供了一些非常有用的功能。但重要的是要注意，这个视图并不打算直接使用。作为所有其他视图从中继承的基础视图，它是一个包含我们刚刚提到的粘合代码的绝佳位置。

该功能的良好示例是 `close` 方法，它将负责从 DOM 中移除 `.el`。此方法还将负责调用解绑所有事件，从而避免称为僵尸视图的问题。这是一个如果你在常规 Backbone 视图中不小心这样做可能会遇到的问题，其中先前关闭的实例化会触发事件。这些事件现在绑定到了视图使用的 HTML 元素上。现在视图已重新渲染，在视图重新创建期间，新的事件监听器被附加到这些 HTML 元素上。

从 `Marionette.View` 的文档中，我们确切地知道 `close` 方法的作用。

+   如果提供了，它会在视图中调用一个 `onBeforeClose` 事件

+   如果提供了，它会在视图中调用一个 `onClose` 事件

+   它解绑了所有自定义视图事件

+   它解绑了所有 DOM 事件

+   它从 DOM 中移除 `this.el`

+   它解绑了所有 `listenTo` 事件

`Marionette.View` 对象的官方文档链接是 [`github.com/marionettejs/backbone.marionette/blob/master/docs/marionette.view.md`](https://github.com/marionettejs/backbone.marionette/blob/master/docs/marionette.view.md)。

重要的一点是，第三点，*解绑所有自定义视图事件*，将解绑使用 `modelEvents` 哈希创建的事件、在事件哈希上创建的事件以及通过 `this.listenTo` 创建的事件。

由于 `close` 方法已经提供并实现，你不需要执行之前列出的解绑和移除任务。虽然大多数情况下这已经足够，但有时你的某个视图可能需要你执行额外的工作才能正确关闭它；在这种情况下，将同时触发两个事件来关闭视图。

如其名所示，`onBeforeClose` 事件将在 `close` 方法之前触发。它将调用同名的函数 `onBeforeClose`，我们可以在该点添加需要执行的代码。

```js
function : onBeforeClose () {
  //  code to be run before closing the view
}
```

第二个事件将是 `onClose`，它将在 `close` 方法之后触发，这样视图的 `.el` 就不再存在，并且所有解绑任务都已执行。

```js
 function : onClose () {
  //  code to be run after closing the view
 }
```

Marionette 背后的一个核心思想是减少你在使用 Backbone 构建应用程序时需要编写的样板代码。一个很好的例子是每个 Backbone 视图中都必须实现的 `render` 方法，那里的代码在每一个视图中几乎都是相同的。使用 underscore 的 `_.template` 函数加载模板，然后将转换为 JSON 的模型传递给模板。

以下是一个在 Backbone 中渲染视图所需的重复代码示例：

```js
render : function () {
  var template = $( '#mytemplate' ).html(); 
  var templateFunction = _.template( template ); 
  var modelToJSON = this.model.toJSON();
  var result = templateFunction(modelToJSON); 
  var myElement = $( '#MyElement' ); 
  myElement.html( result );
}
```

由于 Marionette 不再需要定义 `render` 函数，就像 `close` 方法一样，前面的代码将在幕后为你调用。为了渲染一个视图，我们只需要声明它，并设置模板属性。

```js
var SampleView = Backbone.Marionette.ItemView.extend({
  template : '#sample-template'
});
```

接下来，我们只需创建一个 Backbone 模型，并将其传递给`ItemView`构造函数。

```js
var SampleModel = Backbone.Model.extend({
  defaults : {
    value1 : "A random Value",
    value2 : "Another Random Value"
  }
})
var sampleModel = new SampleModel();
var sampleView = new SampleView({model:sampleModel);
```

然后剩下的唯一事情就是调用`render`函数。

```js
sampleView.render();
```

### 注意

如果你想看到它运行，请通过以下 JSFiddle 查看之前的代码：

[`jsfiddle.net/rayweb_on/VS9hA/`](http://jsfiddle.net/rayweb_on/VS9hA/)

需要注意的是，我们只需一行代码来指定模板，Marionette 就会通过使用指定的模板渲染我们的视图来完成剩余的工作。注意，在这个例子中，我们使用了`ItemView`构造函数；我们不应该直接使用`Marionette.View`，因为它本身没有很多功能。它只是作为其他视图的基础。

因此，以下将使用`ItemView`演示`Marionette.View`提供的部分功能，因为这个视图通过扩展继承了所有这些功能。

正如我们在前面的例子中看到的，`ItemView`在用模板渲染单个模型时工作得很好，但渲染模型集合怎么办？

如果你只需要渲染，例如，书籍或分类列表，你仍然可以使用`ItemView`。为了完成这个任务，分配给`ItemView`的模板必须知道如何处理 DOM 的创建，以正确显示该列表项。

让我们渲染一本书籍列表。

Backbone 模型将有两个属性：书籍名称和书籍 ID。我们只想创建一个使用书籍名称作为显示值的链接列表；书籍的 ID 将用于创建查看特定书籍的链接。

首先，让我们为这个例子创建书籍 Backbone 模型及其集合：

```js
var BookModel = Backbone.Model.extend({
  defaults : {
    id : "1",
    name : "First",
  }
});

var BookCollection = Backbone.Collection.extend({
  model : BookModel
});
```

现在，让我们实例化集合并向其中添加三个模型：

```js
var bookModel = new BookModel();
var bookModel2 = new BookModel({id:"2",name:"second"});
var bookModel3 = new BookModel({id:"3",name:"third"});
var bookCollection = new BookCollection();
bookCollection.add(bookModel);
bookCollection.add(bookModel2);
bookCollection.add(bookModel3);
```

在我们的 HTML 中，让我们创建一个用于此视图的模板；模板应如下所示：

```js
<script id="books-template" type="text/html">
  <ul>
    <% _.each(items, function(item){ %>
    <li><a href="book/'+<%= item.id %> +"><%= item.name %> </li>
    <% }); %>
  </ul>
</script>
```

现在，我们可以使用以下代码片段来渲染书籍列表：

```js
var BookListView = Marionette.ItemView.extend({
  template: "#books-template"
});

var view = new BookListView ({
  collection: bookCollection
});
view.Render();
```

### 注意

如果你想看到它的实际效果，请访问 JSFiddle 中的工作代码，链接为[`jsfiddle.net/rayweb_on/8QAgQ/`](http://jsfiddle.net/rayweb_on/8QAgQ/)。

之前的代码将生成一个包含特定书籍链接的无序列表。再次强调，我们再次获得了编写少量代码的好处，因为我们不需要指定`Render`函数，这可能会产生误导，因为`ItemView`完全能够渲染模型或集合。是否使用`CollectionView`或`ItemView`将取决于我们想要实现的目标。如果我们需要一组具有自身功能的独立视图，`CollectionView`是正确的选择，正如我们在回顾它时将看到的。但如果我们只需要渲染集合的值，`ItemView`将是完美的选择。

## 在视图中处理事件

为了跟踪模型事件或集合事件，我们必须在常规 Backbone 视图中编写以下代码片段：

```js
this.listenTo(this.model, "change:title", this.titleChanged);
this.listenTo(this.collection, "add", this.collectionChanged);
```

要启动这些事件，我们使用以下处理函数：

```js
titleChanged: function(model, value){alert("changed");},
collectionChanged: function(model, value){alert("added");},
```

这在 Marionette 中仍然有效，但我们可以通过以下配置哈希来声明这些事件，以实现相同的功能：

```js
modelEvents: {
  "change:title": "titleChanged" 
},
collectionEvents: {
  "add": "collectionChanged" 
},
```

这将给出完全相同的结果，但配置哈希非常方便，因为我们可以在我们的模型或集合中继续添加事件，代码更简洁，更容易理解。

`modelEvents` 和 `collectionEvents` 不是我们在每个 Marionette 视图中可用的唯一配置哈希集；UI 配置哈希也是可用的。可能存在这样的情况，即您的视图中的一个 DOM 元素将被多次使用以读取其值，并且使用 jQuery 来做这件事在性能方面可能不是最优的。此外，我们将在几个地方重复 jQuery 引用，这会使我们的代码更冗余。

在一个 Backbone 视图中，我们可以定义一组事件，一旦在 DOM 中执行操作就会触发；例如，我们传递一个函数来处理按钮点击事件。

```js
events : { 
  "click #button2" : "updateValue" 
},
```

这将在我们点击 `button2` 时调用 `updateValue` 函数。这没问题，但如果我们想调用不在视图内的方法怎么办？

为了实现这一点，Marionette 提供了 `triggers` 功能，它将触发可以在视图外部监听的事件。要声明一个 `trigger`，我们可以使用与 `events` 对象相同的语法，如下所示：

```js
triggers : { "click #button1": "trigger:alert"},
```

然后，我们可以使用以下代码在别处监听该事件：

```js
sampleView.on("trigger:alert", function(args){
  alert(args.model.get("value2"));
});
```

在前面的代码中，我们使用了模型来警告并显示属性 `value2` 的值。

函数接收到的 `args` 参数将包含您可以使用的对象：

+   触发事件的视图

+   该视图的 Backbone 模型或集合

## UI 和模板

当与视图一起工作时，您可能需要在视图的多个地方通过 jQuery 引用一个特定的 HTML 元素。这意味着您将在初始化和视图的几个其他方法中引用一个按钮。为了避免在每个这些方法中重复 jQuery 选择器，您可以在哈希中映射该 UI 元素，以便保留选择器。如果您需要更改它，更改将在一个地方完成。

要创建 UI 元素的映射，我们需要添加以下声明：

```js
ui: {
  quantity: "#quantity"
  saveButton : "#Save"
},
```

要使用这些映射 UI 元素，我们只需在配置中给出的名称下，在任意函数中引用它们。

```js
validateQuantity: function() {
  if (this.ui.quantity.val() > 0 {
    this.ui.saveButton.addClass('active');
  }
}
```

有时候您需要向视图传递不同的模板。在 Marionette 中，我们移除模板声明，而是添加一个名为 `getTemplate` 的函数。

以下代码片段将说明该函数的使用：

```js
getTemplate: function(){
  if (this.model.get("foo"){
    return "#sample-template";
  }else {     
    return "#a-different-template";
  }
},
```

在这种情况下，我们检查属性 `foo` 的存在性；如果不存在，我们使用不同的模板，这就足够了。您不需要指定 `render` 函数，因为它将以与在先前的示例中声明模板变量相同的方式工作。

如果你想要了解更多关于我们迄今为止讨论的所有概念，请参考 JSFiddle 链接[`jsfiddle.net/rayweb_on/NaHQS/`](http://jsfiddle.net/rayweb_on/NaHQS/)。

如果你发现自己需要在渲染值时进行涉及复杂过程的计算，你可以使用包含在名为`templateHelpers`的对象中的`templeteHelpers`函数。让我们看看一个更好的例子来展示它的使用。

假设我们需要显示书籍的值，但提供了一个需要计算的折扣，可以使用以下代码：

```js
var PriceView = Backbone.Marionette.ItemView.extend({
  template: "#price-template",

  templateHelpers: {
    calculatePrice: function(){
    // logic to calculate the price goes here
      return price; 
    }
  }
});
```

如前述代码所示，我们声明了一个包含可以从中调用的函数的对象`literal`。

```js
<script id="my-template" type="text/html">
  Take this book with you for just : <%= calculatePrice () %>
</script>
```

## Marionette.CollectionView

在一个视图中渲染类似书籍的列表是可能的，但我们希望能够与每个项目进行交互。这个解决方案将是使用循环逐个创建视图。但是 Marionette 通过引入`CollectionView`的概念以非常优雅的方式解决了这个问题，该概念将为我们要显示的集合中的每个元素渲染一个子视图。

一个很好的实践例子可能是按类别列出书籍并创建一个集合视图。这非常简单。

首先，你需要定义每个项目应该如何显示；这意味着每个项目将如何在视图中转换。

对于我们的类别示例，我们希望每个项目都是一个列表`<li>`元素，并且是集合的一部分；`<ul>`列表将包含每个类别视图。

我们首先声明`ItemView`如下：

```js
var CategoryView = Backbone.Marionette.ItemView.extend({
        tagName : 'li',
        template: "#categoryTemplate",
});
```

然后我们声明`CollectionView`，它指定了要使用的视图项。

```js
var CategoriesView = Backbone.Marionette.CollectionView.extend({
        tagName : 'ul',
        className : 'unstyled',
        itemView: CategoryView
}); 
```

一个值得注意的好事是，即使我们使用 Marionette 视图，我们仍然能够使用 Backbone 视图提供的标准属性，例如`tagName`和`ClassName`。

最后，我们创建一个集合，并通过传递集合作为参数来实例化`CollectionView`。

```js
var categoriesView = new CategoriesView({collection:categories);
categoriesView.render();
```

就这样。简单吧？

使用这个视图的优势在于它将为每个项目渲染一个视图，并且它可以有很多功能；我们可以在作为容器的`CollectionView`中控制所有这些视图。

你可以在[`jsfiddle.net/rayweb_on/7usdJ/`](http://jsfiddle.net/rayweb_on/7usdJ/)中看到它的实际应用。

## Marionette.CompositeView

`Marionette.Composite`视图不仅提供了渲染模型或集合模型的可能性，还提供了同时渲染模型和集合的可能性。这就是为什么这个视图非常适合我们的 BookStore 网站。我们将向购物车添加单个项目，在这种情况下是书籍，并将这些书籍存储在集合中。但是，我们需要计算订单的小计，显示计算出的税费和订单总额；所有这些属性都将是我们将要显示的`totals`模型的一部分，我们将与订购的书籍一起显示。

但存在一个问题。当没有添加项目时，我们应该在顺序区域显示什么？好吧，在 `CompositeView` 和 `CollectionView` 中，我们可以设置一个 `emptyView` 属性，这个属性将是一个在集合中没有模型时要显示的视图。一旦我们添加了一个模型，我们就可以渲染项目以及 `totals` 模型。

可能在这个时候，你可能认为你失去了对渲染功能的控制，并且会有一些需要你进行修改 HTML 的场景。好吧，在这种情况下，你应该使用 `onRender()` 函数，这是一个非常有用的方法，它将允许你在 `render` 方法被调用后立即操作 DOM。

最后，我们希望设置一个带有一些标题的模板。这些标题不是 `ItemView` 的一部分，那么我们如何显示它？

让我们看看代码片段的一部分，解释每个部分是如何解决我们的需求的。

```js
var OrderListView = Backbone.Marionette.CompositeView.extend({
        tagName: "table",
        template: "#orderGrid",
        itemView: CartApp.OrderItemView,
        emptyView: CartApp.EmptyOrderView,
        className: "table table-hover table-condensed",

        appendHtml: function (collectionView, itemView) {
            collectionView.$("tbody").append(itemView.el);
        },
```

到目前为止，我们已经定义了视图并设置了模板；`Itemview` 和 `EmptyView` 属性将被用来渲染我们的视图。

`onBeforeRender` 是一个函数，正如其名所示，它将在 `render` 方法之前被调用；这个函数将允许我们计算将在 `total` 模型中显示的总数。

```js
        onBeforeRender: function () {
            var subtotal = this.collection.getTotal();
            var tax = subtotal * .08;
            var total = subtotal + tax;
            this.model.set({ subtotal: subtotal });
            this.model.set({ tax: tax });
            this.model.set({ total: total });
        },
```

`onRender` 方法在这里用来检查集合中是否有模型（即用户没有将书籍添加到购物车）。如果没有，我们不应该显示视图的标题和页脚区域。

```js
        onRender: function () {
            if (this.collection.length > 0) {
                this.$('thead').removeClass('hide');
                this.$('tfoot').removeClass('hide');
            }
        },
```

如我们所见，Marionette 提供了非常出色的函数，可以移除大量的样板代码，并让我们完全控制渲染的内容。

# 使用 Marionette.Layout 构建我们应用程序的布局

我们需要最后审查的最终视图是 `Marionette.Layout` 视图。这个视图是 `Itemview` 和 `Region` 的组合；我们还没有审查 `Marionette.Region` 组件，但到目前为止，只需要说它是一个负责在其 `el` 上渲染视图的组件。

因此，布局作为一个 `ItemView` 工作，因为它需要一个模板来自我渲染。这个模板可以是你的初始 HTML，通过逻辑区域划分，例如包含显示网站导航部分的导航区域，应该显示在页脚区域的页脚视图，等等。你可以先渲染布局，然后在每个区域上正确渲染视图。

让我们创建 `Marionette.Layout` 视图。

```js
varCatalogLayout = Backbone.Marionette.Layout.extend({
        template: "#CatalogLayout",
        regions: {
            categories : '#categories',
            products : '#products',
            order : '#order',
            book: '#book'
        }
    });
```

在你复制的 第二章 的 HTML 中，你会找到对应区域的 `<div>` 标签。

在这个视图中，我们指定了视图将使用的脚本/模板来渲染。这个指定的模板被添加到初始 HTML 中，并且其中包含了 `<div>` 标签，这些标签将作为区域。每个区域都有一个与将要显示的视图相匹配的名称，我们使用一个对象 `literal` 来定义区域。

`Layout` 视图继承了所有其他视图的功能，因此如果您想监听事件，可以像在其他任何视图中一样进行操作。

要渲染这个初始布局，我们只需实例化它并将其渲染成任何其他视图。

```js
var catalogLayout = new CatalogLayout();
catalogLaout.render();
```

您仍然可以通过调用 `addRegion` 和 `removeRegion` 方法在运行时向您的布局添加和删除区域。

```js
layout.addRegion("footer", "#footer");
layout.removeRegion("footer ");
```

要添加多个区域，`Layout` 视图提供了一个 `addRegions` 方法，该方法接收一个包含要添加的区域的对象 `literal`。

```js
layout.addRegions({
  favoriteBooks: "#favoritebooks",
  bestRated: "#best"
});
```

这个视图上的 `Close` 函数的行为将略有不同，因为它将调用所有区域的 `close` 方法。然后，这些区域将调用它们包含的视图上的 `close` 方法，确保所有包含的视图都正确关闭。

一个开始您应用程序的好方法是定义一个 `Body` 区域；这个区域将包含所有逻辑区域的应用程序 `Layout`。也许您需要在这些区域中的一个显示子布局，这是完全可以的。嵌套布局没有限制；根据您应用程序的需求使用它们。

# 扩展 Marionette 视图

在使用 Backbone 和 Marionette 以及几乎任何语言时，一个常见的需求是尽可能多地重用代码。如果您想让所有视图以某种方式表现，您可以通过扩展您的 Marionette 视图来实现。在下面的示例中，我们将通过扩展 `Marionette.ItemView` 来为所有项目视图添加一个 `log` 方法。

```js
var HandyView = Backbone.Marionette.ItemView.extend({
  initialize:function(){
  Backbone.Marionette.ItemView.prototype.initialize.apply(this,arguments);
  },
  logMessage : function (message){
    console.log(message);
  }
});
```

现在，您只需开始使用您的 `HandyView`，以便获得 `logMessage` 函数的好处。

```js
  var BookView = HandyView.extend({
  alertMessage : function () {
    alert(message);
  }
  });

  var bookView = new BookView();
  bookView.logMessage("Hi");
  bookView.alertMessage("Bye");
```

这里的想法是让您知道，您可以像扩展 Backbone 视图一样扩展 Marionette 视图，并利用继承的好处。

# 摘要

在本章中，我们学习了 Marionette 提供的所有类型的视图，何时使用它们，如何有效地利用其便捷的方法来更好地管理 DOM 创建和交互，以及最后如何扩展它们。

在下一章中，我们将学习如何借助 Marionette 的 `Regions`、`RegionManager` 和 `BabySitter` 对象来管理一组视图。
