# 第二章：设置开发环境

除了作为 OCaml 的新语法之外，Reason 还是一个工具链，可以让我们轻松入门。在本章中，我们将做以下事情：

+   了解 Reason 工具链

+   配置我们的编辑器

+   使用 `bsb` 启动一个纯 Reason 项目

+   了解 `bsconfig.json`

+   编写一个操作 DOM 的纯 Reason 应用程序示例

+   使用 `bsb` 启动一个 ReasonReact 项目

+   在 Reason 项目中熟悉使用 `webpack`

要跟着做，请克隆本书的 GitHub 存储库，并从本章的目录开始。您也可以从一个空白项目开始：

```js
git clone https://github.com/PacktPublishing/ReasonML-Quick-Start-Guide.git
cd ReasonML-Quick-Start-Guide
cd Chapter02/pure-reason-start
npm install
```

本章旨在让您熟悉 Reason 工具链。我们将为纯 Reason 项目和 ReasonReact 项目分别设置开发环境。跟着做一遍后，您将足够熟悉来调整开发环境以满足您的喜好。不用担心搞砸了什么，因为我们将在另一个目录中从零开始，即 第三章 *创建 ReasonReact 组件*。

# Reason 工具链

在撰写本文时，Reason 工具链本质上是 BuckleScript—Reason 的合作项目—和熟悉的 JavaScript 工具链，即 `npm` 和 `webpack`（或其他 JavaScript 模块打包工具）。

由于 BuckleScript 编译成了 ES5 版本的 JavaScript，所以不再需要 `babel`。编译输出可以配置为使用 CommonJS、AMD 或 ES 模块格式。Reason 强大的静态类型系统取代了 Flow 和 ESlint 的需求。此外，Reason 的编辑器插件都带有 `refmt`，这本质上就是 Reason 的 `prettier`。

# 安装 BuckleScript

BuckleScript 是一个编译器，它接受 OCaml AST 并生成干净、可读和高性能的 JavaScript。可以通过 `npm` 安装它，如下所示：

```js
npm install -g bs-platform
```

安装 `bs-platform` 提供了一个名为 `bsb` 的二进制文件，这是 BuckleScript 的构建系统。

未来，Reason 工具链将大大简化针对本机平台和 JavaScript 的目标。目前，Reason 通过使用名为 `bsb-native` 的 `bsb` 分支编译为本机代码。

# 编辑器配置

Reason 支持各种编辑器，包括 VSCode、Sublime Text、Atom、Vim 和 Emacs。推荐使用 VSCode。要配置 VSCode，只需安装 `reason-vscode` 扩展即可。

请参阅编辑器特定的说明文档。

