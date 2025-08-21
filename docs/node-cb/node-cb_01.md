# 第一章：创建 Web 服务器

在本章中，我们将涵盖：

+   设置路由

+   提供静态文件

+   在内存中缓存内容以立即提供

+   使用流优化性能

+   防止文件系统黑客攻击

# 介绍

Node 的一个伟大特点是它的简单性。与 PHP 或 ASP 不同，它没有将 web 服务器和代码分开，也不需要定制大型配置文件来获得我们想要的行为。使用 Node，我们可以创建服务器，自定义它，并在代码级别提供内容。本章演示了如何使用 Node 创建 web 服务器，并通过它提供内容，同时实现安全性和性能增强以满足各种情况。

# 设置路由

为了提供 web 内容，我们需要使 URI 可用。本教程将指导我们创建一个公开路由的 HTTP 服务器。

## 准备工作

首先，让我们创建我们的服务器文件。如果我们的主要目的是公开服务器功能，通常的做法是将文件命名为`server.js`，然后将其放在一个新文件夹中。安装和使用`hotnode`也是一个好主意：

```js
sudo npm -g install hotnode
hotnode server.js

```

当我们保存更改时，`hotnode`将方便地自动重新启动服务器。

## 如何做...

为了创建服务器，我们需要`http`模块，所以让我们加载它并使用`http.createServer`方法：

```js
	var http = require('http');
	http.createServer(function (request, response) {
	response.writeHead(200, {'Content-Type': 'text/html'});
	response.end('Woohoo!');
	}).listen(8080);

```

### 提示

**下载示例代码**

