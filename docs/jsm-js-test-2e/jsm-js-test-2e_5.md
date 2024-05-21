# 第五章：Jasmine 间谍

测试替身是单元测试的一种模式。它用一个等效的实现替换测试依赖组件，该实现特定于测试场景。这些实现被称为**替身**，因为尽管它们的行为可能特定于测试，但它们的行为就像并且具有与其模拟的对象相同的 API。

间谍是 Jasmine 对测试替身的解决方案。在其核心，Jasmine 的**spy**是一种特殊类型的函数，记录与其发生的所有交互。因此，当返回值或对象状态的变化不能用于确定测试期望是否成功时，它们非常有用。换句话说，当测试成功只能通过**行为检查**来确定时，Jasmine 间谍是完美的。

# “裸”间谍

为了理解行为检查的概念，让我们重新访问第三章中提出的示例，*测试前端代码*，并测试`NewInvestmentView`测试套件的可观察行为：

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
        callbackSpy = **jasmine.createSpy('callback')**;
        view.onCreate(callbackSpy);

        investment = view.create();
      });

      it("should invoke the 'onCreate' callback with the created investment", function() {

 **expect(callbackSpy).toHaveBeenCalled();**
 **expect(callbackSpy).toHaveBeenCalledWith(investment);**
      });
    });
  });
});
```

在规范设置期间，使用`jasmine.createSpy`函数创建一个新的 Jasmine 间谍，并为其传递一个名称（`callback`）。Jasmine 间谍是一种特殊类型的函数，用于跟踪对其进行的调用和参数。

然后，它将这个间谍设置为 View 的 create 事件的观察者，使用`onCreate`函数，最后调用`create`函数创建一个新的投资。

随后，在期望中，规范使用`toHaveBeenCalled`和`toHaveBeenCalledWith`匹配器来检查`callbackSpy`是否被调用，并且使用了正确的参数（`investment`），从而进行行为检查。

# 对对象的函数进行间谍活动

一个间谍本身非常有用，但它的真正力量在于使用对应的间谍来更改对象的原始实现。

考虑以下示例，旨在验证当提交表单时，必须调用`view`的`create`函数：

```js
describe("NewInvestmentView", function() {
  var view;

  // setup and other specs ...

  describe("with its inputs correctly filled", function() {

    // setup and other specs ...

    describe("and when the form is submitted", function() {
      beforeEach(function() {
        **spyOn(view, 'create');**
        view.$element.submit();
      });

      it("should create an investment", function() {
        **expect(view.create).toHaveBeenCalled();**
      });
    });
  });
});
```

在这里，我们利用全局 Jasmine 函数`spyOn`来使用间谍更改`view`的`create`函数。

然后，在规范中稍后，我们使用`toHaveBeenCalled` Jasmine 匹配器来验证`view.create`函数是否被调用。

规范完成后，Jasmine 会恢复对象的原始行为。

# 测试 DOM 事件

在编写前端应用程序时，DOM 事件经常被使用，有时我们打算编写一个规范，检查事件是否被触发。

事件可能是表单提交或输入已更改之类的东西，那么我们如何使用间谍来做到这一点呢？

我们可以向`NewInvestmentView`测试套件添加一个新的验收标准，以检查在单击添加按钮时是否提交了其表单：

```js
describe("and when its add button is clicked", function() {
  beforeEach(function() {
    **spyOnEvent(view.$element, 'submit');**
    view.20.18.find('input[type=submit]').click();
  });

  it("should submit the form", function() {
    **expect('submit').toHaveBeenTriggeredOn(view.20.18);**
  });
});
```

为了编写这个规范，我们使用 Jasmine jQuery 插件提供的`spyOnEvent`全局函数。

它通过接受`view.20.18`，这是一个 DOM 元素，以及我们想要监视的`submit`事件来工作。然后，稍后，我们使用 jasmine jQuery 匹配器`toHaveBeenTriggeredOn`来检查事件是否在元素上触发。

# 总结

在本章中，您了解了测试替身的概念以及如何使用间谍来执行规范上的行为检查。

在下一章中，我们将看看如何使用伪造和存根来替换我们规范的真实依赖项，并加快其执行速度。
