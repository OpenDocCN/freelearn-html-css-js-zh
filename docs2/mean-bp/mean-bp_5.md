# 第五章 电子商务应用程序

本章将专注于构建一个类似电子商务的应用程序。我们将通过构建一个包含所有业务逻辑的核心并使用较小的应用程序来消费它来尝试不同的应用程序架构。还有一个有趣的事情要注意，那就是我们的电子商务应用程序的前端商店将使用服务器端渲染。

这种新的架构将使我们能够构建微应用；例如，一个应用可以是管理产品目录的管理应用程序。好处是每个微应用都可以使用不同的方法构建。

作为演示，我们不会在 Angular 中构建我们的前端商店。我知道这听起来很疯狂，但出于教育目的，这将非常棒。此外，我们还想强调构建混合应用程序是多么容易。

应用程序的管理部分将使用 Angular 2 来构建。因此，我们将构建一个无头核心后端服务。这个核心应用程序将被我们的微应用消费。

# 设置基本应用程序

在前面的章节中，我们使用自己的样板文件来启动应用程序的开发。这一章将有一个全新的文件夹结构，但请放心；我们仍然会使用大量现有的样板代码。

新的文件夹结构将给我们更多的灵活性，因为我们已经超出了最初的架构。一个好处，我们不会在本章中介绍，就是您可以将每个模块移动到单独的包中，并将它们作为依赖项安装。

在深入探讨之前，让我们先看看我们架构的高层次视图：

```js
apps/
-- admin/
-- api/
-- auth/
-- frontstore/
-- shared/
core/
---- helpers/
---- middlewares
---- models
---- services
config/
---- environments/
---- strategies
tests/
```

文件夹结构的解释如下：

+   `apps`: 这个文件夹将包含多个微应用，例如`frontstore`，它将为访问我们电子商务商店的用户提供客户端应用程序。

+   `core`: 这将是我们的应用程序的核心，包含所有必要的业务逻辑：

    +   `middlewares`: 在这个文件夹中，我们将存储所有将操作请求和响应对象的函数片段。一个很好的例子就是身份验证中间件。

    +   `models`: 这个文件夹将存储所有后端模型。

    +   `services`: 这将分组所有适用于不同客户端的通用应用程序逻辑集合，并将协调业务逻辑的消耗。

+   `config`: 所有应用程序配置文件都放在这里。

    +   `environments`: 这个文件夹包含根据当前环境加载的文件

+   `tests`: 这个文件夹包含测试应用程序后端逻辑所需的所有测试。

# 数据建模

现在我们已经对架构有了高层次的认识，让我们定义我们的模型并看看它们是如何交互的。这将为您提供一个高层次的数据存储方式，您将如何将数据存储在数据库中。此外，它还将反映不同实体之间的连接，您可以在 MongoDB 的情况下决定哪些文档将被嵌入，哪些将被引用。

## 自定义货币数据类型

在之前的支出跟踪应用程序中，我们得出结论，在 JavaScript 和 MongoDB 中处理货币数据是有方法的。只需要额外的应用程序逻辑来处理精确精度解决方案。

由于我们使用 Mongoose 作为我们的 MongoDB ODM，我们可以为货币数据定义一个自定义模型。我知道这听起来很奇怪，但通过定义虚拟属性和在我们的应用程序中重用货币数据类型，这将给我们带来优势。

让我们创建一个名为 `core/models/money.js` 的文件，并添加以下 Mongoose 架构：

```js
'use strict';

const DEF_CURRENCY = 'USD';
const DEF_SCALE_FACTOR = 100;

const mongoose = require('mongoose');
const Schema = mongoose.Schema;
const ObjectId = Schema.ObjectId;

const MoneySchema = new Schema({
  amount:   { type: Number, default: 0 },
  currency: { type: String, default: DEF_CURRENCY },
  factor:   { type: Number, default: DEF_SCALE_FACTOR }
}, {
  _id:      false,
  toObject: { virtuals: true },
  toJSON:   { virtuals: true }
});

MoneySchema
.virtual('display')
.set(function(value) {
  if (value) {
    this.set('amount', value * this.factor);
  }
})
.get(function() {
  return this.amount / this.factor;
});

module.exports = mongoose.model('Money', MoneySchema);
```

为了提高可读性，我做了以下操作：

1.  定义一个默认货币和默认缩放因子。为了实现更好的定制，你可以将这些添加到配置文件中。

1.  添加了一个名为 `display` 的虚拟属性，它将是货币模型的显示值，例如，18.99。

现在，我们已经解决了这个问题，让我们看看前面的代码。我们创建了一个自定义的 Money 模型，它将作为 Money 数据类型为我们服务。正如你所看到的，我们禁用了 `_id` 属性的自动生成。这样，如果我们将模型用作嵌入式文档，Mongoose 不会生成 `_id` 属性。

让我们来看一个例子：

```js
var price = new Money();
price.display = 18.99;
console.log(price.toObject());
// { amount: 1899, currency: 'USD', factor: 100, display: 18.99 }
```

当将价格转换为对象时，输出将包含所有必要的信息，我们不需要使用浮点数进行任何计算。记住，我们因为需要在整个应用程序中与货币保持一致性，所以将缩放因子和货币存储在价格模型中。

## 产品模型

当创建电子商务应用程序时，你必须考虑在目录中存储许多不同的产品类型。由于我们可以以任何结构表示数据，MongoDB 数据模型在这种情况下将非常有用。

在关系型数据库管理系统（RDBMS）中结构化数据会稍微困难一些；例如，一种方法是将每种产品类型表示为单独的表。每个表都会有不同的结构。另一种替代且流行的方法是 **EAV**，代表 **实体属性值**。在这种情况下，你维护一个至少包含三列的表：`entity_id`、`attribute_id` 和 `value`。EAV 解决方案非常灵活，但也有一些缺点。复杂的查询需要大量的 JOIN 操作，这可能会降低性能。

幸运的是，正如之前指出的那样，MongoDB 有一个动态架构解决方案，我们可以将所有产品数据存储在一个集合中。我们可以存储产品的通用信息和针对不同产品类型的产品特定信息。让我们开始定义我们的产品架构。创建一个名为 `core/models/product.js` 的文件，并添加以下代码：

```js
'use strict';

const mongoose = require('mongoose');
const Money = require('./money').schema;
const commonHelper = require('../helpers/common');
const Schema = mongoose.Schema;
const ObjectId = Schema.ObjectId;
const Mixed = Schema.Types.Mixed;

const ProductSchema = new Schema({
  sku:          { type: String, required: true },
  category:     { type: String },
  title:        { type: String, required: true },
  summary:      { type: String },
  description:  { type: String },
  slug:         { type: String },
  images:       { type: [
    {
      caption:  { type: String },
      filename: { type: String }
    }
  ] },
  price:        { type: Money },
  details:      { type: Mixed },
  active:       { type: Boolean, default: false }
});

module.exports = mongoose.model('Product', ProductSchema);
```

