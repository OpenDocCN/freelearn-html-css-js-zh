# 第十章 生产环境的打包

将 JavaScript 应用程序部署到生产环境时，打包是一个重要的性能实践。通过将资源（主要是 JavaScript 代码、HTML 模板和 CSS 表单）合并成单个文件，我们可以大大减少浏览器为服务应用程序而必须进行的 HTTP 调用次数。

CLI 总是打包它运行的应用程序，即使在开发环境中也是如此。这使得将应用程序部署到服务器变得相当简单；只需构建它，然后复制一堆文件即可。

然而随后版本控制问题出现了。当部署我们应用程序的新版本时，如果打包文件保持相同的名称，缓存的打包文件可能不会刷新，导致用户运行我们应用程序的过时版本。我们如何处理这个问题？

在本章中，我们将了解如何自定义联系人管理应用程序的打包。我们还将了解如何利用 CLI 的修订功能对打包文件进行版本控制，以便我们可以充分利用 HTTP 缓存。最后，我们将向项目中添加一个新的构建任务，以方便部署。

# 配置打包

默认情况下，使用 CLI 创建的项目包含两个打包文件：第一个名为`vendor-bundle.js`，其中包含应用程序使用的所有外部库；第二个名为`app-bundle.js`，其中包含应用程序本身。

打包配置在`aurelia_project/aurelia.json`文件中的构建部分。以下是在典型应用程序中的样子：

```js
"bundles": [ 
  { 
    "name": "app-bundle.js", 
    "source": [ 
      "[**/*.js]", 
      "**/*.{css,html}" 
    ] 
  }, 
  { 
    "name": "vendor-bundle.js", 
    "prepend": [ 
      "node_modules/bluebird/js/browser/bluebird.core.js", 
      "scripts/require.js" 
    ], 
    "dependencies": [ 
      "aurelia-binding", 
      "aurelia-bootstrapper", 
      "aurelia-dependency-injection", 
      "aurelia-framework", 
      //Omitted snippet... 
    ] 
  } 
] 

```

每个打包文件都有一个唯一的名称，必须定义其内容，这些内容可以来自应用程序和外部依赖项。通常，`app-bundle`包括应用程序源中的所有 JS、HTML 和 CSS，而`vendor-bundle`包括外部依赖项。

这通常是小到中等应用程序的最佳配置。外部依赖项通常不会经常更改，它们被组合在它们自己的打包文件中，因此用户在新版本的应用程序发布时不需要下载这些依赖项。在大多数情况下，他们只需要下载新的`app-bundle`。

## 将应用程序合并到单一打包中

然而，如果您出于某种原因希望将应用程序及其依赖项打包成一个单一的包，这样做是相当简单的。您只需要定义一个包含应用程序源代码和外部依赖项的单一打包文件：

### 注意

以下部分代码片段摘自本书资源中的`chapter-10/samples/app-single-bundle`示例。

`aurelia_project/aurelia.json`

```js
"bundles": [ 
  { 
    "name": "app-bundle.js", 
    "prepend": [ 
      "node_modules/bluebird/js/browser/bluebird.core.js", 
      "scripts/require.js" 
    ], 
    "source": [ 
      "[**/*.js]", 
      "**/*.{css,html}" 
    ], 
    "dependencies": [ 
      "aurelia-binding", 
      "aurelia-bootstrapper", 
      //Omitted snippet... 
    ] 
  } 
] 

```

由于 Aurelia 应用程序的入口点是`aurelia-bootstrapper`库，入口点打包文件必须包含`bootstrapper`。默认情况下，这是`vendor-bundle`。如果您在此处更改入口点打包文件，它将成为`app-bundle`；您需要更改几件事。

首先，仍然在`aurelia_project/aurelia.json`的`build`下，加载器的`configTarget`属性必须改为新的入口点捆绑文件：

`aurelia_project/aurelia.json`

```js
"loader": { 
  "type": "require", 
  "configTarget": "app-bundle.js", 
  // Omitted snippet... 
}, 

```

此外，`index.html`的主要`script`标签也必须引用新的入口点捆绑文件：

`index.html`

```js
<!-- Omitted snippet... --> 
<body aurelia-app="main"> 
  <script src="img/app-bundle.js" 
          data-main="aurelia-bootstrapper"></script> 
</body> 
<!-- Omitted snippet... --> 

```

如果你在此时运行应用程序，你会看到生成了一个单一的捆绑文件，浏览器在启动应用程序时只加载这个文件。

## 将应用程序拆分为多个捆绑文件

在某些情况下，将整个应用程序源代码放在一个`app-bundle`中是不理想的。我们很容易想象一个基于高度分隔的用户故事构建的应用程序。用户，根据他们的角色，只使用这个应用程序的特定部分。

这样的应用程序可以被拆分为多个较小的捆绑文件，每个文件对应一个与角色相关的部分。这样，用户就不会下载他们从未使用的应用程序部分的捆绑文件。

