# 第十一章：*第十章*

# 使用 JavaScript 进行函数式编程

## 学习目标

在本章结束时，您将能够：

+   在 Redux 减速器和选择器中使用纯函数

+   解决高级函数测试情况

+   在现代 JavaScript 应用程序中应用柯里化、部分应用和闭包

+   为在使用微服务构建的前端后端（BFF）中使用组合函数

+   应用 JavaScript 内置函数以在 Redux 应用程序中以不可变的方式编写

+   在 BFF 的上下文中使用 GraphQL 实现查询和变异

+   从三种方法中选择处理 React/Redux 应用程序中的副作用

在本章中，您将学习函数式编程的概念，如何在 JavaScript 中应用它们，并在流行的库（如 React、Redux）和系统（如 GraphQL 查询语言）中识别它们。

## 介绍

**函数式编程**在数学函数的定义上有很大依赖。数学函数是通过声明表达式定义的。函数式编程风格也是声明式的（与命令式编程相对），并且提倡表达式而不是语句。

JavaScript 中内置了函数式编程构造。解锁 JavaScript 中的函数式编程风格对于更深入地理解语言及其生态系统至关重要。

作为每个部分的一部分，将使用 React、Redux 和 JavaScript 中的 DOM 访问和测试模式来说明 JavaScript 中实用的函数式编程。还将包括最近的发展，如 GraphQL 和**前端后端**（**BFFs**），以展示函数式编程如何渗透到 JavaScript 编程语言的现在和未来。

函数式编程概念可以解释为什么 Redux 减速器和 React 渲染函数不能包含 API 调用。很多 JavaScript 模式和最佳实践都是由语言中的函数式构造实现的；利用函数式编程可以实现更具表现力、简洁的 JavaScript 程序，更易于理解、修改和扩展。

## 函数-一流公民

函数作为**一流**意味着它们被语言视为与任何其他“值”类型一样。这意味着在 JavaScript 中，函数可以像数字、字符串、布尔值、数组、对象等一样使用。

#### 注意

现在可能是时候看看每个人对 JavaScript 数据类型有多熟练了。原始数据类型包括布尔值、空值、未定义值、数字、（大整数）、字符串、符号、对象 à 数组/集合/映射。它们可以在对象数据类型下找到。

### 一流函数-成熟的 JavaScript 构建模块

定义一流支持的另一种方式是“如果函数是常规值，则函数是一流的”。这意味着函数可以被赋值（作为值）给一个变量，作为参数传递给其他函数，并且可以是另一个函数的返回值。让我们尝试用代码示例来理解前面的概念。

在 JavaScript 中，函数可以被赋值给变量，并应用于函数表达式（如下所示）和箭头函数。变量可以保存对已定义函数的引用，或者内联声明的函数。函数可以是命名的，也可以是匿名的：

```js
const fn = function run() {
  return 'Running...';
};
function fnExpression() {}
const otherFn = fnExpression;
const anonymousArrow = () => {};
```

函数可以作为数组中的值：

```js
const fn = () => {};
const operations = [
  fn,
  function() {
    console.log('Regular functions work');
  },
  () => console.log('Arrows work too')
];
```

函数可以作为对象中的值。此示例使用 ECMAScript 6/2015 的简写属性和方法。我们还断言`Module.fn`的输出与`fn`的输出相同：

```js
const fn = () => 'Running...';
const Module = {
  fn,
  method1() {},
  arrow: () => console.log('works too')
};
console.assert(Module.fn() === 'Running...');
```

一个函数可以作为参数传递给另一个函数：

```js
const fn = () => 'Running...';
function runner(fn) {
  return fn();
}
console.assert(runner(fn) === 'Running...');
```

### 使用一流函数的控制反转

在 JavaScript 中拥有一流函数意味着注入依赖可以像传递函数一样小。

在函数不是一等公民的语言中，我们可能需要将一个对象（类的实例）传递给构造函数，以便将依赖项注入到该依赖项的消费者中。在 JavaScript 中，我们可以利用函数是一等公民这一事实，简单地注入函数实现。这种情况的最简单示例来自前面的`runner`函数。它调用作为参数传递给它的任何函数。

这种依赖关系在 JavaScript 中非常有用。类型是动态的，往往不受检查。类和类类型的好处，比如检查错误和方法重载，在 JavaScript 中并不存在。

JavaScript 函数具有简单的接口。它们被调用时带有零个或多个参数，并引起副作用（网络请求、文件 I/O）和/或输出一些数据。

在没有类型或类型检查的依赖注入场景中，传递单个函数而不是整个实例对于依赖项的消费者（依赖项注入的代码）非常有益。

以下示例说明了一种情景，即 JavaScript 应用程序可以在客户端和服务器上运行。这称为通用 JavaScript 应用程序，即在 Node.js 和浏览器中都运行的 JavaScript 程序。通用 JavaScript 通常通过构建工具和依赖注入等模式的组合来实现。

在这种情况下，当服务器端进行 HTTP 调用时，使用基于头的授权机制。当从客户端进行 HTTP 调用时，使用基于 cookie 的授权机制。

看下面的函数定义：

```js
function getData(transport) {
  return transport('https://hello-world-micro.glitch.me').then(res => res.text())
}
```

消耗`getData`的服务器端代码如下所示，其中创建了一个`axios`函数实例来默认授权头。然后将此函数实例作为`transport`传递给`getData`：

```js
const axios = require('axios');
const axiosWithServerHeaders = axios.create({
  headers: { Authorization: 'Server-side allowed' }
});
getData(axiosWithServerHeaders);
```

消耗`getData`的客户端代码如下所示。再次创建了一个`axios`函数实例，这次使用了`withCredentials`选项（启用了发送/接收 cookie）：

```js
import axios from 'axios';
const axiosWithCookies = axios.create({
  withCredentials: true
})
getData(axiosWithCookies);
```

前面的示例展示了我们如何利用一等函数支持来在不同的 JavaScript 环境中运行的应用程序之间共享代码，方法是委托 HTTP 请求的传输机制的实现。将函数作为参数传递是 JavaScript 中依赖注入的惯用方式。

### 在 JavaScript 中启用异步 I/O 和事件驱动编程的功能

I/O，即非阻塞，以及 JavaScript 事件循环是 JavaScript 在基于浏览器的应用程序和最近的 Node.js 服务器端应用程序中受欢迎的核心。JavaScript 是单线程的，这意味着很容易理解。在 JavaScript 程序中几乎不可能找到竞争条件和死锁。

JavaScript 的异步编程模型是非阻塞交互的输入和输出机制，这意味着如果程序受到 I/O 限制，JavaScript 是处理它的一种非常有效的方式。JavaScript 不会等待 I/O 完成；相反，它会使用事件循环安排代码在 I/O 完成时恢复执行。

对于事件驱动编程，函数是需要在以后执行的逻辑的轻量级容器。JavaScript 中的函数和事件驱动编程导致了诸如`addEventListener` Web API、Node.js 错误优先回调以及随后在 ECMAScript 6/ECMAScript 2015 中移动到 A+ Promise 兼容规范等模式。

这里的所有模式都公开了一个接受函数作为其参数之一的函数。

`addEventListener` Web API 允许在浏览器中运行的 JavaScript 程序在 DOM 元素上发生事件时执行函数；例如，我们可以监听`scroll`、`click`或键盘事件。以下示例将在滚动时打印`Scrolling…`。它应该在浏览器 JavaScript 环境中运行：

```js
document.addEventListener('scroll', () => {
  console.log('Scrolling...');
});
```

Node.js 使用错误优先回调来处理其暴露的任何 I/O API。下面的例子展示了如何处理来自 Node.js 文件系统模块 `fs` 的错误。传递的回调始终具有一个错误属性作为其第一个参数。当没有错误时，此错误为 `null` 或 `undefined`，如果发生错误，则具有一个 `Error` 值：

```js
const fs = require('fs');
fs.readdir('.', (err, data) => {
  // Shouldn't error
  console.assert(Boolean(data));
  console.assert(!err);
});
fs.readdir('/tmp/nonexistent', (err, data) => {
  // Should error
  console.assert(!data);
  console.assert(Boolean(err));
});
```

Web Fetch API 提供了 A+ Promise 实现。A+ Promise 是一个封装了异步逻辑并具有 `.then` 和 `.catch` 函数的对象，这些函数接受一个函数作为参数。与 Node.js 的错误优先回调方法相比，Promise 是 JavaScript 中抽象 I/O 的一种更近期和更高级的方式。Fetch API 在 Node.js 中不是原生可用的；然而，它作为一个 npm 模块可用于在 Node.js 中使用。这意味着以下代码在 Node.js 中可用：

```js
const fetch = require('node-fetch');
fetch('https://google.com')
  .then(response => {
    console.assert(response.ok);
  })
  .catch(error => {
    // Shouldn't error
    console.assert(false);
    console.error(error.stack);
  });
```

更近期的 Node.js 版本（10+）为其一些 API 暴露了 Promise 接口。以下代码等同于之前的文件系统访问和错误处理，但使用了 Promise 接口而不是错误优先回调：

```js
const fs = require('fs').promises;
fs.readdir('.')
  .then(data => {
    console.assert(Boolean(data));
  })
  .catch(() => {
    // Shouldn't error
    console.assert(false);
  });
fs.readdir('/tmp/nonexistent')
  .then(() => {
    // Should error
    console.assert(false);
  })
  .catch(error => {
    // Should error
    console.assert(Boolean(error));
  });
```

### JavaScript 内置的数组方法展示了一流函数的支持

JavaScript 自带了一些数组对象的内置方法。其中许多方法展示了一流函数的支持。

`Array#map` 函数返回传递的函数应用于每个元素后的输出数组。下面的例子展示了一个常见的用例，即通过提取每个元素的特定对象键将对象数组转换为原始值数组。在这种情况下，对象的 `id` 属性在新数组中返回：

```js
const assert = require('assert').strict
assert.deepStrictEqual(
  [{id: '1'}, {id: '2'}].map(el => el.id),
  ['1', '2']
);
```

`Array#filter` 函数返回数组中使得作为参数传递的函数返回真值的元素。在下面的例子中，我们过滤掉任何小于或等于 2 的元素：

```js
const assert = require('assert').strict
assert.deepStrictEqual(
 [1, 2, 3, 4, 5].filter(el => el > 2),
 [3, 4, 5]
);
```

`Array#reduce` 函数接受一个函数参数，对每个元素使用累加器和当前元素值调用该函数。`reduce` 返回传递的函数参数的最后输出。它用于改变数组的形状，例如对数组中的每个元素求和：

```js
console.assert([2, 4].reduce((acc, curr) => acc + curr) === 6);
```

`Array#flatMap` 函数返回作为参数传递的函数应用于数组中每个元素后的扁平化输出。下面的例子中，新数组的长度是初始数组的两倍，因为我们为 `flatMap` 返回了一对值，以便扁平化成一个数组：

```js
const assert = require('assert').strict
assert.deepStrictEqual(
  [1, 2, 3, 4, 5, 6].flatMap(el => [el, el + 1]),
  [ 1, 2, 2, 3, 3, 4, 4, 5, 5, 6, 6, 7 ]
);
```

#### 注意

`flatMap` 是一个第四阶段的特性，在 Node.js 11+ 中可用，并且在 Chrome 69+、Firefox 62+ 和 Safari 12+ 中原生支持。

`Array#forEach` 函数对数组中的每个元素调用传递的函数。它相当于一个 for 循环，但无法中断。传递的函数将始终对每个元素调用：

```js
let sum = 0;
[1, 2, 3].forEach(n => {
  sum += n;
});
console.assert(sum === 6);
```

`Array#find` 函数对数组中的每个元素调用传递的函数，直到函数返回真值为止，此时返回该值，或者没有更多元素可以调用函数时返回 `undefined`：

```js
console.assert(['a', 'b'].find(el => el === 'c') === undefined);
```

`Array#findIndex` 函数对数组中的每个元素调用作为参数传递的函数，直到函数返回真值为止，此时返回该元素的索引，或者没有更多元素可以调用函数时返回 `-1`：

```js
console.assert(['a', 'b'].findIndex(el => el === 'b') === 1);
```

`Array#every` 函数对数组中的每个元素调用传递的函数。在每次迭代中，如果传递的函数返回 `false` 值，`.every` 将中断并返回 `false`。如果 `.every` 在不让传递的函数返回 `false` 值的情况下到达数组末尾，它将返回 `true`：

```js
console.assert([5, 6, 7, 8].every(el => el > 4));
```

`Array#some` 函数在数组的每个元素上调用传递的函数。在每次迭代中，如果传递的函数返回一个真值，`.some` 就会中断并返回 `true`。如果 `.some` 在不传递函数作为参数返回真值的情况下到达数组的末尾，它就会返回 `false`：

```js
console.assert([0, 1, 2, 5, 6, 7, 8].some(el => el < 4));
```

`Array#sort` 函数调用传递的函数来对数组进行排序。传递的函数会用数组的两个元素（我们将称之为 `a` 和 `b`）来调用。如果它返回大于 0 的值，`a` 将出现在排序后的数组中的 `b` 之前。如果比较函数返回小于 0 的值，`b` 将出现在排序后的数组中的 `a` 之前。如果比较函数返回等于 0 的值，`a` 和 `b` 将按照原始数组中的顺序出现，即相对于彼此：

```js
const assert = require('assert').strict
assert.deepStrictEqual(
  [3, 5, 1, 4].sort((a, b) => (a > b ? 1 : -1)),
  [1, 3, 4, 5]
);
```

还有其他数组方法，特别是那些对非函数参数进行操作的方法。这是展示支持传递函数的方法有多么强大的好方法。

### 练习 70：使用 some、findIndex 和 reduce 重新实现 includes、indexOf 和 join

在这个练习中，你将使用数组方法 `Array#some`、`Array#findIndex` 和 `Array#reduce` 来重新实现 `Array#includes`、`Array#indexOf` 和 `Array#join`，利用一级函数支持。它们是原始方法的更强大版本。

`npm run Exercise70` 的最终输出应该让所有断言都通过。这意味着我们现在有了符合以下断言的 `includes`、`indexOf` 和 `join` 函数：

+   如果值在数组中，`includes` 应该返回 true。

+   如果值在数组中，`includes` 应该返回 false。

+   如果值在数组中，`indexOf` 应该返回索引。

+   如果值不在数组中，`indexOf` 应该返回 `-1`。

+   `join` 应该不带分隔符工作。

+   `join` 应该使用逗号作为分隔符。

#### 注意

在这个练习中，我们将在起始文件 `exercise-re-implement-array-methods-start.js` 中为这些方法编写测试和骨架。可以使用 `node exercise-re-implement-array-methods-start.js` 运行这个文件。这个命令已经在 npm 脚本中别名为 `npm run Exercise70`。

执行以下步骤完成这个练习：

1.  将当前目录更改为 `Lesson10`。这样我们就可以使用预先映射的命令来运行我们的代码。现在，运行 `npm run Exercise70` 命令（或 `node exercise-re-implement-array-methods-start.js`）：

#### 注意

npm 脚本是在 `package.json` 的 `scripts` 部分定义的。可以使用 `npm run` `Exercise70.js` 运行这个练习的工作解决方案。文件在 GitHub 上。

![图 10.1：运行 npm run exercise1 的初始输出](img/C14587_10_01.jpg)

###### 图 10.1：运行 npm run Exercise70 的初始输出

这些错误表明提供的测试当前失败，因为实现不符合预期（因为它们目前什么也不做）。

1.  在 `exercise-re-implement-array-methods-start.js` 中实现 `includes`：

```js
function includes(array, needle) {
  return array.some(el => el === needle);
}
```

有一个我们将替换的 `includes` 骨架。我们可以用来实现 includes 的函数是 `.some`。我们将检查数组的任何/一些元素是否等于 `needle` 参数。

1.  运行 `npm run Exercise70`。这应该给出以下输出，这意味着 `includes` 按照我们的两个断言正常工作（`includes` 的断言错误已经消失）：![图 10.2：实现 includes 后的输出](img/C14587_10_02.jpg)

###### 图 10.2：实现 includes 后的输出

`needle` 是一个原始类型，所以如果我们需要比较某些东西，`el === needle` 就足够了。

1.  使用 `.findIndex` 来实现 `indexOf`：

```js
function indexOf(array, needle) {
  return array.findIndex(el => el === needle);
}
```

在这一步之后，运行 `npm run Exercise70` 应该给出以下输出，这意味着 `indexOf` 按照我们的两个断言正常工作（`indexOf` 的断言错误已经消失）：

![图 10.3：实现 includes 和 indexOf 后的输出](img/C14587_10_03.jpg)

###### 图 10.3：实现包含和 indexOf 后的输出

最后，我们将使用`.reduce`来实现`join`。这个函数更难实现，因为`reduce`是一个非常通用的遍历/累加运算符。

1.  首先，将累加器与当前元素连接起来:

```js
function join(array, delimiter = '') {
  return array.reduce((acc, curr) => acc + curr);
}
```

1.  运行 `npm run Exercise70`。您将看到“不应传递分隔符”现在通过了:![图 10.4：实现包含、indexOf 和天真的连接](img/C14587_10_04.jpg)

###### 图 10.4：实现包含、indexOf 和天真的连接

1.  除了将累加器与当前元素连接起来，还要在它们之间添加分隔符:

```js
function join(array, delimiter = '') {
  return array.reduce((acc, curr) => acc + delimiter + curr);
}
```

以下是前面代码的输出:

![图 10.5：运行练习后 npm 的最终输出](img/C14587_10_05.jpg)

###### 图 10.5：运行练习后 npm 的最终输出

这个练习展示了支持将另一个函数传递给它们的函数比仅接收原始参数的函数更强大。我们通过使用函数参数的对应项重新实现了原始参数函数来证明这一点。

在下一个练习中，我们将向您展示另一个 JavaScript 用例，用于支持函数参数的数组函数。

### 练习 71：使用 Map 和 Reduce 计算购物篮的价格

在这个练习中，您将使用数组的`map`、`filter`和`reduce`函数来完成从线项目列表到购物篮总成本的简单转换。

#### 注意

在这个练习中，您将在起始文件`exercise-price-of-basket-start.js`中有测试和方法的框架。可以使用`node exercise-price-of-basket-start.js`运行该文件。这个命令已经被别名为 npm 脚本`npm run Exercise71`。可以在 GitHub 上使用`npm run Exercise71`文件运行这个练习的工作解决方案。

1.  将当前目录更改为`Lesson10`。这样我们就可以使用预先映射的命令来运行我们的代码。运行 `npm run Exercise71`（或 `node exercise-price-of-basket-start.js`）。您将看到以下内容:![图 10.6：npm 运行的初始输出](img/C14587_10_06.jpg)

###### 图 10.6：npm 运行的初始输出

失败的断言表明，我们的框架实现没有输出它应该输出的内容，因为`basket1`的内容应该合计为`5197`，`basket2`的内容应该合计为`897`。我们可以手动运行这个计算：*1 * 199 + 2 * 2499* 是 *5197*，*2 * 199 + 1 * 499* 是 *897*。

