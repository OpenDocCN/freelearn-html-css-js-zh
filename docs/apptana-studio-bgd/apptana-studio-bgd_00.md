# 前言

Aptana Studio 是一个功能强大的开源集成开发环境（IDE），专注于构建 Web 应用程序。Aptana Studio 自 2008 年以来一直存在。它通过插件提供对 HTML、CSS、JavaScript、Ruby、Rails、PHP、Python 等多种语言的支持。自 3.0.4 版本以来，Aptana Studio 的开发团队已经集成了最新的 HTML5 和 CSS3 规范。这使得大多数现代浏览器的能力可以用于 Aptana Studio 的开发。最新版本已被下载超过 600 万次。

Aptana Studio 还包括其他任务，如 FTP 和 Git 集成、JavaScript 库和 JavaScript 调试。此外，还包括专门用于处理 AJAX 应用程序和网站的 Aptana Jaxer Web 服务器。

Aptana Studio 基于 Java 平台的 Eclipse，因此它是一个跨平台软件，可以在常见的操作系统上运行，如 Linux、Mac OS-X 和 Windows。

### 提示

**什么是 Eclipse SDK？**

Eclipse 软件开发工具包（SDK）是一个完全用 Java 编写的开源项目，由 IBM 公司于 2001 年启动。

由于 Aptana Studio 基于 Eclipse，因此可以将其作为 Eclipse 插件或独立版本安装。已经使用过 Eclipse IDE 的经验用户可以在现有的 Eclipse 安装中集成 Eclipse 插件。经验较少的用户可以安装独立版本，因为它可以在没有 Eclipse 安装的情况下运行。

但为什么 Aptana Studio 适合 Web 开发？

Aptana Studio 允许您使用单一环境开发和测试您的整个 Web 应用程序。

Aptana Studio 的一些核心功能如下：

+   HTML、CSS、JavaScript 等的代码辅助。它还支持最新的 HTML5 和 CSS3 规范，并包含有关主要网络浏览器中支持级别的信息。

+   JavaScript 调试器集成。

+   FTP、SFTP 和 FTPS 集成为您提供了远程开发的可能性。

+   Git 集成使您能够使用 Git 源代码控制来管理您的项目。

让我们看看使用 Aptana Studio 进行 Web 开发有多简单。

# 本书涵盖的内容

第一章，*入门*，向您展示如何将完全可操作的 Aptana Studio 版本安装到您的系统上，如何执行系统更新或集成新插件。到本章结束时，您的 Aptana Studio 版本应该完全可用。

第二章，*基础知识和如何使用视角和视图*，讨论了 Aptana Studio 的基本功能，以及如何使用视角和视图。到本章结束时，您应该能够根据您的需求修改 Aptana Studio 的外观以进行优化。

第三章，*使用工作空间和项目*，主要关于在项目中创建和配置你的源代码，并将这些项目分组到有用的工作空间中。

第四章，*调试 JavaScript*，教你如何调试你的 JavaScript 应用程序，以及如何尽可能快地找到错误。

第五章，*代码文档和内容辅助*，展示了如何以最佳方式记录你的代码，以便每个开发团队成员都能理解功能，并且 Aptana Studio 构建器能够从你的源代码中读取更多信息。

第六章，*使用 Firebug 检查代码*，探讨了如何检查你的源代码，并帮助你理解为什么你的 Web 应用程序看起来和表现是这样的。

第七章，*使用 JavaScript 库*，提供了如何将 jQuery 或 Dojo 工具包等 JavaScript 库集成到项目中的详细指南。

第八章，*通过 FTP 远程工作*，指导你如何在 Web 服务器上远程使用 FTP。

第九章，*使用 SVN 和 Git 进行协作工作*，帮助你了解如何使用 Aptana Studio 与 Subversion 或 GitHub 一起，与你的开发团队一起开发大型项目。

第十章，*PHP 项目*，教你如何创建和配置 PHP 项目，以开发你的 Web 应用程序的后端。

第十一章，*优化工作和提高协作*，探讨了优化工作流程的各种可能性。

第十二章，*故障排除*，讨论了开发人员在开发过程中使用 Aptana Studio 时遇到的最常见问题。

# 阅读本书所需的条件

本书的所有章节都在以下软件设置上进行了测试和验证：

+   Ubuntu/Debian/LinuxMint Linux，带有 3.0.x 内核和 Gnome3.2

+   Aptana Studio 3.0.6（本书从这里开始，并逐步通过版本升级到 Aptana Studio 3.3.1）

因此，你需要的只是带有互联网连接的工作站来下载 Aptana Studio 安装包，以及部署你的 Web 应用程序，或者通过 FTP、SVN 或 Git 远程工作。

如果你使用的是 Apple 或 Windows 操作系统，不要害怕。整个系统在不同操作系统上看起来都一样，但操作系统之间的 GUI 元素可能略有不同。

# 本书面向的对象

本书是 Aptana Studio 初学者和经验丰富的 Web 开发者的完美入门指南。

如果您已经对 Aptana Studio 或 Eclipse IDE 有些熟悉，这本书将帮助您了解 Aptana Studio 如何优化您在大型 Web 应用程序和项目上的日常工作。

总而言之，这本书是配置整个开发环境的完整指南，以获得您工作的最佳效果。

# **约定**

在本书中，您将找到许多不同风格的文本，以区分不同类型的信息。以下是一些这些样式的示例及其含义的解释。

文本中的代码词如下所示：“我们可以通过使用`include`指令来包含其他上下文。”

代码块应如下设置：

```js
function someLoopWithinAFunction(loopEnd) {
    var someResult = 0;
    for(var i=0 ; i<loopEnd ; i++) {
        someResult += i;
        aptana.log("i: " +i); // Add a breakpoint here
    }
    return someResults)
```

任何命令行输入或输出都应如下所示：

```js
sudo rm -r /opt/Aptana\ Studio\3
```

**新术语**和**重要词汇**将以粗体显示。你在屏幕上看到的，例如在菜单或对话框中的文字，将以如下方式显示：“点击**下一个**按钮将您带到下一屏幕”。

### **注意**

警告或重要注意事项将以如下框中的方式显示。

### **提示**

**技巧和窍门**将以如下方式显示。

# **读者反馈**

我们始终欢迎读者的反馈。请告诉我们您对这本书的看法——您喜欢什么或可能不喜欢什么。读者的反馈对我们开发您真正能从中受益的标题非常重要。

要发送给我们一般性的反馈，只需发送电子邮件到`<feedback@packtpub.com>`，并在您的邮件主题中提及书名。

如果您在某个主题上具有专业知识，并且您对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# **客户支持**

现在，您已经成为 Packt 图书的骄傲拥有者，我们有许多事情可以帮助您从您的购买中获得最大收益。

## **下载示例代码**

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)的账户下载您购买的所有 Packt 图书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)，注册以直接将文件通过电子邮件发送给您。

## **勘误表**

尽管我们已经尽一切努力确保我们内容的准确性，但错误仍然可能发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/support`](http://www.packtpub.com/support)，选择您的书籍，点击**勘误****提交****表**链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站，或添加到该标题的**勘误表**部分下的现有勘误列表中。

## **盗版**

在互联网上，版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现我们作品的任何非法副本，无论形式如何，请立即提供位置地址或网站名称，以便我们可以追究补救措施。

请通过 `<copyright@packtpub.com>` 联系我们，并提供涉嫌盗版材料的链接。

我们感谢您在保护我们作者以及为我们提供有价值内容方面的帮助。

## 问题

如果您在本书的任何方面遇到问题，可以通过 `<questions@packtpub.com>` 联系我们，我们将尽力解决。
