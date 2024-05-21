# 第四章。异步测试 - AJAX

不可避免地，每个 JavaScript 应用程序都会有一个时刻，需要测试异步代码。

异步意味着您无法以线性方式处理它——一个函数可能在执行后立即返回，但结果通常会在稍后通过回调返回。

这在处理 AJAX 请求时是一种非常常见的模式，例如通过 jQuery：

```js
$.ajax('http://localhost/data.json', {
  success: function (data) {
    // handle the result
  }
});
```

在本章中，我们将学习 Jasmine 允许我们以不同方式编写异步代码的测试。

# 验收标准

为了演示 Jasmine 对异步测试的支持，我们将实现以下验收标准：

获取股票时，应更新其股价

使用我们到目前为止向您展示的技术，您可以在`spec`文件夹中的`StockSpec.js`文件中编写这个验收标准，如下所示：

```js
describe("Stock", function() {
  var stock;
  var originalSharePrice = 0;

  beforeEach(function() {
    stock = new Stock({
      symbol: 'AOUE',
      sharePrice: originalSharePrice
    });
  });

  it("should have a share price", function() {
    expect(stock.sharePrice).toEqual(originalSharePrice);
  });

  **describe("when fetched", function() {**
 **var fetched = false;**
 **beforeEach(function() {**
 **stock.fetch();**
 **});**

 **it("should update its share price", function() {**
 **expect(stock.sharePrice).toEqual(20.18);**
 **});**
 **});**
});
```

这将导致在`src`文件夹中的`Stock.js`文件中实现`fetch`函数，如下所示：

```js
Stock.prototype.**fetch** = function() {
  var that = this;
  var url = 'http://localhost:8000/stocks/'+that.symbol;

  **$.getJSON**(url, function (data) {
    that.sharePrice = data.sharePrice;
  });
};
```

在前面的代码中，重要的部分是`$.getJSON`调用，这是一个期望包含更新后的股价的 JSON 响应的 AJAX 请求，例如：

```js
{
  "sharePrice": 20.18
}
```

到目前为止，您可以看到我们被卡住了；为了运行这个规范，我们需要一个运行的服务器。

# 设置场景

由于本书都是关于 JavaScript 的，我们将创建一个非常简单的**Node.js**服务器供规范使用。Node.js 是一个允许使用 JavaScript 开发网络应用程序（如 Web 服务器）的平台。

在第六章*轻量级单元测试*中，我们将看到测试 AJAX 请求的替代解决方案，而无需服务器。在第八章*构建自动化*中，我们将看到如何使用 Node.js 作为高级构建系统的基础。

## 安装 Node.js

如果您已经安装了 Node.js，可以跳转到下一节。

Windows 和 Mac OS X 都有安装程序。执行以下步骤安装 Node.js：

