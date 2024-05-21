# 第五章：库和框架

本章将探讨应用程序如何与库和框架一起工作。许多框架和库与 Webpack 一起工作。通常，这些是 JavaScript 框架。它们正变得越来越成为编程的核心部分，了解如何将它们集成到应用程序捆绑包中可能会成为一个日益增长的需求。

与其他 Webpack 元素的工作有些不同，使用库和框架。通过典型的例子和用例，本书将探讨 Angular 以及如何构建 Angular 框架以便进行包捆绑。这包括对 Webpack 捆绑的期望，期望的结果，优势和局限性。

完成本章后，您应该能够自信地使用这些主要框架和库与 Webpack 一起使用。您还将知道如何集成和安装它们，以及如何在集成中使用最佳实践。

在本章中，我们将涵盖以下主题：

+   最佳实践

+   使用 JSON

+   使用 YAML

+   使用 Angular

+   使用 Vue.js

# 最佳实践

到目前为止，我们只涵盖了构建 Vanilla JavaScript，这不应与 Vanilla 框架混淆。尽管这是学习的最佳方式，但更有可能的是您将使用某种框架。Webpack 将与任何 JavaScript 或 TypeScript 框架一起工作，包括 Ionic 和 jQuery；然而，更棘手的框架包括 Angular、Vue 和 YAML。

现在，我们将开始使用 YAML，但在深入研究之前，您可能想知道是否可以集成后端框架。简单的答案是可以，但它们不会被捆绑。然而，唯一的集成级别是通过链接源代码，就像我们在大多数项目或 API 中所做的那样，比如 REST API。

正如我们已经讨论过的，Webpack 有生产模式和开发模式。生产模式将您的项目捆绑成最终状态，准备好进行网络交付或发布，并且提供了很少的调整空间。开发模式给开发人员自由修改数据库连接的权限；这是后端集成的方式。您的项目的后端可能是**ASP.NET**或**PHP**，但有些后端更复杂，使用`OpenAuth`。作为开发人员，您需要对所有这些有一个概览。然而，本指南只涉及 Webpack。

请放心，所有这些框架都将集成，这是通过 REST API 完成的，它以**JavaScript 对象表示**（**JSON**）格式返回数据。也可以使用**AJAX**来完成这一点。无论如何，请确保遵循安全的最佳实践，因为与使用服务器端脚本相比，对数据库的 JSON 调用并不安全。

如果您的项目使用 Ionic，那么您应该按照 Angular 的说明进行操作，因为 Ionic 框架是基于此的。

这应该为您提供了与后端和库一起工作的最佳实践的全面概述。现在我们将讨论您在 Webpack 中会遇到的每个常见库。让我们从 JSON 开始，因为它是最容易理解的，也是外部或后端代码和数据库与您的 Webpack 应用程序交互的最重要方式。

# 使用 JSON

当您使用框架时，大部分时间您需要在不同语言和应用程序之间进行通信。这是通过 JSON 完成的。JSON 在这方面与 YAML 类似，但更容易理解 Webpack 如何与 JSON 一起工作。

JSON 文件可以被 Webpack 的编译器理解，无需专用加载程序，因此可以被视为 Webpack 捆绑器的本地脚本，就像 JavaScript 一样。

到目前为止，本指南已经提到，JSON 文件在包组合中起着重要作用。Webpack 记录和跟踪加载器和依赖项的使用是通过 JSON 文件的模式进行的。这通常是`package.json`文件，有时是`package.lock.json`文件，它记录了每个安装包的确切版本，以便可以重新安装。在这种情况下，“包”是指加载器和依赖项的集合。

每个 JSON 文件必须正确编程，否则 Webpack 将无法读取。与 JavaScript 不同，代码中不允许使用注释来指导用户，因此您可能希望使用`README`文件向用户解释其内容和目的。

JSON 文件的结构与 JavaScipt 有所不同，并包含不同的元素数组，例如键。以以下代码块为例：

```js
module: {
    rules: [{
    test: /\.yaml$/,
    use: 'js-yaml-loader',
    }]
}
```

这是`package.json`文件的一部分，我们稍后在本章中将使用。此块的内容基本上声明了模块的参数。命名模块用作键，并在其后加上冒号。这些键有时被称为名称，它们是选项的放置位置。此代码设置了一系列规则，在这个规则中是指示 Webpack 在转译内容模块时使用`js-yaml-loader`。

