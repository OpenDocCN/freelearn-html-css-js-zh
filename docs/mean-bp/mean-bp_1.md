# 第一章. 联系人管理器

在这一章中，你将学习如何构建联系人管理器应用程序。该应用程序将分为两个独立的部分：一部分是后端，我们使用 Express 编写的 Node.js API，另一部分是使用 Angular 2 精心制作的客户端应用程序。

别担心！这一章将更多地作为一个指南，设置一个基础项目，并在 Node.js 中了解 TDD（即**测试驱动开发**）。我们还将看到 Angular 2 的实际应用。我们不会在客户端编写测试，因为一章节中已经有很多东西要积累。

# 设置基本应用程序

开始的最佳方式是建立一个坚实的基础。这就是为什么我们将专注于构建我们应用程序的基础结构。一个好的基础给你模块化和灵活性，文件也应该很容易被你和你团队成员找到。

总是以简单的东西开始，然后围绕它构建。随着你的应用程序增长，你可能会超出你最初的应用程序结构，所以提前思考会给你带来长远的好处。

## 文件夹结构

在直接开始构建你的功能之前，你应该花点时间勾勒出你初始应用程序的结构。在规划过程中，一支笔和一张纸总是足够的，但我已经节省了一些时间并提出了一个初始版本：

```js
app/
--controllers/
--middlewares/
--models/
--routes/
config/
--environments/
--strategies/
tests/
--integration/
--unit/
public/
--app/
--src/
--assets/
--typings/
--package.json
--tsconfig.json
--typings.json
package.json
server.js
```

让我们看看我们对文件夹结构的更详细解释：

+   `app`: 这个文件夹包含应用程序中使用的所有服务器文件：

    +   `controllers`: 这个文件夹将存储应用程序控制器，主要是后端业务逻辑。

    +   `middlewares`: 在这个文件夹中，我们将存储所有将操作请求和响应对象的函数片段。一个很好的例子就是一个身份验证中间件。

    +   `models`: 这个文件夹将存储所有后端模型。

    +   `routes`: 这个文件夹将包含所有路由文件，这是我们定义所有 Express 路由的地方。

+   `config`: 所有应用程序配置文件都放在这里：

    +   `environments`: 这个文件夹包含根据当前环境加载的文件

    +   `strategies`: 所有你的身份验证策略都应该放在这里

+   `tests`: 这个文件夹包含测试应用程序后端逻辑所需的所有测试：

    +   `integration`: 如果某些东西使用了外部模块，创建一个集成测试是个好习惯

    +   `unit`: 这应该包含对小型代码单元的测试，例如密码散列

+   `public`: 这个文件夹应该包含我们应用程序提供的所有静态文件。我喜欢这种分离，因为它很容易让另一个 Web 服务器来处理我们的静态文件。比如说，你想让 nginx 来处理静态文件服务：

    +   `app`: 这是我们的客户端应用程序文件夹。所有编译后的 TypeScript 文件都将放在这里。这个文件夹应该自动填充。

    +   `src`：这个文件夹包含所有用于构建我们应用程序的客户端文件。我们将使用 TypeScript 来构建我们的 Angular 应用程序。

    +   `typings`：这包含 TypeScript 定义。

## 服务器端 package.json

在设置初始文件夹结构之后，接下来要做的事情是创建 `package.json` 文件。这个文件将包含所有应用程序的元数据和依赖项。`package.json` 文件将放置在我们的项目文件夹根目录。路径应该是 `contact-manager/package.json`：

```js
{
  "name": "mean-blueprints-contact-manager",
  "version": "0.0.9",
  "repository": {
    "type": "git",
    "url": "https://github.com/robert52/mean-blueprints-cm.git"
  },
  "engines": {
    "node": ">=4.4.3"
  },
  "scripts": {
    "start": "node app.js",
    "unit": "node_modules/.bin/mocha tests/unit/ --ui bdd --recursive --reporter spec --timeout 10000 --slow 900",
    "integration": "node_modules/.bin/mocha tests/integration/ --ui bdd --recursive --reporter spec --timeout 10000 --slow 900",
    "less": "node_modules/.bin/autoless public/assets/less public/assets/css --no-watch",
    "less-watch": "node_modules/.bin/autoless public/assets/less public/assets/css"
  },
  "dependencies": {
    "async": "⁰.9.2",
    "body-parser": "¹.15.0",
    "connect-mongo": "¹.1.0",
    "express": "⁴.13.4",
    "express-session": "¹.13.0",
    "lodash": "³.10.1",
    "method-override": "².3.5",
    "mongoose": "⁴.4.12",
    "passport": "⁰.2.2",
    "passport-local": "¹.0.0",
    "serve-static": "¹.10.2"
  },
  "devDependencies": {
    "autoless": "⁰.1.7",
    "chai": "².3.0",
    "chai-things": "⁰.2.0",
    "mocha": "².4.5",
    "request": "².71.0"
  }
}
```

我们在我们的 `package.json` 文件中添加了一些脚本以运行我们的单元和集成测试以及编译 Less 文件。你始终可以使用 `npm` 直接运行不同的脚本，而不是使用构建工具，如 Grunt 或 Gulp。

在撰写本书时，我们正在使用定义的依赖项及其版本。现在这应该足够了。让我们使用以下命令安装它们：

```js
$ npm install

```

你应该看到 `npm` 正在拉取一些文件并将必要的依赖项添加到 `node_modules` 文件夹中。耐心等待直到一切安装完成。你将返回到命令提示符。现在你应该看到已创建的 `node_modules` 文件夹，并且所有依赖项都已就位。

## 第一个应用程序文件

在做任何事情之前，我们需要为我们的环境创建一个简单的配置文件。让我们在 `config` 文件夹中创建文件，位于 `contact-manager/config/environments/development.js`，并添加以下内容：

```js
'use strict';

module.exports = {
  port: 3000,
  hostname: '127.0.0.1',
  baseUrl: 'http://localhost:3000',
  mongodb: {
    uri: 'mongodb://localhost/cm_dev_db'
  },
  app: {
    name: 'Contact manager'
  },
  serveStatic: true,
  session: {
    type: 'mongo',
    secret: 'u+J%E⁹!hx?piXLCfiMY.EDc',
    resave: false,
    saveUninitialized: true
  }
};
```

现在，让我们为我们的应用程序创建主要的 `server.js` 文件。这个文件将是我们的应用程序的核心。该文件应位于我们的文件夹根目录，`contact-manager/server.js`。从以下代码行开始：

```js
'use strict';

// Get environment or set default environment to development
const ENV = process.env.NODE_ENV || 'development';
const DEFAULT_PORT = 3000;
const DEFAULT_HOSTNAME = '127.0.0.1';

const http = require('http');
const express = require('express');
const config = require('./config');
const app = express();

var server;

// Set express variables
app.set('config', config);
app.set('root', __dirname);
app.set('env', ENV);

require('./config/mongoose').init(app);
require('./config/models').init(app);
require('./config/passport').init(app);
require('./config/express').init(app);
require('./config/routes').init(app);

// Start the app if not loaded by another module
if (!module.parent) {
  server = http.createServer(app);
  server.listen(
    config.port || DEFAULT_PORT,
    config.hostname || DEFAULT_HOSTNAME,
    () => {
      console.log(`${config.app.name} is running`);
      console.log(`   listening on port: ${config.port}`);
      console.log(`   environment: ${ENV.toLowerCase()}`);
    }
  );
}

module.exports = app;
```

我们定义了一些主要依赖项并初始化了应用程序必要的模块。为了模块化，我们将把我们的堆栈中的每个包放入单独的配置文件中。这些配置文件中会包含一些逻辑。我喜欢称它们为智能配置文件。

别担心！我们将逐个查看每个配置文件。最后，我们将导出我们的 Express 应用程序实例。如果我们的模块没有被另一个模块加载，例如，一个测试用例，那么我们可以安全地开始监听传入的请求。

## 创建 Express 配置文件

我们需要为 Express 创建一个配置文件。该文件应创建在 `config` 文件夹中，位于 `contact-manager/config/express.js`，并且我们必须添加以下代码行：

```js
'use strict';

const path = require('path');
const bodyParser = require('body-parser');
const methodOverride = require('method-override');
const serveStatic = require('serve-static');
const session = require('express-session');
const passport = require('passport');
const MongoStore = require('connect-mongo')(session);
const config = require('./index');

module.exports.init = initExpress;

function initExpress(app) {
  const root = app.get('root');
  const sessionOpts = {
    secret: config.session.secret,
    key: 'skey.sid',
    resave: config.session.resave,
    saveUninitialized: config.session.saveUninitialized
  };

  //common express configs
  app.use(bodyParser.urlencoded({ extended: true }));
  app.use(bodyParser.json());
  app.use(methodOverride());
  app.disable('x-powered-by');

  if (config.session.type === 'mongo') {
    sessionOpts.store = new MongoStore({
      url: config.mongodb.uri
    });
  }

  app.use(session(sessionOpts));
  app.use(passport.initialize());
  app.use(passport.session());

  app.use(function(req, res, next) {
    res.locals.app = config.app;

    next();
  });

  // always load static files if dev env
  if (config.serveStatic) {
    app.use(serveStatic(path.join(root, 'public')));
  }
};
```

到现在为止，你应该已经熟悉了前面代码中的许多行，例如，设置我们 Express 应用的期望体解析器。此外，我们还设置了会话管理，以防万一我们还需要设置服务器静态文件，我们定义了服务器文件的路径。

