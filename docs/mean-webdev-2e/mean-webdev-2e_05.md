# 第五章：Mongoose 简介

Mongoose 是一个强大的 Node.js ODM 模块，为您的 Express 应用程序添加了 MongoDB 支持。它使用模式来对实体进行建模，提供预定义验证以及自定义验证，允许您定义虚拟属性，并使用中间件钩子来拦截操作。Mongoose 的设计目标是弥合 MongoDB 无模式方法与现实世界应用程序开发要求之间的差距。在本章中，您将了解 Mongoose 的以下基本特性：

+   Mongoose 模式和模型

+   模式索引、修饰符和虚拟属性

+   使用模型的方法和执行 CRUD 操作

+   使用预定义和自定义验证器验证您的数据

+   使用中间件拦截模型的方法

# 介绍 Mongoose

Mongoose 是一个 Node.js 模块，为开发人员提供了将对象建模并将其保存为 MongoDB 文档的能力。虽然 MongoDB 是一个无模式的数据库，但在处理 Mongoose 模型时，Mongoose 为您提供了享受严格和宽松模式方法的机会。与任何其他 Node.js 模块一样，在您的应用程序中开始使用它之前，您首先需要安装它。本章中的示例将直接从前几章中的示例继续进行；因此，在本章中，从第三章中复制最终示例，*构建一个 Express Web 应用程序*，然后从那里开始。

## 安装 Mongoose

安装并验证您的 MongoDB 本地实例正在运行后，您将能够使用 Mongoose 模块连接到它。首先，您需要在`node_modules`文件夹中安装 Mongoose，因此将您的`package.json`文件更改为以下代码片段所示的样子：

```js
{
  "name": "MEAN",
  "version": "0.0.5",
  "dependencies": {
    "body-parser": "1.15.2",
    "compression": "1.6.0",
    "ejs": "2.5.2",
    "express": "4.14.0",
    "express-session": "1.14.1",
    "method-override": "2.3.6",
 "mongoose": "4.6.5",
    "morgan": "1.7.0"
}
```

要安装应用程序依赖项，请转到应用程序文件夹，并在命令行工具中发出以下命令：

```js
$ npm install

```

这将在您的`node_modules`文件夹中安装最新版本的 Mongoose。安装过程成功完成后，下一步将是连接到您的 MongoDB 实例。

## 连接到 MongoDB

要连接到 MongoDB，您需要使用 MongoDB 连接 URI。MongoDB 连接 URI 是一个字符串 URL，告诉 MongoDB 驱动程序如何连接到数据库实例。MongoDB URI 通常构造如下：

```js
mongodb://username:password@hostname:port/database

```

由于您正在连接到本地实例，可以跳过用户名和密码，使用以下 URI：

```js
mongodb://localhost/mean-book

```

最简单的方法是直接在您的`config/express.js`配置文件中定义此连接 URI，并使用`mongoose`模块连接到数据库，如下所示：

```js
const uri = 'mongodb://localhost/mean-book';
const db = require('mongoose').connect(uri);
```

但是，由于您正在构建一个真实的应用程序，直接在`config/express.js`文件中保存 URI 是一种不好的做法。存储应用程序变量的正确方法是使用您的环境配置文件。转到您的`config/env/development.js`文件，并将其更改为以下代码片段所示的样子：

```js
module.exports = {
 db: 'mongodb://localhost/mean-book',
  sessionSecret: 'developmentSessionSecret'
};
```

现在在您的`config`文件夹中，创建一个名为`mongoose.js`的新文件，其中包含以下代码片段：

```js
const config = require('./config');
const mongoose = require('mongoose');

module.exports = function() {
 const db = mongoose.connect(config.db);

  return db;
};
```

请注意，您需要`mongoose`模块并使用配置对象的`db`属性连接到 MongoDB 实例。要初始化 Mongoose 配置，请返回到您的`server.js`文件，并将其更改为以下代码片段所示的样子：

```js
process.env.NODE_ENV = process.env.NODE_ENV || 'development';

const configureMongoose = require('./config/mongoose');
const configureExpress = require('./config/express');

const db = configureMongoose();
const app = configureExpress();
app.listen(3000);

module.exports = app;
console.log('Server running at http://localhost:3000/');
```

就是这样；您已经安装了 Mongoose，更新了配置文件，并连接到了 MongoDB 实例。要启动应用程序，请使用命令行工具并导航到应用程序文件夹，执行以下命令：

```js
$ node server

```

您的应用程序应该正在运行并连接到 MongoDB 本地实例。

### 注意

