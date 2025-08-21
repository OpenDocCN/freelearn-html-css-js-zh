# 第六章：使用 JSON 与 CouchDB 配合

在上一章中，我们研究了如何使用 JSON 与 MongoDB 配合，MongoDB 是一个流行的 NoSQL 数据库。在本章中，我们继续这一主题，向您展示如何使用 JSON 与 CouchDB 配合，CouchDB 又是另一个流行的 NoSQL 数据库。在这里，你会发现有关以下方面的食谱：

+   安装和设置 CouchDB 和 Cradle

+   使用 Node.js 和 Cradle 连接到 CouchDB 文档

+   使用 Node.js 和 Cradle 创建 CouchDB 数据库

+   使用 Node.js 和 Cradle 在 CouchDB 中创建文档

+   使用 Node.js 和 Cradle 设置数据视图

+   使用 Node.js 和 Cradle 在 CouchDB 中搜索文档

+   使用 Node.js 和 Cradle 在 CouchDB 中更新文档

+   使用 Node.js 和 Cradle 在 CouchDB 中删除文档

+   使用 REST 枚举 CouchDB 记录

+   使用 REST 搜索 CouchDB

+   使用 REST 在 CouchDB 中更新或创建文档

+   使用 REST 在 CouchDB 中删除文档

# 简介

CouchDB 是一个高可用性、可扩展的文档数据库。与 MongoDB 一样，它也是一个 NoSQL 数据库；不同的是，你不是将数据组织成通过 ID 相关联的表，而是将文档放入数据库中。与 MongoDB 不同，CouchDB 有一个有趣的特性，即 *视图*。

你将具有特定的 map 和 reduce 函数的文档放入数据库中，这些函数遍历数据以提供通过索引提供的特定数据视图。视图是缓存的，这使得构建高性能查询变得容易，这些查询返回数据子集或计算的数据（如报告）。

你与 CouchDB 交互的主要方式是通过 REST 接口；即使在本章中讨论的 Cradle 驱动程序，也是利用 REST 接口在幕后进行文档的创建、更新和删除。你还可以用 REST 接口进行查询，无论是通过文档 ID，还是将索引查询转换为视图。

在本章中，我们将研究如何使用 Cradle 模块将 CouchDB 与 Node.js 集成，以及如何从 Web 端对 CouchDB 进行 REST 查询。

# 安装和设置 CouchDB 和 Cradle

CouchDB 提供了主要平台的点击即可运行安装程序。

## 如何进行…

