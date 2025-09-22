# 第七章 利用 MEAN 堆栈

MEAN 堆栈是 MongoDB、Express、AngularJS 和 Node.js 的缩写。它代表仅使用 JavaScript 进行全栈开发的实践。自然，你需要一些 HTML 和 CSS 来将内容渲染到浏览器中，并使其看起来更美观。

MongoDB 是一个基于文档的 NoSQL 数据库，它以可以像普通 JavaScript 对象一样处理的方式存储数据。由于数据和数据库方法本质上都是 JavaScript，MongoDB 与基于 JavaScript 的应用程序配合得很好。此时，如果你正确地输入了代码，控制台将不会有除了新行之外的输出。打开你最喜欢的浏览器

Express 是一个用 JavaScript 编写的 Web 应用程序框架，它在 Node.js 上运行得很好。它与 Sinatra 等其他框架类似，但更不显眼，也没有太多偏见。Express 本质上是一种路由和中间件，用于处理 Web 请求和响应。

AngularJS 是一个主要用于构建单页 Web 应用程序的前端 JavaScript 框架。它是由 Google 精心策划的一个强大且具有指导性的框架，已经成为更受欢迎的 JavaScript MV*工具包之一。

在本章中，你将在开始构建你自己的单页应用程序框架的同时，探索以下 MEAN 堆栈组件：

+   使用 REPL 从命令行运行 Node.js 代码

+   编写和运行 Node.js 脚本

+   在 Mongo shell 中安装 MongoDB 以及基本的 CRUD 操作

+   通过标准方法和 Express 生成器安装 Express

+   Express 路由、中间件和视图渲染

+   使用 Angular 构建前端应用程序的基本组件

# Node.js 环境

Node.js 是一个用于执行 JavaScript 的运行时环境。使用它，你可以构建强大的软件，例如完整的后端 Web 服务器。在*第一章*，*使用 NPM、Bower 和 Grunt 进行组织*中，你开始使用一些基于 Node.js 的工具，如 Grunt、NPM 和 Node 包管理器。

Node.js 功能强大，速度极快。它基于 Chrome 浏览器中使用的 V8 JavaScript 引擎，并针对速度进行了优化。它使用非阻塞 I/O，允许它同时处理多个请求。