如果您遇到任何问题或出现“错误：无法连接到[localhost:27017]”的输出，请确保您的 MongoDB 实例正常运行。

# 了解 Mongoose 模式

连接到您的 MongoDB 实例是第一步，但 Mongoose 模块的真正魔力在于其定义文档模式的能力。正如您已经知道的，MongoDB 使用集合来存储多个文档，这些文档不需要具有相同的结构。但是，在处理对象时，有时需要文档相似。Mongoose 使用模式对象来定义文档属性列表，每个属性都有自己的类型和约束，以强制执行文档结构。在指定模式之后，您将继续定义一个模型构造函数，用于创建 MongoDB 文档的实例。在本节中，您将学习如何定义用户模式和模型，以及如何使用模型实例来创建、检索和更新用户文档。

## 创建用户模式和模型

要创建您的第一个模式，请转到`app/models`文件夹并创建一个名为`user.server.model.js`的新文件。在此文件中，粘贴以下代码行：

```js
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

const UserSchema = new Schema({
  firstName: String,
  lastName: String,
  email: String,
  username: String,
  password: String
});

mongoose.model('User', UserSchema);
```

在上述代码片段中，您做了两件事：首先，使用`Schema`构造函数定义了您的`UserSchema`对象，然后使用模式实例定义了您的 User 模型。请注意，出于简单起见，我们将密码保存为明文；但是，在实际应用程序中，用户密码应该得到适当的加密。接下来，您将学习如何使用 User 模型在应用程序逻辑层执行 CRUD 操作。

## 注册 User 模型

在您可以开始使用 User 模型之前，您需要在 Mongoose 配置文件中包含`user.server.model.js`文件，以注册 User 模型。为此，请更改您的`config/mongoose.js`文件，使其看起来像以下代码片段中所示：

```js
const config = require('./config');
const mongoose = require('mongoose');

module.exports = function() {
  const db = mongoose.connect(config.db);

 require('../app/models/user.server.model');

  return db;
};
```

确保在`server.js`文件中执行任何其他配置之前加载 Mongoose 配置文件。这很重要，因为在此模块之后加载的任何模块都将能够使用 User 模型，而无需自行加载它。

## 使用 save()创建新用户

您可以立即开始使用 User 模型，但为了保持组织有序，最好创建一个`Users`控制器，用于处理所有与用户相关的操作。在`app/controllers`文件夹中，创建一个名为`users.server.controller.js`的新文件，并粘贴以下代码行：

```js
const User = require('mongoose').model('User');

exports.create = function(req, res, next) {
  const user = new User(req.body);

  user.save((err) => {
    if (err) {
      return next(err);
    } else {
      res.status(200).json(user);
    }
  });
};
```

让我们来看看这段代码。首先，您使用`mongoose`模块调用`model`方法，该方法将返回您之前定义的`User`模型。接下来，您创建了一个名为`create()`的控制器方法，稍后将用于创建新用户。使用`new`关键字，`create()`方法创建一个新的模型实例，该实例使用请求体进行填充。最后，您调用模型实例的`save()`方法，该方法要么保存用户并输出`user`对象，要么失败并将错误传递给下一个中间件。

要测试您的新控制器，让我们添加一组调用控制器方法的与用户相关的路由。首先，在`app/routes`文件夹中创建一个名为`users.server.routes.js`的文件。在这个新创建的文件中，粘贴以下代码行：

```js
const users = require('../../app/controllers/users.server.controller');

module.exports = function(app) {
  app.route('/users').post(users.create);
};
```

由于您的 Express 应用程序主要将作为 AngularJS 应用程序的 RESTful API，因此最佳实践是根据 REST 原则构建路由。在这种情况下，创建新用户的正确方式是使用 HTTP POST 请求到您在此定义的基本`users`路由。更改您的`config/express.js`文件，使其看起来像以下代码片段中所示：

```js
const config = require('./config');
const express = require('express');
const morgan = require('morgan');
const compress = require('compression');
const bodyParser = require('body-parser');
const methodOverride = require('method-override');
const session = require('express-session');

module.exports = function() {
  const app = express();

  if (process.env.NODE_ENV === 'development') {
    app.use(morgan('dev'));
  } else if (process.env.NODE_ENV === 'production') {
    app.use(compress());
  }

  app.use(bodyParser.urlencoded({
    extended: true
  }));
  app.use(bodyParser.json());
  app.use(methodOverride());

  app.use(session({
    saveUninitialized: true,
    resave: true,
    secret: config.sessionSecret
  }));

  app.set('views', './app/views');
  app.set('view engine', 'ejs');

  require('../app/routes/index.server.routes.js')(app);
  require('../app/routes/users.server.routes.js')(app);

  app.use(express.static('./public'));

  return app;
};
```