如您所见，我们有一些字段，所有类型的商品都将拥有这些字段，并且我们有一个混合属性称为`details`，它将包含关于特定商品的所有必要细节。此外，我们为`price`属性使用了自定义数据类型。默认情况下，产品在产品目录中将被标记为非活动状态，这样它只有在所有必要信息添加后才会显示。

在本书的早期——更确切地说是在第三章中——我们使用了`slug`定义来为我们的职位空缺创建 URL 友好的标题。这次，我们将用它来为我们的产品标题。为了简化问题，我们将在创建新条目时自动生成它们。

在您的产品模型文件中，在`module.exports`行之前，添加以下代码：

```js
ProductSchema.pre('save', function(next) {
  this.slug = commonHelper.createSlug(this.title);
  next();
});
```

为了唤醒您的记忆，我们在第三章中使用了相同的技巧，即*职位板*，从标题中创建一个缩略词。因此，这基本上是在数据库保存之前，从产品标题生成一个 URL 友好的字符串。

这基本上总结了我们的产品模式，并应该为我们存储产品在 MongoDB 中提供一个坚实的基础。

## 订单模型

由于我们正在尝试构建一个电子商务应用程序，我们某种程度上需要能够存储用户从我们的商店购买的商品。我们将所有这些信息存储在 MongoDB 的`orders`集合中。一个`order`条目应包含有关购买的产品、运输细节以及谁进行了购买的信息。

当您分析这个问题时，您首先想到的是，我们在下订单之前也需要有一个购物车。但如果我们把一切都简化为简单的用例，我们可以考虑购物车是一种特殊的订单。我的意思是，购物车包含将要购买的产品项，而订单将为这次购买创建。

简而言之，只有视角的改变会影响我们看待订单的方式。我们可以为订单添加一个`type`属性来决定其状态。因此，我们有几个关键点来定义我们的订单模式。现在我们可以创建一个名为`core/models/order.js`的新文件，并添加以下模式：

```js
'use strict';

const mongoose = require('mongoose');
const Money = require('./money').schema;
const Schema = mongoose.Schema;
const ObjectId = Schema.ObjectId;
const Mixed = Schema.Types.Mixed;

const OrderSchema = new Schema({
  identifier:   { type: String },
  user:         { type: ObjectId, ref: 'User' },
  type:         { type: String, default: 'cart' },
  status:       { type: String, default 'active' },
  total:        { type: Money },
  details:      { type: Mixed },
  shipping:     { type: Mixed },
  items:        { type: [
    {
      sku:      { type: String },
      qty:      { type: Number, default: 1},
      title:    { type: String },
      price:    { type: Money },
      product:  { type: ObjectId, ref: 'Product' }
    }
  ]},
  expiresAt:    { type: Date, default: null },
  updatedAt:    { type: Date, default: Date.now },
  createdAt:    { type: Date, default: Date.now }
},  {
  toObject:     { virtuals: true },
  toJSON:       { virtuals: true }
});

module.exports = mongoose.model('Order', OrderSchema);
```

如您所见，订单将在`items`属性中存储所有选定的产品，以及一些简单信息，如产品的`sku`、`quantity`和`price`。我们在项目列表中存储一些非平凡数据，如产品的`title`，这样我们就不必在非平凡操作中检索它。

当我们处理一个`cart`条目时，我们希望它最终过期，如果它没有被作为订单最终确定。这是因为我们希望释放购物车中的项目以供使用。

很可能，我们将存储关于订单和运输细节的额外信息，这些信息可能因订单而异。这就是为什么我们将它们标记为混合数据类型。

## 库存模型

到目前为止，我们已经定义了产品模式（schema）和订单模式（schema）。两者都没有提及库存状态。在 `order` 模式中，我们为每个产品项存储了订单中放置的数量，但这不会反映初始库存或当前库存。

在处理库存数据时，有几种方法，每种方法都有其自身的优点和缺点。例如，我们可以为每个实体产品存储单个记录；因此，如果我们有一个产品的 100 个库存单位，我们将在库存中存储 100 条记录。

在大型产品目录中，这不会是一个好的解决方案，因为 `inventory` 集合会迅速增长。当你有实体产品和低库存单位数量时，为每个单位存储单独条目可能是有益的。例如，一个木工店制作家具并希望跟踪每个物理单位的更多详细信息。

另一个选择是为每个产品存储单个条目，其中包含产品的库存数量。现在我们已经很好地了解了需要做什么，让我们创建一个名为 `core/models/inventory.js` 的库存模型，代码如下：

```js
'use strict';

const mongoose = require('mongoose');
const Schema = mongoose.Schema;
const ObjectId = Schema.ObjectId;
const Mixed = Schema.Types.Mixed;

const InventorySchema = new Schema({
  sku:            { type: String },
  status:         { type: String, default: 'available' },
  qty:            { type: Number, default: 0 },
  carted:         { type: [
    { type: {
        sku:      { type: String },
        qty:      { type: Number, default: 1 },
        order:    { type: ObjectId, ref: 'Order' },
        product:  { type: ObjectId, ref: 'Product' }
      }
    }
  ]},
  createdAt:      { type: Date, default: Date.now }
});

module.exports = mongoose.model('Inventory', InventorySchema);
```

我们将事情推进了一步，并添加了一个 `carted` 属性。这将保存购物车中所有活跃的项目，帮助我们跟踪库存中每个预留项目的进度。

这样，你可以有一个干净的库存水平历史记录。你可以省略 `carted` 属性，只依赖 `orders` 集合中的信息。

# 核心服务层

因为我们的应用程序将有不同的客户端消费业务逻辑，我们将添加一个服务层；它将协调不同用例的操作。因此，我们将把大部分业务逻辑从控制器移动到服务中。可能现在还太早看到这种做法的好处，但随着我们继续本章的学习，它将变得更加有意义。

一个好处是，你可以简单地暴露你的服务层作为 RESTful API，或者添加另一个将在服务器端渲染模板并显示所有必要信息的客户端。无论应用程序的客户端实现如何，你都可以测试应用程序的业务逻辑。

## 产品目录

产品目录将包含你希望显示或仅存在于系统中的所有产品。目录中的每个项目都将存储在 MongoDB 的 `products` 集合中。我们将创建一个 `ProductCatalog` 服务，它将包含我们电子商务应用程序中管理产品的所有业务逻辑。

让我们按照以下步骤创建产品目录服务：

1.  创建名为 `core/services/product-catalog.js` 的服务文件。

1.  添加以下代码：

    ```js
    'use strict';

    const MAX_PRODUCT_SHOWN = 50;
    const _ = require('lodash');
    const Product = require('../models/product');

    class ProductCatalog {
      constructor() {
      }
    }

    module.exports = ProductCatalog;
    ```

1.  声明类构造函数：

    ```js
      constructor(opts, ProductModel) {
        opts = opts || {};
        this.maxProductsShown = opts.maxProductsShown || MAX_PRODUCT_SHOWN;
        this.Product = ProductModel || Product;
      }
    ```

