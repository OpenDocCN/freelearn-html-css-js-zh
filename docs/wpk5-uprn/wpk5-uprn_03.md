# 第三章：使用配置和选项

本章将包括配置和选项的实际用法，以及它们在任何给定构建中的相互关系和作用。它还将详细说明输出管理，也就是捆绑过程的输出和资产管理，以及作为依赖图的一部分的资产。这将涵盖文件放置和文件结构等子主题。

模块用于解决 JavaScript 具有全局函数的特性。Webpack 与这些模块一起工作，并隔离了变量和函数的暗示全局性质。

配置和选项是必要的，以便充分利用 Webpack。每个项目都是定制的，因此每个项目都需要对其参数进行特定的定制。本章将详细探讨这两个主题的确切性质，每个主题的限制以及何时使用它们。

本章讨论以下主题：

+   理解配置

+   理解资产管理

+   理解输出管理

+   探索 Webpack 5 的选项

# 理解配置

通过使用配置文件，在 Webpack 中进行配置通常是`webpack.config.js`，除非在特殊情况下可以有一个以上的文件分配给这个任务。在`webpack.config.js`的情况下，它是一个 JavaScript 文件，应该被修改以改变任何特定项目的配置设置。

在启动时，Webpack 和 Webpack 5 不需要配置文件，但软件会将`src/index`识别为默认项目输入。它还将结果输出到名为`dist/main.js`的位置。这个输出将被"缩小"并优化为生产环境。

*缩小*，或*最小化*，简单地指的是 Webpack 的主要功能之一：将使用的代码量减少到最小。这是通过消除重复、错误或多余的代码来实现的。

然而，一个 Webpack 项目通常需要改变其默认配置。默认配置是 Webpack 在没有任何加载器或特殊参数的情况下运行的方式，比如在第一章中描述的*Webpack 5 简介*的*Webpack 工作原理*子部分。这是通过使用配置文件来完成的。开发人员应该创建一个名为`webpack.config.js`的文件，并将其放在项目的根文件夹中。这个文件将被 Webpack 自动检测和读取。

让我们通过探索使用多个配置文件来开始我们的讨论。

# 使用不同的配置文件

Webpack 5 提供了使用不同配置文件的选项，具体取决于情况。不仅如此，还可以使用命令行实用程序来更改正在使用的文件。在一个项目中使用多个捆绑包时，可能会遇到这种情况，这个主题将在本指南的后面详细介绍。以下代码片段显示了开发人员如何更改正在使用的配置文件。在这个例子中，一个文件被指向一个名为`package.json`的文件，这是 Webpack 经常使用的一个常见文件。这种技术被称为*config flag*：

```js
"scripts": {
  "build": "webpack --config example.config.js" }
```

请注意，Webpack 5 还允许自定义配置，正如在第一章中所解释的*Webpack 5 简介*，这是使用 Webpack 5 的一个显著优势。这是通过使用自定义配置文件来完成的。这与选项不同，因为这些变量不是使用**命令行界面**（**CLI**）设置的。

# 使用选项

在 Webpack 中，*选项*指的是通过命令行而不是配置文件进行的设置，这是通过修改配置脚本来完成的。

在下面的例子中，我们将首先修改配置文件，简单地为我们的选项教程奠定基础。

在接下来的配置中，Node 的**路径模块**被使用，并且前缀是`_dirname`全局变量。Node 的路径模块只是 Node 用于处理文件或目录路径的实用程序。在操作系统之间工作时可能会出现文件路径问题，这可以防止这些问题发生，并确保相对路径正常工作。

示例中涉及的文件名为`webpack.config.js`。我们将用它来设置项目的模式，并且我们需要在到达选项之前这样做：

```js
const path = require('path');

module.exports = {
  mode: "production", // "production" | "development" | "none"
  entry: "./app/entry", // string | object | array
```

在前面的代码块中，所选择的**模式**指示 Webpack 相应地使用其内置的优化。**entry**路径默认为`./src`。这是应用程序执行开始和捆绑开始的地方。

下面的代码块将显示相同文件的其余部分：

```js
output: {
  path: path.resolve(__dirname, "dist"), // string
 filename: "bundle.js", // string
  publicPath: "/assets/", // string
  library: "MyLibrary", // string,
 libraryTarget: "umd", // universal module definition
  },
```

代码片段的这一部分显示了与 Webpack 发出结果相关的选项。

所有输出文件的目标目录必须是绝对路径（使用**Node.js**路径模块）。

`filename`指示入口块的文件名模板，`publicPath`是**统一资源定位符**（**URL**），指的是相对于相关 HTML 页面解析到输出目录的路径。简而言之，这意味着从您可能使用的 HTML 页面到捆绑项目文件的文件路径。代码的其余部分涉及导出库的名称和导出库的性质。

接下来的主题涉及与模块相关的配置。在处理输出选项之后，这将是项目开发中的下一个逻辑步骤：

```js
module: { 
   rules: [
        {
        test: /\.jsx?$/,
        include: [
          path.resolve(__dirname, "app")
        ],
        exclude: [
          path.resolve(__dirname, "app/demo-files")
        ],
```

前面的代码块包括了对模块的规则，比如解析器选项和加载器的配置。这些都是匹配条件，每个都接受一个字符串或正则表达式。术语`test`的行为与`include`相同。它们都必须匹配，但对于`exclude`来说并非如此。`exclude`优先于`test`和`include`选项。

为了最佳实践，`RegExp`应该只在文件名匹配时用于`test`。在使用路径数组时，应优先使用绝对路径而不是`include`和`exclude`选项。`include`选项应优先于`exclude`方法：

```js
issuer: { test, include, exclude },
        enforce: "pre",
        enforce: "post",
        loader: "babel-loader",
        options: { presets: ["es2015"] },
},
      {
        test: /\.html$/,
        use: [ "htmllint-loader",
{
            loader: "html-loader",
            options: {
              / *...* /
            }
          }
        ]
      },
```

前面的代码块包括了对发行者和导入元素的来源的条件。代码还包括了标记这些规则的应用的选项，即使它们被覆盖了。然而，这是一个高级选项。

对`loader`的引用指示应用哪个加载器。这是相对于上下文位置解析的。自 Webpack 2 以来，加载器后缀不再是可选的，为了清晰起见。还有空间应用多个其他选项和加载器。

在相同的配置中，我们将探讨可以在同一过程中应用的规则和条件，如下面的代码块所示：

