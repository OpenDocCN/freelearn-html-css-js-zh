# 前言

本书的主要重点是使用 JavaScript 在真实的 Web 应用程序中应用数据结构和算法。

随着 JavaScript 进入服务器端，并且单页应用程序（SPA）框架接管客户端，很多，如果不是全部，业务逻辑都被移植到了客户端。这使得使用手工制作的数据结构和算法对于特定用例至关重要。

例如，在处理数据可视化（如图表、图形和 3D 或 4D 模型）时，可能会有数以万计甚至数十万个复杂对象从服务器提供，有时几乎是实时的。处理这些数据的方式有多种，这就是我们将要探讨的，配以真实世界的例子。

# 这本书适合谁

这本书适合对 HTML、CSS 和 JavaScript 有兴趣和基本知识的任何人。我们还将使用 Node.js、Express 和 Angular 来创建一些利用我们的数据结构的 Web 应用程序和 API。

# 本书涵盖了什么

第一章，“构建堆栈管理应用程序状态”，介绍了构建和使用堆栈，例如为应用程序创建自定义返回按钮以及在线 IDE 的语法解析器和评估器。

第二章，“为顺序执行创建队列”，演示了使用队列及其变体来创建一个能够处理消息失败的消息服务。然后，我们对不同类型的队列进行了快速比较。

第三章，“使用集合和映射加速应用程序”，使用集合和映射创建键盘快捷方式以在应用程序状态之间导航。然后，我们创建了一个自定义应用程序跟踪器，用于记录 Web 应用程序的分析信息。最后，我们对集合和映射与数组和对象进行了性能比较。

第四章，“使用树加速查找和修改”，利用树数据结构构建了一个自动完成组件。然后，我们创建了一个信用卡批准预测器，根据历史数据确定信用卡申请是否会被接受。

第五章，“使用图简化复杂应用程序”，讨论了图，并附有示例，例如为职业门户创建参考生成器以及在社交媒体网站上的朋友推荐系统。

第六章，“探索各种类型的算法”，探讨了一些最重要的算法，如 Dijkstra 算法、0/1 背包问题、贪婪算法等。

第七章，“排序及其应用”，探讨了归并排序、插入排序和快速排序，并附有示例。然后，我们对它们进行了性能比较。

第八章，“大 O 符号、空间和时间复杂度”，讨论了表示复杂性的符号，然后讨论了空间和时间复杂度以及它们如何影响我们的应用程序。

第九章，“微优化和内存管理”，探讨了 HTML、CSS、JavaScript 的最佳实践，然后讨论了 Google Chrome 的一些内部工作原理，以及我们如何利用它更好地和更快地渲染我们的应用程序。

# 充分利用本书

+   JavaScript、HTML 和 CSS 的基本知识

+   已安装 Node.js（[`nodejs.org/en/download/`](https://nodejs.org/en/download)）

+   安装 WebStorm IDE（[`www.jetbrains.com/webstorm/download`](https://www.jetbrains.com/webstorm/download)）或类似软件

+   下一代浏览器，如 Google Chrome ([`www.google.com/chrome/browser/desktop/`](https://www.google.com/chrome/browser/desktop/))

+   熟悉 Angular 2.0 或更高版本是一个优势，但不是必需的

+   本书中的屏幕截图是在 macOS 上拍摄的。对于任何其他操作系统的用户，可能会有一些差异（如果有的话）。但是，无论操作系统如何，代码示例都将运行而不会出现任何差异。在任何我们指定`CMD/cmd/command`的地方，请在 Windows 对应的地方使用`CTRL/ctrl/control`键。如果看到`return`，请使用*Enter*，如果看到术语`terminal/Terminal`，请在 Windows 上使用其等效的`command prompt`。

+   在本书中，代码库是随着主题的进展逐步构建的。因此，当您将代码示例的开头与 GitHub 中的代码库进行比较时，请注意 GitHub 中的代码是您所参考的主题或示例的最终形式。

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，文件将直接发送到您的邮箱。

您可以按照以下步骤下载代码文件：

1.  登录或注册[www.packtpub.com](http://www.packtpub.com/support)。

1.  选择 SUPPORT 选项卡。

1.  点击 Code Downloads & Errata。

1.  在搜索框中输入书名，然后按照屏幕上的说明操作。

下载文件后，请确保使用最新版本的解压缩软件解压文件夹：

+   WinRAR/7-Zip 适用于 Windows

+   Zipeg/iZip/UnRarX 适用于 Mac

+   7-Zip/PeaZip 适用于 Linux

该书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Practical-JavaScript-Data-Structures-and-Algorithms`](https://github.com/PacktPublishing/Hands-On-Data-Structures-and-Algorithms-with-JavaScript)。我们还有来自丰富书籍和视频目录的其他代码包可供下载，网址为**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。去看看吧！

# 下载彩色图像

我们还提供了一个 PDF 文件，其中包含本书中使用的屏幕截图/图表的彩色图像。您可以在这里下载它：

[`www.packtpub.com/sites/default/files/downloads/HandsOnDataStructuresandAlgorithmswithJavaScript_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/HandsOnDataStructuresandAlgorithmswithJavaScript_ColorImages.pdf)。

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄。这是一个例子：“本地数组操作的时间复杂度各不相同。让我们来看一下`Array.prototype.splice`和`Array.prototype.push`。”

代码块设置如下：

```js
class Stack {
    constructor() {

    }
}
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目将以粗体显示：

```js
var express = require('express');
var app = express();
var data = require('./books.json');
var Insertion = require('./sort/insertion');
```

任何命令行输入或输出都以以下形式书写：

```js
ng new back-button
```

**粗体**：表示新术语、重要单词或您在屏幕上看到的单词。例如，菜单或对话框中的单词会以这种形式出现在文本中。这是一个例子：“当用户点击`back`按钮时，我们将从堆栈中导航到应用程序的上一个状态。”

警告或重要提示会出现在这样的形式中。提示和技巧会出现在这样的形式中。
