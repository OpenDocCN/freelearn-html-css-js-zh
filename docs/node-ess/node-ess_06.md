# 第六章：Level DB 和 NoSQL

在本章中，我们将介绍两种可以与 Node.js 一起使用的数据库变体；一种提供了非常轻量级和简单的功能集，而另一种则为我们提供了更灵活和通用的功能集。在本章中，我们将介绍 LevelDB 和 MongoDB

# Level DB

Node.js 的一个很棒的地方是我们在前端和后端都使用相同的语言，NoSQL 数据库也是如此。它们中的大多数从一开始就支持 JSON；这对于使用 Node.js 的任何人来说都很棒，因为不需要花时间制作关系模型，将其转换为类似 JSON 的结构，将其传递到浏览器，对其进行操作，然后再反转这个过程。

使用原生支持 JSON 的数据库，您可以立即开始工作并投入使用。

Google 为我们提供了一个简单的入口到一个可以安装并准备使用的 NoSQL 数据库，只需一个命令即可：

```js
[~/examples/example-18]$ npm install level

```

您将看到这将安装`LevelDOWN`和`LevelUP`。

`LevelDOWN`是`LevelDB`的低级绑定，`LevelUP`是对其的简单封装。

`LevelDB`在设置方面非常简单。一旦安装完成，我们只需创建一个`LevelUP`实例，并将其传递到我们希望存储数据库的位置：

```js
var LevelUP = require( 'level' ),
    db = new LevelUP( './example-db');
```

现在我们有了一种快速简单的存储数据的方法。

由于`LevelDB`只是一个简单的键/值存储，它默认使用字符串键和字符串值。如果这是您希望存储的所有信息，这是很有用的。您还可以将其用作简单的缓存存储。它有一个非常简单的 API，此阶段我们只关注四种方法：`put`、`get`、`del`和`createReadStream`；大多数方法的作用都很明显：

| 方法 | 用途 | 参数 |
| --- | --- | --- |
| put | 插入键值对 | 键，值，回调函数（错误） |
| get | 获取键值对 | 键，回调函数（错误，值） |
| del | 删除键值对 | 键，回调函数（错误） |
| createReadStream | 获取多个键值对 |   |

一旦我们创建了数据库，要插入数据，我们只需要做以下操作：

```js
db.put( 'key', 'value', function( error ) {
    if ( error ) return console.log( 'Error!', error )

    db.get( 'key', function( error, value ) {
        if ( error ) return console.log( 'Error!', error )

        console.log( "key =", value )
    });
});
```

如果运行代码，我们将看到我们插入并检索到了我们的值：

```js
[~/examples/example-18]$ node index.js
key = value

```

这不是我们简单的 JSON 结构；但是，它只是一个字符串。要使我们的存储保存 JSON，我们只需要将值编码作为选项传递给数据库，如下所示：

```js
var LevelUP = require( 'level' ),
    db = new LevelUP( './example-db', {
        valueEncoding: 'json'
    });
```

现在我们可以存储 JSON 数据：

```js
db.put( 'jsonKey', { inner: 'value' }, function ( error ) {
    if ( error ) return console.log( 'Error!', error )

    db.get( 'jsonKey', function( error, value ) {
        if ( error ) return console.log( 'Error!', error )

        console.log( "jsonKey =", value )
    });
});
```

然而，字符串可以存储为 JSON，我们仍然可以将字符串作为值传递，并且也可以检索它。

运行此示例将显示以下内容：

```js
[~/examples/example-18]$ node index.js
key = value
jsonKey = { inner: 'value' }

```

现在，我们已经掌握了简单的方法，现在我们可以继续使用`createReadStream`。

此函数返回一个对象，可以与 Node.js 内置的`ReadableStream`进行比较。对于数据库中的每个键/值对，它将发出一个`data`事件；它还会发出其他事件，如`error`和`end`。如果`error`没有事件监听器，那么它将传播，从而终止整个进程（或域），如下所示：

```js
db.put( 'key1', { inner: 'value' }, function( error ) {
    if ( error ) return console.log( 'Error!', error )

    var stream = db.createReadStream( );

    stream
    .on( 'data', function( pair ) {
        console.log( pair.key, "=", pair.value );
    })
    .on( 'error', function( error ) {
        console.log( error );
    })
    .on( 'end', function( ) {
        console.log( 'end' );
    });
});
```

运行此示例：

```js
[~/examples/example-20]$ node index.js
key1 = { inner: 'value' }
end

```

如果我们在数据库中放入更多数据，将会发出多个`data`事件：

```js
[~/examples/example-20]$ node index.js
key1 = { inner: 'value' }
key2 = { inner: 'value' }
end

```

# MongoDB

正如您所看到的，使用 Node.js 的数据库可以非常简单。如果我们想要更完整的东西，我们可以使用另一个名为**MongoDB**的 NoSQL 数据库——另一个非常受欢迎的基于文档的数据库。

