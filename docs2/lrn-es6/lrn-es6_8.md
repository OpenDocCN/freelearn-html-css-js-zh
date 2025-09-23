# 第八章 模块化编程

**模块化编程**是软件设计中最重要且最常用的技术之一。不幸的是，JavaScript 没有原生支持模块，这导致 JavaScript 程序员使用替代技术来实现 JavaScript 中的模块化编程。但现在，ES6 正式将模块引入 JavaScript。

本章全部关于如何创建和导入 JavaScript 模块。在本章中，我们将首先学习模块是如何在早期创建的，然后我们将跳转到 ES6 中引入的新内置模块系统，称为 ES6 模块。

在本章中，我们将涵盖：

+   什么是模块化编程？

+   模块化编程的好处

+   IIFE 模块、AMD、UMD 和 CommonJS 的基本原理

+   创建和导入 ES6 模块

+   模块加载器的基本原理

+   使用模块创建基本的 JavaScript 库

# 简要介绍 JavaScript 模块

将程序和库分解为模块的实践称为模块化编程。

在 JavaScript 中，模块是一组相关的对象、函数和其他程序或库的组件，它们被封装在一起，并从程序或库的其余部分或库的作用域中隔离出来。

模块将一些变量导出到外部程序，以便它访问模块封装的组件。要使用模块，程序需要导入模块及其导出的变量。

模块也可以进一步拆分为称为其子模块的模块，从而创建模块层次结构。

模块化编程有许多好处。其中一些好处包括：

+   通过将代码拆分为多个模块，它既保持了代码的清晰分离，又进行了组织。

+   模块化编程导致全局变量更少，也就是说，它消除了全局变量的问题，因为模块不通过全局作用域进行接口，每个模块都有自己的作用域

+   使得代码的可重用性更容易，因为在不同项目中导入和使用相同的模块更容易。

+   它允许许多程序员在同一程序或库上协作，通过让每个程序员针对具有特定功能的特定模块进行工作。

+   应用程序中的错误可以很容易地识别，因为它们被局部化到特定的模块中

# 实现模块——旧的方式

在 ES6 之前，JavaScript 从未支持过原生的模块。开发者使用其他技术和第三方库在 JavaScript 中实现模块。

使用**立即执行函数表达式**（**IIFE**）、**异步模块定义**（**AMD**）、**CommonJS**和**通用模块定义**（**UMD**）是 ES5 中实现模块的多种流行方式。由于这些方式并非 JavaScript 的本地特性，它们存在一些问题。让我们来概述一下这些旧的模块实现方式。

## 立即执行函数表达式

立即执行函数表达式（IIFE）用于创建一个自调用的匿名函数。使用 IIFE 创建模块是创建模块最流行的方式。

让我们看看如何使用 IIFE 创建模块的示例：

```js
//Module Starts

(function(window){
  var sum = function(x, y){
    return x + y;
  }

  var sub = function(x, y){
    return x - y;
  }

  var math = {
    findSum: function(a, b){
      return sum(a,b);
    },
    findSub: function(a, b){
      return sub(a, b);
    }
  }

  window.math = math;
})(window)

//Module Ends

console.log(math.findSum(1, 2)); //Output "3"
console.log(math.findSub(1, 2)); //Output "-1"
```

在这里，我们使用 IIFE 创建了一个模块。`sum`和`sub`变量在模块内部是全局的，但在模块外部不可见。`math`变量由模块导出到主程序，以公开它提供的功能。

此模块完全独立于程序，可以通过将其复制到源代码中或作为单独的文件导入，由任何其他程序导入。

### 注意

使用立即执行函数表达式（IIFE）的库，例如 jQuery，将其所有 API 封装在一个单独的 IIFE 模块中。当程序使用 jQuery 库时，它会自动导入该模块。

## 异步模块定义

AMD 是浏览器中实现模块化的规范。AMD 在设计时考虑到浏览器的限制，即异步导入模块以防止阻塞网页的加载。由于 AMD 不是浏览器的原生规范，我们需要使用 AMD 库。**RequireJS**是最流行的 AMD 库。

让我们看看如何使用 RequireJS 创建和导入模块的示例。根据 AMD 规范，每个模块都需要由一个单独的文件表示。因此，首先创建一个名为`math.js`的文件，表示一个模块。以下是模块中的示例代码：

