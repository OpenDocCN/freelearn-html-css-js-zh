# 第三章：使用 Cypress 命令行工具

在上一章中，我们了解了 Cypress 与 Selenium 等其他测试自动化工具的不同之处，以及在 Web 自动化测试方面的突出表现。在本章中，我们将继续使用 Cypress 命令行工具来构建我们对 Cypress 的使用知识。为此，我们将介绍您可以使用的命令，以利用 Cypress 的功能。

其中一些命令将涉及功能，如运行单个或所有测试，调试 Cypress，在不同浏览器上启动 Cypress 测试，以及其他 Cypress 命令行功能。我们将参考本章的 GitHub 存储库文件夹，并将为您的参考和练习包括在存储库中编写的每个命令和代码。

在本章中，我们将涵盖以下关键主题：

+   运行 Cypress 命令

+   理解基本的 Cypress 命令

+   Cypress 在命令行上调试

一旦您完成了每个主题，您将准备好编写您的第一个测试。

## 技术要求

本章的 GitHub 存储库可以在[`github.com/PacktPublishing/End-to-End-Web-Testing-with-Cypress`](https://github.com/PacktPublishing/End-to-End-Web-Testing-with-Cypress)找到。

本章的源代码可以在`chapter-03`目录中找到。

要运行本章中的示例，您需要克隆本书的 GitHub 存储库，并按照`READMe.md`文件中的说明正确设置和运行测试。您可以在`docs.github.com/en/free-pro-team@latest/github/creating-cloning-and-archiving-repositories/cloning-a-repository`上阅读有关如何使用 GitHub 在本地机器上克隆项目的更多信息。

# 运行 Cypress 命令

有效利用 Cypress 框架需要您了解 Cypress 以及如何使用命令行运行不同功能。Cypress 命令允许 Cypress 框架的用户自动化流程，并在初始化和运行时向框架和测试提供特定指令。

在大多数情况下，通过命令行运行 Cypress 测试比使用浏览器运行测试更快。这是因为通过命令行运行测试可以减少运行特定测试所需的资源数量。原因是在命令行中运行的测试通常是无头的，这意味着分配给运行测试的资源较少，而在有头模式下执行测试则不同。

重要提示

有头模式是指测试可以在浏览器上可视化运行，而无头模式是指测试执行过程不会打开可见的浏览器。相反，所有测试都在命令行上运行并输出。

首先，让我们看看如何运行全局和本地的 Cypress 命令。

## 全局和本地命令

Cypress 命令可以从包含 Cypress 安装和代码的特定目录中运行，也可以从全局 Cypress 安装中运行。全局安装 Cypress 可确保用户可以从操作系统中的任何目录运行 Cypress，而使用本地 Cypress 安装，Cypress 只能从安装的单个目录中访问。

### 运行全局 Cypress 命令

在 Cypress 中，全局命令是通过访问全局安装的 Cypress 版本来运行的。运行全局版本的 Cypress 时调用的命令不一定是用户生成或定义的，因为它们内置在框架中。要能够全局运行 Cypress 命令，您需要使用以下命令全局安装 Cypress：

```js
npm install cypress --global
or (shorter version)
npm i -g cypress
```

上述命令将全局安装 Cypress，并确保从任何 Cypress 安装目录调用任何已知的 Cypress 命令都会产生结果或错误，具体取决于提供的命令的执行情况。

要运行全局命令，您需要使用`cypress`关键字定义命令，然后是命令；例如，`cypress run`或`cypress open`。

### 本地运行 Cypress 命令

本地 Cypress 命令源自 Cypress 全局命令，并且是运行命令的另一种选择。要在本地运行 Cypress 命令，您需要使用以下命令在您的目录中安装 Cypress：

```js
npm install cypress 
or (shorter version)
npm i cypress
```

我们可以通过在`package.json`文件的`scripts`部分中定义所需的命令来将其集成到开发环境中，如下所示：

```js
{
  "scripts": {
    "cypress:run": "cypress run",
    "cypress:open": "cypress open"
  }
}
```

将命令添加到`package.json`中允许我们以与执行 JavaScript 包的 npm 命令相同的方式使用这些命令。`package.json`文件中定义的命令在运行时由 Node.js 环境解释，并且在执行时，它们会被执行为全局命令。

重要提示

建议在运行`npm install cypress`命令之前在终端中运行`npm init`命令。如果在未初始化项目的情况下运行 Cypress，Cypress 的目录将不可见。通过运行`init`命令，Cypress 将识别项目目录为现有项目，因此它会初始化并创建其目录，而无需我们在终端上运行额外的命令。

在`package.json`中定义命令不仅使开发人员和质量保证工程师更容易知道要运行哪些命令，而且简化了运行、调试或维护测试时需要运行的命令的性质。

重要提示

Cypress 开发团队建议按项目安装 Cypress，而不是使用全局安装方法。本地安装提供了某些优势，例如用户能够快速更新 Cypress 依赖项，并减少循环依赖问题，这些问题会在不同项目中破坏一些测试，而 Cypress 在另一个项目中运行良好。

要在命令行中运行脚本，您需要调用`npm run`，然后是命令的名称。在我们之前定义的命令中，您只需要运行以下命令来同时执行这些命令：

```js
npm run cypress:run  // command  to run tests on terminal
npm run cypress:open // command to run tests on cypress runner
```

是时候快速回顾一下了。

## 回顾-运行 Cypress 命令

在本节中，我们学习了如何调用本地或全局命令，以及如何从 Cypress 终端或测试运行程序中运行测试，后者利用了图形用户界面。在接下来的部分中，我们将在已经获得的运行 Cypress 命令的知识基础上，了解如何在 Cypress 中使用不同的命令。

# 了解基本的 Cypress 命令

在本节中，我们将探讨各种 Cypress 命令，我们可以使用这些命令通过终端或使用 Cypress 测试运行程序来运行我们的测试。我们还将观察这些命令如何用于实现不同的结果。本节还将介绍如何定制与我们的应用程序交互的不同测试，以实现特定的结果。我们将深入了解最常见的 Cypress 命令以及如何使用 Cypress 框架中预先构建的选项来扩展这些命令。我们将探讨的命令如下：

+   `cypress run`

+   `cypress open`

+   `cypress info`

+   `cypress version`

让我们从`cypress run`开始。

## cypress run

`cypress run`命令以无头方式执行 Cypress 套件中的所有测试，并默认在 Electron 浏览器中运行测试。如果没有使用其他配置扩展，该命令将运行 Cypress 的`integration`文件夹中所有`.spec.js`格式的文件。`Cypress run`命令可以使用以下配置选项运行：

```js
cypress run {configuration-options}
```

最常见的 Cypress 配置选项包括以下内容：

![](img/Table_3.1.jpg)

接下来的几节将扩展前面表格中显示的每个配置选项。

### cypress run --env <env-variable>

Cypress 环境变量是*动态名称-值对*，影响 Cypress 执行测试的方式。当需要在多个环境中运行测试，或者定义的值容易快速更改时，这些环境变量非常有用。

在 Cypress 中，您可以将单个或多个环境变量定义为字符串或 JSON 对象。

在本节中，我们将为一个开源的**todoMVC**应用程序编写测试。这些测试的代码基础可以在本书的 GitHub 存储库中的`chapter 03`目录中找到。被测试的应用程序是一个使用 React 开发的**待办事项列表**应用程序。使用该应用程序，我们可以添加待办事项，标记为已完成，删除它们，查看已完成的项目，甚至在*活动*、*全部*和*已完成*待办事项之间切换。

使用该应用程序，我们可能计划扩展应用程序以使用**HTTPS**而不是当前的**HTTP**协议。即使目前不支持 HTTPS 功能，我们可以在 Cypress 测试中添加对其的支持。为此，我们将把**传输协议**URL 部分定义为环境变量，然后将其传递给`package.json`中的命令，如下例所示。

以下代码片段可以在`chapter 03`的子文件夹中提到的 GitHub 存储库中找到。`Todo-app.spec.js`文件的完整源代码位于`Cypress/integration/examples`文件夹下。我们将在本章中探讨的 Cypress 测试是版本 1 的测试。

下面的`Todo-app.spec.js`文件演示了在导航到 URL 时如何使用环境变量。它是主要的测试文件，位于本书的 GitHub 存储库中的`chapter-03/cypress/integration/examples/todo-app.spec.js`中。

```js
... 
context('TODO MVC Application Tests', () => {
  beforeEach(() => {
    cy.visit(
      `${Cypress.env('TransferProtocol')}://todomvc.com/examples/react/#/`)
  });
