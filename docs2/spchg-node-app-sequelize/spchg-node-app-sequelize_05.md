

# 将钩子和生命周期事件添加到您的模型中

ORM 通常提供一种方法，使我们能够在执行某些操作时发生的事件中转换状态或对象。这些方法通常被称为钩子、生命周期事件、对象生命周期，甚至回调（后者在 Node.js 社区中不常用，因为与 Node.js 的原生环境存在命名冲突）。通常，这些方法有一个时间前缀（例如，`before` 和 `after`）在事件名称之前。

ORM 在其整个生命周期中对事件的要求没有严格的规则。ORM 中通常包括的事件被称为：验证、保存、创建、更新和销毁。其他 ORM 框架提供更广泛的事件范围或更细粒度的控制，例如在连接到数据库之前/之后、定义您的模型以及调用查找查询时。

Sequelize 将钩子分为全局钩子和局部钩子。全局钩子用于为每个模型定义默认的生命周期事件，强制事件（在 Sequelize 中称为永久钩子），以及与连接相关的事件。局部钩子包括在模型上定义的实例/记录的生命周期事件。

在本章中，我们将介绍以下内容：

+   生命周期事件的执行顺序

+   定义、删除和执行生命周期事件

+   在关联和事务中使用生命周期事件

注意

您可以始终参考 Sequelize 的代码库来维护一个可用的生命周期事件列表：[`sequelize.org/docs/v6/other-topics/hooks/`](https://sequelize.org/docs/v6/other-topics/hooks/)。

# 技术要求

您可以在[`github.com/PacktPublishing/Supercharging-Node.js-Application-with-Sequelize/blob/main/ch5`](https://github.com/PacktPublishing/Supercharging-Node.js-Application-with-Sequelize/blob/main/ch5)找到本章的代码文件。

# 生命周期事件的执行顺序

当我们想要引入超出数据库引擎范围的项目特定行为/约束时，生命周期事件是一个重要的特性。了解生命周期事件只是方程的一半，另一半是了解这些生命周期事件何时被触发。

假设我们被分配了一个任务，即向所有员工免费提供我们的产品。第一个动作可能是添加一个 `beforeValidate` 钩子，如果用户是员工，则将事务的小计设置为 `0`。这对我们来说很简单，但不幸的是，这对会计部门来说是一个噩梦。更好的方法是在 `beforeValidate` 或 `beforeCreate` 钩子中添加一个表示员工折扣的额外项目。

知道使用哪些生命周期事件的真实答案取决于项目的需求。从我们之前的例子中，一些交易需要移动法定货币，这涉及到先向员工收费，然后作为单独的交易提供退款/信用。在这种情况下，我们无法使用`beforeValidate`或`beforeCreate`，但`afterCreate`可能适用。在 Sequelize 的上下文中，知道在哪里放置你的代码逻辑就是知道生命周期事件的操作顺序。

在 Sequelize 中，生命周期事件遵循`before`/`after`前缀风格命名钩子，就像其他 ORM 框架一样。Sequelize 的所有*连接*生命周期事件都定义在`sequelize`对象本身上，所有*实例*事件类型都定义在模型上。*模型*事件类型可以在两个地方定义。这些规则的例外情况是我们想要为所有模型全局定义实例事件（以下章节将提供示例）。以下是按执行顺序列出生命周期事件及其回调函数签名的表格： 

**钩子定义按生命周期执行顺序排序**

| **事件名称** | **事件类型** | **需要同步** |
| --- | --- | --- |
| `beforeConnect(config)``beforeDisconnect(connection)` | 连接 | 否 |
| `beforeSync(options)``afterSync(options)` | 连接 | 否 |
| `beforeBulkSync(options)``afterBulkSync(options)` | 连接 | 否 |
| `beforeQuery(options, query)` | 连接 | 否 |
| `beforeDefine(attributes, options)``afterDefine(factory)` | 连接（模型） | 是 |
| `beforeInit(config, options)``afterInit(sequelize)` | 连接（模型） | 是 |
| `beforeAssociate({ source, target, type }, options)``afterAssociate({ source, target, type, association }, options)` | 连接（模型） | 是 |
| `beforeBulkCreate(instances, options)``beforeBulkDestroy(options)``beforeBulkRestore(options)``beforeBulkUpdate(options)` | 模型 | 否 |
| `beforeValidate(instance, options)` | 实例 | 否 |
| `afterValidate(instance, options)``validationFailed(instance, options, error)` | 实例 | 否 |
| `beforeCreate(instance, options)``beforeDestroy(instance, options)``beforeRestore(instance, options)``beforeUpdate(instance, options)``beforeSave(instance, options)``beforeUpsert(values, options)` | 实例 | 否 |
| `afterCreate(instance, options)``afterDestroy(instance, options)``afterRestore(instance, options)``afterUpdate(instance, options)``afterSave(instance, options)``afterUpsert(created, options)` | 实例 | 否 |
| `afterBulkCreate(instances, options)``afterBulkDestroy(options)``afterBulkRestore(options)``afterBulkUpdate(options)` | 实例 | 否 |
| `afterQuery(options, query)` | 连接 | 否 |
| `beforeDisconnect(connection)``afterDisconnect(connection)` | 连接 | 否 |

*这些生命周期事件只有在调用`sequelize.sync()`时才会触发。

这些生命周期事件中的大多数都与它们的 Sequelize 函数（例如，`beforeSave` 对应于 `Model.save()`）相对应。然而，有两种类型的事件是隐含的，并且可能一开始并不明显。第一种是与偏执模型（记录被视为通过列标志标记为`delete`而不是物理删除）相关的`restore`事件。第二种是`Upsert`事件，这些事件在`create`、`update`和`save`相关方法中被调用，向我们指示记录是新建的还是从现有记录更新而来。

Sequelize 与其他 ORM 生命周期事件的不同之处在于，除了与*实例*和*连接*相关的事件外，Sequelize 还将提供围绕*查询*方法的钩子（例如，`findAll`和`findOne`）。以下是一个包含每个查询事件简要说明的列表：

+   `beforeFind(options)`: 在 Sequelize 内部对选项进行任何转换之前发生

+   `beforeFindAfterExpandIncludeAll(options)`: 在 Sequelize 扩展*include*属性（例如，为特定关联设置适当的默认值）后触发的事件

+   `beforeFindAfterOptions(options)`: 在查询方法调用查询之前以及 Sequelize 完成填充/转换选项之后发生

+   `afterFind(instances, options)`: 在查询方法完成后返回单个实例或实例数组

+   `beforeCount(options)`: 在`count()`实例方法查询数据库之前触发此事件

现在我们对可以使用哪些钩子和生命周期执行的顺序有了更好的理解，我们可以开始构建带有附加生命周期事件的模型。

# 定义、删除和执行生命周期事件

有几种方法可以将生命周期事件附加到模型和 Sequelize 的行为上。这些方法中的每一种都允许我们通过引用传递来更改从钩子参数派生的属性值。例如，您可以通过在生命周期方法内部更新对象上的属性来简单地向`afterFind`返回的实例添加额外的属性。默认情况下，Sequelize 将生命周期事件视为同步操作，但如果您需要异步功能，则可以返回一个`Promise`对象或一个`async`函数。

## 定义实例和模型生命周期事件

实例和模型生命周期事件可以通过多种方式定义，包括将这些事件定义为本地钩子（直接从模型本身定义）。定义本地钩子有几种方法；我们将从在模型初始化期间声明钩子的基本示例开始：

```js
class Receipt extends Model {}
Receipt.init({
  subtotal: DataTypes.DECIMAL(7, 2)
}, {
  hooks: {
    beforeValidate: (receipt, options) => {
      if (isEmployee(receipt.customer)) {
        receipt.subtotal = 0;
      }
    }
  }
});
// or with the define() method
sequelize.define('Receipts', {
  subtotal: DataTypes.DECIMAL(7, 2)
}, {
  hooks: {
    beforeValidate(receipt, options) => { … })
  }
});
```

要在初始化之外定义完全相同的钩子，我们可以使用 `addHook()` 方法或直接调用相应的生命周期方法。此方法为插件和适配器在定义模型后轻松集成提供了方便。以下是如何使用此方法的简单示例：

```js
function employeeDiscount(receipt, options) {
  if (isEmployee(receipt.customer)) {
    receipt.subtotal = 0;
  }
}
class Receipt extends Model {}
Receipt.init({ subtotal: DataTypes.DECIMAL(7, 2) });
Receipt.addHook('beforeValidate',employeeDiscount);
// or you can use the direct method:
Receipt.beforeValidate(employeeDiscount);
```

之前的例子提供了同步事件的说明。异步钩子的一个例子是返回一个 Promise（如前所述），如下所示：

```js
async function employeeDiscount(receipt, options) {
  if (!customerIsEmployee) {
    return;
  }
  const discountTotal = await 
  getDiscountFromExternalAccountingService(employeeId);
  receipt.subtotal = discountTotal;
}
Receipt.addHook('beforeValidate', employeeDiscount);
// or…
Receipt.beforeValidate(employeeDiscount);
```

要从同步生命周期事件中抛出错误，你可以返回一个被拒绝的 `Promise` 对象：

```js
Receipt.beforeValidate((receipts, options) => {
  return Promise.reject(new Error("Invalid receipt"));
});
```

为了组织目的，你可以使用 `addHook()` 或直接方法为你的生命周期事件声明名称：

```js
Receipt.addHook('beforeValidate', 'checkForNegativeSubtotal', (receipt, options) => { … });
// or
Receipt.beforeValidate('checkForNegativeSubtotal', (receipt, options) => {…});
```

这些例子为我们提供了在模型本身的局部作用域上分配生命周期事件的方法。如果我们想在全局作用域（适用于所有模型）上定义生命周期事件，我们将使用 Sequelize 构造函数来完成：

```js
const sequelize = new Sequelize(…, {
  define: {
    hooks: {
      beforeValidate() {
     // perform some kind of data transformation/validation
      }
  }
});
```

这将为未定义自己的 `beforeValidate` 钩子的模型生成默认的 `beforeValidate` 钩子。如果你希望无论模型是否有自己的定义都运行全局钩子，我们可以定义**永久钩子**：

```js
sequelize.addHook('beforeValidate', () => { … });
```

即使模型有自己的 `beforeValidate` 钩子定义，Sequelize 仍然会执行全局钩子。如果我们有一个与同一生命周期事件关联的全局和局部钩子，那么 Sequelize 会首先执行局部钩子（们），然后是全局钩子（们）。

对于特定的模型事件类型（如 `bulkDestroy` 和 `bulkUpdate`），Sequelize 默认不会按行执行单个删除和更新钩子。要修改这种行为，我们可以在调用这些方法时添加 `{ individualHooks: true }` 选项，如下所示：

```js
await Receipt.destroy({
  where: { … },
  individualHooks: true
});
```

注意

使用 `{ indvidualHooks: true }` 选项可能会降低性能，这取决于 Sequelize 是否需要检索行、在内存中存储行/附加信息（例如，`bulkDestroy` 和 `bulkUpdate` 但不是 `bulkCreate`），以及为每条记录执行单个钩子。

## 移除生命周期事件

一些项目可能需要条件性地调用生命周期事件。例如，我们可能需要进行某种验证来检查用户是否仍然有资格回复论坛上的评论。这种验证对于生产环境是合适的，但对于开发环境则不是必需的。

一种方法是在钩子定义周围创建条件逻辑——例如，以下内容：

```js
if (!isDev) {
  User.addHook('beforeValidate', 'checkForPermissions', …);
}
```

这在技术上是可以工作的，但如果我们有多个规定，比如在 `afterCreate` 钩子中发送订单邮件或在生产环境中仅退款？我们将在代码库中有很多“`if`”语句。Sequelize 提供了一种移除生命周期事件的方法来帮助组织这种类型的工作流程，称为 `removeHook`。

我们可以像平时一样加载所有生命周期事件，但如果我们的环境处于开发阶段，那么我们可以遍历所有模型并移除适用的钩子。所有这些细粒度的调整都可以通过`removeHook`方法组织在一个函数中：

```js
function removeProductionOnlyHooks() {
  // this will remove all matching hooks by event type and 
     name
  User.removeHook('beforeValidate', 'checkForPermissions');
  // this will remove all beforeValidate hooks on the User 
     model
  User.removeHook('beforeValidate');
  // this will remove all of the User model's hooks
  User.removeHook();
}
 // load our models…
if (isDev) {
  removeProductionHooksOnly();
}
```

移除生命周期事件对于应用程序中的定时行为或移除显式调试钩子很有用。下一节将帮助我们了解执行生命周期事件时的操作顺序以及特定生命周期事件将在何时执行。

## 执行生命周期事件

Sequelize 将根据你调用的方法运行相应的/适用的生命周期事件。使用我们之前的`Transactions`模型示例，如果我们运行`Transactions.create({ … })`，那么 Sequelize 将自动运行以下生命周期事件（按顺序）：

1.  `beforeValidate`

1.  `afterValidate`/`validationFailed`

1.  `beforeCreate`

1.  `beforeSave`

1.  `afterSave`

1.  `afterCreate`

需要注意的一个问题是，在执行生命周期事件时，当你使用`update()`方法时，重要的是要记住，除非某个属性的值已更改，否则 Sequelize 不会执行生命周期事件。

例如，这不会调用相应的生命周期事件：

```js
var post = await Post.findOne();
await Post.update(post.dataValues, {
  where: { id: post.id }
});
```

由于值没有改变，Sequelize 将忽略生命周期事件。如果我们想强制这种行为，我们可以在更新的配置中添加一个`hooks: true`参数：

```js
await Post.update(post.dataValues, {
  where: { id: post.id },
  hooks: true
});
```

现在我们已经了解了如何定义、移除和执行生命周期事件的基础知识，我们可以继续探讨如何利用关联和事务的钩子。

# 使用关联和事务的生命周期事件

作为默认行为，Sequelize 将在不将事务与生命周期范围内的任何数据库查询相关联的情况下执行生命周期事件。然而，有时我们的项目需要在生命周期事件中使用事务，例如会计的账簿或创建日志条目。当调用某些方法，如`update`、`create`、`destroy`和`findAll`时，Sequelize 提供了一个`transaction`参数，这将允许我们使用在生命周期范围外定义的事务在生命周期内部使用。

注意

当在模型上调用`beforeDestroy`和`afterDestroy`时，Sequelize 会故意跳过与该模型关联的任何销毁操作，除非`onDelete`参数设置为`CASCADE`且`hooks`参数设置为`true`。这是由于 Sequelize 需要逐行显式删除每个关联行，如果不小心可能会导致拥堵。

如果我们要编写一个简单的会计系统，并希望作为单独的账簿创建日志条目，我们首先定义我们的模型如下：

```js
class Account extends Model {}
Account.init({
    name: {
        type: DataTypes.STRING,
        primaryKey: true,
    },
    balance: DataTypes.DECIMAL,
});
class Book extends Model {}
Book.init({
    from: DataTypes.STRING,
    to: DataTypes.STRING,
    amount: DataTypes.DECIMAL,
});
```

然后，我们可以添加我们的`Ledger`模型，这是一个`Book`模型的副本，包含一个简单的引用列（为了简洁）和一个签名列，以指示交易已被外部来源批准：

```js
class Ledger extends Model {}
Ledger.init({
    bookId: DataTypes.INTEGER,
    signature: DataTypes.STRING,
    amount: DataTypes.DECIMAL,
    from: DataTypes.STRING,
    to: DataTypes.STRING,
});
```

为了自动化`Ledger`工作流程，我们可以在 Book 模型中添加一个`afterCreate`钩子来记录账户余额的变化：

```js
Book.addHook('afterCreate', async (book, options) => {
    const from = await Account.findOne(book.from);
    const to = await Account.findOne(book.to);
    // pretend that we have an external service that "signs" 
       our transactions
    const signature = await getSignatureFromOracle(book);
    await Ledger.create({
        transactionId: book.id,
        signature: signature,
        amount: book.amount,
        from: from.name,
        to: to.name,
    });
});
```

现在，当我们创建一个新的预订条目时，我们可以传递一个`transaction`引用，这样 Sequelize 就可以在同一个事务范围内执行生命周期范围内的查询。我们将在*第六章*中更深入地介绍事务，*使用 Sequelize 实现事务*，但到目前为止，我们将给出一个简单的示例，说明事务将是什么样子：

```js
const Sequelize = require('@sequelize/core');
const sequelize = new Sequelize('db', 'username',  
                                'password');
await sequelize.transaction(async t => {
    // validate our balances here and some other work…

    await Book.create({
        to: 'Joe',
        from: 'Bob',
        amount: 20.21,
    }, {
        transaction: t,
    });
   // double check our new balances
   await checkBalances(t, 'Joe', 'Bob', 20.21);
});
```

在生命周期事件中使用事务的好处是，如果事务工作流程的任何部分执行失败，我们可以停止其余的工作流程，而不会稀释我们数据库记录的质量。在没有设置`transaction`参数的前一个示例中，即使`checkBalances`方法返回错误且未提交事务，Sequelize 仍然会创建一个账簿条目。

注意

Sequelize 有时会为`findOrCreate`等方法使用其自身的内部事务。您始终可以用您自己的事务覆盖此参数。

现在我们已经掌握了向我们的模型添加生命周期事件的基础，我们可以开始更新我们的 Avalon Airlines 项目。

# 将所有这些放在一起

对于本节，我们只需要更新`BoardingTicket`模型（位于`models/boardingticket.js`），添加两个属性`cost`和`isEmployee`，以及我们登机座位工作流程的一些生命周期事件。让我们看看步骤：

1.  首先，我们需要在`init`方法中添加我们的属性，最终应该看起来像这样：

    ```js
      BoardingTicket.init({
        seat: {
          type: DataTypes.STRING,
          validate: {
            notEmpty: {
       msg: 'Please enter in a valid seating arrangement'
            }
          }
        },
        cost: {
          type: DataTypes.DECIMAL(7, 2)
        },
        isEmployee: {
          type: DataTypes.VIRTUAL,
          async get() {
            const customer = await this.getCustomer();
    if (!customer || !customer.email) 
                 return false;
       return customer.email.endsWith('avalonairlines');
          }
        }
      }, {
        sequelize,
        modelName: 'BoardingTicket'
      });
    ```

1.  在`init`函数下方，我们希望添加我们的生命周期事件。第一个将检查票是否被认为是员工票，如果是，则将小计标记为零：

    ```js
      // Employees should be able to fly for free
      BoardingTicket.beforeValidate('checkEmployee', 
                                    (ticket, options) => {
        if (ticket.isEmployee) {
           ticket.subtotal = 0;
        }
      });
    ```

1.  接下来，我们希望确保小计永远不会小于零（`beforeValidate`事件也适用于此处）：

    ```js
      // Subtotal should never be less than zero
      BoardingTicket.beforeSave('checkSubtotal', (ticket, options) => {
        if (ticket.subtotal < 0) {
          throw new Error('Invalid subtotal for this ticket.');
        }
      });
    ```

1.  对于我们模型的最后一个生命周期事件，我们希望检查客户是否选择了被认为是可用的座位：

    ```js
      // Ensure that the seat the customer has requested 
         is available
      BoardingTicket.beforeSave('checkSeat', async (tick
                                 et, options) => {
      // getDataValue will retrieve the new value (as 
         opposed to the previous/current value)
        const newSeat = ticket.getDataValue('seat');
        if (ticket.changed('seat')) {
          const boardingTicketExists = 
          BoardingTick-et.findOne({
            where: { seat: newSeat }
          });
          if (boardingTicketExists) {
            throw new Error(`The seat ${newSeat} has 
            al-ready been taken.`)
          }
        }
      });
    ```

1.  这些更改之后，每次我们创建一个新的登机牌，我们的应用程序现在在保存记录之前将执行三个生命周期事件。仅作参考，以下是我们如何将事务传递给`BoardingTicket`模型的示例：

    ```js
    await sequelize.transaction(async t => {
      await BookingTicket.create({
        seat: 'A1',
        cost: 12,
        customerId: 1,
      }, {
        transaction: t,
      });
    });
    ```

这就完成了本章对 Avalon Airlines 项目的必要更改。我们添加了一个检查小计和座位可用性的生命周期事件。我们还通过一个示例展示了如何将事务传递给特定的查询，我们将在下一章中进一步展开。

# 摘要

在本章中，我们介绍了生命周期事件是什么，以及如何在日常应用中使用它，Sequelize 提供了哪些生命周期事件以及它们的启动顺序，以及如何向 Sequelize 模型添加或删除生命周期事件。

在下一章中，我们将介绍事务的工作原理，它们的使用方式以及如何在 Sequelize 中进行配置。此外，下一章还将涵盖事务的不同锁定类型以及受管理和非受管理事务之间的区别。

# 参考文献

如果你在生命周期事件方面遇到问题，可以在此处找到快速参考：[`sequelize.org/master/manual/hooks.xhtml`](https://sequelize.org/master/manual/hooks.xhtml)。
