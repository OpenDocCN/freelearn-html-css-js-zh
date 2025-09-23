# 第三章。前端模式

本章将介绍一组有用的模式，这将加快您的前端工作流程。Meteor 使构建丰富的前端体验变得容易，但我们必须注意我们的界面需要多少计算才能运行。过多的计算会减慢移动设备。在本章中，我们将学习如何使我们的计算简短、DOM 简单和动画有效。您将学习以下主题：

+   响应式设计

+   超级助手

+   变量类型

+   快速表单

+   加载

+   动画

+   搜索引擎优化

# 响应式设计

应用程序总是根据其外观来评判。现在我们知道了如何让它表现良好，我们需要学习如何让它看起来很好。如今，在构建前端时，同时考虑以下两点是一种常见的做法：

+   **极简主义**：这种设计风格通过仅显示使网站正常运行的必要元素来使您的网站简洁。少即是多。

+   **响应式**：这种编程风格使您的网站无论屏幕大小如何都能使用。HTML 标记将始终根据屏幕宽度进行调整。

在这里，我们只将介绍如何编写响应式前端，因为极简主义是一个广泛且高度争议的话题。

## 通用设置

定制化的第一层开始于我们已安装的 `kyleking:customizable-bootstrap-stylus` 包。按照以下步骤设置此包：

创建以下目录：

`/_globals/client/bootstrap/custom.bootstrap.json`。

（保留 JSON 文件为空。）

使用以下方式运行 Meteor 项目：

```js
meteor

```

您会注意到已添加了三个新文件。这些文件将在运行时编译以生成您版本的 Twitter Bootstrap。

打开 `/_globals/client/bootstrap/custom.bootstrap.json`，并确保所有变量都设置为 `true`：

```js
{"_route":"/_globals/client/bootstrap/custom.bootstrap.json"},
{
  "modules" : {
    "variables": true,
    "mixins": true,

    "normalize": true,
    "print": true,
    "glyphicons": true,

    "scaffolding": true,
    "utilities": true,
    "type": true,
    "code": true,
    "grid": true,
    "tables": true,
    "forms": true,
    "buttons": true,

    "component-animations": true,
    "dropdowns": true,
    "button-groups": true,
    "input-groups": true,
    "navs": true,
    "navbar": true,
    "breadcrumbs": true,
    "pagination": true,
    "pager": true,
    "labels": true,
    "badges": true,
    "jumbotron": true,
    "thumbnails": true,
    "alerts": true,
    "progress-bars": true,
    "media": true,
    "list-group": true,
    "panels": true,
    "responsive-embed": true,
    "wells": true,
    "close": true,

    "modals": true,
    "tooltip": true,
    "popovers": true,
    "carousel": true,

    "responsive-utilities": true
  }
}
```

通过将所有变量设置为 `true`，我们正在引入 `bootstrap` 所有的功能。如果您不想使用框架中的任何功能，您可以通过在此文件中将它设置为 `false` 来禁用它。

您需要手动重新启动 Meteor 以使这些更改生效。因此，请在您的终端上按 *Ctrl* + *C* 停止 Meteor 并再次运行 `meteor` 命令。

现在，我们可以编辑 `bootstrap`。首先打开 `/_globals/client/bootstrap/custom.bootstrap.import.styl` 目录。在这里，您将找到一个我们可以玩转的 Stylus 变量库。

让我们移除圆角并修改按钮边框颜色：

```js
// /_globals/client/bootstrap/custom.bootstrap.import.styl

...

// Line #114
$border-radius-base ?=       0px
$border-radius-large ?=      0px
$border-radius-small ?=      0px
...

// Line #156
$btn-default-color ?=            #333
$btn-default-bg ?=               #fff
$btn-default-border ?=           $btn-default-color

$btn-primary-color ?=            #fff
$btn-primary-bg ?=               $brand-primary
$btn-primary-border ?=           $btn-primary-bg

$btn-success-color ?=            #fff
$btn-success-bg ?=               $brand-success
$btn-success-border ?=           $btn-success-bg

$btn-info-color ?=               #fff
$btn-info-bg ?=                  $brand-info
$btn-info-border ?=              $btn-info-bg

$btn-warning-color ?=            #fff
$btn-warning-bg ?=               $brand-warning
$btn-warning-border ?=           $btn-warning-bg

$btn-danger-color ?=             #fff
$btn-danger-bg ?=                $brand-danger
$btn-danger-border ?=            $btn-danger-bg
```

如果我们需要为自定义样式表使用 bootstrap 混合和变量怎么办？我们可以在需要访问一切时使用 `@import` 命令：

```js
@import "_globals/bootstrap/client/custom.bootstrap.import.styl"
```

让我们通过添加一个大型推广者来自定义我们的产品页面。用新的 `#promoter` 标题重写产品模板：

```js
//- /products/client/products.jade

template(name="products")
  div#products.template
    header#promoter
      h3 Your Brand
```

在我们编写产品样式之前，让我们在 `/_globals/client/main.styl` 中创建一个全局样式表，用于任何应用程序范围内的样式。在这里，我们要确保我们的初始视图设置正确，以便我们可以正确地使用相对单位：

```js
// /_globals/client/main.styl

html, body, #__flow-root, #__flow-root > .template
html, body, body > .template
    height:100%
```

使用这组规则，我们基本上确保了包括我们的第一个 `.template` DOM 元素在内的所有内容都具有视口的高度。这种技术使我们能够使相对百分比高度按预期工作。

我们还需要为 BlazeLayout 设置根 DOM 元素。让我们在客户端/服务器配置文件中完成这项操作：

```js
# /_globals/router/config.coffee

if Meteor.isClient
  BlazeLayout.setRoot 'body'
```

现在，创建 `products` 样式的目录：`/products/client/products.styl`。我们将首先将变量导入到文件中，并设置一些简单的规则，使促销看起来更好：

```js
// products/client/products.styl
@import "_globals/bootstrap/custom.bootstrap.import.styl"

#products
  #promoter
    background: $brand-primary
    height: 80%
```

太好了，这为我们可能想要添加的任何促销图片设置了 `promoter` 部分，并且网站在移动设备上的外观相当不错（但还不够好）。

## Bootstrap

Bootstrap 是一个包含 CSS 规则的库，它通过包含的类库使简单布局的开发变得快速。

让我们扩展我们的产品模板，升级我们的 `#promoter` 标题并添加一个 `#features` 部分。在这种情况下，我们将使用 Bootstrap 来管理网格：

```js
//- /products/client/products.jade

template(name="products")
  div#products.template
    header#promoter
      div.container
        div.row
          div.col-xs-12
            h3 Your Brand

    section#features
      div.container
        div.row
          div.col-xs-12.col-sm-4
            h3.text-center
              span.fa-stack.fa-lg
                i.fa.fa-square.fa-stack-2x.text-primary
                i.fa.fa-users.fa-inverse.fa-stack-1x
            h5.text-center 24 Hour Support

          div.col-xs-12.col-sm-4
            h3.text-center
              span.fa-stack.fa-lg
                i.fa.fa-square.fa-stack-2x.text-primary
                i.fa.fa-truck.fa-inverse.fa-stack-1x
            h5.text-center Next Day Delivery

          div.col-xs-12.col-sm-4
            h3.text-center
              span.fa-stack.fa-lg
                i.fa.fa-square.fa-stack-2x.text-primary
                i.fa.fa-bomb.fa-inverse.fa-stack-1x
            h5.text-center Mind Blown Guarantee
```

