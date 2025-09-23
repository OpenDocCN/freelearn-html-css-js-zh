# 第一章. 优化您的流程

在本章中，我们将涵盖以下主题：

+   安装 Meteor

+   查找 Meteor 的文档

+   获取问题的帮助

+   设置您的项目文件结构

+   设置您的开发环境

+   使用 Web 控制台

+   将测试应用部署到 Meteor

+   使用 CNAME 重定向部署到 Meteor

+   部署到自定义托管环境

+   使用 Meteor Up (MUP)进行部署

+   使用 CoffeeScript

+   使用 CSS 编译器

# 简介

欢迎来到 Meteor 的奇妙世界！本章将向您介绍一些优化您的 Meteor 工作流程的最佳实践。从安装 Meteor 到将您的成品应用部署到您自己的定制服务器上，这里找到的食谱将帮助您快速启动并运行，并在您开始每天使用 Meteor 时提供极佳的参考。

# 安装 Meteor

Meteor 团队使安装变得简单。只要您正在工作的系统（Linux 或 Mac OS X）相对较新，安装 Meteor 应该非常简单。我们将其包括为食谱，以便当您在新机器上安装 Meteor 时，您手头就有它。我们还将包括来自 GitHub 的安装说明，以防您想使用开发分支。

## 准备工作

您必须安装`curl`才能安装 Meteor。如果您在 Mac OS X 上，`curl`已经安装。如果您在 Linux 上，`curl`通常已经安装。要检查`curl`，请打开一个终端窗口并执行以下命令：

```js
$ curl

```

如果已安装`curl`，您将收到以下消息：

```js
curl: try 'curl --help' or 'curl --manual' for more information

```

如果没有安装`curl`，您将收到类似以下的消息：

```js
-bash: /usr/bin/curl: No such file or directory

```

要安装`curl`，使用`apt-get`或`yum`（根据您的 Linux 版本）。以下是在 Ubuntu 上安装`curl`的命令。只需将命令中的`apt-get`替换为`yum`，即可在 Fedora、Debian 或 CentOS 上安装：

```js
$ sudo apt-get install curl

```

您应该看到一个安装消息，可能还会有提示安装。完成安装过程；然后您将准备好安装 Meteor。

## 如何做到这一点...

打开一个终端窗口并执行以下命令：

```js
$ curl https://install.meteor.com/ | sh

```

## 它是如何工作的...

`curl https://install.meteor.com/`命令用于直接从 Meteor 获取安装脚本。一旦脚本下载完成，`| sh`参数告诉您的系统执行该脚本，该脚本将 Meteor 安装到`/usr/local/bin/meteor`（它应该在您的路径中）。一旦整个脚本执行完成，您将能够从任何位置运行`meteor`命令。

## 更多...

之前的食谱涵盖了默认/稳定的 Meteor 安装。如果您想直接从源代码安装 Meteor，或者如果您想安装一个开发分支（例如，当前的夜间版本），您可以使用`git`来完成。

