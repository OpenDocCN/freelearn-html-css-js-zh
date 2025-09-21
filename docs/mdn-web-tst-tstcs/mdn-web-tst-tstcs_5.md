# 第五章：*第五章：改进测试*

本章的主要学习目标是熟悉如何改进一组端到端测试。这将通过测试设置和清理来实现。此外，我们还将查看不同的命令行设置来运行测试。本章中我们将涵盖的测试技术是通用的，可以重用来为任何 Web 项目编写自动化测试。到本章结束时，我们将拥有一个改进的测试套件，并学习如何使用命令行选项运行它。

本章我们将涵盖以下主要内容：

+   执行选定的测试。

+   探索测试设置和清理。

+   将设置和清理添加到测试项目中。

+   使用命令行设置运行测试。

# 技术要求

本章的所有代码示例都可以在 GitHub 上找到：[`github.com/PacktPublishing/Modern-Web-Testing-with-TestCafe/blob/master/ch5`](https://github.com/PacktPublishing/Modern-Web-Testing-with-TestCafe/blob/master/ch5)。

# 执行选定的测试

很常见，在编写或扩展一组测试时，我们需要专注于一个特定的测试，同时忽略所有其他测试。测试通常被组织成集合（测试组也称为固定装置）。幸运的是，TestCafe 提供了`fixture.only`和`test.only`方法来指定仅执行选定的测试或固定装置，而其他所有测试将被跳过。让我们以简化形式使用我们的测试集来回顾这一点，其中所有测试操作都被注释掉：

```js
// ...fixture('Redmine log in tests')    .page('http://demo.redmine.org/');test.only('Create a new user', async (t) => { /* ... */ });test('Log in', async (t) => { /* ... */ });test('Log out', async (t) => { /* ... */ });fixture('Redmine entities creation tests')    .page('http://demo.redmine.org/');test('Create a new project', async (t) => { /* ... */ });test('Create a new issue', async (t) => { /* ... */ });test('Verify that the issue is displayed on a project page', async (t) => { /* ... */ });test('Upload a file', async (t) => { /* ... */ });fixture('Redmine entities editing tests')    .page('http://demo.redmine.org/');test('Edit the issue', async (t) => { /* ... */ });test('Verify that the updated issue is displayed on a project page', async (t) => { /* ... */ });test('Search for the issue', async (t) => { /* ... */ });fixture.only('Redmine entities deletion tests')    .page('http://demo.redmine.org/');test('Delete the issue', async (t) => { /* ... */ });test('Delete the file', async (t) => { /* ... */ });
```

如示例所示，`test.only`在`创建新用户`测试中使用，而`fixture.only`在`Redmine 实体删除测试`固定装置中使用，因此只有`创建新用户`、`删除问题`和`删除文件`测试将被执行。

备注

如果有多个测试（或固定装置）被标记为`test.only`（或`fixture.only`），则所有标记的测试和固定装置都将被执行。

此外，TestCafe 允许您使用`test.skip`和`fixture.skip`方法来指定在运行测试时跳过的测试或固定装置：

```js
// ...fixture('Redmine log in tests')    .page('http://demo.redmine.org/');test('Create a new user', async (t) => { /* ... */ });test.skip('Log in', async (t) => { /* ... */ });test.skip('Log out', async (t) => { /* ... */ });fixture.skip('Redmine entities creation tests')    .page('http://demo.redmine.org/');test('Create a new project', async (t) => { /* ... */ });test('Create a new issue', async (t) => { /* ... */ });test('Verify that the issue is displayed on a project page', async (t) => { /* ... */ });test('Upload a file', async (t) => { /* ... */ });fixture('Redmine entities editing tests')    .page('http://demo.redmine.org/');test('Edit the issue', async (t) => { /* ... */ });test.skip('Verify that the updated issue is displayed on a project page', async (t) => { /* ... */ });test.skip('Search for the issue', async (t) => { /* ... */ });fixture.skip('Redmine entities deletion tests')    .page('http://demo.redmine.org/');test('Delete the issue', async (t) => { /* ... */ });test('Delete the file', async (t) => { /* ... */ });
```

如前例所示，只有`创建新用户`和`编辑问题`测试将被执行。

现在我们已经学会了如何执行特定的测试或固定装置，跳过所有其他测试，让我们看看如何进行测试设置和清理。

# 探索测试设置和清理

由于测试可能相当长且包含大量重复操作，TestCafe 通过测试设置和清理提供了一种优化方法。

设置通常是在固定装置或测试开始之前执行一系列特定函数（也称为钩子）时进行的（包括`fixture.before`、`fixture.beforeEach`和`test.before`）。

清理通常是在固定装置或测试完成后执行一系列特定函数时进行的（包括`fixture.after`、`fixture.afterEach`和`test.after`）。

在 TestCafe 中有六种使用钩子的方法。

前两个（`fixture.before` 和 `fixture.after`）没有访问测试页面，因此应用于执行服务器端操作，例如准备测试应用的服务器或预先在数据库中创建一些测试数据：

+   `fixture.before` 可以用来指定在 fixture 中的第一个测试开始之前应执行的操作 ([`devexpress.github.io/testcafe/documentation/reference/test-api/fixture/before.html`](https://devexpress.github.io/testcafe/documentation/reference/test-api/fixture/before.html)). 在以下示例中，`createTestData` 函数将在 `My first set of tests` fixture 的第一个测试之前被调用：

    ```js
    fixture('My first set of tests')    .page('https://test-site.com')    .before(async (t) => {        await createTestData();    });
    ```

+   `fixture.after` 可以用来指定在 fixture 中最后一个测试完成后应执行的操作 ([`devexpress.github.io/testcafe/documentation/reference/test-api/fixture/after.html`](https://devexpress.github.io/testcafe/documentation/reference/test-api/fixture/after.html)). 在以下示例中，`deleteTestData` 函数将在 `My first set of tests` fixture 的最后一个测试之后被调用：

    ```js
    fixture('My first set of tests')    .page('https://test-site.com')    .after(async (t) => {        await deleteTestData();    });
    ```

下面的四个方法（`fixture.beforeEach`、`fixture.afterEach`、`test.before` 和 `test.after`）在测试的网页已经加载时启动，因此您可以在这些测试钩子内部执行测试操作和其他测试 API 方法：

+   `fixture.beforeEach` 可以用来指定在 fixture 中的每个测试之前应执行的操作 ([`devexpress.github.io/testcafe/documentation/reference/test-api/fixture/beforeeach.html`](https://devexpress.github.io/testcafe/documentation/reference/test-api/fixture/beforeeach.html)). 在以下示例中，`click` 操作将在 `My first set of tests` fixture 的每个测试之前执行：

    ```js
    fixture('My first set of tests')    .page('https://test-site.com')    .beforeEach(async (t) => {        await t.click('#log-in');    });
    ```

+   `fixture.afterEach` 可以用来指定在 fixture 中的每个测试之后应执行的操作 ([`devexpress.github.io/testcafe/documentation/reference/test-api/fixture/aftereach.html`](https://devexpress.github.io/testcafe/documentation/reference/test-api/fixture/aftereach.html)). 在以下示例中，`click` 操作将在 `My first set of tests` fixture 的每个测试之后执行：

    ```js
    fixture('My first set of tests')    .page('https://test-site.com')    .afterEach(async (t) => {        await t.click('#delete-test-data');    });
    ```

+   `test.before` 可以用来指定在特定测试之前应执行的操作 ([`devexpress.github.io/testcafe/documentation/reference/test-api/test/before.html`](https://devexpress.github.io/testcafe/documentation/reference/test-api/test/before.html)). 在以下示例中，`click` 操作将在 `My first Test` 测试之前执行：

    ```js
    test     .before(async (t) => {        await t.click('#log-in');    })    ('My first Test', async (t) => { /* ... */ });
    ```

+   `test.after` 可以用来指定在特定测试之后应执行的操作 ([`devexpress.github.io/testcafe/documentation/reference/test-api/test/after.html`](https://devexpress.github.io/testcafe/documentation/reference/test-api/test/after.html)). 在以下示例中，`click` 操作将在 `My first Test` 测试之后执行：

    ```js
    test     .after(async (t) => {        await t.click('#delete-test-data');    })    ('My first Test', async (t) => { /* ... */ });
    ```

    注意

    如果一个测试在多个浏览器中运行，测试钩子将在每个浏览器中执行。如果同时使用了`fixture.beforeEach`和`test.before`（或`fixture.afterEach`和`test.after`）钩子，则最具体的钩子将覆盖。因此，`test.before`（或`test.after`）将被执行，`fixture.beforeEach`（或`fixture.afterEach`）将被省略，并且不会为此测试运行。

你可以在[`devexpress.github.io/testcafe/documentation/guides/basic-guides/organize-tests.html#initialization-and-clean-up`](https://devexpress.github.io/testcafe/documentation/guides/basic-guides/organize-tests.html#initialization-and-clean-up)了解更多关于钩子的信息。

在本节中，我们介绍了 TestCafe 中可用的钩子类型。现在，让我们将这一知识应用到我们的测试集中。

将设置和清理添加到测试项目

在本节中，我们将看到如何通过设置和清理块来优化我们的测试项目代码。

正如我们在*探索测试设置和清理*部分中看到的，`fixture.beforeEach`在需要用户在测试之前登录的每个测试中特别有用。这正是我们的情况，因此让我们将`beforeEach`块添加到`Redmine entities creation tests`测试用例中：

```js
// ...fixture('Redmine entities creation tests')    .page('http://demo.redmine.org/')    .beforeEach(async (t) => {        await t.click('.login')            .typeText('#username', `test_user_testcafe_poc${randomDigits1}@sharklasers.com`)            .typeText('#password', 'test_user_testcafe_poc')            .click('[name="login"]');    });
```

让我们也从`Redmine entities creation tests`测试用例的所有测试中移除登录操作，因为这些操作将在`beforeEach`块中执行。因此，`创建一个新项目`测试将看起来像这样：

```js
test('Create a new project', async (t) => {    await t.click('#top-menu .projects')        .click('.icon-add')        .typeText('#project_name', `test_project${randomDigits1}`)        .click('[value="Create"]')        .expect(Selector('#flash_notice').innerText).eql('Successful creation.')        .expect(getPageUrl()).contains(`/projects/test_project${randomDigits1}/settings`);});
```

在所有登录操作都移动到`beforeEach`块之后，`创建一个新问题`测试将看起来像这样：

```js
test('Create a new issue', async (t) => {    await t.click('#top-menu .projects')        .click('.icon-add')        .typeText('#project_name', `test_project${randomDigits2}`)        .click('[value="Create"]')        .click('#top-menu .projects')        .click(`[href*="/projects/test_project${randomDigits2}"]`)        .click('.new-issue')        .typeText('#issue_subject', `Test issue   ${randomDigits2}`)        .typeText('#issue_description', `Test issue description ${randomDigits2}`)        .click('#issue_priority_id')        .click(Selector('#issue_priority_id option').withText('High'))        .click('[value="Create"]')        .expect(Selector('#flash_notice').innerText).contains('created.');});
```

并且没有登录操作的`验证问题是否显示在项目页面上`测试将看起来像这样：

```js
test('Verify that the issue is displayed on a project page', async (t) => {    await t.click('#top-menu .projects')        .click('.icon-add')        .typeText('#project_name', `test_    project${randomDigits3}`)        .click('[value="Create"]')        .click('#top-menu .projects')        .click(`[href*="/projects/test_project${randomDigits3}"]`)        .click('.new-issue')        .typeText('#issue_subject', `Test issue ${randomDigits3}`)        .typeText('#issue_description', `Test issue description ${randomDigits3}`)        .click('#issue_priority_id')        .click(Selector('#issue_priority_id option').withText('High'))        .click('[value="Create"]')        .click('#top-menu .projects')        .click(`[href*="/projects/test_project${randomDigits3}"]`)        .click('#main-menu .issues')        .expect(Selector('.subject a').innerText).
contains(`Test issue ${randomDigits3}`);});
```

最后，没有登录操作的`上传文件`测试将看起来像这样：

```js
test('Upload a file', async (t) => {    await t.click('#top-menu .projects')        .click('.icon-add')        .typeText('#project_name', `test_project${randomDigits8}`)        .click('[value="Create"]')        .click('#top-menu .projects')        .click(`[href*="/projects/test_project${randomDigits8}"]`)        .click('.files')        .click('.icon-add')        .setFilesToUpload('input.file_selector', './uploads/test-file.txt')        .click('[value="Add"]')        .expect(Selector('.filename').innerText).eql('test-file.txt')        .expect(Selector('.digest').innerText).eql('d8e8fca2dc0f896fd7cb4cb0031ba249');});
```

现在，让我们将`beforeEach`块添加到`Redmine entities editing tests`测试用例中：

```js
fixture('Redmine entities editing tests')    .page('http://demo.redmine.org/')    .beforeEach(async (t) => {        await t.click('.login')            .typeText('#username', `test_user_testcafe_poc${randomDigits1}@sharklasers.com`)            .typeText('#password', 'test_user_testcafe_poc')            .click('[name="login"]');    });
```

让我们也从`Redmine entities editing tests`测试用例的所有测试中移除登录操作，因为这个操作现在将在`beforeEach`块中执行。因此，`编辑问题`测试将看起来像这样：

```js
test('Edit the issue', async (t) => {    await t.click('#top-menu .projects')        .click('.icon-add')        .typeText('#project_name', `test_project${randomDigits4}`)        .click('[value="Create"]')        .click('#top-menu .projects')        .click(`[href*="/projects/test_project${randomDigits4}"]`)        .click('.new-issue')        .typeText('#issue_subject', `Test issue ${randomDigits4}`)        .typeText('#issue_description', `Test issue description ${randomDigits4}`)        .click('#issue_priority_id')        .click(Selector('#issue_priority_id option').withText('High'))        .click('[value="Create"]')        .click('#top-menu .projects')        .click(`[href*="/projects/test_project${randomDigits4}"]`)        .click('#main-menu .issues')        .click(Selector('.subject a').withText(`Test issue ${randomDigits4}`))        .click('.icon-edit')        .selectText('#issue_subject')        .pressKey('delete')        .typeText('#issue_subject', `Issue ${randomDigits4} updated`)        .click('#issue_priority_id')        .click(Selector('#issue_priority_id option').withText('Normal'))        .click('[value="Submit"]')        .expect(Selector('#flash_notice').innerText).eql('Successful update.');});
```

所有登录操作都已移动到`beforeEach`块，因此`验证更新的问题是否显示在项目页面上`测试将看起来像这样：

```js
test('Verify that the updated issue is displayed on a project page', async (t) => {    await t.click('#top-menu .projects')        .click('.icon-add')        .typeText('#project_name', `test_project${randomDigits5}`)        .click('[value="Create"]')        .click('#top-menu .projects')        .click(`[href*="/projects/test_project${randomDigits5}"]`)        .click('.new-issue')        .typeText('#issue_subject', `Test issue ${randomDigits5}`)        .typeText('#issue_description', `Test issue description ${randomDigits5}`)        .click('#issue_priority_id')        .click(Selector('#issue_priority_id option').withText('High'))        .click('[value="Create"]')        .click('#top-menu .projects')        .click(`[href*="/projects/test_project${randomDigits5}"]`)        .click('#main-menu .issues')        .click(Selector('.subject a').withText(`Test issue ${randomDigits5}`))        .click('.icon-edit')        .selectText('#issue_subject')        .pressKey('delete')        .typeText('#issue_subject', `Issue ${randomDigits5} updated`)        .click('#issue_priority_id')        .click(Selector('#issue_priority_id option').         withText('Normal'))        .click('[value="Submit"]')        .click('#main-menu .issues')        .expect(Selector('.subject a').innerText).eql(`Issue ${randomDigits5} updated`);});
```

没有所有登录操作的`搜索问题`测试将看起来像这样：

```js
test('Search for the issue', async (t) => {    await t.click('#top-menu .projects')        .click('.icon-add')        .typeText('#project_name', `test_project${randomDigits6}`)        .click('[value="Create"]')        .click('#top-menu .projects')        .click(`[href*="/projects/test_project${randomDigits6}"]`)        .click('.new-issue')        .typeText('#issue_subject', `Test issue ${randomDigits6}`)        .typeText('#issue_description', `Test issue description ${randomDigits6}`)        .click('#issue_priority_id')        .click(Selector('#issue_priority_id option').         withText('High'))        .click('[value="Create"]')        .navigateTo('http://demo.redmine.org/search')        .typeText('#search-input', `Test issue ${randomDigits6}`)        .click('[value="Submit"]')        .expect(Selector('#search-results').innerText).contains(`Test issue ${randomDigits6}`);});
```

现在，让我们将`beforeEach`块添加到`Redmine entities deletion tests`测试用例中：

```js
fixture('Redmine entities deletion tests')    .page('http://demo.redmine.org/')    .beforeEach(async (t) => {        await t.click('.login')            .typeText('#username', `test_user_testcafe_poc${randomDigits1}@sharklasers.com`)            .typeText('#password', 'test_user_testcafe_poc')            .click('[name="login"]');});
```

让我们也从`Redmine entities deletion tests`测试用例的所有测试中移除登录操作，因为这些操作现在将在`beforeEach`块中执行。因此，`删除问题`测试将看起来像这样：

```js
test('Delete the issue', async (t) => {    await t.click('#top-menu .projects')        .click('.icon-add')        .typeText('#project_name', `test_  project${randomDigits7}`)        .click('[value="Create"]')        .click('#top-menu .projects')        .click(`[href*="/projects/test_project${randomDigits7}"]`)        .click('.new-issue')        .typeText('#issue_subject', `Test issue ${randomDigits7}`)        .typeText('#issue_description', `Test issue description ${randomDigits7}`)        .click('#issue_priority_id')        .click(Selector('#issue_priority_id option').withText('High'))        .click('[value="Create"]')        .click('#top-menu .projects')        .click(`[href*="/projects/test_project${randomDigits7}"]`)        .click('#main-menu .issues')        .click(Selector('.subject a').withText(`Test issue ${randomDigits7}`))        .setNativeDialogHandler(() => true)        .click('.icon-del')        .expect(Selector('.subject a').withText(`Test issue ${randomDigits7}`).exists).notOk()        .expect(Selector('.nodata').innerText).eql('No data to display');});
```

在所有登录操作都移动到`beforeEach`块之后，`删除文件`测试将看起来像这样：

```js
test('Delete the file', async (t) => {    await t.click('#top-menu .projects')        .click('.icon-add')        .typeText('#project_name', `test_  project${randomDigits9}`)        .click('[value="Create"]')        .click('#top-menu .projects')        .click(`[href*="/projects/test_project${randomDigits9}"]`)        .click('.files')        .click('.icon-add')        .setFilesToUpload('input.file_selector', './uploads/test-file.txt')        .click('[value="Add"]')        .click('#top-menu .projects')        .click(`[href*="/projects/test_
         project${randomDigits9}"]`)        .click('.files')        .setNativeDialogHandler(() => true)        .click(Selector('.filename a').withText('test-file.txt').parent('.file').find('.buttons a').withAttribute('data-method', 'delete'))        .expect(Selector('.filename').withText('test-file.txt').exists).notOk()        .expect(Selector('.digest').withText('d8e8fca2dc0f896fd7cb4cb0031ba249').exists).         notOk();});
```

注意

您还可以在 GitHub 上查看和下载此文件：[`github.com/PacktPublishing/Modern-Web-Testing-with-TestCafe/blob/master/ch5/test-project/tests/basic-tests17.js`](https://github.com/PacktPublishing/Modern-Web-Testing-with-TestCafe/blob/master/ch5/test-project/tests/basic-tests17.js)。

由于我们已经集成了设置和清理块，让我们看看如何使用命令行设置运行测试。

# 使用命令行设置运行测试

正如我们在*第三章*，“设置环境”中已经学到的，当你通过执行 `testcafe` 命令来触发测试时，TestCafe 会从 `.testcaferc.json` 配置文件中读取设置，如果该文件存在，然后在此基础上应用命令行设置。如果命令行设置与配置文件中的值不同，则命令行设置会覆盖配置文件中的值。TestCafe 会将每个覆盖属性的信息输出到控制台。

注意

如果在配置文件中提供了 `browsers` 和 `src` 属性，则可以在命令行中省略它们。

让我们回顾一下在启动测试时可以使用 `testcafe` 命令的一些主要命令行设置：

+   `--help` 或 `-h` 输出所有可用命令行选项的列表（[`devexpress.github.io/testcafe/documentation/reference/command-line-interface.html#-h---help`](https://devexpress.github.io/testcafe/documentation/reference/command-line-interface.html#-h---help)）。打开任何 shell 并运行以下命令：

    ```js
    $ npx testcafe --help
    ```

+   `--quarantine-mode` 或 `-q` 为失败的测试启用隔离模式（[`devexpress.github.io/testcafe/documentation/reference/command-line-interface.html#-q---quarantine-mode`](https://devexpress.github.io/testcafe/documentation/reference/command-line-interface.html#-q---quarantine-mode)）。打开任何 shell 并运行以下命令：

    ```js
    $ npx testcafe chrome tests/basic-tests.js --quarantine-mode
    ```

+   `--debug-mode` 或 `-d` 逐个执行测试步骤，在每个步骤后暂停测试以进行调试（[`devexpress.github.io/testcafe/documentation/reference/command-line-interface.html#-d---debug-mode`](https://devexpress.github.io/testcafe/documentation/reference/command-line-interface.html#-d---debug-mode)）。打开任何 shell 并运行以下命令：

    ```js
    $ npx testcafe chrome tests/basic-tests.js --debug-mode
    ```

+   `--debug-on-fail`：如果测试失败，则自动暂停并进入调试模式（[`devexpress.github.io/testcafe/documentation/reference/command-line-interface.html#--debug-on-fail`](https://devexpress.github.io/testcafe/documentation/reference/command-line-interface.html#--debug-on-fail)）。打开任何 shell 并运行以下命令：

    ```js
    $ npx testcafe chrome tests/basic-tests.js --debug-on-fail
    ```

+   `--disable-page-caching` 在测试执行期间禁用浏览器页面缓存（[`devexpress.github.io/testcafe/documentation/reference/command-line-interface.html#--disable-page-caching`](https://devexpress.github.io/testcafe/documentation/reference/command-line-interface.html#--disable-page-caching)）。打开任何 shell 并运行以下命令：

    ```js
    $ npx testcafe chrome tests/basic-tests.js --disable-page-caching
    ```

+   `--skip-js-errors`, 或 `-e`，确保在测试页面发生 JavaScript 错误时测试不会失败（[`devexpress.github.io/testcafe/documentation/reference/command-line-interface.html#-e---skip-js-errors`](https://devexpress.github.io/testcafe/documentation/reference/command-line-interface.html#-e---skip-js-errors)）。打开任何 shell 并运行以下命令：

    ```js
    $ npx testcafe chrome tests/basic-tests.js --skip-js-errors
    ```

+   `--skip-uncaught-errors`，或 `-u`，忽略测试执行期间发生的未捕获错误和未处理的承诺拒绝（[`devexpress.github.io/testcafe/documentation/reference/command-line-interface.html#-u---skip-uncaught-errors`](https://devexpress.github.io/testcafe/documentation/reference/command-line-interface.html#-u---skip-uncaught-errors)）。打开任何 shell 并运行以下命令：

    ```js
    $ npx testcafe chrome tests/basic-tests.js --skip-uncaught-errors
    ```

+   `--test <name>`，或 `-t <name>`，仅运行具有指定名称的测试用例（[`devexpress.github.io/testcafe/documentation/reference/command-line-interface.html#-t-name---test-name`](https://devexpress.github.io/testcafe/documentation/reference/command-line-interface.html#-t-name---test-name)）。打开任何 shell 并运行以下命令：

    ```js
    $ npx testcafe chrome tests/basic-tests.js --test "Click a link"
    ```

+   `--test-grep <pattern>`，或 `-T <pattern>`，仅运行与指定模式匹配的测试用例（[`devexpress.github.io/testcafe/documentation/reference/command-line-interface.html#-t-pattern---test-grep-pattern`](https://devexpress.github.io/testcafe/documentation/reference/command-line-interface.html#-t-pattern---test-grep-pattern)）。例如，要运行名为`Click a link`、`Click a dropdown`等测试用例，打开任何 shell 并运行以下命令：

    ```js
    $ npx testcafe chrome tests/basic-tests.js --test-grep "Click.*"
    ```

+   `--fixture <name>`，或 `-f <name>`，仅运行具有指定名称的固定测试用例（[`devexpress.github.io/testcafe/documentation/reference/command-line-interface.html#-f-name---fixture-name`](https://devexpress.github.io/testcafe/documentation/reference/command-line-interface.html#-f-name---fixture-name)）。打开任何 shell 并运行以下命令：

    ```js
    $ npx testcafe chrome tests/basic-tests.js --fixture "My first Fixture"
    ```

+   `--fixture-grep <pattern>`，或 `-F <pattern>`，仅运行与指定模式匹配的固定测试用例（[`devexpress.github.io/testcafe/documentation/reference/command-line-interface.html#-f-pattern---fixture-grep-pattern`](https://devexpress.github.io/testcafe/documentation/reference/command-line-interface.html#-f-pattern---fixture-grep-pattern)）。例如，要运行名为`Suite1`、`Suite2`等固定测试用例的测试，打开任何 shell 并运行以下命令：

    ```js
    $ npx testcafe chrome tests/basic-tests.js --fixture-grep "Suite.*"
    ```

+   `--test-meta <key=value[,key2=value2,...]>` 运行其元数据与指定键值对匹配的测试用例（[`devexpress.github.io/testcafe/documentation/reference/command-line-interface.html#--test-meta-keyvaluekey2value2`](https://devexpress.github.io/testcafe/documentation/reference/command-line-interface.html#--test-meta-keyvaluekey2value2)）。例如，要运行元数据的`suite`属性设置为`fast`且`env`属性设置为`staging`的测试用例，打开任何 shell 并运行以下命令：

    ```js
    $ npx testcafe chrome tests/basic-tests.js --test-meta suite=fast,env=staging
    ```

+   `--fixture-meta <key=value[,key2=value2,...]>` 从符合指定键值对的元数据的测试用例中运行测试 ([`devexpress.github.io/testcafe/documentation/reference/command-line-interface.html#--fixture-meta-keyvaluekey2value2`](https://devexpress.github.io/testcafe/documentation/reference/command-line-interface.html#--fixture-meta-keyvaluekey2value2)). 例如，要运行设置 `suite` 属性为 `long` 和 `env` 属性为 `production` 的元数据的测试用例，打开任何 shell 并运行以下命令：

    ```js
    $ npx testcafe chrome tests/basic-tests.js --fixture-meta suite=long,env=production
    ```

+   `--app <command>` 或 `-a <command>` 在测试开始之前执行指定的 shell 命令，通常用于在运行测试之前使用指定的命令启动测试应用 ([`devexpress.github.io/testcafe/documentation/reference/command-line-interface.html#-a-command---app-command`](https://devexpress.github.io/testcafe/documentation/reference/command-line-interface.html#-a-command---app-command))。打开任何 shell 并运行以下命令：

    ```js
    $ npx testcafe chrome tests/basic-tests.js --app "node index.js"
    ```

+   `--concurrency <number>` 或 `-c <number>` 通过启动提供的浏览器实例数量并行（同时）运行测试 ([`devexpress.github.io/testcafe/documentation/reference/command-line-interface.html#-c-n---concurrency-n`](https://devexpress.github.io/testcafe/documentation/reference/command-line-interface.html#-c-n---concurrency-n))。打开任何 shell 并运行以下命令：

    ```js
    $ npx testcafe chrome tests/basic-tests.js --concurrency 4
    ```

+   `--speed <factor>` 设置测试执行的速率，从最慢的 `0.01` 到最快的 `1` ([`devexpress.github.io/testcafe/documentation/reference/command-line-interface.html#--speed-factor`](https://devexpress.github.io/testcafe/documentation/reference/command-line-interface.html#--speed-factor))。打开任何 shell 并运行以下命令：

    ```js
    $ npx testcafe chrome tests/basic-tests.js --speed 0.8
    ```

你可以在 [`devexpress.github.io/testcafe/documentation/reference/command-line-interface.html`](https://devexpress.github.io/testcafe/documentation/reference/command-line-interface.html) 上阅读有关所有命令行选项的更多信息。

将所有主要设置保留在 `.testcaferc.json` 配置文件中是一种良好的实践，在需要时用命令行设置覆盖它们 - 例如，`--debug-on-fail --speed 0.8` 的组合将非常方便用于调试。

总结来说，在本节中，我们了解了一些主要的命令行设置以及它们在启动测试时的使用方法。

# 摘要

在本章中，我们探讨了如何选择性地执行测试，以及如何通过测试设置和清理来泛化一些测试操作。此外，我们还回顾了一些用于运行测试的命令行设置。现在我们有一个改进的测试套件，并知道如何使用命令行选项来运行它。

在下一章中，我们将通过将一些测试逻辑移动到单独的函数中，并使用 `PageObjects` 重构测试来继续改进我们的测试套件。