在生产环境中，你应该使用与默认内存存储不同的东西来存储会话。这就是为什么我们添加了一个特殊的会话存储，它将在 MongoDB 中存储数据。

获取全局环境配置文件的一个好做法是设置一个根配置文件，所有应用程序文件都将加载，创建一个名为`contact-manager/config/index.js`的新文件，并将以下代码添加到其中：

```js
'use strict';

var ENV = process.env.NODE_ENV || 'development';
var config = require('./environments/'+ENV.toLowerCase());

module.exports = config;
```

前面的代码将仅根据`NODE_ENV`进程环境变量加载必要的环境配置文件。如果环境变量不存在，则将考虑应用程序的默认开发状态。这是一个好的做法，以免我们犯错误并连接到错误的数据库。

通常，可以在启动您的 node 服务器时设置`NODE_ENV`变量；例如，在 Unix 系统中，您可以运行以下命令：

```js
$ NODE_ENV=production node server.js

```

## 设置 mocha 进行测试

在我们实现任何功能之前，我们将为其编写测试。Mocha 是一个基于 Node.js 的测试框架。这种方法将使我们能够知道我们将要编写的代码，并在编写客户端应用程序的任何一行代码之前测试我们的 Node.js API。

如果您没有 Mocha，您可以在全局范围内安装它。如果您希望 Mocha 在您的命令行中全局可用，请运行以下命令：

```js
$ npm install -g mocha

```

## 设置 Mongoose

为了在 MongoDB 中存储数据，我们将使用 Mongoose。Mongoose 提供了一个定义用于模型应用程序数据的模式的方法。我们已经在`package.json`文件中包含了 mongoose，因此它应该已经安装。

我们需要为我们的 mongoose 库创建一个配置文件。让我们创建我们的配置文件`contact-manager/config/mongoose.js`。首先，我们开始加载 Mongoose 库，获取适当的环境配置，并与数据库建立连接。将以下代码添加到`mongoose.js`文件中：

```js
'use strict';

const mongoose = require('mongoose');
const config = require('./index');

module.exports.init = initMongoose;

function initMongoose(app) {
  mongoose.connect(config.mongodb.uri);

  // If the Node process ends, cleanup existing connections
  process.on('SIGINT', cleanup);
  process.on('SIGTERM', cleanup);
  process.on('SIGHUP', cleanup);

  if (app) {
    app.set('mongoose', mongoose);
  }

  return mongoose;
};

function cleanup() {
  mongoose.connection.close(function () {
    console.log('Closing DB connections and stopping the app. Bye bye.');
    process.exit(0);
  });
}
```

此外，我们正在使用`cleanup()`函数来关闭与 MongoDB 数据库的所有连接。前面的代码将导出在主`server.js`文件中使用的必要`init()`函数。

# 管理联系人

现在我们有了启动开发和添加功能所需的文件，我们可以开始实现所有与联系人管理相关的业务逻辑。为此，我们首先需要定义联系人的数据模型。

## 创建联系人 mongoose 模式

我们的系统需要某种功能来存储可能的客户或只是其他公司的联系人。为此，我们将创建一个表示存储 MongoDB 中所有联系人的相同集合的联系人模式。我们将保持我们的联系人模式简单。让我们在`contact-manager/app/models/contact.js`中创建一个模型文件，它将包含模式，并将以下代码添加到其中：

```js
'use strict';

const mongoose = require('mongoose');
const Schema = mongoose.Schema;

var ContactSchema = new Schema({
  email:  {
    type: String
  },
  name: {
    type: String
  },
  city: {
    type: String
  },
  phoneNumber: {
    type: String
  },
  company: {
    type: String
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});

// compile and export the Contact model
module.exports = mongoose.model('Contact', ContactSchema);
```

以下表格给出了模式中字段的描述：

| 字段 | 描述 |
| --- | --- |
| `email` | 联系人的电子邮件地址 |
| `name` | 联系人的全名 |
| `company` | 联系人工作的公司的名称 |
| `phoneNumber` | 人员的完整电话号码或公司的电话号码 |
| `city` | 联系人的位置 |
| `createdAt` | 联系人对象被创建的日期 |

我们所有的模型文件都将注册在以下配置文件中，该文件位于 `contact-manager/config/models.js` 下。该文件的最终版本将类似于以下内容：

```js
'use strict';

module.exports.init = initModels;

function initModels(app) {
  let modelsPath = app.get('root') + '/app/models/';

  ['user', 'contact'].forEach(function(model) {
    require(modelsPath + model);
  });
};
```

## 描述联系人路由

为了与服务器通信，我们需要为客户端应用程序提供路由以供消费。这些将是响应客户端请求的端点（URI）。主要，我们的路由将发送 JSON 响应。

我们将首先描述联系人模块的 CRUD 功能。路由应公开以下功能：

+   创建新的联系人

+   通过 ID 获取联系人

+   获取所有联系人

+   更新联系人

+   通过 ID 删除联系人

在这个应用程序中，我们不会涵盖批量插入和删除。

以下表格显示了这些操作如何映射到 HTTP 路由和动词：

| 路由 | 动词 | 描述 | 数据 |
| --- | --- | --- | --- |
| `/contacts` | `POST` | 创建新的联系人 | `email`、`name`、`company`、`phoneNumber` 和 `city` |
| `/contacts` | `GET` | 从系统中获取所有联系人 |   |
| `/contacts/<id>` | `GET` | 获取特定的联系人 |   |
| `/contacts/<id>` | `PUT` | 更新特定的联系人 | `email`、`name`、`company`、`phoneNumber` 和 `city` |
| `/contacts/<id>` | `DELETE` | 删除特定的联系人 |   |

按照前面的表格作为指南，我们将描述我们的主要功能和使用 Mocha 进行测试。Mocha 允许我们通过提供封装我们期望的 `describe()` 函数的能力来描述我们正在实施的功能。函数的第一个参数是一个简单的字符串，描述了功能。第二个参数是一个函数体，代表了描述。

您已经创建了一个名为 `contact-manger/tests` 的文件夹。在您的 `tests` 文件夹中，创建另一个名为 `integration` 的文件夹。创建一个名为 `contact-manager/tests/integration/contact_test.js` 的文件，并添加以下代码：

```js
'use strict';

/**
 * Important! Set the environment to test
 */
process.env.NODE_ENV = 'test';

const http = require('http');
const request = require('request');
const chai = require('chai');
const userFixture = require('../fixtures/user');
const should = chai.should();

let app;
let appServer;
let mongoose;
let User;
let Contact;
let config;
let baseUrl;
let apiUrl;

describe('Contacts endpoints test', function() {

  before((done) => {
    // boot app
    // start listening to requests
  });

  after(function(done) {
    // close app
    // cleanup database
    // close connection to mongo
  });

  afterEach((done) => {
    // remove contacts
  });

  describe('Save contact', () => {});

  describe('Get contacts', () => {});

  describe('Get contact', function() {});

  describe('Update contact', function() {});

  describe('Delete contact', function() {});
});
```

在我们的测试文件中，我们要求了依赖项，并使用 Chai 作为我们的断言库。正如您所看到的，除了 `describe()` 函数外，mocha 还为我们提供了额外的方法：`before()`、`after()`、`beforeEach()` 和 `afterEach()`。

这些是钩子，可以是异步或同步的，但我们将使用它们的异步版本。钩子对于在运行测试之前准备先决条件非常有用；例如，您可以用模拟数据填充您的数据库或清理它。

在主要描述主体中，我们使用了三个钩子：`before()`、`after()` 和 `afterEach()`。在 `before()` 钩子中，它将在任何 `describe()` 函数之前运行，我们设置服务器监听指定的端口，并在服务器开始监听时调用 `done()` 函数。

`after()` 函数将在所有 `describe()` 函数运行完毕后执行，并将停止服务器运行。现在，`afterEach()` 钩子将在每个 `describe()` 函数之后运行，并允许我们在每个测试运行后从数据库中删除所有联系人。

最终版本可以在应用程序的代码包中找到。您仍然可以了解我们如何添加所有必要的描述。

### 创建联系人

我们还添加了四到五个单独的描述，这些描述将定义之前表格中的 CRUD 操作。首先，我们希望能够创建一个新的联系人。将以下代码添加到测试用例中：

```js
  describe('Create contact', () => {
    it('should create a new contact', (done) => {
      request({
        method: 'POST',
        url: `${apiUrl}/contacts`,
        form: {
          'email': 'jane.doe@test.com',
          'name': 'Jane Doe'
        },
        json:true
      }, (err, res, body) => {
        if (err) throw err;

        res.statusCode.should.equal(201);
        body.email.should.equal('jane.doe@test.com');
        body.name.should.equal('Jane Doe');
        done();
      });
    });
  });
```

### 获取联系人

接下来，我们希望从系统中获取所有联系人。以下代码应描述此功能：

```js
  describe('Get contacts', () => {
    before((done) => {
      Contact.collection.insert([
        { email: 'jane.doe@test.com' },
        { email: 'john.doe@test.com' }
      ], (err, contacts) => {
        if (err) throw err;

        done();
      });
    });

    it('should get a list of contacts', (done) => {
      request({
        method: 'GET',
        url: `${apiUrl}/contacts`,
        json:true
      }, (err, res, body) => {
        if (err) throw err;

        res.statusCode.should.equal(200);
        body.should.be.instanceof(Array);
        body.length.should.equal(2);
        body.should.contain.a.thing.with.property('email', 'jane.doe@test.com');
        body.should.contain.a.thing.with.property('email', 'john.doe@test.com');
        done();
      });
    });
  });
```

