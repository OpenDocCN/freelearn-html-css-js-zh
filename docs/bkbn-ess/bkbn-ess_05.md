# 第五章。使用视图添加和修改元素

在本章中，我们将探讨以下内容：

+   创建新的`View`类和实例

+   使用视图渲染 DOM 元素

+   将视图连接到模型和集合

+   使用视图响应 DOM 事件

+   确定最适合您项目的渲染样式

# 视图是 Backbone 网站的核心

虽然数据在任何应用程序中都很重要，但没有用户界面的应用程序是不完整的。在网络上，这意味着您有一个 DOM 元素的组合（用于向用户显示信息）和事件处理器（用于接收用户的输入）。在 Backbone 中，这两者都由视图管理；事实上，可以说视图几乎控制了 Backbone 网站上的所有输入和输出。

正如模型封装一个`attributes`对象和集合封装一个`models`数组一样，`Views`在名为`el`的属性中封装一个 DOM 元素。然而，与属性和模型不同的是，`el`不是隐藏的，Backbone 不会监视其变化，因此直接引用视图的`el`是没有问题的：

```js
someModel.attributes.foo = 'bar'; // don't do this
$('#foo').append(someView.el); // feel free to do this
```

要创建一个新的`View`子类，只需像创建新的`Model`和`Collection`子类一样扩展`Backbone.View`，如下所示：

```js
var MyView = Backbone.View.extend({
    // instance properties and methods of MyView go here
}, {
    // static properties and methods of MyView go here
});
```

## 实例化视图

正如我们在第二章中提到的，*使用 Backbone 类的面向对象 JavaScript*，视图在实例化时只接受一个`options`参数。这些选项中最重要的部分是`el`属性，它定义了视图将封装为其`el`的 DOM 元素。视图的`el`选项可以通过以下三种方式之一定义：

+   HTML（`new Backbone.View ({el: <div id='foo'></div>})`）

+   jQuery 选择器（`new Backbone.View ({el: '#foo'})`）

+   DOM 元素（`new Backbone.View ({el: document.getElementById('foo')})`）

您也可以选择不提供`el`选项，在这种情况下，Backbone 将为您创建视图的`el`选项。默认情况下，Backbone 将简单地创建一个空的`DIV`元素（`<div></div>`），尽管您可以在创建视图时提供其他选项来更改这一点。您可以提供以下选项：

+   `tagName`：这更改了生成的元素的标签从`div`到指定的值

+   `className`：这指定了元素应该具有的 HTML `class`属性

+   `id`：这指定了元素应该具有的 HTML `id`属性

+   `attributes`：这指定了元素应该具有的 HTML 属性

技术上，您可以使用`attributes`选项同时指定视图元素的类和 ID，但由于它们对于视图的定义非常重要，Backbone 提供了单独的`id`和`className`选项。

在实例化视图时，除了定义前面的选项外，你也可以选择在 `View` 类中定义它们。例如，如果你想创建一个生成具有 `nifty` 类的 `<form>` 元素的 `View` 类，你应该这样做：

```js
var NiftyFormView = Backbone.View.extend({
    className: 'nifty',
    tagName: 'form'
});
var niftyFormView = new NiftyFormView();
var niftyFormView2 = new Backbone.View({
    className: 'nifty',
    tagName: 'form'});
// niftyFormView and niftyFormView2 have identical "el" properties
```

## 渲染视图内容

虽然 Views 可以通过 `el` 选项来定义它们的初始元素，但它们很少会保持这个 `el` 选项不变。例如，一个 `list` 视图可能会将 `<ul>` 元素作为其 `el` 选项，然后填充这个列表的 `<li>` 元素（可能使用来自集合的数据）。在 Backbone 中，这种内部 HTML 的生成是在视图的 `render` 方法中完成的。

然而，当你尝试使用未修改视图的 `render` 方法时，你很快就会注意到 Backbone 默认实现的问题，如下所示：

```js
render: function() {
    return this;
}
```

如你所见，默认的 `render` 方法实际上并没有渲染任何内容。这是因为不同的视图可能具有完全不同的内容，所以 Backbone 提供仅一种生成内容的方式并不合理。相反，Backbone 完全将视图的 `render` 方法的实现留给了你。

在本章的后面部分，我们将考虑你可能在网站上实现 `render` 方法的各种策略，但在我们到达那里之前，让我们首先检查如何将模型和集合连接到视图，以及视图如何处理事件绑定。

## 将视图连接到模型和集合

