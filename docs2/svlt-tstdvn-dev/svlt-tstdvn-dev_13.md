

# 第十三章：添加 Cucumber 测试

到目前为止，你已经看到了两种类型的自动化测试：Vitest 单元测试和 Playwright 端到端测试。本章添加了第三种测试类型：**Cucumber**（[`cucumber.io`](https://cucumber.io)）。

就像 Playwright 一样，Cucumber 也有自己的测试运行器，通常设置为以与 Playwright 相同的方式驱动应用程序。区别在于 Cucumber 测试不是用 JavaScript 代码编写的。

Cucumber 测试包含在*特性文件*中，这些文件包含以特殊语法格式化的测试，这种语法称为**Gherkin**。这些测试，被称为**特性**，并组织成场景，读起来就像普通的英语。这有几个优点。

首先，它们可以被整个团队编写和理解，而不仅仅是开发者。这意味着你可以在开发团队之外扩展测试驱动实践。

其次，代码的缺失鼓励你编写专注于用户行为的测试，而不是软件的技术细节。这反过来又鼓励你为用户构建正确的东西。

Cucumber 如何将特性转换为可执行代码？好吧，看看特性文件中的一个示例行（或步骤）：

```js
When I navigate to the "/birthdays" page
```

黄瓜（Cucumber）执行这一步并寻找一个匹配的 JavaScript 定义的步骤定义。在这个例子中，步骤定义看起来是这样的：

```js
When(
  'I navigate to the {string} page',
  async function (url) { ... }
);
```

注意步骤定义有一个相关的代码块。一旦 Cucumber 找到一个匹配的步骤定义，它就会执行这个代码块，并传递任何解析后的参数。在这种情况下，`url`参数将以`/birthdays`字符串的形式提供。Cucumber 还支持其他数据类型，例如`int`和`bigdecimal`（[`github.com/cucumber/cucumber-expressions#parameter-types`](https://github.com/cucumber/cucumber-expressions#parameter-types)）。

在本章中，我们将涵盖以下关键主题：

+   创建特性文件

+   设置 Playwright 世界对象

+   实现步骤定义

到本章结束时，你将自信地为你的应用程序添加 Cucumber 测试。

# 技术要求

本章的代码可以在网上找到，地址为[`github.com/PacktPublishing/Svelte-with-Test-Driven-Development/tree/main/Chapter13/Start`](https://github.com/PacktPublishing/Svelte-with-Test-Driven-Development/tree/main/Chapter13/Start)。

# 创建特性文件

我们首先编写一个示例 Cucumber 特性文件。

用于编写特性的 Gherkin 语法以三个词*Given*、*When*和*Then*为特征。这些与所有良好单元测试的*安排*、*行动*和*断言*部分相对应，因此应该感觉熟悉。

让我们直接从特性文件开始，看看执行测试时会发生什么：

1.  首先，添加一个新的目录`features`，并在其中创建一个名为`features/birthdays.feature`的新文件，内容如下。它描述了一个用户场景，其中*生日*应用程序已经支持编辑生日。如下所示：

    ```js
    Feature: Editing a birthday
      Scenario: Correcting the year of birth
        Given An existing birthday for "Hercules" on
          "1992-03-04"
        When I navigate to the "/birthdays" page
        And I edit the birthday for "Hercules" to be
          "1994-04-06"
        Then the birthday for "Hercules" should show
          "1994-04-06"
        And the text "1992-03-04" should not appear on the
          page
    ```

1.  使用以下命令安装 Cucumber：

    ```js
    npm install --save-dev @cucumber/cucumber
    ```

1.  然后，使用 `npx @cucumber/cucumber` 运行测试。你应该会看到一个输出，开始如下：

    ```js
    Failures:
    1) Scenario: Correcting the year of birth #
         features/birthdays.feature:2
       ? Given An existing birthday for "Hercules" on
         "1992-03-04"
       Undefined. Implement with the following snippet:
         Given('An existing birthday for {string} on
         {string}', function (string, string2) {
            // Write code here that turns the phrase above
                 into concrete actions
               return 'pending';
         });
    ```

正如所有好的测试运行器应该做的那样，Cucumber 正在告诉我们下一个任务：定义步骤定义。但在我们到达那里之前，我们需要引入 Playwright API。

# 设置 Playwright 世界对象

Cucumber 如何执行你的测试？就像 Playwright 测试一样，我们需要一个运行中的应用服务器和一个运行的浏览器来驱动 **用户界面**（**UI**）。在本节中，我们将编写所有准备测试执行的代码。

Cucumber.js 使用每个步骤中的 `this` 变量的概念。我们也在特殊的 `Before` 和 `After` 钩子中获得了对它的访问，这些钩子在每次场景前后运行。

世界对象应包含允许你驱动 UI 的函数（和状态）。由于你已经学习和使用了 Playwright API 来定位页面上的对象，如果我们能够使用相同的 API 那将是极好的。实际上，我们确实可以这样做。我们还可以使用我们熟悉的相同 `expect` API，我们将在下一节开始编写步骤定义时这样做。

这里我们要做的是：我们将构建一个名为 `PlaywrightWorld` 的世界级类，它具有以下功能：

+   `launchServer` 和 `killServer` 用于启动和停止服务器

+   `launchBrowser` 和 `closeBrowser` 用于打开和关闭无头网络浏览器，并在我们的世界对象上公开 Playwright 页面和 `request` API

然后，我们将使用 `Before` 和 `After` 钩子来启动和停止服务器和浏览器。

在我们开始之前的一个注意事项：我们的代码文件将使用 `mjs` 扩展名而不是 `js`，以向 `Cucumber.js` 表明这些文件使用 ECMAScript 模块语法。

让我们开始吧：

1.  首先，创建一个新文件，`features/support/world.mjs`，包含以下导入定义。我们稍后会添加更多，但这些都是启动服务器所需的：

    ```js
    import * as childProcess from 'child_process';
    import {
      setWorldConstructor
    } from '@cucumber/cucumber';
    ```

1.  现在，定义 `removeAnsiColorCodes` 函数。这对于会返回 `stdout` 流数据中颜色代码的执行环境（主要是 Windows）来说很重要：

    ```js
    const removeAnsiColorCodes = (string) =>
      string.replace(/\x1b\[[0-9;]+m/g, '');
    ```

1.  我们准备好定义 `PlaywrightWorld` 类，从单个方法 `launchServer` 开始。该方法以调用 `setWorldConstructor` 结尾，使其成为指定的世界类：

    ```js
    class PlaywrightWorld {
      async launchServer() {
        console.log('launching server');
        this.serverProcess = childProcess.spawn(
          config.webServer.command,
          [],
          { shell: true, env: config.webServer.env }
        );
        this.baseUrl = await new Promise((resolve) => {
          this.serverProcess.stdout.on('data', (data) => {
          let text = removeAnsiColorCodes(String(data));
          let match = text.match(
            /http[s]?:\/\/[a-z]+:[0-9]+\//
          );
          if (match) {
            resolve(match[0]);
          }
        });
        console.log(`started at ${this.baseUrl}`);
      }
    }
    setWorldConstructor(PlaywrightWorld);
    ```

我们 `launchServer` 函数中的代码非常简陋，但它完成了工作。它读取 Playwright 配置文件，并提取 `config.webServer.command` 的值，在我的项目中是这样的：

```js
npm run build && npm run preview
```

使用 Playwright 配置启动网络服务器

因为这是一个 shell 命令，所以当我们调用 Node 的 `childProcess.spawn` 函数时，必须使用 `detached` 和 `shell` 属性。

设置`env`意味着 Cucumber 接收到的任何环境变量也会传递给这个新的 shell。一旦服务器启动，我们就读取`stdout`数据流，直到我们看到包含 HTTP URL 的行。这是运行中 Web 服务器的 URL，因此我们解析该值并将其作为`resolve`的参数返回。

使用`Promise`对象意味着线程将等待直到检索到值，然后将世界的`baseUrl`属性设置为该值。

1.  添加以下错误处理逻辑，它简单地记录出出现在`stderr`数据流上的任何非空消息：

    ```js
    class PlaywrightWorld {
      async launchServer() {
        console.log('launching server');
        this.serverProcess = childProcess.spawn(...);
        this.serverProcess.stderr.on('data', (data) => {
          const trimmed = String(data).trim();
          if (trimmed !== '') {
            console.log(trimmed);
          }
        });
        ...
        console.log(`started at ${this.baseUrl}`);
      }
    ```

1.  现在，让我们继续到`killServer`，首先添加一个新的包，`tree-kill-promise`，这将使我们能够轻松关闭服务器进程：

    ```js
    npm install --save-dev tree-kill-promise
    ```

1.  然后将它作为导入添加到同一个世界文件顶部：

    ```js
    import kill from 'tree-kill-promise';
    ```

1.  定义`killServer`方法，如下所示：

    ```js
    class PlaywrightWorld {
      ...
      killServer() {
        await kill(this.serverProcess.pid);
      }
    }
    ```

1.  是时候启动浏览器了。首先在文件顶部引入以下 Playwright `import`语句：

    ```js
    import { chromium, request } from '@playwright/test';
    ```

1.  然后定义`launchBrowser`和`closeBrowser`函数，如下面的代码所示。关键部分是我们最终得到了`request`和`page`对象，这些对象与我们的 Playwright 端到端测试中的对象完全相同：

    ```js
    class PlaywrightWorld {
      ...
      async launchBrowser() {
        this.browser = await chromium.launch();
        this.context = await this.browser.newContext({
          baseURL: this.baseUrl
        });
        this.request = await request.newContext({
          baseURL: `${this.baseUrl}api/`
        });
        this.page = await this.context.newPage();
      }
      async closeBrowser() {
        await this.browser.close();
      }
    }
    ```

1.  有了完整的世界对象，现在是时候添加钩子了。添加一个名为`features/support/hooks.mjs`的新文件，并给它以下内容：

    ```js
    import { Before, After } from '@cucumber/cucumber';
    Before(async function () {
      await this.launchServer();
      await this.launchBrowser();
    });
    After(async function () {
      await this.killServer();
      this.closeBrowser();
    });
    ```

所有的设置都完成了。唯一剩下的事情是步骤定义。

# 实现步骤定义

最后一部分是匹配功能步骤与其实现的`Given`、`When`和`Then`函数。

在进行过程中检查你的工作

在本节中，我们将快速浏览定义，但请确保在实现每个函数后通过运行 Cucumber（使用`npx @cucumber/cucumber`命令）来验证每个步骤是否正常工作。

让我们开始吧！

1.  创建另一个新目录，`features/support`，并创建一个名为`features/support/steps.mjs`的文件，它以以下导入开始：

    ```js
    import {
      Given,
      When,
      Then
    } from '@cucumber/cucumber';
    ```

1.  然后实现我们的功能文件中的第一个`Given`步骤。这个步骤使用 Playwright 的`this.request.post`函数调用 API。注意`failOnStatusCode`的使用，它确保如果未收到`200 OK`响应，Cucumber 将失败测试：

    ```js
    Given(
      'An existing birthday for {string} on {string}',
      async function (name, dob) {
        await this.request.post('birthdays', {
          data: { name, dob },
          failOnStatusCode: true
        });
      }
    );
    ```

1.  现在我们进入`When`步骤。这里有两大步骤；可以说，其中之一可以是`Given`，但我认为它们作为一个组工作得很好，因为它们都是用户操作。第一个步骤简单地调用`this.page.goto`，你之前也见过：

    ```js
    When(
      'I navigate to the {string} page',
      async function (url) {
        await this.page.goto(url);
      }
    );
    ```

1.  对于剩余的步骤，我们将使用我们在*第七章*中定义的自己的`BirthdayListPage`页面模型对象，*整理测试套件*。首先在文件顶部导入它：

    ```js
    import {
      BirthdayListPage
    } from '../../tests/BirthdayListPage.js';
    ```

1.  然后实现下一个`When`步骤定义，它使用我们经过考验的`beginEditingFor`、`dateOfBirthField`和`saveButton`函数：

    ```js
    When(
      'I edit the birthday for {string} to be {string}',
      async function (name, dob) {
        const birthdayListPage = new BirthdayListPage(
          this.page
        );
        await birthdayListPage.beginEditingFor(name);
        await birthdayListPage
          .dateOfBirthField()
          .fill(dob);
        await birthdayListPage.saveButton().click();
      }
    );
    ```

1.  是时候编写 `Then` 步骤定义了。这些是包含期望的步骤定义。首先，在文件顶部添加一个 `import` 语句用于 `expect`：

    ```js
    import { expect } from '@playwright/test';
    ```

1.  实现以下代码块中的第一个 `Then` 子句。这会检查新生日是否显示在页面上：

    ```js
    Then(
      'the birthday for {string} should show {string}',
      async function (name, dob) {
        const birthdayListPage = new BirthdayListPage(
          this.page
        );
        await expect(
          birthdayListPage.entryFor(name)
        ).toContainText(dob);
      }
    );
    ```

1.  使用最后的 `Then` 子句完成步骤定义：

    ```js
    Then(
      'the text {string} should not appear on the page',
      async function (text) {
        await expect(
          this.page.getByText(text)
        ).not.toBeVisible();
      }
    );
    ```

1.  现在运行测试，你应该会看到所有测试都通过：

    ```js
    work/birthdays % npx @cucumber/cucumber
    launching server
    started at http://localhost:4173/
    .......
    1 scenario (1 passed)
    5 steps (5 passed)
    ```

你现在已经看到了如何使用 Gherkin 语法编写功能文件，以及如何编写使用与您用于端到端测试相同的 Playwright API 的步骤定义。你还看到了标准 `expect` 语法可以用于编写适用于所有三种测试类型的断言。

# 摘要

本章介绍了如何使用 Cucumber 测试运行器执行 Gherkin 功能文件。Gherkin 的纯英文语法使得这项技术对于将自动化测试引入更广泛的产品开发团队变得非常重要。

功能文件由步骤定义支持。这些步骤定义是通过使用 `Given`、`When` 和 `Then` 函数实现的，这些函数将 Gherkin 步骤描述映射到 JavaScript 代码。

你已经看到了如何使用步骤定义重复使用现有的 Playwright API 代码来管理浏览器交互。

这完成了我们对自动化测试技术的探讨。在 *第三部分* 中，我们将探讨如何为 SvelteKit 特定功能编写单元测试，从测试认证的策略章节开始。

# 第三部分：测试 SvelteKit 功能

本部分简要介绍了需要仔细测试的一些特定功能。这些章节的顺序与前面部分不同。相反，它们是关于你如何进行测试的讨论。包含的代码示例仅关注前面章节中没有涉及的新颖部分。你始终可以参考在线存储库以获取完整的实现。

本部分包含以下章节：

+   *第十四章*，*测试认证*

+   *第十五章*，*测试 Svelte 存储*

+   *第十六章*，*测试服务工作者*
