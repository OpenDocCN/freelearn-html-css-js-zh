# 第七章：JavaScript 生态系统

## 学习目标

在本章结束时，您将能够做到以下事情：

+   比较不同的 JavaScript 生态系统

+   解释服务器端 JavaScript 的基本概念

+   构建一个 Node.js 和 Express 服务器

+   构建一个 React 前端网站

+   将前端框架与后端服务器结合起来

最后一章详细介绍了 JavaScript 生态系统，并教导学生如何使用 Node.js 的不同功能和部分，以及 Node 包管理器（NPM）。

## 介绍

在第五章“函数式编程”中，我们介绍了“函数式编程范式”。我们讨论了面向对象编程和函数式编程，讨论了两者之间的区别，并概述了为什么我们应该使用函数式编程。在第二部分中，我们讨论了函数式编程的关键概念，并演示了它们如何应用于 JavaScript 代码。

在过去的 10 多年里，JavaScript 生态系统已经大幅增长。JavaScript 不再只是用于在基本的 HTML 网页上添加动画等效果的编程语言。现在 JavaScript 可以用于构建完整的后端 Web 服务器和服务、命令行界面、移动应用程序和前端网站。在本章中，我们将介绍 JavaScript 生态系统，讨论使用 Node.js 在 JavaScript 中构建 Web 服务器，并讨论使用 React 框架在 JavaScript 中构建网站。

## JavaScript 生态系统

我们将讨论 JavaScript 生态系统的四个主要类别：前端、命令行界面、移动和后端。

+   前端 JavaScript 用于用户界面网站。

+   命令行界面（CLI）JavaScript 用于构建命令行任务，以帮助开发人员。

+   移动开发 JavaScript 用于构建手机应用程序。

+   后端 JavaScript 用于构建 Web 服务器和服务。

对于最初是为了在浏览器中嵌入简单应用程序而创建的语言来说，JavaScript 已经走了很长的路。

### 前端 JavaScript

前端 JavaScript 用于创建复杂和动态的用户界面网站。Facebook、Google Maps、Spotify 和 YouTube 等网站都严重依赖 JavaScript。在前端开发中，JavaScript 用于操作 DOM 和处理事件。许多 JavaScript 库，如 jQuery，已被创建以通过将每个浏览器的 DOM 操作 API 封装成标准化 API 来增加 JavaScript DOM 操作的效率和便利性。最常见的 DOM 操作库是 jQuery，在第三章“DOM 操作和事件处理”中进行了讨论。还创建了 JavaScript 框架，以更无缝地将 DOM 操作和事件与 HTML 设计方面整合在一起。前端开发中最常见的两个 JavaScript 框架是 AngularJS 和 React。AngularJS 由 Google 创建和维护，React 由 Facebook 创建和维护。

Facebook 和 Google 管理其各自框架的错误修复和版本发布。React 将在本章的后面部分进行更详细的讨论。

### 命令行界面

命令行集成（CLI）JavaScript 通常用于创建实用程序，以帮助开发人员处理重复或耗时的任务。JavaScript 的 CLI 程序通常用于诸如代码检查、启动服务器、构建发布、转译代码、文件最小化以及安装开发依赖和包等任务。JavaScript 的 CLI 程序通常是用 Node.js 编写的。Node.js 是一个跨平台环境，允许开发人员在浏览器之外执行 JavaScript 代码。Node.js 将在本章的后面部分进行更详细的讨论。许多开发人员在日常开发中依赖 CLI 实用程序。

### 移动开发

**使用 JavaScript 进行移动开发**正在迅速成为主流。自智能手机兴起以来，移动开发人员已成为炙手可热的商品。尽管 JavaScript 不能在大多数移动操作系统上本地运行，但存在允许将 JavaScript 和 HTML 构建到 Android 和 IOS 手机应用程序中的框架。JavaScript 移动开发最常见的框架是 Ionic、React Native 和 Cordova/PhoneGap。这些框架都允许您编写 JavaScript 来构建应用程序的框架和逻辑，然后将 JavaScript 编译为本机移动操作系统代码。移动开发框架非常强大，因为它们允许我们使用首选的 JavaScript 构建完整的移动应用程序。

### 后端开发

**使用 JavaScript 进行后端开发**通常使用 Node.js。Node.js 可用于构建强大的 Web 服务器和服务。正如前面所述，Node.js 及其在后端服务器开发中的应用将在本章的后续部分中进行更详细的讨论。

JavaScript 生态系统非常广泛。几乎可以用 JavaScript 编写任何类型的程序。尽管现代 JavaScript 具有许多框架和功能，但重要的是要记住，框架不能取代对核心 JavaScript 的深入理解。框架很好地封装了核心 JavaScript，使我们能够执行强大的任务，如构建移动和桌面应用程序，但如果不深刻理解 JavaScript 和异步编程的核心原则，应用程序可能会出现缺陷。

## Node.js

**Node.js**（简称 Node），由 Ryan Dahl 于 2009 年开发，是最流行的非浏览器 JavaScript 引擎。Node 是一个基于 Chrome 的 V8 JavaScript 引擎的开源、跨平台 JavaScript 运行时环境。它用于在浏览器之外运行 JavaScript 代码，用于非客户端的应用程序。

与 Chrome 中的 Google V8 JavaScript 引擎一样，Node.js 使用单线程、事件驱动、异步架构。它允许开发人员使用 JavaScript 的事件驱动编程风格来构建 Web 服务器、服务和 CLI 工具。如*第二章，异步 JavaScript*中所讨论的，JavaScript 是一种非阻塞和事件驱动的编程语言。JavaScript 的异步特性（单线程事件循环），加上 Node 的轻量设计，使我们能够构建非常可扩展的网络应用程序，而无需担心线程。

#### 注意

如*第二章，异步 JavaScript*中所讨论的，JavaScript 是单线程的。在单线程上运行的同步代码是阻塞的。CPU 密集型操作将阻塞事件，如 I/O 文件系统操作和网络操作。

### 设置 Node.js

