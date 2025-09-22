# 优化性能

在本章中，将涵盖以下食谱：

+   调优函数即服务

+   批量请求

+   利用异步非阻塞 IO

+   在流处理器中分组事件

+   自动扩展 DynamoDB

+   使用缓存控制

+   利用会话一致性

# 简介

云原生将性能测试、调优和优化颠倒过来。许多利用的完全管理的无服务器云服务具有隐式可伸缩性。这些服务按请求购买，并将自动扩展以满足峰值和意外的需求。对于这些资源，进行前期性能测试的必要性要小得多；相反，我们根据在生产环境中进行的持续测试收集到的信息进行优化，如第七章优化可观察性中所述，并基于收集到的信息持续调优。我们还利用持续部署来推动必要的改进。这种基于价值的开发方法有助于确保我们专注于最高价值的改进。

尽管如此，还有一些资源，如某些流处理器和数据存储，严重依赖于显式定义的批量大小和读写容量。对于关键服务，这些资源必须得到充分分配，以确保峰值数据处理量不会使其过载。因此，本章的食谱将专注于在设计和发展过程中值得提前应用的性能优化技术，以帮助确保服务不会相互对抗。

# 调优函数即服务

调优函数与传统服务调优非常不同，因为可调整的旋钮很少。函数的短生命周期也带来了许多影响，使得传统的技术和框架成为反模式。以下食谱解释了一个常见的内存错误，讨论了传统语言和库选择对冷启动的影响，并展示如何使用**webpack**打包 JavaScript 函数以最小化下载时间。

# 如何做...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch9/tuning-faas --path cncb-tuning-faas
```

1.  导航到`cncb-tuning-faas`目录，`cd cncb-tuning-faas`。

1.  查看以下内容的`serverless.yml`文件：

```js
service: cncb-tuning-faas

plugins:
  - serverless-webpack

provider:
  memorySize: 1024 # default
  ...

package:
  individually: true
custom:
 webpack:
   includeModules: true

functions:
  save:
    memorySize: 512 # function specific
    ...
```

1.  查看以下内容的`webpack.config.js`文件：

```js
const slsw = require('serverless-webpack');
const nodeExternals = require("webpack-node-externals");
const path = require('path');

