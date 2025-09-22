# 第十二章. Node.js 中的认证

我们迄今为止构建的应用程序允许用户选择一个用户名来识别自己。然而，他们只能在浏览器会话期间保留这个身份。允许用户从一个会话持续到下一个会话保持一致的身份是很重要的。这使我们能够构建更丰富的用户体验。一些网站（如 Facebook）如果不能识别用户，甚至无法提供其主要功能。

识别用户需要我们实现认证。在本章中，我们将涵盖以下主题：

+   通过社交网站实现第三方认证

+   将第三方身份与我们的用户数据关联

+   模拟用户认证以支持集成测试

# 介绍 Passport

Passport 是一个 Node.js 的认证框架。它可以作为 Express 中间件，使得与我们的应用程序集成变得容易。

就像我们之前讨论的一些其他库一样，Passport 非常模块化。它的核心包提供了一个通用的认证范式。Passport 的中间件执行认证，并在请求对象中添加一个`user`属性。

额外的 Passport npm 包支持数百种不同的**策略**进行认证。每个 Passport 策略都提供了识别用户的不同机制。在本章中，我们将探讨这些策略中的几个。Passport 使得为每个应用程序添加新策略变得容易。

## 选择认证策略

一个常见的入门示例是基于用户名/密码的认证。这使用登录表单来验证用户的凭据与应用程序数据库的匹配。尽管这是最容易理解的认证机制之一，但它并不是最有用的。强迫用户为我们的网站创建账户是使用它的额外障碍。用户也会厌倦为每个新网站创建账户和选择密码。

Passport 支持这种类型的认证，通过`passport-local`策略。我们将在本章后面部分使用这个策略进行测试，但不会在我们的生产代码中使用。最好允许用户使用已在其他地方建立的身份进行认证。这可以节省用户挑选新凭据的时间和精力，也可以让我们的网站不必管理这些凭据。这只是良好的关注点分离。

如果你登录 StackOverflow，你会注意到它建议使用 Google+或 Facebook 进行登录。它还支持 OpenID 和其他提供者。从头开始实现对这些每种登录机制的支持将会是一项大量工作。幸运的是，有 Passport 策略支持所有这些。

## 理解第三方认证

Passport 将为我们做大部分繁重的工作，但仍然值得了解第三方认证的基本工作原理。当客户端想要登录网站时，它会将他们发送到第三方提供者。第三方提供者会向客户端返回一个他们可以使用来对网站进行身份验证的令牌。当客户端是网络浏览器时，这个过程可以通过自动重定向几乎对用户不可见。

网站必须验证客户端向其展示的令牌确实来自第三方提供者。网站和第三方提供者可能已经为此目的建立了一个预共享密钥，该密钥可以用来创建一个可加密验证的令牌。或者，网站可能直接调用第三方提供者来验证令牌。在实践中，网站通常会调用第三方提供者以获取更多与用户身份相关的信息，例如他们的用户名或其他个人资料信息。

# 使用 Express 会话

Passport 的许多策略都是基于 HTTP 会话的。目前，我们的应用程序只是使用简单的 cookie 来存储用户 ID。为了使用 Passport 进行第三方认证，我们需要在我们的应用程序中添加会话支持。Express 在`express-session`模块中提供了会话支持。首先，我们将此添加到我们的应用程序中：

```js
> npm install express-session --save

```

我们还需要一个地方来存储会话数据。Express 通过额外的模块支持多种会话存储。Redis 非常适合这项任务，我们已经有了一个 Redis 实例可用。我们可以使用`connect-redis`模块将会话存储在 Redis 中：

```js
> npm install connect-redis --save

```

我们现在可以创建一个新的配置模块，将所有的会话逻辑放在一个地方。由于这将返回中间件，我们将它放在这里的`middleware`文件夹中`src/middleware/sessions.js`：

