# 第四章：介绍 Node.js 模块

现在我们已经熟悉了 JavaScript 语言的语法，我们可以开始构建我们的应用程序。为此，我们需要知道如何构建我们的应用程序，使其能够以可维护的方式增长。

在本章中，我们将涵盖以下主题：

+   使用模块结构 JavaScript 代码

+   声明和使用我们自己的模块

+   将模块组织到文件和目录中

+   实现 Express 中间件模块

# 组织你的代码库

大多数编程平台都提供了几种结构代码的机制。考虑 C#/.NET 或 Java：你可以使用类、命名空间或包，以及编译单元（程序集或 JAR/WAR 文件）。注意从小规模的组织单元（类）到大规模的组织单元（程序集）的范围。这允许你通过在每个细节级别提供秩序来使代码库更易于接近。

经典的基于浏览器的 JavaScript 开发相当无结构。函数是唯一内置的语言特性，用于组织你的代码。你可以将你的代码分割成单独的脚本文件，但这些文件都在网页的相同全局上下文中共享。

随着时间的推移，人们已经发展出了组织 JavaScript 代码的方法。现在的标准做法是使用**模块**。JavaScript 有几种不同的模块系统可用，但它们的工作方式相似。每个模块系统都包括以下方面：

+   一种通过名称和其自己的作用域声明模块的方法

+   定义模块提供功能的方法

+   将模块导入另一个脚本的方法

在每个系统中，当你导入一个模块时，你会得到一个普通的 JavaScript 对象，你可以将其分配给一个变量。对于大多数模块，这将是一个包含多个属性和函数的对象。但它可以是任何有效的 JavaScript 对象，例如，一个单独的函数。

大多数模块系统都期望或至少鼓励你将每个模块定义在单独的文件中，就像在其他语言中使用类一样。大型模块通常由其他更小的模块组成。这些模块将组合在一起放在同一个目录下。这样，模块更像命名空间或包。

模块的灵活性意味着你可以使用它们以不同的规模来结构你的代码。JavaScript 中缺乏内置的组织单元层次结构提供了更多的灵活性。这也迫使你更多地思考如何结构你的代码。

## JavaScript 模块系统

ECMAScript 2015 将模块作为语言的一个内置特性引入。虽然这已经是一种常见的做法，但对于客户端编程来说，这种做法一直依赖于使用第三方库来提供模块系统。

你可能见过 RequireJS，它提供了一种使用函数来定义模块的方法。RequireJS 使用纯 JavaScript 并在任何环境中工作。它在浏览器中最有用，因为可以通过互联网加载额外的模块。RequireJS 解决了动态和异步加载额外脚本的一些问题。

Node.js 环境有其自己的模块系统，我们将在本章的剩余部分探讨。它利用文件系统来组织模块。

### 小贴士

你可能会遇到 **AMD** 或 **CommonJS** 这样的术语。这些是定义模块的标准。RequireJS 是 AMD 的一个实现，而 Node.js 模块遵循 CommonJS 标准。ECMAScript 2015 模块定义了一个新的标准，具有新的 `export` 和 `import` 语言关键字。虽然语法与本书中我们将使用的 Node.js 模块系统非常相似，但两者之间切换也很容易。

# 在 Node.js 中创建模块

我们实际上已经使用了一些 Node.js 模块并创建了一些自己的。让我们再次从 第二章，*Node.js 入门*，回顾我们的应用程序。

以下代码来自 `routes/index.js` 和 `routes/users.js`：

```js
module.exports = router;
```

以下是从 `app.js` 的代码：

```js
var express = require('express');
var path = require('path');
var favicon = require('serve-favicon');
var logger = require('morgan');
var cookieParser = require('cookie-parser');
var bodyParser = require('body-parser');

var routes = require('./routes/index');
var users = require('./routes/users');
```

我们的每个路由（index 和 users）都是一个模块。它们通过 Node.js 定义的内置 `module` 对象暴露其功能，该对象是每个模块的变量作用域。在前面的例子中，我们每个路由模块提供的对象是一个 Express 路由实例。`app.js` 脚本使用内置的 `require` 函数导入这些模块。

注意到 `app.js` 也使用 `require` 导入了各种 `npm` 包。请注意，它使用文件路径来引用我们自己的模块，而 `npm` 模块则通过名称引用。

让我们看看 Node.js 模块是如何满足 JavaScript 模块功能的三个方面的。

## 声明一个具有名称和自身作用域的模块

在 Node.js 中，每个独立的 JavaScript 文件自动被视为一个新的模块。与加载到网页中的脚本不同，每个文件都有自己的作用域。模块的名称是文件的名称。

## 定义模块提供的功能

Node.js 为从模块中导出功能提供了两个内置变量。这些是 `module.exports` 和 `exports`。`module.exports` 被初始化为一个空对象。`exports` 只是 `module.exports` 的引用。它等同于以下内容出现在你的脚本之前：

```js
var exports = module.exports = {};
```

在你的脚本末尾包含在 `module.exports` 变量中的任何内容都是你模块的导出值。当你的模块在其他地方导入时，这将返回。以下都是等效的：

```js
module.exports.foo = 1;
module.exports.bar = 2;

module.exports = { foo: 1, bar: 2 };

exports.foo = 1;
exports.bar = 2;
```

