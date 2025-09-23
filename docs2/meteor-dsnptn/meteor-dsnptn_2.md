# 第二章。发布和订阅模式

这本书到目前为止最重要的章节。我们控制发布者和订阅者的方式将决定我们的应用程序在生产中的响应速度有多快。发布者和订阅者是我们数据库和客户端之间的链接。服务器使用发布者向客户端发布信息，而客户端通过订阅发布者来从发布者请求信息。这一切都通过`Meteor.publish`和`Meteor.subscribe`函数来管理。我们应该能够根据两个目标产生客户端想要看到的数据：

+   减少服务器的压力

+   只发送客户端需要的信息

本章将教会你你可以用来实现每个构建的模板的这些目标的不同模式。以下是我们将涵盖的主题概述，以了解这些模式：

+   模板级别的订阅

+   数据库关系

+   带关系的发布

+   聚合发布者

+   外部 API 发布者

# 模板级别的订阅

这种模式将`Meteor.subscribe`函数附加到模板上。从模板订阅的关键优势是能够隔离模板，并知道它在渲染时仍然有效。

许多 Meteor 开发者将他们的订阅方法附加到他们的路由上。这意味着模板只有在特定路由上才会渲染正确的数据。

使用这种模式，我们将能够在任何地方重用模板，而不用担心数据问题。

## 为在线商店设置产品

让我们先为我们的`online_shop`项目在 MongoDB 中设置一个`Products`集合。在第一章，*开始使用 Meteor*中，我们了解到我们需要将这个定义放在`/globals/lib/collections`目录下：

```js
# /globals/lib/collections/products/products.coffee
@Products = new Mongo.Collection "products"

# fields:
#  name
#  description
#  sku
```

重要的是要注意，我们在`Products`变量开头添加了`@`符号。这编译成`this.Products`。在 Meteor 的 CoffeeScript 中，我们这样做是为了定义一个全局作用域的变量。这意味着`Products`变量现在存在于我们将要创建的每个 CoffeeScript 文件中。

我们还需要添加一个权限文件，这样我们才能从控制台修改集合。`allow`/`deny`函数是给集合添加安全层的规则。如果一个`allow`规则对某个操作返回`true`，它将允许更改通过。目前，我们将设置所有规则以允许一切。当我们查看第四章，*应用模式*时，我们将处理权限问题。

```js
# /globals
# ./lib/collections/products/server/products_permissions.coffee
Meteor.startup ->
  Products.allow
    insert: -> true
    update: -> true
    remove: -> true
```

我们使用`Meteor.startup`函数确保在设置`allow`/`deny`规则之前，我们已经将`Products`设置为集合。现在我们有了集合，让我们创建一个着陆页面来显示产品列表。让我们首先在`/products/client`目录中构建**视图**：

```js
//- /products/client/products.jade
template(name="products")
  h3.text-center products
```

这个视图只是一个占位符。我们总是在实际开始使用出版商和订阅者之前创建模板对象。

## 构建出版商

让我们使用`Meteor.publish`函数构建一个简单的出版商用于我们的视图。这个出版商将只向订阅客户端发送 10 个文档。我们将在第四章中讨论分页，*应用模式*。

```js
# /products/server/products_pub.coffee
Meteor.publish "products_pub", ->
  Products.find {},
    limit:10
```

## 订阅出版商

这里魔法开始了。我们将使用`Template.<template>.onCreated`函数来订阅我们的出版商。每当在 DOM 中创建模板实例时，`onCreated`函数就会运行。因此，如果我们在这个函数中放置一个`Meteor.subscribe`函数，这将自动在模板被使用时重新订阅。让我们试一试：

```js
# /products/client/products.coffee
Template.products.onCreated ->
  @autorun =>
    @subscribe "products_pub"
```

```js
onCreated hooks. This means that if we were to define a second onCreated function, the first function will not run. We change this behavior using the xorax:multiple-callbacks package. This package, basically, concatenates onCreated, onRendered, and onDestroyed functions so that they cannot be overwritten. Let's take a look:
```