```js
{ oneOf: [ / rules / ] },
{ rules: [ / rules / ] },
{ resource: { and: [ / conditions / ] } }, 
{ resource: { or: [ / conditions / ] } },
{ resource: [ / conditions / ] },
{ resource: { not: / condition / } }],
    /* Advanced module configuration */
  },
  resolve: {
```

前面的代码块包括了嵌套规则，所有这些规则都与条件结合在一起是有用的。解释一下，注意以下每个命令及其表示的含义：

+   `and`选项只有在所有条件也匹配时才会进行匹配。

+   `or`匹配在条件匹配时应用——这是数组的默认值。

+   `not`指示条件是否不匹配。

还有一个选项用于解析模块请求；这不适用于解析加载器。以下示例显示了使用此`resolve`模块请求：

```js
modules: [
      "node_modules",
      path.resolve(__dirname, "app")
    ], extensions: [".js", ".json", ".jsx", ".css"], 
    alias: { 
              "module": "new-module",
              "only-module$": "new-module",
              "module": path.resolve(__dirname, "app/third/module.js"),
           },
},

  performance: {
    hints: "warning", // enum
    maxAssetSize: 200000, // int (in bytes),
    maxEntrypointSize: 400000, // int (in bytes)
    assetFilter: function(assetFilename) {
    return assetFilename.endsWith('.css') || assetFilename.endsWith('.js');
    }
  },
```

前面的代码块显示了我们在本节中一直在遵循的相同配置文件。然而，让我们看一下一些关键元素。在`path.resolve`处，这指的是要查找模块的目录。直接下面的`], extensions:`指的是使用的文件扩展名。

在此部分之后是代码，按降序列出模块名称的别名列表。模块的别名是相对于当前位置上下文导入的，如下面的代码块所示：

```js
devtool: "source-map", // enum
context: __dirname, // string (absolute path!)
target: "web", // enum
externals: ["react", /^@angular/],
serve: { //object
    port: 1337,
    content: './dist',
    // ...
  },
stats: "errors-only",
```

`devtool`配置通过为浏览器添加元数据来增强调试。请注意，`source-map`选项可能更详细，但这是以构建速度为代价的，`web`选项指示 Webpack 的主目录。入口和`module.rules.loader`选项相对于此目录解析，并指的是捆绑包应该运行的环境。`serve`配置允许您为`webpack-serve`提供选项，并精确控制显示哪些捆绑信息，例如以下内容：

```js
devServer: { proxy: { // proxy URLs to backend development server '/api': 'http://localhost:3000' },
    contentBase: path.join(__dirname, 'public'), 
    compress: true, 
    historyApiFallback: true, 
    hot: true, 
    https: false, 
    noInfo: true, 

  },
  plugins: [

  ],
  // list of additional plugins
```

让我们解释前面的代码块。当它声明`compress: true`时，这启用了内容的**gzip**压缩。`historyApiFallback: true`部分是当遇到任何 404 页面加载错误时为真。`hot: true`文本指的是是否允许热模块替换；这取决于是否首先安装了`HotModuleReplacementPlugin`。`https`应设置为`true`以用于自签名对象或证书授权对象。如果`noInfo`键设置为`true`，则只会在热重新加载时获得错误和警告。

配置完成，现在可以运行构建。要做到这一点，使用以下命令：

```js
npx webpack-cli init
```

一旦在命令行环境中运行了前面的代码，用户可能会被提示安装`@webpack-cli/init`，如果它尚未安装在项目中。

运行`npx webpack-cli init`后，根据配置生成期间所做的选择，可能会在项目中安装更多的包。以下代码块显示了运行 NPX Webpack 的 CLI 初始化时的输出：

```js
npx webpack-cli init

 INFO For more information and a detailed description of each question, have a look at https://github.com/webpack/webpack-cli/blob/master/INIT.md
 INFO Alternatively, run `webpack(-cli) --help` for usage info.

 Will your application have multiple bundles? No
 Which module will be the first to enter the application? [default: ./src/index]
 Which folder will your generated bundles be in? [default: dist]:
 Will you be using ES2015? Yes
 Will you use one of the below CSS solutions? No

  babel-plugin-syntax-dynamic-import@6.18.0
  uglifyjs-webpack-plugin@2.0.1
  webpack-cli@3.2.3
  @babel/core@7.2.2
  babel-loader@8.0.4
  @babel/preset-env@7.1.0
  webpack@4.29.3
  added 124 packages from 39 contributors, updated 4 packages and audited 
  25221 packages in 7.463s
  found 0 vulnerabilities

Congratulations! Your new webpack configuration file has been created!

```

如果你在 CLI 中的输出看起来像前面的代码块，那么你的配置就成功了。这基本上是从命令行自动读取的，应该表示在前面的代码块中设置的所有选项都已记录。

我们已经通过配置和选项，你现在应该知道每个选项的区别和使用范围。现在自然而然地转向资产管理。

# 理解资产管理

资产主要通过依赖图进行管理，我们在第一章中已经介绍过，*Webpack 5 简介*。

在 Webpack 伟大的出现之前，开发人员会使用诸如**grunt**和**gulp**之类的工具来处理这些资产，并将它们从源文件夹移动到生产目录或开发目录（通常分别命名为`/build`和`/dist`）。

JavaScript 模块也使用了相同的原则，但 Webpack 5 会动态捆绑所有依赖项。由于每个模块都明确声明了它的依赖项，未使用的模块将不会被捆绑。

在 Webpack 5 中，除了 JavaScript 之外，现在还可以包含任何其他类型的文件，使用加载器。这意味着使用 JavaScript 时可能的所有功能也可以被利用。

在接下来的小节中，我们将探讨实际的资产管理。将涵盖以下主题：

+   为资产管理配置设置项目

+   加载**层叠样式表**（**CSS**）文件

+   加载图像

+   加载字体

+   加载数据

+   添加全局资产

然后，将有一个小节来总结。

每个小节都将有步骤和指导内容要遵循。这可能是一个相当大的主题，所以紧紧抓住！我们将从准备项目的配置开始。

# 为资产管理配置设置项目

为了在项目中设置资产管理配置，我们需要通过以下步骤准备我们的项目索引和配置文件：

1.  首先，通过使用`dist/index.html`文件对示例项目进行微小的更改，如下所示：

```js
  <!doctype html>
  <html>
    <head>
    <title>Asset Management</title>
    </head>
    <body>
     <script src="img/bundle.js"></script>
    </body>
  </html>
```

1.  现在，使用`webpack.config.js`编写以下内容：

