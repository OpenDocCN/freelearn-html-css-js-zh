# 第八章：编写教程和实时编码黑客

这是 Webpack 5.0 的最后一章。到目前为止，您可能已经感觉自己是一个专家，但是您的掌握能力不仅来自于您对代码本身的掌握，还包括自定义平台的能力，甚至是对其进行黑客。纯粹主义者可能会避免使用“黑客”这样的术语，而更倾向于使用变通方法或修补程序，但我们基本上讨论的是同一个主题（除非向外行解释您在实时生产中黑客了 Webpack 肯定会给编码超能力的最坚定的怀疑者留下深刻印象）。

话虽如此，本章是 Webpack 中最难做的事情和最简单的实现方法的汇编。我们将从编写库开始，然后转向自定义加载程序，这在本指南中已经讨论过，特别是使用 Babel 的**应用程序编程接口**（**API**）。然而，本章将讨论一种更本地的定制方法，而无需 API。

当然，我们将涵盖一些在本指南中已经详细讨论过的主题，例如测试和 shimming，但是本指南将更加突出和相关于定制。

然后，我们将涵盖一些非常有趣的黑客，特别是在**热模块替换**（**HMR**）启用的环境中进行实时发布时。

本章涵盖的一些主题如下：

+   编写库

+   自定义加载程序

+   实时编码黑客

# 编写库

本章的这一部分对于希望简化其捆绑策略的任何人都非常有用。人们并不太了解 Webpack 可以用于捆绑库和应用程序。

我们将从一个假设的自定义库项目开始，我们将其称为`numbers-to-text`。它将通过将数字从`1`到`5`转换为数字的文本表示，例如将`3`转换为`three`。

让我们详细讨论每个任务，并解释代码中发生的事情，通过示例帮助我们更清楚地理解发生了什么以及代码的行为如何。我们将按以下方式进行：

1.  我们将首先整理我们的项目结构。基本项目结构应如下所示：

```js
|- webpack.config.js
  |- package.json
  |- /src
  |- index.js
  |- ref.json
```

请注意，结构可能与以前的教程不同，特别是`ref.json`文件的存在。在继续之前，您需要创建这些额外的文件。

1.  接下来，我们回到**命令行界面**（**CLI**），首先需要初始化`npm`，然后确保我们已安装了 Webpack 和`lodash`。如果您已经按照以前章节中的示例安装了这些内容，那就不用担心了，重复安装尝试只会覆盖最后一次，并不会造成任何伤害。运行以下代码：

```js
npm init -y
npm install --save-dev webpack lodash
```

1.  完成这些工作后，我们将注意力转向新构建的`src/ref.json` **JSON**文件。这是自定义库的基本数据。它应该看起来像以下示例：

```js
[
  {
    "num": 1,
    "word": "One"
  },
  {
    "num": 2,
    "word": "Two"
  },
  {
    "num": 3,
    "word": "Three"
  },
  {
    "num": 4,
    "word": "Four"
  },
  {
    "num": 5,
    "word": "Five"
  },
  {
    "num": 0,
    "word": "Zero"
  }
]
```

正如您所看到的，这构成了一个简单的选项列表，代表一个数字，以及该数字的对应文字版本。这将构成我们非常简单的库结构的支柱，以原则上演示概念。一旦教程完成，您应该自然地看到如何根据需要调整库，无论多么复杂。

1.  现在，我们需要制作一个索引文件（例如`src/index.js`）。您应该按照此块中显示的编码进行操作：

```js
import _ from 'lodash';
import numRef from './ref.json';
export function numToWord(num) {
  return _.reduce(numRef, (accum, ref) => {
  return ref.num === num ? ref.word : accum;
 }, '');
}
export function wordToNum(word) {
  return _.reduce(numRef, (accum, ref) => {
  return ref.word === word && word.toLowerCase() ? ref.num : accum;
  }, -1);
}
```

您可以从前面的代码中看到索引文件实质上包含一系列与**JSON**文件内容相关的`export`和`return`函数。

1.  现在，我们需要定义使用规范。操作如下：

+   +   ES2015 模块导入：

```js
 import * as numbersToText from 'numbers-to-text';
// ...
 numbersToText.wordToNum('Two'); 
```

+   +   CommonJS 模块要求：

```js
 const numbersToText = require('numbersToText');
// ...
```

1.  在相同的文件中使用以下代码片段来设置使用`AMD`时的函数：

```js
numbersToText.wordToNum('Two');
AMD module requires:
require(['numbersToText'], function (numbersToText) {
 numbersToText.wordToNum('Two');
 });
```

用户还可以通过`script`标签加载库，如下所示：

