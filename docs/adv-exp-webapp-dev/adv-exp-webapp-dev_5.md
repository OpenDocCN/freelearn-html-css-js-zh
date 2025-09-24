# 第五章 安全

在本章中，我们将使用 GitHub 账户和**OAuth 2.0**令牌来认证用户。这将允许我们保护网站并支持多个用户；目前我们有一个硬编码的令牌和用户。我们还将向我们的网站添加 HTTPS，并探索一些我们可以用来保护其他常见安全漏洞的模块。

# 设置 Passport

Passport 是 Node 的认证中间件，通过插件支持多种认证策略，包括基本认证、OAuth 和 OAuth 2.。Passport 通过定义一个用于认证请求的路由中间件来工作。

让我们安装 Passport：

```js
npm install passport --save

```

Passport 不包含 GitHub 策略；为此我们需要安装`passport-github`；这是一个使用 OAuth 2.0 API 进行 GitHub 身份验证的策略：

```js
npm install passport-github --save

```

# 使用 Cucumber 和 Zombie.js 进行验收测试

OAuth 认证使用回调机制；这对于使用 SuperTest 等集成测试工具进行测试来说很麻烦；我们需要一点更端到端的东西。

Cucumber 允许团队使用一种简单的纯文本语言**Gherkin**来描述软件行为。描述此行为的流程有助于开发；输出作为文档，可以作为一系列测试自动运行。让我们安装 cucumber：

```js
npm install -g cucumber

```

Zombie.js 是一个用于进行无头全栈测试的简单、轻量级框架。让我们安装`Zombie.js`：

```js
npm install zombie --save-dev

```

让我们使用 Grunt 任务自动化运行 Cucumber：

```js
npm install grunt-cucumber --save-dev

```

将以下内容添加到我们的`gruntfile` `./gruntfile.js`中。`files`部分定义了我们的特征文件的位置，而`options:steps`定义了我们的步骤定义文件的位置：

```js
    cucumberjs: {
      files: 'features',
      options: {
        steps: "features/step_definitions",format: "pretty"
      }
    },
```

# 功能：认证

```js
As a vision user
I want to be able to authenticate via Github
So that I can view project activity
```

让我们创建我们的第一个特征文件`./features/authentication.feature`。以下特征文件包含一个`Feature`部分，对于敏捷开发人员来说，这将定义故事及其对业务的价值，以及一系列场景。我们的验收标准；用 Gherkin 语言编写。

以下`Authenticate`功能包含两个场景，包括一个用于登录，标题为`用户成功登录`，以及一个用于登出，标题为`用户成功登出`：

```js
Feature: Authentication
As a vision user
I want to be able to authenticate via Github
So that I can view project activity 

  Scenario: User logs in successfully
    Given I have a GitHub Account
    When I click the GitHub authentication button
    Then I should be logged in
    And I should see my name and a logout link

  Scenario: User logs out successfully
    Given I am logged in to Vision
    When I click the logout button
    Then I should see the GitHub login button
```

让我们使用我们的 Grunt 任务运行 Cucumber：

```js
grunt cucumberjs

```

这将生成以下输出：

```js
2 scenarios (2 undefined)
7 steps (7 undefined)
You can implement step definitions for undefined steps with these snippets:
this.Given(/^I have a GitHub Account$/, function(callback) {
  callback.pending();
});

this.When(/^I click the GitHub authentication button$/, function(callback) {
  callback.pending();
});

this.Then(/^I should be logged in$/, function(callback) {
  callback.pending();
});

this.Then(/^I should see my name and a logout link$/, function(callback) {
  callback.pending();
});

this.Given(/^I am logged in to Vision$/, function(callback) {
  callback.pending();
});

this.When(/^I click the logout button$/, function(callback) {
  callback.pending();
});

this.Then(/^I should see the GitHub login button$/, function(callback) {
  callback.pending();
});
```

从前面的输出中，你可以看到 Cucumber 生成了一系列设置为`pending`的存根步骤。这些步骤代表我们在`./features/authentication/authentication.feature`特征文件中定义的`Given`、`When`和`Then`场景。

我们可以使用这些步骤来实现我们的 Cucumber 测试。让我们创建一个步骤定义文件`./features/step_definitions/authentication/authenticate.js`：

```js
var steps = function() {
  var Given = When = Then = this.defineStep;
  ..add generated steps here
};

module.exports = steps;
```

让我们使用我们的 Grunt 任务运行 Cucumber：

```js
grunt cucumberjs

```

