# 模块化编程

模块化编程是软件设计中最重要的和最常用的技术之一。模块化编程基本上意味着将你的代码拆分成多个文件，这些文件通常彼此独立。这使得管理和维护程序的不同模块变得轻而易举。它有助于轻松调试讨厌的 bug，向特定模块推送更新等等。

不幸的是，长期以来，JavaScript 没有原生支持模块；这导致程序员使用替代技术来实现 JavaScript 中的模块化编程。然而，ES6 首次在 JavaScript 中引入了原生的模块技术。

本章全部关于如何创建和导入 JavaScript 模块。在本章中，我们将首先学习模块是如何早期创建的，然后我们将跳转到新的内置 JavaScript 模块系统。

在本章中，我们将涵盖：

+   什么是模块化编程？

+   模块化编程的好处

+   IIFE 模块、AMD、UMD 和 CommonJS 模块的基本原理

+   创建和导入 ES6 模块

+   在浏览器中实现模块

# JavaScript 模块 101

将程序和库分解为模块的实践称为模块化编程。

在 JavaScript 中，**模块**是一组相关的对象、函数和程序或库的其他组件的集合，这些组件被封装在一起，并从程序或库的其余部分的作用域中隔离出来。

模块导出一些变量到外部程序，以便它能够访问模块封装的组件。要使用模块，程序需要导入模块及其导出的变量。

一个模块也可以进一步拆分为称为子模块的模块，从而创建模块层次结构。

模块化编程有许多好处。以下是一些好处：

+   通过将其拆分为多个模块，它既保持了代码的清晰分离，又保持了组织性

+   模块化编程导致全局变量更少，也就是说，它消除了全局变量的问题，因为模块不通过全局作用域进行接口交互，每个模块都有自己的作用域

+   它使得代码的可重用性更容易，因为在不同项目中导入和使用相同的模块更容易

+   它允许许多程序员在同一程序或库上协作，通过让每个程序员专注于具有特定功能的特定模块

+   应用程序中的 bug 很容易被识别，因为它们被局部化到特定的模块中

# 实现模块 - 旧方法

在 ES6 之前，JavaScript 从未原生支持模块。开发者使用其他技术和第三方库在 JavaScript 中实现模块。使用**立即执行的函数表达式**（**IIFE**）、**异步模块定义**（**AMD**）、CommonJS 和**通用模块定义**（**UMD**）是 ES5 中实现模块的几种流行方式。由于这些方式不是 JavaScript 的本地特性，它们存在一些问题。让我们概述一下这些旧的模块实现方式。

# 立即执行的函数表达式（IIFE）

我们在前面章节中简要讨论了 IIFE 函数。它基本上是一个自动执行的匿名函数。让我们来看一个例子。这是一个典型的使用 IIFE 的老式 JS 模块的例子：

```js
//Module Starts 

(function(window){ 
const sum =(x, y) => x + y;
const sub = (x,y) => x - y;
const math = { 
  findSum(a, b) { return sum(a, b) }, 
  findSub(a,b) { return sub(a, b) }
} 
window.math = math; 
})(window) 

//Module Ends 

console.log(math.findSum(1, 2)); //Output "3" 
console.log(math.findSub(1, 2)); //Output "-1"
```

在这里，我们使用 IIFE 创建了一个模块。`sum`和`sub`变量在模块内部是全局的，但在模块外部不可见。`math`变量由模块导出到主程序，以暴露它提供的功能。

此模块完全独立于程序，可以通过将其复制到源代码中或作为单独的文件导入（通过包括脚本 src 或使用外部库）来由任何其他程序导入。

这之所以能工作，是因为最终你将`math`对象附加到了全局的 window 对象上（即全局作用域）。

使用立即执行函数表达式（IIFE）的库，例如 jQuery，会将所有的 API 封装在一个单独的 IIFE 模块中。当程序使用 jQuery 库时，它会自动导入该模块。

