# 第八章：构建自动化

我们看到如何使用 Jasmine 从头开始创建应用程序。然而，随着应用程序的增长和文件数量的增加，管理它们之间的依赖关系可能会变得有点困难。

例如，我们在 Investment 和 Stock 模型之间有一个依赖关系，它们必须按正确的顺序加载才能工作。因此，我们尽力而为；我们按照脚本的加载顺序进行排序，以便在加载 Investment 后 Stock 可用。下面是我们的做法：

```js
<script type="text/javascript" src="src/Stock.js"></"script>
<script type="text/javascript" src="src/Investment.js"></"script>
```

然而，这很快就会变得繁琐和难以管理。

另一个问题是应用程序用于加载所有文件的请求数量；一旦应用程序开始增长，这个数量可能会增加到数百个。

因此，我们在这里有一个悖论；尽管将其分解为小模块有利于代码的可维护性，但对于客户端性能来说却是不利的，单个文件更加可取。

在同一时间满足以下两个要求将是完美的：

+   在开发中，我们有一堆包含不同模块的小文件

+   在生产中，我们有一个包含所有这些模块内容的单个文件

显然，我们需要一些构建过程。有许多不同的方法可以用 JavaScript 实现这些目标，但我们将专注于**webpack**。

# 模块捆绑器 - webpack

Webpack 是由 Tobias Koppers 创建的模块捆绑器，用于帮助创建大型和模块化的前端 JavaScript 应用程序。

它与其他解决方案的主要区别在于它支持任何类型的模块系统（AMD 和 CommonJS）、语言（CoffeeScript、TypeScript 和 JSX）甚至通过加载器支持资产（图像和模板）。

你没看错，甚至包括图片；如果在 React 应用中，一切都是组件，在 webpack 项目中，一切都是模块。

它构建了所有资产的依赖图，在开发环境中为它们提供服务，并在生产环境中对它们进行优化。

## 模块定义

JavaScript 是一种基于 ECMA 脚本规范的语言，直到第 6 版，仍没有对模块的标准定义。这种缺乏正式标准导致了许多竞争的社区标准（AMD 和 CommonJS）和实现（RequireJS 和 browserify）。

现在，有一个标准可供遵循，但不幸的是，现代浏览器不支持它，那么我们应该使用哪种样式来编写我们的模块呢？

好消息是，通过转译器，我们可以今天就使用 ES6，这给了我们未来的优势。

