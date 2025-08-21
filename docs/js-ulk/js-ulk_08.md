# 第七章：JavaScript 浏览器之外

最初，JavaScript 被设计为客户端脚本语言，但今天，它被用在实实在在的每个地方：在服务器脚本、移动和桌面软件编程、游戏开发、数据库查询、硬件控制和操作系统自动化。当你有客户端 JavaScript 的经验时，加上一些额外的知识，你也可以将你的技能应用到其他编程领域。在这里，我们将学习如何使用 JavaScript 编写命令行工具、web 服务器、桌面应用程序和移动软件。

在本章中，我们将学习以下内容：

+   用 JavaScript 提升命令行程序的编程水平

+   用 JavaScript 建立 web 服务器

+   编写桌面 HTML5 应用程序

+   使用 PhoneGap 制作移动原生应用

# 用 JavaScript 提升命令行程序的编程水平

你一定听说过 Node.js。这是一个开源的跨平台开发环境，它允许使用 JavaScript 创建 web 服务器、网络和其他工具。（https://nodejs.org/api/index.html）。Node.js 在经典的 JavaScript 上增加了一系列专门的模块。这些模块处理文件系统 I/O、网络、操作系统级操作、二进制数据、加密功能、数据流等。Node.js 使用事件驱动的 I/O 模型。与 JavaScript 类似，它在一个单线程上执行非阻塞调用。因此，耗时的函数可以通过在完成时调用回调来并发运行。

为了感受 Node.js，我们从一个简单地打印*Hello world*的示例开始：

**hello.js**

```js
console.log( "Hello world!" );
```

现在让我们打开控制台（命令行界面：Windows 中的**CMD**，或 Linux 和 Mac OS 中的**Terminal**），导航到示例脚本位置，并运行以下命令：

```js
node hello.js
```

好了，我们在输出中得到了`Hello world!`。

下面的屏幕截图显示了 Windows CMD

![用 JavaScript 提升命令行程序的编程水平](img/00016.jpeg)

Node.js 模块遵循与我们在第二章 *Modular Programming with JavaScript*中考察过的相同的 CommonJS 规范：

**foo.js**

```js
console.log( "Running foo.js" );
module.exports = "foo";
main.js
var foo = require( "./foo" );
console.log( "Running main.js" );
console.log( "Exported value:", foo );
```

当我们运行`main.js`时，我们预计会得到以下输出：

```js
Running foo.js
Running main.js
Exported value: foo
```

