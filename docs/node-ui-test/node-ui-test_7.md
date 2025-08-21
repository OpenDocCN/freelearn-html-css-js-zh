# 第七章：调试

本章介绍了如何使用浏览器对象来检查应用程序的一些内部状态。

本章涵盖的主题包括：

+   启用调试输出

+   转储浏览器状态

默认情况下，Zombie 不会将内部事件输出到控制台，但您可以将 Zombie 运行时的`DEBUG`环境变量设置为`true`。如果您使用 UNIX shell 命令行，可以在启动测试套件时添加`DEBUG=true`，如下所示：

```js
$ DEBUG=true node_modules/.bin/mocha test/todos
```

如果您使用 Windows，可以按照以下方式设置和取消设置`DEBUG`环境变量：

```js
$ SET DEBUG=true
$ SET DEBUG=
```

通过启用此环境变量，Zombie 将输出其进行的每个 HTTP 请求，以及收到的 HTTP 状态代码：

```js
…
Zombie: GET http://localhost:3000/js/todos.js => 200
Zombie: 303 => http://localhost:3000/todos
Zombie: GET http://localhost:3000/todos => 200
Zombie: GET http://localhost:3000/js/jquery-1.8.2.js => 200
Zombie: GET http://localhost:3000/js/jquery-ui.js => 200
Zombie: GET http://localhost:3000/js/todos.js => 200
Zombie: GET http://localhost:3000/js/bootstrap.min.js => 200
…
```

### 注意

正如您所看到的，Zombie 还报告了所有`3xx-class`的 HTTP 重定向以及新的 URL 是什么。

这种输出可能有助于调试一些 URL 加载问题，但很难追踪特定 HTTP 请求所指的测试。

幸运的是，可以通过更改 Mocha 报告器来为测试输出带来一些澄清。Mocha 带有一种称为报告器的功能。到目前为止，我们使用的是默认报告器，它为每个测试报告一个有颜色的点。但是，如果您指定`spec`报告器，Mocha 会在测试开始之前和测试结束之后输出测试名称。

要启用`spec`报告器，只需将`-R spec`添加到 Mocha 参数中，如下所示：

```js
$ DEBUG=true node_modules/.bin/mocha -R spec test/todos
```

这样，您将获得类似以下的输出：

```js
...
      . should start with an empty list: Zombie: GET http://localhost:3000/session/new => 200
Zombie: GET http://localhost:3000/js/jquery-1.8.2.js => 200
Zombie: GET http://localhost:3000/js/jquery-ui.js => 200
Zombie: GET http://localhost:3000/js/bootstrap.min.js => 200
Zombie: GET http://localhost:3000/js/todos.js => 200
Zombie: 302 => http://localhost:3000/todos
Zombie: GET http://localhost:3000/todos => 200
Zombie: GET http://localhost:3000/js/jquery-1.8.2.js => 200
Zombie: GET http://localhost:3000/js/jquery-ui.js => 200
Zombie: GET http://localhost:3000/js/todos.js => 200
Zombie: GET http://localhost:3000/js/bootstrap.min.js => 200
      ✓ should start with an empty list (378ms)
      . should not load when the user is not logged in: Zombie: 303 => http://localhost:3000/session/new
Zombie: GET http://localhost:3000/session/new => 200
Zombie: GET http://localhost:3000/js/jquery-1.8.2.js => 200
Zombie: GET http://localhost:3000/js/jquery-ui.js => 200
Zombie: GET http://localhost:3000/js/bootstrap.min.js => 200
Zombie: GET http://localhost:3000/js/todos.js => 200
      ✓ should not load when the user is not logged in (179ms)
...
```

这不仅告诉您给定测试对应的资源加载情况，还告诉您运行该测试所花费的时间。

# 运行特定测试

如果您遇到特定测试的问题，您无需运行整个测试套件甚至整个测试文件。Mocha 接受`-g <expression>`命令行选项，并且只运行与该表达式匹配的测试。

例如，您可以仅运行描述中包含`remove`一词的测试，如下所示：

