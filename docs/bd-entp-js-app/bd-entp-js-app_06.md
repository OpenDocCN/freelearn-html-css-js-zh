# 第六章：在 Elasticsearch 中存储数据

在上一章中，我们通过遵循 TDD 流程并首先编写所有端到端测试用例来开发了大部分的创建用户功能。最后一部分是实际上将用户数据持久化到数据库中。

在本章中，我们将在本地开发机器上安装和运行 **ElasticSearch**，并将其用作我们的数据库。然后，我们将实现我们最后剩余的步骤定义，使用它来驱动我们的应用程序代码的开发。具体来说，我们将涵盖以下内容：

+   安装 Java 和 Elasticsearch

+   理解 Elasticsearch 概念，例如 **索引**、**类型**和**文档**

+   使用 Elasticsearch JavaScript 客户端完成我们的创建用户端点

+   编写一个 Bash 脚本来使用单个命令运行我们的端到端测试

# Elasticsearch 简介

那么，Elasticsearch 是什么呢？首先，Elasticsearch 不应被视为一个单一、一维的工具。相反，它是一套工具，包括一个**分布式数据库**、一个**全文搜索引擎**，以及一个**分析引擎**。在本章中，我们将重点关注“数据库”部分，稍后处理“分布式”和“全文搜索”部分。

在其核心，Elasticsearch 是一个针对 Apache Lucene 的高级抽象层，Apache Lucene 是一个全文搜索引擎。Lucene 可以说是最强大的全文搜索引擎之一；它被 Apache Solr 使用，另一个类似于 Elasticsearch 的搜索平台。然而，Lucene 非常复杂，入门门槛高；因此，Elasticsearch 将这种复杂性抽象成 RESTful API。

我们可以直接使用 Java 与 Lucene 交互，而不是发送 HTTP 请求到 API。此外，Elasticsearch 还提供了许多特定语言的客户端，它们将 API 进一步抽象成包装良好的对象和方法。我们将使用 Elasticsearch 的 JavaScript 客户端来与我们的数据库交互。

你可以在 [`www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/api-reference.html`](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/api-reference.html) 找到最新 JavaScript 客户端的文档。

# Elasticsearch 与其他分布式文档存储的比较

对于简单的文档存储，你通常会选择一个通用数据库，如 MongoDB，并存储**规范化**的数据。

规范化是通过确保数据结构组件是原子元素来减少数据冗余并提高数据完整性的过程。

反规范化是通过引入数据冗余以获得其他好处的过程，例如性能。

然而，在规范化数据上搜索效率极低。因此，为了执行全文搜索，你通常会**反规范化**数据并将其复制到更专业的数据库，如 Elasticsearch。

因此，在大多数设置中，您可能需要运行两个不同的数据库。然而，在这本书中，我们将使用 Elasticsearch 进行数据存储和搜索，以下是一些原因：

+   这本书的页数有限

+   在 MongoDB 与 Elasticsearch 同步方面的工具还不够成熟

+   我们的数据需求非常基础，所以这不会造成太大的差异

# 安装 Java 和 Elasticsearch

首先，让我们安装 Elasticsearch 和其依赖项。Apache Lucene 和 Elasticsearch 都是用 Java 编写的，因此我们必须首先安装 Java。

# 安装 Java

当您安装 Java 时，通常意味着以下两种情况之一：您正在安装 **Java 运行时环境**（**JRE**）或 **Java 开发工具包**（**JDK**）。JRE 提供了运行 Java 程序所需的运行时，而 JDK 包含了 JRE 以及其他工具，允许您在 Java 中进行开发。

我们将在这里安装 JDK，但为了使事情更加复杂，JDK 有不同的实现——OpenJDK、Oracle Java、IBM Java——我们将使用的是随 Ubuntu 安装提供的 `default-jdk` APT 软件包：

```js
$ sudo apt update
$ sudo apt install default-jdk
```

接下来，我们需要设置一个系统级别的环境变量，以便其他使用 Java 的程序（例如 Elasticsearch）知道它的位置。运行以下命令以获取 Java 安装列表：

```js
$ sudo update-alternatives --config java
There is only one alternative in link group java (providing /usr/bin/java): /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java
Nothing to configure.
```

