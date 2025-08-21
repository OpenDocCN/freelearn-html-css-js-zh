# 附录 A：使用 JSPM

**JSPM** ([`jspm.io/`](http://jspm.io/)) 是基于未来网络标准的、[`github.com/systemjs/systemjs`](https://github.com/systemjs/systemjs) 通用模块加载器的包管理器，这可能是目前最前瞻性的模块加载器。

在撰写本文的时刻，创建基于 JSPM 的 Aurelia 项目的最简单方法是使用合适的骨架。然而，Aurelia 团队计划在未来的 CLI 中添加创建基于 JSPM 的项目功能。

在这个附录中，我们将看到在撰写本文时使用`requirejs`的 CLI 基础项目和基于 JSPM 的项目骨架之间的区别。

### 注意

附录的目的是不详细介绍 JSPM 和 SystemJS。因此，我强烈建议如果你打算在你的项目中使用它们，你要更多地熟悉它们。

### 提示

使用基于 JSPM 的骨架重建的我们的联系人管理应用程序，可以在书籍资源中的`appendix-a\using-jspm`找到，并可在整个附录中作为参考。

# 入门

创建基于 JSPM 的应用程序的第一步是从[`github.com/aurelia/skeleton-navigation/releases/latest`](https://github.com/aurelia/skeleton-navigation/releases/latest)下载最新的骨架版本，并解压文件。在根目录中，你会发现每个可用的骨架都有一个独特的目录。我们将要查看的是名为`skeleton-esnext`的一个。

JSPM 骨架使用 Gulp 作为其构建系统。因此，如果你还没有安装它，首先打开一个控制台并运行以下命令来全局安装它：

```js
> npm install -g gulp

```

另外，我们还需要安装 JSPM 本身：

```js
> npm install -g jspm

```

一旦我们需要的工具安装完毕，让我们通过在项目目录中打开一个控制台并运行以下命令来恢复项目的构建系统的依赖项：

```js
> npm install

```

此命令将恢复运行和构建我们应用程序所需的所有依赖项，基本上就是`package.json`文件中的`devDependencies`部分的所有内容。

接下来，我们需要通过运行以下命令来恢复我们应用程序本身使用的库：

```js
> jspm install -y

```

此命令将使用 JSPM 恢复`package.json`文件中`jspm`部分的所有的`dependencies`。

到此为止，一切准备就绪。

# 运行任务

JSPM 骨架附带一套相当完整的 Gulp 任务。这些任务可以在`build/tasks`目录中找到。

你最可能想做的第一件事是运行来自骨架的示例应用程序。这可以通过在项目目录中打开一个控制台并运行以下命令来完成：

```js
> gulp watch

```

此命令将启动一个带监视器的开发 Web 服务器，每当源文件更改时都会刷新浏览器。

如果你想在不用监听文件和自动刷新浏览器的情况下运行应用程序，可以通过运行`serve`任务来实现：

```js
> gulp serve

```

## 运行单元测试

默认情况下，基于 JSPM 的骨架的单元测试可以在`test/unit`目录中找到。它通常还包含三个与单元测试相关的不同 Gulp 任务：

+   `test`：运行单元测试一次

+   `tdd`：运行单元测试一次，然后监视文件并当代码变化时重新运行测试

+   `cover`：使用 Istanbul（[`github.com/gotwarlost/istanbul`](https://github.com/gotwarlost/istanbul)）启用代码覆盖率运行单元测试一次。

例如，如果你想进行测试驱动开发并且让测试在编码过程中持续运行，你可以运行以下命令：

```js
> gulp tdd

```

由于骨架依赖于 Karma 来运行测试，所以在运行上述任何任务之前，你需要在你的环境中安装 Karma CLI：

```js
> npm install -g karma-cli

```

## 运行端到端测试

基于 JSPM 的骨架还包含一个`e2e`任务，它将启动在`test/e2e/src`目录中找到的端到端测试。

然而，由于端到端测试依赖于 Protractor，你首先需要通过运行正确的任务来更新 Selenium 驱动程序：

```js
> gulp webdriver-update

```

然后，由于端到端测试需要与应用程序本身交互，你需要启动应用程序：

```js
> gulp serve

```

最后，你可以打开第二个控制台并启动端到端测试：

```js
> gulp e2e

```

# 添加库

使用 JSPM 添加库只需运行正确的命令：

```js
> jspm install aurelia-validation

```

此命令将为项目安装`aurelia-validation`库。由于 JSPM 被设计为与 SystemJS 一起工作，它还将添加适当的条目到 SystemJS 映射配置中，该配置在`config.js`文件中，并由 SystemJS 用来将模块名称映射到 URL 或本地路径。

一旦这个命令完成，SystemJS 模块加载器将能够定位到`aurelia-validation`及其任何依赖项，所以你可以立即在你的应用程序中使用它。

在基于 JSPM 的应用程序中使用库类似于基于 CLI 的项目。如果你需要使用库的一些 JS 导出，只需在 JS 文件中导入它们：

```js
import {ValidationController} from 'aurelia-validation'; 

```

如果你想要导入其他资源，比如 CSS 文件，只需在适当的模板中`require`它：

```js
<require from="bootstrap/css/bootstrap.css"></require> 

```

# 打包

与 CLI 或基于 Webpack 的骨架相反，基于 JSPM 的骨架在开发环境中运行时不会自动打包应用程序。但它包含了一个专门的 Gulp 任务用于打包：

```js
> gulp bundle

```

此任务将根据打包配置创建一些捆绑包。它还将更新`config.js`文件中的 SystemJS 映射，所以加载器知道从每个正确的捆绑包中加载每个模块。

这意味着，如果你手动从开发环境部署应用程序，而不是使用自动构建系统，那么在部署后你需要解包你的应用程序：

```js
> gulp unbundle

```

此命令将重置`config.js`文件中的 SystemJS 映射到其原始的未捆绑状态。然而，当运行`watch`任务时，它会自动调用，所以你不需要手动经常运行它。

## 配置捆绑包

bundling 配置可以在`build/bundles.js`文件中找到。它看起来像这样：

`build/bundles.js`

```js
module.exports = { 
  "bundles": { 
    "dist/app-bundle": { 
      "includes": [ 
        "[**/*.js]", 
        "**/*.html!text", 
        "**/*.css!text" 
      ], 
      "options": { 
        "inject": true, 
        "minify": true, 
        "depCache": true, 
        "rev": true 
      } 
    }, 
    "dist/aurelia": { 
      "includes": [ 
        "aurelia-framework", 
        "aurelia-bootstrapper", 
        // Omitted snippet... 
      ], 
      "options": { 
        "inject": true, 
        "minify": true, 
        "depCache": false, 
        "rev": true 
      } 
    } 
  } 
}; 

```

默认情况下，此配置描述了两个包：

+   `app-build`：包含从`src`目录中所有的 JS 模块、模板和 CSS 文件。

+   `aurelia`：包含 Aurelia 库、Bootstrap、fetch polyfill 和 jQuery。

`app-build`包的 JS glob 模式`[**/*.js]`周围的括号，告诉打包器忽略依赖关系。如果没有这些括号，打包器将递归地遍历每个 JS 文件的每个`import`语句，并将所有依赖关系包含在包中。由于默认的打包配置将应用程序的资源放在第一个包中，所有外部依赖放在第二个包中，所以我们不想在`app-build`包中包含任何依赖关系，因此使用了括号。

当向您的应用程序添加外部库时，您需要将其添加到包的`includes`中，通常它会在`aurelia`包中，我通常将其重命名为`vendor-bundle`。如果您不这样做，SystemJS 的映射将引用未打包的库，并尝试从`jspm_packages`目录中加载它，这在生产环境中不是我们想要的结果。

除了其内容外，包的配置还有`options`。这些选项中最有用的大概是`rev`，当设置为`true`时，启用包版本控制。因此，每个包的名称将附上一个基于内容的哈希，SystemJS 映射也将用这些版本化的包名称更新。

# 总结

在 Aurelia 的大部分开发过程中，JSPM 一直是*事实上的*包管理器，SystemJS 是首选的模块加载器；也就是说，直到 CLI 发布为止。然而，JSPM 和 SystemJS 在 Aurelia 生态系统中仍然非常重要，大多数在 CLI 发布之前启动的项目都运行在这项技术上。
