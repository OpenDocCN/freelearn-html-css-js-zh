# 前言

制作地图曾经需要广泛的制图知识、昂贵的软件和专业技术。如今，有众多工具可供选择，其中许多是免费的，简化了地图制作过程。本书是关于使用这样一个库，即 Leaflet.js。

Leaflet.js 是一个 JavaScript 库，虽然体积小，但几乎包含了你需要的所有功能。如果一个功能在核心库中不可用，它可能作为众多插件之一提供。最大的地图制作软件供应商，环境系统研究学院（ESRI），甚至已经发布了 Leaflet.js 的插件。如果你对制作地图或数据可视化感兴趣，Leaflet.js 是你要学习的库。

无论你是想构建简单的地图还是高级的地图应用，这本书都将基于你的 JavaScript 知识，帮助你实现目标。这本书旨在让那些对地图制作新手以及那些可能了解地图但刚开始学习编码的人都能轻松阅读。

# 本书涵盖内容

第一章，*使用 Leaflet 创建地图*，将引导你了解在 Leaflet.js 中制作地图的基础知识。你将从创建一个包含最少 JavaScript 代码的 HTML 文件开始，以显示地图。你将学习如何选择不同的底图和提供者以及不同的底图格式。然后，你将学习如何显示地理特征，如点、折线和多边形。

第二章，*地图 GeoJSON 数据*，介绍了 GeoJSON 数据格式的地理版本。你将学习如何创建自己的 GeoJSON 数据以及从其他来源获取数据。在本章中，你将学习如何对数据进行样式化，并通过特征迭代添加弹出窗口。

第三章，*创建热力图和等值线图*，从简单地显示点转向显示数据的显著性或比较。它建立在之前所学的基础上，并教你如何使用不同的插件创建热力图。你还将学习如何利用你对 GeoJSON 样式的知识来创建等值线图。

第四章，*创建自定义标记*，将指导你如何定制你在地图中使用的标记。你将学习如何绘制自己的图像或修改现有图像，以便将其用作地图中的标记。你将了解几个提供可定制预制标记的插件。此外，你还将学习如何使标记动画化以及如何结合插件以产生附加效果。

第五章，*Leaflet 中的 ESRI*，介绍了地图中最常用的数据格式和服务器端点。本章将教会您如何在地图中加载 shapefiles。您还将学习如何连接到一个提供公开 REST 服务的 ESRI 服务器。使用 ESRI-Leaflet 插件，您将学习如何进行地址的地理编码和反向地理编码，从服务器过滤数据，以及通过位置进行查询。

第六章，*Node.js、Python 和 C#中的 Leaflet*，在您所学的所有内容的基础上，教您如何构建其他框架和语言的应用程序。本章将教会您如何构建前端和后端。您将使用 JavaScript 和 Python 构建服务器。您将了解 NoSQL 数据库和 AJAX，以在不刷新网页的情况下显示和更新数据。最后，您将学习如何通过在 C#中嵌入 Leaflet 来创建 Windows 桌面应用程序。

# 您需要为本书准备的

至少需要以下软件来完成本书：