module.exports = {
  entry: slsw.lib.entries,
  output: {
    libraryTarget: 'commonjs',
    path: path.join(__dirname, '.webpack'),
    filename: '[name].js'
  },
  optimization: {
    minimize: false
  },
  target: 'node',
  mode: slsw.lib.webpack.isLocal ? "development" : "production",
  externals: [
    nodeExternals()
  ],
  module: {
    rules: [
      {
        test: /\.js$/,
        use: [
          {
            loader: 'babel-loader',
          }
        ],
        include: __dirname,
        exclude: /node_modules/
      }
    ]
  }
};
```

1.  使用`npm install`安装依赖项。

1.  使用`npm test`运行测试。

1.  查看`.serverless`目录中生成的内容，注意每个函数的 ZIP 文件，然后解压每个文件以查看其内容。

1.  部署堆栈，`npm run dp:lcl -- -s $MY_STAGE`。

1.  在 AWS 控制台中查看堆栈和资源，并在 Lambda 控制台中注意代码大小。

1.  最后，一旦完成`npm run rm:lcl -- -s $MY_STAGE`，请删除堆栈。

# 它是如何工作的...

我们调整**函数即服务**（**FaaS**）的最明显旋钮是`memorySize`分配。这个设置也驱动着价格计算。不幸的是，价格的相关性往往具有反直觉性，可能会导致增加成本的同时降低性能的决策。AWS Lambda 的价格机制是这样的：如果你将内存分配加倍，但相应地将执行时间减半，价格保持不变。由此推论，如果你将内存分配减半，相应地将执行时间加倍，你将花费相同数量的钱，但性能却更低。这是因为内存分配实际上与函数执行的机器实例大小相关。更多的内存分配意味着函数也将拥有更快的 CPU 和更多的网络 IO 吞吐量。简而言之，*不要在内存分配上节省*。

函数即服务的主要担忧和混淆来源是冷启动时间。对于异步函数，例如处理 Kinesis 流，冷启动时间并不那么令人担忧。这是因为冷启动频率较低，因为函数通常在每个分片上重复使用数小时。然而，最小化 API 网关后面同步函数的冷启动时间非常重要，因为需要启动多个函数来适应并发负载。因此，冷启动时间可能会影响最终用户的使用体验。

影响冷启动时间的第一个因素是必须下载到容器中的函数包的大小。这也是为什么分配更多内存和更多的网络 IO 吞吐量可以提高函数性能的一个原因。同时，最小化必须下载的包的大小也很重要。我们将在稍后讨论如何使用 webpack 来优化 JavaScript 函数。

影响冷启动时间的下一个因素是语言或运行时的选择。脚本语言，如 JavaScript 和 Python，在启动时做的工作很少，因此对冷启动时间的影响很小。相反，Java 必须在启动时执行工作来加载类和准备 JVM。随着类数量的增加，对冷启动的影响也增加。这导致下一个影响冷启动时间的因素：库和框架的选择，例如**对象关系映射**（**ORM**）和依赖注入框架，以及连接池库。这些库和框架通常在启动时做大量工作，因为它们被设计成在长时间运行的服务器上工作。

在使用 Java 编写的函数中，FaaS 开发者面临的一个常见问题是提高 Spring 和 Hibernate 函数的冷启动时间；然而，这些工具最初并不是为 FaaS 设计的。我使用 Java 编程已经超过 20 年，从它在 1990 年代初出现时开始。起初我对转向 JavaScript 表示怀疑，但这个食谱证明了它与云原生和无服务器架构的契合度。然而，值得注意的是，多语言编程是最好的策略；为特定服务使用合适的编程语言，但了解在使用 Faas 时其带来的影响。

为了最小化 JavaScript 函数的包大小，我们利用 webpack，原因与我们在浏览器中用于最小化下载的原因相同。Webpack 执行摇树优化，移除未使用的代码以减小包大小。在 `serverless.yml` 文件中，我们包括 `serverless-webpack` 插件并配置它以单独打包函数。单独打包函数使我们能够最大化摇树优化的好处。`webpack.config.js` 文件进一步控制打包过程。`serverless-webpack` 插件提供了 `slsw.lib.entries` 工具，这样我们就不需要重复函数名称来定义所有的 `entry` 点。我们还关闭了 `minimize` 功能，这会使代码变得难以阅读。我们这样做是为了避免包含源映射进行调试，这会显著增加包大小。我们还排除了 `node_modules` 文件夹中的所有外部库，并配置插件为 `includeModules`，这包括那些实际用作运行时的库。一个特殊的例外是 `aws-sdk` 模块，因为它已经在函数容器中可用，所以永远不会被包含。最终结果是包含仅必要内容的精简函数包。

# 批处理请求

流处理器的设计必须考虑到它将接收的数据量。数据应实时处理，处理器不应落后。以下菜谱演示了如何使用 DynamoDB 批写入来确保足够的吞吐量。

# 准备工作

在开始此菜谱之前，您需要一个 AWS Kinesis Stream，例如在 *创建事件流* 菜谱中创建的那个。

# 如何做到这一点...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch9/frp-batching --path cncb-frp-batching
```

1.  导航到 `cncb-frp-batching` 目录，`cd cncb-frp-batching`。

1.  查看名为 `serverless.yml` 的文件。

1.  查看名为 `handler.js` 的文件，其内容如下：

```js
module.exports.listener = (event, context, cb) => {
  _(event.Records)
    .map(recordToUow)
    .filter(forPurple)
    .ratelimit(Number(process.env.WRITE_CAPACITY) /
      Number(process.env.SHARD_COUNT) /
      Number(process.env.WRITE_BATCH_SIZE) / 10, 100)
    .batch(Number(process.env.WRITE_BATCH_SIZE))
    .map(batchUow)
    .flatMap(batchWrite)
    .collect().toCallback(cb);
};

const batchUow = batch => ({ batch });

const batchWrite = batchUow => {
  batchUow.params = {
    RequestItems: {
      [process.env.TABLE_NAME]: batchUow.batch.map(uow =>
        ({
          PutRequest: {
            Item: uow.event
          }
        })
      )
    },
  };

  ...

  return _(db.batchWrite(batchUow.params).promise()
    .then(data => (
      Object.keys(data.UnprocessedItems).length > 0 ?
        Promise.reject(data) :
        batchUow
    ))
    .catch(err => {
      err.uow = batchUow;
      throw err;
    })
  );
};
```

