# 为失败而设计

在本章中，将涵盖以下食谱：

+   使用适当的超时和重试

+   实现背压和速率限制

+   处理故障

+   重新提交故障事件

+   使用反向 OpLock 实现幂等性

+   使用事件溯源实现幂等性

# 简介

处理失败是云原生的基础。我们构建自主服务，当它们失败时限制爆炸半径，并在其他服务失败时继续运行。我们将部署与发布解耦，并控制每次部署的批量大小，以便在部署出错时容易识别问题。我们将测试左移到持续部署管道中，以在部署之前捕获问题，以及一直移到生产环境中，在那里我们持续测试系统并在异常情况下发出警报，以最小化恢复时间。本章中的食谱演示了如何设计一个在失败面前具有弹性和宽容性的服务，以便正确处理瞬态故障，最小化其影响，并使服务能够自我修复。

# 使用适当的超时和重试

计算机网络的现实是它们不可靠。云计算的现实是它依赖于计算机网络，因此我们必须实施服务来正确处理网络异常。这个食谱演示了如何正确配置函数和 SDK 调用，使用适当的超时和重试。

# 如何做...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch8/timeout-retry --path cncb-timeout-retry
```

1.  使用 `cd cncb-timeout-retry` 切换到 `cncb-timeout-retry` 目录。

1.  查看名为 `serverless.yml` 的文件，其内容如下：

```js
service: cncb-timeout-retry
...
functions:
  command:
    handler: handler.command
    timeout: 6
    memorySize: 1024
...
```

1.  查看名为 `handler.js` 的文件，其内容如下：

```js
module.exports.command = (request, context, callback) => {
  const db = new aws.DynamoDB.DocumentClient({
    httpOptions: { timeout: 1000 },
    logger: console,
  });
...
  db.put(params, callback);
};
```

1.  使用 `npm install` 安装依赖项。

1.  使用 `npm test` 运行测试。

1.  查看在 `.serverless` 目录中生成的内容。

1.  使用 `npm run dp:lcl -- -s $MY_STAGE` 部署堆栈。

1.  查看 AWS 控制台中的堆栈和资源。

1.  使用以下命令调用函数：

```js
$ sls invoke -f command -r us-east-1 -s $MY_STAGE -d '{"name":"thing one"}'
```

1.  查看以下命令函数日志：

```js
$ sls logs -f command -r us-east-1 -s $MY_STAGE

2018-07-14 23:41:14.229 (-04:00) ... [AWS dynamodb 200 0.063s 0 retries] putItem({ TableName: 'john-cncb-timeout-retry-things',
  Item:
   { id: { S: 'c22f3ce3-551a-4999-b750-a20e33c3d53b' },
     name: { S: 'thing one' } } })   
```

1.  完成后，使用 `npm run rm:lcl -- -s $MY_STAGE` 删除堆栈。

# 它是如何工作的...

**网络中断**可能随时发生。一个请求可能无法通过，而下一个请求可能顺利通过；因此，我们应该设置较短的超时时间，以便我们能够尽快重试过程，但又不至于太短，以至于一个良好的请求在正常完成之前就超时了。我们还必须确保我们的超时和重试周期在函数超时之前有足够的时间完成。默认情况下，`aws-sdk` 配置为两分钟后超时，并执行三次带有递增延迟时间的重试。当然，两分钟太长了。将 `timeout` 设置为 1000（1 秒）通常足以让请求完成，并允许在 6 秒的函数超时之前完成三次重试。

如果请求频繁超时，这可能表明函数分配的资源过少。例如，函数分配的 `memorySize` 与使用的机器实例大小之间存在关联。较小的机器实例也有更少的网络 I/O 容量，这可能导致更频繁的网络中断。因此，增加 `memorySize` 将降低网络波动性。

# 实现背压和速率限制

云原生系统中的各种服务必须能够处理系统中的流量起伏。上游服务不应过载下游服务，下游服务必须能够处理峰值负载而不会落后或过载更下游的服务。本食谱展示了如何利用流处理器的自然背压并实现额外的速率限制来管理节流。

# 准备工作

在开始此食谱之前，您需要一个 AWS Kinesis Stream，例如在 *创建事件流* 食谱中创建的那个。

# 如何操作...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch8/backpressure-ratelimit --path cncb-backpressure-ratelimit
```