当创建视图时，它可以接受两个重要的选项：`Model` 和 `Collection`。这两个选项都是简单的属性选项，也就是说 Backbone 实际上并没有对它们做任何事情，只是将它们作为属性添加到视图中。即便如此，这些属性在你想要显示或编辑之前生成的数据时非常有用。例如，如果你想将一个视图与一个你创建的 `book` 模型关联起来，你可以这样做：

```js
var book = new Backbone.Model({ 
    title: 'Another Fake Title? Why?'
});
var bookView = new Backbone.View({model: book});
// book == bookView.model;
```

当你为你的 `book` 视图编写 `render` 方法时，你可以使用该模型来获取生成适当 HTML 所需的数据。例如，以下是一个简单的 `render` 实现，借鉴了 Backbone 文档的内容：

```js
render: function() {
    this.$el.html(this.template(this.model.toJSON()));
    return this;
}
```

如你所见，假想的 `render` 方法将模型的 `toJSON` 方法的输出传递给视图的模板系统，这可能是为了让模板系统可以使用模型的属性来渲染视图。

## 访问视图的 `el` 元素

一旦创建了一个视图，你可以通过引用其 `el` 属性在任何时候访问它所包裹的元素。你还可以通过引用视图的 `$el` 属性来访问相同元素的 jQuery 包装版本。以下是一个代码示例：

```js
var formView = new Backbone.View({tagName: 'form'});
formView.$el.is('form'); // returns true
```

Backbone 还提供了一个方便的快捷方式，当你想要访问视图元素内部的元素时：`$` 方法。当你使用这个方法时，它实际上等同于从视图元素调用 jQuery 的 `find` 方法。因为元素搜索仅限于仅查看视图的 `el` 元素，而不是整个页面的 DOM，所以它的性能将远远优于全局 jQuery 选择。

例如，如果你创建了一个包含 `<input>` 元素的 `<form>` 元素的视图，你可以使用视图的 `$` 方法来访问 `<input>` 元素，如下所示：

```js
var formView = new Backbone.View({
    el: '<form><input value="foo" /></form>'
});
var $input = formView.$('input');
$input.val(); // == "foo"
```

## 简单谈谈变量命名问题

当在 Backbone（或更一般地，在 JavaScript 中）中处理 jQuery 对象时，可能很难判断给定的变量是指 `View` 元素还是其 `el` 元素。为了避免混淆，许多程序员（包括 Backbone 和 jQuery 的作者）在指向 jQuery 对象的任何变量前都加上 `$` 符号，如下所示：

```js
var fooForm = new Backbone.View({id: 'foo', tagName: 'form'});
var $fooForm = fooForm.$el;
```

虽然这种做法在 Backbone 中并非必需，但采用它可能会让你在未来避免困惑。

# 事件处理

虽然我们之前描述的视图非常适合创建和/或包装现有的 HTML 元素，但它们在响应用户交互方面并不很好。解决这个问题的方法之一是在视图的 `initialize` 方法内部连接事件处理程序，如下所示：

```js
var FormView = Backbone.View.extend({
    id: 'clickableForm',
    initialize: function() {
        this.$el.on('click', _.bind(this.handleClick, this));
    },
    handleClick: function() {
        alert('You clicked on ' + this.id + '!');
    }
});
var $form = new FormView().$el;
$form.click(); // alerts "You clicked on clickableForm!"
```

然而，这种做法有两个问题，如下所述：

+   它的可读性并不高

+   我们必须在创建事件处理程序时绑定它，这样我们才能从其中引用 `this`

幸运的是，Backbone 提供了一种更好的方法，形式为一个可选属性，称为 `events`。我们可以使用这个 `events` 属性来简化之前的示例：

```js
var FormView = Backbone.View.extend({
    events: {'click': 'handleClick'},
    id: 'clickableForm',

    handleClick: function() {
        alert('You clicked on ' + this.id + '!');
    }
});
var $form = new FormView().$el;
$form.click(); // alerts "You clicked on clickableForm!"
```

如你所见，使用 `events` 属性甚至可以让我们不必编写任何 `initialize` 方法，同时它还负责将 `handleClick` 事件处理程序绑定到视图本身。此外，通过以这种方式定义事件处理程序，我们让 Backbone 了解它们，以便它可以根据需要管理它们的移除或重新绑定。

当你实例化一个视图时，Backbone 会调用一个 `delegateEvents` 方法，该方法检查 `events` 属性，绑定其中找到的所有处理程序，然后使用 jQuery 创建适当事件的监听器。还有一个相应的 `undelegateEvents` 方法，可以用来移除所有事件处理程序。通常，你不需要自己调用这些方法，因为 Backbone 会为你调用它们。

