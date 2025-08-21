# 第八章：与服务器通信

在本章中，我们将涵盖以下主题：

+   创建一个 HTTP GET 请求来获取 JSON

+   创建带有自定义头部的请求

+   为你的 API 进行版本控制

+   使用 JSONP 获取 JSON 数据

+   从服务器读取 XML 数据

+   使用 FormData 接口

+   将二进制文件发布到服务器

+   使用 Node.js 创建 SSL 连接

+   使用 Ajax Push 进行实时更新

+   使用 WebSockets 交换实时消息

# 创建一个 HTTP GET 请求来获取 JSON

从服务器检索信息的基本方法之一是使用 HTTP GET。在 RESTful 方式中，这种方法应该仅用于读取数据。因此，GET 调用不应该改变服务器状态。现在，这对于每种可能的情况可能并不正确，例如，如果我们在某个资源上有一个视图计数器，那么这是一个真正的改变吗？如果我们严格遵循定义，那么是的，这是一个改变，但这远非重要到足以被考虑。

在浏览器中打开一个网页会发出一个 GET 请求，但通常我们希望以一种脚本化的方式来检索数据。这通常是为了实现**异步 JavaScript 和 XML**（**AJAX**），允许重新加载数据而不进行完整的页面重新加载。尽管名称中包含 XML，但并不是必需的，如今，JSON 是首选的格式。

JavaScript 和`XMLHttpRequest`对象的组合提供了一种异步交换数据的方法，在这个示例中，我们将看到如何使用纯 JavaScript 和 jQuery 从服务器读取 JSON。为什么使用纯 JavaScript 而不直接使用 jQuery？我们坚信 jQuery 简化了 DOM API，但它并不总是可用，此外，我们需要了解异步数据传输背后的基础代码，以充分理解应用程序的工作原理。

## 准备工作

