单元测试和功能测试

单元测试已成为良好软件开发实践的主要部分。这是一种通过测试源代码的各个单元来确保它们正常运行的方法。每个单元在理论上都是应用程序中最小的可测试部分。

在单元测试中，每个单元都是单独测试的，尽可能地将被测试的单元与应用程序的其他部分隔离开来。如果测试失败，你希望它是由于你的代码中的错误而不是你的代码碰巧使用的包中的错误。一个常见的技术是使用模拟对象或模拟数据来将应用程序的各个部分相互隔离开来。

另一方面，功能测试并不试图测试单独的组件。相反，它测试整个系统。一般来说，单元测试由开发团队执行，而功能测试由**质量保证**（**QA**）或**质量工程**（**QE**）团队执行。这两种测试模型都需要完全认证一个应用程序。一个类比可能是，单元测试类似于确保句子中的每个单词都拼写正确，而功能测试确保包含该句子的段落具有良好的结构。

写一本书不仅需要确保单词拼写正确，还需要确保单词串成有用的语法正确的句子和传达预期含义的章节。同样，一个成功的软件应用程序需要远不止确保每个“单元”行为正确。整个系统是否执行了预期的操作？

在本章中，我们将涵盖以下主题：

+   断言作为软件测试的基础

+   Mocha 单元测试框架和 Chai 断言库

+   使用测试来查找错误并修复错误

+   使用 Docker 管理测试基础设施

+   测试 REST 后端服务

+   在真实的 Web 浏览器中使用 Puppeteer 进行 UI 功能测试

+   使用元素 ID 属性改进 UI 可测试性

在本章结束时，你将知道如何使用 Mocha，以及如何为直接调用的测试代码和通过 REST 服务访问的测试代码编写测试用例。你还将学会如何使用 Docker Compose 来管理测试基础设施，无论是在你的笔记本电脑上还是在来自第十二章的 AWS EC2 Swarm 基础设施上，*使用 Terraform 部署 Docker Swarm 到 AWS EC2*。

这是一个需要覆盖的大片领土，所以让我们开始吧。

# 第十七章：Assert - 测试方法学的基础

Node.js 有一个有用的内置测试工具，称为`assert`模块。其功能类似于其他语言中的 assert 库。换句话说，它是一组用于测试条件的函数，如果条件表明存在错误，`assert`函数会抛出异常。虽然它并不是完整的测试框架，但仍然可以用于一定程度的测试。

在其最简单的形式中，测试套件是一系列`assert`调用，用于验证被测试对象的行为。例如，一个测试套件可以实例化用户认证服务，然后进行 API 调用并使用`assert`方法验证结果，然后进行另一个 API 调用来验证其结果，依此类推。

考虑以下代码片段，你可以将其保存在名为`deleteFile.mjs`的文件中：

```

The first thing to notice is this contains several layers of asynchronous callback functions. This presents a couple of challenges:  

*   Capturing errors from deep inside a callback
*   Detecting conditions where the callbacks are never called

The following is an example of using `assert` for testing. Create a file named `test-deleteFile.mjs` containing the following:

```

这就是所谓的负面测试场景，它测试的是请求删除一个不存在的文件是否会抛出正确的错误。如果要删除的文件不存在，`deleteFile`函数会抛出一个包含*不存在*文本的错误。这个测试确保正确的错误被抛出，如果抛出了错误的错误，或者没有抛出错误，测试将失败。

如果您正在寻找一种快速测试的方法，`assert`模块在这种用法下可能很有用。每个测试用例都会调用一个函数，然后使用一个或多个`assert`语句来测试结果。在这种情况下，`assert`语句首先确保`err`具有某种值，然后确保该值是`Error`实例，最后确保`message`属性具有预期的文本。如果运行并且没有消息被打印，那么测试通过。但是如果`deleteFile`回调从未被调用会发生什么？这个测试用例会捕获到这个错误吗？

```

No news is good news, meaning it ran without messages and therefore the test passed.

The `assert` module is used by many of the test frameworks as a core tool for writing test cases. What the test frameworks do is create a familiar test suite and test case structure to encapsulate your test code, plus create a context in which a series of test cases are robustly executed.

For example, we asked about the error of the callback function never being called. Test frameworks usually have a timeout so that if no result of any kind is supplied within a set number of milliseconds, then the test case is considered an error.

There are many styles of assertion libraries available in Node.js. Later in this chapter, we'll use the Chai assertion library ([`chaijs.com/`](http://chaijs.com/)), which gives you a choice between three different assertion styles (should, expect, and assert).

# Testing a Notes model

Let's start our unit testing journey with the data models we wrote for the Notes application. Because this is unit testing, the models should be tested separately from the rest of the Notes application.

In the case of most of the Notes models, isolating their dependencies implies creating a mock database. Are you going to test the data model or the underlying database? Mocking out a database means creating a fake database implementation, which does not look like a productive use of our time. You can argue that testing a data model is really about testing the interaction between your code and the database. Since mocking out the database means not testing that interaction, we should test our code against the database engine in order to validate that interaction.

With that line of reasoning in mind, we'll skip mocking out the database, and instead run the tests against a database containing test data. To simplify launching the test database, we'll use Docker to start and stop a version of the Notes application stack that's set up for testing.

Let's start by setting up the tools.

## Mocha and Chai­ – the chosen test tools

If you haven't already done so, duplicate the source tree so that you can use it in this chapter. For example, if you had a directory named `chap12`, create one named `chap13` containing everything from `chap12` to `chap13`.

In the `notes` directory, create a new directory named `test`.

Mocha ([`mochajs.org/`](http://mochajs.org/)) is one of many test frameworks available for Node.js. As you'll see shortly, it helps us write test cases and test suites, and it provides a test results reporting mechanism. It was chosen over the alternatives because it supports Promises. It fits very well with the Chai assertion library mentioned earlier. 

While in the `notes/test` directory, type the following to install Mocha and Chai:

```

当然，这会设置一个`package.json`文件并安装所需的软件包。

除了 Mocha 和 Chai 之外，我们还安装了两个额外的工具。第一个是`cross-env`，这是我们以前使用过的，它可以在命令行上设置环境变量的跨平台支持。第二个是`npm-run-all`，它简化了使用`package.json`来驱动构建或测试过程。

