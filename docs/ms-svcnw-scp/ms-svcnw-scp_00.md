# 前言

ServiceNow 正逐渐成为 IT 服务管理的首选平台。像 RedHat 和 NetApp 这样的行业巨头已经采用 ServiceNow 来满足他们的运营需求。ServiceNow 在其基线实例中为客户端提供了额外的附加功能，因为脚本可以用来自定义和改进实例的性能。ServiceNow 提供内置的 JavaScript API 用于脚本编写和通过 JavaScript 改进实例。

本书将首先介绍 ServiceNow 脚本的基础知识以及何时在 ServiceNow 环境中编写脚本。然后，我们将深入探讨使用 JavaScript API 的客户端和服务器端脚本。我们还将涵盖诸如按需函数、脚本操作和最佳实践等高级概念。本书将作为编写、测试和调试 ServiceNow 脚本的端到端指南。我们还将涵盖用于在 ServiceNow 实例之间移动自定义设置的更新集、用于创建自定义页面的 Jelly 脚本以及 ServiceNow 中所有类型脚本的最佳实践。

在本书结束时，您将获得使用内置 JavaScript API 在 ServiceNow 中编写脚本的实践经验。

# 本书面向的对象

本书的目标读者是 ServiceNow 管理员或任何愿意学习用于脚本编写和自定义 ServiceNow 实例的内置 JavaScript API 的利益相关者。拥有使用 ServiceNow 的经验是强制性的。

# 本书涵盖的内容

第一章，*入门*，介绍了 ServiceNow 脚本的基础知识。也就是说，何时以及为什么通过脚本开发自定义功能是合适的。它还介绍了何时进行配置和自定义。

第二章，*探索 ServiceNow Glide 类*，为您提供了 ServiceNow 如何公开其 JavaScript API 的详细信息，这使得您能够方便地编写脚本。使用这些公开的 API，您可以执行各种数据库操作。您将探索一些常用的服务器端 glide 类和客户端 glide 类。

第三章，*客户端脚本编程简介*，帮助您理解 ServiceNow 的客户端脚本。您将学习关于客户端脚本和 UI 策略的概念。您将了解如何编写和测试基本的客户端脚本。您将通过一些客户端脚本的实践示例来更好地理解其功能。

第四章，*高级客户端脚本编程*，将帮助您理解 ServiceNow 中客户端脚本的更高级方面。您将学习关于 AJAX 调用和 UI 操作的知识。您将通过一些客户端脚本的高级实践示例来更好地理解其功能。

第五章，*服务器端脚本简介*，涵盖了 ServiceNow 中服务器端脚本的详细信息。这将帮助您深入理解业务规则、UI 操作和访问控制的概念。您还将学习如何编写和测试服务器端脚本。您将通过一些服务器端脚本的实际示例来更好地理解其功能。

第六章，*高级服务器端脚本*，将涵盖 ServiceNow 中服务器端脚本的高级方面。它将帮助您理解脚本包含、后台脚本、工作流脚本和计划任务的概念。您将通过一些高级服务器端脚本的实际示例来理解脚本调用、系统调度程序和系统中的排队事件。

第七章，*自定义页面简介*，向您介绍了 Jelly。这将为您揭示 Jelly 在 ServiceNow 中的应用。您还将学习如何使用 Jelly 脚本用 JavaScript 创建 UI 页面。

第八章，*使用 Jelly 脚本*，进一步扩展了 Jelly 的知识。这将进一步揭示 Jelly 脚本在 ServiceNow 中的应用。您还将学习如何创建 UI 宏以增强 UI 页面。

第九章，*调试脚本*，介绍了在 ServiceNow 中调试脚本的方法。您将探索在 ServiceNow 中用于故障排除和调试代码的各种工具和方法。

第十章，*最佳实践*，涵盖了开发人员应遵循的各种最佳实践，以高效地使用 ServiceNow。它还讨论了记录和监控系统性能以控制 ServiceNow 环境。

第十一章，*使用更新集进行部署*，指导您如何将配置和自定义设置从实例迁移到实例。它还帮助您了解在处理全局和范围应用时如何使用更新集。您还将学习如何在处理更新集时避免一些常见陷阱。

