# 前言

从在跑道上闲置到在天空中飞行，本书将向您介绍使用 Sequelize 和 MySQL 对 Node.js 应用程序进行数据库交互的世界，从生成架构和适合航空公司到在云端部署预订航班的应用程序。

涵盖了诸如事件生命周期、关联、事务和连接池等概念，以帮助您在开发下一个应用程序时从开始到结束。本书结束时，您将能够熟练并自信地使用 Sequelize 在数据库管理系统和 Node.js 应用程序之间创建、删除和转换数据。

# 本书面向的对象

本书面向初学者到中级 JavaScript 开发者，他们对于创建 Node.js 应用程序是新手，并希望将其数据库附加到他们的 Web 应用程序中。拥有 SQL 知识是加分项，但不是理解本书内容的先决条件。

# 本书涵盖的内容

*第一章*, *Sequelize 和 Node.js 中 ORM 的介绍*，涵盖了为本书课程安装必要的先决条件。

*第二章*, *定义和使用 Sequelize 模型*，涵盖了绘制数据库架构以及读取或写入它的内容。

*第三章*, *验证模型*，介绍了如何确保模型数据的完整性。

*第四章*, *关联模型*，将帮助您了解创建模型之间关系的基本知识和优势。

*第五章*, *将钩子和生命周期事件添加到您的模型中*，将通过实际应用示例介绍生命周期事件的操作顺序。

*第六章*, *使用 Sequelize 实现事务*，介绍了使用不同的隔离和锁定级别封装事务查询。

*第七章*, *处理自定义、JSON 和 Blob 数据类型*，介绍了在关系型数据库中使用文档化和杂项存储。

*第八章*, *记录和监控您的应用程序*，帮助您识别应用程序中的问题和瓶颈。

*第九章*, *使用和创建适配器*，介绍了如何扩展、添加和集成 Sequelize 库以创建新的工具和平台。

*第十章*, *部署 Sequelize 应用程序*，介绍了如何将 Avalon Airlines 项目部署到云应用平台，如 Heroku。

# 为了最大限度地利用本书

所有代码示例均已在 macOS 和 Linux 上使用 Node.js 16、MySQL 5.7 和 Sequelize 6 进行测试。然而，代码库仍然应该与未来的版本发布兼容。

| **本书涵盖的软件/硬件** | **操作系统要求** |
| --- | --- |
| Node.js 16 | Windows、macOS 或 Linux |
| MySQL 5.7 | Windows, macOS, 或 Linux |
| Sequelize 6 | Windows, macOS, 或 Linux |

**如果您正在使用本书的数字版，我们建议您亲自输入代码或从书的 GitHub 仓库（下一节中有一个链接）获取代码。这样做将帮助您避免与代码复制和粘贴相关的任何潜在错误。**

# 下载示例代码文件

您可以从 GitHub 下载本书的示例代码文件 [`github.com/PacktPublishing/Supercharging-Node.js-Application-with-Sequelize`](https://github.com/PacktPublishing/Supercharging-Node.js-Application-with-Sequelize)。如果代码有更新，它将在 GitHub 仓库中更新。

我们还有其他来自我们丰富图书和视频目录的代码包可供在 [`github.com/PacktPublishing/`](https://github.com/PacktPublishing/) 获取。查看它们吧！

# 下载彩色图像

我们还提供了一份包含本书中使用的截图和图表彩色图像的 PDF 文件。您可以从这里下载：[`packt.link/FqVKp`](https://packt.link/FqVKp)。

# 使用的约定

本书中使用了多种文本约定。

`文本中的代码`: 表示文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 昵称。以下是一个示例：“将下载的`WebStorm-10*.dmg`磁盘映像文件挂载为系统中的另一个磁盘。”

代码块设置如下：

```js
models.sequelize.sync({
    force: true,
    logging: false
})
```

当我们希望您注意代码块中的特定部分时，相关的行或项目将以粗体显示：

```js
// INT(4)
var unsignedInteger = DataTypes.NUMBER({
    length: 4,
    zerofill: false,
    unsigned: true,
});
```

任何命令行输入或输出都应如下编写：

```js
sudo apt-get update
sudo apt install mysql-server
```

**粗体**: 表示新术语、重要单词或您在屏幕上看到的单词。例如，菜单或对话框中的单词以粗体显示。以下是一个示例：“我们将想要选择**开发者默认**和**安装所有产品**选项。”

小贴士或重要提示

看起来是这样的。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**: 如果您对本书的任何方面有疑问，请通过电子邮件发送给我们 customercare@packtpub.com，并在邮件主题中提及书名。

**勘误**: 尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果您在这本书中发现了错误，我们将非常感激您能向我们报告。请访问 [www.packtpub.com/support/errata](http://www.packtpub.com/support/errata) 并填写表格。

**盗版**: 如果您在互联网上以任何形式遇到我们作品的非法副本，我们将非常感激您能提供位置地址或网站名称。请通过电子邮件发送给我们 copyright@packt.com，并附上材料的链接。

**如果您有兴趣成为作者**：如果您在某个领域有专业知识，并且您有兴趣撰写或为书籍做出贡献，请访问[authors.packtpub.com](http://authors.packtpub.com)。

# 分享您的想法

一旦您阅读了*用 Sequelize 提升 Node.js 应用程序性能*，我们非常乐意听听您的想法！请[点击此处直接访问此书的亚马逊评论页面](https://packt.link/r/1801811555)并分享您的反馈。

您的评论对我们和科技社区都非常重要，它将帮助我们确保我们提供高质量的内容。

# 第一部分 – 安装、配置和基础知识

在本部分，您将学习如何为您的操作系统安装和配置 Sequelize，以及如何从数据库中插入、删除、更新和查询数据。

本部分包括以下章节：

+   *第一章*，*Node.js 中的 Sequelize 和 ORM 简介*

+   *第二章*，*定义和使用 Sequelize 模型*
