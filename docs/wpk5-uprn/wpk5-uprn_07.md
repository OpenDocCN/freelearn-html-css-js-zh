# 第七章：调试和迁移

本章将进一步探讨迁移和调试，提供对这些主题的广泛概述和详细检查。

迁移是指将内容和项目从 Webpack 的早期版本迁移到较新版本的过程。我们将特别关注从 Webpack 3 版本迁移到 4 版本以及从 4 版本迁移到 5 版本的过程。我们还将介绍如何处理已弃用的插件以及如何删除或更新它们。这将包括在使用 Node.js v4 和 CLI 时的迁移。

本章将讨论`resolve`方法以及`module.loaders`现已被`module.rules`方法取代。它还将涵盖加载器的链接，包括不再需要或已删除的加载器的链接。

然后，本章将继续探讨调试。调试涉及消除复杂软件系统中出现的常见故障和错误的过程。本章将解释常见问题及其解决方案、故障排除、避免这些问题的最佳实践以及如何找到故障。

本章涵盖的主题如下：

+   调试

+   热模块替换

+   添加实用程序

+   迁移

# 调试

调试工具对于工作流程至关重要，特别是在贡献核心复制、编写加载器或任何其他复杂形式的编码时。本指南将带您了解在解决诸如性能缓慢或不可原谅的回溯等问题时最有用的实用工具。

+   通过 Node.js 和 CLI 提供的`stats`数据

+   通过`node-nightly`和最新的 Node.js 版本使用 Chrome DevTools

在 Webpack 5 中，截至撰写本文时，存在一些已知问题；例如，DevTools 不支持持久缓存和包含绝对路径的持久缓存文件尚不可移植。

在调试构建问题、手动筛选数据或使用工具时，`stats`数据非常有用。它可用于查找以下内容：

+   构建错误和警告

+   每个模块的内容

+   模块编译和解析统计

+   模块之间的相互关系

+   任何给定块中包含的模块

此外，官方的 Webpack 分析工具将接受这些数据并为您可视化。

有时，当控制台语句无法完成工作时，需要更强大的解决方案。

正如前端开发人员社区中普遍认为的那样，Chrome DevTools 在调试应用程序时是不可或缺的，但它并不止步于此。从 Node.js v6.3.0+开始，开发人员可以使用内置的检查标志在 DevTools 中调试 Node.js 程序。

这个简短的演示将利用`node-nightly`包，该包提供了最新的检查功能。这提供了创建断点、调试内存使用问题、在控制台中公开对象等功能：

1.  首先全局安装`node-nightly`包：

```js
npm install --global node-nightly
```

1.  现在必须使用命令行来运行此包以完成安装：

```js
node-nightly
```

1.  现在，使用`node-nightly`的`inspect`标志功能，我们可以开始调试任何 Webpack 项目。需要注意的是，现在无法运行 npm 脚本；相反，需要表达完整的`node_module`路径：

```js
node-nightly --inspect ./node_modules/webpack/bin/webpack.js
```

1.  输出应该在命令行实用程序窗口中显示如下内容：

```js
Debugger listening on ws://127.0.0.1:9229/c624201a-250f-416e-a018-300bbec7be2c For help see https://nodejs.org/en/docs/inspector
```

1.  现在，转到 Chrome 的检查功能（`chrome://inspect`），任何活动脚本现在应该在`远程目标`标题下可见。

单击每个脚本下的“检查”链接将在会话中为节点打开专用的调试器或 DevTools 链接，这将自动连接。请注意，NiM 是 Chrome 的一个方便的扩展，每次进行检查时都会自动在新标签中打开 DevTools。这对于较长的项目非常有用。

还可以使用`inspect-brk`标志，它会在任何脚本的第一个语句上中断，允许查看源代码，设置断点，并临时停止和启动进程。这也允许程序员继续向所讨论的脚本传递参数；这可能对进行并行配置更改很有用。

所有这些都与一个关键功能有关，这也是本指南中先前提到的令人兴奋的**热模块替换**（HMR）主题。下一节将介绍它是什么以及如何使用它，以及教程。

# 热模块替换

HMR 可能是 Webpack 中最有用的元素。它允许运行时更新需要完全刷新的模块。本节将探讨 HMR 的实现，以及它的工作原理和为什么它如此有用。

非常重要的一点是，HMR 不适用于生产模式，也不应该在生产模式下使用；它只应该在开发模式下使用。

值得注意的是，根据开发人员的说法，插件的内部 HMR API 在将来的 Webpack 5 更新中可能会发生变化。

要启用 HMR，我们首先需要更新我们的`webpack-dev-server`配置，并使用 Webpack 内置的 HMR 插件。这个功能对提高生产力非常有用。

删除`print.js`的入口点也是一个好主意，因为它现在将被`index.js`模块使用。

任何使用`webpack-dev-middleware`而不是`webpack-dev-server`的人现在应该使用`webpack-hot-middleware`包来启用 HMR：

1.  要开始使用 HMR，我们需要返回到配置文件`webpack.config.js`。按照这里的修改：

