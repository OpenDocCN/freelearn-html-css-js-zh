关于 Node.js

JavaScript 是每个前端 Web 开发人员的得心应手，使其成为一种非常流行的编程语言，以至于被刻板地认为是用于 Web 页面中的客户端代码。有可能，拿起这本书的时候，你已经听说过 Node.js，这是一个用于在 Web 浏览器之外编写 JavaScript 代码的编程平台。现在大约有十年的历史，Node.js 正在成为一个成熟的编程平台，在大大小小的项目中被广泛使用。

本书将为您介绍 Node.js。通过本书，您将学习使用 Node.js 开发服务器端 Web 应用程序的完整生命周期，从概念到部署和安全性。在撰写本书时，我们假设以下内容：

+   你已经知道如何编写软件。

+   你熟悉 JavaScript。

+   你对其他语言中开发 Web 应用程序有所了解。

当我们评估一个新的编程工具时，我们是因为它是流行的新工具而抓住它吗？也许我们中的一些人会这样做，但成熟的方法是将一个工具与另一个工具进行比较。这就是本章的内容，介绍使用 Node.js 的技术基础。在着手编写代码之前，我们必须考虑 Node.js 是什么，以及它如何适应软件开发工具的整体市场。然后我们将立即着手开发工作应用程序，并认识到通常学习的最佳方式是通过在工作代码中进行搜索。

我们将在本章中涵盖以下主题：

+   Node.js 简介

+   Node.js 可以做什么

+   为什么你应该使用 Node.js

+   Node.js 的架构

+   使用 Node.js 的性能、利用率和可扩展性

+   Node.js、微服务架构和测试

+   使用 Node.js 实现十二要素应用程序模型

# 第三章：Node.js 概述

Node.js 是一个令人兴奋的新平台，用于开发 Web 应用程序、应用服务器、任何类型的网络服务器或客户端以及通用编程。它旨在通过服务器端 JavaScript、异步 I/O 和异步编程的巧妙组合，在网络应用程序中实现极端可扩展性。

尽管只有十年的历史，Node.js 迅速崭露头角，现在正发挥着重要作用。无论是大公司还是小公司，都在大规模和小规模项目中使用它。例如，PayPal 已经将许多服务从 Java 转换为 Node.js。

Node.js 的架构与其他应用平台通常选择的方式有所不同。在其他应用平台中，线程被广泛使用来扩展应用程序以填充 CPU，而 Node.js 则避免使用线程，因为线程具有固有的复杂性。据称，采用单线程事件驱动架构，内存占用低，吞吐量高，负载下的延迟配置文件更好，并且编程模型更简单。Node.js 平台正处于快速增长阶段，许多人认为它是传统的使用 Java、PHP、Python 或 Ruby on Rails 的 Web 应用程序架构的一个引人注目的替代方案。

在其核心，它是一个独立的 JavaScript 引擎，具有适用于通用编程的扩展，并且专注于应用服务器开发。尽管我们正在将 Node.js 与应用服务器平台进行比较，但它并不是一个应用服务器。相反，Node.js 是一个类似于 Python、Go 或 Java SE 的编程运行时。虽然有一些用 Node.js 编写的 Web 应用程序框架和应用服务器，但它只是一个执行 JavaScript 程序的系统。

关键的架构选择是 Node.js 是事件驱动的，而不是多线程的。Node.js 架构基于将阻塞操作分派到单线程事件循环，结果以调用事件处理程序的事件返回给调用者。在大多数情况下，事件被转换为由`async`函数处理的 promise。由于 Node.js 基于 Chrome 的 V8 JavaScript 引擎，Chrome 中实现的性能和功能改进很快就会流入 Node.js 平台。

Node.js 核心模块足够通用，可以实现执行任何 TCP 或 UDP 协议的服务器，无论是 DNS、HTTP、互联网中继聊天（IRC）还是 FTP。虽然它支持互联网服务器或客户端的开发，但它最大的用例是常规网站开发，取代了像 Apache/PHP 或 Rails 堆栈这样的技术，或者作为现有网站的补充，例如，使用 Node.js 的 Socket.IO 库可以轻松地添加实时聊天或监控现有网站。它的轻量级、高性能的特性经常被用作 Node.js 的“胶水”服务。

特别有趣的组合是在现代云基础设施上部署小型服务，使用诸如 Docker 和 Kubernetes 之类的工具，或者像 AWS Lambda 这样的函数即服务平台。将大型应用程序划分为易于部署的微服务时，Node.js 在规模上表现良好。

掌握了 Node.js 的高级理解后，让我们深入一点。

# Node.js 的能力

Node.js 是一个在 Web 浏览器之外编写 JavaScript 应用程序的平台。这不是我们在 Web 浏览器中熟悉的 JavaScript 环境！虽然 Node.js 执行与我们在浏览器中使用的相同的 JavaScript 语言，但它没有一些与浏览器相关的功能。例如，Node.js 中没有内置 HTML DOM。

除了其本身执行 JavaScript 的能力外，内置模块提供了以下类型的功能：

+   命令行工具（以 shell 脚本风格）

+   交互式终端风格的程序，即 REPL

+   优秀的进程控制功能来监督子进程

+   处理二进制数据的缓冲对象

+   TCP 或 UDP 套接字与全面的事件驱动回调

+   DNS 查找

+   HTTP、HTTPS 和 HTTP/2 客户端服务器在 TCP 库文件系统访问之上

