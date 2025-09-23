# 第二章。支出跟踪器

在本章中，我们将看到如何构建支出跟踪器应用程序。它将存储给定类别的所有支出。我们将能够看到我们支出的汇总余额或按类别划分的支出。每个用户都将有一个单独的账户来管理他们的支出。

我们将涵盖的一些有趣的主题包括：

+   创建多用户系统

+   处理货币数据

+   使用 MongoDB 聚合框架

+   不同的身份验证策略，例如 HTTP 基本身份验证和基于令牌的身份验证

# 设置基本应用程序

让我们设置应用程序的基本结构和文件。项目的全部源代码将以捆绑包的形式在 [`www.packtpub.com/`](https://www.packtpub.com/) 上提供。因此，我们只将详细说明设置基本应用程序的最重要部分。

## 安装依赖项

让我们从在项目根目录中创建我们的 `package.json` 文件并添加以下代码开始：

```js
{
  "name": "mean-blueprints-expensetracker",
  "version": "0.0.1",
  "repository": {
    "type": "git",
    "url": "https://github.com/robert52/mean-blueprints-expensetracker.git"
  },
  "engines": {
    "node": ">=0.12.0"
  },
  "scripts": {
    "start": "node app.js",
    "unit": "mocha tests/unit/ --ui bdd --recursive --reporter spec --timeout 10000 --slow 900",
    "integration": "mocha tests/integration/ --ui bdd --recursive --reporter spec --timeout 10000 --slow 900"
  },
  "dependencies": {
    "async": "⁰.9.0",
    "body-parser": "¹.12.3",
    "express": "⁴.12.4",
    "express-session": "¹.11.2",
    "lodash": "³.7.0",
    "method-override": "².3.2",
    "mongoose": "⁴.0.2",
    "passport": "⁰.2.1",
    "passport-local": "¹.0.0",
    "serve-static": "¹.9.2"
  },
  "devDependencies": {
    "chai": "².3.0",
    "chai-things": "⁰.2.0",
    "mocha": "².2.4",
    "request": "².55.0"
  }
}
```

定义 `package.json` 文件后的下一步是安装必要的依赖项。运行以下命令：

```js
npm install

```

在 `npm` 拉取所有必要的文件后，你应该会返回到命令提示符。

## 创建基本配置文件

我们将重用之前联系人管理器项目中的大量代码。我们创建了一个文件来根据当前运行的 Node 环境加载必要的环境配置文件。添加一个新的配置文件。

创建一个名为 `config/environments/development.js` 的文件，并添加以下代码：

```js
'use strict';

module.exports = {
  port: 3000,
  hostname: 'localhost',
  baseUrl: 'http://localhost:3000',
  mongodb: {
    uri: 'mongodb://localhost/expense_dev_db'
  },
  app: {
    name: 'Expense tracker'
  },
  serveStatic: true,
  session: {
    session: {
      type: 'mongo',
      secret: 'someVeRyN1c3S#cr3tHer34U',
      resave: false,
      saveUninitialized: true
    }
  }
};
```

接下来，我们将为 Express 创建配置文件，并将以下代码行添加到 `config/express.js` 文件中：

```js
'use strict';

const path = require('path');
const bodyParser = require('body-parser');
const methodOverride = require('method-override');
const serveStatic = require('serve-static');
const session = require('express-session');
const MongoStore = require('connect-mongo')(session);
const passport = require('passport');
const config = require('./index');

module.exports.init = initExpress

function initExpress(app) {
  const env = app.get('env');
  const root = app.get('root');
  const sessionOpts = {
    secret: config.session.secret,
    key: 'skey.sid',
    resave: config.session.resave,
    saveUninitialized: config.session.saveUninitialized
  };

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

  if (config.serveStatic) {
    app.use(serveStatic(path.join(root, 'public')));
  }
}
```

最后，我们将添加一个名为 `config/mongoose.js` 的文件来连接到 MongoDB，内容如下：

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
}

function cleanup() {
  mongoose.connection.close(function () {
    console.log('Closing DB connections and stopping the app. Bye bye.');
    process.exit(0);
  });
}
```

## 创建主 server.js 文件

我们应用程序的主要入口点是 `server.js` 文件。在项目的根目录中创建它。此文件启动网络服务器并引导所有逻辑。添加以下代码行：

```js
'use strict';

// Get process environment or set default environment to development
const ENV = process.env.NODE_ENV || 'development';
const DEFAULT_PORT = 3000;
const DEFAULT_HOSTNAME = 'localhost';

const http = require('http');
const express = require('express');
const config = require('./config');
const app = express();
let server;

/**
 * Set express (app) variables
 */
app.set('config', config);
app.set('root', __dirname);
app.set('env', ENV);

require('./config/mongoose').init(app);
require('./config/models').init(app);
require('./config/passport').init(app);
require('./config/express').init(app);
require('./config/routes').init(app);

app.use((err, req, res, next) => {
  res.status(500).json(err);
});

/**
 * Start the app if not loaded by another module
 */
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

# 设置用户部分

在上一章中，我们也有一个应用程序的用户部分。在本章中，我们将通过添加注册和更改密码功能来扩展这些功能。我们将重用现有的代码库并添加新功能。

## 描述用户模型

我们将创建一个专门针对用户模型的测试文件。这将有助于在不启动整个应用程序的情况下测试其所有功能。创建一个名为 `test/integration/user.model.test.js` 的文件，并添加以下内容：

```js
'use strict';

/**
 * Important! Set the environment to test
 */
process.env.NODE_ENV = 'test';

const chai = require('chai');
const should = chai.should();
consst config = require('../../config/environments/test');

describe('User model', function() {
  const mongoose;
  const User;
  const _user;
  const newUserData = {
    email: 'jane.doe@test.com',
    password: 'user_password',
    name: 'Jane Doe'
  };

  before(function(done) {
    mongoose = require('../../config/mongoose').init();
    User = require('../../app/models/user');
    done();
  });

  after(function(done) {
    User.remove({}).exec(function(err) {
      if (err) throw err;

      mongoose.connection.close(function() {
        setTimeout(function() { done(); }, 1000);
      });
    });
  });
});
```

我们已经为测试文件定义了基础。现在我们将逐个添加测试用例，在最后一个闭合括号之前：

1.  用户应该能够使用我们的系统进行注册。我们可以用以下代码行进行测试：

    ```js
      it('should register a user', function(done) {
        User.register(newUserData, function(err, user) {
          if (err) throw err;

          should.exist(user);
          user.email.should.equal(newUserData.email);
          should.not.exist(user.password);
          should.not.exist(user.passwordSalt);
          should.exist(user.createdAt);
          user.active.should.equal(true);

          _user = user;
          done();
        });
      });
    ```

1.  同一用户不能使用相同的电子邮件地址进行两次注册：

    ```js
      it('should not register a user if already exists', function(done) {
        User.register(newUserData, function(err, user) {
          should.exist(err);
          err.code.should.equal(11000); // duplicate key error
          should.not.exist(user);
          done();
        });
      });
    ```

1.  注册成功后，用户应该能够对我们的系统进行身份验证：

    ```js
      it('should authenticate a user with valid credentials', function(done) {
        User.authenticate(newUserData.email, 'user_password', function(err, user) {
          if (err) throw err;

          should.exist(user);
          should.not.exist(user.password);
          should.not.exist(user.passwordSalt);
          user.email.should.equal(newUserData.email);
          done();
        });
      });
    ```

1.  如果用户提供了无效的凭据，则不应成功认证：

    ```js
      it('should not authenticate user with invalid credentials', function(done) {
        User.authenticate(newUserData.email, 'notuserpassowrd', function(err, user) {
          if (err) throw err;

          should.not.exist(user);
          done();
        });
      });
    ```

1.  用户应该能够更改当前密码：

    ```js
      it('should change the password of a user', function(done) {
        _user.changePassword('user_password', 'new_user_password', function(err, result) {
          if (err) throw err;

          should.exist(result);
          result.success.should.equal(true);
          result.message.should.equal('Password changed successfully.');
          result.type.should.equal('password_change_success');

          // run a check credential with the new password
          User.authenticate(_user.email, 'new_user_password', function(err, user) {
            if (err) throw err;

            should.exist(user);
            user.email.should.equal(_user.email);
            done();
          });
        });
      });
    ```

1.  必须通过旧密码挑战才能设置新密码：

    ```js
      it('should not change password if old password does not match', function(done) {
        _user.changePassword('not_good', 'new_user_password', function(err, result) {
          should.not.exist(result);
          should.exist(err);
          err.type.should.equal('old_password_does_not_match');

          // run a check credential with the old password
          User.authenticate(_user.email, 'new_user_password', function(err, user) {
            if (err) throw err;

            should.exist(user);
            user.email.should.equal(_user.email);
            done();
          });
        });
      });
    ```

使用前面的测试套件，我们描述并将要测试我们实现的方法的功能。

## 实现用户模型

`user` 模型将使用与 第一章 中相同的密码辅助原则，即 *联系人管理器*。让我们创建一个名为 `app/helpers/password.js` 的文件。该文件应包含以下代码：

```js
'use strict';

const LEN = 256;
const SALT_LEN = 64;
const ITERATIONS = 10000;
const DIGEST = 'sha256';
const crypto = require('crypto');

module.exports.hash = hashPassword;
module.exports.verify = verify;
```

现在添加 `hashPassword()` 函数：

```js
function hashPassword(password, salt, callback) {
  let len = LEN / 2;

  if (3 === arguments.length) {
    generateDerivedKey(password, salt, ITERATIONS, len, DIGEST, callback);
  } else {
    callback = salt;
    crypto.randomBytes(SALT_LEN / 2, (err, salt) => {
      if (err) {
        return callback(err);
      }

      salt = salt.toString('hex');
      generateDerivedKey(password, salt, ITERATIONS, len, DIGEST, callback);
    });
  }
}
```

我们添加了一个额外的函数，称为 `generateDerivedKey()`，以避免重复代码块：

```js
function generateDerivedKey(password, salt, iterations, len, digest, callback) {
  crypto.pbkdf2(password, salt, ITERATIONS, len, DIGEST, (err, derivedKey) => {
    if (err) {
      return callback(err);
    }

    return callback(null, derivedKey.toString('hex'), salt);
  });
}
```

最后，添加 `verify()` 函数：

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

接下来，让我们在模型文件中创建一个用户模式。创建一个名为 `app/models/user.js` 的新文件，并添加以下内容：

```js
'use strict';

const _ = require('lodash');
const mongoose = require('mongoose');
const passwordHelper = require('../helpers/password');
const Schema = mongoose.Schema;

const UserSchema = new Schema({
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
  phoneNumber: {
    type: String
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

UserSchema.statics.register = registerUser;
UserSchema.statics.authenticate = authenticateUser;
UserSchema.methods.changePassword = changeUserPassword;
```

现在，让我们逐个添加测试中所需的方法。我们将从 `register()` 方法开始。将这些代码行追加到用户模型文件中：

```js
function registerUser(opts, callback) {
  let data = _.cloneDeep(opts);

  //hash the password
  passwordHelper.hash(opts.password, (err, hashedPassword, salt) => {
    if (err) {
      return callback(err);
    }

    data.password = hashedPassword;
    data.passwordSalt = salt;

    //create the user
    this.model('User').create(data, (err, user) => {
      if (err) {
        return callback(err, null);
      }

      // remove password and salt from the result
      user.password = undefined;
      user.passwordSalt = undefined;
      // return user if everything is ok
      callback(err, user);
    });
  });
}
```

