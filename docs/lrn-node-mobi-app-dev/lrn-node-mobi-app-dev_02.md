# 第二章：配置 MongoDB 持久化

在本章中，我们将向您展示如何配置应用程序的持久化层，该层负责提供中央数据存储和检索服务。为此，我们将使用 MongoDB，这个广受欢迎的 NoSQL 数据库，以及其针对 Node.js 的驱动程序和接口。

在本章中，我们将涵盖以下主题：

+   配置文档、集合和数据库

+   使用产品订单数据库连接到 MongoDB 作为示例

+   创建文档之间的关系

+   查询数据并在命令行中显示结果

# MongoDB 的学习成果

阅读完本章后，您将对 MongoDB 存储数据的基础有扎实的理解。您还将学会如何对 MongoDB 实例运行查询，以便在其中存储、操作和检索数据。您还将了解如何使用 Node.js MongoDB 驱动程序来直接从 Node.js 操作数据。

最后，您将对 Node.js 的内部工作方式有一个健康的回顾，以帮助您理解它如何与 MongoDB 相互连接。

# MongoDB 简介

让我们从一个简短但信息丰富的 MongoDB 导览开始，这将为您提供必要的知识，以便有效地使用它。

首先，让我们深入了解一下数据在 MongoDB 实例中是如何组织的。这将为我们提供后续理解存储和检索操作的基础。

## 文档

MongoDB 是一个 NoSQL 数据库管理系统（DBMS）。这意味着它摒弃了 SQL 导向系统（如 MySQL、Oracle 和 Microsoft SQL Server）使用的传统基于表的数据存储模型。相反，它将数据存储为文档，这些数据结构几乎与标准的 JSON 对象完全相同。例如，MongoDB 文档可以如下所示：

```js
{
  "_id" : ObjectId("547cb6f109ce675dbffe0da5"),
  "name" : "Fleur-De-Lys Pharmacy",
  "licenseNumber" : "DL 133",
  "address" : "430, Triq Fleur-de-Lys",
  "geolocation" : {
  "lat" : 35.8938857,
  "lng" : 14.46954679999999
  },
  "postCode" : "BKR 9060",
  "localityId" : ObjectId("54c66564e11825536f510963")
}
```

这个文档代表了一家药店，包括一些基本信息，如名称、地址和国家许可证号。如果您熟悉 JSON，您会感到非常熟悉；这是标准的对象表示法。但是，请注意这里的一个不寻常的数据类型——`ObjectId`。这是 MongoDB 中的内置数据类型，它是用于唯一标识单个文档的默认方法。您在 MongoDB 数据库中存储的每个文档都保证具有相对于该数据库的唯一`_id`成员。

### 注意

如果您熟悉 SQL，您可能会倾向于将其视为列 ID。但是！`_id`在整个数据库中唯一标识一个文档，而 SQL 列 ID 仅在表中唯一标识一行。

## 集合

尽管您可以通过其`_id`唯一标识一个文档，但如果我们能以某种方式根据一些共同特征组织文档，生活将会简单得多。这就是集合概念发挥作用的地方。简单地说，集合只是存在于一个共同文件夹中的一组文档。例如，我们可以有一个名为“药店”的集合，其中存储类似我们之前示例的文档。

如果您习惯于 SQL，您可能本能地认为同一集合中的文档必须以某种方式具有相同的结构，就像 SQL 表中的行一样。令人惊讶的是，这甚至远非真实。集合只是对文档进行分组；它们不对其施加任何结构上的要求（除了需要有一个`_id`，但这适用于所有文档，与特定集合无关）。这意味着在我们存储药店相关数据的集合中，我们也可以存储描述水果、人、汽车或电影的文档。是否应该这样做完全取决于程序员。这种结构的自由度是 MongoDB 最强大的方面之一，也是它与传统 DBMS 的关键因素之一。

## 数据库