+   一个网络浏览器，最好是 Google Chrome，可在[`www.google.com/chrome/browser/`](https://www.google.com/chrome/browser/)找到。

对于第五章中的示例，*Leaflet 中的 ESRI*和第六章，*Node.js、Python 和 C#中的 Leaflet*，需要以下软件：

+   Node.js，可在[`nodejs.org/`](http://nodejs.org/)找到。

+   Python 2.7，最好是 3.x 版本。您可以从[`www.python.org/`](https://www.python.org/)下载任一版本。

+   Visual Studio Express 2010。您可以在[`www.visualstudio.com/downloads/download-visual-studio-vs`](http://www.visualstudio.com/downloads/download-visual-studio-vs)找到免费副本。

+   CherryPy 可在[`www.cherrypy.org/`](http://www.cherrypy.org/)找到。

+   MongoDB 可在[`www.mongodb.org/`](http://www.mongodb.org/)找到。

+   Pymongo 是用于 MongoDB 的 Python 库，可在[`pypi.python.org/pypi/pymongo/`](https://pypi.python.org/pypi/pymongo/)免费获取。

+   MongoDB 的 C#驱动程序可在[`github.com/mongodb/mongo-csharp-driver/releases`](https://github.com/mongodb/mongo-csharp-driver/releases)以多种格式提供。

+   WAMP 可以从[`www.wampserver.com/en/`](http://www.wampserver.com/en/)下载。

# 本书面向的对象

如果您是一位具有一些 JavaScript 知识的地图制作者，这本书是您理想的资源，它教会您如何将地图带到网络并使其交互式。如果您是一位 JavaScript 开发者，这本书将向您展示如何使用这些技能来构建强大的地图应用程序。

# 规范

在本书中，您会发现许多不同风格的文本，用于区分不同类型的信息。以下是一些这些风格的示例及其含义的解释。

文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名都按照以下方式显示：“创建一个查询的文本字符串，并初始化你的`StringBuilder()`方法以保存函数和结果中的 JavaScript。”

代码块按照以下方式设置：

```js
var layer = new L.TileLayer('http://{s}.tile.thunderforest.com/landscape/{z}/{x}/{y}.png');
map.addLayer(layer);
```

当我们希望引起你对代码块中特定部分的注意时，相关的行或项目将以粗体显示：

```js
<style>
  html, body, #map {
      padding: 0;
      margin: 0;
      height: 100%;
    }
#points.hidden {
 display: none;
}
</style>
<body>
<div id="map"></div>
<div id="points"></div>

```

任何命令行输入或输出都按照以下方式书写：

```js
npm install –g connect

```

**新术语**和**重要词汇**以粗体显示。你在屏幕上看到的，例如在菜单或对话框中的文字，在文本中会这样显示：“你现在需要在**解决方案资源管理器**窗口中右键单击项目，并选择**添加引用**。”

### 注意

警告或重要提示以如下方式显示在框中。

### 小贴士

小贴士和技巧看起来像这样。

# 读者反馈

我们始终欢迎读者的反馈。告诉我们你对这本书的看法——你喜欢什么或可能不喜欢什么。读者反馈对我们开发你真正能从中获得最大价值的标题非常重要。

要发送给我们一般性的反馈，只需发送电子邮件到`<feedback@packtpub.com>`，并在你的邮件主题中提及书名。

如果你在一个你具有专业知识的话题上感兴趣，无论是写作还是为书籍做出贡献，请参阅我们的作者指南[www.packtpub.com/authors](http://www.packtpub.com/authors)。

# 客户支持

现在你已经是 Packt 书籍的骄傲拥有者，我们有一些事情可以帮助你从你的购买中获得最大价值。

## 下载示例代码

你可以从你购买的所有 Packt 书籍的账户中下载示例代码文件。[`www.packtpub.com`](http://www.packtpub.com)。如果你在其他地方购买了这本书，你可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)，并注册以将文件直接通过电子邮件发送给你。

## 错误清单

尽管我们已经尽一切努力确保内容的准确性，但错误仍然可能发生。如果你在我们的书中发现错误——可能是文本或代码中的错误——如果你能向我们报告这一点，我们将不胜感激。通过这样做，你可以帮助其他读者避免挫败感，并帮助我们改进本书的后续版本。如果你发现任何错误清单，请通过访问[`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择你的书籍，点击**错误清单提交表单**链接，并输入你的错误清单详情。一旦你的错误清单得到验证，你的提交将被接受，错误清单将被上传到我们的网站，或添加到该标题的错误清单部分。任何现有的错误清单都可以通过从[`www.packtpub.com/support`](http://www.packtpub.com/support)中选择你的标题来查看。

## 侵权

在互联网上，版权材料的盗版是一个跨所有媒体的持续问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现我们作品的任何非法副本，无论形式如何，请立即提供位置地址或网站名称，以便我们可以追究补救措施。

请通过 `<copyright@packtpub.com>` 联系我们，并提供涉嫌盗版材料的链接。

我们感谢您在保护我们作者方面的帮助，以及我们为您提供有价值内容的能力。

## 问题和建议

如果您在本书的任何方面遇到问题，可以通过 `<questions@packtpub.com>` 联系我们，我们将尽力解决。
