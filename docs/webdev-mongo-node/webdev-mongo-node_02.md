# 第二章：启动和运行

在本章中，我们将介绍设置开发环境所需的必要步骤。这些步骤包括以下内容：

+   在您的计算机上安装 Node.js

+   在您的计算机上安装 MongoDB

+   验证一切是否设置正确

仔细遵循这些部分，因为我们需要在跳转到实际编码的章节之前，开发环境已经准备就绪。在本书的其余部分中，我们将假定您使用的是 Mac OS X、Linux 或 Windows 7/Windows 8。您还需要在计算机上拥有超级用户和/或管理员权限，因为您将安装 Node 和 MongoDB 服务器。本章之后的代码和示例将是与操作系统无关的，并且应该在任何环境中工作，只要您提前采取了我概述的准备步骤。

您需要一个合适的文本编辑器来编写和编辑代码。虽然您选择的任何文本编辑器都可以满足此目的，但选择一个更好的文本编辑器将极大地提高您的生产力。Sublime Text 3 似乎是目前最受欢迎的文本编辑器，无论在哪个平台上。这是一个简单、轻量级的编辑器，由全球开发人员提供了无限的插件。如果您使用的是 Windows 机器，那么*Notepad++*也是一个不错的选择。此外，还有基于 JavaScript 的开源编辑器，如 Atom 和 Brackets，也值得一试。

