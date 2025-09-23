# 前言

目前，对于 Google Maps JavaScript API，既有开源也有专有替代方案，但使 API 对开发者特别特殊的是，它是一个包含基本地图、覆盖层和技术能力的完整解决方案。

对于开发者来说，API 特别令人兴奋，因为它非常容易构建通用结果，同时，它在其同一个盒子中拥有自己的技巧和窍门以及高级功能。因此，当你使用 API 时，你可以浮在水面或深入水底。

Google Maps JavaScript API v3 使移动场景的快速和简单开发成为可能，便于基于位置解决方案的开发者深入研究该主题。关于移动开发的增长，尤其是基于位置的应用程序，Google Maps JavaScript API v3 值得应有的关注。

最后但同样重要的是，没有哪个地图 API 比谷歌地图 API 在没有持续更新和彻底处理矢量数据和卫星数据支持的情况下取得如此成功。谷歌投入了巨大的资源来维护矢量数据的统一结构和其制图质量，这些努力在 API 使用方面得到了回报。

# 本书涵盖的内容

第一章, *Google Maps JavaScript API 基础*，指导你如何围绕一个主要食谱创建一个简单的 Google Maps 应用程序。通过添加细节到食谱中，将介绍地图对象及其主要选项，包括地图类型。

第二章, *添加栅格图层*，通过一系列食谱展示了如何通过添加像瓦片、交通、公交和天气图层这样的 Google 图层来添加外部栅格数据。

第三章, *添加矢量图层*，介绍了如何绘制矢量特征，以及如何显示外部矢量数据源，如 KML 和 GeoRSS。

第四章, *使用控件*，详细解释了控件。本章将介绍为 Web 和移动设备创建和自定义自定义用户界面的方法。

第五章, *理解 Google Maps JavaScript API 事件*，详细描述了如何响应地图、图层或标记的行为，例如缩放结束、图层更改或标记添加。事件将为地图编程增加更多交互性。

第六章, *Google Maps JavaScript 库*，详细解释了将扩展 Google Maps JavaScript API 功能的库。这些库具有不同的能力，可以增强 Google Maps JavaScript API 的功能。

第七章，*使用服务*，详细介绍了将扩展 Google Maps JavaScript API 的服务。这些服务包括地理编码和街景，展示了 Google Maps JavaScript API 在地图方面的真正实力。

第八章，*通过高级食谱精通 Google Maps JavaScript API*，解释了外部 GIS 服务器和服务与 Google Maps JavaScript API 的集成。这些包括 ArcGIS Server、GeoServer、CartoDB 和 Google Fusion Tables，以及 OGC 服务如 WMS。

# 你需要这本书什么

Google Maps JavaScript API 与 HTML、CSS 和 JavaScript 代码一起工作。因此，一个具有 HTML、JavaScript 和 CSS 处理能力的文本编辑器在探索这本书时会成为你的好朋友。

对于 Mac 用户来说，有很多商业或免费的文本编辑器，例如 TextWrangler、BBEdit、Sublime Text 或 WebStorm。它们都能很好地处理 HTML、JavaScript 和 CSS。

对于 Windows 用户来说，也有不同的文本编辑器，但 Notepad++ 是最常用和推荐的。

选择编辑器取决于你的电脑习惯，因此没有确切的方法或推荐供用户选择一个编辑器。每个人都有不同的看法，这些选择会受到这些看法的影响。

实现这些食谱还需要一个 HTTP 服务器。有很多 HTTP 服务器，包括 Apache、IIS 等。但是，独立服务器的安装过程可能对大多数用户来说是个问题。我们鼓励您使用将 HTTP 服务器、数据库服务器和脚本语言捆绑在一起的一体化解决方案。XAMPP 和 MAMP 分别是 Windows 和 Mac OS X 平台上的这类解决方案。