我们得到以下输出：

```js
2 scenarios (2 pending)
7 steps (2 pending, 5 skipped)

```

现在，我们已经准备好开始实现我们的第一个场景。

## 场景：用户成功登录

让我们开始实现这个场景。首先，我们需要一个 GitHub `clientId`和`clientSecret`。访问您的 GitHub 账户，点击**设置**然后**应用程序**，再次点击**注册新应用程序**。通过添加`homepage` URL 和`callback` URL（与我们的主页相同）来填写表单，并将生成一个`clientId`和`clientSecret`。

让我们将这些详细信息添加到我们的配置文件`./config/*.json`中：

```js
"auth": {
    "homepage": "http://127.0.0.1:3000"
  , "callback": "http://127.0.0.1:3000/auth/github/callback"
  , "clientId": "5bb691b4ebb5417f4ab9"
  , "clientSecret": "15310740929666983d52808dda32417d733791d0"
}
```

让我们移除在第二章中设置的临时登录，*构建 Web API*，并移除以下行及其所有相关代码`./lib/routes/project.js`：

```js
, login = require('../../test/login');
```

我们现在准备好实现我们的 GitHub 策略`./lib/github/authentication.js`。我们首先定义一个函数`GitHubAuth`；我们导入`passport`和`passport-github`模块。我们实例化一个`GitHubStrategy`，将其添加到`passport`中，并传递一个`clientID`、`clientSecret`、一个`callbackUrl`和一个`verify`函数（所有 passport 策略都需要一个验证函数），当 GitHub 进行身份验证并返回一个`accessToken`、`refreshToken`和一个`profile`时，该函数将被调用。

在这个验证函数内部，我们有通过从`callback`函数传递一个`false`来拒绝用户的选择。我们将接受任何拥有 GitHub 访问令牌的人；因此只需返回一个用户资料；我们使用 GitHub 传递给我们的资料创建它。在验证函数内部，我们实例化一个`GitHubRepo`并调用`updateTokens`，这将更新它们的访问令牌，以便用于我们的 Redis 缓存填充。

我们的应用程序将支持用户会话，因此我们在`passport`模块中添加两个函数，包括`serializeUser`和`deserializeUser`，这些函数将 GitHub 用户资料序列化和反序列化到用户会话中：

```js
var async = require('async')
, GitHubRepo = require('../github')
, config = require('../configuration');

function GitHubAuth() {
  this.passport = require('passport')
  var GitHubStrategy = require('passport-github').Strategy;

  this.passport.use(new GitHubStrategy({
      clientID     : config.get('auth:clientId'),
      clientSecret : config.get('auth:clientSecret'),
      callbackURL  : config.get('auth:callback')
  },
  function(accessToken, refreshToken, profile, done) {

    var user = {
      id : profile.username,
      displayName : profile.displayName,
      token : accessToken
    };

    var git = new GitHubRepo(user.token, user.id);

    git.updateTokens(function(){
      process.nextTick(function () {
        return done(null, user);
      });
    });
  };
  ));

  this.passport.serializeUser(function(user, done) {
    done(null, user);
  });

  this.passport.deserializeUser(function(user, done) {
    done(null, user);
  });
};

module.exports = new GitHubAuth();
```

让我们在`GitHubRepo`中添加一个`updateTokens`函数，该函数获取所有用户的工程并通过`async.each`遍历每个工程来更新其令牌：

```js
GitHubRepo.prototype.updateTokens = function(done) {
  var query = { "user" :  this.user };

  Project.find(query, function(error, projects) {
    if (error) return done();
    if (projects == null) done();

    async.each(projects, function(project, callback) {
      project.token = this.token;

      project.save(function(error, p) {
        callback();
      });
    }
    , function(error) {
      done();
    });
  });
};
```

让我们在配置文件`./config/*.json`中添加配置，以支持 Express 会话：

```js
  "session": {
    "secret": "th1$1$a$ecret"
    , "maxAge": null
    , "secure": true
    , "httpOnly": true
  }
```

让我们将 GitHub 策略连接到我们的 Express 服务器：`./lib/express/index.js`。我们做的第一个更改是包含我们新的 GitHub `authentication`策略：

```js
var gitHubAuth = require('../github/authentication')
```

我们创建一个`cookieParser`中间件，并将其包含在`bodyParser`中间件之前，这将解析 cookie 头字段并填充`req.cookies`。我们传递一个`secret`；这是一个用于创建签名 cookie 的字符串，以便检测修改过的 cookie：