```js
<!doctype html>
 <html>
   ...
   <script src="img/webpack-numbers"></script>
   <script>
    // Global variable
    numbersToText.wordToNum('Five')
    // Property in the window object
    window.numbersToText.wordToNum('Five')
   </script>
 </html>
```

这是加载库的最常见方式，作为 JavaScript 开发人员，这应该是一种应该很容易遵循的事情。也就是说，它可以配置为在全局对象中公开属性，用于 Node.js，或者作为`this`对象中的属性。

这将带我们进入库的基本配置。授权这个库有多个级别的配置，所以这只是第一步。

# 基本配置

与任何 Webpack 项目一样，我们需要对其进行配置。与典型的配置相比，处理自定义库时需要实现一些不寻常的事情。现在让我们考虑这些目标。库应该以一种实现以下目标的方式捆绑：

+   使用外部来避免捆绑`lodash`，因此用户需要加载它

+   指定库的外部限制

+   将库公开为名为`numbersToText`的变量

+   将库名称设置为`numbers-to-text`

+   允许在**Node.js**库中访问

此外，请注意用户必须能够以以下方式访问并使用库：

+   通过从`numbers-to-text`导入`numbersToText`作为**ECMAScript 2015** (**ES 2015**)模块

+   通过 CommonJS 模块，例如使用`require('webpack-numbers')`方法

+   通过全局变量，当通过脚本标签包含时

考虑到所有这些，首先要做的是通过我们用于`webpack.config.js`文件的常规文件设置 Webpack 配置，如下所示：

```js
const path = require('path');
module.exports = {
  entry: './src/index.js',
  output: {
   path: path.resolve(__dirname, 'dist'),
   filename: 'numbers-to-text.js'
  }
};
```

这是基本配置，但现在我们将转向下一个确定的目标：将`lodash`外部化。

# 使用外部来避免捆绑 lodash

如果现在执行构建，你会看到创建了一个相当大的捆绑包。检查文件会发现`lodash`已经与其捆绑在一起。出于本教程的目的，最好将`lodash`视为对等依赖项。这基本上意味着用户必须安装`lodash`，有效地将这个外部库的控制权交给了这个库的用户。

这可以通过外部的配置来完成，如`webpack.config.js`中所示，如下所示：

```js
const path = require('path');
module.exports = {
  entry: './src/index.js',
  output: {
   path: path.resolve(__dirname, 'dist'),
   filename: 'numbers-to-text.js'
   }
  },
   externals: {
     lodash: {
      commonjs: 'lodash',
      commonjs2: 'lodash',
      amd: 'lodash',
      root: '_'
    }
  }
};
```

前面的代码意味着库期望用户环境中有一个名为`lodash`的依赖项。

请注意，如果计划在并行的 Webpack 捆绑包中将库用作依赖项，则可以将外部指定为数组。

在外部指定限制也很重要，我们现在将讨论这一点。

# 指定外部限制

您可以使用多个文件的库，例如以下描述性块中的文件：

+   `import A from 'library/one';`

+   `import B from 'library/two';`

在这种情况下，它们不能通过在外部指定库来从捆绑包中排除。它们需要逐个排除，或者使用正则表达式，例如以下示例：

```js
module.exports = {
 externals: [
   'library/one',
   'library/two',
   // Everything that starts with "library/"
   /^library\/.+$/
  ]
};
```

完成这些后，我们需要公开库或允许它加载到我们的前端。这将在下一小节中介绍。

# 公开库

公开库是本指南中已经讨论过的内容，但如果您跳到本章，您可能会感到困惑。我们只是允许我们的应用程序从外部源加载库，就像任何外部加载到网页中的库一样。库应该与不同的环境兼容，例如**CommonJS**、**MD**或**Node.js**，以确保库的广泛可用性。为了确保这一点，按照以下步骤进行：

1.  应该在`webpack.config.js`配置文件的输出中添加`library`属性，如下所示：

```js
const path = require('path');
module.exports = {
  entry: './src/index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'numbers-to-text.js'
    filename: 'numbers-to-text.js',
 library: 'numbersToText'
 },
 externals: {
 lodash: {
 commonjs: 'lodash',
 commonjs2: 'lodash',
 amd: 'lodash',
 root: '_'
 }
 }
};
```

库设置与配置相关。在大多数情况下，指定一个入口点就足够了。多部分库是可能的；然而，通过一个索引脚本公开部分导出更简单，作为入口点。

尝试使用数组作为库入口点是不明智的，因为它不会编译得很好。

这实现了将库捆绑包公开为全局变量。

1.  您可以通过向配置文件添加`libraryTarget`属性，以不同的选项公开库，使用`webpack.config.js`，使您的库与其他环境兼容，如下所示：

