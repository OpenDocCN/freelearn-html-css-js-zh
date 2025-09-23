# 第一章. 构建一个基本的 Express 网站

Express 是 Node.js 的 Web 开发框架。Node.js 是一个开源的、跨平台的运行时环境，用于服务器端和网络应用程序。它使用 Google Chrome 的 JavaScript 引擎 V8 来执行代码。Node.js 是单线程和事件驱动的。它使用非阻塞 I/O 来榨取 CPU 的每一分处理能力。Express 建立在 Node.js 之上，提供了开发健壮 Web 应用程序所需的所有工具。

此外，通过使用 Express，你可以访问大量的开源软件来帮助解决开发中的常见痛点。该框架是中立的，这意味着它不会在实现或接口方面引导你走向一个方向或另一个方向。因为它是不偏不倚的，开发者有更多的控制权，可以使用该框架完成几乎任何任务；然而，Express 提供的功能很容易被滥用。在这本书中，你将通过探索以下不同风格的应用程序来学习如何正确使用该框架：

+   为静态网站设置 Express

+   本地用户身份验证

+   OAuth 与护照

+   个人资料页面

+   测试

# 为静态网站设置 Express

为了让我们熟悉环境，我们首先会讲解如何响应基本的 HTTP 请求。在这个例子中，我们将处理几个`GET`请求，首先以纯文本形式响应，然后以静态 HTML 响应。然而，在我们开始之前，你必须安装两个基本工具：node 和 npm，即 node 包管理器。

### 注意