1.  使用 `npm install` 安装依赖项。

1.  使用 `npm test -- -s $MY_STAGE` 运行测试。

1.  查看在 `.serverless` 目录中生成的内容。

1.  使用以下命令部署堆栈，`npm run dp:lcl -- -s $MY_STAGE`。

1.  在 AWS 控制台中查看堆栈和资源。

1.  使用以下命令调用 `simulate` 函数：

```js
$ sls invoke -f simulate -r us-east-1 -s $MY_STAGE
[
    {
        "total": 3850,
        "orange": 942,
        "purple": 952,
        "blue": 1008,
        "green": 948
    }
]
```

1.  查看以下 `listener` 函数日志：

```js
$ sls logs -f listener -r us-east-1 -s $MY_STAGE --filter 'event count' 2018-08-04 23:46:53 ... event count: 100
2018-08-04 23:46:57 ... event count: 1000
2018-08-04 23:47:23 ... event count: 250
2018-08-04 23:47:30 ... event count: 1000
2018-08-04 23:47:54 ... event count: 1000
2018-08-04 23:48:18 ... event count: 500$ sls logs -f listener -r us-east-1 -s $MY_STAGE --filter 'Duration' REPORT ... Duration: 3688.65 ms ...
REPORT ... Duration: 25869.08 ms ...
REPORT ... Duration: 7293.39 ms ...
REPORT ... Duration: 23662.65 ms ...
REPORT ... Duration: 24752.11 ms ...
REPORT ... Duration: 13983.72 ms ...

$ sls logs -f listener -r us-east-1 -s $MY_STAGE --filter 'retries' ...
2018-08-04 23:48:20 ... [AWS dynamodb 200 0.031s 0 retries] batchWriteItem({ RequestItems:
   { 'john-cncb-frp-batching-things':
      [ { PutRequest:
           { Item:
              { id: { S: '320e6023-9862-11e8-b0f6-01e9feb460f5' },
                type: { S: 'purple' },
                timestamp: { N: '1533440814882' },
                partitionKey: { S: '5' },
                tags: { M: { region: { S: 'us-east-1' } } } } } },
        { PutRequest:
           { Item:
              { id: { S: '320e6025-9862-11e8-b0f6-01e9feb460f5' },
                type: { S: 'purple' },
                timestamp: { N: '1533440814882' },
                partitionKey: { S: '1' },
                tags: { M: { region: { S: 'us-east-1' } } } } } },
        ...
        [length]: 10 ] },
  ReturnConsumedCapacity: 'TOTAL',
  ReturnItemCollectionMetrics: 'SIZE' })
...
```

1.  最后，使用 `npm run rm:lcl -- -s $MY_STAGE` 删除堆栈。

# 它是如何工作的...

如果流处理器接收一批 1,000 个事件，它是否需要向数据库发出 1,000 次请求或只需 100 次请求才能执行得更快？答案当然取决于许多变量，但一般来说，通过网络进行更少的调用更好，因为它最小化了网络延迟的影响。为此，像 DynamoDB 和 Elasticsearch 这样的服务提供了 API，允许在单个请求中提交命令批处理。在这个配方中，我们使用 DynamoDB 的`batchWrite`操作。为了准备一个批处理，我们只需在管道中添加一个`batch`步骤并指定`WRITE_BATCH_SIZE`。这种性能提升非常简单添加，但重要的是要记住，批处理请求会增加 DynamoDB 写入容量消耗的速度。因此，有必要在`ratelimit`计算中包含`WRITE_BATCH_SIZE`并相应地增加写入容量，正如在*实现背压和速率限制*配方中讨论的那样。

另一个需要注意的重要事项是，这些批处理请求不被视为单个事务。某些命令可能在单个请求中成功，而其他命令可能失败；因此，有必要检查响应中的`UnprocessedItems`，这些项目需要重新提交。在这个配方中，我们将每个批处理视为一个**工作单元**（**uow**）并引发整个批次的故障，正如在*处理故障*配方中讨论的那样。在调整逻辑以仅重试失败的命令之前，这是一个良好的、安全的地方。请注意，最终，你只有在尝试了最大重试次数后才会引发故障。

# 利用异步非阻塞 IO