```js
const path = require('path');
module.exports = {
  entry: './src/index.js',
  output: {
   path: path.resolve(__dirname, 'dist'),
   filename: 'numbers-to-text.js',
   library: 'numbersToText'
   library: 'numbersToText',
   libraryTarget: 'umd'
 },
 externals: {
   lodash: {
    commonjs: 'lodash',
    commonjs2: 'lodash',
    amd: 'lodash',
    root: '_'
   }
  }
};
```

配置设置后，我们现在需要公开库。请注意，可以通过以下方式完成：

+   作为变量——作为脚本标签提供的全局变量，例如`libraryTarget:'var'`

+   作为对象——通过`this`对象可用，例如`libraryTarget:'this'`

+   Window——通过`window`对象可用，例如`libraryTarget:'window'`

+   **通用模块定义**（**UMD**）—在 CommonJS 或 AMD `require`语句之后可用，例如`libraryTarget:'umd'`

如果您设置了库但没有为`libraryTarget`函数执行此操作，后者将默认为变量，如输出配置中所指定的那样。

# 命名库和使用 Node.js

正如所解释的，我们现在处于编写的最后步骤。随着我们的进行，应该按照指南进行输出的优化。在这样做的同时，我们将在`package.json`文件中将捆绑输出的路径添加为包的主字段，如下所示：

```js
{
 ...
 "main": "dist/numbers-to-text.js",
 ...
 }
 Or, to add as standard module as per this guide:
{
 ...
 "module": "src/index.js",
 ...
 }
```

前面代码块中名为`"main"`的选项键指的是我们从`package.json`文件中检索到的标准。`"module"`键指的是一个提案，允许 JavaScript 环境升级，能够在不损害任何向后兼容性能力的情况下使用 ES2015 模块。

在`"module"`属性的情况下，这应该指向一个使用 ES2015 模块语法的脚本，但不应使用浏览器或 Node.js 尚不支持的其他语法。这将使 Webpack 能够解析模块语法，并通过摇树来允许更轻的捆绑包，因为用户可能只使用任何给定库的某些部分。

完成后，捆绑包可以作为`npm`包发布。

您已经学会了如何使用具有相应数字和文本的 JSON 文件设置和配置您的第一个自定义库，包括在前端公开新库并指定库的范围限制。

在一个相关的领域，现在让我们谈谈自定义加载器。

# 自定义加载器

加载器在本指南的先前章节中已经详细讨论过。但是，我们只是提到了它们的定制或编写。至少展示您对 Webpack 的掌握至关重要，因此我们现在应该讨论一下。

以下教程将按照以下结构进行：

+   设置

+   简单使用

+   复杂使用

+   指南

*指南*部分本身将被细分，但现在，让我们从设置开始。

# 设置

开始本节的最佳方法是看看我们如何在本地开发和测试加载器。这是一个很好的和可接受的开始方式，我们将按照以下步骤进行：

1.  在测试单个加载器时，您可以简单地在**`webpack.config.js`**的规则对象中使用路径来解析到本地文件，如下所示：

```js
module.exports = {
 //...
 module: {
 rules: [
 {
   test: /\.js$/,
   use: [
 {
    loader: path.resolve('path/to/loader.js'),
    options: {/* ... */}
    }
   ]
  }
 ]
 }
};
```

1.  要测试多个加载器，可以利用`resolveLoader.modules`配置，Webpack 将在`webpack.config.js`中搜索加载器。例如，如果您的项目中有一个包含加载器的本地目录，代码将如下所示：

```js
module.exports = {
  resolveLoader: {
    modules: [
     'node_modules',
     path.resolve(__dirname, 'loaders')
   ]
  }
};
```

这应该是你开始的所有内容。但是，如果您已经为您的加载器创建了一个单独的存储库，您可以在您想要运行测试的项目中使用`npm`链接到该项目。

根据加载器的使用方式，有不止一种方法，因此——自然地——我们将从简单的使用开始。

# 简单使用

简单加载器的概念已经被提到过，即加载器在执行非常简单和特定的任务时更有用。这将使测试更容易，因为有这么多加载器，它们可以*链接*到其他加载器以执行更多样的任务。

当单个加载器应用于资源时，加载器只会被调用一个参数。这是一个包含正在加载的资源内容的字符串。

同步加载器可以返回表示转换模块的单个值。在更复杂的情况下，加载器可以通过使用以下函数返回任意数量的值：`this.callback(err, values...)`。

然后错误要么传递给函数，要么在同步加载器中抛出。

在这种情况下，加载器预计会返回一个或两个值。第一个值是一些作为字符串的结果 JavaScript 代码。第二个值是可选的，会产生一个`SourceMap`和 JavaScript 对象。

在加载器被链接的情况下，加载器往往变得更加复杂。当讨论自定义加载器的复杂用法时，这将是一个很好的开始，所以现在让我们开始吧。

# 复杂用法

