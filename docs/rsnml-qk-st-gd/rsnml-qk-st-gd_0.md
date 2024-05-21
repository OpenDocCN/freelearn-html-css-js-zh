# 前言

ReactJS 已经改变了我们所知的前端开发世界。它的创造者 Jordan Walke 也创建了 ReasonML 和 ReasonReact 作为 React 的未来。React 对 DOM 的抽象允许强大的编程范式，有助于解决 JavaScript 的可维护性问题，在本书中，我们将深入探讨 Reason 如何帮助您构建更简单，更易维护的 React 应用程序。本书是使用 ReasonML 构建 React 应用程序的实用指南。

# 本书的受众

本书的目标读者是熟悉 ReactJS 的 JavaScript 开发人员。不需要具有静态类型语言的先前经验。

# 本书涵盖的内容

第一章，ReasonML 简介，讨论了当前的 Web 开发状态以及为什么我们会考虑 ReasonML 用于前端开发（以及更多）。

第二章，设置开发环境，让我们开始运行。

第三章，创建 ReasonReact 组件，演示了如何使用 ReasonML 和 ReasonReact 创建 React 组件。在这里，我们开始构建一个应用程序外壳，然后在本书的其余部分进行添加。

第四章，BuckleScript，Belt 和互操作性，让我们全面了解 Reason 的生态系统和标准库。

第五章，有效的 ML，深入探讨了 Reason 类型系统的一些更高级特性，使用了商业示例。

第六章，CSS-in-JS（在 Reason 中），展示了 CSS-in-JS 在 Reason 中的工作原理以及类型系统如何帮助。

第七章，Reason 中的 JSON，演示了如何将 JSON 转换为 Reason 中的数据结构，并说明了 GraphQL 如何帮助。

第八章，使用 Jest 进行单元测试，介绍了流行的 Jest 测试库的测试。

# 为了充分利用本书

您应该熟悉以下内容：

+   命令行界面

+   GitHub 和 Git

+   诸如 Visual Studio Code 之类的文本编辑器

# 下载示例代码文件

您可以从[www.packt.com](http://www.packt.com)的帐户中下载本书的示例代码文件。如果您在其他地方购买了本书，您可以访问[www.packt.com/support](http://www.packt.com/support)并注册，文件将直接发送到您的邮箱。

您可以按照以下步骤下载代码文件：

1.  在[www.packt.com](http://www.packt.com)上登录或注册。

1.  选择“支持”选项卡。

1.  单击“代码下载和勘误”。

1.  在搜索框中输入书名，然后按照屏幕上的说明进行操作。

文件下载后，请确保使用以下最新版本解压或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书的代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/ReasonML-Quick-Start-Guide`](https://github.com/PacktPublishing/ReasonML-Quick-Start-Guide)。如果代码有更新，将在现有的 GitHub 存储库上进行更新。

我们还有其他代码包来自我们丰富的图书和视频目录，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)上找到。去看看吧！

# 使用的约定

本书中使用了许多文本约定。

`CodeInText`：指示文本中的代码词，文件夹名称，文件名，文件扩展名，路径名和变量名。这是一个例子：“运行`npm run build`来将`Demo.re`编译为 JavaScript。”

代码块设置如下：

```js
"warnings": {
  "error": "A"
},
```

当我们希望引起您对代码块的特定部分的注意时，相关的行或项目将以粗体显示：

```js
/* bsconfig.json */
...
"sources": {
  "dir": "src",
  "subdirs": true
},
...
```

任何命令行输入或输出都将按以下方式编写：

```js
bsb -init my-reason-react-app -theme react
cd my-reason-react-app
```

**粗体**：表示一个新术语，一个重要词，或者你在屏幕上看到的词。例如，菜单或对话框中的单词会以这种方式出现在文本中。这是一个例子：“padLeft 的类型是(string, some_variant) => string，其中 some_variant 使用了一个称为**多态变体**的高级类型系统特性，它使用[@bs.unwrap]来转换为 JavaScript 可以理解的东西。”

警告或重要说明会显示为这样。

提示和技巧会显示为这样。
