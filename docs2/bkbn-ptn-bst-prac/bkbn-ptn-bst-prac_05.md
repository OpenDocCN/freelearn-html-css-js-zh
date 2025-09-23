# 第五章. 路由最佳实践和子路由

路由器是 Backbone 中最有用的对象之一；它主要用于使用哈希片段或标准 URL 路由应用程序 URL。在 Backbone 的早期版本中，`Backbone.Controller`被用来处理路由以及默认控制器任务，而不是`Backbone.Router`。后来，它被改为`Backbone.Router`，因为路由器旨在仅处理路由客户端页面并将它们通过 URL 连接到事件和动作，而功能逻辑必须由演示者（即 Backbone 视图）负责。路由器的概念相当简单——它将方法名称与 URL 片段匹配并调用该方法。然后，该方法负责所需的事件和动作。

在本章中，我们将学习一些最佳实践，例如如何为中型和大型应用程序组织路由器，以及哪些类型的任务应由路由器处理。主要涵盖的主题如下：

+   **与路由器一起工作**：这提供了一个基本示例，说明了路由器的工作原理，以及在使用路由器时可能出现的错误分析。

+   **与路由器一起工作的最佳实践**：我们将探讨一些在使用路由器时应遵循的良好实践。

+   **子路由——组织复杂应用程序的关键**：一旦应用程序增长，维护单个路由器变成了一项艰巨的任务。将应用程序路由器划分为多个子路由是一种更可取的管理方式。

# 与路由器一起工作

Backbone 路由器提供方法，通过使用哈希片段或标准 URL（根据历史 API）来路由客户端页面。使用哈希片段或历史 API 的路由可能看起来像这样：

```js
// Hash fragment
http://www.foo.com/#user/23

// Standard URL
http://www.foo.com/user/23
```

当 URL 片段与特定路由匹配时将触发路由和动作，这些路由和动作在路由器的`routes`对象中定义：

```js
routes: {
  'users' : 'showUsers',
  'user/:id' : 'showUserDetails',
  'user/:id/update' : 'updateUser'
}
```

现在，让我们看看如何创建一个基本的路由器。假设我们正在开发一个包含几个模块的应用程序，其中`User`模块是一个重要的模块。

```js
var AppRouter = Backbone.Router.extend({
  routes: {
    'users': 'showUsers',
    'user/:id': 'showUserDetails',
    'user/:id/update': 'updateUser',
    'user/:id/remove': 'removeUser'
  },

  showUsers: function () {
    // Get all the user details from server and 
    // show the users view 
  },

  showUserDetails: function (userId) {
    // Get the user details for the user id as received
  },

  updateUser: function (userId) {},
  removeUser: function (userId) {}
});
```