如前一小节所讨论的，*简单用法*，复杂用法通常指的是在上下文中使用加载器或一组加载器进行链接模式。

当多个加载器被链接在一起时，重要的是要记住它们是以相反的顺序执行的！

这将是从右到左或从下到上，取决于您使用的数组格式。例如，以下将适用：

+   最后一个加载器首先被脚本调用，将传递原始资源的内容（加载器运行的数据或脚本）。

+   第一个加载器，称为最后一个，预计会返回 JavaScript 和一个可选的源映射。

+   中间的加载器将使用链中前一个加载器的结果进行执行。

因此，在以下常见示例中，`foo-loader`将传递原始资源，而`bar-loader`将接收`foo-loader`的输出，并返回最终转换的模块和源映射（如果需要）。

要以这种方式链接加载器，从`webpack.config.js`配置文件开始，就像这样：

```js
module.exports = {
  //...
  module: {
    rules: [
      {
        test: /\.js/,
        use: [
         'bar-loader',
         'foo-loader'
        ]
      }
    ]
   }
 };
```

这是配置，但由于链接加载器本质上是复杂的，最好遵循一个标准。接下来是一组指南，将帮助您的项目在不受影响的情况下完成。

# 指南

在编写加载器时，应遵循以下指南。它们按重要性排序，有些只适用于特定情况：

+   简化加载器的目的

+   利用链接

+   模块化输出

+   确保无状态

+   使用加载器实用程序

+   标记加载器依赖关系

+   解决模块依赖关系

+   提取公共代码

+   不惜一切代价避免绝对路径！

+   使用对等依赖

让我们更详细地讨论每一个，从简化开始。

# 简化加载器的目的

加载器在执行简单和清晰的任务时效果最佳。这可以使每个加载器的维护工作更简单，也可以允许链式使用在更复杂的任务中。这是因为可能有许多不同的加载器针对不同的任务；因此，为了允许多样性，它们经常被顺序使用。

因此，就像 Webpack 捆绑包一样，它们应该是模块化的。因此，它们执行的特定任务可以被隔离和完善，这也将允许在与其他加载器链式使用时更广泛地应用。这将引出下一个概念：加载器的链接。

# 利用链接

利用加载器可以链接在一起的事实。不要编写一个处理多个任务的单个加载器，而是编写多个加载器。将它们隔离开不仅使每个加载器简单，还可以使它们用于更多样的任务。

例如，当使用加载程序选项或查询参数指定数据来呈现模板文件时，可以编写为单个加载程序，该加载程序从源代码编译模板，执行它，并返回一个导出包含 HTML 代码的字符串的模块。但是，在以下准则中，存在一个简单的`apply-loader`，可以与其他开源加载程序链接：

+   `jade-loader`：这将模板转换为导出函数的模块。

+   `apply-loader`：这将使用加载程序选项执行函数并返回基本的 HTML 代码。

+   `e idea HTML-loader`：这个加载程序接受 HTML 代码并输出一个有效的 JavaScript 模块。

+   加载程序可以链接，这意味着它们不一定必须输出 JavaScript，只要链中的下一个加载程序可以处理其输出。

Webpack 始终是模块化的，因此在使用链接加载程序时，让我们看一下关于这方面的建议。

# 模块化输出

在最好的情况下，您应该始终保持输出模块化。加载程序生成的模块应遵循与普通模块相同的设计启发。

这可能是显而易见的原因，但与现有项目的兼容性意味着应该遵循这一标准。这对于加载程序的链接也将变得越来越重要，正如之前讨论的那样。加载程序通常按顺序与彼此一起使用，并且许多 Webpack 项目需要安装和使用其中许多加载程序。

因此，遵循模块化输出约定将防止项目变得过于复杂，并且实际上可能导致 Webpack 捆绑的目的发生逆转，即使应用程序更小，更简洁和更优化。

这种传统的遵循或标准格式对大多数开发人员来说将是第二天性的，但在使用 Webpack 时，可能会有一些兼容性考虑您可能已经忽略了，因为它们对捆绑是如此特殊。其中一个考虑因素是加载程序的“状态”。

# 确保无状态

确保加载程序在模块转换之间不保留状态。每次运行都应始终独立于其他已编译的模块。

在编译过程中，您可能需要运行多个构建，以微调您的捆绑包，您不希望在每次运行时进行更正，而是保留每个构建的原始状态。

如果出现问题，这将使跟踪变得更加容易，因为您始终可以从源文件重新开始，而无需从命令行会话的最开始开始。

考虑与其他加载程序的兼容性也很重要。由于这是加载程序之间的约定，除非对于您的加载程序执行其特定任务是基本必要的，否则您应该保持相同的约定，并且如果是这样，应该向开发人员明确说明，以免出现错误。

