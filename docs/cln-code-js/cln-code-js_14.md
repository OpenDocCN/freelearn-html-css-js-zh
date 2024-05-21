# 第十一章：设计模式

我们遇到的大多数问题都不是新问题。许多在我们之前的程序员已经解决了类似的问题，并通过他们的努力，各种编程模式已经出现。我们称这些为设计模式。

设计模式是我们的代码所在的有用结构、样式和模板。设计模式可以规定从代码基础的整体脚手架到用于构建表达式、函数和模块的单个语法片段的任何内容。通过构建软件，我们不断地，而且通常是不知不觉地，处于*设计*的过程中。正是通过这个设计过程，我们正在定义用户和维护者在接触我们代码时将经历的体验。

为了使我们适应设计师而不是程序员的视角，让我们考虑一个简单软件抽象的设计。

在本章中，我们将涵盖以下主题：

+   设计师的视角

+   架构设计模式

+   JavaScript 模块

+   模块化设计模式

+   规划和和谐

# 设计师的视角

为了赋予我们设计师的视角，让我们探索一个简单的问题。我们必须构建一个抽象，允许用户给我们两个字符串，一个主题字符串和一个查询字符串。然后我们必须计算主题字符串中查询字符串的计数。

因此，请考虑以下查询字符串：

```js
"the"
```

并看一下以下主题字符串：

```js
"the fox jumped over the lazy brown dog"
```

我们应该收到一个结果为`2`。

对于我们作为设计师的目的，我们关心那些必须使用我们代码的人的体验。现在，我们不会担心我们的实现；我们只会考虑接口，因为主要是我们代码的接口将驱动我们的同行程序员的体验。

作为设计师，我们可能会做的第一件事是定义一个带有精心选择的名称和一组特定命名参数的函数：

```js
function countNeedlesInHaystack(needle, haystack) { }
```

这个函数接受`needle`和`haystack`，并将返回`Number`，表示`needle`在`haystack`中的计数。我们代码的使用者会这样使用它：

```js
countNeedlesInHaystack('abc', 'abc abc abc'); // => 3
```

我们使用了寻找大海捞针的流行习语来描述在另一个字符串中寻找子字符串的问题。考虑流行的习语是设计代码的关键部分，但我们必须警惕习语被误解。

代码的设计应该由我们希望解决的问题领域和我们希望展现的用户体验来定义。另一个程序员在相同的问题领域下可能会选择不同的解决方案。例如，他们可能会使用部分应用来允许以下调用语法：

```js
needleCounter('app')('apple apple'); // => 2
```

或者他们可能设计了一个更冗长的语法，涉及调用`Haystack`构造函数并调用其`count()`方法，如下所示：

```js
new Haystack('apple apple'),count('app'); // => 2
```

这种*经典*方法可以说在对象（`Haystack`）和`count`方法之间有很好的语义关系。它与我们在之前章节中探讨过的面向对象编程概念很契合。尽管如此，一些程序员可能会觉得它过于冗长。

还有可能是更具描述性的 API，其中参数在配置对象中定义（即作为唯一参数传递的普通对象文字）：

```js
countOccurancesOfNeedleInHaystack({
  haystack: 'abc abc abc',
  needle: 'abc'
}); // => 3
```

还有可能这种计数功能可能是更大的一组与字符串相关的实用程序的一部分，因此可以并入一个更大的自定义命名模块：

```js
str('omg omg omg').count('omg'); // => 3
```

我们甚至可以考虑修改原生的`String.prototype`，尽管这是不可取的，以便我们在所有字符串上都有一个`count`方法可用：

```js
'omg omg omg'.count('omg'); // => 3
```

在我们的命名约定方面，我们可能希望避免使用大海捞针的习语，而是使用更具描述性的名称，也许有较少误解的风险，如下所示：

+   `searchableString`和`subString`

+   `查询`和`内容`

+   `search`和`corpus`

即使在这个非常狭窄的问题域内，我们可选择的选项也是令人难以置信的。您可能对哪种方法和命名约定更优越有很多自己的强烈意见。

我们可以用许多不同的方法解决一个看似简单的问题，这表明我们需要一个决策过程。而这个过程就是软件设计。有效的软件设计利用设计模式来封装问题域，并为同行程序员提供熟悉性和易于理解。

我们探索“大海捞针”问题的意图并不是为了找到解决方案，而是为了突出软件设计的困难，并让我们的思维接触更加以用户为导向的视角。它也提醒我们，很少有一个理想的设计。

任何问题域中，一个精心选择的设计模式可以说具有两个基本特征：

+   **它很好地解决了问题**：一个精心选择的设计模式将很好地适应问题域，以便我们可以轻松地表达问题的性质和其解决方案。

+   **它是熟悉和可用的**：一个精心选择的设计模式对我们的同行程序员来说是熟悉的。他们会立刻明白如何使用它或对代码进行更改。

设计模式在各种情境和规模下都是有用的。我们在编写单个操作和函数时使用它们，但我们在构建整个代码库结构时也使用它们。设计模式本身是分层的。它们存在于代码库的宏观和微观层面。一个单一的代码库很容易包含许多设计模式。

在第二章中，*《清洁代码的原则》*，我们谈到熟悉性是一个至关重要的特征。一个汽车技师打开汽车的引擎盖时，希望看到许多熟悉的模式：从独立的电线和组件的焊接到气缸、阀门和活塞的更大的构造。他们期望找到一定的布局，如果没有，他们将不知所措，想知道如何解决他们正在尝试解决的问题。

熟悉性增加了我们解决方案的可维护性和可用性。考虑以下目录结构和显示的`logger.js`源代码：

![](img/9c3cb805-3237-4651-b5b4-4e65a6188a83.png)

我们可以观察到哪些设计模式？让我们看一些例子：

+   使用顶层`app/`目录来包含所有源代码

+   **模型、视图和控制器**（MVC）的存在

+   将实用程序分离到自己的目录中（`utils/`）

+   文件的驼峰命名（例如，`binarySearch.js`）

+   `logger.js`中使用传统模块模式（即导出一组方法的普通对象）

+   使用`... && msgs.length`来确认非零（即真值）长度

+   在文件顶部声明常量（即`const ALL_LOGS_LEVEL`）

+   （可能还有其他的...）