最后，您需要访问命令行。Linux 和 Mac 可以通过终端程序访问命令行。Mac 上一个很好的替代品是 iTerm2 ([`iterm2.com`](http://iterm2.com))。对于 Windows，默认的命令行程序可以工作，但不是最好的。那里一个很好的替代品是 ConEmu ([`conemu.codeplex.com`](http://conemu.codeplex.com))。

在本书的其余部分，每当我提到命令行或命令提示符时，它看起来像下面这样：

```js
$ command -parameters -etc
```

# 安装 Node.js

可以通过访问官方 Node 网站并访问下载部分轻松获取 Node.js 安装程序。一旦进入那里，请确保根据您的操作系统和 CPU（32 位或 64 位）下载正确的版本。作为替代方案，您还可以使用特定于操作系统的软件包管理器进行安装。根据您使用的操作系统，只需跳转到特定的子部分，以获取有关要遵循的步骤的更多详细信息。

您可以通过以下链接跳转到 Node.js 下载部分：[`nodejs.org/en/download`](https://nodejs.org/en/download)。

# Mac OS X

Node 网站上有一个专门为 OS X 设计的通用安装程序。

我们需要按照以下步骤在 Mac 上安装 Node.js：

1.  访问 Node.js 官方网站的下载页面，如前所述，单击 Mac OS X 安装程序，这与处理器类型（32 位或 64 位）无关。

1.  下载完成后，双击`.pkg`文件，这将启动 Node 安装程序。

1.  按照向导的每一步进行操作，这应该是相当简单易懂的。

此外，如果您已安装了 OS X 软件包管理器之一，则无需手动下载安装程序。

您可以通过各自的软件包管理器安装 Node.js。

+   使用 Homebrew 进行安装：

```js
    brew install node
```

+   使用 Mac ports 进行安装：

```js
    port install nodejs
```

通过安装程序或软件包管理器安装 Node.js 时，将包括 npm。因此，我们不需要单独安装它。

# Windows

在 Windows 上安装 Node.js，我们将按照以下步骤进行：

1.  我们需要确定您的处理器类型，32 位还是 64 位。您可以通过在命令提示符下执行以下命令来执行此操作：

```js
    $ wmic os get osarchitecture
```

输出如下：

```js
      OSArchiecture
      64-bit
```

1.  根据此命令的结果下载安装程序。

1.  下载完成后，双击`.msi`文件，这将启动 Node 安装程序。

1.  按照向导的每一步进行操作。

1.  当您到达自定义设置屏幕时，您应该注意安装向导不仅会安装 Node.js 运行时，还会安装 npm 软件包管理器，并配置路径变量。

1.  因此，一旦安装完成，Node 和 npm 可以在任何文件夹中通过命令行执行。

此外，如果您安装了任何 Windows 软件包管理器，则无需手动下载安装程序。您可以通过相应的软件包管理器安装 Node.js：

+   使用 chocolatey：

```js
    cinst nodejs.install
```

+   使用`scoop`：

```js
    scoop install nodejs
```

# Linux

由于 Linux 有许多不同的版本和发行版，安装 Node 并不那么简单。但是，如果您一开始就在运行 Linux，那么您很清楚这一点，并且可能对一些额外的步骤感到满意。

Joyent 在如何使用许多不同的软件包管理器选项在 Linux 上安装 Node 的出色 wiki。这涵盖了几乎所有流行的`deb`和`rpm`-based 软件包管理器。您可以通过访问[`github.com/joyent/node/wiki/Installing-Node.js-via-package-manager`](https://github.com/joyent/node/wiki/Installing-Node.js-via-package-manager)阅读该 wiki。

以 Ubuntu 14.04 和之前版本为例，安装 Node 的步骤如下：

```js
$ sudo apt-get install python-software-properties $ sudo curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash - $ sudo apt-get install nodejs
```

完成这些步骤后，Node 和 npm 应该已安装在您的系统上。

# 测试 Node.js 是否正确安装

现在 Node 已经安装在您的系统上，让我们运行一个快速测试，以确保一切正常运行。

通过终端程序访问命令行并执行以下命令：

```js
    $ node --version
    v8.4.3
    $ npm --version
    5.3.0
```

假设您的 Node 安装成功，您应该在屏幕上看到安装的版本号作为输出，就在您执行的命令下面。

您的版本号很可能比之前打印的要新。

您还可以启动 Node `repl`，这是一个命令行 shell，可以让您直接执行 JavaScript：

```js
    $ node
    > console.log('Hello world!')
    Hello World!
    Undefined
    [press Ctrl-C twice to exit]  
```

# 在线文档

您需要确保将浏览器指向 Node 的在线文档并将其加为书签，因为它无疑会成为您经常访问的资源。您不一定要逐个阅读每个部分，但一旦开始在 Node.js 中编写代码，您将需要经常参考此文档，以更多地了解 Node.js 公开的 API。该文档可在[`nodejs.org/api/`](http://nodejs.org/api/)上找到。

还可以查看 npm 注册表，网址为[`npmjs.com`](http://npmjs.com)，在那里您可以找到数以万计的 Node 开发人员可用的模块。

# 安装 MongoDB

MongoDB 也可以通过访问官方 MongoDB 网站并从[`www.MongoDB.org/downloads`](http://www.mongodb.org/downloads)访问下载部分轻松下载。在那里，请务必根据您的操作系统和 CPU（32 位或 64 位）下载正确的版本。

对于 Windows 用户，您可以选择下载 MSI 安装程序文件，这将使安装更简单。

根据您下载的 MongoDB 版本，您将需要在以下部分中用适当的版本号替换`<version>`。

# Mac OS X 安装说明

如果您使用 Homebrew 软件包管理器，可以使用以下两个命令安装 MongoDB：

```js
    $ brew update
    $ brew install MongoDB
```

本章的其余部分假设您没有使用 Homebrew，并且需要手动安装 MongoDB。如果您通过 Homebrew 安装 MongoDB，可以直接转到*确认成功安装 MongoDB*部分。

下载完成后，打开并提取`.tgz`文件的内容。您需要将提取的内容移动到目标文件夹`/MongoDB`。您可以通过查找器或命令行执行此操作，具体取决于您的喜好，如下所示：

```js
    $ mkdir -p /MongoDB
    $ cd ~/Downloads
    $ cp -R -n MongoDB-osx-x86_64-2.4.9/ MongoDB 
```

您需要确保 MongoDB 二进制文件的位置已配置在您的环境路径中，以便您可以从任何工作目录执行`MongoDB`和 Mongo。要做到这一点，编辑您家目录（`~/`）中的`.profile`文件，并将 MongoDB 的位置追加到其中。您的`.profile`文件应该看起来像以下内容：

```js
export PATH=~/bin:/some/of/my/stuff:/more/stuff:/MongoDB/bin:$PATH
```

如果您没有这行或完全缺少`.bash_profile`，您可以通过执行以下命令轻松创建一个：

```js
    $ touch .bash_profile
    $ [edit] .bash_profile
    export PATH=$PATH:/MongoDB/bin
```

您很可能在前面的代码行中有比我更多的内容。重要的是在最后的`$PATH`之前添加`:/MongoDB/bin`。`:`是不同路径之间的分隔符（因此您可能会将您的路径添加到现有列表的末尾，但在结尾的`$PATH`之前）。

在这里，`mongod`指的是您需要调用的 MongoDB 服务器实例，`mongo`指的是 Mongo shell，它将是您与数据库交互的控制台。

接下来，您需要创建一个默认的`data`文件夹，MongoDB 将用它来存储所有数据文档。从命令行执行以下操作：

```js
    $ mkdir -p /data/db
    $ chown `id -u` /data/db
```

一旦文件已经正确解压到`/MongoDB`文件夹并且数据文件夹已创建，您可以通过从命令行执行以下命令来启动 MongoDB 数据库服务器：

```js
    $ mongod
```

这应该会在服务器启动时输出一大堆日志语句，但最终会以以下结束：

```js
2017-08-04T10:10:47.853+0530 I NETWORK [thread1] waiting for connections on port 27017
```

就是这样！您的 MongoDB 服务器已经启动并运行。您可以输入*Ctrl-C*来取消并关闭服务器。

# Windows 7/Windows 8 安装说明

完成下载后，MongoDB 网站将自动将您重定向到一个带有 Windows *快速入门*指南链接的页面：

[`docs.MongoDB.org/manual/tutorial/install-MongoDB-on-windows/`](http://docs.mongodb.org/manual/tutorial/install-mongodb-on-windows/)。

强烈建议您遵循该指南，因为它将是最新的，并且通常会比我在这里提供的更详细。

解压已下载的 ZIP 文件到根目录`c:\`。默认情况下，这应该会解压一个名为`MongoDB-osx-x86_64-2.4.9`的文件夹。根据您用于解压的工具，您可以保持原样，也可以将目标文件夹更改为`MongoDB`。如果在解压过程中没有更改目标文件夹，完成后应该将文件夹重命名。无论哪种方式，确保解压出的文件位于名为`c:\MongoDB`的文件夹中。

接下来，您需要创建一个默认的`data`文件夹，MongoDB 将用它来存储所有数据文档。使用 Windows 资源管理器或命令提示符，您最熟悉的方式创建`c:\data`文件夹，然后使用以下命令创建`c:\data\db`：

```js
    $ md data
    $ md data\db
```

一旦文件已经正确解压到`c:\MongoDB`文件夹，并且数据文件夹随后创建，您可以通过从提示符执行以下命令来启动 MongoDB 数据库服务器：

```js
$ c:\MongoDB\bin\mongod.exe
```

这应该会在服务器启动时输出一大堆日志语句，但最终会以以下结束：

```js
2017-08-04T10:10:47.853+0530 I NETWORK [thread1] waiting for connections on port 27017
```

就是这样！您的 MongoDB 服务器已经启动并运行。您可以输入*Ctrl*-*C*来取消并关闭服务器。

# Linux 安装说明

再次，与 Windows 或 Mac 相比，我们将面临稍微更具挑战性的 Linux 安装过程。官方网站[`docs.MongoDB.org/manual/administration/install-on-linux/`](http://docs.mongodb.org/manual/administration/install-on-linux/)上有关于如何在许多不同的 Linux 发行版上安装 MongoDB 的详细说明。

我们将继续使用 Ubuntu 作为我们的首选版本，并使用 APT 软件包管理器进行安装：

```js
    $ sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 
      7F0CEB10
    $ echo 'deb http://downloads-distro.MongoDB.org/repo/ubuntu-upstart 
     dist 10gen' | sudo tee /etc/apt/sources.list.d/MongoDB.list
    $ sudo apt-get update
    $ sudo apt-get install MongoDB-10gen
```

完成这些步骤后，MongoDB 应该已经安装并准备在您的系统上运行。在终端中执行以下命令以确保。这会启动 MongoDB 守护程序，并监听连接：

```js
    $ mongod
 2017-08-04T10:10:47.853+0530 I NETWORK [thread1] waiting for   
 connections on port 27017
```

成功！您的 MongoDB 服务器已经运行。您可以输入*Ctrl*-*C*来取消并关闭服务器。

由于您正在开发本地开发机器而不是生产服务器，您不需要 MongoDB 服务器始终运行。这将是对您的机器不必要的负担，因为大部分时间您不会与服务器进行开发。因此，在本书的其余部分，每次启动期望连接到 MongoDB 服务器的代码时，您都需要手动启动服务器。如果您愿意，您当然可以配置 MongoDB 在本地作为服务运行并始终运行，但是如何配置超出了本章的范围。

# 确认 MongoDB 安装成功

现在 MongoDB 已经安装在您的系统上，让我们运行一个快速测试，确保一切正常运行。

通过终端程序访问命令行并执行以下命令：

```js
    $ mongod --version
    db version v3.4.4
    $ mongo --version
    MongoDB shell version v3.4.4  
```

假设您的 MongoDB 安装成功，您应该在屏幕上看到安装的版本号作为输出。

您的版本号很可能比之前打印的要更新。

# 将在线文档加为书签

您需要确保将浏览器指向 MongoDB 的在线文档，网址为[`docs.MongoDB.org/manual/`](http://docs.mongodb.org/manual/)，并将其加为书签，因为它无疑将成为您经常访问的资源。

# 编写您的第一个应用程序

现在您已经安装了所有内容并确认一切正常运行，您可以编写您的第一个快速应用程序，该应用程序将同时使用 Node 和 MongoDB。这将证明您的环境已经准备就绪，并且您已经准备好开始。此外，这将让您简单了解 Node 和 MongoDB 开发的世界！如果以下内容让您感到困惑或不合理，不要担心，本书的其余部分将会澄清一切！

首先，我们需要为我们的应用程序创建一个文件夹，该文件夹将包含此应用程序的特定代码，如下所示：

```js
    $ mkdir testapp
    $ cd testapp
```

# 创建示例应用程序

我们刚刚创建的`testapp`文件夹将是我们示例 Node 应用程序的根目录。虽然这不是必需的，但是创建`package.json`文件对我们的 Node 应用程序非常重要，这将包含有关应用程序的必要数据，如版本、名称、描述、开发和运行时依赖项。可以通过从`testapp`文件夹根目录发出以下命令来完成：

```js
    $ npm init 
```

这个命令将会询问您一些问题，比如您新创建的应用的名称和版本号。您不需要一次填写所有细节，可以通过按下*Enter*跳过步骤，系统将使用默认值，稍后您可以更新。

# 准备依赖模块

在我们开始编写任何 Node.js 代码之前，我们需要使用`npm`安装我们的依赖项。由于这是一个基本应用程序，我们将使用它来测试我们的 Node.js 与 MongoDB 服务器的连接。因此，我们唯一需要的依赖模块是 Node.js 的原生 MongoDB 客户端。我们可以通过执行以下命令轻松安装：

```js
    $ npm install MongoDB --save
```

在`npm`安装 MongoDB 驱动程序后，您可以列出目录的内容，您会注意到一个新文件夹被创建，名为`node_modules`。这是所有 Node 模块的存储位置，当您从`npm`安装它们时，它们会存储在这里。在`node_modules`文件夹中，应该有一个名为`MongoDB`的单个文件夹。此外，您会注意到我们示例应用程序的`package.json`文件将被此新依赖项条目更新。

# 添加应用程序代码

现在，让我们编写简单的应用程序代码来测试一下。这个应用程序基本上会连接到我们本地运行的 MongoDB 服务器，插入一些记录作为种子数据，然后提供输出，告诉我们数据是否被正确插入到 MongoDB 中。你可以通过以下 URL 下载代码的 Gist：[`bit.ly/1JpT8QL`](http://bit.ly/1JpT8QL)。

使用你喜欢的编辑器，创建一个名为`app.js`的新文件，并将其保存到应用程序根目录，即`testapp`文件夹。只需将上面 Gist 的内容复制到`app.js`文件中。

# 理解代码

现在，让我们逐个解释代码的每个部分在做什么。

```js
    //require the mongoClient from MongoDB module 
    var MongoClient = require('MongoDB').MongoClient;  
```

上面的一行需要我们通过`npm`安装的 MongoDB Node 驱动程序。这是 Node.js 中用于将外部文件依赖项引入当前上下文文件的必需约定。

我们将在接下来的章节中更详细地解释这一点。

```js
//MongoDB configs  
var connectionUrl = 'MongoDB://localhost:27017/myproject',  
    sampleCollection = 'chapters'; 
```

在上面的代码中，我们声明了要使用的数据库服务器信息和集合的变量。在这里，`myproject`是我们想要使用的数据库，`chapters`是集合。在 MongoDB 中，如果你引用并尝试使用一个不存在的集合，它将自动被创建。

下一步将是定义一些数据，我们可以将其插入到 MongoDB 中以验证一切是否正常。因此，我们在这里创建了一个章节的数组，可以将其插入到我们在前面步骤中设置的数据库和集合中：

```js
//We need to insert these chapters into MongoDB 
var chapters = [{  
    'Title': 'Snow Crash',  
    'Author': 'Neal Stephenson'  
},{  
    'Title': 'Snow Crash',  
    'Author': 'Neal Stephenson'  
}]; 
```

现在，我们可以看一下其余的代码，我们将这些数据插入到 MongoDB 数据库中：

```js
MongoClient.connect(connectionUrl, function(err, db) {    
  console.log("Connected correctly to server");     
  // Get some collection  
  var collection = db.collection(sampleCollection);   
  collection.insert(chapters,function(error,result){     
    //here result will contain an array of records inserted  
    if(!error) {  
      console.log("Success :"+result.ops.length+" chapters 
 inserted!");  
    } else {  
      console.log("Some error was encountered!");  
    }     
    db.close();    
  });    
}); 
```

在这里，我们与 MongoDB 服务器建立连接，如果连接正常，`db`变量将拥有我们可以用于进一步操作的`connection`对象：

```js
MongoClient.connect(url, function(err, db) {   
```

仔细看一下上面的代码-你还记得我们在第一章中学到的内容吗？我们在这里为我们进行的`connection`调用使用了一个`callback`。正如在第一章中讨论的，这个函数将被注册为一个`callback`，一旦连接尝试完成，就会被触发。连接完成后，这将由`error`或`db`对象触发，具体取决于我们是否能够建立正确的连接。因此，如果你看一下`callback`函数中的代码，我们在记录正确连接到服务器之前并没有检查是否有任何错误。现在，这就是你要在我们尝试运行这个应用程序时添加和检查的任务！看一下本节中以下代码块：

```js
var collection = db.collection(sampleCollection); 
collection.insert(chapters,function(error,result){ 
```

这只是使用我们在连接调用中得到的`db`对象，并获取名为`chapters`的`collection`。记住，我们在代码开头将该值设置为`sampleCollection`。一旦我们获得了`collection`，我们就会进行`insert`调用，将我们在数组`chapters`中定义的章节放入其中。正如你所看到的，这个`insert`调用也是通过附加`callback`函数来进行的，这是一个异步调用。一旦 MongoDB 原生客户端中的代码完成了`insert`操作，这个`callback`函数就会被触发，而我们将其作为一个依赖项来使用。

接下来，我们将看一下我们传递给`insert`函数调用的`callback`函数中的代码：

```js
if(!error) {  
  console.log("Success :"+result.ops.length+" chapters 
               inserted!");  
} else {  
  console.log("Some error was encountered!");  
}     
db.close(); 
```

在这里，我们处理通过`callback`传递的值，以找出`insert`操作是否成功，以及已插入记录相关的数据。因此，我们检查是否有错误，如果没有，就继续打印插入的记录数。在这里，如果操作成功，结果数组将包含我们插入到 MongoDB 中的记录。

现在我们可以继续尝试运行这段代码，因为我们已经理解了它的作用。

# 启动示例应用程序

一旦您将完整的代码保存到`app.js`中，就可以执行它并查看发生了什么。但是，在启动明显依赖于与 MongoDB 的连接的应用程序之前，您需要首先启动 MongoDB 守护程序实例：

```js
    $ mongod
```

在 Windows 中，如果您尚未为`mongod`设置`PATH`变量，则在执行 MongoDB 时可能需要使用完整路径，即`c:\MongoDB\bin\mongod.exe`。对于您的需求，本书的其余部分将引用`mongod`命令，但您可能始终需要在每个实例中执行完整路径。

现在，要启动应用程序本身，请在`app.js`所在的`root`文件夹中执行以下命令：

```js
    $ node app.js
```

当应用程序首次执行时，您应该会看到以下内容：

```js
    Connected correctly to server
    Success :2 chapters inserted!  
```

# 检查实际数据库

让我们快速查看一下数据库本身，看看在应用程序执行过程中发生了什么。由于服务器目前正在运行，我们可以使用 Mongo shell 连接到它-这是 MongoDB 服务器的命令行界面。执行以下命令以使用 Mongo 连接到服务器并针对章节集合运行查询。正如您在即将看到的代码中，Mongo shell 最初连接到名为`test`的默认数据库。如果要切换到其他数据库，我们需要手动指定数据库名称：

```js
    $ mongo
    MongoDB shell version: 2.4.8
    connecting to: test
    > use myproject
    > show collections
    chapters
    system.indexes
    > db.chapters.find().pretty()  
```

在这里，`pretty`被用作命令的一部分，用于格式化`find`命令的结果。这仅在 shell 上下文中使用。它对 JSON 执行更多的美化任务。

您应该会看到类似以下输出的内容：

```js
{  
    'id' : ObjectId("5547e734cdf16a5ca59531a7"), 
    'Title': 'Snow Crash',  
    'Author': 'Neal Stephenson'  
}, 
{  
    'id' : ObjectId("5547e734cdf16a5ca59531a7"), 
    'Title': 'Snow Crash',  
    'Author': 'Neal Stephenson' 
} 
```

如果再次运行 Node 应用程序，记录将再次插入 Mongo 服务器。因此，如果重复执行命令多次，输出中将有更多的记录。在本章中，我们没有处理这种情况，因为我们打算只有特定的代码，这将足够简单易懂。

# 摘要

在本章中，我们花时间确保您的开发环境正确配置了 Node 运行环境和 MongoDB 服务器。在确保两者都正确安装后，我们编写了一个利用了这两种技术的基本应用程序。该应用程序连接到本地运行的 MongoDB 服务器，并插入了示例记录。

现在，繁琐但必要的设置和安装任务已经完成，我们可以继续一些有趣的事情并开始学习了！

在下一章中，我们将回顾 JavaScript 语言的入门知识，并了解 Node 的基础知识。然后，我们将使用 Mongo shell 回顾 MongoDB 的基本**CRUD**（`create`，`read`，`update`，`delete`）操作。