对于我的机器，只有一个 Java 安装，位于 `/usr/lib/jvm/java-8-openjdk-amd64/`。然而，如果您机器上有多个 Java 版本，您将被提示选择您喜欢的版本：

```js
$ sudo update-alternatives --config java
There are 2 choices for the alternative java (providing /usr/bin/java).

 Selection Path Priority Status
------------------------------------------------------------
* 0 /usr/lib/jvm/java-11-openjdk-amd64/bin/java 1101 auto mode
 1 /usr/lib/jvm/java-11-openjdk-amd64/bin/java 1101 manual mode
 2 /usr/lib/jvm/java-8-openjdk-amd64/jre/bin/java 1081 manual mode

Press <enter> to keep the current choice[*], or type selection number: 
```

接下来，打开 `/etc/environment` 文件并添加 `JAVA_HOME` 环境变量的路径：

```js
JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64"
```

`JAVA_HOME` 将在用户登录时为任何用户设置；要立即应用更改，我们需要源文件：

```js
$ . /etc/environment
$ echo $JAVA_HOME
/usr/lib/jvm/java-8-openjdk-amd64
```

# 安装和启动 Elasticsearch

前往 [elastic.co/downloads/elasticsearch](https://www.elastic.co/downloads/elasticsearch) 并下载适合您机器的最新 Elasticsearch 版本。对于 Ubuntu，我们可以下载官方的 `.deb` 软件包并使用 `dpkg` 进行安装：

```js
$ sudo dpkg -i elasticsearch-6.3.2.deb
```

您的 Elasticsearch 版本可能与这里的不同。这没关系。

接下来，我们需要配置 Elasticsearch 以使用我们刚刚安装的 Java 版本。我们已经为整个系统做了这件事，但 Elasticsearch 也有自己的配置文件来指定 Java 二进制文件的路径。打开 `/etc/default/elasticsearch` 文件并添加一个 `JAVA_HOME` 变量的条目，就像之前做的那样：

```js
# Elasticsearch Java path
JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

现在，我们可以开始使用 Elasticsearch 了！Elasticsearch 作为服务安装，因此我们可以使用 `systemctl` 来启动和停止它：

```js
sudo systemctl start elasticsearch.service
sudo systemctl stop elasticsearch.service
```

为了简化开发，我们可以通过启用它来让 Elasticsearch 在系统重启时自动启动：

```js
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch.service
```

现在，我们可以使用 `systemctl` 检查 Elasticsearch 是否正在运行：

```js
$ sudo systemctl start elasticsearch.service
$ sudo systemctl status elasticsearch.service
● elasticsearch.service - Elasticsearch
 Loaded: loaded (/usr/lib/systemd/system/elasticsearch.service; enabled; vendo
 Active: active (running) since Wed 2017-12-27 17:52:06 GMT; 4s ago
 Docs: http://www.elastic.co
 Main PID: 20699 (java)
 Tasks: 42 (limit: 4915)
 Memory: 1.1G
 CPU: 12.431s
 CGroup: /system.slice/elasticsearch.service
 └─20699 /usr/lib/jvm/java-8-openjdk-amd64/bin/java -Xms1g -Xmx1g -XX:

Dec 27 17:52:06 nucleolus systemd[1]: Started Elasticsearch.
```

或者，一个更直接的方法就是向 Elasticsearch 默认端口 `9200` 发送查询：

```js
$ curl 'http://localhost:9200/?pretty'
{
 "name" : "6pAE96Q",
 "cluster_name" : "elasticsearch",
 "cluster_uuid" : "n6vLxwydTmeN4H6rX0tqlA",
 "version" : {
 "number" : "6.3.2",
 "build_date" : "2018-07-20T05:20:23.451332Z",
 "lucene_version" : "7.3.1"
 },
 "tagline" : "You Know, for Search"

}
```

我们收到了回复，这意味着 Elasticsearch 正在您的机器上运行！

# 理解 Elasticsearch 的关键概念

我们很快就会向 Elasticsearch 发送查询，但如果我们理解一些基本概念会很有帮助。

# Elasticsearch 是一个 JSON 文档存储

如您可能已经从我们的 API 调用响应体中注意到，Elasticsearch 以**JavaScript 对象表示法**（**JSON**）格式存储数据。这使得开发者能够存储比**关系型数据库**强加的具有更复杂（通常是嵌套）结构的对象，后者具有**行**和**表**的扁平结构。

这并不是说文档数据库比关系型数据库更好，或者反之亦然；它们是不同的，它们的适用性取决于它们的使用。

# 文档与关系数据存储对比

例如，你的应用程序可能是一个学校目录，存储有关学校、用户（包括教师、员工、家长和学生）、考试、教室、班级以及它们之间关系的信息。鉴于数据结构可以保持相对扁平（即，主要是简单的键值条目），关系型数据库将是最合适的。

另一方面，如果你正在构建一个社交网络，并想存储用户的设置，文档数据库可能更适合。这是因为设置可能相当复杂，例如这里所示：

```js
{
  "profile": {
    "firstName": "",
    "lastName": "",
    "avatar": "",
    "cover": "",
    "color": "#fedcab"
  },
  "active": true,
  "notifications": {
    "email": {
      "disable": false,
      "like": true,
      "comment": true,
      "follow": true
    },
    "app": {
      "disable": false,
      "like": true,
      "comment": true,
      "follow": true }
    }};
```

在关系型数据库中，你必须为列（例如`settings.notification.app.follow`）建立命名约定，以便保留层次信息。然而，要使用这些设置，你必须手动在可以处理它之前重建对象。每次检索条目时都需要这样做。

将此用户信息作为文档存储允许你以对象本身的形式存储它们，保留其结构，并且可以以它们本身的形式检索它们，而无需进行额外的工作。

几个关系型数据库已经开始允许用户将文档作为值存储。例如，从 MySQL 5.7 开始，你可以存储无模式的文档。

然而，如果你的意图是以非关系型方式结构化你的数据，那么从一开始就使用一个 NoSQL 数据库会更好。我建议只有在你有现有数据并且在其之上添加新的数据结构时，才将文档存储在传统的数据库中。

# 理解索引、类型、文档和版本

在 Elasticsearch 中，每个文档都由四个属性唯一标识：其**索引**、**类型**、**ID**和**版本**。

相关文档应存储在同一个索引下。虽然不完全相同，但索引在关系型数据库中类似于数据库。例如，我们用户目录 API 中使用的所有文档可能都存储在`directory`索引中，或者由于我们的平台名为 Hobnob，我们也可以将我们的索引命名为`hobnob`。

存储在索引内的文档必须属于某个类型。对于我们的用户目录 API，你可能有一些属于`person`和`company`类型的文档。虽然不完全相同，但类型在关系型数据库中类似于表。

每个文档也必须有一个 ID 和版本。每当以任何方式修改文档时，其版本都会增加一定量（通常是 `1`）。

Elasticsearch 不存储文档的旧版本。版本计数器存在是为了允许我们执行 **并发更新** 和 **乐观锁定**（稍后详细介绍这些技术）。

# 从端到端测试中查询 Elasticsearch

我们现在在 Elasticsearch 中拥有了实现最后未定义步骤定义所需的所有知识，该步骤定义从数据库读取以查看我们的用户文档是否已正确索引。我们将使用 JavaScript 客户端，它只是 REST API 的包装器，与其端点一对一映射。因此，首先，让我们安装它：

```js
$ yarn add elasticsearch
```

接下来，将包导入到我们的 `spec/cucumber/steps/index.js` 文件中，并创建一个 `elasticsearch.Client` 实例：

```js
const client = new elasticsearch.Client({
  host: `${process.env.ELASTICSEARCH_PROTOCOL}://${process.env.ELASTICSEARCH_HOSTNAME}:${process.env.ELASTICSEARCH_PORT}`,
});
```

默认情况下，Elasticsearch 在端口 `9200` 上运行。然而，为了避免硬编码的值，我们明确传递了一个选项对象，指定了 `host` 选项，其值来自环境变量。为了使这生效，请将这些环境变量添加到我们的 `.env` 和 `.env.example` 文件中：

```js
ELASTICSEARCH_PROTOCOL=http
ELASTICSEARCH_HOSTNAME=localhost
ELASTICSEARCH_PORT=9200
```

要查看 `elasticsearch.Client` 构造函数接受的选项的完整列表，请查看 [elastic.co/guide/en/elasticsearch/client/javascript-api/current/configuration.html](https://www.elastic.co/guide/en/elasticsearch/client/javascript-api/current/configuration.html)。

如我们的 Cucumber 测试场景中指定，我们需要创建用户端点返回一个字符串，我们将其存储在 `this.responsePayload` 中。这应该是用户的 ID。因此，如果我们可以使用此 ID 再次找到用户文档，这意味着文档在数据库中，我们已经完成了我们的特性。

要通过 ID 查找文档，我们可以使用 Elasticsearch 客户端的 `get` 方法，该方法将根据其 ID 从索引中获取一个类型化的 JSON 文档。Elasticsearch 客户端中的所有方法都是异步的——如果我们提供了一个回调，它将调用该回调；否则，它将返回一个承诺。

Elasticsearch 的结果将具有以下结构：

```js
{ _index: <index>,
  _type: <type>,
  _id: <id>,
  _version: <version>,
  found: true,
  _source: <document> }
```

`_source` 属性包含实际的文档。为了确保它与请求中发送的相同，我们可以使用 Node 的 `assert` 模块的 `deepEqual` 方法来比较 `_source` 文档与 `this.requestPayload`。

根据这些信息，尝试自己实现最终步骤定义，并在此处查看答案：

```js
Then(/^the payload object should be added to the database, grouped under the "([a-zA-Z]+)" type$/, function (type, callback) {
  client.get({
    index: 'hobnob',
    type,
    id: this.responsePayload,
  }).then((result) => {
    assert.deepEqual(result._source, this.requestPayload);
    callback();
  }).catch(callback);
});
```

ESLint 可能会抱怨 `_source` 违反了 `no-underscore-dangle` 规则。传统上，标识符中的下划线用于表示变量或方法应该是“私有的”，但由于 JavaScript 中没有真正的私有变量，这种约定非常具有争议性。

然而，我们在这里使用的是 Elasticsearch 客户端，这是它们的约定。因此，我们应该在项目级别的 `.eslintrc` 文件中添加一个规则来禁用此规则。

再次运行测试，应该不再有未定义的步骤定义。但是，它仍然失败，因为我们还没有在我们的`src/index.js`中实现实际的成功场景。所以，让我们开始吧！

# 将文档索引到 Elasticsearch

在`src/index.js`中，导入 Elasticsearch 库并像之前一样初始化客户端；然后在`POST /users`的请求处理器中，使用 Elasticsearch JavaScript 客户端的`index`方法将有效负载对象添加到 Elasticsearch 索引中：

```js
import elasticsearch from 'elasticsearch';
const client = new elasticsearch.Client({
  host: `${process.env.ELASTICSEARCH_PROTOCOL}://${process.env.ELASTICSEARCH_HOSTNAME}:${process.env.ELASTICSEARCH_PORT}`,
});
...

app.post('/users', (req, res, next) => {
  ...
  client.index({
    index: 'hobnob',
    type: 'user',
    body: req.body
  })
}
```

`index`方法返回一个承诺，它应该解析为类似以下内容：

```js
{ _index: 'hobnob',
  _type: 'users',
  _id: 'AV7HyAlRmIBlG9P7rgWY',
  _version: 1,
  result: 'created',
  _shards: { total: 2, successful: 1, failed: 0 },
  created: true }
```

我们可以向客户端返回的唯一有用且相关的信息是新自动生成的`_id`字段。因此，我们应该提取该信息，并使函数返回一个承诺，该承诺解析为仅`_id`字段值。作为最后的手段，返回一个`500 内部服务器错误`，以向客户端表明他们的请求是有效的，但我们的服务器遇到了一些问题：

```js
client.index({
  index: 'hobnob',
  type: 'user',
  body: req.body,
}).then((result) => {
  res.status(201);
  res.set('Content-Type', 'text/plain');
  res.send(result._id);
}).catch(() => {
  res.status(500);
  res.set('Content-Type', 'application/json');
  res.json({ message: 'Internal Server Error' });
});
```

现在，我们的端到端测试应该全部通过！

# 清理测试后的工作

当我们运行测试时，它将用户文档索引到我们的本地开发数据库。经过多次运行，我们的数据库将充满大量测试用户文档。理想情况下，我们希望所有测试都是自包含的。这意味着每次测试运行时，我们应该将数据库的状态重置到测试运行之前的状态。为了实现这一点，我们必须对我们的测试代码进行两项进一步的修改：

+   在我们做出必要的断言后删除测试用户

+   在测试数据库上运行测试；在 Elasticsearch 的情况下，我们可以简单地为我们的测试使用不同的索引

# 删除我们的测试用户

首先，在 Cucumber 规范的特征列表中添加一个新条目：

```js
...
And the payload of the response should be a string
And the payload object should be added to the database, grouped under the "user" type
And the newly-created user should be deleted
```

接下来，为这个步骤定义相应的步骤定义。但首先，我们将修改索引文档的步骤定义，并将其更改为在上下文中持久化文档类型：

```js
Then(/^the payload object should be added to the database, grouped under the "([a-zA-Z]+)" type$/, function (type, callback) {
  this.type = type;
  client.get({
    index: 'hobnob'
    type: type,
    id: this.responsePayload
  })
  ...
});
```

然后，添加一个新的步骤定义，该定义使用`client.delete`通过 ID 删除文档：

```js
Then('the newly-created user should be deleted', function () {
  client.delete({
    index: 'hobnob',
    type: this.type,
    id: this.responsePayload,
  });
});
```

`delete`方法的输出看起来像这样：

```js
{ _index: 'hobnob',
  _type: 'user',
  _id: 'N2hWu2ABiAD9b15yOZTt',
  _version: 2,
  result: 'deleted',
  _shards: { total: 2, successful: 1, failed: 0 },
  _seq_no: 4,
  _primary_term: 2 }
```

成功操作将将其`result`属性设置为`'deleted'`；因此，我们可以用它来断言步骤是否成功。更新步骤定义如下：

```js
Then('the newly-created user should be deleted', function (callback) {
  client.delete({
    index: 'hobnob',
    type: this.type,
    id: this.responsePayload,
  }).then(function (res) {
    assert.equal(res.result, 'deleted');
    callback();
  }).catch(callback);
});
```

运行测试并确保它们通过。我们现在已经实现了我们的快乐路径/成功场景，所以现在是时候提交我们的更改：

```js
$ git add -A && git commit -m "Implement Create User success scenario"
```

# 提高我们的测试体验

尽管我们现在正在清理自己的事情，但使用相同的索引进行测试和开发并不理想。相反，我们应该为开发使用一个索引，为测试使用另一个索引。

# 在测试数据库中运行测试

对于我们的项目，让我们在开发时使用索引名称 `hobnob`，在测试时使用 `test`。我们不需要将索引名称硬编码到我们的代码中，可以使用环境变量来动态设置它。因此，在我们的应用程序和测试代码中，将所有 `index: 'hobnob'` 的实例替换为 `index: process.env.ELASTICSEARCH_INDEX`。

目前，我们正在使用 `dotenv-cli` 包来加载我们的环境变量。实际上，该包还提供了一个 `-e` 标志，允许我们加载多个文件。这意味着我们可以在 `.env` 文件中存储默认环境变量，并创建一个新的 `test.env` 来存储特定于测试的环境变量，这将覆盖默认值。

因此，将以下行添加到我们的 `.env` 文件中：

```js
ELASTICSEARCH_INDEX=hobnob
```

然后，创建两个新文件——`test.env` 和 `test.env.example`——并添加以下行：

```js
ELASTICSEARCH_INDEX=test
```

最后，更新我们的 `test` 脚本，在默认之前加载测试环境。

```js
"test:e2e": "dotenv -e test.env -e .env cucumber-js -- spec/cucumber/features --require-module @babel/register --require spec/cucumber/steps",
```

停止 API 服务器，并使用以下命令重新启动它：

```js
$ npx dotenv -e test.env yarn run watch
```

再次运行我们的端到端测试，它们应该全部通过。现在唯一的区别是测试根本不会影响我们的开发索引！

最后，为了整理一下，让我们将所有环境文件移动到一个名为 `envs` 的新目录中，并更新我们的 `.gitignore` 以忽略所有具有 `.env` 扩展名的文件：

```js
$ mkdir envs && mv -t envs .env .env.example test.env test.env.example
$ sed -i 's/^.env$/*.env/g' .gitignore
```

当然，您还需要更新您的 `serve` 和 `test` 脚本：

```js
"serve": "yarn run build && dotenv -e envs/.env node dist/index.js",
"test:e2e": "dotenv -e envs/test.env -e envs/.env cucumber-js -- spec/cucumber/features --require-module @babel/register --require spec/cucumber/steps",
```

再次运行测试并确保它们通过。一旦您满意，将这些更改提交到 Git：

```js
$ git add -A && git commit -m "Use test index for E2E tests"
```

# 分离开发和测试服务器

干得好。使用测试数据库确实是一个进步，但我们的测试工作流程仍然不连贯。目前，为了运行我们的测试，我们需要停止我们的开发 API 服务器，设置环境变量，然后重新启动它。同样，一旦测试完成，我们还需要停止并重新启动它，以使用开发环境。

理想情况下，我们应该运行两个独立的 API 服务器实例——一个用于开发，一个用于测试——每个都绑定到自己的端口。这样，我们就不需要停止和重新启动服务器来运行测试。

为了实现这一点，只需通过在 `envs/test.env` 和 `envs/test.env.example` 中添加以下行来覆盖我们的测试环境的 `SERVER_PORT` 环境变量：

```js
SERVER_PORT=8888
```

现在，我们可以运行 `yarn run watch` 来运行我们的开发 API 服务器，并且同时运行 `npx dotenv -e envs/test.env yarn run watch` 来运行我们的测试 API 服务器。我们不再需要停止和重新启动！

虽然这是一个小的改动，但让我们仍然将其提交到我们的仓库中：

```js
$ git add -A && git commit -m "Run test API server on different port"
```

# 创建独立的端到端测试脚本

但，我们还没有完成！我们绝对可以进一步提高我们的测试工作流程。目前，为了运行我们的端到端测试，我们必须确保以下条件：

+   一个 Elasticsearch 实例正在运行

+   我们使用 `dotenv-cli` 来加载我们的测试环境，然后运行我们的 API 服务器。

虽然我们可以在 `README.md` 文件中简单地记录这些说明，但如果我们提供一个单独的命令来运行，这将自动加载 Elasticsearch，设置正确的环境，运行我们的 API 服务器，运行我们的测试，并在完成后拆毁一切，这将提供更好的开发者体验。

这似乎逻辑过于复杂，不适合放入一行 npm 脚本；相反，我们可以编写一个 shell 脚本，它允许我们在文件中指定这种逻辑。我们将使用 **Bash** 作为 shell 语言，因为它是最流行和广泛支持的 shell。

对于 Windows 用户，请确保您已安装 *Windows Subsystem for Linux* (WSL)，它允许您在 Windows 机器上本地运行 GNU/Linux 工具和 Bash 脚本。您可以在 [docs.microsoft.com/en-us/windows/wsl/](https://docs.microsoft.com/en-us/windows/wsl/) 找到详细说明。

让我们先创建一个名为 `scripts` 的新目录，在其内部添加一个名为 `e2e.test.sh` 的新文件，并设置其文件权限以便可执行：

```js
$ mkdir scripts && touch scripts/e2e.test.sh && chmod +x scripts/e2e.test.sh
```

然后，更新我们的 `test:e2e` npm 脚本来执行 shell 脚本而不是直接运行 `cucumber-js` 命令：

```js
"test:e2e": "dotenv -e envs/test.env -e envs/.env ./scripts/e2e.test.sh",
```

# Shebang 解释器指令

Shell 脚本的第一行总是 shebang 解释器指令；它基本上告诉我们的 shell 应该使用哪个解释器来解析和运行此脚本文件中包含的指令。

它被称为 *shebang* 解释器指令，因为它以一个 **shebang** 开头，它只是一个由两个字符组成的序列：一个井号 (`#`) 后跟一个感叹号 (!)。

一些脚本可能用 Perl 或 Python 或其他 shell 变体编写；然而，我们的脚本将用 Bash shell 编写，因此我们应该将指令设置为 `bash` 可执行文件的位置，我们可以通过运行 `/usr/bin/env bash` 来获取。因此，在 `e2e.test.sh` 文件的第一行添加以下 shebang：

```js
#!/usr/bin/env bash
```

# 确保 Elasticsearch 正在运行

我们的 API 服务器依赖于一个活跃的 Elasticsearch 实例。因此，在我们启动 API 服务器之前，让我们确保 Elasticsearch 服务是活跃的。在 shebang 行下添加以下检查：

```js
RETRY_INTERVAL=${RETRY_INTERVAL:-0.2}
if ! systemctl --quiet is-active elasticsearch.service; then
  sudo systemctl start elasticsearch.service
  # Wait until Elasticsearch is ready to respond
  until curl --silent $ELASTICSEARCH_HOSTNAME:$ELASTICSEARCH_PORT -w "" -o /dev/null; do
    sleep $RETRY_INTERVAL
  done
fi
```

首先，我们使用 `systemctl` 的 `is-active` 命令来检查 Elasticsearch 服务是否活跃；如果活跃，命令将以 `0` 状态退出，如果不活跃，则以非零值退出。

通常，当进程成功执行时，它将以零 (`0`) 状态退出；否则，它将以非零状态码退出。在 `if` 块中，退出代码有特殊含义——`0` 退出代码表示 `true`，非零退出代码表示 `false`。

这意味着如果服务不活跃，我们将使用 `systemctl` 的 `start` 命令来启动它。然而，Elasticsearch 在能够响应请求之前需要一段时间来初始化。因此，我们使用 `curl` 轮询其端点，并阻塞下游执行，直到 Elasticsearch 准备就绪。

如果你对命令中的标志感兴趣，可以通过使用 `man` 命令获取它们的详细文档。尝试运行 `man systemctl`、`man curl`，甚至 `man man`！一些命令也支持 `-h` 或 `--help` 标志，它包含较少的信息，但通常更容易理解。

我们将每 0.2 秒重试端点一次。这由 `RETRY_INTERVAL` 环境变量设置。`${RETRY_INTERVAL:-0.2}` 语法表示如果环境变量尚未设置，我们应该仅使用 `0.2` 值；换句话说，`0.2` 值应作为默认值。

# 在后台运行测试 API 服务器

接下来，在我们能够运行测试之前，我们必须运行我们的 API 服务器。然而，API 服务器和测试需要同时运行，但终端只能附加一个前台进程组。我们希望这是我们的测试，以便在需要时与之交互（例如，停止测试）。因此，我们需要将我们的 API 服务器作为后台进程运行。

在 Bash（以及其他支持 **作业控制** 的 shell）中，我们可以通过在命令后附加单个 ampersand (`&`) 来将命令作为后台进程运行。因此，在我们的 Elasticsearch 初始化块之后添加以下行：

```js
# Run our API server as a background process
yarn run serve & 
```

# 检查我们的 API 服务器是否已准备好

接下来，我们需要运行我们的测试。但是，如果我们立即在执行 `yarn run serve &` 之后进行，它将不会工作：

```js
# This won't work!
yarn run serve &
yarn run test:e2e
```

这是因为测试是在我们的 API 服务器准备好处理请求之前运行的。因此，就像我们处理 Elasticsearch 服务一样，我们必须在运行测试之前等待我们的 API 服务器准备好。

# 使用 netstat/ss 检查 API 状态

但是，我们如何知道 API 是否已准备好？我们可以向 API 的一个端点发送请求，看看它是否返回结果。然而，这将使我们的脚本与 API 的实现耦合。更好的方法是通过检查 API 是否正在积极监听服务器端口。我们可以使用 `netstat` 工具或其替代品 `ss`（代表 **s**ocket **s**tatistics）来完成此操作。这两个命令都用于显示与网络相关的信息，例如打开的连接和套接字端口：

```js
$ netstat -lnt
$ ss -lnt
```

对于这两个命令，`-l` 标志将结果限制为仅监听套接字，`-n` 标志将显示所有主机和端口号作为数值（例如，它将输出 `127.0.0.1:631` 而不是 `127.0.0.1:ipp`），而 `-t` 标志将过滤掉非 TCP 套接字。最终的结果看起来像这样：

```js
$ netstat -lnt
Proto Recv-Q Send-Q Local Address     Foreign Address   State
tcp        0      0 127.0.0.1:5939    0.0.0.0:*         LISTEN
tcp        0      0 127.0.0.1:53      0.0.0.0:*         LISTEN
tcp6       0      0 ::1:631           :::*              LISTEN
tcp6       0      0 :::8888           :::*              LISTEN
tcp6       0      0 :::3128           :::*              LISTEN
```

要检查特定端口是否正在监听，我们可以在输出上简单地运行 `grep`（例如，`ss -lnt | grep -q 8888`）。如果 `grep` 找到结果，它将以状态码 `0` 退出；如果没有找到匹配项，它将以非零代码退出。我们可以使用 grep 的这个特性以固定间隔轮询 `ss`，直到端口绑定。

在我们的 `yarn run serve &` 命令下方添加以下块：

```js
RETRY_INTERVAL=0.2
until ss -lnt | grep -q :$SERVER_PORT; do
  sleep $RETRY_INTERVAL
done
```

# 清理后台进程

在我们可以运行测试之前，我们需要对我们的测试脚本做一些最后的修改。目前，我们正在后台运行 API 服务器。然而，当我们的脚本退出时，API 仍然会继续运行；这意味着我们下次运行 E2E 测试时将得到`listen EADDRINUSE :::8888`错误。

因此，在测试脚本退出之前，我们需要终止那个后台进程。这可以通过`kill`命令完成。在测试脚本末尾添加以下行：

```js
# Terminate all processes within the same process group by sending a SIGTERM signal
kill -15 0
```

**进程 ID**（**PID**）`0`（零）是一个特殊的 PID，它代表与触发信号的进程属于同一进程组内的所有进程。因此，我们之前的命令向同一进程组内的所有进程发送了`SIGTERM`信号（其数字代码为`15`）。

为了确保没有其他进程绑定到与我们的 API 服务器相同的端口，让我们在 Bash 脚本的开头添加一个检查，如果端口不可用，则立即退出：

```js
# Make sure the port is not already bound
if ss -lnt | grep -q :$SERVER_PORT; then
  echo "Another process is already listening to port $SERVER_PORT"
  exit 1;
fi
```

# 运行我们的测试

最后，我们能够运行我们的测试！在`kill`命令之前添加`cucumber-js`命令：

```js
npx cucumber-js spec/cucumber/features --require-module @babel/register --require spec/cucumber/steps
```

您最终的`scripts/e2e.test.sh`脚本应如下所示（注释已删除）：

```js
#!/usr/bin/env bash
if ss -lnt | grep -q :$SERVER_PORT; then
  echo "Another process is already listening to port $SERVER_PORT"
  exit 1;
fi
RETRY_INTERVAL=${RETRY_INTERVAL:-0.2}
if ! systemctl is-active --quiet elasticsearch.service; then
  sudo systemctl start elasticsearch.service
  until curl --silent $ELASTICSEARCH_HOSTNAME:$ELASTICSEARCH_PORT -w "" -o /dev/null; do
    sleep $RETRY_INTERVAL
  done
fi
yarn run serve &
until ss -lnt | grep -q :$SERVER_PORT; do
  sleep $RETRY_INTERVAL
done
npx cucumber-js spec/cucumber/features --require-module @babel/register --require spec/cucumber/steps
kill -15 0
```

为了双重检查，运行 E2E 测试以确保它们仍然通过。当然，将这些更改提交到 Git：

```js
$ git add -A && git commit -m "Make standalone E2E test script"
```

# 摘要

在本章中，我们继续在创建用户端点上的工作。具体来说，我们通过将数据持久化到 Elasticsearch 实现了成功场景。然后，我们通过创建一个 Bash 脚本来自动加载所有依赖项，在运行测试之前重构了我们的测试工作流程。

在下一章中，我们将进一步重构我们的代码，将其分解成更小的单元，并使用 Mocha、Chai 和 Sinon 编写的单元和集成测试来覆盖它们。我们还将继续实现其余的端点，确保我们遵循良好的 API 设计原则。