设计模式不仅仅是庞大而高远的架构结构。它们可以存在于我们代码库的每个部分：目录结构、文件命名和我们代码的个别表达。在每个层面上，我们对常见模式的使用都可以增加我们表达问题域的能力，并增加我们代码对新手的熟悉度。模式存在于模式之中。

良好地使用设计模式可以对我们之前涵盖的清洁代码原则的所有要点产生有益影响——可靠性、效率、可维护性和可用性：

+   **可靠性**：一个好的设计模式将适合问题域，并允许您轻松表达所需的逻辑和数据结构，而不会太复杂。您采用的设计模式的熟悉性也将使其他程序员能够轻松理解和随着时间改进代码的可靠性。

+   **效率**：一个好的设计模式将使你不必过多地担心如何构建代码库或个别模块。它将使你能够花更多的时间关注问题域。精心选择的设计模式还将有助于使不同代码片段之间的接口变得简洁和易懂。

+   **可维护性**：一个好的设计模式可以轻松适应。如果有规范的变化或需要修复的错误，程序员可以轻松找到需要更改/插入的区域，并进行更改而不费力。

+   **可用性**：一个好的设计模式易于理解，因为它很熟悉。其他程序员可以轻松理解代码的流程，并快速做出正确的断言，了解它的工作原理以及如何使用它。一个好的设计模式还将为用户创造愉快的体验，无论是通过编程 API 还是 GUI 表达。

你可以看到，设计模式的许多有用之处只有在选择正确的模式时才能实现。我们将探讨一些流行的设计模式，并讨论它们适用的情况。这种探索应该让你对选择一个好的设计模式有一个很好的理解。

**请注意**：好的设计通过惯例传播，坏的设计也是如此。我们在第三章中讨论了“模仿神教”的现象，因此我们对这些类型的坏设计如何传播并不陌生，但在使用设计模式时保持警惕是很重要的。

# 架构设计模式

架构设计模式是我们将代码联系在一起的方式。如果我们有十几个不同的模块，那么这些模块如何相互通信就定义了我们的架构。

近年来，JavaScript 代码库中使用的架构设计模式发生了巨大变化。随着 React 和 Angular 等流行框架的不断普及，我们看到代码库采用了新的约定。这个领域仍在不断变化，所以我们不应该指望任何特定的标准很快出现。尽管如此，大多数框架往往遵循相同的广泛的架构模式。

一个流行的架构模式的例子是数据逻辑和渲染逻辑的分离。这被许多不同的 UI 框架所采用，尽管风格不同。这很可能是由于软件 UI 的传统和最终成为事实上的方法的 MVC 模式的传承。

在本节中，我们将介绍两种著名的架构设计模式，MVC 及其分支**Model-View-ViewModel**（**MVVM**）。这些应该让我们意识到通常分离的关注点，并希望激励我们在创建架构时寻求类似的清晰度。

# MVC

MVC 的特点是这三个概念之间的分离。MVC 架构可能涉及许多单独的模型、视图和控制器，它们共同解决一个给定的问题。这些部分可以描述如下：

+   **模型**：描述数据以及业务逻辑如何改变数据。数据的变化将导致视图的变化。

+   **视图**：描述模型如何呈现（其格式、布局和外观），并在需要发生动作时调用控制器，可能是响应用户事件。

+   **控制器**：接受来自视图的指令，并告知模型要执行哪些操作或更改，这将影响通过视图呈现给用户的内容。

我们可以在以下图表中观察控制流：

![](img/d5ca9398-52c3-4cdd-b323-74d3941bd397.png)

MVC 模式为我们提供了一种分离各种关注点的方法。它规定了我们应该将业务决策逻辑（即在模型中）放在哪里，以及我们应该将关于向用户显示事物的逻辑（即在视图中）放在哪里。此外，它还给我们提供了控制器，使这两个关注点能够相互交流。MVC 所促进的分离是非常有益的，因为这意味着我们的同行程序员可以很容易地辨别在哪里进行所需的更改或修复。

MVC 最初是由 Trygve Reenskaug 在 1978 年在施乐帕克（Xerox PARC）工作时提出的。它最初的目的是支持用户直接看到和操作域信息的幻觉。当时，这是相当革命性的，但现在，作为最终用户，我们认为这样的用户界面（以及它们与数据的透明关系）是理所当然的。

# MVC 的一个工作示例

为了说明 MVC 在 JavaScript 中的实现可能是什么样子，让我们构建一个非常简单的程序。它将是一个基本的可变数字应用程序，可以呈现一个简单的用户界面，用户可以在其中查看当前数字，并选择通过增加或减少其值来更新它。

首先，我们可以使用模型来实现数据的逻辑和包含：

```js
class MutableNumberModel {
  constructor(value) {
    this.value = value;
  }
  increment() {
    this.value++;
    this.onChangeCallback();
  }
  decrement() {
    this.value--;
    this.onChangeCallback();
  }
  registerChangeCallback(onChangeCallback) {
    this.onChangeCallback = onChangeCallback;
  }
}
```

除了存储值本身，这个类还接受并依赖于一个名为`onChangeCallback`的回调函数。这个回调函数将由控制器提供，并在值更改时调用。这是必要的，以便我们可以在模型更改时启动视图的重新渲染。

接下来，我们需要构建控制器，它将作为“视图”和“模型”之间非常简单的桥梁（或粘合剂）进行操作。它注册了必要的回调函数，以便知道用户通过“视图”请求更改或“模型”的基础数据发生更改时：

```js
class MutableNumberController {

  constructor(model, view) {

    this.model = model;
    this.view = view;

    this.model.registerChangeCallback(
      () => this.view.renderUpdate()
    );
    this.view.registerIncrementCallback(
      () => this.model.increment()
    );
    this.view.registerDecrementCallback(
      () => this.model.decrement()
    );
  }

}
```

我们的“视图”负责从“模型”中检索数据并将其呈现给用户。为此，它创建了一个 DOM 层次结构，其中数据将位于其中。它还在“增量”或“减量”按钮被点击时监听并升级用户事件到“控制器”：