您必须确保大括号和方括号的使用顺序正确，否则 Webpack 将无法读取代码。

作为 JavaScript 开发人员，您可能对 JSON 及其用法非常熟悉。但是，为了排除任何盲点，值得详细说明。YAML 是一个更复杂的框架，但您经常会遇到它，因此它只比 JSON 逐渐更复杂。现在让我们开始逐步了解它。

# 使用 YAML

YAML 是一种常见的框架，类似于 JSON 的使用方式。不同之处在于 YAML 更常用于配置文件，这就是为什么在使用 Webpack 时您可能会更频繁地或者第一次遇到它。

要在 Webpack 中使用 YAML，我们必须安装它。让我们开始吧：

1.  您可以在命令行实用程序中使用`npm`安装 YAML。使用以下命令：

```js
yarn add js-yaml-loader
```

请注意`yarn`语句。这是指 JavaScript 文件的开源包管理器，预装在`Node.js`中。它的工作方式类似于`npm`，应该是预安装的。如果您在此处使用的代码没有得到响应，请再次检查是否已预安装。

1.  要从命令行界面检查 YAML 文件，您应该全局安装它们：

```js
npm install -g js-yaml

```

1.  接下来，打开配置文件`webpack.config.js`，并进行以下修改：

```js
import doc from 'js-yaml-loader!./file.yml';
```

前一行将返回一个 JavaScript 对象。这种方法适用于不受信任的数据。请参阅*进一步阅读*部分，了解 GitHub YAML 示例。

1.  之后，使用`webpack.config.js`配置文件来允许使用 YAML 加载器：

```js
module: {
    rules: [{
    test: /\.yaml$/,
    use: 'js-yaml-loader',
    }]
}
```

您可能还想为 Webpack 使用 YAML front-matter 加载器。这是一个 MIT 许可的软件，它将 YAML 文件转换为 JSON，如果您更习惯于使用后者，这将特别有用。如果您是 JavaScript 开发人员，您很可能习惯使用 JSON，因为它往往比 YAML 更常用于 JavaScript。

此模块需要在您的设备上安装 Node v6.9.0 和 Webpack v4.0.0 的最低版本。Webpack 5 是本指南的主题，所以不应该有问题。但是，请注意，此功能仅适用于 Webpack 4 和 5。

以下步骤与前面的步骤分开，因为它们涉及安装`yaml-loader`而不是`yaml-frontmatter`，后者用于将 YAML 转换为 JSON 文件（这更符合 Webpack 包结构的典型情况）：

1.  首先，您需要使用命令行实用程序安装`yaml-frontmatter-loader`：

```js
npm install yaml-frontmatter-loader --save-dev
```

这个特定的命令行可能在语法上与本指南过去展示的命令行不同，但无论格式如何，这个命令都应该有效。

1.  然后，按照以下方式向配置中添加加载器：

```js
const json = require('yaml-frontmatter-loader!./file.md');
```

此代码将`file.md`文件作为 JavaScript 对象返回。

1.  接下来，再次打开`webpack.config.js`，并对`rules`键进行以下更改，确保引用 YAML 加载器：

```js
module.exports = {
  module: {
    rules: [
      {
         test: /\.md$/,
         use: [ 'json-loader', 'yaml-frontmatter-loader' ]
      }
    ]
  }
}
```

1.  接下来，通过你喜欢的方法运行 Webpack 5，并查看结果！

如果你一口气通过了这一点，你可能已经足够勇敢去应对 Angular 了。这是一个更难处理的框架，所以让我们开始吧。

# 使用 Angular

Angular 是一个库和框架，与所有框架一样，它旨在使构建应用程序更容易。Angular 利用依赖注入、集成最佳实践和端到端工具，所有这些都可以帮助解决开发中的挑战。

Angular 项目通常使用 Webpack。在撰写本文时，使用的最新版本是**Angular 9**。每年都会推出更新版本。

现在，让我们看看 Webpack 在捆绑 Angular 项目或将 Angular 添加到现有项目时的工作原理。

Angular 寻找`window.jQuery`来确定是否存在 jQuery。查看以下源代码的代码块：

```js
new webpack.ProvidePlugin({
  'window.jQuery': 'jquery'
});
```

要添加`lodash`映射，请将现有代码附加到以下内容：

```js
new webpack.ProvidePlugin({
  _map: ['lodash', 'map']
});
```

