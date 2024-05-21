# 第四章：API、插件和加载器

**应用程序编程接口**（**API**）通常用于远程站点程序之间的接口，例如当公司通过移动应用程序部分访问其网站功能作为集成系统的一部分时。

Webpack 旨在编译和优化本地化代码，因此了解本地化代码和外部 API 之间的区别对于操作软件至关重要。

插件和加载器类似。加载器基本上指示 Webpack 如何处理与更不寻常的编程语言和捆绑相关的特定任务。加载器通常由用户社区开发，而不是内部 Webpack 开发人员开发。另一方面，插件提供了一些加载器目前不提供的过程，因此在其操作上比加载器更通用。本章将在课程中提供每个功能的简明解释和详细示例。

Webpack 5 提供了丰富的插件接口。Webpack 中的大多数功能都使用这个插件接口，使得 Webpack 非常灵活。

本章将探讨插件、加载器和 API，以及每个的重要性以及每个功能在 Webpack 操作中的作用。

本章讨论的主题如下：

+   加载器

+   API

+   插件

# 加载器

加载器对于 Webpack 是基础的，其中许多加载器使得更多功能成为可能，特别是对于不是原生 ECMAScript 的脚本和框架，比如 JavaScript 和 JSON。

本章旨在为您提供可用加载器的广泛概述，以及您可能需要购买的一些加载器。当处理与您的项目特定的显著或独特的代码时，您应该搜索 Webpack 在线注册表，以确保代码可以被转译。

特别是，本节将讨论以下加载器：

+   `cache-loader`

+   `coffee-loader`

+   `coffee-redux-loader`

+   `worker-loader`

+   `cover.js`

+   `i18n-loader`

+   `imports-loader`

+   `polymer-webpack-loader`

+   `script-loader`

+   `source-map-loader`

+   `less-loader`

我们将讨论并举例说明每个加载器，如果适用的话，尽管有些可能不需要真正的详细说明。让我们从 `cache-loader` 开始。

# cache-loader

缓存是我们在上一章中提到的内容。`cache-loader` 允许从加载器创建缓存。我们可以按照以下方式设置它：

1.  首先通过**命令行界面**（**CLI**）安装加载器，如下所示：

```js
npm install --save-dev cache-loader
```

请注意，我们的配置中执行了其他加载器（请参见以下代码），并且将由以 `cache-loader` 开头的任何加载器生成的任何输出进行缓存。默认情况下，这将在项目文件夹中进行，但也可以配置为在数据库中进行。

1.  要配置此内容，请使用 `webpack.config.js`：

```js
module.exports = {
 module: {
  rules: [
  {
    test: /\.ext$/,
    use: ['cache-loader', 'babel-loader'],
    include: path.resolve('src'),
   },
  ],
 },
};
```

请注意，`cache-loader` 放置在配置中的位置应该始终放在其他加载器之前。

这解释了安装和使用 `cache-loader`。我们将以相同的方式介绍其他几个加载器，从 `worker-loader` 开始。虽然它总是与其他加载器以链式或序列的方式使用，但由于它需要首先声明，因此我们应该首先讨论它。

# worker-loader

`worker-loader` 本质上为开发人员提供了一个解决方案，用于在后台处理大型计算任务。要使加载器运行起来，我们将采取以下步骤：

1.  首先，使用命令行安装 `worker-loader`：

```js
npm install worker-loader --save-dev
```

另外，还有一种内联方式从 `App.js` 文件导入 `worker-loader`。在任何 Webpack 项目目录中的这个文件中，如果还没有进行以下修改，可以进行以下修改：

```js
import Worker from 'worker-loader!./Worker.js';
```

1.  一旦导入了加载器，就使用 `webpack.config.js` 进行配置：

```js
{
 module: {
  rules: [
  {
   test: /\.worker\.js$/,
   use: { 
      loader: 'worker-loader' 
      }
    }
   ]
  }
 }
```

`use` 指的是允许访问加载器所需的配置。

1.  在 `App.js` 中编写以下代码，以允许该文件成为加载器的导出位置：

```js
import Worker from './file.worker.js';
const worker = new Worker();
worker.postMessage({ a: 1 });
worker.onmessage = function (event) {};
worker.addEventListener("message", function (event) {});
```

前面的代码还添加了一个`event`监听器，以便以后在开发控制台中进行测试。

1.  最后，通过您喜欢的方法运行 Webpack 以查看结果：

```js
npm run build
```

如果您选择了该方法，您应该看到`worker-loader`已安装并从`App.js`导入。这可以在控制台窗口中观察到，也可以通过查看页面源代码来观察到。

