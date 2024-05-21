# 第七章：代码质量

## 学习目标

在本章结束时，你将能够：

+   确定编写清晰 JavaScript 代码的最佳实践

+   执行代码检查并在你的 node 项目中添加一个检查命令

+   在你的代码上使用单元测试、集成测试和端到端测试方法

+   使用 Git 钩子自动化代码检查和测试

在本章中，我们将专注于提高代码质量，设置测试，并在 Git 提交之前自动运行测试。这些技术可以确保错误或问题能够及早被发现，从而不会进入生产环境。

## 介绍

在上一章中，我们探讨了模块化设计、ES6 模块以及它们在 Node.js 中的使用。我们将我们编译的 ES6 JavaScript 转换为兼容的脚本使用 Babel。

在本章中，我们将讨论代码质量，这是专业 JavaScript 开发的关键品质之一。当我们开始编写代码时，我们往往会专注于解决简单的问题和评估结果。对于大多数开发人员开始的小型项目，很少需要与他人沟通或作为大团队的一部分工作。

随着你参与的项目范围变得更大，代码质量的重要性也增加。除了确保代码能够正常工作，我们还必须考虑其他开发人员将使用我们创建的组件或更新我们编写的代码。

代码质量有几个方面。最明显的是它能够实现预期的功能。这通常说起来容易做起来难。很难满足大型项目的要求。更复杂的是，通常添加新功能可能会导致应用程序的某些现有部分出现错误。通过良好的设计可以减少这些错误，但即便如此，这些类型的故障还是会发生。

随着敏捷开发变得越来越流行，代码变更的速度也在增加。因此，测试比以往任何时候都更加重要。我们将演示如何使用单元测试来确认函数和类的正确功能。除了单元测试，我们还将研究集成测试，以确保程序的所有方面都能正确地一起运行。

代码质量的第二个组成部分是性能。我们代码中的算法可能会产生期望的结果，但它们是否能够高效地实现？我们将看看如何测试函数的性能，以确保算法在处理大量输入时能够在合理的时间内返回结果。例如，你可能有一个排序算法在处理 10 行数据时效果很好，但一旦尝试处理 100 行数据，就需要几分钟的时间。

本章我们将讨论代码质量的第三个方面，即可读性。可读性是衡量人类阅读和理解代码的难易程度。你是否曾经看过使用模糊函数和变量名称或者误导性变量名称编写的代码？在编写代码时，要考虑其他人可能需要阅读或修改它。遵循一些基本准则可以帮助提高可读性。

## 清晰命名

使代码更易读的最简单方法之一是**清晰命名**。尽可能使变量和函数的使用明显。即使是一个人的项目，也很容易在 6 个月后回到自己的代码时，难以记住每个函数的作用。当你阅读别人的代码时，这一点更加明显。

确保你的名称清晰且可读。考虑以下示例，开发人员创建了一个以`yymm`格式返回日期的函数：

```js
function yymm() {
  let date = new Date();
  Return date.getFullYear() + "/" + date.getMonth();
}
```

当我们了解了这个函数的上下文和解释时，它是明显的。但对于第一次浏览代码的外部开发人员来说，`yymm`很容易引起一些困惑。

将模糊函数重命名为使用明显的方式：

```js
function getYearAndMonth() {
  let date = new Date();
  return date.getFullYear() + "/" + date.getMonth();
}
```

当使用正确的函数和变量命名时，编写易读的代码变得容易。再举一个例子，我们想在夜间打开灯：

```js
if(time>1600 || time<600) {
  light.state = true;
}
```

在前面的代码中并不清楚发生了什么。`1600`和`600`到底是什么意思，如果灯的状态是`true`又代表什么？现在考虑将相同的函数重写如下：

```js
if(time.isNight) {
  light.turnOn;
}
```

前面的代码使相同的过程变得清晰。我们不再询问时间是否在 600 和 1600 之间，而是简单地询问是否是夜晚，如果是，我们就打开灯。

除了更易读外，我们还将夜间的定义放在了一个中心位置，`isNight`。如果我们想在 5:00 而不是 6:00 结束夜晚，我们只需要在`isNight`中更改一行，而不是在代码中找到所有`time<600`的实例。

### 规范

在格式化或编写代码的**规范**方面，有两类：行业或语言范例和公司/组织范例。行业或语言特定的规范通常被大多数使用该语言的程序员所接受。例如，在 JavaScript 中，行业范例是使用驼峰命名法来命名变量。

行业范例的良好来源包括 W3 JavaScript 样式指南和 Mozilla MDN Web 文档。

除了行业范例外，软件开发团队或项目通常会有一套更进一步的规范。有时，这些规范被编制成样式指南文件；在其他情况下，这些规范是未记录的。

如果你是一个有着相对庞大代码库的团队的一部分，记录特定的样式选择可能是有用的。这将帮助你考虑你想要保留和强制执行新更新的哪些方面，以及你可能想要更改的哪些方面。它还有助于培训可能熟悉 JavaScript 但不熟悉公司具体规范的新员工。