流处理器的设计必须考虑到它将接收的数据量。数据应实时处理，处理器不应落后。以下配方演示了如何利用异步、非阻塞 IO 并行处理数据，以确保足够的吞吐量。

# 如何操作...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch9/frp-async-non-blocking-io --path cncb-frp-async-non-blocking-io
```

1.  导航到`cncb-frp-async-non-blocking-io`目录，`cd cncb-frp-async-non-blocking-io`。

1.  检查名为`serverless.yml`的文件。

1.  检查以下内容的`handler.js`文件：

```js
module.exports.listener = (event, context, cb) => {
  _(event.Records)
    .map(recordToUow)
    .filter(forPurple)
    .ratelimit(Number(process.env.WRITE_CAPACITY) /
      Number(process.env.SHARD_COUNT) / 
      Number(process.env.PARALLEL) / 10, 100)
    .map(put)
    .parallel(Number(process.env.PARALLEL))
    .collect().toCallback(cb);
};
```

1.  使用`npm install`安装依赖项。

1.  使用`npm test -- -s $MY_STAGE`运行测试。

1.  检查`.serverless`目录中生成的内容。

1.  部署堆栈`npm run dp:lcl -- -s $MY_STAGE`。

1.  在 AWS 控制台中检查堆栈和资源。

1.  使用以下命令调用`simulate`函数：

```js
$ sls invoke -f simulate -r us-east-1 -s $MY_STAGE
[
    {
        "total": 4675,
        "blue": 1136,
        "green": 1201,
        "purple": 1167,
        "orange": 1171
    }
]
```

1.  查看以下`listener`函数日志：

```js
$ sls logs -f listener -r us-east-1 -s $MY_STAGE --filter 'event count' 2018-08-05 00:03:05 ... event count: 1675
2018-08-05 00:03:46 ... event count: 1751
2018-08-05 00:04:34 ... event count: 1249$ sls logs -f listener -r us-east-1 -s $MY_STAGE --filter 'Duration' REPORT ... Duration: 41104.28 ms ...
REPORT ... Duration: 48312.47 ms ...
REPORT ... Duration: 31450.13 ms ...

$ sls logs -f listener -r us-east-1 -s $MY_STAGE --filter 'retries' ...
2018-08-05 00:04:58.034 ... [AWS dynamodb 200 0.024s 0 retries] ...
2018-08-05 00:04:58.136 ... [AWS dynamodb 200 0.022s 0 retries] ...
2018-08-05 00:04:58.254 ... [AWS dynamodb 200 0.034s 0 retries] ...
2018-08-05 00:04:58.329 ... [AWS dynamodb 200 0.007s 0 retries] ...
2018-08-05 00:04:58.430 ... [AWS dynamodb 200 0.007s 0 retries] ...
2018-08-05 00:04:58.540 ... [AWS dynamodb 200 0.015s 0 retries] ...
2018-08-05 00:04:58.661 ... [AWS dynamodb 200 0.035s 0 retries] ...
2018-08-05 00:04:58.744 ... [AWS dynamodb 200 0.016s 0 retries] ...
2018-08-05 00:04:58.843 ... [AWS dynamodb 200 0.014s 0 retries] ...
2018-08-05 00:04:58.953 ... [AWS dynamodb 200 0.023s 0 retries] ...
...
```

1.  最后，一旦你完成了`npm run rm:lcl -- -s $MY_STAGE`，请移除堆栈。

# 它是如何工作的...

*异步非阻塞 I/O* 对于最大化吞吐量至关重要。没有它，流处理器将阻塞并无所事事，直到外部调用完成。这个配方演示了如何使用 `parallel` 步骤来控制可以执行的并发调用数量。作为一个例子，我曾经有一个从 S3 读取的脚本，处理时间超过一个小时，但一旦我添加了一个设置为 16 的 `parallel` 步骤，脚本只用了五分钟就执行完毕。改进如此显著，Datadog 几乎立即联系我，看看我们是否有失控的进程。

为了允许并发调用，我们只需在外部调用步骤之后将 `parallel` 步骤添加到管道中，并指定 `PARALLEL` 数量。这种性能改进非常简单添加，但重要的是要记住，并行请求会增加 DynamoDB 写入容量消耗的速度。因此，有必要在 `ratelimit` 计算中包含 `PARALLEL` 数量，并相应地增加写入容量，正如在 *实现背压和速率限制* 配方中讨论的那样。通过结合并行执行、分组和批量，可以实现进一步的性能改进。

# 流处理器中的事件分组

流处理器的设计必须考虑到它将接收的数据量。数据应实时处理，处理器不应落后。以下配方演示了如何通过在流中将相关数据分组来确保足够的吞吐量。

# 如何操作...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch9/frp-grouping --path cncb-frp-grouping
```

