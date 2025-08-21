# 附录 B. 使用 Webpack

**Webpack** ([`webpack.github.io/`](https://webpack.github.io/)) 又是另一个流行的 Web 模块打包器。Aurelia 已经提供了使用 Webpack 的 ES next 和 Typescript 应用程序骨架。

此外，还有计划将 CLI 对 Webpack-based 项目的支持。然而，目前，骨架是基于 Webpack 创建 Aurelia 项目的最佳起点。

在本附录中，我们将看到在撰写本文档时使用`requirejs`的基于 CLI 的项目和从骨架开始的基于 Webpack 的项目之间的差异。

### 注意

本附录的目的并不是要覆盖 Webpack 本身。因此，我强烈建议如果你还不熟悉 Webpack，请在继续阅读之前先熟悉一下 Webpack。

### 提示

我们的联系人管理应用程序，使用 Webpack 骨架重建，可以在书籍的资源中的`appendix-b\using-webpack`找到，并可作为本附录的参考。

# 入门

为了创建一个基于 Webpack 的应用程序，第一步是下载[`github.com/aurelia/skeleton-navigation/releases/latest`](https://github.com/aurelia/skeleton-navigation/releases/latest)的骨架并解压文件。根目录包含每个可用骨架的独立目录。我们在这里要保留的是名为`skeleton-esnext-webpack`。

Webpack 骨架使用 NPM 作为其包管理器。因此，我们需要通过在项目目录中打开控制台并运行以下命令来安装项目的依赖项：

```js
> npm install

```

完成此操作后，示例应用程序即可运行。

# 运行任务

Webpack 骨架不使用 Gulp 作为其构建系统，而是简单地依赖于 NPM 任务。如果你查看`package.json`文件中的`scripts`部分，你会看到项目的任务列表及其相应的命令。以下是最常见的：

+   `start`：启动开发 Web 服务器。当第一次访问`index.html`时，应用程序被捆绑并提供，然后该过程监视源文件，以便在检测到更改时重新创建捆绑并刷新浏览器。`start`命令是`server`的别名，而`server`又是`server:dev`的别名。

+   `test`：运行单元测试。使用 Istanbul（[`github.com/gotwarlost/istanbul`](https://github.com/gotwarlost/istanbul)）启用代码覆盖。

+   `e2e`：运行端到端测试。此任务将启动应用程序，该应用程序将在端口 19876 上运行，以及 E2E 测试套件。

+   `build:prod`：为生产环境打包应用程序。捆绑包和`index.html`文件将被优化以适应生产环境，并将在`dist`文件夹中生成。此外，生产构建将在每个捆绑包的名称中添加基于内容的全局哈希，以便对它们进行版本控制。这与在 CLI-based 项目中在`aurelia_project/aurelia.json`中设置`rev`选项以启用捆绑修订的效果相同。

+   `server:prod`：启动一个 Web 服务器以提供生产捆绑包。它必须在`build:prod`之后运行。

# 添加库

外部库是通过 NPM 添加的，与基于 CLI 的项目一样。然而，为了使文件被包含在捆绑包中，外部库必须在 JS 文件中引用，因为 Webpack 通过分析应用程序中每个 JS 模块的`import`声明来确定必须包含在捆绑包中的内容。

你可以通过查看骨架的`main`模块来查看这个示例：

`src/main.js`

```js
// we want font-awesome to load as soon as possible to show the fa-spinner 
import '../styles/styles.css'; 
import 'font-awesome/css/font-awesome.css'; 
import 'bootstrap/dist/css/bootstrap.css'; 
import 'bootstrap'; 
//Omitted snippet... 

```

在骨架的示例应用程序中，所有全局资源，如应用程序的样式表、Font Awesome、Bootstrap 的样式表以及 Bootstrap 的 JS 文件都在`main.js`文件中被导入。这些导入将告诉 Webpack 将这些资源包含在应用程序捆绑包中。此外，Webpack 足够智能，可以分析 CSS 文件以找出它们的依赖关系。这意味着它知道如何处理导入的 CSS 文件、图片和字体文件。

# 捆绑

捆绑包本身是在`webpack.config.js`文件中配置的。默认情况下，骨架定义了三个入口捆绑包：

+   `aurelia-bootstrap`：包含 Aurelia 的启动器、默认的 polyfill、Aurelia 的浏览器平台抽象以及 Bluebird Promise 库。

+   `aurelia`：包含所有 Aurelia 的默认库

+   `app`：包含所有应用程序模块

除了直接列为其内容的模块外，一个捆绑包还将包含所有未包含在其他捆绑包中的其内容的依赖项。例如，在骨架的示例中，Bootstrap 的 JS 文件被包含在`app`捆绑包中，因为它们没有被包含在任何其他捆绑包中，并且包含在`app`捆绑包中的模块会导入它们。

如果你想要，例如，让`aurelia`捆绑包包含所有外部库，你应该将其添加到它的模块列表中：

`webpack.config.js`

```js
//Omitted snippet... 
const coreBundles = { 
  bootstrap: [ 
    //Omitted snippet... 
  ], 
  aurelia: [ 
    //Omitted snippet... 
    'aurelia-templating-binding', 
    'aurelia-templating-router', 
    'aurelia-templating-resources', 
    'bootstrap' 
  ] 
} 
//Omitted snippet... 

```

如果你在此时运行示例应用程序，Bootstrap 的 JS 文件应该现在会被包含在`aurelia`捆绑包中，而不是`app`捆绑包。

## 懒加载捆绑包

骨架的示例应用程序中定义的所有捆绑包都是入口捆绑包，这意味着这些捆绑包直接由`index.html`文件加载。所有这些代码都在应用程序启动之前一次性加载。

正如在第十章，*生产环境下的打包*中讨论的，根据应用程序的使用模式和结构，从性能角度来看，将应用程序的不同部分分别打包可能更好，并且只有在需要时才加载其中一些捆绑包。

懒加载包的配置是在`package.json`文件中完成的：

```js
{ 
  //Omitted snippet... 
  "aurelia": { 
 "build": { 
 "resources": [ 
 { 
 "bundle": "contacts", 
 "path": [ 
 "contacts/components/creation", 
 "contacts/components/details", 
 "contacts/components/edition", 
 "contacts/components/form", 
 "contacts/components/list", 
 "contacts/components/photo", 
 "contacts/models/address", 
 "contacts/models/contact", 
 "contacts/models/email-address", 
 "contacts/models/phone-number", 
 "contacts/models/social-profile", 
 "contacts/main" 
 ], 
 "lazy": true 
 } 
 ] 
 } 
 } 
  //Omitted snippet... 
} 

```

在这个例子中，我们将我们联系管理应用程序的`contacts`特性的所有组件和模型打包在一个独特的、懒加载的捆绑包中。有了这个配置，`contacts`捆绑包只有在用户导航到某个联系人路由组件时才会从服务器加载。

至于依赖项包含，懒加载的捆绑包将表现得就像一个入口捆绑包。除了在其配置中列出的模块外，懒加载的捆绑包还将包含所有没有包含在任何入口捆绑包中的依赖项。这意味着如果你只在一个模块中从外部库`import`东西（并且在这个应用程序中的其他地方没有用到），并且你没有将这个外部库包含在某个入口捆绑包中，这个库将被包含在你的懒加载捆绑包中。这是优化应用程序打包时需要考虑的一个重要因素。

## 基于环境的配置

Webpack 骨架使用一个名为`NODE_ENV`的环境变量来根据上下文定制打包过程。这个环境变量通过`package.json`中描述的任务设置为`development`、`test`或`production`。

如果你查看`webpack.config.js`文件，你会看到一个`switch`语句，它根据环境生成一个 Webpack 配置对象。这就是你可以根据环境定制打包的地方。

例如，如果你使用`aurelia-i18n`插件，你可能希望在构建应用程序时将`locales`目录复制到`dist`目录。最简单的方法是在生产和开发配置中都添加以下行：

`webpack.config.js`

```js
//Omitted snippet... 
config = generateConfig( 
  baseConfig, 
  //Omitted snippet... 
  require('@easy-webpack/config-copy-files') 
    ({patterns: [{ from: 'favicon.ico', to: 'favicon.ico' }]}),
  require('@easy-webpack/config-copy-files') 
 ({patterns: [{ from: 'locales', to: 'locales' }]}), 
  //Omitted snippet... 
); 
//Omitted snippet... 

```

另外，如果你想使用`aurelia-testing`插件，无论是用于单元测试中的组件测试器，还是用于调试目的的`view-spy`和`compile-spy`属性，你应该使用 NPM 安装它，并将其添加到`aurelia`捆绑包中，适用于`test`和`development`环境：

`webpack.config.js`

```js
//Omitted snippet... 
coreBundles.aurelia.push('aurelia-testing'); 
config = generateConfig( 
  baseConfig, 
  //Omitted snippet... 
); 
//Omitted snippet... 

```

配置 Webpack 可能一开始会感到复杂和令人畏惧。Webpack 骨架使用`easy-webpack`（[`github.com/easy-webpack/core`](https://github.com/easy-webpack/core)）来简化这个配置过程。使用`easy-webpack`的另一个巨大优势是，它强制执行社区标准，并且使得复用复杂的配置片段变得相当容易。

因此，你可以使用位于[`github.com/easy-webpack`](https://github.com/easy-webpack)或其他地方提供的众多配置模块之一，甚至是你自己的模块，来进一步定制 Webpack 的配置。

# 总结

尽管 Webpack 并非 Aurelia 的首选打包工具，但它已经得到了很好的支持。此外，无论使用 Webpack 还是 CLI 进行打包，Aurelia 应用程序本身的变化并不大，主要是围绕它的基础代码发生了变化。这使得从一个打包工具迁移到另一个打包工具变得更为简单。