Node.js 本地模块，如`fs`（[`nodejs.org/api/index.html`](https://nodejs.org/api/index.html)），不需要下载。我们只需在`require()`中引用它们，在运行时，它将知道在哪里找到它们：

```js
"use strict";
var fs = require( "fs" );
fs.readFile( __filename, "UTF-8", function( err, data ){
  if ( err ) {
    throw new Error( err );
  }
  console.log( "Source of ", __filename, ":\n", data );
});
```

这里我们使用文件系统 I/O（`fs`）模块来读取一个文件。模块作用域中的`__filename`属性包含执行源文件的绝对路径。记住我们在第五章 *Asynchronous JavaScript*中考察过的*错误优先回调*方法。这是 Node.js 中异步函数的主要接口。

现在让我们尝试一些更实际的东西。我们将编写一个工具，递归地扫描给定目录中的所有源文件，以确保每个文件都有带有最新版权的块注释。首先，我们需要一个模块，它可以测试提供的块注释文本是否包含实际的版权行：

```js
./Lib/BlockComment.js 
   /**
   * Block comment entity
   * @class
   * @param {String} code
   */
var BlockComment = function( code ){
  return {
    /**
     * Check a block comment
     * @returns {Boolean}
     */
    isValid: function(){
      var lines = code.split( "\n" );
      return lines.some(function( line ){
          var date = new Date();
          return line.indexOf( "@copyright " + date.getFullYear() ) !== -1;
        });
    }
  };
};

module.exports = BlockComment;
```

在这里，我们有一个构造函数，用于创建代表`BlockComment`的对象。该对象有一个方法（`isValid`），用于测试其有效性。因此，如果我们用块注释文本创建一个`BlockComment`实例，我们可以将其与我们的要求进行验证：

```js
var comment = new BlockComment( "/**\n* @copyright 2015 \n*/" );
comment.isValid() // true 
```

现在，我们将编写一个模块，用于测试给定源代码中所有版权行是否包含实际年份：

```js
./Lib/SourceFile.js
    /** @type {module:esprima} */
var esprima = require( "esprima" ),

/**
 * Source file entity
 * @class
 * @param {String} fileSrc
 * @param {module:Lib/BlockComment} BlockComment - dependency injection
 */
SourceFile = function( fileSrc, BlockComment ){
  return {
    /**
     * Test if source file has valid copyright
     */
    isValid: function() {
      var blockComments = this.parse( fileSrc );
      return Boolean( blockComments.filter(function( comment ){
        return comment.isValid();
      }).length );
    },
    /**
     * Extract all the block comments as array of BlockComment instances
     * @param {String} src
     * @returns {Array} - collection of BlockComment
     */
    parse: function( src ){
      return esprima.parse( src, {
        comment: true
      }).comments.filter(function( item ){
        return item.type === "Block";
      }).map(function( item ){
        return new BlockComment( item.value );
      });
    }

  };
};

module.exports = SourceFile;
```

在这个例子中，我们引入了一个`SourceFile`对象，它有两个方法，`parse`和`isValid`。私有方法`parse`从给定的 JavaScript 源代码中提取所有块注释，并返回`BlockComment`对象的数组。`isValid`方法检查所有接收的`BlockComment`对象是否符合我们的要求。在这些方法中，为了操作数组，我们使用了我们在第一章中介绍的*深入 JavaScript 核心*的`Array.prototype.filter`和`Array.prototype.map`。

那么，我们如何可靠地从 JavaScript 源代码中提取`blockComments`呢？最好的方法是使用一个叫做**esprima**解析器的解决方案（[`esprima.org/`](http://esprima.org/)），它执行代码静态分析，并返回包括注释在内的完整语法树。然而，esprima 是一个第三方包，应该从应用程序中下载并链接。通常，一个包可能依赖于其他包，这些包也有依赖关系。看起来把所需的依赖项集合在一起可能是一项艰巨的工作。幸运的是，Node.js 随 NPM 包管理器一起分发。这个工具可以用来在 NPM 仓库（[`www.npmjs.com/`](https://www.npmjs.com/)）中安装和管理第三方模块。NPM 不仅下载请求的模块，还解析模块依赖项，允许在项目范围或全局范围内有一个细粒度的可重用组件结构。

所以，为了在我们的应用程序中使用`esprima`，我们只需使用这个命令请求它：`npm install esprima`。

通过在控制台运行这个命令，我们自动得到一个包含`esprima`包的新`node_modules`子目录。如果该包需要任何依赖项，它们将被获取并在`node_modules`中分配。一旦通过 NPM 安装了包，Node.js 就可以通过名称找到它。例如，`require( "esprima"` ）。现在我们有了`SourceFile`对象，我们只需要主脚本，它将读取给定目录中的文件并与`SourceFile`进行测试：

**copyright-checker.js**

```js
        /** @type {module:cli-color} */
var clc = require( "cli-color" ),
    /** @type {module:fs-walk} */
    walk = require( "fs-walk" ),
    /** @type {module:path} */
    path = require( "path" ),
    /** @type {module:fs} */
    fs = require( "fs" ),
    /**
     * Source file entity
     * @type {module:Lib/SourceFile}
     */
    SourceFile = require( "./Lib/SourceFile" ),
    /** @type {module:Lib/BlockComment} */
    BlockComment = require( "./Lib/BlockComment" ),
    /**
     * Command-line first argument (if none given, go with ".")
     * @type {String}
     */
    dir = process.argv[ 2 ] || ".";

console.log( "Checking in " + clc.yellow( dir ) );

// Traverse directory tree recursively beginning from 'dir'
walk.files( dir, function( basedir, filename ) {
      /** @type {Function} */
  var next = arguments[ 3 ],
      /** @type {String} */
      fpath = path.join( basedir, filename ),
      /** @type {String} */
      fileSrc = fs.readFileSync( fpath, "UTF-8" ),
      /**
       * Get entity associated with the file located in fpath
       * @type {SourceFile}
       */
      file = new SourceFile( fileSrc, BlockComment );
  // ignore non-js files
  if ( !filename.match( /\.js$/i ) ) {
    return next();
  }
  if ( file.isValid() ) {
    console.log( fpath + ": " + clc.green( "valid" ) );
  } else {
    console.log( fpath + ": " + clc.red( "invalid" ) );
  }
  next();
}, function( err ) {
  err && console.log( err );
});
```

在这段代码中，我们依赖了一个第三方模块，`cli-color`，来为命令行输出着色。我们使用了`fs-walk`模块递归地遍历目录。而 Node.js 本地模块，path，允许我们通过给定的相对目录和文件名解析绝对路径，`fs`内置模块用于读取文件。

由于我们打算从控制台运行我们的应用程序，我们可以使用命令行选项来传递一个我们想要测试的目录：

```js
node copyright-checker.js some-dir
```

我们可以从内置进程（`process.argv`）对象中提取脚本参数。对于这个命令，`process.argv`将包含一个数组，像这样：

```js
[ "node", "/AbsolutePath/copyright-checker.js", "some-dir" ]
```

因此，在主脚本中，现在我们可以将这个数组的第三个元素传递给`walk.files`。该函数将遍历给定目录，为找到的每个文件运行回调函数。在回调函数中，如果文件名看起来像 JavaScript，我们就读取内容并使用`SourceFile`对象进行测试。

在我们能够运行主脚本之前，我们需要从 NPM 那里获取第三方包，这些包将在脚本中使用：

```js
npm install fs-walk cli-color
```

现在我们可以运行了。当我们运行`node copyright-checker.js fixtures`时，我们得到了一个有关位于 fixtures 中的 JavaScript 文件有效性的报告。

下面的屏幕截图显示了 Mac OS X 终端：

![用 JavaScript 提升命令行程序的编码水平](img/00017.jpeg)

# 使用 JavaScript 构建网页服务器

我们刚刚学习了如何使用 Node.js 编写命令行脚本。然而，这种运行时通常被称为服务器端 JavaScript，意味着这是运行 HTTP 服务器的软件。实际上，Node.js 特别适合这类工作。如果我们基于 Node.js 启动一个服务器应用程序，它会持续运行，只初始化一次。例如，我们可能创建一个单一的数据库连接对象，并在有人请求应用程序时重复使用它。此外，它还赋予我们所有 JavaScript 的灵活性和力量，包括事件驱动、非阻塞 I/O。

那么我们如何利用这一点呢？多亏了 Node.js 的 HTTP 本地模块，一个简单的网页服务器可以像这样轻易实现：

```js
simple-server.js
"use strict";
    /** @type {module:http}  */
var http = require( "http" ),
    /** @type {HttpServer}  */
    server = http.createServer(function( request, response ) {
      response.writeHead( 200, {"Content-Type": "text/html"} );
      response.write( "<h1>Requested: " + request.url + "</h1>" );
      response.end();
    });

server.listen( 80 );
console.log( "Server is listening..." );
```

在此我们创建了一个带有调度程序回调的服务器来处理 HTTP 请求。然后，让这个服务器监听 80 端口。现在从控制台运行`node simple-server.js`，然后在浏览器中访问`http://localhost`。我们会看到如下内容：

```js
Requested: /
```

所以，我们只需要路由传入的请求，读取相应的 HTML 文件，并通过响应将它们发送出去，以创建一个简单的静态网页服务器。或者我们可以安装现有的模块，`connect`和`serve-static`：

```js
npm install connect serve-static
```

使用以下方式实现服务器：

```js
"use strict";
    /** @type {module:connect}  */
var connect = require( "connect" ),
    /** @type {module:serve-static}  */
    serveStatic = require( "serve-static" );

connect().use( serveStatic( __dirname ) ).listen( 80 );
```

在实际应用中，路由请求可能是一个具有挑战性的任务，因此我们更倾向于使用一个框架。例如，Express.js ([`expressjs.com`](http://expressjs.com))。然后，我们的路由可能如下所示：

```js
"use strict";
    /** @type {module:express}  */
var express = require( "express" ),
    /** @type {module:http}  */
    http = require( "http" ),
    /** @type {Object}  */
    app = express();
// Send common HTTP header for every incoming request
app.all( "*", function( request, response, next ) {
  response.writeHead( 200, { "Content-Type": "text/plain" } );
  next();
});
// Say hello for the landing page
app.get( "/", function( request, response ) {
  response.end( "Welcome to the homepage!" );
});
// Show use if for requests like http://localhost/user/1
app.get( "/user/:id", function( request, response ) {
  response.end( "Requested ID: "  + req.params.id );
});
// Show `Page not found` for any other requests
app.get( "*", function( request, response ) {
  response.end( "Opps... Page not found!" );
});

http.createServer( app ).listen( 80 );
```

# 编写桌面 HTML5 应用程序

你是否曾经想过用 HTML5 和 JavaScript 编写桌面应用程序？现在，我们可以使用 NW.js 非常容易地做到这一点。这个项目是一个基于 Chromium 和 Node.js 的跨平台应用程序运行时。因此，它提供了一个无框架浏览器，其中既可以使用 DOM API，也可以使用 Node.js API。换句话说，我们可以运行 NW.js 经典网络应用程序，访问低级 API（文件系统，网络，进程等），并重用 NPM 仓库的模块。有趣吗？我们将开始一个教程，我们将创建一个简单的 HTML5 应用程序并使用 NW.js 运行它。它将是一个具有输入名字表单和已提交列表的阵容应用程序。名字将存储在 localStorage 中。让我们摇滚起来。

## 设置项目

首先，我们必须从[`nwjs.io`](http://nwjs.io)下载与我们的平台（Mac OS X，Windows 或 Linux）相关的 NW.js 运行时。在 NW.js 可执行文件（`nw.exe`，`new.app`或`nw.`，取决于平台）旁边，我们将`package.json`文件放置在描述我们项目的位置：[`github.com/nwjs/nw.js/wiki/manifest-format`](https://github.com/nwjs/nw.js/wiki/manifest-format)

```js
{
  "name": "roster",
  "main": "wwwroot/index.html",
  "window": {
    "title": "The Roster",
    "icon": "wwwroot/roaster.png",
    "position": "center",
    "resizable": false,
    "toolbar": false,
    "frame": false,
    "focus": true,
    "width": 800,
    "height": 600,
    "transparent": true
  }
}
```

我们的`package.json`文件有三个主要字段。`name`包含与项目关联的唯一名称。请注意，此值将是应用程序数据（sessionStorage，localStorage 等）存储的目录路径的一部分。`main`接受项目主要 HTML 页面的相对路径。最后，`window`描述了将显示 HTML 的浏览器窗口。

## 添加 HTML5 应用程序

根据`package.json`中的`main`字段，我们将把我们的`index.html`放入`wwwroot`子目录中。我们可以尝试用简单的 HTML 如下：

```js
<html>
  <body>
    Hello world!
  </body>
</html>
```

NW.js 以与浏览器相同的方式处理 HTML，因此如果我们现在启动 NW.js 可执行文件，我们将看到`Hello world!`。为了给它外观和感觉，我们可以添加 CSS 和 JavaScript。因此，我们可以用与浏览器相同的方式编写 NW.js 的代码。在这里，我们有一个很好的机会来应用我们在第六章中学习到的原则，*大规模 JavaScript 应用程序架构*。为了使示例简洁但具有表现力，我们将采用 AngularJS 方法。首先，我们将创建 HTML。主体的标记将如下所示：

```js
<main class="container">
  <form >
    <div class="form-group">
      <label for="name">Name</label>
      <input class="form-control">
    </div>
    <button class="btn btn-danger">Empty List</button>
    <button type="submit" class="btn btn-primary">Submit</button>
  </form>
  <table class="table table-condensed">
    <tr>
      <td></td>
    </tr>
  </table>
</main>
```

我们定义了一个表单来提交新名字和一个表格来显示已经存储的名字。为了使其更漂亮，我们使用了 Bootstrap([`getbootstrap.com`](http://getbootstrap.com))样式。CSS 文件可以从 CDN 加载，如下所示：

```js
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/css/bootstrap.min.css">
```

现在我们将通过添加 AngularJS 指令来使其生动起来：

```js
<html>
<body ng-app="myApp" >
      <main ng-controller="RosterController" class="container">
        <form ng-submit="submit()">
          <div class="form-group">
            <label for="name">Name</label>
            <input class="form-control" id="name" name="name" ng-model="name" required placeholder="Name">
          </div>
          <button ng-click="empty()" class="btn btn-danger">Empty List</button>
          <button type="submit" class="btn btn-primary">Submit</button>
        </form>
        <table class="table table-condensed">
          <tr ng-repeat="person in persons">
            <td>{{person.value}}</td>
          </tr>
        </table>
      </main>
  </body>
</html>
```

在这里我们声明了一个`myApp`模块作用域（`<body ng-app="myApp" >`）。在此范围内，我们定义了一个`RosterController`控制器。在控制器的边界内，我们将输入字段绑定到模型名称（`<input ng-model="name">`）并为表单提交和“空列表”按钮点击事件（`<form ng-submit="submit()">`和`<button ng-click="empty()">`）设置处理程序。最后，我们将一个模板从表格中绑定到`$scope.persons`集合。因此，每当集合发生变化时，表格就会更新：

```js
<table class="table table-condensed">
  <tr ng-repeat="person in persons">
    <td>{{person.value}}</td>
  </tr>
</table>
```

现在是我们向我们的 HTML 添加一些 JavaScript 的时候了：

```js
<script>
  var app = angular.module( "myApp", [ "ngStorage" ]);

  app.controller("RosterController", function( $scope, $localStorage ) {
    var sync = function() {
      $scope.persons = JSON.parse( $localStorage.persons || "[]" );
    };
    sync();
    $scope.name = "";
    $scope.submit = function() {
      sync();
      $scope.persons.push({ value: $scope.name });
      $localStorage.persons = JSON.stringify( $scope.persons );
    };
    $scope.empty = function() {
      $localStorage.persons = "[]";
      sync();
    };
  });
</script>
```

由于我们打算存储表单提交的数据，我们可以使用我们在第四章中讨论的*HTML5 APIs*中提到的 localStorage。为了以 AngularJS 的方式获取 localStorage，我们使用了`ngStorage`模块（[`github.com/gsklee/ngStorage`](https://github.com/gsklee/ngStorage)）。因此，我们在模块初始化时指定插件，这使得插件在控制器中作为一个参数（`$localStorage`）可用。在控制器主体中，我们有一个`sync`函数，它将`$scope.persons`设置为 localStorage 中的人数组。我们在表单提交处理程序（`$scope.submit`）和“空列表”按钮单击处理程序（`$scope.empty`）中调用`sync`函数。它每次都会更新人员表格。在处理提交事件时，我们将`$scope.persons`的值附加到`$scope.persons`并将其保存到 localStorage 中。

为了启用此功能，我们必须加载 AngularJS 和 ngStorage 插件：

```js
<script src="img/angular.min.js"></script>

<script src="img/ngStorage.min.js"></script>
```

现在我们启动 NW.js 可执行文件并让应用程序运行起来。下面的截图展示了在 NW.js 中没有样式的 Roaster 示例应用：

![添加 HTML5 应用程序](img/00018.jpeg)

这很好，但是当我们以无框架方式运行 NW.js 时，我们甚至没有办法关闭应用程序。此外，我们不能在桌面上拖动应用程序窗口。这个问题很容易解决。我们可以在 HTML 正文中添加一个 HTML 片段，带有两个按钮来关闭和最小化应用程序：

```js
<header ng-controller="ToolbarController">
  <a href="#" ng-click="minimize()">Minimize</a>
  <a href="#" ng-click="close()">Close</a>
</header>
```

现在我们为这些按钮订阅监听器，分别调用 NW.js 窗口 API（[`github.com/nwjs/nw.js/wiki/Window`](https://github.com/nwjs/nw.js/wiki/Window)）的关闭和最小化方法：

```js
var win = require( "nw.gui" ).Window.get();
app.controller("ToolbarController", function( $scope ) {
  $scope.close = function(){
    win.close();
  };
  $scope.minimize = function(){
    win.minimize();
  };
});
```

为了使我们的窗口可拖动（[`github.com/nwjs/nw.js/wiki/Frameless-window`](https://github.com/nwjs/nw.js/wiki/Frameless-window)），我们可以使用`-webkit-app-region`CSS 伪类。我们将此设置为在处理容器（头部）上具有拖动值，并在其中设置为不可拖动的值：

```js
header {
  -webkit-app-region: drag;
}
header a {
   -webkit-app-region: no-drag;
}
```

此外，我们美化页面的外观和感觉。注意，在 NW.js 中，我们可以拥有一个透明的背景。因此，我们在`html`元素上设置`border-radius`，使窗口变得圆角：

```js
html {
 height: 100%;
 border-radius: 20px;
 background-color: rgba(0,0,0,0);
}
body {
  min-height: 100%;
  background: linear-gradient(to bottom,  #deefff 0%,#98bede 100%);
  overflow: auto;
}
header {
  text-align: right;
  width: auto;
  padding: 12px;
  background: rgba(255,255,255, 0.5);
  border-radius: 20px 20px 0 0;
  -webkit-app-region: drag;
}
header a {
  margin: 12px;
  -webkit-app-region: no-drag;
}
```

现在我们可以再次启动我们的 NW.js 可执行文件。带有样式的 Roaster 示例应用在 NW.js 中的截图如下：

![添加 HTML5 应用程序](img/00019.jpeg)

请注意，在 Mac OS X/Linux 上，我们必须使用特殊参数（[`github.com/nwjs/nw.js/wiki/Transparency`](https://github.com/nwjs/nw.js/wiki/Transparency)）才能获得透明效果。例如，在 Mac OS X 上我们必须这样做：

```js
open -n ./nwjs.app --args --enable-transparent-visuals –disable-gpu
```

## 调试

还有一些东西缺失了。如果出了问题，我们如何调试和追踪错误？有以下几个选项可供选择：

+   使用`--enable-logging`参数启动 NW.js 可执行文件，并在`stdout`中获取日志。

+   使用`--remote-debugging-port`参数启动 NW.js 可执行文件，并在远程运行的 Chrome 中访问 DevTools 应用程序。例如，我们以`nw --remote-debugging-port=9222`的方式启动项目，并在 Chrome 中寻找`http://localhost:9222`页面。

+   在`package.json`中为窗口启用工具栏和框架。

第一个选项在调试时并不太方便。第二个选项为您提供了一个 DevTools 的简化版，最后一个选项带来了框架，可能会使应用程序看起来很糟糕。幸运的是，我们可以从应用程序中以编程方式调用 DevTools。所以在`DEVELOPMENT/TEST`环境中，您可以添加这段按下*Ctrl* + *Shift* + *I*即可显示 DevTools 的代码：

```js
console.info( "Here we go!" );

document.addEventListener( "keydown", function( e ){
  var key = parseInt( e.key || e.keyCode, 10 );
  // Ctrl-Shift-i
  if ( e.ctrlKey && e.shiftKey && key === 73 ) {
    e.preventDefault();
    win.showDevTools();
  }
}, false );
```

NW.JS 中以编程方式调用的 DevTools 在以下屏幕快照中显示：

![调试](img/00020.jpeg)

## 打包

为了拥有真正的桌面应用程序体验，我们可以将项目的资源和 NW.js 文件打包成一个可执行文件。首先使用 ZIP，我们将项目目录（`wwwroot`）和伴随文件（`node_modules`目录和`NAPI`插件）压缩成`app.nw`。然后，我们将该压缩文件与 NW.js 可执行文件结合。在 Windows 上，可以这样操作：

```js
run copy /b nw.exe+app.nw app.exe
```

如果针对您平台的 NW.js 发行版包含任何组件（例如，Windows 发行版包括 DLLs），可以使用 Enigma 虚拟盒（[`enigmaprotector.com`](http://enigmaprotector.com)）将它们注入到新创建的应用程序可执行文件中。完成啦，现在我们可以将项目以单一文件的形式分发。

# 使用 PhoneGap 制作移动原生应用

好了，现在我们可以用 JavaScript 制作桌面应用程序，那原生移动应用程序呢？有许多基于 web 的框架可用于移动开发（[`en.wikipedia.org/wiki/Multiple_phone_web-based_application_framework`](https://en.wikipedia.org/wiki/Multiple_phone_web-based_application_framework)）。最流行的解决方案之一称为 Adobe PhoneGap，它是在 Apache Cordova 项目之上构建的。总的来说，PhoneGap 应用程序由一个 web 堆栈（HTML5、CSS 和 JavaScript）组成。尽管现在 HTML5 可以访问一些原生功能（加速计、相机、联系人、振动、GPS 等），但不同设备的兼容性不一致且古怪，性能相对较差。所以 PhoneGap 在设备的本地 WebView 中运行 HTML5，并提供对设备资源和 API 的访问（[`en.wikipedia.org/wiki/Foreign_function_interface`](https://en.wikipedia.org/wiki/Foreign_function_interface)）。结果是，我们可以基于 HTML5 编写一个移动应用程序，并使用 PhoneGap 为我们支持（iPhone、Android、黑莓、Windows、Ubuntu、Firefox OS 等）的设备和操作系统构建它。这里的一个好处是，在为移动设备开发时，我们可以重用为 Web 创建的组件。事实上，我们可以将我们为 NW.js 制作的 `roster` 应用程序作为移动应用程序捆绑。那么让我们这样做。

## 设置项目

首先我们需要一个框架。最简单的方法是使用 NPM 工具进行安装：

```js
npm install -g cordova
```

`-g` 选项意味着我们将在全局安装此软件，在设置任何新项目时无需再次安装。

现在我们可以使用以下命令创建一个新项目：

```js
cordova create roster org.tempuri.roster Roster
```

在 `roster` 子目录中，工具为项目创建了一个名为 `Roster` 的项目文件结构，该项目注册在 `org.tempuri.roster` 命名空间中。

现在，我们需要通知 PhoneGap 我们想要支持哪些平台。所以，我们导航到 `roster` 子目录并输入以下内容：

```js
cordova platform add ios
cordova platform add android
```

## 构建项目

在 `www` 子目录中，我们可以找到一个占位符 HTML5 应用程序。我们可以用为 NW.js 编写的 `roster` 应用程序替换它（当然，不包括环境特定的头部容器及其监听器代码）。为了检查项目是否正确初始化，我们运行以下内容：

```js
cordova build ios
cordova emulate ios
```

或者，我们可以使用这个：

```js
cordova build android
cordova emulate android
```

这会构建项目并在特定平台的模拟器中显示它。在 Mac 上，它看起来是这样的。PhoneGap 提供的 `roster` 示例应用程序如下屏幕截图所示：

![构建项目](img/00021.jpeg)

## 添加插件

如前所述，使用 PhoneGap，我们可以访问原生设备功能（[`phonegap.com/about/feature`](http://phonegap.com/about/feature)）。而且，我们还可以安装和使用在`Cordova`仓库中可用的原生插件（[`cordova.apache.org/plugins/`](http://cordova.apache.org/plugins/)）。让我们拿其中一个来说——*cordova-plugin-vibration*。我们可以像这样轻松地将其添加到项目中：

```js
cordova plugin add cordova-plugin-vibration
```

既然我们有了插件，我们可以在我们的 JavaScript 代码中使用其 API：

```js
// Vibrate for 3 seconds
navigator.vibrate(3000);
```

## 调试

至于调试移动应用程序，有多种选择（[`github.com/phonegap/phonegap/wiki/Debugging-in-PhoneGap`](https://github.com/phonegap/phonegap/wiki/Debugging-in-PhoneGap)）。主要思想是使用桌面检查工具来达到应用程序。在 iOS 的情况下，我们选择 Safari WebInspector 桌面。只需在**开发**菜单中找到**iPhone Simulator**选项，并按下与你应用程序 HTML 相对应的**WebView**。同样，我们可以在 Chrome DevTools 中访问 Android WebView([`developer.chrome.com/devtools/docs/remote-debugging#debugging-webviews`](https://developer.chrome.com/devtools/docs/remote-debugging#debugging-webviews))。

# 总结

广泛使用的 Node.js 运行时通过低级 API 扩展 JavaScript，这为我们提供了创建命令行工具、网络服务器和专用服务器（例如 UDP-TCP/WebSocket/SSE 服务器）的方法。只需考虑使用 Node.js 构建的独立操作系统 NodeOS，看看我们可以在 Web 之外走多远。使用 HTML5 和 JavaScript，我们可以编写桌面软件，并轻松地在不同平台上分发。同样，我们可以使用 HTML5/JavaScript 和原生 API 组成移动应用程序。使用诸如 PhoneGap 之类的工具，我们可以为多种移动平台构建应用程序。

在本章中，我们学习了如何访问 DevTools 来调试 NW.js 和 PhoneGap 应用程序。在下一章中，我们将讨论如何高效地使用 DevTools。
