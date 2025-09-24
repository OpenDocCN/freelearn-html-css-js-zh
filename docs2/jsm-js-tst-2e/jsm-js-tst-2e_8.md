# 第八章. 构建自动化

我们看到了如何使用 Jasmine 测试从头开始创建一个应用程序。然而，随着应用程序的增长和文件数量的增加，管理它们之间的依赖关系可能会变得有些困难。

例如，投资模型和股票模型之间存在依赖关系，它们必须按正确的顺序加载才能工作。因此，我们尽我们所能；我们按顺序加载脚本，以便在投资加载后股票可用。下面是如何做到这一点的示例：

```js
<script type="text/javascript" src="img/Stock.js"></"script>
<script type="text/javascript" src="img/Investment.js"></"script>
```

然而，这很快就会变得繁琐且难以管理。

另一个问题是在加载所有文件时应用程序使用的请求数量；一旦应用程序开始增长，它可能达到数百。

因此，我们在这里遇到了一个悖论；虽然将代码拆分成小模块对于代码可维护性是有好处的，但对于客户端性能来说却是不利的，因为单个文件更受欢迎。

一个完美的世界是同时满足以下两个要求：

+   在开发过程中，我们有一堆包含不同模块的小文件

+   在生产环境中，我们有一个包含所有这些模块内容的单个文件

显然，我们需要某种构建过程。使用 JavaScript 实现这些目标有几种不同的方法，但我们将专注于**webpack**。

# 模块打包器 – webpack

Webpack 是由 Tobias Koppers 创建的一个模块打包器，旨在帮助创建大型和模块化的前端 JavaScript 应用程序。

它与其他解决方案的主要区别在于它支持任何类型的模块系统（AMD 和 CommonJS），语言（CoffeeScript、TypeScript 和 JSX）以及通过加载器甚至资产（图像和模板）。

你没有看错，即使是图像；如果在 React 应用程序中，一切都是组件，在 webpack 项目中，一切都是模块。

它构建了所有资产的依赖关系图，在开发环境中提供服务，并针对生产环境进行优化。

## 模块定义

JavaScript 是一种基于 ECMA Script 规范的编程语言，直到版本 6 之前，还没有模块的标准定义。这种缺乏正式标准导致了多个竞争性的社区标准（AMD 和 CommonJS）和实现（RequireJS 和 browserify）。

现在，有一个标准可以遵循，但不幸的是，在现代浏览器中并没有对其的支持，所以我们应该使用哪种风格来编写我们的模块？

好消息是，现在可以通过转译器使用 ES6，这给我们带来了未来保障的优势。