```js
  const path = require('path');
  const HtmlWebpackPlugin = require('html-webpack-plugin');
  const CleanWebpackPlugin = require('clean-webpack-plugin');
  const webpack = require('webpack');

  module.exports = {
    entry: {
       app: './src/index.js',
       print: './src/print.js'
       app: './src/index.js'
    },
    devtool: 'inline-source-map',
    devServer: {
      contentBase: './dist',
 hot: true
    },
    plugins: [
      // new CleanWebpackPlugin(['dist/*']) for < v2 versions of 
      // CleanWebpackPlugin
      new CleanWebpackPlugin(),
      new HtmlWebpackPlugin({
        title: 'Hot Module Replacement'
      }),
      new webpack.HotModuleReplacementPlugin()
    ],
    output: {
      filename: '[name].bundle.js',
      path: path.resolve(__dirname, 'dist')
    }
  };
```

请注意前面代码中的添加——`hot:`选项设置为`true`，并添加了`'Hot Module Replacement'`插件，以及在 HMR 配置中创建新的 Webpack 插件。所有这些都应该被做来使用插件和 HMR。

1.  可以使用命令行修改`webpack-dev-server`配置，命令如下：

```js
webpack-dev-server --hot 
```

这将允许对捆绑应用程序进行临时更改。

1.  `index.js`现在应该更新，以便当检测到`print.js`的更改时，Webpack 可以接受更新的模块。更改在以下示例中以粗体显示;我们只是用`import`表达式和函数暴露`print.js`文件：

```js
  import _ from 'lodash';
 import printMe from './print.js';

  function component() {
    const element = document.createElement('div');
    const btn = document.createElement('button');

    element.innerHTML = _.join(['Hello', 'Webpack'], ' ');

    btn.innerHTML = 'Click me and check the console!';
    btn.onclick = printMe;

    element.appendChild(btn);

    return element;
  }

  document.body.appendChild(component());

 if (module.hot) {
    module.hot.accept('./print.js', function() {
      console.log('Accepting the updated printMe module');
      printMe();
```

```js
    })
  }
```

如果您更改`print.js`中的控制台日志，则将在浏览器控制台中看到以下输出。现在，强制性的`printMe()`按钮已经消失，但稍后可以更新：

```js
  export default function printMe() {
    console.log('This got called from print.js!');
    console.log('Updating print.js...')
  }
```

查看控制台窗口应该会显示以下输出：

```js
[HMR] Waiting for update signal from WDS...
main.js:4395 [WDS] Hot Module Replacement enabled.
  2main.js:4395 [WDS] App updated. Recompiling...
  main.js:4395 [WDS] App hot update...
  main.js:4330 [HMR] Checking for updates on the server...
  main.js:10024 Accepting the updated printMe module!
  0.4b8ee77….hot-update.js:10 Updating print.js...
  main.js:4330 [HMR] Updated modules:
  main.js:4330 [HMR]  - 20
```

前面的代码块显示 HMR 正在等待来自 Webpack 的信号，如果发生 HMR，命令行实用程序可以执行自动捆绑修订。当命令行窗口保持打开时，它也会显示这一点。Node.js 有一个类似的 API 可以使用。

# 在使用 Node.js API 时使用 DevServer

在使用**DevServer**和 Node.js API 时，不应将`dev server`选项放在 Webpack 配置对象上；而应始终将其作为第二参数传递。

在这里，DevServer 只是指在开发模式下使用 Webpack，而不是`watching`或`production`模式。要在 Node.js API 中使用 DevServer，请按照以下步骤进行操作：

1.  该函数放在`webpack.config.js`文件中，如下：

```js
new WebpackDevServer(compiler, options)
```

要启用 HMR，首先必须修改配置对象以包括 HMR 入口点。`webpack-dev-server`包包括一个名为`addDevServerEntryPoints`的方法，可以用来执行此操作。

1.  接下来是使用`dev-server.js`的简短示例：

```js
const webpackDevServer = require('webpack-dev-server');
const webpack = require('webpack');

  const config = require('./webpack.config.js');
  const options = {
    contentBase: './dist',
    hot: true,
    host: 'localhost'
};

webpackDevServer.addDevServerEntrypoints(config, options);
const compiler = webpack(config);
const server = new webpackDevServer(compiler, options);

server.listen(5000, 'localhost', () => {
  console.log('dev server listening on port 5000');
});
```

HMR 可能很困难。为了证明这一点，在我们的示例中，单击在示例网页中创建的按钮。显然控制台正在打印旧函数。这是因为事件处理程序绑定到原始函数。

1.  为了解决这个问题以便与 HMR 一起使用，必须使用`module.hot.accept`更新绑定到新函数。请参阅以下示例，使用`index.js`：