这为您提供了两种使用`worker-loader`的选择，可以通过命令行实用程序或通过`App.js`文件的配置。接下来，我们将讨论`coffee-loader`。

# coffee-loader

CoffeeScript 是 JavaScript 的简化形式，但它并不完全是 JavaScript，因此必须使用加载器才能在 Webpack 中使用它。

让我们按照以下步骤来使用`coffee-loader`：

1.  首先安装`coffee-loader`。要安装加载器，请使用以下命令行：

```js
npm install --save-dev coffee-loader
```

1.  确保您正在使用推荐的加载器配置进行测试，并使用`literate`键。为了进行测试，加载`coffee-loader`并将`literate`键设置为`true`。`literate`键将确保加载器的使用由编译器解释：

```js
module.exports = {
    module: {
      rules: [
       {
          test: /\.coffee.md$/,
          use: [{
          loader: 'coffee-loader',
          options: {
            literate: true
         }
      }]
    }]
  }
}
```

上述示例中的代码显示了如何使用加载器以及设置新规则。

1.  如果需要，我们将向您展示如何安装`coffee-redux`。**Redux**是用于管理 JavaScript 应用程序状态的开源库。它经常与**React**和**Angular**等库一起使用。要安装它，请键入以下命令：

```js
npm i -D coffee-redux-loader
```

上述示例不仅将帮助您了解在包中安装和使用 CoffeeScript 的过程，还将帮助您了解这里未提及的加载器的工作原理，因为它们的工作方式基本相同。

但是，您会看到，已经使用了快捷安装和开发模式来设置命令行—`i`和`-D`。在大多数情况下，这是有效的，尽管您可能会发现，在您的命令行实用程序与您使用的 Webpack 版本之间存在兼容性问题时，现在会有响应。

以这种方式做事可以节省您的时间，但是如果有疑问，请使用本指南中演示的冗长命令行约定。

现在，让我们转向`coverjs`，它的工作方式略有不同。

# coverjs

`coverjs`允许对您的代码进行仪器化。这基本上意味着它允许对您的代码进行性能测量或监视。

`coverjs`加载器不需要与`mocha-loader`结合使用，因为它是独立的。`reportHtml`函数将附加到输出的主体部分。

在以下示例中，`webpackOptions.js`是代码的主题。在第一组花括号（`{`）中是与模块导出程序相关的选项。在双花括号（`[{`）中是绑定`coverjs`加载器和`test:""`语句的代码（表示每个文件都将被测试）：

```js
webpack - dev - server "mocha!./cover-my-client-tests.js"--options webpackOptions.js
// webpackOptions.js
module.exports = {
    output: "bundle.js",
    publicPrefix: "/",
    debug: true,
    includeFilenames: true,
    watch: true,
    postLoaders: [{
       test: "",
       exclude: [
         "node_modules.chai",
         "node_modules.coverjs-loader",
         "node_modules.webpack.buildin"
    ],
    loader: "coverjs-loader"
 }]
}
// cover-my-client-tests.js
require("./my-client-tests");
after(function() {
   require("cover-loader").reportHtml();
});
```

正如您所看到的，这个特定的加载器通过本地文件设置了它的选项。对此的修改将产生与上一章讨论的配置相同的效果。这应该是您在应用程序中启动`coverjs`所需的全部内容。接下来是一个更复杂的主题，涉及使用国际语言。

# i18n-loader

`i18n-loader`处理国际化（`i18n`），这是准备应用程序以支持本地语言和文化设置的过程。让我们通过以下步骤进行设置：

1.  从命令行安装开始：

```js
npm install i18n-loader
```

1.  现在，让我们从使用 CSS 开始。我们将为`i18n`**设置我们的样式表。**这是通过我们通常的项目样式表`css/styles.css`完成的。如果您导入另一个样式表，也可以在那里进行修改：

```js
. / colors.json {
        "red": "red",
        "green": "green",
        "blue": "blue"
    }
    . / de - de.colors.json {
        "red": "rot",
        "green": "green"
    }
```

假设我们的区域设置是`de-de-berlin`（德语语言和区域设置，以此为例），现在可以调用加载器。

1.  接下来，我们将通过使用`index.js`文件为`i18n`设置本地化代码的颜色方案：

```js
var locale = require("i18n!./colors.json");
```

现在，等待状态准备就绪。这只需要一次，因为同一语言的所有区域设置都会合并成一个模块块：

```js
locale(function() {
 console.log(locale.red); // prints red
 console.log(locale.blue); // prints blue
});
```

通常这都是在与之前相同的文件中完成的。前面的代码将在`console.log`函数中的`locale`变量中添加一个子节点，这将有助于测试。

