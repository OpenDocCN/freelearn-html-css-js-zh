# 第六章。部署

本章将介绍使我们的网络应用程序上线所需的步骤。您还将学习如何设置 SSL 证书以及如何积极跟踪生产中发生的错误。本章将涵盖以下主题：

+   设置 Modulus.io

+   设置 Compose.io

+   自动错误跟踪

+   设置 SSL 证书

# 设置 Modulus

Modulus 目前是托管 Meteor 项目的最佳场所。为什么？它易于设置和维护。虽然还有其他几个服务可以提供相同的结果，但每个都需要更多的服务器专业知识，并且您需要投入相当多的时间。我们不是在开发服务器。我们是在开发应用程序。

Modulus.io 提供了对粘性会话、WebSockets 和免费 SSL 端点的支持。

请记住，Meteor 的 Galaxy 主机服务即将发布，并将无疑成为托管 Meteor 网络应用程序的最佳场所。在此之前，Modulus 是最佳选择。

让我们从在 [`modulus.io`](http://modulus.io) 创建一个免费账户开始：

![设置 Modulus](img/00005.jpeg)

接下来，您需要安装 `demeteorizer` 工具和 `modulus CLI` 工具。在您的终端中运行以下命令：

```js
npm install –g demeteorizer
npm install –g modulus

```

这没有工作！如果您的 `npm install` 命令失败，那么您需要安装 `node` 和 `npm`。

为了确保我们正确安装 `npm`，我们首先需要添加 `homebrew`。通过运行以下命令进行安装：

```js
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"

```

完成此操作后，运行以下命令：

```js
brew install node

```

当这个命令完成所有安装后，我们将在命令行中全局可用 `node` 和 `npm`。

现在，我们可以轻松安装 `demeteorizer` 和 `modulus`：

```js
npm install –g demeteorizer
npm install –g modulus

```

优秀。现在我们拥有了所有需要的工具。让我们创建一个新的项目（如果您想感觉像专业人士，您也可以使用 CLI）：

![设置 Modulus](img/00006.jpeg)

确保选择项目的运行环境为 **Node.JS** 并将服务器的内存大小设置为 **192MB**。如果您发现有很多流量进入，您可以在任何时候增加服务器的内存以轻松扩展应用程序。

![设置 Modulus](img/00007.jpeg)

创建您的服务器后，转到 **管理** 选项卡：

![设置 Modulus](img/00008.jpeg)

现在复制位于 **您的项目 URL** 文本旁边的项目生成的 URL：

![设置 Modulus](img/00009.jpeg)

将页面滚动到最底部的 **环境变量**。我们需要做的第一件事是设置 `ROOT_URL` 环境变量。在此粘贴项目 URL。确保它具有 `https` 协议。在这里，我们正在利用 Modulus 的安全 SSL 端点。`wizonesolutions:canonical` 包将确保所有路由都将击中我们的 `ROOT_URL`：

![设置 Modulus](img/00010.jpeg)

接下来，我们需要设置我们的数据库。我们无法部署此应用程序，因为应用程序不知道它应该在哪里保存信息。

# 设置 Compose

尽管 Modulus.io 提供 MongoDB 托管服务，但 Modulus 并不提供对 Meteor 的**oplog tailing**功能的访问权限，也没有对多个副本集的支持。这两个功能对于生产环境来说是必不可少的，原因如下。

MongoDB 副本集是您数据库的精确副本。当您在 Compose 中首次创建数据库时，您会自动获得一个主副本集和一个辅助副本集来支持它。由于故障是不可避免的，辅助副本集的存在是为了在主副本集失败时立即替换它。在数据库领域，这被称为数据冗余。

这层安全性带来了一个有趣的问题；辅助数据库如何知道主数据库正在发生的变化？操作日志，简称 oplog，是 Mongo 用于记录对数据库所做的所有更改的特殊集合。这些更改由副本集读取，以反映所有必要的更改。

太好了，现在让我们了解 Meteor 如何使用 oplog。Meteor 默认的`poll`和`diff`方法用于监视数据库中的更改速度较慢。它通过每 10 秒比较数据库和客户端之间的更改来实现。这默认会在您的数据库上创建多次不必要的访问，并且速度不快（因为更改可能在 Meteor 寻找更改之前 9 秒发生）。为了使 Meteor 表现更好，Meteor 团队利用了 Mongo 的 oplog。通过监听 Mongo 的 oplog 中的更改，Meteor 可以确切地知道何时以及哪些更改需要推送到客户端。这被称为 oplog tailing。

Oplog tailing 通过有效地跟踪 Mongo 的操作日志，极大地提高了 Meteor 的反应性能。保证没有利用这个功能的实际生产应用将无法顺利运行。

使用`compose.io`创建您的账户：

![设置 Compose](img/00011.jpeg)

现在创建一个新的 MongoDB：

![设置 Compose](img/00012.jpeg)

现在点击**用户**标签页。在这里，我们需要添加一个可以访问数据库和 oplog 的用户。这就是 Modulus 将使用的用户。在以下示例中，我们将用户设置为`root`，密码为`root`，并将`oplog access`设置为`true`：

![设置 Compose](img/00013.jpeg)

点击**管理**标签页。复制显示在**副本集 URI**下的数据库 URI：

![设置 Compose](img/00014.jpeg)

注意字符串中的`<user>`和`<password>`部分。我们将用之前定义的用户凭据替换这些字段。

返回 Modulus.io 并访问项目的**管理**页面。滚动到**环境变量**部分。在这里，我们需要将刚刚复制的 URI 添加到两个不同的变量中：`MONGO_URL`和`MONGO_OPLOG_URL`。

`MONGO_URL`将看起来像这样：

```js
mongodb://root:root@candidate.1.mongolayer.com:10197,candidate.13.mongolayer.com:10810/online_shop
```

注意您可以删除查询参数（问号及其之后的内容）。

`MONGO_OPLOG_URL`将看起来像这样：

```js
mongodb://root:root@candidate.1.mongolayer.com:10197,candidate.13.mongolayer.com:10810/local?authSource=online_shop
```

注意，我们在尾随斜杠之后修改了信息，将其修改为 `/local?authSource=online_shop`，其中 `online_shop` 是数据库名称。

极好！这是我们最后一次为项目部署配置任何内容。现在我们可以使用 `modulus` CLI 进行部署。

前往您的终端并运行以下命令以登录到您的用户：

```js
modulus login

```

如果您使用 GitHub 账户创建您的 Modulus 账户，那么请传递 `github` 标志：

```js
modulus login --github

```

按照命令行的说明操作。一旦登录，请确保您位于 Meteor 项目的 `root` 目录（运行 Meteor 命令的地方）。从这里，运行以下命令以部署到您的新服务器：

```js
modulus deploy

```

Modulus 会询问您要将项目部署到哪个项目，并开始部署。Modulus 做什么？它首先确定项目是一个 Meteor 项目，然后运行 **demeteorizer** 将项目转换为通用的 Node.js 应用程序。然后，该应用程序被部署到服务器并自动启动。

# 设置 Kadira

完美！现在我们可以部署生产质量的网络应用程序了，我们需要了解如何识别它们的问题。这就是 `kadira.io` 发挥作用的地方。Kadira 是一个针对 Meteor 的性能和错误监控工具。它将收集客户端和服务器上触发的 `Meteor.Errors`。它还将显示发布者和订阅者的性能数据。

Kadira 可以做更多的事情，而且起始计划完全是免费的。让我们先注册一下。注册后，您需要创建一个新的应用程序：

![设置 Kadira](img/00015.jpeg)

为应用程序决定一个名称，并在字段中输入相同的名称，然后点击**创建应用程序**：

![设置 Kadira](img/00016.jpeg)

创建后，您将看到如下视图：

![设置 Kadira](img/00017.jpeg)

完成，您需要复制步骤 2 中的代码并安装 `meteorhacks:kadira` 和 `meteorhacks:zones` 包：

```js
meteor add meteorhacks:kadira
meteor add meteorhacks:zones

```

`meteorhacks:zones` 包改进了客户端的错误描述。Kadira 将自动利用这个包。需要注意的是，`meteorhacks:zones` 包是可选的，因为它仍在积极开发中，可能会在 Meteor 中引起奇怪的行为。现在，让我们为 Kadira 创建一个配置文件：

```js
# /_globals/server/kadira.coffee

if process.env.NODE_ENV is "production"
  Kadira.connect 'appId', 'secret'
```

我们完成了！使用这个简单的配置，我们现在可以轻松地在 Kadira 上跟踪应用程序错误等：

![设置 Kadira](img/00018.jpeg)

您会在屏幕的左上角注意到**错误**标签。此标签将显示我们应用程序中可能发生的所有错误列表。查看 `meteorhacks` 和他们的学院以了解更多关于优化您的 Meteor 网络应用程序的信息。

# 设置 SSL 证书

**SSL**，或**安全套接字层**，是一种在客户端和服务器之间创建加密连接的技术。如果我们想确保传输到我们服务器的数据是加密的，这是必要的；这包括信用卡信息等数据。

设置 SSL 可能会很痛苦，因为它需要一些命令行知识。我们喜欢从 [`www.namecheap.com/`](https://www.namecheap.com/) 购买我们的 SSL 证书，因为它们便宜且能完成任务。

您可以获得的最低成本的 SSL 证书是**PositiveSSL**；您可以在以下端点找到该产品：[`www.namecheap.com/security/ssl-certificates/single-domain.aspx`](https://www.namecheap.com/security/ssl-certificates/single-domain.aspx)。

在购买证书后，您需要生成一个**证书签名请求**（**CSR**）。让我们来做这件事。您首先将被重定向到**购买摘要**页面。点击**管理**：

![设置 SSL 证书](img/00019.jpeg)

现在点击**立即激活**并保持窗口打开：

![设置 SSL 证书](img/00020.jpeg)

接下来，在您的项目目录中打开终端，并运行以下命令创建一个 `/.csr` 目录：

```js
mkdir .csr
cd .csr

```

现在，让我们使用 `openssl` 创建我们的 CSR。运行以下命令：

```js
openssl req -nodes -newkey rsa:2048 -keyout private.key -out server.csr

```

这将提示您输入网站联系信息；所有信息字段都必须填写您的公司或您的信息。最重要的字段是**通用名称**字段，它必须是 `www.yourdomain.com`。包括 `www` 将同时保护 `www.yourdomain.com` 和 `yourdomain.com`。

命令已创建一个 `private.key` 文件和一个 `server.csr` 文件；您可以通过运行以下命令来检查：

```js
ls

```

请将 `private.key` 放在一个安全的地方！您稍后需要这个文件。现在使用任何文本编辑器（如 Sublime、Atom）或如果您更喜欢更基础的编辑器，可以使用 nano 或 vim 打开 `server.csr` 文件。您可以通过运行以下命令来查看当前目录文件夹的内容：

```js
open .

```

复制此文件中的所有文本。它应该看起来像这样：

```js
-----BEGIN CERTIFICATE REQUEST-----
A bunch of letters and numbers
-----END CERTIFICATE REQUEST-----
```

前往 namecheap，将选择框更改为**其他**并将 CSR 粘贴到 namecheap 的**输入 csr**文本区域字段。它应该看起来像这样：

![设置 SSL 证书](img/00021.jpeg)

现在填写审批人信息：

![设置 SSL 证书](img/00022.jpeg)

确保您选择的电子邮件是活跃的并且受您控制！如果您无法从您选择的电子邮件接收电子邮件，您将无法将 SSL 证书应用到您的 webapp！在下一屏，只需提交您的订单并等待**SSL 证书验证**电子邮件到达：

![设置 SSL 证书](img/00023.jpeg)

复制您的验证码，然后在电子邮件中的**此处**链接上点击。这将带您到另一个网站，您需要粘贴复制的验证码并点击**下一步**：

![设置 SSL 证书](img/00024.jpeg)

关闭窗口，等待您收到一个包含 ZIP 文件的新的电子邮件。下载 ZIP 文件并将其解压。这将包含四个文件：`AddTrustExternalCARoot.crt`、`COMODORSAAddTrustCA.crt`、`COMODORSADomainValidationSecureServerCA.crt` 和 `www_yourdomain_com.crt`。

现在，我们需要使用这些文件为 Modulus 创建一个证书颁发机构包。这基本上是我们所有证书的串联版本。通过在证书所在命令行中运行此命令来生成它：

```js
cat www_yourdomain_com.crt COMODORSADomainValidationSecureServerCA.crt COMODORSAAddTrustCA.crt AddTrustExternalCARoot.crt > certificate_bundle.crt

```

此命令将生成一个包含所有证书的`certificate_bundle.crt`目录。

现在，浏览到 Modulus.io 的管理页面；确保您已经通过将域名添加到自定义域名列表中，将自定义 URL 指向您的 Modulus 实例。点击加号图标：

![设置 SSL 证书](img/00025.jpeg)

现在，打开与 CSR 一起生成的`private.key`文件，复制文件内的全部文本。将信息粘贴到**私钥**文本区域。然后打开`certificate_bundle.crt`文件，复制信息到这里，并将其粘贴到**证书**文本区域：

![设置 SSL 证书](img/00026.jpeg)

确保将您的证书指向`www`域名和非`www`域名。

现在为了完成这个过程，我们需要确保`wizonesolutions:canonical`包将流量路由到受保护的域名。滚动到**环境变量**部分，并将`ROOT_URL`替换为以`https`协议开始的您的域名。您的`ROOT_URL`应如下所示：`https://yourdomain.com`。

您可能需要重新启动服务器以确保所有设置生效。现在访问您的网站，您将看到您被自动重定向到您网站的受保护版本！

# 摘要

本章内容直接明了。我们学习了如何将我们的应用程序部署到由 modulus.io 托管的托管服务器上。我们还学习了如何将我们的 Modulus 项目设置到由 Compose 提供的生产质量数据库服务器上。我们选择 Compose 而不是 Modulus，因为我们可以通过它设置 Meteor 的 oplog tailing。为了帮助我们跟踪应用程序错误，我们安装了 Kadira。我们还学习了如何为我们的服务器设置 SSL 证书以进一步保护我们的网站。有了这些知识，我们可以构建生产质量的 Web 应用程序。
