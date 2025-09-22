# 构建持续部署管道

在本章中，将涵盖以下食谱：

+   创建 CI/CD 管道

+   编写单元测试

+   编写集成测试

+   为同步 API 编写合同测试

+   为异步 API 编写合同测试

+   组装传递性端到端测试

+   利用功能标志

# 简介

在前面的章节中，我们看到了云原生是如何精益和自主的。利用完全管理的云服务和建立适当的防波堤，使自给自足的全栈团队能够快速且持续地交付自主服务，并具有信心，即任何单个服务的故障都不会损害依赖于它的上游和下游服务。这种架构是一个重大进步，因为这些保障措施保护我们免受不可避免的人类错误的影响。然而，我们仍然必须努力减少人类错误并提高我们对系统的信心。

为了最小化和控制潜在的错误，我们需要最小化和控制我们的批次大小。我们通过遵循将部署与发布**解耦**的实践来实现这一点。**部署**只是将软件部署到环境中的行为，而**发布**只是将软件提供给一组用户的行为。遵循**精益**方法，我们通过一系列小而专注的实验向用户发布功能，以确定解决方案是否在正确的轨道上，以便及时进行纠正。每个实验由一系列故事组成，每个故事由一系列小而专注的任务组成。这些任务是我们的部署单元。对于每个故事，我们制定一个路线图，以连续部署这些任务，并考虑到所有相互依赖关系，以确保零停机时间。管理每个单独任务的实践统称为**任务分支工作流程**。本章中的食谱展示了任务分支工作流程的内部运作方式以及我们如何最终通过功能标志为用户提供这些功能。

# 创建 CI/CD 管道

小批次大小可以降低部署风险，因为更容易推理它们的正确性，并且在出错时更容易纠正。任务分支工作流程是一种 Git 工作流程，专注于极短生命周期的分支，范围仅为几小时而不是几天。它类似于**问题分支工作流程**，因为每个任务都作为项目管理工具中的问题进行跟踪。然而，问题的长度是模糊的，因为一个问题可以用来跟踪整个功能。这个食谱展示了在任务分支工作流程中，问题跟踪、Git 分支、拉取请求、测试、代码审查和 CI/CD 管道如何协同工作，以管理小而专注的部署单元。

# 准备工作

