# 第七章：生产

在本章中，我们将讨论将 Express 应用程序投入生产。我们首先通过查看异常处理使我们的 Express 应用程序更加健壮。然后我们看一下为了使应用程序能够在生产环境中生存，我们需要进行的一系列性能改进。

# 错误处理、域和仅崩溃设计

Node 社区采用了仅崩溃的设计模式，这简单意味着：如果你遇到未捕获的异常，捕获它，记录它，然后重启进程。仅崩溃设计和域作为一个模式工作得相当好，尤其是如果你的应用程序正在使用`cluster`。让我们在`vision-core`上的`cluster`模块`./lib/cluster/index.js`中做一些更改，这里我们包含了`domain`模块；而不是简单地包含我们的模块以在集群中运行，我们创建了一个域并调用`run`方法。然后我们包含了一个基于域的`error`处理器，它通过`process.exit(1)`记录并关闭进程。集群的`exit`处理器将捕获这个信号并`fork`一个新的进程：

```js
var cluster = require('cluster')
, http = require('http')
, numCPUs = require('os').cpus().length
, logger = require('../logger')
, domain = require('domain');

function Cluster() {}

Cluster.prototype.run = function(module) {
  if (cluster.isMaster) {
    for (var i = 0; i < numCPUs; i++) {
      cluster.fork();
    }

    cluster.on('exit', function(worker, code, signal) {
      logger.info('Worker ' + worker.process.pid + ' died');
      cluster.fork();
    });
  } else {
    var d = domain.create();

    d.on('error', function(err) {
      logger.info('Error ', err);
      process.exit(1);
    });

    d.run(function() {
      require(module);
    });
  }
}

module.exports = Cluster;
```

# Redis 会话

在生产环境中，大多数需要会话支持的 Express 应用程序可能会使用 Redis，因此使 Redis 性能良好非常重要。我们的 Redis 客户端`node-redis`使用纯 JavaScript 解析器；node-redis 文档建议使用一个替代模块进行解析。

Hiredis 是对官方 Hiredis C 库的绑定；它是非阻塞的且速度快。如果你安装了`hiredis`，node-redis 将默认使用它。让我们在`vision-core`上安装 Hiredis：

```js
cd vision-core
npm install hiredis redis --save

```

# SSL 终止

**SSL 终止**是指将 TLS 加密（HTTPS）流解密为纯文本（HTTP）。Node 核心中的 TLS 模块不如一些用于终止 SSL 的其他技术快，通常不用于生产。我们的应用程序完全运行在 HTTPS 上，因此 TLS 性能至关重要。

幸运的是，我们有 SSL 选项；我们将使用`stud`，这是一个网络代理，它终止 TLS/SSL 连接并将未加密的流量转发到 Web 服务器。Stud 基于`libev`构建，是非阻塞的；它被设计用来在多核机器上高效地处理数万个连接。让我们克隆 stud 的 GitHub 仓库：

```js
git clone http://github.com/bumptech/stud.git

```

现在从源代码编译 stud：

```js
cd stud
make
sudo make install

```

安装完成后，我们可以生成一个 stud 文件。Stud 附带一个默认配置，我们可以通过以下方式请求：

```js
cd vision-web
stud --default-config > stud.conf

```

我们的研究文件`./vision-web/stud.conf`需要一些重要的更改才能正常工作；`frontend`配置应设置为端口`8443`，而`backend`配置应设置为托管在端口`3003`上的 Hipache 负载均衡器`vision-web`。最后，我们设置`pem-file`，这是一个包含 SSL 证书和私钥的单个 PEM 文件：

```js
# stud(8), The Scalable TLS Unwrapping Daemon's configuration

# Listening address. REQUIRED.
# type: string
# syntax: [HOST]:PORT
frontend = "[127.0.0.1]:8443"

# Upstream server address. REQUIRED.
# type: string
# syntax: [HOST]:PORT.
backend = "[127.0.0.1]:3003"

# SSL x509 certificate file. REQUIRED.
# List multiple certs to use SNI. Certs are used in the order they
# are listed; the last cert listed will be used if none of the others match
# type: string
pem-file = "lib/secure/vision.pem"

# EOF
```