一个流行的转译器是**Babel**([`babeljs.io/`](http://babeljs.io/))，我们将通过加载器与 webpack 一起使用。

我们稍后会看到如何与 webpack 一起使用它，但首先重要的是要理解是什么让 ES6 模块成为可能。下面是一个没有依赖关系的简单定义：

```js
function MyModule () {};
export default MyModule;
```

让我们将其与我们之前声明模块的方式进行比较。下一个示例显示了如果使用第三章（第三章）中提出的约定编写的代码将会是什么样子：

```js
(function () {
  function MyModule() {};
  this.MyModule = MyModule;
}());
```

最大的区别是缺少 IIFE。ES6 模块默认具有自己的作用域，因此不可能意外地污染全局命名空间。

第二个区别是模块值不再附加到全局对象上，而是作为默认模块值导出：

```js
function MyModule () {};
export default MyModule;

```

关于模块的依赖项，到目前为止，所有内容都是全局可用的，因此我们将依赖项作为 IIFE（立即执行函数表达式）的参数传递给模块，如下所示：

```js
(function ($) {
  function MyModule() {};
  this.MyModule = MyModule;
}(jQuery));
```

然而，随着你在项目中开始使用 ES6 模块，将不再有全局变量。那么，你如何将这些依赖项放入模块中？

如果你还记得之前的内容，ES6 示例是通过 `export default` 语法导出模块值的。所以，给定一个模块有一个值，我们只需要将其作为依赖项请求。让我们将 jQuery 依赖项添加到我们的 ES6 模块中：

```js
import $ from 'jQuery';
function MyModule () {};
export default MyModule;
```

在这里，`$` 代表将加载依赖项的变量名称，而 `jQuery` 是文件名。

还可以将多个值作为模块的结果导出，并将这些值导入不同的变量中，但就本书的范围而言，默认值就足够了。

ES6 标准向 JavaScript 语言引入了许多不同的结构，这些结构也超出了本书的范围。有关更多信息，请参阅 Babel 的优秀文档[`babeljs.io/docs/learn-es6/`](http://babeljs.io/docs/learn-es6/)。

## Webpack 项目设置

Webpack 作为 NPM 包可用，其设置非常简单，将在下一节中演示。

### 小贴士

理解 NPM 和 Node.js 之间的区别很重要。NPM 既是包管理器也是包格式，而 Node.js 是一个平台，NPM 模块通常在其中运行。

### 使用 NPM 管理依赖项

我们已经得到了一个 Node.js 项目的雏形，但随着我们在本章中开始使用更多依赖项，我们需要正式定义项目所依赖的所有 NPM 包。

要将项目定义为 NPM 包，同时包含其所有依赖项，我们需要在应用程序的根目录中创建一个特殊文件 `package.json`。它可以通过单个命令轻松创建：

```js
npm init

```

它将提示一系列关于项目的问题，所有这些问题都可以保留它们的默认值。最后，你应该有一个包含类似以下内容的文件，具体取决于你的文件夹名称：

```js
{
  "name": "jasmine-testing-project",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts":" {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
  Is this ok? (Yes)
```

下一步是安装所有依赖项，目前只有 express。

```js
npm install --save express

```

之前的命令不仅会安装第四章中描述的 express，*异步测试 - AJAX*，还会将其添加到`package.json`文件的依赖项中。运行`npm init`命令（如之前所述）后，我们得到以下输出，显示了`dependencies`属性：

```js
{
  "name": "jasmine-testing-project",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
 "express": "⁴.12.0"
 }
}
```

现在我们已经了解了如何管理我们项目的依赖关系，我们可以将**webpack**和**Babel**作为开发依赖项安装，以开始捆绑我们的模块，如下所示：

```js
npm install --save-dev babel-loader webpack webpack-dev-server

```

最后一步是在`package.json`中添加一个脚本来启动开发服务器：

```js
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1",
  "dev": "webpack-dev-server"
}
```

这允许我们使用简单的命令启动开发服务器：

```js
npm run dev

```

`webpack-dev-server`可执行文件的实际位置在`./node_modules/.bin`文件夹中。因此，`npm run dev`与以下命令相同：

```js
./node_modules/.bin/webpack-dev-server

```

这之所以有效，是因为当你运行`npm run <scriptName>`时，NPM 会将`./node_modules/.bin`文件夹添加到路径中。

### Webpack 配置

接下来，我们需要配置 webpack，使其知道要捆绑哪些文件。这可以通过在项目的根目录下创建一个`webpack.config.js`文件来实现。其内容应该是：

```js
module.exports = {
  context: __dirname,
  entry: {
    spec: [
      './spec/StockSpec.js',
      './spec/InvestmentSpec.js',
      './spec/components/NewInvestmentSpec.jsx',
      './spec/components/InvestmentListItemSpec.jsx',
      './spec/components/InvestmentListSpec.jsx'
    ]
  },

  output: {
    filename: '[name].js'
  },

  module: {
    loaders: [
      {
        test: /(\.js)|(\.jsx)$/,
        exclude: /node_modules/,
        loader: 'babel-loader'
      }
    ]
  }
};
```

关于此配置文件有几个要点：

+   `context`指令告诉 webpack 在`__dirname`中查找模块，这意味着项目的根目录。

+   `entry`指令指定了应用程序的入口点。由于我们目前只进行测试，因此有一个名为`spec`的单个入口点，它引用了所有规范文件。

+   `output.filename`指令用于告知每个入口点的文件名。在编译过程中，`[name]`模式将被入口点名称替换。因此，`spec.js`实际上将包含我们所有的规范代码。

+   `module.loaders`的最终指令告诉 webpack 如何处理不同类型的文件。在这里，我们使用`babel-loader`参数来为源文件添加对 ES6 模块和 JSX 语法的支持。`exclude`指令很重要，以免浪费编译`node_modules`文件夹中的任何依赖。

完成此设置后，您可以通过访问`http://localhost:8080/spec.js`（配置文件中定义的文件名）来检查转换后的捆绑包的外观。

到目前为止，webpack 配置已经完成，我们可以开始将 Jasmine 运行器适配以运行规范。

## 规范运行器

如前所述，我们使用 webpack 来编译和捆绑源文件，因此 Jasmine 规范将变得非常简单：

```js
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>Jasmine Spec Runner v2.1.3</title>

  <link rel="shortcut icon" type="image/png" href="lib/jasmine-2.1.3/jasmine_favicon.png">
  <link rel="stylesheet" href="lib/jasmine-2.1.3/jasmine.css">

  <script src="img/jasmine.js"></script>
  <script src="img/jasmine-html.js"></script>
  <script src="img/boot.js"></script>

  <script src="img/jquery.js"></script>
  <script src="img/jasmine-jquery.js"></script>

  <script src="img/mock-ajax.js"></script>

  <script src="img/SpecHelper.js"></script>

  <script src="img/spec.js"></script>
</head>
<body>
</body>
</html>
```

有几点需要注意：

首先，我们不再需要上一章中解释的 JSX 转换器黑客技巧；转换现在由 webpack 和 babel-loader 完成。因此，我们可以很好地使用默认的 Jasmine 引导。

其次，我们选择将测试运行器的依赖项作为全局（Jasmine、Mock Ajax、Jasmine JQuery 和 Spec helper）。将它们设置为全局使我们的测试运行器变得更加简单，并且从模块化的角度来看，我们不会伤害我们的代码。

在此时，尝试在`http://localhost:8080/SpecRunner.html`运行测试应该会因为缺少引用而产生很多失败。这是因为我们仍然需要将我们的 spec 和源文件转换为 ES6 模块。

## 测试一个模块

要运行所有测试，需要将所有源文件和 spec 文件转换为 ES6 模块。在 spec 中，这意味着添加所有源模块作为依赖项：

```js
import Stock from '../src/Stock';
describe("Stock", function() {
  // the original spec code
});
```

在源文件中，这意味着声明所有依赖项以及导出其默认值，如下所示：

```js
import React from 'react';
var InvestmentListItem = React.createClass({
  // original code
});
export default InvestmentListItem;

```

一旦所有代码都转换为，在启动开发服务器并将浏览器再次指向运行器 URL 时，测试应该会工作。

# 测试运行器：Karma

记得我们在介绍中提到过，我们可以执行 Jasmine 而不需要浏览器窗口？为了做到这一点，我们将使用**PhantomJS**，一个可脚本化的无头 WebKit 浏览器（与 Safari 浏览器相同的渲染引擎）和**Karma**，一个测试运行器。

设置非常简单；使用 NPM，我们再次安装一些依赖项：

```js
npm install –save-dev karma karma-jasmine karma-webpack karma-phantomjs-launcher es5-shim

```

这里唯一的奇怪依赖是`es5-shim`，它用于为 PhantomJS 提供一些它仍然缺少的 ES5 功能，而 React 需要。

下一步是创建一个配置文件，命名为`karma.conf.js`，位于项目的根目录：

```js
module.exports = function(config) {
  config.set({
    basePath: '.',

    frameworks: ['jasmine'],
    browsers: ['PhantomJS'],

    files: [
      // shim to workaroud PhantomJS 1.x lack of 'bind' support
      // see: https://github.com/ariya/phantomjs/issues/10522
      'node_modules/es5-shim/es5-shim.js',
      'lib/jquery.js',
      'lib/jasmine-jquery.js',
      'lib/mock-ajax.js',
      'spec/SpecHelper.js',
      'spec/**/*Spec.*'
    ],

    preprocessors: {
      'spec/**/*Spec.*': ['webpack']
    },

    webpack: require('./webpack.config.js'),
    webpackServer: { noInfo: true },
    singleRun: true
  });
};
```

在其中，我们设置了 Jasmine 框架和 PhantomJS 浏览器：

```js
frameworks: ['jasmine'],
browsers: ['PhantomJS'],
```

通过加载`es5-shim`来修复 PhantomJS 上的浏览器兼容性问题，如下代码所示：

```js
// shim to workaroud PhantomJS 1.x lack of 'bind' support
// see: https://github.com/ariya/phantomjs/issues/10522
'node_modules/es5-shim/es5-shim.js',
```

加载测试运行器依赖项，这些依赖项之前在`SpecRunner.html`文件中是全局的，如下代码所示：

```js
'lib/jquery.js',
'lib/jasmine-jquery.js',
'lib/mock-ajax.js',
'spec/SpecHelper.js',
```

最后，加载所有 spec，如下所示：

```js
'spec/**/*Spec.*',
```

到目前为止，你可以删除`SpecRunner.html`文件，`webpack.config.js`文件中的 spec 条目，以及`lib/jasmine-2.1.3`文件夹。

通过调用 Karma 来运行测试，它将在控制台打印测试结果，如下所示：

```js
./node_modules/karma/bin/karma start karma.conf.js
> investment-tracker@0.0.1 test /Users/paulo/Dropbox/jasmine_book/second_edition/book/chapter_8/code/webpack-karma
> ./node_modules/karma/bin/karma start karma.conf.js
INFO [karma]: Karma v0.12.31 server started at http://localhost:9876/
INFO [launcher]: Starting browser PhantomJS
INFO [PhantomJS 1.9.8 (Mac OS X)]: Connected on socket cGbcpcpaDgX14wdyzLZh with id 37309028
PhantomJS 1.9.8 (Mac OS X): Executed 36 of 36 SUCCESS (0.21 secs / 0.247 secs)

```

为了简化测试的运行，我们可以更改`package.json`项目文件并描述其测试脚本：

```js
"scripts": {
  "test": "./node_modules/karma/bin/karma start karma.conf.js",
  "dev": "webpack-dev-server"
},
```

你可以通过简单地调用以下命令来运行测试：

```js
npm test

```

# 快速反馈循环

自动化测试的核心是快速反馈循环，所以想象一下，在文件更改后，测试能在控制台运行，而应用在浏览器上刷新，这会是可能的吗？答案是肯定的！

## 监视并运行测试

通过在启动 Karma 时使用一个简单的参数，我们可以实现测试的极乐境界，如下所示：

```js
./node_modules/karma/bin/karma start karma.conf.js --auto-watch --no-single-run

```

亲自尝试；运行此命令，更改一个文件，看看测试是否自动运行——就像魔法一样。

再次，我们不想记住这些复杂的命令，所以让我们在`package.json`文件中添加另一个脚本：

```js
"scripts": {
  "test": "./node_modules/karma/bin/karma start karma.conf.js",
  "watch-test": "./node_modules/karma/bin/karma start karma.conf.js --auto-watch --no-single-run",
  "dev": "webpack-dev-server"
},
```

我们可以通过以下命令来运行它：

```js
npm run watch-test

```

## 监视并更新浏览器

为了达到开发上的极乐境界，我们只需一个参数即可。

在启动开发服务器时，将以下内容添加到`package.json`文件中：

```js
./node_modules/.bin/webpack-dev-server --inline –hot

```

再次尝试在你的浏览器上运行；在文本编辑器中更改一个文件，浏览器应该刷新。

你还被鼓励更新`package.json`文件，以便运行`npm run dev`时获得“实时重新加载”的好处。

# 优化生产环境

我们模块打包器目标的最后一步是生成一个压缩并准备好生产的文件。

大部分的配置已经完成，只需再进行几个步骤。

第一步是为应用程序设置一个入口点，然后创建一个启动所有内容的索引文件，`index.js`，需要放置在`src`文件夹内，内容如下：

```js
import React from 'react';
import Application from './Application.jsx';

var mountNode = document.getElementById('application-container''');
React.render(React.createElement(Application, {}), mountNode);
```

我们在书中没有详细讨论此文件的实现，所以请确保检查附带的源文件以更好地理解其工作原理。

在 webpack 配置文件中，我们需要添加一个输出路径来指示捆绑文件将放置的位置，以及我们刚刚创建的新入口文件，如下所示：

```js
module.exports = {
  context: __dirname,
  entry: {
 index: './src/index.js'
 },

  output: {
    path: 'dist',
    filename: '[name]-[hash].js'
  },

  module: {
    loaders: [
      {
        test: /(\.js)|(\.jsx)$/,
        exclude: /node_modules/,
        loader: 'babel-loader'
      }
    ]
  }
};
```

然后，剩下的就是在我们`package.json`文件中创建一个构建任务：

```js
"scripts": {
    "test": "./node_modules/karma/bin/karma start karma.conf.js",
    "watch-test": "./node_modules/karma/bin/karma start karma.conf.js --auto-watch --no-single-run",
    "build": "webpack -p",
    "dev": "webpack-dev-server --inline --hot"
  },
```

运行它，并检查构建文件是否已放入`dist`文件夹，如下所示：

```js
npm run build

```

# 静态代码分析：JSHint

如第一章所述，JavaScript 不是一种编译型语言，但运行代码（如自动化测试的情况）并不是检查错误的唯一方式。

一类工具能够读取源文件，解释它们，并在不实际运行源文件的情况下查找常见的错误或不良实践。

一个非常流行的工具是**JSHint**——一个可以通过 NPM 安装的简单二进制文件，如下所示：

```js
npm install --save-dev jshint jsxhint

```

你可以看到我们还在安装**JSXHint**，这是另一个用于执行 JSX 文件静态分析的工具。它基本上是在执行 JSX 转换的同时，围绕原始 JSHint 的一个包装器。

如果你记得上一章的内容，JSXTransformer 不会改变行号，所以 JavaScript 文件中给定行号的警告将在原始 JSX 文件中的相同行号处。

执行它们非常简单，如下所示：

```js
./node_modules/.bin/jshint .
./node_modules/.bin/jsxhint .
```

然而，在运行测试时让它们运行也是一个好主意：

```js
"scripts": {
    "start": "node bin/server.js",
    "test": "./node_modules/.bin/jshint . && ./node_modules/.bin/jsxhint . && ./node_modules/karma/bin/karma start karma.conf.js",
    "watch-test": "./node_modules/karma/bin/karma start karma.conf.js --auto-watch --no-single-run",
    "build": "webpack -p",
    "dev": "webpack-dev-server --inline --hot"
  },
```

最后一步是配置我们希望 JSHint 和 JSXHint 捕获的错误。再次，我们在项目的根目录中创建另一个配置文件，这次叫做`.jshintrc`：

```js
{
  "esnext": true,
  "undef": true,
  "unused": true,
  "indent": 2,
  "noempty": true,
  "browser": true,
  "node": true,
  "globals": {
    "jasmine": false,
    "spyOn": false,
    "describe": false,
    "beforeEach": false,
    "afterEach": false,
    "expect": false,
    "it": false,
    "xit": false,
    "setFixtures": false
  }
}
```

这是一个选项标志列表，要么启用要么禁用，其中最重要的是以下内容：

+   `esnext`：此标志告诉我们我们正在使用 ES6 版本

+   `unused`：此标志会在任何未使用的声明变量上断言

+   `undef`：此选项标志会在任何未声明的变量被使用时断言

此外，还有一个由测试使用的`globals`变量列表，用于防止由于`undef`标志导致的错误。

前往 JSHint 网站 [`jshint.com/docs/options/`](http://jshint.com/docs/options/) 查看选项的完整列表。

唯一缺少的步骤是防止在别人的代码中运行 linter（Jasmine、React 等等）。这可以通过简单地创建一个包含它应该忽略的文件夹的文件来实现。这个名为 `.jshintignore` 的文件应该包含：

+   `node_modules`

+   `lib`

现在运行静态分析和所有测试就像这样简单：

```js
npm test

```

# 持续集成 – Travis-CI

我们在项目周围创建了很多自动化，这对团队中引入新开发者非常有用；运行测试只需两个命令：

```js
npm install
npm test

```

然而，这并不是唯一的好处；我们可以在持续集成环境中通过这两个命令运行测试。

为了演示一个可能的设置，我们将使用 Travis-CI ([`travis-ci.org`](https://travis-ci.org))，这是一个开源项目的免费解决方案。

在我们开始之前，你需要有一个 GitHub ([`github.com/`](https://github.com/)) 账户，并且项目已经托管在那里。我预计你已经熟悉 git ([`www.git-scm.com/`](http://www.git-scm.com/)) 和 GitHub。

一旦你准备好了，我们就可以开始 Travis-CI 的设置了。

## 将项目添加到 Travis-CI

在我们能够将 Travis-CI 支持添加到项目中之前，首先我们需要将项目添加到 Travis-CI。

前往 Travis-CI 网站 [`travis-ci.org`](https://travis-ci.org)，然后在右上角点击 **Sign in with GitHub**。

输入你的 GitHub 凭据，一旦你登录，它应该会显示包含你所有仓库的列表：

如果你的仓库没有显示出来，你可以点击右上角的 **Sync Now** 按钮让 Travis-CI 更新列表。

一旦你的仓库出现，通过点击开关来启用它。这将在你 GitHub 项目上设置钩子，这样 Travis-CI 就会在仓库中任何更改推送时收到通知。

## 项目设置

设置 Travis-CI 项目非常简单。由于我们的构建过程和测试都已脚本化，我们只需要告诉 Travis-CI 它应该使用什么运行时。

Travis-CI 知道 Node.js 项目依赖是通过 `npm install` 安装的，测试是通过 `npm test` 运行的，因此没有额外的步骤来运行我们的测试。

在项目根目录下创建一个名为 `.travis.yml` 的新文件，并将 Travis 的语言配置为 Node.js：

```js
language: node_js
```

就这些了。

使用 Travis-CI 的步骤相当直接，将这些相同的概念应用到其他持续集成环境中，如 Jenkins ([`jenkins-ci.org/`](http://jenkins-ci.org/))，应该相当简单。

# 摘要

在本章中，我希望已经向你展示了自动化的力量以及我们如何可以使用脚本使我们的生活变得更轻松。你学习了 webpack 及其如何被用来管理模块之间的依赖关系，并帮助你生成生产代码（打包和压缩）。

静态代码分析在帮助我们甚至在代码运行之前发现错误方面的力量。

你还看到了如何无头运行你的规范，甚至可以自动运行，让你可以一直专注于代码编辑器。

最后，我们看到了使用持续集成环境是多么简单，以及我们如何可以使用这个强大的概念来确保我们的项目始终处于测试状态。
