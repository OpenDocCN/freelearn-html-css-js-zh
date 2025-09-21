

# Node.js 核心库

在本章中，我们将深入研究 Node.js 的核心库，并探讨模块化代码的技术。JavaScript 已经从仅限于浏览器的限制中走出来，Node.js 为我们提供了新的代码结构方式。我们将从理解在浏览器中组织代码的历史局限性以及它们如何导致各种模块系统的开发开始。我们将主要关注两个模块系统，**CommonJS**（**CJS**）和**ECMAScript 模块**（**ESM**），并讨论它们的用法、导入和导出。实现这两个系统之间的互操作性至关重要，我们将探讨使它们无缝工作的策略。

理解 Node.js 核心库的结构是关键。我们将更深入地研究包括 `fs` 和 `http` 在内的核心库，这些库处理文件操作，并探讨使用回调、同步函数和承诺进行异步 I/O 操作的使用。

此外，还将讨论更多与使用 C++ 插件扩展 Node.js 功能和通过 `child_process` 库执行外部命令相关的先进主题。我们还将回顾各种命令行选项（包括启用实验性功能和控制内存分配）以及允许您自定义 Node.js 行为的环境变量。我们将提供如何使用这些选项来启用实验性功能、控制内存分配以及微调您的 Node.js 应用程序的示例。

总结一下，以下是本章我们将探讨的主要主题：

+   如何使用 ESM 和 CJS 方法创建和消费模块

+   如何在 ESM 和 CJS 模块之间进行互操作

+   Node.js 核心库接口的结构

+   在开始使用 Node.js 时，最相关的 Node.js 核心库是什么

如何通过使用命令行选项和 `NODE_OPTIONS` 环境变量来扩展 Node.js 功能

# 技术要求