```js
# /products/client/products.coffee
Template.created "products", ->
  @autorun =>
    @subscribe "products_pub"
```

我们只需要更改代码的第一行。这样做是为了我们可以将多个函数附加到`onCreated`模板钩子而不会覆盖任何其他函数。

# 数据库关系

我们的所有集合都将以某种方式与数据库中的其他集合相关联。这个主题将探讨你在设计数据库时应考虑的三种不同类型的关系。

我们数据库的最终形状定义了我们的出版商将是什么样子。如果你的数据都混合在只有一个或两个不同的集合中，你很快就会发现自己难以筛选数据。如果你的数据在多个集合之间分布得太广，代码在长期内将难以维护。那么这个问题的解决方案是什么？

解决数据库关系的方案是理解数据将在客户端如何使用、将修改多频繁以及集合可以有多大。

让我们构建我们其余的集合，以了解良好的关系看起来像什么。

## 一对一

一对一数据库关系描述了集合中的一个 MongoDB 文档将如何仅与另一个集合中的 MongoDB 文档相关联。你可以将其想象为另一个对象内部的 JavaScript 对象。

在 Meteor 中，你应该为那些你将很少使用、具有独特接口或你将使用而不一定需要访问产品的字段子集创建一对一关系。有了这种关系，构建一个仅发送`master_image`的图片上传器和图片拼贴将变得非常容易。

首先，添加一个将处理我们产品图片的集合：`ProductImages`。我们将假设我们的前端将具有多种类型的图片，每种图片都将展示在我们构建的界面中的不同部分：

```js
# /lib/collections/product_images/product_images_collection.coffee
@ProductImages = new Mongo.Collection "product_images"

# fields:
#  product
#  master_image
#  side_image
#  front_image
#  top_image
#  cart_image

# /lib/collections/
# ./product_images/server/product_images_permissions.coffee
Meteor.startup ->
  ProductImages.allow
    insert: -> true
    update: -> true
    remove: -> true
```

`ProductImages` 集合将包含一个名为 `product` 的字段。该字段通过保存来自 `Products` 集合的唯一 `_id` 字段来建立我们的关系。这意味着每次我们想要发布带有 `ProductImages` 的产品时，我们还需要通过在 `ProductImages` 中搜索 `Product_id` 来查询数据库。所以一个辅助器可能看起来像这样：

```js
# COFFEESCRIPT
Template.landing.helpers
  "images": ->
    ProductImages.findOne product:@_id
```

## 一对多

一对多数据库关系描述了在一个集合中的一个 MongoDB 文档将如何与另一个集合中的多个 MongoDB 文档相连接。你可以将其想象为一个存在于特定字段下的 JavaScript 对象数组，但数据如此复杂，以至于你需要将其分离。

现在让我们创建一个 `Orders` 集合。这个集合将作为一个购物车。虽然你的第一反应可能是创建一个 `Carts` 集合，但你很快会发现你正在重复订单信息（一个用于购物车，一个用于订单，一旦下单）。我们可以通过添加一个 `status` 字段来轻松识别订单是否为新的：

```js
# /lib/collections/
# ./orders/orders_collection.coffee
@Orders = new Mongo.Collection "orders"

# fields
  # status ("new","pending","complete")
  # total_products
  # subtotal
  # tax
  #   rate
  #   amount
  # discounts
  #   discount
  #   amount
  # total
  # date_created
```

我们不要忘记为集合添加权限。每次我们创建一个新的集合时，我们都需要这样做，以确保集合免受黑客攻击并确保客户端可以相应地修改集合。我们将在第四章的 *安全* 部分中对此进行更多介绍，*应用模式*。

```js
# /lib/collections/
# ./orders/server/orders_permissions.coffee
Meteor.startup ->
  Orders.allow
    insert: -> true
    update: -> true
    remove: -> true
```