在开始这个食谱之前，您需要在 GitLab 上有一个账户([`about.gitlab.com/`](https://about.gitlab.com/))。

# 如何做...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch6/pipeline --path cncb-pipeline
```

1.  导航到 `cncb-pipeline` 目录，`cd cncb-pipeline`。

1.  在 `gitlab.com` 上本地和远程初始化 Git 仓库，如下所示：

```js
$ git init
$ git remote add origin git@gitlab.com:<username>/cncb-pipeline.git
$ git add .gitignore
$ git commit -m "initial commit"
$ git push -u origin master
```

1.  确认项目已在您的 Gitlab 账户中创建。

1.  在项目中创建一个名为 `intialize-project` 的新问题。

1.  点击创建合并请求按钮并注意分支名称，例如 `1-initialize-project`。

1.  使用 `git pull && git checkout 1-initialize-project` 在本地检出分支。

1.  查看名为 `.gitlab-ci.yml` 的文件，其内容如下：

```js
image: node:8

before_script:
  - cp .npmrc-conf .npmrc
  - npm install --unsafe-perm

test:
  stage: test
  script:
    - npm test
    - npm run test:int

stg-east:
  stage: deploy
  variables:
    AWS_ACCESS_KEY_ID: $DEV_AWS_ACCESS_KEY_ID
    AWS_SECRET_ACCESS_KEY: $DEV_AWS_SECRET_ACCESS_KEY
  script:
    - npm run dp:stg:e
  except:
    - master

production-east:
  stage: deploy
  variables:
    AWS_ACCESS_KEY_ID: $PROD_AWS_ACCESS_KEY_ID
    AWS_SECRET_ACCESS_KEY: $PROD_AWS_SECRET_ACCESS_KEY
  script:
    - npm run dp:prd:e
  only:
    - master
```

1.  查看名为 `package.json` 的文件，其内容如下：

```js
...
  "scripts": {
    "test": "echo running unit tests...",
    "test:int": "echo running integration tests...",
    ...
    "dp:stg:e": "sls deploy -v -r us-east-1 -s stg --acct dev",
    "dp:prd:e": "sls deploy -v -r us-east-1 -s prd --acct prod"
  },
...
```

1.  查看名为 `serverless.yml` 的文件，其内容如下：

```js
service: cncb-pipeline

provider:
  name: aws
  # cfnRole: arn:aws:iam::${self:custom.accounts.${opt:acct}.accountNumber}:role/${opt:stage}-cfnRole

custom:
  accounts:
    dev:
      accountNumber: 123456789012
    prod:
      accountNumber: 123456789012
```

1.  在 GitLab 项目（[`gitlab.com/`](https://gitlab.com/)）的设置* | *CI/CD | 变量下配置 `DEV_AWS_ACCESS_KEY_ID`、`DEV_AWS_SECRET_ACCESS_KEY`、`PROD_AWS_ACCESS_KEY_ID`、`PROD_AWS_SECRET_ACCESS_KEY` 和 `NPM_TOKEN` 环境变量。

1.  按如下方式将项目文件推送到远程仓库：

```js
$ git pull
$ git add .
$ git commit -m "initialize project"
$ git push origin 1-initialize-project
```

在推送更改之前，你应该始终执行 `git pull` 以保持你的任务分支与主分支同步。

1.  查看合并请求中的代码。

1.  查看分支管道的进度。

1.  在 AWS 控制台中查看 `cncb-pipeline-stg` 堆栈。

1.  从合并请求的名称中移除 `WIP:` 前缀并接受合并请求。

最好尽早开始拉取请求，以便尽早收到反馈。`WIP:` 前缀向审阅者表明工作仍在进行中。前缀纯粹是程序性的，但 GitLab 不会允许带有 WIP 前缀的合并请求被意外接受。

1.  查看主管道的进度。

1.  在 AWS 控制台中查看 `cncb-pipeline-prd` 堆栈。

1.  完成后，删除两个堆栈。

# 它是如何工作的...

我们使用 `GitLab.com` 简单是因为它是一个免费且托管的工具集，且集成良好。还有其他替代方案，如 Bitbucket Pipelines，虽然需要更多努力才能设置，但它们仍然提供类似的功能。比较的食谱中包含了一个 `bitbucket-pipelines.yml` 文件。

正如我们在整个食谱中看到的那样，我们的部署单元是一个堆栈，由项目目录根目录中的 `serverless.yml` 文件定义。正如我们在本食谱中看到的那样，每个项目都在其自己的 Git 仓库中管理，并有自己的 CI/CD 管道。管道由位于项目根目录的配置文件定义，例如 `.gitlab-ci.yml` 或 `bitbucket-pipelines.yml` 文件。这些管道与 Git 分支策略集成，并由拉取请求管理。

注意 GitLab 使用术语 *merge request*，而其他工具使用术语 *pull request*。这两个术语可以互换使用。

当从待办事项中拉取问题或任务以在仓库中创建分支时，任务分支工作流程开始。创建一个拉取请求来管理分支。管道在分支上执行所有测试，并在拉取请求中显示其进度。一旦测试被认为成功，管道将堆栈部署到开发账户的预发布环境中。在拉取请求中执行代码审查，并通过评论记录讨论。一旦一切就绪，拉取请求可以被接受以合并更改到主分支，并触发将堆栈部署到生产账户的生产环境的部署。

管道定义的第一行表示将使用`node:8` Docker 镜像来执行管道。管道定义的其余部分编排了我们在此菜谱中手动执行的步骤。首先，`npm install`安装了在`package.json`文件中定义的所有依赖项。然后，我们在给定的分支上执行所有测试。最后，我们使用`npm`将堆栈部署到特定的环境和区域；在这种情况下，`npm run dp:stg:e`或`npm run dp:prd:e`。每个步骤的详细信息都封装在`npm`脚本中。

注意，在整个菜谱中，我们一直在使用`npm run dp:lcl`脚本来执行我们的部署。这些允许每个开发者在一个个人堆栈（即本地或`lcl`）中进行开发和测试，以确保预发布（`stg`）环境保持稳定，从而生产（`prd`）环境也是如此。

环境变量，如`AWS_ACCESS_KEY_ID`和`AWS_SECRET_ACCESS_KEY`，由管道安全存储且不会被记录。我们为每个账户定义一组变量，由`DEV_`和`PROD_`前缀标识，然后在管道定义中映射它们。在*保护您的云账户*菜谱中，我们创建了`CiCdUser`来授予管道权限。在这里，我们需要为这些用户手动创建访问密钥并将它们作为管道变量安全存储。密钥和管道变量随后定期轮换和更新。

管道将堆栈部署到开发账户的每个任务分支的预发布环境中，以及生产账户的主分支的生产环境中。访问密钥决定了使用哪个账户，Serverless Framework `-s`选项完全限定堆栈的名称。然后我们添加一个额外的选项`--acct`，允许我们索引到账户范围的自定义变量，例如`${self:custom.accounts.${opt:acct}.accountNumber}`。为了避免在生产阶段和生产账户之间产生混淆，我们需要使用略微不同的缩写，例如`prd`和`prod`。

# 编写单元测试

单元测试可能是最重要的测试类型，并且应该当然占据测试金字塔中的大多数测试用例。测试应遵循科学方法，其中我们保持一些变量不变，调整输入，并测量输出。单元测试通过在隔离状态下测试单个单元来实现这一点。这使得单元测试能够专注于功能并最大化覆盖率。

# 如何操作...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch6/unit-testing --path cncb-unit-testing
```

1.  使用 `cd cncb-unit-testing` 命令导航到 `cncb-unit-testing` 目录。

1.  使用 `npm install` 安装依赖项。

1.  检查名为 `package.json` 的文件，其内容如下：

```js
  "scripts": {
    ...
    "pretest": "npm run clean && npm run lint",
    "test": "nyc mocha ... ./test/unit/**/*.test.js",
    ...
  },
```

1.  按以下方式运行单元测试：

```js
$ npm test
...
  14 passing (76ms)
...
===== Coverage summary =====
Statements   : 100% ( 57/57 )
Branches     : 100% ( 4/4 )
Functions    : 100% ( 28/28 )
Lines        : 100% ( 51/51 )
```

1.  检查名为 `test/unit/connector/db.test.js` 的文件，其内容如下：

```js
  it('should get by id', async () => {
    const spy = sinon.spy((params, cb) => cb(null, {
      Item: ...
    }));

    AWS.mock('DynamoDB.DocumentClient', 'get', spy);

    const data = await new Connector('t1').getById(ID);
    expect(spy).to.have.been.calledOnce;
    ...
  });
```

1.  检查名为 `test/unit/get/index.test.js` 的文件，其内容如下：

```js
  it('should get by id', async () => {
    ...
    const stub = sinon.stub(Connector.prototype, 'getById')
      .returns(Promise.resolve(THING));

    const data = await new Handler(TABLE_NAME).handle(REQUEST);
    expect(stub).to.have.been.calledWith(ID);
    ...
  });
```

1.  (可选) 使用此项目重复从“创建 CI/CD 管道”菜谱中的步骤。

# 工作原理...

在之前的菜谱中，我们故意将示例简化到合理的程度，以突出特定主题。代码是正确的，但菜谱没有包含任何单元测试，因为该主题尚未被涉及。你在这个章节的菜谱中可能首先注意到的是，我们正在向代码中添加额外的结构；例如，每个函数都有自己的目录和文件，我们还在代码中添加了一些轻量级分层。这种结构旨在通过使隔离测试单元更容易而简化测试过程。因此，现在让我们更深入地探讨已添加的工具和结构。

在 `npm test` 脚本中调用的第一个工具是 `nyc`，它是 `istanbul` 代码覆盖率工具的命令行界面。`.nycrc` 文件配置了代码覆盖率过程。在这里，我们要求达到 100% 的覆盖率。对于我们的边界、隔离和自主服务的范围来说，这是完全合理的。这也合理，因为我们正在以增量方式编写单元测试，同时也在一系列任务分支工作流程中增量构建服务。此外，如果不保持覆盖率在 100%，那么在开发过程的后期跳过测试就会变得过于容易，这在持续部署管道中是危险的，并且违背了测试的目的。幸运的是，代码的结构使得识别缺少测试的功能变得更加容易。

`npm pretest` 脚本运行 linting 过程。`eslint` 是一个非常宝贵的工具。它强制执行最佳实践，自动修复许多违规行为，并识别常见问题。从本质上讲，linting 有助于教会开发者如何编写更好的代码。可以通过 `.eslintignore` 和 `.eslintrc.js` 文件调整 linting 过程。

隔离外部依赖是单元测试的一个基本部分。我们选择的测试工具是 `mocha`、`chai`、`sinon` 和 `aws-sdk-mock`。有许多工具可用，但这个组合非常受欢迎。mocha 是总的测试框架；`chai` 是断言库；`sinon` 提供测试间谍、存根和模拟；`aws-sdk-mock` 基于 `sinon` 来简化对 `aws-sdk` 的测试。为了进一步便于这个过程，我们在连接器类中隔离了我们的 `aws-sdk` 调用。这确实有代码在整个服务中重用的额外好处，但其主要好处是简化测试。我们专门为使用 `aws-sdk-mock` 的类编写单元测试。在整个单元测试代码中，我们存根化连接器层，这极大地简化了每个测试的设置，因为我们已经隔离了 `aws-sdk` 的复杂性。

`Handler` 类负责大部分的测试。这些类封装和编排业务逻辑，因此将需要最多的测试组合。为了便于这项工作，我们将 `Handler` 类与 `callback` 函数解耦。`handle` 方法要么返回一个 *promise*，要么返回一个 *stream*，然后顶层函数将这些适配到回调。这允许测试轻松地接入处理流程以断言输出。

我们的测试本质上是异步的；因此，防止永远不失败的测试（即代码出错时不会失败的测试）非常重要。对于返回 promises 的处理器，最佳做法是使用 `async`/`await` 来防止吞没异常。对于返回 stream 的处理器，最佳做法是使用 collect/tap/done 模式来保护数据没有完全通过流流动的场景。

# 编写集成测试

集成测试专注于测试依赖服务之间的 API 调用。在我们的云原生系统中，这些集中在与完全托管的云服务之间的服务内交互。它们确保交互被正确编码以发送和接收正确的有效负载。这些调用需要网络，但网络是出了名的不可靠。这是随机和随意失败的测试不稳定的主要原因。不稳定测试反过来又是团队士气低落的主要原因。这个食谱展示了如何使用 VCR 库来创建测试替身，允许在没有网络依赖或外部服务部署的情况下独立执行集成测试。

# 准备工作

在开始这个食谱之前，你需要一个 AWS Kinesis Stream，例如在 *创建事件流* 食谱中创建的那个。

# 如何操作...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch6/integration-testing --path cncb-integration-testing
```

1.  使用 `cd cncb-integration-testing` 命令导航到 `cncb-integration-testing` 目录。

1.  使用 `npm install` 安装依赖。

1.  查看名为 `package.json` 的文件，其内容如下：

```js
  "scripts": {
    ...
    "test:int": "npm start -- --exec \"mocha ... ./test/int/**/*.test.js\"",
    "start": "sls offline start --port 3001 -r us-east-1 -s stg --acct dev",
    ...
  },
```

1.  使用 `npm test` 运行单元测试。

1.  以`replay`模式运行集成测试，如下所示：

```js
$ npm run test:int
...
Serverless: Replay mode = replay
Serverless: Starting Offline: stg/us-east-1.
...
  get/index.js
    ✓ should get by id
...
  3 passing (358ms)
...
Serverless: Halting offline server
```

1.  检查`./fixtures`目录中的文件。

1.  检查与同步集成测试相关的文件，如下所示：

```js
serverless.yml

...
plugins:
  - serverless-webpack
  - baton-vcr-serverless-plugin
  - serverless-offline
...

test/int/get/index.test.js

...
const supertest = require('supertest');
const client = supertest('http://localhost:3001');
...
describe('get/index.js', () => {
  it('should get by id', () => client
    .get('/things/00000000-0000-0000-0000-000000000000')
    .expect(200)
    .expect((res) => {
      expect(JSON.parse(res.text)).to.deep.equal(THING);
    }));
});

webpack.config.js

...
const injectMocks = (entries) =>
  Object.keys(entries).reduce((e, key) => {
    e[key] = ['./test/int/mocks.js', entries[key]];
    return e;
  }, {});

const includeMocks = () => slsw.lib.webpack.isLocal && process.env.REPLAY != 'bloody';

module.exports = {
  entry: includeMocks() ? injectMocks(slsw.lib.entries) : slsw.lib.entries,
...
```

1.  检查与异步集成测试相关的文件，如下所示：

```js
test/int/trigger/index.test.js

describe('trigger/index.js', () => {
  before(() => {
    require('baton-vcr-replay-for-aws-sdk');
    process.env.STREAM_NAME = 'stg-cncb-event-stream-s1';
    aws.config.update({ region: 'us-east-1' });
  });

  it('should trigger', (done) => {
    new Handler(process.env.STREAM_NAME).handle(TRIGGER)
      .collect()
      .tap((data) => {
        expect(data).to.deep.equal([{
          response: RESPONSE,
          event: EVENT,
        }]);
      })
      .done(done);
  });
})
```

1.  使用`npm run dp:stg:e`部署堆栈。

1.  删除`./fixtures`目录，并以`record`模式再次运行集成测试，如下所示：

```js
$ DEBUG=replay REPLAY=record npm run test:int

...
Serverless: GET /things/00000000-0000-0000-0000-000000000000 (λ: get)
  replay Requesting POST https://dynamodb.us-east-1.amazonaws.com:443/ +0ms
  replay Creating ./fixtures/dynamodb.us-east-1.amazonaws.com-443 +203ms
  replay Received 200 https://dynamodb.us-east-1.amazonaws.com:443/ +5ms
...
```

1.  （可选）使用此项目重复“创建 CI/CD 管道”配方中的步骤。

# 它是如何工作的...

集成测试使用与单元测试相同的工具，并增加了一些额外的工具。这有一个优点，就是学习曲线是逐步增加的。第一个引起兴趣的新工具是`serverless-offline`插件。此插件读取`serverless.yml`文件，并在本地模拟 API 网关，以方便同步 API 的测试。接下来，我们使用`supertest`向本地运行的服务发送 HTTP 请求并断言响应。不可避免的是，这些服务会使用默认或指定的访问密钥调用 AWS 服务。这些是我们想在 CI/CD 管道中记录和回放的调用。"baton-vcr-serverless-plugin"在`serverless-offline`过程中初始化`Replay` VCR 库。默认情况下，VCR 以`replay`模式运行，如果固定目录下找不到记录，则会失败。当编写新测试时，开发者通过设置`REPLAY`环境变量以`record`模式运行测试，`REPLAY=record npm run test:int`。为了确保找到记录，我们必须保持所有在请求中使用的动态生成的值不变；为此，我们在 webpack 配置中注入模拟。在这个例子中，`./test/int/mocks.js`文件使用`sinon`来模拟 UUID。

为异步函数，如`triggers`和`listeners`编写集成测试，大部分是相似的。首先，我们需要手动捕获日志文件中的事件，例如 DynamoDB 流事件，并将它们包含在测试用例中。然后，在测试开始时初始化`baton-vcr-replay-for-aws-sdk`库。从这里开始，记录请求的过程是相同的。对于`trigger`函数，它将记录向 Kinesis 发布事件的调用，而对于`listener`函数，通常记录对服务表的调用。

在编写集成测试时，重要的是要记住测试金字塔。集成测试专注于特定 API 调用内的交互；它们不需要覆盖所有不同的功能场景，因为这是单元测试的工作。集成测试只需要关注消息的结构，以确保它们与外部服务期望和返回的内容兼容。为了支持这些测试的记录创建，外部系统可能需要使用特定的数据集进行初始化。这个初始化应该作为测试的一部分进行编码，并记录下来。

# 为同步 API 编写合约测试

合约测试和集成测试是同一枚硬币的两面。集成测试确保消费者正确调用提供商服务，而合约测试确保提供商服务继续满足其对消费者的义务，并且任何更改都是向后兼容的。这些测试也是由消费者驱动的。这意味着消费者向提供商的项目提交拉取请求以添加这些额外的测试。提供商不应该更改这些测试。如果合约测试失败，则意味着已经做出了不兼容的更改。提供商必须使更改兼容，然后与消费者团队合作创建升级路线图。

# 如何操作...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch6/contract-testing-sync --path cncb-contract-testing-sync
```

1.  使用 `cd cncb-contract-testing-sync/bff` 命令导航到 `bff` 目录

1.  使用 `npm install` 安装依赖项。

1.  使用 `npm test` 运行单元测试。

1.  使用 `npm run test:int` 在 `replay` 模式下运行集成测试。

1.  检查 `./fixtures` 目录中的文件。

1.  检查名为 `./test/int/frontend/contract.test.js` 的文件，其内容如下：

```js
import 'mocha';

const supertest = require('supertest');
const relay = require('baton-request-relay');
const client = supertest('http://localhost:3001');

describe('contract/frontend', () => {
  it('should relay the frontend save request', 
    () => run('./fixtures/frontend/save'));

  it('should relay the frontend get request', 
    () => run('./fixtures/frontend/get'));
});

const run = fixture => {
  const rec = relay(fixture);

  return clientrec.request.method
    .set(rec.request.headers)
    .send(rec.request.body)

    .expect(rec.response.statusCode)
    .expect(rec.response.body);
};
```

1.  检查 `./frontend` 下的消费者的 `test` 和 `fixture` 文件。

1.  （可选）使用此项目重复 *创建 CI/CD 管道* 菜单中的步骤。

# 它是如何工作的...

当涉及到集成和合约测试时，魔鬼藏在细节中。具体来说，是各个字段的细节、它们的数据类型以及它们的有效值。提供商改变看似微不足道的细节，可能会违反消费者对合约的理解。这些变化往往直到最糟糕的时刻才被发现。在这方面，手工制作的合约测试是不可靠的，因为关于合约的相同误解通常也会反映在测试中。相反，我们需要在交互的双方使用相同的录音。

消费者在其项目中创建一个集成测试，记录与提供商服务的交互。然后，将这些相同的录音复制到提供商的项目中，并用于驱动合约测试。为录音创建一个针对消费者的特定 `fixtures` 子目录。合约测试使用 `baton-request-relay` 库读取录音，以便它们可以用来驱动 `supertest` 在提供商的项目中执行相同的请求。

在我们的云原生系统中，这些同步请求通常是在前端和其 **Backend for Frontend**（**BFF**）服务之间进行的，该服务由同一团队拥有。同一团队拥有消费者和提供商的事实并不否定这些测试的价值，因为即使是细微的变化，例如将短整数更改为长整数，如果任一方做出了所有错误的假设，都可能产生巨大的影响。

# 为异步 API 编写合约测试

异步 API 的合同测试目标是确保提供者和消费者之间的向后兼容性——就像同步 API 一样。测试异步通信自然是不可靠的，因为网络不可靠加上异步消息的不可靠延迟。这些测试经常失败，因为消息到达缓慢，测试超时。为了解决这个问题，我们将测试从消息系统中隔离出来；我们在一端记录发送消息请求，然后中继消息并在接收端断言合同。

# 如何操作...

1.  从以下模板创建上游和下游项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch6/contract-testing-async/upstream --path cncb-contract-testing-async-upstream

$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch6/contract-testing-async/downstream --path cncb-contract-testing-async-downstream
```

1.  使用 `cd cncb-contract-testing-async-upstream` 命令导航到 `upstream` 目录。

1.  使用 `npm install` 安装依赖。

1.  使用 `npm test` 运行单元测试。

1.  使用 `npm run test:int` 在 `replay` 模式下运行集成测试。

1.  检查 `./fixtures/downstream-consumer-x` 目录中的文件。

1.  检查名为 `./test/int/downstream-consumer-x/contract.test.js` 的文件，内容如下：

```js
...
const EVENT = require('../../../fixtures/downstream-consumer-x/thing-created.json');

describe('contract/downstream-consumer-x', () => {
  before(() => {
    const replay = require('baton-vcr-replay-for-aws-sdk');
    replay.fixtures = './fixtures/downstream-consumer-x';
    process.env.STREAM_NAME = 'stg-cncb-event-stream-s1';
    aws.config.update({ region: 'us-east-1' });
  });

  afterEach(() => {
    sinon.restore();
  });

  it('should publish thing-created', (done) => {
    sinon.stub(utils, 'uuidv4').returns('00000000-0000-0000-0000-000000000001');
    handle(EVENT, {}, done);
  });
});
```

1.  使用 `cd ../cncb-contract-testing-async-downstream` 命令导航到 `downstream` 目录。

1.  重复相同的步骤来运行测试。

1.  检查下游消费者的 `./test/int/upstream-provider-y` 和 `./fixture/upstream-provider-y` 文件，如下所示：

```js
...
const relay = require('baton-event-relay');

describe('contract/upstream-provider-y', () => {
  before(() => {
    ...
    const replay = require('baton-vcr-replay-for-aws-sdk');
    replay.fixtures = './fixtures/upstream-provider-y';
  });

  it('should process the thing-created event', (done) => {
    const rec = relay('./fixtures/upstream-provider-y/thing-created');
    handle(rec.event, {}, done);
  });
});
```

1.  （可选）使用此项目重复 *创建 CI/CD 管道* 配方中的步骤。

# 它是如何工作的...

记录和为异步 API 创建合同测试的过程与同步 API 的过程相反。对于同步 API，消费者启动交互，而对于异步 API，提供者启动事件的发布。然而，异步提供者不知道他们的消费者，因此这些测试仍然需要由消费者驱动。

首先，消费者项目在上游项目中创建一个测试，并记录向事件流发布事件的调用。然后，消费者使用 `baton-event-relay` 库在其自己的项目中中继记录，以断言事件的内容符合预期。再次强调，提供者项目不拥有这些测试，如果由于向后不兼容的更改而测试失败，不应修复测试。

# 组装传递的端到端测试

传统的端到端测试劳动密集且成本高昂。因此，传统的批量大小很大，端到端测试执行频率低，甚至完全跳过。这与我们持续部署的目标正好相反。我们希望批量大小小，并且希望每次部署都进行全面测试，每天多次，这时我们通常听到异端的呼声。这个配方展示了如何使用本章中已介绍的工具和技术实现这一点。

# 如何操作...

1.  从以下模板创建 `author-frontend`、`author-bff`、`customer-bff` 和 `customer-frontend` 项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch6/transitive-testing/author-frontend --path cncb-transitive-testing-author-frontend

$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch6/transitive-testing/author-bff --path cncb-transitive-testing-author-bff

$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch6/transitive-testing/customer-bff --path cncb-transitive-testing-customer-bff

$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch6/transitive-testing/customer-frontend --path cncb-transitive-testing-customer-frontend
```

1.  使用 `npm install` 在每个项目中安装依赖。

1.  按照以下方式在每个项目中以 `replay` 模式运行端到端测试：

```js
$ cd ./author-frontend
$ DEBUG=replay npm run test:int

$ cd ../author-bff
$ npm test
$ DEBUG=replay npm run test:int

$ cd ../customer-bff
$ npm test
$ DEBUG=replay npm run test:int

$ cd ../customer-frontend
$ DEBUG=replay npm run test:int
```

1.  审查以下所有固定文件：

```js
./author-frontend/fixtures/0.0.0.0-3001/save-thing0
./author-frontend/fixtures/0.0.0.0-3001/get-thing0

./author-bff/fixtures/author-frontend/save-thing0
./author-bff/fixtures/dynamodb.us-east-1.amazonaws.com-443/save-thing0

./author-bff/fixtures/author-frontend/get-thing0
./author-bff/fixtures/dynamodb.us-east-1.amazonaws.com-443/get-thing0

./author-bff/fixtures/downstream-customer-bff/thing0-INSERT.json
./author-bff/fixtures/kinesis.us-east-1.amazonaws.com-443/thing0-created

./customer-bff/fixtures/upstream-author-bff/thing0-created
./customer-bff/fixtures/dynamodb.us-east-1.amazonaws.com-443/save-thing0

./customer-bff/fixtures/customer-frontend/get-thing0
./customer-bff/fixtures/dynamodb.us-east-1.amazonaws.com-443/get-thing0

./customer-frontend/fixtures/0.0.0.0-3001/get-thing0
```

1.  审查以下所有端到端测试用例文件：

```js
./author-frontend/test/int/e2e.test.js

./author-bff/test/int/author-frontend/e2e.test.js
./author-bff/test/int/downstream-customer-bff/e2e.test.js

./customer-bff/test/int/upstream-author-bff/e2e.test.js
./customer-bff/test/int/customer-frontend/e2e.test.js

./customer-frontend/test/int/e2e.test.js
```

# 它是如何工作的...

集成测试和契约测试是同一枚硬币的两面。我们已经看到了如何使用测试双倍体来实现这些测试，这些测试双倍体重新播放之前记录的请求和响应对。这使得每个服务都可以在无需部署任何其他服务的情况下独立测试。服务提供者团队创建集成测试以确保他们的服务按预期工作，而服务消费者团队创建契约测试以确保提供者的服务按预期工作。然后我们在此基础上构建，使得所有团队中的测试工程师一起工作，定义足够的端到端测试场景以确保所有服务协同工作。对于每个场景，我们在所有项目中串联一系列集成和契约测试，其中来自一个项目的记录用于驱动下一个项目的测试，依此类推。借鉴等价传递性质，如果服务 *A* 产生有效载荷 1，它与服务 *B* 一起工作产生有效载荷 2，它与服务 *C* 一起工作产生预期的有效载荷 3，那么我们可以断言当服务 *A* 产生有效载荷 1 时，最终服务 *C* 将产生有效载荷 3。

在这个食谱中，我们有一个作者应用和一个客户应用。每个应用都包含一个前端项目和 bff 项目。`author-frontend` 项目制作了 `save-thing0` 记录。这个记录被复制到 `author-bff` 项目，最终产生了 `thing0-created` 记录。然后，`thing0-created` 记录被复制到 `customer-bff` 项目，最终产生了 `get-thing0` 记录。`get-thing0` 记录随后被复制到 `customer-frontend` 项目以支持其测试。

最终结果是完全自主的测试套件集合。每次修改特定服务的测试套件时都会断言该服务的测试套件，无需在每个项目中重新运行测试套件。只有不兼容的更改需要依赖项目更新其测试用例和记录；因此，我们不再需要维护端到端测试环境。不再需要数据库脚本将数据库重置到已知状态，因为数据体现在测试用例和记录中。这些传递性的端到端测试构成了一个全面测试金字塔的尖端，增加了我们对我们持续部署管道中人为错误最小化的信心。

# 利用功能标志

将部署与发布解耦的实践基于使用功能标志。我们持续部署小批量的变更以减轻每次部署的风险。这些变更一直部署到生产环境，因此我们需要一个功能标志机制来禁用这些功能，直到我们准备好发布它们并使其普遍可用。我们还需要能够为用户子集，如测试用户和内部测试人员，启用这些功能。利用系统的自然功能标志，如权限和偏好，以最小化因在代码中添加自定义功能标志而产生的技术债务也是更好的选择。这个配方将向您展示如何利用 JWT 令牌中的声明来启用和禁用功能。

# 准备工作

在开始这个配方之前，您需要一个 AWS Cognito 用户池，例如在*创建联合身份池*配方中创建的那个。用户池应该定义以下两个组：`Author`和`BetaUser`。

# 如何操作...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch6/feature-flag --path cncb-feature-flag
```

1.  使用`cd cncb-feature-flag`命令导航到`cncb-feature-flag`目录。

1.  检查名为`src/Authorize.js`的文件，如下所示：

```js
...

const getGroups = props => get(props,  
  'auth.cognito.signInUserSession.idToken.payload.cognito:groups', '');

const check = (allowedRoles, props) => {
  const groups = getGroups(props);
  return intersection(groups, allowedRoles).length > 0;
};

const HasRole = allowedRoles =>
  props => check(allowedRoles, props) ?
    props.children :
    null;

export const HasAuthorRole = HasRole(['Author']);
export const HasBetaUserRole = HasRole(['BetaUser']);
```

1.  检查名为`src/Home.js`的文件，如下所示：

```js
const Home = ({ auth }) => (
  <div>
    ...
    <HasAuthorRole>
      This is the Author Feature!
    </HasAuthorRole>
    <HasBetaUserRole>
      This is a New Beta Feature...
    </HasBetaUserRole>
    ...
  </div>
);
```

1.  检查名为`src/App.js`的文件，并使用用户池堆栈的值更新`clientId`和`domain`字段。

1.  使用`npm install`安装依赖项。

1.  使用`npm start`命令在本地运行应用程序。

1.  点击`Sign Up`链接并遵循指示。

1.  点击`Sign Out`链接，然后为另一个用户`Sign Up`。

1.  在 AWS 用户池控制台中为每个用户分配不同的组——一个分配到`Author`组，另一个分配到`BetaUser`组。

1.  以每个用户身份`Sign In`，并注意屏幕对每个用户的渲染方式都不同。

# 它是如何工作的...

首先最重要的是，零停机部署必须被视为一级要求，必须在任务路线图和系统设计中予以考虑。如果增强功能只是在一个现有的域实体上添加一个新可选字段，那么根本不需要功能标志。如果只是删除一个字段，部署的顺序很重要，从最依赖到最不依赖。如果添加了一个全新的功能，则可以限制对整个功能的访问。最有趣的场景是当对现有且流行的功能进行重大修改时。如果变化很大，那么同时支持两个版本的功能可能是最好的选择，比如两个版本的页面——在这种情况下，场景本质上与全新的功能场景相同。如果没有必要添加新版本，那么应该注意不要使代码变得过于复杂。

在这个简化的 ReactJS 示例中，JWT 令牌在登录后通过`auth`属性可访问。`HasRole`组件的实例配置了`allowedRoles`。该组件会检查 JWT 令牌在`cognito:groups`字段中是否有匹配的组。如果找到匹配项，则渲染`children`组件；否则，返回`null`且不进行渲染。