1.  导航到 `cncb-frp-grouping` 目录，`cd cncb-frp-grouping`。

1.  查看名为 `serverless.yml` 的文件。

1.  查看以下内容的 `handler.js` 文件：

```js
module.exports.listener = (event, context, cb) => {
  _(event.Records)
    .map(recordToUow)
    .filter(forPurple)
    .group(uow => uow.event.partitionKey)
    .flatMap(groupUow)
    .ratelimit(Number(process.env.WRITE_CAPACITY) /
      Number(process.env.SHARD_COUNT) / 10, 100)
    .flatMap(put)
    .collect().toCallback(cb);
};

const groupUow = groups => _(Object.keys(groups).map(key => ({ batch: groups[key]})));

const put = groupUow => {
  const params = {
    TableName: process.env.TABLE_NAME,
    Item: groupUow.batch[groupUow.batch.length - 1].event, // last
  };
  ...
  return _(db.put(params).promise()
    .then(() => groupUow)
  );
};
```

1.  使用 `npm install` 安装依赖。

1.  使用 `npm test -- -s $MY_STAGE` 运行测试。

1.  查看在 `.serverless` 目录中生成的内容。

1.  部署堆栈 `npm run dp:lcl -- -s $MY_STAGE`。

1.  在 AWS 控制台中查看堆栈和资源。

1.  使用以下命令调用 `simulate` 函数：

```js
$ sls invoke -f simulate -r us-east-1 -s $MY_STAGE
[
    {
        "total": 4500,
        "blue": 1134,
        "green": 1114,
        "purple": 1144,
        "orange": 1108
    }
]
```

1.  查看以下 `listener` 函数日志：

```js
$ sls logs -f listener -r us-east-1 -s $MY_STAGE --filter 'event count' 2018-08-05 00:28:19 ... event count: 1000
2018-08-05 00:28:20 ... event count: 1000
2018-08-05 00:28:21 ... event count: 650
2018-08-05 00:28:22 ... event count: 1000
2018-08-05 00:28:23 ... event count: 850$ sls logs -f listener -r us-east-1 -s $MY_STAGE --filter 'Duration' REPORT ... Duration: 759.50 ms ...
REPORT ... Duration: 611.70 ms ...
REPORT ... Duration: 629.91 ms ...
REPORT ... Duration: 612.90 ms ...
REPORT ... Duration: 623.11 ms ...

$ sls logs -f listener -r us-east-1 -s $MY_STAGE --filter 'retries' 2018-08-05 00:28:20.197 ... [AWS dynamodb 200 0.112s 0 retries] ...
2018-08-05 00:28:20.320 ... [AWS dynamodb 200 0.018s 0 retries] ...
...
2018-08-05 00:28:23.537 ... [AWS dynamodb 200 0.008s 0 retries] ...
2018-08-05 00:28:23.657 ... [AWS dynamodb 200 0.019s 0 retries] ...
```

1.  最后，一旦完成，使用 `npm run rm:lcl -- -s $MY_STAGE` 删除堆栈。

# 它是如何工作的...

*批量请求* 配方演示了如何通过将多个不相关的命令批量到一个请求中，以最小化网络 I/O 的开销。另一种减少网络 I/O 的方法是简单地减少需要执行的命令数量，通过分组相关事件，并且每组只执行一个命令。例如，我们可能对每个组进行计算，或者只是采样一些数据。在这个配方中，我们根据 `partitionKey` 对事件进行分组。我们可以根据事件中的任何数据进行分组，但最佳结果是在分组相对于分区键时实现；这是因为分区键确保相关事件被发送到同一个分片。