```js
define(function(){
  var sum = function(x, y){
    return x + y;
  }
  var sub = function(x, y){
    return x - y;
  }
  var math = {
    findSum: function(a, b){
      return sum(a,b);
    },
    findSub: function(a, b){
      return sub(a, b);
    }
  }
  return math;
});
```

在这里，模块导出`math`变量以公开其功能。

现在，让我们创建一个名为`index.js`的文件，它充当主程序，导入模块和导出的变量。以下是`index.js`文件中的代码：

```js
require(["math"], function(math){
  console.log(math.findSum(1, 2)); //Output "3"
  console.log(math.findSub(1, 2)); //Output "-1"
})
```

在这里，第一个参数中的`math`变量是作为 AMD 模块处理的文件名。文件名`.js`扩展名由 RequireJS 自动添加。

第二个参数中的`math`变量引用了导出的变量。

在这里，模块是异步导入的，回调函数也是异步执行的。

## CommonJS

CommonJS 是 Node.js 中实现模块化的规范。根据 CommonJS 规范，每个模块都需要由一个单独的文件表示。CommonJS 模块是同步导入的。

让我们看看如何使用 CommonJS 创建和导入模块的示例。首先，我们将创建一个名为`math.js`的文件，表示一个模块。以下是模块中的示例代码：

```js
var sum = function(x, y){
  return x + y;
}

var sub = function(x, y){
  return x - y;
}

var math = {
  findSum: function(a, b){
    return sum(a,b);
  },

  findSub: function(a, b){
    return sub(a, b);
  }
}

exports.math = math;
```

在这里，模块导出`math`变量以公开其功能。

现在，让我们创建一个名为`index.js`的文件，它充当主程序，导入模块。以下是`index.js`文件中的代码：

```js
var math = require("./math").math;

console.log(math.findSum(1, 2)); //Output "3"
console.log(math.findSub(1, 2)); //Output "-1"
```

在这里，`math`变量是作为模块处理的文件名。CommonJS 会自动为文件名添加`.js`扩展名。

## 通用模块定义

我们看到了实现模块的三个不同规范。这三个规范都有它们各自创建和导入模块的方式。如果我们可以创建可以以 IIFE、AMD 或 CommonJS 模块导入的模块，那岂不是很好？

UMD 是一组技术，用于创建可以作为 IIFE、CommonJS 或 AMD 模块导入的模块。因此，现在程序可以导入第三方模块，无论它使用的是哪种模块规范。

最流行的 UMD 技术是 `returnExports`。根据 `returnExports` 技术规范，每个模块都需要由一个单独的文件表示。所以，让我们创建一个名为 `math.js` 的文件来表示一个模块。以下是模块内部的示例代码：

```js
(function (root, factory) {
  //Environment Detection
  if (typeof define === 'function' && define.amd) {
    define([], factory);
  } else if (typeof exports === 'object') {
    module.exports = factory();
  } else {
    root.returnExports = factory();
  }
}(this, function () {
  //Module Definition
  var sum = function(x, y){
    return x + y;
  }
  var sub = function(x, y){
    return x - y;
  }
  var math = {
    findSum: function(a, b){
      return sum(a,b);
    },
    findSub: function(a, b){
      return sub(a, b);
    }
  }
  return math;
}));
```

现在，你可以根据需要成功导入 `math.js` 模块，例如，使用 CommonJS、RequireJS 或 IIFE。

# 实现模块 – 新方法

ES6 引入了一种新的模块系统，称为 ES6 模块。ES6 模块是原生支持的，因此它们可以被称为标准 JavaScript 模块。

你应该考虑使用 ES6 模块而不是旧方法，因为它们有更简洁的语法、更好的性能，以及许多可能作为 ES6 模块打包的新 API。

让我们详细看看 ES6 模块。

## 创建 ES6 模块

每个 ES6 模块都需要由一个单独的 `.js` 文件表示。ES6 模块可以包含任何 JavaScript 代码，并且可以导出任意数量的变量。

一个模块可以导出一个变量、函数、类或任何其他实体。

我们需要在模块中使用 `export` 语句来导出变量。`export` 语句有多种不同的格式。以下是这些格式：

```js
export {variableName};
export {variableName1, variableName2, variableName3};
export {variableName as myVariableName};
export {variableName1 as myVariableName1, variableName2 as myVariableName2};
export {variableName as default};
export {variableName as default, variableName1 as myVariableName1, variableName2};
export default function(){};
export {variableName1, variableName2} from "myAnotherModule";
export * from "myAnotherModule";
```

