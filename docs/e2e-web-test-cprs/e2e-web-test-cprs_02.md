# 第一章：安装和设置 Cypress

Cypress 是一个为现代 Web 应用程序构建和设计的端到端测试自动化框架。它专注于通过确保您可以在浏览器上编写、调试和运行测试而无需额外的配置或额外的包来消除测试中的不一致性。Cypress 作为一个独立的应用程序工作，并且可以在 macOS、Unix/Linux 和 Windows 操作系统上使用连字符应用程序或命令行工具进行安装。Cypress 主要是为使用 JavaScript 编写他们的应用程序的开发人员构建的，因为它可以用来测试在浏览器上运行的所有应用程序。在本章中，我们将涵盖以下主题：

+   在 Windows 上安装 Cypress

+   在 macOS 上安装 Cypress

+   通过直接下载安装 Cypress

+   打开 Cypress 测试运行器

+   切换 Cypress 浏览器

+   添加 npm 脚本

+   运行 Cypress 测试

通过本章结束时，您将了解如何在 Windows 和 Mac 操作系统上正确设置 Cypress 以及如何运行 Cypress 测试。您还将了解如何使用 npm 脚本来自动化运行测试和打开测试运行器的过程。

## 技术要求

Cypress 可以作为一个独立的应用程序安装在您的计算机上，并且可以在至少有 2GB RAM 并满足以下任一操作系统要求的机器上运行：

+   macOS 10.9 及以上版本（仅 64 位）

+   Linux Ubuntu 12.04 及以上版本，Fedora 21 和 Debian 8（仅 64 位）

+   Windows 7 及以上