如果传递状态对于您的加载程序的功能是必要的，那么有一个方便的解决方案可以提供对约定的遵循：加载程序实用程序包。

# 使用加载程序实用程序

为什么不利用`loader-utils`包呢？它提供了各种有用的工具，但其中最常见的之一是能够检索正在使用的任何加载程序传递的选项。除了`loader-utils`，还应该使用`schema-utils`包进行一致的基于 JSON 模式的验证。以下代码块显示了一个利用这两个包的示例，使用`loader.js`：

```js
import { getOptions } from 'loader-utils';
 import validateOptions from 'schema-utils';
const schema = {
  type: 'object',
  properties: {
    test: {
     type: 'string'
    }
  }
 };
export default function(source) {
  const options = getOptions(this);
  validateOptions(schema, options, 'Example Loader');
```

现在，我们可以对源代码应用一些转换，就像这样：

```js
return `export default ${ JSON.stringify(source) }`;
 }
```

这种转换将**字符串化**代码-基本上，将内容输出为一行代码，这对人类来说很难阅读，但对计算机来说是理想的。这通常也有助于隐私问题，如果有人希望手动复制代码。了解了这一点，让我们继续讨论加载程序依赖项的准则。

# 标记加载程序依赖项

如果加载器使用外部资源，比如从文件系统中读取，加载器必须指示这一点。这些信息用于使“可缓存”的加载器失效，并在监视模式下重新编译它们。接下来是一个简短的示例，说明如何使用`loader.js`中的`addDependency`方法来实现这一点：

```js
import path from 'path';
export default function(source) {
  var callback = this.async();
  var headerPath = path.resolve('header.js');
  this.addDependency(headerPath);
  fs.readFile(headerPath, 'utf-8', function(err, header) {
    if(err) return callback(err);
    callback(null, header + '\n' + source);
  });
 }
```

加载器和模块依赖之间存在一些差异。现在让我们讨论后者。

# 解析模块依赖

根据您使用的模块类型的不同模式，可能会使用不同的模式来指定任何依赖项。例如，在**层叠样式表**（**CSS**）中，使用`@import`和`URL(...)`语句。在这种情况下，这些依赖项应该由模块系统解析。

可以通过以下两种方式之一实现这一点：

+   通过将语句转换为`require`语句

+   使用`this.resolve`函数来解析路径

`css-loader`是第一种方法的一个很好的例子。它通过将`@import`语句替换为对其他样式表的请求和将`url(...)`替换为对被引用文件的请求，将依赖项转换为`require`语句。

关于 LESS 加载器，每个`@import`语句都无法转换为`require`语句，因为在单次迭代中必须编译**更少**的文件。因此，LESS 加载器将使用自定义路径解析逻辑来扩展 LESS 编译器。然后使用`this.resolve`方法来解析依赖项。

如果您使用的语言只接受相对**统一资源定位符**（**URL**），则可以使用*~*波浪号约定来指定对安装模块的引用。例如`url('~some-library/image.png')`。

# 提取公共代码

作为最佳实践的一部分，应该避免在加载器过程的每个模块中生成公共代码。更好的方法是在加载器中使用运行时文件，并生成对任何共享模块的`require`语句过程。这更适合 Webpack 解析代码的方式。

Webpack 的基本目的是编译项目，以便代码不会重复，因此这可能是不言而喻的，但加载器本身应该这样做，而不是留给 Webpack 核心处理。否则，应用程序将会变得非常庞大，或者编译时间会变得非常长。

如果您有编程插件的经验，您可能会忽视这个非常明显的原则，但是在这里提到它是值得的，因为它对于 Webpack 的操作和流程非常重要。

# 避免绝对路径

正如所暗示的，不应该在与模块相关的任何代码中插入绝对路径，因为如果根目录被移动，哈希将会破坏。另外，请注意，在`loader-utils`加载器中有一个`stringifyRequest`方法，可以用来将绝对路径转换为相对路径，以帮助自动化您的流程。

参考第二章，*使用模块和代码拆分*，如果您认为需要的话，可以重新了解绝对路径。

与公共代码一样，这是 Webpack 工作的非常基本的部分，如果在编写过程中没有考虑到这一点，您可能会忽视它，因此值得一提。相对路径是正确的方式。

关于标准的最后一点与对等依赖项有关。现在让我们来看看这些。

# 使用对等依赖项

在开发一个简单的包装器加载器时（基本上是在更高级代码内部充当外壳的代码），操作代码或包应该作为`peerDependency`包含。这是因为它将允许您使用`package.json`文件指定包的确切版本。

在下面的示例中，`sass-loader`将`node-sass`指定为对等依赖项。看一下代码：

```js
{
 "peerDependencies": {
   "node-sass": "⁴.0.0"
  }
}
```

