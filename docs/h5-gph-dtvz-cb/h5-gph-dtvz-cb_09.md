# 第九章：使用 Google 地图

在本章中，我们将涵盖：

+   使用 Google Visualization API 创建地理图表

+   获取 Google API 密钥

+   构建 Google 地图

+   添加标记和事件

+   自定义控件和重叠地图

+   使用样式重新设计地图

# 介绍

本章将致力于探索 Google 地图上的一些功能，以便让我们准备好处理地图工作。单独的地图并不是数据可视化，但是在我们通过了解如何处理地图来建立基础之后，我们将能够通过整合数据和数据可视化来创建许多尖端、酷炫的项目。

在本章中，我们将探索在 Google 领域创建地图的主要方法。

# 使用 Google Visualization API 创建地理图表

在本章的第一个配方中，我们将开始使用基于矢量的世界地图。我们将用它来根据数据源突出显示国家。在我们的情况下，我们将使用维基百科的国家列表，根据故意谋杀率（最新数据）。

要查看原始数据，请访问[`en.wikipedia.org/wiki/List_of_countries_by_intentional_homicide_rate`](http://en.wikipedia.org/wiki/List_of_countries_by_intentional_homicide_rate)。

我们的目标是拥有一张世界地图，根据每 10 万人中故意谋杀的数量而突出显示一系列颜色。根据维基百科 2012 年的最新数据，它听起来像是最不安全的地方是洪都拉斯——如果你不想被故意杀害的话——而在日本你应该感到非常安全。你的国家怎么样？我的国家还不错。我可能应该避开让我感觉自己生活在战区的当地新闻台。

![使用 Google Visualization API 创建地理图表](img/3707OT_09_01.jpg)

## 准备工作

不需要做太多事情。我们将使用 Google Visualization API 来创建地理图表。

## 如何做...

我们将创建一个新的 HTML 和一个新的 JavaScript 文件，并将它们命名为`08.01.geo-chart.html`和`08.01.geo-chart.js`。按照以下步骤进行：

1.  在 HTML 文件中添加以下代码：

```js
<!DOCTYPE html>
<html>
  <head>
    <title>Geo Charts</title>
    <meta charset="utf-8" />   
    <script src="img/jsapi"></script>
    <script src="img/08.01.geo-chart.js"></script>
  </head>
  <body style="background:#fafafa">
    <div id="chart"></div>
  </body>
</html>
```

1.  让我们转到`js`文件。这一次，我们将要求从 Google Visualization 包中请求`geochart`功能。为此，我们将从以下代码开始：

```js
google.load('visualization','1',{'packages': ['geochart']});
```

1.  然后我们将添加一个回调，当包准备就绪时将触发`init`函数：

```js
google.setOnLoadCallback(init);
function init(){
 //...
}
```

1.  现在是时候在`init`函数中添加逻辑了。在第一步中，我们将从维基百科格式化数据为另一种格式，以便适用于 Google Visualization API：

```js
var data = google.visualization.arrayToDataTable([
    ['Country','Intentional Homicide Rate per 100,000'],
    ['Honduras',87],['El Salvador',71],['Saint Kitts and Nevis',68],
    ['Venezuela',67],['Belize',39],['Guatemala',39],['Jamaica',39],
    ['Bahamas',36],['Colombia',33],['South Africa', 32],
    ['Dominican Republic',31],['Trinidad and Tobago',28],['Brazil',26],
    ['Dominica', 22],['Saint Lucia',22],['Saint Vincent and the Grenadines',22],
    ['Panama',20],['Guyana',18],['Mexico',18],['Ecuador',16],
    ['Nicaragua',13],['Grenada',12],['Paraguay',12],['Russia',12],
    ['Barbados',11],['Costa Rica',10 ],['Bolivia',8.9],
    ['Estonia',7.5],['Moldova',7.4],['Haiti',6.9],
    ['Antigua and Barbuda',6.8],['Uruguay',6.1],['Thailand',5.3],
    ['Ukraine',5.2],['United States',4.7 ],['Georgia',4.1],['Latvia',4.1 ],
    ['India',3.2],['Taiwan',3.0 ],['Bangladesh',2.4 ],['Lebanon',2.2],
    ['Finland',2.1 ],['Israel', 2.1],['Macedonia',1.94 ],['Canada',1.7],
    ['Czech Republic',1.67],['New Zealand',1.41],['Morocco',1.40 ],
    ['Chile',1.33],['United Kingdom',1.23 ],['Australia',1.16],
    ['Poland',1.1 ],['Ireland',0.96 ],['Italy',.87 ],['Netherlands',.86 ],
    ['Sweden',.86],['Denmark',.85],['Germany',.81 ],['Spain',0.72],
    ['Norway',0.68],['Austria',0.56],['Japan',.35] 
]);
```

1.  让我们配置我们的图表选项：

```js
var options = {width:800,height:600};
```

1.  最后但绝不是最不重要的，让我们创建我们的图表：

```js
 var chart = new google.visualization.GeoChart(document.getElementById('chart'));
  chart.draw(data,options);
}//end of init function
```

当您加载 HTML 文件时，您会发现世界各国以反映谋杀率的突出颜色显示出来。（我们没有所有世界国家的完整列表，有些国家太小，很难找到它们。）

## 它是如何工作的...

这个配方的逻辑非常简单，所以让我们快速浏览一下，并添加一些额外的功能。与所有其他可视化图表一样，有三个单独的步骤：

+   定义数据源

+   设置图表

+   绘制图表

并非所有国家都是相同的。如果您在处理一个有轮廓的国家时遇到问题，请搜索最新的 Google 文档，了解支持的国家。您可以在[`gmaps-samples.googlecode.com/svn/trunk/mapcoverage_filtered.html`](http://gmaps-samples.googlecode.com/svn/trunk/mapcoverage_filtered.html)上查看完整列表。

## 还有更多...

让我们对我们的图表添加一些额外的自定义。与所有 Google Visualization 库元素一样，我们可以通过`options`对象控制许多可视化效果。

我们地图中突出显示的绿色看起来不对。你会认为杀戮越少，一个国家就会越绿，所以在杀戮更多的地方，更深的红色更合适。所以让我们通过更新`options`对象来改变颜色：

```js
  var options = {width:800,height:600,
    colorAxis: {colors: ['#eeffee', 'red']}
      };
```

![还有更多...](img/3707OT_09_02.jpg)

### 使较小的区域更可见

为了解决真正小的不可见的国家的问题，我们可以将我们的渲染切换为基于标记的。我们可以切换到基于标记的渲染模式，而不是突出显示土地本身：

```js
var options = {width:800,height:600,
    displayMode: 'markers',
        colorAxis: {colors: ['#22ff22', 'red']}
      };
```

默认情况下，当使用标记渲染可视化地图时，当您在压缩区域上滚动时，高亮的缩放视图将帮助创建更清晰的视图：

![使较小的区域更可见](img/3707OT_09_03.jpg)

另一个选择是放大到该区域（我们可以两者都做，或者只是放大）。要放大到一个区域，我们将使用这段代码：

```js
  var options = {width:800,height:600,
    region:'MX',
        colorAxis: {colors: ['#22ff22', 'red']}
      };
```

要了解可能的值列表，请参阅本章前面的国家列表。在这种情况下，我们正在放大到`MX`地区：

![使较小的区域更可见](img/3707OT_09_04.jpg)

这涵盖了使用地理图表的基础知识。有关使用 Google Visualization API 的更多信息，请参阅第八章*玩转 Google 图表*。

# 获取 Google API 密钥

要使用大多数 Google API，你必须有一个 Google API 密钥。因此，我们将介绍获取 Google API 密钥所涉及的步骤。

Google API 有一些限制和约束。尽管大多数 API 对于中小型网站是免费的，但你仍然受到一些规则的约束。请参考每个库的规则和条例。

## 准备工作

要完成这个示例，你必须有一个 Google ID；如果你没有，你需要创建一个。

## 如何做...

让我们列出获得访问 Google API 所需步骤：

1.  登录到[`code.google.com/apis/console`](https://code.google.com/apis/console)的 API 控制台。

1.  从左侧菜单中选择**服务**选项：![操作步骤...](img/3707OT_09_05.jpg)

1.  激活你想要使用的 API（例如，在下一个示例*构建 Google 地图*中，我们将使用 Google Maps API v3 服务）：![操作步骤...](img/3707OT_09_06.jpg)

1.  同样，在左侧菜单中选择**API 访问**选项。您将需要复制**API 密钥**并在将来的 Google API 项目中替换它：![操作步骤...](img/3707OT_09_07.jpg)

这是我们唯一一次讨论与 Google API 平台的密钥和权限有关的问题。请验证您已激活密钥，并设置正确的库以便您可以访问。

## 它是如何工作的...

理解这是如何工作的并不难。你只需要记住这些步骤，因为它们将成为我们创建未来 Google API 交互的基础。

正如你可能已经注意到的，Google 库中有许多 API，我们甚至无法全部涉及，但我建议你浏览一下并探索你的选择。在接下来的几个示例中，我们将使用 Google API 来执行一些与地图相关的任务。

# 构建 Google 地图

数据和地理有着非常自然的关系。数据在地图上更有意义。使用实时地图是一个非常好的选择，因为它可以让用户与地理区域内集成了您自己数据呈现的 UI 进行交互。在这个示例中，我们将集成我们的第一个真实实时地图。

## 准备工作

要完成这个示例，你必须有一个 Google ID。如果你没有，你需要创建一个。除此之外，你还需要在 API 控制台中激活 Google Maps API v3 服务。有关更多信息，请参阅本章前面讨论的*获取 Google API 密钥*示例。

我们的目标是创建一个全屏的 Google 地图，将放大并聚焦在法国：

![准备工作](img/3707OT_09_08.jpg)

## 如何做...

让我们列出创建此示例的步骤。要创建此示例，我们将创建两个文件——一个`.html`文件和一个`.js`文件：

1.  让我们从 HTML 文件开始。我们将为我们的项目创建一个基本的 HTML 文件基线：

```js
<!DOCTYPE html>
<html>
  <head>
    <title>Google Maps Hello world</title>
    <meta charset="utf-8" />
  </head>
  <body>
    <div id="jsmap"></div>
  </body>
</html>
```

1.  我们将添加 HTML 视口信息。这是移动设备如何呈现页面的指示（如果您不关心在移动设备上访问地图，可以跳过此步骤）：

```js
<head>
  <title>Google Maps Hello world</title>
  <meta charset="utf-8" />
 <meta name="viewport" content="initial-scale=1.0, user-scalable=no" /> 
</head>
```

1.  将样式信息添加到头部：

```js
<style>
  html { height: 100% }
  body { height: 100%; margin: 0; padding: 0 }
  #jsmap { height: 100%; width:100% }
</style>
```

1.  加载 Google Maps v3 API（用您的 API 密钥替换粗体文本）：

```js
<script src="img/strong>&sensor=true">
```

1.  添加我们的`09.03.googleJSmaps.js` JavaScript 文件的脚本源：

```js
<script src="img/09.03.googleJSmaps.js"></script>
```

1.  添加一个`onload`触发器，将调用`init`函数（这将在下一步中创建）：

```js
<body onload="init();">
```

1.  在`09.03.googleJSmaps.js` JavaScript 文件中，添加`init`函数：

```js
function init() {
  var mapOptions = {
    center: new google.maps.LatLng(45.52, 0),
    zoom: 7,
    mapTypeId: google.maps.MapTypeId.ROADMAP
  };
  var map = new google.maps.Map(document.getElementById("jsmap"), mapOptions);
}
```

1.  加载 HTML 文件，您应该会发现一个全屏幕的路线图缩放到法国。

## 它是如何工作的...

最重要和第一步是加载`maps` API。为了让 Google 满足您的请求，您必须拥有有效的 API 密钥。因此，请不要忘记用您的密钥替换粗体文本：

```js
<script src="img/strong>&sensor=true">
```

不要忘记使用您自己的密钥。您可能会发现自己的网站地图出现故障。URL 中的`sensor`参数是强制性的，必须设置为`true`或`false`。如果您的地图需要知道用户位置在哪里，您必须将其设置为`true`，如果不需要，可以将其设置为`false`。

在我们的应用程序中另一个有趣的事情是，这是我们第一次在示例中使用视口。由于这个主题超出了本书的范围，我想留下来。我知道你们中的许多人最终会在移动设备上使用地图，并希望地图默认为垂直/水平视图。要了解更多有关视口如何工作的信息，请查看此处提供的文章：[`developer.mozilla.org/en/Mobile/Viewport_meta_tag/`](https://developer.mozilla.org/en/Mobile/Viewport_meta_tag/)。

您可能已经注意到，我们在我们的 CSS 中设置了许多东西为 100%，正如您可能猜到的那样，这是为了向后兼容性和验证地图将填满整个屏幕。如果您只想创建一个固定的宽度/高度，您可以通过用以下代码替换 CSS 来实现：

```js
<style>
    #jsmap { height: 200px; width:300px; }
</style>
```

这涵盖了我们在 HTML 文件中需要做的主要事情。

## 还有更多...

我们还没有涵盖`init`函数如何工作的细节。`init`函数的基本原理非常简单。创建地图只涉及两个步骤。我们需要知道我们希望地图位于哪个`div`层，并且我们希望将哪些选项发送到我们的地图：

```js
var map = new google.maps.Map(div,options);
```

与上一个配方中的 Google 可视化 API 有三个步骤不同，我们可以看到 Google `maps` API 只有一个步骤，在其中我们直接发送两个选项以进行渲染（在创建和渲染之间没有步骤）。

让我们更深入地了解选项，因为它们将改变地图的大部分视觉和功能。

### 使用纬度和经度

**纬度和经度**（**lat/long**）是一种将地球划分为网格模式的坐标系统，使得在地球上定位点变得容易。纬度代表垂直空间，而经度代表水平空间。需要注意的是，谷歌使用世界大地测量系统 WGS84 标准。还有其他标准存在，所以如果你的纬度/经度不使用相同的标准，你会发现自己位于一个与最初寻找的位置不同的地方。

基于纬度/经度定位区域的最简单方法是通过我们地图上的辅助工具或搜索主要城市的纬度/经度信息。

[`www.gorissen.info/Pierre/maps/googleMapLocation.php`](http://www.gorissen.info/Pierre/maps/googleMapLocation.php)将帮助您直接在谷歌地图上点击以定位一个点。在此类别中的另一个选项是在主谷歌地图站点（[`maps.google.com/`](http://maps.google.com/)）上打开实验室功能。在屏幕左下角的主谷歌地图站点上，您会找到**地图实验室**。在那里，您会找到一些纬度/经度助手。

或者您可以通过访问[`www.realestate3d.com/gps/latlong.htm`](http://www.realestate3d.com/gps/latlong.htm)按城市搜索数据。

在我们的情况下，当我们准备好做出选择时，我们将更新`options center`属性，以反映我们希望地图居中的位置，并调整缩放级别，直到感觉合适：

```js
var mapOptions = {
    center: new google.maps.LatLng(45.52, 0),
    zoom: 7,
    mapTypeId: google.maps.MapTypeId.ROADMAP
};
```

### 地图类型

有许多地图类型，甚至可以创建自定义的地图类型，但是对于我们的需求，我们将专注于最常用的基本类型：

+   `google.maps.MapTypeId.ROADMAP`：显示谷歌地图的正常、默认的 2D 瓦片

+   `google.maps.MapTypeId.SATELLITE`：显示摄影瓦片

+   `google.maps.MapTypeId.HYBRID`：显示摄影瓦片和突出特征的瓦片图层（道路、城市名称等）

+   `google.maps.MapTypeId.TERRAIN`：显示用于显示海拔和水体特征（山脉、河流等）的物理地形瓦片

这涵盖了您需要了解的基础知识，以便将地图集成到网站上。

# 添加标记和事件

我们屏幕上有地图很棒（假设您已经按照上一篇文章*构建谷歌地图*），但是如何连接数据并将其集成到我们的地图中呢。我很高兴你问到了这个问题，因为这篇文章将是我们第一步，将数据添加到标记和事件的形式中。

在这个示例中，我们的目标是在纽约市放置四个标记。当点击标记时，我们将放大到该区域并切换地图视图类型。

![添加标记和事件](img/3707OT_09_09.jpg)

## 准备工作

在这个阶段，您应该已经使用 JS API 创建了（至少一次）谷歌地图；如果没有，请回到*构建谷歌地图*的步骤。

## 如何做到这一点...

我们不会在上一篇文章*构建谷歌地图*中创建的 HTML 页面中进行进一步的更改；因此，我们将把注意力集中在 JavaScript 文件上：

1.  创建一个`init`函数：

```js
function init(){
//all the rest of logic in here
}
```

1.  在`base`状态中创建地图常量，然后放大到该状态：

```js
function init() {
  var BASE_CENTER = new google.maps.LatLng(40.7142,-74.0064 );
  var BASE_ZOOM = 11;
  var BASE_MAP_TYPE = google.maps.MapTypeId.SATELLITE;
  var INNER_ZOOM = 14;
  var INNER_MAP_TYPE = google.maps.MapTypeId.ROADMAP;
```

1.  创建默认地图选项：

```js
//40.7142° N, -74.0064 E NYC
var mapOptions = {
  center: BASE_CENTER,
  zoom: BASE_ZOOM,
  mapTypeId: BASE_MAP_TYPE
};
var map = new google.maps.Map(document.getElementById("jsmap"), mapOptions);
```

1.  为我们的点创建数据源：

```js
var aMarkers = [
  {label:'New York City',
  local: map.getCenter()},
  {label:'Brooklyn',
  local: new google.maps.LatLng(40.648, -73.957)},
  {label:'Queens',
  local: new google.maps.LatLng(40.732, -73.800)}, 
  {label:'Bronx',
  local: new google.maps.LatLng(40.851, -73.871)},  

];
```

1.  循环遍历每个数组元素，并创建一个带有事件的标记，该事件将放大到该位置，切换视图并平移到正确的位置：

```js
var marker;

for(var i=0; i<aMarkers.length; i++){
  marker = new google.maps.Marker({
    position: aMarkers[i].local,
    map: map,
    title: aMarkers[i].label
  });
  google.maps.event.addListener(marker, 'click', function(ev) {
    map.setZoom(INNER_ZOOM);
    map.panTo(ev.latLng);
    map.setMapTypeId(INNER_MAP_TYPE);
  });

}
```

1.  最后但并非最不重要的是，使地图可点击。因此，当用户点击地图时，它应该重置为其原始状态：

```js
google.maps.event.addListener(map, 'click', function() {
   	 map.setZoom(BASE_ZOOM);
    map.panTo(BASE_CENTER);
    map.setMapTypeId(BASE_MAP_TYPE);

});
```

当您运行应用程序时，您会在屏幕上找到四个标记。当您点击它们时，您将跳转到更深的缩放视图。当您点击空白区域时，它将带您回到原始视图。

## 工作原理...

与事件和谷歌地图一起工作非常容易。所涉及的步骤始终从调用静态方法`google.maps.event.addListener`开始。此函数接受三个参数，即要监听的项目、事件类型（作为字符串）和一个函数。

例如，在我们的`for`循环中，我们创建标记，然后为它们添加事件：

```js
 google.maps.event.addListener(marker, 'click', function(ev) {
    map.setZoom(INNER_ZOOM);
    map.panTo(ev.latLng);
    map.setMapTypeId(INNER_MAP_TYPE);
  });
```

相反，我们可以创建事件，然后不需要每次循环时重新创建一个新的匿名函数：

```js
for(var i=0; i<aMarkers.length; i++){
  marker = new google.maps.Marker({
    position: aMarkers[i].local,
    map: map,
    title: aMarkers[i].label
  });	

  google.maps.event.addListener(marker, 'click', onMarkerClicked);
  }

 function onMarkerClicked(ev){
 map.setZoom(INNER_ZOOM);
 map.panTo(ev.latLng);
 map.setMapTypeId(INNER_MAP_TYPE);
  }
```

优势真的很大。我们不是为每个循环创建一个函数，而是在整个过程中使用相同的函数（更智能，内存占用更小）。在我们的代码中，我们没有提及任何硬编码的值。相反，我们使用事件信息来获取`latLng`属性。我们可以毫无问题地重复使用相同的函数。顺便说一句，您可能已经注意到，这是我们第一次将一个命名函数放在另一个命名函数（`init`函数）中。这并不是问题，它的工作方式与变量作用域完全相同。换句话说，我们创建的这个函数只在`init`函数范围内可见。

创建标记非常简单；我们只需要创建一个新的`google.maps.Marker`并为其分配一个位置和一个地图。所有其他选项都是可选的。（有关完整列表，请查看[`developers.google.com/maps/documentation/javascript/reference#MarkerOptions`](https://developers.google.com/maps/documentation/javascript/reference#MarkerOptions)上可用的 Google API 文档。）

## 还有更多...

您可能已经注意到我们使用了`map.panTo`方法，但实际上没有发生平移，一切都会立即到位。如果运行地图，您会发现我们实际上并没有看到任何平移；这是因为我们同时切换了地图类型，缩小了地图，并进行了平移。只有平移可以在没有一些技巧和绕过的情况下实际动画化，但所有这些步骤使我们的应用程序变得更加复杂，对动画的实际控制非常有限。在下一个示例中，我们将提出一个解决方案，因为我们使用了两张地图而不是一张地图*自定义控件和重叠地图*。如果我们愿意，我们可以添加延迟并分别执行每个步骤并动画化平移，但如果我们想要创建一个平滑的过渡，我会考虑使用两张叠放在一起的地图，然后淡入和淡出主世界地图的想法。

# 自定义控件和重叠地图

这个示例的目标是练习使用 Google 地图。我们将在本章学到的关于使用 Google 地图的知识，并将我们对用户行为的控制，例如用户可以使用哪些控制器，整合到其中。我们将开始挖掘创建我们自己不支持的未记录的行为，例如锁定用户的平移区域。

在这个示例中，我们的主要任务是将我们在上一个示例中的工作，而不是让地图放大和移动，而是在放大和缩小选项之间创建清晰的过渡；但由于界面不支持以清晰的方式进行，我们将使用外部焦点。这个想法很简单；我们将两张地图叠放在一起，淡入和淡出顶部地图，从而完全控制过渡的流畅性。

## 准备工作

尽管我们是从头开始的，但我们在上一个示例中所做的大部分工作都被重复使用，因此我强烈建议您在进入本示例之前先阅读上一个示例*添加标记和事件*。

在这个示例中，我们还将把 jQuery 整合到我们的工作中，以节省我们在创建自己的动画工具上的时间（或者重用我们在第六章中创建的动画独立图层的工具），因为这会让我们偏离主题。

## 如何做到...

在这个示例中，我们将创建两个文件。一个 HTML 文件和一个 JS 文件。让我们来看看，从 HTML 文件开始：

1.  创建一个 HTML 文件并导入 Google `maps` API 和 jQuery：

```js
<!DOCTYPE html>
<html>
  <head>
    <title>Google Maps Markers and Events</title>
    <meta charset="utf-8" />
    <meta name="viewport" content="initial-scale=1.0, user-scalable=no" />
    <script src="img/jquery.min.js"></script>
    <script src="img/js?key=AIzaSyAywwIFJPo67Yd4vZgPz4EUSVu10BLHroE&sensor=true"></script>
    <script src="img/09.05.controls.js"></script>
  </head>
  <body onload="init();">
    <div id="mapIn"></div>
  <div id="mapOut"></div>
  </body>
</html>
```

1.  使用 CSS 将地图的图层堆叠在一起：

```js
<style>
    html { height: 100% }
    body { height: 100%; margin: 0; padding: 0 }
    #mapIn, #mapOut { height: 100%; width:100%; position:absolute; top:0px; left:0px }
</style>
```

1.  创建`09.05.controls.js` JS 文件，并在其中创建一个`init`函数（从这一点开始，其余的代码将在`init`函数中）：

```js
function init(){
  //rest of code in here
}
```

1.  创建具有自定义信息的两张地图：

```js
var BASE_CENTER = new google.maps.LatLng(40.7142,-74.0064 );

//40.7142¬∞ N, -74.0064 E NYC
var mapOut = new google.maps.Map(document.getElementById("mapOut"),{
  center: BASE_CENTER,
  zoom: 11,
  mapTypeId: google.maps.MapTypeId.SATELLITE,
  disableDefaultUI: true
});
var mapIn = new google.maps.Map(document.getElementById("mapIn"),{
  center: BASE_CENTER,
  zoom: 14,
  mapTypeId: google.maps.MapTypeId.ROADMAP,
  disableDefaultUI: true,
  panControl:true
});
```

1.  将标记添加到上层地图：

```js
var aMarkers = [
  {label:'New York City',
  local: mapOut.getCenter()},
  {label:'Brooklyn',
  local: new google.maps.LatLng(40.648, -73.957)},
  {label:'Queens',
  local: new google.maps.LatLng(40.732, -73.800)}, 
  {label:'Bronx',
  local: new google.maps.LatLng(40.851, -73.871)},  

];
var marker;

  for(var i=0; i<aMarkers.length; i++){
    marker = new google.maps.Marker({
      position: aMarkers[i].local,
      map: mapOut,
      title: aMarkers[i].label
    });

  google.maps.event.addListener(marker, 'click', onMarkerClicked);

  }

  function onMarkerClicked(ev){
    mapIn.panTo(ev.latLng);
    $("#mapOut").fadeOut(1000);
  }
```

1.  将`click`事件添加到内部地图，当您点击它时，将返回到上层地图：

```js
google.maps.event.addListener(mapIn, 'click', function() {
  mapIn.panTo(BASE_CENTER);
  $("#mapOut").fadeIn(1000);
  });
```

1.  使用`center_changed`事件强制用户禁用上层地图中的`pan`：

```js
google.maps.event.addListener(mapOut, 'center_changed', function() {
        mapOut.panTo(BASE_CENTER); 
//always force users back to center point in external map
});
```

当您加载 HTML 文件时，您会发现一个全屏地图，无法拖动。当您点击标记时，它将淡入所选区域。现在您可以在地图周围拖动光标。下次您在内部地图上点击（在任何区域上进行常规点击）时，地图将再次淡出到原始的上层。

## 它是如何工作的...

我们最大的一步是创建两个地图，一个重叠在另一个上面。我们通过一些 CSS 魔术来实现这一点，通过叠加元素并将我们的顶层放在堆栈的最后位置（我们可能可以使用 z-index 来验证它，但它有效，所以我没有将其添加到 CSS 中）。之后，我们创建了两个`div`层并设置了它们的 CSS 代码。在 JavaScript 代码中，与上一个示例中的方式相反，我们将我们想要的值硬编码到了两个地图中。

在两个地图的选项中，我们将通过将属性`disableDefaultUI`设置为`true`来设置默认控制器不生效，而在`mapIn`中，我们将`panControl`设置为`true`，以展示地图可以通过平移来移动：

```js
var mapOut = new google.maps.Map(document.getElementById("mapOut"),{
  center: BASE_CENTER,
  zoom: 11,
  mapTypeId: google.maps.MapTypeId.SATELLITE,
  disableDefaultUI: true
});
var mapIn = new google.maps.Map(document.getElementById("mapIn"),{
  center: BASE_CENTER,
  zoom: 14,
  mapTypeId: google.maps.MapTypeId.ROADMAP,
  disableDefaultUI: true,
  panControl:true
});
```

我们可以通过将布尔值设置为以下任何选项来手动设置所有控制器：

+   `panControl`

+   `zoomControl`

+   `mapTypeControl`

+   `streetViewControl`

+   `overviewMapControl`

我们的`event`逻辑与上一个示例中的逻辑完全相同。唯一的变化在于实际的监听器中，我们使用 jQuery 在地图之间进行切换：

```js
function onMarkerClicked(ev){
  mapIn.panTo(ev.latLng);
  $("#mapOut").fadeOut(1000);
}

google.maps.event.addListener(mapIn, 'click', function() {
  mapIn.panTo(BASE_CENTER);
  $("#mapOut").fadeIn(1000);
});
```

在标记的事件和地图的`click`事件中，我们使用 jQuery 的`fadeIn`和`fadeOut`方法来动画显示我们外部地图的可见性。

## 还有更多...

当您尝试在高级地图（第一个可见地图）周围拖动时，您会注意到地图无法移动——它是不可平移的。Google API v3 不支持禁用平移的功能，但它支持在地图中心点更改时每次获得更新。

因此，我们监听以下更改：

```js
google.maps.event.addListener(mapOut, 'center_changed', function() {
        mapOut.panTo(BASE_CENTER); 
});
```

我们所做的就是每次地图位置发生变化时，强制将其恢复到原始位置，使我们的地图无法移动。

# 使用样式重新设计地图

在使用 Google Maps 创建更高级的应用程序时，您经常会希望创建自己的自定义样式地图。当您希望拥有前景内容并且不希望它与背景内容竞争时，这是非常有用的。

在本示例中，我们将创建一些样式化地图。在本示例结束时，您将知道如何创建全局定制、个体样式，以及添加新地图类型。

这是我们将创建的一个样式：

![使用样式重新设计地图](img/3707OT_09_10.jpg)

这是我们将创建的第二个样式：

![使用样式重新设计地图](img/3707OT_09_11.jpg)

## 准备工作

要完成本示例，您需要从上一个示例的副本开始。我们只描述与本示例中上一个示例不同的新步骤。要查看和理解所有步骤，请阅读*自定义控件和重叠地图*示例。

因此，我们将跳过 HTML 代码，因为它与上一个示例中的代码完全相同。

## 如何做到...

打开上一个示例中的 JavaScript 文件（`09.05.controls.js`），并按照以下步骤操作：

1.  在`init`函数中创建一个`aVeinStyle`数组。该数组包含了所有用于定制地图样式的视觉指南：

```js
var aVeinStyle =  [
  {
    featureType:'water',
    elementType: "geometry",
    stylers:[{color:'#E398BF'}]
  },
  {
    featureType:'road',
    elementType: "geometry",
    stylers:[{color:'#C26580'}]
  },
  {
    featureType:'road.arterial',
    elementType: "geometry",
    stylers:[{color:'#9B2559'}]
  },
  {
    featureType:'road.highway',
    elementType: "geometry",
    stylers:[{color:'#75000D'}]
  },
  {
    featureType:'landscape.man_made',
    elementType: "geometry",
    stylers:[{color:'#F2D2E0'}]
  },
  {
    featureType:'poi',
    elementType: "geometry",
    stylers:[{color:'#C96FB9'}]
  },
  {
    elementType: "labels",
    stylers:[{visibility:'off'}]
  }
];
```

1.  创建一个名为`Veins`的新`google.maps.StyledMapType`地图：

```js
var veinStyle = new google.maps.StyledMapType(aveinStyle,{name: "Veins"});
```

1.  创建一个公交样式：

```js
var aBusStyle =  [
  {
    stylers: [{saturation: -100}]
  },
  {
    featureType:'transit.station.rail',
    stylers:[{ saturation: 60},{hue:'#0044ff'},{visibility:'on'}]

  }
];

var busStyle = new google.maps.StyledMapType(aBusStyle,{name: "Buses"}); 
```

1.  对于内部地图，使地图类型控制器可见，并在其中包括我们新地图样式的 ID：

```js
var mapIn = new google.maps.Map(document.getElementById("mapIn"),{
  center: BASE_CENTER,
  zoom: 14,
  mapTypeId: google.maps.MapTypeId.ROADMAP,
  disableDefaultUI: true,
  panControl:true,
  mapTypeControl:true,
  mapTypeControlOptions: {
    mapTypeIds: [google.maps.MapTypeId.ROADMAP, 'veinStyle', 'busStyle']
  }

});
```

1.  将地图样式信息添加到`mapIn`对象中：

```js
mapIn.mapTypes.set('veinStyle', veinStyle);
mapIn.mapTypes.set('busStyle', busStyle);    
```

1.  设置默认地图类型：

```js
mapIn.setMapTypeId('busStyle');
```

当您重新启动 HTML 文件中的内部地图（在单击标记中的一个后），您将找到一个控制器菜单，可以在自定义地图类型之间切换。

## 它是如何工作的...

使用 Google 样式很有趣，它们的工作方式与 CSS 非常相似。我们设置的样式有几个步骤；第一步是创建样式的规则，下一步是定义一个 Google 样式对象（`google.maps.StyledMapType`），最后一步是定义这个样式信息与哪个地图相关联。样式只能应用于`google.maps.MapTypeId.ROADMAP`类型的地图。

第一个示例是创建公交车样式。这种样式的目标是使地图变成黑白色，并只突出显示公共交通站点：

```js
var aBusStyle =  [
  {
    stylers: [{saturation: -100}]
  },
  {
    featureType:'transit.station.rail',
    stylers:[{ saturation: 60},{hue:'#0044ff'},{visibility:'on'}]

  }
];

var busStyle = new google.maps.StyledMapType(aBusStyle,{name: "Buses"});
```

第一个变量是一个常规数组。我们可以添加任意多个样式；每次我们想要定义规则（搜索条件）之前，都会应用这些规则。让我们更深入地看一下一个样式规则：

```js
{stylers: [{saturation: -100}]}
```

这个例子是最基本的。我们没有规则，或者换句话说，我们想将这种样式应用到所有东西上。就像在这个例子中，我们将饱和度设置为`-100`，我们正在使一切变成黑白色（饱和度默认值为`0`，可以取值在`-100`和`100`之间）。

可能的样式属性如下：

+   `可见性`：这是一个字符串值（`no`，`off`或`simplified`）。这会向地图添加或移除元素；在大多数情况下，它将用于根据提供的信息删除文本，如标签和细节。

+   `伽马`：这是一个介于`0.01`和`10`之间的数字值（默认值为`1.0`）。这个选项控制视图中有多少光。较低的值（低于`1`）会加强较浅和较暗颜色之间的差异，较高的数字（大于`1`）会产生更全局的效果，使一切随着数值的增加而更加发光。

+   `色调`：这是一个十六进制颜色值，包装成字符串（例如#`222222`）。最好的描述色调的方式是，想象戴上与提供的十六进制值匹配的有色玻璃的太阳镜。有色玻璃如何影响你周围的颜色并改变它们的方式，就像地图的色调颜色改变的方式一样。

+   `亮度`：这是一个介于`-100`和`100`之间的值（默认值为`0`）。如果提供一个小于`0`的值，这个效果就非常简单。这与在地图上放置一个黑色矩形并改变其不透明度的效果相同（即，`-30`将与 30%的不透明度相匹配）。你可能已经猜到了正值的结果——对于正值，想法是一样的，但只是用一个白色矩形。

+   `饱和度`：这是一个介于`-100`和`100`之间的值（默认值为`0`）。这个效果侧重于像素级的值，`-100`会创建更接近`100`的灰度图像值。它会从图像中去除所有灰色，使一切更加生动。

这就是所有可用的样式信息，有了它，我们可以控制地图内的每个样式元素。每个样式属性的信息都需要作为`stylers`数组中的单独对象发送；例如，如果我们想要在我们的片段中添加一个`色调`，它会看起来像这样：

```js
{stylers: [{saturation: -40},{hue:60}]}
```

现在我们知道了可以改变地图视觉效果的所有不同方式，是时候了解我们将如何定义应该被选择的内容。在最后的代码片段中，我们控制了整个地图，但我们可以通过添加过滤逻辑来过滤我们想要控制的内容：

```js
{elementType: "geometry",
  stylers:[{color:'#E398BF'}]
```

在这个片段中，我们正在过滤我们想要改变所有`geometry`元素的颜色，这意味着不是`geometry`元素的任何东西都不会受到影响。有三种类型的元素类型选项：

+   `全部`（默认选项）

+   `几何`

+   `标签`

还有一种过滤信息的方法，就是使用`featureType`属性。例如：

```js
  {
    featureType:'landscape.man_made',
    elementType: "geometry",
    stylers:[{color:'#F2D2E0'}]
  }
```

在这种情况下，我们正在列出我们想要关注的内容。我们想关注特征类型和元素类型。如果我们提取`elementType`属性，我们的颜色效果将影响`geometry`和`labels`。而如果我们提取`featureType`，它将影响地图中的所有`geometry`元素。

有关`featureType`属性选项的完整列表，请访问[`goo.gl/H7HSO`](http://goo.gl/H7HSO)。

## 还有更多...

现在我们已经掌握了如何创建我们想要使用的样式，下一个关键步骤是实际将我们的样式与地图连接起来。最简单的方法（如果我们只有一个样式）是直接将其连接到地图上：

```js
inMap.setOptions({styles: styles});
```

这可以通过调用`setOptions`函数或在创建地图时添加`style`属性来完成。样式只能添加到路线图中，因此如果将此样式添加到不是路线图的地图上，它将不会被应用。

由于我们想要添加多个样式选项，我们必须列出地图类型。在这之前，我们需要使用以下代码创建一个新的地图类型对象：

```js
var busStyle = new google.maps.StyledMapType(aBusStyle,{name: "Buses"});
```

在创建新地图时，我们提供了一个名称，该名称将用作我们在控制器中的名称 - 如果我们选择创建一个控制器（在我们的示例中我们会这样做）。重要的是要注意，这个名称不是我们元素的 ID，而只是元素的标签，我们仍然需要在将其发送到地图之前为我们的元素创建一个 ID。为此，我们将首先将 ID 添加到我们的控制器中，并使我们的控制器可见：

```js
var mapIn = new google.maps.Map(document.getElementById("mapIn"),{
  center: BASE_CENTER,
  zoom: 14,
  mapTypeId: google.maps.MapTypeId.ROADMAP,
  disableDefaultUI: true,
  panControl:true,
 mapTypeControl:true,
 mapTypeControlOptions: {
 mapTypeIds: [google.maps.MapTypeId.ROADMAP, 'veinStyle', 'busStyle']
 }

});
```

在此之后，我们将添加设置指令，将我们的新地图类型连接到它们的样式对象：

```js
mapIn.mapTypes.set('veinStyle', veinStyle);
mapIn.mapTypes.set('busStyle', busStyle);
```

最后但同样重要的是，我们可以将默认地图更改为我们的样式地图之一：

```js
mapIn.setMapTypeId('busStyle');
```

就是这样。现在你已经知道了如何在谷歌地图中使用样式的所有必要信息。
