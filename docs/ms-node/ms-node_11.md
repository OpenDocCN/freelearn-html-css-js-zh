# 第十一章：将工作组织成模块

“复杂性必须从已经工作的简单系统中增长。”

– 凯文·凯利，“失控”

Node 的简单模块管理系统鼓励开发可持续增长和可维护的代码库。Node 开发人员有幸拥有一个丰富的生态系统，其中包含了清晰定义的具有一致接口的软件包，易于组合，并通过 npm 交付。在开发解决方案时，Node 开发人员会发现许多他们需要的功能片段已经准备就绪，并且可以迅速将这些开源模块组合成更大的、但仍然一致和可预测的系统。Node 的简单且可扩展的模块架构使得 Node 生态系统迅速增长。

在本章中，我们将介绍 Node 如何理解模块和模块路径的细节，如何定义模块，如何在 npm 软件包存储库中使用模块，以及如何创建和共享新的 npm 模块。通过遵循一些简单的规则，您会发现很容易塑造应用程序的结构，并帮助他人使用您创建的内容。

*模块*和*软件包*将被互换使用，以描述由`require()`编译和返回的文件或文件集合。

# 如何加载和使用模块

在我们开始之前，看一下这三个命令：

```js
$ node --version
v8.1.2 $ npm --version
5.5.1 $ npm install npm@latest -g
```