1.  将产品添加到目录中：

    ```js
      add(data, callback) {
        this.Product.create(data, callback);
      }
    ```

1.  我们将逐个添加类方法。

1.  编辑现有产品：

    ```js
      edit(sku, data, callback) {
        //  remove sku; this should not change,
        //  add a new product if it needs to change
        delete data.sku;

        this.Product.findBySKU(sku, (err, product) => {
          if (err) {
            return callback(err);
          }

          _.assign(product, data);
          //  tell mongoose to increment the doc version `__v`
          product.increment();
          product.save(callback);
        });
      }
    ```

1.  列出所有产品：

    ```js
      list(query, limit, skip, callback) {
        if (typeof query === 'funciton') {
          callback = limit;
          limit = this.maxProductsShown;
          skip = 0;
        }

        // make sure we only allow retriving `50` products from the catalog
        if (+limit > this.maxProductsShown) {
          limit = this.maxProductsShown;
        }

        this.Product.find(query).limit(limit).skip(skip).exec(callback);
      }
    ```

1.  使用 `sku` 标识符获取更多详细信息：

    ```js
      details(sku, callback) {
        this.Product.findBySKU(sku, callback);
      }
    ```

1.  通过 `slug` 获取产品：

    ```js
      detailsBySlug(slug, callback) {
        this.Product.findBySlug(slug, callback);
      }
    Remove a product:
      remove(sku, callback) {
        this.Product.findBySKU(sku, (err, product) => {
          if (err) {
            return callback(err);
          }

          product.remove(callback);
        });
      }
    ```

我们成功地为我们产品目录服务奠定了基础。正如你所看到的，它只掩盖了最终模块中的一些功能，这些模块不应该知道底层或数据是如何存储的。它可以是数据库，比如我们案例中的 MongoDB，或者简单地是一个文件系统。

我们获得的第一大好处是可测试性，因为我们可以在实现高级层之前测试应用程序的业务逻辑并运行集成测试。例如，我们可以有如下代码片段，它来自`tests/integration/product-catalog.test.js`：

```js
const ProductCatalog = require('../../core/services/product-catalog');

// … rest of the required modules

describe('Product catalog', () => {
  let mongoose;
  let Product;
  let productCatalog;
  let productData = { ... };  //  will hold the product related data

  before(done => {
    mongoose = require('../../config/mongoose').init();
    productCatalog = new ProductCatalog();
    // … more code
    done();
  });

  it('should add a new product to the catalog', done => {
    productCatalog.add(productData, (err, product) => {
      if (err) { throw err; }

      should.exist(prod);
      prod.title.should.equal('M.E.A.N. Blueprints');
      done();
    });
  });  
});
```

前面的测试用例将简单地检查服务执行的所有操作是否正确。我们在前面的章节中进行了大量的测试驱动开发，在后面的章节中，我们更多地关注功能，但这并不意味着我们忽略了编写测试。测试可以在完整的源代码中找到，供你在开发应用程序时检查和遵循。

## 库存管理器

野外有许多电子商务解决方案都包含库存管理器，这将帮助你跟踪产品的库存水平，补充你的产品库存，或者按需调整。

我们不想在产品文档中嵌入库存信息，所以我们将为每个产品单独存储它。你可以用很多方法跟踪你的库存；我们选择了一个适合大多数用例的解决方案，并且易于实现。

在我们开始编码之前，我想通过测试用例给你一些提示，关于我们将要实现的内容：

1.  我们应该能够跟踪产品的库存：

    ```js
      it('should create an inventory item for a product', done => {
        inventoryManager.create({
          sku: 'MEANB',
          qty: 1
        }, (err, inventoryItem) => {
          if (err) throw err;

          should.exist(inventoryItem);
          inventoryItem.sku.should.equal('MEANB');
          inventoryItem.qty.should.equal(1);
          done();
        });
      });
    ```

1.  根据需求，应从库存中预留给定产品的期望数量：

    ```js
      it('should reserve an item if there is enough on stock', done => {
        inventoryManager.reserve('MEANB', new mongoose.Types.ObjectId(), 2, (err, result) => {
          if (err) throw err;

          should.exist(result);
          result.sku.should.equal('MEANB');
          done();
        });
      });
    ```

1.  如果库存不足，服务不应满足请求：

    ```js
      it('should not reserve an item if there is not enough on stock', done => {
        inventoryManager.reserve('MEANB', new mongoose.Types.ObjectId(), 2, (err, result) => {
          should.not.exist(result);
          should.exist(err);
          err.message.should.equal('Stock lever is lower then the desired quantity.');
          err.status.should.equal(409);
          err.type.should.equal('not_enough_stock_units')
          done();
        });
      });
    ```

1.  增加可用数量：

    ```js
      it('should increase the quantity for an inventory unit', done => {
        inventoryManager.increase('MEANB', 5, (err, inventory) => {
          if (err) throw err;

          inventory.qty.should.equal(6);
          done();
        });
      });
    ```

1.  或者，你可以减少可用数量以进行调整：

    ```js
      it('should decrease the quantity for an inventory unit', done => {
        inventoryManager.decrease('MEANB', 2, (err, inventory) => {
          if (err) throw err;

          inventory.qty.should.equal(4);
          done();
        });
      });
    ```

现在我们已经大致了解了需要做什么，让我们遵循以下步骤来创建我们的库存管理器服务：

1.  创建一个新的文件，`core/services/inventory-manager.js`。

1.  定义一个起点：

    ```js
    'use strict';

    const Inventory = require('../models/inventory');

    class InventoryManager {
      constructor() {}
    }

    module.exports = InventoryManager;
    ```

1.  完成类构造函数：

    ```js
      constructor(opts, InventoryModel) {
        this.opts = opts || {};
        this.Inventory = InventoryModel || Inventory;
      }
    ```

    记住，只要它具有至少必要的属性和方法，我们就可以在我们的服务中注入自定义的`InventoryModel`。

1.  创建一个新的库存项目方法：

    ```js
      create(data, callback) {
        data.carted = [];
        this.Inventory.create(data, callback);
      }
    ```

1.  修改数量私有方法：

    ```js
      _modifyQuantity(sku, qty, reduce, callback) {
        qty = (reduce) ? qty * -1 : qty;

        this.Inventory.update({
          sku: sku
        }, {
          $inc: { qty: qty }
        }, (err, result) => {
          if (err) {
            return callback(err);
          }

          if (result.nModified === 0) {
            let err = new Error('Nothing modified.');
            err.type = 'nothing_modified';
            err.status = 400;
            return callback(err);
          }

          this.Inventory.findOne({ sku: sku }, callback);
        });
      }
    ```

    我们创建了一个私有方法，以下划线为前缀以增强语义。当操作库存水平时，这将成为主要入口点。如果没有变化，我们返回一个错误。在操作成功后，我们返回库存条目的当前状态。

