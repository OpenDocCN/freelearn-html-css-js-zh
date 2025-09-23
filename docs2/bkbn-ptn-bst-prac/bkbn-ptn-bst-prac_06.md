# 第六章. 与事件、同步和存储一起工作

在前面的章节中，我们分别详细讨论了每个 Backbone 组件（视图、模型、集合和路由器）。在本章中，我们将讨论自定义事件、`Backbone.sync()` 方法以及 `Backbone.LocalStorage`。尽管这些主题之间并不完全相关，但我们将它们放在一个章节中，因为我们需要在进入下一章的应用程序架构和模式之前涵盖它们。

事件始终被认为是 JavaScript 中最强大的概念之一。它们是观察者模式（一个著名的松散耦合设计模式）的表示，并被大多数 JavaScript 库使用。在 Backbone 中，`Backbone.Events` 是一个非平凡的模块，可以与任何对象一起使用以具有事件相关功能。这是在 Backbone 文档中定义 `Backbone.Events` 的方式（[`backbonejs.org/#Events`](http://backbonejs.org/#Events)）：

> 事件是一个可以混合到任何对象中的模块，它赋予对象绑定和触发自定义命名事件的能力。

在本章中，我们将讨论为什么事件对于 Backbone 应用程序开发很重要，以及我们如何使用它们来实现更高的重用性和更结构化的应用程序架构。本章将涵盖的主要内容包括：

+   **自定义事件**：自定义事件是由应用程序为了某个目的而初始化的，这个目的不是我们使用的基库所服务的。我们将学习如何在 Backbone 中创建和使用自定义事件。

+   **事件分发器**：有时，我们寻求一个应用级的事件管理器，它可以作为一个基于事件通信的集中工具。应用程序的不同组件可以通过这个事件管理器相互交互，而无需直接相互通信。在本节中，我们将学习如何使用我们的应用程序创建和使用这样的事件分发器。

+   **方法覆盖**：在本主题中，我们将学习如何通过覆盖 Backbone 的 `sync()` 方法来创建针对公共 REST API 或 `LocalStorage` 的不同持久化策略。

+   **离线存储**：`Backbone.LocalStorage` 适配器可以与任何 Backbone 模型或集合一起使用，将数据保存到 `LocalStorage` 数据库中。

# 理解自定义事件

在 JavaScript 中，创建和使用自定义事件并不是什么大问题——所有主要的 JavaScript 库都严重依赖于它们自己的事件来实现组件的松散耦合。每个组件都有一组自定义事件，以实现更好的重用性和与应用程序的集成。

在 Backbone 中创建一个自定义事件相当简单——任何扩展了 `Backbone.Events` 类的对象都会获得所有与事件相关的功能，即监听、触发和删除事件。Backbone 的 `View`、`Model`、`Collection` 和 `Router` 是扩展了 `Backbone.Events` 类的主要组件，并且当需要时，你可以在它们中的任何一个上触发一个自定义事件：

```js
var myView = new Backbone.View();
myView.on('myevent', function () {
  console.log('"myevent" is fired');
});

myView.trigger('myevent');
```

在这里，我们创建了一个 Backbone 视图实例，向其注册一个自定义事件，并触发该事件。一旦事件被触发，注册的函数就会立即按预期运行。

### 小贴士

**避免回调，使用自定义事件**

这个标题并不意味着你应该总是使用事件而不是回调方法。这取决于程序员想要实现的目标，这绝对是一个架构选择。我们打算讨论这个点，以便理解在哪些情况下事件比回调提供了更多的灵活性。每当有需要通知他人关于任何任务特定条件的情况时，你会发现自定义事件比回调函数是一个更好的选择。入门级开发者常常陷入这个陷阱，最终使用回调方法，这通常提供了一种更私密和隔离的方法。

在接下来的部分，我们将通过一个简单的示例来展示自定义事件必要性的例子。

## 自定义事件的一个简单案例研究

假设我们有一个 `登录` 对话框，一旦用户输入用户名-密码组合并点击 **提交** 按钮，就会向服务器发送一个请求来验证登录。验证成功后，用户关闭对话框并执行一些其他任务，例如存储 cookies、更改页面标题等。在这种情况下，我们通常使用回调方法，并将所有登录后的功能放在其中。如果登录成功后需要调用 10 个不同的函数，每个函数在单独的对象上执行，那会怎样？你可能需要在那个回调方法中包含所有这些对象引用来逐个调用方法。虽然这可以无问题地实现，但为什么不在用户登录成功后只触发一个自定义事件 `loggedin` 呢？已经监听 `loggedin` 事件的 10 个对象就可以相应地调用相关方法：

```js
var Login = Backbone.View.extend({
  'click #login-btn': 'doLogin',
  doLogin: function () {
    var me = this;

    // send a login request
    $.ajax({
      url: '/login',
      method: 'POST',
      data: {
        username: 'foo',
        password: 'foo'
      },
      success: function (response) {
        me.trigger('loggedin', response);
      }
    });
  }
});

var loginView = new Login();
login.doLogin();
```

需要执行那些登录后任务的其它组件应该已经监听了那个 `loginView` 的 `loggedin` 事件，并且回调函数将在事件触发时立即执行：

```js
// in some other component
loginView.on('loggedin', function(response){
  // Do something
});
```

在触发事件时，你也可以传递一些参数；这些数据将在事件回调方法中以参数的形式可用：

```js
// Pass multiple data
me.trigger('loggedin', response, 'foo', 'bar');

// All the passed params will be available as function arguments
loginView.on('loggedin', function(response, foo, bar){
  console.log(response, foo, bar);
});
```

# 使用事件分发器

注意上一个场景中的一个细节：要监听 `loggedin` 事件，应用程序的所有其他组件都应该有对该 `loginView` 的引用。这真的是必需的吗？对于一些组件来说，保持对 `loginView` 的引用可能看起来无关紧要，但它们必须这样做，因为它们需要在这个 `loginView` 对象上监听 `loggedin` 事件。即使在开发简单应用程序时，这种依赖注入也可能很痛苦。有时我们可能需要一个充当中央事件管理器的通用对象，并且可以在整个应用程序中用来触发和监听事件。最简单的事件分发器可以按以下方式定义：

```js
var vent = _.extend({}, Backbone.Events);

// Listen to a custom event
vent.on('customevent', function(){
  console.log('Custom event fired');
});

// Fire the event
vent.trigger('customevent');
```

当我们将 `vent` 变量在应用程序级别公开时，它可以作为一个中央事件分发器来发布和订阅事件。这种模式被称为 Pub/Sub 模式，在基于模块或小部件的应用程序架构中使用时非常有益。

### 小贴士

您应该理解在什么情况下这将是一个好的选择。当您有太多组件需要监听（正如我们在登录示例中看到的那样）或者当您有一些完全无关的对象需要相互通信时，应使用事件分发器。

在使用通用事件分发器工作时，您可能会遇到的主要问题是，当通过单个事件分发器注册了太多事件时，发布者和订阅者的数量可能会失去控制。例如，假设我们有两个模块，`User` 和 `Company`，并且这两个模块分别订阅了事件分发器上的名为 `addcomment` 的事件；它们被定义为如下：

```js
vent.on('addcomment', user.addComment);
vent.on('addcomment', company.addComment);
```

注意，事件名称相同，但要调用的函数不同。因此，如果您想通知另一个订阅者同一事件，您需要首先清除该事件的全部其他订阅者，然后发布该事件。然而，还有一些其他简单的解决方案可以解决这个问题，例如创建多个事件分发器或使用不同的事件命名空间。

## 创建多个事件分发器

为单个模块或功能定义单独的事件分发器可以为我们之前描述的问题提供一个解决方案：

```js
App.userVent = _.extend({}, Backbone.Events);
App.documentVent = extend({}, Backbone.Events);
```

现在，不同分发器的相同事件名称永远不会冲突：

```js
App.userVent.on('addcomment', user.addComment);
App.documentVent.on('addcomment', company.addComment);
```

## 使用不同的事件命名空间

这就像在自定义事件中使用特定的命名约定一样简单：

```js
App.vent.trigger('before:login');
App.vent.trigger('after:login');
App.vent.trigger('user:add:comment');
```

在事件名称中添加冒号并不会使事件变得特殊，但它确实使其不同且独特，因为它现在与应用程序的某些特定模块相关联。这是 JavaScript 开发者广泛使用的一种约定，我们鼓励您在需要时使用它。

# 使用 listenTo() 方法避免内存泄漏

内存管理是任何应用程序都非常重要的一部分。通常，对于前端开发，开发者不会太在意内存泄漏；然而，当我们开发单页前端密集型应用时，这一点并不成立。这类应用处理许多前端组件和最少的页面刷新次数，这可能会为内存泄漏创造几个机会。在开发这类应用时，我们应该始终小心地清理对象销毁时的事件。为了通过例子来理解这一点，假设我们有一个显示其模型数据的视图。每当在该模型上触发`change`事件时，视图的`render()`方法就会被调用：

```js
// Memory leak
var MyView = Backbone.View.extend({
  tpl: '<%= name %>',
  model: new Backbone.Model({
    name: 'Suramya'
  }),

  initialize: function () {
    this.model.on('change', this.render, this);
  },

  render: function () {
    var html = _.template(this.tpl, this.model.toJSON());
    this.$el.html(html);
    return this;  
  }
});

var myView = new MyView();
$(document.body).append(myView.render().el);

myView.model.set('name', 'Arup');
myView.remove();
```

现在，正如你所看到的，我们在模型上注册了一个`change`事件，以便每当其任何属性发生变化时，`render`方法就会被调用。然后我们创建了一个视图实例，更改了模型的`name`属性，并销毁了视图。`remove()`方法销毁了`view`实例，并将视图从 DOM 中移除。

整个过程按预期工作，尽管存在一个小问题。当你用 JavaScript 创建一个视图时，你创建了 DOM 节点并将事件监听器绑定到它们上。当你从 DOM 中移除节点时，它们的监听器仍然持有对它们的引用。结果，你的 JavaScript 引擎不会自动回收这些节点，因为作用域中仍然有对它们的引用。在我们的情况下，即使视图被销毁，模型上的`change`事件监听器仍然存在，我们需要明确处理它。我们如何做到这一点？我们可以在视图中添加一个`close()`方法，并在销毁视图之前在这个方法中解绑所有此类事件：

```js
close: function () {
  this.model.off('change', this.render, this);
  this.remove();
}
```

现在一切都被正确清理了。但我们是否需要为所有视图都这样做？不，因为 Backbone V9.9 引入了一个`listenTo()`方法，它告诉一个对象去监听另一个对象的某个事件：

```js
this.listenTo(this.model, 'change', this.render);
```

这与`on()`方法的工作方式完全相同，但它的优势在于它允许对象跟踪事件，并且可以在稍后一次性移除它们。因此，我们不需要额外的`close()`方法来在销毁视图之前解绑所有事件。相反，Backbone 视图的`remove()`方法现在会通过调用`stopListening()`方法来清理任何已绑定的事件。

因此，当你想自己处理处理器，并且不会出现事件清理或僵尸处理等场景时，请使用`on()`方法。否则，选择`listenTo()`，我们将在 Backbone 视图的上下文中发现它非常有用。

# 覆盖 Backbone 的 sync()方法

Backbone 为所有数据通信提供了一个单一的网关。所有的数据请求都是通过 `sync()` 方法发送的，每当进行任何 `CRUD` 操作时都会调用这个方法。这个 `sync()` 方法执行多项任务，例如设置 URL、参数和内容类型，以及模拟不支持 `PUT` 和 `DELETE` 请求的老式浏览器的 HTTP 请求。每当我们在模型或集合上调用 `fetch()` 或 `save()` 方法时，都会执行 `sync()` 方法。

但我们何时需要覆盖这个方法呢？有时，你可能需要一个单独的 REST API 方法的实现，而 Backbone 并不提供。这可能适用于某个模型或集合，或者这种实现可能在整个项目中持续存在。这就是 Backbone 默认如何编写方法映射的方式：

```js
var methodMap = {
  'create': 'POST',
  'update': 'PUT',
  'patch': 'PATCH',
  'delete': 'DELETE',
  'read': 'GET'
};
```

现在，你可能有一个特定的模型或集合，它将监听除默认之外的其他 API，比如 Google 或 Twitter API，而你无法更改它。或者，你可能想实现一个使用浏览器本地存储来操作数据的离线存储。在这种情况下，你需要覆盖该集合或模型的 `sync()` 方法，或者如果它在整个应用程序中是通用的，你需要覆盖 `Backbone.sync()` 方法。让我们通过一个例子来理解其重要性。在这里，我们希望我们的 `User` 模块直接与公共 API `FooApi` 交互：

```js
// FooApi is a public api with 'add', 'edit', 'read' 
// and 'delete' methods
var User = Backbone.Model.extend({
  sync: function (method, model, options) {
    options || (options = {});

    switch (method) {
    case 'create':
      FooApi.add(options.data);
      break;

    case 'update':
      FooApi.edit(options.data);
      break;

    case 'delete':
      FooApi.delete({
        id: options.data.id
      });
      break;

    case 'read':
      FooApi.read({
        id: options.data.id
      });
      break;
    }
    // Other stuff
  }
});

var user = new User({
  name: 'Manali',
  age: 29
});

// This will call FooApi.add() method
user.save();
```

看看我们是怎样使用 `switch` case 来调用每个数据操作方法的 `FooApi` 方法。现在，当你对用户实例调用 `save()` 方法时，它将直接调用 `FooApi.add()` 方法。你可以用类似的方式使用其他数据操作。所以，这就是 Backbone 的 `sync()` 方法被覆盖以创建一个映射其他 API 的模型和方法包装器的方式。

# 使用 Backbone.LocalStorage 适配器的离线存储

在上一节中，我们看到了如何通过覆盖 `Backbone.sync()` 方法来提供定制的数据操作，包括模型和集合。大多数时候，我们使用 HTML5 的 `LocalStorage` 功能在浏览器中存储我们的数据以供离线浏览。这对于开发移动网站和移动网络应用时存储小数据来说是一个相当常见的需求。`LocalStorage` 通信也可以通过使用 `sync()` 方法以与上一节中通过覆盖 `sync()` 方法所用的相同技术来完成。

我们不会自己创建这个解决方案，而是会查看一个优秀的适配器，`Backbone.LocalStorage` ([`documentup.com/jeromegn/backbone.localStorage`](http://documentup.com/jeromegn/backbone.localStorage))，这是由 Jerome Gravel-Niquet 开发的，并被 Backbone.js 开发者社区广泛用于与 `LocalStorage` 交互。这个适配器可以插入到任何模型或集合中，从而使得它们可以使用 `save()` 或 `fetch()` 方法与 `LocalStorage` 通信，如下所示：

```js
var Users = Backbone.Collection.extend({
  model: Backbone.Model,
  localStorage: new Backbone.LocalStorage("users")
});

var users = new Users();

// Add items to collection
users.add([{
  name: 'Soumendu De'
}, {
  name: 'Bikash Debroy'
}])

// Sync collection data to localstorage
users.each(function (user) {
  user.save();
}); 
```

在这里，我们通过将 `Backbone.LocalStorage` 的实例传递给 `LocalStorage` 属性来定义一个集合。这是将集合附加到 `LocalStorage` 并在其上执行所有数据操作所需的唯一配置。此外，此配置适用于模型和集合。如果你想了解此适配器中 `sync()` 功能是如何实现的，请查看适配器代码——它很小，而且写得相当不错。

另一个流行的 `LocalStorage` 适配器是 `Backbone.dualStorage` ([`github.com/nilbus/Backbone.dualStorage`](https://github.com/nilbus/Backbone.dualStorage))。网上有相当多的适配器可供选择，并且它们都提供了类似的功能。因此，如果你想跟随我们未提及的工具，你完全自由地这样做。

# 摘要

JavaScript 中的事件是其中最有趣的概念之一；关于这个主题有很多文章和书籍。我们并没有在本章中尝试查看所有细节，但我们分析了在 Backbone 应用程序中使用自定义事件和事件分发器如何为应用架构提供巨大的灵活性和可扩展性。我们鼓励你探索 JavaScript 事件、函数作用域和事件分发器或 PubSub 模式，如果你需要更详细的想法的话。

在本章中，我们也学习了 Backbone 的 `sync()` 方法以及我们如何覆盖 `sync()` 方法以获取用于公共 API 或 HTML5 `LocalStorage` 的自定义数据操作。

此外，我们还研究了 Backbone 的各种组件，讨论了它们的最佳实践，以及与它们相关的几个插件和扩展，以及一些常见问题。在下一章中，我们将看到如何使用不同的设计模式和架构来组织 Backbone 应用程序。