非常简单！有许多选项可以修改路由以获得预期的结果。例如，您可以在路由中使用“splat”或可选部分；有关路由的详细概述，请参阅 Backbone.js API [`backbonejs.org/#Router`](http://backbonejs.org/#Router)。

在前面的示例中，所有的显示方法（`showUsers()`和`showUserDetails()`）很可能会创建视图实例，向服务器发送 AJAX 请求以获取其详细信息，然后在 DOM 中显示视图。同样，更新和删除方法也会向服务器发送请求以执行所需操作。现在，假设`User`模块在这个路由器中还有其他多个方法，除了这些 CRUD 方法。此外，整个应用程序还有许多类似的模块，其路由也将添加到这个路由器中。结果，路由器很快就会变得庞大，有数百行代码；这是超出我们控制范围的事情。

在与路由器一起工作时，我们应该注意一些事项，以避免出现此类情况。在下一节中，我们将探讨一些良好的实践，这些实践将使我们的路由器变得简单、灵活且易于维护。

# 与路由器一起工作的最佳实践

当应用程序规模较小时，使用骨干路由器相对容易。然而，随着复杂性的增加，除非遵循某些规则，否则维护路由器会变得困难。在下一节中，我们将讨论在使用路由器时应注意的一些要点。

## 避免在路由方法中使用大量功能代码

尽管路由器的基本任务是监控路由并触发函数，但它也管理应用程序的一些业务逻辑。在 MVC 架构中，控制器的任务是处理客户端发送的数据请求，并处理从服务器响应中来的数据。同样，对于路由器，由于 URL 片段反映了应用程序数据的一部分，数据通信、调用视图方法或更新模型属性都是通过路由器方法完成的。

我在初级开发者代码中经常看到的一个趋势是，他们经常在路由器方法中包含大量功能代码。一方面，这增加了路由器的大小，另一方面，它也复杂化了逻辑。始终建议尽可能保持路由器方法尽可能短，通过将功能逻辑推送到视图并使用事件而不是回调来实现。在下一节中，我们将看到如何保持我们的路由器整洁。此外，我们将在第六章“与事件、存储和同步一起工作”中更详细地探讨事件委托、自定义事件和回调方法。

## 在路由器方法中实例化视图

我见过许多开发者将应用程序视图实例化在路由器方法中。虽然在这种情况下在路由器方法中实例化视图或修改 DOM 元素没有限制，但避免在路由器中执行此类操作是一种良好的实践。这与我在本节中提到的第一个要点有些相关。在下面的代码中，我们实例化了一个`UserView`类，并在路由器方法中将其渲染到 DOM 中：

```js
Backbone.Router.extend({
  routes: {
    "users": "showUsers"
  },

  showUsers: function () {
    var usersView = new UsersView();
    usersView.render();
    $("#user_list").html(usersView.el);
  }
});
```

这看起来很简单，并且工作得完美。但是，如果有 20 个或更多这样的方法，它不会使路由器变得杂乱吗？为什么不创建一个控制器或高级应用程序对象，并将方法添加到那里？然后你可以从路由器方法中调用这个控制器方法，如下所示：

```js
Backbone.Router.extend({
  routes: {
    "users": "showUsers"
  },

  showUsers: function() {
    UserController.showUsers();
  }
});

var UserController = {
  showUsers: function() {
    var usersView = new UsersView();
    usersView.render();
    $("#user_list").html(usersView.el);
  }
}
```

现在对`showUsers()`方法功能性的任何更改都不会强迫你接触路由器。实际上，并没有太大的可见差异——但就我个人而言，因为我已经多次使用第二种模式并从中受益，我可以向你保证，关注点的分离将产生一个更干净的路由器，以及可维护的代码库。

此外，在此上下文中，我们建议你查看`Marionette.AppRouter` ([`github.com/marionettejs/backbone.marionette/blob/master/docs/marionette.approuter.md`](https://github.com/marionettejs/backbone.marionette/blob/master/docs/marionette.approuter.md)) 和 `Marionette.Controller` ([`github.com/marionettejs/backbone.marionette/blob/master/docs/marionette.controller.md`](https://github.com/marionettejs/backbone.marionette/blob/master/docs/marionette.controller.md))。Marionette 的`AppRouter`和`Controller`与我们的`UserController`和基础路由器工作方式相同。控制器实际上执行工作（例如组装数据、实例化视图、在区域中显示它们）并可以更新 URL 以反映应用程序的状态（例如，显示的内容）。路由器只是根据在地址栏中输入的 URL 触发控制器动作。这两个类都非常有用，如果需要，你可以选择其中的任何一个。

## 使用正则表达式进行选择性路由

如果你只想在特定条件匹配时触发路由器方法，正则表达式就能派上用场。正则表达式在路由方面非常灵活，Backbone 完全支持它们。实际上，当它们被添加到路由表时，所有的路由首先都会被转换为`RegExp`对象。

然而，与其它字符串 URL 片段不同，JavaScript 不允许你将正则表达式作为`routes`对象的属性添加：

```js
// This will give error
routes : {
  /^user\/(\d+)/ : 'showUserDetails' 
}
```

解决方案是在路由器的`initialize()`方法中的`routes`对象中添加正则表达式：

```js
initialize: function () {
  this.route(/^user\/(\d+)/, 'showUserDetails');
},

showUserDetails: function (id) {
  console.log(id);
}
```

这是一个例子，其中在`#user`之后的 URL 片段中只允许使用数字。如果你尝试打开以`#user/abc`结尾的 URL，`showUserDetails()`方法将不会被调用，但使用 URL 片段`#user/123`时，将触发相同的方法。因此，这是一个使用正则表达式作为路由的非常通用的例子。正则表达式对于更复杂的 URL 片段级别非常有用，以便提供一种限制级别。

# 子路由——组织复杂应用程序的关键

子路由是将应用程序的路由器划分为多个特定模块路由器的想法。在小型或中型应用程序中，你可能永远不需要这样的东西。然而，对于具有多个模块的应用程序，处理所有路由的单个路由器很快就会变成一个庞大且难以管理的类。因此，总是将主路由器拆分为一组特定模块的路由器更为可取。

`Backbone.Subroute` ([`github.com/ModelN/backbone.subroute`](https://github.com/ModelN/backbone.subroute))，由 Dave Cadwallader 开发的一个出色的扩展，提供了我们所讨论的功能。它允许基础路由器将所有特定模块的路由委托给与该模块关联的子路由器。让我们通过以下两个示例来了解路由器和子路由器之间的区别。

## 一体化路由器

以下是为处理应用程序中所有模块路由的单个路由器编写的代码：

```js
var App = {};

var App.BaseRouter = Backbone.Router.extend({
  routes: {
    // Generic routes
    '': 'showHome',
    'logout': 'doLogout',

    // User specific routes
    'users/view/:id': 'showUserDetails',
    'users/search': 'searchUsers',

    // Company specific routes
    'company/:id': 'showCompanyDetails',
    'company/users': 'showCompanyDetails'
  },

  showHome: function () {},
  doLogout: function () {},

  showUserDetails: function () {},
  searchUsers: function () {},

  showCompanyDetails: function () {},
  showCompanyDetails: function () {},
});
```

现在，让我们使用 `Backbone.Subroute` 来看看如何定义基础路由器和特定模块路由器。使用 `Subroute`，基础路由器变成了一个微小的路由器，它只负责路由重定向。

## 基础路由器

以下是为基础路由器编写的代码：

```js
var App.BaseRouter = Backbone.Router.extend({
  routes: {
    // Generic routes
    '': 'showHome',
    'logout': 'doLogout',

    // Route all users related routes to users subroute
    'users/*subroute': 'redirectToUsersModule',

    // Route all company related routes to company subroute
    'company/*subroute': 'redirectToCompanyModule'
  },

  showHome: function () {},
  doLogout: function () {},

  redirectToUsersModule: function () {
    if (!App.usersRouter) {
      App.usersRouter = new App.UsersRouter('/users');
    }
  },

  redirectToCompanyModule: function () {
    if (!App.companyRouter) {
      App.companyRouter = new App.CompanyRouter('/company');
    }
  },
});
```

## Users 模块路由器

以下是为 `Users` 模块路由器编写的代码：

```js
var App.UsersRouter = Backbone.SubRoute.extend({
  routes: {
    '': 'showUsers',
    'view/:id': 'showUserDetails',
    'search': 'searchUsers'
  },

  showUsers: function () {},
  showUserDetails: function () {},
  searchUsers: function () {}
});
```

看看基础路由器。我们在这里做的事情非常简单——我们使用通配符（或 splat）来触发一个懒加载实例化子路由器并传递哈希的初始参数。现在，因为 `Backbone.Subroute` 扩展了 Backbone 的 `Router` 类，我们可以预期在 `/users/` 或 `/company/` 之后传递的任何内容都应该由相应的子路由器处理，即 `App.UsersRouter` 或 `App.CompanyRouter`。

我们可以添加任意多的子路由器，而基础路由器不会关心它们。同样，子路由器也不知道它们的前缀是什么。模块名称或前缀的任何更改都应该仅在基础路由器中进行，而无需修改相关的子路由器。

`Backbone.Subroute` 是一个小巧而优秀的插件，你应该始终将其包含在你的应用程序中，以保持你的基础路由器整洁。随着你的应用程序中模块的增加，你会越来越理解子路由器的优势。

# 摘要

Backbone 路由器的功能非常简单且易于学习。对于简单的应用程序，你将不会在维护它时遇到任何问题。一旦你的应用程序增长并且路由器变得庞大，问题就会逐渐出现。在本章中，我们讨论了路由器管理的最佳实践，那些你应该始终遵守的。我们还学习了子路由，它通过将主路由器拆分为多个特定模块的路由器并分配任务来提供帮助。

在下一章中，我们将讨论 Backbone 事件、自定义事件、存储和同步。