如您所见，我们在描述中添加了一个 `before()` 钩子。这是绝对正常的，并且可以这样做。Mocha 允许这种行为以便轻松设置先决条件。我们在获取所有联系人之前，使用批量插入 `Contact.collection.insert()` 将数据添加到 MongoDB 中。

### 通过 ID 获取联系人

当通过 ID 获取联系人时，我们还想检查插入的 ID 是否符合我们的 `ObjectId` 标准。如果未找到联系人，我们希望返回 404 HTTP 状态码：

```js
  describe('Get contact', function() {
    let _contact;

    before((done) => {
      Contact.create({
        email: 'john.doe@test.com'
      }, (err, contact) => {
        if (err) throw err;

        _contact = contact;
        done();
      });
    });

    it('should get a single contact by id', (done) => {
      request({
        method: 'GET',
        url: `${apiUrl}/contacts/${_contact.id}`,
        json:true
      }, (err, res, body) => {
        if (err) throw err;

        res.statusCode.should.equal(200);
        body.email.should.equal(_contact.email);
        done();
      });
    });

    it('should not get a contact if the id is not 24 characters', (done) => {
      request({
        method: 'GET',
        url: `${apiUrl}/contacts/U5ZArj3hjzj3zusT8JnZbWFu`,
        json:true
      }, (err, res, body) => {
        if (err) throw err;

        res.statusCode.should.equal(404);
        done();
      });
    });
  });
```

我们使用了 `.create()` 方法。对于单条插入，使用它更为方便，可以预先将数据填充到数据库中。当我们通过 ID 获取单个联系人时，我们想要确保这是一个有效的 ID，因此我们添加了一个测试来反映这一点，如果 ID 无效或未找到联系人，则应返回 `404 Not Found` 响应。

### 更新联系人

我们还希望能够使用给定的 ID 更新现有的联系人。添加以下代码来描述此功能：

```js
  describe('Update contact', () => {
    let _contact;

    before((done) => {
      Contact.create({
        email: 'jane.doe@test.com'
      }, (err, contact) => {
        if (err) throw err;

        _contact = contact;
        done();
      });
    });

    it('should update an existing contact', (done) => {
      request({
        method: 'PUT',
        url: `${apiUrl}/contacts/${_contact.id}`,
        form: {
          'name': 'Jane Doe'
        },
        json:true
      }, (err, res, body) => {
        if (err) throw err;

        res.statusCode.should.equal(200);
        body.email.should.equal(_contact.email);
        body.name.should.equal('Jane Doe');
        done();
      });
    });
  });
```

### 删除联系人

最后，我们将通过添加以下代码来描述删除联系人操作（CRUD 中的 DELETE）：

```js
  describe('Delete contact', () => {
    var _contact;

    before((done) => {
      Contact.create({
        email: 'jane.doe@test.com'
      }, (err, contact) => {
        if (err) throw err;

        _contact = contact;
        done();
      });
    });

    it('should update an existing contact', (done) => {
      request({
        method: 'DELETE',
        url: `${apiUrl}/contacts/${_contact.id}`,
        json:true
      }, (err, res, body) => {
        if (err) throw err;

        res.statusCode.should.equal(204);
        should.not.exist(body);
        done();
      });
    });
  });
```

在删除联系人后，服务器应响应 `HTTP 204 No Content` 状态码，这意味着服务器已成功解析请求并处理它，但由于联系人已成功删除，不应返回任何内容。

### 运行我们的测试

假设我们运行以下命令：

```js
$ mocha test/integration/contact_test.js

```

在这一点上，我们将收到大量的 `HTTP 404 Not Found` 状态码，因为我们的路由尚未实现。输出应类似于以下内容：

```js
 Contact
 Save contact
 1) should save a new contact
 Get contacts
 2) should get a list of contacts
 Get contact
 3) should get a single contact by id
 √ should not get a contact if the id is not 24 characters
 Update contact
 4) should update an existing contact
 Delete contact
 5) should update an existing contact

 1 passing (485ms)
 5 failing
 1) Contact Save contact should save a new contact:

 Uncaught AssertionError: expected 404 to equal 201
 + expected - actual

 +201
 -404

```

## 实现联系人路由

现在，我们将开始实现联系人 CRUD 操作。我们将首先创建我们的控制器。创建一个新的文件，`contact-manager/app/controllers/contact.js`，并添加以下代码：

```js
'use strict';

const _ = require('lodash');
const mongoose = require('mongoose');
const Contact = mongoose.model('Contact');
const ObjectId = mongoose.Types.ObjectId;

module.exports.create = createContact;
module.exports.findById = findContactById;
module.exports.getOne = getOneContact;
module.exports.getAll = getAllContacts;
module.exports.update = updateContact;
module.exports.remove = removeContact;

function createContact(req, res, next) {
  Contact.create(req.body, (err, contact) => {
    if (err) {
      return next(err);
    }

    res.status(201).json(contact);
  });
}
```

前面的代码所做的是导出控制器中所有用于 CRUD 操作的方法。为了创建一个新的联系人，我们使用 `Contact` 架构中的 `create()` 方法。

我们返回一个包含新创建联系人的 JSON 响应。如果发生错误，我们只需使用错误对象调用 `next()` 函数。我们将在稍后添加一个特殊处理程序来捕获所有错误。

让我们为我们的路由创建一个新的文件，`contact-manager/app/routes/contacts.js`。以下代码行应该是我们路由器的良好开端：

```js
'use strict';

const express = require('express');
const router = express.Router();
const contactController = require('../controllers/contact');

router.post('/contacts', auth.ensured, contactController.create);

module.exports = router;
```

假设我们现在使用这个运行测试，如下所示：

```js
$ mocha tests/integration/contact_test.js

```

我们应该得到类似以下的内容：

```js
Contact
 Create contact
 √ should save a new contact
 Get contacts
 1) should get a list of contacts
 Get contact
 2) should get a single contact by id
 √ should not get a contact if the id is not 24 characters
 Update contact
 3) should update an existing contact
 Delete contact
 4) should update an existing contact

 2 passing (502ms)
 4 failing

```

### 添加所有端点

接下来，我们将添加其余的路由，通过将以下代码添加到`contact-manager/app/routes/contact.js`文件中：

```js
router.param('contactId', contactController.findById);

router.get('/contacts', auth.ensured, contactController.getAll);
router.get('/contacts/:contactId', auth.ensured, contactController.getOne);
router.put('/contacts/:contactId', auth.ensured, contactController.update);
router.delete('/contacts/:contactId', auth.ensured, contactController.remove);
```

我们定义了所有路由，并为`contactId`路由参数添加了回调触发。在 Express 中，我们可以使用名为参数名称的`param()`方法添加回调触发。

回调函数类似于任何正常的路由回调，但它会多一个参数，表示路由参数的值。一个具体的例子如下：

```js
app.param('contactId', function(req, res, next, id) { 
  // do something with the id ...
});
```

按照前面的示例，当`:contactId`在路由路径中存在时，我们可以映射联系人加载逻辑，并将联系人提供给下一个处理器。

### 通过 ID 查找联系人

我们将在位于`contact-manager/app/controllers/contact.js`的控制器文件中添加其余缺失的功能：

```js
function findContactById(req, res, next, id) {
  if (!ObjectId.isValid(id)) {
    res.status(404).send({ message: 'Not found.'});
  }

  Contact.findById(id, (err, contact) => {
    if (err) {
      next(err);
    } else if (contact) {
      req.contact = contact;
      next();
    } else {
      next(new Error('failed to find contact'));
    }
  });
}
```

前面的函数是一个特殊情况。它将获取四个参数，最后一个将是与触发参数值匹配的 ID。

### 获取联系人信息

要获取所有联系人，我们将查询数据库。我们将根据创建日期对结果进行排序。一个好的做法是始终限制返回的数据集的大小。为此，我们使用`MAX_LIMIT`常量：

```js
function getAllContacts(req, res, next) {
  const limit = +req.query.limit || MAX_LIMIT;
  const skip = +req.query.offset || 0;
  const query = {};

  if (limit > MAX_LIMIT) {
    limit = MAX_LIMIT;
  }

  Contact
  .find(query)
  .skip(skip)
  .limit(limit)
  .sort({createdAt: 'desc'})
  .exec((err, contacts) => {
    if (err) {
      return next(err);
    }

    res.json(contacts);
  });
}
```

要返回单个联系人，你可以使用以下代码：

```js
function getOneContact(req, res, next) {
  if (!req.contact) {
    return next(err);
  }

  res.json(req.contact);
}
```

理论上，我们将在路由定义中拥有`:contactId`参数。在这种情况下，`param`回调会被触发，将请求的联系人填充到`req`对象中。

### 更新联系人

当更新联系人时，也应用了相同的原则；请求的实体应由`param`回调填充。我们只需将传入的数据分配给联系人对象，并将更改保存到 MongoDB 中：

```js
function updateContact(req, res, next) {
  let contact = req.contact;
  _.assign(contact, req.body);

  contact.save((err, updatedContact) => {
    if (err) {
      return next(err);
    }

    res.json(updatedContact);
  });
}
```

### 删除联系人

删除联系人应该相当简单，因为它没有依赖的文档。因此，我们可以简单地从数据库中删除文档，使用以下代码：