```js
  const path = require('path');

  module.exports = {
    entry: './src/index.js',
    output: {
     filename: 'bundle.js',
     path: path.resolve(__dirname, 'dist')
    }
  };
```

前面的两个代码块只是显示了一个占位符索引文件，我们将用它来进行资产管理的实验。后一个代码块显示了一个标准配置文件，将索引文件设置为第一个入口点，并设置输出捆绑包的名称。这将在我们完成资产管理实验后为我们的项目准备捆绑。

您的项目现在已经设置好了资产管理配置。本指南现在将向您展示如何加载 CSS 文件。

# 加载 CSS 文件

示例项目现在将显示 CSS 的包含。这是一个非常容易掌握的事情，因为大多数从 Webpack 5 开始的前端开发人员应该对它很熟悉。

要加载 CSS 并运行构建，请执行以下步骤：

1.  首先，使用以下命令行指令将`style-loader`和`css-loader`安装并添加到项目的模块配置中：

```js
npm install --save-dev style-loader css-loader
```

1.  接下来，向`webpack.config.js`文件添加以下内容：

```js
  const path = require('path');

  module.exports = {
    entry: './src/index.js',
    output: {
      filename: 'bundle.js',
      path: path.resolve(__dirname, 'dist')
    },
   module: {
     rules: [
       {
         test: /\.css$/,
         use: [
           'style-loader',
           'css-loader'
         ]
       }
     ]
   }
  };
```

从前面的代码块中可以看出，以下添加是指向代码块末尾的`style-loader`和`css-loader`的使用。为了避免出现错误，您应该确保您的代码与示例相符。

`style-loader`和`css-loader`之间的区别在于前者确定样式将如何被注入到文档中，比如使用样式标签，而后者将解释`@import`和`require`语句，然后解析它们。

建议同时使用这两个加载程序，因为几乎所有**CSS**操作在项目开发的某个阶段都涉及这些方法的组合。

在 Webpack 中，正则表达式用于确定应该查找哪些文件并将其提供给特定的加载程序。这允许将样式表导入到依赖它进行样式设置的文件中。当运行该模块时，一个带有字符串化 CSS 的`<style>`标签将被插入到 HTML 文件的`<head>`中。

1.  现在，导航到目录结构，我们可以在以下示例中看到：

```js
webpack5-demo 
package.json 
webpack.config.js 
/dist 
bundle.js 
index.html 
/src  
style.css 
index.js 
/node_modules
```

从这个结构中我们可以看到有一个名为`style.css`的样式表。我们将使用这个来演示`style-loader`的使用。

1.  在`src/style.css`中输入以下代码：

```js
.hello {
  color: blue;
}
```

上面的代码只是创建了一个颜色类样式，我们将使用它来附加样式到我们的前端，并展示 CSS 加载的工作原理。

1.  同样，将以下内容追加到`src/index.js`中：

```js
  import _ from 'lodash';
  import './style.css';

  function component() {
    const element = document.createElement('div');

    // Lodash, now imported by this script
    element.innerHTML = _.join(['Hello', 'Webpack'], ' ');
    element.classList.add('hello');

    return element;
  }

  document.body.appendChild(component());
```

前面的代码都发生在`index.js`文件中。它基本上创建了一个 JavaScript 函数，该函数在从浏览器调用它的任何文件中插入一个`<div>`元素。在这个示例中，它将是`index.html`文件，在目录结构示例中提到的。前面的代码将在网页上“连接”一个**HTML**元素，其中包含文本“`Hello, Webpack`”。我们将使用这个来测试`style-loader`和`css-loader`是否被正确使用。正如脚本的注释部分所述，这个元素附加将自动导入`lodash`以便与 Webpack 一起使用。

1.  最后，运行`build`命令，如下所示：

```js
npm run build

...
    Asset      Size  Chunks             Chunk Names
bundle.js  76.4 KiB       0  [emitted]  main
Entrypoint main = bundle.js
...
```

当在浏览器窗口中打开`index.html`文件时，您应该看到“`Hello Webpack`”现在以蓝色样式显示。

要查看发生了什么，请检查页面（不是页面源代码，因为它不会显示结果），并查看页面的头标签。最好使用谷歌的 Chrome 浏览器进行。它应该包含我们在`index.js`中导入的样式块。

您可以并且在大多数情况下应该最小化 CSS 以获得更好的生产加载时间。

下一个自然的步骤是开始添加图片。图片可以以与任何网站应用程序相同的方式添加到项目中。将这些图片放在图像文件夹中以任何所需的格式。这必须在`/src`文件夹中，但它们可以放在其中的任何位置。下一个步骤是使用 Webpack 加载图片，我们现在将进行这一步。

# 加载图片

现在，让我们尝试使用文件加载程序加载图像和图标，这可以很容易地整合到我们的系统中。

要做到这一点，执行以下步骤：

1.  使用命令行，安装`file-loader`，如下所示：

```js
npm install --save-dev file-loader
```

1.  现在，使用通常的`webpack.config.js`Webpack 配置文件，对其进行以下修改：

```js
  const path = require('path');
  module.exports = {
    entry: './src/index.js',
    output: {
      filename: 'bundle.js',
      path: path.resolve(__dirname, 'dist')
    },
    module: {
      rules: [
        {
          test: /\.css$/,
          use: [
            'style-loader',
            'css-loader'
          ]
        },
        {
          test: /\.(png|svg|jpg|gif)$/, 
          use: [  
             'file-loader'
          ]
        }
      ]
    }
  };
```

现在，由于前面的代码块中的代码，当您导入图像时，该图像将被处理到输出目录，并且与该图像相关联的变量将在处理后包含该图像的最终**URL**。当使用`css-loader`时，类似的过程将发生在**CSS**文件中图像文件的**URL**上。加载程序将识别这是一个本地文件，并将本地路径替换为输出目录中图像的最终路径。**`html-loader`**以相同的方式处理`<img src="img/my-image.png" />`。

1.  接下来，要开始添加图像，您需要导航到项目文件结构，看起来像这样：

```js
  webpack5-demo
  |- package.json
  |- webpack.config.js
  |- /dist
    |- bundle.js
    |- index.html
  |- /src
 |- icon.png
    |- style.css
    |- index.js
  |- /node_modules
```

这个结构看起来与之前的项目非常相似，直接用于大部分“加载 CSS 文件”教程，只是增加了`icon.png`图像文件。

