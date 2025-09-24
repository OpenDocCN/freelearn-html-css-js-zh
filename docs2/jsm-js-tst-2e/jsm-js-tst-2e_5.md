# 第五章。Jasmine 间谍

测试替身是单元测试中的一种模式。它用一个特定于测试场景的等效实现替换了测试依赖的组件。这些实现被称为**替身**，因为尽管它们的行为可能特定于测试，但它们的行为和 API 与它们模仿的对象相同。

间谍是 Jasmine 测试替身的一种解决方案。在其核心，一个 Jasmine **间谍**是一种特殊的函数，它记录了与它发生的所有交互。因此，当返回值或对象状态的改变无法用来确定测试期望是否成功时，它们非常有用。换句话说，Jasmine 间谍在测试成功只能通过**行为检查**来确定时非常完美。

# “裸”间谍

要理解行为检查的概念，让我们回顾一下在第三章中提出的示例，*测试前端代码*，并测试`NewInvestmentView`测试套件的观察行为：

```js
 describe("NewInvestmentView", function() {
  var view;

  // setup and other specs ...

  describe("with its inputs correctly filled", function() {

    // setup and other specs ...

    describe("and when an investment is created", function() {
      var callbackSpy;
      var investment;

      beforeEach(function() {
        callbackSpy = jasmine.createSpy('callback');
        view.onCreate(callbackSpy);

        investment = view.create();
      });

      it("should invoke the 'onCreate' callback with the created investment", function() {

 expect(callbackSpy).toHaveBeenCalled();
 expect(callbackSpy).toHaveBeenCalledWith(investment);
      });
    });
  });
});
```

在规范设置期间，它使用`jasmine.createSpy`函数创建一个新的 Jasmine 间谍，并为其传递一个名称（`callback`）。Jasmine 间谍是一种特殊的函数，它跟踪对其的调用和参数。

然后，它使用`onCreate`函数将这个间谍设置为视图创建事件的观察者，并最终调用`create`函数来创建一个新的投资。

在后续的期望中，规范使用`toHaveBeenCalled`和`toHaveBeenCalledWith`匹配器来检查`callbackSpy`是否被调用，并且带有正确的参数（`investment`），从而进行行为检查。

# 监视对象的函数

间谍本身非常有用，但它的真正力量在于使用对应间谍改变一个对象的原生实现。

考虑以下示例，目的是验证当表单提交时，`view`的`create`函数必须被调用：

```js
describe("NewInvestmentView", function() {
  var view;

  // setup and other specs ...

  describe("with its inputs correctly filled", function() {

    // setup and other specs ...

    describe("and when the form is submitted", function() {
      beforeEach(function() {
        spyOn(view, 'create');
        view.$element.submit();
      });

      it("should create an investment", function() {
        expect(view.create).toHaveBeenCalled();
      });
    });
  });
});
```

在这里，我们使用全局 Jasmine 函数`spyOn`来用间谍替换`view`的`create`函数。

然后，在规范中稍后，我们使用`toHaveBeenCalled` Jasmine 匹配器来验证`view.create`函数是否被调用。

在规范完成后，Jasmine 恢复对象的原始行为。

# 测试 DOM 事件

在编写前端应用程序时，DOM 事件被广泛使用，有时我们打算编写一个规范来检查事件是否被触发。

事件可能像表单提交或已更改的输入一样，那么我们如何使用间谍来完成这个任务呢？

我们可以向`NewInvestmentView`测试套件添加一个新的验收标准，以检查当我们点击添加按钮时，其表单是否正在提交：

```js
describe("and when its add button is clicked", function() {
  beforeEach(function() {
    spyOnEvent(view.$element, 'submit');
    view.20.18.find('input[type=submit]').click();
  });

  it("should submit the form", function() {
    expect('submit').toHaveBeenTriggeredOn(view.20.18);
  });
});
```

要编写这个规范，我们使用 Jasmine jQuery 插件提供的全局函数`spyOnEvent`。

它通过接受`view.20.18`，这是一个 DOM 元素，以及我们想要间谍监视的`submit`事件来实现。然后，稍后我们使用 Jasmine jQuery 匹配器`toHaveBeenTriggeredOn`来检查事件是否在元素上触发。

# 摘要

在本章中，您已经接触到了测试替身的概念以及如何使用间谍来对您的规格进行行为检查。

在下一章中，我们将探讨如何使用模拟和存根来替换我们规格的真实依赖项，并加快它们的执行速度。