`group` 步骤使得根据事件的内容将相关事件归组变得简单。对于更复杂的逻辑，可以直接使用 `reduce` 步骤。接下来，我们将每个组映射到一个必须成功或失败一起的工作单元（`groupUow`），正如在“处理故障”配方中讨论的那样。最后，如前例所示，我们写入每个组的最后一个事件。从日志中注意，分组导致写入次数显著减少；对于这次特定的运行，模拟了 4,500 个事件，但只有 25 次写入。通过结合分组、批处理和并行调用，可以进一步提高性能。

# 自动扩展 DynamoDB

云原生，使用 FaaS 和无服务器架构，可以最小化支持服务层的基础设施扩展所需的努力。然而，我们现在需要关注调整流处理器并最小化对目标数据存储的任何限制。以下配方演示了如何使用 DynamoDB 自动扩展来确保分配足够的容量以提供足够的吞吐量并避免限制。

# 如何操作...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch9/dynamodb-autoscaling --path cncb-dynamodb-autoscaling
```

1.  导航到 `cncb-dynamodb-autoscaling` 目录，`cd cncb-dynamodb-autoscaling`。

1.  使用以下内容审查名为 `serverless.yml` 的文件：

```js
service: cncb-dynamodb-autoscaling

...

plugins:
  - serverless-dynamodb-autoscaling-plugin

custom:
  autoscaling:
    - table: Table
      write:
        minimum: 5
        maximum: 50
        usage: 0.6
        actions:
          - name: morning
            minimum: 5
            maximum: 50
            schedule: cron(0 6 * * ? *)
          - name: night
            minimum: 1
            maximum: 1
            schedule: cron(0 0 * * ? *)
      read:
        ...

resources:
 Resources:
   Table:
     Type: AWS::DynamoDB::Table
     ...
```

1.  审查名为 `handler.js` 的文件。

1.  使用以下命令安装依赖项：`npm install`。

1.  使用以下命令运行测试：`npm test -- -s $MY_STAGE`。

1.  审查 `.serverless` 目录中生成的内容。

1.  使用以下命令部署堆栈：`npm run dp:lcl -- -s $MY_STAGE`。

1.  在 AWS 控制台中审查堆栈和资源。

1.  使用以下命令多次调用 `simulate` 函数以触发自动扩展：`sls invoke -f simulate -r us-east-1 -s $MY_STAGE`。

1.  使用以下命令查看以下“监听器”函数日志：

```js
$ sls logs -f listener -r us-east-1 -s $MY_STAGE --filter 'event count'
$ sls logs -f listener -r us-east-1 -s $MY_STAGE --filter 'Duration'
$ sls logs -f listener -r us-east-1 -s $MY_STAGE --filter '"1 retries"'$ sls logs -f listener -r us-east-1 -s $MY_STAGE --filter '"2 retries"'
$ sls logs -f listener -r us-east-1 -s $MY_STAGE --filter '"3 retries"'
$ sls logs -f listener -r us-east-1 -s $MY_STAGE --filter '"4 retries"'
$ sls logs -f listener -r us-east-1 -s $MY_STAGE --filter '"5 retries"'
$ sls logs -f listener -r us-east-1 -s $MY_STAGE --filter '"6 retries"'
```

1.  在 AWS 控制台的 DynamoDB 指标选项卡上查看“写入容量”和“受限写入请求数量”指标，以查看自动扩展增量是否满足需求，然后在夜间缩小规模，并在早晨再次扩大。

1.  最后，一旦完成，使用以下命令删除堆栈：`npm run rm:lcl -- -s $MY_STAGE`。

# 它是如何工作的...

在“实现背压和速率限制”配方中，我们看到流处理器最小化限制以最大化吞吐量的重要性。在本章中，我们讨论了优化吞吐量的技术，例如批处理、分组和异步非阻塞请求，这些都增加了必须分配的数据存储容量。然而，虽然我们需要确保有足够的容量，但我们还希望最小化浪费的容量，自动扩展帮助我们实现这一点。自动扩展可以解决随着时间的推移而增长到预期峰值的需求，如已知事件的可预测需求，以及不可预测的需求。

在本配方中，我们使用 `serverless-dynamodb-autoscaling-plugin` 在每个表的基础上定义 `autoscaling` 策略。对于 `read` 和 `write` 容量，我们指定 `minimum` 和 `maximum` 容量和期望的 `usage` 百分比。这个 `usage` 百分比定义了我们希望拥有的额外空间量，这样我们就可以提前增加容量，以确保在达到 100% 利用率并开始节流之前分配额外的容量。我们还可以在特定时间安排自动扩展 `actions`。在本配方中，我们在夜间缩小规模以最小化浪费，然后在典型需求到来之前在早上恢复规模。

# 利用缓存控制

自主、云原生服务维护自己的物化视图，并将复制的数据存储在高度可用且性能极优的云原生数据库中。当与 API 网关和 FaaS 的性能结合时，通常没有必要添加传统的缓存机制来实现面向用户的 **前端后端**（**BFF**）服务的预期性能。但话虽如此，这并不意味着我们不应该利用已经包装服务的 CDN，例如 CloudFront。因此，以下配方将向您展示如何利用缓存控制头和利用 CDN 来提高最终用户性能，同时减少对服务的负载。

# 如何操作...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch9/cache-control --path cncb-cache-control
```

