# 第五章：使用 MongoDB 的 JSON

在本章中，我们将介绍以下食谱：

+   设置 MongoDB

+   为 Node.js 安装 MongoDB 数据库驱动程序

+   为 Node.js 安装 express 模块

+   使用 Node.js 连接 MongoDB 数据库

+   使用 Node.js 在 MongoDB 中创建文档

+   使用 Node.js 在 MongoDB 中搜索文档

+   使用 Node.js 在 MongoDB 中更新文档

+   使用 Node.js 在 MongoDB 中删除文档

+   使用 REST 搜索 MongoDB

+   使用 REST 在 MongoDB 中创建文档

+   使用 REST 更新 MongoDB 中的文档

+   使用 REST 在 MongoDB 中删除文档

# 简介

在本章中，我们将介绍如何使用 MongoDB 作为 Web 应用程序的后端存储。虽然不是完全专注于 JSON，但正如你所见，本章的食谱将帮助你管理使用 MongoDB 在 Node.js 中直接创建、读取、更新和删除文档，然后使用为 Node.js 和 MongoDB 构建的 REST 服务器，这样你就可以从网络客户端（如 Web 应用程序）管理文档。

# 设置 MongoDB

安装 MongoDB 取决于平台；在 Linux 上，你可能可以使用像 apt 这样的包安装器，而在 Windows 和 Mac OS X（以及如果你有没有包含 MongoDB 包的包管理器的 Linux 发行版）上，可以使用网页下载。

## 如何做到…