我们在这里做了很多事情，但没有什么复杂的。首先，我们将 `#promoter` 标题的内容包装在 Bootstrap 的网格中。Bootstrap 网格只有在具有 `row` 类的情况下才会按预期工作。`container` 类控制我们的页面宽度断点。

Bootstrap 列的设置使用 `col-` 类前缀，后跟目标屏幕尺寸，然后是一个可选的偏移指示符，最后是元素将跨越或偏移的列数：

```js
div.col-<screen size>-<optional offset>-<number of columns>
```

| Class | Breakpoint | Offset |
| --- | --- | --- |
| `col-xs-*` | <768px | `col-xs-offset-*` |
| `col-sm-*` | ≥768px | `col-sm-offset-*` |
| `col-md-*` | ≥992px | `col-md-offset-*` |
| `col-lg-*` | ≥1200px | `col-lg-offset-*` |

Bootstrap 总共有 12 列（这可以在我们的 `variables` 文件中修改）；每一列都支持偏移和嵌套。记住，如果未定义其他屏幕尺寸，则最小的屏幕尺寸将覆盖所有其他屏幕尺寸。因此，如果您想为每个尺寸定义三个列，您只需为最小尺寸的屏幕定义它即可。在这个例子中，我们将我们的 `#features` 部分分成三个列，每个列的尺寸都大于 768 px，并在小于 768 px 的任何地方将其视为单列。

如果在任何时候需要嵌套 Bootstrap 列，那么您需要将这些新列放置在 `row` 类下。以下示例在一个六列长的 `div` 内部中心对另一个六列长的 `div`：

```js
div.row
  div.col-xs-6
  div.col-xs-6
    div.row
      div.col-xs-6.col-xs-offset-3
```

我们还使用 Font Awesome 为我们的 `#features` 部分创建了一些图标。在这里，我们使用 Font Awesome 的不太为人所知的 **图标堆叠** 功能。要使用此功能，您只需将一组 Font Awesome 图标包装在一个 `span.fa-stack` 元素中。