现在我们知道 MongoDB 将数据存储为集合中的文档。我们需要提到的最后一个存储概念就是数据库本身。简而言之，MongoDB 中的数据库是一个顶层的组织结构，其中包含一组集合以及有关可以访问数据库的用户、安全设置、优化和其他配置选项的信息。单个 MongoDB 实例可以管理尽可能多的数据库，只要服务器资源允许。

### 注意

很容易被误导以为 MongoDB 本身就是数据库。相反，MongoDB 是一个 DBMS，可以管理任意数量的数据库。

## 一个示例 - 产品订单数据库

让我们将迄今为止学到的东西付诸实践，并构建一个包含有关产品、客户和客户为特定产品下的订单数据的简单 MongoDB 数据库。如果您习惯于其他 DBMS，如 MySQL，您可能会惊讶地发现这个过程是多么简单和直观。

# 连接到 MongoDB

为了能够与 MongoDB 实例进行交互，我们首先需要确保我们的服务器正在运行它。然后，我们可以通过 Mongo shell 应用程序访问它。在第一章*设置您的工作空间*中，我们详细介绍了如何在特定操作系统上安装和运行 MongoDB。如果您还没有这样做，您应该按照这些步骤进行。一旦您验证了 MongoDB 正在运行，打开适用于您的操作系统的 MongoDB shell。

## Linux 和 Mac OS X

启动控制台并运行以下命令：

```js
mongo

```

## Windows

启动命令提示符并运行以下命令：

```js
C:\mongodb\bin\mongo.exe

```

您将看到一个以`>`字符开头的提示符。从这里，我们可以与 MongoDB 交互地发出命令并阅读生成的输出。

# 创建数据库

让我们首先定义我们要使用的数据库。在您的 shell 中，执行以下操作：

```js
> use OrderBase

```

这将要求 MongoDB 切换到一个名为`OrderBase`的新数据库，我们希望对其运行命令。响应将如下所示：

```js
switched to db OrderBase

```

但是，我们如何切换到尚不存在的数据库？MongoDB 的灵活性来拯救！当您告诉 MongoDB 使用数据库时，它将在切换到该数据库之前自动为您创建该数据库。

# 创建我们的集合

现在我们已经创建了一个数据库，让我们通过以下步骤填充一些集合：

1.  运行以下命令为`Products`创建一个集合：

```js
> db.createCollection('Products')

```

MongoDB 将以以下方式做出响应：

```js
{ "ok" : 1 }

```

前面的代码表明命令已成功执行。请注意，响应以 JSON 格式返回给我们。

1.  让我们停下来一分钟，分解一下前面的命令，以便理解我们刚刚做了什么：

+   `db`是一个 JavaScript 对象，表示当前选择的数据库。在我们的案例中，它是`OrderBase`。

+   `createCollection('Products')`函数是`db`的许多成员方法之一。不用说，它会创建一个新的集合并将其添加到`db`中。它的参数是一个字符串，是新集合的名称。

换句话说，使用 MongoDB 实际上是在纯 JavaScript 中发出命令的问题。不仅如此，数据本身和对命令的响应都被编码为 JSON！很明显，MongoDB 为 JavaScript 项目提供了完美无缝的适配。

1.  让我们也创建另外两个集合来存储我们的订单：

```js
> db.createCollection('Orders')
> db.createCollection(Customers'Customers')

```

您将得到与之前相同的**ok**响应。

1.  现在，让我们向我们的`Product`集合添加一些产品。在我们的案例中，让我们假设一个产品具有以下定义特征：

+   字符串类型的名称

+   浮点类型的价格

我们可以将其表示为一个简单的 JSON 对象，如下所示：

```js
{
  "name" : "Apple",
  "price" : 2.5
}
```

1.  将名称和价格插入`Products`集合同样简单：

```js
> db.Products.insert({"name" : "Apple", "price" : 2.5})

```

响应将如下所示：

```js
WriteResult({ "nInserted" : 1 })

```

前面的结果包含一个`WriteResult`对象，提供了针对 MongoDB 实例的写操作结果的详细信息。这个特定的`WriteResult`实例告诉我们写入成功了（因为没有返回错误），并且我们插入了一个新文档。