这里有问题，产品没有在集合中定义！我们故意创建了一个将具有与订单详情一对多关系的订单摘要集合。我们不知道订单实际上可能有多广泛。如果我们包含一个包含产品数组的字段，这个列表不仅难以管理，而且可能变得足够大，以至于会崩溃数据库。

让我们把订单详情放在一个名为 `OrderDetails` 的单独集合中：

```js
# /lib/collections/
# ./order_details/order_details_collection.coffee
@OrderDetails = new Mongo.Collection "order_details"

# fields
  # order
  # product
  # price
  # quantity
  # subtotal
  # tax
  #   rate
  #   amount
  # total
  # discounts
  #   discount
  #   amount

# /lib/collections/
# ./order_details/server/order_details_permissions.coffee
Meteor.startup ->
  OrderDetails.allow
    insert: -> true
    update: -> true
    remove: -> true
```

完美！现在我们可以这样说，一个 `Order` 有多个 `OrderDetails`。在这种情况下，`OrderDetails` 集合中的每个文档代表一个单独的产品以及该产品在订单中的详细信息。我们已经添加了 `order` 字段来识别这些详情（或在这种情况下，产品）属于哪个特定的订单。

这是一个优秀的设计。通过将细节与顺序分开，我们能够精确控制服务器发送给客户端的数据。记住，这里的目的是尽可能发送最少的可能数据给客户端，以便网站快速加载并减少服务器压力。管理插入和更新也变得更容易。由于我们不需要处理细节数组的索引，我们可以简单地使用 ID 来查找和操作数据。现在当商店管理员订阅`Orders`集合时，我们可以这样操作，即服务器只发送管理员需要查看订单的数据。点击订单后，就会订阅仅该订单的详细信息。

## 多对多

多对多数据库关系描述了一个集合中的 MongoDB 文档如何链接到另一个集合中的多个 MongoDB 文档。你可以将其想象为一个表，其中你重复信息，并且对于每一行只更改一个字段的值，这样你就可以通过该行过滤信息。

这类关系需要**映射表**。在 MongoDB 的情况下，映射表是一个单独的集合。映射表是每行重复信息但不实际重复条目的部分；它只是将每个集合 ID 配对。

在我们即将编写的示例中，我们想要在产品和标签之间创建一个多对多关系，因为一个产品可以有多个标签，而一个标签也可以有多个产品。在这种情况下，映射表将保存每个 ID 对。所以如果一个产品有两个标签，映射表中将有两个条目，具有相同的产品 ID，但每个条目都有一个不同的标签 ID。如果一个标签是两个产品的组成部分，那么映射表将有两个条目，具有相同的标签 ID，但每个条目都有一个不同的产品 ID。

重要的是要注意，由于它们的复杂性，这类关系往往被忽视。如果你发现自己试图通过每个集合的数组来匹配两个集合，那么你肯定是在尝试一个“多对多”关系。

为了解释这种关系，我们将为我们的产品创建一个`Tags`集合：

```js
# /lib/collections/
# ./tags/tags_collection.coffee
@Tags = new Mongo.Collection "tags"

# fields
#  name

# /lib/collections/
# ./tags/server/tags_permissions.coffee
Meteor.startup ->
  Tags.allow
    insert: -> true
    update: -> true
    remove: -> true
```

你会注意到，在这种关系中，`Tags`集合没有与产品关联的内容，但如果它有，它将是一个包含标签所属产品的数组。为了建立这种关系，我们将创建映射表，即`ProductsTags`集合：

```js
# /lib/collections/
# ./products_tags/products_tags_collection.coffee
@ProductsTags = new Mongo.Collection "products_tags"

# fields
#  product
#  tag

# /lib/collections/
# ./tags/server/products_tags_permissions.coffee
Meteor.startup ->
  ProductsTags.allow
    insert: -> true
    update: -> true
    remove: -> true
```

