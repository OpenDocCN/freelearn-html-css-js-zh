# 第四章。管理视图

如我们在第三章中学习的那样，*Marionette 视图类型及其使用*，`Marionette.js` 视图为我们提供了许多功能，以极少的代码编写量渲染数据。在本章中，我们将探讨 Marionette 中的 `Region` 是什么，以及 `RegionManager` 和 `BabySitter` 对象是什么。所有这些都是为了帮助我们以更简单的方式管理视图。

我们还将了解在应用程序中渲染模板时的一个实用对象：`Renderer` 对象。之后，我们将简要总结前四章所学的内容。

本章我们将涵盖以下主题：

+   `Marionette.Region`

+   `Marionette.RegionManager`

+   `Marionette.BabySitter`

+   `Marionette.Renderer`

+   使用 `Marionette.TemplateCache` 提高应用程序的性能

所有这些都非常有用的对象，将帮助我们轻松管理 Marionette 视图，同时考虑到性能和重用。

# 理解 Marionette.Region 对象

在构建应用程序时，我们需要将屏幕划分为小的、逻辑的部分，如页眉、页脚、导航和内容区域。这些是大多数应用程序中常见的部分。通常，您的导航选项可能会根据用户而变化。页眉也可能根据您的用户配置文件而不同，当然，您的内容区域将忙于显示不同的视图——需要渲染以执行操作并关闭以显示新视图的视图，可能还有一些结果或应用程序中的下一个逻辑步骤。这就是为什么我们应该将应用程序的页脚或内容部分视为应用程序内的区域，在那里我们将交换不同的视图。

以下代码演示了创建 `Marionette.Region` 对象的一种方法：

```js
var FooterRegion = new Backbone.Marionette.Region({
  el: "#footer"
});
```

要定义一个区域，我们只需指定一个 DOM 元素，该元素将作为应用程序逻辑部分中视图的容器。在这种情况下，`#footer` 是一个 DIV 元素，但可以是任何 HTML 元素，只要在其中添加视图可以生成有效的 HTML。

区域背后的想法是将其用作应用程序中视图的容器，一次一个。当我们调用区域的 `.show` 方法时，它将负责调用指定视图的 `render` 函数。当我们调用区域的 `.close` 方法时，它将调用当前视图的关闭方法并将其从 DOM 中移除。

以下是需要使用区域来渲染视图的代码：

```js
// definition of a view to be shown in the region
var footerView = new FooterView();

//the Region will show the footerview in its DOM element
//in this case it will render the footer view inside the #footer element in the DOM
FooterRegion.show(footerView);
// the footerView is now rendered

//Finally we can call close on the region and the footer view will be removed from the #footer element
FooterRegion.close();
```

如前述代码片段所示，区域的 `show` 方法将接受要渲染的视图实例作为参数。

在一开始，渲染视图然后交换为新视图似乎很简单。我们只需调用`close`和`render`，对吧？那么为什么我们需要一个区域对象来为我们做这件事呢？这是因为区域为我们做这些事情，而且更多，我们不需要担心当前显示在其中的视图是哪一个。可以这样想：你有内容区域，你将在其中显示一个视图。我们只需通过调用`.show`方法来替换这个视图，区域将负责移除第一个视图。所以我们不需要调用它的`close`方法，因为这是区域`.show`方法功能的一部分。这意味着如果区域中已经显示了一个视图，通过调用`show`并传递一个新的视图来调用，将调用现有视图的`close`方法，从而确保其`.el`事件绑定被正确移除。

让我们以一个巫师为例。我们分为四个步骤，在每一步中，我们展示一个视图，我们将在这里填写一些数据。在这些视图中的每一个，我们都有链接，这些链接将引导我们进入下一步或回到前一步，以防我们想要修改数据。这些链接将修改 URL，这将由路由器负责调用适当的步骤。

对于这个例子，控制器内部的代码如下所示：

```js
// each view step syncs with the server at the time to initialize and close in order to preserve the data
stepOne : function () {
  var stepOneView = new StepOneView();
  content.region.show(stepOneView);
},
stepTwo : function () {
  var stepTwoView = new StepTwoView();
  content.region.show(stepTwoView);
}
stepThree : function () {
  ar stepThreeView = new StepThreeView();
  content.region.show(stepThreeView);
}
finalStep : function () {
  var finalStepView = new FinalStepView();
  content.region.show(finalStepView);
}
```

