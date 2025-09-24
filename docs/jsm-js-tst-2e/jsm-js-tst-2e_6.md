# 第六章. 光速单元测试

在 第四章 中，我们看到了如何将 AJAX 测试包含在应用程序中，这会增加测试的复杂性。在该章节的示例中，我们创建了一个结果可预测的服务器。它基本上是一个复杂的测试工具。即使我们可以使用真实的服务器实现，它也会增加测试的复杂性；尝试从浏览器更改具有数据库或第三方服务的服务器的状态——这不是一个简单或可扩展的解决方案。

这也对生产力有影响；这些请求需要时间来处理和传输，这损害了单元测试通常提供的快速反馈循环。

你也可以说，这些测试用例同时测试了客户端和服务器代码，因此不能被视为单元测试；相反，它们可以被视为集成测试。

解决所有这些问题的方法是在代码的真实依赖项中使用 **模拟** 或 **存根**。因此，而不是向服务器发出请求，我们在浏览器中使用服务器的测试替身。

我们将使用与 第四章 相同的示例，*异步测试 – AJAX*，并使用不同的技术重写它。

# Jasmine 模拟

我们已经看到了一些 Jasmine 监视器的用例。记住，监视器是一个特殊的函数，它记录了它的调用方式。你可以将模拟视为具有行为的监视器。

当我们想要在我们的测试用例中强制执行特定的路径或用一个更简单的实现替换真实实现时，我们会使用模拟。

让我们回顾一下验收标准的示例，“当股票被检索时，应更新其股票价格”，通过使用 Jasmine 模拟重写它。

我们知道股票的 `fetch` 函数是使用 `$.getJSON` 函数实现的，如下所示：

```js
Stock.prototype.fetch = function(parameters) {
  $.getJSON(url, function (data) {
    that.sharePrice = data.sharePrice;
    success(that);
  });
};
```

我们可以使用 `spyOn` 函数通过以下代码设置对 `getJSON` 函数的监视：

```js
describe("when fetched", function() {
  beforeEach(function() {
    spyOn($, 'getJSON').and.callFake(function(url, callback) {
      callback({ sharePrice: 20.18 });
    });
    stock.fetch();
  });

  it("should update its share price", function() {
    expect(stock.sharePrice).toEqual(20.18);
  });
});
```

但这次，我们将使用 `and.callFake` 函数为我们的监视器设置一个行为（默认情况下，监视器什么都不做并返回 `undefined`）。我们让监视器调用其 `callback` 参数，传递一个对象响应（`{ sharePrice: 20.18 }`）。

在后续的期望中，我们使用 `toEqual` 断言来验证股票的 `sharePrice` 是否已更改。

要运行这个测试用例，你不再需要服务器来处理请求，这是一个好事，但这种方法存在一个问题。如果 `fetch` 函数被重构为使用 `$.ajax` 而不是 `$.getJSON`，那么测试将会失败。一个更好的解决方案是由一个名为 **jasmine-ajax** 的 Jasmine 插件提供的，即模拟浏览器的 AJAX 基础设施，这样 AJAX 请求的实现就可以自由地以不同的方式完成。

# Jasmine Ajax

Jasmine Ajax 是一个官方插件，旨在帮助测试 AJAX 请求。它将浏览器的 AJAX 请求基础设施更改为模拟实现。

这个模拟（或模拟）实现，虽然更简单，但仍然对使用其 API 的任何代码表现得像真实实现。

## 安装插件

