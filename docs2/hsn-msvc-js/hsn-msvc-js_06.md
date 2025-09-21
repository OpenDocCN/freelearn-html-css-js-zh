# 6

# 同步微服务

我们在上一章实现了我们的第一个微服务，但为了演示微服务通信，我们需要运行至少一个额外的服务。为了展示 JavaScript 在微服务开发中的美感，我们将使用不同的 Node.js 框架 NestJS 来完成本章。

正如我们之前讨论的，微服务架构由多个服务组成，这种方法带来的复杂性之一是通信。我们已经知道，虽然微服务在可扩展性和开发方面提供了优势，但与单体应用相比，它们在通信方面引入了额外的复杂性。与所有内容都在一起运行的单体应用不同，微服务通过网络进行通信。这引入了*延迟*（请求处理和响应接收所需的时间）、*可靠性*（因为网络问题可能会中断通信）和*安全性*（因为您需要确保服务之间的通信安全）的挑战。

在本章中，我们将深入了解实践中微服务之间的同步通信的细节，并学习服务之间同步通信的用例。

本章涵盖了以下主题：

+   理解事务微服务的要求

+   开发事务微服务的工具

+   实践事务微服务开发

+   建立与账户微服务的同步通信

# 技术需求

为了开发和测试第二个微服务，我们需要以下内容：

+   IDE（我们更喜欢**Visual Studio Code**（**VS Code**））

+   Postman

+   您选择的浏览器