从前面的代码中，我们可以看到使用区域渲染视图的好处。如果用户点击第二个步骤并决定返回，我们不需要检查`stepTwoView`方法的实例是否在内存中，并关闭它以渲染`stepOneView`方法，因为这将由区域处理。

将会有一些情况，我们无法跟踪区域中存在哪个视图，在大多数这些情况下，我们并不关心。我们只需要在这个区域上渲染一个新的视图，而无需担心之前的视图是否被正确移除。

另一种声明区域的方法是将它直接附加到 Marionette 应用程序上，如下所示：

```js
BookStoreApp = new Backbone.Marionette.Application();
BookStoreApp.addRegions({
  contentRegion: "#mainContent",
});
```

在这种情况下，我们使用了应用程序对象的`.addRegions`方法，它期望一个包含区域名称和要使用的 DOM 元素的字面量对象。

要使用这些新的区域，我们只需通过在对象字面量中给出的名称来调用它们，如下所示：

```js
BookStoreApp.contentRegion.show(stepOneView);
```

在第三章中，我们定义了一个布局。"Marionette 视图类型及其使用"，Marionette 的布局视图充当区域或容器。布局将渲染一个包含 HTML 骨架的模板。在这个骨架内部，我们将放置作为区域的 DIV 元素或元素，一旦这个布局被渲染，我们就可以使用它的区域来显示视图。

因此，让我们按照以下方式回顾代码：

```js
CatalogLayout = Backbone.Marionette.Layout.extend({
  template: "#CatalogLayout",
  regions: {
    categoriesRegion : '#categories',
    productsRegion : '#products',
    orderRegion : '#order',
    bookRegion: '#book'
  }
});
```

在布局声明中，我们定义了一个模板和区域对象字面量，为区域命名，并将它们与 DOM 元素匹配。

对于这个`BookStoreApp`，我们将创建`mainRegion`。这个区域的责任是渲染应用程序的布局视图。布局视图将包含初始 HTML 文件和应用程序的逻辑区域，这些区域将显示适当的视图。以下代码示例说明了创建应用程序对象、布局视图以及在布局区域中渲染视图：

```js
BookStoreApp = new Backbone.Marionette.Application();
BookStoreApp.addRegions({
  mainRegion: "#mainContent",
});
CatalogLayout = Backbone.Marionette.Layout.extend({
  template: "#CatalogLayout",
  regions: {
    categoriesRegion : '#categories',
    productsRegion : '#products',
    orderRegion : '#order',
    bookRegion: '#book'
  }
});
var catalogLayput  = new CatalogLayout();
BookStoreApp.mainRegion.show(catalogLayput );

catalogLayput.categoriesRegion.show(new CategoriesView());
catalogLayput.productsRegion.show(new ProductsView());
```

在布局和区域的帮助下，我们可以创建屏幕的逻辑分割，这将允许我们在每个区域上渲染视图。为这些区域使用有意义的名称将肯定有所帮助，因为我们只需要将正确的视图传递给区域，并停止担心渲染和清理所需的粘合代码。

在渲染视图时，区域将引发以下两个事件，这将帮助我们执行对 DOM 的额外操作：

+   `"show"/onShow`: 当视图渲染并显示时，此事件在视图实例上被调用

+   `"show"/onShow`: 当视图渲染并显示时，此事件在区域实例上被调用

最后，在关闭视图时，以下方法将被触发，可以用来执行一些最终任务，例如用友好的消息通知用户：

+   `"close"/onClose`: 当视图被关闭时，此方法被调用

您可以使用`.on`方法声明像往常一样订阅这些事件，如下所示：

```js
BookStoreApp.mainRegion.on("show", function(view){
  // extra functionality needed to be added once the view is rendered
});
```

以下代码示例说明了如何订阅`close`方法：

```js
BookStoreApp.mainRegion.on("close", function(view){
  // code to notify that the view as been removed
});
```

# 使用 Marionette.RegionManager 对象

使用区域可以帮助以非常优雅的方式管理视图。但对于某些应用程序来说，这可能还不够，这些应用程序在其生命周期中可能需要添加和删除数十个区域。为了完成这项管理，我们可以利用 Marionette 的`RegionManager`对象，它将作为区域的容器。