为了提供更好的用户体验，我们创建了一个主应用程序，允许运行所需的食谱并显示其源代码。假设你已经安装并配置了一个本地 Web 服务器，如 XAMPP 或 MAMP，并且捆绑代码已复制到 `googlemaps-cookbook` 文件夹中的 HTTP 服务器根内容文件夹内，用户可以通过在浏览器中访问 `http://localhost/googlemaps-cookbook/index.html` URL 来运行主应用程序。

# 这本书面向的对象

这本书非常适合对在他们的网站上嵌入简单联系地图感兴趣的开发者，以及那些希望开发现实世界复杂 GIS 应用程序的人。它也适合那些想从他们的地理相关数据创建基于地图的信息图形，如热图的人。假设你已经具备一些 HTML、CSS 和 JavaScript 的经验，以及与 GIS 相关的简单概念的经验，并且对一些 GIS 服务器或服务有所了解。

# 习惯用法

在这本书中，你会找到许多不同风格的文本，以区分不同类型的信息。以下是一些这些风格的示例，以及它们含义的解释。

文本中的代码词如下所示："我们可以通过使用`include`指令来包含其他上下文。"

代码块设置如下：

```js
<!DOCTYPE html>
<html>
    <head>
        <!-- Include Google Maps JS API -->
        <script type="text/javascript"
        src="img/js?key=INSERT_YOUR_MAP_API_KEY_HERE&sensor=false"></script
```

**新术语**和**重要词汇**以粗体显示。

### 注意

警告或重要注意事项以如下框中显示。

### 小贴士

小贴士和技巧看起来是这样的。

# 读者反馈

我们始终欢迎读者的反馈。请告诉我们您对这本书的看法——您喜欢什么或可能不喜欢什么。读者的反馈对我们开发您真正能从中受益的标题非常重要。

要发送一般反馈，只需发送一封电子邮件到 `<feedback@packtpub.com>`，并在邮件主题中提及书名。

如果您在某个主题上具有专业知识，并且您对撰写或为书籍做出贡献感兴趣，请参阅我们在 [www.packtpub.com/authors](http://www.packtpub.com/authors) 的作者指南。

# 客户支持

现在，您已经成为 Packt 书籍的骄傲拥有者，我们有一些事情可以帮助您最大限度地利用您的购买。

## 下载示例代码

您可以从您在 [`www.packtpub.com`](http://www.packtpub.com) 的账户下载您购买的所有 Packt 书籍的示例代码文件。如果您在其他地方购买了这本书，您可以访问 [`www.packtpub.com/support`](http://www.packtpub.com/support) 并注册，以便将文件直接通过电子邮件发送给您。

## 错误清单

尽管我们已经尽一切努力确保我们内容的准确性，但错误仍然会发生。如果您在我们的书中发现错误——可能是文本或代码中的错误——如果您能向我们报告这一点，我们将不胜感激。通过这样做，您可以节省其他读者的挫败感，并帮助我们改进本书的后续版本。如果您发现任何错误清单，请通过访问 [`www.packtpub.com/submit-errata`](http://www.packtpub.com/submit-errata)，选择您的书，点击**错误清单提交表**链接，并输入您的错误清单详情。一旦您的错误清单得到验证，您的提交将被接受，错误清单将被上传到我们的网站，或添加到该标题的错误清单部分。任何现有的错误清单都可以通过从 [`www.packtpub.com/support`](http://www.packtpub.com/support) 选择您的标题来查看。

## 盗版

互联网上版权材料的盗版是一个跨所有媒体持续存在的问题。在 Packt，我们非常重视我们版权和许可证的保护。如果您在互联网上发现我们作品的任何非法副本，无论形式如何，请立即提供位置地址或网站名称，以便我们可以追究补救措施。

请通过以下链接联系我们 `<copyright@packtpub.com>`，并提供涉嫌盗版材料的链接。

我们感谢您在保护我们作者和提供有价值内容方面的帮助。

## 问题

如果你在本书的任何方面遇到问题，可以通过 `<questions@packtpub.com>` 联系我们，我们将尽力解决。
