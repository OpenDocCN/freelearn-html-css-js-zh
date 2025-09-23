# 第三章：使用模型访问服务器数据

在本章中，你将学习如何：

+   使用 Backbone 的主要类 `Model` 来处理数据

+   创建新的 `Model` 子类

+   获取和设置 `Model` 属性，并在这些属性发生变化时触发其他代码

+   将这些属性存储到远程服务器并从服务器检索它们

+   使用 `Model` 从 Underscore 借用的几个便利方法

# 模型的目的

**模型** 在 Backbone 中是所有数据交互的核心，无论是客户端代码内部还是与远程服务器通信时。虽然可以使用普通的 JavaScript 对象代替 `Model`，但模型提供了三个主要优势：

+   模型使用 Backbone 的类系统，这使得定义方法、创建子类等变得容易

+   模型允许其他代码监听并响应 `Model` 属性的变化

+   模型简化并封装了与服务器通信所使用的逻辑

如前一章所述，我们可以使用 `extend` 创建自己的 `Model` 子类：

```js
var Cat = Backbone.Model.extend({
     // The properties and methods of our "Cat" class would go here
});
```

一旦我们创建了那个类，我们可以使用 JavaScript 的 `new` 关键字实例化新的 `Model` 对象，并且（可选地）我们可以传入一组初始属性和选项，如下所示：

```js
var garfield = new Cat({
    name: 'Garfield',
    owner: 'John'
}, {
    estimateWeight: true
});
```

# 属性、选项和属性

当我们谈论 Backbone 中的属性时，它们通常听起来与常规 JavaScript 属性很相似。毕竟，属性和属性都是存储在 `Model` 对象上的键值对。这两者之间的区别在于，属性在技术意义上根本不是模型的属性；相反，它们是模型属性的一个属性的属性。每个 `Model` 类都有一个名为 `attributes` 的属性，而属性本身则存储在那个 `attributes` 属性的属性中。以下是一个代码片段的例子：

```js
var book = new Backbone.Model({pages: 200});
book.renderBlackAndWhite = true;
book.renderBlackAndWhite // is a property of book
book.attributes.pages; // is an attribute of book
book.attributes.currentPage = 1; // is also an attribute of book}
```

### 注意

**属性与属性**

如前述代码片段所示，Backbone 的 `Models` 类也可以有常规的非属性属性。如果您需要在模型中存储一些数据，您可以选择使用属性或属性。通常，只有当数据将要同步到服务器或您希望代码的其他部分能够监听数据变化时，才应使用属性。如果您的数据不满足这些要求，最好将其作为常规 JavaScript 属性而不是属性存储，因为将此类数据作为属性存储将在保存 `Model` 类时给您带来更多的工作。 

在纯粹的概念层面上，任何 `Model` 类设计用来存储的核心持久信息都应该属于其属性，而任何次要的或派生的信息，或者仅设计在用户刷新页面之前持续存在的信息，应该作为属性存储。

如果你想在创建新的`Model`类时传递属性，你可以简单地将其作为第一个参数提供，Backbone 会自动添加它们。你还可以在扩展你的`Model`类时定义默认属性，这样所有新的`Model`都将具有这些属性，如下所示：

```js
var Book = Backbone.Model.extend({

    defaults: {publisher: 'Packt Publishing'}
});
var book = new Book();
book.attributes.publisher; // 'Packt Publishing'
```

然而，重要的是要记住，在 JavaScript 中，对象是通过引用传递的，这意味着你提供的任何对象将与你的`Model`类的实例共享，而不是在实例之间复制。换句话说，看看以下代码片段：

```js
var Book = Backbone.Model.extend({
    defaults: {publisher: {name: 'Packt Publishing'}}
});
var book1 = new Book();
book1.attributes.publisher.name // == 'Packt Publishing'
var book2 = new Book();
book2.attributes.publisher.name = 'Wack Publishing';
book1.attributes.publisher.name // == 'Wack Publishing'!
```