1.  首先，获取行项目价格，这是通过在`totalBasket`中的每个项目上进行映射并将`item.price`乘以`item.quantity`来完成的:

```js
function totalBasket(basket) {
  return basket.map(item => item.price * item.quantity);
}
console.log(totalBasket(basket1))
console.log(totalBasket(basket2))
```

运行`npm run Exercise71`应该给出以下输出:

![图 10.7：npm 运行和 totalBasket 的输出，包括在.map 中计算行项目](img/C14587_10_07.jpg)

###### 图 10.7：npm 运行和 totalBasket 的输出，包括在.map 中计算行项目

请注意，断言仍然失败，因为我们没有将行项目价格相加；我们只是返回了一个行项目价格的数组。

1.  接下来，使用`reduce`来对累加器和当前行项目价格进行求和，并删除`console.log`:

```js
function totalBasket(basket) {
  return basket
    .map(item => item.price * item.quantity)
    .reduce((acc, curr) => acc + curr);
}
```

`npm run Exercise71`的最终输出不应该有断言错误:

![图 10.8：实现 totalBasket 的最终输出](img/C14587_10_08.jpg)

###### 图 10.8：实现 totalBasket 的最终输出

添加`reduce`步骤对我们用初始`map`计算的行项目价格进行求和。现在`totalBasket`返回了`basket1`和`basket2`的正确总价格，分别为`5197`和`897`。因此，以下断言现在为真:

+   `basket1`应该合计为`5197`。

+   `basket2`应该合计为`897`。

这个练习展示了如何使用 map 和 reduce 首先将对象数组转换为原始值数组，然后从中间数组中聚合数据。

### 在 React 中进行子父组件通信

流行的 JavaScript 用户界面库 React 利用 JavaScript 中函数的一流特性来实现其组件 API 接口。

组件只能明确地从消费它的组件接收属性。在 React 中，一个组件被另一个组件消费的过程通常被称为渲染，因为它自己的渲染是唯一可以使用另一个组件的地方。

在这种情况下，渲染的父组件（渲染的组件）可以将属性传递给正在渲染的子组件，如下所示：

```js
import React from 'react';
class Child extends React.Component {
  render() {
    return <div>Hello {this.props.who}</div>;
  }
}
class Parent extends React.Component {
  render() {
    return (
      <div>
        <Child who="JavaScript" />
      </div>
    );
  }
}
```

与其他流行的用户界面库（如 Vue.js 和 Angular）不同，在 Vue.js 中，属性从父级传递到子级，事件从子级发出到父级。在 Angular 中，使用输入绑定将数据从父级传递到子级。父级监听子级发出的事件并对其做出反应。

React 没有公开的构造允许数据传递回父级；只有属性。为了实现子父通信，React 提倡一种模式，即将一个函数作为属性传递给子级。传递的函数在父级上下文中定义，因此可以在父组件中执行其所需的操作，例如更新状态，触发 Redux 动作等：

```js
import React from 'react';
class Child extends React.Component {
  render() {
    return (
      <div>
        <button onClick={this.props.onDecrement}>-</button>
        <button onClick={this.props.onIncrement}>+</button>
      </div>
    );
  }
}
class Parent extends React.Component {
  constructor() {
    super();
    this.state = {
      count: 0
    };
  }
  increment() {
    this.setState({
      count: this.state.count + 1
    });
  }
  decrement() {
    this.setState({
      count: this.state.count - 1
    });
  }
  render() {
    return (
      <div>
        <p>{this.state.count}</p>
        <Child
          onIncrement={this.increment.bind(this)}
          onDecrement={this.decrement.bind(this)}
        />
      </div>
    );
  }
}
```

这种模式还暴露了 JavaScript 中一流函数的一个重大问题。当混合类/实例和一流函数时，默认情况下，类实例对象上的函数不会自动绑定到它。换句话说，我们有以下情况：

```js
import React from 'react';
class Child extends React.Component {
  render() {
    return <div>
      <p><button onClick={() => this.props.withInlineBind('inline-bind')}>inline bind</button></p>
      <p><button onClick={() => this.props.withConstructorBind('constructor-bind')}>constructor bind</button></p>
      <p><button onClick={() => this.props.withArrowProperty('arrow-property')}>arrow property</button></p>
    </div>;
  }
}
class Parent extends React.Component {
  constructor() {
    super();
    this.state = {
      display: 'default'
    };
    this.withConstructorBind = this.withConstructorBind.bind(this);
  }
  // check the render() function
  // for the .bind()
  withInlineBind(value) {
    this.setState({
      display: value
    })
  }
  // check the constructor() function
  // for the .bind()
  withConstructorBind(value) {
    this.setState({
      display: value
    })
  }
  // works as is but needs an
  // experimental JavaScript feature
  withArrowProperty = (value) => {
    this.setState({
      display: value
    })
  }
  render() {
    return (
      <div>
        <p>{this.state.display}</p>
        <Child
          withInlineBind={this.withInlineBind.bind(this)}
          withConstructorBind={this.withConstructorBind}
          withArrowProperty={this.withArrowProperty}
          />
      </div>
    );
  }
}
```

回调属性对于 React 中任何类型的子父通信都是核心的，因为它们的属性是从父级到子级和从子级到父级的唯一通信方式。下一个活动旨在实现一个`onCheckout`属性，使`Basket`组件的消费者在单击 Basket 的结账按钮时可以做出反应。

### 活动 15：onCheckout 回调属性

在这个活动中，我们将实现一个`onCheckout`属性，以在结账时显示购物车中商品的数量。

#### 注意

活动 15 配备了一个预配置的开发服务器和起始文件中方法的框架，即`activity-on-checkout-prop-start.js`和`activity-on-checkout-prop-start.html`。可以使用`npm run Activity15`运行开发服务器。此活动的工作解决方案可以在 GitHub 上使用 npm run `Activity15`文件运行。

1.  如果您之前没有在此目录中执行过，将当前目录更改为`Lesson10`并运行`npm install`。`npm install`会下载运行此活动所需的依赖项（React 和 Parcel）。此命令是`npx parcel serve activity-on-checkout-prop-start.html`的别名。

1.  访问 `http://localhost:1234`（或者启动脚本输出的任何 URL）以查看 HTML 页面。

1.  单击**继续结账**按钮。您会注意到什么都没有发生。

#### 注意

此活动的解决方案可以在第 625 页找到。

下一个练习将向您展示如何利用状态和属性将产品添加到我们的购物篮中。这个练习的起始代码并不严格与我们在活动结束后完成的代码相同。例如，状态是从 Basket 组件提升到了`App`组件。

### 练习 72：向购物篮添加产品

在这个练习中，我们将修改`addProduct`方法，以在单击`添加到购物篮`选项时更新购物篮中商品的数量。

#### 注意

练习 72 配备了一个预配置的开发服务器和起始文件中方法的框架，即`exercise-add-product-start.js`和`exercise-add-product-start.html`。可以使用`npm run Exercise72`运行开发服务器。此命令是`npx parcel serve exercise-add-product-start.html`的别名。可以在 GitHub 上使用`npm run Exercise72`文件运行此练习的工作解决方案。

1.  将当前目录更改为`Lesson10`。如果您以前没有在此目录中这样做，请运行`npm install`。现在运行`npm run Exercise 72`。您将看到应用程序启动，如下所示：![图 10.9：运行 npm run Exe](img/C14587_10_09.jpg)

###### 图 10.9：运行 npm run Exercise 72 的输出

为了使开发服务器实时重新加载我们的更改并避免配置问题，请直接编辑`exercise-add-product-start.js`文件。

1.  转到`http://localhost:1234`（或者启动脚本输出的任何 URL）。您应该看到以下 HTML 页面：![图 10.10：浏览器中的初始应用程序](img/C14587_10_10.jpg)

###### 图 10.10：浏览器中的初始应用程序

单击“添加到篮子”时，应用程序崩溃并显示空白 HTML 页面。

1.  更新“App＃addProduct”以修复崩溃。

```js
addProduct(product) {
    this.setState({
      basket: {
        items: this.state.basket.items.concat({
          name: product.name,
          price: product.price,
          quantity: 1
        })
      }
    });
  }
```

我们不是将篮子的值设置为`{}`，而是使用 JavaScript 数组的`concatenate`方法来获取篮子中的当前项目（`this.state.basket.items`）并添加传入的`product`参数，其数量为 1。

1.  要找出单击“添加到篮子”时会发生什么，我们需要找到“添加到篮子”按钮的`onClick`处理程序，然后诊断`this.addProduct()`调用的问题（篮子被设置为`{}`）：

```js
<button onClick={() => this.addProduct(this.state.product)}>
  Add to Basket
</button>
```

当我们单击“添加到篮子”按钮时，我们将看到以下内容：

![图 10.11：单击后实现添加到篮子](img/C14587_10_11.jpg)

###### 图 10.11：单击一次后实现添加到篮子

当我们再次单击“添加到篮子”时，我们将看到以下内容：

![图 10.12：单击 2 次后实现添加到篮子](img/C14587_10_12.jpg)

###### 图 10.12：单击两次后实现添加到篮子

### React 中的一级函数渲染属性

渲染属性是一种 React 组件模式，其中组件将整个区域的呈现委托给其父组件。

渲染属性是一个返回 JSX 的函数（因为它需要可呈现）。它往往会使用特定于子级的数据进行调用。然后，实现属性的数据由于呈现 JSX 而使用。这种模式在库作者中非常受欢迎，因为这意味着他们可以专注于实现组件的逻辑，而不必担心如何允许用户覆盖呈现的输出（因为这一切都被委托给用户）。

渲染属性的一个非常简单的例子是将呈现委托给父组件，但操作或数据来自公开渲染属性的组件。`ExitComponent`包装了“window.close（）”功能，但将呈现委托给其`renderExit`属性：

```js
class ExitComponent extends React.Component {
  exitPage() {
    window.close();
  }
  render() {
    return <div>{this.props.renderExit(this.exitPage.bind(this))}</div>;
  }
}
```

这意味着，例如，我们的`ExitComponent`可以用于退出页面上的链接和按钮。

这是`ExitButton`代码可能看起来像的样子：

```js
class ExitButton extends React.Component {
  render() {
    return (
      <ExitComponent
        renderExit={exit => (
          <button
            onClick={() => {
              exit();
            }}
          >
            Exit Page
          </button>
        )}
      />
    );
  }
}
```

请注意，实际页面退出逻辑在组件中没有处理；一切都由`ExitComponent`来实现。按钮的渲染完全在这里处理；`ExitComponent`不需要知道它。

以下是`ExitLink`组件可能实现的方式。再次注意，`ExitComponent`对链接一无所知，`ExitLink`对关闭窗口一无所知。

```js
class ExitLink extends React.Component {
  render() {
    return (
      <ExitComponent
        renderExit={exit => (
          <a
            onClick={e => {
              e.preventDefault();
              exit();
            }}
          >
            Exit
          </a>
        )}
      />
    );
  }
}
```

### 练习 73：使用渲染属性呈现篮子内容

在这个练习中，我们将使用渲染属性将商品呈现到购物篮中，从而使得篮子组件更加灵活。

#### 注意

练习 73 配有预配置的开发服务器和起始文件中方法的骨架，即`exercise-render-prop-start.js`和`exercise-render-prop-start.html`。可以使用`npm run Exercise73`运行开发服务器。此命令是`npx parcel serve exercise-render-prop-start.html`的别名。可以在 GitHub 上使用`npm run Exercise73`文件运行此练习的工作解决方案。

执行以下步骤以完成此练习：

1.  如果您以前没有在此目录中这样做，请将当前目录更改为`Lesson10`并运行`npm install`。`npm install`下载所需的依赖项，以便运行此活动（React 和 Parcel）。现在，运行`npm run Exercise73`。您将看到应用程序启动，如下所示：![图 10.13：运行启动文件后的输出](img/C14587_10_13.jpg)

###### 图 10.13：运行启动文件后的输出

为了使开发服务器实时重新加载我们的更改并避免配置问题，直接编辑`exercise-render-prop-start.js`文件。

1.  转到`http://localhost:1234`（或者启动脚本输出的任何 URL）。您应该看到以下 HTML 页面：![图 10.14：浏览器中的初始应用程序](img/C14587_10_14.jpg)

###### 图 10.14：浏览器中的初始应用程序

1.  找到`Basket`被呈现的地方，并添加一个`renderItem`属性，这是从项目到 JSX 的函数。这是`Basket`将用于呈现每个篮子项目的渲染属性的实现：

```js
{this.state.status === 'SHOPPING' && (
  <Basket
    items={this.state.basket.items}
    renderItem={item => (
      <div>
        x{item.quantity} - {item.name} - $
        {(item.price / 100).toFixed(2)} each{' '}
      </div>
    )}
    onCheckout={this.handleCheckout}
  />
)} 
```

1.  转到`Basket#render`方法，并映射每个`this.props.items`，使用`this.props.renderItem`来呈现项目：

```js
render() {
  return (
    <div>
      <p>You have {this.props.items.length} items in your basket</p>
      <div>{this.props.items.map(item => this.props.renderItem(item))}</div>
      <button onClick={() => this.props.onCheckout(this.props.items)}>
        Proceed to checkout
      </button>
    </div>
  );
}
```

要查看我们的更改，我们可以转到浏览器，看看篮子项目是如何呈现的：

![图 10.15：渲染篮子项目](img/C14587_10_15.jpg)

###### 图 10.15：渲染篮子项目

我们的`Basket`组件现在根据组件定义的函数呈现项目。这使得`Basket`更加强大（它可以呈现项目），但仍然非常可重用。在不同的实例中，我们可以使用具有`renderItem`属性的`Basket`，该属性不呈现任何内容，项目的分解，或者篮子项目的单价，例如。

第一类函数和我们所涵盖的模式对于编写符合惯用法的 JavaScript 至关重要。我们在 JavaScript 中利用函数式编程的另一种方式是使用纯函数。

## 纯函数

**纯函数**是指没有副作用的函数，对于相同的输入，参数将返回相同的输出值。副作用可以是任何东西，从通过引用传递的参数的值的变异（在 JavaScript 中变异原始值）到变异本地变量的值，或执行任何类型的 I/O。

纯函数可以被认为是数学函数。它只使用输入并且只影响自己的输出。

这是一个简单的纯函数，`identity`函数，它返回传递给它的任何内容作为参数：

```js
const identity = i => i;
```

注意没有副作用，也没有参数的变异或创建新变量。这个函数甚至没有主体。

纯函数的优势在于简单易于理解。它们也很容易测试；通常不需要模拟任何依赖关系，因为任何依赖关系都应该作为参数传递。纯函数倾向于操作数据，因为如果数据是它们唯一的依赖关系，它们是不允许有副作用的。这减少了测试表面积。

纯函数的缺点是纯函数从技术上讲不能做任何有趣的事情，比如 I/O，这意味着不能发送 HTTP 请求和数据库调用。

#### 注意

纯函数定义中的一个有趣的空白是 JavaScript 异步函数。从技术上讲，如果它们不包含副作用，它们仍然可以是纯的。实际上，异步函数可能被用于使用`await`运行异步操作，例如访问文件系统、HTTP 或数据库请求。一个很好的经验法则是，如果一个函数是异步的，它可能使用`await`来执行某种 I/O，因此它不是纯的。

### Redux 减速器和操作

Redux 是一个状态管理库。它对用户施加了一些限制，以提高状态更新的可预测性和代码库的长期可扩展性。

让我们看一个简单的 Redux 计数器实现来突出一些特性：

```js
const {createStore} = require('redux');
const counterReducer = (state = 0, action) => {
  switch (action.type) {
    case 'INCREMENT':
      return state + 1;
    case 'DECREMENT':
      return state - 1;
    default:
      return state;
  }
};
const store = createStore(counterReducer);
```

商店将其状态初始化为 0：

```js
console.assert(store.getState() === 0, 'initalises to 0');
```

该商店的内部状态只通过`getState`的只读接口暴露出来。要更新状态，需要分派一个动作。调用`dispatch`与`INCREMENT`和`DECREMENT`类型表明`counterReducer`按预期工作，并减少存储中的动作：

```js
store.dispatch({type: 'INCREMENT'});
console.assert(store.getState() === 1, 'incrementing works');
store.dispatch({type: 'DECREMENT'});
console.assert(store.getState() === 0, 'decrementing works');
```

#### 注意