1.  现在，使用`webpack.config.js`配置加载器，并利用相关的可用选项。由于该加载器也有选项，如果你想要一次加载所有区域设置并且想要同步使用它们，你应该告诉加载器所有的区域设置：

```js
{
    "i18n": {
        "locales": [
            "de",
            "de-de",
            "fr"
        ],
        // "bundleTogether": false
    }
}
```

请注意前面代码中的`// "bundleTogether": false`语句，这可以取消注释并设置为禁用区域设置的捆绑。

还有其他的调用方式。以下代码通过区域设置选择正确的文件：

```js
require("i18n/choose!./file.js"); 
```

但是，这不会合并对象。在下面的代码中，第一行将连接所有符合条件的区域设置，第二行将合并生成的对象：

```js
require("i18n/concat!./file.js"); 
require("i18n/merge!./file.js"); 
```

因此，在前面的代码块中，编译时会执行`./file.js`：

```js
require("i18n!./file.json") ==
   require("i18n/merge!json!./file.json")
```

前面的代码块只是加强了正则表达式。它确保`require`语句中的任何一个都会加载相同的文件。

如果你想在 Node.js 中使用`require`，不要忘记进行 polyfill。请参阅本章末尾的*进一步阅读*部分，了解相关的 Webpack 文档。

前面的代码块只是调整了你的项目，使其与德语领土和德语听众兼容。接下来是`imports-loader`。

# imports-loader

`imports-loader`允许你使用依赖于特定全局变量的模块。这对于可能依赖于全局变量的第三方模块很有用。`imports-loader`可以添加必要的`require`调用，使它们能够与 Webpack 一起工作。让我们通过以下步骤进行设置：

1.  要在命令行中安装加载器，请使用以下语句：

```js
npm install imports-loader
```

假设你有`example.js`文件，这个加载器允许你使用 jQuery 将导入的脚本附加到图像标签中，如下所示：

```js
$("img").doSomeAwesomeJqueryPluginStuff();
```

1.  然后可以通过配置`imports-loader`将`$`变量注入到模块中，如下所示：

```js
require("imports-loader?$=jquery!./example.js");
```

这将简单地将`var $ = require("jquery");`添加到`example.js`的开头。例如，如果你正在优化代码以在本地运行库，这可能会很有用。

类似地，使用`polymer-loader`可以优化代码或自动化流程以允许转换。这是我们讨论的下一个主题。

# polymer-loader

`polymer-loader`用于将 HTML 文件转换为 JavaScript 文件。要配置加载器，请在`webpack.config.js`中使用以下代码：

```js
{
  test: /\.html$/,
  include: Condition(s) (optional),
  exclude: Condition(s) (optional),
  options: {
    ignoreLinks: Condition(s) (optional),
    ignorePathReWrite: Condition(s) (optional),
    processStyleLinks: Boolean (optional),
    htmlLoader: Object (optional)
  },
  loader: 'polymer-webpack-loader'
},
```

`polymer-webpack-loader`短语允许开发人员在单个文档中编写 HTML、CSS 和 JavaScript 代码作为聚合元素，例如，同时仍然能够使用完整的 Webpack 系统，包括模块捆绑和代码拆分。

# script-loader

`script-loader`基本上允许 JavaScript 在单个实例中加载。这适用于整个项目。让我们通过以下步骤进行设置：

1.  要安装`script-loader`，在命令行中输入以下内容：

```js
npm install --save-dev script-loader
```

请注意这在 Node.js 中不起作用。

1.  使用`webpack.config.js`配置 Webpack，将`exec`从`'script.exec.js';`导出：

```js
module.exports = {
    module: {
        rules: [{
            test: /\.exec\.js$/,
            use: ['script-loader']
        }]
    }
}
```

还有一种内联的方法，如下所示：

```js
import exec from 'script-loader!./script.js';
```

这应该是你在 Webpack 应用程序中使用`script-loader`所需要的全部内容。接下来是`source-map-loader`。

# source-map-loader

`source-map-loader`从项目中的所有 JavaScript 入口中提取现有的源映射。这包括内联源映射以及外部加载的源映射。所有源映射数据都按照你在`webpack.config.js`中使用`devtool`选项指定的源映射样式进行处理。以下代码显示了该配置：

```js
module.exports = {
     module: {
     rules: [
     {
       test: /\.script\.js$/,
       use: [
   {
     loader: 'script-loader',
     options: {
           sourceMap: true,
              },
   },
        ]
    }
         ]
    }
 }
```