Webpack 和 Angular 通过提供入口文件并让它包含从这些入口点导出的依赖项来工作。以下示例中的入口点文件是应用程序的根文件`src/main.ts`。让我们来看看：

1.  我们需要在这里使用`webpack.config.js`文件。请注意，这是一个单入口点过程：

```js
entry: {
 'app': './src/main.ts'
},
```

Webpack 现在将解析文件，检查它，并递归地遍历其导入的依赖项。

1.  在`src/main.ts`中进行以下更改：

```js
import { Component } from '@angular/core';
@Component({
 selector: 'sample-app',
 templateUrl: './app.component.html',
 styleUrls: ['./app.component.css']
})
export class AppComponent { }
```

Webpack 将识别出正在导入`@angular/core`文件，因此这将被添加到捆绑包含的依赖项列表中。Webpack 将打开`@angualar/core`文件，并跟踪其一系列`import`语句，直到从中构建出一个依赖图。这将从`main.ts`（一个 TypeScript 文件）构建出来。

1.  然后，将这些文件作为输出提供给在配置中标识的`app.js`捆绑文件：

```js
output: {
 filename: 'app.js'
}
```

包含源代码和依赖项的 JavaScript 文件是一个文件，输出捆绑文件是`app.js`文件。稍后将在`index.html`文件中使用 JavaScript 标签(`<script>`)加载它。

建议不要为所有内容创建一个巨大的捆绑文件，这是显而易见的。因此，建议将易变的应用程序代码与更稳定的供应商代码模块分开。

1.  通过更改配置，将应用程序和供应商代码分开，现在使用两个入口点——`main.ts`和`vendor.ts`，如下所示：

```js
entry: {
 app: 'src/app.ts',
 vendor: 'src/vendor.ts'
},
output: {
 filename: '[name].js'
}
```

通过构建两个单独的依赖图，Webpack 会发出两个捆绑文件。第一个称为`app.js`，而第二个称为`vendor.js`。分别包含应用程序代码和供应商依赖项。

在上面的示例中，`file name: [name]`表示一个占位符，Webpack 插件 app 和 vendor 将其替换为入口名称。插件将在下一章节中详细介绍，所以如果你遇到困难，也许可以标记这一页，然后再回来看看。

1.  现在，通过添加一个仅导入第三方模块的`vendor.ts`文件，指示 Webpack 哪些部分属于供应商捆绑，如下所示：

```js
import '@angular/platform-browser';
import '@angular/platform-browser-dynamic';
import '@angular/core';
import '@angular/common';
import '@angular/http';
import '@angular/router';
// RxJS
import 'rxjs';
```

请注意提到`rxjs`。这是一个用于响应式编程的库，旨在使开发人员更容易地组合异步代码或回调。

其他供应商也可以以这种方式导入，例如 jQuery、Lodash 和 Bootstrap。还可以导入的文件扩展名包括 **JavaScript** 文件（`.js`）、**TypeScript** 文件（`.ts`）、**层叠样式表** 文件（`.css`）和 **Syntactically Awesome Style Sheets** 文件（`.sass`）。

Angular 可能是一个非常复杂的框架，并且与基于 Web 的应用程序非常相关。然而，您的特定需求可能更适合单页应用程序，这种情况下，Vue.js 将是大多数人首选的选择。现在让我们来看一下它。

# 使用 Vue.js

Vue.js 是另一个框架，但是它是开源的。它的使用显著性，或者说明显的目的领域，是**单页应用**（**SPAs**）。这是因为该框架专注于提供无缝的体验，但功能比 Angular 少，可以与许多其他语言一起工作，并且运行速度非常快。使用 Vue.js 构建相当大的应用程序会导致在使用过程中加载非常缓慢，甚至编译速度更慢。

也许理解这一点的最好方法是考虑 jQuery 如何使用内联语句调用页面中的脚本，而 Angular 使用核心模块，每个模块都设计有特定的目的。Vue.js 介于纯粹简单的 jQuery 和 Angular 之间。

使用 Webpack 与 Vue.js 项目是通过使用专用加载器来完成的。

建议安装 `vue-loader` 和 `vue-template-compiler`，除非您是 Vue.js 模板编译器的高级用户。按照以下步骤进行：

1.  要按照这个示例进行，首先安装 `vue-loader` 和 `vue-template-compiler`，使用以下代码：

```js
npm install -D *vue-loader vue-template-compiler*
```

