# 前言

欢迎来到 *Sencha Architect 应用程序开发*。本书的目标是帮助您通过一系列完整的示例和技巧窍门，学习如何使用 Sencha Architect 来提高您开发 Sencha（Ext JS 和 Sencha Touch）应用程序的生产力。

Sencha Architect 是一个 HTML 5 可视化应用程序构建器，允许您通过拖放组件和实时预览应用程序来创建 Sencha Touch（移动）和 Ext JS（桌面）应用程序。在 Sencha Architect 中构建的所有代码都旨在符合最佳实践，这使得构建大型应用程序或简单地学习变得容易。并且所有生成的代码都可以由您手动编写，这展示了这个工具的强大功能。

# 本书涵盖的内容

第一章, *介绍 Sencha Architect*，介绍了 Sencha Architect 及其界面功能，如项目基础、工具箱、配置面板、项目检查器和设计画布。

第二章, *创建一个 Ext JS 应用程序*，演示了如何使用 MVC 架构和 Sencha Architect 创建一个完整的 Ext JS 应用程序，以及如何首次执行项目。

第三章, *创建一个 Sencha Touch 应用程序*，介绍了如何使用 MVC 架构和 Sencha Architect 创建一个完整的 Sencha Touch 应用程序。本章还演示了如何将 Sencha Touch 应用程序与 Phonegap 集成。

第四章, *技巧和窍门*，探讨了可以帮助开发者日常工作中使用 Sencha Architect 的一些有价值的技巧和窍门。涵盖的一些主题包括使用第三方插件、多语言应用程序、创建可重用自定义组件等。

第五章, *处理资源*，解释了如何使用项目检查器中的资源工具箱和资源包。

第六章, *模拟、构建、打包和部署项目/应用程序*，解释了如何通过预览（在浏览器中）、打包、构建和部署来准备应用程序以供生产使用。

# 您需要为本书准备的内容

以下是在执行本书示例之前您需要安装的软件列表。

使用以下浏览器：

+   推荐使用 Google Chrome 执行 Sencha Touch 应用程序 ([`www.google.com/chrome`](http://www.google.com/chrome))

使用以下支持 PHP 的 web 服务器（或您偏好的任何其他 web 服务器）：

+   Xampp ([`www.apachefriends.org/en/xampp.html`](http://www.apachefriends.org/en/xampp.html))

使用以下数据库：

+   MySQL ([`dev.mysql.com/downloads/mysql/`](http://dev.mysql.com/downloads/mysql/)))

+   MySQL Workbench 或其他数据库管理员 ([`dev.mysql.com/downloads/tools/workbench/`](http://dev.mysql.com/downloads/tools/workbench/))

使用以下 Sencha 工具和所需的互补软件：

+   Sencha Cmd ([`www.sencha.com/products/sencha-cmd/download`](http://www.sencha.com/products/sencha-cmd/download))

+   Ruby ([`www.ruby-lang.org/en/downloads/`](http://www.ruby-lang.org/en/downloads/))

+   Sass ([`sass-lang.com/`](http://sass-lang.com/))

+   Compass ([`compass-style.org/`](http://compass-style.org/))

+   Java JDK ([`www.oracle.com/technetwork/java/javase/downloads/index.html`](http://www.oracle.com/technetwork/java/javase/downloads/index.html))

+   Java 环境变量 ([`docs.oracle.com/javase/tutorial/essential/environment/paths.html`](http://docs.oracle.com/javase/tutorial/essential/environment/paths.html))

+   PhoneGap ([`phonegap.com/`](http://phonegap.com/))

+   Ext JS ([`www.sencha.com/products/extjs/`](http://www.sencha.com/products/extjs/))——我们将使用 Ext JS 4.2

+   Sencha Touch ([`www.sencha.com/products/touch/`](http://www.sencha.com/products/touch/))——我们将使用 Sencha Touch 2.2

+   Sencha Architect 2.2 或更高版本 ([`www.sencha.com/products/architect/`](http://www.sencha.com/products/architect/))

使用以下 iOS 工具（可选——仅限 Mac OS）：

+   带有命令行的 XCode ([`developer.apple.com/xcode/`](https://developer.apple.com/xcode/))

使用以下 Android 工具：

+   Eclipse 是可选的，但强烈推荐 ([`www.eclipse.org/downloads/`](http://www.eclipse.org/downloads/))

+   Android SDK ([`developer.android.com/sdk/index.html`](http://developer.android.com/sdk/index.html))

# 这本书面向的对象

这本书是为已经熟悉 Ext JS 和 Sencha Touch（至少具备基本知识）的开发者编写的。对 HTML 编码的了解、CSS 的基本背景、JavaScript 的深厚背景以及 JSON 的基本理解也非常受欢迎。这本书也适合想要了解更多关于 Sencha Architect 及其如何提高生产力的开发者。

# 习惯用法

在这本书中，您将找到许多不同风格的文本，以区分不同类型的信息。以下是一些这些风格的示例及其含义的解释。

文本中的代码单词如下所示："Sencha Architect 适用于 Windows（`.exe`）、Linux（`.run`）和 Mac OS（`.dmg`）用户。"

**新术语**和**重要词汇**将以粗体显示。您在屏幕上看到的单词，例如在菜单或对话框中，将以如下方式显示："接下来，将显示**欢迎来到 Sencha Architect**的屏幕，如下面的截图所示。"

### 注意

警告或重要注意事项将以这样的框显示。

### 小贴士

小贴士和技巧如下所示。

# 读者反馈

我们始终欢迎读者的反馈。告诉我们您对这本书的看法——您喜欢什么或可能不喜欢什么。读者反馈对我们开发您真正能从中获得最大价值的标题非常重要。

要向我们发送一般反馈，只需发送一封电子邮件到`<feedback@packtpub.com>`，并在邮件主题中提及书名。

如果您在某个主题上具有专业知识，并且您对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在您是 Packt 图书的骄傲拥有者，我们有多种方式帮助您从您的购买中获得最大收益。

## 勘误

尽管我们已经尽最大努力确保内容的准确性，错误仍然可能发生。如果您在我们的某本书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以避免其他读者感到沮丧，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情来报告它们。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站，或添加到该标题的勘误部分下的现有勘误列表中。您可以通过从[`www.packtpub.com/support`](http://www.packtpub.com/support)选择您的标题来查看任何现有勘误。

## 侵权

互联网上版权材料的侵权是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现任何我们作品的非法副本，无论形式如何，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过`<copyright@packtpub.com>`与我们联系，并提供疑似侵权材料的链接。

我们感谢您在保护我们作者和我们为您提供有价值内容的能力方面的帮助。

## 询问

如果您在本书的任何方面遇到问题，可以通过`<questions@packtpub.com>`联系我们，我们将尽力解决。
