# 第六章：生产、集成和联合模块

在本章中，我们将涵盖生产、集成和联合模块。我们将概述正确的部署程序、快捷方式和替代方案。尽管本章的一些内容已经在更多细节上进行了讨论，但再次复习一下是很好的，这样您就可以更清楚地了解到目前为止学到的内容。

到目前为止，我们已经讨论并执行了开发构建，并暗示要进入生产，但适当的发布级生产过程有点不同，涉及交叉检查和遵循最佳实践。

本书的这一部分将探讨我们可以使用的各种选项，以部署 Webpack 5 与各种 Web 实用程序。这将为您提供这些 Web 实用程序的概述，并解释哪些对于特定情况和平台更为合适，包括使用 Babel 进行部署。

所有这些主题都与我们在生产捆绑的开头部分相关，该部分还涵盖了生产和发布目的的部署主题。

在本章中，我们将涵盖以下主题：

+   生产设置

+   Shimming

+   渐进式 Web 应用程序

+   集成任务运行器

+   GitHub

+   提取样板

+   模块联合

# 生产设置

现在我们了解了基础知识，我们可以继续学习如何实际部署生产捆绑包。在开发模式和生产模式下构建项目的目标有很大的不同。在生产模式下，目标转向使用轻量级源映射对构建进行最小化，并优化资产以提高加载时间。在开发模式下，强大的源映射至关重要，以及具有**localhost**服务器和实时重新加载或热模块替换。由于它们的不同目的，通常建议为每种模式编写单独的 Webpack 配置。

尽管它们之间存在差异，但通用配置应该在模式之间保持一致。为了合并这些配置，可以使用一个叫做`webpack-merge`的实用工具。这个通用配置过程意味着代码不需要在每个模式下重复。让我们开始吧：

1.  首先打开命令行工具，安装`webpack-merge`，并以开发模式保存它，如下所示：

```js
npm install --save-dev *webpack-merge*
```

1.  现在，让我们来看一下**项目目录**。它的内容应该结构类似于以下内容：

```js
 webpack5-demo
  |- package.json
  |- webpack.config.js
  |- webpack.common.js
  |- webpack.dev.js
  |- webpack.prod.js
  |- /dist
  |- /src
    |- index.js
    |- math.js
  |- /node_modules
```

请注意，前面的输出显示了在这个特定示例中存在额外的文件。例如，我们在这里包括了`webpack.common.js`文件。

1.  让我们仔细看一下`webpack.common.js`文件：

```js
  const path = require('path');
  const { CleanWebpackPlugin } = require('clean-webpack-plugin');
  const HtmlWebpackPlugin = require('html-webpack-plugin');

  module.exports = {
    entry: {
      app: './src/index.js'
    },
    plugins: [
      // new CleanWebpackPlugin(['dist/*']) for < v2 versions of 
      // CleanWebpackPlugin
      new CleanWebpackPlugin(),
      new HtmlWebpackPlugin({
        title: 'Production'
      })
    ],
    output: {
      filename: '[name].bundle.js',
      path: path.resolve(__dirname, 'dist')
    }
  };
```

`webpack.common.js`文件处理**CommonJS**请求。它的做法与 ECMAScript 类似，但格式不同。让我们确保在**ECMA**环境中工作的`webpack.config.js`文件与**CommonJS**配置文件做同样的事情。注意入口点和捆绑名称，以及**`title`**选项。后者是模式的对应项，因此您必须确保项目中的两个文件之间存在对等性。

1.  在这里，我们来看一下`webpack.dev.js`文件：

```js
  const merge = require('webpack-merge');
  const common = require('./webpack.common.js');

  module.exports = merge(common, {
    mode: 'development',
    devtool: 'inline-source-map',
    devServer: {
      contentBase: './dist'
    }
  });
```

正如您所看到的，前面的代码提供了关于如何在开发模式下使用`webpack.common.js`的说明。这只是一个交叉检查的情况，以确保您的工作内容在最终生产中格式正确，并且在编译过程中不会出现错误。

