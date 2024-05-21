# 第七章：测试 React 应用程序

作为 Web 开发人员，您熟悉今天构建大多数网站的方式。通常有一个 Web 服务器（使用 Java、Ruby 或 PHP 等语言），它处理用户请求并响应标记（HTML）。

这意味着在每个请求上，Web 服务器通过 URL 解释用户操作并呈现整个页面。

为了改善用户体验，越来越多的功能开始从服务器端推送到客户端，并且 JavaScript 不再仅仅是为页面添加行为，而是完全渲染页面。最大的优势是用户操作不再触发整个页面刷新；JavaScript 代码可以处理整个浏览器文档并相应地进行变异。

尽管这确实改善了用户体验，但它开始给应用程序代码增加了很多复杂性，导致维护成本增加，最糟糕的是——在屏幕不同部分之间存在不一致的错误形式。

为了使这种情况变得理智，建立了许多库和框架，但它们都失败了，因为它们没有解决整个问题的根本原因——可变性。

服务器端渲染很容易，因为没有变异要处理。给定一个新的应用程序状态，服务器将简单地重新渲染所有内容。如果我们能从这种方法中在客户端 JavaScript 代码中获益会怎样呢？

这正是**React**提出的。您可以通过组件声明性地编写接口代码，并告诉 React 进行渲染。在应用程序状态发生任何变化时，您可以简单地告诉 React 再次进行重新渲染；然后它将计算移动 DOM 到所需状态所需的变异，并为您应用它们。

在本章中，我们将通过将到目前为止构建的代码重构为 SPA 来了解 React 的工作原理。

# 项目设置

但是，在我们可以深入了解 React 之前，首先我们需要在我们的项目中进行一些小的设置，以便我们可以创建 React 组件。

