# 第六章：模块化 JavaScript

## 学习目标

在本章结束时，您将能够：

+   在 JavaScript 中导入和导出函数和对象以实现代码的可重用性

+   使用 JavaScript ES6 类来减少代码复杂性

+   在 JavaScript 中实现面向对象编程概念

+   使用封装为对象创建私有变量

+   使用 Babel 将 ES6 转换为通用 JavaScript

+   在 JavaScript 中创建和发布 npm 包

+   使用组合性和策略结合模块创建更高级的模块。

在本章中，我们将学习现代 JavaScript 中可重用代码的重要性，以及 ES6 如何引入了用于轻松创建和使用模块的语法。我们将创建一个 JavaScript 模块，可以被 API 的不同端点导入和使用。

## 介绍

在上一章中，我们使用 Node.js 和 Express 构建了一个 API。我们讨论了设计 API 结构、HTTP 方法和**JSON Web Token**（**JWT**）身份验证。在本章中，我们将研究 JavaScript 模块和基于模块的设计的各个方面。

模块对于编程生产力很重要，将软件分解为可重用的模块。模块化设计鼓励开发人员将软件构建成小的、单一焦点的组件。您可能熟悉流行的 UI 库，如 Bootstrap、Material-UI 和 jQuery UI。这些都是一组组件 - 专门构建的最小图形元素，可以在许多情况下使用。

由于广泛使用外部库来处理图形元素和编程方面，大多数开发人员已经熟悉了模块的使用。也就是说，使用模块比创建模块或以模块化方式编写应用程序要容易得多。

#### 注意组件、模块和 ES6 模块

关于这些术语的确切用法和关系有各种不同的观点。在本章中，我们将组件称为可以在网站上使用的视觉小部件。

