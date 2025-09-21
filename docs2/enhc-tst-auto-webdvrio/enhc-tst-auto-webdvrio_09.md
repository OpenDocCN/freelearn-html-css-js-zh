# 9

# 古代咒语书 - 构建页面对象模型

所有框架都有三到四个抽象层。这最好被想象为一本古代咒语书。最简单的咒语在开头。这些是构建中间更复杂和强大的魔法的基础。而最黑暗的秘密总是隐藏在神秘的咒语书末尾。同样，有一个测试层，它调用中间页面对象类层中引用对象的方法，而这个类层又利用底层核心层中的辅助包装和其他功能。在 Cucumber 框架中，从测试功能文件层到步骤定义、到粘合代码再到核心代码层，还有一个额外的抽象层。在本章中，我们专注于创建页面元素和用于在页面上执行操作的这些方法。

在此之前，以下是本章我们将涵盖的所有主要主题列表：

+   理解页面对象模型是什么

+   为测试创建页面类

+   添加对象选择器

+   module.exports 语句

+   通过常用对象和方法减少代码

+   使用 Klassi-js 的 POM

# 技术要求

所有测试示例都可以在这个 GitHub 仓库中找到：[`github.com/PacktPublishing/Enhanced-Test-Automation-with-WebdriverIO`](https://github.com/PacktPublishing/Enhanced-Test-Automation-with-WebdriverIO)

# 什么是页面对象模型？

**页面对象模型**（**POM**）是一种在测试自动化中使用的模式，用于创建一个结构化和可维护的框架，用于 Web 应用程序测试。它促进了测试代码与网页实现细节的分离。

在 POM 中，每个网页都表示为一个单独的类，页面的属性和行为封装在这个类中。测试方法通过页面类提供的方法与网页交互，而不是直接访问 Web 元素或使用低级浏览器 API。

使用 WebdriverIO 或 Klassi-js 等框架，可以在 Node.js 中实现 POM

# 什么构成了一个好的页面对象模式？

在测试自动化中，一个好的页面对象模式是那种促进代码的可维护性、可重用性和可读性的模式。一个良好实现的页面对象模式的一些特征包括：

+   **遵循单一职责原则（SRP）**：每个页面类应该有一个单一职责，并代表应用程序的特定页面或组件。这确保了代码组织良好且易于维护。

+   **封装**：页面类应该封装它所代表的网页或组件的细节和行为。它应该提供方法来与页面元素交互，而不暴露底层实现细节。这种抽象简化了测试代码，并使其更易于阅读。

+   **模块化和可重用**：页面类应模块化，并在不同的测试和测试套件中可重用。它们应提供一致的接口来与页面元素交互，从而便于重用并减少代码重复。

+   **遵循关注点分离（SoC）原则**：页面对象模式将测试逻辑与网页的实现细节分离。测试方法应利用页面类提供的方法，而不是直接与网页元素交互或使用低级浏览器 API。这种分离提高了代码的可维护性，并使得在应用程序更改时更新测试变得更加容易。

+   **独立于测试框架**：页面类应独立于所使用的特定测试框架。它们不应依赖于测试框架，例如断言或测试执行逻辑。这确保了页面类可以轻松地与不同的测试框架或工具一起重用。请看以下示例：

    ```js
    class LoginPage {
      get usernameField() { return $('#username'); }
      get passwordField() { return $('#password'); }
      get loginButton() { return $('#login'); }
      enterUsername(username) {
        this.usernameField.setValue(username);
      }
    ```

+   **清晰的命名约定**：页面类、方法和变量应具有有意义的描述性名称，准确反映其目的和功能。这有助于提高代码的可读性和理解性：

    ```js
    loadPage: Async (url, seconds) => {
    Await browser.url(url, seconds)
    }
    ```

+   **定期维护**：页面类应随着应用程序的发展而定期维护和更新。它们应与应用程序 UI 和功能的变化保持同步。定期审查和更新页面类有助于确保其准确性和可靠性。

# 为测试创建页面类

我们已创建一个 `LoginPage` 类，代表网络应用程序的特定页面。网页元素选择器定义为使用 WebdriverIO 的 `$` 函数作为获取器，这允许我们使用 CSS 选择器在页面上定位元素。

该类还包括页面方法，如 `enterUsername`、`enterPassword` 和 `clickLoginButton`。这些方法封装了可以在页面上执行的操作，例如在输入字段中输入文本和点击按钮。

Linux/Unix 中的 `mkdir` 命令允许用户创建或新建目录。`mkdir` 代表“创建目录”：

转到你的 `mkdir` 命令：

```js
mkdir loginPage.ts
homePage.ts
testClass.ts
```

# 添加对象选择器

`TestClass` 测试类利用了页面类的导出实例。在测试用例中，我们通过页面对象中定义的方法与网页进行交互。

## // LoginPage.ts

`LoginPage`：此类封装了相应网页的属性和行为：

```js
class LoginPage {
  get usernameField() { return $('#username'); }
  get passwordField() { return $('#password'); }
  get loginButton() { return $('#login'); }
  enterUsername(username) {
    this.usernameField.setValue(username);
  }
  enterPassword(password) {
    this.passwordField.setValue(password);
  }
  clickLoginButton() {
    this.loginButton.click();
  }
}
module.exports = new LoginPage();
```

## // HomePage.ts

`HomePage`：此类封装了相应网页的属性和行为：

```js
class HomePage {
  get welcomeMessage() { return $('#welcome'); }
  get logoutButton() { return $('#logout'); }
  getWelcomeMessage() {
    return this.welcomeMessage.getText();
  }
  clickLogoutButton() {
    this.logoutButton.click();
  }
}
```

# `module.exports = new HomePage();` 调用测试中要使用的方法

使用 `module.exports` 语句导出每个页面类的实例作为一个模块：

```js
module.exports = new HomePage();
module.exports = new loginPage();
```

## // TestName.ts

`TestName`测试文件利用页面类的导出实例。在这个示例测试用例中，我们使用在相应页面类中定义的方法和对象与 LoginPage 和 HomePage 网页进行交互：

```js
import LoginPage from('../PageObjects/LoginPage');
import HomePage from('../PageObjects/HomePage');
import assert from('assert');
describe('Test Name', () => {
  before(() => {
    // Set up WebDriverIO configuration
  });
  it('should perform login and logout', () => {
    LoginPage.enterUsername('username');
    LoginPage.enterPassword('password');
    LoginPage.clickLoginButton();
    const welcomeMessage = HomePage.getWelcomeMessage();
    assert.strictEqual(welcomeMessage, 'Welcome, User!');
    HomePage.clickLogoutButton();
  });
  after(() => {
    // Quit WebDriverIO instance
  });
});
```

# 使用常见对象和方法减少代码

通过利用页面对象模式中的常见对象和方法，可以减少代码重复并提高可维护性。以下是一些实现代码减少的策略：

+   **基础页面类**: 创建一个包含多个页面共享的常见对象和方法的基页面类。这个基类可以封装多个页面共有的元素和行为，例如一个**主页**按钮、一个**万圣节派对**按钮，然后是一个**寻找我的糖果！**按钮，以减少重复：

![图 9.1 – CandyMapper 派对页面网站头部，包含所有页面共有的链接](img/B19395_09_1.jpg)

图 9.1 – CandyMapper 派对页面网站头部，包含所有页面共有的链接

这些元素出现在网站每个页面的头部。因此，在顶级页面类中声明它们并将其扩展到所有其他页面是有意义的。

![图 9.2 – Candymapper 着陆页面头部，包含三个常见链接](img/B19395_09_2.jpg)

图 9.2 – Candymapper 着陆页面头部，包含三个常见链接

如果选择器在每个页面类中，随着时间的推移，维护级别将会增加。因此，我们将创建选择器在公共页面类中，如下所示：

```js
  get homeButton() {return $(`//a[text()='Home']`); }
```

![图 9.3 – 链接选择器中突出显示的 HOME 链接](img/B19395_09_3.jpg)

图 9.3 – 链接选择器中突出显示的 HOME 链接

让我们花点时间看看这三个例子，因为可能需要进行一些更改。这些选择器匹配页面头部和底部的按钮。我们可以锁定到第一个元素匹配，如下所示：

```js
get homeButton() {return $(`(//a[text()='Home'])[1]`); }
get halloweenPartyButton() {return $(` (//a[contains(text(), 'Party')])[1]`); }
```

![图 9.4 – 为第一个万圣节派对链接识别的定位器](img/B19395_09_4.jpg)

图 9.4 – 为第一个万圣节派对链接识别的定位器

由于这些元素在所有我们的页面中都是通用的，因此创建一个通用的基`Page`类并将它们全部存储在那里是有意义的：

```js
export default class Page {
  get homeButton() {return $(`//a[text()='Home']`); }
  get halloweenPartyButton() {return $(` (//a[contains(text(), 'Party')])[1]`); }
}
```

其他页面类现在可以继承这个基`Page`类及其所有常见对象和功能。我们可以使用`extends`关键字将这些对象添加到任何`Page`类中：

```js
import * as helpers from "../../helpers/helpers";
import Page from "./page";
class CandymapperPage extends Page{
  await helpers.clickAdv(await    
    super.halloweenPartyButton);
}
```

最后，我们使用`super`关键字来引用公共父类中的对象和方法，以减少重复代码。

如果我们发现常见元素的文本在不同页面之间或在不同版本中频繁变化，我们可以使用以下方法来减少维护。考虑下面的‘FIND MY CANDY’链接元素：

```js
 get findMyCandyButton() {return $(` (//a[contains(translate(normalize-space(), 'ABCDEFGHIJKLMNOPQRSTUVWXYZ', 'abcdefghijklmnopqrstuvwxyz'), 'my candy') and not(contains(@style, 'display: none')) and not(contains(@style, 'visibility: hidden'))])[1]`); }
```

![图 9.5 – 对“寻找我的糖果！”链接的忽略大小写匹配定位器](img/B19395_09_5.jpg)

图 9.5 – 对“寻找我的糖果！”链接的忽略大小写匹配定位器

这是通过元素的样式显示或可见性属性来完成的。有一个技巧可以找到仅可见的元素。最终，可以返回一个元素集合，并对每个元素进行立即可见性的检查。

其他页面类可以继承这个基类并继承其通用功能。

+   **页面组件**：识别应用程序页面中重复出现在多个页面上的常见组件或部分。创建单独的页面组件类来表示这些可重用组件。然后，将这些组件包含在页面类中以重用通用功能并减少代码重复。

+   **辅助方法**：识别在多个页面中执行的操作或动作，例如登录、在页面之间导航或处理弹出窗口。将这些操作提取到可以从不同的页面类中调用的辅助方法中。这集中了实现并避免了为这些常见动作重复编写代码。

+   **参数化**：如果您有基于输入参数而变化的类似元素或动作，您可以在页面类中创建参数化方法。这些方法可以接受参数并根据提供的输入执行所需动作，从而减少为类似功能创建单独方法的需求。

+   **外部配置**：将可配置的值，如 URL、超时或测试数据，移动到外部配置文件中。这允许您在多个测试和页面之间集中和重用配置，减少在单个页面类中硬编码值的需求。

+   **可维护的选择器**：使用可靠且可维护的方式来定位页面上的元素，例如 CSS 选择器或 XPath 表达式。避免在测试方法中使用硬编码的选择器。相反，将选择器定义为页面类中的属性，这样在 UI 发生变化时更容易更新它们。

# 使用 Klassi-js 的 POM

Klassi-js 是一个强大且灵活的**行为驱动开发**（**BDD**）JavaScript 测试自动化框架，它赋予开发人员和 QA 专业人员创建和执行针对 Web 和移动应用程序的全面测试的能力。在其核心，Klassi-js 利用了 WebdriverIO 的力量，这是一个 Node.js 的尖端自动化框架。这个基础使得 Klassi-js 能够无缝地与 Web 浏览器和移动设备交互，使其成为跨浏览器和跨平台测试的绝佳选择。

Klassi-js 的一个突出特点是它与 cucumber.js 的无缝集成，cucumber.js 是一个流行的 BDD 测试工具。这种集成允许创建人类可读的、富有表现力的测试场景，这有助于促进开发人员、测试人员和其他利益相关者之间的更好协作。它推广了一种共同的语言来讨论应用程序的行为，并有助于构建更可靠的测试。

Klassi-js 更进一步，通过提供集成视觉、可访问性和 API 测试功能，确保你的应用程序不仅功能正常，而且用户友好，符合可访问性标准，并能够提供预期的 API 响应。此外，Klassi-js 提供了在本地运行测试或利用基于云的测试平台（如 LambdaTest、BrowserStack 或 Sauce Labs）的灵活性，允许在各种环境中进行可扩展和高效的测试。

POM 是一种设计模式，用于组织你的 UI 自动化代码，使其更易于维护和阅读。当使用 Klassi-js 与 Cucumber 结合时，你可以按照以下方式实现 POM 设计模式。

## 项目结构

首先，组织你的项目结构以分离不同的关注点。一个常见的结构可能如下所示：

```js
project-root
-- features
     -- login.feature
-- step-definitions
     -- login.steps.ts
-- pages
     -- login.page.ts
-- index.ts
-- runtime
-- world.ts
-- package.json
```

让我们逐一分析：

+   `features`：存储你的 Cucumber 功能文件

+   `step-definitions`：存储你的 Cucumber 步骤定义

+   `pages`：存储你的页面对象

## Cucumber 功能文件

在你的项目的`features`目录中创建 Cucumber 功能文件。这些功能文件以纯文本形式描述了应用程序的行为。例如，你可以创建一个`login.feature`文件：

```js
Feature: Login functionality
  Scenario: Successful login
    Given I am on the login page
    When I enter my username and password
    And I click the login button
    Then I should be logged in
```

## 页面对象

为你想要交互的每个网页或组件创建一个页面对象。以下是一个示例，`login.page.ts`:

```js
class LoginPage {
    get usernameInput() { return $('#username'); }
    get passwordInput() { return $('#password'); }
    get loginButton() { return $('#login-button'); }
    get welcomeMessage() { return $('#welcome-message'); }
    open() {
        browser.url('/login'); // Adjust the URL as needed
    }
    login(username, password) {
        this.usernameInput.setValue(username);
        this.passwordInput.setValue(password);
        this.loginButton.click();
    }
}
module.exports = new LoginPage();
```

## Cucumber 步骤定义

在你的步骤定义中，使用页面对象与网页元素进行交互。以下是一个示例，`login.steps.ts`:

```js
const { Given, When, Then } = require('cucumber');
const LoginPage = require('../pageobjects/login.page');
Given('I am on the login page', () => { LoginPage.open(); });
When('I enter my username and password', () => {
    LoginPage.username.setValue('your_username');
    LoginPage.password.setValue('your_password');
});
When('I click the login button', () => {  LoginPage.loginButton.click(); });
Then('I should be logged in', () => {
    expect(LoginPage.welcomeMessage).toHaveText('Welcome, User');
});
```

## 运行测试

你可以使用 Klassi-js 像往常一样运行你的 Cucumber 测试。使用如下命令：

```js
> node index.ts --tags @login
```

Klassi-js 将自动发现你的 Cucumber 功能文件并执行相应的步骤定义。

采用这种结构，你的 UI 自动化代码变得更加模块化，更容易维护。每个页面对象封装了特定页面或组件的功能和交互，使得更新和管理测试变得更加容易。

# 概述

遵循上述原则，你可以创建一个类似于超级英雄细致规划的页面对象模式，确保你的代码更加易于维护、可重用和可读。就像超级英雄调整他们的能力以更有效，通过遵循这些策略，你可以减少代码重复并提高页面对象模式的可维护性。跨页面重用常见对象、方法和组件就像利用超级英雄工具带的隔间，简化你的代码并确保修改易于实施和维护。

在下一章中，我们将通过利用系统变量和动态配置来进一步增强我们框架的能力，就像超级英雄适应不同的环境，无缝地在开发版和预发布版着陆页之间切换。