+   内置的基本单元测试支持通过断言

Node.js 的网络层是低级的，同时使用起来很简单，例如，HTTP 模块允许您使用几行代码编写 HTTP 服务器（或客户端）。这很强大，但它让您，程序员，非常接近协议请求，并让您实现应该在请求响应中返回的那些 HTTP 头部。

典型的 Web 应用程序开发人员不需要在 HTTP 或其他协议的低级别上工作；相反，我们倾向于使用更高级别的接口更加高效，例如，PHP 程序员假设 Apache/Nginx 等已经提供了 HTTP，并且他们不必实现堆栈的 HTTP 服务器部分。相比之下，Node.js 程序员确实实现了一个 HTTP 服务器，他们的应用代码附加到其中。

为了简化情况，Node.js 社区有几个 Web 应用程序框架，比如 Express，提供了典型程序员所需的更高级别的接口。您可以快速配置一个具有内置功能的 HTTP 服务器，比如会话、cookie、提供静态文件和日志记录，让开发人员专注于业务逻辑。其他框架提供 OAuth 2 支持或专注于 REST API 等等。

使用 Node.js 的社区在这个基础上构建了各种令人惊叹的东西。

## 人们如何使用 Node.js？

Node.js 不仅限于 Web 服务应用程序开发；Node.js 周围的社区已经将其引向了许多其他方向：

+   构建工具：Node.js 已经成为开发命令行工具的热门选择，这些工具用于软件开发或与服务基础设施通信。Grunt、Gulp 和 Webpack 被广泛用于前端开发人员构建网站资产。Babel 被广泛用于将现代 ES-2016 代码转译为在旧版浏览器上运行。流行的 CSS 优化器和处理器，如 PostCSS，都是用 Node.js 编写的。静态网站生成系统，如 Metalsmith、Punch 和 AkashaCMS，在命令行上运行，并生成您上传到 Web 服务器的网站内容。

+   Web UI 测试：Puppeteer 让您控制一个无头 Chrome 浏览器实例。借助它，您可以通过控制现代、功能齐全的 Web 浏览器来开发 Node.js 脚本。一些典型的用例是 Web 抓取和 Web 应用程序测试。

+   桌面应用程序：Electron 和 node-webkit（NW.js）都是用于开发 Windows、macOS 和 Linux 桌面应用程序的框架。这些框架利用大量的 Chrome，由 Node.js 库包装，使用 Web UI 技术开发桌面应用程序。应用程序使用现代的 HTML5、CSS3 和 JavaScript 编写，并可以利用领先的 Web 框架，如 Bootstrap、React、VueJS 和 AngularJS。许多流行的应用程序都是使用 Electron 构建的，包括 Slack 桌面客户端应用程序、Atom、Microsoft Visual Code 编程编辑器、Postman REST 客户端、GitKraken GIT 客户端和 Etcher 等。

+   移动应用程序：Node.js for Mobile Systems 项目允许您使用 Node.js 开发 iOS 和 Android 的智能手机或平板电脑应用程序。苹果的 App Store 规定不允许将具有 JIT 功能的 JavaScript 引擎纳入其中，这意味着普通的 Node.js 不能在 iOS 应用程序中使用。对于 iOS 应用程序开发，该项目使用 Node.js-on-ChakraCore 来规避 App Store 规定。对于 Android 应用程序开发，该项目使用常规的 Node.js 在 Android 上运行。在撰写本文时，该项目处于早期开发阶段，但看起来很有前景。

+   物联网（IoT）：Node.js 是物联网项目中非常流行的语言，Node.js 可以在大多数基于 ARM 的单板计算机上运行。最明显的例子是 NodeRED 项目。它提供了一个图形化的编程环境，让您通过连接块来绘制程序。它具有面向硬件的输入和输出机制，例如与树莓派或 Beaglebone 单板计算机上的通用 I/O（GPIO）引脚进行交互。

您可能已经在使用 Node.js 应用程序而没有意识到！JavaScript 在 Web 浏览器之外也有用武之地，这不仅仅是因为 Node.js。

## 服务器端 JavaScript

别再挠头了！当然，您正在这样做，挠头并自言自语地说：“浏览器语言在服务器上做什么？”事实上，JavaScript 在浏览器之外有着悠久而鲜为人知的历史。JavaScript 是一种编程语言，就像任何其他语言一样，更好的问题是“为什么 JavaScript 应该被困在 Web 浏览器内部？”

回到网络时代的黎明，编写 Web 应用程序的工具处于萌芽阶段。一些开发人员尝试使用 Perl 或 TCL 编写 CGI 脚本，PHP 和 Java 语言刚刚被开发出来。即便那时，JavaScript 也在服务器端使用。早期的 Web 应用程序服务器之一是网景的 LiveWire 服务器，它使用了 JavaScript。微软的 ASP 的一些版本使用了 JScript，他们的 JavaScript 版本。一个更近期的服务器端 JavaScript 项目是 Java 领域的 RingoJS 应用程序框架。Java 6 和 Java 7 都附带了 Rhino JavaScript 引擎。在 Java 8 中，Rhino 被新的 Nashorn JavaScript 引擎所取代。

换句话说，JavaScript 在浏览器之外并不是一件新事物，尽管它并不常见。