模板编译器必须单独安装，以便可以指定版本。

Vue.js 每次发布新版本都会发布相应版本的模板编译器。这两个版本必须同步，以便加载器生成的代码是运行时兼容的。因此，每次升级一个版本时，应该升级另一个版本。

与 Vue.js 相关的加载器与大多数加载器的配置略有不同。在处理扩展名为 `.vue` 的文件时，确保将 Vue.js 加载器的插件添加到您的 Webpack 配置中。

1.  这是通过修改 Webpack 的配置来完成的，下面的示例使用 `webpack.config.js`：

```js
const VueLoaderPlugin = require('vue-loader/lib/plugin')
module.exports = {
 module: {
 rules: [
 // other rules
 {
   test: /\.vue$/,
   loader: 'vue-loader'
  }
 ]
},
 plugins: [
 // make sure to include the plugin.
 new VueLoaderPlugin()
 ]
}
```

这个插件是必需的，因为它负责克隆已定义的文件，并将它们应用于与 `.vue` 文件对应的语言块。

1.  使用 Vue.js，我们添加以下内容：

```js
new webpack.ProvidePlugin({ Vue: ['vue/dist/vue.esm.js', 'default'] });

```

先前的代码必须添加，因为它包含了 ECMAScript 模块的完整安装，用于与打包工具一起使用 Vue.js。这应该存在于项目的 `npm` 包的 `/dist` 文件夹中。注意 `.ems.` 表示 ECMAScript 模块。Vue.js 安装页面上显示了运行时和生产特定的安装方法，这些方法可以在本章的 *进一步阅读* 部分找到。**UMD** 和 **CommonJS** 的安装方法类似，分别使用 `vue.js` 和 `vue.common.js` 文件。

由于我们的项目将使用 `.esm` 格式，了解更多关于它可能会很有用。它被设计为静态分析，这允许打包工具执行**摇树**，即消除未使用的代码。请注意，打包工具的默认文件是 `pkg.module`，它负责运行时 ECMAScript 模块编译。有关更多信息，请参阅 Vue.js 安装页面——URL 可在本章的 *进一步阅读* 部分找到。

这就结束了关于框架和库的内容。现在，您应该已经具备了处理复杂项目的强大能力，甚至可能会使用多个框架。

# 总结

本章涵盖了典型的框架以及如何开始使用它们。这包括应该遵循的安装过程以及需要的标准和外围设备。本指南关注最佳实践和安全性。在开始项目时，您应该提前遵循这些示例，密切关注程序、警告和要求。

本指南为您概述了其他框架，如用于回调的 RxJS 和 jQuery，以及在使用不寻常的文件扩展名时指引您正确方向。我们还探讨了在使用 Webpack 5 时 Angular 的核心功能和 Vue.js 的用法和安装程序，以及 Vue.js 如何更适合单页面应用程序，而 Angular 在更大的项目上的工作效果更好。

在涵盖了大部分核心主题之后，下一章我们将深入探讨部署和安装。在处理数据库并确保满足安全要求时，这将变得更加重要。下一章将对这个主题进行深入探讨，并希望能解决您作为开发人员可能遇到的任何问题。

# 进一步阅读

+   GitHub 的 YAML 示例：[`github.com/nodeca/js-yaml`](https://github.com/nodeca/js-yaml)

+   window.jQuery 源代码：[`github.com/angular/angular.js/blob/v1.5.9/src/Angular.js#L1821-L1823`](https://github.com/angular/angular.js/blob/v1.5.9/src/Angular.js#L1821-L1823.)

+   Vue.js 安装指南：[`vuejs.org/v2/guide/installation.html`](https://vuejs.org/v2/guide/installation.html)

# 问题

1.  与正在使用的`Vue.js`版本对应的编译器是什么？

1.  在使用 Angular 时，本指南建议将易变代码和稳定供应商代码分开。这是通过使用两个入口点来实现的。它们是什么？

1.  在使用 Webpack 时，使用 YAML 的最低安装要求是什么？

1.  为什么应该全局安装 YAML 文件？

1.  什么是 SPA？

1.  处理`.vue`文件时，应该在哪里添加 Vue.js 的加载器？

1.  以下代码行缺少什么？

`import 'angular/http';`

1.  在使用 Angular 时，如何加载`app.js`？

1.  什么是 YARN？

1.  默认的`pkg.module`文件用于什么？