这些格式之间的区别如下：

+   第一种格式导出一个变量。

+   第二种格式用于导出多个变量。

+   第三种格式用于使用另一个名称导出一个变量，即别名。

+   第四种格式用于导出具有不同名称的多个变量。

+   第五种格式使用 `default` 作为别名。我们将在本章后面了解这种用法。

+   第六种格式与第四种格式类似，但它也具有 `default` 别名。

+   第七种格式与第五种格式类似，但在这里你可以放置一个表达式而不是变量名。

+   第八种格式用于导出子模块的导出变量。

+   第九种格式用于导出子模块的所有导出变量。

关于 `export` 语句，以下是一些你需要了解的重要事项：

+   导出语句可以在模块的任何位置使用。不需要在模块的末尾使用它。

+   模块中可以有任意数量的 `export` 语句。

+   你不能按需导出变量。例如，将 `export` 语句放在 `if…else` 条件中会抛出错误。因此，我们可以说模块结构需要是静态的，即导出可以在编译时确定。

+   您不能多次导出相同的变量名或别名。但您可以多次导出变量，使用不同的别名。

+   默认情况下，模块内的所有代码都在 `strict` 模式下执行。

+   导出变量的值可以在导出它们的模块内部更改。

## 导入 ES6 模块

要导入一个模块，我们需要使用 `import` 语句。`import` 语句有多种不同的格式。以下是格式：

```js
import x from "module-relative-path";
import {x} from "module-relative-path";
import {x1 as x2} from "module-relative-path";
import {x1, x2} from "module-relative-path";
import {x1, x2 as x3} from "module-relative-path";
import x, {x1, x2} from "module-relative-path";
import "module-relative-path";
import * as x from "module-relative-path";
import x1, * as x2 from "module-relative-path";
```

一个 `import` 语句由两部分组成：我们想要导入的变量名和模块的相对路径。

这里是这些格式之间的差异：

+   在第一种格式中，导入的是 `default` 别名。`x` 是 `default` 别名的别名。

+   在第二种格式中，导入的是 `x` 变量。

+   第三种格式与第二种格式相同。只是 `x2` 是 `x1` 的别名。

+   在第四种格式中，我们导入 `x1` 和 `x2` 变量。

+   在第五种格式中，我们导入 `x1` 和 `x2` 变量。`x3` 是 `x2` 变量的别名。

+   在第六种格式中，我们导入 `x1` 和 `x2` 变量以及 `default` 别名。`x` 是 `default` 别名的别名。

+   在第七种格式中，我们只导入模块。我们不导入模块导出的任何变量。

+   在第八种格式中，我们导入所有变量，并将它们包装在一个名为 `x` 的对象中。即使是 `default` 别名也被导入。

+   第九种格式与第八种格式相同。在这里，我们给 `default` 别名提供了另一个别名。

这里有一些关于 `import` 语句的重要事项，您需要了解：

+   在导入变量时，如果我们用别名导入它，那么要引用该变量，我们必须使用别名而不是实际变量名，也就是说，实际变量名将不可见，只有别名将是可见的。

+   `import` 语句不会导入导出变量的副本；相反，它使变量在导入它的程序的作用域中可用。因此，如果您在模块内部更改导出的变量，则更改对导入它的程序是可见的。

+   导入的变量是只读的，也就是说，您不能在导出它们的模块作用域之外将它们重新分配给其他内容。

+   在 JavaScript 引擎的单个实例中，一个模块只能导入一次。如果我们再次尝试导入，则将使用已导入的模块实例。

+   我们不能按需导入模块。例如，将 `import` 语句放在 `if…else` 条件中会抛出错误。因此，我们可以说导入应该在编译时确定。

+   ES6 导入比 AMD 和 CommonJS 导入更快，因为 ES6 导入是原生支持的，并且导入模块和导出变量不是按需决定的。因此，这使得 JavaScript 引擎更容易优化性能。

## 模块加载器

模块加载器是 JavaScript 引擎的一个组件，负责导入模块。

`import` 语句使用内置模块加载器来导入模块。

不同 JavaScript 环境的内置模块加载器使用不同的模块加载机制。例如，当我们导入在浏览器中运行的 JavaScript 模块时，模块是从服务器加载的。另一方面，当我们导入 Node.js 中的模块时，模块是从文件系统中加载的。

