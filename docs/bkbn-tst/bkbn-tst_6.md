# 第六章. 自动化 Web 测试

在讨论了测试 Backbone.js 应用程序的实际技术之后，我们现在将探讨自动化测试基础设施的各种方法。能够以编程方式运行我们的测试集合，使得在开发过程中单个开发者手动运行测试驱动器页面的单一用例之外，出现了新的和令人兴奋的用例。在本章中，我们将探讨以下自动化和开发主题：

+   调查自动化测试基础设施的场景和动机

+   调查以编程方式运行 Backbone.js 应用程序测试套件的多种方法

+   介绍用于前端测试的 PhantomJS 和适配器工具

+   将我们现有的测试基础设施集成到 PhantomJS 环境中

+   在完成本书后，总结我们对 Backbone.js 应用程序测试原则和实践的讨论，并提供下一步的建议和资源

# 超越人类和浏览器的测试世界

到目前为止，我们的测试开发工作流程包括编写测试套件，将它们添加到测试驱动器页面，并在开发计算机上的网络浏览器中启动测试页面。然而，测试基础设施可以用于比手动运行网络报告更多的场景。通过检查以下用例，我们将看到自动在任意环境中运行我们的测试集合（例如，从命令行或构建脚本，可能无需网络浏览器）在应用程序开发过程中具有巨大的潜力。

## 持续集成

在协作软件开发中，当工程师分别开发代码，稍后将其合并到公共代码库中时，可能会出现问题。更改之间的意外交互可能导致集成错误，破坏整体应用程序。