注意，以下内容与之前的示例不同。它只是重新分配了 `exports`，但完全没有改变 `module.exports`：

```js
exports = { foo: 1, bar: 2 };
```

## 将模块导入到另一个脚本中

Node.js 提供了另一个用于导入模块的内置变量。这就是我们在本章前面 `app.js` 中看到的 `require` 函数。这个函数由 Node.js 提供，始终可用。它接受一个参数，即你想要导入的模块的名称或路径。以下是从 `app.js` 中摘录的内容，展示了如何通过名称加载第三方模块，以及通过文件路径加载我们自己的模块：

```js
var express = require('express');
...
var routes = require('./routes/index');
```

注意，我们不需要为我们的模块指定 `.js` 文件扩展名。Node.js 会自动为我们添加这个扩展名。

# 定义目录级模块

如本章开头所述，模块也可以更像命名空间。我们可以将整个目录视为一个模块，由单个文件中的较小模块组成。最简单的方法是在目录中创建一个 `index.js` 文件。

当调用 `require('./directoryName')` 时，Node.js 将尝试加载一个名为 `'./directoryName/index.js'` 的文件（相对于当前脚本）。`index.js` 本身并没有什么特殊之处。这只是一个暴露模块入口点的另一个脚本文件。如果 `directoryName` 包含一个 `package.json` 文件，Node.js 将首先加载这个文件，并查看是否指定了一个 `main` 脚本，在这种情况下，Node.js 将加载这个脚本而不是寻找 `index.js`。

要导入本地模块，我们使用文件或目录路径，即以 `'/'`、`'../'` 或 `'./'` 开头，就像前面的例子一样。如果我们用纯字符串调用 `require`，Node.js 会将其视为相对于 `node_modules` 文件夹。`npm` 包只是这个文件夹下的目录级模块。我们将在后面的章节中更详细地探讨定义我们自己的 `npm` 包。

# 实现一个 Express 中间件模块

让我们回到我们在 第二章 中开始的 Node.js 应用程序，*Node.js 入门*。我们将编写一个应用程序，用户可以为彼此设置谜题。首先，我们需要一种识别当前用户的方法。我们将在大多数请求中这样做，使其成为一个跨领域关注点。这是一个中间件的好用例。

现在，我们将以最简单的方式实现用户，只需在 cookie 中存储一个 ID。我们将在后面的章节中探讨更健壮的识别方法。然而，请注意，我们使用中间件意味着我们可以很容易地稍后更改我们的方法。这个关注点封装在我们的用户中间件中，所以我们只需要在一个地方更改它。

首先，我们需要一种生成唯一 ID 的方法。为此，我们将使用 npm 中的 UUID 模块。我们可以在命令行上运行以下命令将其添加到我们的项目中：

```js
> npm install uuid --save

```

`--save` 标志将此模块的名称存储在我们的 `package.json` 文件中，以便它可以通过 `npm install` 自动安装。这对于从源代码的干净检出中恢复我们的应用程序非常有用（回想一下，人们通常会将 `node_modules` 目录排除在源代码控制之外，正是因为它可以通过这种方式轻松恢复）。

现在我们已经准备好创建我们的中间件，它将被放置在 `middleware/users.js` 下：

```js
'use strict';

const uuid = require('uuid');

module.exports = function(req, res, next) {
    let userId = req.cookies.userId;
    if (!userId) {
        userId = uuid.v4();
        res.cookie('userId', userId);
    }
    req.user = {
        id: userId
    };
    next();
};
```

注意，我们使用 ES2015 的 `const` 关键字来引用 `uuid` 模块，因为这个引用永远不会改变。但我们使用 `let` 关键字来引用 `userId` 变量，因为这个变量可以被重新赋值。另外，注意我们调用 `next()` 而不是返回响应，这样下一个中间件就可以继续处理请求。

最后，我们需要将此中间件添加到我们的应用程序中的 `app.js` 文件：

```js
var users = require('./middleware/users');
var routes = require('./routes/index');
var app = express();

...

app.use(users);
app.use('/', routes);

...
```

注意，这替换了为我们生成的 `./routes/users` 模块的导入和使用。这个路由并不特别有用，但我们很快就会添加更多路由。

我们可以通过修改我们的索引路由和视图来检查我们的中间件是否工作：

```js
routes/index.jsrouter.get('/', function(req, res, next) {
 res.render('index', { title: 'Welcome', userId: req.user.id });
});
```

以下为 `views/index.hjs` 的代码：

```js
  <body>
 <h1>{{ title }}</h1>
 <p>Your user ID is {{ userId }}.</p>
  </body>
```

启动应用程序并访问 `http://localhost:3000/`。你应该看到一个随机生成的用户 ID。刷新页面，你应该保留相同的 ID。在不同的浏览器（或隐身/私密浏览窗口）中打开网站。这个单独的浏览器会话应该看到不同的 ID。

# 摘要

在本章中，我们看到了如何使用 Node.js 模块来结构化我们的代码库，以及如何创建一个 Express 中间件模块来实现跨切面关注点。

现在我们有了结构化我们的代码库的方法和识别用户的方法，我们可以继续实现我们应用程序的功能。在下一章中，我们将开始向我们的应用程序添加一些交互性。