这是一个简单的函数，它将在 MongoDB 中保存用户。在保存用户之前，我们希望从给定的密码中构建一个哈希值，并将该哈希值与盐一起保存在数据库中，而不是明文密码字符串。Mongoose 还会在保存之前根据用户模式验证用户数据。

对于 `authenticate()` 方法，我们将追加以下代码行：

```js
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

认证方法将通过电子邮件查找用户。对于此查询，`password` 和 `passwordSalt` 字段被明确设置为仅从数据库中读取。将调用密码验证函数来匹配现有的密码哈希与发送给认证方法的密码。

最后，我们将添加一个 `changePassword()` 方法。此方法仅对用户实例可用。Mongoose 允许我们在模式上使用 `methods` 属性来附加新函数。追加以下代码：

```js
function changeUserPassword(oldPassword, newPassword, callback) {
  this
  .model('User')
  .findById(this.id)
  .select('+password +passwordSalt')
  .exec((err, user) => {
    if (err) {
      return callback(err, null);
    }

    // no user found just return the empty user
    if (!user) {
      return callback(err, user);
    }

    passwordHelper.verify(
      oldPassword,
      user.password,
      user.passwordSalt,
      (err, result) => {
        if (err) {
          return callback(err, null);
        }

        // if password does not match don't return user
        if (result === false) {
          let PassNoMatchError = new Error('Old password does not match.');
          PassNoMatchError.type = 'old_password_does_not_match';
          return callback(PassNoMatchError, null);
        }

        // generate the new password and save the user
        passwordHelper.hash(newPassword, (err, hashedPassword, salt) => {
          this.password = hashedPassword;
          this.passwordSalt = salt;

          this.save((err, saved) => {
            if (err) {
              return callback(err, null);
            }

            if (callback) {
              return callback(null, {
                success: true,
                message: 'Password changed successfully.',
                type: 'password_change_success'
              });
            }
          });
        });
      }
    );
  });
}
```

更改密码功能是使用三个小步骤构建的。第一步是从数据库中获取用户的密码和盐。返回的数据用于验证用户输入的旧密码与现有的密码哈希和盐。如果一切顺利，则使用生成的盐对新的密码进行哈希处理，并将用户实例保存到 MongoDB 中。

不要忘记将以下代码行移动到文件末尾，以便编译用户模型：

```js
module.exports = mongoose.model('User', UserSchema);
```

假设我们使用以下命令运行我们的用户模型测试：

```js
mocha tests/integration/user.mode.test.js

```

我们应该看到所有测试都通过：

```js
 User Model Integration
 #register()
 √ should create a new user (124ms)
 √ should not create a new user if email already exists (100ms)
 #authenticate()
 √ should return the user if the credentials are valid (63ms)
 √ should return nothing if the credential of the user are invalid (62ms)
 #changePassword()
 √ should change the password of a user (223ms)
 √ should not change password if old password does not match (146ms)

 6 passing (1s)

```

# 用户认证

在上一章中，我们使用了基于会话的认证。对于本章，我们将探索不同的解决方案——使用访问令牌来认证我们的用户。

访问令牌在 RESTful API 中被广泛使用。因为我们构建应用程序的前提是它不仅可以用作我们的 Angular 应用，还可以用于许多其他客户端应用程序，我们需要依赖某种可以用来识别用户的东西。

访问令牌是一个标识用户（甚至是一个应用程序）的字符串，它可以用来对我们的系统进行 API 调用。令牌可以通过多种方式发放。例如，可以使用 OAuth 2.0 简单地发放令牌。

对于本章，我们将构建一个自定义模块，该模块负责创建令牌。这将使我们能够轻松切换到任何其他可用的解决方案。

我们将实现两种策略来认证我们的用户。其中之一将是一个 HTTP Basic 认证策略，它将使用简单的用户名（在我们的情况下是电子邮件）和密码组合来认证用户，并生成一个用于后续 API 调用的令牌。第二种策略是 HTTP Bearer 认证，它将使用 Basic 认证发放的访问令牌来授予用户访问资源的权限。

## 描述认证策略

在实现任何代码之前，我们应该创建一个测试来描述关于用户认证的期望行为。创建一个名为 `tests/integration/authentication.test.js` 的文件，并描述主要测试用例：

1.  第一个测试用例应考虑一个积极场景，即当用户尝试使用有效凭据进行认证时。这看起来如下所示：

    ```js
        it('should authenticate a user and return a new token', function(done) {
          request({
            method: 'POST',
            url: baseUrl + '/auth/basic',
            auth: {
              username: userFixture.email,
              password: 'P@ssw0rd!'
            },
            json:true
          }, function(err, res, body) {
            if (err) throw err;

            res.statusCode.should.equal(200);
            body.email.should.equal(userFixture.email);
            should.not.exist(body.password);
            should.not.exist(body.passwordSalt);
            should.exist(body.token);
            should.exist(body.token.hash);
            should.exist(body.token.expiresAt);
            done();
          });
        });
    ```

1.  如果用户尝试使用无效凭据进行认证，系统应返回一个错误请求消息：

    ```js
        it('should not authenticate a user with invalid credentials', function(done) {
          request({
            method: 'POST',
            url: baseUrl + '/auth/basic',
            auth: {
              username: userFixture.email,
              password: 'incorrectpassword'
            },
            json:true
          }, function(err, res, body) {
            if (err) throw err;

            res.statusCode.should.equal(400);
            body.message.should.equal('Invalid email or password.');
            done();
          });
        });
    ```

我们描述了基本策略。我们考虑了用户必须通过 POST 调用 `/api/auth` 端点发送电子邮件作为用户名和密码，并获取用户详情和有效令牌的事实。

### 提示

`request` 库有一个名为 `auth` 的特殊属性，它将使用 base64 对用户名和密码元组进行编码，并设置适当的 HTTP Basic 认证头。

如您所见，我们的假设是当用户成功登录到我们的系统时，将生成一个有效的令牌。因此，在继续前进之前，我们将实现令牌生成功能。

## 实现令牌生成

令牌可以通过多种方式生成。在本章中，我们将使用 Node.js 的内置加密库。我们可以使用 `randomBytes()` 方法生成指定长度的随机字符串。需要注意的是，如果累积的熵不足，`randomBytes()` 将抛出错误。这意味着如果熵源中信息不足，无法生成随机数，它将抛出错误。

让我们创建一个名为 `app/helpers/token.js` 的新文件，并添加以下代码行：

```js
'use strict';

const LEN = 16;
const crypto = require('crypto');

module.exports.generate = generateToken;

function generateToken(size, callback) {
  if (typeof size === 'function') {
    callback = size;
    size = LEN;
  }

  // we will return the token in `hex`
  size = size / 2;

  crypto.randomBytes(size, (err, buf) => {
    if (err) {
      return callback(err);
    }

    const token = buf.toString('hex');

    callback(null, token);
  });
} 
```

我们创建了一个辅助函数，它将为我们生成一个随机令牌。该函数接受两个参数：随机字节数，这是可选的，以及一个回调函数。

## 在 MongoDB 中持久化令牌

为了检查用户发送的访问令牌——即它是否有效——我们应该将其存储在某个地方。为此，我们将使用 MongoDB 作为我们的令牌存储引擎。

### 提示

注意，你应该像对待用户密码一样对待你的令牌，因为令牌将提供对系统功能的访问。为了进一步提高安全性，可以考虑将令牌加密存储在数据库中，或者甚至将它们存储在单独的令牌存储中。

在做任何事情之前，让我们为令牌模型创建一个测试。创建一个名为 `tests/integration/token.model.js` 的文件，并添加以下代码：

```js
process.env.NODE_ENV = 'test';

const chai = require('chai');
const should = chai.should();
const mongoose = require('../../config/mongoose').init();
const Token = require('../../app/models/token');

describe('Token Model Integration', function() {
  after(function(done) {
    mongoose.connection.db.dropDatabase(function(err) {
      if (err) throw err;

      setTimeout(done, 200);
    });
  });

  describe('#generate() - Token class method', function() {
    var _userId = new mongoose.Types.ObjectId();

    it('should generate a new token for a user', function(done) {
      Token.generate({
        user: _userId
      }, function(err, token) {
        if (err) throw err;

        should.exist(token);
        should.exist(token.id);
        token.hash.length.should.equal(32);
        token.user.toString().should.equal(_userId.toString());
        done();
      });
    });
  });

});
```

我们将向 `Token` 模型添加一个 `generate()` 方法，该方法将返回一个加密强度高的令牌。

创建一个名为 `app/models/token.js` 的文件。它将包含 `Token` Mongoose 模式以及前面的方法：

```js
'use strict';

const EXPIRATION = 30; // in days
const LEN = 32;

const mongoose = require('mongoose');
const tokenHelper = require('../helpers/token');
const Schema = mongoose.Schema;
const ObjectId = Schema.ObjectId;

