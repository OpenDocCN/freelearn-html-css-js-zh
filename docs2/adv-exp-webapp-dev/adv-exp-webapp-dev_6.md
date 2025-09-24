# 第六章。扩展

在本章中，我们将探讨扩展 Express 的选项。我们的当前解决方案无法扩展到单个进程/服务器；引入一些简单的更改将允许我们水平扩展和垂直扩展 Vision。我们还将查看一种替代的 Web 架构，并检查解耦我们的应用程序如何改进我们的应用程序并帮助我们进一步扩展 Express。

# 使用 Redis 扩展 Express 会话

将 `NODE_ENV` 设置为 `production` 运行我们的 Express 应用程序将输出以下消息：

```js
NODE_ENV=production npm start

Warning: connection.session() MemoryStore is not
designed for a production environment, as it will leak
memory, and obviously only work within a single process.

```

Express 的默认会话存储是一个内存存储；将会话绑定到单个进程无法扩展。

此外，如果服务器崩溃，我们将丢失那些会话。如果我们想将 Express 应用程序扩展到多个服务器，我们需要一个与 Express 应用程序解耦的内存存储。Express 有几个可选的存储方案；在这里，我们将使用 `connect-redis` 通过 Redis。让我们配置视觉应用程序以使用 Redis 作为会话存储。

```js
npm install connect-redis ––save

```

现在，我们将对 Express 服务器 `./lib/express/index.js` 进行一些更改。我们首先引入我们之前创建的 `Redis` 模块，该模块配置并连接到 Redis 服务器。我们将其中一个实例化为 `redis`。然后我们引入 `connect-redis`，它返回 `RedisStore`。

```js
, Redis = require('../cache/redis')
, redis = new Redis()
, RedisStore = require('connect-redis')(express);
```

我们已经有一个现有的 `sessionStore`，配置为使用 `MemoryStore`：

```js
var sessionStore = new express.session.MemoryStore();
```

让我们用新的 `RedisStore` 替换它：

```js
var sessionStore = new RedisStore({client: redis.client});
```

我们的应用程序现在可以使用 Redis 存储会话。我们可以通过运行以下命令来通过 `redis-cli` 监控 Redis 会话活动：

```js
redis-cli
monitor

```

# 使用 Redis 扩展 Socket.IO

Socket.IO 也使用内存存储来存储其事件。这里有几个问题；第一个是如果服务器失败，我们将丢失存储在内存中的那些消息。第二个是如果我们尝试通过添加更多服务器来扩展我们的应用程序，Socket.IO 的内存存储将绑定到单个服务器；我们添加的服务器将不知道其他服务器上哪些 Socket.IO 连接是打开的。

我们可以通过使用 Socket.IO 的 `RedisStore` 解决这些问题。我们首先引入一个 `RedisStore`，它是 Socket.IO 命名空间中的 `redis` 模块。我们还可以使用视觉的 `Redis` 模块来创建三个 Redis 客户端：`pub`、`sub` 和 `client`。为了配置 Socket.IO 使用 `RedisStore`，我们将 Socket.IO 的 `'store'` 设置为 `RedisStore`，并将 `redis`、`pub`、`sub` 和 `client` 作为参数传递。

```js
var config = require('../configuration')
, RedisStore = require('socket.io/lib/stores/redis')
, redis  = require('socket.io/node_modules/redis')
, Redis = require('../cache/redis')
, pub    = new Redis().client
, sub    = new Redis().client
, client = new Redis().client;

function Socket(server) {
    /....

 socketio.set('store', new RedisStore({
 redis    : redis
 , redisPub : pub
 , redisSub : sub
 , redisClient : client
 }));

    return socketio;
};
```

# 水平扩展 Express

我们当前的应用程序架构将 API、消费型网络客户端和填充 Redis 缓存的工人耦合在一起。这种方法适用于许多应用程序，并将在负载均衡器的帮助下允许其水平扩展。