...
```

下面的`package.json`文件也位于`chapter-03/`目录中，包含了用于执行 JavaScript 应用程序的测试或执行命令的所有命令。它位于`chapter-03/`目录的根位置：

```js
...
"scripts": {
    "cypress:run": "cypress run --env 
    TransferProtocol='http'",
    "cypress:run:v2": "cypress run --env 
    TransferProtocol='https'",
  },
...
```

上述脚本表明，如果 URL 协议发生变化，我们可以运行前述任何测试命令来替换我们在运行 Cypress 测试时声明的环境变量。我们可以依次执行前述脚本，使用`npm run cypress:run`和`npm run cypress:run:v2`。

重要提示

HTTPS 与 HTTP 相同，不同之处在于 HTTPS 更安全。这是因为发送请求和接收响应的过程使用 TLS(SSL)安全协议进行加密。

传输协议是 URL 的一部分，它确定 URL 是否使用 HTTP 或 HTTPS 协议。使用 HTTP 协议的 URL 以`http://`开头，而使用 HTTPS 的 URL 以`https://`开头。

### cypress run --browser <browser-name>

Cypress 命令行具有内置功能，可以在主机计算机上安装的不同浏览器中运行 Cypress 测试，并且这些浏览器受 Cypress 框架支持。Cypress 会尝试自动检测已安装的浏览器，并可以在`chrome`、`chromium`、`edge`、`firefox`或`electron`中运行测试。要在特定浏览器中运行测试，您需要使用`--browser`配置选项提供浏览器名称。您还可以选择提供浏览器路径而不是浏览器名称；只要它是有效的并且受 Cypress 支持，Cypress 仍然会在提供的浏览器路径中运行测试。

