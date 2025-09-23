# 第九章. 持久化数据

大多数应用程序都需要持久化某种类型的数据。在本章中，我们将探讨 Node.js 应用程序的数据持久化的一些方法。

在很长一段时间内，持久化的默认选择一直是传统的数据库。你可能使用过**RDBMS**（关系型数据库管理系统），例如 Microsoft SQL Server、Oracle、MySQL 或 PostgreSQL。这些系统通常被归类为*SQL 数据库*，因为它们都使用 SQL 作为它们的查询语言。

最近，所谓的**NoSQL**数据库的数量激增。这个总称作为分类并不特别有用。一些 NoSQL 数据库与传统的关系型数据库没有更多的共同之处。

最有趣的是可用的数据库范围以及它们所满足的使用案例。传统的 RDBMS（关系型数据库管理系统）依然强大且灵活，是许多情况下的正确选择。在本章中，我们将考虑其他两种类型的数据库，以及如何和何时使用它们。

我们将要探讨的系统是**MongoDB**和**Redis**。这两个系统都于 2009 年首次发布，现在被广泛使用。深入探讨其中的任何一个都足以写成一本书。本章的目标是提供一个对每个系统的介绍和高级概述。

在本章中，我们将涵盖以下主题：

+   这些系统使用的概念数据模型

+   它们提供最大利益的用例

+   将它们与 Express 应用程序集成

+   测试数据持久化代码

# 介绍 MongoDB

MongoDB 是一个**面向文档**的 DBMS。MongoDB 文档以**二进制 JSON**（**BSON**）的形式存储。这类似于 JSON，但支持更多的数据类型。JSON 字段值只能是字符串、数字、对象、数组、布尔值或 null。BSON 支持更具体的数值类型、日期和时间戳、正则表达式和二进制数据。正如其名所示，BSON 文档以二进制数据的形式存储和传输。这比 JSON 的字符串表示可能更高效。

MongoDB 文档存储在**集合**中。它们在传统的关系型数据库中的工作方式非常相似。文档可以被插入、更新和查询。与传统的关系型数据库有两个关键的区别：

+   MongoDB 不支持服务器端连接。在传统的 RDBMS 中，你会将数据规范化到多个表中，并通过外键在它们之间进行连接。在 MongoDB 中，你相反地使用 BSON 的嵌套结构将每个实体的数据非规范化到一个单独的文档中。

+   关系型数据库的*关系*属性是表中的所有行都包含相同字段，且具有相同的意义。在 MongoDB 中，文档可以拥有任何一组字段。

在实践中，同一集合中的文档通常具有相同的字段，或者至少有一个共同的字段集。MongoDB 支持在集合的常用字段上创建索引，以提高查询效率。

## 为什么选择 MongoDB？

MongoDB 有一些特性使其在某些用例中具有吸引力，尤其是在基于 Node.js 的应用程序中。我们将在本节中介绍这些内容。

### 对象建模

MongoDB 的基于文档的方法非常适合持久化领域实体。你可能有过在关系型数据库中使用**对象关系映射器**（**ORM**）存储领域实体的经验。Hibernate 和 Entity Framework 是 ORM 的流行例子。ORM 执行的一项工作是将单个实体映射到规范化模式中的多个表。当实体从数据库加载时，它通过这些表之间的`JOIN`查询重建。这是 ORM 的关键特性之一。这也是使用 ORM 时配置问题和性能问题最常见的原因之一。MongoDB 将每个实体持久化为单个文档，这可以简单得多。

当然，跨表连接也可以用于遍历实体之间的关系。虽然 ORM 通常使这变得容易，但这本身也可能成为性能问题的来源。相关实体的隐式加载经常导致*N+1*问题，发出数千个数据库查询。无论使用哪种类型的数据库，妥善处理这些关系都需要仔细思考。

当使用 ORM 和 RDBMS 时，所有实体间的关系都是外键，但你需要仔细考虑如何加载它们。在 MongoDB 中建模数据时，你必须选择嵌入文档或文档引用来表示实体间的关系。在任一技术栈下，设计决策取决于应用程序的数据访问需求，以及设计数据模型以减少实体间关系的普遍性将简化问题。

### JavaScript

MongoDB 特别适合 Node.js。JSON-like 格式的使用很好地映射到基于 JavaScript 的编程环境。MongoDB 本身也原生支持 JavaScript。数据库操作可以利用在服务器上执行的定制 JavaScript 函数。

### 可扩展性

MongoDB 的扩展方式与 Node.js 类似。它使用分区和复制来支持在通用硬件上的水平扩展。没有技术原因要求你的应用程序和数据库必须以相同的方式扩展，但从业务角度来看，这可能更容易规划可扩展性。

当使用 RDBMS 时，垂直扩展数据库更为直接。这意味着提供一台高性能的数据库服务器，它可以支持多个应用程序服务器。这比水平扩展应用程序和数据库服务器需要更仔细的计划和更多的前期投资。