1.  在 Mac OS X 和 Windows 上，只需前往[`www.mongodb.org/`](http://www.mongodb.org/)并点击下载链接。在撰写本文时，MongoDB 处于 2.6.7 版本；有一个 3.0 版本的候选发布，我们在这里不再进一步讨论。

    Mongo 还提供了针对几种常见 Linux 发行版的包，包括 Debian 和 Fedora。还有一个适用于 FreeBSD 的包。

1.  一旦你下载并安装了 Mongo，你需要为 MongoDB 提供一个存储数据库的地方。

    这取决于平台；在 Windows 上，它是`c:\data\db.`

1.  一旦你这样做，你可以通过运行`mongod`来启动数据库服务器。你可能还想将 MongoDB 客户端和服务器二进制文件的路径添加到你的路径中，这样你就可以从命令行轻松访问它们。

1.  当你运行 MongoDB 服务器时，你应该会看到一堆类似于这样的日志消息：

    ```js
    C:\Program Files\MongoDB 2.6 Standard\bin\mongod.exe --help for help and startup options
    2015-02-15T13:10:07.909-0800 [initandlisten] MongoDB starting : pid=13436 port=27017 dbpath=\data\db\ 64-bit host=KF6GPE-SURFACE
    2015-02-15T13:10:07.911-0800 [initandlisten] targetMinOS: Windows 7/Windows Server 2008 R2
    2015-02-15T13:10:07.913-0800 [initandlisten] db version v2.6.7
    2015-02-15T13:10:07.914-0800 [initandlisten] git version: a7d57ad27c382de82e9cb93bf983a80fd9ac9899
    2015-02-15T13:10:07.915-0800 [initandlisten] build info: windows sys.getwindowsversion(major=6, minor=1, build=7601, pla
    tform=2, service_pack='Service Pack 1') BOOST_LIB_VERSION=1_49
    2015-02-15T13:10:07.917-0800 [initandlisten] allocator: system
    2015-02-15T13:10:07.920-0800 [initandlisten] options: {}
    2015-02-15T13:10:07.930-0800 [initandlisten] journal dir=\data\db\journal
    2015-02-15T13:10:07.931-0800 [initandlisten] recover : no journal files present, no recovery needed
    2015-02-15T13:10:07.967-0800 [initandlisten] waiting for connections on port 27017
    ```

    你可能会注意到服务器正在运行的主机名（在这个例子中，`KF6GPE-SURFACE`）和端口号，默认应该是`27017`。

1.  要直接连接到 MongoDB 服务器，你可以在命令行上运行`mongo`，像这样：

    ```js
    C:\>mongo
    MongoDB shell version: 2.6.7
    connecting to: test
    >
    ```

1.  要退出`mongo`二进制文件，请按*Ctrl* + *C*或输入`exit`。

## 它是如何工作的…

双击可执行安装程序和 Linux 包将安装 mongod 二进制文件，即数据库，以及 Mongo 命令行客户端。

# 安装 MongoDB 数据库驱动程序（重复）

你需要为 Node.js 安装数据库驱动程序，这样 Node.js 就可以直接与 MongoDB 服务器通信。

## 如何做到…

要获取数据库驱动程序，只需前往你拥有 Node.js 文件的项目的目录，并运行以下命令：

```js
npm install mongodb

```

这个命令将下载数据库驱动程序并为 Node.js 安装它们。

# 为 Node.js 安装 express 模块

Node.js 的 express 模块使得使用 Node.js 构建表示状态转移（REST）服务器应用程序变得容易。REST 是一种在网络编程中使用的强大范式，它使用 HTTP 方法`GET`、`POST`、`PUT`和`DELETE`来管理 Web 服务的文档管理的创建、读取、更新和删除（通常缩写为 CRUD）操作。

使用 REST，URL 是表示你想要操纵什么的名词，HTTP 方法是动词，对那些名词执行动作。

在接下来的食谱中，我们将使用 Node.js 的 express 模块构建一个 RESTful 服务器，该服务器从 Mongo 返回文档，并支持基本的 CRUD 操作。在开始之前，你需要安装三个额外的模块。

## 如何去做…

你将使用`npm`，Node.js 的包管理器，来安装跨对象资源模块以支持跨域脚本，express 模块，以及 express 使用的 body-parser 模块。为此，在你的项目目录中运行以下命令：

```js
npm install cors
npm install express
npm install body-parser

```

你还需要一个基本的应用程序，或者骨架，用于你的 REST 服务器，它包括 REST 服务器之间的 URL 路由、HTTP 方法以及执行必要数据库操作的函数。这个骨架包括使用 express 模块的两个 Node.js 脚本和一个 HTML 文档。

第一个 Node.js 脚本是 REST 服务器本身，位于`rest-server.js`中，它看起来像这样：

```js
var express = require('express'),
  documents = require('./routes/documents'),
  cors = require('cors'),
  bodyParser = require('body-parser');

var app = express();

app.use(cors());
var jsonParser = bodyParser.json();

app.get('/documents', documents.findAll);
app.get('/documents/:id', documents.findById);
app.post('/documents', jsonParser, documents.addDocuments);
app.put('/documents/:id', jsonParser, documents.updateDocuments);
app.delete('/documents/:id', jsonParser, 
documents.deleteDocuments);

app.listen(3000);
console.log('Listening on port 3000...');
```

## 它是如何工作的…

包管理器安装每个模块，如有需要，从源代码构建它们。你需要所有三个模块：CORS 模块以支持跨域脚本请求、express 模块用于 REST 服务器框架，最后，body-parser 模块将客户端对象体从 JSON 转换为 JavaScript 对象。

骨架脚本包括 express 模块、我们的*路由*文件，它将定义处理每个 REST 用例的函数、CORS 模块以及 express 需要的 body-parser 模块来解释客户端发送的对象体。

一旦包含这些，它定义了一个名为`app`的 express 模块实例，并用 CORS 对其进行配置。这是必要的，因为默认情况下，浏览器不会对页面的内容来源不同的域名服务器发起 AJAX 请求，以防止服务器被攻陷并注入恶意 JavaScript 的跨站脚本攻击。CORS 模块为服务器设置必要的头，以便让我们可以使用上一章中的旧 Node.js 服务器在端口`1337`上提供内容，并让我们的内容访问在此不同端口上运行的 REST 服务器。

接下来，我们获取一个对 body-parser 的 JSON 解析器的引用，我们将用它来解析客户端为插入和更新请求发送的对象体。之后，我们用处理顶级文档 URL 的手动器配置 Express 应用服务器实例，该 URL 用于通过 REST 访问我们的 MongoDB 文档。在这个 URL 上有五种可能的操作：

+   对 URL `/documents`的 HTTP GET simply returns a list of all the documents in the database

+   对 URL `/documents/<id>`的 HTTP GET 返回具有给定 ID 的数据库中的文档

+   对`/documents`的 HTTP POST，带有 JSON 格式的文档，将该文档保存到数据库中

+   对`/documents/<id>`的 HTTP PUT，带有 JSON 格式的文档，更新具有给定 ID 的文档，使其包含客户端传递的内容

+   对`/documents/<id>`的 HTTP DELETE 删除具有给定 ID 的文档

最后，脚本在端口`3000`上启动服务器，并记录服务器已启动的事实。

当然，我们需要在文档对象中定义函数；我们是在文件`routes/documents.js`中完成的，该文件最初应看起来像这样：

```js
var mongo = require('mongodb');

var mongoServer = mongo.Server,
    database = mongo.Db,
    objectId = require('mongodb').ObjectID;

var server = new mongoServer('localhost', 27017, 
{auto_reconnect: true});
var db = new database('test', server);

db.open(function(err, db) {
  if(!err) {
    console.log("Connected to 'test' database");
    db.collection('documents', 
    {strict:true}, 
    function(err, collection) {
      if (err) {
        console.log("Inserting sample data...");
        populate();
      }
    });
  }
});

exports.findById = function(req, res) {
  res.send('');
};

exports.findAll = function(req, res) {
  res.send('');
};

exports.addDocuments = function(req, res) {
  res.send('');
};

exports.updateDocuments = function(req, res) {
  res.send('');
};

exports.deleteDocuments = function(req, res) {
  res.send('');
};

var populate = function() {
var documents = [
  {
    call: 'kf6gpe',
    lat: 37,
    lng: -122  }
];
db.collection('documents', function(err, collection) {
  collection.insert(wines, {safe:true}, 
  function(err, result) {});
  });
};
```

上述代码首先通过导入本地 MongoDB 驱动程序开始，设置变量以保存服务器实例、数据库实例和一个转换器接口，该接口将字符串转换为 MongoDB 对象 ID。接下来，它创建一个服务器实例，连接到我们的服务器实例（必须运行才能成功），并获得对我们数据库的引用。最后，它打开到数据库的连接，如果数据库为空，则在数据库中插入一些示例数据。（这个代码在阅读本章的前两个食谱后会更清晰，所以如果现在有些困惑，只需继续阅读，您会做得很好的！）

`routes/documents.js`文件的其余部分定义了处理我们在这`rest-server.js`脚本中连接的每个 REST 用例的函数。我们将在食谱中逐步完善每个函数。

最后，我们需要一个 HTML 文档来访问 REST 服务器。我们的文档看起来像这样：

```js
<!DOCTYPE html>
<html>
<head>
<script type="text/javascript"
  src="img/jquery-1.11.2.min.js"></script>
</head>
<body>

<p>Hello world</p>
<p>
<div id="debug"></div>
</p>
<p>
<div id="json"></div>
</p>
<p>
<div id="result"></div>
</p>

<button type="button" id="get" onclick="doGet()">Get</button><br/>
<form>
  Id: <input type="text" id="id"/>
  Call: <input type="text" id="call"/>
  Lat: <input type="text" id="lat"/>
  Lng: <input type="text" id="lng"/>
<button type="button" id="insert" 
    onClick="doUpsert('insert')">Insert</button>
<button type="button" id="update" 
onClick="doUpsert('update')">Update</button>
<button type="button" id="remove" 
onClick="doRemove()">Remove</button>
</form>
</body>
</html>
```

我们在脚本中使用一些 jQuery 来使字段访问更加容易（您将在即将到来的 REST 插入、更新、删除和查询食谱中看到脚本）。HTML 本身由三个`div`标签组成，分别用于调试、显示原始 JSON 和每个 REST 操作的结果，以及一个表单，让您输入创建、更新或删除记录所需的字段。

## 也见

关于卓越的 Node.js express 模块的更多信息，请参见[`expressjs.com/`](http://expressjs.com/)。

MongoDB 是一个强大的文档数据库，这里涵盖的内容远远不够。更多信息，请上网搜索，或查看 PacktPub 网站上的以下资源：

+   *Instant MongoDB* by *Amol Nayak*。

+   *MongoDB Cookbook* by *Amol Nayak*。

# 使用 Node.js 连接到 MongoDB 数据库

在你 Node.js 应用程序能够与 MongoDB 实例做任何事情之前，它必须通过网络连接到它。

## 如何做到这一点...

MongoDB 的 Node.js 驱动包含了所有必要的网络代码，用于与本地或远程机器上运行的 MongoDB 建立和断开连接。

你需要在代码中包含对原生驱动的引用，并指定要连接的数据库的 URL。

下面是一个简单的例子，它连接到数据库然后立即断开连接：

```js
var mongo = require('mongodb').MongoClient;

var url = 'mongodb://localhost:27017/test';

mongo.connect(url, function(error, db) {
  console.log("mongo.connect returned " + error);
  db.close();
});
```

让我们逐行分解这个问题。

## 它是如何工作的…

第一行包括了 Node.js 应用程序中 Mongo 的本地驱动实现，并提取了它定义的`MongoClient`对象的引用。这个对象包含了与数据库通过网络交互所需的基本接口，定义了`connect`和`close`方法。

下一行定义了一个字符串`url`，它包含了要连接的数据库的 URL。这个 URL 的格式很简单：它以`mongodb`方案开始，以表示它是 MongoDB 服务器的 URL。接下来是主机名和端口（在这个例子中，我们连接到本地主机的默认端口，即`27017`）。最后，我们来到你想要连接的数据库的名称：在我们的例子中，是`test`。

如果你使用 MongoDB 的用户访问控制来控制对数据库的访问，你还需要指定一个用户名和密码。你这样做的方式和你对任何其他 URL 的做法一样，像这样：

```js
mongodb://user:password@host:port/database
```

当然，是否保护你的数据库取决于你的网络结构和部署；通常来说，这样做是个好主意。

我们将这个 URL 传递给 mongo 对象的`connect`方法，同时提供一个函数，当连接成功建立，或者连接失败时，MongoDB 原生驱动会回调这个函数。驱动会以两个参数调用回调函数：第一个是出现错误时的错误代码（成功时为`null`），第二个是一个包含对你指定的数据库连接的数据库对象引用（如果建立连接时出现错误，则可能为`null`）。

我们的回调函数非常直接；它打印一个包含传递给它的错误代码值的消息，然后我们使用`close`断开与数据库的连接。

### 提示

当你使用完数据库对象时，总是调用其`close`方法，以确保原生驱动能够成功清理自身并从数据库断开连接。如果你不这么做，你可能会导致数据库连接泄露。

## 参见 also

关于为 Node.js 设计的 MongoDB 原生驱动的更多信息，请参阅[`docs.mongodb.org/ecosystem/drivers/node-js/`](http://docs.mongodb.org/ecosystem/drivers/node-js/)。

# 使用 Node.js 在 MongoDB 中创建文档

MongoDB 数据库通过*集合*来组织其文档，这些集合通常是相关联的一组文档（例如表示相同种类信息的文档）。由于这个原因，您与文档交互的主要界面是通过一个集合。让我们看看如何获取一个集合并向其中添加一个文档。

### 提示

集合在关系型数据库中类似于一个表，但并没有规定集合中的所有文档必须具有相同的字段或每个字段相同的类型。可以将其视为一个用于分组类似文档的抽象概念。

## 怎么做...

以下是一个函数，它使用 Node.js 在我们的测试数据库中名为`documents`的集合中插入两个静态条目：

```js
var mongo = require('mongodb').MongoClient;

var url = 'mongodb://localhost:27017/test';

var insert = function(collection, callback) {
  var documents = 
    [{ 
        call: 'kf6gpe-7', lat: 37.0, lng: -122.0 
      },
      {
        call: 'kf6gpe-9', lat: 38.0, lng: -123.0
      }];
  // Insert some documents
  collection.insert(documents, 
    function(error, result) {
      console.log('Inserted ' +result.length + ' documents ' + 
        'with result: ');
      console.log(result);
      callback(result);
  });
};

mongo.connect(url, function(error, db) {
  console.log('mongo.connect returned ' + error);

  // Get the documents collection
  var collection = db.collection('documents');
  insert(collection, function(result) {
    db.close();
  });
});
```

我把代码分成两部分，以便使回调结构更清晰：实际执行插入的`insert`函数和连接回调，该回调调用插入函数。

让我们仔细看看。

## 它是如何工作的...

代码的开始方式是一样的，通过获取一个对`MongoClient`对象的引用，它用这个对象与数据库通信。连接代码基本上也是一样的；URL 是一样的，唯一的改变是对数据库的`collection`方法的调用，传递我们感兴趣的集合的名称。`collection`方法返回一个`collection`对象，该对象提供了我们对文档集合执行 CRUD 操作的方法。

`insert`函数做几件事情。它接收一个您想要操作的集合和一个回调函数，当插入操作完成或失败时，它将调用这个回调函数。

首先，它定义了要在数据库中插入的一对静态条目。请注意，这些只是普通的旧 JavaScript 对象；基本上，任何您可以表示为 JavaScript 对象的东西，您都可以存储在 MongoDB 中。接下来，它调用集合的`insert`方法，传递要存储的对象和一个回调函数，驱动程序在尝试插入后调用该函数。

驱动程序再次调用回调函数，传递一个错误值（在成功时为`null`）和作为它们被插入到集合中的 JavaScript 对象。我们的回调函数将结果日志记录到控制台，并调用回调插入函数的回调，关闭数据库。

插入的记录看起来是什么样子呢？以下是从我的控制台获取的示例，确保我们正在运行 MongoDB：

```js
PS C:\Users\rarischp\Documents\Node.js\mongodb> node .\example.js
mongo.connect returned null
Inserted 2 documents with result:
[ { call: 'kf6gpe-7',
    lat: 37,
    lng: -122,
    _id: 54e2a0d0d00e5d240f22e0c0 },
  { call: 'kf6gpe-9',
    lat: 38,
    lng: -123,
    _id: 54e2a0d0d00e5d240f22e0c1 } ]
```

请注意，这些对象有相同的字段，但它们还有一个额外的`_id`字段，这是对象在数据库中的唯一标识符。在下一节中，您将学习如何针对该字段进行查询。

## 还有更多内容

如果你多次将同一个对象插入数据库，会发生什么？试试看！你会发现数据库中有该对象的多个副本；字段不用于指定唯一性（例外是`_id`字段，它在整个数据库中是唯一的）。注意你不能自己指定一个`_id`字段，除非您确信它是唯一的。要更新现有元素，请使用更新方法，我在本章的*使用 Node.js 在 MongoDB 中更新文档*菜谱中描述了该方法。

默认情况下，MongoDB 的插入操作很快，可能会失败（比如说，如果网络存在临时问题，或者服务器暂时过载）。为了保证安全，你可以将`{ safe: true }`作为插入操作的第二个参数，或者等待操作成功，或者在操作失败时返回一个错误。

## 也见

参考[`docs.mongodb.org/manual/reference/method/db.collection.insert/`](http://docs.mongodb.org/manual/reference/method/db.collection.insert/)获取有关如何将文档插入 MongoDB 集合的文档。

# 使用 Node.js 在 MongoDB 中搜索文档

如果你不能搜索文档，那么能够插入文档也帮助不大。MongoDB 允许你指定一个模板进行匹配，并返回匹配该模板的对象。

与插入和更新操作一样，你将处理一个文档集合，调用集合的`find`方法。

## 如何做到...

这是一个例子，它找到 test 集合中所有`kf6gpe-7`的文档，并将它们打印到控制台：

```js
var mongo = require('mongodb').MongoClient;

var url = 'mongodb://localhost:27017/test';

mongo.connect(url, function(error, db) {
  console.log("mongo.connect returned " + error);

  var cursor = collection.find({call: 'kf6gpe-7'});
  cursor.toArray(function(error, documents) {
    console.log(documents);

    db.close();
  });
});
```

## 它是如何工作的...

连接到数据库后，我们在集合中调用`find`，它返回一个游标，您可以使用它遍历找到的值。`find`方法接受一个 JavaScript 对象，作为模板指示您想要匹配的字段；我们的例子匹配名为`call`的字段等于`kf6gpe-7`的记录。

我们不是遍历游标，而是通过使用游标的`toArray`方法，将找到的所有值转换成一个单一的数组。这对于我们的例子来说是可以的，因为结果并不多，但是在具有很多项的数据库上这样做要小心！一次性从数据库中获取比你实际需要更多的数据，会使用到应该分配给应用程序其他部分的 RAM 和 CPU 资源。最好是遍历集合，或者使用分页，我们接下来会讨论。

## 还有更多内容

游标有几种方法可供您遍历搜索结果：

+   `hasNext`方法如果游标还有其他可以返回的项，则返回`true`。

+   `next`方法返回游标中的下一个匹配项。

+   `forEach`迭代器接收一个函数，按顺序对游标的每个结果调用该函数。

遍历游标时，最好使用带有`hasNext`的 while 循环并调用 next，或者使用`forEach`；不要只是将结果转换为数组并在列表上循环！这样做需要数据库一次性获取所有记录，可能会非常占用内存。

有时，可能仍然有太多的项目需要处理；您可以使用游标方法`limit`和`skip`来限制返回的条目数量。`limit`方法将搜索限制为您传递的参数数量的条目；`skip`方法跳过您指定的条目数量。

实际上，find 方法实际上接受两个参数：一个 JavaScript 对象是请求的准则，一个可选的 JavaScript 对象定义了结果集的投影，以新的 JavaScript 对象形式返回。

条件可以是精确匹配条件，正如你在上一个例子中看到的那样。你还可以使用特殊操作`$gt`和`$lt`进行匹配，这些操作允许你按基数顺序过滤给定字段。例如，你可能这样写：

```js
var cursor = collection.find({lng: { $gt: 122 } });
```

这将返回所有`lng`字段值大于 122 的记录。

投影是一个你感兴趣的从数据库接收的字段列表，每个字段设置为`true`或`1`。例如，以下代码返回只包含`call`和`_id`字段的 JavaScript 对象：

```js
var cursor = collection.find(
{call: 'kf6gpe-7'}, 
{call: 1, _id: 1});
```

## 参见

参见[`docs.mongodb.org/manual/reference/method/db.collection.find/`](http://docs.mongodb.org/manual/reference/method/db.collection.find/)关于 MongoDB find 方法的文档，该方法是原生驱动程序使 Node.js 应用程序可用的。

# 使用 Node.js 在 MongoDB 中更新文档

在集合中更新一个文档很容易；只需使用集合的`update`方法并传递您想要更新的数据。

## 如何做到…

这是一个简单的例子：

```js
var mongo = require('mongodb').MongoClient;

var url = 'mongodb://localhost:27017/test';

var update = function(collection, callback) {
  collection.update({ call:'kf6gpe-7' }, 
    { $set: { lat: 39.0, lng: -121.0, another: true } }, 
    function(error, result) {
      console.log('Updated with error ' + error);
      console.log(result);
      callback(result);
    });
};

mongo.connect(url, function(error, db) {
  console.log("mongo.connect returned " + error);

  // Get the documents collection
  var collection = db.collection('documents');
  update(collection, function(result) {
    db.close();
  });
});
```

这个模式与`insert`方法相同；`update`是一个异步方法，它调用一个带有错误代码和结果的回调。

## 它是如何工作的…

`update`方法采用一个模板来匹配文档，并用传递给`$set`的 JavaScript 对象的值更新第一个匹配的文档。注意，你也可以向文档中添加新字段，就像我们在这里做的那样；我们添加了一个名为`another`的新字段，其值为`true`。

您可以通过传递文档的 ID 来指定与特定文档的精确匹配，该 ID 位于传递给 update 的模板的`_id`字段中。传递给`update`的模板是一个标准的查询模板，就像你会传递给`find`的那样。

## 还有更多…

默认情况下，`update`更新第一个匹配的文档。如果您想要它更新与您的模板匹配的所有文档，请在更新中传递一个可选的第三个参数，即 JavaScript 对象`{ multi: true }`。您还可以让`update`执行*upsert*，即在匹配成功时进行更新，如果匹配不成功则进行插入。为此，在更新的第三个参数中传递 JavaScript 对象`{ upsert: true }`。这些可以组合使用以匹配多个文档和执行 upsert；如果没有找到，则传递。

```js
{
  multi: true,
  upsert: true
}
```

类似于插入操作，您还可以在这个选项的参数中传递`safe: true`，以确保在返回之前 update 尝试成功，但这样做会牺牲性能。

`update`方法将更新的文档数作为其结果传递给您的回调。

## 也见

参见 MongoDB 原生驱动程序文档中的 update 部分[`github.com/mongodb/node-mongodb-native`](https://github.com/mongodb/node-mongodb-native)或 MongoDB update 方法文档[`docs.mongodb.org/manual/reference/method/db.collection.update/`](http://docs.mongodb.org/manual/reference/method/db.collection.update/)。

# 使用 Node.js 在 MongoDB 中删除文档

在某个时候，您可能希望使用 Node.js 在集合中删除文档。

## 如何做到...

您使用`remove`方法来实现，该方法会从您指定的集合中移除匹配的文档。以下是调用`remove`方法的示例：

```js
var remove = function(collection, callback) {
  collection.remove({ call: 'kf6gpe-7'},
    function(error, result)
    {
      console.log('remove returned ' + error);
      console.log(result);
      callback(result);
    });
};
```

## 如何工作…

这段代码移除了字段`call`值为`kf6gpe-7`的文档。正如您可能猜到的那样，`remove`的搜索条件可以是您会传递给 find 的任何东西。`remove`方法会移除*所有*与您的搜索条件匹配的文档，所以要小心！调用`remove({})`会移除当前集合中的所有文档。

`remove`方法返回从集合中删除的项目的数量。

## 也见

关于 MongoDB 的 remove 方法，请参阅其文档[`docs.mongodb.org/manual/reference/method/db.collection.remove/`](http://docs.mongodb.org/manual/reference/method/db.collection.remove/)。

# 使用 REST 搜索 MongoDB

到目前为止，您可能想知道在使用 MongoDB 时 JSON 扮演什么角色。当您使用像 mongo-rest 这样的 RESTful 接口访问 MongoDB 数据库实例时，文档会使用 JSON 传输到客户端。让我们看看如何从 MongoDB 获取文档列表。

## 如何做到...

使用 Node.js、MongoDB 和 REST 需要几个步骤。

1.  确保您已经按照介绍中的讨论设置了 REST 服务器。您需要创建`rest-server.js`、`routes/documents.js`和`mongo-rest-example.html`这些文件，其中包含我们 RESTful 应用的 UI，并用 Node.js 同时运行 REST 服务器和文档服务器。

1.  其次，确保您正在运行 MongoDB。

1.  接下来，为了处理 REST `GET`请求，我们需要在`documents.js`中定义函数`exports.findAll`，它应该如下所示：

    ```js
    exports.findAll = function(req, res) {
      db.collection('documents', function(err, collection) {
        collection.find().toArray(function(err, items) {
          res.send(items);
        });
      });
    };
    ```

1.  之后，我们需要`mongo-rest-example.html`文件中的`doGet`脚本，它对 REST 服务器上的数据库文档发起 AJAX `GET`请求。这段代码向服务器的`/documents/` URL 发起 AJAX `GET`请求，将返回的 JSON 放入具有`id`为 json 的`div`中，并构建一个 HTML 表格，每个结果文档的结果有一行，提供每个文档的 ID、呼号、纬度和经度等列：

    ```js
    function doGet() { 
      $.ajax({
        type: "GET",
        url: "http://localhost:3000/documents/",
        dataType: 'json',
      })
    .done(function(result) {
        $('#json').html(JSON.stringify(result));
        var resultHtml = 
    '<table><thead>' + 
    '<th><td><b>id</b></td><td><b>call</b></th>' + 
    '<tbody>';
        resultHtml += '<td><b>lat</b></td><td><b>lng</b></td></tr>';

          $.each(result), function(index, item)
          {
            resultHtml += '<tr>';
            resultHtml += '<td>' + item._id + '</td>';
            resultHtml += '<td>' + item.call + '</td>';
            resultHtml += '<td>' + item.lat + '</td>';
            resultHtml += '<td>' + item.lng + '</td>';
            resultHtml += "</tr>";
          };
        $resultHtml += '</tbody></table>';

        $('#result').html(resultHtml);
      })
    }
    ```

## 它是如何工作的…

`findAll`方法是对数据库的直接查询，它使用`find`在我们的集合中匹配所有的文档。你可以扩展它以接受一个查询模板作为 URL 参数，然后将该参数作为 URL 编码的参数传递给 GET URL。

你还可以添加其他参数，例如限制和跳过的参数，如果你处理的数据量很大，你应该考虑这样做。请注意，Express 模块知道它需要将 JavaScript 对象 JSON 编码以 JSON 的形式发送给客户端。

`doGet` JavaScript 代码更简单；它是一个纯粹的 AJAX 调用，后面跟着一个循环，将返回的 JSON 数组解包为对象，并将每个对象作为表格中的一行呈现。

## 还有更多

一个好的 REST 接口还提供了一个通过 ID 查询特定项目的接口，因为通常你希望查询集合，在其中找到一些有趣的内容，然后可能需要对这个特定的 ID 做些什么。我们定义了`findById`方法来接收来自 URL 的 ID，将 ID 转换为 MongoDB 对象`id`，然后仅对该 ID 执行`find`，如下所示：

```js
exports.findById = function(req, res) {
  var id = new objectId(req.params.id);
  db.collection('documents', function(err, collection) {
    collection.findOne({'_id':id}, function(err, item) {
      res.send(item);
    });
  });
};
```

# 使用 REST 在 MongoDB 中创建文档

原则上，使用 REST 创建文档是简单的：在客户端创建 JavaScript 对象，将其编码为 JSON，并`POST`到服务器。让我们看看这个在实际中是如何工作的。

## 如何做到…

这有两部分：客户端部分和服务器部分。

1.  在客户端，我们需要一种方式来获取我们新 MongoDB 文档的数据。在我们的例子中，它是 HTML 页面上的表单字段，我们将它们包装起来，并使用客户端（在 HTML 中）的`doUpsert`方法`POST`到服务器：

    ```js
    function doUpsert(which)
    {
    Var id = $('#id').val();
    var value = {};
      value.call = $('#call').val();
      value.lat = $('#lat').val();
      value.lng = $('#lng').val();

      $('#debug').html(JSON.stringify(value));

    var reqType = which == 'insert' ? "POST" : 'PUT';
      var reqUrl = 'http://localhost:3000/documents/' + 
    (which == 'insert' ? '' : id);

      $.ajax({
        type: reqType,
        url: reqUrl,
        dataType: 'json',
        headers: { 'Content-Type' : 'application/json' },
        data: JSON.stringify(value)
      })
    .done(function(result) {
        $('#json').html(JSON.stringify(result));
    var resultHtml = which == 'insert' ? 'Inserted' : "Updated";
        $('#result').html(resultHtml);
      });
    }
    ```

1.  服务器接受提交的文档，自动使用 body-parser 模块将其从 JSON 转换，并在 documents.js 文件中执行数据库插入：

    ```js
    exports.addDocuments = function(req, res) {
      var documents = req.body;
      db.collection('documents', {safe:true}, 
    function(err, collection) {
    collection.insert(documents, function(err, result) {
     if (err) {
    res.send({'error':'An error has occurred'});
    } else {
     console.log('Success: ' + JSON.stringify(result[0]));
    res.send(result[0]);
            }
        });
     });
    };
    ```

## 它是如何工作的…

客户端代码被 UI 中的插入和更新按钮共同使用，这就是它比你可能最初想的要复杂一点的原因。然而，在 REST 中，插入和更新之间的唯一区别是 URL 和 HTTP 方法（`POST`与`PUT`），因此使用一个方法来处理两者是合理的。

客户端代码首先使用 jQuery 从表单中获取字段值，然后将请求类型设置为`POST`以进行更新。接下来，它构建 REST URL，这应该只是基本文档的 URL，因为新文档没有 ID。最后，它使用`POST`将文档的 JSON 发送到服务器。服务器代码很简单：取请求的一部分作为对象体，并将其插入到数据库的文档集合中，将插入的结果返回给客户端（这是一个很好的模式，以防客户端是新创建文档的 ID 用于任何事情）。

在服务器端，因为我们在使用 body-parser 模块的`jsonParser`实例注册`POST`请求的处理程序时，JSON 解码是自动处理的。

```js
app.post('/documents', jsonParser, documents.addDocuments);
```

### 提示

如果你在路由注册时忘记传递 JSON 解析器，请求体字段甚至不会被定义！所以如果你在使用 Express 向数据库插入空文档，一定要检查这一点。

# 使用 REST 在 MongoDB 中更新文档

更新与插入相同，不同之处在于它需要一个文档 ID，并且客户端使用 HTTP `POST`请求而不是`PUT`请求来信号更新请求。

## 如何做到...

客户端代码与上一个食谱完全相同；只有服务器代码会更改，因为它需要从 URL 中提取 ID 并执行更新而不是插入：

```js
exports.updateDocuments = function(req, res) {
  var id = new objectId(req.params.id);
  var document = req.body;
  db.collection('documents', function(err, collection) {
    collection.update({'_id':id}, document, {safe:true}, 
      function(err, result) {
        if (err) {
          console.log('Error updating documents: ' + err);
          res.send({'error':'An error has occurred'});
        } else {
          console.log('' + result + ' document(s) updated');
          res.send(documents);
        }
    });
  });
};
```

让我们更详细地看看。

## 它是如何工作的...

回到前面食谱中的客户端实现，你看到对于更新，我们在 URL 中包含了 ID。`updateDocuments`方法从请求参数中获取 ID，并将其转换为 MongoDB 对象`id`对象，然后调用`update`，客户端通过`POST`请求传递的文档。

# 使用 REST 在 MongoDB 中删除文档

与更新一样，删除需要一个对象`id`，我们将它在 URL 中传递给 HTTP `DELETE`请求。

## 如何做到...

`doRemove`方法从表单中的`id`字段获取对象`id`，并向由基本 URL 加上对象`id`组成的 URL 发送一个`DELETE`消息：

```js
function doRemove()
{
  var id = $('#id').val();

  if(id == "")'') 
  {
    alert("Must provide an ID to delete!");
    return;
  }

  $.ajax({
    type: 'DELETE',
    url: "http://localhost:3000/documents/" + id  })
  .done(function(result) {
    $('#json').html(JSON.stringify(result));
    var resultHtml = "Deleted";
    $('#result').html(resultHtml);
  });
  }
```

服务器上的删除消息处理程序从 URL 中提取 ID，然后执行`remove`操作：

```js
exports.deleteDocuments = function(req, res) {
  var id = new objectId(req.params.id);
  db.collection('documents', function(err, collection) {
    collection.remove({'_id':id}, {safe:true}, 
    function(err, result) {
      if (err) {
        res.send({'error':'An error has occurred - ' + err});
      } else {
        console.log('' + result + ' document(s) deleted');
        res.send({ result: 'ok' });
      }
    });
  });
};
```

## 它是如何工作的...

在客户端，流程与更新流程相似；我们从`id`表单元素中获取 ID，如果它是 null，它将弹出错误对话框而不是执行 AJAX post。我们使用 HTTP `DELETE`方法进行 AJAX post，在 URL 中将`id`作为文档名称传递给服务器。

在服务器端，我们从请求参数中获取 ID，将其转换为 MongoDB 本地对象 ID，然后将其传递给集合的`remove`方法以删除文档。然后将成功或错误返回给客户端。