Font Awesome 有一份大约 500 个图标的列表。如果你想使用不同的图标集，请随意查看在线图标列表[`fortawesome.github.io/Font-Awesome/icons`](http://fortawesome.github.io/Font-Awesome/icons)。

现在让我们在我们的前端添加一点 Jeet 和 Rupture。

## 使用 Rupture 的 Jeet 网格系统

我们安装的 Stylus 版本（`mquandalle:stylus`）附带了许多有用的混合函数，其中最显著的是**Jeet**和**Rupture**。Jeet 是一个比 Twitter Bootstrap 更优越的网格系统，因为它更灵活，而 Rupture 是一组简化媒体查询的函数。了解如何利用 Jeet 和 Twitter Bootstrap 将提高你前端设计的质量。

让我们修复我们的`#promoter`标题，使我们的页面对在较小设备上以横幅模式浏览的用户更加友好：

```js
// products/client/products.styl
@import "_globals/client/bootstrap/custom.bootstrap.import.styl"
@import "jeet" // Add jeet

#products
  #promoter
    background: $brand-primary
    height: 80%

    +below($screen-sm-min) // If the screen width is smaller than -sm
      +portrait() // And the device is in portrait mode
        height: 30%

      +landscape() // And the device is in landscape mode
        col(1/3)
        height: 100%

  #features
    +below($screen-sm-min) // If the screen width is smaller than -sm
      +landscape() // And the device is in landscape mode
        col(2/3)
        height: 100%

        .container
          width:100% // Override bootstrap container
```

Rupture 添加了几个混合函数来轻松控制`@media`查询：`below`、`above`、`between`、`landscape`、`portrait`以及更多。你会发现自己在使用`below`、`landscape`和`portrait`时比其他情况更频繁。要在 Stylus 中使用这些混合函数，我们只需在命令前加上一个加号（`+`）。

在这个例子中，我们使用`+below($screen-sm-min)`来定义所有宽度低于 Bootstrap 的`-sm`断点的设备的 CSS 规则。然后我们将使用`+portrait()`和`+landscape()`来定义横幅和横幅模式的 CSS 规则。

接下来，我们将使用 Jeet 的`col()`混合函数来控制主列。这个混合函数接受一个分数，该分数定义了元素的宽度作为分数。考虑分子是你希望元素占用的列数，分母是总列数：

```js
col(<column width>/<number of columns>)
```

在这个例子中，当我们切换到小设备上的横幅模式时，我们的`#promoter`标题宽度为三列中的一列，而我们的`#features`部分宽度为三列中的两列。

# 超级辅助函数

在 Meteor 中，你很快会发现自己在模板中重复使用辅助函数，用于简单的格式化等任务。我们可以通过创建一个全局函数字典来防止重复，这就是我们所说的超级辅助函数。为此，我们将利用 Meteor 的渲染引擎——Blaze。

重要的是要理解 Blaze 与 Spacebars 深度集成，而 Spacebars 是 Meteor 对 HandlebarsJS 的更新版本。HandlebarsJS 是一个 JavaScript 模板引擎，它通过使用`{{}}`在前端启用辅助函数和组件的使用。这个遗留意味着 Spacebars 拥有很多 HandlebarsJS 的功能。因此，HandlebarsJS 中的大部分文档也适用于 Meteor 辅助函数。

## 定义 Blaze 辅助函数

Meteor 提供了`Template.registerHelper`函数来创建全局辅助函数。让我们创建一些帮助我们格式化货币的工具：

```js
# /_globals/client/formatters.coffee

Template.registerHelper "format_money", (value) ->
  if _.isNumber value
    "$#{(value / 100).toFixed(2)}"
```

如您所见，全局辅助器的语法与模板辅助器的语法完全相同。现在，我们可以在任何模板中使用这个辅助器。让我们在我们的 `/products` 目录下创建一个产品模板，并在这里尝试它：

```js
//- /products/client/product.jade

template(name="product")
  div.col-xs-12.col-sm-4.col-md-3
    article#product
      h5 {{name}}

      div.offer
        span.price {{format_money price}}
```

我们还需要修补我们的产品模板：

```js
//- /products/client/products.jade

template(name="products")
  div#products.template
    header#promoter
      ...
    section#features
      ...

    br
    section#featured_products
      div.container
        div.row
          each products
            +product
```

让我们在我们的产品集合中添加一些临时条目。我们将在这章的其余部分使用这些条目：

```js
@Products = new Mongo.Collection "products"

if Meteor.isServer
  if Products.find().count() is 0
    Products.insert
      name:"Nuka Cola"
      price: 1099

    Products.insert
      name:"1up Soda"
      price: 999

    Products.insert
      name:"JuggerNog"
      price: 899

# /products/client/products.coffee

Template.created "products", ->
  ...

Template.products.helpers
  products: ->
    Products.find()
```

注意，在这个数据模式中，我们定义 `price` 为分。当在 JavaScript 中处理金钱时，这是一种常见的做法，以避免算术浮点问题。这意味着什么？请转到您的网络浏览器控制台，并运行以下操作：`0.1 + 0.2`。奇怪的是，JavaScript 并没有响应 `0.3`；它响应的是 `0.3000000000004`。这是因为 JavaScript 对十进制数的内部 64 位浮点表示。这意味着 JavaScript 并不完全理解十进制数是什么。这就是为什么，当我们处理金钱时，我们不使用十进制数，我们只使用最小的可用货币单位。

## 制作全局字典

等等，我们还可以让这个格式化器在我们的应用程序中更广泛地可用。让我们将其转换成一个字典，并与我们的控制器共享。换句话说，我们希望能够使用一个如 `{{format.money amount}}` 的辅助器，并在我们的 CoffeeScript 中以 `format.money amount` 的形式使用它：

```js
# /_globals/client/formatters.coffee

@format =
  money: (value) ->
    if _.isNumber value
      "$#{(value / 100).toFixed(2)}"

Template.registerHelper "format", ->
  format
```

首先，我们创建一个包含我们定义的对象，并使用 `@`（等于 `this`）将其设置为全局。然后我们将这个定义对象放入一个全局辅助器中。很简单！现在我们知道了如何构建一个可以在任何地方使用的超级辅助器。为了测试这个，请在浏览器控制台中输入 `format.money(1099)`；这应该返回 **"$10.99"**。

# 变量类型

我们需要了解四种类型的变量来优化我们的前端工作流程：**会话**、**持久**、**文件作用域**和 **ReactiveVar**。有了这些变量，我们可以根据用户与视图的交互轻松构建动态网站。

会话变量是仅在会话期间存在的变量。每当用户访问网站时，会话就开始了。这些变量可以在不同的路由之间共享，并且是响应式的。

持久变量是存储在本地存储中的变量，这样即使在会话关闭后也可以读取。

文件作用域变量是仅存在于文件作用域内的变量。这些变量通常不是响应式的。

ReactiveVar 变量可以作用域到文件或全局，并且是响应式的。这些变量不会像会话变量那样持久化。

## 会话变量

一些 Meteor 开发者认为会话变量被过度使用。这绝对是正确的，如果你没有管理它们。大部分情况下，会话变量应该用于处理在应用程序中跨多个路由共享的数据，没有数据库中的位置，需要响应性，并且可以通过热代码重载持久存在。

如果你停下来思考一下，不是很多事物都需要所有这些属性。一开始，你可能严重依赖它们来控制你的前端，但随着你的前端变得更加简约，你会意识到它们其实没有必要。

然而，如果你打算使用它们，你应该遵循两条规则来维护它们，如下所示：

+   基于模板的分类法

+   如果你不会使用它们，请清除它们

会话变量的分类法应该始终遵循`<template.variable>`模式，以防止它们相互污染。假设我们想使用会话变量来控制全局警告，会话变量看起来会是这样：

```js
Session.set "products.alert",true
```

当我们的用户离开视图时，我们应该清除会话变量。`clear`函数是由我们在第一章中安装的`u2622:persistent-session`包添加的，*使用 Meteor 入门*：

```js
# Clear "products.alert" variable after "products" template is destroyed
Template.destroyed "products", ->
  Session.clear "products.alert"

# Clear all variables after the "products" template is destroyed
Template.destroyed "products", ->
  Session.clear()
```

### 小贴士

不要将会话变量与 Meteor 方法一起使用来获取服务器数据。虽然方法可以工作，但它会使你的数据容易受到攻击并在客户端被修改。

此外，尽量远离使用由多个键或数组组成的复杂对象的会话变量。你很快会发现自己在使用像`_.find`和`_.findWhere`这样的较小查找工具来使用数据。

## 持久变量

持久变量是会话变量，它们在会话中持续存在。在 Meteor 中，一个常见的会话变量如果用户刷新页面将会重置。持久变量利用 AmplifyJS，它反过来使用 HTML5 本地存储并回退到将变量存储在用户的浏览器中。这些类型的变量存在是因为`u2622:persistent-session`包。

我们可以使用这个变量来跟踪用户的订单，以防他们离开网站且未登录。因此，如果用户打开一个订单并填充它，他们可以在稍后回来而不需要在我们的网站上创建用户账户。

让我们创建一个`add-to-cart`按钮，它将利用持久变量：

```js
# /products/client/product.coffee

Template.product.events
  "click button.add-to-cart": (event) ->
 # Get the session variable
    order_id = Session.get "global.order"
    order = Orders.findOne order_id

 # Insert Order if it doesn't exist
    unless order
      order_id = Orders.insert
        status:"new"
        total_products:0
        subtotal:0
        total:0
    else
      order_id = order._id

 # Set the session variable for future reference
    Session.setPersistent "global.order",order_id

 # Find the order
    order = Orders.findOne order_id

 # Check for details on this product
    detail = OrderDetails.findOne
      product:@_id
      order:order._id

    if detail
 # Increase by one if the details exist
      OrderDetails.update detail._id,
        $inc:
          quantity:1

      Orders.update order._id,
        $inc:
          total_products:1
          subtotal:@price
          total:@price
    else
 # Insert if details do not exist
      OrderDetails.insert
        quantity:1
        product:@_id
        order:order._id

      Orders.update order._id,
        $inc:
          total_products:1
          subtotal:@price
          total:@price
```

我们需要构建一个发布者来实现这一点，但我们必须做出选择。因为我们既有`products`和`product`模板，每个都可以订阅一个发布者并拉取相同的数据。我们应该从`product`还是`products`模板订阅？让我们两者都做！扩展产品发布者将确保我们获得模板所需的所有数据，而附加的发布者将帮助我们管理在需要时“产品详情”页面和促销活动：

```js
# /products/server/products_pub.coffee
Meteor.publish "products", (ops={}) ->
  ...

  if ops.order and not _.isEmpty ops.order
    @relations
      collection:Orders
      filter:
        _id:ops.order
        status:"new"
      mappings:[
        {
          collection:OrderDetails
          key:"order"
        }
      ]
```

然后我们在客户端使用订单键连接我们的订阅者：

```js
# /products/client/products.coffee

Template.created "products", ->
  @autorun =>
    ...

    order = Session.get "global.order"
    if order and not _.isEmpty order
      _.extend filter,
        order:order

    @subscribe "products", filter
```

这样一来，现在我们可以向订单中添加产品，刷新网站，并且仍然回到我们的订单。让我们完成 `product` 模板的发布者构建：

```js
# /products/server/product_pub.coffee

Meteor.publish "product", (ops={}) ->
  if ops.product and not _.isEmpty ops.product
    @relations
      collection:Products
      options:
        _id:ops.product
      mappings:[
        {
          key:"product"
          collection:ProductImages
        }
        {
          collection:ProductsTags
          key:"product"
          mappings: [
            {
              collection:Tags
              foreign_key:"tag"
            }
          ]
        }
      ]

    if ops.order and not _.isEmpty ops.order
      @relations
        collection:Orders
        filter:
          _id:ops.order
          status:"new"
        mappings:[
          {
            collection:OrderDetails
            key:"order"
            filter:
              product:ops.product
          }
        ]

  @ready()
```

注意，我们的产品只带来了与该产品以及该订单相关的数据。现在我们可以编写我们的订阅者程序：

```js
# /products/client/product.coffee

Template.created "product", ->
  @autorun =>
    filter = {}

    # Get the product ID from the context
    product = @data._id
    _.extend filter,
      product:product

    # Get the order if any
    order = Session.get "global.order"
    if order and not _.isEmpty order
      _.extend filter,
        order:order

    @subscribe "product", filter

...
```

## 文件作用域变量

这些是您在大多数情况下想要使用的变量类型。它们是什么？它们是常规变量！这意味着它们不是反应式的，所以要注意。常规变量仅在文件范围内有效，因此您不需要担心与其他文件中具有相同名称的变量发生冲突。

这些变量对于诸如列表等您知道将不会改变且不需要在应用程序的其他地方使用的项目非常有用。要使用一个，只需在您的 Meteor 特定函数之外输入一个变量。

## ReactiveVar 变量

ReactiveVar 变量是反应式变量，它们在控制台中的可用性不同于会话变量。这使得它们非常适合那些需要保持相对难以从控制台访问的反应式事物。

如果在任何时候您正在创建一个高度反应式的界面，您应该用这些变量填充它，这样您就不必担心管理它们。如果有热代码重新加载或页面刷新，这些变量会自动清除。

要创建一个反应式变量，您只需使用其构造函数定义一个变量：

```js
reactive_variable = new ReactiveVar(<optional-default-value>)
```

这个变量将公开设置器和获取器函数。这意味着您使用 `.get` 命令来查看变量的值，使用 `.set` 命令来更改它：

```js
# Get the value of the variable
reactive_variable.get()

# Set the value of the variable
reactive_variable.set "hello"
```

让我们使用 ReactiveVar 变量来为我们的数量字段创建一个丰富的数字输入界面。我们将首先定义一个路由名称为 `order_quantity` 并带有 `product` 参数：

```js
# /orders/cart/cart_route.coffee

FlowRouter.route "/cart",
  ...

FlowRouter.route "/cart/:product/quantity",
  name:"order_quantity"
  action: ->
    FlowLayout.render "layout",
      content:"order_quantity"
```

我们可以使用一个新按钮将我们的产品连接起来，该按钮将带我们进入修改数量视图，并传递我们的产品 ID：

```js
# /products/client/product.coffee

Template.created "product", ->
  ...

Template.product.events
  "click button.add-to-cart": (event) ->
    ...

  "click button.modify-quantity": ->
    FlowRouter.go "order_quantity",
      product:@_id

//- /products/client/product.jade

template(name="product")
  div.col-xs-12.col-sm-4.col-md-3
    article#product
      h5 {{name}}

      div.offer
        span.price {{format.money price}}

      button.add-to-cart.btn.btn-block.btn-primary Add to Cart

      button.modify-quantity.btn.btn-block.btn-info Quantity
```

现在，我们需要设计一个响应式的数字键盘。首先，让我们在我们的 `/_globals/client/main.styl` 目录中添加一个全局的 `vertical-align` 类，这样我们就可以轻松地垂直对齐元素：

```js
// /_globals/client/main.styl

.vertical-align
  position:relative
  transform:translateY(-50%)
  top:50%
```

我们将使用 Jeet 来控制这个设计，因为它需要对我们的 DOM 元素进行更具体的控制。让我们首先在我们的 `order_quantity` 模板中创建页面的布局：

```js
//- /orders/cart/client/order_quantity.jade

template(name="order_quantity")
  div#order_quantity.template
    section#number
      h1.text-center.vertical-align {{total}}

    section#number-pad
      each numbers
        div.number
          h2.text-center.vertical-align {{number}}

      div.delete
        p.text-center.vertical-align
          i.fa.fa-undo.fa-2x

      div.add-to-cart
        p.text-center.vertical-align
          i.fa.fa-check.fa-2x
```

注意，我们正在使用 `vertical-align` 类来对齐我们想要垂直对齐的元素。我们将将这些元素中的每一个都做成一个大按钮。此外，我们将创建一个 `numbers` 辅助器来将数字写入数字键盘，而 `total` 辅助器将是我们的反应式变量。在我们创建辅助器之前，让我们看看我们的样式：

```js
// /orders/cart/client/order_quantity.styl

@import "_globals/client/bootstrap/custom.bootstrap.import.styl"
@import "jeet" // Add jeet

#order_quantity
  overflow:hidden
  background:$brand-primary
  color:white

  section#number
    height:50%
    h1
      margin:0

  section#number-pad
    height:50%
    .number, .delete, .add-to-cart
      cursor:pointer
      height:25%
      col(1/3,gutter:0,cycle:3)
      h2, p
        margin:0
```

在这个例子中，我们向 Jeet 传递了参数：`gutter`和`cycle`。`gutter`确保没有边距，而`cycle`确保元素只能以每行最多 3 个元素的方式形成行。我们让每个部分恰好占据屏幕高度的 50%，但通过将`overflow`设置为隐藏来确保不会发生滚动。因为我们总是会有四行数字，所以我们为每个数字设置了 25%的高度。

现在，让我们将我们的`ReactiveVar`变量知识付诸实践。首先，我们设置我们的变量并将其附加到模板实例上，如下所示：

```js
# /orders/cart/client/order_quantity.coffee

# Attach a reactive variable to the instance
# this variable controls our total
Template.created "order_quantity", ->
  @total = new ReactiveVar()
```

让我们构建`numbers`辅助函数和将渲染我们的(`total`)反应性变量的辅助函数：

```js
# /orders/cart/client/order_quantity.coffee

...

Template.order_quantity.helpers
 # Create a list of numbers for the number pad
  "numbers": ->
    _.map [1,2,3,4,5,6,7,8,9,0], (v,k) ->
      number:String v

 # Get the reactive variable
 # this will automatically update when the variable changes
  "total": ->
    Template.instance().total.get()
```

我们需要处理事件来更新我们的`ReactiveVar`变量：

```js
# /orders/cart/client/order_quantity.coffee

...

Template.order_quantity.events
 # Concatenate numbers to make it work like a number pad
  "click .number": (event,i) ->
    total = i.total.get()

    if total
      new_total = "#{total}#{@number}"
    else
      new_total = "#{@number}"

    i.total.set new_total

 # Remove last number from string
  "click .delete": (event,i) ->
    total = i.total.get()

    if total
      i.total.set total.slice 0,-1

  "click .add-to-cart": (event,i) ->
 # Get the session variable
    order_id = Session.get "global.order"
    order = Orders.findOne order_id

 # Get the total
    total = i.total.get()
    unless total
      return
    else
      total = Number total

 # Get the product with the ID from the router
    product = Products.findOne FlowRouter.current().params.product

 # Insert Order if it doesn't exist
    unless order
      order_id = Orders.insert
        status:"new"
        total_products:0
        subtotal:0
        total:0
    else
      order_id = order._id

 # Set the session variable for future reference
    Session.setPersistent "global.order",order_id

 # Find the order
    order = Orders.findOne order_id

 # Check for details on this product
    detail = OrderDetails.findOne
      product:product._id
      order:order._id

    if detail
 # Increase by one if the details exist
      OrderDetails.update detail._id,
        $inc:
          quantity:total

      Orders.update order_id,
        $inc:
          total_products:1
          subtotal:product.price * total
          total:product.price * total
    else
 # Insert if details do not exist
      OrderDetails.insert
        quantity:total
        product:product
        order:order._id

      Orders.update order._id,
        $inc:
          total_products:1
          subtotal:product.price * total
          total:product.price * total

    FlowRouter.go "products"
```

如果你仔细看看`add-to-cart`事件的工作方式，你会注意到它与为产品模板编写的那个事件几乎相同。在这个时候，我们实际上是在重复自己，但我们将通过`Meteor.methods`来减少代码的重复性。

注意我们是如何使用 ReactiveVar 的，就像我们使用会话变量一样，不同之处在于我们没有将变量暴露给控制台。这有助于使界面更加安全。

让我们别忘了添加这个视图所需的发布者，我们需要`Products`、`Orders`和`OrderDetails`：

```js
# /orders/cart/client/order_quantity.coffee

Template.created "order_quantity", ->
  @total = new ReactiveVar()

  @autorun =>
    @subscribe "order_quantity",
      product:FlowRouter.current().params.product
      order:Session.get "global.order"

# /orders/cart/server/order_quantity_pub.coffee

Meteor.publish "order_quantity", (ops={}) ->
  if ops.product and not _.isEmpty ops.product
    @relations
      collection:Products
      filter:
        _id:ops.product

    @relations
      collection:Orders
      filter:
        _id:ops.order
        status:"new"
      mappings:[
        {
          key:"order"
          collection:OrderDetails
          filter:
            product:ops.product
        }
      ]

  @ready()
```

我们学习了如何使用 ReactiveVar 变量构建自定义表单。这个主题涵盖了 ReactiveVar 并触及了 Jeet。我们学习了如何使用反应性变量来创建丰富的用户体验。

# 表单

到目前为止，我们的按钮和表单非常低效，因为它们不容易重复，因此也不容易维护。我们可以使用两种方法来使我们的代码更容易管理：

+   Meteor Methods

+   Autoform

使用 Meteor 方法，我们可以轻松创建**可重复**和**安全**的函数，而 autoforms 可以识别集合的结构并从中生成表单元素。这提供了相当一层的安全保障，而且不需要太多努力。Autoform 和 Meteor Methods 也可以一起使用，以进一步增强安全性。

autoform 方法可以通过`aldeed:autoform`和`aldeed:collection2`包实现。

## Meteor Methods

Meteor Methods 可以在客户端或服务器上使用。它们是通过使用`Meteor.methods(<object-of-functions>)`函数创建的，并通过`Meteor.call(<function-name>, <callback function>)`函数运行。从客户端运行一个 Meteor 方法会导致同一函数运行两个版本。一个在服务器上运行并操作数据，另一个在客户端运行并模拟数据操作。这就是我们所说的方法占位符。让我们看看一些示例代码来理解这个概念：

```js
# Define your method CLIENT SIDE
Meteor.methods
  say_hello: ->
    console.log "hello"

# Define your method SERVER SIDE
Meteor.methods
  say_hello: ->
    console.log "I don't want to say hello"

# Run the function CLIENT SIDE
Meteor.call "say_hello"
```

一旦运行函数，Meteor 方法将激活服务器端和客户端函数。因此，在这种情况下，服务器将在控制台输出 "我不想说你好"，而客户端将输出 "hello"。这是 Meteor 提供的一个特殊功能，用于帮助验证代码或在服务器端并行运行其他函数，同时不向客户端暴露敏感数据。

然而，值得注意的是，客户端方法将立即运行，而服务器端方法将需要更长的时间。因此，客户端代码应该用于模拟，而服务器端代码应该用于验证。

让我们定义并使用一个在客户端和服务器端都运行的 Meteor 方法。让我们首先重新定义我们之前编程的 `add-to-cart` 事件：

```js
# /orders/cart/cart_methods.coffee

Meteor.methods
  "cart.add-to-cart": (ops={},callback) ->
    #ops
      # order
      # product
      # quantity

    order = Orders.findOne ops.order
    product = Products.findOne ops.product

    # Insert Order if it doesn't exist
    unless order
      order_id = Orders.insert
        status:"new"
        total_products:0
        subtotal:0
        total:0
    else
      order_id = order._id

    # Set the session variable for future reference
    if Meteor.isClient
      Session.setPersistent "global.order",order_id

    # Find the order
    order = Orders.findOne order_id

    # Check for details on this product
    detail = OrderDetails.findOne
      product:product._id
      order:order._id

    if detail
      # Increase by one if the details exist
      OrderDetails.update detail._id,
        $inc:
          quantity:ops.quantity

      Orders.update order._id,
        $inc:
          total_products:ops.quantity
          subtotal:product.price * ops.quantity
          total:product.price * ops.quantity
    else
      # Insert if details do not exist
      OrderDetails.insert
        quantity:ops.quantity
        product:product._id
        order:order._id

      Orders.update order._id,
        $inc:
          total_products:ops.quantity
          subtotal:product.price * ops.quantity
          total:product.price * ops.quantity

    # Run the callback function if it exists
    callback and callback(null, true)
```

```js
Meteor.isClient. We have modified the rest of our code to accept quantity, a product ID, and an order ID, which is all we need to be able to manage both the events.
```

此外，我们在方法末尾添加了一行代码来检查回调函数，如果存在则执行它。观察我们如何在函数开始时将一个函数作为我们的参数之一接受。为了遵循 Meteor 的做法，函数执行完成后，我们返回 `null` 作为错误对象，并返回 `true` 作为结果。现在让我们替换事件：

```js
# /orders/cart/client/order_quantity.coffee

# Attach a reactive variable to the instance
# this variable controls our total
Template.created "order_quantity", ->
  ...

Template.order_quantity.helpers
  ...

Template.order_quantity.events
  ...

  "click .add-to-cart": (event,i) ->
    # Get the total
    total = i.total.get()
    unless total
      return
    else
      total = Number total

    Meteor.call "cart.add-to-cart",
      order:Session.get "global.order"
      product:FlowRouter.current().params.product
      quantity:total
      (error,r) ->
        if not error
          FlowRouter.go "products"

# /products/client/product.coffee

Template.created "product", ->
  ...

Template.product.events
  "click button.add-to-cart": (event) ->
    Meteor.call "cart.add-to-cart",
      order:Session.get "global.order"
      product:@_id
      quantity:1
      (error,r) ->
        if not error
          FlowRouter.go "products"
```

Meteor 方法非常适合创建丰富且易于重复的用户界面，但如果你只更新单个集合，你最终会发现自己在重复某些模式。这就是自动表单发挥作用的地方。

## 自动表单

`autoform` 插件是与 `collection2` 插件一起安装的。这两个插件协同工作，可以快速创建支持错误处理的表单。关于这些插件的文档非常丰富，但阅读这些文档对于确保你的视图安全且易于构建是必要的。你可以在 [`github.com/aldeed/meteor-autoform`](https://github.com/aldeed/meteor-autoform) 找到这些文档。我们将介绍一个简单的模式，以展示如何轻松地操作自动表单。

要使用自动表单，你首先需要为你的集合创建一个 `collection2` 架构。这定义了自动表单将要使用的规则来验证和生成我们的输入字段。让我们填充我们的产品架构：

```js
# /_globals/lib/collections/products/products_collection.coffee

@Products = new Mongo.Collection "products"

Products.attachSchema new SimpleSchema
  name:
    type:String
    label:"Name"

  description:
    type:String
    label:"Description"
    optional:true

  sku:
    type:String
    label:"SKU"
    optional:true

  price:
    type:Number
    label:"Price"
```

```js
new SimpleSchema constructor, then we have attached the schema to the collection using the .attachSchema function. The SimpleSchema constructor takes an object where the first key defines the name of the field and the object within that key defines the way the field will behave. We will dive into this further in the next chapter.
```

将架构附加到我们的集合中确保客户端控制台命令不能添加不属于架构的信息到数据库。让我们重置我们的项目以确保我们的新产品适应新的架构：

```js
meteor reset

```

现在，我们可以使用自动表单为我们的产品集合创建一个简单的插入表单。让我们创建模板和路由：

```js
//- /products/client/create_product.jade

template(name="create_product")
  h3.text-center create product

  div.container
    div.row
      div.col-xs-12
        +autoForm collection="Products" type="insert" id="insert_product" preserveForm="true"
          +afQuickField name="name" autocorrect="off" autocomplete="off"
          +afQuickField name="price"
          +afQuickField name="description"
          +afQuickField name="sku"

          button.btn.btn-block.btn-primary Add Product

# /products/products_route.coffee

FlowRouter.route "/",
  ...

FlowRouter.route "/products/create",
  name:"create_product"
  action: ->
    FlowLayout.render "layout",
      content:"create_product"
```

注意，我们不需要编写控制器来使这个功能工作。在这个情况下，我们需要理解两个关键组件：`autoForm` 和 `afQuickField`。

当我们使用 `autoForm` 与一个集合一起时，我们必须使用 `collection` 参数声明集合的名称。此外，我们需要使用 `type` 参数定义表单是进行插入、更新还是方法调用，最后，我们给表单一个 `id`，它将被用作表单的 HTML `id` 属性。此外，我们传递 `preserveForm` 参数以确保数据在热代码重新加载中保持持久。autoform 还接受其他一些参数，但这些是最常用的。

`afQuickField` 组件专门创建了一个可以轻松处理错误的 `bootstrap3` 输入字段。请注意，我们还可以将这些组件以及参数添加 HTML 属性（如 `autocorrect` 和 `autocomplete`）。有各种方法可以自定义这些输入，但我们建议只使用两种：直接修改 CSS 或利用 `afFieldInput` 和 `afFieldIsInvalid` 来构建新元素。

让我们进一步自定义我们的 `price` 字段：

```js
    div.form-group(class="{{#if afFieldIsInvalid name='price'}} has-error has-feedback {{/if}}")

      label.control-label {{afFieldLabelText name="price"}}

      +afFieldInput name="price" validation="submit"

      if afFieldIsInvalid name="price"
        span.glyphicon.glyphicon-certificate.form-control-feedback
        span.help-block {{afFieldMessage name="price"}}
```

通过这样做，我们非常容易地使用 bootstraps 的类为我们的字段添加一个 `glyphicon`，但这在需要时可以用于适应任何框架。

尽管如此，这个表单仍然存在一些问题，人们输入价格时使用的是美元而不是分，但我们的模式只接受分。让我们在提交之前使用 autoform 的 **钩子** 来转换这些数据：

```js
# /products/client/create_product.coffee

AutoForm.addHooks "insert_product",
  formToDoc: (product) ->
    product.price = product.price * 100

    product

  docToForm: (product) ->
    product.price = product.price / 100

    product
```

Autoform 有一个很长的钩子列表，但到目前为止，对于转换来说最有用的是一个 `formToDoc` 钩子。这个函数在每次表单被转换为文档以进行处理时都会运行，这包括在错误处理之前。我们不使用“before hook”，因为这个钩子将在错误处理之后运行。此外，我们正在使用一个 `docToForm` 钩子来确保在代码重新加载后我们的 `price` 字段保持完整。Autoform 有以下钩子：

```js
before:
  insert:
  update:
  update-pushArray:
  method:
  method-update:
  normal:
after:
  insert:
  update:
  update-pushArray:
  method:
  method-update:
  normal:
onSubmit:
onSuccess:
onError:
formToDoc:
formToModifier:
docToForm:
beginSubmit:
endSubmit:
```

通过这种方式，我们可以快速创建和修改表单，使其符合我们的喜好，同时保持可用性。远离需要数组的字段！在我们的应用程序中，它们很难管理，并且总可以用我们模型中的新集合来表示。

# 加载数据

如果我们没有正确加载数据，我们的前端性能可能会受到影响，因为它可能会使 DOM 快速多次重绘。如果计算复杂，那么我们需要等待所有数据都可用，并在一次单独的扫描中计算一切。如果我们不等待数据，那么计算将在数据接收时运行，这反过来又会导致 DOM 非常快地多次重绘。

为了解决这个问题，我们可以轻松地检查我们的订阅者是否已准备好，并在他们准备好之前显示一个加载符号。

## 设计加载指示器

我们首先创建一个加载模板。让我们使用 font awesome 来处理我们的动画，并确保我们可以轻松地更改加载器的颜色：

```js
//- /loader/loader.jade

template(name="loader")
  div#loader.template
    div(class="{{color_class}}").vertical-align.text-center
      i.fa.fa-5x.fa-cog.fa-spin

// /loader/loader.styl

#loader
  height:100%
```

在这种情况下，我们使用 `fa-spin` 类来使齿轮旋转，并添加了一个 `color_class` 辅助类，它将接受 Bootstraps 的 `text-` 类来定义颜色。同时，我们确保加载器填充了它所持有的内容的全部，这样当我们使用它时，一切都能整齐对齐。

## 实现加载指示器

让我们为我们的产品列表实现加载指示器。由于我们将要实现的方式，我们很容易让我们的界面为每个产品或产品列表显示加载指示器。控制后者将更有效率。

因此，让我们修改我们的产品模板：

```js
//- /products/client/products.jade

template(name="products")
  div#products.template
    header#promoter
      ...

    section#features
      ...

    section#featured_products
      div.container
        div.row
 if Template.subscriptionsReady
            each products
              +product
          else
            div(style="height:160px;")
              +loader color_class="text-primary"

    br
```

Meteor 核心提供了一个特殊的 `Template.subscriptionsReady` 辅助函数，用于检查在该模板实例中创建的订阅是否都处于就绪状态。这是使用我们 `Template.created` 函数中的 `@subscribe` 而不是 `Meteor.subscribe` 的隐藏优势之一。

在这种情况下，我们只在 `products` 变量上运行 `each` 迭代，直到所有订阅都准备就绪。在此之前，我们使用我们的主色调渲染 `loader` 组件。

这种模式的优点在于我们可以精确控制加载器想要放置的位置以及它何时出现。

# 动画和过渡

网页动画是一个庞大且仍在增长的领域，如果你想要它变得复杂，它确实可以变得非常复杂。然而，最终，高效的动画才是真正能产生影响的。你如何使动画高效？你让它保持简单，并让浏览器为你进行动画处理。远离使用 JavaScript 来识别位置和改变颜色。过于依赖 JavaScript 的动画将始终有糟糕的性能。这种糟糕的性能发生是因为 JavaScript 无法在所有设备上有效运行，因此，在不需要使用视频内存的情况下，DOM 的多次更改会在设备允许的最大速度下发生。

Canvas 是一个设计用来通过 JavaScript 和 WebGL 渲染变化的 HTML 元素。通过 canvas 动画任何内容都能提升性能，但它更适合那些想要进行图形密集型工作，如 3D 渲染或游戏精灵动画的应用程序。在这本书中，我们不会涉及 canvas，因为你可能永远不需要它。

忽略 canvas 的强大功能，我们只剩下两个 CSS 工具来制作丰富的动画：`animation` 和 `transition`。`animation` 属性旨在用于需要关键帧的复杂动画，而 `transition` 属性旨在用于简单的状态转换动画。让我们先看看每个属性的例子，以了解它们是如何工作的，然后我们将把它们应用到 Meteor 中。

## 使用 CSS 进行动画

`transition` 属性是最容易理解的属性。你基本上在 CSS 属性中定义你将要动画化的属性以及如何进行动画化：

```js
.box
  transition: <CSS property> <duration> <timing function> <delay>
```

重要的是要意识到，由于 **Stylus Nib**（此包中包含的另一个有用的 Stylus 插件），我们不必担心这个特定属性的供应商前缀。Nib 会自动为我们处理这个问题。通过将过渡属性添加到 `box` 类中，我们告诉浏览器对定义的属性上发生的任何变化进行动画处理。这些变化是如何发生的呢？通过新的类。让我们看一个小例子：

```js
.box
  transition: opacity 300ms ease-in
  opacity: 0
  .in
    opacity: 1
```

通过这组 CSS 规则，我们使盒子在 `.in` 类动态添加之前不可见。一旦添加了类，DOM 元素就会淡入。可用的时序函数有：`ease`、`linear`、`ease-in`、`ease-out`、`ease-in-out`、`step-start` 和 `step-end`。

`animation` 属性稍微复杂一些，因为它不需要动态类来激活动画：

```js
.box
  animation:
    <animation name>
    <duration>
    <timing function>
    <delay>
    <direction>
    <iteration count>
    <fill mode>
    <play state>

@keyframes <animation name>
  0%
    background:blue
  50%
    background:green
  100%
    background:red
```

如您所见，过渡和动画之间的区别在于你可以对动画有多少控制力。通过使用 `animation name` 和 `@keyframes` 选择器，我们可以控制动画的哪个点我们想要改变属性以及如何改变。你可以将关键帧上的数字更改为你想要的任何数字。

注意到 `animation` 属性有几个新的参数：`迭代次数`、`填充模式`和`播放状态`。`迭代次数`参数定义了动画将要播放的次数；你可以将其设置为无限或一个特定的数字。`填充模式`定义了当元素不在动画状态时你希望元素处于的状态：`forwards` 将值设置为最后一个定义的关键帧，`backwards` 设置为第一个定义的关键帧，两者都设置为 `first` 和 `last`，而 `none` 将不执行任何操作（默认）。`播放状态`控制动画是否播放（暂停或运行）。

如果需要，我们可以使用第二个 CSS 选择器来控制我们的 `play state`，就像我们为 `transition` 属性所做的那样。

## 在 Meteor 中执行动画

在 Meteor 中，使用助手控制渲染到 DOM 上的类非常容易，因此我们可以快速注入一个将触发我们的 CSS 动画的类。当我们想要在 DOM 中不存在的元素上触发动画时，我们会遇到问题。

例如，一旦我们的产品加载完成，它们就会突然出现。如果我们想使列表淡入，而加载器淡出呢？这就是 Meteor 的隐藏的 `@_uihooks` 函数发挥作用的地方。

在撰写本文时，此功能处于测试阶段，并且仍然有些问题。处理由此引起所有问题的最佳方式是通过 `percolate:momentum` 包。这样可以避免理解 `uihooks` 当前存在的问题以及如何绕过它们。

`momentum`包设计用来通过使用在`Template.rendered`函数上下文中提供的`@_uihooks`函数来拦截 Blaze。它非常容易使用，并且与 VelocityJS 一起打包，这是一个处理过渡相当有效的 jQuery 插件。我们可以使用两者的组合来帮助保持事情简单。

要自定义动量，我们需要注册处理我们 DOM 中可能发生的三个钩子的动量插件：`insertElement`、`removeElement`和`moveElement`。

| `insertElement` | 当一个 DOM 元素被创建时发生 |
| --- | --- |
| `removeElement` | 当一个 DOM 元素被销毁时发生 |
| `moveElement` | 当一个 DOM 元素从其 DOM 位置移动时发生（排序可能会触发此事件） |

让我们构建一个名为`fade-fast`的插件，该插件将控制产品加载指示器的淡入和淡出：

```js
# /_globals/client/momentum/fade-fast.coffee

Momentum.registerPlugin 'fade-fast', (options) ->
  insertElement: (node, next) ->
    $(node)
      .addClass "animate opacity invisible"
      .insertBefore(next)

    Meteor.setTimeout ->
        $(node).removeClass "invisible"
      ,250

  removeElement: (node) ->
    $(node).velocity opacity:0, 250, "easeOut", ->
      $(this).remove()
```

我们使用`node`变量来修改将要注入 DOM 的元素，并且我们总是使用`insertBefore`方法，将`next`变量作为参数来渲染。请注意，在这种情况下，当我们插入时，我们是在元素渲染到 DOM 之前添加三个类，然后在 250 毫秒后移除不可见类。

当一个元素被移除时，我们将使用`velocity`来修改`opacity`属性，并在恰好 250 毫秒内将其降至`0`。Velocity 接受一个回调命令，我们将使用它最终从 DOM 中移除元素。

现在我们可以添加我们的样式并将助手放置在我们的视图中：

```js
//- /products/client/products.jade

  ...

    section#featured_products
      div.container
        div.row
 +momentum(plugin="fade-fast")
            if Template.subscriptionsReady
              each products
                div.col-xs-12.col-sm-4.col-md-3
                  +product
            else
              div(style="height:160px;")
                +loader color_class="text-primary"

// _globals/client/main.styl

.vertical-align
  ...

.animate
  &.opacity
    transition: opacity 500ms
    opacity:1

    &.invisible
      opacity:0
```

注意，我们添加了一个`momentum(plugin="fade-fast")`组件，它定义了将要渲染具有动画的元素的区域的插件。然后我们创建将使用 CSS 过渡使每个 DOM 元素淡入的类。

# SEO

Meteor 有一个大问题。它默认不支持服务器端渲染。这意味着爬虫不知道如何解析我们的页面，因为服务器没有渲染页面，而是客户端！有许多方法可以解决这个问题。你可以尝试使用`meteorhacks:ssr`来启用服务器端渲染，或者你可以让一个免费的服务为你自动化这个过程。我们将使用一个服务来简化事情。

## Prerender.io

认识一下 prerender.io，这是获取网站解析的最简单方法。Prerender.io 主要是一个免费服务，它确切地知道如何解析你的页面。它基本上导航到你的`webapp`并解析每个页面，然后当爬虫击中我们的`webapp`时，我们的服务器将从 prerender.io 获取解析后的页面，并用这个页面进行响应。

对于页面数量最少且被爬虫访问的 web 应用，该服务是免费的。你拥有的页面越多，动态性越强，价格就越高，但除非你正在构建一个高度动态的公共网页，否则这种情况不太可能。即使如此，`prerender.io`项目可以下载并个人设置（因此仍然是免费的）。

我们已经安装了`dfischer:prerenderio`包来轻松设置服务。我们唯一剩下的事情就是进行授权。首先，前往[www.prerender.io](http://www.prerender.io)并创建一个账户，然后点击他们导航栏上的**安装令牌**标签，并复制您的令牌。现在将您的令牌粘贴到配置文件下：

```js
# /_globals/server/prerenderio.coffee

prerenderio.set "prerenderToken","<yourtoken>"
```

简单！您的网站现在可被爬取。现在让我们配置我们的路由器以与预渲染一起工作，并添加一个 404 页面：

```js
# /router/config.coffee

if Meteor.isClient
  Template.created ->
    except = [
      "Template.__dynamicWithDataContext"
      "Template.__dynamic"
      "Template.layout"
      "Template.layout"
      "body"
    ]

    unless _.contains except, @view.name
      window.prerenderReady = false

      if @subscriptionsReady()
        window.prerenderReady = true

FlowRouter.notFound =
  action: ->
    FlowLayout.render "layout",
      content:"not_found"
```

注意，我们正在创建一个全局的`Template.created`钩子函数，并将`prerenderReady`默认设置为`false`。然后，我们使用`@subscriptionsReady()`检查我们的订阅是否就绪，并将`prerenderReady`设置为`true`。重要的是要理解`@subscriptionsReady()`只检查在模板中通过`@subscribe`创建的订阅！

此外，请注意，我们正在创建一个异常列表。这是为了确保我们只检查需要它们的模板上的订阅者。

这种模式确保预渲染在实际上已加载所有数据之前不会缓存页面。为什么不使用`FlowRouter`全局触发器呢？遗憾的是，这些触发器无法访问正在渲染的模板实例（因为可能有多个）。如果您需要更多控制，您可以按路由/模板设置此选项。

让我们着手处理我们的 404 页面模板：

```js
//- /router/client/not_found.jade

template(name="not_found")
  div#not_found.template
    div.text-center.vertical-align
      h3 404!
      h5 Page not found!
```

为了确保我们的 404 页面打印出正确的 404 响应，我们不得不使用`Meta`。

## 使用 Meta

要控制预渲染，我们需要使用`yasinuslu:blaze-meta`包。这个包基本上向我们的`head` HTML 标签注入一个模板，这样我们就可以轻松地设置标签。预渲染使用标签来识别页面是否应该响应 404。我们还可以使用 Meta 来设置我们的标题、描述、关键词和 robots 属性。让我们先从让我们的 404 页面响应 404 开始：

```js
# /router/client/not_found.coffee

Template.rendered "not_found", ->
  Meta.set [
    {
      name:"name"
      property:"prerender-status-code"
      content:"404"
    }
    {
      name:"name"
      property:"robots"
      content:"noindex, nofollow"
    }
  ]
```

`Meta`对象只有四个函数：`set`、`setTitle`、`unset`和`config`。您不太可能使用`unset`，因为元标签不会通过路由持久化。`set`和`unset`函数都需要一个对象数组，并且每个对象都需要所有三个键：`name`、`property`和`content`。

在前面的例子中，我们将`prerender-status-code`设置为`404`，将`robots`设置为`noindex, nofollow`。通过这种方式，我们确保我们的 404 页面实际上返回了一个 404 错误，并且爬虫不会索引该页面。

此外，我们还需要配置`Meta`，以确保页面的名称始终正确：

```js
# /router/config.coffee

if Meteor.isClient
  Meta.config
    options:
      title:"Crashing Meteor"
      suffix:""

  ...
```

由于`Meta`控制着我们的`title`标签，我们需要从`layout.jade`中移除该标签。

## Schema.org

现在既然我们的网站可被爬取，我们希望我们的产品在搜索引擎中脱颖而出。让我们升级我们的产品以启用丰富片段。丰富片段遵循 schema.org 标准为搜索引擎爬虫（如 Google）构建结构化数据。有了这些信息，搜索引擎可以在其网站上更突出地展示您的数据，使搜索更有效，甚至可以在其他服务中使用它。遵循他们的指南是个好主意，因为 Google 使用这些指南来指导其爬虫。

要做到这一点，我们只需要在我们的产品页面上设置尽可能多的 schema.org 属性。访问[`schema.org/Product`](http://schema.org/Product)获取可用属性的完整列表。最常用的属性包括：

| 属性 | 类型 | 描述 |
| --- | --- | --- |
| `name` (必需) | `Text` | 产品的名称 |
| `image` | `URL` | 产品链接 |
| `description` | `Text` | 产品描述 |
| `offers` | `OFFER OBJECT` | 这包括多个定义对象价格的参数 |
| `price` (必需) | `Number` | 产品的价格 |
| `priceCurrency` (必需) | `Text (ISO 4217) [ex: USD]` | 价格的货币 |

现在让我们通过使用`MicroData`规范来实践：

```js
//- /products/client/product.jade

template(name="product")
  article#product(itemscope itemtype="http://schema.org/Product")
    h5(itemprop="name") {{name}}

    div.offer(itemprop="offers" itemscope itemtype="http://schema.org/Offer")
      meta(itemprop="priceCurrency" content="USD")
      span.price(itemprop="price") {{format.money price}}

    button.add-to-cart.btn.btn-block.btn-primary Add 1 to Cart

    button.modify-quantity.btn.btn-block.btn-info Add more to Cart
```

如您所见，要使用`MicroData`规范，您需要向您的视图添加各种自定义属性。在大多数情况下，每次您要声明某个特定事物时，您都需要通过`itemscope`和`itemtype`属性来表示这一点。当我们想要声明此声明下的属性时，我们需要确保我们的 DOM 元素是此声明的子元素，并且属性是通过`itemprop`属性声明的。

# 摘要

本章介绍了几个有用的模式，使得实现丰富和最小化界面变得容易。首先，我们学习了如何结合使用 Twitter Bootstrap 框架、Jeet 和 Rupture；这改善了我们的 DOM 组织方式。我们还学习了如何创建超级助手——可以在视图和控制器中调用的函数。

我们深入研究了我们可以使用的不同类型的变量，以保持我们的代码安全并创建丰富的界面。我们还学习了如何使用 Meteor 方法和使用 autoform 轻松保存信息。我们介绍了如何实现简单的加载指示器以及如何在 Meteor 中实现动画。最后，我们学习了如何使用 prerender.io 和`Meta`包实现一点 SEO。

下一章将涵盖应用范围内的模式。第四章，*应用模式*将教会我们如何过滤和分页浏览集合，如何全面保护我们的应用程序，以及如何与外部 API 集成。