我们将把一个模块称为在一个文件中编写的源代码，以便在另一个文件中导入和使用。由于大多数组件都存在为可重用代码，通常通过脚本标签导入，我们将把它们视为模块。当然，当您导入 Bootstrap 库时，您导入了所有组件。也就是说，大多数库都提供了编译和导入所需的特定组件的能力 - 例如，[`getbootstrap.com/docs/3.4/customize/`](https://getbootstrap.com/docs/3.4/customize/)。

当我们提到 ES6 模块时，我们指的是 ES6 中添加的特定语法，允许在一个文件中导出一个模块，并在另一个文件中导入它。虽然 ES6 模块是 ES6 标准的一部分，但重要的是要记住它们目前不受浏览器支持。使用它们需要一个预编译步骤，我们将在本章中介绍。

JavaScript 的受欢迎程度和生产力的最近爆炸部分原因是**node 包管理器**（**npm**）生态系统。无论是使用 JavaScript 进行前端还是后端开发，您都可能在某个时候使用 npm。通过简单的`npm install`命令，开发人员可以获得数百个有用的包。

npm 现在已成为互联网上模块化代码的最大来源，超过任何编程语言。npm 现在包含了将近 50 亿个包。

npm 上的所有包本质上都是模块。通过将相关函数分组为一个模块，我们使得该功能可以在多个项目或单个项目的多个方面中重复使用。

所有在 npm 上的优秀包都是以一种使其在许多项目中易于重用的方式构建的。例如，一个很好的日期时间选择器小部件可以在成千上万个项目中使用，节省了许多开发时间，并且可能产生更好的最终产品。

在本节中，我们将讨论模块化的 JavaScript 以及如何通过以模块化的方式编写 JavaScript 来改进我们的代码。这包括导出和导入的基本语法，但除此之外，还有几种模式和技术可用于编写更好的模块，例如在模块开发中有用的面向对象编程的概念。然而，JavaScript 在技术上是原型导向的，这是一种与经典面向对象风格不同的特定风格的面向对象编程，它使用原型而不是类。我们将在本章后面讨论原型和类。

### 依赖关系和安全性

模块是一种强大的技术，但如果不小心使用，它们也可能失控。例如，添加到`node.js`项目中的每个包都包含自己的依赖关系。因此，重要的是要密切关注您正在使用的包，以确保您不会导入任何恶意内容。在网站[`npm.broofa.com`](http://npm.broofa.com)上有一个有用的工具，您可以在那里上传`package.json`文件并获得依赖关系的可视化。

如果我们以*第一章练习 1，使用 Express 创建项目并添加索引路由*中的`package.json`文件为例，它只包含四个`dependencies`：

```js
  "dependencies": {
   "express": "⁴.16.4",
   "express-validator": "⁵.3.1",
   "jwt-simple": "⁰.5.6",
   "mongodb": "³.2.3"
  }
```

然而，当我们上传这个`package.json`文件时，我们可以看到我们的 4 个依赖项在考虑子依赖时激增到了 60 多个：

![](img/C14587_05_01.jpg)

###### 图 5.1：package.json 中的 61 个依赖项

这突显了基于模块的设计所带来的风险，以及在制作和使用模块时需要深思熟虑的设计。糟糕编写的包或模块可能会产生意想不到的后果。近年来，有关广泛使用的包变得恶意的报道。例如，`event-stream`包在 2018 年的 2.5 个月内被下载了 800 多万次。发现这个曾经合法的模块已经更新，试图从用户的机器中窃取加密货币。除了安全风险和错误之外，还存在污染全局命名空间或降低父项目性能的风险。

#### 注意 npm audit

作为对恶意依赖或子依赖的情况的回应，npm 添加了一个`audit`命令，可以用来检查包的依赖关系，以查看已知为恶意的模块。在 Node.js 项目的目录中运行`npm audit`来检查项目的依赖关系。当您安装从 GitHub 等地方下载的项目时，该命令也会自动作为`npm install`的一部分运行。

### 模块化的其他成本

与模块化设计相关的其他成本包括：

+   加载多个部分的成本

+   坏模块的成本（安全性和性能）

+   使用的模块总量迅速增加

总的来说，这些成本通常是可以接受的，但应该谨慎使用。当涉及到加载许多模块所带来的开销时，预编译器（如`webpack`和`babel`）可以通过将整个程序转换为单个文件来帮助。

在创建模块或导入模块时需要牢记以下几点：

+   使用模块是否隐藏了重要的复杂性或节省了大量的工作？

+   模块是否来自可信任的来源？

+   它是否有很多子依赖？

以 npm 包`isarray`为例。该包包含一个简单的函数，只是运行：

```js
return toString.call(arr) == '[object Array]';
```

这是一个例子，第一个问题的答案是“使用模块是否隐藏了重要的复杂性？”不是。第二个问题 - “它是来自可信任的来源吗？”并不特别。最后，对于关于子依赖的最后一个问题的回答是不是 - 这是一件好事。鉴于这个模块的简单性，建议根据前面的单行编写自己的函数。

应避免随意安装增加项目复杂性而几乎没有好处的包。如果您考虑到了提到的三点，您可能不会觉得值得导入诸如`isarray`之类的包。

### 审查进口和出口

在上一节中，我们使用了导入和导出，但没有深入讨论这个主题。每当我们创建一个新的路由时，我们都会确保将其放在`routes`文件夹中的自己的文件中。如果您还记得，我们所有的路由文件都以导出`router`对象的行结束：

```js
module.exports = router;
```

我们还使用了 Node.js 内置的`require`函数来使用我们的路由：

```js
let light = require('./routes/devices/light');
```

### 关注点分离

在设计模块时，关键概念之一是**关注点分离**。关注点分离意味着我们应该将软件分成处理程序的单个关注点的部分。一个好的模块将专注于很好地执行单个功能方面。流行的例子包括：

+   MySQL - 一个具有多种方法连接和使用 MySQL 数据库的包

+   Lodash - 一个用于高效解析和处理数组、对象和字符串的包

+   Moment - 一个用于处理日期和时间的流行包

在这些包或我们自己的项目中，通常还会进一步分成子模块。

#### 注意 ES6

在之前的章节中，我们已经使用了一些 ES6 的特性，但是作为提醒，ES6，或者更长的 ECMAScript，是欧洲计算机制造商协会脚本的缩写。ECMA 是负责标准化标准的组织，包括 2015 年标准化的新版本 JavaScript。

## ES6 模块

在使用 Node.js 编写 JavaScript 时，长期以来一直使用内置的`require()`函数来导入模块的能力。由于这个功能很有用，许多前端开发人员开始利用它，通过使用诸如 Babel 之类的编译器对他们的 JavaScript 进行预处理。JavaScript 预编译器处理通常无法在大多数浏览器上运行的代码，并生成一个兼容的新 JavaScript 文件。

由于 JavaScript 中对导入样式函数的需求很大，它最终被添加到了 ES6 版本的语言中。在撰写本文时，大多数浏览器的最新版本几乎完全兼容 ES6。然而，不能认为使用`import`是理所当然的，因为许多设备将继续运行多年前的旧版本。

ES6 的快速标准化告诉我们，未来，ES6 的导入将是最流行的方法。

在上一章中，我们使用了 Node.js 的`require`方法来导入模块。例如，看看这一行：

```js
const express = require('express');
```

另一方面，ES6 的`import`函数具有以下语法：

```js
import React from 'react';
```

ES6 的`import`函数还允许您导入模块的子部分，而不是整个模块。这是 ES6 的`import`相对于 Node.js 的`require`函数的一个能力。导入单个组件有助于节省应用程序中的内存。例如，如果我们只想使用 React 版本的 Bootstrap 中的`button`组件，我们可以只导入那个：

```js
import { Button } from 'reactstrap';
```

如果我们想要导入额外的组件，我们只需将它们添加到列表中：

```js
import { Button, Dropdown, Card } from 'reactstrap';
```

#### 注意 React

如果您曾经使用过流行的前端框架 React，您可能已经看到过这种导入方式。该框架以模块化为重点而闻名。它将交互式前端元素打包为组件。

在传统的纯 JavaScript/HTML 中，项目通常被分成 HTML/CSS/JavaScript，各种组件分散在这些文件中。相反，React 将元素的相关 HTML/CSS/JavaScript 打包到单个文件中。然后将该组件导入到另一个 React 文件中，并在应用程序中用作元素。

### 练习 22：编写一个简单的 ES6 模块

#### 注意

本章有一个起始点目录，可以在[`github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson05/start`](https://github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson05/start)找到。

此练习的完成代码可以在[`github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson05/Exercise22`](https://github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson05/Exercise22)找到。

在这个练习中，我们将使用 ES6 语法导出和导入一个模块：

1.  切换到`/Lesson_05/start/`目录；我们将使用这个作为起点。

1.  使用`npm install`安装项目依赖项。

1.  创建`js/light.js`文件，其中包含以下代码：

```js
let light = {};
light.state = true;
light.level = 0.5;
var log = function () {
  console.log(light);
};
export default log;
```

1.  打开名为`js/viewer.js`的文件。这是将在我们页面上运行的 JavaScript。在文件顶部添加：

```js
import light from './light.js';
```

1.  在`js/viewer.js`的底部，添加：

```js
light();
```

1.  `js/viewer.js`已经包含在`index.html`中，所以现在我们可以使用`npm start`启动程序。

1.  在服务器运行时，打开一个 Web 浏览器，转到`localhost:8000`。一旦到达那里，按下*F12*打开开发者工具。

如果您做得没错，您应该在 Google Chrome 控制台中看到我们的对象被记录：

![图 5.2：在 Google Chrome 控制台中记录的对象](img/C14587_05_02.jpg)

###### 图 5.2：在 Google Chrome 控制台中记录的对象

### JavaScript 中的对象

如果您已经写了一段时间的 JavaScript，您很快就会遇到`object`类型。JavaScript 是使用原型设计的，这是一种基于对象的编程类型。JavaScript 中的对象是一个可以包含多个属性的变量。这些属性可以指向值、子对象，甚至函数。

JavaScript 程序中的每个变量都是对象或原始值。原始值是一种更基本的类型，只包含单个信息片段，没有属性或方法。使 JavaScript 变得更加复杂并使对象变得更加重要的是，即使是最基本的类型，如字符串和数字，一旦分配给变量，也会被包装在对象中。

例如：

```js
let myString = "hello";
console.log(myString.toUpperCase()); // returns HELLO
console.log(myString.length); // returns 5
```

上述代码显示，即使在 JavaScript 中，基本的字符串变量也具有属性和方法。

真正的原始值没有属性或方法。例如，直接声明的数字是原始值：

```js
5.toString(); // this doesn't work because 5 is a primitive integer
let num = 5;
num.toString(); // this works because num is a Number object
```

### 原型

如前所述，JavaScript 是一种基于原型的语言。这是面向对象编程的一种变体，其中使用原型而不是类。原型是另一个对象作为另一个对象的起点。例如，在上一节中，我们看了一个简单的字符串变量：

```js
let myString = "hello";
```

正如我们在上一节中看到的，`myString`带有一些内置函数，比如`toUpperCase()`，以及属性，比如`length`。在幕后，`myString`是从字符串原型创建的对象。这意味着字符串原型中存在的所有属性和函数也存在于`myString`中。

JavaScript 对象包含一个名为`__proto__`属性的特殊属性，该属性包含对象的父原型。为了查看这一点，让我们在 Google Chrome 开发者控制台中运行`console.dir(myString)`：

![图 5.3：JavaScript 中的原型（字符串）](img/C14587_05_03.jpg)

###### 图 5.3：JavaScript 中的原型（字符串）

运行该命令返回`String`，一个包含多个方法的对象。内置的`String`对象本身具有原型。接下来，运行`console.dir(myString.__proto__.__proto__)`：

![图 5.4：JavaScript 中的原型（对象）](img/C14587_05_04.jpg)

###### 图 5.4：JavaScript 中的原型（对象）

再次运行带有附加`__proto__`属性的命令将返回`null`。JavaScript 中的所有原型最终都指向`null`，这是唯一一个本身没有原型的原型：

![图 5.5：附加 _proto_ 返回 null](img/C14587_05_05.jpg)

###### 图 5.5：附加 _proto_ 返回 null

这种一个原型导致另一个原型，依此类推的关系被称为原型链：

![图 5.6：原型链](img/C14587_05_06.jpg)

###### 图 5.6：原型链

在 JavaScript 中，当你使用变量的属性时，它从当前对象开始查找，如果找不到，就会在父原型中查找。因此，当我们运行`myString.toUpperCase()`时，它首先在`myString`中查找。在那里找不到该名称的方法后，它会检查`String`，在那里找到该方法。如果`String`中没有包含该方法，它将检查`Object`原型，然后达到`null`，此时会返回`not found error`。

JavaScript 提供了重新定义任何原型函数行为的语法，无论是内置的还是用户定义的。可以使用以下命令来实现：

```js
Number.prototype.functionName = function () {
  console.log("do something here");
}
```

在下一个练习中，我们将修改内置的`Number`原型，以赋予它一些额外的功能。请记住，这种技术可以应用于内置和自定义的原型。

### 练习 23：扩展 Number 原型

在这个练习中，我们将看一个例子，扩展 JavaScript 的内置原型`Number`，以包含一些额外的函数。在*步骤 1*之后，看看你是否能自己想出第二个解决方案：

+   double（返回值乘以二）

+   square（返回数字乘以自身）

+   Fibonacci（返回斐波那契序列中的`n`，其中每个数字是前两个数字的和）

+   阶乘（返回 1 和`n`之间所有数字的乘积的结果）

以下是要遵循的步骤：

1.  在一个新的文件夹中，创建一个名为`number.js`的文件。我们将首先向`Number`原型添加一个`double`函数。注意使用`this.valueOf()`来检索数字的值：

```js
Number.prototype.double = function () {
  return this.valueOf()*2;
}
```

1.  接下来，按照相同的模式，我们将为任意数字的平方添加一个解决方案：

```js
Number.prototype.square = function () {
  return this.valueOf()*this.valueOf();
}
```

1.  同样，我们将遵循相同的模式，尽管这个问题的解决方案有点棘手，因为它使用了记忆递归，并且使用了`BigInt`原型：

```js
Number.prototype.fibonacci = function () {
  function iterator(a, b, n) {
   return n == 0n ? b : iterator((a+b), a, (n-1n))
  }
  function fibonacci(n) {
   n = BigInt(n);
   return iterator(1n, 0n, n);
  }
  return fibonacci(this.valueOf());
}
```

#### 注意 BigInt（大整数）

在前面的步骤中，你会注意到我们使用了`BigInt`关键字。`BigInt`和`Number`一样，是 JavaScript 内置的另一个原型。它是 ES6 中的第一个新的原始类型。主要区别在于`BigInt`可以安全处理非常大的数字。`Number`原型在大于`9007199254740991`的值时开始失败。

一个数字可以通过用`BigInt()`包装它或附加`n`来转换为`BigInt`；注意使用`0n`和`1n`。

1.  接下来，我们将使用相同的模式和`BigInt`添加阶乘的解决方案：

```js
Number.prototype.factorial = function () {
  factorial = (n) => {
   n = BigInt(n);
   return (n>1) ? n * factorial(n-1n) : n;
  }
  return factorial(this.valueOf());
}
```

1.  为了演示，定义一个数字并调用函数：

```js
let n = 100;
console.log(
  "for number " + n +"\n",
  "double is " + n.double() + "\n",
  "square is " + n.square() + "\n",
  "fibonacci is " + n.fibonacci() + "\n",
  "factorial is " + n.factorial() + "\n"
);
```

1.  使用 Node.js 运行脚本：

```js
node number.js
```

你应该得到类似以下的结果：

![图 5.7：扩展 JavaScript 内置原型后的输出](img/C14587_05_07.jpg)

###### 图 5.7：扩展 JavaScript 内置原型后的输出

### ES6 类

如前所述，基于原型的语言和经典面向对象语言之间的关键区别之一是使用原型而不是类。然而，ES6 引入了内置类。我们将通过创建`Vehicle`原型/类和`Car`原型/类，比较并使用原型语法和 ES6 类语法创建对象。

首先是原型的方式：

```js
function Vehicle(name, color, sound) {
   this.name = name;
   this.color = color;
   this.sound = sound;
   this.makeSound = function() {console.log(this.sound);};
}
var car = new Vehicle("car", "red", "beep");
car.makeSound();
```

然后，使用 ES6 类做同样的事情：

```js
class Vehicle {
   constructor(name, color, sound) {
      this.name = name;
      this.color = color;
      this.sound = sound;
      this.makeSound = () => console.log(this.sound);
   }
}
const car = new Vehicle("car", "red", "beep");
car.makeSound();
```

ES6 类语法允许我们以面向对象的方式编写代码。在语言的较低级别上，类只是用于创建原型的语法样式。

在接下来的部分，我们将讨论使用 ES6 类以面向对象的方式进行编程。

## 面向对象编程（OOP）

重要的是要清楚地区分 JavaScript 对象和面向对象编程（OOP）。这是两个非常不同的东西。JavaScript 对象只是一个包含属性和方法的键值对。另一方面，面向对象编程是一组原则，可以用来编写更有组织和高效的代码。

模块化 JavaScript 并不需要面向对象编程，但它包含许多与模块化 JavaScript 相关的概念。类的使用是面向对象编程的一个基本方面，它允许我们通过创建类和子类来重用代码。

它教导我们以使维护和调试更容易的方式对程序的相关方面进行分组。它侧重于类和子类，使得代码重用更加实际。

从历史上看，面向对象编程成为处理过程代码中常见的混乱、难以阅读的代码（意思不明确的代码）的一种流行方式。通常，无组织的过程代码由于函数之间的相互依赖而变得脆弱和僵化。程序的某一方面的变化可能会导致完全不相关的错误出现。

想象一下我们正在修理一辆汽车，更换前灯导致发动机出现问题。我们会认为这是汽车设计者的糟糕架构。模块化编程拥抱程序的共同方面的分组。

面向对象编程有四个核心概念：

+   抽象

+   封装

+   继承

+   多态

在本章中，我们将看看这四个原则以及如何使用 ES6 语法在 JavaScript 编程语言中使用它们。在本章中，我们将尝试专注于实际应用，但与上述核心概念相关。

### 抽象

抽象是编程中使用的高级概念，也是面向对象编程的基础。它允许我们通过不必处理具体实现来创建复杂系统。当我们使用 JavaScript 时，许多东西默认被抽象化。例如，考虑以下数组和内置的`includes()`函数的使用：

```js
let list = ["car", "boat", "plane"];
let answer = list.includes("car") ? "yes" : "no";
console.log(answer);
```

我们不需要知道在运行`includes()`时使用的算法或代码。我们只需要知道如果数组中包含`car`，它将返回`true`，如果不包含则返回`false`。这是一个抽象的例子。随着 JavaScript 版本的更改，`include()`的内部工作方式可能会发生变化。它可能变得更快或更智能，但因为它已经被抽象化，我们不需要担心程序会出错。我们只需要知道它将返回`true`或`false`的条件。

我们不需要考虑计算机如何将二进制转换为屏幕上的图像，或者按下键盘如何在浏览器中创建事件。甚至构成 JavaScript 语言的关键字本身也是代码。

我们可以查看在使用内置 JavaScript 函数时执行的低级代码，这些代码在浏览器引擎之间会有所不同。使用`JSON.stringify()`。

让我们花一点时间思考抽象对象是什么。想象一下你桌子上的一个苹果，这是一个具体的苹果。它是苹果的一个实例或分类的概念。我们也可以谈论苹果的概念以及什么使苹果成为苹果；哪些属性在苹果中是共同的，哪些是必需的。

当我说“苹果”这个词时，你脑海中会浮现出水果的图片。你想象中的苹果的确切细节取决于你对苹果概念的理解。当我们在计算机程序中定义一个苹果类时，我们正在定义程序如何定义苹果类。就像我们的想象力一样，一个事物的概念可以是具体的或不具体的。它可能只包含一些因素，比如形状和颜色，也可能包含几十个因素，包括重量、产地和口味。

### 类和构造函数

在第一个练习中，我们创建了一个灯模块。虽然它是一个模块，但它不是面向对象的。在本节中，我们将以面向对象的方式重新设计该模块。

类最重要的一个方面是它的构造函数。构造函数是在创建类的实例时调用的内置函数。通常，构造函数用于定义对象的属性。例如，您经常会看到类似于这样的东西：

```js
class Apple {
  constructor(color, weight) {
   this.color = color;
   this.weight = weight;
  }
}
```

传递的参数将保存到实例中以供以后使用。您还可以根据传递的参数添加一些额外的属性。例如，假设我们想通过附加日期时间戳来给我们的苹果一个出生日期。我们可以在我们的构造函数内添加第三行：

```js
  this.birthdate = Date.now();
```

或者我们可能想在灯模块中调用一些其他函数。想象一下，每个进入世界的苹果都有 1/10 的机会是腐烂的：

```js
  this.checkIfRotten();
```

我们的类需要包含一个`checkIfRotten`函数，该函数将`isRotten`属性设置为 10 次中的 1 次为`true`：

```js
checkIfRotten() {
  If (Math.floor(Math.random() * Math.floor(10)) == 0) {
   this.isRotten = true;
  } else {
   this.isRotten = false;
  }
}
```

### 练习 24：将灯模块转换为类

#### 注意

本练习使用本章*练习 22，编写一个简单的 ES6 模块*的最终产品作为起点。完成此练习后的代码状态可以在[`github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson05/Exercise24`](https://github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson05/Exercise24)找到。

让我们回到本章*练习 22，编写一个简单的 ES6 模块*中的灯示例。我们将使用在上一章中为灯模块定义的属性，并在创建时进行分配。此外，我们将编写函数来检查灯属性的格式。如果使用无效的属性值创建了灯，我们将将其设置为默认值。

执行练习的步骤如下：

1.  打开`js/light.js`并删除上一个练习中的代码。

1.  为我们的`Light`类创建一个类声明：

```js
class Light  {
}
```

1.  向类添加`constructor`函数，并从参数中设置属性以及`datetime`属性。我们将首先将参数传递给两个函数以检查正确的格式，而不是直接设置`state`和`brightness`。这些函数的逻辑将在以下步骤中编写：

```js
class Light  {
  constructor(state, brightness) {
   // Check that inputs are the right types
   this.state = this.checkStateFormat(state);
   this.brightness = this.checkBrightnessFormat(brightness);
   this.createdAt = Date.now();
  }
}
```

1.  将`checkStateFormat`和`checkBrightnessFormat`函数添加到类声明中：

```js
  checkStateFormat(state) {
   // state must be true or false
   if(state) {
    return true;
   } else {
    return false;
   }
  }
  checkBrightnessFormat(brightness) {
   // brightness must be a number between 0.01 and 1
   if(isNaN(brightness)) {
    brightness = 1;
   } else if(brightness > 1) {
    brightness = 1;
   } else if(brightness < 0.01) {
    brightness = 0.01;
   }
   return brightness;
  }
```

1.  添加一个`toggle`函数和一个`test`函数，我们将用于调试。这两个函数也应该在类声明内。`toggle`函数将简单地将灯的状态转换为当前状态的相反状态；例如，从开到关，反之亦然：

```js
  toggle() {
   this.state = !this.state;
  }
  test() {
   alert("state is " + this.state);
  }
```

1.  在`js/lightBulb.js`中，在类声明下面，添加一个模块导出，就像我们在上一个练习中所做的那样：

```js
export default Light;
```

1.  打开`js/viewer.js`，并用包含`Light`类实例的变量替换我们在*练习 22，编写一个简单的 ES6 模块*中编写的`light()`行：

```js
let light = new Light(true, 0.5);
```

1.  在`js/viewer.js`中的前一行下面，添加以下代码。此代码将图像的源连接到`state`，并将图像的不透明度连接到`brightness`：

```js
// Set image based on light state
bulb.src = light.state ? onImage : offImage;
// Set opacity based on brightness
bulb.style.opacity = light.brightness;
// Set slider value to brightness
slider.value = light.brightness;
bulb.onclick = function () {
  light.toggle();
  bulb.src = light.state ? onImage : offImage;
}
slider.onchange = function () {
  light.brightness = this.value;
  bulb.style.opacity = light.brightness;
}
```

1.  返回项目目录并运行`npm start`。项目运行后，在浏览器中打开`localhost:8000`。您应该看到灯的新图片，指示它是开启的：![图 5.8：状态为 true 的灯](img/C14587_05_08.jpg)

###### 图 5.8：状态为 true 的灯

打开页面后，单击图像并确保这样做会导致图像更改。还要注意页面底部的输入滑块。尝试更改值以确认这样做是否会更新图像的不透明度。

#### 类的命名约定

在上面的代码中，我们创建了一个`Light`类。请注意，我们使用的是大写的“L”，而不是 JavaScript 中通常使用的驼峰命名法。将类的名称大写是一种常见的做法；有关命名约定的更多详细信息，请参阅 Google 的 JavaScript 样式指南：[`google.github.io/styleguide/javascriptguide.xml#Naming`](https://google.github.io/styleguide/javascriptguide.xml#Naming)。

Camelcase 是 JavaScript 中最流行的命名风格。其他风格包括 snake_case、kebab-case 和 PascalCase。

### 默认属性

使用类时，您最常用的功能之一是默认属性值。通常，您希望创建类的实例，但不关心属性的具体值-例如，不指定参数：

```js
myLight = new Light();
```

`state`和`brightness`都将默认为`undefined`。

根据我们编写的代码，调用没有属性的`light`不会引发错误，因为我们编写了`checkStateFormat`和`checkBrightnessFormat`来处理所有无效值。然而，在许多情况下，您可以通过在构造函数中提供默认值来简化代码，如下所示：

```js
  constructor(state=false, brightness=100) {
```

上述语法不是特定于类`constructor`，可以用于设置任何函数的默认参数，假设您使用的是 ES6、ES2015 或更新版本的 JavaScript。默认参数在 ES2015 之前的版本中不可用。

### 封装

封装是模块只在必要时才公开对象属性的想法。此外，应该使用函数而不是直接访问和修改属性。例如，让我们回到我们的灯模块。在`constructor`函数内部，我们确保首先通过状态检查器运行值：

```js
  constructor(state, brightness) {
   // Check that input has the right format
   this.brightness = this.checkBrightnessFormat(brightness);
  }
```

假设您开发了前面的模块并发布供同事使用。您不必担心他们使用错误的值初始化类，因为如果他们这样做，`checkBrightnessFormat()`将自动更正该值。但是，一旦我们的类的实例存在，其他人就可以直接修改该值，没有任何阻止：

```js
let light = new Light();
light.brightness = "hello";
```

在一个命令中，我们绕过了`Light`类的`checkBrightnessFormat`函数，并且我们有了一个`brightness`值为`hello`的灯。

封装是以使这种情况不可能的方式编写我们的代码的想法。诸如 C#和 Java 之类的语言使封装变得容易。不幸的是，即使在 ES6 更新后，JavaScript 中使用封装也不明显。有几种方法可以做到这一点；其中最受欢迎的方法之一是利用内置的`WeakMap`对象类型，这也是 ES6 的新功能之一。

### WeakMap

**WeakMap**对象是一个键值对集合，其中键是对象。WeakMap 具有一个特殊的特性，即如果 WeakMap 中的键对象被从程序中移除并且没有对它的引用存在，WeakMap 将从其集合中删除关联的键值对。这个删除键值对的过程称为垃圾回收。因此，在使用映射可能导致内存泄漏的情况下，该元素特别有用。

WeakMap 比 Map 更适合的一个例子是，一个脚本跟踪动态变化的 HTML 页面中的每个元素。假设 DOM 中的每个元素都被迭代，我们在 Map 中创建了一些关于每个元素的额外数据。然后，随着时间的推移，元素被添加到 DOM 中并从中删除。使用 Map，所有旧的 DOM 元素将继续被引用，导致存储与已删除的 DOM 元素相关的无用信息，从而导致随着时间的推移内存使用量增加。使用 WeakMap，DOM 元素的删除（它是集合中的键对象）会导致在垃圾回收期间删除集合中的关联条目。

在这里，我们将使用`WeakMap()`。首先，我们创建一个空的`map`变量，然后创建一个带有一些属性的`light`对象。然后，我们将对象本身与一个字符串`kitchen light`关联起来。这不是向`light`添加属性的情况；相反，我们使用对象就像它是地图中的属性名称一样：

```js
var map = new WeakMap();
var light = {state: true, brightness: 100};
map.set(light, "kitchen light");
console.log(map.get(light));
```

另外，需要注意的是，键对象是基于对对象的特定引用。如果我们创建具有相同属性值的第二个灯，那将算作一个新的键：

```js
let light2 = {state: true, brightness: 100};
map.set(light2, "bedroom light");
// above has not changed kitchen light reference
console.log(map.get(light));
```

如果我们更新对象的属性，那不会改变映射：

```js
light.state = false;
// reference does not change
console.log(map.get(light));
```

映射将存在，直到键对象超出范围，或者直到它被设置为 null 并进行垃圾回收；例如：

```js
light = null;
// value will not be returned here
console.log(map.get(light));
```

### 练习 25：封装的 WeakMap

#### 注意

本练习以本章的*练习 24，将灯模块转换为类*的最终产品为起点。完成此练习后的代码状态可以在[`github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson05/Exercise25`](https://github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson05/Exercise25)找到。

在这个练习中，我们将使用`WeakMap`来创建无法直接从模块外部访问的私有变量。执行以下步骤完成练习：

1.  打开`js/light.js`，并在文件顶部添加一个名为`privateVars`的`WeakMap`对象：

```js
let privateVars = new WeakMap();
```

1.  在`js/light.js`中，修改`constructor`函数，使得对象属性通过`set`方法保存到`privateVars`中，而不是直接在对象上：

```js
constructor(state, brightness) { 
  // Parse values
  state = this.checkStateFormat(state);
  brightness = this.checkBrightnessFormat(brightness);
  // Create info object 
  let info = {
   "state": state,
   "brightness": brightness,
   "createdAt": Date.now()
  };
// Save info into privateVars 
  privateVars.set(this, info); 
}
```

1.  现在，在`js/light.js`中，修改`toggle`函数，以便我们从名为`privateVars`的`WeakMap`对象获取状态信息。请注意，当我们设置变量时，我们发送回一个包含所有信息而不仅仅是`state`的对象。在我们的示例中，每个`light`实例都与`WeakMap`关联的单个`info`对象：

```js
toggle() { 
  let info = privateVars.get(this); 
  info.state = !info.state;
  privateVars.set(this, info); 
}
```

1.  我们还需要以类似的方式修改`js/light.js`中的`test`函数。我们将改变发送给用户的`state`的来源，以便在警报中使用`WeakMap`：

```js
test() { 
  let info = privateVars.get(this); 
  alert("state is " + privateVars.get(this).state);
}
```

1.  由于封装夺走了直接更改状态和亮度的能力，我们需要添加允许这样做的方法。我们将从在`js/light.js`中添加`setState`函数开始。请注意，它几乎与我们的`toggle`函数相同：

```js
setState(state) {
  let info = privateVars.get(this);
  info.state = checkStateFormat(state); 
  privateVars.set(this, info); 
}
```

1.  接下来，在`js/light.js`中添加 getter 方法：

```js
getState() {
  let info = privateVars.get(this); 
  return info.state;
}
```

1.  按照最后两个步骤的模式，在`js/light.js`中为`brightness`属性添加 getter 和 setter 函数：

```js
setBrightness(brightness) { 
  let info = privateVars.get(this);
  info.brightness = checkBrightnessFormat(brightness);
  privateVars.set(this, info);
}
getBrightness() { 
  let info = privateVars.get(this);
  return info.brightness;
}
```

1.  我们需要做的最后一个更改是在`js/viewer.js`中。在变量声明下面，将每个对光亮度和状态的引用更改为使用我们创建的 getter 方法：

```js
// Set image based on light state
bulb.src = light.getState() ? onImage : offImage;
// Set opacity based on brightness
bulb.style.opacity = light.getBrightness();
// Set slider value to brightness
slider.value = light.getBrightness();
bulb.onclick = function () {
  light.toggle();
  bulb.src = light.getState() ? onImage : offImage;
}
slider.onchange = function () {
  light.setBrightness(this.value);
  bulb.style.opacity = light.getBrightness();
}
```

1.  使用`npm start`运行代码，并在浏览器中查看`localhost:8000`上的页面项目。检查确保单击图像有效，以及使用输入滑块更改亮度有效：

![图 5.9：使用单击和滑块功能正确呈现网站](img/C14587_05_09.jpg)

###### 图 5.9：使用单击和滑块功能正确呈现网站

### 获取器和设置器

在使用封装时，由于我们不再允许用户直接访问属性，大多数对象最终将具有一些或全部属性的 getter 和 setter 函数：

```js
console.log(light.brightness);
// will return undefined
```

相反，我们专门创建允许获取和设置属性的函数。这些被称为 getter 和 setter，它们是一种流行的设计模式，特别是在诸如 Java 和 C++等语言中。如果您在上一个练习中完成了第 7 步，应该已经为`brightness`添加了 setter 和 getter：

```js
setBrightness(brightness) {
  let info = privateVars.get(this);
  info.brightness = checkBrightnessFormat(state);
  privateVars.set(this, info);
}
getBrightness() {
  let info = privateVars.get(this);
  return info.brightness;
}
```

### 继承

继承是一个类继承另一个类的属性和方法的概念。从另一个类继承的类称为子类，被继承的类称为超类。

正是从术语**超类**中，我们得到了内置的`super()`函数，它可以用于调用子类的超类的构造函数。我们将在本章后面使用`super()`来创建自己的子类。

应该注意的是，一个类既可以是子类，也可以是超类。例如，假设我们有一个模拟不同类型动物的程序。在我们的程序中，我们有一个哺乳动物类，它是动物类的子类，也是狗类的超类。

通过这种方式组织我们的程序，我们可以将所有动物相关的属性和方法放在动物类中。哺乳动物子类包含哺乳动物相关的方法，但不包括爬行动物；例如：

![图 5.10：JavaScript 中的继承](img/C14587_05_10.jpg)

###### 图 5.10：JavaScript 中的继承

这一开始可能听起来很复杂，但通常可以节省大量的编码工作。如果不使用类，我们将不得不将方法从一个动物复制并粘贴到另一个动物中。这就带来了在多个地方更新函数的困难。 

回到我们的智能家居场景，假设我们收到了一个新的彩色灯泡设备。我们希望我们的彩色灯泡具有灯泡中包含的所有属性和函数。此外，彩色灯应该有一个额外的`color`属性，包含一个十六进制颜色代码，一个颜色格式检查器和与改变颜色相关的函数。

我们的代码也应该以一种方式编写，如果我们对底层的`Light`类进行更改，彩色灯泡将自动获得任何添加的功能。

### 练习 26：扩展一个类

#### 注意

本练习使用*练习 25，封装的 WeakMap*的最终产品作为起点。完成此练习后的代码状态可以在[`github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson05/Exercise26`](https://github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson05/Exercise26)找到。

为了扩展上一个练习中编写的`Light`类，我们将创建一个新的`ColorLight`类：

1.  在`/js/colorLight.js`中创建一个新文件。在第一行，我们将导入`./light.js`，这将作为起点：

```js
import Light from './light.js';
```

1.  接下来，我们将为私有变量创建`WeakMap`。然后，我们将为我们的`ColorLight`类创建一个类语句，并使用`extends`关键字告诉 JavaScript 它将使用`Light`作为起点：

```js
let privateVars = new WeakMap();
class ColorLight extends Light {
}
```

1.  在`ColorLight`类语句内部，我们将创建一个新的`constructor`，它使用内置的`super()`函数，运行我们基类`Light`的`constructor()`函数：

```js
class ColorLight extends Light {
  constructor(state=false, brightness=100, color="ffffff") {
   super(state, brightness);
   // Create info object
   let info = {"color": this.checkColorFormat(color)};
   // Save info into privateVars
   privateVars.set(this, info);
  }
}
```

1.  请注意在上述构造函数中，我们调用了`checkColorFormat()`，这是一个检查提供的颜色值是否是有效十六进制值的函数。如果不是，我们将把值设置为白色的十六进制值(#FFFFFF)。该函数应该在`ColorLight`类语句内部：

```js
  checkColorFormat(color) {
   // color must be a valid hex color
   var isHexColor  = /^#[0-9A-F]{6}$/i.test('#'+color);
   if(!isHexColor) {
    // if invalid make white
    color = "ffffff";
   }
   return color;
  }
```

1.  添加 getter 和 setter 函数，就像我们在后面的练习中所做的那样：

```js
  getColor() {
   let info = privateVars.get(this);
   return info.color;
  }
  setColor(color) {
   let info = privateVars.get(this);
   info.color = this.checkColorFormat(color);
   privateVars.set(this, info);
  }
```

1.  在`js/colorLight.js`的底部，添加一个`export`语句以使模块可供导入：

```js
export default ColorLight;
```

1.  在文件顶部打开`js/viewer.js`，并将`Light`导入切换为`ColorLight`。在下面，我们将导入一个预先编写的名为`changeColor.js`的脚本：

```js
import ColorLight from './colorLight.js';
import changeColor from './__extra__/changeColor.js';
```

1.  在`js/viewer.js`中更下面，找到初始化`light`变量的行，并将其替换为以下内容：

```js
let light = new ColorLight(true, 1, "61AD85");
```

1.  在`js/viewer.js`的底部，添加以下内容：

```js
// Update image color
changeColor(light.getColor());
```

1.  再次使用`npm start`启动程序，并在浏览器中转到`localhost:8000`：

如果您按照说明正确操作，现在应该看到灯泡呈浅绿色，如下图所示。尝试打开`js/viewer.js`并更改十六进制值；这样做应该会导致灯泡图像显示不同的颜色：

![图 5.11：change-color 函数应用 CSS 滤镜使灯泡变绿](img/C14587_05_11.jpg)

###### 图 5.11：change-color 函数应用 CSS 滤镜使灯泡变绿

### 多态

多态性就是简单地覆盖父类的默认行为。在 Java 和 C#等强类型语言中，多态性可能需要花费一些精力。而在 JavaScript 中，多态性是直接的。你只需要重写一个函数。

例如，在上一个练习中，我们将`Light`和`ColorLight`类扩展了。假设我们想要获取在`Light`中编写的`test()`函数，并覆盖它，以便不是弹出灯的状态，而是弹出灯的当前颜色值。

因此，我们的`js/light.js`文件将包含以下内容：

```js
  test() {
   let info = privateVars.get(this); 
   alert("state is " + privateVars.get(this).state);
  }
Then all we have to do is create a new function in js/colorLight.js which has the same name, and replace state with color:
  test() { 
   let info = privateVars.get(this); 
   alert("color is " + privateVars.get(this).color);
  }
```

### 练习 27：LightBulb Builder

#### 注意

这个练习使用*Exercise 26, Extending a Class*的最终产品作为起点。完成这个练习后的代码状态可以在[`github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson05/Exercise27`](https://github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson05/Exercise27)找到。

在这个练习中，我们将运用到目前为止学到的概念来增强我们的示例项目。我们将修改项目，使我们能够创建无限个`lightbulb`类的实例，选择颜色、亮度和状态：

1.  打开`js/light.js`，并在`WeakMap`引用的下面添加两个图像源的值：

```js
let onImage = "images/bulb_on.png";
let offImage = "images/bulb_off.png";
```

1.  接下来，在`js/light.js`中，在`info`变量定义的下面，添加以下内容：

```js
   // Create html element
   let div = document.createElement("div");
   let img = document.createElement("img");
   let slider = document.createElement("input");
   // Save reference to element as private variable
   info.div = div;
   info.img = img;
   info.slider = slider;
   this.createDiv(div, img, slider, state, brightness);
```

1.  在`js/light.js`的最后一步中，我们引用了`this.createDiv`。在这一步中，我们将在`js/light.js`的构造函数下面创建该函数。该函数为`Light`类的每个实例创建 HTML：

```js
  createDiv(div, img, slider, state, brightness) {
   // make it so we can access this in a lower scope
   let that = this;
   // modify html
   div.style.width = "200px";
   div.style.float = "left";
   img.onclick = function () { that.toggle() };
   img.width = "200";
   img.src = state ? onImage : offImage;
   img.style.opacity = brightness;
   slider.onchange = function () { that.setBrightness(this.value) };
   slider.type = "range";
   slider.min = 0.01;
   slider.max = 1;
   slider.step = 0.01;
   slider.value = brightness;
   div.appendChild(img);
   div.appendChild(slider);
   // append to document
   document.body.appendChild(div);
  }
```

1.  接下来，在`js/light.js`中，找到`setState`函数，并在函数内添加以下行：

```js
info.img.src = info.state ? onImage : offImage;
```

1.  在`js/light.js`的`toggle`函数中添加相同的行：

```js
info.img.src = info.state ? onImage : offImage;
```

1.  同样地，我们将更新`js/light.js`中的`setBrightness`函数，以根据亮度设置图像的不透明度：

```js
info.img.style.opacity = brightness;
```

1.  `js/light.js`中的最后一个更改是为`img` HTML 对象添加一个 getter 函数。我们将它放在`getBrightness`和`toggle`函数之间：

```js
  getImg() {
   let info = privateVars.get(this);
   return info.img;
  }
```

1.  在`js/colorLight.js`中，我们将导入预先构建的`colorChange`函数。这应该放在你的导入下面的位置，就在`Light`导入的下面：

```js
import changeLight from './__extra__/changeColor.js';
```

1.  接下来，在`js/colorLight.js`中，我们将通过添加以下行来更新构造函数：

```js
   let img = this.getImg();
   img.style.webkitFilter = changeLight(color);
```

1.  在`js/viewer.js`中，删除所有代码并替换为以下内容：

```js
import ColorLight from './colorLight.js';
let slider = document.getElementById("brightnessSlider");
let color = document.getElementById("color");
let button = document.getElementById("build");
button.onclick = function () {
  new ColorLight(true, slider.value, color.value);
}
```

1.  最后的更改是`index.html`；删除`img`和`input`标签，并替换为以下内容：

```js
  <div style="position: 'fixed', top: 0, left: 0">
   <input type="color" id="color" name="head" value="#e66465">
   <input id="brightnessSlider" min="0.01" max="1" step="0.01" type="range"/>
   <button id="build">build</button>
  </div>
```

1.  完成所有更改后，运行`npm start`并在浏览器中打开`localhost:8000`。如果一切都做对了，点击`build`按钮应该根据所选的颜色向页面添加一个新元素：

![图 5.12：创建多个 lightclub 类的实例](img/C14587_05_12.jpg)

###### 图 5.12：创建多个 lightclub 类的实例

如你所见，一旦你创建了许多相同的实例，类就真的开始变得非常有用了。在下一节中，我们将看看 npm 包以及如何将我们的`Light`类导出为一个。

## npm 包

**npm 包**是一个已经打包并上传到 npm 服务器的 JavaScript 模块。一旦模块被上传到 npm，任何人都可以快速安装和使用它。

这对你可能不是新鲜事，因为任何使用 Node.js 的人很快就会安装一个包。不太常见的是如何创建和上传一个包。作为开发人员，很容易花费数年的时间而不需要发布一个公共模块，但了解这一点是值得的。这不仅有助于当你想要导出自己的模块时，还有助于阅读和理解你的项目使用的包。

创建 npm 模块的第一步是确保您有一个完整的`package.json`文件。在本地运行项目时，通常不必过多担心诸如**author**和**description**之类的字段。但是，当您准备将模块用于公共使用时情况就不同了。您应该花时间填写与您的软件包相关的所有字段。

以下是包括 npm 推荐的常见属性的表格。其中许多是可选的。有关更多信息和完整列表，请参阅[`docs.npmjs.com/files/package.json`](https://docs.npmjs.com/files/package.json)。

至少，元数据应包括名称、版本和描述。此外，大多数软件包将需要一个`dependencies`属性；但是，这应该通过在使用`npm install`安装依赖项时自动生成使用`--save`或`-s`选项：

![图 5.13：npm 属性表](img/C14587_05_13.jpg)

###### 图 5.13：npm 属性表

以下表格显示了 npm 的一些更多属性：

![图 5.14：npm 属性表续](img/C14587_05_14.jpg)

###### 图 5.14：npm 属性表续

### npm 链接命令

完成`package.json`并且您想要测试的软件包的第一个版本后，您可以使用`npm link`命令。链接命令将将您的本地 npm 项目与命名空间关联起来。例如，首先导航到要使用本地`npm`软件包的项目文件夹：

```js
cd ~/projects/helloWorld
npm link
```

然后，进入另一个项目文件夹，您想要使用该软件包，并运行`npm link helloWorld`，其中`helloWorld`是您正在测试的软件包的名称：

```js
cd ~/projects/otherProject
npm link helloWorld
```

这两个步骤将使您能够像使用`npm install helloWorld`安装`helloWorld`一样工作。通过这样做，您可以确保在另一个项目中使用时，您的软件包在本地工作。

### Npm 发布命令

一旦您对在本地测试软件包的结果感到满意，您可以使用`npm publish`命令轻松将其上传到 npm。要使用`publish`命令，您首先需要在[`www.npmjs.com/`](https://www.npmjs.com/)上创建一个帐户。一旦您拥有帐户，您可以通过在命令行上运行`npm login`来本地登录。

登录后，发布软件包非常简单。只需导航到您的`project`文件夹并运行`npm publish`。以下是成功上传到 npm 供他人使用的软件包的示例：

![图 5.15：已发布的 npm 软件包示例](img/C14587_05_15.jpg)

###### 图 5.15：已发布的 npm 软件包示例

### ESM 与 CommonJS

ESM 是 ECMAScript 模块的缩写，这是 ES6 中模块的标准。因此，您可能会听到将“ES6 模块”称为 ESM。这是因为 ESM 标准在 ES6 成为标准之前就已经在开发中。

您可能已经看到了在上一章中使用的 CommonJS 格式：

```js
const express = require('express');
```

ES6 模块样式中的相同代码将是这样的：

```js
import express from 'express';
```

ES6 模块非常棒，因为它们使 JavaScript 开发人员对其导入有更多控制。但是，重要的是要注意，目前 JavaScript 正处于过渡期。ES6 已经明确规定了 ES6 模块应该如何工作的标准。尽管大多数浏览器已经实现了它，但 npm 仍在使用自己的标准 CommonJS。

也就是说，ES6 的引入正在迅速得到接受。npm 现在附带一个实验性标志，`--experimental-modules`，允许使用 ES6 样式模块。但是，不建议使用此标志，因为它增加了不必要的复杂性，例如必须将文件扩展名从`.js`更改为`.mjs`。

### Babel

使用 ES6 模块与 Node.js 的更常见和推荐的方法是运行 JavaScript 编译器。最流行的编译器是`Babel.js`，它将 ES6 代码编译为可以在任何地方运行的较旧版本的 JavaScript。

Babel 是 Node.js 生态系统中广泛使用的工具。通常，项目使用具有 Babel 和其他捆绑工具（如 webpack）的起始模板。这些起始项目允许开发人员开始使用 ES6 导入，而无需考虑是否需要编译步骤。例如，有 Facebook 的 create-react-app，它会在文件更改时编译和显示您的应用程序。

React 是推动 ES6 的最大社区之一。在 React 生态系统中，标准导入使用的是 ES6。以下内容摘自 React 关于创建组件的文档：

```js
import React, { Component } from 'react';
class Button extends Component {
  render() {
   // ...
  }
}
export default Button; // Don't forget to use export default!
```

注意前面的代码与我们一直在进行的工作之间的相似之处。这是继承的一个例子，其中`Button`继承了`Component`的属性，就像`ColorLight`继承了`Light`的属性一样。React 是一个基于组件的框架，大量使用 ES6 功能，如导入和类。

### webpack

另一个常见的 JavaScript 编译器是 webpack。webpack 接受多个 JavaScript 文件并将它们编译成单个捆绑文件。此外，webpack 可以采取步骤来提高性能，例如缩小代码以减少总大小。在使用模块时，webpack 特别有用，因为每个加载到 HTML 站点中的单独文件都会增加加载时间，因为会产生额外的 HTTP 调用。

使用 webpack，我们可以非常简单地指定要编译的 JavaScript 的入口点，并且它将自动合并任何引用的文件。例如，如果我们想要编译上一个练习中的代码，我们将创建一个`webpack.config.js`文件来指定入口点：

```js
const path = require("path");
module.exports = {
  mode: 'development',
  entry: "./src/js/viewer.js",
  output: {
   path: path.resolve(__dirname, "build"),
   filename: "bundle.js"
  }
};
```

注意上面定义的`entry`；这将是我们程序的起点，webpack 将自动找到所有引用的文件。另一个重要的值要注意的是`output`。这定义了编译器创建的结果捆绑 JavaScript 文件的位置和文件名。

在下一个练习中，我们将使用 Babel 将我们的代码从 ES6 转换为通用 JavaScript。一旦我们转换了我们的 JavaScript，我们将使用 webpack 将生成的文件编译成一个捆绑的 JavaScript 文件。

### 练习 28：使用 webpack 和 Babel 转换 ES6 和包

#### 注意

此练习使用*练习 27，LightBulb Builder*的最终产品作为起点。完成此练习后的代码状态可以在[`github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson05/Exercise28`](https://github.com/TrainingByPackt/Professional-JavaScript/tree/master/Lesson05/Exercise28)找到。

在这个练习中，我们将使用 Babel 将我们的 ES6 转换为与旧浏览器（如 Internet Explorer）兼容的通用 JavaScript。我们要做的第二件事是运行 webpack 将所有 JavaScript 文件编译成单个文件：

1.  在项目的基础上创建两个新文件夹，一个名为`build`，另一个名为`src`：

```js
mkdir src build
```

1.  将`images`，`index.html`和`js`文件夹移动到新的`src`文件夹中。源文件夹将用于稍后生成`build`文件夹的内容：

```js
mv images index.html js src
```

1.  安装`babel-cli`和`babel preset`作为开发人员依赖项：

```js
npm install --save-dev webpack webpack-cli @babel/core @babel/cli @babel/preset-env
```

1.  在根目录下添加一个名为`.babelrc`的文件。在其中，我们将告诉 Babel 使用预设设置：

```js
{
  "presets": ["@babel/preset-env"]
}
```

1.  在根目录中添加一个名为`webpack.config.js`的 webpack 配置文件：

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

1.  要从`src`生成`build`文件夹的内容，我们需要向项目添加一个新的脚本命令。打开`package.json`，查找列出脚本的部分。在该部分，我们将添加一个`build`命令，该命令运行 Babel 和 webpack，并将我们的`image`文件复制到`build`文件夹中。我们还将修改`start`命令以引用我们的`build`文件夹，以便在构建后进行测试：

```js
  "scripts": {
   "start": "ws --directory build",
   "build": "babel src -d build && cp -r src/index.html src/images build && webpack --config webpack.config.js"
  },
```

#### 注意

Windows 用户应使用以下命令：

`"build": "babel src -d build && copy src build && webpack --config webpack.config.js"`

1.  为了确保命令已经正确添加，运行命令行上的`npm run build`。你应该会看到这样的输出：![图 5.16：npm run build 输出](img/C14587_05_16.jpg)

###### 图 5.16：npm run build 输出

1.  接下来，打开`build/index.html`并将`script`标签更改为导入我们新创建的文件`bundle.js`：

```js
<script src="bundle.js"></script>
```

1.  要测试，运行`npm start`并在浏览器中打开`localhost:8000`。你应该会看到与上次练习相同的网站。按几次`build`按钮以确保它按预期工作：![图 5.17：使用构建按钮进行测试运行](img/C14587_05_17.jpg)

###### 图 5.17：使用构建按钮进行测试运行

1.  为了双重检查一切是否编译正确，去浏览器中输入`localhost:8000/bundle.js`。你应该会看到一个包含所有我们的 JavaScript 源文件编译版本的大文件：

![图 5.18：所有我们的 JavaScript 源文件的编译版本](img/C14587_05_18.jpg)

###### 图 5.18：所有我们的 JavaScript 源文件的编译版本

如果你做的一切都正确，你应该有一个包含所有我们的 JavaScript 代码编译成单个文件的`bundle.js`文件。

### 可组合性和组合模块的策略

我们已经看到模块如何成为另一个模块的扩展，就像`ColorLight`是`Light`的扩展一样。当项目增长时，另一个常见的策略是有模块本身由多个子模块组成。

使用子模块就像在模块文件本身导入模块一样简单。例如，假设我们想要改进我们灯模块中的亮度滑块。也许如果我们创建一个新的`Slider`模块，我们可以在除了`Light`类之外的多种情况下使用它。这是一种情况，我们建议将我们的“高级滑块输入”作为子模块。

另一方面，如果你认为你的新滑块只会在`Light`类中使用，那么将它添加为一个新类只会增加更多的开销。不要陷入过度模块化的陷阱。关键因素在于可重用性和实用性。

### 活动 6：创建带有闪光模式的灯泡

你工作的灯泡公司要求你为他们的产品工作。他们想要一个带有特殊“闪光模式”的灯泡，可以在活动和音乐会上使用。闪光模式的灯允许人们将灯置于闪光模式，并在给定的时间间隔内自动打开和关闭。

创建一个`FlashingLight`类，它扩展了`Light`。该类应该与`Light`相同，只是有一个名为`flashMode`的属性。如果`flashMode`打开，则状态的值应该每五秒切换一次。

创建了这个新组件后，将其添加到`js/index.js`中的包导出，并使用 Babel 编译项目。

执行以下步骤完成活动：

1.  安装`babel-cli`和`babel`预设为开发人员依赖项。

1.  添加`.babelrc`告诉 Babel 使用`preset-env`。

1.  添加一个 webpack 配置文件，指定模式、入口和输出位置。

1.  创建一个名为`js/flashingLight.js`的新文件；它应该作为一个空的 ES6 组件开始，扩展`Light`。

1.  在文件顶部，添加一个`weakMap`类型的`privateVars`变量。

1.  在构造函数中，设置`flashMode`属性并将其保存到构造函数中的`privateVars`中。

1.  为`FlashingLight`对象添加一个 setter 方法。

1.  为`FlashingLight`对象添加一个 getter 方法。

1.  在第 2 行，添加一个空变量，用于在类的全局级别跟踪闪烁计时器。

1.  创建一个引用父类的`lightSwitch()`函数的`startFlashing`函数。这一步很棘手，因为我们必须将它绑定到`setInterval`。

1.  创建一个`stopFlashing`函数，用于关闭计时器。

1.  在构造函数中，检查`flashMode`是否为 true，如果是，则运行`startFlashing`。

1.  在设置`mode`时，还要检查`flashMode` - 如果为 true，则`startFlashing`；否则，`stopFlashing`。

1.  在`index.js`中导入和导出新组件。

1.  通过使用 npm 运行我们的`build`函数来编译代码。

**预期输出**：

![图 5.19：带闪光模式的灯泡](img/C14587_05_19.jpg)

###### 图 5.19：带闪光模式的灯泡

#### 注意

这个活动的解决方案可以在第 599 页找到。

## 总结

在本章中，我们探讨了模块化设计的概念，ES6 模块以及它们在 node 中的使用。面向对象设计原则在设计由多个模块层组成的复杂系统的程序时非常有用。

ES6 类允许我们比以前的 JavaScript 版本更轻松地创建类。这些类可以使用`extends`关键字构建。这允许在更复杂的对象之上构建更复杂的对象等等。

我们还看到了新的 ES6 `WeakMap`类型如何允许我们创建私有变量。这种模式限制了将被其他人使用的模块中的错误数量。例如，通过要求更改属性，我们可以在允许更改之前检查格式和值。这就是灯泡示例的情况，我们希望检查`state`在允许设置之前是否为布尔值。我们通过为我们想要向我们代码的其他部分公开的每个私有变量创建 getter 和 setter 方法来实现这一点。

之后，我们谈到了 ES6 模块目前在 Node.js 中没有得到原生支持，尽管像 Facebook 支持的 React 这样的知名项目广泛使用它们。作为解决这一限制的方法，我们安装了 Babel，一个 ES6 到 JavaScript 的编译器，并用它将我们的`src`文件夹转换为最终的构建代码。

我们还谈到了一旦在本地使项目工作，就可以将其转换为可以通过 npm 共享和更新的 npm 包。这个过程涉及使用`npm link`在本地进行测试。然后，一旦满意包的工作方式，使用`npm publish`进行发布。

在下一章中，我们将讨论代码质量以及如何实施自动化测试来防御回归，因为我们更新我们的代码。
