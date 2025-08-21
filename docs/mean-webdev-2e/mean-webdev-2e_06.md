# 第六章：使用护照管理用户身份验证

护照是一个强大的 Node.js 身份验证中间件，可帮助您对发送到 Express 应用程序的请求进行身份验证。护照使用策略来利用本地身份验证和 OAuth 身份验证提供程序，例如 Facebook、Twitter 和 Google。使用护照策略，您将能够无缝地为用户提供不同的身份验证选项，同时保持统一的用户模型。在本章中，您将了解护照的以下基本功能：

+   了解护照策略

+   将护照集成到用户的 MVC 架构中

+   使用护照的本地策略来验证用户

+   利用护照 OAuth 策略

+   通过社交 OAuth 提供程序提供身份验证

# 介绍护照

身份验证是大多数 Web 应用程序的重要部分。处理用户注册和登录是一个重要的功能，有时可能会带来开发开销。Express 以其精简的方式缺少了这个功能，因此，与 node 一样，需要一个外部模块。护照是一个使用中间件设计模式来验证请求的 Node.js 模块。它允许开发人员使用称为**策略**的机制提供各种身份验证方法，这使您能够实现复杂的身份验证层，同时保持代码清晰简洁。与任何其他 Node.js 模块一样，在应用程序中开始使用它之前，您首先需要安装它。本章中的示例将直接从前几章中的示例继续。因此，在本章中，从第五章*Mongoose 简介*中复制最终示例，然后从那里开始。

## 安装护照

护照使用不同的模块，每个模块代表不同的身份验证策略，但所有这些模块都依赖于基本的护照模块。要安装护照基本模块，请更改您的`package.json`文件如下：

```js
{
  "name": "MEAN",
  "version": "0.0.6",
  "dependencies": {
    "body-parser": "1.15.2",
    "compression": "1.6.0",
    "ejs": "2.5.2",
    "express": "4.14.0",
    "express-session": "1.14.1",
    "method-override": "2.3.6",
    "mongoose": "4.6.5",
    "morgan": "1.7.0",
 "passport": "0.3.2"
  }
}
```

在继续开发应用程序之前，请确保安装新的护照依赖项。要这样做，请转到应用程序的文件夹，并在命令行工具中发出以下命令：

```js
$ npm install

```

这将在您的`node_modules`文件夹中安装指定版本的护照。安装过程成功完成后，您将需要配置应用程序以加载护照模块。

## 配置护照

配置护照需要几个步骤。要创建护照配置文件，请转到`config`文件夹并创建一个名为`passport.js`的新文件。现在先留空；我们一会儿会回来的。接下来，您需要引用刚刚创建的文件，因此更改您的`server.js`文件如下：

```js
process.env.NODE_ENV = process.env.NODE_ENV || 'development';

const configureMongoose = require('./config/mongoose');
const configureExpress = require('./config/express');
const configurePassport = require('./config/passport');

const db = configureMongoose();
const app = configureExpress();
const passport = configurePassport();
app.listen(3000);

module.exports = app;

console.log('Server running at http://localhost:3000/');
```

接下来，您需要在 Express 应用程序中注册 Passport 中间件。要这样做，请更改您的`config/express.js`文件如下：

```js
const config = require('./config');
const express = require('express');
const morgan = require('morgan');
const compress = require('compression');
const bodyParser = require('body-parser');
const methodOverride = require('method-override');
const session = require('express-session');
const passport = require('passport');

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

 app.use(passport.initialize());
 app.use(passport.session());

  require('../app/routes/index.server.routes.js')(app);
  require('../app/routes/users.server.routes.js')(app);

  app.use(express.static('./public'));

  return app;
};
```

让我们回顾一下您刚刚添加的代码。首先，您需要引用护照模块，然后注册两个中间件：`passport.initialize()`中间件，负责引导护照模块，以及`passport.session()`中间件，使用 Express 会话来跟踪用户的会话。

护照现在已安装和配置，但要开始使用它，您将需要安装至少一个身份验证策略。我们将从本地策略开始，该策略提供了一个简单的用户名/密码身份验证层；但首先，让我们讨论一下护照策略的工作原理。

# 了解护照策略

为了提供各种身份验证选项，Passport 使用单独的模块来实现不同的身份验证策略。每个模块提供不同的身份验证方法，例如用户名/密码身份验证和 OAuth 身份验证。因此，为了提供 Passport 支持的身份验证，您需要安装和配置您想要使用的策略模块。让我们从本地身份验证策略开始。

## 使用 Passport 的本地策略

Passport 的本地策略是一个 Node.js 模块，允许您实现用户名/密码身份验证机制。您需要像安装其他模块一样安装它，并配置它以使用您的 User Mongoose 模型。让我们开始安装本地策略模块。

### 安装 Passport 的本地策略模块

要安装 Passport 的本地策略模块，您需要将`passport-local`添加到您的`package.json`文件中，如下所示：

```js
{
  "name": "MEAN",
  "version": "0.0.6",
  "dependencies": {
    "body-parser": "1.15.2",
    "compression": "1.6.0",
    "ejs": "2.5.2",
    "express": "4.14.0",
    "express-session": "1.14.1",
    "method-override": "2.3.6",
    "mongoose": "4.6.5",
    "morgan": "1.7.0",
    "passport": "0.3.2",
 "passport-local": "1.0.0"
  }
}
```

然后，转到应用程序的`根`文件夹，并在命令行工具中输入以下命令：

```js
$ npm install

```

这将在您的`node_modules`文件夹中安装指定版本的本地策略模块。安装过程成功完成后，您需要配置 Passport 以使用本地策略。

### 配置 Passport 的本地策略

您将使用的每种身份验证策略基本上都是一个允许您定义该策略将如何使用的节点模块。为了保持逻辑的清晰分离，每个策略都应该在其自己的分离文件中进行配置。在您的`config`文件夹中，创建一个名为`strategies`的新文件夹。在这个新文件夹中，创建一个名为`local.js`的文件，其中包含以下代码片段：