这对于兼容性问题可能非常有价值，特别是在复杂的编程项目中。

# 单元测试

到目前为止，我们已经编写了一个自定义加载器，遵循了指南，甚至在本地运行了它。下一步是测试。以下示例是一个简单的单元测试过程。它使用`babel-jest` **Jest**框架和一些其他预设，允许使用`import/export`和`async/await`方法：

1.  我们将首先安装并保存这些内容，称为`devDependencies`，如下所示：

```js
npm install --save-dev jest babel-jest babel-preset-env 
```

前面的命令行条目以开发模式安装了**Jest**框架和**Babel**与**Jest**。

1.  接下来，我们必须查看`webpack.config.js`中使用的配置，关于这个特定的单元测试过程，如下所示：

```js
.babelrc
{
 "presets": [[
 "env",
 {
 "targets": {
 "node": "4"
 }
 }
 ]]
 }
```

1.  示例中加载器的功能是处理文本文件，并用加载器提供的选项替换`[name]`的任何实例。然后，它输出一个包含文本的有效**JavaScript**模块作为其默认导出，如下例所示，位于**`src/loader.js`**中：

```js
import { getOptions } from 'loader-utils';
export default function loader(source) {
 const options = getOptions(this);
source = source.replace(/\[name\]/g, options.name);
return `export default ${ JSON.stringify(source) }`;
 }
```

1.  这个加载器将用于处理以下文本文件，名为`test/example.txt`：

```js
Hi Reader!
```

1.  下一步有点复杂。它使用**Node.js** API 和`memory-fs`来执行 Webpack。这将避免内容输出到本地硬盘（非常方便），并使我们能够访问可以用来控制我们转换模块的统计数据。它从以下命令行开始：

```js
npm install --save-dev webpack memory-fs
```

1.  安装完成后，我们需要在其关联的编译器脚本上做一些工作。使用以下**`test/compiler.js` **文件：

```js
import path from 'path';
 import webpack from 'webpack';
 import memoryfs from 'memory-fs';
export default (fixture, options = {}) => {
 const compiler = webpack({
  context: __dirname,
  entry: `./${fixture}`,
  output: {
   path: path.resolve(__dirname),
   filename: 'bundle.js',
 },
 module: {
  rules: [{
  test: /\.txt$/,
  use: {
   loader: path.resolve(__dirname, '../src/loader.js'),
   options: {
     name: 'Alice'
    }
   }
 }]
 }
});
compiler.outputFileSystem = new memoryfs();
return new Promise((resolve, reject) => {
  compiler.run((err, stats) => {
 if (err) reject(err);
 if (stats.hasErrors()) reject(new Error(stats.toJson().errors));
resolve(stats);
 });
});
};
```

在前面的例子中，我们内联了我们的配置，但是也可以将配置作为`export`函数的参数来接受。这允许使用相同的编译器模块测试多个设置。

1.  这样做之后，我们现在可以编写我们的测试并添加一个`npm`脚本来运行它。让我们首先将以下代码添加到我们的`test/loader.test.js`文件中：

```js
import compiler from './compiler.js';
test('Inserts name and outputs JavaScript', async () => {
 const stats = await compiler('example.txt');
 const output = stats.toJson().modules[0].source;
expect(output).toBe('export default "Hi Reader!\\n"');
 });
 package.json
{
 "scripts": {
  "test": "jest"
 }
 }
```

前面的代码块显示了测试函数，它加载示例文本，如果程序工作正常的话。

现在一切都应该就绪了。

1.  现在可以运行代码了，我们将通过在命令行中运行`npm`构建并查看命令行窗口来检查新加载器是否通过了测试，如下所示：

```js
 Inserts name and outputs JavaScript (229ms)
Test Suites: 1 passed, 1 total
Tests: 1 passed, 1 total
Snapshots: 0 total
Time: 1.853s, estimated 2s
Ran all test suites.
```

如果您在相应文件中看到类似前面文本的内容，那么它通过了测试。干得好！

此时，您应该能够开发、测试和部署您的加载器。不要忘记与 Webpack 社区的其他成员分享您的创作，并帮助扩展开发人员的能力，同时留下您的印记。

在本指南中，我们涵盖了大量内容，我想您现在感觉像一个专家，已经构建了自定义库和加载器，但是进行实时编码会使您的技能更加令人印象深刻。这些是有用的知识，特别是在自定义工作中，解决方法和临时方法比经过试验的代码更有可能，所以让我们开始吧！

# 实时编码技巧

在这一部分，我们将看到一些非常有趣的东西，这些东西会让专家感觉像超级英雄，如果他或她陷入困境或者只是想炫耀的话。我们将讨论与**HMR**很好配合的加载器，比如`monkey-hot-loader`和`react-hot-loader`，以及`eval`和`__Eval`的各种用途。