Meteor 仓库位于[`github.com/meteor/meteor`](https://github.com/meteor/meteor)。您可以在该位置找到各种开发分支，以及主要的 Meteor 仓库。

假设您已经安装了`git`并且想要安装主要的 Meteor 构建，打开一个终端窗口并执行以下命令：

```js
$ git clone git://github.com/meteor/meteor.git

```

这将把最新的 Meteor 构建克隆到名为`meteor`的子文件夹中。请注意，这不会在您的机器上全局安装 Meteor。这意味着，为了运行`meteor`命令，您需要导航到（或引用）您刚刚克隆的 Meteor 仓库所在的`meteor`文件夹。

### 小贴士

根据您的具体情况，可以使用各种其他安装参数和自定义选项。要查看更全面的说明，请访问 Meteor GitHub 主`README.md`页面[`github.com/meteor/meteor`](https://github.com/meteor/meteor)。

# 查找 Meteor 的文档

Meteor 终于达到了 1.0 版本，因此，它的各种库和功能开始稳定下来。然而，Meteor 的一些部分仍然在变化中。因此，拥有最新的文档非常重要。本指南将向您展示在需要时您可以在哪里找到官方 Meteor 文档。

## 如何操作…

简单地在一个浏览器中导航到[`docs.meteor.com/`](http://docs.meteor.com/)并收藏该页面。

## 它是如何工作的…

就这些了。Meteor 团队会保持文档的更新，您可以确信在那里找到所有生产就绪的功能，以及大量的宝贵建议。

## 还有更多…

如果您想要有一个离线文档副本并确保您拥有文档的最新和最佳版本，您可以使用 Meteor 的 GitHub 仓库来提供最新文档的本地副本。

要获取文档的初始副本，请导航到您想要存储文档的位置（例如，您的`Documents`目录）并使用以下命令克隆最新的 Meteor 仓库：

```js
$ git clone git://github.com/meteor/meteor.git

```

如果你已经完成了这个操作，并且想要确保你拥有最新和最好的版本，请导航到`meteor`子文件夹（在你执行初始 Git 克隆时创建）并运行以下命令：

```js
$ git pull origin master

```

这将更新 Meteor 仓库到最新和最好的版本。

一旦您获得了最新的文档，您可以通过在浏览器中导航到`docs`目录来访问它：

```js
$ cd docs/

```

然后，运行`meteor`命令：

```js
$ meteor

```

一旦包更新，Meteor 文档项目将启动，您将在命令行中看到以下内容：

```js
=> Meteor server running on: http://localhost:3000/

```

您现在可以使用浏览器导航到`http://localhost:3000/`并阅读最新的文档。

# 获取帮助解答问题

您使用 Meteor 的越多，主要包就越熟悉。然而，您可能还希望开始尝试一些不太为人所知的包的高级功能。您也可能会有一个关于 Meteor 的新应用想法。无论如何，您最终会遇到需要比文档中包含的信息更多的信息的情况。以下是一些关于您需要问题时去哪里寻找答案的快速提示。

## 如何做到这一点…

获取问题解答有三个主要来源：

1.  对于具体的技术问题或难题，请访问 [`stackoverflow.com/questions/tagged/meteor`](http://stackoverflow.com/questions/tagged/meteor)。搜索其他人已经提出的问题；如果您找不到答案，请提交您自己的问题。

1.  对于一般问题和评论，请访问 [`forums.meteor.com/`](https://forums.meteor.com/) 上的 Meteor 论坛。

1.  要与多个 Meteor 开发者（包括核心团队成员）实时工作，您可以访问 `irc.freenode.net` 上的 IRC #meteor 聊天室。

## 它是如何工作的…

让我们看看如何使用之前提到的每个资源的技巧。

### Stack Overflow

Stack Overflow 是一个非常活跃的开发者社区，其中许多人乐于帮助他人解决技术问题。对于 Meteor 特定的帮助，一旦您导航到 [`stackoverflow.com/questions/tagged/meteor`](http://stackoverflow.com/questions/tagged/meteor)，您可以使用最能描述您问题的关键词进行搜索，例如，如果您想在 PhoneGap 应用程序内运行 Meteor，您可能想搜索所有标记为 **[meteor]** 的问题中的 `cordova`，如下所示：

![Stack Overflow](img/image00311.jpeg)

这将搜索所有之前提交的、带有关键词 **[meteor]** 并在问题中包含 `cordova` 的标记问题。您将想要查看结果，看看您的问题是否已经被提出。如果其中一个问题（及其答案）满足您的需求，那就太好了！如果您的提问尚未得到解答，您可以提出自己的问题。

点击标有 **提问** 的按钮：

![Stack Overflow](img/image00312.jpeg)

填写您问题的详细信息。别忘了包括标签，以便更容易识别和分类您的问题。提交您的问题，Stack Overflow 社区很可能很快就会为您找到答案。

### Meteor 论坛

Meteor 论坛同样非常活跃，每天都有许多热情的 Meteor 开发者访问该网站。您可以在许多不同的 Meteor 相关主题上提问、查看讨论和发表评论。大多数时候，如果您有一个技术问题或难题，最好使用 Stack Overflow。然而，论坛也是获取更多抽象或具体 Meteor 问题的绝佳资源。

### #meteor 在 IRC 上

#meteor IRC 频道是一个很好的频道，你可以加入并收听。你将能够与各种优秀的开发者互动，其中许多人愿意立即回答你的问题。请记住，你遇到的大多数问题或问题（尤其是当你刚开始接触 Meteor 时）已经在 Stack Overflow 或 Meteor 论坛上得到了解决，因此在 IRC 聊天中提问之前，先检查并确保你的问题还没有被回答是礼貌的。

## 还有更多…

请记住，由于 Meteor 仍然很新，有许多人在努力解决你可能能够回答的问题。此外，当你找到自己问题的答案时，可能值得将其发布在 Stack Overflow 上。这将帮助其他可能遇到相同问题的人节省时间。

在 Stack Overflow、Meteor 论坛以及甚至 #meteor IRC 上回答他人的问题是一种为社区做出贡献的好方法。

### 小贴士

如果你自己找到了答案，别忘了在 Stack Overflow 上回答你自己的问题！

# 设置你的项目文件结构

当你在 Meteor 中启动一个新项目时，默认的文件配置是为了让你立即开始运行。这很好，但如果你希望保持代码优化且易于理解，你可能需要稍微调整一下文件结构。以下食谱概述了你可以在大多数项目中使用的文件结构。

## 准备工作

你将想要创建一个新的基本 Meteor 项目，然后我们将对其进行修改以适应我们的需求。在终端窗口中，导航到你想要创建项目的文件夹，并输入以下命令以创建一个名为 `FileTemplate` 的项目：

```js
$ meteor create FileTemplate

```

### 小贴士

**下载示例代码**

你可以从你购买的所有 Packt 出版物书籍的账户中下载示例代码文件。[`www.packtpub.com`](http://www.packtpub.com)。如果你在其他地方购买了这本书，你可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 并注册，以便将文件直接通过电子邮件发送给你。可选地，代码示例也可以从 [`github.com/strack/PacktMeteorRecipes`](https://github.com/strack/PacktMeteorRecipes) 下载。

这将创建基本文件结构。现在，导航到你在文件资源管理器窗口中选择的文件夹。在这个练习中，我们将使用 Mac OS X Finder 窗口，这样我们就可以可视化文件夹结构：

![准备工作](img/image00313.jpeg)

### 小贴士

总是以一个新的 Meteor 项目开始（不要尝试手动创建你的文件夹结构）。尽管文件结构看起来很简单，但 `meteor create` 命令创建的隐藏文件和文件夹是你无法离开的！

## 如何做…

要设置你的项目文件结构，请按照以下步骤进行：

1.  在您的根目录中，删除默认的`.css`、`.js`和`.html`文件。**FileTemplate**现在应该看起来是空的，类似于以下截图：![如何操作…](img/image00314.jpeg)

    ### 小贴士

    而不是删除基本的`.html`、`.css`和`.js`文件，您可以使用以下步骤中讨论的代码/文件条目将它们分开。

1.  在您的根文件夹中创建以下子文件夹：

    +   `client`

    +   `server`

    +   `both`

    +   `private`

    +   `public`

1.  导航到`client`子文件夹。创建一个名为`main.html`的新文件。编辑此文件并添加以下代码（从`FileTemplate.html`获取）：

    ```js
    <head>
      <title>FileTemplate</title>
    </head>

    <body>
      <h1>Welcome to Meteor!</h1>

      {{> hello}}
    </body>

    <template name="hello">
      <button>Click Me</button>
      <p>You've pressed the button {{counter}} times.</p>
    </template>
    ```

1.  在`client`文件夹中创建一个新的子文件夹，命名为`scripts`。

1.  在`scripts`内部，创建一个名为`main.js`的新文件。编辑此文件，添加以下代码（从`FileTemplate.js`中的`Meteor.isClient`部分获取）：

    ```js
       // counter starts at 0
      Session.setDefault('counter', 0);

      Template.hello.helpers({ counter: function () { return Session.get('counter');
        }
      });

      Template.hello.events({ 'click button': function () { // increment the counter when button is clicked
          Session.set('counter', Session.get('counter') + 1);
        }
      });
    ```

1.  在`client`文件夹中创建一个新的子文件夹，命名为`lib`。创建两个子文件夹，分别命名为`scripts`和`styles`。

1.  在`styles`内部，创建一个名为`style.css`的新文件。

1.  导航到`server`子文件夹。创建一个名为`server.js`的新文件。将以下代码添加到`server.js`文件中并保存更改：

    ```js
    Meteor.startup(function () {
      // code to run on server at startup
    });
    ```

    您完成的文件结构现在应该看起来像以下截图：

    ![如何操作…](img/image00315.jpeg)

## 它是如何工作的…

我们将主要成对地分解这个文件结构。这是因为几乎对于每个文件夹，都有一个服务于相反目的的另一个文件夹。

### client/server

`client`文件夹被 Meteor 解释为仅属于客户端的代码。`server`文件夹则正好相反，告诉 Meteor，包含在其中的任何文件仅应用于服务器端处理。

将`client`和`server`文件夹分开逻辑上分离了您的代码（使其更容易可视化），节省了不必要的处理，并防止了错误（如果由服务器处理，客户端模板和渲染代码将引发错误）。

在`client`文件夹内部，我们放置了一个`scripts`子文件夹和两个文件，它们都具有主前缀（`main.html`和`main.js`）。`scripts`子文件夹是为了我们的便利，将 HTML 和 JavaScript 分开，这样当我们回头查看代码时，它将更容易阅读和分段。

### main/lib

`main`前缀用于告诉 Meteor：*最后运行此代码*。当 Meteor 开始处理和编译文件时，它会等待处理所有名为`main`的文件，直到最后。这有助于在需要在使用它们之前初始化库或脚本的情况下。

相反，`lib`文件夹告诉 Meteor：*首先运行此代码*。放置在`lib`文件夹内部或作为其子项的任何内容都将由 Meteor 在运行时首先运行。这是一个放置全局变量、尚未作为包包含的通用库（参见第二章，*使用包进行自定义*）以及当然任何样式表的好地方。这就是为什么我们包括了`scripts`和`styles`子文件夹，并创建了一个初始的通用`style.css`文件。

使用`main`前缀和`lib`文件夹一起可以帮助我们保持正确的顺序，并确保我们的样式尽可能快地加载。

### public/private

`public`文件夹是一个存储需要全局访问的资产的仓库。这是一个存储非敏感信息（如图像和文本文件）非常好的地方。它也很容易记住。如果你需要将某些内容公开显示，请将其放在`public`文件夹中。

`private`文件夹正好相反。这是一个只能由服务器查看的资产存储位置。这是一个放置适度敏感信息的好地方（你将需要额外的安全措施，如数据库，来保护你想要更安全的信息）。任何放置在`private`文件夹中的文件都可以由服务器通过没有特殊路径或导航参数来引用。

### both

`both`文件夹独立存在，是客户端和服务器都可以访问的文件夹。当你创建模型和集合（参见第四章，*创建模型*）时，你希望将声明放在`both`文件夹内的一个文件中。这样做的主要原因是不重复代码，并确保客户端和服务器都从相同的模型声明开始工作。

## 还有更多...

显然，你不必每次都从头开始创建这个文件结构。你可以使用你喜欢的脚本引擎（Grunt、Gulp、Shell Script 等）并创建一个脚本，它可以通过单个命令进行必要的更改。或者，你可以创建一个模板，然后将整个文件夹复制到新位置，然后就像它是新项目一样使用它。

如果你确实决定使用这个配方而不是自动化脚本，请确保定期将你的基础模板更新到最新的 Meteor 构建版本，每次复制时也是如此。这可以在终端窗口中非常容易地完成。导航到根文件夹（例如`~/Documents/Meteor/FileTemplate`）并运行以下命令：

```js
$ meteor update

```

你将收到一条消息，告诉你文件夹已经是最新的，或者 Meteor 将更新项目到最新版本。

## 参见也

+   第二章，*使用包进行自定义*

# 设置你的开发环境

在你开发项目的同时，有几个默认包和设置可以使你的调试和开发工作变得更加容易。这个配方将为你提供一些建议，帮助你创建一个理想的发展环境。

## 准备工作

你需要确保已经安装了 Meteor（当然），并且你需要设置一个默认的项目文件结构。你可以使用本章早些时候的“设置你的项目文件结构”配方，或者使用`meteor create [...]`命令创建一个新的 Meteor 项目。

这个配方假设你已经设置了一些 Mongo 集合和/或某些类型的用户账户。如果你没有添加集合，你可能只想为了测试目的添加一个。为此，创建一个名为`[项目根]/both/model.js`的文件，并输入以下代码行：

```js
testCollection = new Mongo.Collection('testCollection');
```

在有一个项目和至少一个要监控的 Mongo 集合的情况下，我们准备开始开发和调试了！

## 如何操作…

首先，我们想确保`insecure`和`autopublish`包已安装，这对于默认的 Meteor 项目通常是默认情况。

1.  在终端窗口中，导航到你的项目根目录，并输入以下命令：

    ```js
    $ meteor list

    ```

    这将列出你的项目使用的所有包。你应该会收到类似于以下的内容：

    ```js
    autopublish      1.0.2
    insecure         1.0.2
    meteor-platform  1.2.1
    …

    ```

1.  如果缺少`autopublish`或`insecure`包，你可以通过输入以下命令手动添加它们：

    ```js
    $ meteor add autopublish
    $ meteor add insecure

    ```

1.  接下来，安装`msavin:mongol`智能包。在终端窗口中，输入以下命令：

    ```js
    $ meteor add msavin:mongol

    ```

    在简短的安装过程之后，你应该会收到一条消息，表明`msavin:mongol`包已正确安装。

1.  通过在终端窗口中输入以下命令来启动你的`meteor`项目：

    ```js
    $ meteor

    ```

1.  通过导航到`http://localhost:3000`在浏览器中打开你的应用程序。一旦进入，按下*Ctrl* + *M*，并查看屏幕的左下角。你会看到一个可展开的控制台，类似于以下截图：![如何操作…](img/image00316.jpeg)

    点击任何集合名称或**账户**标签，你可以查看、更新、删除和插入记录，如下面的截图所示：

    ![如何操作…](img/image00317.jpeg)

当你在构建你的应用程序时，这会非常有用，因为你可以立即访问你的集合和用户账户配置文件。

## 它是如何工作的…

以下行是安装包所需的所有内容：

```js
$ meteor add msavin:mongol

```

这告诉 Meteor 去寻找包含所有依赖的`msavin:mongol`包，并将其添加到你的项目中。这是一个第三方包，更多关于这个包的详细信息可以在[`atmospherejs.com/msavin/mongol`](https://atmospherejs.com/msavin/mongol)找到。

安装完成后，您可以通过浏览器页面上的 *Ctrl* + *M* 快捷键访问它。在底层，`mongol` 包正在监控任何集合，特别是将用户账户相关的集合过滤到 **账户** 选项卡中。除此之外，所有集合都同等对待，这个界面只是简化了您在需要查看集合内部情况时的调试/开发过程。

## 还有更多...

Mongol 非常新，但它一直在不断改进。您可以通过访问 [`github.com/msavin/Mongol`](https://github.com/msavin/Mongol) 了解所有当前功能并预览即将推出的功能。

## 参见

+   在第二章 *Customizing with Packages* 中的 *Adding Meteor packages*、*Removing Meteor packages* 和 *Using npm modules* 配方，*Customizing with Packages*

+   在第九章 *Securing Your Application* 中的 *Basic safety – turning off autopublish* 和 *Basic safety – removing insecure* 配方，*Securing Your Application*

# 使用网页控制台

有时在调试时修改代码会太慢。这个配方将向您介绍网页控制台，并给您一些使用它的提示。

## 准备工作

我们需要一个沙盒来玩耍。为了这个配方，我们将使用 Meteor 内置的默认示例之一。

在终端窗口中，输入以下命令：

```js
$ meteor create --example leaderboard

```

创建完成后，导航到 `leaderboard` 文件夹并启动 Meteor：

```js
$ cd leaderboard
$ meteor

```

我们现在可以使用我们选择的网页浏览器导航到我们的排行榜示例页面（`http://localhost:3000`）。

我们需要在各种浏览器中启用网页控制台。不分先后，以下是如何在您可能工作的浏览器上访问网页控制台的方法：

### Safari

通过转到 **Safari** | **偏好设置** | **高级** (*CMD+*, 如果您更喜欢快捷键) 并确保启用 **菜单栏中显示开发菜单** 选项来启用 **开发** 菜单：

![Safari](img/image00318.jpeg)

现在，在任何网页上，点击 **开发** 下的 **显示错误控制台**（或使用 *CMD* + *Alt* + *C* 快捷键）：

![Safari](img/image00319.jpeg)

这将打开网页控制台，如下面的截图所示：

![Safari](img/image00320.jpeg)

### Firefox

在任何网页上，点击 **工具** | **Web 开发者** 下的 **Web 控制台**菜单选项（*CMD* + *Alt* + *K* 快捷键）：

![Firefox](img/image00321.jpeg)

网页控制台现在如以下截图所示：

![Firefox](img/image00322.jpeg)

### Chrome

在任何网页上，点击 **查看** | **开发者** 下的 **开发者工具**菜单项（*CMD* + *Alt* + *I* 快捷键）：

![Chrome](img/image00323.jpeg)

这将打开网页控制台，如下面的截图所示：

![Chrome](img/image00324.jpeg)

## 如何做到...

为了展示网页控制台可以实现的功能，让我们创建一个新的科学家，并为 Tesla 添加一些分数。因为，我要让你完成，但尼古拉·特斯拉是有史以来最伟大的科学家。就是。所有。时间。

1.  如果您还没有这样做，请导航到 `http://localhost:3000/`。

1.  现在，如果尚未打开，请打开网页控制台。

1.  在控制台中运行以下命令：

    ```js
    > Players.insert({name:"Stephen Hawking"})

    ```

1.  **斯蒂芬·霍金** 的名字现在将在列表中。您可以选择他的名字并添加一些分数：![如何操作…](img/image00325.jpeg)

1.  现在，让我们给 Tesla 应得的关爱。首先，让我们找到 Tesla 记录的 `_id` 值。在控制台中运行以下命令：

    ```js
    > Players.findOne({name:"Nikola Tesla"})

    ```

1.  从结果中提取 `_id` 值并修改分数，类似于以下内容：

    ```js
    > Players.update({_id:"bPDpqp7hgEx4d6Eui"}, {$set:{score:334823}})

    ```

    现在，Tesla 将被放置在排行榜的顶部：

    ![如何操作…](img/image00326.jpeg)

1.  最后一次操作，让我们取消选择之前截图中所提到的任何科学家。在控制台中输入以下命令：

    ```js
    > Session.set("selectedPlayer",null)

    ```

    您会注意到，现在没有选择任何科学家。

### 小贴士

您可以通过按上箭头或下箭头键在网页控制台中运行之前的命令。

## 它是如何工作的...

网页控制台充当您代码的扩展。您能够接收日志消息，调用公开的方法和事件，或更改和实例化全局变量。您能够在实时动态地做到这一点，这在测试或调试时可以带来很多优势。

通过几行精心编写的代码，我们能够复制事件处理程序和属于应用程序的方法中的行为。换句话说，我们能够手动测试事物，而无需修改、保存和重新测试代码。这种即兴测试可以真正节省我们时间。

## 还有更多…

网络控制台和相关开发工具可以执行各种客户端调试和测试任务。您可以运行预写的脚本，在执行过程中暂停代码，监控变量和调用堆栈，甚至可以动态地创建自己的方法和变量。

### 小贴士

网页控制台在各个浏览器之间略有不同，但您可以在 [`developers.google.com/chrome-developer-tools/docs/console`](https://developers.google.com/chrome-developer-tools/docs/console) 找到适用于几乎所有网页控制台的说明。

# 将测试应用部署到 Meteor

如您所期待的那样，Meteor 使几乎所有事情都变得更容易做。在服务器上测试您的应用程序也不例外。这个食谱将向您展示如何将您的应用程序部署到 Meteor 服务器，在那里您可以测试（并展示！）您的新应用程序。

## 准备工作

这个食谱的唯一要求是您有一个可工作的应用程序。

您的应用不需要很复杂或完整，但它至少应该在屏幕上渲染一些内容，以便您可以验证它是否在 Meteor 服务器上运行。

对于这个食谱，我们将使用默认的 `leaderboard` 示例，该示例是在终端窗口中使用以下命令创建的：

```js
$ meteor create --example leaderboard

```

## 如何操作...

要将测试应用程序部署到 Meteor，请按照以下步骤进行：

1.  首先，你需要选择一个应用程序名称。

    应用程序名称将作为你的应用程序被服务的 URL 的子域出现在 Meteor 服务器上，例如 `http://myproject.meteor.com`。

    然而，有很多人在 Meteor 服务器上测试应用程序。很可能一个通用名称，如 "myproject" 或 "leaderboard"，已经被占用。所以，我们需要选择一个独特的名称。

    对于这个食谱，我将使用这个应用程序名称 `packtrecipe`。你显然需要选择其他名称。我是第一个到这里来的！

    ### 小贴士

    你的项目和应用程序名称不需要匹配。应用程序名称只是一个标识符，这样 Meteor 服务器就知道如何路由到你的应用程序。

1.  现在我们已经选择了应用程序名称，我们将打开一个终端窗口并导航到我们项目的根目录：

    ```js
    $ cd leaderboard

    ```

1.  一旦我们进入正确的文件夹，我们将按照以下方式发出 `deploy` 命令：

    ```js
    $ meteor deploy packtrecipe

    ```

1.  Meteor 将打包并部署你的应用程序。完成后，你应该在终端窗口中看到以下类似的内容：

    ```js
    Deploying to packtrecipe.meteor.com.  Bundling...
    Uploading...
    Now serving at packtrecipe.meteor.com

    ```

1.  在浏览器中导航到 Meteor 给你的 URL（在这种情况下，`http://packtrecipe.meteor.com`），你将看到你新部署的应用程序，如下所示：![如何操作…](img/image00327.jpeg)

## 它是如何工作的…

这太神奇了！Meteor 的人都是巫师！

实际上，核心开发团队已经非常努力地使部署尽可能简单。在底层，你的本地 Meteor 安装正在打包正确的文件，编译包列表和依赖项，然后向 Meteor 服务器发送请求。一旦 Meteor 服务器收到请求，就会进行所有必要的错误检查，并创建和初始化包、数据库和应用程序。添加一个虚拟主机地址，然后 voila！你的应用程序就上线并运行了。

其中涉及很多细节（和代码），但这些都应该能给你一个关于如何部署到 Meteor 测试服务器的总体概念。你不高兴自己不必编写所有这些代码吗？

## 还有更多…

你可能已经注意到，你必须创建一个 Meteor 开发者账户才能部署。这是应该的，因为没有登录/安全细节，有人可能会来并用自己的部署覆盖你的部署。

然而，这个要求相当灵活，允许你添加其他用户甚至组织，以便多个人可以部署或更新你的应用程序。

要设置一个组织，导航到 [`www.meteor.com/account-settings/organizations`](https://www.meteor.com/account-settings/organizations) 并点击屏幕底部的“**新建组织**”按钮。然后你可以添加个人用户到组织中，类似于以下截图：

![还有更多…](img/image00328.jpeg)

一旦部署了你的应用程序，你可以通过`meteor authorized`命令授权个人或组织所做的更改。使用以下语法添加授权账户：

```js
$ meteor authorized [your-url.meteor.com] --add [user/organization]

```

例如，我们会使用以下命令将`packtmeteor`组织添加到我们的部署应用中：

```js
$ meteor authorized packt.meteor.com --add packtmeteor

```

同样，如果我们想取消授权（例如，让我们取消`strack2`账户的授权），我们会输入类似以下的内容：

```js
$ meteor authorized packt.meteor.com --remove strack2

```

### 小贴士

Meteor 开发者账户还有其他有用的功能。要探索你可以用你的账户做什么，请访问[`www.meteor.com/account-settings`](https://www.meteor.com/account-settings)。

## 参见

+   在第二章的*构建自定义包*食谱中，*使用包进行定制*

# 使用 CNAME 重定向部署到 Meteor

将应用部署到 Meteor 测试服务器一切都很顺利，但如果你想使用自己的自定义域名怎么办？Meteor 团队为你提供了一个简单的方法来实现这一点。这个食谱将向你展示如何利用这个简单而强大的功能。

## 准备就绪

你需要在你的域名注册的主机服务上创建一个指向`origin.meteor.com`的**CNAME**重定向。具体操作方法差异很大，因此请咨询你的主机服务知识库以获取确切步骤。对于这个食谱，我们将使用`packtpub.com`域名的主机服务的 cPanel 界面。

在你的**CNAME**重定向中输入你希望使用的子域名（例如，`meteor.packtpub.com`），并将重定向位置设置为`origin.meteor.com`。点击**添加 CNAME 记录**以提交记录：

![准备就绪](img/image00329.jpeg)

## 如何操作...

假设我们将要使用的子域名是`meteor.packtpub.com`。为了部署到 Meteor 环境，请执行以下步骤：

1.  一旦你的**CNAME**重定向设置正确，打开一个终端窗口，导航到你的项目根目录，然后输入以下命令：

    ```js
    $ meteor deploy meteor.packtpub.com

    ```

1.  当捆绑、上传和提供步骤完成时，Meteor 会部署你的应用并提供反馈：

    ```js
    Deploying to meteor.packtpub.com.  Bundling...
    Uploading...
    Now serving at meteor.packtpub.com

    ```

1.  要验证应用程序，请在浏览器中导航到你的应用程序 URL（例如，`http://meteor.packtpub.com`）。如果一切部署正确，你将看到你的应用程序正在运行。

## 它是如何工作的...

这个 Meteor 部署功能几乎与默认部署相同，只是内置了一些额外的代码来解释**CNAME**重定向的来源。

当请求通过**CNAME**重定向到达`origin.meteor.com`时，Meteor 会使用**CNAME**重定向的原始目的地作为应用程序部署的唯一标识符。Meteor 还会使用来自此**CNAME**重定向的后续请求来提供应用程序。

在这个特定情况下，原始**CNAME**目标地址是`meteor.packtpub.com`。当请求被重定向到`origin.meteor.com`时，Meteor 会识别**CNAME**重定向并使用它来将流量导向已部署的应用程序。

## 参见

+   本章中关于*将测试应用程序部署到 Meteor*的配方

# 部署到自定义托管环境

当你准备将你的应用程序实际部署到生产环境时，你需要将你的 Meteor 应用程序转换为纯 Node 应用程序。有多种方法可以做到这一点，所有这些方法（目前）都是手动过程，并且不适合初学者。可能会出错很多。这个配方将向你展示如何手动将你的 Meteor 应用程序作为 Node 应用程序部署到生产服务器。

## 准备工作

正如提到的，有多种方法和服务器配置可以成功托管 Node 应用程序。然而，有一些共同点，这些共同点对于这个配方都是必需的。

首先，你需要一个托管**MongoDB**数据库。

你的 MongoDB 数据库可以托管在远程服务器上，或者在你将部署 Node 应用程序的同一台机器上。对于这个配方，我们已经将默认的 MongoDB 数据库部署到了我们将托管应用程序的同一台机器上。因此，我们的`MONGO_URL`值将是：

```js
MONGO_URL=mongodb://localhost:27017

```

### 小贴士

MongoDB 的安装说明（和教程）可以在[`docs.mongodb.org/manual/installation/`](http://docs.mongodb.org/manual/installation/)找到。

免费或便宜的专用 MongoDB 服务托管也存在。运行一个快速的互联网搜索以查找*MongoDB 托管*，或者访问[`www.mongodb.com/partners/cloud`](http://www.mongodb.com/partners/cloud)以获取提供商列表。

接下来，你的托管环境需要最新的稳定 Node 和 npm 模块。

这些程序的安装说明超出了本书的范围。有关安装说明和如何开始的建议，请访问以下链接：

+   Node ([`nodejs.org`](http://nodejs.org))

+   npm ([`www.npmjs.com/`](https://www.npmjs.com/))

### 小贴士

这些程序的最新或夜间构建在生产环境中可能会给你带来问题。请确保使用稳定版本，除非你已经彻底测试了其他版本。

最后，你需要在你的托管环境中安装`forever` npm 模块。从你的托管环境的一个终端窗口中，运行以下命令：

```js
$ npm install –g forever

```

注意，你可能需要使用`sudo`命令全局安装软件包。这取决于 npm 在你的服务器上是如何安装的。前面的命令将在你的机器上安装`forever`，现在你就可以准备和部署你的应用程序到生产环境了。

## 如何做这件事...

要部署到自定义托管环境，请按照以下步骤操作：

1.  在你的开发环境中打开一个终端窗口，导航到你的 Meteor 项目的根目录。在这个根目录中，执行以下命令：

    ```js
    $ meteor build [your-build-location] --architecture [x]

    ```

    将上一行代码中的占位符替换为你的构建位置（例如：`~/Documents/builds/mybuild`）和一个架构（选项有 `os.osx.x86_64`、`os.linux.x86_64` 和 `os.linux.x86_32`）。如果你是在将要部署应用程序的服务器上构建，`--architecture` 选项是可选的。

1.  Meteor 将捆绑、提取并重新打包你的 Meteor 项目的干净生产副本，为在托管环境中使用做好准备。在构建过程中，Meteor 将在终端窗口中更新你的状态。

1.  一旦构建完成，导航到你指定的构建文件夹，例如，如果你指定 `~/Documents/builds/mybuild` 作为你的构建位置，你需要输入以下命令：

    ```js
    $ cd ~/Documents/builds/mybuild

    ```

1.  在构建文件夹中，你会看到一个名称类似于 `[your-project-name].tar.gz` 的 `tarball` 文件。

    例如，如果我的项目名称是 `leaderboard`，那么 `tarball` 文件名称将是 `leaderboard.tar.gz`。

1.  记下名称和位置，因为当你将构建复制并提取到服务器时，你需要用到它。

1.  对于这个食谱，让我们假设你正在使用 Linux 服务器托管你的生产应用程序。让我们使用以下终端命令创建一个 `meteorapps` 文件夹：

    ```js
    $ mkdir /home/meteorapps

    ```

1.  接下来，将 `tarball` 文件从你的开发环境复制到生产托管环境中的 `/home/meteorapps/` 文件夹。

    如果你在一个不同的机器上构建你的应用程序，你可以通过 SCP、FTP、常见的文件服务器、Git 仓库等方式进行传输。实际上，你如何复制它并不重要，只要现在你的托管环境中有一个 `tarball` 文件副本即可。

1.  一旦复制完成，在 `tarball` 文件上运行以下命令：

    ```js
    $ tar –xf  [your-tarball-name].tar.gz

    ```

    这将把文件提取到一个名为 `bundle` 的文件夹中。如果你导航到 `bundle` 文件夹，你会找到一个 `README` 文件。本食谱的下一步操作来自这个 `README` 文件，所以你可以自由查看它们以获得更简洁的指令集。如果你检查文件夹内容，你应该会看到以下类似的内容：

    ```js
    README   main.js   programs   server   star.json
    ```

1.  现在，我们将安装运行我们的应用程序所需的 npm 软件包。导航到 `bundle/programs/server` 并执行以下命令：

    ```js
    $ npm install

    ```

    ### 注意

    注意，我们没有使用 `–g` 参数，因为我们只安装了捆绑文件夹中指定的本地 npm 软件包。

1.  接下来，我们需要设置 `PORT` 和 `MONGO_URL` 导出参数，以便 Node 知道如何托管我们的应用程序。在终端窗口中输入以下命令：

    ```js
    $ export PORT=8080
    $ export MONGO_URL=mongodb://localhost:27017

    ```

    这两个导出命令告诉你的 Node 服务器要监听哪个端口，以及在哪里找到 `mongodb` 托管实例（在这个例子中我们使用的是本地实例）。

    你还需要配置 `ROOT_URL` 和 `MAIL_URL` 环境变量。输入它们的语法类似于以下导出命令：

    ```js
    $ export ROOT_URL='http://[your-hostname.com]'
    $ export MAIL_URL='smtp://user:password@mailhost:port/'

    ```

现在，我们可以运行我们的应用程序。记住，我们安装了 npm 的 `forever` 包。`forever` 包允许我们运行 Node 应用程序，并在遇到错误时自动重启它。在终端窗口中运行以下命令：

```js
$ forever start main.js

```

此命令指示 Node 以节点应用程序的方式启动 `main.js`，并在出现任何问题时重启它。

### 小贴士

你可以通过从 `bundle/` 目录发出以下命令来稍后停止应用程序：

```js
$ forever stop main.js

```

现在是时候通过打开浏览器并将它指向你指定的主机环境来测试你的应用程序是否成功运行了，例如，如果我们的生产环境托管了 `meteorapp.packtpub.com` 子域名，并且我们指定 `端口 8080`，如前例所示，我们将在浏览器中导航到 `http://meteorapp.packtpub.com:8080`。

你的应用程序应该已经启动并开始服务页面。

## 它是如何工作的...

Node 是为了尽可能快地运行而构建的。为了做到这一点，它会在不同的硬件和软件配置上以不同的方式运行。这意味着你在开发环境中使用的 Node 和 npm 包（例如，在 MAC OS X 上）与生产环境中的相应 Node 和 npm 包（例如，Linux Ubuntu 12.4 LTS Precise）略有不同。这尤其适用于基础包，如 npm 的 fibers 包。

此外，尽管 Meteor 是基于 Node 构建的，但它并不是一个原生 Node 应用程序。它增加了一些抽象层和处理层，这些层可以让你作为开发者更容易地生活，但它们并不构成最漂亮的原生 Node 生产环境。

`meteor build` 命令会为我们处理这个问题，并创建一个不包含已安装 npm 包的构建。相反，它将任何 npm 包列为依赖项。由于具体的 npm 包没有被包含在内（Meteor 在一个包清单文件中列出了它们），因此没有兼容性问题。我们只是简单地告诉 Node 使用包清单文件作为某种清单来查找和安装特定于当前环境的包。我们是在发出 `npm install` 命令时这样做。

一旦 npm 读取了清单文件，检索并安装了所有需要的包，并通知我们安装已正确完成，我们就可以运行我们的新原生 Node 应用程序。

我们然后设置了一些导出变量（`PORT`、`MONGO_URL`、`ROOT_URL` 和 `MAIL_URL`），并使用 `forever` npm 包而不是正常的 node 命令来运行我们的应用程序。使用 `forever` 可以帮助我们不必每次服务器崩溃时都返回服务器。理想情况下，我们永远不会有一个应用程序崩溃，但我们生活在这个真实的世界中，当需要重启应用程序时，`forever` 是一个巨大的时间节省者。

## 更多信息...

如果我们的应用程序真正准备好投入生产，我们也希望它在服务器重启时自动启动。这取决于你的生产服务器运行的操作系统的各种方法，我们不会涵盖所有这些方法。

我们将提供一个示例脚本和如何在 Ubuntu 上执行此操作的说明，并让你根据需要修改脚本以适应其他环境。

每当 Ubuntu 服务器重启时，它都会运行在`/etc/init/`文件夹中找到的任何`*.conf scripts`脚本。我们假设我们的应用程序位于`/home/meteorapps/prodapp/bundle/`，我们将监听`端口 8080`，并且我们使用本地 MongoDB 服务（请根据需要调整这些设置）。

使用具有`sudo`权限的终端窗口，将以下`meteorapp.conf`脚本添加到生产服务器的`/etc/init/`：

```js
#  prodapp configuration file
#  /etc/init/meteorapp.conf

start on (local-filesystems)
stop on shutdown
script
    cd /home/meteorapps/prodapp/bundle/
    export MONGO_URL=mongodb://localhost:27107 PORT=8080
    export ROOT_URL='http://example.com'
    export MAIL_URL='smtp://user:password@mailhost:port/'
    exec forever start main.js
end script
```

让我们分解一下这个脚本做了什么。

前两行（`start on`和`stop on`）告诉操作系统何时运行此脚本。在这种情况下，我们在本地文件系统准备好后立即运行它，并在收到关机请求后立即停止它。

然后，我们有将要运行的脚本。我们使用`cd /home/meteorapps/prodapp/bundle`导航到我们的应用程序文件夹。

然后，我们声明我们的 MongoDB 服务位置和我们希望 Node 监听的端口。

最后，我们执行`forever`命令，要求它以 Node 应用程序启动`main.js`。

Meteor Group 目前正在开发一个名为**Galaxy**的项目，它将使部署到自定义服务器变得与部署到测试服务器一样简单。一旦 Galaxy 可用，这个配方就会变得过时，但这是一件好事！如前所述，这个配方只涵盖了一种部署到生产环境的方法。你只需浏览[`forums.meteor.com`](https://forums.meteor.com)或访问其他 Meteor 信息来源，就可能找到另一种更简单的方法。

## 相关内容

+   本章的*使用问题获取帮助*配方

+   本章的*使用 Meteor Up (MUP)部署*配方

+   在第十二章“创建有用的项目”中的*将应用部署到移动设备*配方，*创建有用的项目*

# 使用 Meteor Up (MUP)部署

如果我们没有添加一种将 Meteor 应用部署到生产服务器的最终方法，那将是我们的疏忽。这种方法被称为**Meteor Up**（**MUP**）。使用一个相当简单的配置文件，MUP 极大地简化了服务器部署过程。在我们看到 Galaxy（Meteor 的部署平台）的样子之前，MUP 是部署到自定义环境的最简单方式。这个配方将向你展示如何使用 Meteor Up 将你的应用部署到自定义服务器。

## 准备工作

要完成这个配方，你需要一个可工作的 Meteor 应用程序。它不需要很复杂，但确实需要是功能性的，这样你就可以一眼看出它是否已经正确安装在你的服务器上。

## 如何做到这一点…

MUP 通过 npm 安装。它需要全局安装，因此我们将使用 `-g` 参数。要使用 MUP 将您的应用程序部署到自定义服务器，请执行以下步骤：

1.  要安装 MUP，打开一个终端窗口并执行以下命令：

    ```js
    $ npm install -g mup
    ```

    这将需要一些时间来安装，但一旦完成，您就可以配置和使用 MUP 来部署您的应用程序。

1.  要运行 `mup init`，在终端窗口中导航到您的项目根目录并输入以下命令：

    ```js
    $ mup init

    ```

    您将收到类似以下的消息：

    ```js
    Empty Project Initialized!

    ```

    此过程将在您的项目根目录中创建两个文件。我们不需要担心第一个文件（`settings.json`）。我们将使用第二个文件，命名为 `mup.json` 来配置我们的应用程序。

1.  现在，我们将配置 `mup.json`。在编辑器中打开 `[项目根目录]/mup.json` 文件。文件的顶部将包含我们的远程服务器信息。它应该类似于以下代码行：

    ```js
    "servers": [
        {
          "host": "hostname",
          "username": "root",
          "password": "password"
          // or pem file (ssh based authentication)
          //"pem": "~/.ssh/id_rsa"
        }
      ],
    ```

    `host` 属性将是您将通过 SSH/SCP 访问的服务器名称。如果我们的服务器名称是 `my-production-server.com`，则主机属性将类似于以下内容：

    ```js
    "host": "my-production-server.com",
    ```

    `username` 和 `password` 属性是如果您要 `ssh` 登录远程服务器时将使用的用户/密码组合。

    下面的四个属性帮助我们配置服务器。如果我们想让 MUP 安装 Mongo、Node 或 PhantomJS（所有这些都是通过 MUP 部署所必需的），我们可以指定如下代码：

    ```js
      "setupMongo": true,
      "setupNode": true,
      "nodeVersion": "0.10.33",
      "setupPhantom": true,
    ```

    ### 小贴士

    就本书的印刷而言，最新的稳定 Node 版本是 `0.10.33`。您可能想检查这一点并相应地修改 `nodeVersion` 属性。

    如果 Mongo/Node/PhantomJS 已经安装，您可以将前面的属性更改为 `false`（这将加快您的部署速度）。

    接下来，我们指定我们想要我们的应用程序被称为什么。这是通过修改 `appName` 属性来完成的：

    ```js
    "appName": "[your-app-name-here]",
    ```

    我们需要指定我们本地机器上即将部署的应用程序所在的文件夹。这由 `app` 属性确定：

    ```js
    "app": "/path/to/the/app",
    ```

    我们需要设置的最后一个属性是 `env`。这指示 Node 从哪个 `ROOT_URL` 和 `PORT` 运行我们的 Meteor 应用程序。例如，如果我们正在将传入的 HTTP 流量重定向到本地的 `port 1337`（这是通过反向代理，如 `nginx`，或虚拟主机，如 `apache` 来完成的），我们的 `env` 配置将类似于以下代码：

    ```js
    "env": {
      "ROOT_URL": "http://localhost",
      "PORT": 1337
    },
    ```

1.  现在，让我们使用 `mup setup` 来配置我们的远程服务器。在终端窗口中，导航到您的项目根目录并输入以下命令：

    ```js
    $ mup setup

    ```

    这将在远程服务器上安装 MongoDB、Node 和 PhantomJS。它还将配置我们的远程服务器环境并安装一些辅助 npm 包，例如 `upstart`。

1.  让我们使用 `mup deploy` 来部署我们的应用程序。一旦 `mup setup` 命令完成，我们就可以准备部署我们的应用程序。在终端窗口中执行以下命令：

    ```js
    $ mup deploy

    ```

MUP 会将您的应用程序本地打包，上传构建，在远程服务器上配置必要的 npm 包，然后提供您的应用程序。在这个过程中，MUP 将在终端中提供状态更新。

完成后，您的应用程序应该已经启动并运行。您可以通过访问外部 URL（例如，`http://my-custom-server.com`）或通过 `SSH` 登录到远程服务器，并在本地主机上使用 `curl` 命令测试构建（例如，`curl http://localhost:3000`）来测试它。

您可能需要进行一些故障排除，以确保您的虚拟主机或反向代理配置正确，但经过一些小的调整后，您会发现，目前 MUP 无疑是部署到自定义服务器的最佳方式。

## 它是如何工作的…

MUP 执行我们通常需要手动实施的步骤（如本章中 *部署到自定义托管环境* 的食谱），并自动实现它们。成功执行此操作有三个主要部分。

首先，当我们执行 `mup init` 命令时，MUP 会创建一个默认配置文件。我们使用所有设置编辑新创建的文件，以便在远程服务器上安装正确的软件，配置环境变量，并上传我们的生产构建。

其次，我们使用 `mup install` 命令在远程服务器上安装所有需要的软件，这些软件在 `mup.json` 配置文件中已指定。

最后，我们的应用程序被打包、上传、解压，使用环境变量初始化，并设置为在远程服务器上运行。这一切都是通过 `mup deploy` 命令完成的。

MUP 根据我们编辑的配置文件执行这些任务。

### 小贴士

我们可以配置许多额外的设置，并且可以通过访问 MUP 仓库来探索 MUP 的许多优秀功能，MUP 仓库位于 [`github.com/arunoda/meteor-up`](https://github.com/arunoda/meteor-up)。

## 参见

+   本章中 *部署到自定义托管环境* 的食谱

# 使用 CoffeeScript

许多人更喜欢 CoffeeScript 而不是标准的 JavaScript 语法。如果您是这些人中的一员，并且有偏好，Meteor 会满足您的需求。这个食谱将向您展示如何快速在您的开发环境中启用 CoffeeScript。

## 准备工作

除了安装 Meteor 和创建一个项目以便开始使用 CoffeeScript 之外，无需为这个食谱做任何准备。

## 如何操作…

要使用 CoffeeScript，请按照以下步骤操作：

1.  打开一个终端窗口并导航到您的项目根目录。

1.  输入以下命令：

    ```js
    $ meteor add coffeescript

    ```

    您应该在终端窗口中看到以下响应：

    ```js
    coffeescript  added, version 1.0.

    ```

    CoffeeScript 现已安装并准备就绪！

您可以通过创建一个 `.coffee` 文件并添加一个脚本（例如，将 `test.coffee` 添加到您的 `client` 文件夹）来测试安装。当您使用 `meteor` 命令启动您的 Meteor 应用程序时，脚本应该按预期执行。

### 小贴士

你可以轻松地卸载 CoffeeScript。只需执行以下终端命令：

```js
$ meteor remove coffeescript

```

## 工作原理…

在幕后，你刚刚安装的 CoffeeScript 包会寻找任何`*.coffee`和`*.litcoffee`文件，将它们编译成原生 JavaScript，然后打包供你的运行中的 Meteor 应用程序使用。

### 注意

就像在运行中的 Meteor 应用程序中的所有其他文件一样，你的 CoffeeScript 更改将立即被处理。

## 相关链接

+   在第二章的*添加 Meteor 包*食谱中，*使用包进行自定义*

# 使用 CSS 编译器

任何 CSS 爱好者都知道，使用标准 CSS 创建样式表可能是繁琐且重复的工作。许多设计师和开发者更喜欢使用动态样式表语言或预处理器，例如 Less、Stylus 和 SCSS/SASS。

Meteor 不仅使预处理器可用，而且将它们视为任何其他文件，以便更改能够立即反映出来。

这个食谱将向你展示如何在你的 Meteor 应用程序中启用一些更流行的 CSS 编译器。

## 准备工作

除了安装 Meteor 和创建一个项目以便你可以开始使用 CSS 编译器之外，无需为这个食谱做任何准备。

## 如何操作…

我们将介绍三种不同的预处理器，因为它们都以类似的方式工作。

### 使用 Stylus

1.  打开终端窗口并导航到你的项目根目录。

1.  输入以下命令：

    ```js
    $ meteor add stylus

    ```

    你应该在终端窗口中看到类似的响应：

    ```js
    stylus  added, version 1.0.7

    ```

    Stylus 现在已安装并准备好使用。你可以通过创建一个`.styl`文件并添加一个脚本（例如，将`test.styl`添加到你的`client/styles`文件夹）来测试这一点。

当你使用`meteor`命令启动你的 Meteor 应用程序时，Stylus 文件将被处理，并渲染适当的 CSS。

### 小贴士

你也可以在 Meteor 中使用`nib`代码。只需将`@import 'nib'`添加到你的`.styl`文件中，Meteor 就会处理其余部分。

### 使用 Less

1.  打开终端窗口并导航到你的项目根目录。

1.  输入以下命令：

    ```js
    $ meteor add less

    ```

    你应该在终端窗口中看到以下响应：

    ```js
    less  added, version 1.0.14

    ```

    现在 Less 包已安装，你可以使用 Less 样式表语法来创建你的 CSS。

就像 Stylus 一样，你可以通过创建一个`.less`文件并添加一些样式声明（例如，将`test.less`添加到你的`client/styles`文件夹）来测试这一点。

当你使用`meteor`命令启动你的 Meteor 应用程序时，Less 文件将被 Meteor 编译成标准 CSS 并按常规渲染。

### 小贴士

如果你喜欢在你的 Less 样式表中使用`@import`语句，请确保使用`.lessimport`扩展名。否则，Meteor 将自动导入和编译它找到的任何`.less`文件。

### 使用 SCSS / SASS

1.  打开终端窗口并导航到你的项目根目录。

1.  输入以下命令：

    ```js
    $ meteor add fourseven:scss

    ```

    你应该在终端窗口中看到类似的响应：

    ```js
    fourseven:scss  added, version 2.1.1

    ```

现在可以使用 SCSS 和 SASS 文件来设置你的 CSS 样式。就像之前一样，你可以通过创建一个 `.scss` 或 `.sass` 文件并添加一些样式声明（例如，将 `test.sass` 添加到你的 `client/styles` 文件夹中）来测试这一点。

当你使用 `meteor` 命令启动你的 Meteor 应用程序时，SCSS 或 SASS 文件将被 Meteor 编译成标准 CSS 并渲染。

## 它是如何工作的...

当你使用 `meteor add` 命令安装任何预处理程序时，它会安装相应的 npm 包，以便在 Meteor 内部工作。

就像其他文件一样，Meteor 会监控任何 `*.styl`、`.less`、`.scss` 和 `.sass` 文件的更改，将更改编译成 CSS，并立即渲染更改。

## 参见

+   在第二章的 *添加 Meteor 包* 菜单中，*使用包进行自定义*
