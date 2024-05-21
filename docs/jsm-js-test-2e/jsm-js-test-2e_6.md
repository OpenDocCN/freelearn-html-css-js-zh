# 第六章：光速单元测试

在第四章中，*异步测试 - AJAX*，我们看到了如何在应用程序中包含 AJAX 测试会增加测试的复杂性。在该章节的示例中，我们创建了一个结果可预测的服务器。基本上是一个复杂的测试装置。即使我们可以使用真实的服务器实现，它也会增加测试的复杂性；尝试从浏览器更改具有数据库或第三方服务的服务器的状态并不是一种容易或可扩展的解决方案。

还有对生产力的影响；这些请求需要时间来处理和传输，这会影响通常提供的快速反馈循环。

您还可以说这些规范测试了客户端和服务器代码，因此不能被视为单元测试；相反，它们可以被视为集成测试。

解决所有这些问题的一个方法是使用**存根**或**虚假**来代替代码的真实依赖关系。因此，我们不是向服务器发出请求，而是在浏览器内部使用服务器的测试替身。

我们将使用第四章中的相同示例，*异步测试 - AJAX*，并使用不同的技术进行重写。

# Jasmine 存根

我们已经看到了 Jasmine 间谍的一些用例。记住，间谍是一个特殊的函数，记录了它的调用方式。你可以把存根看作是带有行为的间谍。

我们在想要在规范中强制执行特定路径或替换真实实现为更简单实现时使用存根。

让我们通过使用 Jasmine 存根来重新审视接受标准的示例，“获取股票时，应更新其股价”。

我们知道股票的`fetch`函数是使用`$.getJSON`函数实现的，如下所示：

```js
Stock.prototype.fetch = function(parameters) {
  **$.getJSON**(url, function (data) {
    that.sharePrice = data.sharePrice;
    success(that);
  });
};
```

我们可以使用`spyOn`函数来设置对`getJSON`函数的间谍，代码如下：

```js
describe("when fetched", function() {
  beforeEach(function() {
    **spyOn($, 'getJSON').and.callFake(function(url, callback) {**
      **callback({ sharePrice: 20.18 });**
    **});**
    stock.fetch();
  });

  it("should update its share price", function() {
    expect(stock.sharePrice).toEqual(20.18);
  });
});
```

但这一次，我们将使用`and.callFake`函数为我们的间谍设置行为（默认情况下，间谍什么也不做，返回 undefined）。我们让间谍使用其`callback`参数调用一个对象响应（`{ sharePrice: 20.18 }`）。

随后，在期望中，我们使用`toEqual`断言来验证股票的`sharePrice`是否已更改。

要运行此规范，您不再需要服务器来进行请求，这是一件好事，但这种方法存在一个问题。如果`fetch`函数被重构为使用`$.ajax`而不是`$.getJSON`，那么测试将失败。一个更好的解决方案，由 Jasmine 插件**jasmine-ajax**提供，是代替浏览器的 AJAX 基础设施，因此 AJAX 请求的实现可以以不同的方式进行。

# Jasmine Ajax

Jasmine Ajax 是一个官方插件，旨在帮助测试 AJAX 请求。它将浏览器的 AJAX 请求基础设施更改为虚假实现。

这个虚假（或模拟）实现，虽然更简单，但对于使用其 API 的任何代码来说，仍然表现得像真实的实现一样。

## 安装插件

在深入规范实现之前，首先需要将插件添加到项目中。转到[`github.com/jasmine/jasmine-ajax/`](https://github.com/jasmine/jasmine-ajax/)并下载当前版本（应与 Jasmine 2.x 版本兼容）。将其放在`lib`文件夹中。

它还需要添加到`SpecRunner.html`文件中，所以继续添加另一个脚本：

```js
<script type="text/javascript" src="lib/mock-ajax.js"></script>
```

## 一个虚假的 XMLHttpRequest

每当你使用 jQuery 进行 AJAX 请求时，在幕后实际上是使用`XMLHttpRequest`对象来执行请求。

`XMLHttpRequest`是标准的 JavaScript HTTP API。尽管它的名称暗示它使用 XML，但它支持其他类型的内容，比如 JSON；出于兼容性原因，名称保持不变。

因此，我们可以改变`XMLHttpRequest`对象的假实现，而不是存根 jQuery。这正是这个插件所做的。

让我们重写先前的规范以使用这个虚假实现：

```js
describe("when fetched", function() {
  **beforeEach(function() {**
 **jasmine.Ajax.install();**
 **});**

  beforeEach(function() {
    stock.fetch();

    **jasmine.Ajax.requests.mostRecent().respondWith({**
 **'status': 200,**
 **'contentType': 'application/json',**
 **'responseText': '{ "sharePrice": 20.18 }'**
 **});**
  });

  **afterEach(function() {**
 **jasmine.Ajax.uninstall();**
 **});**

  it("should update its share price", function() {
    expect(stock.sharePrice).toEqual(20.18);
  });
});
```

深入实施：

1.  首先，我们告诉插件使用`jasmine.Ajax.install`函数将`XMLHttpRequest`对象的原始实现替换为假实现。

1.  然后调用`stock.fetch`函数，该函数将调用`$.getJSON`，在幕后创建新的`XMLHttpRequest`。

1.  最后，我们使用`jasmine.Ajax.requests.mostRecent().respondWith`函数来获取最近发出的请求，并用假响应对其进行响应。

我们使用`respondWith`函数，该函数接受一个具有三个属性的对象：

1.  `status`属性用于定义 HTTP 状态码。

1.  `contentType`（在示例中为 JSON）属性。

1.  `responseText`属性，其中包含请求的响应主体的文本字符串。

然后，一切都是运行期望的问题：

```js
it("should update its share price", function() {
  expect(stock.sharePrice).toEqual(20.18);
});
```

由于插件更改了全局的`XMLHttpRequest`对象，您必须记住告诉 Jasmine 在测试运行后将其恢复到原始实现；否则，您可能会干扰其他规范（例如 Jasmine jQuery fixtures 模块）中的代码。以下是您可以实现这一点的方法：

```js
afterEach(function() {
  jasmine.Ajax.uninstall();
});
```

还有一种略有不同的方法来编写这个规范；在这里，首先对请求进行存根（带有响应细节），然后执行要执行的代码。

先前的示例更改为以下内容：

```js
beforeEach(function() {
  **jasmine.Ajax.stubRequest('http://localhost:8000/stocks/AOUE').andReturn({**
 **'status': 200,**
 **'contentType': 'application/json',**
 **'responseText': '{ "sharePrice": 20.18 }'**
 **});**

  stock.fetch();
});
```

可以使用`jasmine.Ajax.stubRequest`函数来存根对特定请求的任何请求。在示例中，它由 URL `http://localhost:8000/stocks/AOUE`定义，并且响应定义如下：

```js
{
  'status': 200,
  'contentType': 'application/json',
  'responseText': '{ "sharePrice": 20.18 }'
}
```

响应定义遵循与先前使用的`respondWith`函数相同的属性。

# 摘要

在本章中，您了解了异步测试如何影响您可以通过单元测试获得的快速反馈循环。我展示了如何使用存根或假实现使您的规范更快地运行并减少依赖关系。

我们已经看到了两种不同的方式，您可以使用简单的 Jasmine 存根或更高级的`XMLHttpRequest`的假实现来测试 AJAX 请求。

您还更加熟悉间谍和存根，并应该更加舒适地在不同场景中使用它们。

在下一章中，我们将进一步探讨我们应用程序的复杂性，并进行全面重构，将其转换为一个完全功能的单页面应用程序，使用`React.js`库。