1.  增加和减少数量：

    ```js
      increase(sku, quantity, callback) {
        this._modifyQuantity(sku, quantity, false, callback);
      }

      decrease(sku, quantity, callback) {
        this._modifyQuantity(sku, quantity, true, callback);
      }
    Reserve the quantity in the inventory:
      reserve(sku, orderId, quantity, callback) {
        let query = {
          sku: sku,
          qty: { $gte: quantity }
        };

        let update = {
          $inc: { qty: -quantity },
          $push: {
            carted: {
              qty: quantity,
              order: orderId
            }
          }
        };

        this.Inventory.update(query, update, (err, result) => {
          if (err) {
            return callback(err);
          }

          if (result.nModified === 0) {
            let err = new Error('Stock lever is lower then the desired quantity.');
            err.type = 'not_enough_stock_units';
            err.status = 409;
            return callback(err);
          }

          callback(null, {
            sku: sku,
            order: orderId,
            qty: quantity
          });
        });
      }
    ```

上述代码将预留库存中产品的可用数量。在某些情况下，系统无法满足请求的数量，所以我们检查在减少数量之前是否有所需的可用性。如果我们无法满足请求，我们返回一个特定的错误。

你可能还会注意到，我们逐渐添加了自己的自定义 `Error` 对象，它还包含了对状态码本身的建议。目前，由于底层 ODM 可能返回不同的 `Error` 对象，服务返回的错误没有标准格式。

在这本书中，我们可能无法满足所有用例，所以有时你必须将各个部分组合起来。

## 购物车

到目前为止，我们应该已经拥有了购物车服务所需的所有必要服务。现在，如果你允许我说的话，这个服务将会非常有趣。通常，电子商务解决方案都有一个购物车，客户可以轻松地添加或移除商品，更改数量，甚至放弃购物。

有一个重要的事情需要注意，那就是我们必须确保客户不能添加不可用的商品。换句话说，如果产品库存与请求的数量不匹配，添加操作不应成功。

基本上，我们的购物车服务将处理之前描述的所有业务逻辑。此外，当客户向购物车添加商品时，库存应该得到适当的更新。记住，我们的订单集合也会保存购物车。

关于需要做什么事情已经很明确了。如果不清楚，可以去快速查看一下测试用例。让我们创建我们的购物车服务，`core/services/shopping-cart.js`，并添加以下类：

```js
'use strict';

const EXPIRATION_TIME = 15*60; // 15 minutes
const commonHelper = require('../helpers/common');
const Order = require('../models/order');
const InventoryManager = require('./inventory-manager');
const ProductCatalog = require('./product-catalog');

class ShoppingCart {
}

module.exports = ShoppingCart;
```

这里没有太多花哨的地方。我们可以通过添加构造函数来继续：

```js
  constructor(opts, OrderModel, ProductService, InventoryService) {
    InventoryService = InventoryService || InventoryManager;
    ProductService = ProductService || ProductCatalog;
    this.opts = opts || {};
    this.opts.expirationTime = this.opts.expirationTime || EXPIRATION_TIME;
    this.Order = OrderModel || Order;
    this.inventoryManager = new InventoryService();
    this.productCatalog = new ProductService();
  }
```

在我忘记之前，我们将使用之前实现的其他两个服务来管理库存并从我们的目录中检索产品。此外，在将新商品添加到购物车之前，我们需要创建它。所以让我们添加 `createCart()` 方法：

```js
  createCart(userId, data, callback) {
    data.user = userId;
    data.expiresAt = commonHelper.generateExpirationTime(this.opts.expirationTime);
    this.Order.create(data, callback);
  }
```

当将新产品添加到购物车时，我们必须注意一些事情，并且必须验证库存是否符合请求的要求。让我们绘制购物车服务的 `addProduct()` 方法：

```js
  addProduct(cartId, sku, qty, callback) {
    this.productCatalog.findBySKU(sku, (err, product) => {
      if (err) {
        return callback(err);
      }

      let prod = {
        sku: product.sku,
        qty: qty
        title: product.title,
        price: product.price,
        product: product._id
      };

      //  push carted items into the order
      this._pushItems(cartId, prod, (err, result) => {
        if (err) {
          return callback(err);
        }

        //  reserve inventory
        this.inventoryManager.reserve(product.sku, cartId, qty, (err, result) => {
          //  roll back our cart updates
          if (err && err.type === 'not_enough_stock_units') {
            return this._pullItems(cartId, sku, () => callback(err));
          }

          // retrive current cart state
          this.findById(cartId, callback);
        });
      });
    });
  }
```

当将产品添加到购物车时，我们希望存储一些额外的信息，因此我们首先需要使用 SKU 从目录中检索产品。需要将所需数量的产品添加到购物车的商品中。在成功填充购物车后，我们需要减少库存中可用的单位数量。

如果库存中的商品不足，我们必须回滚购物车更新并在应用程序中引发错误。最后，我们得到一个持久化的购物车副本。

除了从其他两个服务中使用的其他方法外，我们还需要为 `ShoppingCart` 类实现一些方法，例如 `_pushItems()` 方法：

```js
  _pushItems(cartId, prod, callback) {
    let exdate = commonHelper.generateExpirationTime(this.opts.expirationTime);
    let now = new Date();
    //  make sure the cart is still active and add items
    this.Order.update({
      { _id: cartId, status: 'active' },
      {
        $set: { expiresAt: exdate, updatedAt: now },
        $push: { items: prod }
      }
    }, (err, result) => {
      if (err) {
        return callback(err);
      }

      if (result.nModified === 0) {
        let err = new Error('Cart expired.');
        err.type = 'cart_expired';
        err.status = 400;
        return callback(err);
      }

      //  TODO: proper response
      callback(null, result);
    });
  }
```

购物车必须处于活动状态才能向其中添加商品。此外，我们还需要更新过期日期。记住，我们正在对文档执行原子操作，因此只返回操作的原始响应。

如果我们要回滚购物车，我们需要移除已添加的商品；`_pullItems()` 方法正是如此操作：

```js
  _pullItems(cartId, sku, callback) {
    this.Order.update({
      { _id: cartId },
      { $pull: { items: { sku: sku } } }
    }, (err, result) => {
      if (err) {
        return callback(err);
      }

      if (result.nModified === 0) {
        let err = new Error('Nothing modified.');
        err.type = 'nothing_modified';
        err.status = 400;
        return callback(err);
      }

      //  TODO: proper response
      callback(null, result);
    });
  }
```

到目前为止，我们应该能够使用实现的功能轻松管理我们的购物车。`ShoppingCart`服务使用了`InventoryManager`和`ProductCatalog`服务，暴露了处理购物车操作所需的精确业务逻辑。

# 认证微应用

`Auth`微应用将在不同场景下处理认证。它将成为我们认证用户的入口点，使用有状态和无状态的方法。

我们的核心模块已经暴露了中间件来检查用户是否已认证，以及与授权相关的中间件。此功能可以在任何模块或微应用中使用。