当使用具有源映射的第三方库时，此加载器可能非常有用。如果不将其提取并处理为包的源映射，浏览器可能会错误地解释源映射数据。此加载器允许在库和框架之间维护源映射数据的连续性，以确保轻松调试。

该加载器将从任何 JavaScript 文件中提取，包括`node_modules`目录中的文件。在设置`include`和`exclude`规则条件时，应注意优化捆绑性能。

# less-loader

**`less-loader`**加载**LESS**（一种**CSS**类型）脚本。你应该以通常的方式使用命令行安装它，例如`npm i less-loader`。对于未经培训的人来说，LESS 是 CSS 的一种更简洁的语法形式，对于向后兼容性非常有用。

你应该将`less-loader`与`css-loader`和`style-loader`链接起来，以立即将所有样式应用到文档中。要配置此项，请使用以下示例代码，使用`webpack.config.js`文件：

```js
module.exports = {
    module: {
        rules: [{
            test: /\.less$/,
            use: [{
                    loader: 'style-loader', // creates style nodes 
                                               from JS strings

                },
                {
                    loader: 'css-loader', // translates CSS 
                                              into  CommonJS
                },
                {
                    loader: 'less-loader', // compiles Less to CSS
                },
            ],
        }, 
      ],
    },
 };
```

你可以通过加载器选项将任何 LESS 特定选项传递给`less-loader`。由于这些选项作为程序的一部分传递给 LESS，它们需要以`camelCase`形式传递；以下示例在**`webpack.config.js`**中展示了如何做到这一点：

```js
module.exports = {
    module: {
        rules: [{
            test: /\.less$/,
            use: [{
                    loader: 'style-loader',
                },
                {
                    loader: 'css-loader',
                },
                {
                    loader: 'less-loader',
                    options: {
                        strictMath: true,
                        noIeCompat: true,
                    },
                },
            ],
         },
       ],
    },
 };
```

请注意，LESS 不会将所有选项单独映射到`camelCase`。建议你检查相关的可执行文件，并搜索`dash-case`选项。

在生产模式下，通常建议使用`MiniCssExtractPlugin`将样式表提取到专用文件中。这样，你的样式就不会依赖于 JavaScript（稍后会详细介绍）。

在本节中，我们深入讨论了加载器，并对一些更有用的加载器进行了详细的检查。大多数加载器都遵循相同的安装和配置逻辑，并由 Webpack 社区构建，而不是由 Webpack 自己构建。我们将在本书的最后一章中更详细地讨论自定义加载器。

这里有太多其他加载器要提及，但这为你提供了一个非常坚实的基础，可以以各种富有想象力的方式与它们一起工作。与加载器通常处理的非本地脚本相关的东西是使用 Webpack 的 API。我们将在下一节中进行调查。

# API

API 对于充分利用 Webpack 至关重要。简单来说，API 在应用程序和网站之间进行通信时是必需的。对于像 Webpack 这样的 JavaScript 捆绑器，这包括数据库和后端连接。

在使用 JavaScript 时，有大量的 API 可用，但我们无法在这里逐一介绍它们；然而，有一些更有用的常见但也复杂的 API 在使用 Webpack 进行编程时经常被用作工具。

这些工具是特定于 Webpack 的，允许在 Webpack 中进行广泛或多功能的功能，而不仅仅是简单地访问外部代码。其中最值得注意的工具是 Babel 和 Node.js。因此，让我们以这些工具为例，并在接下来的小节中学习它们的用法，首先是 Babel。

# Babel 及其加载器构建者

如果你不知道，Babel 是一个主要将 ECMAScript 2015 及其前版本代码转换为与当前和旧版浏览器或环境兼容的 JavaScript 的工具链。Babel 可以为开发者做的主要事情如下：

+   转换语法

+   使用`@babel/polyfill`填充目标环境中缺失的功能

+   执行源代码转换，称为代码修改

Webpack 使用 Babel 接口的 API。如果你尝试使用命令行安装它，你应该会收到以下消息：

```js
The Node.js API for babel has been moved to babel-core.
```

如果收到此消息，意味着你已经安装了`Babel`并且在 Webpack 配置文件中使用了加载器的简短表示法（这在 Webpack 2 及更高版本中已不再有效）。

在`webpack.config.js`中，您可能会看到以下脚本：

```js
  {
    test: /\.m?js$/,
    loader: 'babel',
  }
```

由于安装了 Babel，Webpack 尝试加载`babel`包而不是`babel-loader`。