```js
const passport = require('passport');
const LocalStrategy = require('passport-local').Strategy;
const User = require('mongoose').model('User');

module.exports = function() {
  passport.use(new LocalStrategy((username, password, done) => {
    User.findOne({
      username: username
    }, (err, user) => {
      if (err) {
        return done(err);
      }

      if (!user) {
        return done(null, false, {
          message: 'Unknown user'
        });
      }

      if (!user.authenticate(password)) {
        return done(null, false, {
          message: 'Invalid password'
        });
      }

      return done(null, user);
      });
  }));
};
```

前面的代码首先需要`Passport`模块、本地策略模块的`Strategy`对象和您的`User` Mongoose 模型。然后，您可以使用`passport.use()`方法注册策略，该方法使用`LocalStrategy`对象的实例。请注意`LocalStrategy`构造函数将回调函数作为参数。稍后在尝试对用户进行身份验证时，它将调用此回调。

回调函数接受三个参数——`用户名`、`密码`和一个`完成`回调——当认证过程结束时将被调用。在回调函数内部，您将使用`User` Mongoose 模型来查找具有该用户名的用户并尝试对其进行身份验证。在出现错误时，您将把`error`对象传递给`done`回调。当用户经过身份验证时，您将使用`user Mongoose`对象调用`done`回调。

还记得空的`config/passport.js`文件吗？现在您已经准备好本地策略，可以返回并使用它来配置本地身份验证。为此，请返回到您的`config/passport.js`文件并粘贴以下代码行：

```js
const passport = require('passport');
const mongoose = require('mongoose');

module.exports = function() {
  const User = mongoose.model('User');

  passport.serializeUser((user, done) => {
    done(null, user.id);
  });

  passport.deserializeUser((id, done) => {
    User.findOne({
      _id: id
    }, '-password -salt', (err, user) => {
      done(err, user);
    });
  });

  require('./strategies/local.js')();
};
```

在前面的代码片段中，使用`passport.serializeUser()`和`passport.deserializeUser()`方法来定义 Passport 将如何处理用户序列化。当用户经过身份验证时，Passport 将其`_id`属性保存到会话中。稍后，当需要`user`对象时，Passport 将使用`_id`属性从数据库中获取`user`对象。请注意，我们使用了字段选项参数来确保 Mongoose 不会获取用户的密码和`salt`属性。前面的代码的第二件事是包含本地策略配置文件。这样，您的`server.js`文件将加载 Passport 配置文件，然后加载其策略配置文件。接下来，您需要修改您的`User`模型以支持 Passport 的身份验证。

## 调整用户模型

在上一章中，我们开始讨论`User`模型并创建了其基本结构。为了在您的 MEAN 应用程序中使用`User`模型，您将需要修改它以满足一些认证流程的要求。这些变化将包括修改`UserSchema`，添加一些`pre`中间件和添加一些新的实例方法。要做到这一点，请转到您的`app/models/user.js`文件，并按照以下方式进行更改：

```js
const mongoose = require('mongoose');
const crypto = require('crypto');
const Schema = mongoose.Schema;
const UserSchema = new Schema({
    firstName: String,
    lastName: String,
    email: {
        type: String,
        match: [/.+\@.+\..+/, "Please fill a valid e-mail address"]
    },
    username: {
        type: String,
        unique: true,
        required: 'Username is required',
        trim: true
    },
    password: {
        type: String,
        validate: [(password) => {
            return password && password.length > 6;
        }, 'Password should be longer']
    },
 salt: {
 type: String
 },
 provider: {
 type: String,
 required: 'Provider is required'
 },
 providerId: String,
 providerData: {},
    created: {
        type: Date,
        default: Date.now
    }
});

UserSchema.virtual('fullName').get(function() {
    return this.firstName + ' ' + this.lastName;
}).set(function(fullName) {
    const splitName = fullName.split(' ');
    this.firstName = splitName[0] || '';
    this.lastName = splitName[1] || '';
});

UserSchema.pre('save', function(next) {
 if (this.password) {
 this.salt = new
 Buffer(crypto.randomBytes(16).toString('base64'), 'base64');
 this.password = this.hashPassword(this.password);
 }
 next();
});

UserSchema.methods.hashPassword = function(password) {
 return crypto.pbkdf2Sync(password, this.salt, 10000,
 64).toString('base64');
};

UserSchema.methods.authenticate = function(password) {
 return this.password === this.hashPassword(password);
};

UserSchema.statics.findUniqueUsername = function(username, suffix,
 callback) {
 var possibleUsername = username + (suffix || '');
 this.findOne({
 username: possibleUsername
 }, (err, user) => {
 if (!err) {
 if (!user) {
 callback(possibleUsername);
 } else {
 return this.findUniqueUsername(username, (suffix || 0) +
 1, callback);
 }
 } else {
 callback(null);
 }
 });
};
UserSchema.set('toJSON', {
    getters: true,
    virtuals: true
});

mongoose.model('User', UserSchema);
```

让我们来看看这些变化。首先，您向`UserSchema`对象添加了四个字段：一个`salt`属性，用于对密码进行哈希处理；一个`provider`属性，用于指示注册用户所使用的策略；一个`providerId`属性，用于指示认证策略的用户标识符；以及一个`providerData`属性，稍后您将用它来存储从 OAuth 提供程序检索到的`user`对象。

接下来，您创建了一些`pre-save`中间件来处理用户密码的哈希处理。众所周知，存储用户密码的明文版本是一种非常糟糕的做法，可能导致用户密码泄露。为了解决这个问题，您的`pre-save`中间件执行了两个重要的步骤：首先，它创建了一个自动生成的伪随机哈希盐，然后使用`hashPassword()`实例方法将当前用户密码替换为哈希密码。

您还添加了两个实例方法：一个`hashPassword()`实例方法，用于通过利用 Node.js 的`crypto`模块对密码字符串进行哈希处理；以及一个`authenticate()`实例方法，它接受一个字符串参数，对其进行哈希处理，并将其与当前用户的哈希密码进行比较。最后，您添加了`findUniqueUsername()`静态方法，用于为新用户找到一个可用的唯一用户名。在本章后面处理 OAuth 认证时，您将使用这个方法。