```js
$ DEBUG=true node_modules/.bin/mocha -R spec -g 'remove' test/todos

  Todos
    Todo removal form
      When one todo item exists
        ◦ should allow you to remove: Zombie: GET http://localhost:3000/session/new => 200
...
        ✓ should allow you to remove (959ms)
      When more than one todo item exists
        ◦ should allow you to remove one todo item: Zombie: GET http://localhost:3000/session/new => 200
...
        ✓ should allow you to remove one todo item (683ms)

  ✔ 2 tests complete (1780ms)
```

这样，您将只运行这些特定测试。

## 启用每个测试的调试输出

将`DEBUG`环境变量设置为`true`可启用所有测试的调试输出，但您也可以通过将`browser.debug`设置为`true`来指定要调试的测试。例如，更改`test/todos.js`文件，大约在第 204 行添加以下内容：

```js
...
      it("should allow you to remove", login(function(browser, done) {
 browser.debug = true;

      browser.visit('http://localhost:3000/todos', function(err, browser) {
...
```

这样，当运行以下测试时，您无需指定`DEBUG`环境变量：

```js
$ node_modules/.bin/mocha -R spec -g 'remove' test/todos

  Todos
    Todo removal form
      When one todo item exists
        . should allow you to remove: Zombie: GET http://localhost:3000/todos => 200
Zombie: GET http://localhost:3000/js/jquery.min.js => 200
Zombie: GET http://localhost:3000/js/jquery-ui-1.8.23.custom.min.js => 200
Zombie: GET http://localhost:3000/js/bootstrap.min.js => 200
Zombie: GET http://localhost:3000/js/todos.js => 200
Zombie: 303 => http://localhost:3000/todos
Zombie: GET http://localhost:3000/todos => 200
Zombie: GET http://localhost:3000/js/jquery.min.js => 200
Zombie: GET http://localhost:3000/js/jquery-ui-1.8.23.custom.min.js => 200
Zombie: GET http://localhost:3000/js/bootstrap.min.js => 200
Zombie: GET http://localhost:3000/js/todos.js => 200
        ✓ should allow you to remove (1191ms)
      When more than one todo item exists
        ✓ should allow you to remove one todo item (926ms)

  ✔ 2 tests complete (2308ms)
```

在这里，您可以看到，正如预期的那样，Zombie 仅为名为`should allow you to remove`的测试输出调试信息。

# 使用浏览器 JavaScript 控制台

除了浏览器发出的 HTTP 请求之外，Zombie 不会输出其他可能有趣或有用的内容，以便您调试应用程序。

一个很好的选择，提供了更多的灵活性和洞察力，是在真实浏览器中运行应用程序，并使用开发者工具和/或调试器。

在特定于 Zombie.js 的问题调试中，一个特别有用的替代方法是在浏览器代码中使用`console.log()`函数（在本应用程序的情况下，该代码位于`public/js`目录中）。

例如，假设您在处理待办事项创建表单时遇到问题：警报选项未正确触发警报选项窗格的显示和隐藏。为此，我们可以在`public/js/todos.js`文件中引入以下`console.log`语句，以检查`ringAlarm`变量的值：`hideOrShowDateTime()`函数。

```js
  {
    var ringAlarm = $('input[name=alarm]:checked',
      '#new-todo-form').val() === 'true';

 console.log('\ntriggered hide or show. ringAlarm is ', ringAlarm);

    if (ringAlarm) {
      $('#alarm-date-time').slideDown();
    } else {
      $('#alarm-date-time').slideUp();
    }
  }
```

这样，当您运行测试时，您将获得以下输出：

```js
$ node_modules/.bin/mocha -R spec -g 'alarm' test/todos

  Todos
    Todo creation form
      . should have an alarm option: 
triggered hide or show. ringAlarm is  false

triggered hide or show. ringAlarm is  false

triggered hide or show. ringAlarm is  false
      ✓ should have an alarm option (625ms)
      . should present the alarm date form fields when alarm: 
triggered hide or show. ringAlarm is  false

triggered hide or show. ringAlarm is  false

triggered hide or show. ringAlarm is  false

triggered hide or show. ringAlarm is  false

triggered hide or show. ringAlarm is  true

triggered hide or show. ringAlarm is  true

triggered hide or show. ringAlarm is  false

triggered hide or show. ringAlarm is  false
      ✓ should present the alarm date form fields when alarm (1641ms)

  ✔ 2 tests complete (2393ms)
```