1.  然后，导航到 JavaScript 前端文件`src/index.js`。以下代码块显示了内容：

```js
import _ from 'lodash'; import './style.css';  
import Icon from './icon.png'; 
function component() { 
    const element = document.createElement('div'); 
    // Lodash, now imported by this script 
        element.innerHTML = _.join(['Hello', 'Webpack'], ' '); 
        element.classList.add('hello');  
    // Add the image to our existing div.  
    const myIcon = new Image(); myIcon.src = Icon; 
    element.appendChild(myIcon); 
    return element; 
} 
document.body.appendChild(component());
```

从前面的代码块可以看出，导入**lodash**将允许您的页面的**HTML**附加`Hello Webpack`文本。除此之外，这段代码只是用一些巧妙的 JavaScript 设置了我们的网页和图像。它首先创建一个名为`Icon`的变量，并为其赋予图像文件的**URL**的值。在代码的后面，它将这个值分配给一个名为`myIcon`的元素的源。

1.  从这里开始，我们想要设置一些非常基本的样式来处理我们的图像。在`src/style.css`文件中，追加以下代码：

```js
  .hello {
    color: red;
    background: url('./icon.png');
  }
```

当然，它将显示您的图像图标作为我们在**HTML**中分配代码的`div`的背景，其中应用了`.hello`类的地方文本变为**红色**。

1.  运行新的构建并打开`index.html`文件，如下所示：

```js
npm run build

...
Asset                                 Size          Chunks         Chunk Names
da4574bb234ddc4bb47cbe1ca4b20303.png  3.01 MiB          [emitted]  [big]
bundle.js                             76.7 KiB       0  [emitted]         main
Entrypoint main = bundle.js
...
```

这将创建图标重复作为背景图像的效果。在`Hello Webpack`文本旁边还会有一个`img`元素。

通常，即使对于经验丰富的开发人员，这个命令也可能出错。例如，图像可能根本不加载，太大，或者无法正确捆绑。这可能是由多种因素造成的，包括以不寻常的方式使用加载程序。在使用长文件名时，Webpack 也可能会出现代码跳过的情况。

如果是这种情况，只需重复以下步骤：

1.  使用命令行安装`file-loader`。

1.  按照前面的示例修改`webpack.config.js`文件。

1.  检查项目文件结构和索引文件是否正确格式化以加载图像文件。

1.  检查**CSS**是否也按您的要求格式化。

1.  然后，使用`npm`和命令行运行构建。

1.  检查索引文件是否正确加载图像。

如果检查该元素，可以看到实际的文件名已更改为类似于`da4574bb234ddc4bb47cbe1ca4b20303.png`的内容。这意味着 Webpack 在源文件夹中找到了我们的文件并对其进行了处理。

这为您提供了一个管理图像的坚实框架。在下一小节中，我们将讨论 Webpack 资产的字体管理。

# 加载字体

现在，我们将在资产的上下文中检查字体。文件和 URL 加载程序将接受通过它们加载的任何文件，并将其输出到您的构建目录。这意味着我们可以将它们用于任何类型的文件，包括字体。

我们将首先更新 Webpack 配置 JavaScript 文件，需要处理字体，如下所示：

1.  确保更新配置文件。我们在这里更新了通常的`webpack.config.js`配置文件，但您会注意到在末尾添加了一些字体类型，例如`.woff`、`.woff2`、`.eot`、`.ttf`和`.otf`，如下面的代码块所示：

```js
  const path = require('path');

  module.exports = {
    entry: './src/index.js',
    output: {
      filename: 'bundle.js',
      path: path.resolve(__dirname, 'dist')
    },
    module: {
      rules: [
        {
          test: /\.css$/,
          use: [
            'style-loader',
            'css-loader'
          ]
        },
        {
          test: /\.(png|svg|jpg|gif)$/,
          use: [
            'file-loader'
          ]
        },
       {
         test: /\.(woff|woff2|eot|ttf|otf)$/,
         use: [
           'file-loader'
         ]
       }
      ]
    }
  };
```

此配置允许 Webpack 的`file-loader`合并字体类型，但我们仍然需要向项目添加一些字体文件。

1.  现在，我们可以执行将字体添加到源目录的基本任务。下面的代码块说明了文件结构，指示新字体文件可以添加的位置：

```js
  webpack5-demo
  |- package.json
  |- webpack.config.js
  |- /dist
    |- bundle.js
    |- index.html
  |- /src
    |- sample-font.woff
    |- sample-font.woff2
    |- icon.png
    |- style.css
    |- index.js
  |- /node_modules
```

注意`src`目录和`sample-font.woff`和`sample-font.woff2`文件。这两个文件应该被您选择的任何字体文件替换。**Web Open Font**（**WOFF**）格式通常建议与 Webpack 项目一起使用。

通过使用`@font-face`声明，可以将字体合并到项目的样式中。Webpack 将以与处理图像相同的方式找到本地 URL 指令。

1.  使用`src/style.css`文件更新样式表，以在我们的主页上包含示例字体。这是通过在代码块顶部使用字体声明和在下面使用类定义来完成的，如下面的代码块所示：

```js
 @font-face {
    font-family: 'SampleFont';
    src:  url('./sample-font.woff2') format('woff2'),
          url('./sample-font.woff') format('woff');
    font-weight: 600;
    font-style: normal;
  }

  .hello {
    color: blue;
 font-family: 'SampleFont';
    background: url('./icon.png');
  }
```

请注意，您必须将`'SampleFont'`文本更改为与您选择的字体文件相对应的文本。前面的代码显示了通过 CSS 加载字体以及设置自定义值，如`font-weight`和`font-style`。**CSS**代码然后使用`.hello`类将该字体分配给任何潜在的**HTML**元素。请注意，我们在前两个教程中已经为此准备好了我们的`index.html`文件，*加载 CSS* *文件*和*加载图像*。

1.  现在，像往常一样使用命令行实用程序以开发模式运行`npm`构建，如下所示：

```js
npm run build

...
                                 Asset      Size  Chunks                    Chunk Names
5439466351d432b73fdb518c6ae9654a.woff2  19.5 KiB          [emitted]
 387c65cc923ad19790469cfb5b7cb583.woff  23.4 KiB          [emitted]
  da4574bb234ddc4bb47cbe1ca4b20303.png  3.01 MiB          [emitted]  [big]
bundle.js                                 77 KiB       0  [emitted]         main
Entrypoint main = bundle.js
...
```