本章的代码文件可以在 [`github.com/PacktPublishing/NodeJS-for-Beginners`](https://github.com/PacktPublishing/NodeJS-for-Beginners) 找到。

查看本章的代码执行视频，请访问 [`youtu.be/WQzdXAFxdsc`](https://youtu.be/WQzdXAFxdsc)

# 模块化您的代码（ESM 与 CJS）

许多年来，JavaScript 仅限于浏览器中，我们组织代码的唯一方式是使用在 HTML 页面中按正确顺序加载的脚本文件。这是通过在 HTML 文件中包含特定引用来实现的，例如以下内容：

```js
<!-- External Sources -->
<script src="img/jquery-3.7.0.min.js"></script>
<!-- Other files -->
<script src="img/script1.js"></script>
<!-- Direct Scripts -->
<script>
console.log("Hello world");
</script>
```

这种方法不可扩展，很容易污染全局作用域。为了解决这个问题，历史上我们使用了 IIFE 模式和模块模式。随着 JavaScript 的采用率开始增长，现代网站所需的 JavaScript 量急剧增加，社区开始创建库和框架来解决上述问题。结果包括 RequireJS ([`requirejs.org/`](https://requirejs.org/))。

多年来，我们有四种不同的方式来组织我们的代码：

+   **CommonJS (CJS)**

+   **ECMAScript** **Modules (ESM)**

+   **异步模块** **定义 (AMD)**

+   **通用模块** **定义 (UMD)**

在这本书中，我们将重点关注前两种方法，CJS 和 ESM。目前 CJS 是 Node.js 的默认模块系统，但自从 Node.js 12 发布以来，ESM 现在也已可用。在本节中，我们将探讨如何使用这两种方法创建和消费模块。

重要提示

现在，在浏览器环境中，使用模块打包器（如 webpack 或 Rollup）来整合我们的代码是非常常见的。然而，在 Node.js 中，我们仍然直接使用 CJS 或 ESM。在本节中，我们将探讨如何使用这两种方法创建和消费模块。

## CommonJS (CJS)

CommonJS 是 Node.js 默认使用的模块系统。这个模块系统是同步的，基于 `require` 和 `module.exports` 函数。需要注意的是，这个模块系统不是 ECMAScript 规范的一部分，但在 Node.js 生态系统中是最常用的模块系统，特别是如果你在寻找文档或教程的话。

在这里，我们需要了解 CJS 使用的两个方面：导入和导出。让我们从导入开始。

### 导入

因此，在我们的项目中，有两个文件，`utils.js` 和 `index.js`。在这个例子中，我们在 `index.js` 文件中从 `utils.js` 文件导入 `sayHello` 函数，如下所示：

```js
const sayHello = require('./utils.js');
sayHello();
```

`require` 函数是 Node.js 中可用的全局函数，用于导入模块。`require` 函数接收一个字符串作为参数，这个字符串是我们想要导入的模块的路径。在这个例子中，我们使用的是相对路径，但也可以使用绝对路径，甚至可以使用安装在 `node_modules` 文件夹中的模块名称。

### 导出

在这个例子中，我们在 `utils.js` 文件中导出 `sayHello` 函数：

```js
function sayHello() {
  console.log('Hello world');
}
module.exports = sayHello;
```

`module.exports` 是 Node.js 中可用的全局对象，用于导出模块。在这个例子中，我们导出 `sayHello` 函数，但我们可以导出任何类型的值。

如果你执行 `index.js` 文件，你会看到以下输出：

```js
$ node index.js
Hello world
```

但如果我们执行 `utils.js` 文件，我们将看不到任何东西。即使文件被执行，`sayHello` 函数本身也不会执行，只是定义了：

```js
$ node utils.js
```

### 导出对象结构

在导出模块时，最常用的结构是对象结构，因为它非常灵活，允许我们导出多个值。如果我们想导出多个值，我们可以使用`exports`对象：

```js
// You can export directly
exports.sayHello = () => {
  console.log('Hello world');
}
function sayGoodbye() {
  console.log('Goodbye world');
}
// You can also export using references
exports.sayGoodbye = sayGoodbye;
```

但我们也可以直接使用`module.exports = {}`导出一个对象：

```js
const sayHello = () => {
  console.log('Hello world');
}
function sayGoodbye() {
  console.log('Goodbye world');
}
module.exports = {
  sayHello,
  sayGoodbye
}
```

我推荐前面的选项，因为它在处理大型文件时更易读。为了导入导出的值，我们可以使用解构语法：

```js
const { sayHello, sayGoodbye } = require('./utils.js');
sayHello();
sayGoodbye();
```

### JSON 支持

是的，你可以在 Node.js 项目中直接添加 JSON 文件，而无需使用任何外部库或解析内容：

```js
{
  "name": "John",
  "lastName": "Doe"
}
```

现在我们可以直接引入该文件：

```js
const user = require('./user.json');
console.log(user);
// { name: 'John', lastName: 'Doe' }
```

Node.js 中模块的工作方式与 IIFE 模式非常相似。当我们导入一个模块时，代码会被执行并且模块会被缓存。如果我们再次导入相同的模块，代码将不会再次执行，并且模块将从缓存中检索。基本上，模块只执行一次（单例模式）。

例如，如果我们修改了导入的 JSON 文件，一旦导入模块后，这些更改将不会反映在导入的模块中，因为它是只读的，并且内容被缓存在了程序内存中。

## ECMAScript 模块（ESM）

Node.js 12 引入了对`import`和`export`关键字的支持。

为了使用 Node.js 20.11.0 中的模块，你需要创建一个`package.json`文件并添加以下配置：

```js
{
  "type": "module"
}
```

重要提示

在*第六章*中，我们将探讨如何创建`package.json`文件以及如何对其进行深度配置。

### 基本用法

在本例中，我们正在导出`utils.js`文件中的`sayHello`函数：

```js
export default function sayHello() {
  console.log('Hello world');
}
```

`export`关键字用于导出模块。在这种情况下，我们正在导出`sayHello`函数，但我们可以导出任何类型的值。注意，我们使用了`default`关键字，这是因为我们正在导出一个单一值。如果我们想导出多个值，我们可以使用不带`default`关键字的`export`关键字。

在本例中，我们正在从`utils.js`文件中导入`sayHello`函数：

```js
import sayHello from './utils.js';
sayHello();
```

### 导出对象结构

在导出模块时，最常用的结构是对象结构，因为它非常灵活，允许我们导出多个值。如果我们想导出多个值，我们可以使用`export`关键字：

```js
const sayHello = () => {
  console.log('Hello world');
}
function sayGoodbye() {
  console.log('Goodbye world');
}
export { sayGoodbye, sayHello };
```

在本例中，我们以几种方式从`utils.js`文件中导入`sayHello`和`sayGoodbye`函数：

```js
// Import values directly
import { sayHello, sayGoodbye } from './utils.js';
// Use wildcards to import all the exported values
import * as utils from './utils.js';
sayHello();
utils.sayHello();
```

### 支持 JSON 文件

在使用 ESM 时，我们无法直接导入 JSON 文件，就像我们为 CJS 所做的那样。如果我们尝试导入一个 JSON 文件，我们将得到以下错误：

```js
TypeError [ERR_IMPORT_ASSERTION_TYPE_MISSING]: Module "file:///{REDACTED}/user.json" needs an import assertion of type "json"
```

在未来，将可以直接导入 JSON 文件，有一个提案([`github.com/tc39/proposal-import-attributes`](https://github.com/tc39/proposal-import-attributes))将允许我们使用导入属性，例如 `import json from "./foo.json" with { type: "json" };`。但到目前为止，我们需要使用一种变通方法来导入 JSON 文件。我们可以通过理解 ESM 和 CJS 之间的互操作性来修复这个错误。

## 理解模块互操作性是如何工作的

虽然 ESM 是未来，但仍有许多库和框架仍在使用 CJS。好消息是 Node.js 支持这两种模块系统，并且可以在同一个项目中无问题地使用它们，但我们需要注意一些事项以确保其正常工作。

重要提示

互操作性一直是 Node.js 社区中的一个非常具有争议的话题，并且对此有很多讨论。如果你想了解更多，我推荐你阅读 Gil Tayar 的这篇文章：[`medium.com/@giltayar/native-es-modules-in-nodejs-status-and-future-directions-part-i-ee5ea3001f71`](https://medium.com/@giltayar/native-es-modules-in-nodejs-status-and-future-directions-part-i-ee5ea3001f71)。

### ESM 中的 JSON 文件

在前面的章节中，我们看到了在 ESM 中今天无法直接导入 JSON 文件。但我们可以使用 Node.js 内置的 `module` 库来导入 JSON 文件。`module` 库是一个全局对象，在所有模块中都是可用的，它包含 `createRequire` 方法，允许我们创建一个 `require` 函数，该函数可以用来导入 CJS 模块：

```js
import { createRequire } from "module";
const require = createRequire(import.meta.url);
const user = require("./user.json");
console.log(user);
// { name: 'John', lastName: 'Doe' }
```

### 文件扩展名 (.cjs 和 .mjs)

为了在同一个项目中使用这两种模块系统，我们需要在我们的文件中使用不同的文件扩展名。`.mjs` 扩展名用于 ESM 模块，而 `.cjs` 扩展名用于 CJS 模块。

重要提示

如果你使用 `.js` 扩展名来为你的文件命名，Node.js 将默认尝试使用 CJS 模块系统，就像你使用 `.cjs` 扩展名一样。

这里是使用这两种模块系统的项目的文件结构：

```js
├── index.cjs
├── index.mjs
├── utils.cjs
└── utils.mjs
```

`utils.cjs` 文件是一个 CJS 模块：

```js
const sayGoodbye = () => {
  console.log('Goodbye world');
}
module.exports = { sayGoodbye }
```

`utils.mjs` 文件是一个 ESM 模块：

```js
const sayHello = () => {
  console.log('Hello world');
}
export { sayHello }
```

`index.mjs` 文件是一个 ESM 模块。只要我们使用不同的文件扩展名，我们就可以在同一个文件中结合使用这两种模块系统：

```js
import { sayHello } from './utils.mjs';
import { sayGoodbye } from './utils.cjs';
sayHello();
sayGoodbye();
```

`index.cjs` 文件是一个 CJS 模块。在这种情况下，由于 `require` 被设计为同步函数，而 ESM 模块是异步的，因此不能直接导入 ESM 模块。但我们可以使用 `import` 函数异步导入 ESM 模块：

```js
const { sayGoodbye } = require('./utils.cjs');
import("./utils.mjs").then(({ sayHello }) => {
    sayHello();
    sayGoodbye();
});
```

重要信息

这种导入模块的方式是标准的一部分，被称为动态导入([`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import#dynamic_imports`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import#dynamic_imports))。

在这本书中，我们默认使用 ESM 模块，但当我们需要与其他库和框架进行互操作性时，我们将使用 CJS 模块。

现在我们对如何以不同格式创建模块有了更清晰的认识，让我们在下一节中探讨 Node.js 核心 API 如何使用类似的方法来公开我们在项目中非常常用的众多功能。

# 核心库的结构

几年来，Node.js 已经发展壮大，核心库也是如此。有许多库可供我们使用，了解它们的结构对于能够正确使用它们非常重要。

大多数核心库都非常简单，并且以类似的方式构建，因此你知道在实际应用中可以期待什么。一旦你学会了如何使用其中一个，你将能够毫无问题地使用其余的库。

此外，你将能够创建自己的库并在 npm 上发布它们，其他开发者将能够轻松地使用它们，但我们在下一章中会讨论这一点。

## 库结构

让我们以`fs`库为例。`fs`库用于与文件系统交互，它是 Node.js 中最常用的库之一。

任何执行 I/O 操作的库都是异步的。从历史上看，Node.js 提供了两种处理 I/O 操作的方法：回调或同步函数。虽然回调仍然被支持，但 Node.js 目前提供了相同的功能，提供了一个 promise 接口。

在这个示例中，我们将使用`readFile`函数异步读取文件。这个函数接收要读取的文件的路径，以及当文件被读取时将被调用的回调函数。回调函数接收两个参数：一个错误对象和文件的内容：

```js
import { readFile } from 'node:fs';
readFile('hello.txt', (err, content) => {
  if (err)  {
    console.error("OMG, there is an error:", err);
    return;
  }
  console.log(`File content: ${content}`);
  // File content: Hello world
});
```

当你运行前面的示例时，它将抛出一个错误，因为文件不存在。然而，我们使用回调中的错误优先模式来处理错误。你可以看到“OMG，那里... 错误信息”。现在，如果你创建一个包含内容`Hello world`的`hello.txt`文件，并再次运行脚本，你将看到预期的内容被打印出来。

在下一个示例中，我们将使用`readFileSync`函数来同步读取文件。这个函数接收要读取的文件的路径，并返回文件的内容：

```js
import { readFileSync } from 'node:fs';
try {
  const content = readFileSync('hello.txt');
  console.log(`File content: ${content}`);
  // File content: Hello world
} catch (err) {
  console.error("OMG, there is an error:", err);
};
```

最后，在这个示例中，我们将使用`readFile`函数异步读取文件。这个函数接收要读取的文件的路径，并返回一个当文件被读取时将被解决的 promise。这个 promise 将被文件的内容解决：

```js
import { readFile } from 'node:fs/promises';
readFile('hello.txt')
  .then(content => console.log(`File content: ${content}`))
  .catch(err => console.error("OMG, there is an error:", err))
```

### 无前缀的核心库

从历史上看，Node.js 提供了不带`node:*`前缀的核心库，例如`const { readFile } from 'fs'`。这主要是为了向后兼容。但建议使用带有前缀`node:*`的新语法。你可以在互联网上找到许多使用旧语法的示例。更多信息请参阅[`nodejs.org/api/modules.html`](https://nodejs.org/api/modules.html)。

### CJS 支持

所有核心库都作为 CJS 模块提供，因此你可以在项目中使用它们而不会遇到任何问题。你可以使用`require`函数来导入它们：

| CJS | ESM |
| --- | --- |
| `const { readFile } =` `require('node:fs')` | `import { readFile }` `from 'node:fs'` |
| `const { readFileSync } =` `require('node:fs')` | `import { readFileSync }` `from 'node:fs'` |
| `const { readFile } =` `require('node:fs/promises')` | `import { readFile }` `from 'node:fs/promises'` |
| `const { readFile } =` `require('node:fs')` | `import { readFile }` `from 'node:fs'` |
| `const { readFileSync } =` `require('node:fs')` | `import { readFileSync }` `from 'node:fs'` |

### 其他接口

你将经常使用的其他核心库，例如`http`或`https`，结构类似，并提供了一个用于处理事件的接口。我们将在*第七章*中深入探讨这个主题。

## 稳定指数

稳定指数是一个表示核心库稳定性的数字。稳定指数介于 0 到 3 之间，其中 0 表示已弃用，1 表示实验性，2 表示稳定，3 表示遗留。

你可以在官方文档中找到每个核心库的稳定指数，以及有关稳定指数的更多详细信息，请参阅[`nodejs.org/dist/latest-v20.x/docs/api/documentation.html#stability-index`](https://nodejs.org/dist/latest-v20.x/docs/api/documentation.html#stability-index)。

重要提示

如果你刚开始使用 Node.js，你应该使用稳定指数为 2 或 3 的核心库。稳定指数为 0 或 1 的核心库不建议用于生产环境。

让我们看看 Node.js 20 的一些示例：

+   权限模型([`nodejs.org/docs/latest-v20.x/api/permissions.html#permission-model`](https://nodejs.org/docs/latest-v20.x/api/permissions.html#permission-model))：这是一个允许我们限制对系统资源（如网络或文件）访问的 API。目前，它处于积极开发中（稳定性=1），因此你可以尝试使用它，但它还不够成熟，不能用于构建生产系统，因为 API 可能会更改或出现意外的行为。

+   http ([`nodejs.org/docs/latest-v20.x/api/http.html#http`](https://nodejs.org/docs/latest-v20.x/api/http.html#http))：这是自 Node.js 诞生以来用于构建网络服务器应用程序和对外部资源发起 HTTP 请求的 API。目前它是稳定的（稳定性=2），但某些方法是遗留的（稳定性=3）。这个库非常适合在生产系统中使用。

## 其他核心库

`fs` 库只是一个例子；在这本书中，我们将介绍最重要的核心库，但你可以在 Node.js 文档中找到所有核心库的文档，网址为 [`nodejs.org/docs/latest-v20.x/api/index.html`](https://nodejs.org/docs/latest-v20.x/api/index.html)。

在我谦逊的观点中，当你刚开始使用 Node.js 时，最重要的核心库如下，按字母顺序排列：

+   `Buffer` 在内存中高效地处理二进制数据，常用于文件操作和网络通信等任务。

+   `Crypto` 提供了加密、解密、哈希和数字签名等加密功能。

+   `Events` 允许我们在应用程序内部创建、触发和监听事件。

+   `File System` 提供了一个稳定的接口来处理文件系统（文件、文件夹、创建、删除等）。

+   `HTTP` 允许我们创建 HTTP 服务器并执行 HTTP 请求。

+   `OS` 提供了各种实用工具来检索有关系统架构、平台、CPU、内存、网络接口等信息。

+   `Path` 提供了处理文件路径和目录路径的实用工具。

+   `Process` 提供了关于当前 Node.js 进程方面的信息和控制，包括环境变量、生命周期事件等。

+   `Stream` 提供了可读和可写流，以及用于在数据通过时修改数据的转换流。这个模块对于在 Node.js 中构建可扩展和内存高效的、处理大量数据的应用程序至关重要。

+   `Timers` 包含了诸如 `setTimeout()`、`setInterval()` 和 `setImmediate()` 等函数。

还有其他一些核心库对于扩展 Node.js 的功能非常重要，但当你刚开始使用 Node.js 时，它们并不那么重要。

例如，`child_process` 库对于从 Node.js 执行外部命令（如 `ls` 和 `cat`）、复杂的应用程序（如 `ffmpeg` 和 `imagemagick`）以及直接执行 Python 脚本至关重要。

`C++ Addons` ([`nodejs.org/dist/latest-v20.x/docs/api/addons.html`](https://nodejs.org/dist/latest-v20.x/docs/api/addons.html)) 对于使用 C++ 代码扩展 Node.js 的功能非常重要。当你需要在 Node.js 应用程序中使用 C++ 库时，这非常有用。

## 命令行选项

Node.js 提供了许多命令行选项和环境变量，您可以使用它们来定制 Node.js 的行为。您可以在 Node.js 文档中找到完整的命令行选项列表，文档地址为 [`nodejs.org/dist/latest-v20.x/docs/api/cli.html#cli_command_line_options`](https://nodejs.org/dist/latest-v20.x/docs/api/cli.html#cli_command_line_options)。

例如，您可以使用 `--experimental-json-modules` 命令行选项来启用 ESM 中的 JSON 模块，例如 `node --``experimental-json-modules index.js`。

`index.js` 文件的代码如下：

```js
import data from './data.json' assert { type: 'json' };
console.log(data);
```

这确实有效，终端输出将指出 JSON 模块是实验性的：

```js
(node:21490) ExperimentalWarning: Import assertions are not a stable feature of the JavaScript language. Avoid relying on their current behavior and syntax as those might change in a future version of Node.js.
(Use `node --trace-warnings ...` to show where the warning was created)
(node:21490) ExperimentalWarning: Importing JSON modules is an experimental feature and might change at any time
```

除了启用实验性功能外，您还可以使用 `--max-old-space-size` 命令行选项来增加 Node.js 的 RAM 使用限制。当您处理大文件、内存中有大量数据或调试复杂的内存泄漏时，这非常有用。

例如，您可以使用 `--max-old-space-size=4096` 命令行选项将 RAM 限制增加到 4GB：`node --``max-old-space-size=4096 index.js`。

重要提示

您无法使用计算机中的所有 RAM，因为操作系统和其他应用程序也需要一些 RAM 来正常工作。

### 环境变量

您可以使用环境变量来定制 Node.js 的行为。您可以在 Node.js 文档中找到完整的环境变量列表，文档地址为 [`nodejs.org/dist/latest-v20.x/docs/api/cli.html#cli_environmental_variables`](https://nodejs.org/dist/latest-v20.x/docs/api/cli.html#cli_environmental_variables)。

有时候使用环境变量而不是直接使用命令行选项会更方便，例如在使用基于 UNIX 的系统时：

```js
# Define the environmental variable
export NODE_OPTIONS='--experimental-json-modules,--max-old-space-size=4096'
# Run the Node.js application as usual
node index.js
```

上述代码允许您使用 `NODE_OPTIONS` 环境变量来设置您想要使用的命令行选项。当您使用 `nodemon` 或 `pm2` 等工具运行 Node.js 应用程序时，这非常有用。我们将使用来自 *第十二章* 的许多环境变量。

# 摘要

在本章中，我们介绍了 Node.js 中模块的工作方式、CJS 和 ESM 之间的区别以及它们之间的互操作性。

此外，我们还介绍了 Node.js 的核心库、如何使用它们、它们的结构和稳定性指数。我们列出了在开始使用 Node.js 时最重要的核心库，以及在其他更高级项目中变得至关重要的库。

最后，我们学习了如何使用命令行选项和环境变量来修改 Node.js 的行为。

在下一章中，我们将深入学习如何使用 **node.js 包管理器**（**npm**）。我们将发布我们的第一个包，并了解我们如何将可用的庞大模块生态系统集成到我们的 Node.js 项目中。

# 进一步阅读

+   Node.js 文档：[`nodejs.org/dist/latest-v20.x/docs/api/documentation.html#documentation_stability_index`](https://nodejs.org/dist/latest-v20.x/docs/api/documentation.html#documentation_stability_index)

)

+   保持 Node.js 内核小巧：[`medium.com/the-node-js-collection/keeping-the-node-js-core-small-137f83d18152`](https://medium.com/the-node-js-collection/keeping-the-node-js-core-small-137f83d18152)

+   Moz://a Hacks | 深入了解 ES6：模块：[`hacks.mozilla.org/2015/08/es6-in-depth-modules`](https://hacks.mozilla.org/2015/08/es6-in-depth-modules)

+   Node.js 核心中的 Promises API：第“Do”，更新！ - Joe Sepi，IBM：[`www.youtube.com/watch?v=f7YSsYQmNSI`](https://www.youtube.com/watch?v=f7YSsYQmNSI)