1.  如果您正在生产模式下工作，将调用以下文件`webpack.prod.js`：

```js
  const merge = require('webpack-merge');
  const common = require('./webpack.common.js');

  module.exports = merge(common, {
    mode: 'production',
  });
```

使用`webpack.common.js`，设置入口和输出配置，并包括在两种环境模式下都需要的任何插件。在使用`webpack.dev.js`时，模式应设置为开发模式。还要在该环境中添加推荐的**devtool**，以及简单的`devServer`配置。在`webpack.prod.js`中，模式当然设置为生产模式，加载`TerserPlugin`。

请注意，`merge()` 可以在特定环境配置中使用，以便您可以轻松地在开发和生产模式中包含通用配置。值得注意的是，在使用 `**webpack-merge**` 工具时还有各种高级功能可用。

1.  让我们在 **`webpack.common.js`** 中进行这些开发配置：

```js
  { 
   "name": "development", 
   "version": "1.0.0", 
   "description": "", 
   "main": "src/index.js",
   "scripts": { 
   "start": "webpack-dev-server --open --config webpack.dev.js", 
   "build": "webpack --config webpack.prod.js" 
  }, 
    "keywords": [], 
    "author": "", 
    "license": "ISC", 
    "devDependencies": { 
      "clean-webpack-plugin": "⁰.1.17", 
      "css-loader": "⁰.28.4", 
      "csv-loader": "².1.1", 
      "express": "⁴.15.3",
      "file-loader": "⁰.11.2", 
      "html-webpack-plugin": "².29.0", 
      "style-loader": "⁰.18.2", 
      "webpack": "⁵.0.0",
      "webpack-dev-middleware": "¹.12.0",
      "webpack-dev-server": "².9.1", 
      "webpack-merge": "⁴.1.0", 
      "xml-loader": "¹.2.1"
   } 
 }
```

前面的示例只是展示了 CommonJS 的完成配置。请注意通过 `devDependancies` 选项加载的插件依赖项及其版本列表。

1.  现在，运行这些脚本，看看输出如何变化。

1.  以下代码显示了如何继续添加到生产配置：

```js
 document.body.appendChild(component());
```

请注意，当处于生产模式时，Webpack 5 将默认压缩项目的代码。

`TerserPlugin` 是开始压缩的好地方，应该作为默认选项使用。然而，还有一些选择，比如 `BabelMinifyPlugin` 和 `closureWebpackPlugin`。

尝试不同的压缩插件时，请确保选择也会删除任何死代码，类似于我们在本书中早些时候描述的摇树。与摇树相关的是 shimming，我们将在下面讨论。

# Shimming

Shimming，或者更具体地说，`shim-loaders`。现在是深入探讨这个概念的好时机，因为在您继续之前，您需要牢牢掌握它。

Webpack 使用的编译器可以理解用 **ECMAScript** **2015**，**CommonJS** 或 **AMD** 编写的模块。值得注意的是，一些第三方库可能需要全局依赖，例如使用 jQuery 时。在这种情况下，这些库可能需要导出全局变量。模块的这种几乎破碎的特性是 Shimming 发挥作用的地方。

Shimming 可以让我们将一种语言规范转换为另一种。在 Webpack 中，这通常是通过专门针对您的环境的加载器来完成的。

Webpack 的主要概念是模块化开发的使用 - 孤立的模块，安全地包含，不依赖于隐藏或全局依赖关系 - 因此重要的是要注意，使用这样的依赖关系应该是稀少的。

当您希望对浏览器进行 polyfill 以支持多个用户时，Shimming 可以派上用场。在这种情况下，polyfill 只需要在需要的地方进行修补并按需加载。

Shimming 与修补有关，但往往发生在浏览器中，这使得它与渐进式网络应用程序高度相关。

在下一节中，我们将更详细地了解渐进式网络应用程序 - 它们对于 Webpack 5 非常重要。

# 渐进式网络应用