下面的代码片段显示了在本书 GitHub 存储库的`chapter-03`目录中的`package.json`的`scripts`部分中定义的脚本。这些脚本定义了我们的测试将在其中运行的浏览器，并且还传递了 URL 的一部分作为环境变量：

```js
...
"scripts": {
    "cypress:chrome": "cypress run --env 
    TransferProtocol='http' --browser chrome",
    "cypress:firefox": " cypress run --env 
    TransferProtocol='http' --browser firefox"
  },
...
```

在上述命令中，我们可以使用`npm run cypress:chrome`和`npm run cypress:firefox`命令在 Chrome 或 Firefox 中运行测试。

重要提示

要在特定浏览器中运行测试，该浏览器必须安装在您的计算机上，并且还必须是 Cypress 支持的浏览器列表中的一员。

### Cypress run --config <configuration(s)-option>

Cypress 可以使用在终端上运行的命令设置和覆盖配置。Cypress 的配置可以作为单个值、以逗号分隔的多个值，或作为字符串化的 JSON 对象传递。Cypress 中的任何定义的配置都可以通过`cypress run --config`配置选项进行更改或修改。配置选项可能包括指定替代的`viewportHeight`和`ViewportWidth`、超时和文件更改等其他配置。在我们的脚本中，我们将更改 Cypress 运行测试的视口，而不是默认的视口，即 1000x660，我们将在平板视口的 763x700 上运行测试。