## 定义类

这将是我们的第一个微应用，所以让我们一步一步来：

1.  创建一个名为`apps/auth/index.js`的新微应用。

1.  添加以下基本内容：

    ```js
    'use strict'

    const express = require('express');
    const router = express.Router();
    const Controller = require('./controller');

    class Auth {
    }
    ```

1.  定义构造函数：

    ```js
      constructor(config, core, app) {
        this.core = core;
        this.controller = new Controller(core);
        this.app = app;
        this.router = router;
        this.rootUrl = '/auth';
        this.regiterRoutes();
        this.app.use(this.rootUrl, this.router);
      }
    ```

    我们为我们的微应用定义了一个基本 URL，并在主 Express 应用上挂载了路由器。我们还创建了一个用于`Auth`微应用的 Controller 的新实例。

1.  注册所有必要的路由：

    ```js
      regiterRoutes() {
        this.router.post('/register', this.controller.register);

        /**
         *  Stateful authentication
         */
        this.router.post('/signin', this.controller.signin);
        this.router.get('/signout', this.controller.signout);

        /**
         *  Stateless authentication
         */
        this.router.post('/basic', this.controller.basic);
      }
    ```

    为了节省开发时间，我们从前几章借用了代码，所以前面的代码可能已经熟悉了。

1.  在主`server.js`文件中初始化您的微应用：

    ```js
    const Auth = require('./apps/auth');
    let auth = new Auth(config, core, app);
    ```

在主`server.js`文件中，我们将初始化每个应用。您可以查看`server.js`文件的最终版本，以确切了解放置内容的位置。

## 控制器

之前，我提到我们正在重用前几章中的代码。我们也为 Controller 做了这件事。我们将 Controller 转换成了一个名为`AuthController`的类，并暴露了以下方法：

1.  使用有状态认证策略登录用户：

    ```js
      signin(req, res, next) {
        passport.authenticate('local', (err, user, info) => {
          if (err || !user) {
            return res.status(400).json(info);
          }

          req.logIn(user, function(err) {
            if (err) {
              return next(err);
            }

            res.status(200).json(user);
          });
        })(req, res, next);
      }
    ```

1.  使用无状态策略进行认证：

    ```js
      basic(req, res, next) {
        passport.authenticate('basic', (err, user, info) => {
          if (err) {
            return next(err);
          }

          if (!user) {
            return res.status(400).json({ message: 'Invalid email or password.' });
          }

          Token.generate({ user: user.id }, (err, token) => {
            if (err) {
              return next(err);
            }

            if (!token) {
              return res.status(400).json({ message: 'Invalid email or password.' });
            }

            const result = user.toJSON();
            result.token = _.pick(token, ['hash', 'expiresAt']);

            res.json(result);
          });

        })(req, res, next);
      }
    ```

    在某些情况下，我们不需要持久化用户的会话。相反，我们创建一个将在每个请求中使用的令牌，以查看谁试图访问我们的端点。

1.  在我们的系统中注册一个用户：

    ```js
      register(req, res, next) {
        const userData = _.pick(req.body, 'name', 'email', 'password');

        User.register(userData, (err, user) => {
          if (err && (11000 === err.code || 11001 === err.code)) {
            return res.status(400).json({ message: 'E-mail is already in use.' });
          }

          if (err) {
            return next(err);
          }

          // just in case :)
          delete user.password;
          delete user.passwordSalt;

          res.json(user);
        });
      }
    ```

# 暴露 API

我们的核心业务逻辑需要以某种方式访问，我认为 RESTful API 会为我们提供很好的服务。为了更好地理解并遍历整个应用，我们只展示 API 的几个部分。

我们更关注整个应用从架构的角度，而不是拥有详细和完全集成的功能。

## Api 类

对于这个微应用，我们将按类型上下文分组我们的文件。首先，我们将创建我们的微应用类，`apps/api/index.js`，并添加以下内容：

```js
'use strict';

const ProductsRoutes = require('./routes/products');
const ProductController = require('./controllers/product');

class Api {
  constructor(config, core, app) {
    let productController = new ProductController(core);
    let productRoutes = new ProductsRoutes(core, productController);

    this.config = config;
    this.core = core;
    this.app = app;
    this.root = app.get('root');
    this.rootUrl = '/api';

    this.app.get('/api/status', (req, res, next) => {
      res.json({ message: 'API is running.' });
    });

    this.app.use(this.rootUrl, productRoutes.router);
  }
}

module.exports = Api;
```

此应用的这部分将`ProductRoutes`暴露的路由挂载到主 Express 应用上。前面的`ProductRoutes`类需要一个`ProductController`作为必需参数。

现在我们不会特别讨论每个 Controller 和路由，我们只关注产品部分。我们将使用`ProductCatalog`核心服务并调用所需业务逻辑。

## 产品控制器

这个控制器将处理管理产品的请求。我们将按照以下步骤来实现它：

1.  创建一个名为 `apps/api/controller/product.js` 的新文件。

1.  定义控制器：

    ```js
    'use strict';

    const _ = require('lodash');

    let productCatalog;

    class ProductsController {
      constructor(core) {
        this.core = core;
        productCatalog = new core.services.ProductCatalog();
      }
    Add the create product method:
      create(req, res, next) {
        productCatalog.add(req.body, (err, product) => {
          if (err && err.name === 'ValidationError') {
            return res.status(400).json(err);
          }

          if (err) {
            return next(err);
          }

          res.status(201).json(product);
        });
      }
    ```

1.  添加 `getAll` 产品方法：

    ```js
      getAll(req, res, next) {
        const limit = +req.query.limit || 10;
        const skip = +req.query.skip || 0;
        const query = {} // you cloud filter products

        productCatalog.list(query, limit, skip, (err, products) => {
          if (err) {
            return next(err);
          }

          res.json(products);
        });
      }
    Implement a method that retrieves a single product:
      getOne(req, res, next) {
        productCatalog.details(req.params.sku, (err, product) => {
          if (err) {
            return next(err);
          }

          res.json(product);
        });
      }
    ```

## 产品路由

定义路由与我们在 `Auth` 微应用中之前所做的方式类似，但我们把路由移动到了一个单独的文件中，称为 `apps/api/routes/products.js`。文件的内容相当简单：

```js
'use strict';

const express = require('express');
const router = express.Router();

class ProductsRoutes {
  constructor(core, controller) {
    this.core = core;
    this.controller = controller;
    this.router = router;
    this.authBearer = this.core.authentication.bearer;
    this.regiterRoutes();
  }

  regiterRoutes() {
    this.router.post(
      '/products',
      this.authBearer(),
      this.controller.create
    );

    this.router.get(
      '/products',
      this.authBearer(),
      this.controller.getAll
    );

    this.router.get(
      '/products/:sku',
      this.authBearer(),
      this.controller.getOne
    );
  }
}

module.exports = ProductsRoutes;
```

