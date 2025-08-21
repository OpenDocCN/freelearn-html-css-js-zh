# 第二章：创建一个简单的 Web 应用程序

当您到达本章末尾时，您应该能够使用 Node.js、CouchDB 和 Flatiron 创建一个简单的 Web 应用程序。

本章涵盖的主题包括：

+   设置 Node 和 Flatiron

+   创建和处理用户表单

# 定义我们的 Web 应用程序的要求

在我们深入研究 Zombie.js 世界之前，我们需要为我们的测试创建一个目标，即提供待办事项列表的 Web 应用程序。这是这样一个应用程序的顶级要求集：

+   用户可以注册该服务，需要提供电子邮件地址作为用户名和密码。通过提供用户名和密码，用户可以创建一个经过身份验证的会话，该会话将在进一步的交互中识别他。

+   用户可以创建一个待办事项。

+   用户可以查看待办事项列表。

+   用户可以删除待办事项。

为了实现这个应用程序，我们将使用 Node.js，这是一个用 JavaScript 构建网络应用程序的平台，Zombie.js 也使用它。我们还将使用 Flatiron，这是一组组件，将帮助您在 Node.js 之上构建 Web 应用程序。

### 注意

为了保持简单，我们正在使用 Node.js 构建我们的应用程序。但是，Zombie.js 适用于测试使用任何框架构建的应用程序，这些框架利用动态 HTTP 服务器。

还要记住，构建这个 Web 应用程序的目标不是向您展示如何构建 Web 应用程序，而是在已知和简单的域上提供一个可用的应用程序，以便我们可以将其用作我们测试的主题。

在接下来的章节中，您将学习如何安装 Node.js 和 Flatiron，以及如何创建您的待办应用程序服务器。

# 设置 Node.js 和 Flatiron

如果您没有安装最新版本的 Node.js，您将需要安装它。您将需要 Node.js 出于几个原因。我们的 Web 应用程序将使用 Flatiron，它在 Node.js 之上运行。您还需要使用**Node Package Manager**（**NPM**），它与 Node 捆绑在一起。最后，您将需要 Node.js 来安装和运行 Zombie.js 测试。

## 安装 Node.js

1.  要安装 Node.js，请前往 nodejs.org 网站。![安装 Node.js](img/0526OS_02_01.jpg)

1.  然后点击**下载**按钮，这将打开以下页面：![安装 Node.js](img/0526OS_02_02.jpg)

1.  如果您正在运行 Windows 或 Macintosh 系统，请单击相应的安装程序图标。这将下载并启动图形安装程序。

### 从源代码安装 Node

如果您没有运行其中一个系统，并且您在类 Unix 系统上，您可以按照以下步骤从源代码安装 Node.js：

1.  单击源代码图标，将开始下载源代码 tarball。下载完成后，使用终端展开它：

```js
$ tar xvfz node-v0.8.7.tar.gz
```

导航到创建的目录：

```js
$ cd node-v0.8.7
```

1.  配置它：

```js
$ ./configure
```

1.  构建它：

```js
$ make
```

1.  最后安装它：

```js
$ make install
```

如果您没有足够的权限将节点二进制文件复制到最终目标位置，您将需要在命令前加上`sudo`：

```js
$ sudo make install
```

1.  现在您应该已经在系统上安装了 Node.js。尝试运行它：

```js
$ node -v
v0.8.7
```

1.  现在让我们尝试打开 Node 命令行并输入一些内容：

```js
$ node
> console.log('Hello World!');
```

1.  如果您现在按*Enter*键，您应该会得到以下输出：

```js
...
> Hello World!
```

1.  通过安装 Node.js，您还安装了它的忠实伴侣 NPM，Node Package Manager。您可以尝试从终端调用它：

```js
$ npm -v
1.1.48
```

## 安装 Flatiron 并启动您的应用程序

现在您需要安装 Flatiron 框架，这样您就可以开始构建您的应用程序。

1.  使用 NPM 按照以下方式下载和安装 Flatiron：

```js
$ npm install -g flatiron
```

### 注意

再次，如果您没有足够的权限安装 Flatiron，请在最后一个命令前加上`sudo`。

这将全局安装 Flatiron，使`flatiron`命令行实用程序可用。

1.  现在您应该进入一个将拥有应用程序代码的目录。然后，您可以通过执行以下命令为您的 Web 应用程序创建基本的脚手架：

```js
$ flatiron create todo
```

