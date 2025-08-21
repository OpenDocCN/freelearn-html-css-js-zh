# 第一章：入门

每个 Web 开发人员都必须偶尔遇到它，即使他们只是涉足简单的网页。每当您想要使您的网页更加交互式时，您会使用您值得信赖的朋友，比如 JavaScript 和 jQuery，并一起开发一些新的东西。您可能已经使用 AngularJS 或 Backbone 开发了一些令人兴奋的前端应用程序，并且想了解您可以用 JavaScript 做些什么。

在多个浏览器上测试您的网站时，您可能会偶尔遇到谷歌浏览器，并且您可能已经注意到它是 JavaScript 应用程序的一个很好的平台。

谷歌浏览器和 Node.js 有一个非常大的共同点：它们都在谷歌的高性能 V8 JavaScript 引擎上运行，这使我们在浏览器中使用的引擎与后端使用的引擎相同，非常酷，对吧？

# 设置

为了开始使用 Node.js，我们需要下载并安装 Node.js。最好的安装方式是前往[`nodejs.org/`](https://nodejs.org/)并下载安装程序。

在撰写本文时，当前版本的 Node.js 是 4.2.1。

为了确保一致性，我们将使用`npm`包来安装正确版本的 Node.JS，为此，我们将使用[`www.npmjs.com/package/n`](https://www.npmjs.com/package/n)中描述的`n`包。

目前，这个包只支持`*nix`机器。对于 Windows，请参见 nvm-windows 或从[`nodejs.org/dist/v4.2.1/`](https://nodejs.org/dist/v4.2.1/)下载二进制文件。

一旦你安装了 Node.js，打开终端并运行：

```js
[~]$ npm install -g n

```

`-g`参数将全局安装包，这样我们就可以在任何地方使用这个包。

Linux 用户可能需要运行安装全局包的命令作为`sudo`。

使用最近安装的包，运行：

```js
[~]$ n

```

这将显示一个包含以下包的屏幕：

```js
 node/0.10.38
 node/0.11.16
 node/0.12.0
 node/0.12.7
 node/4.2.1

```

如果`node/4.2.1`没有标记，我们可以简单地运行以下包；这将确保安装`node/4.2.1`：

```js
[~]$ sudo n 4.2.1

```

为了确保`node`运行正常，让我们创建并运行一个简单的`hello world`示例：

```js
[~/src/examples/example-1]$ touch example.js
[~/src/examples/example-1]$ echo "console.log(\"Hello world\")" > example.js
[~/src/examples/example-1]$ node example.js
Hello World

```

很好，它起作用了；现在让我们开始做正事。

# Hello require

在前面的示例中，我们只是记录了一个简单的消息，没有什么有趣的，所以让我们在这一部分深入一点。

在浏览器中使用多个脚本时，我们通常只需要包含另一个脚本标签，如：

```js
<script type='application/javascript' src='script_a.js'></script>
<script type='application/javascript' src='script_b.js'></script>
```

这两个脚本共享相同的全局范围，这通常会导致一些不寻常的冲突，当人们想要给变量赋予相同的名称时。

```js
//script_a.js
function run( ) {
    console.log( "I'm running from script_a.js!" );
}
$( run );

//script_b.js
function run( ) {
    console.log( "I'm running from script_b.js!" );
}
$( run );
```

这可能会导致混乱，当许多文件被压缩并挤在一起时会导致问题；`script_a`声明了一个全局变量，然后在`script_b`中再次声明，运行代码时，我们在控制台上看到以下内容：

```js
> I'm running from script_b.js!
> I'm running from script_b.js!

```

解决这个问题并限制全局范围的污染最常见的方法是将我们的文件包装在一个匿名函数中，如下所示：

```js
//script_a.js
(function( $, undefined ) {
    function run( ) {
        console.log( "I'm running from script_a.js!" );
    }
    $( run );
})( jQuery );

//script_b.js
(function( $, undefined ) {
    function run( ) {
        console.log( "I'm running from script_b.js!" );
    }
    $( run );
})( jQuery );
```

现在当我们运行这个时，它按预期工作：

```js
> I'm running from script_a.js!
> I'm running from script_b.js!
```

这对于不依赖外部的代码来说是很好的，但是对于依赖外部代码的代码该怎么办呢？我们只需要*导出*它，对吧？

类似以下代码将会起作用：

```js
(function( undefined ) {
    function Logger(){  
    }
    Logger.prototype.log = function( message /*...*/ ){
        console.log.apply( console, arguments );
    }
    this.Logger = Logger; 
})( )
```

现在，当我们运行这个脚本时，我们可以从全局范围访问 Logger：

```js
var logger = new Logger( );
logger.log( "This", "is", "pretty", "cool" )
> This is pretty cool
```

所以现在我们可以分享我们的库，一切都很好；但是如果其他人已经有一个暴露相同`Logger`类的库呢。

`node`是如何解决这个问题的呢？Hello require！

Node.js 有一种简单的方式来从外部来源引入脚本和模块，类似于 PHP 中的 require。

让我们在这个结构中创建一些文件：

```js
/example-2
    /util
        index.js
        logger.js
    main.js

/* util/index.js */
var logger = new Logger( )
var util = {
    logger: logger
};

/* util/logger.js */

function Logger(){
}
Logger.prototype.log = function( message /*...*/ ){
    console.log.apply( console, arguments );
};

/* main.js */
util.logger.log( "This is pretty cool" );
```

我们可以看到`main.js`依赖于`util/index.js`，而`util/index.js`又依赖于`util/logger.js`。

这应该可以正常工作吧？也许不是。让我们运行命令：

```js
[~/src/examples/example-2]$ node main.js
ReferenceError: logger is not defined
 at Object.<anonymous> (/Users/fabian/examples/example-2/main.js:1:63)
 /* Removed for simplicity */
 at Node.js:814:3

```

那么为什么会这样呢？它们不应该共享相同的全局范围吗？嗯，在 Node.js 中，情况有些不同。还记得我们之前包装文件的那些匿名函数吗？Node.js 会自动将我们的脚本包装在其中，这就是 Require 适用的地方。

让我们修复我们的文件，如下所示：

```js
/* util/index.js */
Logger = require( "./logger" )

/* main.js */
util = require( "./util" );  
```

如果您注意到，我在需要`util/index.js`时没有使用`index.js`；原因是当您需要一个文件夹而不是一个文件时，您可以指定一个代表该文件夹代码的索引文件。这对于像模型文件夹这样的东西非常方便，您可以在一个 require 中公开所有模型，而不是为每个模型单独 require。

现在，我们已经需要了我们的文件。但是我们得到了什么？

```js
[~/src/examples/example-2]$ node
> var util = require( "./util" );
> console.log( util );
{} 

```

但是，还没有日志记录器。我们错过了一个重要的步骤；我们没有告诉 Node.js 我们想要在我们的文件中公开什么。

要在 Node.js 中公开某些内容，我们使用一个名为`module.exports`的对象。有一个简写引用，就是*exports*。当我们的文件包装在一个匿名函数中时，*module*和*exports*都作为参数传递，如下例所示：

```js
function Module( ) {
    this.exports = { };
}

function require( file ) {
    // .....
    returns module.exports;
} 

var module = new Module( );
var exports = module.exports;

(function( exports, require, module ) {
    exports = "Value a"
    module.exports = "Value b"
})( exports, require, module );
console.log( module.exports );
// Value b
```

### 提示

**下载示例代码**

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)购买的所有 Packt 图书中下载示例代码文件。如果您在其他地方购买了本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便直接通过电子邮件接收文件。

示例显示*exports*最初只是对`module.exports`的引用。这意味着，如果您使用`exports = { }`，则您设置的值在函数范围之外将无法访问。但是，当您向*exports*对象添加属性时，实际上是向`module.exports`对象添加属性，因为它们都是相同的值。将值分配给`module.exports`将导出该值，因为它可以通过模块在函数范围之外访问。

有了这个知识，我们最终可以以以下方式运行我们的脚本：

```js
/* util/index.js */
Logger = require( "./logger.js" );
exports.logger = new Logger( );

/* util/logger.js */
function Logger( ){
} 
Logger.prototype.log = ( message /*...*/ ) {
    console.log.apply( console, arguments );
};
module.exports = Logger;

/* main.js */
util = require( "./utils" );
util.logger.log( "This is pretty cool" );
```

运行`main.js`：

```js
[~/src/examples/example-2]$ node main.js
This is pretty cool

```

还可以使用 Require 在我们的代码中包含模块。在需要模块时，我们不需要使用文件路径，只需要使用我们想要的`node`模块的名称。

Node.js 包括许多预构建的核心模块，其中之一是`util`模块。您可以在[`nodejs.org/api/util.html`](https://nodejs.org/api/util.html)找到`util`模块的详细信息。

让我们看看`util`模块命令：

```js
[~]$ node
> var util = require( "util" )
> util.log( 'This is pretty cool as well' )
01 Jan 00:00:00 - This is pretty cool as well 

```

# 你好 npm

除了内部模块之外，还有一个完整的包生态系统；Node.js 最常见的包管理器是`npm`。截至目前，共有 192,875 个可用的包。

我们可以使用`npm`来访问为我们执行许多操作的包，从路由 HTTP 请求到构建我们的项目。您还可以浏览[`www.npmjs.com/`](https://www.npmjs.com/)上提供的包。

使用包管理器，您可以引入其他模块，这很好，因为您可以花更多时间在业务逻辑上，而不是重新发明轮子。

让我们下载以下包，使我们的日志消息变得丰富多彩：

```js
[~/src/examples/example-3]$ npm install chalk

```

现在，要使用它，创建一个文件并需要它：

```js
[~/src/examples/example-3]$ touch index.js
/* index.js */
var chalk = require( "chalk" );
console.log( "I am just normal text" )
console.log( chalk.blue( "I am blue text!" ) )

```

运行此代码时，您将看到默认颜色的第一条消息和蓝色的第二条消息。让我们看看这个命令：

```js
[~/src/examples/example-3]$ node index.js
I am just normal text
I am blue text!

```

当您需要某个其他人已经实现的东西时，下载现有包的能力非常方便。正如我们之前所说，有很多可供选择的包。

我们需要跟踪这些依赖关系，有一个简单的解决方案：`package.json`。

使用`package.json`，我们可以定义诸如项目名称、主要脚本是什么、如何运行测试、我们的依赖关系等内容。您可以在[`docs.npmjs.com/files/package.json`](https://docs.npmjs.com/files/package.json)找到属性的完整列表。

`npm`提供了一个方便的命令来创建这些文件，并且会询问您创建`package.json`文件所需的相关问题：

```js
[~/src/examples/example-3]$ npm init

```

上述实用程序将引导您完成创建`package.json`文件的过程。

它只涵盖了最常见的项目，并尝试猜测有效的默认值。

运行`npm help json`命令以获取有关这些字段的最终文档，并了解它们的确切作用。

之后，使用`npm`和安装`<pkg> --save`来安装一个包并将其保存为`package.json`文件中的依赖项。

按`^C`随时退出：

```js
name: (example-3)
version: (1.0.0) 
description: 
entry point: (main.js)
test command: 
git repository: 
keywords:
license: (ISC) 
About to write to /examples/example-3/package.json:
{
  "name": "example-3",
  "version": "1.0.0",
  "description": "",
  "main": "main.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "....",
  "license": "ISC"
}
Is this ok? (yes) 
```

该实用程序将为您提供默认值，因此最好只需使用*Enter*键跳过它们。

现在，在安装我们的包时，我们可以使用`--save`选项将`chalk`保存为依赖项，如下所示：

```js
[~/src/examples/example-3]$ npm install --save chalk
```

我们可以看到 chalk 已经被添加了：

```js
[~/examples/example-3]$ cat package.json
{
  "name": "example-3",
  "version": "1.0.0",
  "description": "",
  "main": "main.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "...",
  "license": "ISC",
  "dependencies": {
    "chalk": "¹.0.0"
  }
}
```

我们可以通过修改`package.json`文件手动添加这些依赖项；这是保存依赖项的最常见方法。

您可以在此处阅读有关包文件的更多信息：[`docs.npmjs.com/files/package.json`](https://docs.npmjs.com/files/package.json)。

如果您正在创建服务器或应用程序而不是模块，您很可能希望找到一种方法，以便无需始终提供主文件的路径来启动您的进程；这就是`package.json`文件中的脚本对象发挥作用的地方。

要设置启动脚本，您只需在`scripts`对象中设置`start`属性，如下所示：

```js
"scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node server.js"
}
```

现在，我们需要做的就是运行 `npm` start，然后 `npm` 将运行我们已经指定的启动脚本。

我们可以定义更多的脚本，例如，如果我们想要一个用于开发环境的启动脚本，我们也可以定义一个开发属性；但是，对于非标准的脚本名称，我们需要使用`npm run <script>`而不是只使用`npm <script>`。例如，如果我们想要运行我们的新开发脚本，我们将不得不使用`npm run development`。

`npm`具有在不同时间触发的脚本。我们可以定义一个`postinstall`脚本，该脚本在运行`npm install`后运行；如果我们想要触发包管理器来安装模块（例如，bower），我们可以使用这个。

您可以在此处阅读有关脚本对象的更多信息：[`docs.npmjs.com/misc/scripts`](https://docs.npmjs.com/misc/scripts)。

如果您正在团队开发中工作，需要定义一个包，其中项目将安装在不同的机器上。如果您使用诸如**git**之类的源代码控制工具，建议您将`node_modules`目录添加到您的忽略文件中，如下所示：

```js
[~/examples/example-3]$ echo "node_modules" > .gitignore
[~/examples/example-3]$ cat .gitignore
node_modules

```

# 总结

这很快，不是吗？我们已经涵盖了我们继续旅程所需的 Node.js 的基础知识。

我们已经介绍了相对于常规 JavaScript 代码在浏览器中，如何轻松地暴露和保护公共和私有代码，全局范围可能会受到严重污染。

我们还知道如何从外部源包括包和代码，以及如何确保所包含的包是一致的。

正如您所看到的，在许多包管理器中有一个庞大的包生态系统，例如`npm`，正等待我们使用和消耗。

在下一章中，我们将专注于创建一个简单的服务器来路由、认证和消耗请求。

为 Bentham Chang 准备，Safari ID 为 bentham@gmail.com 用户编号：2843974 © 2015 Safari Books Online，LLC。此下载文件仅供个人使用，并受到服务条款的约束。任何其他用途均需版权所有者的事先书面同意。未经授权的使用、复制和/或分发严格禁止，并违反适用法律。保留所有权利。