```js
  var cookieParser = express.
    cookieParser(config.get('session:secret'));
  app.use(cookieParser);
```

应用程序将需要持久登录会话，因此我们将`connect` `session`中间件包含在我们的 Express 服务器中，以提供会话支持。我们将使用`sessionStore`，这是一个内存中的会话存储。我们传递一个`secret`和一个用于 cookie 的`maxAge`值（一个 null 值将在关闭浏览器时使会话过期），`httpOnly`（不允许客户端 JavaScript 访问 cookie；XSS 攻击），以及`secure`（仅通过 HTTPS 发送 cookie）：

```js
app.use(express.bodyParser());
var sessionStore = new express.session.MemoryStore();
app.use(express.session({ store: sessionStore,
  secret: config.get('session:secret'),
  cookie: { secure: config.get('session:secure'),
  httpOnly: config.get('session:httpOnly'),
  maxAge: config.get('session:maxAge') }}));
```

Passport 模块要求我们调用 `passport.initialize()` 以初始化 `passport`，并且为了提供会话支持，我们还必须调用 `passport.session()` 中间件；我们将两者都添加到我们的 Express 服务器中：

```js
app.use(gitHubAuth.passport.initialize());
app.use(gitHubAuth.passport.session());
```

我们现在定义我们的 Express 服务器上的第一个两个路由；两者都使用 GitHub 策略。第一个路由是登录路由 `/auth/github`；访问此路由将重定向您到 GitHub 并尝试进行身份验证。如果您未登录到 GitHub，您将被要求登录。如果您是第一次这样做，您将收到提示。您将被询问是否希望授予 Vision 访问权限。第二个路由是 GitHub 在身份验证完成后将回调的路由：

```js
app.get('/auth/github',gitHubAuth.passport.authenticate('github'),routes.auth.login);

app.get('/auth/github/callback',gitHubAuth.passport.authenticate('github',{ failureRedirect: '/' }), routes.auth.callback);
```

我们已经使用 GitHub passport 策略配置了我们的 Express 服务器。现在让我们向 `./lib/routes/auth.js` 中的路由添加两个缺失的路由：一个用于登录，另一个用于回调，正如之前所描述的：

```js
exports.callback = function(req, res) {
  logger.info('Request.' + req.url);
  res.redirect('/');
};

exports.login = function(req, res){
  logger.info('Request.' + req.url);
};
```

为了模拟我们的项目表单包含一个 `user` 和 `token` 的主体，我们将添加一个中间件，该中间件简单地为此数据添加到已认证用户的表单中。我们可以通过使用 `app.all` 简单地添加 `projectForm.addToken` 中间件到所有我们的路由中，这将应用此中间件到所有后续的路由。

让我们对我们的 Express 服务器进行进一步更改：`./lib/express/index.js`，并通过移除涉及它的所有 require 语句和使用 `require-directory` 与 `./lib/middleware/index.js` 文件来清理我们的中间件，就像我们对路由所做的那样。现在我们可以在所有需要身份验证的路由之上添加此 `projectForm`：

```js
   , middleware = require('../middleware')

app.all('*', middleware.projectForm.addToken);
.. all routes below
```

让我们在 `./lib/middleware/projectForm.js` 中创建 `projectForm.addToken` 中间件。`AddToken` 中间件检查请求是否通过 `req.isAuthenticated` 进行了身份验证；我们将 `user` 和 `token` 添加到请求中：

```js
exports.addToken = function(req, res, next){
  if (req.isAuthenticated()) {
    req.body.user = req.session.passport.user.id;
    req.body.token = req.session.passport.user.token;
    req.user = req.session.passport.user;
  };

  next();
}
```

现在我们已经设置了身份验证，让我们从 `./lib/routes/home.js` 中移除硬编码的用户：

```js
exports.index = function(req, res){
  var model = {
    title: 'vision.',
    description: 'a project based dashboard for github',
    author: 'airasoul',
 user: req.isAuthenticated() ? req.user.displayName : ''
  };

  res.render('index', model);
};
```

现在我们点击页眉中的 GitHub 标志，我们将被重定向到 GitHub，它将要求您登录。一旦您登录到 GitHub，您必须授权我们的 Vision 应用程序；然而，未来的登录尝试将不需要您授权 Vision。

