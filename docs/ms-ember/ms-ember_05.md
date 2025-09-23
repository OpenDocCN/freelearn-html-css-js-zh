# 第五章. 控制器

在上一章中，我们讨论了 Ember.js 中模板如何用于向用户展示数据。我们还介绍了如何通过这些模板在我们的应用程序中轻松实现用户交互。我们指出，模板通过与其控制器通信来发挥作用。本章将对此进行深入探讨，并涵盖以下主题：

+   定义控制器

+   在控制器中存储模型和对象

+   使用对象和数组控制器

+   指定控制器依赖项

+   在控制器中注册动作处理器

+   控制器中的状态转换

# 定义控制器

就像路由处理器一样，可以通过扩展`Ember.Controller`类来定义控制器，如下面的代码行所示：

```js
AppNamespace.ControllernameController = Ember.Controller.extend();
```

可以进一步扩展已定义的控制器以创建另一个新的控制器类：

```js
App.TweetsController = Ember.Controller.extend();
App.RetweetsController = App.TweetsController.extend();
```

这些控制器类可以通过`create`方法进行实例化，如下面的示例所示：

```js
var controller = Ember.Controller.create();
var tweetsController = App.TweetsController.create();
```

就像对象一样，如果我们需要在实例化控制器时使用混入，我们需要使用`createWithMixins`方法：

```js
var mixin = Ember.Mixin.create({
  model: [1, 2, 3]
});
var controller = Ember.Controller.createWithMixins(mixin);
```

这相当于以下内容：

```js
var Controller = Ember.Controller.extend({
  model: [1, 2, 3]
});
var controller  Controller.create();
```

我们很少自己实例化应用程序控制器，因为当需要时，Ember.js 会为我们做这件事。

# 向控制器提供模型

在我们继续之前，让我们回顾一下数据在控制器中的加载和存储方式。我们将构建的大多数应用程序都将与 REST 端点进行通信，因此，Ember.js 自带了一些使创建此类应用程序变得简单的功能。在第三章中，我们学习了数据可以通过路由处理器的`model`钩子以异步方式从服务器加载。例如，让我们定义一个博客文章路由，从我们的服务器加载特定的博客文章。首先，我们将定义我们的应用程序的路由如下：

```js
App.Router.map(function(){
  this.resource('posts', function(){
  });
  this.resource('post', {path: '/post/:post_id'});
});
```

我们刚刚定义了一个将处理对文章详情页请求的资源，如下面的代码行所示：

```js
this.resource('post', {path: '/post/:post_id'});
```

如果用户访问文章的路径，比如`/post/100`，Ember.js 期望文章路由处理器将定义一个`model`钩子，该钩子将从服务器加载匹配的文章。以下是一个使用 jQuery 说明此点的示例：

```js
App.PostRoute = Ember.Route.extend({
  model: function(params){
    // load matching post from server
    return Ember.$.getJSON('/posts/'+params.post_id);
  }
});
```

在前面的示例中，处理器的`model`钩子接受一个包含文章 ID 的`options`对象。然后，使用 jQuery 的`getJSON()`方法使用此 ID 从服务器加载匹配的文章，该方法返回一个承诺，我们的应用程序在加载时解决。一旦解决，Ember.js 期望此路由定义一个`setupController`钩子，将解决的文章存储到相应的控制器中。这是作为以下代码实现的默认行为：

```js
App.PostRoute = Ember.Route.extend({
  model: function(params){
    return Ember.$.getJSON('/posts/'+params.post_id);
  },
  setupController: function(controller, model){
    controller.set('model', model);
  }
});
```

`setupController` 钩子接收两个参数：相应控制器的实例和解析的模型。此模型存储在控制器的 `model` 属性中。请注意，这只是默认实现；我们可以将模型存储在任何其他所需的属性或控制器中。

# 从控制器渲染动态数据

在从服务器加载数据后，控制器的作用是使这个模型可用于相应的模板进行显示。然后这些模板将注册对提供模型属性的绑定，并使用表单控件发送对这些属性所做的更改的更新。由于控制器是 `Ember.Object` 的扩展，它们通过以下方式实现了对浏览器环境事件特性的更好管理：

+   属性

+   计算属性

+   可观察者

## 属性

模板可以使用表达式显示绑定控制器的属性。例如，在先前的示例中，帖子模板将显示已加载的帖子如下：

