# 第十章：使用基于云的服务部署

不可避免地，您会希望您一直在构建的应用程序在线并且对世界可用，无论您是想在开发过程中在线托管您的应用程序，还是在应用程序完成并准备投入生产时。目前有许多不同的托管选项可供 Node.js 和基于 MongoDB 的应用程序使用，在本章中，我们将介绍一些不同的热门服务的部署方式。

在这一章中，我们将涵盖以下内容：

+   云与传统 Web 托管

+   Git 源代码控制的介绍

+   使用 Nodejitsu 部署应用程序

+   使用 Heroku 部署应用程序

+   使用 Amazon Web Services 部署应用程序

+   使用 Microsoft Azure 部署应用程序

+   对 DigitalOcean 的简要介绍

# 云与传统托管

如果您之前有网站托管的经验，我将称之为传统托管，您可能对使用 FTP 上传网页文件到托管提供商的过程非常熟悉。在传统 Web 托管中，服务提供商通常为每个用户提供共享空间，每个用户都配置有自己的公共文件夹，用于存放网页文件。在这种情况下，每个客户都托管相同类型的网站，他们的文件都存储在同一台 Web 服务器上并由其提供服务。

传统的 Web 托管成本相对较低，因为单个 Web 服务器可以托管成百上千个个人网站。传统托管通常存在扩展性问题，因为如果您的网站需要更多的资源，它需要被迁移到另一台服务器（具有更多硬件），并且在此迁移过程中可能会出现潜在的停机时间。作为一个副作用，如果与您的网站位于同一服务器上的网站对硬件要求特别高，那么该服务器上的每个网站都可能会受到影响。

使用基于云的托管，每个网站或服务的实例都托管在自己的虚拟专用服务器（VPS）上。当客户上传其网站的副本时，该网站在其自己的隔离环境中运行，并且该环境专门设计用于仅运行该网站。虚拟专用服务器是服务器的实例，通常都同时在同一硬件上运行。由于其隔离性质，VPS 的扩展性非常好，因为只需更改硬件分配的设置，服务器就会重新启动。如果您的 VPS 托管在与其他 VPS 相同的硬件上，并且它们正在经历高流量峰值，您的网站不会因 VPS 的隔离性质而受到影响。

# 基础设施即服务与平台即服务

云的美妙之处在于可以获得的服务级别和数量变化很大。对于运行您的 Web 应用程序的基本托管计划，您可以使用许多被视为平台即服务（PaaS）的服务。这是一种为您提供托管和运行 Web 应用程序的平台。随着规模和复杂性的增加，您可以转向提供整个基于云的数据中心的基础设施即服务（IaaS）提供商。