现在我们已经设置了 stud 配置，我们的 Hipache 负载均衡器将不再需要终止 SSL。让我们从 Hipache 配置`./vision-web/config/server.json`中移除 SSL 配置：

```js
{
"server": {
    "accessLog": "hipache_access.log",
    "port": 3000,
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

在配置就绪后，让我们创建一个带有私钥的单个 PEM 文件证书。

简单地将您的`cert.pem`和`key.pem`复制到一个名为`./lib/secure/vision.pem`的单个文件中；首先放置私钥，然后是您的证书。

现在，我们可以在 Hipache 负载均衡器前面运行 stud；stud 将处理 SSL 并将未加密的流量按如下方式定向到 Hipache：

```js
cd vision-web
stud --config=stud.conf

```

请运行以下命令集以在 stud 后面运行我们的堆栈：

```js
/vision-web/hipache --config ./config/server-no-ssl.json
/vision-api/hipache --config ./config/server.json

redis-cli (these may already exist in redis)
rpush frontend:web.vision.net web.vision
rpush frontend:web.vision.net http://127.0.0.1:3003
rpush frontend:api.vision.net api.vision
rpush frontend:api.vision.net http://127.0.0.1:3005

/vision-web/NODE_ENV=production PORT=3003 npm start
/vision-api/NODE_ENV=production PORT=3005 npm start
/vision-worker/npm start

```

# 缓存

我们的静态文件需求很小；我们提供的唯一静态内容将是应用客户端使用的组件。为了缓存我们的静态文件/组件，让我们对`vision-web/lib/express/index.js`进行简单的修改。我们将`maxAge`属性设置为一周，并将其存储在配置中，如下所示：

```js
app.use(express.static('public',
  { maxAge: config.get('express:staticCache') }));
app.use(express.static('public/components',
  { maxAge: config.get('express:staticCache') }));
app.use('/bootstrap',express.static('public/components/bootstrap/docs/assets/css',
  { maxAge: config.get('express:staticCache') }));
app.use('/sockets',
  express.static('public/components/socket.io-client/dist/', { maxAge: config.get('express:staticCache') }));
