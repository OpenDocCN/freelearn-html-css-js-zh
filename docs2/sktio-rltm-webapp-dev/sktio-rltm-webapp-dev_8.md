# 附录 B. Socket.IO 后端

Socket.io 始于 Node.js，其主后端仍然是 Node.js。本书专注于使用 socket.io、Node.js 和 Express.js 构建聊天系统。但如果你的首选平台不是 Node.js，或者你正在从事一个希望获得与 socket.io 相同功能但无法实现的项目，因为你有一个现有的标准化平台并且不能引入新的系统。在你之前，许多人面临过同样的困境，本着开源的精神，socket.io 服务器适用于各种平台。在本附录中，让我们看看 socket.io 后端的各种实现。

每个平台都将要求你应用本书中的学习和逻辑来重写针对该平台的服务器端代码。客户端代码可以保持不变。

以下是一个按语言/平台字母顺序排列的实现列表：

# Erlang

在 erlang 上，socket.io 有两个不同的后端，Yurii Rashkovskii 的**socket.io-erlang** ([`github.com/yrashk/socket.io-erlang`](https://github.com/yrashk/socket.io-erlang)) 和 Yongboy 的 **erlang-socketio** ([`code.google.com/p/erlang-scoketio/`](https://code.google.com/p/erlang-scoketio/))。

Yurii 似乎对 socket.io 0.6.x 之后的版本所采取的道路存在分歧，因此该库仅支持规范的第 0.6 版。自然地，本书中的大多数示例以及互联网上的许多其他示例，都无法在它上面运行。

Yongboy 的 erlang-socketio 似乎在持续关注 socket.io 的最新动态，并且在撰写本文时与 socket.io-1.0 的最新规范兼容。因此，我们将在本节的剩余部分专注于这个库。

这个库适用于**Cowboy**和**Mochiweb**，这两个是 erlang 中流行的服务器端框架。这两个版本都支持 socket.io 规范 1.0。Cowboy 版本支持所有传输方式，而 Mochiweb 版本仅限于`xhr-polling`、`htmlfile`和`json-polling`。

# Google Go

**Go**是一种处于早期阶段的语言，但正在获得越来越多的关注，这主要得益于 Google 的企业支持，并且它是 Google App Engine 上支持的三种语言之一，除了 Python 和 Java。

`go-socket.io`实现为 Go 提供了 socket.io 支持。该项目支持几乎所有传输方式，并且也支持 Google App Engine 上的 socket.io。该项目的原始代码库位于[`github.com/madari/go-socket.io`](https://github.com/madari/go-socket.io)，但那里的开发已经停滞了一段时间；但其他人似乎已经接过了接力棒。socket.io 维基指向这个分支：[`github.com/davies/go-socket.io`](https://github.com/davies/go-socket.io)。

这里需要注意的一点是，这个代码库仍然不支持高于 0.6.x 的版本。

查看 GitHub 上创建的分支，你会发现代码正在进行的有趣开发。就像这个更新较新的分支：[`github.com/justinfx/go-socket.io`](https://github.com/justinfx/go-socket.io)。

如果你想要使用 socket.io 的新版本，[`github.com/murz/go-socket.io`](https://github.com/murz/go-socket.io)这个分支应该支持到 0.8.x 版本（这是在撰写本文时的情况）。

# Java

在 Java 服务器上，socket.io 有多种实现方式。让我们来看看它们。

第一个是**Socket.IO-Java**，在[`github.com/Ovea/Socket.IO-Java`](https://github.com/Ovea/Socket.IO-Java)上维护得最为活跃。它已经被分支并修改，以适应各种服务器和平台。

然后是**Atmosphere**。Atmosphere 最初是一个将服务器推送引入 glassfish 服务器的项目，但后来作为一个独立的项目分离出来，并且几乎与任何 Java 服务器一起工作。Atmosphere 服务器自带 atmosphere.js，这是它自己的 JS 客户端，但任何 Atmosphere 应用都可以与 socket.io 客户端无缝工作，无需任何修改；使用[`github.com/Atmosphere/atmosphere/wiki/Getting-Started-with-Socket.IO`](https://github.com/Atmosphere/atmosphere/wiki/Getting-Started-with-Socket.IO)开始使用 Atmosphere。如果你正在启动一个新的 Java 项目或者在你的现有 Java 项目中引入推送，在做出决定之前，一定要看看 Atmosphere。

**Netty**为 Java 带来了异步服务器；并且非常重要的一点是，Yongboy 的**socket** **io-netty** ([`code.google.com/p/socketio-netty/`](http://code.google.com/p/socketio-netty/))。由于其异步特性，Netty 非常适合这些应用，因此它被高度推荐。

**Gisio** ([`bitbucket.org/c58/gnisio/wiki/Home`](https://bitbucket.org/c58/gnisio/wiki/Home))将 socket.io 引入了 GWT 框架，这是谷歌的 Java 编写和编译为 JS 的库。如果你的应用是用 GWT 构建的，并且你想要在你的应用中引入服务器推送，你可以使用这个库。

对于新出现的完全异步服务器**Vert.x**，有**mod-socket-io** ([`github.com/keesun/mod-socket-io`](https://github.com/keesun/mod-socket-io))。再次提醒，如果你正在寻找高度异步应用的应用，我建议你看看这个服务器和这个模块。

# Perl

Perl 可能是一个非常古老的语言，但仍然在许多地方被使用，并且有一个名为**pocketio** ([`github.com/vti/pocketio`](https://github.com/vti/pocketio))的活跃维护的 socket.io 服务器模块。

# Python

Python 是另一种正在获得广泛接受和流行的语言。Python 也有多个 socket.io 服务器实现。

我们首先来看的是**gevent-socket.io** ([`github.com/abourget/gevent-socketio`](https://github.com/abourget/gevent-socketio))，它与任何基于 WSGI 的 Web 框架兼容。所以如果你使用的是任何框架，如 Pyramid、Pylons、Flask 和 Django，这将适用于你。唯一的依赖项是**gevent**和**gevent-websocket**。

如果 Tornado 是你的首选框架，请查看**Tornadio 2** ([`github.com/MrJoes/tornadio2`](https://github.com/MrJoes/tornadio2))，它提供了对 socket.io 0.7 及以上版本的支持。再次强调，Tornado 是一个异步框架，非常适合此类应用。

专注于将 socket.io 引入 Django 的是**django-socketio** ([`github.com/stephenmcd/django-socketio`](https://github.com/stephenmcd/django-socketio))。

# 摘要

在本章中，我们看到了一些流行平台的 socket.io 后端实现。如果你使用的是其他平台，只需在互联网上搜索 socket.io 服务器实现，我确信你一定能找到。它可能不是最好的，也可能不是处于理想状态，但你肯定能找到一个开始的方法。
