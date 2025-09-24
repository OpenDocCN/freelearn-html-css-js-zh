# 7

# 处理自定义、JSON 和 Blob 数据类型

一些数据库管理系统提供了一种存储特定列类型（如 JSON 和 Blob 相关数据）的方法。这些列类型对于快速原型设计、处理无模式数据和发送接收缓冲数据非常有用。

通常，应用程序会使用**NoSQL**数据库，如 MongoDB，来处理和查询 JSON 文档，但这带来了一系列自己的问题。如果没有详尽的验证列表，我们就不能再坚持某种形式的规范化结构，NoSQL 数据库也无法执行事务或提供 ACID 兼容的功能。

注意

一些 NoSQL 数据库声称提供 ACID 兼容性，但它们通常附带一些规定和限制，例如单次事务中可以更新的最大文档数，或者事务不能超过某个时间窗口；否则，您将失去 NoSQL 数据库相对于 SQL 数据库的所有性能优势。

JSON 和 Blob 列数据类型有几种用例。使用 JSON，您可以存储一组非确定性的值记录集，这对于创建交易收据和审计系统等用例非常有用。Blob 列数据类型可以存储任何有助于集中检索和插入的文件，但内部，DBMS 可能会对该文件进行分片或分发。

通常，由于失去外部访问控制列表、阻塞写前日志文件以及存储这些文件时的虚假安全感，我们不推荐在 DBMS 中存储文件。我们还可能遇到页面大小增加的情况，这将增加检索记录所需的时间。一般来说，对于快速原型设计处理文件，使用 DBMS 是可以的，但不适用于生产环境。

注意