```js
class MutableNumberView {

  constructor(model, controller) {
    this.model = model;
    this.controller = controller;
  }

  registerIncrementCallback(onIncrementCallback) {
    this.onIncrementCallback = onIncrementCallback;
  }

  registerDecrementCallback(onDecrementCallback) {
    this.onDecrementCallback = onDecrementCallback;
  }

  renderUpdate() {
    this.numberSpan.textContent = this.model.value;
  }

  renderInitial() {

    this.container = document.createElement('div');
    this.numberSpan = document.createElement('span');
    this.incrementButton = document.createElement('button');
    this.decrementButton = document.createElement('button');

    this.incrementButton.textContent = '+';
    this.decrementButton.textContent = '-';

    this.incrementButton.onclick =
      () => this.onIncrementCallback();
    this.decrementButton.onclick =
      () => this.onDecrementCallback();

    this.container.appendChild(this.numberSpan);
    this.container.appendChild(this.incrementButton);
    this.container.appendChild(this.decrementButton);

    this.renderUpdate();

    return this.container;

  }

}
```

这是一个相当冗长的视图，因为我们必须手动创建其 DOM 表示。许多现代框架（React、Angular、Svelte 等）允许您使用纯 HTML 或混合语法（例如 JSX，它是 JavaScript 本身的语法扩展，允许在 JavaScript 代码中使用类似 XML 的标记）来声明性地表达您的层次结构。

这个*视图*有两种渲染方法：`renderInitial`将进行初始渲染，设置 DOM 元素，然后`renderUpdate`方法负责在数字更改时更新数字。

将所有这些联系在一起，我们的简单程序将初始化如下：

```js
const model = new MutableNumberModel(5);
const view = new MutableNumberView(model);
const controller = new MutableNumberController(model, view);

document.body.appendChild(view.renderInitial());
```

“视图”可以访问“模型”，以便可以检索数据进行渲染。“控制器”分别提供给“模型”和“视图”，以便通过设置适当的回调将它们粘合在一起。

在用户点击`+`（增量）按钮的情况下，将启动以下过程：

1.  “增量按钮”的 DOM 点击事件由视图接收

1.  视图触发其`onIncrementCallback()`，由控制器监听

1.  控制器指示模型`increment()`

1.  模型调用其变异回调，即`onChangeCallback`，由控制器监听

1.  控制器指示视图重新渲染

也许你会想为什么我们要在控制器和模型之间进行分离。为什么*视图*不能直接与模型通信，反之亦然？其实可以！但如果我们这样做，我们会在*视图*和*模型*中都添加更多的逻辑，从而增加更多的复杂性。我们也可以把所有东西都放在*视图*中，而没有模型，但你可以想象那会变得多么笨重。从根本上讲，分离的程度和数量将随着你追求的每个项目而变化。在本质上，MVC 教会我们如何将问题域与其呈现分离。我们如何使用这种分离取决于我们自己。

自 1978 年 MVC 首次被提出以来，它的许多适配版本已经出现，但其将*模型*和*视图*分离的核心主题已经持续了几十年。考虑一下 React 应用程序的架构设计。它包括组件，其中包含呈现状态的逻辑，并且通常会包含几个特定领域的 reducer，它们会根据动作（例如*用户点击了某些东西！*）来推导状态。

这种架构看起来与传统的 MVC 非常相似：

![](img/3fb419a2-2ce5-4af8-a214-a7ad4d6d9d45.png)

作为一个通用的指导模式，MVC 在过去几十年中影响了无数框架和代码库的设计，并将继续如此。并非每个适配、复制或 MVC 都会遵循 1978 年提出的原始描述，但通常这些适配都会忠于将模型与视图分离以及使视图成为模型的反映（甚至是派生）的核心重要主题。

# MVVM

MVVM 的精神与其祖先 MVC 相似。它规定了程序的基础业务逻辑和驱动程序的数据与数据的呈现之间严格的分离：

+   **模型**：描述数据以及业务逻辑如何改变数据。数据的变化将体现在**视图**中。

+   **视图**：描述**模型**的呈现方式（其结构、布局和外观），并且在需要发生动作时会调用**视图模型**的**数据绑定**机制，可能是响应用户事件。

+   **视图模型**：这是**模型**和**视图**之间的粘合剂，并通过**数据绑定**机制使它们能够相互通信。这种机制在不同的实现中往往有很大的变化。

这些部分之间的关系如下图所示：

![](img/e3109724-cf07-41a4-8303-a04730b7f4ce.png)

MVVM 架构在前端 JavaScript 中更受欢迎，因为它适应了需要不断更新的**视图**，而传统的 MVC 在后端更受欢迎，因为它很好地满足了大多数 HTTP 响应只需简单渲染一次的特性。

在 MVVM 中，**视图模型**和**视图**之间的数据绑定通常使用 DOM 事件来跟踪用户意图，然后在**模型**上改变数据，**模型**再发出自己的变化事件，**视图模型**可以监听这些事件，从而使**视图**始终保持与数据的变化同步。

许多框架都会有自己的数据绑定适配。例如，Angular 允许您在 HTML 模板中指定一个名为`ng-model`的自定义属性，它将把用户输入元素（如`<input>`）与给定的数据模型绑定在一起，从而实现数据的双向流动。如果模型更新了，`<input>`也会更新以反映这一点，反之亦然。

# MV*和软件的性质

在您作为 JavaScript 程序员的时间里，您将遇到 MVC 和 MVVM 的各种变体。作为模式，它们是无限适用的，因为它们关注的是软件系统的非常基本的原则：将数据输入系统，处理该数据，然后输出处理后的数据。我们可以选择将这些原则架构到代码库中的其他几种方式，但最终，几乎每次，我们最终都会得到一个类似 MVC（或 MVVM）的系统，它以类似的精神划分了这些关注点。

现在我们已经对如何构建代码库以及表征良好设计架构的类型有了明确的想法，我们可以探索代码库的各个部分：模块本身。

# JavaScript 模块

在 JavaScript 中，模块这个词多年来发生了变化。模块曾经是任何独立且自包含的代码片段。几年前，您可能会在同一个文件中表达几个模块，就像这样：

```js
// main.js

// The Dropdown Module
var dropdown = /* ... definition ... */;

// The Data Fetcher Module
var dataFetcher = /* ... definition ...*/;
```

然而，如今，*module*这个词倾向于指的是 ECMAScript 规范所规定的 Modules（大写*M)*。这些模块是通过`import`和`export`语句在代码库中导入和导出的不同文件。使用这样的模块，我们可能有一个`DropdownComponent.js`文件，看起来像这样：