要安装 Node，您可能会在您喜欢的网络浏览器中导航到[`nodejs.org/en/`](https://nodejs.org/en/)，下载适合您操作系统的安装程序应用，并点击一些确定按钮。当您这样做时，您也会得到 npm。然而，npm 经常更新，所以即使您最近更新了 Node，您可能没有最新版本的 npm。

此外，下载和安装新的 Node 安装程序将更新 Node，但并不总是更新 npm，因此使用`npm install npm@latest -g`来确保您拥有最新版本。

Node 的设计者认为，大多数模块应该由开发人员在用户空间开发。因此，他们努力限制标准库的增长。在撰写本文时，Node 的标准模块库包含以下简短的模块列表：

| **网络和 I/O** | **字符串和缓冲区** | **实用工具** |
| --- | --- | --- |

| TTY UDP/Datagram

HTTP

HTTPS

Net

DNS

TLS/SSL

Readline

FileSystem | Path Buffer

Url

StringDecoder

QueryString | Utilities VM

Readline

Domain

Console

Assert |

| **加密和压缩** | **环境** | **事件和流** |
| --- | --- | --- |

| ZLIB Crypto

PunyCode | Process OS

模块 | 子进程集群

Events

Stream |

模块是通过全局的`require`语句加载的，它接受模块名称或路径作为单个参数。作为 Node 开发人员，您被鼓励通过创建新模块或自己的模块组合来增强模块生态系统，并与世界分享它们。

模块系统本身是在 require（`module`）模块中实现的。

# 模块对象

一个 Node 模块只是一个 Javascript 文件。将可能对外部代码有用的函数（以及其他任何东西）引用到 exports 中，如下所示：

```js
// library1.js
function function1a() {
  return "hello from 1a";
}
exports.function1a = function1a;
```

我们现在有一个可以被另一个文件所需的模块。回到我们的主应用程序，让我们使用它：

```js
// app.js
const library1 = require('./library1'); // Require it
const function1a = library1.function1a; // Unpack it
let s = function1a(); // Use it
console.log(s);
```

请注意，不需要使用`.js`后缀。我们将很快讨论 Node 如何解析路径。

让我们将我们的库变得更大一点，扩展到三个函数，如下所示：

```js
// library1.js
exports.function1a = () => "hello from 1a";
exports.function1b = () => "hello from 1b";
exports.function1c = () => "hello from 1c";

// app.js
const {function1a, function1b, function1c} = require('./library1'); // Require and unpack
console.log(function1a());
console.log(function1b());
console.log(function1c());
```

解构赋值，随着 ES6 引入到 JavaScript 中，是一种很好的方式，可以在一行代码中将许多由所需模块导出的函数分配给它们的本地变量。

# 模块、导出和 module.exports

当您检查 Node 模块的代码时，您可能会看到一些模块使用`module.exports`导出它们的功能，而其他模块则简单地使用`exports`：

```js
module.exports.foo = 'bar';
// vs...
exports.foo = 'bar';
```

有区别吗？简短的答案是否定的。在构建代码时，你可以大多数情况下将属性分配给任何一个。上面提到的两种方法都会“做”同样的事情--导出模块的属性'foo'在两种情况下都将解析为'bar'。

更长的答案是它们之间存在微妙的差异，与 JavaScript 引用工作方式有关。考虑模块首先是如何包装的：

```js
// https://github.com/nodejs/node/blob/master/lib/module.js#L92
Module.wrap = function(script) {
    return Module.wrapper[0] + script + Module.wrapper[1];
};

Module.wrapper = [
    '(function (exports, require, module, __filename, __dirname) { ',
    '\n});'
];
```

创建模块时，它将使用上述代码进行包装。这就是如何将 __dirname 和当然 exports 的“全局变量”注入到您的执行范围中的脚本（内容）中的方式：

```js
// https://github.com/nodejs/node/blob/master/lib/module.js#L625
var wrapper = Module.wrap(content);

var compiledWrapper = vm.runInThisContext(wrapper, {
    filename: filename,
    lineOffset: 0,
    displayErrors: true
});

...
result = compiledWrapper.call(this.exports, this.exports, require, this, filename, dirname);
```

回想一下第十章中关于`vm`上下文的讨论，*测试您的应用程序*？`Module`构造函数本身演示了`exports`只是`Module`对象上的一个空对象文字：

```js
// https://github.com/nodejs/node/blob/master/lib/module.js#L70
function Module(id, parent) {
    this.id = id;
    this.exports = {};
    this.parent = parent;
    updateChildren(parent, this, false);
    this.filename = null;
    this.loaded = false;
    this.children = [];
}
```

总结一下，在最终编译中，`module.exports`包含的内容将被返回给`require`：

```js
// https://github.com/nodejs/node/blob/master/lib/module.js#L500
var module = new Module(filename, parent);
...
Module._cache[filename] = module;
...
return module.exports;
```

总之，当您创建一个模块时，实质上是在定义其在此上下文中的导出：

```js
var module = { exports: {} };
var exports = module.exports;
// ...your code, which can apply to either
```

因此，`exports`只是对`module.exports`的引用，这就是为什么在`exports`对象上设置 foo 与在`module.exports`上设置 foo 是相同的。但是，*如果您将`exports`设置为其他内容*，`module.exports`将**不会**反映出这种变化：

```js
function MyClass() {
    this.foo = 'bar';
}

// require('thismodule').foo will be 'bar'
module.exports = new MyClass();

// require('thismodule').foo will be undefined
exports = new MyClass();
```

正如我们在上面看到的，只有`module.exports`被返回；`exports`从未被返回。如果`exports`覆盖了对`module.exports`的引用，那么该值永远不会逃离编译上下文。为了安全起见，只需使用`module.exports`。

Node 的核心模块也是使用标准的`module.exports`模式定义的。您可以通过浏览定义控制台的源代码来查看这一点：[`github.com/nodejs/node/blob/master/lib/console.js`](https://github.com/nodejs/node/blob/master/lib/console.js)。

# 模块和缓存

一旦加载，模块将被缓存。模块是基于其解析后的文件名缓存的，相对于调用模块进行解析。对 require（`./myModule`）的后续调用将返回相同的（缓存的）对象。

为了证明这一点，假设我们有三个（在这种情况下设计不佳的）模块，每个模块都需要另外两个模块：

```js
// library1.js
console.log("library 1 -\\");
const {function2a, function2b, function2c} = require('./library2');
const {function3a, function3b, function3c} = require('./library3');
exports.function1a = () => "hello from 1a";
exports.function1b = () => "hello from 1b";
exports.function1c = () => "hello from 1c";
console.log("library 1 -/");
```

```js
// library2.js
console.log("library 2 --\\");
const {function1a, function1b, function1c} = require('./library1');
const {function3a, function3b, function3c} = require('./library3');
exports.function2a = () => "hello from 2a";
exports.function2b = () => "hello from 2b";
exports.function2c = () => "hello from 2c";
console.log("library 2 --/");
```

```js
// library3.js
console.log("library 3 ---\\");
const {function1a, function1b, function1c} = require('./library1');
const {function2a, function2b, function2c} = require('./library2');
exports.function3a = () => "hello from 3a";
exports.function3b = () => "hello from 3b";
exports.function3c = () => "hello from 3c";
console.log("library 3 ---/");
```

如果没有缓存，需要其中任何一个将导致无限循环。但是，由于 Node 不会重新运行已加载（或当前正在加载）的模块，所以一切正常：

```js
$ node library1.js
library 1 -\
library 2 --\
library 3 ---\
library 3 ---/
library 2 --/
library 1 -/

$ node library2.js
library 2 --\
library 1 -\
library 3 ---\
library 3 ---/
library 1 -/
library 2 --/

$ node library3.js
library 3 ---\
library 1 -\
library 2 --\
library 2 --/
library 1 -/
library 3 ---/
```

但是，请注意，通过不同的相对路径（例如`../../myModule`）访问相同的模块将返回不同的对象；可以将缓存视为由相对模块路径键入。

可以通过`require('module')._cache`获取当前缓存的快照。让我们来看一下：

```js
// app.js
const u = require('util');
const m = require('module');
console.log(u.inspect(m._cache));
const library1 = require('./library1');
console.log("and again, after bringing in library1:")
console.log(u.inspect(m._cache));

{
  'C:\code\example\app.js': Module {
    id: '.',
    exports: {},
    parent: null,
    filename: 'C:\\code\\example\\app.js',
    loaded: false,
    children: [],
    paths:
    [ 'C:\\code\\example\\node_modules',
      'C:\\code\\node_modules',
      'C:\\node_modules' ]
  }
}

and again, after bringing in library1:

{ 
  'C:\code\example\app.js': Module {
    id: '.',
    exports: {},
    parent: null,
    filename: 'C:\\code\\example\\app.js',
    loaded: false,
    children: [ [Object] ],
    paths: [ 
      'C:\\code\\example\\node_modules',
      'C:\\code\\node_modules',
      'C:\\node_modules' 
    ] 
  },
  'C:\code\example\library1.js': Module {
    id: 'C:\\code\\example\\library1.js',
    exports: { 
      function1a: [Function],
      function1b: [Function],
      function1c: [Function] 
    },
    parent: Module {
      id: '.',
      exports: {},
      parent: null,
      filename: 'C:\\code\\example\\app.js',
      loaded: false,
      children: [Array],
      paths: [Array] 
    },
    filename: 'C:\\code\\example\\library1.js',
    loaded: true,
    children: [],
    paths: [ 
      'C:\\code\\example\\node_modules',
      'C:\\code\\node_modules',
      'C:\\node_modules' 
    ] 
  }
}
```

模块对象本身包含几个有用的可读属性：

+   `module.filename`：定义此模块的文件名。您可以在前面的代码块中看到这些路径。

+   `module.loaded`：模块是否正在加载过程中。如果加载完成，则为布尔值 true。在前面的代码中，library1 已经加载完成（true），而 app 仍在加载中（false）。

+   `module.parent`：需要此模块的模块（如果有）。您可以看到 library1 是如何知道 app 需要它的。

+   `module.children`：此模块所需的模块（如果有）。

您可以通过检查`require.main === module`来确定模块是直接执行的（通过`node module.js`）还是通过`require('./module.js')`，在前一种情况下将返回 true。

# Node 如何处理模块路径

由于模块化应用程序组合是 Node 的方式，您经常会看到（并使用）require 语句。您可能已经注意到，传递给 require 的参数可以采用许多形式，例如核心模块的名称或文件路径。

以下伪代码摘自 Node 文档，按顺序描述了解析模块路径时所采取的步骤：

```js
// require(X) from module at path Y
REQUIRE(X) 
  1\. If X is a core module,
    a. return the core module
    b. STOP
  2\. If X begins with '/'
    a. set Y to be the filesystem root
  3\. If X begins with './' or '/' or '../'
    a. LOAD_AS_FILE(Y + X)
    b. LOAD_AS_DIRECTORY(Y + X)
  4\. LOAD_NODE_MODULES(X, dirname(Y))
  5\. THROW "not found"
LOAD_AS_FILE(X)
  1\. If X is a file, load X as JavaScript text. STOP
  2\. If X.js is a file, load X.js as JavaScript text. STOP
  3\. If X.json is a file, parse X.json to a JavaScript Object. STOP
  4\. If X.node is a file, load X.node as binary addon. STOP
LOAD_INDEX(X)
  1\. If X/index.js is a file, load X/index.js as JavaScript text. STOP
  2\. If X/index.json is a file, parse X/index.json to a JavaScript Object. STOP
  3\. If X/index.node is a file, load X/index.node as a binary addon. STOP
LOAD_AS_DIRECTORY(X)
  1\. If X/package.json is a file,
    a. Parse X/package.json, and look for "main" field.
    b. let M = X + ("main" field)
    c. LOAD_AS_FILE(M)
    d. LOAD_INDEX(M)
  2\. LOAD_INDEX(X)
LOAD_NODE_MODULES(X, START)
  1\. let DIRS=NODE_MODULES_PATHS(START)
  2\. for each DIR in DIRS:
    a. LOAD_AS_FILE(DIR/X)
    b. LOAD_AS_DIRECTORY(DIR/X)
NODE_MODULES_PATHS(START)
  1\. let PARTS = path split(START)
  2\. let I = count of PARTS - 1
  3\. let DIRS = []
  4\. while I >= 0,
    a. if PARTS[I] = "node_modules" CONTINUE
    b. DIR = path join(PARTS[0 .. I] + "node_modules")
    c. DIRS = DIRS + DIR
    d. let I = I - 1
  5\. return DIRS
```

文件路径可以是绝对的或相对的。请注意，本地相对路径不会被隐式解析，必须声明。例如，如果你想要从当前目录中要求`myModule.js`文件，至少需要在文件名前加上`./`；`– require('myModule.js')`将不起作用。Node 将假定你要么引用一个核心模块，要么引用`./node_modules`文件夹中的模块。如果两者都不存在，将抛出一个`MODULE_NOT_FOUND`错误。

如前面的伪代码所示，这个`node_modules`查找会从调用模块或文件的解析路径开始向上查找目录树。例如，如果位于`/user/home/sandro/project.js`的文件调用了`require('library.js')`，Node 将按照以下顺序寻找：

```js
/user/home/sandro/node_modules/library.js
/user/home/node_modules/library.js
/user/node_modules/library.js
/node_modules/library.js
```

将文件和/或模块组织到目录中总是一个好主意。有用的是，Node 允许通过它们所在的文件夹的两种方式来引用模块。给定一个目录，Node 首先会尝试在该目录中找到一个`package.json`文件，或者寻找一个`index.js`文件。我们将在下一节讨论`package.json`文件的使用。在这里，我们只需要指出，如果 require 传递了`./myModule`目录，它将寻找`./myModule/index.js`。

如果你设置了`NODE_PATH`环境变量，那么 Node 将使用该路径信息来进行进一步搜索，如果通过正常渠道找不到请求的模块。出于历史原因，还将搜索`$HOME/.node_modules`、`$HOME/.node_libraries`和`$PREFIX/lib/node`。`$HOME`代表用户的主目录，`$PREFIX`通常是 Node 安装的位置。

# 创建一个包文件

正如在讨论 Node 如何进行路径查找时提到的，模块可能包含在一个文件夹中。如果你正在开发一个适合作为别人使用的模块的程序，你应该将该模块组织在它自己的文件夹中，并在该文件夹中创建一个`package.json`文件。

正如我们在本书的示例中所看到的，`package.json`文件描述了一个模块，有用地记录了模块的名称、版本号、依赖关系等。如果你想通过 npm 发布你的包，它必须存在。在本节中，我们将仅概述该文件的一些关键属性，并对一些不常见的属性提供更多详细信息。

尝试`$ npm help json`以获取所有可用 package.json 字段的详细文档，或访问：[`docs.npmjs.com/files/package.json`](https://docs.npmjs.com/files/package.json)。

`package.json`文件必须符合 JSON 规范。属性和值必须用双引号引起来，例如。

# 简单初始化

你可以手动创建一个包文件，或者使用方便的`$ npm init`命令行工具，它会询问一些问题并为你生成一个`package.json`文件。让我们来看看其中的一些：

+   **名称**：（必需）这个字符串将被传递给`require()`，以加载你的模块。让它简短和描述性，只使用字母数字字符；这个名称将被用在 URL、命令行参数和文件夹名称中。尽量避免在名称中使用`js`或`node`。

+   **版本**：（必需）npm 使用语义化版本，以下都是有效的：

+   >=1.0.2 <2.1.2

+   2.1.x

+   ~1.2

有关版本号的更多信息，请访问：[`docs.npmjs.com/misc/semver`](https://docs.npmjs.com/misc/semver)。

+   **描述**：当人们在`npmjs.org`上搜索包时，他们将会读到这个。让它简短和描述性。

+   **入口点**（主要）：这是应该设置`module.exports`的文件；它定义了模块对象定义的位置。

+   **关键字**：一个逗号分隔的关键字列表，将帮助其他人在注册表中找到你的模块。

+   **许可证**：Node 是一个喜欢宽松许可证的开放社区。*MIT*和*BSD*在这里都是不错的选择。

您可能还希望在开发模块时将`private`字段设置为`true`。这样可以确保 npm 拒绝发布它，避免意外发布尚未完善或时间敏感的代码。

# 向 package.json 添加脚本

另一个优势是 npm 也可以用作构建工具。包文件中的`scripts`字段允许您设置在某些 npm 命令后执行的各种构建指令。例如，您可能希望最小化 Javascript，或执行一些其他构建依赖项的过程，每当执行`npm install`时，您的模块都需要。可用的指令如下：

+   `prepublish`，`publish`，`postpublish`：通过`npm publish`命令以及在本地`npm install`命令中没有任何参数时运行。

+   `prepublishOnly`：在`npm publish`命令上发布之前运行。

+   `prepare`：在包发布之前和在`npm install`命令中没有任何参数的情况下运行。在`prepublish`之后但在`prepublishOnly`之前运行。

+   `prepack`：在通过`npm pack`或`npm publish`打包 tarball 之前运行，并在安装 git 依赖项时运行。

+   `postpack`：在 tarball 生成并移动到其最终位置后运行。

+   `preinstall`，`install`，`postinstall`：通过`npm install`命令运行。

+   `preuninstall`，`uninstall`，`postuninstall`：通过`npm uninstall`命令运行。

+   `preversion`，`version`，`postversion`：通过`npm version`命令运行。

+   `preshrinkwrap`，`shrinkwrap`，`postshrinkwrap`：通过`npm shrinkwrap`命令运行。

+   `pretest`，`test`，`posttest`：通过`npm test`命令运行。

+   `prestop`，`stop`，`poststop`：通过`npm stop`命令运行。

+   `prestart`，`start`，`poststart`：通过`npm start`命令运行。

+   `prerestart`，`restart`，`postrestart`：通过`npm restart`命令运行。请注意，如果没有提供`restart`脚本，`npm restart`将运行`stop`和`start`脚本。

应该清楚的是，pre-命令将在其主要命令（如`publish`）执行之前运行，而 post-命令将在其主要命令执行之后运行。

# npm 作为一个使用自定义脚本的构建系统

您不仅限于仅使用此预定义的默认脚本命令包。在包文件中扩展脚本集合，例如构建说明，是一种非常常见的做法。考虑以下脚本定义：

```js
"dev": "NODE_ENV=development node --inspect --expose-gc index.js"
```

当通过`npm run dev`命令运行此命令时，我们以调试模式（--inspect）启动一个假设的服务器，并公开垃圾收集器，以便我们可以跟踪其对我们应用程序性能的影响。

这也意味着 npm 脚本在许多情况下可以完全替代更复杂的构建系统，如**gulp**或**webpack**。例如，您可能希望使用**Browserify**来捆绑您的应用程序以进行部署，而该构建步骤很容易在脚本中描述：

```js
"scripts" : {
  "build:browserify" : "browserify -t [babelify --presets [react]] src/js/index.js -o build/app.js"
}
```

执行`npm run build:browserify`后，Browserify 将处理 src/js/index.js 文件，通过一个可以编译 React 代码（**babelify**）的转换器（-t）运行它，并将结果输出（-o）到 build/app.js。

此外，npm 脚本在 npm 的主机系统上运行，因此您可以执行系统命令并访问本地安装的模块。您可能要实现的另一个构建步骤是 JavaScript 文件的最小化，并将编译后的文件移动到目标文件夹：

```js
"build:minify": "mkdir -p dist/js uglify src/js/**/*.js > dist/js/script.min.js"
```

在这里，我们使用 OS 命令 mkdir 创建编译文件的目标文件夹，在一个文件夹中对所有 JavaScript 文件执行最小化（本地安装的）**uglify**模块，并将结果的最小化脚本捆绑重定向到一个单独的构建文件。

现在我们可以向我们的脚本集合添加一个通用的构建命令，并在需要部署新构建时简单地使用`npm run build`：

```js
"build": "npm run build:minify && npm run build:browserify"
```

可以以这种方式链接任意数量的步骤。您可以添加测试，运行文件监视器等。

对于您的下一个项目，考虑使用 npm 作为构建系统，而不是用大型和抽象的系统来复杂化您的堆栈，当它们出现问题时很难调试。例如，公司**Mapbox**使用 npm 脚本来管理复杂的构建/测试流水线：[`github.com/mapbox/mapbox-gl-js/blob/master/package.json`](https://github.com/mapbox/mapbox-gl-js/blob/master/package.json)。

# 注册包依赖项

很可能一个给定的模块本身会依赖于其他模块。这些依赖关系在`package.json`文件中使用四个相关属性声明：

+   `dependencies`：您的模块的核心依赖应该驻留在这里。

+   `devDependencies`：在开发模块时，您可能依赖于一些对于将来使用它的人来说并不必要的模块。通常测试套件会包含在这里。这将为使用您的模块的人节省一些空间。

+   `bundledDependencies`：Node 正在迅速变化，npm 包也在变化。您可能希望将一定的依赖包锁定到一个单独的捆绑文件中，并将其与您的包一起发布，以便它们不会通过正常的`npm update`过程发生变化。

+   `optionalDependencies`：包含可选的模块。如果找不到或安装不了这些模块，构建过程不会停止（与其他依赖加载失败时会停止的情况不同）。然后您可以在应用程序代码中检查此模块的存在。

依赖通常使用 npm 包名称定义，后面跟着版本信息：

```js
"dependencies" : {
  "express" : "3.3.5"
}
```

但是，它们也可以指向一个 tarball：

```js
"foo" : "http://foo.com/foo.tar.gz"
```

您可以指向一个 GitHub 存储库：

```js
"herder": "git://github.com/sandro-pasquali/herder.git#master"
```

它们甚至可以指向快捷方式：

```js
"herder": "sandro-pasquali/herder"
```

这些 GitHub 路径也可用于`npm install`，例如，`npm install sandro-pasquali/herder`。

此外，在只有具有适当身份验证的人才能安装模块的情况下，可以使用以下格式来获取安全存储库：

```js
"dependencies": {
  "a-private-repo":
    "git+ssh://git@github.com:user/repo.git#master"
}
```

通过按类型正确组织您的依赖项，并智能地获取这些依赖项，使用 Node 的包系统应该很容易满足构建需求。

# 发布和管理 NPM 包

当您安装 Node 时，npm 会被自动安装，并且它作为 Node 社区的主要包管理器。让我们学习如何在 npm 存储库上设置帐户，发布（和取消发布）模块，并使用 GitHub 作为替代源目标。

为了发布到 npm，您需要创建一个用户；`npm adduser`将触发一系列提示，要求您的姓名、电子邮件和密码。然后您可以在多台机器上使用此命令来授权相同的用户帐户。

要重置您的 npm 密码，请访问：[`npmjs.org/forgot`](https://npmjs.org/forgot)。

一旦您通过 npm 进行了身份验证，您就可以使用`npm publish`命令发布您的包。最简单的方法是从您的包文件夹内运行此命令。您也可以将另一个文件夹作为发布目标（记住该文件夹中必须存在`package.json`文件）。

您还可以发布一个包含正确配置的包文件夹的 gzipped tar 归档文件。

请注意，如果当前`package.json`文件的`version`字段低于或等于现有已发布包的版本，npm 会抱怨并拒绝发布。您可以使用`--force`参数与`publish`来覆盖此行为，但您可能希望更新版本并重新发布。

要删除一个包，请使用`npm unpublish <name>[@<version>]`。请注意，一旦一个包被发布，其他开发人员可能会依赖于它。因此，强烈建议您不要删除其他人正在使用的包。如果您想要阻止某个版本的使用，请使用 npm deprecate `<name>[@<version>] <message>`。

为了进一步协助协作，npm 允许为一个包设置多个所有者：

+   `npm owner ls <package name>`：列出对模块具有访问权限的用户

+   `npm owner add <user> <package name>`：添加的所有者将拥有完全访问权限，包括修改包和添加其他所有者的能力

+   `npm owner rm <user> <package name>`：删除所有者并立即撤销所有权限

所有所有者都拥有相同的权限—无法使用特殊访问控制，例如能够给予写入但不能删除的权限。

# 全局安装和二进制文件

一些 Node 模块作为命令行程序非常有用。与其要求像`$ node module.js`这样运行程序，我们可能希望在控制台上简单地键入`$ module`并执行程序。换句话说，我们可能希望将模块视为安装在系统 PATH 上的可执行文件，并且因此可以从任何地方访问。使用 npm 可以通过两种方式实现这一点。

第一种最简单的方法是使用`-g（全局）`参数安装包如下：

```js
$ npm install -g module
```

如果一个包旨在作为应该全局安装的命令行应用程序，将`package.json`文件的`preferGlobal`属性设置为`true`是一个好主意。该模块仍将在本地安装，但用户将收到有关其全局意图的警告。

确保全局访问的另一种方法是设置包的`bin`属性：

```js
"name": "aModule",
  "bin" : {
    "aModule" : "./path/to/program"
}
```

当安装此模块时，`aModule`将被理解为全局 CLI 命令。任意数量的此类程序可以映射到`bin`。作为快捷方式，可以映射单个程序，如下所示：

```js
"name": "aModule",
  "bin" : "./path/to/program"
```

在这种情况下，包本身的名称（`aModule`）将被理解为活动命令。

# 其他存储库

Node 模块通常存储在版本控制系统中，允许多个开发人员管理包代码。因此，`package.json`的`repository`字段可用于指向这样的存储库，如果需要合作，可以将开发人员指向这样的存储库。考虑以下示例：

```js
"repository" : {
  "type" : "git",
  "url" : "http://github.com/sandro-pasquali/herder.git"
}
"repository" : {
  "type" : "svn",
  "url" : "http://v8.googlecode.com/svn/trunk/"
}
```

同样，您可能希望使用 bugs 字段将用户指向应该提交错误报告的位置：

```js
"bugs": {
  "url": "https://github.com/sandro-pasquali/herder/issues"
}
```

# 锁定文件

最终，npm install 是一个命令，它从`package.json`构建一个`node_modules`文件夹。但是，它总是生成相同的文件夹吗？答案有时是，我们将在稍后详细介绍。

如果您创建了一个新项目，或者最近将 npm 更新到版本 5，您可能已经注意到熟悉的`package.json`旁边有一个新文件—`package-lock.json`。

里面的内容如下：

```js
{
  "name": "app1",
  "version": "1.0.0",
  "lockfileVersion": 1,
  "dependencies": {
    "align-text": {
      "version": "0.1.4",
      "resolved": "https://registry.npmjs.org/align-text/-/align-text-0.1.4.tgz",
      "integrity": "sha1-DNkKVhCT810KmSVsIrcGlDP60Rc=",
      "dev": true
    },
    "babel-core": {
      "version": "6.25.0",
      "resolved": "https://registry.npmjs.org/babel-core/-/babel-core-6.25.0.tgz",
      "integrity": "sha1-fdQrBGPHQunVKW3rPsZ6kyLa1yk=",
      "dev": true,
      "dependencies": {
        "source-map": {
          "version": "0.5.6",
          "resolved": "https://registry.npmjs.org/source-map/-/source-map-0.5.6.tgz",
          "integrity": "sha1-dc449SvwczxafwwRjYEzSiu19BI=",
          "dev": true
        }
      }
    }
  }
}
```

部分内容会立即变得熟悉。这里是您的项目依赖的 npm 包。依赖项的依赖项会适当地嵌套：`align-text`不需要任何东西，而`babel-core`需要`source-map`。

除了`package.json`之外的真正有用的部分是通过解析和完整性字段提供的。在这里，您可以看到 npm 下载并解压缩以创建`npm_modules`中相应文件夹的确切文件，更重要的是，该文件的加密安全哈希摘要。

使用`package-lock.json`，您现在可以获得一个确切和可重现的`node_modules`文件夹。提交到源代码控制中，您可以在代码审查期间的差异中看到依赖模块版本何时发生了变化。此外，到处都是哈希值，您可以更加确信您的应用程序依赖的代码没有被篡改。

`package-lock.json`在这里；它很长，充满了哈希值，但实际上，您可以忽略它。npm 5 中文件的外观并没有改变您习惯的 npm install 和 npm update 等命令的行为。要解释为什么有帮助，有两个开发人员在遇到该文件时通常会提出的常见问题（或感叹）：

1.  这意味着我的`node_modules`文件夹将由这些哈希值组成，对吗？

1.  为什么我的`package-lock.json`文件一直在变化？

答案是（1）不是，（2）这就是为什么。

当 npm 发现一个包的新版本时，它会下载并更新你的`node_modules`文件夹，就像之前一样。使用 npm 5，它还会更新`package-lock.json`，包括新的版本号和新的哈希值。

此外，大多数情况下，这就是你希望它做的。如果有一个包的新版本是你正在开发的项目所依赖的，你可能希望 npm install 给你最新的版本。

但是，如果你不想让 npm 这样做呢？如果你希望它获取确切的这些版本和确切的这些哈希值的模块呢？要做到这一点，不在`package-lock.json`中，而是回到`package.json`中，并处理语义版本号。看看这三个：

+   `1.2.3`

+   `~1.2.3`

+   `¹.2.3`

`1.2.3`确切表示那个版本，没有更早的，也没有更晚的。`~1.2.3`匹配该版本或任何更新的版本。第三个例子中的插入符号`¹.2.3`将引入该版本或更晚的版本，但保持在 1 版本。插入符号是默认的，很可能已经在你的`package.json`文件中写好了。这是有道理的，因为对第一个数字的更改表示一个可能与先前版本不兼容的主要版本，反过来可能会破坏你之前的代码。

除了这三个常见的例子之外，语义版本和 npm 支持的比较器、运算符、标识符和范围还有一个完整的语言。好奇的读者可以在[`docs.npmjs.com/misc/semver`](https://docs.npmjs.com/misc/semver)查看。但是，请记住保持简单！你现在的合作者和未来的自己会感谢你。

所以，npm 正在改变你的`node_modules`文件夹和`package-lock.json`，因为你告诉它在`package.json`中使用`^`。你可以删除所有的插入符号，让 npm 坚持使用确切的版本，但在你想要这样做的情况下，有一个更好的方法：

```js
$ npm shrinkwrap
```

npm 的`shrinkwrap`命令实际上只是将`package-lock.json`重命名为`npm-shrinkwrap.json`。其重要性在于 npm 后续如何使用这些文件。当发布到 npm 时，`package-lock.json`会留下，因为它可能会随着你正在使用的依赖项的新版本的出现而改变。另一方面，`npm-shrinkwrap.json`旨在与你的模块一起发布。

当 npm 在一个带有`npm-shrinkwrap.json`文件的项目上操作时，`shrinkwrap`文件及其确切的版本和哈希值，而不是`package.json`及其版本范围，决定了 npm 如何构建`node_modules`文件夹。就像上世纪 90 年代商场里软件商店的纸板盒一样，你知道里面的东西是从工厂出来时没有改变的，因为去掉了塑料包装。