让我们使用 Zombie.js 完成我们的 Cucumber 登录步骤。`./features/step_definitions/authentication/authenticate.js`。首先，我们包含 zombie 并定义一个 `steps` 函数。然后，我们将 `silent` 和 `debug` 设置为启用 Zombie.js 调试输出。我们定义 `Given = When = Then` 作为 Cucumber 步骤，并添加一个 `Before` 步骤，该步骤在每个测试之前运行。从这里我们实例化一个 zombie `Browser`：

```js
var Browser = require('zombie')
, assert = require('assert')
S = require('string')
config = require('../../../lib/configuration');

var steps = function() {
  var silent = false;
  var debug = false;
  var Given = When = Then = this.defineStep;
  var browser = null;
  var me = this;

   this.Before(function(callback) {
     browser = new Browser();
     browser.setMaxListeners(20);
     setTimeout(callback(), 5000);
   });
};

module.exports = steps;
```

步骤 `I have a GitHub Account` 使用 zombie 浏览器访问 GitHub 登录页面，并等待页面加载并填写登录详细信息；然后我们点击登录按钮：

```js
this.Given(/^I have a GitHub Account$/, function(callback) {browser.visit('https://github.com/login',{silent: silent, debug: debug});

    browser.wait(function(){
      browser
        .fill('login', '#LOGIN#')
        .fill('password', '#PASSWORD#')
        .pressButton('Sign in', function() {
          callback();
    });
  });
});
```

步骤`我点击 GitHub 认证按钮`使用 Zombie 浏览器访问 GitHub 登录页面，等待页面加载并填写登录详细信息；然后我们点击登录按钮：

```js
this.When(/^I click the GitHub authentication button$/, function(callback) {
    browser.visit(config.get('auth:homepage'),
    {silent: silent, debug: debug});

    browser.wait(function(){
      browser
        .clickLink('#login', function() {
          callback();
        });
      });
});
```

步骤`我应该已登录`使用 Zombie 浏览器访问 GitHub 登录页面，等待页面加载并填写登录详细信息；然后我们点击登录按钮：

```js
this.Then(/^I should be logged in$/, function(callback) {
  assert.ok(browser.success);
  callback();
});
```

步骤`我应看到我的名字和一个登出链接`使用 Zombie 浏览器访问 GitHub 登录页面，等待页面加载并填写登录详细信息；然后我们点击登录按钮：

```js
this.Then(/^I should see my name and a logout link$/, function(callback) {
  assert.equal(browser.text('#welcome'),'welcome Andrew Keig, click here to sign out');
      callback();
});
```

## 场景：用户成功登出

```js
  Given I am logged in to Vision
  When I click the logout button
  Then I should see the GitHub login button
```

让我们在 Express 服务器中添加一个登出路由：`./lib/express/index.js`：

```js
app.get('/logout', routes.auth.logout);
```

现在将路由添加到我们的路由中：`./lib/routes/auth.js`：

```js
exports.logout = function(req, res){
  logger.info('Request.' + req.url);
  req.logout();
  res.redirect('/');
};
```

让我们在`./features/step_definitions/authentication/authenticate.js`中使用 Zombie.js 完成我们的登出步骤。

步骤`我已登录到 Vision`使用 Zombie 浏览器访问 Vision 主页，等待页面加载，并点击登录链接：

```js
this.Given(/^I am logged in to Vision$/, function(callback) {
  browser.visit(config.get('auth:homepage'),{silent: silent, debug: debug});

  browser.wait(function(){
    browser
    .clickLink('#login', function() {
      callback();
    });
  });
});
```

步骤`我点击登出按钮`使用 Zombie 浏览器访问 Vision 主页，等待页面加载，并点击登出链接：

```js
this.When(/^I click the logout button$/, function(callback) {
  browser.visit(config.get('auth:homepage'),{silent: silent, debug: debug});

  browser.wait(function(){
    browser
    .clickLink('#logout', function(err) {
      callback();
    });
  });
});
```

步骤`我应该看到 GitHub 登录按钮`检查浏览器响应是否返回`成功`，然后检查 GitHub 登录链接是否可访问：

```js
this.Then(/^I should see the GitHub login button$/, function(callback) {
  assert.ok(browser.success);
  var containsLogin = S(browser.html('#login')).contains('vision/github.png')
    assert.equal(true, containsLogin);
    callback();
  });
```

# 使用 HTTPS 保护我们的站点

为了使我们的站点安全，我们将整个应用程序运行在 HTTPS 下。我们需要两个文件：一个 PEM 编码的 SSL 证书`./lib/secure/cert.pem`和一个私钥`./lib/secure/key.pem`。为了创建 SSL 证书，我们首先需要生成一个私钥和证书签名请求（CSR）。出于开发目的，我们将创建一个自签名证书。运行以下命令：