如您所见，我们从核心模块使用了 bearer 认证中间件来检查用户是否拥有有效的令牌。这个函数具有以下结构：

```js
function bearerAuthentication(req, res, next) {
  return passport.authenticate('bearer', { session: false });
}
```

我认为我们已经了解了我们的 `Api` 微应用是如何工作的以及需要做什么。你可以通过项目的代码仓库来查看其余的代码。

# 共享资源

许多微应用将使用相同的静态资源，以避免在应用程序之间重复这些资源。我们可以创建一个微应用来提供所有共享资源。

而不是有一个主要的 `public` 文件夹，每个想要提供静态文件的微应用都可以有一个单独的 `public` 文件夹。这意味着我们可以将所有共享的静态资源移动到内部的 `public` 文件夹中。

我们将拥有以下文件夹结构：

```js
apps/
-- shared/
---- public
------ assets/
---- index.js
```

`index.js` 文件将包含以下内容：

```js
'use strict';

const path = require('path');
const serveStatic = require('serve-static');

class Shared {
  constructor(config, core, app) {
    this.app = app;
    this.root = app.get('root');
    this.rootUrl = '/';
    this.serverStaticFiles();
  }

  serverStaticFiles() {
    let folderPath = path.resolve(this.root, __dirname, './public');
    this.app.use(this.rootUrl, serveStatic(folderPath));
  }
}

module.exports = Shared;
```

我们定义一个类，并从 `public` 文件夹提供所有静态资源。我们使用了 `path` 模块中的 `resolve` 方法来解析到 `public` 文件夹的路径。

如您所见，修改我们之前章节中的架构相当简单。此外，前面的技术也将用于我们的 `admin` 微应用。

# 管理部分

通常，电子商务解决方案都包含一个管理部分，您可以在其中管理您的产品和库存。我们的应用的管理部分将使用 Angular 2 构建。没有什么特别的；我们不是已经用 Angular 构建了一些应用吗？

我们不会详细介绍所有细节，只介绍应用中最重要的一部分。不用担心！项目的完整源代码都是可用的。

## 管理微应用

我们从一开始就做了一些架构上的改变。我们的每个微应用都将服务于特定的目的。`admin` 微应用将托管使用 Angular 2 构建的管理应用。

在前面的章节中，我们使用了 server-static 来公开 `public` 文件夹的内容。这个应用将拥有自己的 `public` 文件夹，并且只包含与我们的 Angular 管理应用相关的文件。

这个微应用将会相当简单。创建一个名为 `apps/admin/index.js` 的文件，内容如下：

```js
'use strict';

const path = require('path');
const serveStatic = require('serve-static');

class Admin {
  constructor(config, core, app) {
    this.app = app;
    this.root = app.get('root');
    this.rootUrl = '/admin';
    this.serverStaticFiles();
  }

  serverStaticFiles() {
    let folderPath = path.resolve(this.root, __dirname, './public');
    this.app.use(this.rootUrl, serveStatic(folderPath));
  }
}

module.exports = Admin;
```

`Admin` 类将定义我们的微应用，并使用 `serverStaticFiles()` 方法公开公共文件夹的内容以供外部使用。文件服务在 `/admin` URL 路径上挂载。

不要忘记查看主要的 `server.js` 文件以正确初始化您的 `admin` 微应用。

## 修改认证模块

`admin`应用使用令牌来授权访问 API 的端点。因此，我们需要对我们的`AuthHttp`服务进行一些更改，从`apps/admin/public/src/auth/auth-http.ts`开始。

这些更改会影响`request`方法，其外观将如下所示：

```js
  private request(requestArgs: RequestOptionsArgs, additionalArgs?: RequestOptionsArgs) {
    let opts = new RequestOptions(requestArgs);

    if (additionalArgs) {
      opts = opts.merge(additionalArgs);
    }

    let req:Request = new Request(opts);

    if (!req.headers) {
      req.headers = new Headers();
    }

    if (!req.headers.has('Authorization')) {
      req.headers.append('Authorization', `Bearer ${this.getToken()}`);
    }

    return this._http.request(req).catch((err: any) => {
      if (err.status === 401) {
        this.unauthorized.next(err);
      }

      return Observable.throw(err);
    });
  }
```

对于每个请求，我们添加必要的令牌的`Authorization`头。此外，我们还需要使用以下方法从`localStorage`检索令牌：

```js
  private getToken() {
    return localStorage.getItem('token');
  }
```

令牌将在成功登录后持久化到`localStorage`。在`AuthService`中，我们将存储当前用户及其令牌并将其持久化到`localStorage`：

```js
  public setCurrentUser(user: any) {
    this.currentUser.next(user);
  }

  private _initSession() {
    let user = this._deserialize(localStorage.getItem('currentUser'));
    this.currentUser = new BehaviorSubject<Response>(user);
    // persist the user to the local storage
    this.currentUser.subscribe((user) => {
      localStorage.setItem('currentUser', this._serialize(user));
      localStorage.setItem('token', user.token.hash || '');
    });
  }
```

当用户成功登录时，我们将当前用户存储在主题中，并通知所有订阅该更改的订阅者。

记住，我们可以通过使用位于边界上下文根目录的单个`index.ts`文件来简单地公开上下文的所有成员。对于`auth`模块，我们可以有如下结构：

```js
auth/
-- components/
-- services/
-- index.ts
```

例如，我们的`AuthHttp`服务可以通过以下方式在`index.ts`中导出：

```js
export * from './services/auth-http';
```

我们可以使用此行将其导入到另一个组件中：

```js
import { AuthHttp } from './auth/index';
```

而不是以下方法：

```js
import { AuthHttp } from './auth/services/auth-http';
```

## 产品管理

在后端部分，我们创建了一个服务并公开了一个 API 来管理产品。现在在客户端，我们需要创建一个模块来消费该 API 并允许我们执行不同的操作。

### 产品服务

我们将仅讨论产品服务中的几个方法，因为我们基本上将在管理部分执行简单的 CRUD 操作。让我们创建一个名为`apps/admin/public/src/services/product.service.ts`的文件，并包含以下基本内容：

```js
import { Injectable } from 'angular2/core';
import { Http, Response, Headers } from 'angular2/http';
import { Observable } from 'rxjs/Observable';
import { ProductService } from './product.service';
import { contentHeaders } from '../common/headers';
import { Product } from './product.model';

type ObservableProducts = Observable<Array<Product>>;

@Injectable()
export class ProductService {
  public products: ObservableProducts;

  private _authHttp: AuthHttp;
  private _productsObservers: any;
  private _dataStore: { products: Array<Product> };

  constructor(authHttp: Http) {
    this._authHttp = authHttp;
    this.products = new Observable(observer => this._productsObservers = observer).share();
    this._dataStore = { products: [] };
  }
}
```

接下来，我们将添加`getAll`产品方法。当我们想要显示产品列表时，我们将使用此方法。将以下代码添加到`ProductService`中：

