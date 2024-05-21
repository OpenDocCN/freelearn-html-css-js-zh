# 第二章：IoTFW.js - I

在本章和第三章，*IoTFW.js - II*中，我们将开发一个用于构建各种物联网解决方案的参考架构。该参考架构或物联网框架将作为我们未来在本书中要开发的物联网解决方案的基础。我们将称这个参考架构或框架为 IoTFW.js。我们将致力于以下主题，以使 IoTFW.js 生动起来：

+   设计 IoTFW.js 架构

+   开发基于 Node.js 的服务器端层

+   开发一个基于 Angular 4 的 Web 应用程序

+   开发一个基于 Ionic 3 的移动应用程序

+   开发一个基于 Angular 4 和 Electron.js 的桌面应用程序

+   在树莓派 3 上设置和安装所需的依赖关系

+   整合所有的部分

我们将在本章中涵盖一些先前的主题，以及第三章，*IoTFW.js - II*中的一些主题。

# 设计一个参考架构

正如我们在第一章，*物联网的世界*中所看到的，我们将要处理的所有示例都有一个共同的设置。那就是硬件、固件（在硬件上运行的软件）、代理、API 引擎和用户应用程序。

我们将在遇到相关框架的部分时进行扩展。

在需要时，我们将扩展硬件、移动应用程序或 API 引擎。

通过这个参考架构，我们将在现实世界中的设备与虚拟世界中的云之间建立一个管道。换句话说，物联网是设备和互联网之间的最后一英里解决方案。

# 架构

使用树莓派、Wi-Fi 网关、云引擎和用户界面应用程序组合在一起的简单参考架构如下图所示：

![](img/00008.jpeg)

在非常高的层面上，我们在左侧有智能设备，在右侧有用户设备。它们之间的所有通信都通过云端进行。

以下是先前架构中每个关键实体的描述。我们将从左侧开始，向右移动。

# 智能设备

智能设备是硬件实体，包括传感器、执行器或两者，任何微控制器或微处理器，在我们的案例中是树莓派 3。

传感器是一种可以感知或测量物理特性并将其传回微控制器或微处理器的电子元件。传回的数据可以是周期性的或事件驱动的；事件驱动的意思是只有在数据发生变化时才会传回。温度传感器，如 LM35 或 DHT11，就是传感器的一个例子。

执行器也是可以触发现实世界中动作的电机械组件。通常，执行器不会自行行动。微控制器、微处理器或电子逻辑发送信号给执行器。执行器的一个例子是机械继电器。

我们在本书中提到的微处理器将是树莓派 3。

树莓派 3 是由树莓派基金会设计和开发的单板计算机。树莓派 3 是第三代树莓派。

在本书中，我们将使用树莓派 3 型 B 来进行所有示例。树莓派 3 型 B 的一些规格如下：

| **特性** | **规格** |
| --- | --- |
| 代 | 3 |
| 发布日期 | 2016 年 2 月 |
| 架构 | ARMv8-A（64/32 位） |
| 系统芯片（SoC） | Broadcom BCM2837 |
| CPU | 1.2 GHz 64 位四核 ARM Cortex-A53 |
| 内存（SDRAM） | 1 GB（与 GPU 共享） |
| USB 2.0 端口 | 4（通过机载 5 端口 USB 集线器） |
| 机载网络 | 10/100 兆比特/秒以太网，802.11n 无线，蓝牙 4.1 |
| 低级外围设备 | 17× GPIO 加上相同的特定功能，和 HAT ID 总线 |
| 功率评级 | 空闲时平均 300 毫安（1.5 瓦），在压力下最大 1.34 安（6.7 瓦）（监视器、键盘、鼠标和 Wi-Fi 连接） |
| 电源 | 通过 MicroUSB 或 GPIO 引脚的 5V |