再次打开`index.html`，看看我们使用的`Hello Webpack`示例文本是否已更改为新字体。如果一切正常，您应该能看到变化。

这应该作为一个简单的教程来理解字体管理。下一节将涵盖文件的数据管理，如**可扩展标记语言**（**XML**）和**JavaScript 对象表示**（**JSON**）文件。

# 加载数据

另一个有用的资源是数据。数据是一个非常重要的要加载的资源。这将包括**JSON**、**逗号分隔值**（**CSV**）、**制表符分隔值**（**TSV**）和**XML**文件等文件。使用诸如`import Data from './data.json'`这样的命令默认情况下可以工作，这意味着**JSON**支持内置到 Webpack 5 中。

要导入其他格式，必须使用**加载器**。以下子节演示了处理所有三种格式的方法。应采取以下步骤：

1.  首先，您必须使用以下命令行安装`csv-loader`和`xml-loader`加载器。

```js
npm install --save-dev csv-loader xml-loader
```

前面的代码块只是显示了安装两个数据加载器的命令行。

1.  打开并追加`webpack.config.js`配置文件，并确保其看起来像以下示例：

```js
  const path = require('path');

  module.exports = {
    entry: './src/index.js',
    output: {
      filename: 'bundle.js',
      path: path.resolve(__dirname, 'dist')
    },
    module: {
      rules: [
        {
          test: /\.css$/,
          use: [
            'style-loader',
            'css-loader'
          ]
        },
        {
          test: /\.(png|svg|jpg|gif)$/,
          use: [
            'file-loader'
          ]
        },
        {
          test: /\.(woff|woff2|eot|ttf|otf)$/,
          use: [
            'file-loader'
          ]
        },
        {
          test: /\.(csv|tsv)$/,
          use: [
            'csv-loader'
          ]
        },
        {
          test: /\.xml$/,
          use: [
            'xml-loader'
          ]
        }
      ]
    }
  };
```

在前面的代码块中，下部显示了`csv-loader`和`xml-loader`的使用。这次需要进行的修改是将数据加载到我们的项目中。

1.  接下来，我们必须向源目录添加一个数据文件。我们将在我们的项目中添加一个**XML**数据文件，如下面代码块中的粗体文本所示：

```js
  webpack5-demo
  |- package.json
  |- webpack.config.js
  |- /dist
    |- bundle.js
    |- index.html
  |- /src
    |- data.xml
    |- samplefont.woff
    |- sample-font.woff2
    |- icon.png
    |- style.css
    |- index.js
  |- /node_modules
```

查看您**项目**文件夹的`src`目录中的前述`data.xml`文件。让我们更仔细地查看一下这个文件里的数据，如下所示：

```js
<?xml version="1.0" encoding="UTF-8"?>
<note>
  <to>Tim</to>
  <from>Jakob</from>
  <heading>Reminder</heading>
  <body>Call me tomorrow</body>
</note>
```

从前面的代码块中可以看出，内容是一个非常基本的**XML**数据集。我们将使用它来导入**XML**数据到我们项目的`index.html`页面中，并且需要正确格式化以确保其正常工作。

这四种类型的数据（**JSON**、**CSV**、**TSV**和**XML**）中的任何一种都可以被导入，并且您导入的`data`变量将包含解析后的 JSON。

1.  确保修改`src/index.js`文件以公开数据文件。注意`./data.xml`的导入，如下面的代码块所示：

```js
  import _ from 'lodash';
  import './style.css';
  import Icon from './icon.png';
  import Data from './data.xml';

  function component() {
    const element = document.createElement('div');

    // Lodash, now imported by this script
    element.innerHTML = _.join(['Hello', 'Webpack'], ' ');
    element.classList.add('hello');

    // Add the image to our existing div.
    const myIcon = new Image();
    myIcon.src = Icon;

    element.appendChild(myIcon);

    console.log(Data);

    return element;
  }

  document.body.appendChild(component());
```

这次我们只需要添加`import`函数，几乎没有别的东西，来演示使用方法。熟悉 JavaScript 的人也会知道如何轻松地运行他们的特定项目。

1.  运行构建并检查数据是否正确加载，方法如下：

```js
npm run build
```

运行`npm`构建后，可以打开`index.html`文件。检查控制台（例如在 Chrome 中使用**开发者工具**）将显示导入后记录的数据。

与项目架构相关的是为项目消耗安排全局资产的方式。让我们在下一小节中探讨这一点。

# 添加全局资产

以前述方式加载资产允许模块以更直观、实用和可用的方式进行分组。

与包含每个资产的全局资产目录不同，资产可以与使用它们的代码分组。以下文件结构或树演示了一个非常实用和可用的示例：

```js
  |- /assets
  |– /components
  |  |– /my-component |  |  |– index.jsx
  |  |  |– index.css
  |  |  |– icon.svg
  |  |  |– img.png
```

前面的例子使您的代码更具可移植性。如果您想将一个组件放在另一个目录中，只需将其复制或移动到那里。或者，如果您的开发工作遵循老式的方式，也可以使用基本目录。此外，别名也是一个选择。

# 用最佳实践结束教程

这是一个漫长的教程，你的一些代码可能已经出错了。清理这些代码并检查是否有任何错误是一个好习惯。

清理是一个好习惯。在接下来的部分中，*理解输出管理*，我们不会使用很多资产，所以让我们从那里开始。

1.  我们开始用项目目录**项目树**结束。让我们检查它们是否正确。它应该看起来像下面这样：

```js
  webpack5-demo
  |- package.json
  |- webpack.config.js
  |- /dist
    |- bundle.js
    |- index.html
  |- /src
    |- data.xml
    |- sample-font.woff
    |- sample-font.woff2
    |- icon.png
    |- style.css
    |- index.js
  |- /node_modules
```

在结束时，您应该删除与前面代码块中加粗文本相对应的文件。

这应该让你对项目文件和文件夹的外观有一个很好的了解。确保我们一直在使用的所有文件都在那里，并且在适当的文件夹中。

1.  让我们检查我们配置的格式。

在`webpack.config.js`上已经做了很多工作，我们必须确保内容格式正确。请参考以下代码块，并将其与您自己的代码进行对比，以确保正确。通常有用的是计算`{`的数量，并使用传统结构美化您的代码，以使这个过程更容易：

