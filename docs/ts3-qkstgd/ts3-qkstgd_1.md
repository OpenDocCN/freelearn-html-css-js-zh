# 第一章：TypeScript 入门

在本章中，我们将探讨 TypeScript 并了解如何在新的项目或现有的 JavaScript 项目中使用 TypeScript。我们将看到如何适应你当前的构建环境。TypeScript 可以由各种构建工具支持，例如 Grunt、Gulp、webpack 或简单地通过使用 **命令行界面**（**CLI**）。我们还探讨了在所有可用的选项中，开始使用 TypeScript 的最佳选项。

在配置任何构建工具之前，重要的是要理解它们都使用相同的 TypeScript 编译器，通常被称为转换器。TypeScript 编译器可以通过 npm 使用：

```js
npm install -g typescript
```

Npm 可能默认没有安装在你的电脑上。如果是这种情况，之前的指令将会失败。这意味着你需要安装 Node.js。你可以通过访问官方网站 [`nodejs.org/`](https://nodejs.org/) 来安装 Node.js。

在任何时候，你都可以使用以下命令来验证你是否已经安装了 Node.js、npm 和 TypeScript：

```js
node -v
npm -v
tsc -v
```

TSC 是 TypeScript 编译器的可执行文件。这是所有构建工具都使用的文件。Grunt、Gulp 和 webpack 通过它们各自的插件基础设施使用 TSC，这些基础设施将 TSC 的功能映射到它们自己的平台。请注意，最近的 TSC 功能可能需要几周时间才能到达这些平台。这可能是使用这三个构建系统时编译器选项差异的原因。相比之下，使用 TSC CLI 确保你直接使用 TypeScript。

本章涵盖了以下内容：

+   Grunt

+   Gulp

+   Webpack

+   NPM/CLI

+   TypeScript 编译器

# Grunt

Grunt 是一个 JavaScript 任务运行器。它可以通过 NPM 安装，NPM 用来列出所有它的插件：

```js
npm install -g grunt-cli
```

在新项目的情况下，请确保你的 TypeScript 项目的根目录中存在 `package.json` 文件。你可以通过使用 `npm init` 命令来生成一个简单的 `package.json`。

完成后，你可以在你的项目中安装 Grunt：

```js
npm install grunt --save-dev
```

一旦 Grunt 在你的机器上可用并在你的项目中指定，你需要获取一个 TypeScript 插件。Grunt 有两个插件名为 `grunt -TypeScript` 和 `grunt-TS`。前者已经有一段时间没有维护，并且缺少最新的 TypeScript 编译器配置。我强烈建议使用后者：

```js
npm install grunt-ts --save-dev
```

最后一个包应该被安装为一个开发依赖，以便 Grunt 能够编译 TypeScript 并本地安装它。Grunt 将会在本地搜索这个包。省略 TypeScript 作为本地依赖将会在执行 Grunt 时导致以下错误。

ENOENT：找不到文件或目录，打开 `'/.../node_modules/grunt-ts/node_modules/typescript/package.json'` 使用 `--force` 继续操作。

将 TypeScript 本地作为 `dev` 依赖项安装很简单：

```js
npm install typescript --save-dev
```

在 `grunt-ts` 版本 6 之前，TypeScript 和 Grunt 在 `grunt-ts` 的安装过程中一起安装。现在不再是这样了，所以它们必须手动添加。

下一步是配置 Grunt 以使用 TypeScript 插件。如果您没有使用 Grunt，您需要在项目的根目录下创建一个 `Gruntfile.js` 文件。否则，您可以编辑现有的一个。该插件允许您在 `Gruntfile.js` 中指定许多 TypeScript 选项，但一个好的做法是直接在文件中限制 TypeScript 选项，并利用 TypeScript 配置文件。通过在 Grunt 之外配置 TypeScript，这为您提供了在不重复或更改 TypeScript 首选项的情况下，编译代码而不使用 Grunt 或迁移到另一个构建工具的可能性。

一个仅用于将 TypeScript 编译成 JavaScript 的极简 Grunt 配置可能看起来像以下这样：

```js
module.exports = function(grunt) {
 grunt.initConfig({
   ts: {
    default : {
     tsconfig: './tsconfig.json'
    }
   }
 });
 grunt.loadNpmTasks("grunt-ts");
 grunt.registerTask("default", ["ts"]);
};
```

Grunt 配置创建了一个默认任务，该任务执行一个自定义的 `ts` 任务，该任务链接到 `tsconfig.json` 文件，这是默认的 TypeScript 配置文件。

`tsconfig.json` 文件可能看起来像以下这样，它将编译所有扩展名为 `.ts` 的 TypeScript 文件，并将结果输出到 `build` 文件夹：

```js
{
 "compilerOptions": {
   "rootDir": "src",
   "outDir": "build",
 }
}
```

当使用 `grunt` 和 `grunt-ts` 时，您必须确保 `tsconfig.json` 文件中的 JSON 是有效的，且没有尾随逗号。否则，您可能会遇到以下错误：

```js
tsconfig error: "Error parsing \"./tsconfig.json\".  It may not be valid JSON in UTF-8."
```

要测试配置，在项目根目录下的 `src` 文件夹中创建一个简单的 `index.ts` 文件。您可以输入 `console.log('test')`。之后，在项目根目录的命令行中运行 `grunt`。这将创建一个包含相同代码行的 `index.js` 文件的 `build` 文件夹。它还将创建一个 `js.map` 文件，让您可以直接在浏览器中调试 TypeScript 代码。

如果出于某种原因，您不想依赖 `tsconfig.json`，您可以直接在 `Gruntfile.js` 文件中指定源和目标：

```js
module.exports = function (grunt) {
 grunt.initConfig({
  ts: {
   default: {
    src: ["src/**/*.ts"],
     outDir: "build",
     options: {
     rootDir: "src"
    }
   }
  }
 });
 grunt.loadNpmTasks("grunt-ts");
 grunt.registerTask("default", ["ts"]);
};
```

最后，`grunt-ts` 包装了 TypeScript 命令行。它提供了如 *快速编译* 等选项，只编译自上次编译以来发生变化的文件。如果您已经在项目中使用 Grunt，并希望开始使用 TypeScript 而不修改您的构建过程，这也是一个有趣的选项。

# Gulp

Gulp 是一个具有 TypeScript 插件的自动化工具包。NPM 中有两个插件可用，分别是 `gulp-tsb` 和 `gulp-typescript`。后者是最受欢迎且维护得更好的。您可以使用以下命令获取 `gulp` 和插件：

```js
npm install -g gulp
npm install --save-dev gulp-typescript
```

如果您没有 Gulp 配置文件，您需要在 `gulpfile.js` 项目的根目录下创建一个。

没有明确选项的配置将依赖于默认配置。这意味着配置 Gulp 可以像将源代码管道输入 TypeScript 插件，然后将结果管道输入目标文件夹一样简单，该文件夹将放置 `build` 文件，即 JavaScript 文件，以便消费。一旦将以下代码放置在 `gulpfile.js` 中，您就可以通过在命令行中使用 `gulp` 来执行它。这将自动执行一次 *默认任务*：

```js
var gulp = require("gulp");
var ts = require("gulp-typescript");

gulp.task("default", function () {
 var tsResult = gulp.src("src/**/*.ts")
 .pipe(ts());
 return tsResult.js.pipe(gulp.dest("build"));
});
```

在 Gulp 中有一个任务可以增量构建更改的 TypeScript 文件，而不是构建所有文件。这对于大型项目来说可能很有用，可以减少编辑和访问结果之间的时间。这与 Grunt 的 *快速编译* 类似。为了实现持续编译，您必须创建一个新的 Gulp 任务。在这个例子中，我们将把 Gulp 改为依赖于 `tsconfig.json` 文件，这将允许我们将 TypeScript 编译器选项与 Gulp 配置分开：

```js
var gulp = require('gulp');
var ts = require('gulp-typescript');
var tsProject = ts.createProject('tsconfig.json');

gulp.task('scripts', function() {
 return gulp.src('src/**/*.ts')
 .pipe(tsProject())
 .pipe(gulp.dest('build'));
});
gulp.task('watch', ['scripts'], function() {
 gulp.watch('src/**/*.ts', ['scripts']);
});
```

要运行 `watch` 任务，您需要先执行 Gulp，然后跟随着任务的名称：`gulp watch`。与 Grunt 不同，Gulp 不会生成 `map` 文件。它需要一个额外的 Gulp 插件。因为源映射对于拥有高效的调试环境至关重要，所以强烈建议下载 `gulp-sourcemap` 包，并将之前的配置更改为以下内容。但首先，让我们下载 `gulp-sourcemaps` 包：

```js
npm install --save-dev gulp-sourcemaps
```

然后创建一个新的任务：

```js
var sourcemaps = require('gulp-sourcemaps');
gulp.task('scriptswithsourcemap', function () {
 return gulp.src('src/**/*.ts')
 .pipe(sourcemaps.init())
 .pipe(tsProject())
 .pipe(sourcemaps.write('.', { includeContent: false, sourceRoot: '.'}))
 .pipe(gulp.dest('build'));
});
```

配置将在与生成的 JavaScript 文件同名但扩展名不同的文件中创建源映射。扩展名将是 `.jsmap`。如果您想在 JavaScript 文件中直接包含映射，可以移除在 `write` 函数中传入的两个参数。我建议有一个单独的脚本任务，在文件中生成源映射，以将映射与生成的代码分开，并始终创建源映射。这是编译过程中的小代价，但在调试中能带来巨大的收益。

# Webpack

Webpack 是当与 JavaScript 和 Web 开发一起工作时自动化工作流程最常用的方式之一。其主要目的是捆绑，但它可以执行许多连续步骤，例如编译 TypeScript。类似于 Grunt 和 Gulp，webpack 有两个用于 TypeScript 的加载器（类似于插件）。一个是`ts-loader`，另一个是`awesome-typescript-loader`。在 Grunt 和 Gulp 中，用户更喜欢哪一个很明确，但在 Webpack 中并非如此。这两个加载器在受欢迎程度方面相似。如果需要，它们之间切换也不困难。最初，`awesome-typescript-loader`比`ts-loader`更快，但随着 TypeScript 的发展，差异通常很小。此外，有时一个或另一个的高级功能可能会出现问题，因此能够根据您的项目进行切换是实用的。我将介绍`ts-loader`，它稍微更受欢迎，仍然积极维护，并且比`awesome-typescript-loader`有更多的使用。

如果您尚未使用`webpack`，我们需要安装它：

```js
npm install --save-dev webpack
npm install --save-dev webpack-cli
```

一旦安装了 webpack，您就可以安装 TypeScript 加载器：

```js
npm install --save-dev ts-loader
```

一旦所有工具都安装完毕，您就可以配置 webpack 以捆绑由`webpack`加载器产生的 JavaScript。然而，在项目的根目录中需要`webpack.config.js`文件。像任何 Webpack 配置一样，入口属性必须被定义。确保您正在引用 TypeScript 文件。输出也在输出属性中指定。Webpack 需要提及要分析的扩展名。在 TypeScript 的情况下是`.ts`，但如果您使用 React 工作，您可能还希望在`resolve:extensions`下添加`.tsx`。最后，在`module:rules`下指定`ts-loader`。再次强调，需要 TypeScript 的扩展名和加载器的名称：

```js
module.exports = {
 mode: "development",
 devtool: "source-map",
 entry: "./src/index.ts",
 output: {
  path: __dirname + "/build",
  filename: "bundle.js"
 },
 resolve: {
  extensions: [".ts"]
 },
 module: {
  rules: [
  { test: /\.ts$/, loader: "ts-loader" }
  ]
 }
};
```

您可以通过访问二进制文件来运行 webpack 命令行（`cli`），这将读取`webpack.config.js`文件：

```js
node node-modules/webpack-cli/bin/cli.js
```

如果您想避免引用`node_modules`，您可以在全局空间中安装`webpack-cli`，使用`npm install -g webpack-cli`。

关于 webpack 的一些小细节。与仅验证 TypeScript 相比，webpack 有一个额外的模块，它将包的生产和编译分开。当您的项目开始增长并且希望拥有更快的编译速度时，这些模块可能很有趣。您可以随意检查`fork-ts-checker-webpack-plugin`和`thread-loader`。在深入研究其他库之前，`ts-loader`有一种增量构建的方法，并使用 TypeScript 的 watch API 来避免总是构建一切。这将提高每次编译的性能。要允许监视，将`ts-loader`的规则更改为以下内容：

```js
rules: [
 {
 test: /\.ts$/,
 use: [
  {
   loader: 'ts-loader',
   options: {
    transpileOnly: true,
    experimentalWatchApi: true,
   },
  },
 ],
 }
]
```

关于 webpack 的最后一个细节是它依赖于`tsconfig.json`来进行所有 TypeScript 相关配置。Grunt 和 Gulp 允许您在它们的工具配置中覆盖配置，而 webpack 则不是这样。在打包时，webpack 会生成`bundle.js.map`，但前提是开发工具指定了配置。然而，您必须将`tsconfig.json`的“sourceMap”设置为`true`，以便生成与 TypeScript 兼容的映射。

# NPM/CLI

几乎所有 Web 项目都使用 NPM。NPM 是我们用来获取 TypeScript 的机制。它会在项目根目录创建`package.json`，并可以直接用来启动 TypeScript。这是因为 TypeScript 有一个名为**tsc**（**TypeScript 编译器**）的 CLI。

NPM 配置中有一个名为`scripts`的部分，您可以在其中添加任何想要的命令。您可以创建一个`build`命令，该命令调用 tsc。如果没有任何参数，tsc 将使用项目根目录下的`tsconfig.json`。在下面的代码片段中，定义了`build`脚本。要运行该命令，需要使用 NPM 的`run`命令，即`npm run build`：

```js
"scripts": {
"build": "node_modules/typescript/bin/tsc"
},
```

使用指定了源映射、`rootDir`和`outDir`的 TypeScript 配置文件，结果将与 Gulp 和 Grunt 相同（与 webpack 不同，因为它不会进行打包）：

```js
{
 "compilerOptions": {
 "rootDir": "src",
 "outDir": "build",
 "sourceMap": true
 }
}
```

这种配置通常不是首选，因为它过于简单且有限。然而，可以使用双与号(`&&`)创建一系列命令，一个接一个地执行。这个选项速度快，不需要依赖 NPM 库，通常足以在基本级别上开始。

NPM 和 CLI 方法的优势在于 TypeScript 可以轻松执行。因此，如果您有一个自定义的构建系统，您可以轻松通过调用 CLI 来集成 TypeScript。

# TypeScript 编译器

考虑到所有可用的工具都可以将 TypeScript 编译成 JavaScript，一个核心概念保持不变：您必须知道使用哪种配置。既然您不负责配置编译器，您可以跳过这一节——配置 TypeScript 是您很少做的事情，而且当它按预期工作后，可以长时间保持不变。然而，为了了解 TypeScript 的能力，您需要了解一些核心选项。在本节中，我们将看到您可以为项目启用和自定义的主要设置。

# 文件位置

这一节主要关于文件系统中的文件配置。它指导 TypeScript 在您的机器上查找不同的文件位置，以及在哪里生成 JavaScript 文件。

# rootDir 和 outDir

对于您的项目，您需要设置的最基本的配置是告诉 TypeScript 从哪里获取 TypeScript 文件以及将编译结果发布到何处。TypeScript (`.ts`) 文件和 JavaScript (`.js`) 文件将生产在哪里。这是通过指定 `rootDir` 和 `outDir` 来完成的。避免使用 `rootDir` 可能会在 `outDir` 中给您带来惊喜。默认情况下，TypeScript 会计算它应该是什么，并尝试找到一个公共路径，即所有输入文件的最长公共前缀。这有一个缺点，即当文件结构发生变化时，它可能是不一致的。最近，TypeScript 改变了其行为，默认为 `.`，这缓解了问题。尽管如此，具有显式配置是最佳实践，以避免对哪种版本的 TypeScript 应用了此新规则的混淆。

示例：

```js
rootDir:src
outDir:build
```

# baseUrl 和 paths

当 baseUrl 和 paths 一起使用时，可能会产生混淆。baseUrl 允许使用非相对名称进行解析。Paths 与 baseUrl 密切相关，是一个键值映射，允许使用名称作为链接到库的特定路径，使用 baseUrl 作为根。

这里有一个示例：

```js
{
 "compilerOptions": {
 "baseUrl": ".", // This must be specified if "paths" is.
 "paths": {
 "jquery": ["node_modules/jquery/dist/jquery"] // This mapping is relative to "baseUrl"
 }
 }
}
```

在代码中：

```js
Import * from "jquery"
```

Paths 也可以用于更高级的场景，其中您可以定义后备文件夹。作为良好的实践，我建议尽可能使用相对路径，并避免复杂的结构和潜在的解析问题。

# sourceRoot 和 sourceMap 以及 mapRoot

sourceMap 属性是一个布尔值，当设置为 `true` 时，将生成生成的 JavaScript 和 TypeScript 之间的映射。如果您在浏览器中调试并希望进入 TypeScript 代码而不是进入生成的代码，这是一个很好的选项。这简化了调试，因为您正在工作的区域完全相同。这通常是打开的。

然而，在正常情况下，sourceRoot 很少被使用。如果你将 sourceMap 移动到其他位置以在运行时指示其位置，则它可用。这将改变生成的 sourceMap 路径。以下代码展示了注释，说明了映射文件的路径。SourceRoot 会改变 `index.js.map` 前的部分：

```js
const text = "Text for test1";
console.log(text);
//# sourceMappingURL=index.js.map
```

类似地，mapRoot 允许在映射文件位于 JavaScript 文件不同位置时更改源。sourceRoot 和 mapRoot 的区别在于这次我们更改的是映射文件而不是 JavaScript 文件。在以下映射文件代码的部分提取中，我们看到可以由 mapRoot 修改的路径：

```js
{"version":3,"file":"index.js","sourceRoot":"","sources":["../src/index.ts"]......
```

使用或配置的时机取决于你如何配置你的构建文件。如果你将映射移动到其他地方，那么`sourceRoot`就很有趣。然而，如果你保留映射但将 JavaScript 移动到其他地方，你可能需要更改`mapRoot`。我提到这些配置的唯一原因是你可能已经有一个你想要迁移到 TypeScript 的 JavaScript 项目。根据你的现有配置，你可能需要调整这些配置。然而，对于任何标准项目，都不应该对这些配置进行修改。

# 文件和包含与排除

包含是一个数组，指定了 glob 模式，该模式指定了哪些文件/文件夹要包含在编译中。数组`exclude`补充了`include`，并移除了要编译的文件。当这两个属性都被指定时，`exclude`将从`include`中过滤掉包含的文件。默认情况下，`include`包括`rootDir`下的所有 TypeScript 文件，因此不需要添加`**/*.ts`的条目。

文件很少使用，因为它不如`include`灵活。它允许通过路径和名称指定要编译的文件，而不是使用 glob 模式。模式方法更灵活，允许你一次性配置，而不是通过在文件中添加和删除条目来不断修改配置文件。

这里有一个例子：

```js
"include": [
 "src/**/*"
 ],
 "exclude": [
 "node_modules",
 "dont/compile/*.mock.ts"
 ]
```

关于这三个文件配置的最后一句话是，与大多数选项相反，这些配置不是位于`compilerOptions`下，而是直接设置在`tsconfig.json`文件的根目录。

# 输出文件

输出文件是一个有用的选项，如果你需要从多个 TypeScript 文件生成单个 JavaScript 文件。当使用输出文件时，你可以移除`outDir`并设置一个相对于项目根目录的路径，然后跟上生成文件的名称和扩展名：

```js
{
 "compilerOptions": {
 "rootDir": "src",
 "outDir": "build",
 "target": "es6",
 "sourceMap": true,
 "outFile": "build/mySingleFile.js"
 }
}
```

上面的示例代码创建了一个单个文件，但也创建了`sourceMap`文件，这是因为配置了`sourceMap`。

# 类型

本节包含有关 TypeScript 类型的说明。第一个配置为 TypeScript 提供了查找类型的提示，同时也告诉 TypeScript 在编译时是否必须生成定义文件。

# `typeRoots`和`types`

默认情况下，`node_modules`库中提供的每个类型，包括所有特定的`@types/`和直接在库文件夹内的`.d.ts`包，都会被 TypeScript 编译器读取。然而，在某些没有定义文件可用且你需要提供一个自定义文件的场景中，你需要指定定义文件的位置。这可以通过指定一个文件夹来完成，在该文件夹中你可以定义所有你的定义文件。需要注意的是，如果你想让 TypeScript 继续在`node_modules`中查找定义文件，你需要指定`node_modules`。

```js
{
 "compilerOptions": {
 "typeRoots" : ["./typings", “./node_modules”]
 }
}
```

Types 配置允许 TypeScript 选择包含哪些文件。它与 typeRoots 协同工作，是一个数组。它将类型名称列入白名单。

# 声明和声明目录

如果你正在构建一个库而不是网站或程序，那么提供定义文件与生成的 JavaScript 文件一起可能是明智的。原因是当在 TypeScript 中构建库时，我们从不共享实际的 TypeScript（`.ts`）文件，而是共享 JavaScript 文件。其理由是 TypeScript 只是 JavaScript 的超集，我们希望将我们的代码暴露给尽可能广泛的受众。通过提供 JavaScript 文件，我们允许每个 JavaScript 开发者消费我们的工作。然而，TypeScript 开发者可以休息。为了解决这个问题，我们可以提供一个包含所有签名函数以及导出变量的定义文件（`.d.ts`）。TypeScript 编译器允许你通过使用 `declaration` 自动生成定义文件：

该选项是 `boolean`：

```js
{
 "compilerOptions": {
 "declaration" : true
 }
}
```

默认情况下，生成的声明文件是 TypeScript 文件，并且位于与 TypeScript 文件相同的目录。这意味着对于每个 JavaScript（`.js`）文件，你都会看到紧挨着它的兄弟声明文件（`.d.ts`）：

```js
{
 "compilerOptions": {
 "declaration" : true,
 "declarationDir": "definitionfiles/here"
 }
}
```

`declarationDir` 有一个需要注意的问题，那就是它不能与 `outFile` 一起使用。你将得到一个编译错误，指出这两个选项不能同时定义：

```js
error TS5053: Option 'declarationDir' cannot be specified with option 'outFile'.
```

# 配置文件

TypeScript 有自己的配置文件，这是一种方便的方式，可以避免通过命令行参数传递每个选项。该文件位于项目的根目录。一种可能性是拥有几个配置文件，可以在不同情况下使用。这是通过向 `tsc` 提供带有选项 `-p` 后跟配置文件名的选项来实现的。以下三个命令行调用显示了一个没有任何参数的例子，它实际上与第二行执行相同的编译。尽管如此，第三条编译指令是不同的，指向一组完全不同的选项：

```js
tsc
tsc -p tsconfig.json
tsc -p tsconfig.test.json
```

配置文件的一个好处是可以通过扩展配置来实现可重用性。你可以将这个原则比作面向对象中的继承——一个文件可以继承另一个文件。这可以通过使用 `extends` 属性作为键，以及要继承的文件作为值来实现。提供的文件必须相对于项目的根目录。它可以有或没有扩展名（`.json`）：

```js
{
 "extends": "./tsconfig.json",
 "compilerOptions": {
 "outDir": "buildtest",
 "sourceMap": false,
 "declaration": false
 }
}
```

以下示例显示了一个命令，该命令使用 `tsconfig.test.json` 中的选项进行编译，并包含一组小指令：

```js
tsc -p tsconfig.test.json
```

第一个是要扩展的文件。在这种情况下，也可以是 `./tsconfig` 而不带扩展名。该文件覆盖了 `tsconfig.json` 文件中提供的 `outDir`，并添加了额外的值。一个好的模式是有一个 `base` 配置，其中包含你已知将在许多配置中共享的配置。

# 模块和模块解析

JavaScript 中代码分离的一个关键概念是模块。模块引入了导入和导出代码的概念。通过指定特定的名称和代码的哪一部分可以导出，这种能力增加了共享代码的能力。然后，其他软件可以导入代码并利用其功能。然而，并没有一种方法可以构建模块。TypeScript 允许你以单一的方式编写代码，并在编译期间生成一个尊重不同流行模块语法的输出。以下是 TypeScript 可以解释的模块列表：

```js
"None", "CommonJS", "AMD", "System", "UMD", "ES6", "ES2015" or "ESNext".
```

`module` 选项可以看作是 TypeScript 如何生成模块，而 `moduleResolution` 则是它如何读取模块。TypeScript 可以理解 `import` 语句的两种方式：类和节点。前者是传统的 TypeScript 方法，它有不同的规则来查找导入的文件。更受欢迎的选择是 `node`。

无论模块解析选项如何，你应该尝试通过在导入中指定从导入文件开始的路径来使用相对解析。相对导入通过路径以单个点或双点开头来表示，以向后移动。以下是一些相对导入的例子：

```js
import x from "./sameFolder";
import y from "../parent/folder";
import z from "../../../deeper/";
```

使用它的原因在于代码导入的清晰性。使用绝对解析会带来混淆，因为它依赖于其他配置，如 `baseUrl` 以及 `moduleResolution`。

相反，依赖于更复杂解析的导入是非相对的，其形式如下：

```js
import a from "module123";
```

不深入复杂的规则，经典示例中的最后一个例子会寻找与导入文件相邻的模块，并沿着文件夹结构向下查找，而不会尝试从 `node_modules` 中导入模块。然而，如果 `moduleResolution` 设置为 `node`，则第一个检查将是查找 `node_modules` 文件夹中的 `module123`。正如你所看到的，如果你使用一个常见的名称，你可能会加载一个意外的模块。

# ECMAScript

本节包含与生成的 ECMAScript 类型相关的 TypeScript 配置，以及可以合并的附加包。

# 目标

必须指定目标选项，但很少更改。此选项指示 TypeScript 生成哪个版本的 JavaScript 文件。默认情况下，它生成 ECMAScript 3 版本，该版本没有 TypeScript 允许的所有内置功能。然而，TypeScript 仍然可以通过生成模拟这些功能的 JavaScript 代码来生成这样的旧版本 ECMAScript。这伴随着运行时性能惩罚的代价，但这是在较旧的浏览器上构建现代代码的绝佳方式。以下是你可以指定的实际目标：

```js
"ES3", "ES5", "ES6"/"ES2015", "ES2016", "ES2017" or "ESNext"
```

如果你通常在网络上部署，ES5 是一个安全的选择，因为所有浏览器都对其提供 100% 的支持。但是，ES6 非常接近，Chrome 支持 98% 的其功能，Firefox 97%，Edge 96%。

# Lib

TypeScript 可以将 ECMAScript 的核心库注入到生成的代码中。默认情况下，一些库会自动添加。例如，如果您指定目标为 ES5，TypeScript 会添加库：DOM、ES5 和 ScriptHost。您可以手动添加额外的库。例如，如果您想使用可迭代对象，可以在 lib 数组中添加字符串 `ES2015.Iterable`。您还可以使用超出您主要目标的功能。例如，您可以将目标设置为 ES2015 并使用 ES2018 的功能。将 "target" 视为编译中要包含的主要功能集，将 `lib` 视为可以添加到编译中的附加功能的子集。

# 编译器严格性

TypeScript 在如何严格分析您的代码方面有许多选项。本节向您展示了每个设置的差异，如果您来自现有的 JavaScript，允许您逐步开始。

# 严格

这是将每个配置转换为严格的选项。如果您从 JavaScript 开始一个新的 TypeScript 项目，这是您应该使用的。

# StrictFunctionTypes

这是一个高级检查，不允许函数参数的二向变异性。它使用逆变异性。这意味着如果函数期望类型 A 作为参数，您不能设置一个具有类型 B 的函数，该类型继承自类型 A - 您必须只传递类型 A。以下在 `StrictFunctionTypes` 设置为 `true` 的情况下无法编译。严格的选项有助于避免传递比预期类型有更多成员的对象。以下示例中 `B` 是 `firstName` 字段，并继承自 `A`，因此得名：

```js
interface A {
 name: string;
}

interface B extends A {
 firstName: string;
}

declare let f1: (x: A) => void;
declare let f2: (x: B) => void;

f1 = f2; // DOESNT COMPILE
f2 = f1;
```

在编译过程中，TypeScript 发现参数被传递给具有更多成员的对象，因此无法编译：

```js
Error message :  Type 'A' is not assignable to type 'B'. Property 'firstName' is missing in type 'A'.
```

# StrictPropertyInitialization 和 StricNullChecks

`StrictPropertyInitialization` 属性应始终设置为 true。这确保了类的所有属性都在声明级别或类的构造函数中与直接关联初始化。

```js
class A {
 public field1: number;
}
```

示例无法编译，因为 `Field1` 是一个未定义的数字。该字段的值是未定义的。有许多解决方案可以保持严格性并使代码符合规范。第一个解决方案是在初始化时设置值：

```js
class A {
 public field1: number = 1;
}
```

在初始化时设置默认值并不总是可能的。在某些情况下，可以在构造类型中指定值：

```js
class A {
 public field1: number;
 constructor(p:number){
   This.field1 = p;
 }
}
```

编译代码的第三种方法是使用成员名称之后的感叹号运算符（`!`）。该运算符指示 TypeScript 该值将在以后提供。延迟初始化的场景通常是通过某些注入框架或使用函数来初始化类。

使用 `strictPropertyInitialization` 执行此任务的注意事项之一是依赖于另一个必须启用的严格属性——`strictNullChecks`。空值检查也应该始终设置为 true。如果没有这个设置，被识别为类型的字段将自动接受 null 和 undefined 作为有效的类型。仅支持具有显式类型的字段，并在 null 或/和 undefined 的情况下使用本书后面将看到的属性定义，这样更不容易混淆，也更具有声明性。

# 摘要

在本章中，我们设置了不同的配置，使您能够以直接的方式开始使用 TypeScript 进行编码。在设置您喜欢的开发环境后，我们简要介绍了最重要的编译器选项，以帮助您走上正确的道路。TypeScript 是一个灵活的编译器，由于设置的选择方式，您应该能够快速掌握开发技能。

在下一章中，我们将通过介绍如何将 ECMAScript 原始类型强类型化来探讨使用 TypeScript 进行编程。
