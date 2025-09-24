# 第四章 实时通信

我们的应用程序开始成形。我们有一个项目列表和一个表单，允许我们添加、删除和更新项目。我们还能将这些仓库分配给这些项目，这样我们就可以查看项目中的所有仓库的问题/提交列表。本章将指导你完成客户端设置的下一阶段：使用 Redis 和 Socket.IO 实时显示项目仓库提交和问题列表。

我们理想情况下希望应用程序在 Socket.IO/Redis 关闭的情况下继续工作，这样应用程序就没有实时元素。我们将尝试考虑这些功能来实现这些特性。

# 使用 Redis 缓存数据

Redis 是一个极快、开源的内存键值存储。Redis 有一个有用的 Pub/Sub 机制，我们将使用它将消息推送到一个 Socket.IO 订阅者，该订阅者将向客户端发出事件。

访问此网站以下载和安装 Redis：[`redis.io/download`](http://redis.io/download)。

一旦 Redis 安装完成，你可以使用以下命令启动它：

```js
redis-server

```

为了启动 Redis 命令行界面，CLI 会发出以下命令：

```js
redis-cli

```

可以从 CLI 发出以下命令：

+   要监控 Redis 上的活动：

    ```js
    monitor

    ```

+   要清除 Redis 存储：

    ```js
    flushall

    ```

+   要查看 Redis 中存储的所有键：

    ```js
    keys *

    ```

+   要获取键的值：

    ```js
    get <key>

    ```

为了在我们的应用程序中使用 Redis，按照以下步骤安装 `node-redis` 客户端：

```js
npm install redis --save

```

让我们通过更新 `./lib/config/*.json` 配置文件来配置我们的应用程序以使用 Redis，如下所示：

```js
  "redis": {
    "port": 6379
  , "host": "localhost"
  }
```

首先，我们创建一个简单的模块，`Redis`，它封装了 Redis 连接 `./lib/cache/redis.js`。我们首先导入 `redis` 模块。我们定义一个 `Redis` 模块，它调用 `createClient` 来创建 Redis 客户端。

我们从前面拉取 Redis 配置数据：

```js
var redis = require('redis')
, config = require('../configuration');

function Redis() {
  this.port = config.get("redis:port");
  this.host = config.get("redis:host");
  this.password = config.get("redis:password");
  this.client = redis.createClient(this.port, this.host);
  if (this.password) this.client.auth(this.password, function() {});
}

module.exports = Redis;
```

让我们扩展我们的 `Redis` 模块并创建一个 `Publisher` 模块，该模块将使用 Redis Pub/Sub 功能发布消息，`./lib/cache/publisher/index.js`。我们首先导入我们的 `Redis` 模块，并使用 `util` 模块将 `Publisher` 模块扩展到 `Redis` 模块中。然后我们定义我们的 `Publisher` 模块，它包括一个 `save` 函数，该函数将对象作为字符串保存到 Redis 中，以及一个 `publish` 函数，该函数将消息发布到 Redis 中。

`Publisher` 模块的定义如下所示：

```js
var Redis = require('../../cache/redis')
  , util = require('util');

util.inherits(Publisher, Redis);

function Publisher() {
  Redis.apply(this, arguments);
};

Redis.prototype.save = function(key, items) {
  this.client.set(key, JSON.stringify(items));
};

Redis.prototype.publish = function(key, items) {
  this.client.publish(key, JSON.stringify(items));
};

module.exports = Publisher;
```

接下来，我们扩展我们的 `Redis` 模块并创建一个 `Subscribe` 模块 `./lib/cache/subscriber/index.js`，它消费发布的消息。我们首先导入我们的 `Redis` 模块，并使用 `util` 模块将 `Subscriber` 模块扩展到 `Redis` 模块中。然后我们定义我们的 `Subscriber` 模块，它包括一个 `subscribe` 函数。这允许用户订阅 `key` 上的消息：

```js
var Redis = require('../../cache/redis')
  , util = require('util');

util.inherits(Subscriber, Redis);

function Subscriber() {
  Redis.apply(this, arguments);
};

Subscriber.prototype.subscribe = function(key) {
  this.client.subscribe(key);
};

module.exports = Subscriber;
```

# 填充 Redis

`./lib/cache/populate.js`脚本使用我们前面的模块将新的提交/问题填充到 Redis 存储中。我们将在本章后面演示如何安排此脚本。我们首先导入`Publisher`模块，并使用`util.inherits`来扩展`Publisher`模块，添加一个`Populate`函数，使我们的`Populate`模块具有发布消息的能力。

我们然后定义`Populate`函数并添加一个`run`函数，该函数从 MongoDB 获取所有项目。我们使用`async.each`遍历每个项目，使用项目的`user`和`token`来实例化一个`GitHubRepo`模块。然后我们调用`git.commits`，传递一个`repositories`列表；返回的是按顺序排列的最近 10 个提交的列表。我们使用`project._id`作为键将响应保存到 Redis 中。然后我们通过`publish`函数发布`project._id`和`commits`，以激活刷新。然后我们重复整个过程以处理`issues`。

```js
var async =  require('async')
  , _ =  require('underscore')
  , util = require('util')
  , db = require('../db')
  , Publisher = require('../cache/publisher')
  , GitHubRepo = require('../github')
  , Project = require('../models').model('Project');

util.inherits(Populate, Publisher);

function Populate() {
  Publisher.apply(this, arguments);
};

Populate.prototype.run = function(callback) {
  var me = this;

  Project.find({}, function(error, projects) {
    if (error) callback();
    if (projects == null) callback();

    async.each(projects, function(project, callback) {
      var git = new GitHubRepo(project.token, project.user);

      git.commits(project.repositories, function(error, commits) {
        if (error || !commits) callback();

        me.save('commits:' + project._id, commits);
        me.publish('commits', { projectId : project._id, commits : commits});

        git.issues(project.repositories, function(error, issues) {
          if (error || !issues) callback();

          me.save('issues' + project._id, issues);
          me.publish('issues', { projectId : project._id, issues : issues});
        });
      });

      callback(error);
    }
    , function(error) {
      callback(error);
    });
  });
};
module.exports = Populate;
```

# Socket.IO

Socket.IO 是一个实时应用程序框架，它允许浏览器和服务器之间进行跨浏览器的实时通信。

由于浏览器和服务器对新兴的 WebSocket 标准的支持不足，我们无法轻松地在浏览器之间实现实时通信。为了实现这一点，Socket.IO 支持包括 WebSockets、长轮询、XHR 和 flashsockets 在内的多种传输协议，这些协议作为旧浏览器的后备机制。不支持 WebSockets 的浏览器将简单地回退到它们支持的传输协议。

Socket.IO 由两部分组成：服务器端模块和客户端脚本。为了使我们的应用程序支持双向全双工通信，两部分都需要安装。让我们通过 NPM 安装服务器部分：

```js
npm install socket.io --save

```

让我们通过更新我们的`./config/*.json`配置文件来配置我们的应用程序使用 Socket.IO，如下所示：

```js
"sockets": {
    "loglevel": 3
  , "pollingduration": 10
  , "browserclientminification" : false
  , "browserclientetag" : false
  , "browserclientgzip" : false
  }
```

下一步是将 Socket.IO 连接到 Express。让我们创建并配置一个典型的 Socket.IO 服务器：`./lib/socket/index.js`。我们定义我们的`Socket`模块，它接受一个参数：`server`。我们引入`socket.io`模块并创建一个新的 Socket.IO 服务器，将我们的 Express 启用 HTTP 服务器传递给它。然后我们通过设置合理的`日志级别`、`传输协议`和`轮询持续时间`来配置我们的 Socket.IO 服务器，这些值在之前的配置文件中已定义，并返回 Socket.IO 服务器。

```js
var config = require('../configuration');

function Socket(server) {
    var socketio = require('socket.io').listen(server);

    if (config.get('sockets:browserclientminification'))
      socketio.enable('browser client minification');
    if (config.get('sockets:browserclientetag'))
      socketio.enable('browser client etag');
    if (config.get('sockets:browserclientgzip'))
      socketio.enable('browser client gzip');
    socketio.set("polling duration",
      config.get('sockets:pollingduration'));
    socketio.set('log level', config.get('sockets:loglevel'));

    socketio.set('transports', [
        'websocket'
        , 'flashsocket'
        , 'htmlfile'
        , 'xhr-polling'
        , 'jsonp-polling'
    ]);

    return socketio;
};

module.exports = Socket;
```

设置`日志级别`对调试很有用。Socket.IO 支持以下几种：

+   `0`: 错误

+   `1`: 警告

+   `2`: 信息

+   `3`: 调试，默认为`3`

关于配置 Socket.IO 的更多信息可以在[`github.com/LearnBoost/Socket.IO/wiki/Configuring-Socket.IO`](https://github.com/LearnBoost/Socket.IO/wiki/Configuring-Socket.IO)找到。

现在我们使用我们的 Socket.IO 服务器并创建一个 Socket.IO 处理器`./lib/socket/handler.js`。

我们首先导入`Socket`模块，实例化它，并传递一个启用了 Express 的`httpServer`参数。我们创建一个 Redis `Subscriber`模块，并定义一个接受`httpServer`作为输入的`SocketHandler`函数。我们为连接事件设置一个 Socket.IO 处理程序。当准备就绪时，这将返回已连接的 socket。

然后，我们订阅两个 Redis 频道——`issues`和`commits`——并为新消息事件定义一个 Redis 处理程序。此处理程序将频道和消息广播到监听由`message.projectId`定义的频道的客户端。

我们定义一个 Socket.IO `subscribe`处理程序，允许客户端加入或订阅给定项目的事件。我们还定义了一个 Socket.IO `unsubscribe`处理程序，允许客户端离开或取消订阅给定项目的事件。我们还在 Socket.IO 上定义了一个`error`处理程序，将任何错误记录到`logger`：

```js
var http = require('http')
  , logger = require("../logger")
  , Socket = require('../socket')
  , Subscriber = require('../cache/subscriber')
  , subscriber = new Subscriber();

function SocketHandler(httpServer) {

  var socketIo = new Socket(httpServer)

  socketIo.sockets.on('connection', function(socket) {
    subscriber.subscribe("issues");
    subscriber.subscribe("commits");

    subscriber.client.on("message", function (channel, message) {
      socket.broadcast.to(message.projectId).emit(channel, JSON.parse(message));
    });

    socket.on('subscribe', function (data) {
      socket.join(data.channel);
    });

    socket.on('unsubscribe', function () {
      var rooms = socketIo.sockets.manager.roomClients[socket.id];

      for (var room in rooms) {
          if (room.length > 0) {
            room = room.substr(1);
            socket.leave(room);
          }
      }
    });
  });

  socketIo.sockets.on('error', function() {
    logger.error(arguments);
  });
};

module.exports = SocketHandler;
```

现在我们可以将 Socket.IO 连接到我们的`./lib/express/index.js` Express 服务器。让我们导入`SocketHandler`模块，将其传递给一个名为`httpServer`的 Express 服务器：

```js
, SocketHandler = require('../socket/handler')
..
var httpServer = http.createServer(app).listen(app.get('port'))
socketHandler = new SocketHandler(httpServer);
```

# 客户端的 Socket.IO

为了显示这些 Socket.IO 发布的消息，我们需要对客户端进行一些更改。让我们使用 bower 安装 Socket.IO 客户端组件：

```js
bower install socketio-client

```

让我们对`./lib/express/index.js` Express 服务器进行一次性的更改，并简化我们的`socket.io-client`位置，使用`static`中间件：

```js
app.use('/sockets', express.static('public/components/socket.io-client/dist/'));
```

我们现在将 Socket.IO 客户端脚本添加到`./views/index.html`：

```js
<script src="img/socket.io.js"></script>
```

现在我们将 Socket.IO 集成到我们的 backbone 组件中。让我们更新我们的`Backbone.js Router`。现在，路由`initialise`方法接受`socket`作为参数，并包含两个 Socket.IO 事件处理程序：一个用于问题，调用问题方法；一个用于提交，调用提交方法。`join`方法现在将发出一个 Socket.IO `unsubscribe`事件，取消用户对任何当前已订阅项目的订阅。然后它将发出一个 Socket.IO `subscribe`事件，将用户订阅到新选择的项目。所选项目通过`args`参数传递给`join`方法。

```js
Vision.Router = Backbone.Router.extend({
  projectListView : '',
  repositoryListView:'',
  issueListView:'',
  commitListView:'',
 socket: null,

  routes: {
    '' : 'index',
    'add' : 'add'
  },

  initialize : function(socket) {
 this.socket = socket;
    this.project();
    this.listenTo(this.projectListView, 'join', this.join);
 this.socket.on('issues', this.issues);
 this.socket.on('commits', this.commits);
  },

  join : function(args) {
    this.repository(args);
    this.issues(args);
    this.commits(args);
 this.socket.emit('unsubscribe');
 this.socket.emit('subscribe', {channel : args.projectId});
  },

  project : function() {
    this.projectListView = new Vision.ProjectListView();
  },

  repository : function(args) {

    this.repositoryListView = new Vision.RepositoryListView(
      {el: 'ul#repository-list', projectId: args.projectId,
        editMode: args.editMode });
  },

  issues : function(args) {

    this.issueListView = new Vision.IssueListView(
      {el: 'ul#issues-list', projectId: args.projectId,
        issues : args.issues});
  },

  commits : function(args) {

    this.commitListView = new Vision.CommitListView(
      { el: 'ul#commits-list', projectId: args.projectId,
        commits : args.commits});
  },

  index : function(){
    this.projectListView.render();
  },

  add : function(){
    this.projectListView.showForm();
  }
});
```

我们现在需要将我们的 Socket.IO 客户端实例传递给我们的`Router`。我们调用`io.connect`，创建一个 socket，并将其传递给我们的`Router`。

```js
Vision.Application = function() {
  this.start = function() {
 var socketio = io.connect('/');
    var router = new Vision.Router(socketio);
    Backbone.history.start();
    router.navigate('index', true);
  }
};
```

# 调度 Redis 人口统计

剩下的唯一事情是创建一个调度器，轮询我们的 Redis `populate`脚本，`./populate.js`。

首先，让我们通过 NPM 安装一个名为`node-schedule`的调度程序：

```js
npm install node-schedule --save
```

我们首先导入`node-schedule`，它允许我们进行类似于 cron 的调度。我们使用`*/5`每五分钟调用`schedule.scheduleJob`；然而，它也会在脚本启动时立即运行。然后我们调用`populate.run`来启动人口统计：

```js
var schedule = require('node-schedule')
  , logger = require('./lib/logger')
  , Populate = require('./lib/cache/populate')
  , populate = new Populate();

schedule.scheduleJob('*/5 * * * *', function() {
  populate.run(function(err) {
    if (err) logger.error('Redis Population error', err);
    if (!err) logger.info('Redis Population complete');
  });
});
```

为了使用实时更新运行应用程序，打开一个新的终端并运行以下命令：

```js
npm start

```

现在，打开另一个终端来运行 Redis 人口统计脚本。

```js
node populate.js

```

我们已将之前的脚本配置为每五分钟运行一次，因此请前往您的 GitHub 项目仓库添加一些问题/提交，以便查看结果。

# 摘要

Socket.IO 和 Redis 是强大的工具。我们几乎只是触及了它们所能实现的功能的表面。在本书的后续章节中，我们将重新探讨 Redis 和 Socket.IO，因为 Redis 也被用于扩展 Express 会话和 Socket.IOs 的 Pub/Sub 机制。

下一章将专注于在我们通过 GitHub 使用 Passport 实现身份验证策略并添加 SSL 支持时，如何确保我们的应用程序的安全。