就是这样！要进行测试，请转到根应用程序文件夹并执行以下命令：

```js
$ node server

```

您的应用程序应该正在运行。要创建新用户，请执行 HTTP POST 请求到基本的`users`路由，并确保请求体包含以下 JSON：

```js
{
  "firstName": "First",
  "lastName": "Last",
  "email": "user@example.com",
  "username": "username",
  "password": "password"
}
```

另一种测试应用程序的方法是在命令行工具中执行以下`curl`命令：

```js
$ curl -X POST -H "Content-Type: application/json" -d '{"firstName":"First", "lastName":"Last","email":"user@example.com","username":"username","password":"password"}' localhost:3000/users

```

### 提示

您将执行许多不同的 HTTP 请求来测试您的应用程序。对于 Mac OS X 和 Linux 用户，`curl`是一个有用的工具，但还有其他几种专门设计用于此任务的工具；我们建议您找到自己喜欢的工具并从现在开始使用它。

## 使用`find()`查找多个用户文档

`find()`方法是一个模型方法，它使用查询检索存储在同一集合中的多个文档，并且是 MongoDB `find()`集合方法的 Mongoose 实现。为了更好地理解这一点，请将以下`list()`方法添加到您的`app/controllers/users.server.controller.js`文件中：

```js
exports.list = function(req, res, next) {
  User.find({}, (err, users) => {
    if (err) {
      return next(err);
    } else {
      res.status(200).json(users);
    }
  });
};
```

注意新的`list()`方法如何使用`find()`方法来检索`users`集合中所有文档的数组。要使用您创建的新方法，您需要为其注册一个路由，因此转到您的`app/routes/users.server.routes.js`文件并更改为以下代码片段所示：

```js
const users = require('../../app/controllers/users.server.controller');

module.exports = function(app) {
  app.route('/users')
    .post(users.create)
    .get(users.list);
};
```

您需要做的就是通过执行以下命令运行应用程序：

```js
$ node server

```

然后，您将能够通过在浏览器中访问`http://localhost:3000/users`来检索用户列表。

### 使用`find()`进行高级查询

在上面的代码示例中，`find()`方法接受了两个参数，一个是 MongoDB 查询对象，另一个是回调函数，但它最多可以接受四个参数：

+   `Query`：这是一个 MongoDB 查询对象

+   `[Fields]`：这是一个可选的字符串对象，表示要返回的文档字段

+   `[Options]`：这是一个可选的`options`对象

+   `[Callback]`：这是一个可选的回调函数

例如，为了仅检索用户的用户名和电子邮件，您需要修改调用，使其类似于以下代码行所示：

```js
User.find({}, 'username email', (err, users) => {
  …
});
```

此外，当调用`find()`方法时，还可以传递一个`options`对象，该对象将操作查询结果。例如，要通过`skip`和`limit`选项分页浏览`users`集合并仅检索`users`集合的子集，可以使用以下方法：

```js
User.find({}, 'username email', {
  skip: 10,
  limit: 10
}, (err, users) => {
  ...
});
```

这将返回最多 10 个用户文档的子集，同时跳过前 10 个文档。

### 注意

