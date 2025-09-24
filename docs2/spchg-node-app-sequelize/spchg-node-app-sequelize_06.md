

# 使用 Sequelize 实现事务

在前面的章节中，我们介绍了如何使用生命周期事件、验证和约束来确保从我们的 Node.js 应用程序内部维护数据完整性。然而，这些方法并不能保证数据库内部数据的一致性。数据库提供了一种使用**事务**来原子化完整性的方法。

事务用于确保一个过程在没有中断（如连接失败或电源突然断电）的情况下完成。它们还用于隔离或锁定应用程序，防止并发地操作数据，从而减轻**竞争条件**问题。事务通过遵循**ACID**原则来保证数据的有效性，其中**ACID**代表**原子性**（“全有或全无”行为）、**一致性**（遵守约束）、**隔离性**（事务按顺序发生，彼此之间不知情）和**持久性**（持久存储）。

事务的一个通用用例是从一个用户账户向另一个用户账户转账。如果*用户 A*账户中有 30 个硬币，并且在*用户 A*决定从另一个用户那里购买价值 15 个硬币的商品之前，*用户 B*向*用户 A*收取了 20 个硬币，那么数据库应该防止*用户 A*因为余额不足而无法购买价值 15 个硬币的商品。数据库会看到*用户 A*在收取 15 个硬币之前已经被收取了 20 个硬币，然后会对第二个事务发出**回滚**。

注意

事务还提供了一种称为**保存点**的功能，它作为“数据库时间快照”，记录事务本身所做的更改，这对于多步事务非常有用。例如，在银行场景中，我们只交易货币本身，但在供应商的商店中，我们必须确保商品和货币处于适当的位置。

默认情况下，Sequelize 不会在事务下执行查询，但它确实提供了两种与事务交互的方法，分别称为管理事务和非管理事务。管理事务将自动/隐式地提交更改或回滚更改，具体取决于是否存在错误。非管理事务依赖于开发者调用适当的提交或回滚更改的方法。

在本章中，我们将涵盖以下内容：

+   深入探讨并举例说明管理事务和非管理事务

+   使用**延续本地存储**（**CLS**）进行部分事务

+   管理和配置高级事务选项，包括隔离级别

+   使用生命周期事件和锁与事务一起使用

具体来说，我们将探讨以下主题：

+   管理事务和非管理事务

+   并发运行事务

+   隔离级别和高级配置

+   将所有内容整合在一起

注意

您可以始终参考 Sequelize 的代码库，以维护这里可用的交易方法列表的最新状态：