有关`cross-env`的文档，请访问[`www.npmjs.com/package/cross-env`](https://www.npmjs.com/package/cross-env)。

有关`npm-run-all`的文档，请访问[`www.npmjs.com/package/npm-run-all`](https://www.npmjs.com/package/npm-run-all)。

有了设置好的工具，我们可以继续创建测试。

## Notes 模型测试套件

因为我们有几个 Notes 模型，测试套件应该针对任何模型运行。我们可以使用 NotesStore API 编写测试，并且应该使用环境变量来声明要测试的模型。因此，测试脚本将加载`notes-store.mjs`并调用它提供的对象上的函数。其他环境变量将用于其他配置设置。

因为我们使用 ES6 模块编写了 Notes 应用程序，所以我们需要考虑一个小问题。旧版的 Mocha 只支持在 CommonJS 模块中运行测试，因此这需要我们跳过一些步骤来测试 Notes 模块。但是当前版本的 Mocha 支持它们，这意味着我们可以自由使用 ES6 模块。

我们将首先编写一个单独的测试用例，并按照运行该测试和获取结果的步骤进行。之后，我们将编写更多的测试用例，甚至找到一些错误。这些错误将给我们一个机会来调试应用程序并解决任何问题。我们将通过讨论如何运行需要设置后台服务的测试来结束本节。

### 创建初始 Notes 模型测试用例

在`test`目录中，创建一个名为`test-model.mjs`的文件，其中包含以下内容。这将是测试套件的外壳：

```

This loads in the required modules and implements the first test case.

The Chai library supports three flavors of assertions. We're using the `assert` style here, but it's easy to use a different style if you prefer.

For the other assertion styles supported by Chai, see [`chaijs.com/guide/styles/`](http://chaijs.com/guide/styles/).

Chai's assertions include a very long list of useful assertion functions. For the documentation, see [`chaijs.com/api/assert/`](http://chaijs.com/api/assert/).

To load the model to be tested, we call the `useModel` function (renamed as `useNotesModel`). You'll remember that this uses the `import()` function to dynamically select the actual NotesStore implementation to use. The `NOTES_MODEL` environment variable is used to select which to load.

Calling `this.timeout` adjusts the time allowed for completing the test. By default, Mocha allows 2,000 milliseconds (2 seconds) for a test case to be completed. This particular test case might take longer than that, so we've given it more time.

The test function is declared as `async`.  Mocha can be used in a callback fashion, where Mocha passes in a callback to the test to invoke and indicate errors. However, it can also be used with `async` test functions, meaning that we can throw errors in the normal way and Mocha will automatically capture those errors to determine if the test fails.

Generally, Mocha looks to see if the function throws an exception or whether the test case takes too long to execute (a timeout situation). In either case, Mocha will indicate a test failure. That's, of course, simple to determine for non-asynchronous code. But Node.js is all about asynchronous code, and Mocha has two models for testing asynchronous code. In the first (not seen here), Mocha passes in a callback function, and the test code is to call the callback function. In the second, as seen here, it looks for a Promise being returned by the test function and determines a pass/fail regarding whether the Promise is in the `resolve` or `reject` state.

We are keeping the NotesStore model in the global `store` variable so that it can be used by all tests. The test, in this case, is whether we can load a given NotesStore implementation. As the comment states, if this executes without throwing an exception, the test has succeeded.  The other purpose of this test is to initialize the variable for use by other test cases.

It is useful to notice that this code carefully avoids loading `app.mjs`. Instead, it loads the test driver module, `models/notes-store.mjs`, and whatever module is loaded by `useNotesModel`. The `NotesStore` implementation is what's being tested, and the spirit of unit testing says to isolate it as much as possible.

Before we proceed further, let's talk about how Mocha structures tests.

With Mocha, a test suite is contained within a `describe` block. The first argument is a piece of descriptive text that you use to tailor the presentation of test results. The second argument is a `function` that contains the contents of the given test suite.

The `it` function is a test case. The intent is for us to read this as *it should successfully load the module*. Then, the code within the `function` is used to check that assertion.

With Mocha, it is important to not use arrow functions in the `describe` and `it` blocks. By now, you will have grown fond of arrow functions because of how much easier they are to write. However, Mocha calls these functions with a `this` object containing useful functions for Mocha. Because arrow functions avoid setting up a `this` object, Mocha would break.

Now that we have a test case written, let's learn how to run tests.

### Running the first test case

Now that we have a test case, let's run the test. In the `package.json` file, add the following `scripts` section:

```

我们在这里做的是创建一个`test-all`脚本，它将针对各个 NotesStore 实现运行测试套件。我们可以运行此脚本来运行每个测试组合，或者我们可以运行特定脚本来测试只有一个组合。例如，`test-notes-sequelize-sqlite`将针对使用 SQLite3 数据库的`SequelizeNotesStore`运行测试。

它使用`npm-run-all`来支持按顺序运行测试。通常，在`package.json`脚本中，我们会这样写：

```

This runs a series of steps one after another, relying on a feature of the Bash shell. The `npm-run-all` tool serves the same purpose, namely running one `package.json` script after another in the series. The first advantage is that the code is simpler and more compact, making it easier to read, while the other advantage is that it is cross-platform. We're using `cross-env` for the same purpose so that the test scripts can be executed on Windows as easily as they can be on Linux or macOS.

For the `test-notes-sequelize-sqlite` test, look closely. Here, you can see that we need a database configuration file named `sequelize-sqlite.yaml`. Create that file with the following code:

```

正如测试脚本名称所示，这使用 SQLite3 作为底层数据库，并将其存储在指定的文件中。

我们缺少两种组合，`test-notes-sequelize-mysql`用于使用 MySQL 的`SequelizeNotesStore`和`test-notes-mongodb`，它测试`MongoDBNotesStore`。我们稍后将实现这些组合。

自动运行所有测试组合后，我们可以尝试一下：

```

If all has gone well, you'll get this result for every test combination currently supported in the `test-all` script.

This completes the first test, which was to demonstrate how to create tests and execute them. All that remains is to write more tests.

### Adding some tests

That was easy, but if we want to find what bugs we created, we need to test some functionality. Now, let's create a test suite for testing `NotesStore`, which will contain several test suites for different aspects of `NotesStore`.

What does that mean? Remember that the `describe` function is the container for a test suite and that the `it` function is the container for a test case. By simply nesting `describe` functions, we can contain a test suite within a test suite. It will be clearer what that means after we implement this:

```

在这里，我们有一个`describe`函数，它定义了一个包含另一个`describe`函数的测试套件。这是嵌套测试套件的结构。

目前在`it`函数中没有测试用例，但是我们有`before`和`after`函数。这两个函数的功能就像它们的名字一样；`before`函数在所有测试用例之前运行，而`after`函数在所有测试用例完成后运行。`before`函数旨在设置将被测试的条件，而`after`函数旨在进行拆卸。

在这种情况下，`before`函数向`NotesStore`添加条目，而`after`函数删除所有条目。其想法是在每个嵌套测试套件执行后都有一个干净的状态。

`before`和`after`函数是 Mocha 称之为钩子的函数。其他钩子是`beforeEach`和`afterEach`。区别在于`Each`钩子在每个测试用例执行之前或之后触发。

这两个钩子也充当测试用例，因为`create`和`destroy`方法可能会失败，如果失败，钩子也会失败。

在`before`和`after`钩子函数之间，添加以下测试用例：

```

As suggested by the description for this test suite, the functions all test the `keylist` method.

For each test case, we start by calling `keylist`, then using `assert` methods to check different aspects of the array that is returned. The idea is to call `NotesStore` API functions, then test the results to check whether they matched the expected results.

Now, we can run the tests and get the following:

```

将输出与`describe`和`it`函数中的描述字符串进行比较。您会发现，此输出的结构与测试套件和测试用例的结构相匹配。换句话说，我们应该将它们结构化，使其具有良好结构化的测试输出。

正如他们所说，测试永远不会完成，只会耗尽。所以，在我们耗尽之前，让我们看看我们能走多远。

### Notes 模型的更多测试

这还不足以进行太多测试，所以让我们继续添加一些测试：

```

These tests check the `read` method. In the first test case, we check whether it successfully reads a known Note, while in the second test case, we have a negative test of what happens if we read a non-existent Note.

Negative tests are very important to ensure that functions fail when they're supposed to fail and that failures are indicated correctly.

The Chai Assertions API includes some very expressive assertions. In this case, we've used the `deepEqual` method, which does a deep comparison of two objects. You'll see that for the first argument, we pass in an object and that for the second, we pass an object that's used to check the first. To see why this is useful, let's force it to indicate an error by inserting `FAIL` into one of the test strings.

After running the tests, we get the following output:

```

这就是失败的测试样子。没有勾号，而是一个数字，数字对应下面的报告。在失败报告中，`deepEqual`函数为我们提供了关于对象字段差异的清晰信息。在这种情况下，这是我们故意让`deepEqual`函数失败的测试，因为我们想看看它是如何工作的。

请注意，对于负面测试——如果抛出错误则测试通过——我们在`try/catch`块中运行它。每种情况中的`throw new Error`行不应该执行，因为前面的代码应该抛出一个错误。因此，我们可以检查抛出的错误中的消息是否是到达的消息，并且如果是这种情况，则使测试失败。

### 诊断测试失败

我们可以添加更多的测试，因为显然，这些测试还不足以能够将 Notes 发布给公众。这样做之后，运行测试以针对不同的测试组合，我们将在 SQLite3 组合的结果中找到这个结果：

```

Our test suite found two errors, one of which is the error we mentioned in Chapter 7, *Data Storage and Retrieval*. Both failures came from the negative test cases. In one case, the test calls `store.read("badkey12")`, while in the other, it calls `store.delete("badkey12")`.

It is easy enough to insert `console.log` calls and learn what is going on.

For the `read` method, SQLite3 gave us `undefined` for `row`. The test suite successfully calls the `read` function multiple times with a `notekey` value that does exist. Obviously, the failure is limited to the case of an invalid `notekey` value. In such cases, the query gives an empty result set and SQLite3 invokes the callback with `undefined` in both the error and the row values. Indeed, the equivalent `SQL SELECT` statement does not throw an error; it simply returns an empty result set. An empty result set isn't an error, so we received no error and an undefined `row`.

However, we defined `read` to throw an error if no such Note exists. This means this function must be written to detect this condition and throw an error.

There is a difference between the `read` functions in `models/notes-sqlite3.mjs` and `models/notes-sequelize.mjs`. On the day we wrote `SequelizeNotesStore`, we must have thought through this function more carefully than we did on the day we wrote `SQLITE3NotesStore`. In `SequelizeNotesStore.read`, there is an error that's thrown when we receive an empty result set, and it has a check that we can adapt. Let's rewrite the `read` function in `models/notes-sqlite.mjs` so that it reads as follows:

```

如果这收到一个空结果，就会抛出一个错误。虽然数据库不会将空结果集视为错误，但 Notes 会。此外，Notes 已经知道如何处理这种情况下抛出的错误。进行这个更改，那个特定的测试用例就会通过。

`destroy`逻辑中还有第二个类似的错误。在 SQL 中，如果这个 SQL（来自`models/notes-sqlite3.mjs`）没有删除任何内容，这显然不是一个 SQL 错误：

```

Unfortunately, there isn't a method in the SQL option to fail if it does not delete any records. Therefore, we must add a check to see if a record exists, namely the following:

```

因此，我们读取笔记，并且作为副产品，我们验证笔记是否存在。如果笔记不存在，`read`将抛出一个错误，而`DELETE`操作甚至不会运行。

当我们运行`test-notes-sequelize-sqlite`时，它的`destroy`方法也出现了类似的失败。在`models/notes-sequelize.mjs`中，进行以下更改：

```

This is the same change; that is, to first `read` the Note corresponding to the given `key`, and if the Note does not exist, to throw an error.

Likewise, when running `test-level`, we get a similar failure, and the solution is to edit `models/notes-level.mjs` to make the following change:

```

与其他 NotesStore 实现一样，在销毁之前先读取 Note。如果`read`操作失败，那么测试用例会看到预期的错误。

这些是我们在第七章中提到的错误，*数据存储和检索*。我们只是忘记在这个特定模型中检查这些条件。幸运的是，我们勤奋的测试捕捉到了问题。至少，这是要告诉经理的故事，而不是告诉他们我们忘记检查我们已经知道可能发生的事情。

### 针对需要服务器设置的数据库进行测试——MySQL 和 MongoDB

这很好，但显然我们不会在生产中使用 SQLite3 或 Level 等数据库运行 Notes。我们可以在 Sequelize 支持的 SQL 数据库（如 MySQL）和 MongoDB 上运行 Notes。显然，我们疏忽了没有测试这两种组合。

我们的测试结果矩阵如下：

+   `notes-fs`: 通过

+   `notes-memory`: 通过

+   `notes-level`: 1 个失败，现已修复

+   `notes-sqlite3`: 2 个失败，现已修复

+   `notes-sequelize`: 使用 SQLite3：1 个失败，现已修复

+   `notes-sequelize`: 使用 MySQL：未经测试

+   `notes-mongodb`: 未经测试

两个未经测试的 NotesStore 实现都需要我们设置一个数据库服务器。我们避免测试这些组合，但我们的经理不会接受这个借口，因为 CEO 需要知道我们已经完成了测试周期。笔记必须使用类似于生产环境的配置进行测试。

在生产中，我们将使用常规的数据库服务器，MySQL 或 MongoDB 是主要选择。因此，我们需要一种低开销的方式来对这些数据库进行测试。对生产配置进行测试必须如此简单，以至于我们在进行测试时不会感到阻力，以确保测试运行得足够频繁，以产生期望的影响。

在本节中，我们取得了很大的进展，并在 NotesStore 数据库模块的测试套件上有了一个良好的开端。我们学会了如何在 Mocha 中设置测试套件和测试用例，以及如何获得有用的测试报告。我们学会了如何使用`package.json`来驱动测试套件执行。我们还学会了负面测试场景以及如何诊断出现的错误。

但我们需要解决针对数据库服务器进行测试的问题。幸运的是，我们已经使用了一个支持轻松创建和销毁部署基础设施的技术。你好，Docker！

在下一节中，我们将学习如何将 Docker Compose 部署重新用作测试基础设施。

# 使用 Docker Swarm 管理测试基础设施

Docker 给我们带来的一个优势是能够在我们的笔记本电脑上安装生产环境。在第十二章中，*使用 Terraform 将 Docker Swarm 部署到 AWS EC2*，我们将一个在我们的笔记本电脑上运行的 Docker 设置转换为可以部署在真实云托管基础设施上的设置。这依赖于将 Docker Compose 文件转换为 Docker Stack 文件，并对我们在 AWS EC2 实例上构建的环境进行定制。

在本节中，我们将将 Stack 文件重新用作部署到 Docker Swarm 的测试基础设施。一种方法是简单地运行相同的部署，到 AWS EC2，并替换`var.project_name`和`var.vpc_name`变量的新值。换句话说，EC2 基础设施可以这样部署：

```

This would deploy a second VPC with a different name that's explicitly for test execution and that would not disturb the production deployment. It's quite common in Terraform to customize the deployment this way for different targets.

In this section, we'll try something different. We can use Docker Swarm in other contexts, not just the AWS EC2 infrastructure we set up. Specifically, it is easy to use Docker Swarm with the Docker for Windows or Docker for macOS that's running on our laptop.

What we'll do is configure Docker on our laptop so that it supports swarm mode and create a slightly modified version of the Stack file in order to run the tests on our laptop. This will solve the issue of running tests against a MySQL database server, and also lets us test the long-neglected MongoDB module. This will demonstrate how to use Docker Swarm for test infrastructure and how to perform semi-automated test execution inside the containers using a shell script.

Let's get started.

## Using Docker Swarm to deploy test infrastructure

We had a great experience using Docker Compose and Swarm to orchestrate Notes application deployment on both our laptop and our AWS infrastructure. The whole system, with five independent services, is easily described in `compose-local/docker-compose.yml` and `compose-swarm/docker-compose.yml`. What we'll do is duplicate the Stack file, then make a couple of small changes required to support test execution in a local swarm.

To configure the Docker installation on our laptop for swarm mode, simply type the following:

```

与以前一样，这将打印有关加入令牌的消息。如果需要的话，如果你的办公室有多台电脑，你可能会对设置本地 Swarm 进行实验感兴趣。但对于这个练习来说，这并不重要。这是因为我们可以用单节点 Swarm 完成所有需要的工作。

这不是单行道，这意味着当你完成这个练习时，关闭 swarm 模式是很容易的。只需关闭部署到本地 Swarm 的任何内容，并运行以下命令：

```

Normally, this is used for a host that you wish to detach from an existing swarm. If there is only one host remaining in a swarm, the effect will be to shut down the swarm.

Now that we know how to initialize swarm mode on our laptop, let's set about creating a stack file suitable for use on our laptop.

Create a new directory, `compose-stack-test-local`, as a sibling to the `notes`, `users`, and `compose-local` directories. Copy `compose-stack/docker-compose.yml` to that directory. We'll be making several small changes to this file and no changes to the existing Dockerfiles. As much as it is possible, it is important to test the same containers that are used in the production deployment. This means it's acceptable to inject test files into the containers, but not modify them.

Make every `deploy` tag look like this:

```

这将删除我们在 AWS EC2 上声明的放置约束，并将其设置为每个服务的一个副本。对于单节点集群，当然我们不用担心放置，也没有必要多个服务实例。

对于数据库服务，删除`volumes`标签。当需要在数据库数据目录中持久保存数据时，使用此标签是必需的。对于测试基础设施，数据目录并不重要，可以随意丢弃。同样，删除顶级`volumes`标签。

对于`svc-notes`和`svc-userauth`服务，进行以下更改：

```

This injects the files required for testing into the `svc-notes` container. Obviously, this is the `test` directory that we created in the previous section for the Notes service. Those tests also require the SQLite3 schema file since it is used by the corresponding test script. In both cases, we can use `bind` mounts to inject the files into the running container.

The Notes test suite follows a normal practice for Node.js projects of putting `test` files in the test directory. When building the container, we obviously don't include the test files because they're not required for deployment. But running tests requires having that directory inside the running container. Fortunately, Docker makes this easy. We simply mount the directory into the correct place.

The bottom line is this approach gives us the following advantages:

*   The test code is in `notes/test`, where it belongs.
*   The test code is not copied into the production container.
*   In test mode, the `test` directory appears where it belongs.

For Docker (using `docker run`) and Docker Compose, the volume is mounted from a directory on the localhost. But for swarm mode, with a multi-node swarm, the container could be deployed on any host matching the placement constraints we declare. In a swarm, bind volume mounts like the ones shown here will try to mount from a directory on the host that the container has been deployed in. But we are not using a multi-node swarm; instead, we are using a single-node swarm. Therefore, the container will mount the named directory from our laptop, and all will be fine. But as soon as we decide to run testing on a multi-node swarm, we'll need to come up with a different strategy for injecting these files into the container.

We've also changed the `ports` mappings. For `svc-userauth`, we've made its port visible to give ourselves the option of testing the REST service from the host computer. For the `svc-notes` service, this will make it appear on port `3000`. In the `environment` section, make sure you did not set a `PORT` variable. Finally, we adjust `TWITTER_CALLBACK_HOST` so that it uses `localhost:3000` since we're deploying on the localhost.

For both services, we're changing the image tag from the one associated with the AWS ECR repository to one of our own designs. We won't be publishing these images to an image repository, so we can use any image tag we like.  

For both services, we are using the Sequelize data model, using the existing MySQL-oriented configuration file, and setting the `SEQUELIZE_DBHOST` variable to refer to the container holding the database. 

We've defined a Docker Stack file that should be useful for deploying the Notes application stack in a Swarm. The difference between the deployment on AWS EC2 and here is simply the configuration. With a few simple configuration changes, we've mounted test files into the appropriate container, reconfigured the volumes and the environment variables, and changed the deployment descriptors so that they're suitable for a single-node swarm running on our laptop.

Let's deploy this and see how well we did.

## Executing tests under Docker Swarm

We've repurposed our Docker Stack file so that it describes deploying to a single-node swarm, ensuring the containers are set up to be useful for testing. Our next step is to deploy the Stack to a swarm and execute the tests inside the Notes container.

To set it up, run the following commands:

```

我们运行`swarm init`在我们的笔记本上打开 swarm 模式，然后将两个`TWITTER`秘密添加到 swarm 中。由于它是单节点 swarm，我们不需要运行`docker swarm join`命令来添加新节点到 swarm 中。

然后，在`compose-stack-test-local`目录中，我们可以运行这些命令：

```

Because a Stack file is also a Compose file, we can run `docker-compose build` to build the images. Because of the `image` tags, this will automatically tag the images so that they match the image names we specified.

Then, we use `docker stack deploy`, as we did when deploying to AWS EC2\. Unlike the AWS deployment, we do not need to push the images to repositories, which means we do not need to use the `--with-registry-auth` option. This will behave almost identically to the swarm we deployed to EC2, so we explore the deployed services in the same way:

```

因为这是单主机 swarm，我们不需要使用 SSH 访问 swarm 节点，也不需要使用`docker context`设置远程访问。相反，我们运行 Docker 命令，它们会在本地主机上的 Docker 实例上执行。 

`docker ps`命令将告诉我们每个服务的精确容器名称。有了这个知识，我们可以运行以下命令来获得访问权限：

```

Because, in swarm mode, the containers have unique names, we have to run `docker ps` to get the container name, then paste it into this command to start a Bash shell inside the container.

Inside the container, we see the `test` directory is there as expected. But we have a couple of setup steps to perform. The first is to install the SQLite3 command-line tools since the scripts in `package.json` use that command. The second is to remove any existing `node_modules` directory because we don't know if it was built for this container or for the laptop. After that, we need to run `npm install` to install the dependencies.

Having done this, we can run the tests:

```

测试应该像在我们的笔记本电脑上一样执行，但是它们是在容器内运行的。但是，MySQL 测试不会运行，因为`package.json`脚本没有设置自动运行。因此，我们可以将其添加到`package.json`中：

```

This is the command that's required to execute the test suite against the MySQL database.

Then, we can run the tests against MySQL, like so:

```

测试应该对 MySQL 执行正确。

为了自动化这一过程，我们可以创建一个名为`run.sh`的文件，其中包含以下代码：

```

The script executes each script in `notes/test/package.json` individually. If you prefer, you can replace these with a single line that executes `npm run test-all`.

This script takes a command-line argument for the container name holding the `svc-notes` service. Since the tests are located in that container, that's where the tests must be run. The script can be executed like so:

```

这运行了前面的脚本，将每个测试组合单独运行，并确保`DEBUG`变量未设置。这个变量在 Dockerfile 中设置，会导致在测试结果输出中打印调试信息。在脚本中，`--workdir`选项将命令的当前目录设置为`test`目录，以简化运行测试脚本。

当然，这个脚本在 Windows 上不会直接执行。要将其转换为 PowerShell 使用，将从第二行开始的文本保存到`run.ps1`中，然后将`SVC_NOTES`引用更改为`%SVC_NOTES%`引用。

我们已经成功地将大部分测试矩阵的执行部分自动化。但是，测试矩阵中存在一个明显的漏洞，即缺乏对 MongoDB 的测试。填补这个漏洞将让我们看到如何在 Docker 下设置 MongoDB。

### 在 Docker 下设置 MongoDB 并对 Notes 进行测试

在第七章，*数据存储和检索*中，我们为 Notes 开发了 MongoDB 支持。从那时起，我们专注于`Sequelize`。为了弥补这一点，让我们确保至少测试我们的 MongoDB 支持。在 MongoDB 上进行测试只需要定义一个 MongoDB 数据库的容器和一点配置。

访问[`hub.docker.com/_/mongo/`](https://hub.docker.com/_/mongo/)获取官方 MongoDB 容器。您可以将其改装以部署在 MongoDB 上运行的 Notes 应用程序。

将以下代码添加到`compose-stack-test-local/docker-compose.yml`中：

```

That's all that's required to add a MongoDB container to a Docker Compose/Stack file. We've connected it to `frontnet` so that the database is accessible by `svc-notes`. If we wanted the `svc-notes` container to use MongoDB, we'd need some environment variables (`MONGO_URL`, `MONGO_DBNAME`, and `NOTES_MODEL`) to tell Notes to use MongoDB. 

But we'd also run into a problem that we created for ourselves in Chapter 9, *Dynamic Client/Server Interaction with Socket.IO*. In that chapter, we created a messaging subsystem so that our users can leave messages for each other. That messaging system is currently implemented to store messages in the same Sequelize database where the Notes are stored. But to run Notes with no Sequelize database would mean a failure in the messaging system. Obviously, the messaging system can be rewritten, for instance, to allow storage in a MongoDB database, or to support running both MongoDB and Sequelize at the same time.

Because we were careful, we can execute code in `models/notes-mongodb.mjs` without it being affected by other code. With that in mind, we'll simply execute the Notes test suite against MongoDB and report the results.

Then, in `notes/test/package.json`, we can add a line to facilitate running tests on MongoDB:

```

我们只是将 MongoDB 容器添加到了`frontnet`，使得数据库可以在此处显示的 URL 上使用。因此，现在可以简单地使用 Notes MongoDB 模型运行测试套件。

`--no-timeouts`选项是必要的，以避免针对 MongoDB 测试套件时出现错误。此选项指示 Mocha 不检查测试用例执行是否太长时间。

最后的要求是将以下一行添加到`run.sh`（或`run.ps1`适用于 Windows）中：

```

This ensures MongoDB can be tested alongside the other test combinations. But when we run this, an error might crop up:

```

问题在于 MongoClient 对象的初始化程序略有变化。因此，我们必须修改`notes/models/notes-mongodb.mjs`，使用这个新的`connectDB`函数：

```

This adds a pair of useful configuration options, including the option explicitly named in the error message. Otherwise, the code is unchanged.

To make sure the container is running with the updated code, rerun the `docker-compose build` and `docker stack deploy` steps shown earlier. Doing so rebuilds the images, and then updates the services. Because the `svc-notes` container will relaunch, you'll need to install the Ubuntu `sqlite3` package again.

Once you've done that, the tests will all execute correctly, including the MongoDB combination.

We can now report the final test results matrix to the manager:

*   `models-fs`: PASS
*   `models-memory`: PASS
*   `models-levelup`: 1 failure, now fixed, PASS
*   `models-sqlite3`: Two failures, now fixed, PASS
*   `models-sequelize` with SQLite3: 1 failure, now fixed, PASS
*   `models-sequelize` with MySQL: PASS
*   `models-mongodb`: PASS

The manager will tell you "good job" and then remember that the models are only a portion of the Notes application. We've left two areas completely untested:

*   The REST API for the user authentication service
*   Functional testing of the user interface

In this section, we've learned how to repurpose a Docker Stack file so that we can launch the Notes stack on our laptop. It took a few simple reconfigurations of the Stack file and we were ready to go, and we even injected the files that are useful for testing. With a little bit more work, we finished testing against all configuration combinations of the Notes database modules.

Our next task is to handle testing the REST API for the user authentication service.

# Testing REST backend services

It's now time to turn our attention to the user authentication service. We've mentioned testing this service, saying that we'll get to them later. We developed a command-line tool for both administration and ad hoc testing. While that has been useful all along, it's time to get cracking with some real tests.

There's a question of which tool to use for testing the authentication service. Mocha does a good job of organizing a series of test cases, and we should reuse it here. But the thing we have to test is a REST service. The customer of this service, the Notes application, uses it through the REST API, giving us a perfect rationalization to test the REST interface rather than calling the functions directly. Our ad hoc scripts used the SuperAgent library to simplify making REST API calls. There happens to be a companion library, SuperTest, that is meant for REST API testing. It's easy to use that library within a Mocha test suite, so let's take that route.

For the documentation on SuperTest, look here: [`www.npmjs.com/package/supertest`](https://www.npmjs.com/package/supertest).

Create a directory named `compose-stack-test-local/userauth`. This directory will contain a test suite for the user authentication REST service. In that directory, create a file named `test.mjs` that contains the following code:

```

这设置了 Mocha 和 SuperTest 客户端。`URL_USERS_TEST`环境变量指定了要针对其运行测试的服务器的基本 URL。鉴于我们之前使用的配置，您几乎肯定会使用`http://localhost:5858`，但它可以是指向任何主机的任何 URL。SuperTest 的初始化方式与 SuperAgent 略有不同。

`SuperTest`模块提供了一个函数，我们使用`URL_USERS_TEST`变量调用该函数。这给了我们一个对象，我们称之为`request`，用于与正在测试的服务进行交互。

我们还设置了一对变量来存储认证用户 ID 和密钥。这些值与用户认证服务器中的值相同。我们只需要在进行 API 调用时提供它们。

最后，这是 Mocha 测试套件的外壳。所以，让我们开始填写`before`和`after`测试用例：

```

These are our `before` and `after` tests. We'll use them to establish a user and then clean them up by removing the user at the end.

This gives us a taste of how the `SuperTest` API works. If you refer back to `cli.mjs`, you'll see the similarities to `SuperAgent`.

The `post` and `delete` methods we can see here declare the HTTP verb to use. The `send` method provides an object for the `POST` operation. The `set` method sets header values, while the `auth` method sets up authentication:

```

现在，我们可以测试一些 API 方法，比如`/list`操作。

我们已经保证在`before`方法中有一个帐户，所以`/list`应该给我们一个包含一个条目的数组。

这遵循了使用 Mocha 测试 REST API 方法的一般模式。首先，我们使用 SuperTest 的`request`对象调用 API 方法并`await`其结果。一旦我们得到结果，我们使用`assert`方法来验证它是否符合预期。

添加以下测试用例：

```

We are checking the `/find` operation in two ways:

*   **Positive test**: Looking for the account we know exists – failure is indicated if the user account is not found
*   **Negative test**: Looking for the one we know does not exist – failure is indicated if we receive something other than an error or an empty object

Add the following test case:

```

最后，我们应该检查`/destroy`操作。这个操作已经在`after`方法中检查过，我们在那里`destroy`了一个已知的用户帐户。我们还需要执行负面测试，并验证其对我们知道不存在的帐户的行为。

期望的行为是抛出错误或结果显示一个指示错误的 HTTP `status`。实际上，当前的认证服务器代码给出了 500 状态码，以及其他一些信息。

这给了我们足够的测试来继续并自动化测试运行。

在`compose-stack-test-local/docker-compose.yml`中，我们需要将`test.js`脚本注入到`svc-userauth-test`容器中。我们将在这里添加：

```

This injects the `userauth` directory into the container as the `/userauth/test` directory. As we did previously, we then must get into the container and run the test script.

The next step is creating a `package.json` file to hold any dependencies and a script to run the test:

```

在依赖项中，我们列出了 Mocha，Chai，SuperTest 和 cross-env。然后，在`test`脚本中，我们运行 Mocha 以及所需的环境变量。这应该运行测试。

我们可以从我们的笔记本电脑使用这个测试套件。因为测试目录被注入到容器中，我们也可以在容器内运行它们。要这样做，将以下代码添加到`run.sh`中：

```

This adds a second argument – in this case, the container name for `svc-userauth`. We can then run the test suite, using this script to run them inside the container. The first two commands ensure the installed packages were installed for the operating system in this container, while the last runs the test suite.

Now, if you run the `run.sh` test script, you'll see the required packages get installed. Then, the test suite will be executed.

The result will look like this:

```

因为`URL_USERS_TEST`可以使用任何 URL，我们可以针对用户认证服务的任何实例运行测试套件。例如，我们可以使用适当的`URL_USERS_TEST`值从我们的笔记本电脑上测试在 AWS EC2 上部署的实例。

我们取得了很好的进展。我们现在已经为笔记和用户认证服务准备了测试套件。我们已经学会了如何使用 REST API 测试 REST 服务。这与直接调用内部函数不同，因为它是对完整系统的端到端测试，扮演服务的消费者角色。

我们的下一个任务是自动化测试结果报告。

# 自动化测试结果报告

我们已经自动化了测试执行，Mocha 通过所有这些勾号使测试结果看起来很好。但是，如果管理层想要一个显示测试失败趋势的图表怎么办？报告测试结果作为数据而不是作为控制台上的用户友好的打印输出可能有很多原因。

例如，测试通常不是在开发人员的笔记本电脑上运行，也不是由质量团队的测试人员运行，而是由自动化后台系统运行。CI/CD 模型被广泛使用，其中测试由 CI/CD 系统在每次提交到共享代码存储库时运行。当完全实施时，如果在特定提交上所有测试都通过，那么系统将自动部署到服务器，可能是生产服务器。在这种情况下，用户友好的测试结果报告是没有用的，而必须以数据的形式传递，可以在 CI/CD 结果仪表板网站上显示。

Mocha 使用所谓的**Reporter**来报告测试结果。Mocha Reporter 是一个模块，以其支持的任何格式打印数据。有关此信息的更多信息可以在 Mocha 网站上找到：[`mochajs.org/#reporters`](https://mochajs.org/#reporters)。

您将找到当前可用的`reporters`列表如下：

```

Then, you can use a specific Reporter, like so:

```

在`npm run script-name`命令中，我们可以注入命令行参数，就像我们在这里所做的那样。`--`标记告诉 npm 将其命令行的其余部分附加到执行的命令上。效果就像我们运行了这个命令：

```

For Mocha, the `--reporter` option selects which Reporter to use. In this case, we selected the TAP reporter, and the output follows that format.

**Test Anything Protocol** (**TAP**) is a widely used test results format that increases the possibility of finding higher-level reporting tools. Obviously, the next step would be to save the results into a file somewhere, after mounting a host directory into the container.

In this section, we learned about the test results reporting formats supported by Mocha. This will give you a starting point for collecting long-term results tracking and other useful software quality metrics. Often, software teams rely on quality metrics trends as part of deciding whether a product can be shipped to the public.

In the next section, we'll round off our tour of testing methodologies by learning about a framework for frontend testing.

# Frontend headless browser testing with Puppeteer

A big cost area in testing is manual user interface testing. Therefore, a wide range of tools has been developed to automate running tests at the HTTP level. Selenium is a popular tool implemented in Java, for example. In the Node.js world, we have a few interesting choices. The *chai-http* plugin to Chai would let us interact at the HTTP level with the Notes application while staying within the now-familiar Chai environment. 

However, in this section, we'll use Puppeteer ([`github.com/GoogleChrome/puppeteer`](https://github.com/GoogleChrome/puppeteer)). This tool is a high-level Node.js module used to control a headless Chrome or Chromium browser, using the DevTools protocol. This protocol allows tools to instrument, inspect, debug, and profile Chromium or Chrome browser instances. The key result is that we can test the Notes application in a real browser so that we have greater assurance it behaves correctly for users. 

The Puppeteer website has extensive documentation that's worth reading: [`pptr.dev/`](https://pptr.dev/).

Puppeteer is meant to be a general-purpose test automation tool and has a strong feature set for that purpose. Because it's easy to make web page screenshots with Puppeteer, it can also be used in a screenshot service.

Because Puppeteer is controlling a real web browser, your user interface tests will be very close to live browser testing, without having to hire a human to do the work. Because it uses a headless version of Chrome, no visible browser window will show on your screen, and tests can be run in the background instead. It can also drive other browsers by using the DevTools protocol.

First, let's set up a directory to work in.

## Setting up a Puppeteer-based testing project directory

First, let's set up the directory that we'll install Puppeteer in, as well as the other packages that will be required for this project:

```

这不仅安装了 Puppeteer，还安装了 Mocha、Chai 和 Supertest。我们还将使用`package.json`文件记录脚本。

在安装过程中，您会发现 Puppeteer 会导致 Chromium 被下载，就像这样：

```

The Puppeteer package will launch that Chromium instance as needed, managing it as a background process and communicating with it using the DevTools protocol.

The approach we'll follow is to test against the Notes stack we've deployed in the test Docker infrastructure. Therefore, we need to launch that infrastructure:

```

根据您的需求，可能还需要执行`docker-compose build`。无论如何，这都会启动测试基础架构，并让您看到运行中的系统。

我们可以使用浏览器访问`http://localhost:3000`等网址。因为这个系统不包含任何用户，我们的测试脚本将不得不添加一个测试用户，以便测试可以登录并添加笔记。

另一个重要的事项是测试将在一个匿名的 Chromium 实例中运行。即使我们在正常的桌面浏览器中使用 Chrome，这个 Chromium 实例也与我们正常的桌面设置没有任何连接。从可测试性的角度来看，这是一件好事，因为这意味着您的测试结果不会受到个人网络浏览器配置的影响。另一方面，这意味着无法进行 Twitter 登录测试，因为该 Chromium 实例没有 Twitter 登录会话。

记住这些，让我们编写一个初始的测试套件。我们将从一个简单的初始测试用例开始，以证明我们可以在 Mocha 中运行 Puppeteer。然后，我们将测试登录和注销功能，添加笔记的能力，以及一些负面测试场景。我们将在本节中讨论如何改进 HTML 应用程序的可测试性。让我们开始吧。

## 为 Notes 应用程序堆栈创建一个初始的 Puppeteer 测试

我们的第一个测试目标是建立一个测试套件的大纲。我们需要按顺序执行以下操作：

1.  向用户身份验证服务添加一个测试用户。

1.  启动浏览器。

1.  访问首页。

1.  验证首页是否正常显示。

1.  关闭浏览器。

1.  删除测试用户。

这将确保我们有能力与启动的基础架构进行交互，启动浏览器并查看 Notes 应用程序。我们将继续执行策略并在测试后进行清理，以确保后续测试运行的干净环境，并添加，然后删除，一个测试用户。

在`notesui`目录中，创建一个名为`uitest.mjs`的文件，其中包含以下代码：

```

This imports and configures the required modules. This includes setting up `bcrypt` support in the same way that is used in the authentication server. We've also copied in the authentication key for the user authentication backend service. As we did for the REST test suite, we will use the `SuperTest` library to add, verify, and remove the test user using the REST API snippets copied from the REST tests.

Add the following test block:

```

这将向身份验证服务添加一个用户。回顾一下，您会发现这与 REST 测试套件中的测试用例类似。如果您需要验证阶段，还有另一个测试用例调用`/find/testme`端点来验证结果。由于我们已经验证了身份验证系统，因此我们不需要在这里重新验证它。我们只需要确保我们有一个已知的测试用户，可以在需要浏览器登录的场景中使用。

将此代码放在`uitest.mjs`的最后：

```

At the end of the test execution, we should run this to delete the test user. The policy is to clean up after we execute the test. Again, this was copied from the user authentication service test suite. Between those two, add the following:

```

记住，在`describe`中，测试是`it`块。`before`块在所有`it`块之前执行，`after`块在之后执行。

在`before`函数中，我们通过启动 Puppeteer 实例并启动一个新的 Page 对象来设置 Puppeteer。因为`puppeteer.launch`的`headless`选项设置为`false`，我们将在屏幕上看到一个浏览器窗口。这将很有用，因为我们可以看到发生了什么。`sloMo`选项也通过减慢浏览器交互来帮助我们看到发生了什么。在`after`函数中，我们调用这些对象的`close`方法来关闭浏览器。`puppeteer.launch`方法接受一个`options`对象，其中有很多值得学习的属性。

`browser`对象代表正在运行测试的整个浏览器实例。相比之下，`page`对象代表的是实质上是浏览器中当前打开的标签页。大多数 Puppeteer 函数都是异步执行的。因此，我们可以使用`async`函数和`await`关键字。

`timeout`设置是必需的，因为有时浏览器实例启动需要很长时间。我们慷慨地设置了超时时间，以最小化偶发测试失败的风险。

对于`it`子句，我们进行了少量的浏览器交互。作为浏览器标签页的包装器，`page`对象具有与管理打开标签页相关的方法。例如，`goto`方法告诉浏览器标签页导航到给定的 URL。在这种情况下，URL 是笔记主页，作为环境变量传递。

`waitForSelector`方法是一组等待特定条件的方法之一。这些条件包括`waitForFileChooser`、`waitForFunction`、`waitForNavigation`、`waitForRequest`、`waitForResponse`和`waitForXPath`。这些方法以及`waitFor`方法都会导致 Puppeteer 异步等待浏览器中发生的某些条件。这些方法的目的是给浏览器时间来响应某些输入，比如点击按钮。在这种情况下，它会等到网页加载过程中在给定的 CSS 选择器下有一个可见的元素。该选择器指的是在页眉中的登录按钮。

换句话说，这个测试访问笔记主页，然后等待直到登录按钮出现。我们可以称之为一个简单的冒烟测试，快速执行并确定基本功能是否存在。

### 执行初始的 Puppeteer 测试

我们已经启动了使用`docker-compose`的测试基础设施。要运行测试脚本，请将以下内容添加到`package.json`文件的脚本部分：

```

The test infrastructure we deployed earlier exposes the user authentication service on port `5858` and the Notes application on port `3000`. If you want to test against a different deployment, adjust these URLs appropriately. Before running this, the Docker test infrastructure must be launched, which should have already happened.

Let's try running this initial test suite:

```

我们已经成功地创建了可以运行这些测试的结构。我们已经设置了 Puppeteer 和相关的包，并创建了一个有用的测试。主要的收获是有一个结构可以在其基础上构建更多的测试。

我们的下一步是添加更多的测试。

## 在笔记中测试登录/注销功能

在上一节中，我们创建了测试笔记用户界面的大纲。关于应用程序的测试并不多，但我们证明了可以使用 Puppeteer 测试笔记。

在本节中，我们将添加一个实际的测试。也就是说，我们将测试登录和注销功能。具体步骤如下：

1.  使用测试用户身份登录。

1.  验证浏览器是否已登录。

1.  注销。

1.  验证浏览器是否已注销。

在`uitest.js`中，插入以下测试代码：

```

This is our test implementation for logging in and out. We have to specify the `timeout` value because it is a new `describe` block.

The `click` method takes a CSS selector, meaning this first click event is sent to the Login button. A CSS selector, as the name implies, is similar to or identical to the selectors we'd write in a CSS file. With a CSS selector, we can target specific elements on the page.

To determine the selector to use, look at the HTML for the templates and learn how to describe the element you wish to target. It may be necessary to add ID attributes into the HTML to improve testability.

The Puppeteer documentation refers to the CSS Selectors documentation on the Mozilla Developer Network website: [`developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors`](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors).

Clicking on the Login button will, of course, cause the Login page to appear. To verify this, we wait until the page contains a form that posts to `/users/login`. That form is in `login.hbs`.

The `type` method acts as a user typing text. In this case, the selectors target the `Username` and `Password` fields of the login form. The `delay` option inserts a pause of 100 milliseconds after typing each character. It was noted in testing that sometimes, the text arrived with missing letters, indicating that Puppeteer can type faster than the browser can accept.

The `page.keyboard` object has various methods related to keyboard events. In this case, we're asking to generate the equivalent to pressing *Enter* on the keyboard. Since, at that point, the focus is in the Login form, that will cause the form to be submitted to the Notes application. Alternatively, there is a button on that form, and the test could instead click on the button.

The `waitForNavigation` method has a number of options for waiting on page refreshes to finish. The selected option causes a wait until the DOM content of the new page is loaded.

The `$` method searches the DOM for elements matching the selector, returning an array of matching elements. If no elements match, `null` is returned instead. Therefore, this is a way to test whether the application got logged in, by looking to see if the page has a Logout button.

To log out, we click on the Logout button. Then, to verify the application logged out, we wait for the page to refresh and show a Login button:

```

有了这些，我们的新测试都通过了。请注意，执行一些测试所需的时间相当长。在调试测试时观察到了更长的时间，这就是我们设置长超时时间的原因。

这很好，但当然，还有更多需要测试的，比如添加笔记的能力。

## 测试添加笔记的能力

我们有一个测试用例来验证登录/注销功能。这个应用程序的重点是添加笔记，所以我们需要测试这个功能。作为副作用，我们将学习如何使用 Puppeteer 验证页面内容。

为了测试这个功能，我们需要按照以下步骤进行：

1.  登录并验证我们已经登录。

1.  点击“添加笔记”按钮进入表单。

1.  输入笔记的信息。

1.  验证我们是否显示了笔记，并且内容是正确的。

1.  点击删除按钮并确认删除笔记。

1.  验证我们最终进入了主页。

1.  注销。

你可能会想“*再次登录不是重复的吗*？”之前的测试集中在登录/注销上。当然，浏览器可能已经处于登录状态了吧？如果浏览器仍然登录，这个测试就不需要再次登录。虽然这是真的，但这会导致登录/注销场景的测试不完整。每个场景在用户是否登录方面都应该是独立的。为了避免重复，让我们稍微重构一下测试。

在`最外层`的描述块中，添加以下两个函数：

```

This is the same code as the code for the body of the test cases shown previously, but we've moved the code to their own functions. With this change, any test case that wishes to log into the test user can use these functions.

Then, we need to change the login/logout tests to this:

```

我们所做的只是将此处的代码移动到它们自己的函数中。这意味着我们可以在其他测试中重用这些函数，从而避免重复的代码。

将以下代码添加到`uitest.mjs`中的笔记创建测试套件：

```

These are our test cases for adding and deleting Notes. We start with the `doLogin` and `checkLogin` functions to ensure the browser is logged in.

After clicking on the Add Note button and waiting for the browser to show the form in which we enter the Note details, we need to enter text into the form fields. The `page.type` method acts as a user typing on a keyboard and types the given text into the field identified by the selector.

The interesting part comes when we verify the note being shown. After clicking the **Submit** button, the browser is, of course, taken to the page to view the newly created Note. To do this, we use `page.$eval` to retrieve text from certain elements on the screen.

The `page.$eval` method scans the page for matching elements, and for each, it calls the supplied callback function. The callback function is given the element, and in our case, we call the `textContent` method to retrieve the textual form of the element. Then, we're able to use the `assert.include` function to test that the element contains the required text.

The `page.url()` method, as its name suggests, returns the URL currently being viewed. We can test whether that URL contains `/notes/view` to be certain the browser is viewing a note.

To delete the note, we start by verifying that the **Delete** button is on the screen. Of course, this button is there if the user is logged in. Once the button is verified, we click on it and wait for the `FORM` that confirms that we want to delete the Note. Once it shows up, we can click on the button, after which we are supposed to land on the home page.

Notice that to find the Delete button, we need to refer to `a#notedestroy`. As it stands, the template in question does not have that ID anywhere. Because the HTML for the Delete button was not set up so that we could easily create a CSS selector, we must edit `views/noteedit.hbs` to change the Delete button to this:

```

我们所做的就是添加了 ID 属性。这是改进可测试性的一个例子，我们稍后会讨论。

我们使用的一种技术是调用`page.$`来查询给定元素是否在页面上。这种方法检查页面，返回一个包含任何匹配元素的数组。我们只是测试返回值是否非空，因为如果没有匹配元素，`page.$`会返回`null`。这是一种简单的测试元素是否存在的方法。

点击**注销**按钮退出登录。

创建了这些测试用例后，我们可以再次运行测试套件：

```

We have more passing tests and have made good progress. Notice how one of the test cases took 18 seconds to finish. That's partly because we slowed text entry down to make sure it is correctly received in the browser, and there is a fair amount of text to enter. There was a reason we increased the timeout.

In earlier tests, we had success with negative tests, so let's see if we can find any bugs that way.

## Implementing negative tests with Puppeteer

Remember that a negative test is used to purposely invoke scenarios that will fail. The idea is to ensure the application fails correctly, in the expected manner.

We have two scenarios for an easy negative test:

*   Attempt to log in using a bad user ID and password
*   Access a bad URL

Both of these are easy to implement, so let's see how it works.

### Testing login with a bad user ID

A simple way to ensure we have a bad username and password is to generate random text strings for both. An easy way to do that is with the `uuid` package. This package is about generating Universal Unique IDs (that is, UUIDs), and one of the modes of using the package simply generates a unique random string. That's all we need for this test; it is a guarantee that the string will be unique.

To make this crystal clear, by using a unique random string, we ensure that we don't accidentally use a username that might be in the database. Therefore, we will be certain of supplying an unknown username when trying to log in.

In `uitest.mjs`, add the following to the imports:

```

`uuid`包支持几种方法，`v4`方法是生成随机字符串的方法。

然后，添加以下场景：

```

This starts with the login scenario. Instead of a fixed username and password, we instead use the results of calling `uuidv4()`, or the random UUID string.

This does the login action, and then we wait for the resulting page. In trying this manually, we learn that it simply returns us to the login screen and that there is no additional message. Therefore, the test looks for the login form and ensures there is a Login button. Between the two, we are certain the user is not logged in.

We did not find a code error with this test, but there is a user experience error: namely, the fact that, for a failed login attempt, we simply show the login form and do not provide a message (that is, *unknown username or password*), which leads to a bad user experience. The user is left feeling confused over what just happened. So, let's put that on our backlog to fix.

### Testing a response to a bad URL 

Our next negative test is to try a bad URL in Notes. We coded Notes to return a 404 status code, which means the page or resource was not found. The test is to ask the browser to visit the bad URL, then verify that the result uses the correct error message.

Add the following test case:

```

通过获取主页的 URL（`NOTES_HOME_URL`）并将 URL 的*pathname*部分设置为`/bad-unknown-url`来计算错误的 URL。由于在笔记中没有这条路径，我们肯定会收到一个错误。如果我们想要更确定，似乎可以使用`uuidv4()`函数使 URL 变得随机。

调用`page.goto()`只是让浏览器转到请求的 URL。对于后续页面，我们等到出现一个带有`header`元素的页面。因为这个页面上没有太多内容，所以`header`元素是确定我们是否有了后续页面的最佳选择。

要检查 404 状态码，我们调用`response.status()`，这是在 HTTP 响应中收到的状态码。然后，我们调用`page.$eval`从页面中获取一些项目，并确保它们包含预期的文本。

在这种情况下，我们没有发现任何代码问题，但我们发现了另一个用户体验问题。错误页面非常丑陋且不友好。我们知道用户体验团队会对此大声抱怨，所以将其添加到待办事项中，以改进此页面。

在这一部分中，我们通过创建一些负面测试来结束了测试开发。虽然这并没有导致发现代码错误，但我们发现了一对用户体验问题。我们知道这将导致与用户体验团队进行不愉快的讨论，因此我们已经主动将修复这些页面的任务添加到了待办事项中。但我们也学会了随时留意沿途出现的任何问题。众所周知，由开发或测试团队发现的问题的修复成本最低。当用户社区报告问题时，修复问题的成本会大大增加。

在我们结束本章之前，我们需要更深入地讨论一下可测试性。

## 改进笔记 UI 的可测试性

虽然 Notes 应用程序在浏览器中显示良好，但我们如何编写测试软件来区分一个页面和另一个页面？正如我们在本节中看到的，UI 测试经常执行一个导致页面刷新的操作，并且必须等待下一个页面出现。这意味着我们的测试必须能够检查页面，并确定浏览器是否显示了正确的页面。一个错误的页面本身就是应用程序中的一个错误。一旦测试确定它是正确的页面，它就可以验证页面上的数据。

底线是，每个 HTML 元素必须能够轻松地使用 CSS 选择器进行定位。

虽然在大多数情况下，为每个元素编写 CSS 选择器很容易，在少数情况下，这很困难。**软件质量工程**（**SQE**）经理请求我们的帮助。涉及的是测试预算，SQE 团队能够自动化他们的测试，预算将被进一步拉伸。

所需的只是为 HTML 元素添加一些`id`或`class`属性，以提高可测试性。有了一些标识符和对这些标识符的承诺，SQE 团队可以编写可重复的测试脚本来验证应用程序。

我们已经看到了一个例子：`views/noteview.hbs`中的删除按钮。我们无法为该按钮编写 CSS 选择器，因此我们添加了一个 ID 属性，让我们能够编写测试。

总的来说，*可测试性*是为了软件质量测试人员的利益而向 API 或用户界面添加东西。对于 HTML 用户界面来说，这意味着确保测试脚本可以定位 HTML DOM 中的任何元素。正如我们所见，`id`和`class`属性在满足这一需求方面起到了很大作用。

在这一部分，我们学习了用户界面测试作为功能测试的一种形式。我们使用了 Puppeteer，一个用于驱动无头 Chromium 浏览器实例的框架，作为测试 Notes 用户界面的工具。我们学会了如何自动化用户界面操作，以及如何验证显示的网页是否与其正确的行为匹配。这包括覆盖登录、注销、添加笔记和使用错误的用户 ID 登录的测试场景。虽然这没有发现任何明显的失败，但观察用户交互告诉我们 Notes 存在一些可用性问题。

有了这些，我们准备结束本章。

# 总结

在本章中，我们涵盖了很多领域，并查看了三个不同的测试领域：单元测试、REST API 测试和 UI 功能测试。确保应用程序经过充分测试是通往软件成功的重要一步。一个不遵循良好测试实践的团队往往会陷入修复回归问题的泥潭。

首先，我们谈到了只使用断言模块进行测试的潜在简单性。虽然测试框架，比如 Mocha，提供了很好的功能，但我们可以用一个简单的脚本走得更远。

测试框架，比如 Mocha，有其存在的价值，至少是为了规范我们的测试用例并生成测试结果报告。我们用 Mocha 和 Chai 做到了这一点，这些工具非常成功。我们甚至在一个小的测试套件中发现了一些错误。

在开始单元测试之路时，一个设计考虑是模拟依赖关系。但并不总是一个好的做法用模拟版本替换每个依赖。因此，我们对一个实时数据库运行了我们的测试，但使用了测试数据。

为了减轻运行测试的行政负担，我们使用 Docker 来自动设置和拆除测试基础设施。就像 Docker 在自动部署 Notes 应用程序方面很有用一样，它在自动化测试基础设施部署方面也很有用。

最后，我们能够在真实的 Web 浏览器中测试 Notes 网络用户界面。我们不能指望单元测试能够找到每一个错误；有些错误只会在 Web 浏览器中显示。

在本书中，我们已经涵盖了 Node.js 开发的整个生命周期，从概念、通过各个开发阶段，到部署和测试。这将为您提供一个坚实的基础，从而开始开发 Node.js 应用程序。

在下一章中，我们将探讨另一个关键领域——安全性。我们将首先使用 HTTPS 对用户访问 Notes 进行加密和认证。我们将使用几个 Node.js 包来减少安全入侵的机会。