然而，如果你在未通知 Backbone 的情况下更改视图的 `el` 元素（例如，`yourView.el = $('#foo')[0]`），Backbone 就不会知道它需要将事件连接到新元素，你将不得不自己调用 `delegateEvents`。

或者，你不必手动更改视图的 `el` 元素，然后再调用 `delegateEvents`，你可以使用视图的 `setElement` 方法同时完成这两步，如下所示：

```js
var formView = new Backbone.View({el: '#someForm'});
formView.setElement($('#someOtherForm'));
// formView.el == someOtherForm
```

# 渲染策略

现在我们已经涵盖了视图的所有功能，是时候回到如何渲染视图的问题了。具体来说，让我们看看当你重写 `render` 方法时，你可以选择的主要选项，这些选项将在接下来的章节中解释。

## 简单模板

渲染的第一种，也许是最明显的方法，是使用一个简单、无逻辑的模板系统。Backbone 文档中提供的 `render` 方法是这种方法的完美示例，因为它依赖于 Underscore 库的 `template` 方法：

```js
render: function() {
    this.$el.html(this.template(this.model.toJSON()));
    return this;
}
```

`template` 方法接受一个字符串，该字符串包含一个或多个特别指定的部分，然后将这个字符串与一个对象组合起来，用该对象的值填充指定的部分。以下示例是最好的解释：

```js
var author ={
    firstName: 'Isaac',
    lastName: 'Asimov',
    genre: 'science-fiction'
};
var templateString = '<%= firstName %> <%= lastName %> was a '+
                     'great <%= genre %> author.';
var template = _.template(templateString);
alert(template(author));
// will alert "Isaac Asimov was a great science-fiction author."
```

虽然 `template` 函数可以与任何字符串和数据源一起使用，但当它作为 Backbone 视图的一部分使用时，通常与 HTML 字符串和 `Backbone.Model` 数据源一起使用。Underscore 的 `template` 函数让我们能够将它们结合起来，轻松创建一个 `render` 方法。

例如，如果你想创建一个包含一个 `<h1>` 标签的 `<div>` 标签，该标签内包含作者姓名和类型，并且围绕类型添加强调（换句话说，一个 `<em>` 标签），你可以创建一个包含所需 HTML 和用于第一个名字、姓氏和类型的占位符的 `template` 字符串。然后我们可以使用 `_.` 模板创建一个 `template` 函数，并在 `render` 方法中使用作者模型的属性调用这个 `template` 函数。

当然，正如我们在第三章中提到的，*使用模型访问服务器数据*，如果我们不直接访问模型属性，会更安全；因此，我们将想要使用模型的 `toJSON` 方法。将所有这些放在一起，我们得到以下代码块：

```js
var authorTemplate =  _.template(
    '<h1>' +
        '<%= firstName %> <%= lastName %> was a ' +
        '<em><%= genre %></em> author.' +
    '</h1>'
);
var AuthorView = Backbone.View.extend({
  template: authorTemplate,
  render: function() {
    this.$el.html(this.template(this.model.toJSON()));
    return this;
  }
});
var tolkien = new Backbone.Model({
    firstName: 'J.R.R.', // Tolkien's actual first name was John
    lastName: 'Tolkien',
    genre: 'fantasy'
});
var tolkienView = new AuthorView({model: tolkien});
var tolkientHtml = tolkienView.render().$el.html();
alert(tolkienHtml);
// alerts "<h1>J.R.R. Tolkien was a <em>fantasy</em> author.</h1>"
```

这种方法的一个主要优点是，因为我们的 HTML 完全分离成一个字符串，我们可以选择将其移动到一个单独的文件中，然后使用 jQuery 或依赖库，如 `Require.js` 带入。如果团队中有这样的设计师，以这种方式存储的 HTML 可以更容易地被设计师处理。

## 高级模板

除了使用 Underscore 的`template`方法外，您还可以使用许多高质量的第三方模板库之一，例如 Handlebars、Mustache、Hamljs 或 Eco。所有这些库都提供了将数据对象与模板字符串结合的基本能力，但它们还提供了在模板中包含逻辑的可能性。例如，这里有一个 Handlebars 模板字符串的例子，它使用基于提供的`isMale`数据属性的`if`语句，在模板中选择正确的性别代词：

```js
var heOrShe = '{{#if isMale }}he{{else}}she{{/if}}';
```

如果我们使用 Handlebars 的`compile`方法将其转换为模板，我们就可以像使用 Underscore 模板一样使用它：

```js
var heOrSheTemplate = Handlebars.compile(heOrShe);
alert(heOrSheTemplate({isMale: false})); // alerts "she"
```

