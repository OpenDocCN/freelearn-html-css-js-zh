

# 日志记录和监控您的应用程序

维护记录和指标在我们的开发周期中提供了许多优势。它们可以帮助我们提高应用程序的性能，在问题成为问题之前观察问题，并让我们深入了解应用程序的状态。日志记录和监控您的应用程序可以减少您开发（和调试）所需的时间，以及在整个项目过程中您所获得的头痛次数。日志记录是经常被忽视或最少考虑的事情，但它可能是在丢失一小时的正常运行时间或整个一天的正常运行时间之间做出差别的关键。

假设我们有一个应用程序，它只是将注册表单的详细信息插入到数据库表中。有一天，团队不小心将 `first_name` 列重命名为 `firstname`，现在没有新的记录被插入。使用日志记录，我们会看到类似“`first_name` 列不存在”类型的错误。这将帮助我们查看数据库的模式，并找出断开连接发生的地方（在这种情况下，我们删除下划线的错误）。

如果错误比这更复杂怎么办？我们的应用程序现在正在集群中运行，集群中的每个节点都只从其他节点接收独特的消息。偶尔，我们会注意到我们的表中缺少一些记录，而数据本身并没有明显的模式。使用日志机制，我们偶尔会看到“无法建立连接”的错误。我们可以双重检查我们的连接池管理（如果适用）或测试每个节点，看我们是否可以成功连接到数据库。在一个小型集群中，这不会是问题，但在一个大型系统中，这可能会变得繁琐且耗时。

为帮助管理更大集群中的应用程序提供一个解决方案，就是为您的应用程序的日志记录记录自定义（或添加）额外的上下文。元信息，如机器的标识符，可能有助于我们之前的例子。Sequelize 提供了一种使用 `options.logging` 参数自定义我们的日志记录的方法，并能够通过不同的方法调用更改日志记录行为。

在本章中，我们将涵盖以下主题：

+   使用所有可用接口配置日志记录

+   集成第三方日志记录应用程序，如 Pino 或 Bunyan

+   使用 OpenTelemetry 收集 Sequelize 的指标和统计数据

# 技术要求