您可以通过阅读一篇详细的文章了解 IaaS、PaaS 和软件即服务（SaaS）之间的区别，该文章可在[`www.rackspace.com/knowledge_center/whitepaper/understanding-the-cloud-computing-stack-saas-paas-iaas`](http://www.rackspace.com/knowledge_center/whitepaper/understanding-the-cloud-computing-stack-saas-paas-iaas)上找到。

基于云的托管成本可能会有很大的变化，因为它们非常可扩展。您的成本可能会在一个月内发生剧烈波动，这取决于您对资源的需求（即，在一个月中需求更高的时间和/或像 HackerNews 或 Reddit 这样的大型社交媒体的点击）。另一方面，如果您对服务器的需求非常小，您通常可以免费获得云托管！

传统的 Web 托管服务提供商包括 GoDaddy、Dreamhost、1&1、HostGator 和 Network Solutions。热门的基于云的托管选项包括 Nodejitsu（PaaS）、Heroku（PaaS）、Amazon Web Services（IaaS）、Microsoft Azure（IaaS）和 Digital Ocean。

# Git 简介

对于传统的托管提供商，连接到服务器并上传文件的标准方法是使用**文件传输协议**（**FTP**）。您可以使用任何标准的 FTP 软件进行连接，并将文件副本推送到服务器，这些更改将在访问您的网站 URL 时立即反映在线。对于基于云的托管提供商，标准通常是使用 Git 源代码控制。Git 是一种源代码控制技术，允许您跟踪项目源代码的更改和历史，以及提供与多个开发人员轻松协作的简便方法。目前最受欢迎的 Git 在线代码存储库提供商是[www.github.com](http://www.github.com)。

我们将使用 Git 来跟踪我们的应用项目源代码，并将我们的代码推送到各种云托管提供商。当您使用 Git 推送代码时，您实际上是将所有或仅更改版本的代码传输到在线存储库（例如，Git 和[www.github.com](http://www.github.com)相对容易进入，但可能看起来令人生畏和复杂）。如果您对 Git 和/或[`GitHub.com`](https://GitHub.com)不熟悉，我强烈建议您花点时间通过查看以下指南来熟悉：

+   [`help.github.com/articles/set-up-git`](https://help.github.com/articles/set-up-git)

+   [`gist.github.com/andrewpmiller/9668225`](https://gist.github.com/andrewpmiller/9668225)

指南将带您了解以下概念：

+   下载和安装 Git

+   在[`github.com`](https://github.com)注册帐户

+   使用[`github.com`](https://github.com)对您的计算机进行身份验证并创建您的第一个存储库

+   将项目源代码提交到存储库

一旦您将项目源代码配置为本地 Git 存储库，并且所有代码都提交到主分支，就可以继续阅读以下各节。

# 部署您的应用程序

现在，您已经将项目设置为本地 GitHub 存储库，是时候将该代码上线了！接下来的各节将分别介绍将应用程序部署到几个不同的热门基于云的托管提供商的过程。

请随意探索和尝试每个提供商，因为大多数都有免费或相对便宜的计划。每个提供商都有其优势和劣势，所以我将由您决定哪个适合您的特定需求。我们介绍的服务没有特定的顺序。

为了本章的目的，我将一贯地将我的应用命名为`imgploadr`；但是，您的应用名称需要不同和独特。在本章中，无论我何时提到`imgploadr`，您都应该用您自己应用的独特名称替换它。

# Nodejitsu

要开始使用 Nodejitsu，请访问[www.nodejitsu.com](http://www.nodejitsu.com)并首先注册一个免费帐户。在提供您的电子邮件地址、用户名和密码后，您将看到一个定价计划页面，您可以在该页面配置您的服务。如果您只想创建免费帐户并进行实验，只需单击“不，谢谢”按钮，注册过程就完成了。然后，只需单击右上角的“登录”按钮即可登录并转到您的应用程序仪表板。

![](img/c7398050-dfa5-4944-99b2-a69b657b3c89.png)

将您的应用程序部署到 Nodejitsu 将需要一个新的命令行界面工具；具体来说，是`jitsu` CLI。单击大蓝色的使用 jitsu 部署应用程序按钮将带您到此工具的[www.github.com](http://www.github.com)存储库。您可以跳过这一步，只需使用以下`npm`命令手动安装 CLI：

```js
    $ sudo npm install -g-g install jitsu
```

安装`npm`包时，有时需要使用`sudo`命令来全局安装（使用`-g`标志）。取决于您所使用的机器的访问级别，您可能需要或者不需要包括`sudo`。

现在`jitsu` CLI 已安装，您可以使用这个方便的工具来登录到您的 Nodejitsu 帐户，创建一个应用程序，并部署您的项目代码。首先，让我们登录：

```js
$ jitsu login 
info:    Welcome to Nodejitsu 
info:    jitsu v0.13.18, node v0.10.26 
info:    It worked if it ends with Nodejitsu ok 
info:    Executing command login 
help:    An activated nodejitsu account is required to login 
help:    To create a new account use the jitsu signup command 
prompt: username:  jkat98 
prompt: password: 
info:    Authenticated as jkat98 
info:    Nodejitsu ok 
```

您可以看到，在成功提供用户名和密码后，您现在已经通过 Nodejitsu 进行了身份验证，准备好开始了。

在我们实际部署应用程序之前，我们需要在 Nodejitsu 仪表板中配置 MongoDB 数据库。切换回浏览器，在 Nodejitsu 应用程序仪表板上，通过单击数据库选项卡切换部分。

![](img/135e760f-d109-4013-8f8f-94fd238cc1eb.png)

通过单击大的 MongoHQ 按钮，让我们选择 MongoHQ 来满足我们的需求。您将被提示为新数据库命名，然后它将在屏幕底部的“您的数据库”部分列出。我们需要的重要部分是连接字符串，旁边有一个方便的复制链接，可以将其复制到剪贴板。

编辑`server.js`文件，并更新`mongoose.connect`行以使用您为 Nodejitsu 数据库复制的新连接字符串：

```js
[/server.js] 
mongoose.connect('YOUR_NODEJITSU_CONNECTION_STRING_HERE'); 
mongoose.connection.on('open', ()=>{ 
    console.log('Mongoose connected.'); 
}); 
```

唯一剩下的就是打开终端，切换到项目主目录，并执行以下命令来打包您的应用程序并将其推送到 Nodejitsu：

```js
$ jitsu deploy 
info:    Welcome to Nodejitsu jkat98 
info:    jitsu v0.13.18, node v0.10.26 
info:    It worked if it ends with Nodejitsu ok 
info:    Executing command deploy 
warn:  
warn:    The package.json file is missing required fields: 
warn: 
warn:      Subdomain name 
warn: 
warn:    Prompting user for required fields. 
warn:    Press ^C at any time to quit. 
warn: 
prompt: Subdomain name:  (jkat98-imgploadr) imgploadr 
warn:    About to write /Users/jasonk/repos/nodebook/imgploadr/package.json 
... (a lot of npm install output) ... 
info:    Done creating snapshot 0.0.1 
info:    Updating app myapp 
info:    Activating snapshot 0.0.1 for myapp 
info:    Starting app myapp 
info:    App myapp is now started 
info:    http://imgploadr.nodejitsu.com on Port 80 
info:    Nodejitsu ok
```

执行`jitsu deploy`后，CLI 首先会提示您确认在[www.nodejitsu.com](http://www.nodejitsu.com)域名下的子域名是什么。随意更改为您喜欢的内容（它将检查确认可用性）。然后，它会对您的`package.json`文件进行一些微小的修改，具体包括使用您提供的任何值包括`subdomain`选项。最后，它会上传您的源代码并执行远程`npm install`操作。假设一切顺利，应用程序应该已部署，并且 URL 的确认应该输出到屏幕上。随意在浏览器中打开该 URL 以查看在线应用程序！

现在，您还可以看到应用程序在应用程序仪表板中列出：

![](img/40844f95-248b-4505-93e8-5dc413ba1d82.jpg)

现在应用程序已成功上传，通过其 URL 启动它，并尝试上传一个新图像进行测试运行。您应该注意到的第一件事是，尝试上传图像失败，并显示一个相当无用的错误（您可以通过从应用程序仪表板的日志选项卡访问以下错误）：

```js
400 Error: ENOENT, open 
'/opt/run/snapshot/package/public/upload/temp/72118-89rld0.png 
```

这个错误远非有用！基本上，这里发生的是应用程序试图上传并保存图像到实际上并不存在的`temp`文件夹！我们需要向我们的应用程序添加一小段代码来检查这种情况，并在必要时创建文件夹。

编辑`server/configure.js`文件，并在`routes(app);`和`return app;`之间插入以下代码片段：

```js
// Ensure the temporary upload folders exist 
        fs.mkdir(path.join(__dirname, '../public/upload'), 
         (err)=>{ 
            console.log(err); 
            fs.mkdir(path.join(__dirname, 
'../public/upload/temp'),  
                (err)=>{ 
                    console.log(err); 
                }); 
        }); 
```

在这里，我们使用文件系统`fs`模块来创建父`upload`文件夹和`temp`子文件夹。也不要忘记在文件顶部`require` `fs`模块：

```js
const connect = require('connect'), 
    path = require('path'), 
    routes = require('./routes'), 
    exphbs = require('express3-handlebars'), 
    moment = require('moment'), 
    fs = require('fs'); 
```

有一个名为`node-mkdirp`的`npm`模块，它将执行递归的`mkdir`，基本上实现了我们在前面示例中调用的双重`mkdir`。我之所以没有包括它，是为了简洁起见，不包括额外的安装模块、要求它并不必要地使用它的指示。更多信息可以在[`www.npmjs.org/package/mkdirp`](https://www.npmjs.org/package/mkdirp)找到。

在对代码进行了上述更改后，你需要再次部署你的应用程序。只需执行另一个`jitsu deploy`，你的代码的新副本将被上传到你的实例：

```js
$ jitsu deploy 
```

再次打开你的应用程序 URL，这次你应该能够与应用程序进行交互并成功上传新的图片！恭喜，你已成功部署了你的应用程序，现在它正在使用 Nodejitsu 托管服务在线运行！

# Heroku

另一个流行的基于云的 Node.js 应用程序托管提供商是[www.Heroku.com](http://www.Heroku.com)。Heroku 与其他提供商的一个不同之处在于其提供的强大附加组件的数量。任何你能想象到的你的应用程序需要的服务都可以作为附加组件使用，包括数据存储、搜索、日志和分析、电子邮件和短信、工作和排队、监控和媒体。这些附加组件可以快速而轻松地添加到你的服务中，并集成到你的应用程序中。

与 Nodejitsu 一样，Heroku 允许你注册一个免费帐户，并在其*沙箱*定价计划范围内工作。这些计划是免费的，但在带宽、处理能力等方面有限。大多数，如果不是全部，附加组件通常也提供某种免费的沙箱或基于试用的计划。与 Nodejitsu 一样，我们将在 Heroku 应用程序中使用的附加组件之一是 MongoHQ，一个基于云的 MongoDB 服务提供商。

首先，去[`heroku.com`](http://heroku.com)注册一个免费帐户。虽然注册不需要信用卡，但是为了在你的应用程序中包含任何附加组件，你必须在文件中提供信用卡（即使你不选择扩展服务，也不会被收费）。注册后，点击确认电子邮件中的链接并提供密码；你将看到你的应用程序仪表板。

![](img/eed40703-a4d0-4d18-8035-9968d35e4bb7.png)

请注意，你需要做的第一件事是下载 Heroku Toolbelt（与 Nodejitsu 的`jitsu` CLI 类似）。点击下载按钮下载并安装 Toolbelt。Toolbelt 是一个专门用于创建和部署应用程序到 Heroku 的 CLI，并提供了`heroku`命令。

安装 Toolbelt 后，打开命令行终端并切换到项目的根目录。然后执行以下命令

登录 Heroku：

```js
    $ heroku login
    Enter your Heroku credentials.
    Email: jkat98@gmail.com
    Password (typing will be hidden):
    Authentication successful.

```

现在你已经登录，可以直接向 Heroku 发出命令了

帐户并使用这些命令来创建应用程序，安装附加组件并部署你的项目。

你需要做的第一件事是创建一个新的应用程序。通过在命令行中执行`heroku create`来完成：

```js
    $ heroku create
    Creating secret-shore-2839... done, stack is cedar
    http://secret-shore-2839.herokuapp.com/ | git@heroku.com:secret-
 shore-2839.git

```

创建应用程序后，Heroku 会随机分配一个唯一的名称；在我的情况下是`secret-shore-2839`（不过不用担心，这很容易改变）：

```js
    $ heroku apps:rename imgploadr --app secret-shore-2839
    Renaming secret-shore-2839 to imgploadr... done
    http://imgploadr.herokuapp.com/ | git@heroku.com:imgploadr.git
    Don't forget to update your Git remotes on any local checkouts.

```

让我们接下来解决最后一部分。Heroku 依赖于你机器上的 Git 源代码控制，以便将你的项目源代码推送到服务器，而不像 Nodejitsu 那样使用自己的文件传输机制。假设你之前按照关于 Git 和[www.github.com](http://www.github.com)的说明进行了操作，你的项目源代码应该已经准备就绪并提交到主分支，准备好了。接下来我们需要做的是在你的机器上为 Git 添加一个指向 Heroku 的新远程。

让我们从`git init`开始，在当前工作目录中初始化`git`，然后执行以下命令为 Heroku 创建一个新的远程：

```js
    $ git remote add heroku git@heroku.com:imgploadr.git
```

在将源代码推送到 Heroku 帐户之前，我们需要处理一些事情。

在您的 Heroku 服务器上运行应用程序之前，需要一个特殊的文件。这个文件称为`Procfile`，它专门包含启动应用程序所需的命令。在项目的根目录中创建一个名为`Procfile`（无扩展名）的新文件，并包含以下行：

```js
    web: node server.js 
```

就是这样！有了那个文件，Heroku 将使用该命令启动您的应用程序。现在您已经设置了`Procfile`并且您的项目源代码已准备就绪，只剩下一件事要做--安装 MongoHQ 附加组件并配置您的应用程序以使用它：

```js
    $ heroku addons:create mongohq --app imgploadr
    Adding mongohq on imgploadr... done, v3 (free)
    Use 'heroku addons:docs mongohq' to view documentation.
```

添加了 MongoHQ 附加组件后，您现在可以配置数据库本身并检索连接字符串（就像您之前在 Nodejitsu 中所做的那样）。访问您的[`heroku.com`](http://heroku.com)应用程序仪表板，它应该看起来像以下截图：

![](img/4991395a-bda2-4621-980b-56564bf02f32.png)

应用程序的仪表板屏幕是获取应用程序快照和快速查看当前成本的好地方。由于我正在为我的应用程序和附加组件使用沙箱和/或免费计划，我的当前预计月费用为$0.00。但是，如果您需要更多的功能，您可以快速轻松地扩展您的应用程序。请注意，您也可以快速轻松地将您的月费用飙升到天际！（将所有内容扩展到最大，我能够将我的预计费用提高到大约每月$60,000！）。

要配置您的 MongoHQ 数据库，只需在应用程序仪表板的附加组件部分下点击 MongoHQ 链接：

![](img/6ae6d0cd-0d61-4134-8027-2e0245c59c21.jpg)

点击 Collections 标签下方的带有齿轮图标的 Admin 标签。点击 Users 标签，并提供应用程序将用于连接 MongoHQ 数据库的用户名和密码。这将创建具有安全密码的`imgploadrdb`用户名。添加新用户后，切换回概述标签并复制 Mongo URI 字符串。

就像在 Nodejitsu 中一样，编辑项目中的`server.js`文件，并用刚刚复制的新 URI 替换`mongoose.connect`字符串。编辑字符串，并根据您刚刚创建的新用户帐户的情况，用适当的值替换`<username>`和`<password>`。`server.jsmongoose.connect`代码应如图所示：

```js
mongoose.connect('mongodb://imgploadrdb:password@kahana.mongohq.co
 m:10089/app26'); 
mongoose.connection.on('open', ()=>{ 
    console.log('Mongoose connected.'); 
});
```

由于您刚刚对项目的源代码进行了更改，因此需要记住将这些更改提交到 Git 存储库的主分支，以便它们可以上传到 Heroku。执行以下命令，将这些更改永久提交到您的源代码并将代码上传到 Heroku 服务器：

```js
    $ git commit -am "Update mongoose connection string"
    $ git push heroku master
    Initializing repository, done.
    Counting objects: 50, done.
    Delta compression using up to 8 threads.
    Compressing objects: 100% (43/43), done.
    Writing objects: 100% (50/50), 182.80 KiB | 0 bytes/s, done.
    Total 50 (delta 3), reused 0 (delta 0)
    ... npm install output ...
    To git@heroku.com:imgploadr.git
     * [new branch]      master -> master

```

将应用程序启动的最后一步是创建服务器的实例（基本上相当于打开它）。要做到这一点，执行以下命令：

```js
    $ heroku ps:scale web=1 --app imgploadr
    Scaling dynos... done, now running web at 1:1X.
    $ heroku open
    Opening imgploadr... done

```

成功！希望您的浏览器已启动并且您的网站正在运行。继续，尝试上传一张图片！由于我们在 Nodejitsu 部署期间发现的错误，这个应用程序的更新版本应该可以正常工作。

虽然使用 Heroku 部署似乎比 Nodejitsu 更复杂，这可能是因为它使用 Git 源代码控制来促进项目文件的传输。此外，由于 Heroku 在扩展和附加组件的功能方面非常灵活，因此 Toolbelt CLI 更加强大。

# 亚马逊网络服务

虽然 Nodejitsu 和 Heroku 可以被认为是开发人员级别的服务提供商，因为它们是 PaaS，但亚马逊网络服务（和微软 Azure）将被认为是企业级服务，因为它们更像是 IaaS。AWS 和 Azure 提供的选项和服务的数量是令人震惊的。这绝对是顶级服务，像我们这样托管应用程序就像用火箭筒打苍蝇一样！

AWS 确实提供了自己的 NoSQL 数据库，称为 DynamoDB，但是对于我们的目的，我们希望继续使用 MongoDB 并在我们的应用程序中使用 Mongoose。为此，我们可以使用第三方 MongoDB 提供商。如果你还记得，当我们最初设置 Nodejitsu 时，列出的一个 MongoDB 提供商是 MongoLab。MongoLab 提供**MongoDB 作为服务**，这意味着我们可以使用它的服务来托管我们的 MongoDB 数据库，但使用 AWS 的所有功能来托管我们的 Node.js 应用程序（这与 Nodejitsu 和 Heroku 已经发生的情况并没有太大不同；它们只是更好地简化了这个过程）。请记住，AWS 是一个 IaaS 提供商，所以你也可以创建另一个服务器实例并自己安装 MongoDB，并将其用作数据源。但是，这略微超出了本章的范围。

# 创建 MongoLab 帐户和数据库

为了在 AWS 中使用 MongoLab，我们首先需要在[`mlab.com/`](https://mlab.com/)上注册一个新帐户并创建 AWS 数据库订阅。注册新帐户并使用他们通过电子邮件发送给你的链接进行激活后，你可以创建你的第一个数据库订阅。

![](img/9bd62362-3612-43b7-88d0-ff9e03c301de.png)

从你的主仪表板上，点击创建新按钮（带闪电图标）

闪电图标）。

从创建新订阅页面，配置以下设置：

+   云提供商：亚马逊网络服务

+   位置：你喜欢的任何地区

+   计划：选择单节点（开发）

+   选择沙盒（共享/免费）

+   MongoDB 版本：`2.4.x`

+   数据库名称：`anything_you_want`（我选择了`imgploadr`）

+   确认价格为每月$0

+   点击创建新的 MongoDB 部署

回到你的主仪表板，你现在应该看到你的新数据库已经创建并准备就绪。我们需要做的下一件事是创建一个用户帐户，我们的应用程序将用它来连接服务器。点击主仪表板上列出的数据库，然后选择用户选项卡。提供一个新的用户名和密码。添加新用户帐户后，复制位于屏幕顶部的 URI（只有在添加用户后才会出现）以`mongodb://`开头。

现在你有了新的 URI 连接字符串，我们需要更新`server.js`以在`mongoose.connect`中包含这个新的连接字符串。编辑文件并使用以下代码进行更新：

```js
mongoose.connect('mongodb://imgploadrdb:password@ds061248.mongolab
 .com:61248/imgploadr'); 
mongoose.connection.on('open', ()=>{ 
    console.log('Mongoose connected.'); 
}); 
```

确保用 MongoLab 仪表板上创建的用户帐户的适当信息替换`<username>`和`<password>`。

将我们的应用程序代码更新为指向新的 MongoLab 数据库连接字符串后，我们需要将项目文件压缩，以便可以通过 AWS 仪表板上传。从你计算机的文件浏览器中，找到包含所有应用程序源代码文件的项目根目录，选择它们所有，右键单击它们以添加到存档或 ZIP 文件中。ZIP 文件的名称可以是任何你选择的。需要注意的一点是，你不应该在这个 ZIP 文件中包含`node_modules`文件夹（最简单的解决方案可能是直接删除整个文件夹）。如果你需要更多信息，AWS 在线文档有一个关于创建 ZIP 文件的很好的介绍（[`docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.deployment.source.html`](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.deployment.source.html)）。

一旦你的源代码已经更新为使用新的 MongoLab 连接字符串，并且你已经创建了整个项目的 ZIP 文件（不包括`node_modules`文件夹），你就可以创建新的 AWS 应用程序并部署你的应用程序了。

# 创建和配置 AWS 环境

如果您还没有亚马逊帐户，您需要一个才能使用他们的 AWS 服务。将浏览器指向[`aws.amazon.com`](http://aws.amazon.com)，然后点击注册（即使您已经有亚马逊帐户）。在随后的屏幕上，您可以使用现有的亚马逊帐户登录或注册一个新帐户。注册并登录后，您将看到 AWS 提供的整套云服务。

我们感兴趣的主要服务是弹性 Beanstalk（位于部署和管理下，带有绿色图标）：

![](img/1bbff13f-5a46-4007-8623-7fb52c338d51.jpg)

从此屏幕，点击右上角的创建新应用程序链接。随后的屏幕将引导您完成一个多步向导过程，在其中您将配置应用程序所在的环境。在适当的情况下配置以下设置：

+   应用程序信息：

+   应用程序名称：`任何你想要的`

+   环境类型：

+   环境层：`Web 服务器`

+   预定义配置：`Node.js`

+   环境类型：`负载均衡`，`自动扩展`

+   应用程序版本：

+   上传您自己的（选择之前创建的 ZIP 文件）

+   环境信息：

+   环境名称：`任何你想要的`

+   环境 URL：`任何你想要的`（这是您应用程序的子域）

+   配置详情：

+   实例类型：`t1.micro`

其余字段可以留空或使用它们的默认值

+   环境标签：跳过此步骤；对于此应用程序是不必要的

最后一步是审查配置设置，然后启动环境（点击蓝色的 Launch 按钮）。

![](img/c084bb2c-9f8f-4c84-9c31-0ec876a31f7f.png)

弹性 Beanstalk 可能需要几分钟来配置和启动您的环境

应用程序，所以您可能需要耐心等待。环境正式启动并且应用程序在线后，继续打开您的应用程序（点击页面顶部的链接）并进行测试运行。假设一切按计划进行，您的应用程序应该已经启动并且应该正常运行！

# 微软 Azure

微软的 Azure 服务与亚马逊的 AWS 非常相似。两者都可以被视为企业级服务，并且都提供了极大的灵活性和功能，具有非常流畅的用户界面。令人惊讶的是，尽管它是微软产品，您也可以使用 Azure 启动 Linux 环境的实例，以及托管您的 Node.js 和 MongoDB 应用程序。

您需要的第一件事，就像任何其他服务一样，是在[`azure.microsoft.com`](http://azure.microsoft.com)注册帐户。如果您有一个现有的 Microsoft Live 登录，您可以使用它；否则，您可以相当容易地注册一个新帐户。一旦您登录到 Azure 服务，您将首先看到的是您的主要仪表板。左边的图标是 Azure 提供的各种服务和选项。

![](img/1e6ff7d5-9fb1-45cd-9820-6b1e4f130ae7.png)

点击左下角的+NEW 图标将呈现给您

可以用来添加任何新服务的主要对话框。对于我们的目的，我们希望

添加网站：

1.  选择计算、网站和从库中选择。

1.  从众多的库选项中选择 Node JS Empty Site。这将创建必要的环境，以便您有一个可以放置应用程序的地方。

1.  在随后的屏幕上，提供您应用程序的 URL。

1.  将其余字段保留为默认值。

1.  点击对勾图标完成设置过程，您的网站将被创建。

1.  下一步是设置数据库服务器。与 AWS 或 Nodejitsu 类似，我们将再次选择 MongoLab 作为我们的数据库服务提供商。

1.  再次点击+NEW 图标，选择 Store，并浏览列表，直到找到并选择 MongoLab。

1.  点击下一个箭头并浏览各种计划。对于我们的需求，我们将保留 Sandbox 选项（因为它是免费的）。

1.  为您的数据库提供一个名称；在我的情况下，我输入了`imgploadrdb`。

1.  再次单击下一步以查看和确认计划和每月价格（应为每月$0.00）。

1.  最后，单击复选标志图标以购买这个新的订阅计划。

几秒钟后，您应该会回到仪表板，在那里您将看到网站和数据库应用服务的条目：

![](img/fb1656ca-1da9-4573-889b-d0efd47375c2.png)

现在数据库已经创建并准备就绪，我们需要在应用程序中包含其连接字符串，然后才能上传我们的代码：

1.  单击数据库行以选择它并转到其概述。

1.  屏幕底部将包含一些图标，其中一个标有连接信息（并且有一个看起来像>i 的图标）。单击该图标，会弹出一个模态窗口，其中包含您的新 MongoLab 数据库服务器的连接字符串 URI。

1.  将该 URI 复制到剪贴板。

1.  编辑本地应用程序中的`server.js`，并用刚刚复制的新字符串替换`mongoose.connect`连接字符串。无需更新`username`和`password`，因为 Azure 已经使用以下代码为您处理了这个问题：

```js
mongoose.connect('mongodb://your_specific_azure_
                  mongolab_uri'); 
mongoose.connection.on('open', ()=>{ 
    console.log('Mongoose connected.'); 
});
```

一旦更改完成，保存文件，并不要忘记使用 Git 更新您的本地 Git 存储库，因为在下一节中我们将使用 Git 将您的代码推送到 Azure（就像我们之前在 Heroku 上做的那样）：

```js
    $ git commit -am "Azure connection string"
```

回到 Azure 仪表板，在所有项目列表中单击 Web Site（或使用左侧工具栏上的图标筛选网站）。从概述屏幕中，找到朝向底部的集成源控制部分，并单击设置从源控制进行部署的链接。以下屏幕截图显示了此时您应该看到的内容：

![](img/29b5fb99-253e-447f-b760-bd49458a4fa5.png)

选择本地 Git 存储库，然后通过单击下一个箭头图标继续。

接下来的屏幕将呈现如何将本地代码推送到刚刚为您的 Azure 网站创建的远程 Git 存储库的说明。要点是添加一个指向 Azure 存储库的新 Git 远程（就像我们之前在 Heroku 上做的那样），然后推送您的代码：

```js
    $ git remote add azure SPECIFIC_URL_FOR_YOUR_SERVER
    $ git push azure master  
```

当您的代码开始在`git push`命令之后推送时，您应该注意到 Azure 仪表板中的 Git 信息屏幕会实时更新。从命令行中，您将看到大量远程`npm install`输出。完成后，Azure 仪表板中的部署历史将更新，显示最后一次活动部署的信息。

![](img/4b10af1f-3a09-42c9-8a3e-a5a4640d65f1.png)

现在，您的代码已部署到 Azure 网站，并且您的网站连接字符串指向您的 MongoLab Azure 应用服务，您已经准备好测试网站运行情况了。通过将浏览器指向[`yourappname.azurewebsites.net`](http://yourappname.azurewebsites.net)来启动它。Azure 做了很多正确的事情（UI/UX），并且提供了一些非常强大的选项和扩展功能！快速浏览网站仪表板（上述屏幕截图），您会发现有很多事情正在进行。

![](img/bfa39f54-7930-4030-9a86-f10cbfb1c16f.png)

有许多不同的配置选项，以及健康监控和一般信息（FTP 设置，网站 URL，使用度量等），所以请随意查看和探索。

# Digital Ocean

我想要提到并简要介绍的最后一个服务是 Digital Ocean - [`digitalocean.com`](http://digitalocean.com)。Digital Ocean 是一个真正的**虚拟专用服务器**（**VPS**）服务提供商，是一个让您尽可能*接近底层*的服务的很好的例子。这意味着 Digital Ocean 实际上并没有其他我们看到的服务所提供的所有花里胡哨的功能。然而，Digital Ocean 提供的是对您所创建的 Linux 服务器实例的直接、未经过滤的访问，在这种情况下被称为**Droplets**。

Digital Ocean 允许您快速启动新的 Linux 虚拟服务器实例。他们提供非常有竞争力的价格，如果您需要快速获取 Linux 服务器，因为您只需要短时间内的一个，或者您想要启动自己的 Linux 服务器，用于托管生产环境，那么他们是一个很好的选择。唯一的*缺点*（如果我不得不这样说的话）是您必须对 Linux 非常熟悉，特别是对服务器管理和相关责任。

您可以在新的 Droplet 上使用 Git 非常容易地克隆您的项目，但新 Droplet 的实际原始性的一个例子是，Git 不会默认安装在服务器上。您需要在克隆存储库之前手动安装 Git。取决于您在创建新 Droplet 时决定克隆哪个镜像，您可能还需要安装和配置 Node.js 以及 MongoDB。幸运的是，Digital Ocean 在创建新服务器时提供了许多预定义的服务器供您选择，其中包括**MongoDB，Express，Angular**和**Node.js**（**MEAN**）堆栈。除此之外，实际上启动您的应用程序只会在您当前登录的会话期间运行；一旦您退出登录，您的应用程序就会关闭。您需要进一步管理服务器，配置您的应用程序以作为服务运行。

Digital Ocean 允许您直接使用网站内的控制台访问工具连接到您的服务器，或者通过在自己的计算机上的终端直接使用 SSH 连接：

![](img/551b8d9d-c20d-48d3-b477-6e98e6307c0f.jpg)

我之所以提到 Digital Ocean，是因为很多人会觉得这种原始的力量非常令人耳目一新，并且希望自己动手配置和维护他们的服务器。Digital Ocean 是一个很棒的服务，但并不适合每个人。我之所以特别想谈论它，是因为我觉得它完善了我们迄今为止所涵盖的服务列表。

# 总结

我们已经涵盖了基于云的托管服务提供商的整个范围，并介绍了配置您的服务和部署项目代码。Nodejitsu 和 Heroku 是更多面向开发人员的优秀服务，并通过非常易于访问和流畅的用户界面赋予他们很大的权力。亚马逊和微软，作为行业巨头，代表了您可以期望的企业级服务提供商的实力和复杂性。Digital Ocean 是一个无花俏、*接近底层*的基于云的 VPS 提供商，牺牲了花里胡哨的功能，以换取对服务器的原始和直接访问。

我们涵盖的所有托管选项都很棒，但并不是唯一的选择。它们只是一个样本，但它们展示了云的力量！在几分钟内，几乎没有成本，您就可以配置一个环境，并让您的网站在线运行！

在下一章中，我们将介绍单页应用程序的概念以及流行的客户端开发框架和工具。