转到[`facebook.github.io/react/downloads.html`](http://facebook.github.io/react/downloads.html)并下载 React Starter Kit 版本 0.12.2 或更高版本。

下载后，您可以解压其内容，并将构建文件夹中的所有文件移动到我们应用程序的 lib 文件夹中。然后，只需将 React 库加载到`SpecRunner.html`文件中。

```js
**<script src="lib/react-with-addons.js"></script>**
<script src="lib/jquery.js"></script>
```

设置完成后，我们可以继续编写我们的第一个组件。

# 我们的第一个 React 组件

正如本章的介绍所述，使用 React，您可以通过组件声明性地编写接口代码。

React 组件的概念类似于第三章中介绍的组件概念，因此可以期待看到一些相似之处。

有了这个想法，让我们创建我们的第一个组件。为了更好地理解 React 组件是什么，我们将使用一个非常简单的验收标准，并像往常一样从规范开始。

让我们实现"InvestmentListItem 应该呈现"。这很简单，不是真正*面向特性*，但是是一个很好的例子，让我们开始。

根据我们在第三章中学到的知识，我们可以通过创建一个名为`InvestmentListItemSpec.js`的新文件并将其保存在`spec`文件夹内的`components`文件夹中来开始编写这个规范：

```js
describe("InvestmentListItem", function() {

  beforeEach(function() {
    // render the React component
  });

**it("should render", function() {**
 **expect(component.$el).toEqual('li.investment-list-item');**
 **});**
});
```

将新文件添加到`SpecRunner.html`文件中，就像在之前的章节中已经演示的那样。

在规范中，我们基本上使用`jasmine-jquery`插件来期望我们组件的封装 DOM 元素等于特定的 CSS 选择器。

我们如何将这个示例更改为 React 组件的测试？唯一的区别是获取 DOM 节点的 API。React 暴露了一个名为`getDOMNode()`的函数，它返回它所声明的 DOM 节点。

有了这个，我们可以使用与之前相同的断言，并准备好我们的测试，如下所示：

```js
it("should render", function() {
  expect(component.**getDOMNode()**).toEqual('li.investment-list-item');
});
```

那很容易！所以下一步是创建组件，渲染它，并将其附加到文档中。这也很简单；看一下以下要点：

```js
describe("InvestmentListItem", function() {
  var component;

  beforeEach(function() {

 **setFixtures('<div id="application-container"></div>');**
 **var container = document.getElementById('application-container');**

 **var element = React.createElement(InvestmentListItem);**
 **component = React.render(element, container);**
  });

  it("should render", function() {
    expect(component.getDOMNode()).toEqual('li.investment-list-item');
  });
});
```

它可能看起来像是很多代码，但其中一半只是样板文件，用于设置我们可以在其中渲染 React 组件的文档元素装置：

1.  首先，我们使用`jasmine-jquery`中的`setFixtures`函数在文档中创建一个带有`application-container`ID 的元素。然后，使用`getElementById` API，我们查询此元素并将其保存在`container`变量中。接下来的两个步骤是特定于 React 的步骤：

1.  首先，为了使用组件，我们必须首先从其类创建一个元素；这是通过`React.createElement`函数完成的，如下所示：

```js
var element = **React.createElement**(InvestmentListItem);
```

1.  接下来，使用元素实例，我们最终可以通过`React.render`函数告诉 React 渲染它，如下所示：

```js
component = **React.render**(element, container);
```

1.  `render`函数接受以下两个参数：

+   React 元素

+   要渲染元素的 DOM 节点

1.  到目前为止，规范已经完成。您可以运行它并查看它失败，显示以下错误：

```js
ReferenceError: InvestmentListItem is not defined.
```

1.  下一步是编写组件。因此，让我们满足规范，在`src`文件夹中创建一个新文件，命名为`InvestmentListItem.js`，并将其添加到规范运行程序。此文件应遵循我们到目前为止一直在使用的模块模式。

1.  然后，使用`React.createClass`方法创建一个新的组件类：

```js
(function (React) {
  var InvestmentListItem = **React.createClass**({

**render**: function () {
      return React.createElement('li', { className: 'investment-list-item' }, 'Investment');
    }
  });

  this.InvestmentListItem = InvestmentListItem;
})(React);
```

1.  至少，`React.createClass`方法期望一个应该返回 React 元素树的`render`函数。

1.  我们再次使用`React.createElement`方法来创建将成为渲染树根的元素，如下所示：

```js
**React.createElement**('li', { className: 'investment-list-item' }, 'Investment')
```

与在`beforeEach`块中以前的用法的不同之处在于，这里还传递了一个**props**列表（带有`className`）和包含文本`Investment`的单个子元素。

我们将更深入地了解 props 参数的含义，但您可以将其视为类似于 HTML DOM 元素的属性。`className` prop 将变成`li`元素的 class HTML 属性。

`React.createElement`方法签名接受三个参数：

+   组件的类型可以是表示真实 DOM 元素的字符串（例如`div`，`h1`，`p`）或 React 组件类

+   包含 props 值的对象

+   以及可变数量的子组件，在这种情况下，只是`Investment`字符串

在渲染此组件（通过调用`React.render()`方法）时，结果将是：

```js
<li class="investment-list-item">Investment</li>
```

这是生成它的 JavaScript 代码的直接表示：

```js
React.createElement('li', { className: 'investment-list-item' }, 'Investment');
```

恭喜！您已经构建了您的第一个完全测试的 React 组件。

# 虚拟 DOM

当您定义组件的渲染方法并调用`React.createElement`方法时，您实际上并没有在文档中渲染任何内容（甚至没有创建 DOM 元素）。

只有通过调用这些`React.createElement`调用创建的表示才能有效地转换为真实的 DOM 元素并附加到文档中。

由`ReactElements`定义的这种表示是 React 称之为虚拟 DOM。`ReactElement`不应与 DOM 元素混淆；它实际上是 DOM 元素的轻量、无状态、不可变的虚拟表示。

那么为什么 React 要费力地创建一种新的表示 DOM 的方式呢？答案在于*性能*。

随着浏览器的发展，JavaScript 的性能不断提高，如今的应用程序瓶颈实际上并不是 JavaScript。您可能听说过应该尽量少地触及 DOM，而 React 允许您通过让您与其自己的 DOM 版本交互来做到这一点。

然而，这并不是唯一的原因。React 构建了一个非常强大的差异算法，可以比较虚拟 DOM 的两个不同表示，计算它们的差异，并根据这些信息创建变化，然后应用于真实 DOM。

它使我们能够回到以前在服务器端渲染中使用的流程。基本上，我们可以在应用程序状态的任何更改时要求 React 重新渲染所有内容，然后它将计算所需的最小更改数量，并仅将其应用于真实 DOM。

它使我们开发人员不必担心改变 DOM，并赋予我们以声明方式编写用户界面的能力，同时减少错误并提高生产力。

# JSX

如果您有编写前端 JavaScript 应用程序的经验，您可能熟悉一些模板语言。此时，您可能想知道在哪里可以使用您喜欢的模板语言（如 Handlebars）与 React 一起使用。答案是不能。

React 不会区分标记和逻辑；在 React 组件中，它们实际上是相同的。

然而，当我们开始创建更复杂的组件时会发生什么？我们在第三章中构建的表单会如何转换为 React 组件？

要仅呈现它而没有其他逻辑，需要进行一系列的`React.createElement`调用，如下所示：

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

这非常冗长且难以阅读。因此，考虑到 React 组件既是标记又是逻辑，如果我们能够将其编写为 HTML 和 JavaScript 的混合，那不是更好吗？下面是方法：

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

这就是**JSX**，一种看起来像 XML 的 JavaScript 语法扩展，专为与 React 一起使用而构建。

它会转换为 JavaScript，因此，根据后面的示例，它将直接编译为之前呈现的普通 JavaScript 代码。

转换过程的一个重要特性是它不会改变行号；因此，在 JSX 中的*第 10 行*将转换为转换后的 JavaScript 文件中的*第 10 行*。这有助于调试代码和进行静态代码分析。

有关该语言的更多信息，您可以在[`facebook.github.io/jsx/`](http://facebook.github.io/jsx/)上查看官方规范，但现在，您可以随着我们深入了解该语言的特性，跟随下面的示例。 

重要的是要知道，在实现 React 组件时并不要求使用 JSX，但它会让这个过程变得更容易。考虑到这一点，我们暂时会继续使用它。

## 使用 JSX 与 Jasmine

为了让我们能够在 Jasmine 运行器中使用 JSX，我们需要做一些更改。

首先，我们需要将要使用 JSX 语法的文件重命名为`.jsx`。虽然这不是必需的，但它可以让我们轻松地识别出文件是否使用了这种特殊语法。

接下来，在`SpecRunner.html`文件中，我们需要更改脚本标签，以指示这些不是常规的 JavaScript 文件，如下所示：

```js
<script src="src/components/InvestmentListItem**.jsx**" **type="text/jsx"**></script>
<script src="spec/components/InvestmentListItemSpec**.jsx**" **type="text/jsx"**></script>
```

不幸的是，这些不是我们需要做的唯一更改。浏览器无法理解 JSX 语法，因此我们需要加载一个特殊的转换器，首先将这些文件转换为常规的 JavaScript。

这个转换器已经捆绑在 React 起始套件中，所以只需在加载 React 后立即加载它，如下所示：

```js
<script src="lib/react-with-addons.js"></script>
**<script src="lib/JSXTransformer.js"></script>**

```

完成此设置后，我们应该能够运行测试，不是吗？不幸的是，还有一步。

如果您尝试在浏览器中打开`SpecRunner.html`文件，您会发现`InvestmentListItem`的测试没有被执行。这是因为转换器通过 AJAX 加载脚本文件，对其进行转换，最后将其附加到 DOM。在此过程完成时，Jasmine 已经运行了测试。

我们需要一种方法来告诉 Jasmine 等待这些文件加载和转换。

最简单的方法是更改`jasmine-2.1.3`文件夹中`lib`文件夹内的`jasmine`的`boot.js`文件。

在原始文件中，你需要找到包含`env.execute();`方法的行并将其注释掉。它应该类似于以下代码：

```js
window.onload = function() {
  if (currentWindowOnload) {
    currentWindowOnload();
  }
  htmlReporter.initialize();

**// delays execution so that JSX files can be loaded**
 **// env.execute();**
};
```

文件中的其他内容应该保持不变。在这个更改之后，你会发现测试不再运行——一个都没有。

唯一缺失的部分是一旦加载了 JSX 文件就调用这个`execute`方法。为此，我们将在`jasmine.2.1.3`文件夹中创建一个名为`boot-exec.js`的新文件，内容如下：

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

正如你所看到的，它只是执行原始引导文件中以前注释的代码。

运行这个自定义引导非常简单。我们将它作为 JSX 类型添加到`SpecRunner.html`的`<head>`标签的最后一行：

```js

**<!-- After all JSX files were loaded and processed, the tests can finally run -->**
 **<script src="lib/jasmine-2.1.3/boot-exec.js" type="text/jsx"></script>**

</head>
```

`JSXTransformer`库保证脚本按声明的顺序加载。因此，当`boot-exec.js`文件加载时，源文件和测试文件已经加载完毕。

有了这个，我们的测试运行器现在支持 JSX 了。

# 组件属性（props）

Props 是在 React 中从父组件传递数据到子组件的方式。

对于下一个示例，我们想要更改`InvestmentListItem`组件，以便以百分比格式呈现`roi`变量的值。

为了实现下一个规范，我们将使用 React 通过`React.addons.TestUtils`对象提供的一些辅助方法，如下所示：

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

      component = **TestUtils.renderIntoDocument**(
        <InvestmentListItem investment={investment}/>
      );
    });

    it("should render the return of investment as a percentage", function() {
      var roi = **TestUtils.findRenderedDOMComponentWithClass**(component, 'roi');
      expect(roi.getDOMNode()).toHaveText('25%');
    });
  });
});
```

如你所见，我们不再使用`jasmine-jquery`匹配器中的`setFixture`方法。相反，我们使用`TestUtils`模块来渲染组件。

这里最大的区别是`TestUtils.renderIntoDocument`实际上并没有在文档中渲染，而是渲染到一个分离的节点中。

你将注意到的下一件事是`InvestmentListItem`组件有一个属性（实际上称为**prop**），我们通过它传递`investment`。

然后，在规范中，我们使用另一个名为`findRenderedDOMComponentWithClass`的辅助方法来查找`component`变量中的 DOM 元素。

这个方法返回`ReactElement`。然后，我们将使用`getDOMNode`方法获取实际的 DOM 元素，然后使用`jasmine-jquery`匹配器来检查其文本值，如下所示：

```js
var roi = TestUtils.findRenderedDOMComponentWithClass(component, 'roi');
expect(roi.getDOMNode()).**toHaveText**('25%');
```

在组件中实现这种行为实际上非常简单：

```js
(function (React) {
  var InvestmentListItem = React.createClass({
    render: function () {
      var investment = **this.props.investment**;

      return <li className="investment-list-item">
        <article>
          <span className="roi">**{formatPercentage(investment.roi())}**</span>
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

我们可以通过`this.props`对象访问传递给组件的任何 props。

扩展原始实现，我们添加了一个带有规范中预期类的`span`元素。

为了使投资回报率动态化，JSX 有一种特殊的语法。使用`{}`，你可以在 XML 中间调用任何 JavaScript 代码。我们在传递`investment.roi()`值时调用`formatPercentage`函数，如下所示：

```js
<span className="roi">{formatPercentage(investment.roi())}</span>
```

再次强调一下，这个 JSX 转换成 JavaScript 将是：

```js
React.createElement("span", {className: "roi"}, formatPercentage(investment.roi()))
```

重要的是要知道，prop 应该是不可变的。改变自己的 prop 值不是组件的责任。你可以将只有 props 的 React 组件视为纯函数，因为它总是在给定相同参数值的情况下返回相同的结果值。

这使得测试非常简单，因为没有变异或更改状态来测试组件。

# 组件事件

UI 应用程序有用户事件；在 Web 中，它们以 DOM 事件的形式出现。由于 React 将每个 DOM 元素包装成 React 元素，处理它们会有一点不同，但非常熟悉。

对于下一个示例，假设我们的应用程序允许用户删除一个投资。我们可以通过以下验收标准来表达这个要求：

给定一个投资，当单击删除按钮时，InvestmentListItem 应该通知观察者 onClickDelete。

这里的想法与第三章中的*将视图与观察者集成*部分中提出的想法是一样的，*测试前端代码*。

那么，我们应该如何在组件中设置观察者？正如我们之前已经看到的，**props**是将属性传递给我们的组件的方式，如下所示：

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
        <InvestmentListItem investment={investment} **onClickDelete={onClickDelete}**/>
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

正如你所看到的，我们将另一个 prop 传递给`onClickDelete`组件，并将其值设置为 Jasmine spy，如下所示：

```js
**onClickDelete = jasmine.createSpy('onClickDelete');**

component = TestUtils.renderIntoDocument(
  <InvestmentListItem investment={investment} **onClickDelete={onClickDelete}**
/>
);
```

然后，我们通过其标签找到了删除按钮，并使用`TestUtils`模块模拟了一个点击，期望之前创建的间谍被调用，如下所示：

```js
var deleteButton = TestUtils.findRenderedDOMComponentWithTag(component, 'button');
TestUtils.Simulate.click(deleteButton);
expect(onClickDelete).toHaveBeenCalled();
```

`TestUtils.Simulate`模块包含了模拟所有类型的 DOM 事件的辅助方法，如下所示：

```js
TestUtils.Simulate.**click**(node);
TestUtils.Simulate.**change**(node, {target: {value: 'Hello, world'}});
TestUtils.Simulate.**keyDown**(node, {key: "Enter"});
```

然后，我们回到了实现：

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

正如你所看到的，它就像嵌套另一个`button`组件并将`onClickDelete`属性值作为其`onClick`属性传递一样简单。

React 标准化事件，以便它们在不同浏览器中具有一致的属性，但其命名约定和语法类似于 HTML 中的内联 JavaScript 代码。要获取支持的事件的全面列表，可以在官方文档中查看[`facebook.github.io/react/docs/events.html`](http://facebook.github.io/react/docs/events.html)。

#组件状态

到目前为止，我们已经将 React 视为一个无状态的渲染引擎，但是我们知道，应用程序有状态，特别是在使用表单时。那么，我们应该如何实现`NewInvestment`组件，以便它保持正在创建的投资的值，然后在用户完成表单后通知观察者？

为了帮助我们实现这种行为，我们将使用另一个组件内部 API——它的**state**。

让我们看一下以下验收标准：

鉴于`NewInvestment`组件的输入已正确填写，当提交表单时，它应该使用投资属性通知`onCreate`观察者：

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

这个规范基本上使用了我们到目前为止学到的所有技巧，所以不要深入细节，让我们直接进入组件实现。

任何具有状态的组件必须声明的第一件事是通过定义`getInitialState`方法来定义其初始状态，如下所示：

```js
var NewInvestment = React.createClass({
 **getInitialState: function () {**
 **return {**
 **stockSymbol: '',**
 **shares: 0,**
 **sharePrice: 0**
 **};**

**},**

  render: function () {
 **var state = this.state;**

    return <form className="new-investment">
      <h1>New investment</h1>
      <label>
        Symbol:
        <input type="text" ref="stockSymbol" className="new-investment-stock-symbol" **value={state.stockSymbol}** maxLength="4"/>
      </label>
      <label>
        Shares:
        <input type="number" className="new-investment-shares" **value={state.shares}**/>
      </label>
      <label>
        Share price:
        <input type="number" className="new-investment-share-price" **value={state.sharePrice}**/>
      </label>
      <input type="submit" className="new-investment-submit" value="Add"/>
    </form>;
  }
});
```

正如前面的代码所示，我们清楚地定义了表单的初始状态，并在渲染方法中将状态作为`value`属性传递给输入组件。

如果您在浏览器中运行此示例，您会注意到您无法更改输入的值。您可以聚焦在输入上，但尝试输入不会更改其值，这是因为 React 的工作方式。

与 HTML 不同，React 组件必须在任何时间点表示视图的状态，而不仅仅是在初始化时。如果我们想要更改输入的值，我们需要监听输入的`onChange`事件，并根据该信息更新状态。状态的更改将触发渲染，从而更新屏幕上的值。

为了演示这是如何工作的，让我们在`stockSymbol`输入中实现这种行为。

首先，我们需要更改渲染方法，为`onChange`事件添加一个处理程序：

```js
<input type="text" ref="stockSymbol" className="new-investment-stock-symbol" value={state.stockSymbol} maxLength="4" **onChange={this._handleStockSymbolChange}**/>
```

一旦触发事件，它将调用`_handleStockSymbolChange`方法。它的实现应该通过调用`this.setState`方法来更新状态，新的输入值如下所示：

```js
var NewInvestment = React.createClass({
  getInitialState: function () {
    // ... Method implementation
  },

  render: function () {
    // ... Method implementation
  },

**_handleStockSymbolChange: function (event) {**
 **this.setState({ stockSymbol: event.target.value });**
 **}**
});
```

事件处理程序是在将输入数据传递给状态之前执行简单验证或转换的好地方。

正如你所看到的，这是大量样板代码，只是为了处理单个输入。由于我们没有在事件处理程序中实现任何自定义行为，我们可以使用特殊的 React 功能来为我们实现这个“链接状态”。

我们将使用一个名为`LinkedStateMixin`的**Mixin**；但首先，什么是 Mixin？它是在组件之间共享常见功能的一种方式，这种情况下是“链接状态”。看一下以下代码：

```js
var NewInvestment = React.createClass({

**mixins: [React.addons.LinkedStateMixin],**

  // ...

  render: function () {
    // ...
    <input type="text" ref="stockSymbol" className="new-investment-stock-symbol" **valueLink={this.linkState('stockSymbol')}** maxLength="4" />
    // ...
  }
});
```

`LinkedStateMixin`通过向组件添加`linkState`函数工作，而不是设置输入的`value`，我们使用由函数`this.linkState`返回的链接对象设置一个名为`valueLink`的特殊属性。

`linkState`函数期望**state**的属性名称，它应该将其链接到输入的值。

# 组件生命周期

你可能已经注意到，React 对组件的 API 有自己的看法。但它对组件的生命周期也有非常强烈的看法，允许我们开发人员添加钩子来创建自定义行为并在开发组件时执行清理任务。

这是 React 的最大胜利之一，因为通过这种标准化，我们可以通过组合创建更大更好的组件；通过这样，我们不仅可以使用我们自己的组件，还可以使用其他人的组件。

为了演示一个用例，我们将实现一个非常简单的行为：在页面加载时，我们希望新的投资表单股票符号输入获得焦点，以便用户可以立即开始输入。

但是，在我们开始编写测试之前，有一件事情我们需要做。如前所述，`TestUtils.renderIntoDocument`实际上并不在文档中呈现任何内容，而是在一个分离的节点上呈现。因此，如果我们使用它来呈现我们的组件，我们将无法对输入的焦点进行断言。

因此，我们必须再次使用`setFixtures`方法来实际在文档中呈现 React 组件，如下所示：

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

完成了这个小改变，并编写了规范，我们可以回到实现中。

React 提供了一些钩子，我们可以在组件的生命周期中实现自定义代码；它们如下：

+   `componentWillMount`

+   `componentDidMount`

+   `componentWillReceiveProps`

+   `shouldComponentUpdate`

+   `componentWillUpdate`

+   `componentDidUpdate`

+   `componentWillUnmount`

为了实现我们的自定义行为，我们将使用`componentDidMount`钩子，该钩子仅在组件被呈现并附加到 DOM 元素后调用一次。

因此，我们想要在这个钩子内部以某种方式访问输入 DOM 元素并触发其焦点。我们已经知道如何获取 DOM 节点；通过`getDOMNode` API。但是，我们如何获取输入的 React 元素呢？

React 针对这个问题的另一个特性称为**ref**。基本上，它是一种为组件的子元素命名的方法，以允许以后访问。

由于我们想要股票符号输入，我们需要向其添加一个`ref`属性，如下所示：

```js
<input type="text" **ref="stockSymbol"** className="new-investment-stock-symbol" valueLink={this.linkState('stockSymbol')} maxLength="4" />
```

然后，在`componentDidMount`钩子中，我们可以通过其`ref`名称获取输入，然后获取其 DOM 元素并触发焦点，如下所示：

```js
var NewInvestment = React.createClass({
  // ...

**componentDidMount: function () {**
 **this.refs.stockSymbol.getDOMNode().focus();**
 **}**
,
  // ...
});
```

其他钩子以相同的方式设置，只需在类定义对象上定义它们作为属性。但是每个钩子在不同的场合被调用，并且有不同的规则。官方文档是关于它们定义和可能用例的很好资源，可以在[`facebook.github.io/react/docs/component-specs.html#lifecycle-methods`](http://facebook.github.io/react/docs/component-specs.html#lifecycle-methods)找到。

# 组合组件

我们已经谈了很多关于通过组合 React 的默认组件来创建组件的**可组合性**。然而，我们还没有展示如何将自定义组件组合到更大的组件中。

你可能已经猜到，这应该是一个非常简单的练习，为了演示这个工作原理，我们将实现一个列出投资的组件，如下所示：

```js
var InvestmentList = React.createClass({
  render: function () {
    var onClickDelete = this.props.onClickDelete;

    var listItems = this.props.investments.map(function (investment) {
      return <**InvestmentListItem** investment={investment}
                  onClickDelete={onClickDelete.bind(null, investment)}/>;
    });

    return <ul className="investment-list">{listItems}</ul>;
  }
});
```

只需使用已经可用的`InvestmentListItem`全局变量作为`InvestmentList`组件的根元素即可。

该组件期望`investments`属性是一个投资数组。然后，它通过为数组中的每个投资创建一个`InvestmentListItem`元素来映射它。

最后，它使用`listItems`数组作为`ul`元素的子元素，有效地定义了如何呈现投资列表。

# 摘要

React 是一个快速发展的库，受到 JavaScript 社区的广泛关注；它引入了一些有趣的模式，并质疑了一些既定的教条，不断改进了丰富的 Web 应用程序的开发。

本章的目标不是深入了解这个库，而是概述其主要特性和理念。它证明了在使用 React 编写界面时可以进行测试驱动开发。

你学到了**prop**和**state**以及它们的区别：**prop**不是组件所拥有的，如果需要，应该由其父组件进行更改。**state**是组件拥有的数据。它可以被组件更改，这样就会触发新的渲染。

在你的应用程序中，拥有状态的组件越少，就越容易理解和测试。

通过 React 的有主见的 API 和生命周期，我们可以最大程度地实现组合性和代码重用的好处。

当你开始使用 React 进行应用程序开发时，建议你了解 Flux，这是 Facebook 推荐的构建应用程序的架构，网址是[`facebook.github.io/flux/`](http://facebook.github.io/flux/)。