```js
cd ../vision/lib/secure
openssl req -newkey rsa:2048 -new -nodes -x509 -days 3650 -keyout key.pem -out cert.pem

```

在运行第二个命令后，你将进入一个交互式提示，以生成 2048 位的 RSA 私钥和证书签名请求（CSR）。你需要输入包括地址详情、通用名称或域名、公司详情和电子邮件地址在内的各种信息。

让我们添加一个模块，`./lib/express/server.js`，它将基于我们刚刚创建的`key`/`cert`创建一个 HTTP 服务器。我们导入`https`模块，从磁盘读取`key`和`cert`文件，并将它们添加到选项对象中。然后使用`https`模块，我们创建一个服务器，传入这些选项：

```js
var fs = require('fs')
, https = require('https');

function Server(app){
  var httpsOptions = {
    key: fs.readFileSync('./lib/secure/key.pem'),cert: fs.readFileSync('./lib/secure/cert.pem')
  };

  return https.createServer(httpsOptions,app).listen(app.get('port'));
}

module.exports = Server;
```

让我们在 Express 服务器`./lib/express/index.js`中使用`server`；删除创建我们的 HTTP 服务器的行：

```js
var httpServer = http.createServer(app).listen(app.get('port'));
```

用对新的 HTTPS 服务器的调用替换它：

```js
var server = require('./server')(app);
```

现在我们需要将所有对`http://127.0.0.1:3000`；端口 3000 的引用替换为`https://127.0.0.1:8443`；端口 8443。我们的配置文件包含两个引用：

```js
"auth": {
    "homepage": "https://127.0.0.1:8443"
  , "callback": "https://127.0.0.1:8443/auth/github/callback"
  , "clientId": "5bb691b4ebb5417f4ab9"
  , "clientSecret": "15310740929666983d52808dda32417d733791d0"
  },
```

在我们的 `backbone.js` 脚本 `./public/components/vision.js` 中，我们有一个进一步的参考。当我们连接到我们的 Socket.IO 服务器时，我们传递一个 URL `127.0.0.1:3000`。在这里，我们进行了另一个重要的更改；我们连接到 Socket.IO 时传递一个选项对象，设置 `secure: true, port: '8443'`：

```js
Vision.Application = function(){        
  this.start = function(){
 var socketio = io.connect('/', {secure: true, port: '8443'});
    var router = new Vision.Router(socketio);
    Backbone.history.start();
    router.navigate('index', true);
  }
};
```

# 与 Socket.IO 共享 Express 会话

现在我们已经设置了会话支持，我们可以通过 Socket.IO 共享会话，这样我们就可以根据这些会话数据接受或拒绝连接。Express 和 Socket.IO 使用握手机制来完成此操作。当客户端连接到服务器时，握手被启动，它包括在 Socket.IO 上执行授权函数。在这里，检查与握手请求关联的 cookie，如果无效则拒绝。让我们安装 `session.socket.io`；这是一个封装了此过程的模块：

```js
npm install session.socket.io --save

```

首先，让我们更改我们的 Express 服务器 `./lib/express/index.js`，并将 `sessionStore` 和 `cookieParser` 传递给我们的 `SocketHandler` 模块：

```js
var socketHandler = new SocketHandler(httpServer, sessionStore, cookieParser);
```

`SocketHandler` 模块现在接受参数 `httpServer`、`sessionStore` 和 `cookieParser`。`SocketHandler` 现在将实例化一个 `SessionSockets` 模块，传递 `socketIo`、`sessionStore` 模块和 `cookieParser`。我们将 `connection` 事件更改为监听 `SessionSockets` 模块而不是 `socket.Io` 模块，这样我们就可以访问 `session`。现在，在 `subscribe` 事件内部，我们可以检查以确保 `session.passport.user` 是有效的。我们调用 `session.touch`，它更新会话的 `maxAge` 和 `lastAccess` 属性：

```js
function SocketHandler(httpServer, sessionStore, cookieParser) {
  var socketIo = new Socket(httpServer)
  var sessionSockets = new SessionSockets(socketIo, sessionStore, cookieParser);

  sessionSockets.on('connection', function(err, socket, session) {
    subscriber.subscribe("issues");
    subscriber.subscribe("commits");

    subscriber.client.on("message", function (channel, message) {
      socket.broadcast.to(message.projectId)
      .emit(channel, JSON.parse(message));
    });

    socket.on('subscribe', function (data) {
      var user = session ? session.passport.user : null;
      if (!user) return;
      socket.join(data.channel);
      session.touch();
    });
  });

  sessionSockets.on('error', function() {
    logger.error(arguments);
  });
};

module.exports = SocketHandler;
```

