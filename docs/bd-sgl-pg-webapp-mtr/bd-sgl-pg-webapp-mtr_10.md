# 第十章：部署我们的应用程序

我们的应用程序现在已准备好部署。在本章中，我们将了解如何将我们的应用程序部署到不同的服务器上，使其公开并向世界展示我们所构建的内容。

Meteor 使得在自身的服务器基础设施上部署应用程序变得非常容易。操作免费且迅速，但可能不适合生产环境。因此，我们将探讨手动部署以及一些为在任何 Node.js 服务器上部署而构建的优秀工具。

在本章中，我们将涵盖以下主题：

+   注册 Meteor 开发者账户

+   在 Meteor 的自有服务器基础设施上部署

+   手动打包和部署 Meteor

+   使用 Demeteorizer 部署

+   使用 Meteor Up 部署

    ### 注意

    如果你想要部署本书中构建的完整应用程序，可以从以下网址下载代码：[`www.packtpub.com/books/content/support/17713`](https://www.packtpub.com/books/content/support/17713) 或从 GitHub 仓库：[`github.com/frozeman/book-building-single-page-web-apps-with-meteor/tree/chapter10`](https://github.com/frozeman/book-building-single-page-web-apps-with-meteor/tree/chapter10)。

    这段代码将不包括创建虚拟帖子的部分，因此你可以在自己的服务器上启动一个干净的博客。

# 在 meteor.com 上部署

Meteor 提供了自己的托管环境，其中每个人都可以用一个命令免费部署应用程序。为了部署应用程序，Meteor 会为我们创建一个开发者账户，以便我们稍后管理和部署应用程序。首先，让我们执行以下步骤，在 [meteor.com](http://meteor.com) 上部署我们的应用程序：

1.  在 meteor.com 的子域上部署就像在我们的应用程序文件夹中的终端运行以下命令那么简单：

    ```js
    $ meteor deploy myCoolNewBlog

    ```

    我们可以自由选择要部署的子域。如果 `myCoolNewBlog.meteor.com` 已经被占用，Meteor 会要求我们登录所有者的账户以覆盖当前部署的应用程序，或者我们必须选择另一个名字。

1.  如果域名可用，Meteor 会要求我们提供一个电子邮件地址，以便它为我们创建一个开发者账户。输入电子邮件地址后，我们将收到一封电子邮件，其中有一个链接设置我们的 Meteor 开发者账户，如下面的屏幕截图所示：![在 meteor.com 上部署](img/00028.jpeg)

1.  为了创建我们的账户，我们需要遵循 Meteor 给出的链接，以便我们通过添加用户名和密码完全设置我们的账户，如下面的屏幕截图所示：![在 meteor.com 上部署](img/00029.jpeg)

1.  完成这些操作后，我们将访问我们的开发者账户页面，在那里我们可以添加电子邮件地址，检查我们的最后登录，以及授权其他 Meteor 开发者登录到我们的应用程序（尽管我们首先必须添加 `accounts-meteor-developer` 包）。

1.  最后，要在终端中使用 `$ meteor login` 登录我们的 Meteor 开发者账户，输入我们的凭据，并再次运行 `deploy` 命令来最终部署我们的应用程序：

    ```js
    $ meteor deploy myCoolNewBlog

    ```

1.  使用`$ meteor authorized –add <username>`命令，我们可以允许其他 Meteor 开发者将应用程序部署到我们应用程序的子域，如下所示屏幕截图：![在 meteor.com 上部署](img/00030.jpeg)

1.  如果我们想更新我们部署的应用程序，我们只需在我们应用程序的文件夹内运行`$ meteor deploy`。 Meteor 将要求我们提供凭据，然后我们可以部署我们的应用程序。

如果我们正在朋友的计算机上，并且想使用我们的 Meteor 账户，可以使用`$ meteor login`。 Meteor 将保持我们登录状态，并且每个人都可以重新部署我们的任何应用程序。 我们需要确保在完成时使用`$ meteor logout`。

## 使用域名在 meteor.com 上部署

我们还可以将应用程序托管在[meteor.com](http://meteor.com)，但可以定义我们自己的域名。

要这样做，我们只需使用我们的域名进行部署，如下所示：

```js
$ meteor deploy mydomain.com

```

这将使应用程序托管在 meteor.com 上，但没有类似于[myapp.meteor.com](http://myapp.meteor.com)的直接 URL。

要将我们的域名指向 Meteor 服务器上的应用程序，我们需要将域名的**A 记录**更改为`origin.meteor.com`的 IP 地址（在撰写本书时为`107.22.210.133`），或**CNAME 记录**更改为`origin.meteor.com`。 您可以在注册域名的 DNS 配置中提供商处进行此操作。

Meteor 然后从我们的域名获取请求并在内部将其重定向到托管我们应用程序的服务器。

## 备份并恢复托管在 meteor.com 上的数据库

如果您需要备份数据库或将它移动到另一个服务器，您可以使用以下命令获取部署数据库的临时 Mongo 数据库凭据：

```js
$ meteor mongo myapp.meteor.com –url

```

这将获取类似于以下凭据：

```js
mongodb://client-ID:xyz@production-db-b1.meteor.io:27017/yourapp_meteor_com

```

然后，您可以使用前面输出的凭据使用`mongodump`备份您的数据库：

```js
$ mongodump -h production-db-b1.meteor.io --port 27017 --username client-ID --password xyz --db yourapp_meteor_com

```

这将在您所在位置创建一个名为`dump/yourapp_meteor_com`的文件夹，并将数据库的转储文件放在里面。

要恢复到另一个服务器，请使用`mongorestore`，最后一个参数是你放置数据库转储的文件夹：

```js
$ mongorestore -h mymongoserver.com --port 27017 --username myuser --password xyz --db my_new_database dump/yourapp_meteor_com

```

如果你只想将数据放入您本地的 Meteor 应用程序数据库中，请使用`$ meteor`启动 Meteor 服务器并运行以下命令：

```js
$ mongorestore --port 3001

```

# 在其他服务器上部署

Meteor 的免费托管很棒，但当涉及到在生产中使用应用程序时，我们希望能够控制我们正在使用的服务器。

Meteor 允许我们将应用程序捆绑在一起，这样我们就可以在任何 Node.js 服务器上部署它。唯一的缺点是我们需要自己安装某些依赖项。此外，还有两个使部署应用程序几乎像 Meteor 本身一样简单的包，尽管它们的配置仍然需要。

## 捆绑我们的应用程序

为了在我们的服务器上部署应用，我们需要一个安装了最新版本的 Node.js 和 NPM 的 Linux 服务器。服务器应该和我们将要创建捆绑包的本地机器是同一平台。如果你想在另一个平台上部署你的应用，查看下一节。现在让我们通过以下步骤构建应用：

1.  如果我们的服务器符合上述要求，我们可以在本地机器上的应用文件夹中运行以下命令：

    ```js
    $ meteor build myAppBuildFolder

    ```

1.  这将创建一个名为`myAppBuildFolder`的文件夹，里面有一个`*.tar.gz`文件。然后我们可以将这个文件上传到我们的服务器，并在例如`~/Sites/myApp`下提取它。然后我们进入提取的文件夹并运行以下命令：

    ```js
    $ cd programs/server
    $ npm install

    ```

1.  这将安装所有的 NPM 依赖。安装完成后，我们设置必要的环境变量：

    ```js
    $ export MONGO_URL='mongodb://user:password@host:port/databasename'
    $ export ROOT_URL='http://example.com'
    $ export MAIL_URL='smtp://user:password@mailhost:port/'
    $ export PORT=8080

    ```

    `export`命令将设置`MONGO_URL`、`ROOT_URL`和`MAIL_URL`环境变量。

1.  由于这种手动部署没有预装 MongoDB，我们需要在我们的机器上安装它，或者使用像 Compose 这样的托管服务（[`mongohq.com`](http://mongohq.com)）。如果我们更愿意自己在服务器上安装 MongoDB，我们可以遵循在[`docs.mongodb.org/manual/installation`](http://docs.mongodb.org/manual/installation)的指南。

1.  `ROOT_URL`变量应该是指向我们服务器的域的 URL。如果我们的应用发送电子邮件，我们还可以设置自己的 SMTP 服务器，或使用像 Mailgun 这样的服务（[`mailgun.com`](http://mailgun.com)）并更改`MAIL_URL`变量中的 SMTP 主机。

    我们也可以指定我们希望应用运行的端口，使用`PORT`环境变量。如果我们没有设置`PORT`变量，它将默认使用端口`80`。

1.  设置这些变量后，我们转到应用的根目录，并使用以下命令启动服务器：

    ```js
    $ node main.js

    ```

    ### 提示

    如果你想确保你的应用在崩溃或服务器重启时能够重新启动，可以查看`forever` NPM 包，具体解释请参阅[`github.com/nodejitsu/forever`](https://github.com/nodejitsu/forever)。

如果一切顺利，我们的应用应该可以通过`<your server's ip>:8080`访问。

如果我们手动部署应用时遇到麻烦，我们可以使用接下来的方法。

## 使用 Demeteorizer 部署

使用`$ meteor build`的缺点是，大多数 node 模块已经被编译，因此在服务器环境中可能会造成问题。因此出现了 Demeteorizer，它与`$ meteor build`非常相似，但还会额外解压捆绑包，并创建一个包含所有 node 依赖项和项目正确 node 版本的`package.json`文件。以下是使用 Demeteorizer 部署的方法：

1.  Demeteorizer 作为一个 NPM 包提供，我们可以使用以下命令安装：

    ```js
    $ npm install -g demeteorizer

    ```

    ### 注意

    如果`npm`文件夹没有正确的权限，请在命令前使用`sudo`。

1.  现在我们可以去应用文件夹并输入以下命令：

    ```js
    $ demeteorizer -o ~/my-meteor-blog-converted

    ```

1.  这将把准备分发的应用程序输出到`my-meteor-blog-converted`文件夹。我们只需将这个文件夹复制到我们的服务器上，设置与之前描述相同的环境变量，并运行以下命令：

    ```js
    $ cd /my/server/my-meteor-blog-converted
    $ npm install
    $ node main.js

    ```

这应该会在我们指定的端口上启动我们的应用程序。

## 使用 Meteor Up 部署

前面的步骤可以帮助我们在自己的服务器上部署应用程序，但这种方法仍然需要我们构建、上传和设置环境变量。

**Meteor Up**（**mup**）旨在使部署像运行`$ meteor deploy`一样简单。然而，如果我们想要使用 Meteor Up，我们需要在服务器上拥有完全的管理权限。

此外，这允许我们在应用程序崩溃时自动重新启动它，使用`forever` NPM 包，以及在服务器重新启动时启动应用程序，使用`upstart` NPM 包。我们还可以恢复先前的部署版本，这为我们提供了在生产环境部署的良好基础。

### 注意

接下来的步骤是针对更高级的开发人员，因为它们需要在服务器机器上设置`sudo`权限。因此，如果您在部署方面没有经验，可以考虑使用像 Modulus 这样的服务（[`modulus.io`](http://modulus.io)），它提供在线 Meteor 部署，使用自己的命令行工具，可在[`modulus.io/codex/meteor_apps`](https://modulus.io/codex/meteor_apps)找到。

Meteor Up 将按照以下方式设置服务器并部署我们的应用程序：

1.  要在我们的本地机器上安装`mup`，我们输入以下命令：

    ```js
    $ npm install -g mup

    ```

1.  现在我们需要创建一个用于部署配置的文件夹，这个文件夹可以位于我们的应用程序所在的同一个文件夹中：

    ```js
    $ mkdir ~/my-meteor-blog-deployment
    $ cd ~/my-meteor-blog-deployment
    $ mup init

    ```

1.  Meteor Up 为我们创建了一个配置文件，它看起来像以下这样：

    ```js
    {
      "servers": [
        {
          "host": "hostname",
          "username": "root",
          "password": "password"
          // or pem file (ssh based authentication)
          //"pem": "~/.ssh/id_rsa"
        }
      ],
      "setupMongo": true,
      "setupNode": true,
      "nodeVersion": "0.10.26",
      "setupPhantom": true,
      "appName": "meteor",
      "app": "/Users/arunoda/Meteor/my-app",
      "env": {
        "PORT": 80,
        "ROOT_URL": "http://myapp.com",
        "MONGO_URL": "mongodb://arunoda:fd8dsjsfh7@hanso.mongohq.com:10023/MyApp",
        "MAIL_URL": "smtp://postmaster%40myapp.mailgun.org:adj87sjhd7s@smtp.mailgun.org:587/"
      },
      "deployCheckWaitTime": 15
    }
    ```

1.  现在我们可以编辑这个文件以适应我们的服务器环境。

1.  首先，我们将添加 SSH 服务器认证。我们可以提供我们的 RSA 密钥文件，或者提供一个用户名和密码。如果我们想要使用后者，我们需要安装`sshpass`，一个用于在不使用命令行的前提下提供 SSH 密码的工具：

    ```js
    "servers": [
        {
          "host": "myServer.com",
          "username": "johndoe",
          "password": "xyz"
          // or pem file (ssh based authentication)
          //"pem": "~/.ssh/id_rsa"
        }
    ],
    ```

    ### 注意

    要为我们的环境安装`sshpass`，我们可以按照[`gist.github.com/arunoda/7790979`](https://gist.github.com/arunoda/7790979)的步骤进行，或者如果您在 Mac OS X 上，可以查看[`www.hashbangcode.com/blog/installing-sshpass-osx-mavericks`](http://www.hashbangcode.com/blog/installing-sshpass-osx-mavericks)。

1.  接下来，我们可以设置一些选项，例如选择在服务器上安装 MongoDB。如果我们使用像 Compose 这样的服务，我们将将其设置为`false`：

    ```js
    "setupMongo": false,
    ```

    如果我们已经在我们的服务器上安装了 Node.js，我们还将将下一个选项设置为`false`：

    ```js
    "setupNode": false,
    ```

    如果我们想要指定一个特定的 Node.js 版本，我们可以如下设置：

    ```js
    "nodeVersion": "0.10.25",
    ```

    Meteor Up 还可以为我们安装 PhantomJS，这对于我们使用 Meteor 的 spiderable 包是必要的，这个包可以使我们的应用程序被搜索引擎爬取：

    ```js
    "setupPhantom": true,
    ```

    在下一个选项中，我们将设置我们应用程序的名称，它可以与我们的应用程序文件夹名称相同：

    ```js
    "appName": "my-meteor-blog",
    ```

    最后，我们需要指向我们的本地应用程序文件夹，以便 Meteor Up 知道要部署什么：

    ```js
    "app": "~/my-meteor-blog",
    ```

1.  Meteor Up 还允许我们预设所有必要的环境变量，例如正确的`MONGO_URL`变量：

    ```js
    "env": {
        "ROOT_URL": "http://myServer.com",
        "MONGO_URL": "mongodb://user:password@host:port/databasename",
        "PORT": 8080
    },
    ```

1.  最后一个选项设置了 Meteor Up 在检查应用是否成功启动前会等待的时间：

    ```js
    "deployCheckWaitTime": 15
    ```

### 设置服务器

为了使用 Meteor Up 设置服务器，我们需要对`sudo`进行无密码访问。按照以下步骤设置服务器：

1.  为了启用无密码访问，我们需要将当前用户添加到服务器的`sudo`组中：

    ```js
    $ sudo adduser <username> sudo

    ```

1.  然后在`sudoers`文件中添加`NOPASSWD`：

    ```js
    $ sudo visudo

    ```

1.  现在用以下这行替换`%sudo ALL=(ALL) ALL`行：

    ```js
    %sudo ALL=(ALL) NOPASSWD:ALL

    ```

### 使用 mup 部署

如果一切顺利，我们可以设置我们的服务器。以下步骤解释了如何使用`mup`进行部署：

1.  从本地`my-meteor-blog-deployment`目录中运行以下命令：

    ```js
    $ mup setup

    ```

    这将配置我们的服务器并安装配置文件中选择的全部要求。

    一旦这个过程完成，我们随时可以通过在同一目录下运行以下命令来部署我们的应用：

    ```js
    $ mup deploy

    ```

通过创建两个具有不同应用名称的 Meteor Up 配置，我们还可以创建生产和演示环境，并将它们部署到同一服务器上。

# 前景

目前，Meteor 将原生部署限制在其自己的服务器上，对环境控制有限。计划推出一款企业级服务器基础设施，名为**Galaxy**，它将使部署和扩展 Meteor 应用像 Meteor 本身一样简单。

尽管如此，凭借 Meteor 的简洁性和强大的社区，我们已经拥有部署到任何基于 Node.js 的托管和 PaaS 环境的丰富工具集。

### 注意

例如，如果我们想在 Heroku 上部署，我们可以查看 Jordan Sissel 在[`github.com/jordansissel/heroku-buildpack-meteor`](https://github.com/jordansissel/heroku-buildpack-meteor)上的构建包。

# 总结

在本章中，我们学习了如何部署 Meteor，以及在 Meteor 自己的服务器架构上部署可以有多么简单。我们还使用了 Demegorizer 和 Meteor Up 这样的工具来部署我们自己的服务器架构。

要了解更多具体的部署方法，请查看以下资源：

+   [`www.meteor.com/services/developer-accounts`](https://www.meteor.com/services/developer-accounts)

+   [`docs.meteor.com/#/full/deploying`](https://docs.meteor.com/#/full/deploying)

+   [`www.meteor.com/services/build`](https://www.meteor.com/services/build)

+   [`github.com/onmodulus/demeteorizer`](https://github.com/onmodulus/demeteorizer)

+   [`github.com/arunoda/meteor-up`](https://github.com/arunoda/meteor-up)

您可以在这个应用的[完整示例代码](https://www.packtpub.com/books/content/support/17713)中找到准备部署的版本，或者在 GitHub 上查看[`github.com/frozeman/book-building-single-page-web-apps-with-meteor/tree/chapter10`](https://github.com/frozeman/book-building-single-page-web-apps-with-meteor/tree/chapter10)。

在下一章中，我们将创建一个包含我们之前创建的`ReactiveTimer`对象的包，并将其发布到 Meteor 的官方包仓库。
