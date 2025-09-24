# 2

# 定义和使用 Sequelize 模型

对于我们在上一章中介绍的 *Avalon Airlines* 项目，我们需要指导我们的应用程序如何定义我们数据库的图。数据库可以有多种角色和应用，但只有一个目的，那就是组织我们的数据（存储是文件系统的职责）。在我们开始在 Node.js 应用程序中定义模型之前，我们需要从项目角度考虑我们的业务逻辑和模型（每个项目都会有不同的要求）。大多数项目都会以一种方式结构化其图，将 *组织*（例如，客户、员工、供应商和公司）以及 *事物*（如产品、飞机和交易收据）进行分类。

`snake_case` 或 `PascalCase` 模式）。模型之间的关系或关联将由 Sequelize 自动创建和管理。还可以建立业务逻辑工作流程，这样您就不必记住像 *如果客户取消了行程，则删除客户的登机牌* 这样的工作流程。这部分将由一个组织良好的地方处理，而不是在每个取消行程的代码部分调用 `RemoveBoardingPass(...)`（无论该方法是从客户、员工等调用）。本章将教会您如何定义和同步您的模型与数据库，以及如何使用 Sequelize 将数据应用到 Node.js 运行时应用程序。这将是我们操作 Sequelize 的初始基础。

本章将向您介绍以下概念：

+   定义数据库模型

+   探索各种 Sequelize 数据类型及其使用时机

+   将 Sequelize 中的图和数据进行迁移到数据库

+   使用 Sequelize 操作和查询数据

+   定义模型的 Sequelize 高级选项

# 技术要求