1.  导航到 `cncb-cache-control` 目录，`cd cncb-cache-control`。

1.  检查名为 `serverless.yml` 的文件。

1.  检查名为 `handler.js` 的文件，其内容如下：

```js
module.exports.get = (request, context, callback) => {
  const response = {
    statusCode: 200,
    headers: {
      'Cache-Control': 'max-age=5',
    },
    body: ...,
  };

  callback(null, response);
};

module.exports.save = (request, context, callback) => {
 const response = {
   statusCode: 200,
   headers: {
     'Cache-Control': 'no-cache, no-store, must-revalidate',
   },
 };

 callback(null, response);
};
```

1.  使用 `npm install` 安装依赖项。

1.  使用 `npm test` 运行测试。

1.  检查 `.serverless` 目录中生成的内容。

1.  按照以下步骤部署堆栈：

```js
$ npm run dp:lcl -- -s $MY_STAGE
...
Stack Outputs
ApiDistributionEndpoint: https://d2thj6o092tkou.cloudfront.net
...
```

部署 CloudFront 分发通常需要超过 20 分钟。

1.  在 AWS 控制台中检查堆栈和资源。

1.  使用以下 `curl` 命令多次调用堆栈输出中显示的端点，以查看缓存结果的性能差异：

```js
$ curl -s -D - -w "Time: %{time_total}" -o /dev/null  https://<see stack output>.cloudfront.net/things/123 | egrep -i 'X-Cache|Time' 
X-Cache: Miss from cloudfront
Time: 0.712
$ curl ...
X-Cache: Hit from cloudfront
Time: 0.145

$ curl -v -X POST -H 'Content-Type: application/json' -d '{"name":"thing 1"}' https://<see stack output>.cloudfront.net/things ...
< HTTP/1.1 200 OK
< Cache-Control: no-cache, no-store, must-revalidate
< X-Cache: Miss from cloudfront
...
```

1.  最后，使用 `npm run rm:lcl -- -s $MY_STAGE` 删除堆栈。

# 它是如何工作的...

在这个相关的配方中，我们关注等式的服务端。云原生数据库，如 DynamoDB，响应时间在低 10 毫秒，AWS API 网关和 AWS Lambda 对于 BFF 服务的整体延迟通常在低 100 毫秒。只要数据库容量设置得当且节流最小化，就很难从最终用户的角度观察到明显的性能提升。真正改善这一点的唯一方法就是根本不需要发出请求。

这是一个云原生可能真正反直觉的案例。传统上，为了提高性能，我们需要增加基础设施的数量，并在代码和数据库之间添加一个昂贵的缓存层。换句话说，我们需要花费更多的钱来提高性能。然而，在这个菜谱中，我们利用了一个极低成本的边缘缓存，既提高了性能，又降低了成本。通过向我们的响应添加 `Cache-Control` 头部，例如 `max-age`，我们可以告诉浏览器不要重复请求，并告诉 CDN 为其他用户重用响应。因此，我们减少了 API 网关和函数的负载，并减少了数据库所需的容量，从而降低了所有这些服务的成本。明确控制哪些操作应该存储 `no-cache`，例如 `PUT`、`POST` 和 `DELETE` 方法，也是一种良好的实践。

# 利用会话一致性