首先，您应该注意**HMR**存在一个副作用。当它更新时，它总是重新评估整个模块。这包括依赖链，它会更新指向新模块。然而，我们可能只想让原始模块评估新代码，而不是整个模块。幸运的是，有一个绕过这个问题的方法：使用`monkey-hot-loader`！

# Monkey Hot Loader

这也意味着，如果您的模块具有副作用，比如启动服务器，那么`monkey-hot-loader`就不太适用。如果有全局状态，情况就不一样了，但如果编写正确，就不应该有全局状态。

另外，请注意，当我们通过“猴子补丁”来更改一个模块并用更新的内容来修补原始模块时。

在本节中，我们将详细探讨如何对顶级函数进行补丁。

`monkey-hot-loader`是一个 Webpack 加载程序，它解析 JavaScript 文件并提取文件中所有顶级函数的名称：

1.  例如，看一下以下代码，我们将其放在`app.js`文件中，但可以放在任何地方，因为该代码在全局范围内工作，并且将影响顶级函数：

```js
function foo() {
  return 5;
}
function bar() {
  return function() {
  // ...
 }
}
module.exports = function() {
 // ...
}
```

关于前面的例子，`monkey-hot-loader`的最新版本目前只提取函数名称`foo`和`bar`，因为只有这些函数可以进行补丁。还有一些其他类型的函数可以被制作成可补丁的，但为了解释的简单起见，我们现在先保持简单。

`foo`和`bar`函数可以通过设置它们为新函数来进行补丁。这是因为它们是顶级函数。更新后，函数将在相同的作用域中创建。然后很容易在那里注入新代码。

唯一的真正问题是，关于导出函数：使用导出函数的模块仍将引用旧的变体。需要做大量的工作来解决这个问题，我们现在将详细介绍。

这些函数的名称将被提供给`monkey-hot-loader`附加到每个模块的运行时代码。

1.  当模块最初运行时，它将遍历这些名称，并使每个函数都可以进行补丁，我们通过以下代码来实现这一点：

```js
var patched = function() {
 if(patchedBindings[binding]) {
  return patchedBindings[binding].apply(this, arguments);
 }
 else {
  return f.apply(this, arguments);
 }
};
 patched.prototype = f.prototype;
```

在这里，如果我们要对其进行补丁，`f`变量将引用名称`foo`。请注意，`patched`的语义应与`f`变量相同。对`patched`变量的任何调用应产生与对`f`的调用相同的结果。

在保持`foo`的初始语义的同时，我们安装了一个“钩子”来执行检查，以查看是否有新版本的函数要调用。在所有顶级函数都用这种变体替换后，我们可以通过将函数加载到`patchedBindings`中来简单地覆盖其中的任何一个。即使导出的函数也将调用新的变体。

目前，`monkey-hot-loader`将此顶级函数补丁实现为一个初始实验。您可以考虑使用`backend-with-webpack`项目来玩一下，看看如何将其与您的应用程序集成。

根据上下文，可能需要采用不同的启发式方法进行补丁。例如，如果您的前端使用**React**，那么大部分代码将存在于**React**库组件内。`react-hot-loader`对此效果非常好。但是，对于后端代码，大部分可能是类和方法，这种情况下最好的方法是对原型上的方法进行补丁。

# React 热补丁

`react-hot-loader`的工作原理是将原始模块绑定到任何新代码，无论是函数、类还是方法。它将修补所有**React**组件的方法以使用新方法。

这留下的问题是如何对原始模块进行补丁。如果尝试接受更新，事情将变得非常复杂。例如，如果更改闭包内的代码，将很难在不丢失现有状态的情况下对闭包进行补丁。可能可以使用**React**引擎的调试器 API，但要在整个项目中进行此更改可能会很困难：困难程度将在很大程度上取决于您的具体上下文。

请注意，当对闭包进行补丁时，最好只允许进行非常基本的补丁，因为直观地跟踪一切是很容易的。

当我们处理复杂和定制项目时，通常需要进行补丁代码。Webpack 中非常常见的一个工具是一个内在的补丁工具叫做`eval`。我们现在将更仔细地看一下它。

# Eval

无论是 React 热修补还是 Monkey 热加载器，我们都可以通过`eval`在整个模块范围内安装所有这些补丁版本。之后，我们保存模块的范围。这只会在模块第一次运行时发生：

1.  如果我们需要在特定范围内更改代码的能力，可以通过创建一个`eval`代理来在`app.js`文件中维护任何状态来实现这一点：

```js
var moduleEval = function(some code) {
 return eval(some code);
 }
```

此函数稍后通过处理程序传递给此模块的未来变体。