首先，你需要安装服务器。为此，请访问 [`couchdb.apache.org/`](http://couchdb.apache.org/) 并下载适合您平台的安装程序。在安装 Cradle 之前，一定要运行安装程序。

接下来，在命令行上运行以下命令来安装 Cradle：

```js
npm install cradle

```

最后，你需要在 CouchDB 服务器上启用跨资源请求，以允许在 Web 上进行这些请求。为此，请编辑 `/etc/couchdb/default.ini` 文件，并更改以下行：

```js
enable_cors = false
```

以下行：

```js
enable_cors = true
```

你还需要指示你将接受 CORS 请求的哪些源服务器；要启用对所有域名的跨资源请求，请在 `/etc/couchdb/default.ini` 中 `[cors]` 部分添加以下行：

```js
origins = *
```

如果你想要更具体一点，你可以提供一个由逗号分隔的域名列表，来自这些域名的 HTML 内容和脚本将被加载。

最后，你必须启动（或重新启动）CouchDB 服务器。在 Windows 上，假设你没有将其作为服务安装，就去你安装它的 `bin` 目录下运行 `couchdb.bat`；在 Linux 和 Mac OS X 上，杀死并重新启动 CouchDB 服务器进程。

## 它是如何工作的…

Cradle 模块是整合 CouchDB 和 Node.js 的流行方式，尽管如果你愿意，你也可以使用 Node.js 的 request 模块直接进行 REST 请求。

## 也见

关于 CouchDB 的更多信息，请参见 Apache CouchDB 维基百科上的页面：[`docs.couchdb.org/en/latest/contents.html`](http://docs.couchdb.org/en/latest/contents.html)。

# 使用 Node.js 和 Cradle 连接 CouchDB 数据库

尽管 CouchDB 提供了 RESTful 接口，但严格来说，在使用 CouchDB 之前并不需要一定要建立一个数据库连接；Cradle 模块使用连接的概念来管理其内部状态，你仍然需要创建一个连接对象。

## 怎样做到…

下面是如何在你的 Node.js 应用程序中包含 Cradle 模块并初始化它，获取对特定数据库的引用的方法：

```js
var cradle = require('cradle');
var db = new(cradle.Connection)().database('documents');
```

## 它是如何工作的…

这段代码首先包含了 Cradle 模块，然后创建了一个新的 Cradle `Connection` 对象，将其数据库设置为 `documents` 数据库。这初始化了 Cradle，使其使用默认的 CouchDB 主机（localhost）和端口（5984）。如果你需要覆盖主机或端口，可以通过将主机和端口作为 `Connection` 构造函数的第一个和第二个参数来这样做，像这样：

```js
var connection = new(cradle.Connection)('http://example.com', 
  1234);
```

# 使用 Node.js 和 Cradle 创建 CouchDB 数据库

在使用 CouchDB 中的数据库之前，你必须先创建它。

## 怎样做到…

一旦你获得了你想要使用的数据库的句柄，你应该检查它是否存在，如果不存在，则创建它：

```js
db.exists(function (err, exists) {
if (err) {
  console.log('error', err);
} elseif (!exists) {
{
  db.create();
}
});
```

## 它是如何工作的…

`exists` 方法检查数据库是否存在，如果发生错误，调用你提供的回调函数，并带有一个指示数据库是否存在或不存在的标志。如果数据库不存在，你可以使用 `create` 方法来创建它。

这是 Cradle 的一个常见模式，因为 RESTful 接口本质上是非同步的。你会将你想要执行的方法的参数和回调函数传递给它。

### 提示

初学者常犯的一个错误是认为可以调用这些方法而不带回调函数，然后立即执行一些依赖于之前结果的操作。这是行不通的，因为原始操作还没有发生。考虑对同一记录进行插入和更新。插入是异步完成的；如果你尝试同步执行更新，将没有东西可以更新！

## 还有更多…

如果你想销毁一个数据库，你可以使用 `destroy` 方法，它也接受一个回调函数，就像 create 一样。这会销毁数据库中的所有记录，就像你想象的那么彻底，所以要小心使用！

# 使用 Node.js 和 Cradle 在 CouchDB 中创建文档

Cradle 模块提供了`save`方法来将新文档保存到数据库中。你传递要保存的文档和一个当操作完成或失败时调用的回调函数。

## 如何做到这一点...

下面是如何使用`save`保存一个简单记录的方法：

```js
var item =  {
  call: 'kf6gpe-7',
  lat: 37,
  lng: -122
};

db.save(item, function (error, result) {
  if (error) {
    console.log(error);
    // Handle error
  } else {
    var id = result.id;
    var rev = result.rev;
    }
  });
```

## 它是如何工作的…

`save`方法返回一个 JavaScript 对象给你的回调函数，其中包含新创建文档的 ID 和一个内部修订号，以及一个名为 ok 的字段，该字段应该是 true。正如你在标题为《使用 Node.js 在 CouchDB 中更新记录》的食谱中看到的，为了更新一个文档，你需要存储文档的修订版和 ID；否则，你最终会创建一个新的文档或记录保存失败。一个示例结果可能看起来像这样：

```js
{ ok: true,
  id: '80b20994ecdd307b188b11e223001e64',
  rev: '1-60ba89d42cc4bbc1301164a6ae5c3935' }
```

# 如何在 CouchDB 中使用 Node.js 和 Cradle 设置数据视图

你可以通过它们的 ID 查询 CouchDB 的文档，但当然，大多数时候，你会希望发出更复杂的查询，比如将记录中的字段与特定值匹配。CouchDB 允许你定义*视图*，这些视图由集合中的任意键和从视图中派生的对象组成。当你指定一个视图时，你是在指定两个 JavaScript 函数：一个`map`函数将键映射到集合中的项目，然后一个可选的`reduce`函数遍历键和值以创建最终集合。在本食谱中，我们将使用视图的`map`函数通过单个字段创建记录的索引。

## 如何做到这一点...

下面是使用 CouchDB 为数据库添加一个简单视图的方法：

```js
db.save('_design/stations', {
  views: {
    byCall: {
      map: function(doc) {
        if (doc.call) {
          emit(doc.call, doc);
        }
      }
    }
  }
});
```

这为我们的数据库定义了一个单一视图，即`byCall`视图，它由一个呼号到数据库中文档的映射组成。

## 它是如何工作的…

视图是一种强大的方法，可以引用数据库中的文档，因为你可以根据数据库中的每个文档构建任意简单或复杂的文档。

我们的示例创建了一个单一视图`byCall`，存储在`views`目录下（你应该把视图放在这里），由每个记录的呼号字段组成，然后重复记录。CouchDB 定义了`emit`函数，让你为你的视图创建键值对；在这里，我们使用`call`字段作为每个值的关键字，文档本身作为值。你完全可以轻松地定义一个 JavaScript 对象中的字段子集，或者在 JavaScript 字段上计算某物，并发出那个东西。你可以定义多个视图，每个视图在`views`字段中是一个单独的`map`函数。

CouchDB 缓存视图，并根据数据库的变化按需更新它们，将视图数据存储为 B-树，因此在运行时更新和查询视图非常快。正如你在下一个示例中看到的，搜索特定键的视图简单到只需将键传递给视图。

视图在 CouchDB 中只是存储在特定位置的文档，使用函数而不是数据值。内部实现上，CouchDB 在存储视图时编译视图的函数，并在存储发生插入和删除等更改时运行它们。

## 也请参阅

+   关于 CouchDB 视图概念的更多信息，请参阅 CouchDB 维基百科中的[`wiki.apache.org/couchdb/Introduction_to_CouchDB_views`](http://wiki.apache.org/couchdb/Introduction_to_CouchDB_views)。

+   CouchDB 视图 API 文档在[`wiki.apache.org/couchdb/HTTP_view_API`](http://wiki.apache.org/couchdb/HTTP_view_API)。

# 使用 Node.js 和 Cradle 在 CouchDB 中搜索文档

在 CouchDB 中搜索文档就是查询特定视图以获取特定键的问题。Cradle 模块定义了`view`函数来实现这一点。

## 如何进行...

您将传递要执行查询的视图的 URL，然后将您正在搜索的键作为键参数传递，像这样：

```js
var call = "kf6gpe-7";
db.view('stations/byCall/key="' + call + '"', 
  function (error, result) {
    if (result) {
      result.forEach(function (row) {
        console.log(row);
});
```

除了传递您所寻找的视图和键外，您必须传递一个处理结果的回调函数。

## 它是如何工作的…

在这里，我们在`byCall`视图中搜索调用信号为`kf6gpe-7`。回想一下上一个食谱，视图由`call`字段中的调用信号映射到记录组成；当我们使用数据库的`view`方法发出视图请求时，它在那个映射中查找键匹配`kf6gpe-7`的记录，并返回由匹配记录组成的数组结果。该方法使用数组的`forEach`方法遍历数组中的每个元素，一次将每个元素写入控制台。

## 还有更多内容

您可以向视图传递多个参数。最明显的是`key`参数，它让您传递一个键以进行匹配。还有`keys`参数，它让您传递一个键的数组。您还可以传递`startkey`和`endkey`，以查询一个键范围的视图。如果您需要限制结果，您可以使用`limit`和`skip`参数来限制结果数量，或跳过前*n*个匹配的结果。

如果您知道一个文档的 ID，您还可以使用 Cradle 的`get`方法直接获取该对象：

```js
db.get(id, function(error, doc) {
  console.log(doc);
});
```

## 也请参阅

关于您可以对视图执行的查询操作的详细信息，请参阅 CouchDB 维基百科中的[`wiki.apache.org/couchdb/HTTP_view_API#Querying_Options`](http://wiki.apache.org/couchdb/HTTP_view_API#Querying_Options)。

# 使用 Node.js 和 Cradle 在 CouchDB 中更新文档

Cradle 模块定义了`merge`方法，以便让您更新现有文档。

## 如何进行...

以下是一个示例，我们通过指定其 ID 将记录的调用从`kf6gpe-7`更改为`kf6gpe-9`，然后使用新数据执行合并：

```js
var call = "kf6gpe-7";

db.merge(id, {call: 'kf6gpe-9'}, function(error, doc) {
  db.get(id, function(error, doc) {
    console.log(doc);
  });
});
```

从函数中，你可以看到`merge`接收要合并记录的 ID 和一个 JavaScript 对象，该对象包含要替换或添加到现有对象的字段。你还可以传递一个回调函数，当合并操作完成时由 merge 调用。在出错的情况下，错误值将为非零，文档作为第二个参数返回。在这里，我们只是将修订后的文档的内容记录到控制台。

# 使用 Node.js 和 Cradle 在 CouchDB 中删除文档

要删除一个记录，你使用 Cradle 模块的`remove`方法，并传递你想要删除的文档的 ID。

## 如何进行...

下面是一个删除记录的例子：

```js
db.remove(id);
```

通过 ID 删除文档会移除具有指定 ID 的文档。

## 还有更多...

如果你有多个文档要删除，你可以像以下代码那样遍历所有文档，逐一删除每个文档：

```js
db.all(function(err, doc) {
  for(var i = 0; i < doc.length; i++) {
    db.remove(doc[i].id, doc[i].value.rev, function(err, doc) {
      console.log('Removing ' + doc._id);
    });
  }
});
```

这是`remove`的一个更复杂的使用方式；它需要文档的 ID、文档的版本以及一个回调函数，该函数会将每个被移除文档的 ID 记录到控制台。

# 使用 REST 枚举 CouchDB 记录

REST 语义规定，要获取对象集合的完整内容，我们只需向集合的根发送一个`GET`请求。我们可以从启用了 CORS 的 CouchDB 中使用 jQuery 用一个调用完成这个操作。

## 如何进行...

这里有一些 HTML、jQuery 和 JavaScript 代码，它枚举了 CouchDB 视图中的所有项目，并在内嵌表格中显示了每个对象的一些字段：

```js
<!DOCTYPE html>
<html>
<head>
<script src="img/"></script>
<script src="img/"></script>
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
  Rev: <input type="text" id="rev"/>
  Call: <input type="text" id="call"/>
  Lat: <input type="text" id="lat"/>
  Lng: <input type="text" id="lng"/>
  <button type="button" id="insert" 
    onClick="doUpsert('insert')">Insert</button>
  <button type="button" id="update" 
    onClick="doUpsert('update')">Update</button>
  <button type="button" id="remove" 
    onClick="doRemove()">Remove</button>
</form><br/>

<script>

function doGet() { 
  $.ajax({
    type: "GET",
    url: 
"http://localhost:5984/documents/_design/stations/_view/byCall",
    dataType:"json",
  })
  .done(function(result) {
    $('#json').html(JSON.stringify(result));
    var resultHtml = '<table><tr><td><b>id</b></td>';
    resultHtml += '<td><b>revision</b></td><td><b>call</b></td>';
    resultHtml += '<td><b>lat</b></td><td><b>lng</b></td></tr>';
    for(var i = 0; i < result.rows.length; i++)
    {
      var item = result.rows[i]
      resultHtml += "<tr>";
      resultHtml += "<td>" + item.id + "</td>";
      resultHtml += "<td>" + item.value._rev + "</td>";
      resultHtml += "<td>" + item.value.call + "</td>";
      resultHtml += "<td>" + item.value.lat + "</td>";
      resultHtml += "<td>" + item.value.lng + "</td>";
      resultHtml += "</tr>";
    }
    $('#result').html(resultHtml);
});
}
</script>
</html>
```

## 它是如何工作的…

HTML 结构很简单；它包含了 jQuery，然后定义了三个`div`区域来显示请求的结果。之后，它定义了一个表单，包含文档的 ID、版本、呼号、纬度和经度字段，并添加了获取记录列表、执行插入或更新以及移除记录的按钮。

我们需要定义`byCall`视图才能使其工作（参见食谱*使用 Node.js 在 CouchDB 中设置数据视图*，了解如何使用 Node.js 设置数据视图）。这段代码对视图的基本 URL 执行一个 HTTP GET 请求，并取回的 JavaScript 对象（由 jQuery 从 JSON 解析而来）进行格式化，使其成为一个表格。（注意我们本可以附加一个特定的键到 URL 上，以获取单一的 URL）。

REST 响应的格式与使用 Cradle 查询集合的响应略有不同；你看到的是 CouchDB 的实际响应，而不是由 Cradle 处理的成果。以原始形式来看，它看起来像这样：

```js
{"total_rows":1,"offset":0,
  "rows":[
    {"id":"80b20994ecdd307b188b11e223001e64",
"key":"kf6gpe-7",
      "value":{
"_id":"80b20994ecdd307b188b11e223001e64",
"_rev":"1-60ba89d42cc4bbc1301164a6ae5c3935",
"call":"kf6gpe-7","lat":37,"lng":-122
      }
    }
  ]
} 
```

具体来说，`total_rows`字段表示集合中结果有多少行；`offset`字段表示在返回的第一行之前在集合中跳过了多少行，然后`rows`数组包含了映射视图生成的每个键值对。`rows`字段有一个 ID 字段，它是生成该映射条目的唯一 ID，由映射操作生成的键，以及由映射操作生成的记录。

请注意，如果你对数据库的基本 URL 执行一个`GET`请求，你会得到一些不同的事物；不是数据库中的所有记录，而是有关数据库的信息：

```js
{"db_name":"documents",
"doc_count":5,
"doc_del_count":33,
"update_seq":96,
"purge_seq":0,
"compact_running":false,
"disk_size":196712,
"data_size":6587,
"instance_start_time":"1425000784214001",
"disk_format_version":6,
"committed_update_seq":96
}
```

这些字段可能因您运行的 CouchDB 版本而异。

## 参见

有关 CouchDB 的 HTTP REST 接口的信息，请参阅位于[`wiki.apache.org/couchdb/HTTP_Document_API`](http://wiki.apache.org/couchdb/HTTP_Document_API)的文档。

# 使用 REST 搜索 CouchDB

使用 REST 搜索 CouchDB 时，使用一个带有映射的视图来创建你的索引，你插入一次，然后是一个 GET HTTP 请求。

## 如何做到...

我们可以修改之前的`doGet`函数，以搜索特定的呼号，如下所示：

```js
function doGet(call) { 
  $.ajax({
    type: "GET",
    url: 
"http://localhost:5984/documents/_design/stations/_view/byCall" + 
       (call != null & call != '') ? ( '?key=' + call ) : '' ),
    dataType:"json",
  })
  .done(function(result) {
    $('#json').html(JSON.stringify(result));
    var resultHtml = '<table><tr><td><b>id</b></td>';
    resultHtml += '<td><b>revision</b></td><td><b>call</b></td>';
    resultHtml += '<td><b>lat</b></td><td><b>lng</b></td></tr>';
    for(var i = 0; i < result.rows.length; i++)
    {
      var item = result.rows[i]
      resultHtml += "<tr>";
      resultHtml += "<td>" + item.id + "</td>";
      resultHtml += "<td>" + item.value._rev + "</td>";
      resultHtml += "<td>" + item.value.call + "</td>";
      resultHtml += "<td>" + item.value.lat + "</td>";
      resultHtml += "<td>" + item.value.lng + "</td>";
      resultHtml += "</tr>";
    }
    $('#result').html(resultHtml);
  });
}
```

## 它是如何工作的…

相关的行是传递给`doGet`的参数调用，以及我们构造的 URL，我们通过`GET`请求发送到该 URL。注意我们如何检查 null 或空调用以获取整个集合；你的代码可能希望做些不同的事情，比如报告一个错误，特别是如果集合很大的话。

### 提示

请注意，视图必须在这样做之前存在。我喜欢使用 Node.js 在最初更新我的数据库时创建我的视图，并在更改时更新视图，而不是将视图嵌入客户端，因为对于大多数应用程序来说，有很多客户端，没有必要让存储重复更新相同的视图。

# 使用 REST 在 CouchDB 中更新或插入文档

当你想要执行一个更新或插入操作时，Cradle 并没有 REST 等效的合并功能；相反，插入操作由 HTTP `POST`请求处理，而更新操作则由`PUT`请求处理。

## 如何做到...

以下是一些 HTML 和一个`doUpsert`方法，它查看你 HTML 页面上的表单元素，如果数据库中尚不存在文档，则创建新文档，或者如果已存在文档并且你传递了 ID 和修订字段，则更新现有文档：

```js
<!DOCTYPE html>
<html>
<head>
<script src="img/"></script>
<script src="img/"></script>
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
  Rev: <input type="text" id="rev"/>
  Call: <input type="text" id="call"/>
  Lat: <input type="text" id="lat"/>
  Lng: <input type="text" id="lng"/>
  <button type="button" id="insert" 
    onClick="doUpsert('insert')">Insert</button>
  <button type="button" id="update" 
    onClick="doUpsert('update')">Update</button>
  <button type="button" id="remove" 
    onClick="doRemove()">Remove</button>
</form><br/>

<script>

function doUpsert();
{	
  var value = {};
  var which = null;
  id = $('#id').val();

  if (id != '') {
    which = 'insert';
  }

  value.call = $('#call').val();
  value.lat = $('#lat').val();
  value.lng = $('#lng').val();

  if (which != 'insert') {
    value._rev = $('#rev').val();
    value._id = id;
  }

  $('#debug').html(JSON.stringify(value));

  var reqType = which == 'insert' ? "POST" : "PUT";
  var reqUrl = "http://localhost:5984/documents/" + 
    (which == 'insert' ? '' : id);

  $.ajax({
    type: reqType,
    url: reqUrl,
    dataType:"json",
    headers: { 'Content-Type' : 'application/json' },
    data: JSON.stringify(value)
  })
  .done(function(result) {
    $('#json').html(JSON.stringify(result));
    var resultHtml = which == 'insert' ? "Inserted" : "Updated";
    $('#result').html(resultHtml);
  })
}
</script>
</html>
```

## 它是如何工作的…

`doUpsert`方法首先定义一个空 JavaScript 对象，这是我们将其填充并通过`PUT`或`POST`请求发送到服务器的对象。然后我们提取表单字段的值；如果`id`字段设置了 ID，我们假设这是更新操作，而不是插入操作，并且还捕获了名为`rev`的修订字段的值。

如果没有设置 ID 值，它是一个插入操作，我们将请求类型设置为`POST`。如果它是更新，我们将请求类型设置为`PUT`，向 CouchDB 表明这是一个更新。

接下来，我们构造 URL；更新文档的 URL 必须包括要更新的文档的 ID；这就是 CouchDB 知道要更新哪个文档的方式。

最后，我们执行一个我们之前定义类型的 AJAX 请求（`PUT`或`POST`）。当然，我们将发送给服务器的 JavaScript 文档进行 JSON 编码，并包含一个指示发送的文档是 JSON 的头部。

返回的值是一个 JSON 文档（由 jQuery 转换为 JavaScript 对象），包括插入文档的 ID 和修订版，类似于这样：

```js
{ "ok":true,
  "id":"80b20994ecdd307b188b11e223001e64",
  "rev":"2-e7b2a85adef5e721634bdf9a5707eb42"}
```

### 提示

请注意，您更新文档的请求必须包括文档的当前修订版和 ID，否则`PUT`请求将因 HTTP 409 错误而失败。

# 使用 REST 在 CouchDB 中删除文档

您通过向要删除的文档发送带有 ID 和修订版的 HTTP `DELETE`请求来表示 RESTful 删除。

## 如何做到…

使用之前的食谱中的 HTML，这是一个脚本，它从表单字段中提取 ID 和修订版，进行一些简单的错误检查，并向服务器发送具有指示 ID 和修订版的文档的删除请求：

```js
function doRemove()
{	
  id = $('#id').val();
  rev = $('#rev').val();
  if (id == '') 
  {
    alert("Must provide an ID to delete!");
    return;
  }
  if (rev == '')
  {
    alert("Must provide a document revision!");
    return;
  }

  $.ajax({
    type: "DELETE",
    url: "http://localhost:5984/documents/" + id + '?rev=' + rev,
  })
  .done(function(result) {
    $('#json').html(JSON.stringify(result));
    var resultHtml = "Deleted";
    $('#result').html(resultHtml);
  })
}
```

## 它是如何工作的…

代码首先从表单元素中提取 ID 和修订版，如果任何一个为空则弹出错误对话框。接下来，构建一个 AJAX HTTP `DELETE`请求。URL 是文档的 URL - 数据库和文档 ID，修订版作为名为`rev`的参数传递。假设您正确指定了 ID 和修订版，您将得到与更新相同的响应：被删除文档的 ID 和修订版。 如果失败，您将得到一个 HTTP 错误。