建议您从[`github.com/PacktPublishing/Hands-on-Microservices-with-JavaScript`](https://github.com/PacktPublishing/Hands-on-Microservices-with-JavaScript)下载我们的仓库，并打开`Ch06`文件夹，以便轻松跟随我们的代码片段。

# 理解事务微服务的要求

一切都从需求开始。软件需求基本上是告诉程序员软件程序需要做什么的指令。它们就像程序的食谱，概述了所需的成分（功能）和步骤（功能）。在我们开始开发之前，我们需要了解我们的需求。

系统由两个主要微服务组成：

+   **事务微服务**：这个微服务将负责处理事务。它将接收事务信息，验证与事务关联的账户，并处理事务。

+   **账户微服务**：这个微服务将提供账户信息和验证功能。我们在*第五章*中实现了这个服务。它负责验证账户是否存在且状态良好。

交易微服务将与账户微服务通信，以验证提供的 `accountId` 值。账户微服务将验证 `accountId` 值是否存在。

只有当账户存在并且处于 *活动* 或 *新* 状态时，交易才应该是成功的。对于其他状态，我们应该在交易服务中添加一个新的条目，带有 *失败* 状态：

![图 6.1：交易微服务和账户微服务之间的通信](img/B09148_06_001.jpg)

图 6.1：交易微服务和账户微服务之间的通信

首先，我们需要开发我们的交易微服务。在拥有一个正常工作的微服务之后，我们将创建交易和之前构建的账户微服务之间的同步通信。

# 开发交易微服务的工具

为了构建我们的第二个微服务，我们计划使用完全不同的工具来展示我们即使在 JavaScript 中也不依赖于具体的工具和技术。你可以使用不同的技术来开发相同的微服务，拥有多个技术选项允许你为开发选择最佳的堆栈工具。

## NestJS

作为 Node.js 框架，我们计划使用 NestJS。官方页面将其描述为“*一个用于构建高效、可靠和可扩展的服务器端应用的渐进式 Node.js 框架*。”尽管 Express.js 已经成为使用 Node.js 构建网络应用的既定标准，但它并不强迫你在所有类型的网络应用中使用 Express.js。

首先——NestJS 是另一个 Node.js 框架。查看 *第四章* 的 *Node.js 框架* 部分，了解更多关于 NestJS 的信息。以下是它提供的一些总结：

+   它推广了一种 **模块化架构**，允许你构建可扩展且易于组织的应用程序。你可以轻松地将你的应用程序组织成模块、组件、控制器和服务。

+   NestJS 是基于 **TypeScript** 构建的，并且在其核心使用 TypeScript。如果你像我一样是 **强类型** 工具/语言的忠实粉丝，那么它非常适合你。强类型语言强制执行类型安全，这意味着编译器会检查操作。这可以防止意外的崩溃和后续的不正确结果。

+   NestJS 默认支持 **验证**。它验证传入的数据，这在构建 API 时可能很有帮助。

直接将所有必需的应用程序安装到您的计算机上并不总是最佳选择。这就是为什么我们使用 Docker。虽然直接安装应用程序本身并没有错误，但 Docker 为某些情况提供了一种更高效和可管理的解决方案。

## Docker

Docker 帮助开发者更轻松地构建诸如软件程序之类的东西。想象一个包含程序运行所需的所有工具和部件的盒子。这个盒子就像一个 Docker 容器。Docker 允许你将你的程序及其所有组成部分放入这个盒子中，这样它就可以在任何计算机上以相同的方式运行。

查看*第四章*以了解如何在你的计算机上设置 Docker 的更多信息。

## Prisma ORM

Prisma 是下一代 **对象关系映射器**（**ORM**）。在编程的世界里，ORM 作为两种不同数据处理方式之间的桥梁：**面向对象编程**（**OOP**）和关系数据库。

Prisma，作为一个开源的 ORM，简化了 Node.js 应用程序中的数据库交互。它就像一套有用的工具，为你处理很多复杂的任务。好消息是，你不需要处理纯 SQL 查询。以下是它提供的内容：

+   **Prisma 客户端**：这个工具自动构建代码以访问你的数据库，使其安全且简单。它甚至在你编写代码时检查错误（如果你使用 TypeScript）。

+   **Prisma Migrate**：这个工具帮助你定义数据库的结构，并随着应用程序的变化保持其最新状态。

+   **Prisma Studio**：这是一个可视化工具，让你可以直接查看和编辑存储在数据库中的信息，就像一个用户友好的仪表板。

在底层，你可以使用 PostgreSQL、MySQL、SQL Server、SQLite、MongoDB 以及更多。当你的应用程序需要从 Prisma 支持的数据库迁移到另一个数据库时，它不会影响你的项目源代码，因为 Prisma 将你的代码从内部细节中抽象出来。

Prisma 客户端与多种构建 Node.js 应用程序的方式兼容：

+   传统的 REST API

+   现代 GraphQL API

+   高效的 gRPC API

+   任何需要使用数据库的 Node.js 项目

简而言之，Prisma 简化了 Node.js 中与数据库的交互，节省了你的时间和精力。它提供各种功能以满足你项目的需求。

现在，我们已经准备好开发我们的事务服务，从下一节开始，我们将深入了解开发过程的细节。

# 实践事务微服务开发

从技术书籍中学习的最佳方式是遵循其指示。对于所有实践章节，最好是跟随我们一起经历这个过程，并和我们一起输入每个命令。从 Git 仓库下载源代码并调查源代码也是实践中学习事物的好方法。

使用 NestJS 开始比 Express.js 更容易。它具有代码生成步骤和易于使用的包，帮助你快速开发。如果你在寻找一个更好的、现代的模板来开始，NestJS 是实现这一目标的一种方式。NestJS 提供了一个内置的命令行工具，称为 *Nest CLI*。它在你的 NestJS 应用程序的生命周期中充当一个强大的助手。它提供了以下有趣和有用的功能：

+   **项目初始化**：快速使用遵循最佳实践的目录布局设置新的 NestJS 项目。

+   **开发支持**：以开发模式运行你的应用程序以实现热重载和简化调试。

+   **生产构建**：将应用程序捆绑以部署到生产环境，优化其效率。

+   **代码生成**：使用脚本来生成各种组件，如控制器、服务、模块等，这样可以节省您的时间并确保一致性。

让我们开始开发过程：

1.  为您的项目创建一个文件夹（在我们的 Git 仓库中是 `Ch06`）。

1.  打开 VS Code 并从其中打开您的文件夹。

1.  前往 **终端** 菜单，然后选择 **新建终端**。

1.  输入 `npm i -g @nestjs/cli` 并按 *Enter* 键。

安装 NestJS CLI 后，我们可以使用单个命令创建我们的项目模板。只需输入 `nest new transactionservice` 并再次按 *Enter* 键。

如果在尝试在 Windows 中运行脚本时遇到 `Cannot Be Loaded Because Running Scripts is Disabled on This System` 错误消息，请按照以下步骤解决：

1.  **打开 Windows PowerShell**：按 *Win* + *X* 并选择 **Windows PowerShell (管理员)** 以具有管理员权限打开它。

1.  **设置执行策略**：在 PowerShell 窗口中，输入以下命令并按 *Enter* 键：

    ```js
    Y (for Yes) and press Enter to confirm.
    ```

1.  **再次运行您的脚本**：再次尝试运行您的脚本。问题现在应该已经解决。

在项目设置期间，您将得到一个提示，要求您选择包管理器（*图 6.2*）。我们有以下选项：

+   `npm`

+   Yarn

+   Pnpm

我们选择 `npm` 作为这个项目的包管理器。它在包管理器世界中是一个强有力的竞争者，尤其是对于 Node.js 项目。它拥有庞大的包注册库，是 Node.js 的默认选择，并且拥有庞大的社区（*图 6.2*）：

![图 6.2：NestJS 提供选择包管理器](img/B09148_06_002.jpg)

图 6.2：NestJS 提供选择包管理器

在您做出选择后，CLI 将为我们生成一个项目模板（*图 6.3*）：

![图 6.3：NestJS 的 CLI 生成的文件夹结构](img/B09148_06_003.jpg)

图 6.3：NestJS 的 CLI 生成的文件夹结构

`src` 和 `test` 文件夹不为空，并包含初始项目骨架（*图 6.4*）：

![图 6.4：NestJS 的 CLI 生成的 src 和 test 文件夹](img/B09148_06_004.jpg)

图 6.4：NestJS 的 CLI 生成的 src 和 test 文件夹

要成功运行生成的模板，请按照以下步骤操作：

1.  使用 `cd transactionservice` 命令从终端导航到 `transactionservice` 文件夹。对于所有类型的命令，我们都需要导航到这个文件夹才能正确运行它们。如果您不想每次都输入 `cd` 命令，可以直接从 VS Code 中打开 `transactionservice` 文件夹。

1.  输入 `npm run start:dev`。

    此命令启动一个特殊的服务器，它可以帮助您快速查看更改。它密切监视您的文件，如果它看到任何不同，它会自动修复并刷新服务器。这意味着您可以直接看到更新，而无需自己重新启动一切。

1.  打开您喜欢的浏览器并导航到 `http://localhost:3000`（*图 6.5*）：

![图 6.5：成功的 NestJS 项目运行结果](img/B09148_06_005.jpg)

图 6.5：成功的 NestJS 项目运行结果

下一个子节将帮助我们了解如何准备我们的环境，并轻松构建我们的微服务。

## Docker 化你的 PostgreSQL 实例

**PostgreSQL** 是在数据库中存储数据时最佳选择之一。我们将使用 Docker 来容器化我们的数据库，将其与其他环境隔离开来。

右键单击你的根项目文件夹（对于我们来说是 `transactionservice`），并添加一个 `docker-compose.yml` 文件。

打开这个空文件并添加以下行：

```js
networks:
  my-app-network:  # Define the network name exactly as used later
services:
  postgres:
    image: postgres
    env_file:
      - .env
    environment:
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB}
    ports:
      - ${POSTGRES_PORT}:5432
    volumes:
      - postgres_data:/var/lib/postgresql/data
    networks:
      - my-app-network  # Add the service to the network
  pgadmin:
    image: dpage/pgadmin4
    env_file:
      - .env
    environment:
      - PGADMIN_DEFAULT_EMAIL=${PGADMIN_DEFAULT_EMAIL}
      - PGADMIN_DEFAULT_PASSWORD=${PGADMIN_DEFAULT_PASSWORD}
      - POSTGRES_HOST=postgreshost
      - POSTGRES_USER=${POSTGRES_USER}
      - POSTGRES_PASSWORD=${POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB}
      - PGADMIN_CONFIG_SERVER_MODE=False
      - PGADMIN_CONFIG_MASTER_PASSWORD_REQUIRED=False
    ports:
      - ${PGADMIN_PORT}:80
    depends_on:
      - postgres
    user: root
    volumes:
      - postgres_data:/var/lib/pgadmin/data
    networks:
      - my-app-network  # Add the service to the network
volumes:
  postgres_data:
```

当我们讨论安装 Apache Kafka 时，我们使用了 `docker-compose.yml` 在 *第四章*。我们有一个关于容器化的单独章节，但让我们在这里解释这个文件本身，以便更清晰。

`docker-compose.yml` 文件是一个 YAML 配置文件，用于定义和管理多容器 Docker 应用程序。Docker Compose 是一个工具，允许你在一个文件中定义应用程序所需的服务、网络和卷，这使得管理复杂的设置更加容易。

每个服务代表一个容器化的应用程序组件。

你可以为你的服务定义自定义网络，以便它们之间进行通信。默认情况下，Docker Compose 为你的应用程序创建一个默认网络，但你也可以定义自定义网络来控制特定服务之间的通信。

你也可以定义命名卷或将主机目录挂载到容器中，以持久化数据或在不同容器之间共享文件。

Docker Compose 允许你通过单个命令启动所有服务，而不是逐个运行服务，`docker-compose` 帮助你通过单个命令 (`docker-compose up`) 启动整个基础设施，并在不同的环境中一致地管理它。

这个 `docker-compose.yml` 文件定义了一个 Docker Compose 配置，用于设置两个服务：`postgres` 和 `pgadmin`。让我们来分解一下：

+   `networks`：在你的 Docker Compose YAML 文件中的此部分定义了命名网络，这些网络可以被应用程序的服务使用。这些网络提供了一种容器之间以受控和隔离的方式相互通信的方法。

+   `services`：此部分定义要创建的服务。

    +   `postgres`：此服务使用官方的 PostgreSQL Docker 镜像。它设置了一个 PostgreSQL 数据库容器。

    +   `image: postgres`：指定用于此服务要使用的 Docker 镜像。

    +   `env_file`：指定一个文件，从中读取环境变量。

    +   `environment`：为 PostgreSQL 容器设置环境变量，包括用户名、密码和数据库名。

    +   `ports`：将容器的 PostgreSQL 端口映射到主机上的端口，允许外部访问。

    +   `volumes`：挂载一个卷以持久化 PostgreSQL 数据。

+   `pgadmin`：此服务使用 `pgAdmin` 4 Docker 镜像来为 PostgreSQL 设置基于 Web 的管理界面。

    +   `image: dpage/pgadmin4`：指定 `pgAdmin` 4 的 Docker 镜像。

    +   `env_file`：类似于 `postgres` 服务，这指定了一个用于读取环境变量的文件。

    +   `environment`：为 `pgAdmin` 设置环境变量，包括默认电子邮件、密码和 PostgreSQL 连接详情。

    +   `ports`：将容器的端口 `80` 映射到主机上的一个端口。

    +   `depends_on`：指定此服务依赖于 `postgres` 服务，确保在启动 `pgAdmin` 之前 PostgreSQL 数据库可用。

    +   `user: root`：指定容器应以 root 用户运行。

    +   `volumes`：将卷挂载以持久化 `pgAdmin` 数据。

+   `volumes`：此部分定义了一个名为 `postgres_data` 的命名卷，它被两个服务用于持久化数据。

最后，这个 Docker Compose 配置设置了一个 PostgreSQL 数据库容器和一个 `pgAdmin` 容器，提供了一个方便的方式来使用基于 Web 的界面管理和交互 PostgreSQL 数据库。要运行你的 `docker-compose` 文件，只需导航到该文件的文件夹，并在终端中输入 `docker-compose up -d`。

我们不是直接在 `docker-compose` 文件中添加凭据/值，而是可以从一个 `.env` 文件中指定（我们之前已经讨论过这个文件），Docker 可以从环境变量中读取所需的数据。只需在你的主文件夹内（对我们来说就是 `transactionservice` 文件夹）创建一个 `.env` 文件，并添加 Docker 运行成功所需的缺失配置：

```js
# PostgreSQL settings
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DB=tservice_db
POSTGRES_PORT=5438
# pgAdmin settings
PGADMIN_DEFAULT_EMAIL=admin@tservice.com
PGADMIN_DEFAULT_PASSWORD=tservice_password
PGADMIN_PORT=5050
```

现在我们已经启动并运行了我们的 PostgreSQL 数据库。在大多数情况下，开发者更喜欢不直接使用 SQL 查询与数据库交互。并非所有开发者都对 SQL 有坚实的理解，即使如此，在大多数情况下，使用纯 SQL 查询来操作数据库也不是一个好的选择。相反，我们有各种可用的包，它们抽象了原始 SQL 的复杂性，使我们能够在不需要深入 SQL 知识的情况下创建美观的应用程序。其中之一就是 Prisma。正如我们之前提到的，Prisma 是一个开源的 ORM，它自动化并抽象了你处理数据库时需要执行的大多数操作。

要开始使用 Prisma 的工作，我们需要一个 *CLI*。Prisma CLI 是一系列工具的组合，帮助我们轻松地进行迁移、种子和执行其他数据库相关操作。你只需从终端运行 `npm install prisma -D` 命令。执行命令后，`npm` 应该成功地将 Prisma CLI 作为开发依赖项安装。你可以在 `package.json` 的 `devDependencies` 部分进行检查。

在 Prisma CLI 之后，现在是时候安装 Prisma 本身了。`npx prisma init` 命令处理 Prisma 包的初始化。它将创建一个名为 `prisma` 的额外文件夹，其中包含一个 `schema.prisma` 文件和一个 `.env` 文件。在我们的情况下，我们已经有了一个 `.env` 文件，所以运行前面的命令将更新我们现有的 `.env` 文件。打开你的 `.env` 文件，并在文件末尾更新 `DATABASE_URL` 的值：

```js
DATABASE_URL="postgres://postgres:postgres@localhost:5438/tservice_db"
```

在你的 Prisma 设置的核心是 `schema.prisma` 文件。这个文件使用 **Prisma 模式语言**（**PSL**），这是一种定义数据库结构的声明式方法。它作为 Prisma 的中心配置，指定了数据库连接和 Prisma 客户端 API 的生成。以下代码演示了如何为 Prisma 定义一个简单的模式文件：

```js
generator client {
  provider = "prisma-client-js"
}
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}
```

用 PSL 编写的 `schema.prisma` 文件充当你的数据库蓝图，有三个关键部分：

+   `generator`: 这个部分配置了 Prisma 客户端生成器。然后生成强大的 API Prisma 客户端，以帮助你访问数据库。

+   `datasource`: 在这里，你定义数据库连接的详细信息。这包括数据库提供者和连接字符串，通常利用 `DATABASE_URL` 环境变量以提高便利性。

+   `Model:` 这里是数据库模式的核心所在。你通过指定表及其对应的字段来定义你的数据结构。

下一个部分描述了如何在 `schema.prisma` 文件中建模你的数据。

## 数据建模

`schema.prisma` 文件是我们需要添加模型的主要地方。建模是一种特殊的语言，在 SQL 之上。它将你与 SQL 的内部细节隔离开来，并以更易于阅读的语言提供数据。

打开 `prisma` 文件夹下的 `schema.prisma` 文件，并添加以下模型结构：

```js
model Transaction {
  id           Int      @id @default(autoincrement())
  status       Status
  accountId    String   @default(uuid())
  description  String?
  createdAt    DateTime @default(now())
  updatedAt    DateTime @updatedAt
}
enum Status {
  CREATED
  SETTLED
  FAILED
}
```

提供的代码定义了一个名为 `Transaction` 的 Prisma 模型以及在你 NestJS 应用程序模式中的一个名为 `Status` 的枚举。

下面是每个部分的分解：

+   `id`: 这个字段代表每个交易的唯一标识符。它的类型是 `Int`，并自动标记为使用 `@id` 指令的主键。此外，`@default(autoincrement())` 确保每个交易自动生成一个新且唯一的 ID。

+   `status`: 这个字段定义了交易的当前状态。它的类型是 `Status`，将引用 `Status` 枚举（`Enumeration(enum Status)`）。

+   `accountId`: 这个字段存储参与交易的关联账户的标识符。它的类型是 `String`，并使用 `@default(uuid())` 默认生成一个 **全局唯一标识符**（**UUID**）。

+   `description`: 这个可选字段允许存储交易的简要描述。它的类型是 `String?`，表示它可以 `null`。

+   `createdAt`: 这个字段捕获了交易创建的时间戳。它的类型是 `DateTime`，并使用 `@default(now())` 自动将创建时间设置为当前时刻。

+   `updatedAt`: 这个字段在交易记录被修改时自动更新。它的类型是 `DateTime`，并使用 `@updatedAt` 指令来实现这种行为。

好的——但是如何根据我们在 `schema.prisma` 中定义的模式生成 SQL 呢？

从命令行（VS Code 终端）运行 `npx prisma migrate dev --name init` 以开始迁移之旅。在 NestJS 和 Prisma 的上下文中，**迁移**指的是管理数据库模式随时间变化的过程。

这里是命令的分解：

+   `npx prisma migrate dev`：此命令以开发模式调用 Prisma 迁移工具。

+   `--name init`：此选项指定新迁移的名称。在这里，它设置为 `init`，可能表示数据库模式的初始设置。

通过运行此命令，你实际上是在创建一个起点，用于使用 Prisma 迁移管理你的数据库模式变化。随着你对 `schema.prisma` 文件进行修改，Prisma 将自动生成新的迁移来反映这些更改。

命令最终会在 `migrations` 文件夹中创建一个 `migration.sql` 文件（*图 6**.6*）：

![图 6.6：自动生成的迁移结构](img/B09148_06_006.jpg)

图 6.6：自动生成的迁移结构

检查生成的 `migration.sql` 文件：

```js
-- CreateEnum
CREATE TYPE "Status" AS ENUM ('CREATED', 'SETTLED', 'FAILED');
-- CreateTable
CREATE TABLE "Transaction" (
    "id" SERIAL NOT NULL,
    "status" "Status" NOT NULL,
    "accountId" TEXT NOT NULL,
    "description" TEXT,
    "createdAt" TIMESTAMP(3) NOT NULL DEFAULT CURRENT_TIMESTAMP,
    "updatedAt" TIMESTAMP(3) NOT NULL,
    CONSTRAINT "Transaction_pkey" PRIMARY KEY ("id")
);
```

现在，你应该在你的数据库中有名为 `Transaction` 和 `_prisma_migrations` 的表格。Docker 设置服务可能需要几秒钟，所以可能需要稍等片刻。为了检查这一点，让我们执行以下操作：

1.  打开 Docker Desktop 并确保所有服务都在运行。

1.  点击 `postgres` 容器。

1.  （从 `172.26.0.2`）：

1.  从浏览器导航到 `http://localhost:5050/browser/`。

1.  右键点击 **服务器**，然后从菜单中选择 **注册** | **服务器**。

1.  在 `localhost` 下检查其值。

1.  前往 `172.26.0.2`（为我们操作）

1.  `5432`

1.  `tservice_db`

1.  `postgres`

1.  `postgres`

![图 6.7：postgres 服务器注册窗口](img/B09148_06_007.jpg)

图 6.7：postgres 服务器注册窗口

1.  点击 **保存** 按钮，你的服务器连接应该成功。

1.  现在，展开 `localhost`（或你的名称）| `tservice_db` | **模式** | **公共** | **表格**（*图 6**.8*）：

![图 6.8：通过 Prisma 迁移后的 postgres 表格](img/B09148_06_008.jpg)

图 6.8：通过 Prisma 迁移后的 postgres 表格

1.  如果你已经在本地上安装了 `pgAdmin`，要连接到你的 Docker `postgres` 实例，只需将以下内容输入到 `postgres` 的服务器注册窗口（*图 6**.7*）：

    +   `localhost`

    +   `5438`

    +   `tservice_db`

    +   `postgres`

    +   `postgres`

1.  点击 **保存** 按钮，你的服务器连接应该成功。

当使用 NestJS 和 Prisma ORM 时，你看到的 `_prisma_migrations` 表在管理数据库模式变化中起着至关重要的作用。它有以下职责：

+   跟踪应用的数据库迁移

+   确保迁移只应用一次

+   保持 Prisma 模式和实际数据库结构之间的一致性

每次你运行 Prisma 迁移时，都会在 `_prisma_migrations` 表中添加一个新条目。

当 Prisma 需要应用迁移时，它会检查 `_prisma_migrations` 表以查看基于唯一哈希哪些迁移已经被运行。

这防止了多次应用相同的迁移，从而可能损坏您的数据。

手动修改 `_prisma_migrations` 表可能会导致不一致性和错误。不要编辑、删除或修改它。此表对于 Prisma 有效地管理迁移至关重要。

简而言之，`_prisma_migrations` 表充当您数据库模式更改的日志簿，确保迁移过程平稳且受控。

## 种子测试数据

种子数据涉及将一组初始数据填充到您的数据库中。如果您希望在运行应用程序之前数据库已有初始数据，您可以使用种子操作。

在 `prisma` 文件夹下添加一个 `seed.ts` 文件，并包含以下内容：

```js
// prisma/seed.ts
import { PrismaClient } from '@prisma/client';
// initialize Prisma Client
const prismaClient = new PrismaClient();
async function seedData() {
  // create two dummy recipes
  const first_transaction = await
    prismaClient.transaction.upsert({
    where: { id:1 },
    update: {},
    create: {
      id:1,
      status: 'CREATED',
      accountId: '662c081370bd2ba6b5f04e94',
      description: 'simple transaction',
    }
  });
  console.log(first_transaction);
}
// execute the seed function
seedData()
  .catch(e => {
    console.error(e);
    process.exit(1);
  })
  .finally(async () => {
    // close Prisma Client at the end
    await prismaClient.$disconnect();
  });
```

前往 `package.json` 并在 `devDependencies` 之后添加以下内容：

```js
"prisma": {
    "seed": "ts-node prisma/seed.ts"
  }
```

现在，打开终端窗口并输入 `npx prisma db seed` 命令。您应该会看到一个表示操作成功的消息（*图 6.9*）：

![图 6.9：执行种子操作](img/B09148_06_009.jpg)

图 6.9：执行种子操作

使用 `PgAdmin` 打开 `Transaction` 表，您将看到您成功插入的第一行数据（*图 6.10*）：

![图 6.10：种子操作后的交易表](img/B09148_06_010.jpg)

图 6.10：种子操作后的交易表

是时候解释我们已插入 `seed.ts` 文件中的内容了：

+   `PrismaClient from @prisma/client`: 这行代码导入必要的类，以便使用 Prisma 与我们的数据库模式进行交互。

+   `const prismaClient = new PrismaClient()`: 在这里，我们创建 `PrismaClient` 类的实例，该实例将用于执行数据库操作。

+   `async function seedData() { ... }`: 这个函数是脚本的灵魂，被标记为 `async`，因为它包含涉及与数据库交互的异步操作。

+   `const first_transaction = await prismaClient.transaction.upsert({ ... })`: 这行代码执行核心的种子操作。

+   `prismaClient.transaction`: 这部分通过初始化客户端访问 Prisma 模式中的事务模型。

+   `.upsert({ ... }):` `upsert` 方法是一种方便的方式，用于在数据库中创建或更新记录。它根据提供的 `where` 子句检查现有数据，并执行相应的操作。

+   如果已存在具有 `id: 1`（假设您的模式具有 ID 字段）的记录，则会发生以下情况：

    +   `update` 对象（此处为空）将用于更新具有 `id:` `1` 的现有记录（但由于它是空的，因此不会发生更新）。

    +   `create` 对象定义了如果不存在具有 ID 的记录时新事务记录的数据。

        +   `create Object (Seed Data)`: 此对象定义了要创建的虚拟事务的详细信息。

        +   `id: 1`: 将事务的 ID 设置为 1（如果需要，请替换为唯一值）。

        +   `status: 'CREATED'`：将事务的初始状态设置为`CREATED`。

        +   `accountId: '662c081370bd2ba6b5f04e94'`：将账户 ID 分配给事务（你可以使用任何 ID）。

        +   `description: 'simple transaction'`：为事务提供描述性文本。

要将业务规则应用到我们的应用程序中，我们需要在数据库之上添加一个额外的层，这将是我们的服务层。下一节将介绍用于事务数据库的服务层。

## 实现事务服务

我们已经完成了数据库的工作。作为一个经典的开发风格，现在是时候在我们的数据库上创建一个服务了。使用 NestJS 创建服务层很简单，尤其是如果你处理 Prisma ORM 的话。

首先，让我们使用`npx nest generate module` `prisma`命令创建我们的模块。

这个 Prisma CLI 命令应该会生成一个名为`prisma`的新文件夹，并在该文件夹下生成一个`prisma.module.ts`文件。此命令还将影响`src`文件夹下的`app.module.js`文件。

我们还需要运行一个额外的命令来生成我们的服务文件：

```js
Npx nest generate service prisma
```

此命令将在`src/prisma`下创建`prisma.service.ts`、`prisma.service.spec.ts`文件，并更新`prisma.module.ts`文件。

对于本章，你可以从项目中删除所有具有`.spec.ts`扩展名的文件。这些文件包含应用程序组件的单元测试，通常是服务和控制器。我们有一个单独的章节来处理单元测试；为了章节的简洁性，我们不需要它们。现在，将`prisma.service.ts`的内容替换为以下内容：

```js
import { Injectable } from '@nestjs/common';
import { PrismaClient } from '@prisma/client';
@Injectable()
export class PrismaService extends PrismaClient {}
```

在这段代码中，我们对`prisma`服务有一个直接的实现：

+   `import { Injectable } from '@nestjs/common';`：这一行从`@nestjs/common`模块中导入了`Injectable`装饰器。这个装饰器将类标记为 NestJS 可注入的服务，使其在应用程序的其他部分可用，以便进行**依赖注入**（**DI**）。

+   `import { PrismaClient } from '@prisma/client';`：这一行从`@prisma/client`包中导入了`PrismaClient`类。这个类提供了一个接口，用于使用 Prisma 查询与数据库进行交互。

+   `@Injectable()`: 这个应用于类声明的装饰器将类标记为 NestJS 可注入的服务。NestJS 将管理这个服务的生命周期，并将其提供给需要数据库访问的其他组件。

+   `export class PrismaService extends PrismaClient {}`: 这一行定义了`PrismaService`类。它从`PrismaClient`类继承，从而获得了 Prisma 提供的一切数据库交互方法。

从本质上讲，这段代码创建了一个专门用于通过 Prisma 与我们的数据库交互的服务。然后，这个服务可以被注入到我们应用程序的其他部分（如控制器）中，以执行数据库操作。

`prisma.service.ts`文件作为 Prisma 客户端的包装器。它是一个可注入的元素，我们可以将其注入到模块中。

让我们更新我们的`prisma.module.ts`文件以包含以下内容：

```js
import { Module } from '@nestjs/common';
import { PrismaService } from './prisma.service';
@Module({
  providers: [PrismaService],
  exports: [PrismaService]
})
export class PrismaModule {}
```

要使 Prisma 服务在您的 NestJS 应用程序中可用，您需要创建一个专门的模块。此模块将导入`PrismaService`类并将其提供给其他模块或组件注入。这就是为什么我们有`prisma.module.ts`文件的原因。

现在，使用 UI 来处理 API 很流行，它允许我们记录和轻松使用端点。允许我们这样做的一个包是 Swagger。下一节将解释如何为我们的端点集成 Swagger。

## 配置 Swagger

为了使我们的 API 具有可见的文档和可视化的使用方式，我们将配置 Swagger UI。

打开 VS Code 终端并输入以下内容：

```js
npm install --save @nestjs/swagger swagger-ui-express
```

打开`src/main.ts`并更新其内容以集成 Swagger：

```js
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { SwaggerModule, DocumentBuilder } from '@nestjs/swagger';
// bootstrap function
async function bootstrap() {
  // Create a NestJS application instance
  const app = await NestFactory.create(AppModule);
  // new Swagger document configuration
  const config = new DocumentBuilder()
    .setTitle('Transaction API') // title of the API
    .setDescription('Transaction API description')
    // description of the API
    .setVersion('1.0') // version of the API
    .build(); // Build the document
  // Create a Swagger document
  const document = SwaggerModule.createDocument(app,
    config);
  // Setup Swagger module
  SwaggerModule.setup('api', app, document);
  // Start the application and listen for requests on port 3000
  await app.listen(3000);
}
// Call the bootstrap function to start the application
bootstrap();
```

让我们理解这里的代码：

+   `NestFactory`来自`@nestjs/core`: 这个导入提供了创建 NestJS 应用程序实例的核心功能。

+   `AppModule`来自`./app.module`: 这将导入您的主应用程序模块，其中定义了您的 NestJS 应用程序的所有必要组件和服务。

+   `SwaggerModule`和`DocumentBuilder`来自`@nestjs/swagger`: 这些导入用于将 Swagger 文档与您的 NestJS 应用程序集成。

+   Bootstrap 函数（`async`）:

    +   这个函数被标记为`async`，因为它涉及异步操作，例如创建应用程序实例和监听传入的请求。

    +   它作为您的 NestJS 应用程序的入口点，通常在`main.ts`文件的底部调用。

+   `const app = await NestFactory.create(AppModule);`: 这行代码使用`AppModule`类创建一个新的 NestJS 应用程序实例。`await`关键字表示该函数将在应用程序创建完成后才继续执行。

+   Swagger 配置：

    +   `const config = new DocumentBuilder()...`: 这里，您正在使用`DocumentBuilder`类配置 Swagger 文档。

    +   `.setTitle('Transaction API')`: 将您的 API 文档的标题设置为`Transaction API`。

    +   `.setDescription('Transaction API description')`: 提供了您 API 的简要描述。

    +   `.setVersion('1.0')`: 将您的 API 版本设置为`1.0`。

    +   `.build()`: 根据提供的配置选项构建 Swagger 文档。

+   `const document = SwaggerModule.createDocument(app, config);`: 这行代码使用`SwaggerModule`类生成实际的 Swagger 文档。它将 NestJS 应用程序实例（`app`）和构建的配置（`config`）作为参数。

+   `SwaggerModule.setup('api', app, document);`: 这段代码将 Swagger 文档与您的应用程序集成。它将文档的路径前缀设置为`api`（例如，`http://localhost:3000/api`），并将生成的文档（`document`）与应用程序（`app`）关联。这使得开发人员可以在指定的 URL 访问交互式 Swagger 文档。

+   `await app.listen(3000);`：此行启动 NestJS 应用程序并使其在端口`3000`上监听传入的请求。您可以将此端口号更改为您想要的选项。

总体而言，此`main.ts`文件执行两个关键任务：

+   `AppModule`类和启动服务器监听请求

+   **集成 Swagger 文档**：它为您的 API 配置并提供 Swagger 文档，允许开发者通过交互式界面探索您的 API 端点、理解数据模型并与您的 API 交互

导航到`localhost:3000/api`，您应该能看到 Swagger 页面（*图 6.11*）：

![图 6.11：Swagger UI](img/B09148_06_011.jpg)

图 6.11：Swagger UI

如您将意识到的那样，我们还没有任何端点；下一节将讨论创建它们。

## 在事务实现上工作

要开始处理事务，首先，我们需要生成资源。为了实现事务的**创建、读取、更新和删除**（**CRUD**）操作，我们首先将生成 REST 资源，为模块、控制器、服务和**数据传输对象**（**DTO**）创建样板代码。

运行`npx nest generate resource transaction`命令以生成事务的资源：

![图 6.12：选择传输层](img/B09148_06_012.jpg)

图 6.12：选择传输层

它将询问您要选择哪个传输层（*图 6.12*）。选择`REST API`并按*Enter*键。接下来，您将被询问`您想要生成 CRUD 入口点吗？`选择`Y`，然后应该会生成以下文件（*图 6.13*）：

![图 6.13：事务的 CRUD 生成](img/B09148_06_013.jpg)

图 6.13：事务的 CRUD 生成

运行`npm run start:dev`并导航到`localhost:3000/api`。您应该会看到一个存储事务样板端点的页面（*图 6.14*）：

![图 6.14：事务的 Swagger UI](img/B09148_06_014.jpg)

图 6.14：事务的 Swagger UI

当然，我们不需要实现所有这些 CRUD 端点。我们需要以下函数：

+   获取所有事务（`GET /transaction`）

+   通过 ID 获取事务（`GET /transaction/{id}`）

+   创建事务（`POST /transaction`）

让我们移除其余未使用的代码块和文件。

移除以下文件：

+   `transaction/transaction.controller.spec.ts`

+   `transaction/dto/update-transaction.dto.ts`

+   `transaction/transaction.service.spec.ts`

移除以下代码块：

+   `transaction.service.ts`中的`remove`和`update`函数

+   `transaction.controller.ts`中的`remove`和`update`函数

如果您还没有移除它们，请也移除以下文件：

+   `app.controller.ts`

+   `app.module.ts`

+   `app.service.ts`

将`main.ts`更新为与`TransactionModule`一起工作，而不是`AppModule`：

```js
import { NestFactory } from '@nestjs/core';
import { TransactionModule } from
  './transaction/transaction.module';
import { SwaggerModule, DocumentBuilder } from
  '@nestjs/swagger';
// bootstrap function
async function bootstrap() {
  // Create a NestJS application instance
  const app = await NestFactory.create(TransactionModule);
……..
```

最终，您将拥有三个端点（*图 6.15*）：

![图 6.15：最终事务端点](img/B09148_06_015.jpg)

图 6.15：最终事务端点

在生成 REST 资源后，我们准备好集成我们的 `PrismaClient` 类。拥有这个客户端将帮助我们轻松地与数据库交互。首先，让我们更新我们的 `transaction.module.ts` 文件以包含 `PrismaModule`：

```js
import { Module } from '@nestjs/common';
import { TransactionService } from './transaction.service';
import { TransactionController } from
  './transaction.controller';
import { PrismaModule } from '../prisma/prisma.module';
@Module({
  imports: [PrismaModule],
  controllers: [TransactionController],
  providers: [TransactionService],
})
export class TransactionModule {}
```

在 `imports` 数组中包含 `PrismaModule` 将使 `PrismaService` 可用于 `TransactionService`。

现在，打开 `transaction.service.ts` 文件并做出以下更改：

```js
import { Injectable } from '@nestjs/common';
import { CreateTransactionDto } from
  './dto/create-transaction.dto';
import { PrismaService } from 'src/prisma/prisma.service';
@Injectable()
export class TransactionService {
  constructor(private readonly prisma: PrismaService) {}
…….
```

交易控制器文件 (`transaction.controller.ts`) 已经将 `transactionservice` 作为注入的服务。它具有请求交易服务并检索数据所需的所有合约。

打开 `transaction.controller.ts` 并查看 `findAll()` 方法：

```js
@Get()
  findAll() {
    return this.transactionService.findAll();
  }
```

同样适用于 `POST` 和单个 `GET` 请求。我们唯一需要做的是在调用 `transactionservice` 的 `findAll()` 方法时调用 Prisma 以提供所有数据。因此，打开 `transaction.service.ts` 并更新 `findAll()` 方法的内容 (*图 6.14*）：

```js
findAll() {
    return this.prisma.transaction.findMany();
  }
```

使用 `findMany()`，我们能够通过 Prisma 从交易表调用所有交易数据。让我们运行我们的应用程序（运行 `npm run start:dev`）并从 Swagger UI 运行我们的端点。

从 Swagger UI 中打开 `GET /transaction`，点击 **尝试操作** 按钮，然后点击 **执行** 按钮。现在，你应该只看到我们在讨论播种数据时迁移到我们数据库中的数据 (*图 6.16*）：

![图 6.16：获取所有交易的响应](img/B09148_06_016.jpg)

图 6.16：获取所有交易的响应

为了修改你的通过 ID 的 `GET` 请求端点使其正常工作，打开 `transaction.service.ts` 并将 `findOne()` 方法替换为以下内容：

```js
findOne(id: number) {
    return this.prisma.transaction.findUnique({
      where: { id },
    });
  }
```

当涉及到检索数据时，一切都非常简单，但创建数据又是如何呢？

我们在 `transaction.controller.ts` 中有一个 `POST` 端点，这是在我们生成文件本身时自动生成的：

```js
@Post()
  create(@Body() createTransactionDto:
    CreateTransactionDto) {
      return
      this.transactionService.create(createTransactionDto);
    }
```

当我们的资源被创建时，`CreateTransactionDTO` 也被生成；你可以在 `src/transaction/dto` 文件夹中找到它。令人惊讶的是，它只有一个类声明：

```js
export class CreateTransactionDto {}
```

你应该手动将所需的属性添加到类中。我们的 DTOs 只是从源到目的地传输数据。DTOs 在各种编程语言和框架中使用，并不特定于 NestJS。它们作为在不同应用层之间高效传输数据的方式。我们还在从用户获取数据并基于此数据创建 DTOs 之前有验证的可能性。这就是为什么我们将使用 `class-validator` 包来验证我们的数据。要从终端安装它，请运行以下命令：

```js
npm install class-validator
```

打开 `create-transaction.dto.ts` 并添加以下内容：

```js
import { IsString, IsOptional, IsEnum, IsNotEmpty,IsUUID }
  from 'class-validator';
enum Status {
    CREATED = 'CREATED',
      SETTLED= 'SETTLED',
    FAILED = 'FAILED',
    }
export class CreateTransactionDto {
  @IsNotEmpty()
  @IsEnum(Status)
  status: Status;
  @IsUUID()
  @IsNotEmpty()
  accountID: string;
  @IsOptional()
  @IsString()
  description?: string;
}
```

更新你的 `POST` 方法 (`transaction.controller.ts`) 以接受 `CreateTransactionDTO` 并执行它：

```js
@Post()
  create(@Body() createTransactionDto:
    CreateTransactionDto) {
      return
      this.transactionService.create(createTransactionDto);
    }
```

现在，让我们运行我们的应用程序。从 Swagger UI 中，打开 `POST/ transaction` 并提供以下 JSON 负载：

```js
{
  "status": "CREATED",
  "accountId": "662c081370bd2ba6b5f04e94",
  "description": "Optional transaction description"
}
```

点击 **执行** 按钮，现在我们在这里 (*图 6.17*）：

![图 6.17：成功创建交易](img/B09148_06_017.jpg)

图 6.17：成功创建交易

从下一节开始，我们将探讨如何建立交易和账户微服务之间的通信。

# 建立与账户微服务的同步通信

我们已经完成了交易服务，但唯一缺少的是我们的账户服务。交易服务允许我们从有效载荷中指定`accountId`值和状态。我们需要进行以下更改：

+   验证提供的`accountId`是否存在且处于有效状态（新或活动状态）

+   如果`accountId`有效，则创建一个状态为`Created`的交易

+   如果`accountId`无效，则创建一个状态为`Failed`的交易

这里的目的不是完全实现交易域。当然，当前的域比之前的域有更多的要求，但我们的重点是练习并建立交易和账户服务之间的同步通信。

我们已经讨论了微服务之间同步通信的优缺点。虽然异步通信为微服务提供了许多好处，但在某些情况下，同步通信可能更适合。如果微服务之间的交互需要直接的逻辑和即时响应，同步通信可能更容易实现。此外，对于需要立即显示信息或确认操作的用例，使用同步通信是有益的。它创建了紧密耦合，但在某些情况下，微服务可能紧密耦合，并且高度依赖于彼此的结果来完成任务。同步通信允许更受控的流程，确保一个服务在另一个服务提供必要信息之前不继续进行。在某些时候，同步通信可能更容易调试。由于整个交互是一次性发生的，因此跟踪错误和理解数据流更为直接。这有助于开发或解决特定问题。 

为了与账户服务通信，我们需要一个 HTTP 客户端包。最常用的包之一是`axios`。让我们使用`npm`来安装它：

```js
npm i --save @nestjs/axios axios
```

现在，我们需要从`axios`导入`HttpModule`，并从`transaction.module.ts`中导入它。以下是该文件中的最终代码：

```js
import { Module } from '@nestjs/common';
import { TransactionService } from './transaction.service';
import { TransactionController } from
  './transaction.controller';
import { PrismaModule } from '../prisma/prisma.module';
import { HttpModule } from '@nestjs/axios';
@Module({
  imports: [PrismaModule,HttpModule],
  controllers: [TransactionController],
  providers: [TransactionService],
})
export class TransactionModule {}
```

在从`transaction.module.ts`导入`HttpModule`之后，我们能够在`transaction.service.ts`中使用`HttpService`从`axios`。让我们将其导入并注入为交易服务的一个服务。打开`transaction.service.ts`并将代码修改为以下行：

```js
import { Injectable } from '@nestjs/common';
import { CreateTransactionDto } from
  './dto/create-transaction.dto';
import { PrismaService } from 'src/prisma/prisma.service';
import { HttpService } from '@nestjs/axios';
import { AccountApiResponse } from './dto/account.dto';
@Injectable()
export class TransactionService {
  constructor(
    private readonly prisma: PrismaService,
    private readonly httpService: HttpService) {}
//the rest of the code
```

根据我们的业务需求，我们需要从 `createTransactionDto` 中移除状态属性，因为根据账户服务，我们应该在内部定义交易的状态。这就是为什么我们要从 `src/transaction/dto/create-transaction.dto` 中移除状态。以下是该文件的最终版本：

```js
import { IsString, IsOptional, IsNotEmpty,IsUUID } from 
  'class-validator';
export class CreateTransactionDto {
  @IsUUID()
  @IsNotEmpty()
  accountId: string;
  @IsOptional()
  @IsString()
  description?: string;
}
```

太好了！现在，让我们再次打开 `transaction.service.ts` 文件并更改我们的 *创建* 功能。在注入 `httpService` 之后，我们应该能够向任何服务发送请求并获取响应。

这是我们的计划：

+   向账户服务的 `http://url/v1/accounts/{account_id}` 端点发送请求，并根据端点提供的 ID 获取账户信息

+   如果通过 `accountId` 给定的账户不存在，我们将抛出异常

+   如果账户存在，并且其状态是 `'new'` 或 `'active'`，则交易应该以 `'CREATED'` 状态创建

+   如果账户存在，并且其状态既不是 `'new'` 也不是 `'active'`，我们应该创建一个状态为 `'FAILED'` 的交易

就这么简单。打开 `transaction.service.ts` 并使用以下代码行进行更新：

```js
import { Injectable } from '@nestjs/common';
import { CreateTransactionDto } from
  './dto/create-transaction.dto';
import { PrismaService } from 'src/prisma/prisma.service';
import { HttpService } from '@nestjs/axios';
import { AccountApiResponse } from './dto/account.dto';
@Injectable()
export class TransactionService {
  constructor(
    private readonly prisma: PrismaService,
    private readonly httpService: HttpService,
  ) {}
 async create(createTransactionDto: CreateTransactionDto) {
  const { accountId, description } = createTransactionDto;
  let accountApiResponse = await
    this.httpService.axiosRef.get<AccountApiResponse>(
      `http://localhost:3001/v1/accounts/${createTransactionDto.accountId}`,
      );
     const {account} = accountApiResponse.data;
     if (!account) {
      throw new Error('Transaction creation failed: Account
        not found');
    }
      if(account.status == 'new' || account.status ==
        'active')
        {
          return this.prisma.transaction.create({
            data: { accountId, description, status:
              'CREATED' },
          });
        }
        else
        {
          return this.prisma.transaction.create({
            data: { accountId, description, status:
              'FAILED' },
          });
        }
  }
//rest of the functionalities
}
```

我们最后需要做的是在账户服务中配置 **跨源资源共享**（**CORS**）。按照以下步骤操作：

1.  从 VS Code 中打开您的账户服务。

1.  从菜单中选择 **终端** | **新建终端**。

1.  导航到 `src` 文件夹。

1.  执行 `npm install cors` 以安装 `cors` 包。

1.  打开 `app.js` 并在创建 `app` 对象之后添加以下代码：

    ```js
    const corsOptions = {
        origin: 'http://localhost:3001', //(https://your-client-app.com)
        optionsSuccessStatus: 200,
    };
    app.use(cors(corsOptions));
    ```

`app.js` 中的最终代码应如下所示：

```js
const express = require('express');
const v1 = require('./routes/v1');
const cors = require('cors');
const app = express();
//added while implementing transaction service, for Ch06
const corsOptions = {
    origin: 'http://localhost:3001', //(https://your-client-app.com)
    optionsSuccessStatus: 200,
};
app.use(cors(corsOptions));
// service
app.use(express.json());
// V1 API
app.use('/v1', v1);
module.exports = app;
```

在运行我们的应用程序之前，请确保以下内容：

+   您的账户服务位于 `localhost:3001`

+   账户服务在您的表中至少有一条有效的账户信息

+   确保 Docker 正在运行并且 `postgres` 容器处于活动状态

前往我们的账户服务并运行它（更多详情，请参阅*第五章*）。使用 `npm run start:dev` 运行我们新创建的交易服务。打开您喜欢的浏览器，导航到 `http://localhost:3000/api`。打开 `POST /transaction` 并添加以下有效载荷：

```js
{
"accountId": "663fd142ecbdce73baf1ed1a",
 "description": "Optional transaction description"
}
```

您从有效载荷中提供的 `accountId` 值应在账户服务中存在，以便成功操作。在成功请求的情况下，您将收到以下响应：

```js
{
  "id": {your_transaction_id},
  "status": "CREATED",
  "accountId": "663fd142ecbdce73baf1ed1a",
  "description": "Optional transaction description",
  "createdAt": "2024-05-12T17:21:38.727Z",
  "updatedAt": "2024-05-12T17:21:38.727Z"
}
```

如果服务不可用且账户不存在，您将收到错误。我们没有在本章中涵盖异常处理，但我们将在我们即将到来的章节中学习它。

# 摘要

另一次微服务之旅在这里结束。本章的主要内容是创建第二个微服务并在微服务之间建立同步通信。我们以业务需求开始本章。在明确了解我们应该做什么之后，我们开始介绍用于开发事务微服务的主要工具栈。我们没有使用之前开发账户服务时使用的工具。当涉及到开发微服务时，JavaScript 确实拥有丰富的工具和框架。为了展示拥有多个工具的美丽之处，我们使用了 NestJS 以及一些流行的包，如 Prisma 和 Axios。最后，我们使用同步通信模型与一个已经存在的微服务（账户微服务）建立了通信。当然，我们仍然遗漏了很多。我们没有涵盖异常处理、弹性以及我们计划在后续章节中介绍的其他许多有趣话题。

*第七章* 探讨了如何使用 Apache Kafka 和 NestJS 在 JavaScript 微服务中实现异步通信，重点关注构建可扩展的系统、配置 Kafka 以及为事务和账户等服务适配异步消息。