以下部分中的示例是从书籍资源中的`chapter-10/samples/ app-with-home` `sample`中摘录的。

让我们尝试通过将我们应用程序中的`contacts`特性移动到其自己的捆绑文件中来尝试这个方法。为此，我们首先需要从`app-bundle`中排除`contacts`目录中的所有内容：

`aurelia_project/aurelia.json`

```js
{ 
  "name": "app-bundle.js", 
  "source": { 
    "include": [ 
      "[**/*.js]", 
      "**/*.{css,html}" 
    ], 
    "exclude": [ 
      "**/contacts/**/*" 
    ] 
  } 
} 

```

`source`属性支持数组形式的通配符模式，或者一个对象，该对象具有`include`和可选的`exclude`属性，都预期包含一个通配符模式的数组。

在这里，我们只是将`source`的先前值移动到`include`属性中，并添加一个匹配`contacts`目录中所有内容的`exclude`属性。

接下来，我们需要定义新的捆绑文件：

`aurelia_project/aurelia.json`

```js
{ 
  "name": "app-bundle.js", 
  //Omitted snippet... 
}, 
{ 
  "name": "contacts-bundle.js", 
  "source": [ 
    "[**/contacts/**/*.js]", 
    "**/contacts/**/*.{css,html}" 
  ] 
},

```

这个名为`contacts-bundle.js`的新捆绑文件将包括`contacts`目录中的所有 JS、HTML 和 CSS 文件。

如果你在此时运行应用程序，你应该首先看到`scripts`目录现在包含了三个捆绑文件：`app-bundle.js`、`contacts-bundle.js`和`vendor-bundle.js`。如果你在浏览器中打开应用程序并检查调试控制台，你应该看到在加载应用程序时，浏览器首先加载`vendor-bundle`，然后是`app-bundle`，最后是`contacts-bundle`。

当主`configure`函数在应用程序启动过程中加载`contacts`特性时，会加载`contact-bundle`。Aurelia 的特性有一个局限性：很难将一个特性隔离在一个单独的捆绑文件中。实际上，一个特性的`index`文件以及它所有的依赖项应该被包含在`app-bundle`中。将其单独打包是没有用的，因为另一个捆绑文件在启动时会被加载。然而，特性中的其他所有内容都可以单独打包。

在我们应用程序中，即使你做了这个改动，当应用程序启动时`contacts-bundle`仍然会被加载，因为`app`组件会自动将用户重定向到联系人的默认路由，即联系人列表。

如果你在应用程序中添加一个主页组件作为默认路由，并确保这个主页组件包含在`app-bundle`中，你应该可以看到只有在导航到它时才会加载`contacts-bundle`。

# 版本化捆绑包

默认情况下，捆绑包是使用静态名称生成的。这意味着一个已经缓存了捆绑包的浏览器无法知道其缓存是否最新。如果应用程序发布了新版本怎么办？

为了解决这个问题，一个（糟糕）的解决方案是设置缓存持续时间到一个很短的时间段，这会强制所有用户频繁地下载所有捆绑包，或者接受一些用户可能运行我们应用程序的过时版本，这意味着相应地管理后端、网络服务等的兼容性。这似乎是一个导致噩梦的好配方。

一个更好的解决方案是在每个捆绑包的名称中添加某种修订号，并将缓存时间设置为让`index.html`的缓存时间非常短，甚至完全禁用其缓存。由于`index.html`与捆绑包相比非常小，这是一个有趣的选择，因为每次给定用户访问应用程序时，他会下载`index.html`的最新副本，该副本又会引用最新版本的捆绑包。这意味着捆绑包可以永久缓存，因为给定捆绑包名称的内容永远不会改变。用户永远不会下载某个捆绑包版本超过一次。

Aurelia CLI 通过在文件名后添加后缀来支持捆绑包版本化。这个后缀是文件内容计算出的哈希值。默认情况下，版本化是禁用的。要启用它，请打开`aurelia_project/aurelia.json`文件，在`build`部分的`options`设置`rev`属性：

`aurelia_project/aurelia.json`

```js
"options": { 
  "minify": "stage & prod", 
  "sourcemaps": "dev & stage", 
  "rev": "stage & prod" 
}, 

```

修订机制是按环境单独启用的。通常，它会在 staging 和 production 环境中启用。然而，它不应该在开发环境中使用，因为它与浏览器重新加载以及使用`watch`开关时的捆绑重建机制不太友好。此外，由于大多数开发人员系统地在与缓存禁用的浏览器中进行测试，它将没有多大价值。

你还需要始终确保在`aurelia_project/aurelia.json`文件中，在`build`部分下`targets`的第一个条目有一个`index`属性，其值为`index.html`：

`aurelia_project/aurelia.json`

```js
"targets": [ 
  { 
    "id": "web", 
    "displayName": "Web", 
    "output": "scripts", 
    "index": "index.html" 
  } 
], 

```

这使得捆绑器知道加载应用程序的 HTML 文件的名称，因此它可以更新加载入口点捆绑的`script`标签。