```

让我们将配置值`staticCache`添加到`vision-web/config/*.json`中，如下所示：

```js
  "express": {
    "port": 8443,
    "staticCache" : 6048000000
  },
```

现在我们访问我们的应用时，响应头将包含一个缓存控制头。如果您访问我们应用的首页并通过浏览器工具检查提供的任何资源的响应头，您应该看到以下内容：

```js
Cache-Control:public, max-age = 86400

```

# 网站图标

让我们使用`connect.favicon`中间件为我们的应用添加一个网站图标。从性能角度来看，这有一些价值，因为我们可以缓存它。此外，即使不存在图标，您的浏览器也会请求它，这可能导致抛出 404 错误。我们将使用现有的`staticCache`配置值来设置图标的`maxAge`。让我们编辑 Express 服务器`/vision-web/lib/express/index.js`并添加`favicon`中间件：

```js
app.set('views', 'views');
app.use(express.favicon('public/components/vision/favicon.ico'), { maxAge: config.get('express:staticCache') });
```

## 压缩

我们可以通过压缩我们的静态资源来提高页面加载时间。我们将通过安装以下两个 grunt 任务来压缩我们的 JavaScript 和 CSS 文件：

`grunt-contrib-uglify`：这允许您压缩 JavaScript 文件：

```js
npm install grunt-contrib-uglify --save-dev

```

`grunt-contrib-cssmin`：这允许您压缩 CSS 文件：

```js
npm install grunt-contrib-cssmin --save-dev

```

让我们将这些压缩任务添加到我们的 grunt 文件中，如下所示：

```js
grunt.loadNpmTasks('grunt-contrib-uglify');
grunt.loadNpmTasks('grunt-contrib-cssmin');

uglify: {
  dist: {
    files: {
      'public/components/vision/templates.min.js':
      'public/components/vision/templates.js',
      'public/components/vision/vision.min.js':
      'public/components/vision/vision.js',
      'public/components/json2/json2.min.js':
      'public/components/json2/json2.js',
      'public/components/handlebars/handlebars.runtime.min.js':
      'public/components/handlebars/handlebars.runtime.js'
   }
 }
  },
  cssmin: {
    minify: {
      expand: true,
      src: ['public/components/vision/vision.css'],
      ext: '.min.css'
    }
  }
```

让我们运行以下命令：

```js
grunt uglify
grunt cssmin

```

并非我们所有的 JavaScript 组件都有压缩版本，因此我们也对这些组件进行压缩，为 json2 和 handlebars 添加`.min`版本。

# 压缩

我们可以通过压缩静态文件进一步提高页面加载时间。Express 包括`compress`中间件，它将 gzip HTTP 响应。让我们编辑 Express 服务器`/vision-web/lib/express/index.js`并添加`compress`中间件，如下所示：

```js
app.set('views', 'views');
app.use(express.logger({ immediate: true, format: 'dev' }));
app.use(express.compress());
```

如果您访问我们应用的首页并检查通过浏览器工具提供的所有资源的响应头，您应该看到以下内容：

```js
Content-Encoding: gzip

```

# 记录日志

Express 服务器，`./lib/express/index.js`，使用 `logger` 中间件进行日志记录。Express 日志记录器应仅在开发环境中使用。实际上，在生产环境中，这将对性能产生严重影响，因为控制台函数是同步的。让我们更改 Express 服务器，并在生产时关闭日志，如下面的代码片段所示：

```js
if (process.env['NODE_ENV'] !== "production")
 app.use(express.logger({ immediate: true, format: 'dev' }));
```

# 摘要

在商业生产环境中，Express 可能看起来有些不同，但这是有充分理由的。许多 Express/Node 支持的任务可以由其他工具更好地执行。在我们的应用程序中，我们尽量保持在 node 堆栈上；我们选择使用 stud 终止 SSL，因为我们的整个应用程序都运行在 SSL 上。Stud 将在这个领域超越所有其他工具，包括 Nginx 和 Haproxy。Stud 将将未加密的响应转发到 Hipache，以平衡负载。Hipache 基于 node-http-proxy；它使用集群进行故障转移。更重要的是，与 node-http-proxy 不同，它可以管理内存，使其成为负载均衡器的合理选择。

Hipache 工作得很好，但如果你真正寻求性能，Nginx 和 Haproxy 是事实上的工具选择。对于故障转移，我们使用 node 的集群模块，结合域名，使我们的应用程序更加健壮。

我们的静态文件需求很小，因此我们选择通过 Express 提供静态资源，包括缓存、压缩和精简。任何偏离这些最小需求的行为都会让我选择 Nginx 或 Haproxy 来提供静态内容，或者使用内容分发网络。

我们已经成功自动化了许多任务。我们的代码覆盖率大约在 80%，在我们的应用程序上运行 YSlow 和 PageSpeed 产生了良好的结果。理想情况下，我们希望通过测试驱动所有需求，通过单元测试驱动一些较小的代码模块，并使用 Cucumber 添加更多验收测试。我希望你至少已经能够感受到所有这些元素，并能够根据自己的判断做出关于测试的明智选择。

Node/Express 堆栈是构建 Web 应用程序的一个非常好的平台。与全栈 JavaScript 一起工作是一种非常好的开发体验。Node 社区和数千名 Node 模块开发者使 Node 成为一个充满活力和有趣的领域来工作。