```js
// DropdownComponent.js
class DropdownComponent {}
export default DropdownComponent;
```

正如您所看到的，它使用`export`语句来导出其类。如果我们希望将这个类用作依赖项，我们将这样导入它：

```js
// app.js
import DropdownComponent from './DropdownComponent.js'; 
```

ECMAScript 模块正在逐渐在各种环境中得到更多支持。要在浏览器中使用它们，可以提供一个类型为*module*的*entry*脚本标签，即`<script type="module" />`。在 Node.js 中，目前，ES 模块仍然是一个实验性功能，因此您可以依赖旧的导入方式（`const thing = require('./thing')`），或者可以通过使用`--experimental-modules`标志并在所有 JavaScript 文件上使用`.mjs`扩展名来启用*实验性模块*。

`import`和`export`语句都允许各种语法。这使您可以定义要导出或导入的名称。在只导出一个项目的情况下，惯例上使用`export default [item]`，就像我们在`DropdownComponent.js`中所做的那样。这确保了模块的任何依赖项都可以导入它并根据需要命名它，就像这个例子中所示的那样：

```js
import MyLocallyDifferentNameForDropdown from './DropdownComponent.js';
```

与此相反，您可以通过在花括号内声明它们并使用`as`关键字来明确命名您的导出项：

```js
export { DropdownComponent as TheDropdown };
```

这意味着任何导入者都需要明确指定`TheDropdown`的名称，如下所示：

```js
import { TheDropdown } from './DropdownComponent.js'; 
```

或者，您可以通过在`export`语句中具体声明来导出命名项，例如`var`、`const`、`let`、函数声明或类定义：

```js
// things.js
export let x = 1;
export const y = 2;
export var z = 3;
export function myFunction() {}
export class MyClass {}
```

在导入方面，这些命名导出可以通过再次使用花括号来导入：

```js
import { x, y, z, myFunction, MyClass } from './things.js'; 
```

在导入时，您还可以选择使用`as`关键字指定该导入的本地名称，使其本地名称与其导出的名称不同（在命名冲突的情况下，这是特别有用的）：

```js
import { MyClass as TheClass } from './things.js';
TheClass; // => The class
MyClass; // ! ReferenceError
```

在您的代码中，将导出聚合在一起的区域通常是惯例，这些区域提供了几个相关的抽象。例如，如果您已经组合了一个小的组件库，其中每个组件都将自己作为`default`导出，那么您可以有一个`index.js`，将所有组件一起公开：

```js
// components/index.js
export {default as DropdownComponent} from './DropdownComponent.js';
export {default as AccordianComponent} from './AccordianComponent.js';
export {default as NavigationComponent} from './NavigationComponent.js';
```

在 Node.js 中，默认情况下，如果尝试导入整个目录，将导入`index.js`/`index.mjs`文件。也就是说，如果您导入`'./components/'`，它首先会查找索引文件，如果可用，会导入它。在浏览器中，目前没有这样的约定。所有导入必须是完全限定的文件名。

我们现在可以通过在`import`语句中使用星号来非常方便地导入我们的整套组件：

```js
// app.js
import * from 'components/index.js';

// Make use of the imported components:
new DropdownComponent();
new AccordianComponent();
new NavigationComponent();
```

在 JavaScript 中，模块存在一些额外的细微差别和复杂性，特别是考虑到 Node.js 的遗留问题，不幸的是，我们没有时间深入讨论，但到目前为止，我们所涵盖的内容应该足够让你对这个主题有一个很好的了解，以便提高生产力，并为我们探讨模块化设计模式铺平道路。

# 模块化设计模式

模块化设计模式是我们用来构建单个模块的结构和语法约定。我们通常会在不同的 JavaScript 模块中使用这些模式。每个不同的文件应该提供并导出一个特定的抽象。

如果您发现自己在同一个文件中多次使用这些模式，那么将它们拆分出来可能是值得的。给定代码库的目录和文件结构应该理想地反映其抽象的景观。您不应该将多个抽象塞进一个文件中。

# 构造函数模式

构造函数模式使用一个构造函数，然后手动填充其`prototype`方法和属性。这是在类定义语法存在之前，在 JavaScript 中创建经典面向对象类的传统方法。

通常，它以构造函数的定义作为函数声明开始：

```js
function Book(title) {
  // Initialization Logic
  this.title = title;
}
```

然后会跟着将单独的方法分配给原型：

```js
Book.prototype.getNumberOfPages = function() { /* ... */ };
Book.prototype.renderFrontCover: function() { /* ... */ };
Book.prototype.renderBackCover: function () { /* ... */ };
```

或者它将被跟随着用一个对象字面量替换整个`prototype`：

```js
Book.prototype = {
  getNumberOfPages: function() { /* ... */ },
  renderFrontCover: function() { /* ... */ },
  renderBackCover: function () { /* ... */ }
};
```

后一种方法往往更受青睐，因为它更封装和简洁。当然，现在，如果您希望使用构造函数模式，您可能会选择方法定义，因为它们占用的空间比单独的键值对少：

```js
Book.prototype = {
  getNumberOfPages() { /* ... */ },
  renderFrontCover() { /* ... */ },
  renderBackCover () { /* ... */ }
};
```

构造函数的实例化将通过`new`关键字进行：

```js
const myBook = new Book();
```

这将创建一个具有构造函数的`prototype`的内部`[[Prototype]]`（也就是我们的对象，其中包含`getNumberOfPages`、`renderFrontCover`和`renderBackCover`）。

如果您对构造函数和实例化的原型机制记忆不太清楚，请重新阅读第六章，*基本类型和内置类型*，特别是名为*原型*的部分。

# 何时使用构造函数模式

构造函数模式在您希望有一个封装名词概念的抽象的情况下非常有用，也就是说，一个有意义的实例。例如`NavigationComponent`或`StorageDevice`。构造函数模式允许您创建类似于传统面向对象编程类的抽象。因此，如果您来自经典的面向对象编程语言，那么您可以随意使用构造函数模式，以前可能使用类。

如果您不确定构造函数模式是否适用，请考虑以下问题是否属实：

+   概念可以表达为名词吗？

+   概念需要构造吗？

+   概念在实例之间会有变化吗？