在我们深入研究规范实现之前，首先我们需要将插件添加到项目中。访问 [`github.com/jasmine/jasmine-ajax/`](https://github.com/jasmine/jasmine-ajax/) 并下载当前版本（应与 Jasmine 2.x 版本兼容）。将其放置在 `lib` 文件夹中。

它还需要添加到 `SpecRunner.html` 文件中，所以请继续添加另一个脚本：

```js
<script type="text/javascript" src="img/mock-ajax.js"></script>
```

## 模拟的 XMLHttpRequest

无论何时使用 jQuery 进行 AJAX 请求，实际上底层都是使用 `XMLHttpRequest` 对象来执行请求。

`XMLHttpRequest` 是标准的 JavaScript HTTP API。尽管其名称暗示它使用 XML，但它支持其他类型的内容，如 JSON；出于兼容性原因，名称保持不变。

因此，我们不是模拟 jQuery，而是可以用模拟实现更改 `XMLHttpRequest` 对象。这正是此插件所做的。

让我们重写之前的规范以使用这个模拟实现：

```js
describe("when fetched", function() {
  beforeEach(function() {
 jasmine.Ajax.install();
 });

  beforeEach(function() {
    stock.fetch();

    jasmine.Ajax.requests.mostRecent().respondWith({
 'status': 200,
 'contentType': 'application/json',
 'responseText': '{ "sharePrice": 20.18 }'
 });
  });

  afterEach(function() {
 jasmine.Ajax.uninstall();
 });

  it("should update its share price", function() {
    expect(stock.sharePrice).toEqual(20.18);
  });
});
```

深入实现：

1.  首先，我们告诉插件使用 `jasmine.Ajax.install` 函数用模拟实现替换 `XMLHttpRequest` 对象的原实现。

1.  然后，我们调用 `stock.fetch` 函数，这将调用 `$.getJSON`，在底层重新创建 `XMLHttpRequest`。

1.  最后，我们使用 `jasmine.Ajax.requests.mostRecent().respondWith` 函数获取最近发出的请求，并用模拟响应来响应它。

我们使用 `respondWith` 函数，它接受一个具有三个属性的对象：

1.  `status` 属性用于定义 HTTP 状态码。

1.  `contentType`（示例中为 JSON）属性。

1.  `responseText` 属性，它是一个包含请求响应体的文本字符串。

然后，所有的事情都是关于运行预期：

```js
it("should update its share price", function() {
  expect(stock.sharePrice).toEqual(20.18);
});
```

由于插件更改了全局的 `XMLHttpRequest` 对象，你必须记得在测试运行后告诉 Jasmine 恢复其原始实现；否则，可能会干扰其他规范（如 Jasmine jQuery 固件模块）中的代码。以下是完成此操作的方法：

```js
afterEach(function() {
  jasmine.Ajax.uninstall();
});
```

写这个规范还有稍微不同的方法；在这里，请求首先被模拟（带有响应详情），然后稍后执行要测试的代码。

之前的示例已更改为以下内容：

```js
beforeEach(function() {
  jasmine.Ajax.stubRequest('http://localhost:8000/stocks/AOUE').andReturn({
 'status': 200,
 'contentType': 'application/json',
 'responseText': '{ "sharePrice": 20.18 }'
 });

  stock.fetch();
});
```

可以使用 `jasmine.Ajax.stubRequest` 函数模拟对特定请求的任何请求。在示例中，它由 URL `http://localhost:8000/stocks/AOUE` 定义，响应定义如下：

```js
{
  'status': 200,
  'contentType': 'application/json',
  'responseText': '{ "sharePrice": 20.18 }'
}
```

响应定义遵循之前使用的 `respondWith` 函数相同的属性。

# 摘要

在本章中，你学习了异步测试如何损害你通过单元测试获得的快速反馈循环。我展示了你可以如何使用存根（stubs）或模拟（fakes）来使你的规范（specs）运行更快，并且依赖性更少。

我们已经看到了两种不同的方式，你可以使用简单的 Jasmine 存根和更高级的 `XMLHttpRequest` 模拟来实现 AJAX 请求的测试。

你也对间谍（spies）和存根（stubs）有了更多的了解，应该更习惯于在不同场景中使用它们。

在下一章中，我们将进一步探讨我们应用程序的复杂性，并将进行整体重构，将其转换为一个功能齐全的单页应用程序，使用 `React.js` 库。