## 开始使用 MongoDB

访问 [`www.mongodb.org/downloads`](https://www.mongodb.org/downloads) 下载并安装适用于您操作系统的最新版本的 MongoDB 社区服务器版。用户手册中有更详细的安装说明，请参阅 [`docs.mongodb.org/manual/installation/`](https://docs.mongodb.org/manual/installation/)。

本节余下的命令使用了 MongoDB `/bin` 目录中的可执行文件。您可以在该目录中运行这些命令，或者更好的做法是将它添加到您的 `PATH`。

为 MongoDB 创建一个目录以存储其数据。然后启动 MongoDB 守护进程（即服务），如下提供该目录的路径：

```js
> mongod --dbpath C:\data\mongodb

```

### 使用 MongoDB shell

您可以使用 MongoDB 的内置 shell 应用程序从控制台与 MongoDB 交互。您可以通过运行 `mongo` 命令来启动 MongoDB shell，如下所示：

```js
> mongo demo

```

这将连接到本地服务器上名为 `demo` 的数据库（如果需要则创建它）。如果您没有指定数据库，则 shell 将连接到名为 `test` 的数据库。

首先要注意的是，shell 只是另一个 JavaScript 环境。我们可以尝试运行与第二章开头相同的某些命令，*Node.js 入门*。

```js
> function square(x) { return x*x; }
> square(42)
1764
> new Date()
ISODate("2016-01-01T20:05:39.652Z")
> var foo = { bar: "baz" }
> typeof foo
object
> foo.bar
baz

```

正如 Node.js 通过在服务器端应用程序开发中构建在 JavaScript 之上，使其更适合服务器端应用程序开发一样，MongoDB 添加了更多对数据持久性有用的功能。请注意，前面代码中的 `new Date()` 返回一个 ISODate，这是 MongoDB 在 BSON 文档中表示日期的标准数据类型。

您可以随时通过输入 `exit` 来退出控制台。

MongoDB 还添加了一些新的全局变量，用于与数据库交互。其中最重要的是 `db` 对象。让我们尝试向我们的数据库添加一些文档。回想一下，MongoDB 将文档存储在集合中。要创建一个新的集合，我们只需要开始向其中插入文档。作为一个简单的例子，我们将使用 2016 年的英国银行假日。我们可以使用以下脚本填充这个集合：

```js
db.holidays.insert(
  { name: "New Year's Day", date: ISODate("2016-01-01") });
db.holidays.insert(
  { name: "Good Friday", date: ISODate("2016-03-25") });
db.holidays.insert(
  { name: "Easter Monday", date: ISODate("2016-03-28") });
db.holidays.insert(
  { name: "Early May bank holiday", date: ISODate("2016-05-02") });
db.holidays.insert(
  { name: "Spring bank holiday", date: ISODate("2016-05-30") });
db.holidays.insert(
  { name: "Summer bank holiday", date: ISODate("2016-08-29") });
db.holidays.insert(
  { name: "Boxing Day", date: ISODate("2016-12-26") });
db.holidays.insert(
  { name: "Christmas Day", date: ISODate("2016-12-27"),
    substitute_for: ISODate("2016-12-25") });
```

注意，2016 年的圣诞节在星期日，因此银行假日发生在下一个工作日。这给了我们一个理由来有一个只与集合中某些文档相关的字段。

您可以将这些 `insert` 命令手动输入到控制台中，但告诉 MongoDB 从脚本文件中加载它们会更简单：

```js
> mongo demo holidays.js --shell

```

之前的命令连接到名为 demo 的数据库，运行了（书中配套代码中的）`holiday.js` 脚本，然后打开一个 shell 以允许我们与数据库交互。我们可以在 MongoDB 控制台中运行以下命令来查看集合的完整内容：

```js
> db.holidays.find()
{ "_id" : ObjectId("572f760fffb6888d70c45eeb"), "name" : "New Year's Day", "date" : ISODate("2016-01-01T00:00:00Z") }
{ "_id" : ObjectId("572f7610ffb6888d70c45eec"), "name" : "Good Friday", "date" : ISODate("2016-03-25T00:00:00Z") }
...

```

注意，MongoDB 已经为我们自动在每个文档中添加了一个 `_id` 字段。

### 小贴士

您可以通过查看 `insert` 方法的源代码来了解 MongoDB 是如何做到这一点的。只需在 shell 中输入 `db.holidays.insert`（不带括号）即可。

我们可以通过 `_id` 或其他单个字段提取记录：

```js
> db.holidays.find({name: "Boxing Day"})

```

这将返回与传递给 `find` 的对象匹配的所有对象。要按除精确相等之外的其他方式查找文档，我们可以使用 MongoDB 的查询运算符。这些运算符以美元符号为前缀，并指定为对象属性。例如，要查找下半年的假日，我们可以使用如下所示的 *大于等于* 运算符：

```js
> db.holidays.find({ date: { $gte: new Date("2016-07-01") }})

```

MongoDB 的 **聚合管道** 允许我们通过一系列称为 **管道阶段** 的操作构建复杂的查询。这是 MongoDB 中与 SQL 中的复杂查询最接近的东西。在这里，我们使用 MongoDB 的 `$group` 管道阶段来计算每个月的银行假日数量，这与 SQL 的 `GROUP BY` 子句类似：

```js
> db.holidays.aggregate({
 $group: { _id: { $month: "$date" }, count: { $sum: 1 } }})

```

2016 年日历的一个奇怪特性意味着圣诞节银行假日实际上是在节礼日之后（因为圣诞节本身是在星期日）。在以下示例中，我们按标记的节日日期（如果与银行假日日期不同，则存储在 `$substitute_for` 字段中）对银行假日进行排序：

```js
> db.holidays.aggregate([
 { $project: { _id: false, name: "$name",
 date: { $ifNull: ["$substitute_for", "$date"] } } },
 { $sort: { date: 1 } }
])

```

之前的管道由两个阶段组成：

+   `$project` 阶段指定一组基于底层数据的字段（类似于 SQL 中的 `SELECT`）。请注意，`_id` 字段默认包含在内，但在这里我们将其排除。

+   `$sort` 阶段指定一个排序字段和方向（类似于 SQL 的 `SORT BY` 子句）。这里的 `1` 表示升序排序。

我们在这里只是触及了表面。MongoDB 中还有许多其他管道阶段可用。您可以在 MongoDB 文档中了解更多关于聚合的信息，请参阅 [`docs.mongodb.com/manual/core/aggregation-pipeline/`](https://docs.mongodb.com/manual/core/aggregation-pipeline/)。

MongoDB 还有一个内置的 Map-Reduce 函数，用于使用任意 JavaScript 函数进行强大的聚合数据处理。这超出了本书的范围，但您可以在 [`docs.mongodb.com/manual/core/map-reduce/`](https://docs.mongodb.com/manual/core/map-reduce/) 中了解更多关于 Map-Reduce 和 MongoDB 对其的实现。

# 使用 MongoDB 与 Express

我们应用程序中的游戏服务模块目前将其所有数据存储在内存中。这对于演示目的来说已经足够好了，但并不适合实际应用。每次应用程序重启时，我们都会丢失所有数据。这也阻止了我们跨多个进程扩展应用程序。每个实例都会有自己的游戏服务，具有不同的数据。用户会看到不同的数据，这取决于哪个服务器恰好处理了他们的请求。

我们将更新我们的游戏服务，以便将其数据存储在 MongoDB 中。为此，我们将使用一个名为 **Mongoose** 的库。

## 使用 Mongoose 持久化对象

请记住，与关系型数据库不同，MongoDB 不要求同一集合中的文档具有相同的字段。然而，我们通常期望集合中的大多数项目至少共享一个共同的字段核心。

Mongoose 是一个用于在 MongoDB 中存储实体的对象建模库。它帮助编写常见的功能，如验证、查询构建和类型转换。它还提供了将业务逻辑与我们的实体关联的钩子。这些功能与 ORM（如 Entity Framework 或 Hibernate）提供的一些功能类似。然而，Mongoose 本身不是一个 ORM。回想一下，对象关系映射对于文档数据库（如 MongoDB）来说是不相关的。

要使用 Mongoose，我们首先定义一个**模式**。这定义了 MongoDB 集合中文档的常见字段。回到前几章的演示应用程序，让我们安装 Mongoose 并为我们的游戏集合定义一个模式：

```js
> npm install mongoose --save

```

以下代码添加到`src/services/games.js`中：

```js
'use strict';

const mongoose = require('mongoose');

const Schema = mongoose.Schema;
const gameSchema = new Schema({
 word: String,
 setBy: String
});
```

模式定义了文档字段并指定了每个字段的类型。为了使用此模式开始持久化文档，我们需要创建一个**模型**。

模型是与 MongoDB 集合对应的构造函数。Mongoose 模型的实例对应于该集合中的文档。模型还提供了修改集合的函数。我们通过指定模式和（单数）集合名称来创建模型：

```js
const gameSchema = new Schema({
    word: String,
    setBy: String
});

const Game = mongoose.model('Game', gameSchema);

```

模型构造函数替换了之前的游戏类及其构造函数。这个类还包含两个实例方法：`positionsOf`和`remove`。我们可以在模式上定义自定义实例方法，这些方法将在所有模型实例上可用。这些方法必须在创建模型之前定义：

```js
const gameSchema = new Schema({
    word: String,
    setBy: String
});

gameSchema.methods.positionsOf = function(character) {
 let positions = [];
 for (let i in this.word) {
 if (this.word[i] === character.toUpperCase()) {
 positions.push(i);
 }
 }
 return positions;
};

const Game = mongoose.model('Game', gameSchema);
```

### 注意

注意，在前面的代码中，我们使用的是传统的函数定义，而不是箭头函数。这对于函数内部的`this`关键字正确工作来说是必要的。更多详情请见[`derickbailey.com/2015/09/28/do-es6-arrow-functions-really-solve-this-in-javascript/`](http://derickbailey.com/2015/09/28/do-es6-arrow-functions-really-solve-this-in-javascript/)。

我们不再需要定义`remove`方法，因为 Mongoose 会自动提供这个方法。它还提供了一个`save`方法，我们可以用它来持久化新的游戏：

```js
const Game = mongoose.model('Game', gameSchema);

module.exports.create = (userId, word) => {
 let game = new Game({setBy: userId, word: word.toUpperCase()});
 return game.save();
};
```

我们不再需要指定 ID，因为 Mongoose 也会提供这个 ID。注意，我们确实需要指定`word.toUpperCase()`，这曾经位于游戏构造函数中。这不是问题，因为构造函数是模块私有的。模块外部的代码不能直接调用构造函数。`toUpperCase`调用发生的地方只是一个实现细节。

还要注意，Mongoose 的所有异步操作都返回承诺作为使用回调的替代方案。Mongoose 支持前一章中讨论的两种异步编程模式。Mongoose 使用自己的承诺实现。尽管如此，我们可以配置 Mongoose 使用 ECMAScript 6 承诺。我们还需要告诉 Mongoose 连接到 MongoDB 数据库。目前，我们将硬编码 URL，但我们将很快看到如何使其可配置：

```js
const mongoose = require('mongoose');
mongoose.Promise = Promise;
mongoose.connect('mongodb://localhost/hangman');

```

最后，我们需要实现从数据库检索游戏的三种方法。我们可以使用 Mongoose 的 `find` 方法来完成此操作：

```js
module.exports.create = (userId, word) => {
    ...
};
module.exports.createdBy =
 (userId) => Game.find({setBy: userId});
module.exports.availableTo =
 (userId) => Game.find({setBy: { $ne: userId } });
module.exports.get =
 (id) => Game.findById(id);

```

Mongoose 的 `find` 方法与我们在上一节中在 *使用 MongoDB shell* 中看到的 MongoDB `find` 方法类似。它接受一组 MongoDB 查询条件，并异步提供文档列表。`findById` 接受一个 ID 并异步提供一个单独的文档，或 null。

Mongoose 还提供了一个 `where` 方法，可以通过函数调用构建条件。`availableTo` 函数可以重写如下：

```js
module.exports.availableTo =
  (userId) => Game.where('setBy').ne(userId);
```

只要您仍然在本地上运行 MongoDB（如本章前面在 *MongoDB 入门* 中所述），现在应该能够运行应用程序。尝试停止并重新启动应用程序，并注意游戏现在可以在重启之间持久化。

## 隔离持久化代码

将代码与真实数据库集成以验证我们的持久化代码是否正常工作是有用的。但并非总是适合让我们的测试依赖于外部 MongoDB 实例。

我们希望开发者能够检出代码并运行应用程序，而无需运行数据库实例。此外，外部依赖项会减慢我们的测试速度。MongoDB 在磁盘上存储数据，因此给我们的测试引入了额外的 I/O 工作。

应用程序在生产中应依赖于外部数据库。在集成测试中，我们希望在本地服务器上使用真实数据库。在开发机器上，默认使用内存数据库会更好。因此，我们需要能够配置数据库 URL，并在开发环境中回退到内存数据库。

最后，我们需要在使用我们的游戏服务之前初始化 Mongoose。这包括指定数据库 URL 并等待建立连接。这是异步发生的，因此不能成为游戏服务模块定义的一部分。我们也不想让游戏服务的客户端在每个函数调用中传递 Mongoose 实例。

通过在我们的应用程序中引入依赖注入，我们可以解决所有这些问题。我们将把游戏服务作为依赖项传递给需要它的模块，并将 Mongoose 作为依赖项传递给游戏服务。

### 小贴士

这也将给我们提供为传递给游戏服务本身的测试双胞胎的模块编写单元测试的选项，因此不要使用 MongoDB。在更大的应用程序中，这种测试隔离对于编写快速且可维护的测试非常重要。

## Node.js 中的依赖注入

你可能已经使用过 .NET 或 Java 中的 **依赖注入**（**DI**）框架，如 Unity、Autofac、NInject 或 Spring。这些框架提供了声明性配置和自动绑定依赖项的功能。JavaScript 也有类似的 DI 容器。然而，在 JavaScript 中，显式传递依赖项更为常见。JavaScript 的模块模式使这种方法比其他语言更自然。我们不需要添加很多字段和构造函数/属性来设置依赖项。我们只需将模块包裹在一个初始化函数中，该函数接受依赖项作为参数。

在我们的应用程序中，`app` 模块将连接一切。整个应用程序依赖于数据库。游戏和索引路由依赖于游戏服务。为了允许路由依赖游戏服务，我们只需要用函数来首尾包裹它们：

```js
'use strict';

module.exports = (gamesService) => {
    var express = require('express');
    var router = express.Router();
    ...
 return router;
};

```

游戏服务本身稍微复杂一些。我们之前向`module.exports`添加了几个函数，所以我们需要将这些函数放在一个对象上。然而，这实际上使代码更短。此外，请注意，我们只在`Game`模式尚未定义的情况下创建它，以防止我们的导出函数被多次调用：

```js
module.exports = (mongoose) => {
    'use strict';

 let Game = mongoose.models['Game'];

 if (!Game) {
        const Schema = mongoose.Schema;
        const gameSchema = new Schema({
            word: String,
            setBy: String
        });

        gameSchema.methods.positionsOf = function(character) {
            ...
        };

        Game = mongoose.model('Game', gameSchema);
 }

 return {
 create: (userId, word) => {
 const game = new Game({
 setBy: userId, word: word.toUpperCase()
 });
 return game.save();
 },
 createdBy: userId => Game.find({setBy: userId}),
 availableTo: userId => Game.where('setBy').ne(userId),
 get: id => Game.findById(id)
 };
};

```

最后，应用程序本身依赖于数据库连接，并连接其他依赖项：

```js
module.exports = (mongoose) => {
    ...

 let gamesService = require('./services/games')(mongoose);
 let routes = require('./routes/index')(gamesService);
 let games = require('./routes/games')(gamesService);
    ...

 return app;
};
```

## 提供依赖项

我们可以在环境变量中指定数据库 URL。如果这个变量不存在，我们的应用程序将使用 MongoDB 的内存实例。这将由一个名为 **Mockgoose** 的库提供。我们将它作为一个开发依赖项安装，以防我们在生产服务器上忘记设置环境变量。如果发生错误，我们将不会安静地使用非持久性数据库。

```js
> npm install mockgoose@~5.x --save-dev

```

我们在`src/config/mongoose.js`下创建一个新的模块来初始化 Mongoose，并返回一个当它连接到数据库时将解决的承诺：

```js
'use strict';

const mongoose = require('mongoose');
const debug = require('debug')('hangman:config:mongoose');

mongoose.Promise = Promise;
if (!process.env.MONGODB_URL) {
    debug('MongoDB URL not found. Falling back to in-memory database...');
    require('mockgoose')(mongoose);
}

let db = mongoose.connection;
mongoose.connect(process.env.MONGODB_URL);
module.exports = new Promise(function(resolve, reject) {
    db.once('open', () => resolve(mongoose));
    db.on('error', reject);
});
```

现在我们只需要将这个模块传递给我们的应用程序。以下是从`bin/www`的代码：

```js
...

require('../src/config/mongoose').then((mongoose) => {
 var app = require('../src/app')(mongoose);
    ...
    server.on('listening', onListening);
}).catch(function(error) {
 console.log(error);
 process.exit(1);
});

```

为了允许我们的测试运行，我们还需要添加新的`before`函数来使用这个模块。以下代码来自`test/services/games.js`：

```js
'use strict';

const expect = require('chai').expect;

describe('Game service', () => {
  const firstUserId = 'user-id-1';
  const secondUserId = 'user-id-2';

 let service;
 before(done => {
 require('../../src/config/mongoose.js').then((mongoose) => {
 service = require('../../src/services/games.js')(mongoose);
 done();
 }).catch(done);;
 });
  ...
```

以下代码来自`test/routes/games.js`：

```js
'use strict';

const request = require('supertest');
const expect = require('chai').expect;

describe('/games', () => {
  let agent, userId;
 let mongoose, gamesService, app;

 before(function(done) {
 require('../../src/config/mongoose.js').then((mongoose) => {
 app = require('../../src/app.js')(mongoose);
 gamesService =
 require('../../src/services/games.js')(mongoose);
 done();
 }).catch(done);
 });

    ...
```

我们还会添加一个全局清理函数，在所有测试完成后关闭数据库连接。这只是一个在任意`describe`块上下文之外的 Mocha `after` 钩子。我们在`test/global.js`下的一个新文件中添加这个函数：

```js
'use strict';

after(function(done) {
    require('../src/config/mongoose.js').then(
        (mongoose) => mongoose.disconnect(done));
});
```

最后，我们需要更新我们的`gulpfile.js`，以便我们的集成测试能够运行新的依赖项：

```js
gulp.task('integration-test',
        ['lint-integration-test', 'test'], function(done) {
    const TEST_PORT = 5000;

 require('./src/config/mongoose.js').then((mongoose) => {
 let server, teardown = (error) => {
 server.close(() =>
 mongoose.disconnect(() => done(error)));
 };
 server = require('http')
 .createServer(require('./src/app.js')(mongoose))
            .listen(TEST_PORT, function() {
                gulp.src('integration-test/**/*.js')
                    .pipe(
                        ...
                    )
 .on('error', teardown)
 .on('end', teardown)
            });
 });
});
```

我们现在可以在本地开发机器上运行我们的应用程序和测试，而无需运行 MongoDB，或者如果我们想使用真实的 MongoDB 实例，可以指定`MONGO_DB`环境变量。

## 在 Travis CI 上运行数据库集成测试

我们确实希望定期对我们的应用程序进行集成测试，以针对真实的 MongoDB 实例。幸运的是，Travis CI 作为其环境的一部分提供了各种数据存储。我们只需要将其添加到我们的`travis.yml`文件中，告诉它我们的构建需要 MongoDB。我们还需要设置`MONGODB_URL`环境变量，以便测试能够连接到数据库：

```js
services:
 - mongodb
env:
 global:
 - MONGODB_URL=mongodb://localhost/hangman

```

现在我们可以使用适合开发机器和 CI 服务器的合适 MongoDB 实例来运行我们的应用程序以及单元和集成测试。

# 介绍 Redis

Redis 通常被归类为**键值**数据存储。Redis 将自己描述为数据结构存储。它提供了与大多数编程语言中找到的基本数据结构类似的数据存储类型。

## 为什么使用 Redis？

Redis 完全在内存中运行，这使得它非常快。这一点，加上其键值特性，使其非常适合用作缓存。它还支持发布/订阅通道，这使得它可以作为一个消息代理。我们将在第十章创建实时 Web 应用中进一步探讨。

更普遍地，Redis 可以作为有用的后端，允许多个 Node.js 进程相互协调。Node.js 水平扩展，大多数网站都会运行多个 Node.js 进程。许多网站有“工作”数据，这些数据不需要长期持久化，但需要在所有进程中快速且一致地可用。Redis 的内存特性和一系列原子操作使其非常适合这个用途。

Redis 更注重速度而不是持久性。有多种配置选项，但所有选项都预期在故障发生时会有一定程度的数据丢失。这是 Redis 为了速度而在内存中完全运行所做的妥协。可以通过在故障前不显著影响速度的情况下将数据丢失减少到最多一秒前的写入，来减少数据丢失。Redis 可以通过在每次操作后同步到磁盘来完全最小化数据丢失，但这会对性能产生更大的影响，并抵消了 Redis 内存特性的优势。

## 安装 Redis

Redis 的源代码分发可以从[`redis.io/download`](http://redis.io/download)获取。

对于 Windows 系统，下载预构建的二进制文件更有用。它可以通过 NuGet 和 Chocolatey 作为签名包提供。如果你有 Chocolatey，可以通过运行以下命令来安装 Redis：

```js
> choco install redis-64

```

或者，你可以从[`github.com/MSOpenTech/redis/releases`](https://github.com/MSOpenTech/redis/releases)下载未签名的安装程序版本。

安装完成后，可以通过运行`redis-server`来启动 Redis。在另一个窗口中，运行`redis-cli`以连接到服务器并运行命令。

## 将 Redis 用作键值存储

Redis 中的所有内容都是针对键进行存储的。Redis 中的键可以是任何二进制数据，但最好将它们视为字符串。各种类型的值可以存储在每个键下。

Redis 将简单的标量值称为**字符串**。Redis 还对标量整数有特殊处理。以下示例设置和更新名为`counter`的键：

```js
127.0.0.1:6379> set counter 100
OK
127.0.0.1:6379> get counter
"100"
127.0.0.1:6379> incr counter
(integer) 101

```

这个增量操作是原子的。Redis 还支持原子地设置值。以下命令将失败，因为键已经存在：

```js
127.0.0.1:6379> set counter 200 nx
(nil)

```

这些功能可以帮助服务器之间的协调。Redis 还支持为键设置过期时间。这使得提供类似于 memcache 的缓存行为成为可能。然而，Redis 的灵活性更大，我们将在下一节中看到。

## 在 Redis 中存储结构化数据

除了简单的键值对之外，Redis 还支持其他更结构化的数据类型。

**列表**是有序的值集合。它们作为链表而不是数组存储。这使得在列表的末尾添加/删除元素效率更高（以牺牲通过索引检索列表项的速度为代价），例如：

```js
127.0.0.1:6379> rpush fruit apple banana pear
(integer) 3
127.0.0.1:6379> rpop fruit
"pear"
127.0.0.1:6379> lpush fruit orange
(integer) 3
127.0.0.1:6379> lrange fruit 0 -1
1) "orange"
2) "apple"
3) "banana"

```

注意，`lrange`接受起始和结束索引。负值从列表的末尾开始计数，因此`-1`指的是最后一个元素。能够从列表的两端进行 push/pop 意味着它们可以用作栈或队列，例如，允许进程以生产者-消费者模式进行通信。

**哈希**是一组字段-值对。它们不如 MongoDB 文档丰富，但允许我们将一些数据关联在一起。例如，我们可以使用 Redis 实现我们的游戏服务：

```js
127.0.0.1:6379> hmset game:2 word JavaScript setBy user-id-7
OK
127.0.0.1:6379> hget game:2 word
"JavaScript"
127.0.0.1:6379> hgetall game:2
1) "word"
2) "JavaScript"
3) "setBy"
4) "user-id-7"

```

注意，这里的顶级键`game:2`只是一个约定。对于开发者来说，以这种方式对键进行命名空间可能很有用，但 Redis 只将它们视为字符串。

**集合**是无序的值集合，例如：

```js
127.0.0.1:6379> sadd numbers one two three
(integer) 3
127.0.0.1:6379> smembers numbers
1) "two"
2) "three"
3) "one"

```

集合支持数学运算，如并集和交集。它们还支持检索（可选原子删除）随机元素。

**有序集合**是带有数值分数的值集合：

```js
127.0.0.1:6379> zadd votes 3 Aye
(integer) 1
127.0.0.1:6379> zadd votes 4 No
(integer) 1
127.0.0.1:6379> zadd votes 1 Abstain
(integer) 1
127.0.0.1:6379> zrevrange votes 0 1
1) "No"
2) "Aye"

```

注意，默认情况下，范围是有序的最小到最大。我们请求一个反向范围，以首先获取具有最高分数的元素。有序集合对于实现投票系统（如前所述）或排名系统很有用。

# 使用 Redis 构建用户排名系统

我们希望能够根据用户完成的游戏数量来对用户进行排名。我们将创建一个用户服务，该服务在 Redis 中实现，并提供以下功能：

+   记录用户成功完成游戏的时间

+   返回网站上的前三个用户

+   返回指定用户的排名

我们将首先添加一个功能，通过允许用户选择一个屏幕名来使网站更加用户友好。

## 从 Node.js 使用 Redis

首先，我们需要安装一个用于 Redis 的 Node.js 客户端库。我们还将使用 promise 库 Bluebird 将 Redis 客户端库转换为 promise：

```js
> npm install redis --save
> npm install bluebird --save

```

首先，我们将创建一个模块来配置 Redis 客户端，如下所示在 `src/config/redis.js`：

```js
'use strict';

const bluebird = require('bluebird');
const redis = require('redis');
bluebird.promisifyAll(redis.RedisClient.prototype);
module.exports = redis.createClient(process.env.REDIS_URL);
```

现在，我们可以在 `src/services/users.js` 中创建一个新的用户服务，其中包含获取和设置用户名的方法：

```js
'use strict';

let redisClient = require('../config/redis.js');

module.exports = {
    getUsername: userId =>
        redisClient.getAsync(`user:${userId}:name`),
    setUsername: (userId, name) =>
        redisClient.setAsync(`user:${userId}:name`, name)
};
```

注意，Redis 客户端为每个 Redis 命令提供了函数（如 `get` 和 `set`）。Bluebird 提供了每个函数的基于 promise 的版本，后缀为 `Async`。

当然，现在我们为我们的项目有了测试基础设施，我们应该在编写新代码时添加测试，如下所示 `test/services/users.js`：

```js
'use strict';

const expect = require('chai').expect;
const service = require('../../src/services/users.js');

describe('User service', function() {
    describe('getUsername', function() { 
        it('should return a previously set username', done => {
            const userId = 'user-id-1';
            const name = 'User Name';
            service.setUsername(userId, name)
                .then(() => service.getUsername(userId))
                .then(actual => expect(actual).to.equal(name))
                .then(() => done(), done);
        });

        it('should return null if no username is set', done => {
            const userId = 'user-id-2';

            service.getUsername(userId)
                .then(name => expect(name).to.be.null)
                .then(() => done(), done);
        });
    });
});
```

### 使用 redis-js 进行测试

就像我们对游戏服务的测试一样，我们希望能够与 CI 服务器上的 Redis 实例集成。但我们不希望为开发引入任何新的依赖项。这次，我们将使用一个名为 redis-js 的库进行本地测试。与 Mockgoose 不同，它不使用真实 DB 引擎的内存版本（Redis 已经是内存中的）。相反，这是一个 Node.js Redis 客户端的重新实现，它将所有数据存储在进程内：

```js
> npm install redis-js --save-dev

```

现在，我们可以创建一个模块来获取适合环境的 Redis 引用，如下所示 `src/config/redis.js`：

```js
'use strict';

const bluebird = require('bluebird');
const debug = require('debug')('hangman:config:redis');

if (process.env.REDIS_URL) {
 let redis = require('redis');
 bluebird.promisifyAll(redis.RedisClient.prototype);
 module.exports = redis.createClient(process.env.REDIS_URL);
} else {
 debug('Redis URL not found. Falling back to mock DB...');
 let redisClient = require('redis-js');
 bluebird.promisifyAll(redisClient);
 module.exports = redisClient;
}
```

注意，与 Mongoose 不同，Node.js Redis 客户端可以立即使用。在它连接之前发出的任何命令实际上都是内部排队。这意味着我们只需从模块中返回客户端并直接 require 它。在这种情况下使用 Mongoose 的依赖注入不会有任何好处。

我们还需要将 Redis 添加到我们的 `.travis.yml` 文件中，以便它在 CI 服务器上运行：

```js
services:
 - mongodb
 - redis-server
env:
  global:
    - MONGODB_URL=mongodb://localhost/hangman
 - REDIS_URL=redis://127.0.0.1:6379/

```

最后，一旦我们的测试完成，我们需要关闭客户端，就像我们使用 Mongoose 一样。我们还确保在启动时清空数据库（因为我们没有通过服务接口删除用户数据的方法，就像我们在游戏中做的那样）。以下代码来自 `test/global.js`：

```js
'use strict';

before(function(done) {
 require('../src/config/redis.js').flushdbAsync().then(() => done());
});

after(function(done) {
 require('../src/config/redis.js').quit();
    require('../src/config/mongoose.js').then(
        (mongoose) => mongoose.disconnect(done));
});
```

以下代码来自 `gulpfile.js`：

```js
        let server, teardown = (error) => {
 require('./src/config/redis.js').quit();
            server.close(() =>
                mongoose.disconnect(() => done(error)));
        };
```

## 使用 Redis 实现用户排名

现在我们已经准备好将用户排名功能添加到我们的服务中。以下代码来自 `src/services/users.js`：

```js
module.exports = {
  ...

  recordWin: userId =>
    redisClient.zincrbyAsync('user:wins', 1, userId),

  getTopPlayers: () =>
    redisClient.zrevrangeAsync('user:wins', 0, 2, 'withscores')
    .then(interleaved => {
      if (interleaved.length === 0) {
        return [];
      }
      let userIds = interleaved
        .filter((user, index) => index % 2 === 0)
        .map((userId) => `user:${userId}:name`);
      return redisClient.mgetAsync(userIds)
        .then(names => names.map((username, index) => ({
          name: username,
          userId: interleaved[index * 2],
          wins: parseInt(interleaved[index * 2 + 1], 10)
        })));
    }),

  getRanking: userId => {
    return Promise.all([
      redisClient.zrevrankAsync('user:wins', userId),
      redisClient.zscoreAsync('user:wins', userId)
    ]).then(out => {
      if (out[0] === null) {
        return null;
      }
      return { rank: out[0] + 1, wins: parseInt(out[1], 10) };
    });
  }
};
```

在这里使用的 Redis 命令大多数都来自本章前面的内容。最有趣的功能是 `getTopPlayers`。它使用带有 `withscores` 选项的 `zrevrange`。这返回一个包含用户 ID 和分数的数组（交错在一起）。我们使用 `mget`（多值获取）向数据库发出第二个请求，以检索所有用户的名称。一旦返回，我们就可以将每个用户的所有数据组合成一个对象。

## 利用用户服务

将此功能连接到我们的应用程序的其他部分不使用我们之前没有见过的技术，因此为了简洁起见，省略了打印的代码列表。完整的实现可以在本章的配套代码中找到，包括对用户服务其他方法的测试，在 [`github.com/NodeJsForDevelopers/chapter09`](https://github.com/NodeJsForDevelopers/chapter09)。

# 关于安全性的说明

我们一直使用 MongoDB 和 Redis 的默认出厂设置运行。这对于开发目的来说是不错的。将这些服务部署到生产环境中需要考虑额外的安全问题。您可以在[`docs.mongodb.com/manual/administration/security-checklist/`](https://docs.mongodb.com/manual/administration/security-checklist/)和[`redis.io/topics/security`](http://redis.io/topics/security)找到更多资源。

# 摘要

在本章中，我们已经了解了不同类型数据库之间的区别，并学习了 MongoDB 和 Redis 的关键特性。我们还使用这些数据库持久化我们的应用程序数据，并使用依赖注入使我们的应用程序更加灵活。我们还学习了如何配置我们的开发和集成环境以使用适当的数据库实例。

持久性可能被视为我们系统的底层。在下一章中，我们将引入实时客户端/服务器通信到我们的应用程序中。这种前端功能意味着更多地关注我们系统的顶层。然而，我们也会看到 Redis 在支持这一功能中扮演着重要的角色。
