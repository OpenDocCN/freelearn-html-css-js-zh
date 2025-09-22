# 第一章：使用 NPM、Bower 和 Grunt 进行组织

在浏览器渲染互联网的早期阶段，JavaScript 一直是网络开发行业的噩梦。现在，它为像 jQuery 这样具有巨大影响力的库提供动力，并且 JavaScript 渲染的内容（与服务器端渲染的内容相对）甚至被许多搜索引擎索引。曾经被认为主要用来生成弹出窗口和警告框的令人烦恼的语言，现在可能已成为世界上最受欢迎的编程语言。

不仅 JavaScript 现在在前端架构中比以往任何时候都更普遍，而且得益于*Node.js*运行时，它也已成为一种服务器端语言。我们还看到了文档型数据库的激增，例如 MongoDB，这些数据库存储并返回 JSON 数据。由于 JavaScript 贯穿整个开发栈，JavaScript 开发者现在可以成为全栈开发者，而无需学习传统的服务器端语言。有了合适的工具和知识，任何 JavaScript 开发者都可以创建完全由他们最擅长的语言组成的*单页应用程序*，并且可以使用像*MEAN*（MongoDB、Express、AngularJS 和 Node.js）这样的架构来实现。

组织对于任何复杂**单页应用程序**（**SPA**）的开发至关重要。如果您从一开始就没有组织起来，您肯定会向您的应用程序引入大量的回归。Node.js 生态系统将帮助您使用一套不可或缺的开源工具来实现这一点，其中我们将讨论三个。

在本章中，您将了解：

+   **Node 包管理器**（**NPM**）

+   Bower 前端包管理器

+   **Grunt** JavaScript 任务运行器

+   如何使用这三个工具共同创建一个有组织的开发环境，这对于创建 SPA 和 MEAN 堆栈架构至关重要。

# 什么是 Node 包管理器？

在任何全栈 JavaScript 环境中，**Node 包管理器**将是您设置开发环境和管理服务器端库的*首选*工具。NPM 可以在全局和隔离环境上下文中使用。我们将首先探讨 NPM 的全局使用。

## 安装 Node.js 和 NPM

NPM 是*Node.js*的一个组件，因此在使用它之前，您必须首先安装 Node.js。您可以在 nodejs.org 上找到 Mac 和 Windows 的安装程序。一旦安装了 Node.js，使用 NPM 就非常简单，并且通过**命令行界面**（**CLI**）完成。首先确保您安装了最新版本的 NPM，因为它比 Node.js 本身更新得更频繁：

```js
$ npm install -g npm

```

当使用 NPM 时，`-g`选项将您的更改应用于全局环境。在这种情况下，您希望您的 NPM 版本应用于全局。如前所述，NPM 可以用于全局和隔离环境中的包管理。在下面的例子中，我们希望将基本开发工具应用于全局，这样您就可以在同一系统上的多个项目中使用它们。

### 小贴士

