# 前言

ArcGIS Server 是用于开发 Web 的 GIS 应用程序的主要平台。您可以使用多种编程语言来开发 ArcGIS Server 应用程序，包括 JavaScript，Flex 和 Silverlight。JavaScript 已成为在此平台上开发应用程序的首选语言，因为它既可用于 Web 应用程序，也可用于移动应用程序，并且不需要在浏览器中安装插件。Flex 和 Silverlight 在移动开发方面都表现不佳，并且都需要在浏览器中运行应用程序时使用插件。

本书将教会您如何使用 ArcGIS API for JavaScript 构建基于 Web 的 GIS 应用程序。通过实际的，动手学习方式，您将学习如何使用 ArcGIS Server 开发完全功能的应用程序，并开发一套高需求的技能。

您将学习如何从各种来源创建地图并添加地理图层，包括瓦片和动态地图服务。此外，您还将学习如何向地图添加图形，并使用`FeatureLayer`将地理要素流式传输到浏览器。大多数应用程序还包括由 ArcGIS Server 实现的特定功能。您将学习如何使用 ArcGIS Server 提供的各种任务，包括查询，通过属性查找要素，地理处理任务等。最后，您将了解使用 ArcGIS API for JavaScript 开发移动应用程序有多么容易。

# 本书涵盖内容

第一章，HTML，CSS 和 JavaScript 简介，介绍了在使用 ArcGIS API for JavaScript 开发 GIS 应用程序之前的基本 HTML，CSS 和 JavaScript 概念。

第二章，创建地图和添加图层，教会你如何创建地图并向地图添加图层。您将学习如何创建`Map`类的实例，向地图添加数据图层，并在网页上显示这些信息。`Map`类是 API 中最基本的类，因为它为数据图层提供了画布，以及应用程序中发生的任何后续活动。但是，在添加数据图层之前，您的地图是无用的。可以向地图添加几种类型的数据图层，包括瓦片，动态和要素。读者将在本章中了解更多关于这些图层类型的信息。

第三章，向地图添加图形，教会读者如何在地图上显示临时点，线和多边形，并在`GraphicsLayer`中存储。`GraphicsLayer`是一个单独的图层，始终位于其他图层的顶部，并存储与地图相关的所有图形。

第四章，要素图层，除了继承自`GraphicsLayer`的其他功能，还具有执行查询和选择的能力。要素图层还用于在线编辑要素。要素图层与瓦片和动态地图服务图层不同，因为要素图层将几何信息带到客户端计算机，由 Web 浏览器绘制和存储。要素图层可能减少了对服务器的往返。客户端可以请求所需的要素，并对这些要素执行选择和查询，而无需从服务器请求更多信息。

第五章，*使用小部件和工具栏*，介绍了可以将其放入应用程序以提高生产力的开箱即用小部件。包括 BasemapGallery、Bookmarks、Print、Geocoding、Legend、Measurement、Scalebar、Gauge 和 Overview map 小部件。此外，ArcGIS API for JavaScript 还包括用于向应用程序添加各种工具栏的辅助类，包括导航和绘图工具栏。

第六章，*执行空间和属性查询*，介绍了 ArcGIS Server 查询任务，允许您对已公开的地图服务中的数据图层执行属性和空间查询。您还可以结合这些查询类型执行组合属性和空间查询。

第七章，*识别和查找要素*，介绍了在任何 GIS 应用程序中都可以找到的两个常见操作。这些操作要求用户在识别的情况下在地图上单击要素，或者在查找要素的情况下执行查询。在任何情况下，都会返回有关特定要素的信息。在本章中，读者将学习如何使用`IdentifyTask`和`FindTask`对象获取有关要素的信息。

第八章，*将地址转换为点，将点转换为地址*，介绍了使用 Locator 任务执行地理编码和反向地理编码。地理编码是将坐标分配给地址的过程，而反向地理编码是将地址分配给坐标的过程。

第九章，*网络分析任务*，允许您对街道网络执行分析，例如从一个地址到另一个地址找到最佳路线，找到最近的学校，识别位置周围的服务区域，或者用一组服务车辆响应一组订单。

