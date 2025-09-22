# 单元测试和功能测试

单元测试已成为良好软件开发实践的重要组成部分。这是一种通过测试源代码的各个单元来确保其正确功能的方法。每个单元在理论上都是应用程序中最小的可测试部分。在一个 Node.js 应用程序中，你可能将每个模块视为一个单元。

在单元测试中，每个单元都是单独测试的，尽可能地将测试单元与其他应用程序部分隔离。如果测试失败，你希望它是因为你的代码中的错误，而不是你代码所使用的包中的错误。一个常见的技巧是使用模拟对象或模拟数据来隔离应用程序的各个部分。

与此相反，功能测试并不试图测试单个组件，而是测试整个系统。一般来说，单元测试由开发团队执行，而功能测试由质量保证（**QA**）或质量工程（**QE**）团队执行。这两种测试模型都是完全认证应用程序所必需的。一个类比可能是，单元测试类似于确保句子中的每个单词都拼写正确，而功能测试则确保包含该句子的段落具有良好的结构。

在本章中，我们将涵盖：

+   断言作为软件测试的基础

+   Mocha 单元测试框架和 Chai 断言库

+   使用测试来查找错误并修复错误

+   使用 Docker 管理测试基础设施

+   测试 REST 后端服务

+   使用 Puppeteer 在真实网页浏览器中进行 UI 测试

+   使用元素 ID 属性提高 UI 可测试性

# 断言 – 测试方法的基础

Node.js 有一个有用的内置测试工具，即 `assert` 模块。它的功能与其他语言的 `assert` 库类似。也就是说，它是一组用于测试条件的函数，如果条件指示有错误，则 `assert` 函数会抛出异常。

最简单的情况下，测试套件是一系列 `assert` 调用来验证被测试事物的行为。例如，测试套件可以实例化用户身份验证服务，然后进行 API 调用，使用 `assert` 方法验证结果，然后进行另一个 API 调用，验证其结果，依此类推。

考虑一个代码片段，例如，你可以将其保存为名为 `deleteFile.js` 的文件：

```js
const fs = require('fs'); 

exports.deleteFile = function(fname, callback) { 
  fs.stat(fname, (err, stats) => { 
    if (err) callback(new Error(`the file ${fname} does not exist`)); 
    else { 
      fs.unlink(fname, err2 => { 
        if (err) callback(new Error(`could not delete ${fname}`)); 
        else callback(); 
      }); 
    } 
  }); 
}; 
```

首先要注意的是，这里包含了几层异步回调函数。这带来了一些挑战：

+   从回调函数的深处捕获错误，以确保测试场景失败

+   检测回调函数从未被调用的情况

以下是一个使用 `assert` 进行测试的示例。创建一个名为 `test-deleteFile.js` 的文件，包含以下内容：

```js
const fs = require('fs'); 
const assert = require('assert'); 
const df = require('./deleteFile'); 

df.deleteFile("no-such-file", (err) => { 
    assert.throws( 
        function() { if (err) throw err; }, 
        function(error) { 
            if ((error instanceof Error) 
             && /does not exist/.test(error)) { 
               return true; 
            } else return false; 
        }, 
        "unexpected error" 
    ); 
}); 
```

这被称为负测试场景，因为它测试的是请求删除一个不存在的文件是否会抛出错误。

如果你正在寻找一种快速测试的方法，当这样使用时，`assert` 模块可能很有用。如果它运行并且没有打印出消息，那么测试就通过了。但是，它是否捕捉到了 `deleteFile` 回调从未被调用的情况？

```js
$ node test-deleteFile.js 
```

`assert` 模块被许多测试框架用作编写测试用例的核心工具。测试框架所做的是创建一个熟悉的测试套件和测试用例结构来封装你的测试代码。