```js
  const path = require('path'); module.exports = {
    entry: './src/index.js',
    output: {
      filename: 'bundle.js',
      path: path.resolve(__dirname, 'dist')
    },
    module: {
      rules: [
        {
          test: /\.css$/,
          use: [
            'style-loader',
            'css-loader'
          ]
        },
        {
          test: /\.(png|svg|jpg|gif)$/,
          use: [
            'file-loader'
          ]
        },
        {
          test: /\.(woff|woff2|eot|ttf|otf)$/,
          use: [
            'file-loader'
          ]
        },
        {
          test: /\.(csv|tsv)$/,
          use: [
            'csv-loader'
          ]
        },
        {
          test: /\.xml$/,
          use: [
            'xml-loader'
          ]
        }
      ]
    }
  };
```

注意到对 CSS、图像文件、诸如`.woff`的字体以及独立处理程序中的数据文件（如`.csv`和`.xml`）的广泛引用。所有这些都很重要，您应该花时间确保脚本准确，因为这是一个广泛的主题和实际练习，所以很多东西可能被忽视。

1.  接下来，我们需要检查`src/index.js`文件的脚本，方法如下：

```js
  import _ from 'lodash';
 import './style.css';
  import Icon from './icon.png';
  import Data from './data.xml';

  function component() {
    const element = document.createElement('div');

 // Lodash, now imported by this script
    element.innerHTML = _.join(['Hello', 'Webpack'], ' ');
 element.classList.add('hello');

    // Add the image to our existing div.
    const myIcon = new Image();
    myIcon.src = Icon;

    element.appendChild(sampleIcon);

    console.log(Data);

    return element;
  }

  document.body.appendChild(component());
```

再次，我们在这里结束，以便在使用多个教程后代码是可重用的，所以请确保在您的版本中删除加粗的文本。

我们已经经历了一系列的资产管理操作，并以项目整理过程结束。为了使其正常运行，您的所有代码应该看起来像包装部分中的以前的代码块。

现在您应该对 Webpack 如何管理这些资产以及在使用 Webpack 时如何管理它们有了清晰的理解。通过整理文件结构和代码，我们现在可以开始输出管理。

# 理解输出管理

输出是指从源文件创建的包。源文件在 Webpack 中被称为输入。输出管理指的是对这些新打包文件的管理。根据 Webpack 在构建开始时运行的模式，这些包将是开发包还是生产包。

Webpack 从源文件生成输出或包的过程称为编译。编译是 Webpack 5 组装信息（包括资产、文件和文件夹）的过程。配置的主题涉及 Webpack 中可能的各种选项和配置，这些选项和配置将改变编译的样式和方法。

开发包允许一些定制（例如本地测试），但生产包是成品和完全压缩的版本，准备发布。

在本章中，资产已经手动添加到了**HTML**文件中。随着项目的发展，手动处理将变得困难，特别是在使用多个包时。也就是说，存在一些插件可以使这个过程变得更容易。

现在我们将讨论这些选项，但首先要准备您现在非常繁忙的项目结构，这将成为项目发展中越来越重要的实践。

# 输出管理教程准备

首先，让我们稍微调整一下项目文件结构树，使事情变得更容易。这个过程遵循以下步骤：

1.  首先，找到项目文件夹中的`print.js`文件，如下所示：

```js
  webpack5-demo
  |- package.json
  |- webpack.config.js
  |- /dist
  |- /src
    |- index.js |- print.js  |- /node_modules
```

注意我们项目结构的添加——特别是`print.js`文件。

1.  通过向`src/print.js`文件添加一些逻辑来追加代码，如下所示：

```js
export default function printIt() {
  console.log('This is called from print.js!');
}
```

您应该在`src/index.js`文件中使用`printIt()`JavaScript 函数，就像前面的代码块中所示。

1.  准备`src/index.js`文件，以导入所需的外部文件，并在其中编写一个简单的函数以允许交互，如下所示：

```js
  import _ from 'lodash';
  import printMe from './print.js';

  function component() {
    const element = document.createElement('div');
    const btn = document.createElement('button');

    element.innerHTML = _.join(['Hello', 'Webpack'], ' ');

    btn.innerHTML = 'Click here then check the console!';
    btn.onclick = printIt();

    element.appendChild(btn);

    return element;
  }

  document.body.appendChild(component());
```

我们已经更新了我们的`index.js`文件，在顶部导入了`print.js`文件，并在底部添加了一个新的`printIt();`函数按钮。

1.  我们必须更新`dist/index.html`文件。这次更新是为了准备拆分条目，并在下面的代码块中进行了说明：

```js
  <!doctype html>
  <html>
    <head>
      <title>Output Management</title>
      <script src="img/print.bundle.js"></script>
    </head>
    <body>
      <script src="img/app.bundle.js"></script>
    </body>
  </html>
```

前面的**HTML**脚本将加载`print.bundle.js`文件，以及下面的`bundle.js`和`app.bundle.js`文件。

1.  接下来，确保项目的配置符合动态入口点。`src/print.js`文件将被添加为新的入口点。输出也将被更改，以便根据入口点名称动态生成包的名称。在`webpack.config.js`中，由于这个自动过程，不需要更改目录名称。下面的代码块显示了`webpack.config.js`的内容：

```js
  const path = require('path');

  module.exports = {
    entry: './src/index.js',
    entry: {
      app: './src/index.js',
      print: './src/print.js'
    },
    output: {
      filename: 'bundle.js',
      filename: '[name].bundle.js',
      path: path.resolve(__dirname, 'dist')
    }
  };
```

配置简单地为我们正在工作的新文件`index.js`和`print.js`设置了新的入口点。

1.  确保您执行了构建。一旦您运行了`npm`构建，您将会看到以下内容：

```js
...
Asset           Size      Chunks                  Chunk Names
app.bundle.js   545 kB    0, 1  [emitted]  [big]  app
print.bundle.js  2.74 kB  1     [emitted]         print
...
```

在浏览器中打开`index.html`文件后，您会看到 Webpack 生成了`print.bundle.js`和`app.bundle.js`文件。我们现在应该检查它是否工作了！如果更改了入口点名称或添加了新的入口点，**index HTML**仍然会引用旧的名称。这可以通过`HtmlWebpackPlugin`来纠正。

# 设置 HtmlWebpackPlugin

`HtmlWebpackPlugin`将允许 Webpack 处理包含 JavaScript 的 HTML 文件。要开始使用它，我们需要使用命令行安装它，然后正确设置配置，如下所示：

1.  首先，使用命令行实用程序安装插件，然后调整`webpack.config.js`文件，如下所示：

```js
npm install --save-dev html-webpack-plugin
```