要了解更多有关查询选项的信息，建议您访问官方的 Mongoose 文档[`mongoosejs.com/docs/api.html`](http://mongoosejs.com/docs/api.html)。

## 使用`findOne()`读取单个用户文档

使用`findOne()`方法检索单个用户文档，这与`find()`方法非常相似，但它仅检索子集的第一个文档。要开始处理单个用户文档，我们需要添加两个新方法。将以下代码行添加到您的`app/controllers/users.server.controller.js`文件的末尾：

```js
exports.read = function(req, res) {
  res.json(req.user);
};

exports.userByID = function(req, res, next, id) {
  User.findOne({
    _id: id
  }, (err, user) => {
    if (err) {
      return next(err);
    } else {
      req.user = user;
      next();
    }
  });
};
```

`read()`方法很容易理解；它只是用`req.user`对象的 JSON 表示进行响应，但是是谁创建了`req.user`对象呢？嗯，`userById()`方法负责填充`req.user`对象。在执行读取、删除和更新操作时，您将使用`userById()`方法作为中间件来处理单个文档的操作。为此，您需要修改`app/routes/users.server.routes.js`文件，使其类似于以下代码行所示：

```js
const users = require('../../app/controllers/users.server.controller');

module.exports = function(app) {
  app.route('/users')
     .post(users.create)
     .get(users.list);

 app.route('/users/:userId')
 .get(users.read);

 app.param('userId', users.userByID);
};
```

请注意，您添加了包含`userId`的请求路径的`users.read()`方法。在 Express 中，在路由定义中的子字符串前添加冒号意味着该子字符串将被处理为请求参数。为了处理`req.user`对象的填充，您使用`app.param()`方法，该方法定义了在使用该参数的任何其他中间件之前执行的中间件。在这里，`users.userById()`方法将在此情况下`users.read()`中注册的任何其他使用`userId`参数的中间件之前执行。在构建 RESTful API 时，此设计模式非常有用，其中您经常向路由字符串添加请求参数。

要测试这个，使用以下命令运行您的应用程序：

```js
$ node server

```

然后，在浏览器中导航到`http://localhost:3000/users`，获取其中一个用户的`_id`值，并导航到`http://localhost:3000/users/[id]`，将`[id]`部分替换为用户的`_id`值。

## 更新现有用户文档

Mongoose 模型有几种可用的方法来更新现有文档。其中包括`update()`、`findOneAndUpdate()`和`findByIdAndUpdate()`方法。每种方法在可能时都提供了不同级别的抽象，简化了`update`操作。在我们的情况下，由于我们已经使用了`userById()`中间件，更新现有文档的最简单方法是使用`findByIdAndUpdate()`方法。要做到这一点，返回到您的`app/controllers/users.server.controller.js`文件并添加一个新的`update()`方法：

```js
exports.update = function(req, res, next) {
  User.findByIdAndUpdate(req.user.id, req.body, {
    'new': true
  }, (err, user) => {
    if (err) {
      return next(err);
    } else {
      res.status(200).json(user);
    }
  });
};
```

注意您如何使用用户的`id`字段来查找和更新正确的文档。请注意，默认的 Mongoose 行为是在更新文档之前将回调传递给文档；通过将`new`选项设置为`true`，我们确保我们收到更新后的文档。接下来您应该做的是在用户的路由模块中连接您的新的`update()`方法。返回到您的`app/routes/users.server.routes.js`文件并将其更改为以下代码片段所示的样子：

```js
const users = require('../../app/controllers/users.server.controller');

module.exports = function(app) {
  app.route('/users')
     .post(users.create)
     .get(users.list);

  app.route('/users/:userId')
     .get(users.read)
     .put(users.update);

  app.param('userId', users.userByID);
};
```

注意您如何使用之前创建的路由，并如何使用路由的`put()`方法链接`update()`方法。要测试您的`update()`方法，请使用以下命令运行您的应用程序：

```js
$ node server

```

然后，使用您喜欢的 REST 工具发出 PUT 请求，或者使用`curl`并执行此命令，将`[id]`部分替换为实际文档的`_id`属性：

```js
$ curl -X PUT -H "Content-Type: application/json" -d '{"lastName": "Updated"}' localhost:3000/users/[id]

```

## 删除现有用户文档

Mongoose 模型有几种可用的方法来删除现有文档。其中包括`remove()`、`findOneAndRemove()`和`findByIdAndRemove()`方法。在我们的情况下，由于我们已经使用了`userById()`中间件，删除现有文档的最简单方法就是简单地使用`remove()`方法。要做到这一点，返回到您的`app/controllers/users.server.controller.js`文件并添加以下`delete()`方法：

```js
exports.delete = function(req, res, next) {
  req.user.remove(err => {
    if (err) {
      return next(err);
    } else {
      res.status(200).json(req.user);
    }
  })
};
```

注意您如何使用`user`对象来删除正确的文档。接下来您应该做的是在用户的路由文件中使用您的新的`delete()`方法。转到您的`app/routes/users.server.routes.js`文件并将其更改为以下代码片段所示的样子：

```js
const users = require('../../app/controllers/users.server.controller');

module.exports = function(app) { 
  app.route('/users')
    .post(users.create)
    .get(users.list);

  app.route('/users/:userId')
    .get(users.read)
    .put(users.update)
    .delete(users.delete);

  app.param('userId', users.userByID);
};
```

注意您如何使用之前创建的路由，并如何使用路由的`delete()`方法链接`delete()`方法。要测试您的`delete`方法，请使用以下命令运行您的应用程序：

```js
$ node server

```

然后，使用您喜欢的 REST 工具发出`DELETE`请求，或者使用`curl`并执行以下命令，将`[id]`部分替换为实际文档的`_id`属性：

```js
$ curl -X DELETE localhost:3000/users/[id]

```

这完成了四个 CRUD 操作的实现，让您简要了解了 Mongoose 模型的能力。然而，这些方法只是 Mongoose 包含的众多功能的示例。在下一节中，您将学习如何定义默认值，为模式字段提供动态功能，并验证您的数据。

# 扩展您的 Mongoose 模式

进行数据操作是很好的，但为了开发复杂的应用程序，您需要让您的 ODM 模块做更多的事情。幸运的是，Mongoose 支持各种其他功能，帮助您安全地对文档进行建模并保持数据的一致性。

## 定义默认值

定义默认字段值是数据建模框架的常见功能。您可以直接将此功能添加到应用程序的逻辑层，但这样会很混乱，通常是一种不好的做法。Mongoose 提供在模式级别定义默认值的功能，帮助您更好地组织代码并保证文档的有效性。

假设你想要向你的`UserSchema`添加一个创建日期字段。创建日期字段应该在创建时初始化，并且应该保存用户文档最初创建的时间，这是一个完美的例子，你可以利用默认值。为了做到这一点，你需要更改你的`UserSchema`；所以，回到你的`app/models/user.server.model.js`文件，并将其更改为以下代码片段所示的样子：

```js
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

const UserSchema = new Schema({
  firstName: String,
  lastName: String,
  email: String,
  username: String,
  password: String,
  created: {
    type: Date,
    default: Date.now
  }
});

mongoose.model('User', UserSchema);
```

注意`created`字段的添加和其默认值的定义。从现在开始，每个新的用户文档都将被创建一个默认的创建日期，代表文档创建的时刻。你还应该注意，在此模式更改之前创建的每个用户文档都将被分配一个创建字段，代表你查询它的时刻，因为这些文档没有初始化创建字段。

要测试你的新更改，使用以下命令运行你的应用程序：

```js
$ node server

```

然后，使用你喜欢的 REST 工具发出一个 POST 请求，或者使用`curl`并执行以下命令：

```js
$ curl -X POST -H "Content-Type: application/json" -d '{"firstName":"First", "lastName":"Last","email":"user@example.com","username":"username","password":"password"}' localhost:3000/users

```

将会创建一个新的用户文档，其中包含一个默认的创建字段，在创建时初始化。

## 使用模式修饰符

有时，你可能希望在保存或呈现给客户端之前对模式字段进行操作。为此，Mongoose 使用了一个称为*修饰符*的功能。修饰符可以在保存文档之前更改字段的值，也可以在查询时以不同的方式表示它。

### 预定义的修饰符

最简单的修饰符是 Mongoose 附带的预定义修饰符。例如，字符串类型的字段可以有一个修剪修饰符来去除空格，一个大写修饰符来将字段值大写，等等。为了理解预定义修饰符的工作原理，让我们确保你的用户的用户名不包含前导和尾随空格。要做到这一点，你只需要更改你的`app/models/user.server.model.js`文件，使其看起来像以下代码片段所示的样子：

```js
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

const UserSchema = new Schema({
  firstName: String,
  lastName: String,
  email: String,
  username: {
    type: String,
    trim: true
  },
  password: String,
  created: {
    type: Date,
    default: Date.now
  }
});

mongoose.model('User', UserSchema);
```

注意`username`字段中添加的`trim`属性。这将确保你的用户名数据将被保持修剪。

### 自定义 setter 修饰符

预定义的修饰符很棒，但你也可以定义自己的自定义 setter 修饰符来处理保存文档之前的数据操作。为了更好地理解这一点，让我们向你的用户模型添加一个新的`website`字段。`website`字段应该以`http://`或`https://`开头，但是不要强迫你的客户在 UI 中添加这些前缀，你可以简单地编写一个自定义修饰符来验证这些前缀的存在，并在需要时添加它们。要添加你的自定义修饰符，你需要创建一个带有`set`属性的新的`website`字段，如下所示：

```js
const UserSchema = new Schema({
  …
  website: {
    type: String,
    set: function(url) {
      if (!url) {
        return url;
      } else {
        if (url.indexOf('http://') !== 0   &&           url.indexOf('https://') !== 0) {
          url = 'http://' + url;
        }

        return url;
        }
    }
  },
  …
});
```

现在，每个创建的用户都将拥有一个在创建时修改的正确形式的网站 URL。然而，如果你已经有了一个大量的用户文档集合，你当然可以迁移你现有的数据，但是当处理大型数据集时，这将会对性能产生严重影响，所以你可以简单地使用 getter 修饰符。

### 自定义 getter 修饰符

**Getter**修饰符用于在将文档输出到下一层之前修改现有数据。例如，在我们之前的示例中，getter 修饰符有时会更好地通过在查询时修改网站字段来更改已经存在的用户文档，而不是遍历你的 MongoDB 集合并更新每个文档。要做到这一点，你只需要更改你的`UserSchema`，如下面的代码片段所示：

```js
const UserSchema = new Schema({
  ...
  website: {
    type: String,
    get: function(url) {
      if (!url) {
        return url;
      } else {
        if (url.indexOf('http://') !== 0 &&           url.indexOf('https://') !== 0) {
            url = 'http://' + url;
          }

        return url;
     }
    }
  },
  …
});

UserSchema.set('toJSON', { getters: true });

```

你只需通过将`set`属性更改为`get`来将 setter 修改器更改为 getter 修改器。然而，这里需要注意的重要事情是如何使用`UserSchema.set()`配置了你的模式。这将强制 Mongoose 在将 MongoDB 文档转换为 JSON 表示时包含 getter，并允许使用`res.json()`输出文档以包含 getter 的行为。如果你没有包含这个，你的文档的 JSON 表示将忽略 getter 修改器。

### 注意

修改器非常强大，可以节省大量时间，但应谨慎使用，以防止出现意外的应用程序行为。建议您访问[`mongoosejs.com/docs/api.html`](http://mongoosejs.com/docs/api.html)获取更多信息。

## 添加虚拟属性

有时，你可能希望有动态计算的文档属性，这些属性实际上并不在文档中呈现。这些属性称为*虚拟属性*，它们可以用来满足几个常见的需求。例如，假设你想要添加一个新的`fullName`字段，它将表示用户的名和姓的连接。为此，你将需要使用`virtual()`模式方法；因此，修改后的`UserSchema`将包括以下代码片段：

```js
UserSchema.virtual('fullName').get(function(){
  return this.firstName + ' ' + this.lastName;
});

UserSchema.set('toJSON', { getters: true, virtuals: true });
```

在上面的代码示例中，你向`UserSchema`添加了一个名为`fullName`的虚拟属性，为该虚拟属性添加了一个`getter`方法，然后配置了你的模式以在将 MongoDB 文档转换为 JSON 表示时包含虚拟属性。

然而，虚拟属性也可以有 setter，以帮助你保存你的文档，而不仅仅是添加更多字段属性。在这种情况下，假设你想要将输入的`fullName`字段分解为名和姓字段。为此，修改后的虚拟声明将如下代码片段所示：

```js
UserSchema.virtual('fullName').get(function() {
  return this.firstName + ' ' + this.lastName;
}).set(function(fullName) {
 const splitName = fullName.split(' '); 
 this.firstName = splitName[0] || ''; 
 this.lastName = splitName[1] || ''; 
});

```

虚拟属性是 Mongoose 的一个很棒的特性，允许你在文档表示被传递到应用程序的各个层时修改它们，而不会被持久化到 MongoDB 中。

## 使用索引优化查询

正如我们之前讨论的，MongoDB 支持各种类型的索引来优化查询执行。Mongoose 也支持索引功能，甚至允许你定义次要索引。

索引的基本示例是唯一索引，它验证了集合中`document`字段的唯一性。在我们的示例中，保持用户名唯一是很常见的，因此为了传达这一点给 MongoDB，你需要修改你的`UserSchema`定义，包括以下代码片段：

```js
const UserSchema = new Schema({
  ...
  username: {
    type: String,
    trim: true,
    unique: true
  },
  ...
});
```

这将告诉 MongoDB 为`users`集合的`username`字段创建一个唯一索引。Mongoose 还支持使用`index`属性创建次要索引。因此，如果你知道你的应用程序将使用大量涉及`email`字段的查询，你可以通过以下方式优化这些查询，创建一个电子邮件次要索引：

```js
const UserSchema = new Schema({
  …
  email: {
    type: String,
    index: true
  },
  …
});
```

索引是 MongoDB 的一个很棒的特性，但你应该记住它可能会给你带来一些麻烦。例如，如果你在已经存储数据的集合上定义了唯一索引，你可能会在运行应用程序时遇到一些错误，直到你解决了集合数据的问题。另一个常见问题是 Mongoose 在应用程序启动时自动创建索引，这个特性可能会在生产环境中导致严重的性能问题。

# 定义自定义模型方法

Mongoose 模型中既包含静态方法又包含实例预定义方法，其中一些你已经使用过。然而，Mongoose 还允许你定义自己的自定义方法来增强你的模型，为你提供一个模块化的工具来正确分离你的应用程序逻辑。让我们来看一下定义这些方法的正确方式。

## 定义自定义静态方法

模型静态方法使您有自由进行模型级操作，例如添加额外的`find`方法。例如，假设您想通过他们的用户名搜索用户。当然，您可以在控制器中定义`this`方法，但那不是正确的地方。您要找的是静态模型方法。要添加静态方法，您需要将其声明为模式的`statics`属性的成员。在我们的情况下，添加一个`findOneByUsername()`方法将看起来像下面的代码片段所示：

```js
UserSchema.statics.findOneByUsername = function(username, callback) {
    this.findOne({ username: new RegExp(username, 'i') }, 
  callback);
};
```

这种方法使用模型的`findOne()`方法来检索具有特定用户名的用户文档。使用新的`findOneByUsername()`方法类似于直接从`User`模型调用标准的`static`方法，如下所示：

```js
User.findOneByUsername('username', (err, user) => {
  …
});
```

当开发应用程序时，您当然可以想出许多其他静态方法；您可能在开发应用程序时需要它们，所以不要害怕添加它们。

## 定义自定义实例方法

静态方法很棒，但如果您需要执行实例操作的方法怎么办？好吧，Mongoose 也支持这些方法，帮助您精简代码库并正确重用应用程序代码。要添加实例方法，您需要将其声明为模式的`methods`属性的成员。假设您想使用`authenticate()`方法验证用户的密码。添加此方法将类似于下面的代码片段所示：

```js
UserSchema.methods.authenticate = function(password) {
  return this.password === password;
};
```

这将允许您从任何`User`模型实例调用`authenticate()`方法，如下所示：

```js
user.authenticate('password');
```

正如您所看到的，定义自定义模型方法是保持项目正确组织并重用常见代码的好方法。在接下来的章节中，您将发现实例方法和静态方法都非常有用。

# 模型验证

在处理数据编组时的一个主要问题是验证。当用户向您的应用程序输入信息时，您经常需要在将信息传递给 MongoDB 之前验证该信息。虽然您可以在应用程序的逻辑层验证数据，但在模型级别进行此操作更有用。幸运的是，Mongoose 支持简单的预定义验证器和更复杂的自定义验证器。验证器在文档的字段级别定义，并在保存文档时执行。如果发生验证错误，则保存操作将被中止，并将错误传递给回调函数。

## 预定义验证器

Mongoose 支持不同类型的预定义验证器，其中大多数是特定于类型的。当然，任何应用程序的基本验证是值的存在。要在 Mongoose 中验证字段的存在，您需要在要验证的字段中使用`required`属性。假设您想在保存用户文档之前验证`username`字段的存在。为此，您需要对`UserSchema`进行以下更改：

```js
const UserSchema = new Schema({
  ...
  username: {
    type: String,
    trim: true,
    unique: true,
    required: true
  },
  ...
});
```

这将在保存文档时验证`username`字段的存在，从而防止保存不包含该字段的任何文档。

除了`required`验证器之外，Mongoose 还包括基于类型的预定义验证器，例如用于字符串的`enum`和`match`验证器。例如，要验证您的`email`字段，您需要将`UserSchema`更改如下：

```js
const UserSchema = new Schema({
  …
  email: {
    type: String,
    index: true,
    match: /.+\@.+\..+/
  },
  …
});
```

在这里使用`match`验证器将确保`email`字段值与给定的`regex`表达式匹配，从而防止保存任何不符合正确模式的电子邮件的文档。

另一个例子是`enum`验证器，它可以帮助您定义可用于该字段值的一组字符串。假设您添加了一个`role`字段。可能的验证如下所示：

```js
const UserSchema = new Schema({
  ...
  role: {
    type: String,
    enum: ['Admin', 'Owner', 'User']
  },
  ...
});
```

前面的条件将只允许插入这三个可能的字符串，从而防止您保存文档。

### 注意

要了解更多关于预定义验证器的信息，建议您访问[`mongoosejs.com/docs/validation.html`](http://mongoosejs.com/docs/validation.html)。

## 自定义验证器

除了预定义的验证器，Mongoose 还允许您定义自己的自定义验证器。使用`validate`属性来定义自定义验证器。`validate`属性的值应该是一个包含**验证**函数和错误消息的数组。假设您想要验证用户密码的长度。为此，您需要在`UserSchema`中进行以下更改：

```js
const UserSchema = new Schema({
  ...
  password: {
    type: String,
    validate: [
      function(password) {
        return password.length >= 6;
      },
      'Password should be longer'
    ]
  },
  ...
});
```

该验证器将确保您的用户密码至少为六个字符长，否则它将阻止文档的保存并将您定义的错误消息传递给回调函数。

Mongoose 验证是一个强大的功能，允许您控制模型并提供适当的错误处理，您可以用它来帮助用户理解出了什么问题。在接下来的章节中，您将学习如何使用 Mongoose 验证器来处理用户输入并防止常见的数据不一致性。

# 使用 Mongoose 中间件

Mongoose 中间件是可以拦截`init`、`validate`、`save`和`remove`实例方法的函数。中间件在实例级别执行，并且有两种类型：预中间件和后中间件。

## 使用预中间件

预中间件在操作发生前执行。例如，一个预保存中间件将在保存文档之前执行。这个功能使得预中间件非常适合更复杂的验证和默认值分配。

使用`pre()`方法定义预中间件，因此使用预中间件验证模型将如下所示：

```js
UserSchema.pre('save', function(next){
  if (...) {
    next()
  } else {
    next(new Error('An Error Occurred'));
  }
});
```

## 使用后中间件

后中间件在操作发生后执行。例如，一个后保存中间件将在保存文档后执行。这个功能使得后中间件非常适合记录应用程序逻辑。

使用`post()`方法定义后中间件，因此使用后中间件记录模型的`save()`方法将如下所示：

```js
UserSchema.post('save', function(next){
    console.log('The user "' + this.username +  '" details were saved.');
});
```

Mongoose 中间件非常适合执行各种操作，包括日志记录、验证和执行各种数据一致性操作。如果您现在感到不知所措，不要担心，因为在本书的后面，您将更好地理解这些内容。

### 注意

要了解更多关于中间件的信息，建议您访问[`mongoosejs.com/docs/middleware.html`](http://mongoosejs.com/docs/middleware.html)。

# 使用 Mongoose 的 ref 字段

尽管 MongoDB 不支持连接，但它支持使用名为**DBRef**的约定从一个文档到另一个文档的引用。DBRef 使得可以使用一个特殊字段来引用另一个文档，该字段包含集合名称和文档的`ObjectId`字段。Mongoose 实现了类似的行为，支持使用`ObjectID`模式类型和`ref`属性来支持文档引用。它还支持在查询数据库时将父文档与子文档进行关联。

为了更好地理解这一点，假设您为博客文章创建了另一个模式，称为`PostSchema`。因为用户是博客文章的作者，`PostSchema`将包含一个`author`字段，该字段将由`User`模型实例填充。因此，`PostSchema`将如下所示：

```js
const PostSchema = new Schema({
  title: {
    type: String,
    required: true
  },
  content: {
    type: String,
    required: true
  },
  author: {
    type: Schema.ObjectId,
    ref: 'User'
  }
});

mongoose.model('Post', PostSchema);
```

注意`ref`属性告诉 Mongoose`author`字段将使用`User`模型来填充值。

使用这个新模式是一个简单的任务。要创建一个新的博客文章，您需要检索或创建一个`User`模型的实例，创建一个`Post`模型的实例，然后将`post author`属性分配给`user`实例。示例如下：

```js
const user = new User();
user.save();

const post = new Post();
post.author = user;
post.save();
```

Mongoose 将在 MongoDB`post`文档中创建一个引用，并稍后使用它来检索引用的用户文档。

由于它只是对真实文档的`ObjectID`引用，Mongoose 将不得不使用“populate（）”方法来填充`post`实例中的`user`实例。为此，您将需要告诉 Mongoose 在检索文档时使用“populate（）”方法。例如，一个填充`author`属性的“find（）”方法将如下面的代码片段所示：

```js
Post.find().populate('author').exec((err, posts) => {
  ...
});
```

然后，Mongoose 将检索`posts`集合中的所有文档，并填充它们的`author`属性。

Mongoose 对此功能的支持使您能够放心地依赖对象引用来保持数据模型的组织。在本书的后面，您将学习如何引用以支持您的应用程序逻辑。

### 注意

要了解更多关于引用字段和填充的信息，建议您访问[`mongoosejs.com/docs/populate.html`](http://mongoosejs.com/docs/populate.html)。

# 总结

在本章中，您已经了解了强大的 Mongoose 模型。您连接到了您的 MongoDB 实例，并创建了您的第一个 Mongoose 模式和模型。您还学会了如何验证您的数据，并使用模式修改器和 Mongoose 中间件进行修改。您发现了虚拟属性和修改器，并学会了如何使用它们来改变文档的表示。您还发现了如何使用 Mongoose 来实现文档之间的引用。在下一章中，我们将介绍 Passport 身份验证模块，它将使用您的`User`模型来处理用户身份验证。