# 异步模块定义（AMD）

AMD 是为在浏览器中实现模块而设计的规范。AMD 在设计时考虑了浏览器的限制，即异步导入模块以防止阻塞网页的加载。由于 AMD 不是原生的浏览器规范，我们需要使用 AMD 库。

RequireJS 是最流行的 AMD 库。让我们看看如何使用 RequireJS 创建和导入模块的例子。根据 AMD 规范，每个模块都需要由一个单独的文件表示。所以首先，创建一个名为`math.js`的文件来表示模块。以下是模块中的示例代码：

```js
define(function(){ 
  const sum = (x, y) => x + y
  const sub = (x, y) => x - y
  const math = { 
    findSum(a, b) { return sum(a,b) }, 
    findSub(a, b){ return sub(a, b); } 
  } 
  return math; 
});
```

在这里，模块导出`math`变量以暴露其功能。

现在，让我们创建一个名为`index.js`的文件，作为导入模块和导出变量的主程序。以下是`index.js`文件中的代码：

```js
require(["math"], function(math){ 
  console.log(math.findSum(1, 2)); //Output "3" 
  console.log(math.findSub(1, 2)); //Output "-1" 
}) 
```

在这里，第一个参数中的`math`变量是作为 AMD 模块处理的文件名。文件名添加的`.js`扩展名是由 RequireJS 自动添加的。

第二个参数中的`math`变量引用了导出的变量。在这里，模块是异步导入的，回调也是异步执行的。

你可以在这里了解更多关于 RequireJS 及其使用的信息：[`bit.ly/requirejs-tutorials`](http://bit.ly/requirejs-tutorials)。

# CommonJS

CommonJS 是目前最广泛使用的非官方规范。CommonJS 是在 Node.js 中实现模块的规范。根据 CommonJS 规范，每个模块都需要由一个单独的文件表示。CommonJS 模块是 **同步** 导入的。这也是为什么浏览器不使用 CommonJS 作为模块加载器的原因！

让我们看看如何使用 CommonJS 创建和导入模块的示例。首先，我们将创建一个名为 `math.js` 的文件，它代表一个模块。以下是模块内部的示例代码：

```js
const sum =(x, y) => x + y;
const sub = (x, y) => x - y;
const math = { 
  findSum(a, b) { 
    return sum(a,b); 
  }, 
  findSub(a, b){ 
    return sub(a, b); 
  } 
} 

exports.math = math; // or module.exports.math = math
```

在这里，模块导出 `math` 变量以公开其功能。现在，让我们创建一个名为 `index.js` 的文件，作为导入模块的主程序。

这里是 `index.js` 文件内部的代码：

```js
const math = require("./math").math; 
console.log(math.findSum(1, 2)); //Output "3" 
console.log(math.findSub(1, 2)); //Output "-1" 
```

在这里，`math` 变量是作为模块处理的文件的名称。CommonJS 会自动为文件名添加 `.js` 扩展名。

# exports 与 module.exports 的区别

此前，我们说过，你可以使用 `exports.math` 或 `module.exports.math` 来从模块导出一个变量。两者之间有什么区别？

技术上讲，exports 和 `module.exports` 指向同一个对象。你可以这样考虑：

```js
exports = module.exports = { }
```

这就是开始的方式。现在，无论你是否将属性值分配给 `module.exports` 或 exports，因为它们都指向同一个对象。然而，你必须记住，实际上是 **`module.exports`** 被实际导出！

例如，考虑 `string1.js`、`string2.js` 和 `index.js`。

`string1.js` 如下所示：

```js
// string1.js
module.exports = () => "Amazing string" // correct export of function
```

`string2.js` 如下所示：

```js
// string2.js
exports = () => "Amazing string" // this fails to export this function
```

`index.js` 如下所示：

```js
// index.js
console.log(require('string1.js')()); // <-- we're executing the imported function
console.log(require('string2.js')());
```

你认为输出是什么？

很明显，正如我们之前所说的，`exports` 并没有被导出。因此，第二次调用 (`require('string2.js')()`) 抛出错误，因为 `require('string2.js')` 返回一个空对象（因此你不能将其作为函数执行）。

另一方面，当 `module.exports` 从对象更改为函数时，该函数被导出，并由开发者在一行调用 `(string1.js)` 中调用。

# 通用模块定义 (UMD)

我们看到了实现模块的三个不同规范。这三个规范都有它们各自创建和导入模块的方式。如果我们能够创建可以被作为 IIFE、AMD 或 CommonJS 模块导入的模块，那岂不是很好？

UMD 是一组用于创建可以作为 IIFE、CommonJS 或 AMD 模块导入的模块的技术。因此，现在一个程序可以导入第三方模块，无论它使用什么模块规范。

最流行的 UMD 技术是 `returnExports`。根据 `returnExports` 技术，每个模块都需要由一个单独的文件表示。所以，让我们创建一个名为 `math.js` 的文件来表示一个模块。以下是模块内部的示例代码：

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
  const sum = (x, y) => x + y;
  const sub = (x, y) => x - y;
  const math = { 
    findSum(a, b) { 
        return sum(a,b); 
    }, 
    findSub(a, b) { 
        return sub(a, b); 
    } 
  } 
return math; 
})
);
```

现在，你可以以你希望的方式成功导入 `math.js` 模块，例如，使用 CommonJS、RequireJS 或 IIFE。

# 实现模块 – 新方法

在 JavaScript 中导入和导出模块的新方法就是官方模块系统。由于它被语言原生支持，因此可以称为标准 JavaScript 模块系统。你应该在实际应用中考虑使用官方模块系统，因为它原生且因此针对速度和性能进行了优化。

# 导入/导出模块

假设你正在编写一个模块文件，现在你准备好将其导入到主文件中。你将如何使用官方模块系统导出它？以下是方法：

```js
// module.js

