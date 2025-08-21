# 第五章：操纵僵尸浏览器

现在我们有了待办 HTTP 应用程序，并且了解了 Mocha 测试框架的工作原理，我们准备开始使用 Zombie.js 创建测试。

如前所述，Zombie.js 允许您创建一个模拟的浏览器环境并对其进行操作。这些操作是用户在浏览器中通常做的事情，比如访问 URL，点击链接，填写和提交表单等。

本章涵盖以下内容：

+   访问 URL

+   填写和提交表单

+   检查浏览器中的错误

+   验证文档内容

+   理解 CSS 选择器语法

本章向您展示了如何设置一个与您的 Web 应用程序交互的 Zombie.js 浏览器。

**访问 URL**：首先，我们将从上次离开的地方继续进行应用测试。整个应用涉及用户，但在这部分中，我们主要将关注`Users`路由涉及的功能-渲染注册表单和实际在数据库中创建用户记录。

如前所述，我们离开了这个单一的测试文件：

```js
var assert  = require('assert'),
    Browser = require('zombie'),
    app     = require('../app')
    ;

describe('Users', function() {

  before(function(done) {
    app.start(3000, done);
  });

  after(function(done) {
    app.server.close(done);
  });

  describe('Signup Form', function() {

    it('should load the signup form', function(done) {
      var browser = new Browser();
      browser.visit("http://localhost:3000/users/new", function() {
        assert.ok(browser.success, 'page loaded');
        done();
      });
    });

  });
});
```

这个测试只是加载了用户注册表单，并测试浏览器是否认为它是成功的。让我们通过这个测试来完全理解发生了什么。

首先，我们通过实例化一个新的浏览器对象来创建一个新的浏览器：

```js
var browser = new Browser();
```

这样创建了一个 Zombie.js 浏览器，它代表一个独立的浏览器进程，主要工作是在请求之间保持状态：URL 历史记录，cookies 和本地存储。

浏览器还有一个主窗口，你可以使用`browser.visit()`在其中加载一个 URL，就像这样：

```js
browser.visit("http://localhost:3000/users/new");
```

这使得浏览器执行一个 HTTP `GET`请求来从该 URL 加载 HTML 页面。由于 Node.js 和 Zombie.js 进行异步 I/O 处理，这只会使 Zombie.js 开始加载页面。然后 Zombie.js 尝试获取 URL，解析 HTML 文档，并通过加载引用的 JavaScript 文件来解析所有依赖项。

一旦所有这些都完成了，我们可以通过将回调函数传递给`browser.wait()`方法来得到通知，就像这样：

```js
browser.visit("http://localhost:3000/users/new");
browser.wait(function() {
  console.log('browser page loaded');
});
```

我们不是使用`browser.wait`函数，而是直接将回调传递给`browser.visit()`调用，就像这样：

```js
browser.visit("http://localhost:3000/users/new",
  function(err, browser) {
    if (err) throw err;
    assert.ok(browser.success, 'page loaded');
    done();
  }
);
```

在这里，您传递一个回调函数，一旦出现错误或浏览器准备好，就会被调用。如果发生错误，它将作为第一个参数返回-我们检查是否存在错误，并在存在时抛出它，以便测试失败。

第二个参数包含浏览器对象，与我们已经有的浏览器对象相同。这意味着我们可以完全省略第二个参数，并使用之前的浏览器引用，就像这样：

```js
browser.visit("http://localhost:3000/users/new",
  function(err) {
    if (err) throw err;
    assert.ok(browser.success, 'page loaded');
    done();
  }
);
```

如果是同一个浏览器对象，你可能会问为什么要传递那个对象。它是为了支持这种调用形式：

```js
var Browser = require('zombie');

Browser.visit(("http://localhost:3000/users/new",
  function(err, browser) {
    if (err) throw err;
    assert.ok(browser.success, 'page loaded');
    done();
  }
);
```

请注意，这里我们正在使用大写的伪类`Browser`对象；我们没有实例化`browser`。相反，我们将这个工作留给`Browser`模块来做，并将它作为回调函数的第二个参数传递给我们。

### 注意

从现在开始，我们将更喜欢这种简洁的形式，而不是这里显示的其他形式。

# 浏览器何时准备好？

当我们要求浏览器访问一个 URL 时，它在完成时会回调我们，但是正如网页开发者所知，很难准确知道何时可以认为页面加载完全完成

浏览器对象有自己的事件循环，处理异步事件，如加载资源、事件、超时和间隔。页面加载和解析完成后，所有依赖项都会异步加载和解析-就像在真实浏览器中一样-使用这个事件循环。

