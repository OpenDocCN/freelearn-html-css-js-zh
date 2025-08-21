# 第三章：安装 Zombie.js 和 Mocha

在本章结束时，您应该能够为使用 Zombie.js 和 Mocha 的应用程序设置测试环境的基本结构。

本章涵盖的主题有：

+   在应用程序清单中设置 Zombie.js 和 Mocha 包

+   设置测试环境

+   运行你的第一个测试

# 更改应用程序清单

现在，您将扩展上一章开始构建的待办事项应用程序，并开始为其提供自我测试的能力。

在应用程序的根目录中，有一个名为`package.json`的文件，您已经修改过，引入了一些应用程序依赖的模块。现在，您需要添加一个新的部分，指定在开发和测试阶段对其他模块的依赖关系。这个部分名为`devDependencies`，只有在`NODE_ENV`环境变量没有设置为`production`时，NPM 才会安装它。这是一个很好的地方，可以介绍那些需要在运行测试时存在的模块的依赖关系。

首先，您需要添加`mocha`和`zombie`模块：

```js
{
  "description": "To-do App",
  "version": "0.0.0",
  "private": true, 
  "dependencies": {
    "union": "0.3.0",
    "flatiron": "0.2.8",
    "plates": "0.4.x",
    "node-static": "0.6.0",
    "nano": "3.3.0",
    "flatware-cookie-parser": "0.1.x",
    "flatware-session": "0.1.x"
  },
 "devDependencies": {
 "mocha": "1.4.x",
 "zombie": "1.4.x"
 },
  "scripts": {
    "test": "vows --spec",
    "start": "node app.js"
  },
  "name": "todo",
  "author": "Pedro",
  "homepage": ""
}
```

然后，您需要使用 NPM 安装这些缺失的依赖项：

```js
$ npm install
...
mocha@1.4.2 node_modules/mocha
...

zombie@1.4.1 node_modules/zombie
...
```

这将在`node_modules`文件夹中安装这两个模块及其内部依赖项，使它们随时可用于您的应用程序。

# 设置测试环境

现在，您需要设置一个测试脚本。首先，您将测试用户注册流程。

但在此之前，为了能够在测试中启动我们的服务器，我们需要对`app.js`文件进行轻微修改：

```js
var flatiron = require('flatiron'),
    path = require('path'),
    nstatic = require('node-static'),
    app = flatiron.app;

app.config.file({ file: path.join(__dirname, 'config', 'config.json') });

var file = new nstatic.Server(__dirname + '/public/');

app.use(flatiron.plugins.http, {
  before: [
    require('flatware-method-override')(),
    require('flatware-cookie-parser')(),
    require('flatware-session')(),
    function(req, res) {
      var found = app.router.dispatch(req, res);
      if (! found) {
        file.serve(req, res);
      }
    }
  ]
});

app.router.path('/users', require('./routes/users'));
app.router.path('/session', require('./routes/session'));
app.router.path('/todos', require('./routes/todos'));

module.exports = app;

if (process.mainModule === module) {
 app.start(3000);
}

```

我们的测试将使用它们自己的服务器，所以在这种情况下，我们不需要`app.js`来为我们运行服务器。最后几行代码导出了应用程序，并且只有在主模块（使用`node`命令行调用的模块）是`app.js`时才启动服务器。由于测试将有一个不同的主模块，所以在运行测试时服务器不会启动。

现在，作为第一个例子，我们将测试获取用户注册表单。我们将把所有与用户路由相关的测试都集中在`test/users.js`文件中。这个文件可以从以下内容开始：

```js
var assert  = require('assert'),
    Browser = require('zombie'),
    app     = require('../app')
    ;

before(function(done) {
  app.start(3000, done);
});

after(function(done) {
  app.server.close(done);
});

describe('Users', function() {

  describe('Signup Form', function() {

    it('should load the signup form', function(done) {
      var browser = new Browser();
      browser.visit("http://localhost:3000/users/new", function() {
        assert.ok(browser.success, 'page loaded');
        done();
      });
    });

  });
});
```

在前面的代码中，我们在顶部包含了`assert`模块（用于验证应用程序是否按预期运行）、`zombie`模块（赋值给`Browser`变量）和`app`模块。`app`模块获取了 Flatiron 应用程序对象，因此您可以启动和停止相应的服务器。

接下来，我们声明，在运行任何测试之前，应该启动应用程序，并且在所有测试完成后，应该关闭服务器。

接下来是一系列嵌套的`describe`调用。这些调用用于为每个测试提供上下文，允许您稍后区分在每个测试之前和之后将发生的设置和拆卸函数。

然后是一个`it`语句，您在其中实现测试。这个语句接受两个参数，即正在测试的主题的描述和在开始测试时将被调用的函数。这个函数得到一个回调函数`done`，在测试完成时调用。这种安排使得异步测试成为可能且可靠。每个测试只有在相应的`done`函数被调用后才结束，这可能是在一系列异步 I/O 调用之后。

然后我们开始创建一个浏览器，并加载用户注册表单的 URL，使用`assert.ok`函数来验证页面是否成功加载。`assert`模块是 Node.js 的核心模块，提供基本的断言测试。在测试代码中，我们放置一些断言来验证一些值是否符合我们的预期。如果任何断言失败，`assert`会抛出一个错误，测试运行器会捕获到这个错误，表示测试失败。

除了基本的`assert.ok`函数之外，如果值不为 true（即通过`x == true`测试），它将抛出错误，该模块还提供了一组辅助函数，以提供更复杂的比较，如`assert.deepEqual`等。有关`assert`模块的更多信息，您可以阅读[`nodejs.org/api/assert.html`](http://nodejs.org/api/assert.html)上的 API 文档。

现在我们需要通过替换`package.json`中 Flatiron 提供的默认值来指定测试命令脚本：

```js
  "scripts": {
 "test": "mocha test/users.js",
    "start": "node app.js"
  },...
```

这指定了当告诉 NPM 运行测试时，NPM 应该执行什么操作。要运行测试，请在命令行上输入以下命令：

```js
$ npm test
```

输出应该是成功的：

```js
...
> mocha test/users.js

  .

  ✔ 1 test complete (284ms)
```

# 摘要

要安装 Mocha 和 Zombie，你需要将它们作为开发依赖项包含在应用程序清单中，然后使用 NPM 安装它们。

一旦这些模块安装好，你可以在名为`test`的目录中为应用程序的每个逻辑组件创建测试文件。每个文件应包含一系列测试，每个测试都应嵌套在`describe`语句中。

您还应该修改应用程序清单，以指定测试脚本，以便可以使用 NPM 运行测试。

在接下来的章节中，我们将不断完善这个测试，并引入一些新的测试，以覆盖我们应用程序的更多使用情况。