const takeSquareAndAdd2 = num => {
    return num*num + 2;
}

export { takeSquareAndAdd2 }; // #1
export const someVariable = 100; // #2
export function yourName(name) {
    return `Your name ${name} is a nice name` 
}; // #3
export default "Holy moly this is interesting!" // #4
```

+   `#1`: 我们首先编写了一个函数，然后使用 `export` 关键字使其对导入此特定模块的其他模块可用。

+   `#2`: 你可以在一行中直接声明、初始化和导出变量/函数。

+   `#3`: 如 `#2` 所述，你可以在创建函数的同时直接导出它们。

+   `#4`: 前三个被称为命名导出，而这是默认导出。我们很快就会看到两者之间的区别。

让我们看看如何在一个单独的文件中导入这个先前的模块：

```js
// index.js
import myString, { takeSquareAndAdd2 } from './module.js'
console.log(myString) // "Holy moly this is interesting"
console.log(takeSquareAndAdd2(2)) // 6
```

等等。这里发生了什么？让我们来研究一下。

# 命名导出与默认导出

之前，我们看到了我们使用了 `export default`：*Holy moly this is interesting*。它所做的，当我们使用 `import <varname> from './module'` 时，将 `<varname>` 赋值为默认导出的值。因此，请看以下内容：

```js
// index.js
import string from './module.js'
console.log(string)
```

这将在控制台输出 *Holy moly this is interesting*。

这被称为默认导出。

另一方面，命名导出与一个名称相关联（变量的名称或函数的名称）。你必须使用解构语法导入命名导出变量。这是因为你可以将 `export` 关键字视为导出一个默认值 *加上* 包含所有其他导出（命名导出）的对象。

因此，请看以下内容：

```js
import { takeSquareAndAdd2 } from './module.js';
console.log(takeSquareAndAdd2(3))
```

这将输出 `11`。

当进行默认导出时，你不能使用 `var`/`let`/`const`；只需执行 `export default <YOUR VALUE HERE>`。然而，你可以执行 `export default` 后跟一个函数。

