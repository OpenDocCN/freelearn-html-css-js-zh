# 第七章。将方钉塞入圆孔——高级 Backbone 技巧

在本章中，我们将探讨 Backbone 的各种高级用法，包括以下内容：

+   用方法代替属性

+   覆盖类的构造函数

+   使用混入（mixins）在类之间共享逻辑

+   实现发布/订阅模式

+   在 Backbone `Views` 中包装来自其他库的控件

# 提升一个层次

在前几章中，我们讨论了使用 Backbone 的四个类的基本方法，虽然这些基本方法已经足够让你在 Backbone 中变得高效，但我们故意省略了一些更复杂的细节。在本章中，我们将探讨这些省略的细节以及它们如何与某些高级 Backbone 技巧相关。

然而，在我们继续之前，重要的是要注意，本章中讨论的所有技术都是为了解决特定问题而设计的，而不是作为通用模式。虽然使用这些技术中任何一项都没有“错误”，但它们会使你的代码更加复杂，因此，更难以直观理解和维护。因此，建议除非你确实需要解决特定问题，否则避免使用本章中讨论的技术。 

# 用方法代替属性

Backbone 类的许多属性都可以定义为函数而不是原始数据类型。实际上，我们在 `Models` 和 `Collections` 的 `url` 属性中已经看到了这种行为的一个例子。正如我们在第三章使用模型访问服务器数据中学到的，这个属性可以是一个简单的字符串，但如果需要逻辑来生成更复杂的 URL，则可以使用返回字符串的函数。

这种相同的方法可以用于许多 Backbone 属性，包括以下内容：

+   对于模型：

    +   `defaults`

    +   `url`

    +   `urlRoot`

+   对于视图：

    +   `attributes`

    +   `className`

    +   `events`

    +   `id`

    +   `tagName`

+   对于路由器：

    +   `routes`

例如，假设你想要一个 `View`，它可以根据是否有 `Collection` 生成 `<input>` 或 `<select>`。通过为 `Views` 的 `tagName` 属性使用函数而不是字符串，我们可以做到这一点：

```js
var VariableTagView = Backbone.View.extend({
    tagName: function() {
        if (this.collection) {
            return 'select';
        } else {
            return 'input';
        }
    }
});
```

## 将 Collection.model 作为工厂方法使用

`Collection` 类也有一个可以用函数替换的属性，但与其它属性不同，这个属性通常不是一个原始数据类型……它是一个 Backbone `Model` 子类。正如在第四章使用集合组织模型中讨论的，`Collection` 类的 `model` 属性决定了该 `Collection` 将创建哪种类型的 `Model`。通常，每个 `Collection` 类只能有一个 `model` 属性，因此只能创建一种类型的 `Model`。

然而，有一种方法可以绕过这个限制：如果我们用返回新`Models`的函数替换我们的`Collection`的`model`属性，Backbone 将愉快地忽略`model`属性实际上根本不是`Model`子类的事实。例如，假设我们的服务器有一个返回两种类型书籍（虚构和非虚构）JSON 的"/book" API 端点。假设我们有两个不同的`Model`类，每个类对应一种类型的书籍。如果我们想要有一个能够从单个端点获取两种类型书籍的 JSON 的`Collection`类，但使用适当的`model`函数来实例化每种类型的书籍，我们可以使用一个工厂`model`函数，如下所示：

```js
var FictionBook = Backbone.Model.extend({
    // insert logic for fiction books here});
var NonFictionBook = Backbone.Model.extend({
    // insert logic for non-fiction books here});
var FictionAndNonFictionBooks = Backbone.Collection.extend({
    model: function(attributes, options) {
        if (attributes.isFiction) {
            return new FictionBook(attributes, options);
        } else {
            return new NonFictionBook(attributes, options);
        }
    },
    url: '/book'});
```

# 覆盖类构造函数

每当 Backbone 类被实例化时，它都会运行其`initialize`方法中定义的代码。然而，这段代码是在新对象实例化之后才运行的。这意味着即使您为`Model`类定义了一个`initialize`方法，该`Model`类的属性已经在您的`initialize`代码被调用之前设置了。

通常，这是好事，因为像`Model`类的属性已经设置这样的东西很方便，但有时我们希望在发生之前就掌握控制权。例如，让我们重新想象我们之前的虚构和非虚构书籍的场景，但这次我们不是有一个可以创建两种类型书籍的单个`Collection`类，我们想要一个只能创建一种类型的`Collection`，并且我们希望这种类型由我们给`Collection`类实例化的第一本书决定。