这个集合允许我们在`Products`和`Tags`之间有任意的组合关系。假设我们想查看与一个标签相关的所有产品。在这种情况下，我们首先查询该标签的映射表，然后使用结果查询标签。

小心！多对多关系可能难以发现。如果在数据库设计过程中遇到任何困难，不要忘记考虑这种可能性。

# 带关系的发布

我们理解了我们的集合是如何相关的，但我们如何能够轻松地发布带有这些关系的数据？

在 Meteor 中，由于反应性和发布者的工作方式，发布关系可能会出现问题。你可能会期望通过简单地向两个相关集合查询并返回一个数组就能发布一个完美的反应性集合。但这并不是事实。`Meteor.publish`函数在依赖项改变时不会重新运行。这意味着如果关系断开，相关的文档将保持发布状态，或者更糟糕的是，如果另一个客户端创建了新的关系，相关的数据将不会发布。

为了处理 Meteor 中的数据库关系和反应性，我们使用`lepozepo:publish-with-relations`包。当关系断开时，此包会自动以最有效的方式订阅新数据。如果你熟悉 MySQL，这个包使得 JOIN 操作变得简单。

## 发布带有图片的产品（一对一）

我们将使用`/products`目录，这个模块将是我们的首页。首先，让我们为我们的视图设置一个路由：

```js
# /products/products_route.coffee
FlowRouter.route "/",
  name:"products"
  action: ->
    FlowLayout.render "layout",
      content:"products"
```

我们正在使用`FlowRouter.route`函数来定义我们首页的路径，以及`FlowLayout.render`函数来定义要使用哪个布局模板。你会注意到`FlowLayout.render`函数接收两个参数：第一个参数定义要渲染哪个布局模板，第二个参数定义在布局模板中渲染的位置。

现在我们可以开始处理发布者了。目标是发布 10 个产品，只配对它们的`master_image`：

```js
# /products/server/products_pub.coffee
Meteor.publish "products", ->
  @relations
    collection:Products
    options:
      limit:10
    mappings:[
      {
        key:"product"
        collection:ProductImages
      }
    ]

  @ready()
```

注意我们通过`@relations`函数传递的对象中的`mappings`键。数组内的所有对象都必须至少有一个`collection`键。可选的，它们可以包含`key`和`foreign_key`。在这种情况下，我们使用`key`来表示`Products`集合中的`_id`字段等于`ProductImages`集合中`product`字段的字符串。这是发布数据最有效的方式。该包将自动确保所有集合的变化都能实时反映。

`Meteor.publish`函数有一个特性：当数据发生变化时，光标会相应地反应，但持有光标的函数不会。当涉及到创建关系时，这种影响是明显的。让我们看看不使用包来处理我们的关系时，我们的代码会是什么样子：

```js
# DO NOT CODE THIS INTO YOUR PROJECT
products_cursor = Products.find {},limit:10

# Make products an array with all the _ids
products = products_cursor.map (product) ->
  product._id

# Find their images
images_cursor = ProductImages.find
  product:
    $in:products

# Return the cursors in an array
[products_cursor,images_cursor]
```

在这个例子中，假设由于某种原因`Products`集合的顺序发生了变化。这将使`products_cursor`按预期在客户端反应性地改变，但由于`ProductImages`集合没有变化，并且`Meteor.publish`在依赖项发生变化时不会重新运行，因此新发布产品的图片将不会反应性地发布！

## 发布带有详细信息的订单（一对多）

现在我们将处理我们的`Orders`和`OrderDetails`集合。让我们为创建订单的功能设置模板、路由和订阅者。我们可以将此模块命名为`cart`并将其存储在`orders`目录下：

```js
# /orders/cart/client/cart.jade
template(name="cart")
  h3.text-center cart

# /orders/cart/client/cart.coffee
Template.created "cart", ->
  @autorun =>
    order = Session.get "cart.order"
    @subscribe "cart",
      order:order
```

