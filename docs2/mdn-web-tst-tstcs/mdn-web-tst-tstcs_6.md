# *第六章：使用 PageObjects 重构*

这里的主要学习目标将是熟悉如何使用 TestCafe 角色和 `PageObject` 模式升级一组端到端测试（测试套件）。我们将使用 `Role` 来加速测试，并利用 `PageObjects` 来减少代码重复并提高可维护性。到本章结束时，我们将拥有一组结构化和优化的测试，并了解如何将角色和 `PageObject` 模式应用于任何未来的项目。

在本章中，我们将涵盖以下主要主题：

+   添加登录 `Role`。

+   使用 `PageObjects` 重构测试。

+   使用函数改进 `PageObjects`。

# 技术要求

本章的所有代码示例都可以在 GitHub 上找到：[`github.com/PacktPublishing/Modern-Web-Testing-with-TestCafe/tree/master/ch6`](https://github.com/PacktPublishing/Modern-Web-Testing-with-TestCafe/tree/master/ch6)。

# 添加登录 `Role`

如我们从 *第二章* 中所学到的，*探索 TestCafe 的内部机制*，TestCafe 具有内置的用户角色机制，该机制模拟用户动作以登录网站。角色机制将每个用户的登录状态保存在一个单独的角色中，然后可以在测试的任何部分重复使用该角色以在用户账户之间切换。

因此，让我们首先添加一个 `Role` 来优化和加速我们在每个测试中执行的登录过程：

```js
const { Selector, ClientFunction, Role } = require('testcafe');// ...const regularUser = Role('http://demo.redmine.org/', async (t) => {    await t.click('.login')        .typeText('#username', `test_user_testcafe_ poc${randomDigits1}@sharklasers.com`)        .typeText('#password', 'test_user_testcafe_poc')        .click('[name="login"]');});
```

如你所见，我们正在使用 `Role` 内部的登录步骤。这些步骤将在 `regularUser` 角色首次被调用时执行。一旦登录步骤执行完毕，认证 cookie 和浏览器存储的最终（登录）状态将被保存并用于 `regularUser` 角色的后续调用（登录步骤将不再执行，因为已保存的登录状态将被应用）。现在，让我们在 `Log out` 测试中使用 `regularUser` 角色吧：

```js
test('Log out', async (t) => {    await t.useRole(regularUser)        .click('.logout')        .expect(Selector('#loggedas').exists).notOk()        .expect(Selector('.login').exists).ok();});
```

让我们还在所有其他固定装置的 `beforeEach` 块中将登录步骤替换为 `regularUser` 角色吧：

```js
// ...fixture('Redmine entities creation tests')    .page('http://demo.redmine.org/')    .beforeEach(async (t) => {        await t.useRole(regularUser);    });// ...fixture('Redmine entities editing tests')    .page('http://demo.redmine.org/')    .beforeEach(async (t) => {        await t.useRole(regularUser);    });// ...fixture('Redmine entities deletion tests')    .page('http://demo.redmine.org/')    .beforeEach(async (t) => {        await t.useRole(regularUser);    });// ...
```

注意

你也可以在 GitHub 上查看和下载此文件：[`github.com/PacktPublishing/Modern-Web-Testing-with-TestCafe/blob/master/ch6/test-project/tests/basic-tests18.js`](https://github.com/PacktPublishing/Modern-Web-Testing-with-TestCafe/blob/master/ch6/test-project/tests/basic-tests18.js)。

利用角色可以加快测试的执行速度，并且我们的测试项目变得更加结构化和细化，这是一个很好的做法。

在我们已经构建了一组测试，通过设置、拆解和角色进行了改进之后，现在让我们通过使用 `PageObjects` 进行重构，使它们更加有效和易于维护。

使用 PageObjects 重构测试

`PageObject` 是一种测试自动化模式，它允许你创建一个单独的文件，其中包含选择器和函数，这些选择器和函数代表了被测试页面的抽象。这个单独的文件可以随后被包含并用于测试代码中，以便引用页面元素。

注意，我们的测试中包含过多的代码。例如，CSS 选择器`#top-menu .projects`（https://github.com/PacktPublishing/Modern-Web-Testing-with-TestCafe/blob/master/ch6/test-project/tests/basic-tests18.js#L62）在 22 行代码中使用 - 也就是说，每次测试点击`PageObjects`时，你都可以将所有选择器放在一个地方，这样下次网页发生变化时，你只需修改`PageObject`文件即可。

因此，让我们在`tests`文件夹内创建一个简单的`redmine-page.js`文件。打开任何 shell，转到`test-project`文件夹，并执行以下命令：

```js
$ touch tests/redmine-page.js
```

现在，在您选择的代码编辑器中打开`redmine-page.js`文件，在`redminePage`对象内添加声明`linkProjects`选择器的代码，然后导出此对象：

```js
let redminePage = {    linkProjects: '#top-menu .projects'};module.exports = redminePage;
```

注意

您也可以在 GitHub 上查看和下载此文件：[`github.com/PacktPublishing/Modern-Web-Testing-with-TestCafe/blob/master/ch6/test-project/tests/redmine-page1.js`](https://github.com/PacktPublishing/Modern-Web-Testing-with-TestCafe/blob/master/ch6/test-project/tests/redmine-page1.js)。

因此，我们创建了一个包含`linkProjects`属性的对象，该属性包含此元素的选择器。现在我们需要将此对象包含在我们的测试中：

```js
const { Selector, ClientFunction, Role } = require('testcafe');const { stamp } = require('js-automation-tools');const redminePage = require('./redmine-page.js');// ...
```

此外，我们还需要将所有`#top-menu .projects`的出现替换为相应的`PageObject`元素。因此，更新后的`创建新项目`测试将如下所示：

```js
test('Create a new project', async (t) => {    await t.click(redminePage.linkProjects)        .click('.icon-add')        .typeText('#project_name', `test_project${randomDigits1}`)        .click('[value="Create"]')        .expect(Selector('#flash_notice').innerText).eql('Successful creation.')        .expect(getPageUrl()).contains(`/projects/test_project${randomDigits1}/settings`);});
```

注意

您也可以在 GitHub 上查看和下载此文件，其中所有测试都已更新以使用`redminePage.linkProjects`：[`github.com/PacktPublishing/Modern-Web-Testing-with-TestCafe/blob/master/ch6/test-project/tests/basic-tests19.js`](https://github.com/PacktPublishing/Modern-Web-Testing-with-TestCafe/blob/master/ch6/test-project/tests/basic-tests19.js)。

现在，将所有带有随机数字生成的常量移动到`redmine-page.js`中，因为所有使用随机数字的字符串也将移动到`redmine-page.js`中：

```js
const { stamp } = require('js-automation-tools');const randomDigits1 = stamp.getTimestamp();const randomDigits2 = stamp.resetTimestamp();const randomDigits3 = stamp.resetTimestamp();const randomDigits4 = stamp.resetTimestamp();const randomDigits5 = stamp.resetTimestamp();const randomDigits6 = stamp.resetTimestamp();const randomDigits7 = stamp.resetTimestamp();const randomDigits8 = stamp.resetTimestamp();const randomDigits9 = stamp.resetTimestamp();
```

现在，让我们将所有选择器移动到`redmine-page.js`文件中的`redminePage`对象内。我们将从登录凭据开始：

```js
let redminePage = {    urlRedmine: 'http://demo.redmine.org/',    emailRegularUser: `test_user_testcafe_poc${randomDigits1}@sharklasers.com`,    passwordRegularUser: 'test_user_testcafe_poc',
```

现在，让我们添加页面元素的选择器：

```js
    linkLogin: '.login',    inputUsername: '#username',    inputPassword: '#password',    buttonLogin: '[name="login"]',    linkRegister: '.register',    inputUserLogin: '#user_login',    inputUserPassword: '#user_password',    inputUserPasswordConfirmation: '#user_password_confirmation',    inputUserFirstName: '#user_firstname',    inputUserLastName: '#user_lastname',    inputUserMail: '#user_mail',    buttonSubmit: '[value="Submit"]',    blockNotification: '#flash_notice',    blockLoggedAs: '#loggedas',    linkLogout: '.logout',    linkProjects: '#top-menu .projects',    iconAdd: '.icon-add',    inputProjectName: '#project_name',    buttonCreate: '[value="Create"]',    urlProjectSettings: `/projects/test_project${randomDigits1}/settings`,    link2TestProject: `[href*="/projects/test_project${randomDigits2}"]`,// ...
```

最后，让我们增强`redminePage`对象，添加包含文本的属性：

```js
    textFirstNameRegularUser: 'test_user',    textLastNameRegularUser: 'testcafe_poc',    textAccountActivated: 'Your account has been activated. You can now log in.',    text1ProjectName: `test_project${randomDigits1}`,    text2ProjectName: `test_project${randomDigits2}`,// ...};module.exports = redminePage;
```

注意

您也可以在 GitHub 上查看和下载此文件：[`github.com/PacktPublishing/Modern-Web-Testing-with-TestCafe/blob/master/ch6/test-project/tests/redmine-page2.js`](https://github.com/PacktPublishing/Modern-Web-Testing-with-TestCafe/blob/master/ch6/test-project/tests/redmine-page2.js)。

如您所见，URL 如`http://demo.redmine.org/`和通知文本字符串如`'您的账户已激活。您现在可以登录。'`也可以通过`PageObjects`轻松重构。

现在，让我们在每个测试中使用`PageObject`元素。因此，更新的`Redmine 登录测试`固定装置将如下所示：

```js
const { Selector, ClientFunction, Role } = require('testcafe');const redminePage = require('./redmine-page.js');const getPageUrl = ClientFunction(() => {    return window.location.href;});const regularUser = Role(redminePage.urlRedmine, async (t) => {    await t.click(redminePage.linkLogin)        .typeText(redminePage.inputUsername, redminePage.emailRegularUser)        .typeText(redminePage.inputPassword, redminePage.passwordRegularUser)        .click(redminePage.buttonLogin);});fixture('Redmine log in tests').page(redminePage.urlRedmine);
```

带有`PageObject`元素的`创建新用户`测试将如下所示：

```js
test('Create a new user', async (t) => {    await t.click(redminePage.linkRegister)        .typeText(redminePage.inputUserLogin, redminePage.emailRegularUser)        .typeText(redminePage.inputUserPassword, redminePage.passwordRegularUser)        .typeText(redminePage.inputUserPasswordConfirmation, redminePage.passwordRegularUser)        .typeText(redminePage.inputUserFirstName, redminePage.textFirstNameRegularUser)        .typeText(redminePage.inputUserLastName, redminePage.textLastNameRegularUser)        .typeText(redminePage.inputUserMail, redminePage.emailRegularUser)        .click(redminePage.buttonSubmit)        .expect(Selector(redminePage.blockNotification).innerText).eql(redminePage.textAccountActivated);});
```

`登录`和`登出`测试将如下所示：

```js
test('Log in', async (t) => {    await t.click(redminePage.linkLogin)        .typeText(redminePage.inputUsername, redminePage.emailRegularUser)        .typeText(redminePage.inputPassword, redminePage.passwordRegularUser)        .click(redminePage.buttonLogin)        .expect(Selector(redminePage.blockLoggedAs).exists).ok();});test('Log out', async (t) => {    await t.useRole(regularUser)        .click(redminePage.linkLogout)        .expect(Selector(redminePage.blockLoggedAs).exists).notOk()        .expect(Selector(redminePage.linkLogin).exists).ok();});
```

更新后的`Redmine 实体创建测试`固定装置将看起来如下：

```js
fixture('Redmine entities creation tests')    .page(redminePage.urlRedmine)    .beforeEach(async (t) => {        await t.useRole(regularUser);    });
```

`创建新项目`和`创建新问题`测试将如下所示：

```js
test('Create a new project', async (t) => {    await t.click(redminePage.linkProjects)        .click(redminePage.iconAdd)        .typeText(redminePage.inputProjectName, redminePage.text1ProjectName)        .click(redminePage.buttonCreate)        .expect(Selector(redminePage.blockNotification).innerText).eql(redminePage.textSuccessfulCreation)        .expect(getPageUrl()).contains(redminePage.         urlProjectSettings);});test('Create a new issue', async (t) => {    await t.click(redminePage.linkProjects)        .click(redminePage.iconAdd)        .typeText(redminePage.inputProjectName, redminePage.text2ProjectName)        .click(redminePage.buttonCreate)        .click(redminePage.linkProjects)        .click(redminePage.link2TestProject)        .click(redminePage.linkNewIssue)        .typeText(redminePage.inputIssueSubject, redminePage.text2IssueName)        .typeText(redminePage.inputIssueDescription, redminePage.text2IssueDescription)        .click(redminePage.dropdownIssuePriority)        .click(Selector(redminePage.optionIssuePriority).withText(redminePage.textHigh))        .click(redminePage.buttonCreate)        .expect(Selector(redminePage.blockNotification).innerText).contains(redminePage.textCreated);});
```

注意

您还可以在 GitHub 上审查和下载所有测试更新为使用`PageObjects`的文件：[`github.com/PacktPublishing/Modern-Web-Testing-with-TestCafe/blob/master/ch6/test-project/tests/basic-tests20.js`](https://github.com/PacktPublishing/Modern-Web-Testing-with-TestCafe/blob/master/ch6/test-project/tests/basic-tests20.js)。

在本节中，我们学习了如何使用`PageObjects`增强测试，并相应地重构了我们的测试项目。现在，让我们通过向`PageObject`添加函数来进一步提高测试的可维护性。

使用函数改进 PageObjects

如我们在`redmine-page.js`中观察到的，`PageObject`内部的一些属性仍然包含一些重复的代码。让我们通过将此类重复代码移动到单独的函数中来进一步优化我们的`PageObject`：

```js
// ...const createButtonSelector = (text) => {    return `[value="${text}"]`;};const createLinkTestProjectSelector = (randomDigits) => {    return `[href*="/projects/test_project${randomDigits}"]`;};const createProjectNameText = (randomDigits) => {    return `test_project${randomDigits}`;};const createIssueNameText = (randomDigits) => {    return `Test issue ${randomDigits}`;};const createIssueDescriptionText = (randomDigits) => {    return `Test issue description ${randomDigits}`;};const createIssueNameUpdatedText = (randomDigits) => {    return `Issue ${randomDigits} updated`;};redminePage.buttonLogin = createButtonSelector('Login »');redminePage.buttonSubmit = createButtonSelector('Submit');redminePage.buttonCreate = createButtonSelector('Create');redminePage.buttonAdd = createButtonSelector('Add');
```

注意，`buttonLogin`选择器已更新。之前它是`[name="login"]`，但现在`createButtonSelector`函数将返回它为`[value="Login »"]`。这样做是为了通用我们的选择器生成，因此现在`buttonLogin`选择器是用与所有其他按钮元素相同的`createButtonSelector`函数创建的。

让我们现在生成一组测试项目链接的选择器：

```js
redminePage.link2TestProject = createLinkTestProjectSelector(randomDigits2);redminePage.link3TestProject = createLinkTestProjectSelector(randomDigits3);redminePage.link4TestProject = createLinkTestProjectSelector(randomDigits8);redminePage.link5TestProject = createLinkTestProjectSelector(randomDigits4);redminePage.link6TestProject = createLinkTestProjectSelector(randomDigits5);redminePage.link7TestProject = createLinkTestProjectSelector(randomDigits6);redminePage.link8TestProject = createLinkTestProjectSelector(randomDigits7);redminePage.link9TestProject = createLinkTestProjectSelector(randomDigits9);
```

现在，让我们生成一组测试项目名称的文本：

```js
redminePage.text1ProjectName = createProjectNameText(randomDigits1);redminePage.text2ProjectName = createProjectNameText(randomDigits2);redminePage.text3ProjectName = createProjectNameText(randomDigits3);redminePage.text4ProjectName = createProjectNameText(randomDigits8);redminePage.text5ProjectName = createProjectNameText(randomDigits4);redminePage.text6ProjectName = createProjectNameText(randomDigits5);redminePage.text7ProjectName = createProjectNameText(randomDigits6);redminePage.text8ProjectName = createProjectNameText(randomDigits7);redminePage.text9ProjectName = createProjectNameText(randomDigits9);
```

最后，让我们生成一组包含问题名称（例如，`Test issue 1598717241841`）和问题描述（例如，`Test issue description 1598717241841`）文本的对象属性：

```js
redminePage.text2IssueName = createIssueNameText(randomDigits2);redminePage.text3IssueName = createIssueNameText(randomDigits3);redminePage.text5IssueName = createIssueNameText(randomDigits4);redminePage.text6IssueName = createIssueNameText(randomDigits5);redminePage.text7IssueName = createIssueNameText(randomDigits6);redminePage.text8IssueName = createIssueNameText(randomDigits7);redminePage.text2IssueDescription = createIssueDescriptionText(randomDigits2);redminePage.text3IssueDescription = createIssueDescriptionText(randomDigits3);redminePage.text5IssueDescription = createIssueDescriptionText(randomDigits4);redminePage.text6IssueDescription = createIssueDescriptionText(randomDigits5);redminePage.text7IssueDescription = createIssueDescriptionText(randomDigits6);redminePage.text8IssueDescription = createIssueDescriptionText(randomDigits7);redminePage.text5IssueNameUpdated = createIssueNameUpdatedText(randomDigits4);redminePage.text6IssueNameUpdated = createIssueNameUpdatedText(randomDigits5);module.exports = redminePage;
```

注意

您还可以使用`PageObject`函数审查和下载此文件：[`github.com/PacktPublishing/Modern-Web-Testing-with-TestCafe/blob/master/ch6/test-project/tests/redmine-page3.js`](https://github.com/PacktPublishing/Modern-Web-Testing-with-TestCafe/blob/master/ch6/test-project/tests/redmine-page3.js)，以及相应的测试文件：[`github.com/PacktPublishing/Modern-Web-Testing-with-TestCafe/blob/master/ch6/test-project/tests/basic-tests21.js`](https://github.com/PacktPublishing/Modern-Web-Testing-with-TestCafe/blob/master/ch6/test-project/tests/basic-tests21.js)。

因此，现在我们有函数来创建类似选择器和文本的组。这种技术最终非常有用，因为可以通过更改创建它们的相应函数来在一个地方编辑一组类似元素。

摘要

在本章中，我们探讨了如何在登录时使用`Role`来加速测试执行，使用`PageObject`重构了测试，并通过函数改进了`PageObject`。现在我们有一组快速且易于维护和扩展的测试（如果将来有必要的话）。这些知识可以用来重构现有测试，或者构建一组新的健壮且易于维护的自动化测试集。

在下一章中，我们将总结测试项目，并快速浏览 TestCafe 的未来。