1.  在提示您输入作者的姓名、应用程序描述和主页（可选）后，它将创建一个名为`todo`的目录，其中包含您的应用程序代码的基础。使用以下命令进入该目录：

```js
$ cd todo
```

在那里，您将找到两个文件和三个文件夹：

```js
$ tree
.
├── app.js
├── config
│   └── config.json
├── lib
├── package.json
└── test
```

其中一个文件`package.json`包含应用程序清单，其中，除其他字段外，还包含应用程序依赖的软件包。现在，您将从该文件中删除`devDependencies`字段。

您还需要为名为`plates`的软件包添加一个依赖项，该软件包将用于动态更改 HTML 模板。

此外，您将为一些不需要任何修改的静态文件提供服务。为此，您将使用一个名为`node-static`的包，您还需要将其添加到应用程序清单的依赖项列表中。

到目前为止，您的`package.json`应该看起来像这样：

```js
{
  "description": "To-do App",
  "version": "0.0.0",
  "private": true,
  "dependencies": {
    "union": "0.3.0",
    "flatiron": "0.2.8",
    "plates": "0.4.x",
    "node-static": "0.6.0"
  },
  "scripts": {
    "test": "vows --spec",
    "start": "node app.js"
  },
  "name": "todo",
  "author": "Pedro",
  "homepage": ""
}
```

1.  接下来，通过以下方式安装这些依赖项：

```js
$ npm install
```

这将在本地的`node_modules`目录中安装所有依赖项，并应该输出类似以下内容：

```js
union@0.3.0 node_modules/union
├── qs@0.4.2
└── pkginfo@0.2.3

flatiron@0.2.8 node_modules/flatiron
├── pkginfo@0.2.3
├── director@1.1.0
├── optimist@0.3.4 (wordwrap@0.0.2)
├── broadway@0.2.5 (eventemitter2@0.4.9, cliff@0.1.8, utile@0.1.2, nconf@0.6.4, winston@0.6.2)
└── prompt@0.2.6 (revalidator@0.1.2, read@1.0.4, utile@0.1.3, winston@0.6.2)

plates@0.4.6 node_modules/plates

node-static@0.6.0 node_modules/node-static
```

### 注意

您不必担心这一点，因为 Node 将能够自动获取这些依赖项。

1.  现在您可以尝试启动您的应用程序：

```js
$ node app.js
```

如果您打开浏览器并将其指向`http://localhost:3000`，您将得到以下响应：

```js
{"hello":"world"}
```

# 创建您的待办事项应用程序

现在，您已经有一个 Flatiron“hello world”示例正在运行，您需要扩展它，以便我们的待办事项应用程序成形。为此，您需要创建和更改一些文件。如果您迷失了方向，您可以随时参考本章的源代码。另外，供您参考，本章末尾包含了项目文件的完整列表。

## 设置数据库

与任何真实应用程序一样，您将需要一种可靠的方式来持久保存数据。在这里，我们将使用 CouchDB，这是一个开源的面向文档的数据库。您可以选择在本地安装 CouchDB，也可以使用互联网上的服务，如 Iris Couch。