```js
function removeContact(req, res, next) {
  req.contact.remove((err) => {
    if (err) {
      return next(err);
    }

    res.status(204).json();
  });
}
```

## 运行联系人测试

到目前为止，我们应该已经实现了后端管理联系人的所有要求。为了测试一切，我们运行以下命令：

```js
$ mocha tests/integration/contact.test.js

```

输出应该类似于以下内容：

```js
 Contact

 Save contact
 √ should save a new contact
 Get contacts
 √ should get a list of contacts
 Get contact
 √ should get a single contact by id
 √ should not get a contact if the id is not 24 characters
 Update contact
 √ should update an existing contact
 Delete contact
 √ should update an existing contact

 6 passing (576ms)

```

这意味着所有测试都成功通过，并且我们已经实现了所有要求。

# 保护应用程序路由

你可能不希望让任何人看到你的联系人，所以是时候保护你的端点了。我们可以使用许多策略来在应用程序中验证受信任的用户。我们将使用经典的、基于状态的全局电子邮件和密码验证。这意味着会话将存储在服务器端。

记得我们在本章开头讨论了我们将如何将我们的会话存储在服务器端吗？我们选择了两个集成，一个是默认的内存会话管理，另一个是将会话存储在 MongoDB 中。所有这些都可以从环境配置文件中进行配置。

当涉及到在 Node.js 中处理认证时，一个好的选择是 Passport 模块，它是一块认证中间件。Passport 提供了一套全面的认证策略，使用简单的用户名和密码组合来支持 Facebook、Google、Twitter 等许多服务。

我们已经将此依赖项添加到我们的应用程序中，并在 express 配置文件中进行了必要的初始化。我们仍然需要添加一些内容，但在那之前，我们必须在我们的后端应用程序中创建一些可重用的组件。我们将创建一个辅助文件，这将简化我们与密码的交互。

## 描述密码辅助器

在我们深入探讨认证机制之前，我们需要能够在 MongoDB 中存储密码哈希而不是明文密码。我们希望创建一个辅助器来完成这个任务，使我们能够执行与密码相关的操作。

在`tests`文件夹中创建一个新的文件夹，命名为`unit`。添加一个新文件，`contact-manager/tests/unit/password.test.js`，然后向其中添加以下代码：

```js
'use strict';

const chai = require('chai');
const should = chai.should();
const passwordHelper = require('../../app/helpers/password');

describe('Password Helper', () => {
});
```

在我们的主要描述体中，我们将添加代表我们功能的详细段。添加以下代码：

```js
describe('#hash() - password hashing', () => {
});
describe('#verify() - compare a password with a hash', () => {
});
```

Mocha 还提供了一个`it()`函数，我们将使用它来设置一个具体的测试。`it()`函数与`describe()`非常相似，只不过我们只放置了功能应该执行的内容。对于断言，我们将使用 Chai 库。将以下代码添加到`tests/unit/password.test.js`文件中：

```js
  describe('#hash() - password hashing', () => {
    it('should return a hash and a salt from a plain string', (done) => {
      passwordHelper.hash('P@ssw0rd!', (err, hash, salt) => {
        if (err) throw err;

        should.exist(hash);
        should.exist(salt);
        hash.should.be.a('string');
        salt.should.be.a('string');
        hash.should.not.equal('P@ssw0rd!');
        done();
      });
    });

    it('should return only a hash from a plain string if salt is given', (done) => {
      passwordHelper.hash('P@ssw0rd!', 'secret salt', (err, hash, salt) => {
        if (err) throw err;

        should.exist(hash);
        should.not.exist(salt);
        hash.should.be.a('string');
        hash.should.not.equal('P@ssw0rd!');
        done();
      });
    });

    it('should return the same hash if the password and salt ar the same', (done) => {
      passwordHelper.hash('P@ssw0rd!', (err, hash, salt) => {
        if (err) throw err;

        passwordHelper.hash('P@ssw0rd!', salt, function(err, hashWithSalt) {
          if (err) throw err;

          should.exist(hash);
          hash.should.be.a('string');
          hash.should.not.equal('P@ssw0rd!');
          hash.should.equal(hashWithSalt);
          done();
        });
      });
    });
  });
```

`passwordHelper`还应测试密码是否与给定的哈希和盐组合匹配。为此，我们将添加以下描述方法：

```js
  describe('#verify() - compare a password with a hash', () => {
    it('should return true if the password matches the hash', (done) => {
      passwordHelper.hash('P@ssw0rd!', (err, hash, salt) => {
        if (err) throw err;

        passwordHelper.verify('P@ssw0rd!', hash, salt, (err, result) => {
          if (err) throw err;

          should.exist(result);
          result.should.be.a('boolean');
          result.should.equal(true);
          done();
        });
      });
    });

    it('should return false if the password does not matches the hash', (done) => {
      passwordHelper.hash('P@ssw0rd!', (err, hash, salt) => {
        if (err) throw err;

        passwordHelper.verify('password!', hash, salt, (err, result) => {
          if (err) throw err;

          should.exist(result);
          result.should.be.a('boolean');
          result.should.equal(false);
          done();
        });
      });
    });
  });
```

## 实现密码辅助器

我们将在以下文件中实现我们的密码辅助器：`contact-manager/app/helpers/password.js`。

我们密码辅助器的第一个描述描述了一个从明文密码创建哈希的函数。在我们的实现中，我们将使用一个密钥派生函数，它将从我们的密码计算哈希，也称为密钥拉伸。

我们将使用内置的 Node.js `crypto`库中的`pbkdf2`函数。该函数的异步版本接受一个明文密码并应用 HMAC 摘要函数。我们将使用`sha256`来获取给定长度的派生密钥，并通过多次迭代与盐结合。

我们希望对于两种情况都使用相同的哈希函数：当我们已经有一个密码哈希和一个盐时，以及当我们只有明文密码时。让我们看看我们的哈希函数的最终代码。添加以下内容：

```js
'use strict';

const crypto = require('crypto');
const len = 512;
const iterations = 18000;
const digest = 'sha256';

module.exports.hash = hashPassword;
module.exports.verify = verify;

function hashPassword(password, salt, callback) {
  if (3 === arguments.length) {
    crypto.pbkdf2(password, salt, iterations, len, digest, (err, derivedKey) => {
      if (err) {
        return callback(err);
      }

      return callback(null, derivedKey.toString('base64'));
    });
  } else {
    callback = salt;
    crypto.randomBytes(len, (err, salt) => {
      if (err) {
        return callback(err);
      }

      salt = salt.toString('base64');
      crypto.pbkdf2(password, salt, iterations, len, digest, (err, derivedKey) => {
        if (err) {
          return callback(err);
        }

        callback(null, derivedKey.toString('base64'), salt);
      });
    });
  }
}
```

让我们看看现在运行我们的测试会得到什么。运行以下命令：

```js
$ mocha tests/unit/password.test.js

```

输出应该类似于以下内容：

```js
 Password Helper
 #hash() - password hashing
 √ should return a hash and a salt from a plain string (269ms)
 √ should return only a hash from a plain string if salt is given (274ms)
 √ should return the same hash if the password and salt are the same (538ms)

3 passing (2s)

```

如您所见，我们已经成功实现了我们的哈希函数。所有测试用例的要求都已通过。请注意，运行测试可能需要最多 2 秒钟。不用担心这个问题；这是因为密钥拉伸函数需要时间从密码生成哈希值。

接下来，我们将实现 `verify()` 函数，该函数检查密码是否与现有用户的密码-哈希-盐组合匹配。根据我们的测试描述，此函数接受四个参数：明文密码、使用第三个盐参数生成的哈希值，以及回调函数。

回调函数接收两个参数：`err` 和 `result`。`result` 可以是 `true` 或 `false`。这将反映密码是否与现有的哈希值匹配。考虑到测试的约束和前面的解释，我们可以在 `password.helpr.js` 文件中添加以下代码：

```js
function verify(password, hash, salt, callback) {
  hashPassword(password, salt, (err, hashedPassword) => {
    if (err) {
      return callback(err);
    }

    if (hashedPassword === hash) {
      callback(null, true);
    } else {
      callback(null, false);
    }
  });
}
```

到目前为止，我们应该已经实现了测试中所有的规范。

## 创建用户 Mongoose 模式

为了授予应用程序中用户的访问权限，我们需要将它们存储在 MongoDB 集合中。我们将创建一个名为 `contact-manager/app/models/user.model.js` 的新文件，并添加以下代码：

```js
'use strict';

const mongoose = require('mongoose');
const passwordHelper = require('../helpers/password');
const Schema = mongoose.Schema;
const _ = require('lodash');

var UserSchema = new Schema({
  email:  {
    type: String,
    required: true,
    unique: true
  },
  name: {
    type: String
  },
  password: {
    type: String,
    required: true,
    select: false
  },
  passwordSalt: {
    type: String,
    required: true,
    select: false
  },
  active: {
    type: Boolean,
    default: true
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});
```

以下表格描述了模式中的字段：

| 字段 | 描述 |
| --- | --- |
| `email` | 用户的电子邮件。这用于识别用户。电子邮件在系统中将是唯一的。 |
| `name` | 用户的完整姓名。 |
| `password` | 这是用户提供的密码。它不会以明文形式存储在数据库中，而是以哈希形式存储。 |
| `passwordSalt` | 每个密码都将为给定用户生成一个唯一的盐。 |
| `active` | 这指定了用户的状态。可以是活跃的或非活跃的。 |
| `createdAt` | 用户创建的日期。 |