换句话说，如果给我们的`Collection`类提供的第一个书具有`isFiction`属性，我们希望我们的`Collection`类具有`FictionBook`的`Models`属性；否则，我们希望它具有`NonFictionBook`的`Models`属性。如果我们在这个`initialize`方法内部这样做，那么在`initialize`方法运行之前，“model”选项已经在`Collection`类上设置了，并且我们传递给它的任何`Model`属性已经转换成了`Models`。因此，我们需要添加在`initialize`之前运行的逻辑。我们可以通过覆盖`Collection Constructor`方法的构造函数来实现这一点，如下所示：

```js
var FictionOrNonFictionBooks = Backbone.Collection.extend({
    constructor: function(models, options) {
        if (models[0].isFiction) {
            options.model = FictionBook;
        } else {
            options.model = NonFictionBook;
        }
        return Backbone.Collection.apply(models, options);
    }
});
```

注意我们如何在我们的覆盖版本中仍然调用了原始的`Collection` `Constructor`方法；如果我们没有这样做，Backbone 的正常逻辑将不会运行，并且在我们完成时甚至可能没有正确的`Collection`类。

与本章讨论的其他技术一样，不建议您经常覆盖构造函数。如果可能的话，您应该覆盖`initialize`方法。构造函数的覆盖主要应留给那些您希望在 Backbone 对象实际创建之前预处理传入参数的情况。

# 类混合

Backbone 提供了一个非常强大的类系统，但有时，即使是这个类系统也不够用。例如，假设你有几个类，它们都包含几个相同的方法。流行的**不要重复自己**（**DRY**）编程原则建议你应该消除方法的重复版本，而是在代码中的一个地方定义它们。

通常，你可以通过将这些方法重构为这些类的一个公共父类来实现这一点。但如果这些类不能共享一个公共父类怎么办？例如，如果其中一个类是`Model`类，而另一个是`Collection`类呢？

在这种情况下，你需要依赖一种称为混入（mixin）的东西。混入只是一个包含一个或多个你想要在几个类之间共享的方法（甚至原始属性）的对象。例如，如果我们想在几个类之间共享一些与日志记录相关的功能，我们可以创建一个包含这些方法的混入，如下所示：

```js
var loggingMixin = {
    startLogging: function() {
        Logger.startLogging(this);
    },
    stopLogging: function() {
        Logger.stopLogging(this);
    }
}
```

一旦你定义了你的混入（mixin），下一步就是将其“混合”到你的类定义中。这意味着修改类的`prototype`以包含混入中的方法。例如，假设我们有一个书`Model`：

```js
var Book = Backbone.Model.extend({
    defaults: {currentPage: 1},

    read: function() {
        // TODO: add logic to read a book here
    }
});
```

因为 Backbone 的语法使得创建我们的新类变得如此简单，所以我们很容易忽略这样一个事实：我们实际上在这里定义了一个原型对象，但如果我们稍作改变，它突然就变得明显了：

```js
var bookPrototype = {
    defaults: {currentPage: 1},

    read: function() {
        // TODO: add logic to read a book here
    }
};Book = Backbone.Model.extend(bookPrototype);
```

一旦我们分离了书的原型（prototype），我们就可以用我们的混入来增强它。我们可以手动做这件事：

```js
bookPrototype.startLogging = loggingMixin.startLogging;
bookPrototype.stopLogging = loggingMixin.stopLogging;
```

然而，如果我们有很多方法要混入，或者我们的混入后来添加了新方法，这就不太适用了。相反，我们可以使用 Underscore 的`extend`函数来通过单个命令将混入中的所有方法扩展到`Book`原型上：`_.extend(bookPrototype, loggingMixin);`

就这一行代码，我们就为我们的`Book`类添加了一整套与日志记录相关的功能。此外，我们还可以通过仅使用一行代码将这些方法与任何其他类共享，而无需更改任何相关类的父类。

当然，重要的是要注意，因为我们是在现有原型上应用混入，所以它将覆盖与原型共有的任何属性。换句话说，假设我们的`Book`类已经有一个自己的`startLogging`方法来阻止日志记录：

```js
bookPrototype = {
    defaults: {currentPage: 1},

    read: function() {
        // TODO: add logic to read a book here
    },
    startLogging: $.noop // don't log this class
};
```

我们将通过应用混入来擦除这个方法，即使我们不想这样做，也会在我们的类上实现日志记录。进一步来说，即使`Book`没有这样的方法，未来的开发者可能会添加一个，然后，当它不起作用时感到困惑。如果未来的开发者没有看到（或没有理解）混入行，他可能会花几个小时查看`Book`的父类，试图弄清楚发生了什么。

由于这个问题，在设计你的类时，通常最好尽可能多地依赖标准面向对象编程，并且只有在无法通过正常类层次结构共享方法时才使用混入。然而，在这些情况下，混入可以是一个强大的工具，并作为 JavaScript 中但大多数其他语言中不可能实现的另一个例子（祝你好运，将方法混入一个 Java 类！）。

# 发布/订阅

