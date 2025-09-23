# 第四章 jQueryMX

jQueryMX 是一组 jQuery 库，提供了实现和组织大型 JavaScript 应用程序所需的功能。

它提供了经典的继承模拟，模型-视图-控制器层以提供逻辑上分离的代码库。它还提供了有用的 DOM 辅助工具、自定义事件和语言辅助工具。

在本章中，我们将介绍最常见或最有趣的例子。

### 注意

要获取完整的插件列表，请访问 [`javascriptmvc.com/docs.html#!jquerymx`](http://javascriptmvc.com/docs.html#!jquerymx)。

我们使用现有的 `Todo` 应用程序文件夹结构来尝试 jQueryMX 插件。

在 `Todo` 文件夹中创建一个 `jquerymx_playground` 文件夹，包含两个文件，使用以下代码片段：

```js
<!doctype html>

<html>
<head>
    <title>jQueryMX playground</title>
    <meta charset="UTF-8"/>
</head>
<body>

<script src="img/jquerymx_playground_0.js"></script>
</body>
</html>
```

当代码片段指示在控制台中运行时，意味着将示例粘贴并执行在打开的 index.html 页面上的 Google Chrome 控制台。我们可以使用任何网络浏览器控制台，如 Firebug，然而，目前 Google Chrome（和 Safari）似乎是最好的，并且具有非常方便的代码补全功能。

# $.Class

$.Class 提供了基于 John Resig 的 *Simple JavaScript Inheritance* 的经典继承模拟，该模拟可在 [`ejohn.org/blog/simple-javascript-inheritance/`](http://ejohn.org/blog/ simple-javascript-inheritance/) 找到。

### 提示

类签名是 `$.Class( [NAME , STATIC,] PROTOTYPE ) -> Class`。

`class` 方法对所有类实例都可用，而 `instance` 方法仅对特定实例可用。

让我们在文件 `jquerymx_playground_0.js` 中编写一些示例：

```js
steal(
    'jquery/class',
    function ($) {
        $.Class('Bank.Account', {
                setup: function () {
                    console.log('Bank.Account class: setup');
                },

                init: function () {
                    console.log('Bank.Account class: init');
                },

                getType: function () {
                    return 'Bank.Account class method';
                }
            },
            {
                setup: function () {
                    console.log('Bank.Account instance: setup');
                },

                init:  function () {
                    console.log('Bank.Account instance: init');
                },

                getType: function () {
                    return 'Bank.Account instance method';
                }
            }
        );

        Bank.Account('Bank.Account.SavingsAccount', {
                setup: function () {
                    console.log('Bank.Account.SavingsAccountclass: setup');
                },

                init: function () {
                    console.log('Bank.Account.SavingsAccountclass: init');
                }
            },
            {
                setup: function () {
                    console.log('Bank.Account.SavingsAccountinstance: setup');
                },

                init: function () {
                    console.log('Bank.Account.SavingsAccountinstance: init');
                },

//                getExtendedType: function () {
//                    return 'Hello ' + this.getType();
//                },

                getType: function () {
                    return 'Hello ' + this._super();
                }
            }
        );

//        console.log('instantiate: acc, savingAcc');
//        window.acc = new Bank.Account();
//        window.savingAcc = new Bank.Account.SavingsAccount();
    }
);
```

在控制台中，我们可以看到类已被创建。让我们按照以下方式创建一些实例：

```js
var acc = new Bank.Account();
var savingAcc = new Bank.Account.SavingAccount();
```

我们可以按照以下方式执行实例方法：

```js
acc.getType();
savingAcc.getType();
```

在以下章节中，让我们分解代码并看看这里发生了什么。

## 第一个参数

使用 `$.Class`，我们创建了一个名为 `Account` 的新类，将字符串 `Bank.Account` 作为第一个参数传递。通过使用点表示法，我们创建了一个命名空间 `Bank`。这就是为什么我们创建了一个名为 `Bank.Account` 的 `Account` 类的新实例。在这种情况下，`Bank` 只是一个空对象，帮助我们创建整洁的应用程序对象结构。

例如，可以有一个替代命名空间，如 `CompanyName.Product.SomeClass`。

## 第二个参数

作为第二个参数，我们传递了一个具有属性的对象，这些属性是所有类实例共享的类属性。

在我们的案例中，`Account` 类的 `getType` 类方法在 `SavingAccount` 类中可用。因此，我们可以在控制台中输入以下内容：

```js
Bank.Account.SavingsAccount.getType();
```

## 第三个参数

作为第三个参数，我们传递了一个具有属性的对象，这些属性是所有实例共享的实例属性。因此，我们在控制台中输入以下命令：

```js
savingAcc.getType();
```

## 方法重写

在 `getType` 实例方法示例中，我们可以看到如何在子对象中重写方法。

在`SavingAccount`中，我们通过向同名祖先方法添加额外的`Hello`字符串来覆盖`getType`方法，并使用以下方式调用祖先方法：

```js
this._super();
```

如果我们不希望使用相同的名称，可以使用以下：

```js
getExtendedType: function () {
    return 'Hello ' + this.getType();
}
```

## 生命周期

在类和实例中，我们都可以使用预定义的方法`setup`和`init`。

如果它存在，它总是会被调用，因此没有必要手动调用它。

`setup`方法首先被调用，然后是`init`方法。在大多数情况下，不需要使用`setup`方法。

# $.Model

$.Model 是应用程序数据层。它提供了一种简单的方法来连接提供 RESTful API 的服务，监听数据变化，并将 HTML 元素绑定到模型、延迟器和验证。

$.Model 非常方便；我们不需要手动使用 jQuery 的 Ajax 方法编写 XHR 调用。我们可以使用$.Model 映射我们的后端 API，然后使用其方法从服务器拉取/推送数据。

我们可以使用`$.Model.List`列表来组织$.Models，这与 Backbone.js 的集合([`backbonejs.org/#Collection`](http://backbonejs.org/#Collection))类似。

让我们在文件`jquerymx_playground_1.js`中编写一些代码：

```js
steal(
    'jquery/model',
    'jquery/dom/fixture',
    function ($) {
        $.Model('AccountModel', {
                findAll: 'GET /accounts',
                findOne: 'GET /accounts/{id}',
                create:  'POST /accounts',
                update:  'PUT /accounts/{id}',
                destroy: 'DELETE /accounts/{id}'
            },
            {

            }
        );

        // Fixtures
        (function () {
            var accounts = [
                {
                    id: 1,
                    type: 'USD'
                },
                {
                    id: 2,
                    type: 'EUR'
                }
                ];

            // findAll
            $.fixture('GET /accounts', function () {
                return [accounts];
            });

            // findOne
            $.fixture('GET /accounts/{id}', function () {
                return accounts;
            });

            // create
            $.fixture('POST /accounts', function () {
                return {};
            });

            // update
            $.fixture('PUT /accounts/{id}', function (id, acc) {
                return acc;
            });

            // destroy
            $.fixture('DELETE /accounts/{id}', function () {
                return {};
            });
        }());

        AccountModel.findAll({}, function (accounts) {
            $.each(accounts, function (i, acc) {
                $('<p>').model(acc).text(acc.type).appendTo('body');
            });
        });

        AccountModel.bind('updated', function (e, acc) {
acc.elements($('body')).remove();
        });

        AccountModel.bind('created', function (e, acc) {
            console.log('AccountModel.bind: event: ', e,'Account: ', acc);
        });
    }
);
```

让我们分解这段代码，看看这里发生了什么：

+   从`$.Model`开始的代码负责将 API 映射到我们的模型。

+   以`$.fixtures`开头的行负责模拟服务器响应。当我们需要在没有准备好或不可用的 Web 服务器 API 的情况下开始开发时，固定值非常有用。

+   `model`类中的`bind`方法负责绑定模型方法`update`和`create`。我们可以尝试使用它们，通过在`AccountModel`实例上执行这些方法来查看它们在浏览器控制台中的工作方式。

# $.View

$.View 是一个客户端模板解决方案。它使用数据填充 HTML 模板。

它包含四个预包装的模板引擎，可以从以下网站下载：

+   **EJS**：[`embeddedjs.com`](http://embeddedjs.com)（由 JMVC 团队创建的默认模板）

+   **Jaml**：[`javascriptmvc.com/docs.html#!Jaml`](http://javascriptmvc.com/docs.html#!Jaml)

+   **Micro**：[`javascriptmvc.com/docs.html#!Micro`](http://javascriptmvc.com/docs.html#!Micro)

+   **jQuery** **模板**：[`api.jquery.com/category/plugins/templates`](http://api.jquery.com/category/plugins/templates)

通过使用`$.View.register`很容易扩展它。

模板可以嵌入到 HTML 文档中，或者从外部文件同步或异步加载。$.View 支持生产构建中的模板缓存和打包。

## 嵌入

模板如下嵌入到 HTML 文档中：

让我们将以下代码复制到`index.html`文件中：

```js
<script type='text/ejs' id="accounts">
    <p>JavaScriptMVC is <%= message %></p>
</script>
```

还可以将以下代码复制到文件`jquerymx_playground_2.js`中：

```js
steal(
    'jquery/view',
    'jquery/view/ejs',
    function ($) {

    }
);
```

在控制台中，输入以下内容：

```js
$('body').html('accounts', {message: 'Awesome'});
```

因此，应该创建以下 DOM 节点：

```js
<p>JavaScriptMVC is Awesome</p>
```

## 外部

这种使用模板的方法是最常见的，因为它允许更好地组织项目文件的结构：

创建一个名为`message.ejs`的文件，并将之前的模板复制到其中。文件内容应如下所示：

```js
<p>JavaScriptMVC is <%= message %></p>
```

在控制台中输入以下内容：

```js
$('body').html('message.ejs', {message: 'Awesome'});
```

具有 message 属性的 object 被传递到 HTML 方法中，该方法使用`message.ejs`文件将文本"Awesome"渲染到`<%= message %>`的位置，并将其附加到 body DOM 节点中。

结果应该与内嵌的结果相同。

## 子模板

在模板内部我们可以嵌入另一个模板，如下所示：

```js
<%= $.View('sub-message.ejs', message); %>
```

# $.Controller

`$.Controller`插件帮助创建一个有组织、无内存泄漏的 JavaScript 代码。

使用`$.Controller`的一个很好的例子是来自第一章的`Todos`控制器，*使用 JavaScriptMVC 入门*，如下所示：

```js
$.Controller('Todos', {
            // init method is called when new instance is created
            'init': function (element, options) {
                this.element.html('todos.ejs', Todo.findAll());
            },

            // add event listener to strong element on click
            'li strong click': function (el, e) {
                // trigger custom event
                el.trigger('selected', el.closest('li').model());

                // log current model to the console
                console.log('li strong click', el.closest('.todo').model());
            },

            // add event listener to em element on click
            'li .destroy click': function (el, e) {
                // call destroy on the model to prevent memoryleaking
                el.closest('.todo').model().destroy();
            },

            // add event listener to Todo model on destroyed
            '{Todo} destroyed': function (Todo, e, destroyedTodo) {
                // remove element from the DOM tree
                destroyedTodo.elements(this.element).remove();

                console.log('destroyed: ', destroyedTodo);
            }
        });
```

# DOM 助手

DOM 助手扩展为 DOM 添加了一组有用的插件。它们将在以下章节中描述。

## $.cookie

`$.cookie`插件包含用于管理 cookie 的有用方法。

将以下代码粘贴到`jquerymx_cookie.js`文件中：

```js
steal(
    'jquery/dom/cookie',
    function ($) {

    }
);
```

我们可以使用以下命令在控制台中创建一个 cookie：

```js
$.cookie('CookieName', 'CookieValue');

```

在资源标签中我们可以看到已经创建了 cookie。

我们可以使用 cookie 名称来获取 cookie，如下所示：

```js
$.cookie('CookieName');

```

我们也可以使用以下命令删除 cookie：

```js
$.cookie('CookieName', null);

```

## $.fn.compare

`$.fn.compare`插件比较两个节点，并返回一个数字，描述它们相对于彼此的位置。

将以下代码粘贴到`jquerymx_compare.js`文件中：

```js
steal(
    'jquery/dom/compare',
    function ($) {
        $('body').append('<p>paragraph</p><strong>strong</strong>');
    }
);
```

在控制台中运行以下命令：

```js
$('p').compare($('strong'));

```

接下来，运行以下命令：

```js
$('strong').compare($('p'));

```

在第一种情况下我们应该得到`4`，在第二种情况下得到`2`。

这里数字的含义如下：

+   `0`: 元素是相同的

+   `1`: 节点在不同的文档中（或一个在文档之外）

+   `2`: `strong`在`p`之前

+   `4`: `p`在`strong`之前

+   `8`: `strong`包含`p`

+   `16`: `p`包含`strong`

## $.fn.selection

`$.fn.selection`插件设置或获取任何元素上的当前文本选择。

将以下代码粘贴到`jquerymx_selection.js`文件中：

```js
steal(
    'jquery/dom/selection',
    function ($) {
        $('body').append('<p>Hello from paragraph!</p>');
    }
);
```

在控制台中运行以下命令：

```js
$('p').selection();

```

它应该返回`null`。

现在，选择文本的一部分并再次运行命令，它应该返回以下对象：

```js
{
 end: 15,
 start: 4
}

```

要设置选择，使用以下命令：

```js
$('p').selection(6, 10);

```

## $.fn.within

`$.fn.within`插件返回给定位置内的元素。

将以下代码粘贴到`jquerymx_within.js`文件中：

```js
steal(
    'jquery/dom/within',
    function ($) {
        $('body').append('<p>Hello from paragraph!</p>');
    }
);
```

在控制台中运行以下命令：

```js
$('p').within(30, 20);

```

它应该返回一个包含所有左位置为 30 px 和上位置为 20 px 的`p`元素的数组。

## $.Range

`$.Range`插件包含对文本选择进行操作的有用方法，以支持创建、移动和比较选择。

将以下代码粘贴到`jquerymx_range.js`文件中：

```js
steal(
    'jquery/dom/range',
    function ($) {
        $('body').append('<p>Hello from paragraph!</p>');
    }
);
```

在控制台中运行以下命令：

```js
$.Range.current();

```

要获取当前范围，选择文本的一部分并再次执行代码，然后比较返回的对象。

要获取当前选择文本，请在控制台中运行以下命令：

```js
$.Range.current().toString();

```

## $.route

`$.route` 插件包含用于管理应用程序状态的有用方法。

将以下代码粘贴到 `jquerymx_route.js` 文件中：

```js
steal(
    'jquery/dom/route',
    function ($) {
        $.route.bind('change', function (e, attr, how, newVal, oldVal) {
            console.log('event: ', e, '| attribute changes: ',attr, '| how changes: ', how, '| new value: ',newVal, '| old value: ', oldVal);
        });
    }
);
```

在 URL 的末尾，输入以下内容：

```js
#!&type=UTC
```

然后，输入以下命令并观察控制台输出：

```js
#!&type=GTM
```

另一个路由示例可以在 `Todo` 应用程序的 第一章 *使用 JavaScriptMVC 入门* 中找到。

# 特殊事件

特殊事件扩展添加了一组特殊事件插件。

## $.Drag 和 $.Drop

`$.Drag` 和 `$.Drop` 插件包含拖放事件。

将以下代码粘贴到 `jquerymx_draganddrop.js` 文件中：

```js
steal(
    'jquery/event/drag',
    'jquery/event/drag/limit',
    'jquery/event/drag/scroll',
    'jquery/event/drag/step',
    'jquery/event/drop',
    function ($) {

        $('body').append('<p>Drag me, but not too far...</p>');

        $('p').bind('dragmove', function (e, drag) {
            if (drag.location.top() > 150 || drag.location.left() > 450) {
                console.log('limiter');
                e.preventDefault();
            }
        });
    }
);
```

# 语言辅助工具

语言辅助工具是一组 jQuery 插件。它们将在以下章节中描述。

## $.Object

`$.Object` 插件包含以下三个有用方法：

+   `same`：它比较两个对象

+   `subset`：它检查一个对象是否是另一个对象的集合

+   `subsets`：它返回对象的子集

### same

`same` 方法可以比较两个对象。它支持嵌套对象。我们还可以指定比较是否区分大小写或是否跳过特定属性的比较。

将以下代码粘贴到 `jquerymx_object.js` 文件中：

```js
steal(
    'jquery/lang/object',
    function ($) {
        window.object_1 = {
            property_1: 'foo',
            property_2: {
                property_1: 'bar',
                property_2: {
                    property_1: 'Hello JMVC!'
                }
            }
        };

        window.object_2 = {
            property_1: 'foo',
            property_2: {
                property_1: 'bar',
                property_2: {
                    property_1: 'HELLO JMVC!'
                }
            }
        };
    }
);
```

在控制台中运行以下命令：

```js
$.Object.same(object_1, object_2);

```

它应该返回 `false`。

现在尝试忽略大小写：

```js
$.Object.same(object_1, object_2, {property_2: 'i'});

```

它应该返回 `true`，因为 `property_2` 及其所有子项都与忽略大小写标志进行比较。

要忽略特定属性的区分大小写，我们可以指定如下：

```js
$.Object.same(object_1, object_2, {property_2: {property_2: { property_1: 'i'}}});

```

结果也应该为 `true`。

## $.Observe

`$.Observe` 插件为 JavaScript 对象和数组提供观察者模式。

将以下代码粘贴到 `jquerymx_observe.js` 文件中：

```js
steal(
    'jquery/lang/observe',
    'jquery/lang/observe/delegate',
    function ($) {
        window.data = {
            accNumber: {
                iban: 'SWISSQX',
                number: 6987687
            },
            owner: {
                fName: 'Nicolaus',
                lName: 'Copernicus'
            }
        };

        window.oData = new $.Observe(data);

        oData.bind('change', function (e, attr, how, newVal, oldVal) {
            console.log('event: ', e, '| attribute changes: ', attr, '| how changes: ', how, '| new value: ', newVal, '| old value: ', oldVal);
        });
    }
);
```

在控制台中运行以下命令：

```js
oData.attr('accNumber.number');

```

接下来，运行以下命令：

```js
data.accNumber.number;

```

应该显示相同的数字。

使用以下命令更改 `number` 属性的值：

```js
oData.attr('accNumber.number', 123456);

```

由于我们将匿名函数绑定到 `change` 事件，该事件在任何可观察对象属性发生变化时发出，因此应该显示带有所有传递信息的 `console.log`。

请注意，`oData` 是数据的副本，因此命令：

```js
oData.attr('accNumber.number');

```

与以下内容不同：

```js
data.accNumber.number;

```

## $.String

`$.String` 插件包含有用的字符串方法。

将以下代码粘贴到 `jquerymx_string.js` 文件中：

```js
steal(
    'jquery/lang/string',
    'jquery/lang/string/deparam',
    'jquery/lang/string/rsplit',
    function ($) {

    }
);
```

### deparam

此方法将 URL 参数转换为对象字面量。

在控制台中运行以下命令：

```js
$.String.deparam('en=1&home=a3373dsf6wfd&page[main]=uy7887d');
```

它应该将字符串转换为以下对象：

```js
{
    en: '1',
    home: 'a3373dsf6wfd',
    page: {
        main: 'uy7887d'
    }
}
```

## $.toJSON

`$.toJSON` 插件包含有用的对象方法。

将以下代码粘贴到 `jquerymx_tojson.js` 文件中：

```js
steal(
    'jquery/lang/json',
    function ($) {
        window.object_1 = {
            property_1: 'foo',
            property_2: {
                property_1: 'bar',
                property_2: {
                    property_1: 'Hello JMVC!'
                }
            }
        };
    }
);
```

在控制台中运行以下命令：

```js
$.toJSON(object_1);

```

它应该返回给定对象的 JSON 表示形式。

## $.Vector

`$.Vector` 插件包含用于创建和操作向量的有用方法。

让我们将以下代码粘贴到 `jquerymx_vector.js` 文件中：

```js
steal(
    'jquery/lang/vector',
    function ($) {

    }
);
```

在控制台中，运行以下命令：

```js
new jQuery.Vector(1,2);

```

它应该返回一个新的 `Vector` 实例。

# 摘要

在本章中，我们学习了 jQueryMX 插件能提供什么，以及我们如何使用它们来使我们的日常编码更加高效。

在下一章中，我们将学习关于依赖管理工具 StealJS。