```js
  import _ from 'lodash';
  import printMe from './print.js';

  function component() {
    const element = document.createElement('div');
    const btn = document.createElement('button');

    element.innerHTML = _.join(['Hello', 'Webpack'], ' ');

    btn.innerHTML = 'Click me and view the console!';
    btn.onclick = printMe;  // onclick event is bind to the 
                            // original printMe function

    element.appendChild(btn);

    return element;
  }

  document.body.appendChild(component());
 let element = component(); // Store the element to re-render on 
                             // print.js changes
  document.body.appendChild(element);

  if (module.hot) {
    module.hot.accept('./print.js', function() {
      console.log('Accepting the updated printMe module!');
      printMe();
      document.body.removeChild(element);
 element = component(); 
      document.body.appendChild(element);
    })
  }
```

通过解释，`btn.onclick = printMe;`是绑定到原始`printMe`函数的`onclick`事件。`let element = component();`将存储元素以便在`print.js`发生任何更改时重新渲染。还要注意`element - component();`语句，它将重新渲染组件并更新单击处理程序。

这只是您可能会遇到的陷阱的一个例子。幸运的是，Webpack 提供了许多加载程序，其中一些稍后将讨论，这使得 HMR 变得不那么棘手。现在让我们来看看 HMR 和样式表。

# HMR 和样式表

使用`style-loader`可以更轻松地使用 HMR 处理 CSS。此加载程序使用`module.hot.accept`在更新 CSS 依赖项时修补样式标签。

在我们的实际示例的下一个阶段，我们将采取以下步骤：

1.  首先，使用以下命令安装两个加载程序：

```js
npm install --save-dev style-loader css-loader
```

1.  接下来，更新配置文件`webpack.config.js`以使用这些加载程序：

```js
  const path = require('path');
  const HtmlWebpackPlugin = require('html-webpack-plugin');
  const CleanWebpackPlugin = require('clean-webpack-plugin');
  const webpack = require('webpack');

  module.exports = {
    entry: {
      app: './src/index.js'
    },
    devtool: 'inline-source-map',
    devServer: {
      contentBase: './dist',
      hot: true
    },
    module: {
      rules: [
        {
 test: /\.css$/,
          use: ['style-loader', 'css-loader']
        }
      ]
    },
    plugins: [
      // new CleanWebpackPlugin(['dist/*']) for < v2 versions of 
        // CleanWebpackPlugin
      new CleanWebpackPlugin(),
      new HtmlWebpackPlugin({
        title: 'Hot Module Replacement'
      }),
      new webpack.HotModuleReplacementPlugin()
    ],
    output: {
      filename: '[name].bundle.js',
      path: path.resolve(__dirname, 'dist')
    }
  };
```

热加载样式表就像将它们导入模块一样简单，您可以从前面的配置示例和接下来的目录结构示例中看到。

1.  确保按照以下结构组织项目文件和目录：

```js
 webpack5-demo
  | - package.json
  | - webpack.config.js
  | - /dist
    | - bundle.js
  | - /src
    | - index.js
    | - print.js
    | - styles.css
```

1.  通过向样式表添加`body`样式来将文档主体的背景与其关联的蓝色进行样式化。使用`styles.css`文件执行此操作：

```js
body {
  background: blue;
}
```

1.  之后，我们需要确保内容正确加载到`index.js`文件中，如下所示：

```js
  import _ from 'lodash';
  import printMe from './print.js';
 import './styles.css'; 
  function component() {
    const element = document.createElement('div');
    const btn = document.createElement('button');

    element.innerHTML = _.join(['Hello', 'Webpack'], ' ');

    btn.innerHTML = 'Click me and check the console!';
    btn.onclick = printMe;  // onclick event is bind to the 
                            // original printMe function

    element.appendChild(btn);

    return element;
  }

  let element = component();
  document.body.appendChild(element);

  if (module.hot) {
    module.hot.accept('./print.js', function() {
      console.log('Accepting the updated printMe module!');
      document.body.removeChild(element);
      element = component(); // Re-render the "component" to update 
                             // the click handler
      document.body.appendChild(element);
    })
  }
```

现在，当`body`标签背景类的样式更改为红色时，颜色变化应立即注意到，无需刷新页面，表明了热编码的实时性。

1.  现在，您应该使用`styles.css`对背景进行这些更改：

```js
  body {
    background: blue;
    background: red;
  }
```

这以一种非常简单的方式演示了如何进行实时代码编辑。这只是一个简单的例子，但它是一个很好的介绍。现在，让我们进一步探讨一些更棘手的内容——加载程序和框架。

# 其他加载程序和框架

我们已经介绍了许多可用的加载程序，这些加载程序使 HMR 与各种框架和库更加顺畅地交互。其中一些更有用的加载程序在这里进行了描述：

+   **Angular HMR**：只需对主`NgModule`文件进行简单更改，即可完全控制 HMR API（不需要加载程序）。

+   **React Hot Loader**：此加载程序可以实时调整 React 组件。

+   **Elm Hot Webpack Loader**：此加载程序支持 Elm 编程语言的 HMR。

+   **Vue Loader**：此加载程序支持 Vue 组件的 HMR。

我们已经讨论了 HMR 和相关加载程序和框架，但有一件事我们尚未讨论——与我们迄今为止涵盖的内容相关的添加实用程序。我们将在接下来的部分中深入了解。

# 添加实用程序