1.  再次，让我们仔细看一下我们刚刚发出的命令：

+   `db`仍然是我们正在操作的数据库，即`OrderBase`。

+   `Products`是我们的产品集合，属于`db`。

+   `insert()`方法属于产品集合（请注意，甚至集合都表示为带有属性和方法的普通 JavaScript 对象）。它接受一个 JSON 对象，比如我们在前面的代码中定义的对象，并将其插入到集合中作为新文档。

现在我们的集合中实际包含一个文档，我们可以要求 MongoDB 告诉我们其中包含什么。

1.  发出以下命令：

```js
> db.Products.find()

```

`find()`方法告诉 MongoDB 在关联集合的文档中查找。如果您不向其传递任何参数（空查找），它将返回集合中的所有文档。对我们来说，幸运的是，我们还没有足够的文档（尚未）导致屏幕滚动太多：

```js
{ "_id" : ObjectId("54f8f04a598e782be72d6294"),
 "name" : "Apple",
"price" : 2.5 }
```

这是我们之前插入的同一个苹果...还是吗？请注意，MongoDB 为其创建了一个`ObjectId`实例，并自动将其添加到对象成员中。这将始终执行（除非您指定手动`_id`），因为 MongoDB 数据库中的所有文档都需要具有自己独特的`_id`。

### 注意

如果您在自己的机器上运行此示例，您会很快注意到您的对象的`_id`值与此处看到的值不同，因为 ID 是在插入时随机生成的。

1.  让我们继续插入两个产品。但是，与其为它们中的每一个执行一个`insert`语句，这次我们可以通过传递我们想要插入的所有对象的 JSON 数组来执行批量插入，如下所示：

```js
> db.Products.insert([{"name" : "Pear", "price" : 3.0}, {"name" : "Orange", "price" : 3.0}])

```

响应将如下所示：

```js
BulkWriteResult({
    "writeErrors" : [ ],
    "writeConcernErrors" : [ ],
    "nInserted" : 2,
    "nUpserted" : 0,
    "nMatched" : 0,
    "nModified" : 0,
    "nRemoved" : 0,
    "upserted" : [ ]
})
```

这个响应，一个`BulkWriteResult`方法，显然比普通的`WriteResult`复杂得多。我们暂时不需要关心它的属性意味着什么。我们只需要从中读取到两个文档被写入数据库（`"nInserted" : 2`）。

1.  让我们再发出一个`find()`方法，以确保我们的数据库包含我们期望的内容：

```js
{ "_id" : ObjectId("54f8f04a598e782be72d6294"), "name" : "Apple", "price" : 2.5 }
{ "_id" : ObjectId("54f8f6b8598e782be72d6295"), "name" : "Pear", "price" : 3 }
{ "_id" : ObjectId("54f8f6b8598e782be72d6296"), "name" : "Orange", "price" : 3 }

```

1.  现在，让我们通过添加一些客户来结束。我们稍后会添加我们的订单：

```js
> db.Customers.insert(
[
{"firstName" : "Jane", "lastName" : "Doley"},
{"firstName" : "John", "lastName" : "Doley"}
])

```

1.  最后，通过执行以下命令验证我们现在有客户可以使用：

```js
> db.Customers.find()

```

响应将如下所示：

```js
{
"_id" : ObjectId("54f94003ea8d3ea069f2f652"),
"firstName" : "Jane",
"lastName" : "Doley"
},
{
"_id" : ObjectId("54f94003ea8d3ea069f2f653"),
"firstName" : "John",
"lastName" : "Doley"
}
```

# 创建文档之间的关系

我们现在知道如何在数据库的集合中创建文档。然而，在现实生活中，通常仅仅拥有独立的文档是远远不够的。我们还希望在文档之间建立某种关系。

例如，在我们的数据库中，我们存储有关客户和产品的信息，但我们还希望存储有关订单的信息，这些订单本质上是说明客户*X*已经订购产品*Y*的销售单据。

假设*Jane*想要订购一个*梨*。为了实现这一点，我们的订单可以看起来像这样：