模块加载器以不同的方式在不同的环境中加载模块，以优化性能。例如，在浏览器中，模块加载器异步加载和执行模块，以防止导入的模块阻塞网页的加载。

您可以使用模块加载器 API 以编程方式与内置模块加载器交互，以自定义其行为，拦截模块加载，并按需获取模块。

我们还可以使用此 API 创建我们自己的自定义模块加载器。

ES6 中没有指定模块加载器的规范。它是一个独立的标准，由**WHATWG**浏览器标准小组控制。您可以在[`whatwg.github.io/loader/`](http://whatwg.github.io/loader/)找到模块加载器的规范。

ES6 规范仅指定了 `import` 和 `export` 语句。

## 在浏览器中使用模块

`<script>` 标签内的代码不支持 `import` 语句，因为标签的同步性质与浏览器中模块的异步性不兼容。相反，您需要使用新的 `<module>` 标签来导入模块。

使用新的 `<module>` 标签，我们可以将脚本定义为模块。现在，这个模块可以使用 `import` 语句导入其他模块。

如果您想使用 `<script>` 标签导入模块，那么您必须使用**模块加载器 API**。

`<module>` 标签的规范没有在 ES6 中指定。

## 在 eval() 函数中使用模块

您不能在 `eval()` 函数中使用 `import` 和 `export` 语句。要在 `eval()` 函数中导入模块，您需要使用模块加载器 API。

## 默认导出与命名导出的比较

当我们使用 `default` 别名导出一个变量时，它被称为**默认导出**。显然，一个模块中只能有一个默认导出，因为别名只能使用一次。

除了默认导出之外的所有导出都称为**命名导出**。

建议模块要么使用默认导出，要么使用命名导出。同时使用两者不是一个好的做法。

当我们只想导出一个变量时，使用默认导出。另一方面，当我们要导出多个变量时，使用命名导出。

## 深入一个例子

让我们使用 ES6 模块创建一个基本的 JavaScript 库。这将帮助我们了解如何使用 `import` 和 `export` 语句。我们还将学习一个模块如何导入其他模块。

我们将要创建的库将是一个数学库，它提供基本的对数和三角函数。让我们开始创建我们的库：

+   创建一个名为 `math.js` 的文件，以及一个名为 `math_modules` 的目录。在 `math_modules` 目录内，分别创建两个名为 `logarithm.js` 和 `trigonometry.js` 的文件。

    在这里，`math.js` 文件是根模块，而 `logarithm.js` 和 `trigonometry.js` 文件是其子模块。

+   将此代码放置在 `logarithm.js` 文件中：

    ```js
    var LN2 = Math.LN2;
    var N10 = Math.LN10;

    function getLN2()
    {
      return LN2;
    }

    function getLN10()
    {
      return LN10;
    }

    export {getLN2, getLN10};
    ```

    在这里，模块导出的是名为 exports 的函数。

在模块层次结构中，低级模块应该分别导出所有变量是首选的，因为可能程序只需要库中的一个导出变量。在这种情况下，程序可以直接导入此模块和特定函数。当你只需要一个模块时加载所有模块在性能方面是不好的。

同样，将此代码放置在 `trigonometry.js` 文件中：

```js
var cos = Math.cos;
var sin = Math.sin;

function getSin(value)
{
  return sin(value);
}

function getCos(value)
{
  return cos(value);
}

export {getCos, getSin};
```

这里我们做类似的事情。将此代码放置在 `math.js` 文件中，该文件作为根模块：

```js
import * as logarithm from "math_modules/logarithm";
import * as trigonometry from "math_modules/trigonometry";

export default {
  logarithm: logarithm,
  trigonometry: trigonometry
}
```

它不包含任何库函数。相反，它使得程序能够轻松导入整个库。它导入其子模块，然后将它们导出的变量导出到主程序中。

在这里，如果 `logarithm.js` 和 `trigonometry.js` 脚本依赖于其他子模块，那么 `math.js` 模块不应该导入这些子模块，因为 `logarithm.js` 和 `trigonometry.js` 已经导入了它们。

这里是程序可以用来导入整个库的代码：

```js
import math from "math";

console.log(math.trigonometry.getSin(3));
console.log(math.logarithm.getLN2(3));
```

# 摘要

在本章中，我们了解了模块化编程是什么，学习了不同的模块化编程规范。最后，我们使用模块化编程设计技术创建了一个基本的库。现在，你应该有足够的信心使用 ES6 模块构建 JavaScript 应用程序。