注意，`order`变量将通过使用`Session`变量来获取我们的订单`_id`。我们这样做是因为`Session`变量是响应式的；如果由于某种原因值发生变化，这将确保订阅者重新运行。订阅者重新运行是因为变量是在`@autorun`函数内定义的。

此外，我们正在将一个对象作为参数传递给`@subscribe`函数，以便发布者知道我们正在谈论哪个订单：

```js
# /orders/cart/cart_route.coffee
FlowRouter.route "/cart",
  name:"cart"
  action: ->
    FlowLayout.render "layout",
      content:"cart"
```

我们需要发布三个不同的集合以获取购物车所需的所有信息：`Orders`、`OrderDetails`和`Products`。此发布者可以遵循我们在第一个发布者中找到的相同逻辑：

```js
# /orders/cart/server/cart_pub.coffee
Meteor.publish "cart", (ops={}) ->
  if ops.order and not _.isEmpty ops.order
    @relations
      collection:Orders
      filter:
        _id:ops.order
        status:"new"
      mappings:[
        {
          key:"order"
          collection:OrderDetails
          options:
            limit:25
          mappings:[
            {
              foreign_key:"product"
              collection:Products
            }
          ]
        }
      ]

  @ready()
```

此发布者通过在参数中传递`ops={}`来预定义函数的默认值。在确保`ops.order`存在且不是空字符串后，我们建立关系。我们需要确保`Order`的状态具有`"new"`值以进行安全起见，所以我们将其硬编码在`filter`键中。

注意，在这种情况下，我们正在使用`foreign_key`。这表明`OrderDetails`集合有一个包含与`Products`集合的`_id`字段相等的字符串的`product`字段。

目前，我们将`OrderDetails`限制为`25`。

## 发布带有产品的标签（多对多）

我们绝对希望能够通过标签过滤我们的产品。我们可以遵循前一个主题中的相同模式。让我们修改我们的`products`模块。

首先，我们的订阅者需要反应性地将一个标签 _id 数组转换为数组。我们可以现在使用一个`Session`变量来设置一个标签 _id 数组，这样我们就可以轻松地从控制台直接修改它：

```js
# /products/client/products.coffee
Template.created "products", ->
  @autorun =>
    # tags is an array of tag _ids
    tags = Session.get "products.tags"
    filter = {}

    if tags and not _.isEmpty tags
      _.extend filter,
        tags:tags

    @subscribe "products", filter
```

我们使用下划线的`_.extend`函数确保如果存在`tags`，则`filter`变量有一个`tags`键。现在我们的发布者将变得更加复杂：

```js
# /products/server/products_pub.coffee
Meteor.publish "products", (ops = {}) ->
  limit = 10

  if ops.tags and not _.isEmpty ops.tags
    @relations
      collection:Tags
      filter:
        _id:
          $in:ops.tags
      mappings:[
        {
          collection:ProductsTags
          key:"tag"
          mappings:[
            {
              collection:Products
              foreign_key:"product"
              options:
                limit:limit
              mappings:[
                {
                  collection:ProductImages
                  key:"product"
                }
              ]
            }
          ]
        }
      ]

  else
    @relations
      collection:Products
      options:
        limit:limit
      mappings:[
        {
          key:"product"
          collection:ProductImages
        }
      ]

  @ready()
```

虽然代码可能看起来很长，但逻辑很容易理解。首先，我们使用我们在订阅者中定义的`tags`数组过滤了`Tags`集合。然后，我们使用`key`和`foreign_key`的组合链式关系。在这种情况下，映射表（`ProductsTags`）所服务的函数是清晰的。

## 关键，外键，选项和过滤器

从`lepozepo:publish-with-relations`包中理解的核心概念是`@relations`函数中提供的选项。

`key`和`foreign_key`默认为`_id`字段。`key`始终引用集合内的字段，而`foreign_key`始终引用父集合的字段。

