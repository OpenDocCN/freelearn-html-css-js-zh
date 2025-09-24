# 7

# 持久化到数据库

在应用开发的世界里，能够保存和检索数据是至关重要的。想象一下，你正在构建一个游戏，需要记录分数，或者一个社交媒体应用，用户需要保存他们的个人资料和帖子。很多时候，传统的关系型数据库正是你所需要的。它就像一个有组织的文件系统，其中每一项内容都有其所在的位置，并且这些表格可以以特定的方式相互关联。例如，一个表格可能存储关于书籍的信息，而另一个表格存储关于作者的信息，两者之间的链接可以显示哪个作者写了哪本书。

但如果你的数据不适合这种结构化格式怎么办？如果你处理的是更灵活或不可预测的事物，比如社交媒体上的帖子，其中一些帖子有图片，一些有视频，而另一些只有文本呢？这就是非关系型，或 NoSQL，数据库发挥作用的地方。它们被设计来处理各种数据结构，从简单的键值对到更复杂的文档或图。这使得它们成为现代应用程序的理想选择，这些应用程序需要灵活性和可扩展性。

重要提示

本章将专注于在 Node.js 中与这些数据库交互。因此，假设您对数据库和 **结构化查询语言**（**SQL**）有一些基本知识。

我们将从设置一个简单的 SQL 数据库开始，以了解数据库操作的基本原理。然后，我们将探索 NoSQL 数据库的动态世界，学习如何与它们交互以处理更灵活的数据结构。到本章结束时，你将在 Node.js 应用程序中使用不同类型的数据库方面打下基础，这将为你提供选择适合你项目的正确存储解决方案的灵活性。

本章将涵盖以下食谱：

+   连接到并持久化到 MySQL 数据库

+   连接到并持久化到 PostgreSQL 数据库

+   连接到并持久化到 MongoDB

+   使用 Redis 持久化数据

+   探索 GraphQL

# 技术要求

在本章中，我们将使用 Docker 在容器中提供数据库。在构建可扩展和弹性架构时，使用数据库容器是常见的做法——尤其是在使用容器编排器如 Kubernetes 时。

然而，我们将在本章中使用 Docker 容器的主要原因是为了避免手动在我们的系统上安装每个数据库的 **命令行界面**（**CLIs**）和服务器。在本章中，我们将使用 Docker 来提供容器化的 MySQL、PostgreSQL、MongoDB 和 Redis 数据存储。

