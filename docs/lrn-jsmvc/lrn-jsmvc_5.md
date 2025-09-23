# 第五章. StealJS

**StealJS**是一个独立的代码管理和打包工具，它允许我们将 JavaScript 和其他文件类型加载到应用程序中，连接多个 JavaScript 或 CSS 文件，并压缩其内容。StealJS 提供跨浏览器消息日志、代码生成器和简单的包管理工具。

在本章中，我们将介绍 StealJS 的所有功能。

### 注意

StealJS 需要 Java 1.6 或更高版本。

# 依赖管理

**依赖管理**是一个工具，它提供了一种有组织的方式来管理软件组件协同工作作为一个单一系统。

以下是一些 StealJS 的关键特性：

+   只加载单个文件一次

+   从不同领域加载文件

+   加载 JavaScript 和 CoffeeScript

+   加载 CSS less

我们在前面章节中多次使用了 StealJS。现在让我们更详细地看看它。

使用 StealJS，我们可以按如下方式加载文件：

```js
steal(
    'file_one',
    'file_two',
    function ($) {

    }
);
```

这些文件并行且随机顺序加载。如果`file_two`在`file_one`中有依赖，我们可以在开始获取`file_two`之前等待`file_one`，如下所示：

```js
steal(
    'file_one').then(

    'file_two',
    function ($) {

    }
);
```

# 记录器

`steal.dev`提供了两个类似于流行`console.log()`函数的日志功能，并自动从生产构建中移除它们。

我们可以这样使用它：

```js
steal.dev.log('See me in the console');
steal.dev.warn('Me too!');
```

### 注意

所有日志都从生产构建中移除。

# 代码清理器

`steal.clean`美化 JavaScript 代码，并使用**JSLint**代码质量工具进行检查：

+   [`www.jslint.com`](http://www.jslint.com)

+   [`jsbeautifier.org`](http://jsbeautifier.org)

我们可以使用`cleanjs`命令来美化单个文件中的代码：

```js
$ ./js steal/cleanjs todo/todo.js
```

或者我们项目中的所有文件：

```js
$ ./js steal/cleanjs todo/todo.html
```

要对 JSLint 运行我们的代码，请添加`-jslint true`参数：

```js
$ ./js steal/cleanjs todo/todo.js -jslint true
```

我们可以通过添加类似以下注释来忽略文件被清理：

```js
//!steal-clean
```

# 连接和压缩

`steal.build`将 CSS 和 JavaScript 文件压缩和连接到单个或多个文件中。默认情况下，它使用 Google Closure compressor。

由于它通过 Envjs 打开应用程序，它可以针对不使用 StealJS 的应用程序运行。

要制作我们的`Todo`应用程序的生产就绪文件，我们可以运行以下命令：

```js
$ ./js steal/buildjs todo/todo.html -to todo_prod
```

我们可以对 URL 运行脚本：

```js
$ ./js steal/buildjs http://YOUR_SERVER/todo/todo.html -to todo_prod
```

# 摘要

在本章中，我们学习了如何将文件加载到项目中，使用了在生产构建中移除的跨浏览器日志系统，清理了代码并制作了一个生产就绪的应用程序。