在 Mac 和一些基于 Unix 的系统上，您可能需要以超级用户身份运行`npm`命令（在命令前加上`sudo`）来全局安装包，这取决于 NPM 是如何安装的。如果您遇到这个问题并且希望不再需要在`npm`前加上`sudo`，请参阅[docs.npmjs.com/getting-started/fixing-npm-permissions](http://docs.npmjs.com/getting-started/fixing-npm-permissions)。

## 配置您的`package.json`文件

对于您开发的任何项目，您将保留一个本地的`package.json`文件来管理您的 Node.js 依赖项。此文件应存储在项目目录的根目录中，并且它仅适用于该隔离环境。这允许您在同一系统上拥有多个具有不同依赖链的 Node.js 项目。

当开始一个新的项目时，您可以从命令行自动创建`package.json`文件：

```js
$ npm init

```

运行`npm init`将引导您通过一系列 JSON 属性名称，通过命令行提示来定义，包括您的应用程序的`name`、`version`版本号、`description`描述等。`name`和`version`属性是必需的，并且如果没有定义它们，您的 Node.js 包将无法安装。在提示中，一些属性将给出括号内的默认值，这样您只需按*Enter*键即可继续。其他属性将允许您按*Enter*键并留空条目，这些条目将不会保存到`package.json`文件中，或者将保存为空值：

```js
name: (my-app)
version: (1.0.0)
description:
entry point: (index.js)

```

`entry point`提示将被定义为`package.json`中的`main`属性，除非您正在开发 Node.js 应用程序，否则这不是必需的。在我们的例子中，我们可以省略这个字段。实际上，`npm init`命令可能强制您保存`main`属性，因此您必须在之后编辑`package.json`来删除它；然而，该字段对您的 Web 应用程序没有任何影响。

如果您知道要使用的适当结构，您也可以选择使用文本编辑器手动创建`package.json`文件。无论您选择哪种方法，您的`package.json`文件的初始版本应类似于以下示例：

```js
{ 
  "name": "my-app", 
  "version": "1.0.0", 
  "author": "Philip Klauzinski", 
  "license": "MIT", 
  "description": "My JavaScript single page application." 
} 

```

如果您希望您的项目是私有的，并确保它不会意外地发布到 NPM 注册表，您可能希望将`private`属性添加到您的`package.json`文件中，并将其设置为`true`。此外，您还可以删除仅适用于已注册包的一些属性：

```js
{ 
  "name": "my-app", 
  "author": "Philip Klauzinski", 
  "description": "My JavaScript single page application.", 
  "private": true 
} 

```

一旦你将 `package.json` 文件设置成你喜欢的样子，你就可以开始为你的应用程序本地安装 Node.js 包了。这就是依赖的重要性开始显现的地方。

### NPM 依赖

在你的 `package.json` 文件中，可以为任何 Node.js 项目定义三种类型的依赖：`dependencies`、`devDependencies` 和 `peerDependencies`。为了构建基于 Web 的 SPA，你将只需要使用 `devDependencies` 声明。

`devDependencies` 是那些在开发你的应用程序时所需的，但不是在它的生产环境或仅仅运行它时所需的。如果其他开发者想为你的 Node.js 应用程序做出贡献，他们需要从命令行运行 `npm install` 来设置适当的发展环境。有关其他类型依赖的信息，请参阅 docs.npmjs.com。

当将 `devDependencies` 添加到你的 `package.json` 文件时，命令行再次发挥作用。让我们以安装 Browserify 为例：

```js
$ npm install browserify --save-dev

```

这将在本地安装 Browserify 并将其及其版本范围保存到你的 `package.json` 文件中的 `devDependencies` 对象中。一旦安装，你的 `package.json` 文件应该类似于以下示例：

```js
{ 
  "name": "my-app", 
  "version": "1.0.0", 
  "author": "Philip Klauzinski", 
  "license": "MIT", 
  "devDependencies": { 
    "browserify": "¹².0.1" 
  } 
} 

```

`devDependencies` 对象将把每个包存储为一个键值对，其中键是 *包名*，值是 *版本号* 或 *版本范围*。Node.js 使用语义版本，其中版本号的三个数字代表 `MAJOR.MINOR.PATCH`。有关语义版本格式化的更多信息，请参阅 [semver.org](http://semver.org)。

#### 更新你的开发依赖

你会注意到默认情况下安装的包的版本号前面有一个 ** caret ** (`^`) 符号。这意味着对于版本号高于 1.0.0 的包，更新只会允许 * patch * 和 * minor * 更新。这是为了防止在更新包到最新版本时，主要版本的变化破坏你的依赖链。

要更新你的 `devDependencies` 并保存新的版本号，你可以在命令行中输入以下内容：

```js
$ npm update --save-dev

```

或者，你可以使用 `-D` 选项作为 `--save-dev` 的快捷方式：

```js
$ npm update -D

```

要更新所有全局安装的 NPM 包到它们的最新版本，运行带有 `-g` 选项的 `npm update`：

```js
$ npm update -g

```

有关 NPM 中的语义版本信息的更多信息，请参阅 [docs.npmjs.com/misc/semver](http://docs.npmjs.com/misc/semver)。

现在你已经设置了 NPM 并知道如何安装你的开发依赖，你可以继续安装 Bower。

# Bower

Bower 是一个用于前端 Web 资产和库的包管理器。你将使用它来维护你的前端堆栈并控制库如 jQuery、AngularJS 以及任何其他对你应用程序 Web 界面必要的组件的版本链。

## 安装 Bower

Bower 也是一个 Node.js 包，所以你将使用 NPM 安装它，就像你在上一节中安装 Browserify 示例时做的那样，但这次你将全局安装该包。这将允许你在系统上的任何位置运行 `bower` 命令，而无需为每个项目本地安装它：

```js
$ npm install -g bower

```

你还可以将 Bower 作为开发依赖项本地安装，这样你就可以在同一系统上为不同的项目维护不同的版本，但这通常不是必要的：

```js
$ npm install bower --save-dev

```

接下来，通过命令行查询版本来检查 Bower 是否已正确安装：

```js
$ bower -v

```

Bower 还需要在你的系统上安装一个 *Git* 版本控制系统，或 *VCS*，以便与包进行交互。这是因为 Bower 直接与 GitHub 通信以获取包管理数据。如果你系统上没有安装 Git，你可以在 [git-scm.com](http://git-scm.com) 找到 Linux、Mac 和 Windows 的安装说明。

## 配置你的 bower.json 文件

设置你的 `bower.json` 文件的过程与 NPM 的 `package.json` 文件类似。它使用相同的 JSON 格式，既有 `dependencies` 也有 `devDependencies`，并且也可以自动创建：

```js
$ bower init

```

一旦你在命令行中输入 `bower init`，你将被提示定义几个属性，其中一些默认值在括号内给出：

```js
? name: my-app 
? version: 0.0.0 
? description: My app description. 
? main file: index.html 
? what types of modules does this package expose? (Press <space> to? what types of modules does this package expose? globals 
? keywords: my, app, keywords 
? authors: Philip Klauzinski 
? license: MIT 
? homepage: http://gui.ninja 
? set currently installed components as dependencies? No 
? add commonly ignored files to ignore list? Yes 
? would you like to mark this package as private which prevents it from being accidentally published to the registry? Yes 

```

### 小贴士

这些问题可能因你安装的 Bower 版本而异。

在 `bower.json` 文件中，大多数属性在你不打算将你的项目发布到 Bower 注册表的情况下是不必要的。你很可能会想将你的包标记为私有，除非你计划注册它并允许其他人将其作为 Bower 包下载。

一旦你创建了 `bower.json` 文件，你可以在文本编辑器中打开它，更改或删除你想要的任何属性。它应该看起来像以下示例：

```js
{ 
  "name": "my-app", 
  "version": "0.0.0", 
  "authors": [ 
    "Philip Klauzinski" 
  ], 
  "description": "My app description.", 
  "main": "index.html", 
  "moduleType": [ 
    "globals" 
  ], 
  "keywords": [ 
    "my", 
    "app", 
    "keywords" 
  ], 
  "license": "MIT", 
  "homepage": "http://gui.ninja", 
  "ignore": [ 
    "**/.*", 
    "node_modules", 
    "bower_components", 
    "test", 
    "tests" 
  ], 
  "private": true 
} 

```

如果你希望保持你的项目私有，你可以在继续之前将你的 `bower.json` 文件减少到两个属性：

```js
{ 
  "name": "my-app", 
  "private": true 
} 

```

一旦你以你喜欢的样子设置了 `bower.json` 文件的初始版本，你就可以开始为你的应用程序安装组件。

### Bower 组件位置和 .bowerrc 文件

Bower 默认会将组件安装到名为 `bower_components` 的目录中。这个目录将直接位于你的项目根目录下。如果你希望将你的 Bower 组件安装到不同的目录名下，你必须创建一个名为 `.bowerrc` 的本地系统文件，并在其中定义自定义目录名：

```js
{ 
  "directory": "path/to/my_components" 
} 

```

只需一个具有单个 `directory` 属性名的对象，就可以定义你的 Bower 组件的自定义位置。在 `.bowerrc` 文件中可以配置许多其他属性。有关配置 Bower 的更多信息，请参阅 bower.io/docs/config/。

### Bower 依赖项

Bower 允许您定义与 NPM 类似的 `dependencies` 和 `devDependencies` 对象。然而，与 Bower 的区别在于，`dependencies` 对象将包含运行您的应用程序所需的组件，而 `devDependencies` 对象则保留用于测试、转译或其他不需要包含在前端堆栈中的组件。

Bower 包使用 CLI 中的 `bower` 命令进行管理。这是一个用户命令，因此不需要超级用户（sudo）权限。让我们从将 jQuery 作为应用程序的前端依赖项安装开始：

```js
$ bower install jquery --save

```

命令行上的 `--save` 选项会将包和版本号保存到 `bower.json` 文件中的 `dependencies` 对象中。或者，您可以使用 `-S` 选项作为 `--save` 的快捷方式：

```js
$ bower install jquery -S

```

接下来，让我们安装 Mocha JavaScript 测试框架作为开发依赖项：

```js
$ bower install mocha --save-dev

```

在这种情况下，我们将使用命令行上的 `--save-dev` 来将包保存到 `devDependencies` 对象中。现在，您的 `bower.json` 文件应该类似于以下示例：

```js
{ 
  "name": "my-app", 
  "private": true, 
  "dependencies": { 
    "jquery": "~2.1.4" 
  }, 
  "devDependencies": { 
    "mocha": "~2.3.4" 
  } 
} 

```

或者，您可以使用 `-D` 选项作为 `--save-dev` 的快捷方式：

```js
$ bower install mocha -D

```

您会注意到，默认情况下，包版本号前面带有波浪线符号 (`~`)，这与 NPM 中的 caret (`^`) 符号不同。波浪线作为对包版本更新的更严格的保护。使用 `MAJOR.MINOR.PATCH` 版本号，运行 `bower update` 只会更新到最新的补丁版本。如果版本号仅由主版本和次要版本组成，`bower update` 将将包更新到最新的次要版本。

## 搜索 Bower 注册表

所有注册的 Bower 组件都通过命令行索引和可搜索。如果您不知道要安装的组件的确切包名，可以执行搜索以检索匹配名称的列表。

大多数组件在其 `bower.json` 文件中都会有一个关键词列表，这样您就可以更容易地找到包，而无需知道确切的名称。例如，您可能想安装 PhantomJS 用于无头浏览器测试：

```js
$ bower search phantomjs

```

返回的列表将包括任何包名中包含 `phantomjs` 或在其关键词列表中的包：

```js
 phantom git://github.com/ariya/phantomjs.git
 dt-phantomjs git://github.com/keesey/dt-phantomjs
 qunit-phantomjs-runner git://github.com/jonkemp/...
 parse-cookie-phantomjs git://github.com/sindresorhus/...
 highcharts-phantomjs git://github.com/pesla/highcharts-phantomjs.git
 mocha-phantomjs git://github.com/metaskills/mocha-phantomjs.git
 purescript-phantomjs git://github.com/cxfreeio/purescript-phantomjs.git

```

您可以从返回的列表中看到，PhantomJS 的正确包名实际上是 `phantom`，而不是 `phantomjs`。现在您知道了正确的名称，可以继续安装包：

```js
$ bower install phantom --save-dev

```

现在您已经安装了 Bower 并了解了如何管理您的前端 Web 组件和开发工具，但如何将它们集成到您的 SPA 中呢？这正是 Grunt 发挥作用的地方。

# Grunt

Grunt 是 Node.js 的 *JavaScript 任务运行器*，如果你之前没有使用过它，那么它可能是你从未意识到需要的最佳工具。你会发现它在包括 CSS 和 JavaScript 代码检查和压缩、JavaScript 模板预编译、LESS 和 SASS 预处理等众多任务中非常有用。确实有 Grunt 的替代品，但它们都没有像 Grunt 那样庞大的插件生态系统（截至撰写本文时）。

Grunt 有两个组件：*Grunt CLI* 和 *Grunt 任务运行器* 本身。Grunt CLI 允许你在安装了 Grunt 的目录中从命令行运行 Grunt 任务运行器命令。这使得你可以在机器上的每个项目中运行不同版本的 Grunt，从而使每个应用程序更容易维护。有关更多信息，请参阅 [gruntjs.com](http://gruntjs.com)。

## 安装 Grunt CLI

你将想要全局安装 Grunt CLI，就像你安装 Bower 一样：

```js
$ npm install -g grunt-cli

```

请记住，Grunt CLI 并非 Grunt 任务运行器。它只是将 `grunt` 命令从命令行提供给你。这种区别很重要，因为虽然 `grunt` 命令将在命令行中全局可用，但它始终会在你运行它的目录中寻找本地安装。

## 安装 Grunt 任务运行器

你将从应用程序根目录（其中包含你的 `package.json` 文件）安装 Grunt 任务运行器本地版本。Grunt 作为 Node.js 包安装：

```js
$ npm install grunt --save-dev

```

一旦你在本地安装了 Grunt，你的 `package.json` 文件应该看起来像以下示例：

```js
{ 
  "name": "my-app", 
  "version": "1.0.0", 
  "author": "Philip Klauzinski", 
  "license": "MIT", 
  "devDependencies": { 
    "grunt": "⁰.4.5" 
  } 
} 

```

你会注意到，如果之前没有安装，你的 `package.json` 文件中已经添加了一个 `devDependencies` 对象。

现在你已经在本地上安装了 Grunt，让我们开始安装一些插件来与之配合使用。

## 安装 Grunt 插件

所有 Grunt 任务插件都是 Node.js 包，因此它们也将使用 NPM 进行安装。由于 Grunt 是一个开源项目，因此有成千上万的 Grunt 插件由众多作者编写。每个 Grunt 的 Node.js 包名称都以 `grunt` 开头。然而，Grunt 团队也维护了许多插件。官方维护的 Grunt 插件都以 `grunt-contrib` 开头，因此如果你只想使用官方维护的 Grunt 插件，你可以通过这种方式来区分它们。要查看和搜索所有注册的 Grunt 插件，请参阅 gruntjs.com/plugins。

由于你将编写一个 JavaScript SPA，让我们首先安装一个用于 Grunt 的 JavaScript *代码检查* 插件。代码检查是指运行一个程序来分析你的代码，以查找错误，在某些情况下，还可以检查格式。始终运行代码检查工具来测试你的 JavaScript 代码的有效语法和格式是一个好主意：

```js
$ npm install grunt-contrib-jshint --save-dev

```

这将安装官方维护的 `JSHint` Grunt 插件，并将其添加到 `package.json` 文件中的 `devDependencies` 对象中，如下所示示例：

```js
{ 
  "name": "my-app", 
  "version": "1.0.0", 
  "author": "Philip Klauzinski", 
  "license": "MIT", 
  "devDependencies": { 
    "grunt": "⁰.4.5", 
    "grunt-contrib-jshint": "⁰.11.3" 
  } 
} 

```

`JSHint`是一个流行的工具，用于检测 JavaScript 代码中的错误和潜在问题。Grunt 插件本身将允许你自动化此过程，这样你就可以在开发过程中轻松检查你的代码。

另一个非常有价值的 Grunt 插件是`grunt-contrib-watch`。此插件允许你运行一个任务，当你在项目中添加、删除或编辑与预定义规则匹配的文件时，它会自动运行其他 Grunt 任务。

```js
$ npm install grunt-contrib-watch --save-dev

```

安装`grunt-contrib-watch`插件后，你的`package.json`文件中的`devDependencies`对象应如下所示：

```js
  "devDependencies": { 
    "grunt": "⁰.4.5", 
    "grunt-contrib-jshint": "⁰.11.3", 
    "grunt-contrib-watch": "⁰.6.1" 
  } 

```

现在你已经安装了一些 Grunt 插件，让我们开始为它们编写一些任务。为了做到这一点，你首先需要为 Grunt 创建一个本地配置文件。

## 配置 Grunt

与 NPM 和 Bower 不同，Grunt 不提供用于初始化其配置文件的`init`命令。相反，可以使用*脚手架*工具来完成这项工作。项目脚手架工具旨在为开发项目设置一些基本的目录结构和配置文件。Grunt 维护一个官方的脚手架工具，称为`grunt-init`，该工具在他们的网站上有所提及。`grunt-init`工具必须与`grunt-cli`全局包和特定项目的本地`grunt`包分开安装。如果全局安装，它将非常有用，因为这样就可以与任何项目一起使用。

```js
$ npm install -g grunt-init

```

我们在这里不会进一步详细介绍`grunt-init`，但如果你想了解更多，可以访问[gruntjs.com/project-scaffolding](http://gruntjs.com/project-scaffolding)。

了解 Grunt 配置的最佳方式是手动编写其配置文件。Grunt 的配置保存在一个名为`Gruntfile.js`的文件中，称为`Gruntfile`，位于项目的根目录中，与`package.json`和`bower.json`一起。如果你不熟悉 Node.js 及其模块和导出概念，`Gruntfile`的语法可能一开始会有些令人困惑。由于 Node.js 文件在服务器上运行，而不是在浏览器中，因此它们与浏览器中加载的文件在交互方式上并不相同，这与浏览器的`globals`有关。

### 理解 Node.js 模块

在 Node.js 中，模块是一个定义在文件内的 JavaScript 对象。模块名称是文件名。例如，如果你想声明一个名为`foo`的模块，你需要创建一个名为`foo.js`的文件。为了使`foo`模块能够被另一个模块访问，它必须被导出。在最基本的形式下，一个模块看起来像以下示例：

```js
module.exports = { 
    // Object properties here 
};
```

每个模块都有一个本地的`exports`变量，它允许你使模块对他人可用。换句话说，文件内的`module`对象指的是当前模块本身，而`module`的`exports`属性使该模块对任何其他模块（或文件）可用。

定义模块的另一种方式是通过导出一个函数，这当然本身也是一个 JavaScript 对象：

```js
module.exports = function() { 
    // Code for the module here 
};
```

当你在文件内部调用 Node.js 模块时，它首先会寻找一个核心模块，所有这些模块都是编译到 Node.js 本身中的。如果名称不匹配核心模块，它将会从你的项目的当前目录或根目录开始寻找名为 `node_modules` 的目录。这个目录就是所有你的本地 NPM 包，包括 Grunt 插件，将被存储的地方。如果你之前执行了 `grunt-contrib-jshint` 和 `grunt-contrib-watch` 的安装，你将看到这个目录现在存在于你的项目中。

现在你对 Node.js 模块的工作方式有了更多的了解，让我们创建一个 `Gruntfile`。

### 创建一个 Gruntfile

`Gruntfile` 使用了之前展示的函数形式的 `module.exports`。这被称为 *包装函数*。`grunt` 模块本身被传递给包装函数。`grunt` 模块将可用于你的 `Gruntfile`，因为你已经在本地上安装了 `grunt` NPM 包：

```js
module.exports = function(grunt) { 
    // Grunt code here 
};
```

这个例子展示了你的初始 `Gruntfile` 应该是什么样子。现在让我们进一步展开它。为了配置 Grunt 并使用它运行任务，你需要访问传递给 `Gruntfile` 的 `grunt` 模块。

```js
module.exports = function(grunt) { 
    'use strict'; 
    grunt.initConfig({ 
        pkg: grunt.file.readJSON('package.json') 
});
};
```

这种基本格式是你接下来将要使用的方式。你可以看到这里调用了 `grunt.initConfig` 方法，并传递了一个配置对象作为参数。这个配置对象就是所有你的 Grunt 任务代码将要放置的地方。在这个例子中显示的 `pkg` 属性，它被分配了 `grunt.file.readJSON('package.json')` 的值，允许你直接从你的 `package.json` 文件中传递关于你的项目的信息。这个属性的用法将在后面的例子中展示。

#### 定义 Grunt 任务配置

大多数 Grunt 任务期望它们的配置定义在具有与任务相同名称的属性中，这是包名的后缀。例如，`jshint` 是我们之前安装的 `grunt-contrib-jshint` 包的 Grunt 任务名称：

```js
module.exports = function(grunt) { 
    'use strict'; 
    grunt.initConfig({ 
        pkg: grunt.file.readJSON('package.json'), 
        jshint: { 
            options: { 
                curly: true, 
                eqeqeq: true, 
                eqnull: true, 
                browser: true, 
                newcap: false, 
                es3: true, 
                forin: true, 
                indent: 4, 
                unused: 'vars', 
                strict: true, 
                trailing: true, 
                quotmark: 'single', 
                latedef: true, 
                globals: { 
                    jQuery: true 
                } 
            }, 
            files: { 
                src: ['Gruntfile.js', 'js/src/*.js'] 
            } 
        } 
    }); 
}; 

```

在这里，你可以看到配置对象的 `jshint` 属性被定义，并且被分配了自己的属性，这些属性适用于 `jshint` Grunt 任务本身。`jshint` 中定义的 `options` 属性包含了你在检查 JavaScript 文件时希望验证的设置。`files` 属性定义了你希望验证的文件列表。有关 `JSHint` 支持的选项及其含义的更多信息，请参阅 jshint.com/docs/。

现在让我们在 `jshint` 任务配置下方添加一个额外的配置，用于 `grunt-contrib-watch` 插件的 `watch` 任务：

```js
watch: { 
    jshint: { 
        files: ['js/src/*.js'], 
        tasks: ['jshint'] 
    } 
} 

```

在这里，我们在 `watch` 任务下添加了一个额外的 `jshint` 命名空间，这允许在同一个配置属性中定义其他 *目标*，并在需要时单独运行。这被称为 *多任务*。多任务中的目标可以任意命名，如果单独调用多任务，它们将按照定义的顺序运行。目标也可以直接调用，这样做将忽略多任务配置中定义的其他目标：

```js
$ grunt watch:jshint

```

这个特定的 `jshint` 目标配置告诉 `watch` 任务，如果任何匹配 `js/src/*.js` 的文件被更改，则运行 `jshint` 任务。

现在你已经在你的 `Gruntfile` 中定义了前两个 Grunt 任务配置，但为了使用它们，我们必须加载 Grunt 任务本身。

#### 加载 Grunt 插件

你已经将 `grunt-contrib-jshint` 插件作为 Node.js 模块安装了，但为了执行 `jshint` 任务，你必须在 `Gruntfile` 中加载该插件。这是在 `grunt.initConfig` 调用之后完成的：

```js
grunt.loadNpmTasks('grunt-contrib-jshint'); 

```

这是你在 `Gruntfile` 中加载所有 Grunt 任务的相同方法调用，并且如果不这样做，任何 Grunt 任务都将不可用。让我们为 `grunt-contrib-watch` 也做同样的事情：

```js
grunt.loadNpmTasks('grunt-contrib-watch'); 

```

你的完整 `Gruntfile` 现在应该看起来像这样：

```js
module.exports = function(grunt) { 
    'use strict'; 
    grunt.initConfig({ 
        pkg: grunt.file.readJSON('package.json'), 
        jshint: { 
            options: { 
                curly: true, 
                eqeqeq: true, 
                eqnull: true, 
                browser: true, 
                newcap: false, 
                es3: true, 
                forin: true, 
                indent: 4, 
                unused: 'vars', 
                strict: true, 
                trailing: true, 
                quotmark: 'single', 
                latedef: true, 
                globals: { 
                    jQuery: true 
                } 
            }, 
            files: { 
                src: ['Gruntfile.js', 'js/src/*.js'] 
            } 
        }, 
        watch: { 
            jshint: { 
                files: ['js/src/*.js'], 
                tasks: ['jshint'] 
            } 
        } 
    }); 
    grunt.loadNpmTasks('grunt-contrib-jshint'); 
    grunt.loadNpmTasks('grunt-contrib-watch'); 
}; 

```

#### 运行 jshint Grunt 任务

现在你已经加载了插件，你可以简单地从命令行运行 `grunt jshint` 来执行带有其定义配置的任务。你应该会看到以下输出：

```js
$ grunt jshint
Running "jshint:files" (jshint) task
>> 1 file lint free.
Done, without errors

```

这将运行你的 `JSHint` 检查选项与定义的文件，目前只包括 `Gruntfile.js`。如果看起来像示例文件所示，并且包括对 `grunt.loadNpmTasks('grunt-contrib-jshint')` 的调用，那么它应该没有错误通过。

现在让我们创建一个新的 JavaScript 文件，并故意包含一些不会通过 JSHint 配置的代码，这样我们就可以看到错误是如何报告的。首先，创建 `js/src` 目录，该目录在 `jshint` 任务的 `files` 属性中定义：

```js
$ mkdir -p js/src

```

然后在当前目录下创建一个名为 `app.js` 的文件，并将以下代码放入其中：

```js
var test = function() { 
    console.log('test'); 
}; 

```

现在再次从命令行运行 `grunt jshint`。你应该会看到以下输出：

```js
$ grunt jshint
Running "jshint:files" (jshint) task
 js/src/app.js
 2 |    console.log('test');
 ^ Missing "use strict" statement.
 1 |var test = function() {
 ^ 'test' is defined but never used.
>> 2 errors in 2 files
Warning: Task "jshint:files" failed. Use --force to continue.
Aborted due to warnings.

```

你会注意到根据 `jshint` 任务配置选项，`js/src/app.js` 报告了两个错误。让我们通过将 `app.js` 中的代码更改为以下内容来修复这些错误：

```js
var test = function() { 
    'use strict'; 
    console.log('test'); 
}; 
test(); 

```

现在如果你再次从命令行运行 `grunt jshint`，它将报告文件没有错误：

```js
$ grunt jshint
Running "jshint:files" (jshint) task
>> 2 files lint free.
Done, without errors.

```

#### 运行 watch Grunt 任务

如前所述，当运行 `watch` 任务时，它将等待与配置中定义的文件模式匹配的更改，并运行任何相应的任务。在这种情况下，我们配置了当任何匹配 `js/src/*.js` 的文件被更改时运行 `jshint`。由于我们在 `watch` 任务中定义了一个名为 `jshint` 的目标，因此 `watch` 任务可以以两种不同的方式运行：

```js
$ grunt watch

```

运行 `grunt watch` 将监视 `watch` 任务中定义的所有目标配置的变化：

```js
$ grunt watch:jshint

```

使用冒号 (`:`) 语法运行 `grunt watch:jshint` 将只运行与该目标配置匹配的文件模式。在我们的例子中，只定义了一个目标，所以让我们只运行 `grunt watch` 并看看控制台会发生什么：

```js
$ grunt watch
Running "watch" task
Waiting...

```

你会看到现在任务在命令行上显示为 `Waiting...` 的状态。这表明任务正在运行以监视其配置中的匹配更改，如果发生任何这些更改，它将自动运行相应的任务。在我们的 `jshint` 任务例子中，它将允许你的代码在每次你更改并保存你的 JavaScript 文件时自动进行代码检查。如果发生 `JSHint` 错误，控制台将发出警报并显示错误。

让我们通过打开一个文本编辑器并再次更改 `js/src/app.js` 来测试它：

```js
var test = function() { 
    console.log('test'); 
}; 
test() 

```

在这里，我们移除了文件末尾 `test()` 调用后的 `use strict` 语句和分号。这应该会引发两个 `JSHint` 错误：

```js
>> File "js/src/app.js" changed.
Running "jshint:files" (jshint) task 
 js/src/app.js
 2 |    console.log('test');
 ^ Missing "use strict" statement.
 4 |test()
 ^ Missing semicolon.
>> 2 errors in 2 files
Warning: Task "jshint:files" failed. Use --force to continue.
Aborted due to warnings.

```

现在我们来纠正这些错误，并将文件恢复到之前的状态：

```js
var test = function() { 
    'use strict'; 
    console.log('test'); 
}; 
test(); 

```

在任何时间从命令行按 *Ctrl* + *C* 可以中止正在运行的 `watch` 任务，或者任何 Grunt 任务。

#### 定义默认的 Grunt 任务

Grunt 允许你定义一个 `default` 任务，当你没有参数地简单地输入 `grunt` 到命令行时，这个任务将会运行。为此，你将使用 `grunt.registerTask()` 方法：

```js
grunt.registerTask('default', ['jshint', 'watch:jshint']);

```

这个例子将默认的 Grunt 任务设置为首先运行定义的 `jshint` 任务，然后运行 `watch:jshint` 多任务目标。你可以看到传递给 `default` 任务的都是在数组中，所以你只需在命令行上简单地输入 `grunt`，就可以为 Grunt 设置运行任意数量的任务：

```js
$ grunt
Running "jshint:files" (jshint) task
>> 2 files lint free.
Running "watch:jshint" (watch) task
Waiting...

```

从输出中可以看出，`jshint` 任务最初运行了一次，然后运行 `watch:jshint` 来等待配置文件模式的额外更改。

#### 定义自定义任务

Grunt 允许你定义你自己的自定义任务，就像你定义默认任务一样。这样，你实际上可以直接在 `Gruntfile` 中编写你自己的自定义任务，或者你可以从外部文件加载它们，就像你处理 `grunt-contrib-jshint` 和 `grunt-contrib-watch` 一样。

**别名任务**

定义自定义任务的一种方式是简单地按照你想要它们运行的顺序调用一个或多个现有任务：

```js
grunt.registerTask('my-task', 'My custom task.', ['jshint']); 

```

在这个例子中，我们简单地定义了一个名为 `my-task` 的任务，作为 `jshint` 的代理。第二个参数是任务的可选描述，它必须是一个字符串。第三个参数，传递一个数组，在这个例子中只包括 `jshint`，必须始终是一个数组。你也可以省略描述参数，直接在那里传递你的任务数组。这种定义任务的方式被称为 *别名任务*。

**基本任务**

当你定义自定义 Grunt 任务时，你不仅限于调用配置中存在的其他任务，你还可以编写可以直接作为函数调用的 JavaScript 代码。这被称为*基本任务*：

```js
grunt.registerTask('my-task', 'My custom task.', function() { 
    grunt.log.writeln('This is my custom task.'); 
}); 

```

在这个例子中，我们只是将一个字符串写入任务的命令行输出。输出应该看起来像这样：

```js
$ grunt my-task
Running "my-task" task
This is my custom task.

```

让我们在此基础上扩展这个例子，向我们的基本任务函数传递一些参数，以及从函数内部访问这些参数：

```js
grunt.registerTask('my-task', 'My custom task.', function(arg1, arg2) { 
    grunt.log.writeln(this.name + ' output...'); 
    grunt.log.writeln('arg1: ' + arg1 + ', arg2: ' + arg2); 
}); 

```

你会注意到基本任务有一个属性`this.name`可用，它只是对任务名称的引用。为了从命令行调用基本任务并传递参数，你将在任务名称后使用冒号来定义每个参数的连续性。这种语法与运行多任务目标的语法类似；然而，在这种情况下，你正在传递任意参数：

```js
$ grunt my-task:1:2

```

运行此命令将输出以下内容：

```js
Running "my-task:1:2" (my-task) task
my-task output...
arg1: 1, arg2: 2

```

如果你没有传递参数给一个期望参数的任务，它将简单地将其解析为`undefined`：

```js
$ grunt my-task
Running "my-task" task
my-task output...
arg1: undefined, arg2: undefined
Done, without errors.

```

你也可以在自定义任务中调用其他任务：

```js
grunt.registerTask('foo', 'My custom task.', function() { 
    grunt.log.writeln('Now calling the jshint and watch tasks...'); 
    grunt.task.run('jshint', 'watch'); 
}); 

```

在这个例子中，我们创建了一个名为`foo`的任务，它定义了一个自定义函数，该函数调用现有的`jshint`和`watch`任务：

```js
$ grunt foo
Running "foo" task
Now calling the jshint and watch tasks...
Running "jshint:files" (jshint) task
>> 2 files lint free.
Running "watch" task
Waiting...

```

想要了解更多关于使用 Grunt 创建自定义任务的信息，请访问 gruntjs.com/creating-tasks。

这些任务的例子只是触及了 Grunt 所能做到的一小部分，但你应该能够从中领悟到它的强大之处，并开始思考在构建自己的 SPA 时，Grunt 任务可能实现的可能性。

# 摘要

现在你已经学会了如何使用 NPM 设置一个最佳的开发环境，使用 Bower 提供前端依赖，以及使用 Grunt 自动化开发任务，现在是时候开始学习更多关于构建真实应用程序的知识了。在下一章中，我们将深入探讨常见的 SPA 架构设计模式，它们的意义，以及根据你构建的 SPA 类型，最佳的设计模式是什么。