下面的代码片段在我们的`chapter-03`根目录的`package.json`文件中定义。以下脚本用于在平板视图中运行测试。为此，您必须覆盖 Cypress 默认配置的视口高度和宽度：

```js
...
"scripts": {
"cypress:tablet-view": "cypress run --env TransferProtocol='http' --config viewportHeight=763,viewportWidth=700",
}
...
```

前面的脚本可以使用`npm run cypress:tablet-view`命令运行。

重要提示

在 Cypress 中传递多个配置选项时，不要在不同配置的逗号分隔值之间留下空格（如上面的代码所示）；否则，Cypress 会抛出错误。

### cypress run --config-file <configuration-file>

Cypress 可以覆盖位于`/cypressRootDirectory/cypress.json`的默认配置文件。您可以定义一个或多个次要的 Cypress 配置文件以运行它们的测试。Cypress 还允许您完全禁用使用配置文件。

下面的脚本位于本书 GitHub 存储库的`chapter-03`目录中的`package.json`中，这是一个命令，使 Cypress 能够覆盖用于运行测试的配置文件。执行该命令时，将使用位于`chapter-03/config/cypress-config.json`下的`cypress-config.json`文件，而不是使用位于`chapter-03`中的默认`cypress.json`文件：

```js
...
"scripts": {
"cypress:run:secondary-configuraton": "cypress run --env TransferProtocol='http' --browser chrome --config-file config/cypress-config.json"
},...
```

运行上述脚本，您需要运行`npm run cypress:run:secondary-configuraton`命令，该命令将使用位于`/cypressRootDirectory/config/cypress-config.json`的配置文件运行测试。

### cypress run --headed

Cypress 提供了一个命令，允许您以无头和有头模式运行浏览器。当定义有头模式时，测试在运行时会打开浏览器。此选项可以在 Cypress 捆绑的默认 Electron 浏览器中使用。在 Electron 中使用`run`命令运行 Cypress 测试的默认模式是无头模式，要覆盖这一点，我们需要在运行测试时传递`--headed`配置。

下面的脚本可以在本书 GitHub 存储库的`chapter-03`目录中的`package.json`文件中找到。运行以下脚本命令将使 Cypress 以有头模式运行，允许在浏览器上看到运行的测试：

```js
...
"scripts": {
"cypress:electron:headed": "cypress run --env TransferProtocol='http' --headed"
},
...
```

前面的脚本可以使用`npm run cypress:electron:headed`命令运行。

### cypress run --headless

Cypress 在 Chrome 和 Firefox 浏览器中以有头模式运行测试，并且每次运行测试时都会启动一个浏览器。要更改此行为并确保测试在不启动浏览器的情况下运行，您需要配置运行 Chrome 或 Firefox 浏览器的命令，以便它们以无头模式运行。

以下脚本位于本书 GitHub 仓库的 `chapter-03` 目录中的 `package.json` 文件中。运行以下命令将使 Cypress 以无头模式运行，测试命令只能在命令行界面上看到：

```js
...
"scripts": {
"cypress:chrome:headless": "cypress run --env TransferProtocol='http' --browser chrome --headless",
"cypress:firefox:headless": "cypress run --env TransferProtocol='http' --browser firefox --headless"
},
...
```

使用上述命令以无头模式运行 Chrome，您需要运行 `npm run cypress:chrome:headless`。要在 Firefox 中以无头模式运行命令，您需要运行 `npm run cypress:firefox:headless` 命令。

### cypress run --spec <spec-file>

Cypress 允许我们指定可以运行的不同测试文件。使用此命令，可以指定在目录中运行 *单个* 测试文件，而不是在目录中运行 *所有* 测试文件。还可以指定不同目录中的不同测试，以便它们同时运行，并指定与特定目录匹配的正则表达式模式。