如果你抽象出的概念不满足以上任何标准，那么你可能需要考虑另一种模块化设计模式。一个例子是一个具有各种辅助方法的实用程序模块。这样的模块可能不需要构造，因为它本质上是一组方法，这些方法及其行为在实例之间不会有变化。

自从 JavaScript 引入类定义以来，构造函数模式已经大多不受青睐，类定义允许您以更类似于经典面向对象编程语言的方式声明类（即`class X extends Y {...}`）。跳转到*类模式*部分，看看这个模式是如何运作的！

# 构造函数模式中的继承

要使用构造函数模式实现继承，您需要手动使您的`prototype`对象从父构造函数的`prototype`继承。

冒着过于简化的风险，我们将以`Animal`超类和`Monkey`子类的经典示例来说明这一点。这是我们对`Animal`的定义：

```js
function Animal() {}
Animal.prototype = {
  isAnimal: true,
  grow() {}
};
```

从技术上讲，为了实现继承，我们希望创建一个具有`Animal.prototype`原型的`[[Prototype]]`的对象，然后将这个新创建的对象用作我们的子类`prototype`子类。最终目标是一个看起来像这样的原型树：

```js
Object.prototype
 └── Animal.prototype
      └── Monkey.prototype
```

使用`Object.create(ThePrototype)`是创建具有给定`[[Prototype]]`的对象的最简单方法。在这里，我们可以使用它来扩展`Animal.prototype`并将结果分配给`Monkey.prototype`：

```js
function Monkey() {}
Monkey.prototype = Object.create(Animal.prototype);
```

然后我们可以自由地将方法和属性分配给这个新对象：

```js
Monkey.prototype.isMonkey = true;
Monkey.prototype.screech = function() {};
```

如果我们现在尝试实例化`Monkey`，那么我们应该能够访问不仅其自己的方法和属性，还有我们从`Animal.prototype`继承的方法和属性：

```js
new Monkey().isAnimal; // => true
new Monkey().isMonkey; // => true
typeof new Monkey().grow; // => "function"
typeof new Monkey().screech; // => "function"
```

**记住**，这只能工作是因为`Monkey.prototype`（也就是每个`Monkey`实例的`[[Prototype]]`）本身具有`Animal.prototype`的`[[Prototype]]`。而且，正如我们所知，如果在给定对象上找不到属性，那么它将在其`[[Prototype]]`上寻找（*如果可用*）。

一次设置一个原型的属性和方法可能会非常麻烦，就像在这个例子中所示的那样：

```js
Monkey.prototype.method1 = ...;
Monkey.prototype.method2 = ...;
Monkey.prototype.method3 = ...;
Monkey.prototype.method4 = ...;
```

由于这一点，另一种模式已经出现，使事情变得更容易：使用`Object.assign()`。这允许我们以对象文字的形式批量设置属性和方法，并且意味着我们也可以利用方法定义语法：

```js
function Monkey() {}
Monkey.prototype = Object.assign(Object.create(Animal.prototype), {
  isMonkey: true, 
  screech() {},
  groom() {}
});
```

`Object.assign`将会将其第二个（第三个、第四个等等）参数中的任何属性分配给作为第一个参数传递的对象。这为我们提供了一种更简洁的语法，用于向我们的子`prototype`对象添加属性。

由于较新的类定义语法，构造函数模式及其继承约定已经大大失宠，它允许以更简洁和简单的方式在 JavaScript 中利用原型继承。因此，我们将要探索的下一个内容就是类模式，它使用了这种较新的语法。

**提醒**：要更彻底地了解`[[Prototype]]`（这对于理解 JavaScript 中的构造函数和类至关重要），您应该重新访问第六章中关于*原型*的部分，*基本类型和内置类型*。本章中的许多设计模式都使用了原型机制，因此将其牢记在心是很有用的。

# 类模式

类模式依赖于较新的类定义语法，已经在很大程度上取代了构造函数模式。它涉及创建类，类似于经典的面向对象编程语言，尽管在幕后它使用了与构造函数模式相同的原型机制。因此，可以说这只是一点额外的语法*糖*，使语言更加表达。

这是一个抽象名称概念的基本类的示例：

```js
class Name {
  constructor(forename, surname) {
    this.forename = forename;
    this.surname = surname;
  }
  sayHello() {
   return `My name is ${this.forename} ${this.surname}`;
  }
}
```

通过这种语法创建类实际上是创建一个带有附加原型的构造函数，因此以下代码是完全等效的：

```js
function Name(forename, surname) {
  this.forename = forename;
  this.surname = surname;
}

Name.prototype.sayHello = function() {
  return `My name is ${this.forename} ${this.surname}`;
};
```

使用类模式在美学上确实比笨重且陈旧的构造函数模式更可取，但不要被误导！在幕后，完全相同的机制正在发挥作用。

# 何时使用类模式

类模式，就像构造函数模式一样，在满足以下标准的自包含概念时非常有用：

+   这个概念可以表示为一个名词

+   这个概念需要构建

+   这个概念将在其自身的实例之间变化

以下是一些符合这些标准并因此可以通过类模式表达的概念的示例：

+   数据库记录（表示一条数据并允许查询和操作）

+   待办事项组件（表示待办事项并允许其被渲染）

+   二叉树（表示二叉树数据结构）

这样的情况通常会很明显地显露出来。如果你遇到麻烦，请考虑你的抽象的用例，并尝试编写一些消费者代码，也就是使用你的抽象的伪代码。如果它看起来合理，使用起来不会太尴尬，那么你可能已经找到了一个好的模式。

# 静态方法

静态方法和属性可以通过使用`static`关键字声明：

```js
class Accounts {
  static allAccounts = [];
  static tallyAllAccounts() {
    // ...
  }
}

Accounts.tallyAllAccounts();
Accounts.allAccounts; // => []
```

这些属性和方法也可以在初始类定义之后轻松添加：

```js
Accounts.countAccounts = () => {
  return Accounts.allAccounts.length;
};
```

静态方法在整个类的语义上与单个实例不同，因此在这种情况下非常有用。

# 公共和私有字段

要在实例上声明公共字段（即属性），您可以在类定义语法中简单地声明它：

```js
class Rectangle {
  width = 100;
  height = 100;
}
```

这些字段为每个实例初始化，并且因此在实例本身上是可变的。当您需要为给定属性定义一些合理的默认值时，它们最有用。然后可以在构造函数中轻松地覆盖它们。