使用这种技术，您可以在运行测试时检查应用程序的状态。

# 转储浏览器状态

您还可以通过在测试代码中调用`browser.dump()`函数来检查浏览器状态。

1.  例如，您可能想在`test/todos.js`文件中的`should present the alarm date form fields when alarm`测试中了解完整的浏览器状态。为此，在我们选择“无警报”选项后立即引入`browser.dump()`调用：

```js
...
    it('should present the alarm date form fields when alarm', 
      login(function(browser, done) {
        browser.visit('http://localhost:3000/todos/new', function(err) {
          if (err) throw err;

          var container = browser.query('#alarm-date-time');

          browser.choose('No Alarm', function(err) {
            if (err) throw err;

            assert.equal(container.style.display, 'none');

            browser.choose('Use Alarm', function(err) {
              if (err) throw err;

              assert.equal(container.style.display, '');

              browser.choose('No Alarm', function(err) {
                if (err) throw err;

 browser.dump();

                assert.equal(container.style.display, 'none');

                done();
              });
            });
          });
        });
      })
    );
...
```

1.  在文件中进行更改并运行此测试：

```js
$ node_modules/.bin/mocha -R spec -g 'alarm' test/todos

  Todos
    Todo creation form
      ✓ should have an alarm option (659ms)
      ◦ should present the alarm date form fields when alarm: Zombie: 1.4.1

URL: http://localhost:3000/todos/new
History:
  1\. http://localhost:3000/session/new
  2\. http://localhost:3000/todos
  3: http://localhost:3000/todos/new

sid=AIUjSvUl79S8Qz4Q8foRRAS7; Domain=localhost; Path=/
Cookies:
  true

Storage:

Eventloop:
  The time:   Mon Feb 18 2013 10:59:43 GMT+0000 (WET)
  Timers:     0
  Processing: 0
  Waiting:    0

Document:
  <html>
    <head>    <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
      <title id="title">New To-Do</title>
      <link href="/css/bootstrap.min.css" rel="stylesheet" />
      <link href="/css/todo.css" rel="stylesheet" />
  </head>
    <body>    <section role="main" class="container">      <div id="messages"></div>
        <div id="main-body">
          <h1>New To-Do</h1>
          <form id="new-todo-form" action="/todos" method="POST">          <p>            <label for="what">What</...

      ✓ should present the alarm date form fields when alarm (1426ms)

  ✔ 2 tests complete (2236ms)

```

进行`browser.dump()`调用时，您将在输出中获得以下内容：

+   当前 URL

+   历史记录，即此浏览器实例在创建后访问的所有 URL

+   离线存储，如果您使用任何

+   事件循环状态：如果它正在等待任何处理或计时器

+   HTML 文档的第一行，这可能足以调试当前状态

# 转储整个文档

如果您需要随时检查文档的全部内容，可以检查`browser.html()`的返回值。例如，如果您想在重新加载浏览器之前检查文档的状态，可以在`test/todo.js`文件中添加以下行，而不是`browser.dump()`：

```js
...
            browser.choose('Use Alarm', function(err) {
              if (err) throw err;

              assert.equal(container.style.display, '');

              browser.choose('No Alarm', function(err) {
                if (err) throw err;

                console.log(browser.html());

                assert.equal(container.style.display, 'none');

                done();
              });
            });
...
```

现在您可以运行测试并观察输出：

```js
$ node_modules/.bin/mocha -g 'alarm' test/todos
...
  <html style=""><head>

...
```

# 摘要

您的浏览器开发人员工具更适合调试浏览器应用程序。但是，如果遇到特定于 Zombie 的问题，有几种技术可能会对您有所帮助。

一种是启用 Zombie 调试输出。这将显示浏览器正在加载的资源以及显示在旁边的相应响应状态代码。

您可以运行特定的测试。在调试测试中的特定问题时，还可以通过使用`-g <pattern>`选项来限制 Mocha 仅运行该测试。

您可以在浏览器中运行的代码中使用`console.log`命令；输出将显示在控制台中。

您可以查看当前的浏览器状态。您可以通过使用`browser.dump`调用或将`browser.html`的结果记录到控制台来检查浏览器状态。

如果您需要在测试的某个阶段访问整个文档，还可以记录`browser.html()`的返回值。
