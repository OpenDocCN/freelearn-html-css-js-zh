# 第二章：Web 开发工具

*每个专业人士都有一套工具，可以简化他们的工作并完成任务。同样，我们也需要我们自己的工具来构建响应式网站。因此，在我们开始本书中的项目之前，我们需要准备以下工具。*

我们需要准备的工具包括：

+   用于编写代码的代码编辑器

+   一个编译器，将编译 CSS 预处理器语法为纯 CSS

+   在开发阶段本地托管网站的本地服务器

+   一个 Bower 来管理网站库

# 选择一个代码编辑器

一旦我们开始编写 HTML、CSS 和 JavaScript 代码，我们就需要一个代码编辑器。代码编辑器是开发网站的必不可少的工具。从技术上讲，你只需要文本编辑器，比如 OS X 中的 TextEdit 或 Windows 中的记事本来编写和编辑代码。然而，使用代码编辑器可以减少眼睛的刺激。

类似于 Microsoft Word，专门设计用于使单词和段落格式更直观，代码编辑器设计有一组特殊功能，可以改善代码编写体验，如语法高亮、自动补全、代码片段、多行选择，并支持大量语言。语法高亮将以不同的颜色显示代码，增强代码的可读性，并使查找代码中的错误变得容易。

我的个人偏好是 Sublime Text（[`www.sublimetext.com/`](http://www.sublimetext.com/)），这也是我在这本书中将要使用的。Sublime Text 是一个跨平台的代码编辑器，适用于 Windows、OS X 和 Linux。可以免费下载进行无限期评估。

### 注意

请记住，虽然 Sublime Text 允许我们无限期免费评估，但有时可能会提示您购买许可证。如果你开始感到烦恼，请考虑购买许可证。

## Sublime Text Package Control

我最喜欢 Sublime Text 的一件事是 Package Control，我们可以方便地从 Sublime Text 中搜索、安装、列出和删除扩展。但是，Package Control 并不是 Sublime Text 的预装软件。因此，假设你已经安装了 Sublime Text（我认为你应该已经安装了），我们将在 Sublime Text 中安装 Package Control。

# 行动时间-安装 Sublime Text Package Control

执行以下步骤安装 Sublime Text Package Control；这将允许我们轻松安装 Sublime Text 扩展：

1.  在 Sublime Text 中通过 Sublime Text 控制台是安装 Package Control 的最简单方法。在 Sublime Text 中导航到**View** | **Console**菜单以打开控制台。现在你应该看到一个新的输入字段出现在底部，如下面的截图所示：![行动时间-安装 Sublime Text Package Control](img/image00226.jpeg)

1.  由于 Sublime Text 3 进行了大规模的改动，几乎改变了整个 API，Package Control 现在分为两个版本，一个是 Sublime Text 2，另一个是 Sublime Text 3。每个版本都需要不同的代码来安装 Package Control。如果你使用的是 Sublime Text 2，复制代码从[`sublime.wbond.net/installation#st2`](https://sublime.wbond.net/installation#st2)。如果你使用的是 Sublime Text 3，复制代码从[`sublime.wbond.net/installation#st3`](https://sublime.wbond.net/installation#st3)。

1.  将你从第 2 步复制的代码粘贴到控制台输入字段中，如下面的截图所示：![行动时间-安装 Sublime Text Package Control](img/image00227.jpeg)

1.  按下*Enter*运行代码，最终安装 Package Control。请记住，这个过程可能需要一段时间，具体取决于您的互联网连接速度。

## *刚刚发生了什么？*

我们刚刚安装了 Package Control，可以轻松搜索、安装、列出和删除 Sublime Text 中的扩展。您可以通过**命令面板…**访问 Package Control，方法是导航到**工具** | **命令面板…**菜单。或者，您可以按键快捷方式更快地访问它。Windows 和 Linux 用户可以按*Ctrl* + *Shift* + *P*，而 OS X 用户可以按*Command* + *Shift* + *P*。然后，搜索**命令面板…**以列出 Package Control 的所有可用命令。

![刚刚发生了什么？](img/image00228.jpeg)

## 尝试一下-安装 LESS 和 Sass 语法高亮包

如第一章所述，我们将在本书的两个项目中使用这些 CSS 预处理器来编写样式。已经安装了 Sublime Text 和 Package Control，现在可以轻松安装 Sublime Text 包，以便为 LESS 和 Sass/SCSS 语法启用颜色高亮。继续并按照我们刚刚展示的说明安装 LESS 和 Sass/SCSS 包，它们的语法可以在以下位置找到：

+   Sublime Text 的 LESS 语法（[`github.com/danro/LESS-sublime`](https://github.com/danro/LESS-sublime)）

+   Sass 和 SCSS 的语法高亮（[`github.com/P233/Syntax-highlighting-for-Sass`](https://github.com/P233/Syntax-highlighting-for-Sass)）

## 设置本地服务器

在开发网站时，有一个本地服务器在我们的计算机上设置并运行是必要的。当我们使用本地服务器存储我们的网站时，我们可以通过浏览器在`http://localhost/`上访问它，并且我们也可以在手机浏览器和平板电脑上访问它，在`file:///`协议下运行网站时是不可能的。此外，一些脚本可能只能在 HTTP 协议（`http://`）下运行。

有许多应用程序可以通过几次点击轻松设置本地服务器，XAMPP（[`www.apachefriends.org/`](https://www.apachefriends.org/)）是本书中将要使用的应用程序。

# 操作时间-安装 XAMPP

XAMPP 适用于 Windows、OS X 和 Linux。从[`www.apachefriends.org/download.html`](https://www.apachefriends.org/download.html)下载安装程序；根据您当前使用的平台选择安装程序。每个平台都有不同的安装程序和不同的扩展名；Windows 用户将获得`.exe`，OSX 用户将获得`.dmg`，而 Linux 用户将获得`.run`。按照以下步骤在 Windows 中安装 XAMPP：

1.  启动 XAMPP`.exe`安装程序。

1.  如果 Windows 用户帐户控制提示**是否允许以下程序更改此计算机？**，请单击**是**。![操作时间-安装 XAMPP](img/image00229.jpeg)

1.  当**XAMPP 设置向导**窗口出现时，单击**下一步**开始设置。

1.  XAMPP 允许我们选择要安装的组件。在这种情况下，我们的 Web 服务器要求是最低限度的。我们只需要 Apache 来运行服务器，因此我们取消其他选项。（注意：**PHP**选项是灰色的；它无法取消选中）：![操作时间-安装 XAMPP](img/image00230.jpeg)

1.  确认要安装的组件后，单击**下一步**按钮继续。

1.  将提示您安装 XAMPP 的位置。让我们将其保留在默认位置`C:\xampp`，然后单击**下一步**。

1.  然后，简单地点击**下一步**，直到安装 XAMPP 的过程完成。等待过程完成。

1.  安装完成后，您应该会看到窗口上显示**安装 XAMPP 的设置已经完成**。单击**完成**按钮以完成流程并关闭窗口。

按照以下步骤在 OS X 中安装 XAMPP：

1.  对于 OS X 用户，打开 XAMPP`.dmg`文件。一个新的**Finder**窗口应该出现，其中包含实际的安装程序文件，通常命名为`xampp-osx-*-installer`（星号（`*`）代表 XAMPP 版本），如下图所示：![执行操作-安装 XAMPP](img/image00231.jpeg)

1.  双击**安装程序**文件开始安装。XAMPP 需要您的计算机凭据来运行安装程序。因此，请输入您的计算机名称和密码，然后单击**确定**以授予访问权限。

1.  然后会出现**XAMPP 设置向导**窗口；单击**下一步**开始设置。

1.  与 Windows 列出每个项目的组件不同，OS X 版本只显示两个组件，即**XAMPP 核心文件**和**XAMPP 开发人员文件**。在这里，我们只需要**XAMPP 核心文件**，其中包括我们需要运行服务器的 Apache、MySQL 和 PHP。因此，取消选择**XAMPP 开发人员**选项，然后单击**下一步**按钮继续。![执行操作-安装 XAMPP](img/image00232.jpeg)

1.  您将收到提示，XAMPP 将安装在`Applications`文件夹中。与 Windows 不同，此目录无法编辑。因此，单击**下一步**继续。

1.  然后，只需单击**下一步**按钮，继续下两个对话框以开始安装 XAMPP。等待直到完成。

1.  安装完成后，您将看到**安装 XAMPP 的设置已完成**显示在窗口中。单击**完成**按钮完成流程并关闭窗口。

执行以下步骤在 Ubuntu 中安装 XAMPP：

1.  下载 Linux 版 XAMPP 安装程序。安装程序以`.run`扩展名提供，适用于 32 位和 64 位系统。

1.  打开终端并导航到安装程序下载的文件夹。假设它在`Downloads`文件夹中，输入：

```html
cd ~/Downloads
```

1.  使用`chmod u+x`使`.run`安装程序文件可执行，然后输入`.run`安装程序文件名：

```html
chmod u+x xampp-linux-*-installer.run
```

1.  使用`sudo`命令执行文件，后跟`.run`安装程序文件位置，如下所示：

```html
sudo ./xampp-linux-x64-1.8.3-4-installer.run
```

1.  第 4 步的命令将弹出**XAMPP 设置向导**窗口。单击**下一步**继续，如下图所示：![执行操作-安装 XAMPP](img/image00233.jpeg)

1.  安装程序允许您选择要在计算机上安装的组件。与 OS X 版本类似，选项中显示了两个组件：**XAMPP 核心文件**（包含 Apache、MySQL、PHP 和一堆其他运行服务器的东西）和**XAMPP 开发人员文件**。由于我们不需要**XAMPP 开发人员文件**，因此我们可以取消选择它，然后单击**下一步**按钮。

1.  安装程序将向您显示将在`/opt/lampp`中安装 XAMPP。文件夹位置无法自定义。只需单击**下一步**按钮继续。

1.  单击**下一步**按钮，继续下两个对话框屏幕以安装 XAMPP。

## *刚刚发生了什么？*

我们刚刚在计算机上使用 MAMP 设置了本地服务器。您现在可以通过浏览器访问`http://localhost/`地址访问本地服务器。但是，对于 OS X 用户，地址是您的计算机用户名后跟`.local`。假设您的用户名是 john，本地服务器可以通过`john.local`访问。每个平台的本地服务器目录路径都不同。

+   在 Windows：`C:\xampp\htdocs`

+   在 OSX：`/Applications/XAMPP/htdocs`

+   在 Ubuntu：`/opt/lampp/htdocs`

### 提示

Ubuntu 用户可能希望更改权限并在桌面上创建一个`symlink`文件夹以便方便地访问`htdocs`文件夹。为此，您可以通过终端从桌面位置运行`sudo chown username:groupname /opt/lampp/htdocs`命令。将`username`和`groupname`替换为您自己的。

`ln -s /opt/lamp/htdocs`文件夹是我们必须放置项目文件夹和文件的位置。从现在开始，我们将简称这个文件夹为`htdocs`。XAMPP 还配备了一个图形应用程序，您可以在其中打开和关闭服务器，如下图所示：

![刚刚发生了什么？](img/image00234.jpeg)

### 注意

Ubuntu 用户，您需要运行`sudo /opt/lampp/manager-linux.run`或`manager-linux-x64.run`。

## 选择 CSS 预处理器编译器

由于我们将使用 LESS 和 Sass 来生成响应式网站的样式表，我们将需要一个工具来将它们编译或转换成普通的 CSS 格式。

在 CSS 预处理器刚刚开始流行的时候，编译它们的唯一方法是通过命令行，这可能是当时许多人甚至不愿尝试 CSS 预处理器的绊脚石。幸运的是，现在我们有很多应用程序具有良好的图形界面来编译 CSS 预处理器；以下是供您参考的列表：

| 工具 | 语言 | 平台 | 价格 |
| --- | --- | --- | --- |
| --- | --- | --- | --- |
| WinLESS（[`winless.org/`](http://winless.org/)） | LESS | Windows | 免费 |
| SimpLESS（[`wearekiss.com/simpless`](http://wearekiss.com/simpless)） | LESS | Windows，OSX | 免费 |
| ChrunchApp（[`crunchapp.net`](http://crunchapp.net)） | LESS | Windows，OSX | 免费 |
| CompassApp（[`compass.handlino.com`](http://compass.handlino.com)） | Sass | Windows，OSX，Linux | $10 |
| Prepros（[`alphapixels.com/prepros/`](http://alphapixels.com/prepros/)） | LESS，Sass 等 | Windows，OSX | Freemium（$24） |
| Codekit（[`incident57.com/codekit/`](https://incident57.com/codekit/)） | LESS，Sass 等 | OSX | $29 |
| Koala（[`koala-app.com/`](http://koala-app.com/)） | LESS，Sass 等 | Windows，OSX，Linux | 免费 |

我将尽可能涵盖多个平台。无论您使用哪个平台，您都可以跟随本书中的所有项目。因此，我们将使用 Koala。它是免费的，并且可以在 Windows，OSX 和 Linux 这三个主要平台上使用。

在每个平台上安装 Koala 非常简单。

## 开发浏览器

理想情况下，我们必须在尽可能多的浏览器中测试我们的响应式网站，包括 Firefox Nightly（[`nightly.mozilla.org/`](http://nightly.mozilla.org/)）和 Chrome Canary（[`www.google.com/intl/en/chrome/browser/canary.html`](http://www.google.com/intl/en/chrome/browser/canary.html)）等测试版浏览器。这是为了确保我们的网站在不同环境中运行良好。然而，在开发过程中，我们可能会选择一个主要的浏览器进行开发，并作为网站应该如何展示的参考点。

在这本书中，我们将使用 Chrome（[`www.google.com/intl/en/chrome/browser/`](https://www.google.com/intl/en/chrome/browser/)）。我认为 Chrome 不仅运行速度快，而且也是一个非常强大的网页开发工具。Chrome 带有一套领先于其他浏览器的工具。以下是我在开发响应式网站时 Chrome 中两个我最喜欢的工具。

### 源映射

使用 CSS 预处理器的一个缺点是在调试样式表时。由于样式表是生成的，浏览器引用 CSS 样式表时，我们会发现很难发现代码在 CSS 预处理器样式表中的确切位置。

我们可以告诉编译器生成包含代码实际编写位置的注释，但源映射更优雅地解决了这个问题。与其生成一堆最终污染样式表的注释，我们可以在编译 CSS 预处理器时生成一个`.map`文件。通过这个`.map`文件，启用源映射的浏览器（如 Chrome）在检查元素时将能够直接指向源代码，如下面的截图所示：

![源映射](img/image00235.jpeg)

正如您从前面的屏幕截图中所看到的，左侧显示的 Chrome DevTools 启用了源映射，直接指向了允许我们轻松调试网站的`.less`文件。而右侧显示的源映射被禁用，因此它指向`.css`，调试网站将需要一些努力。

源映射功能在最新版本的 Chrome 中默认启用。因此，请确保您的 Chrome 是最新的。

### 移动模拟器

在真实设备上测试响应式网站，如手机或平板电脑，是无法替代的。每个设备都有其自身的优点；一些因素，如屏幕尺寸、屏幕分辨率和移动浏览器的版本，都会影响网站在设备上的显示。然而，如果不可能的话，我们可以使用移动模拟器作为替代方案。

Chrome 还附带了一个开箱即用的移动模拟器。该功能包含许多移动设备的预设值，包括 iPhone、Nexus 和 Blackberry。此功能不仅模拟设备的用户代理，还打开了许多设备规格，包括屏幕分辨率、像素比、视口大小和触摸屏。这个功能对于在开发早期调试我们的响应式网站非常有用，而不需要实际的移动设备。

移动模拟器可以通过 Chrome DevTool 的**控制台**抽屉中的**模拟**选项卡访问，如下面的屏幕截图所示：

![Mobile emulator](img/image00236.jpeg)

在 Chrome 内置的移动模拟器中，我们不需要再从第三方应用程序或 Chrome 扩展中设置另一个模拟器。在这里，我们将使用它来测试我们的响应式网站。

### 提示

Firefox 也有类似于 Chrome 移动模拟器的功能，尽管它只有很少的功能。您可以通过导航到**工具** | **Web 开发人员** | **响应式设计视图**菜单来启用此功能。

# 使用 Bower 管理项目依赖

我们将需要一些库来管理 Bower 的项目依赖关系。在 Web 开发环境中，我们将库称为一组代码，通常是 CSS 和 JavaScript，用于在网站上添加功能。通常，网站依赖于特定的库来实现其主要功能。例如，如果我建立一个用于转换货币的网站，该网站将需要 Account.js（[`josscrowcroft.github.io/accounting.js/`](http://josscrowcroft.github.io/accounting.js/)）；这是一个方便的 JavaScript 库，用于将常规数字转换为带有货币符号的货币格式。

通常，我们可能在单个网站上添加大约五个或更多的库，但是维护网站中使用的所有库，并确保它们都是最新的可能会变得繁琐。这就是 Bower 有用的地方。

Bower（[`bower.io/`](http://bower.io/)）是一个前端包管理器。它是一个方便的工具，可以简化我们在项目中添加、更新和删除库或依赖项（项目所需的库）的方式。Bower 是一个 Node.js 模块，因此我们首先必须在计算机上安装 Node.js（[`nodejs.org/`](http://nodejs.org/)）才能使用 Bower。

# 执行以下操作-安装 Node.js

执行以下步骤在 Windows、OS X 和 Ubuntu（Linux）中安装 Node.js。您可以直接跳转到您正在使用的平台的部分。

执行以下步骤在 Windows 中安装 Node.js：

1.  从 Node.js 下载页面（[`nodejs.org/download/`](http://nodejs.org/download/)）下载 Node.js Windows 安装程序。选择适合您 Windows 系统的 32 位或 64 位版本，以及`.msi`或`.exe`安装程序。

### 提示

**32 位或 64 位**

请按照此页面查看您的 Windows 计算机是运行在 32 位还是 64 位系统上[`windows.microsoft.com/en-us/windows/32-bit-and-64-bit-windows`](http://windows.microsoft.com/en-us/windows/32-bit-and-64-bit-windows)。

1.  运行安装程序（`.exe`或`.msi`文件）。

1.  单击 Node.js 欢迎消息的**下一步**按钮。

1.  通常情况下，当您安装软件或应用程序时，您将首先收到应用程序的许可协议提示。阅读完许可协议后，单击**我接受许可协议中的条款**，然后单击**下一步**按钮继续。

1.  然后，系统会提示您选择 Node.js 应安装的文件夹。将其保留为默认文件夹，即`C:\Program Files\nodejs\`。

1.  如下截图所示，安装程序然后会提示您是否要自定义要安装的项目。将其保留原样，然后单击**下一步**按钮继续，如下截图所示：![执行操作-安装 Node.js](img/image00237.jpeg)

1.  然后，单击**安装**按钮开始安装 Node.js。

1.  安装过程非常快速；只需几秒钟。如果看到通知显示**Node.js 已成功安装**，则可以单击**完成**按钮关闭安装窗口。

执行以下步骤在 OS X 中安装 Node.js：

1.  下载 OS X 的 Node.js 安装程序，其扩展名为`.pkg`。

1.  安装程序将向您显示欢迎消息，并显示将安装 Node.js 的位置（`/usr/local/bin`），如下截图所示：![执行操作-安装 Node.js](img/image00238.jpeg)

1.  然后，安装程序会显示用户许可协议。如果您已阅读并同意许可协议，请单击**同意**按钮，然后单击**下一步**按钮。

1.  OS X 的 Node.js 安装程序允许您在安装到计算机之前选择要安装的 Node.js 功能。在这里，我们将安装所有功能；只需单击**安装**按钮即可开始安装 Node.js，如下截图所示：![执行操作-安装 Node.js](img/image00239.jpeg)

### 注意

如果要自定义 Node.js 安装，请单击左下角的**自定义**按钮，如前面的截图所示。

执行以下步骤在 Ubuntu 中安装 Node.js：

在 Ubuntu 中安装 Node.js 非常简单。Node.js 可以通过 Ubuntu 的**高级打包工具**（**APT**）或`apt-get`安装。如果您使用的是 Ubuntu 13.10 版本或更高版本，可以启动终端并依次运行以下两个命令：

```html
sudo apt-get install nodejs
sudo apt-get install npm

```

如果您使用的是 Ubuntu 13.04 或更早版本，请改为运行以下命令：

```html
sudo apt-get install -y python-software-properties python g++ make
sudo add-apt-repository ppa:chris-lea/node.js
sudo apt-get update
sudo apt-get install nodejs

```

## *刚刚发生了什么？*

我们刚刚安装了 Node.js 和`npm`命令，这使我们能够稍后通过**Node.js 软件包管理器**（**NPM**）使用 Bower。`npm`命令行现在应该可以通过 Windows 命令提示符或 OS X 和 Ubuntu 终端访问。运行以下命令测试`npm`命令是否有效：

```html
npm -v

```

此命令返回计算机中安装的 NPM 版本，如下截图所示：

![刚刚发生了什么？](img/image00240.jpeg)

此外，对于 Windows 用户，您可能会在命令提示符窗口顶部看到一条消息，显示**您的环境已设置为使用 Node.js 和 npm**，如下截图所示：

![刚刚发生了什么？](img/image00241.jpeg)

这表明您可以在命令提示符中执行`node`和`npm`命令。由于我们已经设置了 Node.js 和`npm`正在运行，现在我们将安装 Bower。

## 英雄试试看-熟悉命令行

在本书中，我们将使用许多命令行来安装 Bower 以及 Bower 包。然而，如果您像我一样来自图形设计背景，在那里我们主要使用图形应用程序，第一次操作命令行可能会感到非常尴尬和令人生畏。因此，我建议您花时间熟悉基本的命令行。以下是一些值得参考的参考资料：

+   *设计师的命令行介绍* 由 Jonathan Cutrell 撰写（[`webdesign.tutsplus.com/articles/a-designers-introduction-to-the-command-line--webdesign-6358`](https://webdesign.tutsplus.com/articles/a-designers-introduction-to-the-command-line--webdesign-6358)）

+   《终端导航：温和介绍》由 Marius Masalar 撰写（[`computers.tutsplus.com/tutorials/navigating-the-terminal-a-gentle-introduction--mac-3855`](https://computers.tutsplus.com/tutorials/navigating-the-terminal-a-gentle-introduction--mac-3855)）

+   *Windows 命令提示符介绍* 由 Lawrence Abrams 撰写（[`www.bleepingcomputer.com/tutorials/windows-command-prompt-introduction/`](http://www.bleepingcomputer.com/tutorials/windows-command-prompt-introduction/)）

+   *Linux 命令介绍* 由 Paul Tero 撰写（[`www.smashingmagazine.com/2012/01/23/introduction-to-linux-commands/`](http://www.smashingmagazine.com/2012/01/23/introduction-to-linux-commands/)）

# 行动时间 - 安装 Bower

执行以下步骤安装 Bower：

1.  如果使用 Windows，请打开命令提示符。如果使用 OS X 或 Ubuntu，请打开终端。

1.  运行以下命令：

```html
npm install -g bower

```

### 注

如果在 Ubuntu 中安装 Bower 遇到问题，请使用 `sudo` 运行命令。

## *刚刚发生了什么？*

我们刚刚在计算机上安装了 Bower，这使得 `bower` 命令可用。我们在前面的命令中包含的 `-g` 参数是为了全局安装 Bower，这样我们就能在计算机的任何目录中执行 `bower` 命令。

## Bower 命令

安装了 Bower 之后，我们可以访问一组命令行来操作 Bower 的功能。我们将在终端中运行这些命令，或者在 Windows 中使用命令提示符，就像我们用 `npm` 命令安装 Bower 一样。所有命令都以 `bower` 开头，后面跟着命令关键字。以下是我们可能经常使用的命令列表：

| 命令 | 功能 |
| --- | --- |
| `bower install <library-name>` | 将库安装到项目中。当我们执行此功能时，Bower 会创建一个名为 `bower_components` 的新文件夹来保存所有库文件。 |
| `bower list` | 列出项目中所有已安装的包名称。该命令还会显示新版本（如果有的话）。 |
| `bower init` | 将项目设置为 Bower 项目。该命令还会创建 `bower.json`。 |
| `bower uninstall <library-name>` | 从项目中移除库名称。 |
| `bower version <library-name>` | 检索已安装的库版本。 |

### 注

你可以运行 `bower --help` 来获取完整的命令列表。

## 快速测验 - 网站开发工具和命令行

Q1\. 我们刚刚安装了 Sublime Text 和 Package Control。Package Control 用于什么？

1.  用于轻松安装和移除 Sublime Text 包。

1.  用于安装 LESS 和 Sass/SCSS 包。

1.  用于在 Sublime Text 中管理包。

Q2\. 我们还安装了 XAMPP。我们为什么需要安装 XAMPP？

1.  用于本地托管网站。

1.  用于本地开发网站。

1.  用于本地项目管理。

# 总结

在本章中，我们安装了 Sublime Text、XAMPP、Koala 和 Bower。所有这些工具将有助于我们构建网站。现在我们已经准备好了工具，我们可以开始着手项目了。所以，让我们继续下一章，开始第一个项目。