您可以在此章节的代码文件[`github.com/PacktPublishing/Supercharging-Node.js-Applications-with-Sequelize/tree/main/ch8`](https://github.com/PacktPublishing/Supercharging-Node.js-Applications-with-Sequelize/tree/main/ch8)中找到

# 使用所有可用接口配置日志记录

Sequelize 为将日志集成到应用程序中提供了几个重载签名。默认行为是针对每个查询调用 `console.log`。以下是 Sequelize 将遵守的签名列表：

+   `function (msg) {}`

+   `function (...msg) {}`

+   `true`/`false`

+   `msg => someLogger.debug(msg)`

+   `someLogger.debug.bind(someLogger)`

如果我们想自定义 Sequelize 的日志记录行为，以下示例将快速介绍如何实现：

```js
function customLog(msg) {
    // insert into db/logger app here
    // ...
    // and output to stdout
    console.log(msg);
}
const sequelize = new Sequelize('sqlite::memory:', {
    logging: customLog
});
```

除了 Sequelize 将 SQL 查询发送到我们的 `customLog` 函数外，我们还提供了一个辅助方法，用于在需要记录查询之外的其他信息时。Sequelize 实例提供了一个 `log` 方法，可以像下面这样调用：

```js
sequelize.log('this will send our message to customLog as 
well');
```

如果您的 Sequelize 实例的 `benchmark` 参数设置为 `true`，那么 Sequelize 将在消息末尾添加查询完成的总耗时。使用我们之前的示例，日志条目可能看起来类似如下：

```js
Executed (default): SELECT * FROM ...; Elapsed time: 136ms
```

有时，我们可能需要记录日志查询、查询对象、适用参数或其他任何形式的元数据。Sequelize 将识别这种支持的扩展模式：

```js
function multiLog(...msgs) {
    msgs.forEach(function(msg) {
        console.log(msg);
    });
}
const sequelize = new Sequelize('sqlite::memory:', {
    logging: multiLog
});
```

我们现在可以调用 Sequelize 实例的 `log` 方法，该方法将参数发送到我们的 `multiLog` 函数，如下所示：

```js
sequelize.log('error', 'custom error message', Date.now(), { id: 100 });
```

由于 `multiLog` 函数的行为，这将每个参数打印到其自己的换行符上。

日志参数还可以接受布尔值。`true` 值将合并为 Sequelize 的默认行为（`console.log`）。将值设置为 `false` 将完全禁用日志记录并取消任何日志调用。以下示例将阻止 Sequelize 记录查询：

```js
const sequelize = new Sequelize('sqlite::memory:', {
    logging: false
});
```

注意

日志记录的 `true` 值被认为是过时的，并且不推荐省略日志记录值以实现默认行为或使用 `console.log` 作为参数的值。

Sequelize 还可以通过每个可查询方法的日志参数（例如，`findAll`、`update` 和 `create`）限制日志记录到特定的查询。例如，如果我们想禁用特定查询的日志记录，我们可以通过将以下查询的 `logging` 参数设置为 `false` 来实现：

```js
sequelize.findAll({
  where: {
    id: 1
  }
}, {
  logging: false
});
```

注意

您还可以利用 Sequelize 对 debug NPM 包的使用来查看查询的日志输出。通过设置环境变量为 `DEBUG=sequelize:sql*`，您的终端应显示 Sequelize 执行的查询。

# 集成第三方日志应用，如 Pino 或 Bunyan

如果我们的应用程序已经使用第三方日志应用，Sequelize 可以提供对这些系统进行集成的支持。本节引用了两个日志应用，Pino 和 Bunyan，但任何日志库或框架都应与 Sequelize 兼容。

## 与 Pino 集成

Pino 是一个低开销的 Node.js 日志记录器，它还提供编辑、传输和异步支持。假设我们的项目在 `node_modules` 文件夹中安装了 Pino，我们可以简单地将其与 Sequelize 实例集成，如下所示：

```js
const logger = require('pino')();
const sequelize = new Sequelize('sqlite::memory:', {
    logging: (msg) => logger.debug(msg)
});
```

现在，当我们手动调用 `sequelize.log` 或执行查询时，日志将被发送到 Pino 日志库。输出将类似于以下内容：

```js
{"level":30,"time":1650118644700,"pid":5363,"hostname":"MacBook-Pro-4.local","msg":"Executing (default): SHOW INDEX FROM `Airplanes` FROM `airline`"}
```

关于 Pino 的更多信息，您可以参考项目的仓库[`github.com/pinojs/pino`](https://github.com/pinojs/pino)。

## 与 Bunyan 集成

有时，日志框架需要在将其绑定到 Sequelize 之前进行中间步骤。Bunyan 框架就是这样一个例子。Bunyan 是一个专注于提供序列化和流方法的日志框架。集成此框架看起来类似于以下内容：

```js
const bunyan = require('bunyan');
const logger = bunyan.createLogger({name: 'app'});
const sequelize = new Sequelize('sqlite::memory:', {
    logging: (msg) => logger.info(msg)
});
```

上述示例显示了 Bunyan 日志与 Sequelize 的输出：

```js
{"name":"app","hostname":"MacBook-Pro-4.local","pid":6014,"level":30,"msg":"Executing (default): SHOW INDEX FROM `Airplanes` FROM `airline`","time":"2022-04-16T14:33:13.083Z","v":0}
```

关于 Bunyan 的更多信息，您可以参考项目的仓库[`github.com/trentm/node-bunyan`](https://github.com/trentm/node-bunyan)。

从 Pino 和 Bunyan 的示例中，我们可以看到添加日志框架已经解决了我们独特的机器标识符、错误发生的时间和紧急研究。通过查看日志，现在应该更容易筛选出在集群或应用程序中发生错误的任何地方。

我们现在可以完成在 Avalon Airlines 项目中集成日志框架的工作。从项目的根目录开始，我们需要安装必要的包：

```js
npm i pino
```

在 `models/index.js` 中，查看以下行：

```js
const Sequelize = require('sequelize/core');
```

使用常量导出下面的 Pino 框架：

```js
const logger = require('pino')();
```

导出常量后，请查看此行：

```js
const db = {};
```

在下面，我们可以将日志参数添加到 `config` 对象中，如下所示：

```js
config.logging = (msg) => logger.info(msg);
```

现在，我们的应用程序支持使用 Pino 日志框架进行自定义日志。

# 使用 OpenTelemetry 收集 Sequelize 的指标和统计数据

OpenTelemetry 是一个用于收集、聚合和度量各种统计、指标、跟踪和日志的标准规范。OpenTelemetry 可以帮助我们识别瓶颈可能发生的位置，对日志进行分类和应用拓扑过滤器，并连接到第三方应用程序（例如，用于警报监控）。

要将 OpenTelemetry 与 Sequelize 集成，我们将在 Avalon Airlines 项目中安装以下包：

```js
npm i @opentelemetry/api @opentelemetry/sdk-trace-node @opentelemetry/instrumentation @opentelemetry/sdk-node @opentelemetry/auto-instrumentations-node opentelemetry-instrumentation-sequelize
```

在 `models/index.js` 中，在 `'use strict';` 行下方，我们现在可以添加我们的新包：

```js
const { NodeTracerProvider } = 
    require(‹@opentelemetry/sdk-trace-node');
const { registerInstrumentations } = 
    require('@opentelemetry/instrumentation');
const { SequelizeInstrumentation } = 
    require(‹opentelemetry-instrumentation-sequelize');
```

在 `let sequelize;` 行的上方，我们可以添加跟踪提供者，这将注册正确的 Sequelize OpenTelemetry 插件：

```js
const tracerProvider = new NodeTracerProvider({
  plugins: {
    sequelize: {
      // disabling the default/old plugin is required
      enabled: false,
      path: ‹opentelemetry-plugin-sequelize'
    }
  }
});
```

在 `traceProvider` 声明块下方，我们可以将提供者与 Sequelize 仪器规范关联：

```js
registerInstrumentations({
  tracerProvider,
  instrumentations: [
    new SequelizeInstrumentation({
      // any custom instrument options here
    })
  ]
});
```

注意

你可以在[`github.com/aspecto-io/opentelemetry-ext-js/tree/master/packages/instrumentation-sequelize`](https://github.com/aspecto-io/opentelemetry-ext-js/tree/master/packages/instrumentation-sequelize)找到关于 Sequelize 仪表化的额外参考和选项参数。

在 Avalon Airlines 项目的根目录下，创建一个名为 `tracing.js` 的文件，并包含以下代码：

```js
const opentelemetry = require("@opentelemetry/sdk-node");
const { getNodeAutoInstrumentations } = 
    require(«@opentelemetry/auto-instrumentations-node");
const sdk = new opentelemetry.NodeSDK({
  traceExporter: new opentelemetry.tracing.
      ConsoleSpanExporter(),
      instrumentations: [getNodeAutoInstrumentations()]
});
sdk.start();
```

现在，我们可以使用以下命令调用我们的应用程序：

```js
node -r "./tracing.js" index.js
```

之后，打开浏览器访问项目的 URL（默认为 `http://localhost:3000/`）并刷新页面几次。几秒钟后，你应该能在你的终端看到一些类似以下的事件：

```js
{
  traceId: '7c25880d655f67e5d8e15b83129dc95e',
  parentId: '934dc0ed012f6e37',
  name: ‹SELECT›,
  id: ‹af16347a3fbbf923›,
  kind: 2,
  timestamp: 1650124004289597,
  duration: 1616,
  attributes: {
    ‹db.system': 'mysql',
    ‹net.peer.name›: ‹127.0.0.1›,
    ‹net.peer.port': 3306,
    ‹db.connection_string':'jdbc:mysql://127.0.0.1:3306/
         airline›,
    ‹db.name›: ‹airline›,
    ‹db.user': 'root',
    ‹db.statement': 'SELECT `id`, `planeModel`, 
        `totalSeats`, `createdAt`, `updatedAt` FROM 
        `Airplanes` AS `Airplane`;›
  },
  status: { code: 0 },
  events: []
}
```

传统上，应用程序会将这些数据导出到收集器，如 Zipkin ([`zipkin.io/`](https://zipkin.io/))、Jaeger ([`www.jaegertracing.io/`](https://www.jaegertracing.io/)) 或 Prometheus ([`prometheus.io/`](https://prometheus.io/))。有关如何关联应用程序的遥测数据的说明，请参阅此教程：https://opentelemetry.io/docs/instrumentation/js/exporters/。

如果你打算使用 Zipkin 作为你的收集器，那么在 `models/index.js` 文件中的 `const tracerProvider = new NodeTracerProvider({)` 块下，我们将替换这一行：

```js
provider.addSpanProcessor(new BatchSpanProcessor(new 
    ZipkinExporter()))
```

我们需要将其替换为以下内容：

```js
tracerProvider.addSpanProcessor(new BatchSpanProcessor(new 
    ZipkinExporter()));
```

这将指示我们的跟踪提供程序将跟踪和日志导出到 Zipkin 导出器（可以同时使用多个导出器）。

# 摘要

在本章中，我们探讨了配置 Sequelize 日志的不同重载签名。我们还学习了如何在我们的 Node.js 应用程序中集成第三方框架，例如 OpenTelemetry。

在下一章中，我们将介绍如何将插件或适配器集成到我们的 Sequelize 实例中。下一章也将演示如何创建我们自己的适配器。
