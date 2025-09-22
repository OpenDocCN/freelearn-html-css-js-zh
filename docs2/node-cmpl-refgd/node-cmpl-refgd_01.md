# 关于 Node.js

Node.js 是一个令人兴奋的新平台，用于开发 Web 应用、应用服务器、各种网络服务器或客户端，以及通用编程。它通过巧妙结合服务器端 JavaScript、异步 I/O 和异步编程，旨在实现网络应用的极端可伸缩性。它围绕 JavaScript 匿名函数和单线程事件驱动架构构建。

尽管只有几年历史，Node.js 已经迅速崛起，现在正发挥着重要作用。大小公司都在使用它进行大规模和小规模项目。例如，PayPal 已经将许多服务从 Java 转换为 Node.js。

Node.js 的架构与传统应用平台的选择有所不同。在其他应用平台中，线程被广泛用于扩展应用以填充 CPU，而 Node.js 由于线程固有的复杂性而放弃了线程。据说，使用单线程事件驱动架构，内存占用低，吞吐量高，负载下的延迟性能更好，编程模型更简单。Node.js 平台正处于快速增长的阶段，许多人将其视为传统使用 Java、PHP、Python 或 Ruby on Rails 的 Web 应用架构的有力替代品。

在其核心，它是一个独立的 JavaScript 引擎，通过扩展使其适用于通用编程，并明确关注应用服务器开发。尽管我们正在将 Node.js 与应用服务器平台进行比较，但它本身并不是一个应用服务器。相反，Node.js 是一种类似于 Python、Go 或 Java SE 的编程运行时。尽管有使用 Node.js 编写的 Web 应用框架和应用服务器，但它仅仅是一个执行 JavaScript 程序的系统。