### 从用户模型描述身份验证方法

我们将描述一个用户身份验证方法。它将检查用户是否有有效的凭据。以下文件 `contact-manager/tests/integration/user.model.test.js` 应包含有关 `User` 模型的所有测试用例。以下代码行将测试 `authenticate()` 方法：

```js
  it('should authenticate a user with valid credentials', done => {
    User.authenticate(newUserData.email, newUserData.password, (err, user) => {
      if (err) throw err;

      should.exist(user);
      should.not.exist(user.password);
      should.not.exist(user.passwordSalt);
      user.email.should.equal(newUserData.email);
      done();
    });
  });

  it('should not authenticate user with invalid credentials', done => {
    User.authenticate(newUserData.email, 'notuserpassowrd', (err, user) => {
      if (err) throw err;

      should.not.exist(user);
      done();
    });
  });
```

### 实现身份验证方法

Mongoose 允许我们从模式中添加静态方法到编译后的模型。`authenticate()` 方法将通过电子邮件在数据库中搜索用户，并使用密码辅助器的 `verify()` 函数检查发送的密码是否匹配。

将以下代码行添加到 `contact-manager/app/models/user.js` 文件中：

```js
UserSchema.statics.authenticate = authenticateUser;

function authenticateUser(email, password, callback) {
  this
  .findOne({ email: email })
  .select('+password +passwordSalt')
  .exec((err, user) => {
    if (err) {
      return callback(err, null);
    }

    // no user found just return the empty user
    if (!user) {
      return callback(err, user);
    }

    // verify the password with the existing hash from the user
    passwordHelper.verify(
      password,
      user.password,
      user.passwordSalt,
      (err, result) => {
        if (err) {
          return callback(err, null);
        }

        // if password does not match don't return user
        if (result === false) {
          return callback(err, null);
        }

        // remove password and salt from the result
        user.password = undefined;
        user.passwordSalt = undefined;
        // return user if everything is ok
        callback(err, user);
      }
    );
  });
}
```

在前面的代码中，当从 MongoDB 中选择用户时，我们明确选择了密码和 `passwordSalt` 字段。这是必要的，因为我们设置了密码和 `passwordSalt` 字段，使其不在查询结果中选中。另外，需要注意的是，我们希望在返回用户时从结果中移除密码和盐。

## 身份验证路由

为了在我们的系统中进行认证，我们需要公开一些端点来执行必要的业务逻辑以验证具有有效凭证的用户。在深入任何代码之前，我们将描述期望的行为。

### 描述认证路由

我们只将查看认证功能集成测试的部分代码，该代码位于`contact-manager/tests/integration/authentication.test.js`中。它应该看起来像这样：

```js
  describe('Sign in user', () => {
    it('should sign in a user with valid credentials', (done) => {
      request({
        method: 'POST',
        url: baseUrl + '/auth/signin',
        form: {
          'email': userFixture.email,
          'password': 'P@ssw0rd!'
        },
        json:true
      }, (err, res, body) => {
        if (err) throw err;

        res.statusCode.should.equal(200);
        body.email.should.equal(userFixture.email);
        should.not.exist(body.password);
        should.not.exist(body.passwordSalt);
        done();
      });
    });

    it('should not sign in a user with invalid credentials', (done) => {
      request({
        method: 'POST',
        url: baseUrl + '/auth/signin',
        form: {
          'email': userFixture.email,
          'password': 'incorrectpassword'
        },
        json:true
      }, (err, res, body) => {
        if (err) throw err;

        res.statusCode.should.equal(400);
        body.message.should.equal('Invalid email or password.');
        done();
      });
    });
  });
```

因此，我们描述了一个`auth/signin`端点；它将使用电子邮件和密码组合来验证用户。我们正在测试两种场景。第一种是用户具有有效凭证，第二种是发送了错误的密码。

### 集成 Passport

我们在章节中提到了 Passport 并添加了一些基本逻辑，但我们仍然需要正确集成。Passport 模块应该已经安装，会话管理也已经就绪。因此，接下来我们需要创建一个适当的配置文件，`contact-manager/config/passport.js`，并添加以下内容：

```js
'use strict';

const passport = require('passport');
const mongoose = require('mongoose');
const User = mongoose.model('User');

module.exports.init = initPassport;

function initPassport(app) {
  passport.serializeUser((user, done) => {
    done(null, user.id);
  });

  passport.deserializeUser((id, done) => {
    User.findById(id, done);
  });

  // load strategies
  require('./strategies/local').init();
}
```

对于每个后续请求，我们需要将用户实例序列化和反序列化到会话中。我们只将用户的 ID 序列化到会话中。当后续请求被发起时，用户的 ID 被用来找到匹配的用户并在`req.user`中恢复数据。

Passport 为我们提供了使用不同策略来验证用户的能力。我们只将使用电子邮件和密码来验证用户。为了保持模块化，我们将策略移动到单独的文件中。所谓的本地策略，将用于使用电子邮件和密码验证用户，它将位于`contact-manager/config/strategies/local.js`文件中：

```js
'use strict';

const passport = require('passport');
const LocalStrategy = require('passport-local').Strategy;
const User = require('mongoose').model('User');

module.exports.init = initLocalStrategy;

function initLocalStrategy() {
  passport.use('local', new LocalStrategy({
      usernameField: 'email',
      passwordField: 'password'
    },
    (email, password, done) => {
      User.authenticate(email, password, (err, user) => {
        if (err) {
          return done(err);
        }

        if (!user) {
          return done(null, false, { message: 'Invalid email or password.' });
        }

        return done(null, user);
      });
    }
  ));
}
```

### 实现认证路由

现在我们已经启动了 passport，我们可以定义我们的认证控制器逻辑和适当的登录用户路由。创建一个名为`contact-manager/app/controllers/authentication.js`的新文件：

```js
'use strict';

const passport = require('passport');
const mongoose = require('mongoose');
const User = mongoose.model('User');

module.exports.signin = signin;

function signin(req, res, next) {
  passport.authenticate('local', (err, user, info) => {
    if (err) {
      return next(err);
    }

    if (!user) {
      return res.status(400).send(info);
    }

    req.logIn(user, (err) => {
      if (err) {
        return next(err);
      }

      res.status(200).json(user);
    });
  })(req, res, next);
}
```

在这里，我们使用 Passport 的`.authenticate()`函数来检查用户使用之前实现的本地策略的凭证。接下来，我们将添加认证路由，创建一个名为`contact-manager/app/routes/auth.js`的新文件，并添加以下代码行：

```js
'use strict';

var express = require('express');
var router = express.Router();
var authCtrl = require('../controllers/authentication');

router.post('/signin', authCtrl.signin);
router.post('/register', authCtrl.register);

module.exports = router;
```

注意，我们跳过了注册用户的功能，但别担心！最终的捆绑项目源代码将包含所有必要的逻辑。

## 限制对联系人路由的访问

我们创建了所有验证用户的要求。现在，我们需要限制对一些路由的访问，因此技术上我们将创建一个简单的 ACL。为了限制访问，我们将使用一个中间件来检查用户是否已认证。

让我们创建我们的中间件文件，`contact-manager/app/middlewares/authentication.js`。这个文件应该包含以下精心编写的代码：

```js
'use strict';

module.exports.ensured = ensureAuthenticated;

function ensureAuthenticated(req, res, next) {
  if (req.isAuthenticated()) {
    return next();
  }

  res.status(401).json({
    message: 'Please authenticate.'
  });
}
```

我们已经添加了必要的逻辑来限制用户访问联系人路由；那是在我们首次创建它们时。我们成功添加了所有必要的代码来管理联系人和限制对端点的访问。现在我们可以继续并开始构建我们的 Angular 2 应用程序。

# 将 Angular 2 集成到我们的应用程序中

前端应用程序将使用 Angular 2 构建。在撰写本书时，该项目仍在测试版，但开始尝试使用 Angular 并对环境有良好理解将很有帮助。大部分代码将遵循官方文档中工具和集成方法的视图。

当我们首次描述我们的文件夹结构时，我们看到了客户端应用程序的 `package.json` 文件。让我们看一下它，位于 `contact-manager/public/package.json` 路径下：

```js
{
  "private": true,
  "name": "mean-blueprints-contact-manager-client",
  "dependencies": {
    "systemjs": "⁰.19.25",
    "es6-shim": "⁰.35.0",
    "es6-promise": "³.0.2",
    "rxjs": "⁵.0.0-beta.2",
    "reflect-metadata": "⁰.1.2",
    "zone.js": "⁰.6.6",
    "angular2": "².0.0-beta.14"
  },
  "devDependencies": {
    "typings": "⁰.7.12",
    "typescript": "¹.8.9"
  }
}
```

要安装必要的依赖项，只需使用以下命令：

```js
$ npm install

```

您将看到 `npm` 正在拉取 `package.json` 文件中指定的不同包。

如您所见，我们将在客户端应用程序中使用 TypeScript。如果您已全局安装它，可以使用以下命令编译并监视 `.ts` 文件的变化：

```js
$ tsc -w

```

只会讨论应用程序最重要的部分。其余必要的文件和文件夹可以在最终的打包源代码中找到。

# 授予我们的应用程序访问权限