```js
class Rectangle {
  width = 100;
  height = 100;

  constructor(width, height) {
    if (width && !isNaN(width)) {
      this.width = width;
    }
    if (height && !isNaN(height)) {
      this.height = height;
    }
  }
}
```

您还可以通过在其标识符前加上`#`符号来定义私有字段：

```js
class Rectangle {
  #width = 100;
  #height = 100;

  constructor(width, height) {
    if (width && !isNaN(width)) {
      this.#width = width;
    }
    if (height && !isNaN(height)) {
      this.#height = height;
    }
  }
}
```

传统上，JavaScript 没有私有字段的概念，因此程序员选择用一个或多个下划线作为前缀来表示私有属性（例如，`__somePropertyName`）。这被理解为一种社会契约，其他程序员不会干扰这些属性（知道这样做可能会以意想不到的方式破坏事情）。

私有字段只能被类本身访问。子类无法访问：

```js
class Super { #private = 123; }
class Sub { getPrivate() { return this.#private; } }

// !SyntaxError: Undefined private field #private:
// must be declared in an enclosing class
```

私有字段应该极度谨慎地使用，因为它们可能严重限制代码的可扩展性，从而增加其刚性和缺乏灵活性。如果您使用私有字段，您应该确保已经考虑了后果。也许你需要的是一个伪私有字段，实际上只是一个带有下划线前缀的字段（例如，`_private`）或另一个不常见的标点符号（例如，`$_private`）。这样做将会按照惯例确保使用您接口的其他程序员（希望）理解他们不应该公开使用该字段。如果他们这样做，那么暗示着他们可能会破坏事情。如果他们希望用自己的实现扩展您的类，那么他们可以自由地使用您的私有字段。

# 扩展类

类模式内的继承可以通过使用`class ... extends`语法来非常简单地实现：

```js
class Animal {}
class Tiger extends Animal {}
```

这将确保`Tiger`的任何实例都将具有`[[Prototype]]`，它本身具有`Animal.prototype`的`[[Prototype]]`。

```js
Object.getPrototypeOf(new Tiger()) === Tiger.prototype;
Object.getPrototypeOf(Tiger.prototype) === Animal.prototype;
```

在这里，我们已经确认`Tiger`的每个新实例都有`Tiger.prototype`的`[[Prototype]]`，并且`Tiger.prototype`继承自`Animal.prototype`。

# 混合类

传统上，扩展不仅用于创建语义子类，还用于提供方法的混入。JavaScript 没有提供原生的混入机制，因此为了实现它，您需要在定义之后增加原型，或者有效地从您的混入中继承（就好像它们是超类一样）。

通过将混入作为对象指定，然后通过方便的方法（例如`Object.assign`）将它们添加到类的`prototype`中，我们可以最简单地使用`prototype`来扩充我们的混入。

```js
const fooMixin = { foo() {} };
const bazMixin = { baz() {} };

class MyClass {}
Object.assign(MyClass.prototype, fooMixin, bazMixin);
```

然而，这种方法不允许`MyClass`覆盖自己的混合方法：

```js
// Specify MyClass with its own foo() method:
class MyClass { foo() {} }

// Apply Mixins:
Object.assign(MyClass.prototype, fooMixin, bazMixin);

// Observe that the mixins have overwritten MyClass's foo():
new MyClass().foo === fooMixin.foo; // true (not ideal)
```

这是预期的行为，但在某些情况下会给我们带来麻烦。因此，为了实现更通用的混合方法，我们可以探索不同的机制。我们可以使用继承来实现这一点，这最容易通过所谓的*子类工厂*来实现。这些本质上只是返回一个扩展指定超类的类的函数：

```js
const fooSubclassFactory = SuperClass => {
 return class extends SuperClass {
   fooMethod1() {}
   fooMethod2() {}
 };
};
```

这是它在现实中可能的工作方式的一个例子：

```js
const greetingsMixin = Super => class extends Super {
  hello() { return 'hello'; }
  hi() { return 'hi'; }
  heya() { return 'heya'; }
};

class Human {}
class Programmer extends greetingsMixin(Human) {}

new Programmer().hi(); // => "hi"
```

我们还可以实现一个辅助程序来组合任意数量的这些*子类工厂*。它可以通过构建与我们提供的`mixins`列表一样长的`[[Prototype]]`链接的链（或树）来实现：

```js
function mixin(...mixins) {
  return mixins.reduce((base, mixin) => {
    return mixin(base);
  }, Object);
}
```

请注意我们的 mixin 减少的默认`base`类是`Object`。这是为了确保`Object`始终位于我们的继承树的顶部（并且我们不会创建无意义的中间类）。

以下是我们如何使用我们的`mixin`助手：首先，我们将定义我们的子类工厂（即实际的 mixin）：

```js
const alpha = Super => class extends Super { alphaMethod() {} };
const bravo = Super => class extends Super { braveMethod() {} };
```

然后，我们可以通过`mixin`助手使用这些 mixin 构造一个类定义：

```js
class MyClass extends mixin(alpha, bravo) {
  myMethod() {}
};
```

这意味着结果的`MyClass`实例将可以访问其自己的原型（包含`myMethod`），alpha 的原型（包含`alphaMethod`）和 bravo 的原型（包含`bravoMethod`）：

```js
typeof new MyClass().myMethod;    // => "function"
typeof new MyClass().alphaMethod; // => "function"
typeof new MyClass().braveMethod; // => "function"
```

混合可能很难搞对，因此最好利用一个库或经过验证的代码来为您处理这些。您应该使用的 mixin 机制可能取决于您所寻求的确切特征。在本节中，我们看到了两个例子：一个是通过`Object.assign()`将方法组合成一个单一的`[[Prototype]]`，另一个是创建一个继承树（即`[[Prototypes]]`链）来表示我们的 mixin 层次结构。希望您现在能更好地选择这些（或者在线提供的所有其他）中的哪一个最适合您的需求。

# 访问超类

使用方法定义语法定义的类中的所有函数都可以使用`super`绑定，它提供对超类及其属性的访问。`super()`函数可直接调用（将调用超类的构造函数），并且可以提供对特定方法的访问（`super.methodName()`）。