# 命名导入

你也可以更改你导入的模块中命名导出的名称。这是通过使用 `as` 关键字实现的：

```js
import { takeSquareAndAdd2 as myFunc } from './module.js';
console.log(myFunc(3))
```

这仍然会产生输出 `11`。这在你有长模块名或可能与你基础代码/其他导入模块冲突的名称时是必要的。

类似地，你也可以重命名一些命名导入，其余的保持不变：

```js
import { takeSquareAndAdd2 as myFunc, yourName } from './module.js';
console.log(myFunc(3)) // 11
console.log(yourName("Mehul")) // Your name Mehul is a nice name
```

# 通配符导入

如果你想导入整个模块中导出的所有实体呢？自己编写每个实体的名称会很麻烦；此外，如果你这样做，你将污染全局作用域。让我们看看我们如何解决这两个问题。

让我们假设我们的 `module.js` 看起来像这样：

```js
// module.js
export const PI = 3.14
export const sqrt3 = 1.73
export function returnWhatYouSay(text) { return text; }
```

让我们一次性导入所有内容：

```js
// index.js
import * as myModule from './module.js'
console.log(myModule.PI) // 3.14
console.log(myModule.returnWhatYouSay("This is cool!"))
```

星号（`*`）将导入`myModule`对象作用域下导出的所有内容。这使得访问所有导出的变量/方法更加简洁。

让我们在以下部分中快速收集有关导入/导出语法的所有信息。

# 关于导出的附加信息

我们需要在模块中使用导出语句来导出变量。导出语句有多种不同的格式。以下是这些格式：

+   `export {variableName}` - 这种格式导出一个变量

+   `export {variableName1, variableName2, variableName3}` - 这种格式用于导出多个变量

+   `export {variableName as myVariableName}` - 这种格式用于导出一个具有另一个名称的变量，即别名

+   `export {variableName1 as myVariableName1, variableName2 as myVariableName2}` - 这种格式用于导出具有不同名称的多个变量

+   `export {variableName as default}` - 这种格式使用默认作为别名

+   `export {variableName as default, variableName1 as myVariableName1, variableName2}` - 这种格式与第四种格式类似，但它还有一个默认别名

+   `export default function(){}` - 这种格式与第五种格式类似，但在这里你可以放置一个表达式而不是变量名

+   `export {variableName1, variableName2} from "myAnotherModule"` - 这种格式用于导出子模块的导出变量

+   `export * from "myAnotherModule"` - 这种格式用于导出子模块的所有导出变量

关于导出语句，以下是一些你需要知道的重要事项：

+   导出语句可以在模块的任何位置使用。不需要在模块的末尾使用它。

+   模块中可以有任意数量的导出语句。

+   你不能按需导出变量。例如，将导出语句放在`if...else`条件中会抛出错误。因此，我们可以说模块结构需要是静态的，即导出可以在编译时确定。

+   你不能多次导出相同的变量名或别名。但你可以使用不同的别名多次导出同一个变量。

+   默认情况下，模块内的所有代码都在严格模式下执行。

+   导出变量的值可以在导出它们的模块内部更改。

# 关于导入的附加信息

要`导入`一个模块，我们需要使用`import`语句。`import`语句有多种不同的格式。以下是这些格式：

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

导出语句由两部分组成：我们想要导入的变量名和模块的相对路径。

这些格式之间的差异如下：

+   `import x from "module-relative-path"` - 在这种格式中，默认别名被导入。`x`是默认别名的别名。

+   `import {x} from "module-relative-path"` - 在这种格式中，导入`x`变量。

+   `import {x1 as x2} from "module-relative-path"` - 这种格式与第二种格式相同。只是`x2`是`x1`的别名。

+   `import {x1, x2} from "module-relative-path"` - 在这种格式中，我们导入`x1`和`x2`变量。