我们已经限制了我们对 API 端点的访问，因此现在我们必须从客户端应用程序中授予用户登录功能。我喜欢根据它们的领域上下文分组 Angular 2 应用程序文件。例如，所有我们的身份验证、注册和业务逻辑都应该放入一个单独的文件夹；我们可以称它为 `auth`。

如果您的模块目录增长，根据上下文按类型将其拆分为单独的文件夹是一种良好的做法。没有固定的文件数量。通常，当您需要移动文件时，您会有一个很好的感觉。您的文件应该始终易于定位，并从其在特定上下文中的位置提供足够的信息。

## AuthService

我们将使用 `AuthService` 来实现数据访问层并调用后端。这个服务将成为我们 API 的登录和注册功能之间的桥梁。创建一个名为 `contact-manager/src/auth/auth.service.ts` 的新文件，并将以下 TypeScript 代码添加到其中：

```js
import { Injectable } from 'angular2/core';
import { Http, Response, Headers } from 'angular2/http';
import { contentHeaders } from '../common/headers';

@Injectable()
export class AuthService {
  private _http: Http;

  constructor(http: Http) {
    this._http = http;
  }
}
```

我们导入必要的模块，定义 `AuthService` 类，并导出它。Injectable 标记元数据将标记我们的类以使其可供注入。为了与后端通信，我们使用 HTTP 服务。不要忘记在启动应用程序时添加 `HTTP_PROVIDERS`，以便服务在整个应用程序中可用。

要登录用户，我们将添加以下方法：

```js
  public signin(user: any) {
    let body = this._serialize(user);

    return this._http
    .post('/auth/signin', body, { headers: contentHeaders })
    .map((res: Response) => res.json());
  }
```

我们可以使用 `.map()` 操作符将响应转换为 JSON 文件。在执行 HTTP 请求时，这将返回一个 `Observable`。你可能已经猜到了——我们将大量使用 **RxJs**（**响应式扩展**），这是一个由 Angular 喜爱的第三方库。

RxJs 实现了异步可观察模式。换句话说，它使你能够处理异步数据流并应用不同的操作符。在 Angular 应用程序中广泛使用 Observables。在撰写本书时，Angular 2 提供了 RxJs 的 `Observable` 模块的简化版本。

不要担心；随着我们进一步深入本书，我们将熟悉这项技术和它的好处。现在让我们继续实现我们想要公开的其他缺失方法：

```js
  public register(user: any) {
    let body = this._serialize(user);

    return this._http
    .post('/auth/register', body, { headers: contentHeaders })
    .map((res: Response) => res.json());
  }

  private _serialize(data) {
    return JSON.stringify(data);
  }
```

我们在我们的服务中添加了 `register()` 方法，它将处理用户注册。此外，请注意，我们将序列化移动到了一个单独的私有方法中。我保留了此方法在同一类中，以便更容易理解，但你也可以将其移动到辅助类中。

## 用户登录组件

首先，我们将实现登录组件。让我们创建一个名为 `contact-manager/public/src/auth/sigin.ts` 的新文件，并添加以下 TypeScript 代码行：

```js
import { Component } from 'angular2/core';
import { Router, RouterLink } from 'angular2/router';
import { AuthService } from './auth.service';

export class Signin {
  private _authService: AuthService;
  private _router: Router;

  constructor(
    authService: AuthService,
    router: Router
  ) {
    this._authService = authService;
    this._router = router;
  }

  signin(event, email, password) {
    event.preventDefault();

    let data = { email, password };

    this._authService
    .signin(data)
    .subscribe((user) => {
      this._router.navigateByUrl('/');
    }, err => console.error(err));
  }
}
```

我们仍然需要在 `Signin` 类之前添加 `Component` 注解：

```js
@Component({
    selector: 'signin',
    directives: [
      RouterLink
    ],
    template: `
      <div class="login jumbotron center-block">
        <h1>Login</h1>
        <form role="form" (submit)="signin($event, email.value, password.value)">
          <div class="form-group">
            <label for="email">E-mail</label>
            <input type="text" #email class="form-control" id="email" placeholder="enter your e-mail">
          </div>
          <div class="form-group">
            <label for="password">Password</label>
            <input type="password" #password class="form-control" id="password" placeholder="now your password">
          </div>
          <button type="submit" class="button">Submit</button>
          <a href="#" [routerLink]="['Register']">Click here to register</a>
        </form>
      </div>
    `
})
```

`Signin` 组件将成为我们的登录表单，并使用 `AuthService` 与后端通信。在组件的模板中，我们使用带有 `#` 符号的局部变量标记电子邮件和密码字段。

正如我们之前所说的，HTTP 服务在发起请求时返回一个 `Observable`。这就是为什么我们可以订阅由我们的 `AuthService` 发起的请求生成的响应。在成功认证后，用户将被重定向到默认的主路径。

`Register` 组件将与 `Signin` 组件看起来相似，因此没有必要详细说明这一场景。`auth` 模块的最终版本将在源代码中提供。

## 自定义 HTTP 服务

为了限制对我们的 API 端点的访问，我们必须确保，如果请求未经授权，我们将用户重定向到登录页面。Angular 2 不支持拦截器，我们也不想为每个集成到我们服务中的请求添加处理器。

一个更方便的解决方案是在内置的 HTTP 服务之上构建我们自己的自定义服务。我们可以称它为 `AuthHttp`，用于授权 HTTP 请求。它的目的是检查请求是否返回了 `401 未授权 HTTP 状态码`。

我想进一步思考这个问题，并引入一些响应式编程的提示，因为我们已经在使用 RxJS。因此，我们可以从它提供的完整功能集中受益。响应式编程围绕数据展开。数据流在你的应用程序中传播，并对这些变化做出反应。

让我们开始工作，开始构建我们的自定义服务。创建一个名为 `contact-manager/public/src/auth/auth-http.ts` 的文件。我们将添加几行代码：

```js
import { Injectable } from 'angular2/core';
import { Http, Response, Headers, BaseRequestOptions, Request, RequestOptions, RequestOptionsArgs, RequestMethod } from 'angular2/http';
import { Observable } from 'rxjs/Observable';
import { Subject } from 'rxjs/Subject';
import { BehaviorSubject } from 'rxjs/Subject/BehaviorSubject';

@Injectable()
export class AuthHttp {
  public unauthorized: Subject<Response>;
  private _http: Http;

  constructor(http: Http) {
    this._http = http;
    this.unauthorized = new BehaviorSubject<Response>(null);
  }
}
```

我们在文件顶部导入了一些东西。我们将在本模块中需要所有这些。我们定义了一个名为 `unauthorized` 的公共属性，它是一个 Subject。Subject 既是 `Observable` 也是 `Observer`。这意味着我们可以将我们的主题订阅到后端数据源，并且所有观察者都可以订阅主题。

在我们的案例中，主题将作为我们的数据源和所有已订阅观察者之间的代理。如果一个请求未经授权，所有订阅者都会收到变更通知。这使得我们只需订阅主题，当我们检测到未经授权的请求时，就可以将用户重定向到登录页面。

要成功做到这一点，我们必须向我们的 `AuthHttp` 服务添加几个更多的方法：

```js
  private request(requestArgs: RequestOptionsArgs, additionalArgs?: RequestOptionsArgs) {
    let opts = new RequestOptions(requestArgs);

    if (additionalArgs) {
      opts = opts.merge(additionalArgs);
    }

    let req:Request = new Request(opts);

    return this._http.request(req).catch((err: any) => {
      if (err.status === 401) {
        this.unauthorized.next(err);
      }

      return Observable.throw(err);
    });
  }
```

上述方法创建了一个带有所需 `RequestOptions` 的新请求，并从基本 HTTP 服务中调用 `request` 方法。此外，`catch` 方法捕获所有状态码非 200 级别的请求。

使用这种技术，我们可以通过我们的 `unauthorized` 主题向所有订阅者发送未经授权的请求。现在我们已经有了我们的私有 `request` 方法，我们只需要添加其余的公共 HTTP 方法：

```js
  public get(url: string, opts?: RequestOptionsArgs) {
    return this.request({ url: url, method: RequestMethod.Get}, opts);
  }

  public post(url: string, body?: string, opts?: RequestOptionsArgs) {
    return this.request({ url: url, method: RequestMethod.Post, body: body}, opts);
  }

  public put(url: string, body?: string, opts?: RequestOptionsArgs) {
    return this.request({ url: url, method: RequestMethod.Put, body: body}, opts);
  }

  // rest of the HTTP methods ...
```

我只添加了最常用的方法；其余的可以在完整版本中找到。前面的代码调用了我们的请求方法，并为每种请求类型设置了必要的选项。理论上，我们创建了一个外观来处理未经授权的请求。

我认为我们已经取得了很好的进展，现在是时候继续我们的联系管理应用程序的其他模块了。

# 联系模块

此模块将包含管理联系所需的所有文件。正如我们之前讨论的，我们根据上下文将文件分组，与它们的领域相关。我们模块的起点将是数据层，这意味着我们将开始实现必要的服务。

## 联系服务

我们的联系服务将具有基本的 CRUD 操作和可订阅的 Observable 流。此实现将使用 Node.js 和 Express 构建的后端 API，但可以随时稍作努力转换为基于 WebSocket 的 API。

创建一个名为 `contact-manager/src/contact/contact.service.ts` 的新服务文件，并添加以下代码：