在这种情况下，实用程序意味着负责一组相关功能的文件或模块，旨在优化、分析、配置或维护。这与应用程序形成对比，后者倾向于执行直接面向用户的任务或一组任务。因此，在这种情况下，您可以将实用程序视为前端的一部分，但它被隐藏在后台用于后台任务。

首先，在示例项目中添加一个实用程序文件。在`src/math.js`中执行此操作，以便导出两个函数：

1.  第一步将是组织项目目录：

```js
webpack5-demo
|- package.json
|- webpack.config.js
|- /dist
  |- bundle.js
  |- index.html
|- /src
  |- index.js
  |- math.js
|- /node_modules
```

**项目树**显示了您的文件和文件夹的外观，您将注意到其中一些新的添加，例如`math.js`。

1.  现在让我们更仔细地看一下`math.js`的编码：

```js
export function square(x) {
  return x * x;
}

export function cube(x) {
  return x * x * x;
}
```

您会发现它们是简单易导出的函数；它们将在稍后显现。

1.  还要确保在配置文件`webpack.config.js`中将 Webpack 模式设置为`development`，以确保捆绑包不被最小化：

```js
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  }
  },
  mode: 'development',
  optimization: {
    usedExports: true
  }
};
```

1.  有了这个，接下来更新入口脚本以利用其中一种新方法，并为简单起见删除`lodash`。这是使用`src/index.js`文件完成的：

```js
  import _ from 'lodash';
  import { cube } from './math.js';

  function component() {
    const element = document.createElement('div');
    const element = document.createElement('pre');

    // Lodash, now imported by this script
    element.innerHTML = _.join(['Hello', 'webpack'], ' ');
    element.innerHTML = [
      'Hello Webpack!',
      '5 cubed is equal to ' + cube(5)
    ].join('\n\n');

    return element;
  }

  document.body.appendChild(component());
```

从上面的例子中，我们可以看到`square`方法没有从`src/math.js`模块中导入。这个函数可以被视为死代码，基本上是一个未使用的导出，可以被删除。

1.  现在，再次运行`npm`构建以检查结果：

```js
npm run build
```

1.  完成后，找到`dist/bundle.js`文件——它应该在第 90-100 行左右。搜索文件以查找类似以下示例的代码，以便按照此过程进行：

```js
/* 1 */
/***/ (function(module, __webpack_exports__, __webpack_require__) {
  'use strict';
  /* unused harmony export square */
  /* harmony export (immutable) */ __webpack_exports__['a'] = cube;
  function square(x) {
    return x * x;
  }

  function cube(x) {
    return x * x * x;
  }
});
```

在这个例子中，现在会看到一个`unused harmony export square`的注释。请注意，它没有被导入。但是，它目前仍然包含在捆绑包中。

1.  ECMA 脚本并不完美，因此重要的是向 Webpack 的编译器提供关于代码纯度的提示。`packages.json`属性将有助于处理这些副作用：

```js
{
  "name": "your-project",
  "sideEffects": false
}
```

上述代码不包含副作用；因此，应将该属性标记为`false`，以指示 Webpack 删除未使用的导出。

在这种情况下，副作用被定义为在导入时执行特殊行为的脚本，而不是暴露多个导出等。一个例子是影响全局项目并且通常不提供导出的**polyfills**。

1.  如果代码具有副作用，可以提供一个数组作为补救措施，例如以下示例：

```js
{
  "name": "your-project",
  "sideEffects": [
    "./src/some-side-effectful-file.js"
  ]
}
```

此示例中的数组接受相对和绝对模式。

1.  请注意，任何导入的文件都会受到摇树的影响。例如，如果使用`CSS-loader`导入 CSS 文件，则必须将其添加到副作用列表中，以防止在生产模式下意外删除它：

```js
{
  "name": "your-project",
  "sideEffects": [
    "./src/some-side-effectful-file.js",
    "*.css"
  ]
}
```

1.  最后，`sideEffects`也可以从`module.rules`配置选项中设置。因此，我们使用`import`和`export`语法排队将死代码删除，但我们仍然需要从捆绑包中删除它。要做到这一点，将`mode`配置选项设置为`production`。这是通过以下方式在配置文件**`webpack.config.js`**中完成的：

```js
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  mode: 'development',
  optimization: {
    usedExports: true
  }
  mode: 'production'
};
```

`--optimize-minimize`标志也可以用于启用`TerserPlugin`。现在我们已经了解了这一点，可以再次运行`npm`构建。

现在清楚地看到整个捆绑包已经被最小化和混淆。仔细观察会发现`square`函数已经消失；取而代之的是一个混淆的 cube 函数：

```js
function r(e){return e*e*e}n.a=r
```

通过最小化和摇树，我们的捆绑包现在小了几个字节！虽然在这个假设的例子中可能看起来不多，但在处理具有复杂依赖关系树的大型应用程序时，摇树可以显著减少捆绑包大小。

`ModuleConcatenationPlugin`是摇树功能的必需品。它通过使用`mode: "production"`来添加。如果您没有使用它，请记得手动添加`ModuleConcatenationPlugin`。

必须完成以下任务才能充分利用摇树功能：