```js
'use strict';

const session = require('express-session');

let config = {
    secret: process.env.SESSION_SECRET,
    saveUninitialized: false,
    resave: false
};

if (process.env.REDIS_URL && process.env.NODE_ENV !== 'test') {
    const RedisStore = require('connect-redis')(session);
    config.store = new RedisStore({ url: process.env.REDIS_URL });
}

module.exports = session(config);
```

我们按照以下方式配置 Express 的`session`模块：

+   使用环境变量的值作为会话密钥

+   只保存包含某些数据的会话

+   除非会话已更改，否则不要重新保存会话

+   如果 Redis 可用，请将其用作会话存储

让我们逐一考虑每个配置属性。

## 指定会话密钥

Express 使用会话密钥来保护会话数据不被篡改。你应该通过设置本地的`SESSION_SECRET`环境变量来指定它。值是任意的，可以是任何内容，只要它不为空。我们还需要在我们的集成测试中指定这个值，以便它可以在 CI 服务器上运行。以下代码来自`gulpfile.js`：

```js
gulp.task('integration-test', ..., (done) => {
    const TEST_PORT = 5000;
 process.env.SESSION_SECRET =
 process.env.SESSION_SECRET || 'testOnly';
    require('./src/server.js').then((server) => {
        ...
    });
}); 
```

## 决定何时保存会话