如果您正在扩展另一个类并且正在定义自己的构造函数，则必须调用`super()`，并且必须在构造函数内修改实例（即`this`）的任何其他代码之前这样做：

```js
class Tiger extends Animal {
  constructor() {
    super(); // I.e. Call Animal's constructor
  }
}
```

如果您的构造函数在修改实例后尝试调用`super()`，或者尝试避免调用`super()`，那么您将收到`ReferenceError`：

```js
class Tiger extends Animal {
  constructor() {
    this.someProperty = 123;
    super(); 
  }
}

new Tiger();
// ! ReferenceError: You must call the super constructor in a derived class
// before accessing 'this' or returning from the derived constructor
```

有关`super`绑定及其特殊性的详细描述，请参阅第六章，*基本类型和内置类型*（请参阅*函数绑定*部分）。

# 原型模式

原型模式涉及使用普通对象作为其他对象的模板。原型模式直接扩展此模板对象，而不需要通过`new`或`Constructor.prototype`对象进行实例化。您可以将其视为类似于传统构造函数或类模式，但没有构造函数。

通常，您将首先创建一个对象作为您的模板。这将具有与您的抽象相关的所有方法和属性。对于`inputComponent`抽象，它可能如下所示：

```js
const inputComponent = {
  name: 'Input Component',
  render() {
    return document.createElement('input');
  }
};
```

请注意`inputComponent`以小写字符开头。按照惯例，只有构造函数应以大写字母开头命名。

使用我们的`inputComponent`模板，我们可以使用`Object.create`创建（或实例化）特定的实例：

```js
const inputA = Object.create(inputComponent);
const inputB = Object.create(inputComponent);
```

正如我们所学的，`Object.create(thePrototype)`简单地创建一个新对象，并将其内部的`[[Prototype]]`属性设置为`thePrototype`，这意味着在新对象上访问的任何属性，如果在对象本身上不可用，将在`thePrototype`上查找。因此，我们可以像在更传统的构造函数或类模式产生的实例上一样对待生成的对象，访问属性：

```js
inputA.render();
```

为了方便起见，我们还可以在`inputComponent`本身上引入一个设计用于执行对象创建工作的方法：

```js
inputComponent.extend = function() {
  return Object.create(this);
};
```

这意味着我们可以用稍微少一些的代码创建单独的实例：

```js
const inputA = inputComponent.extend();
const inputB = inputComponent.extend();
```

如果我们希望创建其他类型的输入，那么我们可以像我们已经做的那样轻松扩展`inputComponent`，向生成的对象添加一些方法，然后将新对象提供给其他人扩展：

```js
const numericalInputComponent = Object.assign(inputComponent.extend(), {
  render() {
    const input = InputComponent.render.call(this);
    input.type = 'number';
    return input;
  }
});
```

要覆盖特定方法并访问其父级，如你所见，我们需要直接引用并调用它（`InputComponent.render.call()`）。你可能期望我们应该能够使用`super.render()`，但不幸的是，`super`只是指包含方法所定义的对象（主体）的`[[Prototype]]`。因为`Object.assign()`实际上是从它们的主体对象中窃取这些方法，`super`将指向错误的东西。

原型模式的命名相当令人困惑。正如我们所见，传统的构造函数模式和较新的类模式都涉及原型，因此你可能希望将这种模式称为*对象扩展模式*，甚至是*无构造函数的原型继承方法*。无论你决定如何称呼，这都是一种相当罕见的模式。通常更受青睐的是经典的 OOP 模式。

# 何时使用原型模式

原型模式在具有实例之间变化特征的抽象情景中最有用（或扩展），但不需要构造。在其核心，原型模式实际上只是指扩展机制（即通过`Object.create`），因此它同样可以用于任何情景，其中对象可能通过继承在语义上与其他对象相关。

想象一种需要表示三明治数据的情景。每个三明治都有一个名称，一个面包类型和三个成分槽。例如，这是 BLT 的表示：

```js
const theBLT = {
  name: 'The BLT',
  breadType: 'Granary',
  slotA: 'Bacon',
  slotB: 'Lettuce',
  slotC: 'Tomato'
};
```

我们可能希望创建 BLT 的一个适应版本，重用其大部分特征，除了`Tomato`成分，它将被替换为`Avocado`。我们可以简单地克隆整个对象，通过使用`Object.assign`从`theBLT`复制所有属性到一个新对象，然后专门复制（即覆盖）`slotC`：

```js
const theBLA = Object.assign({}, theBLT, {
  slotC: 'Avocado'
});
```

但是如果 BLT 的`breadType`被更改了呢？让我们来看一下：

```js
theBLT.breadType = 'Sourdough';
theBLA.breadType; // => 'Granary'
```

现在，`theBLA`与`theBLT`不同步。我们已经意识到，这里实际上需要的是一个继承模型，以便`theBLA`的`breadType`始终与其父三明治的`breadType`匹配。为了实现这一点，我们可以简单地改变我们创建`theBLA`的方式，使其继承自`theBLT`（使用原型模式）：

```js
const theBLA = Object.assign(Object.create(theBLT), {
  slotC: 'Avocado'
});
```

如果我们稍后更改`theBLT`的特征，它将有助于通过继承在`theBLA`中得到反映：

```js
theBLT.breadType = 'Sourdough';
theBLA.breadType; // => 'Sourdough'
```

可以看到，这种无构造函数的继承模型在某些情况下是有用的。我们可以同样使用直接的类来表示这些数据，但是对于如此基本的数据来说可能有些过度。原型模式有用之处在于它提供了一个简单明确的继承机制，可以导致更少臃肿的代码（尽管同样地，如果错误应用，也可能导致更多的复杂性）。

# 揭示模块模式

揭示模块模式是一种用于封装一些私有逻辑然后公开公共 API 的模式。这种模式有一些改编，但通常是通过**立即调用的函数表达式**（**IIFE**）来表达的，它返回一个包含公共方法和属性的对象字面量：

```js
const myModule = (() => {
  const privateFoo = 1;
  const privateBaz = 2;

  // (Private Initialization Logic goes here)

  return {
    publicFoo() {},
    publicBaz() {}
  };
})();
```

IIFE 返回的任何函数将形成一个封闭环绕其各自作用域的闭包，这意味着它们将继续访问*私有*作用域。

