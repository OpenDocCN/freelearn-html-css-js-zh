

# Sequelize 和 Node.js 中 ORM 的简介

管理数据库驱动程序、管理模式、维护业务流程以及验证数据可能对任何程序员来说都是一项艰巨的任务。随着业务需求的不断变化，将业务逻辑组织到数据库模型中可能会变得繁琐。这通常意味着程序员需要找到所有适用的引用并手动更新查询。这可能对项目和程序员来说都是一项昂贵的操作；如果没有适当的测试，修改可能会导致应用程序中的错误或错误的逻辑，使程序员、业务和客户陷入混乱的状态。

本书将指导您通过在 Node.js 运行时环境中使用 Node.js 运行环境，通过对象关系映射（**ORM**）框架安装、构建、维护、升级、扩展、查询和应用数据库模式的过程。您可以从头到尾顺序阅读本书，如果您更有经验，可以直接阅读您感兴趣的部分章节。由于我们将从头开始创建整个应用程序，因此每个章节都是相互补充的。然而，更有经验的程序员可以在章节之间跳转，理解到他们的数据模型和章节中展示的内容之间可能存在“差距”。无论您的数据结构如何，每个章节中教授的概念和方法都将适用。

本章的目标是帮助您熟悉 Sequelize 是什么以及使用 Sequelize 提供了哪些功能。我们将介绍安装相关库、框架、运行时引擎和**数据库管理系统**（**DBMS**）的必要先决步骤。到本章结束时，您将获得在 Node.js 运行时下使用 Sequelize 从零开始安装、配置和运行应用程序的知识和技能集。

本书的第一章将涵盖以下主题：

+   介绍 Sequelize

+   使用 Sequelize 而不是其他替代方案的优势

+   安装必要的应用程序、框架和工具以帮助您入门

+   在 Express 应用程序中配置 Sequelize

# 技术要求

在我们开始使用 Sequelize 开发应用程序的旅程之前，有一些先决条件。我们需要安装以下内容：

+   一个数据库管理系统，如 MySQL

+   Node.js 运行时库

+   几个 Node.js 包：Sequelize、Express 和一个 MySQL 驱动

# 介绍 Sequelize

**Sequelize**（也称为**SequelizeJS**）是一个 ORM 框架，它帮助将 Node.js 应用程序连接并对应到数据库。Sequelize 自 2010 年起由 Sascha Depold 开发，并在*财富 100 强*公司中得到广泛使用。多年来，该框架在 GitHub 上已拥有近 25,000 个*星标*，超过 900 位贡献者，并被 300,000 多个开源项目使用。Sequelize 在性能和安全方面经过了超过十年的实战考验，即使在一年中流量最高的时段，也为主要零售店和网络机构（如沃尔玛和 Bitnami）提供了无问题的服务。

最初只是一个硕士论文的项目，最终成为了 Node.js 生态系统的一个主要组成部分。

注意

ORM（对象关系映射）是一种使用**面向对象**（**OO**）装饰和模式将数据库结构和信息关联起来的方法。ORM 的目的在于帮助缓解 DBMS 之间的差异，并提供一种形式上的抽象，以便更人性地查询和操作数据。通常，ORM 还会附带一些辅助函数，以帮助管理连接状态、数据预验证和工作流程。