```js
  {{! post.hbs}}

  <h1>{{model.title}}</h1>
  <p>{{model.body}}</p>
```

当帖子加载时，渲染的帖子模板将类似于以下内容：

```js
<h1>Introduction to Ember.js.</h1>
<p>A gentle introduction to Ember.js.</p>
```

如果帖子标题在以后某个时间点发生变化，模板的标题部分将被重新渲染以显示新的标题，如下所示：

```js
  controller.set(model.title', 'A guide to using Ember.js.');
```

模板还可以将更新推回控制器。这通常是通过 HTML 表单元素完成的。Ember 提供了 Handlebars 表达式，用于抽象这些控件的使用以创建双向绑定，正如我们在上一章中讨论的那样。为了说明这一点，让我们在我们的博客应用程序中添加一个新的路由，这将允许博客的管理员添加一个新的帖子条目，如下面的代码所示：

```js
  this.resource('posts', function(){
    this.route('/new'); // route to add a new post

});
```

管理员当然会在 `/posts/new` 页面上创建新的帖子。Ember.js 会期望为这个路由提供一个 `posts/new` 模板，其代码如下：

```js
{{! posts/new template }}

<form>
  {{input name='title' value=model.title}}
  {{textarea name='body' value=model.body}}
  <button type='submit'>Save post</button>
</form>
```

`PostsNewRoute` 处理器还需要为将作为其上下文的模板提供模型。正如你可能已经猜到的，它的 `model` 钩子将返回一个新帖子对象以供更新，如下面的示例所示：

```js
App.PostsNewRoute = Ember.Route.extend({
  model: function(params){
    return Ember.Object.create();
  }
});
```

`model` 钩子返回一个 Ember.js 对象，该对象将作为管理员要创建的帖子。由于这是常见做法，我们将在后面的章节中学习如何使用 `ember-data`，这是一个高级库，有助于定义和创建此类模型。由于双向绑定，任何对文本控件的更新都将更新新帖子。

## 计算属性

计算属性是返回属性并依赖于其他属性的函数。我们可以使用计算属性创建依赖于其他属性的 states 和 properties。这在我们需要通过聚合或任何形式的 map reduce 获取状态时特别有用。以下是一个用例示例：

```js
{{! application template }}
{{input name="firstName" value=model.firstName}
}
{{input name="lastName" value=model.lastName}}
Full name: {{fullName}}

// application controller

App.ApplicationController = Em.Controller.extend({
  fullName: function(){
    return this.get('model.firstName') + ' ' + this.get('model.lastName');
  }.property('model.firstName', 'model.lastName')
});

// application route
App.ApplicationRoute = Em.Route.extend({
  setupController: function(controller){
    controller.set('model', Em.Object.create({
      firstName: 'Jon',
      lastName: 'Doe',
    });
  });
     });
```

在这个例子中，模板能够根据用户的第一个和最后一个名字显示用户的 `fullName`。我们之前已经提到，仅使用模板层是无法完成这种实现的，如下面的示例所示：

```js
  Full name: {{firstName+lastName}} // wrong
```

需要注意的一点是，除非对象实例被设计为静态的，否则它们永远不会在类定义上设置。例如，如果模板是一个更新模型值的表单，我们可能会倾向于提供控制器的默认模型为：

```js
App.ApplicationController = Em.Controller.extend({
  model: Em.Object.create({
    firstName: 'Jon',
    lastName: 'Doe',
  },

  fullName: function(){
    return this.get('firstName') + ' ' + this.get('lastName');
  }.property('firstName', 'lastName')

});
```

之前实现的代码可能导致更新没有像预期那样被隔离。以下是我们博客应用中可以添加的另一个搜索功能示例：

```js
{{! search template}}

{{input name="search" value=queryTerm}}
<ul>
  {{#each results}}
  <li>{{user}}: {{body}}<li>
  {{/each}}
</ul>

// search controller
App.SearchController = Em.Controller.extend({

  // require posts controller
  needs: ['posts'],

  // compute matching posts
  results: function(){

    var queryTerm = this.get('queryTerm');
    if (!queryTerm && queryTerm.trim() === '') return;

    return this
    .get('controllers.posts.model')
    .filter(function(){
      return post.match(queryTerm);
    });

  }.property('queryTerm')

});
```