对于这组示例，您可以使用托管数据库，使用提供者如 MongoLab（他们提供免费的开发层级），或者您可以按照[`docs.mongodb.org/manual/installation`](http://docs.mongodb.org/manual/installation)上的说明在本地设置数据库。

一旦您有一个要连接的数据库，我们就可以继续。

MongoDB 有几个可以与 Node.js 一起使用的模块，最受欢迎的是 Mongoose；但是，我们将使用核心的 MongoDB 模块：

```js
[~/examples/example-21]$ npm install mongodb

```

要使用我们的数据库，我们首先需要连接到它。我们需要为客户端提供一个连接字符串，一个带有`mongodb`协议的通用 URI。

如果您有一个本地的 mongo 数据库在没有凭据的情况下运行，您将使用：

```js
mongodb://localhost:27017/database
```

默认端口是`27017`，所以你不需要指定它；但是为了完整起见，它已经包含在内。

如果你正在使用 MongoLab，他们会提供给你一个连接字符串；它应该是这种格式：

```js
mongodb://<dbuser>:<dbpassword>@<ds>.mongolab.com:<port>/<db>

```

连接到我们的数据库实际上非常简单。我们只需要提供驱动程序一个连接字符串，然后我们就可以得到一个数据库：

```js
var MongoDB = require('mongodb'),
    MongoClient = MongoDB.MongoClient;

connection = "mongodb://localhost:27017/database"

MongoClient.connect( connection, function( error, db ) {
    if( error ) return console.log( error );

    console.log( 'We have a connection!' );
});
```

MongoDB 中的每组数据都存储在一个集合中。一旦我们有了数据库，我们就可以获取一个集合来运行操作：

```js
var collection = db.collection( 'collection_name' );
```

在一个集合中，我们有一些简单的方法，拥有很大的力量，为我们提供了一个完整的 CRUD“API”。

MongoDB 中的每个文档都有一个 ID，它是`ObjectId`的一个实例。他们用于此 ID 的属性是`_id`。

要保存一个文档，我们只需要调用`save`，它接受一个对象或对象数组。集合中的单个对象称为文档：

```js
var doc = {
    key: 'value_1'  
};
collection.save( doc, { w: 1 }, function( ) {
    console.log( 'Document saved' )
});
```

如果我们使用带有 ID 的文档调用`save`函数，那么该文档将被更新而不是插入：

```js
var ObjectId = MongoDB.ObjectId
// This document already exists in my database
var doc_id = {
    _id: new ObjectId( "55b4b1ffa31f48c6fa33a62a" ),
    key: 'value_2'
};
collection.save( doc_id, { w: 1 }, function( ) {
    console.log( 'Document with ID saved' );
});
```

现在我们在数据库中有了文档，我们可以查询它们，如下所示：

```js
collection.find( ).toArray( function( error, result ) {
    console.log( result.length + " documents in our database!" )
});
```

如果`find`没有提供回调函数，它将返回一个游标；这使我们能够使用`limit`、`sort`和`toArray`等方法。

你可以向`find`传递一个查询来限制返回的内容。为了通过其 ID 查找对象，我们需要使用类似于以下的东西：

```js
collection.find(
    { _id: new ObjectId( "55b4b1ffa31f48c6fa33a62a" ) },
    function( error, documents ) {
        console.log( 'Found document', documents[ 0 ] );
    }
);
```

我们还可以通过任何其他可能使用的属性进行过滤：

```js
collection.find(
    { key: 'value' },
    function( error, documents ) {
        console.log( 'Found', documents.length, 'documents' );  
    }
);
```

如果你以前使用过 SQL，你一定会注意到缺少操作符，比如`OR`、`AND`或`NOT`。但是，你不需要担心，因为 mongo 提供了许多等价物。

你可以在这里看到完整的列表：[`docs.mongodb.org/manual/reference/operator/query/`](http://docs.mongodb.org/manual/reference/operator/query/)。

所有操作符都以美元符号开头，例如`$and`、`$or`、`$gt`和`$lt`。

你可以查看文档以查看使用这些的具体语法。

要使用`$or`条件，你需要将其包含在其中，就好像它是一个属性一样：

```js
collection.find(
    {
        $or: [
            { key: 'value' },
            { key: 'value_2' }
        ]
    },
    function( error, documents ) {
        console.log( 'Found', documents.length, 'documents' );  
    }
);
```

使用诸如 MongoDB 这样的数据库使我们能够更有力地检索数据并创建更具功能的软件。

# 摘要

现在我们有可以存储数据的地方。一方面，我们有一个简单的键/值存储，为我们提供了一种非常方便的存储数据的方式；另一方面，我们有一个功能丰富的数据库，为我们提供了一整套查询操作符。

这两个数据库将在接下来的章节中帮助我们，因为我们将更接近创建我们的全栈应用程序。

在下一章中，我们将介绍`Socket.IO`，这是一个建立在 WebSockets 之上的实时通信框架。

为 Bentham Chang 准备，Safari ID 为 bentham@gmail.com 用户编号：2843974 © 2015 Safari Books Online，LLC。此下载文件仅供个人使用，并受到服务条款的约束。任何其他使用都需要版权所有者的事先书面同意。未经授权的使用、复制和/或分发严格禁止并违反适用法律。保留所有权利。