一个公司特定的样式指南的很好的例子是 Google JavaScript 样式指南([`google.github.io/styleguide/jsguide.html`](https://google.github.io/styleguide/jsguide.html))。它包含一些一般有用的信息。例如，*第 2.3.3 节*讨论了在代码中使用非 ASCII 的问题。它建议如下：

```js
const units = 'μs';
```

最好使用类似于：

```js
const units = '\u03bcs'; // 'μs'
```

没有注释使用`\u03bcs`会更糟。你的代码的意思越明显，越好。

公司通常有一套他们偏爱的库，用于记录日志、处理时间值（例如 Moment.js 库）和测试等。这对于兼容性和代码重用非常有用。例如，如果一个项目已经使用 Bunyan 记录日志，而其他人决定安装 Morgan 等替代库，那么使用不同开发人员使用的执行类似功能的多个依赖项会增加编译项目的大小。

#### 注意：样式指南

值得花时间阅读一些更受欢迎的 JavaScript 样式指南。不要觉得自己必须遵循每一个规则或建议，但要习惯于规则背后的思维方式。一些值得查看的热门指南包括以下内容：

MSDN 样式指南：[`developer.mozilla.org/en-US/docs/Web/JavaScript/Guide`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide)

### 主观与非主观

在规范方面，"主观"这个术语是你可能会遇到的。在探索现有的库和框架时，你经常会看到诸如"一个主观的框架"之类的短语。在这种情况下，"主观"是规范执行的严格程度的衡量标准：

主观的：严格执行其选择的规范和方法

非主观：不强制执行规范，也就是说，只要代码有效，就可以使用

### Linting

**Linting**是一个自动化的过程，其中代码被检查并根据一套样式指南的标准进行验证。例如，一个设置了 linting 以确保使用两个空格而不是制表符的项目将检测到制表符的实例，并提示开发人员进行更改。

了解 linting 很重要，但它并不是项目的严格要求。当我在一个项目上工作时，我考虑的主要因素是项目的规模和项目团队的规模。

在中长期项目和中大型团队中，Linting 确实非常有用。通常，新人加入项目时会有使用其他样式约定的经验。这意味着你会在文件之间甚至在同一个文件中得到混合的样式。这导致项目变得不太有组织且难以阅读。

另一方面，如果你正在为一个黑客马拉松编写原型，我建议你跳过 linting。它会给项目增加额外的开销，除非你使用一个带有你喜欢的 linting 的样板项目作为起点。

还有一种风险是 linting 系统过于严格，最终导致开发速度变慢。

良好的 Linting 应该考虑项目，并在强制执行通用样式和不太严格之间找到平衡。

### 练习 29：设置 ESLint 和 Prettier 来监视代码中的错误

在这个练习中，我们将安装并设置 ESLint 和 Prettier 来监视我们的代码的样式和语法错误。我们将使用由 Airbnb 开发的流行的 ESLint 约定，这已经成为了一种标准。

#### 注意

这个练习的代码文件可以在[`github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson06/Exercise29/result`](https://github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson06/Exercise29/result)找到。

执行以下步骤完成练习：

1.  创建一个新的文件夹并初始化一个`npm`项目：

```js
mkdir Exercise29
cd Exercise29
npm init -y
npm install --save-dev eslint prettier eslint-config-airbnb-base eslint-config-prettier eslint-plugin-jest eslint-plugin-import
```

我们在这里安装了几个开发者依赖项。除了`eslint`和`prettier`之外，我们还安装了由 Airbnb 制作的起始配置，一个与 Prettier 一起工作的配置，以及一个为基于 Jest 的测试文件添加样式异常的扩展。

1.  创建一个`.eslintrc`文件：

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
   "jest": true,
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

1.  创建一个`.prettierignore`文件（类似于`.gitignore`文件，这只是列出应该被 Prettier 忽略的文件）。你的`.prettierignore`文件应包含以下内容：

```js
node_modules
build
dist
```

1.  创建一个`src`文件夹，并在其中创建一个名为`square.js`的文件，其中包含以下代码。确保你包含了不合适的制表符：

```js
var square = x => x * x;
	console.log(square(5));
```

1.  在你的 npm `package.json`文件中创建一个`lint`脚本：

```js
  "scripts": {
   "lint": "prettier --write src/**/*.js"
  },
```

1.  接下来，我们将通过从命令行运行新脚本来测试和演示`prettier --write`：

```js
npm run lint
```

1.  在文本编辑器中打开`src/square.js`，你会看到不合适的制表符已被移除：![图 6.1：不合适的制表符已被移除](img/C14587_06_01.jpg)

###### 图 6.1：不合适的制表符已被移除

1.  接下来，回到`package.json`，扩展我们的 lint 脚本，在`prettier`完成后运行`eslint`：

```js
  "scripts": {
   "lint": "prettier --write src/**/*.js && eslint src/*.js"
  },
```

1.  在命令行中再次运行`npm run lint`。你将因`square.js`中的代码格式而遇到一个 linting 错误：

```js
> prettier --write src/**/*.js && eslint src/*.js
src/square.js 49ms
/home/philip/packt/lesson_6/lint/src/square.js
  1:1  error   Unexpected var, use let or const instead  no-var
  2:1  warning  Unexpected console statement          no-console
  2 problems (1 error, 1 warning)
  1 error and 0 warnings potentially fixable with the --fix option.
```

上述脚本产生了一个错误和一个警告。错误是由于在可以使用`let`或`const`的情况下使用`var`。尽管在这种特殊情况下应该使用`const`，因为`square`的值没有被重新赋值。警告是关于我们使用`console.log`，通常不应该在生产代码中使用，因为这会使在发生错误时难以调试控制台输出。

1.  打开`src/example.js`，并按照下图所示，在第 1 行将`var`更改为`const`：![图 6.2：将 var 语句替换为 const](img/C14587_06_02.jpg)

###### 图 6.2：将 var 语句替换为 const

1.  现在再次运行`npm run lint`。现在你应该只会收到警告：

```js
> prettier --write src/**/*.js && eslint src/*.js
src/js.js 48ms
/home/philip/packt/lesson_6/lint/src/js.js
  2:1  warning  Unexpected console statement  no-console
  1 problem (0 errors, 1 warning)
```

在这个练习中，我们安装并设置了 Prettier 以进行自动代码格式化，并使用 ESLint 检查我们的代码是否存在常见的不良实践。

## 单元测试

**单元测试**是一种自动化软件测试，用于检查某个软件中的单个方面或功能是否按预期工作。例如，计算器应用程序可能被分成处理应用程序的图形用户界面（GUI）的函数和负责每种类型的数学计算的另一组函数。

在这样的计算器中，可以设置单元测试来确保每个数学函数按预期工作。这种设置使我们能够快速发现任何由于任何更改而导致的不一致结果或损坏函数。例如，这样一个计算器的测试文件可能包括以下内容：

```js
test('Check that 5 plus 7 is 12', () => {
  expect(math.add(5, 7)).toBe(12);
});
test('Check that 10 minus 3 is 7', () => {
  expect(math.subtract(10, 3)).toBe(7);
});
test('Check that 5 multiplied by 3 is 15', () => {
  expect(math.multiply(5, 3).toBe(15);
});
test('Check that 100 divided by 5 is 20', () => {
  expect(math.multiply(100, 5).toBe(20);
});
test('Check that square of 5 is 25', () => {
  expect(math.square(5)).toBe(25);
});
```

前面的测试将在每次更改代码库时运行，并被检入版本控制。通常，当更新用于多个地方的函数并引发连锁反应导致某些其他函数损坏时，错误会意外地出现。如果发生这样的更改，并且前面的某个语句变为假（例如，5 乘以 3 返回 16 而不是 15），我们将立即能够将新的代码更改与损坏联系起来。

这是一种非常强大的技术，在已经设置好测试的环境中可能被认为是理所当然的。在没有这样一个系统的工作环境中，开发人员的更改或软件依赖项的更新可能会意外地破坏现有的函数并提交到源代码控制中。后来，发现了错误，并且很难将损坏的函数与导致它的代码更改联系起来。

还要记住，单元测试确保某个子单元的功能，但不确保整个项目的功能（其中多个函数一起工作以产生结果）。这就是集成测试发挥作用的地方。我们将在本章后面探讨集成测试。

### 练习 30：设置 Jest 测试以测试计算器应用程序

在这个练习中，我们将演示如何使用 Jest 设置单元测试，Jest 是 JavaScript 生态系统中最流行的测试框架。我们将继续使用计算器应用程序的示例，并为一个接受一个数字并输出其平方的函数设置自动化测试。

#### 注意

此练习的代码文件可以在[`github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson06/Exercise30`](https://github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson06/Exercise30)找到。

执行以下步骤以完成练习：

1.  在命令行中，导航到`Exercise30/start`练习文件夹。该文件夹包括一个包含我们将运行测试的代码的`src`文件夹。

1.  通过输入以下命令来初始化一个`npm`项目：

```js
npm init -y
```

1.  使用以下命令安装 Jest，使用`--save-dev`标志（表示该依赖项对开发而非生产是必需的）：

```js
npm install --save-dev jest
```

1.  创建一个名为`__tests__`的文件夹。这是 Jest 查找测试的默认位置：

```js
mkdir __tests__
```

1.  现在我们将在`__tests__/math.test.js`中创建我们的第一个测试。它应该导入`src/math.js`并确保运行`math.square(5)`返回`25`：

```js
const math = require('./../src/math.js');
test('Check that square of 5 is 25', () => {
  expect(math.square(5)).toBe(25);
});
```

1.  打开`package.json`并修改测试脚本，使其运行`jest`。注意以下截图中的`scripts`部分：![图 6.3：修改后的测试脚本，使其运行 Jest](img/C14587_06_03.jpg)

###### 图 6.3：修改后的测试脚本，使其运行 Jest

1.  在命令行中，输入`npm run test`。这应该返回一条消息，告诉我们找到了错误的值，如下面的代码所示：

```js
FAIL  __test__/math.test.js
  ✕ Check that square of 5 is 25 (17ms)
  ● Check that square of 5 is 25
   expect(received).toBe(expected) // Object.is equality
   Expected: 25
   Received: 10
    2 | 
    3 | test('Check that square of 5 is 25', () => {
   > 4 |  expect(math.square(5)).toBe(25);
      |                  ^
    5 | });
    6 | 
    at Object.toBe (__test__/math.test.js:4:26)
Test Suites: 1 failed, 1 total
Tests:     1 failed, 1 total
Snapshots:  0 total
Time:      1.263s
```

这个错误是因为起始代码故意在`square`函数中包含了一个错误。我们没有将数字乘以自身，而是将值加倍。请注意，接收到的答案数量是`10`。

1.  通过打开文件并修复`square`函数来修复错误。它应该像下面的代码一样将`x`相乘，而不是将其加倍：

```js
const square = (x) => x * x;
```

1.  修复了我们的代码后，让我们再次用`npm run test`进行测试。你应该会得到一个成功的消息，如下所示：

![图 6.4：使用 npm run test 进行测试后显示的成功消息](img/C14587_06_04.jpg)

###### 图 6.4：使用 npm run test 进行测试后显示的成功消息

在这个练习中，我们设置了一个 Jest 测试，以确保用输入 5 运行我们的`square`函数返回 25。我们还看了一下当代码中返回错误值时会发生什么，比如返回 10 而不是 25。

## 集成测试

因此，我们已经讨论了单元测试，当项目的代码发生变化时，它们非常有用，可以帮助找到错误的原因。然而，也有可能项目通过了所有的单元测试，但并不像预期的那样工作。这是因为整个项目包含了将我们的函数粘合在一起的额外逻辑，以及静态组件，如 HTML、数据和其他工件。

**集成测试**可以用来确保项目在更高层次上工作。例如，虽然我们的单元测试直接调用`math.square`等函数，但集成测试将测试多个功能一起工作以获得特定结果。

通常，这意味着将多个模块组合在一起，或者与数据库或其他外部组件或 API 进行交互。当然，集成更多部分意味着集成测试需要更长的时间，因此它们应该比单元测试更少地使用。集成测试的另一个缺点是，当一个测试失败时，可能有多种可能性作为原因。相比之下，失败的单元测试通常很容易修复，因为被测试的代码位于指定的位置。

### 练习 31：使用 Jest 进行集成测试

在这个练习中，我们将继续上次 Jest 练习的内容，上次我们测试了`square`函数对 5 的响应是否返回 25。在这个练习中，我们将继续添加一些新的测试，使用我们的函数相互结合：

1.  在命令行中，导航到`Exercise31/start`练习文件夹，并使用`npm`安装依赖项：

```js
npm install
```

1.  创建一个名为`__tests__`的文件夹：

```js
mkdir __tests__
```

1.  创建一个名为`__tests__/math.test.js`的文件。然后，在顶部导入`math`库：

```js
const math = require('./../src/math.js');
```

1.  与上一个练习类似，我们将添加一个测试。然而，这里的主要区别是我们将多个函数组合在一起：

```js
test('check that square of result from 1 + 1 is 4', () => {
  expect(math.square(math.add(1,1))).toBe(4);
});
```

1.  在前面的测试中添加一个计时器来测量性能：

```js
test('check that square of result from 1 + 1 is 4', () => {
  const start = new Date();
  expect(math.square(math.add(1,1))).toBe(4);
  expect(new Date() - start).toBeLessThan(5000);
});
```

1.  现在，通过运行`npm test`来测试一切是否正常运行：

![图 6.5：运行 npm test 以确保一切正常](img/C14587_06_05.jpg)

###### 图 6.5：运行 npm test 以确保一切正常

你应该看到与前面图中类似的输出，每个测试都通过了预期的结果。

应该注意，这些集成测试有点简单。在实际情况下，集成测试结合了我们之前演示的不同来源的函数。例如，当你有多个由不同团队创建的组件时，集成测试可以确保一切都能一起工作。通常，错误可能是由简单的事情引起的，比如更新外部库。

这个想法是你的应用程序的多个部分被集成在一起，这样你就有更大的机会找到哪里出了问题。

### 代码性能斐波那契示例

通常，一个问题有不止一种解决方案。虽然所有的解决方案可能会返回相同的结果，但它们的性能可能不同。例如，考虑获取斐波那契数列的第 n 个数字的问题。斐波那契是一个数学模式，其中序列中的下一个数字是前两个数字的和（1, 1, 2, 3, 5, 8, 13, …）。

考虑以下解决方案，其中斐波那契递归调用自身：

```js
function fib(n) {
  return (n<=1) ? n : fib(n - 1) + fib(n - 2);
}
```

前面的例子说明，如果我们想要递归地得到斐波那契数列的第 n 个数字，那么就得到`n`减一的斐波那契加上`n`减二的斐波那契，除非`n`为 1，此时返回 1。它可以返回任何给定数字的正确答案。然而，随着`n`的增加，执行时间呈指数增长。

要查看这个算法的执行速度有多慢，将`fib`函数添加到一个新文件中，并使用以下方式通过控制台记录结果：

```js
console.log(fib(37));
```

接下来，在命令行中运行以下命令（`time`应该在大多数 Unix 和基于 Mac 的环境中可用）：

```js
time node test.js
```

在特定的笔记本电脑上，我得到了以下结果，表明斐波那契的第 37 位数字是`24157817`，执行时间为 0.441 秒：

```js
24157817
real 0m0.441s
user 0m0.438s
sys 0m0.004s
```

现在打开同一个文件，并将`37`改为`44`。然后再次运行相同的`time node test`命令。在我的情况下，仅增加了 7，执行时间就增加了 20 倍：

```js
701408733
real 0m10.664s
user 0m10.653s
sys 0m0.012s
```

我们可以以更高效的方式重写相同的算法，以增加大数字的速度：

```js
function fibonacciIterator(a, b, n) {
  return n === 0 ? b : fibonacciIterator((a+b), a, (n-1));
}
function fibonacci(n) {
  return fibonacciIterator(1, 0, n);
}
```

尽管看起来更复杂，但由于执行速度快，这种生成斐波那契数的方法更优越。

Jest 测试的一个缺点是，鉴于前面的情景，斐波那契的慢速和快速版本都会通过。然而，在现实世界的应用程序中，慢速版本显然是不可接受的，因为需要快速处理。

为了防范这种情况，您可能希望添加一些基于性能的测试，以确保函数在一定时间内完成。以下是一个示例，创建一个自定义计时器，以确保函数在 5 秒内完成：

```js
test('Timer - Slow way of getting Fibonacci of 44', () => {
  const start = new Date();
  expect(fastFib(44)).toBe(701408733);
  expect(new Date() - start).toBeLessThan(5000);
});
```

#### 注意：Jest 的未来版本

手动为所有函数添加计时器可能有些麻烦。因此，在 Jest 项目中有讨论，以创建更简单的语法来实现之前所做的事情。

要查看与此语法相关的讨论以及是否已解决，请在 GitHub 上的 Jest 的问题＃6947 中查看[`github.com/facebook/jest/issues/6947`](https://github.com/facebook/jest/issues/6947)。

### 练习 32：使用 Jest 确保性能

在这个练习中，我们将使用之前描述的技术来测试获取斐波那契的两种算法的性能：

1.  在命令行中，导航到`Exercise32/start`练习文件夹，并使用`npm`安装依赖项：

```js
npm install
```

1.  创建一个名为`__tests__`的文件夹：

```js
mkdir __tests__
```

1.  创建一个名为`__tests__/fib.test.js`的文件。在顶部，导入快速和慢速的斐波那契函数（这些已经在`start`文件夹中创建）：

```js
const fastFib = require('./../fastFib');
const slowFib = require('./../slowFib');
```

1.  为快速斐波那契添加一个测试，创建一个计时器，并确保计时器运行时间不超过 5 秒：

```js
test('Fast way of getting Fibonacci of 44', () => {
  const start = new Date();
  expect(fastFib(44)).toBe(701408733);
  expect(new Date() - start).toBeLessThan(5000);
});
```

1.  接下来，为慢速斐波那契添加一个测试，同时检查运行时间是否少于 5 秒：

```js
test('Timer - Slow way of getting Fibonacci of 44', () => {
  const start = new Date();
  expect(slowFib(44)).toBe(701408733);
  expect(new Date() - start).toBeLessThan(5000);
});
```

1.  从命令行中，使用`npm test`命令运行测试：

![图 6.6：斐波那契测试的结果](img/C14587_06_06.jpg)

###### 图 6.6：斐波那契测试的结果

注意前面提到的关于计时器的错误响应。函数运行时间的预期结果应该在 5,000 毫秒以下，但在我的情况下，我实际收到了 10,961。根据您的计算机速度，您可能会得到不同的结果。如果您没有收到错误，可能是因为您的计算机速度太快，完成时间少于 5,000 毫秒。如果是这种情况，请尝试降低预期的最大时间以触发错误。

## 端到端测试

虽然集成测试结合了软件项目的多个单元或功能，**端到端测试**更进一步，模拟了软件的实际使用。

例如，虽然我们的单元测试直接调用了`math.square`等函数，端到端测试将加载计算器的图形界面，并模拟按下一个数字，比如 5，然后是平方按钮。几秒钟后，端到端测试将查看图形界面中的结果，并确保它等于预期的 25。

由于开销较大，端到端测试应该更加节制地使用，但它是测试过程中的一个很好的最后一步，以确保一切都按预期工作。相比之下，单元测试运行起来相对快速，因此可以更频繁地运行而不会拖慢开发速度。下图显示了测试的推荐分布：

![图 6.7：测试的推荐分布](img/C14587_06_07.jpg)

###### 图 6.7：测试的推荐分布

#### 注意：集成测试与端到端测试

值得注意的是，什么被认为是集成测试和端到端测试之间可能存在一些重叠。对于测试类型的解释可能会在不同公司之间有所不同。

传统上，测试被分类为单元测试或集成测试。随着时间的推移，其他分类变得流行，如系统测试、验收测试和端到端测试。因此，特定测试的类型可能会有重叠。

## Puppeteer

2018 年，谷歌发布了**Puppeteer** JavaScript 库，大大提高了在基于 JavaScript 的项目上设置端到端测试的便利性。Puppeteer 是 Chrome 浏览器的无头版本，意味着它没有 GUI 组件。这是至关重要的，因为这意味着我们使用完整的 Chrome 浏览器来测试我们的应用，而不是模拟。

Puppeteer 可以通过类似于 jQuery 的语法进行控制，其中可以通过 ID 或类选择 HTML 页面上的元素并与之交互。例如，以下代码打开 Google News，找到一个`.rdp59b`类，点击它，等待 3 秒，最后截取屏幕：

```js
(async() => {
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  await page.goto('http://news.google.com');
  const more = await page.$(".rdp59b");
  more.click();
  await page.waitFor(3000);
  await page.screenshot({path: 'news.png'});
  await browser.close();
})();
```

请记住，在上面的示例中，我们选择了一个看起来是自动生成的`.rdp59b`类；因此，很可能这个类将来会发生变化。如果类名发生变化，脚本将不再起作用。

如果在阅读本文时，您发现前面的脚本不起作用，我挑战您更新它。在使用 Puppeteer 时，其中一个最好的工具是 Chrome DevTools。我的常规工作流程是转到我为其编写脚本的网站，并右键单击我将要定位的元素，如下图所示：

![图 6.8：在 Chrome 中右键单击进行检查](img/C14587_06_08.jpg)

###### 图 6.8：在 Chrome 中右键单击进行检查

一旦单击**检查**，DOM 资源管理器将弹出，您将能够看到与元素相关的任何类或 ID：

![图 6.9：Chrome DevTools 中的 DOM 资源管理器](img/C14587_06_09.jpg)

###### 图 6.9：Chrome DevTools 中的 DOM 资源管理器

#### 注意：Puppeteer 用于 Web 抓取和自动化

除了用于编写端到端测试，Puppeteer 还可以用于 Web 抓取和自动化。几乎可以在普通浏览器中完成的任何事情都可以自动化（只要有正确的代码）。

除了能够通过选择器在页面上选择元素之外，正如我们之前所看到的，Puppeteer 还可以完全访问键盘和鼠标模拟。因此，诸如自动化基于 Web 的游戏和日常任务等更复杂的事情是可能的。一些人甚至成功地使用它绕过了验证码等东西。

### 练习 33：使用 Puppeteer 进行端到端测试

在这个练习中，我们将使用 Puppeteer 手动打开一个基于 HTML/JavaScript 的计算器，并像最终用户一样使用它。我不想针对一个实时网站，因为它的内容经常会发生变化或下线。因此，我在项目文件的`Exercise33/start`中包含了一个 HTML 计算器。

您可以通过使用 npm 安装依赖项，运行`npm start`，然后在浏览器中转到`localhost:8080`来查看它：

![图 6.10：显示使用 Puppeteer 创建的计算器演示的网站](img/C14587_06_10.jpg)

###### 图 6.10：显示使用 Puppeteer 创建的计算器演示的网站

在这个练习中，我们将创建一个脚本，打开网站，按下按钮，然后检查网站的正确结果。我们不仅仅是检查函数的输出，而是列出在网站上执行的操作，并指定要用作我们测试对象的值的 HTML 选择器。

执行以下步骤完成练习：

1.  打开`Exercise33/start`文件夹并安装现有的依赖项：

```js
npm install
```

1.  安装所需的`jest`，`puppeteer`和`jest-puppeteer`包：

```js
npm install --save-dev jest puppeteer jest-puppeteer
```

1.  打开`package.json`并配置 Jest 使用`jest-puppeteer`预设，这将自动设置 Jest 以与 Puppeteer 一起工作：

```js
  "jest": {
   "preset": "jest-puppeteer"
  },
```

1.  创建一个名为`jest-puppeteer.config.js`的文件，并添加以下内容：

```js
module.exports = {
  server: {
   command: 'npm start',
   port: 8080,
  },
}
```

前面的配置将确保在测试阶段之前运行`npm start`命令。它还告诉 Puppeteer 在`port: 8080`上查找我们的 Web 应用程序。

1.  创建一个名为`__tests__`的新文件夹，就像我们在之前的示例中所做的那样：

```js
mkdir __test__
```

1.  在`__tests__`文件夹中创建一个名为`test.test.js`的文件，其中包含以下内容：

```js
describe('Calculator', () => {
  beforeAll(async () => {
   await page.goto('http://localhost:8080')
  })
  it('Check that 5 times 5 is 25', async () => {
   const five = await page.$("#five");
   const multiply = await page.$("#multiply");
   const equals = await page.$("#equals");
   await five.click();
   await multiply.click();
   await five.click();
   await equals.click();
   const result = await page.$eval('#screen', e => e.innerText);
   expect(result).toMatch('25');
  })
})
```

前面的代码是一个完整的端到端测试，用于将 5 乘以 5 并确认界面返回的答案为 25。在这里，我们打开本地网站，按下五，按下乘，按下五，按下等于，然后检查具有 ID 为`screen`的`div`的值。

1.  使用`npm`运行测试：

![图 6.11：运行计算器脚本后的输出](img/C14587_06_11.jpg)

###### 图 6.11：运行计算器脚本后的输出

您应该看到一个结果，如前图所示，输出为 25。

### Git 钩子

这里讨论的测试和 linting 命令对于维护和改进代码质量和功能非常有用。然而，在实际开发的热情中，我们的重点是特定问题和截止日期，很容易忘记运行 linting 和测试命令。

解决这个问题的一个流行方法是使用 Git 钩子。Git 钩子是 Git 版本控制系统的一个特性。**Git 钩子**指定要在 Git 过程的某个特定点运行的终端命令。Git 钩子可以在提交之前运行；在用户通过拉取更新时运行；以及在许多其他特定点运行。可以在[`git-scm.com/docs/githooks`](https://git-scm.com/docs/githooks)找到可能的 Git 钩子的完整列表。

对于我们的目的，我们将只关注使用预提交钩子。这将允许我们在提交代码到源代码之前找到任何格式问题。

#### 注意：探索 Git

探索可能的 Git 钩子以及它们通常如何使用的另一个有趣的方法是打开任何 Git 版本控制项目并查看`hooks`文件夹。

默认情况下，任何新的`.git`项目都将在`.git/hooks`文件夹中包含大量的示例。探索它们的内容，并通过使用以下模式重命名它们来触发它们：

`<hook-name>.sample to <hook-name>`

### 练习 34：设置本地 Git 钩子

在这个练习中，我们将设置一个本地 Git 钩子，在我们允许使用 Git 提交之前运行`lint`命令：

1.  在命令行中，导航到`Exercise34/start`练习文件夹并安装依赖项：

```js
npm install
```

1.  将文件夹初始化为 Git 项目：

```js
git init
```

1.  创建`.git/hooks/pre-commit`文件，其中包含以下内容：

```js
#!/bin/sh
npm run lint
```

1.  如果在基于 OS X 或 Linux 的系统上，通过运行以下命令使文件可执行（在 Windows 上不需要）：

```js
chmod +x .git/hooks/pre-commit
```

1.  我们现在将通过进行提交来测试钩子：

```js
git add package.json
git commit -m "testing git hook"
```

以下是前面代码的输出：

![图 6.12：提交到 Git 之前运行的 Git 钩子](img/C14587_06_12.jpg)

###### 图 6.12：提交到 Git 之前运行 Git 钩子

在您的代码提交到源代码之前，您应该看到`lint`命令正在运行，如前面的屏幕截图所示。

1.  接下来，让我们通过添加一些代码来测试失败，这些代码将生成 linting 错误。通过在您的`src/js.js`文件中添加以下行来修改：

```js
      let number = square(5);
```

确保在上一行中保留不必要的制表符，因为这将触发 lint 错误。

1.  重复添加文件并提交的过程：

```js
git add src/js.js
git commit -m "testing bad lint"
```

以下是上述代码的输出：

![图 6.13：提交代码到 git 之前的失败 linting](img/C14587_06_13.jpg)

###### 图 6.13：提交代码到 git 之前的失败 linting

您应该看到`lint`命令像以前一样运行；但是，在运行后，由于 Git 钩子返回错误，代码不会像上次那样被提交。

### 使用 Husky 共享 Git 钩子

要注意的一个重要因素是，由于这些钩子位于`.git`文件夹本身内部，它们不被视为项目的一部分。因此，它们不会被共享到您的中央 Git 存储库供协作者使用。

然而，Git 钩子在协作项目中最有用，新开发人员可能不完全了解项目的约定。当新开发人员克隆项目，进行一些更改，尝试提交，并立即根据 linting 和测试获得反馈时，这是一个非常方便的过程。

`husky`节点库是基于这个想法创建的。它允许您使用一个名为`.huskyrc`的单个配置文件在源代码中跟踪您的 Git 钩子。当新开发人员安装项目时，钩子将处于活动状态，开发人员无需做任何操作。

### 练习 35：使用 Husky 设置提交钩子

在这个练习中，我们将设置一个 Git 钩子，它与*练习 34，设置本地 Git 钩子*中的钩子做相同的事情，但具有可以在团队中共享的优势。通过使用`husky`库而不是直接使用`git`，我们将确保任何克隆项目的人也有在提交任何更改之前运行`lint`的钩子：

1.  在命令行中，导航到`Exercise35/start`练习文件夹并安装依赖项：

```js
npm install
```

1.  创建一个名为`.huskyrc`的文件，其中包含以下内容：

```js
{
  "hooks": {
   "pre-commit": "npm run lint"
  }
}
```

前面的文件是这个练习的最重要部分，因为它确切地定义了在 Git 过程的哪个时刻运行什么命令。在我们的情况下，在将任何代码提交到源代码之前，我们运行`lint`命令。

1.  通过运行`git init`将文件夹初始化为 Git 项目：

```js
git init
```

1.  使用`npm`安装 Husky：

```js
npm install --save-dev husky
```

1.  对`src/js.js`进行更改，以便用于我们的测试提交。例如，我将添加以下注释：![图 6.14：测试提交注释](img/C14587_06_14.jpg)

###### 图 6.14：测试提交注释

1.  现在，我们将运行一个测试，确保它像之前的示例一样工作：

```js
git add src/js.js
git commit -m "test husky hook"
```

以下是上述代码的输出：

![图 6.15：提交测试 husky 钩子后的输出](img/C14587_06_15.jpg)

###### 图 6.15：提交测试 husky 钩子后的输出

我们收到关于我们使用`console.log`的警告，但是出于我们的目的，您可以忽略这一点。主要问题是我们已经使用 Husky 设置了我们的 Git 钩子，因此安装项目的任何其他人也将设置好钩子，而不是我们直接在 Git 中设置它们。

#### 注意：初始化 Husky

请注意，`npm install --save-dev husky`是在创建 Git 存储库后运行的。当您安装 Husky 时，它会运行必需的命令来设置您的 Git 钩子。但是，如果项目不是 Git 存储库，则无法运行。

如果您遇到与此相关的任何问题，请在初始化 Git 存储库后尝试重新运行`npm install --save-dev husky`。

### 练习 36：使用 Puppeteer 按文本获取元素

在这个练习中，我们将编写一个 Puppeteer 测试，验证一个小测验应用程序是否正常工作。如果你进入练习文件夹并找到*练习 36*的起点，你可以运行`npm start`来查看我们将要测试的测验：

![图 6.16：Puppeteer 显示一个小测验应用程序](img/C14587_06_16.jpg)

###### 图 6.16：Puppeteer 显示一个小测验应用程序

在这个应用程序中，点击问题的正确答案会使问题消失，分数增加一：

1.  在命令行中，导航到`Exercise36/start`练习文件夹并安装依赖项：

```js
npm install --save-dev jest puppeteer jest-puppeteer
```

1.  通过修改`scripts`部分，向`package.json`文件添加一个`test`脚本，使其看起来像下面这样：

```js
  "scripts": {
   "start": "http-server",
   "test": "jest"
  },
```

1.  在`package.json`中添加一个 Jest 部分，告诉 Jest 使用 Puppeteer 预设：

```js
  "jest": {
   "preset": "jest-puppeteer"
  },
```

1.  创建一个名为`jest-puppeteer.config.js`的文件，在其中告诉 Jest 在运行任何测试之前打开我们的测验应用程序：

```js
module.exports = {
  server: {
   command: 'npm start',
   port: 8080,
  },
}
```

1.  创建一个名为`__test__`的文件夹，我们将把我们的 Jest 测试放在其中：

```js
mkdir __test__
```

1.  在名为`quiz.test.js`的文件夹中创建一个测试。它应该包含以下内容来初始化我们的测试：

```js
describe('Quiz', () => {
  beforeAll(async () => {
   await page.goto('http://localhost:8080')
  })
// tests will go here
})
```

1.  接下来，用我们测验中的第一个问题的测试替换前面代码中的注释：

```js
  it('Check question #1', async () => {
   const q1 = await page.$("#q1");
   let rightAnswer = await q1.$x("//button[contains(text(), '10')]");
   await rightAnswer[0].click();
   const result = await page.$eval('#score', e => e.innerText);
   expect(result).toMatch('1');
  })
```

注意我们使用的`q1.$x("//button[contains(text(), '10')]")`。我们不是使用 ID，而是在答案中搜索包含文本`10`的按钮。当解析一个网站时，这可能非常有用，该网站没有在您需要交互的元素上使用 ID。

1.  在最后一步添加了以下测试。我们将添加三个新测试，每个问题一个：

```js
  it('Check question #2', async () => {
   const q2 = await page.$("#q2");
   let rightAnswer = await q2.$x("//button[contains(text(), '36')]");
   await rightAnswer[0].click();
   const result = await page.$eval('#score', e => e.innerText);
   expect(result).toMatch('2');
  })
  it('Check question #3', async () => {
   const q3 = await page.$("#q3");
   let rightAnswer = await q3.$x("//button[contains(text(), '9')]");
   await rightAnswer[0].click();
   const result = await page.$eval('#score', e => e.innerText);
   expect(result).toMatch('3');
  })
  it('Check question #4', async () => {
   const q4 = await page.$("#q4");
   let rightAnswer = await q4.$x("//button[contains(text(), '99')]");
   await rightAnswer[0].click();
   const result = await page.$eval('#score', e => e.innerText);
   expect(result).toMatch('4');
  })
```

注意每个测试的底部都有一个预期结果，比上一个高一个；这是我们在页面上跟踪分数。如果一切正常，第四个测试将找到一个分数为 4。

1.  最后，返回到命令行，以便我们可以确认正确的结果。运行以下`test`命令：

```js
npm test
```

以下是前面代码的输出：

![图 6.17：命令行确认正确的结果](img/C14587_06_17.jpg)

###### 图 6.17：命令行确认正确的结果

如果一切正确，运行`npm test`应该看到四个通过的测试作为响应。

### 活动 7：将所有内容组合在一起

在这个活动中，我们将结合本章的几个方面。从使用 HTML/JavaScript 构建的预先构建的计算器开始，你的任务是：

+   创建一个`lint`命令，使用`eslint-config-airbnb-base`包检查项目是否符合`prettier`和`eslint`，就像在之前的练习中所做的那样。

+   使用`jest`安装`puppeteer`并在`package.json`中创建一个运行`jest`的`test`命令。

+   创建一个 Puppeteer 测试，使用计算器计算 777 乘以 777，并确保返回的答案是 603,729。

+   创建另一个 Puppeteer 测试来计算 3.14 除以 2，并确保返回的答案是 1.57。

+   安装并设置 Husky，在使用 Git 提交之前运行 linting 和测试命令。

执行以下步骤完成活动（高级步骤）：

1.  安装在 linting 练习中列出的开发人员依赖项（`eslint`，`prettier`，`eslint-config-airbnb-base`，`eslint-config-prettier`，`eslint-plugin-jest`和`eslint-plugin-import`）。

1.  添加一个`eslint`配置文件`.eslintrc`。

1.  添加一个`.prettierignore`文件。

1.  在`package.json`文件中添加一个`lint`命令。

1.  打开`assignment`文件夹，并安装使用 Puppeteer 和 Jest 的开发人员依赖项。

1.  通过修改`package.json`文件，添加一个选项告诉 Jest 使用`jest-puppeteer`预设。

1.  在`package.json`中添加一个`test`脚本来运行`jest`。

1.  创建一个`jest-puppeteer.config.js`来配置 Puppeteer。

1.  在`__tests__/calculator.js`创建一个测试文件。

1.  创建一个`.huskyrc`文件。

1.  通过运行`npm install --save-dev husky`安装`husky`作为开发人员依赖项。

**预期输出**

![图 6.18：最终输出显示 calc.test 通过](img/C14587_06_18.jpg)

###### 图 6.18：最终输出显示 calc.test 通过

完成任务后，您应该能够运行`npm run` `lint`命令和`npm test`命令，并像前面的截图中一样通过测试。

#### 注意

这个活动的解决方案可以在 602 页找到。

## 总结

在本章中，我们着重介绍了自动化测试的代码质量方面。我们从清晰命名和熟悉语言的行业惯例的基础知识开始。通过遵循这些惯例并清晰地书写，我们能够使我们的代码更易读和可重用。

从那里开始，我们看了一下如何使用一些流行的工具（包括 Prettier、ESLint、Jest、Puppeteer 和 Husky）在 Node.js 中创建 linting 和测试命令。

除了设置测试之外，我们还讨论了测试的类别和它们的用例。我们进行了单元测试，确保单个函数按预期工作，并进行了集成测试，将多个函数或程序的方面结合在一起，以确保它们一起工作。然后，我们进行了端到端测试，打开应用程序的界面并与其进行交互，就像最终用户一样。 

最后，我们看了如何通过 Git 钩子自动运行我们的 linting 和测试脚本。

在下一章中，我们将研究构造函数、promises 和 async/await。我们将使用其中一些技术以一种现代化的方式重构 JavaScript，利用 ES6 中提供的新功能。