为了解决这个问题，应该安装`npm`包 Babel，因为它在 Babel 版本 6 中已被弃用。如果您使用的是 Babel 版本 6，则应该安装`@babel/cli`或`@babel/core`。如果您的一个依赖项正在安装 Babel，并且您无法自行卸载它，请在`webpack.config.js`中使用加载器的完整名称：

```js
  {
    test: /\.m?js$/,
    loader: 'babel-loader',
  }

```

到目前为止，您所遵循的示例应该为您在一般情况下使用 Babel 奠定了良好的基础，但 Babel 的一个关键用途是自定义加载器。这是本书最后一章更全面地介绍的一个主题，但我们现在将讨论 Babel 如何与自定义加载器一起工作，特别是因为您可能不使用自定义的加载器。

`babel-loader`公开了一个`loader-builder`实用程序，允许用户为它处理的每个文件添加 Babel 配置的自定义处理。

`.custom`短语接受一个回调函数，将使用 Babel 的加载器实例进行调用。这样工具可以确保它使用与加载器本身相同的`@babel/core`实例。

在您想要自定义但实际上没有文件调用`.custom`的情况下，您还可以使用`customize`选项，并将其指向导出您自定义回调函数的文件的字符串。

可能了解其工作原理的最佳方法是通过一个实际的例子；让我们在以下练习中进行一次。

这个例子的目标是演示如何使用`babel-loader`来构建自定义加载器。

这个例子首先使用了一个自定义的文件名。这可以是任何你想要的，但为了这个练习，我们选择的名称是`./my-custom-loader.js`。您可以从这个位置或任何您想要的地方导出：

1.  首先，通过使用以下代码在`./my-custom-loader.js`中创建一个自定义文件：

```js
module.exports = require("babel-loader").custom(babel => {
 function myPlugin() {
 return {
 visitor: {},
 };
 }
```

在上面的代码块中，我们可以看到一个`require`语句。这使用了`babel-loader`，我们需要创建自定义加载器。

1.  现在，我们需要配置我们的项目以设置传递给加载器的传递，如下所示：

```js
return {
    customOptions({
        opt1,
        opt2,
        ...loader
    }) {
        return {
            custom: {
                opt1,
                opt2
            },
            loader,
        };
    },
```

请注意，`custom:`指的是提取加载器可能具有的任何自定义选项。还要注意两个选项后面的`loader`引用。这将删除两个`custom`选项并将选项传回。

1.  然后，我们传递 Babel 的`PartialConfig`对象，将正常配置设置为`return cfg.options`：

```js
config(cfg) {
    if (cfg.hasFilesystemConfig()) {
        return cfg.options;
    }

    return {
        ...cfg.options,
        plugins: [
            ...(cfg.options.plugins || []),
            testPlugin,
        ],
    };
},
```

在上述代码中，当`testPlugin`语句被执行时，我们可以看到自定义插件的包含，然后将作为一个选项可用。

1.  现在，让我们创建占位文本来测试自定义加载器。前面的代码应该生成类似以下的内容：

```js
result(result) {
return {
    ...result,
    code: result.code + "\n// Generated by this custom loader",
    };
    },
    };
});
```

这个代码块显示了自定义加载器正在生成代码。

1.  确保您的配置是正确的。请注意，您应该始终将`__dirname`和`custom-loader`替换为您选择的名称。在 Webpack 配置模块中，键入以下内容：

```js
.exports = {
    module: {
        rules: [{
            loader: path.join(__dirname, 'custom-loader.js'),
        }]
    }
};
customOptions(options: Object): {
    custom: Object,
    loader: Object
}

```

上面的代码块向您展示了如何设置和配置`customOptions`。

1.  根据加载器的选项，将自定义选项从`babel-loader`的选项中拆分出来：

```js
config(cfg: PartialConfig): Object
```

1.  给定 Babel 的`PartialConfig`对象，返回应传递给`babel.transform`的`options`对象：

```js
result(result: Result): Result
```

前面两个代码块都涉及到我们构建的自定义文件的内容，在这个例子中是`./my-custom-loader.js`。

请注意，Babel 的`result`对象将允许加载器对其进行额外的调整。

这应该是你需要让自定义加载器与 Babel 一起工作的全部内容。阅读本书最后一章有关编写和自定义加载器的更多信息。在 Webpack 项目中经常使用的另一个关键 API 是 Node.js API。

# Node.js API

当使用自定义开发流程时，Node.js API 非常有用。这是因为所有的报告和错误处理都是手动完成的。在这种情况下，Webpack 只是处理编译过程。请注意，在使用 Node.js API 时，`stats`配置选项将不会产生任何效果。

当您安装 Webpack 5 时，这个 API 将被安装；如果您按顺序阅读本节，可以参考第一章。让我们通过以下步骤设置这个 API：