1.  使用以下命令导航到 `cncb-backpressure-ratelimit` 目录：`cd cncb-backpressure-ratelimit`。

1.  查看名为 `serverless.yml` 的文件，其内容如下：

```js
service: cncb-backpressure-ratelimit
...
functions:
  listener:
    handler: handler.listener
    timeout: 240 # headroom for retries
    events:
      - stream:
          batchSize: 1000 # / (timeout / 2) < write capacity
          ...
    environment:
      WRITE_CAPACITY_UNITS: 10
      SHARD_COUNT: 1
...
resources:
  Resources:
    Table:
      Type: AWS::DynamoDB::Table
      Properties:
        ...
        ProvisionedThroughput:
          ...
          WriteCapacityUnits: ${self:functions.listener.environment.WRITE_CAPACITY_UNITS}
```

1.  查看名为 `handler.js` 的文件，其内容如下：

```js
module.exports.listener = (event, context, cb) => {
  _(event.Records)
    .map(recordToUow)
    .filter(forPurple)
    .ratelimit(Number(process.env.WRITE_CAPACITY) / 
      Number(process.env.SHARD_COUNT) / 10, 100)
    .flatMap(put)
    .collect()
    .toCallback(cb);
};

const put = uow => {
  const params = { ... };
  const db = new aws.DynamoDB.DocumentClient({
    httpOptions: { timeout: 1000 },
    // default values:
    // maxRetries: 10,
    // retryDelayOptions: {
    //   base: 50,
    // },
    logger: console,
  });

  return _(db.put(params).promise()
    .then(() => uow)
  );
};
```

1.  使用以下命令安装依赖项：`npm install`。

1.  使用以下命令运行测试：`npm test -- -s $MY_STAGE`。

1.  查看在 `.serverless` 目录中生成的内容。

1.  使用以下命令部署堆栈：`npm run dp:lcl -- -s $MY_STAGE`。

1.  在 AWS 控制台中查看堆栈和资源。

1.  使用以下命令调用 `simulate` 函数：

```js
$ sls invoke -f simulate -r us-east-1 -s $MY_STAGE
[
    {
        "total": 4775,
        "blue": 1221,
        "green": 1190,
        "purple": 1202,
```

```js
        "orange": 1162
    }
]
```

1.  查看以下 `trigger` 函数日志：

```js
$ sls logs -f listener -r us-east-1 -s $MY_STAGE --filter 'event count' 2018-07-15 22:53:29 ... event count: 1000
2018-07-15 22:54:00 ... event count: 1000
2018-07-15 22:54:33 ... event count: 1000
2018-07-15 22:55:05 ... event count: 425
2018-07-15 22:55:19 ... event count: 1000
2018-07-15 22:55:51 ... event count: 350$ sls logs -f listener -r us-east-1 -s $MY_STAGE --filter 'Duration' REPORT ... Duration: 31011.59 ms ...
REPORT ... Duration: 33038.58 ms ...
REPORT ... Duration: 32399.91 ms ...
REPORT ... Duration: 13999.56 ms ...
REPORT ... Duration: 31670.86 ms ...
REPORT ... Duration: 12856.77 ms ...

$ sls logs -f listener -r us-east-1 -s $MY_STAGE --filter 'retries' ...
2018-07-15 22:55:49 ... [AWS dynamodb 200 0.026s 0 retries] putItem({ TableName: '...',
  Item:
   { id: { S: '686dc03a-88a3-11e8-829c-67d049599dd2' },
     type: { S: 'purple' },
     timestamp: { N: '1531709604787' },
     partitionKey: { S: '3' },
     tags: { M: { region: { S: 'us-east-1' } } } },
  ReturnConsumedCapacity: 'TOTAL' })
2018-07-15 22:55:50 ... [AWS dynamodb 200 0.013s 0 retries] putItem({ TableName: '...',
  Item:
   { id: { S: '686d4b14-88a3-11e8-829c-67d049599dd2' },
     type: { S: 'purple' },
     timestamp: { N: '1531709604784' },
     partitionKey: { S: '4' },
     tags: { M: { region: { S: 'us-east-1' } } } },
  ReturnConsumedCapacity: 'TOTAL' })
...
```