将区域放在这个容器中可以帮助我们完成几乎与使用`Backbone.Collection`对象相同的行为，借助 underscore 方法如`each`、`map`、`invoke`、`contains`和`toArray`。

以下语法可以帮助我们声明`Marionette.RegionManager`：

```js
var regionManager = new Marionette.RegionManager();
```

以下语法演示了如何将区域添加到`regionManager`。`addRegion`方法接受两个参数。第一个将是区域的 ID 或别名，第二个将是用于的 DOM 元素。

```js
regionManager.addRegion("math", "#math");
```

如果我们想添加多个区域，我们需要使用`addRegions`方法，并将带有名称和 ID 的对象字面量作为参数传递给此方法。

```js
regionManager.addRegions({
  art: "#art",
  music: "#music",
  science: "#science"
});
```

我们可以使用`removeRegion`方法从`regionManager`中删除区域，并关闭区域，这将调用包含视图的`close`，从而从应用程序中删除此视图。

```js
regionManager.removeRegion("math");
```

要移除所有区域，我们可以使用以下代码中所示的 `removeRegions` 方法：

```js
regionManager.removeRegions();
```

我们可以继续向这个容器添加更多区域。但现在让我们创建一个 `RegionManager` 对象，用于一个包含子类别区域的类别；每个区域将显示一个具有相同子类别书籍的 `CollectionView` 对象。因此，我们在具有相同名称的区域中显示一个 `CollectionView` 对象。

这个想法是用户希望查看他最喜欢的类别的书籍，并且还希望向屏幕上添加更多要渲染的子类别。例如，在历史类别中，我们可以有一个选项来查看现代历史、罗马、世界历史、第二次世界大战等更多子类别的书籍。

```js
var historyManager = new Marionette.RegionManager();
historyManager.addRegions({
  wwIIRegion: "#wwII",
  romeRegion: "#rome",
  modernRegion: "#modern",
  unitedStatesHistoryRegion : "#us"
});
```

然后在每个区域上渲染适当的视图，如下所示：

```js
historyManager.modernRegion.show(new BooksView({collection : modern}));
historyManager.wwIIRegion.show(new BooksView({collection :wwIIBooks}));
```

如果在某个时刻，用户切换到另一个类别并且不再想看到历史类别，调用 `removeRegions` 将级联到视图级别，从而以安全的方式移除所有视图。

# 使用 Backbone.BabySitter 对象

他们提供的值使得两个对象从 Marionette 库中脱颖而出，并且可以通过将它们包含在您的脚本中或使用 Marionette 的混合版本来单独使用。一个是 `Backbone.Wreqr.EventAggregator` 对象，另一个是 `Backbone.BabySitter` 对象，这是我们即将讨论的对象。

这个对象帮助我们跟踪视图并管理它们。这些视图可以包含在另一个视图或另一个需要跟踪这些视图的对象中。

`BabySitter` 对象可以用来包含视图而不是区域。`RegionManager` 的责任是在您的应用程序中包含和执行区域内的操作。`BabySitter` 对象有相同的责任，但在这个情况下，它帮助我们管理相关的视图。

以下代码是一个 BabySitter 对象实例化的示例：

```js
container = new Backbone.ChildViewContainer();
```

您可以开始向这个容器添加和移除视图，因为它是一个常规的 `Backbone.Collection` 对象；您可以通过使用添加和移除函数来完成此操作。

```js
container.add(someView);
container.add(anotherView);
container.remove(someView);
```

容器提供了许多有用的方法，例如长度，它将返回容器跟踪的视图数量。

```js
var numberofViews  =container.length
```

正如我们之前提到的，容器公开了几个 underscore 集合函数的功能，例如 `each`、`map`、`find`、`select`、`filter`、`all`、`some` 和 `toArray`。

为了演示我们可以在容器内部调用每个视图的方法，我们只需要编写以下代码：

```js
container.each(function(view){
  view.changeColor();
});
```

前面函数的责任是迭代容器中的所有视图，并在该视图中调用一个函数，在这种情况下，是每个视图的 `changeColor` 函数。

