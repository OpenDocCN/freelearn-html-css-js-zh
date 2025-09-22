# 第七章：优化可观察性

在本章中，将介绍以下配方：

+   监控云原生系统

+   实现自定义指标

+   监控域事件

+   创建警报

+   创建持续的合成事务测试

# 简介

信心对于最大限度地发挥我们精简和自主的云原生服务的潜力至关重要，因为信心危机会阻碍进步。利用完全管理的云服务和遵循云原生设计模式来创建自主服务显著增加了团队信心。将部署与发布解耦并将测试左移，以创建流线化的持续交付管道，进一步增加了团队信心。然而，这还不够。我们需要将测试右移，一直进入生产环境，这样我们就可以监控和提醒团队关于系统状态的信息。这使团队有信心，他们将在错误发生时及时获得信息，从而最大限度地减少平均恢复时间。本章中的配方演示了如何优化云原生服务的可观察性，对重要事项发出警报，并在生产环境中持续测试以增加团队信心。

# 监控云原生系统

利用完全管理的云服务是创建精简、云原生服务的关键，因为采用这种可丢弃的架构使自给自足的全栈团队能够基于云服务提供的基础快速自信地交付。团队信心进一步增加，因为这种基础附带良好的可观察性。本配方演示了如何使用云提供商无关的第三方监控服务利用云提供商指标。

# 准备工作