# 跨站请求伪造

**跨站请求伪造** (**CSRF**) 是一种攻击，它欺骗受害者在一个他们已经认证的 Web 应用程序上执行恶意操作。Connect/Express 随包装提供了跨站请求伪造保护中间件。此中间件允许我们确保对更改状态的请求来自有效源。CSRF 中间件创建一个存储在请求会话中的令牌作为 `_csrf`。然后，我们的 Express 服务器请求需要通过头部字段 `X-CSRF-Token` 传递令牌。

让我们创建一个安全模块 `./lib/security/index.js`，该模块将 `csrf` 中间件添加到我们的应用程序中。我们定义一个函数，`Security`，它接受一个 Express `app` 作为参数，并在 `TEST` 或 `COVERAGE` 模式下移除中间件。

```js
var express = require('express');

function Security(app) {
  if (process.env['NODE_ENV'] === "TEST"  ||process.env['NODE_ENV'] === "COVERAGE") return;

  app.use(express.csrf());
};

module.exports = Security;
```

让我们更改我们的 Express 服务器 `./lib/express/index.js`。`crsf` 中间件需要会话支持，所以我们添加以下行在 `session` 和 `passport` 中间件下方：

```js
require('../security')(app);
```

由于我们使用的是 `backbone.js`，它底层使用 jQuery 来发送 AJAX 请求，因此我们需要更改我们的 backbone 代码 `./public/components/vision/vision.js`。我们现在将覆盖 `Backbone.sync` 函数，以便所有通过它的请求都在头部传递 `X-CSRF-Token`。`X-CSRF-Token` 从主页面中的 `meta` 标签中提取：

```js
Backbone.sync = (function(original) {
  return function(method, model, options) {
    options.beforeSend = function(xhr) {
      var token = $("meta[name='csrf-token']").attr('content');
      xhr.setRequestHeader('X-CSRF-Token', token);
    };
    original(method, model, options);
  };
})(Backbone.sync);
```

现在，我们需要通过主页面路由将 `X-CSRF-Token` 传递给我们的主页面。令牌存储在请求会话中作为 `_csrf`，在以下代码中我们将令牌添加到我们的视图对象的 `csrftoken` 中：

```js
exports.index = function(req, res){
  var model = {
    title: 'vision.',
    description: 'a project based dashboard for github',
    author: 'airasoul',
    user: req.isAuthenticated() ? req.user.displayName : '',
 csrftoken: req.session._csrf
  };

  res.render('index', model);
};
```

`csrftoken` 在我们的主页面中以名为 `csrf-token` 的 `meta` 标签中渲染；骨干同步方法将从此 `meta` 标签中获取它：

```js
<meta name="csrf-token" content="{{csrftoken}}">
```

# 通过 HTTP 头和 helmet 提高安全性

Helmet 是一系列中间件，用于为 Express 实现各种安全头；有关 helmet 的更多信息，请访问 [`npmjs.org/package/helmet`](https://npmjs.org/package/helmet)。

Helmet 支持以下功能：

+   csp (内容安全策略)

+   HSTS (HTTP Strict Transport Security)

+   xframe (X-FRAME-OPTIONS)

+   iexss (X-XSS-PROTECTION for IE8+)

+   contentTypeOptions (X-Content-Type-Options nosniff)

+   cacheControl (Cache-Control no-store, no-cache)

让我们扩展我们的安全模块 `./lib/security/index.js`，并为之前的问题添加 helmet 安全：

```js
var express = require('express')
, helmet = require('helmet');

function Security(app) {
  if (process.env['NODE_ENV'] === "TEST"  ||
    process.env['NODE_ENV'] === "COVERAGE") return;

 app.use(helmet.xframe());
 app.use(helmet.hsts());
 app.use(helmet.iexss());
 app.use(helmet.contentTypeOptions());
 app.use(helmet.cacheControl());
  app.use(express.csrf());
};

module.exports = Security;
```

# 摘要

默认情况下，Express 使用内存中的会话。在下一章中，我们将我们的会话移动到 Redis。我们还将配置 Socket.IO 以使用 Redis，并探索一些其他有趣的扩展 Express 的方法。
