# 第五章：*第四章*

# 使用 Node.js 创建 RESTful API

## 学习目标

在本章结束时，您将能够：

+   为 Express.js API 设置项目结构

+   使用不同的 HTTP 方法设计具有端点的 API

+   在本地主机上运行 API，并通过 cURL 或基于 GUI 的工具与其交互

+   解析端点的用户输入，并考虑处理错误的不同方式

+   设置需要用户身份验证的端点

在本章中，我们将使用 Express.js 和 Node.js 来设置一个可以供前端应用程序使用的 API。

## 介绍

**应用程序编程接口**（**API**）变得比以往任何时候都更加重要。使用 API 可以使单个服务器端程序被多个脚本和应用程序使用。由于其有用性，使用 Node.js 的后端开发人员的 API 管理已成为最常见的任务之一。

让我们以一个既有网站又有移动应用程序的公司为例。这两个前端界面都需要服务器端的基本相同功能。通过将这些功能封装在 API 中，我们可以实现服务器端代码的清晰分离和重用。过去那些将后端功能直接嵌入网站界面代码的笨拙 PHP 应用程序的时代已经一去不复返。

我们将使用 Node.js 来设置一个**表述状态转移**（**REST**）API。我们的 API 将在 Express.js 上运行，这是一个具有路由功能的流行 Web 应用程序框架。借助这些工具，我们可以快速在本地主机上运行一个端点。我们将研究设置 API 的最佳实践，以及 Express.js 库中使用的特定语法。除此之外，我们还将考虑 API 设计的基础知识，简化开发人员和使用它的服务的使用。

## 什么是 API？

API 是与软件应用程序进行交互的标准化方式。API 允许不同的软件应用程序相互交互，而无需了解底层功能的内部工作原理。

API 在现代软件工程中变得流行，因为它们允许组织通过重用代码更加有效。以地图的使用为例：在 API 普及之前，需要地图功能的组织必须在内部维护地图小部件。通常，这些地图小部件的性能会很差，因为它们只是业务和工程团队的次要关注点。

现在，使用地图的网站或应用程序很少在内部维护地图。许多网络和手机应用程序都使用来自 Google 或 OpenStreetMap 等替代方案的地图 API。这使得每家公司都可以专注于其核心竞争力，而不必创建和维护自己的地图小部件。

有几家成功的初创公司的业务模式围绕着通过 API 提供服务。一些例子包括著名公司如 Twilio、Mailgun 和 Sentry。除此之外，还有一些较小的公司通过 API 提供独特的服务，比如 Lob，它可以通过其 API 根据请求发送实体信件和明信片。在这里，开发人员只需将信件内容和目的地地址发送到 Lob 的 API，它就会自动打印并代表开发人员寄出。以下是一些知名公司提供的 API 服务的示例。

![](img/C14587_04_01.jpg)

###### 图 4.1：基于 API 的公司示例

这些公司通过提供可用于提供特定服务的构建块，使开发人员能够更好地、更快地开发应用程序。其有效性的证明可以从这些服务的广泛采用中看出。使用 Twilio 提供文本或电话集成的公司包括可口可乐、Airbnb、优步、Twitch 等许多其他公司。这些公司中的许多公司又为其他公司和开发人员提供自己的 API 来构建。这种趋势被称为 API 经济。

这些服务的另一个共同点是它们都通过 HTTP 使用 REST。新开发人员经常认为所有 API 都是通过 HTTP 使用的；然而，当我们谈论 API 时，对使用的协议或介质没有限制。API 的接口理论上可以是任何东西，从按钮到无线电波。虽然有许多接口选项可供选择，但 HTTP 仍然是最广泛使用的介质。在下一节中，我们将更详细地讨论 REST。

## REST 是什么？

REST 是一种用于创建基于 web 的服务的软件架构模式。这意味着资源由特定的 URL 端点表示，例如`website.com/post/12459`，可以使用其特定 ID 访问网站的帖子。REST 是将资源映射到 URL 端点的方法。

在数据库管理领域的一个相关概念是**CRUD**（**创建、读取、更新和删除**）。这是你可以与数据库资源交互的四种方式。同样，我们通常与由我们的 API 端点定义的资源对象交互的方式也有四种。HTTP 协议具有内置方法，可以简化诸如`POST`、`GET`、`PUT`和`DELETE`等任务。

先前提到的任务的功能如下：

+   `POST`：创建对象资源

+   `GET`：检索有关对象资源的信息

+   `PUT`：更新特定对象的信息

+   `DELETE`：删除特定对象

其他方法：除了四种主要方法外，还有一些其他不太常用的方法。我们不会在这里使用它们，你也不必担心它们，因为客户端和服务器很少使用它们：

+   `HEAD`：与`GET`相同，但只检索标头而不是主体。

+   `OPTIONS`：返回服务器或 API 的允许选项列表。

+   `CONNECT`：用于创建 HTTP 隧道。

+   `TRACE`：这是用于调试的消息回路。

+   `PATCH`：这类似于`PUT`，但用于更新单个值。请注意，`PUT`可以代替`PATCH`使用。

### Express.js 用于 Node.js 上的 RESTful API

好消息是，如果你了解基本的 JavaScript，你已经完成了创建你的第一个 API 的一半。使用 Express.js，我们可以轻松构建 HTTP 端点。Express 是一个流行的、最小的 web 框架，用于在节点上创建和托管 web 应用程序。它包括几种内置的路由方法，允许我们映射传入的请求。有许多中间件包可以使常见任务更容易。在本章后面，我们将使用一个验证包。

在本章中，我们将创建一个假设的智能房屋 API 的各个方面。这将需要为具有改变设备状态逻辑的各种设备添加端点。一些端点将对网络中的任何人开放，例如智能灯，而其他一些，如加热器，将需要身份验证。

#### 注意

什么是智能房屋？智能房屋是一个包含互联网连接设备的房屋，您可以通过基于云的控制系统与之交互。与用户和其他设备通信的互联网连接设备的趋势通常被称为**物联网**（**IoT**）。