以下代码片段是 `package.json` 文件的一部分，位于本书 GitHub 仓库的 `chapter-03` 目录中。第一个脚本只能运行目录中的特定文件，而第二个脚本可以运行单个目录中的多个文件：

```js
... 
"scripts": {
  "cypress:integration-v2:todo-app": "cypress run --env 
  TransferProtocol='http' --spec 'cypress/integration/
  integration-v2/todo-app.spec.js'",
  "cypress:integration-v2": "cypress run --env 
  TransferProtocol='http' --spec 'cypress/
  integration/integration-v2/**/'"
},
...
```

第一个命令指定将运行位于 `integration-v2` 文件夹中的 `todo-app.spec.js` 文件的测试。第二个命令将运行位于 `integration-v2` 文件夹中的所有测试文件。

## cypress open

`cypress open` 命令在测试运行器中运行 Cypress 测试，并将配置选项应用于您正在运行的项目的测试。当运行 `cypress open` 命令时传递的配置选项也会覆盖 `cypress.json` 文件中指定的默认值，该文件位于 `tests root` 文件夹中，如果在运行测试时指定了配置。以下命令显示如何运行任何 `cypress open` 命令：

```js
cypress open {configuration-options}
```

命令的第一部分显示了 `cypress open` 命令，而第二部分显示了可以与其链接的配置选项。

最常见的 Cypress 配置选项包括以下内容：

![](img/Table_3.2.jpg)

我们将在接下来的几节中详细介绍每个选项。

### cypress open --env <env-variable(s)>

就像运行 `cypress run` 命令一样，`cypress open` 命令可以在运行测试时使用指定的环境变量运行。与 `cypress run` 命令类似，可以使用 `--env` 配置选项声明一个或多个环境变量来运行测试运行器中的测试。

在前一节中，我们指定了如何通过在 `cypress run` 命令中传递环境变量来通过命令行运行测试。我们将传递相同的环境变量来使用我们的 Cypress 测试运行器运行测试，并且测试应该正常运行。传递的环境变量将确定 **todoMVC** 应用程序 URL 的传输协议是 **HTTP** 还是安全的 **HTTPS**。

以下代码片段位于 `Todo-app.spec.js` 文件中，该文件是我们 `chapter-03/` 目录中的主要测试文件。`todo-app.spec.js` 文件位于 `chapter-03/` 目录中的 `integration/examples` 下。在以下代码片段中，就像在 `cypress run` 中一样，我们可以使用 `cypress open` 命令将环境变量传递给 URL：

```js
... 
context('TODO MVC Application Tests', () => {
  beforeEach(() => {
    cy.visit(
      `${Cypress.env('TransferProtocol')}://todomvc.com/examples/react/#/`)
  });
...
```

以下代码片段位于本书 GitHub 仓库的 `chapter-03/` 根目录中的 `package.json` 文件中。使用此片段，我们将 `'http'` 环境变量传递给我们的测试。这是我们可以完成我们的 URL 并执行我们的测试的时候：

```js
...
"scripts": {
    "cypress:open": "cypress open --env 
    TransferProtocol='http'"
  },