现在，你可以通过在项目目录中打开控制台并运行以下命令来测试这个：

```js
> au build --env stage

```

一旦命令完成，你应该在 `scripts` 目录下看到现在包含在其名称中的哈希的包。你应该看到类似于 `app-bundle-ea03d27d90.js` 和 `vendor-bundle-efd8bd9cd8.js` 的文件，哈希可能不同。

此外，在 `index.html` 中，`script` 标签内的 `src` 属性现在应该指的是带有哈希的 `vendor-bundle` 文件名称。

# 部署应用程序

此时，部署我们的应用程序相当简单。我们需要将以下文件复制到托管它的服务器上：

+   `index.html`

+   `favicon.ico`

+   `locales/`

+   `styles/`

+   `scripts/`

+   `node_modules/bootstrap/`

+   `node_modules/font-awesome/`

现在，大多数项目都会使用某种软件工厂来构建和部署应用程序。当然，我们可以在工厂的构建任务中轻松地放置这些文件列表。然而，这意味着每次我们向该列表添加一个文件或目录时，都需要更改构建任务。

当我在一个 Aurelia 项目中工作时，我喜欢做的一件事是在 `aurelia_project/aurelia.json` 文件中创建一个新的 `deploy` 部分，将其设置为匹配部署包中要包含的文件的 glob 模式列表：

`aurelia_project/aurelia.json`

```js
{ 
  //Omitted snippet... 
  "build": { 
    //Omitted snippet... 
  }, 
  "deploy": { 
    "sources": [ 
      "index.html", 
      "favicon.ico", 
      "locales/**/*", 
      "scripts/*-bundle*.{js,map}", 
      "node_modules/bootstrap/dist/**/*", 
      "node_modules/font-awesome/{css,fonts}/**/*" 
    ] 
  } 
} 

```

除此之外，我通常还在项目中创建一个 `deploy` 任务。这个任务只是构建应用程序，然后将文件复制到要部署的目标目录，该目标目录作为任务的一个参数传递。

让我们首先创建任务定义：

`aurelia_project/tasks/deploy.json`

```js
{ 
  "name": "deploy", 
  "description": "Builds, processes and deploy all application assets.", 
  "flags": [ 
    { 
      "name": "out", 
      "description": "Sets the output directory (required)", 
      "type": "string" 
    }, 
    { 
      "name": "env", 
      "description": "Sets the build environment (uses debug by default).", 
      "type": "string" 
    } 
  ] 
} 

```

接下来，让我们创建一个 `copy` 任务，该任务将由 `deploy` 任务使用：

`aurelia_project/tasks/copy.js`

```js
import gulp from 'gulp'; 
import {CLIOptions} from 'aurelia-cli'; 
import project from '../aurelia.json'; 

export default function copy() { 
  const output = CLIOptions.getFlagValue('out', 'o'); 
  if (!output) { 
    throw new Error('--out argument is required'); 
  } 

  return gulp.src(project.deploy.sources, { base: './' }) 
    .pipe(gulp.dest(output)); 
} 

```

这个任务首先检索作为 `out` 参数传递的目标目录，如果省略则失败，然后使用来自 `aurelia_project/aurelia.json` 中新 `deploy` 部分的 glob 模式列表，并将每个匹配的文件复制到提供的目标目录中。

最后，我们可以创建部署任务本身：

`aurelia_project/tasks/deploy.js`

```js
import gulp from 'gulp'; 
import build from './build'; 
import copy from './copy'; 

export default gulp.series( 
  build, 
  copy 
); 

```

这个任务只是依次执行 `build` 和 `copy`。我们甚至可以在 `build` 和 `copy` 之间运行单元测试任务。

这个 `gulp` 任务极大地简化了软件工厂中的构建任务。典型的软件工厂构建过程首先从版本控制中检出代码，然后运行以下命令：

```js
> npm install
> au deploy --env $(env) --out $(build-artifacts)

```

最后，它会将 `$(build-artifacts)` 下的所有内容复制到 Web 服务器上。

在这个场景中，`$(env)` 和 `$(build-artifacts)` 是一些环境或系统变量。第一个包含了构建所针对的环境，比如 `stage` 或 `prod`，而第二个包含了一些临时文件夹，从中复制要部署到 Web 服务器的工件。例如，它可能仅仅是工作目录中的一个 `dist` 文件夹。

这种解决方案的一个优点是，现在与构建和部署我们的应用程序相关的大多数细节都在项目本身之内。软件工厂不再依赖于应用程序源代码的文件结构和文件名，而是仅依赖于`gulp`任务。

# 总结

由于命令行界面（CLI）始终以捆绑模式运行应用程序，所以最初看起来部署 Aurelia 应用程序相当简单。然后你开始考虑 HTTP 缓存过期的问题，事情就变得有点复杂了。

幸运的是，CLI 已经提供了解决这些问题的工具。再加上一些良好实践，使将应用程序准备部署到现实世界变得足够简单。