您可以从您在[`www.PacktPub.com`](http://www.PacktPub.com)的帐户中下载您购买的所有 Packt 图书的示例代码文件。如果您在其他地方购买了本书，您可以访问[`www.PacktPub.com/support`](http://www.PacktPub.com/support)并注册，以便直接通过电子邮件接收文件。

现在，如果我们保存我们的文件并在 web 浏览器上或使用 curl 访问`localhost:8080`，我们的浏览器（或 curl）将会呼喊：`'Woohoo!'`。然而，在`localhost:8080/foo`上也会发生同样的事情。实际上，任何路径都会产生相同的行为，因此让我们构建一些路由。我们可以使用`path`模块提取路径的`basename`（路径的最后一部分），并使用`decodeURI`从客户端反转任何 URI 编码：

```js
	var http = require('http');
	var path = require('path'); 
	http.createServer(function (request, response) {
	var lookup = path.basename(decodeURI(request.url)); 

```

现在我们需要一种定义路由的方法。一种选择是使用对象数组：

```js
	var pages = [
	  {route: '', output: 'Woohoo!'},
	  {route: 'about', output: 'A simple routing with Node example'},
	  {route: 'another page', output: function() {return 'Here\'s '+this.route;}},
	];

```

我们的`pages`数组应该放在`http.createServer`调用之上。

在我们的服务器内部，我们需要循环遍历我们的数组，并查看查找变量是否与我们的路由中的任何一个匹配。如果匹配，我们可以提供输出。我们还将实现一些`404`处理：

```js
	http.createServer(function (request, response) {
	  var lookup=path.basename(decodeURI(request.url));
	  pages.forEach(function(page) {
	    if (page.route === lookup) {
	      response.writeHead(200, {'Content-Type': 'text/html'});
	      response.end(typeof page.output === 'function' 
	                   ? page.output() : page.output);
	    }
	  });
	  if (!response.finished) {
	     response.writeHead(404);
	     response.end('Page Not Found!');
	  }
	}).listen(8080);

```

## 工作原理...

我们提供给`http.createServer`的回调函数为我们提供了通过`request`和`response`对象与服务器进行交互所需的所有功能。我们使用`request`来获取请求的 URL，然后我们使用`path`获取它的`basename`。我们还使用`decodeURI`，如果没有它，我们的`another page`路由将失败，因为我们的代码将尝试将`another%20page`与我们的`pages`数组进行匹配并返回`false`。

一旦我们有了`basename`，我们可以以任何我们想要的方式进行匹配。我们可以将其发送到数据库查询以检索内容，使用正则表达式进行部分匹配，或者将其与文件名匹配并加载其内容。

我们本可以使用`switch`语句来处理路由，但我们的`pages`数组有几个优点。它更容易阅读和扩展，并且可以无缝转换为 JSON。我们使用`forEach`循环遍历我们的`pages`数组。

Node 是建立在谷歌的 V8 引擎上的，它为我们提供了许多 ECMAScript 5 功能。这些功能不能在所有浏览器中使用，因为它们尚未普遍实现，但在 Node 中使用它们没有问题！`forEach`是 ES5 的实现，但 ES3 的方法是使用不太方便的`for`循环。

在循环遍历每个对象时，我们检查它的`route`属性。如果我们找到匹配，我们将写入`200 OK`状态和`content-type`头。然后我们用对象的输出属性结束响应。

`response.end`允许我们向其传递参数，在完成响应之前写入。在`response.end`中，我们使用了一个三元运算符（?:）来有条件地调用`page.output`作为函数或简单地将其作为字符串传递。请注意，`another page`路由包含一个函数而不是一个字符串。该函数通过`this`变量可以访问其父对象，并允许更灵活地组装我们想要提供的输出。如果在我们的`forEach`循环中没有匹配，`response.end`将永远不会被调用。因此，客户端将继续等待响应，直到超时。为了避免这种情况，我们检查`response.finished`属性，如果为 false，我们写入一个`404`头并结束响应。

`response.finished`取决于`forEach`回调，但它并不嵌套在回调内部。回调函数主要用于异步操作。因此，表面上看起来像是潜在的竞争条件，但`forEach`并不是异步操作。它会继续阻塞，直到所有循环完成。

## 还有更多...

有许多方法可以扩展和修改这个示例。还有一些非核心模块可供我们使用。

### 简单多级路由

到目前为止，我们的路由只处理单级路径。多级路径（例如，`/about/node`）将简单地返回`404`。我们可以修改我们的对象以反映子目录结构，删除`path`，并使用`request.url`而不是`path.basename`来作为我们的路由。

```js
	var http=require('http');
	var pages = [
	  {route: '/', output: 'Woohoo!'},
	  {route: '/about/this', output: 'Multilevel routing with Node'},
	  {route: '/about/node', output: 'Evented I/O for V8 JavaScript.'},
	  {route: '/another page', output: function () {return 'Here\'s ' + this.route; }}
	];
	http.createServer(function (request, response) {
	  var lookup = decodeURI(request.url);

```

### 注意

在提供静态文件时，必须在获取给定文件之前清理`request.url`。请查看本章中讨论的*防止文件系统黑客攻击*部分。

多级路由可以进一步进行，允许我们构建然后遍历一个更复杂的对象。

```js
	{route: 'about', childRoutes: [
	  {route: 'node', output: 'Evented I/O for V8 Javascript'},
	  {route: 'this', output: 'Complex Multilevel Example'}
	]}

```

在第三或第四级之后，查看这个对象将变得非常庞大。我们可以创建一个辅助函数来定义我们的路由，从而为我们拼接对象。或者，我们可以使用开源 Node 社区提供的出色的非核心路由模块之一。已经存在出色的解决方案，提供了帮助方法来处理可扩展多级路由的不断增加的复杂性（请参阅本章和第六章中讨论的*路由模块*，*使用 Express 加速开发*）。

### 解析查询字符串

另外两个有用的核心模块是`url`和`querystring`。`url.parse`方法允许两个参数。首先是 URL 字符串（在我们的情况下，这将是`request.url`），第二个是名为`parseQueryString`的布尔参数。如果设置为`true`，它会延迟加载`querystring`模块，省去了我们需要要求它来解析查询为对象。这使我们可以轻松地与 URL 的查询部分交互。

```js
	var http = require('http');
	var url = require('url');
	var pages = [
		{id: '1', route: '', output: 'Woohoo!'},
		{id: '2', route: 'about', output: 'A simple routing with Node example'},
		{id: '3', route: 'another page', output: function () {return 'Here\'s ' + this.route; }},
	];
	http.createServer(function (request, response) {
		var id = url.parse(decodeURI(request.url), true).query.id;
	if (id) {
		pages.forEach(function (page) {
			if (page.id === id) {
				response.writeHead(200, {'Content-Type': 'text/html'});
				response.end(typeof page.output === 'function'
					? page.output() : page.output);
			}
		});
	}
	if (!response.finished) {
		response.writeHead(404);
		response.end('Page Not Found');
	}
}).listen(8080);

```

通过添加`id`属性，我们可以通过`localhost:8080?id=2`等方式访问我们的对象数据。

### 路由模块

有关 Node 的各种路由模块的最新列表，请访问[`www.github.com/joyent/node/wiki/modules#wiki-web-frameworks-routers`](https://www.github.com/joyent/node/wiki/modules#wiki-web-frameworks-routers)。这些由社区制作的路由器适用于各种场景。在将其引入生产环境之前，重要的是要研究模块的活动和成熟度。在第六章中，*使用 Express 加速开发*，我们将更详细地讨论使用内置的 Express/Connect 路由器来实现更全面的路由解决方案。

## 另请参阅

+   本章中讨论的*提供静态文件*和*防止文件系统黑客攻击*。

+   在第六章中讨论的*动态路由*。

# 提供静态文件

如果我们在磁盘上存储了要作为 Web 内容提供的信息，我们可以使用`fs`（文件系统）模块加载我们的内容并通过`createServer`回调传递。这是提供静态文件的基本概念起点。正如我们将在接下来的示例中学到的，还有更高效的解决方案。

## 准备工作

我们需要一些要提供的文件。让我们创建一个名为`content`的目录，其中包含以下三个文件：

`index.html:`

```js
	<html>
	<head>
	<title>Yay Node!</title>
	<link rel=stylesheet href=styles.css type=text/css>
	<script src=script.js type=text/javascript></script>
	</head>
	<body>
	<span id=yay>Yay!</span>
	</body>
	</html>

```

`script.js:`

```js
window.onload=function() {alert('Yay Node!');};

```

`styles.css:`

```js
#yay {font-size:5em;background:blue;color:yellow;padding:0.5em}

```

## 操作步骤...

与之前的示例一样，我们将使用核心模块`http`和`path`。我们还需要访问文件系统，因此我们也需要`fs`模块。让我们创建我们的服务器：

```js
	var http = require('http');
	var path = require('path');
	var fs = require('fs');
	http.createServer(function (request, response) {
	  var lookup = path.basename(decodeURI(request.url)) || 'index.html',
	    f = 'content/' + lookup;
	  path.exists(f, function (exists) {
	    console.log(exists ? lookup + " is there" : lookup + " doesn't exist");
	  });
	}).listen(8080);

```

如果我们还没有，我们可以初始化我们的`server.js`文件：

```js
 hotnode server.js 

```

尝试加载`localhost:8080/foo`，控制台将显示`foo 不存在`，因为它确实不存在。`localhost:8080/script.js`将告诉我们`script.js 存在`，因为它确实存在。在保存文件之前，我们应该让客户端知道`content-type`，我们可以从文件扩展名中确定。因此，让我们使用对象快速创建一个映射：

```js
	var mimeTypes = {
	  '.js' : 'text/javascript',
	  '.html': 'text/html',
	  '.css' : 'text/css'
	};

```

我们以后可以扩展我们的`mimeTypes`映射以支持更多类型。

### 注意

现代浏览器可能能够解释某些 MIME 类型（例如`text/javascript`）而无需服务器发送`content-type`头。然而，旧版浏览器或较少使用的 MIME 类型将依赖服务器发送正确的`content-type`头。

请记住，将`mimeTypes`放在服务器回调之外，因为我们不希望在每个客户端请求上初始化相同的对象。如果请求的文件存在，我们可以通过将`path.extname`传递给`mimeTypes`，然后将我们检索到的`content-type`传递给`response.writeHead`来将我们的文件扩展名转换为`content-type`。如果请求的文件不存在，我们将写出`404`并结束响应。

```js
	//requires variables, mimeType object...
	http.createServer(function (request, response) {
		var lookup = path.basename(decodeURI(request.url)) || 'index.html',
			f = 'content/' + lookup;
		fs.exists(f, function (exists) {
			if (exists) {
				fs.readFile(f, function (err, data) {
					if (err) { response.writeHead(500);
						response.end('Server Error!'); return; }
					var headers = {'Content-type': mimeTypes[path. extname(lookup)]};
					response.writeHead(200, headers);
					response.end(data);
				});
				return;
			}
			response.writeHead(404); //no such file found!
			response.end();
		});
}).listen(8080);

```

目前，仍然没有内容发送到客户端。我们必须从我们的文件中获取这些内容，因此我们将响应处理包装在`fs.readFile`方法的回调中。

```js
	//http.createServer, inside path.exists:
	if (exists) {
	  fs.readFile(f, function(err, data) {
	    var headers={'Content-type': mimeTypes[path.extname(lookup)]};
	    response.writeHead(200, headers);
	    response.end(data);
	  });
	 return;
	}

```

在我们完成之前，让我们对我们的`fs.readFile`回调应用一些错误处理，如下所示：

```js
	//requires variables, mimeType object...
	//http.createServer,  path exists, inside if(exists):  
	fs.readFile(f, function(err, data) {
	    if (err) {response.writeHead(500); response.end('Server Error!');  return; }
	    var headers = {'Content-type': mimeTypes[path.extname(lookup)]};
	    response.writeHead(200, headers);
	    response.end(data);            
	  });
	 return;
	}

```

请注意，`return`保持在`fs.readFile`回调之外。我们从`fs.exists`回调中返回，以防止进一步的代码执行（例如，发送`404`）。在`if`语句中放置`return`类似于使用`else`分支。然而，在 Node 中，`if return`模式通常比使用`if else`更可取，因为它消除了另一组花括号。

现在我们可以导航到`localhost:8080`，这将提供我们的`index.html`文件。`index.html`文件调用我们的`script.js`和`styles.css`文件，我们的服务器也以适当的 MIME 类型提供这些文件。结果可以在以下截图中看到：

![操作步骤...](img/7188_01_image.jpg)

这个示例用来说明提供静态文件的基本原理。请记住，这不是一个高效的解决方案！在现实世界的情况下，我们不希望每次请求到达服务器时都进行 I/O 调用，尤其是对于较大的文件来说，这是非常昂贵的。在接下来的示例中，我们将学习更好的方法来提供静态文件。

## 工作原理...

我们的脚本创建了一个服务器并声明了一个名为`lookup`的变量。我们使用双管道（||）*或*运算符为`lookup`赋值。这定义了一个默认路由，如果`path.basename`为空的话。然后我们将`lookup`传递给一个新变量，我们将其命名为`f`，以便将我们的`content`目录前置到预期的文件名。接下来，我们通过`fs.exists`方法运行`f`并检查回调中的`exist`参数，以查看文件是否存在。如果文件存在，我们使用`fs.readFile`进行异步读取。如果访问文件出现问题，我们将写入`500`服务器错误，结束响应，并从`fs.readFile`回调中返回。我们可以通过从`index.html`中删除读取权限来测试错误处理功能。

```js
chmod -r index.html 

```

这样做将导致服务器抛出`500`服务器错误状态码。要再次设置正确，请运行以下命令：

```js
chmod +r index.html 

```

只要我们可以访问文件，就可以使用我们方便的`mimeTypes`映射对象来获取`content-type`，编写标头，使用从文件加载的数据结束响应，最后从函数返回。如果请求的文件不存在，我们将绕过所有这些逻辑，写入`404`，并结束响应。

## 还有更多...

需要注意的一点是...

### 网站图标陷阱

当使用浏览器测试我们的服务器时，有时会观察到意外的服务器请求。这是浏览器请求服务器可以提供的默认`favicon.ico`图标文件。除了看到额外的请求之外，这通常不是问题。如果网站图标请求开始干扰，我们可以这样处理：

```js
	if (request.url === '/favicon.ico') {
	  response.end();
	  return;
	}

```

如果我们想对客户端更有礼貌，还可以在发出`response.end`之前使用`response.writeHead(404)`通知它`404`。

## 另请参阅

+   *在本章中讨论的将内容缓存在内存中以进行即时传递*

+   *在本章中讨论的使用流来优化性能*

+   *在本章中讨论的防止文件系统黑客攻击*

# 将内容缓存在内存中以进行即时传递

直接在每个客户端请求上访问存储并不理想。在本例中，我们将探讨如何通过仅在第一次请求时访问磁盘、为第一次请求缓存文件数据以及从进程内存中提供所有后续请求来增强服务器效率。

## 准备工作

我们将改进上一个任务中的代码，因此我们将使用`server.js`，以及`content`目录中的`index.html，styles.css`和`script.js`。

## 操作步骤...

让我们首先看一下上一个配方“提供静态文件”的脚本

```js
	var http = require('http');
	var path = require('path');
	var fs = require('fs');  

	var mimeTypes = {
	  '.js' : 'text/javascript',
	  '.html': 'text/html',
	  '.css' : 'text/css'
	} ;

	http.createServer(function (request, response) {
	  var lookup = path.basename(decodeURI(request.url)) || 'index.html';
	  var f = 'content/'+lookup;
	  path.exists(f, function (exists) {
	    if (exists) {
	      fs.readFile(f, function(err,data) {
	      if (err) {response.writeHead(500); response.end('Server Error!'); return; }
	      var headers = {'Content-type': mimeTypes[path.extname(lookup)]};
	        response.writeHead(200, headers);
	        response.end(data);            
	      });
	      return;
	    }
	      response.writeHead(404); //no such file found!
	      response.end('Page Not Found!');
	  });

```

我们需要修改这段代码，只读取文件一次，将其内容加载到内存中，然后从内存中响应所有对该文件的请求。为了保持简单和可维护性，我们将缓存处理和内容传递提取到一个单独的函数中。因此，在`http.createServer`上方，并在`mimeTypes`下方，我们将添加以下内容：

```js
	var cache = {};
	function cacheAndDeliver(f, cb) {
	  if (!cache[f]) {
	    fs.readFile(f, function(err, data) {
	      if (!err) {
	        cache[f] = {content: data} ;
	      }     
	      cb(err, data);
	    });
	    return;
	  }
	  console.log('loading ' + f + ' from cache');
	  cb(null, cache[f].content);
	}
	//http.createServer …..

```

添加了一个新的`cache`对象，用于将文件存储在内存中，以及一个名为`cacheAndDeliver`的新函数。我们的函数接受与`fs.readFile`相同的参数，因此我们可以在`http.createServer`回调中替换`fs.readFile`，同时保持其余代码不变：

```js
	//...inside http.createServer:
	path.exists(f, function (exists) {
	    if (exists) {
	      cacheAndDeliver(f, function(err, data) {
	        if (err) {response.writeHead(500); response.end('Server Error!'); return; }
	        var headers = {'Content-type': mimeTypes[path.extname(f)]};
	        response.writeHead(200, headers);
	        response.end(data);      
	      });
	  return;
	    }
	//rest of path exists code (404 handling)...

```

当我们执行`server.js`文件并连续两次访问`localhost:8080`时，第二个请求会导致控制台输出以下内容：

```js
 loading content/index.html from cache
	loading content/styles.css from cache
	loading content/script.js from cache

```

## 工作原理...

我们定义了一个名为`cacheAndDeliver`的函数，类似于`fs.readFile`，它接受文件名和回调作为参数。这很棒，因为我们可以将完全相同的`fs.readFile`回调传递给`cacheAndDeliver`，在不向`http.createServer`回调内部添加任何额外可视复杂性的情况下，为服务器添加缓存逻辑。目前来看，将我们的缓存逻辑抽象成外部函数的价值是有争议的，但是随着我们不断增强服务器的缓存能力，这种抽象变得越来越可行和有用。我们的`cacheAndDeliver`函数检查所请求的内容是否已经缓存，如果没有，我们调用`fs.readFile`并从磁盘加载数据。一旦我们有了这些数据，我们可能会保留它，因此它被放入由其文件路径引用的`cache`对象中（`f`变量）。下次有人请求文件时，`cacheAndDeliver`将看到我们在`cache`对象中存储了文件，并将发出包含缓存数据的替代回调。请注意，我们使用另一个新对象填充了`cache[f]`属性，其中包含一个`content`属性。这样做可以更容易地扩展将来的缓存功能，因为我们只需要将额外的属性放入我们的`cache[f]`对象中，并提供与这些属性相对应的接口逻辑。

## 还有更多...

如果我们修改正在提供的文件，任何更改都不会反映在我们重新启动服务器之前。我们可以解决这个问题。

### 反映内容更改

要检测请求的文件自上次缓存以来是否发生了更改，我们必须知道文件何时被缓存以及上次修改时间。为了记录文件上次缓存的时间，让我们扩展`cache[f]`对象：

```js
	cache[f] = {content: data,
	                      timestamp: Date.now() //store a Unix time stamp
	                     };

```

现在我们需要找出文件上次更新的时间。`fs.stat`方法在其回调的第二个参数中返回一个对象。该对象包含与命令行 GNU coreutils `stat.fs.stat`提供的相同有用信息：上次访问时间（`atime`）、上次修改时间（`mtime`）和上次更改时间（`ctime`）。`mtime`和`ctime`之间的区别在于`ctime`将反映对文件的任何更改，而`mtime`只会反映对文件内容的更改。因此，如果我们更改了文件的权限，`ctime`会更新，但`mtime`会保持不变。我们希望在发生权限更改时注意到，因此让我们使用`ctime`属性：

```js
	//requires and mimeType object....
	var cache = {};
	function cacheAndDeliver(f, cb) {
		fs.stat(f, function (err, stats) {
			var lastChanged = Date.parse(stats.ctime),
				isUpdated = (cache[f]) && lastChanged > cache[f].timestamp;
			if (!cache[f] || isUpdated) {
				fs.readFile(f, function (err, data) {
					console.log('loading ' + f + ' from file');
					//rest of cacheAndDeliver
		}); //end of fs.stat
	} // end of cacheAndDeliver

```

`cacheAndDeliver`的内容已经包装在`fs.stat`回调中。添加了两个变量，并修改了`if(!cache[f])`语句。我们解析了第二个参数`stats`的`ctime`属性，使用`Date.parse`将其转换为自 1970 年 1 月 1 日午夜以来的毫秒数（Unix 纪元），并将其分配给我们的`lastChanged`变量。然后我们检查所请求文件的上次更改时间是否大于我们缓存文件的时间（假设文件确实已缓存），并将结果分配给我们的`isUpdated`变量。之后，只需通过`||`（或）运算符将`isUpdated`布尔值添加到条件`if(!cache[f])`语句中。如果文件比我们缓存的版本更新（或者尚未缓存），我们将文件从磁盘加载到缓存对象中。

## 另请参阅

+   在本章中讨论了通过流优化性能

+   *在* 第三章 *中讨论了通过 AJAX 进行浏览器-服务器传输*，*数据序列化处理*

# 通过流优化性能

缓存内容确实改进了每次请求时从磁盘读取文件。但是，使用`fs.readFile`时，我们是在将整个文件读入内存后再将其发送到`response`中。为了提高性能，我们可以从磁盘流式传输文件，并将其直接传输到`response`对象，一次发送一小部分数据到网络套接字。

## 准备工作

我们正在构建上一个示例中的代码，所以让我们准备好`server.js, index.html, styles.css`和`script.js`。

## 如何做...

我们将使用`fs.createReadStream`来初始化一个流，可以将其传输到`response`对象。在这种情况下，在我们的`cacheAndDeliver`函数中实现`fs.createReadStream`并不理想，因为`fs.createReadStream`的事件监听器将需要与`request`和`response`对象进行接口。为了简单起见，这些最好在`http.createServer`回调中处理。为了简洁起见，我们将放弃我们的`cacheAndDeliver`函数，并在服务器回调中实现基本的缓存：

```js
	//requires, mime types, createServer, lookup and f vars...
	path.exists(f, function (exists) {
	    if (exists) {  
	      var headers = {'Content-type': mimeTypes[path.extname(f)]};
	      if (cache[f]) {
	        response.writeHead(200, headers);              
	        response.end(cache[f].content);  
	        return;
	      } //...rest of server code...

```

稍后，当我们与`readStream`对象进行接口时，我们将填充`cache[f].content`。以下是我们如何使用`fs.createReadStream：`

```js
var s = fs.createReadStream(f);

```

这将返回一个`readStream`对象，该对象流式传输由`f`变量指向的文件。`readStream`发出我们需要监听的事件。我们可以使用`addEventListener`进行监听，也可以使用简写的`on:`

```js
var s = fs.createReadStream(f).on('open', function () {
//do stuff when the readStream opens
});

```

由于`createReadStream`返回`readStream`对象，我们可以使用点符号的方法链接将我们的事件监听器直接附加到它上面。每个流只会打开一次，我们不需要继续监听它。因此，我们可以使用`once`方法而不是`on`方法，在第一次事件发生后自动停止监听：

```js
var s = fs.createReadStream(f).once('open', function () {
//do stuff when the readStream opens
});

```

在我们填写`open`事件回调之前，让我们按照以下方式实现错误处理：

```js
	var s = fs.createReadStream(f).once('open', function () {
	//do stuff when the readStream opens
	}).once('error', function (e) {
	    console.log(e);
	    response.writeHead(500);
	    response.end('Server Error!');
	});

```

整个努力的关键是`stream.pipe`方法。这使我们能够直接从磁盘获取文件并将其直接通过我们的`response`对象流式传输到网络套接字。

```js
	var s = fs.createReadStream(f).once('open', function () {
	    response.writeHead(200, headers);      
	    this.pipe(response);
	}).once('error', function (e) {
	    console.log(e);
	    response.writeHead(500);
	    response.end('Server Error!');
	});

```

结束响应怎么办？方便的是，`stream.pipe`会检测流何时结束，并为我们调用`response.end`。出于缓存目的，我们需要监听另一个事件。在我们的`fs.exists`回调中，在`createReadStream`代码块下面，我们编写以下代码：

```js
	 fs.stat(f, function(err, stats) {
		        var bufferOffset = 0;
	      	  cache[f] = {content: new Buffer(stats.size)};
		       s.on('data', function (chunk) {
	             chunk.copy(cache[f].content, bufferOffset);
	             bufferOffset += chunk.length;
	        });
	      }); 

```

我们使用`data`事件来捕获正在流式传输的缓冲区，并将其复制到我们提供给`cache[f].content`的缓冲区中，使用`fs.stat`来获取文件的缓冲区大小。

## 它是如何工作的...

客户端不需要等待服务器从磁盘加载完整的文件然后再发送给客户端，我们使用流来以小的、有序的片段加载文件，并立即发送给客户端。对于较大的文件，这是特别有用的，因为在文件被请求和客户端开始接收文件之间几乎没有延迟。

我们通过使用`fs.createReadStream`来开始从磁盘流式传输我们的文件。`fs.createReadStream`创建了`readStream`，它继承自`EventEmitter`类。

`EventEmitter`类实现了 Node 标语中的*evented*部分：Evented I/O for V8 JavaScript。因此，我们将使用监听器而不是回调来控制流逻辑的流程。

然后我们使用`once`方法添加了一个`open`事件监听器，因为我们希望一旦触发就停止监听`open`。我们通过编写标头并使用`stream.pipe`方法将传入的数据直接传输到客户端来响应`open`事件。

`stream.pipe`处理数据流。如果客户端在处理过程中变得不堪重负，它会向服务器发送一个信号，服务器应该通过暂停流来予以尊重。在底层，`stream.pipe`使用`stream.pause`和`stream.resume`来管理这种相互作用。

当响应被传输到客户端时，内容缓存同时被填充。为了实现这一点，我们必须为`cache[f].content`属性创建一个`Buffer`类的实例。`Buffer`必须提供一个大小（或数组或字符串），在我们的情况下是文件的大小。为了获取大小，我们使用了异步的`fs.stat`并在回调中捕获了`size`属性。`data`事件将`Buffer`作为其唯一的回调参数返回。

流的默认`bufferSize`为 64 KB。任何大小小于`bufferSize`的文件将只触发一个`data`事件，因为整个文件将适合第一个数据块中。但是，对于大于`bufferSize`的文件，我们必须一次填充我们的`cache[f].content`属性的一部分。

### 注意

更改默认的`readStream`缓冲区大小：

我们可以通过传递一个`options`对象并在`fs.createReadStream`的第二个参数中添加一个`bufferSize`属性来更改`readStream`的缓冲区大小。

例如，要将缓冲区加倍，可以使用`fs.createReadStream(f,{bufferSize: 128 * 1024})`；

我们不能简单地将每个`chunk`与`cache[f].content`连接起来，因为这样会将二进制数据强制转换为字符串格式，尽管不再是二进制格式，但以后会被解释为二进制格式。相反，我们必须将所有小的二进制缓冲区`chunks`复制到我们的二进制`cache[f].content`缓冲区中。

我们创建了一个`bufferOffset`变量来帮助我们。每次我们向我们的`cache[f].content`缓冲区添加另一个`chunk`时，我们通过将`chunk`缓冲区的长度添加到它来更新我们的新`bufferOffset`。当我们在`chunk`缓冲区上调用`Buffer.copy`方法时，我们将`bufferOffset`作为第二个参数传递，以便我们的`cache[f].content`缓冲区被正确填充。

此外，使用`Buffer`类进行操作可以提高性能，因为它可以绕过 V8 的垃圾回收方法。这些方法往往会使大量数据碎片化，从而减慢 Node 处理它们的能力。

## 还有更多...

虽然流解决了等待文件加载到内存中然后传递它们的问题，但我们仍然通过我们的`cache`对象将文件加载到内存中。对于较大的文件或大量文件，这可能会产生潜在的影响。

### 防止进程内存溢出

进程内存有限。默认情况下，V8 的内存在 64 位系统上设置为 1400 MB，在 32 位系统上设置为 700 MB。可以通过在 Node 中运行`--max-old-space-size=N`来改变这个值，其中`N`是以兆字节为单位的数量（实际可以设置的最大值取决于操作系统和可用的物理 RAM 数量）。如果我们绝对需要占用大量内存，我们可以在大型云平台上运行服务器，分割逻辑，并使用`child_process`类启动新的 node 实例。

在这种情况下，高内存使用并不一定是必需的，我们可以优化我们的代码，显著减少内存溢出的可能性。对于缓存较大的文件，好处较少。与总下载时间相比，轻微的速度提高是微不足道的，而缓存它们的成本相对于我们可用的进程内存来说是相当显著的。我们还可以通过在缓存对象上实现过期时间来提高缓存效率，然后用它来清理缓存，从而删除低需求的文件，并优先处理高需求的文件以实现更快的传递。让我们稍微重新排列一下我们的`cache`对象：

```js
	var cache = {
	  store: {},
	  maxSize : 26214400, //(bytes) 25mb
	}

```

为了更清晰的思维模型，我们要区分缓存作为一个功能实体和缓存作为存储（这是更广泛的缓存实体的一部分）。我们的第一个目标是只缓存一定大小的文件。我们为此定义了`cache.maxSize`。现在我们只需要在`fs.stat`回调中插入一个`if`条件：

```js
	 fs.stat(f, function (err, stats) {
	        if (stats.size < cache.maxSize) {
	          var bufferOffset = 0;
	          cache.store[f] = {content: new Buffer(stats.size),
	                                     timestamp: Date.now() };
	          s.on('data', function (data) {
	            data.copy(cache.store[f].content, bufferOffset);
	            bufferOffset += data.length;
	          });
	        }  
	      });

```

请注意，我们还在我们的`cache.store[f]`中悄悄地添加了一个新的`timestamp`属性。这是为了清理缓存，这是我们的第二个目标。让我们扩展`cache:`。

```js
	var cache = {
	  store: {},
	  maxSize: 26214400, //(bytes) 25mb
	  maxAge: 5400 * 1000, //(ms) 1 and a half hours
	  clean: function(now) {
	      var that = this;
	      Object.keys(this.store).forEach(function (file) {
	        if (now > that.store[file].timestamp + that.maxAge) {
	          delete that.store[file];      
	        }
	      });
	  }
	};

```

因此，除了`maxSize`，我们创建了一个`maxAge`属性并添加了一个`clean`方法。我们在服务器底部调用`cache.clean`，如下所示：

```js
	//all of our code prior
	  cache.clean(Date.now());
	}).listen(8080); //end of the http.createServer

```

`cache.clean`循环遍历`cache.store`，并检查它是否已超过指定的生命周期。如果是，我们就从`store`中移除它。我们将再添加一个改进，然后就完成了。`cache.clean`在每个请求上都会被调用。这意味着`cache.store`将在每次服务器命中时被循环遍历，这既不必要也不高效。如果我们每隔两个小时或者更长时间清理一次缓存，效果会更好。我们将向`cache`添加两个属性。第一个是`cleanAfter`，用于指定清理缓存的时间间隔。第二个是`cleanedAt`，用于确定自上次清理缓存以来的时间。

```js
	var cache = {
	  store: {},
	  maxSize: 26214400, //(bytes) 25mb
	  maxAge : 5400 * 1000, //(ms) 1 and a half hours
	   cleanAfter: 7200 * 1000,//(ms) two hours
	  cleanedAt: 0, //to be set dynamically
	  clean: function (now) {
	     if (now - this.cleanAfter > this.cleanedAt) {
	      this.cleanedAt = now;
	      that = this;
	        Object.keys(this.store).forEach(function (file) {
	          if (now > that.store[file].timestamp + that.maxAge) {
	            delete that.store[file];      
	          }
	        });
	    }
	  }
	};

```

我们将我们的`cache.clean`方法包裹在一个`if`语句中，只有当它距离上次清理已经超过两个小时（或者`cleanAfter`设置为其他值）时，才允许对`cache.store`进行循环。

## 另请参阅

+   *处理文件上传*在第二章中讨论过，*探索 HTTP 对象*

+   *防止文件系统黑客攻击*在本章中讨论。

# 防止文件系统黑客攻击

要使 Node 应用程序不安全，必须有攻击者可以与之交互以进行利用的东西。由于 Node 的极简主义方法，大部分责任都落在程序员身上，以确保他们的实现不会暴露安全漏洞。这个配方将帮助识别在处理文件系统时可能出现的一些安全风险反模式。

## 准备工作

我们将使用与以前的配方中相同的`content`目录，但我们将从头开始创建一个新的`insecure_server.js`文件（名字中有提示！）来演示错误的技术。

## 如何做...

我们以前的静态文件配方倾向于使用`path.basename`来获取路由，但这会使所有请求都处于平级。如果我们访问`localhost:8080/foo/bar/styles.css`，我们的代码会将`styles.css`作为`basename`，并将`content/styles.css`交付给我们。让我们在`content`文件夹中创建一个子目录，称之为`subcontent`，并将我们的`script.js`和`styles.css`文件移动到其中。我们需要修改`index.html`中的脚本和链接标签：

```js
	<link rel=stylesheet type=text/css href=subcontent/styles.css>
	<script src=subcontent/script.js type=text/javascript></script>

```

我们可以使用`url`模块来获取整个`pathname`。所以让我们在我们的新的`insecure_server.js`文件中包含`url`模块，创建我们的 HTTP 服务器，并使用`pathname`来获取整个请求路径：

```js
	var http = require('http'); var path = require('path'); 
	var url = require('url');
	var fs = require('fs'); 
	http.createServer(function (request, response) {
	  var lookup = url.parse(decodeURI(request.url)).pathname;
	  lookup = (lookup === "/") ? '/index.html' : lookup;
	  var f = 'content' + lookup;
	  console.log(f);
	  fs.readFile(f, function (err, data) {
	    response.end(data);
	  });
	}).listen(8080);

```

如果我们导航到`localhost:8080`，一切都很顺利。我们已经多级了，万岁。出于演示目的，一些东西已经从以前的配方中剥离出来（比如`fs.exists`），但即使有了它们，以下代码也会呈现相同的安全隐患：

```js
curl localhost:8080/../insecure_server.js 

```

现在我们有了我们服务器的代码。攻击者也可以通过几次猜测相对路径来访问`/etc/passwd`：

```js
curl localhost:8080/../../../../../../../etc/passwd 

```

为了测试这些攻击，我们必须使用 curl 或其他等效工具，因为现代浏览器会过滤这些请求。作为解决方案，如果我们为要提供的每个文件添加一个唯一的后缀，并且要求服务器在提供文件之前必须存在这个后缀，会怎么样？这样，攻击者就可以请求`/etc/passwd`或我们的`insecure_server.js`，因为它们没有唯一的后缀。为了尝试这个方法，让我们复制`content`文件夹，并将其命名为`content-pseudosafe`，并将我们的文件重命名为`index.html-serve`、`script.js-serve`和`styles.css-serve`。让我们创建一个新的服务器文件，并将其命名为`pseudosafe_server.js`。现在我们只需要让`-serve`后缀成为必需的：

```js
	//requires section...
	http.createServer(function (request, response) {
	  var lookup = url.parse(decodeURI(request.url)).pathname;
	  lookup = (lookup === "/") ? '/index.html-serve' : lookup + '-serve';
	  var f = 'content-pseudosafe' + lookup;

```

出于反馈目的，我们还将使用`fs.exists`来处理一些`404`。

```js
	//requires, create server etc  
	path.exists(f, function (exists) {
	    if (!exists) {  
	      response.writeHead(404);
	      response.end('Page Not Found!');
	      return;
	    }
	//read file etc

```

让我们启动我们的`pseudosafe_server.js`文件，并尝试相同的攻击：

```js
curl -i localhost:8080/../insecure_server.js 

```

我们使用了`-i`参数，以便 curl 输出头部。结果是什么？一个`404`，因为它实际上正在寻找的文件是`../insecure_server.js-serve`，这个文件不存在。这种方法有什么问题？嗯，它很不方便，容易出错。然而，更重要的是，攻击者仍然可以绕过它！

```js
curl localhost:8080/../insecure_server.js%00/index.html 

```

然后！这是我们的服务器代码。我们问题的解决方案是`path.normalize`，它可以在`fs.readFile`之前清理我们的`pathname`。

```js
	http.createServer(function (request, response) {
	  var lookup = url.parse(decodeURI(request.url)).pathname;
	  lookup = path.normalize(lookup);
	  lookup = (lookup === "/") ? '/index.html' : lookup;
	  var f = 'content' + lookup

```

之前的示例没有使用`path.normalize`，但它们仍然相对安全。`path.basename`给出了路径的最后部分，因此任何前导的相对目录指针（../）都被丢弃，从而防止了目录遍历的利用。

## 它是如何工作的...

在这里，我们有两种文件系统利用技术：**相对目录遍历**和**毒空字节攻击**。这些攻击可以采取不同的形式，比如在 POST 请求中或来自外部文件。它们可能会产生不同的影响。例如，如果我们正在写入文件而不是读取它们，攻击者可能会开始对我们的服务器进行更改。在所有情况下，安全性的关键是验证和清理来自用户的任何数据。在`insecure_server.js`中，我们将用户请求传递给我们的`fs.readFile`方法。这是愚蠢的，因为它允许攻击者利用我们操作系统中相对路径功能，通过使用`../`来访问本应禁止访问的区域。通过添加`-serve`后缀，我们没有解决问题。我们只是贴了一张创可贴，这可以被毒空字节绕过。这种攻击的关键是`%00`，这是空字节的 URL 十六进制代码。在这种情况下，空字节使 Node 对`../insecure_server.js`部分变得盲目，但当同样的空字节通过我们的`fs.readFile`方法发送时，它必须与内核进行接口。然而，内核对`index.html`部分变得盲目。所以我们的代码看到的是`index.html`，但读取操作看到的是`../insecure_server.js`。这就是空字节毒害。为了保护自己，我们可以使用`regex`语句来删除路径中的`../`部分。我们还可以检查空字节并输出`400 Bad Request`语句。但我们不需要，因为`path.normalize`已经为我们过滤了空字节和相对部分。

## 还有更多...

让我们进一步探讨在提供静态文件时如何保护我们的服务器。

### 白名单

如果安全性是一个极端重要的优先事项，我们可以采用严格的白名单方法。在这种方法中，我们将为我们愿意交付的每个文件创建一个手动路由。不在我们的白名单上的任何内容都将返回`404`。我们可以在`http.createServer`上方放置一个`whitelist`数组，如下面的代码所示：

```js
	var whitelist = [
	  '/index.html',
	  '/subcontent/styles.css',
	  '/subcontent/script.js'
	];

```

在我们的`http.createServer`回调中，我们将放置一个`if`语句来检查请求的路径是否在`whitelist`数组中：

```js
	if (whitelist.indexOf(lookup) === -1) {
	  response.writeHead(404);
	  response.end('Page Not Found!');
	  return;
	}

```

就是这样。我们可以通过在我们的`content`目录中放置一个文件`non-whitelisted.html`来测试这个。

```js
curl -i localhost:8080/non-whitelisted.html 

```

上述命令将返回`404`，因为`non-whitelisted.html`不在白名单上。

### Node-static

[`github.com/joyent/node/wiki/modules#wiki-web-frameworks-static`](https://github.com/joyent/node/wiki/modules#wiki-web-frameworks-static)列出了可用于不同目的的静态文件服务器模块的列表。在依赖它来提供您的内容之前，确保项目是成熟和活跃的是一个好主意。Node-static 是一个开发完善的模块，内置缓存。它还符合 RFC2616 HTTP 标准规范。这定义了如何通过 HTTP 传递文件。Node-static 实现了本章讨论的所有基本要点，以及更多。这段代码略有改动，来自 node-static 的 Github 页面[`github.com/cloudhead/node-static:`](https://github.com/cloudhead/node-static)

```js
	var static = require('node-static');
	var fileServer = new static.Server('./content');
	require('http').createServer(function (request, response) {
	  request.addListener('end', function () {
	    fileServer.serve(request, response);
	  });
	}).listen(8080);

```

上述代码将与`node-static`模块进行接口，以处理服务器端和客户端缓存，使用流来传递内容，并过滤相对请求和空字节，等等。

## 另请参阅

+   *防止跨站点请求伪造*在第七章中讨论，*实施安全、加密和身份验证*

+   设置 HTTPS Web 服务器 在第七章 *实施安全、加密和认证*

+   部署到服务器环境 在第十章 *上线*

+   密码哈希加密 在第七章 *实施安全、加密和认证*
