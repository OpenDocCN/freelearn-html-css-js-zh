# 第六章. 使用路由器创建客户端页面

在本章中，我们将检查 Backbone 的 `Router` 类，以了解以下内容：

+   如何通过模拟浏览器历史记录来创建虚拟页面

+   如何创建新的 `Router` 子类和实例

+   定义路由模式的各种机制

+   如何处理特殊案例，例如不存在的路由

+   如何使用页面视图来一致地渲染路由

# Backbone 路由器使单页应用程序成为可能

到目前为止，我们查看的所有 Backbone 功能都是对先前存在功能的增强；Backbone 的类改进了原生 JavaScript 类——`Models` 和 `Collections`——以及增强对象和数组的功能，而 `Views` 则改进了 jQuery 已经提供的 DOM 操作和事件绑定。

然而，`Routers` 打破了这一趋势。`Routers` 不是增强现有功能，而是允许你做一件完全新的事情：仅使用一个 HTML 页面创建整个网站。如 第一章 中所述，*使用 Backbone 构建单页网站*，单页应用程序相对于传统多页网站具有几个优点：最显著的是性能增强和完全的客户端控制。Backbone 的 `Router` 类使得所有这些成为可能。

# 路由器的工作原理

在传统的多页网站上，浏览器在无需额外努力的情况下提供路由和页面历史记录。当用户访问网站上的 URL 时，浏览器会请求该 URL 的内容，然后当用户移动到另一个 URL 时，浏览器会跟踪用户浏览历史记录中的变化。然而，在由 Backbone 驱动的网站上，这两者都必须由 `Router` 来处理。当用户访问新页面时，`Router` 确定要渲染哪些 `Views`，当他们离开该页面时，`Router` 会通知浏览器新的历史记录条目。这使得由 Backbone 驱动的网站可以像传统网站一样运行，包括允许用户使用浏览器的 `Back` 按钮。