您可以在 GitHub 上找到本章中包含的代码文件，地址为 [`github.com/PacktPublishing/Supercharging-Node.js-Applications-with-Sequelize/tree/main/ch2`](https://github.com/PacktPublishing/Supercharging-Node.js-Applications-with-Sequelize/tree/main/ch2)

# 为数据库定义模型

在本节中，我们将简要概述我们项目的要求，并确定我们需要定义哪些类型的模型。之后，我们将从 Sequelize **命令行界面**（**CLI**）工具运行一个脚本生成命令，并检查模型定义的基本结构。

对于 *Avalon Airlines*，我们将从以下 *组织* 和 *事物* 开始建模：

+   飞机

+   客户

+   飞行计划

+   登机牌

每个模型将在数据库中拥有自己的表。我们最终将把这些模型或表与列、索引、验证和其他模型的关系关联起来。目前，我们将在 Node.js 应用程序中使用 Sequelize 定义、选择（或查询）、插入、更新和删除这些表中的数据。如果您正在处理一个已经存在数据库的现有项目，那么“使用 Sequelize 操作和查询数据”这一部分将比从零开始的项目更有相关性。

我们首先将使用 Sequelize CLI 工具生成满足列的最小要求的模型。然后，我们将回顾 CLI 生成的代码，以便您更熟悉如何在不依赖 CLI 的情况下定义 Sequelize 模型。在项目的根目录下使用以下命令生成之前提到的模型：

```js
sequelize model:generate --name Airplane --attributes planeModel:string,totalSeats:integer
sequelize model:generate --name Customer --attributes name:string,email:string
sequelize model:generate --name FlightSchedule --attributes originAirport:string,destinationAirport:string,departureTime:date
sequelize model:generate --name BoardingTicket --attributes seat:string
```

您可能已经注意到我们为模型名称使用了单数名词。Sequelize 会自动为我们将关联的表和模型复数化。您可以通过 Sequelize 配置设置来禁用此行为，这将在本章后面详细讨论。对于我们的 `BoardingTickets` 模型，我们将在下一章生成客户和航班计划的关联，但就目前而言，我们可以为表构建最基本的结构。

提示

Sequelize 为开发者提供了一些有用的实用函数。该框架使用一个名为 `Sequelize` 的库。

打开 `models/flightschedule.js` 文件，我们应该看到以下生成的代码：

```js
'use strict';
const {
  Model
} = require('@sequelize/core');
module.exports = (sequelize, DataTypes) => {
  class FlightSchedule extends Model {
    /**
     * Helper method for defining associations.
     * This method is not a part of Sequelize lifecycle.
     * The `models/index` file will call this method
       automatically.
     */
    static associate(models) {
      // define association here
    }
  };
  FlightSchedule.init({
    originAirport: DataTypes.STRING,
    destinationAirport: DataTypes.STRING,
    departureTime: DataTypes.DATE
  }, {
    sequelize,
    modelName: 'FlightSchedule',
  });
  return FlightSchedule;
};
```

代码片段中的 `'use strict';` 行将告诉我们的 Node.js 运行时使用一组规则来执行 JavaScript 文件（`models/flightschedule.js`），以帮助减轻**宽松模式**。**严格模式**将禁止开发者向未声明的变量赋值，使用由 **ECMAScript 2015**（**ES6**）定义的保留关键字等。这种模式对于本书的内容完全是可选的；然而，如果您想了解更多，Mozilla 提供了一份关于严格模式和宽松模式之间差异的有用指南：[`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode/Transitioning_to_strict_mode`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode/Transitioning_to_strict_mode)。

下一条指令是从 Sequelize 导入 `Model` 类，我们将在接下来的几个命令中使用它来初始化模型。随后的 `module.exports` 行是一个模式，Sequelize 模型加载器（`models/index.js` 文件）可以解释并调用该文件。第一个参数为我们提供了一个 Sequelize 实例，其中包含我们从 `models/index.js` 文件中设置的参数和配置设置。第二个参数 `DataTypes` 提供了一种更方便的方式来声明我们列的各种数据类型（例如，我们不需要输入 `sequelize.DataTypes.STRING`，我们可以省略 `sequelize.` 实例前缀，直接使用 `DataTypes.STRING`）。

接下来，我们定义 `FlightSchedule` 类并扩展 Sequelize 的 `Model` 类。在这里，我们可以定义我们的关联、钩子/事件、表信息等。CLI 将为我们生成一个名为 **associations** 的静态函数。目前，我们可以保持这个函数不变；我们将在本书的后续部分对其进行修改。

文件中的最后一条指令是调用 `FlightSchedule` 类的 `init()` 函数，这将设置内部规则和指令以帮助 Sequelize 导航模型定义。这也是 Sequelize 学习如何与数据库同步的地方，如下面的截图所示。如果你将 Sequelize 的选项设置为 `sync: true`，则会在每次运行时添加额外的 `DROP TABLE IF EXISTS` 指令，以便我们在每次运行时都能在一个干净的状态下测试我们的应用程序。将同步选项设置为 `true` 对于单元测试和早期原型开发很有用。**不推荐**在生产环境中将此选项设置为 `true`：

![图 2.1 – Sequelize 的自动同步](img/Figure_2.1_B17841.jpg)

图 2.1 – Sequelize 的自动同步

注意

运行我们的 Node.js 应用程序并让 Sequelize 同步数据库对于初始实现阶段来说是可行的，但我们将介绍使用 Sequelize 的 CLI 和迁移来执行必要的 SQL 命令以同步数据库的方法。迁移提供了增量更改/更新，而 Sequelize 同步选项则更像是通用的解决方案。

`init()` 函数的第一个参数是我们定义模型属性（或列）的地方。这种模式的典型形式是一个对象，其键为列名，每个键的值可以是数据类型、一个字面字符串值，或者包含每个列高级选项的对象。从这个例子中，我们可以看到三个列（`originAirport`、`destinationAirport` 和 `departureTime`），它们分别对应 `string`、`string` 和 `date` 数据类型。

第二个参数允许我们明确地定义模型的实例类型设置。在这里，我们可以定义不同的表名，选择是否使我们的表名复数化，禁用元列（如 `createdAt` 和 `updatedAt`），等等。我们将在本章后面详细介绍这些选项。

如果你更喜欢不在你的项目中使用类，还有一种定义我们模型的方法。以下代码片段提供了一个使用 Sequelize 的 `define()` 函数的示例：

```js
module.exports = (sequelize, DataTypes) => {
    return sequelize.define('FlightSchedule', {
        originAirport: DataTypes.STRING,
        destinationAirport: DataTypes.STRING,
        departureTime: DataTypes.DATE
    }, {
        sequelize,
    });
};
```

参数与 `init()` 几乎相同，只是第一个参数现在是模型名称。两种方式都是可接受的，并且从 Sequelize 的角度来看，一种方式并不比另一种方式有优势。本书将在其代码库（`Model` 类）中使用前者示例，但对于高级配置设置和添加关联，本书将展示两种风格，因为从人体工程学的角度来看，有一些基本差异。对于更喜欢使用 TypeScript 而不是 JavaScript 的程序员，`Model` 类方法可能为你提供更原生体验。

现在我们已经了解了如何在 Sequelize 中定义模型，我们可以回顾 Sequelize 提供的内置属性数据类型，以及简要说明以帮助指导你未来的模型设计。

# 探索各种 Sequelize 数据类型及其使用时机

如前所述，Sequelize 提供了各种数据类型，以帮助将模型的属性映射到相应的 **数据库管理系统**（**DBMS**）列类型。以下是 Sequelize 提供的内容列表，以及简要说明。

## STRING

`STRING` 数据类型指的是通常包含元信息的 `VARCHAR` 字段，以帮助优化数据库管理系统的查询计划。如果字符串的大小超过 255 字节，MySQL 会明确地在列的前缀标题中添加另一个字节。查询计划器可以使用这些信息来帮助减轻内存压力，或者对于具有固定分页长度的 `VARCHAR`，你将定义列为 `DataTypes.STRING(100)` 而不是 `DataTypes.STRING`。

在 `VARCHAR` 列类型中，数据库管理系统不会以固定长度存储值（不需要填充）。如果你需要以与存储时完全相同的方式检索数据，可以使用 `VARCHAR BINARY` 列类型。这可以通过将列的数据类型声明为 `DataTypes.STRING.BINARY` 来实现。

尽管数据类型名称中包含“二进制”一词，但在存储电影、图片等时，通常建议使用 `BLOB` 类型而不是 `VARCHAR BINARY`。`VARCHAR BINARY` 的二进制部分是在该列的二进制表示与 **字符集**（**charset**）之间进行比较。

例如，假设我们在数据库中有以下行：`A`、`a`、`B` 和 `b`。`VARCHAR` 列类型将有一个内部映射来告诉数据库，`A`和`a`将排在`B`和`b`之前。在 `VARCHAR BINARY` 列中，`A`、`a`、`B` 和 `b` 的二进制表示之和将是 `0`、`2`、`1`、`3`，这将排序为以下顺序：`A`、`B`、`a`、`b`。`VARCHAR BINARY` 列没有内部映射/字符集，因此数据库无法知道`a`和`A`实际上是同一个字母。

对于大多数情况，我们可以在 MySQL 版本 5.0.2 以上将 `VARCHAR BINARY` 和 `BLOB` 互换使用。这里有一些细微的差别，如下所示：

+   对于 `BLOB` 索引必须指定索引前缀长度

+   `BLOB` 列类型不能有默认值

## CHAR

`CHAR` 数据类型与 `STRING` 数据类型相似，不同之处在于它引用的是 `CHAR` 列类型。传统上，数据库管理系统会将 `CHAR` 列的长度限制为 255 个字符。`VARCHAR` 类型允许您超过指定的分页大小而不会出现错误或异常。`CHAR` 列可以用作最后的手段来验证您的数据，并确保它不超过指定的长度（例如，`CHAR(20)` 将数据限制为表中排序规则定义的 20 个字符）。`CHAR` 列类型会被填充到其固定长度，这有助于优化数据库管理系统——甚至您的应用程序——前提是预定的长度适合该场景的分页大小。

## TEXT/TINYTEXT/MEDIUMTEXT/LONGTEXT

数据库设计者知道，有时我们的文本数据需要相当大的空间，或者需要与大于 65,535 字节（MySQL `VARCHAR` 限制）的行相关联。在这种情况下，我们会使用 `TEXT` 列类型。每个数据库管理系统都有其细微差别和限制；由于本书使用 MySQL，我们将简要介绍 MySQL 的 `TEXT` 限制，如下所示：

+   `TINYTEXT`: 255 字节

+   `TEXT`: 64 **千字节** (**KB**)

+   `MEDIUMTEXT`: 16 **兆字节** (**MB**)

+   `LONGTEXT`: 4 **千兆字节** (**GB**)

`DataTypes.TEXT` 将默认为 `TEXT` 列类型，如果您想将列类型声明为 `TINYTEXT`、`MEDIUMTEXT` 或 `LONGTEXT`，则分别使用 `DataTypes.TEXT('TINY')`、`DataTypes.TEXT('MEDIUM')` 或 `DataTypes.TEXT('LONG')`。与 `VARCHAR` 类型不同，`TEXT` 列类型没有 `BINARY` 选项。对于存储序列化的二进制类型，您将使用 `VARCHAR BINARY` 或 `BLOB`。

## CITEXT

**CITEXT** 代表 **不区分大小写的文本**，这是一个在比较操作之外保留数据大小写的列。此选项仅适用于 Postgres 和 SQLite 数据库。

## NUMBER

不要与 Postgres 的`NUMERIC`类型混淆，`NUMBER`数据类型是根据其配置设置的一个抽象类型，而不仅仅是显式类型。除非您正在扩展/添加自己的数值数据类型，否则不应直接使用此抽象数据类型。如果您的数据库中使用了相同的精度和比例值，或者您的在线商店以不同的货币销售产品，则此抽象数据类型可以帮助组织您的代码。

以下代码片段提供了一个如何扩展您自己的数值数据类型的示例：

```js
// INT(4)
var unsignedInteger = DataTypes.NUMBER({
    length: 4,
    zerofill: false,
    unsigned: true,
});
// FLOAT(5,4)
var specificFloat = DataTypes.NUMBER({
    length: 5,
    zerofill: false,
    unsigned: false,
    decimals: 4
});
// DECIMAL(6,4)
var specificPrecision = DataTypes.NUMBER({
    zerofill: false,
    unsigned: false,
    precision: 6,
    scale: 4
});
```

## INTEGER/TINYINT/SMALLINT/MEDIUMINT/BIGINT

使用`DataTypes.INTEGER`、`DataTypes.SMALLINT`等，我们可以将我们的属性与相应的列类型关联起来。您可以在 MySQL 中找到每个整数类型的最大和最小值参考：[`dev.mysql.com/doc/refman/5.7/en/integer-types.xhtml`](https://dev.mysql.com/doc/refman/5.7/en/integer-types.xhtml)。要声明您的模型属性为无符号值，我们可以在数据类型上附加`UNSIGNED`选项，如下所示：

```js
DataTypes.INTEGER(21).UNSIGNED
```

如果我们想要我们的属性无符号且零填充，我们可以链式调用数据类型选项，如下所示：

```js
DataTypes.INTEGER(21).UNSIGNED.ZEROFILL
```

注意

根据您使用的数据库管理系统（DBMS），`ZEROFILL`选项可能不可用。如果您使用的是 Postgres 数据库，那么分配这些属性的顺序很重要（`UNSIGNED`必须在`ZEROFILL`之前声明）。在 MySQL 中，`ZEROFILL`选项也会自动隐含`UNSIGNED`。`ZEROFILL`属性将只从美学角度（当您选择数据时）影响数据，而不会修改您的存储数据。

## FLOAT/REAL

传统上，数据库管理系统（DBMS）会根据位精度来区分`FLOAT`和`REAL`列类型。`FLOAT`列通常以 32 位精度存储，而`REAL`列类型则以 64 位精度存储。`REAL`列类型是 64 位的，而`FLOAT`列是 32 位的。更令人困惑的是，MySQL 会将`REAL`视为与`DOUBLE`（也称为`DOUBLE PRECISION`和`DECIMAL`）列相同。

在内部，Sequelize 以相同的方式处理`FLOAT`、`REAL`和`DOUBLE`。对于`FLOAT`类型，会显式执行一个小浮点验证，但除此之外，Sequelize 将直接将列类型转换为数据库管理系统。就像整数数据类型一样，`UNSIGNED`和`ZEROFILL`也可以定义在这些属性上，如下所示：

```js
DataTypes.FLOAT.UNSIGNED.ZEROFILL
```

## DECIMAL/DOUBLE

`DECIMAL`或`DOUBLE`数据类型允许我们使用传统的`DECIMAL(P, S)`格式定义列的精确长度和比例，其中`P` > `S`。`P`变量是数字的精度，而`S`变量是数字的比例。精度决定了整个数字部分的长度，而比例定义了小数部分的长度。例如，`DataTypes.DECIMAL(6, 4)`将给我们一个精度为 6 和比例为 4 的十进制列。此列的一个示例值可以是`38.3411`。

注意

你可以将 `DataTypes.NUMERIC` 作为 `DataTypes.DECIMAL` 的别名使用。

## BOOLEAN

表达 `false` 和 `true` 的 1 有许多方式。有时，布尔值可能以字符串形式存储，例如 `true`、`false`、`t` 或 `f`。Sequelize 会自动处理数值或位值，以及“`true`”或“`false`”字符串表达式作为 Node.js 的适当布尔值。如果值标记为“`t`”或“`f`”，则 Sequelize 将将原始值传递给程序员处理（作为一种避免过于自信的方式——此行为可能在将来发生变化）。布尔列可以用 `DataTypes.BOOLEAN` 定义。此数据类型没有参数或输入要处理。

## DATE/DATEONLY/TIME

`DATE` 数据类型引用 MySQL、MariaDB 和 SQLite 的 `DATETIME` 列类型。对于 Postgres，`DATE` 数据类型将被转换为 `TIMESTAMP WITH TIME ZONE`。

在 MySQL 中，你可以为 `DATETIME` 列定义最多六位小数的分数秒，如下所示：

```js
DataTypes.DATE(6)
```

如果你只想保留日期或时间，你可以分别使用 `DataTypes.DATEONLY` 或 `DataTypes.TIME`。

关于不带时区的 Postgres 的快速说明

如果你使用的是带有 `TIMESTAMP WITHOUT TIME ZONE` 列类型的 Postgres，并且你知道数据的时间与运行应用程序的服务器不同，建议设置时区偏移量。这可以通过 `pg` Node.js 库实现，如下所示：

`var types = require('pg').types`

`function setTimestampWithoutTimezoneOffset(val) {`

`    // '+0000' 是 UTC 偏移量，将其更改为所需时区`

`    return val === null ? null : new Date(stringValue + '+0000');`

`}`

`types.setTypeParser(types.builtins.TIMESTAMP, setTimestampWithoutTimezoneOffset);`

有关在 Node.js 中为 Postgres 设置类型的更多信息，请参阅以下链接：[`github.com/brianc/node-pg-types`](https://github.com/brianc/node-pg-types)

## NOW

`DataTypes.NOW` 是 Sequelize 中的一个特殊类型。它不应用作列的类型，而应作为属性值，并且传统上设置为属性的 `defaultValue` 选项。如果我们想要一个 `Receipt` 模型来跟踪交易时间，它看起来会像这样：

```js
Receipt.init({
    total: DataTypes.DECIMAL(10,2),
    tax: DataTypes.DECIMAL(10,2),
    dateOfPurchase: {
        type: DataTypes.DATE,
        defaultValue: DataTypes.NOW
    }
}, {
    sequelize,
    modelName: 'Receipt'
});
```

每当我们插入一个 `Receipt` 记录时，Sequelize 会自动将 `dateOfPurchase` 属性的值转换为 DBMS 的 `NOW()` 函数，使用 Sequelize 的 `DataTypes.NOW` 数据类型，并从属性的 `defaultValue` 选项中获取。如果我们最初为属性定义了一个值，那么 Sequelize 将使用该值。

## HSTORE

`HSTORE` 仅适用于 Postgres。此数据类型用于映射键值类型，但通常被 `JSON` 或 `HSTORE` 替换，但需要注意一个注意事项，即需要安装 `pg-hstore` Node.js 库。完整的安装命令如下所示：

```js
npm install --save sequelize pg pg-hstore
```

在 Sequelize 中选择数据时，您的 `where` 子句将是一个对象，而不是整数、字符串等。一个例子如下所示：

```js
MyModel.find({
  where: {
    myHstoreColumn: {
      someFieldKey: 'value',
    }
  }
});
```

## JSON

`JSON` 数据类型适用于 SQLite、MariaDB、MySQL 和 Postgres。当使用 `JSON` 类型定义属性时，您可以查询类似于 `HSTORE` 类型查询的信息，除了您无法深度嵌套您的搜索子句。假设我们有一个以下 `JSON` 数据类型存储在列中：

```js
{
    "someKey": {
        "deeply": {
            "nested": true
        }
    }
}
```

我们将按如下方式搜索嵌套值：

```js
MyModel.find({
    where: {
        myJsonColumn: {
            someKey: { deeply: { nested: true } }
        }
    }
});
```

请注意，MySQL 和 MariaDB 引入了对 `DataTypes.JSON` 属性类型支持，将不会与您的数据库兼容。要解决这个问题，您可以定义具有获取器/设置器的模型，这些获取器/设置器将存储和检索 JSON 文档，如下所示：

```js
sequelize.define('MyModel', {
    myJsonColumn: {
        type: DataTypes.TEXT,
        get: function () {
            return JSON.parse(this.getDataValue('value'));
        },
        set: function (val) {
            this.setDataValue('value',JSON.stringify(val));
        }
    }
});
```

注意

对于使用 MSSQL 2016 及以上版本的用户，请参阅 [`sequelize.org/master/manual/other-data-types.xhtml#mssql`](https://sequelize.org/master/manual/other-data-types.xhtml#mssql) 作为处理此 DBMS 中 JSON 列类型的解决方案。

## JSONB

`JSONB` 数据类型仅适用于 Postgres。如果您使用 JSON 列进行存储，建议使用 `JSON` 列类型；如果您在该列上使用比较运算符，建议使用 `JSONB` 列类型。

除了之前提到的查询 JSON 数据的方法之外，您还可以使用以下格式查询 JSONB 数据类型：

```js
// String matching
MyModel.find({
  where: {
    "someKey.deeply.nested": {
      [Op.eq]: true
    }
  }
});
// Using the Op.contains operator
MyModel.find({
  where: {
    someKey: {
      [Op.contains]: {
        deeply: {
          nested: true
        }
      }
    }
  }
});
```

## BLOB

包括 MySQL 在内的几个数据库提供了一系列 `BLOB` 属性类型，Postgres 总是将它们转换为 `bytea`（字节数组）列类型。这种数据类型适用于存储任何与二进制相关的数据，例如图像、文档或序列化数据。您可以在以下示例中看到它的使用：

```js
DataTypes.BLOB // BLOB
DataTypes.BLOB('tiny') // TINYBLOB
DataTypes.BLOB('medium') // MEDIUMBLOB
DataTypes.BLOB('long') // LONGBLOB
```

这里是一个不同 `BLOB` 类型及其字节前缀长度和最大存储长度的表格，适用于 MySQL：

| BLOB 类型 | 字节前缀长度 | 最大存储（以字节为单位） |
| --- | --- | --- |
| TINYBLOB | 1 字节 | 2⁸ - 1 |
| BLOB | 2 字节 | 2¹⁶ - 1 |
| MEDIUMBLOB | 3 字节 | 2²⁴-1 |
| LONGBLOB | 4 字节 | 2³² - 1 |

RANGE

`RANGE` 数据类型仅适用于 Postgres。支持的范围类型是 `INTEGER`、`BIGINT`、`DATE`、`DATEONLY` 和 `DECIMAL`。您可以定义具有范围类型的属性，如下所示：

```js
var MyModel = sequelize.define('MyModel', {
    myRangeColumn: DataTypes.RANGE(DataTypes.INTEGER)
});
```

如此所示，我们可以为我们的模型创建范围，有几种方法可以做到这一点：

```js
// inclusive boundaries are the default for Sequelize
   var inclusiveRange = [10, 20];
MyModel.create({ myRangeColumn: inclusiveRange });
// inclusion may be toggled with a parameter
   var range = [
    { value: 10, inclusive: false },
    { value: 20, inclusive: true }
];
MyModel.create({ myRangeColumn: range });
```

当查询范围列时，该属性的值将始终以对象表示法返回，带有 `value` 和 `inclusive` 键。

## UUID/UUIDV1/UUIDV4

`UUIDV1`/`UUIDV4` 数据类型与 `UUID` 属性类型协同工作。我们可以声明一个具有默认 `UUIDV4` 值作为其 **主键**（**PK**）的模型，如下所示：

```js
sequelize.define('MyModel', {
    id: {
        type: DataTypes.UUID,
        defaultValue: DataTypes.UUIDV4,
        allowNull: false,
        primaryKey: true
    }
});
```

## 虚拟

`VIRTUAL`属性类型是一种特殊类型，它将在 Sequelize 中填充数据，但不会将数据填充到数据库中。`VIRTUAL`字段可用于组织代码、验证以及扩展 Sequelize 到任何需要嵌套类型（例如，GraphQL、**Protocol Buffers**（**Protobuf**）等）的协议或框架，这将在*第九章*，*使用和创建适配器*中介绍。

我们可以这样定义一个`VIRTUAL`属性：

```js
sequelize.define('MyModel', {
    envelope: DataTypes.STRING,
    message: {
        type: DataTypes.VIRTUAL,
        set: function(val) {
            // the following line is optional
            // but required if you wish to use the
               validation associated with the attribute
            this.setDataValue('message', val);
            this.setDataValue('envelope', 
                               encryptTheMessage(val));
        },
        validate: {
            noDadJokes: function(val) {
                if (val === "knock knock") {
                    throw new Error("Who is there? Not this 
                                     message")
                }
            }
        }
    }
});
```

对于检索`VIRTUAL`属性，我们需要为`DataTypes.VIRTUAL`调用定义一个数据类型作为参数。如果我们想在我们的`VIRTUAL`属性中传递其他属性，我们将定义一个列表作为第二个参数。以下是一个示例：

```js
sequelize.define('MyModel', {
    envelope: DataTypes.STRING,
    message: {
        type: DataTypes.VIRTUAL(DataTypes.STRING, ['en
        velope']),
        get: function() {
            return decryptTheMessage(this.get('envelope'));
        },
        set: function(val) {
            this.setDataValue('envelope', 
                               encryptTheMessage(val));
        }
    }
});
```

## ENUM

Sequelize 有一个`DataTypes.ENUM`属性类型用于枚举列。目前，只有 Postgres 启用了此功能。对于其他数据库引擎的解决方案是定义一个自定义验证规则，该规则执行某种包含操作符。下一章将讨论我们模型的自定义验证。定义枚举属性有三种不同的方式，如下所示：

```js
// Defining enums with function arguments
DataTypes.ENUM('a', 'b')
// Defining enums with an array argument
DataTypes.ENUM(['a', 'b'])
// Defining enums with an object argument
DataTypes.ENUM({
    values: ['a', 'b']
})
```

## ARRAY

目前，只有 Postgres 支持`ARRAY`属性类型。此类型需要一个适用数据类型的参数。您可以在以下示例中查看：

```js
DataTypes.ARRAY(DataTypes.STRING) // text[]
DataTypes.ARRAY(DataTypes.DECIMAL) // double precision[]
```

## GEOMETRY

Sequelize 可以处理 MariaDB、MySQL 和 Postgres 的几何数据（只要启用了 PostGIS 扩展）。**GeoJSON**规范([`tools.ietf.org/html/rfc7946`](https://tools.ietf.org/html/rfc7946))对于查询航空业务中的几何数据可能很有用。例如，我们可以标记机场的坐标和飞机的当前位置，以确定预计到达时间，而无需手动记住 Haversine 算法（一个确定球面上两点之间距离的公式）。以下代码片段中可以找到参考示例：

```js
var MyModel = sequelize.define('MyModel', {
    point: DataTypes.GEOMETRY('POINT'),
    polygon: DataTypes.GEOMETRY('POLYGON')
});
var point = {
    type: 'Point',
    coordinates: [44.386815, -82.755759]
}
var polygon = { type: 'Polygon', coordinates: [
    [
        [100.0, 0.0], [101.0, 0.0], [101.0, 1.0],
        [100.0, 1.0], [100.0, 0.0]
    ]
]};
await MyModel.create({ point, polygon });
```

在前面的代码片段中，我们首先使用两个属性（`point`和`polygon`）及其相应的几何数据类型（对于完整列表，您可以参考之前提到的**请求评论**（**RFC**）手册）定义我们的模型。然后，我们使用一组定义的值创建我们的几何对象（一个点将接受两个坐标，一个多边形可以接受 N 个坐标）。最后一行将为相应的属性创建一个具有定义值的条目。

注意

GeoJSON 的处理方式取决于我们是否使用 Postgres 或 MariaDB/MySQL 方言。Postgres 方言将调用`ST_GeomFromGeoJSON`函数来解释 GeoJSON，而 MariaDB/MySQL 将使用`GeomFromText`函数。以下参考详细介绍了 MySQL 中的空间列：[`dev.mysql.com/doc/refman/5.7/en/spatial-types.xhtml`](https://dev.mysql.com/doc/refman/5.7/en/spatial-types.xhtml)。

## GEOGRAPHY

对于 MariaDB/MySQL，`GEOGRAPHY`属性类型将像`GEOMETRY`类型一样工作，但对于 Postgres，Sequelize 将使用 PostGIS 的地形数据类型。`GEOGRAPHY`属性类型遵循与`GEOMETRY`类型相同的 GeoJSON 语法。

注意

如果你正在寻找一组完整的实用函数，并查询多个坐标之间的复杂关系，那么建议使用`GEOMETRY`类型而不是`GEOGRAPHY`类型。如果你需要使用大地测量而不是笛卡尔测量，或者如果你在大面积上具有更简单的关系，那么`GEOGRAPHY`类型将更适合你。

## CIDR/INET/MACADDR

这三种属性类型仅适用于 Postgres。这些类型各自执行一些内部验证。这些类型没有输入参数。以下是对这些数据类型的简要说明，并附有参考：

+   `CIDR`—这代表**无类别域间路由**，用于分配**互联网协议**（**IP**）地址和路由表 ([`datatracker.ietf.org/doc/html/rfc4632`](https://datatracker.ietf.org/doc/html/rfc4632))

+   `INET`—在网络上识别的常见方式 ([`datatracker.ietf.org/doc/html/rfc6991`](https://datatracker.ietf.org/doc/html/rfc6991))

+   `MACADDR`—网络接口的**唯一标识符**（**UIDs**）([`www.rfc-editor.org/rfc/rfc7042.xhtml`](https://www.rfc-editor.org/rfc/rfc7042.xhtml))

## TSVECTOR

`TSVECTOR`数据类型用于通过 Postgres 的`to_tsquery()`函数中可用的高级运算符搜索文本列。这些运算符包括通配符匹配、否定匹配和布尔搜索。此属性类型仅适用于 Postgres，并且仅接受字符串变量作为值。在查询`TSVECTOR`属性时，Sequelize 不会隐式解释与关联函数（例如，`to_tsvector`）相关的属性类型。假设我们有以下模型定义：

```js
var MyModel = sequelize.define('MyModel', {
    col: DataTypes.TSVECTOR
});
MyModel.create({
    col: 'The quick brown fox jumps over the lazy dog'
});
```

然后，我们想要查询`col`字段上的值，如下所示：

```js
MyModel.find({
    where: { col: 'fox' }
});
```

生成的 SQL 将类似于以下内容：

```js
SELECT * FROM MyModel WHERE col = 'fox';
```

Sequelize 将使用等于运算符解释此查询的`where`子句。为了利用`TSVECTOR`列类型，我们必须明确我们的意图，如下所示：

```js
MyModel.find({
    where: {
        col: {
            [Op.match]: sequelize.fn('to_tsvector', 'fox')
        }
    }
});
```

这将把`where`子句的运算符从等于转换为匹配（`@@`）。`sequelize.fn`方法允许你显式调用你的 DBMS 中的函数。此过程生成的查询将如下所示：

```js
SELECT * FROM MyModel WHERE col @@ to_tsvector('fox');
```

在学习如何定义我们的模型以及 Sequelize 中可用的数据类型之后，我们现在可以将我们的定义迁移到实际的数据库中。Sequelize 在其命令行工具中提供了一个迁移子命令，以便我们更容易地完成此操作。

# 将 Sequelize 中的图示更改和数据迁移到数据库

我们已经使用命令行工具生成的文件定义了我们的数据库模式，我们现在准备将这些定义迁移到我们的 DBMS。使用 Sequelize 的迁移可以帮助开发团队在多台机器上维护相同的模式结构。迁移可以提供历史参考，说明数据库是如何随时间变化的，这也可以帮助我们撤销某些更改，并将数据库的模式回滚到特定的时间。

## 迁移电路图更改

Sequelize CLI 提供了一种方便的方式来将更新传播到数据库。我们所有的电路图更改都将位于`migrations`目录中，我们所有的数据种子都将位于`seeders`目录中。本章将仅涵盖数据库结构的初始化。在随后的章节中，将有使用迁移工具添加和删除列（或索引）的示例。

在*定义数据库模型*部分，我们使用了 Sequelize CLI 来生成我们的模型，这应该在`migrations`目录中创建了几个文件。每个文件都以时间戳、一个`create`和模型的名称为前缀。其中一个文件（例如`20210914153156-create-airplane.js`）的示例如下：

```js
'use strict';
module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.createTable('Airplanes', {
      id: {
        allowNull: false,
        autoIncrement: true,
        primaryKey: true,
        type: Sequelize.INTEGER
      },
      planeModel: {
        type: Sequelize.STRING
      },
      totalSeats: {
        type: Sequelize.INTEGER
      },
      createdAt: {
        allowNull: false,
        type: Sequelize.DATE
      },
      updatedAt: {
        allowNull: false,
        type: Sequelize.DATE
      }
    });
  },
  down: async (queryInterface, Sequelize) => {
    await queryInterface.dropTable('Airplanes');
  }
};
```

当我们调用`migrations`子命令时，Sequelize 将使用`up(…)`方法的范围。`down(…)`方法保留用于当我们决定撤销或回滚迁移时。**查询接口**是一个数据库无关的适配器，它执行所有支持数据库引擎都可用的一般 SQL 命令。我们将在后面的章节中详细介绍查询接口。

你可能已经注意到 Sequelize 已经在我们的模型定义中添加了几个列。使用默认设置，Sequelize 将生成三个额外的列，如下所示：

+   `id`—一个设置为`autoIncrement`为`true`的整数 PK。`autoIncrement`标志将创建一个序列值（例如，MySQL 等数据库将序列称为*auto-increment*列）。

+   `createdAt`—此字段将在行创建时生成一个时间戳。由于这是一个 Sequelize 识别的列，因此此列的默认值不需要我们明确声明`DataTypes.NOW`或任何等效值。当使用`create()`等适用方法时，Sequelize 将自动填充行的值。

+   `updatedAt`—与`createdAt`字段类似，但此值将自动从 Sequelize 更新，每次行更新时都会更新。

注意

我们可以通过配置设置来防止 Sequelize 自动创建这些属性。这些设置将在本章后面详细解释。

在我们项目的根目录中，运行以下命令以初始化迁移：

```js
sequelize db:migrate
```

此命令将执行比遍历 `migrations` 目录更多的指令。Sequelize 首先会寻找一个名为 `SequelizeMeta` 的表，该表包含通过 `migrations` 子命令已处理的文件的相关元信息。找到或创建该表后，Sequelize 将按文件名的顺序遍历 `migrations` 表（时间戳是保持这种顺序的便捷方式），并跳过 `SequelizeMeta` 表中找到的任何文件。

注意

`sequelize-cli` 的 `db:migrate` 和 `db:seed` 命令将使用 `NODE_ENV` 环境变量来确定迁移/初始化数据的位置。作为替代，你可以使用 `--url` 选项指定要连接到的数据库，如下所示：`sequelize db:migrate --url 'mysql://user:password@host.com/database'`。

如果我们在模型定义上犯了错误，迁移后，我们始终可以选择撤销更改，如下所示：

```js
sequelize db:migrate:undo
```

这将撤销 Sequelize 执行的最后一个迁移。如果我们想撤销所有更改，还有一个子命令，如下所示：

```js
sequelize db:migrate:undo:all
```

如果我们想撤销所有迁移直到某个点（这就是为什么在文件名前加时间戳很重要），我们可以运行以下命令：

```js
sequelize db:migrate:undo:all --to XXXXXXXXXXXXXX-airlines.js
```

迁移完成后，我们应该运行以下命令：

```js
$ mysql -uroot airline
mysql> show tables;
```

下面的屏幕截图应显示以下表：

![图 2.2 – 显示项目表](img/Figure_2.2_B17841.jpg)

图 2.2 – 显示项目表

## 初始化种子数据

现在我们已经建立了模式，我们可以通过在 `seeders` 目录内生成种子文件来开始用实际数据填充我们的数据库。种子数据传统上用于初始配置数据、静态信息等。好事成双——我们的商业伙伴刚刚通知我们，他们购买了五架飞机以帮助我们开始。我们可以为这些飞机创建种子数据，如下所示：

```js
sequelize seed:generate --name initial-airplanes
```

这将在我们的项目 `seeders` 目录中生成一个文件，该文件包含将种子数据迁移到数据库所需的最基本信息。类似于我们的迁移文件，CLI 只公开了两种方法：`up(…)` 和 `down(…)`。

我们将用以下代码替换文件内容：

```js
'use strict';
module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.bulkInsert('Airplanes', [{
      planeModel: 'Airbus A220-100',
      totalSeats: 110,
      createdAt: new Date(),
      updatedAt: new Date()
    }, {
      planeModel: 'Airbus A220-300',
      totalSeats: 110,
      createdAt: new Date(),
      updatedAt: new Date()
    }, {
      planeModel: 'Airbus A 318',
      totalSeats: 115,
      createdAt: new Date(),
      updatedAt: new Date()
    }, {
      planeModel: 'Boeing 707-100',
      totalSeats: 100,
      createdAt: new Date(),
      updatedAt: new Date()
    }, {
      planeModel: 'Boeing 737-100',
      totalSeats: 85,
      createdAt: new Date(),
      updatedAt: new Date()
    }], {});
  },
  down: async (queryInterface, Sequelize) => {
    await queryInterface.bulkDelete('Airplanes', null, {});
  }
};
```

注意

与 Sequelize 的 `create()` 函数不同，查询接口的 `bulkInsert()` 方法不会自动填充 `createdAt` 或 `updatedAt` 列。如果你从种子文件中省略这些列，数据库将返回错误，因为这些列没有默认值。

现在，我们可以通过以下命令处理我们的种子数据：

```js
sequelize db:seed:all
```

我们可以通过在数据库中输入以下 SQL 命令来确认更改：

```js
SELECT * FROM airplanes;
```

我们随后得到以下结果：

![图 2.3 – 查询飞机列表](img/Figure_2.3_B17841.jpg)

图 2.3 – 查询飞机列表

回滚种子数据与 `migrations` 子命令类似，如下所示：

```js
sequelize db:seed:undo
sequelize db:seed:undo:all
sequelize db:seed:undo --seed <the name of your seed file>
```

提示

Sequelize 在内部使用另一个名为 Umzug 的项目进行迁移。完整的参考和更多关于如何调整迁移周期的示例可以在项目的 GitHub 仓库中找到：[`github.com/sequelize/umzug`](https://github.com/sequelize/umzug)。

在将种子数据插入数据库后，我们现在可以使用 Sequelize 查询或操作这些数据。以下部分将提供一个关于如何将 Sequelize 集成到 Express 应用程序的简要介绍，并遵循 Sequelize 的参考风格。这将帮助您了解我们将在后续章节中如何应用 Sequelize 到我们的航空公司项目中，并为您提供足够的知识，以便您能够舒适地调整自己的设置。

# 使用 Sequelize 操作和查询数据

在初始化我们的数据库结构和数据后，我们应该能够从我们的仪表板中查看、修改和删除飞机。目前，我们将为我们的管理任务创建非常简单和基础的实现，但由于我们是 *Avalon Airlines* 唯一的技术员工，这不会成为问题。随着我们继续创建项目，我们将修改我们的应用程序，使其更加健壮，并考虑安全措施。

## 读取数据

将 `app.get('/', …)` 块替换为以下代码（在 `index.js` 中）：

```js
app.get('/', async function (req, res) {
    const airplanes = await models.Airplane.findAll();
    res.send("<pre>" + JSON.stringify(airplanes, undefined, 
              4) + "</pre>");
});
```

然后，保存文件，并使用以下命令运行我们的应用程序：

```js
npm run start
```

现在，我们可以访问我们的网站 `http://localhost:3000`，应该会看到与这里显示的类似的结果：

![图 2.4 – 列出我们的飞机](img/Figure_2.4_B17841.jpg)

图 2.4 – 列出我们的飞机

现在，我们将创建另一个路由，该路由将返回特定飞机的结果。如果找不到飞机，则应发送 *未找到* `404`）。在根 `app.get('/', …)` 块下方添加以下路由）：

```js
app.get('/airplanes/:id', async function (req, res) {
    var airplane = await models.Airplane.findByPk
                   (req.params.id);
    if (!airplane) {
        return res.sendStatus(404);
    }
    res.send("<pre>" + JSON.stringify(airplane, undefined, 
              4) + "</pre>");
});
```

`findByPk` 方法将尝试从模型的 PK 属性（默认情况下，这将是由 Sequelize 生成的 `id` 列）中查找记录。当找到记录时（例如，[`localhost:3000/airplanes/1`](http://localhost:3000/airplanes/1)），应用程序将返回记录给我们，但如果我们将 `id` 参数从 `1` 更改为 `10` (`http://localhost:3000/airplanes/10`)，我们应该收到一个 *未找到* 的响应。

下面是一个包含 Sequelize 数据检索相关函数简要说明的列表：

+   `findAll`—当您想在查询中使用 `where` 子句并检索多行时使用此函数。

+   `findOne`—与 `findAll` 函数类似，但此函数将返回单行记录。

+   `findByPk`—一个返回使用模型定义的 PK 的单行记录的函数。

+   `findOrCreate`—此函数将返回数据库中找到或实例化的单个行实例。Sequelize 将组合 `where` 和 `defaults` 键中定义的属性。

## 复杂查询

有时候，你可能需要比简单的 `where` 子句和 `AND` 运算符更多的东西。Sequelize 内置了几个运算符来帮助编写具有更复杂 `where` 子句的查询。这些运算符的完整列表如下所示：

+   `and`/`or`—逻辑 `AND` 和逻辑 `OR`。这些值包含一个 `where` 子句对象的数组。

+   `eq`/`ne`—等于 (`=`) 或不等于 (`!=`)。

+   `gte`/`gt`—大于等于 (`>=`) 和大于 (`>`)。

+   `lte`/`lt`—小于等于 (`<=`) 和小于 (`<`)。

+   `is`/`not`—`IS NULL` 和 `IS NOT TRUE`，分别。

+   `in`/`notIn`—任何具有值的数组的 `IN` 和 `NOT IN` 运算符。

+   `any`/`all`/`values`—`ANY`（仅适用于 Postgres）、`ALL` 和 `VALUES` 运算符，分别。

+   `col`—将列字符串值转换为数据库/方言指定的 **标识符**（**IDs**）。

+   `placeholder`—Sequelize 使用的内部运算符。

+   `join`—由 Sequelize 内部使用。

+   `match`—用于文本搜索的匹配运算符（仅适用于 Postgres）。

+   `like`/`notLike`—`LIKE` 和 `NOT LIKE`，分别。

+   `iLike`/`notILike`—`LIKE` 和 `NOT LIKE` 的不区分大小写的版本（仅适用于 Postgres）。

+   `startsWith`/`endsWith`—`LIKE '%...'` 和 `LIKE '...%'` 表达式的简写。

+   `substring`—`LIKE '%...%'` 的简写表达式。

+   `regexp`/`notRegexp`—仅适用于 MySQL 和 Postgres 的 `REGEXP` 和 `NOT REGEXP`。

+   `between`/`notBetween`—`BETWEEN x AND y` 和 `NOT BETWEEN x AND y`。

+   `overlap`—仅适用于 Postgres 的范围重叠运算符 (`&&`)。

+   `contains`/`contained`—分别对应 `@>` 和 `<@` 的 Postgres 专用范围运算符。

+   `Adjacent`—仅适用于 Postgres 的相邻查询范围运算符 (`-|-`)。

+   `strictLeft`/`strictRight`—Postgres 范围（`<<` 和 `>>`）的严格运算符。

+   `noExtendRight`/`noExtendLeft`—Postgres 的无扩展范围运算符 (`&<` 和 `&>`)。

查询复杂的 `where` 子句可能看起来像这样：

```js
const { Op } = require("sequelize");
MyModel.findAll({
    where: {
        [Op.or]: [
            { status: 'active' },
            sequelize.where(sequelize.fn('lower', se
            quelize.col('name')), {
                [Op.eq]: 'bob'
            },
            {
                [Op.and]: {
                    age: {
                        [Op.gte]: 40
                    },
                    name: {
                        [Op.like]: 'mary%'
                    }
                }
            }
        }]
    }
});
```

这将产生以下查询：

```js
SELECT
    ...
FROM "MyModel"
WHERE (
    status = 'active'
    OR
    lower(name) = 'bob'
    OR (
        age >= 40
        AND
        name LIKE 'mary%'
    )
)
```

## 删除数据

对于删除一个实例（单个记录），我们可以调用一个 `destroy()` 函数，如下所示：

```js
var record = MyModel.findOne({ where: { /* ... */ } });
await record.destroy();
```

注意

如果在你的模型定义中没有标记为 PK 的属性，那么 Sequelize 可能无法删除正确的记录。实例的 `destroy()` 方法会使用一个尝试匹配实例所有属性的 `where` 子句被调用。这可能导致意外的删除。

要一次性删除多行，执行以下代码：

```js
MyModel.destroy({ where: { name: 'Bob' }});
```

你可以通过传递配置选项到 `destroy()` 方法来删除表中所有数据，如下所示：

```js
await MyModel.destroy({ truncate: true });
// or
await MyModel.truncate();
```

## 更新和保存数据

Sequelize 提供了几种更新属性/数据的方法，具体取决于你从哪里更新。如果你希望更新多行，我们可以使用模型的 `update()` 函数，如下所示：

```js
await MyModel.update({ name: "John" }, {
  where: { name: null }
});
```

此查询将更新所有记录的名称为 `John`，其中当前值为 `NULL`。对于更新特定实例，我们会更改属性值，然后调用 `save()` 函数，如下所示：

```js
var record = MyModel.findOne();
record.name = "John";
await record.save();
```

如果你正在更改记录的属性，并且你的工作流程需要你将记录的数据重置回原始值（而不接触数据库），你可以使用`reload()`方法，如下所示：

```js
var record = MyModel.findOne({ where: { name: 'John' } });
record.name = "Bob";
record.reload();
// the record.name attribute's value is now back to John
```

## 创建数据

要创建单行，Sequelize 的代码将类似于以下内容：

```js
await MyModel.create({ firstName: 'Bob' }, { ... });
```

`create()`的第二个参数接受以下选项：

+   `raw`—如果此布尔值设置为`true`，则 Sequelize 将忽略模型定义中的**虚拟设置器属性**。当你想跳过通过设置器函数转换数据并直接使用查询提供的原始值时，这很有用。

+   `isNewRecord`—一个布尔值，可以启用（如果设置为`true`）Sequelize 应用默认值、更新时间戳列等行为。此方法的默认值为`true`。

+   `include`—包含 Sequelize 的包含选项的数组。本书将在后面的章节中提供示例和更多细节。

+   `fields`—包含将过滤要更新、验证和保存的属性名称的字符串数组。

+   `silent`—如果此值设置为`true`，则 Sequelize 不会更新`updatedAt`时间戳列。

+   `validate`—一个布尔值，用于切换是否执行验证。

+   `hooks`—一个布尔值，用于启用/禁用`create`、`update`和`validate`生命周期事件之前/之后运行。

+   `logging`—一个函数，将传递查询的语句。

+   `benchmark`—记录执行查询时间（以毫秒为单位），并将作为`logging`函数的第二个参数传递。

+   `transaction`—你可以传递一个事务 Sequelize 实例作为此选项。

+   `searchPath`—仅 Postgres 选项，用于定义查询时使用哪个`search_path`。

+   `returning`—仅 Postgres 选项，用于创建新记录时选择返回哪些字段。布尔值`true`将返回所有字段，但字符串数组将过滤要返回的列。

批量插入数据与使用 Sequelize 创建单行非常相似。以下代码片段展示了这一示例：

```js
await MyModel.bulkCreate([
    { firstName: 'Bob' },
    { firstName: 'William' }
], {...});
```

第一个参数是值数组，第二个参数是配置选项。这些选项与`create()`方法相同：`fields`、`validate`、`hooks`、`transaction`、`logging`、`benchmark`、`returning`和`searchPath`。此外，`bulkCreate()`方法还提供了以下选项：

+   `individualHooks`—在为每个记录执行创建生命周期事件之前/之后运行。这不会影响批量之前/之后生命周期事件。

+   `ignoreDuplicates`—通过在表上定义的任何约束键忽略重复的行。此功能不支持 MSSQL 或低于 9.5 版本的 Postgres。

+   `k`—如果存在重复键条目，则更新字段的数组（仅适用于 MySQL/MariaDB、SQLite 3.24.0+和 Postgres 9.5+）。

## 排序和分组

当筛选你的数据时，你可以按如下方式对列进行排序（或分组）：

```js
MyModel.findAll({
    where: { name: 'Bob' },
    order: [
        ['name', 'DESC']
    ]
});
```

对于分组，根据你使用的数据库，你可能与其他数据库引擎（例如，要求你只选择聚合函数和分组列）得到不同的结果。请咨询你数据库的文档，以了解分组所需的特定细微差别和规则。以下是一个简单的 `GROUP BY` 语句示例：

```js
MyModel.findAll({ group: 'name' });
```

警告

Sequelize 将将组的输入视为字面值。如果你是按用户生成的内容分组，强烈建议你转义你的值以避免 SQL 注入（[`en.wikipedia.org/wiki/SQL_injection`](https://en.wikipedia.org/wiki/SQL_injection)）。你可以使用 `sequelize.escape('...');` 方法来转义值。

## 限制和分页

我们可以简单地使用 `offset` 和 `limit` 键值来为我们的查找方法，如下所示：

```js
MyModel.findAll({ offset: 5, limit: 10 });
```

这将选择 `MyModel` 表，限制为 10 行，偏移量为 5。

注意

`limit` 属性会告诉数据库只检索指定数量的行（在 MSSQL 中，这将是 `SELECT TOP N` 或 `FETCH NEXT N ROWS`）。`offset` 属性会在检索结果之前跳过 N 行。对于 MSSQL 2008（及更早版本）的用户，Sequelize 将通过嵌套查询来模拟偏移行为，以实现兼容性和完整性。

现在我们已经完成了对 Sequelize 查询和操作数据方法的引用，我们现在可以讨论在定义模型时更高级的选项。这些选项可以改变 Sequelize 内部转换数据的方式，过滤查询数据，并调整命名约定，使你能够更好地适应 Sequelize 的行为以满足你公司/项目的需求。

# 定义模型的高级 Sequelize 选项

在 Sequelize 中定义模型时，`init()` 和 `define()` 方法的最后一个输入参数为我们提供了一种微调项目需求和 Sequelize 行为的方式。这些参数选项对于需要在我们不遵循 Sequelize 命名约定（例如，列名为 `PersonIsMIA` 而不是 Sequelize 的 `"PersonIsMia"` 约定）的现有环境中构建 Sequelize 的情况非常有用。

## sequelize

一个（或新的）Sequelize 实例，用于与模型关联。如果未提供此字段，Sequelize 将返回错误（除了使用 `sequelize.define` 方法时）。这对于跨数据中心或数据库查询非常有用。

## modelName

明确使用字符串定义模型的名称。这将是 Sequelize 的 `define()` 方法的第一个参数。如果你正在使用 ES6 类定义，此值的默认值将是类名。

## defaultScope/scopes

一个对象，用于设置模型的默认作用域和为模型设置适用的作用域。作用域对于代码组织或强制执行基本访问控制列表作为默认行为非常有用。我们将在后面的章节中详细介绍作用域。

## omitNull

将此布尔值设置为 `true` 将告诉 Sequelize 在保存记录时省略任何具有 `null` 值的列。

## timestamps

此选项允许我们控制 Sequelize 为模型添加 `createdAt` 和 `updatedAt` 时间戳列的行为。此设置的默认值为 `true`（Sequelize 将创建时间戳列）。

注意

你可以通过在你的模型中显式定义它们来始终覆盖 `createdAt` 和 `updatedAt` 属性的默认设置。Sequelize 将知道使用这些属性来处理时间戳相关的列。

## paranoid

当此布尔选项设置为 `true` 时，将防止 Sequelize 删除数据（默认行为）并添加 `deletedAt` 时间戳列。为了使 `paranoid` 选项生效，必须将 `timestamps` 选项设置为 `true`。`paranoid` 的默认值为 `false`。

以下查询将执行“软删除”：

```js
await Post.destroy({
  where: {
    id: 1
  }
});
```

此查询将更新 ID 为 1 的 `Post` 记录并更新 `deletedAt` 列。如果我们想从数据库中删除记录（而不是更新它），我们将使用 `force` 参数，如下面的代码片段所示：

```js
await Post.destroy({
  where: {
    id: 1
  },
  force: true
});
```

这将执行数据库上的 `delete` 查询而不是 `update` 查询。

## createdAt/updatedAt/deletedAt

此选项将分别重命名 `createdAt`、`updatedAt` 和 `deletedAt` 属性。如果你提供驼峰式命名的值，并且设置了下划线选项为 `true`，Sequelize 将自动转换列的命名格式。将值设置为 `false` 而不是字符串将告诉 Sequelize 禁用该列的默认行为。

## underscored

默认情况下，Sequelize 将使用驼峰式命名创建列（例如，`updatedAt`、`firstName` 等）。如果你更喜欢下划线或蛇形命名（例如，`updated_at`、`first_name` 等），则应将此值设置为 `true`。

## freezeTableName

如前所述，Sequelize 默认会将从模型名称派生的表名进行复数化。将此值设置为 `true` 将防止 Sequelize 转换表名。

## tableName

明确定义 Sequelize 在创建 SQL 查询时使用的表名。此选项的典型用例是将 Sequelize 集成到现有的数据库/模式中，或者当复数设置不正确时。

## name

一个对象，包含两个可用的选项来定义在将此模型与其他模型关联时使用的单数和复数名称。在后面的章节中介绍模型关联和关系时，我们将提供更清晰的解释和示例，但你可以在此处查看这两个选项的概述：

+   `singular`—在引用模型的单个实例时使用的名称（默认为 `Sequelize.Utils.singularize(modelName)`）

+   `pluralize`—在引用模型的多实例时使用的名称（默认为 `Sequelize.Utils.pluralize(modelName)`）

## schema

定义模型的模式（在 Postgres 中这将被称为 `search_path`）。并非所有数据库都支持模式，有些会将模式完全视为数据库。

## engine

仅适用于 MySQL，这是您定义表引擎类型的地方（通常是 `InnoDB` 或 `MyISAM`）。默认为 `InnoDB`。

## charset

指定表的字符集。当您的表内容可以确定性地定义为一组字符，这有助于减少数据库大小时很有用（如果您不需要通用编码，并且只需要拉丁字符，则应使用拉丁派生的字符集）。

## collation

指定表的校对（字符的排序和排序规则）。

## comment

为表添加注释（如果适用）。

## initialAutoIncrement

为适用方言设置初始 `AUTO_INCREMENT` 值（MySQL 和 MariaDB）。

## 钩子

一个对象，其键映射到钩子（或生命周期事件）。值可以是函数或函数数组。我们将在下一章详细介绍钩子。

## validate

一个对象，用于定义模型验证。我们将在下一章详细介绍验证。

## indexes

定义表索引定义的对象数组。这些索引在调用 `sync()` 或使用迁移工具时创建。每个对象都有以下选项：

+   `name`—索引的名称（Sequelize 将默认使用模型的名称和通过下划线连接的相关字段）。

+   `type`—用于定义索引类型的字符串值（仅限 MySQL/MariaDB）。通常，您会在这里定义 `FULLTEXT` 或 `SPATIAL` 索引（`UNIQUE` 也可以，但有一个与方言无关的选项用于创建唯一索引）。

+   `unique`—将此值设置为 true 将创建一个唯一索引。

+   `using`—索引 SQL 语句的 `USING` 子句值。一些示例包括 `BTREE`（通常，数据库管理系统将使用此索引类型作为默认值），`HASH`（仅限 MySQL/MariaDB/Postgres），以及 `GIST`/`GIN`（仅限 Postgres）。

+   `operator`—定义用于此索引的运算符（主要用于 Postgres，但也可以用于其他方言）。

+   `concurrently`—将此设置为 `true` 提供了一种创建索引而不写入锁的方法（仅限 Postgres）。

+   `fields`—为模型定义的索引字段数组。请参阅下一节的 *索引字段*。

### 索引字段

每个索引定义的字段值可以是以下之一：

+   表示索引名称的字符串

+   Sequelize 文字对象函数（例如，`sequelize.fn()`）

+   一个具有以下键的对象：

    +   `attribute`—用于索引的列的字符串值

    +   `length`—定义前缀索引的长度（如果适用）

    +   `order`—确定排序是升序还是降序

    +   `collate`—定义列的校对规则

这里提供了一个快速示例，说明如何在定义 Sequelize 模型时使用一些这些高级选项：

```js
class User extends Model { }
User.init({
    name: DataTypes.STRING,
}, {
    sequelize,
    modelName: 'User',
    omitNull: true,
    // renames deletedAt to removedAt
    deletedAt: 'removedAt',
    // start with ID 1000
    initialAutoIncrement: 1000,
    validate: {
        isNotBob() {
            if (this.name === 'bob') {
                throw new Error("Bob is not allowed to be a 
                                 user.");
            }
        }
    },
    indexes: [
        { unique: true, fields: ['name'] }
    ],
});
```

# 摘要

在本章中，我们详细概述了使用 Sequelize 定义模型的各种参数和配置设置。我们还学习了如何使用 Sequelize CLI 自动生成模型定义（和数据）文件，以及如何将这些定义迁移到数据库。本章还涵盖了 Sequelize 提供的各种属性类型，以及从 Sequelize 查询或更新数据到数据库的方法。

在下一章中，我们将讨论模型验证、建立外键关系以及如何约束数据以满足项目需求。

# 第二部分 – 验证、自定义和关联您的数据

在本部分，您将更深入地了解 Sequelize 的模型属性，并添加验证、自定义列类型和相关关联模型。您将探索钩子、JSON 和 Blob 类型，以及事务。

本部分包括以下章节：

+   *第三章*, *验证模型*

+   *第四章*, *关联模型*

+   *第五章*, *将钩子和生命周期事件添加到您的模型中*

+   *第六章*, *使用 Sequelize 实现事务*

+   *第七章*, *处理自定义、JSON 和 Blob 数据类型*
