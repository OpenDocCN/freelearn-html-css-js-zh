# 第七章。测试 React 应用程序

作为一名 Web 开发者，您熟悉大多数网站今天是如何构建的。通常有一个 Web 服务器（在 Java、Ruby 或 PHP 等语言中），它处理用户请求并以标记（HTML）的形式响应。

这意味着在每次请求时，Web 服务器通过 URL 解释用户操作并渲染整个页面。

为了提高用户体验，越来越多的功能开始从服务器端推向客户端，JavaScript 不再仅仅是向页面添加行为，而是完全渲染页面。最大的优势是用户操作不再触发整个页面的刷新；JavaScript 代码可以处理整个浏览器文档并根据需要进行修改。

虽然这确实提高了用户体验，但它开始给应用程序代码增加了很多复杂性，导致维护成本增加，最糟糕的是——屏幕不同部分之间不一致的 bug。

为了使这种场景变得合理，许多库和框架被构建出来，但它们都失败了，因为它们没有解决整个问题的根本原因——可变性。

服务器端渲染很容易，因为没有要处理的可变性。给定一个新的应用程序状态，服务器会简单地重新渲染一切。如果我们能在客户端 JavaScript 代码中从这种方法中获得好处怎么办？

这正是**React**所提出的。您以组件的形式声明性地编写界面代码，并告诉 React 进行渲染。在任何应用程序状态变化时，您可以简单地告诉 React 重新渲染；然后它会计算将 DOM 移动到所需状态的所需更改，并为您应用它们。

在本章中，我们将通过重构到目前为止所编写的代码，将其重构为一个单页应用（SPA），来理解 React 是如何工作的。

# 项目设置

然而，在我们深入 React 之前，首先需要在我们的项目中做一些小的设置，以便我们能够创建 React 组件。