这完成了对您的`User`模型的修改，但在您测试应用程序的认证层之前，还有一些其他事情需要处理。

## 创建认证视图

就像任何 Web 应用程序一样，您需要有注册和登录页面来处理用户认证。我们将使用**EJS**模板引擎创建这些视图，因此在您的`app/views`文件夹中，创建一个名为`signup.ejs`的新文件。在您新创建的文件中，粘贴以下代码片段：

```js
<!DOCTYPE html>
<html>
<head>
  <title>
    <%=title %>
  </title>
</head>
<body>
  <% for(var i in messages) { %>
    <div class="flash"><%= messages[i] %></div>
  <% } %>
  <form action="/signup" method="post">
    <div>
      <label>First Name:</label>
      <input type="text" name="firstName" />
    </div>
    <div>
      <label>Last Name:</label>
      <input type="text" name="lastName" />
    </div>
    <div>
      <label>Email:</label>
      <input type="text" name="email" />
    </div>
    <div>
      <label>Username:</label>
      <input type="text" name="username" />
    </div>
    <div>
      <label>Password:</label>
      <input type="password" name="password" />
    </div>
    <div>
      <input type="submit" value="Sign up" />
    </div>
  </form>
</body>
</html>
```

`signup.ejs`视图只包含一个 HTML 表单；一个 EJS 标签，用于呈现`title`变量；以及一个 EJS 循环，用于呈现`messages`列表变量。返回到您的`app/views`文件夹，并创建另一个文件，命名为`signin.ejs`。在这个文件中，粘贴以下代码片段：

```js
<!DOCTYPE html>
<html>
<head>
  <title>
    <%=title %>
  </title>
</head>
<body>
  <% for(var i in messages) { %>
    <div class="flash"><%= messages[i] %></div>
  <% } %>
  <form action="/signin" method="post">
    <div>
      <label>Username:</label>
      <input type="text" name="username" />
    </div>
    <div>
      <label>Password:</label>
      <input type="password" name="password" />
    </div>
    <div>
      <input type="submit" value="Sign In" />
    </div>
  </form>
</body>
</html>
```

如您所见，`signin.ejs`视图甚至更简单，也包含一个 HTML 表单；一个 EJS 标签，用于呈现`title`变量；以及一个 EJS 循环，用于呈现`messages`列表变量。现在您已经设置了模型和视图，是时候使用您的 Users 控制器将它们连接起来了。

## 修改 Users 控制器

要修改 Users 控制器，转到您的`app/controllers/users.server.controller.js`文件，并按照以下方式更改其内容：

```js
const User = require('mongoose').model('User');
const passport = require('passport');

function getErrorMessage(err) {
  let message = '';

  if (err.code) {
    switch (err.code) {
      case 11000:
      case 11001:
        message = 'Username already exists';
        break;
      default:
        message = 'Something went wrong';
    }
  } else {
    for (var errName in err.errors) {
      if (err.errors[errName].message) message = err.errors[errName].message;
    }
  }

  return message;
};

exports.renderSignin = function(req, res, next) {
  if (!req.user) {
    res.render('signin', {
      title: 'Sign-in Form',
      messages: req.flash('error') || req.flash('info')
    });
  } else {
    return res.redirect('/');
  }
};

exports.renderSignup = function(req, res, next) {
  if (!req.user) {
    res.render('signup', {
      title: 'Sign-up Form',
      messages: req.flash('error')
    });
  } else {
    return res.redirect('/');
  }
};

exports.signup = function(req, res, next) {
  if (!req.user) {
    const user = new User(req.body);
    user.provider = 'local';

    user.save((err) => {
      if (err) {
        const message = getErrorMessage(err);

        req.flash('error', message);
        return res.redirect('/signup');
      }
      req.login(user, (err) => {
        if (err) return next(err);
        return res.redirect('/');
      });
    });
  } else {
    return res.redirect('/');
  }
};

exports.signout = function(req, res) {
  req.logout();
  res.redirect('/');
};
```

`getErrorMessage()`方法是一个私有方法，它从 Mongoose `error`对象返回统一的错误消息。值得注意的是这里有两种可能的错误：使用错误代码处理的 MongoDB 索引错误，以及使用`err.errors`对象处理的 Mongoose 验证错误。

接下来的两个控制器方法非常简单，将用于呈现登录和注册页面。`signout()`方法也很简单，使用了 Passport 模块提供的`req.logout()`方法来使认证会话失效。

`signup()`方法使用您的`User`模型来创建新用户。正如您所看到的，它首先从 HTTP 请求体创建一个用户对象。然后，尝试将其保存到 MongoDB。如果发生错误，`signup()`方法将使用`getErrorMessage()`方法为用户提供适当的错误消息。如果用户创建成功，将使用`req.login()`方法创建用户会话。`req.login()`方法由`Passport`模块公开，并用于建立成功的登录会话。登录操作完成后，用户对象将被签名到`req.user`对象中。

### 注意

`req.login()`方法将在使用`passport.authenticate()`方法时自动调用，因此在注册新用户时主要使用手动调用`req.login()`。

在前面的代码中，使用了一个您尚不熟悉的模块。当身份验证过程失败时，通常会将请求重定向回注册或登录页面。当发生错误时，这里会这样做，但是您的用户如何知道到底出了什么问题？问题在于当重定向到另一个页面时，您无法将变量传递给该页面。解决方案是使用某种机制在请求之间传递临时消息。幸运的是，这种机制已经存在，以一个名为`Connect-Flash`的节点模块的形式存在。 

### 显示闪存错误消息

`Connect-Flash`模块是一个允许您将临时消息存储在会话对象的`flash`区域中的节点模块。存储在`flash`对象上的消息在呈现给用户后将被清除。这种架构使`Connect-Flash`模块非常适合在将请求重定向到另一个页面之前传递消息。