Reason 编辑器支持文档可以在 [`reasonml.github.io/docs/editor-plugins`](https://reasonml.github.io/docs/editor-plugins) 找到。

# 设置一个纯 Reason 项目

`bsb` 二进制文件包括一个项目生成器。我们将使用它使用 `basic-reason` 主题创建一个纯 Reason 项目。运行 `bsb -themes` 以查看所有可用的项目模板：

```js
Available themes: 
basic
basic-reason
generator
minimal
node
react
react-lite
tea
```

由于 BuckleScript 可以与 OCaml 和 Reason 一起使用，因此有些主题仅适用于 OCaml 项目。也就是说，可以在任何 BuckleScript 项目中自由混合 OCaml 的 `.ml` 文件和 Reason 的 `.re` 文件。

在本章中，我们将专注于使用 `basic-reason` 和 `react` 模板。如果您感兴趣，`react-lite` 主题类似于 `react`，只是用一个更简单、更快速、更可靠的模块打包工具替换了 `webpack`，该模块打包工具仅用于开发目的。

让我们首先创建一个纯 Reason 项目：

```js
bsb -init my-first-app -theme basic-reason
cd my-first-app
```

当我们在编辑器中打开项目时，我们看到以下项目结构：

```js
├── .gitignore
├── README.md
├── bsconfig.json
├── node_modules
│   ├── .bin
│   │   ├── bsb
│   │   ├── bsc
│   │   └── bsrefmt
│   └── bs-platform
├── package.json
└── src
    └── Demo.re
```

总的来说，这里没有太多东西，这在从 JavaScript 转过来的人来说有点令人耳目一新。在 `node_modules` 中，我们看到了 `bs-platform` 以及一些二进制文件：

+   `bsb`：构建系统

+   `bsc`：编译器

+   `bsrefmt`：这本质上就是 JavaScript 的 `prettier`，但用于 Reason。

正如我们将很快看到的，`bsb` 二进制文件在 `npm` 脚本中使用。`bsc` 二进制文件很少直接使用。`bsrefmt` 二进制文件被编辑器插件使用。

在 `Demo.re` 中，我们看到一个简单的日志消息：

```js
/* Demo.re */
Js.log("Hello, BuckleScript and Reason!");
```

`package.json` 看起来有点熟悉。`scripts` 字段显示了我们当前可用的 `npm` 脚本：

```js
/* package.json */
{
  "name": "my-first-app",
  "version": "0.1.0",
  "scripts": {
    "build": "bsb -make-world",
    "start": "bsb -make-world -w",
    "clean": "bsb -clean-world"
  },
  "keywords": [
    "BuckleScript"
  ],
  "author": "",
  "license": "MIT",
  "devDependencies": {
    "bs-platform": "⁴.0.5"
  }
}
```

运行`npm run build`将`Demo.re`编译为 JavaScript。默认情况下，编译输出会出现在源文件旁边，名称为`Demo.bs.js`。它是如何知道要编译哪些文件，以及在哪里输出它们的？这就是`bsconfig.json`的作用。

# bsconfig.json 文件

`bsconfig.json`文件是所有 BuckleScript 项目的必需文件。让我们来探索一下：

```js
// This is the configuration file used by BuckleScript's build system bsb. Its documentation lives here: http://bucklescript.github.io/bucklescript/docson/#build-schema.json
// BuckleScript comes with its own parser for bsconfig.json, which is normal JSON, with the extra support of comments and trailing commas.
{
  "name": "my-first-app",
  "version": "0.1.0",
  "sources": {
    "dir" : "src",
    "subdirs" : true
  },
  "package-specs": {
    "module": "commonjs",
    "in-source": true
  },
  "suffix": ".bs.js",
  "bs-dependencies": [
      // add your dependencies here. You'd usually install them normally through `npm install my-dependency`. If my-dependency has a bsconfig.json too, then everything will work seamlessly.
  ],
  "warnings": {
    "error" : "+101"
  },
  "namespace": true,
  "refmt": 3
}
```

我们很快将更改其中一些默认值，以便更加熟悉 BuckleScript 的配置文件。让我们首先将以下代码添加到`Demo.re`中：

```js
type decision =
  | Yes
  | No
  | Maybe;

let decision = Maybe;

let response =
  switch (decision) {
  | Yes => "Yes!"
  | No => "I'm afraid not."
  };

Js.log(response);
```

正如您所看到的，`switch`表达式没有处理所有`decision`的可能情况。运行`npm run build`的结果如下：

```js
ninja: Entering directory `lib/bs'
[3/3] Building src/Demo.mlast.d
[1/1] Building src/Demo-MyFirstApp.cmj

  Warning number 8
  .../Demo.re 9:3-12:3

   7 │ 
   8 │ let response =
   9 │ switch (decision) {
  10 │ | Yes => "Yes!"
  11 │ | No => "I'm afraid not."
  12 │ };
  13 │ 
  14 │ Js.log(response);

  You forgot to handle a possible value here, for example: 
Maybe
```

# 警告字段

如果我们想要强制此警告抛出错误，我们可以注意到前面片段中的错误编号，并将`bsconfig.json`的`warnings`字段更改为以下内容：

```js
"warnings": {
  "error": "+101+8" // added "+8"
},
```

要将所有警告转换为错误，请使用以下代码：

```js
"warnings": {
  "error": "A"
},
```

有关警告编号的完整列表，请查看[`caml.inria.fr/pub/docs/manual-ocaml/comp.html#sec281`](https://caml.inria.fr/pub/docs/manual-ocaml/comp.html#sec281)（向下滚动一点）。

# 包规范字段

`package-specs`字段包含两个字段：`module`和`in-source`。

`module`字段控制 JavaScript 模块格式。默认值为`commonjs`，其他可用选项包括`amdjs`、`amdjs-global`、`es6`和`es6-global`。`-global`部分告诉 BuckleScript 将`node_modules`解析为浏览器的相对路径。

`in-source`字段控制生成的 JavaScript 文件的目标；`true`会导致生成的文件放在源文件旁边，`false`会导致生成的文件放在`lib/js`中。将`in-source`设置为`false`对于在现有 JavaScript 项目中使用 Reason 非常有用，这样就可以在不进行更改的情况下使用现有的构建流程。

让我们暂时使用`"es6"`模块格式，并将编译后的资产放在`lib/js`中：

```js
"package-specs": {
  "module": "es6",
  "in-source": false
},
```

# 后缀字段

`suffix`字段配置生成的 JavaScript 文件的扩展名。通常最好保留`".bs.js"`后缀，因为这有助于`bsb`更好地跟踪生成的工件。

# 来源字段

BuckleScript 知道要查找`src`目录，是因为以下配置：

```js
"sources": {
  "dir" : "src",
  "subdirs" : true
},
```

如果`subdirs`为`false`，则`src`子目录中的任何`.re`和`.ml`文件都不会被编译。

有关`bsconfig.json`的更多信息，请参阅 BuckleScript 文档的以下部分：[`bucklescript.github.io/docs/build-configuration`](https://bucklescript.github.io/docs/build-configuration)。

# 使用 DOM

在跳入 ReasonReact 之前，让我们尝试在纯 Reason 中使用 DOM。我们将编写一个模块，执行以下操作：

+   创建一个 DOM 元素

+   设置该元素的`innerText`

+   将该元素附加到文档的主体

在项目的根目录中创建一个`index.html`文件，内容如下：

```js
<html>
  <head></head>
  <body>
    <!-- if "in-source": false -->
    <script type="module" src="img/Demo.bs.js"></script>

    <!-- if "in-source": true -->
    <!-- <script type="module" src="img/Demo.bs.js"></script> -->
  </body>
</html>
```

注意`script`标签上的`type="module"`属性。如果所有模块依赖项都符合**ES Module**（**ESM**）规范，并且它们都可以在浏览器内使用，那么您就不需要模块捆绑器来开始（假设您使用支持 ES 模块的浏览器）。

在`Greeting.re`中，添加以下问候函数：

```js
let greeting = name => {j|hello $name|j};
```

在`Demo.re`中，添加以下代码：

```js
[@bs.val] [@bs.scope "document"]
external createElement : string => Dom.element = "";

[@bs.set] external setInnerText : (Dom.element, string) => unit = "innerText";

[@bs.val] [@bs.scope "document.body"]
external appendChild : Dom.element => Dom.element = "";

let div = createElement("div");
setInnerText(div, Greeting.greeting("world"));
appendChild(div);
```

使用 BuckleScript 强大的互操作功能（我们将在第四章中深入探讨），上述代码绑定到现有的浏览器 API，即`document.createElement`、`innerText`和`document.body.appendChild`，然后使用这些绑定创建一个带有一些文本的`div`，并将其附加到文档的主体。

运行`npm run build`，启动服务器（也许可以在新的控制台选项卡中使用`php -S localhost:3000`），然后导航到`http://localhost:3000`，以查看我们新创建的 DOM 元素：

![](img/7b54adea-c4aa-49c7-8222-63c1556608c3.png)

重点是以这种方式使用 DOM 真的很繁琐。由于 JavaScript 的动态特性，很难输入 DOM API。例如，`Element.innerText`根据使用方式用于获取和设置元素的`innerText`，因此会导致两种不同的类型签名：

```js
[@bs.get] external getInnerText: Dom.element => string = "innerText";
[@bs.set] external setInnerText : (Dom.element, string) => unit = "innerText";
```

幸运的是，我们有 React，它在很大程度上为我们抽象了 DOM。使用 React，我们不需要担心输入 DOM API。当我们想要与各种浏览器 API 交互时，很高兴知道 BuckleScript 有我们需要完成工作的工具。虽然在纯 Reason 中编写前端 Web 应用程序是完全可能的，但使用 ReasonReact 时体验会更加愉快，特别是在初次使用 Reason 时。

# 设置 ReasonReact 项目

要创建一个新的 ReasonReact 项目，请运行以下命令：

```js
bsb -init my-reason-react-app -theme react
cd my-reason-react-app

```

打开文本编辑器后，我们看到有一些变化。`package.json`文件列出了相关的 React 和 webpack 依赖项。让我们安装它们：

```js
npm install
```

我们还有以下与 webpack 相关的 npm 脚本：

```js
"webpack": "webpack -w",
"webpack:production": "NODE_ENV=production webpack"
```

在`bsconfig.json`中，我们有一个新字段，用于为 ReasonReact 启用 JSX：

```js
"reason": {
  "react-jsx": 2
},
```

我们有一个简单的`webpack.config.js`文件：

```js
const path = require("path");
const outputDir = path.join(__dirname, "build/");

const isProd = process.env.NODE_ENV === "production";

module.exports = {
  entry: "./src/Index.bs.js",
  mode: isProd ? "production" : "development",
  output: {
    path: outputDir,
    publicPath: outputDir,
    filename: "Index.js"
  }
};
```

请注意，配置的入口点是`"./src/Index.bs.js"`，这是有道理的，因为在`bsconfig.json`中默认情况下`"in-source"`设置为`true`。其余部分都是正常的 webpack 内容。

要运行这个项目，我们需要同时运行`bsb`和`webpack`：

```js
npm start

/* in another shell */
npm run webpack

/* in another shell */
php -S localhost:3000
```

由于`index.html`文件位于`src`目录中，我们访问`http://localhost:3000/src`来查看默认应用程序。

# 改善开发者体验

现在我们已经了解了工具链在基本层面上的工作原理，让我们改善开发者体验，以便我们可以用一个命令启动我们的项目。我们需要安装一些依赖项，如下所示：

```js
npm install webpack-dev-server --save-dev
npm install npm-run-all --save-dev
```

现在，我们可以更新我们的 npm 脚本：

```js
"scripts": {
  "start": "npm-run-all --parallel start:*",
  "start:bsb": "bsb -clean-world -make-world -w",
  "start:webpack": "webpack-dev-server --port 3000",
  "build": "npm-run-all build:*",
  "build:bsb": "bsb -clean-world -make-world",
  "build:webpack": "NODE_ENV=production webpack",
  "test": "echo \"Error: no test specified\" && exit 1"
},
```

接下来，为了让`webpack-dev-server`在`http://localhost:3000`上提供`index.html`文件，而不是`http://localhost:3000/src`，我们需要安装并配置`HtmlWebpackPlugin`：

```js
npm install html-webpack-plugin --save-dev
```

我们可以在`src/index.html`中删除默认的`<script src="img/Index.js"></script>`标签，因为`HTMLWebpackPlugin`会自动插入脚本标签。

我们还删除了`publicPath`设置，以便使用`"/"`的默认路径：

```js
const path = require("path");
const HtmlWebpackPlugin = require("html-webpack-plugin");

const isProd = process.env.NODE_ENV === "production";

module.exports = {
  entry: "./src/Index.bs.js",
  mode: isProd ? "production" : "development",
  output: {
    path: path.join(__dirname, "build/"),
    filename: "Index.js"
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: "./src/index.html"
    })
  ]
};
```

现在，我们运行`npm start`并访问`http://localhost:3000`，看到相同的 ReasonReact 应用程序正在运行。

# 总结

在本章中，我们看到了使用 Reason 开始的简单程度。在第三章 *创建 ReasonReact 组件*中，我们将开始构建一个 ReasonReact 应用程序，这个应用程序将贯穿本书。这个应用程序将帮助我们在学习更多关于 Reason 语义、BuckleScript 互操作性和 ReasonReact 特定内容时提供上下文。

如果您还不理解这些生成的项目中的所有内容，请不要担心。在第三章 *创建 ReasonReact 组件*结束时，您会感到更加舒适。但是，如果您在学习过程中有问题，请随时在 Reason 的 Discord 频道上寻求实时帮助：[`discord.gg/reasonml`](https://discord.gg/reasonml)。

我希望您会像我一样觉得 Reason 社区是如此的友好和乐于助人。
