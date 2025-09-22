# 欢迎多云

在本章中，将介绍以下食谱：

+   在 Google Cloud Functions 中创建服务

+   在 Azure Functions 中创建服务

# 简介

供应商锁定是服务器无服务器、云原生开发中一个常见的担忧。然而，这种担忧是源于必须从一家供应商整体改变到另一家供应商的单一思维模式的遗物。另一方面，自主服务可以逐个改变。尽管如此，*一次编写；到处运行*的诱人承诺仍然是*多云*方法的战斗口号。然而，这种方法忽略了只编写一次的部分只是一个非常大的冰山的一角，而水下部分则蕴含着最大的风险，并且不能直接在云服务提供商之间转换。这不可避免地导致使用一套最少共同分母的工具和技术，这些工具和技术可以更容易地从一家提供商转移到另一家提供商。

我们选择拥抱完全托管、增值云服务的**可丢弃架构**。这种**无服务器优先**的方法赋予自给自足的全栈团队以精益和实验新想法的能力，快速失败，学习，并快速调整方向。这自然地导致了一种**多语言云**或**多云**的方法，其中团队根据每个服务选择最佳的云服务提供商。最终，公司确实有一个首选的云服务提供商。但是，对于多元化，其中一部分服务是在不同的云服务提供商上实施的，以获得经验和利用，这是一个有争议的观点。目标是在具有类似甚至相同的工具、技术和模式进行开发、测试、部署和监控的服务之间实现一致的管道体验。许多前几章中的食谱专注于 AWS 完全托管、增值云服务。本章中的食谱展示了如何通过添加额外的云服务提供商，如 Google 和 Azure，实现一致的云原生开发管道体验。

# 使用 Google Cloud Functions 创建服务

**无服务器框架**在许多不同的云服务提供商之上提供了一个抽象层，这有助于实现一致的开发和部署体验。本食谱演示了如何使用 Google Cloud Functions 创建服务。

# 准备工作

