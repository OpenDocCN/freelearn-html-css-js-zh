# 第一章. Opa 入门

本章将展示如何安装 Opa 并设置其环境。还将展示一个简单的 Opa 程序，以提供一个对 Opa 编程的第一印象。

# 安装 Opa

本节是关于 Opa 的安装和配置。您可以在 Opa 的网页上找到更详细的安装指南，包括如何从源代码构建 Opa（[`github.com/MLstate/opalang/wiki/Getting-started`](https://github.com/MLstate/opalang/wiki/Getting-started)）。本节将简要说明如何安装 Opa 编译器、Node.js 以及 Node.js 所需的一些模块。

## 安装 Node.js

Node.js ([`nodejs.org`](http://nodejs.org)) 是一个用于构建快速和可扩展网络应用程序的平台。它是 Opa 的后端（自 Opa 1.0.0 版本起）。在安装 Opa 之前，我们需要先安装 Node.js。以下是在不同操作系统上安装 Node.js 的步骤：

+   **Mac OS**: 安装 Node.js 的步骤如下：

    1.  从[`nodejs.org/dist/latest/`](http://nodejs.org/dist/latest/)下载最新的 `.pkg` 包。

    1.  双击安装包以安装 Node.js。

+   **Ubuntu 和 Debian Linux**: 在 Ubuntu 和 Debian Linux 上安装 Node.js，请输入以下命令：

    ```js
    $sudo apt-get install python-software-properties
    $sudo add-apt-repository ppa:chris-lea/node.js
    $sudo apt-get update
    $sudo apt-get install nodejs npm

    ```

    ### 提示

    **下载示例代码文件**

    您可以从[`www.packtpub.com`](http://www.packtpub.com)的账户下载您购买的所有 Packt 书籍的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

+   **Windows**: 安装 Node.js 的步骤如下：

    1.  从[`nodejs.org/dist/latest/`](http://nodejs.org/dist/latest/)下载最新的 `.msi` 包。

    1.  在 Windows 上双击安装包以安装 Node.js。

输入以下命令以确认您的安装。如果一切顺利，您将看到 Node.js 和 npm 的版本信息。

```js
$ node -v
$ npm –v

```

## 安装所需模块

Opa 运行应用程序需要几个模块。输入以下命令安装这些模块：

```js
$ npm install -g mongodb formidable nodemailer simplesmtp imap

```

## 安装 Opa 编译器

安装 Opa 的最简单方法是下载 Opa 网站上的安装程序([`opalang.org/`](http://opalang.org/))。您也可以从 Opa 的 GitHub 仓库([`github.com/MLstate/opalang/downloads`](https://github.com/MLstate/opalang/downloads))获取安装程序。在本书编写时，Opa 的最新版本是 1.1.0。

以下是在不同操作系统上安装 Opa 的步骤：

+   **Mac OS X**: 下载最新的 `.dmg` 包并双击安装。您将需要管理员账户的密码。

+   **Ubuntu 和 Debian Linux**: 下载最新的 `.deb` 包并双击安装。您也可以使用以下命令行进行安装：

    ```js
    $sudo dpkg –i opa-1.1.0.x86.deb

    ```

+   **Windows**：下载最新的 `.exe` 文件，双击以安装。请注意，目前 Windows 上仅提供 64 位包。

+   **其他 Linux**：要安装 Opa，请按照以下步骤操作：

    1.  下载 Linux 的最新 `.run` 包。

    1.  前往下载文件夹，通过运行以下命令为下载的文件添加执行权限：

        ```js
        $ chmod a+x opa-1.1.0.x64.run

        ```

    1.  运行安装脚本：

        ```js
        $ sudo ./opa-1.1.0.x64.run

        ```

## 测试安装

要测试 Opa 是否已正确安装在您的计算机上，请运行以下命令：

```js
$ opa --version

```

如果打印出 Opa 编译器的版本信息，则表示 Opa 已正确安装。

# 设置编辑器

您可以使用您喜欢的任何文本编辑器编写 Opa 代码，但一个好的编辑器可以使编码更容易。本节是关于设置您可能常用的编辑器。目前，Sublime Text 是 Opa 最完整的 **集成开发环境**（**IDE**）。

## Sublime Text

Sublime Text ([`www.sublimetext.com/`](http://www.sublimetext.com/)) 是一个用于代码、标记和散文的复杂文本编辑器。您可以从 [`www.sublimetext.com/2`](http://www.sublimetext.com/2) 免费下载并试用 Sublime Text。

有一个 Opa 插件提供了语法高亮、代码补全和一些其他功能。要安装此插件，请按照以下步骤操作：

1.  从 [`github.com/downloads/MLstate/OpaSublimeText/Opa.sublime-package`](https://github.com/downloads/MLstate/OpaSublimeText/Opa.sublime-package) 获取插件。

1.  将其移动到 `~/.config/sublime-text2/Installed Packages/`（在 Linux 上），或 `%%APPDATA%%\Sublime Text 2\Installed Packages\`（在 Windows 上），或 `~/Library/Application Support/Sublime Text 2/Installed Packages`（在 Mac 上）。

1.  启动 Sublime 并检查是否出现菜单项（**视图** | **语法** | **Opa**）。如果一切顺利，具有 `.opa` 扩展名的文件应该自动具有语法高亮。如果不这样，请确保您正在使用 Opa 插件（**视图** | **语法** | **Opa**）。我们可以导航到 **编辑** | **行** | **自动缩进**来自动缩进 Opa 代码。

## Vim

Vim ([`www.vim.org/`](http://www.vim.org/)) 是一个高度可配置的文本编辑器，免费适用于许多不同的平台。Opa 安装包在 `/usr/share/opa/vim/`（对于 Linux）或 `/opt/mlstate/share/opa/vim/`（对于 Mac OS）提供了 Vim 的模式。要使 Vim 能够检测 Opa 语法，将这些文件复制到您主目录中的 `.vim` 目录（如果尚不存在，请创建它）：

+   在 Linux 上，键入以下命令：

    ```js
    $cp –p /usr/share/opa/vim/* ~/.vim/

    ```

+   在 Mac OS 上，键入以下命令：

    ```js
    $cp –p /opt/mlstate/share/opa/vim/* ~/.vim

    ```

## Emacs

在 Mac OS X 上，您可以使用 Aquamacs，包安装将自动处理，或者您应该将以下行添加到您的配置文件中（该文件可能是 `~/.emacs`；如果尚不存在，请创建它）：

```js
(autoload 'opa-classic-mode "/Library/Application Support/Emacs/site-lisp/opa-mode/opa-mode.el" "Opa CLASSIC editing mode." t)
(autoload 'opa-js-mode "/Library/Application Support/Emacs/site-lisp/opa-mode/opa-js-mode.el" "Opa JS editing mode." t)
(add-to-list 'auto-mode-alist '("\.opa$" . opa-js-mode))
(add-to-list 'auto-mode-alist '("\.js\.opa$" . opa-js-mode))
(add-to-list 'auto-mode-alist '("\.classic\.opa$" . opa-classic-mode))
```

在 Linux 上，将以下行添加到您的配置文件中：

```js
(autoload 'opa-js-mode "/usr/share/opa/emacs/opa-js-mode.el" "Opa JS editing mode." t)
(autoload 'opa-classic-mode "/usr/share/opa/emacs/opa-mode.el" "Opa CLASSIC editing mode." t)
(add-to-list 'auto-mode-alist '("\.opa$" . opa-js-mode))
(add-to-list 'auto-mode-alist '("\.js\.opa$" . opa-js-mode))
(add-to-list 'auto-mode-alist '("\.classic\.opa$" . opa-classic-mode))
```

对于 Eclipse，实验性插件可在 [`github.com/MLstate/opa-eclipse-plugin`](https://github.com/MLstate/opa-eclipse-plugin) 获取。

# 您的第一个 Opa 应用程序

作为第一个示例，这里是最简单的 Opa 程序：

```js
jlog("hello Opa!")
```

编译并运行它：

```js
$ opa hello.opa -o hello.js
$ ./hello.js

```

### 注意

我们可以输入 `opa hello.opa --` 来编译并运行代码，一行完成。

这段代码什么也不做，只是在屏幕上打印 **hello Opa**。如果你能看到这条消息，这意味着 Opa 在你的机器上运行正常。

# 摘要

在本章中，我们学习了如何安装 Opa，设置合适的编辑器，并编写我们的第一个 Opa 程序。在下一章中，我们将简要了解 Opa 语言的语法基础。