1.  完成后，使用以下命令删除堆栈：`npm run rm:lcl -- -s $MY_STAGE`。

# 它是如何工作的...

*背压* 是实现良好的流处理器的关键特性。当使用 *命令式编程范式*，例如在批处理记录数组中循环时，下游目标系统可能会轻易过载，因为循环会尽可能快地处理记录，而不考虑目标系统的吞吐量能力。另一方面，**函数式响应式编程**（**FRP**）范式，例如使用 Highland.js ([`highlandjs.org`](https://highlandjs.org)) 或 RxJS ([`github.com/ReactiveX/rxjs`](https://github.com/ReactiveX/rxjs)) 库，提供了自然的背压，因为数据只以下游步骤能够完成任务的速度被拉取。例如，吞吐量低的系统只会以其容量允许的速度快速拉取数据。

另一方面，像 DynamoDB 或 Kinesis 这样高度可扩展的系统，能够以极高的吞吐量处理数据，它们依赖于节流来限制容量。在这种情况下，FRP 的自然背压不足以应对；需要额外使用 `ratelimit` 功能。例如，当我编写这个菜谱时，我运行了一个没有速率限制的模拟，然后出去吃饭。当我几个小时后回来时，模拟生成的事件仍在尝试处理。这是因为 DynamoDB 节流了请求，而 `aws-sdk` 内置的指数退避和重试消耗了太多时间，导致函数 `timeout` 并重新尝试整个批量。这表明，虽然重试，如 *Employing proper timeouts and retries* 菜谱中讨论的那样，对于同步请求很重要，但不能仅依赖于异步流处理。相反，我们需要积极限制流量速率，以避免节流和指数退避，从而确保批量在函数超时内完成。

在这个菜谱中，我们使用一个简单的算法来计算流量速率——每 `100` 毫秒 `WRITE_CAPACITY / SHARD_COUNT / 10`。这确保了 DynamoDB 每秒接收的请求数量不会超过已分配的数量。我还使用了一个简单的算法来确定批量大小——`batchSize / (timeout / 2) < WRITE_CAPACITY`。这确保了在正常情况下应该有足够的时间来完成批量，但在有节流的情况下，将会有两倍于必要的时间。请注意，这只是一个逻辑起点；这是云原生系统中性能调优应该关注的领域。你的数据和目标系统的特性将决定最有效的设置。正如我们将在 *Autoscaling DynamoDB* 菜谱中看到的那样，自动扩展为背压、速率限制和性能调优增加了另一个维度。无论如何，你可以从这些简单且安全的算法开始，并在一段时间内调整它们。

# 处理故障

流处理器对瞬时错误具有天然的抗性和宽容性，因为它们使用背压并自动重试失败的批量。然而，如果处理不当，硬错误可能会导致交通堵塞，导致消息丢失。这个菜谱将向你展示如何将这些错误委派为故障事件，以便良好的消息可以继续流动。

# 如何操作...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch8/handling-faults --path cncb-handling-faults
```

1.  使用 `cd cncb-handling-faults` 命令进入 `cncb-handling-faults` 目录。

1.  检查名为 `serverless.yml` 的文件。

1.  检查包含以下内容的 `handler.js` 文件：

```js
module.exports.listener = (event, context, cb) => {
  _(event.Records)
    .map(recordToUow)
    .filter(forThingCreated)
    .tap(validate)
    .tap(randomError)
    .flatMap(save)
    .errors(errors)
    .collect().toCallback(cb);
};

const validate = uow => {
  if (uow.event.thing.name === undefined) {
    const err = new Error('Validation Error: name is required');
    // handled
    err.uow = uow;
    throw err;
  }
};

const randomError = () => {
  if (Math.floor((Math.random() * 5) + 1) === 3) {
    // unhandled
    throw new Error('Random Error');
  }
};

const save = uow => {
 ...
 return _(db.put(uow.params).promise()
   .catch(err => {
     // handled
     err.uow = uow;
     throw err;
   }));
};

const errors = (err, push) => {
  if (err.uow) {
    // handled exceptions are adorned with the uow in error
    push(null, publish({
      type: 'fault',
      timestamp: Date.now(),
      tags: {
        functionName: process.env.AWS_LAMBDA_FUNCTION_NAME,
      },
      err: {
        name: err.name,
        message: err.message,
        stack: err.stack,
      },
      uow: err.uow,
    }));
  } else {
    // rethrow unhandled errors to stop processing
    push(err);
  }
};
```

1.  使用 `npm install` 安装依赖项。

1.  使用 `npm test -- -s $MY_STAGE` 运行测试。

1.  检查 `.serverless` 目录中生成的内容。

1.  使用 `npm run dp:lcl -- -s $MY_STAGE` 部署堆栈。

1.  在 AWS 控制台中检查堆栈和资源。

1.  使用以下命令调用`simulate`函数：

```js
$ sls invoke -f simulate -r us-east-1 -s $MY_STAGE
```

1.  查看以下`trigger`函数日志：

```js
$ sls logs -f listener -r us-east-1 -s $MY_STAGE --filter 'publishing fault' ... {"type":"fault","timestamp":...,"tags":{"functionName":"cncb-handling-faults-john-listener"},"err":{"name":"Error","message":"Validation Error: name is required" ...

... {"type":"fault","timestamp":...,"tags":{"functionName":"cncb-handling-faults-john-listener"},"err":{"name":"ValidationException","message":"...Missing the key id..." ...
```

1.  完成后，删除堆栈：`npm run rm:lcl -- -s $MY_STAGE`。

# 它是如何工作的...

这个食谱实现了我在我的书*云原生开发模式和最佳实践*（[`www.packtpub.com/application-development/cloud-native-development-patterns-and-best-practices`](https://www.packtpub.com/application-development/cloud-native-development-patterns-and-best-practices)）中深入讨论的*流断路器*模式。遇到硬错误的流处理器将继续重新处理这些事件，直到它们从流中过期——除非流处理器被实现为将这些错误放在一边，将它们作为故障事件委托给其他地方处理，如*重新提交故障事件*食谱中所述。这缓解了交通堵塞，以便其他事件可以继续流动。

这个食谱模拟了在流处理器中不同阶段可能失败的事件。一些事件模拟了上游的 bug，这些 bug 将导致验证逻辑失败，该逻辑断言事件正在上游正确创建。其他事件在它们被插入到 DynamoDB 时将失败。逻辑还会随机失败一些事件来模拟不会产生故障的短暂错误，这些错误将自动重试。在日志中，您将看到发布了两条故障事件。当生成随机错误时，您将在日志中看到函数正在重试批次。如果模拟没有生成随机错误，那么您应该重新运行它，直到它生成错误。

为了隔离流中的事件，我们需要引入**工作单元**（**UOW**）的概念，它将一批中的一个或多个事件组合成一个原子单元，这个单元必须一起成功或失败。UOW 包含原始的 Kinesis 记录（`uow.record`）、从记录中解析的事件（`uow.event`）以及任何附加到 UOW 的中间处理结果（`uow.params`），这些结果在它通过流处理器时被附加。UOW 还用于标识错误是否被处理或未处理。当验证逻辑识别出预期的错误或从外部调用中捕获到错误时，UOW 将被装饰到错误上，并且错误将被重新抛出。装饰的工作单元（`error.uow`）作为指示器，告知`errors`处理器错误已在流处理器中被处理，并且应该`发布`错误作为故障事件。意外的错误，如随机生成的错误，不会被流处理器逻辑处理，因此不会有装饰的 UOW。错误处理器将`推送`这些错误到下游，导致函数失败并重试。在*创建警报*食谱中，我们讨论了对故障事件以及迭代器年龄的监控，以便团队可以及时收到关于流处理器错误的通知。

# 重新提交故障事件

我们已经设计我们的流处理器将错误作为故障事件进行委派，以便有效的事件可以继续流动。我们监控并警告故障事件，以便可以采取适当的行动来解决问题。一旦问题得到解决，可能需要重新处理失败的事件。本食谱演示了如何将故障事件重新提交给引发故障的流处理器。

# 如何操作...

1.  从以下模板创建 `monitor`、`simulator` 和 `cli` 项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch8/resubmitting-faults/monitor --path cncb-resubmitting-faults-monitor

$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch8/resubmitting-faults/cli --path cncb-resubmitting-faults-cli

$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch8/resubmitting-faults/simulator --path cncb-resubmitting-faults-simulator
```

1.  检查 `cncb-resubmitting-faults-monitor` 和 `cncb-resubmitting-faults-simulator` 目录中的 `serverless.yml` 文件。

1.  按照以下方式部署 `monitor` 和 `simulator` 堆栈：

```js
$ cd ../cncb-resubmitting-faults-monitor
$ npm install
$ npm test -- -s $MY_STAGE
$ npm run dp:lcl -- -s $MY_STAGE

  Stack Outputs
 BucketName: cncb-resubmitting-faults-monitor-john-bucket-1llq835xdczd8

$ cd ../cncb-resubmitting-faults-simulator
$ npm install
$ npm test -- -s $MY_STAGE
$ npm run dp:lcl -- -s $MY_STAGE
```

1.  在 AWS 控制台中检查堆栈和资源。

1.  按照以下方式从 `cncb-resubmitting-faults-simulator` 目录运行模拟器：

```js
$ sls invoke -f simulate -r us-east-1 -s $MY_STAGE
```

1.  确认故障文件已写入监控堆栈输出中指定的存储桶 `cncb-resubmitting-faults-monitor-*-bucket-*` 并注意路径。如果没有生成故障，请再次运行模拟器，直到生成故障。

1.  检查名为 `./cli/lib/resubmit.js` 的文件，其内容如下：

```js
exports.command = 'resubmit [bucket] [prefix]'
exports.desc = 'Resubmit the faults in [bucket] for [prefix]'
...
const invoke = (lambda, options, event) => {
  const Payload = JSON.stringify({
      Records: [event.uow.record],
  });

  const params = {
    FunctionName: event.tags.functionName,
    ...
    Payload: Buffer.from(Payload),
  };

  return _(lambda.invoke(params).promise());
}
```

1.  使用以下命令重新提交故障：

```js
$ cd ../cncb-resubmitting-faults-cli
$ npm install
$ node index.js resubmit -b cncb-resubmitting-faults-monitor-$MY_STAGE-bucket-<suffix> -p <s3-path> --dry false
```

1.  手动清空 `cncb-resubmitting-faults-monitor-*` 存储桶，并在完成 `npm run rm:lcl -- -s $MY_STAGE` 后移除 `monitor` 和 `simulator` 堆栈。

# 它是如何工作的...

在 *处理故障* 食谱中，我们看到了流处理器如何通过发布包含失败工作单元所有相关数据的故障事件来委派硬错误。在本食谱中，故障监控器消费这些故障事件并将它们存储在 S3 存储桶中。这使得团队能够审查特定的故障，以帮助确定问题的根本原因。故障包含捕获的具体异常、失败的事件以及附加到工作单元的所有上下文信息。

一旦解决了根本原因，原始事件可以提交回发布故障的流处理器。这是可能的，因为故障事件包含原始 Kinesis 记录 (`event.uow.record`) 和要调用的函数名称 (`event.tags.functionName`)。命令行实用程序从指定路径的存储桶中读取所有故障事件并调用特定函数。从函数逻辑的角度来看，这种直接调用函数与直接从 Kinesis 流调用没有区别。然而，流处理器必须设计为幂等的，并且能够处理乱序事件，正如我们将在 *使用逆 OpLock 实现幂等性* 和 *使用事件溯源实现幂等性* 食谱中讨论的那样。

# 使用逆 OpLock 实现幂等性

从业务规则的角度来看，事件必须恰好处理一次非常重要；否则，可能会出现问题，例如重复计数或根本不计数。然而，我们的云原生系统必须能够抵御故障并主动重试以确保不丢失任何消息。不幸的是，这意味着消息可能会被多次投递，例如当生产者重新发布事件或流处理器重试可能已部分处理的批次时。解决这个问题的方法是实现所有操作都具有幂等性。这个配方通过我所说的逆 OpLock 来实现幂等性。

# 如何做到这一点...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch8/idempotence-inverse-oplock --path cncb-idempotence-inverse-oplock
```

1.  使用`cd cncb-idempotence-inverse-oplock`命令导航到`cncb-idempotence-inverse-oplock`目录。

1.  检查名为`serverless.yml`的文件。

1.  检查名为`handler.js`的文件，其内容如下：

```js
module.exports.listener = (event, context, cb) => {
  _(event.Records)
    .map(recordToUow)
    .filter(forThingSaved)
    .flatMap(save)
    .collect().toCallback(cb);
};

const save = uow => {
  const params = {
    TableName: process.env.TABLE_NAME,
    Item: {
      ...uow.event.thing,
      oplock: uow.event.timestamp,
    },
    ConditionExpression: 'attribute_not_exists(#oplock) OR #oplock < :timestamp',
    ...
  };

  const db = new aws.DynamoDB.DocumentClient({
    httpOptions: { timeout: 1000 },
    logger: console,
  });

  return _(db.put(params).promise()
    .catch(handleConditionalCheckFailedException)
    .then(() => uow)
  );
}

const handleConditionalCheckFailedException = (err) => {
  if (err.code !== 'ConditionalCheckFailedException') {
    err.uow = uow;
    throw err;
  }
};
```

1.  使用`npm install`命令安装依赖项。

1.  使用`npm test -- -s $MY_STAGE`命令运行测试。

1.  检查`.serverless`目录中生成的内容。

1.  使用`npm run dp:lcl -- -s $MY_STAGE`命令部署栈。

1.  在 AWS 控制台中检查栈和资源。

1.  使用以下命令调用模拟函数：

```js
$ sls invoke -f simulate -r us-east-1 -s $MY_STAGE
```

1.  查看以下`listener`函数的日志：

```js
$ sls logs -f listener -r us-east-1 -s $MY_STAGE

... [AWS dynamodb 200 0.098s 0 retries] putItem({ TableName: '...',
  Item:
   { id: { S: '3022eaeb-45b7-46d5-b0a1-696c0eb9aa25' },
     oplock: { N: '1531628180237' } },
...
... [AWS dynamodb 400 0.026s 0 retries] putItem({ TableName: '...',
  Item:
   { id: { S: '3022eaeb-45b7-46d5-b0a1-696c0eb9aa25' },
     oplock: { N: '1531628180237' } },
...
... { ConditionalCheckFailedException: The conditionalrequest failed
    at Request.extractError ...
...
 message: 'The conditional request failed',
  code: 'ConditionalCheckFailedException',
  time: 2018-07-15T04:16:21.202Z,
  requestId: '36BB14IGTPGM8DE8CFQJS0ME3VVV4KQNSO5AEMVJF66Q9ASUAAJG',
  statusCode: 400,
 retryable: false,
  retryDelay: 20.791698133966396 }

... [AWS dynamodb 200 0.025s 0 retries] putItem({ TableName: '...',
  Item:
   { id: { S: '3022eaeb-45b7-46d5-b0a1-696c0eb9aa25' },
     oplock: { N: '1531628181237' } },
...
... [AWS dynamodb 400 0.038s 0 retries] putItem({ TableName: '...',
  Item:
   { id: { S: '3022eaeb-45b7-46d5-b0a1-696c0eb9aa25' },
     oplock: { N: '1531628180237' } },
...
... { ConditionalCheckFailedException: The conditionalrequest failed
    at Request.extractError ...
...    
  message: 'The conditional request failed',
...
```

1.  完成后，使用`npm run rm:lcl -- -s $MY_STAGE`命令删除栈。

# 它是如何工作的...

传统的乐观锁定防止多个用户同时更新相同的记录。只有当用户检索数据后`oplock`字段没有变化时，记录才会被更新。如果数据已更改，则会抛出异常，并强制用户在继续更新之前重新检索数据。这迫使更新按顺序执行，并需要人工交互来解决任何潜在冲突。

**逆 OpLock**旨在为异步处理提供幂等性。我们不是强制事务重试，而是简单地做相反的事情——我们丢弃较旧或重复的事件。在上游 Backend For Frontend 服务中，可以使用传统的 OpLock 来序列化用户事务，其中下游服务实现逆 OpLock 以确保较旧或重复的事件不会覆盖最新的数据。在这个配方中，我们使用`uow.event.timestamp`作为`oplock`值。在某些场景中，如果多个事件在确切的同一毫秒发生，可能更倾向于使用序列号。`ConditionalCheckFailedException`被捕获并忽略。所有其他异常都会重新抛出，并附加工作单元以发布故障事件，如*处理故障*配方中所述。

这个配方中的模拟会发布一个`thing-created`事件，再次发布它，然后发布一个`thing-updated`事件，最后第三次发布`thing-created`事件。日志显示`thing-created`事件只被处理一次，重复的则被忽略。

# 使用事件溯源实现幂等性

从业务规则的角度来看，事件必须被精确处理一次非常重要；否则，可能会出现问题，例如重复计数或根本不计数。然而，我们的云原生系统必须能够抵御故障并主动重试以确保不丢失任何消息。不幸的是，这意味着消息可能会被多次投递，例如当生产者重新发布事件或流处理器重试可能已部分处理的事务批次时。解决这个问题的方法是实现所有操作的可幂等性。本菜谱演示了如何使用事件溯源和微事件存储来实现幂等性。

# 如何做到这一点...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch8/idempotence-es --path cncb-idempotence-es
```

1.  使用 `cd cncb-idempotence-es` 命令导航到 `cncb-idempotence-es` 目录。

1.  查看名为 `serverless.yml` 的文件。

1.  查看以下内容的 `handler.js` 文件：

```js
module.exports.listener = (event, context, cb) => {
  _(event.Records)
    .map(recordToUow)
    .filter(forThingSaved)
    .flatMap(saveEvent)
    .collect().toCallback(cb);
};

const saveEvent = uow => {
  const params = {
    TableName: process.env.EVENTS_TABLE_NAME,
    Item: {
 id: uow.event.thing.id,
 sequence: uow.event.id,
      event: uow.event,
    }
  };

  const db = new aws.DynamoDB.DocumentClient({
    httpOptions: { timeout: 1000 },
    logger: console,
  });

  return _(db.put(params).promise()
    .then(() => uow)
  );
}
```

1.  使用 `npm install` 安装依赖项。

1.  使用 `npm test -- -s $MY_STAGE` 运行测试。

1.  查看在 `.serverless` 目录中生成的内容。

1.  使用 `npm run dp:lcl -- -s $MY_STAGE` 部署堆栈。

1.  在 AWS 控制台中查看堆栈和资源。

1.  使用以下命令调用模拟函数：

```js
$ sls invoke -f simulate -r us-east-1 -s $MY_STAGE
```

1.  查看以下 `trigger` 函数日志：

```js
$ sls logs -f trigger -r us-east-1 -s $MY_STAGE

... record: { ... "Keys":{"sequence":{"S":"3fdb8c10-87ea-11e8-9cf5-0b6c5b83bdcb"},"id":{"S":"8c083ef9-d180-48b8-a773-db0f61815f38"}}, ...

... record: { ... "Keys":{"sequence":{"S":"3fdb8c11-87ea-11e8-9cf5-0b6c5b83bdcb"},"id":{"S":"8c083ef9-d180-48b8-a773-db0f61815f38"}}, ...

```

1.  完成后，使用 `npm run rm:lcl -- -s $MY_STAGE` 删除堆栈。

# 它是如何工作的...

事件溯源通过事件不可变来促进幂等性。具有相同唯一 ID 的相同事件可以被发布或处理多次，并产生相同的结果。微事件存储充当缓冲区，以过滤掉重复项。服务消费所需事件，并将它们存储在具有 **hashkey** 的微事件存储中，该 **hashkey** 用于分组相关事件，例如领域对象的 `uow.event.thing.id`，以及基于 `uow.event.id` 的范围键。这个主键也是不可变的。因此，相同的事件可以保存多次，但在数据库流上只产生一个事件。因此，在 *创建微事件存储* 或 *实现分析 BFF* 菜谱中讨论的业务逻辑只会触发一次。

本菜谱中的模拟发布了一个 `thing-created` 事件，再次发布它，然后发布一个 `thing-updated` 事件，最后第三次发布 `thing-created` 事件。日志显示，三个 `thing-created` 事件实例只在 DynamoDB 流上产生一个事件。