如果您选择在本地开发机器上安装 CouchDB，您可以前往[`couchdb.apache.org/`](http://couchdb.apache.org/)，点击**下载**并按照说明进行操作。

如果您更喜欢简单地通过互联网使用 CouchDB，您可以前往[`www.iriscouch.com/`](http://www.iriscouch.com/)，点击**立即注册**按钮并填写注册表格。您应该在几秒钟内拥有一个运行的 CouchDB 实例。

![设置数据库](img/0526OS_02_03.jpg)

### 注意

截至目前，Iris Couch 是一个免费为小型数据库提供低流量服务的服务，这使其非常适合原型设计这样的应用程序。

## 从 Node 访问 CouchDB

要从 Node 访问 CouchDB 数据库，我们将使用一个名为`nano`的库，您将把它添加到`package.json`文件的依赖项部分：

```js
{
  "description": "To-do App",
  "version": "0.0.0",
  "private": true,
  "dependencies": {
    "union": "0.3.0",
    "flatiron": "0.2.8",
    "plates": "0.4.6",
    "node-static": "0.6.0",
 "nano": "3.3.0"
  },
  "scripts": {
    "test": "vows --spec",
    "start": "node app.js"
  },
  "name": "todo",
  "author": "Pedro",
  "homepage": ""
}
```

现在，您可以通过在应用程序的根目录运行以下命令来安装此缺少的依赖项：

```js
$ npm install
nano@3.3.0 node_modules/nano
├── errs@0.2.3
├── request@2.9.203.8.0 (request@2.2.9request@2.2.9)
```

这将在`node_modules`文件夹中安装`nano`，使其在构建此应用程序时可用。

要实际连接到数据库，您需要定义 CouchDB 服务器的 URL。如果您在本地运行 CouchDB，则 URL 应类似于`ht` `tp://127.0.0.1:5984`。如果您在 Iris Couch 或类似的服务中运行 CouchDB，则您的 URL 将类似于`https://mytodoappcouchdb.iriscouch.com`。

在任何这些情况下，如果您需要使用用户名和密码进行访问，您应该将它们编码在 URL 中，`http://username:password@mytodoappco` `uchdb.iriscouch.com`

现在应该将此 URL 输入到`config/config.json`文件的配置文件中，`couchdb`键下：

```js
{
  "couchdb": "http://localhost:5984"
}
```

接下来，通过在`lib/couchdb.js`下提供一个简单的模块来封装对数据库的访问：

```js
var nano = require('nano'),
    config = require('../config/config.json');

module.exports = nano(config.couchdb);
```

此模块将用于获取 CouchDB 服务器对象，而不是在整个代码中多次重复`config`和`nano`的操作。

## 应用程序布局

像许多网站现在所做的那样，我们将使用 Twitter Bootstrap 框架来帮助我们使网站看起来和感觉起来简洁而又可观。为此，您将前往 Bootstrap 网站[`twitter.github.com/bootstrap/`](http://twitter.github.com/bootstrap/)，并单击**下载 Bootstrap**按钮：

![应用程序布局](img/0526OS_02_04.jpg)

您将收到一个 zip 文件，您应该将其扩展到本地的`public`文件夹中，最终得到这些文件：

```js
$ tree public/
public/
├── css
│   ├── bootstrap-responsive.css
│   ├── bootstrap-responsive.min.css
│   ├── bootstrap.css
│   └── bootstrap.min.css
├── img
│   ├── glyphicons-halflings-white.png
│   └── glyphicons-halflings.png
└── js
    ├── bootstrap.js
    └── bootstrap.min.js
```

您还需要将 jQuery 添加到混合中，因为 Bootstrap 依赖于它。从[`jquery.com`](http://jquery.com)下载 jQuery，并将其命名为`public/js/jquery.min.js`。

## 开发前端

现在我们安装了 Bootstrap 和 jQuery，是时候创建我们应用程序的前端了。

首先，我们将设置布局 HTML 模板，该模板定义了所有页面的外部结构。为了托管所有模板，我们将有一个名为`templates`的目录，其中包含以下内容`templates/layout.html`：

```js
<html>
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <title id="title"></title>
    <link href="/css/bootstrap.min.css" rel="stylesheet" />
  </head>
  <body>

    <section role="main" class="container">

      <div id="messages"></div>

      <div id="main-body"></div>

    </section>

    <script src="img/jquery.min.js"></script> 
    <script src="img/bootstrap.min.js"></script>

  </body>
</html>
```

此模板加载 CSS 和脚本，并包含消息和主要部分的占位符。

我们还需要一个小模块，该模块获取主要内容和一些其他选项，并将它们应用于此模板。我们将其放在`templates/layout.js`中：

```js
var Plates = require('plates'),
    fs     = require('fs');

var templates = {
  layout : fs.readFileSync(__dirname + '/layout.html', 'utf8'),
  alert  : fs.readFileSync(__dirname + '/alert.html', 'utf8')
};

module.exports = function(main, title, options) {

  if (! options) {
    options = {};
  }

  var data = {
    "main-body": main,
    "title": title,
    'messages': ''
  };

  ['error', 'info'].forEach(function(messageType) {
    if (options[messageType]) {
      data.messages += Plates.bind(templates.alert,
        {message: options[messageType]});
    }
  });

  return Plates.bind(templates.layout, data);
};
```

在 Node.js 中，模块只是一个旨在被其他模块使用的 JavaScript 文件。模块内的所有变量都是私有的；如果模块作者希望向外部世界公开值或函数，它会修改或设置`module.exports`中的特殊变量。

在我们的情况下，这个模块导出一个函数，该函数获取主页面内容的标记，页面标题和一些选项，如信息或错误消息，并将其应用于布局模板。

我们还需要将以下标记文件放在`templates/alert.html`下：

```js
<div class="alert">
  <a class="close" data-dismiss="alert">×</a>
  <p class="message"></p>
</div>
```

现在我们准备开始实现一些要求。

## 用户注册

这个应用程序将为用户提供一个个人待办事项列表。在他们可以访问它之前，他们需要在系统中注册。为此，您需要定义一些 URL，用户将使用这些 URL 来获取我们的用户注册表单并提交它。

现在您将更改`app.js`文件。此文件包含一组初始化过程，包括此块：

```js
app.router.get('/', function () {
  this.res.json({ 'hello': 'world' })
});
```

这个块正在将所有具有`/`URL 的 HTTP 请求路由，并且 HTTP 方法是`GET`到给定的函数。然后，对于具有这两个特征的每个请求，将调用此函数，在这种情况下，您正在回复`{"hello":"world"}`，用户将在浏览器上看到打印出来。

现在我们需要删除这个路由，并添加一些路由，允许用户注册自己。

为此，创建一个名为`routes`的文件夹，您将在其中放置所有路由模块。第一个是`routes/users.js`，将包含以下代码：

```js
var fs      = require('fs'),
    couchdb = require('../lib/couchdb'),
    dbName  = 'users',
    db      = couchdb.use(dbName),
    Plates  = require('plates'),
    layout  = require('../templates/layout');

var templates = {
  'new' : fs.readFileSync(__dirname +
    '/../templates/users/new.html', 'utf8'),
  'show': fs.readFileSync(__dirname +
    '/../templates/users/show.html', 'utf8')
};

function insert(doc, key, callback) {
  var tried = 0, lastError;

  (function doInsert() {
    tried ++;
    if (tried >= 2) {
      return callback(lastError);
    }

    db.insert(doc, key, function(err) {
      if (err) {
        lastError = err;
        if (err.status_code === 404) {
          couchdb.db.create(dbName, function(err) {
            if (err) {
              return callback(err);
            }
            doInsert();
          });
        } else {
          return callback(err);
        }
      }
      callback.apply({}, arguments);
    });
  }());
}

function render(user) {
  var map = Plates.Map();
  map.where('id').is('email').use('email').as('value');
  map.where('id').is('password').use('password').as('value');
  return Plates.bind(templates['new'], user || {}, map);
}

module.exports = function() {
  this.get('/new', function() {
    this.res.writeHead(200, {'Content-Type': 'text/html'});
    this.res.end(layout(render(), 'New User'));
  });

  this.post('/', function() {

    var res = this.res,
        user = this.req.body;

    if (! user.email || ! user.password) {
      return this.res.end(layout(templates['new'],
        'New User', {error: 'Incomplete User Data'}));
    }

    insert(user, this.req.body.email, function(err) {
      if (err) {
        if (err.status_code === 409) {
          return res.end(layout(render(user), 'New User', {
            error: 'We already have a user with that email address.'}));
        }
        console.error(err.trace);
        res.writeHead(500, {'Content-Type': 'text/html'});
        return res.end(err.message);
      }
      res.writeHead(200, {'Content-Type': 'text/html'});
      res.end(layout(templates['show'], 'Registration Complete'));
    });
  });

};
```

这个新模块导出一个函数，将绑定两个新路由`GET /new`和`POST /`。这些路由稍后将被附加到`/users`命名空间，这意味着当服务器接收到`GET`请求`/users/new`和`POST`请求`/users`时，它们将被激活。

在`GET /new`路由上，我们将呈现一个包含用户表单的模板。将其放在`templates/users/new.html`下：

```js
<h1>New User</h1>
<form action="/users" method="POST">
  <p>
    <label for="email">E-mail</label>
    <input type="email" name="email" value="" id="email" />
  </p>
  <p>
    <label for="password">Password</label>
    <input type="password" name="password" id="password" value="" required/>
  </p>
  <input type="submit" value="Submit" />
</form>
```

我们还需要创建一个`感谢您注册`模板，您需要将其放在`templates/users/show.html`中：

```js
<h1>Thank you!</h1>
<p>Thank you for registering. You can now <a href="/session/new">log in here</a></p>
```

在`POST /`路由处理程序中，我们将进行一些简单的验证，并通过调用名为`insert`的函数将用户文档插入 CouchDB 数据库。此函数尝试插入用户文档，并利用一些巧妙的错误处理。如果错误是“404 Not Found”，这意味着`users`数据库尚未创建，我们将利用这个机会创建它，并自动重复用户文档插入。

您还捕获了 409 冲突的 HTTP 状态码，如果我们尝试插入已存在的键的文档，CouchDB 将返回此状态码。由于我们使用用户电子邮件作为文档键，因此我们通知用户该用户名已经存在。

### 注意

在这里，除了其他简化之外，您将用户密码以明文存储在数据库中。这显然是不推荐的，但由于本书的核心不是如何创建 Web 应用程序，因此这个实现细节与您的目标无关。

现在，我们需要通过更新并在`app.js`文件中的`app.start(3000)`之前添加一行来将这些新路由附加到`/users/` URL 命名空间：

```js
var flatiron = require('flatiron'),
    path = require('path'),
    nstatic = require('node-static'),
    app = flatiron.app;

app.config.file({ file: path.join(__dirname, 'config', 'config.json') });

var file = new nstatic.Server(__dirname + '/public/');

app.use(flatiron.plugins.http, {
  before: [
    function(req, res) {
      var found = app.router.dispatch(req, res);
      if (! found) {
        file.serve(req, res);
      }
    }
  ]
});

app.router.path('/users', require('./routes/users'));

app.start(3000);
```

现在，您可以通过在命令行中输入以下命令来启动应用程序：

```js
$ node app
```

这将启动服务器。然后打开 Web 浏览器，访问`http://localhost:3000/users/new`。您将获得一个用户表单：

![用户注册](img/0526OS_02_05.jpg)

提交电子邮件和密码，您将获得一个确认屏幕：

![用户注册](img/0526OS_02_06.jpg)

此屏幕将向您显示一个链接，指向尚不存在的`/session/new` URL。

现在，您已经准备好实现登录屏幕。

## 登录和会话管理

为了能够保持会话，您的 HTTP 服务器需要能够执行两件事：解析 cookie 和存储会话数据。为此，我们使用两个模块，即`flatware-cookie-parser`和`flatware-session`，您应该将它们添加到`package.json`清单中：

```js
{
  "description": "To-do App",
  "version": "0.0.0",
  "private": true,
  "dependencies": {
    "union": "0.3.0",
    "flatiron": "0.2.8",
    "plates": "0.4.x",
    "node-static": "0.6.0",
    "nano": "3.3.0",
 "flatware-cookie-parser": "0.1.x",
 "flatware-session": "0.1.x"
  },
  "scripts": {
    "test": "vows --spec",
    "start": "node app.js"
  },
  "name": "todo",
  "author": "Pedro",
  "homepage": ""
}
```

现在，安装缺少的依赖项：

```js
$ npm install
flatware-cookie-parser@0.1.0 node_modules/flatware-cookie-parser

flatware-session@0.1.0 node_modules/flatware-session
```

接下来，在文件`app.js`中向您的服务器添加这些中间件组件：

```js
var flatiron = require('flatiron'),
    path = require('path'),
    nstatic = require('node-static'),
    app = flatiron.app;

app.config.file({ file: path.join(__dirname, 'config', 'config.json') });

var file = new nstatic.Server(__dirname + '/public/');

app.use(flatiron.plugins.http, {
  before: [
 require('flatware-cookie-parser')(),
 require('flatware-session')(),
    function(req, res) {
      var found = app.router.dispatch(req, res);
      if (! found) {
        file.serve(req, res);
      }
    }
  ]
});

app.router.path('/users', require('./routes/users'));
app.router.path('/session', require('./routes/session'));

app.start(3000);
```

我们还需要创建一个`routes/session.js`模块来处理新的会话路由：

```js
var plates  = require('plates'),
    fs      = require('fs'),
    couchdb = require('../lib/couchdb'),
    dbName  = 'users',
    db      = couchdb.use(dbName),
    Plates  = require('plates'),
    layout  = require('../templates/layout');

var templates = {
  'new' : fs.readFileSync(__dirname +
    '/../templates/session/new.html', 'utf8')
};

module.exports = function() {

  this.get('/new', function() {
    this.res.writeHead(200, {'Content-Type': 'text/html'});
    this.res.end(layout(templates['new'], 'Log In'));
  });

  this.post('/', function() {

    var res   = this.res,
        req   = this.req,
        login = this.req.body;

    if (! login.email || ! login.password) {
      return res.end(layout(templates['new'], 'Log In',
        {error: 'Incomplete Login Data'}));
    }

    db.get(login.email, function(err, user) {
      if (err) {
        if (err.status_code === 404) {
          // User was not found
          return res.end(layout(templates['new'], 'Log In',
            {error: 'No such user'}));
        }
        console.error(err.trace);
        res.writeHead(500, {'Content-Type': 'text/html'});
        return res.end(err.message);
      }

      if (user.password !== login.password) {
        res.writeHead(403, {'Content-Type': 'text/html'});
        return res.end(layout(templates['new'], 'Log In',
            {error: 'Invalid password'}));
      }

      // store session
      req.session.user = user;

      // redirect user to TODO list
      res.writeHead(302, {Location: '/todos'});
      res.end();
    });

  });  

};
```

现在，我们需要在`templates/session/new.html`下添加一个视图模板，其中包含登录表单：

```js
<h1>Log in</h1>
<form action="/session" method="POST">
  <p>
    <label for="email">E-mail</label>
    <input type="email" name="email" value="" id="email"/>
  </p>
  <p>
    <label for="password">Password</label>
    <input type="password" name="password" id="password" value="" required/>
  </p>
  <input type="submit" value="Log In" />
</form>
```

接下来，如果服务器仍在运行，请停止服务器（按下*Ctrl* + *C*），然后重新启动它：

```js
$ node app.js
```

将浏览器指向`http://localhost:3000/session/new`，并插入您已经注册的用户的电子邮件和密码：

![登录和会话管理](img/0526OS_02_07.jpg)

如果登录成功，您将被重定向到`/todos` URL，服务器尚未响应。

接下来，我们将使待办事项列表起作用。

## 待办事项列表

为了显示待办事项列表，我们将使用表格。通过使用 jQuery UI，可以很容易地对待办事项进行排序。启用此功能的简单方法是使用 jQuery UI。仅需此功能，您无需完整的 jQuery UI 库，可以通过将浏览器指向`http://jqueryui.com/download`，取消**交互**元素中除**Sortable**选项之外的所有选项，并单击**Download**按钮来下载自定义构建的 jQuery UI 库。解压缩生成的文件，并将`jquery-ui-1.8.23.custom.min.js`文件复制到`public/js`中。

![待办事项列表](img/0526OS_02_08.jpg)

我们需要在`templates.html`或`layout.html`文件中引用此脚本：

```js
<html>
  <head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
    <title id="title"></title>
    <link href="/css/bootstrap.min.css" rel="stylesheet" />
  </head>
  <body>

    <section role="main" class="container">

      <div id="messages"></div>

      <div id="main-body"></div>

    </section>

    <script src="img/jquery.min.js"></script> 
 <script src="img/jquery-ui-1.8.23.custom.min.js"></script> 
    <script src="img/bootstrap.min.js"></script>
 <script src="img/todos.js"></script>
  </body>
</html>
```

您还应该在`public/js/todos.js`下添加一个文件，其中包含一些前端交互代码。

现在，我们需要通过首先在`app.js`文件中包含新的路由来响应`/todos` URL：

```js
var flatiron = require('flatiron'),
    path = require('path'),
    nstatic = require('node-static'),
    app = flatiron.app;

app.config.file({ file: path.join(__dirname, 'config', 'config.json') });

var file = new nstatic.Server(__dirname + '/public/');

app.use(flatiron.plugins.http, {
  before: [
    require('flatware-cookie-parser')(),
    require('flatware-session')(),
    function(req, res) {
      var found = app.router.dispatch(req, res);
      if (! found) {
        file.serve(req, res);
      }
    }
  ]
});

app.router.path('/users', require('./routes/users'));
app.router.path('/session', require('./routes/session'));
app.router.path('/todos', require('./routes/todos'));

app.start(3000);
```

然后，我们需要将新的待办事项路由模块放在`routes/todos.js`下：

```js
var fs      = require('fs'),
    couchdb = require('../lib/couchdb'),
    dbName  = 'todos',
    db      = couchdb.use(dbName),
    Plates  = require('plates'),
    layout  = require('../templates/layout'),
    loggedIn = require('../middleware/logged_in')();

var templates = {
  index : fs.readFileSync(__dirname +
    '/../templates/todos/index.html', 'utf8'),
  'new' : fs.readFileSync(__dirname +
    '/../templates/todos/new.html', 'utf8')
};

function insert(email, todo, callback) {
  var tries = 0,
      lastError;

  (function doInsert() {
    tries ++;
    if (tries >= 3) return callback(lastError);

    db.get(email, function(err, todos) {
      if (err && err.status_code !== 404) return callback(err);

      if (! todos) todos = {todos: []};
      todos.todos.unshift(todo);

      db.insert(todos, email, function(err) {
        if (err) {
          if (err.status_code === 404) {
            lastError = err;
            // database does not exist, need to create it
            couchdb.db.create(dbName, function(err) {
              if (err) {
                return callback(err);
              }
              doInsert();
            });
            return;
          }
          return callback(err);
        }
        return callback();
      });
    });
  })();

}

module.exports = function() {

  this.get('/', [loggedIn, function() {

    var res = this.res;

    db.get(this.req.session.user.email, function(err, todos) {

      if (err && err.status_code !== 404) {
        res.writeHead(500);
        return res.end(err.stack);
      }

      if (! todos) todos = {todos: []};
      todos = todos.todos;

      todos.forEach(function(todo, idx) {
        if (todo) todo.pos = idx + 1;
      });

      var map = Plates.Map();
      map.className('todo').to('todo');
      map.className('pos').to('pos');
      map.className('what').to('what');
      map.where('name').is('pos').use('pos').as('value');

      var main = Plates.bind(templates.index, {todo: todos}, map);
      res.writeHead(200, {'Content-Type': 'text/html'});
      res.end(layout(main, 'To-Dos'));

    });

  }]);

  this.get('/new', [loggedIn, function() {

    this.res.writeHead(200, {'Content-Type': 'text/html'});
    this.res.end(layout(templates['new'], 'New To-Do'));
  }]);

  this.post('/', [loggedIn, function() {

    var req  = this.req,
        res  = this.res,
        todo = this.req.body
    ;

    if (! todo.what) {
      res.writeHead(200, {'Content-Type': 'text/html'});
      return res.end(layout(templates['new'], 'New To-Do',
        {error: 'Please fill in the To-Do description'}));
    }

    todo.created_at = Date.now();

    insert(req.session.user.email, todo, function(err) {

      if (err) {
        res.writeHead(500);
        return res.end(err.stack);
      }

      res.writeHead(303, {Location: '/todos'});
      res.end();
    });

  }]);

  this.post('/sort', [loggedIn, function() {

    var res = this.res,
        order = this.req.body.order && this.req.body.order.split(','),
        newOrder = []
        ;

    db.get(this.req.session.user.email, function(err, todosDoc) {
      if (err) {
        res.writeHead(500);
        return res.end(err.stack);
      }

      var todos = todosDoc.todos;

      if (order.length !== todos.length) {
        res.writeHead(409);
        return res.end('Conflict');
      }

      order.forEach(function(order) {
        newOrder.push(todos[parseInt(order, 10) - 1]);
      });

      todosDoc.todos = newOrder;

      db.insert(todosDoc, function(err) {
        if (err) {
          res.writeHead(500);
          return res.end(err.stack);
        }
        res.writeHead(200);
        res.end();
      });

    });
  }]);

  this.post('/delete', [loggedIn, function() {

    var req = this.req,
        res = this.res,
        pos = parseInt(req.body.pos, 10)
        ;

    db.get(this.req.session.user.email, function(err, todosDoc) {
      if (err) {
        res.writeHead(500);
        return res.end(err.stack);
      }

      var todos = todosDoc.todos;
      todosDoc.todos = todos.slice(0, pos - 1).concat(todos.slice(pos));

      db.insert(todosDoc, function(err) {
        if (err) {
          res.writeHead(500);
          return res.end(err.stack);
        }
        res.writeHead(303, {Location: '/todos'});
        res.end();
      });

    });

  }]);

};
```

该模块响应待办事项索引（`GET /todos`），获取并呈现已登录用户的所有待办事项。将以下模板放在`templates/todos/index.html`下：

```js
<h1>Your To-Dos</h1>

<a class="btn" href="/todos/new">New To-Do</a>

<table class="table">
  <thead>
    <tr>
      <th>#</th>
      <th>What</th>
      <th></th>
    </tr>
  </thead>
  <tbody id="todo-list">
    <tr class="todo">
      <td class="pos"></td>
      <td class="what"></td>
      <td class="remove">
        <form action="/todos/delete" method="POST">
          <input type="hidden" name="pos" value="" />
          <input type="submit" name="Delete" value="Delete" />
        </form>
      </td>
    </tr>
  </tbody>
</table>
```

另一个新路由是`GET /todos/new`，向用户呈现创建新待办事项的表单。此路由使用放置在`templates/todos/new.html`中的新模板：

```js
<h1>New To-Do</h1>
<form action="/todos" method="POST">
  <p>
    <label for="email">What</label>
    <textarea name="what" id="what" required></textarea>
  </p>
  <input type="submit" value="Create" />
</form>
```

`POST /todos`路由通过调用本地的`insert`函数创建新的待办事项，该函数处理了数据库不存在时的错误，并在需要时创建数据库并稍后重试`insert`函数。

索引模板取决于`public/js/todos.js`下放置的客户端脚本的存在：

```js
$(function() {
  $('#todo-list').sortable({
    update: function() {
      var order = [];
      $('.todo').each(function(idx, row) {
        order.push($(row).find('.pos').text());
      });

      $.post('/todos/sort', {order: order.join(',')}, function() {
        $('.todo').each(function(idx, row) {
          $(row).find('.pos').text(idx + 1);
        });
      });

    } 
  });
});
```

此文件激活并处理拖放项目，通过向`/todos/sort` URL 发出 AJAX 调用，传递待办事项的新顺序。

`todos.js`路由模块还处理了每个项目上的**删除**按钮，它通过加载用户的待办事项，删除给定位置的项目并将项目存储回去来处理。

### 注意

到目前为止，您可能已经注意到我们将给定用户的所有待办事项存储在`todos`数据库中的一个文档中。如果所有用户保持待办事项的数量相对较低，这种技术是简单且有效的。无论如何，这些细节对我们的目的并不重要。

为使其工作，我们需要在`middleware/logged_in.js`下提供一个路由中间件。这个中间件组件负责保护一些路由，并在用户未登录时将用户重定向到登录屏幕，而不是执行该路由：

```js
function LoggedIn() {
  return function(next) {
    if (! this.req.session || ! this.req.session.user) {
      this.res.writeHead(303, {Location: '/session/new'});
      return this.res.end();
    }
    next();
  };
}

module.exports = LoggedIn;
```

最后，如果服务器仍在运行，请停止它（按下*Ctrl* + *C*），然后再次启动它：

```js
$ node app.js
```

将浏览器指向`http://localhost:3000/session/new`，并输入您已经注册的用户的电子邮件和密码。然后，您将被重定向到用户的待办事项列表，该列表将开始为空。

![待办事项列表](img/0526OS_02_09.jpg)

现在您可以单击**新建待办事项**按钮，获取以下表单：

![待办事项列表](img/0526OS_02_10.jpg)

插入一些文本，然后单击**创建**按钮。待办事项将被插入到数据库中，并且更新后的待办事项列表将被呈现：

![待办事项列表](img/0526OS_02_11.jpg)

您可以插入任意数量的待办事项。一旦您满意了，您可以尝试通过拖放表格行来重新排序它们。

![待办事项列表](img/0526OS_02_12.jpg)

您还可以单击**删除**按钮来删除特定的待办事项。

# 文件摘要

以下是组成此应用程序的文件列表：

```js
$ tree
.
├── app.js
├── config
│   └── config.json
├── lib
│   └── couchdb.js
├── middleware
│   └── logged_in.js
├── package.json
├── public
│   ├── css
│   │   ├── bootstrap-responsive.css
│   │   ├── bootstrap-responsive.min.css
│   │   ├── bootstrap.css
│   │   └── bootstrap.min.css
│   ├── img
│   │   ├── glyphicons-halflings-white.png
│   │   └── glyphicons-halflings.png
│   └── js
│       ├── bootstrap.js
│       ├── bootstrap.min.js
│       ├── jquery-ui-1.8.23.custom.min.js
│       ├── jquery.min.js
│       └── todos.js
├── routes
│   ├── session.js
│   ├── todos.js
│   └── users.js
├── templates
│   ├── alert.html
│   ├── layout.html
│   ├── layout.js
│   ├── session
│   │   └── new.html
│   ├── todos
│   │   ├── index.html
│   │   └── new.html
│   └── users
│       ├── new.html
│       └── show.html
└── test

13 directories, 27 files
```

# 摘要

在本章中，您学会了如何使用 Node.js、Flatiron.js 和其他一些组件创建一个简单的 Web 应用程序。

这个应用程序将成为我们将来章节中用户界面测试的目标。