对于此类错误的一种缓解方法是持续集成，它大量依赖于自动化测试。持续集成将应用程序代码聚合并测试，以早期和自动地检测集成错误。关于这个主题的深入介绍，请参阅*马丁·福勒*的*《持续集成》*，可在[`martinfowler.com/articles/continuousIntegration.html`](http://martinfowler.com/articles/continuousIntegration.html)找到。

持续集成的过程通常是通过一个专用的服务器来实现的，该服务器逐步收集代码更改，创建一个干净的应用程序环境，运行构建命令，并对命令输出采取行动。例如，假设我们有一个存储在 GitHub 上的 Node.js 应用程序。一个持续集成服务器可以从 GitHub 下载代码更改，为应用程序创建一个新的构建目录，安装包依赖项（例如，`npm install`），并运行测试（例如，`npm test`）。如果任何测试失败，服务器将通知负责这些更改的开发者。一些流行的持续集成服务器包括 Jenkins ([`jenkins-ci.org/`](http://jenkins-ci.org/)) 和 Travis ([`travis-ci.org/`](https://travis-ci.org/))。

## 持续部署

持续部署服务器是持续集成服务器的增强版本，如果所有测试都通过，它还会将代码部署到实际的应用程序环境（例如，生产环境）。它依赖于自动化测试来验证整个应用程序，以便代码更改可以尽可能快地推出，同时至少保留一些质量保证的表象。《为什么持续部署？》这篇文章由*埃里克·莱斯*撰写，可在[`www.startuplessonslearned.com/2009/06/why-continuous-deployment.html`](http://www.startuplessonslearned.com/2009/06/why-continuous-deployment.html)找到，是了解持续部署背后的动机和实践的好起点。

## 其他场景

测试自动化使得许多其他有用的应用成为可能。例如，称为**监视器**或**守卫**的开发工具定期检查代码的修改，并在文件更改时执行进一步的操作。监视器通常在开发机器上使用，以自动运行测试并在代码更改导致一个或多个测试失败时显示警报，这样开发者可以快速且轻松地发现错误。

**跨浏览器测试**是自动化使工作变得更简单的另一个领域。虽然程序员可以手动在不同的目标浏览器上运行测试集合，但这通常既耗时又容易出错，而且很无聊。幸运的是，有一些测试工具可以在不进行人工交互的情况下，在多个任意浏览器上自动运行测试。

# 自动化浏览器环境

在介绍了一些激励性的用例之后，我们现在转向自动化测试基础设施的细节。我们将介绍以下用于程序化驱动 Backbone.js 测试的技术：

+   在真实浏览器中远程控制测试

+   在浏览器模拟库中运行测试

+   在无头浏览器环境中执行测试

+   结合前三种方法

## 远程控制浏览器

最全面的自动化技术是远程控制网络浏览器。远程控制意味着程序可以执行人类使用真实网络浏览器所能做的操作——打开浏览器到指定页面、点击链接、填写输入等。

最受欢迎的远程控制框架之一是 **Selenium** ([`docs.seleniumhq.org/`](http://docs.seleniumhq.org/))。Selenium 提供了许多 **web drivers**，这些是程序适配器，可以连接到真实浏览器并触发通过正常用户界面的操作。Selenium 支持多种环境，为各种浏览器提供不同操作系统的 web drivers，包括 Chrome、Safari、Firefox 和 Internet Explorer。

### 小贴士

Selenium 项目包含的功能和功能远不止浏览器远程控制。值得注意的是，Selenium 可以使用其他测试执行方法，包括无头网络工具如 PhantomJS。有关起点和更多信息，请参阅 Selenium 项目页面 ([`docs.seleniumhq.org/projects/`](http://docs.seleniumhq.org/projects/)) 和 web driver 列表 ([`docs.seleniumhq.org/projects/webdriver/`](http://docs.seleniumhq.org/projects/webdriver/))。

使用像 Selenium 这样的远程控制工具自动化我们的测试基础设施涉及两个基本步骤：打开并运行测试驱动程序页面，然后推断测试是否通过。例如，我们可以在代码示例中编写一个 Selenium 脚本，打开浏览器窗口到笔记应用程序测试驱动程序页面 `notes/test/test.html`。然后，Selenium 脚本可以抓取报告页面 HTML 以检查 DOM 中的明显字符串，如 `failures: 0`，并使用适当的成功/失败退出代码终止脚本。

因此，像 Selenium 这样的远程控制工具非常强大，因为它们可以自动执行任何真实浏览器可以执行的操作。而且，使用像 Selenium 这样的跨平台兼容工具，我们可以从单个脚本中运行几乎所有现代浏览器/操作系统组合的测试。

### 小贴士

**托管测试自动化提供商**

利用 Selenium 广泛的测试环境支持，供应商现在提供允许用户上传 Selenium 测试脚本、指定所需的操作系统/浏览器配置数组，并让服务运行并返回测试报告的服务。这样的供应商之一是 Sauce Labs ([`saucelabs.com/`](https://saucelabs.com/))，它使用各种 Selenium 支持的测试环境在虚拟机上运行用户脚本。此类托管服务通常是获得广泛浏览器兼容性覆盖的最快方式，且开发者工作量最小。

远程控制方法确实有一些缺点，其中第一个是测试工具可能相对较慢。脚本可能需要几秒钟甚至几分钟才能连接到目标浏览器并运行测试驱动页面。此外，这些框架需要真实网络浏览器和桌面窗口化系统。这对于无头构建/持续集成来说可能是一个问题，这意味着它们默认没有安装图形用户界面或窗口环境。

## 模拟的浏览器环境

另一种自动化方法是使用测试友好的浏览器环境和状态的模拟来替换网络浏览器。通常，浏览器模拟库在浏览器内部提供 JavaScript API 的实现，例如 DOM 对象（例如，`window`和`document`）、CSS 选择器和 JSON 接口。

**JSDom** ([`github.com/tmpvar/jsdom`](https://github.com/tmpvar/jsdom)) 是一个流行的模拟库，它提供了一个相当完整的浏览器环境。JSDom 是用 JavaScript 编写的，并打包为 Node.js 模块。由于 Node.js 可以轻松脚本化，JSDom 为我们提供了一个从命令行集成和运行 Backbone.js 测试的良好起点。

测试自动化是一个如此常见的用例，以至于围绕 JSDom 编写了几个测试友好的库。其中一个这样的库是 Zombie.js ([`zombie.labnotes.org/`](http://zombie.labnotes.org/))，它提供了方便的浏览器抽象和与各种测试框架的集成，包括 Mocha。使用 Zombie.js 这样的库，我们可以编写一个 Node.js 脚本，创建一个模拟的浏览器模拟，导航到我们的 Backbone.js 测试驱动页面，并抓取测试结果的 HTML 以检查是否有测试失败。有关使用 Zombie.js 和 Mocha 测试 JavaScript 网络应用的更深入介绍，请参阅 Pedro Teixeira 的《Using Node.js for UI Testing》([`www.packtpub.com/testing-nodejs-web-uis/book`](http://www.packtpub.com/testing-nodejs-web-uis/book))。

浏览器模拟库之所以运行速度快，是因为它们在与测试代码相同的底层 JavaScript 引擎中运行模拟代码，而不需要外部依赖（例如，在真实的网络浏览器可执行文件中）。由于模拟 JavaScript 代码与应用程序和测试在同一进程中运行，因此模拟库通常非常易于扩展。

然而，模拟存在一些关键缺点。一个主要问题是模拟可能会偏离真实网络浏览器的真实环境。复杂的浏览器交互，如高度链式的事件触发或复杂的 DOM 操作，可能会破坏模拟或与真实浏览器表现不同。此外，浏览器模拟库只提供单一浏览器环境实现，因此无法测试各种真实浏览器之间的怪癖和差异。

## 无头网络浏览器

在远程控制浏览器和模拟库之间是无头网络浏览器。无头浏览器取一个真实的网络浏览器，去掉用户界面，只留下 JavaScript 引擎和环境。剩下的就是一个可以导航到网页、在浏览器环境中执行 JavaScript 以及通过非图形界面（如警报和控制台日志）进行通信的命令行工具。

最受欢迎的无头工具包之一是 PhantomJS ([`phantomjs.org/`](http://phantomjs.org/))，它基于为 Safari 等浏览器提供动力的 **WebKit** 开源浏览器 ([`www.webkit.org/`](http://www.webkit.org/))。PhantomJS 通过脚本支持和 JavaScript API 丰富了 WebKit。

将 Backbone.js 应用程序测试与无头浏览器集成类似于配置远程控制浏览器。方便的是，PhantomJS 内置了对广泛测试基础设施的原生支持，并为许多其他测试提供了第三方适配器。有关更多测试支持细节，请参阅[`github.com/ariya/phantomjs/wiki/Headless-Testing`](https://github.com/ariya/phantomjs/wiki/Headless-Testing)。

无头网络工具结合了之前自动化方法的一些最佳特性，包括以下内容：

+   无头 JavaScript 引擎通常比远程控制框架更快

+   浏览器环境是**真实**的，这避免了在浏览器模拟中可能发现的某些 API 和正确性问题

+   无头框架通常易于安装，可以在没有窗口环境的服务器上运行

同时，无头浏览器在启动和运行浏览器引擎时会带来一些性能损失。它们也放弃了跨浏览器功能，因为无头工具绑定到特定的网络浏览器引擎实现。考虑到整体优缺点，无头框架在许多相互排斥的自动化功能之间提供了一个良好的折衷方案。

## 多环境聚合器

利用各种方法的好处，许多框架将不同的自动化方案聚合到一个单一包中。例如，以下测试框架可以编程驱动主要网络浏览器和 PhantomJS 的测试：

+   **Testem** ([`github.com/airportyh/testem`](https://github.com/airportyh/testem))

+   **因果律** ([`karma-runner.github.io/`](http://karma-runner.github.io/))

聚合框架是可取的，因为它们允许单个测试集合在不同的自动化环境中重复使用，尽管一些工具的设置和维护比单个自动化工具更困难。

# 使用 PhantomJS 进行无头测试

作为具体的自动化示例，我们将调整现有的 Backbone.js 测试基础设施以使用 PhantomJS。PhantomJS 为 Backbone.js 测试提供了一套易于使用的特性和功能——它运行速度快，相对容易设置，并提供了一个真实的（无头）浏览器。从实际的角度来看，较大的 Backbone.js 应用程序通常需要一个真实的浏览器引擎才能正常工作，尤其是那些测试浏览器环境中的模糊和复杂部分的程序。

## 安装 PhantomJS 和支持工具

要使用 PhantomJS 启动，请按照 [`phantomjs.org/download.html`](http://phantomjs.org/download.html) 上的说明安装工具包。请注意，安装过程取决于操作系统，有适用于 Windows、Mac OS X 和 Linux 的软件包。或者，可以直接使用 NPM 通过 `phantomjs` Node.js 包装器安装 PhantomJS ([`github.com/Obvious/phantomjs`](https://github.com/Obvious/phantomjs))。

### 注意

我们在本节中提供了来自类似 UNIX 的操作系统（如 Linux 和 Mac OS X）的命令行示例。同时，PhantomJS 和 Node.js 在 Windows 上也有第一级支持，因此接下来的示例应该与 Windows 上可以运行的内容大致相似。

安装完成后，您可以验证 PhantomJS 二进制文件是否可用：

```js
$ phantomjs --help

```

在 PhantomJS 安装完成后，我们接下来转向 Mocha-PhantomJS 桥接库。Mocha-PhantomJS 使用 PhantomJS 运行 Mocha 测试驱动页面，并将测试结果转换为格式化的命令行输出。该库在测试失败时抛出适当的错误，这使得它对于脚本编写非常有用。有关附加功能和详细信息的在线文档请参阅 [`metaskills.net/mocha-phantomjs/`](http://metaskills.net/mocha-phantomjs/)。

要安装 Mocha-PhantomJS，您需要 Node.js 框架，可以通过查阅 [`nodejs.org/download/`](http://nodejs.org/download/) 上的说明来获取。现代 Node.js 安装包括用于 Mocha-PhantomJS 的 NPM 包管理器工具。我们可以使用以下命令来确认 Node.js 和包管理器已正确安装：

```js
$ node --help
$ npm --help

```

接下来，使用全局 NPM 标志（`-g`）安装 Mocha-PhantomJS，以便在 shell 的任何位置都可以使用 `mocha-phantomjs` 二进制文件：

```js
$ npm install -g mocha-phantomjs

```

在 NPM 完成安装后，使用以下命令检查 Mocha-PhantomJS 是否可用：

```js
$ mocha-phantomjs --help

```

## 使用 PhantomJS 运行 Backbone.js 测试

在安装了必要的工具后，我们现在可以将 Backbone.js 测试基础设施调整以运行 PhantomJS。Mocha-PhantomJS 提供了一个替换代理对象 `mochaPhantomJS`，用于控制 Mocha 测试和报告。我们只需在测试驱动网页中替换通常调用 `mocha.run()` 的真实 `mocha` 对象。将以下代码片段插入测试驱动页面将允许 Mocha 在真实浏览器和 PhantomJS 中同时运行：

```js
<!-- Test Setup -->
<script>
  var expect = chai.expect;
  mocha.setup("bdd");

  window.onload = function () {
    (window.mochaPhantomJS || mocha).run();
  };
</script>
```

一旦我们修改了测试驱动页面中的 `(window.mochaPhantomJS || mocha).run()` 函数调用，我们就可以使用 Mocha-PhantomJS 执行页面测试。例如，如果我们修改了上一章中的 Notes 应用程序测试驱动文件 `chapters/05/test/test.html` 中的 `mochaPhantomJS` 变更，我们就可以运行该文件并生成以下命令行报告：

```js
$ mocha-phantomjs chapters/05/test/test.html

 App.Views.NotesItem
 remove
√ is removed on model destroy
 render
√ renders on model change w/ stub
√ renders on model change w/ mock
 DOM
√ renders data to HTML
 actions
√ views on click
√ edits on click
√ deletes on click
 App.Routers.Router
√ can route to note
√ can route around

 9 tests complete (39 ms)

```

审查这份报告，我们可以看到所有测试都通过了，并且 PhantomJS 测试运行相当快，耗时 39 毫秒。通过这些微小的测试驱动网页更改，我们可以从命令行或构建脚本中使用 PhantomJS 运行几乎任何测试网页。

## 在代码示例中自动化测试

将这些建议的原则付诸实践，本书中展示的大多数测试代码示例都被脚本化为在 PhantomJS 下从命令行运行。如果您审查可下载的代码示例仓库，您会注意到所有章节和应用程序测试页面实际上都使用了 `(window.mochaPhantomJS || mocha).run()` 函数调用，而不是原始的 `mocha.run()` 语句。

将 PhantomJS 集成到代码示例中为我们在本章前面讨论的一些自动化测试用例提供了一个实用的起点。具体来说，以下示例实现了以下自动化场景：

+   **命令行测试**：代码示例包含一个 Node.js NPM `package.json` 文件，其中包含可以运行章节和应用程序测试驱动页面的脚本命令，使用 PhantomJS。

+   **持续集成服务器**：代码示例的 GitHub 仓库（[`github.com/ryan-roemer/backbone-testing/`](https://github.com/ryan-roemer/backbone-testing/））使用 Travis 持续集成服务器进行自动化的失败警报。Travis 被配置为在每次代码更改时使用 PhantomJS 运行所有示例测试。对于本书中展示的测试基础设施，Travis 是一个特别好的选择，因为它的构建环境已经包含了 PhantomJS，并且对 Node.js 和 NPM 模块（如 Mocha-PhantomJS）非常友好。要查看所有这些功能在实际中的运行情况，您可以在任何时间通过浏览器导航到 [https://travis-ci.org/ryan-roemer/backbone-testing](https://travis-ci.org/ryan-roemer/backbone-testing) 来检查我们在这本书中讨论的所有代码的实时构建状态。（希望您会发现我们的所有测试都通过了！）

# 离别时的思考，下一步行动和未来想法

我们现在已经完成了使用 Mocha、Chai 和 Sinon.JS 测试 Backbone.js 应用程序基础知识的旅程。我们探讨了每个测试框架的背景、配置和使用，并尝试了多种互补的工具和助手。我们回顾了 Backbone.js 应用程序开发、特定组件测试目标，并在一个完整的 Backbone.js 应用程序周围编写了测试集合。那么，接下来是什么？

我们的第一建议是回顾各种测试技术的在线文档。书中使用的所有框架的官方 API 和指南都相当不错，可以为实际中可能出现的更复杂的测试场景提供起点。作为一个复习，我们核心测试堆栈的文档网站包括以下内容：

+   **Mocha**：[`visionmedia.github.io/mocha/`](http://visionmedia.github.io/mocha/)

+   **Chai**：[`chaijs.com/`](http://chaijs.com/)

+   **Sinon.JS**：[`sinonjs.org/`](http://sinonjs.org/)

在框架文档之后，您可以回顾本书中提供的文章、博客和书籍建议。特别是，第二章中关于一般测试方法和 Backbone.js 测试的*创建 Backbone.js 应用程序测试计划*参考，对于寻求更广泛背景的软件开发和测试技术的读者来说是非常好的资源。

最后，我们建议您下载并安装本书的代码示例。这些示例基本上是我们在这本书中讨论的原理的实际应用，将有用的应用程序、测试和文件组合成一个单一包。此外，它们还提供了更多测试和自动化技术的示例，供您自行探索，包括以下内容：

+   **样式检查**：JavaScript 样式检查器会自动分析源文件以查找语言或约定错误。在软件开发过程中，检查器非常有价值，通常能早期发现编程错误，并在测试可能遗漏的地方。此外，样式检查器可以强制执行单个应用程序团队成员的一致编码风格。代码示例使用 JSHint（[`www.jshint.com/`](http://www.jshint.com/））来检查本书中讨论的所有应用程序和测试示例。您可以在代码示例中的`package.json`文件中查看我们的 JSHint 使用情况，包括脚本命令中的`style`、`style-server`和`style-client`。

+   **代码覆盖率**：代码覆盖率是一种量化应用程序实际被测试覆盖程度的技术。覆盖率工具在测试过程中在幕后运行，记录哪些应用程序代码行被执行，并提供一个报告，测量每个应用程序文件中覆盖的行数。代码示例为 Notes 应用程序提供了一个测试驱动页面，位于`notes/test/coverage.html`，使用**Blanket.js**（[http://blanketjs.org/](http://blanketjs.org/））提供覆盖率报告。您可以在[http://backbone-testing.com/notes/test/coverage.html](http://backbone-testing.com/notes/test/coverage.html)在线运行 Notes 测试和覆盖率报告。

剩下的部分就留给你了。虽然我们已到达这本书的终点，但测试的世界将继续以新的和有趣的方式不断前进。我们祝愿你在继续学习和探索更多 Backbone.js 应用程序开发的测试工具、方法和主题时好运。

# 摘要

在本章中，我们学习了如何通过介绍测试自动化方法和用例，从测试过程中移除手动浏览器交互。我们调查了不同的工具来从命令行驱动我们的测试，并通过使用 PhantomJS 驱动我们的 Backbone.js 应用程序测试，完成了一个具体的测试自动化实现。

此外，我们还留下了一些关于我们在本书过程中发展出的原则以及下一步该何去何从的最终思考。希望你现在有了创建自己的 Backbone.js 测试基础设施、应用良好的测试驱动开发实践以及自信地应对前端测试的基础和方向。