有关规格的更多信息，请参考树莓派的规格：[`en.wikipedia.org/wiki/Raspberry_Pi#Specifications`](https://en.wikipedia.org/wiki/Raspberry_Pi#Specifications)。

# 网关

我们架构中的下一个部分是 Wi-Fi 路由器。一个普通的家用 Wi-Fi 路由器将作为我们的网关。正如我们在第一章中所看到的，在*物联网的世界*中，*集群设备与独立设备*部分，我们遵循独立设备的方法，其中每个设备都是自给自足的，并且有自己的无线电与外界通信。我们将要构建的所有项目都包括一个树莓派 3，它具有微处理器以及与传感器和执行器进行通信的无线电。

# MQTTS 代理

我们参考框架中的下一个重要部分是设备和云之间的安全通信通道。我们将使用 MQTT 作为我们的通信通道。MQTT 在以下引用中描述：[`mqtt.org/faq`](http://mqtt.org/faq)。

MQTT 代表 MQ 遥测传输。这是一种发布/订阅、非常简单和轻量级的消息传递协议，设计用于受限设备和低带宽、高延迟或不可靠网络。设计原则是尽量减少网络带宽和设备资源需求，同时也尝试确保可靠性和一定程度的传递保证。

我们将使用 MQTT over SSL 或 MQTTS。在我们的架构中，我们将使用 Mosca（[`www.mosca.io/`](http://www.mosca.io/)）作为我们的 MQTTS 代理。Mosca 是一个 Node.js MQTT 代理。当我们开始使用它时，我们将更多地谈论 Mosca。

# API 引擎

API 引擎是一个基于 Node.js、Express 编写的 Web 服务器应用，具有 MongoDB 作为持久化层。该引擎负责与 Mosca 通信作为 MQTT 客户端，将数据持久化到 MongoDB，并使用 Express 公开 API。然后应用程序使用这些 API 来显示数据。

我们还将实现基于套接字的 API，用于用户界面在设备和服务器之间实时获取通知。

# MongoDB

我们将使用 MongoDB 作为我们的数据持久化层。MongoDB 是一个 NoSQL 文档数据库，允许我们在一个集合中保存具有不同模式的文档。这种数据库非常适合处理来自各种设备的传感器数据，因为数据结构或参数因解决方案而异。要了解有关 MongoDB 的更多信息，请参阅[`www.mongodb.com/`](https://www.mongodb.com/)。

# Web 应用

Web 应用是一个简单的 Web/移动 Web 界面，将实现 API 引擎公开的 API。这些 API 将包括身份验证，访问特定的智能设备，获取智能设备的最新数据，并通过 API 将数据发送回智能设备。我们将使用 Angular 4（[`angular.io/`](https://angular.io/)）和 Twitter Bootstrap 3（[`getbootstrap.com/`](http://getbootstrap.com/)）技术来构建 Web 应用。

# 移动应用

我们将采用混合移动方法来构建我们的移动应用。移动应用实现了 API 引擎公开的 API。这些 API 将包括身份验证，访问特定的智能设备，获取智能设备的最新数据，并通过 API 将数据发送回智能设备。我们将使用 Ionic 3（[`ionicframework.com/`](http://ionicframework.com/)），它由 Angular 4 提供支持，来构建移动应用。

# 桌面应用

我们将采用桌面混合方法来构建我们的桌面应用程序。桌面应用程序将实现 API 引擎提供的 API。这些 API 将包括身份验证，访问特定的智能设备，从智能设备获取最新数据，并通过 API 将数据发送回智能设备。我们将使用 Electron ([`electron.atom.io/`](https://electron.atom.io/))作为构建桌面应用程序的外壳。我们将使用 Angular 4 和 Twitter Bootstrap 3 ([`getbootstrap.com/`](http://getbootstrap.com/))技术来构建桌面应用程序。我们会尽量在 Web 和桌面应用程序之间重用尽可能多的代码。

# 数据流

现在我们了解了架构的各个部分，我们现在将看一下组件之间的数据流。我们将讨论从智能设备到应用程序以及反之的数据流。

# 智能设备到应用程序

从传感器到用户设备的简单数据流程如下：

![](img/00009.jpeg)

从上图可以看出，数据源自传感器；树莓派 3 读取这些数据，并通过 Wi-Fi 路由器将数据发布到 MQTTS 代理（Mosca）。一旦代理接收到数据，它将把数据发送到 API 引擎，API 引擎将数据持久化到数据库中。数据成功保存后，API 引擎将把新数据发送到我们的应用程序，以实时显示数据。

这里需要注意的一点是，API 引擎将充当 MQTT 客户端，并订阅设备发布数据的主题。我们将在实施时查看这些主题。

一般来说，在这种流程中的数据将是典型的传感器传输数据。

# 应用程序到智能设备

以下图表显示了数据如何从应用程序流向智能设备：

![](img/00010.jpeg)

从上图可以看出，如果应用程序希望向智能设备发送指令，它会将该消息发送到 API 引擎。然后 API 引擎将数据持久化到数据库，并将数据发布到 MQTTS 代理，以传递给设备。设备将根据这些数据做出反应。

请注意，在这两种流程中，MQTTS 代理管理设备，API 引擎管理应用程序。

# 构建参考架构

在本节中，我们将开始组合所有部件并组合所需的设置。我们将从 Node.js 安装开始，然后是数据库，之后，继续其他部件。

# 在服务器上安装 Node.js

在继续开发之前，我们需要在服务器上安装 Node.js。这里的服务器可以是您自己的台式机、笔记本电脑、AWS 机器，或者是一个可能具有或不具有公共 IP 的 digitalocean 实例（[`www.iplocation.net/public-vs-private-ip-address`](https://www.iplocation.net/public-vs-private-ip-address)）。

安装 Node.js，转到[`nodejs.org/en/`](https://nodejs.org/en/)并下载适合您的机器的版本。安装完成后，您可以通过从命令提示符/终端运行以下命令来测试安装：

```js
node -v
# v6.10.1
```

和

```js
npm -v
# 3.10.10  
```

您可能拥有比之前显示的版本更新的版本。

现在我们有了所需的软件，我们将继续。

# 安装 nodemon

现在我们已经安装了 Node.js，我们将安装 nodemon。这将负责自动重新启动我们的节点应用程序。运行：

```js
npm install -g nodemon
```

# MongoDB

您可以按照以下列出的两种方式之一设置数据库。

# 本地安装

我们可以在服务器上设置 MongoDB 作为独立安装。这样，数据库就在服务器上运行，并且数据会持久保存在那里。

根据您的操作系统，您可以按照提供的说明在[`docs.mongodb.com/manual/installation/`](https://docs.mongodb.com/manual/installation/)上设置数据库。

安装完数据库后，为了测试一切是否正常工作，您可以打开一个新的终端，并通过运行以下命令启动 Mongo 守护进程：

```js
mongod  
```

您应该看到类似于以下内容：

![](img/00011.jpeg)

我正在使用默认端口`27017`运行数据库。

现在我们将使用 mongo shell 与数据库进行交互。打开一个新的命令提示符/终端，并运行以下命令：

```js
mongo  
```

这将带我们进入`mongo` shell，通过它我们可以与 MongoDB 进行交互。以下是一些方便的命令：

| **描述** | **命令** |
| --- | --- |
| 显示所有数据库 | `show dbs` |
| 使用特定数据库 | `use local` |
| 创建数据库 | `use testdb` |
| 检查正在使用的数据库 | `db` |
| 创建集合 | `db.createCollection("user");` |
| 显示数据库中的所有集合 | `show collections` |
| （创建）在集合中插入文档 | `db.user.insert({"name":"arvind"});` |
| （读取）查询集合 | `db.user.find({});` |
| （更新）修改集合中的文档 | `db.user.update({"name": "arvind"}, {"name" : "arvind2"}, {"upsert":true});` |
| （删除）删除文档 | `db.user.remove({"name": "arvind2"});` |

使用前面的命令，您可以熟悉 Mongo shell。在我们的 API 引擎中，我们将使用 Mongoose ODM（[`mongoosejs.com/`](http://mongoosejs.com/)）来管理 Node.js/Express--API 引擎。

# 使用 mLab

如果您不想费心在本地设置数据库，可以使用 mLab（[`mlab.com/`](https://mlab.com/)）等 MongoDB 作为服务。在本书中，我将遵循这种方法。我将使用 mLab 的实例，而不是本地数据库。

要设置 mLab MongoDB 实例，首先转到[`mlab.com/login/`](https://mlab.com/login/)并登录。如果您没有帐户，可以通过转到[`mlab.com/signup/`](https://mlab.com/signup/)来创建一个。

mLab 有一个免费的层，我们将利用它来构建我们的参考架构。免费层非常适合像我们这样的开发和原型项目。一旦我们完成了实际的开发，并且准备好生产级应用程序，我们可以考虑一些更可靠的计划。您可以在[`mlab.com/plans/pricing/`](https://mlab.com/plans/pricing/)上了解定价。

一旦您登录，单击“创建新”按钮以创建新的数据库。现在，在云提供商下选择亚马逊网络服务，然后选择计划类型为 FREE，如下图所示：

![](img/00012.jpeg)

最后，将数据库命名为`iotfwjs`，然后单击“创建”。然后，几秒钟后，我们应该为我们创建一个新的 MongoDB 实例。

一旦数据库创建完成，打开`iotfwjs`数据库。我们应该看到一些警告：一个是指出这个沙箱数据库不应该用于生产，我们知道这一点，另一个是没有数据库用户存在。

所以，让我们继续创建一个。单击“用户”选项卡，然后单击“添加数据库用户”按钮，并使用用户名`admin`和密码`admin123`填写表单，如下所示：

![](img/00013.jpeg)

您可以选择自己的用户名和密码，并相应地在本书的其余部分进行更新。

现在来测试连接到我们的数据库，使用页面顶部的部分使用`mongo` shell 进行连接。在我的情况下，如下所示：

![](img/00014.jpeg)

打开一个新的命令提示符，并运行以下命令（在相应地更新 mLab URL 和凭据之后）：

```js
mongo ds241055.mlab.com:41055/iotfwjs -u admin -p admin123  
```

我们应该能够登录到 shell，并可以从这里运行查询，如下所示：

![](img/00015.jpeg)

这完成了我们的 MongoDB 设置。

# MQTTS 代理 - Mosca

在本节中，我们将组装 MQTTS 代理。我们将使用 Mosca ([`www.mosca.io/`](http://www.mosca.io/))作为独立服务 ([`github.com/mcollina/mosca/wiki/Mosca-as-a-standalone-service`](https://github.com/mcollina/mosca/wiki/Mosca-as-a-standalone-service))。

创建一个名为`chapter2`的新文件夹。在`chapter2`文件夹内，创建一个名为`broker`的新文件夹，并在该文件夹内打开一个新的命令提示符/终端。然后运行以下命令：

```js
npm install mosca pino -g  
```

这将全局安装 Mosca 和 Pino。Pino ([`github.com/pinojs/pino`](https://github.com/pinojs/pino))是一个 Node.js 日志记录器，它记录 Mosca 抛出的所有消息到控制台。

现在，Mosca 的默认版本实现了 MQTT。但我们希望在智能设备和云之间保护我们的通信，以避免中间人攻击。

因此，为了设置 MQTTS，我们需要 SSL 密钥和 SSL 证书。为了在本地创建 SSL 密钥和证书，我们将使用`openssl`。

要检查您的计算机上是否存在`openssl`，运行`openssl version -a`，您应该看到关于您的`openssl`本地安装的信息。

如果您没有`openssl`，您可以从[`www.openssl.org/source/`](https://www.openssl.org/source/)下载。

现在，在`broker`文件夹内，创建一个名为`certs`的文件夹，并`cd`进入该文件夹。运行以下命令生成所需的密钥和证书文件：

```js
openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out certificate.pem  
```

这将提示一些问题，您可以按照以下方式填写相同的内容：

![](img/00016.jpeg)

这将在`certs`文件夹内创建两个名为`key.pem`和`certificate.pem`的新文件。我们将在我们的 Mosca 设置中使用这些文件。

接下来，在`broker`文件夹的根目录下，创建一个名为`index.js`的新文件，并按以下方式更新它：

```js
let SSL_KEY = __dirname + '/certs/key.pem';
let SSL_CERT = __dirname + '/certs/certificate.pem';
let MONGOURL = 'mongodb://admin:admin123@ds241055.mlab.com:41055/iotfwjs';

module.exports = {
    id: 'broker',
    stats: false,
    port: 8443,
    logger: {
        name: 'iotfwjs',
        level: 'debug'
    },
    secure: {
        keyPath: SSL_KEY,
        certPath: SSL_CERT,
    },
    backend: {
        type: 'mongodb',
        url: MONGOURL
    },
    persistence: {
        factory: 'mongo',
        url: MONGOURL
    }
};
```

前面的代码是我们将用于启动 Mosca 的配置。此配置加载 SSL 证书和密钥，并将 Mongo 设置为我们的持久层。

保存`index.js`，然后返回到终端/提示符，并`cd`到我们有`index.js`文件的位置。接下来，运行以下命令：

```js
mosca -c index.js -v | pino  
```

我们应该看到以下内容：

![](img/00017.jpeg)

正如您从前面看到的，我们连接到`iotfwjs`数据库，代理将监听端口`8883`以进行连接。

这完成了我们使用 Mosca 设置 MQTTS 代理的设置。

在下一步中，我们将实现 API 引擎，此时我们将测试 MQTTS 代理与 API 引擎的集成。

# API 引擎 - Node.js 和 Express

在本节中，我们将构建 API 引擎。该引擎与我们的应用程序进行交互，并将智能设备的信息从和到代理进行级联连接。

要开始，我们将克隆一个使用 Yeoman ([`yeoman.io/`](http://yeoman.io/))生成器创建的存储库，名为`generator-node-express-mongo` ([`www.npmjs.com/package/generator-node-express-mongo`](https://www.npmjs.com/package/generator-node-express-mongo))。我们已经使用`generator-node-express-mongo`生成的代码并根据我们的需求进行了修改。

在您的计算机上的某个位置，使用以下命令下载本书的完整代码库：

```js
git clone https://github.com/PacktPublishing/Practical-Internet-of-Things-with-JavaScript.git
```

或者，您也可以从[`github.com/PacktPublishing/Practical-Internet-of-Things-with-JavaScript`](https://github.com/PacktPublishing/Practical-Internet-of-Things-with-JavaScript)下载 zip 文件。

一旦存储库被下载，`cd`进入`base`文件夹，并将`api-engine-base`文件夹复制到`chapter2`文件夹中。

这将下载`api-engine`样板代码。一旦`repo`被克隆，`cd`进入文件夹并运行以下命令：

```js
npm install  
```

这将安装所需的依赖项。

如果我们打开`cloned`文件夹，我们应该看到以下内容：

```js
.
├── package.json
└── server
    ├── api
    │ └── user
    │ ├── index.js
    │ ├── user.controller.js
    │ ├── user.model.js
    ├── app.js
    ├── auth
    │ ├── auth.service.js
    │ ├── index.js
    │ └── local
    │ ├── index.js
    │ └── passport.js
    ├── config
    │ ├── environment
    │ │ ├── development.js
    │ │ ├── index.js
    │ │ ├── production.js
    │ │ └── test.js
    │ ├── express.js
    │ └── socketio.js
    ├── mqtt
    │ └── index.js
    └── routes.js
```

该文件夹包含我们启动 API 引擎所需的所有基本内容。

从前面的结构中可以看出，我们在文件夹的根目录中有一个`package.json`。这个文件包含了所有需要的依赖项。我们还在这里定义了我们的启动脚本。

我们的所有应用程序文件都位于`server`文件夹中。一切都始于`api-engine/server/app.js`。我们初始化`mongoose`、`express`、`socketio`、`config`、`routes`和`mqtt`。最后，我们启动服务器，并在`localhost`的`9000`端口上侦听，借助`server.listen()`。

`api-engine/server/config/express.js`具有初始化 Express 中间件所需的设置。`api-engine/server/config/socketio.js`包含管理 Web 套接字所需的逻辑。

我们将使用`api-engine/server/config/environment`来配置环境变量。在大部分的书中，我们将使用开发环境。如果我们打开`api-engine/server/config/environment/development.js`，我们应该看到`mongo`和`mqtt`的配置。更新它们如下：

```js
// MongoDB connection options
    mongo: {
        uri: 'mongodb://admin:admin123@ds241055.mlab.com:41055/iotfwjs'
    },

    mqtt: {
        host: process.env.EMQTT_HOST || '127.0.0.1',
        clientId: 'API_Server_Dev',
        port: 8883
    }
};
```

根据您的设置更新 mongo URL（mLab 或本地）。由于我们将连接到在本地计算机上运行的 Mosca 代理，我们使用`127.0.0.1`作为主机。

# 授权

接下来，我们将看看开箱即用的身份验证。我们将使用**JSON Web Tokens**（**JWTs**）来验证要与我们的 API 引擎通信的客户端。我们将使用 Passport（[`passportjs.org/`](http://passportjs.org/)）进行身份验证。

打开`api-engine/server/auth/index.js`，我们应该看到使用`require('./local/passport').setup(User, config);`设置护照，并且我们正在创建一个新的身份验证路由。

路由在`api-engine/server/routes.js`中配置。如果我们打开`api-engine/server/routes.js`，我们应该看到`app.use('/auth', require('./auth'));`。这将创建一个名为`/auth`的新端点，在`api-engine/server/auth/index.js`中，我们已经添加了`router.use('/local', require('./local'));`现在，如果我们想要访问`api-engine/server/auth/local/index.js`中的`POST`方法，我们将向`/auth/local`发出 HTTP `POST`请求。

在`api-engine`中，我们使用护照本地认证策略（[`github.com/jaredhanson/passport-local`](https://github.com/jaredhanson/passport-local)）来使用 MongoDB 进行持久化验证用户。

要创建新用户，我们将使用用户 API。如果我们打开`api-engine/server/routes.js`，我们应该看到定义了一个路由来访问用户集合`app.use('/api/v1/users', require('./api/user'));`。我们已经添加了`/api/v1/users`前缀，以便稍后可以对我们的 API 层进行版本控制。

如果我们打开`api-engine/server/api/user/index.js`，我们应该看到定义了以下六个路由：

+   `router.get('/', auth.hasRole('admin'), controller.index);`

+   `router.delete('/:id', auth.hasRole('admin'), controller.destroy);`

+   `router.get('/me', auth.isAuthenticated(), controller.me);`

+   `router.put('/:id/password', auth.isAuthenticated(), controller.changePassword);`

+   `router.get('/:id', auth.isAuthenticated(), controller.show);`

+   `router.post('/', controller.create);`

第一个路由是用于获取数据库中所有用户的路由，并使用`api-engine/server/auth/auth.service.js`中定义的`auth.hasRole`中间件，我们将检查用户是否经过身份验证并具有管理员角色。

下一步是删除具有 ID 的用户的路由；之后，我们有一个根据令牌获取用户信息的路由。我们有一个`PUT`路由来更新用户的信息；一个`GET`路由根据用户 ID 获取用户的信息；最后，一个`POST`路由来创建用户。请注意，`POST`路由没有任何身份验证或授权中间件，因为访问此端点的用户将是第一次使用我们的应用程序（或者正在尝试注册）。

使用`POST`路由，我们将创建一个新用户；这就是我们注册用户的方式：`api-engine/server/api/user/user.model.js`包含了用户的 Mongoose 模式，`api-engine/server/api/user/user.controller.js`包含了我们定义的路由的逻辑。

# MQTT 客户端

最后，我们将看一下 MQTT 客户端与我们的`api-engine`的集成。如果我们打开`api-engine/server/mqtt/index.js`，我们应该会看到 MQTTS 客户端的默认设置。

我们使用以下配置来连接到 MQTTS 上的 Mosca 经纪人：

```js
var client = mqtt.connect({
    port: config.mqtt.port,
    protocol: 'mqtts',
    host: config.mqtt.host,
    clientId: config.mqtt.clientId,
    reconnectPeriod: 1000,
    username: config.mqtt.clientId,
    password: config.mqtt.clientId,
    keepalive: 300,
    rejectUnauthorized: false
});
```

我们正在订阅两个事件：一个是在连接建立时，另一个是在接收消息时。在`connect`事件上，我们订阅了一个名为`greet`的主题，并在下一行发布了一个简单的消息到该主题。在`message`事件上，我们正在监听来自经纪人的任何消息，并打印主题和消息。

通过这样，我们已经了解了与`api-engine`一起工作所需的大部分代码。要启动`api-engine`，`cd`进入`chapter2/api-engine`文件夹，并运行以下命令：

```js
npm start  
```

这将在端口`9000`上启动一个新的 Express 服务器应用程序。

# API 引擎测试

为了快速检查我们创建的 API，我们将使用一个名为 Postman 的 Chrome 扩展。您可以从这里设置 Chrome 扩展：[`chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop?hl=en`](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop?hl=en)。

一旦 Postman 设置好，我们将测试两个 API 调用以验证注册和登录方法。

打开 Postman，并输入请求的 URL 为`http://localhost:9000/api/v1/users`。接下来，选择方法类型为`POST`。完成后，我们将设置标头。添加一个新的标头，键为`content-type`，值为`application/json`。

现在我们将构建请求体/有效载荷。点击 Headers 旁边的 Body 选项卡，选择 Raw request。并更新为以下内容：

```js
{ 
   "email" : "arvind@myapp.com", 
   "password" : "123456", 
   "name" : "Arvind" 
} 
```

您可以根据需要更新数据。然后点击发送。这将向 API 引擎发出请求，API 引擎将将数据保存到数据库，并以新用户对象和授权令牌作出响应。

我们的输出应该如下：

![](img/00018.jpeg)

现在，如果我们再次点击发送按钮并使用相同的数据，我们应该会看到一个验证错误，类似于以下内容：

![](img/00019.jpeg)

现在，为了验证新注册的用户，我们将向`http://localhost:9000/auth/local`发出请求，只包含电子邮件和密码。我们应该会看到类似以下内容的东西：

![](img/00020.jpeg)

这验证了我们创建的 API。

通过这样，我们完成了 API 引擎的演练。在下一节中，我们将集成`api-engine`与经纪人，并测试它们之间的连接。

# 经纪人和 API 引擎之间的通信

现在我们在云上完成了这两个软件，我们将对它们进行接口化。在`api-engine/server/config/environment/development.js`中，我们已经定义了`api-engine`需要连接到的经纪人 IP 和端口。

稍后，如果我们将这两个部分部署到不同的机器上，这就是我们更新 IP 和端口的地方，以便`api-engine`引用经纪人。

现在，为了测试通信，`cd`进入`chapter2/broker`文件夹，并运行以下命令：

```js
mosca -c index.js -v | pino  
```

我们应该会看到以下内容：

![](img/00021.jpeg)

接下来，打开一个新的命令提示符/终端，`cd`进入`chapter2/api-engine`文件夹，并运行以下命令：

```js
npm start 
```

我们应该会看到以下内容：

![](img/00022.jpeg)

API 引擎连接到 mLab MongoDB 实例，然后启动了一个新的 Express 服务器，最后连接到了 Mosca 经纪人，并发布了一条消息到 greet 主题。

现在，如果我们查看 Mosca 终端，我们应该会看到以下内容：

![](img/00023.jpeg)

经纪人记录了迄今为止发生的活动。一个客户端连接了用户名`API_Server_Dev`并订阅了一个名为 greet 的主题，**服务质量**（**QoS**）为`0`。

有了这个，我们的经纪人和 API 引擎之间的集成就完成了。

接下来，我们将转向 Raspberry Pi 3，并开始使用 MQTTS 客户端。

如果您对 MQTT 协议不熟悉，可以参考*MQTT Essentials: Part 1 - Introducing MQTT* ([`www.hivemq.com/blog/mqtt-essentials-part-1-introducing-mqtt`](http://www.hivemq.com/blog/mqtt-essentials-part-1-introducing-mqtt))以及后续部分。要了解更多关于 QoS 的信息，请参考*MQTT Essentials Part 6: Quality of Service 0, 1 & 2* ([`www.hivemq.com/blog/mqtt-essentials-part-6-mqtt-quality-of-service-levels`](https://www.hivemq.com/blog/mqtt-essentials-part-6-mqtt-quality-of-service-levels))。

# 树莓派软件

在这一部分，我们将构建所需的软件，使树莓派通过 Wi-Fi 路由器成为我们 Mosca 经纪人的客户端。

我们已经在数据流程图中看到了树莓派是如何站在传感器和 Mosca 经纪人之间的。现在我们将设置所需的代码和软件。

# 设置树莓派

在这一部分，我们将看看如何在树莓派上安装所需的软件。

树莓派安装了 Raspbian OS ([`www.raspberrypi.org/downloads/raspbian/`](https://www.raspberrypi.org/downloads/raspbian/))是必需的。在继续之前，Wi-Fi 应该已经设置并连接好。

如果您是第一次设置树莓派 3，请参考*在树莓派上安装 Node.js 的初学者指南* ([`thisdavej.com/beginners-guide-to-installing-node-js-on-a-raspberry-pi/`](http://thisdavej.com/beginners-guide-to-installing-node-js-on-a-raspberry-pi/))。但是，我们将涵盖 Node.js 部分，您可以在启动 Pi 并配置 Wi-Fi 之前参考。

操作系统安装完成后，启动树莓派并登录。此时，它将通过您自己的访问点连接到互联网，您应该能够正常浏览互联网。

我正在使用 VNC Viewer 从我的 Apple MacBook Pro 访问我的树莓派 3。这样，我不会总是连接到树莓派 3。

我们将从下载 Node.js 开始。打开一个新的终端并运行以下命令：

```js
$ sudo apt update
$ sudo apt full-upgrade 
```

这将升级所有需要升级的软件包。接下来，我们将安装最新版本的 Node.js。在撰写本文时，Node 7.x 是最新版本：

```js
$ curl -sL https://deb.nodesource.com/setup_7.x | sudo -E bash -
$ sudo apt install nodejs  
```

这将需要一些时间来安装，一旦安装完成，您应该能够运行以下命令来查看 Node.js 和`npm`的版本：

```js
node -v
npm -v  
```

有了这个，我们已经设置好了在 Raspberry Pi 3 上运行 MQTTS 客户端所需的软件。

# 树莓派 MQTTS 客户端

现在我们将使用 Node.js 的 MQTTS 客户端进行工作。

在树莓派 3 的桌面上，创建一个名为`pi-client`的文件夹。打开一个终端并`cd`到`pi-client`文件夹。

我们要做的第一件事是创建一个`package.json`文件。从`pi-client`文件夹内部运行以下命令：

```js
$ npm init
```

然后根据情况回答问题。完成后，我们将在 Raspberry Pi 3 上安装 MQTT.js ([`www.npmjs.com/package/mqtt`](https://www.npmjs.com/package/mqtt))。运行以下命令：

```js
$ npm install mqtt -save  
```

一旦这个安装也完成了，最终的`package.json`将与这个相同：

```js
{
    "name": "pi-client",
    "version": "0.1.0",
    "description": "",
    "main": "index.js",
    "scripts": {
        "start": "node index.js"
    },
    "keywords": ["pi", "mqtts"],
    "author": "Arvind Ravulavaru",
    "private": true,
    "license": "ISC",
    "dependencies": {
        "mqtt": "².7.1"
    }
}
```

请注意，我们已经添加了一个启动脚本来启动我们的`index.js`文件。我们将在片刻之后创建`index.js`文件。

接下来，在`pi-client`文件夹的根目录下，创建一个名为`config.js`的文件。更新`config.js`如下：

```js
module.exports = { 
    mqtt: { 
        host: '10.2.192.141', 
        clientId: 'rPI_3', 
        port: 8883 
    } 
}; 
```

请注意主机属性。这是设置为我的 MacBook 的 IP 地址，我的 MacBook 是我将运行 Mosca 经纪人 API 引擎的地方。确保它们三个（Mosca 经纪人、API 引擎和树莓派 3）都在同一个 Wi-Fi 网络上。

接下来，我们将编写所需的 MQTT 客户端代码。在`pi-client`文件夹的根目录下创建一个名为`index.js`的文件，并更新如下：

```js
var config = require('./config.js'); 
var mqtt = require('mqtt') 
var client = mqtt.connect({ 
    port: config.mqtt.port, 
    protocol: 'mqtts', 
    host: config.mqtt.host, 
    clientId: config.mqtt.clientId, 
    reconnectPeriod: 1000, 
    username: config.mqtt.clientId, 
    password: config.mqtt.clientId, 
    keepalive: 300, 
    rejectUnauthorized: false 
}); 

client.on('connect', function() { 
    client.subscribe('greet') 
    client.publish('greet', 'Hello, IoTjs!') 
}); 

client.on('message', function(topic, message) { 
    // message is Buffer 
    console.log('Topic >> ', topic); 
    console.log('Message >> ', message.toString()) 
}); 
```

这是我们在 API 引擎上编写的相同测试代码，用于测试连接。保存所有文件并转向您的 Mosca 经纪人。

# 经纪人和树莓派之间的通信

在本节中，我们将通过 MQTTS 在经纪人和树莓派之间进行通信。

转到`broker`文件夹并运行以下命令：

```js
mosca -c index.js -v | pino  
```

接下来，转到树莓派，`cd`进入`pi-client`文件夹，并运行以下命令：

```js
$ npm start  
```

我们应该在树莓派上看到以下消息：

>![](img/00024.jpeg)

当我们查看 Mosca 的控制台时，我们应该看到以下内容：

![](img/00025.jpeg)

这结束了我们在树莓派 3 和 Mosca 经纪人之间的连接测试。

# 故障排除

如果您无法看到以前的消息，请检查以下内容：

+   检查树莓派和运行经纪人的机器是否在同一个 Wi-Fi 网络上

+   检查运行经纪人的机器的 IP 地址

# 树莓派、经纪人和 API 引擎之间的通信

现在我们将集成树莓派、经纪人和 API 引擎，并将数据从树莓派传递到 API 引擎。

我们将实现这一点的方式是创建一个名为`api-engine`的主题和另一个名为`rpi`的主题。

要将数据从树莓派发送到 API 引擎，我们将使用`api-engine`主题，当我们需要将数据从 API 引擎发送到树莓派时，我们将使用`rpi`主题。

在这个例子中，我们将获取树莓派的 MAC 地址并将其发送到 API 引擎。API 引擎将通过将相同的 MAC 地址发送回树莓派来确认相同。API 引擎和树莓派之间的通信将在前面提到的两个主题上进行。

因此，首先，我们将按照以下方式更新`api-engine/server/mqtt/index.js`：

```js
var mqtt = require('mqtt'); 
var config = require('../config/environment'); 

var client = mqtt.connect({ 
    port: config.mqtt.port, 
    protocol: 'mqtts', 
    host: config.mqtt.host, 
    clientId: config.mqtt.clientId, 
    reconnectPeriod: 1000, 
    username: config.mqtt.clientId, 
    password: config.mqtt.clientId, 
    keepalive: 300, 
    rejectUnauthorized: false 
}); 

client.on('connect', function() { 
    client.subscribe('api-engine'); 
}); 

client.on('message', function(topic, message) { 
    // message is Buffer 
    // console.log('Topic >> ', topic); 
    // console.log('Message >> ', message.toString()); 
    if (topic === 'api-engine') { 
        var macAddress = message.toString(); 
        console.log('Mac Address >> ', macAddress); 
        client.publish('rpi', 'Got Mac Address: ' + macAddress); 
    } else { 
        console.log('Unknown topic', topic); 
    } 
}); 
```

在这里，一旦建立了 MQTT 连接，我们就会订阅`api-engine`主题。当我们从`api-engine`主题接收到任何数据时，我们将把相同的数据发送回`rpi`主题。

在`broker`文件夹中运行以下命令：

```js
mosca -c index.js -v | pino  
```

接下来，在`api-engine`文件夹中运行以下命令：

```js
npm start  
```

接下来，回到树莓派。我们将安装`getmac`模块（[`www.npmjs.com/package/getmac`](https://www.npmjs.com/package/getmac)），这将帮助我们获取设备的 MAC 地址。

在`pi-client`文件夹中运行以下命令：

```js
$ npm install getmac --save  
```

完成后，更新`/home/pi/Desktop/pi-client/index.js`如下：

```js
var config = require('./config.js'); 
var mqtt = require('mqtt'); 
var GetMac = require('getmac'); 

var client = mqtt.connect({ 
    port: config.mqtt.port, 
    protocol: 'mqtts', 
    host: config.mqtt.host, 
    clientId: config.mqtt.clientId, 
    reconnectPeriod: 1000, 
    username: config.mqtt.clientId, 
    password: config.mqtt.clientId, 
    keepalive: 300, 
    rejectUnauthorized: false 
}); 

client.on('connect', function() { 
    client.subscribe('rpi'); 
    GetMac.getMac(function(err, macAddress) { 
        if (err) throw err; 
        client.publish('api-engine', macAddress); 
    }); 
}); 

client.on('message', function(topic, message) { 
    // message is Buffer 
    // console.log('Topic >> ', topic); 
    // console.log('Message >> ', message.toString()); 
    if (topic === 'rpi') { 
        console.log('API Engine Response >> ', message.toString()); 
    } else { 
        console.log('Unknown topic', topic); 
    } 
}); 
```

在以前的代码中，我们等待树莓派和经纪人之间的连接建立。一旦完成，我们就订阅了`rpi`主题。接下来，我们使用`GetMac.getMac()`获取了树莓派的 MAC 地址，并将其发布到`api-engine`主题。

在`message`事件回调中，我们正在监听`rpi`主题。如果我们从服务器收到任何数据，它将在这里打印出来。

保存文件，并在`pi-client`文件夹中运行以下命令：

```js
$ npm start  
```

现在，如果我们查看经纪人终端/提示，我们应该看到以下内容：

![](img/00026.jpeg)

这两个设备都连接并订阅了感兴趣的主题。

接下来，如果我们查看`api-engine`终端/提示，我们应该看到以下内容：

![](img/00027.jpeg)

最后，树莓派终端应该看起来与这个一样：

![](img/00028.jpeg)

有了这个，我们完成了树莓派与经纪人和 API 引擎的集成。

在下一节中，我们将实现一个 Web 应用程序，该应用程序可以通过经纪人和 API 引擎与树莓派发送和接收数据。

# Web 应用程序

在本节中，我们将构建一个与我们的 API 引擎进行交互的 Web 应用程序。Web 应用程序是我们将与智能设备进行交互的主要界面。

我们将使用 Angular (4)和 Twitter Bootstrap (3)构建 Web 应用程序。界面不一定要使用 Angular 和 Bootstrap 构建；也可以使用 jQuery 或 React.js。我们将做的只是使用浏览器中的 JavaScript 与 API 引擎的 API 进行接口。我们之所以使用 Angular，只是为了保持所有应用程序的框架一致。由于我们将使用 Ionic 框架，它也遵循 Angular 的方法，因此对我们来说管理和重用都会很容易。

要开始使用 Web 应用程序，我们将安装 Angular CLI ([`github.com/angular/angular-cli`](https://github.com/angular/angular-cli))。

在运行我们的代理和 API 引擎的机器上，我们也将设置 Web 应用程序。

# 设置应用程序

从`chapter2`文件夹内，打开一个新的命令提示符/终端，并运行以下命令：

```js
npm install -g @angular/cli  
```

这将安装 Angular CLI 生成器。安装完成后运行`ng -v`，您应该看到一个大于或等于 1.0.2 的版本号。

如果在设置和运行 IoTFW.js 时遇到任何问题，请随时在此处留下您的评论：[`github.com/PacktPublishing/Practical-Internet-of-Things-with-JavaScript/issues/1`](https://github.com/PacktPublishing/Practical-Internet-of-Things-with-JavaScript/issues/1)

对于 Web 应用程序，我们已经使用 Angular CLI 创建了一个基本项目，并添加了必要的部分来与 API 引擎集成。我们将克隆项目并在此基础上开始工作。

要开始，我们需要 Web 应用程序的基础。如果您还没有克隆该书的代码存储库，可以在您的任何位置使用以下命令行进行克隆：

```js
git clone git@github.com:PacktPublishing/Practical-Internet-of-Things-with-JavaScript.git
```

或者您也可以从[`github.com/PacktPublishing/Practical-Internet-of-Things-with-JavaScript`](https://github.com/PacktPublishing/Practical-Internet-of-Things-with-JavaScript)下载 zip 文件。

一旦存储库被下载，`cd`进入`base`文件夹，并将`web-app-base`文件夹复制到`chapter2`文件夹中。

基础已经复制完成后，`cd`进入`web-app`文件夹，并运行以下命令：

```js
npm install  
```

这将安装所需的依赖项。

# 项目结构

如果打开`cloned`文件夹，我们应该看到以下内容：

```js
.
├── README.md
├── e2e
│ ├── app.e2e-spec.ts
│ ├── app.po.ts
│ └── tsconfig.e2e.json
├── karma.conf.js
├── package.json
├── protractor.conf.js
├── src
│ ├── app
│ │ ├── add-device
│ │ │ ├── add-device.component.css
│ │ │ ├── add-device.component.html
│ │ │ ├── add-device.component.spec.ts
│ │ │ └── add-device.component.ts
│ │ ├── app.component.css
│ │ ├── app.component.html
│ │ ├── app.component.spec.ts
│ │ ├── app.component.ts
│ │ ├── app.global.ts
│ │ ├── app.module.ts
│ │ ├── device
│ │ │ ├── device.component.css
│ │ │ ├── device.component.html
│ │ │ ├── device.component.spec.ts
│ │ │ └── device.component.ts
│ │ ├── device-template
│ │ │ ├── device-template.component.css
│ │ │ ├── device-template.component.html
│ │ │ ├── device-template.component.spec.ts
│ │ │ └── device-template.component.ts
│ │ ├── guard
│ │ │ ├── auth.guard.spec.ts
│ │ │ └── auth.guard.ts
│ │ ├── home
│ │ │ ├── home.component.css
│ │ │ ├── home.component.html
│ │ │ ├── home.component.spec.ts
│ │ │ └── home.component.ts
│ │ ├── login
│ │ │ ├── login.component.css
│ │ │ ├── login.component.html
│ │ │ ├── login.component.spec.ts
│ │ │ └── login.component.ts
│ │ ├── nav-bar
│ │ │ ├── nav-bar.component.css
│ │ │ ├── nav-bar.component.html
│ │ │ ├── nav-bar.component.spec.ts
│ │ │ └── nav-bar.component.ts
│ │ ├── register
│ │ │ ├── register.component.css
│ │ │ ├── register.component.html
│ │ │ ├── register.component.spec.ts
│ │ │ └── register.component.ts
│ │ └── services
│ │ ├── auth.service.spec.ts
│ │ ├── auth.service.ts
│ │ ├── data.service.spec.ts
│ │ ├── data.service.ts
│ │ ├── devices.service.spec.ts
│ │ ├── devices.service.ts
│ │ ├── http-interceptor.service.spec.ts
│ │ ├── http-interceptor.service.ts
│ │ ├── loader.service.spec.ts
│ │ ├── loader.service.ts
│ │ ├── socket.service.spec.ts
│ │ └── socket.service.ts
│ ├── assets
│ ├── environments
│ │ ├── environment.prod.ts
│ │ └── environment.ts
│ ├── favicon.ico
│ ├── index.html
│ ├── main.ts
│ ├── polyfills.ts
│ ├── styles.css
│ ├── test.ts
│ ├── tsconfig.app.json
│ ├── tsconfig.spec.json
│ └── typings.d.ts
├── tsconfig.json
└── tslint.json
```

现在，让我们来看一下项目结构和代码设置的步骤。

在高层次上，我们有一个`src`文件夹，其中包含所有的源代码和单元测试代码，还有一个`e2e`文件夹，其中包含端到端测试。

我们将大部分时间花在`src/app`文件夹内。在进入这个文件夹之前，打开`web-app/src/main.ts`，这是一切的开始。接下来，我们在这里添加了 Twitter Bootstrap Cosmos 主题([`bootswatch.com/cosmo/`](https://bootswatch.com/cosmo/))，并定义了一些布局样式。

现在，`app/src`文件夹：在这里，我们定义了根组件、根模块和所需的组件和服务。

# 应用模块

打开`web-app/src/app/app.module.ts`。这个文件包括`@NgModule`声明，定义了我们将要使用的所有组件和服务。

我们已经创建了以下组件：

+   `AppComponent`：应用程序根组件，包含路由出口

+   `NavBarComponent`：这是出现在所有页面上的导航栏组件。该组件会自动检测认证状态，并相应地显示菜单栏

+   `LoginComponent`：处理登录功能

+   `RegisterComponent`：用于与 API 引擎进行注册

+   `HomeComponent`：这个组件显示当前登录用户附加的所有设备

+   `DeviceComponent`：这个组件显示有关一个设备的信息

+   `AddDeviceComponent`：这个组件让我们向设备列表中添加新的组件

+   `DeviceTemplateComponent`：用于表示应用程序中设备的通用模板

除了上述内容，我们还添加了所需的模块到导入中：

+   路由模块：用于管理路由

+   `LocalStorageModule`：为了在浏览器中管理用户数据，我们将使用`LocalStorgae`

+   `SimpleNotificationsModule`：使用 Angular 2 通知显示通知（[`github.com/flauc/angular2-notifications`](https://github.com/flauc/angular2-notifications)）

对于服务，我们有以下内容：

+   `AuthService`：管理 API 引擎提供的身份验证 API

+   `DevicesService`：管理 API 引擎提供的设备 API

+   `DataService`：管理 API 引擎提供的数据 API

+   `SocketService`：管理从 API 引擎实时发送数据的 Web 套接字

+   `AuthGuard`：一个 Angular 守卫，用于保护需要身份验证的路由。阅读*使用 Angular 中的守卫保护路由*（[`blog.thoughtram.io/angular/2016/07/18/guards-in-angular-2.html`](https://blog.thoughtram.io/angular/2016/07/18/guards-in-angular-2.html)）获取有关守卫的更多信息

+   `LoaderService`：在进行活动时显示和隐藏加载器栏

+   `Http`：我们用来发出 HTTP 请求的 HTTP 服务。在这里，我们没有直接使用 HTTP 服务，而是扩展了该类，并在其中添加了我们的逻辑，以更好地使用加载器服务来管理 HTTP 请求体验

请注意，此时 API 引擎没有设备和数据的 API，并且数据的套接字未设置。一旦我们完成 Web 应用程序，我们将在 API 引擎中实现它。

在这个 Web 应用程序中，我们将有以下路由：

+   `login`：让用户登录应用程序

+   `register`：注册我们的应用程序

+   `home`：显示用户帐户中所有设备的页面

+   `add-device`：向用户的设备列表添加新设备的页面

+   `view-device/:id`：查看由 URL 中的 id 参数标识的一个设备的页面

+   `**`：默认路由设置为登录

+   `''`：如果没有匹配的路由，我们将用户重定向到登录页面

# Web 应用程序服务

现在我们在高层次上了解了这个 Web 应用程序中的所有内容，我们将逐步介绍服务和组件。

打开`web-app/src/app/services/http-interceptor.service.ts`；在这个类中，我们扩展了`Http`类并实现了类方法。我们添加了两个自己的方法，名为`requestInterceptor()`和`responseInterceptor()`，分别拦截请求和响应。

当请求即将发送时，我们调用`requestInterceptor()`来显示加载器，指示 HTTP 活动，我们使用`responseInterceptor()`一旦响应到达就隐藏加载器。这样，用户清楚地知道是否有任何后台活动正在进行。

接下来是`LoaderService`类；打开`web-app/src/app/services/loader.service.ts`，从这里我们可以看到，我们添加了一个名为`status`的类属性，类型为`BehaviorSubject<boolean>`（要了解更多关于`Behavior`主题的信息，请参阅[`github.com/Reactive-Extensions/RxJS/blob/master/doc/api/subjects/behaviorsubject.md`](https://github.com/Reactive-Extensions/RxJS/blob/master/doc/api/subjects/behaviorsubject.md)）。我们还有一个方法，如果 HTTP 服务或任何其他组件希望显示或隐藏加载器栏，它们将调用该方法，然后将值设置为 true 或 false。

加载器服务所需的 HTML 位于`web-app/src/app/app.component.html`，所需的样式位于`web-app/src/app/app.component.css`。

我们将使用 Web 套接字在 Web 应用程序和 API 引擎之间实时流式传输数据。打开`web-app/src/app/services/socket.service.ts`，我们应该看到构造函数和`getData()`方法。我们在我们的 Web 应用程序中使用`socket.io-client`（[`github.com/socketio/socket.io-client`](https://github.com/socketio/socket.io-client)）来管理 Web 套接字。

在构造函数中，我们已经创建了一个新的套接字连接到我们的 API 引擎，并将身份验证令牌作为查询参数传递。我们也将通过 Web 套接字验证传入的连接。只有在令牌有效的情况下，我们才允许连接，否则我们关闭 Web 套接字。

在`getData()`内，我们订阅了设备的`data:save`主题。这是我们从 API 引擎得到通知的方式，当设备有新数据可用时。

现在我们将查看三个 API 服务，用于验证用户，获取用户设备和获取设备数据：

+   `AuthService`：打开`web-app/src/app/services/auth.service.ts`。在这里，我们已经定义了`register()`，`login()`和`logout()`，它们负责管理身份验证状态，我们还有`isAuthenticated()`，它返回当前的身份验证状态，即用户是已登录还是已注销。

+   `DevicesService`：打开`web-app/src/app/services/devices.service.ts`。在这里，我们实现了三种方法：创建一个，读取一个，删除一个。通过这样，我们为用户管理我们的设备。

+   `DataService`：打开`web-app/src/app/services/data.service.ts`，它管理设备的数据。我们这里只有两种方法：创建一个新的数据记录和获取设备的最后 30 条记录。

请注意，我们正在使用`web-app/src/app/app.global.ts`来保存所有我们的常量全局变量。

现在我们已经完成了所需的服务，我们将浏览组件。

# Web 应用程序组件

我们将从应用程序组件开始。应用程序组件是根组件，它包含路由器出口，加载器服务 HTML 和通知服务 HTML。您可以在这里找到相同的内容：`web-app/src/app/app.component.html`。在`web-app/src/app/app.component.ts`中，我们已经定义了`showLoader`，它决定是否应该显示加载器。我们还定义了通知选项，它存储通知服务的配置。

在构造函数内，我们正在监听路由器上的路由更改事件，以便在页面更改时显示加载栏。我们还在监听加载器服务状态变量。如果这个变化，我们就会显示或隐藏加载器。

用户登陆的第一个页面是登录页面。登录页面/组件`web-app/src/app/login/login.component.ts`只有一个方法，从`web-app/src/app/login/login.component.html`获取用户的电子邮件和密码，并对用户进行身份验证。

使用主页上的注册按钮，用户注册自己。在`RegisterComponent`类内，`web-app/src/app/register/register.component.ts`，我们已经定义了`register()`，它获取用户的信息，并使用`AuthService`注册用户。

一旦用户成功验证，我们将用户重定向到`LoginComponent`。在`HomeComponent`，`web-app/src/app/home/home.component.ts`中，我们获取与用户关联的所有设备并在加载时显示它们。此页面还有一个按钮，用于使用`AddDeviceComponent`添加新设备。

要查看一个设备，我们使用`DeviceComponent`来查看一个设备。

目前，我们还没有任何可用于处理设备和数据的 API。在下一节中完成 API 引擎更新后，我们将重新访问此页面。

# 启动应用程序

要运行应用程序，请在`web-app`文件夹内打开终端/提示符，并运行以下命令：

```js
ng serve
```

在运行上一个命令之前，请确保 API 引擎和 Mosca 正在运行。

一旦 webpack 编译成功，导航到`http://localhost:4200/login`，我们应该看到登录页面，这是第一个页面。

![](img/00029.jpeg)

我们可以使用在测试 API 引擎时创建的帐户，使用 Postman，或者我们可以通过点击“使用 Web 应用程序注册”来创建一个新帐户，如下所示：

![](img/00030.jpeg)

如果注册成功，我们应该被重定向到主页，如下所示：

![](img/00031.jpeg)

如果我们打开开发者工具，应该会看到先前的消息。API 引擎没有实现设备的 API，因此出现了先前的“404”。我们将在第三章中修复这个问题，*IoTFW.js - II*。

我们还将在第三章中逐步完成 Web 应用程序的剩余部分，一旦 API 引擎更新完成。

# 总结

在本章中，我们已经了解了建立物联网解决方案的过程。我们使用 JavaScript 作为编程语言构建了大部分框架。

我们首先了解了从树莓派到 Web 应用程序、桌面应用程序或移动应用程序等最终用户设备的架构和数据流。然后我们开始使用 Mosca 工作代理，设置了 MongoDB。接下来，我们设计并开发了 API 引擎，并完成了基本的树莓派设置。

我们已经在 Web 应用程序上工作，并设置了必要的模板，以便与应用程序的剩余部分配合使用。在第三章中，我们将完成整个框架，并集成 DHT11（温湿度）传感器和 LED，以验证端到端的双向数据流。