第十二章，*使用 ServiceNow 脚本构建自定义应用程序*，为您提供了 ServiceNow 中脚本的端到端实现。您将学习如何使用 ServiceNow 提供的脚本构建自定义应用程序。

# 为了充分利用这本书，

在开始学习 ServiceNow 脚本精通之前，建议您对 ServiceNow 平台有一个良好的理解。建议您拥有系统管理员认证或类似知识，并了解 ServiceNow 中的表单、列表和表格。您还应能够轻松地在 ServiceNow 实例中导航。

对 JavaScript 有一些了解将很有帮助，但不是强制性的，因为提供的示例将为您提供尝试并开始编写的脚本。

建议您拥有一个 ServiceNow 实例，以便尝试示例并创建一些自己的脚本。如果您目前没有实例，ServiceNow 可以为希望提高 ServiceNow 技能的用户提供个人开发者实例（PDI）。

# 下载示例代码文件

您可以从[www.packtpub.com](http://www.packtpub.com)的账户下载本书的示例代码文件。如果您在其他地方购买了此书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

您可以通过以下步骤下载代码文件：

1.  在[www.packtpub.com](http://www.packtpub.com/support)上登录或注册。

1.  选择“支持”选项卡。

1.  点击“代码下载与勘误”。

1.  在搜索框中输入书名，并遵循屏幕上的说明。

文件下载完成后，请确保使用最新版本的以下工具解压或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

本书代码包也托管在 GitHub 上，网址为[`github.com/PacktPublishing/Mastering-ServiceNow-Scripting`](https://github.com/PacktPublishing/Mastering-ServiceNow-Scripting)。如果代码有更新，它将在现有的 GitHub 仓库中更新。

我们还有其他来自我们丰富图书和视频目录的代码包，可在**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**上找到。查看它们吧！

# 下载彩色图像

我们还提供包含本书中使用的截图/图表彩色图像的 PDF 文件。您可以从这里下载：[`www.packtpub.com/sites/default/files/downloads/MasteringServiceNowScripting_ColorImages.pdf`](https://www.packtpub.com/sites/default/files/downloads/MasteringServiceNowScripting_ColorImages.pdf)。

# 使用的约定

本书使用了多种文本约定。

`CodeInText`：表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“`onLoad`客户端脚本类型在表单加载时运行脚本。”

代码块设置如下：

```js
function onChange(control, oldValue, newValue, isLoading, isTemplate) {
    if (isLoading || newValue === '') {
        return;
    }

    //Type appropriate comment here, and begin script below

}
```

**粗体**：表示新术语、重要单词或您在屏幕上看到的单词。例如，菜单或对话框中的单词在文本中显示如下。以下是一个示例：“要访问工作室，请导航到系统应用 | 工作室。”

警告或重要注意事项看起来像这样。

小贴士和技巧看起来像这样。

# 联系我们

我们欢迎读者反馈。

**一般反馈**：请通过`feedback@packtpub.com`发送电子邮件，并在邮件主题中提及书籍标题。如果您对本书的任何方面有疑问，请通过`questions@packtpub.com`发送电子邮件给我们。

**勘误**：尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将非常感激您能向我们报告。请访问[www.packtpub.com/submit-errata](http://www.packtpub.com/submit-errata)，选择您的书籍，点击勘误提交表单链接，并输入详细信息。

**盗版**：如果您在互联网上发现我们作品的任何非法副本，我们将非常感激您能提供位置地址或网站名称。请通过`copyright@packtpub.com`与我们联系，并附上材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com/)。

# 评论

请留下评论。一旦您阅读并使用过这本书，为何不在您购买它的网站上留下评论？潜在读者可以查看并使用您的客观意见来做出购买决定，Packt 公司可以了解您对我们产品的看法，我们的作者也可以看到他们对书籍的反馈。谢谢！

如需更多关于 Packt 的信息，请访问 [packtpub.com](https://www.packtpub.com/)。

# 免责声明

本书提供的代码仅供开发场景使用，以进一步了解 ServiceNow 脚本。使用本书提供的代码由读者自行决定。作者和出版社不对任何由此书中的代码引起的损害或负面影响承担责任，也不接受任何责任。