根据 Redux 文档，Redux 有三个支柱：[`redux.js.org/introduction/three-principles`](https://redux.js.org/introduction/three-principles)。

Redux 的三个支柱在上面的例子中有所体现。我们有一个具有单一存储的系统，状态是只读的（通过`getState`访问），并且更改是由我们的减速器进行的，它是一个纯函数。`counterReducer`接受状态和动作，并返回一个新值，而不会改变`state`或`action`。

遵循这些规则，我们可以获得一个可预测且高性能的 JavaScript 应用程序状态容器。单一存储意味着不需要考虑状态存储在哪里；只读状态强制通过分派动作和减少它们来进行更新。由于减速器是纯函数，它们易于测试和推理，因为对于相同的输入它们将产生相同的输出，并且不会引起副作用或不需要的突变。

Redux 用于管理状态。到目前为止，我们一直将数据存储在 React 状态中。

### 练习 74：Redux 分派动作并将其减少为状态

在这个练习中，我们将把我们的数据状态移到 Redux 中，以便将数据操作和状态更新与呈现数据到页面的代码分离开来。

#### 注意

练习 74 配备了一个预配置的开发服务器和起始文件中方法的骨架，即`exercise-redux-dispatch-start.js`和`exercise-redux-dispatch-start.html`。可以使用`npm run Exercise74`运行开发服务器。可以在 GitHub 上使用`npm run Exercise74`文件运行此练习的工作解决方案。

执行以下步骤完成此练习：

1.  如果您以前没有在此目录中执行过此操作，请将当前目录更改为`Lesson10`并运行`npm install`。此命令是`npx parcel serve exercise-redux-dispatch-start.html`的别名。现在，运行`npm run Exercise74`。您将看到应用程序启动，如下所示：![图 10.16：npm run Exercise74 的输出](img/C14587_10_16.jpg)

###### 图 10.16：npm run Exercise74 的输出

1.  转到`http://localhost:1234`（或者起始脚本输出的任何 URL）。您应该看到以下 HTML 页面：![图 10.17：浏览器中的初始 Exercise74 应用程序](img/C14587_10_17.jpg)

###### 图 10.17：浏览器中的初始 Exercise74 应用程序

注意点击按钮没有起作用。

1.  通过分派`CONTINUE_SHOPPING`类型的动作来实现`App#continueShopping`：

```js
continueShopping() {
  this.props.dispatch({
    type: 'CONTINUE_SHOPPING'
  });
}
```

1.  在`appReducer`中，实现相应的状态减少。对于`CONTINUE_SHOPPING`，我们只需要更改状态中的`status`，因为这是我们用来显示结账视图或主产品和购物篮视图的内容：

```js
switch(action.type) {
  // other cases
  case 'CONTINUE_SHOPPING':
    return {
      ...state,
      status: 'SHOPPING'
    };
  // other cases
}
```

1.  通过分派`DONE`类型的动作来实现`App#finish`：

```js
finish() {
  this.props.dispatch({
    type: 'DONE'
  });
}
```

1.  在`appReducer`中，实现相应的状态减少。我们只需要更改状态中的`status`，因为这是我们用来显示`Done`视图的内容：

```js
switch(action.type) {
  // other cases
  case 'DONE':
    return {
      ...state,
      status: 'DONE'
    };
  // other cases
}
```

1.  通过分派`START_CHECKOUT`类型的动作来实现`handleCheckout`：

```js
handleCheckout(items) {
  this.props.dispatch({
    type: 'START_CHECKOUT',
    basket: {
      items
    }
  });
}
```

1.  在`appReducer`中，实现相应的状态减少。对于`START_CHECKOUT`，我们只需要更改状态中的`status`，因为这是我们用来显示结账视图或主产品和购物篮视图的内容：

```js
switch(action.type) {
  // other cases
  case 'START_CHECKOUT':
    return {
      ...state,
      status: 'CHECKING_OUT'
    };
  // other cases
}
```

#### 注意

`basket`对象没有被减少，因此可以在分派时省略。

1.  通过以下方式分派一个动作来实现`addProduct`。对于`ADD_PRODUCT`，我们需要`newProduct`，以及动作类型：

```js
addProduct(product) {
  this.props.dispatch({
    type: 'ADD_PRODUCT',
    newProduct: {
      name: product.name,
      price: product.price,
      quantity: 1
    }
  });
}
```

1.  在`appReducer`中，实现相应的状态减少，将新产品添加到当前商品篮中：

```js
switch(action.type) {
  // other cases
  case 'ADD_PRODUCT':
    return {
      ...state,
      basket: {
        items: state.basket.items.concat(action.newProduct)
      }
    };
  // other cases
}
```

`appReducer`完整的应该如下所示：

```js
const appReducer = (state = defaultState, action) => {
  switch (action.type) {
    case 'START_CHECKOUT':
      return {
        ...state,
        status: 'CHECKING_OUT'
      };
    case 'CONTINUE_SHOPPING':
      return {
        ...state,
        status: 'SHOPPING'
      };
    case 'DONE':
      return {
        ...state,
        status: 'DONE'
      };
    case 'ADD_PRODUCT':
      return {
        ...state,
        basket: {
          items: state.basket.items.concat(action.newProduct)
        }
      };
    default:
      return state;
  }
};
```

1.  转到`http://localhost:1234`（或者启动脚本输出的任何 URL）。应用现在应该如预期般响应点击：

![图 10.18：应用程序 wo](img/C14587_10_18.jpg)

###### 图 10.18：具有响应点击的应用程序

添加物品到购物篮并浏览应用程序（继续结账，完成，继续购物）应该与 Redux 存储实现之前的行为一样。

### 测试纯函数

纯函数很容易测试，因为它们是完全封装的。唯一可以改变的是输出，也就是返回值。唯一可以影响输出的是参数/参数值。而且，对于相同的输入集，纯函数的输出需要是相同的。

测试纯函数就像使用不同的输入调用它们并断言输出一样简单：

```js
const double = x => x * 2;
function test() {
  console.assert(double(1) === 2, '1 doubled should be 2');
  console.assert(double(-1) === -2, '-1 doubled should be -1');
  console.assert(double(0) === 0, '0 doubled should be 0');
  console.assert(double(500) === 1000, '500 doubled should be 1000');
}
test();
```

Redux 减速器是纯函数，这意味着为了测试它们，我们可以使用我们在上一个示例中刚刚看到的方法。

### 练习 75：测试减速器

在这个练习中，我们将为前一个练习中使用的减速器的一部分编写测试，即`appReducer`的`ADD_PRODUCT`情况。

#### 注意

练习 75 带有测试和起始文件中方法的框架，`exercise-reducer-test-start.js`。可以使用`node exercise-reducer-test-start.js`运行文件。这个命令已经被别名为 npm 脚本的`npm run Exercise75`。这个练习的工作解决方案可以在 GitHub 上使用 npm run exercise6 文件运行。

执行以下步骤完成这个练习：

1.  将当前目录更改为`Lesson10`。这样我们可以使用预映射的命令来运行我们的代码。

1.  现在，运行`npm run Exercise75`（或`node exercise-reducer-test-start.js`）。您将看到以下输出：![图 10.19：运行启动文件后空测试通过](img/C14587_10_19.jpg)

###### 图 10.19：运行启动文件后空测试通过

这个起始文件中只包含`ADD_PRODUCT`动作减少的简化的`appReducer`，还有一个`test`函数，新的测试将会被添加到这里。输出中没有包含错误，因为我们还没有创建任何测试。

#### 注意

为了获得`appReducer`的输出，它应该被调用与一个`state`对象和相关的`action`。在这种情况下，类型应该是`'ADD_PRODUCT'`。

1.  与之前的示例一样，我们将使用`assert.deepStrictEqual`，它检查两个对象的深度相等性。我们可以这样编写一个失败的测试。我们使用`state`和相关的`action`调用`appReducer`：

```js
function test() {
  assert.deepStrictEqual(
    appReducer(
      {
        basket: {items: []}
      },
      {
        type: 'ADD_PRODUCT',
        newProduct: {
          price: 499,
          name: 'Biscuits',
          quantity: 1
        }
      }
    ),
    {}
  );
}
```

如果我们运行`npm run Exercise75`，我们将看到以下错误。这是预期的，因为`appReducer`不会返回一个空对象作为状态：

![图 10.20：执行启动文件后显示错误](img/C14587_10_20.jpg)

###### 图 10.20：执行启动文件后显示错误

1.  我们应该使用`assert.deepStrictEqual`来确保`appReducer`按预期添加新产品。我们将预期值分配给`expected`变量，实际值分配给`actual`变量。这将有助于使测试更可读：

```js
function test() {
  const expected = {
    basket: {
      items: [
        {
          price: 499,
          name: 'Biscuits',
          quantity: 1
        }
      ]
    }
  };
  const actual = appReducer(
    {
      basket: {items: []}
    },
    {
      type: 'ADD_PRODUCT',
      newProduct: {
        price: 499,
        name: 'Biscuits',
        quantity: 1
      }
    }
  );
  assert.deepStrictEqual(actual, expected);
}
```

输出现在不应该抛出任何错误：

![图 10.21：测试通过，因为没有发现错误](img/C14587_10_21.jpg)

###### 图 10.21：测试通过，因为没有发现错误

在运行`node exercise-reducer-test.js`命令后，以下是输出：

![图 10.22：显示断言失败的输出](img/C14587_10_22.jpg)

###### 图 10.22：显示断言失败的输出

### Redux 选择器

选择器是 Redux 的另一个概念，这意味着我们可以使用选择器封装内部存储状态形状。选择器的使用者要求它想要的东西；选择器则留给使用存储状态形状特定知识来实现。选择器是纯函数；它们接受存储状态并返回一个或多个部分。

由于选择器是纯函数，它们很容易实现。下面的练习向我们展示了如何使用选择器，以便不是将消息数据放在渲染函数中或在传递 props 时，而是在一个纯函数中进行。

### 练习 76：实现一个选择器

在这个练习中，我们将使用选择器并利用它们的简单性来将项目呈现到购物篮中。

#### 注意

练习 76 带有预配置的开发服务器和起始文件中方法的框架，即`exercise-items-selector-start.js`和`exercise-items-selector-start.html`。可以使用`npm run Exercise76`运行开发服务器。可以在 GitHub 上使用`npm run Exercise76`文件运行此练习的工作解决方案。

1.  将当前目录更改为`Lesson10`，如果之前在此目录中尚未运行`npm install`，则运行它。

1.  运行`npx parcel serve exercise-items-selector-start.html`并执行`npm run Exercise76`。您将看到应用程序启动，如下所示：![图 10.23：运行起始 html 文件后的输出](img/C14587_10_23.jpg)

###### 图 10.23：运行起始 html 文件后的输出

为了使开发服务器能够实时重新加载我们的更改并避免配置问题，直接编辑`exercise-items-selector-start.js`文件。

1.  转到`http://localhost:1234`（或者起始脚本输出的任何 URL）。您应该看到以下 HTML 页面：![图 10.24：浏览器中的初始应用程序](img/C14587_10_24.jpg)

###### 图 10.24：浏览器中的初始应用程序

注意没有购物篮项目被呈现。这是因为`selectBasketItems`的初始实现。它返回一个空数组：

```js
const selectBasketItems = state => [];
```

1.  通过使用点符号和短路来实现`selectBasketItems`。如果状态有任何问题，则默认为`[]`：

```js
const selectBasketItems = state =>
  (state && state.basket && state.basket.items) || [];
```

应用程序现在应该再次按预期工作；项目将被显示：

![图 10.25：实现 selectBasketItems 后的应用程序](img/C14587_10_25.jpg)

###### 图 10.25：实现 selectBasketItems 后的应用程序

`selectBasketItems`选择器获取完整状态并返回其切片（项目）。选择器允许我们进一步将 Redux 存储库内部状态的形状与在 React 组件中使用它的方式分离。

选择器是 React/Redux 应用程序的重要组成部分。正如我们所见，它们允许 React 组件与 Redux 的内部状态形状解耦。以下活动旨在使我们能够为选择器编写测试。这与在先前的练习中测试 reducer 的情况类似。

### Activity 16：测试一个选择器

在这个活动中，我们将测试项目数组的各种状态的选择器，并确保选择器返回与购物篮中的项目对应的数组。让我们开始吧：

1.  将当前目录更改为`Lesson10`。这样可以使用预映射命令来运行我们的代码。

#### 注意

Activity 16 带有测试和起始文件中方法的框架，即`activity-items-selector-test-start.js`。可以使用`node activity-items-selector-test-start.js`运行此文件。此命令已经被别名为 npm 脚本`npm run Activity16`。可以在 GitHub 上使用`npm run Activity16`文件运行此练习的工作解决方案。

在测试函数中，使用`assert.deepStrictEqual`，执行以下操作：

1.  测试一下，对于空状态，选择器返回`[]`。

1.  测试一下，对于一个空的购物篮对象，选择器返回`[]`。

1.  测试一下，如果`items`数组已设置但为空，则选择器返回`[]`。

1.  测试一下，如果项目数组不为空并已设置，则选择器返回它。

#### 注意

此活动的解决方案可以在第 626 页找到。

纯函数是可预测的，易于测试和易于理解的。一等函数和纯函数都与下一个 JavaScript 函数式编程概念相关联：高阶函数。

## 高阶函数

高阶函数是一个要么接受函数作为参数，要么返回函数作为值的函数。

这是建立在 JavaScript 的一级函数支持之上的。在不支持一级函数的语言中，实现高阶函数是困难的。

高阶函数实现了函数组合模式。在大多数情况下，我们使用高阶函数来增强现有的函数。

### 绑定、应用和调用

`Function`对象上有一些内置的 JavaScript 方法：`bind`、`apply`和`call`。

`Function#bind`允许你为一个函数设置执行上下文。当调用时，bind 返回一个新的函数，其中第一个参数被绑定为函数的`this`上下文。bind 后面的参数在返回的函数被调用时使用。当绑定的函数被调用时，可以提供参数。这些参数将出现在参数列表中，在调用 bind 时设置参数之后。

在 React 代码中，当传递函数作为 props 时，经常使用 bind 来访问当前组件的`this`进行操作，比如`setState`或调用其他组件方法：

```js
import React from 'react';
class Parent extends React.Component {
  constructor() {
    super();
    this.state = {
      display: 'default'
    };
    this.withConstructorBind = this.withConstructorBind.bind(this);
  }
  // Check the render() function
  // for the .bind()
  withInlineBind(value) {
    this.setState({
      display: value
    });
  }
  // Check the constructor() function
  // for the .bind()
  withConstructorBind(value) {
    this.setState({
      display: value
    });
  }
  render() {
    return (
      <div>
        <p>{this.state.display}</p>
        <Child
          withInlineBind={this.withInlineBind.bind(this)}
          withConstructorBind={this.withConstructorBind}
        />
      </div>
    );
  }
}
```

`Function#bind`方法也可以在测试中用于测试函数是否被抛出。例如，运行函数意味着必须编写一个 try/catch，如果 catch 没有触发，则测试失败。使用 bind 和`assert`模块，可以以更简洁的形式编写：

```js
// Node.js built-in
const assert = require('assert').strict;
function mightThrow(shouldBeSet) {
  if (!shouldBeSet) {
    throw new Error("Doesn't work without shouldBeSet parameter");
  }
  return shouldBeSet;
}
function test() {
  assert.throws(mightThrow.bind(null), 'should throw on empty parameter');
  assert.doesNotThrow(
    mightThrow.bind(null, 'some-value'),
    'should not throw if not empty'
  );
  assert.deepStrictEqual(
    mightThrow('some-value'),
    'some-value',
    'should return input if set'
  );
}
test();
```

`Function#apply`和`Function#call`允许你调用一个函数，而不使用`fn(param1, param2, [paramX])`的语法，同时以类似`Function#bind`的方式设置`this`上下文。`Function#apply`的第一个参数是`this`上下文，第二个参数是一个数组或类数组，包含函数期望的参数。类似地，`Function#call`的第一个参数是`this`上下文；与`Function#apply`的区别在于参数的定义。在`Function#call`中，它们是一个参数列表，就像使用`Function#bind`时一样，而不是`Function#apply`期望的数组。

#### 注意

类数组对象，也称为索引集合，其中最常用的是函数中的 arguments 对象和 NodeList Web API，它们是实现了部分 Array API（例如实现`.length`）但没有完全实现的对象。仍然可以使用 JavaScript 的 apply/call 在它们上面使用数组函数。

`Function#apply`和`Function#call`严格来说不符合高阶函数的标准。在某种程度上，因为它们是函数对象的方法，我们可以说它们是隐式的高阶函数。它们被调用的函数对象是 apply/call 方法调用的隐式参数。通过从函数原型中读取，我们甚至可以这样使用它们：

```js
function identity(x) {
  return x;
}
const identityApplyBound = Function.prototype.bind.apply(identity, [
  null,
  'applyBound'
]);
const identityCallBound = Function.prototype.bind.call(
  identity,
  null,
  'callBound'
);
console.assert(
  identityApplyBound() === 'applyBound',
  'bind.apply should set parameter correctly'
);
console.assert(
  identityCallBound() === 'callBound',
  'bind.call should set parameter correctly'
);
```

在这个例子中，我们展示了 apply 和 call 是高阶函数，但只能用于其他函数上的函数。

`Function#apply`和`Function#call`在历史上将类似数组的对象转换为数组。在符合 ECMAScript 2015+的环境中，可以使用`spread`操作符以类似的方式使用。

以下三个函数允许你使用`Function#apply`、`Function#call`和数组展开将类数组对象转换为数组：

```js
function toArrayApply(arrayLike) {
  return Array.prototype.slice.apply(arrayLike);
}
function toArrayCall(arrayLike) {
  return Array.prototype.slice.call(arrayLike);
}
function toArraySpread(arrayLike) {
  return [...arrayLike];
}
```

### 柯里化和部分应用

柯里化函数是一个函数，它不是一次性接受它需要的参数数量，而是一次接受一个参数。

例如，如果一个函数接受两个参数，它的柯里化等价物将被调用两次，每次一个参数。

因此，柯里化可以被表达为将一个 n 参数函数转化为一个可以被调用 n 次的函数，每次只有一个参数。n 参数函数的经典称呼是 n 元。因此，柯里化是将一个 n 元函数转化为 n 个一元函数调用的转换：

```js
const sum = (x, y) => x + y;
const sumCurried = x => y => x + y;
console.assert(
  sum(1, 2) === sumCurried(1)(2),
  'curried version works the same for positive numbers'
);
console.assert(
  sum(10, -5) === sumCurried(10)(-5),
  'curried version works the same with a negative operand'
);
```

部分应用和柯里化经常一起介绍，概念上它们是相辅相成的。

使用柯里化的两参数函数，需要两次调用，每次使用一个参数，每次都执行与两参数非柯里化函数相同的工作。当它被调用一次时，它有一半的必要参数完全应用。从第一次调用中得到的函数是整体函数的部分应用：

```js
const sum = (x, y) => x + y;
const sumCurried = x => y => x + y;
const add1Bind = sum.bind(null, 1);
const add1Curried = sumCurried(1);
console.assert(
  add1Bind(2) === add1Curried(2),
  'curried and bound versions behave the same'
);
console.assert(add1Bind(2) === 3, 'bound version behaves correctly');
console.assert(add1Curried(2) === 3, 'curried version behaves correctly');
```

换句话说，部分应用是一种表达从接受 n 个参数的函数到接受`n` - `m`个参数的函数的转换的方式，其中 m 是已部分应用的参数数量。

如果我们想要能够重用通用功能，则柯里化和部分应用非常有用。部分应用不需要柯里化；柯里化是将函数转换为可以部分应用的函数。也可以使用 bind 进行部分应用。

柯里化和部分应用允许您从一个非常通用的函数开始，并在每次应用时将其转换为更专业的函数。

柯里化在每次调用时标准化参数的数量。部分应用没有这样的限制。您可以一次部分应用多个参数。

一元函数比二元函数更简单，二元函数比 N 元（其中 N > 2）函数更简单。

此外，如果我们只允许一次应用一个参数，那么柯里化会更简单。我们可以看到任意 n 参数部分应用具有更多的运行时复杂性，因为每个函数都需要在是否为最终调用上运行一些逻辑。

在 ES2015 中可以定义通用的 n 元柯里化如下：

```js
const curry = fn => {
  return function curried(...args) {
    if (fn.length === args.length) {
      return fn.apply(this, args);
    }
    return (...args2) => curried.apply(this, args.concat(args2));
  };
};
```

### 利用闭包 React 函数组件

在定义函数时，函数定义时作用域中的任何内容在调用/执行时仍然保持在作用域中。历史上，闭包被用于创建私有变量作用域。闭包是这个函数及其记住的定义时作用域：

```js
const counter = (function(startCount = 0) {
  let count = startCount;
  return {
    add(x) {
      count += x;
    },
    substract(x) {
      count -= x;
    },
    current() {
      return count;
    }
  };
})(0);
```

我们在 React 渲染函数中利用这一点，以在本地渲染范围中缓存 props 和 state。

React 函数组件还利用闭包，特别是使用钩子：

```js
import React from 'react';
function Hello({who}) {
  return <p>Hello {who}</p>;
}
const App = () => (
  <>
    <Hello who="Function Components!" />
  </>
);
```

函数组件非常强大，因为它们比类组件更简单。

在使用 Redux 等状态管理解决方案时，大部分重要状态都在 Redux 存储中。这意味着我们可以编写主要是无状态的功能组件，因为存储管理应用程序的任何有状态部分。

高阶函数使我们能够有效地处理函数并增强它们。高阶函数建立在一级函数支持和纯函数之上。同样，函数组合建立在高阶函数之上。

## 函数组合

函数组合是从数学中泄漏出来的另一个概念。

给定两个函数 a 和 b，compose 返回一个新函数，该函数将 a 应用于 b 的输出，然后应用于给定的一组参数。

函数组合是一种从一组较小函数创建复杂函数的方法。

这意味着您可能最终会得到一堆做一件事情的简单函数。具有单一目的的函数更擅长封装其功能，因此有助于关注点分离。

组合函数与柯里化和函数的部分应用相结合，因为柯里化/部分应用是一种允许您拥有通用函数的专业版本的技术，就像这样：

```js
const sum = x => y => x + y;
const multiply = x => y => x * y;
const compose = (f, g) => x => f(g(x));
const add1 = sum(1);
const add2 = sum(2);
const double = multiply(2);
```

要解释以下代码，我们可以得出以下结论：

+   将 2 加倍然后加 1 是 5（4 + 1）。

+   将 1 加到 2 然后加倍是 6（3 * 2）。

+   将 2 加到 2 然后加倍是 8（4 * 2）。

+   将 2 加倍然后加 2 是 6（4 + 2）。

以下使用我们已经定义的函数`add1`，`add2`和`double`，并展示了如何使用`compose`来实现前面的情况。请注意，compose 首先应用最右边的参数：

```js
console.assert(
  compose(add1, double)(2) === 5
);
console.assert(
  compose(double, add1)(2) === 6
);
console.assert(
  compose(double, add2)(2) === 8
);
console.assert(
  compose(add2, double)(2) === 6
);
```

定义`compose`的另一种方法是使用从左到右的遍历（使用`reduce`）。这样做的好处是在调用组合输出时允许我们传递任意数量的参数。为此，我们从第一个参数减少到最后一个参数，但是`reducing`的输出是一个支持任意数量参数的函数，并在调用时在当前函数之后调用先前的输出。

以下代码使用参数 rest 来允许任意数量的函数进行组合：

```js
const composeManyUnary = (...fns) => x =>
  fns.reduceRight((acc, curr) => curr(acc), x);
```

然后，它返回一个接受单个参数`x`（因此是一元的）的函数。当调用这个第二个函数时，它将从右到左调用传递给`composeManyUnary`的所有函数（最后一个参数的函数将首先被调用）。`reduceRight`的第一次迭代将使用`x`作为其参数调用最右边的函数。接下来的函数将在前一个函数调用的输出上调用。参数列表中倒数第二个函数将使用参数列表中最后一个函数的输出作为`x`的参数进行调用。参数列表中倒数第三个函数将使用倒数第二个函数的输出，依此类推，直到没有更多的函数可以调用。

### 练习 77：一个二进制到 n-ary 组合函数

在这个练习中，我们将实现一个 n-ary `compose`函数，可以用来组合任意数量的函数。

#### 注意

练习 77 带有测试和起始文件中方法的框架，`exercise-2-to-n-compose-start.js`。可以使用`node exercise-2-to-n-compose-start.js`运行该文件。该命令已经被别名为 npm 脚本`npm run Exercise77`。可以在 GitHub 上使用 npm run Exercise77 文件来运行这个练习的工作解决方案。

1.  将当前目录更改为`Lesson10`。这样我们可以使用预先映射的命令来运行我们的代码。

1.  现在，运行`npm run Exercise77`（或`node exercise-to-n-compose-start.js`）。您将看到以下输出：![图 10.26：运行练习的起始文件](img/C14587_10_26.jpg)

###### 图 10.26：运行练习的起始文件

`compose3`，`composeManyUnary`和`composeManyReduce`的断言都失败了，主要是因为它们当前被别名为`compose2`。

1.  已经实现了两个函数的`compose`：

```js
const compose2 = (f, g) => x => f(g(x));
```

`compose3`是一个天真的三参数`compose`函数，它先取第三个参数，然后在第一次调用的输出上调用第二个参数。

1.  最后，它调用第一个参数在第二个参数的输出上，就像这样：

```js
const compose3 = (f, g, h) => x => f(g(h(x)))
```

#### 注意

参数定义中最右边的函数首先被调用。

考虑参数作为一个数组，并且 JavaScript 有一个`reduceRight`函数（它从右到左遍历数组，同时保持一个累加器，就像`reduce`一样），有一个形成的前进路径。

1.  在实现`compose3`之后，我们可以再次运行`npm run Exercise77`，看到`compose3`的断言不再失败了：![图 10.27：实现 compose3 后的输出](img/C14587_10_27.jpg)

###### 图 10.27：实现 compose3 后的输出

1.  使用参数 rest 来允许任意数量的函数进行组合：

```js
const composeManyUnary = (...fns) => x =>
  fns.reduceRight((acc, curr) => curr(acc), x);
```

1.  在实现`composeManyUnary`之后，相应的失败断言现在通过了：![图 10.28：实现 compose3 和 composeManyUnary 后的输出](img/C14587_10_28.jpg)

###### 图 10.28：实现 compose3 和 composeManyUnary 后的输出

1.  定义`compose`使用从左到右的遍历（使用`reduce`）：

```js
const composeManyReduce = (...fns) =>
  fns.reduce((acc, curr) => (...args) => acc(curr(...args)));
```

我们可以使用三个函数`f`，`g`和`h`来`composeManyReduce`。我们的实现将通过这些函数开始减少。在第一次迭代时，它将返回一个函数，该函数将接受任意数量的参数（`args`）。当调用时，它将调用`f(g(args))`。在第二次迭代中，它将返回一个接受任意数量参数并返回`f(g(h(args))`的函数。在这一点上，没有更多的函数可以迭代，因此接受一组参数并返回`f(g(h(arguments)))`的函数的最终输出是`composeManyReduce`函数的输出。

在实现了`composeManyReduce`之后，相应的失败断言现在通过了：

![图 10.29：实现 compose3、composeManyUnary 和 composeManyReduce](img/C14587_10_29.jpg)

###### 图 10.29：实现 compose3、composeManyUnary 和 composeManyReduce

### 在现实世界中使用简单的 BFF 进行函数组合

BFF 是一个服务器端组件，以特定于其服务的用户界面的方式包装（API）功能。这与设计用于导出通用业务逻辑的 API 相对。前端后端可能会消耗上游 API 或直接使用后端服务，这取决于架构。公司可能有一组核心服务来实现业务逻辑，然后为其移动应用程序创建一个 BFF，为其 Web 前端创建另一个 BFF，并为其内部仪表板创建最终的 BFF。每个 BFF 都将具有不同的约束和数据形状，这对于它们各自的消费者来说是最合理的。

通用 API 往往具有更大的表面积，由不同的团队维护，并且有多个消费者，这反过来导致 API 的形状演变缓慢。API 端点不特定于用户界面，因此前端应用程序可能必须进行大量的 API 请求才能加载单个屏幕。前端后端可以缓解这些问题，因为每个页面或屏幕可以有自己的端点或数据集。前端后端将协调获取任何相关数据。

为了实现前端后端，将使用`micro`。micro 是一个用于“异步 HTTP 微服务”的库，由 Zeit 构建。与 Express 或 Hapi 相比，它非常小。为了做到这一点，它利用了现代 JavaScript 特性，如 async/await 调用，其组合模型基于函数组合。也就是说，在 Express 或 Hapi 中的中间件是一个以函数作为参数并返回一个新函数的高阶函数。这是一个很好的使用`compose`的机会，因为被组合的函数的接口是以函数作为参数和以函数作为返回值。

#### 注意

可以在[`github.com/zeit/micro`](https://github.com/zeit/micro)找到 micro 的非常简要的文档。该库本身只有几百行 JavaScript 代码。

一个 micro 的“Hello world”可能如下所示。micro 接受一个可以是异步的 HTTP 处理程序函数。无论哪种方式，都会被等待。它没有内置路由器，这是 Express 或 Hapi 公开的核心 API 之一。处理程序的输出作为 HTTP 响应主体发送回去，状态码为 200：

```js
const micro = require('micro');
const server = micro(async () => {
  return '<p>Hello micro!</p>Run this with <code>node example-2-micro-hello.js</code>';
});
server.listen(3000, () => {
  console.log('Listening on http://localhost:3000');
});
```

可以使用内置的 JavaScript `console.time`和`console.timeEnd`函数来添加请求计时器日志：

```js
// handler and server.listen are unchanged
const timer = fn => async (req, res) => {
  console.time('request');
  const value = await fn(req, res);
  console.timeEnd('request');
  return value;
};
const server = micro(timer(hello));
```

函数组合是前端，而 micro 的中心是 API。添加诸如 API 密钥身份验证之类的更复杂操作并不会使集成变得更加困难。

`authenticate`函数可以具有任意复杂性。如果它接受一个函数参数并返回一个接受`req`（请求）和`res`（响应）对象的函数，它将与其他 micro 包和处理程序兼容：

```js
// handler, timer and server.listen are unchanged
const ALLOWED_API_KEYS = new Set(['api-key-1', 'key-2-for-api']);
const authenticate = fn => async (req, res) => {
  const {authorization} = req.headers;
  if (authorization && authorization.startsWith('ApiKey')) {
    const apiKey = authorization.replace('ApiKey', '').trim();
    if (ALLOWED_API_KEYS.has(apiKey)) {
      return fn(req, res);
    }
  }
  return sendError(
    req,
    res,
    createError(401, `Unauthorizsed: ${responseText}`)
  );
};
const server = micro(timer(authenticate(handler)));
```

micro 库利用函数组合，以便使每个请求处理级别之间的依赖关系变得明显。

### 练习 78：利用 Compose 简化微服务器创建步骤

在这个练习中，您将重构前一节中的计时器和身份验证示例，以使用`compose`。

#### 注意

练习 78 带有预配置的服务器和在起始文件中的 run 方法别名，即`exercise-micro-compose-start.js`。可以使用`npm run Exercise78`运行服务器。可以在 GitHub 上使用 npm run Exercise78 文件运行此练习的工作解决方案。

执行以下步骤完成此练习：

1.  将当前目录更改为`Lesson10`，如果之前没有在此目录中这样做，请运行`npm install`。

1.  首先运行`node exercise-micro-compose-start.js`命令。然后运行`npm run Exercise78`。您将看到应用程序启动，如下所示：![图 10.30：运行此练习的 start 文件](img/C14587_10_30.jpg)

###### 图 10.30：运行此练习的 start 文件

1.  使用以下`curl`访问应用程序应该会产生未经授权的响应：

```js
curl http://localhost:3000
```

以下是前面代码的输出：

![图 10.31：微应用的 cURL](img/C14587_10_31.jpg)

###### 图 10.31：微应用的 cURL

请注意，compose 函数在此模块中预先填充。

1.  我们将使用 compose 而不是在上一个函数的输出上调用每个函数，并调用其输出来创建服务器。这将替换服务器创建步骤：

```js
const server = compose(
  micro,
  timer,
  authenticate,
  handler
)();
```

最初的服务器创建步骤如下，这相当冗长，可能难以阅读。`compose`版本清楚地显示了请求将经过的管道：

```js
const server = micro(timer(authenticate(handler)));
```

1.  重新启动应用程序以使更改生效。一旦`npm run Exercise78`运行起来，您应该能够`curl`：

```js
curl http://localhost:3000
```

以下是前面代码的输出：

![图 10.32：使用“compose”的微应用的 cURL](img/C14587_10_32.jpg)

###### 图 10.32：使用 compose 的微应用的 cURL

在这个练习中，我们看到`compose`的重构并没有影响应用程序的功能。可以根据响应尝试不同的请求。

可以使用以下代码解决上述问题：

```js
curl http://localhost:3000 -H 'Authorization: ApiKey api-key-1' -I
```

以下请求将因为我们没有设置有效的授权头而失败 401 错误：

```js
curl http://localhost:3000 -H 'Authorization: ApiKey bad-key' -I
curl http://localhost:3000 -H 'Authorization: Bearer bearer-token' -I
```

为了比较，这里是使用 Express 及其基于中间件的组合模型的等效 BFF 应用程序。它实现了与我们完成此练习的微 BFF 类似的功能：

```js
const express = require('express');
const app = express();
const responseText = `Hello authenticated Express!`;
const timerStart = (req, res, next) => {
  const timerName = `request_${(Math.random() * 100).toFixed(2)}`;
  console.time(timerName);
  req.on('end', () => {
    console.timeEnd(timerName);
  });
  next();
};
const ALLOWED_API_KEYS = new Set(['api-key-1', 'key-2-for-api']);
const authenticate = (req, res, next) => {
  const {authorization} = req.headers;
  if (authorization && authorization.startsWith('ApiKey')) {
    const apiKey = authorization.replace('ApiKey', '').trim();
    if (ALLOWED_API_KEYS.has(apiKey)) {
      return next();
    }
  }
  return res.status(401).send(`Unauthorized: <pre>${responseText}</pre>`);
};
const requestHandler = (req, res) => {  return res.send(responseText);
};
app.use(timerStart, authenticate, requestHandler);
app.listen(3000, () => {
  console.log('Listening on http://localhost:3000');
});
```

了解函数组合带来的可能性将意味着更多的反思进入函数接口（输入和输出）的设计，以便例如可以利用`compose`。下一节涵盖了不可变性和副作用，这是必要的，以便我们可以组合一组部分应用或纯函数。

## 不可变性和副作用

在纯函数的上下文中，变量的突变被认为是副作用，因此发生变异的函数，特别是超出函数执行范围的变量，不是纯的。

在 JavaScript 中，不可变性很难强制执行，但语言为我们提供了良好的原语，以不可变的方式编写。这种风格严重依赖于操作符和函数，它们创建数据的副本，而不是就地突变。

可以在不使用副作用的情况下编写应用程序的整个部分。任何数据操作都可以在没有副作用的情况下进行。然而，大多数应用程序需要加载数据，以便从某个地方显示数据，并可能在某个地方保存一些数据。这些都是需要管理的副作用。

### Redux 动作创建者的一瞥

动作创建者创建 Redux 动作。它们对于抽象常量并集中 Redux 存储支持的动作非常有用。

动作创建器总是返回一个新的动作对象。创建并返回一个新对象是保证返回值的不可变性的一种好方法，至少就动作创建器而言是这样。如果动作创建器返回其参数的某个版本，可能会产生令人惊讶的输出：

```js
const ADD_PRODUCT = 'ADD_PRODUCT';
function addProduct(newProduct) {
  return {
    type: ADD_PRODUCT,
    newProduct
  };
}
```

可以调用`dispatch`并手动编组对象，也可以调用动作创建器的输出：

```js
this.props.dispatch(addProduct(newProduct))
```

### 练习 79：重构 React/Redux 应用以使用动作创建器

动作创建器是将动作形状与 React 组件分离的好方法。

#### 注意

练习 79 带有一个预配置的开发服务器和起始文件中方法的骨架，即`exercise--refactor-action-creators-start.js`和`exercise-refactor-action-creators-start.html`。可以使用`npm run Exercise79`来运行开发服务器。可以在 GitHub 上使用`npm run exercise10`文件来运行这个练习的工作解决方案。

在这个练习中，您将从使用内联动作定义转为使用动作创建器。

执行以下步骤完成此练习：

1.  将当前目录更改为`Lesson10`并运行`npm install`，如果您之前没有在此目录中执行过。`npm install`会下载运行此活动所需的依赖项（React、Redux、react-redux 和 Parcel）。

1.  首先运行`npx parcel serve exercise-refactor-action-creators-start.html`。要在开发过程中查看应用程序，请运行`npm run Exercise79`。您将看到应用程序正在启动，如下所示：![图 10.33：运行此练习的起始文件](img/C14587_10_33.jpg)

###### 图 10.33：运行此练习的起始文件

为了使开发服务器实时重新加载我们的更改并避免配置问题，请直接编辑`exercise-refactor-action-creators-start.js`文件。

1.  转到`http://localhost:1234`（或者起始脚本输出的任何 URL）。您应该看到以下 HTML 页面：![图 10.34：浏览器中的初始应用](img/C14587_10_34.jpg)

###### 图 10.34：浏览器中的初始应用

1.  实现`startCheckout`、`continueShopping`、`done`和`addProduct`动作创建器：

```js
function startCheckout(items) {
  return {
    type: START_CHECKOUT,
    basket: {
      items
    }
  };
}
function continueShopping() {
  return {
    type: CONTINUE_SHOPPING
  };
}
function done() {
  return {
    type: DONE
  };
}
function addProduct(newProduct) {
  return {
    type: ADD_PRODUCT,
    newProduct: {
      ...newProduct,
      quantity: 1
    }
  };
}
```

这些分别返回以下动作类型：`START_CHECKOUT`、`CONTINUE_SHOPPING`、`DONE`和`ADD_PRODUCT`。

1.  更新`handleCheckout`以使用相应的`startCheckout`动作创建器：

```js
handleCheckout(items) {
  this.props.dispatch(startCheckout(items));
}
```

1.  更新`continueShopping`以使用相应的`continueShopping`动作创建器：

```js
continueShopping() {
  this.props.dispatch(continueShopping());
}
```

1.  更新`finish`以使用相应的`done`动作创建器：

```js
finish() {
  this.props.dispatch(done());
}
```

1.  更新`addProduct`以使用相应的`addProduct`动作创建器：

```js
addProduct(product) {
  this.props.dispatch(addProduct(product));
}
```

1.  检查应用程序是否仍然按预期运行：

![图 10.35：重构动作创建器后的应用](img/C14587_10_35.jpg)

###### 图 10.35：重构动作创建器后的应用

### React-Redux mapStateToProps 和 mapDispatchToProps

react-redux 的核心命题是 connect 函数，正如其名称所示，它将组件连接到存储。它的签名是`connect(mapStateToProps, mapDispatchToProps) (component)`，并返回一个`connect`组件。

在大多数示例中，`mapStateToProps`函数都是`stated => state`，这在一个小应用程序中是有意义的，因为所有状态都与连接的组件相关。原则上，应该在`mapStateToProps`中使用选择器，以避免传递太多的 props，因此当它不使用的数据发生变化时，组件重新渲染。以下是`mapStateToProps`函数的一个小例子：

```js
const mapStateToProps = state => {
  return {
    items: selectBasketItems(state),
    status: selectStatus(state),
    product: selectProduct(state)
  };
};
```

让我们使用`mapStateToProps`完成一个练习。

### 练习 80：使用 mapDispatchToProps 函数抽象状态管理

在这个练习中，您将使用`mapDispatchToProps`函数来管理状态，该函数利用选择器来抽象 redux 存储的内部形状。

执行以下步骤完成此练习：

1.  如果您之前没有在此目录中执行过`npm install`，请将当前目录更改为`Lesson10`并运行`npm install`。

1.  首先，运行`npx parcel serve exercise-map-to-props-start.html`。 然后，在开发过程中，运行`npm run Exercise80`。 您将看到应用程序启动，如下所示：

#### 注意

Exercise 80 带有预配置的开发服务器和起始文件中方法的框架，即`exercise-map-to-props-start.js`和`exercise-map-to-props-start.html`。 可以使用`npm run Exercise80`运行开发服务器。 可以在 GitHub 上使用 npm run Exercise80 文件运行此练习的工作解决方案。

![图 10.36：npm run Exercise80 的输出](img/C14587_10_36.jpg)

###### 图 10.36：npm run Exercise80 的输出

1.  转到`http://localhost:1234`（或起始脚本输出的任何 URL）。 您应该看到一个空白的 HTML 页面。 这是因为`mapStateToProps`返回了一个空状态对象。

#### 注意

审核解释了 App 组件使用的状态片段（来自存储）以及产品，项目和状态是正在使用的状态片段。

1.  为`status`创建一个新的选择器：

```js
const selectStatus = state => state && state.status;
```

1.  为`product`创建一个新的选择器：

```js
const selectProduct = state => state && state.product;
```

1.  在`mapStateToProps`中，将`items`，`product`和`status`映射到它们对应的选择器，这些选择器应用于状态：

```js
const mapStateToProps = state => {
  return {
    items: selectBasketItems(state),
    status: selectStatus(state),
    product: selectProduct(state)
  };
};
```

1.  将在 App 组件中调用`dispatch`的函数提取到`mapDispatchToProps`中，注意从`this.props.dispatch`中删除`this.props`。 Dispatch 是`mapDispatchToProps`的第一个参数。 我们的代码现在应该如下所示：

```js
const mapDispatchToProps = dispatch => {
  return {
    handleCheckout(items) {
      dispatch(startCheckout(items))
    },
    continueShopping() {
      dispatch(continueShopping());
    },
    finish() {
      dispatch(done());
    },
    addProduct(product) {
      dispatch(addProduct(product));
    }
  };
};
```

1.  替换`App#render`中对`this.handleCheckout`的引用。 相反，调用`this.props.handleCheckout`：

```js
{status === 'SHOPPING' && (
  <Basket
    items={items}
    renderItem={item => (
      <div>
        x{item.quantity} - {item.name} - $
        {(item.price / 100).toFixed(2)} each{' '}
      </div>
    )}
    onCheckout={this.props.handleCheckout}
    />
)}
```

1.  替换`App#render`中对`this.continueShopping`和`this.finish`的引用。 相反，分别调用`this.props.continueShopping`和`this.props.finish`：

```js
{status === 'CHECKING_OUT' && (
  <div>
    <p>You have started checking out with {items.length} items.</p>
    <button onClick={this.props.continueShopping}>
      Continue shopping
    </button>
    <button onClick={this.props.finish}>Finish</button>
  </div>
)}
```

1.  替换`App#render`中对`this.addProduct`的引用。 相反，调用`this.props.addProduct`：

```js
{status === 'SHOPPING' && (
  <div style={{marginTop: 50}}>
    <h2>{product.name}</h2>
    <p>Price: ${product.price / 100}</p>
    <button onClick={() => this.props.addProduct(product)}>
      Add to Basket
    </button>
  </div>
)}
```

1.  打开`http://localhost:1234`，查看应用程序现在的预期行为。 您可以添加产品，转到结账，完成或继续购物：

![图 10.37：mapStateToProps/mapDispatchToProps 重构后的应用程序](img/C14587_10_37.jpg)

###### 图 10.37：mapStateToProps/mapDispatchToProps 重构后的应用程序

该应用程序现在使用正确实现的`mapStateToProps`和`mapDispatchToProps`函数工作。 React 和 Redux 进一步从彼此抽象出来。 React 组件中不再有状态，也不再直接调用存储的`dispatch`方法。 这意味着原则上，可以使用另一个状态管理库来替换 Redux，而 React 代码不会改变； 只有状态管理器和 React`App`组件之间的粘合代码会改变。

### Redux Reducers In Depth

Redux 减速器不应该改变 Redux 存储状态。 与第一原则相比，纯函数更容易测试，其结果更容易预测。 作为状态管理解决方案，Redux 有两个作用：保持状态如预期，并确保更新能够高效和及时地传播。

纯函数可以帮助我们通过考虑不可变性来实现这一目标。 返回副本有助于进行更改检测。 例如，检测对象内的大部分键已更新的成本更高，而检测对象已被其浅复制替换的成本更低。 在第一种情况下，必须进行昂贵的深度比较，以遍历整个对象以检测原始值和/或结构的差异。 在浅复制情况下，仅需要检测对象引用不同即可检测到更改。 这是微不足道的，与`===` JavaScript 运算符有关，该运算符通过引用比较对象。

### 将 JavaScript-Native 方法更改为不可变的函数式样式

Map/filter/reduce 不会改变它们操作的初始数组。在以下片段中，`initial`的值保持不变。`Array#map`返回数组的副本，因此不会改变它正在操作的数组。`Array#reduce`和`Array#filter`也是如此；它们都用于数组，但不会在原地更改任何值。相反，它们会创建新的对象：

```js
// Node.js built-in
const assert = require('assert').strict;
const initial = [
  {
    count: 1,
    name: 'Shampoo'
  },
  {
    count: 2,
    name: 'Soap'
  }
];
assert.deepStrictEqual(
  initial.map(item => item.name),
  ['Shampoo', 'Soap'],
  'demo map'
);
assert(
  initial.map(item => item.count).reduce((acc, curr) => acc + curr) === 3,
  'demo map reduce'
);
assert.deepStrictEqual(
  initial.filter(item => item.count > 1),
  [{count: 2, name: 'Soap'}],
  'demo filter'
);
```

对象的`rest`和`spread`语法是 ECMAScript 2018 中引入的，是创建对象的浅拷贝和排除/覆盖键的好方法。以下代码结合了`Array#map`和对象 rest/spread 来创建数组的浅拷贝（使用`Array#map`），但也使用 rest/spread 来创建数组中对象的浅拷贝：

```js
// Node.js built-in
const assert = require('assert').strict;
const initial = [
  {
    count: 1,
    name: 'Shampoo'
  },
  {
    count: 2,
    name: 'Soap'
  }
];
assert.deepStrictEqual(
  initial.map(item => {
    return {
      ...item,
      category: 'care'
    };
  }),
  [
    {
      category: 'care',
      count: 1,
      name: 'Shampoo'
    },
    {
      category: 'care',
      count: 2,
      name: 'Soap'
    }
  ],
  'demo of spread (creates copies)'
);
assert.deepStrictEqual(
  initial.map(({name, ...rest}) => {
    return {
      ...rest,
      name: `${name.toLowerCase()}-care`
    };
  }),
  [
    {
      count: 1,
      name: 'shampoo-care'
    },
    {
      count: 2,
      name: 'soap-care'
    }
  ],
  'demo of rest in parameter + spread'
);
```

数组的`rest`和`spread`语法早于对象的 spread/rest，因为它是 ECMAScript 2015（也称为 ES6）的一部分。与其对象对应物一样，它非常有用于创建浅拷贝。我们已经看到的另一个用例是将类似数组的对象转换为完整的数组。相同的技巧也可以用于可迭代对象，如`Set`。

在以下示例中，使用数组 spread 来创建数组的副本，然后对其进行排序，并使用它将 Set 转换为数组。数组 spread 还用于创建除第一个元素之外的所有元素的副本：

```js
// Node.js built-in
const assert = require('assert').strict;
const initial = [
  {
    count: 1,
    name: 'Shampoo'
  },
  {
    count: 2,
    name: 'Soap'
  }
];
assert.deepStrictEqual(
  // Without the spread, reverse() mutates the array in-place
  [...initial].reverse(),
  [
    {
      count: 2,
      name: 'Soap'
    },
    {
      count: 1,
      name: 'Shampoo'
    }
  ],
  'demo of immutable reverse'
);
assert.deepStrictEqual(
  [...new Set([1, 2, 1, 2])],
  [1, 2],
  'demo of spread on Sets'
);
const [first, ...rest] = initial;
assert.deepStrictEqual(first, {count: 1, name: 'Shampoo'});
assert.deepStrictEqual(rest, [
  {
    count: 2,
    name: 'Soap'
  }
]);
```

`Object.freeze`使对象在严格模式下变为只读。

例如，以下代码片段将使用 throw，因为我们试图在严格模式下向冻结的对象添加属性：

```js
// Node.js built-in
const assert = require('assert').strict;
const myProduct = Object.freeze({
  name: 'Special Sauce',
  price: 1999
});
assert.throws(() => {
  'use strict';
  myProduct.category = 'condiments';
}, 'writing to an existing property is an error in strict mode');
assert.throws(() => {
  'use strict';
  myProduct.name = 'Super Special Sauce';
}, 'writing a new property is an error in strict mode');
```

`Object.freeze`在实践中很少使用。作为一种设计用于在浏览器中运行的语言，JavaScript 被设计为非常宽松。运行时错误存在，但应该避免，特别是对于绑定为应用程序问题的事情：写入只读属性。

而且，`Object.freeze`只在非严格模式下抛出。看看以下示例，其中允许在冻结对象上访问和修改属性，因为默认情况下 JavaScript 在非严格模式下运行：

```js
// Node.js built-in
const assert = require('assert').strict;
const myProduct = Object.freeze({
  name: 'Special Sauce',
  price: 1999
});
assert.doesNotThrow(() => {
  myProduct.category = 'condiments';
}, 'writing to an existing property is fine in non-strict mode');
assert.doesNotThrow(() => {
  myProduct.name = 'Super Special Sauce';
}, 'writing a new property is fine in non-strict mode');
```

工程团队通常选择遵循支持不可变风格的编码标准，而不是强制不可变性。

#### 注意

还可以利用诸如 Immutable.js 之类的库，该库提供了以高效方式实现的持久不可变数据结构。

### 在 React/Redux 应用程序中处理副作用 React 生命周期钩子

React 组件的`render()`方法应该是纯的，因此不支持副作用。能够根据其输入（props 和 state）来预测组件是否需要重新渲染意味着可以避免很多不必要的更新。由于每次状态或属性更新都可能导致调用`render`，这可能不是放置 API 调用的最佳位置。

React 文档建议使用`componentDidMount`生命周期方法。`componentDidMount`在组件挂载后运行。换句话说，如果在 React 应用程序的先前状态中未渲染组件，则它在页面上第一次渲染时运行。

我们可以使用`componentDidMount`发送带有`fetch`的 HTTP 请求。`fetch` Promise 的`.then`可以用于从服务器响应中更新状态：

```js
import React from 'react';
class App extends React.Component {
  constructor() {
    super();
    this.state = {};
  }
  componentDidMount() {
    fetch('https://hello-world-micro.glitch.me')
      .then(response => {
        if (response.ok) {
          return response.text();
        }
      })
      .then(data => {
        this.setState({
          message: data
        });
      });
  }
  render() {
    return (
      <div>
        <p>Message: {this.state.message}</p>
      </div>
    );
  }
}
```

### 在 React/Redux 应用程序中处理副作用 React Hooks

作为 React 的最新添加，钩子允许函数组件利用以前专门用于类组件的所有功能。

前面的示例可以重构为使用`useState`和`useEffect`钩子的函数组件。`useState`是我们可以使用钩子在 React 函数组件中使用状态的一种方式。当来自`useState`的状态发生变化时，React 将重新渲染函数组件。`useEffect`是`componentDidMount`的对应物，在组件渲染之前调用，如果组件不是在应用程序的先前状态中渲染的：

```js
import React, {useEffect, useState} from 'react';
const App = () => {
  const [message, setMessage] = useState(null);
  useEffect(() => {
    if (!message) {
      fetch('https://hello-world-micro.glitch.me')
        .then(response => {
          if (response.ok) {
            return response.text();
          }
        })
        .then(data => {
          setMessage(data);
        });
    }
  });
  return (
    <div>
      <p>Message: {message}</p>
    </div>
  );
};
```

### 在 React/Redux 应用程序中处理副作用 Redux-Thunk

Thunk 是延迟评估函数的一种方式。这是一种在不支持它的语言中进行惰性评估的方法：

```js
let globalState;
function thunk() {
  return () => {
    globalState = 'updated';
  };
}
const lazy = thunk();
console.assert(!globalState, 'executing the thunk does nothing');
lazy();
console.assert(
  globalState === 'updated',
  'executing the output of the thunk runs the update'
);
```

这也是一种封装副作用的方式。由于我们有一流函数，我们传递 thunk，这在纯函数中是允许的（thunk 只是一个函数），尽管调用 thunk 本身可能会有副作用。

redux-thunk 非常简单；而不是传递返回对象的操作创建者（带有类型字段和可能的有效负载），操作创建者返回一个接受存储的分派和`getState`作为参数的函数。

在 thunk 中，可以访问当前存储状态并分派操作，这些操作将被减少到存储中。请参见以下使用 Redux 和 redux-thunk 的示例：

```js
// store is set up, App is connected, redux-thunk middleware is applied
import React from 'react';
class App extends React.Component {
  componentDidMount() {
    // this looks like any action creator
    this.props.dispatch(requestHelloWorld());
  }
  render() {
    return (
      <div>
        <p>Message: {this.props.message}</p>
      </div>
    );
  }
}
function requestHelloWorld() {
  // this action creator returns a function
  return (dispatch, getState) => {
    fetch('https://hello-world-micro.glitch.me')
      .then(response => {
        if (response.ok) {
          return response.text();
        }
      })
      .then(data => {
        dispatch({
          type: 'REQUEST_HELLO_WORLD_SUCCESS',
          message: data
        });
      })
      .catch(error => {
        dispatch({
          type: 'REQUEST_HELLO_WORLD_ERROR',
          error
        });
      });
  };
}
```

## GraphQL 语言模式和查询简介

GraphQL 是一种查询语言。它公开了一个类型化的模式来运行查询。GraphQL 的巨大好处是客户端请求所需的信息。这是有类型模式的直接效果。

我们将使用`express-graphql`将 GraphQL 添加到我们的 BFF，它与 micro 兼容。我们需要为我们的 GraphQL 端点提供模式和解析器，以便它可以响应客户端请求。在 Exercise 12 开始文件中提供了这样的服务器（将工作目录更改为`Lesson10`，运行`npm install`，然后运行`npm run Exercise81`，并转到`http://localhost:3000`以查看其运行情况）。

一个返回篮子的示例 GraphQL 查询可以在以下 GraphQL 模式定义中工作。请注意我们有三种类型，即`Query`、`basket`和`basketItem`。`basket`在`items`属性下包含`basketItems`列表。`query`包含顶级 GraphQL 查询字段，这种情况下只是`basket`。要查询`basketItems`，我们必须加载相应的`basket`并展开`items`字段：

```js
type basket {
  items: [basketItem]
}
"""BasketItem"""
type basketItem {
  name: String
  price: Int
  quantity: Int
  id: String
}
"""Root query"""
type Query {
  basket: basket
}
```

Node.js GraphQL 服务器组件中内置的工具是 GraphiQL。它是一个用于 GraphQL 的接口，允许用户浏览模式并提供模式的文档。

我们输入的查询如下：加载`basket`顶级查询字段，展开其`items`字段，并填充篮子`items`字段中`basketItem`元素的`name`、`quantity`和`price`：

![图 10.38：GraphiQL 用户界面和获取完全展开的篮子项目](img/C14587_10_38.jpg)

###### 图 10.38：GraphiQL 用户界面和获取完全展开的篮子项目

### 使用 GraphQL 变异和解析器进行运行更新

在查询和模式世界中，一个非常缺少的东西是运行写操作的方法。这就是 GraphQL 变异的作用。变异是结构化的更新操作。

解析器是服务器端 GraphQL 实现的细节。解析器是解析 GraphQL 查询的东西。解析器从模式链的顶部到底部运行。在解析查询时，对象上的字段并行执行；在解析变异时，它们按顺序解析。以下是使用变异的示例：

```js
const mutation = new GraphQLObjectType({
  name: 'Mutation',
  fields() {
    return {};
  }
});
```

#### 注意

有关 GraphQL 的更多指南，请访问[`graphql.org`](https://graphql.org)。

### 练习 81：使用 micro 和 GraphQL 实现 BFF 变异

在这个练习中，我们将使用 micro 和 GraphQL 来实现 BFF 变异。

执行以下步骤以完成此练习：

#### 注意

练习 81 包括一个预配置的服务器和起始文件`exercise-graphql-micro-start.js`中方法的框架。可以使用`npm run Exercise81`运行开发服务器。可以在 GitHub 上使用`npm run Exercise81`文件运行此练习的工作解决方案。

1.  如果您以前没有在此目录中执行过此操作，请将当前目录更改为`Lesson10`并运行`npm install`。`npm install`下载所需的依赖项，以便我们可以运行此活动（micro 和`express-graphql`）。

1.  运行`node exercise-graphql-micro-start.js`。然后，在开发过程中，运行`npm run Exercise81`。您将看到应用程序启动，如下所示：![图 10.39：运行此练习的起始文件](img/C14587_10_39.jpg)

###### 图 10.39：运行此练习的起始文件

1.  转到`http://localhost:3000`（或者起始脚本输出的任何 URL）。您应该看到以下 GraphiQL 页面：![图 10.40：空的 GraphiQL 用户界面](img/C14587_10_40.jpg)

###### 图 10.40：空的 GraphiQL 用户界面

1.  添加一个`LineItemCost`常量，它是一个字段定义（普通的 JavaScript 对象）：

```js
const LineItemCost:
= {
  type: GraphQLInt,
  args: {id: {type: GraphQLString}},
  resolve(root, args, context) {
    return 4;
  }
};
```

我们的`LineItemCost`应该有一个`type`属性设置为`GraphQLInt`，因为`LineItemCost`计算的输出是一个整数。`LineItemCost`还应该有一个`args`字段，应该设置为`{id: {type: GraphQLString}}`。换句话说，我们的 mutation 接受一个是字符串的`id`参数（这与我们的示例数据一致）。为了使 mutation 返回一些东西，它需要一个`resolve()`方法。目前，它可以返回任何整数。mutations 的`resolve`方法将根作为第一个参数，`args`作为第二个参数。

1.  现在让我们实现`LineItemCost`的实际`resolve`方法。首先，我们需要使用`.find(el => el.id === args.id)`从`basketItems`中查找 ID 查找项目。然后，我们可以计算项目的成本（`item.price * item.quantity`），如下所示：

```js
const LineItemCost = {
  type: GraphQLInt,
  args: {id: {type: GraphQLString}},
  resolve(root, args, context) {
    const item = basketItems.find(i => i.id === args.id);
    return item ? item.quantity * item.price : null;
  }
};
```

1.  创建一个`GraphQLObjectType`的 mutation 常量。查看查询如何初始化；其名称应为`Mutation:`

```js
const mutation = new GraphQLObjectType({
  name: 'Mutation',
  fields() {
    return {};
  }
});
```

1.  将`LineItemCost`添加到`fields()`返回值的 mutation 中。这意味着`LineItemCost`现在是顶级 mutation。如果在 GraphQL 模式上存在`mutation`，则可以调用它：

```js
const mutation = new GraphQLObjectType({
  name: 'Mutation',
  fields() {
    return {LineItemCost};
  }
});
```

1.  将`mutation`添加到`GraphQLSchema`模式中：

```js
const handler = graphqlServer({
  schema: new GraphQLSchema({query, mutation}),
  graphiql: true
});
```

1.  将以下查询发送到服务器（通过 GraphiQL）。在左侧编辑器中输入并单击**播放**按钮：

```js
mutation {
  cost1: LineItemCost(id: "1")
  cost2: LineItemCost(id: "2")
}
```

#### 注意

这个 mutation 使用了所谓的 GraphQL 别名，因为我们不能两次使用相同的名称运行 mutation。

输出应该如下所示：

![图 10.41：带有 LineItemCost 别名的 ID“1”和“2”的 mutation 查询的 GraphiQL](img/C14587_10_41.jpg)

###### 图 10.41：带有 LineItemCost 别名的 ID“1”和“2”的 mutation 查询的 GraphiQL

为了使购物篮示例更加真实，我们将使用 GraphQL 查询，redux-thunk 来处理副作用，并使用新的 reducer 来更新 Redux 存储状态，从 GraphQL BFF 加载初始购物篮数据。下一个活动的目的是向您展示我们如何将 GraphQL BFF 与使用 redux-thunk 的 React/Redux 应用集成。

### 活动 17：从 BFF 获取当前购物篮

在这个活动中，我们将从 GraphQL BFF 获取初始购物篮数据，以便重新渲染物品到购物篮，从而更新购物篮的初始状态。让我们开始吧：

#### 注意

活动 17 配备了一个预配置的开发服务器和起始文件中方法的骨架，即`activity-app-start.js`和`activity-app-start.html`。可以使用`npm run Activity17`运行开发服务器。可以在 GitHub 上使用 npm run Activity17 文件运行此活动的工作解决方案。

1.  如果以前没有在此目录中这样做，请将当前目录更改为`Lesson10`并运行`npm install`。

1.  运行活动 17 的 BFF 和`npx parcel serve activity-app-start.html`。在开发过程中，运行`npm run Activity17`。

1.  转到`http://localhost:1234`（或者起始脚本输出的任何 URL）以检查 HTML 页面。

1.  编写一个查询，从 BFF 获取购物篮中的物品。您可以使用`http://localhost:3000`上的 GraphQL UI 进行实验。

1.  创建一个`requestBasket`（thunk）动作创建器，它将使用上一步的查询调用`fetchFromBff`。

1.  在`fetchFromBff()`调用上链接一个`.then`，以使用正确的`basket`有效负载分派`REQUEST_BASKET_SUCCESS`动作。

1.  向`appReducer`添加一个案例，将带有`basket`有效负载的`REQUEST_BASKET_SUCCESS`操作减少到状态中。

1.  将`requestBasket`添加到`mapDispatchToProps`。

1.  在`App#componentDidMount`中调用`requestBasket`，它被映射到`dispatch`。

#### 注意

此活动的解决方案可在第 628 页找到。

## 摘要

头等函数是使用流行库（如 React 及其模式）的一部分。它们还为任何实现的委托提供动力，特别是在内置功能（如 Array）上。函数式编程的另一个核心原则是纯函数。使用纯函数进行复杂数据操作逻辑或数据结构周围的抽象层是由流行的 Redux 状态管理解决方案提出的一个很好的模式。任何必须被模拟的副作用和/或依赖关系都会使复杂数据操作变得更加难以理解。高阶函数和特定技术，如柯里化和部分应用，在日常 JavaScript 开发中广泛使用。柯里化和部分应用是一种设计函数的方式，使得每个专门化步骤都是“可保存的”，因为它已经是一个已经应用了一定数量参数的函数。

如果发现函数应用程序管道，则组合可以具有真正的价值。例如，将 HTTP 服务建模为管道非常有意义。另一方面，Node.js HTTP 服务器生态系统领导者使用基于中间件的组合模型，micro 暴露了一个函数组合模型。以不可变的方式编写 JavaScript 允许库以廉价的方式检查某些东西是否已更改。在 React 和 Redux 中，副作用是在纯函数的常规流程之外处理的，即渲染函数和 reducer。Redux-thunk 是解决这个问题的一个相当功能性的解决方案，尽管以使函数有效的行动为代价。纯 Redux 操作是具有类型属性的 JavaScript 对象。

在本书中，我们学习了包括 React、Angular 和相关工具和库在内的各种框架。它教会了我们构建现代应用程序所需的高级概念。然后，我们学习了如何在文档对象模型（DOM）中表示 HTML 文档。之后，我们结合了对 DOM 和 Node.js 的知识，为实际情况创建了一个网络爬虫。

在接下来的部分中，我们使用 Express 库为 Node.js 创建了基于 Node.js 的 RESTful API。我们看了看如何使用模块化设计来实现更好的可重用性，并与单个项目上的多个开发人员进行协作。我们还学习了如何构建单元测试，以确保我们程序的核心功能随着时间的推移不会出现问题。我们看到构造函数、async/await 和事件如何可以使我们的应用程序具有高速和性能。本书的最后部分向您介绍了函数式编程概念，如不可变性、纯函数和高阶函数。

# *附录*

## 关于

本节旨在帮助学生完成书中的活动。它包括学生执行的详细步骤，以实现活动的目标。

## 第一章：JavaScript、HTML 和 DOM

### 活动 1：从页面中提取数据

**解决方案**：

1.  初始化一个变量来存储 CSV 的整个内容：

```js
var csv = 'name,price,unit\n';
```

1.  查询 DOM 以查找表示每个产品的所有元素。注意我们如何将返回的`HTMLCollection`实例包装在`Array.from`中，以便我们可以像处理普通数组一样处理它：

```js
var elements = Array.from(document.getElementsByClassName('item'));
```

1.  遍历找到的每个元素：

```js
elements.forEach((el) => {});
```

1.  在闭包内，使用`product`元素，查询以找到带单位的价格。使用斜杠拆分字符串：

```js
var priceAndUnitElement = el.getElementsByTagName('span')[0];
var priceAndUnit = priceAndUnitElement.textContent.split("/");
var price = priceAndUnit[0].trim();
var unit = priceAndUnit[1].trim();
```

1.  然后查询名称：

```js
var name = el.getElementsByTagName('a')[0].textContent;
```

1.  将所有信息附加到步骤 1 中初始化的变量中，使用逗号分隔值。不要忘记为附加到每行的换行符添加：

```js
csv += `${name},${price},${unit}\n`;
```

1.  使用`console.log`函数打印包含累积数据的变量：

```js
console.log(csv);
```

1.  将代码粘贴到 Chrome **控制台**选项卡中；它应该看起来像这样：

![图 1.62：准备在控制台选项卡中运行的代码](img/C14587_01_62.jpg)

###### 图 1.62：准备在控制台选项卡中运行的代码

按下*Enter*执行代码后，您应该在控制台中看到打印的 CSV，如下所示：

![图 1.63：带有代码和控制台选项卡输出的商店](img/C14587_01_63.jpg)

###### 图 1.63：带有代码和控制台选项卡输出的商店

### 活动 2：用 Web 组件替换标签过滤器

**解决方案**：

1.  首先将`Exercise07`中的代码复制到一个新文件夹中。

1.  创建一个名为`tags_holder.js`的新文件，并在其中添加一个名为`TagsHolder`的扩展`HTMLElement`的类，然后定义一个名为`tags-holder`的新自定义组件：

```js
class TagsHolder extends HTMLElement {
}
customElements.define('tags-holder', TagsHolder);
```

1.  创建两个`render`方法：一个用于呈现基本状态，一个用于呈现标签或指示未选择任何标签进行过滤的文本：

```js
render() {
  this.shadowRoot.innerHTML = `
  <link rel="stylesheet" type="text/css" href="../css/semantic.min.css" />
  <div>
    Filtered by tags:
    <span class="tags"></span>
  </div>`;
}
renderTagList() {
  const tagsHolderElement = this.shadowRoot.querySelector('.tags');
  tagsHolderElement.innerHTML = '';
  const tags = this._selectedTags;
  if (tags.length == 0) {
    tagsHolderElement.innerHTML = 'No filters';
    return;
  }
  tags.forEach(tag => {
    const tagEl = document.createElement('span');
    tagEl.className = "ui label orange";
    tagEl.addEventListener('click', () => this.triggerTagClicked(tag));
    tagEl.innerHTML = tag;
    tagsHolderElement.appendChild(tagEl);
  });
}
```

1.  在构造函数中，调用`w`，将组件附加到阴影根，初始化所选标签列表，并调用两个`render`方法：

```js
constructor() {
  super();
  this.attachShadow({ mode: 'open' });
  this._selectedTags = [];
  this.render();
  this.renderTagList();
}
```

1.  创建一个 getter 来公开所选标签的列表：

```js
get selectedTags() {
  return this._selectedTags.slice(0);
}
```

1.  创建两个触发方法：一个用于触发更改事件，另一个用于触发`tag-clicked`事件：

```js
triggerChanged(tag) {
  const event = new CustomEvent('changed', { bubbles: true });
  this.dispatchEvent(event);
}
triggerTagClicked(tag) {
  const event = new CustomEvent('tag-clicked', {
    bubbles: true,
    detail: { tag },
  });
  this.dispatchEvent(event);
}
```

1.  创建两个`mutator`方法：`addTag`和`removeTag`。这些方法接收标签名称，如果不存在，则添加标签，如果存在，则删除标签，从所选标签列表中。如果列表已修改，则触发`changed`事件并调用重新呈现标签列表的方法：

```js
addTag(tag) {
  if (!this._selectedTags.includes(tag)) {
    this._selectedTags.push(tag);
    this._selectedTags.sort();
    this.renderTagList();
    this.triggerChanged();
  }
}
removeTag(tag) {
  const index = this._selectedTags.indexOf(tag);
  if (index >= 0) {
    this._selectedTags.splice(index, 1);
    this.renderTagList();
    this.triggerChanged();
  }
}
```

1.  在 HTML 中，用新组件替换现有代码。删除以下行：

```js
<div class="item">
  Filtered by tags: <span class="tags"></span>
</div>
And add:
<tags-holder class="item"></tags-holder>
Also add:
<script src="tags_holder.js"></script>
```

#### 注意

您可以在 GitHub 上查看最终的 HTML，网址为[`github.com/TrainingByPackt/Professional-JavaScript/blob/master/Lesson01/Activity02/dynamic_storefront.html`](https://github.com/TrainingByPackt/Professional-JavaScript/blob/master/Lesson01/Activity02/dynamic_storefront.html)。

1.  在`filter_and_search.js`中，执行以下操作：

在顶部，创建对`tags-holder`组件的引用：

```js
const filterByTagElement = document.querySelector('tags-holder');
```

添加事件侦听器以处理`changed`和`tag-clicked`事件：

```js
filterByTagElement.addEventListener('tag-clicked', (e) => filterByTagElement.removeTag(e.detail.tag));
filterByTagElement.addEventListener('changed', () => applyFilters());
```

删除以下函数及其所有引用：`createTagFilterLabel`和`updateTagFilterList`。

在`filterByTags`函数中，用`filterByTagElement.selectedTags`替换`tagsToFilterBy`。

在`addTagFilter`方法中，用`filterByTagElement.addTag`替换对`tagsToFilterBy`的引用。

## 第二章：Node.js 和 npm

### 活动 3：创建一个 npm 包来解析 HTML

**解决方案**：

1.  在空文件夹中，使用 npm 创建一个新包。您可以使用所有选项的默认值：

```js
$ npm init
This utility will walk you through creating a package.json file.
It only covers the most common items, and tries to guess sensible defaults.
See 'npm help json' for definitive documentation on these fields and exactly what they do.
Use 'npm install <pkg>' afterwards to install a package and save it as a dependency in the package.json file.
Press ^C at any time to quit.
package name: (Activity03) 
version: (1.0.0) 
description: 
entry point: (index.js) 
test command: 
git repository: 
keywords: 
author: 
license: (ISC) 
About to write to .../Lesson02/Activity03/package.json:
{
  "name": "Activity03",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISCs"
}
Is this OK? (yes)
```

1.  要安装`cheerio`，运行`npm install`。确保您错误地输入库的名称：

```js
$ npm install cheerio
npm notice created a lockfile as package-lock.json. You should commit this file.
npm WARN Activity03@1.0.0 No description
npm WARN Activity03@1.0.0 No repository field.
+ cheerio@1.0.0-rc.3added 19 packages from 45 contributors and audited 34 packages in 6.334s
found 0 vulnerabilities
```

1.  在此文件夹中，创建一个名为`index.js`的文件，并将以下内容添加到其中：

```js
const cheerio = require('cheerio');
```

1.  创建一个变量，存储来自 GitHub 示例代码（[`github.com/TrainingByPackt/Professional-JavaScript/blob/master/Lesson01/Example/sample_001/sample-page.html`](https://github.com/TrainingByPackt/Professional-JavaScript/blob/master/Lesson01/Example/sample_001/sample-page.html)）的 HTML。创建多行字符串时，可以使用反引号：

```js
const html = `
<html>
  <head>
    <title>Sample Page</title>
  </head>
  <body>
    <p>This is a paragraph.</p>
    <div>
      <p>This is a paragraph inside a div.</p>
    </div>
    <button>Click me!</button>
  </body>
</html>
`;
```

1.  解析 HTML 并将其传递给 cheerio。在 cheerio 的示例中，您将看到它们将解析的变量命名为“`$`”（美元符号）。这是 jQuery 世界中使用的一个旧约定。它看起来像这样：

```js
const $ = cheerio.load(html);
```

1.  现在，我们可以使用该变量来操作 HTML。首先，我们将向页面添加一个带有文本的段落：

```js
$('div').append('<p>This is another paragraph.</p>');
```

我们还可以查询 HTML，类似于我们在*第一章 JavaScript、HTML 和 DOM*中所做的，使用 CSS 选择器。让我们查询所有段落并将其内容打印到控制台。请注意，cheerio 元素的行为与 DOM 元素并不完全相同，但它们非常相似。

1.  使用`firstChild`属性找到每个段落的第一个节点并打印其内容，假设它将是文本元素：

```js
$('p').each((index, p) => {
  console.log(`${index} - ${p.firstChild.data}`);
});
```

1.  最后，在`index.js`中，通过调用`html`函数将操作后的 HTML 打印到控制台：

```js
console.log($.html());
```

现在，您可以通过从 Node.js 调用它来运行您的应用程序：

![图 2.7：从 node.js 调用应用程序](img/C14587_02_07.jpg)

###### 图 2.7：从 Node.js 调用应用程序

## 第三章：Node.js API 和 Web 爬虫

### 活动 4：从商店前端爬取产品和价格

**解决方案**

1.  使用本章中*练习 14，提供动态内容*中的代码启动动态服务器以提供商店前端应用程序：

```js
$ node Lesson03/Activity04/
Static resources from /path/to/repo/Lesson03/Activity04/static
Loaded 21 products...
Go to: http://localhost:3000
```

1.  在新的终端中，创建一个新的`npm`包，安装`jsdom`，并创建`index.js`入口文件：

```js
$ npm init
...
$ npm install jsdom
+ jsdom@15.1.1
added 97 packages from 126 contributors and audited 140 packages in 12.278s
found 0 vulnerabilities
```

1.  调用`require()`方法加载项目中需要的所有模块：

```js
const fs = require('fs');
const http = require('http');
const JSDOM = require('jsdom').JSDOM;
```

1.  向`http://localhost:3000`发出 HTTP 请求：

```js
const page = 'http://localhost:3000';
console.log(`Downloading ${page}...`);
const request = http.get(page, (response) => {
```

1.  确保成功响应并使用数据事件从主体收集数据：

```js
if (response.statusCode != 200) {
  console.error(`Error while fetching page ${page}: ${response.statusCode}`);
  console.error(`Status message: ${response.statusMessage}`);
  return;
}
let content = '';
response.on('data', (chunk) => content += chunk.toString());
```

1.  在`close`事件中，使用`JSDOM`解析 HTML：

```js
response.on('close', () => {
  console.log('Download finished.');
  const document = new JSDOM(content).window.document;
  writeCSV(extractProducts(document));
});
```

前面的回调调用了两个函数：`extractProducts`和`writeCSV`。这些函数将在接下来的步骤中描述。

1.  使用`extractProducts`函数查询 DOM 并从中获取产品信息。它将所有产品存储在一个数组中，并在最后返回：

```js
function extractProducts(document) {
  const products = [];
  console.log('Parsing product data...');
  Array.from(document.getElementsByClassName('item'))
    .forEach((el) => {
      process.stdout.write('.');
      const priceAndUnitElement = el.getElementsByTagName('span')[0];
      const priceAndUnit = priceAndUnitElement.textContent.split("/");
     const price = priceAndUnit[0].trim().substr(1);
      const unit = priceAndUnit[1].trim();
      const name = el.getElementsByTagName('a')[0].textContent;
      products.push({ name, price: parseFloat(price), unit });
    });
  console.log();
  console.log(`Found ${products.length} products.`);
  return products;
}
```

1.  使用`writeCSV`函数打开 CSV 文件进行写入，确保没有发生错误：

```js
function writeCSV(products) {
 const fileName = 'products.csv';
  console.log(`Writing data to ${fileName}...`);
  fs.open(fileName, 'w', (error, fileDescriptor) => {
    if (error != null) {
      console.error(`Can not write to file: ${fileName}`, error);
      return;
    }
```

1.  现在文件已打开，我们可以将产品数据写入文件：

```js
    // Write header
    fs.writeSync(fileDescriptor, 'name,price,unit\n');
    // Write content
    products.forEach((product) => {
      const line = `${product.name},${product.price},${product.unit}\n`;
      fs.writeSync(fileDescriptor, line);
    });
    console.log('Done.');
  });
}
```

1.  在新的终端中，运行应用程序：

```js
$ node .
Downloading http://localhost:3000...
Download finished.
Parsing product data...
.....................
Found 21 products.
Writing data to products.csv...
```

## 第四章：使用 Node.js 构建 RESTful API

### 活动 5：为键盘门锁创建 API 端点

**解决方案**

1.  创建一个新的项目文件夹，并将目录更改为以下内容：

```js
mkdir passcode
cd passcode
```

1.  初始化一个`npm`项目并安装`express`，`express-validator`和`jwt-simple`。然后，创建一个`routes`目录：

```js
npm init -y
npm install --save express express-validator jwt-simple
mkdir routes
```

1.  创建一个`config.js`文件，就像在*练习 21，设置需要身份验证的端点*中所做的那样。这应该包含一个随机生成的秘密值：

```js
let config = {};
// random value below generated with command: openssl rand -base64 32
config.secret = "cSmdV7Nh4e3gIFTO0ljJlH1f/F0ROKZR/hZfRYTSO0A=";
module.exports = config;
```

1.  创建`routes/check-in.js`文件以创建一个签到路由。这可以从*练习 21，设置需要身份验证的端点*中完整复制：

```js
const express = require('express');
const jwt = require('jwt-simple');
const { check, validationResult } = require('express-validator/check');
const router = express.Router();
// import our config file and get the secret value
const config = require('../config');
```

1.  创建一个名为`routes/lock.js`的第二个路由文件。首先导入所需的库和模块，创建一个空数组来保存我们的有效密码：

```js
const express = require('express');
const app = express();
const { check, validationResult } = require('express-validator/check');
const router = express.Router();
// Import path and file system libraries for importing our route files
const path = require('path');
const fs = require('fs');
// Import library for handling HTTP errors
const createError = require('http-errors');
// Import library for working with JWT tokens
const jwt = require('jwt-simple');
// import our config file and get the secret value
const config = require('./../config');
const secret = config.secret;
// Create an array to keep track of valid passcodes
let passCodes = [];
```

1.  为`/code`创建一个`GET`路由，需要一个`name`值，在前面步骤中的`routes/lock.js`文件中继续：

```js
router.get(['/code'], [
    check('name').isString().isAlphanumeric().exists()
  ],
  (req, res) => {
    let codeObj = {};
    codeObj.guest = req.body.name;
    // Check that authorization header was sent
    if (req.headers.authorization) {
      let token = req.headers.authorization.split(" ")[1];
      try {
        req._guest = jwt.decode(token, secret);
      } catch {
        res.status(403).json({ error: 'Token is not valid.' });
      }
      // If the decoded object guest name property
      if (req._guest.name) {
        codeObj.creator = req._guest.name;
```

1.  在`routes/lock.js`中创建另一个路由。这个路由将是`/open`，需要一个四位代码，将根据`passCodes`数组进行检查以查看它是否有效。在该路由下面，确保导出`router`，以便在`server.js`中使用：

```js
router.post(['/open'], [
    check('code').isLength({ min: 4, max: 4 })
  ],
  (req, res) => {
    let code = passCodes.findIndex(obj => {
      return obj.code === req.body.code;
    });
    if(code !== -1) {
      passCodes.splice(code, 1);
      res.json({ message: 'Pass code is valid, door opened.' });
    } else {
      res.status(403).json({ error: 'Pass code is not valid.' });
    }
});
// Export route so it is available to import
module.exports = router;
```

1.  创建主文件，在其中使用我们的路由`server.js`。首先导入所需的库，并设置 URL 编码 JSON：

```js
const express = require('express');
const app = express();
// Import path and file system libraries for importing our route files
const path = require('path');
const fs = require('fs');
// Import library for handling HTTP errors
const createError = require('http-errors');
// Tell express to enable url encoding
app.use(express.urlencoded({extended: true}));
app.use(express.json());
```

1.  接下来，在`server.js`中，在前面的代码下面，导入两个路由，实现一个`404`捕获，并告诉 API 监听端口`3000`：

```js
// Import our index route
let lock = require('./routes/lock');
let checkIn = require('./routes/check-in');
app.use('/check-in', checkIn);
app.use('/lock', lock);
// catch 404 and forward to error handler
app.use(function(req, res, next) {
```

1.  最后，我们将测试 API 以确保它被正确完成。首先运行您的程序：

```js
npm start
```

1.  在程序运行时，打开第二个终端窗口，并使用`/check-in`端点获取 JWT 并将值保存为`TOKEN`。然后，回显该值以确保成功：

```js
TOKEN=$(curl -sd "name=john" -X POST http://localhost:3000/check-in \
  | jq -r ".token")
echo $TOKEN
```

您应该收到一个包含字母和数字的长字符串，就像下面这样：

![图 4.24：从签到端点获取 TOKEN](img/C14587_04_24.jpg)

###### 图 4.24：从签到端点获取 TOKEN

1.  接下来，我们将使用我们的 JWT 使用`/lock/code`端点为 Sarah 获取一次性通行码：

```js
curl -sd "name=sarah" -X GET \
  -H "Authorization: Bearer ${TOKEN}" \
  http://localhost:3000/lock/code \
  | jq
```

您应该收到一个包含消息和四位代码的对象，就像下面这样：

![图 4.25：一个四位一次性代码](img/C14587_04_25.jpg)

###### 图 4.25：一个四位一次性代码

1.  为了确保代码正常工作，将其发送到`/lock/open`端点。我们将发送以下命令一次，期望它成功。然后我们将发送相同的命令第二次，期望它失败，因为每个代码只能使用一次。运行以下命令两次：

```js
# IMPORTANT: Make sure to replace 4594, with your specific passcode!
curl -sd "code=4594" -X POST \
  http://localhost:3000/lock/open \
  | jq
```

连续运行上述命令两次应该返回类似以下内容：

![图 4.26：运行命令两次会导致错误](img/C14587_04_26.jpg)

###### 图 4.26：运行命令两次会导致错误

如果你的结果与前面的图像相同，那么你已成功完成了这个活动。

## 第五章：模块化 JavaScript

### 活动 6：创建带有闪光模式的灯泡

**解决方案**：

1.  安装`babel-cli`和`babel`预设为开发人员依赖项：

```js
npm install --save-dev webpack webpack-cli @babel/core @babel/cli @babel/preset-env
```

1.  在根目录下添加一个名为`.babelrc`的文件。在其中，我们将告诉 Babel 使用预设设置：

```js
{
  "presets": ["@babel/preset-env"]
}
```

1.  在根目录下的`webpack.config.js`添加一个 webpack 配置文件：

```js
const path = require("path");
module.exports = {
  mode: 'development',
  entry: "./build/js/viewer.js",
  output: {
    path: path.resolve(__dirname, "build"),
    filename: "bundle.js"
  }
};
```

1.  创建一个名为`js/flashingLight.js`的新文件。这应该是一个空的 ES6 组件，扩展`Light`。在构造函数中，我们将包括`state`，`brightness`和`flashMode`：

```js
import Light from './light.js';
let privateVars = new WeakMap();
class FlashingLight extends Light {
  constructor(state=false, brightness=100, flashMode=true) {
    super(state, brightness);
    let info = {"flashMode": flashMode};
    privateVars.set(this, info);
    if(flashMode===true) {
      this.startFlashing();
    }
  }
```

1.  为`FlashingLight`对象添加一个 setter 方法，这也将触发停止和开始闪光方法。

```js
  setFlashMode(flashMode) {
    let info = privateVars.get(this);
    info.flashMode = checkStateFormat(flashMode);
    privateVars.set(this, info);
    if(flashMode===true) {
      this.startFlashing();
    } else {
      this.stopFlashing();
    }
  }
```

1.  为`FlashingLight`对象添加一个 getter 方法：

```js
  getFlashMode() {
    let info = privateVars.get(this);
    return info.flashMode;
  }
```

1.  创建一个`startFlashing`函数，引用父类的`lightSwitch()`函数。这一步很棘手，因为我们必须将它绑定到`setInterval`：

```js
  startFlashing() {
    let info = privateVars.get(this);
    info.flashing = setInterval(this.toggle.bind(this),5000);
  }
```

1.  创建一个`stopFlashing`函数，用于关闭定时器：

```js
  stopFlashing() {
    let info = privateVars.get(this);
    clearInterval(info.flashing);
  }
```

1.  作为`flashingLight.js`的最后部分，关闭类并导出它：

```js
}
export default FlashingLight;
```

1.  打开`src/js/viewer.js`并修改按钮以创建一个闪光灯而不是一个彩色灯：

```js
button.onclick = function () {
  new FlashingLight(true, slider.value, true);
}
```

1.  通过运行我们的`build`函数使用 npm 编译代码：

```js
npm run build
```

1.  打开`build/index.html`并将脚本位置设置为`bundle.js`：

```js
<script src="bundle.js" type="module"></script>
```

1.  为了测试一切是否按预期工作，请运行`npm start`并在浏览器中打开`localhost:8000`。点击`build`按钮创建一个完整页面的灯。如果一切都做对了，你应该看到每盏灯在 5 秒的间隔内闪烁：

![图 5.20：带有闪光模式的灯泡](img/C14587_05_20.jpg)

###### 图 5.20：带有闪光模式的灯泡

## 第六章：代码质量

### 活动 7：将所有内容整合在一起

**解决方案**

1.  安装 linting 练习中列出的开发人员依赖项（`eslint`，`prettier`，`eslint-config-airbnb-base`，`eslint-config-prettier`，`eslint-plugin-jest`和`eslint-plugin-import`）：

```js
npm install --save-dev eslint prettier eslint-config-airbnb-base eslint-config-prettier eslint-plugin-jest eslint-plugin-import
```

1.  添加一个`eslint`配置文件`.eslintrc`，其中包含以下内容：

```js
{
 "extends": ["airbnb-base", "prettier"],
  "parserOptions": {
    "ecmaVersion": 2018,
    "sourceType": "module"
  },
  "env": {
    "browser": true,
    "node": true,
    "es6": true,
    "mocha": true,
    "jest": true
  },
  "plugins": [],
  "rules": {
    "no-unused-vars": [
      "error",
      {
        "vars": "local",
        "args": "none"
      }
    ],
    "no-plusplus": "off",
  }
}
```

1.  添加一个`.prettierignore`文件：

```js
node_modules
build
dist
```

1.  在你的`package.json`文件中添加一个`lint`命令：

```js
  "scripts": {
    "start": "http-server",
    "lint": "prettier --write js/*.js && eslint js/*.js"
  },
```

1.  打开`assignment`文件夹并安装使用 Puppeteer 与 Jest 的开发人员依赖项：

```js
npm install --save-dev puppeteer jest jest-puppeteer
```

1.  通过添加一个选项告诉 Jest 使用`jest-puppeteer`预设来修改你的`package.json`文件：

```js
  "jest": {
    "preset": "jest-puppeteer"
  },
```

1.  在`package.json`中添加一个`test`脚本，运行`jest`：

```js
  "scripts": {
    "start": "http-server",
    "lint": "prettier --write js/*.js && eslint js/*.js",
    "test": "jest"
  },
```

1.  创建一个包含以下内容的`jest-puppeteer.config.js`文件：

```js
module.exports = {
  server: {
    command: 'npm start',
    port: 8080,
  },
}
```

1.  在`__tests__/calculator.js`创建一个测试文件，其中包含以下内容：

```js
describe('Calculator', () => {
  beforeAll(async () => {
    await page.goto('http://localhost:8080');
  })
  it('Check that 777 times 777 is 603729', async () => {
    const seven = await page.$("#seven");
    const multiply = await page.$("#multiply");
    const equals = await page.$("#equals");
    const clear = await page.$("#clear");
    await seven.click();
    await seven.click();
    await seven.click();
    await multiply.click();
    await seven.click();
    await seven.click();
    await seven.click();
    await equals.click();
    const result = await page.$eval('#screen', e => e.innerText);
    expect(result).toMatch('603729');
    await clear.click();
  })
  it('Check that 3.14 divided by 2 is 1.57', async () => {
    const one = await page.$("#one");
    const two = await page.$("#two");
    const three = await page.$("#three");
    const four = await page.$("#four");
    const divide = await page.$("#divide");
    const decimal = await page.$("#decimal");
    const equals = await page.$("#equals");
    await three.click();
    await decimal.click();
    await one.click();
    await four.click();
    await divide.click();
    await two.click();
    await equals.click();
    const result = await page.$eval('#screen', e => e.innerText);
    expect(result).toMatch('1.57');
  })
})
```

1.  创建一个包含以下内容的`.huskyrc`文件：

```js
{
  "hooks": {
    "pre-commit": "npm run lint && npm test"
  }
}
```

1.  通过运行`npm install --save-dev husky`安装`husky`作为开发人员依赖项：![图 6.19：安装 Husky](img/C14587_06_19.jpg)

###### 图 6.19：安装 Husky

1.  确保使用`npm test`命令正确运行测试：

```js
npm test
```

这应该返回两个测试的正面结果，如下图所示：

![图 6.20：显示两个测试的正面结果](img/C14587_06_20.jpg)

###### 图 6.20：显示两个测试的正面结果

通过进行测试提交来确保 Git 钩子和 linting 正常工作。

## 第七章：高级 JavaScript

### 活动 8：创建一个用户跟踪器

**解决方案**

1.  打开`Activity08.js`文件并定义`logUser`。它将把用户添加到`userList`参数中。确保不会添加重复项：

```js
function logUser(userList, user) {
if(!userList.includes(user)) {
userList.push(user);
}
}
```

在这里，我们使用`includes`方法来检查用户是否已经存在。如果他们不存在，他们将被添加到我们的列表中。

1.  定义`userLeft`。它将从`userList`参数中移除用户。如果用户不存在，它将不执行任何操作：

```js
function userLeft(userList, user) {
const userIndex = userList.indexOf(user);
if (userIndex >= 0) {
    userList.splice(userIndex, 1);
}
}
```

在这里，我们使用`indexOf`来获取要移除的用户的当前索引。如果该项不存在，`indexOf`将`返回-1`，因此我们只在存在时使用`splice`来移除该项。

1.  定义`numUsers`，返回当前列表中的用户数：

```js
function numUsers(userList) {
return userLeft.length;
}
```

1.  定义一个名为`runSite`的函数。我们将创建一个`users`数组，并调用我们之前声明的函数来测试我们的实现。之后我们也会调用该函数：

```js
function runSite() {
    // Your user list for your website
    const users = [];
    // Simulate user viewing your site
    logUser(users, 'user1');
    logUser(users, 'user2');
    logUser(users, 'user3');
    // User left your website
    userLeft(users, 'user2');
    // More user goes to your website
    logUser(users, 'user4');
    logUser(users, 'user4');
    logUser(users, 'user5');
    logUser(users, 'user6');
    // More user left your website
    userLeft(users, 'user1');
    userLeft(users, 'user4');
    userLeft(users, 'user2');
    console.log('Current user: ', users.join(', '));
}
runSite();
```

在定义函数之后，运行上述代码将返回以下输出：

![图 7.62：运行 log_users.js 的输出](img/C14587_07_62.jpg)

###### 图 7.62：运行 log_users.js 的输出

### 活动 9：使用 JavaScript 数组和类创建学生管理器

**解决方案**

1.  创建一个包含所有学生信息的`School`类：

```js
class School {
constructor() {
    this.students = [];
}
}
```

在`School`构造函数中，我们只是初始化了一个学生列表。稍后，我们将向此列表添加新学生。

1.  创建一个`Student`类，包括有关学生的所有相关信息：

```js
class Student {
constructor(name, age, gradeLevel) {
    this.name = name;
    this.age = age;
    this.gradeLevel = gradeLevel;
    this.courses = [];
}
}
```

在学生`constructor`中，我们存储了课程列表，以及学生的`age`，`name`和`gradeLevel`。

1.  创建一个`Course`类，其中包括有关课程的`name`和`grade`的信息：

```js
class Course {
constructor(name, grade) {
    this.name = name;
    this.grade = grade;
}
}
```

课程构造函数只是将课程的名称和成绩存储在`object`中。

1.  在`School`类中创建`addStudent`：

```js
addStudent(student) {
this.students.push(student);
}
```

1.  在`School`类中创建`findByGrade`：

```js
findByGrade(gradeLevel) {
    return this.students.filter((s) => s.gradeLevel === gradeLevel);
}
```

1.  在`School`类中创建`findByAge`：

```js
findByAge(age) {
return this.students.filter((s) => s.age === age);
}
```

1.  在`School`类中创建`findByName`：

```js
findByName(name) {
return this.students.filter((s) => s.name === name);
}
```

1.  在`Student`类中，创建一个`calculateAverageGrade`方法来计算学生的平均成绩：

```js
calculateAverageGrade() {
const totalGrades = this.courses.reduce((prev, curr) => prev + curr.grade, 0);
return (totalGrades / this.courses.length).toFixed(2);
}
```

在`calculateAverageGrade`方法中，我们使用数组 reduce 来获取学生所有课程的总成绩。然后，我们将其除以课程列表中的课程数。

1.  在`Student`类中，创建一个名为`assignGrade`的方法，用于为学生正在上的课程分配数字成绩：

```js
assignGrade(name, grade) {
this.courses.push(new Course(name, grade))
}
```

您应该在`student_manager.js`文件中进行工作，并修改提供的方法模板。如果您正确实现了所有内容，您应该看到**TEST PASSED**消息：

![图 7.63：显示 TEST PASSED 消息的屏幕截图](img/C14587_07_63.jpg)

###### 图 7.63：显示 TEST PASSED 消息的屏幕截图

### 活动 10：重构函数以使用现代 JavaScript 功能

**解决方案**

1.  打开`Activity03.js`；它应该包含用传统 JavaScript 编写的各种函数。当您使用 Node.js 运行`Activity03.js`时，您应该看到以下输出：![图 7.64：运行 Lesson7-activity.js 后的输出](img/C14587_07_64.jpg)

###### 图 7.64：运行 Lesson7-activity.js 后的输出

1.  您需要重构`itemExist`，使用`includes`数组：

```js
function itemExist(array, item) {
    return array.includes(item);
}

In pushUnique we will use array push to add new item to the bottom
function pushUnique(array, item) {
    if (!itemExist(array, item)) {
        array.push(item);
    }
}

```

1.  在`createFilledArray`中，我们将使用`array.fill`来用初始值填充我们的数组：

```js
function createFilledArray(size, init) {
    const newArray = new Array(size).fill(init);
    return newArray;
}

In removeFirst we will use array.shift to remove the first item
function removeFirst(array) {
    return array.shift();
}
```

1.  在`removeLast`中，我们将使用`array.pop`来移除最后一项：

```js
function removeLast(array) {
    return array.pop();
}

In cloneArray we will use spread operation to make clone for our array
function cloneArray(array) {
    return […array];
}

```

1.  我们将使用`ES6`类重构我们的`Food`类：

```js
class Food {
    constructor(type, calories) {
        this.type = type;
        this.calories = calories;
    }
    getCalories() {
        return this.calories;
    }
}
```

在您完成重构并运行现有代码后，您应该看到相同的输出：

![图 7.65：显示 TEST PASSED 消息的输出](img/C14587_07_65.jpg)

###### 图 7.65：显示 TEST PASSED 消息的输出

## 第八章：异步编程

### 活动 11：使用回调接收结果

**解决方案**：

1.  创建一个`calculate`函数，它接受`id`和`callback`作为参数：

```js
function calculate(id, callback) {
}
```

1.  我们将首先调用`getUsers`来获取所有用户。这将给我们所需的地址：

```js
function calculate(id, callback) {
clientApi.getUsers((error, result) => {
if (error) { return callback(error); }
const currentUser = result.users.find((user) => user.id === id);
if (!currentUser) { return callback(new Error('user not found')); }
});
  }
```

在这里，我们获取所有用户，然后对`user`应用`find`方法来从列表中找到我们想要的用户。如果该用户不存在，我们将使用`User not found`错误调用`callback`函数。

1.  调用`getUsage`来获取用户的使用情况：

```js
clientApi.getUsage(id, (error, usage) => {
if (error) { return callback(error); }
  });
```

然后，我们需要将对`getUsers`的调用放在`getUsage`的回调函数中，这样它将在我们完成调用`getUsers`后运行。在这里，回调函数将被调用并传入一个数字列表，这将是使用情况。如果我们从`getUsage`收到错误，我们还将使用错误对象调用回调函数。

1.  最后，调用`getRate`以获取我们正在计算的用户的费率：

```js
clientApi.getRate(id, (error, rate) => {
if (error) { return callback(error); }
let totalUsage = 0;
for (let i = 0; i < usage.length; i++) {
    totalUsage += usage[i];
}
callback(null, {
id,
address: currentUser.address,
due: rate * totalUsage
});
});
```

我们将把这个调用放在`getUsage`的回调函数中。这为我们需要的所有信息创建了一个嵌套的链请求。最后，我们将使用数组 reduce 来计算该用户的总使用量，然后将其乘以费率以获得最终应付金额。

1.  当函数完成时，使用现有 ID 调用它，如下面的代码：

```js
calculate('DDW2AU', (error, result) => {
    console.log(error, result);
});
```

您应该看到以下输出：

![图 8.43：使用现有 ID 调用函数](img/C14587_08_43.jpg)

###### 图 8.43：使用现有 ID 调用函数

1.  使用一个不存在的 ID 调用函数：

```js
calculate('XXX', (error, result) => {
    console.log(error, result);
});
```

您应该看到返回的错误如下：

![图 8.44：使用不存在的 ID 调用函数](img/C14587_08_44.jpg)

###### 图 8.44：使用不存在的 ID 调用函数

### 活动 12：使用异步和等待重构账单计算器

**解决方案**

1.  将`calculate`函数创建为`async`函数：

```js
async function calculate(id) {
}
```

1.  使用`await`调用`getUsers`以获取`users`中的解析结果：

```js
const users = await clientApi.getUsers();
const currentUser = users.users.find((user) => user.id === id);
```

当我们使用`await`关键字时，我们必须使用`async`函数。`await`关键字将打破程序的控制，并且只有在等待的 promise 被解析后才会返回并继续执行。

1.  使用`await`调用`getUsage`以获取用户的使用情况：

```js
const usage = await clientApi.getUsage(currentUser.id);
```

1.  使用`await`调用`getRate`以获取用户的费率：

```js
const rate = await clientApi.getRate(currentUser.id);
```

1.  最后，我们将调用`return`以检索`id`，`address`和`due`：

```js
return {
id,
address: currentUser.address,
due: (rate * usage.reduce((prev, curr) => curr + prev)).toFixed(2)
};
```

1.  将`calculateAll`函数创建为`async`函数：

```js
async function calculateAll() {
}
```

1.  在调用`getUsers`时使用`await`并将结果存储在`result`中：

```js
const result = await clientApi.getUsers();
```

1.  使用映射数组创建一个 promise 列表，并使用`Promise.all`将它们包装起来。然后，应该在`Promise.all`返回的 promise 上使用`await`：

```js
return await Promise.all(result.users.map((user) => calculate(user.id)));
```

因为`await`将在任何 promise 上工作，并且会等待直到值被解析，它也会等待我们的`Promise.all`。在它被解析后，最终数组将被返回。

1.  在一个用户上调用`calculate`：

```js
calculate('DDW2AU').then(console.log)
```

输出应如下所示：

![图 8.45：在一个用户上调用 calculate](img/C14587_08_45.jpg)

###### 图 8.45：在一个用户上调用 calculate

1.  调用`calculateAll`函数：

```js
calculateAll().then(console.log)
```

输出应如下所示：

![图 8.46：调用 calculateAll 函数](img/C14587_08_46.jpg)

###### 图 8.46：调用 calculateAll 函数

正如您所看到的，当我们调用`async`函数时，我们可以将它们视为返回 promise 的函数。

## 第九章：事件驱动编程和内置模块

### 活动 13：构建事件驱动模块

**解决方案**：

执行以下步骤完成此活动：

1.  导入`events`模块：

```js
const EventEmitter = require('events');
```

1.  创建`SmokeDetector`类，它扩展了`EventEmitter`并将`batteryLevel`设置为`10`：

```js
class SmokeDetector extends EventEmitter {
    constructor() {
        super();
        this.batteryLevel = 10;
    }
}
```

在我们的构造函数中，因为我们正在扩展`EventEmitter`类并且正在分配一个自定义属性`batteryLevel`，我们需要在构造函数中调用`super`并将`batteryLevel`设置为`10`。

1.  在`SmokeDetector`类中创建一个`test`方法，该方法将测试电池电量，并在电池电量低时发出`低电量`消息：

```js
test() {
        if (this.batteryLevel > 0) {
            this.batteryLevel -= 0.1;
            if (this.batteryLevel < 0.5) {
                this.emit('low battery');
            }
            return true;
        }
        return false;
    }
```

我们的`test()`方法将检查电池电量，并在电池电量低于 0.5 单位时发出`低电量`事件。每次运行`test`方法时，我们还会减少电池电量。

1.  创建`House`类，它将存储我们事件监听器的实例：

```js
class House {
    constructor(numBedroom, numBathroom, numKitchen) {
        this.numBathroom = numBathroom;
        this.numBedroom = numBedroom;
        this.numKitchen = numKitchen;
        this.alarmListener = () => {
            console.log('alarm is raised');
        }
        this.lowBatteryListener = () => {
            console.log('alarm battery is low');
        }
    }
}
```

在`House`类中，我们存储了一些关于房子的信息。我们还将两个事件侦听器函数存储为此对象的属性。这样，我们可以使用函数引用在想要分离侦听器时调用`removeListener`。

1.  在`House`类中创建一个`addDetector`方法。在这里，我们将附加事件侦听器：

```js
addDetector(detector) {
        detector.on('alarm', this.alarmListener);
        detector.on('low battery', this.lowBatteryListener);
    }
```

在这里，我们期望传入的探测器是一个`EventEmitter`。我们将两个事件侦听器附加到我们的`detector`参数。当这些事件被触发时，它将调用我们对象内的事件发射器。

1.  创建一个`removeDetector`方法，它将帮助我们删除先前附加的警报事件侦听器：

```js
removeDetector(detector) {
        detector.removeListener('alarm', this.alarmListener);
        detector.removeListener('low battery', this.lowBatteryListener);
    }
```

在这里，我们使用函数引用和警报参数来删除附加到我们侦听器的侦听器。一旦调用了这个，事件就不应该再次调用我们的侦听器。

1.  创建一个名为`myHouse`的`House`实例。这将包含关于我们房子的一些示例信息。它还将用于监听我们的烟雾探测器发出的事件：

```js
const myHouse = new House(2, 2, 1);
```

1.  创建一个名为`detector`的`SmokeDetector`实例：

```js
const detector = new SmokeDetector();
```

1.  将我们的`detector`添加到`myHouse`：

```js
myHouse.addDetector(detector);
```

1.  创建一个循环来调用测试函数`96`次：

```js
for (let i = 0; i < 96; i++) {
    detector.test();
}
```

因为测试函数将减少电池电量，如果我们调用它`96`次，我们将期望发出*低电量*警报。这将产生以下输出：

![图 9.50：发出低电量警报](img/C14587_09_50.jpg)

###### 图 9.50：发出低电量警报

1.  在`detector`对象上发出警报：

```js
detector.emit('alarm');
```

以下是前面代码的输出：

![图 9.51：对检测器对象发出的警报](img/C14587_09_51.jpg)

###### 图 9.51：对检测器对象发出的警报

1.  从`myHouse`对象中删除`detector`：

```js
myHouse.removeDetector(detector);
```

1.  测试这个以在`detector`上发出警报：

```js
detector.test();
detector.emit('alarm');
```

因为我们刚刚从我们的房子中移除了`detector`，所以我们不应该看到这个输出：

![图 9.52：测试检测器上的发出警报](img/C14587_09_52.jpg)

###### 图 9.52：测试检测器上的发出警报

### 活动 14：构建文件监视器

**解决方案**：

1.  导入`fs`和`events`：

```js
const fs = require('fs').promises;
const EventEmitter = require('events');
```

1.  创建一个扩展`EventEmitter`类的`fileWatcher`类。使用`modify`时间戳来跟踪文件更改。

我们需要创建一个扩展`EventEmitter`的`FileWatcher`类。它将在构造函数中以文件名和延迟作为参数。在构造函数中，我们还需要设置上次修改时间和计时器变量。现在我们将它们保持为未定义：

```js
class FileWatcher extends EventEmitter {
    constructor(file, delay) {
        super();
        this.timeModified = undefined;
        this.file = file;
        this.delay = delay;
        this.watchTimer = undefined;
    }
}
```

这是查看文件是否已更改的最基本方法。

1.  创建`startWatch`方法以开始监视文件的更改：

```js
startWatch() {
        if (!this.watchTimer) {
            this.watchTimer = setInterval(() => {
                fs.stat(this.file).then((stat) => {
                    if (this.timeModified !== stat.mtime.toString()) {
                        console.log('modified');
                        this.timeModified = stat.mtime.toString();
                    }
                }).catch((error) => {
                    console.error(error);
                });
            }, this.delay);
        }
    }
```

在这里，我们使用`fs.stat`来获取文件的信息，并将修改时间与上次修改时间进行比较。如果它们不相等，我们将在控制台中输出**修改**。

1.  创建`stopWatch`方法以停止监视文件的更改：

```js
stopWatch() {
        if (this.watchTimer) {
            clearInterval(this.watchTimer);
            this.watchTimer = undefined;
        }
       }
```

`stopWatch`方法非常简单：我们将检查这个对象中是否有一个计时器。如果有，那么我们将在该计时器上运行`clearInterval`以清除该计时器。

1.  在与`filewatch.js`相同的目录中创建一个名为`test.txt`的文件。

1.  创建一个`FileWatcher`实例并每`1000`毫秒开始监视文件：

```js
const watcher = new FileWatcher('test.txt', 1000);
watcher.startWatch();
```

1.  修改`test.txt`中的一些内容并保存。您应该看到以下输出：![图 9.53：修改`test.txt`文件内容后的输出](img/C14587_09_53.jpg)

###### 图 9.53：修改`test.txt`文件内容后的输出

我们修改了文件两次，这意味着我们看到了三条修改消息。这是因为当我们开始观察时，我们将其视为文件已被修改。

1.  修改`startWatch`以检索新内容：

```js
startWatch() {
        if (!this.watchTimer) {
            this.watchTimer = setInterval(() => {
                fs.stat(this.file).then((stat) => {
                    if (this.timeModified !== stat.mtime.toString()) {
                        fs.readFile(this.file, 'utf-8').then((content) => {
                            console.log('new content is: ', content);
                        }).catch((error) => {
                            console.error(error);
                        });
                        this.timeModified = stat.mtime.toString();
                    }
                }).catch((error) => {
                    console.error(error);
                });
            }, this.delay);
        }
    }
```

当我们修改`test.txt`并保存时，我们的代码应该检测到并输出新内容：

![图 9.54：可以使用 startWatch 函数看到对文件所做的修改](img/C14587_09_54.jpg)

###### 图 9.54：可以使用 startWatch 函数看到对文件所做的修改

1.  修改`startWatch`，使其在文件被修改时发出事件，并在遇到错误时发出错误：

```js
startWatch() {
        if (!this.watchTimer) {
            this.watchTimer = setInterval(() => {
                fs.stat(this.file).then((stat) => {
                    if (this.timeModified !== stat.mtime.toString()) {
                        fs.readFile(this.file, 'utf-8').then((content) => {
                            this.emit('change', content);
                        }).catch((error) => {
                            this.emit('error', error);
                        });
                        this.timeModified = stat.mtime.toString();
                    }
                }).catch((error) => {
                    this.emit('error', error);
                });
            }, this.delay);
        }
    }
```

我们将不再输出内容，而是发出一个带有新内容的事件。这使我们的代码更加灵活。

1.  将事件处理程序附加到`error`并在我们的文件`watcher`上更改它们：

```js
watcher.on('error', console.error);
watcher.on('change', (change) => {
    console.log('new change:', change);
});
```

1.  运行代码并修改`test.txt`：

![图 9.55：更改文件监视器后的输出](img/C14587_09_55.jpg)

###### 图 9.55：更改文件监视器后的输出

## 第十章：使用 JavaScript 进行函数式编程

### 活动 15：onCheckout 回调属性

解决方案

1.  将当前目录更改为`Lesson10`，如果之前在此目录中尚未执行过`npm install`，则运行`npm install`。`npm install`会下载运行此活动所需的依赖项（React 和 Parcel）。

1.  运行`parcel serve activity-on-checkout-prop-start.html`，然后执行`npm run Activity15`。您将看到应用程序启动，如下所示：![图 10.42：运行 start html 脚本后的输出](img/C14587_10_42.jpg)

###### 图 10.42：运行 start html 脚本后的输出

1.  转到`http://localhost:1234`（或者启动脚本输出的任何 URL）。您应该看到以下 HTML 页面：![图 10.43：浏览器中的初始应用程序](img/C14587_10_43.jpg)

###### 图 10.43：浏览器中的初始应用程序

1.  **继续结账**的`onClick`可以实现如下：

```js
  render() {
    return (
      <div>
        <p>You have {this.state.items.length} items in your basket</p>
        <button onClick={() => this.props.onCheckout(this.state.items)}>
          Proceed to checkout
        </button>
      </div>
    );
  }
```

这是以下调查的延续：

在`Basket`组件的`render`方法中查找文本为“继续结账”的按钮。

注意到它的`onClick`处理程序当前是一个在调用时什么都不做的函数，`() => {}`。

用正确的调用`this.props.onCheckout`替换`onClick`处理程序。

1.  单击“继续结账”按钮后，我们应该看到以下内容：

![图 10.44：单击“继续结账”按钮后的输出](img/C14587_10_44.jpg)

###### 图 10.44：单击“继续结账”按钮后的输出

### 活动 16：测试选择器

解决方案

1.  运行`npm run Activity16`（或`node activity-items-selector-test-start.js`）。您将看到以下输出：![图 10.45：运行活动的初始启动文件后的预期输出](img/C14587_10_45.jpg)

###### 图 10.45：运行活动的初始启动文件后的预期输出

1.  测试一下，对于空状态，选择器返回`[]`：

```js
function test() {
  assert.deepStrictEqual(
    selectBasketItems(),
    [],
    'should be [] when selecting with no state'
  );
  assert.deepStrictEqual(
    selectBasketItems({}),
    [],
    'should be [] when selecting with {} state'
  );
}
```

1.  测试一下，对于一个空的购物篮对象，选择器返回[]：

```js
function test() {
  // other assertions
  assert.deepStrictEqual(
    selectBasketItems({basket: {}}),
    [],
    'should be [] when selecting with {} state.basket'
  );
}
```

1.  测试一下，如果项目数组已设置但为空，则选择器返回`[]`：

```js
function test() {
  // other assertions
  assert.deepStrictEqual(
    selectBasketItems({basket: {items: []}}),
    [],
    'should be [] when items is []'
  );
}
```

1.  测试一下，如果`items`数组不为空且已设置，则选择器返回它：

```js
function test() {
  // other assertions
  assert.deepStrictEqual(
    selectBasketItems({
      basket: {items: [{name: 'product-name'}]}
    }),
    [{name: 'product-name'}],
    'should be items when items is set'
  );
}
The full test function content after following the previous solution steps:
function test() {
  assert.deepStrictEqual(
    selectBasketItems(),
    [],
    'should be [] when selecting with no state'
  );
  assert.deepStrictEqual(
    selectBasketItems({}),
    [],
    'should be [] when selecting with {} state'
  );
  assert.deepStrictEqual(
    selectBasketItems({basket: {}}),
    [],
    'should be [] when selecting with {} state.basket'
  );
  assert.deepStrictEqual(
    selectBasketItems({basket: {items: []}}),
    [],
    'should be [] when items is []'
  );
  assert.deepStrictEqual(
    selectBasketItems({
      basket: {items: [{name: 'product-name'}]}
    }),
    [{name: 'product-name'}],
    'should be items when items is set'
  );
}
```

1.  实施测试的输出中不应该有错误：

![图 10.46：最终输出显示没有错误](img/C14587_10_46.jpg)

###### 图 10.46：最终输出显示没有错误

### 活动 17：从 BFF 获取当前购物篮

解决方案

1.  将当前目录更改为`Lesson10`，如果之前在此目录中尚未执行过`npm install`，则运行`npm install`。

1.  运行 Activity 17 的 BFF 和`npx parcel serve activity-app-start.html`。在开发过程中，运行`npm run Activity17`。您将看到应用程序启动，如下所示：![图 10.47：运行活动的初始启动文件](img/C14587_10_47.jpg)

###### 图 10.47：运行活动的初始启动文件

1.  转到`http://localhost:1234`（或者启动脚本输出的任何 URL）。您应该看到以下 HTML 页面：![图 10.48：浏览器中的初始应用程序](img/C14587_10_48.jpg)

###### 图 10.48：浏览器中的初始应用程序

1.  在 GraphiQL UI 中运行以下查询：

```js
{
  basket {
    items {
      id
      name
      price
      quantity
    }
  }
}
```

以下是前面代码的输出：

![图 10.49：带有购物篮查询的 GraphiQL UI](img/C14587_10_49.jpg)

###### 图 10.49：带有购物篮查询的 GraphiQL UI

1.  创建一个新的`requestBasket`动作创建器（利用 redux-thunk）。它使用上一步的查询调用`fetchFromBff`，并分派一个从 GraphQL 响应中提取的购物篮有效负载的`REQUEST_BASKET_SUCCESS`动作：

```js
function requestBasket() {
  return dispatch => {
    fetchFromBff(`{
      basket {
        items {
          id
          name
          price
          quantity
        }
      }
    }`).then(data => {
      dispatch({
        type: REQUEST_BASKET_SUCCESS,
        basket: data.basket
      });
    });
  };
}
```

1.  将篮子数据减少到存储中，并将以下情况添加到`appReducer`中，以将我们新的`REQUEST_BASKET_SUCCESS`动作的`basket`负载减少到状态中：

```js
const appReducer = (state = defaultState, action) => {
  switch (action.type) {
    // other cases
    case REQUEST_BASKET_SUCCESS:
      return {
        ...state,
        basket: action.basket
      };
    // other cases
  }
};
```

1.  在`mapDispatchToProps`中添加`requestBasket`，如下所示：

```js
const mapDispatchToProps = dispatch => {
  return {
    // other mapped functions
    requestBasket() {
      dispatch(requestBasket());
    }
  };
};
```

1.  在`componentDidMount`上调用`requestBasket`：

```js
class App extends React.Component {
  componentDidMount() {
    this.props.requestBasket();
  }
  // render method
}
```

当使用所有前述步骤加载应用程序时，它会闪烁显示“您的篮子中有 0 件物品”的消息，然后变成以下屏幕截图。当从 BFF 获取完成时，它会减少到存储中并导致重新渲染。这将再次显示篮子，如下所示：

![图 10.50：一旦与 BFF 集成，最终应用程序](img/C14587_10_50.jpg)

###### 图 10.50：一旦与 BFF 集成，最终应用程序