由于你可能不希望对一个模型所做的更改影响到你的其他模型，因此你应该避免在默认设置中设置对象。如果你确实需要将一个对象作为默认值设置，你可以在模型的`initialize`方法中这样做，或者使用更高级的基于函数的默认形式，我们将在第七章中介绍，*将方钉嵌入圆孔 - 高级 Backbone 技术*。

虽然为模型设置默认属性很容易，但为模型首次创建时添加默认直接属性则不然。设置这些属性的最佳方式是使用模型的`initialize`方法。例如，如果你想在你创建`Book`模型时设置`renderBlackAndWhite`属性，你可以按照以下代码片段中描述的操作进行：

```js
var Book = Backbone.Model.extend({
    initialize: function(attributes, options) {
        this.renderBlackAndWhite =  options.renderBlackAndWhite;
    }
});
```

上述代码将每个新创建的书的`renderBlackAndWhite`属性设置为在创建书时传入的`renderBlackAndWhite`选项。

# 获取器和设置器

虽然`attributes`属性（在幕后）只是一个 JavaScript 对象，但这并不意味着我们应该将其视为这样的对象。例如，你永远不应该通过直接更改`Model`类的`attributes`属性来设置属性：

```js
var book = new Book({pages: 200});
book.attributes.pages = 100; // don't do this!
```

相反，Backbone 的`Model`的属性应该使用模型的`set`方法设置：

```js
book.set('pages', 100); // do this!
```

`set`方法有两种形式或*签名*。第一种（之前已展示）接受两个参数：一个用于数据的键，一个用于其值。如果你只想一次设置一个值，这种形式非常好，但如果你需要设置多个值，你可以使用第二种形式，它只接受一个包含所有`set`值的参数：

```js
book.set({pages: 50, currentPage: 49});
```

此外，模型还有一个`unset`方法，它只接受一个*键*参数，其工作方式与 JavaScript 的`delete`语句类似，但除此之外，它还允许任何监听模型的代码知道这个变化：

```js
book.unset('currentPage');// book.attributes.currentPage == undefined
delete book.attributes.currentPage; don't do this!
```

如你所猜，通过 Backbone 检索属性是通过使用`get`方法完成的，它只接受一个*键*参数，并返回该键的值：

```js
book.set({pages: 50});
var bookPages = book.get('pages'); // == 50
bookPages = book.attributes.pages;// same effect, but again don't do this!
```

`get`方法非常简单；我实际上想在这里展示它的整个源代码：

```js
get: function(attr) {
    return this.attributes[attr];
}
```

现在，你可能正在想，如果这就是`get`的全部内容，为什么不直接使用`attributes`对象呢？毕竟，它们之间没有区别，对吧？

```js
book.get('pages'); // good
book.attributes.pages; // bad?
```

实际上，使用`get`方法有两个好处，如下所示：

+   第一个好处是使用`get`可以使你的代码更短，更易读，并且更一致，因为你的`get`调用将与你的`set`调用相匹配。

+   第二个好处，并且仅限于程序性优势，是使用`get`提供了未来可扩展性的选项。像许多 Backbone 方法一样，`get`不仅是为了即插即用而设计，也是为了由你，Backbone 开发者，进行扩展。

让我们暂时想象一下，你正在构建一个 Backbone 应用程序，有一天，你意识到你需要知道何时某些代码访问了某个属性。也许，当你添加审计或日志功能时，你需要这样做，或者你可能只是试图调试一个棘手的问题。

如果你已经编写了直接访问属性的应用程序，你需要找到代码中获取属性的所有地方，然后更新这些代码。然而，如果你已经使用`get`方法，你只需简单地覆盖相关`Model`（或`Models`）上的`get`方法，就可以从代码的单个位置访问所有现有的`get`逻辑。

# 监听事件

使用`set`和`unset`方法的主要优势之一，正如我们刚才描述的，是它们允许代码的其他部分监听 Model 属性的变化。这与在 jQuery 中监听 DOM 事件类似，并且使用`Model`的`on`和`off`方法来完成：

```js
var book = new Backbone.Model({pages: 50});
book.on('change:pages', function() { // triggers when the "pages" of book  change
    alert('the number of pages changed!');
});
book.set('pages', 51); // will alert "the number of pages has changed"
book.off('changes:pages');
book.set('pages', 52); // will no longer alert
```