本章假设你已经安装了 Node 运行时。如果没有，请访问[`nodejs.org`](https://nodejs.org)，并按照你操作系统的安装说明进行安装。本书使用了 Node 版本 4.3.2。

## 运行 REPL

Node.js 提供了一种从命令行运行 JavaScript 代码的方法，称为读取-评估-打印循环或 REPL。你只需在控制台中命令行输入`node`即可启动 REPL。这是一个开始探索 Node.js 一些可能性的好方法，如下面的命令所示：

```js
$ node
> var sum = 1 + 2;
undefined
> sum
3
console.log(sum);
3
undefined

```

### 注意

JavaScript 中的变量赋值返回 undefined。

在这里，可能看起来有两个返回值，第一个是`3`，第二个是`undefined`。在这种情况下，`console.log()`是一个用于将内容输出到屏幕上的函数，但实际上它返回`undefined`。这在编写 Node.js 代码时很有用，你希望将内容输出到屏幕上，类似于其他编程语言中的打印语句。

要退出 REPL，按*Ctrl* + *C*两次。

## 编写 hello Node.js 脚本

单独使用 REPL 本身不会非常有用。Node.js 允许开发者通过将程序编写出来并保存为具有`.js`文件扩展名的文本文件来创建程序。

这其中一个优点是，你可以使用任何文本编辑器和几乎任何 IDE 来编写 Node.js 程序。

为了展示 Node.js 的强大功能，让我们先构建一个简单的 Web 服务器。它不会做很多事情，除了处理 HTTP 请求，但这是一个很好的开始。

让我们创建一个新的文件，命名为`hello.js`，并用你喜欢的文本编辑器或基于文本的 IDE 打开它。然后，输入以下代码行：

```js
var http = require('http'); 
var serverResponse = function(req, res){ 
    res.end("Hello Node"); 
} 

var server = http.createServer(serverResponse); 
server.listen(3000); 

```

保存文件，并在控制台中导航到存储该文件的目录。然后，输入以下命令：

```js
$ node hello

```

在这一点上，如果你正确地输入了代码，控制台将不会有除了新行之外的输出。打开你喜欢的浏览器，并在地址栏中输入`localhost:3000`。

你应该在浏览器的主窗口中看到**Hello Node**。就这样，你已经编写了一个 Web 服务器。

你可以通过在控制台中输入 Ctrl + C 来停止服务器。

在这么一小段代码中有很多事情在进行，所以让我们来看看程序正在做什么：

+   在`varhttp=require('http');`代码的第一行中，类似于某些编程语言中的导入语句，Node.js 使用`require`来导入代码模块。Node.js 自带了许多内置代码模块。HTTP 模块是一个内置模块，正如你可能想象的那样，它提供了 HTTP 服务。可以使用包含模块名称的字符串来导入内置模块。通常，它们会被分配给一个变量来使用。非内置模块则使用包含模块完整路径的字符串来导入。

+   在接下来的两行中，`var serverResponse = function(req, res)`和`res.end("Hello Node")`，这里的`serverResponse`函数是一个回调函数，我们将将其传递给我们所创建的 Web 服务器。当我们进入`Express.js`时，我们将更详细地介绍请求和响应对象，但重要的是要意识到我们正在设置`req`来处理 HTTP 请求对象，以及`res`来处理响应。响应对象上的`end`函数发送传递给它的任何文本，并告诉服务器响应已完成。

+   在下一行代码中，`var server = http.createServer(serverResponse);`，我们实际上是通过在之前导入的 HTTP 对象上调用 `createServer` 函数来创建一个 Web 服务器。我们将 `serverResponse` 函数传递给 `createServer` 函数，它成为服务器的回调函数。我们将刚刚创建的服务器引用存储在名为 *server* 的变量中。

+   在代码的最后一行，`server.listen(3000);`，我们将对刚刚创建的服务器对象调用 `listen` 函数。我们将传递一个整数作为端口号，表示我们希望它监听的端口号。这段代码实际上启动了服务器，并使其在端口 `3000` 上监听请求。

## 使用 NPM 设置 Node.js 项目

随着 Node.js 项目的变大，拥有一个能够正确管理它的工具变得非常重要。使用 **Node Package Manager** 和 `package.json` 文件，可以更容易地处理诸如一致的依赖管理、版本管理和环境管理等问题。

幸运的是，创建 Node.js 的聪明人创造了一种使用 NPM 来实现这一目标的方法。在命令行中运行 `npm init` 将会设置一个 Node.js 项目并构建一个 `package.json` 文件，该文件用于管理你的 Node.js 项目。让我们用你的 `hello` Node 项目来试一试。请注意，一些提示是可选的，可以留空，如下面的命令所示：

```js
$ npm init
This utility will walk you through creating a package.json file.
It only covers the most common items, and tries to guess sensible defaults.
See `npm help json` for definitive documentation on these fields
and exactly what they do.
Use `npm install <pkg> --save` afterwards to install a package and
save it as a dependency in the package.json file.
Press ^C at any time to quit.
name: (SPA-js) hello-node
version: (1.0.0) 
description: my new program
entry point: (hello.js) 
test command: 
git repository: 
keywords: 
author: Your Name
license: (ISC) 
About to write to /Users/your-home/helloNode/package.json:
{
 "name": "hello-node",
 "version": "1.0.0",
 "description": "my new program",
 "main": "hello.js",
 "scripts": {
 "test": "echo "Error: no test specified" && exit 1"
 },
 "author": "John Moore",
 "license": "ISC"
}
Is this ok? (yes) 

```

在此按 Enter 键将会在当前目录中创建 `package.json` 文件。在您的 IDE 或文本编辑器中打开它并查看。当你在里面时，让我们对脚本部分进行以下小的修改：

```js
"scripts": { 
  "test": "echo "Error: no test specified" && exit 1", 
  "start": "node hello" 
} 

```

`package.json` 的脚本部分允许你使用 NPM 运行代码。在这种情况下，输入 npm `start` 将运行 `node hello` 命令并启动你的 Web 服务器。目前这并不是一个超级高效的快捷方式，但你可以通过这种方式创建许多高效且有用的命令别名。

NPM 做的非常重要的一件事是管理依赖。在下一节中，关于 Express，你将看到如何将 NPM 模块及其版本存储在 `package.json` 文件中。当作为团队的一部分工作时，这非常重要。通过复制项目的 `package.json` 文件，开发者可以通过运行 NPM 安装来重新创建项目环境。

# 开始使用 Express

Express 自称为 Node.js 的快速、无偏见、极简主义 Web 框架。Express 是一个非常强大且灵活的框架，它运行在 Node.js 之上，但仍然允许你访问 Node 的所有功能。在核心上，Express 作为一组路由和中间件功能运行。

我们将在后面的章节中详细讨论路由和中间件。基本上，路由处理 Web 请求。中间件由可以访问请求和响应对象并调用堆栈中下一个中间件的函数组成。

如果使用 Node.js 随意搭建一个 Web 服务器如此简单，为什么我们还需要像 Express 这样的东西呢？

答案是，您不需要。您可以完全独立地，仅通过编写自己的 Node.js 代码来构建一个功能齐全的 Web 应用程序。Express 已经为您做了很多艰苦的工作和重活。由于 Express 中间件易于添加，添加诸如安全、身份验证和路由等功能相对简单。而且，当您有一个令人兴奋的新 Web 应用程序要构建时，谁愿意从头开始构建这些功能呢？

## 安装 Express

当然，您会使用 NPM 来安装 Express。这里有两种方法可以做到这一点。

标准方法只是使用 NPM 拉取 Express 项目并将其添加到 `package.json` 中的引用。这只是将模块添加到 Node.js 项目中。您将构建一个应用程序脚本，在 Express 中调用它，并在那个 WAR 文件中利用它。

第二种方法是安装 Express 生成器，并使用它来生成一个入门级 Web 应用程序。这种方法简单易行，但它会为您构建整个应用程序的结构，包括文件夹结构。有些人可能更喜欢自己完成所有这些工作，以确保设置方式精确无误。

我们将尝试两种方法，并使用 Express 生成器来构建您将在本书剩余部分构建的单页应用程序的框架。

### 标准方法

再次强调，标准方法只是将模块拉下来并添加到您的项目中。在您的控制台中，在刚刚创建的 `package.json` 文件所在的目录中，输入以下命令：

```js
$ npm install express --save

```

### 小贴士

在 Mac 上，您可能需要在 `npm install` 前面输入 `sudo`。如果您在 Mac 上，我建议您在每次使用 NPM 安装任何东西时都使用 `sudo`。

`-save` 部分告诉 npm 将 Express 添加到您的 `package.json` 文件中的依赖项。现在，请打开您的 `package.json` 文件并查看依赖项部分。

如果您正在使用 `git` 或其他源代码控制系统与其他开发者共享代码库，您通常不会在远程仓库中存储依赖项。相反，`package.json` 文件将包含对所需模块及其版本的引用。

另一位开发者可以拉取您的代码，运行 `npm install`，并安装所有依赖项。

如果您查看 `package.json` 文件所在的文件夹，您将看到一个名为 `node_modules` 的新文件夹。这是您使用 `npm` 安装的依赖项存储的地方。不要移动这个文件夹或更改它，因为 require 函数会在这里查找模块。通常，这个文件夹会被添加到 `.gitignore` 文件中，以确保其文件不会存储在远程 `git` 仓库中。

### 表达生成器

另一种设置 Express 应用程序的方法是，Express 创建了一个生成器，这是一个用于快速搭建 Express 应用程序框架的工具。它假设了一些在 Express 应用程序中常用的约定，并配置了主应用程序、`package.json`以及甚至目录结构。

生成器是通过以下命令全局安装的，而不是在特定项目中使用：

```js
$ npm install express-generator -g

```

`-g`告诉 NPM 全局安装模块及其依赖项。它们不会安装在你项目的`npm_modules`文件夹中，而是在系统上的全局模块文件夹中。

## 设置你的 Express 应用程序

现在我们已经全局安装了 express 生成器，让我们使用它来开始构建本书其余部分将要构建的应用程序。选择一个你希望这个项目存在的文件夹。Express 生成器将在这个文件夹内创建一个新的文件夹，所以如果你是一个包含其他内容的家庭或项目文件夹，那也是可以的。

使用以下命令在你的控制台中导航到这个文件夹：

```js
$ express -e giftapp
 create : giftapp
 create : giftapp/package.json
 create : giftapp/app.js
 create : giftapp/public
 create : giftapp/public/javascripts
 create : giftapp/public/images
 create : giftapp/routes
 create : giftapp/routes/index.js
 create : giftapp/routes/users.js
 create : giftapp/public/stylesheets
 create : giftapp/public/stylesheets/style.css
 create : giftapp/views
 create : giftapp/views/index.ejs
 create : giftapp/views/error.ejs
 create : giftapp/bin
 create : giftapp/bin/www
 install dependencies:
 $ cd giftapp && npm install
 run the app:
 $ DEBUG=giftapp:* npm start

```

如你所见，express 生成器创建了一些文件，包括`app.js`，这是你的主应用程序文件。

我们在 express 命令后输入的`-e`修饰符告诉生成器我们想要使用`ejs`（嵌入式 JavaScript）前端模板。Express 支持多种模板语言，包括 Handlebars 和 Jade。如果你没有添加任何修饰符，Express 生成器将假设你想要使用 Jade。对于这个项目，我们将使用`ejs`，它本质上是一种嵌入 JavaScript 代码的 HTML。

生成器的最终输出会告诉你下一步你需要执行的操作，以便真正搭建你的应用程序。导航到你的新`giftapp`目录并运行`npm install`（如果你在 Mac 或 Linux 机器上，请记住使用`sudo`）。此时，`npm install`命令可能需要几分钟，因为它正在安装多个依赖项。

下一个命令将以`DEBUG`模式启动你的新 Express 应用程序——你将看到所有请求都记录在控制台。在浏览器中导航到`localhost:3000`将显示一个**欢迎使用 Express**页面。

恭喜你，这个页面正在由你自己的强大 Express Web 应用程序提供服务。目前它还没有做什么，但已经为你搭建了很多基础。

## 探索主脚本

打开 giftapp 的新`package.json`文件。注意以下 scripts 对象：

```js
"scripts": { 
  "start": "node ./bin/www" 
} 

```

这意味着当运行`npm start`时，实际调用的脚本位于`./bin/www`。所以，让我们打开你 bin 目录下的`www`文件并查看。

你会看到这个文件正在引入许多东西，包括`app.js`；这是你的主应用程序文件：

```js
var app = require('../app'); 

```

下一段代码设置了应用程序的端口号。它检查是否有设置包含所需端口号的环境变量。如果没有，它将其设置为`3000`。在生产环境中部署应用程序时，通常你会使用`端口 80`用于 HTTP 或`端口 443`用于 HTTPS：

```js
var port = normalizePort(process.env.PORT || '3000'); 
app.set('port', port); 

```

下一个部分创建服务器并开始监听正确的端口：

```js
var server = http.createServer(app); 
... 
server.listen(port); 

```

现在，我们将跳过这个文件的其余部分，看看 `app.js` 文件中有什么。打开它看看。

## 查看主应用

`App.js` 是主应用文件，它加载路由和中间件并配置应用。这个文件中有几个重要的部分。一般来说，在 Express 应用中，加载和利用的顺序以及在这个文件中中间件被调用的顺序是很重要的。让我们来看看。

### 加载依赖

当你打开 `app.js` 时，你首先会遇到对 require 函数的多次调用，如下所示：

```js
var express = require('express'); 
var path = require('path'); 
var favicon = require('serve-favicon'); 
var logger = require('morgan'); 
var cookieParser = require('cookie-parser'); 
var bodyParser = require('body-parser'); 
var routes = require('./routes/index'); 
var users = require('./routes/users'); 

```

这个系统被称为 `CommonJS`。第一组模块包括依赖项，如 Express 和 cookies 解析器。路由和用户模块是由 Express 生成器为路由创建的。你创建的路由也将需要在主文件中导入。

### 配置应用

接下来，通过调用 express 函数声明了一个 `app` 变量，并设置了一些配置变量，如下所示：

```js
var app = express(); 
... 
app.set('views', path.join(__dirname, 'views')); 
app.set('view engine', 'ejs'); 

```

第一项配置，视图，告诉 Express 在哪里查找视图模板。这些模板通常在向网络应用发送请求时渲染成 HTML。

第二项配置设置了视图引擎。正如我们之前讨论的，我们将使用 `ejs` 或嵌入式 JavaScript 模板。

### 应用级别中间件

在这个文件中接下来我们看到的是对 app 对象的 `use` 函数的一堆调用，如下所示：

```js
app.use(logger('dev')); 
app.use(bodyParser.json()); 
app.use(bodyParser.urlencoded({ extended: false })); 
app.use(cookieParser()); 
app.use(express.static(path.join(__dirname, 'public'))); 
app.use('/', routes); 
app.use('/users', users); 
// catch 404 and forward to error handler 
app.use(function(req, res, next) { 
  var err = new Error('Not Found'); 
  err.status = 404; 
  next(err); 
}); 

```

在 Express 中，对应用对象 use 函数的调用是应用级别的中间件。发送到应用的请求将执行与 `app.use` 中设置的路径匹配的每个函数。如果没有设置路径，中间件函数默认为根路径 `/`，并将对每个请求进行调用。

例如，`app.use(cookieParser());` 表示对于发送到应用的每个请求，都会调用 `cookieParser` 函数，因为它默认为根路径。然而，`app.use('/users',users);` 只会在请求以 `/users` 开头时应用。

中间件的调用顺序与声明顺序一致。这将在我们添加身份验证到我们的应用或想要处理 `POST` 数据时变得非常清楚。如果你在尝试管理需要身份验证的请求之前不解析 cookies，它将不起作用。

## 我们的第一条 Express 路由

Express 使用一个称为路由的机制，通过其 `Router` 对象来处理 HTTP 请求，例如来自网络浏览器的请求。在后面的章节中，我们将更深入地探讨 Express 路由。检查 Express 生成器为我们创建的第一个路由是很有必要的。打开你的 `routes/index.js` 文件，如下面的代码块所示：

```js
var express = require('express'); 
var router = express.Router(); 

/* GET home page. */ 
router.get('/', function(req, res, next) { 
  res.render('index', { title: 'Express' }); 
}); 
module.exports = router; 

```

要创建一组新的路由，我们必须通过调用 `Express Router` 函数创建一个新的路由器对象。你会看到我们首先需要 Express，然后做完全一样的事情。

下一个表达式是对路由对象 get 函数的调用。在 Express 中，这是路由级别的中间件。这个函数设置中间件，形式为包含的匿名函数，它响应 HTTP `GET`请求，例如在浏览器地址栏中输入 URL 或点击链接。

函数的第一个参数是要匹配的路径。当路径匹配时，在这种情况下是根路径，函数被调用。这里的匿名函数接收请求对象、响应对象和下一个对象。

函数调用响应对象的 render 函数。这个函数在 views 目录中查找名为 index 的模板并渲染它，传递第二个参数中的对象。模板可以访问该对象的所有属性，在这种情况下，只是标题，并将它们渲染到响应中。

最后，我们将看到`module.exports=router;`表达式。这允许 Node.js 模块系统使用 required 函数加载此代码，并将路由对象分配给一个变量。在我们的`app.js`文件顶部附近，你会看到`var routes = require('./routes/index');`。

## 渲染第一个视图

当你启动 Express 服务器，导航到`localhost:3000`，并看到默认的 Express 页面时，发生了什么？

请求进入 Web 应用程序，根据请求类型`GET`和路径`/`被路由到 index。然后中间件调用响应对象的 render 函数，并告诉它获取 index 模板，传递一个具有名为 title 的属性的对象。

打开你的`views/index.ejs`文件，查看以下模板代码：

```js
<!DOCTYPE html> 
<html> 
  <head> 
    <title><%= title %></title> 
    <link rel='stylesheet' href='/stylesheets/style.css' /> 
  </head> 
  <body> 
    <h1><%= title %></h1> 
    <p>Welcome to <%= title %></p> 
  </body> 
</html> 

```

如果你之前使用过其他动态模板语言，你可能已经理解这很正常，这是包含一些动态元素的普通 HTML。那些被`<%...%>`包含的动态元素由服务器处理。这些*标签*不会发送到浏览器，只是干净的 HTML。

在这种情况下，三个标签是相同的，都渲染了传递给响应对象 render 方法的调用中的对象的标题属性。作为一个快速实验，改变传递的标题值。

# 探索 MongoDB

记住，MEAN 栈中的*M*是一个开源的文档型数据库。因为它不使用 SQL 且不是关系型数据库，所以被认为是一个 NoSQL 数据库。它很好地与基于 JavaScript 的工具集成，因为它不是存储在表中，而是存储在文档中，这些文档可以被我们的 Node.js 应用程序当作 JSON 处理。

### 注意

技术上，MongoDB 以 BSON 格式存储数据，BSON 是 Binary JSON 的缩写。

## 设置 MongoDB

在你的系统上运行 MongoDB 的第一步是安装。访问[`www.mongodb.org/downloads#production`](https://www.mongodb.org/downloads#production)，你将找到 Windows、Mac、Linux 或 Solaris 的最新安装下载。那里也有链接到使用 Homebrew（Mac）和 yum（Linux）等工具安装 MongoDB 的说明。

每个操作系统的最新安装说明可以在 [`docs.mongodb.org/manual/`](https://docs.mongodb.org/manual/) 找到。操作系统之间存在差异，安装说明可能会随着新版本的发布而更改。我建议遵循官方的安装说明。

安装完成后，你可以在控制台中输入 `mongod` 来启动 MongoDB 服务。

当 MongoDB 守护进程运行时，你将无法在该控制台中输入任何其他命令。默认情况下，此进程将在 `端口 27017` 上运行并绑定到 IP 地址 `127.0.0.1`。这两个都可以通过启动时的命令标志或 `.conf` 文件进行更改。就我们的目的而言，默认设置就足够了。

## 使用 MongoDB 命令行界面

我们将开始使用随 MongoDB 安装一起提供的命令行界面来处理 MongoDB。由于你必须运行 MongoDB 守护进程才能与数据库一起工作，因此你需要打开一个新的控制台。

在我们正在构建的应用程序中，我们将依赖 Node.js 插件来处理我们的数据库操作。然而，了解 MongoDB 的工作原理以及它与 SQL 数据库的不同之处是有益的。

要做到这一点，最好的方式是亲自动手，从 MongoDB 命令行界面运行一些基本操作。

### 选择数据库

让我们选择我们想要工作的数据库。打开你的命令行并输入以下命令：

```js
$mongo
> use test
switched to db test
>

```

从命令行运行 `mongo` 而不是 `mongod` 将启动 MongoDB 命令行界面，这允许直接向运行的 MongoDB 守护进程输入命令。

`use` 命令用于选择我们当前正在使用的数据库。但是，等等；我们从未创建过测试数据库。这是正确的，我们确实还没有创建。我们可以使用 `showdbs` 命令列出我们计算机上 MongoDB 所知的所有数据库：

```js
> show dbs
local  0.078GB

```

如果你已经完成了所有前面的示例，那么你已经有了一个名为 `test` 的本地数据库。本地数据库存储启动日志和复制环境中的副本信息。我们可以使用本地数据库并添加我们自己的数据，但这并不是一个好主意。让我们创建一个我们自己的数据库。

### 插入文档

记住，MongoDB 中的记录被称为文档。它们与传统关系型数据库中的行有松散的联系，但更加灵活。

如果我们现在插入一个文档，新数据库将使用以下命令创建：

```js
> db.cat.insert({name:"Tom",color:"grey"})
WriteResult({ "nInserted" : 1 })
> show dbs
local  0.078GB
test   0.078GB

```

`db.cat.insert()` 命令将插入参数中的文档到名为 cat 的集合中。

在 MongoDB 中，集合是一组文档。这类似于关系型数据库中的表，它是一组记录。与关系型数据库不同，集合中的文档不必都是同一类型或具有相同的数据集。

我们可能会收到通知，我们插入的文档看起来像是一个普通的 JavaScript 对象。本质上，它就是这样。这是在 MEAN 栈中将 MongoDB 作为一部分使用时的一个优点——从前端到数据库都是 JavaScript。

当我们输入 `showdbs` 时，我们将看到测试数据库现在已经创建。我们还通过向其中插入文档在测试数据库中创建了一个猫集合。当使用 shell 工作时，这需要小心。很容易因为打字错误而意外创建不需要的数据库和集合。

### 查找文档

现在我们已经有一个数据库，数据库中的一个集合，并且已经将一个文档插入到该集合中，我们需要能够检索该文档。最基本的方法是使用 MongoDB 的查找方法。

让我们看看我们的文档：

```js
> db.cat.find()
{ "_id" : ObjectId("565c010dd9d61e2dc614181f"), "name" : "Tom", "color" : "grey" }

```

`db.collection.find()` 方法是 MongoDB 读取数据的基本方法。它是 MongoDB CRUD 操作中的 *R*——创建、读取、更新、删除。

如您所见，MongoDB 已经为我们添加了一个 `_id` 字段，如下所示：

```js
> db.cat.insert({name:"Bob",color:"orange"})
WriteResult({ "nInserted" : 1 })
> db.cat.find({},{_id:0,name:1})
{ "name" : "Tom" }
{ "name" : "Bob" }

```

我们在这里插入了一只名叫 `Bob` 的新橘猫，这次我们使用的是一种稍微不同的查找方法。它接受两个参数。

第一个参数是查询条件。这告诉 MongoDB 选择哪些文档。在这种情况下，我们使用了一个空对象，因此选择了所有文档。

第二个参数是投影，它限制了 MongoDB 返回的数据量。我们告诉 MongoDB 我们想要名称字段，但抑制默认会返回的 `_id` 字段。

在下一章中，我们将探讨如何限制返回的文档数量并对其进行排序。

看看以下命令：

```js
> db.cat.find({color:"orange"},{_id:0,name:1})
{ "name" : "Bob" }

```

我们已经调整了查询，使其包含查询条件，仅选择颜色为橘色的猫。`Bob` 是唯一的橘猫，因此这是唯一返回的文档。同样，我们将抑制 `_id` 并告诉 MongoDB 我们只想看到名称字段。

### 更新文档

如果我们想更改数据库中的记录怎么办？有几种方法可以做到这一点，但我们将从最简单的一种开始。MongoDB 提供了一个名为 update 的方法来修改数据库中的文档，如下所示：

```js
> db.cat.update({name:"Bob"},{$set:{color:"purple"}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })
> db.cat.find({color:"orange"},{_id:0,name:1})
> db.cat.find({color:"purple"},{_id:0,name:1})
{ "name" : "Bob" }

```

与查找方法类似，更新方法的第一个参数告诉 MongoDB 选择哪些文档。然而，默认情况下，MongoDB 将一次只更新一个文档。要更新多个文档，我们将添加第三个参数，一个修饰符对象，`{multi:true}`。

我们选择了名称字段等于 `Bob`（只有一个）的文档。然后，我们将使用 `$set` 操作符将 `Bob` 的颜色改为紫色。

我们可以通过使用以下命令查询 `orange` 猫来验证这一点：

```js
> db.cat.find({color:"orange"},{_id:0,name:1})
> db.cat.find({color:"purple"},{_id:0,name:1})
{ "name" : "Bob" }

```

没有文档返回。现在对紫色 `cat` 的查询返回了名为 `Bob` 的 `cat` 文档。

### 删除文档

没有一组 CRUD 操作是完整的，没有 *D* 或删除操作。MongoDB 提供了 remove 方法来实现这一点，如下所示：

```js
> db.cat.remove({color:"purple"})
WriteResult({ "nRemoved" : 1 })
> db.cat.find({},{_id:0,name:1})
{ "name" : "Tom" }

```

remove 方法具有与其他 MongoDB CRUD 方法相当类似的签名。第一个参数，正如你可能推测的那样，是一个选择器，用于选择 MongoDB 应该移除哪些文档。

在前面的示例中，我们移除了所有包含颜色属性`purple`值的文档。只有一个，所以再见了可怜的`Bob`。我们通过调用 find 方法进行了验证，现在它只返回`Tom`。

remove 方法的默认操作是删除所有找到的文档，所以请谨慎使用。传递一个空的选择器将删除集合中的所有文档，如下所示：

```js
> db.cat.remove({})
WriteResult({ "nRemoved" : 1 })
> db.cat.find()
>

```

就这样，我们没有了猫。

MongoDB 没有内置的回滚功能，所以没有真正的方法可以撤销这样的删除。在生产环境中，这是为什么复制和定期数据库备份很重要的原因之一。

## 创建你的 SPA 数据库

因此，现在我们已经对 MongoDB 进行了一些操作，让我们创建我们将用于构建 SPA 的开发数据库。

在后面的章节中，我们将使用一个名为*mongoose to model*的 Node.js 插件来验证、查询和操作我们的数据。

现在，让我们只设置数据库。在你的 mongo shell 中，输入以下命令：

```js
> use giftapp
switched to db giftapp

```

记住，我们实际上还没有创建数据库。为此，我们需要将一个文档放入一个集合中。由于我们将在以后让 Mongoose 为我们做所有繁重的工作，我们将先在测试集合中放入一些内容以开始。以下代码作为示例：

```js
> db.test.insert({test:"here is the first document in the new database"})
WriteResult({ "nInserted" : 1 })
> db.test.find()
{ "_id" : ObjectId("565ce0a2d9d61e2dc6141821"), "test" : "here is the first document in the new database" }
> show dbs
giftapp  0.078GB
local    0.078GB
test     0.078GB

```

在这里，我们将向测试集合中插入一个文档。我们可以通过调用 find 方法来验证它是否在那里。最后，我们将运行`showdbs`并查看我们的`giftapp`数据库是否成功创建。

# 从 AngularJS 开始

我们现在已经将应用堆栈的大部分组件放在一起了。我们有一个运行时环境，Node.js。我们已经安装并设置了 Web 应用框架 Express。我们刚刚设置了数据库，MongoDB。

在任何 SPA 中缺失的一个关键部分是——无论后端堆栈看起来如何，都有一个前端框架来使 SPA 魔法发生。

有许多用于 SPAs 的前端库和框架，但最受欢迎的，也是 MEAN 中的*A*，是 AngularJS，也称为 Angular。

Angular 是一个开源的前端框架，特别适合构建 SPA。它非常受欢迎，目前由 Google 维护。

### 注意

在这本书中，我们将使用 AngularJS 版本 1.4.8，该版本于 2015 年 11 月发布。AngularJS 2.0 于 2014 年宣布，在出版时，刚刚推出了生产版本。2.0 版本引入了破坏性的非向后兼容更改。今天的大多数开发都是使用 1.x 分支上的某个版本进行的，计划在未来继续支持。

## 将 AngularJS 安装到应用中

最终，Angular 是一个包含一些可选插件文件的单一 JavaScript 文件。有几种方法可以将 Angular 添加到您的前端应用程序中。您可以下载您感兴趣的包，或者使用 `Bower` 这样的工具来加载它。

我们开始的最简单方式是直接指向公开可用的 CDN 上的 Angular 文件。我们可以从 [`cdnjs.cloudflare.com/ajax/libs/angular.js/1.4.8/angular.js`](https://cdnjs.cloudflare.com/ajax/libs/angular.js/1.4.8/angular.js) 包含 Angular 1.4.8。

让我们打开我们的 `index.ejs` 文件，并包含一个链接到以下文件的脚本标签：

```js
<!DOCTYPE html> 
<html> 
  <head> 
    <title><%= title %></title> 
    <link rel='stylesheet' href='/stylesheets/style.css' /> 
  </head> 
  <body> 
    <h1><%= title %></h1> 
    <p>Welcome to <%= title %></p> 
 <script src="img/angular.js"></script> 
  </body> 
</html> 

```

### 小贴士

当可能时，将链接到外部 JavaScript 文件的脚本标签包含在 HTML 的关闭 body 标签之前是一个好的做法。这是出于性能考虑。您可以在 [`stevesouders.com/hpws/rule-js-bottom.php`](http://stevesouders.com/hpws/rule-js-bottom.php) 上了解更多信息，并查看一些展示这一做法理由的示例。

如果您重新启动服务器并加载 `localhost:3000` 上的默认页面，您不会看到太多。您正在加载 AngularJS，但还没有明显的效果。我们将分层次构建一些示例前端代码。

在添加 AngularJS 脚本后，我们首先想要做的是将 `html` 标签修改为以下代码所示：

```js
<html ng-app> 

```

头部元素上的 `ng-app` 属性被称为 Angular 指令。当 Angular 加载到页面时，它会进入一个引导阶段，寻找如 `ng-app` 这样的指令。这个指令告诉 Angular 应用程序的根元素在哪里。

您可以将此属性视为标记一个 Angular 将要管理的区域的方法。在我们的例子中，我们希望在整个页面上使用 Angular 组件，因此我们将 HTML 元素声明为根元素。

在下一节中，我们将构建一个将成为我们的根应用程序的模块。

## 构建 AngularJS 的第一个模块

AngularJS 是设计成模块化的，并包含一个注册模块的功能。模块充当其他对象的容器，例如 Angular 服务、指令和控制器。

Angular 模块也是可注入的。这意味着模块可以被注入到其他模块中并被消费。Angular 有一个独特的依赖注入系统，我们很快就会看到并大量使用。

在 `giftapp` 的 `public/javavscripts` 目录下创建一个新的 JavaScript 文件，命名为 `app.js`，并将以下代码输入到其中：

```js
var giftAppModule = angular.module('giftAppModule', []); 

giftAppModule.value('appName', 'GiftApp'); 

```

第一行使用 `angular.module()` 函数创建并注册了一个 Angular 模块。此函数的第一个参数是一个字符串，Angular 将使用它作为模块的名称，`giftAppModule`。第二个参数是我们希望注入到该模块中的依赖项数组。目前我们没有，所以数组是空的。

然后我们将模块分配给`giftAppModule`变量。尽管这个变量名恰好与模块名相同，但它是不相关的；我们可以将其命名为任何其他名称。你不必将模块分配给一个变量名，但这样做很有用，因为它允许我们更干净地添加资产到模块中。

下一行，`giftAppModule.value('appName','GiftApp');`通过调用`value`函数在模块上创建了一个新的服务。Angular 中的服务是一个单例，它可以被注入。Angular 包含多种类型的服务。值服务是最简单的一种，它创建一个可以被注入和使用的名称值对。我们将在我们的控制器中使用它。

最后，我们希望在`index.ejs`模板中将我们的新模块加载并作为根 Angular 应用程序`Bootstrap`。

看看下面的代码：

```js
<!DOCTYPE html> 
<html ng-app="giftAppModule"> 
  <head> 
    <title><%= title %></title> 
    <link rel='stylesheet' href='/stylesheets/style.css' /> 
  </head> 
  <body> 
    <h1><%= title %></h1> 
    <p>Welcome to <%= title %></p> 
    <script src="img/angular.js"></script> 
    <script src="img/app.js"></script> 
  </body> 
</html> 

```

这里有两个重要的更改需要注意。首先，在 HTML 元素中，我们通过将`ng-app`指令的值设置为`giftAppModule`来添加了对我们刚刚创建的`giftAppModule`的引用。下一个更改是在关闭`body`标签之前添加了一个新的`script`标签，该标签加载了我们刚刚创建的`app.js`文件。

加载顺序在这里很重要，Angular 必须在`app.js`之前加载，否则会失败。注意我们用来加载`app.js`的路径：`/javascripts/app.js`。这之所以有效，是因为 Express `app.js`中的一段代码将静态文件请求指向公共目录，`app.use(express.static(path.join(__dirname,'public')));`。

如果服务器停止，启动服务器并重新加载页面在这个阶段没有任何可见的变化。要开始对页面进行更改，我们需要添加一个控制器和一个 Angular 表达式。

## 添加控制器

在 Angular 中，控制器是一个 JavaScript 对象，它通过 Angular 的`$scope`对象向视图公开数据和功能。

打开你的`app.js`文件，并添加以下代码：

```js
var giftAppModule = angular.module('giftAppModule', []); 

giftAppModule.value('appName', 'GiftApp'); 

giftAppModule.controller("GreetingController", ['$scope','appName', function($scope)
{ 
 $scope.name = appName; 
 $scope.greeting = "Hello Angular" 
 }
] 
);

```

附加的代码在`giftAppModule`上创建了一个名为`GreetingController`的控制器构造函数。这还不是真正的控制器，直到它在页面上使用`ng-controller`指令被调用时才成为如此。

函数的第一个参数是控制器的名称。第二个参数是一个数组，包含我们希望注入的依赖项。数组中的最后一项是函数本身。Angular 文档称这为数组注解，并且这是创建构造函数的首选方法。

数组第一部分的模块名称字符串映射到函数的参数。每个参数的顺序必须相同。

接下来，我们需要将控制器添加到我们的`index.ejs html`中，如下所示：

```js
<!DOCTYPE html> 
<html ng-app="giftAppModule"> 
  <head> 
    <title><%= title %></title> 
    <link rel='stylesheet' href='/stylesheets/style.css' /> 
  </head> 
  <body> 
    <h1><%= title %></h1> 
    <p>Welcome to <%= title %></p> 
    <div ng-controller="GreetingController"> 

    </div> 
    <script src="img/angular.js"></script> 
    <script src="img/app.js"></script> 
  </body> 
</html> 

```

在这里，我们将添加一个`div`标签，并给它一个值为`GreetingController`的`ng-controller`属性。当我们加载这个页面时，Angular 将创建一个新的`GreetingController`对象，并将一个子作用域附加到页面的这部分。

再次，如果你在浏览器中刷新，你将不会看到任何不同。通常，为了向用户显示数据，你会使用 Angular 表达式。

## 使用 Angular 表达式显示数据

Angular 表达式是放置在双大括号`{{}}`之间的代码片段。Angular 评估这些（JavaScript 的`eval()`函数没有被使用，因为它不是一个安全的机制）。

Angular 文档将表达式称为 JavaScript，因为它们有一些相当大的差异。例如，Angular 表达式没有`控制循环`。

Angular 表达式是在当前作用域的上下文中评估的。

让我们对`index.ejs`文件进行以下修改：

```js
<!DOCTYPE html> 
<html ng-app="giftAppModule"> 
  <head> 
    <title><%= title %></title> 
    <link rel='stylesheet' href='/stylesheets/style.css' /> 
  </head> 
  <body> 
    <h1><%= title %></h1> 
    <p>Welcome to <%= title %></p> 
    <div ng-controller="GreetingController"> 
       {{ greeting }} from {{ name }} {{ 2+3 }} 
    </div> 
    <script src="img/angular.js"></script> 
    <script src="img/app.js"></script> 
  </body> 
</html> 

```

我们在`div`中添加了一些 Angular 表达式，其中调用了`GreetingController`。Angular 使得`GreetingController`的作用域对象在`div`内部可用。

现在，如果你重新加载这个页面，在**欢迎使用 Express**下，你应该看到以下行代码：

```js
Hello Angular from GiftApp 5 

```

Angular 评估了表达式并将它们作为字符串显示。包含问候和名称的表达式从作用域中提取那些值。最后的表达式只是进行了一点算术运算。

## 双向数据绑定

Angular 提供的主要功能之一被称为双向数据绑定。这意味着在视图中更改数据会更新模型中的数据。同样，模型中更改的数据会在视图中反映出来。

打开`app.js`并添加以下属性到作用域中：

```js
var giftAppModule = angular.module('giftAppModule', []); 

giftAppModule.value('appName', 'GiftApp'); 

giftAppModule.controller("GreetingController", ['$scope','appName', function($scope, appName){ 
        $scope.name = appName; 
        $scope.greeting = "Hello Angular"; 
        $scope.newName = "Bob"; 
    }] 
); 

```

我们添加了一个`newName`属性并将其赋值为字符串`Bob`。

现在，我们需要对`index.ejs`文件进行以下修改：

```js
<!DOCTYPE html> 
<html ng-app="giftAppModule"> 
  <head> 
    <title><%= title %></title> 
    <link rel='stylesheet' href='/stylesheets/style.css' /> 
  </head> 
  <body> 
    <h1><%= title %></h1> 
    <p>Welcome to <%= title %></p> 
    <div ng-controller="GreetingController"> 
       {{ greeting }} from {{ name }} {{ 2+3 }} {{ newName }} 
        <p><input type=""text" ng-model="newName"></p> 
    </div> 
    <script src="img/angular.js"></script> 
    <script src="img/app.js"></script> 
  </body> 
</html> 

```

我们对这个文件做了两个修改。第一个是我们在算术表达式之后添加了`{{newName}}`表达式。这将在屏幕上渲染`Bob`字符串。第二个修改是添加了一个文本输入控件，并添加了`ng-model="newName"`指令。这个指令将文本框中的值绑定到作用域上的`newName`属性。

当页面加载时，文本框中的值是`Bob`。但如果我们在文本框中输入除了`Bob`之外的内容会发生什么呢？表达式的渲染值几乎瞬间改变。

这是一个双向数据绑定的明显例子。视图中的数据更改无缝地影响模型。

# 摘要

在本章中，你学习了如何仅使用基于 JavaScript 的工具从数据库到前端构建一个全栈应用程序。在前面的章节中，我们探讨了 MEAN 栈组件。现在我们已经开始将它们组合起来。

你从查看 Node.js 开始，这是我们的基于 JavaScript 的运行环境。你使用了 Node.js 的 REPL 在命令行上执行 JavaScript 代码。然后你编写了一个脚本，一个小型 Web 服务器，它可以由 Node.js 运行

你学习了两种设置 Express 应用程序的方法。此外，你还使用了 express generator 来构建一个功能框架以构建应用程序。你了解了路由和中间件——Express 的两个关键组件。

MongoDB 是一个 NoSQL 数据库，它将数据作为灵活的文档存储在集合中，而不是像关系数据库那样存储在记录/表模型中。你已经在 Mongo 中运行了每个基本的 CRUD（创建、读取、更新、删除）方法，即插入、查找、更新和删除。

在下一章中，我们将深入探讨 MongoDB，通过命令行界面获得经验。
