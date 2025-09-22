# React 中的端到端测试

对于我们的后端开发，我们坚决遵循 **测试驱动开发**（**TDD**）——我们通过编写端到端测试开始开发，并编写了一些实现代码来使这些测试通过。在实现了这个功能之后，我们添加了单元和集成测试来增加我们对底层代码的信心，并帮助捕获回归。

现在我们对 React 有了一个基本的了解，在本章中，我们将探讨如何在 React 中实现 TDD。具体来说，我们将涵盖：

+   使用 **Selenium** 自动化与浏览器的交互

+   使用 **React Router** 实现 **客户端路由**

# 测试策略

事实证明，前端上的 TDD 采用类似的方法，涉及自动化 UI 测试和单元测试。

# 自动化 UI 测试

当我们为我们的 API 编写端到端测试时，我们首先编写我们的请求，发送它，并断言它返回预期的结果。换句话说，我们的端到端测试是在模仿最终用户如何与我们的 API 交互。对于前端，用户将通过用户界面（UI）与我们的应用程序交互。因此，端到端测试的对应物将是自动化 UI 测试。

UI 测试自动化了应用程序用户可能采取的操作。例如，如果我们想测试用户可以注册，我们会编写一个测试，它：

+   导航到 `/register` 页面

+   输入电子邮件

+   输入密码

+   点击注册按钮

+   断言用户已注册

这些测试可以用 Gherkin 编写，并使用 Cucumber 运行。实际的用户操作模拟可以通过使用像 Selenium 这样的浏览器自动化工具来自动化。例如，当我们运行测试步骤“点击注册按钮”时，我们可以指示 Selenium 选择具有 `id` 值 `register-button` 的按钮并触发其上的点击事件。

# 单元测试

对于前端，单元测试涉及两个不同的方面——逻辑单元和组件单元。

# 逻辑单元

单元可以是一个不与 UI 交互的函数或类；例如 `validateInput` 函数就是一个很好的例子。这些逻辑单元使用纯 JavaScript，并且应该能够独立于环境工作。因此，我们可以使用 Mocha、Chai 和 Sinon 以与我们的后端代码相同的方式对它们进行单元测试。

因为逻辑单元是最容易测试的。你应该尽可能多地提取应用程序逻辑并对其进行测试。

# 组件单元

单元也可能指 React 中的单个组件。例如，我们可以测试当输入改变时，组件的状态以预期的方式更新；或者对于受控组件，正确的回调是否以正确的参数被调用。

# 浏览器测试

多亏了无头浏览器——这些浏览器不会渲染到显示界面——我们可以在服务器上运行端到端（E2E）和单元测试。然而，我们也应该在真实浏览器中测试这些单元测试，因为 NodeJS（使用 V8 JavaScript 引擎）和其他浏览器（如 Firefox 使用 SpiderMonkey 引擎、Microsoft Edge 使用 Chakra 引擎、Safari 使用 Nitro 引擎）之间可能存在不一致性。