在开始此食谱之前，您需要一个配置了 Serverless Framework（[`serverless.com/framework/docs/providers/google/guide/credentials`](https://serverless.com/framework/docs/providers/google/guide/credentials)）的 Google Cloud Billing Account、项目和凭证。

# 如何操作...

1.  从以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch11/gcp --path cncb-gcp
```

1.  使用 `cd cncb-gcp` 命令导航到 `cncb-gcp` 目录。

1.  查看名为 `serverless.yml` 的文件，内容如下：

```js
service: cncb-gcp

provider:
  name: google
  runtime: nodejs8
  project: cncb-project
  region: ${opt:region}
  credentials: ~/.gcloud/keyfile.json

plugins:
  - serverless-google-cloudfunctions

functions:
  hello:
    handler: hello
    events:
      - http: path
...
#resources:
#  resources:
#    - type: storage.v1.bucket
#      name: my-serverless-service-bucket
```

1.  查看名为 `index.js` 的文件，内容如下：

```js
exports.hello = (request, response) => {
  console.log('env: %j', process.env);
  response.status(200).send('... Your function executed successfully!');
};
```

1.  使用 `npm install` 安装依赖项。

1.  使用 `npm test` 运行测试。

1.  查看在 `.serverless` 目录中生成的内容。

1.  部署堆栈：

```js
$ npm run dp:lcl -- -s $MY_STAGE

> cncb-gcp@1.0.0 dp:lcl <path-to-your-workspace>/cncb-gcp
> sls deploy -v --r us-east1 "-s" "john"
...
Serverless: Done...
Service Information
service: cncb-gcp
project: cncb-project
stage: john
region: us-east1

Deployed functions
hello
  https://us-east1-cncb-project.cloudfunctions.net/hello
```

1.  在 Google Cloud 控制台中查看部署和资源。

1.  在以下命令中调用堆栈输出中显示的端点：

```js
$ curl -v https://us-east1-cncb-project.cloudfunctions.net/hello
...
JavaScript Cloud Native Development Cookbook! Your function executed successfully!
```

1.  查看日志：

```js
$ sls logs -f hello -r us-east1 -s $MY_STAGE
...
2018-08-24T05:10:20...: Function execution took 12 ms, finished with status code: 200    
...
```

1.  完成后使用 `npm run rm:lcl -- -s $MY_STAGE` 删除堆栈。

# 工作原理...

首先要注意的是，*如何操作…* 部分的步骤几乎与所有之前的菜谱相同。这是因为 Serverless Framework 抽象了部署细节，我们进一步将所有命令包装在 *NPM* 脚本中，以封装依赖项管理。从这里，我们可以使用所有相同的工具和技术进行开发和测试，如第六章构建持续部署管道中概述的。这使得团队成员在跨不同云服务提供商实现的服务工作时可以顺利过渡。

`serverless-google-cloudfunctions` 插件处理与 Google Cloud APIs（如 Cloud Functions 和 Deployment Manager）交互的细节，以提供服务。`serverless.yml` 文件看起来应该非常熟悉。我们指定 `provider.name` 为 `google` 并设置 `plugins`，然后我们专注于定义 `functions` 和 `resources`。具体细节取决于云服务提供商，但通常我们选择特定提供商的原因是特定服务增值服务的细节。`index.js` 文件中的 Node.js 代码也很熟悉，尽管函数签名不同。最终，对于实现本烹饪书各菜谱中列举的云原生模式和技术的 Google Cloud 服务有一个清晰的映射。

# 使用 Azure Functions 创建服务

Serverless Framework 在许多不同的云服务提供商之上提供了一个抽象层，这有助于实现一致的开发和部署体验。本菜谱演示了如何使用 Azure Functions 创建服务。

# 准备工作

在开始此菜谱之前，您需要一个配置了 Serverless Framework 凭据的 Azure 账户（[`serverless.com/framework/docs/providers/azure/guide/credentials`](https://serverless.com/framework/docs/providers/azure/guide/credentials)）。

# 如何操作...

1.  根据以下模板创建项目：

```js
$ sls create --template-url https://github.com/danteinc/js-cloud-native-cookbook/tree/master/ch11/azure --path cncb-azure
```

1.  使用 `cd cncb-azure` 命令进入 `cncb-azure` 目录。

1.  查看以下内容的 `serverless.yml` 文件：

```js
service: cncb-azure-${opt:stage}

provider:
  name: azure
  location: ${opt:region}

plugins:
  - serverless-azure-functions

functions:
  hello:
    handler: handler.hello
    events:
      - http: true
        x-azure-settings:
          authLevel : anonymous
      - http: true
        x-azure-settings:
          direction: out
          name: res
```

1.  查看名为 `handler.js` 的文件，其内容如下：

```js
module.exports.hello = function (context) {
  context.log('context: %j', context);
  context.log('env: %j', process.env);

  context.res = {
    status: 200,
    body: '... Your function executed successfully!',
  };

  context.done();
};
```

1.  使用 `npm install` 安装依赖项。

1.  使用 `npm test` 运行测试。

1.  查看在 `.serverless` 目录中生成的内容。

1.  部署堆栈：

```js
$ npm run dp:lcl -- -s $MY_STAGE

> cncb-azure@1.0.0 dp:lcl <path-to-your-workspace>/cncb-azure
> sls deploy -v -r 'East US' "-s" "john"
...
Serverless: Creating resource group: cncb-azure-john-rg
Serverless: Creating function app: cncb-azure-john
...
Serverless: Successfully created Function App
```

1.  在 Azure 控制台中查看部署和资源。

1.  在另一个终端中跟踪日志：

```js
$ sls logs -f hello -r 'East US' -s $MY_STAGE
Serverless: Logging in to Azure
Serverless: Pinging host status...
2018-08-25T04:02:34  Welcome, you are now connected to log-streaming service.
2018-08-25T04:05:00.843 [Info] Function started (Id=...)
2018-08-25T04:05:00.856 [Info] context: {...}
2018-08-25T04:05:00.856 [Info] env: {...}
2018-08-25T04:05:00.856 [Info] Function completed (Success, Id=..., Duration=19ms)
```

1.  在以下命令中调用堆栈输出中显示的端点：

```js
$ curl -v https://cncb-azure-$MY_STAGE.azurewebsites.net/api/hello
...
JavaScript Cloud Native Development Cookbook! Your function executed successfully!
```

1.  完成后使用 `npm run rm:lcl -- -s $MY_STAGE` 删除堆栈。

# 它是如何工作的…

首先要注意的是，“如何做…”（*How to do it…*）部分的步骤几乎与所有之前的食谱完全相同。这是因为 Serverless Framework 抽象化了部署细节，我们进一步使用*NPM scripts*将所有命令封装起来以实现依赖管理。从这里开始，我们可以使用与第六章，“构建持续部署管道”中概述的相同工具和技术进行开发和测试。这使得团队成员在跨不同云提供商实现的服务工作时能够顺利过渡。

`serverless-azure-cloudfunctions`插件处理与 Azure API 交互的细节，例如 Azure Functions 和资源管理器，以提供该服务。`serverless.yml`文件看起来应该非常熟悉。我们指定`provider.name`为`azure`并设置`plugins`，然后我们专注于定义`functions`。具体细节是云提供商特定的，但云提供商特定的增值服务细节通常是我们选择特定提供商以提供特定服务的原因。`handler.js`文件中的 Node.js 代码也很熟悉，尽管函数签名不同。最终，有一个清晰的 Azure 服务映射，用于实现本烹饪书食谱中列举的云原生模式和技术的实现。