云原生前端应用程序的设计必须考虑到系统最终应该是一致的。例如，在一个传统的前端应用程序中，保存数据然后立即执行查询以检索相同的数据并不罕见。然而，在一个最终一致性的系统中，查询第一次尝试很可能找不到数据。相反，云原生前端利用了单页应用程序至少可以缓存数据的事实，以用户会话的持续时间为限。这种方法被称为会话一致性。以下菜谱演示了如何使用流行的 Apollo Client ([`www.apollographql.com/client`](https://www.apollographql.com/client)) 与 ReactJS 结合使用，通过实现 **会话一致性** 来提高感知性能并减少系统负载。

# 如何做到这一点...

1.  从以下模板创建 `service` 和 `spa` 项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch9/session-consistency/spa --path cncb-session-consistency-spa

$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch9/session-consistency/service --path cncb-session-consistency-service
```

1.  使用以下命令部署 `service`：

```js
$ cd ./cncb-session-consistency-service
$ npm install
$ npm run dp:lcl -- -s $MY_STAGE
```

1.  导航到 `cncb-session-consistency-spa` 目录，`cd ../cncb-session-consistency-spa`。

1.  查看以下内容的 `src/index.js` 文件，并根据 `service` 堆栈输出的值更新 `uri`，如下所示：

```js
...
const client = new ApolloClient({
    link: new HttpLink({
        // CHANGE ME
        uri: 'https://*<API_ID>*.execute-api.us-east-1.amazonaws.com/*<STAGE>*/graphql',
    }),
    cache: new InMemoryCache(),
});
...
```

1.  查看名为 `src/App.js` 的文件，其内容如下：

```js
...
const AddThing = () => {
  ...
  return (
    <Mutation
      mutation={SAVE_THING}
      update={(cache, { data: { saveThing } }) => {
        const { things } = cache.readQuery({ query: GET_THINGS });
        cache.writeQuery({
          query: GET_THINGS,
          data: { things: { items: things.items.concat([saveThing]) } }
        });
      }}
    >
      ...
    </Mutation>
  );
};
...
```

1.  使用 `npm install` 安装依赖项。

1.  使用 `npm start` 在本地运行应用程序。

1.  浏览到 `http://localhost:3000`。

1.  添加和更新一些内容，注意查询结果是如何更新的。

1.  最后，一旦完成，按照以下方式删除服务堆栈：

```js
$ cd ../cncb-session-consistency-service
$ npm run rm:lcl -- -s $MY_STAGE
```

# 它是如何工作的...

在这个菜谱中，我们使用了一个类似于我们在 *实现一个 GraphQL CRUD BFF* 菜谱中创建的 GraphQL BFF。这里的重点是前端应用程序，我们使用 ReactJS 和 Apollo Client 创建它，并且特别关注如何缓存我们与服务的交互。首先，我们在 `src/index.js` 文件中创建 `ApolloClient` 并用服务的端点和最重要的 `InMemoryCache` 对象初始化它。

接下来，我们在`src/App.js`文件中实现用户界面。屏幕显示从`things`查询返回的项目列表。Apollo 客户端将自动缓存查询的结果。更新单个对象的突变将自动更新缓存并触发屏幕重新渲染。请注意，添加新数据需要更多努力。《AddThing》函数使用突变的`update`功能来保持缓存同步并触发重新渲染。`update`函数接收对缓存的引用和突变返回的对象。然后我们调用`readQuery`从缓存中读取查询，将新对象追加到查询结果中，并最终通过调用`writeQuery`来更新缓存。

最终结果是用户体验非常低延迟，因为我们正在优化执行请求的数量、传输的数据量以及使用的内存量。最重要的是，对于新数据和更新数据，我们不会丢弃客户端创建的任何内容，并用相同的检索值来替换它——毕竟，这只是不必要的劳动。我们已经有数据了，为什么还要丢弃它并再次检索呢？我们也无法确定服务端的数据是一致的——除非我们执行一个更慢、成本更高的连续读取，正如所述，这是不必要的。在区域故障的情况下，会话一致性对于多区域部署变得更加有价值。正如我们将在第十章“部署到多个区域”中讨论的，最终一致性的云原生系统对区域故障非常宽容，因为它们已经对最终一致性有容忍度，这在故障转移期间可能会更加持久。因此，会话一致性有助于使区域故障对最终用户透明。对于在区域故障期间必须保持可用的任何数据，我们可以将会话一致性进一步推进，并将用户会话持久化到本地存储。