1.  首先，将`webpack`模块包含到您的 Node.js 脚本中。这是通过`webpack.config.js`文件完成的：

```js
const webpack = require('webpack');

webpack({
  // Configuration Object
}, (some, stats) => { // Stats Object
  if (some || stats.hasErrors()) {
    // Handle errors here
  }
  // Done processing
});
```

在上面的例子中，提供了一个回调函数`webpack()`，它运行编译器。代码呈现了一些条件。这只是一个例子，当然应该用您的代码替换。同样，`some`术语应该被替换为与您的项目相关的正确对象名称。

请注意，`some`对象不会包括编译错误，而只包括与 Webpack 特定的问题，如错误配置相关的问题。这些错误将使用`stats.hasErrors()`函数来处理。

1.  接下来，确保正确传递编译器实例。

如果 Webpack 函数没有提供回调，它将返回一个`compiler`实例。`compiler`实例可以手动触发`webpack()`函数，或者确保它在构建过程中观察更改（使用`.run(callback)`或`.watch(watchOptions, handler)`），甚至在不需要 CLI 的情况下运行构建本身。

`compiler`实例允许使用子编译器，并将所有捆绑、写入和加载工作委托给注册的插件。

有一个叫做`hook`属性，它是`compiler`实例的一部分。它的目的是在编译器的生命周期中注册任何插件到任何钩子事件。您可以使用`WebpackOptionsDefaulter`和`WebpackOptions Apply`工具来配置这个编译器。

在构建运行完成后，之前提到的回调函数将被执行。使用这个函数来最终记录任何错误或统计信息。

Node.js API 只支持单个编译一次。并发的观察或构建可能会破坏输出捆绑包。

使用 API 调用运行类似于使用`compiler`实例。

1.  现在我们应该使用`webpack.config.js`运行一个编译：

```js
const webpack = require('webpack');

const compiler = webpack({
 // Configuration Object
});

compiler.run((some, stats) => { // Stats Object
});
```

1.  从这里，我们还可以触发一个`watch`会话。当`webpack()`函数检测到更改时，它将再次运行并返回一个`watching`实例：

```js
watch(watchOptions, callback);
const webpack = require('webpack');

const compiler = webpack({
 // Configuration Object
});

const watching = compiler.watch({
 // Example watchOptions
 aggregateTimeout: 300,
 poll: undefined
}, (some, stats) => { // Stats Object
 // Print watch/build result here...
 console.log(stats);
});
```

由于文件系统的不准确性可能会触发多次构建，如果检测到更改，前面代码块中的`console.log`语句可能会多次触发任何单个修改。检查`stats.hash`可以帮助您查看文件是否已更改。

1.  使用这种方式的`watch`方法将返回一个`watching`实例和一个`.close(callback)`方法。调用这个方法将结束`watching`会话：

```js
watching.close(() => {
 console.log('Watching Ended.');
});
```

请注意，使用`invalidate`观察函数将允许手动使当前编译无效，而不会停止`watch`过程。这很有用，因为一次只允许运行一次：

```js
watching.invalidate();
```

由于多个同时的编译是受限制的，Webpack 提供了一个叫做`MultiCompiler`的东西来加快项目的开发。它是一个模块，允许 Webpack 在单独的编译器中运行多个配置。如果您的`options`参数是一个数组，Webpack 将应用单独的编译器，并在所有编译器完成其过程后执行任何回调：

```js
var webpack = require('webpack');
webpack([
 { entry: './index1.js', output: { filename: 'bundle1.js' } },
 { entry: './index2.js', output: { filename: 'bundle2.js' } }
], (some, stats) => { // Stats Object
 process.stdout.write(stats.toString() + '\n');
})
```

上面的代码块向您展示了如何在`webpack.config.js`中配置项目以允许这个过程。

正如所解释的，尝试并行运行这些编译将产生不正确的输出。如果这是意外发生的，错误管理就变得很重要。

一般来说，错误处理包括三种类型的错误——严重的 Webpack 错误（例如配置错误）、编译错误（例如缺少资源）和编译警告。

以下代码块向您展示了如何配置您的项目——在本例中，使用 `webpack.config.js`——来处理这些错误：

```js
const webpack = require('webpack');

webpack({
 // Configuration Object
}, (some, stats) => {
 if (some) {
   console.error(some.stack || some);
 if (some.details) {
   console.error(some.details);
 }
 return;
 }
const info = stats.toJson();
if (stats.hasErrors()) {
  console.error(info.errors);
 }
if (stats.hasWarnings()) {
  console.warn(info.warnings);
 }
// Log results...
});
```