+   使用 ES2015 模块语法（即`import`和`export`）。

+   确保没有编译器将您的 ECMAScript 语法转换为 CommonJS 模块。

+   在`package.json`文件中添加一个`sideEffects`属性。

+   使用`production`配置选项来启用各种优化，包括摇树和最小化。

在进行摇树时，通常有助于将您的应用程序视为一棵树。在这个类比中，源代码和库将分别是绿叶和树的活部分。然而，死代码将代表枯叶。摇动这棵树将删除现在无效的代码。

这在迁移时尤其重要，值得考虑。考虑到 Webpack 版本之间的代码弃用变化，重要的是在尝试任何类似操作之前使软件达到最佳状态。这将防止出现非常棘手的错误。

# 迁移

迁移涉及从一个 Webpack 版本迁移到另一个版本。这通常涉及升级到最新版本。作为 Web 开发人员，你可能已经知道在处理其他软件时会有一些棘手的问题，所以这一部分可能是一个重要的部分，也许在未来的开发过程中你可以参考一下。

为了提供更详细的指南，包括了从 Webpack 3.0 迁移到 Webpack 4.0 的迁移策略，所以让我们在继续到版本 5 之前先进行一下这部分。

# 从版本 3 迁移到版本 4 的先决条件

在我们开始从 Webpack 版本 3 迁移到 4 的项目之前，有几个先决条件需要澄清：

+   Node.js

+   命令行

+   插件

+   模式

对于使用 Node.js 版本 4 或更低版本的开发人员，升级到 Node.js 版本 6 或更高版本是必要的。在命令行方面，CLI 已经移动到一个名为`webpack-cli`的单独包中。在使用 Webpack 4 之前，你需要安装它。

在更新插件时，许多第三方插件需要升级到最新版本以确保兼容，所以请注意这一点。浏览项目以找到需要更新的插件也是一个好主意。另外，请确保将新的模式选项添加到你的配置中：

1.  首先在你的配置中将模式设置为`production`或`development`，根据配置类型而定，如下面的代码片段所示，使用`webpack.config.js`：

```js
module.exports = {
    mode: 'production',
}
```

还有一种替代方法，可以通过 CLI 传递模式，就像下面的例子一样：

```js
--mode production
```

前面的例子展示了通过命令行在`production`模式下进行 Webpack 命令的后半部分。下面的例子展示了在`development`模式下的相同情况：

```js
--mode development
```

1.  下一步是移除弃用的插件；这些插件应该从你的配置文件中移除，因为它们在生产模式下是默认的。下面的例子将向你展示如何在`webpack.config.js`中进行编辑：

```js
module.exports = {
 plugins: [
   new NoEmitOnErrorsPlugin(),
   new ModuleConcatenationPlugin(),
   new DefinePlugin({ "process.env.NODE_ENV":
     JSON.stringify("production") })
   new UglifyJsPlugin()
 ],
}
```

下面的例子让你看到了这在开发模式下是如何工作的。请注意，插件在开发模式下是默认的，同样使用`webpack.config.js`：

```js
module.exports = {
 plugins: [
   new NamedModulesPlugin()
 ],
}
```

1.  如果一切都做对了，你会发现已经移除了弃用的插件。你的配置文件**`webpack.config.js`**应该看起来像下面这样：

```js
module.exports = {
 plugins: [
   new NoErrorsPlugin(),
   new NewWatchingPlugin()
 ],
}
```

此外，在 Webpack 4.0 中，`CommonChunkPlugin`已被移除，并提供了`optimization.splitChunks`选项作为替代。

如果你正在从统计数据生成 HTML，现在可以使用`optimization.splitChunks.chunks: "all"`——这在大多数情况下是最佳配置。

关于`import()`和 CommonJS 还有一些工作要做。在 Webpack 4 中，使用`import()`加载任何非 ESM 脚本的结果已经发生了变化：

1.  现在，你需要访问默认属性来获取`module.exports`的值。在这里查看`non-esm.js`文件，看看它是如何运作的：

```js
module.exports = {
      sayHello: () => {
       console.log('Hello World');
     }
 };
```

这是一个简单的 JavaScript 函数，你可以复制它的内容来跟随演示并查看结果的变化。

1.  下一个文件是一个**`example.js`**文件。它可以被命名为任何你想要的名字，你可以执行任何你想要的操作。在这个例子中，它是一个简单的`sayHello();`函数：

```js
function sayHello() {
 import('./non-esm.js').then(module => {
  module.default.sayHello();
 });
}
```

这些代码块展示了如何使用 CommonJS 编写简单的函数。你应该将这种约定应用到你现有的代码中，以确保它不会出错。

1.  当使用自定义加载器转换`.json`文件时，现在需要在`webpack.config.js`中更改模块类型：

```js
module.exports = {
 rules: [
  {
    test: /config\.json$/,
    loader: 'special-loader',
    type: 'javascript/auto',
    options: {...}
  }
 ]
};
```

1.  即使使用`json-loader`，也可以将其移除；参见下面的例子：