虽然前面提到的所有事情都发生在所讨论的模块的初始运行中，但通过不同的路径进行连续更新。在这种情况下，整个模块将再次进行评估，而不是迭代每个顶级绑定，然后调用`func.toString()`返回函数代码，然后使用`moduleEval`在原始模块的范围内重新评估代码-引用原始状态。

1.  然后，将此`eval`函数安装在`patchedBindings`中，以便在系统未来的任何调用中使用：

```js
bindings.forEach(function(binding) {
 // Get the updated function instance
 var f = eval(binding);
// We need to rectify the function in the original module so
 // it references any of the original state. Strip the name
 // and simply eval it.
 var funcCode = (
 '(' + f.toString().replace(/^function \w+\(/, 'function (') + ')'
 );
 patchedBindings[binding] = module.hot.data.moduleEval(funcCode);
 });
```

在理想情况下，我们可能会获取模块的源代码更新，并避免运行模块，因为我们只想要代码的字符串形式。

事实上，可以避免整个`func.toString()`和`moduleEval`过程，而简单地不支持任何全局状态，尽管全局状态对于调试操作非常有用。这对于简单的 REPL（读取-评估-打印循环）交互特别有效。然而，类没有这个问题，因为它们的所有状态都是实例的一部分，这就是为什么`react-hot-loader`可以在没有这个技巧的情况下正常工作。

对于未经培训的人来说，REPL 也被称为交互式顶层或语言外壳，它接受单个用户输入并对其进行评估。

请注意，在 Webpack 5 中，`eval()`在生产模式下存在与`optimization.innerGraph`相关的已知问题。

对于`_eval`，有一个可用的技巧，这是非常有用的。现在让我们来看看这一部分。

# __Eval 技巧

现在是最后一个技巧的时间，这可能是整本书中最大的技巧！`_eval()`函数将字符串作为函数的表达式进行评估。这可以与热加载一起使用，以允许立即评估整个项目中的代码。这本质上就是`_Eval`的技巧。现在让我们更深入地探讨一下。

如果我们想要一个 REPL 来评估模块内的代码，并且能够打开模块以选择要在其中评估的上下文，那么我们无法使用这种基础设施，但我们可以通过`app.js`文件中的以下示例实现大部分功能：

```js
function __eval() {
 var user = getLastUser();
 console.log(findAllDataOn(user));
 }
```

完成这些操作并定义一个名为`__eval`的函数后，`monkey-hot-loader`将在模块更新时执行它。这对于即时反馈非常有用。通过这种方法，您可以调用一些 API 并记录结果，然后即时对这些 API 进行调整，直到看到想要的结果。这样，您只需进行一些编码更改，保存文件，然后立即查看更新后的输出。

此外，您可以使用`__eval`的代码形式作为全局使用的脚本，并允许典型的 HMR 系统在每次更新模块时运行。也就是说，任何具有副作用的模块都需要专门的代码。此外，您可以在评估中构建跨评估的状态以进行调试。

与旧的 Lisp 风格不同（即选择代码并按 Ctrl + E 运行），此技术是针对每个模块进行的，并且您可以选择要在其中运行代码的上下文。

一个考虑是，在`strict`模式下，不能使用`_eval`函数引入新变量，例如更改变量的值。这也适用于`__eval`，因此在开始之前值得记住。

# 总结

本章带您了解了 Webpack 的一些更高级的功能，比如库编写和实时编码技巧，使用`_Eval`技术进行热加载，以实现项目间的即时反馈。这包括了如何自定义加载器的详细解释和示例，甚至修补顶层函数。

现在，您应该对手动捆绑和实时编码有足够深入的了解，可以与任何专家并驾齐驱。为什么不通过在本章末尾的测验中展示这种专业知识来向自己证明呢？如果您在工作面试或向重要客户做演示时能够快速表达自己的专业知识，这将对您大有裨益。

整本书都详细而全面地介绍了如何熟练使用 Webpack，并将应用程序开发提升到全新的水平。随着您的 Webpack 捆绑的发展，本章将变得越来越重要，您可以确信，这一章将成为未来许多项目的书签。

一旦您尝试了练习测验问题，您可能想翻回书的开头，对每一章进行自我测试。您将在本指南后面的一个单独章节中找到评估答案。成功完成将使您的专业知识毋庸置疑，所以试一试吧。

# 问题

1.  Webpack 可以用来捆绑库以及应用程序吗？

1.  在编写库时，如何将外部库排除在捆绑包之外？

1.  Webpack 提供了四种公开自定义库的方式。它们是什么？

1.  在构建自定义模块时为什么不应该使用绝对路径？

1.  `__eval`的函数前缀如何帮助实现即时反馈？

1.  为什么开发人员必须通过加载器指示读取外部资源，比如文件系统？

1.  当加载器被链接时，它们是如何执行的？