`options` 和 `filter` 选项等价于 Meteor - MongoDB 查询的第二个和第一个参数（分别）：`Products.find(filter,options)`。

有许多包的工作方式类似于 `lepozepo:publish-with-relations`，我们在这本书中没有探讨它们，但它们确实值得留意：`reywood:publish-composite`、`lepozepo:reactive-publish` 和 `cottz:publish-relations`。我发现最后一个包是其中最好的，因为它使用简单，并迫使开发者创建更智能的数据库关系。

# 聚合发布者

有时我们的数据库有大量的信息，我们希望综合起来。一些开发者选择将所有信息发布到客户端，并让客户端进行综合。正如我们迄今为止所学的，这可能会对性能产生负面影响。其他开发者可能会使用 `Meteor.method` 来返回综合数据。这当然对客户端更好，但如果计算量很大，这将对我们的服务器造成负担。

处理这类问题的最佳方式是使用 MongoDB 的聚合框架将计算的繁重工作转移到我们的数据库上，然后我们可以将结果与 `Meteor.publish` 特殊函数配对：`@added`、`@changed` 和 `@removed`。

## 聚合框架

MongoDB 的聚合框架使用管道的概念来处理数据。基本上，管道是一系列 Mongo 将要遵循的步骤，以生成您所需的数据。

我们通过添加 `meteorhacks:aggregate` 来安装了对聚合框架的支持。这使我们能够访问框架的所有服务器端命令。您最终会使用最常用的命令是 `$match`、`$group` 和 `$project`。

让我们为我们的 `dashboard` 模块构建一个发布者，并首先构建模板：

```js
# /dashboard/client/dashboard.jade
template(name="dashboard")
  h3.text-center dashboard

# /dashboard/client/dashboard.coffee
Template.created "dashboard", ->
  @autorun =>
    @subscribe "dashboard"

# /dashboard/dashboard_route.coffee
FlowRouter.route "/dashboard",
  name:"dashboard"
  action: ->
    FlowLayout.render "layout",
      content:"dashboard"
```

现在，我们需要构建一个只存在于客户端的集合。我们的服务器将手动与这个集合通信。我们特别将这个集合放在客户端，因为我们不希望服务器将数据记录到数据库中。我们可以将集合附加到 (`/dashboard/client/dashboard.coffee`) 客户端控制器文件中：

```js
# /dashboard/client/dashboard.coffee
@_dashboard = new Mongo.Collection "_dashboard"

Template.created "dashboard", ->
  @autorun =>
    @subscribe "dashboard"
```

由于这个集合是客户端的，我们在名称开头加了一个下划线来区分它。当然，这并不是必需的，但它有助于防止重复的集合名称。

我们的目标是发布 `"pending"` 订单的总计和子总计。我们的管道步骤很简单：

1.  过滤集合以显示状态等于 `"pending"` 的订单。

1.  对每个过滤订单的总计求和：

    ```js
    # /dashboard/server/dashboard_pub.coffee
    Meteor.publish "dashboard", ->
      totals = Orders.aggregate [
          {
            $match:
              status:"pending"
          }
          {
            $group:
              _id:null
              total:
                $sum:"$total"
              subtotal:
                $sum:"$subtotal"
              discount:
                $sum:"$discount.amount"
          }
        ]

      console.log totals
    ```

要使用聚合框架，你使用附加到集合对象的 `.aggregate` 函数；在这种情况下是 `Orders`。此函数仅接受一个数组作为参数——这是因为数组是框架将要遵循的步骤的有序集合。数组中的每个步骤都由一个对象表示，并且总是以一个运算符开始。

在这里，我们决定使用 `$match` 来过滤订单以找到待处理的订单，然后我们使用了 `$group` 来累计 `total` 和 `subtotal` 的值。请注意，`$group` 表达式有一个强制性的 `_id` 键。此键定义了我们想要如何分组集合。通过将 `_id` 键设置为 `null`，我们表示我们希望将集合中的所有文档分组到一个单一的对象中。