正如你可以在前面的代码中看到的那样，`on`方法，就像 jQuery 一样，接受两个参数：一个`event`函数和一个`callback`函数。当事件被触发时，Backbone 会调用`callback`函数。可以使用`off`方法移除事件监听器。如果你想让你的代码监听多个事件，你可以用空格将它们组合起来，如下所示：

```js
book.on('change destroy', function() {    // this callback will trigger after a change or a destroy
});
```

Backbone 相较于 jQuery 有一个显著的改进，那就是可选的第三个`context`参数。如果提供了这个参数，回调将在注册时被绑定（就像你在上一章中提到的`_.bind`方法一样）：

```js
var Author = Backbone.Model.extend({
    listenForDeathOfRival: function(rival) {
        rival.on('destroy', function celebrateRivalDeath() {
            alert('Hurray!  I, ' + this.get('name') + ', hated ' + 
                  rival.get('name') + '!');
        }, this); // "this" is our "context" argument
    }
});
var keats = new Author({name: 'John Keats');
var byron = new Author({name; 'Lord Byron'});
byron.listenForDeathOfRival(keats);
keats.destroy(); // will alert "Hurray!  I, Lord Byron, hated  John Keats!"
```

在之前的代码中，我们定义了一个带有`listenForDeathOfRival`方法的`Author`类，该方法接受一个对手（另一个`Author`类）作为参数，并设置一个监听器以在对手被销毁时触发。我们将原始作者作为`context`参数传递，因此当回调解决时，它的`this`方法将被设置为该作者。然后我们在`byron`上调用`listenForDeathOfRival`并传入`keats`，这样`byron`就会监听`keats`上的销毁事件。

当我们在最后一行触发济慈的毁灭时，我们触发了由 `listenForDeathOfRival` 设置的事件监听器，这给出了警报信息。信息能够包含两位作者的名字，因为当回调解析时，济慈的名字可以从 `rival` 变量中获取，而拜伦的名字则作为 `Model` 的一个属性可用（因为我们设置事件监听器时将他的模型作为 `context` 参数传递）。

# 可用事件

`Model` 有几个不同的事件可供你监听，如下表所示：

| 事件名称 | 触发器 |
| --- | --- |
| `change` | 当模型的任何属性发生变化时 |
| `change:attribute` | 当指定的属性发生变化时 |
| `destroy` | 当模型被销毁时 |
| `request` | 当模型的 AJAX 方法开始时 |
| `sync` | 当模型的 AJAX 方法完成时 |
| `error` | 当模型的 AJAX 方法返回错误时 |
| `invalid` | 当由模型的 `save`/`isValid` 调用触发的验证失败时 |

此外，还有一些与 **Collection** 相关的其他事件，将在下一章中进一步解释：

| 事件名称 | 触发器 |
| --- | --- |
| `add` | 当模型被添加到集合中时 |
| `remove` | 当模型从集合中移除时 |
| `reset` | 当模型所属的集合被重置时 |

模型还有一个其他特殊的事件，称为 `all`。当其他 `Model` 事件被触发时，将响应触发此事件。

## 自定义事件

虽然你不太可能经常使用它们，但在某些场合创建自己的自定义事件可能很有用。这可以通过使用模型的 `trigger` 方法来完成，它允许你模拟来自 `Model` 的事件，如下所示：

```js
someModel.trigger('fakeEvent', 5);
```

你可以通过使用 `on` 方法以与其他任何非自定义事件相同的方式监听这些事件。传递给 `trigger` 的任何附加参数（如前例中的 `5`）都将作为监听该事件的处理器参数传递。

# 服务器端操作

一旦我们用数据填充了一个 `Model` 类，我们可能不想丢失这些数据，这就是模型 AJAX 功能发挥作用的地方。每个模型都有三种与服务器交互的方法，可以用来生成四种类型的 HTTP 请求，如下表所示：

| 方法 | RESTful URL | HTTP 方法 | 服务器操作 |
| --- | --- | --- | --- |
| `fetch` | `/books/id` | `GET` | 获取数据 |
| `save`（对于新的 `Model`） | `/books` | `PUT` | 发送数据 |
| `save`（对于现有的 `Model`） | `/books/id` | `POST` | 发送数据 |
| `destroy` | `/books/id` | `DELETE` | 删除数据 |