Node.js 可以从 Node.js 网站下载，网址为[`nodejs.org/en/`](https://nodejs.org/en/)。有两个可供下载的版本：**长期支持（LTS）**版本和当前版本。我们建议您下载 LTS 版本。当前版本具有最新的功能，但可能不完全没有错误。请务必遂行您操作系统的特定安装说明。可以为所有三种主要操作系统下载安装程序文件，并且可以使用许多软件包管理器安装 Node.js。Node.js 安装调试不在本书的范围之内。但是，可以通过谷歌搜索轻松找到安装提示和调试提示。

#### 注意

Node.js 的下载链接如下：[`nodejs.org/en/download/`](https://nodejs.org/en/download/)。

一旦 Node.js 被下载并安装，就可以使用`node`命令从终端运行它。在执行此命令后不跟任何参数，将运行 Node.js 终端。JavaScript 代码可以直接在终端中输入，就像浏览器的调试控制台一样。重要的是要注意，在终端实例之间没有状态传递。当运行 Node.js 命令行的终端实例关闭时，所有计算将停止，并且 Node.js 命令行进程使用的所有内存将释放回操作系统。要使用 Node.js 运行 JavaScript 代码文件，只需在`node`命令后直接添加文件路径。例如，以下命令将在`./path/to/file`位置以文件名`my_file.js`运行文件：`node ./path/to/file/my_file.js`。

### Node 包管理器

Node.js 是一个开源平台。Node 的最大优势之一是可用的开源第三方库，称为模块。Node 使用**Node 包管理器（NPM）**来处理应用程序使用的第三方模块的安装和管理。NPM 通常与 Node.js 一起安装。要测试 NPM 是否已正确安装到 Node，请打开终端窗口并运行`npm -v`命令。如果 NPM 已正确安装，终端将打印出当前 NPM 的版本。如果 NPM 未随 Node 一起安装，可能需要重新运行 Node.js 安装程序。

#### 注意

本节未涵盖的所有功能的 NPM 文档可以在[`docs.npmjs.com/`](https://docs.npmjs.com/)找到。

在*第一章，介绍 ECMAScript 6*中，我们学习了关于 ES6 模块。非常重要的是，我们要区分 ES6 模块和 Node.js 模块。Node.js 模块是在 ES6 和原始 JavaScript 对模块的支持之前创建的。虽然 Node.js 模块和 ES6 模块用于相同的目的，但它们不遵循相同的技术规范。Node.js 模块和 ES6 模块的加载、解析和构建方式不同。Node.js 模块是同步从磁盘加载、同步解析和同步构建的。在模块加载完成之前，没有其他代码可以运行。不幸的是，ES6 模块的加载方式不同。它们是异步从磁盘加载的。这两种不同的模块加载方法不兼容。在撰写本书时，Node.js 对 ES6 模块的支持处于测试阶段，并且默认情况下未启用。可以启用对 ES6 模块的支持，但我们建议您在 ES6 模块的完全支持发布之前使用标准的 Node 模块。

NPM 包是通过命令行使用`npm install`命令安装的。您可以使用此命令将特定包添加到项目中，或安装所有缺少的依赖项。如果没有向安装命令提供参数，`npm`将在当前目录中查找`package.json`文件。在`package.json`文件中，有一个`dependencies`字段，其中包含为 Node.js 项目安装的所有依赖项。NPM 将遍历依赖项列表，并验证该列表中指定的每个包是否已安装。`packages.json`中的依赖项列表将类似于以下代码片段中显示的代码：

```php
"dependencies": {
 "amqplib": "⁰.5.2",
 "body-parser": "¹.18.3",
 "cookie-parser": "¹.4.3",
 "express": "⁴.16.3",
 "uuid": "³.3.2"
}
```

###### 代码片段 6.1：package.json 中的依赖项列表

`package.json`中的依赖项字段列出了为项目安装的 NPM 模块，以及版本号。在这个片段中，我们安装了`amqplib`模块的版本为`0.5.2`或更高版本，安装了`body-parser`模块的版本为`1.18.3`或更高版本，以及其他几个模块。NPM 模块遵循语义化版本。版本号由三个数字组成，用句点分隔。第一个数字是主要版本。主要版本号的增加表示破坏向后兼容的重大更改。第二个数字是次要版本。次要版本号的更改表示发布了不会破坏向后兼容性的新功能。最后一个数字是补丁号。补丁号的增加表示修复错误或对功能进行小更新。补丁号的增加不包括新功能，也不会破坏向后兼容性。

#### 注意

有关语义化版本的更多信息，请访问[`www.npmjs.com/`](https://www.npmjs.com/)。

安装模块时，可以在`npm install`命令的*install*后添加参数（例如，`npm install express`）。参数可以是包名称、Git 存储库、**tarball**或文件夹。如果参数是一个包，NPM 将在其注册的包列表中搜索并安装与名称匹配的包。如果参数是 Git 存储库，NPM 将尝试从 Git 存储库下载并安装文件。如果没有提供适当的访问凭据，安装可能会失败。

#### 注意

请参阅 NPM 文档，了解如何从私有 git 存储库安装包。

如果参数是一个 tarball，NPM 将解压 tarball 并安装文件。tarball 可以通过指向 tarball 的 URL 或本地文件进行安装。最后，如果指定的参数是本地机器上的文件夹，NPM 将尝试从指定的文件夹安装 NPM 包。

在使用 NPM 安装包时，重要考虑包的安装方式。默认情况下，包是在本地项目范围内安装的，并且不会保存为项目依赖项。如果要安装一个 NPM 包，并希望将其保存在`package.json`中作为项目依赖项，必须在安装命令的包名称后包含`--save`或`-s`参数（例如，`npm install express -s`）。此参数告诉 NPM 将依赖项保存在`package.json`中，以便以后的`npm install`命令会安装它。

NPM 包可以安装在两个范围内：**全局范围**和**本地范围**。在本地范围内安装的包，或本地包，只能在安装它们的 Node.js 项目中使用。在全局范围内安装的包，或全局包，可以被任何 Node.js 项目使用。默认情况下，包是本地安装的。要强制安装模块为全局安装，可以在包名称后添加`-g`或`--global`标志到`npm install`命令（例如，`npm install express -g`）。

并不总是明显应该在哪里安装包，但如果不确定，可以遵循以下一般规则。如果要在具有`require()`函数的项目中使用包，请在本地安装包。如果计划在命令行上使用包，请全局安装包。如果仍然无法决定并且需要在项目和命令行中使用包，可以在两个地方都安装它。

### 加载和创建模块

Node.js 使用**CommonJS**风格的模块规范作为加载和处理模块的标准。CommonJS 是一个旨在为浏览器外的 JavaScript 指定 JavaScript 生态系统的项目。CommonJS 定义了一个模块规范，被 Node.js 采纳。模块允许开发人员封装功能，并仅向其他 JavaScript 文件公开所需部分的封装功能。

在 Node.js 中，我们使用 require 函数将模块导入到我们的代码中（`require('module')`）。`require`函数可以加载任何有效的 JavaScript 文件、NPM 模块或 JSON 文件。我们将使用`require`函数加载为我们的项目安装的任何 NPM 包。要加载一个模块，只需将模块的名称作为参数传递给`require`函数，并将返回的对象保存到一个变量中。例如，我们可以使用以下代码加载 NPM 模块`body-parser`：`const bodyParser = require( 'body-parser' )`。这将导入导出的函数和变量到`bodyParser`对象中。require 函数还可以用于加载 JavaScript 文件和 JSON 文件。要加载其中一个文件，只需将文件路径传递给`require`函数，而不是模块名称。如果未提供文件扩展名，Node.js 将默认查找 JavaScript 文件。

#### 注意

还可以通过 require 函数加载目录。如果提供的是目录而不是 JS 文件，则 require 函数将在指定目录中查找名为`index.js`的文件并加载该文件。如果找不到该文件，将抛出错误。

要创建一个模块，即 Node.js 模块，我们使用`module.exports`属性。在 Node.js 中，每个 JavaScript 文件都有一个名为`module`的全局变量对象。`module`对象中的`exports`字段定义了将从模块中导出的项目。当使用`require()`函数导入模块时，`require()`的返回值是模块的`module.exports`字段中设置的值。模块通常导出一个函数或具有每个导出的函数或变量的属性的对象。以下是导出模块的示例：

```php
module.exports = {
  exportedVariable,
  exportedFn
}
const exportedVariable = 10;
function exportedFn( args ){ console.log( 'exportedFn' ) ;}
```

###### Snippet 6.2：导出 Node.js 模块

### 练习 32：导出和导入 NPM 模块

要构建、导出和导入 NPM 模块，请执行以下步骤：

1.  为我们的模块创建一个名为`module.js`的 JavaScript 文件。

1.  将`module.exports`属性设置为一个对象。

1.  将`exportedConstant`字段添加到对象中，并将其值设置为`An exported constant!`

1.  将`exportedFunction`字段添加到对象中，并将其值设置为记录到控制台的函数，文本为`An exported function!`

1.  为我们的主要代码创建一个`index.js`文件。

1.  使用`require`函数从`module.js`导入模块，并将其保存到`ourModule`变量中。

1.  从 ourModule 中记录`exportedString`的值。

1.  从`ourModule`调用`exportedFunction`函数。

**代码**

##### module.js

```php
module.js
module.exports = {
 exportedString: 'An exported string!',
 exportedFunction(){ console.log( 'An exported function!' ) }
};
```

###### Snippet 6.3：将代码导出为模块

https://bit.ly/2M3SIsT

##### index.js

```php
const ourModule = require('./module.js');
console.log( ourModule.exportedString );
ourModule.exportedFunction();
```

###### Snippet 6.4：将代码导出为模块

https://bit.ly/2RwOIXP

**结果**

![图 6.1：测试值输出](img/Figure_6.1.jpg)

###### 图 6.1：测试值输出

您已成功构建、导出和导入 NPM 模块。

### 基本 Node.js 服务器

Node.js 最常见的应用是 Web 服务器。Node.js 使构建高效和可扩展的 Web 服务器变得非常容易，因为开发人员不需要担心线程。在本节中，我们将演示在 Node.js 中创建基本 Web 服务器所需的代码。

Node.js 服务器可以设置为 HTTP、HTTPS 或 HTTP2 服务器。在本例中，我们将创建一个基本的 HTTP 服务器。Node.js 通过 HTTP 模块提供了 HTTP 服务器的基本功能。使用 require 语句导入 HTTP 模块。如下所示：

```php
const http = require( 'http' );
```

###### Snippet 6.5：加载 HTTP 模块

这行代码将导入模块'HTTP'中包含的功能，并将其保存在变量`http`中以供以后使用。现在我们已经加载了 HTTP 模块，我们需要选择一个主机名和一个端口来运行我们的服务器。由于这个服务器只会在我们的计算机本地运行，我们可以使用机器内部本地网络的 IP 地址，即 localhost（'127.0.0.1'）作为我们的主机名地址。我们可以在任何尚未被其他应用程序使用的网络端口上运行我们的本地服务器。

您可以选择任何有效的端口号，但通常情况下程序不会默认使用端口`8000`，所以在这个演示中使用了这个端口号。在您的代码中添加一个变量来包含端口号和主机名。到目前为止的完整代码如下所示：

```php
const http = require('http');
const hostname = '127.0.0.1';
const port = 8000;
```

###### 代码段 6.6：简单服务器的常量

现在我们已经为我们的服务器设置了所有基本参数，我们可以编写代码来创建和启动服务器。HTTP 模块包含一个名为`createServer()`的函数，它返回一个服务器对象。这个函数可以接受一个可选的回调函数，作为 HTTP 请求监听器。当任何 HTTP 请求进入服务器时，提供的回调方法会被调用。我们需要使用带有请求监听器回调的`createServer`函数，这样我们的服务器才能正确地响应传入的 HTTP 请求。这是在以下代码段中显示的代码行：

```php
const server = http.createServer((req, res) => {  res.statusCode = 200;  res.setHeader('Content-Type', 'text/plain');  res.end('Welcome to my server!\n');});
```

###### 代码段 6.7：创建一个简单的服务器

在前面的代码段中，我们调用`create server`函数并将返回的服务器保存到`server`变量中。我们将一个回调传递给`createServer()`。这个回调接受两个参数：`req`和`res`。`req`参数表示传入的 HTTP 请求，`res`参数表示服务器的 HTTP 响应。在回调的第一行代码中，我们将响应状态码设置为`200`。响应中的`200`状态码表示服务器对 HTTP 请求成功。在状态码之后的一行中，我们在响应中设置了`Content-Type`头为`text/plain`。这一步告诉响应传入的数据将是纯文本。在回调的最后一行中，我们调用了`res.end()`函数。这个函数将传入的数据附加到响应中，然后关闭响应并将其发送回请求者。在这个代码段中，我们将`Welcome to my server!`字符串传递给`end()`函数。响应中附加了这个字符串，并将文本发送回请求者。我们的服务器现在使用这个处理程序处理所有对它的 HTTP 调用。

将我们的迷你服务器启动并运行的最后一步是在服务器对象上调用`.listen()`函数。`listen`函数在指定的`port`和`hostname`上启动 HTTP 服务器。一旦服务器开始监听，它就可以接受 HTTP 请求。以下代码段显示了如何使服务器在指定的`port`和指定的`hostname`上监听：

```php
server.listen( port, hostname, () => {  console.log('Server running at http://${hostname}:${port}/');});
```

###### 代码段 6.8：服务器开始在主机名和端口上监听

前面的代码段显示了如何调用`server.listen()`函数。传递给函数的第一个参数是我们的服务器将暴露在的端口号。第二个参数是我们的服务器将从中访问的主机名。在这个例子中，端口评估为`8000`，主机名评估为`127.0.0.1`（您的计算机的本地网络）。在这个例子中，我们的服务器将在`127.0.0.1:8000`上监听。传递给`.listen()`的最后一个参数是一个回调函数。一旦服务器开始在指定的端口和主机名上监听 HTTP 请求，提供的回调函数就会被调用。在前面的代码段中，回调函数只是打印出我们的服务器可以在本地访问的 URL。您可以将此 URL 输入到浏览器中，然后一个网页将加载。

### 练习 33：创建基本的 HTTP 服务器

要构建一个基本的 HTTP 服务器，请执行以下步骤：

1.  导入`http`模块。

1.  为主机名和端口设置变量，并分别给它们赋值`127.0.0.1`和`8000`。

1.  使用`http.createServer`创建服务器。

1.  为`createServer`函数提供一个回调，该回调接受参数`req`和`res`。

1.  将响应状态码设置为`200`。

1.  将响应内容类型设置为`text/plain`。

1.  使用`My first server!`响应请求

1.  使用`server.listen`函数使服务器监听指定的端口和主机。

1.  为`listen`函数提供一个回调，记录`Server running at ${server uri}`。

1.  启动服务器并加载已记录的网页。

**代码**

##### index.js

```php
const http = require( 'http' );
const hostname = '127.0.0.1';
const port = 8000;
const server = http.createServer( ( req, res ) => {
 res.statusCode = 200;
 res.setHeader( 'Content-Type', 'text/plain' );
 res.end( 'My first server!\n' );
} );
server.listen( port, hostname, () => console.log( 'Server running at http://${hostname}:${port}/' ) );
```

###### 代码片段 6.9：简单的 HTTP 服务器

https://bit.ly/2sihcFw

**结果**

![图 6.2：返回新的购物车数组](img/Figure_6.2.jpg)

###### 图 6.2：返回新的购物车数组

![图 6.3：返回新的购物车数组](img/Figure_6.3.jpg)

###### 图 6.3：返回新的购物车数组

您已成功构建了一个基本的 HTTP 服务器。

### 流和管道

流数据可能是 Node.js 中最复杂和最被误解的方面之一。流也可以说是 Node.js 提供的最强大的功能之一。流只是数据的集合，就像标准数组或字符串一样。主要区别在于，使用流时，所有数据可能不会同时可用。你可以把它想象成从 YouTube 或 Netflix 上流视频。你不需要在开始观看视频之前下载整个视频。视频提供者（YouTube 或 Netflix）以小块的方式向你的计算机发送视频。你可以开始观看视频的一部分，而不需要等待其他部分被加载。流非常强大，因为它们允许服务器和客户端不需要一次性将整个大量数据集加载到内存中。在 JavaScript 服务器中，流对于内存管理至关重要。

Node.js 中的许多内置模块依赖于流。这些模块包括 HTTP 模块（`http`）中的请求和响应对象，文件系统模块（`fs`）中的文件，加密模块（crypto）和子进程模块（`child_process`）。在 Node.js 中，流有四种类型——**可读**，**可写**，**双工**和**转换**。理解它们的作用非常简单。

### 流的类型

数据从**可读流**中消耗。它们抽象了源的加载和分块。数据以一次一个数据块的方式呈现给可读流进行消耗（使用）。在数据块被消耗后，它被流释放，并呈现下一个数据块。可读流不能由消费者推送数据进入其中。可读流的一个例子是 HTTP 请求体。

**可读流**有两种模式——**流动**和**暂停**。这些模式决定了流的数据流动。当流处于流动模式时，数据会自动从底层流系统中读取，并提供给消费者。当流处于暂停模式时，数据不会自动从底层系统中读取。消费者必须使用`stream.read()`函数显式请求流中的数据。所有可读流都以暂停模式开始，并可以通过附加`data`事件处理程序、调用`stream.resume()`或`调用 stream.pipe()`来切换到流动模式。事件处理程序和流管道将在本节后面介绍。可读流可以使用`stream.pause()`方法或`stream.unpipe()`方法从流动切换到暂停。

**可写流**是可以写入或推送数据的流。**可写流**将源的组合和处理抽象化。数据被呈现给流以供提供者消耗。流将一次消耗一个数据块，直到被告知停止。在流消耗了一个数据块并适当处理后，它将消耗或请求下一个可用的数据块。一个可写流的例子是文件系统模块的`createWriteStream`函数，它允许我们将数据流到磁盘上的文件中。

**双工流**是既可读又可写的流。数据可以由提供者以块的形式推送到流中，也可以由消费者以块的形式从流中消耗。双工流的一个例子是网络套接字，比如 TCP 套接字。

**转换流**是允许数据块在流中移动时进行变异的双工流。一个转换流的例子是 Node.js 的`ZLib`模块中的`gzip`方法，它使用`gzip`压缩方法压缩数据。

流以块的形式加载数据，而不是一次性加载，因此为了有效地使用流，我们需要一种方法来确定流是否已加载数据。在 Node.js 中，流是`EventEmitter`原型的实例。当关键事件发生时，流会发出事件，比如错误或数据可用性。事件监听器可以使用`.on()`和`.once()`方法附加到流上。可读流和可写流都有用于数据处理、错误处理和流管理的事件。

以下表格显示了可用的事件及其目的：

### 可写流事件：

![图 6.4：可写流事件](img/Figure_6.4.jpg)

###### 图 6.4：可写流事件

### 可读流事件：

![图 6.5：可读流事件](img/Figure_6.5.jpg)

###### 图 6.5：可读流事件

#### 注意

这些事件监听器可以附加到流上，以处理数据流和管理流的状态。完整的文档可以在 Node.js 网站的流 API 下找到。

现在你了解了流的基础知识，我们必须实现它们。可读流遵循一个基本的工作流程。通常会调用一个返回可读流的方法。一个例子是文件系统 API 的`createReadStream()`函数，它创建一个从磁盘上流出文件的可读流。在返回可读流之后，我们可以通过附加`data`事件处理程序来开始从流中拉取数据。以下片段展示了一个例子：

```php
const fs = require( 'fs' );
fs.createReadStream( './path/to/files.ext' ).on( 'data', data => { 
  console.log( data );  
} );
```

###### 片段 6.10：使用可读流

在上面的例子中，我们导入了`fs`模块并调用了`createReadStream`函数。这个函数返回一个可读流。然后我们给`data`事件附加了一个事件监听器。这将把流放入流动模式，每当数据块准备就绪时，提供的回调函数将被调用。在这个例子中，我们的回调函数简单地记录了可读流放弃的数据。

就像可读流一样，可写流也遵循一个相当标准的工作流程。可写流的最基本工作流程是首先调用一个返回可写流的方法。一个例子是`fs`模块的`createWriteStream`函数。创建了可写流之后，我们可以使用`stream.write()`函数向其写入数据。这个函数将传入的数据写入流中。以下片段展示了一个例子：

```php
const fs = require( 'fs' );
const writeable = fs.createWriteStream( './path/to/files.ext' );
writeable.write( 'some data' );
writeable.write( 'more data!' );
```

###### 片段 6.11：使用可写流

在上面的片段中，我们加载了`fs`模块并调用了`createWriteStream`函数。这返回一个将数据写入文件系统的可写流。然后我们多次调用`stream.write()`函数。每次调用`write`函数时，我们传入的数据都被推送到可写流并写入磁盘。

Node.js 中最强大的功能之一是流的管道功能。管道流简单地将源流“管道”到目标流。您将一个流的数据输出管道到另一个流的输入。这非常强大，因为它允许我们简化连接流的过程。

考虑一个问题，我们必须从磁盘加载文件并将其作为 HTTP 响应发送给客户端。我们可以用两种方式来做这件事。我们可以构建的第一种实现是将整个文件加载到内存中，然后一次性将其推送给客户端。这对我们的服务器来说非常低效。第二种方法是利用流。我们从磁盘流式传输文件，并将数据块推送到请求流中。要做到这一点，我们必须在读取流上附加监听器，并捕获每个数据块，然后将数据块推送到 HTTP 响应。此伪代码如下所示：

```php
const fileSystemStream = load( 'filePath' );
fileSystemStream.on( 'data', data => HTTP_Response.push( data ) );
fileSystemStream.on( 'end', HTTP_Response.end() );
```

###### 片段 6.12：使用流将数据发送到 HTTP 响应

在前面片段的伪代码中，我们创建了一个从指定文件路径加载的流。然后为`data`事件和`end`事件添加了事件处理程序。每当数据事件有数据时，我们将该数据推送到`HTTP_Response`流。一旦没有更多数据并且触发了 end 事件，我们关闭`HTTP_Response`流，数据被发送到客户端。这需要几行代码，并要求开发人员管理数据和数据流。我们可以使用单行代码构建完全相同的功能，使用流管道。

使用`Stream.pipe()`函数进行流的管道传输。管道是在源流上调用的，并将目标流作为参数传递（例如，`readableStream.pipe( writeableStream )`）。管道返回目标流，允许它用于链接管道命令。使用与前面示例相同的场景，我们可以使用管道命令将伪代码简化为一行。如下所示：

```php
load( 'filePath' ).pipe( HTTP_Response );
```

###### 片段 6.13：管道数据伪代码

在前面的片段中，我们加载了文件数据并将其传输到`HTTP_response`。可读流加载的每个数据块都会自动传递给可写流`HTTP_Response`。当可读流完成加载数据时，它会自动关闭并告诉写流也关闭。

### 文件系统操作

Node 的文件系统模块，名为'`fs`'，提供了一个 API，我们可以与文件系统交互。文件系统 API 是围绕 POSIX 标准建模的。**POSIX（可移植操作系统接口）**标准是由 IEEE 计算机学会指定的标准，旨在帮助不同操作系统文件系统之间保持一般兼容性。您不需要学习标准的细节，但要了解 fs 模块遵循它以保持跨平台兼容性。要导入文件系统模块，我们可以使用以下命令：`const fs = require( 'fs' );`。

Node.js 中的大多数文件系统函数都要求您指定要使用的文件路径。在为 fs 模块指定文件路径时，路径可以以三种方式之一指定：作为**字符串**，作为**缓冲区**，或者使用`file:`协议的**URL**对象。当路径是字符串时，文件系统模块将尝试解析字符串以获得有效的文件路径。如果文件路径是缓冲区，文件系统模块将尝试解析缓冲区的内容以获得有效的文件路径。如果路径是 URL 对象，则文件系统将将对象转换为有效的 URL 字符串，然后尝试解析字符串以获得有效的文件路径。三种显示文件路径的示例如下所示：

```php
fs.existsSync( '/some/path/to/file.txt' );
fs.existsSync( Buffer.from( '/some/path/to/file.txt' ) );
fs.existsSync( new URL( 'file://some/path/to/file.txt' ) );
```

###### 片段 6.14：文件系统路径格式

正如您在前面的示例中看到的，我们使用了`fs`模块的`existsSync`函数。在第一行中，我们将文件路径作为字符串传递。在第二行中，我们从文件路径字符串创建了一个缓冲区，并将缓冲区传递给`existsSync`函数。在最后一个示例中，我们从文件路径的`file:`协议 URL 创建了一个 URL 对象，并将 URL 对象传递给`existsSync`函数。

文件路径可以解析为**相对路径**或**绝对路径**。绝对路径是从操作系统的根文件夹解析的。相对路径是从当前工作目录解析的。当前工作目录可以通过`process.cwd()`函数获得。通过字符串或缓冲区指定的路径可以是相对的或绝对的。使用 URL 对象指定的路径必须是对象的绝对路径。

文件系统模块引入了许多函数，允许我们与硬盘交互。对于这些函数的大部分，都有同步和异步实现。同步的 fs 函数是阻塞的！当您编写使用 fs 模块的任何代码时，记住这一点非常重要。

#### 注意

还记得*第二章，异步 JavaScript*中对阻塞操作的定义吗？阻塞操作将阻止事件循环处理任何事件。

如果您使用同步的`fs`函数加载大文件，它将阻塞事件循环。在同步的`fs`函数完成工作之前，不会处理任何事件。Node.js 线程不会执行任何其他操作，包括响应 HTTP 请求，处理事件或任何其他异步工作。您几乎总是应该使用`fs`函数的异步版本。唯一需要使用同步版本的情况是在必须在任何其他操作之前执行文件系统操作时。这可能是加载整个系统或服务器依赖的文件的一个例子。

### Express 服务器

我们在本主题的早期部分讨论了基本的 Node.js HTTP 服务器。我们创建的服务器非常基本，缺乏我们从真正的 Web 服务器中期望的许多功能。在 Node.js 中，用于创建最小和灵活的 Web 服务器的最常见模块之一是**Express**。Express 将 Node.js 服务器对象包装在一个简化功能的 API 中。Express 可以通过 NPM（`npm install express --save`）安装。

#### 注意

Express 的完整文档可以在[`expressjs.com`](https://expressjs.com)找到。

基本的 Express 服务器非常容易创建。让我们回顾一下本章前面创建的基本 Node.js HTTP 服务器。在基本的 HTTP 服务器示例中，我们首先使用`HTTP.createServer()`函数创建了一个服务器，并传递了一个基本的请求处理程序。然后使用`server.listen()`函数启动了服务器。Express 服务器的创建方式类似。就像 HTTP 服务器一样，我们首先需要引入我们的模块。为`Express`模块添加一个`require`语句，并创建变量来保存我们的主机名和端口号。接下来，我们必须创建我们的 Express 服务器。这只需调用默认从`require('express')`语句导入的函数。调用导入的函数并将结果保存在一个变量中。如下面的片段所示：

#### 注意

简单 HTTP 服务器的代码可以在练习 33 的代码下找到。

```php
const express = require( 'express' );
const hostname = '127.0.0.1';
const port = 8000;
const app = express();
```

###### 片段 6.15：设置 Express 服务器

在前面的片段中，我们导入了`Express`模块并将其保存到变量`Express`中。然后创建了两个常量变量——一个用于保存主机名，一个用于保存端口号。在代码的最后一行，我们调用了通过 require 语句导入的函数。这将创建一个带有所有默认参数的基本`Express`服务器。

我们必须做的下一步是复制我们的基本 HTTP 服务器，添加一个基本的 HTTP 请求处理程序。这可以通过`app.get()`函数完成。`App.get`为其提供的路径设置一个 HTTP GET 请求处理程序。它接受两个参数——路径和回调。路径指定处理程序将捕获请求的 URL 路径。`callback`是处理 HTTP 请求时调用的函数。我们应该为服务器的根路径（'`/`'）添加一个路由处理程序。如下片段所示：

```php
app.get( '/', ( req, res ) => res.end( 'Working express server!' ) )
```

###### 片段 6.16：设置路由处理程序

在前面的代码片段中，我们使用`app.get()`添加了一个路由处理程序。我们传入根路径（'`/`'），这样当基本路径（'`localhost/`'）被 HTTP 请求命中时，指定的回调将被调用。在我们的回调中，我们传入一个具有两个参数的函数：`req`和`res`。就像简单的 HTTP 服务器一样，`req`代表传入的 HTTP 请求，`res`代表传出的 HTTP 响应。在函数的主体中，我们使用字符串`Working express server!`关闭 HTTP 响应。这告诉`Express`使用基本的 200 HTTP 响应代码，并将文本作为响应的主体发送。

最后一步，我们必须采取的步骤来使我们的基本`Express`服务器工作是让它监听 HTTP 请求。为此，我们可以使用`app.listen()`函数。此函数告诉服务器开始在指定端口监听 HTTP 请求。我们将三个参数传递给`app.listen()`。第一个参数是**端口号**。第二个参数是**主机名**。第三个参数是一个**回调函数**，一旦服务器开始监听，就会被调用。使用正确的端口、主机名和一个打印我们可以访问服务器的 URL 的回调来调用`listen`函数。以下是一个示例：

```php
app.listen( port, hostname, () => console.log( 'Server running at http://${hostname}:${port}/' ) );
```

###### 片段 6.17：使 Express 服务器监听传入请求

在前面的片段中，我们调用了`listen`函数。我们传入端口号，解析为`8000`；主机名，解析为`127.0.0.1`；和一个`callback`函数，记录服务器 URL。一旦服务器开始在本地网络上的端口`8000`监听 HTTP 请求，就会调用`callback`函数。转到控制台上记录的 URL，看看你的基本服务器是如何工作的！

### 练习 34：创建一个基本的 Express 服务器

要构建一个基本的 Express 服务器，请执行以下步骤：

1.  导入`express`模块。

1.  设置主机名和端口的变量，并分别给它们赋值`127.0.0.1`和`8000`。

1.  通过调用`express()`创建服务器应用程序，并将其保存到`app`变量中。

1.  在基本路由`/`上添加一个 get 请求处理程序。

1.  提供一个接受`req`和`res`的`callback`函数，并使用文本`Working express server!`关闭响应。

1.  使服务器在指定的端口和主机上侦听`app.listen()`。

1.  提供一个回调函数给`app.listen()`，记录`Server running at ${server uri}`。

1.  启动服务器并在浏览器中加载指定的 URL。

**代码**

##### index.js

```php
const express = require( 'express' );
const hostname = '127.0.0.1';
const port = 8000;
const app = express();
app.get( '/', ( req, res ) => res.end( 'Working express server!' ) );
app.listen( port, hostname, () => console.log( 'Server running at http://${hostname}:${port}/' ) );
```

###### 片段 6.18：简单的 Express 服务器

https://bit.ly/2Qz4Z93

**结果**

![图 6.6：返回新的购物车数组](img/Figure_6.6.jpg)

###### 图 6.6：返回新的购物车数组

![图 6.7：返回新的购物车数组](img/Figure_6.7.jpg)

###### 图 6.7：返回新的购物车数组

您已成功构建了一个基本的 Express 服务器。

### 路由

Express 最强大的功能之一是其灵活的路由。**路由**指的是 Web 服务器的端点 URI 如何响应客户端请求。当客户端向 Web 服务器发出请求时，它请求指定的端点（URI 或路径）以及指定的 HTTP 方法（`GET`，`POST`等）。Web 服务器必须明确处理它将接受的路径和方法，以及说明如何处理请求的回调函数。在 Express 中，可以使用以下代码行来实现：`app.METHOD( PATH, HANDLER );`。`app`变量是 Express 服务器的实例。Method 是要为其设置处理程序的 HTTP 方法。方法应为小写。路径是服务器上的 URI 路径，处理程序将对其进行响应。处理程序是如果路径和方法匹配请求将执行的回调函数。以下是此功能的示例：

```php
app.get( '/', ( req, res ) => res.end('GET request at /') );
app.post( '/user', ( req, res ) => res.end( 'POST request at /user') );
app.delete( '/cart/item', ( req, res ) => res.end('DELETE request at /cart/item') );
```

###### 片段 6.19：Express 路由示例

在上述片段中，我们为 Express 服务器设置了三个路由处理程序。第一个是使用`.get()`函数设置的。这意味着服务器将寻找对指定路由的`GET`请求。我们传入了服务器的基本路由（`/`）。当基本路由收到`GET`请求时，将调用提供的回调函数。在我们的回调函数中，我们用字符串`GET request at /`进行响应。在第二行代码中，我们设置服务器响应路径`/user`的`POST`请求。当`POST`请求到达 Express 服务器时，我们调用提供的回调函数，关闭响应并返回字符串`POST request at /user.`在最后一行代码中，我们为`DELETE`请求设置了处理程序。当`DELETE`请求进入 URI`/cart/item`时，我们用提供的回调进行响应。

Express 还支持特殊函数`app.all()`。如果您经常使用 HTTP 请求，您会意识到`ALL`不是有效的 HTTP 方法。`app.all()`是一个特殊的处理程序函数，告诉 Express 响应指定 URI 的所有有效 HTTP 请求方法，并使用指定的回调。它被添加到 Express 中，以帮助减少重复的代码，如果一个路由打算接受任何请求方法。

Express 支持为请求 URI 和 HTTP 方法设置多个回调函数。为了实现这一点，我们必须向回调函数添加第三个参数：`next`。`next`是一个函数，当调用`next`时，Express 将移动到匹配方法和 URI 的下一个回调处理程序。以下是一个示例：

```php
app.get( '/', ( req, res, next ) => next() );
app.get( '/', ( req, res ) => res.end( 'Second handler!' ) );
```

###### 片段 6.20：相同路由的多个请求处理程序

在上述片段中，我们为基本 URI 设置了两个不同的路由处理程序和`GET`请求。当捕获到对基本路由的`GET`请求时，将调用第一个处理程序。此处理程序仅调用`next()`函数，告诉 Express 寻找下一个匹配的处理程序。Express 看到有第二个匹配的处理程序，并调用第二个处理程序函数，关闭 HTTP 响应。重要的是要注意，HTTP 响应只能关闭并一次返回给客户端。如果为 URI 服务器和 HTTP 方法设置了多个处理程序，必须确保只有一个处理程序关闭 HTTP 请求，否则将会出现错误。多个处理程序提供的功能对于 Express 中的中间件和错误处理非常重要。这些应用程序将在本节后面更详细地讨论。

### 高级路由

如前所述，在 Express 中，路由路径是它匹配的路径 URI，以及 HTTP 方法，在检查要调用哪个处理程序回调时。路由路径作为第一个参数传递给函数，例如`app.get()`。Express 的强大之处在于能够创建极其动态的路由路径，以匹配多个 URI。在 Express 中，路由路径可以是字符串、字符串模式或正则表达式。Express 将解析基于字符串的路由，以查找特殊字符`?`，`+`，`*`，`()`，`$`，`[`和`]`。在字符串路径中使用时，特殊字符`?`，`+`，`*`和`()`是正则表达式对应字符的子集。`[`和`]`字符用于转义 URL 的部分，在字符串中不会被字面解释。`$`字符是 Express 路径解析模块中的保留字符。如果必须在路径字符串中使用`$`字符，则必须使用`[`和`]`进行转义。例如，`/user/$22515`应该在 Express 路由处理程序中写成`/data/[\$]22515`。

`*`字符的功能类似于`+`字符，但匹配零个或多个字符的重复。Express 将匹配与字符串完全匹配但不包含额外字符的路由。一个或多个连续字符可以用来代替星号。示例如下：

```php
app.get( '/abc?de', ( req, res ) => {
  console.log( 'Matched /abde or /abcde' );
} );
```

###### 片段 6.23：使用零个或多个字符进行路由

在前面的片段中，我们为 URL 路径`/abc?de`设置了一个`GET`处理程序。当命中此 URL 时，将调用回调函数，该函数记录两个可能的 URI 匹配选项。由于`?`字符跟在`c`字符后面，因此`c`字符被视为可选的。Express 将匹配包含或不包含可选字符的 URI 的`GET`请求。对`/abde`和`/abcde`的请求都将匹配。

`+`符号用于指示字符或字符组的零个或多个重复。Express 将匹配与重复字符的字符串完全匹配的路由，以及包含一个或多个标记字符的连续重复的任何字符串。示例如下：

```php
app.get( '/fo+d', ( req, res ) => {
  console.log( 'Matched /fd, /fod, /food, /fooooooooood' );
} );
```

###### 片段 6.22：路由路径中零个或多个重复字符

在前面的片段中，我们为 URL 路径`fo+d`设置了一个`GET`处理程序。当命中此 URI 时，将调用回调函数，该函数记录一些匹配选项。由于 o 字符后面跟着`+`字符，Express 将解析任何包含零个或多个`o`的路由。Express 将匹配`fd`，`fod`，`food`，`foooooooooooood`和任何其他具有连续`o`的字符串 URI。

片段 6.21：路由路径中的可选字符

```php
app.get( '/fo*d', ( req, res ) => {
  console.log( 'Matched /fd, /fod, /fad, /faeioud' );
} );
```

###### 如果我们希望在路由中使用特殊字符以增加灵活性，我们可以使用字符`?`，`+`，`*`和`()`。这些字符的操作方式与它们的正则表达式对应字符相同。这意味着`?`字符用于表示可选字符。跟在`?`符号后面的任何字符或字符组都将被视为可选的，Express 将匹配要么与可选字符的完整字符串匹配，要么与不包含可选字符的完整字符串匹配。示例如下：

在前面的片段中，我们为 URL 路径`fo*d`设置了一个`GET`处理程序。当命中此 URI 时，将调用回调函数，该函数记录一些匹配选项。由于`o`字符后面跟着`*`字符，Express 将解析任何包含零个或多个额外字符的路由。Express 将匹配`fd`，`fod`，`fad`，`foood`，`faeioud`和任何其他具有连续字符的字符串 URI。在比较`+`和`*`字符的匹配字符串时，请注意匹配字符串之间的差异。`*`字符将匹配`+`字符匹配的所有字符串，还会匹配任何有效字符代替星号的字符串。

最后一组字符是`()`。括号将一组字符分组在一起。当与其他特殊字符（`?`，`+`或`*`）一起使用时，分组字符将被视为单个单位。例如，URI`/ab(cd)?ef`将匹配 URI`/abef`和`/abcdef`。字符`cd`被分组在一起，并且整个组受到`?`字符的影响。示例显示了这种与每个特殊字符的交互在以下片段中：

```php
app.get( '/b(es)?t', ( req, res ) => {
  console.log( 'Matched /bt and /best' );
} );
app.get( '/b(es)+t', ( req, res ) => {
  console.log( 'Matched /bt, /best, /besest, /besesest' );
} );
app.get( '/b(es)*t', ( req, res ) => {
  console.log( 'Matched /bt, /best, /besest, /besesest' );
} );
```

###### 片段 6.24：使用字符组进行路由

在前面的片段中，我们为路径`b(es)?t`，`b(es)+t`，`b(es)*t`设置了`GET`请求处理程序。每个处理程序调用一个回调函数，记录一些匹配选项。在所有处理程序中，字符`es`被分组在一起。在第一个处理程序中，分组字符受到`?`字符的影响，并被视为可选的。处理程序将匹配包含完整字符串并且只包含非可选字符的 URI。两个选项是`bt`和`best`。在第二个处理程序中，字符组受到`+`字符的影响。具有零个或多个连续重复字符组的 URI 将匹配。匹配选项是`bt`，`best`，`besest`，`besesest`，以及任何其他具有更多连续重复的字符串。

Express 还允许我们在路由字符串中设置路由参数。路由参数是命名路由部分，允许我们指定要捕获并保存到变量中的路由 URL 的部分。URL 的捕获部分保存在`req.params`对象中，对象的键名与捕获的名称匹配。URL 参数使用`:`字符指定，后跟捕获名称。任何落在路由的那部分字符串都将被捕获并保存。示例显示在以下片段中：

```php
app.get( '/amazon/audible/:userId/books/:bookId', ( req, res ) => {
  console.log( req.params );
} );
```

###### 片段 6.25：使用 URL 参数进行路由

在前面的片段中，我们为路由`/amazon/audible/:userId/books/:bookId`设置了一个 get 参数。这个路由有两个命名参数捕获：一个是`userId`，另一个是`bookId`。这两个命名捕获可以包含任何一组有效的 URL 字符。在`audible/`和`/books`之间包含的任何字符都将保存在`req.params.userId`变量中，而在`books/`之后的任何字符都将保存在`req.params.bookId`中。重要的是要注意，`/`字符是用来分割 URL 的。保存的捕获组将不包含`/`字符，因为 Express 将其解析为 URL 分隔符。

Express 路由还可以在路径字符串的位置使用正则表达式。如果将正则表达式传递给请求处理程序的第一个参数而不是字符串，Express 将解析正则表达式，并且与正则表达式匹配的任何字符串都将触发提供的回调处理程序。如果您对正则表达式不熟悉，可以在网上找到许多教授基础知识的教程。正则表达式路径的示例显示在以下片段中：

```php
app.get( /^web.*/, ( req, res ) => {
  console.log( 'Matched strings like web, website, and webmail' );
} );
```

###### 片段 6.26：使用正则表达式进行路由

在前面的片段中，我们为正则表达式路由`/^web.*/`设置了一个`GET`处理程序。如果匹配此处理程序，服务器将记录两个匹配的字符串示例。我们提供给`GET`处理程序的正则表达式指定了 URI 必须以字符串`web`开头，可以跟随任意数量的字符。这将匹配诸如`/web`，`/website`和`/webmail`等 URI。

### 中间件

Express 还通过一个名为中间件的功能扩展了服务器的灵活性。Express 是一个路由和中间件框架，本身功能有限。**中间件**是具有对 HTTP 请求请求和响应对象的访问权限并在处理序列的中间某处运行的函数。中间件可以执行四项任务中的一项：执行代码，对请求和响应对象进行更改，结束 HTTP 请求-响应序列，并调用适用于请求的下一个中间件。

#### 注意

中间件函数可以手动编写，也可以通过第三方 NPM 模块下载。在编写中间件之前，请检查中间件是否已经存在。官方中间件模块和一些热门模块可以在[`expressjs.com/en/resources/middleware.html`](https://expressjs.com/en/resources/middleware.html)找到。

中间件函数有三个输入变量：`req`，`res`和`next`。`Req`表示请求对象，`res`表示响应对象，`next`是一个告诉 Express 继续到下一个中间件处理程序的函数。我们在本节的前面看到了`next`函数，当将多个路由处理程序注册到一个 URI 时。`next`函数告诉中间件处理程序将控制权传递给处理程序堆栈中的下一个中间件。简单来说，它告诉`next`中间件运行。如果中间件没有结束请求-响应序列，它必须调用`next`函数。如果不这样做，请求将挂起并最终超时。中间件可以使用`app.use()`和`app.METHOD()`函数附加。使用`app.use()`设置的中间件将对匹配指定可选路径的所有 HTTP 方法触发。使用 HTTP 方法函数附加的中间件将对匹配方法和指定路径的所有请求触发。下面的片段显示了中间件的示例：

```php
app.use( ( req, res, next ) => {
  req.currentTime = new Date();
  next();
} );
app.get( '/', ( req, res ) => {
  console.log( req.currentTime );
} );
```

###### 片段 6.27：设置中间件

在前面的片段中，我们使用`app.use()`设置了一个中间件函数。我们没有为`app.use()`提供路径，因此所有请求都将触发中间件。中间件通过在请求中设置`currentTime`字段为一个新的日期对象来更新请求对象。然后中间件调用下一个函数，该函数将控制权传递给下一个中间件或路由处理程序。假设请求到基本 URI，下一个被触发的处理程序是注册的处理程序，它打印`req.currentTime`中保存的值。

### 错误处理

Express 的最后一个重要方面是错误处理。**错误处理**是 Express 处理和管理错误的过程。Express 可以处理同步错误和异步错误。Express 具有内置的错误处理，因此您不需要编写自己的错误处理程序。Express 的内置错误处理程序将在响应正文中将错误返回给客户端。这可能包括错误详细信息，如堆栈跟踪。如果您希望用户看到自定义错误消息或页面，则必须附加自己的错误处理程序。

内置的 Express 错误处理程序将捕获路由处理程序或中间件函数中同步代码中抛出的任何错误。这包括运行时错误和使用 throw 关键字抛出的错误。但是，Express 不会捕获在异步函数中抛出的错误。要在异步函数中调用错误，必须将`next`函数添加到回调处理程序中。如果发生错误，必须使用要处理的错误调用 next。下面的片段显示了使用默认错误处理程序进行同步和异步错误处理的示例：

```php
app.get( '/synchronousError', ( req, res ) => {
  throw new Error( 'Synchronous error' );
} );
app.get( '/asynchronousError', ( req, res, next ) => {
  setTimeout( () => next( new Error( 'Asynchronous error' ) ), 0 );
} );
```

###### 片段 6.28：同步和异步错误处理

在前面的片段中，我们首先创建了一个`GET`请求处理程序，路径为`/synchronousError`。如果触发了这个处理程序，我们调用回调函数，在同步代码块中抛出一个错误。由于错误是在同步代码块中抛出的，Express 会自动捕获错误并将其传递给客户端。在第二个示例中，我们为路径`/asynchronousError`创建了一个`GET`请求处理程序。当触发了这个处理程序时，我们调用一个回调函数，开始一个超时，并使用错误调用`next`函数。错误发生在一个异步代码块中，因此必须通过 next 函数传递给 Express。当 Express 捕获到错误时，无论是通过抛出事件同步地还是通过 next 函数异步地，它会立即跳过所有适用的中间件和路由处理程序，并跳转到第一个适用的错误处理程序。

定义我们自己的错误处理中间件函数，我们以与其他中间件函数相同的方式添加它，只是有一个关键的区别。错误处理中间件回调函数在回调中有四个参数，而不是三个。参数依次是`err`、`req`、`res`和`next`。它们的解释如下：

+   `err`代表正在处理的错误。

+   `req`代表请求对象。

+   `res`代表响应对象。

+   `next`是一个告诉 Express 继续下一个错误处理程序的函数。

自定义错误处理程序应该是最后定义的中间件。下面是自定义错误处理的示例：

```php
app.get( '/', ( req, res ) => {
  throw new Error( 'OH NO AN ERROR!' );
} );
app.use( ( err, req, res, next ) => {
  req.json( 'Got an error!' + err.message );
} );
```

###### 片段 6.29：添加自定义错误处理程序

在前面的片段中，我们为基本路由添加了一个`GET`请求处理程序。当处理程序被触发时，它调用一个回调函数，该函数抛出一个错误。这个错误会被 Express 自动捕获并传递给下一个错误处理程序。下一个错误处理程序是我们用`app.use()`函数定义的。这个错误处理程序捕获错误，然后用错误消息响应客户端。

### 练习 35：使用 Node.js 构建后端

你被要求为一个笔记应用构建一个 Node.js Express 服务器。服务器应该为基本路由（`/`）提供一个基本的 HTML 页面（在活动文件夹下的`index.html`中提供）。服务器将需要一个 API 路由，从服务器的本地文件系统中的文本文件加载保存的笔记，并且一个 API 路由，将对笔记的更改保存到服务器的本地文件系统中的文本文件。服务器应该在加载笔记时接受一个`GET`请求到 URI`/load`，在保存笔记时接受一个`POST`请求到 URI`/save`。提供的 HTML 文件将假定这些是服务器上使用的 API 路径。在构建服务器时，您可能希望使用`body-parser`中间件，并将 strict 选项设置为 false，以简化请求的处理。

要构建一个工作的 Node.js 服务器，提供一个 HTML 文件并接受 API 调用，执行以下步骤：

1.  使用`npm init`设置项目。

1.  使用 npm 安装`express`和`body-parser`。

1.  导入模块`express`、`http`和`body-parser`保存为`bodyParser`，以及`fs`，并将它们保存在变量中。

1.  创建一个名为`notePath`的变量，其中包含文本文件的路径（`./note.txt`）。

1.  记录我们正在创建一个服务器。

1.  使用`express()`创建服务器应用，并将其保存在`app`变量中。

1.  使用`http.createServer(app)`从 Express 应用创建一个 HTTP 服务器，并将其保存在 server 变量中。

1.  记录我们正在配置服务器。

1.  使用`body-parser`中间件来解析 JSON 请求体。

告诉 Express 应用使用中间件`app.use()`。

将`bodyParser.json()`传递给 use 函数。

将一个选项对象传递给`bodyParser.json()`，使用`key/value`对。strict:false。

1.  使用`express.Router()`创建一个路由来处理路由，并将其保存在变量 router 中。

1.  为基本路由添加一个`GET`路由处理程序，使用`router.route('/').get`。

添加一个接受`req`和`res`的回调函数。

在回调中，使用`res.sendFile()`发送`index.html`文件。

将`index.html`作为第一个参数传递，并将选项对象`{`root: __dirname`}`作为第二个参数传递。

1.  为`/save`路由添加一个路由，接受`POST`请求`router.route( '/save' ).post`。

路由处理程序回调应该接受参数`req`和`res`。

在回调中，使用`fs`函数`writeFile()`和`notePath`以及`req.body`参数和回调函数。

在回调函数中，接受参数`err`和`data`。

如果提供了`err`，则使用状态码`500`和 JSON 形式的错误关闭响应。

如果没有提供错误，则使用数据对象的 JSON 关闭响应，并使用`200`状态码。

1.  为`/load`路由添加一个路由，接受`GET`请求`router.route( '/load ).get`。

路由处理程序回调应该接受参数`req`和`res`。

在回调中，使用`fs`函数`readFile`和`notePath`以及`utf8`参数和回调函数。

在回调函数中，接受参数`err`和`data`。

如果提供了`err`，则使用状态码`500`和 JSON 形式的错误关闭响应。

如果没有提供错误，则使用数据对象的 JSON 关闭响应，并使用`200`状态码。

1.  使`express`应用程序使用路由器处理基本路由的请求`app.use('/', router)`。

1.  设置服务器以侦听正确的端口和主机名，并使用`server.listen( port, hostname, callback )`传入回调。

回调函数应该接受一个错误参数。如果找到错误，抛出该错误。

否则，记录服务器正在侦听的端口。

1.  启动服务器并加载运行在`(localhost:PORT)`的 URL。

1.  通过保存一个笔记，刷新网页，加载保存的笔记（应该与之前保存的内容匹配），然后更新笔记来测试服务器的路由和功能。

**代码**

##### index.js

```php
router.route( '/' ).get( ( req, res ) => res.sendFile( 'index.html', { root: __dirname } ) );
router.route( '/save' ).post( ( req, res ) => {
 fs.writeFile( notePath, req.body, 'utf8', err => {
   if ( err ) {
     res.status( 500 );
   }
   res.end();
 } );
} );
router.route( '/load' ).get( ( req, res ) => {
 fs.readFile( notePath, 'utf8', ( err, data ) => {
   if ( err ) {
     res.status( 500 ).end();
   }
   res.json( data );
 } );
} );
```

###### 片段 6.30：复杂应用的 Express 服务器路由

https://bit.ly/2C4FR64

**结果**

![图 6.8：监听端口 8000](img/Figure_6.8.jpg)

###### 图 6.8：监听端口 8000

![图 6.9：加载测试笔记](img/Figure_6.9.jpg)

###### 图 6.9：加载测试笔记

您已成功构建了一个工作的 Node.js 服务器，可以提供 HTML 文件并接受 API 调用。

## React

**React**是一个用于构建用户界面的 JavaScript 库。React 主要由 FaceBook 维护。React 最初是由 Facebook 软件工程师 Jordal Walke 创建的，并于 2013 年开源。React 旨在简化 Web 开发，使开发人员能够轻松构建单页网站和移动应用程序。

#### 注意

React 的完整文档以及扩展教程可以在它们的主页找到：[`reactjs.org/`](https://reactjs.org/)。

React 使用声明性方法来设计视图，以提高页面的可预测性和调试性。开发人员可以为应用程序中的每个状态声明和设计简单的视图。React 将处理视图的更新和渲染，随着状态的改变。React 依赖于基于组件的模型。开发人员构建封装的组件，跟踪和处理它们自己的内部状态。我们可以组合我们的组件以创建复杂的用户界面，类似于我们如何使用函数组合从简单函数构建复杂函数。通过组件，我们可以通过应用程序在组件之间传递丰富的数据类型。这是允许的，因为组件逻辑纯粹是用 JavaScript 编写的。最后，React 允许我们在框架中非常灵活。不会对应用程序背后的技术栈做出任何假设。React 可以在浏览器中加载时编译，在 Node.js 服务器上编译，或者使用 React Native 编译成移动应用程序。这使得可以在不需要重构现有代码的情况下逐步将 React 纳入新功能。您可以在技术栈的任何点开始纳入 React。

### 安装 React

React 可以通过 NPM 安装，并在服务器上编译，或通过脚本标签集成到应用程序中。有几种安装 React 并将其添加到项目中的方法。

将 React 添加到应用程序的最快方法是通过`<script>`标签包含内置库。如果您有现有项目并希望逐步开始将 React 纳入其中，这种方法通常是最简单的。以这种方式添加 React 不到一分钟，可以让您立即开始添加组件。首先，我们需要在 HTML 页面中添加一个 DOM 容器，我们希望我们的 React 组件附加到其中。通常这是一个带有唯一 ID 的 div。然后使用脚本标签添加**React**和`ReactDOM`模块。添加了脚本标签后，可以使用脚本标签加载 React 组件。以下是一个示例。

```php
<div id="react-attach-point"></div>
<script src="https://unpkg.com/react@16/umd/react.development.js" crossorigin></script><script src="https://unpkg.com/react-dom@16/umd/react-dom.development.js" crossorigin></script>
<script src="react_components.js"></script>
```

###### 代码段 6.31：将 React 添加到网页

设置 React 应用程序并将 React 安装到新项目中的下一个最简单的方法是使用 React 应用程序创建者。这个模块是一个 Node.js 命令行界面，可以自动设置一个具有简单预定义文件夹结构和基本依赖项安装的 React 项目。可以使用命令行命令`npm install create-react-app -g`安装 CLI 工具。这个命令告诉 NPM 在全局范围内安装 CLI 模块，以便可以从命令行运行它。安装了 CLI 后，可以通过运行`create-react-app my-app-name`命令创建一个新的 React 项目。CLI 工具将在工作目录中创建一个文件夹，名称为提供的名称（例如命令中的`my-app-name`），安装 React 依赖项，并为应用程序资源创建两个文件夹。CLI 工具将使用示例应用程序填充源代码文件夹，命名为`src`。可以使用`npm start`命令启动应用程序。从这一点开始，可以开始修改文件，看看 React 是如何工作的，或者可以删除`src`中的所有文件，开始编写自己的应用程序。

安装 React 最困难的方法是逐个安装各个依赖项。这种方法提供了最大的灵活性，允许您将 React 集成到现有的工具链中。要安装 React，必须使用 NPM 安装`react`和`react-dom`模块。这两个模块可以在本地项目范围内安装，并应该使用`--save`或`-s`标志保存到`package.json`依赖项列表中。安装了模块后，可以使用现有的工具链创建和构建 React 组件。

在本主题中，我们将使用带有 JSX 的 React。**JSX**是 JavaScript 的一种语法糖，默认情况下不受浏览器支持。JSX 必须通过 Babel 转译为有效的 JavaScript 代码。要完成 React 的设置，您需要设置 Babel 来将您的 React 和 JSX 代码转译为 JavaScript。如果您的项目尚未安装 Babel，可以使用`npm install babel -s`命令进行安装。

这将保存 Babel 作为项目的依赖项。要将 React JSX 插件添加到 Babel 中，请运行`npm install babel-preset-react-app -s`命令。此命令添加了 Babel 的 JSX 转换库。设置好 Babel 后，我们必须创建一个构建脚本，可以运行以转换我们所有的代码。在 package.json 中，添加以下行：`build": "npx babel src -d lib --presets react-app/prod`。请注意，`npx`不是拼写错误。它是一个随 NPM 一起提供的包运行工具。这行告诉 Babel 将代码从`src`目录编译到`lib`目录，并使用`react-app/prod`预设。每次我们对 React 代码进行更改并希望在前端反映这些更改时，都应该运行此命令。现在，您已经准备好开始构建 React 应用程序了。

#### 注意

您可以提供上一段中所述的 Babel 设置命令，以演示如何为转译设置项目。

### React 基础

React 是围绕称为组件的小型封装代码构建的。在 React 中，组件是通过对`React.Component`或`React.PureComponent`进行子类化来定义的。最常见的方法是使用`React.Component`。在最简单的形式中，React 组件接受属性（通常称为`props`）并通过调用`render()`返回要显示的视图。在初始化组件时定义属性。每个创建的组件必须在子类中定义一个名为`render()`的方法。render 函数以 JSX 形式返回屏幕上将呈现的内容的描述。以下是一个示例组件声明：

```php
class HelloWorld extends React.Component {
  render() {
    return (
      <div>
        Hello World!!! Made by {this.props.by}!!!
      </div>
    );
  }
}
ReactDOM.render(
  <HelloWorld by="Zach"/>,
  document.getElementById('root')
);
```

###### 片段 6.32：基本的 React 元素

在前面的片段中，我们定义了一个名为`HelloWorld`的新的 React 组件`class`。这个新类扩展了基本的`React.Component`。在声明内部，我们定义了`render()`函数。`render()`函数返回一个 JSX 块，定义了将在屏幕上呈现的内容。在这个 JSX 块中，我们创建了一个带有文本`Hello World!!! Made by *!!!`的`div`，其中`*`字符被通过属性传递的值替换。在最后几行中，我们调用了`ReactDom.render()`函数。这告诉`ReactDom`模块呈现我们传递给`render()`函数的所有组件和视图。在前面的片段中，我们将我们的`HelloWorld`组件与属性`by`设置为`Zach`，并告诉渲染函数将呈现的 DOM 附加到根元素上。传递到属性中的数据被传递到我们的组件内部的`this.props`中，并填充到`Hello World!!! div`中。

#### 注意

如果您的代码库不使用 ES6 或 ES6 类，您可以使用 create-react-class 模块，但是，此模块的具体细节超出了本书的范围。

恭喜！您已经了解了 React 的最基本形式。通过扩展这个例子，您现在可以构建基本的静态网页。这可能看起来并不是很有用，但它是所有网页开发的最基本构建块。

### React 特定

在我们之前的片段中的基本示例中，我们可以看到 React 使用了一种奇怪的语法糖叫做 JSX。JSX 既不是 HTML 也不是 JavaScript。它是 JavaScript 的语法扩展，结合了 HTML 和 XML 的一些概念，帮助描述用户界面应该是什么样子。JSX 对于 React 应用并非必需，但建议在构建 React UI 时使用它。它看起来像一个模板语言，但具有 JavaScript 的全部功能。它可以通过 Babel React 插件编译成标准的 JavaScript。以下片段显示了 JSX 和等效的 JavaScript 示例：

```php
const elementJSX = <div>Hello, world!</div>;
const elementJS = React.createElement( "div", null, "Hello, world!" );
```

###### 片段 6.33：JSX vs JS

在前面的片段中，我们定义了一个名为`elementJSX`的变量，并将一个 JSX 元素保存到其中。在第二行，我们创建了一个名为`elementJS`的变量，并用纯 JavaScript 保存了等效的元素。在这个示例中，你可以清楚地看到 JSX 的标记风格如何简化了在 JavaScript 中定义元素的方法。

### JSX

**JSX**可以像标准 JavaScript 中的模板文字一样嵌入表达式。然而，主要区别在于 JSX 只使用花括号（`{}`）来定义表达式。与模板文字一样，在 JSX 中使用的表达式可以是变量、对象引用或函数调用。这使我们能够在 React 中使用 JSX 创建动态元素。以下片段显示了 JSX 表达式的示例：

```php
const name = "David";
function multiplyBy2( num ) { return num * 2; }
const element1 = <div>Hello {name}!</div>;
const element2 = <div>6 * 2 = {multiplyBy2(6)}</div>;
```

###### 片段 6.34：JSX 表达式

在前面的片段中，我们首先创建了一个名为 name 的变量，其中包含字符串`David`，以及一个名为`multiplyBy2`的函数，该函数接受一个数字并返回乘以`2`的数字。然后我们创建了一个名为`element1`的变量，并将一个 JSX 元素保存到其中。这个 JSX 元素包含一个包含对`name`变量的引用的表达式的`div`。当构建这个 JSX 元素时，表达式将`name`变量评估为字符串`David`并将其插入到最终的标记中。在代码的最后一行，我们创建了一个名为`element2`的变量，并将另一个 JSX 元素保存到其中。这个 JSX 元素包含一个包含对`multiplyBy2`函数的表达式的 div。当创建 JSX 元素时，表达式将评估其中的代码并调用函数。函数的返回值被放入最终的标记中。正如你所看到的，JSX 中的表达式与模板文字中的表达式非常相似。

### ReactDOM

当我们创建 React 元素时，我们必须有一种方法将它们渲染到 DOM 中。这在 React 介绍示例中被非常简要地提及了。在那个示例中，我们使用了`ReactDOM`库来渲染我们创建的组件。从`react-dom`模块导入的`ReactDOM`对象提供了可以在整个应用程序中使用的特定于 DOM 的方法；然而，大多数组件并不需要这些方法。你将最常使用的函数是`render()`函数。这个函数接受三个参数。

第一个参数是我们将要渲染或附加到 DOM 的 React 元素。第二个参数是 React 组件将被渲染到的容器或 DOM 节点。最后一个参数是一个可选的回调方法。回调函数将在组件渲染后执行。对于完整的 React 应用程序，`ReactDOM.render()`通常只需要在应用程序的顶层使用，并用于在视图中渲染整个应用程序。在将 React 逐渐纳入现有代码库的应用程序中，`ReactDOM.render()`可能在每个新的 React 组件被纳入非 React 代码的地方使用。以下片段显示了`ReactDOM.render()`的示例：

```php
import ReactDOM from 'react-dom';
const element = <div>HELLO WORLD!!!</div>;
ReactDOM.render( element, document.getElementById('root'), () => {
  console.log( 'Done rendering' );
});
```

###### 片段 6.35：将元素渲染到 DOM 中

在上面的示例中，我们首先导入了`ReactDOM`模块。然后我们使用 JSX 创建了一个新的 React 元素。这个简单的元素只包含一个带有文本`HELLO WORLD!!!`的`div`。然后我们调用了`ReactDOM.render()`函数并传入了所有三个参数。这个函数调用告诉浏览器选择根 DOM 节点并附加我们的 React 元素渲染的标记。当渲染完成时，将调用提供的回调，并将`Done rendering`字符串记录到控制台中。

### React.Component

React 围绕组件展开。正如我们之前学到的，创建新组件的最简单方法是创建一个扩展`React.Component`类的新子类。`React.Component`类可以通过从 React NPM 模块导入的 React 对象访问。当我们定义一个 React 组件时，我们必须至少定义一个`render()`函数。`render`函数返回组件将包含的 JSX 描述。如果我们希望创建更复杂的组件，例如具有状态的组件，我们可以向组件添加构造函数。构造函数必须接受`props`变量，并且必须使用`props`变量调用`super()`函数。`props`变量将包含在创建 React 组件时分配的属性的对象。以下是一个具有构造函数的 React 组件的示例：

```php
class ConstructorExample extends React.Component{
  constructor( props ){
    super( props );
    this.variable = 'test';
  }
  render() { return <div>Constructor Example</div>; }
}
```

###### 片段 6.36：React 类构造函数

在上面的片段中，我们创建了一个名为`ConstructorExample`的新组件。在同一个片段中，我们调用了`constructor`函数。`constructor`函数接受一个参数，即包含属性的对象。在构造函数中，我们调用了`super()`函数并传入`props`变量。然后我们创建了一个名为`variable`的`class`变量，并赋予了值`test`。在类的末尾，作为所有 React 组件所需的，我们添加了一个返回组件的 JSX 标记的`render()`函数。

### 状态

要向 React 组件添加本地状态，我们只需在构造函数内部（`this.state = {}`）初始化状态变量。状态变量是 React 中的一个特殊变量关键字名称。对`this.state`的任何更改都将导致调用`render()`函数。这使我们能够根据组件的当前状态动态更改视图。

关于状态变量，有三个关键要知道的事情。首先，您不应该直接修改状态，比如`this.state.value = 'value'`。以这种方式修改状态不会导致调用`render()`和视图更新。相反，您必须使用`setState()`函数。这将使用传递给函数的数据更新状态。例如，我们必须这样设置状态：`this.setState( { name: 'Zach' } )`。第二个关键细节是状态更新可能是异步的。React 可能会将多个`setState`调用合并为单个更新以提高性能。因此，我们不能依赖它们的值来计算状态。如果我们必须使用当前状态或属性的当前值来计算下一个状态，我们可以使用`setState`的第二种形式，它接受一个函数而不是一个对象。该函数将接收前一个状态作为第一个参数，并在应用状态更新时接收属性对象。这可靠地允许我们使用先前的状态和属性信息来计算下一个状态。最后，状态更新是合并而不是覆盖。与`Object.assign`函数类似，`setState`对状态对象和新状态进行浅合并。在设置状态时，新对象将与旧状态对象合并。只有新状态对象中指定的属性将更改。旧状态对象中的所有属性，如果不在新状态对象中，将保持不变。

在 React 组件中，属性对象从组件内部是只读的。这意味着从组件内部对属性对象的更改不会反映到父组件或 DOM 结构中的任何变量。数据只能向下流动。因此，对子组件属性的父组件 JSX 标记的任何更改都将导致子组件使用新的属性值重新渲染。要使数据向上流动，我们必须以属性的形式将函数从父组件传递到子组件。以下片段显示了一个示例。

```php
class ChildElement extends React.Component {
 render() {
   return (
     <button onClick={this.props.onClick}>
       Click me!
     </button>
   );
 }
}
class ParentElement extends React.Component {
 clicked() { console.log( 'clicked' ); }
 render() {
   return <ChildElement onClick={this.clicked.bind(this)}/>;
 }
}
```

###### 片段 6.37：渲染子组件

在这个片段中，我们创建了两个组件。第一个称为`ChildElement`，第二个称为`ParentElement`。`ChildElement`简单地包含了一个按钮的 JSX，当点击时，通过`onClick`属性传递的函数被调用。`ParentElement`包含一个名为`clicked`的函数，用于记录到控制台，并在渲染时返回带有`ChildElement`实例的 JSX。在`ParentElement`的 JSX 中创建的`ChildElement`将`onClick`属性设置为`ParentElement`的`clicked()`函数。当点击`ChildElement`中的按钮时，将调用`clicked()`函数。在这个例子中，当我们将它传递给子元素时，将父级范围绑定到`this.clicked`（`this.clicked.bind(this)`）。如果`this.clicked`需要访问父组件中的任何内容，我们必须将其范围绑定到父组件的范围。在您的 React 应用程序中，您可以使用此功能创建向上的数据流。

在 React 中处理 DOM 事件与 HTML DOM 元素事件处理非常相似，但有一些主要区别。首先，在 React 中，事件名称使用`驼峰命名法`而不是小写。这意味着在名称的每个“新单词”中，该单词的第一个字母是大写的。例如，在 React 中，DOM 事件`onclick`变成了`onClick`。其次，在 JSX 中，函数事件处理程序直接作为函数传递到处理程序定义中，而不是作为包含处理程序函数名称的字符串。以下代码显示了标准 HTML 和 React 之间的差异：

```php
<button onclick="doSomething()">HTML</button>
<button onClick={doSomething}>JSX and React</button>
```

###### 片段 6.38：JSX 与 HTML 事件

在前面的片段中，我们创建了两个按钮。第一个是 HTML 格式的，它附加了一个`onclick`侦听器，调用`doSomething`函数。第二个按钮是 JSX 格式的，也有一个`onclick`侦听器，调用`doSomething`函数。请注意侦听器定义的不同之处。JSX 事件名称是`camelcase`，HTML 事件名称是小写。在 JSX 中，我们通过表达式设置处理程序函数，该表达式求值为函数。在 HTML 中，我们将事件处理程序设置为调用函数的字符串。

#### 注意

我们在*第三章，DOM 操作和事件处理*中学到，直接在 DOM 中附加事件是一种不好的做法。JSX 不是 HTML，这种做法是可以接受的，因为 JSX 通过转义 JSX 中嵌入的任何值来防止注入攻击。

React 事件处理和标准 DOM 事件处理之间的另一个重要区别是，在 React 中，事件处理程序函数不能返回 false 以阻止默认行为。您必须在事件对象上明确调用`preventDefault()`函数。

在 React 中附加事件侦听器时，我们必须小心处理`this`作用域。在 JavaScript 中，类方法默认情况下不绑定到`this`作用域。如果函数被传递到其他地方并从其他地方调用，则`this`作用域可能无法正确设置。在将它们附加到侦听器或作为属性传递方法时，应确保正确绑定`this`作用域。

### 条件渲染

在 React 中，我们创建不同的组件来封装我们需要的视图或行为。我们需要一种方式，根据应用程序的状态，只渲染我们创建的一些组件。在 React 中，这称为**条件渲染**。在 React 中，条件渲染的工作方式与 JavaScript 条件语句相同。我们可以使用 JavaScript 的 if 或条件运算符来决定渲染哪些元素。这可以通过几种方式来实现。

在两种简单的方式中，一种是根据当前状态返回一个 React 元素（JSX）的函数，而另一种是在 JSX 中有一个条件语句，根据当前状态返回一个 React 元素。这些条件渲染形式的示例显示在以下片段中：

```php
class AccountControl extends React.Component {
  constructor( props ) {
    super( props );
    this.state = { account: this.props.account };
  }
  isLoggedIn() {
    if ( this.state.account ) { return <LogoutButton/>; }
    else { return <LoginButton/>; }
  }
  render() {
    return (
      <div>
        {this.isLoggedIn()}
        {this.state.account ? <LogoutButton/> : <LoginButton/>}
      </div>
    );
  }
}
```

###### 片段 6.39：条件渲染

在前面的片段中，我们创建了一个名为`AccountControl`的元素。在构造函数中，我们将本地状态设置为包含从属性变量传入的帐户信息的对象。渲染函数简单地返回一个带有两个表达式的`div`。这两个表达式都利用条件渲染来根据当前状态显示信息。第一个表达式调用`isLoggedIn`函数，该函数检查`this.state.account`并根据当前状态返回`LogoutButton`或`LoginButton`。第二个表达式使用条件运算符来内联检查`this.state.account`，并根据本地状态返回`LogoutButton`或`LoginButton`。

### 项目列表

在 React 中渲染项目列表非常简单。它基于 JSX 和表达式的概念。正如我们之前学到的，JSX 使用表达式来创建动态代码。如果表达式求值为组件数组，则所有组件将被呈现为如果它们被内联添加到 JSX 中。我们可以构建一个组件的集合或数组，将集合保存在一个变量中，并将变量包含在 JSX 表达式中。这种示例显示在以下片段中：

```php
class ListElement extends React.Component {
  render() {
    return (
      <ul> {this.props.items.map( i => <li>{i}</li> )} </ul>
    );
  }
}
ReactDOM.render(
  <ListElement items={[ 1, 4, 5, 5, 7, 9 ]}/>,
  document.getElementById( 'root' )
);
```

###### 片段 6.40：渲染列表

在前面的片段中，我们创建了一个名为`ListElement`的元素。这个元素简单地接受一个项目数组，并将数组映射到包含`<li>`标签中的数组项值的 JSX 元素数组中。然后将结果列表项数组返回到`<ul>`标签中。当 JSX 将其编译成 HTML 时，数组中的每个项目按顺序插入到`<ul>`元素中。

### HTML 表单

React 的最后一个关键概念是 HTML 表单。HTML 表单在 React 中的工作方式与其他 DOM 元素不同，因为 HTML 表单跟踪其自己的内部状态。如果我们只需要处理表单的默认行为，那么我们可以在 React 中直接使用它们，并且不会出现任何问题。然而，当我们希望让 JavaScript 处理表单提交并访问表单中的所有数据时，我们会遇到一个复杂的问题。这个问题是因为元素和 React 组件同时尝试跟踪表单的状态。

实现这一点的方法是使用受控组件。受控组件的目标是从表单元素中删除状态控制，并使 React 成为控制组件。这是通过为字段的值更改事件（`onChange`）添加一个 React 事件监听器，并让 React 将其内部`state`变量值设置为表单的值来实现的。然后，React 将字段的值设置为保存在`state`变量中的值。React 读取`input`字段中的任何更改，并强制`input`字段采用发生在 React 组件中存储的数据的任何更改。以下片段中显示了这一点的示例：

```php
class ControlledInput extends React.Component {
  constructor(props) {
    super(props);
    this.state = {value: ''};
  }
  handleChange(event) { this.setState({value: event.target.value}); }
  render() {
    return (
      <div>
        <input type="text" value={this.state.value} onChange={this.handleChange.bind(this)} />
        <div>Value: {this.state.value} </div>
      </div>
    );
  }
}
```

###### 片段 6.41：React 组件状态

在前面的片段中，我们创建了一个名为`ControlledInput`的组件。该组件有一个名为 value 的状态变量，用于存储文本输入的值。我们创建了一个名为`handleChange`的函数，简单地通过将值设置为从事件中读取的值来更新组件的状态。在渲染函数中，我们创建一个包含一个`input`字段和另一个`div`的 div。这个输入字段的值映射到`this.state.value`，并且有一个调用`handleChange`函数的事件监听器。第二个`div`简单地镜像了`this.state.value`的值。当我们在文本输入框内进行更改时，`onChange`监听器被调用，组件的`state.value`被设置为输入字段的当前值。每当`this.state.value`被更改时，这种变化都会反映到`input`字段上。组件的`this.state.value`的值是绝对的，`input`字段被强制镜像它。

### 活动 6：使用 React 构建前端

在*练习 32*中负责笔记应用的前端团队意外辞职了。您必须使用 React 构建此应用的前端。您的前端应该有两个视图，一个是主页视图，一个是编辑视图。为每个视图创建一个`React`组件。`主页`视图应该有一个按钮，可以切换到`编辑`视图。`编辑`视图应该有一个按钮，可以切换回`主页`视图，一个包含`笔记文本`的文本输入，一个调用 API 加载路由的**加载**按钮，以及一个调用 API 保存路由的**保存**按钮。已经为您提供了一个 Node.js 服务器。在`activities/activity6/activity/src/index.js`中编写您的 React 代码。当您准备测试您的代码时，在启动服务器之前运行构建脚本（在`package.json`中定义）。您可以参考*练习 35*中的`index.html`文件，了解如何调用 API 路由的提示。

要构建一个可工作的 React 前端并将其与 Node.js Express 服务器集成，执行以下步骤：

1.  打开`activity/activity6/activity`中的起始活动。运行`npm install`以安装所需的依赖项。

1.  在`src/index.js`文件中创建`Home`和`Editor`组件。

1.  `主页`视图应该显示应用程序名称，并有一个按钮，可以将应用状态更改为`编辑`视图。

1.  `编辑`视图应该有一个返回主页的按钮，可以将应用状态更改为`编辑`视图，一个由`编辑视图`状态控制的文本输入，一个向服务器请求保存的`笔记文本`的**加载**按钮，以及一个向服务器请求保存笔记文本的**保存**按钮。

1.  在`App`组件中，使用`app`状态来决定显示哪个视图（`主页`或`编辑`）。

**代码**

**结果**

![图 6.10：主页视图](img/Figure_6.10.jpg)

###### 图 6.10：主页视图

![图 6.11：编辑视图](img/Figure_6.11.jpg)

###### 图 6.11：编辑视图

![图 6.12：服务器视图](img/Figure_6.12.jpg)

###### 图 6.12：服务器视图

您已成功构建了一个可工作的 React 前端，并将其与 Node.js Express 服务器集成。

#### 注意

此活动的解决方案可在第 293 页找到。

## 总结

在过去 10 多年中，JavaScript 生态系统已经大幅增长。在本章中，我们首先讨论了 JavaScript 生态系统。JavaScript 可用于构建完整的后端 Web 服务器和服务、命令行界面、移动应用程序和前端网站。在第二部分中，我们介绍了 Node.js。我们讨论了如何为浏览器外的 JavaScript 开发设置 Node.js，Node 包管理器，加载和创建模块，基本的 HTTP 服务器，流和管道，文件系统操作以及 Express 服务器。在最后一个主题中，我们介绍了用于前端 Web 开发的 React 框架。我们讨论了安装 React 以及 React 的基础知识和特定内容。

这本书到此结束。在本书中，您学习了 ES6 中的主要特性，并实现了这些特性来构建应用程序。然后，您处理了 JavaScript 浏览器事件，并创建了遵循 TDD 模式的程序。最后，您构建了后端框架 Node.js 和前端框架 React。现在，您应该具备将所学知识应用于实际工作中的工具。感谢您选择本高级 JavaScript 书籍。