`$sum` 是一个累加运算符。当你使用这样的运算符时，你可以通过使用货币符号（`$`）后跟字段名称的字符串来访问文档属性。此外，你可以使用点表示法（`"$discount.amount"`）访问对象中的对象。

`totals` 的结果是包含单个对象的数组，具有 `total`、`subtotal` 和 `discount` 键。

## 发布结果

发布结果比看起来要容易得多。我们只需要使用绑定到 `Meteor.publish` 的 `@added` 函数。基本上，`@added` 函数会通知订阅者已向发布集合中添加了数据：

```js
# /dashboard/server/dashboard_pub.coffee
Meteor.publish "dashboard", ->
  totals = Orders.aggregate [
      {
        $match:
          status:"pending"
      }
      {
        $group:
          _id:null
          total:
            $sum:"$total"
          subtotal:
            $sum:"$subtotal"
          discount:
            $sum:"$discount.amount"
      }
    ]

  if totals and totals.length > 0 and totals[0]
    @added "_dashboard","totals",totals[0]
```

最后两行确保 `totals` 数组存在，如果不存在，则将数组中的第一个对象发布到我们的 `_dashboard` 集合。`@added` 函数有三个必需的参数。第一个参数是集合的名称，第二个是文档的 `_id`，第三个是文档。

注意，这类发布者不是响应式的，这意味着我们不需要添加 `@changed` 或 `@removed` 函数。然而，我们可以更进一步。我们不需要为每个需要聚合发布者的模块创建一个集合，我们可以创建一个主集合来管理我们所有的聚合发布者。

首先，我们删除我们的 `_dashboard` 集合并创建一个新的 `Aggregate` 集合：

```js
# /dashboard/client/dashboard.coffee
Template.created "dashboard", ->
  @autorun =>
    @subscribe "dashboard"

# /globals/lib/collections/aggregate/client/aggregate.coffee
@Aggregate = new Mongo.Collection "aggregate"
```

现在，我们需要修改我们的发布者：

```js
# /dashboard/server/dashboard_pub.coffee
Meteor.publish "dashboard", ->
  totals = Orders.aggregate [
      {
        $match:
          status:"pending"
      }
      {
        $group:
          _id:null
          total:
            $sum:"$total"
          subtotal:
            $sum:"$subtotal"
          discount:
            $sum:"$discount.amount"
      }
    ]

  if totals and totals.length > 0 and totals[0]
    @added "aggregate","dashboard.totals",totals[0]
```

通过这种方式，我们现在可以通过在客户端控制台调用以下函数来访问我们的值：

```js
Aggregate.findOne "dashboard.totals"
```

重要的是要理解，一旦我们离开路由，订阅者将被停止，`aggregate` 集合将被清除。这种默认行为给了这个集合很大的灵活性。

为什么使用发布者与聚合框架比使用 `Meteor.method` 更有效？`Meteor.method` 是设计用来触发服务器中的关键函数并返回简单消息的。另一方面，发布者是设计用来控制数据集的。你很快会发现，发布者比方法更容易控制和优化。

# 外部 API 发布者

应该避免这类发布者。它们从外部服务（如 **Stripe** 或 **Square**）获取原始数据很棒，但它们通常稍微慢一些，因为这意味着需要与另一个服务器进行主动通信。当我们与其他服务器集成时，我们应该始终构建一个单独的同步服务器。我们将在另一章中讨论 API。

尽管如此，这种发布模式在边缘情况下可能很有用，因此了解这个选项的存在很重要。

## HTTP 包

HTTP 是一个协作系统的协议；它是允许用户连接到网页的协议。HTTP 协议可以用来从我们的服务器访问其他服务器上的数据。