```js
  getAll() {
    this._authHttp
    .get('/api/products', { headers: contentHeaders })
    .map((res: Response) => res.json())
    .subscribe(products => {
      this._dataStore.products = products;
      this._productsObservers.next(this._dataStore.products);
    });
  }
```

其余的方法都在项目的完整源代码中。

### 列出产品

在主要产品管理部分，我们将列出目录中所有可用的产品。为此，我们在`apps/admin/public/product/components/product-list.component.ts`下创建了一个另一个组件：

```js
import { Component, OnInit } from 'angular2/core';
import { ProductService } from '../product.service';
import { Router, RouterLink } from 'angular2/router';
import { Product } from '../product.model';

@Component({
    selector: 'product-list',
    directives: [RouterLink],
    template: `
      <div class="product-list row">
        <h2 class="col">Products list</h2>
        <div *ngIf="products.length === 0" class="empty-product-list col">
          <h3>Add your first product to you catalog</h3>
        </div>
        <div class="col col-25">
          <a href="#" [routerLink]="['ProductCreate']" class="add-product-sign">+</a>
        </div>
        <div *ngFor="#product of products" class="col col-25">
          <img src="img/208x140?text=product+image&txtsize=18" />
          <h3>
            <a href="#"
              [routerLink]="['ProductEdit', { sku: product.sku }]">
              {{ product.title }}
            </a>
            </h3>
        </div>
      </div>
    `
})
export class ProductListComponent implements OnInit {
  public products: Array<Product> = [];
  private _productService: ProductService;

  constructor(productService: ProductService) {
    this._productService = productService;
  }

  ngOnInit() {
    this._productService.products.subscribe((products) => {
      this.products = products
    });
    this._productService.getAll();
  }
}
```

上述代码将仅列出从服务检索的所有产品，并具有编辑特定产品的路由链接。您可以轻松列出产品的额外详细信息；您只需修改模板即可。

### 主要产品组件

为了管理我们的路由，我们必须创建一个主入口点并创建一个组件来处理。为了有一个完整的画面，我将向您展示`ProductComponent`的最终版本，该版本位于`apps/admin/public/src/product/product.component.ts`下：

```js
import { Component } from 'angular2/core';
import { RouteConfig, RouterOutlet } from 'angular2/router';
import { ProductListComponent } from './product-list.component';
import { ProductEditComponent } from './product-edit.component';
import { ProductCreateComponent } from './product-create.component';

@RouteConfig([
  { path: '/', as: 'ProductList', component: ProductListComponent, useAsDefault: true },
  { path: '/:sku', as: 'ProductEdit', component: ProductEditComponent },
  { path: '/create', as: 'ProductCreate', component: ProductCreateComponent }
])
@Component({
    selector: 'product-component',
    directives: [
      ProductListComponent,
      RouterOutlet
    ],
    template: `
      <div class="col">
        <router-outlet></router-outlet>
      </div>
    `
})
export class ProductComponent {
  constructor() {}
}
```

我们使用此组件来配置产品列表、创建新产品以及通过特定 SKU 编辑现有产品的路由。这样，我们可以轻松地将它挂载在更高层次组件上。

### 添加和编辑产品

基本上，我们所做的是使用了相同的模板来编辑和添加产品。在这个应用程序中，当查看产品详情时，您实际上正在编辑产品。这样，我们就不必从详细视图单独实现或隐藏编辑功能。

由于应用程序处于早期阶段，创建新产品和更新现有产品之间没有区别，我们可以减少工作量，同时实现这两者。

编辑产品源代码可以在 `apps/admin/public/src/product/components/product-edit.component.ts` 中找到。

## 订单处理

系统应处理订单，这意味着有人需要处理订单的状态。通常，订单可以有几种状态。我将在下表中尝试解释其中的一些：

| 名称 | 状态 | 描述 |
| --- | --- | --- |
| 待处理 | pending | 订单已接收（通常是未付款）。 |
| 失败 | failed | 发生了错误；即支付失败或被拒绝。 |
| 处理中 | processing | 订单正在等待履行。 |
| 完成 | completed | 订单已履行并完成。通常，不需要进一步的操作。 |
| 挂起 | on_hold | 股票数量已减少，但等待进一步的确认，即支付。 |
| 取消 | cancelled | 订单被客户或管理员取消。 |

我们不会处理我们刚才描述的所有场景。应用程序的完整版本只支持其中的一些：待处理、处理中、已取消和已完成。由于我们不会实现支付方法，因此没有必要处理所有场景。

在这么多代码之后，我认为我们可以休息一下，只讨论这部分。您可以从 GitHub 仓库中查看工作版本。

### 检索订单

为了管理所有 incoming 订单，我们需要将它们列出来给管理员。我们不会深入到代码的细节，因为它与我们迄今为止所做的是非常相似的。

在 `public/src/order/order.service.ts` 中找到的服务将处理订单实体的所有操作。在这个应用程序中可以添加的一个不错的功能是从后端获取订单流。这与我们在第四章，*聊天应用*中工作时所做的类似，当时我们使用了 WebSockets。

换句话说，我们可以在新订单被添加到系统中时立即通知所有客户。当您有大量订单涌入并希望尽快收到通知以便尽快处理时，这将提供很大的帮助。

### 查看和更新订单

通常，在处理订单之前，您可能想查看更多关于它的信息，例如送货地址，或客户提供给您的任何其他信息。但与此同时，处理订单所需采取的操作应保持在最低限度。

考虑到所有这些，我们选择了这样一个解决方案：可以在同一上下文中查看和编辑订单。因此，`OrderDetailsComponent` 正好做到了这一点；它可以在以下位置找到：`public/src/order/components/order-details.ts`。

完整的源代码可以在仓库中找到，但我会尽量解释我们在那里做了什么。

# 构建 Storefront

正如我们在本章开头所讨论的，我们将尝试一些不同的事情。我们不会为 Storefront 构建单页应用程序，而是将实现服务器端渲染的页面。

技术上，我们将构建一个经典的网页。页面将是动态的，使用视图引擎来渲染我们的模板。

我们希望真正利用我们无头核心应用程序的优势，并看看我们如何将其与不同的客户端应用程序集成，因此我们将使用第三方包进行一些服务器端渲染页面的实验。

我们可以很容易地使用 Angular 来构建它，但我想要添加一些变化，看看更复杂的解决方案是如何实施的。

## Storefront 微应用

正如我们在应用程序的管理部分之前所看到的，我们将它从主应用程序解耦为一个微应用。因此，从技术上讲，我们可以在任何时间从这个应用程序中提取出 Storefront 所需的代码，将其添加到一个全新的 Express 应用程序中，并在网络上进行所有调用。

起初，这可能会显得有些奇怪，但一旦你的应用程序开始增长并且你需要扩展应用程序，这将为你提供优势，以便区分哪些部分需要扩展或移动到单独的应用程序以实现更好的可扩展性。