#### 安装 Connect-Flash 模块

要在应用程序的模块文件夹中安装`Connect-Flash`模块，您需要按照以下方式更改您的`package.json`文件：

```js
{
  "name": "MEAN",
  "version": "0.0.6",
  "dependencies": {
    "body-parser": "1.15.2",
    "compression": "1.6.0",
 "connect-flash": "0.1.1",
    "ejs": "2.5.2",
    "express": "4.14.0",
    "express-session": "1.14.1",
    "method-override": "2.3.6",
    "mongoose": "4.6.5",
    "morgan": "1.7.0",
    "passport": "0.3.2",
    "passport-local": "1.0.0"
  }
}
```

通常，在继续开发应用程序之前，您需要安装新的依赖项。转到应用程序文件夹，并在命令行工具中发出以下命令：

```js
$ npm install

```

这将在您的`node_modules`文件夹中安装指定版本的`Connect-Flash`模块。安装过程成功完成后，您的下一步是配置 Express 应用程序以使用`Connect-Flash`模块。

#### 配置 Connect-Flash 模块

要配置您的 Express 应用程序以使用新的`Connect-Flash`模块，您需要在 Express 配置文件中要求新模块，并使用`app.use()`方法将其注册到 Express 应用程序中。为此，请在您的`config/express.js`文件中进行以下更改：

```js
const config = require('./config');
const express = require('express');
const morgan = require('morgan');
const compress = require('compression');
const bodyParser = require('body-parser');
const methodOverride = require('method-override');
const session = require('express-session');
const flash = require('connect-flash');
const passport = require('passport');

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

 app.use(flash());
  app.use(passport.initialize());
  app.use(passport.session());

  require('../app/routes/index.server.routes.js')(app);
  require('../app/routes/users.server.routes.js')(app);

  app.use(express.static('./public'));

  return app;

};
```

这将告诉您的 Express 应用程序使用`Connect-Flash`模块，并在应用程序会话中创建新的闪存区域。

#### 使用 Connect-Flash 模块

安装后，`Connect-Flash`模块公开了`req.flash()`方法，允许您创建和检索闪存消息。为了更好地理解它，让我们观察您对用户控制器所做的更改。首先，让我们看看负责渲染登录和注册页面的`renderSignup()`和`renderSignin()`方法：

```js
exports.renderSignin = function(req, res, next) {
  if (!req.user) {
    res.render('signin', {
      title: 'Sign-in Form',
 messages: req.flash('error') || req.flash('info')
    });
  } else {
    return res.redirect('/');
  }
};

exports.renderSignup = function(req, res, next) {
  if (!req.user) {
    res.render('signup', {
      title: 'Sign-up Form',
 messages: req.flash('error')
    });
  } else {
    return res.redirect('/');
  }
}; 
```

如您所见，`res.render()`方法使用`title`和`messages`变量执行。messages 变量使用`req.flash()`读取消息写入闪存。现在，如果您查看`signup()`方法，您会注意到以下代码行：

```js
req.flash('error', message);
```

这是如何使用`req.flash()`方法将错误消息写入闪存的方式。在学习如何使用`Connect-Flash`模块之后，您可能已经注意到我们缺少一个`signin()`方法。这是因为 Passport 为您提供了一个身份验证方法，您可以直接在路由定义中使用。最后，让我们继续进行最后需要修改的部分：用户的路由定义文件。

## 连接用户路由

一旦您配置好模型、控制器和视图，剩下的就是定义用户的路由。为此，请在您的`app/routes/users.server.routes.js`文件中进行以下更改：

```js
const users = require('../../app/controllers/users.server.controller');
const passport = require('passport');

module.exports = function(app) {
  app.route('/signup')
     .get(users.renderSignup)
     .post(users.signup);

  app.route('/signin')
     .get(users.renderSignin)
     .post(passport.authenticate('local', {
       successRedirect: '/',
       failureRedirect: '/signin',
       failureFlash: true
     }));

  app.get('/signout', users.signout);
};
```

正如您所看到的，这里的大多数路由定义基本上是指向您的用户控制器中的方法。唯一不同的路由定义是处理发送到`/signin`路径的任何 POST 请求时使用`passport.authenticate()`方法。

当执行`passport.authenticate()`方法时，它将尝试使用其第一个参数定义的策略来验证用户请求。在这种情况下，它将尝试使用本地策略来验证请求。此方法接受的第二个参数是一个`options`对象，其中包含三个属性：

+   `successRedirect`：此属性告诉 Passport 在成功验证用户后将请求重定向到何处

+   `failureRedirect`：此属性告诉 Passport 在未能验证用户时将请求重定向到何处

+   `failureFlash`：此属性告诉 Passport 是否使用闪存消息

您几乎已经完成了基本的身份验证实现。要测试它，请对`app/controllers/index.server.controller.js`文件进行以下更改：

```js
exports.render = function(req, res) {
  res.render('index', {
    title: 'Hello World',
    userFullName: req.user ? req.user.fullName : ''
  });
};
```

这将向您的主页模板传递经过身份验证的用户的全名。您还需要对`app/views/index.ejs`文件进行以下更改：

```js
<!DOCTYPE html>
<html>
  <head>
      <title><%= title %></title>
    </head>
    <body>
      <% if ( userFullName ) { %>
        <h2>Hello <%=userFullName%> </h2> 
        <a href="/signout">Sign out</a>
      <% } else { %>
        <a href="/signup">Signup</a>
        <a href="/signin">Signin</a>
    <% } %>
    <br>
      <img src="img/logo.png" alt="Logo">
    </body>
</html>
```

就是这样！一切都准备好测试您的新身份验证层。转到您的根应用程序文件夹，并使用 node 命令行工具运行您的应用程序，然后输入以下命令：

```js
$ node server

```

通过访问`http://localhost:3000/signin`和`http://localhost:3000/signup`来测试您的应用程序。尝试注册，然后登录，不要忘记返回到您的主页，查看用户详细信息如何通过会话保存。