在本章中，我们将为一个包含智能设备的房屋编写 API，包括智能灯泡和加热器。此练习的代码文件可在[`github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson04/Exercise16`](https://github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson04/Exercise16)上找到。

### 练习 16：创建一个带有索引路由的 Express 项目

在这个练习中，我们的目标是创建一个新的节点项目，安装 Express，然后创建一个返回带有消息单个属性的 JSON 对象的索引路由。一旦它运行起来，我们可以通过在本地主机上进行 cURL 请求来测试它。要做到这一点，执行以下步骤：

1.  创建一个名为`smartHouse`的文件夹并初始化一个`npm`项目：

```js
mkdir smartHouse
cd smartHouse
npm init
```

1.  安装`express`库，使用`-s`标志将其保存到我们的`package.json`文件中：

```js
npm install -s express
```

1.  创建一个名为`server.js`的文件，导入`express`并创建一个`app`对象：

```js
const express = require('express');
const app = express();
```

1.  在`server.js`中添加一个指定'/'的`app.get`方法，用于我们的索引路由：

```js
app.get('/', (req, res) => {
  let info = {};
  info.message = "Welcome home! Our first endpoint.";
  res.json(info);
});
```

前面的代码创建了一个`HTTP GET`函数，返回一个名为`info`的对象，其中包含一个名为`message`的属性。

1.  添加一个`app.listen`函数，告诉我们的应用程序监听`端口 3000`：

```js
// Start our application on port 3000
app.listen(3000, () => console.log('API running on port 3000'));
```

前面的步骤就是一个简单的 Node.js Express API 示例所需的全部内容。通过运行前面的代码，我们将在本地主机上创建一个应用程序，返回一个简单的 JSON 对象。

1.  在另一个终端窗口中，返回到您的`smartHouse`文件夹的根目录并运行以下命令：

```js
npm start
```

1.  通过在 Web 浏览器中转到`localhost:3000`，确认应用程序是否正确运行：

![图 4.2：在 Web 浏览器中显示 localhost:3000](img/C14587_04_02.jpg)

###### 图 4.2：在 Web 浏览器中显示 localhost:3000

如果您已正确复制了代码，您应该在**localhost:3000**看到一个 JSON 对象被提供，就像在前面的屏幕截图中显示的那样。

#### 注意

如果在任何步骤中遇到问题或不确定项目文件应该是什么样子，您可以使用项目文件夹将代码恢复到与项目一致的状态。文件夹将根据它们关联的步骤命名，例如`Exercise01，Exercise02`等。当您第一次进入文件夹时，请确保运行`npm install`来安装项目使用的任何模块。

### 通过 HTTP 与您的 API 进行交互

在这一部分，我们将与*练习 16*中创建的服务器进行交互，*创建一个带有索引路由的 Express 项目*。因此，请确保您保持一个终端窗口打开并运行服务器。如果您已经关闭了该窗口或关闭了它，只需返回到`smartHouse`文件夹并运行`npm start`。

我们通过使用 Web 浏览器验证了我们的 API 正在运行。Web 浏览器是查看路由的最简单方式，但它有限，只适用于`GET`请求。在本节中，我们将介绍另外两种与 API 进行更高级交互的方法，这两种方法都允许进行更高级的请求，包括以下内容：

+   超出`GET`的请求，包括`PUT`、`POST`和`DELETE`

+   向您的请求添加标头信息

+   为受保护的端点包括授权信息

我首选的方法是使用命令行工具 cURL。cURL 代表 URL 的客户端。它已安装在大多数版本的 macOS、Linux 和 Windows 10 上(2018 年及以后的版本)。它是一个用于进行 HTTP 请求的命令行工具。对于一个非常简单的命令，运行以下命令：

```js
curl localhost:3000
```

以下是前面代码的输出：

![](img/C14587_04_03.jpg)

###### 图 4.3：显示 cURL localhost:3000

#### 注意

命令行程序`jq`将在本章中用于格式化 cURL 请求。`jq`是一个轻量级和灵活的命令行 JSON 处理器。该程序适用于 macOS、Linux 和 Windows。如果您无法在系统上安装它，仍然可以使用不带`jq`的`curl`。要这样做，只需从本章中任何 curl 命令的末尾删除`| jq`命令。

安装`jq`的说明可以在[`github.com/stedolan/jq`](https://github.com/stedolan/jq)找到。

通过使用带有`jq`的`curl`，我们可以使阅读输出变得更容易，这将在我们的 JSON 变得更复杂时特别有用。在下面的示例中，我们将重复与前面示例中相同的 curl 命令，但这次使用 Unix 管道(`|`)将输出传送到`jq`：

```js
curl -s localhost:3000 | jq
```

当像前面的命令一样将`curl`传送到`jq`时，我们将使用`-s`标志，该标志代表“静默”。如果`curl`在没有此标志的情况下进行传送，您还将看到关于请求速度的不需要的信息。

假设你已经做了一切正确的事情，你应该观察到一些干净的 JSON 作为输出显示：

![图 4.4：cURL 管道传输到 jq](img/C14587_04_04.jpg)

###### 图 4.4：cURL 管道传输到 jq

如果你喜欢使用基于 GUI 的应用程序，你可以使用 Postman，它是一个 Chrome 扩展程序，可以以直接的方式轻松发送 HTTP 请求。一般来说，我更喜欢在命令行上快速使用 cURL 和 jq。然而，对于更复杂的用例，我可能会打开 Postman，因为 GUI 使得处理头部和授权变得更容易一些。有关安装 Postman 的说明，请访问网站[`www.getpostman.com`](https://www.getpostman.com)：

![图 4.5：Postman 中 cURL 请求的屏幕截图](img/C14587_04_05.jpg)

###### 图 4.5：Postman 中 cURL 请求的屏幕截图

### 练习 17：创建和导入路由文件

目前，我们的应用程序在根 URL 上运行一个端点。通常，一个 API 会有许多路由，将它们全部放在主`server.js`文件中会很快导致项目变得杂乱。为了防止这种情况发生，我们将把每个路由分离成模块，并将每个模块导入到我们的`server.js`文件中。

#### 注意

此示例的完整代码可以在[`github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson04/Exercise17`](https://github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson04/Exercise17)找到。

执行以下步骤完成练习：

1.  要开始，创建`smartHouse`文件夹中的一个新文件夹：

```js
mkdir routes
```

1.  创建`routes/index.js`文件，并将`server.js`中的`import`语句和`main`函数移动到该文件中。然后，在下面，我们将添加一行将`router`对象导出为一个模块：

```js
const express = require('express');
const router = express.Router();
router.get('/', function(req, res, next) {
  let info = {};
  info.message = "Welcome home! Our first endpoint.";
  res.json(info);
});
// Export route so it is available to import
module.exports = router;
```

上述代码本质上是我们在第一个练习中编写的代码移动到不同的文件中。关键的区别在于底部的一行，那里写着 `module.exports = router;`。这一行将我们创建的 `router` 对象导出，并使其可以被导入到另一个文件中。每当我们创建一个新的路由文件时，它都会包含相同的底部导出行。

1.  打开`server.js`并删除第 3 到第 8 行，因为`app.get`方法已经移动到`/routes/index.js`文件中。然后，我们将导入`path`和`fs`（文件系统）库。我们还将导入一个名为`http-errors`的库，稍后将用于管理 HTTP 错误。`server.js`的前九行将如下所示：

```js
const express = require('express');
const app = express();
// Import path and file system libraries for importing our route files
const path = require('path');
const fs = require('fs');
// Import library for handling HTTP errors
const createError = require('http-errors');
```

1.  此外，在`server.js`中，我们将打开 URL 编码，并告诉`express`使用 JSON：

```js
// Tell express to enable url encoding
app.use(express.urlencoded({extended: true}));
app.use(express.json());
```

1.  接下来，我们将导入我们的索引路由并将其与一个路径关联起来。在我们完成了这些步骤之后，`server.js`应该包含以下内容：

```js
// Import our index route
let index = require('./routes/index');
// Tell Express to use our index module for root URL
app.use('/', index);
```

1.  我们可以为任何访问的 URL 创建一个捕获所有的`404`错误，这些 URL 没有对应的函数。在`app.use`方法内部，我们将 HTTP 状态码设置为`404`，然后使用我们在*步骤 2*中导入的`http-errors`库创建一个捕获所有的`404`错误（重要的是以下代码位于所有其他路由声明的下方）：

```js
// catch 404 and forward to error handler
app.use(function(req, res, next) {
  res.status(404);
  res.json(createError(404));
});
```

1.  文件的最后一行应该存在于我们之前的练习中：

```js
// Start our application on port 3000
app.listen(3000, () => console.log('API running on port 3000'));
```

完成这些步骤后，运行我们的代码应该产生以下输出，与*练习 16，创建带有索引路由的 Express 项目*中的结果相同：

![图 4.6：输出消息](img/C14587_04_06.jpg)

###### 图 4.6：输出消息

`routes`文件夹的优势在于，随着 API 的增长，它使得组织我们的 API 变得更容易。每当我们想要创建一个新的路由时，我们只需要在`routes`文件夹中创建一个新文件，使用`require`在`server.js`中导入它，然后使用 Express 的`app.use`函数将文件与一个端点关联起来。

**模板引擎**：在前两行中我们使用`app.use`时，我们修改了`express`的设置以使用扩展的 URL 编码和 JSON。它也可以用于设置模板引擎；例如，**嵌入式 JavaScript**（**EJS**）模板引擎：

```js
app.set('view engine', 'ejs');
```

模板引擎允许 Express 为网站生成和提供动态 HTML 代码。流行的模板引擎包括 EJS、Pug（Jade）和 Handlebars。例如，通过使用 EJS，我们可以使用从路由传递到视图的用户对象动态生成 HTML：

```js
<p><%= user.name %></p>
```

在我们的情况下，我们不需要使用`view`或模板引擎。我们的 API 将专门返回和接受标准的 JSON。如果您有兴趣在 HTML 网站中使用 Express，我们鼓励您研究与 Express 兼容的模板引擎。

### HTTP 状态代码

在*练习 17*的*步骤 6*中，*创建和导入路由文件*，我们将响应的 HTTP 状态代码设置为`404`。大多数人都听说过 404 错误，因为在网站上找不到页面时通常会看到它。然而，大多数人不知道状态代码是什么，也不知道除了`404`之外还有哪些代码。因此，我们将从解释状态代码的概念开始，并介绍一些最常用的代码。

状态代码是服务器在 HTTP 响应中返回给客户端请求的三位数字。每个三位代码对应于一个标准化的状态，例如`未找到`、`成功`和`服务器错误`。这些标准化代码使处理服务器变得更加容易和标准化。通常，状态代码将附带一些额外的消息文本。这些消息对人类很有用，但在编写处理 HTTP 响应的脚本时，仅仅考虑状态代码会更容易。例如，基于返回的状态代码创建一个 case 语句。

响应代码分为由三位数字中的第一位数字确定的类别：

![图 4.7：HTTP 响应代码类别表](img/C14587_04_07.jpg)

###### 图 4.7：HTTP 响应代码类别表

HTTP 代码的每个类别都包含可在特定情况下使用的几个具体代码。这些标准化的代码将帮助客户端处理响应，即使涉及不熟悉的 API。例如，任何 400 系列的客户端错误代码都表示问题出在请求上，而 500 系列的错误代码表示问题可能出在服务器本身。

让我们来看看以下图中每个类别中存在的一些具体 HTTP 状态代码：

![图 4.8：HTTP 响应代码表](img/C14587_04_08.jpg)

###### 图 4.8：HTTP 响应代码表

在下图中，我们可以看到一些更具体的 HTTP 状态代码：

![图 4.9：HTTP 响应代码继续表](img/C14587_04_09.jpg)

###### 图 4.9：HTTP 响应代码继续表

这里列出的代码只是可用的数十种 HTTP 状态代码中的一小部分。在编写 API 时，使用适当的状态代码是有用的。状态代码使响应对用户和机器更容易理解。在测试我们的应用程序时，我们可能希望编写一个脚本，将一系列请求与预期的响应状态代码进行匹配。

在使用 Express 时，默认状态代码始终为`200`，因此如果您在结果中未指定代码，它将为`200`，表示成功的响应。完整的 HTTP 状态代码列表可以在[`developer.mozilla.org/en-US/docs/Web/HTTP/Status`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)找到。

要设置状态代码错误，请使用上面的代码部分，并将`404`替换为`http-errors`库支持的任何错误代码，该库是 Express 的子依赖项。您可以在项目的 GitHub 上找到所有支持的错误代码列表[`github.com/jshttp/http-errors`](https://github.com/jshttp/http-errors)。

您还可以向`createError()`传递一个额外的字符串来设置自定义消息：

```js
res.status(404);
res.json(createError(401, 'Please login to view this page.'));
```

如果您使用成功代码，只需使用`res.status`并像使用默认的`200`状态一样返回您的 JSON 对象：

```js
res.status(201); // Set 201 instead of 200 to indicate resource created
res.json(messageObject); // An object containing your response
```

#### 注意

有许多很少使用的状态代码；其中包括一些在互联网历史上创建的笑话代码：

418-我是一个茶壶：1998 年愚人节的一个笑话。它表示服务器拒绝冲泡咖啡，因为它是一个茶壶。

420-增强您的冷静：在 Twitter 的原始版本中使用，当应用程序被限制速率时。这是对电影《拆弹专家》的引用。

### 设计您的 API

在软件设计过程的早期考虑 API 的设计非常重要。在发布后更改 API 的端点将需要更新依赖于这些端点的任何服务。如果 API 发布供公众使用，则通常需要保持向后兼容。在规划端点、接受的 HTTP 方法、所需的输入类型和返回的 JSON 结构上花费的时间将在长远节省下来。

通常，可以找到与您特定用例或行业相关的指南，因此请务必提前进行研究。在我们的智能家居 API 示例中，我们将从**万维网联盟**（**WC3**）关于 IoT 设备的推荐中汲取灵感。WC3 是致力于制定 Web 标准的最有影响力的组织之一，他们的 IoT 倡议被称为**物联网**（**WoT**）。您可以在[`www.w3.org/WoT/`](https://www.w3.org/WoT/)了解更多信息。

根据 WoT 指南，每个设备都应包含有关模型的信息以及可与设备一起使用的操作列表。以下是 WoT 标准推荐的一些端点：

![图 4.10：标准 WoT 路由表](img/C14587_04_10.jpg)

###### 图 4.10：标准 WoT 路由表

这种设计有两个原因很有用-首先，因为它符合标准，这给用户一组期望。其次，使用诸如`/properties/`和`/actions/`之类的辅助端点使用户能够通过在这些端点请求附加信息来发现 API 的使用方式。

添加到房屋的每个设备都应该有`/model/`、`/properties/`和`/actions/`端点。我们将在我们的 API 中将上表中显示的端点映射到每个设备上。以下树状图显示了从根端点开始的我们 API 的映射。

以下图中的第三级显示了`/devices/light/`端点，并且从该端点开始，我们有上表中列出的端点：

![图 4.11：智能家居 API 设计的树状图](img/C14587_04_11.jpg)

###### 图 4.11：智能家居 API 设计的树状图

作为端点返回的 JSON 的示例，我们将更仔细地查看前图中定义的`/devices/light/actions`路由。以下示例显示了包含名为`Fade`的单个操作的操作对象：

```js
"actions": {
  "fade": {
    "title": "Fade Light",
    "description": "Dim light brightness to a specified level",
    "input": {
      "type": "object",
      "properties": {
        "level": {
          "type": "integer",
          "minimum": 0,
          "maximum": 100
        },
        "duration": {
          "type": "integer",
          "minimum": 0,
          "unit": "milliseconds"
        }
      }
    },
    "links": [{"href": "/light/actions/fade"}]
  }
}
```

我们的`fade`操作是基于 Mozilla 在其 WoT 文档中提出的建议。他们创建了这份文档，目标是补充 W3C 提出的标准，并包含了许多代表 IoT 设备及其相关操作的 JSON 示例。

注意对象包含操作的名称、操作的描述以及使用操作的接受值。在适用的情况下，包含单位的度量单位也总是一个好主意。通过持续时间，我们知道它是以毫秒为单位的；如果没有这些信息，我们就不知道"1"实际上是什么意思。

通过阅读前面的 JSON，我们可以看到我们需要发送一个请求，其中包含所需的照明级别（0 到 100）的数字，以及另一个数字来指定调暗的时间长度。使用`curl`，我们可以这样淡化灯光：

```js
curl -sd "level=80&duration=500" -X PUT localhost:3000/lightBulb/actions/fade
```

根据 API 操作描述，前面的请求应该导致灯泡在 500 毫秒的时间内淡出到 80%的亮度。

#### 注意

**Swagger 文档**:虽然本书不涉及，但你应该了解的另一个项目是 Swagger。这个项目有助于自动化创建、更新和显示 API 文档，并与 Node.js 和 Express 很好地配合。

Swagger 生成的交互式文档示例可在[`petstore.swagger.io/`](https://petstore.swagger.io/)中看到。

### 练习 18：创建操作路由

在这个练习中，我们的目标是创建一个新的路由文件，返回关于`fade`操作的信息，这是我们在上一节中看到的。这个练习的起点将是我们在*练习 17，创建和导入路由文件*结束时留下的地方。

#### 注意

这个示例的完整代码可以在[`github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson04/Exercise18`](https://github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson04/Exercise18)找到。

执行以下步骤完成练习：

1.  在`routes`文件夹中创建一个名为`devices`的子文件夹：

```js
mkdir routes/devices
```

1.  将`routes/index.js`复制到`routes/devices/light.js`：

```js
cp routes/index.js routes/devices/light.js
```

1.  接下来，我们将打开上一个练习中的`/routes/devices/light.js`并修改它。找到第 6 行，应该包含以下内容：

```js
info.message = "Welcome home! Our first endpoint.";
```

我们将用一个大块的 JSON 代替前面的行，表示所有设备操作的列表：

```js
  let info =    {
    "actions": {
      "fade": {
        "title": "Fade Light",
        "description": "Dim light brightness to a specified level",
        "input": {
          "type": "object",
          "properties": {
            "level": {
              "type": "integer",
              "minimum": 0,
              "maximum": 100
            },
```

在我们的情况下，唯一的操作是`fade`。这个操作将在一定的时间内（以毫秒为单位）改变灯泡的亮度级别。这个端点不包含实现功能的逻辑，但它将返回与之交互所需的细节。

1.  在`server.js`文件中，导入我们新创建的设备路由：

```js
let light = require('./routes/devices/light');
```

1.  现在我们将告诉 Express 使用我们的`light`对象来使用前面的路由：

```js
app.use('/devices/light', light);
```

1.  使用`npm start`运行程序：

```js
npm start
```

1.  使用`curl`和`jq`测试路由：

```js
curl -s localhost:3000/devices/light | jq
```

如果你正确复制了前面的代码，你应该得到一个格式化的 JSON 对象，表示`fade`操作如下：

![图 4.12：localhost:3000/devices/light 的 cURL 响应](img/C14587_04_12.jpg)

###### 图 4.12：localhost:3000/devices/light 的 cURL 响应

### 进一步模块化

在项目文件中，我们将通过创建一个`lightStructure.js`文件进一步分离灯路由，其中只包含表示灯的 JSON 对象。我们不会包括包含`model`、`properties`和`action`描述的长字符串的 JSON。

#### 注意

在本节中对所做更改不会有练习，但你可以在[`github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson04/Example/Example18b`](https://github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson04/Example/Example18b)找到代码。

*练习 19*将使用在`Example18b`文件夹中找到的代码开始。

将静态数据（如端点对象和单独文件的函数）分离是有用的。`lightStructure.js`将包含表示模型、属性和操作的数据。这使我们能够专注于`light.js`中端点的逻辑。有了这个，我们将有四个端点，每个端点都返回 JSON 灯对象的相关部分：

```js
// Light structure is imported at the top of the file
const lightStructure = require('./lightStructure.js');
// Create four routes each displaying a different aspect of the JSON object
router.get('/', function(req, res, next) {
  let info = lightStructure;
  res.json(info);
});
router.get('/properties', function(req, res, next) {
  let info = lightStructure.properties;
  res.json(info);
});
router.get('/model', function(req, res, next) {
  let info = lightStructure.model;
  res.json(info);
});
router.get('/actions', function(req, res, next) {
  let info = lightStructure.actions;
  res.json(info);
});
```

在处理像`lightStructure.js`中找到的大块 JSON 时，可以使用 GUI 可视化工具非常有用。一个例子是[`jsoneditoronline.org/`](https://jsoneditoronline.org/)，它提供了一个工具，允许您在页面的左侧部分粘贴一个 JSON 块，并在右侧将其可视化为类似树状对象的形式：

![](img/C14587_04_13.jpg)

###### 图 4.13：在线 JSON 资源管理器/编辑器

可在可视化的任一侧进行更改并复制到另一侧。这很有用，因为 JSON 对象变得越复杂，就越难以看到属性中存在多少级别。

## 对发送到端点的输入进行类型检查和验证

虽然类型检查和验证对于创建 API 并不是严格要求的，但使用它们可以减少调试时间并帮助避免错误。对端点的输入进行验证意味着可以专注于返回期望的结果的代码编写，而不必考虑输入超出预期范围所产生的许多边缘情况。

由于这个任务在创建 API 时非常常见，因此已经创建了一个库来简化验证 Express 端点的输入。使用**express-validator**中间件，我们可以简单地将输入要求作为参数传递给我们的端点。例如，我们在*练习 18*中返回的 JSON 对象描述的要求，可以用以下数组表示：

```js
  check('level').isNumeric().isLength({ min: 0, max: 100 }),
  check('duration').isNumeric().isLength({ min: 0 })
]
```

如您所见，它包含了每个预期输入的条目。对于这些输入的每一个，我们执行两个检查。第一个是`.isNumeric()`，用于检查输入是否为数字。第二个是`.isLength()`，用于检查长度是否在指定的最小到最大范围内。

### 练习 19：创建带有类型检查和验证的路由

#### 注意

此示例的完整代码可以在[`github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson04/Exercise19`](https://github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson04/Exercise19)找到。

在这个练习中，我们将通过在`routes/devices/light.js`文件中添加一个接受`PUT`请求的路由`/actions/fade`来扩展。

路由将检查请求是否符合我们在*练习 18，返回表示动作路由的 JSON*中添加到`devices/light`端点的`fade`动作对象指定的标准。这包括以下方面：

+   请求包含级别和持续时间值。

+   级别和持续时间的值是整数。

+   级别值介于 0 和 100 之间。

+   持续时间值大于 0。

执行以下步骤完成练习：

1.  安装`express-validator`，这是一个中间件，用于在`express`中轻松使用`validation`和`sanitization`函数包装`validator.js`：

```js
npm install -s express-validator
```

1.  通过将`routes/devices/light`放在第 2 行导入`express-validator`库中的`check`和`validationResult`函数，就在`express`的`require`语句下方：

```js
const { check, validationResult } = require('express-validator/check');
```

1.  在上一练习中编写的`route.get`函数下面，创建以下函数来处理`PUT`请求：

```js
// Function to run if the user sends a PUT request
router.put('/actions/fade', [
    check('level').isNumeric().isLength({ min: 0, max: 100 }),
    check('duration').isNumeric().isLength({ min: 0 })
  ],
  (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(422).json({ errors: errors.array() });
    }
    res.json({"message": "success"});
});
```

1.  使用`npm start`运行 API：

```js
npm start
```

1.  对`/devices/light/actions/fade`进行`PUT`请求，使用不正确的值(`na`)来测试验证：

```js
curl -sd "level=na&duration=na" -X PUT \
http://localhost:3000/devices/light/actions/fade | jq
```

`-d`标志表示要传递给端点的“数据”值。`-X`标志表示 HTTP 请求类型。

如果前面的步骤执行正确，当我们对`/devices/light/actions/fade`进行`PUT`请求时，如果级别和持续时间的值为非数字，我们应该会收到错误：

![图 4.14：/device/light/actions/fade 路由的 cURL 错误响应，数据不正确](img/C14587_04_14.jpg)

###### 图 4.14：/device/light/actions/fade 路由的 cURL 错误响应

1.  接下来，我们将像以前一样进行`PUT`请求，但使用正确的值`50`和`60`：

```js
curl -sd "level=50&duration=60" -X PUT \
http://localhost:3000/devices/light/actions/fade | jq
```

发送具有正确范围内值的`PUT`请求应返回以下内容：

![图 4.15：/device/light/actions/fade 路由的 cURL 响应与正确数据](img/C14587_04_15.jpg)

###### 图 4.15：/device/light/actions/fade 路由的 cURL 响应与正确数据

上述截图表明`PUT`请求成功。

## 有用的默认值和简单的输入

因此，我们已经看到了对端点输入施加限制如何有所帮助。然而，过多的限制和要求可能会妨碍 API 的用户体验。让我们更仔细地看一下灯泡淡入淡出动作。为了允许在一段时间内淡入淡出的功能，我们要求用户传递一个持续时间的值。许多人已经有使用物理灯泡上的淡入淡出动作的经验。

对于物理灯泡，我们知道我们通过调节物理开关或其他输入来输入我们期望的亮度级别。持续时间不一定是这个过程的一部分，或者用户有意识地考虑过。这会导致期望您应该能够仅通过所需级别来淡化光线。

因此，我们应该考虑使`duration`值变为可选。如果没有收到`duration`值，脚本将退回到默认值。这使我们能够满足用户的期望，同时仍允许那些想要指定持续时间的用户进行精细控制。

### 练习 20：使持续时间输入变为可选

#### 注意

此示例的完整代码可以在[`github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson04/Exercise20`](https://github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson04/Exercise20)找到。

在这个练习中，我们将修改淡入淡出动作，使持续时间成为可选输入。如果没有提供持续时间值，我们将修改我们的淡入淡出动作端点，使用默认值 500 毫秒：

1.  在`routes/devices/light.js`中，通过在函数链中添加`.optional()`来修改验证`duration`的行。它应该是这样的：

```js
check('duration').isNumeric().optional().isLength({ min: 0 })
```

1.  在`routes/devices/light.js`中，删除`return`语句，并在相同位置添加以下内容：

```js
let level = req.body.level;
let duration;
if(req.body.duration) {
  duration = req.body.duration;
} else {
  duration = 500;
}
```

上述代码使用`level`输入创建了一个`level`变量，并初始化了一个空变量用于持续时间。接下来，我们检查用户是否提供了`duration`输入。如果是，我们将持续时间设置为该值。如果没有，我们将`duration`设置为`500`。

1.  现在，我们将使用我们的`level`和`duration`变量创建一个名为`message`的`message`对象。然后，我们将将该`message`对象返回给客户端：

```js
let message = `success: level to ${level} over ${duration} milliseconds`;
res.json({"message": message});
```

1.  最后，我们将将第二个路由与我们的函数关联起来，以便向`/devices/light`发送`PUT`请求执行与`/devices/light/actions/fade`相同的功能。这是通过将`router.put`的第一个参数更改为包含旧值和新值`/`的数组来实现的。`router.put`部分的开头应该是这样的：

```js
// Function to run if user sends a PUT request
router.put(['/', '/actions/fade'], [
    check('level').isNumeric().isLength({ min: 0, max: 100 }),
    check('duration').isNumeric().optional().isLength({ min: 0 })
  ],
  (req, res) => {
```

1.  现在我们已经完成了编码部分，我们将打开服务器进行测试：

```js
npm start
```

1.  在一个终端中运行服务器，打开另一个终端使用`curl`进行一些测试。在第一条命令中，我们将检查我们的新默认端点是否正常工作，并且在没有提供持续时间时使用我们的默认值：

```js
curl -sd "level=50" -X PUT http://localhost:3000/devices/light | jq
```

如果您已经正确复制了所有内容，您应该会看到这样的输出：

![图 4.16：/device/light 路由的 cURL 响应，没有指定持续时间](img/C14587_04_16.jpg)

###### 图 4.16：/device/light 路由的 cURL 响应，没有指定持续时间

1.  我们还希望确保提供`duration`值会覆盖默认值。我们可以通过进行 cURL 请求来测试这一点，该请求指定了`duration`值：

```js
curl -sd "level=50&duration=250" -X PUT http://localhost:3000/devices/light | jq
```

当将`250`指定为`duration`值时，我们应该在响应中看到`level`将会变为 250 毫秒以上的确认：

![图 4.17：/device/light 路由的 cURL 响应，指定了持续时间](img/C14587_04_17.jpg)

###### 图 4.17：指定持续时间的/device/light 路由的 cURL 响应

通过这些更改，我们现在已经将`fade`设置为`/devices/light`的默认操作，并且如果未提供持续时间输入，则给出了默认值。值得注意的是，我们现在有两个与`/devices/light`端点相关联的函数：

+   `HTTP GET /devices/light`：这将返回与灯交互的信息。

+   `HTTP PUT /devices/light`：这执行灯的默认操作。

多种方法重复使用相同的端点是一个很好的做法。另一个常见的例子是博客条目，其中 API 可能具有基于使用的方法的四个函数的单个端点：

+   `HTTP POST /blog/post/42`：这将创建 ID 为 42 的博客文章。

+   `HTTP GET /blog/post/42`：这将以 JSON 对象返回博客文章＃42。

+   `HTTP PUT /blog/post/42`：这通过发送新内容编辑博客文章＃42。

+   `HTTP DELETE /blog/post/42`：这将删除博客文章＃42。

这在逻辑上使用 REST 模型是有意义的，其中每个端点代表可以以各种方式进行交互的资源。

在我们的案例中，我们已经向`/devices/light`路由发出了`PUT`请求，触发了`fade`函数。可以说，一个打开和关闭灯的`switch`函数更符合大多数人对灯的默认操作的期望。此外，开关将是更好的默认选项，因为它不需要客户端的任何输入。Fade 之所以被选择是因为认为开关过于简单。

我们不会深入讨论`switch`函数，但它可能包含类似以下代码段的内容，允许客户端指定所需的状态。如果未指定状态，则它将成为当前值的相反值：

```js
if(req.body.state) {
  state = req.body.state;
} else {
  state = !state;
}
```

## 中间件

Express 中的中间件函数是在与端点关联的函数之前运行的函数。一些常见的例子包括在运行端点的主函数之前记录请求或检查身份验证。在这些情况下，记录和身份验证函数将在使用它们的所有端点中是常见的。通过使用中间件，我们可以重用在端点之间常见的代码。 

使用 Express，我们可以通过使用`app.use()`来运行所有端点的中间件函数。例如，如果我们想要创建一个在运行主路由之前将请求记录到控制台的函数，我们可以编写一个`logger`中间件：

```js
var logger = function (req, res, next) {
  // Request is logged
  console.log(req);
  // Call the special next function which passes the request to next function
  next();
}
```

要使记录器在所有端点上运行，我们告诉我们的应用程序使用以下内容：

```js
app.use(logger);
```

如果我们希望我们的中间件函数仅在某些路由上运行，我们可以直接附加它：

```js
app.use('/devices/light', logger, light);
```

对于一些或所有路由，可以使用多个中间件函数，没有限制。当使用多个中间件函数时，它们按照在代码中声明的顺序调用。当一个中间件函数完成时，它将`req`和`res`对象传递给链中的下一个函数：

![图 4.18：中间件链接图](img/C14587_04_18.jpg)

###### 图 4.18：中间件链接图

前面的图表可视化了一个请求过程，其中一旦服务器接收到请求，它将运行第一个中间件函数，将结果传递给第二个中间件函数，当完成时，最终运行我们的`/devices/light`目标路由。

在下一节中，我们将创建自己的中间件来检查客人是否已经签到以获取身份验证令牌。

### 练习 21：设置需要身份验证的端点

#### 注意

此示例的完整代码可以在[`github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson04/Exercise21`](https://github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson04/Exercise21)找到。

在下一个练习中，我们将通过添加一个需要身份验证的端点来完善我们的项目，该身份验证需要使用**JSON Web Token**（**JWT**）。我们将创建两个新的端点：第一个`restricted light`，与`light`相同，但需要身份验证。第二个端点`check-in`允许客户端通过向服务器发送他们的名称来获取令牌。

#### 注意

**JWT 和安全性**：此练习旨在突出 JWT 身份验证的工作原理。在生产中，这不是安全的，因为没有办法验证客户端提供的名称是否真实。

在生产中，JWT 还应包含一个到期日期，客户端必须在该日期之前更新令牌以继续使用。例如，给移动应用客户端的令牌可能具有 7 天的到期日期。客户端可能在启动时检查令牌是否即将到期。如果是这样，它将请求更新的令牌，应用程序的用户将不会注意到这个过程。

然而，如果移动应用的用户多天没有打开它，该应用将要求用户重新登录。这增加了安全性，因为任何可能找到 JWT 的第三方只有很短的时间来使用它。例如，如果手机丢失并在几天后被找到，许多使用带有到期日期的 JWT 的应用程序将需要再次登录以与所有者的帐户交互。

执行以下步骤以完成练习：

1.  创建一个带有随机密钥值的`config.js`文件：

```js
let config = {};
config.secret = "LfL0qpg91/ugndUKLWvS6ENutE5Q82ixpRe9MSkX58E=";
module.exports = config;
```

前面的代码创建了一个`config`对象。它将`config`的`secret`属性设置为一个随机字符串。然后，导出`config`对象。

重要的是要记住，密钥是随机的，因此您的密钥应该与此处显示的密钥不同。没有固定的方法来生成随机字符串，但在命令行上的一个简单方法是使用`openssl`，它应该默认安装在大多数 Linux 和 Mac 操作系统上：

```js
openssl rand -base64 32
```

1.  使用`npm`安装`jwt-simple`：

```js
npm install -s jwt-simple
```

1.  为`check-in`端点创建`routes/check-in.js`文件。导入以下模块，我们将需要使用它们：

```js
const express = require('express');
const jwt = require('jwt-simple');
const { check, validationResult } = require('express-validator/check');
const router = express.Router();
// import our config file and get the secret value
const config = require('../config');
const secret = config.secret;
```

1.  在`routes/check-in.js`中的导入下面，我们将创建一个需要`name`的字符串值的`post`路由。然后，我们将对发送的所有信息进行编码成 JWT。然后将此 JWT 返回给客户端用于身份验证：

```js
router.post('/', [
    check('name').isString()
  ],
  (req, res) => {
    // If errors return 422, client didn't provide required values
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(422).json({ errors: errors.array() });
    }
    // Otherwise use the server secret to encode the user's request as a JWT
    let info = {};
    info.token = jwt.encode(req.body, secret);
    res.json(info);
});
// Export route so it is available to import
module.exports = router;
```

1.  在`server.js`中，还要导入`config.js`和`jwt-simple`，并设置密钥值：

```js
// Import library for working with JWT tokens
const jwt = require('jwt-simple');
// import our config file and get the secret value
const config = require('../config');
const secret = config.secret;
```

1.  在`server.js`中，添加一个中间件函数，以查看用户是否具有有效令牌：

```js
// Check if the requesting client has checked in
function isCheckedIn(req, res, next) {
  // Check that authorization header was sent
  if (req.headers.authorization) {
    // Get token from "Bearer: Token" string
    let token = req.headers.authorization.split(" ")[1];
    // Try decoding the client's JWT using the server secret
    try {
      req._guest = jwt.decode(token, secret);
    } catch {
      res.status(403).json({ error: 'Token is not valid.' });
    }
    // If the decoded object has a name protected route can be used
    if (req._guest.name) return next();
  }
  // If no authorization header or guest has no name return a 403 error
  res.status(403).json({ error: 'Please check-in to recieve a token.' });
}
```

1.  在`server.js`中，添加`check-in`端点和第二个`restricted-light`端点的 light：

```js
// Import our index route
let index = require('./routes/index');
let checkIn = require('./routes/check-in');
let light = require('./routes/devices/light');
// Tell Express to use our index module for root URL
app.use('/', index);
app.use('/check-in', checkIn);
app.use('/devices/light', light);
app.use('/devices/restricted-light', isCheckedIn, light);
```

`server.js`的部分，其中导入和设置路由的代码应该看起来像前面的代码，添加了三行新代码。您可以看到有一行用于导入`check-in`路由，另外两行用于创建我们的新路由。请注意，我们不需要导入`restricted-light`，因为它重用了`light`对象。`restricted-light`与`light`的关键区别在于使用了`isCheckedIn`中间件函数。这告诉`express`在提供 light 路由之前运行该函数。

1.  使用`npm start`打开服务器：

```js
npm start
```

1.  打开另一个终端窗口，并运行以下命令以获取签名的 JWT 令牌：

```js
TOKEN=$(curl -sd "name=john" -X POST http://localhost:3000/check-in \
  | jq -r ".token")
```

前面的命令使用`curl`将名称发布到`check-in`端点。它获取服务器的结果并将其保存到名为`TOKEN`的 Bash 变量中。`TOKEN`变量是在运行该命令的终端窗口中本地的；因此，如果关闭终端，则需要再次运行。要检查它是否正确保存，告诉 Bash shell 打印该值：

```js
echo $TOKEN
```

以下是前面代码的输出：

![图 4.19：在 Bash shell 中检查$TOKEN 的值](img/C14587_04_19.jpg)

###### 图 4.19：在 Bash shell 中检查$TOKEN 的值

您应该看到一个 JWT 令牌，如前面的图所示。

1.  通过在终端中运行以下命令，向`restricted-light`发送带有身份验证令牌的 cURL 请求：

```js
curl -sd "level=50&duration=250" -X PUT \
  -H "Authorization: Bearer ${TOKEN}" \
  http://localhost:3000/devices/restricted-light \
  | jq
```

它应该返回一个成功的淡入效果，如下图所示：

![图 4.20：使用 JWT 成功向 restricted-light 发送 cURL 请求](img/C14587_04_20.jpg)

###### 图 4.20：使用 JWT 成功向 restricted-light 发送 cURL 请求

1.  在终端中向`restricted-light`发送不带身份验证令牌的`curl`请求：

```js
curl -sd "level=50&duration=250" -X PUT \
  http://localhost:3000/devices/restricted-light \
  | jq
```

相比之下，发送相同的请求但不带端点会返回错误：

![图 4.21：尝试在没有 JWT 的情况下 cURL restricted-light](img/C14587_04_21.jpg)

###### 图 4.21：尝试在没有 JWT 的情况下 cURL restricted-light

我们现在已经设置了一个端点来分发身份验证令牌，并且有一个需要这些令牌的受保护的端点。我们现在可以通过重用我们的`isCheckedIn`函数与任何新的端点来添加需要身份验证令牌的额外路由。我们只需要将该函数作为第二个参数传递给 Express，就像在`server.js`中所做的那样。

## JWT 的内容

在上一个练习中，在*步骤 7*期间，我们从服务器请求了一个令牌，并将该值保存到我们的本地终端会话中。为了使练习有效，JWT 应该有三个部分，由句点分隔。如果我们将从`echo $TOKEN`命令返回的 JWT 放入网站 jwt.io 中，我们可以更仔细地查看 JWT 的内容。

此外，将您的秘密值粘贴到 GUI 的右下角，应在左下角显示“签名已验证”。这告诉我们，查看的 JWT 是使用私有签名创建的：

![图 4.22：显示 JWT.io 与 JWT 数据](img/C14587_04_22.jpg)

###### 图 4.22：显示 JWT.io 与 JWT 数据

JWT 网站允许我们轻松地可视化 JWT 的三个部分代表什么。红色的第一部分是标头，即描述所使用的编码标准的信息。紫色部分是有效载荷-它包含在创建令牌时服务器验证的数据，在我们的情况下只是一个名称。最后，蓝色部分是签名，它是使用服务器的秘密对其他两个部分的内容进行哈希的结果。

在前面的示例中，**有效载荷**部分是三个部分中最小的。这并不总是这样，因为红色和蓝色部分的大小是固定的，而紫色部分取决于有效载荷的大小。如果我们使用`check-in`端点从服务器请求另一个令牌，那么我们不仅提供一个名称，还提供电子邮件和电话号码。这意味着我们将看到一个具有较大紫色部分的结果令牌：

![图 4.23：JWT.io 显示具有较大负载的令牌](img/C14587_04_23.jpg)

###### 图 4.23：JWT.io 显示具有较大负载的令牌

## MongoDB

许多 API 使用数据库来跟踪 API 读取和写入的基础数据。在其他情况下，例如物联网，端点的功能可能会更新真实对象。即使在跟踪或触发真实对象或事件时，跟踪数据库中的预期状态也是一个好主意。可以快速访问和操作数据库表示。

我们不会深入讨论数据库的使用和设计；但是，我们将简要讨论如何使用数据库来扩展 API 的功能。很少会有一个 API 在不使用某种数据库的情况下超越`hello world`。

与 Node.js 一起使用最广泛的数据库是 MongoDB。MongoDB 是一个面向对象的库，具有方便的语法，可用于处理 JSON 对象。除了将数据存储为类似 JSON 的对象之外，它不需要使用模式。这意味着对象的属性可以随时间改变，而无需对数据库进行任何配置。

例如，我们可以开始在数据库中跟踪事件，这些事件只包含请求正文和时间戳：

```js
{
  "timestamp": 1556116316288,
  "body" : { "level" : "50", "duration" : "250" }
}
```

我们可能会从一个非常简单的事件日志开始，然后决定随着每个事件保存额外的细节。例如，如果我们包括授权数据和请求的确切路径，我们的日志对象将如下所示：

```js
{
  "timestamp": 1556116712777,
  "body" : { "level" : "20", "duration" : "500" },
  "path" : "/devices/light",
  "token" : null
}
```

如果使用 SQL 数据库，我们首先需要向数据库模式添加`path`和`token`列。MongoDB 的灵活性是其伟大特性之一，以及将其添加到已经使用 JSON 进行数据操作的项目的简单性。

通常，API 将完全基于数据库，就像大多数社交媒体应用一样。例如，对于 Twitter、Facebook 和 Instagram，每个用户、帖子和评论最终都是数据库中的一个条目，通过 API 向客户端软件提供访问。

我们不会深入讨论如何在 API 中使用数据库，但是额外的文件夹包含了如何设置 MongoDB 并将其与此 API 一起使用以记录事件的说明（请参见下面的注释）。

使用 JWT 进行事件记录将允许我们将受限端点的任何恶意使用与特定的 JWT 关联起来。通过使用日志系统并强制在所有端点上使用 JWT，我们可以将任何请求的操作与`smartHouse`关联到特定用户。在恶意使用的情况下，JWT 可以被列入黑名单。当然，这将需要更严格的要求来发放 JWT；例如，要求客人出示政府发行的照片身份证明。

#### 注意

**带有 MongoDB 日志记录示例的中间件**：您可以参考项目文件中名为`extra/mongo_logger_middleware`的文件夹，了解创建一个捕获所有信息的中间件的示例，包括请求的方法、数据和用户信息。类似的东西可以用来跟踪由谁发出的请求。

运行此代码时，您需要首先运行`npm install`。除此之外，确保您已经在本地安装并运行了 MongoDB。有关更多详细信息，请参阅文件夹中的 README 文件[`github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson04/Example/extra/mongo_logger_middleware`](https://github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson04/Example/extra/mongo_logger_middleware)。

### 活动 5：为键盘门锁创建 API 端点

在这个活动中，您需要为键盘门锁创建一个 API 端点。该设备需要一个新的端点来支持经过身份验证的用户能够创建一次性密码来打开门的用例。

执行以下步骤完成活动：

1.  创建一个新的项目文件夹并切换到该文件夹。

1.  初始化一个`npm`项目并安装`express`，`express-validator`和`jwt-simple`。然后，创建一个`routes`目录。

1.  创建一个`config.js`文件，其中应包含一个随机生成的秘密值。

1.  创建`routes/check-in.js`文件，以创建一个签到路由。

1.  创建一个名为`routes/lock.js`的第二个路由文件。首先导入所需的库和模块，然后创建一个空数组来保存我们的有效密码。

1.  在`routes/lock.js`中的代码下面，创建一个`GET`路由，用于`/code`，需要一个`name`值。

1.  在`routes/lock.js`中创建另一个路由。这个路由将是`/open`，需要一个四位数的代码，将被检查是否在`passCodes`数组中有效。在该路由下面，确保导出`router`，以便在`server.js`中使用。

1.  创建主文件，在其中我们的路由将在`server.js`中使用。首先导入所需的库，还有设置 URL 编码的 JSON。

1.  接下来，在`server.js`中，导入这两个路由，实现一个`404`捕获，并告诉 API 监听端口`3000`。

1.  测试 API 以确保它被正确完成。首先运行您的程序。

1.  程序运行时，打开第二个终端窗口，使用`/check-in`端点获取 JWT 并将值保存为`TOKEN`。然后，回显该值以确保成功。

1.  使用我们的 JWT 来使用`/lock/code`端点获取新名称的一次性验证码。

1.  两次向`/lock/open`端点发送代码，以获取第二个实例的错误。

#### 注意

此活动的解决方案可在第 594 页找到。

## 摘要

在本章中，我们探讨了使用 Node.js 创建 RESTful API 的用途。我们考虑了 API 的各种用途以及一些设计技巧。通过查看诸如 HTTP 代码和输入验证之类的方面，我们考虑了在创建和维护 API 时处理的常见问题。尽管如此，仍有许多 API 设计和开发领域尚未考虑。

继续提高您关于 API 设计和创建的知识的最佳方法是开始制作自己的 API，无论是在工作中还是通过个人项目。我们在本章的练习中创建的代码可以用作起点。尝试扩展我们在这里所做的工作，创建您自己的端点，最终创建您自己的 API。

在下一章中，我们将讨论代码质量。这将包括编写可读代码的技术，以及用于测试我们代码的技术。这些技术可以与您在这里学到的内容结合使用，以确保您创建的端点在项目增长时继续返回正确的值。