您已经了解到 Node.js 是一个用于在 Web 浏览器之外编写 JavaScript 应用程序的平台。Node.js 社区使用这个平台进行各种类型的应用程序开发，远远超出了最初为该平台构思的范围。这证明了 Node.js 的受欢迎程度，但我们仍然必须考虑使用它的技术原因。

# 为什么要使用 Node.js？

在众多可用的 Web 应用程序开发平台中，为什么应该选择 Node.js？有很多选择，那么 Node.js 有什么特点使其脱颖而出呢？我们将在接下来的部分中找到答案。

## 流行度

Node.js 迅速成为一种受欢迎的开发平台，并被许多大大小小的参与者所采用。其中之一是 PayPal，他们正在用 Node.js 替换其现有的基于 Java 的系统。其他大型 Node.js 采用者包括沃尔玛的在线电子商务平台、LinkedIn 和 eBay。

有关 PayPal 关于此的博客文章，请访问[`www.paypal-engineering.com/2013/11/22/node-js-at-paypal/`](https://www.paypal-engineering.com/2013/11/22/node-js-at-paypal/)。

根据 NodeSource 的说法，Node.js 的使用量正在迅速增长（有关更多信息，请访问[`nodesource.com/node-by-numbers`](https://nodesource.com/node-by-numbers)）。这种增长的证据包括下载 Node.js 版本的带宽增加，与 Node.js 相关的 GitHub 项目的活动增加等。

对 JavaScript 本身的兴趣仍然非常强烈，但在搜索量（Google Insights）和作为编程技能的使用方面（Dice Skills Center）已经停滞多年。Node.js 的兴趣一直在迅速增长，但正在显示出停滞的迹象。

有关更多信息，请参阅[`itnext.io/choosing-typescript-vs-javascript-technology-popularity-ea978afd6b5f`](https://itnext.io/choosing-typescript-vs-javascript-technology-popularity-ea978afd6b5f)或[`bit.ly/2q5cu0w`](http://bit.ly/2q5cu0w)。

最好不要只是跟随潮流，因为有不同的潮流，每一个都声称他们的软件平台有很酷的功能。Node.js 确实有一些很酷的功能，但更重要的是它的技术价值。

## JavaScript 无处不在

在服务器和客户端上使用相同的编程语言一直是网络上的一个长期梦想。这个梦想可以追溯到早期的 Java 时代，当时 Java 小程序在浏览器中被视为用于 Java 编写的服务器应用程序的前端，而 JavaScript 最初被设想为这些小程序的轻量级脚本语言。然而，Java 从未实现其作为客户端编程语言的炒作，甚至“Java 小程序”这个词组也正在逐渐消失，成为被放弃的客户端应用程序模型的模糊记忆。最终，我们选择了 JavaScript 作为浏览器中的主要客户端语言，而不是 Java。通常情况下，前端 JavaScript 开发人员使用的是与服务器端团队不同的语言，后者可能是 PHP、Java、Ruby 或 Python。

随着时间的推移，在浏览器中的 JavaScript 引擎变得非常强大，让我们能够编写越来越复杂的浏览器端应用程序。有了 Node.js，我们终于能够使用相同的编程语言在客户端和服务器上实现应用程序，因为 JavaScript 在网络的两端，即浏览器和服务器上。

前端和后端使用相同的编程语言具有几个潜在的好处：

+   同一编程人员可以在网络两端工作。

+   代码可以更轻松地在服务器和客户端之间迁移。

+   服务器和客户端之间的常见数据格式（JSON）。

+   服务器和客户端存在常见的软件工具。

+   服务器和客户端的常见测试或质量报告工具。

+   在编写 Web 应用程序时，视图模板可以在两端使用。

JavaScript 语言非常受欢迎，因为它在 Web 浏览器中非常普遍。它与其他语言相比具有许多现代、先进的语言概念。由于其受欢迎程度，有许多经验丰富的 JavaScript 程序员。

## 利用谷歌对 V8 的投资

为了使 Chrome 成为一款受欢迎且出色的 Web 浏览器，谷歌投资于使 V8 成为一个超快的 JavaScript 引擎。因此，谷歌有巨大的动力继续改进 V8。V8 是 Chrome 的 JavaScript 引擎，也可以独立执行。

Node.js 建立在 V8 JavaScript 引擎之上，使其能够利用 V8 的所有工作。因此，Node.js 能够在 V8 实现新的 JavaScript 语言特性时迅速采用，并因此获得性能优势。

## 更精简、异步、事件驱动的模型

Node.js 架构建立在单个执行线程上，具有巧妙的事件驱动、异步编程模型和快速的 JavaScript 引擎，据称比基于线程的架构具有更少的开销。其他使用线程进行并发的系统往往具有内存开销和复杂性，而 Node.js 没有。我们稍后会更深入地讨论这一点。

## 微服务架构

软件开发中的一个新感觉是微服务的概念。微服务专注于将大型 Web 应用程序拆分为小型、紧密专注的服务，可以由小团队轻松开发。虽然它们并不是一个全新的想法，它们更像是对旧的客户端-服务器计算模型的重新构架，但是微服务模式与敏捷项目管理技术很匹配，并且为我们提供了更精细的应用部署。

Node.js 是实现微服务的优秀平台。我们稍后会详细介绍。

## Node.js 在一次重大分裂和敌对分支之后变得更加强大

在 2014 年和 2015 年，Node.js 社区因政策、方向和控制而发生了重大分裂。**io.js**项目是一个敌对的分支，由一群人驱动，他们希望合并几个功能并改变决策过程中的人员。最终的结果是合并了 Node.js 和 io.js 存储库，成立了独立的 Node.js 基金会来运作，并且社区共同努力朝着共同的方向前进。

弥合这一分歧的一个具体结果是快速采用新的 ECMAScript 语言特性。V8 引擎迅速采用这些新特性来推进 Web 开发的状态。Node.js 团队也在 V8 中尽快采用这些特性，这意味着承诺和`async`函数很快就会成为 Node.js 程序员的现实。

总之，Node.js 社区不仅在 io.js 分支和后来的 ayo.js 分支中幸存下来，而且社区和它培育的平台因此变得更加强大。

在本节中，您已经了解了使用 Node.js 的几个原因。它不仅是一个受欢迎的平台，有一个强大的社区支持，而且还有一些严肃的技术原因可以使用它。它的架构具有一些关键的技术优势，让我们更深入地了解一下这些优势。

# Node.js 事件驱动架构

据说 Node.js 的出色性能是因为其异步事件驱动架构和使用 V8 JavaScript 引擎。这使其能够同时处理多个任务，例如在多个 Web 浏览器的请求之间进行协调。Node.js 的原始创始人 Ryan Dahl 遵循了这些关键点：

+   单线程、事件驱动的编程模型比依赖线程处理多个并发任务的应用服务器更简单，复杂性更低，开销更小。

+   通过将阻塞函数调用转换为异步代码执行，可以配置系统以在满足阻塞请求时发出事件。

+   您可以利用来自 Chrome 浏览器的 V8 JavaScript 引擎，并且所有工作都用于改进 V8；所有性能增强都进入 V8，因此也有益于 Node.js。

在大多数应用服务器中，并发或处理多个并发请求的能力是通过多线程架构实现的。在这样的系统中，对数据的任何请求或任何其他阻塞函数调用都会导致当前执行线程暂停并等待结果。处理并发请求需要有多个执行线程。当一个线程被暂停时，另一个线程可以执行。这会导致应用服务器启动和停止线程来处理请求。每个暂停的线程（通常在输入/输出操作完成时等待）都会消耗完整的内存调用堆栈，增加开销。线程会给应用服务器增加复杂性和服务器开销。

为了帮助我们理解为什么会这样，Node.js 的创始人 Ryan Dahl 在 2010 年 5 月的 Cinco de NodeJS 演示中提供了以下示例。([`www.youtube.com/watch?v=M-sc73Y-zQA`](https://www.youtube.com/watch?v=M-sc73Y-zQA)) Dahl 问我们当我们执行这样的代码行时会发生什么：

```

Of course, the program pauses at this point while the database layer sends the query to the database and waits for the result or the error. This is an example of a blocking function call. Depending on the query, this pause can be quite long (well, a few milliseconds, which is ages in computer time). This pause is bad because the execution thread can do nothing while it waits for the result to arrive. If your software is running on a single-threaded platform, the entire server would be blocked and unresponsive. If instead your application is running on a thread-based server platform, a thread-context switch is required to satisfy any other requests that arrive. The greater the number of outstanding connections to the server, the greater the number of thread-context switches. Context switching is not free because more threads require more memory per thread state and more time for the CPU to spend on thread management overheads.

The key inspiration guiding the original development of Node.js was the simplicity of a single-threaded system. A single execution thread means that the server doesn't have the complexity of multithreaded systems. This choice meant that Node.js required an event-driven model for handling concurrent tasks. Instead of the code waiting for results from a blocking request, such as retrieving data from a database, an event is instead dispatched to an event handler.

Using threads to implement concurrency often comes with admonitions, such as *expensive and error-prone*, *the error-prone synchronization primitives of Java*, or *designing concurrent software can be complex and error-prone*. The complexity comes from access to shared variables and various strategies to avoid deadlock and competition between threads. The *synchronization primitives of Java* are an example of such a strategy, and obviously many programmers find them difficult to use. There's a tendency to create frameworks such as `java.util.concurrent` to tame the complexity of threaded concurrency, but some argue that papering over complexity only makes things more complex. 

A typical Java programmer might object at this point. Perhaps their application code is written against a framework such as Spring, or maybe they're directly using Java EE. In either case, their application code does not use concurrency features or deal with threads, and therefore where is the complexity that we just described? Just because that complexity is hidden within Spring and Java EE does not mean that there is no complexity and overhead.

Okay, we get it: while multithreaded systems can do amazing things, there is inherent complexity. What does Node.js offer?

## The Node.js answer to complexity

Node.js asks us to think differently about concurrency. Callbacks fired asynchronously from an event loop are a much simpler concurrency model—simpler to understand, simpler to implement, simpler to reason about, and simpler to debug and maintain. 

Node.js has a single execution thread with no waiting on I/O or context switching. Instead, there is an event loop that dispatches events to handler functions as things happen. A request that would have blocked the execution thread instead executes asynchronously, with the results or errors triggering an event. Any operation that would block or otherwise take time to complete must use the asynchronous model. 

The original Node.js paradigm delivered the dispatched event to an anonymous function. Now that JavaScript has `async` functions, the Node.js paradigm is shifting to deliver results and errors via a promise that is handled by the `await` keyword. When an asynchronous function is called, control quickly passes to the event loop rather than causing Node.js to block. The event loop continues handling the variety of events while recording where to send each result or error.

By using an asynchronous event-driven I/O, Node.js removes most of this overhead while introducing very little of its own.

One of the points Ryan Dahl made in the Cinco de Node presentation is a hierarchy of execution time for different requests. Objects in memory are more quickly accessed (in the order of nanoseconds) than objects on disk or objects retrieved over the network (milliseconds or seconds). The longer access time for external objects is measured in zillions of clock cycles, which can be an eternity when your customer is sitting at their web browser ready to move on if it takes longer than two seconds to load the page. 

Therefore, concurrent request handling means using a strategy to handle the requests that take longer to satisfy. If the goal is to avoid the complexity of a multithreaded system, then the system must use asynchronous operations as Node.js does.

What do these asynchronous function calls look like?

## Asynchronous requests in Node.js

In Node.js, the query that we looked at previously will read as follows:

```

程序员提供一个在结果（或错误）可用时被调用的函数（因此称为*回调函数*）。`query`函数仍然需要相同的时间。它不会阻塞执行线程，而是返回到事件循环，然后可以处理其他请求。Node.js 最终会触发一个事件，导致调用此回调函数并返回结果或错误指示。

在客户端 JavaScript 中使用类似的范例，我们经常编写事件处理程序函数。

JavaScript 语言的进步为我们提供了新的选择。与 ES2015 promises 一起使用时，等效的代码如下：

```

This is a little better, especially in instances of deeply nested event handling.

The big advance came with the ES-2017 `async` function:

```

除了`async`和`await`关键字之外，这看起来像我们在其他语言中编写的代码，并且更容易阅读。由于`await`的作用，它仍然是异步代码执行。

这三个代码片段都执行了我们之前编写的相同查询。`query`不再是阻塞函数调用，而是异步的，不会阻塞执行线程。

使用回调函数和 promise 的异步编码，Node.js 也存在自己的复杂性问题。我们经常在一个异步函数之后调用另一个异步函数。使用回调函数意味着深度嵌套的回调函数，而使用 promise 则意味着长长的`.then`处理程序函数链。除了编码的复杂性，我们还有错误和结果出现在不自然的位置。异步执行的回调函数被调用时，不会落在下一行代码上。执行顺序不是像同步编程语言中一行接一行的，而是由回调函数执行的顺序决定的。

`async`函数的方法解决了这种编码复杂性。编码风格更自然，因为结果和错误出现在自然的位置，即下一行代码。`await`关键字集成了异步结果处理，而不会阻塞执行线程。`async/await`功能的背后有很多东西，我们将在本书中广泛涵盖这个模型。

但是 Node.js 的异步架构实际上改善了性能吗？

## 性能和利用率

Node.js 引起了一些兴奋是因为它的吞吐量（每秒请求量）。对比类似应用的基准测试，比如 Apache，显示出 Node.js 有巨大的性能提升。

一个流传的基准是以下简单的 HTTP 服务器（从[`nodejs.org/en/`](https://nodejs.org/en/)借来的），它直接从内存中返回一个`Hello World`消息：

```

This is one of the simpler web servers that you can build with Node.js. The `http` object encapsulates the HTTP protocol, and its `http.createServer` method creates a whole web server, listening on the port specified in the `listen` method. Every request (whether a `GET` or `POST` on any URL) on that web server calls the provided function. It is very simple and lightweight. In this case, regardless of the URL, it returns a simple `text/plain` that is the `Hello World` response.

Ryan Dahl showed a simple benchmark in a video titled *Ryan Dahl: Introduction to Node.js* (on the YUI Library channel on YouTube, [`www.youtube.com/watch?v=M-sc73Y-zQA`](https://www.youtube.com/watch?v=M-sc73Y-zQA)). It used a similar HTTP server to this, but that returned a one-megabyte binary buffer; Node.js gave 822 req/sec, while Nginx gave 708 req/sec, for a 15% improvement over Nginx. He also noted that Nginx peaked at four megabytes of memory, while Node.js peaked at 64 megabytes. 

The key observation was that Node.js, running an interpreted, JIT-compiled, high-level language, was about as fast as Nginx, built of highly optimized C code, while running similar tasks. That presentation was in May 2010, and Node.js has improved hugely since then, as shown in Chris Bailey's talk that we referenced earlier.

Yahoo! search engineer Fabian Frank published a performance case study of a real-world search query suggestion widget implemented with Apache/PHP and two variants of Node.js stacks ([`www.slideshare.net/FabianFrankDe/nodejs-performance-case-study`](http://www.slideshare.net/FabianFrankDe/nodejs-performance-case-study)). The application is a pop-up panel showing search suggestions as the user types in phrases using a JSON-based HTTP query. The Node.js version could handle eight times the number of requests per second with the same request latency. Fabian Frank said both Node.js stacks scaled linearly until CPU usage hit 100%. 

LinkedIn did a massive overhaul of their mobile app using Node.js for the server-side to replace an old Ruby on Rails app. The switch lets them move from 30 servers down to 3, and allowed them to merge the frontend and backend team because everything was written in JavaScript. Before choosing Node.js, they'd evaluated Rails with Event Machine, Python with Twisted, and Node.js, chose Node.js for the reasons that we just discussed. For a look at what LinkedIn did, see [`arstechnica.com/information-technology/2012/10/a-behind-the-scenes-look-at-linkedins-mobile-engineering/`](http://arstechnica.com/information-technology/2012/10/a-behind-the-scenes-look-at-linkedins-mobile-engineering/).

Most existing Node.js performance tips tend to have been written for older V8 versions that used the CrankShaft optimizer. The V8 team has completely dumped CrankShaft, and it has a new optimizer called TurboFan—for example, under CrankShaft, it was slower to use `try/catch`, `let/const`, generator functions, and so on. Therefore, common wisdom said to not use those features, which is depressing because we want to use the new JavaScript features because of how much it has improved the JavaScript language. Peter Marshall, an engineer on the V8 team at Google, gave a talk at Node.js Interactive 2017 claiming that, using TurboFan, you should just write natural JavaScript. With TurboFan, the goal is for across-the-board performance improvements in V8\. To view the presentation, see the video titled *High Performance JS* in V8 at [`www.youtube.com/watch?v=YqOhBezMx1o`](https://www.youtube.com/watch?v=YqOhBezMx1o).

A truism about JavaScript is that it's no good for heavy computation work because of the nature of JavaScript. We'll go over some ideas that are related to this in the next section. A talk by Mikola Lysenko at Node.js Interactive 2016 went over some issues with numerical computing in JavaScript, and some possible solutions. Common numerical computing involves large numerical arrays processed by numerical algorithms that you might have learned in calculus or linear algebra classes. What JavaScript lacks is multidimensional arrays and access to certain CPU instructions. The solution that he presented is a library to implement multidimensional arrays in JavaScript, along with another library full of numerical computing algorithms. To view the presentation, see the video titled *Numerical Computing in JavaScript* by Mikola Lysenko at [`www.youtube.com/watch?v=1ORaKEzlnys`](https://www.youtube.com/watch?v=1ORaKEzlnys)[. ](https://www.youtube.com/watch?v=1ORaKEzlnys)

At the Node.js Interactive conference in 2017, IBM's Chris Bailey made a case for Node.js being an excellent choice for highly scalable microservices. Key performance characteristics are I/O performance (measured in transactions per second), startup time (because that limits how quickly your service can scale up to meet demand), and memory footprint (because that determines how many application instances can be deployed per server). Node.js excels on all those measures; with every subsequent release, it either improves on each measure or remains fairly steady. Bailey presented figures comparing Node.js to a similar benchmark written in Spring Boot showing Node.js to perform much better. To view his talk, see the video titled *Node.js Performance and Highly Scalable Micro-Services - Chris Bailey, IBM* at [`www.youtube.com/watch?v=Fbhhc4jtGW4`](https://www.youtube.com/watch?v=Fbhhc4jtGW4).

The bottom line is that Node.js excels at event-driven I/O throughput. Whether a Node.js program can excel at computational programs depends on your ingenuity in working around some limitations in the JavaScript language.

A big problem with computational programming is that it prevents the event loop from executing. As we will see in the next section, that can make Node.js look like a poor candidate for anything.

### Is Node.js a cancerous scalability disaster?

In October 2011, a blog post (since pulled from the blog where it was published) titled *Node.js is a cancer* called Node.js a scalability disaster. The example shown for proof was a CPU-bound implementation of the Fibonacci sequence algorithm. While the argument was flawed—since nobody implements Fibonacci that way—it made the valid point that Node.js application developers have to consider the following: where do you put the heavy computational tasks?

A key to maintaining high throughput of Node.js applications is by ensuring that events are handled quickly. Because it uses a single execution thread, if that thread is bogged down with a big calculation, Node.js cannot handle events, and event throughput will suffer.

The Fibonacci sequence, serving as a stand-in for heavy computational tasks, quickly becomes computationally expensive to calculate for a naïve implementation such as this:

```

这是一个特别简单的方法来计算斐波那契数。是的，有很多更快的计算斐波那契数的方法。我们展示这个作为 Node.js 在事件处理程序缓慢时会发生什么的一个一般性例子，而不是讨论计算数学函数的最佳方法。考虑以下服务器：

```

This is an extension of the simple web server shown earlier. It looks in the request URL for an argument, `n`, for which to calculate the Fibonacci number. When it's calculated, the result is returned to the caller.

For sufficiently large values of `n` (for example, `40`), the server becomes completely unresponsive because the event loop is not running. Instead, this function has blocked event processing because the event loop cannot dispatch events while the function is grinding through the calculation.

In other words, the Fibonacci function is a stand-in for any blocking operation.

Does this mean that Node.js is a flawed platform? No, it just means that the programmer must take care to identify code with long-running computations and develop solutions. These include rewriting the algorithm to work with the event loop, rewriting the algorithm for efficiency, integrating a native code library, or foisting computationally expensive calculations to a backend server.

A simple rewrite dispatches the computations through the event loop, letting the server continue to handle requests on the event loop. Using callbacks and closures (anonymous functions), we're able to maintain asynchronous I/O and concurrency promises, as shown in the following code:

```

这是一个同样愚蠢的计算斐波那契数的方法，但是通过使用`process.nextTick`，事件循环有机会执行。

因为这是一个需要回调函数的异步函数，它需要对服务器进行小的重构：

```

We've added a callback function to receive the result. In this case, the server is able to handle multiple Fibonacci number requests. But there is still a performance issue because of the inefficient algorithm.

Later in this book, we'll explore this example a little more deeply to explore alternative approaches.

In the meantime, we can discuss why it's important to use efficient software stacks.

## Server utilization, overhead costs, and environmental impact

The striving for optimal efficiency (handling more requests per second) is not just about the geeky satisfaction that comes from optimization. There are real business and environmental benefits. Handling more requests per second, as Node.js servers can do, means the difference between buying lots of servers and buying only a few servers. Node.js potentially lets your organization do more with less.

Roughly speaking, the more servers you buy, the greater the monetary cost and the greater the environmental cost. There's a whole field of expertise around reducing costs and the environmental impact of running web-server facilities to which that rough guideline doesn't do justice. The goal is fairly obvious—fewer servers, lower costs, and a lower environmental impact by using more efficient software.

Intel's paper, *Increasing Data Center Efficiency with Server Power Measurements* ([`www.intel.com/content/dam/doc/white-paper/intel-it-data-center-efficiency-server-power-paper.pdf`](https://www.intel.com/content/dam/doc/white-paper/intel-it-data-center-efficiency-server-power-paper.pdf)), gives an objective framework for understanding efficiency and data center costs. There are many factors, such as buildings, cooling systems, and computer system designs. Efficient building design, efficient cooling systems, and efficient computer systems (data center efficiency, data center density, and storage density) can lower costs and environmental impact. But you can destroy these gains by deploying an inefficient software stack, compelling you to buy more servers than you would if you had an efficient software stack. Alternatively, you can amplify gains from data center efficiency with an efficient software stack that lets you decrease the number of servers required.

This talk about efficient software stacks isn't just for altruistic environmental purposes. This is one of those cases where being green can help your business bottom line.

In this section, we have learned a lot about how Node.js architecture differs from other programming platforms. The choice to eschew threads to implement concurrency simplifies away the complexity and overhead that comes from using threads. This seems to have fulfilled the promise of being more efficient. Efficiency has a number of benefits to many aspects of a business.

# Embracing advances in the JavaScript language

The last couple of years have been an exciting time for JavaScript programmers. The TC-39 committee that oversees the ECMAScript standard has added many new features, some of which are syntactic sugar, but several of which have propelled us into a whole new era of JavaScript programming. By itself, the `async/await` feature promises us a way out of what's called callback fell, the situation that we find ourselves in when nesting callbacks within callbacks. It's such an important feature that it should necessitate a broad rethinking of the prevailing callback-oriented paradigm in Node.js and the rest of the JavaScript ecosystem.

A few pages ago, you saw this:

```

这是 Ryan Dahl 的一个重要洞察，也是推动 Node.js 流行的原因。某些操作需要很长时间才能运行，比如数据库查询，不应该和快速从内存中检索数据的操作一样对待。由于 JavaScript 语言的特性，Node.js 必须以一种不自然的方式表达这种异步编码结构。结果不会出现在下一行代码，而是出现在这个回调函数中。此外，错误必须以一种不自然的方式处理，出现在那个回调函数中。

在 Node.js 中的约定是回调函数的第一个参数是一个错误指示器，随后的参数是结果。这是一个有用的约定，你会在整个 Node.js 领域找到它；然而，它使得处理结果和错误变得复杂，因为两者都出现在一个不方便的位置——那个回调函数。错误和结果自然地应该出现在随后的代码行上。

随着每一层回调函数嵌套，我们陷入了回调地狱。第七层回调嵌套比第六层回调嵌套更复杂。为什么？至少有一点是因为随着回调的嵌套更深，错误处理的特殊考虑变得更加复杂。

但正如我们之前所看到的，这是在 Node.js 中编写异步代码的新首选方式：

```

相反，ES2017 的`async`函数使我们回到了这种非常自然的编程意图表达。结果和错误会在正确的位置上，同时保持了使 Node.js 变得伟大的出色的事件驱动的异步编程模型。我们将在本书的后面看到这是如何工作的。

TC-39 委员会为 JavaScript 添加了许多新功能，比如以下的：

+   改进的类声明语法，使对象继承和 getter/setter 函数非常自然。

+   一个在浏览器和 Node.js 中标准化的新模块格式。

+   字符串的新方法，比如模板字符串表示法。

+   集合和数组的新方法，例如`map`/`reduce`/`filter`的操作。

+   使用`const`关键字来定义不能被改变的变量，使用`let`关键字来定义变量的作用域仅限于它们声明的块，而不是被提升到函数的前面。

+   新的循环结构和与这些新循环配合使用的迭代协议。

+   一种新类型的函数，箭头函数，它更轻量，意味着更少的内存和执行时间影响。

+   `Promise`对象表示将来承诺交付的结果。单独使用，承诺可以缓解回调地狱问题，并且它们构成了`async`函数的一部分基础。

+   生成器函数是一种有趣的方式，用于表示一组值的异步迭代。更重要的是，它们构成了异步函数的基础的另一半。

你可能会看到新的 JavaScript 被描述为 ES6 或 ES2017。描述正在使用的 JavaScript 版本的首选名称是什么？

ES1 到 ES5 标志着 JavaScript 发展的各个阶段。ES5 于 2009 年发布，并在现代浏览器中得到广泛实现。从 ES6 开始，TC-39 委员会决定改变命名约定，因为他们打算每年添加新的语言特性。因此，语言版本现在包括年份，例如，ES2015 于 2015 年发布，ES2016 于 2016 年发布，ES2017 于 2017 年发布。

## 部署 ES2015/2016/2017/2018 JavaScript 代码

问题在于，通常 JavaScript 开发人员无法使用最新的功能。前端 JavaScript 开发人员受到部署的网络浏览器和大量旧浏览器的限制，这些浏览器在长时间未更新操作系统的计算机上使用。幸运的是，Internet Explorer 6 版本几乎已经完全退出使用，但仍然有大量旧浏览器安装在老旧计算机上，仍然为其所有者提供有效的角色。旧浏览器意味着旧的 JavaScript 实现，如果我们希望我们的代码能够运行，我们需要它与旧浏览器兼容。

Babel 和其他代码重写工具的一个用途是处理这个问题。许多产品必须能够被使用旧浏览器的人使用。开发人员仍然可以使用最新的 JavaScript 或 TypeScript 功能编写他们的代码，然后使用 Babel 重写他们的代码，以便在旧浏览器上运行。这样，前端 JavaScript 程序员可以采用（部分）新功能，但需要更复杂的构建工具链，并且代码重写过程可能引入错误的风险。

Node.js 世界没有这个问题。Node.js 迅速采用了 ES2015/2016/2017 功能，就像它们在 V8 引擎中实现一样。从 Node.js 8 开始，我们可以自由地使用`async`函数作为一种原生功能。新的模块格式首次在 Node.js 版本 10 中得到支持。

换句话说，虽然前端 JavaScript 程序员可以主张他们必须等待几年才能采用 ES2015/2016/2017 功能，但 Node.js 程序员无需等待。我们可以简单地使用新功能，而无需任何代码重写工具，除非我们的管理人员坚持支持早于这些功能采用的旧 Node.js 版本。在这种情况下，建议您使用 Babel。

JavaScript 世界的一些进步是在 TC-39 社区之外发生的。

## TypeScript 和 Node.js

TypeScript 语言是 JavaScript 环境的一个有趣的分支。因为 JavaScript 越来越能够用于复杂的应用程序，编译器帮助捕捉编程错误变得越来越有用。其他语言的企业程序员，如 Java，习惯于强类型检查作为防止某些类别的错误的一种方式。

强类型检查在某种程度上与 JavaScript 程序员相悖，但它确实很有用。TypeScript 项目旨在从 Java 和 C#等语言中引入足够的严谨性，同时保留 JavaScript 的松散性。结果是编译时类型检查，而不会像其他语言中的程序员那样承载沉重的负担。

虽然我们在本书中不会使用 TypeScript，但它的工具链在 Node.js 应用程序中非常容易采用。

在本节中，我们了解到随着 JavaScript 语言的变化，Node.js 平台也跟上了这些变化。

# 使用 Node.js 开发微服务或最大服务

新的功能，如云部署系统和 Docker，使得实现一种新的服务架构成为可能。Docker 使得可以在可重复部署到云托管系统中的数百万个容器中定义服务器进程配置。它最适合小型、单一用途的服务实例，可以连接在一起组成一个完整的系统。Docker 并不是唯一可以帮助简化云部署的工具；然而，它的特性非常适合现代应用部署需求。

一些人将微服务概念作为描述这种系统的一种方式。根据[microservices.io](http://microservices.io/)网站，微服务由一组狭义、独立可部署的服务组成。他们将其与单片应用部署模式进行对比，单片应用将系统的每个方面集成到一个捆绑包中（例如 Java EE 应用服务器的单个 WAR 文件）。微服务模型为开发人员提供了非常需要的灵活性。

微服务的一些优势如下：

+   每个微服务可以由一个小团队管理。

+   每个团队可以按照自己的时间表工作，只要保持服务 API 的兼容性。

+   微服务可以独立部署，如果需要的话，比如为了更容易进行测试。

+   更容易切换技术栈选择。

Node.js 在这方面的定位如何？它的设计与微服务模型非常契合：

+   Node.js 鼓励小型、紧密专注、单一用途的模块。

+   这些模块由出色的 npm 包管理系统组成应用程序。

+   发布模块非常简单，无论是通过 NPM 仓库还是 Git URL。

+   虽然 Express 等应用框架可以用于大型服务，但它非常适用于小型轻量级服务，并支持简单易用的部署。

简而言之，使用 Node.js 以精益和敏捷的方式非常容易，可以根据您的架构偏好构建大型或小型服务。

# 总结

在本章中，您学到了很多东西。特别是，您看到了 JavaScript 在 Web 浏览器之外的生活，以及 Node.js 是一个具有许多有趣特性的优秀编程平台。虽然它是一个相对年轻的项目，但 Node.js 已经变得非常流行，不仅广泛用于 Web 应用程序，还用于命令行开发工具等。由于 Node.js 平台基于 Chrome 的 V8 JavaScript 引擎，该项目已经能够跟上 JavaScript 语言的快速改进。

Node.js 架构由事件循环触发回调函数管理的异步函数组成，而不是使用线程和阻塞 I/O。这种架构声称具有性能优势，似乎提供了许多好处，包括能够在更少的硬件上完成更多的工作。但我们也了解到低效的算法可能会抵消任何性能优势。

本书的重点是开发和部署 Node.js 应用程序的现实考虑。我们将尽可能涵盖开发、完善、测试和部署 Node.js 应用程序的许多方面。

既然我们已经对 Node.js 有了介绍，我们准备好开始使用它了。在第二章 *设置 Node.js*中，我们将介绍如何在 Mac、Linux 或 Windows 上设置 Node.js 开发环境，甚至编写一些代码。让我们开始吧。