Node.js 中有众多断言库的风格。在本章的后面部分，我们将使用 Chai 断言库 ([`chaijs.com/`](http://chaijs.com/))，它为你提供了三种不同的断言风格（should、expect 和 assert）的选择。

# 测试 Notes 模型

让我们从为 Notes 应用程序编写的数据模型开始我们的单元测试之旅。因为这是单元测试，模型应该与 Notes 应用程序的其余部分分开进行测试。

在 Notes 模型的大多数情况下，隔离它们的依赖意味着创建一个模拟数据库。你是要测试数据模型还是底层数据库？模拟数据库意味着创建一个假的数据库实现，这并不像是我们时间的高效利用。你可以争论说，测试数据模型实际上是在测试你的代码与数据库之间的交互，模拟数据库意味着没有测试这种交互，因此我们应该针对生产中使用的数据库引擎测试我们的代码。

基于这样的推理，我们将跳过模拟数据库，而是对包含测试数据的数据库运行测试。为了简化测试数据库的启动，我们将使用 Docker 启动和停止一个为测试设置的 Notes 应用程序堆栈版本。

# Mocha 和 Chai – 选择的测试工具

如果你还没有这样做，复制源树以用于本章。例如，如果你有一个名为 `chap10` 的目录，创建一个名为 `chap11` 的目录，其中包含 `chap10` 中的所有内容。

在 `notes` 目录下，创建一个名为 `test` 的新目录。

Mocha ([`mochajs.org/`](http://mochajs.org/)) 是许多适用于 Node.js 的测试框架之一。正如你很快就会看到的，它帮助我们编写测试用例和测试套件，并提供测试结果报告机制。它之所以被选中，是因为它支持 Promises。它与前面提到的 Chai 断言库非常契合。此外，我们需要使用 ES6 模块从 CommonJS 编写的测试套件中，因此我们必须使用 `esm` 模块。

你可能会发现对早期 `@std/esm` 模块的引用。该模块已被弃用，`esm` 取而代之。

当你在 `notes/test` 目录中时，输入以下命令来安装 Mocha、Chai 和 `esm`：

```js
$ npm init
... answer the questions to create package.json
$ npm install mocha@5.x chai@4.1.x esm --save
```

# Notes 模型测试套件

由于我们有多个 Notes 模型，测试套件应该针对任何模型运行。我们可以使用我们开发的 Notes 模型 API 编写测试，并使用环境变量来声明要测试的模型。

由于我们使用 ES6 模块编写了 Notes 应用程序，我们面临一个小挑战。Mocha 只支持在 CommonJS 模块中运行测试，而 Node.js（截至本文写作时）不支持从 CommonJS 模块加载 ES6 模块。ES6 模块可以使用`import`来加载 CommonJS 模块，但 CommonJS 模块不能使用`require`来加载 ES6 模块。背后有各种技术原因，但最终结果是我们在这种限制下。

由于 Mocha 要求测试必须是 CommonJS 模块，我们处于必须将 ES6 模块加载到 CommonJS 模块中的位置。存在一个名为`esm`的模块，它允许这种组合工作。如果你回顾一下，我们在上一节中安装了该模块。让我们看看如何使用它。

在`test`目录中，创建一个名为`test-model.js`的文件，包含以下内容作为测试套件的顶层结构：

```js
'use strict'; 

require = require("esm")(module,{"esm":"js"});
const assert = require('chai').assert; 
const model = require('../models/notes');

describe("Model Test", function() { 
  .. 
}); 
```

通过这里显示的`require('esm')`语句启用了加载 ES6 模块的支持。它用`esm`模块中的一个替换了标准的`require`函数。末尾的参数列表启用了在 CommonJS 模块中加载 ES6 模块的功能。一旦这样做，你的 CommonJS 模块就可以通过`require('../models/notes')`在几行之后加载一个 ES6 模块。

Chai 库支持三种断言风格。我们在这里使用的是`assert`风格，但如果你更喜欢其他风格，也很容易切换。有关 Chai 支持的其他风格，请参阅[`chaijs.com/guide/styles/`](http://chaijs.com/guide/styles/)。

Chai 的断言包括一个非常长的有用断言函数列表，请参阅[`chaijs.com/api/assert/`](http://chaijs.com/api/assert/)。

要测试的 Notes 模型必须通过`NOTES_MODEL`环境变量来选择。对于也咨询环境变量的模型，我们还需要提供该配置。

使用 Mocha，测试套件包含在一个`describe`块中。第一个参数是描述性文本，你用它来定制测试结果的展示。

而不是维护一个单独的测试数据库，我们可以在执行测试时动态创建一个。Mocha 有所谓的钩子，这些是在测试用例执行前后执行的函数。钩子函数让你，测试套件的作者，设置和撤销测试套件操作所需的条件。例如，为了创建一个包含已知测试内容的测试数据库：

```js
describe("Model Test", function() { 
  beforeEach(async function() {
    try {
      const keyz = await model.keylist();
      for (let key of keyz) {
        await model.destroy(key);
      }
      await model.create("n1", "Note 1", "Note 1");
      await model.create("n2", "Note 2", "Note 2");
      await model.create("n3", "Note 3", "Note 3");
    } catch (e) {
      console.error(e);
      throw e;
    }
  });
    .. 
}); 
```

这定义了一个`beforeEach`钩子，它在每个测试用例之前执行。其他的钩子包括`before`、`after`、`beforeEach`和`afterEach`。每个钩子在测试用例执行前后被触发。

这意味着它是一个在每个测试之前的清理/准备步骤。它使用我们的笔记 API 首先从数据库中删除所有笔记（如果有），然后创建一组具有已知特征的新的笔记。这种技术通过确保我们有已知条件来测试，从而简化了测试。

我们还有一个副作用是测试 `model.keylist` 和 `model.create` 方法。

在 Mocha 中，一系列测试用例被封装在一个 `describe` 块中，并使用 `it` 块编写。`describe` 块的目的是描述那一组测试，而 `it` 块是用来检查被测试事物的特定方面的断言。你可以根据需要将 `describe` 块嵌套得尽可能深：

```js
  describe("check keylist", function() {
    it("should have three entries", async function() {
      const keyz = await model.keylist();
      assert.exists(keyz);
      assert.isArray(keyz);
      assert.lengthOf(keyz, 3);
    });
    it("should have keys n1 n2 n3", async function() {
      const keyz = await model.keylist();
      assert.exists(keyz);
      assert.isArray(keyz);
      assert.lengthOf(keyz, 3);
      for (let key of keyz) {
        assert.match(key, /n[123]/, "correct key");
      }
    });
    it("should have titles Node #", async function() {
      const keyz = await model.keylist();
      assert.exists(keyz);
      assert.isArray(keyz);
      assert.lengthOf(keyz, 3);
      var keyPromises = keyz.map(key => model.read(key));
      const notez = await Promise.all(keyPromises);
      for (let note of notez) {
        assert.match(note.title, /Note [123]/, "correct title");
      }
    });
  });
```

策略是调用笔记 API 函数，然后测试结果以检查它们是否与预期结果匹配。

这个 `describe` 块位于外部的 `describe` 块内。`describe` 和 `it` 块中给出的描述用于使测试报告更易于阅读。`it` 块形成了一个类似于 *it (被测试的事物) 应该这样做或那样做* 的伪句子。

在 Mocha 中，重要的是不要在 `describe` 和 `it` 块中使用箭头函数。到现在为止，你可能会因为箭头函数的易用性而喜欢它们。但是，Mocha 使用一个包含对 Mocha 有用函数的 `this` 对象来调用这些函数。因为箭头函数避免了设置 `this` 对象，所以 Mocha 会崩溃。

即使 Mocha 需要 `describe` 和 `it` 块中的常规函数，我们也可以在这些函数中使用箭头函数。

Mocha 如何知道测试代码是否通过？它是如何知道测试何时完成的？这段代码展示了三种方法之一。

通常，Mocha 寻找函数是否抛出异常，或者测试用例是否执行时间过长（超时情况）。在任何情况下，Mocha 都会指示测试失败。当然，对于非异步代码来说，这很简单就能确定。但是，Node.js 全是关于异步代码的，Mocha 有两种异步代码测试模型。

在第一个（此处未展示）中，Mocha 传入一个回调函数，测试代码是调用这个回调函数。在第二个，如这里所示，它寻找测试函数返回的 Promise，并确定是否通过取决于 Promise 是否处于 *resolve* 或 *reject* 状态。

在这种情况下，我们使用 `async` 函数，因为它们会自动返回一个 Promise。在函数内部，我们使用 `await` 调用异步函数，确保任何抛出的异常都会被指示为拒绝的 Promise。

另一个需要注意的事项是之前提出的问题：如果我们正在测试的回调函数从未被调用怎么办？或者，如果 Promise 从未解决怎么办？Mocha 启动一个计时器，如果测试用例在计时器到期之前没有完成，Mocha 会失败该测试用例。

# 配置和运行测试

我们还有更多的测试要写，但让我们首先设置好运行测试的环境。最简单的模型要测试的是内存模型。让我们将其添加到`notes/test/package.json`的`scripts`部分：

```js
"test-notes-memory": "NOTES_MODEL=memory mocha test-model",
```

要安装依赖项，我们必须在`notes/test`和`notes`目录中运行`npm install`。这样，测试代码的依赖项和 Notes 的依赖项都安装在了正确的位置。

然后，我们可以按照以下方式运行它：

```js
$ npm run test-notes-memory

> notes-test@1.0.0 test-notes-memory /Users/david/chap11/notes/test
> NOTES_MODEL=memory mocha test-model

 Model Test
 check keylist
 √ should have three entries
 √ should have keys n1 n2 n3
 √ should have titles Node #

 3 passing (18ms)
```

使用`mocha`命令运行测试套件。

输出的结构遵循`describe`和`it`块的结构。你应该设置描述性文本字符串，使其读起来更舒服。

# Notes 模型的更多测试

这还不够测试很多，所以让我们继续添加一些更多的测试：

```js
describe("read note", function() {
    it("should have proper note", async function() {
        const note = await model.read("n1");
        assert.exists(note);
        assert.deepEqual({ key: note.key, title: note.title, body: 
        note.body }, {
          key: "n1", title: "Note 1 FAIL", body: "Note 1"
        });
    });

    it("Unknown note should fail", async function() {
        try {
          const note = await model.read("badkey12");
          assert.notExists(note);
          throw new Error("should not get here");
        } catch(err) {
          // this is expected, so do not indicate error
          assert.notEqual(err.message, "should not get here");
        }
    });
});

describe("change note", function() {
    it("after a successful model.update", async function() {
        const newnote = await model.update("n1", "Note 1 title 
        changed", "Note 1 body changed");
        const note = await model.read("n1");
        assert.exists(note);
        assert.deepEqual({ key: note.key, title: note.title, body: 
        note.body }, {
          key: "n1", title: "Note 1 title changed", body: "Note 1 body 
        changed"
        });
    });
});

describe("destroy note", function() {
    it("should remove note", async function() {
        await model.destroy("n1");
        const keyz = await model.keylist();
        assert.exists(keyz);
        assert.isArray(keyz);
        assert.lengthOf(keyz, 2);
        for (let key of keyz) {
          assert.match(key, /n[23]/, "correct key");
        }
    });
    it("should fail to remove unknown note", async function() {
        try {
          await model.destroy("badkey12");
          throw new Error("should not get here");
        } catch(err) {
            // this is expected, so do not indicate error
            assert.notEqual(err.message, "should not get here");
        }
    });
  });

  after(function() {  model.close(); });
}); 
```

注意，对于负测试——如果抛出错误则测试通过——我们将其运行在`try/catch`块中。每个情况中的`throw new Error`行不应该执行，因为前面的代码应该抛出错误。因此，我们可以检查抛出的错误中的消息是否是接收到的消息，如果是这种情况，则测试失败。

现在，测试报告：

```js
$ npm run test-notes-memory

> notes-test@1.0.0 test-notes-memory /Users/david/chap11/notes/test
> NOTES_MODEL=memory mocha test-model

  Model Test
    check keylist
      √ should have three entries
      √ should have keys n1 n2 n3
      √ should have titles Node #
    read note
      √ should have proper note
      √ Unknown note should fail
    change note
      √ after a successful model.update
    destroy note
      √ should remove note
      √ should fail to remove unknown note

  8 passing (17ms) 
```

在这些附加测试中，我们有一些负测试。在每个我们期望失败的测试中，我们提供一个我们知道不在数据库中的`notekey`，然后确保模型给我们一个错误。

Chai 断言 API 包括一些非常表达式的断言。在这种情况下，我们使用了`deepEqual`方法，它对两个对象进行深度比较。在我们的情况下，它看起来像这样：

```js
assert.deepEqual({ key: note.key, title: note.title, body: note.body }, {
   key: "n1", title: "Note 1", body: "Note 1"
 });
```

这在测试代码中读起来很舒服，但更重要的是，报告的测试失败看起来非常漂亮。由于这些目前都在通过，尝试通过更改其中一个预期值字符串来引入一个错误。重新运行测试后，你会看到：

```js
 Model Test 
 check keylist 
 √ should have three entries 
 √ should have keys n1 n2 n3 
 √ should have titles Node # 
 read note 
 1) should have proper note 
 √ Unknown note should fail 
 change note 
 √ after a successful model.update 
 destroy note 
 √ should remove note 
 √ should fail to remove unknown note 

 7 passing (42ms) 
 1 failing 

 1) Model Test 
 read note 
 should have proper note: 
 AssertionError: expected { Object (key, title, ...) } to deeply 
 equal { Object (key, title, ...) } 
 + expected - actual 

 { 
 "body": "Note 1" 
 "key": "n1" 
 -  "title": "Note 1" 
 +  "title": "Note 1 FAIL" 
 } 

 at Context.<anonymous> (test-model.js:53:16) 
 at <anonymous> 

```

最上面是每个测试案例的状态报告。对于某个测试，不是用勾号表示，而是用数字，这个数字对应于底部报告的详细信息。当使用`spec`报告器时，Mocha 就是这样呈现测试失败的。Mocha 支持其他测试报告格式，其中一些可以产生可以发送到测试状态报告系统的数据。有关更多信息，请参阅[`mochajs.org/#reporters`](https://mochajs.org/#reporters)。

在这种情况下，失败是通过`deepEqual`方法检测到的，它以这种方式呈现检测到的对象不等式。

# 测试数据库模型

这很好，但我们显然不会在生产中使用内存中的 Notes 模型。这意味着我们需要测试所有其他模型。

测试 LevelUP 和文件系统模型很容易，只需将其添加到`package.json`的脚本部分：

```js
"test-notes-levelup": "NOTES_MODEL=levelup mocha",
"test-notes-fs": "NOTES_MODEL=fs mocha", 
```

然后运行以下命令：

```js
$ npm run test-notes-fs 
$ npm run test-notes-levelup 
```

这将产生一个成功的测试结果。

最简单的数据库来测试是 SQLite3，因为它不需要任何设置。我们有两个 SQLite3 模型要测试，让我们从`notes-sqlite3.js`开始。将以下内容添加到`package.json`的脚本部分：

```js
"test-notes-sqlite3": "rm -f chap11.sqlite3 && sqlite3 chap11.sqlite3 --init ../models/chap07.sql </dev/null && NOTES_MODEL=sqlite3 SQLITE_FILE=chap11.sqlite3 mocha test-model", 
```

这个命令序列将测试数据库放入`chap11.sqlite3`文件。它首先使用`sqlite3`命令行工具初始化该数据库。请注意，我们已经将其输入连接到`/dev/null`，因为否则`sqlite3`命令会提示输入。然后，它运行测试套件，传递运行针对 SQLite3 模型所需的环境变量。

运行测试套件确实找到了两个错误：

```js
$ npm run test-notes-sqlite3 

> notes-test@1.0.0 test-notes-sqlite3 /Users/david/chap11/notes/test 
> rm -f chap11.sqlite3 && sqlite3 chap11.sqlite3 --init ../models/chap07.sql </dev/null && NOTES_MODEL=sqlite3 SQLITE_FILE=chap11.sqlite3 mocha test-model 

  Model Test 
    check keylist 
      √ should have three entries 
      √ should have keys n1 n2 n3 
      √ should have titles Node # 
    read note 
      √ should have proper note 
      1) Unknown note should fail 
    change note 
      √ after a successful model.update (114ms) 
    destroy note 
      √ should remove note (103ms) 
      2) should fail to remove unknown note 

  6 passing (6s) 
  2 failing 

  1) Model Test 
       read note 
         Unknown note should fail: 
     Uncaught TypeError: Cannot read property 'notekey' of undefined 
      at Statement.db.get (/home/david/nodewebdev/node-web-development-
  code-4th-edition/chap11/notes/models/notes-sqlite3.mjs:64:39)                               

  2) Model Test 
       destroy note 
         should fail to remove unknown note: 

      AssertionError: expected 'should not get here' to not equal 
  'should not get here' 
      + expected - actual 
```

失败的测试调用`model.read("badkey12")`，一个我们知道不存在的`key`。编写负面测试是值得的。在`models/notes-sqlite3.mjs`（第 64 行）中失败的代码行如下：

```js
const note = new Note(row.notekey, row.title, row.body);
```

在这之前插入`console.log(util.inspect(row));`非常简单，可以了解到，对于失败的调用，SQLite3 为我们提供了`undefined`的`row`，解释了错误信息。

测试套件多次调用`read`函数，使用一个确实存在的`notekey`值。显然，当提供一个无效的`notekey`值时，查询会返回一个空的结果集，SQLite3 会调用回调函数，同时传递`undefined`错误和`undefined`的行值。这是数据库模块的常见行为。空结果集不是错误，因此我们没有收到错误，也没有`undefined`的`row`。

实际上，我们在`models/notes-sequelize.mjs`中看到了这种行为。`models/notes-sequelize.mjs`中的等效代码做了正确的事情，并且有一个检查，我们可以将其改编。让我们将`models/notes-sqlite.mjs`中的`read`函数重写为以下内容：

```js
export async function read(key) {
  var db = await connectDB();
  var note = await new Promise((resolve, reject) => {
    db.get("SELECT * FROM notes WHERE notekey = ?", [ key ], (err, row) 
    => {
        if (err) return reject(err);
        if (!row) { reject(new Error(`No note found for ${key}`)); } 
        else {
            const note = new Note(row.notekey, row.title, row.body);
            resolve(note);
        }
    });
  });
  return note;
}
```

这很简单，我们只需检查`row`是否为`undefined`，如果是，就抛出一个错误。虽然数据库不会将空结果集视为错误，但 Notes 会。此外，Notes 已经知道如何处理这种情况下的抛出错误。进行这个更改，特定的测试用例就通过了。

在`destroy`逻辑中还有一个类似的错误。测试销毁一个不存在的笔记时，在这一行没有产生错误：

```js
await model.destroy("badkey12");
```

如果我们检查其他模型，它们对于不存在的键会抛出错误。在 SQL 中，如果这个 SQL（来自`models/notes-sqlite3.mjs`）没有删除任何东西，显然这不是一个错误：

```js
db.run("DELETE FROM notes WHERE notekey = ?;", ... );
```

不幸的是，没有 SQL 选项可以在不删除任何记录时使这个 SQL 语句失败。因此，我们必须添加一个检查来查看记录是否存在。具体来说：

```js
export async function destroy(key) {
    const db = await connectDB();
    const note = await read(key);
    return await new Promise((resolve, reject) => {
        db.run("DELETE FROM notes WHERE notekey = ?;", [ key ], err => 
       {
            if (err) return reject(err);
            resolve();
        });
    });
}
```

因此，我们读取了笔记，作为副产品我们也验证了笔记的存在。如果笔记不存在，`read`将抛出一个错误，并且`DELETE`操作甚至不会运行。

这些是我们提到的第七章*数据存储和检索*中的错误。我们只是忘记在这个特定模型中检查这些条件。幸运的是，我们勤奋的测试捕捉到了这个问题。至少，这是我们要告诉经理们的故事，而不是告诉他们我们忘记检查我们已经知道可能发生的事情。

现在我们已经修复了`models/notes-sqlite3.mjs`，让我们也使用 SQLite3 数据库测试`models/notes-sequelize.mjs`。为此，我们需要一个连接对象来在`SEQUELIZE_CONNECT`变量中指定。虽然我们可以重用现有的一个，但让我们创建一个新的。创建一个名为`test/sequelize-sqlite.yaml`的文件，包含以下内容：

```js
dbname: notestest 
username: 
password: 
params: 
    dialect: sqlite 
    storage: notestest-sequelize.sqlite3 
    logging: false 
```

这样做，我们不会用我们的测试套件覆盖生产数据库实例。由于测试套件会销毁它测试的数据库，它必须运行在我们愿意销毁的数据库上。日志参数关闭了`Sequelize`产生的庞大输出，这样我们就可以阅读测试结果报告。

将以下内容添加到`package.json`的脚本部分：

```js
"test-notes-sequelize-sqlite": "NOTES_MODEL=sequelize SEQUELIZE_CONNECT=sequelize-sqlite.yaml mocha test-model" 
```

然后运行测试套件：

```js
$ npm run test-notes-sequelize-sqlite 
.. 
 8 passing (2s) 
```

我们轻松过关！我们已经能够利用相同的测试套件针对多个笔记模型进行测试。我们甚至在其中一个模型中发现了两个错误。但是，我们还有两个测试配置需要测试。

我们测试结果矩阵如下：

+   `models-fs`: 通过

+   `models-memory`: 通过

+   `models-levelup`: 通过

+   `models-sqlite3`: 2 次失败，现已修复

+   `models-sequelize`: 使用 SQLite3：通过

+   `models-sequelize`: 使用 MySQL：未测试

+   `models-mongodb`: 未测试

这两个未测试的模型都需要设置数据库服务器。我们避免测试这些组合，但我们的经理不会接受这个借口，因为 CEO 需要知道我们已经完成了测试周期。笔记必须在类似生产环境的配置中进行测试。

在生产中，我们当然会使用常规数据库服务器，MySQL 或 MongoDB 是首选。因此，我们需要一种低开销的方式来运行对这些数据库的测试。对生产配置的测试必须非常容易，以至于我们不应该有任何阻力，以确保测试经常运行，以达到预期的效果。

幸运的是，我们已经有了一种支持轻松创建和销毁部署基础设施技术的经验。你好，Docker！

# 使用 Docker 管理测试基础设施

Docker 给我们带来的一个优点是能够在我们的笔记本电脑上安装生产环境。然后，将相同的 Docker 设置推送到云托管环境进行预发布或生产部署就变得非常容易。

在本节中，我们将演示重用之前为测试基础设施定义的 Docker Compose 配置，并使用 shell 脚本自动化在容器内执行笔记测试套件。一般来说，在运行测试时复制生产环境是很重要的。Docker 可以使这一点变得容易实现。

使用 Docker，我们将能够轻松地对数据库进行测试，并有一个简单的方法来启动和停止生产环境的测试版本。让我们开始吧。

# 使用 Docker Compose 编排测试基础设施

我们在使用 Docker Compose 进行笔记应用程序部署编排时获得了很好的体验。整个系统，包含四个独立的服务，可以很容易地在 `compose/docker-compose.yml` 中描述。我们将要做的是复制 Compose 文件，然后进行一些必要的更改以支持测试执行。

让我们从创建一个新的目录 `test-compose` 开始，作为 `notes`、`users` 和 `compose` 目录的兄弟目录。将 `compose/docker-compose.yml` 复制到新创建的 `test-compose` 目录。我们将对这个文件进行一些更改，并对现有的 Dockerfile 进行一些小的更改。

我们希望更改容器和网络名称，以便我们的测试基础设施不会破坏生产基础设施。我们将不断删除并重新创建测试容器，为了使开发者满意，我们将保持开发基础设施不变，并在单独的基础设施上执行测试。通过维护独立的测试容器和网络，我们的测试脚本可以随心所欲地执行，而不会干扰开发或生产容器。

考虑对 `db-auth` 和 `db-notes` 容器进行的以下更改：

```js
db-userauth-test:
  build: ../authnet
  container_name: db-userauth-test
  networks:
    - authnet-test
  environment:
    MYSQL_RANDOM_ROOT_PASSWORD: "true"
    MYSQL_USER: userauth-test
    MYSQL_PASSWORD: userauth-test
    MYSQL_DATABASE: userauth-test
  volumes:
    - db-userauth-test-data:/var/lib/mysql
  restart: always
.. 
db-notes-test:
  build: ../frontnet
  container_name: db-notes-test
  networks:
    - frontnet-test
  environment:
    MYSQL_RANDOM_ROOT_PASSWORD: "true"
    MYSQL_USER: notes-test
    MYSQL_PASSWORD: notes12345
    MYSQL_DATABASE: notes-test
  volumes:
    - db-notes-test-data:/var/lib/mysql
  restart: always
```

这与之前相同，但容器和网络名称后添加了 `-test`。

这是第一个必须做出的更改，在 `test-compose/docker-compose.yml` 中的每个容器和网络名称后添加 `-test`。我们将进行的所有测试都将运行在完全独立的容器、主机名和网络中，与开发实例的完全不同。

这个更改将影响 `notes-test` 和 `userauth-test` 服务，因为数据库服务器的主机名现在是 `db-auth-test` 和 `db-notest-test`。有几个环境变量或配置文件需要更新。

另一个考虑因素是配置服务所需的环境变量。之前，我们在 Dockerfile 中定义了所有环境变量。重用这些 Dockerfile 非常有用，因为我们知道我们正在测试与生产中使用的相同的部署。但我们需要调整配置设置以匹配测试基础设施。

这里显示的数据库配置是一个示例。我们使用相同的 Dockerfile，但我们还在 `test-compose/docker-compose.yml` 中定义了环境变量。正如你所期望的，这会覆盖 Dockerfile 中的环境变量，使用这里设置的值：

```js
userauth-test:
  build: ../users
  container_name: userauth-test
  depends_on:
    - db-userauth-test
  networks:
    - authnet-test
    - frontnet-test
  environment:
    DEBUG: ""
    NODE_ENV: "test"
    SEQUELIZE_CONNECT: "sequelize-docker-test-mysql.yaml"
    HOST_USERS_TEST: "localhost"
  restart: always
  volumes:
    - ./reports-userauth:/reports
.. 
notes-test:
  build: ../notes
  container_name: notes-test
  depends_on:
    - db-notes-test
  networks:
    - frontnet-test
  ports:
    - "3000:3000"
  restart: always
  environment:
    NODE_ENV: "test"
    SEQUELIZE_CONNECT: "test/sequelize-mysql.yaml"
    USER_SERVICE_URL: "http://userauth-test:3333"
  volumes:
    - ./reports-notes:/reports
...
networks:
  frontnet-test:
    driver: bridge
  authnet-test:
    driver: bridge

volumes: 
  db-userauth-test-data: 
  db-notes-test-data: 
```

再次，我们将容器和网络名称更改为添加 `-test`。我们将一些环境变量从 Dockerfile 移动到 `test-compose/docker-compose.yml`。最后，我们在容器内挂载了一些数据卷。

另一件事是要设置目录来存储测试代码。在 Node.js 项目中，一个常见的做法是将测试代码放在与应用程序代码相同的目录中。在本章的早期，我们就是这样做的，在`notes/test`目录中实现了一个小的测试套件。目前，`notes/Dockerfile`没有将这个目录复制到容器中。测试代码必须存在于容器中才能执行测试。另一个问题是，不部署测试代码在生产环境中是有帮助的。

我们可以确保`test-compose/docker-compose.yml`将`notes/test`挂载到容器中：

```js
notes-test:
  ...
  volumes:
    - ./reports-notes:/reports
    - ../notes/test:/notesapp/test
```

这让我们得到了两者的最佳结合。

+   测试代码位于`notes/test`中，这是它应该所在的位置

+   测试代码不会被复制到生产容器中

+   在测试模式下，`test`目录出现在它应该出现的地方

我们还有一些配置文件剩余，用于设置`Sequelize`数据库连接。

对于`userauth-test`容器，`SEQUELIZE_CONNECT`变量现在指向一个不存在的配置文件，这是由于在`user/Dockerfile`中覆盖了变量。让我们创建该文件，命名为`test-compose/userauth/sequelize-docker-mysql.yaml`，包含以下内容：

```js
dbname: userauth-test
username: userauth-test
password: userauth-test
params:
    host: db-userauth-test
    port: 3306
    dialect: mysql
```

这些值与传递给`db-userauth-test`容器的变量相匹配。然后我们必须确保此配置文件被挂载到`userauth-test`容器中：

```js
userauth-test:
  ...
  volumes:
    - ./reports-userauth:/reports
    - ./userauth/sequelize-docker-test-mysql.yaml:/userauth/sequelize-
  docker-test-mysql.yaml
```

对于`notes-test`，我们有一个配置文件，`test/sequelize-mysql.yaml`，需要放入`notes/test`目录：

```js
dbname: notes-test 
username: notes-test
password: notes12345
params: 
    host: db-notes-test
    port: 3306 
    dialect: mysql 
    logging: false 
```

再次，这匹配了`db-notes-test`中的配置变量。在`test-compose/docker-compose.yml`中，我们将该文件挂载到容器中。

# 在 Docker Compose 下执行测试

现在，我们准备好在容器内执行一些测试了。我们使用 Docker Compose 文件描述了 Notes 应用程序的测试环境，使用与生产环境相同的架构。测试脚本和配置已注入到容器中。问题是，我们如何自动化测试执行？

我们将使用的技术是运行一个 shell 脚本，并使用`docker exec -it`来执行命令以运行测试脚本。这有点自动化，经过更多的工作，它可以完全自动化。

在`test-compose`中，让我们创建一个名为`run.sh`的 shell 脚本（在 Windows 上为`run.ps1`）：

```js
docker-compose stop

docker-compose build
docker-compose up --force-recreate -d
docker ps
docker network ls

sleep 20
docker exec -it --workdir /notesapp/test -e DEBUG= notes-test npm install

docker exec -it --workdir /notesapp/test -e DEBUG= notes-test npm run test-notes-memory
docker exec -it --workdir /notesapp/test -e DEBUG= notes-test npm run test-notes-fs
docker exec -it --workdir /notesapp/test -e DEBUG= notes-test npm run test-notes-levelup
docker exec -it --workdir /notesapp/test -e DEBUG= notes-test npm run test-notes-sqlite3
docker exec -it --workdir /notesapp/test -e DEBUG= notes-test npm run test-notes-sequelize-sqlite
docker exec -it --workdir /notesapp/test -e DEBUG= notes-test npm run test-notes-sequelize-mysql

docker-compose stop 
```

在持续集成系统（如 Jenkins）中运行测试是一种常见的做法。持续集成系统会自动对软件产品进行构建或测试。构建和测试结果数据用于自动生成状态页面。访问[`jenkins.io/index.html`](https://jenkins.io/index.html)，这是一个 Jenkins 作业的良好起点。

这是我们构建容器的第一步，然后是启动它们。脚本暂停几秒钟，以便容器有时间完全实例化自己。

后续的所有命令都遵循一个特定的模式，这是需要理解的重要点。由于`--workdir`选项，命令在`/notesapp/test`目录下执行。请记住，该目录是通过 Docker Compose 文件注入到容器中的。

使用`-e DEBUG=`我们已禁用`DEBUG`选项。如果设置了这些选项，测试结果中会有过多的不必要输出，所以使用此选项确保不会出现调试输出。

现在您已经了解了选项，您可以看到后续的所有命令都是在`test`目录下使用该目录中的`package.json`执行的。它首先运行`npm install`，然后运行测试矩阵中的每个场景。

要运行测试，只需输入：

```js
$ sh -x run.sh 
```

很好，我们已经将大部分测试矩阵自动化，并且处理得相当好。测试矩阵中有一个明显的漏洞，填补这个漏洞将让我们看到如何在 Docker 下设置 MongoDB。

# 在 Docker 下设置 MongoDB 和测试 Notes 对 MongoDB

在第七章*数据存储和检索*中，我们为 Notes 开发了 MongoDB 支持，并且从那时起我们一直专注于`Sequelize`。为了弥补这个小小的不足，让我们确保至少测试我们的 MongoDB 支持。在 MongoDB 上进行测试只需定义一个 MongoDB 数据库容器和一点配置。

访问[`hub.docker.com/_/mongo/`](https://hub.docker.com/_/mongo/)获取官方 MongoDB 容器。您将能够将其修改为允许部署运行在 MongoDB 上的 Notes 应用程序。

将以下内容添加到`test-compose/docker-compose.yml`：

```js
db-notes-mongo-test:
  image: mongo:3.6-jessie
  container_name: db-notes-mongo-test
  networks:
    - frontnet-test
  volumes:
    - ./db-notes-mongo:/data/db
```

这就是添加 MongoDB 容器到 Docker Compose 文件所需的所有内容。我们已经将其连接到`frontnet`，以便`notes`（`notes-test`）容器可以访问该服务。

然后在`notes/test/package.json`中添加一行以方便在 MongoDB 上运行测试：

```js
"test-notes-mongodb": "MONGO_URL=mongodb://db-notes-mongo-test/ MONGO_DBNAME=chap11-test NOTES_MODEL=mongodb mocha --no-timeouts test-model"
```

只需将 MongoDB 容器添加到`frontnet-test`，数据库就可以通过这里显示的 URL 访问。因此，现在运行使用 Notes MongoDB 模型的测试套件变得很简单。

在对 MongoDB 测试套件进行测试时，使用`--no-timeouts`选项是必要的，以避免出现虚假错误。此选项指示 Mocha 不要检查测试用例执行是否过长。

最终的要求是在`run.sh`（或 Windows 上的`run.ps1`）中添加这一行：

```js
docker exec -it --workdir /notesapp/test -e DEBUG= notes-test npm run test-notes-mongodb
```

这样，就确保了 MongoDB 在每次测试运行时都会被测试。

我们现在可以向经理报告最终的测试结果矩阵：

+   `models-fs`：通过

+   `models-memory`：通过

+   `models-levelup`：通过

+   `models-sqlite3`：两个失败，现已修复，通过

+   `models-sequelize`与 SQLite3：通过

+   `models-sequelize`与 MySQL：通过

+   `models-mongodb`：通过

经理会告诉您“做得好”，然后记住模型只是 Notes 应用程序的一部分。我们留下了两个区域完全未测试：

+   用户身份验证服务的 REST API

+   用户界面功能测试

让我们继续进行这些测试区域。

# 测试 REST 后端服务

现在是时候将注意力转向用户认证服务了。我们提到了对这个服务的测试，说我们稍后会讨论。我们开发了一些临时测试脚本，这些脚本一直很有用。但现在“稍后”就是现在，是时候开始进行一些真正的测试了。

关于测试认证服务应该使用哪个工具，这是一个问题。Mocha 在组织一系列测试用例方面做得很好，我们应该在这里重用它。但我们必须测试的是一个 REST 服务。这个服务的客户，即笔记应用，通过 REST API 使用它，这为我们提供了一个完美的理由在 REST 接口上进行测试。我们使用的临时脚本使用了 SuperAgent 库来简化 REST API 调用。碰巧有一个配套库，SuperTest，它是专门用于 REST API 测试的。在这里阅读其文档：[`www.npmjs.com/package/supertest`](https://www.npmjs.com/package/supertest)。

我们已经创建了 `test-compose/userauth` 目录。在该目录中，创建一个名为 `test.js` 的文件：

```js
'use strict';

const assert = require('chai').assert;
const request = require('supertest')(process.env.URL_USERS_TEST);
const util = require('util');
const url = require('url');
const URL = url.URL;

const authUser = 'them';
const authKey = 'D4ED43C0-8BD6-4FE2-B358-7C0E230D11EF';

describe("Users Test", function() {
    ... Test code follows
});
```

这设置了 Mocha 和 SuperTest 客户端。`URL_USERS_TEST` 环境变量指定了运行测试的服务器的基准 URL。鉴于我们在本书中早期使用的配置，你几乎肯定会使用 `http://localhost:3333`。SuperTest 与 SuperAgent 的初始化方式略有不同。SuperTest 模块暴露了一个函数，我们使用 `URL_USERS_TEST` 环境变量调用它，然后在整个脚本中使用那个 `request` 对象来发起 REST API 请求。

这个变量已经在 `test-compose/docker-compose.yml` 中设置，并包含了所需值。另一个重要的事情是一对变量，用于存储认证用户 ID 和密钥：

```js
beforeEach(async function() {
  await request.post('/create-user')
       .send({ 
          username: "me", password: "w0rd", provider: "local",
          familyName: "Einarrsdottir", givenName: "Ashildr", 
          middleName: "",
          emails: [], photos: []
        })
        .set('Content-Type', 'application/json')
        .set('Acccept', 'application/json')
        .auth(authUser, authKey);
});

afterEach(async function() {
  await request.delete('/destroy/me')
        .set('Content-Type', 'application/json')
        .set('Acccept', 'application/json')
        .auth(authUser, authKey);
}); 
```

如果你还记得，`beforeEach` 函数在每次测试用例之前立即运行，而 `afterEach` 在之后运行。这些函数使用 REST API 在运行测试之前创建我们的测试用户，然后在之后销毁测试用户。这样我们的测试就可以假设这个用户将存在：

```js
describe("List user", function() {
   it("list created users", async function() {
     const res = await request.get('/list')
          .set('Content-Type', 'application/json')
          .set('Acccept', 'application/json')
          .auth(authUser, authKey);
    assert.exists(res.body);
    assert.isArray(res.body);
    assert.lengthOf(res.body, 1);
    assert.deepEqual(res.body[0], { 
          username: "me", id: "me", provider: "local",
          familyName: "Einarrsdottir", givenName: "Ashildr", 
          middleName: "",
          emails: [], photos: []
    });
  });
});
```

现在，我们可以转向测试一些 API 方法，例如 `/list` 操作。

我们已经在 `beforeEach` 方法中保证了账户的存在，所以 `/list` 应该会给我们一个包含一个条目的数组。

这遵循了使用 Mocha 测试 REST API 方法的一般模式。首先，我们使用 SuperTest 的 `request` 对象调用 API 方法，并 `await` 其结果。一旦我们得到结果，我们使用 `assert` 方法来验证它是否符合预期：

```js
describe("find user", function() {
  it("find created users", async function() {
    const res = await request.get('/find/me')
            .set('Content-Type', 'application/json')
            .set('Acccept', 'application/json')
            .auth(authUser, authKey);
    assert.exists(res.body);
    assert.isObject(res.body);
    assert.deepEqual(res.body, { 
            username: "me", id: "me", provider: "local",
            familyName: "Einarrsdottir", givenName: "Ashildr", 
            middleName: "",
            emails: [], photos: []
    });
});
it("fail to find non-existent users", async function() {
    var res;
    try {
      res = await request.get('/find/nonExistentUser')
            .set('Content-Type', 'application/json')
            .set('Acccept', 'application/json')
            .auth(authUser, authKey);
    } catch(e) {
      return; // Test is okay in this case
    }
    assert.exists(res.body);
    assert.isObject(res.body);
    assert.deepEqual(res.body, {});
  });
});    
```

我们正在以两种方式检查 `/find` 操作：

+   寻找我们已知存在的账户——如果用户账户未找到，则表示失败

+   寻找我们知道不存在的账户——如果我们收到错误或空对象以外的任何东西，则表示失败

添加这个测试用例：

```js
describe("delete user", function() {
  it("delete nonexistent users", async function() {
    var res;
    try {
      res = await request.delete('/destroy/nonExistentUser')
              .set('Content-Type', 'application/json')
              .set('Acccept', 'application/json')
              .auth(authUser, authKey);
    } catch(e) {
      return; // Test is okay in this case
    }
    assert.exists(res);
    assert.exists(res.error);
    assert.notEqual(res.status, 200);
  });
}); 
```

最后，我们应该检查 `/destroy` 操作。我们已经在 `afterEach` 方法中检查了这个操作，其中我们 `destroy` 一个已知的用户账户。我们还需要执行负测试并验证其行为与一个我们知道不存在的账户的行为。

所需的行为是抛出一个错误，或者结果显示一个表示错误的 HTTP `状态`。实际上，当前的认证服务器代码会返回一个 500 状态码和一些其他信息。

在 `test-compose/docker-compose.yml` 中，我们需要将此脚本 `test.js` 注入到 `userauth-test` 容器中。我们在这里添加它：

```js
userauth-test:
  ...
  volumes:
    - ./reports-userauth:/reports
    - ./userauth/sequelize-docker-test-mysql.yaml:/userauth/sequelize-docker-test-mysql.yaml
    - ./userauth/test.js:/userauth/test.js
```

我们有一个测试脚本，并且已经将其注入到所需的容器中（`userauth-test`）。下一步是自动化运行这个测试。一种方法是将此添加到 `run.sh`（在 Windows 上称为 `run.ps1`）中：

```js
docker exec -it -e DEBUG= userauth-test npm install supertest mocha chai
docker exec -it -e DEBUG= userauth-test ./node_modules/.bin/mocha test.js
```

现在，如果你运行 `run.sh` 测试脚本，你会看到所需的包被安装，然后执行这个测试套件。

# 自动化测试结果报告

我们有自动化测试执行真是太酷了，Mocha 通过所有那些勾选标记使测试结果看起来很棒。如果管理层想要一个随时间变化的测试失败趋势图呢？或者可能有无数的理由将测试结果作为数据而不是在控制台上友好的打印输出。

Mocha 使用所谓的报告器来报告测试结果。Mocha 报告器是一个模块，以它支持的任何格式打印数据。相关信息可以在 Mocha 网站上找到：[`mochajs.org/#reporters`](https://mochajs.org/#reporters)。

你可以这样找到当前可用的 `reporters` 列表：

```js
# mocha --reporters

 dot - dot matrix
 doc - html documentation
 spec - hierarchical spec list
 json - single json object
 progress - progress bar
 list - spec-style listing
 tap - test-anything-protocol
...
```

然后，你可以使用特定的 `reporter` 如下所示：

```js
# mocha --reporter tap test
1..4
ok 1 Users Test List user list created users
ok 2 Users Test find user find created users
ok 3 Users Test find user fail to find non-existent users
ok 4 Users Test delete user delete nonexistent users
# tests 4
# pass 4
# fail 0
```

**测试任何协议**（**TAP**）是一个广泛使用的测试结果格式，增加了找到高级报告工具的可能性。显然，下一步是将结果保存到某个文件中，在将主机目录挂载到容器之后。

# 使用 Puppeteer 进行前端无头浏览器测试

测试中的一个大成本领域是手动用户界面测试。因此，已经开发了一系列工具来自动化在 HTTP 层面上运行测试。例如，Selenium 是一个流行的用 Java 实现的工具。在 Node.js 世界中，我们有几个有趣的选择。Chai 的 *chai-http* 插件允许我们在 Chai 环境中与 Notes 应用程序进行 HTTP 层面的交互，同时保持熟悉的环境。

然而，对于本节，我们将使用 Puppeteer ([`github.com/GoogleChrome/puppeteer`](https://github.com/GoogleChrome/puppeteer))。这个工具是一个高级 Node.js 模块，用于通过 DevTools 协议控制无头 Chrome 或 Chromium 浏览器。该协议允许工具对 Chromium 或 Chrome 进行仪器化、检查、调试和性能分析。

Puppeteer 旨在成为一个通用的测试自动化工具，并且为此目的拥有强大的功能集。由于 Puppeteer 很容易制作网页截图，它也可以用于截图服务。

由于 Puppeteer 控制的是真实的网络浏览器，因此你的用户界面测试将非常接近于实际浏览器测试，无需雇佣人类来完成这项工作。因为它使用的是无头版本的 Chrome，所以屏幕上不会显示任何可见的浏览器窗口，测试可以在后台运行。然而，这个吸引人的故事的缺点是 Puppeteer 只能在 Chrome 上工作。这意味着针对 Chrome 的自动化测试并不能测试你的应用程序在其他浏览器上的表现，例如 Opera 或 Firefox。

# 设置 Puppeteer

让我们先设置目录并安装包：

```js
$ mkdir test-compose/notesui
$ cd test-compose/notesui
$ npm init
... answer the questions
$ npm install puppeteer@1.1.x mocha@5.x chai@4.1.x --save
```

在安装过程中，你会看到 Puppeteer 会像这样下载 Chromium：

```js
Downloading Chromium r497674 - 92.5 Mb [====================] 100% 0.0s 
```

`puppeteer` 模块将根据需要启动那个 Chromium 实例，将其作为后台进程管理，并使用 DevTools 协议与之通信。

在我们即将编写的脚本中，我们需要一个用户账户，我们可以用它来登录并执行一些操作。幸运的是，我们已经有了一个设置测试账户的脚本。在 `users/package.json` 文件中，在 scripts 部分添加以下行：

```js
"setupuser": "PORT=3333 node users-add", 
```

我们即将编写这个测试脚本，但让我们先完成设置，最后一步是将以下行添加到 `run.sh` 文件中：

```js
docker exec -it userauth-test npm run setupuser 
docker exec -it notesapp-test npm run test-docker-ui 
```

当执行时，这两行确保测试用户被设置，然后运行用户界面测试。

# 提高 Notes UI 的可测试性

当 Notes 应用程序在浏览器中显示良好时，我们如何编写测试软件来区分不同的页面？关键要求是测试脚本需要检查页面，确定显示的是哪个页面，并读取页面上的数据。这意味着每个 HTML 元素都必须能够通过 CSS 选择器轻松访问。

在开发 Notes 应用程序时，我们忘记了这一点，而 **软件质量工程** (**SQE**) 经理要求我们提供帮助。这关系到测试预算，而 SQE 团队可以自动化的测试越多，预算就会越紧张。

所需的只是给 HTML 元素添加一些 `id` 或 `class` 属性来提高可测试性。有了几个标识符，并承诺维护这些标识符，SQE 团队可以编写可重复的测试脚本以验证应用程序。

在 `notes/partials/header.hbs` 文件中，修改以下行：

```js
...
<a id="btnGoHome" class="navbar-brand" href='/'>
...
{{#if user}}
...
<a class="nav-item nav-link btn btn-dark col-auto" id="btnLogout" href="/users/logout">...</a>
<a class="nav-item nav-link btn btn-dark col-auto" id="btnAddNote" href='/notes/add'>...</a>
{{else}}
...
<a class="nav-item nav-link btn btn-dark col-auto" id="btnloginlocal" href="/users/login">..</a>
<a class="nav-item nav-link btn btn-dark col-auto" 
                        id="btnLoginTwitter" href="/users/auth/twitter">...</a>
...
{{/if}}
...
```

在 `notes/views/index.hbs` 文件中，进行以下修改：

```js
<div id="notesHomePage" class="container-fluid">
  <div class="row">
    <div id="notetitles" class="col-12 btn-group-vertical" role="group">
      {{#each notelist}}
      <a id="{{key}}" class="btn btn-lg btn-block btn-outline-dark" 
          href="/notes/view?key={{ key }}">...</a>
      {{/each}}
    </div>
  </div>
</div>
```

在 `notes/views/login.hbs` 文件中，进行以下修改：

```js
<div id="notesLoginPage" class="container-fluid">
...
<form id="notesLoginForm" method='POST' action='/users/login'> 
...
<button type="submit" id="formLoginBtn" class="btn btn-default">Submit</button> 
</form>
...
</div>
```

在 `notes/views/notedestroy.hbs` 文件中，进行以下修改：

```js
<form id="formDestroyNote" method='POST' action='/notes/destroy/confirm'>
...
<button id="btnConfirmDeleteNote" type="submit" value='DELETE' 
                class="btn btn-outline-dark">DELETE</button>
...
</form>
```

在 `notes/views/noteedit.hbs` 文件中，进行以下修改：

```js
<form id="formAddEditNote" method='POST' action='/notes/save'>
...
<button id='btnSave' type="submit" class="btn btn-default">Submit</button>
...
</form>
```

在 `notes/views/noteview.hbs` 文件中，进行以下修改：

```js
<div id="noteView" class="container-fluid">
...
<p id="showKey">Key: {{ notekey }}</p>
...
<a id="btnDestroyNote" class="btn btn-outline-dark" 
     href="/notes/destroy?key={{notekey}}" role="button"> ...  </a>
<a id="btnEditNote" class="btn btn-outline-dark" 
     href="/notes/edit?key={{notekey}}" role="button"> ... </a>
<button id="btnComment" type="button" class="btn btn-outline-dark" 
     data-toggle="modal" data-target="#notes-comment-modal"> ... </button> 
...
</div>
```

我们所做的是在模板中选定的元素上添加了 `id=` 属性。现在我们可以轻松地编写 CSS 选择器来定位任何元素。工程团队也可以开始在 UI 代码中使用这些选择器。

# Notes 的 Puppeteer 测试脚本

在 `test-compose/notesui` 目录中，创建一个名为 `uitest.js` 的文件，包含以下内容：

```js
const puppeteer = require('puppeteer');
const assert = require('chai').assert;
const util = require('util');
const { URL } = require('url');

describe('Notes', function() {
    this.timeout(10000);
    let browser;
    let page;

    before(async function() {
        browser = await puppeteer.launch({ slomo: 500 });
        page = await browser.newPage();
        await page.goto(process.env.NOTES_HOME_URL);
    });

    after(async function() {
        await page.close();
        await browser.close();
    });
});
```

这是 Mocha 测试套件的开始。在 `before` 函数中，我们通过启动 Puppeteer 实例、创建一个新的 Page 对象，并指示该 Page 访问笔记应用程序的主页来设置 Puppeteer。该 URL 使用命名环境变量传递。

首先考虑我们可能想要使用 Notes 应用程序验证的场景是有用的：

+   登录到 Notes 应用程序

+   向应用程序添加笔记

+   查看附加说明

+   删除添加的笔记

+   注销

+   等等

下面是登录场景实现的代码：

```js
describe('Login', function() {
    before(async function() { ... });
    it('should click on login button', async function() {
        const btnLogin = await page.waitForSelector('#btnloginlocal');
        await btnLogin.click();
    });
    it('should fill in login form', async function() {
        const loginForm = await page.waitForSelector('#notesLoginPage 
        #notesLoginForm');
        await page.type('#notesLoginForm #username', "me");
        await page.type('#notesLoginForm #password', "w0rd");
        await page.click('#formLoginBtn');
    });
    it('should return to home page', async function() {
        const home = await page.waitForSelector('#notesHomePage');
        const btnLogout = await page.waitForSelector('#btnLogout');
        const btnAddNote = await page.$('#btnAddNote');
        assert.exists(btnAddNote);
    });
    after(async function() { ... });
});
```

此测试序列处理登录场景。它展示了几个 Puppeteer API 方法。完整 API 文档位于 [`github.com/GoogleChrome/puppeteer/blob/master/docs/api.md`](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md)。Page 对象封装了 Chrome/Chromium 中浏览器标签页的等效功能。

`waitForSelector` 函数如其名所示——它等待匹配 CSS 选择器的 HTML 元素出现，并且它将在一个或多个页面刷新中等待。此函数有几个变体，允许等待多种类型的事物。此函数返回一个 Promise，这使得在我们的测试代码中使用异步函数变得值得。Promise 将解析为 `ElementHandle`，这是一个围绕 HTML 元素的包装器，或者抛出一个异常，这会方便地使测试失败。

命名的元素 `#btnloginlocal` 在 `partials/header.hbs` 中，只有在用户未登录时才会显示。因此，我们可以确定浏览器当前正在显示 Notes 页面，并且未登录。

`click` 方法如其所暗示的那样，在引用的 HTML 元素上触发鼠标按钮点击。如果您想模拟触摸，例如用于移动设备，有一个 `tap` 方法用于此目的。

测试序列的下一个阶段从点击那里开始。浏览器应该已经跳转到登录页面，因此此 CSS 选择器应该有效：`#notesLoginPage #notesLoginForm`。我们接下来要做的是在相应的表单元素中输入测试用户的用户 ID 和密码，然后点击登录按钮。

下一个测试阶段从那里开始，浏览器应该根据此 CSS 选择器位于主页上：`#notesHomePage`。如果我们成功登录，页面应该有注销（`#btnLogout`）和添加笔记按钮（`#btnAddNote`）。

在这种情况下，我们使用了一个不同的函数 `$` 来检查添加笔记按钮是否存在。与 `wait` 函数不同，`$` 仅查询当前页面而不等待。如果命名的 CSS 选择器不在当前页面中，它将简单地返回 `null` 而不是抛出异常。因此，为了确定元素是否存在，我们使用 `assert.exists` 而不是依赖于抛出的异常。

# 运行登录场景

现在我们已经输入了一个测试场景，让我们试一试。在一个窗口中，启动 Notes 测试基础设施：

```js
$ cd test-compose
$ docker-compose up --force-rebuild
```

然后在另一个窗口中：

```js
$ docker exec -it userauth bash
userauth# PORT=3333 node ./users-add.js
userauth# exit
$ cd test-compare/notesui
$ NOTES_HOME_URL=http://localhost:3000 mocha --no-timeouts uitest.js 

 Notes
 Login
 √ should click on login button
 √ should fill in login form (72ms)
 √ should return to home page (1493ms)

 3 passing (3s)
```

`NOTES_HOME_URL` 变量是脚本查找以指导 Chromium 浏览器使用笔记应用程序的内容。为了运行测试，我们应该使用 Docker Compose 启动测试基础设施，并确保测试用户已安装到用户数据库中。

# 添加备注的场景

将以下内容添加到 `uitest.js`：

```js
describe('Add Note', function() {
    // before(async function() { ... });
    it('should see Add Note button', async function() {
        const btnAddNote = await page.waitForSelector('#btnAddNote');
        await btnAddNote.click();
    });
    it('should fill in Add Note form', async function() {
        const formAddEditNote = await 
        page.waitForSelector('#formAddEditNote');
        await page.type('#notekey', 'key42');
        await page.type('#title', 'Hello, world!');
        await page.type('#body', 'Lorem ipsum dolor');
        await page.click('#btnSave');
    });
    it('should view note', async function() {
        await page.waitForSelector('#noteView');
        const shownKey = await page.$eval('#showKey', el => 
        el.innerText);
        assert.exists(shownKey);

        assert.isString(shownKey);
        assert.include(shownKey, 'key42');
        const shownTitle = await page.$eval('#notetitle', el => 
        el.innerText);
        assert.exists(shownTitle);
        assert.isString(shownTitle);
        assert.include(shownTitle, 'Hello, world!');
        const shownBody = await page.$eval('#notebody', el => 
        el.innerText);
        assert.exists(shownBody);
        assert.isString(shownBody);
        assert.include(shownBody, 'Lorem ipsum dolor');
    });
    it('should go to home page', async function() {
        await page.waitForSelector('#btnGoHome');
        await page.goto(process.env.NOTES_HOME_URL);
        // await page.click('#btnGoHome');
        await page.waitForSelector('#notesHomePage');
        const titles = await page.$('#notetitles');
        assert.exists(titles);
        const key42 = await page.$('#key42');
        assert.exists(key42);
        const btnLogout = await page.$('#btnLogout');
        assert.exists(btnLogout);
        const btnAddNote = await page.$('#btnAddNote');
        assert.exists(btnAddNote);
    });
    // after(async function() { ... });
});
```

这是一个更复杂的场景，其中我们：

+   点击“添加备注”按钮

+   等待备注编辑屏幕出现

+   填写备注的文本并点击“保存”按钮

+   验证备注查看页面以确保正确无误

+   验证主页以确保正确无误。

这大部分是使用与之前相同的 Puppeteer 函数，但增加了一些功能。

`$eval` 函数查找匹配 CSS 选择器的元素，并在该元素上调用回调函数。如果没有找到元素，则会抛出错误。在此处使用时，我们从屏幕上的某些元素中检索文本，并验证它是否与测试输入的备注匹配。这是一个添加和检索备注的端到端测试。

下一个区别是使用 `goto` 而不是点击 `#btnGoHome`。

当您向测试脚本添加测试场景时，您会发现 Puppeteer 很容易产生虚假的超时，或者登录过程神秘地不起作用，或者出现其他虚假错误。

我们不会详细说明剩余的场景，而是在下一节中讨论如何缓解此类问题。但首先我们需要证明即使我们必须运行测试 10 次才能得到这个结果，该场景仍然有效：

```js
$ NOTES_HOME_URL=http://localhost:3000 ./node_modules/.bin/mocha --no-timeouts uitest3.js 

 Notes
 Login
 √ should click on login button (50ms)
 √ should fill in login form (160ms)
 √ should return to home page (281ms)
 Add Note
 √ should see Add Note button
 √ should fill in Add Note form (1843ms)
 √ should view note
 √ should go to home page (871ms)

 7 passing (5s)
```

# 缓解/预防 Puppeteer 脚本中的虚假测试错误

目标是完全自动化测试运行，以避免需要雇佣人类来监视测试执行并花费时间重新运行测试，因为虚假错误。为此，测试需要在没有任何虚假错误的情况下可重复。Puppeteer 是一个复杂的系统——有一个 Node.js 模块与在后台运行的无头 Chromium 实例进行通信——并且似乎时间问题很容易导致虚假错误。

# 配置超时

Mocha 和 Puppeteer 都允许您设置超时值，并且一个较长的超时值可以避免触发错误，如果某些操作需要较长时间运行。在测试套件的顶部，我们使用了这个 Mocha 函数：

```js
this.timeout(10000);
```

这为每个测试案例提供了 10 秒的时间。如果您想使用更长的超时，请增加该数值。

`puppeteer.launch` 函数可以在其选项对象中接受超时值。默认情况下，Puppeteer 在大多数操作上使用 30 秒的超时，并且它们都带有选项对象，可以更改该超时期间。在这种情况下，我们添加了 `slowMo` 选项来减慢浏览器上的操作。

# 跟踪页面和 Puppeteer 实例上的事件

另一个有用的策略是生成发生事件的跟踪，这样您就可以进行推理。插入 `console.log` 语句很繁琐，并且会使您的代码看起来有些丑陋。Puppeteer 提供了一些方法来跟踪操作，并动态地关闭跟踪。

在 `uitest.js` 中，添加以下代码：

```js

function frameEvent(evtname, frame) {
    console.log(`${evtname} ${frame.url()} ${frame.title()}`);
}

function ignoreURL(url) {
    if (url.match(/\/assets\//) === null 
     && url.match(/\/socket.io\//) === null
     && url.match(/fonts.gstatic.com/) === null
     && url.match(/fonts.googleapis.com/) === null) {
        return false;
    } else {
        return true;
    }
}
...
before(async function() {
    browser = await puppeteer.launch({ slomo: 500 });
    page = await browser.newPage();
    page.on('console', msg => {
        console.log(`${msg.type()} ${msg.text()} ${msg.args().join(' ')}`);
    });
    page.on('error', err => {
        console.error(`page ERROR ${err.stack}`);
    });
    page.on('pageerror', err => {
        console.error(`page PAGEERROR ${err.stack}`);
    });
    page.on('request', req => {
        if (ignoreURL(req.url())) return;
        console.log(`page request ${req.method()} ${req.url()}`);
    });
    page.on('response', async (res) => {
        if (ignoreURL(res.url())) return;
        console.log(`page response ${res.status()} ${res.url()}`);
    });
    page.on('frameattached', async (frame) => frameEvent('frameattached', await frame));
    page.on('framedetached', async (frame) => frameEvent('framedetached', await frame));
    page.on('framenavigated', async (frame) => frameEvent('framenavigated', await frame));
    await page.goto(process.env.NOTES_HOME_URL);
});
...
```

即，页面对象提供了几个事件监听器，我们可以输出有关各种事件的详细信息，包括 HTTP 请求和响应。我们甚至可以打印出响应的 HTML 文本。`ignoreURL` 函数让我们抑制一些选定的 URL，这样我们不会收到大量不重要的请求和响应。

您可以使用 Puppeteer 的 DEBUG 环境变量来跟踪 Puppeteer 本身。更多信息请参阅 README：[`github.com/GoogleChrome/puppeteer`](https://github.com/GoogleChrome/puppeteer)

# 插入暂停

在某些点上插入较长的暂停，以便浏览器有时间执行某些操作是有用的。尝试这个函数：

```js
function waitFor(timeToWait) {
    return new Promise(resolve => {
      setTimeout(() => { resolve(true); }, timeToWait);
    });
};
```

这是我们使用 Promises 实现类似 `sleep` 函数的方法。使用 `setTimeOut` 以这种方式，结合超时值，只是简单地延迟指定的毫秒数。

要使用此函数，只需将其插入到测试场景中：

```js
await waitFor(3000);
```

另一种变体是等待浏览器中内容完全渲染。例如，您可能已经看到在左上角的 Home 图标完全渲染之前有一个暂停。这个暂停可能会引起虚假的错误，而这个函数可以等待该按钮完全渲染：

```js
async function waitForBtnGoHome() {
    return page.waitForSelector('#btnGoHome');
}
```

要使用它：

```js
await waitForBtnGoHome();
```

如果您不想维护这个额外的函数，将 `waitForSelector` 调用添加到您的测试用例中也很容易。

# 避免 WebSocket 冲突

Puppeteer 可能会抛出一个错误，`Cannot find context with specified id undefined`。根据 Puppeteer 问题队列中的一个问题，这可能是由于 Puppeteer 和 WebSocket 之间的意外交互引起的：[`github.com/GoogleChrome/puppeteer/issues/1325`](https://github.com/GoogleChrome/puppeteer/issues/1325) 这个问题反过来会影响笔记应用程序中的 Socket.IO 支持，因此，在测试运行期间禁用 Socket.IO 支持可能是有用的。

允许禁用 Socket.IO 相对简单。在 `app.mjs` 中，添加这个导出的函数：

```js
export function enableSocketio() {
  var ret = true;
  const env = process.env.NOTES_DISABLE_SOCKETIO;
  if (!env || env !== 'true') {
    ret = true;
  }
  return ret;
}
```

这个函数会查找环境变量以使函数返回 `true` 或 `false`。

在 `routes/index.mjs` 和 `routes/notes.mjs` 中，添加以下行：

```js
import { enableSocketio, sessionCookieName } from '../app';
```

我们这样做是为了导入前面的函数。这也展示了我们从 ES6 模块中获得的一些灵活性，因为我们可以只导入所需的函数。

在 `routes/index.mjs` 和 `routes/notes.mjs` 中，对于每个调用 `res.render` 以发送结果的路由函数，使用 `enableSocketio` 函数如下：

```js
res.render('*view-name*', { 
  ...
  enableSocketio: enableSocketio()
});
```

因此，我们已经导入了该函数，并且对于每个视图，我们将 `enableSocketio` 作为数据传递给视图模板。

在 `views/index.hbs` 和 `views/noteview.hbs` 中，我们有一个 JavaScript 代码段来实现基于 SocketIO 的半实时功能。像这样包围每个这样的部分：

```js
{{#if enableSocketio}}
...  JavaScript code for SocketIO support
{{/if}}
```

通过消除客户端的 SocketIO 代码，我们确保用户界面不会打开到 SocketIO 服务的连接。这个练习的目的是避免使用 WebSockets，以避免与 Puppeteer 相关的问题。

类似地，在 `views/noteview.hbs` 中，可以这样禁用评论按钮：

```js
{{#if enableSocketio}}
    <button id="btnComment" type="button" class="btn btn-outline-dark" 
        data-toggle="modal" data-target="#notes-comment-modal">Comment</button>
{{/if}}
```

最后一步是在 Docker Compose 文件中设置环境变量，`NOTES_DISABLE_SOCKETIO`。

# 捕获截图

Puppeteer 的核心功能之一是捕获截图，可以是 PNG 或 PDF 文件。在我们的测试脚本中，我们可以捕获截图以跟踪测试过程中任何给定时间屏幕上的内容。例如，如果登录场景意外失败，我们可以在截图看到这一点：

```js
await page.screenshot({
      type: 'png',
      path: `./screen/login-01-start.png`
});
```

在你的测试脚本中添加类似这样的代码片段。这里显示的文件名遵循一种约定，其中第一个部分命名测试场景，数字是测试场景内的序列号，最后一个是测试场景内的步骤描述。

捕获截图也提供了另一阶段的验证。你可能还想对你的应用程序进行视觉验证。`pixelmatch` 模块可以比较两个 PNG 文件，因此可以在测试运行期间维护一组所谓的黄金图像进行比较。

有关使用 Puppeteer 的示例，请参阅：[`meowni.ca/posts/2017-puppeteer-tests/`](https://meowni.ca/posts/2017-puppeteer-tests/)。

# 摘要

在本章中，我们涵盖了大量的内容，探讨了三个不同的测试领域：单元测试、REST API 测试和 UI 功能测试。确保应用程序经过良好测试是通往软件成功之路的重要一步。一个不遵循良好测试实践的团队往往会陷入修复回归问题的困境。

我们讨论了仅使用 assert 模块进行测试的潜在简单性。虽然测试框架，如 Mocha，提供了许多功能，但我们可以通过简单的脚本走得很远。

测试框架，例如 Mocha，有其存在的理由，即使只是为了规范我们的测试用例，并生成测试结果报告。我们使用了 Mocha 和 Chai 来完成这项工作，并且这些工具非常成功。我们甚至通过一个小型的测试套件发现了几处错误。

在开始单元测试的道路上，一个设计考虑因素是模拟依赖项。但并不是总是值得花费我们的时间用模拟版本替换每个依赖项。

为了减轻运行测试的管理负担，我们使用了 Docker 来自动设置和拆除测试基础设施。正如 Docker 在自动化部署笔记应用程序时很有用一样，它也有助于自动化测试基础设施的部署。

最后，我们能够在真实网络浏览器中测试笔记网络用户界面。我们不能相信单元测试会找到每一个错误；一些错误只有在网络浏览器中才会出现。即便如此，我们只是触及了 Notes 中可能测试的冰山一角。

在这本书中，我们涵盖了 Node.js 开发的各个方面，为你提供了一个强大的基础，从这里开始开发 Node.js 应用程序。

在下一章中，我们将探讨 RESTful 网络服务。
