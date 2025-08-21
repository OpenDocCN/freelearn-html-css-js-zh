# 第四章：理解 Mocha

在上一章中，我们安装并介绍了 Mocha。Mocha 是一个 JavaScript 测试框架，可以在 Node.js 内部或浏览器内部运行。你可以使用它来定义和运行自己的测试。Mocha 会报告测试的结果：哪些测试运行正常，哪些测试失败以及失败发生在哪里。Mocha 依次运行每个测试，等待一个测试完成或超时后再运行下一个。

尽管 Mocha 设计为能够在任何现代浏览器上运行，但我们将仅通过 Node.js 通过命令行来运行它。Mocha 还有其他功能，这将在本章中解释。有关 Mocha 功能的更完整参考，请访问 Mocha 的官方文档网站，[`visionmedia.github.com/mocha/`](http://visionmedia.github.com/mocha/) 了解更多信息。

本章涵盖的主题包括：

+   描述功能并使用断言

+   理解 Mocha 如何执行异步测试

通过本章结束时，你应该能够使用 Mocha 执行异步测试，并理解 Mocha 如何控制测试流程。

# 组织你的测试

有两种策略可以用来组织你的测试。第一种是以某种方式将它们分成单独的文件，每个文件代表应用程序的一个功能或逻辑单元。另一种策略是，可以与第一种策略一起使用，即按功能进行分组。

为应用程序的每个功能单元单独创建一个文件是分离测试关注点的好方法。你应该分析应用程序的结构，并将其分成具有最小重叠量的不同关注点。例如，你的应用程序可能需要处理用户注册 - 这可能是一个功能组。另一个功能组可能是用户登录。如果你的应用程序涉及待办事项列表，你可能希望有一个单独的文件包含该部分的测试。

通过为每个功能组单独创建文件，你可以在处理特定组时独立调用你的测试。这种技术还允许你保持每个文件的行数较低，这在导航和维护测试时很有帮助。

**描述功能**：在定义测试时，你还可以按功能对应用程序功能进行分组。例如，在描述待办事项列表功能时，你可以进一步将这些功能分开如下：

+   创建待办事项

+   删除待办事项

+   显示待办事项列表

+   更改待办事项列表项目的顺序

在我们的测试脚本中，我们将描述先前提到的可测试的待办事项功能。

待办事项测试文件的布局可以如下：

```js
describe('To-do items', function() {

  describe('creating', function() {
    // to-do item creating tests here...
  });

  describe('removing', function() {
    // removing a to-do item tests here...
  });

  describe('showing', function() {
    // to-do item list showing tests here...
  });

  describe('ordering', function() {
    // to-do item ordering tests here...
  });

});
```

你可以嵌套任意多个`describe`语句，尽可能细化测试的范围，但作为一个经验法则，你应该使用两个描述级别：一个用于功能组（例如，待办事项），另一个级别用于每个功能。在每个功能定义内，你可以放置所有相关的测试。

# 使用 before 和 after 钩子

对于任何一组测试，你可以设置某些代码在所有测试之前或之后运行。这对于设置数据库、清理一些状态或一般设置或拆除一些你需要以便运行测试本身的状态非常有用。

在下一个示例中，名为`runBefore`的函数在任何描述的测试之前运行：

```js
describe('some feature', function() {

 before(function runBefore() {
    console.log('running before function...');  });

  it('should do A', function() {
    console.log('test A');
  });

  it('should do B', function() {
    console.log('test B');
  });
});
```

将此文件代码保存为名为`test.js`的文件，并在本地安装 Mocha：

```js
$ npm install mocha
```

运行测试：

```js
$ node_modules/.bin/mocha test.js
```

它应该给出以下输出：

```js
  running before function...
test A
.test B
.

  ✔ 2 tests complete (6ms)
```

类似地，你还可以指定一个函数，在所有测试完成后执行：

```js
describe('some feature', function() {

  after(function runAfter() {
    console.log('running after function...');  });

  it('should do A', function() {
    console.log('test A');
  });

  it('should do B', function() {
    console.log('test B');
  });
});
```

运行此代码会产生以下输出，正如你所期望的那样：

```js
  test A
.test B
.running after function...

  ✔ 2 tests complete (6ms)
```

还可以定义一个函数，在每个测试块之前（或之后）调用，分别使用`beforeEach`和`afterEach`关键字。`beforeEach`关键字的示例用法如下：

```js
describe('some feature', function() {

  beforeEach(function runBeforeEach() {
    console.log('running beforeEach function...');  });

  it('should do A', function() {
    console.log('test A');
  });

  it('should do B', function() {
    console.log('test B');
  });
});
```

如果运行此测试，输出将为：

```js
  running beforeEach function...
test A
.running beforeEach function...
test B
.

  ✔ 2 tests complete (6ms)
```

当然，`afterEach`代码在每次测试执行后调用该函数。

# 使用异步钩子

在任何测试之前运行的这些函数都可以是异步的。如果一个函数是异步的，只需接受一个回调参数，就像这样：

```js
describe('some feature', function() {
  function runBeforeEach(done) {
    console.log('running afterEach function...');
    setTimeout(done, 1000);
  }
  beforeEach(runBeforeEach);

  it('should do A', function() {
    console.log('test A');
  });

  it('should do B', function() {
    console.log('test B');
  });
});
```

运行此测试代码时，您会注意到每次测试运行前有一秒的延迟，如果没有提供回调参数，这一点是不会被观察到的。

## 钩子如何与测试组交互

正如我们所见，在描述范围内，您可以有相应的`before`、`after`、`beforeEach`和`afterEach`钩子。如果您有一个嵌套的`describe`范围，该范围也可以有钩子。除了当前范围上的钩子之外，Mocha 还将调用所有父范围上的钩子。考虑一下这段代码，我们在其中声明了一个两级嵌套：

```js
describe('feature A', function() {

  before(function() {
    console.log('before A');
  });

  after(function() {
    console.log('after A');
  });

  beforeEach(function() {
    console.log('beforeEach A');
  });

  afterEach(function() {
    console.log('afterEach A');
  });

  describe('feature A.1', function() {
    before(function() {
      console.log('before A.1');
    });

    after(function() {
      console.log('after A.1');
    });

    beforeEach(function() {
      console.log('beforeEach A.1');
    });

    afterEach(function() {
      console.log('afterEach A.1');
    });

    it('should do A.1.1', function() {
      console.log('A.1.1');
    });

    it('should do A.1.2', function() {
      console.log('A.1.2');
    });

  });

});
```

运行上述代码时，输出为：

```js
  before A
before A.1
beforeEach A
beforeEach A.1
A.1.1
.afterEach A.1
afterEach A
beforeEach A
beforeEach A.1
A.1.2
.afterEach A.1
afterEach A
after A.1
after A

  ✔ 4 tests complete (16ms)
```

# 使用断言

现在您有一个用于测试代码的地方，您需要一种验证代码是否按预期运行的方法。为此，您需要一个断言测试库。

有许多断言测试库适用于许多编程风格，但在这里，我们将使用 Node.js 已经捆绑的一个，即`assert`模块。它包含了您需要描述每个测试的期望的最小一组实用函数。在每个测试文件的顶部，您需要使用`require`引入断言库：

```js
var assert = require('assert');
```

### 注意

您可以断言任何表达式的“真实性”。“真实”和“虚假”是 JavaScript（以及其他语言）中的概念，其中类型强制转换允许某些值等同于布尔值 true 或 false。一些例子如下：

```js
var a = true;
assert.ok(a, 'a should be truthy');
```

虚假值为：

+   `false`

+   `null`

+   `undefined`

+   空字符串

+   `0`（数字零）

+   `NaN`

所有其他值都为真。

您还可以使用`assert.equal`来进行相等的测试：

```js
var a = 'ABC';
assert.equal(a, 'ABC');
```

您还可以使用`assert.notEqual`来进行不相等的测试：

```js
var a = 'ABC';
assert.notEqual(a, 'ABCDEF');
```

这最后两个测试等同于 JavaScript 的`==`（宽松相等）运算符，这意味着它们适用于布尔值、字符串、`undefined`和`null`，但不适用于对象和数组。例如，这个断言将失败：

```js
assert.equal({a:1}, {a:1});
```

它将失败，因为在 JavaScript 中，没有本地方法来比较两个对象的等价性，从而使以下表达式为假：

```js
{a: 1} == {a:1}
```

要比较对象（包括数组），您应该使用`assert.deepEqual`：

```js
assert.deepEqual({a:1}, {a:1});
assert.deepEqual([0,1], [0,1]);
```

这个函数递归地比较对象，找出它们是否有某种不同。这个函数也可以用于深度嵌套的对象，正如其名称所暗示的那样：

```js
assert.deepEqual({a:[0,1], b: {c:2}}, {a:[0,1], b: {c:2}});
```

您还可以测试深层不相等：

```js
assert.notDeepEqual({a:[0,1], b: {c:2}}, {a:[0,1], b: {c:2, d: 3}});
```

## 更改断言消息

当断言失败时，将抛出一个包含消息的错误，其中打印了预期值和实际值：

```js
> var a = false;
> assert.ok(a)
AssertionError: false == true
    at repl:1:9
    at REPLServer.self.eval (repl.js:111:21)
    at Interface.<anonymous> (repl.js:250:12)
    at Interface.EventEmitter.emit (events.js:88:17)
    at Interface._onLine (readline.js:199:10)
    at Interface._line (readline.js:517:8)
    at Interface._ttyWrite (readline.js:735:14)
    at ReadStream.onkeypress (readline.js:98:10)
    at ReadStream.EventEmitter.emit (events.js:115:20)
    at emitKey (readline.js:1057:12)
```

如果愿意，可以用另一种更具上下文的消息类型替换默认消息类型。通过将消息作为任何断言函数的最后一个参数传入来实现这一点：

```js
var result = 'ABC';
assert.equal(result, 'DEF', 'the result of operation X should be DEF');
```

# 执行异步测试

Mocha 按顺序运行所有测试，每个测试可以是同步的或异步的。对于同步测试，测试回调函数不应接受任何参数，就像前面的例子一样。但由于 Node.js 不会阻塞 I/O 操作，我们需要对每个测试执行 I/O 操作（至少向服务器发出一个 HTTP 请求），因此我们的测试需要是异步的。

要使测试变成异步，测试函数应该接受一个回调函数，就像这样：

```js
it('tests something asynchronous', function(done) {
  doSomethingAsynchronous(function(err) {
    assert.ok(! err);
 done();
  });
});
```

`done`回调函数还接受一个错误作为第一个参数，这意味着您可以直接调用`done`，而不是抛出错误：

```js
it('tests something asynchronous', function(done) {
  doSomethingAsynchronous(function(err) {
    done(err);
  });
});
```

如果不需要测试异步函数的返回值，可以直接传递`done`函数，就像这样：

```js
it('tests something asynchronous', function(done) {
 doSomethingAsynchronous(done);
});
```

**超时**：默认情况下，Mocha 为每个异步测试保留 2 秒。您可以通过向 Mocha 传递`-t`参数来全局更改这个时间：

```js
$ node_modules/.bin/mocha test.js -t 4s
```

在这里，您可以使用以`s`为后缀的秒数，如所示，或者您可以简单地传递毫秒数：

```js
$ node_modules/.bin/mocha test.js -t 4000
```

您还可以通过使用`this.timeout(ms)`来指定任何测试的超时，就像这样：

```js
it('tests something asynchronous', function(done) {
  this.timeout(500); // 500 milliseconds
  doSomethingAsynchronous(done);
});
```

# 总结

Mocha 是一个运行您的测试的框架。您应该根据您想要覆盖的功能区域将测试拆分为几个文件，然后描述每个功能并为每个功能定义必要的测试。

对于这些测试组中的每一个，您可以选择指定要使用`before`、`beforeEach`、`after`和`afterEach`来调用的回调函数。这些回调函数是指定设置和拆卸函数的地方。这些拆卸或设置函数中的每一个都可以是同步的或异步的。此外，这些测试本身也可以通过简单地将回调传递给测试来使其异步运行，一旦测试完成，回调就会被调用。

对于异步测试，Mocha 保留了默认的 2 秒超时，您可以在全局范围或每个测试的基础上进行覆盖。

在接下来的章节中，我们将看到如何开始使用 Zombie.js 来模拟和操纵浏览器。