我们将在第十一章中更详细地讨论 Handlebars，*(不)重新发明轮子：利用第三方库*，但您现在需要理解的重要事情是，无论您选择哪个模板库，您都可以轻松地将它作为您视图的`render`方法的一部分。困难的部分是决定您是否想在模板中包含逻辑，如果是的话，包含多少。

## 基于逻辑

除了依赖模板库外，另一种选择是使用 Backbone 的 View 逻辑、jQuery 方法以及/或字符串连接来驱动您的`render`方法。例如，您可以在不使用模板的情况下重新实现前面的`AuthorView`：

```js
var AuthorView = Backbone.View.extend({
  render: function() {
    var h1View = new Backbone.View({tagName: 'h1'});
    var emView = new Backbone.View({tagName: 'em'});
    emView.text(this.model.get('genre'));
    h1View.$el.html(this.model.get('firstName') + ' ' +
                    this.model.get('lastName') + ' was a ');
    h1View.$el.append(emView.el, ' author.');
    this.$el.html(h1View.el);
    return this;
  }
});
```

如您所见，纯逻辑方法既有优点也有缺点。当然，主要优点是我们根本不需要处理任何模板。这意味着我们可以清楚地看到正在使用的逻辑，因为模板库的代码中没有隐藏任何内容。此外，这种逻辑没有限制；您可以做任何在 JavaScript 代码中通常会做的事情。

然而，由于没有使用模板，我们也失去了一部分可读性，如果我们的团队中有设计师想要编辑 HTML，他们会发现`that`代码非常难以处理。在第一个版本中，他们会看到熟悉的 HTML 结构，但在第二个版本中，他们即使（可能）不熟悉编程，也必须与 JavaScript 代码打交道。另一个缺点是，由于我们将逻辑与 HTML 代码混合在一起，我们无法将 HTML 存储在单独的文件中，就像它是模板一样。

## 综合方法

前三种方法都有优点和缺点，您可以选择结合其中一些方法，而不是仅仅满足于一种。例如，您没有理由不能使用模板系统（为了简单起见是 Underscore 的，为了强大是外部的），然后在需要做不适合模板的事情时切换到使用逻辑。

例如，假设你想要渲染一个带有由模型属性派生的`class` HTML 属性的`<ul>`，这似乎是使用 JavaScript 逻辑更容易做到的事情。然而，假设你还想让这个`<ul>`包含基于模板的文本的`<li>`元素，并用集合中模型的属性填充；这似乎是我们最好用模板来处理的事情。幸运的是，没有任何东西阻止你结合这两种方法，如下所示：

```js
var ListItemView = Backbone.View({
    tagName: 'li',
    template: _.template('<%= value1 %> <%= value2 %> are both ' +
                         'values, just like <%= value3 %>'),
    render: function() {
        this.$el.html(this.template(this.model.toJSON()));
        return this;
    }
});
var ListView = Backbone.View.extend({
  tagName: 'ul',
  render: function() {
    this.$el.toggleClass('boldList', this.model.get('isBold'));
    this.$el.toggleClass('largeFontList',
                         this.model.get('useLargeFont'));
    this.$el.empty();
    this.collection.each(function(model) {
        var itemView = new ListItemView({model: model});
        this.$el.append(itemView.render().el);
    }, this);
    this.$el.html(this.template(this.model.toJSON()));
    return this;
  }
});
```

当然，混合方法意味着你不会总是得到两种方法的所有好处。最明显的问题将是，通过 Backbone 的视图或其他 JavaScript 逻辑渲染的任何 HTML 将无法被非编程设计师访问。如果你的团队中没有这样的角色，那么这种限制不会让你烦恼，但如果你有，你可能应该尽量将尽可能多的 HTML 放在模板中。

## 其他渲染考虑因素

除了决定你能在渲染方法中依赖模板以及你能依赖多少之外，还有一些其他的事情在你编写它们时需要考虑。这些选择都不是完全二元的，但你应努力追求尽可能一致的方法。这意味着真正的疑问不在于你应该选择哪一个，而在于你将在何时选择一个而不是另一个。最终，一个一致的战略将使得你和你的同事在编写新的`render`方法或编辑现有的方法时不需要思考应该使用哪种方法。

### 子视图

在 Backbone 中，你通常会想要为容器元素（如`<ul>`元素）创建一个视图，并为它的子元素（如`<li>`元素）创建另一个视图。在创建这些子视图时，你可以选择让子视图创建子元素，或者你可以让父视图创建它们并将一个 jQuery 选择器作为子视图的`el`元素传递，如下所示：

```js
var ListView1 = Backbone.View.extend({
    render: function() {
        this.$el.append(new ListItemView().render().el);
    }
});
var ListView2 = Backbone.View.extend({
    render: function() {
        this.$el.append('<li>');
        new ListItemView({el: this.$('li')}).render();
    }
});
```