服务器将使用 Node.js 实现。请参考附录 A 中关于如何在您的计算机上安装 Node.js 以及如何使用 npm 的内容。在这个示例中，为了简单起见，我们将使用**restify**（[`mcavage.github.io/node-restify/`](http://mcavage.github.io/node-restify/)），这是一个用于创建正确的 REST web 服务的 Node.js 模块。

## 如何做到...

让我们执行以下步骤。

1.  为了在服务器端脚本的根目录中包含`restify`到我们的项目中，使用以下命令：

```html
npm install restify

```

1.  添加依赖项后，我们可以继续创建服务器代码。我们创建一个`server.js`文件，它将由 Node.js 运行，在其开头我们添加`restify`：

```html
var restify = require('restify');
```

1.  有了这个`restify`对象，我们现在可以创建一个服务器对象，并为`get`方法添加处理程序：

```html
var server = restify.createServer();
server.get('hi', respond);
server.get('hi/:index', respond);
```

1.  `get`处理程序回调到一个名为`respond`的函数，因此我们现在可以定义这个函数，它将返回 JSON 数据。我们将创建一个名为`hello`的示例 JavaScript 对象，并且如果函数被调用时具有请求的参数索引部分，则在`"hi/:index"`处理程序中调用它：

```html
 function respond(req, res, next) {
  console.log("Got HTTP " + req.method + " on " + req.url + " responding");
  var hello = [{
    'id':'0',
    'hello': 'world'
  },{
    'id':'1',
    'say':'what'
  }];
  if(req.params.index){
    var found = hello[req.params.index];
    if(found){
      res.send(found);
    } else {
      res.status(404);
      res.send();
    }
  };
  res.send(hello);
  addHeaders(req,res);
  return next();
}
```

1.  我们在开始时调用的`addHeaders`函数是为了添加头部，以便访问来自不同域或不同服务器端口的资源：

```html
function addHeaders(req, res) {
  res.header("Access-Control-Allow-Origin", "*");
  res.header("Access-Control-Allow-Headers", "X-Requested-With");
 };
```

1.  头部的定义和它们的含义将在本章后面讨论。现在，让我们说它们使得浏览器可以使用 AJAX 访问资源。最后，我们添加一段代码，将服务器设置为监听 8080 端口：

```html
server.listen(8080, function() {
  console.log('%s listening at %s', server.name, server.url);
});
```

1.  要使用命令行启动服务器，我们输入以下命令：

```html
node server.js

```

1.  如果一切顺利，我们将在日志中收到一条消息：

```html
restify listening at http://0.0.0.0:8080
```

1.  然后我们可以通过直接从浏览器访问我们定义的 URL `http://localhost:8080/hi`来测试它，或者使用附录 A 中讨论的一些工具来查看通信，*安装 Node.js 和使用 npm*。

现在我们可以继续进行客户端 HTML 和 JavaScript。我们将实现两种从服务器读取数据的方式，一种使用标准的`XMLHttpRequest`，另一种使用`jQuery.get()`。请注意，并非所有功能都与所有浏览器完全兼容。

1.  我们创建了一个简单的页面，其中有两个`div`元素，一个带有 ID`data`，另一个带有 ID`say`。这些元素将用作从服务器加载数据的占位符：

```html
    Hello <div id="data">loading</div>
    <hr/>
    Say <div id="say">No</div>s
    <script src="img/jquery.min.js"></script>
    <script src="img/example.js"></script>
    <script src="img/exampleJQuery.js"></script>
```

1.  在`example.js`文件中，我们定义了一个名为`getData`的函数，它将创建一个 AJAX 调用到给定的`url`，并在请求成功时进行回调：

```html
  function getData(url, onSuccess) {
    var request = new XMLHttpRequest();
    request.open("GET", url);
    request.onload = function() {
      if (request.status === 200) {
        console.log(request);
        onSuccess(request.response);
      }
    };
    request.send(null);
  }
```

1.  之后，我们可以直接调用该函数，但为了演示调用发生在页面加载后，我们将在三秒后调用它：

```html
 setTimeout(
    function() {
      getData(
        'http://localhost:8080/hi',
        function(response){
          console.log('finished getting data');
          var div = document.getElementById('data');
          var data = JSON.parse(response);
          div.innerHTML = data[0].hello;
        })
    },
    3000);
```

1.  jQuery 版本更加简洁，因为标准 DOM API 和事件处理带来的复杂性大大减少：

```html
    (function(){
    $.getJSON('http://localhost:8080/hi/1', function(data) {
      $('#say').text(data.say);
 });
}())
```

## 工作原理...

一开始，我们使用`npm install restify`安装了依赖项；这足以使其工作，但为了更加明确地定义依赖关系，npm 有一种指定的方法。我们可以添加一个名为`package.json`的文件，这是一个主要用于发布 Node.js 应用程序的打包格式。在我们的情况下，我们可以使用以下代码定义`package.json`：

```html
{
  "name" : "ch8-tip1-http-get-example",
  "description" : "example on http get",
  "dependencies" : ["restify"],
  "author" : "Mite Mitreski",
  "main" : "html5dasc",
  "version" : "0.0.1"
}
```

如果我们有一个像这样的文件，npm 将在调用`npm install`时自动处理依赖项的安装，该命令是在放置`package.json`文件的目录中从命令行中调用的。

`Restify`有一个简单的路由，其中函数被映射到给定 URL 的适当方法。`'/hi'`的 HTTP GET 请求与`server.get('hi', theCallback)`映射，其中执行`theCallback`，并应返回一个响应。

当我们有一个参数化的资源时，例如在`'hi/:index'`中，与`:index`相关联的值将在`req.params`下可用。例如，在对`'/hi/john'`的请求中，要访问`john`值，我们只需使用`req.params.index`。此外，index 的值在传递给我们的处理程序之前将自动进行 URL 解码。在`restify`中请求处理程序的另一个值得注意的部分是我们在最后调用的`next()`函数。在我们的情况下，这大多数情况下并没有太多意义，但一般来说，如果我们希望调用链中的下一个处理程序函数被调用，我们负责调用它。在特殊情况下，还有一种使用`error`对象触发自定义响应的方法来调用`next()`。

在客户端代码方面，`XMLHttpRequest`是异步调用背后的机制，当调用`request.open("GET", url, true)`并将最后一个参数值设置为`true`时，我们获得了真正的异步执行。现在你可能会想为什么这个参数在这里，难道不是在加载页面后已经完成了调用吗？这是真的，调用是在加载页面后完成的，但是，例如，如果参数设置为`false`，请求的执行将是一个阻塞方法，或者用通俗的话来说，脚本将暂停，直到我们得到一个响应。这可能看起来是一个小细节，但它对性能有很大的影响。

jQuery 部分非常简单；有一个函数接受资源的 URL 值，数据处理函数，以及一个`success`函数，在成功获取响应后调用：

```html
jQuery.getJSON( url [, data ] [, success(data, textStatus, jqXHR) ] )
```

当我们打开`index.htm`时，服务器应该记录类似以下的内容：

```html
Got HTTP GET on /hi/1 responding
Got HTTP GET on /hi responding

```

这里一个来自 jQuery 请求，另一个来自纯 JavaScript。

## 还有更多...

**XMLHttpRequest Level 2**是添加到浏览器中的新改进之一，尽管它不是 HTML5 的一部分，但仍然是一个重大变化。Level 2 的变化中有几个功能，主要是为了使其能够处理文件和数据流，但也有一个我们已经使用的简化。以前，我们必须使用`onreadystatechange`并遍历所有状态，如果`readyState`是`4`，即等于`DONE`，我们才能读取数据：

```html
var xhr = new XMLHttpRequest();
xhr.open('GET', 'someurl', true);
xhr.onreadystatechange = function(e) {
  if (this.readyState == 4 && this.status == 200) {
   // response is loaded
  }
}
```

然而，在 Level 2 请求中，我们可以直接使用`request.onload = function() {}`而不必检查状态。表中可以看到可能的状态：

| 状态名称 | 数值 | 描述 |
| --- | --- | --- |
| `UNSENT` | `0` | 对象已创建 |
| `OPENED` | `1` | 调用了`open`方法 |
| `HEADERS_RECEIVED` | `2` | 已经跟踪了所有重定向，并且最终对象的所有标头现在都可用 |
| `LOADING` | `3` | 响应正在被恢复 |
| `DONE` | `4` | 已接收数据或在传输过程中出现问题，例如无限重定向 |

还有一件事需要注意的是，`XMLHttpRequest` Level 2 在所有主要浏览器和 IE 10 中都受支持；旧版`XMLHttpRequest`在较旧版本的 IE（早于 IE 7）中实例化的方式不同，我们可以通过新的`ActiveXObject("Msxml2.XMLHTTP.6.0")`通过 ActiveX 对象访问它。

# 使用自定义标头创建请求

HTTP 头是发送到服务器的`request`对象的一部分。其中许多提供有关客户端用户代理设置和配置的信息，因为这有时是制作从服务器获取的资源的描述的基础。其中一些，如`Etag`、`Expires`和`If-Modified-Since`与缓存密切相关，而其他一些，如`DNT`代表“不要跟踪”（[`www.w3.org/2011/tracking-protection/drafts/tracking-dnt.html`](http://www.w3.org/2011/tracking-protection/drafts/tracking-dnt.html)）可能是相当有争议的。在这个示例中，我们将看一种在服务器和客户端代码中使用自定义`X-Myapp`头的方法。

## 准备工作

服务器将使用 Node.js 实现，因此您可以参考附录 A，*安装 Node.js 和使用 npm*，了解如何在您的计算机上安装 Node.js 以及如何使用 npm。在这个例子中，为了简单起见，我们将使用 restify ([`mcavage.github.io/node-restify/`](http://mcavage.github.io/node-restify/))。此外，在浏览器和服务器中监视控制台对于理解后台发生的事情至关重要。

## 如何做...

1.  我们可以从`package.json`文件中定义服务器端的依赖项开始：

```html
{
  "name" : "ch8-tip2-custom-headers",
  "dependencies" : ["restify"],
  "main" : "html5dasc",
  "version" : "0.0.1"
}
```

1.  之后，我们可以从命令行调用`npm install`，这将自动检索`restify`并将其放置在项目根目录中创建的`node_modules`文件夹中。在这部分之后，我们可以继续在`server.js`文件中创建服务器端代码，在那里我们将服务器设置为侦听端口 8080，并为`'hi'`和其他路径添加一个路由处理程序，当请求方法为`HTTP OPTIONS`时：

```html
var restify = require('restify');
var server = restify.createServer();
server.get('hi', addHeaders, respond);
server.opts(/\.*/, addHeaders, function (req, res, next) {
  console.log("Got HTTP " + req.method + " on " + req.url + " with headers\n");
 res.send(200);
  return next();
});
server.listen(8080, function() {
  console.log('%s listening at %s', server.name, server.url);
});
```

### 注意

在大多数情况下，当我们将应用程序的构建写入 Restify 时，文档应该足够了，但有时，查看源代码也是一个好主意。它可以在[`github.com/mcavage/node-restify/`](https://github.com/mcavage/node-restify/)上找到。

1.  值得注意的一件事是，我们可以有多个链接的处理程序；在这种情况下，我们在其他处理程序之前有`addHeaders`。为了使每个处理程序都能传播，应该调用`next()`：

```html
function addHeaders(req, res, next) {
  res.setHeader("Access-Control-Allow-Origin", "*");
  res.setHeader('Access-Control-Allow-Headers', 'X-Requested-With, X-Myapp');
  res.setHeader('Access-Control-Allow-Methods', 'GET, OPTIONS');
  res.setHeader('Access-Control-Expose-Headers', 'X-Myapp, X-Requested-With');
  return next();
};
```

`addHeaders`添加了访问控制选项，以启用跨源资源共享。**跨源资源共享**（**CORS**）定义了浏览器和服务器可以相互交互以确定是否应该允许请求的方式。它比允许所有跨源请求更安全，但比简单地允许所有跨源请求更强大。

1.  之后，我们可以创建处理程序函数，该函数将返回服务器接收到的标题和一个 hello world 类型的 JSON 响应：

```html
   function respond(req, res, next) {
  console.log("Got HTTP " + req.method + " on " + req.url + " with headers\n");
  console.log("Request: ", req.headers);
  var hello = [{
    'id':'0',
    'hello': 'world',
    'headers': req.headers
  }];
  res.send(hello);
  console.log('Response:\n ', res.headers());
  return next();
}
```

此外，我们还将请求和响应头记录到服务器控制台日志中，以便查看后台发生了什么。

1.  对于客户端代码，我们需要一个简单的"纯净"JavaScript 方法和 jQuery 方法，因此为了做到这一点，包括`example.js`和`exampleJquery.js`以及一些`div`元素，我们将用它们来显示从服务器检索到的数据：

```html
     Hi <div id="data">loading</div>
    <hr/>
    Headers list from the request: <div id="headers"></div>
    <hr/>
    Data from jQuery: <div id="dataRecieved">loading</div>
    <script src="img/jquery.min.js"></script>
    <script src="img/example.js"></script>
    <script src="img/exampleJQuery.js"></script>
```

1.  添加标题的一个简单方法是在`open()`调用之后在`XMLHttpRequest`对象上调用`setRequestHeader`：

```html
  function getData(url, onSucess) {
    var request = new XMLHttpRequest();
    request.open("GET", url, true);
    request.setRequestHeader("X-Myapp","super");
    request.setRequestHeader("X-Myapp","awesome");
    request.onload = function() {
      if (request.status === 200) {
        onSuccess(request.response);
      }
    };
    request.send(null);
  }
```

1.  XMLHttpRequest 会自动设置标题，比如"Content-Length"，"Referer"和"User-Agent"，并且不允许你使用 JavaScript 更改它们。

### 注意

关于这一点的更完整的标题列表和背后的原因可以在 W3C 文档中找到，网址是[`www.w3.org/TR/XMLHttpRequest/#the-setrequestheader%28%29-method`](http://www.w3.org/TR/XMLHttpRequest/#the-setrequestheader%28%29-method)。

1.  为了打印结果，我们添加一个函数，该函数将把每个标题键和值添加到无序列表中：

```html
  getData(
    'http://localhost:8080/hi',
    function(response){
      console.log('finished getting data');
      var data = JSON.parse(response);
      document.getElementById('data').innerHTML = data[0].hello;
      var headers = data[0].headers,
          headersList = "<ul>";
      for(var key in headers){
        headersList += '<li><b>' + key + '</b>: ' + headers[key] +'</li>';
      };
      headersList += "</ul>";
      document.getElementById('headers').innerHTML = headersList;
    });
```

1.  当这个被执行时，所有请求头的列表应该显示在页面上，我们自定义的`x-myapp`应该显示出来：

```html
 host: localhost:8080
 connection: keep-alive
 origin: http://localhost:8000
 x-myapp: super, awesome
 user-agent: Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.27 (KHTML, like Gecko) Chrome/26.0.1386.0 Safari/537.27
```

1.  jQuery 方法更简单，我们可以使用`beforeSend`钩子调用一个函数来设置`'x-myapp'`标题。当我们收到响应时，将其写入到 ID 为`dataRecived`的元素中：

```html
$.ajax({
    beforeSend: function (xhr) {
      xhr.setRequestHeader('x-myapp', 'this was easy');
    },
    success: function (data) {
     $('#dataRecieved').text(data[0].headers['x-myapp']);
    }
```

1.  jQuery 示例的输出将是包含在`x-myapp`标题中的数据：

```html
Data from jQuery: this was easy

```

## 工作原理...

您可能已经注意到，在服务器端，我们添加了一个处理`HTTP OPTIONS`方法的路由，但我们从未明确地在那里调用过。如果我们查看服务器日志，应该会有类似以下输出的内容：

```html
      Got HTTP OPTIONS on /hi with headers
      Got HTTP GET on /hi with headers
```

这是因为浏览器首先发出一个**预检请求**，这在某种程度上是浏览器询问是否有权限发出"真正"请求。一旦获得了许可，原始的 GET 请求就会发生。如果`OPTIONS`响应被缓存，浏览器将不会为后续请求发出任何额外的预检请求。

`XMLHttpRequest`的`setRequestHeader`函数实际上将每个值附加为逗号分隔的值列表。由于我们调用了两次该函数，标题的值如下：

```html
'x-myapp': 'super, awesome'
```

## 还有更多...

对于大多数用例，我们不需要自定义标题成为我们逻辑的一部分，但有很多 API 会充分利用它们。例如，许多服务器端技术会添加包含一些元信息的`X-Powered-By`标题，比如`JBoss 6`或`PHP/5.3.0`。另一个例子是 Google Cloud Storage，其中除了其他标题之外，还有以`x-goog-meta`为前缀的标题，比如`x-goog-meta-project-name`和`x-goog-meta-project-manager`。

# 对 API 进行版本控制

在进行第一次实现时，我们并不总是有最佳解决方案。API 可以扩展到一定程度，但之后需要进行一些结构性的更改。但我们可能已经有依赖于当前版本的用户，因此我们需要一种方式来拥有同一资源的不同表示版本。一旦一个模块有了用户，API 就不能随我们的意愿改变。

解决此问题的一种方法是使用所谓的 URL 版本控制，我们只需添加一个前缀。例如，如果旧的 URL 是`http://example.com/rest/employees`，新的 URL 可以是`http://example.com/rest/v1/employees`，或者在子域下可以是[`v1.example.com/rest/employee`](http://v1.example.com/rest/employee)。只有在您对所有服务器和客户端都有直接控制权时，此方法才有效。否则，您需要有一种处理回退到旧版本的方法。

在这个示例中，我们将实现所谓的"语义版本"，[`semver.org/`](http://semver.org/)，使用 HTTP 头来指定接受的版本。

## 准备工作

服务器将使用 Node.js 实现，因此您可以参考附录 A，*安装 Node.js 和使用 npm*，了解如何在您的计算机上安装 Node.js 以及如何使用 npm。在本例中，我们将使用 restify ([`mcavage.github.io/node-restify/`](http://mcavage.github.io/node-restify/)) 作为服务器端逻辑来监视请求以了解发送了什么。

## 如何做...

让我们执行以下步骤。

1.  我们需要首先定义依赖关系，然后安装`restify`，然后我们可以继续创建服务器代码。与之前的示例的主要区别是定义`"Accept-version"`头。restify 具有内置处理此头的功能，使用**版本化路由**。创建服务器对象后，我们可以设置哪些方法将在哪个版本上调用：

```html
server.get({ path: "hi", version: '2.1.1'}, addHeaders, helloV2, logReqRes);
server.get({ path: "hi", version: '1.1.1'}, addHeaders, helloV1, logReqRes);
```

1.  我们还需要处理`HTTP OPTIONS`，因为我们使用跨域资源共享，浏览器需要进行额外的请求以获取权限：

```html
server.opts(/\.*/, addHeaders, logReqRes, function (req, res, next) {
  res.send(200);
  return next();
});
```

1.  版本 1 和版本 2 的处理程序将返回不同的对象，以便我们可以轻松地注意到 API 调用之间的差异。在一般情况下，资源应该是相同的，但可能有不同的结构变化。对于版本 1，我们可以有以下内容：

```html
function helloV1(req, res, next) {
  var hello = [{
    'id':'0',
    'hello': 'grumpy old data',
    'headers': req.headers
  }];
  res.send(hello);
  return next()
}
```

1.  至于版本 2，我们有以下内容：

```html
function helloV2(req, res, next) {
  var hello = [{
    'id':'0',
    'awesome-new-feature':{
      'hello': 'awesomeness'
    },
    'headers': req.headers
  }];
  res.send(hello);
  return next();
}
```

1.  我们必须做的另一件事是添加 CORS 头，以启用`accept-version`头，因此在路由中包含了`addHeaders`，应该类似于以下内容：

```html
function addHeaders(req, res, next) {
  res.setHeader("Access-Control-Allow-Origin", "*");
  res.setHeader('Access-Control-Allow-Headers', 'X-Requested-With, accept-version');
  res.setHeader('Access-Control-Allow-Methods', 'GET, OPTIONS');
  res.setHeader('Access-Control-Expose-Headers', 'X-Requested-With, accept-version');
  return next();
};
```

### 注意

请注意，您不应忘记调用`next()`，以便在路由链中调用下一个函数。

1.  为了简单起见，我们只会在 jQuery 中实现客户端，因此我们创建一个简单的 HTML 文档，其中包括必要的 JavaScript 依赖项：

```html
     Old api: <div id="data">loading</div>
    <hr/>
     New one: <div id="dataNew"> </div>
    <hr/>
    <script src="img/jquery.min.js"></script>
    <script src="img/exampleJQuery.js"></script>
```

1.  在`example.js`文件中，我们对我们的 REST API 进行了两次 AJAX 调用，一次设置为使用版本 1，另一次设置为使用版本 2：

```html
  $.ajax({
      url: 'http://localhost:8080/hi',
      type: 'GET',
      dataType: 'json',
      success: function (data) {
      $('#data').text(data[0].hello);
    },
    beforeSend: function (xhr) {
      xhr.setRequestHeader('accept-version', '~1');
    }
  });
  $.ajax({
      url: 'http://localhost:8080/hi',
      type: 'GET',
      dataType: 'json',
      success: function (data) {
      $('#dataNew').text(data[0]['awesome-new-feature'].hello);
    },
    beforeSend: function (xhr) {
      xhr.setRequestHeader('accept-version', '~2');
    }
  });
```

注意`accept-version`头包含值`~1`和`~2`。这表示所有语义版本，例如 1.1.0 和 1.1.1 1.2.1，都将被`~1`匹配，类似地对于`~2`。最后，我们应该得到如下文本的输出：

```html
Old api:grumpy old data
New one:awesomeness

```

## 它是如何工作的...

版本化路由是 restify 的内置功能，通过使用`accept-version`来工作。在我们的示例中，我们使用了版本`~1`和`~2`，但是如果我们不指定版本会发生什么呢？restify 将为我们做出选择，因为请求将被视为客户端发送了`*`版本的方式进行处理。我们的代码中定义的第一个匹配路由将被使用。还有一个选项，可以设置路由以匹配多个版本，通过为某个处理程序添加版本列表：

```html
  server.get({path: 'hi', version: ['1.1.0',  '1.1.1',  '1.2.1']}, sendOld);
```

这种类型的版本控制非常适合在不断增长的应用程序中使用，因为随着 API 的变化，客户端可以保持其 API 版本而无需在客户端开发中进行任何额外的努力或更改。这意味着我们不必对应用程序进行更新。另一方面，如果客户端确信他们的应用程序将在更新的 API 版本上运行，他们可以简单地更改请求头。

## 还有更多...

版本控制可以通过使用自定义内容类型来实现，前缀为`vnd`，例如，`application/vnd.mycompany.user-v1`。一个例子是谷歌地球的内容类型 KML，其中定义为`application/vnd.google-earth.kml+xml`。注意内容类型可以有两部分；我们可以有`application/vnd.mycompany-v1+json`，其中第二部分将是响应的格式。

# 使用 JSONP 获取 JSON 数据

JSONP 或带填充的 JSON 是一种利用`<script>`标签进行跨域请求的机制。通过简单地在`script`元素上设置`src`属性或添加元素本身来执行 AJAX 传输。浏览器将执行 HTTP 请求以下载指定的 URL，这不受同源策略的限制，这意味着我们可以使用它从不受我们控制的服务器获取数据。在本示例中，我们将创建一个简单的 JSONP 请求和一个简单的服务器来支持它。

## 准备工作

我们将创建一个简化的服务器实现，该服务器在之前的示例中使用过，因此我们需要安装 Node.js 和 restify（[`mcavage.github.io/node-restify/`](http://mcavage.github.io/node-restify/)），可以通过定义`package.json`或简单安装来安装。有关使用 Node.js，请参阅附录 A，*安装 Node.js 和使用 npm*。

## 如何做...

1.  首先，我们将创建一个简单的路由处理程序，它将返回一个 JSON 对象：

```html
function respond(req, res, next) {
  console.log("Got HTTP " + req.method + " on " + req.url + " responding");
  var hello = [{
    'id':'0',
    'what': 'hi there stranger'
  }];
  res.send(hello);
  return next();
}
```

1.  我们可以自己编写一个版本，将响应包装成具有给定名称的 JavaScript 函数，但是为了在使用 restify 时启用 JSONP，我们可以简单地启用捆绑的插件。这是通过指定要使用的插件来完成的：

```html
var server = restify.createServer();
server.use(restify.jsonp());
server.get('hi', respond);
```

1.  之后，我们只需将服务器设置为监听端口 8080：

```html
server.listen(8080, function() {
  console.log('%s listening at %s', server.name, server.url);
});
```

1.  内置插件会检查请求字符串中是否有名为`callback`或`jsonp`的参数，如果找到，结果将是带有作为这些参数之一的值传递的函数名的 JSONP。例如，在我们的情况下，如果我们在`http://localhost:8080/hi`上打开浏览器，我们会得到以下结果：

```html
[{"id":"0","what":"hi there stranger"}]
```

1.  如果我们使用`callback`参数或设置了 JSONP 的相同 URL，例如`http://localhost:8080/hi?callback=great`，我们应该收到用该函数名包装的相同数据：

```html
great([{"id":"0","what":"hi there stranger"}]);
```

这就是 JSONP 中的 P，表示填充的地方。

1.  因此，我们接下来需要创建一个 HTML 文件，其中我们将显示来自服务器的数据，并包括两个脚本，一个用于纯 JavaScript 方法，另一个用于 jQuery 方法：

```html
    <b>Hello far away server: </b>
    <div id="data">loading</div>
    <hr/>
    <div id="oneMoreTime">...</div>
    <script src="img/jquery.min.js"></script>
    <script src="img/example.js"></script>
    <script src="img/exampleJQuery.js"></script>
```

1.  我们可以继续创建`example.js`，在其中创建两个函数；一个将创建一个`script`元素，并将`src`的值设置为`http://localhost:8080/?callback=cool.run`，另一个将在接收到数据时作为回调服务：

```html
var cool = (function(){
  var module = {};

  module.run = function(data){
    document.getElementById('data').innerHTML = data[0].what;
  }

  module.addElement = function (){
    var script = document.createElement('script');
    script.src = 'http://localhost:8080/hi?callback=cool.run'
    document.getElementById('data').appendChild(script);
    return true;
  }
  return module;
}());
```

1.  之后，我们只需要添加元素的函数：

```html
cool.addElement();
```

这应该从服务器读取数据并显示类似以下的结果：

```html
Hello far away server:
hi there stranger

```

从`cool`对象中，我们可以直接运行`addElement`函数，因为我们将其定义为自执行。

1.  jQuery 示例要简单得多；我们可以将数据类型设置为 JSONP，其他一切都与任何其他 AJAX 调用一样，至少从 API 的角度来看：

```html
$.ajax({
    type : "GET",
    dataType : "jsonp",
    url : 'http://localhost:8080/hi',
    success: function(obj){
      $('#oneMoreTime').text(obj[0].what);
    }
});
```

我们现在可以使用标准的`success`回调来处理从服务器接收到的数据，而且我们不必在请求中指定参数。jQuery 将自动将`callback`参数附加到 URL，并将调用委托给`success`回调。

## 它是如何工作的...

我们在这里所做的第一个重大飞跃是信任数据的来源。从服务器返回的结果在从服务器下载数据后进行评估。已经有一些努力在[`json-p.org/`](http://json-p.org/)上定义更安全的 JSONP，但它远未普及。

下载本身是通过添加另一个主要限制到可用性的`HTTP GET`方法。**超媒体作为应用程序状态的引擎**（**HATEOAS**），等等，定义了使用 HTTP 方法进行创建、更新和删除操作，使得 JSONP 对于这些用例非常不稳定。

另一个有趣的地方是 jQuery 如何将调用委托给`success`回调。为了实现这一点，会创建一个唯一的函数名，并将其发送到`callback`参数，例如：

```html
/hi?callback=jQuery182031846177391707897_1359599143721&_=1359599143727
```

此函数稍后会回调到`jQuey.ajax`的适当处理程序。

## 还有更多...

使用 jQuery，如果应该处理`jsonp`的服务器参数不叫`callback`，我们也可以使用自定义函数。这是通过以下配置完成的：

```html
jsonp: false, jsonpCallback: "my callback"
```

与 JSONP 一样，我们不使用`XMLHttpRequest`，也不期望任何与 AJAX 调用一起使用的函数被执行或者参数被填充。期望这样做是一个非常常见的错误。关于这一点可以在 jQuery 文档中找到更多信息：[`api.jquery.com/category/ajax/`](http://api.jquery.com/category/ajax/)。

# 从服务器读取 XML 数据

REST 服务的另一种常见数据格式是 XML。如果我们有选择格式的选项，那么几乎没有情况下 JSON 不是更好的选择。如果我们需要使用多个命名空间和模式进行严格消息验证，或者出于某种原因，我们使用**可扩展样式表语言转换**（**XSTL**），那么 XML 是更好的选择。最重要的原因是需要处理和支持不使用 JSON 的传统环境。大多数现代服务器端框架都内置了内容协商支持，这意味着根据客户端的请求，它们可以以不同的格式提供相同的资源。在这个示例中，我们将创建一个简单的 XML 服务器，并从客户端使用它。

## 准备工作

对于服务器端，我们将使用 Node.js 和 restify（[`mcavage.github.io/node-restify/`](http://mcavage.github.io/node-restify/)）进行 REST 服务，并使用 xmlbuilder（[`github.com/oozcitak/xmlbuilder-js`](https://github.com/oozcitak/xmlbuilder-js)）创建简单的 XML 文档。为此，我们可以使用 npm 安装依赖项，或者定义一个简单的`package.json`文件，例如示例文件中提供的文件。

## 如何做...

让我们按照以下步骤演示 XML 的使用。

1.  服务器端代码与我们之前创建的其他基于 restify 的示例类似。由于我们只是想演示 XML 的使用，我们可以使用 xmlbuilder 创建一个简单的结构：

```html
var restify = require('restify');
var builder = require('xmlbuilder');
var doc = builder.create();
doc.begin('root')
  .ele('human')
    .att('type', 'female')
      .txt('some gal')
      .up()
  .ele('human')
    .att('type', 'male')
      .txt('some guy')
  .up()
  .ele('alien')
    .txt('complete');
```

1.  使用起来非常简单；`doc.begin('root')`语句创建了文档的根，`ele()`和`att()`语句分别创建了元素和属性。由于我们总是在上次添加的嵌套级别上添加新的部分，为了将光标移动到上一个级别，我们只需调用`up()`函数。

在我们的情况下，将生成的文档如下：

```html
<root>
  <human type="female">some gal</human>
  <human type="male">some guy</human>
  <alien>complete</alien>
</root>
```

1.  为了为资源创建路由，我们可以创建`server.get('hi', addHeaders, respond)`，其中`add` headers 是 CORS 的头部，响应将返回我们创建的 XML 文档作为字符串：

```html
function respond(req, res, next) {
  res.setHeader('content-type', 'application/xml');
  res.send(doc.toString({ pretty: true }));
  return next();
}
```

1.  restify 不直接支持`application/xml`；如果我们保持这样，服务器的响应将是`application/octet-stream`类型。为了添加支持，我们将创建`restify`对象，并添加一个接受 XML 的格式化程序：

```html
var server = restify.createServer({
  formatters: {
   'application/xml': function formatXML(req, res, body) {
      if (body instanceof Error)
        return body.stack;

      if (Buffer.isBuffer(body))
        return body.toString('base64');

      return body;
    }
  }
});
```

服务器应该返回正确的`content-type`和 CORS 头部，以及响应数据：

```html
< HTTP/1.1 200 OK
< Access-Control-Allow-Origin: *
< Access-Control-Allow-Headers: X-Requested-With
< content-type: application/xml
< Date: Sat, 02 Feb 2013 13:08:20 GMT
< Connection: keep-alive
< Transfer-Encoding: chunked

```

1.  由于服务器已经准备好，我们可以继续在客户端创建一个基本的 HTML 文件，其中我们将包含 jQuery 和一个简单的脚本：

```html
   Hello <div id="humans"></div>
   <hr/>
  <script src="img/jquery.min.js">
  </script>
  <script src="img/exampleJQuery.js"></script>
```

1.  为简单起见，我们使用`jQuery.ajax()`，其中`dataType`的值将是`xml`：

```html
        (function(){
  $.ajax({
    type: "GET",
    url: "http://localhost:8080/hi",
    dataType: "xml",
    success: function(xml) {
      $("root > human", xml).each(function(){
        var p = $("<p></p>");
        $(p).text($(this).text()).appendTo("#humans");
      });
    }
  });
}())
```

## 它是如何工作的...

虽然大多数示例代码应该很简单，但你可能会想知道`application/octet-stream`是什么？嗯，它是一种通用二进制数据流的互联网媒体类型。如果我们用浏览器打开资源，它会要求我们保存它或者用什么应用程序打开它。

我们在`restify`实现中添加的`formatter`接受一个带有请求、响应和主体的函数。对我们来说，最感兴趣的是`body`对象；我们检查它是否是`Error`的实例，以便以某种方式处理它。需要进行的另一个检查是`body`是否是`Buffer`的实例。JavaScript 与二进制数据的处理不太好，因此创建了一个`Buffer`对象来存储原始数据。在我们的情况下，我们只返回主体，因为我们已经构建了 XML。如果我们经常进行这样的处理，直接为 JavaScript 对象添加格式化可能是有意义的，而不是手动创建包含 XML 数据的字符串。

在客户端，我们使用`jQuery.ajax()`来获取 XML，当这种情况发生时，`success`回调不仅接收文本，还接受一个 DOM 元素，我们可以使用标准的 jQuery 选择器来遍历。在我们的情况下，使用`"root> human"`，我们选择所有的`human`元素，并且对其中的文本，每个元素都向`"#humans"`添加一个段落，就像处理 HTML 一样：

```html
.   $("root > human", xml).each(function(){
        var p = $("<p></p>");
        $(p).text($(this).text()).appendTo("#humans");
      });  
```

## 还有更多...

JXON ([`developer.mozilla.org/en-US/docs/JXON`](https://developer.mozilla.org/en-US/docs/JXON))是在我们必须支持 XML 时的一个很好的选择。没有标准化，它遵循一个简单的约定，将 XML 转换为 JSON。在处理 XML 时，另一个很好的选择是使用 XPath——XML Path Language ([`www.w3.org/TR/xpath/`](http://www.w3.org/TR/xpath/))，这是一种查询语言，可用于从某些节点中检索值或选择它们进行其他操作。XPath 在大多数情况下是最简单的选择，因此通常应该是我们的首选。

旧版本的 jQuery（1.1.2 版本之前）默认支持 XPath，但后来被移除，因为标准选择器在进行 HTML 转换时更加强大。

ECMAScript for XML 或通常称为 E4X 是一种编程语言扩展，用于原生支持 XML。虽然最新版本的 Firefox 中有几种实现可用，但它正在被移除。

# 使用 FormData 接口

添加到`XMLHttpRequest` Level 2 ([`www.w3.org/TR/XMLHttpRequest2/`](http://www.w3.org/TR/XMLHttpRequest2/))中的新功能之一是`FormData`对象。这使我们能够使用一组可以使用 AJAX 发送的键值对。最常见的用途是发送二进制文件或任何其他大量的数据。在这个示例中，我们将创建两个脚本，一个将发送`FormData`，一个使用纯 JavaScript，另一个使用 jQuery，以及支持它的服务器端代码。

## 准备工作

服务器将在 Nodejs 中使用 restify ([`mcavage.github.io/node-restify/`](http://mcavage.github.io/node-restify/))完成。为了安装依赖项，可以创建一个`package.json`文件，其中将添加 restify。

## 如何做...

1.  服务器应该能够接受`HTTP POST`，类型为`multipart/form-data`；这就是为什么`restify`有一个内置的名为`BodyParser`的插件。这将阻止解析 HTTP 请求体：

```html
var server = restify.createServer();
server.use(restify.bodyParser({ mapParams: false }));
server.post('hi', addHeaders, doPost);
```

1.  这将切换内容类型，并根据内容类型执行适当的逻辑，如`application/json`，`application/x-ww-form-urlencoded`和`mutipart/form-data`。`addHeaders`参数将与我们在其他示例中添加的相同，以启用 CORS。为了简化我们的`doPost`处理程序，我们只记录请求体并返回 HTTP 200：

```html
function doPost(req, res, next) {
  console.log("Got HTTP " + req.method + " on " + req.url + " responding");
  console.log(req.body);
  res.send(200);
  return next();
}
```

1.  对于客户端，我们创建一个简单的脚本的 HTML 文件：

```html
(function (){
var myForm = new FormData();
myForm.append("username", "johndoe");
myForm.append("books", 7);
var xhr = new XMLHttpRequest();
xhr.open("POST", "http://localhost:8080/hi");
xhr.send(myForm);
  }());
```

1.  `jQuery`的方式要简单得多；我们可以将`FormData`设置为`jQuery.ajax()`中的`data`属性的一部分，此外，在发送之前我们需要禁用数据处理并保留原始内容类型：

```html
(function(){
  var formData = new FormData();
  formData.append("text", "some strange data");
  $.ajax({
    url: "http://localhost:8080/hi",
    type: "POST",
    data: formData,
    processData: false,  // don't process data
    contentType: false   // don't set contentType
  });
}());
```

## 它是如何工作的...

传输的数据将具有与提交具有`multipart/form-data`编码类型的表单相同的格式。这种编码的需求来自于将混合数据与文件一起发送。大多数 Web 浏览器和 Web 服务器都支持这种编码。这种编码可以用于不是 HTML 甚至不是浏览器的表单。

如果我们查看发送的请求，我们可以看到它包含以下数据：

```html
Content-Length:239
Content-Type:multipart/form-data; boundary=----WebKitFormBoundaryQXGzNXa82frwui6S
```

负载将如下所示：

```html
------WebKitFormBoundaryQXGzNXa82frwui6S
Content-Disposition: form-data; name="username"
johndoe
------WebKitFormBoundaryQXGzNXa82frwui6S
Content-Disposition: form-data; name="books"
7
------WebKitFormBoundaryQXGzNXa82frwui6S--
```

您可能注意到，每个部分都包含一个`Content-Disposition`部分，其中包含数据的来源或者在我们的情况下，我们在每个附加到`FormData`对象的键中设置的名称。还有一个选项可以在每个单独的部分上设置内容类型，例如，如果我们有一个来自名为`profileImage`的控件的图像，那么该部分可以如下所示：

```html
Content-Disposition: form-data; name="profileImage"; filename="me.png"
Content-Type: image/png
```

`example.js`中对`xhr.sent()`的最后一次调用在发送`FormData`类型的对象时会自动设置内容类型。

如果我们需要支持旧的不支持`XMLHttpRequest`级别 2 的传统浏览器，我们可以检查`FormData`是否存在，并相应地处理该情况：

```html
if (typeof FormData === "undefined")
```

我们作为后备使用的方法不能是一个 AJAX 调用，但这不应该是一个问题，因为所有现代浏览器 IE<10 版本都不支持它。

# 向服务器发送二进制文件

向服务器发送文本、XML 或 JSON 相对容易，大多数 JavaScript 库都针对这种情况进行了优化。

发送二进制数据稍微棘手。现代应用程序可能需要能够上传生成的二进制文件；例如，在 HTML5 画布上绘制的图像、使用 JSZip 创建的 ZIP 文件等。

此外，能够上传使用 HTML5 文件 API 选择的文件是非常方便的。我们可以做一些有趣的事情，比如通过将文件分割成较小的部分，并将每个部分单独上传到服务器来实现可恢复的文件上传。

在这个配方中，我们将使用文件输入来上传用户选择的文件。

## 准备工作

服务器将使用 Node.js 实现-您可以从[`nodejs.org/`](http://nodejs.org/)下载并安装 Node.js。服务器将使用 Node.js 框架**Connec** **t** ([`www.senchalabs.org/connect/`](http://www.senchalabs.org/connect/))实现。

## 操作步骤...

让我们编写客户端和服务器代码。

1.  创建一个名为`index.html`的文件，包括文件输入、上传按钮、进度条和消息容器的文件上传页面：

```html
<!DOCTYPE HTML>
<html>
<head>
<title>Upload binary file</title>
<style type="text/css">
.progress {
    position:relative;
    height:1em; width: 12em;
    border: solid 1px #aaa;
}
.progress div {
    position: absolute;
    top:0; bottom:0; left:0;
    background-color:#336699;
}
</style>
</head>
<body>
<input type="file"   id="file" value="Choose file">
<input type="button" id="upload" value="Upload"><br>
<p id="info"></p>
<div class="progress"><div id="progress"></div></div>
<script src="img/jquery.min.js"></script>
<script type="text/javascript" src="img/uploader.js"></script>
<script type="text/javascript" src="img/example.js"></script>
</body>
</html>
```

1.  创建一个名为`uploader.js`的文件，实现一个二进制文件上传器。它将文件发送到指定的 URL，并返回一个对象，用于绑定进度事件：

```html
window.postBinary = function(url, data) {
    var self = {},
        xhr = new XMLHttpRequest();
    xhr.open('POST', url, true);
    xhr.responseType = 'text';    
    self.done = function(cb) {
        xhr.addEventListener('load', function() {
            if (this.status == 200)
                cb(null, this.response)
            else
                cb(this.status, this.response)
        });
        return self;
    }
    self.progress = function(cb) {
        xhr.upload.addEventListener('progress', function(e) {
            if (e.lengthComputable) 
                cb(null, e.loaded / e.total);
            else
                cb('Progress not available');
        });
        return progress;
    };
    xhr.send(data);    
    return self;
};
```

1.  创建一个名为`example.js`的文件，使用`uploader.js`提供的 API 为上传表单添加上传功能：

```html
$(function() {
    var file;
    $("#file").on('change', function(e) {
        file = this.files[0]
    });
    $("#upload").on('click', function() {
        $("#info").text("Uploading...");
        $("#progress").css({width:0});
        if (!file) {
            $("#info").text('No file selected')
            return;
        }
        var upload =  postBinary('/upload/' + file.name, file);
        upload.progress(function(err, percent) {
            if (err) {
                $("#info").text(err);
                return;
            }
            $("#progress").css({width: percent + '%'});
        });
        upload.done(function(err, res) {
            if (err) {
                $("#info").text(err + ' ' + res);
                return;
            }
            $("#progress").css({width: '100%'});
            $("#info").text("Upload complete");
        });

    });
});
```

1.  创建一个名为`server.js`的文件，这是一个基于 Node.js Connect 框架的 Node.js 服务器，用于处理文件上传和提供静态文件：

```html
var path = require('path'),
    connect = require('connect'),
    fs = require('fs');

connect()
    .use('/upload', function(req, res) {        
        var file = fs.createWriteStream(
            path.join(__dirname, 'uploads', req.url))
        req.pipe(file);
        req.on('end', function() {
            res.end("ok");
        });
    })
    .use(connect.static(__dirname))
    .listen(8080);
```

1.  从存放`server.js`的目录打开命令提示符，并输入以下命令来创建一个用于上传的目录，安装 connect 库，并启动服务器：

```html
mkdir uploads
npm install connect
node server.js

```

1.  将浏览器导航到`http://localhost:8080`以测试示例。所有创建的文件（包括`server.js`）应该在同一个目录中。

## 工作原理...

HTML5 中的新`XMLHttpRequest`对象具有支持更多类型数据的`send`方法。它可以接受`File`、`Blob`和`ArrayBuffer`对象。我们将这种新功能与 HTML5 文件 API 一起使用，以上传用户选择的文件。您可以在第四章的*在客户端使用文件输入*配方中找到有关此 API 的更多信息，*使用 HTML5 输入组件*。

新 API 还提供了一个`upload`对象，类型为`XMLHttpRequestUpload`。它允许我们附加事件监听器来监视上传进度。我们使用这个功能来显示上传的进度条。

服务器接受`'/upload'`处的上传，并将文件保存到`uploads`目录。此外，它还提供示例目录中的静态文件。

## 还有更多...

新的 XHR API 仅适用于 Internet Explorer 10 及以上版本。

有些浏览器可能无法触发上传进度事件。

# 使用 Node.js 创建 SSL 连接

常见的安全问题是所谓的中间人攻击，这是一种窃听的形式，攻击者会独立连接到受害者并将消息转发到所需的位置。攻击者必须能够拦截消息并自行更改。只有在攻击者能够成功冒充两个涉及方时，这才有可能。**安全套接字层**（**SSL**）及其后继者**传输层安全**（**TSL**）通过加密数据来防止这种类型的攻击。在这个示例中，我们创建一个使用 restify 的 Node.js 服务器，该服务器支持 HTTPS。

## 准备就绪

为了启用 HTTPS，我们将使用证书和服务器私钥。为了生成这个，我们需要 OpenSSL（[`www.openssl.org/`](http://www.openssl.org/)），这是一个完整的开源工具包，实现 SSL 和 TLS，以及一个通用的密码库。

首先，在命令行上，生成一个 RSA（[`en.wikipedia.org/wiki/RSA_(algorithm)`](http://en.wikipedia.org/wiki/RSA_(algorithm)）私钥：

```html
 openssl genrsa -out privatekey.pem 1024

```

![准备就绪](img/9282OT_08_01.jpg)

将生成的实际密钥应该类似于以下内容：

![准备就绪](img/9282OT_08_02.jpg)

你生成的应该要长得多。

### 注意

请注意，私钥之所以称为*私钥*是有原因的，你不应该将其放在任何版本控制系统中，也不应该让所有人都可以访问。这应该保持安全，因为这是你的真实身份证明。

接下来，我们将使用刚刚创建的私钥创建一个**证书签名请求**（**CSR**）文件，并输入一些额外的信息：

```html
openssl req -new -key privatekey.pem -out csr.pem

```

填写表格后，我们生成了一个 CSR 文件，用于向证书颁发机构请求签署您的证书。这个文件可以发送给他们进行处理，他们会给我们一个证书。由于我们只是创建一个简单的示例，我们将使用我们的私钥自签名文件：

```html
openssl x509 -req -in csr.pem -signkey privatekey.pem -out publiccert.pem

```

`publiccert.pem`文件是我们将在服务器上用作证书的文件。

## 如何做...

1.  首先，我们添加依赖项，然后创建一个`options`对象，从中读取我们生成的密钥和证书：

```html
var restify = require('restify');
var fs = require('fs');
// create option for the https server instance
var httpsOptions = {
  key: fs.readFileSync('privatekey.pem'),//private key
  certificate: fs.readFileSync('publiccert.pem')//certificate
};
```

### 注意

Node.js 中的文件 IO 是使用`fs`模块提供的。这是标准 POSIX 功能的包装器。可以在[`nodejs.org/api/fs.html`](http://nodejs.org/api/fs.html)上找到有关它的文档。

1.  我们继续创建路由和处理程序，并为了不重复两个服务器实例的逻辑，我们创建一个通用的`serverCreate`函数：

```html
var serverCreate = function(app) {
  function doHi(req, res, next) {
    var name = 'nobody';
    if(req.params.name){
      name = req.params.name;
    }
    res.send('Hi ' + name);
    return next();
  }
  app.get('/hi/', doHi);
  app.get('/hi/:name', doHi);
}
```

1.  然后我们可以使用这个函数来创建两个服务器的实例：

```html
serverCreate(server);
serverCreate(httpsServer);
```

1.  我们可以将标准服务器设置为监听端口`80`，HTTPS 版本设置为端口`443`：

```html
server.listen(80, function() {
  console.log('started at %s', server.url);
});

httpsServer.listen(443, function() {
  console.log('started at %s', httpsServer.url);
});
```

1.  现在我们可以调用`node server.js`来启动服务器，并尝试从浏览器访问以下页面：

+   `http://localhost:80/hi/John`

+   `http://localhost:443/hi/UncleSam`

## 工作原理...

当运行服务器时，你可能会遇到以下类似的错误：

```html
Error: listen EACCES
 at errnoException (net.js:770:11)
 at Server._listen2 (net.js:893:19)

```

问题在于服务器本身无法绑定到小于 1024 的端口，除非具有 root 或管理员权限（众所周知）。

我们刚刚创建的 HTTPS 服务器使用了公钥密码学。每个对等方都有两个密钥：一个公钥和一个私钥。

### 注意

在密码学中，通常涉及的各方被称为 Alice 和 Bob，因此我们将使用相同的名称。关于这个主题的更多信息可以在维基百科上找到[`en.wikipedia.org/wiki/Alice_and_Bob`](http://en.wikipedia.org/wiki/Alice_and_Bob)。

Alice 和 Bob 的公钥与所有人共享，而他们的私钥则保密。为了让 Alice 加密她需要发送给 Bob 的消息，她需要 Bob 的公钥和她自己的私钥。另一方面，如果 Bob 需要解密他从 Alice 那里收到的相同消息，他需要她的公钥和他自己的私钥。

在 TLS 连接中，公钥是证书。这是因为它是签名的，以证明真正的所有者是他们声称的人；例如 Bob。TSL 证书可以由一个实际确认 Bob 是他所声称的人的证书颁发机构签名。Firefox、Chrome 和其他浏览器有一个受信任的用于签发证书的根 CA 列表。这个根 CA 可能会向其他签名机构签发证书，然后将它们出售给普通公众；这是一个非常有趣的业务，你不觉得吗？

在我们的情况下，我们自签了我们的证书，所以它不被浏览器信任，当我们打开它时，我们会得到以下可爱的小页面：

![它是如何工作的...](img/9282OT_08_03.jpg)

当我们使用 CA 签名的证书时，这条消息将不会出现，因为我们将拥有一个被我们的浏览器认可为受信任的权威。

## 还有更多内容...

开放 Web 应用安全项目，或 OWASP ([`www.owasp.org/`](https://www.owasp.org/))，在创建 Web 应用程序时，有一个关于常见安全问题和陷阱的全面数据库。在那里，您可以找到有关 HTML5 应用程序安全的很棒的安全速查表([`www.owasp.org/index.php/HTML5_Security_Cheat_Sheet`](https://www.owasp.org/index.php/HTML5_Security_Cheat_Sheet))。在涉及 HTTPS 时，一个常见的问题是存在不总是来自相同协议的混合内容。增加安全性的一个简单方法是将每个请求都发送到 TLS/SSL。

# 使用 Ajax Push 进行实时更新

Comet 是一种 Web 模型，其中长时间保持的 HTTP 请求允许服务器向浏览器“推送”数据，而无需浏览器明确发出请求。Comet 以许多不同的名称而闻名，如 Ajax Push，Server Push，Reverse Ajax 双向 Web 等。在这个示例中，我们将创建一个简单的服务器，向客户端发送或“推送”当前时间。

## 准备工作

在这个示例中，我们将使用 Node.js 和一个名为**Socket.IO**的库([`socket.io/`](http://socket.io/))。这个依赖项可以包含在`package.json`文件中，也可以直接从 npm 安装。

## 操作方法...

让我们开始吧。

1.  首先，我们将从服务器端开始，我们将添加所需的 Socket.IO、HTTP 和文件系统的`require`语句：

```html
var app = require('http').createServer(requestHandler),
    io = require('socket.io').listen(app),
    fs = require('fs')
```

1.  服务器初始化为`requestHandler`，我们将在其中提供一个位于同一目录中的`index.html`文件，稍后我们将创建：

```html
function requestHandler (req, res) {
  fs.readFile('index.html',
    function (err, data) {
      if (err) {
        res.writeHead(500);
        return res.end('Error loading index.html');
      }
    res.writeHead(200);
    res.end(data);
    });
}
```

1.  如果无法读取文件，它将返回 HTTP 500，如果一切正常，它将返回数据，这是一个非常简化的处理程序。我们将服务器设置为侦听端口 80，然后我们可以继续进行与 Socket.IO 相关的配置：

```html
io.configure(function () {
  io.set("transports", ["xhr-polling"]);
  io.set("polling duration", 10);
});
```

在这里，我们将唯一允许的传输设置为`xhr-polling`，以便进行示例。Socket.IO 支持多种不同的向客户端发送服务器端事件的方式，因此我们禁用了其他所有内容。

### 注意

请注意，在实际应用中，您可能会希望保留其他传输方法，因为它们可能是给定客户端的更好选择，或者作为备用机制。

1.  之后，我们可以继续进行事件。在每次连接时，我们向客户端发出一个带有一些 JSON 数据的`ping`事件，第一次，每当收到`pong`事件时，我们等待 15 秒，然后再次发送一些带有当前服务器时间的 JSON 数据：

```html
io.sockets.on('connection', function (socket) {
  socket.emit('ping', {
    timeIs: new Date()
  });
  socket.on('pong', function (data) {
    setTimeout(function(){
    socket.emit('ping', {
      timeIs: new Date()
    });
    console.log(data);
    }, 15000);
  });
});
```

1.  现在在客户端，我们将包含`socket.io.js`文件，由于我们正在从 node 提供我们的`index.html`文件，它将被添加到以下默认路径：

```html
  <script src="img/socket.io.js"></script>
```

1.  之后，我们连接到`localhost`并等待`ping`事件，每次收到这样的事件时，我们都会附加一个带有服务器时间的`p`元素。然后我们向服务器发出`pong`事件：

```html
    <script>
      var socket = io.connect('http://localhost');
      socket.on('ping', function (data) {
        var p = document.createElement("p");
        p.textContent = 'Server time is ' + data.timeIs;
        document.body.appendChild(p);
        socket.emit('pong', {
          my: 'clientData'
        });
      });
    </script>
```

现在当我们启动服务器并通过打开`http://localhost`访问`index.html`时，我们应该可以在没有明确请求的情况下获得服务器更新：

```html
Server time is 2013-02-05T06:14:33.052Z
```

## 它是如何工作的...

如果我们不将唯一的传输方法设置为 Ajax 轮询或 xhr-polling，Socket.IO 将尝试使用最佳可用方法。目前，支持几种传输：WebSocket、Adobe Flash Socket、AJAX 长轮询、AJAX 多部分流式传输、Forever IFrame 和 JSONP 轮询。

根据使用的浏览器，不同的方法可能更好、更差或不可用，但可以肯定的是 WebSockets 是未来。长轮询在浏览器端更容易实现，并且适用于支持`XMLHttpRequest`的每个浏览器。

顾名思义，长轮询是指客户端请求服务器事件。这个请求保持打开状态，直到服务器向浏览器发送了一些新数据或关闭了连接。

如果我们在我们的示例中打开控制台，我们可以看到向服务器发送了一个请求，但由于响应尚未完成，它没有关闭：

| hOC6eXNTrdIhwO9aHcqX?t=1360049439710/socket.io/1/xhr-polling | GET | (pending) |
| --- | --- | --- |

由于我们将服务器轮询持续时间设置为 10 秒，使用`io.set("polling duration", 10)`，这个连接将被关闭，然后重新打开。您可能会想知道为什么我们需要关闭连接？如果不关闭，服务器上的资源将很容易被耗尽。

您可能会注意到服务器控制台中的关闭和数据发送：

```html
   debug - xhr-polling received data packet 5:::{"name":"pong","args":[{"my":"clientData"}]}
   debug - setting request GET /socket.io/1/xhr-polling/5jBJdDQ6Uc2ZYXzZHcqd?t=1360050667340
   debug - setting poll timeout
   debug - discarding transport
```

还有一件事需要注意的是，一旦连接关闭，无论是因为收到响应还是因为服务器端超时，都会创建一个新连接。新创建的请求通常会有一个等待它的服务器连接，从而显著减少延迟。

## 还有更多...

Socket.IO 还有许多其他功能，我们没有涉及。其中之一是向所有连接的客户端广播消息。例如，为了让每个人都知道有新用户连接，我们可以这样做：

```html
  io.sockets.on('connection', function (soc) {
  soc.broadcast.emit('user connected');
});
```

即使我们不使用 Node.js，大多数编程语言都可以使用彗星技术或“黑客技术”，这是改善用户体验的好方法。

# 使用 WebSockets 交换实时消息

在 HTML5 Web Sockets 之前，需要实现实时更新的 Web 应用程序，如聊天消息和游戏移动，必须采用低效的方法。

最流行的方法是使用长轮询，即保持与服务器的连接直到事件到达。另一种流行的方法是将 JavaScript 的分块流式传输到`iframe`元素，也被称为**彗星流**。

HTML5 WebSockets 使得可以与 Web 服务器交换实时消息。该 API 更清洁、更易于使用，错误更少，并且提供更低的消息延迟。

在这个示例中，我们将实现一个基于 WebSockets 的简单聊天系统。为了使系统更易于扩展，我们将在底层 WebSockets 上使用 dnode。dnode 库为多种语言和平台提供了完整的基于回调的 RPC：Node.js、Ruby、Java 和 Perl。基本上，它使我们能够调用服务器端代码，就好像它在客户端执行一样。

## 准备工作

服务器将使用 Node.js 实现——您可以从[`nodejs.org/`](http://nodejs.org/)下载并安装 Node.js。

为了做好准备，您还需要安装一些 node 模块。为该示例创建一个新目录，并输入以下命令以安装 node 模块：

```html
npm install -g browserify
npm install express shoe dnode

```

## 如何做到这一点...

让我们来编写客户端和服务器。

1.  在`index.html`中创建包含消息列表、用户列表和文本输入框的主要聊天页面。聊天页面的样式填满整个浏览器视口。

```html
<!DOCTYPE HTML>
<html>
<head>
<title>Using websockets</title>
<style type="text/css">
#chat { position: absolute; overflow: auto;
    top:0; left:0; bottom:2em; right:12em; }
#users { position: absolute; overflow: auto;
    top:0; right: 0; width:12em; bottom: 0; }
#input { position: absolute; overflow: auto;
    bottom:0; height:2em; left: 0; right: 12em; }

#chat .name { padding-right:1em; font-weight:bold; }
#chat .msg { padding: 0.33em; }
</style>
</head>
<body>
<div id="chat">
</div>
<div id="users">
</div>
<input type="text" id="input">
<script src="img/jquery.min.js"></script>
<script type="text/javascript" src="img/example.min.js"></script>
</body>
</html>
```

1.  创建一个名为`chat.js`的文件——JavaScript 中的聊天室实现。`chat()`函数创建一个聊天室，并返回`chatroom`的公共 API，包括`join`、`leave`、`msg`、`ping`和`listen`函数。

```html
function keysOf(obj) {
    var k = [];
    for (var key in obj)
        if (obj.hasOwnProperty(key))
            k.push(key);
    return k;
}
function chat() {
    var self = {},
        users = {},
        messages = [];

    // Identify the user by comparing the data provided
    // for identification with the data stored server-side
    function identify(user) {
        return users[user.name] && user.token
            == users[user.name].token;
    }
    // Send an event to all connected chat users that
    // are listening for events
    function emit(event) {
        console.log(event);
        for (var key in users) if (users.hasOwnProperty(key))
            if (users[key].send) users[key].send(event);
    }
    // This function resets the timeout countdown for a
    // specified user. The countdown is reset on every user
    // action and every time the browser sends a ping
    // If the countdown expires, the user is considered
    // to have closed the browser window and no longer present
    function resetTimeout(user) {
        if (user.timeout) {
            clearTimeout(user.timeout);
            user.timeout = null;
        }
        user.timeout = setTimeout(function() {
            self.leave(user, function() {});
        }, 60000);
    }

    // When a user attempts to join, he must reserve a
    // unique name. If this succeeds, he is given an auth
    // token along with the name. Only actions performed
    // using this token will be accepted as coming from
    // the user. After the user joins a list of users and
    // past messages are sent to him along with the
    // authentication information.
    self.join = function(name, cb) {
        if (users[name]) return cb(name + " is in use");
        users[name] = {
            name: name,
            token: Math.round(Math.random() * Math.pow(2, 30))
        }
        resetTimeout(users[name]);
        emit({type: 'join', name: name});
        cb(null, { you: users[name], messages: messages,
           users: keysOf(users) });
    }
    // The leave function is called when the user leaves
    // after closing the browser window.
    self.leave = function(user, cb) {
        if (!identify(user)) return
        clearTimeout(users[user.name].timeout);
        delete users[user.name];
        emit({type: 'leave', name: user.name});
        cb(null);
    }
    // The message function allows the user to send a
    // message. The message is saved with a timestamp
    // then sent to all users as an event.
    self.msg = function(user, text) {
        if (!identify(user)) return;
        resetTimeout(users[user.name]);
        var msg = {
            type: 'msg',
            name: user.name,
            text: text,
            time: Date.now()
        }
        messages.push(msg);
        emit(msg);
    }
    // The ping function allows the browser to reset
    // the timeout. It lets the server know that the
    // user hasn't closed the chat yet.
    self.ping = function(user) {
        if (identify(user))
            resetTimeout(users[user.name]);
    }
    // The listen function allows the user to provide
    // a callback function to be called for every event.
    // This way the server can call client-side code.
    self.listen = function(user, send, cb) {
        if (!identify(user)) return
        users[user.name].send = send;
    }
    return self;
};
module.exports = chat;
```

1.  让我们创建名为`server.js`的 Node.js 脚本，实现 web 服务器：

```html
var express = require('express'),
    http    = require('http'),
    chat    = require('./chat.js'),
    shoe    = require('shoe'),
    dnode   = require('dnode')
// Create an express app
var app = express();
// that serves the static files in this directory
app.use('/', express.static(__dirname));
// then create a web server with this app
var server = http.createServer(app);
// Create a chat room instance,
var room = chat();
// then create a websocket stream that
// provides the chat room API via dnode
// and install that stream on the http server
// at the address /chat
shoe(function (stream) {
    var d = dnode(room);
    d.pipe(stream).pipe(d);
}).install(server, '/chat');
// start the server
server.listen(8080);
```

1.  创建一个名为`example.js`的文件来实现聊天客户端：

```html
var shoe = require('shoe'),
    dnode = require('dnode');

$(function() {

    // Add a message to the message div
    function addMsg(msg) {
        var dMsg = $("<div />").addClass('msg'),
            dName = $("<span />").addClass('name')
                .text(msg.name).appendTo(dMsg),
            dText = $("<span />").addClass('text')
                .text(msg.text).appendTo(dMsg);
        dMsg.appendTo("#chat");
        $("#chat").scrollTop($("#chat")[0].scrollHeight);
    }

    // Re-display a list of the present users.
    function showUsers(users) {
        $("#users").html('');
        users.forEach(function(name) {
            $("<div />").addClass('user')
                .text(name).appendTo('#users');
        });
    }

    // Create a client-side web sockets stream
    // piped to a dnode instance
    var stream = shoe('/chat');
    var d = dnode();
    // When the remote chat API becomes available
    d.on('remote', function (chat) {
        // Attempt to join the room until a suitable
        // nickname that is not already in use is found
        function join(cb, msg) {
            var name = prompt(msg || "Enter a name");
            chat.join(name, function(err, data) {
                if (err) join(cb, err);
                else cb(data);
            });
        }
        join(function(data) {
            var me = data.you,
                users = data.users;
            // Show the users and messages after joining
            showUsers(users);
            data.messages.forEach(addMsg);
            // Allow the user to send messages
            $("#input").on('keydown', function(e) {
                if (e.keyCode == 13) {
                    // sending works by calling the
                    // remote's msg function.
                    chat.msg(me, $(this).val());
                    $(this).val('');
                }

            });
            // Tell the remote we're listening for
            // events
            chat.listen(me, function(e) {
                if (e.type == 'msg')
                    return addMsg(e);
                if (e.type == 'leave')
                    delete users[users.indexOf(e.name)];
                else if (e.type == 'join')
                    users.push(e.name);
                showUsers(users);
            });
            // Tell the remote every 30 seconds that
            // we're still active
            setInterval(function() {
                chat.ping(me);
            }, 30000);

        });
    });
    // pipe dnode messages to the websocket stream
    // and messages from the stream to dnode
    d.pipe(stream).pipe(d);
});
```

1.  使用`browserify`创建`example.min.js`：

```html
browserify example.js –-debug -o example.min.js

```

1.  启动 node 服务器：

```html
node server.js

```

1.  将浏览器导航到[`localhost:8080`](http://localhost:8080)来测试示例。

## 它是如何工作的...

我们在这里没有直接使用 WebSockets API。原因是，使用原始的 WebSockets 发送响应消息并不是很容易——它们不支持请求-响应循环。因此，要实现一些 RPC 调用，比如询问服务器名称是否可用，将会更加困难。

另一方面，dnode 协议支持将本地回调传递给远程函数，远程函数反过来可以将自己的回调传递给接收到的回调，依此类推——从而实现一个非常强大的完整 RPC。这使我们能够扩展我们的应用程序，以满足新的需求。作为一个奖励，结果 API 更加清晰和表达力更强。

这是我们使用 dnode 实现聊天室的步骤：

1.  我们创建了一个简单的对象，使用延续传递风格来返回所有函数的错误和值。这是我们的聊天室对象，定义了应用程序的 RPC API。

1.  我们定义了一个基于`shoe`库的 WebSockets 服务器，为每个连接的客户端创建一个新的 Node.js 流。然后将其安装到`/chat`路由的常规 HTTP 服务器上。

1.  我们通过将每个连接的客户端流传输到基于聊天室对象的新创建的 dnode 流来连接这两者。

就是这样！然后，在客户端使用 API，做如下操作：

1.  我们定义了一个基于`shoe`库的 WebSockets 客户端，它连接到`/chat`路由的 HTTP 服务器，并在建立连接时创建一个新的 Node.js 流。

1.  我们将该流传输到一个新创建的 dnode 客户端。

1.  建立连接后，dnode 客户端接收到一个包含第 1 步中定义的 API 的对象——所有函数都可用。

### 注意

在[`github.com/substack/dnode`](https://github.com/substack/dnode)了解更多关于 dnode 的信息。

截至 2013 年 2 月，IE 9 及以下版本不支持 WebSockets API。最新版本的 Android（v 4.2）内置浏览器也不支持 WebSockets API。