事先考虑总是一件好事，但我也不是特别喜欢过早的优化。你无法从一开始就确定你的应用程序在将来会如何增长，但提前规划是明智的。

Storefront 应用程序将展示我们如何在同一应用程序中集成不同的技术。重点是纯粹的教育性，这被添加到书中，以展示构建 Express 应用程序的不同方法。

让我们谈谈构建我们的 Storefront 所使用的科技。我们将使用 `nunjucks`，这是一个为 JavaScript 设计的优秀的模板引擎。它可以在服务器端和客户端同时使用。

在我们到达模板部分之前，我们需要做一些准备工作：

1.  在 `apps/storefront` 下创建一个新的 `apps` 文件夹。

1.  添加一个新的文件，`apps/storefront/index.js`。

1.  定义微应用的类：

    ```js
    'use strict';

    const express = require('express');
    const nunjucks = require('nunjucks');
    const router = express.Router();
    const ProductController = require('./controllers/products');

    class Storefront {
      constructor(config, core, app) {
        this.config = config;
        this.core = core;
        this.app = app;
        this.router = router;
        this.rootUrl = '/';
        this.productCtrl = new ProductController(core);
        this.configureViews();
        this.regiterRoutes();
        this.app.use(this.rootUrl, this.router);
      }
    }
    ```

1.  配置视图引擎：

    ```js
      configureViews() {
        let opts = {};

        if (!this.config.nunjucks.cache) {
          opts.noCache = true;
        }

        if (this.config.nunjucks.watch) {
          opts.watch = true;
        }

        let loader = new nunjucks.FileSystemLoader('apps/frontstore/views', opts);

        this.nunjucksEnv = new nunjucks.Environment(loader);
        this.nunjucksEnv.express(this.app);
      }
    ```

1.  注册路由：

    ```js
      registerRoutes() {
        this.router.get('/', this.productCtrl.home);
      }
    ```

对于这个微应用，我们开始使用视图引擎在服务器端渲染我们的模板。`configureViews()` 方法将初始化 `nunjucks` 环境，并从文件系统中加载模板文件。我们还检查是否应该从 `nunjucks` 激活缓存和监视功能。你可以在项目的文档中了解更多信息。

最后，我们将应用程序的路由注册为我们在之前一起构建的每个 Express 应用程序所做的那样。为了便于阅读，我只添加了主页位置，并且只实例化了`ProductController`。

如果你想知道`ProductController`是什么，我们只是使用类方法为我们的控制器文件，这样我们就可以实例化它，并传递应用程序的核心。让我们看看`apps/storefront/controllers/product.js`中的一个代码段：

```js
'use strict';

let productCatalog;

class ProductsController {
  constructor(core) {
    this.core = core;
    productCatalog = new core.services.ProductCatalog();
  }

  home(req, res, next) {
    productCatalog.list({}, 10, 0, (err, products) => {
      if (err) {
        next(err);
      }

      res.render('home', { products: products });
    });
  }
}

module.exports = ProductsController;
```

所以基本上，我们导出了一个控制器类，在`home()`方法中，我们使用我们的`ProductCatalog`服务从持久存储（在我们的情况下，是 MongoDB）检索产品。在成功获取所有产品后，我们使用响应对象的`render()`方法从我们的模板中渲染一个 HTML 响应。

## 店面页面

我们不会深入细节；你可以查看整个项目，看看事物是如何粘合在一起的。

### 主要布局

为了有一个单一的布局定义，几乎每个模板都会扩展一个主模板文件。这个主模板文件将包含一个完整 HTML 文档的所有必要标记。主布局文件可以在`apps/storefront/views/layout.html`下找到：

```js
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <title>ecommerce</title>
    {% include "includes/stylesheets.html" %}
  </head>
  <body>
    <div class="container">
      <div class="app-wrapper card whiteframe-z2">

        {% block header %}
        <header>
          <div class="row">
            <div class="col">
              <h1><a href="#">Awesome store</a></h1>
              <span class="pull-right">{{ currentUser.email }}</span>
            </div>
          </div>
        </header>
        {% endblock %}

        <div class="row">
          {% block content %}{% endblock %}
        </div>

      </div>
    <div>
    {% block footer %}
    <footer></footer>
    {% endblock %}
  </body>
</html>
```

主要的`layout.html`文件定义了注入内容的块。因为我们有一个`Shared`微应用，所以所有必要的资产都对我们可用，因此我们可以使用一个单独的文件，`apps/storefront/views/includes/stylesheets.html`来导入这些资产：

```js
<link href='https://fonts.googleapis.com/css?family=Work+Sans' rel='stylesheet' type='text/css'>
<link rel="stylesheet" type="text/css" href="/assets/css/style.css">
```

### 列出产品

为了实现完全集成，让我们看看我们如何列出我们的产品。创建一个新的模板文件`apps/storefront/views/home.html`并添加以下内容：

```js
{% extends "layout.html" %}

{% block content %}
  <div class="product-list row">
    <div class="col">
    {% for product in products %}
      {% include "partials/product.html" %}
    {% endfor %}
    </div>
  </div>
{% endblock %}
```

我们只是使用前面的代码扩展了`content`块，遍历产品列表，并使用部分视图创建一个新的产品。

让我们看看那个部分视图，`apps/storefront/views/partials/product.html`：

```js
<div class="col col-25 product-item">
  <a href="{{ baseUrl }}/products/{{ product.slug }}">
    <img src="img/208x140?text=product+image&txtsize=18" />
  </a>
  <h2>
    <a href="{{ baseUrl }}/products/{{ product.slug }}">{{ product.title }}</a>
  </h2>
  <p>{{ product.summary }}</p>
  <p>price: {{ product.price.display }} {{ product.price.currency}}</p>
  <p><button class="button">add to cart</button></p>
</div> 
```

静态 HTML 标记被转换成了动态视图。我们使用与用 Angular 2 构建的`Admin`微应用相同的结构。

如果你对剩余的代码感兴趣，请前往项目的仓库[`github.com/robert52/mean-blueprints-ecommerce`](https://github.com/robert52/mean-blueprints-ecommerce)以获取更多详细信息。这部分应用程序只是为了展示你可以集成到你的 MEAN 堆栈中的不同方法。你总是可以扩展你的堆栈，使用不同的技术，看看什么更适合你。

有时候，你需要将事物结合起来，但有一个坚实的基础可以使你的生活长期更容易。我们本可以使用 Angular 构建一切，但看到我们如何扩展我们的视野总是很棒。

# 摘要

本章是关于构建电子商务应用程序。从本章的开始，我们就开始尝试新的应用程序架构，这种架构可以很容易地扩展到未来，并且也用于我们的店面实现中的服务器端渲染。

尽管这与前几章有很大不同，但它很好地服务于教育目的，并为新的可能性打开了大门。请保持您的架构模块化，并且首先只对小部分进行实验，以查看事情是否如您所愿地发展。

在下一章中，我们将尝试通过添加拍卖应用来扩展我们现有的电子商务应用。