在开始此配方之前，您需要一个 Datadog 账户 ([`www.datadoghq.com`](https://www.datadoghq.com))。

# 如何做到...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch7/datadog-account --path cncb-datadog-account
```

1.  使用 `cd cncb-datadog-account` 切换到 `cncb-datadog-account` 目录。

1.  查看以下内容的 `serverless.yml` 文件：

```js
service: cncb-datadog-account
...
resources:
  Resources:
    DatadogAWSIntegrationPolicy: 
      Type: AWS::IAM::ManagedPolicy
      Properties:
        PolicyDocument: ${file(includes.yml):PolicyDocument}
    DatadogAWSIntegrationRole:
      Type: AWS::IAM::Role
      Properties: 
        RoleName: DatadogAWSIntegrationRole
        AssumeRolePolicyDocument:
          Statement:
            Effect: Allow
            Principal:
              AWS: arn:aws:iam::464622532012:root
            Action: sts:AssumeRole
            Condition:
              StringEquals:
                sts:ExternalId: <copy value from datadog aws integration dialog>
        ManagedPolicyArns:
          - Ref: DatadogAWSIntegrationPolicy 
```

1.  使用 `npm install` 安装依赖项。

1.  使用 `npm test` 运行测试。

1.  查看在 `.serverless` 目录中生成的内容。

1.  使用 `npm run dp:lcl -- -s $MY_STAGE` 部署堆栈。

1.  登录到 **Datadog** 并转到 **集成** 页面，然后选择 AWS 集成磁贴。

1.  选择角色委派，输入您的 AWS 账户 ID 并将 AWS 角色名称设置为 `DatadogAWSIntegrationRole`。

1.  复制 AWS 外部 ID 值并使用它来更新 `serverless.yml` 中的 `sts:ExternalId`。

1.  将标签设置为 `account:cncb` 并按安装集成。

1.  使用 `npm run dp:lcl -- -s $MY_STAGE` 更新堆栈。

1.  在 AWS 控制台中查看堆栈和资源。

1.  使用以下命令多次调用示例函数：

```js
$ sls invoke -f hello -r us-east-1 -s $MY_STAGE
```

1.  查看预设的 Datadog Lambda 仪表板，在 Dashboards | Dashboard List | All Integrations | AWS Lambda。

1.  完成本章内容后，使用 `npm run rm:lcl -- -s $MY_STAGE` 删除堆栈。

# 它是如何工作的...

云服务提供商为他们的云服务收集有价值的指标。然而，他们不一定保留这些数据很长时间，并且对数据的切片和切块能力可能有限。因此，建议使用第三方监控服务来填补空白并提供更全面的监控能力。此外，在多语言云的使用情况下，云提供商无关的监控服务提供统一的监控体验。我选择的监控服务是 Datadog。这个配方展示了我们如何轻松快速地将 Datadog 连接到 AWS 账户并开始汇总指标，以增加我们云原生系统的可观察性。

要允许 Datadog 从 AWS 账户开始收集指标，我们必须授予它这样做的能力。正如 *如何操作* 部分所示，这需要在 AWS 端和 Datadog 端执行步骤。首先，我们部署一个堆栈来创建具有所有必要权限的 `DatadogAWSIntegrationPolicy`，以及 `DatadogAWSIntegrationRole` 将 AWS 账户与 Datadog 的 AWS 账户连接起来。这一点很重要。Datadog 也在 AWS 中运行。这意味着我们可以使用 *角色委托* 来连接账户，而不是共享访问密钥。一旦创建了 `DatadogAWSIntegrationRole`，我们就可以在 Datadog 端配置 AWS 集成，这需要角色的存在作为先决条件。Datadog 生成 `ExternalId`，我们需要将其添加到 `DatadogAWSIntegrationRole` 中，作为假设角色的条件。一旦集成到位，Datadog 就会从您的 AWS 账户中的 CloudWatch 消费请求的指标，以便它们可以汇总成有意义的仪表板，保留用于历史分析，并监控以发出关于感兴趣条件的警报。

# 实现自定义指标

价值增加型云服务提供的指标，如 *函数即服务*，是一个很好的起点。团队只需这些指标就可以有信心地将云原生服务投入生产。然而，更多的可观察性几乎总是更好的。我们需要了解我们函数内部运作的细粒度详细信息。这个配方展示了如何收集额外的指标，例如冷启动、内存、CPU 利用率和 HTTP 资源的延迟。

# 如何操作...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch7/custom-metrics --path cncb-custom-metrics
```

1.  使用 `cd cncb-custom-metrics` 命令导航到 `cncb-custom-metrics` 目录。

1.  查看名为 `serverless.yml` 的文件，其内容如下：

```js
service: cncb-custom-metrics
...
functions:
  hello:
    ...
    environment:
      ACCOUNT_NAME: ${opt:account}
      SERVERLESS_STAGE: ${opt:stage}
      SERVERLESS_PROJECT: ${self:service}
      MONITOR_ADVANCED: false
      DEBUG: '*'
```

1.  查看名为 `handler.js` 的文件，其内容如下：

```js
const { monitor, count } = require('serverless-datadog-metrics');
const debug = require('debug')('handler');

module.exports.hello = monitor((request, context, callback) => {
  debug('request: %j', request);
  count('hello.count', 1);
  const response = { ... };
  callback(null, response);
});
```

1.  使用 `npm install` 安装依赖项。

1.  使用 `npm test` 运行测试。

1.  查看在 `.serverless` 目录中生成的内容。

1.  部署堆栈：

```js
$ npm run dp:lcl -- -s $MY_STAGE

> cncb-custom-metrics@1.0.0 dp:lcl <path-to-your-workspace>/cncb-custom-metrics
> sls deploy -r us-east-1 --account cncb "-s" "john"
...
endpoints:
  GET - https://h865txqjqj.execute-api.us-east-1.amazonaws.com/john/hello

```

1.  在 AWS 控制台中查看堆栈和资源。

1.  在以下命令中调用堆栈输出中显示的端点：

```js
$ curl https://<APP-ID>.execute-api.us-east-1.amazonaws.com/$MY_STAGE/hello | json_pp
{
   "message" : "JavaScript Cloud Native Development Cookbook! ..."
}
```

1.  查看日志：

```js
$ sls logs -f hello -r us-east-1 -s $MY_STAGE

MONITORING|1530339912|1|count|aws.lambda.coldstart.count|#account:cncb,...
MONITORING|1530339912|0.259|gauge|node.process.uptime|#account:cncb,...
MONITORING|1530339912|1|count|hello.count|#account:cncb,...
MONITORING|1530339912|0|check|aws.lambda.check|#account:cncb,...
MONITORING|1530339912|0.498...|gauge|node.mem.heap.utilization|#account:cncb,...
MONITORING|1530339912|1.740238|histogram|aws.lambda.handler|#account:cncb,...  
```

1.  执行服务几次后，然后在 Datadog 中的仪表板下查看 Lambda 控板：仪表板 | 控板列表 | 所有集成 | AWS Lambda。

1.  在 Datadog 中探索自定义指标，使用图形探索：`hello.count` 和 `aws.lambda.handler.avg`。

1.  完成后，使用 `npm run rm:lcl -- -s $MY_STAGE` 删除堆栈。

# 它是如何工作的...

将自定义指标添加到函数中的操作与传统监控不同。传统方法涉及在每个机器上添加代理来收集指标，并定期将数据发送到监控系统。但是，在 *Function-as-a-service* 中，我们没有机器可以部署代理。一种替代方法是简单地在每个函数调用的末尾发送收集到的指标。然而，这会给每个函数调用增加显著的延迟。Datadog 提供了一种基于结构化日志语句的独特替代方案。计数、仪表、直方图和检查只是按收集时的顺序记录，Datadog 自动从 CloudWatch 日志中消费这些语句。

`serverless-datadog-metrics` 库 ([`www.npmjs.com/package/serverless-datadog-metrics`](https://www.npmjs.com/package/serverless-datadog-metrics)) 简化了此方法的使用。我们只需用 `monitor` 函数包装处理函数，它将收集有用的指标，例如冷启动、错误、执行时间、内存和 CPU 利用率，以及 HTTP 资源的延迟。HTTP 指标非常有价值。所有对资源（如 DynamoDB、S3 和 Kinesis）的 HTTP 调用都会自动记录，这样我们就可以看到函数在其外部资源上花费了多少时间。

此库还导出低级函数，例如 `count`、`gauge` 和 `histogram`，以支持额外的自定义指标。环境变量，如 `ACCOUNT_NAME` 和 `SERVERLESS_PROJECT`，用作仪表板和警报中过滤指标的分标签。

# 监控领域事件

在传统系统中，我们通常专注于观察同步请求的行为。然而，我们的云原生系统是高度异步和事件驱动的。因此，我们需要对系统中的领域事件流给予同等或更大的关注，以便我们可以确定这些流程何时偏离正常。此配方演示了如何收集领域事件指标。

# 准备工作

在开始此配方之前，您需要一个 AWS Kinesis Stream，例如在 *创建事件流* 配方中创建的流。

# 如何操作...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch7/event-metrics --path cncb-event-metrics
```

1.  使用 `cd cncb-event-metrics` 命令进入 `cncb-event-metrics` 目录。

1.  查看名为 `serverless.yml` 的文件。

1.  查看包含以下内容的 `handler.js` 文件：

```js
module.exports.listener = (event, context, cb) => {
  _(event.Records)
    .map(recordToUow)
    .tap(count)
    ...
    .collect().toCallback(cb);
};

const count = (uow) => {
  const tags = [
    `account:${process.env.ACCOUNT_NAME}`,
    `region:${uow.record.awsRegion}`,
    `stream:${uow.record.eventSourceARN.split('/')[1]}`,
    `shard:${uow.record.eventID.split('-')[1].split(':')[0]}`,
    `source:${uow.event.tags && uow.event.tags.source || 'not-specified'}`,
    `type:${uow.event.type}`,
  ];

  console.log(`MONITORING|${uow.event.timestamp}|1|count|domain.event|#${tags.join()}`);
};

...
```

1.  使用 `npm install` 安装依赖项。

1.  使用 `npm test -- -s $MY_STAGE` 运行测试。

1.  查看在 `.serverless` 目录中生成的内容。

1.  部署堆栈：`npm run dp:lcl -- -s $MY_STAGE`

1.  在 AWS 控制台中查看堆栈和资源。

1.  使用以下命令调用`simulate`函数：

```js
$ sls invoke -f simulate -r us-east-1 -s $MY_STAGE 
[
    {
        "total": 4500,
        "purple": 1151,
        "green": 1132,
        "blue": 1069,
        "orange": 1148
    }
]
```

1.  查看日志：

```js
$ sls logs -f listener -r us-east-1 -s $MY_STAGE
...
... MONITORING|1531020609200|1|count|domain.event|#account:cncb,...,type:purple
```

1.  在 Datadog 的指标下探索事件指标，使用图形：`domain.event`，每个`type`一个图形。

1.  完成后删除堆栈：`npm run rm:lcl -- -s $MY_STAGE`

# 它是如何工作的...

监控事件的工作方式与在数据湖中收集事件类似。单个流处理器观察所有流的所有事件，并简单地按事件`type`计数域事件，以及额外的标签，如`region`、`stream`和`source`。同样，这些计数被记录为结构化日志语句，Datadog 从 CloudWatch 日志中消费这些语句。在仪表板上绘制域事件指标可以提供对系统行为的深入了解。我们将在*创建警报*配方中看到如何对域事件流发出警报。我们还会对`fault`事件进行特殊处理。对于这些事件，我们调用 Datadog 事件 API，该 API 提供发送附加上下文信息的功能，例如堆栈跟踪。我们将在第八章，*设计失败*中讨论`fault`事件。

# 创建警报

为了最大限度地提高我们对云原生服务的信心，我们需要在最终用户之前就收到问题警报，以便我们能够快速响应并最小化平均恢复时间。这也意味着我们需要消除警报疲劳，并且只对真正重要的事情发出警报，否则重要的警报将淹没在噪音中。本配方演示了如何创建关键指标的警报。

# 如何做...

1.  登录到您的 Datadog 账户。

1.  使用以下设置创建一个 IAM 警报：

    +   选择监控器 | 新监控器 | 事件

    +   匹配来自`Amazon Cloudtrail`的`aws-service:iam`事件

    +   多警报—`aws_account`

    +   警报阈值—`above 0`

    +   标题—`aws.iam`

    +   包含触发标签—`true`

    +   通知—`yourself`

1.  使用以下设置创建一个年龄迭代器警报：

    +   选择监控器 | 新监控器 | 集成 | AWS Lambda

    +   指标—`aws.lambda.iterator_age`

    +   最大值按—`functionname`、`region`和`account`

    +   多警报—`functionname`、`region`和`account`

    +   警报阈值—`7200000` (2 小时)

    +   警告阈值—`1800000`  (0.5 小时)

    +   标题—`aws.lambda.iterator_age`

    +   包含触发标签—`true`

    +   通知—`yourself`

1.  使用以下设置创建一个请求速率警报：

    +   选择监控器 | 新监控器 | 异常

    +   指标—`aws.apigateway.count`

    +   平均值按—`apiname`、`region`和`account`作为`rate`

    +   多警报—`apiname`、`region`和`account`

    +   警报条件—使用默认值并调整到您的数据

    +   标题—`aws.apigateway.rate`

    +   包含触发标签—`true`

    +   通知—`yourself`

1.  使用以下设置创建一个域事件速率警报：

    +   选择监控器 | 新监控器 | 异常

    +   指标—`domain.event`

    +   平均值按—`type`、`region`和`account`作为`rate`

    +   多警报—`type`、`region`和`account`

    +   警报条件—从默认值开始，并根据您的数据进行调整

    +   标题—`domain.event.rate`

    +   包含触发标签—`true`

    +   通知—`yourself`

1.  使用以下设置创建一个故障事件警报：

    +   选择监控器 | 新监控器 | 事件

    +   匹配包含—`Fault Event`且状态为`error`的事件，来自：`My Apps`

    +   多警报—`functionname`、`region`和`account`

    +   警报阈值—`above 0`

    +   标题—`fault.event`

    +   包含触发标签—`true`

    +   通知—`yourself`

Datadog 自动完成菜单是根据最近收集的指标填充的。

# 它是如何工作的...

现在我们系统是可观察的，我们需要用所有这些数据进行一些积极主动且有用的事情。手动处理的数据太多，而且如果我们能将这些数据转化为有价值、及时的信息，我们的信心就会增加。我们肯定会使用这些数据进行根本原因和事后分析，但我们的信心是通过关注平均恢复时间来增加的。因此，我们需要创建不断*测试*数据、将其转化为信息并在重要事项上发出警报的监控器。然而，我们必须小心避免警报疲劳。最佳实践是广泛发出警报，但明智地针对症状而不是原因进行页面。例如，我们应该创建许多仅记录阈值被跨越的监控器，以便在根本原因分析中使用这些附加信息。其他监控器将通过电子邮件向团队发出潜在问题的警告，但少数精选监控器将向团队发送页面，以便立即采取行动。

为了了解症状和原因之间的区别，我们将我们的指标分为工作指标和资源指标。工作指标代表系统对用户可见的输出。资源指标代表系统的内部工作。我们的资源监控器通常会记录并发送警告，而我们的工作监控器会向团队发送页面。RED 方法([`dzone.com/articles/red-method-for-prometheus-3-key-metrics-for-micros`](https://dzone.com/articles/red-method-for-prometheus-3-key-metrics-for-micros))和 USE 方法([`dzone.com/articles/red-method-for-prometheus-3-key-metrics-for-micros`](https://dzone.com/articles/red-method-for-prometheus-3-key-metrics-for-micros))进一步细分了这些类别。**RED**代表**速率、错误和持续时间**。当一个关键服务的请求数或事件数显著减少，或错误显著增加，或者延迟显著增加时，这可能需要向团队发送页面。**USE**代表**利用率、饱和度和错误**。当资源（如 DynamoDB 或 Kinesis）的利用率达到一定水平时，可能需要向团队发出警告。然而，饱和度和/或错误（如限流）可能只需要记录，因为它们可能会迅速减轻，或者如果持续存在，它们将触发工作监控器。

此配方演示了几种可能的监控器。`故障`监控器代表失败的工作，必须解决。`流迭代器年龄`监控器处于边缘，因为它可能代表临时资源饱和，也可能代表导致工作积压的错误。因此，它在不同的阈值处都有警告和警报。`异常检测`监控器应关注工作指标，例如关键请求的速率或域事件。同时监控 CloudTrail 以检测任何 IAM 更改，例如角色和权限的更改也是一个好主意。

如果您只需要记录条件，则`通知`步骤是可选的。为了警告团队，请将通知发送到聊天和/或群组电子邮件。为了唤醒团队，请将通知发送到 SNS 主题。最好使用 Multi Alert 功能，在相关标签上触发，并在通知标题中包含这些信息，以便一目了然地获取这些信息。

最终，为了有价值并避免疲劳，这些监控器需要得到关注和培养。这些监控器是您在生产中的测试。随着您的团队对系统的理解加深，您将发现更好的测试/监控器。当您的监控器产生误报时，它们需要调整或消除。您的信心水平是成功监控的真实衡量标准。

# 创建合成事务测试

如果森林里有一棵树倒下，没有人听到声音，它还会发出声音吗？或者更贴近话题，如果某个地区的部署出现故障，没有人醒来使用它，这会有所不同吗？当然，答案是肯定的。我们希望被通知关于损坏的部署，以便在正常流量开始之前修复它。为了启用此功能，我们需要通过系统连续泵送合成流量，以便有一个连续的信号进行测试。此配方演示了如何使用云提供商无关的第三方服务生成合成流量。

# 准备工作

在开始此配方之前，您需要一个 Pingdom 账户（[`www.pingdom.com`](https://www.pingdom.com)）。您还需要一个 AWS Cognito 用户池，例如在*创建联合身份池*配方中创建的那个。

# 如何做到这一点...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch7/synthetics --path cncb-synthetics
```

1.  导航到`cncb-synthetics`目录`cd cncb-synthetics`。

1.  使用`npm install`安装依赖项。

1.  检查名为`src/App.js`的文件，并使用用户池堆栈的值更新`clientId`和`domain`字段。

1.  使用`npm run build`构建应用程序。

1.  部署堆栈：

部署 CloudFront 分发可能需要 20 分钟以上。

```js
$ npm run dp:lcl -- -s $MY_STAGE

> cncb-synthetics@1.0.0 dp:lcl <path-to-your-workspace>/cncb-synthetics
> sls deploy -r us-east-1 --account cncb "-s" "john"
...
WebsiteDistributionURL: https://dqvo8ga8z7ao3.cloudfront.net
```

1.  更新用户池堆栈的`callbackURLs`和`logoutURLs`以包括`WebsiteDistributionURL`，然后重新部署。

1.  浏览堆栈输出中提供的`WebsiteDistributionURL`以测试网站配置。

1.  登录到您的 Pingdom 账户。

1.  使用以下设置创建一个可用性检查：

    +   选择“经验监控”|“可用性”|“添加检查”。

    +   名称—`cncb-synthetics`。

    +   检查间隔—`1 分钟`

    +   URL—`WebsiteDistributionURL`（从您的部署输出中获取）

1.  定期审查经验监控 *|* 服务器正常运行时间仪表板。

1.  使用以下设置创建真实用户监控：

    +   选择“经验监控 | 访客洞察（RUM） | 添加站点”。

    +   名称—`cncb-synthetics`

    +   URL—`WebsiteDistributionURL`（从您的部署输出中获取）

1.  审查名为`public/index.html`的文件，取消以下代码的注释，并将 ID 替换为生成的代码片段中的值：

```js
    <!--
    <script src="img/strong>.js" async></script>
    -->
```

1.  构建和重新部署应用程序：

```js
$ npm run build
$ npm run dp:lcl -- -s $MY_STAGE
```

1.  定期审查经验监控 | 访客洞察（RUM）仪表板。

1.  使用以下设置创建合成事务测试：

    +   选择“经验监控 | 事务 | 添加检查”。

    +   名称—`cncb-synthetics`

    +   测试间隔—`10 分钟`

    +   前往`WebsiteDistributionURL` URL

    +   在`username`字段中填写`your-username`

    +   在`password`字段中填写`your-password`

    +   点击“登录”

    +   等待元素`.App-title`包含`Welcome to React`

1.  定期审查经验监控 *|* 事务仪表板。

1.  完成后，使用`npm run rm:lcl -- -s $MY_STAGE`删除堆栈。

1.  在 14 天试用期到期前，在“账户 | 订阅 | 管理计划”下取消 Pingdom 免费试用，以避免费用。

# 它是如何工作的...

生产环境中的测试与传统的前生产测试不同。正如在“创建警报”配方中所示，我们的生产测试是以监控的形式实现的，这些监控不断测试系统发出的信号。然而，如果没有流量，就没有信号，因此也就没有测试来提醒我们这些期间发生的任何部署中的问题。这反过来又降低了我们对这些部署的信心。解决方案是在没有自然流量时生成稳定的合成流量来填补空白。

服务器正常运行检查是最容易实施的，因为它们只进行单个请求。这些至少应该包括在内，因为它们可以快速实施且几乎不需要努力，并且它们具有最高的频率。**真实用户监控**（**RUM**）应该包括在内，因为它只需要简单的代码修改，并且云原生系统中大量的用户体验是由单页应用程序在浏览器中执行的。最后，需要实施一组小型但战略性的合成事务脚本，以持续测试关键功能。这些脚本类似于传统的测试脚本，但它们的重点是持续执行这些关键的成功路径，以确保关键功能不受新功能持续部署的影响。
