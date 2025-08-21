# 附录 A. 安装 Node.js 和使用 npm

# 介绍

Node.js 是建立在 Google Chrome 的 V8 JavaScript 引擎之上的事件驱动平台。该平台为 V8 实现了完全非阻塞的 I/O，并主要用于构建实时 I/O 密集型的 Web 应用程序。

Node.js 安装程序提供以下两个主要组件：

+   node 二进制文件，可用于运行为该平台编写的 JavaScript 文件

+   node 包管理器**npm**，可用于安装由 node 社区编写的 node 库和工具

# 安装 Node.js

Node.js 的安装程序和分发程序可以在其官方网站[`nodejs.org/`](http://nodejs.org/)上找到。安装过程因操作系统而异。

在 Windows 上，提供了两个基于 MSI 的安装程序，一个用于 32 位操作系统，另一个用于 64 位操作系统。要在 Windows 上安装 Node.js，只需下载并执行安装程序。

对于 Mac OS X，同一位置提供了一个`pkg`安装程序；下载并运行 PKG 文件将允许您使用 Apple 安装程序安装 Node.js。

在 Linux 上，安装过程取决于发行版。许多流行发行版的说明可在 node 维基上找到[`github.com/joyent/node/wiki/Installing-Node.js-via-package-manager`](https://github.com/joyent/node/wiki/Installing-Node.js-via-package-manager)。

# 使用 npm

Node.js 安装程序附带了 node 包管理器 npm。npm 用于命令行；要使用它，我们需要运行一个终端程序（命令提示符）。

在 Windows 上，我们可以使用基本的`cmd.exe`，或者我们可以从[`sourceforge.net/projects/console/`](http://sourceforge.net/projects/console/)下载并安装 Console。

在 Mac OS X 上，`Terminal.app`可用于运行命令。

在 Linux 上，使用您喜欢的终端。Ubuntu Linux 上的默认终端是 gnome 终端。

打开终端并输入：`npm`。此命令运行 npm 而不带任何参数。结果，npm 将打印一个列出可用子命令的一般使用概述。

## 安装本地包

让我们为名为`test`的项目创建一个空目录，转到该目录，并在那里使用 npm 安装`underscore`库。运行以下命令：

```html
mkdir test
cd test
npm install underscore

```

最后一个命令将告诉 npm 运行带有参数`underscore`的`install`子命令，这将在本地安装 underscore 包。npm 将在下载和安装包时输出一些进度信息。

在安装包时，npm 会在当前目录中创建一个名为`node_modules`的子目录。在该目录中，它会为安装的包创建另一个目录。在这种情况下，underscore 包将放置在`underscore`目录中。

## 安装全局包

一些 npm 包设计为全局安装。全局包为操作系统添加新功能。例如，可以全局安装 coffee-script 包，这将使`coffee`命令在我们的系统上可用。

要安装全局包，我们使用-g 开关。看下面的例子：

```html
npm install -g coffee-script

```

在某些系统上，需要请求管理员权限来运行此程序。您可以使用`sudo`命令来做到这一点：

```html
sudo npm install -g coffee-script

```

npm 将下载并安装 coffee-script 以及其所有依赖项。完成后，我们可以开始使用`coffee`命令，在系统上现在可用。我们现在可以运行 coffee-script 代码。假设我们想要运行一个简单的内联 hello-world 脚本；我们可以使用`-e`开关。看下面的例子：

```html
coffee -e "echo 'Hello world'"

```

要了解有关 npm 子命令的全局包的更多信息，我们可以使用 npm 的 help 子命令。例如，要了解有关`install`子命令的更多信息，请运行以下命令：

```html
npm help install

```

有关最新版本的 npm 的更多信息可以在官方 npm 文档[`npmjs.org/doc/`](https://npmjs.org/doc/)中找到。