1.  转到 Node.js 网站[`nodejs.org/`](http://nodejs.org/)。

1.  点击**安装**按钮。

1.  下载完成后，执行安装程序并按照步骤进行操作。

要检查其他安装方法以及如何在 Linux 发行版上安装 Node.js 的说明，请查看官方文档[`github.com/joyent/node/wiki/Installing-Node.js-via-package-manager`](https://github.com/joyent/node/wiki/Installing-Node.js-via-package-manager)。

完成后，您应该在命令行上有`node`和`npm`命令可用。

## 编写服务器

为了学习如何编写异步规范，我们将创建一个返回一些假数据的服务器。在项目的根文件夹中创建一个名为`server.js`的新文件，并将以下代码添加到其中：

```js
var express = require('express');
var app = express();

app.get('/stocks/:symbol', function (req, res) {
  res.setHeader('Content-Type', 'application/json');
  res.send({ sharePrice: 20.18 });
});

app.use(express.static(__dirname));

app.listen(8000);
```

为了处理 HTTP 请求，我们使用**Express**，一个 Node.js Web 应用程序框架。通过阅读代码，您可以看到它定义了一个到`/stocks/:symbol`的路由，因此它接受诸如`http://localhost:8000/stocks/AOUE`的请求，并用 JSON 数据做出响应。

我们还使用`express.static`模块在`http://localhost:8000/SpecRunner.html`上提供规范运行器。

有一个要求来规避 SOP。这是一个出于安全原因规定的政策，即不允许在与应用程序不同的域上执行 AJAX 请求。

在第三章*测试前端代码*中首次演示了使用 HTML 固定装置时出现的问题。

使用 Chrome 浏览器检查器，您可以看到在使用`file://`协议打开`SpecRunner.html`文件时控制台中的错误（基本上是您到目前为止一直在做的方式）：

![编写服务器](img/B04138_04_01.jpg)

这显示了同源策略错误

通过为运行器提供相同的基本 URL 下的所有应用程序和测试代码，我们可以防止出现这个问题，并能够在任何浏览器上运行规范。

## 运行服务器

要运行服务器，首先需要使用 Node 的包管理器安装其依赖项（Express）。在应用程序根文件夹中，运行`npm`命令：

```js
**$ npm install express**

```

这个命令将下载 Express 并将其放在项目文件夹内的一个名为`node_modules`的新文件夹中。

现在，您应该能够通过调用以下`node`命令来运行服务器：

```js
**$ node server.js**

```

要检查它是否起作用，请在浏览器上访问`http://localhost:8000/stocks/AOUE`，您应该会收到 JSON 响应：

```js
{"sharePrice": "20.18"}
```

现在我们的服务器依赖项正在运行，我们可以继续编写规范。

# 编写规范

在服务器运行时，打开浏览器访问`http://localhost:8000/SpecRunner.html`，以查看我们规范的结果。

您可以看到，即使服务器正在运行，并且规范似乎是正确的，但它仍然失败了。这是因为`stock.fetch()`是异步的。对`stock.fetch()`的调用会立即返回，允许 Jasmine 在 AJAX 请求完成之前运行期望：

```js
it("should update its share price", function() {
  expect(stock.sharePrice).toEqual(20.18);
});
```

为了解决这个问题，我们需要接受`stock.fetch()`函数的异步性，并指示 Jasmine 在运行期望之前等待其执行。

## 异步设置和拆卸

在所示的示例中，我们在规范的设置（`beforeEach`函数）期间调用`fetch`函数。

我们唯一需要做的是在其函数定义中添加一个`done`参数，以识别这个设置步骤是异步的：

```js
describe("when fetched", function() {
  beforeEach(function(**done**) {

  });

  it("should update its share price", function() {
    expect(stock.sharePrice).toEqual(20.18);
  });
});
```

一旦 Jasmine 识别到这个`done`参数，它会将一个必须在异步操作完成后调用的函数作为其值传递。

因此，我们可以将这个`done`函数作为`fetch`函数的`success`回调传递：

```js
beforeEach(function(done) {
  stock.fetch(**{**
 **success: done**
 **}**);
});
```

在实现时，在 AJAX 操作完成后调用它：

```js
Stock.prototype.fetch = function(params) {
  params = params || {};
  var that = this;
  **var success = params.success || function () {};**
 **var url = 'http://localhost:8000/stocks/'+that.symbol;**

  $.getJSON(url, function (data) {
    that.sharePrice = data.sharePrice;
 **success(that);**
  });
};
```

就是这样；Jasmine 将等待 AJAX 操作完成，测试将通过。

在需要时，还可以使用相同的`done`参数定义异步的`afterEach`。

## 异步规范

另一种方法是使用异步规范而不是异步设置。为了演示这将如何工作，我们需要重新编写我们之前的验收标准：

```js
describe("Stock", function() {
  var stock;
  var originalSharePrice = 0;

  beforeEach(function() {
    stock = new Stock({
      symbol: 'AOUE',
      sharePrice: originalSharePrice
    });
  });

  it("should be able to update its share price", function(done) {
    stock.fetch();
    expect(stock.sharePrice).toEqual(20.18);
  });
});
```

再次，我们只需要在其函数定义中添加一个`done`参数，并在测试完成后调用`done`函数：

```js
it("should be able to update its share price", function(**done**) {
  stock.fetch({
    success: function () {
      expect(stock.sharePrice).toEqual(20.18);
      **done();**
    }
  });
});
```

这里的区别在于，我们必须将期望移到`success`回调中，在调用`done`函数之前。

## 超时

在编写异步规范时，默认情况下，Jasmine 将等待 5 秒钟，等待`done`回调被调用，如果在此超时之前未调用，则规范将失败。

在这个假设的例子中，服务器是一个返回静态数据的简单存根，超时不是问题，但有些情况下，默认时间不足以完成异步任务。

虽然不建议有长时间运行的规范，但知道可以通过更改 Jasmine 中称为`jasmine.DEFAULT_TIMEOUT_INTERVAL`的简单配置变量来避免这种默认行为是很好的。

要使其在整个套件中生效，可以在`SpecHelper.js`文件中设置它，如下所示：

```js
beforeEach(function() {
  **jasmine.DEFAULT_TIMEOUT_INTERVAL = 10000;**

  jasmine.addMatchers({
    // matchers code
  });
});

jasmine.getFixtures().fixturesPath = 'spec/fixtures';
```

要使其在单个规范中生效，请在`beforeEach`中更改其值，并在`afterEach`期间恢复：

```js
describe("Stock", function() {
 **var defaultTimeout;**

  beforeEach(function() {
 **defaultTimeout = jasmine.DEFAULT_TIMEOUT_INTERVAL;**
 **jasmine.DEFAULT_TIMEOUT_INTERVAL = 10000;**
  });

  afterEach(function() {
 **jasmine.DEFAULT_TIMEOUT_INTERVAL = defaultTimeout;**
  });

  it("should be able to update its share price", function(done) {

  });
});
```

# 总结

在本章中，您已经看到了如何测试异步代码，这在测试服务器交互（AJAX）时很常见。

我还向您介绍了 Node.js 平台，并使用它编写了一个简单的服务器，用作测试装置。

在第六章*轻量级单元测试*中，我们将看到不需要运行服务器的 AJAX 测试的不同解决方案。

在下一章中，我们将学习间谍以及如何利用它们来进行行为检查。