我们将使用 Meteor 的集成 HTTP `Request` 模块与 Stripe 的服务器进行通信。我们选择与 Stripe 集成，因为它是一个易于集成且比市场上大多数其他支付处理器更可靠的支付处理器。我们在运行 `meteor add http` 时添加了这个包。此模块具有你期望的所有功能：`.get`、`.post`、`.put` 和 `.del`。对于这个主题，我们只将涵盖 `.get` 函数。

我们的目标是从 Stripe 获取数据。首先在 Stripe 中创建一个免费账户。创建账户后，转到你的仪表板并将其设置为 `"test"`。

现在通过点击 **+ 创建支付** 来创建一个支付。使用以下数据来进行支付：

+   **金额**：`10`

+   **卡号**：`4242 4242 4242 4242`

在我们的测试环境中，我们不需要添加 CVC，Stripe 会自动将到期日期设置为今天起一年。

点击 **账户设置** 并复制你的测试密钥。我们将使用它来授权我们的请求。

我们可以先在 `/dashboard_pub.coffee` 下创建一个新的发布者；我们目前只获取最后三个费用——我们稍后会修改这个设置：

```js
# /dashboard/server/dashboard_pub.coffee
Meteor.publish "dashboard", ->
  ...

Meteor.publish "latest_sales", ->
  @unblock()

  HTTP.get "https://api.stripe.com/v1/charges?limit=3",
    headers:
      "Authorization":"Bearer <TEST SECRET KEY>"
    (error,result) =>
      if not error
        _.each result.data.data, (payment) =>
          @added "aggregate", "dashboard.sales.#{Meteor.uuid()}",payment
      else
        console.log error
```

我们在这里使用的 `@unblock` 函数是通过 `meteorhacks:unblock` 包提供的。它的工作方式与 `Meteor.methods` 中的 `@unblock` 函数相同；这确保了发布者在等待从 Stripe 收到信息时不会阻塞服务器。解除聚合发布者的阻塞至关重要！如果我们不解除发布者的阻塞，那么当用户离开页面时，客户端可能会变得无响应。

你可能自己在想：我们能不能直接在客户端发起 HTTP 请求而不必担心阻塞服务器？不，我们不能。如果你在客户端运行这个函数，你就必须暴露你的密钥，这是一个重大的安全漏洞。

`HTTP.get`函数有三个重要的字段：URL、`options`对象和回调函数。URL 是 Stripe 在其 API 文档中提供的地址——在这种情况下，我们使用了`/charges` URL 并传递了一个参数`limit`来获取最后三笔销售。`options`对象用于传递请求所需的所有信息。在这种情况下，我们将使用`headers`键来设置我们的`Authorization`头。`options`对象可以接受几个键；你可以在[docs.meteor.com](http://docs.meteor.com)找到这些信息。我们的回调函数接收请求的结果。与 Meteor 中的大多数函数一样，它返回两个参数；第一个是一个错误对象，如果没有错误则为未定义，而第二个是实际的结果。

在这种情况下，我们正在寻找的数据包含在`result.data.data`数组中。然后我们可以轻松地使用我们的`@added`函数发布数据。请注意，我们同时绑定了回调函数和`_.each`函数，这样我们就可以访问到`@added`。

让我们订阅我们的新发布者来查看我们的数据：

```js
# /dashboard/client/dashboard.coffee
Template.created "dashboard", ->
  @autorun =>
    @subscribe "dashboard"
    @subscribe "latest_sales"
```

尝试在你的控制台中运行`Aggregate.find().fetch()`来查看结果。

# 摘要

在本章中，我们学习了如何使用模板钩子将订阅者从模板中隔离出来。我们还学习了如何根据我们将如何使用它来优化我们数据库的结构。当涉及到发布者时，我们学习了如何返回每个可能的数据结构而不破坏响应性。我们介绍了聚合发布者，并学习了在发布之前如何合成数据，而不会损害服务器的性能。

在下一章中，我们将介绍一些前端技术，这些技术将帮助我们保持代码的灵活性并使前端看起来很棒。