建议从 [`docs.docker.com/engine/install/`](https://docs.docker.com/engine/install/) 安装 Docker Desktop。

如果你无法安装 Docker，你仍然可以完成这些食谱，但你将需要手动为每个食谱安装特定的数据库或连接到远程数据库服务。

注意，本章不会介绍如何从 Docker 容器中启用持久数据存储，因为这需要超出 Node.js 教程范围的 Docker 知识。因此，一旦容器被销毁或删除，教程期间积累的数据将会丢失。

完成每个配方后，也可以通过以下步骤清理并删除你的数据库容器：

1.  在你的终端中输入 **$ docker ps** 以列出你的 Docker 容器。

1.  从那里，找到容器标识符，并将其传递给 **$ docker stop <ContainerID>** 命令以停止容器。

1.  使用 **$ docker rm --force <ContainerID>** 来删除容器。

或者，你可以使用以下命令来删除所有 Docker 容器：

```js
$ docker rm --force $(docker ps --all --quiet)
```

如果你的设备上运行着与本书配方无关的其他 Docker 容器，使用此命令时要小心。

重要提示

Docker 既指代虚拟化技术，也指代创建了这项技术的公司 Docker Inc.。Docker 允许你将应用程序和服务打包成名为容器的包。有关 Docker 技术的更详细信息，请参阅*第十一章*。

在几个配方中，我们还将使用**dotenv**模块（[`www.npmjs.com/package/dotenv`](https://www.npmjs.com/package/dotenv)）。**dotenv**模块将环境变量从**.env**文件加载到 Node.js 进程中。在需要的情况下，我们将把示例数据库凭据存储在**.env**文件中，然后使用**dotenv**模块将这些解析到我们的 Node.js 进程中。

你还需要安装 Node.js，最好是最新版本，Node.js 22，以及你选择的编辑器和浏览器。本章生成的代码示例可在 GitHub 上的[`github.com/PacktPublishing/Node.js-Cookbook-Fifth-Edition`](https://github.com/PacktPublishing/Node.js-Cookbook-Fifth-Edition)的**Chapter07**目录中找到。

# 连接到 MySQL 数据库并持久化

SQL 是用于与关系型数据库通信的标准。MySQL ([`www.mysql.com/`](https://www.mysql.com/)) 和 PostgreSQL ([`www.postgresql.org/`](https://www.postgresql.org/)) 都是流行的开源**关系型数据库管理系统**（**RDBMSs**）。SQL 数据库有许多实现，每个都有其扩展和专有功能。然而，所有这些 SQL 数据库都实现了存储、更新和查询数据的基本命令集。

在这个配方中，我们将使用**mysql2**模块（[`www.npmjs.com/package/mysql2`](https://www.npmjs.com/package/mysql2)）从 Node.js 与 MySQL 数据库进行通信。

## 准备工作

首先，我们需要在本地运行一个 MySQL 数据库。为此，以及在本章中的其他数据库，尽可能使用 Docker。MySQL 在 Docker Hub 上提供了一个官方镜像（[`hub.docker.com/_/mysql`](https://hub.docker.com/_/mysql)）。本食谱假设您对 SQL 和关系型数据库有一些，但最少量的先验知识。

重要提示

在本教程中，我们将使用来自 **npm** 的 **mysql2** 包来与 Node.js 中的 MySQL 数据库交互，因为它与最新的 MySQL 功能兼容，并支持承诺。选择 **mysql2** 而不是之前使用的 **mysql** 包的原因是它更更新，使我们能够利用新的功能和功能，如 **Promise** 和 **async** / **await** 语法。

要使用 Docker 设置 MySQL 数据库并准备您的项目，请按照以下步骤操作：

1.  在终端窗口中，输入以下命令以启动一个监听在端口 **3306** 的 MySQL 数据库：

    ```js
    $ docker run --publish 3306:3306 --name node-mysql --env MYSQL_ROOT_PASSWORD=PASSWORD --detach mysql:8
    ```

重要提示

Docker 命令中的 **--publish 3306:3306** 选项将主机机器上的端口 **3306** 映射到 Docker 容器上的端口 **3306**，允许外部访问在该端口上运行的服务器。

如果您没有本地镜像，那么 Docker 将首先从 Docker Hub 下载镜像。当 Docker 正在下载镜像时，您可能会看到如下输出：

```js
Unable to find image 'mysql:8' locally
latest: Pulling from library/mysql
ea4e27ae0b4c: Pull complete
837904302482: Pull complete
3c574b61b241: Pull complete
654fc4f3eb2d: Pull complete
32da9c2187e3: Pull complete
dc99c3c88bd6: Pull complete
970181cc0aa6: Pull complete
d77b716c39d5: Pull complete
9e650d7f9f83: Pull complete
acc21ff36b4b: Pull complete
Digest: sha256:ff5ab9cdce0b4c59704b4e2a09deed5ab8467be795e0ea20228b8528f53fcf82
Status: Downloaded newer image for mysql:8
dbb88d7d042966351a79ae159eb73129d69961b2c3dab943d9f4cdd6697d5220
```

**--detach** 参数表示我们希望以分离模式启动容器——这意味着容器将在后台运行。省略 **--detach** 参数将意味着您的终端窗口将被容器占用。

1.  接下来，我们将为这个食谱创建一个新的目录：

    ```js
    $ mkdir mysql-app
    $ cd mysql-app
    ```

1.  由于我们将要安装来自 **npm** 的模块，因此我们还需要初始化我们的项目：

    ```js
    $ npm init --yes
    ```

    我们还将准备两个用于食谱的文件。第一个将是一个名为 **setupDb.mjs** 的脚本，用于创建数据库；第二个将是一个名为 **task.mjs** 的脚本，用于向数据库添加新任务。在此期间，让我们也创建一个 **.env** 文件，以便存储我们的数据库凭据：

    ```js
    $ touch setupDb.mjs tasks.mjs
    $ touch .env
    ```

1.  将我们 MySQL 实例的示例凭据添加到 **.env** 文件中：

    ```js
    DB_MYSQL_USER=root
    DB_MYSQL_PASSWORD=PASSWORD
    ```

    请注意，这些示例凭据是为了简化而设置的——您应该在您的应用程序中使用更强的凭据。此外，请确保不要意外地将 **.env** 文件提交到 **版本控制系统**（**VCSs**，如 Git），因为这可能导致敏感凭据泄露。

现在我们已经启动了 MySQL 数据库并初始化了项目，我们就可以继续进行食谱了。

## 如何操作...

在本食谱中，我们将关注如何从 **npm** 安装 **mysql2** 模块，连接到 MySQL 数据库，并执行基本的 SQL 查询。我们将使用一个简单的任务列表示例来说明这些概念。我们还将使用 **ECMAScript 模块**（**ESM**）语法，这在 *第五章* 的 *使用 ECMAScript 模块* 食谱中有介绍。

这种方法应该有助于您了解在 MySQL 数据库中管理和管理数据的实际应用。

1.  首先，我们需要安装**dotenv**模块，用于解析环境变量配置，以及**mysql2**模块：

    ```js
    $ npm install dotenv mysql2
    ```

1.  我们将首先编写一个脚本来设置我们的任务列表数据库。为此，我们首先需要使用**dotenv**模块导入和加载我们的凭据，并导入**mysql2**模块。将以下内容添加到**setupDb.mjs**以执行此操作：

    ```js
    import dotenv from 'dotenv';
    import mysql from 'mysql2/promise';
    dotenv.config();
    ```

1.  现在，让我们构建一个**main()**函数。我们将随着教程步骤的进行向此函数中添加逻辑。将以下内容添加到**setupDb.mjs**：

    ```js
    async function main () {
    }
    main().catch(console.error);
    ```

1.  现在，让我们开始添加我们的连接逻辑，我们将将其包装在**try** / **catch** / **finally**结构中，其中**finally**将关闭数据库连接。在**main()**函数中，添加以下内容：

    ```js
    async function main() {
      let connection;
      try {
        connection = await mysql.createConnection({
          user: process.env.DB_MYSQL_USER,
          password: process.env.DB_MYSQL_PASSWORD,
        });
        console.log('Connected as id ' +
            connection.threadId);
       } catch (error) {
        console.error('Error connecting: ' + error.stack);
      } finally {
        if (connection) await connection.end();
      }
    }
    ```

    我们可以在我们的终端中运行此文件以测试连接：

    ```js
    $ node setupDb.mjs
    Connected as id 10
    ```

1.  现在，让我们添加我们的逻辑来创建表。为此，我们将使用两个独立的 SQL 语句。第一个将创建一个数据库并指示连接使用它。第二个将创建一个**tasks**数据库表。在**main()**函数中，在**console.log('Connected as ...**行下方添加以下内容：

    ```js
    await connection.query('CREATE DATABASE IF NOT EXISTS
      tasks');
    console.log('Database created or already exists.');
    await connection.query('USE tasks');
    const createTasksTableSql =
      `CREATE TABLE IF NOT EXISTS tasks (
        id INT AUTO_INCREMENT PRIMARY KEY,
        task VARCHAR(255) NOT NULL,
        completed BOOLEAN NOT NULL DEFAULT FALSE
        )`;
    await connection.query(createTasksTableSql);
    console.log('Tasks table created or already exists.');
    ```

1.  使用以下命令在您的终端中运行程序：

    ```js
    $ node setupDb.mjs
    Connected as id 18
    Database created or already exists.
    Tasks table created or already exists.
    ```

1.  现在，我们可以在**tasks.mjs**中实现我们的逻辑，将一些数据输入到我们的表中，同样通过 SQL 查询。首先，复制我们在**setupDb.mjs**中使用的相同连接逻辑：

    ```js
    import dotenv from 'dotenv';
    import mysql from 'mysql2/promise';
    dotenv.config();
    async function main() {
      let connection;
      try {
        connection = await mysql.createConnection({
          user: process.env.DB_MYSQL_USER,
          password: process.env.DB_MYSQL_PASSWORD,
        });
        console.log('Connected as id ' +
            connection.threadId);
      } catch (error) {
        console.error('Error connecting: ' + error.stack);
      } finally {
        if (connection) await connection.end();
      }
    }
    main().catch(console.error);
    ```

    注意，我们使用**connection.end()**结束与 MySQL 数据库的连接。

1.  现在，我们可以添加一些逻辑来从命令行接收任务详情。在**console.log('Connected** **as** …**行下方添加以下逻辑：

    ```js
        if (process.argv[2]) {
          await connection.query(
              `INSERT INTO tasks.tasks (task) VALUES
                  (?);`,
              [process.argv[2]]
          );
        }
    ```

1.  让我们添加一个查询，以获取**tasks**表的内容：

    ```js
    const [results] = await connection.query('SELECT *
      FROM tasks.tasks;');
        console.log(results);
    ```

1.  现在，使用以下命令运行程序：

    ```js
    $ node tasks.mjs "Walk the dog."
    Connected as id 10
    [ { id: 1, task: 'Walk the dog.', completed: 0 } ]
    ```

    每次我们运行程序时，我们的插入查询都会被执行，这意味着在**tasks**表中将创建一个新的条目。

## 它是如何工作的…

从**mysql2**模块暴露的**createConnection()**方法根据传递给方法的配置和凭据建立与 MySQL 服务器的连接。在配方中，我们使用环境变量将数据库的用户名和密码传递给**createConnection()**方法。**mysql2**模块默认在**localhost:3306**查找 MySQL 数据库，这是我们创建的 MySQL Docker 容器在配方中“准备就绪”部分暴露的位置。**npm**中的**mysql2**模块旨在提供与先前的**npm**模块中的**mysql**模块等效的功能。可以在**mysql**模块 API 文档中找到传递给**createConnection()**方法的完整选项列表，网址为[`github.com/mysqljs/mysql#connection-options`](https://github.com/mysqljs/mysql#connection-options)。

重要提示

连接池也可以被利用，通过重用现有连接而不是使用后关闭它们来最小化连接到 MySQL 服务器所需的时间。这种方法通过消除设置新连接相关的开销来提高查询延迟。这种策略对于大规模应用的开发至关重要。更多详情，请参阅[`sidorares.github.io/node-mysql2/docs#using-connection-pools`](https://sidorares.github.io/node-mysql2/docs#using-connection-pools)的 API 文档。

在整个食谱中，我们使用了**query()**方法将 SQL 语句发送到 MySQL 数据库。**setupDb.mjs**文件中的 SQL 语句创建了一个**tasks**数据库和一个**tasks**表。**task.mjs**文件包含了将单个任务插入到**tasks**表的 SQL 语句。我们使用**query()**方法发送到数据库的最后一个 SQL 语句是一个**SELECT**语句，它返回了**tasks**表的内容。

每个 SQL 语句都会排队并异步执行。可以将回调函数作为参数传递给**query()**方法，但我们利用了**async** / **await**语法。

如其名所示，**end()**方法结束与数据库的连接。**end()**方法确保在结束连接之前没有查询仍在排队或正在处理。还有一个方法，**destroy()**，会立即终止与数据库的连接，忽略任何挂起或正在执行的查询的状态。

需要意识到的一种常见的针对面向用户的 Web 应用的攻击类型是 SQL 注入攻击。

SQL 注入是指攻击者向您的数据库发送恶意 SQL 语句。这通常是通过将恶意 SQL 语句插入到网页输入字段中实现的。这不是 Node.js 特有的问题；它也适用于其他编程语言，其中 SQL 查询是通过字符串连接创建的。减轻这些攻击的方法是对用户输入进行清理或转义，以确保我们的 SQL 语句不能被恶意操纵。

您可以通过使用**connection.escape()**手动转义用户提供的直接数据。然而，在食谱中，我们使用占位符（**?**）语法在我们的 SQL 查询中实现相同的效果：

```js
await connection.query(
          `INSERT INTO tasks.tasks (task) VALUES (?);`,
          [process.argv[2]]
      );
```

如果我们将输入值通过**query**函数的第二个参数传递给查询，**mysql2**模块会为我们处理用户输入的清理。在 SQL 查询中，多个占位符（**?**）按照提供的顺序映射到值。

## 还有更多...

在本节中，我们将基于使用 Node.js 与 MySQL 交互的基础，介绍如何结合 MySQL 使用 Fastify 创建 REST API。我们将逐步讲解设置项目、启动 Fastify 服务器、使用**@fastify/mysql**插件（[`www.npmjs.com/package/@fastify/mysql`](https://www.npmjs.com/package/@fastify/mysql)）连接 MySQL 以及创建处理**创建、读取、更新、删除**（**CRUD**）操作的路线。

确保您有一个 MySQL 数据库可用。为此，我们将重用主配方步骤中创建的数据库。

1.  首先，我们将为**fastify-mysql**项目创建一个新的目录，并使用**npm**初始化它：

    ```js
    $ mkdir fastify-mysql
    $ cd fastify-mysql
    $ npm init --yes
    ```

1.  使用**npm**安装**fastify**和**@fastify/mysql**插件：

    ```js
    $ npm install fastify @fastify/mysql
    ```

1.  在您的项目根目录下创建一个名为**server.js**的文件。此文件将配置 Fastify 服务器，连接到 MySQL 数据库，并定义路由：

    ```js
    $ touch server.js
    ```

1.  首先引入 Fastify – 我们也将启用日志记录：

    ```js
    const fastify = require('fastify')({ logger: true });
    ```

1.  现在，我们可以注册之前安装的**@fastify/mysql**插件：

    ```js
    fastify.register(require('@fastify/mysql'), {
      connectionString:
        'mysql://root:PASSWORD@localhost/tasks'
    });
    ```

    注意，连接字符串包含我们 MySQL 数据库的凭据 – 理想情况下，此连接字符串应存储在之前配方中提到的**.env**文件中。

1.  现在，让我们注册一个路由以返回数据库中的所有任务：

    ```js
    fastify.get('/tasks', (req, reply) => {
      fastify.mysql.query(
        'SELECT * FROM tasks.tasks',
        function onResult (err, result) {
          reply.send(err || result);
        }
      );
    });
    ```

1.  最后，我们将添加运行服务器的逻辑：

    ```js
    fastify.listen({ port: 3000 }, err => {
      if (err) throw err;
      console.log(`server listening on
        ${fastify.server.address().port}`);
    });
    ```

1.  让我们启动 Fastify MySQL 应用程序：

    ```js
    $ node server.js
    ```

1.  要测试您的 API，在服务器运行时打开一个新的终端窗口，并使用**curl**：

    ```js
    $ curl http://localhost:3000/tasks
    ```

本教程提供了使用 Fastify 和 MySQL 创建 REST API 的基本介绍，涵盖了项目设置、初始化服务器、连接到数据库以及从数据库检索项目。Fastify 为在此配方中使用的其他数据库提供了等效的插件。

## 参见

+   本章中的*连接并持久化到 PostgreSQL 数据库*配方

+   本章中的*连接并持久化到 MongoDB*配方

+   本章中的*使用 Redis 持久化数据*配方

+   *第九章*

# 连接到并持久化到 PostgreSQL 数据库

PostgreSQL 首次于 1996 年推出，是一个强大的开源对象关系型数据库系统，由于其可靠性、功能健壮性和性能而经受住了时间的考验。PostgreSQL 的突出特点之一是它既可以作为传统的数据库使用，其中数据存储在具有相互关系的表中，也可以作为文档数据库使用，例如 NoSQL 数据库，其中数据可以以 JSON 格式存储。这种灵活性允许开发者根据其应用程序的需求选择最合适的数据存储模型。

在整个教程中，我们将探索从 Node.js 应用程序与 PostgreSQL 数据库交互的基础。我们将使用**pg**模块，这是一个流行的、功能全面的 PostgreSQL 客户端，适用于 Node.js。**pg**模块简化了连接到并执行针对 PostgreSQL 数据库的查询。

## 准备工作

要开始，我们需要一个 PostgreSQL 服务器来连接。我们将使用 Docker 来配置一个容器化的 PostgreSQL 数据库。有关使用 Docker 配置数据库的更多信息，请参阅本章的 *技术要求* 部分。

我们将使用 Docker 官方的 PostgreSQL 镜像，[`hub.docker.com/_/postgres`](https://hub.docker.com/_/postgres)。

以下步骤将初始化我们的 PostgreSQL 服务器并准备我们的项目目录：

1.  在终端窗口中，键入以下命令以配置一个 **postgres** 容器：

    ```js
    $ docker run --publish 5432:5432 --name node-postgres-latest --env POSTGRES_PASSWORD=PASSWORD --detach postgres:16
    ```

    假设您没有本地 PostgreSQL 镜像的副本，在 Docker 下载镜像时，您可能会看到以下输出：

    ```js
    Unable to find image 'postgres:16' locally
    latest: Pulling from library/postgres
    f546e941f15b: Pull complete
    926c64b890ad: Pull complete
    eca757527cc4: Pull complete
    93d9b27ec7dc: Pull complete
    86e78387c4e9: Pull complete
    8776625edd8f: Pull complete
    d1afcbffdf18: Pull complete
    6a6c8f936428: Pull complete
    ae47f32f8312: Pull complete
    82fb85897d06: Pull complete
    ce4a61041646: Pull complete
    ca83cd3ae7cf: Pull complete
    f7fbf31fd41d: Pull complete
    353df72b8bf7: Pull complete
    Digest: sha256:f58300ac8d393b2e3b09d36ea12d7d24ee9440440e421472a300e929ddb63460
    Status: Downloaded newer image for postgres:16
    86ce1ac06849f737e669c34e50e6f91383074cdecb1a18f8f23a6becaa085ba0
    ```

    我们现在应该有一个在端口 **5432** 上监听的 PostgreSQL 数据库。

1.  接下来，我们将设置一个目录和文件，为我们的 PostgreSQL 应用程序做好准备：

    ```js
    $ mkdir postgres-app
    $ cd postgres-app
    $ touch tasks.js .env
    ```

1.  由于我们将使用第三方模块，我们还需要使用 **npm** 来初始化一个项目。让我们只接受默认设置：

    ```js
    $ npm init --yes
    ```

现在，我们已经准备好进入菜谱，我们将使用 **pg** 模块与我们的 PostgreSQL 数据库进行交互。

## 如何操作…

在这个菜谱中，我们将安装 **pg** 模块，使用 Node.js 与我们的 PostgreSQL 数据库进行交互。我们还将向数据库发送一些简单的查询。

1.  首先，我们需要安装第三方 **pg** 模块：

    ```js
    $ npm install pg
    ```

1.  在这个菜谱中，我们还将使用 **dotenv** 模块；使用以下命令安装它：

    ```js
    $ npm install dotenv
    ```

1.  我们还将使用 **.env** 文件来存储我们的 PostgreSQL 数据库凭据，并使用 **dotenv** 模块将它们传递给我们的程序。将以下凭据添加到 **.env** 文件中：

    ```js
    PGUSER=postgres
    PGPASSWORD=PASSWORD
    PGPORT=5432
    ```

1.  打开 **tasks.js** 并使用 **dotenv** 模块导入我们的环境变量：

    ```js
    require('dotenv').config();
    ```

1.  接下来，在 **tasks.js** 中，我们需要导入 **pg** 模块并创建一个 PostgreSQL 客户端：

    ```js
    const pg = require('pg');
    const db = new pg.Client();
    ```

1.  现在，让我们允许我们的程序通过命令行参数处理输入：

    ```js
    const task = process.argv[2];
    ```

1.  接下来，我们将定义我们将要使用的 SQL 查询作为常量。这将提高我们代码的可读性：

    ```js
    const CREATE_TABLE_SQL = `CREATE TABLE IF NOT EXISTS
      tasks (id SERIAL, task TEXT NOT NULL, PRIMARY KEY (
        id ));`;
    const INSERT_TASK_SQL = 'INSERT INTO tasks (task)
      VALUES ($1);';
    const GET_TASKS_SQL = 'SELECT * FROM tasks;';
    ```

    **SELECT * FROM tasks;** SQL 查询返回 **tasks** 表中的所有任务。

1.  接下来，我们将添加以下代码来连接到我们的数据库。如果尚不存在，则创建一个 **tasks** 表，插入一个任务，最后列出数据库中存储的所有任务：

    ```js
    db.connect((err) => {
      if (err) throw err;
      db.query(CREATE_TABLE_SQL, (err) => {
        if (err) throw err;
        if (task) {
          db.query(INSERT_TASK_SQL, [task], (err) => {
            if (err) throw err;
            listTasks();
          });
        } else {
          listTasks();
        }
      });
    });
    ```

1.  最后，我们将创建我们的 **listTasks()** 函数，该函数将使用 **GET_TASKS_SQL** 。此函数还将结束与数据库的连接：

    ```js
    function listTasks () {
      db.query(GET_TASKS_SQL, (err, results) => {
        if (err) throw err;
        console.log(results.rows);
        db.end();
      });
    }
    ```

1.  运行 **tasks.js**，传递一个任务作为命令行参数。任务将被插入到数据库中，并在程序结束前列出：

    ```js
    $ node tasks.js "Bath the dog."
    [
      { id: 1, task: 'Bath the dog.' }
    ]
    ```

1.  我们也可以不传递任务来运行程序。当我们不带 **task** 参数运行 **tasks.js** 时，程序将输出数据库中存储的任务：

    ```js
    $ node tasks.js
    [
      { id: 1, task: 'Bath the dog.' }
    ]
    ```

通过遵循这些步骤，您已经了解了如何将 PostgreSQL 与 Node.js 集成。

## 工作原理…

在本菜谱的 *准备就绪* 部分，我们使用 Docker Hub 上的 Docker 官方镜像配置了一个容器化的 PostgreSQL 数据库。配置的 PostgreSQL 数据库是在名为 **node-postgres** 的 Docker 容器中配置的。默认情况下，PostgreSQL Docker 镜像创建了一个名为 **postgres** 的用户和数据库。我们用于配置数据库的 Docker 命令指示容器在 **localhost:5432** 上使 PostgreSQL 数据库可用，并使用占位符密码 **PASSWORD**。

连接到我们的 PostgreSQL 数据库所需配置信息已在 **.env** 文件中指定。我们使用 **dotenv** 模块将此配置信息作为环境变量加载到我们的 Node.js 进程中。

注意，我们不必直接将任何环境变量传递给客户端。这是因为 **pg** 模块会自动查找特定命名的变量（**PGHOST**、**PGPORT** 和 **PGUSER**）。然而，如果我们想，我们可以在创建客户端时指定这些值，如下所示：

```js
const client = new Client({
  host: 'localhost',
  port: 5432,
  user: 'postgres'
});
```

我们使用 **connect()** 方法连接到我们的 PostgreSQL 数据库。我们向此方法提供一个回调函数，以便在连接尝试完成后执行。我们在回调函数中添加了错误处理，以便如果连接尝试失败，则抛出错误。

在程序的剩余部分，我们使用 **pg** 模块提供的 **query()** 方法对 PostgreSQL 数据库执行 SQL 查询。我们对 **query()** 方法的每次调用都提供了一个回调函数，以便在查询完成后执行。

## 还有更多...

除了存储传统的关联数据，PostgreSQL 还提供了存储对象数据的能力。这使得可以在文档存储的同时存储关联数据。

我们可以将我们在 *连接和持久化到 PostgreSQL 数据库* 菜谱中创建的程序进行修改，以处理关联数据和对象数据。

1.  将 **postgres-app** 目录复制到名为 **postgres-object-app** 的目录中：

    ```js
    $ cp -r postgres-app postgres-object-app
    $ cd postgres-object-app
    ```

1.  现在，我们将编辑我们的 SQL 查询以创建一个名为 **task_docs** 的新表，用于存储文档数据。在 **tasks.js** 文件中将您的 SQL 查询常量更改为以下内容：

    ```js
    const CREATE_TABLE_SQL = `CREATE TABLE IF NOT EXISTS
      task_docs (id SERIAL, doc jsonb);`;
    const INSERT_TASK_SQL = `INSERT INTO task_docs (doc)
      VALUES ($1);`;
    const GET_TASKS_SQL = `SELECT * FROM task_docs;`;
    ```

1.  现在，当我们运行应用程序时，我们可以传递 JSON 输入来表示任务。请注意，我们需要将 JSON 输入用单引号括起来，然后为键值对使用双引号：

    ```js
    $ node tasks.js '{"task":"Walk the dog."}'
    [ { id: 1, doc: { task: 'Walk the dog.' } } ]
    ```

    **doc** 字段是用 **jsonb** 类型创建的，它表示 JSON 二进制类型。PostgreSQL 提供了两种 JSON 数据类型：**json** 和 **jsonb**。**json** 数据类型类似于常规的文本输入字段，但增加了验证 JSON 的功能。**jsonb** 类型是结构化的，便于在文档对象中进行查询和索引。当您需要查询或索引数据时，您会选择 **jsonb** 数据类型而不是 **json** 数据类型。

根据这个示例，一个 **jsonb** 查询看起来如下所示：

```js
SELECT * FROM task_docs WHERE doc ->> task= "Bath the dog."
```

注意，我们可以在文档对象内的 **task** 属性上进行查询。有关 **jsonb** 数据类型的信息，请参阅官方 PostgreSQL 文档，网址为 [`www.postgresql.org/docs/9.4/datatype-json.html`](https://www.postgresql.org/docs/9.4/datatype-json.html)。

## 参见

+   本章中的 *连接和持久化到 MySQL 数据库* 菜谱

+   本章中的 *连接和持久化到 MongoDB* 菜谱

+   本章中的 *使用 Redis 持久化数据* 菜谱

# 连接和持久化到 MongoDB

MongoDB 是一个围绕文档模型构建的 NoSQL 数据库管理系统。数据存储在灵活的、类似 JSON 的文档中，称为 **Binary JSON** ( **BSON** )，这些文档组织成 **集合**，类似于关系数据库中的表。集合中的每个文档都可以有不同的结构，允许动态模式并易于修改数据模型。

MongoDB 支持使用其查询语言进行强大的查询功能，包括用于过滤、排序和操作数据的各种运算符和方法。

本菜谱将使用 MongoDB Node.js 驱动程序直接使用 book/author 示例。我们将编写函数来创建和查找 MongoDB 数据库中的作者和书籍。此脚本将展示基本的 CRUD 操作，不使用任何 Web 框架，纯粹关注数据库交互。

## 准备工作

使用 Docker 设置 MongoDB 数据库并准备好项目目录以供应用程序使用，请按照以下步骤操作：

1.  与本章中的其他数据库一样，我们将使用 Docker 通过 MongoDB Docker 镜像提供 MongoDB 数据库，该镜像可在 [`hub.docker.com/_/mongo`](https://hub.docker.com/_/mongo) 找到：

    ```js
    $ docker run --publish 27017:27017 --name node-mongo --detach mongo:7
    ```

    假设您没有 MongoDB 镜像的本地副本，在 Docker 下载镜像时，您可能会看到以下输出：

    ```js
    Unable to find image 'mongo:7' locally
    latest: Pulling from library/mongo
    bccd10f490ab: Pull complete
    b00c7ff578b0: Pull complete
    a1f43ab85151: Pull complete
    9e72f6a5998a: Pull complete
    8424336879e4: Pull complete
    85a6d3c2e6c8: Pull complete
    c533c21e5fb8: Pull complete
    1fddf702bb73: Pull complete
    Digest: sha256:0e145625e78b94224d16222ff2609c4621ff6e2c390300e4e6bf698305596792
    Status: Downloaded newer image for mongo:7
    9230ee867d2b2272448f2596ddc19a7f4de5112c99e4dd31b2d7746b28fbc674
    ```

1.  我们还将为 MongoDB Node.js 应用程序创建一个目录：

    ```js
    $ mkdir mongodb-app
    $ cd mongodb-app
    ```

1.  在这个菜谱中，我们需要从 **npm** 注册表中安装模块，因此我们需要使用 **$ npm init** 初始化我们的项目：

    ```js
    $ npm init --yes
    ```

1.  创建一个名为 **index.js** 的文件；这将包含与 MongoDB 交互的应用程序代码：

    ```js
    $ touch index.js
    ```

现在我们已经启动了数据库并初始化了项目，我们可以继续进行菜谱。

## 如何做到这一点...

在这个菜谱中，我们将使用 **mongodb** 模块来演示我们如何与我们的 MongoDB 数据库进行交互。

1.  首先，安装 **mongodb** 模块：

    ```js
    $ npm install mongodb
    ```

1.  首先，我们将向 **index.js** 文件添加逻辑以连接到我们的 MongoDB 数据库：

    ```js
    const { MongoClient } = require('mongodb');
    const URI = 'mongodb://localhost:27017';
    const client = new MongoClient(URI);
    async function connectToMongoDB () {
      try {
        await client.connect();
        console.log('Connected successfully to server');
        return client.db('Library');
      } catch (err) {
        console.error('Connection to MongoDB failed:',
          err);
      }
    }
    ```

1.  接下来，我们将创建一个函数将作者插入到 **authors** 集合中：

    ```js
    async function createAuthor (db, author) {
      try {
        const result = await
          db.collection('authors').insertOne(author);
        console.log(`Author created with the following id:
          ${result.insertedId}`);
        return result.insertedId;
      } catch (err) {
        console.error('Create author failed:', err);
      }
    }
    ```

1.  创建一个函数将书籍插入到 **books** 集合中：

    ```js
    async function createBook (db, book) {
      try {
        const result = await
          db.collection('books').insertOne(book);
        console.log(`Book created with the following id:
          ${result.insertedId}`);
        return result.insertedId;
      } catch (err) {
        console.error('Create book failed:', err);
      }
    }
    ```

1.  创建一个函数以查找 **authors** 集合中的所有作者：

    ```js
    async function findAllAuthors (db) {
      try {
        const authors = await
          db.collection('authors').find().toArray();
        console.log('Authors:', authors);
        return authors;
      } catch (err) {
        console.error('Find all authors failed:', err);
      }
    }
    ```

1.  创建一个函数以查找所有书籍并使用聚合管道填充作者详细信息：

    ```js
    async function findAllBooksWithAuthors (db) {
      try {
        const books = await
          db.collection('books').aggregate([
          {
            $lookup: {
              from: 'authors',
              localField: 'authorId',
              foreignField: '_id',
              as: 'authorDetails'
            }
          }
        ]).toArray();
        console.log('Books with author details:', books);
        return books;
      } catch (err) {
        console.error('Find all books with authors
          failed:', err);
      }
    }
    ```

1.  最后，我们将在 **main()** 函数中使用 **createAuthor()**、**createBook()**、**findAllAuthors()** 和 **findAllBooksWithAuthors()** 函数按顺序执行操作：

    ```js
    async function main () {
      const db = await connectToMongoDB();
      if (!db) return;
      const authorId = await createAuthor(db, { name:
        'Richard Adams' });
      if (!authorId) return;
      await createBook(db, { title: 'Watership Down',
        authorId });
      await findAllAuthors(db);
      await findAllBooksWithAuthors(db);
      client.close();
    }
    main().catch(console.error);
    ```

1.  运行脚本：

    ```js
    $ node index.js
    ```

在这个配方中，我们构建了一个 Node.js 脚本，它作为与 MongoDB 数据库交互的功能接口。

## 它是如何工作的…

在这个配方中，我们首先导入必要的模块，特别是来自 **npm** 的 **mongodb** 模块的 **MongoClient** 类。设置 MongoDB 连接涉及定义一个连接到本地 MongoDB 服务器的 URI，并使用此 URI 初始化一个 **MongoClient** 实例。在我们的案例中，我们的数据库托管在 MongoDB 的典型默认主机和端口上：**mongodb://localhost:27017**。

注意，当使用 Docker 时，MongoDB 默认不启用身份验证，因此在连接字符串中不需要任何身份验证参数。

**connectToMongoDB()** 函数异步尝试连接到 MongoDB 服务器，相应地记录成功或失败消息，并在成功时返回指定数据库的引用。

**npm** 的 **mongodb** 模块公开了大量的 CRUD 方法，以与 MongoDB 数据库中的 MongoDB 集合进行交互。术语 *CRUD* 用于表示持久存储的基本功能。在这个配方中，我们使用了 **find()** 和 **insertOne()** CRUD 方法。可用方法的完整列表定义在 Node.js MongoDB 驱动程序 API 文档中（[`mongodb.github.io/node-mongodb-native/6.5/`](https://mongodb.github.io/node-mongodb-native/6.5/)）。

我们还在 **findAllBooksWithAuthors()** 函数中使用了 **aggregate()** 方法。聚合管道可以包含一个或多个阶段，以创建一个处理、转换并返回结果的流程。

**main()** 函数协调执行流程，首先连接到 MongoDB 数据库。连接成功后，它继续为 **理查德·亚当斯** 创建一个作者文档，以及一个标题为 **《风车山》** 的相应书籍文档，并将它们关联起来。随后，它使用定义的函数检索所有作者和书籍及其关联的作者详细信息。在整个脚本中，使用 **try** / **catch** 块实现错误处理，以处理执行过程中可能出现的任何潜在错误。最后，脚本通过关闭 MongoDB 客户端连接来结束。

总体而言，这个脚本作为一个实用的示例，展示了如何利用 Node.js 和 **mongodb** 包在 MongoDB 数据库上执行 CRUD 操作，演示了基本功能，如连接到数据库、插入文档、查询集合和有效处理错误。

## 参见

+   本章中 *连接并持久化到 MySQL 数据库* 的配方

+   本章中 *连接并持久化到 PostgreSQL 数据库* 的配方

+   本章中 *使用 Redis 持久化数据* 的配方

# 使用 Redis 持久化数据

Redis 是一个开源的内存键值数据存储。在正确的设置下，Redis 可以成为一个高性能的数据存储。它通常用于提供应用程序的缓存，但也可以用作数据库。

**Redis**，即 **远程字典服务器** 的缩写，是一个内存数据结构存储，通常用作数据库、缓存和消息代理。它在需要高速和效率的场景中表现出色，如缓存、会话管理、实时分析和消息队列。Redis 支持各种数据结构的能力，结合其原子操作和 **发布/订阅**（**pub/sub**）消息能力，使其成为增强 Node.js 应用程序性能和可扩展性的强大工具。其内存特性确保了数据的快速访问，与传统基于磁盘的数据库相比，显著降低了延迟，使其适用于速度至关重要的应用程序。

在 Node.js 的上下文中，Redis 特别适用于管理 Web 应用程序中的会话数据，实现快速数据检索并提高用户体验。它也被广泛用于实现缓存机制，减轻数据库负载并加快响应时间。此外，其 pub/sub 消息系统通过允许客户端和服务器之间的高效通信，促进了实时应用程序（如聊天应用程序或实时通知）的开发。无论您是想优化应用程序的性能、高效扩展还是构建功能丰富的实时交互，将 Redis 与 Node.js 集成都提供了一个强大的解决方案来满足这些需求。

## 准备工作

在深入研究 Redis 模块集成之前，重要的是要注意我们将使用 ESM 以确保兼容性。有关模块的更多信息，请参阅 *第五章* 。

1.  与本章中之前提到的数据库一样，我们将使用 Docker 来部署 Redis 数据库，基于可在 [`hub.docker.com/_/redis`](https://hub.docker.com/_/redis) 找到的 Docker 镜像。运行以下命令：

    ```js
    $ docker run --publish 6379:6379 --name node-redis --detach redis
    ```

    默认情况下，容器化的 Redis 数据库将在 **localhost:6379** 上可用。

1.  我们还将创建一个名为 **redis-app** 的新文件夹，其中包含一个名为 **tasks.mjs** 的文件：

    ```js
    $ mkdir redis-app
    $ cd redis-app
    $ touch tasks.mjs
    ```

1.  在这个菜谱中，我们将使用第三方 **npm** 模块；因此，我们需要初始化我们的项目：

    ```js
    $ npm init --yes
    ```

现在我们已经启动了 Redis 并设置了我们的项目，我们可以继续进行下一步了。

## 如何操作…

在这个菜谱中，我们将使用 **redis** 模块与我们的 Redis 数据存储进行交互。

1.  首先安装第三方 **redis** 模块：

    ```js
    $ npm install redis
    ```

1.  现在，我们需要在 **tasks.mjs** 中导入并创建一个 Redis 客户端：

    ```js
    import { createClient } from 'redis';
    const client = createClient();
    ```

1.  我们还将接受命令行输入以执行我们的任务：

    ```js
    const task = process.argv[2];
    ```

1.  接下来，我们将添加一个 **错误** 事件处理器来捕获 Redis 客户端上发生的任何错误：

    ```js
    client.on('error', (err) => {
        console.log('Error:', err);
    });
    ```

1.  我们需要初始化连接：

    ```js
    await client.connect();
    ```

1.  现在，我们将添加一个语句来控制我们程序的流程。如果将任务作为输入传递给我们的程序，我们将添加此任务并列出存储在 Redis 中的任务。如果没有提供任务，则只列出存储的任务：

    ```js
    if (!task) {
        listTasks();
    } else {
        addTask(task);
    }
    ```

1.  在此**if**语句下方，我们将创建我们的**addTask()**函数：

    ```js
    async function addTask(task) {
        const key =
          `Task:${Math.random().toString(32).replace('.',
            '')}`;
        await client.hSet(key, 'task', task);
        listTasks();
    }
    ```

1.  最后，在**addTask()**函数之后，我们将添加我们的**listTasks()**函数：

    ```js
    async function listTasks() {
        const keys = await client.keys('Task:*');
        for (const key of keys) {
            const task = await client.hGetAll(key);
            console.log(task);
        }
        client.quit();
    }
    ```

1.  现在，我们可以将任务作为命令行输入运行程序。该任务将被存储在 Redis 中，然后通过**listTasks()**函数打印出来：

    ```js
    $ node tasks.mjs "Walk the dog."
    { task: 'Walk the dog.' }
    ```

我们现在已使用**redis**模块在我们的 Redis 数据存储中持久化了数据。

## 它是如何工作的…

**createClient()**方法初始化一个新的客户端连接。此方法将默认配置为连接到**localhost:6379**上的 Redis 实例，其中**6379**是 Redis 的传统端口。在**npm**的**redis**模块的早期版本中，**createClient()**方法会自动连接到服务器。然而，现在需要显式调用**client.connect()**来建立连接。

在我们的**addTask()**函数中，我们生成一个随机字符串或哈希，并将其附加到我们的任务键上。这确保了每个任务键都是唯一的，同时仍然有一个指定器表明它是一个任务，以帮助调试。这是使用 Redis 时的常见约定。

**hSet()**方法在 Redis 中设置键和值；这正是我们在 Redis 中存储任务的方式。如果我们提供了一个已存在的键，此方法将覆盖其内容。

重要提示

在 Redis 的新版本中，传统的**hmset()**方法被认为是过时的。在配方中使用的**hSet()**方法应用于设置哈希值。

在**listTasks()**函数中，我们使用**keys()**方法搜索所有存储在我们的 Redis 数据存储中且与**Tasks:***通配符匹配的键。我们正在利用**keys()**方法列出我们存储在 Redis 中的所有任务。请注意，在实际应用中，**keys()**方法应谨慎使用。这是因为，在具有许多键的应用程序中，搜索可能会产生负面的性能影响。

一旦我们有了所有任务键，我们使用**hGetAll()**方法返回每个键的值。一旦获取，我们使用**console.log()**将其打印到**STDOUT**。

**redis**模块提供的**npm**将所有可用的 Redis 命令进行了一对一的映射。请参阅[`redis.io/commands`](https://redis.io/commands)以获取 Redis 命令的完整列表。

## 更多内容…

你交互的 Redis 实例可能需要身份验证。让我们看看如何连接到一个需要密码的 Redis 实例。

### 使用 Redis 进行身份验证

要连接到需要身份验证的 Redis 客户端，我们可以在**createClient()**方法中提供凭证。

1.  我们再次可以使用 Docker 来创建一个密码保护的 Redis 实例。这个 Redis 容器将在**localhost:6380**上可用：

    ```js
    $ docker run --publish 6380:6379 --name node-redis-pw --detach redis redis-server --requirepass PASSWORD
    ```

1.  将 **tasks.mjs** 文件复制到一个名为 **tasks-auth.mjs** 的新文件中：

    ```js
    $ cp tasks.mjs tasks-auth.mjs
    ```

1.  现在，我们需要将新 Redis 实例的配置信息传递给 **createClient()** 方法：

    ```js
    import { createClient } from 'redis';
    const client = redis.createClient({
        port: 6380,
        password: 'PASSWORD',
    });
    ```

1.  现在，就像之前一样，我们可以通过命令行输入传递任务来运行程序：

    ```js
    $ node tasks-auth.mjs "Wash the car."
    { task: 'Wash the car.' }
    ```

注意，因为我们指向的是不同的 Redis 实例，它将不会包含我们在主菜谱中添加的任务。

### Redis 事务

**redis** 模块公开了一个名为 **multi()** 的方法，可以用来创建 **事务**。事务是一系列排队并作为单个单元执行的命令。

例如，我们可以使用以下命令作为事务更新任务，通过执行一个 **get()**、**set()**、**get()** 序列：

```js
import { createClient } from 'redis';
const client = createClient();
client.on('error', (err) => {
  console.log('Error:', err);
});
await client.connect();
await client.set('Task:3', 'Write letter.');
const resultsArray = await client
  .multi()
  .get('Task:3')
  .set('Task:3', 'Mail letter.')
  .get('Task:3')
  .exec();
console.log(resultsArray);
// ['Write letter.', 'OK', 'Mail letter.']
client.quit();
```

每个任务都会排队，直到执行 **exec()** 方法。如果任何命令未能排队，则批处理中的所有命令都不会执行。在 **exec()** 方法期间，所有命令按顺序执行。

## 参见

+   本章中的 *连接和持久化到 MySQL 数据库* 菜谱

+   本章中的 *连接和持久化到 PostgreSQL 数据库* 菜谱

+   本章中的 *连接和持久化到 MongoDB* 菜谱

# 探索 GraphQL

GraphQL 作为 API 的查询语言，并提供执行查询的运行时环境。与依赖于严格端点结构的 REST 不同，GraphQL 允许客户端请求他们确切需要的内容，而不需要更多，这使得它对于获取数据非常高效。这种灵活性减少了通过网络传输的数据量，并允许进行更精确和优化的查询。

在那些你的应用程序处理复杂、相互关联的数据结构的项目中，例如社交网络、电子商务平台或 **内容管理系统**（**CMSs**），GraphQL 能够在单个请求中查询深度嵌套的数据，这使得它与 Node.js 完美匹配。这种组合减少了多个 REST 端点的需求，并最小化了数据过取，优化了网络性能和开发者体验。

## 准备工作

在本教程中，我们将使用 Fastify 和 Mercurius（[`npmjs.com/package/mercurius`](http://npmjs.com/package/mercurius)），一个 Fastify 的 GraphQL 适配器，创建一个简单的 GraphQL API，其中包含书籍和作者关系。本教程将指导你设置 Node.js 项目、安装依赖项、定义你的 GraphQL 模式、实现解析器和运行你的服务器。我们将使用一个简单的内存数据结构来模拟作者和书籍的数据库。

在深入使用 Fastify 和 Mercurius 创建 GraphQL API 之前，你需要设置你的开发环境。

1.  首先为你的项目创建一个新的目录：

    ```js
    $ mkdir fastify-graphql
    $ cd fastify-graphql
    ```

1.  使用 **npm** 初始化 Node.js 项目：

    ```js
    $ npm init --yes
    ```

在你的环境准备就绪且已安装依赖项后，让我们继续到菜谱步骤。

## 如何做到这一点…

我们现在可以构建 Fastify GraphQL API 的核心功能。这个过程的一部分涉及定义我们的数据模型，设置 GraphQL 模式，编写解析器来处理数据获取，最后启动我们的服务器。

1.  让我们先安装必要的模块。我们的 GraphQL 服务器需要一些依赖项才能运行。具体来说，我们将使用 Fastify 作为 Web 框架，使用 Mercurius 作为 GraphQL 适配器。通过运行以下命令来安装这些模块：

    ```js
    $ npm install fastify mercurius
    ```

1.  现在，我们需要创建一些模拟数据来工作。这将帮助我们测试 GraphQL API，而无需数据库。在你的项目文件夹中，创建一个名为**data.js**的文件。此文件将包含作者和书籍的数组，建立它们之间简单的关系，其中每本书都与一个作者相关联：

    ```js
    $ touch data.js
    ```

1.  然后，将以下内容添加到**data.js**中，以填充一些作者和书籍数据：

    ```js
    const authors = [
      { id: '1', name: 'Richard Adams' },
      { id: '2', name: 'George Orwell' }
    ];
    const books = [
      { id: '1', name: 'Watership Down', authorId: '1' },
      { id: '2', name: 'Animal Farm', authorId: '2' },
      { id: '3', name: 'Nineteen Eighty-four', authorId:
        '2' },
    ];
    module.exports = { authors, books };
    ```

1.  接下来，我们需要创建一个 GraphQL 模式来表示我们的作者和书籍关系以及查询。创建一个名为**schema.graphql**的文件：

    ```js
    $ touch schema.graphql
    ```

    现在，将以下 GraphQL 模式添加到**schema.graphql**中：

    ```js
    type Query {
      books: [Book]
      authors: [Author]
    }
    type Book {
      id: ID
      name: String
      author: Author
    }
    type Author {
      id: ID
      name: String
      books: [Book]
    }
    ```

1.  现在，创建一个名为**resolvers.js**的文件。此文件将包含处理数据获取逻辑的函数：

    ```js
    $ touch resolvers.js
    ```

1.  要实现我们的 GraphQL 解析器，将以下代码添加到**resolvers.js**中：

    ```js
    const { authors, books } = require('./data');
    const resolvers = {
      Query: {
        books: () => books,
        authors: () => authors,
      },
      Book: {
        author: (parent) => authors.find(author =>
         author.id === parent.authorId),
      },
      Author: {
        books: (parent) => books.filter(book =>
          book.authorId === parent.id),
      },
    };
    module.exports = { resolvers };
    ```

1.  最后，我们可以创建我们的 Fastify 服务器：

    ```js
    $ touch server.js
    ```

    将以下内容添加到**server.js**中：

    ```js
    const fastify = require('fastify')();
    const mercurius = require('mercurius');
    const { readFileSync } = require('node:fs');
    const { resolvers } = require('./resolvers');
    const schema = readFileSync('./schema.graphql', 'utf-
      8');
    fastify.register(mercurius, {
      schema,
      resolvers,
      graphiql: true
    });
    fastify.listen({ port: 3000 }, () => {
      console.log('Server running at
        http://localhost:3000');
    });
    ```

1.  启动你的 Fastify 服务器：

    ```js
    $ node server.js
    ```

1.  打开您的浏览器并导航到**http://localhost:3000/graphiql**以访问 GraphiQL 界面。您应该期望看到一个类似这样的界面：![图 7.1 – 显示查询结果的 GraphiQL 界面](img/B19212_07_01.jpg)

图 7.1 – 显示查询结果的 GraphiQL 界面

1.  尝试在 GraphiQL 界面中构建一些查询。例如，尝试执行以下查询以获取所有书籍及其作者：

    ```js
     query {
      books {
        name
        author {
          name
        }
      }
    }
    ```

    期望看到以下输出：

![图 7.2 – 显示查询结果的 GraphiQL 界面](img/B19212_07_02.jpg)

图 7.2 – 显示查询结果的 GraphiQL 界面

本教程为您提供了使用 Fastify 和 Mercurius 创建 GraphQL API 的基础。从这里，您可以通过添加更复杂的数据类型、查询和突变来扩展您的 API，或者通过集成数据库来实现持久化存储。

## 它是如何工作的...

在本教程中，我们通过定义数据模型、建立 GraphQL 模式、实现数据获取的解析器和设置 Fastify 服务器，探讨了使用 Fastify 和 Mercurius 创建 GraphQL API 的过程。

通过在**data.js**中创建模拟数据，我们模拟了一个包含作者和书籍的后端数据存储。这种方法允许我们专注于 GraphQL 设置，而不必处理集成实际数据库的复杂性。这些数据代表了书籍和它们作者之间基本的关系，为我们 GraphQL 查询提供了基础。

在 **schema.graphql** 中定义的 GraphQL 模式充当服务器和客户端之间的合同。它指定了可以进行的查询类型、可以获取的数据类型以及不同数据类型之间的关系。在我们的案例中，该模式概述了如何查询书籍和作者，并指出每本书都与一个作者相关联，反之亦然。这种结构允许客户端理解和预测 API 返回的数据形状。

**resolvers.js** 中的解析器是处理在模式中指定的每个类型的获取数据逻辑的函数。它们将 GraphQL 查询连接到底层数据，本质上告诉服务器在哪里以及如何检索或修改数据。在本食谱中，解析器从模拟数据中获取书籍和作者，并解决它们之间的关系，例如找到一个书籍的作者或列出一位作者所写的所有书籍。

最后，设置 Fastify 服务器并集成 Mercurius 允许我们在 HTTP 上提供我们的 GraphQL API。服务器监听指定端口上的请求，并使用模式和解析器来处理 GraphQL 查询。

启动服务器后，您可以导航到 GraphiQL 界面，以可视化地构建和执行针对您的 API 的查询。这个交互式环境对于测试和调试查询非常有用。

是否 GraphQL 是您项目的适当架构是一个广泛的话题，它远远超出了本食谱中涵盖的基本内容。它涉及深入考虑，如优化查询性能、确保安全性、高效加载数据以避免过度或不足获取，以及与不同的数据库或 API 集成。虽然我们已经使用 Fastify 和 Mercurius 打下了基础，但深入研究这些更复杂的方面对于开发复杂的、生产就绪的 GraphQL 服务至关重要。

## 参见

+   *第六章*

+   *第十一章*