在上述代码块中，`some` 元素表示这些错误作为一个变量。我们可以看到各种条件将在控制台日志中注册这些错误。

我们已经为您提供了关于如何使用 Webpack 的 API 的密集速成课程，所以如果您幸存下来，现在您是编程技能的专家。干得好！

现在我们已经探讨了各种加载器和 API（包括 Babel 和 Node.js），是时候看一下本章涵盖的最后一个功能了——插件。

# 插件

插件的作用是做任何加载器无法做到的事情。加载器通常帮助运行不适用于 Webpack 的代码，插件也是如此；然而，加载器通常由社区构建，而插件由 Webpack 的内部开发人员构建。

插件被认为是 Webpack 的支柱。该软件是建立在与您在 Webpack 配置中使用的相同的插件系统上的。

Webpack 有丰富的插件接口，Webpack 自身的大多数功能都使用它。以下是本节将详细介绍的可用插件的项目列表。每个名称旁边都有一个插件的简要描述：

+   `BabelMinifyWebpackPlugin`：使用 `babel-minify` 进行最小化

+   `CommonsChunkPlugin`：提取在块之间共享的公共模块

+   `ContextReplacementPlugin`：覆盖 require 表达式的推断上下文

+   `HotModuleReplacementPlugin`：启用 **热模块替换**（**HMR**）（稍后详细介绍）

+   `HtmlWebpackPlugin`：轻松创建用于提供捆绑包的 HTML 文件

+   `LimitChunkCountPlugin`：设置分块的最小/最大限制，以更好地控制该过程

+   `ProgressPlugin`：报告编译进度

+   `ProvidePlugin`：使用模块而无需使用 `import`/`require`

+   `TerserPlugin`：启用对项目中 Terser 版本的控制

Webpack 社区页面上有许多其他插件可用；然而，上述列表说明了更显著和有用的插件。接下来，我们将详细解释每一个。

我们不会详细介绍最后三个插件，因为它们只需要按照通常的方式安装。

每个插件的安装都遵循与加载器相同的过程：

1.  首先，使用命令行安装插件，然后修改配置文件以引入插件，类似于以下示例：

```js
npm install full-name-of-plugin-goes-here-and-should-be-hyphenated-and-not-camelcase --save-dev
```

请记住，这只是一个通用示例；您将需要添加您的插件名称。与之前使用的相同的 Webpack 项目和相同的配置文件 `webpack.config.js` 一样，以下配置也是如此。

1.  现在我们应该准备我们的配置文件：

```js
const MinifyPlugin = require("full-name-of-plugin-goes-here-and-should-be-hyphenated-not-camelcase");
module.exports = {
 entry: //...,
 output: //...,
 plugins: [
 new MinifyPlugin(minifyOpts, pluginOpts)
 ]
}
```

这就结束了关于插件的一般介绍。我们采取了一般方法来防止过度复杂化。现在，我们可以继续讨论 Webpack 上可用的各种插件的一些有趣方面。在下一小节中，我们将讨论以下内容：

+   `BabelMinifyWebpackPlugin`

+   `CommonsChunkPlugin`

+   `ContextReplacementPlugin`

+   `HtmlWebpackPlugin`

+   `LimitChunkCountPlugin`

大多数插件由 Webpack 内部开发，并填补了加载器尚不能填补的开发空白。以下是一些更有趣的插件。让我们从 `BabelMinifyWebpackPlugin` 开始。

# BabelMinifyWebpackPlugin

在本小节中，我们将安装 `BabelMinifyWebpackPlugin`。该插件旨在最小化 Babel 脚本。如前所述，最小化是指删除错误或多余的代码以压缩应用程序大小。要使用 `babel-loader` 并将 `minify` 作为预设包含在内，使用 `babel-minify`。使用此插件的 `babel-loader` 将更快，并且将在较小的文件大小上运行。

Webpack 中的加载器操作单个文件，`minify`的预设将直接在浏览器的全局范围内执行每个文件；这是默认行为。顶层范围中的一些内容将不被优化。可以使用以下代码和`minifyOptions`来优化文件的`topLevel`范围：

```js
mangle: {
topLevel: true
}
```

当`node_modles`被排除在`babel-loader`的运行之外时，排除的文件不会应用缩小优化，因为没有传递给`minifer`。

使用`babel-loader`时，生成的代码不经过加载器，也不被优化。

插件可以操作整个块或捆绑输出，并且可以优化整个捆绑包，可以在缩小的输出中看到一些差异；但是，在这种情况下，文件大小将会很大。