该框架遵循**基于 Promise**的方法，允许程序员异步调用数据。基于 Promise 的方法提供了一种更方便的方式来管理应用程序中返回的值或错误，而无需等待结果立即返回。要了解更多关于 Promise 及其编程方法的信息，请参阅以下链接：[`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)。

什么是异步？

将异步视为一种无需等待响应即可继续执行其他任务的方式。当你给某人发短信时，你不必等待他们的回复就可以继续你的日常生活。发送消息后，你通常不会关注通信，直到你收到有回复或消息发送失败的信号。

目前，Sequelize 支持以下数据库管理系统：MySQL、MariaDB、Postgres、**Microsoft SQL Server**（**MSSQL**）、Snowflake、**Database 2**（**DB2**）和 SQLite。ORM 不仅提供数据库的连接器，通常还提供以下功能：

+   用于迁移模式和数据的工具

+   适配器/插件支持

+   连接池

+   数据预加载

+   管理事务

现在我们已经了解了 Sequelize 是什么以及其基本功能，我们将探讨为什么我们应该使用 Sequelize 这样的 ORM 而不是像**数据访问对象**（**DAOs**）或直接查询数据库这样的替代方法。其中一些有利的特性包括能够在事务中处理和组织查询，或将模式更改迁移到数据库。

# 使用 Sequelize 相比其他替代方案的优势

从您的应用程序查询数据库有许多替代方法。有 ORM、DAO、原始数据库驱动程序等等。每种方法都有其优缺点，并满足不同的编程风格和约定。通常，那些喜欢*约定胜于配置*的人倾向于使用 ORM，而那些喜欢配置的人则倾向于使用 DAO 框架或原始数据库驱动程序。

ORM 可以处理数据验证，类似于 DAO，具有从数据库使用驱动程序读取和写入的附加功能。使用 ORM，您不需要手动管理查询语句，这可能会比 DAO 或原始连接方法节省您的时间。

注意

ORM 并非与 DAO 互斥。您可以将 DAO 视为是显式的，而不是隐式和假设性的。DAO 只提供数据的一个*接口*。它不涉及您如何/在哪里读取或写入数据（数据库驱动程序），也不会关心数据完整性，除非应用程序在 DAO 范围之外手动调用某种形式的数据验证。

当使用如 Sequelize 这样的 ORM 时，您将获得以下功能而无需任何额外代码：

+   事务处理

+   连接池

+   模型/数据验证

+   数据完整性（超出 DBMS 的**外键**（**FKs**）、唯一约束等范围）

+   预加载

+   图形迁移和级联

+   乐观锁定

使用 DAO 或原始数据库驱动程序将放弃这些功能，您将不得不自己构建这些解决方案。使用如 Sequelize 这样的 ORM 将帮助您更高效、更有效地构建项目。

到目前为止，我们已经涵盖了 Sequelize 的*是什么*和*为什么*；现在，我们将介绍安装应用程序所需必要先决条件的*如何*。

# 安装必要的应用程序、框架和工具以帮助您开始

我们的应用程序将要求客户从集中式来源查看信息，并且我们需要捕获他们输入到我们数据库中的信息。通常，客户可以通过安装在他们机器上的应用程序查看您的产品/服务，或者他们可以使用浏览器访问我们的网站。Node.js 是构建网络应用程序的一个好选择，因为本书将围绕它展开，这得益于其**中央处理单元**（**CPU**）限制以及在前端开发（向最终用户显示的内容）和后端开发（最终用户看不到但仍然调用的内容）之间轻松切换上下文的能力，因为 Node.js 是 JavaScript。为了开始，我们需要安装以下应用程序/程序：

+   DBMS（我们将安装 MySQL）

+   Node.js 运行时

+   Sequelize 和 Express

## 安装 MySQL

下一个部分将介绍在三种不同的操作系统发行版上安装 MySQL 的过程：Microsoft Windows、macOS 和 Linux。MySQL 被选择是因为其安装简单（无需配置或**访问控制列表**（**ACLs**））。不要让这些点阻止您使用不同的数据库。就大部分而言，Sequelize 应该能够优雅地将一个 DBMS 转换为另一个，本书的大部分内容将使用通用/标准的**结构化查询语言**（**SQL**）方法。

### Windows

Microsoft Windows 的 MySQL 安装程序可在此处找到：

[`dev.mysql.com/downloads/mysql/5.7.xhtml`](https://dev.mysql.com/downloads/mysql/5.7.xhtml)

注意

默认版本为 `8.0.26`。本书使用版本 `5.7`，但只要 Node.js MySQL 驱动程序与该版本兼容，MySQL 的其他版本也应能适当工作。

下载并打开安装程序应用程序后，您将看到**选择安装类型**屏幕。我们将想要选择**开发者默认**和**安装所有产品**选项，如图所示：

![图 1.1 – Windows MySQL 安装程序：选择安装类型](img/Figure_1.1_B17841.jpg)

图 1.1 – Windows MySQL 安装程序：选择安装类型

如果您的计算机上已安装 Python 或 Visual Studio，您可能会遇到一个**检查要求**步骤（见 *图 1.2*）。如果您使用 Visual Studio 作为您的**集成开发环境**（**IDE**），则可以安装必要的软件产品，但这不是必需的。在整个项目过程中，您可能会遇到用 Python 编写的工具，这些工具与您的数据库进行交互（例如，大多数与数据科学相关的库/框架）。通过选择以下屏幕截图所示的**Connector/Python**选项，我们可以避免未来可能出现的潜在问题：

![图 1.2 – Windows MySQL 安装程序：检查要求](img/Figure_1.2_B17841.jpg)

图 1.2 – Windows MySQL 安装程序：检查要求

下一个步骤应该是**下载**步骤。本书内容所需的主要产品列在这里：

+   **MySQL 服务器**

+   **MySQL Workbench**（用于数据库的**图形用户界面**（**GUI**））

+   **MySQL Shell**

您可以在以下屏幕截图中看到上述产品：

![图 1.3 – Windows MySQL 安装程序：下载](img/Figure_1.3_B17841.jpg)

图 1.3 – Windows MySQL 安装程序：下载

注意

如果您是 MySQL 的新用户，下载**MySQL 文档**和**示例和示例**包可能是个好主意。

在我们完成下载我们的软件包后，我们将为每个适用的选定产品（例如，**MySQL 服务器** 和 **示例和示例**）输入我们的配置详细信息。对于大多数配置设置，我们将使用默认值；然而，将有一些步骤需要您的干预。您可以在以下屏幕截图中查看此概述：

![图 1.4 – Windows MySQL 安装程序：类型和网络](img/Figure_1.4_B17841.jpg)

图 1.4 – Windows MySQL 安装程序：类型和网络

从 **MySQL 服务器** 配置向导中，我们希望以下设置（如图 *图 1.4* 所示）：

+   **配置类型**：**开发计算机**

+   **TCP/IP**：已勾选

+   `3306`

+   **打开 Windows 防火墙端口以进行网络访问**：可选

MySQL 服务器配置步骤的下一部分是声明您的 MySQL 根密码和用户账户。请确保将此信息保存在安全的地方，以防在项目过程中遇到管理问题。如果您忘记了 MySQL 根密码，有几种方法可以重置密码，如以下所述：[`dev.mysql.com/doc/mysql-windows-excerpt/5.7/en/resetting-permissions-windows.xhtml`](https://dev.mysql.com/doc/mysql-windows-excerpt/5.7/en/resetting-permissions-windows.xhtml)。

对于设置具有角色的 MySQL 用户账户，您将看到以下 **账户和角色** 屏幕：

![图 1.5 – Windows MySQL 安装程序：账户和角色](img/Figure_1.5_B17841.jpg)

图 1.5 – Windows MySQL 安装程序：账户和角色

在 **MySQL 用户账户** 部分中，您需要点击 **添加用户** 按钮（如图 *图 1.5* 所示，位于窗口右侧附近）并输入一个您将记住的用户名和密码，以便我们在初始化 Node.js 应用程序时使用。当您完成添加适当的根密码和 MySQL 用户账户后，我们可以继续下一步。

接下来，安装过程将提供一个 **将 MySQL 服务器配置为 Windows 服务** 选项，如图中所示。**Windows 服务** 是一个 **进程控制系统**（**PCS**），它还将编排后台进程（在 Unix/Linux 世界中，这些被称为 *daemons*）：

![图 1.6 – Windows MySQL 安装程序：Windows 服务](img/Figure_1.6_B17841.jpg)

图 1.6 – Windows MySQL 安装程序：Windows 服务

我们希望确保以下参数已配置（如图 *图 1.6* 所示）：

+   **将 MySQL 服务器配置为 Windows 服务**：已勾选

+   **在系统启动时启动 MySQL 服务器**：已勾选

+   **在 **运行 Windows 服务作为...** 部分下选择 **标准系统账户**

点击 **下一步 >** 以应用 MySQL 服务器的配置。如果您之前选择了要安装的附加软件包，您可能会遇到额外的屏幕，要求您提供更多的配置设置和参数。

备注

如果您在上一节中选择了**MySQL 路由器**包，安装过程将要求您提供有关您如何设置集群环境的信息。除非您是数据库管理员或您正在设置生产环境，否则不建议安装此包。只需取消选择**为 InnoDB 集群启动 MySQL 路由器**选项，然后点击**完成**以在不安装 MySQL 集群环境的情况下继续。

如果选择了安装**示例和示例**包，我们将看到一个屏幕，允许我们输入我们的 MySQL 用户名和密码。您可以使用您的**root 凭证**作为用户名和密码输入字段的输入，并点击**下一步 >**按钮继续。屏幕的概述如下截图所示：

![图 1.7 – Windows MySQL 安装程序：连接到服务器](img/Figure_1.7_B17841.jpg)

图 1.7 – Windows MySQL 安装程序：连接到服务器

### macOS

在 macOS 上安装 MySQL 有几种方法。第一种是从**磁盘镜像**（**DMG**）文件下载并安装 MySQL，另一种方法是使用包管理器，如 Homebrew。我们将探讨这两种选项。

#### 从磁盘镜像安装

您可以从以下 URL 找到适当的磁盘镜像：[`dev.mysql.com/downloads/mysql/`](https://dev.mysql.com/downloads/mysql/)（x86 用于英特尔 CPU，**高级精简指令集**（**ARM**）用于 M1 CPU）。

注意

如果您找不到 MySQL 的 5.7 版本，您可以从 MySQL 的存档链接中找到适当的 DMG 文件：[`downloads.mysql.com/archives/community`](https://downloads.mysql.com/archives/community)。

然而，macOS 安装包可能无法下载最新 5.7 版本。在撰写本书时，版本 5.7.34、5.7.33 和 5.7.32 作为 DMG 包不可用（5.7.31 可以下载）。任何适用的 5.7 版本都应与本书的说明和安装程序兼容。

如果在安装过程中被要求安装偏好面板，我们建议您这样做。否则，我们需要查阅位于[`dev.mysql.com/doc/refman/5.7/en/macos-installation-launchd.xhtml`](https://dev.mysql.com/doc/refman/5.7/en/macos-installation-launchd.xhtml)的**安装 MySQL 启动守护进程**页面。

下载并打开 DMG 文件后，我们将想要打开**包**（**pkg**）文件，这将启动我们的安装过程。根据您的 macOS 版本，您可能会看到一个**“[包名]”无法打开，因为苹果无法检查其恶意软件**的屏幕，如下所示：

![图 1.8 – 苹果无法识别该包的恶意性](img/Figure_1.8_B17841.jpg)

图 1.8 – 苹果无法识别该包的恶意性

如果这种情况适用于你，请转到**苹果** | **安全与隐私**，窗口中应该在**“mysql….pkg”由于不是来自已识别的开发者而被阻止使用**旁边有一个**“无论如何打开”**按钮，如下面的屏幕截图所示：

![图 1.9 – 绕过未识别的软件包安装](img/Figure_1.9_B17841.jpg)

图 1.9 – 绕过未识别的软件包安装

一旦安装包再次打开，你可能会收到来自苹果的另一个警告。点击**打开**以继续安装过程。在继续并阅读**软件许可协议**（**SLA**）后，你可以选择默认的安装位置。点击**安装**可能会提示输入你的管理员密码，如下面的屏幕截图所示：

![图 1.10 – MySQL 安装请求管理员权限](img/Figure_1.10_B17841.jpg)

图 1.10 – MySQL 安装请求管理员权限

一旦 MySQL 安装程序完成，将出现一个带有临时密码的警告对话框。以下是一个示例。确保在登录 MySQL 服务器时记下临时密码：

![图 1.11 – MySQL 安装提供临时 root 密码](img/Figure_1.11_B17841.jpg)

图 1.11 – MySQL 安装提供临时 root 密码

#### 从 Homebrew 安装

使用 Homebrew 而不是传统的软件包安装程序可以帮助你保持软件包更新，无需手动干预，同时验证软件包安装和二进制文件。要通过 Homebrew 安装 MySQL，我们需要在我们的本地机器上安装 Homebrew。在终端（位于**应用程序** > **实用工具**）中，只需输入以下命令：

```js
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

注意

在运行外部脚本命令之前，总是检查脚本内容是一个好主意。一个网页可以重定向到任何地方，包括可能导致数据泄露或更恶劣的恶意脚本。

在安装 Homebrew 时，你可能会遇到以下消息：

```js
==> Checking for `sudo` access (which may request your password)
```

你可以在下面的屏幕截图中看到这个示例：

![图 1.12 – 在 macOS 上安装 Homebrew](img/Figure_1.12_B17841.jpg)

图 1.12 – 在 macOS 上安装 Homebrew

你可以在这里输入密码，或者在安装 Homebrew 之前运行`sudo <anything>`（例如，`sudo ls`），输入密码，然后运行安装命令。用户在继续之前必须具有管理员访问权限。

对于这本书，我们将安装 MySQL 版本 5.7。如前所述，MySQL 的其他版本应该与本书的代码库兼容。要明确安装 5.7 版本，请运行以下命令：

```js
brew install mysql@5.7
```

为了正确设置你的实例，可能需要运行额外的步骤和命令，如下面的屏幕截图所示。本书的内容不需要库/头文件进行编译，也不需要配置 `pkg-config`。一般来说，建议运行 `mysql_secure_installation` 并按照提示添加根密码，但这不是必需的：

![图 1.13 – 在 macOS 上使用 Homebrew 安装 MySQL](img/Figure_1.13_B17841.jpg)

图 1.13 – 在 macOS 上使用 Homebrew 安装 MySQL

接下来，我们需要一种管理我们的 MySQL 服务的方法。这里概述了两种可供选择的方法：

+   手动创建启动守护进程配置文件。有关如何实现此操作的更多信息，请参阅此处：[`dev.mysql.com/doc/refman/5.7/en/macos-installation-launchd.xhtml`](https://dev.mysql.com/doc/refman/5.7/en/macos-installation-launchd.xhtml)。

+   我们可以使用一个名为 `services` 的 Homebrew 扩展，通过执行以下命令来自动管理启动配置：

    ```js
    brew tap homebrew/services
    ```

为了启动 MySQL 服务，我们需要运行以下命令：

```js
brew services start mysql@5.7
```

如果你更喜欢使用图形界面来管理你的服务，有一个名为 `brew-services-menubar` 的应用程序可以通过 Homebrew 的 Cask 扩展安装，如下面的代码片段所示：

```js
brew install --cask brewservicesmenubar
```

注意

如果你更喜欢在交互或查询数据库时使用图形界面，有一个名为 Sequel Pro 的免费应用程序可供下载，链接如下：

[`www.sequelpro.com/`](http://www.sequelpro.com/)

### Linux

Linux 有许多发行版；对于本书，我们将使用 Ubuntu（任何 Debian 发行版都应适用，使用相同的命令）。如果你使用的是不同的发行版，请参考此页面获取如何为你的操作系统安装 MySQL 的说明：[`dev.mysql.com/doc/refman/5.7/en/linux-installation.xhtml`](https://dev.mysql.com/doc/refman/5.7/en/linux-installation.xhtml)。

在终端中运行以下命令（这些命令也在下面的屏幕截图中有显示）：

```js
sudo apt-get update
sudo apt install mysql-server
```

![图 1.14 – 在 Ubuntu 上安装 MySQL 服务器](img/Figure_1.14_B17841.jpg)

图 1.14 – 在 Ubuntu 上安装 MySQL 服务器

MySQL 安装完成后，我们需要初始化一个数据库来存储我们所有模型的结构和信息。一些 ORM 和 DBMS 将数据库称为“模式”（不要与模型模式混淆，在 Sequelize 中被称为“属性”）。

## 创建数据库

现在我们已经在我们本地机器上完成了 MySQL 数据库管理系统引擎的安装，我们可以开始创建一些表来构建数据库。在创建表之前，我们需要了解 MySQL 的各种引擎类型。幸运的是，以下内容适用于所有操作系统：

默认情况下，MySQL 将创建 InnoDB 数据库类型（或在 MySQL 术语中，引擎）。数据库引擎与 MySQL 中的数据库表相关联（而不是整个数据库）。当你知道在无约束的读取密集型表（例如，新闻文章）和写入密集型表（例如，聊天室）之间的权衡时，这非常有用。为了简洁起见，我们将简要介绍以下三个主要数据库引擎：

+   **InnoDB**：一个具有事务查询和 FK 支持的数据库引擎。事务查询对于执行一个查询，或多个查询，并保证原子性非常有用。我们将在后面的章节中进一步详细介绍事务和 FK。

+   **MyISAM**：如果你的数据库操作主要是读取相关且你不需要任何数据约束，那么这将是一个首选的数据库引擎。

+   **HEAP**：这些表中的数据存储在机器的内存中。如果你需要快速查询临时数据，这个数据库引擎非常有用。MySQL 不会为你管理内存分配，因此记住在不再使用表时删除它们（并且数据适合机器的可用内存）非常重要。

注意

你可以通过在 MySQL 客户端中输入以下命令来检查你本地 MySQL 服务器的默认引擎类型：`SELECT @@default_storage_engine;`。

你可以跳过这一节并使用 Sequelize 的 `db:create` 命令，只要相应的 MySQL 用户具有适当的权限。为了熟悉终端，我们将使用命令行创建数据库，如下一截图所示。

使用以下命令登录到 MySQL 服务器（你可能需要输入密码，或者需要额外的 `-p` 参数来输入密码）：

```js
mysql --user=root
```

我们可以在 MySQL 客户端命令提示符中执行以下 SQL 命令来创建我们的数据库：

```js
CREATE DATABASE airline;
```

对于 Windows 用户

大多数这些命令都可以通过命令提示符或 PowerShell 应用程序执行。这些应用程序可以通过 **开始** 菜单访问（例如，**开始** > **所有程序** > **附件** > **Windows PowerShell**）。

![图 1.15 – 创建数据库](img/Figure_1.15_B17841.jpg)

图 1.15 – 创建数据库

如果你使用的是 Windows 机器，你可以使用任何你选择的终端应用程序（命令提示符、PowerShell 等），或者你可以使用 MySQL Workbench，如下面的截图所示，我们在上一节中已经安装了它：

![图 1.16 – MySQL Workbench：创建数据库](img/Figure_1.16_B17841.jpg)

图 1.16 – MySQL Workbench：创建数据库

注意

要使用 MySQL Workbench 执行查询，查询工具栏中有一个 *闪电* 图标（图标通常在 *保存* 图标旁边）。你的查询结果将出现在屏幕底部的 **输出** 部分。

## 安装 Node.js

在撰写本书时，Node.js 的**长期支持**（**LTS**）版本是 16。在本书中，我们将使用这个版本的 Node.js，但代码库应该仍然可以使用其他版本无问题执行。所有相应的操作系统安装的 Node.js 都可以在这里找到：[`nodejs.org/en/download/`](https://nodejs.org/en/download/)。

注意

如果 Node.js 的长期支持（LTS）版本不再是 16，而您想使用本书中相同的版本，您可以从这里下载以前的 Node.js 版本：[`nodejs.org/en/download/releases/`](https://nodejs.org/en/download/releases/)。

对于在一台机器上管理多个 Node.js 版本，有一个名为**Node 版本管理器**（**NVM**）的应用程序可以处理和维护同一台机器上的多个 Node.js 版本。有关更多信息，您可以访问他们的仓库：[`github.com/nvm-sh/nvm`](https://github.com/nvm-sh/nvm)。

### Windows

在下载并打开 Node.js Windows 安装程序后，我们将看到一个如下所示的屏幕提示：

![图 1.17 – Windows Node.js 安装程序：目标文件夹](img/Figure_1.17_B17841.jpg)

图 1.17 – Windows Node.js 安装程序：目标文件夹

点击**下一步**将带我们进入安装的**自定义设置**步骤。请确保您正在安装/配置以下内容：

+   **Node.js 运行时**

+   **npm 包管理器**

+   **添加到 PATH**

您可以在此处查看此屏幕的概述：

![图 1.18 – Windows Node.js 安装程序：自定义设置](img/Figure_1.18_B17841.jpg)

图 1.18 – Windows Node.js 安装程序：自定义设置

在完成**自定义设置**步骤后，我们将进入**原生模块工具**部分。默认情况下，安装必要工具的复选框未勾选。为了开发目的，我们希望确保自动安装选项已勾选，如下一截图所示：

![图 1.19 – Windows Node.js 安装程序：原生模块工具](img/Figure_1.19_B17841.jpg)

图 1.19 – Windows Node.js 安装程序：原生模块工具

选择自动工具安装将弹出一个 PowerShell 窗口，如下一张截图所示，显示 Chocolatey、.NET 包、Python 依赖项等的安装进度。

![图 1.20 – Windows Node.js 安装：附加工具](img/Figure_1.20_B17841.jpg)

图 1.20 – Windows Node.js 安装：附加工具

注意

Chocolatey 是 Microsoft Windows 操作系统的包管理器。如果您熟悉 macOS 环境，这类似于 Debian Linux 发行版上的 Homebrew 或 Apt。有关 Chocolatey 的更多信息，请参阅以下链接：[`chocolatey.org/`](https://chocolatey.org/)。

### macOS

您可以通过其包镜像安装 macOS 上的 Node.js，该镜像位于[`nodejs.org/en/download/`](https://nodejs.org/en/download/)，或者您可以通过运行以下命令使用 Homebrew 安装它：

```js
brew install node@16
```

为了确认您的机器正在使用正确的“node”二进制文件，我们可以始终通过运行以下命令来检查版本：

```js
node -v
```

### Linux

对于 Ubuntu/Debian Linux 发行版，我们可以使用特定的仓库来安装 Node.js 14，如下面的代码片段所示：

```js
sudo apt update
curl -sL https://deb.nodesource.com/setup_14.x | sudo bash -
```

在添加了仓库之后，我们可以安装 Node.js 并检查版本，如下所示：

```js
sudo apt -y install nodejs
node -v
```

到目前为止，我们已经完成了 MySQL 作为我们的数据库管理系统（DBMS）、适用的包管理器和 Node.js 运行时库的安装；现在我们可以开始搭建我们的项目，并安装 Sequelize 和 Express 所需的 Node.js 包。

# 在 Express 应用程序中配置 Sequelize

在我们安装了我们的开发工具和数据库之后，我们可以开始使用 Sequelize 和**Express**安装和配置我们的应用程序。Express 是一个针对 Node.js 运行时应用程序的最小化网络框架。我们的 Node.js 应用程序将使用 Sequelize 与数据库进行通信，Express 将把查询结果传递到浏览器。有关 Express 的更多信息以及完整的参考，请参阅此处：[`expressjs.com`](https://expressjs.com)。

在命令提示符、PowerShell 或终端中，输入以下命令以初始化我们的项目：

```js
mkdir airline
cd airline
npm init -y
```

这将创建一个名为`airline`的目录；然后，我们将工作目录更改为`airline`文件夹，并从`npm`命令运行初始化脚本将创建一个`package.json`文件，该文件包含 npm 在此项目上使用的裸配置。之后，我们需要安装我们应用程序所需的最小 Node.js 模块，如下所示：

```js
npm install express sequelize mysql2
```

这里有一个在线资源，您可以参考它以获取 npm 的完整选项列表：

[`docs.npmjs.com/cli/v7/commands`](https://docs.npmjs.com/cli/v7/commands)

Sequelize 有一个伴随的可执行文件，可以帮助我们初始化项目、管理模式更新和处理数据库迁移。我们可以在我们的用户空间中将其作为全局（`--location=global`）二进制文件安装，通过在终端中输入以下命令：

```js
npm install --location=global sequelize
```

对于可用的完整命令列表，CLI 内置了文档，可以使用`-h`或`--help`标志来暴露，如下面的截图所示：

![图 1.21 – Sequelize CLI 安装和帮助指南](img/Figure_1.21_B17841.jpg)

图 1.21 – Sequelize CLI 安装和帮助指南

下一步是初始化 Sequelize 为我们提供的通用模板。这将生成用于配置、迁移、种子和模型文件的几个目录。以下是执行此操作的代码：

```js
sequelize init
```

以下列表简要说明了 CLI 在我们项目目录中创建的目录：

+   `config`：一个包含数据库连接配置文件的目录，`sequelize-cli` 工具使用此配置文件来迁移模式和数据文件，但这些配置设置也可以用于我们的 Node.js 应用程序。

+   `migrations`：一个包含 Node.js 文件的目录，其中包含 Sequelize 的指令，用于构建数据库的模式和结构。

+   `models`：包含 Sequelize 模式定义的 Node.js 文件集合。

+   `seeders`：类似于 `migrations` 目录，但不是定义我们的数据库模式，而是定义我们的数据库数据。

现在我们已经建立了应用程序的初始基础，我们可以编辑位于 `config/config.json` 中的 Sequelize 配置文件。根据您遵循的安装说明，用户名和密码值可能与本书的代码库不同。代码如下所示：

```js
{
  "development": {
    "username": "root",
    "password": null,
    "database": "airline",
    "host": "127.0.0.1",
    "dialect": "mysql"
  },
  "test": {
    "username": "root",
    "password": null,
    "database": "airline",
    "host": "127.0.0.1",
    "dialect": "mysql"
  },
  "production": {
    "username": "root",
    "password": null,
    "database": "airline",
    "host": "127.0.0.1",
    "dialect": "mysql"
  }
} 
```

如果您不想将用户名和密码保存在文件中（这对于生产环境或版本控制仓库来说是个好主意），配置文件有另一种形式，可以接受一个带有连接 `mysql://root:password@127.0.0.1:3306/airline` 的环境键，如下代码片段所示：

```js
{
  "development": {
    "use_env_variable": "DB_DEV_ENV"
  },
  "test": {
    "use_env_variable": "DB_TEST_ENV"
  },
  "production": {
    "use_env_variable": "DB_PRODUCTION_ENV"
  }
}
```

如果我们想使用 `development` 配置，我们的 Node.js 应用程序将知道从名为 `DB_DEV_ENV` 的环境变量中查找连接参数/URI（您可以使用相同的环境变量用于任何阶段）。有关 Sequelize CLI 的更多选项和配置设置，请参阅此资源：[`github.com/sequelize/cli/blob/master/docs/README.md`](https://github.com/sequelize/cli/blob/master/docs/README.md)。

注意

您可以通过设置 `NODE_ENV` 环境变量在您希望应用程序所处的环境之间切换。默认值是 `development`，但如果我们想使用我们的 `production` 环境，我们可以这样设置环境：`NODE_ENV=production`。

## 将 Sequelize 与 Express 连接

我们现在可以开始构建我们的 Node.js 应用程序，通过在项目目录中创建一个 `index.js` 文件并在我们选择的 IDE 中打开该文件。让我们先输入以下代码：

```js
const express = require("express");
const app = express();
const models = require("./models");
models.sequelize.sync().then(function () {
    console.log("> database has been synced");
    }).catch(function (err) {
    console.log(" > there was an issue synchronizing the
                    database", err);
});
app.get('/', function (req, res) {
    res.send("Welcome to Avalon Airlines!");
});
app.listen(3000, function () {
    console.log("> express server has started");
});
```

我们首先通过代码的前两行声明我们的 Express/web 应用程序变量（`express` 和 `app`）。下一行是调用之前由 Sequelize CLI 创建的 `./models/index.js` 文件的快捷方式（我们将在下一章详细介绍该文件）。下一行运行 Sequelize 的 `sync()` 命令，该命令将通过创建必要的表、索引等来同步您的模型定义与数据库。它还将建立关联/关系，执行与同步相关的钩子/事件等。

`sync()` 命令提供了几个选项，这些选项作为第一个参数封装在一个对象中，如下所示：

+   `force`：一个布尔值，将在重新创建之前删除你的数据库表。

+   `match`：生产环境中的一个 `force` 选项。

+   `logging`：一个布尔值或函数值。`true`（默认值）在执行查询时使用 `console.log` 进行日志记录。`false` 将完全禁用，并且可以使用函数将日志和上下文发送到另一个适配器。本书将在后面的章节中详细介绍此选项。

+   `schema`：一个字符串值，用于定义要操作的数据库。当使用允许通过数据库（MySQL 称为“模式”）以及命名空间（Postgres 称为“模式”）来分离表的数据管理系统（如 Postgres）时很有用。

+   `searchPath`：一个字符串值，用于定义仅适用于 Postgres 数据库的默认 `search_path`。此选项与本书的代码库或内容无关。

+   `hooks`：一个布尔值（默认为 `true`），用于执行与同步事件相关的多个钩子/事件（`beforeSync`、`afterSync`、`beforeBulkSync` 和 `afterBulkSync`）。设置为 `false` 将禁用事件的执行。

+   `alter`：一个具有以下参数的对象：

    +   `drop`：一个布尔值，当 Sequelize 需要在数据库中运行 `ALTER` 命令时，防止执行任何 `drop` 语句。

你可以像这样定义这些选项：

```js
models.sequelize.sync({
  force: true,
  logging: false
})
```

注意

Sequelize 社区不建议在生产环境中将 `force` 选项设置为 `true`。这可能会产生意外的后果，例如删除重要的客户/用户信息。`force` 选项用于当你还在原型化应用程序时，并希望每次迭代都从零开始启动应用程序。

下一个命令，`app.get(...)`，指示 Express 框架将我们的 web 应用程序的根路径 `"/"` 路由到作用域函数（在这种情况下，我们向浏览器发送文本，如 *图 1.22* 所示）。之后，我们通过调用 `app.listen(...)` 启动 Express 服务器，这将告诉我们的应用程序监听 `3000` 端口，可以通过 `http://localhost:3000` 或 `http://127.0.0.1:3000` 访问，具体取决于你的网络接口设置。要启动我们的应用程序，我们可以在终端/PowerShell 中运行以下命令：

```js
node index.js
```

你应该在屏幕上看到以下显示的文本：

+   Express 已启动

+   执行了一个 SQL 查询

+   数据库已同步

注意

Sequelize 将自动执行一个 `SELECT 1+1 AS result` 查询，作为检查数据库连接健康状态的方法。并非所有数据库管理系统（DBMS）都提供发送 `ping` 数据包来检查连接是否成功的方法。

现在，当你打开浏览器并访问之前提到的 URL 时，你应该看到与这里显示的页面类似的页面：

![图 1.22 – 欢迎页面](img/Figure_1.22_B17841.jpg)

图 1.22 – 欢迎页面

每次我们对应用程序进行更改时，我们都需要在终端中终止当前进程（*Ctrl* + *C*）。这将向进程发送`SIGINT`信号，该信号将向进程发送中断信号，以便开始清理然后退出/停止。为了避免每次更改后都需要手动重新启动我们的进程，我们可以安装一个名为 Nodemon 的独立进程来帮助我们完成这项工作（更多信息请见：[`nodemon.io/`](https://nodemon.io/))。

可以通过运行以下命令将 Nodemon 安装为全局二进制文件：

```js
npm install -g nodemon
```

您可以通过输入以下内容来确认安装是否成功：

```js
nodemon index.js
```

这应该会同时启动我们的 Node.js 应用程序并监视我们项目目录中的更改文件。一旦我们对项目进行了修改，我们应该看到 Nodemon 自动重启我们的进程，如下面的截图所示：

![图 1.23 – Nodemon 自动重启应用程序](img/Figure_1.23_B17841.jpg)

图 1.23 – Nodemon 自动重启应用程序

本章的最后一步是对我们的`package.json`文件进行一些调整，如下所示：

+   在`"name": "airline,"`行下添加`"private": true`。这个调整将防止我们（或团队中的任何人）将我们的项目发布到公共 npm 注册表。

+   查找`scripts`对象，并将其中内容替换为`"start": "nodemon index.js"`。这样我们就可以通过运行以下命令来启动我们的应用程序：

    ```js
    npm run start
    ```

最终的`package.json`文件应类似于以下内容：

```js
{
  "name": "airline",
  "private": true,
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "start": "nodemon index.js"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "express": "⁴.17.1",
    "mysql2": "².3.0",
    "sequelize": "⁶.6.5"
  }
}
```

# 摘要

在本章中，我们介绍了使用 ORM 的好处以及 Sequelize 能提供什么。我们学习了如何设置我们的开发/本地环境以运行 DBMS（MySQL）和 Node.js 运行时。然后我们使用 npm 和 Sequelize CLI 构建了一个项目，并将 Sequelize 库与 Express 网络框架集成。

在下一章中，我们将开始向我们的数据库中插入数据并定义 Sequelize 模型。