`Router` 可以通过两种不同的方法来实现这一点，Backbone 允许你决定使用哪种方法。第一种，也是默认的方法是基于哈希的路由，它利用了 URL 片段（也称为命名锚点）。你可能之前见过带有这些片段的 URL，例如 [www.example.com/foo#bar](http://www.example.com/foo#bar)。通过使用这些片段来定义网站的页面，Backbone 可以知道要加载哪个页面，并告诉浏览器如何跟踪用户在页面之间的移动。

如果你不想让你的用户在 URL 中看到哈希值，Backbone 提供了一个基于最近添加的 HTML5 pushState 技术的第二个选项。虽然这确实有助于使 URL 看起来更干净，但 pushState 方法存在一个显著的缺点。尽管 pushState 在大多数网络浏览器中都可用，但像 Internet Explorer 9 及以下版本这样的旧浏览器不支持它。如果你尝试在一个不支持 pushState 的浏览器中使用基于 pushState 的路由，Backbone 将回退到基于哈希的路由。即使只有 1%的用户使用旧浏览器，这也意味着这 1%的用户将看到与其它用户不同的 URL，这可能会在（例如）不同浏览器的用户分享链接时造成混淆。

基于 pushState 的路由也有一个小的缺点：如果一个用户在 pushState 驱动的网站上刷新页面，浏览器将请求从服务器获取该 URL。这可以通过简单地让服务器在浏览器请求此类 URL 时返回你的应用程序的单个 HTML 页面来解决。然而，鉴于这两个缺点，使用 pushState 仅在你非常关心 URL 的外观并且相信所有用户都将使用现代浏览器的情况下推荐。

# Backbone.history

由于可以同时使用多个`Routers`（尽管通常不推荐这样做，我们将在本章后面讨论），Backbone 有一个名为`Backbone.history`的独立全局对象，用于处理历史管理。然而，重要的是要理解，这个对象实际上并没有替换你的浏览器历史；相反，它只是帮助管理添加到历史记录的内容以及何时添加。

当你加载一个使用 Backbone `Router`的页面时，你需要调用这个历史对象的`start`方法，以便告诉 Backbone 开始路由。这个`start`方法还允许你告诉 Backbone 你需要使用哪种路由技术。如果你想 Backbone 依赖于基于哈希的路由，你可以简单地调用这个`this`方法而不带任何参数：

```js
Backbone.history.start();
```

如果你希望使用基于 pushState 的路由，你需要提供一个额外的参数来表示这一点：

```js
Backbone.history.start({pushState: true});
```

# 路由和页面的区别

路由和页面是非常相似的概念，并且经常被 Backbone 程序员互换使用。这是自然的，因为路由本质上只是 Backbone 对页面的实现。然而，两者之间至少在将多页网站的页面与 Backbone 单页应用的路由进行比较时，存在重要的区别。

在传统的多页网站上，用户访问的每个新页面都需要你向网站服务器发送一个新的请求。然而，在一个由 Backbone 驱动的网站上，用户可以导航到他们想要的任意多页（路由），只有在需要获取新数据时才会发送新的请求。仅此一项功能就使得 Backbone 驱动的网站速度显著提高。

另一个重要的区别是，标准网页会触发整个页面的加载，而路由只会触发特定的 JavaScript 函数。这意味着与传统应用不同，传统应用必须限制其创建的页面数量（以及因此的 HTTP 请求数量），路由可以用于几乎任何可能的状态变化。路由可以以类似传统网页的方式使用，或者用于对页面进行更小的更改。它们甚至（很少）可以在 DOM 没有任何变化的情况下使用。

当然，虽然 Backbone 路由有很多优点，但它们也有一些缺点。最重要的缺点是，由于它们不会刷新页面，现有的 DOM 元素、样式更改和事件绑定不会被自动清除。这意味着你需要自己处理这些清理任务。幸运的是，这并不难做，特别是如果你依赖于我们很快将要介绍的页面视图。

# 创建一个新的 Router

与所有 Backbone 类一样，你可以通过使用`extend`来创建一个新的`Router`子类，其中第一个参数提供了类的实例属性和方法，第二个参数提供了静态属性和方法：

```js
var MyRouter = Backbone.Router.extend({
    // instance methods/properties go here
}, {
    // static methods/properties go here
);
```

与`Views`类似，`Routers`在实例化时只接受一个`options`参数。这个参数完全是可选的，它唯一真正接受的选项是`routes`选项。正如之前提到的，一旦`Router`被创建，你需要在它能够处理路由之前运行`Backbone.history.start()`：

```js
myRouter = new Backbone.Router({
    routes: {
        'foo': function() {
            // logic for the "/foo" or "#foo" route would go here }
    }
});
Backbone.History.start(); // siteRouter won't work without this
```

# 创建路由

在创建`Router`时传递路由的问题在于，你必须提供所有路由函数以及你的路由，这（特别是如果你网站上有很多路由）很容易变得混乱。相反，许多 Backbone 程序员将他们的路由定义在`Router`类本身上，如下所示：

```js
var SiteRouter = Backbone.Router.extend({
    routes: {
        'foo': 'fooRoute'
    },
    fooRoute: function() {
        // logic for the "/foo" or "#foo" route would go here }
});
var siteRouter = new SiteRouter();
Backbone.History.start(); // siteRouter won't work without this
```

如你所见，这种方法简化了路由的定义，使它们更类似于`Models`和`Collections`的`events`属性。你不需要与路由本身一起定义路由方法，只需给路由指定一个路由方法的名字，Backbone 就会在`Router`类内部查找该方法。

还有一种第三种方法可以添加路由，那就是使用 Backbone 的`route`方法。这个方法接受`route`作为第一个参数，路由的名称作为第二个参数，路由的处理函数作为第三个参数。如果省略第三个参数，Backbone 将在`Router`上查找与`route`本身同名的方法：

```js
For example:SiteRouter = new Backbone.Router({
    initialize: function() {
        this.route('foo', 'fooRoute');
    },
    fooRoute: function() {
        // logic for the "/foo" or "#foo" route would go here}
});
```

正如`Models`和`Collections`的`on`方法允许你在创建类的实例之后创建事件绑定一样，`Routers`的`route`方法允许你在任何时间定义一个新的路由。这意味着你可以在创建`Router`类之后动态地创建所有路由。然而，正如你通常希望在用户首次访问你的网站时所有页面都可用一样，你通常也希望所有路由都可用，这意味着你通常会在你的`Routers initialize`方法中创建它们。

使用`route`方法的主要优势是你可以将逻辑应用到你的路由上。例如，假设你在网站上有多条仅针对管理员的路由。如果你的`t`可以检查用户`Model`的`isAdmin`属性，那么它可以使用这个属性在`Router`类初始化时动态地添加或删除这些仅限管理员的路由，如下所示：

```js
// NOTE: In a real case user data would come from the server
var user = new Backbone.Model({isAdmin: true});SiteRouter = new Backbone.Router({
     initialize: function(options) {
        if(user.get('isAdmin')) {
            this.addAdminRoutes();
        }
    },
    addAdminRoutes: function() {
        this.route('adminPage1', 'adminPage1Route');
        this.route('adminPage2', 'adminPage2Route');
        // etc.
    }
});
```

如前一个示例所示，使用`route`方法的另一个优势是你可以将路由分组组织成单独的函数。如果你想要一起开启或关闭某些路由组，这可能会很有用；但它也可以简单地用来组织常量路由。实际上，你甚至可以在一个完全不同的文件中定义一些路由，只要你的`Router`可以访问它们，如下所示：

```js
// File #1
window.addFooRoutes = function(router) {
    router.route('foo', function() {
        //...
    })
}

// File #2SiteRouter = new Backbone.Router({
    initialize: function(options) {
        if(options.includeFooRoutes) {
            addFooRoutes(this);
        }
    }
});
```

# 路由样式

除了有选择如何连接你的路由之外，你还可以选择如何定义路由。到目前为止，你看到的都是简单路由；它们定义了一串字符，当 Backbone 在 URL 中看到这个确切的字符串时，它会触发相应的路由方法。然而，有时你可能想要更灵活的路由。

例如，假设我们有一个销售书籍的网站，我们想要有一个“了解特定书籍”的路由。如果我们网站上有很多书籍，为每本书单独创建路由可能会变得痛苦：

```js
var SiteRouter = new Backbone.Router({
    initialize: function(options) {
        this.route('book/1', 'book1Route'); // for the book with ID 1
        this.route('book/2', 'book2Route'); // for the book with ID 2
        this.route('book/3', 'book3Route'); // for the book with ID 3
        // this will get old fast
    }
});
```

幸运的是，有两种替代方案可以解决这个问题：路由字符串和正则表达式路由。首先，让我们看看正则表达式路由如何解决这个问题：

```js
SiteRouter = new Backbone.Router({
    initialize: function(options) {
        // This regex will match "book/" followed by a number
        this.route(/^book\/(\d+)$/, 'bookRoute');
    },
    bookRoute: function(bookId) {
        // book route logic would go here
    }
});
```

如前一个示例所示，我们能够使用一个匹配所有可能书籍路由的正则表达式创建一个单独的路由。因为这个表达式包含一个组（正则表达式中被括号包围的部分），Backbone 会将匹配的路由部分传递给路由函数作为参数，这样我们就可以在路由函数内部知道书籍的 ID。如果我们要在正则表达式中包含多个组，Backbone 会将每个匹配的路由部分作为单独的参数传递给我们的路由函数。

正则表达式功能强大，这使得它们成为定义路由的好选择。然而，它们有一个主要的缺点：它们对人类来说很难阅读，尤其是在你几个月后再次查看它们时。正如古老的编程格言所说：你遇到了一个问题，并试图使用正则表达式来解决它。现在你有两个问题。因此，Backbone 为定义路由提供了第三个选项：路由字符串。

路由字符串类似于正则表达式；在这一点上，它们允许你定义动态路由，但它们的限制性更大。你只能定义组（也称为 **参数**），通配符组（也称为 **splat**），以及可选部分。通过放弃正则表达式的一些功能，路由字符串获得了很大的可读性。与使用 `(\d+)` 匹配书籍 ID 的路由部分相比，路由字符串将使用更易读的 `bookId`。

正如正则表达式路由一样，Backbone 将将路由的参数作为参数传递给路由函数，如下所示：

```js
SiteRouter = new Backbone.Router({
    initialize: function(options) {
        this.route('book/:bookId', 'bookRoute');
    },
    bookRoute: function(bookId) {
        // book route logic would go here
    }
});
```

如果我们想要使路由的一部分是可选的，我们可以用括号将其包裹（例如，`book/(:bookId)`）。然而，这里有一个问题；当路由字符串看到 `/` 字符时，它们会停止匹配参数（无论是否可选），这意味着如果你的书 ID 中包含 */*，它们将不起作用。例如，如果我们尝试导航到 `book/example/with/slash/5`，我们的路由就不会被触发。为了解决这个问题，Backbone 提供了一个 `wildcard` 或 `splat` 选项，它将匹配 `route` 类的剩余部分。这些 splats 就像参数一样，但它们使用 `*` 字符而不是 `:` 字符：

```js
SiteRouter = new Backbone.Router({
    initialize: function(options) {
        this.route('book/*bookId', 'bookRoute');
    },
    bookRoute: function(bookId) {
        // book route logic would go here
    }
});
// if we now navigate to "book/example/with/slash/5 " our bookRoute 
// method will receive an argument of "example/with/slash/5 "
```

由于 Backbone 对你是否使用简单路由、正则表达式或路由字符串并不特别关心，你可以在任何给定的 `Router` 中自由组合这三种类型。然而，你可能发现，每次都使用最易读的形式是最好的。换句话说，你应该先尝试使用简单路由，然后如果你需要添加一些变量，再切换到路由字符串，只有在你无法使用路由字符串定义路由时才使用正则表达式。

# 关于路由冲突的注意事项

无论你使用哪种方法来创建你的路由，都应该小心避免定义相同的路由两次，或者定义两个相互重叠的路由，例如 `foo` 和 `splatThatCouldBeFoo`。虽然 Backbone 允许你定义这样的路由，并且即使有这些路由也能继续正常工作，但它会静默地忽略第一个匹配之后的任何路由。以下是一个代码片段的例子：

```js
new Backbone.Router({
    routes: {
        'foo': function() {alert('bar')},
        ':splatThatCouldBeFoo': function() {alert('baz')},
    }
}) 
Backbone.history.start();
// navigating to #foo alerts('bar')
```

可以通过故意定义特定的路由，然后是更不具体的重叠路由来利用这种行为，但这通常不推荐。虽然几个简单的重叠路由不太可能引起问题，但如果太多，它们可能会使你的`Router`难以操作。当你有这种重叠路由时，你必须首先停下来理解所有涉及的路线，然后才能对它们中的任何一个进行更改，然后如果你犯了一个错误，很容易就会创建一个难以找到的错误。

# 跟随斜杠

在我们离开路由之前，还有一个值得注意的细节：跟随斜杠。通常，网络开发者不需要考虑跟随斜杠，因为大多数网络服务器将`foo`和`foo/`视为相同。然而，从技术上讲，它们是不同的，Backbone 也将它们视为不同，这意味着当用户导航到`foo/`时，`foo`路由不会被触发。

幸运的是，如果你使用路由字符串，可以通过在每个字符串的末尾添加（`/`）来轻松解决这个问题。如果你使用正则表达式，你可以通过在每个正则表达式的末尾添加`\/?`来实现类似的效果。然而，如果你使用简单的字符串，你可能需要定义两个不同的路由（`foo`和`foo/`），或者你只需小心不要创建任何指向`foo/`的链接。

# 重定向

通常用户通过点击锚点标签（也称为超链接）来遍历网站。对于由 Backbone 驱动的网站来说，也是如此；即使你使用基于 hash 的路由，你仍然可以为他们创建正常的超链接：

```js
<a href="#foo">Click here to go to the "foo" route</a>
```

然而，有时你可能希望使用 JavaScript 将用户移动到不同的路由。例如，你可能在页面上有一个**提交**按钮，在它触发页面的`Model`保存后，你希望它将用户重定向到不同的路由。这可以通过使用`Routers navigate`方法来完成，如下所示：

```js
var router = new Backbone.Router({
    routes: {
        foo: function() {
            alert('You have navigated to the "foo" route!');
        }
    }
});
router.navigate('foo', {trigger: true});
```

重要的是要注意，将`trigger`选项作为`navigate`的第二个参数添加。如果没有这个额外的参数，Backbone 会将用户带到路由的 URL，但不会实际触发路由的逻辑。

`navigate`方法还可以接受另一个选项：`replace`。如果提供了这个选项，Backbone 将导航到指定的路由，但不会在浏览器的历史记录中添加条目。以下是一行代码的示例：

```js
router.navigate('bar', {replace: true, trigger: true});
```

不创建历史记录条目意味着，例如，如果用户访问了另一个页面然后点击浏览器上的**后退**按钮而不是回到被替换的页面，他们将被带到浏览器历史记录中的上一个页面。由于这种行为可能会让用户感到困惑，建议你将`replace: true`的使用限制在临时路由上，例如加载页面的路由。

# 404 和其他错误

到目前为止，我们一直关注路由匹配时会发生什么，但用户访问一个没有匹配路由的 URL 时会发生什么呢？这种情况可能是因为一个过时的链接或用户输入了错误的 URL。在一个传统的网站上，服务器将通过抛出`404`错误来处理这种情况，但在 Backbone 驱动的网站上，`Router`类默认情况下将不会做任何事情。这意味着，如果你想在网站上有一个`404`页面，你必须自己创建它。

实现这一点的其中一种方法是通过`start`方法。此方法根据当前 URL 是否找到任何匹配的路由返回`true`或`false`：

```js
if (!Backbone.history.start()) {
    // add logic to handle the "404 Page Not Found" case here
}
```

然而，由于该方法只会在页面首次加载时被调用一次，因此它不会允许你捕获页面加载后发生的非匹配路由。为了捕获这些情况，你需要定义一个特殊的`404`路由，你可以使用路由字符串的`splat`语法来完成：

```js
var SiteRouter = Backbone.Router.extend({
    initialize: function(options) {
        this.route('normalRoute/:id', 'normalRoute');
        this.route('*nothingMatched', 'pageNotFoundRoute');
    },
    pageNotFoundRoute: function(failedRoute) {
        alert( failedRoute + ' did not match any routes');
    }
});
```

# 路由事件

通常，基于路由的逻辑是由`Router`类本身处理的。然而，有时你可能希望*触发*额外的逻辑。例如，你可能希望在用户访问几个路由之一时，向页面添加某个元素。在这种情况下，你可以利用 Backbone 在每次路由发生时触发事件的特性，允许你从`Router`类外部监听和响应路由变化。

你可以通过使用`on`方法以与`Models`和`Collections`中的事件相同的方式监听路由事件，如下所示：

```js
var router = new Backbone.Router();
router.on('route:foo', function() {
    // do something whenever the route "foo" is navigated to
});
```

下表显示了可以在`Router`类上监听的三种不同的路由事件：

| 事件名称 | 触发 |
| --- | --- |
| `route` | 当任何路由匹配时触发 |
| `route:name` | 当匹配指定名称的路由时触发 |
| `all` | 当在`Router`类上触发任何事件时触发 |

此外，Backbone 还提供了一个可以在`Backbone.history`对象上监听的`route`事件，如下所示：

```js
Backbone.history.on('route', function() { ... });
```

监听`history`而不是特定`Router`的优点是，它将捕获来自你网站上所有`Routers`的事件，而不仅仅是特定的一个。

# 多个路由器

通常，你只需要一个`Router`来驱动你的整个网站。然而，如果你有理由这样做，你可以轻松地包含多个`Routers`，Backbone 将愉快地允许这样做。如果同一页面上有两个或更多`Routers`匹配特定的路由，Backbone 将触发第一个具有匹配路由定义的`Router`的路由。

你应该使用多个`Routers`的主要原因有两个。第一个原因是将你的路由分成逻辑组。例如，在先前的例子中，我们使用条件语句在当前用户是管理员时将某些仅限管理员的路由添加到`Router`类中。如果我们的网站有足够的这些路由，那么为仅限管理员的路由创建一个单独的`Router`可能是有意义的，如下所示：

```js
var NormalRouter = Backbone.Router({
     routes: {
        // routes for all users would go here
    }
};
var AdminRouter = Backbone.Router({
     routes: {
        // routes for admin users only would go here
    }
};
new NormalRouter();
if (user.get('isAdmin') {
    new AdminRouter();
}
```

需要使用多个 `Routers` 的另一个原因是当你有两个不同的网站需要共享一些公共代码时。例如，一个图书销售网站可能希望有一个主网站供购物者购买书籍，并为出版商提供一个完全不同的网站来提交新书。然而，尽管这些网站是不同的，它们可能都希望共享一些公共代码，例如一个 `Book Model` 或书籍渲染的 `View`。通过使用多个 `Routers`，我们的图书销售商可以在两个网站之间共享他们想要的任何代码，同时保持每个网站页面/路由的完全独立。

在实践中，使用多个 `Routers` 可能会让人感到困惑，尤其是当它们有重叠的路由时。由于你可以轻松地动态添加路由（就像我们在早期的管理员示例中所做的那样），通常你不需要依赖多个 `Routers`，并且通过坚持只使用一个，你可能会避免混淆。

# 页面视图

在我们完成这一章之前，讨论一下 `Views` 如何与 `Routers` 交互是值得的。在本章中，我们故意对你在路由函数内部实际应该做什么保持模糊，Backbone 的美妙之处在于它完全将这个决定留给了你。

然而，对于许多 Backbone 用户来说，一个非常常见的模式是创建一个特殊的 `Page View`，然后在每个路由处理方法中实例化该 `View` 的不同子类。以下代码片段是一个例子：

```js
var Book = Backbone.Model.extend({urlRoot: '/book/'});
var Page = Backbone.View.extend({render: function() {
        var data = this.model ? this.model.toJSON() : {};
        this.$el.html(this.template(data));
        return this;
    }
});
var BookPage = Page.extend({
   template: 'Title: <%= title %>' 
});
var SiteRouter = new Backbone.Router({
    route: {
        'book/:bookId(/)': 'bookRoute',
    },
    bookRoute: function(bookId) {
        var book = new Book({id:  bookId});
        book.fetch().done(function() {
            var page = new BookPage({model: book});
            page.render();
        });
    }
});
```

如果你的网站有多个部分，例如侧导航栏和主要内容区域，你可以将这些部分拆分成它们自己的 `View` 类，然后让这些部分成为你的 `Page View` 类的默认使用部分。然后，你可以在 `Page View` 的子类中根据需要覆盖这些默认设置，以允许这些 `Views` 改变你网站的各个部分。

例如，假设大多数时候，你的网站只有一个侧导航栏，但在某些页面上，你希望向其中添加一些额外的链接。通过使用 `Page View` 架构，这样的场景很容易实现：

```js
var StandardSidebar = Backbone.View.extend({
    // Logic for rendering the standard sidebar });Page = Backbone.View.extend({
    sideBarClass:  StandardSidebar,

    render: function() {
        // render the base page HTML
        this.$el.html('<div id="sidebar"></div>' +
                      '<div id="content"></div>');

        // render the sidebar
        var sidebar = new this.sideBarClass({
            el: this.$('#sidebar')
        });
        sidebar.render();

        // logic for rendering the content area would go here
        return this;
    }
});
var AlternateSidebar  = Backbone.View.extend({
    // Logic for rendering the alternate sidebar });
var AlternateSidebarPage = Page.extend({
   sidebarClass: AlternateSidebar
});
```

正如前一个示例所示，我们的 `AlternateSidebar` 和 `AlternateSidebarPage` 类非常简短。多亏了继承的力量，它们不需要重新定义任何现有的逻辑，而是可以完全专注于它们的不同之处：渲染替代侧边栏的逻辑。虽然你的网站可能甚至没有侧边栏，但它无疑是由可组合的部分组成的，通过将这些部分拆分成单独的 `View` 类，你的路由可以简单地使用它们希望渲染的适当类。

# 摘要

在本章中，我们探讨了 Backbone 的 `Router` 类。你学习了 Backbone 如何通过在 `Router` 类上创建 `routes` 来模拟页面，以及 `Routers` 可以如何使用基于 hash 或基于 pushState 的路由来操作。你还了解了添加路由的三种不同方式（通过 `routes` 选项、`routes` 属性或 `route` 方法）以及三种类型的路由（简单路由、路由字符串和正则表达式）。最后，你看到了如何处理缺失的路由、响应路由事件、使用多个 `Routers`，以及最重要的是如何将页面视图与可组合的子 `Views` 结合起来，以增强你的路由方法。

在下一章中，我们将探讨 Backbone 的更多高级用法，例如使用方法代替 Backbone 属性，如 `model`，或者将方法集混合到多个类中。