上面表格中的示例 URL 是 Backbone 在尝试执行任何三种 AJAX 方法时默认生成的。Backbone 与使用这种 RESTful 架构组织的服务器端 API 配合得最好。RESTful 服务器端 API 的基本思想是它由客户端可以消费的各种资源的 URL 端点组成。

在这种架构中，不同的 HTTP 方法被用来控制服务器应该对那个资源采取什么操作。因此，为了删除一个 ID 为 7 的书，RESTful API 预期一个 `DELETE` 请求到 `/books/7`。

当然，除非你通过指定一个 `urlRoot` 属性来告诉它，否则 Backbone 不会知道一个 `Book` 模型的服务器端端点将是 `/books`。你可以通过指定一个 `urlRoot` 属性来完成：

```js
var Book = new Backbone.Model.extend({
    urlRoot: '/books'
});
new Book().save(); // will initiate a PUT request to "/books"
```

如果由于某种原因，你的服务器端 API 在不同的域上，你也可以使用 `urlRoot` 方法来指定一个完全不同的域。例如，为了表明我们的 `Book` 类应该使用 [`www.example.com/`](http://www.example.com/) 的 papers 端点，我们可以设置 [`www.example.com/papers`](http://www.example.com/papers) 的 `urlRoot`。然而，由于浏览器强加的跨站脚本限制，这种方法在大多数网站上可能不起作用；因此，除非你采用特殊技术（例如，服务器端代理），否则你很可能希望你的服务器端 API 在提供你的 JavaScript 文件的同一域上。

如果你的服务器端 API 不是 RESTful 的，你仍然可以通过覆盖其内置的 `url` 方法来使用 Backbone 的 AJAX 功能。虽然这个方法的默认版本只是简单地将模型的 `urlRoot` 方法与其 ID（如果有的话）组合起来，但你可以重新定义它来指定你希望 Backbone 使用的任何 URL。

例如，假设我们的书模型有一个 `fiction` 属性，我们想要使用两个不同的端点来保存我们的书：`/fiction` 用于小说书籍，`/nonfiction` 用于非小说书籍。我们可以覆盖我们的 `url` 方法来告诉 Backbone，如下所示：

```js
var Book = Backbone.extend({
    url: function() {
        if (this.get('fiction')) {
            return '/fiction;
        } else {
            return  '/nonfiction';
        }
    }
});
var theHobbit = new Book({fiction: true});
alert(theHobbit.url()); // alerts "/fiction"
```

## 在客户端存储 URL

正如我们刚刚提到的，Backbone 允许你为你的模型指定任何你想要的 URL。然而，如果你的网站相当复杂，将实际的 URL 字符串或甚至 URL 生成函数存储在单独的 `urls` 对象中是个好主意。以下是一个代码片段的例子：

```js
var urls = {
    books: function() {
        return this.get('fiction') ? '/fiction' : '/nonfiction';
    },
    magazines: '/magazines'
};
var Book = Backbone.Model.extend({url:  urls.books});
var Magazine = Backbone.Model.extend({urlRoot:  urls.magazines});
```

虽然这并不是严格必要的（在较小的网站上可能过于冗余），但在较大的网站上，这种方法有几个好处：

+   你可以轻松地在任何模型之间共享 URL

+   你可以在你的模型和非 Backbone AJAX 调用之间轻松共享 URL

+   查找现有 URL 快速且简单

+   如果任何 URL 发生变化，你只需要编辑一个文件

## 识别

到目前为止，我们一直避免讨论 Backbone 如何确定模型（Model）的确切 ID。实际上，Backbone 有一个非常简单的机制：它使用你指定的作为 `Model` 类的 `idAttribute` 属性的任何属性。默认的 `idAttribute` 属性仅仅是 `id`；所以，如果你的服务器返回的 JSON 包含 `id` 属性，你甚至不需要指定 `idAttribute` 属性。然而，由于许多 API 不使用 `id`（例如，一些使用 `_id` 代替），Backbone 会监视其属性的变化，当它看到与 `idAttribute` 属性匹配的属性时，它将设置模型的特殊 `id` 属性为此属性的值，如下所示：

```js
var Book = Backbone.Model.extend({idAttribute: 'deweyDecimalNumber'})
var warAndPeace = new Book({{deweyDecimalNumber: '082 s 891.73/3'});
warAndPeace.get('deweyDecimalNumber'); // '082 s 891.73/3'
warAndPeace.id; // also '082 s 891.73/3'
```

除了告诉 Backbone 保存模型时要使用哪个 URL 之外，`ID` 属性还有一个功能：它的缺失告诉 Backbone 模型是新的，你可以通过使用 `isNew` 方法看到这一点：

```js
var warAndPeace = new Book({deweyDecimalNumber: '082 s 891.73/3'});
var fiftyShades = new Book();
warAndPeace.isNew(); // false
fiftyShades.isNew(); // true
```

如果你需要使用其他机制来确定 `Model` 类是否为新实例，此方法可以被覆盖。不过，关于 `isNew` 的问题还有另一个：如果新的模型没有 ID，你将如何识别它们？例如，如果你通过 `ID` 存储一组 `Model`，你会如何处理新的模型？

```js
var warAndPeace = new Backbone.Model({{id: 55});
var shades = new Backbone.Model();
var bookGroup = {};
bookGroup[warAndPeace.id] =  warAndPeace; // bookGroup = {55: warAndPeace}
bookGroup[shades.id] = shades; // doesn't work because shades.id is undefined
```

为了解决这个问题，Backbone 为每个模型提供了一个特殊的仅客户端的 `ID`，或 `cid` 属性。这个 `cid` 属性保证是唯一的，但与模型的实际 `ID`（如果有的话）没有任何联系。它也不保证一致性；如果你刷新页面，你的模型可能会有完全不同的 `cid` 属性生成。

`cid` 属性的存在允许你使用模型（Model）的 ID 来执行所有与服务器相关的任务，以及使用其 `cid` 来执行所有客户端任务，而无需每次创建新模型时都从服务器获取一个新的 ID。通过使用 `cid` 属性，我们可以解决我们之前的问题，并成功索引新的书籍：

```js
var bookGroup = {};
bookGroup[warAndPeace.cid] = warAndPeace; // bookGroup = {c1: warAndPeace}
bookGroup[fiftyShades .cid] = fiftyShades;
// bookGroup = {c1: warAndPeace, c2: fiftyShades};
```

## 从服务器获取数据

Backbone 的三个 AJAX 方法中的第一个，`fetch`，用于从服务器检索模型的数据。`fetch` 方法不需要任何参数，但它接受一个可选的 `options` 参数。此参数可以接受 Backbone 特定的选项，例如 silent（这将防止模型在 fetch 的过程中触发事件），或者任何你通常传递给 `$.ajax` 的选项，例如 *async*。

你可以通过两种方式之一指定 `fetch` 请求完成时要执行的操作。首先，你可以在传递给 `fetch` 的选项中指定一个 `success` 或 `error` 回调：

```js
var book = new Book({id: 55});
book.fetch({
    success: function() {
        alert('the fetch completed successfully');
    },
    error: function() {
        alert('an error occurred during the fetch');
    }
});
```

`fetch` 方法同样返回一个 jQuery 承诺（promise），这是一个对象，允许你说“当 fetch 完成时，做 ___”或“如果 fetch 失败，做 ___”。我们可以使用承诺（promises）而不是成功回调（callback）来在 AJAX 操作完成后触发逻辑，这种方法甚至允许我们将多个回调链式调用在一起：

```js
var promise = book.fetch().done(function() {
        alert('the fetch completed successfully');
}).fail(function() {
        alert('an error occurred during the fetch');
});
```

虽然两种方法都可行，但 Promise 风格稍微更强大，因为它允许你轻松地从单个`fetch`操作中触发多个回调，或者只在多个`fetch`调用完成后触发回调。例如，假设我们想要获取两个不同的模型，并且只有在它们都从服务器返回后显示一个警告。使用成功/错误方法可能会有些尴尬，但使用 Promise 风格（结合 jQuery 的`when`函数），问题就简单解决了：

```js
var warAndPeace = new Backbone.Model({{id: 55});
var fiftyShades = new Backbone.Model({id: 56});
var warAndPeacePromise = warAndPeace.fetch();
var fiftyShadesPromise = fiftyShades.fetch();
$.when( warAndPeacePromise, fiftyShadesPromise).then(function() {
    alert('Both books have now been successfully fetched!');
});
```

一旦`fetch`方法完成，Backbone 将获取服务器的响应，假设它是一个表示模型属性的 JSON 对象，并在该对象上调用`set`。换句话说，`fetch`实际上只是一个`GET`请求，后面跟着响应中返回的一系列内容。`fetch`方法旨在与仅返回模型 JSON 的 RESTful 风格 API 一起工作，因此如果你的服务器返回了其他内容，例如包裹在`envelope`对象中的相同 JSON，你需要覆盖一个名为`parse`的特殊`Model`方法。

当`fetch`方法完成时，Backbone 在调用`set`之前通过`parse`传递服务器的响应，并且`parse`方法的默认实现简单地返回给它的对象而不做任何修改。然而，通过用你自己的逻辑覆盖`parse`方法，你可以告诉 Backbone 如何将这个响应从服务器发送的任何格式转换为 Backbone 属性对象。

例如，假设你的服务器不是直接返回书籍的数据，而是返回一个包含书籍属性的对象，这些属性包含在一个名为`book`的键中，如下所示：

```js
{
    book: {
        pages: 300,
        name: 'The Hobbit'
    },
    otherInfo: 'stuff we don't care about'
}
```

通过覆盖`Book`类的`parse`方法，我们可以告诉 Backbone 丢弃`otherInfo`并仅使用`Book`属性作为我们书籍的属性：

```js
var Book = Backbone.Model.extend({
    parse: function(response) {
        return response.pages; // Backbone will call this.set(response.pages);
    }
});
```

## 将数据保存到服务器

一旦你开始创建模型，你无疑会发现自己想要将它们的数据保存到远程服务器，这就是 Backbone 的`save`方法发挥作用的地方。正如其名所示，`save`允许你将模型的属性发送到你的服务器。它以 JSON 格式执行，通过模型`url`方法的指定 URL，通过 AJAX `POST`或`PUT`请求。与`fetch`一样，`save`允许你提供`success`和`error`回调选项，并且（与`fetch`类似）它返回一个 promise，可以在`save`方法完成后触发代码。

下面是一个使用基于 Promise 的回调的`save`示例：

```js
var book = new Book({
    pages: 20,
    title: 'Ideas for Great Book Titles'
});
book.save().done(function(response) {
    alert(response); // alerts the the response's JSON
});
```

上述代码揭示了另一个问题：它只有在我们的服务器设置为接收所有模型的属性以 JSON 格式时才会工作。如果我们只想保存一些属性，或者如果我们想发送其他 JSON，例如包装的`envelope`对象，Backbone 的`save`方法将无法直接工作。幸运的是，Backbone 通过其`toJSON`方法提供了一个解决方案。当你保存时，Backbone 通过`toJSON`传递你的`Models`属性，而`toJSON`（就像`parse`一样）通常什么都不做，因为默认情况下`toJSON`只是简单地返回传递给它的任何内容。

然而，通过重写`toJSON`，你可以完全控制 Backbone 发送到你的服务器的内容。例如，如果我们想在另一个对象中包装我们的书籍 JSON 作为`book`属性，并添加一些其他信息，我们可以如下重写`toJSON`：

```js
var Book = Backbone.Model.extend({
    toJSON: function(originalJson) {
        return {
            data:  originalJson,
            otherInfo: 'stuff'
        };
    }
});
var book = new Book({pages: 100);
book.save(); // will send: {book: {pages: 100}, otherInfo: 'stuff'}
```

## 验证

在你向服务器发送任何内容之前，确保你发送的数据实际上是有效的是个好主意。为了解决这个问题，Backbone 为你提供了一个可选的重写方法：`validate`。`validate`方法在每次保存时都会被调用，但其默认实现只是简单地返回`true`。然而，如果你重写了它，你可以添加任何你想要的验证逻辑，如果该逻辑返回`false`，则整个`save`操作将失败并返回`false`。例如，假设我们想要确保每本新书至少有 10 页：

```js
var Book = Backbone.Model.extend({
    validate: function(attributes) {
        var isValid = this.get('pages') >= 10;
        return isValid;
    }
});
var tooShort = new Book({pages: 5});
var wasAbleToSave = tooShort.save(); // == false
```

### 注意

注意，如果验证失败，Backbone 甚至不会返回一个承诺；它只会返回`false`。

因此，如果你在`Model`类中添加验证逻辑，你将需要在每次`save`时单独测试失败的验证情况；你不能仅仅依赖于返回的承诺的`fail`方法，因为不会返回这样的承诺。换句话说，以下代码将不起作用（并且会导致错误，因为`false`没有`fail`方法）：

```js
tooShort.save().fail(function() {
    // this code will never be reached
});
```

相反，你应该使用以下代码片段：

```js
var savePromise = tooShort.save();
if (savePromise) {
    savePromise.done(function() {
        // this code will be reached if both the validation and AJAX call succeed
    }).fail(function() {
        // this code will be reached if the validation passes but the AJAX fails
    });
} else {
     // this code will be reached if the validation fails
}
```

### 注意

注意，你还可以通过使用**isValid**方法在任何时候检查你的模型的有效性，它将只返回验证结果（而不是`save`）。

# 返回 Underscore

这涵盖了`Model`的所有核心功能，但在我们继续探索`Collections`之前，值得提一下`Model`的一些便利方法。除了需要 Underscore 之外，Backbone 还将其许多`Underscore`方法作为快捷方法纳入其类中，`Model`是一个完美的例子。使用这些内置快捷方法的主要优势，除了更易于阅读之外，是它们在模型的属性上而不是在`Model`本身上操作。

例如，Underscore 有一个名为`keys`的方法，你可以使用它来获取对象上的所有键。你可以直接使用此方法来获取模型属性的键，如下所示：

```js
var book = new Backbone.Model({pages: 20, title: 'Short Title'};
var attributeKeys = _.keys(book.attributes);
alert(attributeKeys); // alerts ['pages', 'title']
```

然而，如果你使用 Model 的那个相同方法版本，你可以稍微简化你的代码，并使其稍微易于阅读：

```js
var attributeKeys =  book.keys();
alert(attributeKeys); // alerts ['pages', 'title'];
```

在 `Model` 上总共有六个这样的方法，虽然我们没有时间在这本书中解释所有这些方法，但这里简要总结一下每个方法的作用：

| 名称 | 它的作用 |
| --- | --- |
| `keys` | 这将返回每个属性的键 |
| `values` | 这将返回每个属性的值 |
| `pairs` | 这将返回一个属性键/值对的数组 |
| `invert` | 这将返回键和值颠倒的属性；例如，属性 `{'pages': 20}` 将变为 `{'20': 'pages'}` |
| `pick` | 这将返回仅指定属性的键和值；例如，`book.pick('pages')` 将返回 `{pages: 20}`，不包含任何标题或其他属性 |
| `omit` | 这将返回除指定属性外的每个属性的键和值；例如，`book.omit('pages')` 将返回 `{title: 'Short Title'}` |

# 摘要

在本章中，我们探讨了 Backbone 的 `Model` 类。你学习了如何使用 `get` 和 `set` 来更改属性，如何使用 `on` 和 `off` 来监听事件，以及如何使用 `fetch`、`save` 和 `destroy` 与远程服务器交换数据。你还学习了如何通过修改 `url`、`urlRoot` 和 `idAttribute` 属性来自定义 Backbone 以处理你的服务器 API，以及如何使用 `parse` 和 `toJSON` 来处理不同结构的数据。

在下一章中，我们将探讨 Backbone 的其他数据类 `Collection`。集合允许你一起存储多个模型，并且（就像模型一样）它们允许你监听变化并向/从服务器发送或检索数据。
