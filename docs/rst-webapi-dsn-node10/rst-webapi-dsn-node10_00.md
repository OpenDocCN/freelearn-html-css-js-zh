# 前言

RESTful 服务已成为社交服务、新闻订阅和移动设备的事实标准数据提供者。它们向数百万用户提供大量数据。因此，它们需要满足高可用性要求，如可靠性和可扩展性。本书将向您展示如何利用 Node.js 平台实现强大和高性能的数据服务。通过本书，您将学会如何实现一个真实的 RESTful 服务，利用现代 NoSQL 数据库来提供 JSON 和二进制内容。

重要的主题，如正确的 URI 结构和安全功能也有详细的例子，向您展示开始实施强大的 RESTful API 所需的一切。

# 这本书是为谁准备的

这本书的目标读者是想通过学习如何基于 Node.js 平台开发可扩展的服务器端 RESTful 应用程序来丰富他们的开发技能的开发人员。您还需要了解 HTTP 通信概念，并且应该具备 JavaScript 语言的工作知识。请记住，这不是一本教你如何在 JavaScript 中编程的书。了解 REST 将是一个额外的优势，但绝对不是必需的。

# 为了充分利用这本书

1.  告知读者在开始之前需要了解的事项，并明确您所假设的知识

1.  他们需要获取的任何额外安装说明和信息

# 下载示例代码文件

您可以从您的帐户在[www.packtpub.com](http://www.packtpub.com)下载这本书的示例代码文件。如果您在其他地方购买了这本书，您可以访问[www.packtpub.com/support](http://www.packtpub.com/support)并注册，文件将直接发送到您的邮箱。

您可以按照以下步骤下载代码文件：

1.  登录或注册[www.packtpub.com](http://www.packtpub.com/support)。

1.  选择“支持”选项卡。

1.  点击“代码下载和勘误”。

1.  在搜索框中输入书名，然后按照屏幕上的说明操作。

下载文件后，请确保使用最新版本的解压缩或提取文件夹：

+   WinRAR/7-Zip for Windows

+   Zipeg/iZip/UnRarX for Mac

+   7-Zip/PeaZip for Linux

该书的代码包也托管在 GitHub 上**[`github.com/PacktPublishing/RESTful-Web-API-Design-with-Node.js-10-Third-Edition`](https://github.com/PacktPublishing/RESTful-Web-API-Design-with-Node.js-10-Third-Edition)**。如果代码有更新，将在现有的 GitHub 存储库上更新。

我们还有其他代码包，可以在我们丰富的书籍和视频目录中找到**[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)**。去看看吧！

# 使用的约定

在这本书中，您会发现一些不同类型信息的文本样式。以下是一些这些样式的例子，以及它们的含义解释。

文本中的代码字，数据库表名，文件夹名，文件名，文件扩展名，路径名，虚拟 URL，用户输入和 Twitter 句柄显示如下：

“这告诉`npm`我们的包依赖于 URL 和 express 模块。”

代码块设置如下：

```js
router.get('/v1/item/:itemId', function(request, response, next) {
  console.log(request.url + ' : querying for ' + request.params.itemId);
  catalogV1.findItemById(request.params.itemId, response);
});

router.get('/v1/:categoryId', function(request, response, next) {
  console.log(request.url + ' : querying for ' + request.params.categoryId);
  catalogV1.findItemsByCategory(request.params.categoryId, response);
});
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目将以粗体显示：

```js
router.get('/v1/:categoryId', function(request, response, next) {
  console.log(request.url + ' : querying for ' + request.params.categoryId);
  catalogV1.findItemsByCategory(request.params.categoryId, response);
});
```

任何命令行输入或输出都以以下形式书写：

```js
$ npm install -g express
```

**粗体**：表示一个新术语，一个重要的词或您在屏幕上看到的词。例如，菜单或对话框中的单词会在文本中以这种形式出现。这是一个例子：

警告或重要说明会出现在这样的形式中。

提示和技巧会以这种形式出现。