```js
{
"customer" :
{
"firstName" : "Jane",
"lastName" : "Doley"
},
"product" :
{
"name" : "Pear",
"price" : 3
}
}
```

然而，这种方法的缺点立即变得清晰。它会导致数据膨胀，因为同一个客户或产品可能出现在多个订单中。因此，它的数据将需要在每个订单中重复。这也使得维护变得一团糟。如果我们想要更新，比如说，产品的价格，我们需要在数据库中搜索出该产品出现的每一个实例并进行更改。

作为 MongoDB 开发人员建议的更好的方法是使用手动引用。在这种方法中，我们只存储我们希望引用的文档的`_id`，而不是完整的文档。

### 注意

MongoDB 内置了替代方法，但通常它们处理边缘情况并不适用于一般用途。在本书中，我们只会使用这里描述的方法。

然后让访问数据库的应用程序检索所需的其他文档的信息。回到我们的订单示例，这意味着最终订单文档将如下所示：

```js
{
"customerId" : ObjectId("54f94003ea8d3ea069f2f652")
"productId" : ObjectId("54f8f6b8598e782be72d6295")
}
```

请注意，在前面的代码中，我们在属性名称后面添加了`Id`。这是处理对其他文档的引用时的正常约定，强烈建议你遵循它。

正如我们现在已经习惯的那样，插入这个新文档并不比以下更难：

```js
db.Orders.insert({
"customerId" : ObjectId("54f94003ea8d3ea069f2f652"),
"productId" : ObjectId("54f8f6b8598e782be72d6295")
})
```

然后我们可以运行`db.Orders.find()`来确保一切都如预期那样：

```js
{
"_id" : ObjectId("54f976ccea8d3ea069f2f654"),
"customerId" : ObjectId("54f94003ea8d3ea069f2f652"),
"productId" : ObjectId("54f8f6b8598e782be72d6295")
}
```

重要的是要注意，即使我们的订单除了将两个其他文档联系在一起之外没有其他目的，它仍然有自己独特的 ID。

就是这样！我们现在已经构建了一个简单的数据库，用于存储有关客户、产品和订单的信息。接下来，我们将学习如何查询它以检索数据。

# 查询 MongoDB

我们现在熟悉了 MongoDB 中数据存储的总体结构，以及如何使用`find()`方法插入和执行一些基本的检索。在这里，我们将看一下更高级的`find()`用法，以便进行更精细的数据检索。

## 按 ID 搜索

在 MongoDB 实例上最常见的操作之一是基于 ID 进行查找。正如你可能记得的那样，数据库中的每个文档都有一个唯一的`_id`字段，MongoDB 使得使用它来查找文档变得很容易。

让我们试试吧！启动你的 Mongo shell 并再次打开`OrderBase`数据库。如果你在上一个示例之后关闭了它，你可以通过发出以下命令重新打开数据库：

```js
> use OrderBase

```

一旦数据库被选择，假设我们想要按 ID 查找给定的产品。我们随机从之前的示例中选择一个 ID，看看我们最终得到了什么。请记住，ID 在你自己的机器上会有所不同。因此，你需要选择与你自己对象相关联的 ID：

```js
> db.Products.find(
{
_id: ObjectId("54f8f6b8598e782be72d6295")
})
```

我们将得到的示例响应如下：

```js
{ "_id" : ObjectId("54f8f6b8598e782be72d6295"), "name" : "Pear", "price" : 1.5 }
```

看起来很像我们的梨！现在，让我们回溯一下，看看查找是如何工作的。请注意，我们基本上做的事情与我们想要查看所有可用的`Products`时所做的事情相同：

```js
db.Products.find()
```

然而，这次我们通过向`find()`传递参数来限定我们要查找的内容。由于我们现在已经习惯了这个过程，参数像大多数 MongoDB 中的东西一样，只是 JSON 格式：

```js
{ _id: ObjectId("54f8f6b8598e782be72d6295") }
```

通过这个参数，我们告诉 MongoDB 我们想要找到`Products`集合中所有`_id`属性等于我们 JSON 参数中相应值的文档，这种情况下是`ObjectId("54f8f6b8598e782be72d6295")`。