Babel 插件在使用 Webpack 时非常有用，并且可以跨多个平台使用。我们将讨论的下一个插件是`CommonsChunkPlugin`，它旨在与多个模块块一起使用，这是 Webpack 非常本地的功能。

# CommonsChunkPlugin

`CommonsChunkPlugin`是一个可选功能，它创建一个称为块的单独功能。块由多个入口点之间共享的公共模块组成。

这个插件已经在 Webpack 4（Legato）中被移除了。

查看`SplitChunkPlugin`将更多地了解 Legato 中如何处理块。

生成的分块文件可以通过分离形成捆绑包的公共模块来最初加载一次。这将被存储在缓存中以供以后使用。

以这种方式使用缓存将允许浏览器更快地加载页面，而不是强制它加载更大的捆绑包。

在您的配置中使用以下声明来使用这个插件：

```js
new webpack.optimize.CommonsChunkPlugin(options);
```

一个简短而简单的安装，但是值得介绍。接下来，让我们使用`ContextReplacementPlugin`。

# ContextReplacementPlugin

`ContextReplacementPlugin`是指带有扩展名的`require`语句，例如以下内容：

```js
require('./locale/' + name + '.json')
```

当遇到这样的表达式时，插件将推断`./local/`的目录和一个正则表达式。如果在编译时没有包含名称，那么每个文件都将作为模块包含在捆绑包中。这个插件将允许覆盖推断的信息。

下一个要讨论的插件是`HtmlWebpackPlugin`。

# HtmlWebpackPlugin

`HtmlWebpackPlugin`简化了创建用于提供捆绑包的 HTML 文件。这对于包含文件名中的哈希值的捆绑包特别有用，因为每次编译都会更改。

使用这种方法时，我们有三种选择——使用带有`lodash`模板的模板，使用您的加载器，或者让插件生成一个 HTML 文件。模板只是一个 HTML 模板，您可以使用`lodash`自动加载以提高速度。加载器或插件可以生成自己的 HTML 文件。这都可以加快任何自动化流程。

当使用多个入口点时，它们将与生成的 HTML 中的`<script>`标签一起包含。

如果 Webpack 的输出中有任何 CSS 资产，那么这些资产将包含在生成的 HTML 的`<head>`元素中的`<link>`标签中。例如，如果使用`MiniCssExtractPlugin`提取 CSS。

回到处理块。下一个要看的插件是处理限制块计数的插件。

# LimitChunkCountPlugin

在按需加载时使用`LimitChunkCountPlugin`。在编译时，您可能会注意到一些块非常小，这会产生更大的 HTTP 开销。`LimitChunkCountPlugin`可以通过合并块来后处理块。

通过使用大于或等于`1`的值来限制最大块数。使用`1`可以防止将额外的块添加为主块或入口点块：

```js
[/plugins/min-chunk-size-plugin]
```

保持块大小在指定限制以上不是 Webpack 5 中的一个功能；在这种情况下应该使用`MinChuckSizePlugin`。

这就结束了我们对插件的介绍以及本章的总结。插件使 Webpack 能够以各种方式工作，而 Webpack 使开发人员能够构建加载程序并填补功能问题的空白。在处理大型项目或需要自动化的复杂项目时，它们是不可或缺的。

我们关于 API 的部分向您展示了，有时我们并不总是想使用本地代码，并为您提供了一个很好的过渡到我们将在下一章中讨论的库。

# 摘要

本章深入介绍了加载程序及其在 Webpack 中的使用方式。加载程序对于 Webpack 来说是基础，但插件是核心和支柱。本章带您了解了这些主题的主要特点，展示了每个主题的最佳用法以及何时切换使用它们的好时机。

然后我们探讨了插件及其使用方式，包括 Babel、自定义插件和加载程序。我们还研究了 API 及其使用方式，特别是那些在 Webpack 中实现更广泛功能的 API，比如 Babel 和 Node API。

在下一章中，我们将讨论库和框架。我们对插件、API 和加载程序的研究表明，有时我们不想使用诸如库之类的远程代码，但有时我们确实需要。Webpack 通常处理本地托管的代码，但有时我们可能需要使用库。这为我们提供了一个很好的过渡到这个主题。

# 问题

1.  什么是`i18n`加载程序？

1.  Webpack 通常使用哪种工具链来转换 ECMA 脚本？

1.  Babel 主要用于什么？

1.  哪个加载程序允许用户添加对 Babel 配置的自定义处理？

1.  `polymer-webpack-loader`是做什么的？

1.  `polymer-webpack-loader`为开发人员提供了什么？

1.  在使用 Node.js API 时，提供的回调函数将运行什么？