```js
module.exports = {
  rules: [
   {
     test: /\.json$/,
     loader: 'json-loader'
   }
  ]
};
```

完成所有必需的迁移先决条件后，下一步是内置在 Webpack 中的自动更新过程。

# 从版本 4 迁移到版本 5 时的先决条件

本指南旨在帮助您在直接使用 Webpack 时迁移到 Webpack 5。如果您使用更高级的工具来运行 Webpack，请参考该工具的迁移说明。

如第一章中所解释的，*Webpack 5 简介*，Webpack 5 需要 Node.js 版本 10.13.0（LTS）才能运行；但是，使用更新版本可以进一步提高构建性能：

1.  在升级到主要版本时，应确保检查相关插件和加载程序的个别迁移说明，特别是通过作者提供的副本。在这种情况下，在构建过程中注意弃用警告。您可以通过这种方式调用 Webpack 来获取弃用警告的堆栈跟踪，以找出哪些插件和加载程序负责。Webpack 5 将删除所有已弃用的功能。要继续，构建过程中不应出现弃用警告。

1.  确保从统计数据中使用入口点信息。如果使用`HtmlWebpackPlugin`，则无需执行此步骤。

对于包含静态 HTML 或以其他方式创建静态 HTML 的构建，您必须确保使用入口点来生成任何脚本和链接任何 HTML 标签的统计 JSON 文件。如果不可能，您应该避免将`splitChunks.chunks`键设置为`all`，并且不要针对`splitChunks.maxSize`键设置任何设置。然而，这只是一个变通方法，可以被认为不是理想的解决方案。

1.  确保将模式设置为`production`或`development`，以确保设置相应模式的默认值。

1.  此外，如果您使用了以下选项，请确保将它们更新为更新版本：

```js
optimization.hashedModuleIds: true => optimization.moduleIds:
  'hashed'
optimization.namedChunks: true => optimization.chunkIds: 'named'
optimization.namedModules: true => optimization.moduleIds: 'named'
NamedModulesPlugin => optimization.moduleIds: 'named'
NamedChunksPlugin => optimization.chunkIds: 'named'
HashedModulesPlugin => optimization.moduleIds: 'hashed'
optimization.occurrenceOrder: true => optimization: { chunkIds:
   'total-size', moduleIds: 'size' }
optimization.splitChunks.cacheGroups.vendors =>
   optimization.splitChunks.cacheGroups.defaultVendors
```

1.  接下来，我们需要测试 Webpack 5 与您的应用程序的兼容性。为此，请为您的 Webpack 4 配置设置以下选项。如果在 Webpack 4 中没有任何构建错误，我们将知道任何后续故障是否是版本 5 独有的。这可能听起来很繁琐，但它可以消除递归故障查找：

```js
module.exports = {
 // ...
   node: {
     Buffer: false,
     process: false
   }
 };
```

在 Webpack 5 中，上述选项已从配置中删除，并默认设置为`false`。确保在 Webpack 4 测试构建中执行此操作，但在版本 5 构建中需要再次删除。

1.  接下来是一个简单和简洁的命令行执行，用于升级您的 Webpack 版本：

```js
npm: npm install webpack@next --dev
Yarn: yarn add webpack@next -D
```

现在，我们需要清理我们的配置。

建议您将配置中的`[hash]`占位符更改为`[contenthash]`。这已被证明更有效，并且可以帮助加固您的代码。

如果您正在使用`pnp-webpack-plugin`，它现在默认支持 Webpack 的版本 5，但现在需要从配置模式中删除。

`IgnorePlugin`现在接受一个选项对象，因此如果您将其用作正则表达式，则需要进行重写，例如以下内容：

```js
new IgnorePlugin({ resourceRegExp: /regExp/ }).
```

对于通过`import`使用 WASM 的开发人员，您应该通过将`experiments.syncWebAssembly`变量设置为`true`来启用已弃用的规范。这将在 Webpack 5 中设置与 Webpack 4 相同的行为。一旦您已经迁移到 Webpack 5，现在应该更改实验的值以使用 WASM 的最新规范——`{ asyncWebAssembly: true, importAsync: true }`。

在使用自定义配置时，还应注意将`name`值替换为`idHint`。

在 Webpack 5 中，不支持从 JSON 模块中导出命名的导出，并且会收到警告。要以这种方式导入任何内容，应该从`package.json`中使用`const[version]=package;`。

1.  现在，清理构建代码是一个好的做法。其中一部分是在使用`const compiler =webpack(...);`时关闭编译器。这可以通过`compiler.close();`来完成。

一旦运行构建，可能会出现一些问题。例如，模式验证可能会失败，Webpack 可能会退出并显示错误，或者可能会出现构建错误、构建警告或弃用警告。

在每种情况下，都会有一个破坏性变更说明或一个带有指令的错误消息，可以通过命令行获得，就像往常一样。

在弃用警告的情况下，可能会有很多弃用警告，因为 Webpack 5 是新的，插件需要时间来适应核心变化。在每个版本完成测试之前，应该忽略它们，这是一个良好的做法。

您可以通过使用`--no-deprecation`标志来隐藏弃用警告，例如`node --no-deprecation`。