我们创建了一个 JSFiddle 来演示 `BabySitter` 对象的使用。您可以在 [`jsfiddle.net/rayweb_on/fxzAs/`](http://jsfiddle.net/rayweb_on/fxzAs/) 找到它。

### 注意

[jsfiddle.net](http://jsfiddle.net)是一个非常有用的网站，用于共享代码片段，促进开发者之间的协作。

# 利用 Marionette.Renderer 对象

正如我们在第三章中看到的，“Marionette 视图类型及其用途”，Marionette 出色地完成了删除渲染视图所需重复代码的工作。但在幕后，Marionette 所做的是将此任务委托给`Renderer`对象。其责任是加载和编译模板到`.template`函数，并传递用于正确渲染视图模型的数据。这意味着 Marionette 中的每个视图对象都使用`Renderer`对象。此对象还可以通过仅传递一个带有用于此目的的数据的模板来帮助将 HTML 渲染到 DOM 中。

假设由于某种原因我们需要将 HTML 附加到我们的视图中，但我们不想调用我们正在处理的视图的渲染。在这种情况下，我们可以使用`Marionette.Renderer`对象。

```js
callRenderer : function () {
  var template = "#sample-template2";
  var data = {foo: "I was appended with Marionette Renderer"};
  var html = Backbone.Marionette.Renderer.render(template, data);
  this.$el.append(html);
}
```

在前面的函数中，我们声明了一个在 DOM 中的模板，然后创建了一个包含虚拟数据的对象字面量，然后将这两个传递给`Renderer`对象，它返回一个 HTML，我们最终将其附加到视图的`$el`上。

要查看此示例的运行效果，您可以访问[`jsfiddle.net/rayweb_on/BeMfz/`](http://jsfiddle.net/rayweb_on/BeMfz/) JSFiddle 链接。

但也许更好的用途是渲染将作为区域容器的 DOM 元素。

如果我们需要渲染子类别，我们需要确保 DIVs（`<div id="rome"><div>`）可用，以便附加区域，然后调用`show`来渲染这些容器（DIV 元素）内的视图。

# 使用 TemplateCache 提高应用程序的性能

在每个应用程序中，性能都很重要。这就是为什么我们的模板在缓存中可用将肯定提高未来调用中渲染过程的速度。

Marionette 有一个名为`TemplateCache`的对象，该对象由`Renderer`对象使用。这意味着所有模板都存储在这个`TemplateCache`对象中，要开始使用它，我们只需调用`get`方法。内部，此方法将确认是否已有模板，然后返回它；否则，它将从 DOM 中加载模板，并返回模板以便我们可以使用，同时也会保留其引用。因此，在后续调用中，我们将通过以下代码获取模板的缓存版本：

```js
var template = Backbone.Marionette.TemplateCache.get("#my-template");
```

要从我们的缓存中删除一个或多个模板，我们需要调用`clear`函数，如下所示：

```js
Backbone.Marionette.TemplateCache.clear("#my-template")
```

如果我们需要删除多个模板，我们可以传递要删除的模板列表，或者简单地调用不带参数的`clear()`函数来删除所有缓存的模板，如下所示：

```js
Backbone.Marionette.TemplateCache.clear("#my-template", "#this0-template")
```

或者，我们可以使用以下代码来完成此操作：

```js
Backbone.Marionette.TemplateCache.clear()
```

默认情况下，Marionette 的`TemplateCache`与 underscore 模板一起工作，但为了覆盖这一点，我们需要提供我们自己的`compileTemplate`函数定义。

为了保持一致性，我们将覆盖这个函数以返回 handlebars 模板。

```js
Backbone.Marionette.TemplateCache.prototype.compileTemplate = function(rawTemplate) {
  return Handlebars.compile(rawTemplate);
}
```

### 注意

处理 Handlebars 是一个非常流行的模板引擎，通常用作 underscore 模板的替代品。你可以在其网站上了解更多信息，[`handlebarsjs.com/`](http://handlebarsjs.com/)。

正如我们所见，利用`TemplateCache`非常简单，它提供的优势无疑是其最大的卖点。

# 摘要

在本章中，我们学习了 Marionette 提供的一些不同对象来管理视图，例如`Region`和`BabySitter`对象。这种管理确实是必要的，但实现它需要大量的粘合代码。因此，在构建应用程序时将其排除在外是一个很好的理由来开始使用这些对象。

在下一章中，我们将学习如何将我们的应用程序模块化成小的子应用程序模块，以便将网站的不同功能隔离开来，但仍然可以协同工作。