在视图中生成子元素的主要优势是封装。通过采取这种方法，你可以将所有与子元素相关的逻辑都保持在子视图中。这将使得代码更容易处理，因为你不需要同时考虑子视图和父视图。

另一方面，当你使用模板时，可能不方便将模板分成几部分来渲染整体容器元素。此外，在父视图中渲染子元素可以在 DOM 生成视图和纯粹的事件处理视图之间提供良好的分离，这可以帮助你更好地整理逻辑。

两种方法都不必走向极端。虽然你可以有一个单页 View，其中包含所有内容的单个模板，或者你可以让每个子 View 生成其 DOM 元素，但你也可以选择在你的 Views 中采用混合方法。实际上，除非你想要一个用于顶级渲染的单页模板，该模板生成整个 DOM，否则你可能想要采取结合的方法，所以真正的问题是你希望依赖每种技术的程度有多大？你越依赖父 View，就越容易看到大局；但如果你更多地依赖子 View 中的 DOM 生成，就越容易封装你的逻辑。

### 可重复渲染与一次性渲染

你还应该考虑是否设计你的`render`方法只渲染一次（通常在 View 首次创建时），或者是否希望能够在响应变化时多次调用你的`render`方法。只设计为渲染一次的`render`方法编写和操作起来最简单；当你的 View 的 DOM 元素需要更改时，你只需再次调用 View 的`render`方法，DOM 就会更新。

然而，这种方法也可能导致性能问题，尤其是如果涉及的 View 包含大量子 View。为了避免这种情况，你可能想要编写`render`方法，而不是盲目地替换其内容，而是根据变化来更新它们。例如，如果你有一个`<ul>`元素，其中一些子`<li>`元素需要根据用户操作出现或消失，你可能希望你的`render`方法首先检查`<li>`元素是否已经存在，然后简单地应用更改它们的显示样式，而不是每次都从头开始重建整个`<ul>`。

再次强调，你可能根据你编写的 View 类型混合两种方法。如果你将性能视为高优先级，那么你可能会仔细思考如何在`render`方法中重用之前生成的 DOM 元素。另一方面，如果性能不太重要，为一次性 DOM 生成设计你的`render`方法将使它的逻辑更加简单。此外，如果出现性能问题，你总是可以更改相关的`render`方法以重用现有元素。

### 返回值 – 这个或这个.$el

Backbone 文档中的示例`render`方法在完成时返回`this`。这种方法允许你通过简单地调用`render`方法的返回值来轻松访问`el`属性，如下所示：

```js
var ViewThatReturnsThis = Backbone.View.extend({render: function() {
        // do the rendering
        return this;
    }
});
var childView = new ViewThatReturnsThis();
parentView.$el.append(childView.render().$el);
```

然而，你可以让它更容易访问 View 的`el`，通过返回那个`el`而不是这个：

```js
var ViewThatReturns$el = Backbone.View.extend({render: function() {
        // do the rendering
        return this.$el;
    }
});
var childView = new ViewThatReturns$el();
parentView.$el.append(childView.render());
```

从`render`方法返回`$el`的缺点是，你不能再从`render`方法的返回值链式调用其他 View 方法：

```js
parentView.$el.append(childView.render().nextMethod());// won't work
```

因此，这个选择实际上取决于你计划对视图进行多少次后渲染以及你计划从视图外部调用多少次这些后渲染方法（如果你在渲染方法内部调用它们，它们的返回值是不相关的）。如果你不确定，最安全的做法是遵循 Backbone 文档中的 `render` 方法，并返回 `this`，因为它提供了最大的灵活性（代价是代码稍微不那么简洁）。

### 小贴士

无论你选择什么，你可能会希望在整个视图之间保持一致性。如果不这样做，你将发现自己每次想要使用视图的 `render` 方法时，都需要不断查找。

# 摘要

在本章中，我们探讨了 Backbone 的 `View` 类。你学习了如何通过实例化视图来创建 DOM 元素，以及如何将现有元素传递给新的视图。你还学习了如何使用视图在视图的 `el` 上设置事件处理器，以及如何使用 `undelegate` 和 `delegate` 方法移除/重新附加这些处理器。最后，我们考虑了 Backbone（故意）为 `render` 方法提供的空实现，这为我们提供了许多不同的选项来实现自己的 `render` 方法，以及哪些因素影响了这些选择。

在下一章中，我们将探讨 Backbone 的最后一个类，即 `Router` 类。这个类允许我们仅使用视图而不是单独的 HTML 文件来模拟传统网页。