它是基于非阻塞 I/O 事件循环和文件及网络 I/O 库层实现的，所有这些都是在 Chrome 浏览器背后的 V8 JavaScript 引擎之上构建的。Chrome 中实现的快速性能和功能改进迅速传递到 Node.js 平台。此外，一个团队正在开发一个在 Microsoft 的 ChakraCore JavaScript 引擎（来自 Edge 浏览器）之上运行的 Node.js 实现。这将通过不依赖于单一 JavaScript 引擎提供商来为 Node.js 社区提供更大的灵活性。访问 [`github.com/nodejs/node-chakracore`](https://github.com/nodejs/node-chakracore) 查看该项目。

Node.js 的 I/O 库足够通用，可以执行任何类型的服务器，执行任何 TCP 或 UDP 协议，无论是**域名系统**（**DNS**），HTTP，**互联网中继聊天**（**IRC**），还是 FTP。虽然它支持开发互联网服务器或客户端，但其最大的用例是在常规网站上，替代 Apache/PHP 或 Rails 堆栈，或补充现有网站。例如，使用 Socket.IO 库为 Node.js 添加实时聊天或监控现有网站可以轻松完成。它的轻量级和高性能特性通常使 Node.js 被用作**粘合剂**服务。

一个特别吸引人的组合是将使用 Docker 部署的小型服务部署到云托管基础设施中。一个大型应用程序可以分解成现在称为微服务的形式，这些服务可以使用 Docker 轻松地大规模部署。结果是适合敏捷项目管理方法，因为每个微服务都可以由一个小团队轻松管理，该团队在其各自的 API 边界上进行协作。

本书将为你介绍 Node.js。我们假设以下：

+   你已经知道如何编写软件

+   你熟悉 JavaScript

+   你对用其他语言开发 Web 应用程序有所了解

在本章中，我们将涵盖以下主题：

+   Node.js 简介

+   为什么你应该使用 Node.js

+   Node.js 的架构

+   使用 Node.js 的性能、利用率和可伸缩性

+   Node.js、微服务架构和测试

+   使用 Node.js 实现十二要素应用模型

我们将直接进入开发工作应用程序，并认识到通常最好的学习方法是通过在工作代码中翻找。

# Node.js 的功能

Node.js 是一个在 Web 浏览器之外编写 JavaScript 应用程序的平台。这并不是我们在 Web 浏览器中熟悉的 JavaScript！例如，Node.js 中没有内置 DOM，也没有其他浏览器功能。

除了其执行 JavaScript 的本地能力之外，捆绑的模块提供了以下这类功能：

+   命令行工具（以 shell 脚本风格）

+   一种交互式终端风格的程序，即**读取-评估-打印循环**（**REPL**）

+   优秀的进程控制功能，用于监督子进程

+   一个用于处理二进制数据的缓冲对象

+   带有全面事件驱动回调的 TCP 或 UDP 套接字

+   DNS 查找

+   在 TCP 库文件系统访问之上构建的 HTTP、HTTPS 和 HTTP/2 客户端/服务器层

+   通过断言提供的内置基本单元测试支持

Node.js 的网络层是低级别的，但使用简单。例如，HTTP 模块允许你使用几行代码编写 HTTP 服务器（或客户端）。这很强大，但它让你，即程序员，非常接近协议请求，并要求你精确实现那些你应该在请求响应中返回的 HTTP 头。

典型的 Web 应用程序开发者不需要在 HTTP 或其他协议的低级别工作。相反，我们倾向于更高效地工作，使用高级接口。例如，PHP 开发者假设 Apache（或其他 HTTP 服务器）已经存在，提供 HTTP 协议，并且他们不需要实现堆栈的 HTTP 服务器部分。相比之下，Node.js 程序员确实实现了 HTTP 服务器，并将他们的应用程序代码附加到该服务器上。

为了简化情况，Node.js 社区有几个 Web 应用程序框架，如 Express，为典型程序员提供所需的更高级接口。你可以快速配置具有内置功能（如会话、cookies、静态文件服务、日志记录）的 HTTP 服务器，让开发者专注于他们的业务逻辑。其他框架提供 OAuth 2 支持，或专注于 REST API，等等。

Node.js 并不仅限于 Web 服务应用开发。围绕 Node.js 的社区已经将其应用于许多其他方向，

**构建工具**：Node.js 已成为开发用于软件开发或与服务基础设施通信的命令行工具的流行选择。Grunt 和 Gulp 被前端开发者广泛用于构建网站资产。Babel 被广泛用于将现代 ES-2016 代码转换为在旧浏览器上运行的代码。流行的 CSS 优化器和处理器，如 PostCSS，是用 Node.js 编写的。静态网站生成系统，如 Metalsmith、Punch 和 AkashaCMS，在命令行上运行并生成上传到 Web 服务器的网站内容。

**Web UI 测试**：Puppeteer 允许你控制一个无头-Chrome 网络浏览器实例。利用它，你可以开发控制现代全功能网络浏览器的 Node.js 脚本。典型的用例包括网络爬取和测试 Web 应用程序。

**桌面应用程序**：Electron 和 **node-webkit**（**NW.js**）是用于开发 Windows、macOS 和 Linux 桌面应用程序的框架。这些框架利用了大量的 Chrome，并通过 Node.js 库进行包装，使用 Web UI 技术开发桌面应用程序。应用程序使用现代的 HTML5、CSS3 和 JavaScript 编写，并可以利用领先的 Web 框架，如 Bootstrap、React 或 AngularJS。许多流行的应用程序都是使用 Electron 开发的，包括 Slack 桌面客户端应用程序、Atom 和 Microsoft Visual Code 编程编辑器、Postman REST 客户端、GitKraken GIT 客户端，以及 Etcher，它使得将操作系统镜像烧录到闪存驱动器以在单板计算机上运行变得极其简单。

**移动应用**：Node.js for Mobile Systems 项目允许你使用 Node.js 开发智能手机或平板电脑应用，适用于 iOS 和 Android。苹果的 App Store 规则禁止包含具有 JIT 功能的 JavaScript 引擎，这意味着正常的 Node.js 不能用于 iOS 应用。对于 iOS 应用开发，该项目使用 Node.js-on-ChakraCore 来规避 App Store 规则。对于 Android 应用开发，该项目使用常规的 Node.js 在 Android 上。截至写作时，该项目处于早期开发阶段，但看起来很有希望。

**物联网**（**IoT**）：据报道，它是一种非常流行的物联网项目语言，Node.js 也能在大多数基于 ARM 的单板计算机上运行。最明显的例子是 NodeRED 项目。它提供了一个图形编程环境，允许你通过连接模块来绘制程序。它具有面向硬件的输入和输出机制，例如，与树莓派或 Beaglebone 单板计算机上的通用输入/输出（**GPIO**）引脚进行交互。

# 服务器端 JavaScript

别再挠头了！当然你在做，挠你的头，自言自语，“浏览器语言怎么会出现在服务器上？”事实上，JavaScript 在浏览器之外有着漫长且在很大程度上不为人知的历史。JavaScript 是一种编程语言，就像任何其他语言一样，更好的问题是“为什么 JavaScript 应该被困在浏览器内部？”。

回到网络时代的初期，编写网络应用的工具还处于起步阶段。有些人正在尝试使用 Perl 或 TCL 来编写 CGI 脚本，而 PHP 和 Java 语言刚刚被开发出来。即使那时，JavaScript 也已经在服务器端得到了应用。一个早期的网络应用服务器是 Netscape 的 LiveWire 服务器，它使用了 JavaScript。微软的一些 ASP 版本使用了 JScript，这是他们版本的 JavaScript。在 Java 宇宙中，还有一个更近期的服务器端 JavaScript 项目，即 RingoJS 应用框架。Java 6 和 Java 7 都配备了 Rhino JavaScript 引擎。在 Java 8 中，Rhino 被新的 Nashorn JavaScript 引擎所取代。

换句话说，浏览器之外的 JavaScript 并不是什么新鲜事物，即使它并不常见。

# 你为什么应该使用 Node.js？

在众多可用的网络应用开发平台中，为什么你应该选择 Node.js？有很多技术栈可供选择；是什么让 Node.js 脱颖而出，超越其他平台？我们将在接下来的章节中看到。

# 流行度

Node.js 正迅速成为流行的开发平台，许多大小玩家都在采用。其中之一是 PayPal，他们正在用 Node.js 编写的新系统替换现有的基于 Java 的系统。有关 PayPal 的这篇博客文章，请访问[`www.paypal-engineering.com/2013/11/22/node-js-at-paypal/`](https://www.paypal-engineering.com/2013/11/22/node-js-at-paypal/)。其他大型 Node.js 采用者包括沃尔玛的在线电子商务平台、领英和 eBay。

根据 NodeSource 的数据，Node.js 的使用正在快速增长（访问[`nodesource.com/node-by-numbers`](https://nodesource.com/node-by-numbers)）。这些指标包括下载 Node.js 发布版的带宽增加、Node.js 相关 GitHub 项目的活动增加以及更多。

最好不要仅仅跟随潮流，因为潮流声称他们的软件平台可以做酷的事情。Node.js 确实做一些酷的事情，但更重要的是它的技术优势。

# 在堆栈的所有级别上使用 JavaScript

在服务器和客户端使用相同的编程语言一直是网络上的一个长期梦想。这个梦想可以追溯到 Java 的早期，当时 Java 小程序被设想为用 Java 编写的服务器应用程序的前端，而 JavaScript 最初被设想为这些小程序的轻量级脚本语言。由于各种原因，Java 从未实现其作为客户端编程语言的炒作。我们最终得到了 JavaScript 作为主要的浏览器端、客户端语言，而不是 Java。通常，前端 JavaScript 开发者与后端团队处于不同的语言宇宙中，后端团队可能使用 PHP、Java、Ruby 或 Python 进行编码。

随着时间的推移，浏览器中的 JavaScript 引擎变得极其强大，使我们能够编写越来越复杂的浏览器端应用程序。通过 Node.js，我们可能最终能够在客户端和服务器两端都使用 JavaScript，从而在浏览器和服务器上实现使用相同编程语言的应用程序。

前端和后端使用相同的语言提供了几个潜在的好处：

+   同样的编程人员可以在网络两端工作

+   代码可以在服务器和客户端之间更容易地迁移

+   服务器和客户端之间存在共同的数据格式（JSON）

+   服务器和客户端存在共同的软件工具

+   服务器和客户端的常见测试或质量报告工具

+   在编写网络应用程序时，可以在两端使用视图模板

由于其在网络浏览器中的普遍存在，JavaScript 语言非常受欢迎。它在与其他语言的比较中表现良好，同时拥有许多现代、高级的语言概念。由于其受欢迎程度，有经验的 JavaScript 程序员人才库非常深厚。

# 利用谷歌对 V8 的投资

为了使 Chrome 成为一个受欢迎且优秀的网络浏览器，谷歌投资于将 V8 打造成为一个超级快速的 JavaScript 引擎。因此，谷歌有巨大的动力不断改进 V8。V8 是 Chrome 的 JavaScript 引擎，也可以独立执行。Node.js 建立在 V8 JavaScript 引擎之上。

随着 Node.js 对 V8 团队的重要性日益增加，随着更多的人关注 V8 的改进，有可能实现更快 V8 性能的协同效应。

# 更精简、异步、事件驱动的模型

我们稍后会深入探讨这个问题。Node.js 的架构，一个单一的执行线程，一个巧妙的事件驱动异步编程模型，以及一个快速的 JavaScript 引擎，比基于线程的架构开销更小。

# 微服务架构

软件开发中的一个新趋势是微服务理念。微服务专注于将大型网络应用拆分为小型、高度专注的服务，这些服务可以由小型团队轻松开发。虽然这并不是一个全新的想法，但它更多的是对旧客户端-服务器计算模型的重构，微服务模式与敏捷项目管理技术很好地结合，并为我们提供了更细粒度的应用部署。

Node.js 是一个实现微服务的优秀平台。我们稍后会深入探讨这个问题。

# Node.js 因为经历了重大分裂和敌对分支而变得更加强大

在 2014 年和 2015 年期间，Node.js 社区在政策、方向和控制方面面临了一次重大的分裂。**io.js**项目是由一群希望整合几个特性并改变决策过程的人推动的一个敌对分支。最终结果是 Node.js 和 io.js 仓库的合并，成立了一个独立的 Node.js 基金会来管理事务，社区正共同努力朝着共同的方向前进。

治愈这一裂痕的一个具体结果是新 ECMAScript 语言特性的快速采用。V8 引擎正在快速采用这些新特性以推进 Web 开发的状态。反过来，Node.js 团队也在 V8 中快速采用这些特性，这意味着 Promise 和`async`函数正迅速成为 Node.js 程序员的现实。

底线是，Node.js 社区不仅存活了下来，而且由于 io.js 分支，社区及其培养的平台变得更加强大。

# 线程与事件驱动架构的比较

据说 Node.js 的卓越性能归因于其异步事件驱动架构以及使用 V8 JavaScript 引擎。这听起来很好，但这个声明的依据是什么？

V8 JavaScript 引擎是 JavaScript 实现中最快的之一。因此，Chrome 不仅被广泛用于查看网站内容，还被用于运行复杂的应用程序。例如，Gmail、Google GSuite 应用程序（文档、幻灯片等）、图像编辑器如 Pixlr 以及绘图应用程序如 draw.io 和 Canva。Atom 和微软的 Visual Studio Code 都是优秀的 IDE，它们恰好是用 Node.js 和 Chrome 中的 Electron 实现的。这些应用程序的存在以及大量人群的愉快使用是对 V8 性能的证明。Node.js 从 V8 的性能改进中受益。

正常的应用服务器模型使用阻塞 I/O 来检索数据，并使用线程来实现并发。阻塞 I/O 会导致线程在等待结果时停滞。这导致在应用服务器启动和停止线程以处理请求时，线程之间产生波动。每个挂起的线程（通常在等待 I/O 操作完成）会消耗完整的堆栈跟踪内存，从而增加内存消耗开销。线程不仅增加了应用服务器的复杂性，还增加了服务器开销。

Node.js 有一个单独的执行线程，没有等待 I/O 或上下文切换。相反，有一个事件循环在寻找事件并将它们分派给处理函数。范式是任何会阻塞或需要时间来完成操作的任何操作都必须使用异步模型。这些函数应该提供一个匿名函数作为处理回调，或者（随着 ES2015 promises 的出现），函数会返回一个 Promise。处理函数或 Promise 在操作完成时被调用。在此期间，控制权返回到事件循环，它继续分派事件。

在 2017 年的 Node.js 交互式会议上，IBM 的 Chris Bailey 提出了 Node.js 是高度可扩展微服务的优秀选择。关键性能特征是 I/O 性能，以每秒事务数衡量，启动时间，因为这限制了你的服务可以多快地扩展以满足需求，以及内存占用，因为这决定了每个服务器可以部署多少个应用程序实例。Node.js 在这些方面都表现出色；随着每个后续版本的发布，它要么在改进，要么保持相当稳定。Bailey 展示了将 Node.js 与用 Spring Boot 编写的类似基准进行比较的数字，表明 Node.js 的表现要好得多。要观看他的演讲，请参阅[`www.youtube.com/watch?v=Fbhhc4jtGW4`](https://www.youtube.com/watch?v=Fbhhc4jtGW4)。

为了帮助我们理解为什么会这样，让我们回到 Node.js 的创造者 Ryan Dahl，以及激发他创造 Node.js 的关键灵感。在 2010 年 5 月的*Cinco de NodeJS*演讲中，[`www.youtube.com/watch?v=M-sc73Y-zQA`](https://www.youtube.com/watch?v=M-sc73Y-zQA)，Dahl 询问我们在执行类似以下代码的行时会发生什么：

```js
result = query('SELECT * from db'); 
// operate on the result 
```

当然，程序会在那个点暂停，因为数据库层将查询发送到数据库，数据库确定结果并返回数据。根据查询的不同，这个暂停可能相当长；好吧，几毫秒，这在计算机时间中是一个漫长的时代。这个暂停很不好，因为执行线程在等待结果到来时什么都不能做。如果你的软件运行在单线程平台上，整个服务器将会被阻塞并且无响应。如果相反，你的应用程序运行在基于线程的服务器平台上，则需要线程上下文切换来满足任何到达的其他请求。服务器上未完成的连接数越多，线程上下文切换的次数就越多。上下文切换不是免费的，因为更多的线程需要更多的内存来存储每个线程的状态，以及 CPU 在线程管理开销上花费更多的时间。

仅使用异步的事件驱动 I/O，Node.js 就消除了大部分这种开销，同时引入了很少的自身开销。

使用线程来实现并发通常伴随着这样的警告：*昂贵且容易出错*、*Java 的错误易发同步原语*，或者*设计并发软件可能很复杂且容易出错*。复杂性来自于对共享变量的访问以及避免线程死锁和竞争的各种策略。Java 的*同步原语*是这种策略的一个例子，显然许多程序员发现它们很难使用。有创建框架如`java.util.concurrent`的倾向，以驯服线程并发的复杂性，但有些人可能会认为掩盖复杂性并不会使事情变得更简单。

Node.js 要求我们以不同的方式思考并发。从事件循环异步触发的回调是一个更简单的并发模型——更容易理解、更容易实现、更容易推理、更容易调试和维护。

Ryan Dahl 指出，对象相对访问时间可以理解异步 I/O 的需求。内存中的对象访问速度更快（以纳秒计），比磁盘上的对象或通过网络检索的对象（毫秒或秒）快。外部对象的较长时间访问以数百万个时钟周期来衡量，当你的客户坐在他们的网络浏览器上，如果页面加载时间超过两秒，他们可能就会离开。

在 Node.js 中，之前讨论的查询将如下所示：

```js
query('SELECT * from db', function (err, result) { 
    if (err) throw err; // handle errors 
    // operate on result 
}); 
```

程序员提供一个函数，当结果（或错误）可用时会被调用（因此得名*回调函数*）。而不是线程上下文切换，此代码几乎立即返回到事件循环。该事件循环可以自由处理其他请求。Node.js 运行时跟踪导致此回调函数的堆栈上下文，最终某个事件会触发，导致此回调函数被调用。

JavaScript 语言的进步为我们提供了实现这个想法的新选项。当使用 ES2015 Promise 时，等效代码看起来是这样的：

```js
query('SELECT * from db') 
.then(result => { 
    // operate on result 
}) 
.catch(err => { 
    // handle errors 
}); 
```

以下是一个使用 ES-2017 `async`函数的例子：

```js
try {
    var result = await query('SELECT * from db');
    // operate on result
} catch (err) {
    // handle errors
}
```

这三个代码片段执行的是之前写过的相同查询。区别在于查询不会阻塞执行线程，因为控制权返回到事件循环。通过几乎立即返回到事件循环，它可以自由地服务其他请求。最终，那些事件中的一个将是之前显示的查询的响应，这将调用回调函数。

使用回调或 Promise 方法，`result`不是作为函数调用的结果返回，而是提供给稍后将被调用的回调函数。执行顺序不是一行接一行，就像在同步编程语言中那样。相反，执行顺序由回调函数的执行顺序决定。

当使用`async`函数时，编码风格看起来就像原始的同步代码示例。`result`作为函数调用的结果返回，错误使用`try/catch`以自然的方式处理。`await`关键字集成了异步结果处理，而不会阻塞执行线程。`async/await`功能之下隐藏了很多东西，我们将在整本书中广泛地介绍这个模型。

通常，网页会从数十个来源收集数据。每个来源都有一个前面讨论过的查询和响应。使用异步查询，每个查询可以并行发生，其中页面构建函数可以触发数十个查询——无需等待，每个都有自己的回调——然后返回到事件循环，在完成每个查询时调用回调。因为它是并行的，所以数据可以比如果这些查询是同步地一个接一个地完成收集得更快。现在，网页浏览器的读者会更高兴，因为页面加载得更快。

# 性能和利用率

对 Node.js 的一些兴奋之处归因于其吞吐量（它每秒可以服务的请求数）。例如，与 Apache 等类似应用的比较基准显示，Node.js 有巨大的性能提升。

流传的一个基准是一个简单的 HTTP 服务器（借鉴自[`nodejs.org/en/`](https://nodejs.org/en/))，它直接从内存中返回一个`Hello World`消息：

```js
var http = require('http'); 
http.createServer(function (req, res) { 
  res.writeHead(200, {'Content-Type': 'text/plain'}); 
  res.end('Hello World\n'); 
}).listen(8124, "127.0.0.1"); 
console.log('Server running at http://127.0.0.1:8124/'); 
```

这是在 Node.js 中可以构建的较简单的 Web 服务器之一。`http`对象封装了 HTTP 协议，它的`http.createServer`方法创建了一个完整的 Web 服务器，监听`listen`方法中指定的端口。该 Web 服务器上的每个请求（无论是任何 URL 上的`GET`或`POST`）都会调用提供的函数。它非常简单且轻量。在这种情况下，无论 URL 如何，它都返回一个简单的`text/plain`，即`Hello World`响应。

Ryan Dahl 展示了一个简单的基准测试 ([`www.youtube.com/watch?v=M-sc73Y-zQA`](https://www.youtube.com/watch?v=M-sc73Y-zQA))，返回了一个 1 兆字节的二进制缓冲区；Node.js 提供了 822 req/sec，而 Nginx 提供了 708 req/sec，比 Nginx 提高了 15%。他还指出，Nginx 的内存峰值达到四兆字节，而 Node.js 的内存峰值达到 64 兆字节。

关键观察结果是，Node.js，运行一个解释的 JIT 编译的高级语言，在执行类似任务时与由高度优化的 C 代码构建的 Nginx 大约一样快。那次演示是在 2010 年 5 月，自那时以来，Node.js 已经取得了巨大的进步，正如我们之前提到的 Chris Bailey 的演讲中所示。

Yahoo! 的搜索工程师 Fabian Frank 发布了一篇关于使用 Apache/PHP 和两种 Node.js 栈变体实现的现实世界搜索查询建议小部件的性能案例研究 ([`www.slideshare.net/FabianFrankDe/nodejs-performance-case-study`](http://www.slideshare.net/FabianFrankDe/nodejs-performance-case-study))。该应用程序是一个弹出面板，当用户输入短语时显示搜索建议，使用基于 JSON 的 HTTP 查询。Node.js 版本能够处理每秒八倍数量的请求，同时保持相同的请求延迟。Fabian Frank 表示，这两个 Node.js 栈线性扩展，直到 CPU 使用率达到 100%。在另一个演示 ([`www.slideshare.net/FabianFrankDe/yahoo-scale-nodejs`](http://www.slideshare.net/FabianFrankDe/yahoo-scale-nodejs)) 中，他讨论了 Yahoo! Axis 是如何运行在 Manhattan + Mojito 上，以及能够在前后端都使用相同的语言（JavaScript）和框架（YUI/YQL）的价值。

LinkedIn 使用 Node.js 对其移动应用进行了大规模的重构，用 Node.js 替换了旧的 Ruby on Rails 应用程序，用于服务器端。这次切换使他们从 30 台服务器减少到三台，并且由于所有内容都是用 JavaScript 编写的，他们得以合并前端和后端团队。在选择 Node.js 之前，他们评估了带有 Event Machine 的 Rails、带有 Twisted 的 Python 以及 Node.js，并基于我们刚才讨论的原因选择了 Node.js。要了解 LinkedIn 所做的工作，请参阅 [`arstechnica.com/information-technology/2012/10/a-behind-the-scenes-look-at-linkedins-mobile-engineering/`](http://arstechnica.com/information-technology/2012/10/a-behind-the-scenes-look-at-linkedins-mobile-engineering/)。

大多数关于 Node.js 性能建议的现有内容往往是为较老的 V8 版本所写，这些版本使用了 CrankShaft 优化器。V8 团队已经完全放弃了 CrankShaft，并引入了一个名为 TurboFan 的新优化器。例如，在 CrankShaft 下，使用`try/catch`、`let/const`、生成器函数等会变慢。因此，普遍的智慧是不要使用这些特性，这令人沮丧，因为我们想使用新的 JavaScript 特性，因为它们极大地改进了 JavaScript 语言。谷歌 V8 团队的一名工程师 Peter Marshall 在 Node.js Interactive 2017 上发表了一次演讲，声称在 TurboFan 下，你应该只编写自然的 JavaScript。使用 TurboFan 的目标是在 V8 中实现全面性能提升。要查看演讲，请参阅[`www.youtube.com/watch?v=YqOhBezMx1o`](https://www.youtube.com/watch?v=YqOhBezMx1o)。

关于 JavaScript 的一个普遍真理是它不适合重计算工作，因为 JavaScript 的本质。我们将在下一节中讨论一些与此相关的内容。Mikola Lysenko 在 Node.js Interactive 2016 的一次演讲中讨论了 JavaScript 中数值计算的一些问题以及一些可能的解决方案。常见的数值计算涉及通过数值算法处理的大数值数组，这些算法你可能已经在微积分或线性代数课程中学到。JavaScript 缺乏的是多维数组以及访问某些 CPU 指令。他提出的解决方案是一个库，用于在 JavaScript 中实现多维数组，以及另一个包含大量数值计算算法的库。要查看演讲，请参阅[`www.youtube.com/watch?v=1ORaKEzlnys`](https://www.youtube.com/watch?v=1ORaKEzlnys)。

事实上，Node.js 在事件驱动 I/O 吞吐量方面表现卓越。一个 Node.js 程序是否能在计算程序方面表现出色，取决于你在绕过 JavaScript 语言某些限制方面的独创性。计算编程的一个大问题是它会阻止事件循环执行，正如我们将在下一节中看到的，这可能会使 Node.js 看起来不适合任何东西。

# Node.js 是不是一个有害的可扩展性灾难？

在 2011 年 10 月，软件开发者和博主 Ted Dziuba 撰写了一篇博客文章（后来从他的博客中撤下），标题为*Node.js is a cancer*，称其为*可扩展性灾难*。他用来证明的例子是一个 CPU 密集型的斐波那契数列算法实现。尽管他的论点存在缺陷，但他提出了一个有效的观点，即 Node.js 应用程序开发者必须考虑以下问题：你将把重计算任务放在哪里？

维护 Node.js 应用程序高吞吐量的关键是确保事件处理得快。因为它使用单个执行线程，如果这个线程被大计算拖累，Node.js 就无法处理事件，事件吞吐量将受到影响。

斐波那契数列，作为重型计算任务的替代品，计算起来很快就会变得非常昂贵，尤其是对于像这样的天真实现：

```js
const fibonacci = exports.fibonacci = function(n) { 
    if (n === 1 || n === 2) return 1; 
    else return fibonacci(n-1) + fibonacci(n-2); 
}
```

是的，有许多方法可以更快地计算斐波那契数。我们展示这个例子是为了说明当事件处理器变慢时 Node.js 会发生什么，而不是为了讨论计算数学函数的最佳方法。考虑以下服务器：

```js
const http = require('http'); 
const url  = require('url'); 

const fibonacci = // as above 

http.createServer(function (req, res) { 
  const urlP = url.parse(req.url, true); 
  let fibo; 
  res.writeHead(200, {'Content-Type': 'text/plain'}); 
  if (urlP.query['n']) { 
    fibo = fibonacci(urlP.query['n']); 
    res.end('Fibonacci '+ urlP.query['n'] +'='+ fibo); 
  } else { 
    res.end('USAGE: http://127.0.0.1:8124?n=## where ## is the Fibonacci number desired'); 
  } 
}).listen(8124, '127.0.0.1'); 
console.log('Server running at http://127.0.0.1:8124'); 
```

对于足够大的`n`值（例如，`40`），服务器会完全无响应，因为事件循环没有运行，而是这个函数因为正在缓慢地进行计算而阻塞了事件处理。

这是否意味着 Node.js 是一个有缺陷的平台？不，这只意味着程序员必须小心地识别出具有长时间运行的代码，并开发解决方案。这些包括将算法重写为与事件循环一起工作，或者为了效率重写算法，或者集成本地代码库，或者将计算密集型计算推到后端服务器上。

简单的重写通过事件循环调度计算，让服务器能够继续在事件循环上处理请求。使用回调和闭包（匿名函数），我们能够保持异步 I/O 和并发承诺：

```js
const fibonacciAsync = function(n, done) { 
    if (n === 0) return 0;
    else if (n === 1 || n === 2) done(1); 
    else if (n === 3) return 2;
    else { 
        process.nextTick(function() { 
            fibonacciAsync(n-1, function(val1) { 
                process.nextTick(function() { 
                    fibonacciAsync(n-2, function(val2) {
                    done(val1+val2); }); 
                }); 
            }); 
        }); 
    } 
} 
```

因为这是一个异步函数，所以需要对服务器进行一些小的重构：

```js
const http = require('http'); 
const url  = require('url'); 

const fibonacciAsync = // as above 

http.createServer(function (req, res) { 
  let urlP = url.parse(req.url, true);
  res.writeHead(200, {'Content-Type': 'text/plain'}); 
  if (urlP.query['n']) { 
    fibonacciAsync(urlP.query['n'], fibo => {
 res.end('Fibonacci '+ urlP.query['n'] +'='+ fibo);
 });
  } else { 
    res.end('USAGE: http://127.0.0.1:8124?n=## where ## is the Fibonacci number desired');
  }
}).listen(8124, '127.0.0.1'); console.log('Server running at http://127.0.0.1:8124');
```

Dziuba 在博客文章中并没有很好地表达他的有效观点，而且在文章之后的评论中也有所遗失。具体来说，尽管 Node.js 是一个优秀的 I/O 密集型应用程序平台，但它并不是计算密集型应用程序的好平台。

在本书的后面部分，我们将更深入地探讨这个例子。

# 服务器利用率、商业底线和绿色网络托管

追求最优效率（每秒处理更多请求）并不仅仅是为了从优化中获得的技术满足感。这还带来了真正的商业和环境效益。每秒处理更多请求，正如 Node.js 服务器所能做到的那样，意味着在购买大量服务器和仅购买少量服务器之间的区别。Node.js 有可能让您的组织用更少的资源做更多的事情。

大体上，你购买的服务器越多，成本就越高，这些服务器的环境影响也越大。有一个专门的领域是关于降低运行 Web 服务器设施的成本和环境影响的，而这个粗略的指导原则并没有做到这一点。目标相当明显——更少的服务器，更低的成本，通过利用更高效的软件来减少环境影响。

英特尔的白皮书，*通过服务器功率测量提高数据中心效率* ([`www.intel.com/content/dam/doc/white-paper/intel-it-data-center-efficiency-server-power-paper.pdf`](https://www.intel.com/content/dam/doc/white-paper/intel-it-data-center-efficiency-server-power-paper.pdf))，提供了一个理解效率和数据中心成本的客观框架。有许多因素，例如建筑、冷却系统和计算机系统设计。高效的建筑设计、高效的冷却系统以及高效的计算机系统（数据中心效率、数据中心密度和存储密度）可以降低成本和环境影响。但如果你部署了一个低效的软件栈，迫使你购买比拥有高效软件栈时更多的服务器，那么你可能会破坏这些收益。或者，你可以通过一个高效的软件栈来放大数据中心效率的收益，这个软件栈允许你减少所需服务器的数量。

这场关于高效软件栈的讨论并不仅仅是为了环保的利他目的。这是一个绿色环保能够帮助提升企业盈利能力的案例。

# 拥抱 JavaScript 语言的进步

过去的几年对 JavaScript 程序员来说是非常激动人心的。负责监督 ECMAScript 标准的 TC-39 委员会添加了许多新特性，其中一些是语法糖，但有几个特性推动了我们进入 JavaScript 编程的全新时代。仅 `async/await` 功能就承诺了我们摆脱所谓的回调地狱的方法，或者是我们发现自己嵌套回调的情况。这是一个如此重要的特性，以至于它应该促使我们对 Node.js 和 JavaScript 生态系统中的主流回调范式进行广泛的重新思考。

回顾几页之前的内容：

```js
query('SELECT * from db', function (err, result) { 
    if (err) throw err; // handle errors 
    // operate on result 
});
```

这是对 Ryan Dahl 的重要洞察，也是推动 Node.js 流行起来的原因。某些操作需要很长时间才能运行，例如数据库查询，不应与快速从内存中检索数据的操作同等对待。由于 JavaScript 语言的特性，Node.js 必须以不自然的方式表达这种异步编程结构。结果不会出现在下一行代码中，而是在这个回调函数中显示。此外，错误必须以不自然的方式在回调函数内部处理。

在 Node.js 中，约定回调函数的第一个参数是一个错误指示器，后续的参数是结果。这是一个有用的约定，你会在 Node.js 的各个领域找到它。然而，它使得处理结果和错误变得复杂，因为它们都落在了不方便的位置——那个回调函数。错误和结果的自然落点应该在代码的后续行。

我们在回调函数嵌套的每一层都进一步陷入**回调地狱**。第七层回调嵌套比第六层回调嵌套更复杂。为什么？如果不是其他原因，那至少是因为随着回调嵌套的深度增加，错误处理的特殊考虑变得日益复杂。

```js
var results = await query('SELECT * from db');
```

相反，ES2017 异步函数使我们回归到这种非常自然的编程意图的表达方式。结果和错误落在正确的位置，同时保留了使 Node.js 变得伟大的优秀事件驱动异步编程模型。我们将在本书的后面部分看到它是如何工作的。

TC-39 委员会为 JavaScript 添加了许多新特性，例如：

+   改进的类声明语法，使得对象继承和 getter/setter 函数变得非常自然。

+   一种新的模块格式，它在浏览器和 Node.js 中得到标准化。

+   新的字符串方法，例如模板字符串表示法。

+   集合和数组的新方法——例如，`map`/`reduce`/`filter` 操作。

+   `const` 关键字用于定义不可改变的变量，而 `let` 关键字用于定义作用域仅限于声明它们的代码块的变量，而不是提升到函数的前面。

+   新的循环结构，以及与这些新循环一起工作的迭代协议。

+   一种新的函数类型，箭头函数，它更轻量级，意味着更少的内存和执行时间影响

+   Promise 对象代表一个承诺在未来交付的结果。单独来看，Promise 可以减轻回调地狱问题，并且它们构成了 `async` 函数的基础部分。

+   生成器函数是表示对一组值进行异步迭代的一种有趣方式。更重要的是，它们构成了异步函数基础的一部分。

你可能会看到新的 JavaScript 被描述为 ES6 或 ES2017。那么，描述正在使用的 JavaScript 版本的首选名称是什么？

ES1 到 ES5 标记了 JavaScript 发展的各个阶段。ES5 于 2009 年发布，在现代浏览器中得到广泛实现。从 ES6 开始，TC-39 委员会决定改变命名约定，因为他们打算每年添加新的语言特性。因此，语言版本名称现在包括年份，所以 ES2015 于 2015 年发布，ES2016 于 2016 年发布，ES2017 于 2017 年发布。

# 部署 ES2015/2016/2017/2018 JavaScript 代码

室内那只粉红色的象意味着，由于 JavaScript 的交付方式，我们无法直接开始使用最新的 ES2017 特性。在前端 JavaScript 中，我们受限于旧浏览器仍在使用的事实。幸运的是，Internet Explorer 版本 6 几乎已经完全退役，但仍有大量旧浏览器安装在一些仍在为用户发挥有效作用的旧电脑上。旧浏览器意味着旧的 JavaScript 实现，如果我们想让我们的代码工作，我们需要它兼容旧浏览器。

使用代码重写工具，如 Babel，可以将一些新特性回滚到一些较旧的浏览器上。前端 JavaScript 程序员可以在付出更复杂的构建工具链和代码重写过程中引入的 bug 风险代价下采用（一些）新特性，有些人可能愿意这样做，而其他人则可能更愿意等待一段时间。

Node.js 世界没有这个问题。Node.js 已经迅速采用了 ES2015/2016/2017 特性，就像它们在 V8 引擎中实现的那样快。在 Node.js 8 中，我们现在可以使用异步函数作为一个原生特性，而 ES2015/2016 的大部分特性在 Node.js 版本 6 时就已经可用。新的模块格式现在在 Node.js 版本 10 中得到了支持。

换句话说，尽管前端 JavaScript 程序员可以争论他们必须等待几年才能采用 ES2015/2016/2017 特性，但 Node.js 程序员没有必要等待。我们可以简单地使用新特性，而不需要任何代码重写工具。

# Node.js、微服务架构和易于测试的系统

新的能力，如云部署系统和 Docker，使得实现一种新的服务架构成为可能。Docker 使得在可重复的容器中定义服务器进程配置成为可能，这种容器易于部署到云托管系统中，适用于小型单一用途服务实例，可以将它们连接起来形成一个完整的系统。Docker 不是唯一帮助简化云部署的工具；然而，其特性非常适合现代应用程序部署的需求。

有些人已经将微服务概念普及为描述这类系统的一种方式。根据[microservices.io](http://microservices.io/)网站，微服务由一组专注于特定领域、可独立部署的服务组成。他们将此与单体应用程序部署模式进行对比，在这种模式中，系统的每个方面都集成到一个捆绑包中（例如，Java EE 应用程序服务器的单个 WAR 文件）。微服务模型为开发者提供了急需的灵活性。

微服务的某些优势如下：

+   每个微服务都可以由一个小团队管理

+   每个团队都可以根据自己的时间表工作，只要保持服务 API 的兼容性

+   微服务可以独立部署，例如，为了更容易测试

+   更容易切换技术栈选择

Node.js 在其中扮演什么角色？其设计就像手套一样适合微服务模型：

+   Node.js 鼓励小型、紧密聚焦、单一用途的模块

+   这些模块通过优秀的 npm 包管理系统组合成一个应用程序

+   发布模块非常简单，无论是通过 NPM 仓库还是 Git URL

# Node.js 和十二要素应用模型

在本书中，我们将指出十二要素应用模型的一些方面，以及如何在 Node.js 中实现这些想法。这个模型发布在 [`12factor.net`](http://12factor.net)，是一套适用于现代云计算时代的应用部署指南。十二要素应用模型并不是应用架构范例的终极解决方案。它是一套有用的想法，显然是在许多深夜调试复杂应用之后产生的，这些想法通过更容易维护和更可靠的系统为我们所有人节省了大量精力。

指南简单明了，一旦你阅读了它们，它们就会显得像纯粹的常识。作为一个最佳实践，十二要素应用模型是我们当前计算环境所要求的流动自包含云部署应用的引人入胜的策略。

# 摘要

你在本章中学到了很多。具体来说，你看到了 JavaScript 在网络浏览器之外也有生命，并且你了解了异步和阻塞 I/O 之间的区别。然后我们介绍了 Node.js 的属性以及它在整体网络应用平台市场中的位置，以及线程与异步软件的比较。最后，我们看到了快速事件驱动异步 I/O 的优势，以及与支持匿名闭包的语言相结合的优势。

本书关注的是开发和部署 Node.js 应用程序的实际考虑因素。我们将尽可能涵盖开发、精炼、测试和部署 Node.js 应用程序的各个方面。

现在我们已经对 Node.js 有了一个介绍，我们准备深入学习和使用它。在第二章 *设置 Node.js* 中，我们将介绍如何设置 Node.js 环境，让我们开始吧。