为了在这里列出的操作系统之一上使用 Cypress，必须首先安装 Node.js 8 或更高版本。Node.js 是一个 JavaScript 运行时环境，允许在浏览器之外运行 JavaScript 代码。安装 Node.js 会安装 npm，它允许我们从[`www.npmjs.com/`](https://www.npmjs.com/)安装 JavaScript 包。npm 是 Node.js 的默认包管理器，用户可以使用它或使用第三方包管理器，如 Yarn。在本节中，我们将在 macOS 和 Windows 操作系统上安装 Cypress。

# 在 Windows 上安装 Cypress

在本节中，我们将在 Windows 操作系统上安装 Cypress 和 Node.js，以便我们可以运行我们的测试。

## 下载并安装 Node.js

以下步骤将指导您完成安装 Node.js：

1.  访问官方 Node.js 网站（[`nodejs.org/en/download/`](https://nodejs.org/en/download/)）。

1.  选择 Windows 安装程序选项。

1.  下载安装程序包。

1.  按照 Node.js 网站上的说明安装 Node.js 包。

接下来，让我们初始化项目。

## 初始化项目

作为最佳实践，Cypress 安装在项目所在的目录中；这样，我们可以确保 Cypress 测试属于项目。在我们的情况下，我们将在`Documents`内创建一个名为`cypress-tests`的文件夹，然后在安装 Cypress 时导航到该目录。我们可以在 Windows PowerShell 终端中使用以下命令来创建`cypress-tests`目录并导航到该目录：

```js
$ cd .\Documents
$ cd mkdir cypress-tests
```

成功运行这些命令后，我们将启动 PowerShell 并导航到我们刚刚创建的目录，使用以下命令：

```js
$ cd .\Documents\cypress-tests
```

创建目录后，我们将通过在 PowerShell 中运行以下命令来初始化一个空的 JavaScript 项目：

```js
$ npm init –y
```

这将创建一个默认的`package.json`文件，用于定义我们的项目。

## 在 Windows 上安装 Cypress

现在，我们将使用以下命令在我们的项目目录中使用 npm 安装 Cypress：

```js
$ npm install cypress --save-dev
```

运行此命令后，您应该能够看到 Cypress 的安装和安装进度。这种方法将 Cypress 安装为我们空项目的`dev`依赖项。

有关 macOS 安装，请参阅下一节主要部分。

## 总结-在 Windows 上安装 Cypress

在本节中，我们学习了如何在 Windows 操作系统上安装 Cypress。我们还学会了如何使用 PowerShell 向项目添加 Cypress，以及如何初始化一个空项目。在下一节中，我们将看看如何在 macOS 上安装 Cypress。

# 在 MacOS 上安装 Cypress

在本节中，我将使用 macOS 机器来安装 Cypress 和 Node.js。在本节结束时，您将学会如何初始化一个空的 JavaScript 项目，以及如何将 Cypress 测试框架添加到 macOS 中。我们还将深入探讨如何在我们的项目中使用 npm、Yarn 或直接 Cypress 下载。

## 安装 Node.js

以下步骤将指导您安装 Node.js：

1.  访问官方 Node.js 网站（[`nodejs.org/en/download/`](https://nodejs.org/en/download/)）。

1.  选择 macOS 安装程序选项。

1.  下载安装程序包。

1.  按照 Node.js 网站上的说明安装 Node.js 包。

接下来，让我们初始化项目。

## 初始化项目

要安装 Cypress，我们需要导航到项目文件夹，并在我们希望 Cypress 测试位于的位置进行安装。在我们的情况下，我们将在“文档”中创建一个名为`cypress-tests`的文件夹，然后在安装 Cypress 时使用我们的终端导航到该目录。然后，我们将启动我们的终端应用程序，并使用以下命令导航到我们刚刚创建的目录：

```js
$ cd  ~/Documents/cypress-tests
```

创建目录后，我们将通过运行以下命令初始化一个空的 JavaScript 项目：

```js
$ npm init –y
```

这将创建一个默认的`package.json`文件，用于定义我们的项目。

## 在 Mac 上安装 Cypress

要安装 Cypress，我们将使用 Node.js 捆绑的 npm 包管理器。为了实现这一点，我们需要运行以下命令：

```js
$ npm install cypress --save-dev
```

运行此命令后，您应该能够在`package.json`文件中看到 Cypress 的安装进度，并在命令行上看到安装进度。这种方法将 Cypress 安装为我们空项目的`dev`依赖项。

作为 Windows 和 macOS 都可以使用的替代包管理器，您可以使用 Yarn。我们将在下一节中看到如何使用 Yarn 安装 Cypress。

## 使用 Yarn 安装 Cypress

在 Windows 和 macOS 中，您可以选择另一种包管理器。可用的替代方案之一是 Yarn 包管理器。与 npm 一样，您首先需要使用 macOS Homebrew 包管理器下载 Yarn 包管理器，方法是运行以下命令：

```js
$ brew install yarn
```

就像 npm 一样，Yarn 可以管理项目的依赖关系，并且可以用作项目管理器。Yarn 比 npm 的一个优势是它能够以一种不需要重新下载依赖项的方式缓存已下载的包，因此更好地利用资源。

安装 Yarn 后，我们可以使用它来安装包，就像我们使用 npm 一样，方法是运行以下命令：

```js
$ yarn add cypress –dev
```

我们还有最后一种安装方法，即通过直接下载。这将在下一节中介绍。

## 通过直接下载安装 Cypress

我们可以通过直接下载在 Windows、Linux 或 macOS 上安装 Cypress。如果您不需要安装 Cypress 附带的依赖项，或者只是尝试 Cypress，这种方法是推荐的。重要的是要注意，尽管这是安装 Cypress 的最快方式，但这个版本不具备诸如记录测试到仪表板的功能。

以下步骤将指导您通过直接下载安装 Cypress：

1.  导航到[`cypress.io`](https://cypress.io)。

1.  选择**立即下载**链接。

Cypress 将自动下载，因为它将自动检测下载.zip 文件的用户的操作系统。然后，您应该解压缩 zip 文件并在不安装任何其他依赖项的情况下运行 Cypress。

## 总结-在 macOS 上安装 Cypress

在本节中，我们学习了如何使用 npm 在 macOS 上安装 Cypress 测试框架，以及如何初始化一个将利用 Cypress 测试的空 JavaScript 项目。我们还学习了如何使用 Yarn 软件包管理器安装 Cypress，以及如何在不使用任何软件包管理器的情况下直接下载 Cypress 到我们的项目中。在下一节中，我们将看看如何打开 Cypress 测试框架。

# 打开 Cypress

安装 Cypress 是编写端到端测试旅程的第一步；现在，我们需要学习如何使用 Cypress 提供的工具来使用图形用户界面和仪表板运行测试。有四种方法可以运行已在您的计算机上安装的 Cypress 可执行文件。打开 Cypress 后，您应该看到 Cypress 测试运行器。无论以哪种方式打开 Cypress，您所看到的测试运行器仪表板都是相同的。以下部分详细介绍了打开和运行 Cypress 的不同方法。

## 使用 Npx 运行

npx 用于执行 npm 包二进制文件，并且从版本 5.2 开始，所有 npm 版本都带有 npx。也可以使用 npm 从`npmjs.com`安装 npx。要使用 npx 运行 Cypress，您需要运行以下命令：

```js
 npx cypress open
```

## 使用 Yarn 运行

如果使用 Yarn 安装了 Cypress，您可以使用以下命令打开 Cypress：

```js
Yarn run cypress open
```

## 使用 node 模块路径运行

Cypress 也可以通过引用 node 模块上的安装根路径来运行。这可以通过使用`node_modules` bin 中 Cypress 可执行文件的完整路径或使用 npm bin 快捷方式来实现，如下节所示。

### 使用完整路径启动 Cypress

这种启动 Cypress 的方法引用了`node_modules`中安装的 Cypress 可执行文件，并通过运行可执行文件来打开 Cypress：

```js
$ ./node_modules/.bin/cypress open
```

### 使用快捷方式启动 Cypress

就像使用完整路径启动 Cypress 一样，这种方法以相同的方式启动 Cypress，但是它不是引用完整路径，而是使用 npm bin 变量来定位`node_modules` bin 文件夹的默认位置：

```js
$(npm bin)/cypress open
```

## 桌面应用程序启动

如果您将应用程序下载为桌面应用程序，您可以通过导航到解压后的 Cypress 文件夹的位置，并单击该文件夹中存在的 Cypress 可执行文件来打开 Cypress。

现在我们已经成功通过我们喜欢的方法打开了 Cypress，我们将看看如果我们不想使用 Cypress 捆绑的默认浏览器，我们如何在 Cypress 中选择替代浏览器。

## 总结-打开 Cypress

在本节中，我们学习了如何打开 Cypress 测试框架仪表板，以及如何以不同的方式运行 Cypress，包括使用*npx*、*Yarn*或*node_modules*路径运行 Cypress 仪表板。在下一节中，我们将学习如何切换在 Cypress 中运行的测试的浏览器。

# 切换浏览器

Cypress 在安装时默认使用 Electron 作为浏览器，但它也可以与其他兼容的浏览器集成，这些浏览器包含**Chromium 项目**，除了 Firefox。目前，Cypress 支持 Firefox 浏览器、Chrome 浏览器、Chromium 和 Edge 浏览器。启动 Cypress 时，它将自动查找运行机器上的所有兼容浏览器，您可以随时使用测试运行器在这些浏览器之间切换。要从一个浏览器切换到另一个浏览器，您需要点击右上角的浏览器按钮，并从下拉链接中选择替代浏览器。

Cypress 测试也可以通过命令行在不同的浏览器上运行或打开，可以通过在打开 Cypress 测试运行器或运行 Cypress 测试时指定浏览器来实现。所有基于 Chromium 的浏览器、Edge 和 Firefox 都可以使用以下命令在命令行中启动：

```js
$ cypress run --browser {browser-name}
```

命令中指定的`browser-name`可以是 Edge、Chrome 或 Firefox。要指定 Cypress 应启动的浏览器的路径，您可以选择使用浏览器的可执行二进制文件而不是浏览器的名称来运行浏览器名称，如下所示：

```js
$ cypress run --browser /path/to/binary/of/browser
```

能够在 Cypress 中切换浏览器可以确保用户可以在不同设备上运行其测试套件，并验证不同浏览器的输出在整个测试套件中是一致的。在 Cypress 上切换浏览器还可以确保测试的验证可以进行，并且所有元素可见或可以在一个浏览器上执行的操作也可以在另一个浏览器上执行。

让我们利用到目前为止所学到的知识来尝试使用 Cypress 进行实际练习。

## 练习

结合打开 Cypress 和切换浏览器的知识，尝试以下步骤：

1.  导航到我们在初始化 Cypress 时创建的文件夹。

1.  运行 Cypress 启动时自动生成的所有默认测试。

1.  在测试运行器上切换浏览器。

1.  使用不同的浏览器重新运行测试。

现在我们已经学会了如何在不同的浏览器中运行 Cypress 测试，在接下来的部分中，我们将探讨如何使用 npm 脚本自动化运行测试的过程。

## 回顾 - 切换浏览器

在本节中，我们学习了 Cypress 支持的不同浏览器以及如何切换不同的 Cypress 浏览器，无论是使用命令行还是使用 Cypress 仪表板。我们还进行了一个简单的练习，以帮助我们了解 Cypress 浏览器切换的工作原理，以及如何使用 Cypress 运行我们的测试。在下一节中，我们将学习如何将 npm 脚本添加到我们的`package.json`文件中，以自动化一些 Cypress 任务。

# 添加 npm 脚本

`scripts`是`package.json`的属性，它使用户能够通过 JavaScript 应用程序的命令行运行命令。npm 脚本可用于向应用程序的属性添加环境变量，将应用程序打包成生产就绪的捆绑包，运行测试，或自动化 JavaScript 应用程序中的任何其他活动。npm 脚本可以根据用户的偏好和应用程序进行自定义，也可以按照`npmjs.com`定义的方式使用。在本节中，我们将学习如何编写 npm 脚本来运行我们的 Cypress 测试，打开我们的 Cypress 测试，甚至组合不同的 npm 脚本以实现不同的结果。

## 打开 Cypress 命令脚本

要创建一个`scripts`命令来打开 Cypress，您需要编写脚本名称，然后添加 npm 在执行脚本时将运行的命令。在这种情况下，我们打开 Cypress 的命令将嵌入到一个名为`open`的脚本中。我们可以通过将以下命令添加到`package.json`中的`scripts`对象来实现这一点：

```js
"scripts": {
  "open": "npx cypress open" 
}
```

要运行`open`命令，您只需运行`npm run open`命令，测试运行器应该会在 Cypress 测试运行器中选择的默认浏览器上打开。

## 回顾 - 添加 npm 脚本

在本节中，我们学习了什么是 npm 脚本以及如何将它们添加到`package.json`文件中。我们还学会了如何运行我们已经添加到`package.json`文件中的 npm 脚本，以执行和自动化项目中的任务。接下来，我们将学习如何在 Cypress 中运行测试。

# 运行 Cypress 测试

在本节中，我们将重点放在如何在浏览器上运行 Cypress 测试。为此，我们将编写测试脚本，可以像打开 Cypress 脚本一样运行测试：

```js
"scripts": {
"test:chrome": "cypress run –browser chrome",
"test:firefox": "cypress run –browser firefox" 
}
```

前面的脚本将用于在 Chrome 浏览器或 Firefox 浏览器中运行测试，具体取决于用户在命令行终端上运行的命令。要执行测试，您可以运行`npm run test:chrome`在 Chrome 中运行测试，或者`npm run test:firefox`在 Firefox 中执行测试。命令的第一部分指示 Cypress 以无头模式运行测试，而第二部分指示 Cypress 在哪个浏览器中运行测试。运行 Cypress 测试不仅限于 Chrome 和 Firefox，并且可以扩展到 Cypress 支持的任何浏览器，还可以根据需要自定义运行脚本的名称。

## 使用脚本组合 Cypress 命令

`package.json`中的`scripts`对象为您提供了灵活性，可以组合命令以创建可以执行不同功能的高级命令，例如将环境变量传递给正在运行的测试，甚至指示 Cypress 根据传递的变量运行不同的测试。组合 Cypress 命令确保我们编写短小的可重用语句，然后可以用来构建执行多个功能的命令。在下面的示例中，我们将使用`scripts`对象编写一个命令来打开 Cypress，设置端口，设置环境，并根据我们选择运行的命令设置浏览器为 Chrome 或 Firefox：

```js
"scripts": {
"test": "cypress run",
"test:dev": "npm test --env=dev",
"test:uat": "npm test --env=uat",
"test:dev:chrome": "npm run test:dev –browser chrome",
"test:uat:chrome": " npm run test:uat –browser chrome", 
"test:dev:firefox": "npm run test:dev –browser firefox",
"test:uat:firefox": "npm run test:uat –browser firefox" 
}
```

前面的脚本可以在两个浏览器中运行 Cypress 测试。这些脚本还有助于确定要根据`-env`变量运行测试的环境。最后两个脚本组合了一系列运行 Cypress 的脚本，附加了一个环境变量，并选择了要在其上运行测试的浏览器，这使得`package.json`中的脚本功能在编写要在测试套件中执行的 Cypress 命令时非常有用。要在 Firefox 中运行测试，我们只需运行`npm run test:uat:firefox`命令进行 UAT 测试，或者`test:dev:firefox`进行`dev`环境的测试。您还可以使用`test:uat:chrome`在 Chrome 中运行 UAT 测试，或者`test:dev:chrome`进行`dev`环境的测试。

重要提示

要在不同的环境中运行测试，您需要在项目中已经设置好在不同环境中运行测试的配置。

## 总结-运行 Cypress 测试

在本节中，我们看了如何在 Cypress 中执行我们的测试。我们还看了不同的方法，通过传递环境变量和更改脚本中的参数来执行我们的测试的 npm 脚本。我们还学会了如何组合多个 Cypress 命令来运行我们的测试，从而减少我们需要编写的代码量。

# 总结

在本章中，我们学习了如何在 Windows 和 Mac 操作系统上安装 Cypress。在两种安装中，我们涵盖了作为下载应用程序或通过命令行安装 Cypress。我们还涵盖了使用 Node.js（npm）自带的默认包管理器或第三方依赖管理器（如 Yarn）。我们学会了如何利用测试运行器来运行我们的测试，以及如何在`package.json`中自动化我们的脚本，以帮助我们有效地运行测试。为了测试我们的知识，我们还进行了一个练习，练习在不同的 Cypress 浏览器中运行测试。

在下一章中，我们将深入探讨 Selenium 和 Cypress 之间的区别，以及为什么 Cypress 应该是首选。我们将进一步建立在本章中所获得的对 Cypress 的理解基础上。