请注意，`find()`方法可以返回多个结果。当搜索 ID 时，这没有多大意义，因为只有一个 ID 可以属于任何给定的文档，因此最多只能有一个结果。为了适应这种情况，MongoDB 为集合提供了另一种方法——`findOne()`。它的工作方式与 find()相同，唯一的例外是它最多返回一个结果，如下所示：

```js
> db.Products.findOne({
_id: ObjectId("54f8f6b8598e782be72d6295")
})
```

## 按属性值搜索

我们已经看到了通过 ID 查找文档有多么容易，毫无疑问，通过一般属性值进行搜索同样简单。例如，假设我们想找到所有名称为`Orange`的产品。我们可以这样做：

```js
db.Products.find({"name" : "Orange"})
```

MongoDB 将返回以下结果：

```js
{ "_id" : ObjectId("54f8f6b8598e782be72d6296"), "name" : "Orange", "price" : 3 }
```

在某些情况下，集合中的几个文档将具有我们要搜索的属性的相同值。在这种情况下，MongoDB 将返回所有匹配的文档。以下是一个例子：

```js
db.Products.find({"price" : 3.0})
```

这将返回所有价格为`3.0`的产品。在我们的情况下，它将返回以下结果：

```js
{ "_id" : ObjectId("54f8f6b8598e782be72d6296"), "name" : "Orange", "price" : 3 },
{ "_id" : ObjectId("54f9b82caf8e5041d9e0af09"), "name" : "Pear", "price" : 3 }
```

# 高级查询

我们在这里所涵盖的内容仅仅是你可以通过`find()`做的一切的冰山一角，但这已经足够让我们能够配置一个基本的持久层。在本书的其余部分，随着需要的出现，我们将逐步引入更多高级的查询。

# 连接 MongoDB 和 Node.js

我们现在对 MongoDB 的最基本概念有了扎实的理解，是时候将它们与 Node.js 集成起来，让它们发挥作用了。在本节中，我们将逐步介绍这个过程，并看看我们如何可以直接从正在运行的 Node.js 实例中与 MongoDB 数据库进行交互。

## 建立一个基本项目

在开始之前，请确保你的系统上已经安装了 Node.js 和**Node Package Manager**（**NPM**）。参考第一章，*设置你的工作区*，了解你特定操作系统的步骤。

一旦准备就绪，首先创建一个基本项目，以便对 MongoDB 实例进行一些实验。在某个地方创建一个名为 MongoNode 的文件夹。接下来，打开一个终端（OS X/Linux）或命令提示符（Windows），导航到这个文件夹，并输入以下命令：

```js
npm init

```

这将启动一个用于引导基本 Node.js 应用程序的交互式向导。在下面的代码中，我们提供了一些标准答案，以回答向导将要问的问题：

```js
name: (MongoNode)
version: (0.0.0)
description: Simple project demonstrating how to interface with a MongoDB instance from Node.js
entry point: (index.js)
test command:
git repository:
keywords:
author: Yours Truly
license: (BSD)
About to write to /home/user/IdeaProjects/nodebook-ch2/MongoNode/package.json:

{
  "name": "MongoNode",
  "version": "0.0.0",
  "description": "Simple project demonstrating how to interface with a MongoDB instance   from Node.js",
  "main": "index.js",
  "scripts": {
  "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "Yours Truly",
  "license": "BSD"
}

Is this ok? (yes)
```

一旦引导完成，创建一个名为`index.js`的新文件。在你喜欢的文本编辑器中打开它，并输入以下内容：

```js
console.log('Hello World!');

```

保存文件，然后打开一个终端。进入我们刚创建的文件夹，并运行以下命令：

```js
node index.js

```

你应该会得到以下熟悉的输出：

**你好，世界！**

现在我们可以确信 Node.js 按预期工作了。所以，让我们继续看看如何与之前构建的数据库联系起来。

## 连接到 MongoDB