要在真实浏览器和设备上测试，我们可以使用一个名为*Karma*（[`karma-runner.github.io/2.0/index.html`](https://karma-runner.github.io/2.0/index.html)）的不同测试运行器。

# 使用 Gherkin、Cucumber 和 Selenium 编写端到端测试

现在，我们准备好与可以模拟用户与浏览器交互的工具集成。对于我们的第一个测试，让我们测试一个非常简单的事情——用户将输入一个有效的电子邮件，但他们的密码太短。在这种情况下，我们想要断言注册按钮将被禁用。

就像我们的后端端到端测试一样，我们将使用 Gherkin 编写测试用例，并使用 Cucumber 运行场景。所以，让我们将这些添加为开发依赖项：

```js
$ yarn add cucumber babel-register --dev
```

然后，我们需要创建功能文件和步骤定义文件。对于我们的第一个场景，我选择按照以下结构将功能和步骤分组：

```js
$ tree --dirsfirst spec
spec
└── cucumber
 ├── features
 │   └── users
 │       └── register
 │           └── main.feature
 └── steps
 ├── assertions
 │   └── index.js
 ├── interactions
 │   ├── input.js
 │   └── navigation.js
 └── index.js
```

随意分组，只要确保功能与步骤定义分开。

# 添加测试脚本

尽管我们还没有编写任何测试，但我们可以简单地复制我们为 API 编写的测试脚本，并将其放置在`scripts/e2e.test.sh`中：

```js
#!/bin/bash

# Set environment variables from .env and set NODE_ENV to test
source <(dotenv-export | sed 's/\\n/\n/g')
export NODE_ENV=test

# Run our web server as a background process
yarn run serve > /dev/null 2>&1 &

# Polling to see if the server is up and running yet
TRIES=0
RETRY_LIMIT=50
RETRY_INTERVAL=0.2
SERVER_UP=false
while [ $TRIES -lt $RETRY_LIMIT ]; do
  if netstat -tulpn 2>/dev/null | grep -q ":$SERVER_PORT_TEST.*LISTEN"; then
    SERVER_UP=true
    break
  else
    sleep $RETRY_INTERVAL
    let TRIES=TRIES+1
  fi
done

# Only run this if API server is operational
if $SERVER_UP; then
  # Run the test in the background
  npx dotenv cucumberjs spec/cucumber/features -- --compiler js:babel-register --require spec/cucumber/steps &

  # Waits for the next job to terminate - this should be the tests
  wait -n
fi

# Terminate all processes within the same process group by sending a SIGTERM signal
kill -15 0
```

我们脚本和后端测试脚本之间的唯一区别是这一行：

```js
yarn run serve > /dev/null 2>&1 &
```

使用`> /dev/null`，我们将`stdout`重定向到`null device`（`/dev/null`），它会丢弃任何被管道传输的内容。使用`2>&1`，我们将`stderr`重定向到`stdout`，最终也会到达`/dev/null`。基本上，这一行是在说“我不关心`yarn run serve`的输出，只是把它扔掉”。

我们这样做是因为，当 Selenium 在页面之间导航时，`http-server`的输出将被发送到`stdout`并穿插在测试结果之间，这使得阅读变得困难。

此外，别忘了安装脚本的依赖项：

```js
$ yarn add dotenv-cli --dev
```

我们还需要创建一个`.babelrc`文件来指导`babel-register`使用`env`预设：

```js
{
  "presets": [
    ["env", {
      "targets": {
        "node": "current"
      }
    }]
  ]
}
```

最后，更新`package.json`以包含新的脚本：

```js
"scripts": {
  "build": "rm -rf dist/ && webpack",
  "serve": "./scripts/serve.sh",
  "test:e2e": "./scripts/e2e.test.sh"
}
```

# 指定一个功能

现在，我们准备好定义我们的第一个功能。在`spec/cucumber/features/users/reigster/main.feature`中添加以下规范：

```js
Feature: Register User

  User visits the Registration Page, fills in the form, and submits

  Background: Navigate to the Registration Page

    When user navigates to /

  Scenario: Password Too Short

    When user types in "valid@ema.il" in the "#email" element
    And user types in "shortpw" in the "#password" element
    Then the "#register-button" element should have a "disabled" attribute
```

# 给元素添加 ID

我们将使用 Selenium 来自动化与我们的应用程序 UI 元素的交互。然而，我们必须提供某种选择器，以便 Selenium 可以选择我们想要与之交互的元素。我们可以拥有的最精确的选择器是一个`id`属性。

因此，在我们使用 Selenium 之前，让我们给我们的元素添加一些 ID。打开`src/components/registration-form/index.jsx`并给每个元素添加一个`id`属性：

```js
<Input label="Email" type="email" name="email" id="email" ... />
<Input label="Password" type="password" name="password" id="password" ... />
<Button title="Register" id="register-button" ... />
```

然后，在`src/components/input/index.jsx`和`src/components/button/index.jsx`中，将`id`属性作为属性传递给元素。例如，`Button`组件将变为：

```js
function Button(props) {
  return <button id={props.id} disabled={props.disabled}>{props.title}</button>
}
```

# Selenium

我们现在可以开始使用 Selenium 了。Selenium 是由 Jason Huggins 在 2004 年，在 ThoughtWorks 工作时编写的。它不仅仅是一个单一的工具，而是一套工具，允许您跨多个平台自动化浏览器。我们将使用 Selenium WebDriver 的 JavaScript 绑定，但快速浏览工具套件的每个部分对我们来说是有益的：

+   Selenium **远程控制**（**RC**），也称为 Selenium 1.0，是套件中的第一个工具，允许您自动化浏览器。它通过在页面首次加载时向浏览器注入 JavaScript 脚本来实现，这些脚本通过点击按钮和输入文本来模拟用户交互。Selenium RC 已被弃用，并由 Selenium WebDriver 取代。

+   Selenium WebDriver，也称为 Selenium 2，是 Selenium RC 的继任者，并使用标准化的 WebDriver API 来模拟用户交互。大多数浏览器都内置了对 WebDriver API 的支持，因此工具不再需要将脚本注入页面。

+   Selenium Server 允许您在远程机器上运行测试，例如在使用 Selenium Grid 时。

+   Selenium Grid 允许您将测试分布在多台机器或虚拟机（VM）上。然后，这些测试可以并行运行。如果您的测试套件很大，或者您需要在多个浏览器和/或操作系统上运行测试，那么测试执行可能需要很长时间。通过将这些测试分布在多台机器上，您可以并行运行它们，从而减少总执行时间。

+   Selenium IDE 是一个 Chrome 扩展/Firefox 插件，它提供了一个快速原型设计工具，用于构建测试脚本。本质上，它可以记录用户在页面上的操作，并将它们导出为许多语言的可重用脚本。然后，开发者可以取用这个脚本，并根据他们的需求进一步定制它。

为了测试我们的应用程序，我们将使用 Selenium WebDriver。

# WebDriver API

*WebDriver*是一个标准化的 API，允许您检查和控制用户代理（例如，浏览器或移动应用程序）。它最初由当时在谷歌工作的工程师 Simon Stewart 在 2006 年构思。现在，它已经被万维网联盟（W3C）定义，其规范可以在[`www.w3.org/TR/webdriver/`](https://www.w3.org/TR/webdriver/)找到。该文档目前处于*候选推荐*阶段。

与将 JavaScript 脚本注入网页并使用它们来模拟用户交互不同，Selenium WebDriver 使用 WebDriver API，大多数浏览器都支持这个 API。然而，您可能会看到在不同浏览器之间对标准的支持程度以及实现方式的差异。

虽然 API 是平台和语言中立的，但已经有许多实现。具体来说，我们将使用官方的 JavaScript 绑定，它作为 "selenium-webdriver" 包在 NPM 上提供。

# 使用 Selenium WebDriver

让我们从向我们的项目中添加 Selenium WebDriver JavaScript 包开始：

```js
$ yarn add selenium-webdriver --dev
```

我们将使用 `selenium-webdriver` 来定义我们的步骤定义。

Selenium 需要一个浏览器来运行测试。这可能是一个真实的浏览器，如 Chrome，或者一个无头浏览器，如 PhantomJS。你很可能熟悉不同的真实浏览器，所以让我们花些时间来看看无头浏览器。

# 无头浏览器

无头浏览器是那些不在界面上渲染页面的浏览器。一个头浏览器会获取页面内容，然后下载图片、样式表、脚本等，并像真实浏览器一样处理它们。

使用无头浏览器的优点是它要快得多。这是因为浏览器没有图形用户界面 (GUI)，因此不需要等待显示实际渲染输出：

+   PhantomJS ([`phantomjs.org/`](http://phantomjs.org/)) 使用 WebKit 网络浏览器引擎，这与 Safari 所使用的相同。它可以说是目前最流行的无头浏览器。然而，自 2016 年中以来，其仓库的活动几乎已经停止。

+   SlimerJS ([`slimerjs.org/`](https://slimerjs.org/)) 使用 Gecko 网络浏览器引擎，以及 SpiderMonkey 作为 JavaScript 引擎，这与 Firefox 相同。SlimerJS 默认不是一个无头浏览器，因为它在测试机器上使用 X11 显示服务器。然而，你可以通过与 *Xvfb*（即 *X 虚拟帧缓冲区*）集成来使用它，这是一个内存中的显示服务器，不需要显示。自 Firefox 56 版本起，你也可以通过 `--headless` 标志启用无头模式。

+   ZombieJS ([`zombie.js.org/`](http://zombie.js.org/)) 是一个无头浏览器的更快实现，因为它不使用像 PhantomJS 或 SlimerJS 这样的实际网络浏览器引擎。相反，它使用 JSDOM，这是一个 DOM 和 HTML 的纯 JavaScript 实现。然而，正因为如此，结果可能不会完全准确，或者不如针对实际网络浏览器引擎的测试那样真实。

+   HtmlUnit ([`htmlunit.sourceforge.net/`](http://htmlunit.sourceforge.net/)) 是一个“无 GUI 浏览器，用于 Java 程序”。它使用 Rhino JavaScript 引擎，与 Selenium 一样，是用 Java 编写的。根据经验，HtmlUnit 是最快的无头浏览器，但也是最容易出现错误的。它非常适合简单的静态页面，不涉及大量 JavaScript 使用。

还有更多无头浏览器。Asad Dhamani 编制了一个列表，您可以在 [`github.com/dhamaniasad/HeadlessBrowsers`](https://github.com/dhamaniasad/HeadlessBrowsers) 找到。

然而，纯无头浏览器可能很快就会成为过去式，因为许多“真实”浏览器现在都支持无头模式。以下浏览器支持无头模式：

+   Chrome 59

+   Firefox 55（在 Linux 上）和 56（在 macOS 和 Windows 上）

对于不支持这种情况的用户，我们可以使用 Xvfb 来替代 X11 显示服务器，并在 CI 服务器上运行真实浏览器。然而，这将失去运行无头浏览器的性能优势。

# 浏览器驱动程序

Selenium WebDriver 支持许多浏览器，包括真实和无头浏览器，每个浏览器都需要实现 WebDriver 的特定浏览器线协议的驱动程序。

对于真实浏览器：

+   Chrome 和 Android 上的 Chrome 使用 ChromeDriver ([`sites.google.com/a/chromium.org/chromedriver/`](https://sites.google.com/a/chromium.org/chromedriver/))，由 Chromium 项目本身维护

+   Firefox 使用 geckodriver ([`github.com/mozilla/geckodriver/`](https://github.com/mozilla/geckodriver/))

+   Internet Explorer 使用 Internet Explorer Driver

+   Edge 使用 Microsoft WebDriver ([`developer.microsoft.com/en-us/microsoft-edge/tools/webdriver/`](https://developer.microsoft.com/en-us/microsoft-edge/tools/webdriver/))

+   Safari 使用 SafariDriver ([`webkit.org/blog/6900/webdriver-support-in-safari-10/`](https://webkit.org/blog/6900/webdriver-support-in-safari-10/))

+   Opera 使用 Opera Driver

+   iOS（原生、混合或移动 Web 应用程序）使用 ios-driver ([`ios-driver.github.io/ios-driver/`](http://ios-driver.github.io/ios-driver/))

+   Android（原生、混合或移动 Web 应用程序）使用 Selendroid

对于无头浏览器：

+   HtmlUnit 使用 HtmlUnitDriver ([`github.com/SeleniumHQ/htmlunit-driver`](https://github.com/SeleniumHQ/htmlunit-driver))

+   PhantomJS 使用 GhostDriver ([`github.com/detro/ghostdriver`](https://github.com/detro/ghostdriver))

# 设置和销毁

在我们可以运行任何测试之前，我们必须告诉 Selenium 要使用哪个浏览器。Chrome 目前是使用最广泛的浏览器，因此我们将从使用 ChromeDriver 开始。让我们来安装它：

```js
$ yarn add chromedriver --dev
```

现在，在 `spec/cucumber/steps/index.js` 中，定义在每次场景之前运行的 `Before` 和 `After` 钩子：

```js
import { After, Before } from 'cucumber';
import webdriver from 'selenium-webdriver';

Before(function () {
  this.driver = new webdriver.Builder()
    .forBrowser("chrome")
    .build();
  return this.driver;
});

After(function () {
  this.driver.quit();
});
```

在 `Before` 钩子中，我们正在创建驱动程序的新实例。驱动程序类似于用户会话，一个会话可以打开多个窗口（就像你可以在同一时间打开多个标签页一样）。

`webdriver.Builder` 构造函数返回一个实现 `ThenableWebDriver` 接口的实例，这允许我们通过链式调用方法来指定驱动程序的参数。一些常用方法包括以下内容：

+   `forBrowser`：指定要使用的浏览器。

+   `withCapabilities`：将参数传递给浏览器命令。稍后我们将使用它以无头模式运行 Chrome。

一旦设置了参数，使用 `build` 方法终止链式调用，以返回驱动程序的实例。

在`After`钩子中，我们使用`quit`方法来释放驱动。这将关闭所有窗口并结束会话。

我们将驱动实例存储在 Cucumber 的世界（上下文）中，以便其他步骤可以使用。

# 实现步骤定义

接下来，我们需要实现步骤定义。

# 导航到页面

现在一切准备就绪，让我们实现我们的第一个步骤，即`When user navigates to /`。导航可以通过在驱动对象上使用`.get`方法来完成：

```js
import { Given, When, Then } from 'cucumber';

When(/^user navigates to ([\w-_\/?=:#]+)$/, function (location) {
  return this.driver.get(`http://${process.env.SERVER_HOST_TEST}:${process.env.SERVER_PORT_TEST}${location}`);
});
```

此步骤从环境变量中获取服务器主机和端口。`this.driver.get`返回一个承诺，该承诺被返回。Cucumber 将在移动到下一个步骤之前等待此承诺解决或拒绝。

# 输入到输入框中

这是我们的下一个步骤：

```js
When user types in "valid@ema.il" in the "#email" element
```

这涉及到找到具有`id`为`email`的元素，然后向其发送按键事件。在`spec/cucumber/steps/interactions/input.js`中添加以下步骤定义：

```js
import { Given, When, Then } from 'cucumber';
import { By } from 'selenium-webdriver';

When(/^user types in (?:"|')(.+)(?:"|') in the (?:"|')([\.#\w]+)(?:"|') element$/, async function (text, selector) {
  this.element = await this.driver.findElement(By.css(selector));
  return this.element.sendKeys(text);
});
```

这里，`driver.findElement`返回一个`WebElementPromise`实例。我们使用`async`/`await`语法来避免回调地狱或深度链式承诺。相同的步骤定义将适用于我们的下一个步骤，即在`#password`输入元素中输入一个简短的密码。

# 断言结果

最后一步是执行以下操作：

```js
Then the "#register-button" element should have a "disabled" attribute
```

如前所述，我们需要找到元素，但这次读取其`disabled`属性并断言它设置为`"true"`。

HTML 的`content`属性始终是一个字符串，即使你期望它是布尔值或数字。

在`spec/cucumber/steps/assertions/index.js`中添加以下内容：

```js
import assert from 'assert';
import { Given, When, Then } from 'cucumber';
import { By } from 'selenium-webdriver';

When(/^the (?:"|')([\.#\w-]+)(?:"|') element should have a (?:"|')([\w_-]+)(?:"|') attribute$/, async function (selector, attributeName) {
  const element = await this.driver.findElement(By.css(selector));
  const attributeValue = await element.getAttribute(attributeName);
  assert.equal(attributeValue, 'true');
});
```

这里，我们使用`getAttribute`方法从`WebElement`实例中获取`disabled`属性值。同样，这是一个异步操作，所以我们使用`async`/`await`语法来保持代码整洁。

如果你有时间，阅读官方文档总是一个好主意。`selenium-webdriver`中所有类和方法的 API 可以在[`seleniumhq.github.io/selenium/docs/api/javascript/module/selenium-webdriver/`](https://seleniumhq.github.io/selenium/docs/api/javascript/module/selenium-webdriver/)找到。

# 运行测试

现在，我们已经准备好运行测试：

```js
$ yarn run test:e2e
```

这将运行`./scripts/e2e.test.sh`脚本，该脚本将使用 Webpack 构建项目（这可能需要一些时间）。然后，一个 Google Chrome 浏览器将弹出，你会看到输入字段被自动填充了我们指定的文本。Selenium 执行完所有必要的操作后，我们 After 钩子中的`driver.quit()`方法调用将关闭浏览器，并将结果显示在我们的终端中：

```js
......

1 scenario (1 passed)
4 steps (4 passed)
0m02.663s
```

# 添加多个测试浏览器

使用 Selenium 的最大好处是你可以使用相同的测试来测试多个浏览器。如果我们只对单个浏览器感兴趣，比如 Chrome，那么使用 Puppeteer 会更好。所以，让我们将 Firefox 添加到我们的测试中。

Firefox，就像 Chrome 一样，需要一个驱动程序才能工作。Firefox 的驱动程序是`geckodriver`，它使用*Marionette*代理向 Firefox 发送指令（Marionette 类似于 Chrome 的 DevTools 协议）：

```js
$ yarn add geckodriver --dev
```

现在，我们只需要将`forBrowser`调用更改为使用`"firefox"`：

```js
this.driver = new webdriver.Builder()
  .forBrowser("firefox")
  .build();
```

当我们再次运行测试时，将使用 Firefox 而不是 Chrome。

然而，而不是在我们的代码中硬编码浏览器，让我们更新我们的脚本来允许我们指定我们想要测试的浏览器。我们可以通过将参数传递到 shell 脚本中来实现这一点。例如，如果我们执行以下操作：

```js
$ yarn run test:e2e -- chrome firefox
```

然后，在我们的`scripts/e2e.test.sh`中，我们可以使用`$1`来访问第一个参数（`chrome`），`$2`来访问`firefox`，依此类推。或者，我们可以使用特殊的参数`"$@"`，它是一个类似于数组的结构，包含所有参数。在`scripts/e2e.test.sh`中，将测试块更改为以下内容：

```js
if $SERVER_UP; then
  for browser in "$@"; do
    export TEST_BROWSER="$browser"
    echo -e "\n---------- $TEST_BROWSER test start ----------"
    npx dotenv cucumberjs spec/cucumber/features -- --compiler js:babel-register --require spec/cucumber/steps
    echo -e "----------- $TEST_BROWSER test end -----------\n"
  done
else
  >&2 echo "Web server failed to start"
fi
```

这将遍历我们的浏览器列表，`export`它到`TEST_BROWSER`变量中，并运行我们的测试。然后，在`spec/cucumber/steps/index.js`中的`forBrowser`调用中，传递来自`process.env`的浏览器名称而不是硬编码它：

```js
this.driver = new webdriver.Builder()
  .forBrowser(process.env.TEST_BROWSER || "chrome")
  .build();
```

现在，尝试使用`$ yarn run test:e2e -- chrome firefox`运行它，你应该看到我们的测试首先在 Chrome 上运行，然后是 Firefox，然后结果整洁地显示在标准输出中：

```js
$ yarn run test:e2e

---------- chrome test start ----------
......

1 scenario (1 passed)
4 steps (4 passed)
0m01.899s
----------- chrome test end -----------

---------- firefox test start ----------
......

1 scenario (1 passed)
4 steps (4 passed)
0m03.258s
----------- firefox test end -----------
```

最后，我们应该定义 NPM 脚本来让其他开发者清楚地知道我们可以运行的操作。通过将其添加为 NPM 脚本，用户只需要查看`package.json`，而不必研究 shell 脚本以了解它是如何工作的。因此，在`package.json`的`scripts`部分，将我们的`test:e2e`更改为以下内容：

```js
"test:e2e": "yarn run test:e2e:all",
"test:e2e:all": "yarn run test:e2e:chrome firefox",
"test:e2e:chrome": "./scripts/e2e.test.sh chrome",
"test:e2e:firefox": "./scripts/e2e.test.sh firefox"
```

我们现在已经成功编写并运行了我们的第一个测试。接下来，让我们通过涵盖所有无效情况来使我们的场景更加通用：

```js
Scenario Outline: Invalid Input

  Tests that the 'Register' button is disabled when either input elements contain invalid values

  When user types in "<email>" in the "#email" element
  And user types in "<password>" in the "#password" element
  Then the "#register-button" element should have a "disabled" attribute

Examples:

| testCase       | email         | password       |
| Both Invalid   | invalid-email | shortpw        |
| Invalid Email  | invalid-email | abcd1234qwerty |
| Short Password | valid@ema.il  | shortpw        |
```

# 运行我们的后端 API

接下来，我们需要处理一个快乐的路径场景，其中用户填写了有效详细信息并点击了注册按钮。

在这里，我们将编写一个测试，说明“当用户提交有效详细信息后，在收到服务器响应后，UI 将显示成功消息”。这个功能尚未实现，这意味着这将是我们前端 TDD 的第一步！

# 使用 Webpack 进行动态字符串替换

在我们可以使用 API 后端进行端到端测试之前，我们必须进行一项小的改进。目前，我们正在硬编码生产 API 端点的 URL（`localhost:8080`），尽管在测试期间将使用测试 URL（`localhost:8888`）。因此，我们需要用一个占位符来替换它，我们可以在构建时覆盖这个占位符。

首先，在`src/components/registration-form/index.jsx`中，替换以下行：

```js
const request = new Request('http://localhost:8080/users/', {})
```

使用这个：

```js
const request = new Request('http://%%API_SERVER_HOST%%:%%API_SERVER_PORT%%/users/', {})
```

我们使用`%%`来标记我们的占位符，因为它是一个相对不常见的字符序列。你可以选择任何你喜欢的占位符语法。

接下来，我们需要添加一个新的加载器来在构建时替换此占位符。`string-replace-loader` 完美地符合要求。让我们安装它：

```js
yarn add string-replace-loader --dev
```

然后，在 `.env` 和 `.env.example` 中添加不同环境的 API 主机和端口的详细信息：

```js
API_SERVER_PORT_TEST=8888
 API_SERVER_HOST_TEST=localhost
 API_SERVER_PORT_PROD=8080
 API_SERVER_HOST_PROD=localhost
```

然后，在 `webpack.config.js` 中使用插件。我们希望加载器转换所有 `.js` 和 `.jsx` 文件，因此我们可以使用与 `babel-loader` 相同的规则：

```js
...

if (process.env.NODE_ENV === 'test') {
  process.env.API_SERVER_HOST = process.env.API_SERVER_HOST_TEST;
  process.env.API_SERVER_PORT = process.env.API_SERVER_PORT_TEST;
} else {
  process.env.API_SERVER_HOST = process.env.API_SERVER_HOST_PROD;
  process.env.API_SERVER_PORT = process.env.API_SERVER_PORT_PROD;
}

module.exports = { 
  entry: { ... },
  output: { ... },
  module: {
    rules: [
      {
        test: /\.jsx?$/,
        exclude: /node_modules/,
        use: [
          {
            loader: "babel-loader",
            options: { ... }
          },
          {
            loader: 'string-replace-loader',
            options: {
              multiple: [
                 { search: '%%API_SERVER_HOST%%', replace: process.env.API_SERVER_HOST, flags: 'g' },
                 { search: '%%API_SERVER_PORT%%', replace: process.env.API_SERVER_PORT, flags: 'g' }
              ]
            }
          }
        ]
      }
    ]
  },
  plugins: [...]
};
```

在顶部，我们正在检查 `NODE_ENV` 环境变量，并使用它来确定 API 正在使用哪个端口。然后，在我们的加载器选项中，我们指示它对字符串进行全局正则表达式搜索，并将其替换为动态生成的宿主和端口。

# 从子模块中提供 API 服务

当我们运行测试时，我们想确保我们的后端 API 正在使用设置为 `test` 的 `NODE_ENV` 环境变量运行。目前，我们正在手动执行此操作。然而，将其作为测试脚本的一部分添加更为理想。就像我们为 Swagger UI 所做的那样，我们可以使用 Git 子模块将 Hobnob API 仓库包含在客户端仓库中，而不重复代码：

```js
git submodule add git@github.com:d4nyll/hobnob.git api
```

现在，为了简化后续操作，将以下 NPM 脚本添加到 `package.json` 中：

```js
"api:init": "git submodule update --init",
"api:install": "yarn install --cwd api",
"api:serve": "yarn --cwd api run build && dotenv -e api/.env.example node api/dist/index.js",
"api:update": "git submodule update --init --remote",
```

`api:init` 将使用存储的提交哈希下载 Hobnob API 仓库。`api:install` 使用 `--cwd` 在运行 `yarn install` 之前更改目录到 `api` 目录。`api:serve` 首先运行我们的 API 仓库中的 `build` 脚本，加载环境变量，然后运行 API 服务器。`api:update` 将下载并更新 API 仓库到同一分支的最新提交。

最后，在 `scripts/e2e.test.sh` 中运行 NPM 脚本：

```js
...
export NODE_ENV=test

yarn run api:init > /dev/null 2>&1 &
yarn run api:install > /dev/null 2>&1 &
yarn run api:serve > /dev/null 2>&1 &

yarn run serve > /dev/null 2>&1 &
...
```

# 定义愉快的场景

让我们通过编写功能文件来开始定义我们的愉快场景：

```js
Scenario: Valid Input

  Tests that the 'Register' button is enabled when valid values are provided, and that upon successful registration, the UI display will display the message "You've been registered successfully"

  When user types in a valid email in the "#email" element
  And user types in a valid password in the "#password" element
  Then the "#register-button" element should not have a "disabled" attribute

  When user clicks on the "#register-button" element
  Then the "#registration-success" element should appear within 2000 milliseconds
```

# 生成随机数据

在这个场景中，我们不能硬编码一个单独的电子邮件来测试，因为这可能导致一个 `409 冲突` 错误，因为已经存在具有该电子邮件的账户。因此，每次运行测试时，我们需要生成一个随机电子邮件。我们需要定义一个新的步骤定义，其中数据每次都是随机生成的：

```js
When(/^user types in an? (in)?valid (\w+) in the (?:"|')([\.#\w-]+)(?:"|') element$/, async function (invalid, type, selector) {
  const textToInput = generateSampleData(type, !invalid);
  this.element = await this.driver.findElement(By.css(selector));
  return this.element.sendKeys(textToInput);
});
```

在这里，我们创建一个通用的步骤定义，并使用尚未定义的 `generateSampleData` 函数来提供随机数据。我们将在 `spec/cucumber/steps/utils/index.js` 的新文件中定义 `generateSampleData` 函数，就像我们在后端测试中所做的那样，使用 `chance` 包来生成随机数据。

首先，安装 `chance` 包：

```js
$ yarn add chance --dev
```

然后按照以下方式定义 `generateSampleData`：

```js
import Chance from 'chance';
const chance = new Chance();

function generateSampleData (type, valid = true) {
  switch (type) {
    case 'email':
      return valid ? chance.email() : chance.string()
      break;
    case 'password':
      return valid ? chance.string({ length: 13 }) : chance.string({ length: 5 });
      break;
    default:
      throw new Error('Unsupported data type')
      break;
  }
}

export {
  generateSampleData,
}
```

# 使步骤定义更通用

此场景检查 `disabled` 属性，就像之前一样，但这次测试它没有被设置。因此，更新我们的步骤定义在 `spec/cucumber/steps/assertions/index.js` 中，以考虑这一点：

```js
When(/^the (?:"|')([\.#\w-]+)(?:"|') element should( not)? have a (?:"|')([\w_-]+)(?:"|') attribute$/, async function (selector, negation, attributeName) {
  const element = await this.driver.findElement(By.css(selector));
  const attributeValue = await element.getAttribute(attributeName);
  const expectedValue = negation ? null : 'true';
  assert.equal(attributeValue, expectedValue);
});
```

# 点击

最后两个步骤是 WebDriver 点击注册按钮并等待服务器响应。对于点击步骤，我们只需要找到 `WebElement` 实例并调用它的 `click` 方法。在 `spec/cucumber/steps/interactions/element.js` 中定义以下步骤定义：

```js
import { Given, When, Then } from 'cucumber';
import { By } from 'selenium-webdriver';

When(/^user clicks on the (?:"|')([\.#\w-]+)(?:"|') element$/, async function (selector) {
  const element = await this.driver.findElement(By.css(selector));
  return element.click();
});
```

# 等待

最后一步需要我们等待 API 服务器对我们的请求做出响应，之后我们应该显示一个成功消息。

一种简单但非常常见的方法是在进行断言之前等待几秒钟。然而，这有两个缺点：

+   如果设置的时间太短，可能会导致测试不稳定，即在某些实例上通过，在其他实例上失败。

+   如果设置的时间太长，它会延长测试持续时间。在实践中，长时间的测试意味着测试运行得较少，对开发者提供反馈的作用也较小。

幸运的是，Selenium 提供了 `driver.wait` 方法，它具有以下签名：

```js
driver.wait(<condition>, <timeout>, <message>)
```

`condition` 可以是一个 `Condition` 实例、一个函数或一个类似承诺的 thenable。`driver.wait` 将重复评估 `condition` 的值，直到它返回一个真值。如果 `condition` 是一个承诺，它将等待承诺解决并检查解决值是否为真。`timeout` 是 `driver.wait` 将尝试的时间（以毫秒为单位）。

在 `spec/cucumber/steps/assertions/index.js` 中，添加以下步骤定义：

```js
import chai, { expect } from 'chai';
import chaiAsPromised from 'chai-as-promised';
import { By, until } from 'selenium-webdriver';

chai.use(chaiAsPromised);

Then(/^the (?:"|')([\.#\w-]+)(?:"|') element should appear within (\d+) milliseconds$/, function (selector, timeout) {
  return expect(this.driver.wait(until.elementLocated(By.css(selector)), timeout)).to.be.fulfilled;
});
```

我们使用 `until.elementLocated` 作为条件，如果元素被定位，它将解析为真值。我们还使用 `chai` 和 `chai-as-promised` 作为我们的断言库（而不是 `assert`）；它们为我们提供了 `expect` 和 `.to.be.fulfilled` 语法，这使得涉及承诺的测试更加易于阅读。

运行测试，最后一步应该失败。这是因为我们还没有实现 `#registration-success` 元素：

```js
---------- firefox test start ----------
........................F.

Failures:

1) Scenario: Valid Input # spec/cucumber/features/users/register/main.feature:24
    Before # spec/cucumber/steps/index.js:5
    When user navigates to / # spec/cucumber/steps/interactions/navigation.js:3
    When user types in a valid email in the "#email" element # spec/cucumber/steps/interactions/input.js:10
    And user types in a valid password in the "#password" element # spec/cucumber/steps/interactions/input.js:10
    Then the "#register-button" element should not have a "disabled" attribute # spec/cucumber/steps/assertions/index.js:9
    When user clicks on the "#register-button" element # spec/cucumber/steps/interactions/element.js:4
    Then the "#registration-success" element should appear within 2000 milliseconds # spec/cucumber/steps/assertions/index.js:16
       AssertionError: expected promise to be fulfilled but it was rejected with 'TimeoutError: Waiting for element to be located By(css selector, #registration-success)\nWait timed out after 2002ms'
    After # spec/cucumber/steps/index.js:12

4 scenarios (1 failed, 3 passed)
18 steps (1 failed, 17 passed)
0m10.403s
----------- firefox test end -----------
```

# 根据状态渲染组件

为了能够在合适的时间显示 `#registration-success` 元素，我们必须将我们请求的结果存储在我们的状态中。目前，在我们的 `RegistrationForm` 组件中，我们只是在控制台记录结果：

```js
fetch(request)
  .then(response => {
    if (response.status === 201) {
      return response.text();
    } else {
      throw new Error('Error creating new user');
    }
  })
  .then(console.log)
  .catch(console.log)
```

相反，当服务器响应新用户的 ID 时，我们将它存储在状态下的 `userId` 属性中：

```js
fetch(request)
  .then(response => { ... })
  .then(userId => this.setState({ userId }))
  .catch(console.error)
```

此外，请确保你在类的构造函数中将 `userId` 的初始状态设置为 `null`：

```js
constructor(props) {
  super(props);
  this.state = {
    userId: null,
    ...
  };
}
```

然后，在我们的 `render` 方法中，检查 `userId` 状态是否为真，如果是，则显示一个 ID 为 `registration-success` 的元素，而不是表单：

```js
render() {
  if(this.state.userId) {
    return <div id="registration-success">You have registered successfully</div>
  }
  ...
}
```

再次运行我们的测试，它们应该再次通过！

# 使用 React Router 进行路由

接下来，我们将开发登录页面。这需要我们为每个页面使用不同的路径。例如，注册页面可以在 `/register` 路径下提供服务，而登录页面在 `/login` 路径下。为此，我们需要一个**路由器**。在服务器上，我们使用 Express 来路由击中我们的 API 的请求；对于前端，我们需要一个客户端路由器来完成同样的工作。在 React 生态系统中，最成熟的路由器是 *React Router*。让我们安装它：

```js
$ yarn add react-router react-router-dom
```

`react-router` 提供核心功能，而 `react-router-dom` 允许我们在网页上使用 React Router。这类似于 React 在网页上被分为 `react` 和 `react-dom`。

# 基础知识

如前所述，React 中的所有内容都是组件。React Router 也不例外。React Router 提供了一套**导航组件**，这些组件将从 URL、视口和设备信息中收集数据，以便显示适当的组件。

React Router 中有三种类型的组件：

+   路由组件

+   路由匹配组件

+   导航组件

# 路由器

路由组件是我们应用的包装器。路由组件负责保持路由的历史记录，以便你可以“返回”到上一个屏幕。有两个路由组件 —— `<BrowserRouter>` 和 `<HashRouter>`。`<HashRouter>` 仅用于服务静态文件；因此，我们将使用 `<BrowserRouter>` 组件。

在 `src/index.jsx` 中，用我们的 `BrowserRouter` 组件包裹我们的根组件（目前是 `<RegistrationForm />`）：

```js
import React from 'react';
import ReactDOM from 'react-dom';
import { BrowserRouter } from 'react-router-dom';
import RegistrationForm from './components/registration-form/index.jsx';

ReactDOM.render((
  <BrowserRouter>
    <RegistrationForm />
  </BrowserRouter>
), document.getElementById('renderTarget'));
```

# 路由匹配

目前，如果你提供应用程序，没有什么变化——我们只是将我们的应用包裹在 `BrowserRouter` 中，这样我们就可以在 `<BrowserRouter>` 内定义**路由匹配**组件。假设我们只想在路由是 `/register` 时渲染 `<RegistrationForm>` 组件，我们可以使用一个 `<Route>` 组件：

```js
...
import { BrowserRouter, Route } from 'react-router-dom';

ReactDOM.render((
  <BrowserRouter>
    <Route exact path="/register" component={RegistrationForm} />
  </BrowserRouter>
), document.getElementById('renderTarget'));
```

`<Route>` 组件通常使用两个属性 —— `path` 和 `component`。如果一个 `<Route>` 组件有一个 `path` 属性与当前 URL 的路径名匹配（例如 `window.location.pathname`），则 `component` 属性中指定的组件将被渲染。

匹配是以**包含**的方式进行的。例如，路径名 `/register/user`、`/register/admin` 和 `register` 都会匹配路径 `/register`。然而，对于我们的用例，我们希望此元素仅在路径完全匹配时显示，因此我们使用了 `exact` 属性。

在进行更改后，让我们再次提供应用程序。

# 支持历史 API

但当我们访问 `http://localhost:8200/register` 时，我们得到一个 `404 Not Found` 响应。从终端中，我们可以看到这是因为请求由 `http-server` 处理，而不是我们的应用程序：

```js
"GET /register" "Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/66.0.3359.117 Safari/537.36"
"GET /register" Error (404): "Not found"
```

这是有意义的，因为 `http-server` 是一个非常简单的 *静态* 服务器，而我们需要在客户端 *动态* 执行路由。因此，我们需要使用支持此功能的服务器。`pushstate-server` 是一个静态服务器，它也支持 HTML5 历史 API。让我们安装它：

```js
$ yarn add pushstate-server --dev
```

现在，在 `scripts/serve.sh` 中，将 `http-server` 行替换为以下内容：

```js
pushstate-server dist/ $WEB_SERVER_PORT_TEST
```

当我们运行 `yarn run serve` 并导航到 `localhost:8200/register` 时，一切如预期工作！

最后，更新我们的 Cucumber 测试功能文件，以便测试导航到正确的页面：

```js
- When user navigates to /
 + When user navigates to /register
```

# 导航

React Router 提供的最后重要的组件类是导航组件，共有三种类型：

+   `<Link>`：这将渲染一个锚点 (`<a>`) 组件，例如，`<Link to='/'>Home</Link>`

+   `<NavLink>`：这是一种特殊的 `<Link>` 类型，如果路径名与 `to` 属性匹配，它将向元素添加一个类，例如，`<NavLink to='/profile' activeClassName='active'>Profile</NavLink>`

+   `<Redirect>`：这是一个将导航到 `to` 属性的组件，例如，`<Redirect to='/login'/>`

因此，我们可以更新我们的 `#registration-success` 元素，以包含指向主页和登录页面的链接（我们尚未实现！）：

```js
import { Link } from 'react-router-dom';

...
class RegistrationForm extends React.Component {
  render() {
    ...
    <div id="registration-success">
      <h1>You have registered successfully!</h1>
      <p>Where do you want to go next?</p>
      <Link to='/'><Button title="Home"></Button></Link>
      <Link to='/login'><Button title="Login"></Button></Link>
    </div>
  }
}
```

# TDD

当我们开发注册页面时，我们在编写测试之前实现了功能。我们这样做是因为我们不知道 E2E 测试在 React 中是如何工作的。现在我们知道了，是时候实施一个合适的 TDD 流程了。

要实现 TDD，我们应该查看 UI 的设计，确定测试需要与之交互的关键元素，并为每个元素分配一个唯一的 `id`。这些 `id` 然后形成我们测试和实现之间的契约。

例如，如果我们使用 TDD 开发了注册页面，我们首先将输入分配给 `#email`、`#password` 和 `#register-button` 这些 ID，并使用这些 ID 编写测试代码来选择元素。然后，当我们实现功能时，我们将确保使用测试中指定的相同 ID。

通过使用 `id` 字段，我们可以更改实现细节，但保持测试不变。想象一下，如果我们使用了不同的选择器，比如说，`form > input[name="email"]`；那么，如果我们向 `<form>` 元素内添加一个内部包装器，我们就必须更新我们的测试。

设计和前端是软件开发阶段中最具波动性的任务之一；编写能够承受这种波动性的测试是明智的。一个项目完全更换框架并不罕见。比如说，在几年后，另一个前端框架出现并彻底改变了前端格局。通过使用 `id` 来选择元素，我们可以在不重写测试的情况下切换到这个新框架。

# 登录

在开发登录页面时，我们将遵循 TDD 流程。

# 编写测试

这意味着从编写 Cucumber 功能文件开始。

```js
Feature: Login User

  User visits the Login Page, fills in the form, and submits

  Background: Navigate to the Login Page

    When user navigates to /login

  Scenario Outline: Invalid Input

    Tests that the 'Login' button is disabled when either input elements contain invalid values

    When user types in "<email>" in the "#email" element
    And user types in "<password>" in the "#password" element
    Then the "#login-button" element should have a "disabled" attribute

  Examples:

  | testCase | email | password |
  | Both Invalid | invalid-email | shortpw |
  | Invalid Email | invalid-email | abcd1234qwerty |
  | Short Password | valid@ema.il | shortpw |

  Scenario: Valid Input

    Tests that the 'Login' button is enabled when valid values are provided, and that upon successful login, the UI display will display the message "You've been logged in successfully"

    When a random user is registered
    And user types in his/her email in the "#email" element
    And user types in his/her password in the "#password" element
    Then the "#login-button" element should not have a "disabled" attribute

    When user clicks on the "#login-button" element
    Then the "#login-success" element should appear within 2000 milliseconds
```

这引入了几个新的步骤。`当随机用户注册` 步骤直接调用 API 来注册用户。我们将使用此用户来测试我们的登录步骤。它实现在一个名为 `spec/cucumber/steps/auth/index.js` 的新模块中：

```js
import chai, { expect } from 'chai';
import chaiAsPromised from 'chai-as-promised';
import { Given, When, Then } from 'cucumber';
import { By, until } from 'selenium-webdriver';
import bcrypt from 'bcryptjs';
import fetch, { Request } from 'node-fetch';
import { generateSampleData } from '../utils';

chai.use(chaiAsPromised);

Then(/^a random user is registered$/, function () {

  this.email = generateSampleData('email');
  this.password = generateSampleData('password');
  this.digest = bcrypt.hashSync(this.password, 10);

  const payload = {
    email: this.email,
    digest: this.digest
  };

  const request = new Request(`http://${process.env.API_SERVER_HOST_TEST}:${process.env.API_SERVER_PORT_TEST}/users/`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    mode: 'cors',
    body: JSON.stringify(payload)
  })
  return fetch(request)
    .then(response => {
      if (response.status === 201) {
        this.userId = response.text();
      } else {
        throw new Error('Error creating new user');
      }
    })
});
```

我们正在使用我们之前定义的 `generateSampleData` 工具函数来生成新用户的详细信息。我们也将这些详细信息存储在上下文中。接下来，我们使用 Fetch API 向 API 发送创建用户请求。然而，Fetch API 是浏览器原生的 API。因此，为了在 Node 中使用 Fetch API，我们必须安装一个 polyfill，`node-fetch`：

```js
$ yarn add node-fetch --dev
```

然后，对于步骤 `用户在 "#email" 元素中输入他的/她的电子邮件` 和 `用户在 "#password" 元素中输入他的/她的密码`，我们正在使用上下文中存储的详细信息来填写登录表单并提交它。如果请求成功，预期将出现一个 ID 为 `login-success` 的元素。

如果你忘记了任何 API 端点和参数，只需参考 Swagger 文档，你可以通过运行 `yarn run docs:serve` 来提供该文档。

# 实现登录

实现登录表单与注册表单类似，但是它涉及两个步骤而不是一个。客户端必须首先从 API 中检索盐，使用它来散列密码，然后向 API 发送第二个请求以登录。你的实现可能看起来像这样：

```js
import React from 'react';
import bcrypt from 'bcryptjs';
import { validator } from '../../utils';
import Button from '../button/index.jsx';
import Input from '../input/index.jsx';

function retrieveSalt (email) {
  const url = new URL('http://%%API_SERVER_HOST%%:%%API_SERVER_PORT%%/salt/');
  url.search = new URLSearchParams({ email });

  const request = new Request(url, {
    method: 'GET',
    mode: 'cors'
  });

  return fetch(request)
    .then(response => {
      if (response.status === 200) {
        return response.text();
      } else {
        throw new Error('Error retrieving salt');
      }
    })
}

function login (email, digest) {

  // Send the credentials to the server
  const payload = { email, digest };
  const request = new Request('http://%%API_SERVER_HOST%%:%%API_SERVER_PORT%%/login/', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    mode: 'cors',
    body: JSON.stringify(payload)
  })
  return fetch(request)
    .then(response => {
      if (response.status === 200) {
        return response.text();
      } else {
        throw new Error('Error logging in');
      }
    })
}

class LoginForm extends React.Component {

  constructor(props) {
    super(props);
    this.state = {
      token: null,
      email: {
        value: "",
        valid: null
      },
      password: {
        value: "",
        valid: null
      }
    };
  }

  handleLogin = (event) => {
    event.preventDefault();
    event.stopPropagation();

    const email = this.state.email.value;
    const password = this.state.password.value;

    retrieveSalt(email)
      .then(salt => bcrypt.hashSync(password, salt))
      .then(digest => login(email, digest))
      .then(token => this.setState({ token }))
      .catch(console.error)
  }

  handleInputChange = (name, event) => {
    const value = event.target.value;
    const valid = validatorname;
    this.setState({
      [name]: { value, valid }
    });
  }

  render() {
    if(this.state.token) {
      return (
        <div id="login-success">
          <h1>You have logged in successfully!</h1>
          <p>Where do you want to go next?</p>
          <Link to='/'><Button title="Home"></Button></Link>
          <Link to='/profile'><Button title="Profile"></Button></Link>
        </div>
      )
    }
    return [
      <form onSubmit={this.handleLogin}>
        <Input label="Email" type="email" name="email" id="email" value={this.state.email.value} valid={this.state.email.valid} onChange={this.handleInputChange} />
        <Input label="Password" type="password" name="password" id="password" value={this.state.password.value} valid={this.state.password.valid} onChange={this.handleInputChange} />
        <Button title="Login" id="login-button" disabled={!(this.state.email.valid && this.state.password.valid)}/>
      </form>,
      <p>Don't have an account? <Link to='/register'>Register</Link></p>
    ]
  }
}

export default LoginForm;
```

现在我们有了表单组件，让我们将其添加到路由器中。在 React Router 版本 v4 之前，你只需将一个新的 `<Route>` 组件添加到 `<BrowserRouter>`：

```js
<BrowserRouter>
  <Route exact path="/register" component={RegistrationForm} />,
  <Route exact path="/login" component={LoginForm} />
</BrowserRouter>
```

然而，随着 React Router v4 的推出，路由组件只能有一个子组件。因此，我们必须将 `<Route>` 组件包裹在一个容器中。

`react-router-dom` 包提供了 `<Switch>` 组件，我们将将其用作容器。`<Switch>` 组件将只渲染第一个匹配的 `<Route>` 中指定的组件：

```js
import React from 'react';
import ReactDOM from 'react-dom';
import { BrowserRouter, Route, Switch } from 'react-router-dom';
import RegistrationForm from './components/registration-form/index.jsx';
import LoginForm from './components/login-form/index.jsx';

ReactDOM.render((
  <BrowserRouter>
    <Switch>
      <Route exact path="/register" component={RegistrationForm} />,
      <Route exact path="/login" component={LoginForm} />
    </Switch>
  </BrowserRouter>
), document.getElementById('renderTarget'));
```

在前面的例子中，如果我们导航到 `/register`，`<Switch>` 组件将看到第一个 `<Route>` 组件中有一个匹配项，并将停止寻找更多匹配项并返回 `<RegistrationForm>`。

# 现在轮到你了

我们已经在之前的章节中介绍了如何编写端到端测试，并演示了如何为注册和登录页面应用 TDD。

现在，我们将接力棒传给你，以便你可以改进我们所做的，使其符合设计，并以 TDD 的方式完成应用程序的其余部分。

你不需要专注于使事物看起来很漂亮——这并不是这里的重点。只需确保所有组件都在那里，并且用户流程是正确的。

在完成这些之后，查看我们的实现并使用它来改进你的实现。然后，我们将查看单元测试以及其他可以应用于前端代码的测试类型。

# 摘要

在本章中，我们将为后端 API 所做的工作应用到前端代码中。我们使用了 Cucumber、Gherkin 和 Selenium 来编写直接在真实浏览器上运行的 UI 测试。我们还使用了 React Router 实现了客户端路由。

在下一章中，我们将通过学习**Redux**，一个强大的状态管理库，来结束我们对前端世界的探索之旅。