Backbone 类通常最终会紧密耦合在一起。例如，一个 `View` 类可能会监听 `Model` 类数据的变化，然后，当数据发生变化时，它可能会查看 `Model` 的 `attributes` 来确定要渲染的内容。这种做法将 `View` 类耦合到 `Model` 类，这在正常情况下是好事，因为它允许你定义两个类之间所需的确切关系，同时仍然保持你的代码相对简单和可维护。

当你只有少数 `Models` 和 `Views` 时，以这种方式管理它们之间的关系很容易。然而，如果你正在构建一个特别复杂的用户界面，那么这种相同的耦合可能会变成一种障碍。想象一下，有一个单页面上有大量 `Collections`、`Models` 和 `Views`，它们都在监听并响应彼此的变化。每当发生单个变化时，它都可能引起连锁反应，导致进一步的改变，然后这些改变又可能导致更多的改变，如此等等！这几乎使得理解正在发生什么变化以及它们将产生什么影响变得不可能；在最坏的情况下，它们可以创建无限循环或使修复错误变得困难。

对于这样一个复杂系统，解决方案是使用发布/订阅模式（或简称 pub/sub）来解耦各个组件（`Collections`、`Models` 和 `Views`）。在这个模式中，所有组件仍然通过事件进行通信，但不是每个 `Collection` 和 `Model` 类都有自己的事件总线，所有涉及的 `Collection` 和 `Model` 类共享一个单一的全局事件总线，消除了它们之间的直接连接。每当一个组件想要与另一个组件通信时，它会发布（触发）一个其他组件已订阅（监听）的事件。

要在 Backbone 中实现这种模式，我们需要一个全局事件总线，而实际上 Backbone 已经为我们提供了一个：Backbone 对象本身。就像 `Models` 和 `Collections` 一样，Backbone 对象既有 "on"（订阅）方法，也有 "trigger"（发布）方法。这两个方法的工作方式与其他 Backbone 对象相同。

常常从发布/订阅模式中受益的一种应用类型是游戏，因为它们通常有很多不同的 UI 组件被各种不同的数据源更新。让我们想象一下，我们正在构建一个三人游戏，其中每个玩家都由一个`Model`类表示。由于我们希望保持玩家`Model`与服务器更改同步，我们以周期性的间隔获取这些`Model`：

```js
var Player = Backbone.Model.extend();
var bob = new Player({name: 'Robert', score: 2});
var jose = new Player({name: 'Jose', score: 7});
var sep = new Player({name: 'Sepehr', score: 4});
window.setInterval(1000, function() {
    bob.fetch();
    jose.fetch();
    sep.fetch();
});
```

让我们进一步想象，我们有一个计分板`View`，它需要根据玩家分数的变化来更新自己。当然，在现实世界中，如果我们只有一个`View`，我们很可能甚至不需要发布/订阅模式，但让我们使这个例子尽可能简单。为了避免将`View`直接耦合到玩家`Model`，我们将让它监听 Backbone 对象上的`scoreChange`事件：

```js
var Scoreboard = Backbone.View.extend({
    renderScore: function(player, score) {
        this.$('input[name="' + player + '"]').html(score);
    });
});
var scoreboard = new Scoreboard();
Backbone.on('scoreChange', scoreboard.renderScore, scoreboard);
```

现在，为了使这些`scoreChange`事件发生，我们需要让我们的`Player`类触发它们。我们还需要一种方法来判断分数是否已更改，但幸运的是，每个`Model`类都有一个`hasChanged`方法，它正好可以做到这一点。我们可以在重写的`fetch`方法中使用这个方法来触发我们的`scoreChange`事件，如下所示：

```js
Player = Backbone.Model.extend({
    fetch: function() {
        var promise = Backbone.Model.prototype.fetch.apply(
                          this, arguments
                      );
        promise.done(function() {
            if (this.hasChanged('score')) {
                Backbone.trigger('scoreChange', this.get('name'), 
                                 this.get('score'));
            }
        });
        return promise;
   }
});
```

如您所见，`Model`类和`View`现在是完全独立的。`Model`类通过事件本身传递任何`View`所需的信息，当它调用`Backbone.trigger`时；否则，所有涉及的类都对彼此一无所知。

当然，与任何这些高级技术一样，这种方法也有其缺点，主要是封装性不足。当你有一个`View`直接获取`Model`类时，你知道连接两个类的确切代码，如果你想要重构或更改该代码，很容易确定会受到什么影响。然而，在使用发布/订阅模式的情况下，确定可能受到潜在更改影响的代码可能会困难得多。

在许多方面，这类似于使用全局变量和使用局部变量之间的权衡：虽然前者在最初更强大且易于处理，但它们缺乏限制性可能会使它们在长期使用中更难以处理。因此，尽可能避免依赖发布/订阅模式，而是依赖绑定到特定 Backbone 对象的监听器。然而，当你遇到需要代码中多个不同部分之间通信的问题时，依赖这种模式并在 Backbone 对象上监听/触发事件可以提供一种更优雅的解决方案。