[`github.com/sequelize/sequelize/blob/v6/src/transaction.js`](https://github.com/sequelize/sequelize/blob/v6/src/transaction.js)

# 技术要求

您可以在[`github.com/PacktPublishing/Supercharging-Node.js-Application-with-Sequelize/blob/main/ch6`](https://github.com/PacktPublishing/Supercharging-Node.js-Application-with-Sequelize/blob/main/ch6)找到本章的代码文件。

# 管理事务和非管理事务

对于有先前**对象关系映射**（**ORM**）经验的开发者来说，管理事务通常更容易，而对于直接编写**结构化查询语言**（**SQL**）的开发者来说，非管理事务可能更为熟悉。非管理事务的设计是显式的，但管理事务在状态管理方面有一些隐式行为，例如自动创建事务实例并使用该事务调用您的回调方法。

让我们看看创建非管理事务的步骤，如下所示：

1.  我们首先创建一个事务实例，如下所示：

    ```js
    const tx = await sequelize.transaction();
    ```

1.  接下来，我们将在 `try` 块中包装我们的查询。在这个例子中，我们将使用相同的事务实例增加和减少两个账户余额各 `100`，如下所示：

    ```js
    try {
        const amount = 100;
        await Account.increment(
            { balance: amount * -1 },
            {
                where: { id: 1 },
                transaction: tx
            }
        );
        await Account.increment(
            { balance: amount },
            {
                where: { id: 2 },
                transaction: tx
            }
        );
    ```

以下代码行将在前两个查询成功执行后提交我们的事务：

```js
await tx.commit();
```

1.  现在，我们可以关闭 `try` 块并添加一个 `catch` 块来处理事务中抛出的任何错误，如下所示：

    ```js
    } catch (error) {
      await tx.rollback();
      // log the error here
    }
    ```

`tx.rollback()` 命令会告诉数据库撤销此事务内所做的任何更改。无论是否有条件语句或错误处理，您都可以在任何时候回滚事务。

通过使用 *管理事务*，Sequelize 可以为您自动化很多这项工作。假设我们的 `Account` 模型有一个约束，即余额必须大于零，并且发送账户的余额中只有五枚硬币。我们可以将先前的非管理事务示例重写为管理事务，如下所示：

```js
try {
    const amount = 100;
    await sequelize.transaction(async (tx) => {
        await Account.increment(
            { balance: amount },
            {
                where: { id: 1 },
                transaction: tx
            }
        );

        await Account.increment(
            { balance: amount * -1 },
            {
                where: { id: 2 },
                transaction: tx
            }
        );
    });

    // the transaction has automatically been committed
} catch (error) {
    // Sequelize has already rolled back the transaction 
       from the try block
}
```

管理事务将根据是否有从适用查询抛出异常自动提交或回滚。您仍然可以在管理事务中通过在事务块内抛出错误来手动回滚，如下所示：

```js
try {
    await sequelize.transaction(async (tx) => {
        // some queries
        throw new Error("rolling back the transaction manu
                         ally here");       
        // some more queries
    });
} catch (error) {
    // rolling back the transaction manually here
}
```

有时，我们的应用程序可能需要运行不同的并发事务。我们可以递归地链式多个事务，或者使用名为 CLS 的模块。在下一节中，我们将介绍如何使用这两种方法使用并发事务。

# 并行运行事务

根据您的应用程序是否需要在数据库中读取和写入之间进行隔离，您可能需要显式地同时运行多个事务。Sequelize 提供了两种并行运行事务的方法：递归链式事务（Sequelize 的原生方法）或通过第三方模块 CLS 集成您的应用程序。

注意

SQLite 不支持同时运行多个事务。

## 使用 Sequelize 同时运行事务

我们可以通过链式调用两个事务方法来使用 Sequelize 同时运行事务，如下所示：

```js
sequelize.transaction((tx1) => {
    return sequelize.transaction((tx2) => {
```

现在，我们可以在使用不同事务的同时同时运行多个查询，如下所示：

```js
        return Promise.all([
            Account.create({ id: 1 }, { transaction: null }),
            Account.create({ id: 2 }, { transaction: tx1 }),
            Account.create({ id: 3 }, { transaction: tx2 }),
        ]);
    });
});
```

默认情况下，Sequelize 不会使用事务实例变量。如果我们省略了最后一个`Account.create`命令的`{ transaction: tx2 }`选项，那么 Sequelize 将不会使用事务实例变量，其行为将类似于第一个`Account.create`命令，即`{ transaction: null }`。

## 使用 CLS 运行事务

使用 CLS 与 Sequelize 结合可以帮助您自动将事务传递给所有查询，并提供类似线程局部存储的功能。CLS 自动执行事务的优势在于，某些数据库连接池驱动程序要求我们在网络上提交事务，这手动管理起来会非常繁琐。线程局部存储使我们能够在应用程序的不同部分创建的事务之间共享上下文。

注意

要了解更多关于 CLS 的信息，请参考项目的 Git 仓库：[`github.com/othiym23/node-continuation-local-storage`](https://github.com/othiym23/node-continuation-local-storage)。

本书没有将 CLS 与 Sequelize 集成，但为了完整性，我们将介绍如何为您的项目启用 CLS。我们需要安装必要的包，如下所示：

```js
npm install continuation-local-storage
```

然后，我们可以初始化一个 CLS 命名空间，如下所示：

```js
const cls = require('continuation-local-storage');
const namespace = cls.createNamespace('custom-sequelize-namespace');
```

然后，我们可以将`namespace`变量传递给 Sequelize 构造函数的`useCLS`方法，如下所示：

```js
const Sequelize = require('@sequelize/core');
Sequelize.useCLS(namespace);
const sequelize = new Sequelize(/* … */);
```

由于我们使用 Sequelize 构造函数，所有 Sequelize 的实例都将共享相同的命名空间。目前 Sequelize 不支持单独的 CLS 实例。

从先前的示例中借鉴，我们在查询中省略了事务参数的分类，如下例所示：

```js
sequelize.transaction((tx1) => {
    return sequelize.transaction((tx2) => {
        return Promise.all([
            Account.create({ id: 1 }, { transaction: null }),
            Account.create({ id: 2 }, { transaction: tx1 }),
            Account.create({ id: 3 }),
        ]);
    });
});
```

使用 CLS，Sequelize 将传递最内层作用域的事务实例变量，在先前的例子中将是`tx2`。如果我们省略第一个`Account.create`命令的`{ transaction: null }`选项，那么 Sequelize 将默认使用`tx2`作为其事务，就像在最后的`Account.create`命令中一样。第二个——中间的——`Account.create`命令仍然会显式使用`tx1`事务实例。

由于 Sequelize 自动将事务传递给查询，以下两个示例将使用相同的`tx`实例变量执行：

```js
await sequelize.transaction(async () => { // the tx
argument is not required
        await removeUserInventory(id);
        await User.destroy({ where: { id } }); // tx is also 
        used here
});
async function removeUserInventory(id) {
    // this query will also use the same scope tx variable 
       as User.destroy
    await UserInventory.destroy({ where: { userId: id } });
}
```

如您所见，启用 CLS 与 Sequelize 结合使用可以提供一些优势，并更好地组织项目的代码库。

注意

要获得 CLS 的更多功能，请参考名为 CLS-Hooked 的项目：

[`github.com/jeff-lewis/cls-hooked`](https://github.com/jeff-lewis/cls-hooked)

# 隔离级别和高级配置

在本节中，我们将介绍 Sequelize 为每种类型的事务提供的不同隔离级别和配置选项。对于管理事务，方法签名如下：`sequelize.transaction(options, callback)`。非管理事务的签名是 `sequelize.transaction(options)`。

下面是两种事务类型可配置选项的列表：

+   `type`—SQLite 中的一个选项，用于设置事务类型。可能的值有 `DEFERRED`（默认值）、`IMMEDIATE` 和 `EXCLUSIVE`。有关更多信息，请参阅 [`www.sqlite.org/lang_transaction.xhtml`](https://www.sqlite.org/lang_transaction.xhtml)。

+   `isolationLevel`—设置事务的隔离级别。以下说明是在 MySQL 的上下文中，但应适用于其他数据库。`READ_UNCOMMITTED`—使用非锁定机制读取数据。这可能导致并发问题，使用已回滚的其他事务中的过时或无效数据。

+   `READ_COMMITTED`—即使在同一事务内部也执行一致的读取。换句话说，读取数据将与同一事务内部先前查询执行的更新保持一致。

+   `REPEATABLE_READ`—在读取信息方面类似于 `READ_COMMITTED`，但在 MySQL 的 InnoDB 数据库引擎方面有一些具体规定。有关如何一致性地锁定记录的更多信息，请参阅 [`dev.mysql.com/doc/refman/5.7/en/innodb-transaction-isolation-levels.xhtml#isolevel_repeatable-read`](https://dev.mysql.com/doc/refman/5.7/en/innodb-transaction-isolation-levels.xhtml#isolevel_repeatable-read)。

+   `SERIALIZABLE`—比 `REPEATABLE_READ` 更严格的规则集。此隔离级别通常用于解决数据库的并发和死锁相关的问题。

+   `deferrable`—仅适用于 PostgreSQL，此设置确定约束是否可以延迟或立即检查。*   `logging`—一个函数，Sequelize 将传递查询及其参数作为参数。

Sequelize 提供了用于设置隔离级别的常量变量，以便于使用，如下所示：

```js
const Sequelize = require('@sequelize/core');
sequelize.transaction({
    isolationLevel: Sequelize.Transaction.ISOLATION_LEVELS.
                    SERIALIZABLE
}, (tx) => { /* ... */ });
```

我们还可以通过在初始化 Sequelize 时设置 `isolationLevel` 选项，在实例级别上设置事务的隔离级别，如下所示：

```js
new Sequelize('db', 'user', 'pw', {
    isolationLevel: Sequelize.Transaction.ISOLATION_LEVELS.
                    READ_COMMITTED
});
```

现在，任何后续的事务都将默认使用 `READ_COMMITTED` 级别。这些隔离级别与读取数据相关。在下一节中，我们将介绍写入数据库的锁定机制。

## 使用 Sequelize 锁定行

有时，我们的应用程序需要我们在执行事务时暂时锁定信息，并防止其他事务提交到同一表或行。数据库从业者可能知道这种机制称为 `SELECT FOR UPDATE` 查询。您可以在以下代码片段中看到一个此类查询的示例：

```js
sequelize.transaction((tx) => {
    const seat = Seats.findOne({
        where: { venue: 1, row: 5, seat: 13 }
        transaction: tx,
        lock: true
    });
    // ... more queries ...
});
```

假设每个`row`和`seat`实例在每个`venue`实例中都是唯一的，前面的示例将锁定该特定座位的记录，直到事务提交或回滚。

假设我们想要检索一个不在挂起交易中的座位列表。如果我们的数据库管理系统支持`SKIP LOCKED`命令，那么在查询数据时可以使用`skipLocked: true`配置选项。为了演示`SKIP LOCKED`功能，我们可以首先为特定座位添加一个锁定记录，如下所示：

```js
const tx1 = await sequelize.transaction();
const seat = Seats.findOne({
    where: { venue: 1, row: 5, seat: 13 }
    transaction: tx1,
    lock: true
});
```

接下来，以下查询将使用`SKIP LOCKED`规则并返回任何由其他挂起交易（在本例中为行 5，座位 13）未锁定的适用行：

```js
const tx2 = await sequelize.transaction();
const seats = Seats.findAll({
    where: { venue: 1 }
    transaction: tx2,
    lock: true,
    skipLocked: true
});
```

注意

MySQL 从 8.0.1 版本开始支持`SKIP LOCKED`。本书的代码库不需要`skipLocked`，但如果您使用的是较旧版本并尝试使用`skipLocked`，那么 Sequelize 将默默地从查询中省略`SKIP LOCKED`命令，可能会导致意外的行为或结果。

## 使用生命周期事件进行事务

Sequelize 目前只为事务提供一种生命周期——即使是事务——明确地。这个名为`afterCommit`的生命周期事件可以用于管理和非管理事务。如果事务回滚，则不会触发此事件，并且该事件不能修改其事务对象（与传统生命周期事件不同）。

要调用`afterCommit`钩子，我们可以将事件添加到事务实例中，如下所示：

```js
sequelize.transaction((tx) => {
    tx.afterCommit((trx) => {
        // … your logic here ...
    });
    // ... queries using tx ...
});
```

我们可以通过`afterSave`事件将`afterCommit`事件附加到整个模型上。使用`afterCommit`的一个好例子是将序列化数据发送到其他服务、应用程序、区块链数据库等。以下是如何使用`afterCommit`的示例：

```js
Seats.afterSave((instance, options) => {
    if (options.transaction) {
        // appending afterCommit to the transaction instance here
        options.transaction.afterCommit(() => { /* your logic here */ });
        return;
    }
    // code will continue here if we did not save under a transaction
});
```

到目前为止，我们已经介绍了 Sequelize 提供的不同类型的事务，事务的命名空间环境，隔离级别，锁定和生命周期事件。通过这些技能的组合，我们最终可以开始将一些知识应用到我们的应用程序中。

# 整合所有内容

现在我们已经了解了使用 Sequelize 进行事务的核心原则，我们可以开始向我们的*Avalon Airlines*项目添加内容。我们的业务伙伴刚刚通知我们，投资者想要一个预订航班而不处理支付的简单演示。为此任务，我们需要添加几个新文件，更新`BoardingTicket`和`FlightSchedule`模型，向我们的 Express 应用程序添加新路由，并安装一个新的 Node.js 包。

首先，让我们开始添加项目所需的新 Node.js 包。这个包被称为 Luxon ([`moment.github.io/luxon/`](https://moment.github.io/luxon/))，它是一个日期和时间 JavaScript 库。使用以下命令添加包：

```js
npm i --save luxon
```

接下来，我们将想要修改位于`models/boardingticket.js`中的`BoardingTicket`模型中的一个生命周期事件，通过添加/更改以下突出显示的代码：

```js
BoardingTicket.beforeSave('checkSeat', async (ticket, options) => {
    const newSeat = ticket.getDataValue('seat');
    const { transaction } = options;
    if (ticket.changed('seat')) {
      const boardingTicketExists = await BoardingTicket.findOne({
        where: {
          seat: newSeat
        },
        transaction,
      });
      if (boardingTicketExists !== null) {
        throw new Error(`The seat ${newSeat} has already been taken.`);
      }
    }
  });
```

我们还需要更新另一个模型，即位于`models/flightschedule.js`中的`FlightSchedule`模型。在文件顶部添加以下代码行：

```js
const { DateTime } = require('luxon');
```

然后，在`validDestination`方法下方添加另一个验证到`validate`对象中，如下所示：

```js
      validateDepartureTime() {
        const dt = DateTime.fromJSDate(this.departureTime);
        if (!dt.isValid) {
          throw new Error("Invalid departure time");
        }
        if (dt < DateTime.now()) {
          throw new Error("The departure time must be set 
                           within the future");
        }
      },
```

现在，我们可以在项目主目录中添加一个新文件夹和文件`routes/flights.js`，并添加以下代码行以加载适当的模块和文件：

```js
const { DateTime } = require("luxon");
const models = require("../models");
```

对于我们第一个与航班相关的航线，我们需要找到一种方法来首先创建我们的飞机。查看`models/airplane.js`中的属性，我们可以确定我们需要一个型号名称和每架飞机的座位数。代码如下所示：

```js
async function createAirplane(req, res) {
    const { name, seats } = req.body;
```

在`POST`数据中，我们期望发送`name`和`seats`值到我们的 Express 应用程序。现在，我们可以添加飞机创建逻辑，以及关闭和导出函数，如下所示：

```js
    try {
        const airplane = await models.Airplane.create({
            planeModel: name,
            totalSeats: seats,
        });
        return res.json(airplane);
    } catch (error) {
        res.status(500).send(error);
    }
}
exports.createAirplane = createAirplane;
```

我们下一个函数将是用于创建航班时刻表。我们需要`airplaneId`、`origin`、`destination`和`departure` `POST`值来创建航班时刻表，如下所示：

```js
async function createSchedule(req, res) {
    const { airplaneId, origin, destination, departure } = 
    req.body;
```

首先，让我们验证并解析出发时间到一个本地的`DateTime`对象，如下所示：

```js
        const dt = DateTime.fromISO(departure);
        if (!dt.isValid) {
            return res.status(400).send("invalid departure 
                                         time");
        }
```

接下来，我们将想要检查飞机是否实际存在，因此我们将执行以下代码来找出答案：

```js
    try {
 const plane = await models.Airplane.findByPk(airplaneId);
        if (!plane) {
            return res.status(404).send("airplane does not 
                                         exist");
        }
```

如果飞机确实存在，我们将想要为它创建一个航班时刻表。我们将创建过程封装在一个事务中，以确保创建时刻表以及将时刻表与特定的飞机关联不会产生错误。

对于这个特定的演示，事务不是必需的，但在实际应用中，我们想要确保根据航线和出发时间，飞机不会被过度预订。事务块看起来如下所示：

```js
        const flight = await sequelize.transaction(async 
        (tx) => {
   const schedule = await models.FlightSchedule.create({
                originAirport: origin,
                destinationAirport: destination,
                departureTime: dt,
            }, { transaction: tx });
```

然后，我们设置相关的飞机，返回时刻表记录，完成事务，并使用**JavaScript 对象表示法**（**JSON**）数据渲染响应，如下所示：

```js
            await schedule.setAirplane(plane, 
            { transaction: tx });
            return schedule;
        });
       return res.json(flight);
```

文件的最后部分是捕获之前`try`块中的任何错误，并导出`createSchedule`函数，如下所示：

```js
    } catch (error) {
        return res.status(500).send(error);
    }
}
exports.createSchedule = createSchedule;
```

现在，我们可以在`routes/tickets.js`中创建一个新的文件，该文件将作为预订实际航班的路由。出于演示目的，我们将省略确定价格和客户会话等复杂功能，并用常量值填充这些细节。创建文件后，我们将在文件顶部加载我们的模型，如下所示：

```js
const models = require("../models");
```

对于我们的`bookTicket`方法，我们需要一个`scheduleId`和`seat` `POST`参数，以及为创建登机牌打开一个事务。以下是我们可以添加这些内容的示例：

```js
async function bookTicket(req, res) {
    try {
        const { scheduleId, seat } = req.body;
        const t = await sequelize.transaction(async (tx) => {
```

通过执行以下代码检查`FlightSchedule`模型是否存在：

```js
const schedule = await models.FlightSchedule.findByPk
(scheduleId, {transaction: tx});
            if (!schedule) {
                throw new Error("schedule could not be 
                                 found");
            }
```

让我们创建一个新的登机牌，如下所示：

```js
const boardingTicket = await models.BoardingTicket.create({
                seat,
            }, { transaction: tx });
```

现在，我们将设置登机牌的关联，并在完成事务的同时返回登机牌，如下所示：

```js
            // this is where we would set a customer if we had an application with authentication, etc.
            // await ticket.setCustomer(customerId, { transaction: tx });
            await schedule.addBoardingTicket(boardingTicket, { transaction: tx });

            return boardingTicket;
        });
        return res.json(t.toJSON());
```

捕获任何错误并导出函数，如下所示：

```js
    } catch (error) {
        return res.status(400).send(error.toString());
    }
}
exports.bookTicket = bookTicket;
```

接下来，我们希望添加一个名为 `body-parser` 的模块，该模块有助于转换 Express 中的不同 `req.body`。更多信息，请访问 [`github.com/expressjs/body-parser`](https://github.com/expressjs/body-parser)。我们可以使用以下命令安装并将包添加到我们的 `package.json` 文件中：

```js
npm i --save body-parser
```

我们最后要编辑的文件将是项目主目录中的 `index.js` 文件。我们希望在第一行引入 express 模块之后添加以下模块：

```js
const bodyParser = require("body-parser");
```

在 `const models = require("./models");` 行之下，我们希望添加我们的新导出函数。这是我们的做法：

```js
const { bookTicket } = require("./routes/tickets")
const { createAirplane, createSchedule } = require("./routes/flights");
```

在第一个路由 `app.get('/', …)` 之上，添加以下代码行以支持 JSON `POST`：

```js
app.use(bodyParser.json({ type: 'application/json' }));
```

接下来，我们希望在 `app.get('/airplanes/:id', ...)` 行之上添加以下代码行以创建 `createAirplane` 路由：

```js
app.post('/airplanes', createAirplane);
```

然后，我们可以在 `app.listen(3000, ...)` 行之上添加我们剩余的新路由，如下所示：

```js
app.post('/schedules', createSchedule);
app.post('/book-flight', bookTicket);
```

由于我们所有的更改都已提交，我们现在可以通过执行以下命令来运行我们的应用程序：

```js
npm run start
```

为了测试我们的应用程序，我们可以使用 cURL 或任何 HTTP REST 工具，如 Postman ([`www.postman.com/`](https://www.postman.com/)) 或 HTTPie ([`httpie.io/`](https://httpie.io/))。在创建飞行日程安排之前，让我们创建一架新的飞机，如下所示：

```js
curl -X POST -H "Content-Type: application/json" -d "{\"name\": \"A320\", \"seats\": -1}" http://127.0.0.1:3000/airplanes
```

我们应该会看到类似以下响应：

```js
{"name":"SequelizeValidationError","errors":[{"message":"A plane must have at least one seat","type":"Validation error","path":"totalSeats","value":-1,"origin":"FUNCTION","instance":{"id":null,"planeModel":"A320","totalSeats":-1,"updatedAt":"2022-02-21T16:27:18.336Z","createdAt":"2022-02-21T16:27:18.336Z"},"validatorKey":"min","validatorName":"min","validatorArgs":[1],"original":{"validatorName":"min","validatorArgs":[1]}}]}
```

这个特定的 A320 型号一次最多有 150 个座位可供客户使用。当我们调整可用的总座位数时，我们的新命令将如下所示：

```js
curl -X POST -H "Content-Type: application/json" -d "{\"name\": \"A320\", \"seats\": 150}" http://127.0.0.1:3000/airplanes
```

之前的命令应该返回类似以下内容的响应：

```js
{"id":1,"planeModel":"A320","totalSeats":150,"updatedAt":"2022-02-21T15:49:19.883Z","createdAt":"2022-02-21T15:49:19.883Z"}
```

在创建日程安排的下一个命令时，我们将想要记住 `id` 值：

```js
curl -X POST -H "Content-Type: application/json" -d "{\"airplaneId\": 1, \"origin\": \"LAX\", \"destination\": \"ORD\", \"departure\": \"2060-01-01T14:00:00Z\"}"  http://127.0.0.1:3000/schedules
```

之前的请求应该导致类似以下内容的错误：

```js
{"name":"SequelizeValidationError","errors":[{"message":"Invalid destination airport","type":"Validation error","path":"destinationAirport","value":"ORD","origin":"LAX","instance":{"id":null,"originAirport":"LAX","destinationAirport":"ORD","updatedAt":"2022-02-21T18:11:02.108Z","createdAt":"2022-02-21T18:11:02.108Z"},"validatorKey":"isIn","validatorName":"isIn","validatorArgs":[["MIA","JFK","LAX"]],"original":{"validatorName":"isIn","validatorArgs":[["MIA","JFK","LAX"]]}}]}
```

我们目前不飞往芝加哥的奥黑尔国际机场！新的目的地现在将是使用 `MIA` 代码的迈阿密，如下所示：

```js
curl -X POST -H "Content-Type: application/json" -d "{\"airplaneId\": 1, \"origin\": \"LAX\", \"destination\": \"MIA\", \"departure\": \"2060-01-01T14:00:00Z\"}" http://127.0.0.1:3000/schedules
```

响应应该类似于以下内容：

```js
{"id":1,"originAirport":"LAX","destinationAirport":"MIA","departureTime":"2060-01-01T14:00:00.000Z","updatedAt":"2022-02-21T18:34:46.049Z","createdAt":"2022-02-21T18:34:46.038Z","AirplaneId":1}
```

对于预订请求，我们需要之前响应的 `id` 值和座位分配，如下所示：

```js
curl -X POST -H "Content-Type: application/json" -d "{\"scheduleId\": 1, \"seat\": \"1A\"}" http://127.0.0.1:3000/book-flight
```

响应将类似于以下内容：

```js
{"isEmployee":{},"id":1,"seat":"1A","updatedAt":"2022-02-21T18:55:30.837Z","createdAt":"2022-02-21T18:55:30.837Z"}
```

如果我们重复执行之前的命令，会显示一个错误消息，提示我们座位已被占用，如下所示：

```js
Error: The seat 1A has already been taken.
```

这就完成了我们对 *Avalon Airlines* 项目的更改。我们实现了一种创建飞机和新的飞行日程以及使用事务分配登机牌的方法。这应该完成我们下一次投资者会议的要求。

# 摘要

在本章中，我们通过使用 CLS 进行全局作用域事务、支持的隔离级别、适用的生命周期事件和锁定事务，介绍了托管事务和无托管事务之间的区别。

在下一章中，我们将介绍如何直接从 Sequelize 处理自定义的 JSON 和 **二进制大对象** (**BLOB**) 数据到数据库管理系统（DBMS）。下一章还将包含完成 *Avalon Airlines* 项目的高级指导。