插件和加载器的贡献者应该遵循弃用消息中的警告建议来改进他们的代码。

如果需要，您还可以关闭运行时代码中的 ES2015 语法。默认情况下，Webpack 的运行时代码使用 ES2015 语法来构建更小的包。如果您的构建目标环境不支持此语法，例如 IE 11，您需要将`output.ecmaVersion: 5`设置为恢复到 ES5 语法。

处理遗留问题将是向上迁移时面临的最大障碍，这一规则不仅适用于 Webpack 5。Webpack 5 具有一些功能，将使遗留平台用户的体验更加愉快。在项目规划中考虑的一种方法是持久缓存。

# 启用持久缓存

缓存当然是为了提高加载时间和加快性能而进行的数据的中间存储。持久缓存在数据库驱动项目中非常常见，从数据库中提取的数据会被缓存，以便用户拥有早期版本的副本。然后可以一次性加载，而不会对数据库造成太大的需求，因为数据的交付速度比基于服务器的文件条目要慢。

使用 Webpack 5，应用程序可以利用相同的操作并提高用户的加载速度，如果构建发生变化。

首先，要注意的是，持久缓存不是默认启用的。您必须选择使用它。这是因为 Webpack 5 更看重安全性而不是性能。启用即使提高了性能但会以任何小的方式破坏您的代码，这可能不是最好的主意。至少作为默认，此功能应保持禁用状态。

序列化和反序列化将默认工作；但是，开发人员可能会在缓存失效方面遇到麻烦。

缓存失效是指应用程序中有意更改时，例如开发人员更改文件的内容；在这种情况下，Webpack 会将旧内容的缓存视为无效。

Webpack 5 通过跟踪每个模块使用的`fileDependencies`、`contextDependencies`和`missingDependencies`来实现这一点。然后，Webpack 从中创建一个文件系统图。然后，文件系统与记录的副本进行交叉引用，这反过来会触发该模块的重建。

然后，输出包的缓存条目会为其生成一个标签；这本质上是所有贡献者的哈希。标签与缓存条目之间的匹配表示 Webpack 可以用于打包的内容。

Webpack 4 使用相同的过程进行内存缓存，而在 Webpack 5 中也可以工作，无需额外配置，除非启用了持久缓存。

您还需要在升级加载器或插件时使用`npm`，更改配置，更改要在配置中读取的文件，或者升级配置中使用的依赖项时，通过传递不同的命令行参数来运行构建，或者拥有自定义构建脚本并对其进行更改时，使缓存条目失效。

由于 Webpack 5 无法直接处理这些异常，因此持久缓存被作为一种选择性功能，以确保应用程序的完整性。

# Webpack 更新

有许多步骤必须采取，以确保 Webpack 的更新行为正确。与我们的示例相关的步骤如下：

+   升级和安装。

+   将模式添加到配置文件中。

+   添加 fork 检查器。

+   手动更新相关插件、加载器和实用程序。

+   重新配置`uglify`。

+   跟踪任何进一步的错误并进行更新。

让我们详细了解每个步骤，并探索命令行中到底发生了什么。这应该有助于您更好地理解该过程：

1.  我们需要做的第一件事是升级 Webpack 并安装`webpack-cli`。这是在命令行中完成的，如下所示：

```js
yarn add webpack
yarn add webpack-cli
```

1.  前面的示例显示了使用`yarn`来完成这个操作，并且还会进行版本检查。这也应该在`package.json`文件中可见：

```js
...
"webpack": "⁵.0.0",
"webpack-cli": "³.2.3",
...
```

1.  完成后，应该将相应的模式添加到`webpack.config.dev.js`和`webpack.config.prod.js`。参见以下`webpack.config.dev.js`文件：

```js
module.exports = {
mode: 'development',
```

与生产配置一样，我们在这里为每种模式都有两个配置文件。以下显示了`webpack.config.prod.js`文件的内容：

```js
module.exports = {
mode: 'production',
```

我们正在处理两个版本——旧版本（3）和新版本（4）。如果这是手动完成的，您可能首先要对原始版本进行**fork**。fork 一词指的是通常与此操作相关的图标，它代表一行从另一行分离出来，看起来像一个叉子。因此，fork 一词已经成为一个分支的意思。fork 检查器将自动检查每个版本的差异，以便作为操作的一部分进行更新。

1.  现在，回到命令行添加以下 fork 检查器：

```js
add fork-ts-checker-notifier-webpack-plugin
yarn add fork-ts-checker-notifier-webpack-plugin --dev
```

`package.json`文件中应该看到以下内容：

```js
...
"fork-ts-checker-notifier-webpack-plugin": "¹.0.0",
...
```

前面的代码块显示了 fork 检查器已经安装。

1.  现在，我们需要使用命令行更新`html-webpack-plugin`：

```js
yarn add html-webpack-plugin@next
```

`package.json`现在应该显示如下内容：

```js
"html-webpack-plugin": "⁴.0.0-beta.5",
```

现在，我们需要调整`webpack.config.dev.js`和`webpack.config.prod.js`文件中的插件顺序。