```js
import { Injectable } from 'angular2/core';
import { Response, Headers } from 'angular2/http';
import { Observable } from 'rxjs/Observable';
import { contentHeaders } from '../common/headers';
import { AuthHttp } from '../auth/auth-http';
import { Contact } from '../contact';

type ObservableContacts = Observable<Array<Contact>>;
type ObservableContact = Observable<Contact>;

const DEFAULT_URL = '/api/contacts';

@Injectable()
export class ContactService {
  public contact: ObservableContact;
  public contacts: ObservableContacts;

  private _authHttp: AuthHttp;
  private _dataStore: { contacts: Array<Contact>, contact: Contact };
  private _contactsObserver: any;
  private _contactObserver: any;
  private _url: string;

  constructor(authHttp: AuthHttp) {
    this._authHttp = authHttp;
    this._url = DEFAULT_URL;
    this._dataStore = { contacts: [], contact: new Contact() };
    this.contacts = new Observable(
      observer => this._contactsObserver = observer
    ).share();
    this.contact = new Observable(
      observer => this._contactObserver = observer
    ).share();
  }
}
```

在联系服务中，我们有几个动态部分。首先，我们定义了我们的 Observables，以便任何其他组件或模块都可以订阅并开始获取数据流。

其次，我们声明了一个私有数据存储。这就是我们将存储我们的联系信息的地方。这是一个好的实践，因为你可以轻松地从内存中返回所有资源。

此外，在我们的服务中，当生成新的 Observables 实例时，我们将保留返回的 Observers。使用 Observers，我们可以将新的数据流推送到我们的 Observables。

在我们的公共方法中，我们将公开获取所有联系人、获取一个、更新和删除的功能。为了获取所有联系人，我们将向我们的`ContactService`添加以下方法：

```js
  public getAll() {
    return this._authHttp
    .get(`${this._url}`, { headers: contentHeaders} )
    .map((res: Response) => res.json())
    .map(data => {
      return data.map(contact => {
        return new Contact(
          contact._id,
          contact.email,
          contact.name,
          contact.city,
          contact.phoneNumber,
          contact.company,
          contact.createdAt
        )
      });
    })
    .subscribe((contacts: Array<Contact>) => {
      this._dataStore.contacts = contacts;
      this._contactsObserver.next(this._dataStore.contacts);
    }, err => console.error(err));
  }
```

我们使用自定义构建的`AuthHttp`服务从我们的 Express 应用程序加载数据。当收到响应时，我们将它转换成 JSON 文件，然后，我们为数据集中的每个实体实例化一个新的联系人。

我们不是从 HTTP 服务返回整个`Observable`，而是使用我们的内部数据存储来持久化所有联系人。在我们成功更新数据存储的新数据后，我们将更改推送到我们的`contactsObserver`。

任何订阅我们联系人流的组件都将从`Observable`数据流中获得新值。这样，我们总是通过一个单一的入口点保持我们的组件同步。

我们公共方法的逻辑大部分相同，但我们仍然有一些独特的元素，例如，更新方法：

```js
  public update(contact: Contact) {
    return this._authHttp
    .put(
      `${this._url}/${contact._id}`,
      this._serialize(contact),
      { headers: contentHeaders}
    )
    .map((res: Response) => res.json())
    .map(data => {
      return new Contact(
        data._id,
        data.email,
        data.name,
        data.city,
        data.phoneNumber,
        contact.company,
        data.createdAt
      )
    })
    .subscribe((contact: Contact) => {
      // update the current list of contacts
      this._dataStore.contacts.map((c, i) => {
        if (c._id === contact._id) {
          this._dataStore.contacts[i] = contact;
        }
      });
      // update the current contact
      this._dataStore.contact = contact;
      this._contactObserver.next(this._dataStore.contact);
      this._contactsObserver.next(this._dataStore.contacts);
    }, err => console.error(err));
  }
```

`update`方法几乎与`create()`方法相同，但它将联系人的 ID 作为 URL 参数。我们不是将新值推送到数据流中，而是从`Http`服务返回`Observable`，以便从调用模块应用操作。

现在，如果我们想直接在`datastore`上做出更改并通过`contacts`数据流推送新值，我们可以在删除联系人的方法中展示这一点：

```js
  public remove(contactId: string) {
    this._authHttp
    .delete(`${this._url}/${contactId}`)
    .subscribe(() => {
      this._dataStore.contacts.map((c, i) => {
        if (c._id === contactId) {
          this._dataStore.contacts.splice(i, 1);
        }
      });
      this._contactsObserver.next(this._dataStore.contacts);
    }, err => console.error(err));
  }
```

我们简单地使用`map()`函数来找到我们删除的联系人并将其从内部存储中移除。之后，我们向订阅者发送新数据。

## 联系人组件

随着我们已将所有与接触域相关的功能移动到一起，我们可以在模块中定义一个主要组件。让我们称它为`contact-manager/public/src/contact/contact.component.ts`。添加以下代码行：

```js
import { Component } from 'angular2/core';
import { RouteConfig, RouterOutlet } from 'angular2/router';
import { ContactListComponent } from './contact-list.component';
import { ContactCreateComponent } from './contact-create.component';
import { ContactEditComponent } from './contact-edit.component';

@RouteConfig([
  { path: '/', as: 'ContactList', component: ContactListComponent, useAsDefault: true },
  { path: '/:id', as: 'ContactEdit', component: ContactEditComponent },
  { path: '/create', as: 'ContactCreate', component: ContactCreateComponent }
])
@Component({
    selector: 'contact',
    directives: [
      ContactListComponent,
      RouterOutlet
    ],
    template: `
      <router-outlet></router-outlet>
    `
})
export class ContactComponent {
  constructor() {}
}
```

我们组件没有与之关联的逻辑，但我们使用了`RouterConfig`注解。路由配置装饰器接受一个路由数组。配置中指定的每个路径都将与浏览器的 URL 匹配。每个路由将加载挂载的组件。为了在模板中引用路由，我们需要给它们一个名称。

现在，最吸引人的部分是我们可以将这个组件和配置的路由一起使用，并将其挂载到另一个组件上以拥有`Child`/`Parent`路由。在这种情况下，它变成了嵌套路由，这是 Angular 2 中添加的一个非常强大的功能。

我们应用程序的路由将具有树状结构；其他组件将加载它们配置的路由中的组件。我对这个功能感到非常惊讶，因为它使我们能够真正模块化我们的应用程序并创建令人惊叹的可重用模块。

## 列出联系人组件

在先前的组件中，我们使用了三个不同的组件并将它们挂载到不同的路由上。我们不会讨论每个组件，所以我们将选择其中一个。因为我们已经在`Signin`组件中处理过表单，让我们尝试做一些不同的事情并实现列出联系人的功能。

创建一个名为`contact-manager/public/src/contact/contact-list.component.ts`的新文件，并为你的组件添加以下代码：

```js
import { Component, OnInit } from 'angular2/core';
import { RouterLink } from 'angular2/router';
import { ContactService } from '../contact.service';
import { Contact } from '../contact';

@Component({
    selector: 'contact-list',
    directives: [RouterLink],
    template: `
      <div class="row">
        <h4>
          Total contacts: <span class="muted">({{contacts.length}})</span>
          <a href="#" [routerLink]="['ContactCreate']">add new</a>
        </h4>
        <div class="contact-list">
          <div class="card-item col col-25 contact-item"
            *ngFor="#contact of contacts">
            <img src="img/{{ contact.image }}" />
            <h3>
              <a href="#" [routerLink]="['ContactEdit', { id: contact._id }]">
                {{ contact.name }}
              </a>
            </h3>
            <p>
              <span>{{ contact.city }}</span>
              <span>·</span>
              <span>{{ contact.company }}</span>
            </p>
            <p><span>{{ contact.email }}</span></p>
            <p><span>{{ contact.phoneNumber }}</span></p>
          </div>
        </div>
      </div>
    `
})
export class ContactListComponent implements OnInit {
  public contacts: Array<Contact> = [];
  private _contactService: ContactService;

  constructor(contactService: ContactService) {
    this._contactService = contactService;
  }

  ngOnInit() {
    this._contactService.contacts.subscribe(contacts => {
      this.contacts = contacts;
    });
    this._contactService.getAll();
  }
}
```

在我们的组件的`ngOnInit()`中，我们订阅联系人数据流。之后，我们从后端检索所有联系人。在模板中，我们使用`ngFor`遍历数据集并显示每个联系人。

## 创建联系人组件

现在我们可以在应用程序中列出联系人，我们也应该能够添加新条目。记住，我们之前使用`RouterLink`来导航到`CreateContact`路由。

之前的路由将加载`CreateContactComponent`，这将使我们能够通过 Express API 将新的联系人条目添加到我们的数据库中。让我们创建一个新的组件文件`public/src/contact/components/contact-create.component.ts`：

```js
import { Component, OnInit } from 'angular2/core';
import { Router, RouterLink } from 'angular2/router';
import { ContactService } from '../contact.service';
import { Contact } from '../contact';

@Component({
    selector: 'contact-create,
    directives: [RouterLink],
    templateUrl: 'src/contact/components/contact-form.html'
})
export class ContactCreateComponent implements OnInit {
  public contact: Contact;
  private _router: Router;
  private _contactService: ContactService;

  constructor(
    contactService: ContactService,
    router: Router
  ) {
    this._contactService = contactService;
    this._router = router;
  }

  ngOnInit() {
    this.contact = new Contact();
  }

  onSubmit(event) {
    event.preventDefault();

    this._contactService
    .create(this.contact)
    .subscribe((contact) => {
      this._router.navigate(['ContactList']);
    }, err => console.error(err));
  }
}
```