导航到[`nodejs.org/download/`](https://nodejs.org/download/)来安装 node 和 npm。

## 在 Express 中使用 Hello, World

对于那些不熟悉 Express 的人来说，我们将从一个基本的例子开始——Hello World！我们将从一个空目录开始。与任何 Node.js 项目一样，我们将运行以下代码来生成我们的`package.json`文件，该文件跟踪有关项目元数据的信息，例如依赖项、脚本、许可证，甚至代码托管的位置：

```js
$ npm init

```

`package.json`文件跟踪我们所有的依赖项，这样我们就不需要处理版本问题，不需要将依赖项包含在我们的代码中，并且可以无畏地部署。你将收到几个问题提示。除了入口点外，所有默认值都可以选择，你应该将其设置为`server.js`。

有许多生成器可以帮助你生成新的 Express 应用程序，但这次我们将自己创建框架。让我们安装 Express。要安装一个模块，我们使用`npm`来安装包。我们使用`--save`标志来告诉 npm 将依赖项添加到我们的`package.json`文件中；这样，我们就不需要将依赖项提交到源代码控制中。我们可以根据`package.json`文件的内容安装它们（npm 使这变得很容易）：

```js
$ npm install --save express 

```

在这本书中，我们将使用 Express v4.4.0。

### 注意

警告：Express v4.x 与其之前的版本不向后兼容。

你可以创建一个新的文件`server.js`，如下所示：

```js
var express = require('express');
var app = express();

app.get('/', function(req, res, next) {
 res.send('Hello, World!');
});

app.listen(3000);
console.log('Express started on port 3000');
```

此文件是应用程序的入口点。在这里，我们生成应用程序，注册路由，并最终在端口 `3000` 上监听传入的请求。`require('express')` 方法返回一个应用程序生成器。

我们可以不断地创建尽可能多的应用程序；在这种情况下，我们只创建了一个，并将其分配给变量 `app`。接下来，我们注册一个 `GET` 路由，该路由监听服务器根目录上的 `GET` 请求，并在请求时向客户端发送字符串 `'Hello, World'`。Express 有所有 HTTP 动词的方法，所以我们也可以做 `app.post`、`app.put`、`app.delete` 或甚至 `app.all`，它响应所有 HTTP 动词。最后，我们启动应用程序监听端口 `3000`，然后记录到标准输出。

现在是时候启动我们的服务器并确保一切按预期工作。

```js
$ node server.js

```

我们可以通过在浏览器中导航到 `http://localhost:3000` 或在终端中执行 `curl -v localhost:3000` 来验证一切是否正常工作。

## Jade 模板

现在，我们将把发送给客户端的 HTML 提取到一个单独的模板中。毕竟，仅仅通过使用 `res.send` 来渲染完整的 HTML 页面会相当困难。为了实现这一点，我们将使用与 `Express` 配合频繁使用的模板语言——Jade。你可以使用许多模板语言与 Express 一起使用。我们选择 Jade，因为它极大地简化了 HTML 的编写，并且是由 Express 框架的同一开发者创建的。

```js
$ npm install --save jade

```

安装 Jade 后，我们必须将以下代码添加到 `server.js` 中：

```js
app.set('view engine', 'jade');
app.set('views', __dirname + '/views');

app.get('/', function(req, res, next) {
res.render('index');
});
```

之前的代码为 Express 设置了默认视图引擎——有点像告诉 Express，在未来，除非有其他指定，否则应该假设模板位于 Jade 模板语言中。调用 `app.set` 为 Express 内部设置一个键值对。你可以将这种应用程序视为宽配置。我们可以随时调用 `app.get`（视图引擎）来检索我们设置的值。

我们还指定了 Express 应该查找以找到视图文件的文件夹。这意味着我们应该在我们的应用程序中创建一个 `views` 目录，并向其中添加一个文件，`index.jade`。或者，如果你想包含许多不同的模板类型，你可以执行以下操作：

```js
app.engine('jade', require('jade').__express);
app.engine('html', require('ejs').__express);
app.get('/html', function(req, res, next) {
res.render('index.html');
});

app.get(/'jade, function(req, res, next) {
res.render('index.jade');
});
```

在这里，我们根据我们想要渲染的模板的扩展名设置自定义模板渲染。我们使用 Jade 渲染器为 `.jade` 扩展名，使用 `ejs` 渲染器为 `.html` 扩展名，并通过不同的路由公开我们的索引文件。如果你选择了一种模板选项，后来又想以渐进的方式切换到新的选项，这很有用。你可以参考最基本模板的源代码。

# 本地用户身份验证

大多数应用程序都需要用户账户。有些应用程序只允许通过第三方进行认证，但并非所有用户都希望因为隐私原因通过第三方进行认证，因此包含本地选项很重要。在这里，我们将讨论在 Express 应用程序中实现本地用户认证的最佳实践。我们将使用 MongoDB 来存储我们的用户，并使用 Mongoose 作为 ODM（对象文档映射器）。然后，我们将利用 passport 来简化会话处理并提供统一的认证视图。

### 小贴士

**下载示例代码**

您可以从您在 [`www.packtpub.com`](http://www.packtpub.com) 的账户下载示例代码文件，以获取您购买的所有 Packt Publishing 书籍。如果您在其他地方购买了这本书，您可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 并注册，以便将文件直接通过电子邮件发送给您。

## 用户对象建模

我们将利用 passportjs 来处理用户认证。Passport 集中了所有的认证逻辑，并提供方便的方式在本地进行认证，同时支持第三方认证，例如 Twitter、Google、Github 等等。首先，按照以下步骤安装 passport 和本地认证策略：

```js
$ npm install --save passport-local

```

在我们的第一次尝试中，我们将实现本地认证策略，这意味着用户将能够本地注册账户。我们首先使用 Mongoose 定义一个用户模型。Mongoose 提供了一种定义我们想要存储在 MongoDB 中的对象模式的方法，并提供了一种方便的方式来在数据库中的存储记录和内存表示之间进行映射。

Mongoose 还提供了方便的语法来执行许多 MongoDB 查询，并在模型上执行 CRUD 操作。我们的用户模型目前只包含电子邮件、密码和时间戳。在开始之前，我们需要安装 Mongoose：

```js
$ npm install --save mongoose bcrypt validator

```

现在我们将在 `models/user.js` 文件中定义我们的用户模式，如下所示：

```js
Var mongoose = require('mongoose');

var userSchema = new mongoose.Schema({
 email: {
   type: String,
   required: true,
   unique: true
 },
 password: {
   type: String,
   required: true
 },
 created_at: {
   type: Date,
   default: Date.now
 }
});

userSchema.pre('save', function(next) {
 if (!this.isModified('password')) {
   return next();
 }
 this.password = User.encryptPassword(this.password);
 next();
});
```

在这里，我们创建一个模式来描述我们的用户。Mongoose 提供了方便的方式来描述所需的唯一字段以及每个属性应持有的数据类型。Mongoose 在幕后执行所有必要的验证。对于我们的第一个样板应用程序，我们不需要许多用户字段——电子邮件、密码和时间戳足以开始。

我们还使用 Mongoose 中间件来重新散列用户的密码，如果他们决定更改密码。Mongoose 提供了几个钩子来运行用户定义的回调。在我们的示例中，我们定义了一个在 Mongoose 保存模型之前被调用的回调。这样，每次用户被保存时，我们都会检查他们的密码是否已更改。

没有这个中间件，就有可能以明文形式存储用户的密码，这不仅是一个安全漏洞，还会破坏认证。Mongoose 支持两种中间件类型——串行和并行。并行中间件可以运行异步函数，并得到一个额外的回调函数来调用；你将在本书的后面部分了解更多关于 Mongoose 中间件的内容。

现在，我们想要添加验证以确保我们的数据是正确的。我们将使用 `validator` 库来完成这项任务，如下所示：

```js
Var validator = require('validator');

User.schema.path('email').validate(function(email) {
 return validator.isEmail(email);
});

User.schema.path('password').validate(function(password) {
 return validator.isLength(password, 6);
});

var User = mongoose.model('User', userSchema);
module.exports = User;
```

我们使用名为 `validator` 的库添加了对电子邮件和密码长度的验证，该库为不同类型的字段提供了许多方便的验证器。Validator 提供基于长度、URL、整数、大写的验证；基本上，任何你想验证的内容（别忘了验证所有用户输入！）

我们还添加了一系列关于注册、认证以及加密密码的辅助函数，你可以在 `models/user.js` 文件中找到它们。我们将这些添加到用户模型中，以帮助封装我们想要使用用户抽象的各种交互。

### 注意

想了解更多关于 Mongoose 的信息，请参阅 [`mongoosejs.com/`](http://mongoosejs.com/)。有关 passportjs 的更多信息，请访问 [`passportjs.org/`](http://passportjs.org/)。

这概述了名为 MVC 的设计模式的开始——模型、视图、控制器。基本思想是将不同的关注点封装在不同的对象中：模型代码了解数据库、存储和查询；控制器代码了解路由和请求/响应；视图代码知道为用户渲染什么。

## 介绍 Express 中间件

Passport 是一种可以与 Express 应用程序一起使用的认证中间件。在深入研究 Passport 之前，我们应该回顾一下 Express 中间件。Express 是一个 connect 框架，这意味着它使用 connect 中间件。内部连接有一个处理请求的函数堆栈。

当请求到来时，堆栈中的第一个函数会接收到请求和响应对象以及 `next()` 函数。当调用 `next()` 函数时，它将委托给中间件堆栈中的下一个函数。此外，你可以指定中间件的路径，这样它就只为特定的路径调用。

Express 允许你使用 `app.use()` 函数向应用程序添加中间件。实际上，我们之前编写的 HTTP 处理程序是一种特殊的中间件。内部，Express 为路由器有一个中间件级别，它委托给适当的处理程序。

中间件对于日志记录、提供静态文件、错误处理等非常有用。实际上，Passport 利用中间件进行认证。在发生任何其他事情之前，Passport 会查找请求中的 cookie，找到元数据，然后从数据库中加载用户，将其添加到 req 和 user 中，然后继续沿着中间件堆栈向下。

## 设置 passport

在我们可以充分利用 passport 之前，我们需要告诉它如何完成一些重要的事情。首先，我们需要指导 passport 如何将用户序列化到会话中。然后，我们需要从会话信息中反序列化用户。最后，我们需要告诉 passport 如何判断给定的电子邮件/密码组合是否代表一个有效的用户，如下所示：

```js
// passport.js
var passport = require('passport');
var LocalStrategy = require('passport-local').Strategy;
var User = require('mongoose').model('User');

passport.serializeUser(function(user, done) {
 done(null, user.id);
});

passport.deserializeUser(function(id, done) {
User.findById(id, done);
});
```

在这里，我们告诉 passport 当我们序列化用户时，我们只需要该用户的 `id`。然后，当我们想要从会话数据中反序列化用户时，我们只需通过他们的 ID 查找用户！这用于 passport 的中间件，在请求完成后，我们取 `req.user` 并将他们的 ID 序列化到我们的持久会话中。当我们第一次收到请求时，我们取存储在我们的会话中的 ID，从数据库中检索记录，并用 `user` 属性填充请求对象。所有这些功能都由 passport 透明地提供，只要我们提供以下两个函数的定义：

```js
function authFail(done) {
 done(null, false, { message: 'incorrect email/password combination' });
}

passport.use(new LocalStrategy(function(email, password, done) {
  User.findOne({
    email: email
  }, function(err, user) {
    if (err) return done(err);
    if (!user) {
      return authFail(done);
    }
    if (!user.validPassword(password)) {
      return authFail(done);
    }
    return done(null, user);
  });
}));
```

我们告诉 passport 如何在本地验证用户。我们创建一个新的 `LocalStrategy()` 函数，当给定电子邮件和密码时，将尝试通过电子邮件查找用户。我们可以这样做，因为我们要求电子邮件字段必须是唯一的，所以应该只有一个用户。如果没有用户，我们返回一个错误。如果有用户，但提供了无效的密码，我们仍然返回一个错误。如果有用户并且他们提供了正确的密码，那么我们通过调用 `done` 回调来告诉 passport 验证请求是成功的，从而告诉 passport 验证请求成功。

为了使用护照，我们需要添加我们之前提到的中间件。实际上，我们需要添加几种不同的中间件。Express 中间件的好处在于它鼓励开发者编写小型、专注的模块，这样你就可以引入你想要的功能，排除你不需要的功能。

```js
// server.js
var mongoose = require('mongoose');
var User = require('./models/user');
var passport = require('./passport');

mongoose.connect('mongodb://localhost/chapter01', function(err) {
 if (err) throw err;
});
…
app.use(require('cookie-parser')('my secret string'));
app.use(require('express-session')({ secret: "my other secret string" }));
app.use(require('body-parser')());
app.use(passport.initialize());
app.use(passport.session());
```

为了使用 passport，我们必须为我们的服务器启用一些功能。首先，我们需要启用 cookies 和会话支持。为了启用会话支持，我们添加了一个 cookie 解析器。这个中间件将 cookie 对象解析为 `req.cookies`。会话中间件允许我们修改 `req.session` 并使这些数据在请求之间持久化。默认情况下，它使用 cookies，但它有多种会话存储方式，你可以进行配置。然后，我们必须添加 body-parsing 中间件，它将 HTTP 请求的正文解析为一个 JavaScript 对象 `req.body`。

在我们的用例中，我们需要这个中间件从 `POST` 请求中提取电子邮件和密码字段。最后，我们添加了 passport 中间件和会话支持。

## 注册用户

现在，我们添加了注册路由，包括一个带有基本表单的后端逻辑来创建用户。首先，我们将创建一个用户控制器。到目前为止，我们一直在将路由扔到 `server.js` 文件中，但这通常是一个不好的做法。我们想要做的是为每种我们想要的路线都有单独的控制器。我们已经看到了 MVC 的模型部分。现在是时候看看控制器了。我们的用户控制器将包含所有操作用户模型的路线。让我们在新的目录中创建一个新文件，`controllers/user.js`：

```js
// controllers/user.js
var User = require('mongoose').model('User');

module.exports.showRegistrationForm = function(req, res, next) {
  res.render('register');
};

module.exports.createUser = function(req, res, next) {
  User.register(req.body.email, req.body.password, function(err, user) {
    if (err) return next(err);
    req.login(user, function(err) {
      if (err) return next(err);
      res.redirect('/');
    });
  });  
};
```

### 注意

注意，`User` 模型负责验证和注册逻辑；我们只需提供回调。这样做有助于整合错误处理，并且通常使注册逻辑更容易理解。如果注册成功，我们调用 passport 添加的 `req.login` 函数，为该用户创建一个新的会话，并且该用户将在后续请求中作为 `req.user` 可用。

最后，我们注册了路由。在这个阶段，我们还提取了之前添加到 `server.js` 的路由到它们自己的文件。让我们创建一个名为 `routes.js` 的新文件，如下所示：

```js
// routes.js
app.get('/users/register', userRoutes.showRegistrationForm);
app.post('/users/register', userRoutes.createUser);
```

现在，我们有一个专门用于将控制器处理程序与用户可以访问的实际路径关联的文件。这通常是一个好的做法，因为现在我们有一个地方可以访问并查看我们定义的所有路由。这也帮助清理了我们的 `server.js` 文件，该文件应该专门用于服务器配置。

### 注意

有关详细信息以及使用的注册模板，请参阅前面的代码。

## 用户认证

我们已经完成了认证用户所需的大部分工作（或者说，passport 已经完成了）。实际上，我们只需要设置认证路由和一个允许用户输入其凭据的表单。首先，我们将在用户控制器中添加处理程序：

```js
// controllers/user.js
module.exports.showLoginForm = function(req, res, next) {
  res.render('login');
};

module.exports.createSession = passport.authenticate('local', {
  successRedirect: '/',
  failureRedirect: '/login'
});
```

让我们分解一下我们的登录 post 中的发生的事情。我们创建了一个处理程序，它是调用 `passport.authenticate('local', …)` 的结果。这告诉 passport，该处理程序使用本地认证策略。因此，当有人访问该路由时，passport 将将任务委托给我们的 LocalStrategy。如果他们提供了有效的电子邮件/密码组合，我们的 LocalStrategy 将给 passport 提供现在认证的用户，passport 将将用户重定向到服务器根目录。如果电子邮件/密码组合不成功，passport 将将用户重定向到 `/login`，以便他们可以再次尝试。

然后，我们将这些回调绑定到 `routes.js` 中的路由：

```js
app.get('/users/login', userRoutes.showLoginForm);
app.post('/users/login', userRoutes.createSession);
```

在这个阶段，我们应该能够使用相同的凭据注册账户并登录。（有关我们目前所在位置的详细信息，请参阅标签 0.2）。

# 使用 passport 进行 OAuth

现在，我们将添加对使用 Twitter、Google 和 GitHub 登录我们应用程序的支持。如果用户不想为您的应用程序注册单独的账户，这个功能非常有用。对于这些用户，通过这些提供者允许 OAuth 将提高转化率，并且通常会让用户的注册过程更加简单。

## 将 OAuth 添加到用户模型

在添加 OAuth 之前，我们需要在我们的用户模型上跟踪几个额外的属性。我们跟踪这些属性以确保如果有信息可以查找用户账户，我们不会允许重复的账户，并允许用户通过以下代码链接多个第三方账户：

```js
var userSchema = new mongoose.Schema({
  email: {
    type: String,
    required: true,
    unique: true
  },
  password: {
    type: String,
  },
  created_at: {
    type: Date,
    default: Date.now
  },
  twitter: String,
  google: String,
  github: String,
  profile: {
    name: { type: String, default: '' },
    gender: { type: String, default: '' },
    location: { type: String, default: '' },
    website: { type: String, default: '' },
    picture: { type: String, default: '' }
  },
});
```

首先，我们为每个提供者添加一个属性，我们将存储提供者在授权时给我们提供的唯一标识符。接下来，我们将存储一个令牌数组，这样我们可以方便地访问与该账户链接的提供者列表；如果您想允许用户通过一个注册然后链接到其他账户进行病毒式营销或获取更多信息，这将非常有用。最后，我们跟踪一些提供者给我们提供的关于用户的统计数据，以便我们可以为用户提供更好的体验。

## 获取 API 令牌

现在，我们需要前往适当的第三方并注册我们的应用程序以接收应用程序密钥和秘密令牌。我们将把这些添加到我们的配置中。我们将为开发和生产目的使用不同的令牌（出于明显的原因！）。出于安全考虑，我们只会在最终的部署服务器上以环境变量的形式拥有我们的生产令牌，不会提交到版本控制中。

我会等待您导航到第三方网站，并将它们的令牌按照以下方式添加到您的配置中：

```js
  // config.js
  twitter: {
    consumerKey: process.env.TWITTER_KEY || 'VRE4lt1y0W3yWTpChzJHcAaVf',
    consumerSecret: process.env.TWITTER_SECRET  ||  'TOA4rNzv9Cn8IwrOi6MOmyV894hyaJks6393V6cyLdtmFfkWqe',
    callbackURL: '/auth/twitter/callback'
  },
  google: {
    clientID: process.env.GOOGLE_ID || '627474771522-uskkhdsevat3rn15kgrqt62bdft15cpu.apps.googleusercontent.com',
    clientSecret: process.env.GOOGLE_SECRET || 'FwVkn76DKx_0BBaIAmRb6mjB',
    callbackURL: '/auth/google/callback'
  },
  github: {
    clientID: process.env.GITHUB_ID || '81b233b3394179bfe2bc',
    clientSecret: process.env.GITHUB_SECRET || 'de0322c0aa32eafaa84440ca6877ac5be9db9ca6',
    callbackURL: '/auth/github/callback'
  }
```

### 小贴士

当然，您也不应该将开发密钥公开提交。请确保不要提交此文件或使用私有源代码控制。最好的办法是只让秘密在机器上短暂存在（通常作为环境变量）。您尤其不应该使用我这里提供的密钥！

## 第三方注册和登录

现在，我们需要安装和实现各种第三方注册策略。要安装第三方注册策略，请运行以下命令：

```js
npm install --save passport-twitter passport-google-oAuth passport-github

```

这些大多数都非常相似，所以我只会展示`TwitterStrategy`，如下所示：

```js
passport.use(new TwitterStrategy(config.twitter, function(req, accessToken, tokenSecret, profile, done) {
  User.findOne({ twitter: profile.id }, function(err, existingUser) {
      if (existingUser) return done(null, existingUser);
      var user = new User();
      // Twitter will not provide an email address.  Period.
      // But a person's twitter username is guaranteed to be unique
      // so we can "fake" a twitter email address as follows:
      // username@twitter.mydomain.com
user.email = profile.username + "@twitter." + config.domain + ".com";
      user.twitter = profile.id;
      user.tokens.push({ kind: 'twitter', accessToken: accessToken, tokenSecret: tokenSecret });
      user.profile.name = profile.displayName;
      user.profile.location = profile._json.location;
      user.profile.picture = profile._json.profile_image_url;
      user.save(function(err) {
        done(err, user);
      });
    });
}));
```

在这里，我包括了一个示例，说明我们如何做这件事。首先，我们向 passport 传递一个新的 TwitterStrategy。TwitterStrategy 接受我们的 Twitter 密钥和回调信息，回调用于确保我们可以使用这些信息注册用户。如果用户已经注册，则不执行任何操作；否则，我们保存他们的信息，并将错误和/或成功保存的用户传递给回调。对于其他情况，请参阅源代码。

# 个人资料页面

现在是时候为我们的每个用户添加个人资料页面了。为了做到这一点，我们将进一步讨论 Express 路由以及如何将请求特定数据传递给 Jade 模板。在编写服务器时，通常希望捕获 URL 的一部分用于控制器；这可能是一个用户 ID、用户名，或者任何东西！我们将使用 Express 捕获 URL 部分的能力来获取请求个人资料页面的用户 ID。

## URL 参数

Express，就像任何好的 Web 框架一样，支持从 URL 部分提取数据。例如，你可以这样做：

```js
app.get('/users/:id', function(req, res, next) {
  console.log(req.params.id);
}
```

在前面的例子中，我们将打印出请求 URL 中`/users/`之后的内容。这提供了一种简单的方式来指定针对每个用户的路由，或者只在特定用户上下文中才有意义的路由，也就是说，只有当你指定了特定用户时，个人资料页面才有意义。我们将使用这种路由来实现我们的个人资料页面。目前，我们想要确保只有登录用户能看到自己的个人资料页面（我们可以在以后更改这个功能）：

```js
app.get('/users/:id', function(req, res, next) {
  if (!req.user || (req.user.id != req.params.id)) {
    return next('Not found');
  }
  res.render('users/profile', { user: req.user.toJSON() });
});
```

在这里，我们首先检查用户是否已登录，以及请求的用户 ID 是否与登录用户的 ID 相同。如果不是，则返回错误。如果是，则使用`req.user`作为数据渲染`users/profile.jade`模板。

## 个人资料模板

我们已经详细地讨论了模型和控制器，但我们的模板一直很平淡。最后，我们将展示如何编写一些基本的 Jade 模板。本节将作为对 Jade 模板语言的简要介绍，但并不试图全面介绍。个人资料模板的代码如下：

```js
html
  body
    h1
      =user.email
    h2
      =user.created_at
    - for (var prop in user.profile)
      if user.profile[prop]
        h4
          =prop + "=" + user.profile[prop]
```

显然，因为在控制器中我们传递了用户到视图中，我们可以访问变量`user`，它指的是登录用户！我们可以通过在前面加上`=`前缀来执行任意 JavaScript 以渲染到模板中。在这些块中，我们可以做任何我们通常会做的事情，包括字符串连接、方法调用等等。

类似地，我们可以通过在前面加上`-`前缀，就像我们处理`for`循环那样，包含不打算作为 HTML 编写的 JavaScript 代码。这个基本模板会打印出用户的电子邮件、`created_at`时间戳，以及他们个人资料中的所有属性（如果有的话）。

### 注意

要深入了解 Jade，请参阅[`jade-lang.com/reference/`](http://jade-lang.com/reference/)。

# 测试

测试对于任何应用程序都是必不可少的。我不会过多地讨论原因，而是假设你因为我跳过了前几节中的这个话题而对我感到愤怒。测试 Express 应用程序通常相对简单且痛苦较少。一般格式是我们创建模拟请求，然后对响应做出某些断言。

我们也可以为更复杂的逻辑实现更细粒度的单元测试，但到目前为止，我们做的几乎所有事情都足够直接，可以按路由进行测试。此外，在 API 级别进行测试可以更真实地反映真实客户如何与您的网站互动，并使测试在面对代码重构时更加稳健。

## 介绍 Mocha

Mocha 是一个简单、灵活的测试框架运行器。首先，我建议全局安装 Mocha，这样您就可以轻松地从命令行运行测试，如下所示：

```js
$ npm install --save-dev –g mocha

```

`--save-dev` 选项将 `mocha` 保存为开发依赖项，这意味着我们不需要在生产服务器上安装 Mocha。Mocha 只是一个测试运行器。我们还需要一个断言库。有各种各样的解决方案，但由 Express 和 Mocha 的同一人编写的 `should.js` 语法提供了一个干净的语法来制作断言：

```js
$ npm install --save-dev should

```

`should.js` 语法提供了 BDD 断言，例如 `'hello'.should.equal('hello')` 和 `[1,2].should.have.length(2)`。我们可以通过创建一个名为 `test` 的目录并包含一个名为 `hello-world.js` 的单个文件来开始一个 Hello World 测试示例，如下面的代码所示：

```js
var should = require('should');

describe('The World', function() {
  it('should say hello', function() {
    'Hello, World'.should.equal('Hello, World');
  });
  it('should say hello asynchronously!', function(done) {
    setTimeout(function() {
      'Hello, World'.should.equal('Hello, World');
      done();
    }, 300);
  });
});
```

我们在同一命名空间 `The World` 中有两个不同的测试。第一个测试是一个同步测试的例子。Mocha 执行我们给它提供的函数，看到没有抛出异常，测试通过。如果我们接受一个 `done` 参数作为回调，就像第二个例子中那样，Mocha 将智能地等待我们调用回调函数后再检查测试的有效性。大部分情况下，我们将使用第二种版本，我们必须显式调用 `done` 参数来完成我们的测试，因为这样做对测试 Express 应用更有意义。

现在，如果我们回到命令行，我们应该能够运行 Mocha（或者如果您没有全局安装，则是 `node_modules/.bin/mocha`）并看到我们编写的两个测试都通过了！

## 测试 API 端点

现在我们已经基本了解了如何使用 Mocha 运行测试以及如何使用 `should` 语法进行断言，我们可以将其应用于测试本地用户注册。首先，我们需要引入另一个 `npm` 模块，它将帮助我们以编程方式测试我们的服务器并对我们期望的响应类型进行断言。这个库叫做 `supertest`：

```js
$ npm install --save-dev supertest

```

这个库使测试 Express 应用变得轻而易举，并提供链式断言。让我们看看以下代码中如何测试我们的创建用户路由的示例：

```js
var should = require('should'),
    request = require('supertest'),
    app = require('../server').app,
    User = require('mongoose').model('User');

describe('Users', function() {
  before(function(done) {
    User.remove({}, done);
  });
  describe('registration', function() {
    it('should register valid user', function(done) {
      request(app)
        .post('/users/register')
        .send({
          email: "test@example.com",
          password: "hello world"
        })
        .expect(302)
        .end(function(err, res) {
          res.text.should.containEql("Redirecting to /");
          done(err);
        });
    });
  });
});
```

首先，请注意我们使用了两个命名空间：`Users` 和 `registration`。现在，在我们运行任何测试之前，我们从数据库中删除所有用户。这样做有助于确保我们知道测试的起点。这将删除您保存的所有用户，因此在使用测试环境时使用不同的数据库是有用的。Node 通过查看 `NODE_ENV` 环境变量来检测环境。通常它是测试、开发、预发布或生产。我们可以通过更改配置文件中的数据库 URL 来使用不同的本地数据库，在测试环境中运行 Mocha 测试，命令为 `NODE_ENV=test mocha`。

现在，让我们来看看有趣的部分！Supertest 提供了一个链式 API，用于发送请求并对响应进行断言。要发送请求，我们使用 `request(app)`。从那里，我们指定 HTTP 方法和方法路径。然后，我们可以指定要发送到服务器的 JSON 主体；在这种情况下，是一个示例用户注册表单。在注册时，我们期望发生重定向，即 `302` 响应。如果该断言失败，那么我们回调中的 `err` 参数将被填充，并且当使用 `done(err)` 时测试将失败。此外，我们验证是否被重定向到我们期望的路由，即服务器根 `/`。

# 自动化构建和部署

所有这些开发如果没有一个顺畅的构建和部署应用程序的过程都是相对无用的。幸运的是，Node 社区编写了各种任务运行器。其中包含 Grunt 和 Gulp，它们是最受欢迎的任务运行器之一。两者都与 Express 无缝协作，为我们提供了一系列实用工具，包括连接和压缩 JavaScript、编译 sass/less，以及在本地文件更改时重新加载服务器。我们将专注于 Grunt，以保持简单。

## 介绍 Gruntfile

Grunt 本身是一个简单的任务运行器，但它的可扩展性和插件架构允许您安装第三方脚本以在预定义的任务中运行。为了让我们了解如何使用 Grunt，我们将使用 `sass` 编写我们的 `css`，然后使用 Grunt 将 `sass` 编译为 `css`。通过这个示例，我们将探索 Grunt 引入的不同想法。首先，您需要全局安装 `cli` 以安装编译 `sass` 为 `css` 的插件：

```js
$ npm install -g grunt-cli 
$ npm install --save grunt grunt-contrib-sass

```

现在我们需要创建 `Gruntfile.js`，它包含了所有我们需要执行的任务和构建目标。为此，请执行以下操作：

```js
// Gruntfile.js
module.exports = function(grunt) {
  grunt.loadNpmTasks('grunt-contrib-sass');
  grunt.initConfig({
    sass: {
      dist: {
        files: [{
          expand: true,
          cwd: "public/styles",
          src: ["**.scss"],
          dest: "dist/styles",
          ext: ".css"
        }]
      }
    }
  });

}
```

让我们来看看主要部分。在最上面，我们引入了我们将使用的插件，`grunt-contrib-sass`。这告诉 `grunt` 我们将配置一个名为 `sass` 的任务。在我们的任务定义 `sass` 中，我们指定了一个目标，`dist`，它通常用于生成生产文件的任务（最小化、连接等）。

在那个任务中，我们动态构建文件列表，告诉 Grunt 递归地在 `/public/styles/` 中查找所有 `.scss` 文件，然后将它们编译到 `/dist/styles` 的相同路径。拥有两个并行的静态目录很有用，一个用于开发，一个用于生产，这样我们就不必在开发时查看压缩代码。我们可以通过执行 `grunt sass` 或 `grunt sass:dist` 来调用这个目标。

### 注意

值得注意的是，我们在这个任务中没有明确地连接文件，但如果我们使用 `@imports` 在我们的主 `sass` 文件中，编译器会为我们连接所有内容。

我们还可以配置 Grunt 来运行我们的测试套件。为此，让我们添加另一个插件 -- `npm install --save-dev grunt-mocha-test`。现在我们必须将以下代码添加到我们的 `Gruntfile.js` 文件中：

```js
grunt.loadNpmTasks('grunt-mocha-test');
grunt.registerTask('test', 'mochaTest');
...

  mochaTest: {
    test: {
      src: ["test/**.js"]
    }
  }
```

在这里，任务被称作 `mochaTest`，我们注册了一个名为 `test` 的新任务，它简单地委托给 `mochaTest` 任务。这样，记住如何运行测试就更容易了。同样，如果我们把字符串数组作为 `registerTask` 的第二个参数传递，我们也可以指定要运行的任务列表。这是 Grunt 可以实现的功能的样本。要查看一个更健壮的 Gruntfile 示例，请查看源代码。

## 与 Travis 的持续集成

Travis CI 为开源项目提供免费的持续集成服务，同时也为闭源应用程序提供付费选项。它使用 git 钩子在每次推送后自动测试你的应用程序。这有助于确保没有引入回归。此外，可能只有 CI 才能揭示的依赖性问题，而本地开发可能会掩盖这些问题；Travis 是这些错误的防线。它获取你的源代码，运行 `npm install` 以安装 `package.json` 中指定的依赖项，然后运行 `npm test` 来运行你的测试套件。

Travis 接受一个名为 `travis.yml` 的配置文件。这些通常看起来像这样：

```js
language: node_js
node_js:
  - "0.11"
- "0.10"
- "0.8"
services:
  - mongodb
```

我们可以指定要测试的 node 版本以及我们依赖的服务（特别是 MongoDB）。现在我们必须更新 `package.json` 中的测试命令，以运行 `grunt test`。最后，我们必须为相关的仓库设置一个 webhook。我们可以在 Travis 上通过启用仓库来完成此操作。现在我们只需推送我们的更改，Travis 就会确保所有测试通过！Travis 非常灵活，你可以用它来完成与持续集成相关的几乎所有任务，包括自动部署成功的构建。

## 部署 Node.js 应用程序

部署 Node.js 应用程序的最简单方法之一是利用 Heroku，这是一个平台即服务提供商。Heroku 有自己的工具包，可以从你的机器上创建和部署 Heroku 应用程序。在开始使用 Heroku 之前，你需要安装其工具包。

### 注意

请访问 [`toolbelt.heroku.com/`](https://toolbelt.heroku.com/) 下载 Heroku 工具包。

一旦安装，你可以通过 Web UI 登录 Heroku 或注册，然后运行 Heroku login。Heroku 使用一个特殊的文件，称为 Procfile，它指定了如何运行你的应用程序。

1.  我们的 Procfile 看起来像这样：

    ```js
    web: node server.js

    ```

    非常简单：为了运行 Web 服务器，只需运行 node server.js。

1.  为了验证我们的 Procfile 是否正确，我们可以在本地运行以下命令：

    ```js
    $ foreman start

    ```

1.  Foreman 会查看 Procfile，并使用它来尝试启动我们的服务器。一旦成功运行，我们需要创建一个新的应用程序，然后将我们的应用程序部署到 Heroku。请确保将 Procfile 提交到版本控制：

    ```js
    $ heroku create
    $ git push heroku master

    ```

    Heroku 将在 Heroku 上创建一个新的应用程序和 URL，以及一个名为 heroku 的 git 远程仓库。推送该远程仓库实际上会触发你的代码部署。

    如果你做了所有这些，不幸的是，你的应用程序将无法工作。我们没有为我们的应用程序提供 Mongo 实例进行通信！

1.  首先，我们必须从 Heroku 请求 MongoDB：

    ```js
    $ heroku addons:add mongolab // don't worry, it's free

    ```

    这将启动一个共享的 MongoDB 实例，并给我们的应用程序一个名为`MONOGOLAB_URI`的环境变量，我们应该将其用作我们的 MongoDB 连接 URI。我们需要更改我们的配置文件以反映这些更改。

    在我们的配置文件中，在生产环境中，对于我们的数据库 URL，我们应该查看环境变量`MONGOLAB_URI`。同时，确保 Express 正在监听`process.env.PORT || 3000`，否则你将收到奇怪的错误和/或超时。

1.  在设置好所有这些之后，我们可以提交我们的更改，并将更改再次推送到 Heroku。希望这次能成功！为了查看应用程序日志进行调试，只需使用 Heroku 工具包：

    ```js
    $ heroku logs

    ```

1.  关于部署 Express 应用程序的最后一件事：有时应用程序会崩溃，软件并不完美。我们应该预料到崩溃，并让我们的应用程序相应地做出反应（通过自动重启）。有许多服务器监控工具，包括 pm2 和 forever。我们使用 forever 是因为它的简单性。

    ```js
    $ npm install --save forever

    ```

1.  然后，我们更新我们的 Procfile 以反映我们对 forever 的使用：

    ```js
    // Procfile
    web: node_modules/.bin/forever server.js

    ```

现在，如果应用程序因为任何奇怪的原因崩溃，forever 将自动重启我们的应用程序。你还可以设置 Travis 以自动将成功的构建推送到你的服务器，但这超出了本书中我们将进行的部署。

# 摘要

在本章中，我们在 node 的世界里涉足，并使用 Express 框架。我们从 Hello World 和 MVC 到测试和部署都进行了介绍。你应该能够舒适地使用基本的 Express API，同时也应该有能力掌握整个`Node.js`应用程序栈。

在以下章节中，我们将基于本章介绍的核心思想来创建丰富的用户体验和引人入胜的应用程序。