1.  您应该采取这些步骤来确保`HtmlWebpackPlugin`在**`InterpolateHtmlPlugin`**之前声明，并且`InterpolateHtmlPlugin`在下面的示例中被声明：

```js
plugins: [
 new HtmlWebpackPlugin({
   inject: true,
   template: paths.appHtml
 }),
 new InterpolateHtmlPlugin(HtmlWebpackPlugin, env.raw),
```

1.  还要确保在命令行中更新`ts-loader`、`url-loader`和`file-loader`：

```js
yarn add url-loader file-loader ts-loader
```

1.  `package.json`文件保存了关于使用的版本的信息，就像之前提到的加载器一样，并且应该如下所示：

```js
"file-loader": "¹.1.11",
"ts-loader": "4.0.0",
"url-loader": "0.6.2",
```

如果您正在使用**React**，那么您需要更新开发实用程序，如下所示：

```js
yarn add react-dev-utils
```

同样，`package.json`文件将保存所使用的 React 实用程序的版本信息：

```js
"react-dev-utils": "6.1.1",
```

`extract-text-webpack-plugin`应该被替换为**`mini-css-extract-plugin`。**

1.  请注意，应该完全删除`extract-text-webpack-plugin`，同时添加和配置`mini-css-extract-plugin`：

```js
yarn add mini-css-extract-plugin
```

1.  对于此示例，`package.json`文件中带有插件版本设置应该如下所示：

```js
"mini-css-extract-plugin": "⁰.5.0",
Config:
```

1.  完成所有这些后，我们应该看一下生产模式的配置。这是通过以下`webpack.config.prod.js`文件完成的：

```js
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
plugins: [
 ...
 new MiniCssExtractPlugin({
   filename: "[name].css",
   chunkFilename: "[id].css"
 }),
 ...
 ],
 module: {
   rules: [
    {
     test: /\.css$/,
     use: [
     {
       loader: MiniCssExtractPlugin.loader,
       options: {
       // you can specify a publicPath here
       // by default it use publicPath in webpackOptions.output
       publicPath: '../'
     }
    },
    "css-loader"
   ]
 },
```

我们可以看到在不同版本之间的`webpack.config.prod.js`中有一些差异。前面的示例让你了解了在版本 4 中进行配置的格式。

1.  接下来，请确保使用命令行和`package.json`文件更新和重新配置`uglifyjs-webpack-plugin`：

```js
yarn add uglifyjs-webpack-plugin --dev
```

1.  为了谨慎起见，我们还将展示此处`uglify`插件的版本设置。使用`package.json`应用这些设置：

```js
"uglifyjs-webpack-plugin": "².1.2"
 Config:
```

1.  下一步，最后一步是使用`webpack.config.prod.js`配置生产模式：

```js
const UglifyJsPlugin = require('uglifyjs-webpack-plugin');
module.exports = {
  ...
  optimization: {
    minimizer: [new UglifyJsPlugin()],
 },
```

完成所有这些后，您应该完成更新过程。但是，您可能会遇到一个独特的弃用错误，这意味着您需要使用错误消息跟踪这些错误，然后根据需要更新任何进一步的 Webpack 插件。如果您正在使用自定义插件或加载器，情况将尤其如此。

# 总结

本章深入探讨了代码调试，并讨论了 HMR 和其他调试技术。您现在应该对可以用于增强调试过程的工具和实用程序有了牢固的掌握，包括使用`node nightly`进行代码检查。然后我们深入研究了 HMR，这是 Webpack 的一个显著和令人兴奋的特性。我们看到了如何对模块和样式表进行实时编辑，甚至涵盖了迁移问题。然后我们过渡到添加实用程序，这是任何升级的重要部分。从那里，我们带您完成了版本迁移，即从版本 3 迁移到版本 4，并介绍了相应的步骤。此外，我们还向您展示了如何从版本 4 迁移到版本 5。本节以一个漫长的教程结束，介绍了如何将命令行升级更新为对一些更棘手的元素进行手动更改。

您现在应该对自己的调试和升级技能充满信心，这将使您在下一章中站稳脚跟。在下一章中，我们将进行一些繁重的现场编码、定制和手动捆绑，这无疑会让您兴奋不已！

# 进一步阅读

本章涵盖了一些复杂问题，通过进一步阅读可以更好地理解。以下是一些主题以及在本章中提到的相关内容的查找位置：

+   调试优化退出

+   问题 6074—为`sideEffects`添加对更复杂选择器的支持

# 问题

现在，尝试一下与本章相关的以下问题。您将在本书的*评估*部分中找到答案：

1.  HMR 是什么意思？

1.  React Hot Loader 是做什么的？

1.  Webpack 通过哪个接口进行更新？

1.  Node v6.3.0+的哪个功能允许通过 Chrome DevTools 进行调试？

1.  从 Webpack 版本 3 迁移到版本 4 并使用自定义加载器转换`.json`文件时，您还必须改变什么？

1.  side effects 列表如何帮助开发？

1.  在生产模式中应该从哪里删除不推荐使用的插件？