使用 JSON 列类型进行审计的一个例子是 PGAudit 的 Postgres 扩展。此扩展将转换之前的和新的记录集为 JSON 数据类型以存储不同的值。您可以参考[`www.pgaudit.org/`](https://www.pgaudit.org/)了解更多关于其工作原理的信息。

Sequelize 能够处理所有支持 DBMS 的自定义和 Blob 类型，以及 SQLite、MySQL、MariaDB 和 PostgreSQL 的 JSON 列类型。对于 MSSQL 有一个解决方案，将在本章的*与 JSON 一起工作*部分详细解释。

在本章中，我们将涵盖以下内容：

+   查询 JSON 和 JSONB 数据

+   使用`BLOB`列类型

+   创建自定义数据类型

# 技术要求

您可以在[`github.com/PacktPublishing/Supercharging-Node.js-Application-with-Sequelize/blob/main/ch7`](https://github.com/PacktPublishing/Supercharging-Node.js-Application-with-Sequelize/blob/main/ch7)找到本章的代码文件。

# 查询 JSON 和 JSONB 数据

如前所述，JSON 列类型仅适用于 SQLite、MySQL、MariaDB 和 PostgreSQL。JSONB 列类型仅支持 PostgreSQL 数据库管理系统。这两种列类型之间的区别在于，JSONB 将在内部存储与 JSON 文档中的字段相关的附加信息。这将增加磁盘空间的需求，但将有助于使数据查询更快。

对于本节，假设我们在应用程序中有一个以下模型：

```js
class Receipts extends Model {}
Receipts.init({
  receipt: DataTypes.JSON
});
```

现在，我们可以创建我们的文档：

```js
await Receipts.create({
    receipt: {
        name: {
            first: "Bob",
            last: "Smith"
        },
        items: [
            {
                sku: "abc123",
                quantity: 10
            },
            {
                sku: "xyz321",
                quantity: 1
            }
        ],
        subtotal: 100
    }
});
```

我们现在可以使用传统的 Sequelize 方法查询我们的文档：

```js
await Receipts.findOne({
    where: {
        receipt: {
            name: {
                first: "Bob",
                last: "Smith"
            }
        }
    }
});
```

或者我们可以使用特殊的点符号风格：

```js
await Receipts.findOne({
    where: {
        "receipts.name.first": "Bob",
        "receipts.name.last": "Smith"
    }
});
```

点符号方法也适用于其他查找属性，例如 `order`：

```js
const receipts = await Receipts.findAll({
    where: {
      receipt: {
        name: {
          last: "Smith",
        },
      },
    },
    order: [
      ["receipt.name.first"]
    ]
});
```

在更新记录时，我们必须像传统的 NoSQL 文档存储系统一样重新插入整个文档：

```js
await Receipts.update({
    receipt: {
        name: {
            first: "Bob",
            last: "Smith"
        },
        items: [
            {
                sku: "abc123",
                quantity: 10
            },
            {
                sku: "xyz321",
                quantity: 1
            }
        ],
        subtotal: 120
    }
  }, {
    where: {
      "receipt.name.first": "Bob"
    }
});
```

如果我们想要查询数组中的值，我们可能需要使用 `Sequelize.literal` 函数，如果我们的数据库管理系统没有本地支持 `Op.contains` 操作符（仅 PostgreSQL）。以下是一个使用 PostgreSQL 的 `@>` 操作符查询数组值的示例：

```js
const receipts = await Receipts.findAll({
    where: {
        receipt: {
            items: {
                [Op.contains]: {
                    sku: "abc123"
                }
            }
        }
    }
});
```

由于 MySQL 不支持 `contains` 操作符，上一个查询的等效操作如下：

```js
const receipts = await Receipts.findAll({
    where: Sequelize.literal(`JSON_CONTAINS(JSON_EXTRACT
        (receipt, '$.items[*].sku', '"abc123"')`)
});
```

MSSQL 也可以执行 JSON 的基本操作。以下是一个使用 MSSQL 和 Sequelize 查询 JSON 数据的示例：

```js
class Users extends Model {}
Users.init({
    metadata: DataTypes.STRING
});
await Users.create({
    metadata: JSON.stringify({
        first_name: "Bob",
        last_name: "Smith"
    })
});
await Users.findAll({
    where: sequelize.where(
        sequelize.fn('JSON_VALUE', sequelize.col('metadata'), '$.first_name'),
        'Bob'
    )
});
```

不幸的是，对于 MSSQL，要搜索嵌套数组需要交叉连接和更多超出本书范围的主题，例如 OpenJSON（可以在 [`docs.microsoft.com/en-us/sql/t-sql/functions/openjson-transact-sql?view=sql-server-ver15`](https://docs.microsoft.com/en-us/sql/t-sql/functions/openjson-transact-sql?view=sql-server-ver15) 中参考）。

# 使用 BLOB 列类型

有时，我们的应用程序将需要我们存储缓冲区或二进制数据到我们的系统中。以下是一个使用 Sequelize 创建和读取二进制数据的快速示例：

1.  我们将从我们的定义开始：

    ```js
    class Users extends Model {}
    Users.init({
        avatar: DataTypes.BLOB,
        keycode: DataTypes.BLOB
    });
    ```

1.  接下来，我们可以插入我们的记录，如下所示：

    ```js
    await Users.create({
        avatar: require("fs").readFileSync
                ("/some/path/avatar.jpg"),
        keycode: Buffer.from("secretpassword")
    });
    ```

1.  要检索和使用缓冲数据，我们可以简单地使用查找方法，并直接使用 Node.js 的 `fs` 模块写入：

    ```js
    const user = await Users.findOne({});
    require("fs").writeFileSync(
        "/some/path/to/write/avatar.jpg",
        user.avatar
    );
    ```

现在我们已经了解了所有内置的数据类型，我们现在可以开始创建我们自己的自定义数据类型。自定义数据类型也可以用于将几个验证组合在一起或创建几个规则集到一个数据类型中。

# 创建自定义数据类型

Sequelize 通过扩展 `DataTypes.ABSTRACT` 抽象类为我们提供了一种创建自定义类型的方法。这允许我们保持代码库更加有序和一致。假设我们的应用程序需要大量遵守自然数法则的列。一个快速演示如下：

```js
class Stats extends Model {}
Stats.init({
    A: {
        type: Sequelize.INTEGER(11).UNSIGNED.ZEROFILL,
        validate: {
            min: 1
        }
    },
    B: {
        type: Sequelize.INTEGER(11).UNSIGNED.ZEROFILL,
        validate: {
            min: 1
        }
    },
    C: {
        type: Sequelize.INTEGER(11).UNSIGNED.ZEROFILL,
        validate: {
            min: 1
        }
    },
});
```

如果我们有数百个这样的列，写出这些列可能会很繁琐。解决这个问题的方法之一是创建我们自己的自定义属性。让我们看看步骤：

1.  第一步是扩展 `ABSTRACT` 类：

    ```js
    class NATURAL_NUMBER extends DataTypes.ABSTRACT {
    ```

1.  然后，我们需要告诉 Sequelize 如何将此数据类型转换为列类型。我们可以在类中定义一个 `toSql` 方法来完成此操作：

    ```js
        toSql() {
            return 'INTEGER(11) UNSIGNED ZEROFILL'
        }
    ```

这将告诉 Sequelize 我们想要一个零填充的无符号整数。

1.  接下来，我们可以通过创建一个 `validate` 方法来强制执行验证规则：

    ```js
        validate(value, options) {
            const isNumber = Number.isInteger(value);
            const isAboveZero = Number.parseInt(value) > 0;

            return isNumber && isAboveZero;
        }
    ```

Sequelize 将自动检查此属性类型的值是否为整数且大于零。

1.  下一个步骤是可选的，但为了完整性，以下方法分别用于写入和从数据库读取：

    ```js
        _stringify(value) {
          return value.toString();
        }
        static parse(value) {
          return Number.parseInt(value);
        }
    ```

`_stringify` 方法将在将值发送到您的数据库之前将其转换为字符串，而 `parse` 方法将转换从数据库返回的值。

1.  现在，我们可以关闭我们的类并调用一些强制性的方法：

    ```js
    }
    NATURAL_NUMBER.prototype.key = NATURAL_NUMBER.key = 'NATURAL_NUMBER';
    DataTypes.NATURAL_NUMBER = Sequelize.Utils.classToInvokable(NATURAL_NUMBER);
    ```

Sequelize 将通过映射出类中的 `key` 值来识别您的属性数据类型。下一行将添加您的自定义数据类型到 Sequelize 的 `DataTypes` 命名空间。`classToInvokable` 方法将简单地包装您的类构造函数并返回一个新实例，这样您在定义模型时就不必显式调用 `new DataTypes.NATURAL_NUMBER()`。

1.  现在，我们可以定义我们之前的模型，如下所示：

    ```js
    class Stats extends Model {}
    Stats.init({
        A: DataTypes.NATURAL_NUMBER,
        B: DataTypes.NATURAL_NUMBER,
        C: DataTypes.NATURAL_NUMBER
    });
    ```

当我们去创建或更新时，我们的属性将遵守我们之前设置的规则。以下三个示例将返回验证错误，因为 `C` 列的值不是一个自然数：

```js
await Stats.create({
    A: 100,
    B: 20,
    C: "NotANumber" // not an number
});
await Stats.create({
    A: 100,
    B: 20,
    C: 1.1 // not an integer
});
await Stats.create({
    A: 100,
    B: 20,
    C: -3 // not a natural number
});
```

当我们将 `C` 改为一个自然数（如下面的代码所示）时，我们的查询现在将成功创建记录：

```js
await Stats.create({
    A: 100,
    B: 20,
    C: 10
}); // success!
```

到目前为止，我们已经介绍了如何使用 Sequelize 内置类处理 JSON 和 BLOB 数据类型。我们还通过扩展 Sequelize 的 `ABSTRACT` 数据类型类创建了自定义数据类型。现在，我们可以在我们的项目中开始使用这些数据类型。

现在我们对如何显式处理 JSON 数据类型有了更好的理解，我们可以在 Avalon Airlines 项目中使用该类型。

# 将所有这些放在一起

我们的商业伙伴刚刚通知我们，我们希望能够记录每个适用事件的交易收据。这可能包括登机牌、额外行李或额外的水瓶，这意味着我们的数据没有确定的结构。为此任务，我们需要生成一个新的模型 `Receipts` 并更新我们的 `BoardingTicket` 模型。以下是步骤：

1.  首先，我们可以开始生成一个新的模型，名为 `Receipts`，用于存储交易事件：

    ```js
    sequelize-cli model:generate --name Receipts --attributes receipt:json
    ```

1.  然后，运行我们的迁移：

    ```js
    sequelize db:migrate
    ```

1.  接下来，我们希望在 `models/boardingticket.js` 中的 `BoardingTicket` 模型中添加另一个生命周期事件，方法是在 `module.exports` 块的末尾添加以下代码：

    ```js
      BoardingTicket.afterSave('saveReceipt', 
          async(ticket, options) => {
        await sequelize.models.Receipts.create({
          receipt: ticket.get()
        }, {
          transaction: options.transaction
        });
      });
    ```

这就完成了我们对 Avalon Airlines 项目的更改。我们使用 JSON 实现了一个新的存储收据数据的模型，并在创建或更新 `BoardingTicket` 模型后添加了一个生命周期事件。这应该完成我们下一次投资者会议的要求。

# 摘要

在本章中，我们探讨了使用特定数据类型（如 JSON 和 `BLOB`）读取和写入属性的不同方法。我们还学习了如何通过扩展 `ABSTRACT` 类来创建自定义数据类型，以创建一个更易于维护的代码库。

在下一章中，我们将介绍如何监控和记录您应用程序的查询。下一章还将包含完成 Avalon 航空公司项目的进一步说明。

# 第三部分 – 高级查询、使用适配器和日志查询

在本部分中，您将了解如何监控和衡量您应用程序的性能指标。您将使用与 Sequelize 和日志查询集成的第三方应用程序。您还将学习如何将您的应用程序部署到云应用程序平台。

本部分包括以下章节：

+   *第八章*，*记录和监控您的应用程序*

+   *第九章*，*使用和创建适配器*

+   *第十章*，*部署 Sequelize 应用*