# 了解 Passport OAuth 策略

OAuth 是一种身份验证协议，允许用户使用外部提供者注册您的 Web 应用程序，而无需输入其用户名和密码。OAuth 主要由社交平台（如 Facebook、Twitter 和 Google）使用，允许用户使用其社交账户注册其他网站。

### 提示

要了解有关 OAuth 的更多信息，请访问[`oauth.net/`](http://oauth.net/)上的 OAuth 协议网站。

## 设置 OAuth 策略

Passport 支持基本的 OAuth 策略，这使您能够实现任何基于 OAuth 的身份验证。但是，它还支持通过主要的 OAuth 提供者进行用户身份验证，使用包装策略来帮助您避免自己实现复杂的机制。在本节中，我们将回顾顶级 OAuth 提供者以及如何实现其 Passport 身份验证策略。

### 注意

在开始之前，您需要联系 OAuth 提供者并创建一个开发者应用程序。此应用程序将具有 OAuth 客户端 ID 和 OAuth 客户端密钥，这将允许您对您的应用程序进行 OAuth 提供者的验证。

### 处理 OAuth 用户创建

OAuth 用户创建应该与本地`signup()`方法有些不同。由于用户是使用其他提供者的配置文件注册的，配置文件详细信息已经存在，这意味着您需要以不同的方式对它们进行验证。为此，请返回到您的`app/controllers/users.server.controller.js`文件，并添加以下方法：

```js
exports.saveOAuthUserProfile = function(req, profile, done) {
  User.findOne({
    provider: profile.provider,
    providerId: profile.providerId
  }, (err, user) => {
    if (err) {
      return done(err);
    } else {
      if (!user) {
        const possibleUsername = profile.username || ((profile.email) ? profile.email.split(@''@')[0] : '');

        User.findUniqueUsername(possibleUsername, null, (availableUsername) => {
          const newUser = new User(profile);
          newUser.username = availableUsername;

          newUser.save((err) => {

            return done(err, newUser);
          });
        });
      } else {
        return done(err, user);
      }
    }
  });
};
```

该方法接受一个用户资料，然后查找具有这些`providerId`和`provider`属性的现有用户。如果找到用户，它将使用用户的 MongoDB 文档调用`done()`回调方法。但是，如果找不到现有用户，它将使用 User 模型的`findUniqueUsername()`静态方法找到一个唯一的用户名，并保存一个新的用户实例。如果发生错误，`saveOAuthUserProfile()`方法将使用`done()`方法报告错误；否则，它将把用户对象传递给`done()`回调方法。一旦弄清楚了`saveOAuthUserProfile()`方法，就是时候实现第一个 OAuth 认证策略了。

### 使用 Passport 的 Facebook 策略

Facebook 可能是世界上最大的 OAuth 提供商。许多现代 Web 应用程序允许用户使用他们的 Facebook 资料注册 Web 应用程序。Passport 支持使用`passport-facebook`模块进行 Facebook OAuth 认证。让我们看看如何通过几个简单的步骤实现基于 Facebook 的认证。

#### 安装 Passport 的 Facebook 策略

要在应用程序的模块文件夹中安装 Passport 的 Facebook 模块，你需要按照以下方式更改你的`package.json`文件：

```js
{
  "name": "MEAN",
  "version": "0.0.6",
  "dependencies": {
    "body-parser": "1.15.2",
    "compression": "1.6.0",
    "connect-flash": "0.1.1",
    "ejs": "2.5.2",
    "express": "4.14.0",
    "express-session": "1.14.1",
    "method-override": "2.3.6",
    "mongoose": "4.6.5",
    "morgan": "1.7.0",
    "passport": "0.3.2",
 "passport-facebook": "2.1.1",
    "passport-local": "1.0.0"
  }
}
```

在继续开发应用之前，你需要安装新的 Facebook 策略依赖。为此，前往你应用的`root`文件夹，并在命令行工具中输入以下命令：

```js
$ npm install

```

这将在你的`node_modules`文件夹中安装指定版本的 Passport 的 Facebook 策略。安装过程成功完成后，你需要配置 Facebook 策略。

#### 配置 Passport 的 Facebook 策略

在开始配置 Facebook 策略之前，你需要前往 Facebook 的开发者主页[`developers.facebook.com/`](https://developers.facebook.com/)，创建一个新的 Facebook 应用，并将本地主机设置为应用域。配置完 Facebook 应用后，你将获得一个 Facebook 应用 ID 和密钥。你需要这些信息来通过 Facebook 对用户进行认证，所以让我们将它们保存在环境配置文件中。前往`config/env/development.js`文件，并进行以下更改：

```js
module.exports = {
  db: 'mongodb://localhost/mean-book',
  sessionSecret: 'developmentSessionSecret',
 facebook: {
 clientID: 'Application Id',
 clientSecret: 'Application Secret',
 callbackURL: 'http://localhost:3000/oauth/facebook/callback'
  }
};
```

不要忘记用你的 Facebook 应用 ID 和密钥替换`Application Id`和`Application Secret`。`callbackURL`属性将被传递给 Facebook OAuth 服务，在认证过程结束后将重定向到该 URL。确保`callbackURL`属性与你在开发者主页设置的回调设置匹配。

现在，前往你的`config/strategies`文件夹，创建一个名为`facebook.js`的新文件，其中包含以下代码片段：

```js
const passport = require('passport');
const url = require('url');
const FacebookStrategy = require('passport-facebook').Strategy;
const config = require('../config');
const users = require('../../app/controllers/users.server.controller');

module.exports = function() {
  passport.use(new FacebookStrategy({
    clientID: config.facebook.clientID,
    clientSecret: config.facebook.clientSecret,
    callbackURL: config.facebook.callbackURL,
    profileFields: ['id', 'name', 'displayName', 'emails'],
    passReqToCallback: true
  }, (req, accessToken, refreshToken, profile, done) => {
    const providerData = profile._json;
    providerData.accessToken = accessToken;
    providerData.refreshToken = refreshToken;

    const providerUserProfile = {
      firstName: profile.name.givenName,
      lastName: profile.name.familyName,
      fullName: profile.displayName,
      email: profile.emails[0].value,
      username: profile.name.givenName + profile.name.familyName,
      provider: 'facebook',
      providerId: profile.id,
      providerData: providerData
    };

    users.saveOAuthUserProfile(req, providerUserProfile, done);
  }));
};
```

让我们稍微回顾一下前面的代码片段。你首先需要引入`passport`模块、Facebook 策略对象、你的环境配置文件、你的`User` Mongoose 模型和 Users 控制器。然后，使用`passport.use()`方法注册策略，并创建一个`FacebookStrategy`对象的实例。`FacebookStrategy`构造函数接受两个参数：Facebook 应用信息和稍后在尝试认证用户时将调用的回调函数。

看一下你定义的回调函数。它接受五个参数：`HTTP 请求`对象，一个`accessToken`对象用于验证未来的请求，一个`refreshToken`对象用于获取新的访问令牌，一个包含用户资料的`profile`对象，以及在认证过程结束时调用的`done`回调函数。

在回调函数内部，你将使用 Facebook 资料信息创建一个新的用户对象，并使用控制器的`saveOAuthUserProfile()`方法对当前用户进行认证。

还记得`config/passport.js`文件吗？现在您已经配置了您的 Facebook 策略，您可以返回到该文件并加载策略文件。为此，返回`config/passport.js`文件并按以下方式更改它：

```js
const passport = require('passport');
const mongoose = require('mongoose');

module.exports = function() {
  const User = mongoose.model('User');

  passport.serializeUser((user, done) => {
    done(null, user.id);
  });

  passport.deserializeUser((id, done) => {
    User.findOne({
      _id: id
    }, '-password -salt', (err, user) => {
      done(err, user);
    });
  });

  require('./strategies/local.js')();
 require('./strategies/facebook.js')();
};
```

这将加载您的 Facebook 策略配置文件。现在，剩下的就是设置通过 Facebook 对用户进行身份验证所需的路由，并在您的登录和注册页面中包含指向这些路由的链接。

#### 连接 Passport 的 Facebook 策略路由

Passport OAuth 策略支持使用`passport.authenticate()`方法直接对用户进行身份验证的能力。要这样做，转到`app/routes/users.server.routes.js`，并在本地策略路由定义之后追加以下代码行：

```js
app.get('/oauth/facebook', passport.authenticate('facebook', {
  failureRedirect: '/signin'
}));

app.get('/oauth/facebook/callback', passport.authenticate('facebook', {
  failureRedirect: '/signin',
  successRedirect: '/'
}));
```

第一个路由将使用`passport.authenticate()`方法启动用户身份验证过程，而第二个路由将在用户链接其 Facebook 个人资料后使用`passport.authenticate()`方法完成身份验证过程。

就是这样！一切都为您的用户通过 Facebook 进行身份验证设置好了。现在您只需要转到您的`app/views/signup.ejs`和`app/views/signin.ejs`文件，并在关闭的`BODY`标签之前添加以下代码行：

```js
<a href="/oauth/facebook">Sign in with Facebook</a>
```

这将允许您的用户点击链接并通过其 Facebook 个人资料注册您的应用程序。

### 使用 Passport 的 Twitter 策略

另一个流行的 OAuth 提供程序是 Twitter，许多 Web 应用程序都提供用户使用其 Twitter 个人资料注册 Web 应用程序的功能。Passport 支持使用`passport-twitter`模块的 Twitter OAuth 身份验证方法。让我们看看如何通过几个简单的步骤实现基于 Twitter 的身份验证。

#### 安装 Passport 的 Twitter 策略

要在应用程序的模块文件夹中安装 Passport 的 Twitter 策略模块，您需要按照以下步骤更改您的`package.json`文件：

```js
{
  "name": "MEAN",
  "version": "0.0.6",
  "dependencies": {
    "body-parser": "1.15.2",
    "compression": "1.6.0",
    "connect-flash": "0.1.1",
    "ejs": "2.5.2",
    "express": "4.14.0",
    "express-session": "1.14.1",
    "method-override": "2.3.6",
    "mongoose": "4.6.5",
    "morgan": "1.7.0",
    "passport": "0.3.2",
    "passport-facebook": "2.1.1",
    "passport-local": "1.0.0",
 "passport-twitter": "1.0.4"
  }
}
```

在继续开发应用程序之前，您需要安装新的 Twitter 策略依赖项。转到您的应用程序的`root`文件夹，并在命令行工具中发出以下命令：

```js
$ npm install

```

这将在您的`node_modules`文件夹中安装指定版本的 Passport 的 Twitter 策略。安装过程成功完成后，您需要配置 Twitter 策略。

#### 配置 Passport 的 Twitter 策略

在开始配置 Twitter 策略之前，您需要转到 Twitter 开发者主页[`dev.twitter.com/`](https://dev.twitter.com/)并创建一个新的 Twitter 应用程序。配置 Twitter 应用程序后，您将获得 Twitter 应用程序 ID 和密钥。您需要它们来通过 Twitter 对用户进行身份验证，因此让我们将它们添加到我们的环境配置文件中。转到`config/env/development.js`文件，并按以下方式更改它：

```js
module.exports = {
  db: 'mongodb://localhost/mean-book',
  sessionSecret: 'developmentSessionSecret',
  facebook: {
    clientID: 'Application Id',
    clientSecret: 'Application Secret',
    callbackURL: 'http://localhost:3000/oauth/facebook/callback'
  },
 twitter: {
 clientID: 'Application Id',
 clientSecret: 'Application Secret',
 callbackURL: 'http://localhost:3000/oauth/twitter/callback'
 }
};
```

不要忘记用您的 Twitter 应用程序的 ID 和密钥替换`Application Id`和`Application Secret`。`callbackURL`属性将被传递给 Twitter OAuth 服务，该服务将在认证过程结束后将用户重定向到该 URL。确保`callbackURL`属性与您在开发者主页中设置的回调设置匹配。

如前所述，在您的项目中，每个策略都应该在自己单独的文件中进行配置，这将帮助您保持项目的组织。转到您的`config/strategies`文件夹，并创建一个名为`twitter.js`的新文件，其中包含以下代码行：

```js
const passport = require('passport');
const url = require('url');
const TwitterStrategy = require('passport-twitter').Strategy;
const config = require('../config');
const users = require('../../app/controllers/users.server.controller');

module.exports = function() {
  passport.use(new TwitterStrategy({
    consumerKey: config.twitter.clientID,
    consumerSecret: config.twitter.clientSecret,
    callbackURL: config.twitter.callbackURL,
    passReqToCallback: true
  }, (req, token, tokenSecret, profile, done) => {
    const providerData = profile._json;
    providerData.token = token;
    providerData.tokenSecret = tokenSecret;

    const providerUserProfile = {
      fullName: profile.displayName,
      username: profile.username,
      provider: 'twitter',
      providerId: profile.id,
      providerData: providerData
    };

    users.saveOAuthUserProfile(req, providerUserProfile, done);
  }));
};
```

您首先需要引入`passport`模块、`Twitter Strategy`对象、您的环境配置文件、您的`User` Mongoose 模型和 Users 控制器。然后，您使用`passport.use()`方法注册策略，并创建`TwitterStrategy`对象的实例。`TwitterStrategy`构造函数接受两个参数：Twitter 应用程序信息和稍后在尝试对用户进行身份验证时将调用的回调函数。

查看您定义的回调函数。它接受五个参数：`HTTP 请求`对象，一个`token`对象和一个`tokenSecret`对象来验证未来的请求，一个包含用户配置文件的`profile`对象，以及在身份验证过程结束时调用的`done`回调。

在回调函数中，您将使用 Twitter 配置文件信息创建一个新的用户对象，并使用您之前创建的控制器的`saveOAuthUserProfile()`方法来验证当前用户。

现在您已经配置了 Twitter 策略，您可以返回`config/passport.js`文件，并按照以下方式加载策略文件：

```js
const passport = require('passport');
const mongoose = require('mongoose');

module.exports = function() {
  const User = mongoose.model('User');

  passport.serializeUser((user, done) => {
    done(null, user.id);
  });

  passport.deserializeUser((id, done) => {
    User.findOne({
      _id: id
    }, '-password -salt, ', (err, user) => {
      done(err, user);
    });
  });

  require('./strategies/local.js')();
  require('./strategies/facebook.js')();
 require('./strategies/twitter.js')();
};
```

这将加载您的 Twitter 策略配置文件。现在，您只需要设置所需的路由来通过 Twitter 对用户进行身份验证，并在登录和注册页面中包含指向这些路由的链接。

#### 连接 Passport 的 Twitter 策略路由

要添加 Passport 的 Twitter 路由，请转到您的`app/routes/users.server.routes.js`文件，并在 Facebook 策略路由之后粘贴以下代码：

```js
app.get('/oauth/twitter', passport.authenticate('twitter', {
  failureRedirect: '/signin'
}));

app.get('/oauth/twitter/callback', passport.authenticate('twitter', {
  failureRedirect: '/signin',
  successRedirect: '/'
}));
```

第一个路由将使用`passport.authenticate()`方法启动用户身份验证过程，而第二个路由将在用户使用其 Twitter 配置文件连接后使用`passport.authenticate()`方法完成身份验证过程。

就是这样！您的用户的 Twitter 身份验证已经设置好了。您需要做的就是转到您的`app/views/signup.ejs`和`app/views/signin.ejs`文件，并在关闭的`BODY`标签之前添加以下代码行：

```js
<a href="/oauth/twitter">Sign in with Twitter</a>
```

这将允许您的用户点击链接，并通过其 Twitter 配置文件注册到您的应用程序。

### 使用 Passport 的 Google 策略

我们将实现的最后一个 OAuth 提供程序是 Google，因为许多 Web 应用程序都允许用户使用其 Google 配置文件注册 Web 应用程序。Passport 支持使用`passport-google-oauth`模块的 Google OAuth 身份验证方法。让我们看看如何通过几个简单的步骤实现基于 Google 的身份验证。

#### 安装 Passport 的 Google 策略

要在应用程序的模块文件夹中安装 Passport 的 Google 策略模块，您需要更改您的`package.json`文件，如下所示：

```js
{
  "name": "MEAN",
  "version": "0.0.6",
  "dependencies": {
    "body-parser": "1.15.2",
    "compression": "1.6.0",
    "connect-flash": "0.1.1",
    "ejs": "2.5.2",
    "express": "4.14.0",
    "express-session": "1.14.1",
    "method-override": "2.3.6",
    "mongoose": "4.6.5",
    "morgan": "1.7.0",
    "passport": "0.3.2",
    "passport-facebook": "2.1.1",    
 "passport-google-oauth": "1.0.0",
    "passport-local": "1.0.0",
    "passport-twitter": "1.0.4"
  }
}

```

在您继续开发应用程序之前，您需要安装新的谷歌策略依赖项。转到应用程序的“根”文件夹，并在命令行工具中输入以下命令：

```js
$ npm install

```

这将在您的`node_modules`文件夹中安装 Passport 的 Google 策略的指定版本。安装过程成功完成后，您需要配置 Google 策略。

#### 配置 Passport 的 Google 策略

在我们开始配置您的 Google 策略之前，您需要转到 Google 开发人员主页[`console.developers.google.com/`](https://console.developers.google.com/)并创建一个新的 Google 应用程序。在应用程序的设置中，将`JAVASCRIPT ORIGINS`属性设置为`http://localhost`，将`REDIRECT URLs`属性设置为`http://localhost/oauth/google/callback`。配置完您的 Google 应用程序后，您将获得 Google 应用程序 ID 和密钥。您需要它们来通过 Google 对用户进行身份验证，因此让我们将它们添加到我们的环境配置文件中。转到`config/env/development.js`文件，并更改如下：

```js
module.exports = {
  db: 'mongodb://localhost/mean-book',
  sessionSecret: 'developmentSessionSecret',
  facebook: {
    clientID: 'Application Id',
    clientSecret: 'Application Secret',
    callbackURL: 'http://localhost:3000/oauth/facebook/callback'
  },
  twitter: {
    clientID: 'Application Id',
    clientSecret: 'Application Secret',
    callbackURL: 'http://localhost:3000/oauth/twitter/callback'
  },
 google: {
 clientID: 'Application Id',
 clientSecret: 'Application Secret',
 callbackURL: 'http://localhost:3000/oauth/google/callback'
 }
};
```

不要忘记用您的 Google 应用程序的 ID 和密钥替换`Application Id`和`Application Secret`。`callbackURL`属性将传递给 Google OAuth 服务，在身份验证过程结束后将用户重定向到该 URL。确保`callbackURL`属性与您在开发人员主页中设置的回调设置匹配。

要实现 Google 身份验证策略，请转到您的`config/strategies`文件夹，并创建一个名为`google.js`的新文件，其中包含以下代码行：

```js
const passport = require('passport');
const url = require('url');,
const GoogleStrategy = require('passport-google-oauth').OAuth2Strategy;
const config = require(../config');
const users = require('../../app/controllers/users.server.controller');

module.exports = function() {
  passport.use(new GoogleStrategy({
    clientID: config.google.clientID,
    clientSecret: config.google.clientSecret,
    callbackURL: config.google.callbackURL,
    passReqToCallback: true
  }, (req, accessToken, refreshToken, profile, done) => {
    const providerData = profile._json;
    providerData.accessToken = accessToken;
    providerData.refreshToken = refreshToken;

    const providerUserProfile = {
      firstName: profile.name.givenName,
      lastName: profile.name.familyName,
      fullName: profile.displayName,
      email: profile.emails[0].value,
      username: profile.username,
      provider: 'google''google',
      providerId: profile.id,
      providerData: providerData
    };

    users.saveOAuthUserProfile(req, providerUserProfile, done);
  }));
};
```

让我们稍微回顾一下前面的代码片段。您首先需要引入`passport`模块、Google 策略对象、您的环境配置文件、`User` Mongoose 模型和用户控制器。然后，使用`passport.use()`方法注册策略，并创建一个`GoogleStrategy`对象的实例。`GoogleStrategy`构造函数接受两个参数：Google 应用程序信息和稍后在尝试对用户进行身份验证时将调用的回调函数。

查看您定义的回调函数。它接受五个参数：`HTTP 请求`对象，用于验证未来请求的`accessToken`对象，用于获取新访问令牌的`refreshToken`对象，包含用户配置文件的`profile`对象，以及在认证过程结束时调用的`done`回调。

在回调函数中，您将使用 Google 配置文件信息和控制器的`saveOAuthUserProfile()`方法创建一个新的用户对象，该方法是您之前创建的，用于验证当前用户。

现在您已经配置了 Google 策略，可以返回到`config/passport.js`文件并加载策略文件，如下所示：

```js
const passport = require('passport');
const mongoose = require('mongoose');

module.exports = function() {
  const User = mongoose.model('User');

  passport.serializeUser((user, done) => {
    done(null, user.id);
  });

  passport.deserializeUser((id, done) => {
    User.findOne({
      _id: id
    }, '-password -salt', function(err, user) => {
      done(err, user);
    });
  });

  require('./strategies/local.js')();
  require('./strategies/facebook.js')();
  require('./strategies/twitter.js')();
 require('./strategies/google.js')();
};
```

这将加载您的 Google 策略配置文件。现在剩下的就是设置所需的路由来通过 Google 对用户进行身份验证，并在您的登录和注册页面中包含指向这些路由的链接。

#### 连接 Passport 的 Google 策略路由

要添加 Passport 的 Google 路由，请转到您的`app/routes/users.server.routes.js`文件，并在 Twitter 策略路由之后粘贴以下代码行：

```js
app.get('/oauth/google', passport.authenticate('google', {
  failureRedirect: '/signin',
  scope: [
    'https://www.googleapis.com/auth/userinfo.profile',
    'https://www.googleapis.com/auth/userinfo.email'
  ],
}));

app.get('/oauth/google/callback', passport.authenticate('google', {
  failureRedirect: '/signin',
  successRedirect: '/'
}));
```

第一个路由将使用`passport.authenticate()`方法启动用户身份验证过程，而第二个路由将使用`passport.authenticate()`方法在用户使用其 Google 配置文件连接后完成身份验证过程。

就是这样！一切都为您的用户基于 Google 的身份验证设置好了。您只需转到您的`app/views/signup.ejs`和`app/views/signin.ejs`文件，并在关闭的`BODY`标签之前添加以下代码行：

```js
<a href="/oauth/google">Sign in with Google</a>
```

这将允许您的用户点击链接并通过其 Google 配置文件注册您的应用程序。要测试您的新身份验证层，转到应用程序的`root`文件夹，并使用 node 命令行工具运行您的应用程序：

```js
$ node server

```

通过访问`http://localhost:3000/signin`和`http://localhost:3000/signup`来测试您的应用程序。尝试使用新的 OAuth 方法进行注册和登录。不要忘记访问您的主页，查看用户详细信息在整个会话期间是如何保存的。

### 提示

Passport 还为许多其他 OAuth 提供程序提供类似的支持。要了解更多信息，建议您访问[`passportjs.org/guide/providers/`](http://passportjs.org/guide/providers/)。

# 总结

在本章中，您了解了 Passport 身份验证模块。您了解了其策略以及如何处理其安装和配置。您还学会了如何正确注册用户以及如何验证其请求。您已经了解了 Passport 的本地策略，并学会了如何使用用户名和密码对用户进行身份验证，以及 Passport 如何支持不同的 OAuth 身份验证提供程序。在下一章中，我们将向您介绍 MEAN 拼图的最后一部分，即**Angular**。