...
```

要打开测试运行器并验证测试运行，您可以运行`npm run cypress:open`，这应该会自动将**TransferProtocol**环境变量添加到运行测试的配置中。

### cypress open --browser </path/to/browser>

当指定时，`--browser`选项指向一个自定义浏览器，该浏览器将被添加到测试运行器中的可用浏览器列表中。要添加的浏览器必须得到 Cypress 的支持，并且必须安装在运行 Cypress 测试的计算机上。

默认情况下，在选择要运行的规范之前，可以通过单击测试运行器中的浏览器选择按钮来查看 Cypress 中的所有可用浏览器。浏览器选择下拉菜单包含已在系统上安装并得到 Cypress 支持的所有浏览器。浏览器选择下拉菜单还允许您切换测试浏览器，从而在不同的浏览器下测试功能：

![图 3.1 - 测试浏览器选择下拉菜单](img/Chapter_3_Image01.jpg)

图 3.1 - 测试浏览器选择下拉菜单

要指定路径以便添加浏览器（例如 Chromium），您需要具有以下配置以将 Chromium 添加到可用浏览器列表中。在这里，您需要运行`npm run cypress:chromium`命令。

该书的 GitHub 存储库中的`chapter-03/`目录下的`package.json`文件中包含以下脚本。当执行该脚本作为命令时，它将查找指定位置的浏览器，并将其添加到用于运行 Cypress 测试的浏览器列表中。

```js
...
"scripts": {
    "cypress:chromium": "cypress open --browser 
    /usr/bin/chromium"
  },
..
```

要执行上述脚本来运行我们的测试，我们需要在终端中运行`npm run cypress:chromium`命令。这将在`/usr/bin/chromium`位置找到 Chromium 浏览器并用它来运行我们的测试。

### cypress open --config <configuration-option(s)>

Cypress 框架允许我们在测试运行器中运行测试并提供必须在初始化测试运行器时传递的配置选项。在传递`--config`选项时，可以传递一个环境变量或用逗号分隔的多个环境变量。以下脚本指定了视口尺寸应为平板电脑，并通过`--config`选项传递配置。要运行所需的命令，您需要运行`npm run cypress:open:tablet-view`。

位于该书的`chapter-03/`根目录下的`package.json`文件中的以下脚本用于更改可见浏览器上运行的测试的视口配置。

```js
...
"scripts": {
    "cypress:open:tablet-view":"cypress open --env 
    TransferProtocol='http' --config 
    viewportHeight=763,viewportWidth=700"
  },
...
```

执行时，该命令会修改浏览器尺寸的默认 Cypress 配置。提供的视口高度和视口宽度将以类似于平板显示屏的方式显示内容。

重要提示

使用`--config`选项指定的配置选项将覆盖`cypress.json`文件中指定的默认配置。

### cypress open --config-file <configuration-file>

就像在`Cypress run`命令的情况下一样，通过测试运行器运行的 Cypress 测试可以具有覆盖默认`cypress.json`文件的覆盖配置文件，该文件包含默认的 Cypress 配置。它位于测试文件的根文件夹中。

位于`chapter-03/`目录的根文件夹中的`package.json`文件中的以下代码片段覆盖了默认的 Cypress 配置文件，该文件被标识为`cypress.json`。当执行时，该命令将读取一个已在`chapter-03/config/cpress-config.json`中声明的替代配置文件：

```js
..."scripts": {
"cypress:open:secondary-configuraton": "cypress open --env TransferProtocol='http' --config-file config/cypress-config.json"
},...
```

要执行上述命令并更改默认的 Cypress 配置文件位置，您需要在命令行界面中运行以下命令：

```js
npm run cypress:open:secondary-configuration
```

现在，让我们看另一个命令。

### cypress open --global

正如我们之前提到的，Cypress 可以全局安装。您可以使用全局 Cypress 安装来运行不同的 Cypress 测试，而不是在每个项目中安装它。这种全局安装还允许您触发全局命令，而不必在调用 Cypress 命令的特定目录中安装 Cypress。要以全局模式打开 Cypress，您需要传递`--global`选项，如下所示：

```js
cypress open --global 
```

通过运行此命令，Cypress 将识别我们要使用全局版本的 Cypress 执行测试，而不是我们的本地实例。

### cypress open --project <project-path>

Cypress 具有内置功能，可以覆盖 Cypress 运行测试时的默认路径。当定义了`--project`选项时，它指示 Cypress 放弃默认的目录/项目路径，而是使用提供的项目路径来运行位于指定项目路径中的 Cypress 测试。在这种设置中，可以在不同的目录或嵌套目录中运行完全不同的 Cypress 项目。

本书`chapter-03/`根目录中的`package.json`文件中的以下代码片段执行了完全不同的 Cypress 项目中的测试。该脚本执行了位于`chapter-03/cypress/todo-app-v3`中的项目：

```js
"scripts": {
"cypress:project:v3": "cypress open --env TransferProtocol='http' --project 'cypress/todo-app-v3/'"
},
```

在上一个脚本中，用户可以运行位于`cypress/todo-app-v3`文件夹中的不同 Cypress 项目。要运行该脚本，我们需要运行`npm run:cypress:project:v3`命令。`version-3`项目是一个独立的项目，不依赖于父 Cypress 项目。它可以使用自己的`cypress.json`文件来确定运行配置，如下面的截图所示：

![图 3.2 - todo-app-v3 项目测试文件夹](img/第三章 _ 图像 02.jpg)

图 3.2 - todo-app-v3 项目测试文件夹

如前面的截图所示，我们已经修改了`todo-app-v3`项目中的`integrationFolder`属性，将主测试文件夹中的`cypress/integration`设置为`cypress/tests`。

### cypress open --port <port-number>

默认情况下，Cypress 在端口`8080`上运行。在传递`--port`选项的同时运行`cypress run`命令，可以覆盖测试运行的默认端口，将其更改为您选择的特定端口。

以下代码片段是本书 GitHub 存储库中`chapter-03/`目录中的`package.json`文件的一部分。运行以下命令会更改 Cypress 运行的默认端口：

```js
"scripts": {"cypress:open:changed-port": "cypress open --env TransferProtocol='http' --port 3004"
  },