const TokenSchema = new Schema({
  user: {
    type: ObjectId,
    ref: 'User',
    required: true
  },
  hash: {
    type: String,
  },
  expiresAt: {
    type: Date,
    default: function() {
      var now = new Date();
      now.setDate(now.getDate() + EXPIRATION);

      return now;
    }
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});

TokenSchema.statics.generate = generateToken

function generateToken(opts, callback) {
  tokenHelper.generate(opts.tokenLength || LEN, (err, tokenString) => {
    if (err) {
      return callback(err);
    }

    opts.hash = tokenString;

    this.model('Token').create(opts, callback);
  });
};

// compile Token model
module.exports = mongoose.model('Token', TokenSchema); 
```

如您所见，我们为我们的令牌添加了一个过期日期。这可以用来在指定时间后自动使令牌失效。通常，在应用程序中，您不希望有没有过期日期的令牌。如果需要这样的令牌，应该添加另一层通过 API 密钥的授权，以授权第三方客户端使用系统。

## 使用 HTTP 基本认证进行认证

在生成令牌之前，我们需要对用户进行认证。一个简单的解决方案可能是使用简单的用户名和密码认证，如果输入的信息有效，则生成令牌。

我们可以公开一个处理 HTTP 基本认证的路由。这是强制执行资源访问控制的最简单技术。在我们的例子中，资源将是一个令牌，它不需要 cookies 或标识会话。HTTP 基本认证使用 HTTP 请求头中的标准字段。

此方法不会以任何方式添加加密或散列；只需要简单的 base64 编码即可。因此，它通常在 HTTPS 上使用。如果客户端想要向服务器发送用于认证的必要凭证，它可以使用 `Authorization` 头字段。

我们将使用 `passport-http` 模块来实现基本认证策略。让我们创建一个名为 `app/config/strategies/basic.js` 的文件，并添加以下代码行：

```js
'use strict';

const passport = require('passport');
const BasicStrategy = require('passport-http').BasicStrategy;
const mongoose = require('mongoose');
const User = mongoose.model('User');

module.exports.init = initBasicStrategy;

function initBasicStrategy() {
  passport.use('basic', new BasicStrategy((username, password, done) => {
    User.authenticate(username, password, (err, user) => {
      if (err) {
        return done(err);
      }

      if (!user) {
        return done(null, false);
      }

      return done(null, user);
    });
  }));
} 
```

该策略使用 `authenticate()` 方法来检查凭证是否有效。如您所见，我们在这里没有添加任何额外的逻辑。

接下来，我们将创建一个控制器来处理基本认证。创建一个名为 `app/controllers/authentication.js` 的文件，并添加以下内容：

```js
'use strict';

const _ = require('lodash');
const passport = require('passport');
const mongoose = require('mongoose');
const Token = mongoose.model('Token');

module.exports.basic = basicAuthentication;

function basicAuthentication(req, res, next) {
  passport.authenticate('basic', (err, user, info) => {
    if (err || !user) {
      return res.status(400).send({ message: 'Invalid email or password.' });
    }

    Token.generate({
      user: user.id
    }, (err, token) => {
      if (err || !token) {
        return res.status(400).send({ message: 'Invalid email or password.' });
      }

      var result = user.toJSON();
      result.token = _.pick(token, ['hash', 'expiresAt']);

      res.json(result);
    });

  })(req, res, next);
} 
```

Passport 有一个 `authenticate()` 方法，使我们能够调用一个给定的策略。我们使用自定义回调来生成并持久化 MongoDB 中的令牌。当将令牌返回给客户端时，我们只需要存储数据中的少数几个东西，例如值和过期日期。

## 添加身份验证路由

创建一个名为 `app/routes/authentication.js` 的文件，并添加以下代码行：

```js
'use strict';

const express = require('express');
const router = express.Router();
const authCtrl = require('../controllers/authentication');

router.post('/basic, authCtrl.basic);

module.exports = router;
```

`auth` 路由将允许用户使用基本策略进行 POST 调用并验证。为了创建可重用的路由，我们不会直接将路由挂载到 Express 应用实例上。相反，我们使用 `Router` 类来实例化一个新的路由器。

为了能够配置我们在 Express 应用程序上挂载哪些路由，我们可以创建一个名为 `config/routes.js` 的文件，包含以下代码行：

```js
'use strict';

module.exports.init = function(app) {
  var routesPath = app.get('root') + '/app/routes';

  app.use('/auth, require(routesPath + '/auth));
};
```

上述代码行应该很简单。我们正在定义路由的基本路径并将它们挂载到我们的应用程序上。需要注意的是，我们正在向身份验证路由添加一个前缀。

将以下高亮显示的代码添加到主 `server.js` 文件中，以初始化路由配置文件：

```js
require('./config/express').init(app);
require('./config/routes').init(app);

```

使用以下命令运行我们的身份验证测试：

```js
mocha tests/integration/authentication.test.js

```

这应该有一个类似的积极输出：

```js
 Authentication
 Basic authentication
 √ should authenticate a user and return a new token
 √ should not authenticate a user with invalid credentials
 2 passing

```

## 使用携带者身份验证验证用户

对于每个请求，应使用令牌来确定请求者是否有权访问系统。我们只有在用户发送有效凭据时才使用基本策略颁发令牌。Passport 有一个 `passport-http-bearer` 模块。通常，这用于保护 API 端点，就像在我们的案例中一样。令牌通常使用 OAuth 2.0 颁发，但，在我们的案例中，我们构建了一个自定义解决方案来颁发令牌。

在我们的案例中，令牌是系统向客户端颁发的访问授权键的字符串表示。客户端应用程序，Angular 应用程序，将使用访问令牌从 RESTful API 获取受保护资源。

让我们描述一个简单的用例，使用访问令牌检索信息。将以下代码行追加到 `tests/integration/authentication.test.js`，在基本身份验证测试套件之后：

```js
  describe('Bearer authentication', function() {
    var _token;

    before(function() {
      Token.generate({
        user: _user.id
      }, function(err, token) {
        if (err) throw err;

        _token = token;
        done();
      });
    });

    it('should authenticate a user using an access token', function(done) {
      request({
        method: 'GET',
        url: baseUrl + '/auth/info',
        auth: {
          bearer: _token.value
        },
        json:true
      }, function(err, res, body) {
        if (err) throw err;

        res.statusCode.should.equal(200);
        body.email.should.equal(userFixture.email);
        should.not.exist(body.password);
        should.not.exist(body.passwordSalt);
        done();
      });
    });

    it('should not authenticate a user with an invalid access token', function(done) {
      request({
        method: 'GET',
        url: baseUrl + '/auth/info',
        auth: {
          bearer: _token.value + 'a1e'
        },
        json:true
      }, function(err, res, body) {
        if (err) throw err;

        res.statusCode.should.equal(401);
        body.should.equal('Unauthorized');
        done();
      });
    });
  });
```

我们假设存在一个 `/auth/info` 路由，如果进行 GET 调用，它将返回令牌的所有者凭据。如果令牌无效，将返回一个带有适当 401 HTTP 状态码的未授权消息。

### 携带者策略

让我们创建一个名为 `config/strategies/bearer.js` 的文件。添加以下代码段：

```js
'use strict';

const passport = require('passport');
const BearerStrategy = require('passport-http-bearer').Strategy;
const mongoose = require('mongoose');
const Token = mongoose.model('Token');

module.exports.init = initBearerStrategy;

function initBearerStrategy() {
  passport.use('bearer', new BearerStrategy((token, done) => {
    Token
    .findOne({ hash: token })
    .populate('user')
    .exec((err, result) => {
      if (err) {
        return done(err);
      }

      if (!result) {
        return done(null, false, { message: 'Unauthorized.' });
      }

      if (!result.user) {
        return done(null, false, { message: 'Unauthorized.' });
      }

      done(null, result.user);
    });
  }));
} 
```

上述代码在数据库中搜索给定的令牌。为了检索令牌所有者，我们可以使用 Mongoose 的 `populate()` 方法结合一个常规查询方法，例如 `findOne()`。这是因为我们在 Token 模型中明确添加了对 User 模型的引用。

### 使用令牌保护资源

为了保护我们的资源，我们需要添加一个检查访问令牌存在的层。我们已经完成了 Bearer 策略的第一部分。现在我们只需要使用它；为此，我们可以创建一个中间件来验证令牌。

创建一个名为`app/middlewares/authentication.js`的新文件，并添加以下代码：

```js
'use strict';

const passport = require('passport');

module.exports.bearer = function bearerAuthentication(req, res, next) {
  return passport.authenticate('bearer', { session: false });
};
```

上述代码相当简单。我们只是使用 passport 的内置`authenticate()`方法调用 Bearer 策略。我们不想在服务器上保存任何会话。这个中间件可以与任何其他应用程序逻辑结合使用，应用于每个路由。

将以下代码行追加到`app/controllers/authentication.js`。它将仅检查用户是否存在于请求对象中，并返回包含数据的 JSON：

```js
module.exports.getAuthUser = getAuthUser;

function getAuthUser(req, res, next) {
  if (!req.user) {
    res.status(401).json({ message: 'Unauthorized.' });
  }

  res.json(req.user);
} 
```

现在，让我们回到我们的身份验证路由`app/routes/authentication.js`，并添加以下突出显示的代码行：

```js
'use strict';

var express = require('express');
var router = express.Router();
var authCtrl = require('../controllers/authentication');
var auth = require('../middlewares/authentication');

router.post('/basic', authCtrl.basic);
router.get('/info', auth.bearer(), authCtrl.getAuthUser);

module.exports = router;
```

我们在控制器逻辑执行之前添加了身份验证中间件，以验证和检索令牌的所有者。我们的 Bearer 策略将处理这个问题，并在请求对象上设置用户；更确切地说，它可以在`req.user`上找到。

如果我们运行我们的身份验证测试：

```js
mocha tests/integration/authentication.test.js

```

应该打印以下输出：

```js
 Authentication
 Basic authentication
 √ should authenticate a user and return a new token
 √ should not authenticate a user with invalid credentials 
 Bearer authentication
 √ should authenticate a user using an access token
 √ should not authenticate a user with an invalid access token

 4 passing

```

通过这种方式，我们最终添加了所有必要的身份验证方法，以授予用户访问我们的系统。

# 跟踪支出

我们应用程序的主要功能是跟踪用户的支出。用户应该能够插入支出，使其在系统中持久化，并查看其账户的确切余额。

总是应该有一个清晰的视图来了解想要实现的目标。让我们从高层次的角度看看我们想要实现的目标：

+   用户应该能够在系统中持久化一笔支出

+   用户应该能够获取其所有的支出

+   用户应该能够获取其支出的余额

+   用户应该能够定义一个类别来保存支出，例如，杂货

## 货币价值

在我们的案例中，一笔支出将存储花费的确切货币价值。在某些情况下，处理货币数据可能会变得复杂。通常，处理货币数据的应用程序需要处理货币的分数单位。

我们可以将数据存储为浮点数。然而，在 JavaScript 中，浮点运算通常不符合货币运算。换句话说，像三分之一和十分之一这样的值在二进制浮点数中没有确切的表示。

例如，MongoDB 将数值数据存储为 IEEE 754 标准 64 位浮点数、32 位或 64 位有符号整数。JavaScript 根据规范将数字视为双精度 64 位格式 IEEE 754 值。因此，我们需要注意以下操作：

```js
+ 0.2 = 0.30000000000000004
```

我们将无法存储像 9.99 美元这样的值，它代表小数点后的美分。请别误会；我们可以存储它们，但如果使用内置的 MongoDB 聚合框架或进行服务器端算术（同样适用于 JavaScript 客户端）时，我们将无法得到正确的结果。

不要担心；我们有几种解决方案可以使用。在 MongoDB 中存储货币值有两种常见的方法：

+   精确精度是一种将货币值乘以 10 的幂的方法。

+   相反，任意精度使用两个字段来表示货币值。一个字段以非数字格式（如字符串）存储精确值，另一个字段存储值的浮点近似值。

对于我们的实现，我们将使用精确精度模型。随着代码的进展，我们将讨论所有细节。

## 类别模型

正如我们之前讨论的，我们希望能够向特定类别添加支出。用户还应该能够邀请其他用户向类别添加支出。我们不会详细说明此功能的测试用例，但你应该考虑编写测试以确保一切按预期工作。

让我们创建一个名为`app/models/category.js`的文件，并添加以下代码行：

```js
'use strict';

const mongoose = require('mongoose');
const Schema = mongoose.Schema;
const ObjectId = Schema.ObjectId;

const CategorySchema = new Schema({
  name: {
    type: String,
    required: true
  },
  description: {
    type: String
  },
  owner: {
    type: ObjectId,
    ref: 'User',
    required: true
  },
  collaborators: {
    type: [
      {
        type: ObjectId,
        ref: 'User'
      }
    ]
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});

// compile Category model
module.exports = mongoose.model('Category', CategorySchema);
```

这里有两个重要事项需要注意：我们定义了类别的所有者，它始终是创建类别的认证用户，以及协作者字段，它包含可以插入类别支出的用户。

此外，别忘了通过添加以下突出显示的代码来更改模型配置文件`config/models.js`：

```js
['user', 'token', 'category', 'expense'].forEach(function(model) {
    require(modelsPath + model);
});
```

## 类别路由

为了在类别集合上公开简单的 CRUD 操作，我们必须为这些操作定义路由。为此，我们将创建一个名为`app/routes/categories.js`的路由器文件，并添加以下代码行：

```js
'use strict';

const express = require('express');
const router = express.Router();
const categoryCtrl = require('../controllers/category');
const auth = require('../middlewares/authentication');

router.param('categoryId', expenseCtrl.findById);

router.get('/categories', auth.bearer(), categoryCtrl.getAll);
router.get('/categories/:categoryId', auth.bearer(), categoryCtrl.getOne);
router.post('/categories', auth.bearer(), categoryCtrl.create);
router.put('/categories/:categoryId', auth.bearer(), categoryCtrl.update);
router.delete('/categories/:categoryId', auth.bearer(), categoryCtrl.delete);

module.exports = router;
```

请记住，我们目前实际上没有实现类别控制器。让我们创建一个名为`app/controllers/category.js`的类别控制器。

### 通过 ID 获取类别

将以下代码行添加到`app/controllers/category.js`中：

```js
'use strict';

const _ = require('lodash');
const mongoose = require('mongoose');
const Category = mongoose.model('Category');
const ObjectId = mongoose.Types.ObjectId;

module.exports.findById = findCategoryById;
module.exports.create = createCategory;
module.exports.getOne = getOneCategory;
module.exports.getAll = getAllCategories;
module.exports.update = updateCategory;
module.exports.delete = deleteCategory;

function findCategoryById(req, res, next, id) {
  if (!ObjectId.isValid(id)) {
    return res.status(404).json({ message: 'Not found.'});
  }

  Category.findById(id, (err, category) => {
    if (err) {
      return next(err);
    }

    if (!category) {
      return res.status(404).json({ message: 'Not found.'});
    }

    req.category = category;
    next();
  });
} 
```

当`categoryId`路由`param`存在时，前面的代码将非常有用。它将自动获取一个类别，正如我们在路由文件中定义的那样。

### 创建类别

要创建一个类别，将以下代码行追加到控制器文件中：

```js
function createCategory(req, res, next) {
  const data = req.body;
  data.owner = req.user.id;

  Category.create(data, (err, category) => {
    if (err) {
      return next(err);
    }

    res.status(201).json(category);
  });
} 
```

在创建类别之前，我们添加所有者的 ID，即当前用户的 ID。

### 获取一个和所有类别

我们还希望获取单个类别和所有类别。要获取一个类别，我们将使用通过 ID 获取类别的结果。要检索多个类别，我们将使用 Mongoose 的`find()`查询方法。我们可以轻松添加分页或设置限制，但我们将假设用户不会有这么多类别。这可能是我们应用程序的一个小改进。

将以下代码行添加到控制器中：

```js
function getOneCategory(req, res, next) {
  res.json(req.category);
}

function getAllCategories(req, res, next) {
  Category.find((err, categories) => {
    if (err) {
      return next(err);
    }

    res.json(categories);
  });
} 
```

### 更新和删除分类

当我们通过 ID 获取一个分类时，我们将 Mongoose 返回的实例设置到请求对象中。由于这个原因，我们可以使用该实例来更改其属性并将其保存回 Mongo。添加以下代码：

```js
function updateCategory(req, res, next) {
  const category = req.category;
  const data = _.pick(req.body, ['description', 'name']);
  _.assign(category, data);

  category.save((err, updatedCategory) => {
    if (err) {
      return next(err);
    }

    res.json(updatedCategory);
  });
}
```

删除分类时也可以使用相同的方法；也请添加以下代码行：

```js
function deleteCategory(req, res, next) {
  req.category.remove((err) => {
    if (err) {
      return next(err);
    }

    res.status(204).json();
  });
} 
```

使用前面的代码行，我们已经完成了对分类的 CRUD 操作。

## 定义费用模型

之前，我们讨论了这样一个事实，即我们不能简单地将货币数据作为浮点数存储在数据库中或用于服务器端算术。我们场景的解决方案是使用精确精度来存储货币数据。换句话说，货币值将通过将初始值乘以 10 的幂来存储。

我们将假设所需的最高精度为十分之一分。根据这个假设，我们将初始值乘以 1000。例如，如果我们有一个初始值为 9.99 美元，数据库中存储的值将是 9990。

对于当前应用程序的实现，我们将使用美元（USD）作为货币值。缩放因子将为 1000，以保留到十分之一分的精度。使用精确精度模型，缩放因子需要在应用程序中保持一致，并且任何给定时间都应该从货币中确定。

让我们创建我们的费用模型，`app/models/expense.js`，并添加以下代码行：

```js
'use strict';

const CURRENCY = 'USD';
const SCALE_FACTOR = 1000;

const mongoose = require('mongoose');
const Schema = mongoose.Schema;
const ObjectId = Schema.ObjectId;

const ExpenseSchema = new Schema({
  name: {
    type: String
  },
  amount: {
    type: Number,
    default: 0
  },
  currency: {
    type: String,
    default: CURRENCY
  },
  scaleFactor: {
    type: Number,
    default: SCALE_FACTOR
  },
  user: {
    type: ObjectId,
    ref: 'User',
    required: true
  },
  category: {
    type: ObjectId,
    ref: 'Category',
    required: true
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
}, {
  toObject: {
    virtuals: true
  },
  toJSON: {
    virtuals: true
  }
});

module.exports = mongoose.model('Expense', ExpenseSchema); 
```

以下表格将简要描述模式中的字段：

| 字段 | 描述 |
| --- | --- |
| `name` | 费用的名称 |
| `amount` | 货币的缩放金额 |
| `currency` | 用于表示货币的货币 |
| `scaleFactor` | 用于获取金额的缩放因子 |
| `user` | 费用所属的用户 |
| `category` | 费用所属的分类组 |
| `createdAt` | 费用对象创建的日期 |

Mongoose 有一个有趣的功能，称为 `虚拟属性`。这些属性不会持久化到数据库中，但在许多场景中非常有用。我们将使用一个名为 `value` 的虚拟属性，它将表示 `amount` 属性的货币价值。

在模型编译之前添加以下代码行：

```js
ExpenseSchema.virtual('value')
.set(function(value) {
  if (value) {
    this.set('amount', value * this.scaleFactor);
  }
})
.get(function() {
  return this.amount / this.scaleFactor;
});
```

就像所有属性一样，虚拟属性可以有 `getters` 和 `setters`。我们将利用 `setter` 并添加自己的逻辑，这将根据给定的因子缩放值并获取所需的金额。此外，当获取虚拟的 `value` 属性时，我们将返回正确的货币表示形式，通过将存储的金额除以相应的缩放因子。

默认情况下，当进行查询时，Mongoose 不会返回 `virtual attributes`，但我们已覆盖了模式默认选项，以便在使用 `.toJSON()` 和 `.toObject()` 方法时返回所有 `virtual attributes`。

## 描述费用模块功能

接下来，我们将为费用模块编写一些测试用例，以定义模块所需的行为。

### 提示

为了加快速度，我们只定义几个测试用例。其余的 CRUD 测试用例与早期实现中不同模块的相同。为了参考，您可以在以下链接查看测试套件的完整代码库：[`www.packtpub.com/`](https://www.packtpub.com/).

让我们创建一个名为 `tests/integration/expense.test.js` 的文件。我们将定义最重要的测试用例：

1.  在创建费用时，必须存在一个值和一个类别。该值应该是一个可以接受小数的数字：

    ```js
        it('should save an expense', function(done) {
          request({
            method: 'POST',
            url: baseUrl + '/expenses',
            auth: {
              bearer: _token.value
            },
            form: {
              value: 14.99,
              category: _category.toString()
            },
            json:true
          }, function(err, res, body) {
            if (err) throw err;

            res.statusCode.should.equal(201);
            body.amount.should.equal(14990);
            body.scaleFactor.should.equal(1000);
            body.value.should.equal(14.99);
            body.category.should.equal(_category.toString());
            done();
          });
        });
    ```

1.  我们应该能够从数据库中获取所有用户的费用：

    ```js
        it('should get balance for all expenses', function(done) {
          request({
            method: 'GET',
            url: baseUrl + '/expenses/balance',
            auth: {
              bearer: _token.value
            },
            json:true
          }, function(err, res, body) {
            if (err) throw err;

            res.statusCode.should.equal(200);
            should.exist(body);
            body.balance.should.equal(33.33);
            body.count.should.equal(3);
            done();
          });
        });
    ```

1.  如果需要，我们应该只获取给定类别的费用。当我们要显示特定类别的费用时，这会很有用：

    ```js
        it('should get expenses balance only for a category', function(done) {
          request({
            method: 'GET',
            url: baseUrl + '/expenses/balance?category=' + _categoryOne.toString(),
            auth: {
              bearer: _token.value
            },
            json:true
          }, function(err, res, body) {
            if (err) throw err;

            res.statusCode.should.equal(200);
            should.exist(body);
            body.balance.should.equal(21.21);
            body.count.should.equal(2);
            done();
          });
        });
    ```

上述代码测试了费用的创建以及虚拟值属性是否正确工作。它还检查是否发送了无效的令牌，并且应用程序将相应地处理它。现在，有趣的部分开始了，即 `balance` 功能，它应该返回不同场景下费用的聚合值。

## 费用的 CRUD 操作

接下来，我们将逐一实现费用的 CRUD 操作。在继续之前，我们将创建一个新的路由文件名为 `app/routes/expenses.js` 并添加以下代码行：

```js
'use strict';

const express = require('express');
const router = express.Router();
const expenseCtrl = require('../controllers/expense');
const auth = require('../middlewares/authentication');

router.param('expenseId', expenseCtrl.findById);

router.get('/expenses', auth.bearer(), expenseCtrl.getAll);
router.get('/expenses/:expenseId', auth.bearer(), expenseCtrl.getOne);
router.post('/expenses', auth.bearer(), expenseCtrl.create);
router.put('/expenses/:expenseId', auth.bearer(), expenseCtrl.update);
router.delete('/expenses/:expenseId', auth.bearer(), expenseCtrl.delete);

module.exports = router;
```

我们为每个路由添加了 bearer 认证。您本可以创建一个单独的路由来捕获所有需要认证的资源，但这样，您将能够对每个路由进行更精细的控制。

### 创建费用

让我们创建路由文件所需的控制器——`app/controllers/expense.js`——并添加创建费用的逻辑：

```js
'use strict';

const _ = require('lodash');
const mongoose = require('mongoose');
const Expense = mongoose.model('Expense');
const ObjectId = mongoose.Types.ObjectId;

module.exports.create = createExpense;
module.exports.findById = findExpenseById
module.exports.getOne = getOneExpense;
module.exports.getAll = getAllExpenses;
module.exports.update = updateExpense;
module.exports.delete = deleteExpense;
module.exports.getBalance = getExpensesBalance;

function createExpense(req, res, next) {
  const data = _.pick(req.body, ['name', 'value', 'category', 'createdAt']);
  data.user = req.user.id;

  if (data.createdAt === null) {
    delete data.createdAt;
  }

  Expense.create(data, (err, expense) => {
    if (err) {
      return next(err);
    }

    res.status(201).json(expense);
  });
} 
```

我们想要创建的费用应该是令牌所有者的。因此，我们明确地将用户属性设置为认证用户的 ID。

### 通过 ID 获取费用

获取单个费用和更新费用的逻辑使用费用实例来显示或更新它。因此，我们只添加一个通过 ID 获取费用的逻辑。将以下代码行追加到控制器文件中：

```js
function findExpenseById(req, res, next, id) {
  if (!ObjectId.isValid(id)) {
    return res.status(404).json({ message: 'Not found.'});
  }

  Expense.findById(id, (err, expense) => {
    if (err) {
      return next(err);
    }

    if (!expense) {
      return res.status(404).json({ message: 'Not found.'});
    }

    req.expense = expense;
    next();
  });
} 
```

因为在这里我们不会进行最终操作，所以我们只设置费用在请求对象中存在，并调用路由管道中的下一个处理程序。

### 获取单个费用

我们将扩展“通过 ID 获取费用”并仅响应资源的 JSON 表示。获取费用的逻辑应该是追加到控制器文件中的几行代码：

```js
function getOneExpense(req, res, next) {
  if (!req.expense) {
    return res.status(404).json({ message: 'Not found.'});
  }

  res.json(req.expense);
}
```

### 获取所有费用

当获取所有支出时，我们需要采取不同的方法——一种能够使我们能够通过特定查询过滤它们的方法。支出还应返回特定类别的支出。我们不需要为所有这些场景实现不同的搜索逻辑。相反，我们可以创建一个将围绕我们的需求：

```js
function getAllExpenses(req, res, next) {
  const limit = +req.query.limit || 30;
  const skip = +req.query.skip || 0;
  const query = {};

  if (req.category) {
    query.category = req.category.id;
  } else {
    query.user = req.user.id;
  }

  if (req.query.startDate) {
    query.createdAt = query.createdAt || {};
    query.createdAt.$gte = new Date(req.query.startDate);
  }

  if (req.query.endDate) {
    query.createdAt = query.createdAt || {};
    query.createdAt.$lte = new Date(req.query.endDate);
  }

  if (req.query.category) {
    query.category = req.query.category;
  }

  Expense
  .find(query)
  .limit(limit)
  .skip(skip)
  .sort({ createdAt: 'desc' })
  .populate('category')
  .exec((err, expenses) => {
    if (err) {
      return next(err);
    }

    res.json(expenses);
  });
}  
```

在使用 Mongoose 查询数据库以检索必要数据之前，我们构建一个查询变量，该变量将包含所有我们的标准。这里值得注意的一个好现象是，我们再次使用了 Mongoose 提供的查询构建器对象。支出在 MongoDB 中将以更大的数量存储。因此，我们添加了`limit`和`skip`来检索有限的数据集。

可以使用日期范围查询支出。由于这个原因，`createdAt`属性将逐步扩展以匹配特定时间段内的支出集合。支出还应按时间顺序返回；新添加的支出应首先返回。

为了获取每个支出的所有必要信息，我们将使用数据库中的适当类别对象来填充支出的类别属性。

### 更新支出

将以下更新逻辑的代码添加到控制器文件中：

```js
function updateExpense(req, res, next) {
  const data = _.pick(req.body, ['name', 'value', 'category', 'createdAt']);
  const expense = req.expense;

  if (data.createdAt === null) {
    delete data.createdAt;
  }

  _.assign(expense, data);

  expense.save((err, updatedExpense) => {
    if (err) {
      return next(err);
    }

    res.json(updatedExpense);
  });
} 
```

更新逻辑使用回调触发器为支出 ID 参数设置的请求对象上的支出实例。

### 删除支出

为了删除支出，我们只需使用以下代码从数据库中删除支出实例：

```js
function deleteExpense(req, res, next) {
  req.expense.remove((err) => {
    if (err) {
      return next(err);
    }

    res.status(204).json();
  });
}
```

## 获取支出余额

让我们回到支出模型，并使用余额计算来扩展它。为了在不同场景下获取余额，我们将使用 MongoDB 的聚合框架。聚合数据意味着从集合中的数据操作得到的计算结果。

Mongo 提供了一套复杂的操作来对数据集进行操作。因为我们使用 Mongoose，所以我们有权访问`Model.aggregate()`，这将帮助我们创建聚合管道。

请记住，聚合返回的数据是以纯 JavaScript 对象的形式，而不是 Mongoose 文档。这是由于在使用聚合时，可以返回任何形状的文档。

在编译支出模型之前添加以下代码：

```js
ExpenseSchema.statics.getBalance = getExpensesBalance;

function getExpensesBalance(opts, callback) {
  const query = {};

  // set the current user
  query.user = opts.user;

  if (opts.category || opts.category === null) {
    query.category = new mongoose.Types.ObjectId(opts.category);
  }

  if (opts.startDate && opts.endDate) {
    query.createdAt = {
      $gte: new Date(opts.startDate),
      $lte: new Date(opts.endDate)
    };
  }

  this.model('Expense').aggregate([
    { $match: query },
    { $group: { _id: null, balance: { $sum: '$amount' }, count: { $sum: 1 } } }
  ], (err, result) => {

    // result is an array with a single item, we can just return that
    const final = result[0];
    final.balance = final.balance / SCALE_FACTOR;

    callback(err, final);
  });
}
```

上述静态`.getBalance()`方法将根据测试用例中描述的不同场景计算当前余额。`.aggregate()`方法会经过多个阶段。第一个阶段是匹配阶段，它将选择所有符合我们定义查询的文档。匹配的结果被发送到分组阶段，在这里，文档根据指定的标识符进行分组。

此外，管道阶段可以使用运算符执行不同的任务，例如，在我们的场景中计算余额。我们使用一个累加运算符`$sum`，它为每个组返回一个数值。

在分组阶段，`_id`字段是必需的，但你可以为它指定一个 null 值来计算管道输入文档的所有值。分组操作器的 RAM 限制为 100 兆字节，但你可以将它设置为使用磁盘来写入临时文件。要设置此选项，请使用 Mongoose 并查看`.allowDiskUse()`方法。

添加缺失的控制器函数，`app/controller/expense`：

```js
function getExpensesBalance(req, res, next) {
  Expense.getBalance({
    user: req.user._id,
    category: req.query.category,
    startDate: req.query.start,
    endDate: req.query.end
  }, (err, result) => {
    if (err) {
      return next(err);
    }

    res.json(result);
  });
}
```

# 实现 Angular 客户端应用程序

在我们的项目中，我们已经到达了开始集成 AngularJS 应用程序的阶段。本章将采用不同的方法来构建所需的应用程序。理想的应用程序应该以模块化的方式构建，每个模块处理特定的功能。

当你构建 Angular 应用程序时，你可能已经熟悉了基于组件的方法。这意味着我们将创建小的模块，这些模块封装了特定的功能。这使得我们可以增量地添加功能；想象一下向应用程序添加垂直块。

为了使这生效，我们需要创建一个主要块来将所有东西粘合在一起，将所有功能和模块拉在一起。保持你的主应用程序模块轻薄，并将其余逻辑移动到应用程序模块中。

我喜欢遵循的一条规则是尽可能保持我的文件夹结构扁平。我总是试图降低文件夹的层级，以便我可以快速定位代码和功能。如果你的模块变得太大，你可以将其拆分或添加子文件夹。

## 项目初始化

让我们开始创建一个`public/package.json`文件。我们将使用`npm`来安装项目前端部分的依赖项。`package.json`文件将包含以下内容：

```js
{
  "private": true,
  "name": "mean-blueprints-expensetracker-client",
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

运行此命令以安装所有依赖项：

```js
npm install

```

安装成功后，创建一个名为`public/src`的文件夹。这个文件夹将包含主要的 Angular 应用程序。在这个文件夹内，我们将创建我们的模块文件夹和应用文件。

创建你的主应用程序组件文件，命名为`public/src/app.component.ts`，并按照以下步骤创建文件的最终版本：

1.  添加必要的依赖项：

    ```js
    import { Component, OnInit } from 'angular2/core';
    import { RouteConfig, RouterOutlet, RouterLink } from 'angular2/router';
    import { Router } from 'angular2/router';
    import { AuthHttp, AuthService, SigninComponent, RegisterComponent } from './auth/index';
    import { ExpensesComponent } from './expense/index';
    import { CategoriesComponent } from './expense/index';
    ```

1.  配置你的路由：

    ```js
    @RouteConfig([
      { path: '/', redirectTo: ['/Expenses'], useAsDefault: true },
      { path: '/expenses', as: 'Expenses', component: ExpensesComponent },
      { path: '/categories', as: 'Categories', component: CategoriesComponent },
      { path: '/signin', as: 'Signin', component: SigninComponent },
      { path: '/register', as: 'Register', component: RegisterComponent }
    ])
    ```

    我们定义了一个默认路径，该路径将重定向到`expenses`视图，显示所有条目给用户。还有一个`Signin`和`register`路由可用。

1.  添加组件注解：

    ```js
    @Component({
        selector: 'expense-tracker',
        directives: [
          RouterOutlet,
          RouterLink
        ],
        template: `
          <div class="app-wrapper card whiteframe-z2">
            <div class="row">
              <div class="col">
                <a href="#">Expense tracker</a>
                <a href="#" [routerLink]="['Expenses']">Expenses</a>
              </div>
            </div>
            <div class="row">
              <router-outlet></router-outlet>
            </div>
          </div>
        `
    })
    ```

1.  定义组件的类：

    ```js
    export class AppComponent implements OnInit {
      public currentUser: any;
      private _authHttp: AuthHttp;
      private _authSerivce: AuthService;
      private _router: Router;

      constructor(authHttp: AuthHttp, authSerice: AuthService, router: Router) {
        this._router = router;
        this._authSerivce = authSerice;
        this._authHttp = authHttp;
      }

      ngOnInit() {
        this.currentUser = {};
        this._authHttp.unauthorized.subscribe((res) => {
          if (res) {
            this._router.navigate(['./Signin']);
          }
        });
        this._authSerivce.currentUser.subscribe((user) => {
          this.currentUser = user;
        });
      }
    }
    ```

如果发生未经授权的调用，我们将用户重定向到`Signin`路由，以便使用有效凭据进行身份验证。

# 注册用户

我们的应用程序应该支持用户注册。我们已经有这个功能的后端逻辑。现在，我们只需要将其与我们的 Angular 应用程序结合起来。为此，我们将创建一个名为`auth`的通用模块，该模块将用于注册和验证用户。

## 认证服务

我们将继续使用`auth`服务，该服务将包含与 Node.js 后端应用程序的所有通信逻辑。创建一个名为`public/src/auth/services/auth.service.ts`的文件，并按照以下步骤实现服务的整个逻辑：

1.  导入依赖项：

    ```js
    import { Injectable } from 'angular2/core';
    import { Http, Response, Headers } from 'angular2/http';
    import { Subject } from 'rxjs/Subject';
    import { BehaviorSubject } from 'rxjs/Subject/BehaviorSubject';
    import { contentHeaders } from '../../common/index';
    ```

1.  定义服务类：

    ```js
    @Injectable()
    export class AuthService {
      public currentUser: Subject<any>;
      private _http: Http;

      constructor(http: Http) {
        this._http = http;
        this._initSession();
      }
    }
    ```

1.  添加`signin()`方法：

    ```js
      public signin(user: any) {
        let body = this._serialize(user);
        let basic = btoa(`${user.email}:${user.password}`);
        let headers = new Headers(contentHeaders);
        headers.append('Authorization', `Basic ${basic}`)

        return this._http
        .post('/auth/basic', '', { headers: headers })
        .map((res: Response) => res.json());
      }
    Append the register() method:
      public register(user: any) {
        let body = this._serialize(user);

        return this._http
        .post('/api/users', body, { headers: contentHeaders })
        .map((res: Response) => res.json());
      }
    ```

1.  设置当前用户：

    ```js
      public setCurrentUser(user: any) {
        this.currentUser.next(user);
      }
    ```

    我们希望公开一个简单的函数来设置`currentUser`可观察对象的下一个值。

1.  初始化会话：

    ```js
      private _initSession() {
        let user = this._deserialize(localStorage.getItem('currentUser'));
        this.currentUser = new BehaviorSubject<Response>(user);
        // persist the user to the local storage
        this.currentUser.subscribe((user) => {
          localStorage.setItem('currentUser', this._serialize(user));
          localStorage.setItem('token', user.token.hash || '');
        });
      }
    ```

    当应用程序重新加载时，我们希望从本地存储中检索当前用户以恢复会话。你可以添加的一个改进是检查令牌是否已过期。

1.  添加辅助方法：

    ```js
      private _serialize(data) {
        return JSON.stringify(data);
      }

      private _deserialize(str) {
        try {
          return JSON.parse(str);
        } catch(err) {
          console.error(err);
          return null;
        }
      }
    ```

前面的函数是`stringify`和`parse` JSON 方法的简单抽象。

## 寄存器组件

创建适当的组件文件，`public/src/auth/components/register.component.ts`，其中包含以下代码行：

```js
import { Component } from 'angular2/core';
import { Router, RouterLink } from 'angular2/router';
import { AuthService } from '../services/auth.service';

export class RegisterComponent {
  private _authService: AuthService;
  private _router: Router;

  constructor(authService: AuthService, router: Router) {
    this._router = router;
    this._authService = authService;
  }

  register(event, name, email, password) {
    event.preventDefault();

    let data = { name, email, password };

    this._authService
    .register(data)
    .subscribe((user) => {
      this._router.navigateByUrl('/');
    }, err => console.error(err));
  }
}
```

当调用`register`方法时，我们只是尝试使用`AuthService`注册我们的用户。前面的代码中没有添加错误处理。只会在浏览器的控制台中打印一个简单的日志。让我们添加模板：

```js
@Component({
    selector: 'register',
    directives: [
      RouterLink
    ],
    template: `
      <div class="login jumbotron center-block">
        <h1>Register</h1>
        <form role="form" (submit)="register($event, name.value, email.value, password.value)">
          <div class="form-group">
            <label for="name">Full name</label>
            <input type="text" #name class="form-control" id="email" placeholder="please enter your name">
          </div>
          <div class="form-group">
            <label for="email">E-mail</label>
            <input type="text" #email class="form-control" id="email" placeholder="enter valid e-mail">
          </div>
          <div class="form-group">
            <label for="password">Password</label>
            <input type="password" #password class="form-control" id="password" placeholder="now your password">
          </div>
          <button type="submit" class="button">Submit</button>
        </form>
      </div>
    `
})
```

寄存器组件相当简单。我们正在定义一个简单的寄存器函数，该函数将使用认证服务的`register`方法。所有必要的字段也可以在`template`属性中找到。

# 登录用户组件

为了认证用户，我们在认证服务中添加了一些额外的功能，使我们能够登录用户。因为我们没有在后端持久化用户的会话状态——换句话说，我们的后端是无状态的——我们必须在前端存储用户的当前状态。

记住我们创建了一个端点，该端点将为我们提供一个有效的用户名和密码对的令牌。我们将使用该端点来检索一个令牌，该令牌将使我们能够访问 API 的其余端点。

我们的登录组件相当简单，并且确实是从上一章中复用的，但让我们刷新一下记忆并看看它。`SigninComponent`位于`public/src/auth/components/signin.component.ts`：

```js
import { Component } from 'angular2/core';
import { Router, RouterLink } from 'angular2/router';
import { AuthService } from '../services/auth.service';

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
        </form>
      </div>
    `
})
export class SigninComponent {
  private _authService: AuthService;
  private _router: Router;

  constructor(authService: AuthService, router: Router) {
    this._authService = authService;
    this._router = router;
  }

  signin(event, email, password) {
    event.preventDefault();

    let data = { email, password };

    this._authService
    .signin(data)
    .subscribe((user) => {
      this._authService.setCurrentUser(user);
      this._router.navigateByUrl('/');
    }, err => console.error(err));
  }
}
```

就像在`RegisterComponent`中一样，我们为我们的字段使用局部变量。使用`AuthService`，我们尝试认证我们的用户。我们并不真正关注错误处理，但如果用户成功认证，我们希望导航到`root`路径并设置当前用户。

# 常用功能

在进一步开发之前，我们需要考虑一些之前使用过的功能和一些额外的功能。例如，我们使用了一个常见的头定义，位于`public/src/common/headers.ts`下：

```js
import { Headers } from 'angular2/http';

const HEADERS = {
  'Content-Type': 'application/json',
  'Accept': 'application/json'
};

export const contentHeaders = new Headers(HEADERS);
```

这仅仅是一种定义常量并在整个应用程序中使用它们而不重复的方法。所以，基本上，我们导入了 Angular 2 中的`Headers`并创建了一个新实例。你可以很容易地使用`append()`方法向这个头实例添加额外的字段，例如：

```js
contentHeaders.append('Authorization', 'Bearer <token_value>');
```

现在还有一些其他事情需要考虑：

+   当通过 API 向服务器请求资源时，我们应该发送所需的 Bearer 令牌

+   如果用户发出调用，并且服务器以状态码 401（未经授权）响应，我们应该将用户重定向到登录页面

让我们看看我们能对前面的列表做些什么。

## 自定义 HTTP 服务

在我们创建自定义 HTTP 服务以调用 Express 后端应用程序时，我们在上一章中做了类似的事情。但是我们需要一些额外的东西，比如将令牌附加到通过此服务发出的每个调用中，以识别用户。

记住我们将在浏览器的`LocalStorage`中存储用户的令牌。这应该相当简单，我认为我们甚至可以在服务中添加它。让我们开始，创建一个名为`public/src/auth/services/auth-http.ts`的新文件：

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

  public get(url: string, opts?: RequestOptionsArgs) {
    return this.request({ url: url, method: RequestMethod.Get}, opts);
  }

  public post(url: string, body?: string, opts?: RequestOptionsArgs) {
    return this.request({ url: url, method: RequestMethod.Post, body: body}, opts);
  }

  public put(url: string, body?: string, opts?: RequestOptionsArgs) {
    return this.request({ url: url, method: RequestMethod.Put, body: body}, opts);
  }

  public delete(url: string, body?: string, opts?: RequestOptionsArgs) {
    return this.request({ url: url, method: RequestMethod.Delete, body: body}, opts);
  }

  // rest of the HTTP methods ...
}
```

因此，这是我们自定义的`HttpAuth`服务，它公开了一些公共方法，与上一章相同。现在变化发生在私有的`request()`方法中：

```js
  private request(requestArgs: RequestOptionsArgs, additionalArgs?: RequestOptionsArgs) {
    let opts = new RequestOptions(requestArgs);

    if (additionalArgs) {
      opts = opts.merge(additionalArgs);
    }

    let req:Request = new Request(opts);

    if (!req.headers) {
      req.headers = new Headers();
    }

    if (!req.headers.has('Authorization')) {
      req.headers.append('Authorization', `Bearer ${this.getToken()}`);
    }

    return this._http.request(req).catch((err: any) => {
      if (err.status === 401) {
        this.unauthorized.next(err);
      }

      return Observable.throw(err);
    });
  }
```

在我们发出调用之前，我们将必要的令牌附加到`Authorization`头。令牌存储在浏览器的存储中，因此我们使用`getToken()`方法来检索它。如果请求未经授权，我们将它推送到我们的未经授权数据流中，该数据流包含失败的认证请求。

`getToken()`方法有一个非常简单的实现：

```js
  private getToken() {
    return localStorage.getItem('token');
  }
```

## 使用单个导出文件

我们可以在每个模块文件夹的根目录中添加一个`index.ts`文件，以便导出所有公共成员。在`auth`模块中，我们可以有一个名为`public/src/auth/index.ts`的文件，其内容如下：

```js
export * from './components/register.component';
export * from './components/signin.component';
export * from './services/auth.service';
export * from './services/auth-http';
```

这种技术将用于每个模块，并且不会进一步介绍。

# 类别模块

类别模块将包含执行类别 CRUD 操作和通过 Angular 服务与后端通信所需的所有逻辑。

## 类别服务

类别服务将相对简单，它只将管理类别的 CRUD 操作。以下步骤将描述实现此过程的方法：

1.  创建一个名为`public/app/categories/category.service.js`的文件。

1.  添加必要的业务逻辑：

    ```js
    import { Injectable } from 'angular2/core';
    import { Http, Response, Headers } from 'angular2/http';
    import { Observable } from 'rxjs/Observable';
    import { Subject } from 'rxjs/Subject';
    import { BehaviorSubject } from 'rxjs/Subject/BehaviorSubject';
    import { AuthHttp } from '../auth/index';
    import { contentHeaders } from '../common/index';
    import { Category } from './category.model';

    @Injectable()
    export class CategoryService {
      public category: Subject<Category>;
      public categories: Observable<Array<Category>>;

      private _authHttp: AuthHttp;
      private _categoriesObserver: any;

      constructor(authHttp: AuthHttp) {
        this._authHttp = authHttp;
        this.categories = new Observable(
          observer => {
            this._categoriesObserver = observer
          }
        ).share();
        this.category = new BehaviorSubject<Category>(null);
      }

      getAll() {
        return this._authHttp
        .get('/api/categories', { headers: contentHeaders })
        .map((res: Response) => res.json())
        .map((data) => {
          let categories = data.map((category) => {
            return new Category(
              category._id,
              category.name,
              category.description,
              category.owner,
              category.collaborators
            );
          });

          this._categoriesObserver.next(categories);

          return categories;
        });
      }

      findById(id) {
        return this._authHttp
        .get(`/api/categories/${id}`, { headers: contentHeaders })
        .map((res: Response) => res.json())
      }

      create(category) {
        let body = JSON.stringify(category);

        return this._authHttp
        .post('/api/categories', body, { headers: contentHeaders })
        .map((res: Response) => res.json())
      }

      update(category) {
        let body = JSON.stringify(category);

        return this._authHttp
        .put(`/api/categories/${category._id}`, body, { headers: contentHeaders })
        .map((res: Response) => res.json())
      }

      delete(category) {
        return this._authHttp
        .put(`/api/categories/${category._id}`, '', { headers: contentHeaders })
        .map((res: Response) => res.json())
      }
    } 
    ```

如您所见，该服务将公开所有 CRUD 操作所需的方法。每个方法都将返回一个可观察对象，它将发出单个响应。我们还使用自己的`AuthHttp`来检查请求是否未经授权，用户需要登录。

注意，除了返回的可观察对象外，`getAll()`方法还更新了`categories`数据流，以便将新值推送到每个订阅者。当多个订阅者使用相同的数据源以自己的方式显示数据时，这将非常有用。

## 类别组件

我们将创建一个组件，用于我们在本章开头配置的 `/categories` 路径。本章早些时候使用了 `AppComponent` 的最终版本。

`CategoriesComponent` 将使用另外两个组件来创建一个新的类别并列出系统中的所有可用条目。让我们创建一个新的文件，`public/src/category/categories.component.ts`：

```js
import { Component } from 'angular2/core';
import { CategoryListComponent } from './category-list.component';
import { CategoryCreateComponent } from './category-create.component';

@Component({
    selector: 'categories',
    directives: [
      CategoryCreateComponent,
      CategoryListComponent
    ],
    template: `
      <category-create></category-create>
      <category-list></category-list>
    `
})
export class CategoryComponent {
  constructor() {}
}
```

之前的组件没有太多内容；我们没有移动部件。我们只是导入两个必要的组件并将它们包含在模板中。让我们继续实现此上下文中的其他两个组件。

## 创建一个新的类别

用户必须能够与我们的应用程序交互并添加新类别，因此我们将为此创建一个单独的组件。让我们将其分解为以下步骤：

1.  首先，创建一个名为 `public/src/category/components/category-create.component.ts` 的视图文件。

1.  导入必要的依赖项：

    ```js
    import { Component, OnInit } from 'angular2/core';
    import { CategoryService } from '../category.service';
    import { Category } from '../category.model';
    ```

1.  定义组件注解，其中包含模板：

    ```js
    @Component({
        selector: 'category-create',
        template: `
          <div>
            <form role="form" (submit)="onSubmit($event)">
              <div class="form-group">
                <label for="name">Name</label>
                <input type="text" [(ngModel)]="category.name" class="form-control" id="name">
              </div>
              <div class="form-group">
                <label for="description">Description</label>
                <textarea class="form-control" id="description"
                  name="description" [(ngModel)]="category.description">
                </textarea>
              </div>
              <button type="submit" class="button">Add</button>
            </form>
          </div>
        `
    })
    ```

1.  添加组件的类：

    ```js
    export class CategoryCreateComponent implements OnInit {
      public category: Category;
      public categories: Array<Category>;
      private _categoryService: CategoryService;

      constructor(categoryService: CategoryService) {
        this._categoryService = categoryService;
      }

      ngOnInit() {
        this.category = new Category();
      }

      onSubmit(event) {
        event.preventDefault();

        this._categoryService
        .create(this.category)
        .subscribe((category) => {
          this._categoryService.category.next(category);
          this.category = new Category();
        }, err => console.error(err));
      }
    }
    ```

每次我们添加一个新的类别，我们希望向所有订阅者广播新项目。例如，类别列表应该显示新条目。在我们成功创建类别后，表单应该重置到其初始值。

## 列出所有类别

现在我们能够创建类别，我们应该能够为用户列出它们。为了列出类别，我们将使用两个组件，一个组件用于迭代来自服务器的数据，另一个用于显示类别的信息。

后者组件也将封装更新功能。因此，用户可以随时更改类别的信息，并在后端持久化更改。

让我们创建一个新的组件文件用于类别列表，名为 `public/src/category/components/category-list.component.ts`，并遵循以下步骤：

1.  导入必要的模块：

    ```js
    import { Component, OnInit, OnDestroy } from 'angular2/core';
    import { CategoryService } from '../category.service';
    import { CategoryComponent } from './category.component';
    import { Category } from '../category.model';
    ```

    我们导入了 `CategoryComponent`，但目前它还不存在，但我们应该已经对我们的组件如何使用有一个想法。

1.  定义模板和组件注解：

    ```js
    @Component({
        selector: 'category-list',
        directives: [CategoryComponent],
        template: `
          <div class="jumbotron center-block">
            <h2>List of all your categories</h2>
          </div>
          <div>
            <category *ngFor="#category of categories" [category]="category"></category>
          </div>
        `
    })
    ```

    我们正在使用 `ngFor` 指令来渲染列表中每个项目的 `category` 模板。

1.  声明组件的类：

    ```js
    export class CategoryListComponent implements OnInit, OnDestroy {
      public categories: Array<Category>;
      private _categoryService: CategoryService;
      private _categorySubscription: any;

      constructor(categoryService: CategoryService) {
        this._categoryService = categoryService;
      }

      ngOnInit() {
        this._categorySubscription = this._categoryService.category
        .subscribe((category) => {
          if (category) {
            this.categories.push(category);
          }
        });
        this._categoryService.getAll()
        .subscribe((categories) => {
          this.categories = categories;
        });
      }

      ngOnDestroy() {
        this._categorySubscription.unsubscribe();
      }
    }
    ```

当组件初始化时，我们将使用我们的 `CategoryService` 从后端检索所有可用的类别。除了获取所有必要的数据外，我们还订阅了创建新类别时的情况。基本上，我们订阅了类别数据流。

每次添加新的类别时，它将被推送到 `categories` 列表并显示给用户。为了向用户渲染信息，我们将有一个用于单个类别的组件。

当组件被销毁时，我们希望取消订阅数据流；否则，通知将被推送到数据流中。

## 类别组件

要显示我们列表中单个类别的信息，我们将创建一个新的组件，称为 `public/src/category/components/category.component.ts`：

```js
import { Component } from 'angular2/core';
import { CategoryService } from '../category.service';
import { Category } from '../category.model';

@Component({
    inputs: ['category'],
    selector: 'category',
    template: `
    <div>
      <form role="form" (submit)="onSubmit($event)">
        <div class="form-group">
          <label for="name">Name</label>
          <input type="text" [(ngModel)]="category.name" class="form-control" id="name">
        </div>
        <div class="form-group">
          <label for="description">Description</label>
          <textarea class="form-control" id="description"
            name="description" [(ngModel)]="category.description">
          </textarea>
        </div>
        <button type="submit" class="button">save</button>
      </form>
    </div>
    `
})
export class CategoryComponent {
  public category: Category;
  private _categoryService: CategoryService;

  constructor(categoryService: CategoryService) {
    this._categoryService = categoryService;
  }

  onSubmit(event) {
    event.preventDefault();
    this._categoryService.update(this.category)
    .subscribe((category) => {
      this.category = category;
    }, err => console.error(err));
  }
}
```

这个类别获取输入数据以显示关于类别的信息。当点击**保存**按钮并提交表单时，它还会触发一个事件。我们使用我们的服务与服务器通信并在 MongoDB 中持久化更改。

# 支出模块

在这个模块中，我们将处理与支出相关的功能。这将是前端应用中用户使用的主要模块，因为在这里他们将添加新的支出并通过我们的后端 API 将它们存储在 MongoDB 中。

## 支出服务

支出服务将实现支出上的 CRUD 操作，并且它的重要特性之一是获取支出的余额。为了创建支出服务，我们将遵循以下步骤：

1.  创建一个名为 `public/src/expense/expense.service.js` 的文件。

1.  定义服务的主要逻辑：

    ```js
    import { Injectable } from 'angular2/core';
    import { Http, Response, Headers } from 'angular2/http';
    import { Observable } from 'rxjs/Observable';
    import { Subject } from 'rxjs/Subject';
    import { BehaviorSubject } from 'rxjs/Subject/BehaviorSubject';
    import { AuthHttp } from '../auth/index';
    import { contentHeaders, serializeQuery } from '../common/index';
    import { Expense } from './expense.model';

    @Injectable()
    export class ExpenseService {
      public expense: Subject<Expense>;
      public expenses: Observable<Array<Expense>>;
      public filter: Subject<any>;
      private _authHttp: AuthHttp;
      private _expensesObserver: any;

      constructor(authHttp: AuthHttp) {
        this._authHttp = authHttp;
        this.expenses = new Observable(
          observer => {
            this._expensesObserver = observer
          }
        );
        this.filter = new BehaviorSubject<any>(null);
        this.expense = new BehaviorSubject<Expense>(null);
      }
      create(expense) {
      }
      findById(id) {
      }
      getAll() { 
      }
      update(expense) {
      }
      delete(expense) {
      }
    }
    ```

我们刚刚定义了一个公开方法的列表。我们还公开了一些公共属性，以便外部更新过滤器，例如支出，以及一个可观察的支出数据流。

现在让我们逐个跟踪方法并附加它们的实际实现：

1.  创建支出：

    ```js
      create(expense) {
        let body = JSON.stringify(expense);

        return this._authHttp
        .post('/api/expenses', body, { headers: contentHeaders })
        .map((res: Response) => res.json())
        .map((expense) => {
          return new Expense(
            expense._id,
            expense.name,
            expense.currency,
            expense.amoun,
            expense.scaleFactor,
            expense.value,
            expense.user,
            expense.category,
            expense.createdAt
          );
        });
      }
    Getting one expense by ID:
      findById(id) {
        return this._authHttp
        .get(`/api/expenses/${id}`, { headers: contentHeaders })
        .map((res: Response) => res.json())
        .map((expense) => {
          return new Expense(
            expense._id,
            expense.name,
            expense.currency,
            expense.amoun,
            expense.scaleFactor,
            expense.value,
            expense.user,
            expense.category,
            expense.createdAt
          );
        });
      }
    ```

1.  获取与给定查询标准匹配的所有支出：

    ```js
      getAll(criteria?: any) {
        let query = '';

        if (criteria) {
          query = `?${serializeQuery(criteria)}`
        }

        this._authHttp
        .get(`/api/expenses${query}`, { headers: contentHeaders })
        .map((res: Response) => res.json())
        .map((data) => {
          return data.map((expense) => {
            return new Expense(
              expense._id,
              expense.name,
              expense.currency,
              expense.amoun,
              expense.scaleFactor,
              expense.value,
              expense.user,
              expense.category,
              expense.createdAt
            );
          });
        }).subscribe((expenses: Array<Expense>) => {
          this._expensesObserver.next(expenses);
        }, err => console.error(err));
      }
    ```

    上述方法使用 `serializeQuery()` 方法，该方法将我们的标准转换为 `查询字符串` 参数。我们这样做是为了根据给定的标准过滤支出。而且，我们不是从 HTTP 调用返回一个可观察的，而是更新我们的 `expenses` 数据流，以通知所有订阅者新可用的数据。

1.  获取与查询标准匹配的支出余额：

    ```js
      getExpensesBalance(criteria?: any) {
        let query = '';

        if (criteria) {
          query = `?${serializeQuery(criteria)}`
        }

        return this._authHttp
        .get(`/api/expenses/balance${query}`, { headers: contentHeaders })
        .map((res: Response) => res.json())
      }
    ```

    我们使用相同的 `serializeQuery()` 函数将我们的标准转换为 `查询字符串`。

1.  通过 ID 使用新数据更新支出：

    ```js
      update(expense) {
        let body = JSON.stringify(expense);

        return this._authHttp
        .put(`/api/expenses/${expense._id}`, body, { headers: contentHeaders })
        .map((res: Response) => res.json())
      }
    Removing an existing expense by ID:
      delete(expense) {
        return this._authHttp
        .put(`/api/expenses/${expense._id}`, '', { headers: contentHeaders })
        .map((res: Response) => res.json())
      }
    ```

## 过滤支出

作为开始，我们将实现支出过滤。我们只想有所有必要的块来正确列出支出。基本上，这个组件将是一个简单的表单，包含三个输入：开始日期、结束日期和类别。

使用这些简单的标准，我们将在后端过滤我们的支出。记住，我们需要这些在 `query` 参数中，以便从 `expenses` 集合中检索正确的数据。

这个组件将依赖于 `CategoryService` 并订阅类别数据流。它还将向下推送新的值到过滤器流中，以通知每个订阅者过滤支出。

让我们遵循以下步骤来实现我们的组件：

1.  导入模块：

    ```js
    import { Component, OnInit, OnDestroy } from 'angular2/core';
    import { CategoryService, Category } from '../../category/index';
    import { ExpenseService } from '../expense.service';
    import { Expense } from '../expense.model';
    ```

1.  定义我们组件的模板：

    ```js
    @Component({
        selector: 'expense-filter',
        template: `
          <div>
            <form role="form">
              <div class="form-group">
                <label for="startDate">Start</label>
                <input type="date" [(ngModel)]="filter.startDate" class="form-control" id="startDate">
              </div>
              <div class="form-group">
                <label for="endDate">End</label>
                <input type="date" [(ngModel)]="filter.endDate" class="form-control" id="endDate">
              </div>
              <div class="form-group">
                <label for="category">Category</label>
                <select name="category" [(ngModel)]="filter.category">
                  <option *ngFor="#category of categories" [value]="category._id">
                    {{ category.name }}
                  </option>
                </select>
              </div>
              <button type="submit" class="button" (click)="onFilter($event)">Filter</button>
              <button type="button" class="button" (click)="onReset($event)">Reset</button>
            </form>
          </div>
        `
    })
    ```

1.  添加 `ExpenseFilterComponent` 类：

    ```js
    export class ExpenseFilterComponent implements OnInit, OnDestroy {
      public filter: any;
      public categories: Array<Category>;
      private _expenseService: ExpenseService;
      private _categoryService: CategoryService;

      constructor(
        expenseService: ExpenseService,
        categoryService: CategoryService
      ) {
        this._expenseService = expenseService;
        this._categoryService = categoryService;
      }
    }
    ```

1.  初始化时会发生什么：

    ```js
      ngOnInit() {
        this.filter = {};
        this.categories = [];
        this._subscriptions = [];
        this._subscriptions.push(
          this._categoryService
          .categories
          .subscribe((categories) => {
            this.categories = categories;
          })
        );
      }
    ```

1.  当组件被销毁时：

    ```js
      ngOnDestroy() {
        this._subscriptions.forEach((subscription) => {
          subscription.unsubscribe();
        })
      }
    ```

    我们必须取消订阅数据流。我们使用一个订阅列表来在一个地方保存所有这些，然后稍后遍历订阅并销毁它们。

1.  我们如何更新过滤器流：

    ```js
      onFilter(event) {
        event.preventDefault();
        this._expenseService.filter.next(this.filter);
      }
    ```

1.  重置过滤器：

    ```js
      onReset(event) {
        event.preventDefault();
        this.filter = {};
        this._expenseService.filter.next(this.filter);
      }
    ```

当组件初始化时，我们订阅 `categories` 数据流。如果用户点击 `filter` 按钮，我们将更新 `filter`，以便每个订阅者都可以获取新的过滤标准。

为了重置一切，我们可以使用 `reset` 按钮，回到初始状态。然后我们可以通知所有订阅者，我们可以再次检索所有支出。

## 添加新的支出

由于添加支出将是一个相当常用的功能，我们将把必要的逻辑添加到与列出支出相同的视图和控制台中。

记住，为了添加新的支出，它必须包含在一个类别中。因此，我们需要将类别列表加载到组件中。这应该类似于我们在 `ExpenseFilterComponent` 中之前所做的那样。

让我们通过以下步骤来实现添加支出的功能：

1.  创建一个新的文件，命名为 `public/src/expense/components/expense-create.component.ts`。

1.  导入必要的模块：

    ```js
    import { Component, OnInit, OnDestroy } from 'angular2/core';
    import { Router, RouterLink } from 'angular2/router';
    import { CategoryService, Category } from '../../category/index';
    import { ExpenseService } from '../expense.service';
    import { Expense } from '../expense.model';
    ```

1.  使用以下模板添加注释：

    ```js
    @Component({
        selector: 'expense-create',
        directives: [
          RouterLink
        ],
        template: `
          <div>
            <form role="form" (submit)="onSubmit($event)">
              <div class="form-group">
                <label for="name">Name</label>
                <input type="text" [(ngModel)]="expense.name" class="form-control" id="name">
              </div>
              <div class="form-group">
                <label for="category">Category</label>
                <select name="category" [(ngModel)]="expense.category">
                  <option *ngFor="#category of categories" [value]="category._id">
                    {{ category.name }}
                  </option>
                </select>
              </div>
              <div class="form-group">
                <label for="value">Amount</label>
                <input type="text" [(ngModel)]="expense.value" class="form-control" id="value">
              </div>
              <button type="submit" class="button">Add</button>
            </form>
          </div>
        `
    })
    ```

1.  添加以下类：

    ```js
    export class ExpenseCreateComponent implements OnInit, OnDestroy {
      public expense: Expense;
      public categories: Array<Category>;
      private _expenseService: ExpenseService;
      private _categoryService: CategoryService;
      private _subscriptions: Array<any>;

      constructor(
        expenseService: ExpenseService,
        categoryService: CategoryService
      ) {
        this._expenseService = expenseService;
        this._categoryService = categoryService;
      }
    ```

1.  在初始化时，我们订阅类别数据流并将订阅存储起来，以便我们可以在以后取消订阅：

    ```js
      ngOnInit() {
        this.expense = new Expense();
        this.categories = [];
        this._subscriptions = [];
        this._subscriptions.push(
          this._categoryService
          .categories
          .subscribe((categories) => {
            this.categories = categories;
          })
        );
      }
    ```

1.  当组件销毁时取消订阅：

    ```js
      ngOnDestroy() {
        this._subscriptions.forEach((subscription) => {
          subscription.unsubscribe();
        })
      }
    Create a new expense event:
      onSubmit(event) {
        event.preventDefault();

        this._expenseService
        .create(this.expense)
        .subscribe((expense) => {
          this._expenseService.expense.next(expense);
        }, err => console.error(err));
      }
    ```

## 列出支出

为了显示支出列表，我们将查询服务器获取必要的信息，并创建一个包含检索信息的表格。为此，我们将执行以下步骤：

1.  创建支出控制器文件，命名为 `public/src/expense/components/expense-list.component.ts`。

1.  导入服务和其它依赖项：

    ```js
    import { Component, OnInit, OnDestroy } from 'angular2/core';
    import { ExpenseService } from '../expense.service';
    import { Expense } from '../expense.model'; 
    ```

1.  在模板中定义 `expense` 表：

    ```js
    @Component({
        selector: 'expense-list',
        directives: [],
        template: `
          <div class="jumbotron center-block">
            <h2>List of all your expenses</h2>
          </div>
          <div>
            <table>
              <thead>
                <tr>
                  <th>Name</th>
                  <th>Category</th>
                  <th>Amount</th>
                  <th>Date</th>
                </tr>
              </thead>
              <tbody>
                <tr *ngFor="#expense of expenses">
                  <td>{{ expense.name }}</td>
                  <td>{{ expense.category.name }}</td>
                  <td>{{ expense.value }}</td>
                  <td>{{ expense.createdAt | date }}</td>
                </tr>
              </tbody>
            </table>
          </div>
        `
    })
    ```

1.  声明 `ExpenseListComponent` 类：

    ```js
    export class ExpenseListComponent implements OnInit, OnDestroy {
      public expenses: Array<Expense>;
      private _expenseService: ExpenseService;
      private _subscriptions: Array<any>;

      constructor(expenseService: ExpenseService) {
        this._expenseService = expenseService;
      }
    }
    ```

1.  初始化时订阅所有数据流：

    ```js
      ngOnInit() {
        this.expenses = [];
        this._subscriptions = [];

        this._subscriptions.push(
          this._expenseService
          .expenses
          .subscribe((expenses) => {
            this.expenses = expenses;
          })
        );
        this._subscriptions.push(
          this._expenseService
          .expense
          .subscribe((expense) => {
            if (expense) {
              this.expenses.push(expense);
            }
          })
        );
        this._subscriptions.push(
          this._expenseService
          .filter
          .subscribe((filter) => {
            if (filter) {
              this._expenseService.getAll(filter);
            }
          })
        );
      }
    ```

1.  当组件销毁时取消订阅：

    ```js
      ngOnDestroy() {
        this._subscriptions.forEach(subscription => {
          subscription.unsubscribe();
        });
      }
    ```

我们主要使用数据流来向用户显示信息。当创建新的支出时，我们只是得到通知并更新支出列表。如果加载了新的支出集，列表将使用新值更新。我们还订阅了过滤器的变化，以便我们可以使用该过滤器从后端获取数据。

## 显示平衡

我们希望显示支出的累积值。当我们过滤支出时，相同的过滤器应该应用于余额的查询。例如，我们可能想显示特定类别的支出；在这种情况下，应该显示该类别的余额。

因为我们都在后端做所有繁重的工作，并且通过 API 获取的结果格式良好，所以我们只需要实现一些功能来正确显示余额：

1.  为组件创建一个新的文件，命名为 `public/src/expense/components/expense-balance.component.ts`。

1.  实现基类：

    ```js
    import { Component, OnInit, OnDestroy } from 'angular2/core';
    import { ExpenseService } from '../expense.service';

    @Component({
        selector: 'expense-balance',
        directives: [],
        template: `
          <h2>
            Total balance: {{ info.balance }}
            <span>from {{ info.count }}</span>
          </h2>
        `
    })
    export class ExpenseBalanceComponent implements OnInit, OnDestroy {
      public info: any;
      private _expenseService: ExpenseService;
      private _subscriptions: Array<any>;

      constructor(expenseService: ExpenseService) {
        this._expenseService = expenseService;
      }

      ngOnInit() {

      }

      ngOnDestroy() {

      }

    }
    Subscribe to the change of filter on init:
      ngOnInit() {
        this.info = {};
        this._subscriptions = [];

        this._subscriptions.push(
          this._expenseService
          .filter
          .subscribe((filter) => {
            if (filter) {
              this._getBalance(filter);
            }
          })
        );
      }
    ```

1.  根据标准从后端检索余额：

    ```js
      ngOnDestroy() {
        this._subscriptions.forEach((subscription) => {
          subscription.unsubscribe();
        })
      }
    ```

1.  取消订阅：

    ```js
      ngOnDestroy() {
        this._subscriptions.forEach((subscription) => {
          subscription.unsubscribe();
        })
      }
    ```

## 支出组件

现在我们已经拥有了所有必要的组件，我们可以实现我们的主要支出组件，它将使用之前实现的所有子组件。我们应该创建一个名为 `public/src/expense/components/expenses.component.ts` 的新文件：

```js
import { Component, OnInit } from 'angular2/core';
import { Router, RouterLink } from 'angular2/router';
import { ExpenseService } from '../expense.service';
import { CategoryService } from '../../category/index';
import { ExpenseCreateComponent } from './expense-create.component';
import { ExpenseListComponent } from './expense-list.component';
import { ExpenseBalanceComponent } from './expense-balance.component';
import { ExpenseFilterComponent } from './expense-filter.component';
@Component({
    selector: 'expenses',
    directives: [
      ExpenseCreateComponent,
      ExpenseListComponent,
      ExpenseBalanceComponent,
      ExpenseFilterComponent
    ],
    template: `
      <expense-balance></expense-balance>
      <expense-filter></expense-filter>
      <expense-create></expense-create>
      <expense-list></expense-list>
    `
})
export class ExpensesComponent implements OnInit {
  private _expenseService: ExpenseService;
  private _categoryService: CategoryService;

  constructor(
    expenseService: ExpenseService,
    categoryService: CategoryService
  ) {
    this._expenseService = expenseService;
    this._categoryService = categoryService;
  }

  ngOnInit() {
    this._categoryService.getAll().subscribe();
    this._expenseService.filter.next({});
  }
}
```

该组件相当简单，但在我们仅获取所有类别并将过滤器设置为空对象时，`ngOnInit()` 方法中发生了一件有趣的事情。当这种情况发生时，所有其他组件都会对我们的操作做出反应并相应更新。

通过这种方式，我们已经实现了支出模块，它允许用户添加支出并查看所有支出的列表。我们省略了一些功能，例如错误处理、分页和其他一些小功能，但你可以根据需要改进这段代码。

# 摘要

这就带我们结束了一个相当长的章节。

我们学习了如何使用 JavaScript 和 Node.js 操作货币数据以及如何将其存储在 MongoDB 中。我们实现了一个多用户系统，用户可以轻松地在任何时间注册和登录。

我们通过 API 公开了大部分后端功能。我们使用了一种无状态的认证机制，只有通过展示有效的令牌才能授予访问权限。

在下一章中，我们将构建一个更具公共导向的网页，包含不同的账户类型。