其中一些依赖项可能包含将被加载、解析和评估的 JavaScript 文件。此外，HTML 文档可能包含一些额外的内联脚本，将被执行。如果其中任何脚本有一个等待文档准备就绪的回调，这些回调将在您的`browser.visit()`回调触发测试回调之前执行。这意味着，例如，如果您有在文档准备就绪时触发的 jQuery 代码，它将在您的回调之前运行。对于任何后续的 AJAX 回调也是如此。

要查看此操作，请尝试在`templates/layout.html`文件的关闭`</body>`标记之前立即添加以下代码：

```js
    <script>
      $(function() {
        $.get('/users/new', function() {
          console.log('LOADED NEW');
        });
      });
    </script>
```

然后更改`test/users.js`中的测试代码，以便在访问回调被触发时记录日志：

```js
it('should load the signup form', function(done) {
  Browser.visit("http://localhost:3000/users/new", function(err, browser) {
    if (err) throw err;
    console.log('VISIT IS DONE');
    assert.ok(browser.success, 'page loaded');
    done();
  });
});
```

为了分析这一点，我们将以调试模式运行我们的测试。在此模式下，Zombie.js 输出一些有用的信息，包括浏览器正在执行的 HTTP 请求活动。要启用此模式，请设置`DEBUG`环境变量，像这样：

```js
$ DEBUG=true node_modules/.bin/mocha test/users.js
```

现在您应该获得以下调试输出：

```js
Zombie: GET http://localhost:3000/users/new => 200
Zombie: GET http://localhost:3000/js/jquery.min.js => 200
Zombie: GET http://localhost:3000/js/jquery-ui-1.8.23.custom.min.js => 200
Zombie: GET http://localhost:3000/js/bootstrap.min.js => 200
Zombie: GET http://localhost:3000/js/todos.js => 200
Zombie: GET http://localhost:3000/users/new => 200
LOADED NEW
VISIT IS DONE
.

  ✔ 1 test complete (315ms)
```

### 注意

如果您是 Windows 用户，则最后一个命令将无法工作。在运行 Mocha 命令之前，您需要设置`DEBUG`环境变量：

```js
$ SET DEBUG=true
```