前往[`facebook.github.io/react/downloads.html`](http://facebook.github.io/react/downloads.html)下载 React Starter Kit 版本 0.12.2 或更高版本。

下载后，您可以解压其内容，并将构建文件夹内的所有文件移动到我们的应用程序的 lib 文件夹中。然后，只需将 React 库加载到`SpecRunner.html`文件中。

```js
<script src="img/react-with-addons.js"></script>
<script src="img/jquery.js"></script>
```

设置完成后，我们可以继续编写我们的第一个组件。

# 我们的第一个 React 组件

如本章引言所述，使用 React，您通过组件声明性地编写界面代码。

React 组件的概念与第三章中提出的组件概念类似，即*测试前端代码*，因此期待在下一部分看到一些相似之处。

考虑到这一点，让我们创建我们非常第一个组件。为了更好地理解 React 组件是什么，我们将使用一个非常简单的验收标准，并像往常一样从规范开始。

让我们实现 "InvestmentListItem 应该渲染"。这很简单，并不真正是 *面向功能* 的，但是一个很好的例子，让我们开始。

根据 第三章 中学到的内容，*测试前端代码*，我们可以通过创建一个名为 `InvestmentListItemSpec.js` 的新文件并保存在 `spec` 文件夹内的 `components` 文件夹中来开始编写这个规范：

```js
describe("InvestmentListItem", function() {

  beforeEach(function() {
    // render the React component
  });

it("should render", function() {
 expect(component.$el).toEqual('li.investment-list-item');
 });
});
```

将新文件添加到 `SpecRunner.html` 文件中，如前几章中已演示。

在规范中，我们基本上使用 `jasmine-jquery` 插件来期望我们的组件封装的 DOM 元素等于特定的 CSS 选择器。

我们如何将这个例子改为测试 React 组件？唯一的区别是获取 DOM 节点的 API。而不是使用带有 jQuery 对象的 `$element`，React 提供了一个名为 `getDOMNode()` 的函数，它返回它所声称的——一个 DOM 节点。

这样，我们可以使用之前相同的断言，并使测试准备就绪，如下所示：

```js
it("should render", function() {
  expect(component.getDOMNode()).toEqual('li.investment-list-item');
});
```

这很简单！所以，下一步是创建组件，渲染它，并将其附加到文档上。这也同样简单；请看以下代码片段：

```js
describe("InvestmentListItem", function() {
  var component;

  beforeEach(function() {

 setFixtures('<div id="application-container"></div>');
 var container = document.getElementById('application-container');

 var element = React.createElement(InvestmentListItem);
 component = React.render(element, container);
  });

  it("should render", function() {
    expect(component.getDOMNode()).toEqual('li.investment-list-item');
  });
});
```

这可能看起来像很多代码，但其中一半只是设置文档元素固定装置的样板代码，这样我们就可以在其中渲染 React 组件：

1.  首先，我们使用 `jasmine-jquery` 中的 `setFixtures` 函数在文档中创建一个具有 `application-container` ID 的元素。然后，使用 `getElementById` API 查询此元素并将其保存到 `container` 变量中。接下来的两个步骤是针对 React 的特定步骤：

    1.  首先，为了使用一个组件，我们必须首先从其类中创建一个元素；这是通过 `React.createElement` 函数完成的，如下所示：

        ```js
        var element = React.createElement(InvestmentListItem);
        ```

    1.  接下来，使用元素实例，我们最终可以通过 `React.render` 函数告诉 React 渲染它，如下所示：

        ```js
        component = React.render(element, container);
        ```

    1.  `render` 函数接受以下两个参数：

        +   React 元素

        +   一个用于渲染元素的 DOM 节点

1.  到目前为止，规范已经完成。你可以运行它并看到它失败，显示以下错误：

    ```js
    ReferenceError: InvestmentListItem is not defined.
    ```

1.  下一步是编写组件代码。所以，让我们填充规范，在 `src` 文件夹中创建一个新文件，命名为 `InvestmentListItem.js`，并将其添加到规范运行器中。这个文件应该遵循我们至今为止使用的模块模式。

1.  然后，使用 `React.createClass` 方法创建一个新的组件类：

    ```js
    (function (React) {
      var InvestmentListItem = React.createClass({

    render: function () {
          return React.createElement('li', { className: 'investment-list-item' }, 'Investment');
        }
      });

      this.InvestmentListItem = InvestmentListItem;
    })(React);
    ```

1.  至少，`React.createClass` 方法期望一个单独的 `render` 函数，该函数应该返回一个 React 元素的树。

1.  我们再次使用 `React.createElement` 方法来创建将成为渲染树根的元素，如下所示：

    ```js
    React.createElement('li', { className: 'investment-list-item' }, 'Investment')
    ```

与其在 `beforeEach` 块中之前的用法相比，这里的区别在于它还传递了一个包含 `className` 的 **props** 列表和一个包含文本 `Investment` 的单个子组件。

我们将更深入地探讨 props 参数的含义，但你可以将其视为与 HTML DOM 元素的属性类似。`className` prop 将转换为 `li` 元素的 class HTML 属性。

`React.createElement` 方法签名接受三个参数：

+   组件的类型，可以是表示真实 DOM 元素的字符串（如 `div`、`h1`、`p`）或 React 组件类

+   包含 props 值的对象

+   以及一个可变数量的子组件，在这个例子中，就是 `Investment` 字符串

在渲染这个组件（通过调用 `React.render()` 方法）时，结果将是：

```js
<li class="investment-list-item">Investment</li>
```

这是生成它的 JavaScript 代码的直接表示：

```js
React.createElement('li', { className: 'investment-list-item' }, 'Investment');
```

恭喜！你已经构建了第一个完全测试过的 React 组件。

# 虚拟 DOM

当你定义一个组件的渲染方法并调用 `React.createElement` 方法时，你实际上并没有在文档中渲染任何内容（你甚至没有创建 DOM 元素）。

只有通过 `React.render` 函数，调用这些 `React.createElement` 调用创建的表示才能有效地转换为真实的 DOM 元素并附加到文档上。

这种由 `ReactElements` 定义的表示，是 React 所称的虚拟 DOM。而且 `ReactElement` 不能与 DOM 元素混淆；它实际上是一个轻量级、无状态、不可变、虚拟的 DOM 元素表示。

所以为什么 React 要陷入创建新的 DOM 表示方式的麻烦呢？答案在这里是 *性能*。

随着浏览器的进化，JavaScript 的性能一直在不断提升，而今天的应用瓶颈实际上并不是 JavaScript。你可能听说过，你应该尽量减少对 DOM 的操作，React 允许你通过提供自己的 DOM 版本来实现这一点。

然而，这并不是唯一的原因。React 构建了一个非常强大的 diffing 算法，可以比较虚拟 DOM 的两种不同表示，计算它们之间的差异，并利用这些信息创建应用于真实 DOM 的突变。

它允许我们回到我们曾经使用的服务器端渲染的流程。基本上，在应用状态发生任何变化时，我们可以要求 React 重新渲染一切，然后它会计算所需的最小更改数量，并将这些更改应用到真实的 DOM 上。

它让我们开发者从担心修改 DOM 中解放出来，并赋予我们以声明式方式编写用户界面的能力，同时减少错误并提高生产力。

# JSX

如果您有编写前端 JavaScript 应用程序的经验，您可能熟悉一些模板语言。此时，您可能想知道您最喜欢的模板语言（如 Handlebars）可以在 React 中何处使用。答案是您不能。

React 不会在标记和逻辑之间做出区分；在 React 组件中，它们实际上是相同的。

然而，当我们开始构建更复杂的组件时会发生什么？我们在第三章中构建的表单，*测试前端代码*，将如何转换成 React 组件？

仅为了渲染而不涉及其他逻辑，需要调用一大堆 `React.createElement`，如下所示：

```js
var NewInvestment = React.createClass({
  render: function () {
    return React.createElement("form", {className: "new-investment"},
      React.createElement("h1", null, "New investment"),
      React.createElement("label", null,
        "Symbol:",
        React.createElement("input", {type: "text", className: "new-investment-stock-symbol", maxLength: "4"})
      ),
      React.createElement("label", null,
        "Shares:",
        React.createElement("input", {type: "number", className: "new-investment-shares"})
      ),
      React.createElement("label", null,
        "Share price:",
        React.createElement("input", {type: "number", className: "new-investment-share-price"})
      ),
      React.createElement("input", {type: "submit", className: "new-investment-submit", value: "Add"})
    );
  }
});
```

这非常冗长且难以阅读。因此，鉴于 React 组件既是标记也是逻辑，我们能否将其写成 HTML 和 JavaScript 的混合体会更好？下面是如何做的：

```js
var NewInvestment = React.createClass({
  render: function () {
    return <form className="new-investment">
      <h1>New investment</h1>
      <label>
        Symbol:
        <input type="text" className="new-investment-stock-symbol" maxLength="4" />
      </label>
      <label>
        Shares:
        <input type="number" className="new-investment-shares" />
      </label>
      <label>
        Share price:
        <input type="number" className="new-investment-share-price" />
      </label>
      <input type="submit" className="new-investment-submit" value="Add" />
    </form>;
  }
});
```

那就是 JSX，一种类似于 XML 的 JavaScript 语法扩展，它是为了与 React 一起使用而构建的。

它会转换成 JavaScript，所以根据后一个示例，它将直接编译成前面展示的纯 JavaScript 代码。

转换过程的一个重要特性是它不会改变行号；因此，JSX 中的*第 10 行*将转换成转换后的 JavaScript 文件中的*第 10 行*。这有助于在调试代码和进行静态代码分析时。

关于该语言的更多信息，您可以查看官方规范[`facebook.github.io/jsx/`](http://facebook.github.io/jsx/)，但就目前而言，我们可以按照下面的示例继续，当我们深入探讨该语言的功能时。

重要的是要知道，在实现 React 组件时使用 JSX 并不是强制要求，但它使整个过程变得更加容易。考虑到这一点，我们目前将继续使用它。

## 使用 JSX 与 Jasmine

为了让我们能够使用 JSX 与我们的 Jasmine 运行器一起使用，我们需要进行一些更改。

首先，我们需要将想要使用 JSX 语法的文件重命名为 `.jsx`。虽然这不是强制要求，但它使我们能够轻松地识别出哪些文件正在使用这种特殊语法。

接下来，在 `SpecRunner.html` 文件中，我们需要更改脚本标签，以表明这些不是常规的 JavaScript 文件，如下所示：

```js
<script src="img/strong>" type="text/jsx"></script>
<script src="img/strong>" type="text/jsx"></script>
```

不幸的是，我们需要的更改不止这些。浏览器不理解 JSX 语法，因此我们需要加载一个特殊的转换器，它将首先将这些文件转换成常规 JavaScript。

这个转换器包含在 React 入门套件中，所以只需在加载 React 之后立即加载它，如下所示：

```js
<script src="img/react-with-addons.js"></script>
<script src="img/JSXTransformer.js"></script>

```

在完成此设置后，我们应该能够运行测试，不是吗？不幸的是，还有一步。

如果您尝试在浏览器中打开 `SpecRunner.html` 文件，您将看到 `InvestmentListItem` 的测试没有被执行。这是因为转换器通过 AJAX 加载脚本文件，转换它们，最后将它们附加到 DOM 上。在此过程完成之前，Jasmine 已经运行了测试。

我们需要一种方式来通知 Jasmine 等待这些文件加载并转换。

最简单的方法是更改位于 `jasmine-2.1.3` 文件夹中的 `jasmine-2.1.3` 文件夹内的 `lib` 文件夹中的 Jasmine 的 `boot.js` 文件。

在原始文件中，您需要找到包含 `env.execute();` 方法的行并将其注释掉。它可能如下所示：

```js
window.onload = function() {
  if (currentWindowOnload) {
    currentWindowOnload();
  }
  htmlReporter.initialize();

// delays execution so that JSX files can be loaded
 // env.execute();
};
```

文件中的其他内容应保持不变。在此更改之后，您将看到测试不再运行——一个都没有。

唯一缺少的部分是在 JSX 文件加载后调用此 `execute` 方法。为此，我们将在 `jasmine.2.1.3` 文件夹中创建一个名为 `boot-exec.js` 的新文件，内容如下：

```js
/**
  Custom boot file that actually runs the tests.
  The code below was extracted and commented out from the original boot.js file.
 */
(function() {

  var env = jasmine.getEnv();
  env.execute();

}());
```

如您所见，它只执行了原始启动文件中先前注释的代码。

要运行这个自定义启动程序非常简单。我们将其添加到 `SpecRunner.html` 文件的 `<head>` 标签的最后一行，作为 JSX 类型：

```js

<!-- After all JSX files were loaded and processed, the tests can finally run -->
 <script src="img/boot-exec.js" type="text/jsx"></script>

</head>
```

`JSXTransformer` 库确保脚本按照声明的顺序加载。因此，当 `boot-exec.js` 文件被加载时，源文件和测试文件已经加载完毕。

有了这个，我们的测试运行器现在支持 JSX。

# 组件属性（props）

Props 是在 React 中从父组件传递数据到子组件的方式。

对于下一个示例，我们希望将 `InvestmentListItem` 组件修改为以百分比格式渲染 `roi` 变量的值。

为了实现下一个规范，我们将使用 React 通过 `React.addons.TestUtils` 对象提供的几个辅助方法，如下所示：

```js
describe("InvestmentListItem", function() {
  var TestUtils = React.addons.TestUtils;

  describe("given an Investment", function() {
    var investment, component;

    beforeEach(function() {
      investment = new Investment({
        stock: new Stock({ symbol: 'peto', sharePrice: 0.25 }),
        shares: 100,
        sharePrice: 0.20
      });

      component = TestUtils.renderIntoDocument(
        <InvestmentListItem investment={investment}/>
      );
    });

    it("should render the return of investment as a percentage", function() {
      var roi = TestUtils.findRenderedDOMComponentWithClass(component, 'roi');
      expect(roi.getDOMNode()).toHaveText('25%');
    });
  });
});
```

如您所见，我们不再使用来自 `jasmine-jquery` 匹配器的 `setFixture` 方法。相反，我们使用 `TestUtils` 模块来渲染组件。

这里的最大区别是 `TestUtils.renderIntoDocument` 并没有在文档中渲染，而是在一个分离的节点中渲染。

下一个您会注意到的是，`InvestmentListItem` 组件有一个属性（实际上称为 **prop**），我们通过它传递 `investment`。

然后，在规范中，我们使用另一个名为 `findRenderedDOMComponentWithClass` 的辅助方法来在 `component` 变量中查找 DOM 元素。

此方法返回 `ReactElement`。再次强调，我们将使用 `getDOMNode` 方法获取实际的 DOM 元素，然后使用 `jasmine-jquery` 匹配器检查其文本值，如下所示：

```js
var roi = TestUtils.findRenderedDOMComponentWithClass(component, 'roi');
expect(roi.getDOMNode()).toHaveText('25%');
```

在组件中实现此行为实际上非常简单：

```js
(function (React) {
  var InvestmentListItem = React.createClass({
    render: function () {
      var investment = this.props.investment;

      return <li className="investment-list-item">
        <article>
          <span className="roi">{formatPercentage(investment.roi())}</span>
        </article>
      </li>;
    }
  });

  function formatPercentage (number) {
    return (number * 100).toFixed(0) + '%';
  }

  this.InvestmentListItem = InvestmentListItem;
})(React);
```

我们可以通过 `this.props` 对象访问传递给组件的任何 props。

扩展原始实现，我们添加了一个具有预期类的 `span` 元素。

为了使投资回报率具有动态性，JSX 有一个特殊的语法。使用 `{}`，你可以在 XML 中调用任何 JavaScript 代码。我们在这里调用 `formatPercentage` 函数，并传递 `investment.roi()` 的值，如下所示：

```js
<span className="roi">{formatPercentage(investment.roi())}</span>
```

再次，只是为了使这一点更清晰，这个 JSX 转换为 JavaScript 会是：

```js
React.createElement("span", {className: "roi"}, formatPercentage(investment.roi()))
```

需要知道的是，一个 prop 应该是不可变的。组件改变其自身的 prop 值不是组件的责任。你可以将只有一个 props 的 React 组件视为一个纯函数，因为它总是返回相同的值，前提是给定的参数值相同。

这使得测试变得非常简单，因为没有突变或状态变化来测试组件。

# 组件事件

UI 应用程序有用户事件；在网页中，它们以 DOM 事件的形式出现。由于 React 将每个 DOM 元素包装成 React 元素，因此处理它们会有所不同，但仍然非常熟悉。

对于下一个示例，假设我们的应用程序将允许用户删除投资。我们可以通过以下验收标准来编写这个需求：

给定一个投资，当点击删除按钮时，InvestmentListItem 应该通知观察者 onClickDelete。

这里的想法与第三章“整合视图与观察者”部分中提到的相同，第三章，*测试前端代码*。

那么，我们如何在组件中设置观察者呢？正如我们之前所看到的，**props** 是将属性传递给我们的组件的方式，如下所示：

```js
describe("InvestmentListItem", function() {
  var TestUtils = React.addons.TestUtils;

  describe("given an Investment", function() {
    var investment, component, onClickDelete;

    beforeEach(function() {
      investment = new Investment({
        stock: new Stock({ symbol: 'peto', sharePrice: 0.25 }),
        shares: 100,
        sharePrice: 0.20
      });

      onClickDelete = jasmine.createSpy('onClickDelete');

      component = TestUtils.renderIntoDocument(
        <InvestmentListItem investment={investment} onClickDelete={onClickDelete}/>
      );
    });

    it("should notify an observer onClickDelete when the delete button is clicked", function() {
      var deleteButton = TestUtils.findRenderedDOMComponentWithTag(component, 'button');
      TestUtils.Simulate.click(deleteButton);
      expect(onClickDelete).toHaveBeenCalled();
    });

  });
});
```

正如你所见，我们向下传递了另一个 prop 给 `onClickDelete` 组件，并将其值设置为 Jasmine 间谍，如下所示：

```js
onClickDelete = jasmine.createSpy('onClickDelete');

component = TestUtils.renderIntoDocument(
  <InvestmentListItem investment={investment} onClickDelete={onClickDelete}
/>
);
```

**然后，我们通过其标签找到了删除按钮，并使用 `TestUtils` 模块模拟了一个点击，期望之前创建的间谍被调用，如下所示：**

```js
var deleteButton = TestUtils.findRenderedDOMComponentWithTag(component, 'button');
TestUtils.Simulate.click(deleteButton);
expect(onClickDelete).toHaveBeenCalled();
```

**`TestUtils.Simulate` 模块包含用于模拟所有类型 DOM 事件的辅助方法，如下所示：**

```js
TestUtils.Simulate.**click**(node);
TestUtils.Simulate.**change**(node, {target: {value: 'Hello, world'}});
TestUtils.Simulate.**keyDown**(node, {key: "Enter"});
```

**然后，我们回到了实现阶段：**

```js
(function (React) {
  var InvestmentListItem = React.createClass({
    render: function () {
      var investment = this.props.investment;
      **var onClickDelete = this.props.onClickDelete;**

      return <li className="investment-list-item">
        <article>
          <span className="roi">{formatPercentage(investment.roi())}</span>
          <button className="delete-investment" **onClick={onClickDelete}**>Delete</button>
        </article>
      </li>;
    }
  });

  function formatPercentage (number) {
    return (number * 100).toFixed(0) + '%';
  }

  this.InvestmentListItem = InvestmentListItem;
})(React);
```

**正如你所见，这就像嵌套另一个 `button` 组件并将 `onClickDelete` prop 值作为其 `onClick` prop 传递一样简单。**

**React 规范化事件，以便在不同浏览器中具有一致的属性，但其命名约定和语法与 HTML 中的内联 JavaScript 代码相似。要获取支持事件的完整列表，你可以查看官方文档[`facebook.github.io/react/docs/events.html`](http://facebook.github.io/react/docs/events.html)。**

**# 组件状态**

到目前为止，我们一直将 React 视为一个无状态的渲染引擎，但正如我们所知，应用程序有状态，尤其是在使用表单时。那么，我们如何实现 `NewInvestment` 组件，以便它能够保留正在创建的投资的值，并在用户完成表单后通知观察者？

为了帮助我们实现这种行为，我们将使用另一个组件内部 API——它的**状态**。

让我们看看以下验收标准：

假设`NewInvestment`组件的输入已经正确填写，当表单提交时，它应该通过投资属性通知`onCreate`观察者：

```js
describe("NewInvestment", function() {
  var TestUtils = React.addons.TestUtils;
  var component, onCreateSpy;

  function findNodeWithClass (className) {
    return TestUtils.findRenderedDOMComponentWithClass(component, className).getDOMNode();
  }

  beforeEach(function() {
    onCreateSpy = jasmine.createSpy('onCreateSpy');
    component = TestUtils.renderIntoDocument(
      <NewInvestment onCreate={onCreateSpy}/>
    );
  });

  describe("with its inputs correctly filled", function() {
    beforeEach(function() {
      var stockSymbol = findNodeWithClass('new-investment-stock-symbol');
      var shares = findNodeWithClass('new-investment-shares');
      var sharePrice = findNodeWithClass('new-investment-share-price');

      TestUtils.Simulate.change(stockSymbol, { target: { value: 'AOUE' }});
      TestUtils.Simulate.change(shares, { target: { value: '100' }});
      TestUtils.Simulate.change(sharePrice, { target: { value: '20' }});
    });

    describe("when its form is submitted", function() {
      beforeEach(function() {
        var form = component.getDOMNode();
        TestUtils.Simulate.submit(form);
      });

      it("should invoke the 'onCreate' callback with the investment attributes", function() {
        var investmentAttributes = { stockSymbol: 'AOUE', shares: '100', sharePrice: '20' };

        expect(onCreateSpy).toHaveBeenCalledWith(investmentAttributes);
      });
    });
  });
});
```

这个规范基本上使用了我们至今所学到的每一个技巧，所以不深入细节，让我们直接进入组件实现。

任何具有状态的组件必须首先声明其初始状态，通过定义一个`getInitialState`方法，如下所示：

```js
var NewInvestment = React.createClass({
 getInitialState: function () {
 return {
 stockSymbol: '',
 shares: 0,
 sharePrice: 0
 };

},

  render: function () {
 var state = this.state;

    return <form className="new-investment">
      <h1>New investment</h1>
      <label>
        Symbol:
        <input type="text" ref="stockSymbol" className="new-investment-stock-symbol" value={state.stockSymbol} maxLength="4"/>
      </label>
      <label>
        Shares:
        <input type="number" className="new-investment-shares" value={state.shares}/>
      </label>
      <label>
        Share price:
        <input type="number" className="new-investment-share-price" value={state.sharePrice}/>
      </label>
      <input type="submit" className="new-investment-submit" value="Add"/>
    </form>;
  }
});
```

如前述代码所示，我们明确定义了表单的初始状态，并在渲染方法中将状态作为`value`属性传递给输入组件。

如果你在一个浏览器中运行这个示例，你会注意到你无法更改输入的值。你可以聚焦到输入上，但尝试输入不会改变其值，这是因为 React 的工作方式。

与 HTML 不同，React 组件必须在任何时间点表示视图的状态，而不仅仅是初始化时间。如果我们想改变输入的值，我们需要监听输入的`onChange`事件，并使用这些信息更新状态。状态的改变将触发一个渲染，从而更新屏幕上的值。

为了演示这是如何工作的，让我们在`stockSymbol`输入处实现这种行为。

首先，我们需要更改渲染方法，向`onChange`事件添加一个处理程序：

```js
<input type="text" ref="stockSymbol" className="new-investment-stock-symbol" value={state.stockSymbol} maxLength="4" onChange={this._handleStockSymbolChange}/>
```

一旦事件被触发，它将调用`_handleStockSymbolChange`方法。其实现应该通过调用`this.setState`方法并传入输入的新值来更新状态，如下所示：

```js
var NewInvestment = React.createClass({
  getInitialState: function () {
    // ... Method implementation
  },

  render: function () {
    // ... Method implementation
  },

_handleStockSymbolChange: function (event) {
 this.setState({ stockSymbol: event.target.value });
 }
});
```

事件处理程序是一个在将输入数据传递到状态之前执行简单验证或转换的好地方。

正如你所见，这只是一大堆样板代码，只是为了处理单个输入。由于我们并没有在我们的事件处理程序中实现任何自定义行为，我们可以使用 React 的一个特殊功能，该功能为我们实现了这种“链接状态”。

我们将使用一个名为`Mixin`的`LinkedStateMixin`；但首先，什么是 Mixin？它是一种在组件之间共享常见功能的方法，在这种情况下，是“链接状态”。看看以下代码：

```js
var NewInvestment = React.createClass({

mixins: [React.addons.LinkedStateMixin],

  // ...

  render: function () {
    // ...
    <input type="text" ref="stockSymbol" className="new-investment-stock-symbol" valueLink={this.linkState('stockSymbol')} maxLength="4" />
    // ...
  }
});
```

`LinkedStateMixin`通过向组件添加`linkState`函数来实现，而不是设置输入的`value`，我们设置一个特殊的属性`valueLink`，该属性由`this.linkState`函数返回的链接对象设置。

`linkState`函数期望的是应该链接到输入值的状态属性的名称。

# 组件生命周期

如你可能已经注意到的，React 对组件的 API 有很强的观点。但它也对组件的生命周期有很强的观点，允许我们开发者添加钩子以创建自定义行为并在开发组件时执行清理任务。

这也是 React 最大的成功之一，因为正是通过这种标准化，我们可以通过组合创建更大更好的组件；通过这种方式，我们不仅可以使用自己的组件，还可以使用其他人的组件。

为了演示一个用例，我们将实现一个非常简单的行为：在页面加载时，我们希望新的投资表股票符号输入被聚焦，以便用户可以立即开始输入。

但是，在我们开始编写测试之前，我们只需要做一件事。如前所述，`TestUtils.renderIntoDocument` 并没有在文档中实际渲染任何内容，而是在一个分离的节点上。因此，如果我们用它来渲染我们的组件，我们就无法对输入的焦点进行断言。

因此，我们再次不得不使用 `setFixtures` 方法来在文档中实际渲染 React 组件，如下所示：

```js
/**
  Uses jasmine-jquery fixtures to actually render in the document.
  React.TestUtils.renderIntoDocument renders in a detached node.

  This was required to test the focus behavior.
 */
function actuallyRender (component) {
  setFixtures('<div id="application-container"></div>');
  var container = document.getElementById('application-container');
  return React.render(component, container);
}

describe("NewInvestment", function() {
  var TestUtils = React.addons.TestUtils;
  var component, stockSymbol;

  function findNodeWithClass (className) {
    return TestUtils.findRenderedDOMComponentWithClass(component, className).getDOMNode();
  }

  beforeEach(function() {
    component = actuallyRender(<NewInvestment onCreate={onCreateSpy}/>);
    stockSymbol = findNodeWithClass('new-investment-stock-symbol');
  });

  it("should have its stock symbol input on focus", function() {
    expect(stockSymbol).toBeFocused();
  });
});
```

完成这个小改动，并编写完规范后，我们可以回到实现上。

React 提供了一些钩子，我们可以在组件的生命周期中实现自定义代码；它们如下所示：

+   `componentWillMount`

+   `componentDidMount`

+   `componentWillReceiveProps`

+   `shouldComponentUpdate`

+   `componentWillUpdate`

+   `componentDidUpdate`

+   `componentWillUnmount`

为了实现我们的自定义行为，我们将使用仅在组件渲染并附加到 DOM 元素之后调用的 `componentDidMount` 钩子。

因此，我们想要做的是在这个钩子内部，以某种方式获取对输入 DOM 元素的访问权限并触发其焦点。我们已经知道如何获取 DOM 节点；它是通过 `getDOMNode` API。但是，我们如何获取输入的 React 元素？

React 为此问题提供的另一个特性称为 **ref**。它基本上是一种给组件的子元素命名的方法，以便以后可以访问。

由于我们想要股票符号输入，我们需要给它添加一个 `ref` 属性，如下所示：

```js
<input type="text" ref="stockSymbol" className="new-investment-stock-symbol" valueLink={this.linkState('stockSymbol')} maxLength="4" />
```

然后，在 `componentDidMount` 钩子中，我们可以通过其 `ref` 名称获取输入，然后获取其 DOM 元素并触发焦点，如下所示：

```js
var NewInvestment = React.createClass({
  // ...

componentDidMount: function () {
 this.refs.stockSymbol.getDOMNode().focus();
 }
,
  // ...
});
```

其他钩子以相同的方式设置，只需在类定义对象上定义它们作为属性。但每个钩子在不同的情况下被调用，有不同的规则。官方文档是关于它们定义和可能用例的绝佳资源，可以在 [`facebook.github.io/react/docs/component-specs.html#lifecycle-methods`](http://facebook.github.io/react/docs/component-specs.html#lifecycle-methods) 找到。

# 组合组件

我们已经就通过组合 React 的默认组件来创建组件的方式讨论了很多关于 **composability** 的内容。然而，我们还没有展示如何将自定义组件组合成更大的组件。

如你所猜，这应该是一个相当简单的练习，为了演示它是如何工作的，我们将实现一个组件来列出投资，如下所示：

```js
var InvestmentList = React.createClass({
  render: function () {
    var onClickDelete = this.props.onClickDelete;

    var listItems = this.props.investments.map(function (investment) {
      return <InvestmentListItem investment={investment}
                  onClickDelete={onClickDelete.bind(null, investment)}/>;
    });

    return <ul className="investment-list">{listItems}</ul>;
  }
});
```

这就像使用已可用的`InvestmentListItem`全局变量作为`InvestmentList`组件的根元素一样简单。

组件期望`investments`属性是一个投资数组。然后它通过为数组中的每个投资创建一个`InvestmentListItem`元素来映射它。

最后，它使用`listItems`数组作为`ul`元素的子元素，有效地定义了如何渲染投资列表。

# 摘要

React 是一个快速发展的库，正受到 JavaScript 社区的广泛关注；随着它不断改进丰富网络应用程序的开发，它引入了一些有趣的模式，并对一些根深蒂固的教条提出了质疑。

本章的目标不是深入探讨这个库，而是概述其主要特性和哲学。它展示了在用 React 编写界面时，可以进行测试驱动开发。

你了解了**属性**和**状态**以及它们之间的区别：**属性**不属于组件，如果需要，应由其父组件更改。**状态**是组件拥有的数据。它可以由组件更改，并且通过这样做，会触发新的渲染。

在你的应用程序中，具有状态的组件越少，就越容易对其进行分析和测试。

是通过 React 有见地的 API 和生命周期，我们可以获得最大限度的可组合性和代码重用优势。

当你开始使用 React 进行应用程序开发时，建议你了解 Facebook 推荐的架构 Flux，可以在[`facebook.github.io/flux/`](http://facebook.github.io/flux/)上找到。**