现在，让我们设置底层接口，与 MongoDB 实例进行交互。我们需要做的第一件事是安装官方的 MongoDB 驱动程序，用于 Node.js。在项目文件夹中，在终端中输入以下命令：

```js
npm install mongodb -save

```

这将使`npm`执行完整的安装过程。一旦完成，我们将拥有所有需要与 MongoDB 实例进行交互的基本功能。

安装完成后，创建一个名为`database.js`的新文件，在文本编辑器中打开它，并插入以下内容。如果与我们迄今为止看到的相比，这是相当多的代码，不要担心；我添加了相当多的注释来解释正在发生的事情：

```js
// Our primary interface for the MongoDB instance
var MongoClient = require('mongodb').MongoClient;

// Used in order to verify correct return values
var assert = require('assert');

/**
*
* @param databaseName - name of the database we are connecting to
* @param callBack - callback to execute when connection finishes
*/
var connect = function (databaseName, callback) {

  // URL to the MongoDB instance we are connecting to
  var url = 'mongodb://localhost:27017/' + databaseName;

  // Connect to our MongoDB instance, retrieve the selected
  // database, and execute a callback on it.
  MongoClient.connect(url, function (error, database) {

    // Make sure that no error was thrown
    assert.equal(null, error);

    console.log("Successfully connected to MongoDB instance!");

    callback(database);
  });
};

/**
* Executes the find() method of the target collection in the
* target database, optionally with a query.
* @param databaseName - name of the database
* @param collectionName - name of the collection
* @param query - optional query parameters for find()
*/
exports.find = function (databaseName, collectionName, query) {
  connect(databaseName, function (database) {

    // The collection we want to find documents from
    var collection = database.collection(collectionName);

    // Search the given collection in the given database for
    // all documents which match the criteria, convert them to
    // an array, and finally execute a callback on them.
    collection.find(query).toArray(
      // Callback method
      function (err, documents) {

        // Make sure nothing went wrong
        assert.equal(err, null);

        // Print all the documents that we found, if any
        console.log("MongoDB returned the following documents:");
        console.dir(documents);

        // Close the database connection to free resources
        database.close();
    })
  })
};
```

接下来，在`index.js`文件中导入数据库模块。删除这个文件中的所有内容，并插入以下内容：

```js
var database = require('./database');
```

这将使我们能够像使用常规的 Node.js 模块一样使用我们的数据库接口。

最后，让我们运行一下，确保一切正常。在`index.js`文件中插入以下行：

```js
database.find('OrderBase', 'Products', {});
```

前面的命令对你来说应该立即显得很熟悉；它与我们在之前的示例中运行以下命令时相同：

```js
db.Products.find();
```

在这里，我们简单地将参数包装在逻辑中，以便在 Node.js 实例中运行。

要运行 Node.js 实例，请再次在终端中输入以下命令：

```js
node index.js
```

你应该会收到类似以下的内容：

**成功连接到 MongoDB 实例！**

MongoDB 返回了以下文档：

```js
[ { _id: 54f8f04a598e782be72d6294, name: 'Apple', price: 2.5 },
  { _id: 54f8f6b8598e782be72d6295, name: 'Pear', price: 1.5 },
  { _id: 54f8f6b8598e782be72d6296, name: 'Orange', price: 3 },
  { _id: 54f9b82caf8e5041d9e0af09, name: 'Banana', price: 3 } ]
```

# 总结

就是这样！你刚刚通过使用 MongoDB 编写了你的第一个 Node.js 应用程序！这非常简单。请注意，随着我们的继续，我们将以稍微不同的方式组织我们的代码，但现在，你已经掌握了使其工作所需的基本知识。接下来，我们将学习一些高级主题，并看看如何使用 Node.js 和 MongoDB 来构建一个完整的 API。

通过本章，我们为您提供了开始为 Ionic 移动应用构建自己的数据库所需的基本知识，这是构建跨平台移动应用后端的第一步。

随着我们继续前进，您将意识到通过本章学到的基本知识将为我们提供开始构建自己的 API 所需的必要知识，这将在下一章中完成。