您还需要将正斜杠(`/`)替换为反斜杠(`\`)：

```js
$ node_modules\.bin\mocha test\users.js
```

正如您所看到的，`LOADED NEW`字符串在`VISIT IS DONE`字符串之前打印，这意味着浏览器在访问回调触发之前执行并完成了 AJAX 请求。您现在可能希望返回代码并删除这些额外的控制台日志。

## 访问 URL 时的选项

您还可以向浏览器传递一些选项，以修改它加载页面的一些操作和条件。这些选项以对象的形式传递给`Browser.visit()`调用的参数，就在回调之前，像这样：

```js
Browser.visit(<url>, <options>, <callback>);
```

以下是我们将详细讨论的最有用的选项：

+   调试

+   标题

+   maxWait

### 调试

正如我们所看到的，通过设置`DEBUG`环境变量，您可以从 Zombie.js 获得一些输出。通过将`debug`选项设置为`true`，也可以激活此功能，像这样：

```js
Browser.visit(url, {debug: true}, callback);
```

### 标题

您可以定义一组标头，以便在每个源自此访问的 HTTP 请求上发送。默认情况下，Zombie.js 发送这些标头值：

+   **用户代理**：Mozilla/5.0，Chrome/10.0.613.0，Safari/534.15，或 Zombie.js/1.4.1

+   **接受编码**：身份

+   **主机**：localhost:3000

+   **连接**：保持连接

`user-agent`标头定义了一个虚假的用户代理，有些类似于 Mozilla，Chrome 和 Safari 浏览器，但您可以在此设置中更改它，稍后会看到。

`accept-encoding`标头指定结果文档不应进行编码。

`host`标头是 HTTP 1.1 的必需项，指定了此请求所引用的主机名。

`connection: keep-alive`标头指定在请求完成后应保持与服务器的连接。这是一个内部选项，允许 Node 在许多 HTTP 连接中重用客户端套接字，这将略微加快您的测试速度。

要添加额外的标头值，如果您的应用程序需要任何标头值，请像这样指定它们：

```js
var options = {
  headers: {
    'x-test': 'Test 123',
    'x-test-2': 'Test 234'
  }
};
Browser.visit(url, options, callback);
```

请注意，这些值在加载依赖项时也将发送给每个请求，例如在 HTML 文档中引用的后续 CSS 和 JavaScript 文件。

### maxWait

默认情况下，调用`Browser.visit`时，Zombie.js 加载页面，解析页面，加载依赖项，并在浏览器中运行任何待处理的 JavaScript 代码。如果这需要超过 5 秒，将引发错误并使您的测试失败。如果由于任何原因，5 秒不足以完成所有这些操作，则可以通过像这样更改`maxWait`选项来增加限制：

```js
Browser.visit(url, {maxWait: '10s'}, callback);
```

您可以将值指定为字符串，如`10ms`，`100ms`，`7.5s`等。

# 检查元素的存在

当`Browser.visit()`回调被触发时，我们检查错误。我们还检查页面是否成功加载，如果 HTTP 响应状态码在 200 到 299 之间。这些 2XX 响应代码对应于`ok`请求状态，并且是服务器告知用户代理一切顺利进行的方式的一部分。

尽管收到了一个`ok`响应，我们不应该轻信服务器的话。我们可能已经收到了响应状态码和一个 HTML 文档，但不能确定我们是否得到了包含用户注册表标记的预期文档。

在我们的情况下，我们可能希望验证文档是否包含一个包含`New User`字符串的标题元素，并且新用户表单元素是否存在。以下是完整测试的代码：

```js
it('should load the signup form', function(done) {
  Browser.visit("http://localhost:3000/users/new", function(err, browser) {
    if (err) throw err;
    assert.ok(browser.success, 'page loaded');
 assert.equal(browser.text('h1'), 'New User');

 var form = browser.query('form');
 assert(form, 'form exists');
 assert.equal(form.method, 'POST', 'uses POST method');
 assert.equal(form.action, '/users', 'posts to /users');

 assert(browser.query('input[type=email]#email', form),
 'has email input');
 assert(browser.query('input[type=password]#password', form),
 'has password input');
 assert(browser.query('input[type=submit]', form),
 'has submit button');

    done();
  });
});
```

测试中的新行已经突出显示。现在让我们逐个查看它们。

```js
assert.equal(browser.text('h1'), 'New User');
```

在这里，`browser.text(<selector>)`被用来提取`h1`标签的文本内容（如果至少有一个存在）。

### 注意

如果选择器匹配多个 HTML 元素（如果在文档中有多个`h1`标签），`browser.text(<selector>)`将返回所有匹配节点的连接文本。

在这里，选择器只是标记名称，但您可以使用任何 Sizzle 有效的选择器。这些类似于 CSS3 选择器，也用于 jQuery。如果您对此不熟悉，不用担心，我们将在未来看到更多这方面的例子。

```js
var form = browser.query('form');
assert(form, 'form exists');
```

### 注意

浏览器（以及所有浏览器）将当前文档的表示存储在一个可访问的结构中，称为**文档对象模型**（**DOM**）。文档中的 HTML 标记由浏览器解析，并构建 DOM 树。可以使用 JavaScript 以编程方式遍历此 DOM。

在这里，我们使用`browser.query(<selector>)`方法来提取第一个表单元素。这个元素是一个 DOM 节点，就像您在浏览器中找到的那样，并且符合 DOM 规范。目前，我们只是测试它是否存在。之后，我们将检查一些属性是否正确：

```js
assert.equal(form.method, 'POST', 'uses POST method');
assert.equal(form.action, '/users', 'posts to /users');
```

在这里，我们正在验证表单方法是否为`POST`，以及当用户提交时，它是否实际上发布到`/users` URL。

接下来，我们验证是否存在创建用户所需的表单元素：

```js
assert(browser.query('input[type=email]#email', form),
  'has email input');
assert(browser.query('input[type=password]#password', form),
  'has password input');
assert(browser.query('input[type=submit]', form),
  'has submit button');
```

我们使用`browser.query(<selector>, <context>)`形式来检索第一个匹配的节点，但这次，我们将搜索限制在`<context>`的子集中，这在我们的情况下是我们的`form`节点。我们还在这里使用更复杂的选择器，将标记名称选择器（`form`）与 ID 选择器`#id`和属性选择器`[type=email]`结合使用。例如，第一个选择器`input[type=email]#email`选择具有类型`email`属性和值`email`的 ID 的输入。这样，我们断言这样的元素存在，因为如果不存在，`browser.query()`调用将返回`undefined`，破坏断言调用。

# 填写表单

一旦加载了包含用户订阅表单的页面，您就可以填写表单并将其提交回服务器。为此，我们将使用一个新的测试用例：

```js
it("should submit", function(done) {
  Browser.visit("http://localhost:3000/users/new", function(err, browser) {
    if (err) throw err;

    browser
      .fill('E-mail', 'me@email.com')
      .fill('Password', 'mypassword')
      .pressButton('Submit', function(err) {
        if (err) throw err;
        assert.equal(browser.text('h1'), 'Thank you!');
        assert(browser.query('a[href="/session/new"]'),
          'has login link');
        done();
      });

  });
});
```

在这里，我们重新访问用户创建表单，一旦表单加载完成，我们就使用`browser.fill(<field>, <value>)`方法填写电子邮件和密码填写。在这个表单中，`browser.fill()`接受几种类型的参数作为字段标识符。在这里，我们使用了字段之前的标签文本。如果查看空的用户创建表单的源代码，它将是：

```js
<form action="/users" method="POST">
  <p>
    <label for="email">E-mail</label>
    <input type="email" name="email" value="" id="email">
  </p>
  <p>
    <label for="password">Password</label>
    <input type="password" name="password" id="password" value="" required="">
  </p>
  <input type="submit" value="Submit">
</form>
```

我们在这里使用的两个标签标签都有一个`for`属性，指示它所关联的标签的`id`属性。这是 Zombie.js 用来匹配`browser.fill()`中的字段的方法。或者，我们还可以指定字段名称或 CSS 选择器，使以下填充指令等同于我们所拥有的：

```js
    browser
      .fill('#email', 'me@email.com')
      .fill('#password', 'mypassword')
```

然后，您可以在 shell 控制台上运行测试：

```js
$ ./node_modules/.bin/mocha test/users.js
```

只要 CouchDB 服务器可访问，这些测试就应该通过：

```js
  ..

  ✔ 2 tests complete (577ms)
```

但是，如果再次运行测试，它们应该失败。现在试试看：

```js
  ..

  ✖ 1 of 2 tests failed:

  1) Users Signup Form should submit:
     Error: Server returned status code 409
...
```

这是因为我们不允许使用相同电子邮件地址的两个用户，浏览器会产生 409 响应代码作为这种用户创建请求的结果。您可以在每次测试之前手动从数据库中删除用户文档，但为了完全解决这个问题，我们需要自动化这个过程。

首先，我们将介绍固定装置的概念。这是我们将为用户定义用户名和密码的地方，这将在其他测试中使用。然后，您需要创建一个文件，在`test/fixtures.json`下，目前包含以下数据：

```js
{
  "user" : {
    "email": "me@email.com",
    "password": "mypassword"
  }
}
```

然后，`users`测试文件将通过在顶部放置`require`来消耗此 JSON 文件：

```js
var fixtures = require('./fixtures');
```

然后，您还需要访问数据库，为此我们使用与路由监听器使用相同的库：

```js
var couchdb = require('../lib/couchdb'),
    dbName  = 'users',
    db      = couchdb.use(dbName);
```

现在我们需要在`Signup Form`测试描述范围内添加一个 before hook：

```js
before(function(done) {
  db.get(fixtures.user.email, function(err, doc) {
    if (err && err.status_code === 404) return done();
    if (err) throw err;
    db.destroy(doc._id, doc._rev, done);
  });
});
```

这将确保我们的数据库中没有这样的用户记录。

现在我们正在使用固定装置，让我们从测试代码中删除那些硬编码的用户名和密码字符串：

```js
it("should submit", function(done) {

  Browser.visit("http://localhost:3000/users/new", function(err, browser) {
    if (err) throw err;

    browser
      .fill('E-mail', fixtures.user.email)
      .fill('Password', fixtures.user.password)
      .pressButton('Submit', function(err) {
        if (err) throw err;
        assert.equal(browser.text('h1'), 'Thank you!');
        assert(browser.query('a[href="/session/new"]'),
          'has login link');
        done();
      });

  });
});
```

这将是整个组装的用户测试文件：

```js
var assert  = require('assert'),
    Browser = require('zombie'),
    app     = require('../app'),
    couchdb = require('../lib/couchdb'),
    dbName  = 'users',
    db      = couchdb.use(dbName),
    fixtures = require('./fixtures');

describe('Users', function() {

  before(function(done) {
    app.start(3000, done);
  });

  after(function(done) {
    app.server.close(done);
  });

  describe('Signup Form', function() {

    before(function(done) {
      db.get(fixtures.user.email, function(err, doc) {
        if (err && err.status_code === 404) return done();
        if (err) throw err;
        db.destroy(doc._id, doc._rev, done);
      });
    });

    it('should load the signup form', function(done) {
      Browser.visit("http://localhost:3000/users/new", function(err, browser) {
        if (err) throw err;
        assert.ok(browser.success, 'page loaded');
        assert.equal(browser.text('h1'), 'New User');

        var form = browser.query('form');

        assert(form, 'form exists');
        assert.equal(form.method, 'POST', 'uses POST method');
        assert.equal(form.action, '/users', 'posts to /users');

        assert(browser.query('input[type=email]#email', form),
          'has email input');
        assert(browser.query('input[type=password]#password', form),
          'has password input');
        assert(browser.query('input[type=submit]', form),
          'has submit button');

        done();
      });
    });

    it("should submit", function(done) {

      Browser.visit("http://localhost:3000/users/new", function(err, browser) {
        if (err) throw err;

        browser
          .fill('E-mail', fixtures.user.email)
          .fill('Password', fixtures.user.password)
          .pressButton('Submit', function(err) {
            if (err) throw err;
            assert.equal(browser.text('h1'), 'Thank you!');
            assert(browser.query('a[href="/session/new"]'),
              'has login link');
            done();
          });

      });
    });

  });
});
```

当重复运行此测试时，现在应该总是会收到成功消息。

# 测试登录表单

现在我们已经测试了用户创建流程，让我们测试一下该用户是否可以登录。

按照我们一直使用的测试文件模式，您需要创建一个文件，在`test/session.js`下，内容如下：

1.  首先，导入缺少的依赖项：

```js
var assert  = require('assert'),
    Browser = require('zombie'),
    app     = require('../app'),
    couchdb = require('../lib/couchdb'),
    dbName  = 'users',
    db      = couchdb.use(dbName),
    fixtures = require('./fixtures');

describe('Session', function() {

  before(function(done) {
    app.start(3000, done);
  });

  after(function(done) {
    app.server.close(done);
  });
```

这就结束了开幕式！

1.  现在我们准备开始描述登录表单：

```js
  describe('Log in form', function() {

    before(function(done) {
      db.get(fixtures.user.email, function(err, doc) {
        if (err && err.status_code === 404) {
 return db.insert(fixtures.user, fixtures.user.email, done);
 }
        if (err) throw err;
        done();
      });
    });
```

此`before`钩子将创建测试用户文档（如果不存在）（而不是在存在时删除）。

1.  接下来，我们将测试登录表单是否加载并包含相关元素：

```js

    it('should load', function(done) {
      Browser.visit("http://localhost:3000/session/new",
        function(err, browser) {
          if (err) throw err;
          assert.ok(browser.success, 'page loaded');
          assert.equal(browser.text('h1'), 'Log in');

          var form = browser.query('form');

          assert(form, 'form exists');
          assert.equal(form.method, 'POST', 'uses POST method');
          assert.equal(form.action, '/session', 'posts to /session');

          assert(browser.query('input[type=email]#email', form),
            'has email input');
          assert(browser.query('input[type=password]#password', form),
            'has password input');
          assert(browser.query('input[type=submit]', form),
            'has submit button');

          done();
        });
    });
```

这里与用户代码的唯一区别是标题字符串应为`登录`，而不是`新用户`。这是因为我们目前使用了这样一个最小的用户创建表单。

1.  接下来，我们正在测试登录表单是否实际有效：

```js
    it("should allow you to log in", function(done) {

      Browser.visit("http://localhost:3000/session/new",
        function(err, browser) {
          if (err) throw err;

          browser
            .fill('E-mail', fixtures.user.email)
            .fill('Password', fixtures.user.password)
            .pressButton('Log In', function(err) {
              if (err) throw err;

              assert.equal(browser.location.pathname, '/todos',
                'should be redirected to /todos');
              done();
            });

        });
    });

  });
});
```

在这里，我们正在加载并填写电子邮件和密码字段，然后单击**登录**按钮。单击按钮后，登录表单将被发布，会话将被启动，并且用户将被重定向到待办事项页面。

1.  现在从命令行运行此测试文件：

```js
$ ./node_modules/.bin/mocha test/session.js
  ․․

  ✔ 2 tests complete (750ms)
```

1.  此测试包括用户输入正确用户名和密码的情况，但如果不是这种情况会发生什么？让我们为此创建一个测试用例：

```js
it("should not allow you to log in with wrong password", function(done) {

  Browser.visit("http://localhost:3000/session/new",
    function(err, browser) {
      if (err) throw err;

      browser
        .fill('E-mail', fixtures.user.email)
        .fill('Password', fixtures.user.password +
          'thisisnotmypassword')
        .pressButton('Log In', function(err) {
          assert(err, 'expected an error');
          assert.equal(browser.statusCode, 403, 
            'replied with 403 status code');
          assert.equal(browser.location.pathname, '/session');
          assert.equal(browser.text('#messages .alert .message'),
            'Invalid password');
          done();
        });
    }
  );
});
```

在这里，我们正在加载并填写登录表单，但这次我们提供了错误的密码。单击**登录**按钮后，服务器应返回`403 状态码`，这将触发传递给我们回调函数的错误。然后，我们需要通过检查`browser.statusCode`属性来检查返回状态码，确保它是预期的 403 禁止代码。然后，我们还要验证用户是否没有被重定向到`/todo` URL，并且响应文档是否包含一个警报消息，说`无效密码`。

# 测试待办事项列表

现在我们已经完成了用户注册和会话启动，我们准备测试我们的应用程序的核心，即管理待办事项。我们将首先将应用程序测试的这一部分分离到一个自己的文件中，即`test/todos.js`，它可能以以下样板开始：

```js
var assert   = require('assert'),
    Browser  = require('zombie'),
    app      = require('../app'),
    couchdb  = require('../lib/couchdb'),
    dbName   = 'todos',
    db       = couchdb.use(dbName),
    fixtures = require('./fixtures'),
    login    = require('./login');

describe('Todos', function() {

  before(function(done) {
    app.start(3000, done);
  });

  after(function(done) {
    app.server.close(done);
  });

  beforeEach(function(done) {
    db.get(fixtures.user.email, function(err, doc) {
      if (err && err.status_code === 404) return done();
      if (err) throw err;
      db.destroy(doc._id, doc._rev, done);
    });
  });
});
```

在这里，我们有其他模块的类似样板代码，不同之处在于现在我们处理的是名为`todos`而不是`users`的数据库。另一个不同之处是我们希望每次测试都从一个干净的待办事项列表开始，因此我们添加了一个`beforeEach`钩子，用于删除测试用户的所有待办事项。

我们现在准备开始制定一些测试，但至少有一个繁琐的重复任务可以在早期避免：登录。我们应该假设每个测试都可以单独重现，并且测试的顺序并不重要——每个测试应该依赖于一个浏览器实例，模拟每个测试一个独立的用户会话。此外，由于所有待办事项操作都限定在用户和用户会话中必须初始化，我们需要将其抽象成自己的模块，放在`test/login.js`中：

```js
var Browser = require('zombie'),
    fixtures = require('./fixtures'),
    assert = require('assert'),
    couchdb = require('../lib/couchdb'),
    dbName  = 'users',
    db      = couchdb.use(dbName);

function ensureUserExists(next) {
  db.get(fixtures.user.email, function(err, user) {
    if (err && err.status_code === 404) {
      db.insert(fixtures.user, fixtures.user.email, next);
    }
    if (err) throw err;
    next();
  });
}

module.exports = function(next) {
  return function(done) {

    ensureUserExists(function(err) {
      if (err) throw err;
      Browser.visit("http://localhost:3000/session/new",
        function(err, browser) {
          if (err) throw err;

          browser
            .fill('E-mail', fixtures.user.email)
            .fill('Password', fixtures.user.password)
            .pressButton('Log In', function(err) {
              if (err) throw err;
              assert.equal(browser.location.pathname, '/todos');
              next(browser, done);
            });

        });
    });
  };
};
```

该模块确保在加载、填写和提交用户登录表单之前存在一个测试用户。之后，它将控制权交给`next`函数。

## 测试待办事项列表页面

现在我们准备在`todos`范围内添加更多的描述范围。其中一个范围是待办事项列表，其中将包含以下代码：

```js
  describe('Todo list', function() {

    it('should have core elements', login(function(browser, done) {
      assert.equal(browser.text('h1'), 'Your To-Dos');
      assert(browser.query('a[href="/todos/new"]'),
        'should have a link to create a new Todo');
      assert.equal(browser.text('a[href="/todos/new"]'), 'New To-Do');
      done();
    }));

    it('should start with an empty list', login(function(browser, done) {
      assert.equal(browser.queryAll('#todo-list tr').length, 0,
        'To-do list length should be 0');
      done();
    }));

    it('should not load when the user is not logged in', function(done) {
      Browser.visit('http://localhost:3000/todos', function(err, browser) {
        if (err) throw err;
        assert.equal(browser.location.pathname, '/session/new',
          'should be redirected to login screen');
        done();
      });
    });

  });
```

在这里，我们可以看到我们正在使用我们的`login`模块来抽象出会话初始化过程，确保我们的回调函数只有在用户登录后才会被调用。这里有三个测试。

在我们的第一个测试中，名为`应该具有核心元素`，我们只是加载空的待办事项列表，并断言我们已经放置了一些元素，例如包含`Your To-dos`文本的标题和创建新待办事项的链接。

在以下测试中，名为`应该以空列表开始`，我们只是测试待办事项列表是否包含零个元素。

在此范围的最后一个测试中，名为`当用户未登录时不应加载`，我们断言该列表对尚未初始化会话的用户是不可访问的，确保如果我们尝试加载`待办事项列表`URL，他会被重定向到`/session/new`。

## 测试待办事项创建

现在，我们需要测试待办事项是否真的可以创建。为此，请按照以下步骤进行：

1.  我们需要一个新的描述范围，我们将其命名为`待办事项创建表单`，这将是`Todos`的另一个子范围：

```js
  describe('Todo creation form', function() {
```

1.  现在我们可以测试一下，看看未登录的用户是否可以使用待办事项创建表单：

```js
    it('should not load when the user is not logged in', function(done) {
      Browser.visit('http://localhost:3000/todos/new', function(err, browser) {
        if (err) throw err;
        assert.equal(browser.location.pathname, '/session/new',
          'should be redirected to login screen');
        done();
      });
    });
```

在这里，我们正在验证，如果尝试在未登录的情况下加载待办事项创建表单，用户是否会被重定向到登录界面。

1.  如果用户已登录，我们将检查页面是否加载了一些预期的元素，例如标题和用于创建新待办事项的表单元素：

```js
    it('should load with title and form', login(function(browser, done) {
      browser.visit('http://localhost:3000/todos/new', function(err) {
        if (err) throw err;
        assert.equal(browser.text('h1'), 'New To-Do');

        var form = browser.query('form');
        assert(form, 'should have a form');
        assert.equal(form.method, 'POST', 'form should use post');
        assert.equal(form.action, '/todos', 'form should post to /todos');

        assert(browser.query('textarea[name=what]', form),
          'should have a what textarea input');
        assert(browser.query('input[type=submit]', form),
          'should have an input submit type');

        done();
      });
    }));
```

在这里，我们正在验证表单是否存在，它是否具有必要的属性来向`/todos` URL 发出`POST`请求，以及表单是否具有文本区输入和按钮。

1.  现在，我们还可以测试是否可以通过填写相应的表单并提交来成功创建待办事项：

```js
    it('should allow to create a todo', login(function(browser, done) {
      browser.visit('http://localhost:3000/todos/new', function(err) {
        if (err) throw err;

        browser
          .fill('What', 'Laundry')
          .pressButton('Create', function(err) {
            if (err) throw err;

            assert.equal(browser.location.pathname, '/todos',
              'should be redirected to /todos after creation');

            var list = browser.queryAll('#todo-list tr.todo');
            assert.equal(list.length, 1, 'To-do list length should be 1');
            var todo = list[0];
            assert.equal(browser.text('td.pos', todo), 1);
            assert.equal(browser.text('td.what', todo), 'Laundry');

            done();

          });
      });
    }));
```

在这里，我们最终要测试表单是否允许我们发布新项目，以及项目是否已创建。我们通过加载和填写待办事项创建表单来进行测试；验证我们已被重定向到待办事项列表页面；以及该页面是否包含我们刚刚创建的单个待办事项。

## 测试待办事项删除

现在我们已经测试了待办事项的插入，我们可以测试是否可以从列表中删除这些项目。我们将把这些测试放在一个名为`待办事项删除表单`的描述范围内，在其中我们将测试两件事：当只有一个待办事项存在时删除一个待办事项，以及当存在多个待办事项时删除一个待办事项。

### 注意

我们将这两个测试分开进行，因为先理解单个项目的测试，然后再进行更复杂的测试，以及分开测试我们是否在编程中常见的一次性错误。

以下是从一个项目列表中删除的代码：

```js
describe('Todo removal form', function() {

  describe('When one todo item exists', function() {

 beforeEach(function(done) {
 // insert one todo item
 db.insert(fixtures.todo, fixtures.user.email, done);
 });

    it("should allow you to remove", login(function(browser, done) {

      browser.visit('http://localhost:3000/todos', function(err, browser) {
        if (err) throw err;

        assert.equal(browser.queryAll('#todo-list tr.todo').length, 1);

        browser.pressButton('#todo-list tr.todo .remove form input[type=submit]',
          function(err) {
            if (err) throw err;
            assert.equal(browser.location.pathname, '/todos');
            // assert that all todos have been removed
            assert.equal(browser.queryAll('#todo-list tr').length, 0);
            done();
          }
        );

      });
    }));

  });
```

在运行测试之前，有一个`beforeEach`钩子，它会在测试用户的`todo`数据库中插入一个待办事项。这只是从`fixtures.todo`中取出的一个待办事项，这是我们需要添加到`test/fixtures.json`文件的属性：

```js
{
  "user" : {
    "email": "me@email.com",
    "password": "mypassword"
  },
 "todo": {
 "todos": [
 {
 "what": "Do the laundry",
 "created_at": 1346542066308
 }
 ]
 },
  "todos": {
    "todos": [
      {
        "what": "Do the laundry",
        "created_at": 1346542066308
      },
      {
        "what": "Call mom",
        "created_at": 1346542066308
      },
      {
        "what": "Go to gym",
        "created_at": 1346542066308
      }

    ]
  }

}
```

您可能会注意到，我们在这里利用机会添加一些额外的固定装置，这将有助于未来的测试。

继续分析测试代码，我们看到测试获取待办事项列表，然后验证待办事项的数量实际上是一个：

```js
assert.equal(browser.queryAll('#todo-list tr.todo').length, 1);
```

然后它继续尝试按下那个待办事项的移除按钮：

```js
browser.pressButton('#todo-list tr.todo .remove form input[type=submit]', …
```

选择器假设表格上有一个待办事项，我们之前已经验证过了。

### 注意

如果浏览器无法从给定的 CSS 选择器中找到按钮或提交元素，它将抛出错误，结束当前测试。

然后，在按下按钮并提交移除表单后，我们验证没有发生错误，浏览器被重定向回`/todos` URL，并且现在呈现的列表为空：

```js
assert.equal(browser.queryAll('#todo-list tr').length, 0);
```

现在我们已经测试了从一个一项列表中移除一项的工作情况，让我们创建一个更进化的测试，断言我们可以从三项列表中移除特定的项目：

```js
describe('When more than one todo item exists', function() {

  beforeEach(function(done) {
    // insert one todo item
    db.insert(fixtures.todos, fixtures.user.email, done);
  });

  it("should allow you to remove one todo item", login(
    function(browser, done) {

      browser.visit('http://localhost:3000/todos', function(err, browser) {
        if (err) throw err;

        var expectedList = [
          fixtures.todos.todos[0],
          fixtures.todos.todos[1],
          fixtures.todos.todos[2]
        ];

        var list = browser.queryAll('#todo-list tr');
        assert.equal(list.length, 3);

        list.forEach(function(todoRow, index) {
          assert.equal(browser.text('.pos', todoRow), index + 1);
          assert.equal(browser.text('.what', todoRow),
            expectedList[index].what);
        });

            browser.pressButton(
              '#todo-list tr:nth-child(2) .remove input[type=submit]',
              function(err) {
                if (err) throw err;

                assert.equal(browser.location.pathname, '/todos');

                // assert that the middle todo item has been removed
                var list = browser.queryAll('#todo-list tr');
                assert.equal(list.length, 2);

                // remove the middle element from the expected list
                expectedList.splice(1,1);

                // test that the rendered list is the expected list
                list.forEach(function(todoRow, index) {
                  assert.equal(browser.text('.pos', todoRow), index + 1);
                  assert.equal(browser.text('.what', todoRow),
                    expectedList[index].what);
                });

                done();
              }
            );

      });
    }
  ));

});
```

这个描述范围将与先前的描述范围处于同一级别，还会在`todo`数据库中插入一个文档，但这次文档包含了一个包含三个待办事项的列表，取自`fixtures.todos`属性（而不是先前使用的单数`fixtures.todo`属性）。

测试从访问`todo`列表页面开始，并构建预期待办事项列表，存储在名为`expectedList`的变量中。然后我们检索在 HTML 文档中找到的所有待办事项，并验证内容是否符合预期：

```js
list.forEach(function(todoRow, index) {
  assert.equal(browser.text('.pos', todoRow), index + 1);
  assert.equal(browser.text('.what', todoRow),
    expectedList[index].what);
});
```

一旦我们验证了所有预期的待办事项都已经就位并且顺序正确，我们继续通过以下代码点击列表中第二个项目的按钮：

```js
browser.pressButton(
  '#todo-list tr:nth-child(2) .remove input[type=submit]', ...
```

在这里，我们使用特殊的 CSS 选择器`nth-child`来选择第二个待办事项的行，然后获取其中用于移除提交按钮的代码，并最终按下它。

一旦按钮被按下，表单就会被提交，浏览器会回调，我们验证没有错误，我们被重定向回`/todos` URL，并且它包含了预期的列表。我们通过从先前使用的`expectedList`数组中移除第二个元素来做到这一点，并验证这正是当前页面显示的内容：

```js
var list = browser.queryAll('#todo-list tr');
assert.equal(list.length, 2);
expectedList.splice(1,1);

// test that the rendered list is the expected list
list.forEach(function(todoRow, index) {
  assert.equal(browser.text('.pos', todoRow), index + 1);
  assert.equal(browser.text('.what', todoRow),
    expectedList[index].what);
});
```

# 把所有东西放在一起

您可以手动逐个运行测试，但应该能够一次运行它们全部。为此，您只需要从 shell 命令行中调用：

```js
$ ./node_modules/.bin/mocha test/users.js test/session.js test/todos.js
```

现在我们需要更改`package.json`，以便您可以告诉**node package** **manager** (**npm**)如何运行测试：

```js
{
  "description": "To-do App",
  "version": "0.0.0",
  "private": true,
  "dependencies": {
    "union": "0.3.0",
    "flatiron": "0.2.8",
    "plates": "0.4.x",
    "node-static": "0.6.0",
    "nano": "3.3.0",
    "flatware-cookie-parser": "0.1.x",
    "flatware-session": "0.1.x"
  },
  "devDependencies": {
    "mocha": "1.4.x",
    "zombie": "1.4.x"
  },
  "scripts": {
    "test": "mocha test/users.js test/session.js test/todos.js",
    "start": "node app.js"
  },
  "name": "todo",
  "author": "Pedro",
  "homepage": ""
}
```

现在您可以使用以下命令运行您的测试：

```js
$ npm test
  .............

  ✔ 13 tests complete (3758ms)
```

# 摘要

Zombie.js 允许我们访问 URL，加载 HTML 文档，并使用 CSS 选择器检索 HTML 元素。它还允许我们轻松填写表单并提交它们，点击按钮并跟随链接，验证返回状态代码，并使用简洁方便的 API 以相同的方式分析响应文档。