前面的代码块显示了在我们的项目中使用`HtmlWebpackPlugin`的安装。

1.  接下来，我们需要将插件合并到我们的配置中。让我们看一下与该插件相关联的`webpack.config.js`文件，如下所示：

```js
  const path = require('path');
 const HtmlWebpackPlugin = require('html-webpack-plugin');

  module.exports = {
    entry: {
      app: './src/index.js',
      print: './src/print.js'
    },
    plugins: [
 new HtmlWebpackPlugin({
        title: 'Output Management'
      })
    ],
    output: {
      filename: '[name].bundle.js',
      path: path.resolve(__dirname, 'dist')
    }
  };
```

注意`require`表达式和`plugins:`选项键的使用，这两者都允许使用插件。

在运行构建之前，请注意`HtmlWebpackPlugin`将默认生成它的`index.html`文件，即使`dist/`文件夹中已经有一个。因此，现有文件将被覆盖。

为了最佳实践，复制现有的索引文件并将其命名为`index2.html`。将这个新文件放在原文件旁边，然后运行构建。

1.  现在，使用命令行实用程序运行构建。一旦完成，你将在命令行实用程序窗口中看到以下结果，表明成功捆绑：

```js
...
           Asset       Size  Chunks                    Chunk Names
 print.bundle.js     544 kB       0  [emitted]  [big]  print
   app.bundle.js    2.81 kB       1  [emitted]         app
      index.html  249 bytes          [emitted]
...
```

打开代码编辑器或**记事本**中的`index.html`文件将会显示插件已经创建了一个新文件，并且所有的捆绑包都被自动添加了。

另外，为什么不看一下`html-webpack-template`，它在默认模板的基础上提供了一些额外的功能呢？

这就结束了我们对 Webpack 的`HtmlWebpackPlugin`的教程。在接下来的小节中，我们将再次开始整理你的项目目录。

# 清理分发目录

在项目开发过程中，`/dist`文件夹会变得相当混乱。良好的做法涉及良好的组织，这包括在每次构建之前清理`/dist`文件夹。有一个`clean-webpack-plugin`插件可以帮助你做到这一点，如下所示：

1.  首先安装`clean-webpack-plugin`。以下示例向你展示如何做到这一点：

```js
npm install --save-dev clean-webpack-plugin 
```

插件安装完成后，我们可以重新进入配置文件。

1.  使用**`webpack.config.js`**，在文件中进行以下条目：

```js
  const path = require('path');
  const HtmlWebpackPlugin = require('html-webpack-plugin');
 const CleanWebpackPlugin = require('clean-webpack-plugin');

  module.exports = {
    entry: {
      app: './src/index.js',
      print: './src/print.js'
    },
    plugins: [
 new CleanWebpackPlugin(),
      new HtmlWebpackPlugin({
        title: 'Output Management'
      })
    ],
    output: {
      filename: '[name].bundle.js',
      path: path.resolve(__dirname, 'dist')
    }
  };
```

注意使用`CleanWebpackPlugin`，继续使用`const`限定符。这将是`module.export`插件选项的添加，它将创建一个与插件相关联的新函数，并使插件在 Webpack 编译期间可用。

1.  现在你应该运行一个`npm`构建，这将会将一个捆绑包输出到`/dist`分发文件夹中。

运行`npm`构建后，可以检查`/dist`文件夹。假设过程正常，你应该只看到新生成的文件，不再有旧文件。

我们已经生成了很多文件，为了帮助我们跟踪，有一个叫做清单的东西，我们接下来会介绍。

# 利用清单

Webpack 可以通过清单知道哪些文件正在生成。这使得软件能够跟踪所有输出捆绑包并映射模块。为了以其他方式管理输出，利用清单是个好主意。

Webpack 基本上将代码分为三种类型：开发人员编写的源代码；第三方编写的供应商代码；以及 Webpack 的运行时清单，它负责所有模块的交互。

运行时和清单数据是 Webpack 在浏览器中运行时连接你的模块化应用程序所需的。

如果你决定通过使用浏览器缓存来提高性能，这个过程将成为一个重要的事情。

通过在捆绑文件名中使用内容哈希，你可以告诉浏览器文件内容已经改变，从而使缓存失效。这是由运行时和清单的注入引起的，它们会随着每次构建而改变。

Webpack 有`WebpackManifestPlugin`，可以将清单数据提取到一个**JSON**文件中。

现在你已经了解了动态向你的 HTML 中添加捆绑包，让我们深入开发指南。或者，如果你想深入更高级的主题，我们建议重新阅读上一章节第二章中的*代码拆分*部分。

# 探索 Webpack 5 的选项

选项是可以使用 CLI 进行更改的一组变量。另一方面，通过更改文件内容来进行配置。但是可以使用配置文件来调整选项的设置。以下是当前由 Webpack 5 支持的选项列表：

+   **异步模块定义**（**AMD**）

+   Bail

+   Cache

+   Loader

+   Parallelism

+   Profile

+   Records Path

+   Records Input Path

+   记录输出路径

+   Name

以下各节将更详细地描述和说明每个选项。我们将从一些东西开始，经过对 Webpack 配置的粗略检查后，可能会让你感到困惑：AMD 选项。

# AMD

**AMD**是一个`object bool: false`选项。它也是**异步模块定义**的缩写。本质上，它是为开发人员提供模块化 JavaScript 解决方案的格式。该格式本身是一个用于定义模块的提案，其中模块和依赖项都可以异步加载。

这允许您设置`require.amd`或`define.amd`的值。将`amd`设置为`false`将禁用**AMD**支持。

查看`webpack.config.js`文件，如下所示：

```js
module.exports = {
  amd: {
    jQuery: true
  }
};
```

**AMD**的流行模块，例如 jQuery 版本 1.7.0 至 1.9.1，只有在加载程序指示允许在同一页上使用多个版本时才会注册为**AMD**模块。另一个类似的选项，就布尔变量而言，是**Bail**。让我们仔细看一下。

# Bail

**Bail**是一个`bool`值。这将强制 Webpack 退出其捆绑过程。这将导致 Webpack 在第一个错误上失败，而不是容忍它。默认情况下，Webpack 将在终端（使用**HMR**时也在浏览器控制台）中以红色文本记录这些错误，但将继续捆绑。

要启用此选项，请打开`webpack.config.js`，如下所示：

```js
module.exports = {
  bail: true
};

```

如果您希望 Webpack 在某些情况下退出捆绑过程，这将非常有帮助。也许您只想要项目的一部分捆绑。这完全取决于您。接下来是缓存。