# 包装其他库的组件

Backbone 是一个非常强大的框架，但它不能做所有事情。如果你想向你的网站添加日历小部件、富文本编辑器或节点树，你可能想使用另一个库，该库可以为你提供这样的组件（例如，jQuery UI、TinyMCE 或 jsTree）。然而，仅仅因为你想要使用除 Backbone 之外的工具，并不意味着你必须放弃 Backbone 类的所有便利和功能。事实上，创建包装第三方组件的 Backbone `View` 类有许多好处。

首先，将组件包装在 `View` 中允许你指定使用此组件的通用方式。例如，假设你想要在你的网站上使用 JQuery UI 日历小部件（或 `datepicker`）。在 jQuery UI 中，如果你想让你的日历包含月份选择控件，你必须在每个创建日历的代码位置提供 `changeMonth: true` 选项：

```js
// File #1
$('#datepicker1').datepicker({changeMonth: true});
// File #2
$('#datepicker2').datepicker({changeMonth: true});
```

然而，假设你创建了一个如下包装 `datepicker` 小部件的 `View` 类：

```js
var CalendarView = Backbone.View.extend({
    initialize: function() {
        this.$el.datepicker({changeMonth: true});
    }
});
```

你可以在代码中需要日历的任何地方使用这个类，你不必担心忘记提供 `changeMonth` 选项：

```js
// File #1
new CalendarView({el: '#datepicker1'});
// File #2
new CalendarView({el: '#datepicker2'});
```

如果你的团队中有一个完全不同的开发者想要添加一个日历，即使他对 jQuery UI 或 `changeMonth` 选项一无所知，他仍然能够仅通过使用你创建的 `View` 来创建一个适合你网站的日历。这种将特定组件相关的所有逻辑封装起来并定义自己使用接口的能力是这种包装方法的主要好处之一。

另一个好处是能够轻松地更改组件。例如，假设有一天，我们决定我们希望我们网站上的所有日历都也具有一个年份选择的下拉菜单（换句话说，我们希望它们都具有 `changeYear: true` 选项）。如果没有包装的 Backbone `View`，我们就必须找到网站上所有使用 `datepicker` 小部件的地方并更改其选项，但有了我们的 `View`，我们只需要在代码中的一个地方更新。

当然，这种技术也有其缺点。一个明显的缺点是，如果我们向我们的网站添加任何不使用相同选项的组件，例如没有月份选择器的 `datepicker` 小部件，我们就需要重构我们的包装 `View` 以允许这种可能性。

另一个问题是有包装的组件可以在不调用其包装 `View` 上的 `remove` 方法的情况下从 DOM 中移除自己，这阻止了该 `View` 被垃圾回收。通常情况下，这不会成为问题，因为内存中一些额外的“僵尸” `View` 小部件不会对性能产生重大影响。然而，如果你有大量这些包装小部件，那么你可能想使用小部件的事件系统来确保当其包装小部件离开页面时，包装 `View` 总是会调用 `remove` 方法。例如：

```js
CalendarView = Backbone.View.extend({
    initialize: function() {
        this.$el.datepicker({
            changeMonth: true,
                         onClose: _.bind(function() {
                             this.remove();
                         }, this)
        });
    }
});
```

最后，虽然将小部件的使用细节隐藏在 `View` 中可能很方便，但它也可能无意中隐藏了有关此小部件的相关细节，从而让其他开发者无法了解。例如，一个小部件可能有一个特定的副作用，但由于其他开发者只是使用小部件的包装 `View` 而从未阅读原始小部件的文档，他们可能不会意识到这个副作用。

因此，确保这样的 `Views` 被适当文档化是很重要的。如果被包装的组件有任何形式的副作用，无论是性能上的还是其他方面，这种副作用也应该在 `View` 本身上仔细记录，或者你应该确保团队中的所有开发者都理解底层组件的工作原理。

# 摘要

在本章中，我们探讨了使用 Backbone 的几种高级技术。首先，我们学习了 Backbone 的许多属性可以被方法所替代，特别是如何使用 `Collection` 类的 `model` 属性以这种方式生成不同类型的 `Models`。然后，我们学习了如何覆盖类 `constructor` 以在 `initialize` 之前访问和/或操作其参数，以及如何使用混入（mixins）在原本无关的类之间共享方法。最后，我们考察了如何使用 Backbone 对象本身来实现发布/订阅（pub/sub）模式，以及如何使用 Backbone `Views` 来“包装”来自其他 JavaScript 库的组件。

在下一章中，我们将探讨 Backbone 的性能影响以及如何避免最常见的性能陷阱。
