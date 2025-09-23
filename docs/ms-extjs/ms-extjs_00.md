# 前言

如果你是一名 Ext JS 开发者，你可能花费了不少时间来学习这个框架。我们知道 Ext JS 的学习曲线并不短。在我们掌握了基础知识之后，当我们需要在日常工作中使用 Ext JS 时，会涌现出很多问题：一个组件如何与另一个组件通信？最佳实践是什么？使用这种方法而不是另一种方法是否真的值得？还有没有其他方式可以实现相同的功能？这是正常的。

本书是考虑到这些开发者而编写的。

因此，这就是本书的主题：我们如何将所有内容整合在一起，并使用 Ext JS 创建真正出色的应用程序？我们将创建一个完整的应用程序，从屏幕草图到将其投入生产。我们将创建应用程序结构、启动画面、登录页面、多语言功能、活动监视器、依赖于用户权限的动态菜单以及用于管理数据库信息（简单和复杂信息）的模块。然后，我们将学习如何为生产构建应用程序、如何自定义主题以及如何调试它。

我们将使用现实世界的示例，看看我们如何使用 Ext JS 组件来实现它们。在整个书中，我们还包含了很多技巧和最佳实践，以帮助您提升 Ext JS 知识并达到新的水平。

# 本书涵盖的内容

第一章, *Sencha Ext JS 概述*，介绍了 Sencha Ext JS 及其功能。本章提供了在深入阅读本书其他章节之前可以阅读的参考资料。这是考虑到这可能是你第一次接触这个框架而进行的。

第二章, *入门*，介绍了本书中实现的应用程序，其功能和每个屏幕及模块的草图（每个章节涵盖不同的模块），并展示了如何使用 Sencha Cmd 创建应用程序的结构以及如何创建启动画面。

第三章, *登录页面*，解释了如何使用 Ext JS 创建登录页面以及如何在服务器端处理它，并展示了额外的功能，例如添加大写锁定警告消息以及在按下 *Enter* 键时提交登录页面。

第四章, *退出和多语言功能*，涵盖了如何创建退出功能以及客户端活动监视器超时，这意味着如果用户没有使用鼠标或按下键盘上的任何键，系统将自动结束会话并退出。本章还提供了一个多语言功能的示例，并展示了如何创建一个组件，用户可以使用它来更改系统的语言和区域设置。

第五章，*高级动态菜单*，是关于如何创建一个依赖于用户权限的动态菜单。菜单选项的渲染取决于用户是否有权限；如果没有，则选项将不会显示。

第六章，*用户管理*，解释了如何创建一个屏幕来列出已经访问系统的所有用户。

第七章，*静态数据管理*，涵盖了如何实现一个模块，用户能够像直接从 MySQL 表编辑信息一样编辑信息。本章还探讨了诸如实时搜索、筛选和内联编辑（使用单元格编辑和行编辑插件）等功能。此外，我们开始探讨使用 Ext JS 开发大型应用程序时遇到的真实世界问题，例如在整个应用程序中组件的重用。

第八章，*内容管理*，进一步探讨了从数据库表及其与其他表的所有关系来管理信息复杂性。因此，我们介绍了如何管理复杂信息以及如何在数据网格和表单面板中处理关联。

第九章，*添加额外功能*，涵盖了如何添加诸如打印以及将内容导出为 PDF 和 Excel 等功能，这些功能不是 Ext JS 本地支持的。本章还涵盖了图表以及如何将它们导出为图像和 PDF，以及如何使用第三方插件。

第十章，*路由、触摸支持和调试*，演示了如何在项目中启用路由；它还涉及调试 Ext JS 应用程序，包括我们需要注意的事项以及为什么了解如何调试非常重要。我们还简要讨论了将 Ext JS 项目转换为移动应用（响应式设计和触摸支持），一些有助于您作为开发者日常工作的有用工具，以及一些关于在哪里找到额外和开源插件以用于 Ext JS 项目的推荐。

第十一章，*准备生产和主题*，涵盖了如何自定义主题和创建自定义用户界面。它还探讨了将应用程序打包到生产所需的步骤以及这样做的好处。

# 您需要为此书准备的内容

以下是在执行本书示例之前需要安装的软件列表。以下列表涵盖了实现和执行本书示例所使用的确切软件，但您可以使用任何具有相同功能的已安装类似软件。

对于带有调试工具的浏览器，请使用以下方法：