+   `import {x1, x2 as x3} from "module-relative-path"` - 在这种格式中，我们导入`x1`和`x2`变量。`x3`是`x2`变量的别名。

+   `import x, {x1, x2} from "module-relative-path"` - 在这种格式中，我们导入`x1`、`x2`变量和默认别名。`x`是默认别名的别名。

+   `import "module-relative-path"` - 在这种格式中，我们只是导入模块。我们没有导入模块导出的任何变量。

+   `import * as x from "module-relative-path"` - 在这种格式中，我们导入所有变量并将它们包装在一个名为`x`的对象中。甚至默认别名也被导入。

+   `import x1, * as x2 from "module-relative-path"` - 第九种格式与第八种格式相同。在这里，我们给默认别名提供了另一个别名。

这里有一些关于导入语句的重要事项，你需要了解：

+   当导入一个变量时，如果我们用别名导入它，那么要引用该变量，我们必须使用别名而不是实际的变量名；也就是说，实际的变量名将不可见，只有别名是可见的。

+   导入语句不会导入导出变量的副本；相反，它使变量在导入它的程序的作用域中可用。因此，如果你在模块内部更改导出变量，那么更改对导入它的程序是可见的。

+   导入的变量是只读的，也就是说，你不能在导出它们的模块作用域之外将它们重新分配给其他东西。

+   一个模块在单个 JavaScript 引擎实例中只能导入一次。如果我们再次尝试导入它，那么已经导入的模块实例将被使用。（换句话说，模块在 JavaScript 中是单例的。）

+   我们不能按需导入模块。例如，将导入语句放在`if`…`else`条件中会抛出错误。因此，我们可以这样说，导入应该在编译时确定。

# 树摇动

树摇动是模块打包器如 Webpack 和 Rollup 用来传达可以使用导入-导出模块语法进行死代码消除的术语。

从本质上讲，新的模块加载系统使得这些模块打包器能够执行一种称为树摇动的操作，即摇动树以去除枯叶。

你的 JavaScript 就像一棵树。你导入的模块代表树上的活叶。而那些死（未使用）的代码则由树上的棕色枯叶来表示。为了移除枯叶，打包器必须摇动这棵树，让它们落下。

# 树摇动的执行方式

模块打包器使用的树摇动以以下方式消除未使用的代码：

1.  首先，打包器会将所有导入的模块文件合并在一起（就像一个好的打包器一样）。在这里，任何文件都没有导入的导出项都不会被导出。

1.  之后，打包器会压缩包并同时移除无效代码。因此，那些未导出或在其各自的模块文件内部未使用的变量/函数不会出现在压缩后的包文件中。这样，就执行了摇树优化。

# 在网络上使用模块

现代浏览器最近才开始原生实现模块加载器。要原生使用导入模块的脚本，您需要将其`type`属性设置为`module`。

下面是一个在 Chrome 63 中的非常基础的示例：

```js
// index.html
<!doctype HTML>
<html>
    <head>
        <script src="img/index.js" type="module"></script>
    </head>
    <body>
        <div id="text"></div>
    </body>
</html>

```

这就是`index.js`（主脚本文件）的外观：

```js
// index.js
import { writeText2Div as write2Div } from './module.js';
write2Div('Hello world!')
```

这是`index.js`（在同一目录下）导入的模块：

```js
// module.js
const writeText2Div = text => document.getElementByID('text').innerText = text;
export { writeText2Div };
```

当进行测试时，应该在屏幕上显示`Hello World`。

# 摘要

在本章中，我们了解了模块化编程是什么，并学习了不同的模块化编程规范。我们了解了模块化编程的未来以及如何在真实项目中使用它在网络浏览器中。随着 ECMAScript 的发展，我们期待看到更多功能和性能的提升，这将最终惠及最终用户和开发者。

在下一章中，我们将探讨一个被称为**Reflect API**的东西，这是一个用于可拦截 JavaScript 操作的新颖方法集合。