第十章，*地理处理任务*，允许您执行在 ArcGIS Desktop 中使用 ModelBuilder 构建的自定义模型。模型可以在桌面环境或通过通过 Web 应用程序访问的集中服务器中以自动方式运行。ArcToolbox 中的任何工具，无论是您的 ArcGIS 许可级别的工具还是您构建的自定义工具，都可以在模型中使用，并与其他工具链接在一起。构建完成后，这些模型可以在集中服务器上运行，并通过 Web 应用程序访问。在本章中，我们将探讨如何通过 ArcGIS API for JavaScript 访问这些地理处理任务。

第十一章，*与 ArcGIS Online 集成*，详细介绍了如何使用 ArcGIS API for JavaScript 访问使用[ArcGIS.com](http://ArcGIS.com)创建的数据和地图。网站[ArcGIS.com](http://ArcGIS.com)用于处理地图和其他类型的地理信息。在这个网站上，您将找到用于构建和共享地图的应用程序。您还将找到有用的底图、数据、应用程序和工具，可以查看和使用，以及可以加入的社区。对于应用程序开发人员来说，真正令人兴奋的消息是您可以使用 ArcGIS API for JavaScript 将[ArcGIS.com](http://ArcGIS.com)内容集成到您的自定义开发的应用程序中。在本章中，您将探索如何将[ArcGIS.com](http://ArcGIS.com)地图添加到您的应用程序中。

第十二章，*创建移动应用程序*，详细介绍了如何使用 ArcGIS API for JavaScript 构建移动 GIS 应用程序。ArcGIS Server 目前支持 iOS、Android 和 BlackBerry 操作系统。该 API 与 dojox/mobile 集成。在本章中，您将了解到 API 的紧凑构建，使得通过 web-kit 浏览器以及内置手势支持成为可能。

附录，*使用 ArcGIS 模板和 Dojo 设计应用程序*，涵盖了许多 Web 开发人员最困难的任务之一，即设计和创建用户界面。ArcGIS API for JavaScript 和 Dojo 极大地简化了这项任务。Dojo 的布局 dijits 提供了一种简单、高效的方式来创建应用程序布局，Esri 提供了许多示例应用程序布局和模板，您可以使用它们快速启动。在本附录中，读者将学习快速设计应用程序的技巧。

# 您需要为这本书做好准备

要完成本书中的练习，您需要访问一个 Web 浏览器，最好是 Google Chrome 或 Firefox。每一章都包含了旨在补充所呈现材料的练习。练习将使用 ArcGIS API for JavaScript Sandbox 来编写和测试您的代码。Sandbox 可以在[`developers.arcgis.com/en/javascript/sandbox/sandbox.html`](http://developers.arcgis.com/en/javascript/sandbox/sandbox.html)找到。练习将访问 ArcGIS Server 的公开实例，因此您不需要安装 ArcGIS Server。

# 这本书是为谁准备的

如果您是一名应用程序开发人员，希望使用 ArcGIS Server 和 JavaScript API 开发 Web 和移动 GIS 应用程序，那么这本书非常适合您。它主要面向初学者和中级 GIS 开发人员或应用程序开发人员，他们可能更传统，以前可能没有开发过 GIS 应用程序，但现在被要求在这个平台上实施解决方案。不需要具有 ArcGIS Server、JavaScript、HTML 或 CSS 的先前经验，但这当然是有帮助的。

# 约定

在本书中，您将找到一些区分不同信息类型的文本样式。以下是这些样式的一些示例，以及它们的含义解释。

文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄显示如下："将`onorientationchange()`事件添加到`<body>`标签。"

代码块设置如下：

```js
routeParams = new RouteParameters();
routeParams.stops = new FeatureSet();
routeParams.outSpatialReference = {wkid:4326};
routeParams.stops.features.push(stop1);
routeParams.stops.features.push(stop2);
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目将以粗体显示：

```js
**function computeServiceArea(evt) {**
 **map.graphics.clear();**
 **var pointSymbol = new SimpleMarkerSymbol();**
 **pointSymbol.setOutline = new SimpleLineSymbol(SimpleLineSymbol.STYLE_SOLID, new Color([255, 0, 0]), 1);**
 **pointSymbol.setSize(14);**
 **pointSymbol.setColor(new Color([0, 255, 0, 0.25]));** 
}
```

**新术语**和**重要单词**以粗体显示。您在屏幕上看到的单词，比如菜单或对话框中的单词，会以这样的形式出现在文本中："单击**运行**按钮"。

### 注意

警告或重要说明会出现在这样的框中。

### 提示

提示和技巧会以这样的形式出现。
