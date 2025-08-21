# 第八章：测试 AJAX

在本书中，我们已经测试了在表单上填写文本字段、点击按钮以及生成的 HTML 文档。这使我们准备好测试传统的基于表单的请求-响应应用程序，但典型的现代应用程序通常比这更复杂，因为它们使用异步 HTTP 请求，以某种方式更新文档而无需刷新它。这是因为它们使用了 AJAX。

当我们呈现待办事项列表页面时，我们的应用程序会发出 AJAX 请求；用户可以拖动一个项目并将其放在新位置。我们放在`public/js/todos.js`文件中的代码捕捉到变化，并调用服务器`/todos/sort` URL，改变数据库中的项目顺序。

让我们看看如何使用 Zombie 来测试这个拖放功能。本章涵盖的主题包括：

+   使用 Zombie 触发 AJAX 调用

+   使用 Zombie 来测试 AJAX 调用的结果

在本节结束时，您将知道如何使用 Zombie 来测试使用 AJAX 的应用程序。

# 实现拖放

让我们在`test/todos.js`文件中添加一些测试。

1.  我们首先在`Todo list`作用域结束之前添加一个新的描述作用域：

```js
describe('When there are some items on the list', function() {
```

这个新的作用域允许我们在运行此作用域内的任何测试之前在数据库中设置一个待办事项列表。

1.  现在，让我们在新的作用域内添加这个新的`beforeEach`钩子：

```js
beforeEach(function(done) {
  // insert todo items
  db.insert(fixtures.todos, fixtures.user.email, done);
});
```

1.  然后我们通过登录开始测试：

```js
it('should allow me to reorder items using drag and drop',
  login(function(browser, done) {
```

1.  我们通过确保我们的项目列表页面中有三个待办事项来开始测试：

```js
var items = browser.queryAll('#todo-list tr');
assert.equal(items.length, 3, 'Should have 3 items and has ' +
  items.length);
```

1.  然后我们声明一个辅助函数，将帮助我们验证列表的内容：

```js
function expectOrder(order) {
  var itemTexts = browser.queryAll('#todo-list tr .what').map(
    function(node) {
      return node.textContent.trim();
    }   assert.equal(index + 1, itemPos);
  });
}
```

这个函数获取一个字符串数组，并断言页面中每个待办事项的`what`和`pos`字段的顺序是否符合预期。

1.  然后我们使用这个新的`expectOrder`函数来实际测试顺序是否符合预期：

```js
expectOrder(['Do the laundry', 'Call mom', 'Go to gym']);
```

您可能还记得，在`test/fixtures.json`文件中声明的待办事项的顺序是在`beforeEach`钩子加载的。

1.  接下来我们创建另一个辅助函数，将帮助我们制造和注入鼠标事件：

```js
function mouseEvent(name, target, x, y) {
  var event = browser.document.createEvent('MouseEvents');
  event.initEvent(name, true, true);
  event.clientX = event.screenX = x;
  event.clientY = event.screenY = y;
  event.which = 1;
  browser.dispatchEvent(item, event);
}
```

这个函数模拟用户鼠标事件，设置了`x`和`y`坐标，设置了鼠标按钮（`event.which = 1`），并将事件分派到浏览器中，指定事件发生在哪个项目上。

1.  接下来我们选择要拖动的待办事项；在这种情况下，我们拖动第一个：

```js
var item = items[0];
```

1.  然后我们使用`mouseEvent`辅助函数来注入一系列制造的事件：

```js
mouseEvent('mousedown', item, 50, 50);
mouseEvent('mousemove', browser.document, 51, 51);
mouseEvent('mousemove', browser.document, 51, 150);
mouseEvent('mouseup',  browser.document, 51, 150);
```

这些事件有几个重要方面，即事件的顺序、目标元素和鼠标坐标。让我们来分析一下。

这些是组成拖放的事件。首先我们按下鼠标按钮，稍微移动一下，然后再移动一些，最后释放鼠标按钮。这里我们使用的鼠标事件位置的`x`和`y`值并不重要，重要的是它们之间的相对差异，以便检测到拖动并开始拖动模式。

在第一个事件中，`mousedown`，我们使用了一个任意的坐标`50, 50`。在第二个事件中，`mousemove`，我们将这个坐标增加了一个像素；这开始了拖动。

第二个`mousemove`事件在 y 轴上继续拖动。看起来多余和冗余，但它是必需的，以便拖动检测起作用，使我们执行的拖动移动连续。

最后，我们有`mouseup`，用户释放鼠标。这个事件使用了与前一个`mousemove`相同的坐标，表示用户在拖动后放下了元素。

现在让我们分析事件中的目标元素：

`mouseEvent()`助手函数的第二个参数是目标元素。在第一个`mousedown`事件注入中，我们将目标定位到`item`变量中的待办事项，该变量引用我们要拖动的项目。这表明了我们将要拖动的项目，一旦拖动模式被激活。其余的三个事件将目标定位到浏览器文档，因为用户将在整个文档中拖动待办事项。

我们正在使用的鼠标坐标的进一步澄清：

Zombie 不会渲染项目，因此它不知道每个项目的位置。这是我们可以使用的唯一方法来指示我们正在拖动的元素。在这种情况下，x 和 y 坐标与此无关。

由于 Zombie 不会渲染元素，它不会保留每个元素的位置。事实上，它们都被放置在(0, 0)处，这意味着我们的`mouseup`事件将拖动的项目放置在最后一个项目之后。

如前所述，初始值和拖动距离是完全任意的，您会发现改变这些值仍然可以使测试工作。

1.  在将这些鼠标事件注入浏览器事件队列后，我们使用`browser.wait()`函数等待这些事件被完全处理：

```js
browser.wait(function(err) {
            if (err) throw err;
```

在这个阶段，浏览器已经改变了元素顺序，并发出了一个 AJAX 请求，将新的顺序发送到服务器。

1.  现在我们验证待办事项是否按新顺序排列：

```js
expectOrder(['Call mom', 'Go to gym', 'Do the laundry']);
```

1.  我们还验证浏览器是否执行了我们预期的 HTTP 请求：

```js
var lastRequest = browser.lastRequest;
assert.equal(lastRequest.url, 'http://localhost:3000/todos/sort');
assert.equal(lastRequest.method, 'POST');
```

### 注意

请注意，我们正在使用`browser.lastRequest()`函数来访问浏览器发出的最后一个 AJAX 请求。

如果您需要访问浏览器发出的每个 HTTP 请求，可以检查`browser.resources`对象。

现在我们知道浏览器发出了一个`HTTP POST`请求，命令服务器对待办事项进行排序，我们需要确保数据库中的待办事项已经正确更新。为了验证这一点，我们做了类似于人工测试人员的操作；我们使用`browser.reload()`重新加载页面，并验证是否顺序确实是预期的：

```js
browser.reload(function(err) {
  if (err) throw err;

  expectOrder(['Call mom', 'Go to gym', 'Do the laundry']);

  done();

});
```

# 摘要

使用 Zombie，您可以注入自定义事件来模拟一些复杂的用户操作。您还可以通过使用`browser.lastRequest()`来检测浏览器执行 HTTP 请求的 URL 和方法。