+   带有 Firebug 的 Firefox：[`www.mozilla.org/firefox/`](https://www.mozilla.org/firefox/) 和 [`getfirebug.com/`](http://getfirebug.com/)

+   Google Chrome：[`www.google.com/chrome`](http://www.google.com/chrome)

对于支持 PHP 的 Web 服务器，请使用以下：

+   Xampp：[`www.apachefriends.org/en/xampp.html`](http://www.apachefriends.org/en/xampp.html)

对于数据库，请使用以下：

+   MySQL：[`dev.mysql.com/downloads/mysql/`](http://dev.mysql.com/downloads/mysql/)

+   MySQL Workbench：[`dev.mysql.com/downloads/tools/workbench/`](http://dev.mysql.com/downloads/tools/workbench/)

+   MySQL Sakila 示例数据库：[`dev.mysql.com/doc/index-other.html`](http://dev.mysql.com/doc/index-other.html) 和 [`dev.mysql.com/doc/sakila/en/index.html`](http://dev.mysql.com/doc/sakila/en/index.html)

对于 Sencha Cmd 和所需的工具，请使用以下：

+   Sencha Cmd：[`www.sencha.com/products/sencha-cmd/download`](http://www.sencha.com/products/sencha-cmd/download)

+   Ruby 1.8 或 1.9：[`www.ruby-lang.org/en/downloads/`](http://www.ruby-lang.org/en/downloads/)

+   Sass：[`sass-lang.com/`](http://sass-lang.com/)

+   Compass：[`compass-style.org/`](http://compass-style.org/)

+   Java JDK（版本 7 或更高）：[`www.oracle.com/technetwork/java/javase/downloads/index.html`](http://www.oracle.com/technetwork/java/javase/downloads/index.html)

+   Java 环境变量：[`docs.oracle.com/javase/tutorial/essential/environment/paths.html`](http://docs.oracle.com/javase/tutorial/essential/environment/paths.html)

+   Apache ANT：[`ant.apache.org/bindownload.cgi`](http://ant.apache.org/bindownload.cgi)

+   Apache ANT 环境变量：[`ant.apache.org/manual/install.html`](http://ant.apache.org/manual/install.html)

+   当然，还有 Ext JS：[`www.sencha.com/products/extjs/`](http://www.sencha.com/products/extjs/)

本书将使用 Ext JS 5.0.1。

# 本书面向对象

如果您是一位熟悉 Ext JS 的开发者，并希望提升您的技能以创建更好的 Web 应用程序，这本书就是为您准备的。需要具备 JavaScript/HTML/CSS 的基本知识以及任何服务器端语言（PHP、Java、C#、Ruby 或 Python）的基础知识。

# 习惯用法

在本书中，您将找到许多用于区分不同类型信息的文本样式。以下是一些这些样式的示例及其含义的解释。

文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称应如下所示：“如果我们想创建一个表示客户详情的类，我们可以将其命名为 `ClientDetails`。”

代码块应如下设置：

```js
Ext.define('Packt.model.film.Film', {
    extend: 'Packt.model.staticData.Base', //#1
```

当我们希望引起您对代码块中特定部分的注意时，相关的行或项目将以粗体显示：

```js
Ext.application({
    name: 'Packt',

    extend: 'Packt.Application',

 autoCreateViewport: 'Packt.view.main.Main'
});
```

任何命令行输入或输出都应如下所示：

```js
sencha generate app Packt ../masteringextjs

```

**新术语**和**重要词汇**将以粗体显示。屏幕上显示的词汇，例如在菜单或对话框中，将以如下方式显示：“滚动到页面底部并选择**开源 GPL 许可**。”

### 注意

警告或重要提示将以如下框显示。

### 小贴士

小贴士和技巧将以如下方式显示。

# 读者反馈

我们始终欢迎读者的反馈。告诉我们您对这本书的看法——您喜欢或不喜欢什么。读者反馈对我们很重要，因为它帮助我们开发出您真正能从中获得最大收益的标题。

要发送一般反馈，请简单地将电子邮件发送到 `<feedback@packtpub.com>`，并在邮件主题中提及书籍的标题。

如果您在某个主题上具有专业知识，并且您对撰写或为书籍做出贡献感兴趣，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在，您已经成为 Packt 书籍的骄傲拥有者，我们有一些事情可以帮助您从您的购买中获得最大收益。

## 下载示例代码

您可以从[`www.packtpub.com`](http://www.packtpub.com)下载示例代码文件，这些文件适用于您购买的所有 Packt 出版社书籍。如果您在其他地方购买了这本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便将文件直接通过电子邮件发送给您。

## 勘误

尽管我们已经尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何勘误，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书籍，点击**勘误提交表单**链接，并输入您的勘误详情来报告。一旦您的勘误得到验证，您的提交将被接受，勘误将被上传到我们的网站或添加到该标题的勘误部分下的现有勘误列表中。

要查看之前提交的勘误，请访问[`www.packtpub.com/books/content/support`](https://www.packtpub.com/books/content/support)，并在搜索字段中输入书籍名称。所需信息将出现在**勘误**部分下。

## 盗版

互联网上版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现任何形式的我们作品的非法副本，请立即提供位置地址或网站名称，以便我们可以寻求补救措施。

请通过以下链接联系我们 `<copyright@packtpub.com>`，并提供涉嫌盗版材料的链接。

我们感谢您在保护我们作者和为我们提供有价值内容的能力方面所提供的帮助。

## 问题和答案

如果您对本书的任何方面有问题，您可以联系我们的 `<questions@packtpub.com>`，我们将尽力解决问题。