# Cache

缓存是一个指代`bool`对象的术语。它将缓存生成的 Webpack 模块和块；这提高了构建的速度。它通过在编译器调用之间保持对此对象的引用来实现。在**观察模式**和开发模式下，默认情况下启用缓存。

在观察模式下，在初始构建之后，Webpack 将继续监视任何已处理文件的更改。基本上，**Webpack 配置 JavaScript**文件应该在`module.export`运算符内包含`watch: true`操作数。

要启用缓存，可以在 webpack.config.js 中手动将其设置为`true`，如下例所示：

```js
module.exports = {
  cache: false
};
```

`webpack.config.js`文件显示了允许共享缓存所需的配置，如下所示：

```js
let SharedCache = {};

module.exports = {
  cache: SharedCache
};
```

前两个示例显示了缓存配置设置为`false`和`sharedCache`。这是 Webpack 中可以设置的两个布尔值。

警告：缓存不应在不同选项的调用之间共享。

Webpack 中还有一些可以设置的选项：Loader、Parallelism、Profile、Records Path、Records Input Path、Records Output Path 和 Name。让我们逐个进行解释，现在就开始吧。

# Loader

这被表示为`loader`，并在以下代码块中展示了在 loader 上下文中公开自定义值：

```js
use: [
    {
       loader: worker-loader
    }
]
```

您可以从前面的代码示例中看到如何在配置文件中使用此选项。这个示例对于遵循本指南并配置加载程序的任何人来说应该很熟悉。此示例仅使用`worker-loader`作为示例。其中一些选项是布尔值或二进制值，例如接下来要描述的`profile`选项。

# Profile

`profile`选项将捕获应用程序的配置文件，然后可以使用**Analyze**工具进行分析，如下面的代码片段所示：

```js
profile: true,
```

请注意，这是一个布尔值。您可以使用`StatsPlugin`来更好地控制配置文件。这也可以与`parallelism`选项结合使用以获得更好的结果。

# 并行性

并行性将限制并行处理模块的数量。这可以用于微调性能或更可靠的分析。以下示例将限制数字为`1`，但您可以根据需要更改：

```js
parallelism: 1
```

Webpack 5 允许并行处理模块以及并行捆绑。这可能会占用内存，因此应该在较大的项目上注意此选项。

随着项目变得更加复杂，您可能希望记录编译过程，这可以帮助跟踪错误和错误。**Records Path**将帮助您做到这一点，我们现在将更仔细地看一下。

# Records Path

Records Path 选项表示为字符串。应该使用此选项来生成包含记录的**JSON**文件。这些是用于在多个构建之间存储模块标识符的数据片段。这可以用于跟踪模块在构建之间的变化。要生成一个，只需指定一个位置，如下面的示例中使用`webpack.config.js`文件：

```js
module.exports = {
  recordsPath: path.join(__dirname, 'records.json')
};
```

如果您有一个使用代码拆分的复杂项目，记录非常有用。这些记录的数据可用于确保在处理拆分包时缓存的行为是否正确。

尽管编译器生成此文件，但应使用源代码控制来跟踪它，并随时间保留其使用历史记录。

设置`recordsPath`也会将`recordsInputPath`和`recordsOutputPath`设置为相同的位置。

# 记录输入路径

此选项表示为字符串`recordsInputPath`，如下面的代码块所示：

```js
module.exports = { 
recordsInputPath: path.join(__dirname, 'records.json'), 
};
```

它将指定从中读取最后一组记录的文件，并可用于重命名记录文件。相关的是 Records Output Path 选项，我们现在将对其进行讨论。

# 记录输出路径

Records Output Path 是一个**字符串**，用于指定记录应写入的位置。以下代码示例显示了如何在重命名记录文件时结合使用此选项与`recordsInputPath`。我们将使用`webpack.config.js`来完成这个操作：

```js
module.exports = {
  recordsInputPath: path.join(__dirname, 'records.json'),
  recordsOutputPath: path.join(__dirname, 'outRecords.json')
};
```

上述代码将设置记录写入的位置。如果是输入记录，它将被写入`__dirname/records.json`。如果是输出记录，它将被写入`__dirname/newRecords.json`。

我们需要讨论的下一个选项是 Name 选项。

# 名称

**Name**选项表示为**字符串**，表示配置的名称。在加载多个配置时应使用它。以下示例显示了应该成为`webpack.config.js`文件的一部分的代码：

```js
module.exports = {
  name: 'admin-app'
};
```

在使用多个配置文件时，上述代码非常有用。该代码将将此配置文件命名为`admin-app`。这为您提供了一份选项的详细清单以及如何使用它们。现在让我们回顾一下本章中涵盖的内容。

# 总结

本章遵循了配置文件、资产管理和选项的实践。本章开始时带领读者了解了 Webpack 和配置的各种功能，并探讨了如何管理这些资产并相应地控制内容。您被引导了解了输入和输出管理，以及加载外部内容，如字体和图像。从那里，本章带领我们了解了选项和两者之间的区别，并向读者解释了可以通过简单配置实现的选项可以实现的内容。

然后，您将通过常见的选项方法以及如何使用它们进行了指导。您现在已经完全了解了选项和配置。您现在应该知道两者之间的区别以及采用的最佳方法，无论可能需要哪种技术。

在下一章中，我们将深入研究 API 的加载器和插件世界。Webpack 的这些功能阐述了平台的能力，从配置和选项中跳跃出来。

您将了解加载器和插件之间的区别，以及加载器使用默认不支持的语言和脚本的基本性质。许多这些加载器是由第三方开发人员提供的，因此插件填补了加载器无法使用的功能差距，反之亦然。

然后将扩展 API 的类似主题。API 基本上用于将应用程序连接到网络上的远程应用程序。这使它们具有与加载器类似的特征，并且它们经常用于本机脚本不可用的地方。

# 问题

为了帮助你学习，这里有一组关于本章涵盖的主题的问题（你会在本指南的后面找到答案）：

1.  Webpack 5 中配置和选项的区别是什么？

1.  配置标志是什么？

1.  加载图像到 Webpack 项目需要哪个加载器？

1.  Webpack 允许导入哪种类型的数据文件而不使用加载器？

1.  Webpack 的清单记录表示什么？

1.  Bail 选项是做什么的？

1.  并行选项是做什么的？

1.  Records Input Path 选项是做什么的？

1.  将 AMD 设置为`false`会做什么？

1.  什么是编译？