我们不是使用内嵌模板，而是使用外部模板文件，该文件通过组件注解中的`templateUrl`属性进行配置。每种情况都有其优缺点。使用外部模板文件的优点是您可以重复使用同一文件为多个组件。

写这本书的时候，Angular 2 的一个缺点是难以使用相对路径到模板文件，这会使你的组件不太便携。此外，我喜欢保持我的模板简短，这样它们可以轻松地放在组件内部，所以在大多数情况下，我可能会使用内嵌模板。

让我们看看在进一步讨论组件之前的模板，`public/src/contact/components/contact-form.html`：

```js
<div class="row contact-form-wrapper">
  <a href="#" [routerLink]="['ContactList']">&lt; back to contacts</a>
  <h2>Add new contact</h2>
  <form role="form"
    (submit)="onSubmit($event)">

    <div class="form-group">
      <label for="name">Full name</label>
      <input type="text" [(ngModel)]="contact.name"
        class="form-control" id="name" placeholder="Jane Doe">
    </div>
    <div class="form-group">
      <label for="email">E-mail</label>
      <input type="text" [(ngModel)]="contact.email"
        class="form-control" id="email" placeholder="jane.doe@example.com">
    </div>
    <div class="form-group">
      <label for="city">City</label>
      <input type="text"
        [(ngModel)]="contact.city"
        class="form-control" id="city" placeholder="a nice place ...">
    </div>
    <div class="form-group">
      <label for="company">Company</label>
      <input type="text"
        [(ngModel)]="contact.company"
        class="form-control" id="company" placeholder="working at ...">
    </div>
    <div class="form-group">
      <label for="phoneNumber">Phone</label>
      <input type="text"
        [(ngModel)]="contact.phoneNumber"
        class="form-control" id="phoneNumber" placeholder="mobile or landline">
    </div>

    <button type="submit" class="button">Submit</button>
  </form>
</div>
```

在模板中，我们使用组件的`onSubmit()`方法来处理表单提交，并在这种情况下创建一个新的联系人并将数据存储在 MongoDB 中。当我们成功创建联系人时，我们希望导航到`ContactList`路由。

我们没有使用局部变量，而是使用`ngModel`进行双向数据绑定，每个输入映射到联系人对象的属性。现在，每次用户更改输入值时，这些值都会存储在联系人对象中，并在提交时通过网络发送到后端。

`RouterLink`用于从模板中构建导航到`ContactList`组件。我留下了一个小的改进，即视图标题在创建和编辑时都将相同，更确切地说，“添加新联系人”，我会让你自己想出来。

## 编辑现有联系人

在编辑联系人时，我们希望从后端 API 通过 ID 加载特定资源，并对该联系人进行更改。幸运的是，在 Angular 中实现这一点相当简单。创建一个新的文件`public/src/contact/components/contact-edit.component.ts`：

```js
import { Component, OnInit } from 'angular2/core';
import { RouteParams, RouterLink } from 'angular2/router';
import { ContactService } from '../contact.service';
import { Contact } from '../contact';

@Component({
    selector: 'contact-edit',
    directives: [RouterLink],
    templateUrl: 'src/contact/components/contact-form.html'
})
export class ContactEditComponent implements OnInit {
  public contact: Contact;
  private _contactService: ContactService;
  private _routeParams: RouteParams;

  constructor(
    contactService: ContactService,
    routerParams: RouteParams
  ) {
    this._contactService = contactService;
    this._routeParams = routerParams;
  }

  ngOnInit() {
    const id: string = this._routeParams.get('id');
    this.contact = new Contact();
    this._contactService
    .contact.subscribe((contact) => {
      this.contact = contact;
    });
    this._contactService.getOne(id);
  }

  onSubmit(event) {
    event.preventDefault();

    this._contactService
    .update(this.contact)
    .subscribe((contact) => {
      this.contact = contact;
    }, err => console.error(err));
  }
}
```

我们离`ContactCreateComponent`并不远，类的结构几乎相同。我们不是使用`Router`，而是使用`RouteParams`从 URL 中加载 ID，并从 Express 应用程序中检索所需的联系人。

我们订阅`ContactService`返回的联系人`Observable`以获取新数据。换句话说，我们的组件将响应数据流，当数据可用时，它将显示给用户。

当提交表单时，我们更新 MongoDB 中持久化的联系人，并使用从后端新鲜接收的数据更改视图的`contact`对象。

# 最后的润色

我们已经将所有必要的模块添加到我们的应用程序中。我们还应该最终检查我们的主应用程序组件，该组件位于以下路径下—`contact-manager/public/src/app.component.ts`：

```js
import { Component } from 'angular2/core';
import { RouteConfig, RouterOutlet } from 'angular2/router';
import { Router } from 'angular2/router';
import { AuthHttp } from './auth/auth-http';
import { Signin } from './auth/signin';
import { Register } from './auth/register';
import { ContactComponent } from './contact/components/contact.component';

@RouteConfig([
  { path: '/signin', as: 'Signin', component: Signin },
  { path: '/register', as: 'Register', component: Register },
  { path: '/contacts/...', as: 'Contacts', component: ContactComponent, useAsDefault: true }
])
@Component({
    selector: 'cm-app',
    directives: [
      Signin,
      Register,
      ContactComponent,
      RouterOutlet
    ],
    template: `
      <div class="app-wrapper col card whiteframe-z2">
        <div class="row">
          <h3>Contact manager</h3>
        </div>
        <router-outlet></router-outlet>
      </div>
    `
})
export class AppComponent {
  private _authHttp: AuthHttp;
  private _router: Router;

  constructor(authHttp: AuthHttp, router: Router) {
    this._authHttp = authHttp;
    this._router = router;
    this._authHttp.unauthorized.subscribe((res) => {
      if (res) {
        this._router.navigate(['./Signin']);
      }
    });
  }
}
```

我们将所有组件挂载到它们特定的路由上。此外，当我们挂载`Contact`组件时，我们将引入组件中配置的所有路由。

为了在请求未授权时得到通知，我们订阅了`AuthHttp`服务的`unauthorized`数据流。如果请求需要身份验证，我们将用户重定向到登录页面。

我们应用程序的启动文件看起来可能如下所示：

```js
import { bootstrap } from 'angular2/platform/browser';
import { provide } from 'angular2/core';
import { HTTP_PROVIDERS } from 'angular2/http';
import { ROUTER_PROVIDERS, LocationStrategy, HashLocationStrategy } from 'angular2/router';
import { AuthHttp } from './auth/auth-http';
import { AuthService } from './auth/auth.service';
import { ContactService } from './contact/contact.service';
import { AppComponent } from './app.component';

import 'rxjs/add/operator/map';
import 'rxjs/add/operator/share';
import 'rxjs/add/operator/debounceTime';
import 'rxjs/add/operator/catch';
import 'rxjs/add/observable/throw';

bootstrap(AppComponent, [
  ROUTER_PROVIDERS,
  HTTP_PROVIDERS,
  AuthService,
  AuthHttp,
  ContactService,
  provide(LocationStrategy, {useClass: HashLocationStrategy})
]);
```

我们导入并定义了必要的提供者，并添加了我们从 RxJs 中使用的操作符。这是因为 Angular 默认情况下只使用 Observable 模块的简化版本。

通过联系模块，我们使用了一个名为`Contact`的自定义类，它扮演着`Contact`模型的角色。每次我们想要确保我们正在处理一个联系人实体时，它都会被实例化。此外，TypeScript 的另一个优点是它使我们能够使用结构化代码。

当我们需要初始值时，类非常有用，例如，在我们的组件中，我们使用了`contact.image`属性来显示联系人的个人资料图片。这还没有在后端实现，所以我们使用一个模拟的 URL 作为图片。让我们看看`Contact`类，`public/src/contact/contact.ts`：

```js
export class Contact {
  _id: string;
  email: string;
  name: string;
  city: string;
  phoneNumber: string;
  company: string;
  image: string;
  createdAt: string;

  constructor(
    _id?: string,
    email?: string,
    name?: string,
    city?: string,
    phoneNumber?: string,
    company?: string,
    createdAt?: string
  ) {
    this._id = _id;
    this.email = email;
    this.name = name;
    this.city = city;
    this.phoneNumber = phoneNumber;
    this.company = company;
    this.image = 'http://placehold.it/171x100';
    this.createdAt = createdAt;
  }
}
```

如你所见，我们只定义了联系人实例可以拥有的属性，并为`image`属性创建了一个默认值。标记为`?`的构造函数参数是可选的。

在这个时候，我们应该已经一切就绪；如果你错过了什么，你可以查看代码的最终版本。

本章的关键要点如下：

+   使用 Node.js、Express 和 MongoDB 构建后端 Web 服务

+   在实际实现功能之前先编写测试

+   使用 Passport 保护我们的 API 路由

+   使 Angular 2 和 Express 通信并协同工作

+   进入反应式扩展和反应式编程

+   构建自定义的 Angular HTTP 服务

# 摘要

这就结束了这一相当入门性的章节。

我们从实现后端逻辑到学习在实际实现之前编写测试，全面地进行了全栈开发。我们从 MongoDB 资源公开了 RESTful 路由。我们还构建了一个小的 Angular 2 前端应用程序，该应用程序与 Web 服务器进行交互。

在下一章中，我们将更深入地探讨 MongoDB，并开始处理货币数据。这将是一次有趣的旅程！