但比如说，如果我们希望我们的 API 支持除了 Web 以外的客户端，比如说，我们引入了一个使用我们 API 的移动客户端；理想情况下，我们希望独立扩展我们的 API 并移除与 Web 客户端相关的所有内容。

水平扩展我们的工作器意味着简单地重复相同的工作，这将是毫无意义的。稍后，我们将讨论如何扩展工作器。

在本章的剩余部分，我们将概述如何拆分我们的应用程序以实现水平扩展。我们将使用`vision`应用程序的`chapter-6`版本的源代码。当然，我们将记录任何有助于实现我们目标的有兴趣的内容。我们将创建四个新的项目：`vision-core`、`vision-web`、`vision-api`和`vision-worker`。

### 小贴士

您可以在此处下载本章的源代码：

[`github.com/AndrewKeig/vision-core`](https://github.com/AndrewKeig/vision-core)

[`github.com/AndrewKeig/vision-web`](https://github.com/AndrewKeig/vision-web)

[`github.com/AndrewKeig/vision-api`](https://github.com/AndrewKeig/vision-api)

[`github.com/AndrewKeig/vision-worker`](https://github.com/AndrewKeig/vision-worker)

## vision-core

我们的首要任务是提取`vision-web`、`vision-api`和`vision-worker`项目之间可以共享的所有内容，并将其放入一个新的`vision-core`项目中。

这包括以下部分：`./cache`、`./lib/configuration`、`./lib/db`、`./lib/github`、`./lib/logger`、`./lib/models`和`./lib/project`。

`vision-core`项目不是一个应用程序，所以我们从项目的根目录中移除了所有内容，包括`./app.js`和我们的`./gruntfile.js`，并添加了一个`./index.js`文件，该文件简单地导出了所有显示的功能：

```js
module.exports.redis = require('./lib/cache/redis');
module.exports.publisher = require('./lib/cache/publisher');
module.exports.subscriber = require('./lib/cache/subscriber');
module.exports.configuration = require('./lib/configuration');
module.exports.db = require('./lib/db');
module.exports.github = require('./lib/github');
module.exports.project = require('./lib/project');
module.exports.logger = require('./lib/logger');
module.exports.models = require('./lib/models');
```

为了与其他私有项目共享私有的`vision-core`项目，我们在配置中添加了一个 GitHub 依赖项：`./config/packge.json`：

```js
  "dependencies": {
    "vision-core": "git+ssh://git@github.com:AndrewKeig/vision-core.git#master",
```

## vision-api

让我们创建一个包含 Web API 的`vision-api`项目。在这里，我们需要重用与 API 相关的所有内容，包括以下中间件：`./lib/middleware/id`、`./lib/middleware/notFound`、`./lib/routes/project`、`./lib/routes/github`和`./lib/routes/heartbeat`的路线。我们还包含了`./config`配置文件和所有测试`./test`。

为了确保`vision-api`的安全性，我们将使用基本身份验证，它使用用户名和密码来验证用户。这些凭据以纯文本形式传输，因此建议您使用 HTTPS。我们已经向您展示了如何设置 HTTPS，因此这部分将不会重复。为了设置基本身份验证，我们可以使用`passport-http`；让我们安装它：

```js
npm install passport-http ––save

```

我们首先将用户名和密码添加到`./config/*.json`中：

```js
  "api": {
    "username": "airasoul",
    "password": "1234567890"
  }
```

现在，我们准备在 `./lib/auth/index.js` 中实现一个 `ApiAuth` 策略。我们首先定义一个函数 `ApiAuth`，然后导入 `passport` 和 `passport-http` 模块。我们实例化一个 `BasicStrategy` 函数并将其添加到 `passport` 中，传递一个验证函数。在这个验证函数内部，我们有通过在回调中传递 `false` 来拒绝用户的选择。我们调用 `findUser` 并检查 `username` 和 `password` 是否与存储在 `./config/*.json` 中的相同。

```js
var config = require('vision-core').configuration;

function ApiAuth() {
  this.passport = require('passport');
  var BasicStrategy = require('passport-http').BasicStrategy;

  this.passport.use(new BasicStrategy({
  },
    function(username, password, done) {
      findUser(username, password, function(err, status) {
        return done(null, status);
      })
    }  
  ));

  var findUser = function(username, password, callback){
    var usernameOk = config.get('api:username') === username;
    var passwordOk = config.get('api:password') === password;
    callback(null, usernameOk === passwordOk);
  }
};
module.exports = new ApiAuth();
```

`vision-api` 项目需要一个新的 Express 服务器 `./express/index.js`。我们首先通过 `vision-core` 引入 `config`，然后引入处理身份验证的 `apiAuth` 模块。接着，我们使用 `app.all` 将 passport 基本中间件应用到所有路由上。我们将 `session:false` 设置为 `false`，因为基本身份验证是无状态的。

```js
var express = require('express')
  , http = require('http')
  , config = require('vision-core').configuration
  , db = require('vision-core').db
  , apiAuth = require('../auth')
  , middleware = require('../middleware')
  , routes = require('../routes')
  , app = express();

app.set('port', config.get('express:port'));
app.use(express.logger({ immediate: true, format: 'dev' }));
app.use(express.bodyParser());
app.use(apiAuth.passport.initialize());
app.use(app.router);

app.all('*', apiAuth.passport.
  authenticate('basic', { session: false }));
app.param('id', middleware.id.validate);
app.get('/heartbeat', routes.heartbeat.index);
app.get('/project/:id', routes.project.get);
app.get('/project', routes.project.all);
app.post('/project', routes.project.post);
app.put('/project/:id', routes.project.put);
app.del('/project/:id', routes.project.del);
app.get('/project/:id/repos', routes.github.repos);
app.get('/project/:id/commits', routes.github.commits);
app.get('/project/:id/issues', routes.github.issues);
app.use(middleware.notFound.index);

http.createServer(app).listen(app.get('port'));
module.exports = app;
```

由于我们正在转向多个 Express 服务器以支持我们的应用程序，我们将 `vision-api` 移至端口 `3001`。让我们在 `./config/*.json` 中配置它，如下面的代码所示：

```js
 "express": {
    "port": 3001
  }
```

## vision-worker

让我们继续并创建一个新的项目 `vision-worker`，它包括两个脚本 `./populate.js` 和 `./lib/cache/populate.js`。

当然，我们可以使用 **RabbitMQ** 这样的东西来扩展这个工作进程。这将允许我们产生多个生产者和消费者，从这个角度来看，我们目前的解决方案并不是最优的。如果您有兴趣改进这个应用程序的这部分，请参阅 Packt 的 *Instant RabbitMQ Message Application Development*。这本书解释了如何使用 RabbitMQ 实现工作模式。

## vision-web

最后，我们创建一个新的项目 `vision-web`，它将包括与网络客户端相关的所有内容；简单地将 `第六章` 中的所有内容包含进来，并从 `core` 中移除所有内容，从 `./package.json` 中引用 `core`。我们当前的 `routes` 需要一些重大的更改；现在，我们已经将服务层解耦到其自己的存储库中，称为 `vision-api`。`vision-web` 将不再直接调用项目中的服务调用和 github 服务；这些服务现在存在于 `vision-api` 项目中，我们将调用在 `vision-api` 上公开的 API 服务。

让我们将配置添加到 `./config/*.json` 中，以供我们的 `vision-api` 项目使用。`vision-api` 项目已配置为在端口 `3001` 上运行，并使用基本身份验证来保证安全，因此我们在 `url` 中包含了 `username` 和 `password`。

```js
  "api": {
    "url":  "http://airasoul:1234567890@127.0.0.1:3001"
  }
```

为了在我们的 `vision-api` 项目上调用服务，我们将通过使用 `Request` 模块来简化操作。Request 是一个简单的客户端，允许我们发起 HTTP 请求；让我们安装它：

```js
npm install request --save

```

在配置就绪后，我们转向我们的项目路由 `./lib/routes/project.js`。在这里，我们只是用 vision-api 中的相应调用替换了对我们项目服务的所有调用。我们首先引入上面代码片段中定义的配置。每个路由使用此配置构造一个 URL，我们使用 Request 模块调用 API。我们返回一个响应，该响应由 `response.statusCode` 和响应体组成：

```js
var logger = require('vision-core').logger
, S = require('string')
, config = require('vision-core').configuration
, request = require('request')
, api = config.get('api:url');

exports.all = function(req, res){
  logger.info('Request.' + req.url);

  var userId = req.query.user || req.user.id;
  var url = api + '/project?user=' + userId ;

  request.get(url, function (error, response, body) {
    return res.json(response.statusCode, JSON.parse(body));
  });
};
exports.get = function(req, res){
  logger.info('Request.' + req.url);

  var url = api + '/project/' + req.params.id;

  request.get(url, function (error, response, body) {
    return res.json(response.statusCode, JSON.parse(body));
  });
};

exports.put = function(req, res){
  logger.info('Put.' + req.params.id);

  if (S(req.body.name).isEmpty() )
  return res.json(400, 'Bad Request');

  var url = api + '/project/' + req.params.id;

  request.put(url, { form: req.body },
  function (error, response, body) {
    return res.json(response.statusCode, body);
  });
};

exports.post = function(req, res){
  logger.info('Post.' + req.body.name);

  if (S(req.body.name).isEmpty() )
  return res.json(400, 'Bad Request');

  var url = api + '/project/';

  request.post(url, { form: req.body },
  function (error, response, body) {   
    var parsed = JSON.parse(body);
    res.location('/project/' +  parsed._id);
    return res.json(response.statusCode, parsed);
  });
};

exports.del = function(req, res){
  logger.info('Delete.' + req.params.id);

  var url = api + '/project/' + req.params.id;

  request.del(url, function (error, response, body) {
    return res.json(response.statusCode, body);
  });
};
```

让我们为 GitHub 路由 `./lib/routes/github.js` 重复相同的流程；移除对 GitHub 服务的调用，并替换为对我们 `vision-api` 项目相应端点的调用：

```js
var logger = require('vision-core').logger
, config = require('vision-core').configuration
, request = require('request')
, api = config.get('api:url');

exports.repos = function(req, res){
  logger.info('Request.' + req.url);

  var url = api + '/project/' + req.params.id + "/repos";

  request.get(url, function (error, response, body) {
    return res.json(response.statusCode, JSON.parse(body));
  });
};

exports.commits = function(req, res){
  logger.info('Request.' + req.url);

  var url = api + '/project/' + req.params.id + "/commits";

  request.get(url, function (error, response, body) {
    return res.json(response.statusCode, JSON.parse(body));
  });
};

exports.issues = function(req, res){
  logger.info('Request.' + req.url);

  var url = api + '/project/' + req.params.id + "/issues";

  request.get(url, function (error, response, body) {
    return res.json(response.statusCode, JSON.parse(body));
  });
};
```

让我们更新我们的测试 `./test/project.js`、`./test/github.js`。我们现在移除与 Mongoose 相关的所有内容，使用 `Request` 模块直接调用 vision-api 以将测试数据种入 MongoDB：

```js
beforeEach(function(done){
  var proj = {
    name: "test name"
    , user: login.user  
    , token: login.token
    , image: "/img/"
    , repositories    : [ "node-plates" ]
  };
  var url = api + '/project';

  req.post(url, { form: proj },
    function (error, response, body) {
      id = JSON.parse(body)._id;
      done()
  });    
});

afterEach(function(done){
  var url = api + '/project/' + id;

  req.del(url, function (error, response, body) {   
    done()
  });    
});
```

# 使用集群进行垂直扩展

我们当前的 `vision-web` 和 `vision-api` Express 应用程序在单个线程中运行。为了垂直扩展我们的应用程序，为了利用多核系统，并在出现故障时提供冗余，我们可以使用集群模块并将负载分散到多个进程上。让我们将 Cluster 模块添加到 vision-core `./lib/cluster/index.js`：

```js
var cluster = require('cluster')
, http = require('http')
, numCPUs = require('os').cpus().length
, logger = require('../logger');

function Cluster() {}

Cluster.prototype.run = function(module){
  if (cluster.isMaster) {
    for (var i = 0; i < numCPUs; i++) {
      cluster.fork();
    }

    cluster.on('exit', function(worker, code, signal) {
      logger.info('Worker ' + worker.process.pid + ' died');
    });
  } else {
   require(module);
  }
}

module.exports = Cluster;
```

让我们将集群模块从 `vision-core` 中导出；通过在 `./index.js` 中添加以下内容：

```js
module.exports.cluster = require('./lib/cluster');  
```

让我们更改 `vision-web` 和 `vision-api ./app.js` 中的 Express 应用程序，并添加一个运行我们应用程序的第三个选项，即使用集群支持运行：

```js
switch (process.env['NODE_ENV']) {
  case 'COVERAGE':
    module.exports = require('./lib-cov/express');
    break;
  case 'TEST':
    module.exports = require('./lib/express');
    break;
  default:
 var Cluster = require('vision-core').cluster
 , cluster = new Cluster();
 cluster.run(__dirname + '/lib/express');
    break;
}
```

# 使用 Hipache 进行负载均衡

**Hipache** 是一个设计用于路由大量 HTTP 和 WebSocket 流量的分布式代理。Hipache 支持通过 Redis 进行动态配置，因此更改配置和添加虚拟主机不需要重启。基于 node-http-proxy 库，Hipache 提供了对负载均衡 WebSocket、SSL、检测后端死亡以及集群以实现故障转移的支持。让我们安装它：

```js
npm install hipache -g

```

让我们通过编辑 `hosts` 文件为 `vision-web` 和 `vision-api` 设置一个主机：

```js
sudo nano /private/etc/hosts

```

添加两个新条目：

```js
127.0.0.1  web.vision.net
127.0.0.1  api.vision.net

```

然后刷新缓存以使这些更改生效：

```js
dscacheutil -flushcache

```

为了配置服务器，我们需要为每个我们想要进行负载均衡的应用程序创建一个配置文件。在我们的例子中，是 `vision-web` 和 `vision-api`。以下是 `vision-api` 的配置文件，`./config/server.json`。重要的是，我们正在端口 `8443` 上运行 `vision-api`。我们在 HTTPS 部分配置了一个 SSL 证书，因为 Hipache 会终止 SSL，而不是我们的 Express 服务器：

```js
{
    "server": {
        "accessLog": "hipache_access.log",
        "port": 8443,
        "workers": 5,
        "maxSockets": 100,
        "deadBackendTTL": 30,
        "address": ["127.0.0.1"],
        "address6": ["::1"],
        "https": {
            "port": 8443,
            "key": "lib/secure/key.pem",
            "cert": "lib/secure/cert.pem"
        }
    },
    "redisHost": "127.0.0.1",
    "redisPort": 6379,
    "redisDatabase": 0
}
```

让我们对 Express 服务器 `./lib/express/server.js` 进行更改，并在生产环境中返回一个标准的 HTTP 服务器；Hipache 现在将终止 SSL。

```js
function Server(app){
 if (process.env['NODE_ENV'] === "PRODUCTION")
 return http.createServer(app).listen(app.get('port'));

  var httpsOptions = {
    key: fs.readFileSync('./lib/secure/key.pem'),
    cert: fs.readFileSync('./lib/secure/cert.pem')
  };

  return https.createServer(httpsOptions,app).listen(app.get('port'));
}
```

现在我们为 `vision-api ./config/server.json` 添加 Hipache 配置。请注意，我们正在端口 `3001` 上运行 `vision-api`。

```js
{
    "server": {
        "accessLog": "hipache_access.log",
        "port": 3001,
        "workers": 5, 
        "maxSockets": 100,
        "deadBackendTTL": 30,
        "address": ["127.0.0.1"],
        "address6": ["::1"]
    },
    "redisHost": "127.0.0.1",
    "redisPort": 6379,
    "redisDatabase": 0
}
```

我们需要重新访问 GitHub，并将 `settings/applications/developer applications/vision` 下的 URL 更改为 `https://web.vision.net:8443`。

让我们更新 `vision-web` 的配置 `./config/*.json`，并将 GitHub 认证 URL 更改为 `web.vision.net`。

```js
  "auth": {
    "homepage": "https://web.vision.net:8443"
  , "callback": "https://web.vision.net:8443/auth/github/callback"
  , "clientId": "5bb691b4ebb5417f4ab9"
  , "clientSecret": "44c16f4d81c99e1ff5f694a532833298cae10473"
  }
```

让我们也更新同一组配置文件中的 API `url` 配置：

```js
  "api": {
    "url":  "http://airasoul:1234567890@api.vision.net:3001"
  }
```

我们最后的更改将使我们能够支持每个应用程序的多个端口；我们将更改 Express 服务器中的端口设置 `./lib/express/index.js`，以便它检查 `process.env.PORT` 以获取端口号：

```js
app.set('port', process.env.PORT || config.get('express:port'));
```

我们现在开始运行应用程序在负载均衡器下的过程。为了启动 `vision-api` 的 Hipache 负载均衡器，运行以下命令：

```js
cd vision-web
hipache --config ./config/server.json

```

为了启动 `vision-web` 的 Hipache 负载均衡器，我们运行以下命令：

```js
cd vision-api
hipache --config ./config/server.json

```

因此，我们现在为 `vision-api` 和 `vision-web` 运行了一个正在运行的 Hipache 实例。让我们在 Redis 中创建一个虚拟主机，并将 Hipache 实例与一系列服务器关联。现在运行 redis 命令行界面：

```js
redis-cli

```

首先，让我们让 `vision-web` 应用程序运行起来，并将运行在端口 `3003` 的后端分配给 `web.vision`：

```js
rpush frontend:web.vision.net web.vision
rpush frontend:web.vision.net http://127.0.0.1:3003

```

让我们回顾 `web.vision` 的配置：

```js
lrange frontend:web.vision.net 0 -1

```

让我们让 `vision-api` 应用程序运行起来，并将运行在端口 `3005` 的后端分配给 `api.vision`：

```js
rpush frontend:api.vision.net api.vision
rpush frontend:api.vision.net http://127.0.0.1:3005

```

让我们回顾 `api.vision` 的配置：

```js
lrange frontend:api.vision.net 0 -1

```

让我们在负载均衡器下运行应用程序，设置 `PORT` 环境变量，并在运行 `npm start` 时将 NODE_ENV 设置为 `production`：

```js
/vision-web/NODE_ENV=production PORT=3003 npm start
/vision-api/NODE_ENV=production PORT=3005 npm start
/vision-worker/npm start

```

我们现在有一个在负载均衡器下运行的视觉应用，请访问 `https://web.vision.net:844` `3`。为了向我们的负载均衡器添加更多后端，让我们在另一个端口下启动 `vision-api` 和 `vision-web`：

```js
/vision-web/NODE_ENV=production PORT=3004 npm start
/vision-api/NODE_ENV=production PORT=3006 npm start

```

当我们运行以下命令时，运行在端口 `3004` 和 `3006` 的后端将被添加到负载均衡器中：

```js
rpush frontend:web.vision.net http://127.0.0.1:3004
rpush frontend:api.vision.net http://127.0.0.1:3006

```

# 摘要

扩展 Web 应用程序并非易事。Node；使用集群模块允许我们垂直扩展。水平扩展需要我们向更广泛的社区寻求帮助。在我们的应用程序中，我们选择了 Hipache；一个基于 Node 的负载均衡器。在下一章中，我们将讨论在考虑性能和可靠性问题时，我们可以对应用程序进行的生产级改进。
