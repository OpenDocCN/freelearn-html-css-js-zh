# *第七章:* TestCafe 的发现

本章的主要学习目标是使用函数优化我们的测试动作，并熟悉如何使用 npm 脚本来运行测试。我们还将回顾 TestCafe 框架发展的主要方向，以及一些有用资源的引用。

这将给我们一些额外的想法，关于如何重构测试，如何更有效地运行它们，以及在哪里寻找进一步的改进。

在本章中，我们将涵盖以下主要内容：

+   使用测试函数迈出最后一步。

+   使用 npm 脚本来封装测试项目。

+   探索 TestCafe 的开发和未来计划。

+   对有用资源的额外引用。

# 技术要求

本章的所有代码示例都可以在 GitHub 上找到：[`github.com/PacktPublishing/Modern-Web-Testing-with-TestCafe/blob/master/ch7`](https://github.com/PacktPublishing/Modern-Web-Testing-with-TestCafe/blob/master/ch7)。

# 使用测试函数迈出最后一步

我们创建的测试由一系列动作组成。其中一些，例如`Creating a new project`测试，仍在重复进行。所以，最后一步逻辑步骤将是将这些动作序列分离到函数中。让我们看看如何通过`redmine-page.js`中的`createNewProject`、`createNewIssue`和`uploadFile`函数来实现这一点：

```js
const { Selector, ClientFunction, Role, t } = require('testcafe');const { stamp } = require('js-automation-tools');// ...redminePage.getPageUrl = ClientFunction(() => {    return window.location.href;});redminePage.regularUser = Role(redminePage.urlRedmine, async (t) => {    await t.click(redminePage.linkLogin)        .typeText(redminePage.inputUsername, redminePage.emailRegularUser)        .typeText(redminePage.inputPassword, redminePage.         passwordRegularUser)        .click(redminePage.buttonLogin);});
```

如您所见，我们将`getPageUrl`和`regularUser`移动到了`redmine-page.js`，因为将所有实用函数集中在一个文件中非常方便。

现在，让我们添加`createNewProject`函数，该函数将包含创建新项目的所有动作：

```js
redminePage.createNewProject = async (textProjectName) => {    await t.click(redminePage.linkProjects)        .click(redminePage.iconAdd)        .typeText(redminePage.inputProjectName, textProjectName)        .click(redminePage.buttonCreate);};
```

需要添加另一个函数，该函数将包含创建新问题的所有动作：

```js
redminePage.createNewIssue = async (    linkTestProject, textIssueName, textIssueDescription     ) => {    await t.click(redminePage.linkProjects)        .click(linkTestProject)        .click(redminePage.linkNewIssue)        .typeText(redminePage.inputIssueSubject, textIssueName)        .typeText(redminePage.inputIssueDescription, textIssueDescription)        .click(redminePage.dropdownIssuePriority)        .click(Selector(redminePage.optionIssuePriority).withText(redminePage.textHigh))        .click(redminePage.buttonCreate);};
```

最后，添加包含上传文件所有动作的函数：

```js
redminePage.uploadFile = async (linkTestProject) => {    await t.click(redminePage.linkProjects)        .click(linkTestProject)        .click(redminePage.linkFiles)        .click(redminePage.iconAdd)        .setFilesToUpload(redminePage.inputChooseFiles, redminePage.pathToFile)        .click(redminePage.buttonAdd);};module.exports = redminePage;
```

注意

您也可以在 GitHub 上查看和下载此文件：[`github.com/PacktPublishing/Modern-Web-Testing-with-TestCafe/blob/master/ch7/test-project/tests/redmine-page4.js`](https://github.com/PacktPublishing/Modern-Web-Testing-with-TestCafe/blob/master/ch7/test-project/tests/redmine-page4.js)。

现在，更新的`Create a new project`测试将看起来像这样：

```js
const { Selector } = require('testcafe');const redminePage = require('./redmine-page.js');// ...fixture('Redmine entities creation tests')    .page(redminePage.urlRedmine)    .beforeEach(async (t) => {        await t.useRole(redminePage.regularUser);    });test('Create a new project', async (t) => {    await redminePage.createNewProject(redminePage.text1ProjectName);    await t.expect(Selector(redminePage.blockNotification).innerText).eql(redminePage.textSuccessfulCreation)        .expect(redminePage.getPageUrl()).contains(redminePage.urlProjectSettings);});
```

如您可能已注意到，现在，从`testcafe`中只需要`Selector`，因为`ClientFunction`、`Role`和`t`已被移动到`redmine-page.js`。此外，我们现在使用`redminePage.regularUser`而不是仅仅`regularUser` - 这是因为将`regularUser`函数移动到了`redmine-page.js`。

更新的`Create a new issue`测试将看起来像这样：

```js
test('Create a new issue', async (t) => {    await redminePage.createNewProject(redminePage.text2ProjectName);    await redminePage.createNewIssue(        redminePage.link2TestProject,        redminePage.text2IssueName,        redminePage.text2IssueDescription     );    await t.expect(Selector(redminePage.blockNotification).innerText).contains(redminePage.textCreated);});
```

现在的`Verify that the issue is displayed on a project page`测试看起来也会更短，因为我们现在在测试中使用`createNewProject`和`createNewIssue`函数来创建相应的实体：

```js
test('Verify that the issue is displayed on a project page', async (t) => {    await redminePage.createNewProject(redminePage.text3ProjectName);    await redminePage.createNewIssue(        redminePage.link3TestProject,        redminePage.text3IssueName,        redminePage.text3IssueDescription     );    await t.click(redminePage.linkProjects)        .click(redminePage.link3TestProject)        .click(redminePage.linkIssues)        .expect(Selector(redminePage.linkIssueName).innerText).contains(redminePage.text3IssueName);});
```

在`Upload a file`测试中，我们将利用`createNewProject`和`uploadFile`函数，使其看起来也更紧凑：

```js
test('Upload a file', async (t) => {    await redminePage.createNewProject(redminePage.text4ProjectName);    await redminePage.uploadFile(redminePage.link4TestProject);    await t.expect(Selector(redminePage.linkFileName).innerText).eql(redminePage.textFileName)        .expect(Selector(redminePage.blockDigest).innerText).eql(redminePage.textChecksum);});
```

下面是使用 `createNewProject` 和 `createNewIssue` 函数更新的 `Edit the issue` 测试的显示方式：

```js
// ...test('Edit the issue', async (t) => {    await redminePage.createNewProject(redminePage.text5ProjectName);    await redminePage.createNewIssue(        redminePage.link5TestProject,        redminePage.text5IssueName,        redminePage.text5IssueDescription     );    await t.click(redminePage.linkProjects)        .click(redminePage.link5TestProject)        .click(redminePage.linkIssues)        .click(Selector(redminePage.linkIssueName).withText(redminePage.text5IssueName))        .click(redminePage.iconEdit)        .selectText(redminePage.inputIssueSubject)        .pressKey(redminePage.keyDelete)        .typeText(redminePage.inputIssueSubject, redminePage.text5IssueNameUpdated)        .click(redminePage.dropdownIssuePriority)        .click(Selector(redminePage.optionIssuePriority).withText(redminePage.textNormal))        .click(redminePage.buttonSubmit)        .expect(Selector(redminePage.blockNotification).innerText).eql(redminePage.textSuccessfulUpdate);});
```

并且使用 `createNewProject` 和 `createNewIssue` 函数重构的 `Verify that the updated issue is displayed on a project page` 测试现在看起来是这样的：

```js
test('Verify that the updated issue is displayed on a project page', async (t) => {    await redminePage.createNewProject(redminePage.text6ProjectName);    await redminePage.createNewIssue(        redminePage.link6TestProject,        redminePage.text6IssueName,        redminePage.text6IssueDescription     );    await t.click(redminePage.linkProjects)        .click(redminePage.link6TestProject)        .click(redminePage.linkIssues)        .click(Selector(redminePage.linkIssueName).  withText(redminePage.text6IssueName))        .click(redminePage.iconEdit)        .selectText(redminePage.inputIssueSubject)        .pressKey(redminePage.keyDelete)        .typeText(redminePage.inputIssueSubject, redminePage.text6IssueNameUpdated)        .click(redminePage.dropdownIssuePriority)        .click(Selector(redminePage.optionIssuePriority).withText(redminePage.textNormal))        .click(redminePage.buttonSubmit)        .click(redminePage.linkIssues)        .expect(Selector(redminePage.linkIssueName).innerText).eql(redminePage.text6IssueNameUpdated);});
```

`Search for the issue` 测试也将从利用 `createNewProject` 和 `createNewIssue` 函数中受益，因为它将显著缩短：

```js
test('Search for the issue', async (t) => {    await redminePage.createNewProject(redminePage.text7ProjectName);    await redminePage.createNewIssue(        redminePage.link7TestProject,        redminePage.text7IssueName,        redminePage.text7IssueDescription     );    await t.navigateTo(redminePage.urlRedmineSearch)        .typeText(redminePage.inputSearch, redminePage.text7IssueName)        .click(redminePage.buttonSubmit)        .expect(Selector(redminePage.blockSearchResults).innerText).contains(redminePage.text7IssueName);});
```

最后，以下是重构的 `Delete the issue` 和 `Delete the file` 测试的显示方式：

```js
// ...test('Delete the issue', async (t) => {    await redminePage.createNewProject(redminePage.text8ProjectName);    await redminePage.createNewIssue(        redminePage.link8TestProject,        redminePage.text8IssueName,        redminePage.text8IssueDescription     );    await t.click(redminePage.linkProjects)        .click(redminePage.link8TestProject)        .click(redminePage.linkIssues)        .click(Selector(redminePage.linkIssueName).withText(redminePage.text8IssueName))        .setNativeDialogHandler(() => true)        .click(redminePage.iconDelete)        .expect(Selector(redminePage.linkIssueName).withText(redminePage.text8IssueName).exists).notOk()        .expect(Selector(redminePage.blockNoData).innerText).eql(redminePage.textNoData);});test('Delete the file', async (t) => {    await redminePage.createNewProject(redminePage.text9ProjectName);    await redminePage.uploadFile(redminePage.link9TestProject);    await t.click(redminePage.linkProjects)        .click(redminePage.link9TestProject)        .click(redminePage.linkFiles)        .setNativeDialogHandler(() => true)        .click(Selector(redminePage.linkFileName).withText(redminePage.textFileName).parent(redminePage.blockFile).find(redminePage.buttonAction).withAttribute('data-method', redminePage.dataMethodDelete))        .expect(Selector(redminePage.linkFileName).withText(redminePage.textFileName).exists).notOk()        .expect(Selector(redminePage.blockDigest).withText(redminePage.textChecksum).exists).notOk();});
```

注意

您也可以在 GitHub 上查看并下载此文件：[`github.com/PacktPublishing/Modern-Web-Testing-with-TestCafe/blob/master/ch7/test-project/tests/basic-tests22.js`](https://github.com/PacktPublishing/Modern-Web-Testing-with-TestCafe/blob/master/ch7/test-project/tests/basic-tests22.js)。

因此，我们已经优化了我们的测试集，使其更细粒度并使用函数而不是重复操作。现在，让我们使用 npm 脚本来完成测试项目。

使用 npm 脚本完成测试项目

由于我们已经完成了测试的重构，让我们看看如何更有效地运行它们。如我们回忆在 *第三章**，设置环境*，我们初始化了 `package.json`，以及 *第四章**，使用 TestCafe 构建测试套件*，我们添加了 `js-automation-tools` 库，我们的基本 `package.json` 文件目前看起来是这样的：

```js
{  "name": "test-project",  "version": "1.0.0",  "description": "",  "main": "index.js",  "scripts": {    "test": "echo \"Error: no test specified\" && exit 1"  },  "keywords": [],  "author": "",  "license": "ISC",  "devDependencies": {    "js-automation-tools": "¹.0.5",    "testcafe": "¹.8.7"  }}
```

我们目前通过执行以下命令来运行我们的测试：

```js
$ npx testcafe chrome tests/basic-tests.js
```

或者，如我们讨论的 *第五章*，*改进测试*，我们利用双横线调试失败标志来使我们的开发生活更轻松（这将在测试失败时暂停测试，并允许您查看测试页面并确定失败的原因）：

```js
$ npx testcafe chrome tests/basic-tests.js --debug-on-fail
```

我们还可以使用一个额外的标志：`--speed`（设置测试执行速率）——以降低测试执行速度进行调试：

```js
$ npx testcafe chrome tests/basic-tests.js --debug-on-fail --speed 0.8
```

现在看起来相当长了，不是吗？为了克服这个问题，我们可以使用 npm 脚本。让我们在 `package.json` 中创建一个 `test-debug` 别名来带有调试标志启动测试：

```js
{  "name": "test-project",  "version": "1.0.0",  "description": "",  "main": "index.js",  "scripts": {    "test-debug": "testcafe chrome tests/basic-tests.js --debug-on-fail --speed 0.8"  },  "keywords": [],  "author": "",  "license": "ISC",  "devDependencies": {    "js-automation-tools": "¹.0.5",    "testcafe": "¹.8.7"  }}
```

因此，现在我们可以通过执行我们刚刚创建的别名的一个简短命令来带有调试标志运行我们的测试：

```js
$ npm run test-debug
```

正如我们讨论了如何使用 npm 脚本来添加本地测试调试的命令，现在让我们想象我们还需要在 `--quarantine-mode` 标志下运行我们的测试：

```js
{  "name": "test-project",  "version": "1.0.0",  "description": "",  "main": "index.js",  "scripts": {    "test-ci": "testcafe chrome tests/basic-tests.js --quarantine-mode",    "test-debug": "testcafe chrome tests/basic-tests.js --debug-on-fail --speed 0.8"  },  "keywords": [],  "author": "",  "license": "ISC",  "devDependencies": {    "js-automation-tools": "¹.0.5",    "testcafe": "¹.8.7"  }}
```

注意

您也可以在 GitHub 上查看并下载此文件：[`github.com/PacktPublishing/Modern-Web-Testing-with-TestCafe/blob/master/ch7/test-project/package.json`](https://github.com/PacktPublishing/Modern-Web-Testing-with-TestCafe/blob/master/ch7/test-project/package.json)。

因此，现在我们可以通过执行一个简短且简单的命令在 CI 上运行我们的测试：

```js
$ npm run test-ci
```

总结来说，在本节中，我们讨论了如何使用 npm 脚本来创建本地和远程（持续集成）命令的简短别名。

现在，让我们探讨如何保持对 TestCafe 开发的关注，以及在哪里寻找进一步的改进。

# 探索 TestCafe 的开发和未来计划

TestCafe 的诞生可以追溯到 2010 年初，当时来自 DevExpress 的开发者开始着手开发它。最初，当它在 2013 年发布时，它是一个商业测试框架。到了 2016 年，决定开源 TestCafe 的核心。从那时起，每月下载量已超过 76 万次，并且这个数字仍在增长。DevExpress 还发布了一个名为 TestCafe Studio 的商业测试 IDE（[`www.devexpress.com/products/testcafestudio/`](https://www.devexpress.com/products/testcafestudio/))，它是基于开源的 TestCafe 核心构建的。因此，看起来 TestCafe 将会持续存在。DevExpress 将继续开发它，因为这将为 TestCafe Studio 添加新功能。

让我们回顾一下 TestCafe 的一些优点：

+   开源。

+   安装简单快捷。

+   无头测试。

+   开箱即用的跨平台和跨浏览器支持。

+   支持最受欢迎的 Web 开发编程语言之一：JavaScript/TypeScript。

+   清晰、灵活且文档齐全的 API。

+   开箱即用的智能断言和自动等待机制。

+   为浏览器提供商、框架特定选择器、自定义报告器、Cucumber 支持等提供免费自定义插件。

关于 TestCafe 未来开发方向，根据路线图（[`devexpress.github.io/testcafe/roadmap/`](https://devexpress.github.io/testcafe/roadmap/))，有一个计划通过添加发送 HTTP 请求和检查响应细节的方法来支持 API 测试。除此之外，TestCafe 团队正在积极开发多浏览器窗口功能，并计划进一步改进 TestCafe 的调试流程。

另一条值得提到的建议：关注 TestCafe 的变更日志（[`github.com/DevExpress/testcafe/blob/master/CHANGELOG.md`](https://github.com/DevExpress/testcafe/blob/master/CHANGELOG.md)）。它包含了大量关于新功能和更新的有用信息。这样，你将始终知道何时发布新版本，以及可以期待什么。

在我们回顾了 TestCafe 的开发并触及了其未来计划之后，现在让我们探索一些可用于进一步使用 TestCafe 进行测试自动化的资源。

# 其他有用的资源参考

这里有一些关于 TestCafe 的优秀信息来源：

+   **TestCafe 文档**：[`devexpress.github.io/testcafe/documentation/reference/`](https://devexpress.github.io/testcafe/documentation/reference/)。

+   **TestCafe 变更日志**：[`github.com/DevExpress/testcafe/blob/master/CHANGELOG.md`](https://github.com/DevExpress/testcafe/blob/master/CHANGELOG.md)。

+   **TestCafe 未来路线图**：[`devexpress.github.io/testcafe/roadmap/`](https://devexpress.github.io/testcafe/roadmap/) 和 [`github.com/DevExpress/testcafe/projects`](https://github.com/DevExpress/testcafe/projects).

+   **TestCafe 团队博客**：[`devexpress.github.io/testcafe/media/team-blog/`](https://devexpress.github.io/testcafe/media/team-blog/).

+   **Stack Overflow 过滤器，用于最近关于 TestCafe 的问题**：[`stackoverflow.com/questions/tagged/testcafe`](https://stackoverflow.com/questions/tagged/testcafe).

# 摘要

在本章中，我们探讨了如何使用函数优化测试操作，以及如何使用 npm 脚本来运行测试。我们还回顾了 TestCafe 框架的发展，以及一些有用的资源参考。这些技能和经验旨在通过强调如何重构测试、如何更有效地运行它们以及如何寻找进一步的改进，来帮助你进行任何进一步的测试自动化开发。

这标志着我们对 TestCafe，测试自动化领域的新星，富有成效的探索的结束。我希望你喜欢这次探索，并在未来的项目中继续使用这个出色的工具！