一个真实世界的揭示模块的例子是这个包含渲染通知给用户逻辑的简单 DOM 组件：

```js
const notification = (() => {

  const d = document;
  const container = d.body.appendChild(d.createElement('div'));
  const message = container.appendChild(d.createElement('p'));
  const dismissBtn = container.appendChild(d.createElement('button'));

  container.className = 'notification';

  dismissBtn.textContent = 'Dismiss!';
  dismissBtn.onclick = () => {
    container.style.display = 'none';
  };

  return {
    display(msg) {
      message.textContent = msg;
      container.style.display = 'block';
    }
  };
})();
```

外部作用域中的通知变量将引用 IIFE 返回的对象，这意味着我们可以访问它的公共 API：

```js
notification.display('Hello user! Something happened!');
```

揭示模块模式在需要区分私有和公共部分、具有特定初始化逻辑以及由于某种原因，你的抽象不适合更面向对象的模式（类或构造函数模式）的场景中特别有用。

在类定义和`#private`字段存在之前，揭示模块模式是模拟真正私有性的唯一简单方法。因此，它已经有些过时。一些程序员仍然使用它，但通常只是出于审美偏好。

# 传统模块模式

传统模块模式通常表示为一个普通的对象文字，带有一组方法：

```js
const timeDiffUtility = {
  minutesBetween(dateA, dateB) {},
  hoursBetween(dateA, dataB) {},
  daysBetween(dateA, dateB) {}
};
```

这样的模块通常还会公开特定的初始化方法，例如`initialize`、`init`或`setup`。或者，我们可能希望提供改变整个模块状态或配置的方法（例如`setConfig`）：

```js
const timeDiffUtility = {
  setConfig(config) {
    this.config = config;
  },
  minutesBetween(dateA, dateB) {},
  hoursBetween(dateA, dataB) {},
  daysBetween(dateA, dateB) {}
};
```

传统模块模式非常灵活，因为它只是一个普通对象。JavaScript 将函数视为一等公民（也就是说，它们就像任何其他值一样），这意味着您可以轻松地从其他地方定义的函数组合方法的对象，例如：

```js
const log = () => console.log(this);

const library = {
  books: [],
  addBook() {},
  log // add log method
};
```

传统上，您可能考虑使用继承或混合模式将`log`方法包含在我们的库模块中，但在这里，我们只是通过引用并直接插入到我们的对象中来组合它。这种模式在 JavaScript 中为我们提供了很大的灵活性，可以灵活地重用代码。

# 何时使用传统模块模式

传统模块模式在任何您希望将一组相关方法或属性封装成具有共同名称的东西的情况下都很有用。它们通常用于与彼此相关的常见方法集合，例如日志记录实用程序：

```js
const logger = {
  log(message) { /* ... */ },
  warn(message) { /* ... */ },
  error(message) { /* ... */ }
};
```

传统模块模式只是一个对象，因此可能根本不需要提及它。但从技术上讲，它是抽象定义的其他技术的替代方案，因此将其指定为一种模式是有用的。

# 单例类模式

类模式已经迅速成为创建各种类型的抽象的事实标准模式，包括单例和实用对象，因此您的类可能并不总是需要作为传统的面向对象类进行使用，包括继承和实例化。例如，我们可能希望使用类定义来设置一个实用对象，以便我们可以在构造函数中定义任何初始化逻辑，并在其方法中提供封装的外观：

```js
const utils = new class {
  constructor() {
    this.#privateThing = 123;
    // Other initialization logic here...
  }
  utilityA() {}
  utilityB() {}
  utilityC() {}
};

utils.utilityA(); 
```

在这里，我们创建并立即实例化一个类。这在精神上类似于揭示模块模式，我们利用 IIFE 来封装初始化逻辑和公共 API。在这里，我们不是通过作用域（以及围绕私有变量的闭包）来实现封装，而是使用直接的构造函数来定义初始化。然后，我们使用常规实例属性和方法来定义私有变量和公共接口。

# 何时使用单例类模式

单例在需要一个类的唯一实例时非常有用。产生的单一实例在性质上类似于传统或揭示模块模式。它使您能够用私有变量和隐式构造逻辑封装一个抽象。单例的常见用例包括*实用程序*、*日志记录*、*缓存*、*全局事件总线*等。

# 规划和和谐

决定使用哪些架构和模块化设计模式可能是一个棘手的过程，因为通常在决定的时候，项目的所有要求可能并不立即显而易见。此外，我们作为程序员并不是全知全能的。我们是有缺陷的、自我中心的，通常是充满激情的个体。如果不加以控制，这种组合可能会产生混乱的代码库，其中的设计会阻碍我们试图培养的生产力、可靠性和可维护性。要警惕这些陷阱，记住以下内容：

+   期待变化和适应：每个软件项目都会在某个时候涉及变化。如果我们在架构和模块化设计上有远见，那么我们将能够限制未来的痛苦，但永远不要开始一个项目，认为你会创造“唯一真正的解决方案”。相反，迭代，质疑自己的判断，然后再次迭代。

+   与其他程序员协商：与将使用您的代码的利益相关者交谈。这可能是您团队中的其他程序员，或者将使用您提供的接口的其他程序员。听取意见和数据，然后做出明智的决定。

+   避免模仿和自我：要注意模仿和自我，要小心，如果我们不小心，我们可能会盲目地继承做事情的方式，而不是关键地考虑它们的适用性，或者我们可能会被自己的自我困住：认为某种特定的设计或方法论是完美的，只是因为这是我们个人所知道和喜爱的。

+   偏向和谐与一致性：在设计架构时，最重要的是寻求和谐。代码库中总是可能有许多个性化定制的部分，但太多的内部差异会让维护者困惑，并导致代码库的分裂质量和可靠性。

# 总结

在本章中，我们探讨了 JavaScript 中设计模式的目的和应用。这涵盖了对什么是设计模式的基础性反思，以及对一些常见的模块化和架构设计模式的探索。我们已经探讨了使用 JavaScript 的原生机制（如类和原型）以及一些更新颖的机制（如揭示模块模式）声明抽象的各种方式。我们对这些模式的深入覆盖将确保在未来，我们在构建抽象时有充分的选择。

在下一章中，我们将探讨 JavaScript 程序员遇到的现实挑战，如状态管理和网络通信，并将我们对新视角的知识应用到其中。