此示例包含一些我们之前讨论过的功能。如果用户访问`say/search`搜索页面，他们将看到一个搜索输入框，该输入框会自动在输入时更新搜索控制器的`queryTerm`属性。正如你可能注意到的，这个控制器的`results`属性将重新计算，因为它依赖于控制器的`queryTerm`属性。

我们引入的一个功能是控制器能够引用其他控制器的能力。我们将在稍后的部分讨论这个问题，但重要的是要注意，我们能够通过过滤`posts`控制器的模型来生成结果。搜索模板会自动在用户输入时重新显示结果。这个示例演示了在单页应用中添加这种看似困难的功能是多么简单。以下是一些你可以尝试添加到应用中的其他功能：

+   每次从服务器加载新帖子时都会显示一个旋转器。现在是一个很好的时机来回顾我们在第三章中讨论的加载和错误操作钩子，*路由和状态管理*。

+   为每个加载的帖子定义一个`humanizedDate`计算属性。让这个属性以可读的格式返回帖子的日期，例如`Mon, 15th`。Moment.js ([`momentjs.com`](http://momentjs.com))可能会很有用。

## 可观察对象

除了计算属性之外，我们还学习了如何使用可观察对象。这些是响应其他属性更改的函数。例如，让我们使之前提到的搜索功能更加用户友好。大多数用户期望在停止输入查询词后的几秒钟内启动搜索请求。因此，我们需要一种方法来*防抖*这个搜索。Ember.js 提供了一个提供此功能的函数，可以引用为`Ember.run.debounce`。以下是一个可能的实现：

```js
App.SearchController = Em.Controller.extend({

  needs: ['posts'],

  queryTermDidChange: function(){
    Em.run.debounce(this, this.searchResults, 1000);
  }.observes('queryTerm'),
  searchResults: function(){
    var queryTerm = this.get('queryTerm');
    if (!queryTerm && queryTerm.trim() === '') return;
    var results = this
    .get('controllers.posts.model')
    .filter(function(){
      return post.match(queryTerm);
    });
    this.set('results', results);
  }
});
```

控制器定义了一个观察者`queryTermDidChange`，它在输入仅一秒钟后调用搜索函数。如图所示，`debounce`函数([`emberjs.com/api/classes/Ember.run.html#method_debounce`](http://emberjs.com/api/classes/Ember.run.html#method_debounce))接受三个参数——一个上下文、一个在指定上下文中调用的函数，以及在没有其他调用时等待调用的时长。

# 对象和数组控制器

Ember.js 附带以下控制器，旨在轻松表示对象和可枚举数据：

+   对象控制器

+   数组控制器

这些控制器在某种程度上与其他控制器不同，因为表示的数据通常被设置为控制器的`model`属性，例如：

```js
Ember.ObjectController.create({
  model: {
    name: 'Jon Doe'
    age: 23
  }
});

Ember.ArrayController.create({
  model: [1, 2]
});
```

## 一个对象控制器

对象控制器用于代理表示的对象的属性。这意味着如果访问控制器属性，Ember.js 将首先在控制器中查找该属性，然后是模型。例如，让我们创建一个帖子模型类：

```js
App.Post = Ember.Object.extend({
    title: null,
    body: null
});
```

然后，从这个模型创建一个新的帖子：

```js
var post = App.Post.create({
    title: 'JavaScript prototypes.',
    body: 'This post will discuss JavaScript prototypes.'
});
```

可以显然使用对象的 getter 和 setter 方法访问此帖子的属性：

```js
post.set('title', 'Design patterns.');
post.get('title'); // Design patterns.
```

当此帖子被设置为`ObjectController`实例的模型时，如图所示，访问控制器属性将转换为访问帖子，例如：

```js
var postController = Ember.ObjectController.create();
postController.set('model', post);
postController.get('title'); // Design patterns.
```

注意，使用普通控制器不会产生相同的结果：

```js
var postController = Ember.Controller.create();
postController.set('model', post);
postController.get('title'); //  undefined
  postController.get('model.title'); // Design patterns.
```

当我们希望在控制器上创建依赖于模型属性的计算属性时，对象属性非常有用。例如，让我们将前面定义的帖子作为路由的模型传递：

```js
App.PostRoute = Ember.Route.extend({
  model: function(){
    return App.Post.create({
      title: 'JavaScript prototypes.',
      body: 'This post will discuss JavaScript prototypes.',
      tagIds: [1, 2, 3, 4]
    });
  }
});
```

现在，假设我们想要根据给定的 ID 数组计算一个`tags`属性，我们将在相应的控制器中实现它，如下所示：

```js
App.PostController = Ember.ObjectController.extend({

  TAGS: {
     1: 'ember.js',
     2: 'javascript',
     3: 'web',
     4: 'mvc'
  },

  tags: function(){
    var tags = this.get('TAGS');
    return this.get('tagIds').map(function(id){
    return tags[id];
    });
  }.property('TAGS', 'tags.length')

});
```

定义了计算属性后，我们可以在帖子模板中使用它，如下所示：

```js
<p>{{title}}</p>
<p>{{body}}</p>
<ul>
     {{#each tags}}
     <li>{{this}}</li>
    {{/each}}
</ul>
```

这将产生如下所示的结果：

```js
<p>JavaScript prototypes.</p>
<p>This post will discuss JavaScript prototypes.</p>
<ul>
  <li>ember.js</li>
  <li>javascript</li>
  <li>web</li>
  <li>mvc</li>
</ul>
```

注意，我们不需要像前面章节中那样在变量前加上`model.`前缀，因为模板的上下文、控制器将这些请求转发给了模型。这种类型的控制器使用`Ember.ObjectProxy`([`emberjs.com/api/classes/Ember.ObjectProxy.html`](http://emberjs.com/api/classes/Ember.ObjectProxy.html))混合，这使得代理（在这种情况下，控制器）可以将所有请求转发到它未定义的属性以及其模型，正如我们之前讨论的那样。

## 一个数组控制器

同样，数组控制器用于表示可枚举数据。可枚举数据的一个例子是 JavaScript 的`Array`原始数据类型：

```js
var controller = Ember.ArrayController.create({
  model: [1, 2, 3]
});
controller.get('length'); // 3
```

在这种情况下，相应的模板将列出项目，如下所示：

```js
{{#each}}
   {{this}}
{{/each}}
```

注意，我们也不需要像以下情况那样引用模型：

```js
{{#each model}}
{{this}}
{{/each}}

{{#each controller.model}}
{{this}}
{{/each}}
```

由于数组控制器表示可枚举数据，它们提供了以下有用的方法，可以用来操作它们的模型。

## addObject(object)

`addObject(object)`方法如果控制器模型不包含该对象，则将给定的对象添加到控制器模型的末尾，如下例所示：

```js
var controller = Ember.ArrayController.create({
  model: []
});
controller.addObject('a'); // ['a']
controller.addObject('b'); // ['a', 'b']
controller.addObject('b'); // ['a', 'b'] // already added
```

如果模型已经包含该对象，此方法调用将静默失败。

## pushObject(object)

`pushObject(object)`方法始终添加对象，无论模型是否包含它，例如：

```js
var controller = Ember.ArrayController.create({
  model: []
});
controller.pushObject('a');  // ['a']
controller.pushObject('b'); // ['a', 'b']
controller.pushObject('b'); // ['a', 'b', 'b'] // still added
```

## removeObject(object)

`removeObject(object)`方法用于从控制器模型中移除指定的对象，如下例所示：

```js
var controller = Ember.ArrayController.create({
  model: ['a', 'b', 'c']
});
controller.removeObject('a');    // ['b', 'c']
controller.removeObject('b');    // ['c']
controller.removeObject('b');    // ['c'] // fails silently
```

如果模型不包含该对象，此方法将不执行任何操作。

## addObjects(objects), pushObjects(objects), 和 removeObjects(objects)

之前提到的三种方法用于使用多个对象执行我们刚才讨论的三个方法，例如：

```js
var controller = Ember.ArrayController.create({
  model: []
});
controller.pushObjects(['a', 'b']);       // ['a', 'b']
controller.addObjects(['b', 'c']);        // ['a', 'b', 'c']
controller.removeObjects(['b', 'c']); // ['a']
```

## contains(object)

要检查模型是否包含一个对象，我们可以使用返回布尔值的 `contains(object)` 方法：

```js
var controller = Ember.ArrayController.create({
  model: ['a', 'c']
});
controller.contains('a');    // true
controller.contains('b');    // false
```

## compact()

`compact()` 方法返回一个底层模型的副本，其中已删除 undefined 和 null 项：

```js
var controller = Ember.ArrayController.create({
  model: ['a', 'b', 'c', undefined, null]
});
controller.compact();    // ['a', 'b', 'c']
```

## every(callback)

`every(callback)` 方法用于检查模型中包含的每个项是否满足给定的条件：

```js
var areEven = [2, 2, 4, 24, 80].every(condition); // true
var areEven = [1, 2, 3, 4, 5].every(condition);     // false

function condition(integer){
  return integer %2 === 0;
}
```

## filter(object)

过滤器的工作方式与原生的 JavaScript 数组对象 `Array.filter` 相同：

```js
[2, 2, 4, 24, 80].filter(condition); // [2, 2, 4, 24, 80]
[1, 2, 3, 4, 5].filter(condition);     // [2, 4]

function condition(integer){
  return integer %2 === 0;
}
```

## filterBy(property)

有时候，我们想要压缩一个模型，但只有当包含的项定义了给定的属性时才这样做。我们可以使用前面的 `filter` 方法，如下所示：

```js
var colors = [
  { name: 'red', isPrimary: true },
  { name: 'green', isPrimary: false },
  { name: 'black', isPrimary: undefined },
  { name: 'white', isPrimary: null },
];
colors.filter(condition);     // [{ name: 'red', isPrimary: true }]
function condition(color){
  return !!color.isPrimary;
}
```

我们也可以使用更简短的形式，如下所示：

```js
var colors = [
  { name: 'red', isPrimary: true },
  { name: 'green', isPrimary: false },
  { name: 'black', isPrimary: undefined },
  { name: 'white', isPrimary: null },
];
colors.filterBy('isPrimary');     // [{ name: 'red', isPrimary: true }]
```

## find(callback)

在前面的例子中，我们可以使用 `filter` 方法来返回第一个主要颜色的出现，如下所示：

```js
var colors = [
  { name: 'red', isPrimary: true },
  { name: 'green', isPrimary: false },
  { name: 'black', isPrimary: undefined }
];
colors.filter(condition)[0];     // { name: 'red', isPrimary: true }
function condition(color){
  return !!color.isPrimary;
}
```

这是不高效的，因为我们总是遍历所有模型项。可以使用 `find` 方法来实现这个需求，如下所示：

```js
var colors = [
  { name: 'red', isPrimary: true },
  { name: 'green', isPrimary: false },
  { name: 'black', isPrimary: undefined }
];
colors.find(condition);     // { name: 'red', isPrimary: true }
function condition(color){
  return !!color.isPrimary;
}
```

一旦找到匹配项，检查迭代就会终止。

## findBy(key, value)

正如在 `filter` 与 `filterBy` 的情况下，我们可以使用 `findBy` 方法而不是 `find` 来重新实现前面的例子，如下所示：

```js
var colors = [
  { name: 'red', isPrimary: true },
  { name: 'green', isPrimary: false },
  { name: 'black', isPrimary: undefined }
];
colors.findBy('isPrimary', true);     // { name: 'red', isPrimary: true }
```

## insertAt(index, object), objectAt(index), 和 removeAt(index, length)

`insertAt(index, object)`、`objectAt(index)` 和 `removeAt(index, length)` 方法用于使用项索引执行操作。第一个方法用于在指定的索引处添加一个对象。如果索引超出范围，则抛出错误。第二个方法用于检索指定索引处的对象。如果索引超出范围，则返回 undefined 值。

注意，我们无法使用负索引进行查找，如下例所示：

```js
var colors = ['red', 'blue'];
colors.insertAt(1, 'yellow'); // ['red',  'yellow', 'blue'];
colors.insertAt(10, 'green'); // Error: Index out of range
colors.objectAt(0); // 'red'
colors.objectAt(10); // undefined
colors.objectAt(-1); // undefined - negative index
```

最后一种方法通过可选的范围删除与给定索引匹配的对象：

```js
var fruits = ['mango', 'apple, 'banana', 'orange', 'papaya', 'lemon'];
fruits.removeAt(0); // [apple, 'banana', 'orange', 'papaya', 'lemon'];
fruits.removeAt(1, 2); [apple, 'papaya', 'lemon'];
fruits.removeAt (10, 3); // Error: Index out of range
```

## map(callback)

映射与 `Array.map` 的工作方式相同：

```js
var fruits = ['mango', 'apple', 'banana', 'orange', 'papaya', 'lemon'];
fruits.map(function(fruit){
  return {name: fruit};
});
// [
//  { name: 'mango'},
//  { name: 'apple'},
//  { name: 'banana'},
//  { name: 'orange'},
//  { name: 'papaya'},
//  { name: 'lemon'}
// ];
```

## mapBy(property)

使用前面例子中生成的结果，我们可以使用 `mapBy` 方法来获取原始数组，如下所示：

```js
var fruits = [
 { name: 'mango'},
 { name: 'apple'},
 { name: 'banana'},
 { name: 'orange'},
 { name: 'papaya'},
 { name: 'lemon'}
];
fruits.mapBy('name');
// ['mango', 'apple', 'banana', 'orange', 'papaya', 'lemon'];
```

如上图所示，此方法返回一个新数组，包含在模型项上评估的值。

## forEach(function)

这是一个常用的方法，它对模型中包含的每个项调用给定的函数：

```js
var Dog = Em.Object.extend({
  bark: function(){
    console.log('woof: %s', this.get('name'));
  }
});
// model
var dogs = ['bo', 'sunny'].map(function(name){
  return Dog.create({
    name: name
  });
});
dogs.forEach(function(dog){
  dog.bark();
});
// woof: bo
// woof: sunny
```

## uniq()

如其名所示，`uniq()` 方法返回一个不包含重复项的新数组：

```js
['papaya', 'apple', 'banana', 'orange', 'papaya', 'apple'].uniq();
// ["papaya", "apple", "banana", "orange"]
```

## sortProperties 和 sortAscending

`sortProperties` 和 `sortAscending` 方法用于对表示的数据进行排序。例如，我们可能有一个音乐目录，我们希望按专辑名称排序，然后按歌曲名称排序，如下表所示：

| 专辑名称 | 歌曲名称 |
| --- | --- |
| 双重幻想 | Tiffany Blews |
| 双重幻想 | W.A.M.S |
| 高地上的无限 | 惊悚 |

要完成这个任务，我们需要在控制器中定义以下属性：

+   `sortProperties`

+   `sortAscending`

第一个属性指定了排序项时使用的属性，而第二个属性指定了排序方向。在我们的例子中，我们将按以下方式对音乐目录进行排序：

```js
var controller = Ember.ArrayControler.create({
  model: [
   {name: 'W.A.M.S', album: 'Folie a deux'},    {name: 'Thriller', album: 'Infinity on high'},   {name: 'Tiffany blues', album: 'Folie a deux'},
  ],
  sortProperties: ['name', 'album'],
  sortAscending: true
});
```

要按相反顺序对歌曲进行排序，我们需要将`sortAscending`属性设置为`False`。

这些只是`Ember.ArrayProxy`([`emberjs.com/api/classes/Ember.ArrayProxy.html`](http://emberjs.com/api/classes/Ember.ArrayProxy.html))混合提供的一些常见方法，而`Ember.ArrayController`正是使用了这些方法。

# 处理事件动作

在上一章中，我们学习了如何将用户动作从模板轻松委托给控制器和路由。让我们通过一个例子来回顾一下：

```js
{{! posts/new template }}

<form {{action 'save' model on='submit'}}>
  {{input name='title' value=title}}
  {{textarea name='body' value=body}}
  <button type='submit'>Create</button>
  <button type='cancel' {{action 'cancel' this}}>Cancel</button>
</form>
```

在这个例子中，我们定义了两个将由相应控制器处理的动作，如下所示：

```js
App.PostsNewController = Ember.ObjectController.extend({
  actions: {
    save: function(post){
      post.save();
    },
    cancel: function(post){
      post.rollback();
    }
  }
});
```

我们已经了解到，所有的动作处理器都定义在目标控制器或路由的`actions`属性中。在这种情况下，当用户通过点击**提交**按钮或按*Enter*键提交表单时，控制器中的`save`钩子会以 post 上下文作为唯一参数被调用。同样，点击**取消**按钮会调用相应的`cancel`钩子。

一个典型的由标签组成的部件可以以相同的方式实现，如下面的截图所示：

![处理事件动作](img/00009.jpeg)

在这种情况下，小部件模板可能看起来像以下代码：

```js
<ul class="tabs">
  {{#each}}
  <li    {{bind-attr selected="selected"}} 
    {{action "selected" this}}>
      {{name}}
  </li>
  {{/each}}
<ul>
```

此模板包含由`tabs`上下文属性生成的`tabs`元素组。如果点击了任何标签，它们将需要获取一个`selected`类。以下适合的样式将实现此效果：

```js
.tabs .selected{
  color: deepskyblue
}
```

控件的`selected`行为处理器将实现如下：

```js
App.TabsRoute = Ember.Route.extend({

  model: function(){

    return [{

      name: 'tab 1',

      body: 'tab 1 content'

    }, {

      name: 'tab 2',

      body: 'tab 2 content'

    }];
  }

});

App.TabsController = Ember.ArrayController.extend({

  actions: {

    selected: function(selectedTab){

      this.forEach(function(tab){

        var selected = tab.get('name') === selectedTab.get('name');

        tab.set('selected', selected);

      });

    }

  }

});
```

注意，这些动作不需要像我们在上一章中讨论的那样在上下文控制器中捕获。当一个动作被触发时，通常是从模板元素触发的，Ember.js 会检查是否在直接控制器的`actions`属性中定义了适当的行为处理器。如果不是这种情况，Ember.js 将继续在相应的路由中搜索行为处理器。需要注意的是，如果一个行为处理器返回`True`，Ember.js 仍然会继续搜索这个处理器，这构成了**动作冒泡**。

# 指定控制器依赖项

控制器依赖项使控制器能够关联。因此，每当一个控制器需要访问另一个控制器的属性时，它应该首先将控制器声明为依赖项，以便能够这样做。这些依赖项在受影响控制器的`needs`属性中定义。例如，假设我们决定向我们的博客应用程序添加一个评论系统：

```js
  this.resource('post', {path: '/post/:post_id'}, function(){
    this.resource('comments', function(){
  });    });
```

在一个典型的博客中，评论通常显示在单独的页面上，在我们的案例中，是在路径如 `/post/100/comments` 的页面上。我们需要定义一个 `comments` 模板，该模板列出已加载的评论如下：

```js
{{! comments template}}
<h1> Comments for <h1>
<ul>
  {{#each comments}}
  <li>{{user}}: {{body}}</li>
  {{/each}}
</ul>
```

如您可能已经注意到的，模板需要显示评论帖子的标题。为此，它需要能够访问 `post` 控制器中加载的帖子。通过指定对 `post` 控制器的依赖项，评论控制器将能够在其 `controllers` 对象属性中访问 `post` 控制器。例如：

```js
App.CommentsController = Ember.Controller.extend({
  needs: ['post']
});
```

然后，`comments` 模板将更新为以下内容：

```js
{{! comments template}}
<h1>Comment listing for {{controller.controllers.post.title}}</h1>
<ul>
     {{#each comments}}
  <li>{{user}}: {{body}}</li>
  {{/each}}
</ul>
```

您可能想知道这是否会引发无限依赖循环。好吧，控制器可以相互依赖而不会遭受这种命运：

```js
App.AController = Ember.Controller.extend({
  needs: ['b']
}); 
App.BController = Ember.Controller.extend({
  needs: ['a']
}); 
```

这种关联充当了应用程序不同组件之间正确沟通的渠道。

# 控制器中的状态转换

在 第三章，*路由和状态管理* 中，我们了解到路由可以通过调用它们的 `transitionTo` 方法将应用程序的状态转换到其他路由，如下面的代码所示：

```js
App.IndexRoute = Ember.Route.extend({
  redirect: function(){
    this.transintionTo('posts');
  }
});
```

同样，控制器也通过使用提供的 `transitionToRoute` 方法具有这种能力。例如，我们可以在控制器的事件处理程序中更改状态，如下所示：

```js
App.PostsNewController = Ember.ObjectController.extend({
  actions: {
    cancel: function(post){
      this.transitionToRoute('posts');
    }
  }
});
```

# 摘要

这是一章令人兴奋的章节，帮助我们理解控制器的主要目的，即数据表示。我们学习了控制器是如何根据定义的应用程序路由来定义的。我们还学习了如何使用对象和数组控制器来表示模型。最后，我们学习了如何设置控制器之间的依赖关系，这些控制器可能处理应用程序的不同关注点。在本书的这个阶段，我们真的应该开始思考构建 Ember.js 应用程序的方法。下一章将涵盖视图层，这需要大量关于控制器使用的知识。