一个流行的转译器是**Babel**（[`babeljs.io/`](http://babeljs.io/)），我们将通过一个加载器与 webpack 一起使用它。

我们将看到如何在 webpack 中使用它，但首先重要的是要理解什么是 ES6 模块。这是一个简单的定义，没有任何依赖：

```js
function MyModule () {};
export default MyModule;
```

让我们将其与我们到目前为止一直声明模块的方式进行比较。下一个示例显示了如果使用第三章中介绍的约定编写的代码将会是什么样子，*测试前端代码*：

```js
(function () {
  function MyModule() {};
  this.MyModule = MyModule;
}());
```

最大的区别在于缺少 IIFE。ES6 模块默认具有自己的作用域，因此不可能意外地污染全局命名空间。

第二个区别是模块值不再附加到全局对象上，而是作为默认模块值导出：

```js
function MyModule () {};
**export default MyModule;**

```

关于模块的依赖关系，到目前为止，一切都是全局可用的，因此我们将依赖项作为参数传递给 IIFE 模块，如下所示：

```js
(function (**$**) {
  function MyModule() {};
  this.MyModule = MyModule;
}(**jQuery**));
```

然而，一旦在项目中开始使用 ES6 模块，就不再有全局变量了。那么，如何将这些依赖项引入模块呢？

如果你还记得之前的内容，ES6 示例是通过`export default`语法导出模块值的。因此，给定一个模块有一个值，我们所要做的就是将其作为依赖项请求。让我们将 jQuery 依赖项添加到我们的 ES6 模块中：

```js
**import $ from 'jQuery';**
function MyModule () {};
export default MyModule;
```

这里，`$`代表依赖项将加载到的变量名，`jQuery`是文件名。

也可以导出多个值作为模块的结果，并将这些值导入到不同的变量中，但是在本书的范围内，默认值就足够了。

ES6 标准引入了许多不同的构造到 JavaScript 语言中，这些内容超出了本书的范围。更多信息，请查看 Babel 的优秀文档[`babeljs.io/docs/learn-es6/`](http://babeljs.io/docs/learn-es6/)。

## Webpack 项目设置

Webpack 可以作为一个 NPM 包使用，它的设置非常简单，将在接下来的章节中进行演示。

### 提示

重要的是要理解 NPM 和 Node.js 之间的区别。NPM 既是一个包管理器，也是一个包格式，而 Node.js 是 NPM 模块通常运行的平台。

### 使用 NPM 管理依赖关系

我们已经有了一个 Node.js 项目的雏形，但是在本章中我们将开始使用更多的依赖项，因此我们需要一个正式的定义，列出项目所依赖的所有 NPM 包。

为了将项目定义为一个 NPM 包，同时定义所有的依赖项，我们需要在应用程序的根文件夹中创建一个名为`package.json`的特殊文件。可以通过一个简单的命令轻松创建它：

```js
**npm init**

```

它将提示一系列关于项目的问题，所有这些问题都可以保持默认值。最后，你应该有一个类似以下输出的文件，具体取决于你的文件夹名称：

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

下一步是安装我们所有的依赖项，目前只有 express。

```js
**npm install --save express**

```

前面的命令不仅会安装 express，如第四章中所述，*异步测试 - AJAX*，还会将其添加为`package.json`文件的依赖项。在之前运行`npm init`命令时，我们得到了以下输出，显示了`dependencies`属性：

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
  **"dependencies": {**
 **"express": "⁴.12.0"**
 **}**
}
```

现在我们了解了如何管理项目的依赖关系，我们可以安装**webpack**和**Babel**作为开发依赖项，以开始打包我们的模块，如下所示：

```js
**npm install --save-dev babel-loader webpack webpack-dev-server**

```

最后一步是在`package.json`中添加一个脚本来启动开发服务器：

```js
"scripts": {
  "test": "echo \"Error: no test specified\" && exit 1",
  **"dev": "webpack-dev-server"**
}
```

这使我们可以通过一个简单的命令启动开发服务器：

```js
**npm run dev**

```

`webpack-dev-server`可执行文件的实际位置在`./node_modules/.bin`文件夹中。因此，`npm run dev`等同于：

```js
**./node_modules/.bin/webpack-dev-server**

```

这是因为当你运行`npm run <scriptName>`时，NPM 会将`./node_modules/.bin`文件夹添加到路径中。

### Webpack 配置

接下来，我们需要配置 webpack，让它知道要捆绑哪些文件。可以通过在项目的根文件夹中创建一个名为`webpack.config.js`的文件来实现。它的内容应该是：

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

关于这个配置文件有一些关键点：

+   `context`指令告诉 webpack 在`__dirname`中查找模块，意思是项目的根文件夹。

+   `entry`指令指定了应用程序的入口点。由于我们目前只是在进行测试，所以只有一个名为`spec`的入口点，它指向我们所有的规范文件。

+   `output.filename`指令用于指定每个入口点的文件名。`[name]`模式将在编译时被入口点名称替换。因此，`spec.js`实际上将包含我们所有的规范代码。

+   `module.loaders`最后一条指令告诉 webpack 如何处理不同的文件类型。我们在这里使用`babel-loader`参数来为我们的源文件添加对 ES6 模块和 JSX 语法的支持。`exclude`指令很重要，以免编译`node_modules`文件夹中的任何依赖项。

完成了这个设置后，你可以启动开发服务器，并在`http://localhost:8080/spec.js`上检查转译后的捆绑文件的样子（在配置文件中定义的文件名）。

此时，webpack 配置已经完成，我们可以开始适应 Jasmine 运行器以运行规范。

## 规范运行器

如前所述，我们正在使用 webpack 来编译和捆绑源文件，因此 Jasmine 规范将变得简单得多：

```js
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>Jasmine Spec Runner v2.1.3</title>

  <link rel="shortcut icon" type="image/png" href="lib/jasmine-2.1.3/jasmine_favicon.png">
  <link rel="stylesheet" href="lib/jasmine-2.1.3/jasmine.css">

  <script src="lib/jasmine-2.1.3/jasmine.js"></script>
  <script src="lib/jasmine-2.1.3/jasmine-html.js"></script>
  <script src="lib/jasmine-2.1.3/boot.js"></script>

  <script src="lib/jquery.js"></script>
  <script src="lib/jasmine-jquery.js"></script>

  <script src="lib/mock-ajax.js"></script>

  <script src="spec/SpecHelper.js"></script>

  **<script src="spec.js"></script>**
</head>
<body>
</body>
</html>
```

有一些要点：

首先，我们不再需要在前一章中解释的 JSX 转换器 hack；转换现在由 webpack 和 babel-loader 完成。因此，我们可以很好地使用默认的 Jasmine boot。

其次，我们选择将测试运行器依赖项保留为全局（Jasmine，Mock Ajax，Jasmine JQuery 和 Spec helper）。将它们保留为全局对于我们的测试运行器来说会简单得多，并且就模块化而言，我们不会伤害我们的代码。

此刻，尝试在`http://localhost:8080/SpecRunner.html`上运行测试应该会因缺少引用而产生许多失败。这是因为我们仍然需要将我们的规范和源转换为 ES6 模块。

## 测试一个模块

要能够运行所有测试，需要将所有源文件和规范文件转换为 ES6 模块。在规范中，这意味着将所有源模块添加为依赖项：

```js
**import Stock from '../src/Stock';**
describe("Stock", function() {
  // the original spec code
});
```

在源文件中，这意味着声明所有依赖项并导出其默认值，如下所示：

```js
**import React from 'react';**
var InvestmentListItem = React.createClass({
  // original code
});
**export default InvestmentListItem;**

```

一旦所有代码都转换完成，启动开发服务器并再次将浏览器指向运行器 URL 后，测试应该可以正常工作。

# 测试运行器：Karma

还记得我们在介绍中说过，我们可以执行 Jasmine 而不需要浏览器窗口吗？为此，我们将使用**PhantomJS**，一个可编写脚本的无头 WebKit 浏览器（驱动 Safari 浏览器的相同渲染引擎）和**Karma**，一个测试运行器。

设置非常简单；再次使用 NPM 安装一些依赖项：

```js
**npm install –save-dev karma karma-jasmine karma-webpack karma-phantomjs-launcher es5-shim**

```

这里唯一奇怪的依赖是`es5-shim`，它用于为 PhantomJS 提供对一些 ES5 功能的支持，而 PhantomJS 仍然缺少这些功能，React 需要。

下一步是在项目的根文件夹中创建一个名为`karma.conf.js`的配置文件，用于 Karma。

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
frameworks: [**'jasmine'**],
browsers: [**'PhantomJS'**],
```

通过加载`es5-shim`来修复 PhantomJS 上的浏览器兼容性问题，如下所示：

```js
// shim to workaroud PhantomJS 1.x lack of 'bind' support
// see: https://github.com/ariya/phantomjs/issues/10522
**'node_modules/es5-shim/es5-shim.js'**,
```

加载先前在`SpecRunner.html`文件中全局的测试运行器依赖项，如下所示：

```js
'lib/jquery.js',
'lib/jasmine-jquery.js',
'lib/mock-ajax.js',
'spec/SpecHelper.js',
```

最后，加载所有规范，如下所示：

```js
'spec/**/*Spec.*',
```

到目前为止，您可以删除`SpecRunner.html`文件，`webpack.config.js`文件中的规范条目以及`lib/jasmine-2.1.3`文件夹。

通过调用 Karma 来运行测试，在控制台中打印测试结果，如下所示：

```js
**./node_modules/karma/bin/karma start karma.conf.js**
**> investment-tracker@0.0.1 test /Users/paulo/Dropbox/jasmine_book/second_edition/book/chapter_8/code/webpack-karma**
**> ./node_modules/karma/bin/karma start karma.conf.js**
**INFO [karma]: Karma v0.12.31 server started at http://localhost:9876/**
**INFO [launcher]: Starting browser PhantomJS**
**INFO [PhantomJS 1.9.8 (Mac OS X)]: Connected on socket cGbcpcpaDgX14wdyzLZh with id 37309028**
**PhantomJS 1.9.8 (Mac OS X): Executed 36 of 36 SUCCESS (0.21 secs / 0.247 secs)**

```

为了更简单地运行测试，可以更改`package.json`项目文件并描述其测试脚本：

```js
"scripts": {
  **"test": "./node_modules/karma/bin/karma start karma.conf.js",**
  "dev": "webpack-dev-server"
},
```

然后，您可以通过简单调用以下命令来运行测试：

```js
**npm test**

```

# 快速反馈循环

自动化测试的关键在于快速反馈循环，因此想象一下能够在控制台中运行测试并在任何文件更改后刷新应用程序的浏览器。这可能吗？答案是肯定的！

## 观看并运行测试

通过在启动 Karma 时添加一个简单的参数，我们可以实现测试的极乐世界，如下所示：

```js
**./node_modules/karma/bin/karma start karma.conf.js --auto-watch --no-single-run**

```

自己尝试一下；运行此命令，更改文件，然后查看测试是否自动运行-就像魔术一样。

再次，我们不想记住这些复杂的命令，因此让我们向`package.json`文件添加另一个脚本：

```js
"scripts": {
  "test": "./node_modules/karma/bin/karma start karma.conf.js",
  **"watch-test": "./node_modules/karma/bin/karma start karma.conf.js --auto-watch --no-single-run",**
  "dev": "webpack-dev-server"
},
```

我们可以通过以下命令运行它：

```js
**npm run watch-test**

```

## 观看并更新浏览器

为了实现开发的极乐世界，我们也只差一个参数。

在启动开发服务器时，将以下内容添加到`package.json`文件中：

```js
./node_modules/.bin/webpack-dev-server **--inline –hot**

```

再次在浏览器上尝试一下；更改文本编辑器中的文件，浏览器应该会刷新。

还鼓励您使用这些新参数更新`package.json`文件，以便运行`npm run dev`可以获得“实时重新加载”的好处。

# 为生产进行优化

我们模块打包目标的最后一步是生成一个经过缩小并准备好生产的文件。

大部分配置已经完成，只缺少几个步骤。

第一步是为应用程序设置一个入口点，然后将一个启动所有内容的索引文件`index.js`放在`src`文件夹中，内容如下：

```js
import React from 'react';
import Application from './Application.jsx';

var mountNode = document.getElementById('application-container''');
React.render(React.createElement(Application, {}), mountNode);
```

我们在书中没有详细介绍这个文件的实现，所以一定要检查附加的源文件，以更好地理解它的工作原理。

在 webpack 配置文件中，我们需要添加一个输出路径来指示捆绑文件将放置在哪里，以及我们刚刚创建的新入口文件，如下所示：

```js
module.exports = {
  context: __dirname,
  **entry: {**
 **index: './src/index.js'**
 **},**

  output: {
    **path: 'dist',**
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

然后，剩下的就是在我们的`package.json`文件中创建一个构建任务：

```js
"scripts": {
    "test": "./node_modules/karma/bin/karma start karma.conf.js",
    "watch-test": "./node_modules/karma/bin/karma start karma.conf.js --auto-watch --no-single-run",
    **"build": "webpack -p",**
    "dev": "webpack-dev-server --inline --hot"
  },
```

运行它并检查`dist`文件夹中的构建文件，如下所示：

```js
**npm run build**

```

# 静态代码分析：JSHint

正如第一章所述，JavaScript 不是一种编译语言，但运行代码（如自动化测试）并不是检查错误的唯一方法。

一整类工具能够读取源文件，解释它们，并查找常见错误或不良实践，而无需实际运行源文件。

一个非常流行的工具是**JSHint**—一个简单的二进制文件，也可以通过 NPM 安装，如下所示：

```js
npm install --save-dev **jshint jsxhint**

```

您可以看到我们还安装了**JSXHint**，另一个用于执行 JSX 文件的静态分析的工具。它基本上是原始 JSHint 的包装器，同时执行 JSX 转换。

如果你还记得上一章，JSXTransformer 不会改变行号，所以 JavaScript 文件中给定行号上的警告将在原始 JSX 文件中的相同行号上。

执行它们非常简单，如下所示：

```js
./node_modules/.bin/jshint .
./node_modules/.bin/jsxhint .
```

然而，每当我们运行测试时，让它们运行也是一个好主意：

```js
"scripts": {
    "start": "node bin/server.js",
    "test": **"./node_modules/.bin/jshint . && ./node_modules/.bin/jsxhint . &&** ./node_modules/karma/bin/karma start karma.conf.js",
    "watch-test": "./node_modules/karma/bin/karma start karma.conf.js --auto-watch --no-single-run",
    "build": "webpack -p",
    "dev": "webpack-dev-server --inline --hot"
  },
```

最后一步是配置我们希望 JSHint 和 JSXHint 捕获的错误。再次，在项目的根文件夹中创建另一个配置文件，这次名为`.jshintrc`：

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

这是一个启用或禁用的选项标志列表，其中最重要的是以下内容：

+   `esnext`：此标志告诉我们我们正在使用 ES6 版本

+   `unused`：此标志会在任何未使用的声明变量上中断

+   `undef`：此选项标志会在使用未声明的变量时中断

还有一个`globals`变量列表，用于测试，以防止由于`undef`标志而出现错误。

前往 JSHint 网站[`jshint.com/docs/options/`](http://jshint.com/docs/options/)查看完整的选项列表。

唯一缺少的步骤是防止 linter 在其他人的代码（Jasmine，React 等）中运行。这可以通过简单地创建一个文件来实现，该文件应包含应忽略的文件夹。这个名为`.jshintignore`的文件应包含：

+   `node_modules`

+   `lib`

现在运行静态分析和所有测试就像这样简单：

```js
**npm test**

```

# 持续集成-Travis-CI

我们已经为项目创建了大量自动化，这对于团队中新开发人员的入职非常有帮助；运行测试只需两个命令：

```js
**npm install**
**npm test**

```

然而，这不是唯一的优势；我们可以在持续集成环境中通过这两个命令运行测试。

为了演示可能的设置，我们将使用 Travis-CI（[`travis-ci.org`](https://travis-ci.org)），这是一个免费的开源项目解决方案。

在我们开始之前，需要您拥有一个 GitHub（[`github.com/`](https://github.com/)）帐户，并且项目已经托管在那里。我希望您已经熟悉 git（[`www.git-scm.com/`](http://www.git-scm.com/)）和 GitHub。

一旦准备好，我们就可以开始 Travis-CI 的设置。

## 添加项目到 Travis-CI

在我们可以为项目添加 Travis-CI 支持之前，首先需要将项目添加到 Travis-CI。

转到 Travis-CI 网站[`travis-ci.org`](https://travis-ci.org)，然后点击右上角的**使用 GitHub 登录**。

输入您的 GitHub 凭据，一旦您登录，它应该显示您所有存储库的列表：

如果您的存储库没有显示出来，您可以点击**立即同步**按钮，让 Travis-CI 更新列表。

一旦您的存储库出现，点击开关启用它。这将在您的 GitHub 项目上设置钩子，因此 Travis-CI 会在对存储库进行任何更改时收到通知。

## 项目设置

设置 Travis-CI 项目再简单不过了。因为我们的构建过程和测试都已经脚本化，我们所要做的就是告诉 Travis-CI 应该使用什么运行时。

Travis-CI 知道 Node.js 项目依赖项是通过`npm install`安装的，并且测试是通过`npm test`运行的，因此没有额外的步骤来运行我们的测试。

在项目根目录创建一个名为`.travis.yml`的新文件，并将 Travis 的语言配置为 Node.js：

```js
language: node_js
```

就是这样。

使用 Travis-CI 的步骤非常简单，将这些概念应用到其他持续集成环境，比如 Jenkins（[`jenkins-ci.org/`](http://jenkins-ci.org/)）应该也很简单。

# 总结

在本章中，我希望向您展示自动化的力量，以及我们如何使用脚本来使生活更轻松。您了解了 webpack 以及它如何用于管理模块之间的依赖关系，并帮助您生成生产代码（打包和压缩）。

静态代码分析的力量帮助我们在代码运行之前甚至找到错误。

您还看到了如何在无头模式下甚至自动运行您的规范，让您始终专注于代码编辑器。

最后，我们已经看到了使用持续集成环境是多么简单，以及我们如何使用这个强大的概念来保持我们的项目始终得到测试。