避免不必要的保存是一种小优化，可以避免某些竞态条件。只有保存已初始化的会话，你才能在存储任何 cookie 之前请求用户同意。这可能对于遵守区域法律是必要的，尤其是在欧盟。更多信息请参阅[`www.cookiechoices.org/`](https://www.cookiechoices.org/)。

## 使用替代会话存储

默认情况下，Express 将使用内存中的会话存储。这对于开发目的和在只有一个应用程序进程的测试环境中是可行的，但不适合生产使用。如果我们想要跨多个实例进行扩展，将会话存储在进程外部的 Redis 中很重要。我们使用现有的 Redis URL 配置 Redis 存储。

### 注意

在实践中，您可能希望为会话数据和应用程序的其他数据使用不同的 Redis 实例。这些是相当不同的用例，因此它们可能从不同的 Redis 配置中受益。例如，会话数据可能负载更高，但可以承受更高的易变性。对于本书中的示例应用等小型应用，单个 Redis 实例就足够了。

## 使用会话中间件

我们现在可以在应用程序的其他地方使用会话，而不是直接设置 cookie。以下代码来自`src/app.js`：

```js
 let sessions = require('./middleware/sessions');
    ...
    app.use(bodyParser.urlencoded({ extended: false }));
 app.use(sessions);
    app.use(express.static(path.join(__dirname, 'public')));
    ...
```

以下代码来自`src/middleware/users.js`：

```js
'use strict';

module.exports = (service) => {
    const uuid = require('uuid');

    return function(req, res, next) {
 let userId = req.session.userId;
        if (!userId) {
            userId = uuid.v4();
 req.session.userId = userId;
            req.user = {
                id: userId
            };
            next();
        } else {
            ...
        }
    };
};
```

以下代码来自`src/server.js`：

```js
'use strict';

module.exports = require('./config/mongoose').then(mongoose => {
    ...
 io.use(adapt(require('./middleware/sessions')));
    const usersService = require('./services/users.js');
    ...
});
```

# 实现社交登录

在我们的第一个示例中，我们将使用 Twitter 作为第三方身份验证提供者。如果您想跟随示例，您需要一个 Twitter 账户，设置起来非常快。

## 设置 Twitter 应用

为了让 Twitter 识别我们的应用程序，我们需要在 Twitter 开发者门户中创建一个新的应用：

1.  访问[`apps.twitter.com/`](https://apps.twitter.com/)并点击**创建新应用**。

1.  填写名称、描述、网站和回调 URL 字段：

    +   如果您已将应用程序部署到 Heroku，您可以使用其 Heroku URL

    +   否则，只需为这两个字段填写占位符值（例如，`http://test.example.com/callback`）

1.  点击**创建您的 Twitter 应用**。

1.  点击**设置**选项卡，并确保**启用回调锁定**未勾选（不勾选此选项允许您为 URL 使用占位符值，并且对于本地测试也很有用）。

1.  点击**密钥和访问令牌**选项卡以查看您的应用程序的**消费者密钥（API 密钥**）和**消费者密钥（API 密钥**）。

设置名为`TWITTER_API_KEY`和`TWITTER_API_SECRET`的新本地环境变量，包含来自 Twitter 的相应值。您可能希望创建一个 shell 脚本或批处理文件来在控制台设置这些变量，或者将它们配置为 Heroku 环境变量（见第十一章，*部署 Node.js 应用程序*）

## 配置 Passport

我们现在将使用 Passport 允许用户通过 Twitter 登录我们的网站。首先，我们需要安装相关的 npm 包：

```js
> npm install passport --save
> npm install passport-twitter --save

```

现在，我们可以在`src/config/passport.js`下配置 Passport 以使用 Twitter 进行身份验证。我们添加以下代码：

```js
'use strict';

const passport = require('passport');
const TwitterStrategy = require('passport-twitter').Strategy;

module.exports = (usersService) => {
    if(process.env.TWITTER_API_KEY &&
            process.env.TWITTER_API_SECRET) {
        passport.use(new TwitterStrategy({
            consumerKey: process.env.TWITTER_API_KEY,
            consumerSecret: process.env.TWITTER_API_SECRET,
            callbackURL: '/auth/twitter/callback',
            passReqToCallback: true
        }, (req, token, tokenSecret, profile, done) => {
            usersService.setUsername(req.user.id,
                    profile.username || profile.displayName)
                .then(() => { done(); }, done);
        }));
    }
    return passport;
};
```

这使用`TwitterStrategy`进行 Twitter 认证，通过配置对象传递我们的 API 密钥和密钥。第二个构造函数参数是一个函数，Passport 在通过 Twitter 认证后会调用此函数（在 Passport 文档中称为**验证回调**）。在这里，我们根据 Passport 提供的来自 Twitter 的`profile.username`或`profile.displayName`设置当前用户的名字。

### 注意

`profile`对象包含认证提供者返回的用户个人资料。Passport 将个人资料数据标准化，以便更容易与多个策略一起使用。有一个标准的字段集，例如`displayName`，所有 Passport 策略如果可能都会填充这些字段。我们更愿意使用 Twitter 用户名（例如，hgcummings）而不是显示名称（例如，Harry Cummings）。`profile.username`字段包含 Twitter 用户名。这不是标准字段之一，但许多策略会返回一个具有此名称的字段。因此，我们首先使用`profile.username`，但会回退到更标准的`profile.displayName`。

现在我们只需要在我们的 Express 应用程序中使用新的 passport 模块。以下代码来自`src/app.js`：

```js
 let passport = require('./config/passport')(usersService);
    ... 

    app.use(users);
 app.use(passport.initialize());
app.post('/auth/twitter', passport.authenticate('twitter'));
app.get('/auth/twitter/callback',
 passport.authenticate('twitter',
 { successRedirect: '/', failureRedirect: '/' }));

    app.use('/', routes);
    ...
```

这告诉我们的应用程序执行以下三件事：

+   使用 Passport 的 Express 中间件

+   当用户向`/auth/twitter`发送 POST 请求时，通过 Twitter 进行用户认证

+   在将用户重定向到主页之前，在`/auth/twitter/callback`处理 Twitter 身份验证结果

最后，我们需要提供一个登录按钮来访问我们新的端点，如`src/views/index.js`中所示：

```js
    <h1>{{ title }}</h1>
 <h2>Account</h2>
 {{#ranking}}
 ...
 {{/ranking}}
 <form action="/auth/twitter" method="POST">
 <input type="submit" value="Log in using Twitter" />
 </form>
 <h3>Profile</h3>
    <form action="/profile" method="POST">
      ...
    </form>
    ...
```

如果你运行应用程序并点击**使用 Twitter 登录**，以下将发生：

+   应用程序将重定向你的浏览器到 Twitter

+   如果你尚未登录，Twitter 将提示你登录

+   Twitter 将询问你是否愿意应用程序查看你的个人资料详情和其他公开数据

+   Twitter 将然后重定向你的浏览器到`/auth/twitter/callback`端点

+   你的浏览器将向此端点发送带有 Twitter 认证令牌的请求

+   Passport 将验证此令牌然后调用我们的登录处理函数

+   当我们的函数完成后，Passport 将返回一个重定向响应到主页

我们现在已经将 Twitter 身份验证集成到我们的应用程序中！然而，我们并没有真正使用它来允许用户登录。我们只是将 Twitter 用户名与为每个会话创建的现有用户 ID 关联起来。你可以通过打开两个不同的浏览器会话来查看这一点。尝试使用它们中的每一个登录。如果你在一个浏览器中创建了一个新游戏，它会在另一个浏览器中出现在其他用户创建的游戏列表中。这是因为你现在有两个用户 ID 与同一个 Twitter 用户名相关联。

我们需要在用户使用相同的 Twitter 账号登录时识别出相同的用户。这不应该依赖于是否在同一个浏览器会话中。为了解决这个问题，我们需要执行以下操作：

+   将用户账户持久化到我们的数据库

+   告诉 Passport 如何存储和检索用户

+   让 Passport 将用户与当前会话关联起来

## 使用 Redis 持久化用户数据

我们已经使用 Redis 将用户名与用户 ID 关联。现在我们希望也能将用户 ID 与 Twitter 账户关联起来。当用户第一次使用外部提供者登录时，我们希望创建一个新的用户，其名称来自外部配置文件。后续使用相同提供者进行身份验证的请求将看到相同的用户。

我们可以使用 Redis 的 `SETNX` 操作来实现这一功能。这只会设置一个键，如果它不存在，并返回这是否是这种情况。我们的实现如下，来自 `src/services/users.js`：

```js
'use strict';

const redisClient = require('../config/redis.js');
const uuid = require('uuid');

const getUser = userId =>
 redisClient.getAsync(`user:${userId}:name`)
 .then(userName => ({
 id: userId,
 name: userName
 }));

const setUsername = (userId, name) =>
 redisClient.setAsync(`user:${userId}:name`, name);

module.exports = {
 getOrCreate: (provider, providerId, providerUsername) => {
 let providerKey = `provider:${provider}:${providerId}:user`;
 let newUserId = uuid.v4();
 return redisClient.setnxAsync(providerKey, newUserId)
 .then(created => {
 if (created) {
 return setUsername(newUserId, providerUsername)
 .then(() => getUser(newUserId));
 } else {
 return redisClient
 .getAsync(providerKey).then(getUser);
 }
 });
 },
  getUser: getUser,    getUsername: userId => redisClient.getAsync(`user:${userId}:name`),
 setUsername: setUsername,
  ...
};
```

在这里，我们创建一个新的用户 ID，并告诉 Redis 将其与外部提供者（例如，Twitter）账户关联起来。如果我们之前已经看到过这个外部账户，我们将返回之前已经与之关联的用户。否则，我们将持久化一个新的用户 ID，并将其与外部配置文件中的用户名关联起来。对此功能的测试可以在配套代码中找到。

## 配置 Passport 以实现持久化

既然我们已经有了持久化用户的方式，我们需要告诉 Passport 如何利用这一点。首先，我们更新我们的验证回调，使用新的 `getOrCreate` 函数而不是仅仅设置一个用户名。然后我们需要告诉 Passport 如何通过将用户序列化为字符串并从字符串反序列化来识别和检索与会话关联的用户。以下代码来自 `src/config/passport.js`：

```js
'use strict'; 

const passport = require('passport');
const TwitterStrategy = require('passport-twitter').Strategy;

module.exports = (usersService) => {
    if(process.env.TWITTER_API_KEY &&
            process.env.TWITTER_API_SECRET) {
        passport.use(new TwitterStrategy({
            consumerKey: process.env.TWITTER_API_KEY,
            consumerSecret: process.env.TWITTER_API_SECRET,
            callbackURL: '/auth/twitter/callback',
            passReqToCallback: true
        }, (req, token, tokenSecret, profile, done) => {
 usersService.getOrCreate('twitter', profile.id,
 profile.username || profile.displayName)
 .then(user => done(null, user), done);
        }));
    }

 passport.serializeUser((user, done) => {
 done(null, user.id);
 });

 passport.deserializeUser((id, done) => {
 usersService.getUser(id)
 .then(user => done(null, user))
 .catch(done);
 });

    return passport;
};
```

Passport 将用户（由我们的 `serializeUser` 回调返回）的字符串版本存储在会话中。它使用我们的 `deserializeUser` 回调将此字符串转换为用户对象，并将其添加到请求中。在我们的例子中，用户的字符串表示只是他们的 ID，反序列化只是用户服务中的查找。

为了使这生效，我们还需要告诉我们的应用程序使用 Passport 的自己的会话中间件，它与 Express 会话一起工作。为了避免重复，我们将在会话中间件模块中指定所有与会话相关的中间件。以下是从 `src/middleware/sessions.js` 的代码：

```js
...

const expressSession = session(config);
module.exports = passport => [
 expressSession, passport.initialize(), passport.session()
];
```

此模块现在返回三个中间件实例。我们希望同时使用 Express 和 Socket.IO。其中第一个很简单，因为我们可以将多个中间件对象传递给 Express 的 `app.use` 函数，就像在这里 `src/app.js` 一样：

```js
    ...
    let passport = require('./config/passport')(usersService);
 let sessions = require('./middleware/sessions')(passport);
    ...
    app.use(bodyParser.json());
    app.use(bodyParser.urlencoded({ extended: false }));
 app.use(sessions);
    app.use(express.static(path.join(__dirname, 'public')));

    app.post('/auth/twitter', passport.authenticate('twitter'));
    ...
```

对于 Socket.IO，我们需要逐个调整每个中间件，就像在这里 `src/server.js` 一样：

```js
    ...
    const usersService = require('./services/users.js');
 let passport = require('./config/passport');
 require('./middleware/sessions')(passport).forEach(
 middleware => io.use(adapt(middleware)));

    require('./realtime/chat')(io);
    ...
```

注意，在这两种情况下，我们的用户中间件不再需要，现在可以删除。然而，这个中间件之前确保请求中始终有一个用户对象。现在这只有在有登录用户的情况下才会发生，因此我们需要相应地更新我们的应用程序的其他部分。

在我们的应用程序中有几个地方假设请求中始终存在用户。由于这不再得到保证，有两种解决方法：我们可以更新我们的代码以处理请求中没有用户的情况，或者我们可以从未认证用户那里隐藏功能。

我们仍然希望未认证用户能够查看公共聊天室，并查看和玩游戏，因此我们将相应地更新这些功能。来自`src/realtime/chat.js`的代码更新如下：

```js
    namespace.on('connection', (socket) => {
 let username = null;
 if (socket.request.user) {
 username = socket.request.user.name;
 }
        ...
```

以下代码来自`src/realtime/games.js`：

```js
    function forwardEvent(name, socket) {
        service.events.on(name, game => {
 if (!socket.request.user ||
 game.setBy !== socket.request.user.id) {
                socket.emit(name, game.id);
            }
        });
    }
```

以下代码来自`src/routes/games.js`：

```js
    router.post('/:id/guesses', function(req, res, next) {
        checkGameExists(
            req.params.id,
            res,
            game => {
 if (req.user && game.matches(req.body.word)) {
                    userService.recordWin(req.user.id);
                }
                ...
            },
            next
        );
    });
```

## 隐藏对未认证用户的功能

我们当然希望未认证用户能够访问我们应用程序的主页，但可能不希望向他们显示应用程序的所有功能。为了实现这一点，我们将更新我们的 index 路由，如下所示，来自`src/routes/index.js`：

```js
    router.get('/', function(req, res, next) {
 let userId = null;
 if (req.user) {
 userId = req.user.id;
 }

        Promise.all([gamesService.createdBy(userId),
                    gamesService.availableTo(userId),
                    usersService.getUsername(userId),
                    usersService.getRanking(userId),
                    usersService.getTopPlayers()])
            .then(results => {
                res.render('index', {
                            title: 'Hangman online',
 loggedIn: req.isAuthenticated(),
                            createdGames: results[0],
                            ...
                        });
                    })
            .catch(next);
    });
```

注意，这将在视图数据中添加一个`loggedIn`属性，而不是用户 ID。这个属性的值来自`isAuthenticated`函数，该函数由 Passport 添加到请求中。我们使用这个属性来隐藏对未认证用户不再起作用的特性，并从认证用户那里隐藏登录按钮。以下代码来自`src/views/index.hjs`：

```js
...
  <body>
    ...
 {{^loggedIn}}
      <form action="/auth/twitter" method="POST">
        <input type="submit" value="Log in using Twitter" />
      </form>
 {{/loggedIn}}
 {{#loggedIn}}
      <h3>Profile</h3>
      <form action="/profile" method="POST">    
        ...
      </form>
 {{/loggedIn}}
    <h2>Games</h2> 
 {{#loggedIn}}
      <form action="/games" method="POST" id="createGame">
        ...
      </form>
      <h3>Games created by you</h3>
      ...
 {{/loggedIn}}
    <h3>Games available to play</h3>
    ...
    <h2>Top players</h2>
    ...
    <h3>Lobby</h3>
    <form class="chat" data-room="lobby">
      <div id="messages"></dl>
 {{#loggedIn}}
        <input id="message"/><input type="submit" value="Send"/>
 {{/loggedIn}}
    </form>
  </body>
</html>
```

## 使用 Passport 进行集成测试

我们仍然有一个问题，即我们的集成测试将不再工作。现在只有登录用户可以创建游戏。写一个新的集成测试来检查 Twitter 认证是否工作将是一个好主意。但我们不想将 Twitter 账户依赖性引入当前的测试中。

相反，我们将利用 passport-local 策略来允许我们的测试登录。我们将将其作为开发依赖项安装，这样它就不会意外地在生产环境中运行：

```js
> npm install passport-local --save-dev

```

我们配置 Passport 接受任何用户名和密码。如果实际使用 passport-local，这就是你会在数据存储中检查凭据的地方。以下代码来自`src/config/passport.js`：

```js
if (process.env.NODE_ENV === 'test') {
    const LocalStrategy = require('passport-local');
    const uuid = require('uuid');
    passport.use(new LocalStrategy((username, password, done) => {
            const userId = uuid.v4();
            usersService.setUsername(userId, username)
                .then(() => {
                    done(null, { id: userId, name: username });
                });
        }
    ));
}
```

然后，我们在应用程序中添加一个新的本地认证端点，如下所示`src/app.js`：

```js
  if (process.env.NODE_ENV === 'test') {
    app.post('/auth/test',
      passport.authenticate('local', { successRedirect: '/' }));
  }
```

最后，更新我们的测试以登录为第一步，如下所示，来自`integration-test/game.js`的代码：

```js
    function withGame(word, callback) {        
 page.open(rootUrl + '/auth/test',
 'POST',
 'username=TestUser&password=dummy',
            function() {
                 ...
            }
        );
    }
```

# 允许用户注销

用户也会期望我们提供一种注销我们应用程序的方法。Passport 通过向请求添加一个`logout`函数来简化这一点。我们只需要在我们的其中一个路由中利用这个功能`src/routes/index.js`：

```js
    router.post('/logout', function(req, res){
        req.logout();
        res.redirect('/');
    });
```

我们可以在视图中添加一个注销按钮来利用这个新路由，如下所示`src/views/index.hjs`：

```js
    {{#loggedIn}}
 <form action="/logout" method="POST">
 <input type="submit" value="Log out" />
 </form>
      <h3>Profile</h3>
```

# 添加其他登录提供者

现在我们已经有了所有认证的一般基础设施，添加额外的提供者很容易。让我们以添加 Facebook 认证为例。首先，我们需要安装相关的 Passport 策略：

```js
> npm install passport-facebook --save

```

然后，我们可以更新我们的 Passport 配置文件，如下所示`src/config/passport.js`：

```js
...
const FacebookStrategy = require('passport-facebook').Strategy;

module.exports = (usersService) => {
 const providerCallback = providerName =>
 function(req, token, tokenSecret, profile, done) {
 usersService.getOrCreate(providerName, profile.id,
 profile.username || profile.displayName)
 .then(user => done(null, user), done);
 };

    if(process.env.TWITTER_API_KEY &&
            process.env.TWITTER_API_SECRET) {
        passport.use(new TwitterStrategy({
            consumerKey: process.env.TWITTER_API_KEY,
            consumerSecret: process.env.TWITTER_API_SECRET,
            callbackURL: '/auth/twitter/callback',
            passReqToCallback: true
 }, providerCallback('twitter')));
    }

 if(process.env.FACEBOOK_APP_ID &&
 process.env.FACEBOOK_APP_SECRET) {
 passport.use(new FacebookStrategy({
 clientID: process.env.FACEBOOK_APP_ID,
 clientSecret: process.env.FACEBOOK_APP_SECRET,
 callbackURL: '/auth/facebook/callback',
 passReqToCallback: true
 }, providerCallback('facebook')));
 }
    ...
};
```

在这里，我们已经将验证回调函数泛化，使其接受不同的提供者名称，然后使用它来处理 Twitter 和 Facebook 身份验证策略。我们可以以相同的方式重用此功能来添加更多策略。我们只需设置相关的环境变量，它们就可以工作。

### 注意

要获取 Facebook App ID 和密钥，请在[`developers.facebook.com/apps/`](https://developers.facebook.com/apps/)（需要您有一个 Facebook 账户）创建一个新的 Facebook 应用程序。这个过程与 Twitter 非常相似。只需创建一个类型为网站的新应用程序，其 URL 与您的开发环境相匹配（例如，`http://localhost:3000`）。创建后，App ID 和 App Secret 将在应用程序的仪表板页面上可见。

我们还需要将 Facebook 身份验证路由添加到我们的应用程序配置文件中。这些路由与对应的 Twitter 路由相同。与 Passport `config` 文件一样，我们可以通过参数化提供者名称来实现通用化。`src/app.js`中的代码如下：

```js
  app.use(sessions);
 const addAuthEndpoints = provider => {
 app.post(`/auth/${provider}`, passport.authenticate(provider));
 app.get(`/auth/${provider}/callback`,
 passport.authenticate(provider, { successRedirect: '/',
 failureRedirect: '/', session: true }));
 };
 addAuthEndpoints('twitter');
  addAuthEndpoints('facebook');
```

最后，我们需要添加一个按钮，允许用户使用 Facebook 登录。以下代码来自`src/views/index.hjs`：

```js
    {{^loggedIn}}
      <form action="/auth/twitter" method="POST">
        <input type="submit" value="Log in using Twitter" />
      </form>
 <form action="/auth/facebook" method="POST">
 <input type="submit" value="Log in using Facebook" />
 </form>
    {{/loggedIn}}
```

添加额外的提供者很容易。要添加 Google+身份验证，我们只需遵循以下步骤：

1.  安装`passport-google npm`模块

1.  按照描述在[`developers.google.com/identity/protocols/OpenIDConnect`](https://developers.google.com/identity/protocols/OpenIDConnect)创建一个新的应用程序。

1.  更新上述三个文件，将 Google 提供者传递给我们的新通用函数

# 摘要

在本章中，我们使用 Passport 在我们的 Express 应用程序中添加了身份验证，使用 Redis 作为会话存储引入了 Express 会话，利用多个 Passport 策略来支持不同的外部提供者，并在 Redis 中持久化用户数据。

这完成了我们的示例 Web 应用程序。在下一章中，我们将探讨如何创建不同类型的 Node.js 项目：一个库和一个命令行工具。