```

要运行前面的 Cypress 脚本，您需要运行`npm run cypress:open:changed-port`命令。运行此命令将确保测试在端口`3004`上运行，而不是 Cypress 默认在测试运行器上运行的端口：

![图 3.3 - 覆盖默认的 Cypress 测试端口](img/第三章 _ 图像 03.jpg)

图 3.3 - 覆盖默认的 Cypress 测试端口

前面的截图显示了如何在使用`--port`选项覆盖后，在端口`3004`上运行测试，该选项被传递给`cypress run`命令。这里使用的端口仅用于演示目的；用户的机器上可以传递任何可用端口作为 Cypress 应用程序的覆盖端口。

## 使用 cypress info 命令

在终端上运行`cypress info`命令将在终端上打印 Cypress 安装信息和环境配置。该命令打印的信息包括以下内容：

+   已在计算机上安装并被 Cypress 检测到的浏览器。

+   有关主机操作系统的信息。

+   Cypress 二进制缓存的位置。

+   运行时数据存储的位置。

+   已添加前缀**CYPRESS**的环境变量，用于控制配置，如系统代理。

## 使用 cypress 版本命令

`cypress version`命令打印出 Cypress 的二进制版本和已安装的 Cypress 模块的 npm 版本。大多数情况下，版本应该是相同的，但当安装的模块（如二进制版本）无法作为 npm 包模块安装时，版本可能会有所不同。`cypress version`命令的输出如下截图所示：

![图 3.4 - cypress 版本命令的输出](img/Chapter_3_Image04.jpg)

图 3.4 - cypress 版本命令的输出

前面的截图显示了我机器上安装的 Cypress 包和二进制版本。Cypress 的包版本和二进制版本都是相同的版本。

## Cypress 命令使用的可选练习

在我们项目规范中定义的**todoMVC**项目中，创建一个脚本，可以运行以下测试场景：

+   使用 edge 浏览器进行无头测试。

+   在`chapter 03`根文件夹中的`cypress.json`中指定`TransferProtocol`环境变量的测试运行器上的测试。

通过这个练习，您将了解如何运行有头和无头测试，以及如何向脚本添加环境变量，以便我们可以使用不同的 Cypress 命令来执行。您还将学习如何使用不同的浏览器来执行 Cypress 测试。

## 总结 - 了解基本的 Cypress 命令

在本节中，我们学习了如何使用不同的 Cypress 命令来使用命令行或 Cypress 测试运行器运行 Cypress，后者使用已安装在系统上的不同浏览器运行。我们了解到，尽管 Cypress 带有默认命令来运行测试，但我们可以扩展这些命令，并通过利用可用的命令和选项来定制 Cypress 以增加运行测试的效率。我们还提供了一个练习，让您应用您对`cypress run`和`cypress open`命令的使用知识。在下一节中，您将学习如何使用内置的 Cypress 调试器来查看重要的调试信息，这对于使用我们的终端进行故障排除非常重要。

# 在命令行上进行 Cypress 调试

在本节中，我们将探讨如何使用 Cypress 的命令行调试属性来解决运行测试时可能遇到的问题。我们还将探讨 Cypress 通过命令行提供的不同调试选项。

Cypress 具有内置的调试模块，可以通过在运行测试之前使用`cypress run`或`cypress open`传递调试命令来向用户公开。要从终端接收调试输出，需要在 Mac 或 Linux 环境中设置`DEBUG`环境变量，然后再运行 Cypress 测试。

以下脚本可以在`chapter-03/`根目录的`package.json`文件中找到，并用于在执行命令时显示调试输出。第一个脚本可用于在使用`cypress open`命令运行测试时显示调试输出，而第二个脚本可用于在使用`cypress run`命令运行测试时显示调试输出：

```js
"scripts": {
"cypress:open:debugger": "DEBUG=cypress:* cypress open --env TransferProtocol='http'",
    "cypress:run:debugger": "DEBUG=cypress:* cypress run --
    env TransferProtocol='http'"
  },
} 
```

如前述命令所示，运行`npm run cypress:open:debugger`将在终端中运行 Cypress 测试，并记录运行时的调试输出。第二个命令可以通过`npm run cypress:run:debugger`运行，将在 Cypress 测试运行器上运行测试时运行调试器。

Cypress 使得过滤调试输出变得容易，因为您可以选择有关特定模块的调试信息，例如 Cypress 服务器、CLI 或启动器模块。

以下脚本位于本书 GitHub 存储库的`chapter-03/`目录中的`package.json`文件中。运行时，它将为 Cypress 服务器模块下的所有日志提供调试输出：

```js
...
"scripts": {
"cypress:open:server-debugger": "DEBUG=cypress:server:* cypress open --env TransferProtocol='http'"
} 
...
```

使用`npm run cypress:run:server-debugger`运行上述命令将只输出与 Cypress 服务器相关的调试器信息。使用过滤命令不仅可以轻松缩小 Cypress 中的问题范围，还有助于过滤噪音，留下对于调试 Cypress 信息重要的日志，并将我们带到问题的源头。

## Cypress 调试的可选练习

使用我们项目规范中定义的**todoMVC**项目，在脚本中创建将运行以下测试场景的脚本：

+   调试 Cypress CLI 模块

+   调试`cypress:server`项目模块

通过本次练习，您将掌握 Cypress 调试的概念，并了解如何在`package.json`文件中创建和运行 Cypress 脚本。

## 回顾-在命令行上调试 Cypress

在本节中，我们学习了如何利用 Cypress 通过设置`DEBUG`环境变量查看有关测试运行的其他信息。我们还学习了如何利用 Cypress 的`debug`变量来过滤我们需要的调试输出，并进行了一项练习，以扩展我们在命令行上调试的知识。

# 总结

在本章中，我们学习了`cypress open`和`cypress run`命令，以及如何使用配置选项将这两个命令链接起来以扩展它们的用途。我们还学习了如何检查已安装在系统上的 Cypress 信息和 Cypress 版本。在最后一节中，我们学习了如何使用 Cypress 提供调试输出，并找出测试失败的原因。在下一章中，我们将深入了解编写 Cypress 测试和理解测试的不同部分。