有时被称为 PWA，渐进式网络应用程序在线提供原生应用程序体验。它们有许多 contributing 因素，其中最显著的是应用程序在脱机和在线时都能正常运行的能力。为此，使用了服务工作者。

使您的网络应用程序能够脱机工作将意味着您可以提供诸如推送通知之类的功能。这些丰富的体验也将通过诸如服务工作者之类的设备提供给基于网络的应用程序。此脚本将在浏览器的后台运行，无论用户是否在正确的页面上，都会允许这些相同的通知甚至后台同步。

PWA 提供了网络的覆盖范围，但速度快、可靠，类似于桌面应用程序。它们也可以像移动应用一样引人入胜，并且可以提供相同的沉浸式体验。这展示了应用程序的新质量水平。它们也可以在跨平台兼容性中发挥作用。

响应式设计是网络朝这个方向的第一次推动，但使互联网更普遍的推动使我们走向了 PWA。为了发挥应用程序的潜力，您应该采用渐进式的方法。有关更多信息，请参阅 Google 关于此主题的资源：[`developers.google.com/web/progressive-web-apps/`](https://developers.google.com/web/progressive-web-apps/)。

在使用 Webpack 时，需要注册服务工作者，以便您可以开始将桌面功能集成到基于 Web 的 PWA 中。PWA 并不是为用户本地安装而设计的；它们通过 Web 浏览器本地工作。

通过将以下内容添加到您的代码中，可以注册服务工作者 - 在本例中，这是一个`index.js`文件：

```js
  if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => {
    navigator.serviceWorker.register('/service-
      worker.js').then(registration => {
       console.log('SW registered: ', registration);
    }).catch(registrationError => {
     console.log('SW registration failed: ', registrationError);
    });
  });
 }
```

完成后，运行`npm build` - 您应该在命令行窗口中看到以下输出：

```js
SW registered
```

现在，应用程序可以在命令行界面中使用`npm start`来提供服务。

PWA 应该有一个清单、一个服务工作者，可能还有一个 workbox 来包装和保护服务工作者。有关清单的更多信息，请参见第三章，*使用配置和选项*。Workbox 是一个可以使用以下命令在命令行中安装的插件：

```js
npm install workbox-webpack-plugin --save-dev
```

在假设的`webpack.config.js`文件中，可以在此处看到**Workbox**插件的示例配置：

```js
const WorkboxPlugin = require('workbox-webpack-plugin');
new WorkboxPlugin.GenerateSW({
 clientsClaim: true,
 skipWaiting: true,
 }),
```

大括号`{`内的选项将鼓励服务工作者迅速到达该点，并且不允许它们扼杀先前的服务工作者。

随着项目变得更加复杂，您可能会考虑使用相关的任务运行器。让我们更详细地看一下这一点。

# 集成任务运行器

任务运行器处理自动化任务，如代码检查。Webpack 并不是为此目的而设计的 - 没有捆绑器是。然而，使用 Webpack，我们可以从任务运行器提供的高度关注度中受益，同时仍然具有高性能的捆绑。

虽然任务运行器和捆绑器之间存在一些重叠，但如果正确处理，它们可以很好地集成。在本节中，我们将探讨一些最流行的任务运行器的集成技术。

捆绑器通过准备 JavaScript 和其他前端代码以进行部署，转换其格式，使其适合于浏览器交付。例如，它允许我们将 JavaScript 拆分成块以进行延迟加载，从而提高性能。

捆绑可能是具有挑战性的，但其结果将消除许多繁琐的工作过程。

在本节中，我们将向您展示如何集成以下内容：

+   Gulp

+   Mocha

+   Karma

Gulp 可能是最知名的任务运行器，因此让我们首先从那里开始。它通过使用专用文件来利用，就像其他两个任务运行器一样。在这里，`gulpfile.js`文件将处理 Webpack 与 Gulp 的相互作用。让我们看看如何集成所有这些任务运行器：

1.  首先，让我们看一下`gulpfile.js`文件：

```js
const gulp = require('gulp'); 
const webpack = require('webpack-stream'); 
gulp.task('default', function() { 
   return gulp.src('src/entry.js') 
    .pipe(webpack({ 
 // Any configuration options... 
    })) 
 .pipe(gulp.dest('dist/')); 
});
```

这就是我们使用 Gulp 所需要做的。请注意`gulp.task`函数的使用，`return`入口点以及`.pipe(**gulp**.dest('dist/'));`输出位置。

1.  以下是您可以使用的命令行代码来安装 Mocha：

```js
npm install --save-dev *webpack mocha mocha-webpack*
*mocha-webpack* 'test/**/*.js'
```

有关更多信息，请访问 Webpack 社区存储库。

1.  以下是您需要对`karma.config.js`文件进行的配置文件修改，以便与 Webpack 一起使用 Karma：

```js
module.exports = function(config) {
  config.set({
    files: [
      { pattern: 'test/*_test.js', watched: false },
      { pattern: 'test/**/*_test.js', watched: false }
    ],
    preprocessors: {
      'test/*_test.js': [ 'webpack' ],
      'test/**/*_test.js': [ 'webpack' ]
    },
    webpack: {
      // Any custom webpack configuration...
    },
    webpackMiddleware: {
      // Any custom webpack-dev-middleware configuration...
    }
  });
};
```

`webpack`和`webpackMiddleware`选项已留空，以便您可以使用项目的特定配置填充此内容。如果您不使用它们，这些选项可以完全删除。出于本例的目的，我们不会这样做。

如果您希望在开发环境中使用它们，这些安装过程将对您有所帮助，但 GitHub 是一个越来越重要的工具。我们将在下一节中看一下它如何在开发中发挥关键作用。

# GitHub

如您可能已经知道的那样，GitHub 是一个与 Webpack 配合良好的命令行代码托管平台。在使用 Webpack 时，您将与 GitHub 托管的大部分代码和项目一起工作。

GitHub 基于 Git 版本控制系统。使用 GitHub 与 Webpack 5 允许在线使用一些命令行操作。

Git 命令行指令通常在每个新条目和每个命令之前使用`git`命令。Webpack 的大部分内容文件都可以通过 GitHub 获得，GitHub 的 Webpack 页面可以在这里找到：[`github.com/webpack/webpack`](https://github.com/webpack/webpack)。Webpack 5 的开发可以在这里查看其进展阶段，这可能会很有趣，并且可以让您更好地预期其到来，如果您需要升级您的项目。其 URL 为[`github.com/webpack/webpack/projects/5.`](https://github.com/webpack/webpack/projects/5)

作为开发人员，您可能经常使用 GitHub，但如果您是一名专注于 JavaScript 开发的开发人员，您可能经验有限。在处理 Webpack 项目时，GitHub 平台提供了大量的实时和协作机会。由于提供了版本控制和命令行功能，因此在本地执行软件开发的需求较少。这是 GitHub 在开发人员社区中如此受欢迎并成为开发人员工作证明的主要原因。

GitHub 允许开发人员共同在项目上工作。在处理捆绑项目时，这更加有用，因为一些命令行操作可以在线运行。GitHub 还允许敏捷工作流程或项目管理界面。敏捷方法允许团队通过专用数字平台进行协作，个人自组织。

在使用 GitHub 时，您可能会使用其他人的代码。这可能包括由团队开发的代码框架。即使是经验丰富的开发人员，如果他们对使用的逻辑不熟悉，这也可能变得非常困难。这就引出了样板的主题，通常是标准或有良好文档记录的代码，但尽管如此，您可能希望从项目中提取出这部分。这就是提取过程开始变得非常有用的地方。

# 提取样板

样板代码是需要包含在各个地方的代码部分，但几乎没有或没有进行修改。在使用被认为冗长的语言时，通常需要编写大量的代码。这一大段代码称为样板。它本质上与框架或库具有相同的目的，这些术语经常混淆或相互接受。

Webpack 社区提供了样板安装，例如安装多个常见依赖项的组合安装，如先决条件和加载程序。这些样板有多个版本，使用它们可以加快构建速度。请搜索 Webpack 社区页面（[`webpack.js.org/contribute/`](https://webpack.js.org/contribute/)）或 Webpack 的 GitHub 页面（[`github.com/webpack-contrib`](https://github.com/webpack-contrib)）以获取示例。

也就是说，有时只需要部分样板。为此，可能需要提取 Webpack 的样板功能。

在使用其最小化方法时，Webpack 允许提取样板；也就是说，只有您需要的样板元素包含在捆绑包中。这是在编译过程中自动完成的过程。

缩小是 Webpack 5 提供这种提取过程的关键方式，也是这种类型的捆绑器可以使用的更显著的方式之一。还有另一个关键过程，对于 Webpack 5 来说是非常有用的，也是本地的。它带我们走向了捆绑包的不同方向，但是这是一个毫无疑问会从一个复杂或自定义的构建中跟随的过程，比如一个从模板开始提取的项目。这个过程被称为模块联邦。

# 模块联邦

模块联邦已被描述为 JavaScript 架构的一个改变者。它基本上允许应用程序在服务器之间运行远程存储的模块代码，同时作为捆绑应用程序的一部分。

一些开发人员可能已经了解一种称为**GraphQL**的技术。它基本上是一个由 Apollo 公司开发的用于在应用程序之间共享代码的解决方案。联邦模块是 Webpack 5 的一个功能，允许捆绑应用程序之间发生这种情况。

长期以来，最好的折衷方案是使用`DllPlugin`的外部依赖，它依赖于集中的外部依赖文件，但是这对于有机开发、便利性或大型项目来说并不理想。

使用模块联邦，JavaScript 应用程序可以在应用程序之间动态加载代码并共享依赖关系。如果一个应用程序在其构建中使用了联邦模块，但需要一个依赖项来提供联邦代码，Webpack 也可以从联邦构建的原始位置下载该依赖项。因此，联邦将有效地提供 Webpack 5 可以找到所需依赖代码的地图。

在考虑联邦时，有一些术语需要考虑：远程和主机。远程一词指的是加载到用户应用程序中的应用程序或模块，而主机指的是用户在运行时通过浏览器访问的应用程序。

联邦方法是为独立构建设计的，可以独立部署或在您自己的存储库中部署。在这种意义上，它们可以双向托管，有效地作为远程内容的主机。这意味着单个项目可能在用户的旅程中在托管方向之间切换。

# 构建我们的第一个联邦应用

让我们首先看一下三个独立的应用程序，分别标识为第一个、第二个和第三个应用程序。

# 我们系统中的第一个应用程序

让我们从配置第一个应用程序开始：

1.  我们将在 HTML 中使用`<app>`容器。这个第一个应用程序是联邦系统中的远程应用程序，因此将被其他应用程序消耗。

1.  为了暴露应用程序，我们将使用`AppContainer`方法：

```js
const HtmlWebpackPlugin = require("html-webpack-plugin");
const ModuleFederationPlugin =
   require("webpack/lib/container/ModuleFederationPlugin");

module.exports = {
  plugins: [
   new ModuleFederationPlugin({
    name: "app_one_remote",
    remotes: {
      app_two: "app_two_remote",
      app_three: "app_three_remote"
 },
 exposes: {
   'AppContainer':'./src/App'
 },
 shared: ["react", "react-dom","react-router-dom"]
 }),
 new HtmlWebpackPlugin({
   template: "./public/index.html",
   chunks: ["main"]
  })
 ]
} 
```

这个第一个应用程序还将消耗系统中其他两个联邦应用程序的组件。

1.  为了允许这个应用程序消耗组件，需要指定远程范围。

1.  所有这些步骤都应按照前面的代码块中指定的方式进行。

1.  现在，让我们来看一下构建的 HTML 部分：

```js
<head>
  <script src="img/app_one_remote.js"></script>
  <script src="img/app_two_remote.js"></script>
</head>
<body>
  <div id="root"></div>
</body>
```

前面的代码显示了 HTML 的`<head>`元素。`app_one_remote.js`在运行时连接运行时和临时编排层。它专门设计用于入口点。这些是示例 URL，您可以使用自己的位置。重要的是要注意，这个例子是一个非常低内存的例子，您的构建可能会大得多，但足够好理解这个原则。

1.  为了消耗远程应用程序的代码，第一个应用程序有一个网页，从第二个应用程序中消耗对话框组件，如下所示：

```js
const Dialog = React.lazy(() => import("app_two_remote/Dialogue")); 
const Page1 = () => { 
  return ( 
    <div> 
      <h1>Page 1</h1> 
        <React.Suspense fallback="Loading User Dialogue..."> 
          <Dialog /> 
        </React.Suspense> 
    </div> 
  ); 
}
```

1.  让我们从导出我们正在使用的默认 HTML 页面开始，并设置路由，操作如下：

```js
import { Route, Switch } from "react-router-dom";
import Page1 from "./pages/page1";
import Page2 from "./pages/page2";
import React from "react";
   const Routes = () => (
     <Switch>
       <Route path="/page1">
        <Page1 />
       </Route>
       <Route path="/page2">
        <Page2 />
       </Route>
     </Switch>
  );
```

前面的代码块显示了代码如何工作；它将从我们正在开发的系统中的每个页面导出默认路由。

这个系统是由三个应用程序组成的，我们现在来看第二个应用程序。

# 第二个应用程序

我们正在构建的系统由三个应用程序组成。这个应用程序将暴露对话框，使得这个序列中的第一个应用程序可以消耗它。然而，第二个应用程序将消耗第一个应用程序的`<app>`元素标识符。让我们来看一下：

1.  我们将从配置第二个应用程序开始。这意味着我们需要指定`app-one`作为远程应用程序，并同时演示双向托管：

```js
 const HtmlWebpackPlugin = require("html-webpack-plugin");
 const ModuleFederationPlugin =
   require("webpack/lib/container/ModuleFederationPlugin");
 module.exports = {
   plugins: [
     new ModuleFederationPlugin({
     name: "app_two_remote",
     library: { type: "var", name: "app_two_remote" },
     filename: "remoteEntry.js",
     exposes: {
       Dialog: "./src/Dialogue"
 },
   remotes: {
     app_one: "app_one_remote",
 },
   shared: ["react", "react-dom","react-router-dom"]
 }),
 new HtmlWebpackPlugin({
   template: "./public/index.html",
   chunks: ["main"]
  })
 ]
};
```

1.  为了消耗，以下是根应用程序的样子：

```js
import React from "react";
import Routes from './Routes'
const AppContainer = React.lazy(() =>
  import("app_one_remote/AppContainer"));

const App = () => {
  return (
   <div>
     <React.Suspense fallback="Loading App Container from Host">
       <AppContainer routes={Routes}/>
     </React.Suspense>
   </div>
  );
}
```

1.  接下来，我们需要设置代码，以便我们可以导出默认的应用程序。以下是使用对话框时默认页面应该看起来的示例：

```js
import React from 'react'
import {ThemeProvider} from "@material-ui/core";
import {theme} from "./theme";
import Dialog from "./Dialogue";

function MainPage() {
   return (
     <ThemeProvider theme={theme}>
       <div>
         <h1>Material User Iinterface App</h1>
         <Dialog />
      </div>
     </ThemeProvider>
  );
}
```

1.  现在，我们需要导出默认的`MainPage`。这是通过我们系统中的第三个应用程序完成的。

# 第三个应用程序

让我们来看看我们的第三个和最后一个应用程序：

1.  我们系统中的第三个应用程序将导出一个默认的`MainPage`。这是通过以下脚本完成的：

```js
new ModuleFederationPlugin({
   name: "app_three_remote",
   library: { type: "var", name: "app_three_remote" },
   filename: "remoteEntry.js",
   exposes: {
     Button: "./src/Button"
 },
 shared: ["react", "react-dom"]
}),
```

如预期的那样，第三个应用程序看起来与之前的应用程序类似，只是它不从第一个应用程序中消耗`<app>`。这个应用程序是独立的，没有导航，因此不指定任何远程联邦组件。

在浏览器中查看系统时，您应该密切关注网络选项卡。代码可以在三个不同的服务器（可能）和三个不同的捆绑包（自然）之间进行联邦。

这个组件允许您的构建具有很大的动态性，但除非您希望利用**服务器端渲染（SSR）**或渐进式加载，否则您可能希望避免联邦整个应用程序容器，否则加载时间可能会受到严重影响。

加载问题是一个自然的关注点，但通常会导致项目规模变大的一个问题是潜在的重复的重复代码，这是使用多个并行捆绑的结果。让我们来看看 Webpack 5 是如何处理这个问题的。

# 重复问题

Webpack 的一个关键特性是去除重复的代码。在联邦环境中，宿主应用程序为远程应用程序提供依赖项。在没有可共享的依赖项的情况下，远程应用程序可以自动下载自己的依赖项。这是一种内置的冗余故障安全机制。

在大规模情况下手动添加供应商代码可能会很繁琐，但联邦功能允许我们创建自动化脚本。这是开发者的选择，但这可能是一个很好的机会，让您能够测试自己的知识。

我们已经提到了 SSR。您应该知道服务器构建需要一个`commonjs`库目标，以便它们可以与 Webpack 联邦一起使用。这可以通过 S3 流式传输、ESI，或者通过自动化 npm 发布来完成，以便您可以消耗服务器变体。以下代码显示了包括`commonjs`的一个示例：

```js
module.exports = {
 plugins: [
  new ModuleFederationPlugin({
    name: "container",
    library: { type: "commonjs-module" },
    filename: "container.js",
    remotes: {
      containerB: "../1-container-full/container.js"
 },
   shared: ["react"]
  })
 ]
};
```

您可能希望使用`target: "node"`方法，以避免 URL 而选择文件路径。这将允许在为 Node.js 构建时使用相同的代码库但不同的配置进行 SSR。这也意味着单独的构建将成为单独的部署。

作为一家公司，Webpack 愿意展示 SSR 示例，您作为开发者社区的一部分可能已经制作了。他们将很乐意通过他们的 GitHub 页面接受拉取请求，因为他们有带宽，并且在这个过程中受益于曝光。

# 总结

在本章中，您了解了部署项目在线的过程。我们讨论了安装和设置过程，以及树摇。

首先，我们看了一下生产和开发模式，每种环境的性质，以及它们如何利用 Webpack。然后，我们看了 shimming，最佳使用方法，它的工作原理，以便我们可以修补代码，它与任务运行器的关系，以及它们与 Webpack 等打包工具的集成。

现在，你应该能够提取 boilerplate，集成各种任务运行器，并知道如何使用 GitHub。

在下一章中，我们将讨论热模块替换和实时编码，并掌握一些严肃的教程。

# 问题

1.  术语 boilerplate 是什么意思？

1.  摇树是做什么的？

1.  术语 shimming 是什么意思？

1.  渐进式 Web 应用的目的是什么？

1.  任务运行器的作用是什么？

1.  这一章提到了哪三个任务运行器？

1.  Webpack 的编译器可以理解哪三种规范编写的模块？

# 进一步阅读

+   Webpack 内容文件和 GitHub 的 Webpack 页面可以在这里找到：[`github.com/webpack/webpack`](https://github.com/webpack/webpack)。

+   Webpack 5 可以从这里按照其进展阶段进行查看：[`github.com/webpack/webpack/projects/5.`](https://github.com/webpack/webpack/projects/5)

+   Webpack 社区页面：[`webpack.js.org/contribute/`](https://webpack.js.org/contribute/)

+   Webpack 的 GitHub 页面：[`github.com/webpack-contrib`](https://github.com/webpack-contrib)
