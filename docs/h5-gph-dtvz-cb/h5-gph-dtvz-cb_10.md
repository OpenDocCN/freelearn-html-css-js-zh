# 第十章：地图行动

在本章中，我们将涵盖以下主题：

+   将 Twitter 动态连接到 Google 地图

+   构建一个高级交互式标记

+   将多条推文添加到信息窗口气泡中

+   自定义标记的外观和感觉

+   最终项目：构建实时行程

# 介绍

在这一章中，我们将更深入地与数据可视化这一主题联系起来。如今，可视化数据最流行的方式之一是使用地图。在本章中，我们将探讨如何将数据集成到地图中，使用 Google 地图平台。

# 将 Twitter 动态连接到 Google 地图

这是一个非常有趣的与 Google 地图的实验的开始。任务的目标是在 Twitter 帖子和 Google 地图之间创建一个链接。我们需要几个配方才能达到最终目标。在本配方结束时，我们将拥有一个 Google 地图。这个 Google 地图在屏幕的任何区域都可以点击。当用户点击地图时，他们将连接到 Twitter API，并搜索该区域中包含“HTML5”一词的推文。当结果返回时，它将在被点击的区域上弹出一个新的标记，并添加来自该位置的关于该主题的最新推文。在这个阶段，它只是一个带有悬停效果的标记，显示我们的实际推文而没有更多信息。

![将 Twitter 动态连接到 Google 地图](img/3707OT_10_01.jpg)

## 准备工作

如果你还没有阅读第九章*使用 Google 地图*，你可能会发现本章有点困难，所以我鼓励你在开始本章之前阅读它。在这个阶段，你应该已经设置了 Google API（参见第九章中的*获取 Google API 密钥*配方）。

## 如何做...

我们将创建新的 HTML 和 JavaScript 文件，并分别称它们为`10.01.socielmap.html`和`10.01.socielmap.js`，然后执行以下步骤：

1.  在 HTML 文件中添加以下代码，使用你自己的 API 密钥：

```js
<!DOCTYPE html>
<html>
  <head>
    <title>Google Maps Markers and Events</title>
    <meta charset="utf-8" />
    <meta name="viewport" content="initial-scale=1.0, user-scalable=no" />
    <style>
      html { height: 100% }
      body { height: 100%; margin: 0; padding: 0 }
      #map { height: 100%; width:100%; position:absolute; top:0px; left:0px }
    </style>
  <script src="img/jquery.min.js"></script>
    <script src="img/js?key=AIzaSyAywwIFJPo67Yd4vZgPz4EUSVu10BLHroE&sensor=true">
    </script>
    <script src="img/10.01.socielmap.js"></script>
  </head>
 <body onload="init();">
    <div id="map"></div>
  </body>
</html>
```

1.  让我们进入 JavaScript 文件。由于`onload`事件触发时会调用`init()`函数，我们将把所有代码放在一个新的`init`函数中。

```js
function init(){
 //all code here
}
```

1.  我们将从设置地图的中心点开始。到目前为止，我们一直非常关注我的家乡纽约州，所以让我们把注意力转向欧洲。

```js
var BASE_CENTER = new google.maps.LatLng(48.516817734860105,13.005318750000015 );
```

1.  接下来，让我们为地图创建一个黑白风格，这样更容易专注于我们即将创建的标记。

```js
var aGray =  [
    {
      stylers: [{saturation: -100}]
    }
  ];

  var grayStyle = new google.maps.StyledMapType(aGray,{name: "Black & White"});
```

1.  创建一个新的 Google 地图。

```js
var map = new google.maps.Map(document.getElementById("map"),{
    center: BASE_CENTER,
    zoom: 6,
    mapTypeId: google.maps.MapTypeId.ROADMAP,
    disableDefaultUI: true,

  });
```

1.  设置`grayStyle`样式对象为我们的默认样式。

```js
 map.mapTypes.set('grayStyle', grayStyle);
 map.setMapTypeId('grayStyle');
```

1.  我们的下一步是使用 Google 的 API 为地图创建一个新的`click`事件。当地图被点击时，我们希望触发一个`listener`函数。当发生点击时，我们希望开始我们的 Twitter 搜索，因为我们将连接到 Twitter API，并搜索关键词`html5`在我们点击地图的位置的 50 公里半径内的提交。让我们创建一个新的鼠标事件并启动 Twitter 搜索。

```js
google.maps.event.addListener(map, 'click', function(e) {
    //console.log(e.latLng);
      var searchKeyWord = 'html5';
      var geocode=e.latLng.lat() + "," + e.latLng.lng()+",50km";
      var searchLink = 'http://search.twitter.com/search.json?q='+ searchKeyWord+ '&geocode=' + geocode +"&result_type=recent&rpp=1";

      $.getJSON(searchLink, function(data) {
        showTweet(data.results[0],e.latLng);
    });

    });
```

1.  当 Twitter 搜索值返回时，是展示我们的新推文的时候；如果没有找到推文，我们将放入默认内容，让用户知道找不到任何内容。

```js
function showTweet(obj,latLng){
      if(!obj) obj = {text:'No tweet found in this area for this topic'};
      console.log(obj);

      var marker = new google.maps.Marker({
            map: map,
            position: latLng,
            title:obj.text    });

  }
```

当你再次加载地图时，你会发现一个欧洲地图等待点击。每次点击都会触发一个新的 Twitter 搜索，并根据你点击的位置生成一个新的结果。要阅读推文，请在返回后将鼠标悬停在标记上。

## 它是如何工作的...

我们生活在一个几乎任何数据都与地理位置数据和地图重叠的时代。几乎不可能写一本关于数据的书而不谈论地图，也不可能写一本关于数据可视化的书而不至少打开地图世界和其可能性的潘多拉魔盒。

最近，Twitter 一直在努力捕获用户的位置。大部分时间，位置仍然是空的。话虽如此，Twitter 始终根据用户的信息知道用户的基本位置，尤其是当用户通过他们的手机发送推文时。因此，Twitter 总是大致知道用户发送消息时的位置，未来几年，这种准确性预计只会变得更好。在未来，越来越多的 Twitter 结果将为用户提供如此准确的位置，以至于我们将能够直接在地图上找到他们。

创建地图后，第一步是，一旦用户点击地图的任何区域，我们就开始构建要在 Twitter API 上使用的搜索查询：

```js
google.maps.event.addListener(map, 'click', function(e) {
    //console.log(e.latLng);
 var searchKeyWord = 'html5';
 var geocode=e.latLng.lat() + "," + e.latLng.lng()+",50km";
      var searchLink = 'http://search.twitter.com/search.json?q='+ searchKeyWord+ '&geocode=' + geocode +"&result_type=recent&rpp=1";
```

我们不涵盖搜索的所有可能性，而是专注于两个主要点：搜索查询，在我们的案例中是 HTML5，以及查询的位置。我们直接从传递给标记的事件中获取位置信息。我们重新格式化来自我们的 Google 返回事件的信息，并将其格式化为一个字符串，添加到其中的范围；在我们的案例中，我们将其设置为 50 公里（您也可以选择`ml`表示英里）。由于我们现在正在查看欧洲地图，我认为使用公里而不是英里会更合适。

我们希望将我们的搜索值作为**JavaScript 对象表示**（**JSON**）值返回。 JSON 是在服务器之间以字符串形式传递对象信息的一种非常简洁的方式。在大多数情况下，您通常会使用自动转换器，因此您将发送对象并获取对象，但在幕后有一个 JSON 编码器和解码器来处理请求。

### 注意

如果您不知道 JSON 是什么，不用担心；这一切都是在后台完成的，不必理解 JSON 的工作原理才能使用它。

我们希望以 JSON 格式获取我们的数据；为此，我们将将我们的 URL 参数发送到以下 URL：

[`search.twitter.com/search.json`](http://search.twitter.com/search.json)

附加到其中我们的`q`值和`geocode`值。如果您想更深入地探索 Twitter 搜索 API 的选项和可能性，请访问以下页面：

[`dev.twitter.com/docs/api/1/get/search`](https://dev.twitter.com/docs/api/1/get/search)

下一步是将我们的信息发送到此服务并获取我们的结果。为此，我们将使用 jQuery 中的`$.getJSON`函数。此函数将处理我们所有的需求：发送我们的请求，获取它，然后将信息解码为常规的 JavaScript 对象。

```js
$.getJSON(searchLink, function(data) {
        showTweet(data.results[0],e.latLng);
    });
```

我们需要发送的两个参数是搜索链接和返回函数。在我们的案例中，我们将获取我们的数据并将其发送到外部函数`showTweet`。我们将仅发送数据的第一个结果以返回和我们从点击事件中获取的`e.latLng`对象信息。

是时候创建标记了。在`showTweet`函数中，我们的第一个任务是检查返回的第一个元素中是否实际上有任何数据。如果没有值，这意味着 Twitter 没有找到任何内容。

```js
if(!obj) obj = {text:'No tweet found in this area for this topic'};
```

如果没有返回对象，我们将创建一个新对象，其中包含占位符信息，以替换常规结果信息。这是避免代码复杂性的好方法：通过将异常构建到常规用户体验中。我们完成了；我们所要做的就是创建标记。

```js
var marker = new google.maps.Marker({
            map: map,
            position: latLng,
            title:obj.text    });
```

尽管我们得到了我们心中所想的，但我们拥有的`latLng`信息是我们点击的位置，而不是我们推文的确切位置。目前返回的 Twitter 对象中有一个名为 geo 的属性。在撰写本书时，它总是返回为空。目前看起来它是一个即将发布或部分实现的功能，所以在阅读本书时，请尝试检查并查看`obj.geo`属性是否返回一个值，并在此信息可用时使用它使您的观点更准确。

# 构建一个高级交互式标记

我们社交地图项目的下一步是为我们的 Twitter 搜索结果添加更多细节。当 Twitter 结果出现时，我们希望自动打开一个信息面板。在这个过程中，我们将创建一个 Google 标记的子类，并扩展它，并添加一个新的 InfoWindow，使我们能够将实时 HTML 数据直接添加到我们的地图中。

![构建高级交互式标记](img/3707OT_10_02.jpg)

## 准备工作

如果您没有从本章的开头开始，加入会很困难。由于这个配方是上一个配方的延续，我们不会创建新的 HTML 文件或新的 JavaScript 文件，而是会从我们离开的地方继续。

## 操作步骤...

拿出你最新的 JavaScript 文件，让我们继续下一步：

1.  在`showTweet`函数中，用一个新的`TwitterMarker`标记替换新标记。

```js
function showTweet(obj,latLng){
      if(!obj) obj = {text:'No tweet found in this area for this topic'};
      console.log(obj);	

 var marker = new TwitterMarker({
 map: map,
 position: latLng,
 tweet: obj,
 title:obj.text    }); 

  }
```

1.  现在我们不再使用常规的内置标记，是时候为我们创建自己的标记了。让我们从构造函数开始。

```js
function TwitterMarker(opt){
  var strTweet = this.buildTwitterHTML(opt.tweet);
  this.infoWindow = new google.maps.InfoWindow({
      maxWidth:300,
      content:strTweet
  });

  this.setValues(opt);
  this.infoWindow.open(this.map,this);
  google.maps.event.addListener(this, 'click', this.onMarkerClick);
}
```

1.  我们将要从`google.maps.Marker`标记中扩展我们的新对象，以便我们可以拥有常规标记的所有功能。

```js
TwitterMarker.prototype = new google.maps.Marker();
```

1.  让我们在标记事件监听器中创建一个切换按钮。当调用事件时，它将打开或关闭我们的 InfoWindow：

```js
TwitterMarker.prototype.onMarkerClick=function(evt){
  this.isOpen=!this.isOpen;
  if(this.isOpen)
    this.infoWindow.close();
  else
    this.infoWindow.open(this.map,this);

}
```

1.  是时候通过创建 HTML 字符串来创建 Twitter 消息了。

```js
TwitterMarker.prototype.buildTwitterHTML = function(twt){
  var str;
  if(twt.from_user_name){
   str =	  "<span><img style='float: left' src='"+twt.profile_image_url+"' />"+
         "<b>" +twt.from_user_name + "</b><br/><a href ='http://twitter.com/"
        + twt.from_user + "'>@"+twt.from_user+"</a><br/> "
        + twt.location + "</span>"
        + "<p>"+twt.text+"</p>";
  }else{
    str="The 50 Kilometer radius around this point did not message this value";
  }
  return str;
}
```

如果重新加载 HTML 文件，您会发现它在世界的任何地方都是交互式的；如果它能找到一条推文，它将在地图上输出它。

## 工作原理...

尽管代码行数不多，但其中包含了许多逻辑。让我们从新标记开始。这是本书中第一次使用继承。继承，顾名思义，使我们能够在 JavaScript 中扩展对象的功能，而不影响原始对象。在我们的情况下，我们希望获取标记的所有功能（方法、属性等），并为它们添加一些自定义行为。

在 JavaScript 中，通过定义原型来完成继承。到目前为止，我们使用原型而没有过多地谈论它，但我们主要用它来创建新方法。如果我们将一个完整的对象分配给原型，那么该对象的所有属性和方法也将被复制到我们的新对象中。

```js
TwitterMarker.prototype = new google.maps.Marker();
```

### 注意

始终首先通过扩展要扩展的对象来开始，然后再进行任何其他添加。这是因为如果在这行代码之前放置了任何新的原型方法，它们将被静默删除，因此将无法工作。

`buildTwitterHTML`方法接收返回的 Twitter 对象并将其数据转换为 HTML。我们每个标记使用此方法一次。当我们创建一个新的标记时，我们也创建一个新的`InfoWindow`对象。我们将 InfoWindow 放在标记的顶部，并展示推文信息。

```js
 function TwitterMarker(opt){
  var strTweet = this.buildTwitterHTML(opt.tweet)
  this.infoWindow = new google.maps.InfoWindow({
      maxWidth:300,
      content:strTweet
  });
```

我们还设置了宽度，以避免出现一个非常大的面板。我们将新创建的`strTweet`字符串发送到`infoWindow`对象中。

我们希望我们的标记是一个切换按钮，用于控制 InfoWindow 的状态。为此，我们添加了一个新动态创建的名为`isOpen`的属性。即使我们在构造函数中打开了 InfoWindow，我们也没有在那里设置`isOpen`的值。我们可以在标记的点击事件监听器中执行的第一个操作中解决这个问题。

```js
TwitterMarker.prototype.onMarkerClick=function(evt){
  this.isOpen=!this.isOpen;
```

当点击标记时，我们会自动改变`isOpen`变量的状态。因为它之前没有设置，现在将被设置为`true`。`!`运算符是一个布尔运算符，可以在`true`和`false`之间切换值。它的实际含义是`not`。换句话说，我们是在说：

```js
this.isOpen = is not this.isOpen
```

有趣的事实是，对于非（`!`）运算符的未定义值（未定义的变量的值）与`false`、`null`甚至`0`是相同的。任何其他值都被视为 true。这样，每次点击标记时，变量`this.isOpen`的值都会切换。这就是我们切换按钮逻辑的核心。现在只剩下决定是打开还是关闭 InfoWindow。

```js
  if(this.isOpen)
    this.infoWindow.close();
  else
    this.infoWindow.open(this.map,this);
}
```

这将引导我们进入最后一步，找出我们的 Twitter 文本是什么。我们将在接下来的几个步骤中编辑这个方法。您可以随意尝试，并根据自己的喜好进行个性化。我们有两种可能的结果：*搜索区域中没有 Twitter 消息*和*搜索区域中有 Twitter 消息*。如果有消息，我们将使用一些返回对象属性来构建一个 HTML 大纲，该大纲将用于与此标记相关联的`infoWindow`对象内。如果没有结果，我们将创建一个。

```js
TwitterMarker.prototype.buildTwitterHTML = function(twt){
  var str;
  if(twt.from_user_name){
   //build custom message
   /*notice we are validating based on checking if the twitter has a property (any of the properties would work) */
  }else{
    //the error message
    str="The 50 Kilometer radius around this point did not message this value";
  }
  return str;
}
```

就是这样！我们的社交地图开始变得更加有趣。它仍然缺少一些功能。如果我们可以在 InfoWindow 结果中看到多条消息，那将是非常好的。在下一个步骤中，我们将尝试解决这个问题。

# 将多条推文添加到 InfoWindow 气泡中

到目前为止，在我们的交互式社交地图中，我们在每个点击的位置添加了标记，并打开了一个包含推文信息的 InfoWindow。我们的下一步将是通过在窗口中添加分页系统，使多条推文可以存在于我们的 InfoWindow 中。

![将多条推文添加到 InfoWindow 气泡中](img/3707OT_10_03.jpg)

## 准备工作

要完成这个步骤，您必须深入了解我们的整体章节。如果您刚刚加入，最好回到本章的开头，因为我们将从上一个步骤离开的地方继续进行。

## 操作步骤如下：

我们仍然在我们的 JavaScript 文件中，并将继续添加代码和调整我们的代码，以便将多个 Twitter 帖子添加到我们的社交地图中。

1.  让我们首先将 Twitter 搜索更改为每次搜索返回最多 100 个值。我们这样做是因为我们调用 Twitter API 的次数是有限的。因此，我们将尝试一次性获取尽可能多的内容（此代码应该在第 30 行左右）。

```js
var searchLink = 'http://search.twitter.com/search.json?q='+ searchKeyWord+ '&geocode=' + geocode +"&result_type=recent&rpp=100";

```

1.  由于我们现在要处理返回的所有推文，我们需要更改我们发送到`TwitterMaker`标记的引用，发送完整的数组（代码片段中的更改已突出显示）。

```js
  google.maps.event.addListener(map, 'click', function(e) {
    //console.log(e.latLng);
      var searchKeyWord = 'html5';
      var geocode=e.latLng.lat() + "," + e.latLng.lng()+",50km";
      var searchLink = 'http://search.twitter.com/search.json?q='+ searchKeyWord+ '&geocode=' + geocode +"&result_type=recent&rpp=100";

      $.getJSON(searchLink, function(data) {
 showTweet(data.results,e.latLng);
    });

    });

  function showTweet(a,latLng){
      if(!a) a = [{text:'No tweet found in this area for this topic'}];
      //console.log(obj);	

      var marker = new TwitterMarker({
            map: map,
            position: latLng,
            tweet: a,
            title:a[0].text    });			

  }
}
```

1.  我们希望更新`TwitterMarker`构造函数，包括我们的数组以及有关它的快速信息，例如总推文数和当前所在的推文。我们需要一种方法来标识我们的对象，因此我们也会给它一个 ID（在接下来的几个步骤中会详细介绍）。

```js
 function TwitterMarker(opt){
 this.count = opt.tweet.length;
 this.crnt = 0;
 this.id = TwitterMarker.aMarkers.push(this);
 this.aTweets = opt.tweet;
  var strTweet = this.buildTwitterHTML(opt.tweet[0])
  this.infoWindow = new google.maps.InfoWindow({
      maxWidth:300,
      content:strTweet
  });

  this.setValues(opt);
  this.infoWindow.open(this.map,this);
  google.maps.event.addListener(this, 'click', this.onMarkerClick);
}
```

1.  我们希望在我们的代码中的任何地方都可以访问一个静态数组，其中包含所有创建的标记。为此，我们将在`TwitterMarker`类中添加一个新的状态数组：

```js
TwitterMarker.prototype = new google.maps.Marker();
TwitterMarker.aMarkers= [];

```

1.  在`buildTwitterHTML`方法中，我们希望添加回/下一个链接，用户可以从 InfoWindow 中看到：

```js
TwitterMarker.prototype.buildTwitterHTML = function(twt){
  var str;
  if(twt.from_user_name){
   str =	  "<span><img style='float: left' src='"+twt.profile_image_url+"' />"+
         "<b>" +twt.from_user_name + "</b><br/><a href ='http://twitter.com/"
        + twt.from_user + "'>@"+twt.from_user+"</a><br/> "
        + twt.location + "</span>"
        + "<p>"+twt.text+"</p>";

 if(this.count>1){
 str+="<span style='absolute; bottom: 0;
 right: 0px; width:80px'>";
 if(this.crnt!=0) str+="<a href='javascript:TwitterMarker.aMarkers["+(this.id-1)+"].prev();'>&lt;</a> ";
 str+= (this.crnt+1) + " of " + this.count;
 if(this.crnt<(this.count-1)) str+= "<a href='javascript:TwitterMarker.aMarkers["+(this.id-1)+"].next();'>&gt;</a> ";
 str+= "</span>" 
 }
  }else{
    str="The 50 Kilometer radius around this point did not message this value";
  }
  return str;
}
```

1.  现在让我们添加`next`和`prev`方法。

```js
TwitterMarker.prototype.next =function(){
  this.infoWindow.close();
  this.infoWindow.content = this.buildTwitterHTML(this.aTweets[++this.crnt]);
  this.infoWindow.open(this.map,this);
  return false;	
}

TwitterMarker.prototype.prev =function(){
  this.infoWindow.close();
  this.infoWindow.content = this.buildTwitterHTML(this.aTweets[--this.crnt]);
  this.infoWindow.open(this.map,this);
  return false;	
}
```

加载 HTML 文件，您应该会发现一个可以容纳每次点击最多 100 条推文的工作 InfoWindow。

## 工作原理...

我们的第一个改变是改变了从 Twitter 搜索 API 返回的结果数量。这个改变迫使我们改变了代码中的引用，不再直接引用第一个返回的对象，而是专注于完整的结果对象，并将其发送到我们的`TwitterMarker`构造函数。这个改变也在构造函数内的信息流中创建了一些较小的改变。

我们的目标是创建两个按钮，这些按钮将更新我们的 InfoWindow。这是一个问题，因为我们需要标记和其 InfoWindow 之间的双向连接。到目前为止，我们与 InfoWindow 的所有通信都是单向的。我们解决这个问题并绕过 Google 接口的最简单方法是创建一个静态数组，该数组将存储所有标记，并在`InfoWindow`对象内部触发按钮时引用我们的静态标记。我们只需要向我们的类名添加一个变量 direction。

```js
TwitterMarker.aMarkers= [];
```

通过直接将变量添加到`TwitterMarker`类中，我们现在可以在任何时候直接引用它，并且它不会在我们的对象中重复（因为它不是原型的一部分）。现在我们有了一个数组，是时候回到我们的`TwitterMarker`构造函数中，每次创建一个新的`TwitterMarker`对象时，都向这个数组发送一个新的引用。我们通过这样做得到的另一个好处是，我们自动获得一个唯一的标识符（ID），因为返回的数字将始终是我们需要的唯一数字。

```js
this.id = TwitterMarker.aMarkers.push(this);
```

在这一行代码中，我们执行了前一段中讨论的所有任务。数组`push`方法返回数组的新长度。

现在我们有了一种引用我们的标记并获得标识符的方法，是时候回到`buildTwitterHTML`方法中，并在呈现的 HTML 中添加两个`href`按钮，当单击下一个/上一个选择时，将触发正确的标记。

在我们深入研究之前，我们要检查并验证我们是否有多于一个返回的 Twitter 消息；如果没有，添加新逻辑就没有意义，如果我们为只有一个项目的项目引入了下一个/上一个逻辑，我们将引入一个错误。

```js
if(this.count>1){

}
```

通过以下`if`语句，我们可以确定我们当前是否在第一个 Twitter 消息中，如果不是，我们将添加返回按钮：

```js
if(this.crnt!=0) str+="<a href='javascript:TwitterMarker.aMarkers["+(this.id-1)+"].prev();'>&lt;</a> ";
```

这可能看起来很混乱，但是，如果我们忽略 HTML，专注于当按钮被按下时将触发的实际 JavaScript，我们将得到这样的结果：

```js
TwitterMarker.aMarkers["+(this.id-1)+"].prev();
```

`this.id-1`参数将被实际的当前数字替换：

由于这是渲染为要解析为 HTML 的字符串，将集成到 HTML 中的值将是硬编码的。让我们看一个真实案例来澄清这一点。第一个数组 ID 将是`0`，因此`prev`按钮将如下代码语句所示：

```js
TwitterMarker.aMarkers[0].prev();
```

现在逻辑开始显露出来。通过从数组中抓取我们当前元素的标记，我们所要做的就是触发`prev`方法并让它接管。

对于另一端，同样的逻辑发生了。唯一的条件是我们不在最后的 Twitter 结果中，如果不是，我们调用`next`方法：

```js
if(this.crnt<(this.count-1)) str+= "<a href='javascript:TwitterMarker.aMarkers["+(this.id-1)+"].next();'>&gt;</a> ";

```

到此为止！我们的逻辑核心已经就位。

如果我们愿意，我们可以通过用一个唯一 ID 包装`<div>`标签来创建我们的 InfoWindow，并直接调用它并直接更新我们的内容（尝试自己做这个，因为那将是一个更好的解决方案）。相反，我们正在处理 InfoWindow 的限制。由于我们不能在打开时更新完整的 bucket 容器，我们需要关闭它以更新它，然后再次打开它。因此，我们在`next`和`prev`方法中的逻辑是相似的；两者都对正在呈现的实际值的更改有限制。

```js
TwitterMarker.prototype.next =function(){
  this.infoWindow.close();
  this.infoWindow.content = this.buildTwitterHTML(this.aTweets[++this.crnt]);
  this.infoWindow.open(this.map,this);
  return false;	
}

TwitterMarker.prototype.prev =function(){
  this.infoWindow.close();
 this.infoWindow.content = this.buildTwitterHTML(this.aTweets[--this.crnt]);
  this.infoWindow.open(this.map,this);
  return false;	
}
```

所有的逻辑都是相同的，并且限制在高亮显示的代码片段中。如果你不熟悉这个快捷方式，当`++`和`--`操作符设置在变量之前时，它们使我们能够在变量上加/减 1，并在发送其值之前更新它。因此，在一行中，我们既可以改变变量中的数字，又可以发送新创建的数字以继续其任务。

在`next`方法的情况下，我们想抓取下一个推文，而在`prev`方法的情况下，我们想抓取上一个推文。

# 自定义标记的外观和感觉

这将是我们社交地图的最后一个配方。在这个配方中，我们将重新审视我们的标记本身，并对其进行改头换面。由于我们的标记代表了点击区域的 Twitter 消息，我们将更新我们的标记，使其看起来像 Twitter 鸟（手工制作）。我们不会止步于此；在更新我们的图形后，我们将添加另一个图形层来阴影我们的 Twitter 标记。它将是一个阴影，其不透明度将根据推文数量（最多一百条推文）从零到完全不同。

了解我们的目标的最佳方法是查看以下屏幕截图：

![自定义标记的外观和感觉](img/3707OT_10_04.jpg)

请注意，一些推文没有可见的圆形轮廓，而另一些则有非常深的轮廓（这取决于推文的数量）。

## 准备工作

要完成此任务，您需要首先完成本章中的所有先前的配方。

## 操作步骤...

我们将直接进入 JavaScript 文件，并从上一个步骤离开的地方继续。

1.  更新`showTweet`函数。

```js
function showTweet(a,latLng){
      if(!a) a = [{text:'No tweet found in this area for this topic'}];
      //console.log(obj);	

      var marker = new TwitterMarker({
            map: map,
            position: latLng,
            tweet: a,
            title:a[0].text,
            icon:"img/bird.png"    });			

  }
```

1.  在`TweeterMarker`构造函数中创建`MarkerCounter`对象的实例。

```js
function TwitterMarker(opt){
  this.count = opt.tweet.length;
 this.mc = new MarkerCounter(opt);
  this.crnt = 0;
...
```

1.  创建`MarkerCounter`构造函数。

```js
function MarkerCounter(opt) {
    this.radius = 15;
    this.opacity = (opt.tweet.length) /100;
    this.opt = opt;
    this.setMap(opt.map);
  }
```

1.  为`google.maps.OverlayView`对象创建子类`MarkerCounter`。

```js
MarkerCounter.prototype = new google.maps.OverlayView();
```

1.  创建`onAdd`方法。当元素添加到地图中时，它将自动调用。在此方法中，我们将完成所有绘制的准备工作，但不会绘制元素。

```js
MarkerCounter.prototype.onAdd = function() {
  var div = document.createElement('div');
    div.style.border = "none";
    div.style.borderWidth = "0px";
    div.style.position = "absolute";

  this.canvas = document.createElement("CANVAS");
    this.canvas.width = this.radius*2;
  this.canvas.height = this.radius*2;

    this.context = this.canvas.getContext("2d");
  div.appendChild(this.canvas);
  this.div_ = div;

    var panes = this.getPanes();
  	panes.overlayLayer.appendChild(div);

 }
```

1.  最后但并非最不重要的是，现在是时候重写`draw`方法，并在上一步创建的新画布元素中绘制，并定位`div`元素。

```js
 MarkerCounter.prototype.draw = function() {
    var radius = this.radius;
    var context = this.context;
  context.clearRect(0,0,radius*2,radius*2);

  context.fillStyle = "rgba(73,154,219,"+this.opacity+")";
  context.beginPath();
    context.arc(radius,radius, radius, 0, Math.PI*2, true);
  context.closePath();
  context.fill();
  var projection = this.getProjection();
  var point = projection.fromLatLngToDivPixel(this.opt.position);

    this.div_.style.left = (point.x - radius) + 'px';
    this.div_.style.top = (point.y - radius) + 'px';

  };
```

当您运行应用程序时，您会发现现在我们的标记看起来像 Twitter 的标记，而来自某个位置的推文数量越多，我们 Twitter 鸟下的蛋就会越不透明。

## 工作原理...

第一步是交换默认标记的图形。由于我们正在扩展常规标记，因此我们拥有其所有默认功能和行为。其中一个功能是交换图标。为此，我们将我们的对象参数之一作为图标及其路径传递。

```js
var marker = new TwitterMarker({
            map: map,
            position: latLng,
            tweet: a,
            title:a[0].text,
 icon:"img/bird.png"    });

```

您可能想知道这实际上是如何工作的，因为我们在代码中实际上没有对图标参数做任何事情。这很简单。如果您深入研究`TwitterMaker`构造函数，您会发现以下行：

```js
this.setValues(opt);
```

将`setValues`方法传递给`opt`对象是我们让标记继续并使用我们刚刚在构造函数中获得的信息渲染我们的标记的方式。所有常规标记中可以完成的事情在我们的标记中也可以完成。

在这个阶段，我们的 Twitter 鸟是我们标记的图形界面。不幸的是，这是我们可以自定义标记的最远的地方；接下来，我们需要添加另一个视觉层。由于我们希望创建一个视觉层，其行为就像标记一样（因为它将成为标记的一部分），我们需要为`google.maps.OverlayView`对象创建一个子类。

与标记逻辑类似，当我们准备渲染元素时，我们希望调用`setMap`方法（对于标记，这是一个不同的方法，但是相同的想法）。

```js
function MarkerCounter(opt) {
    this.radius = 15;
    this.opacity = (opt.tweet.length) /100;
    this.opt = opt;
 this.setMap(opt.map);
  }

  MarkerCounter.prototype = new google.maps.OverlayView();
```

在我们的构造函数中，我们只存储非常基本的全局信息，例如我们的目标不透明度、半径和`options`对象。我们可以在这里存储任何我们想要的信息。我们将需要的最重要的信息元素是位置（纬度和经度）。我们将发送该信息到我们的标记，并且它将在我们的`opt`对象中。

`google.maps.OverlayView`对象有一个`onAdd`方法。它就像一个监听器，但另外，我们将覆盖此方法并在元素添加到地图中时添加我们的处理/准备工作。

```js
MarkerCounter.prototype.onAdd = function() {
  var div = document.createElement('div');
    div.style.border = "none";
    div.style.borderWidth = "0px";
    div.style.position = "absolute";

  this.canvas = document.createElement("CANVAS");
    this.canvas.width = this.radius*2;
  this.canvas.height = this.radius*2;

    this.context = this.canvas.getContext("2d");
  div.appendChild(this.canvas);
  this.div_ = div;
    var panes = this.getPanes();
    panes.overlayLayer.appendChild(div);
  }
```

这里的大部分逻辑应该看起来很熟悉。我们首先创建一个新的`div`元素。我们设置它的 CSS 属性，使`div`元素的位置绝对，这样我们就可以轻松地移动它。然后我们创建一个 canvas 元素，并将其宽度和高度设置为我们圆的半径的两倍。我们将 canvas 添加到我们的`div`元素中。最后但并非最不重要的是，是时候将我们的`div`元素添加到地图中了。我们将通过访问`getPanes`方法来实现这一点。这个方法将返回这个元素可以包含的所有视觉层。在我们的情况下，我们将直接进入我们的覆盖层，并将我们的`div`元素添加到其中。我们之所以在`onAdd`方法中这样做，而不是在之前就这样做，是因为覆盖物不会被渲染，我们将无法访问前面代码中的最后两行。

就像我们重写了`onAdd`方法一样，我们也对`draw`方法做了同样的事情。这是我们最后的关键步骤。在大部分情况下，这个方法中的所有工作都会非常熟悉，因为在这本书中我们已经玩过很多 canvas。所以，让我们探索新的步骤，找到我们想要定位覆盖物的位置。

```js
var projection = this.getProjection();
	var point = projection.fromLatLngToDivPixel(this.opt.position);

    this.div_.style.left = (point.x - radius) + 'px';
    this.div_.style.top = (point.y - radius) + 'px';
```

在前面代码的第一行中，我们获取了投影。投影是我们覆盖物的相对视角。通过这个投影，我们可以提取出实际的像素点。我们调用`projection.fromLatLngToDivPixel`方法，向它发送一个经纬度对象，并得到一个点（`x`，`y`值）。现在我们只需要根据这些信息更新我们的`div`元素的样式，并将其定位到这个信息（不要忘记减去我们的半径大小，这样我们的元素就恰好位于被点击的实际点的中间）。

到目前为止，我们一直把我们的`TwitterMarker`构造函数当作世界上总是有推文的地方，但现实是有时候可能什么都没有，而现在我们正在创建一个无法工作的可视化和一个无法可视化它的标记。让我们重写我们的行为，如果没有结果，就放上一个替代的标记，并跳过所有我们的自定义。

![它是如何工作的...](img/3707OT_10_05.jpg)

让我们整理一下。我们首先从`showTweet`方法中删除我们原始的错误逻辑。相反，我们只会更新`text`属性，而不会创建一个新的数组。

```js
function showTweet(a,latLng){
      var marker = new TwitterMarker({
            map: map,
            position: latLng,
            tweet: a,
 title:a.length? a[0].text : 'No tweet found in this area for this topic' ,
            icon:"img/bird.png"    });			

  }
```

如果你对三元运算符不熟悉，它是一种在代码中创建`if...else`语句的非常简洁的方式。它的核心逻辑如下：

```js
condition?true outcome:false outcome;
```

然后结果被发送回来，我们可以将其捕捉到我们的变量中，就像我们在这种情况下所做的那样。

接下来我们想要改变的是`TwitterMarker`构造函数。

```js
function TwitterMarker(opt){
 if(!opt.tweet || !opt.tweet.length){
 opt.icon = "img/x.png"; 
 }else{

    this.count = opt.tweet.length;
    this.mc = new MarkerCounter(opt);
    this.crnt = 0;
    this.id = TwitterMarker.aMarkers.push(this);
    this.aTweets = opt.tweet;
    var strTweet = this.buildTwitterHTML(opt.tweet[0])
    this.infoWindow = new google.maps.InfoWindow({
        maxWidth:300,
        content:strTweet
    });

    this.infoWindow.open(this.map,this);
    google.maps.event.addListener(this, 'click', this.onMarkerClick);

 }
 this.setValues(opt);
}
```

这里的主要变化是，我们首先通过检查是否有任何推文来启动我们的应用程序。如果周围没有推文，我们将图标图形更新为一个新的**X**图标。如果我们有结果，一切都保持不变。我们将`setValues`方法提取出来，以便在任何情况下都需要调用它。

完成了！我们完成了我们的社交地图。你可以用这个项目做更多的事情。一些例子可能是使更改搜索词更容易，并比较两个搜索结果（这可能非常有趣且容易）。我会很感兴趣地看到全世界提到 Flash 与 HTML5 的次数，所以如果你做到了，给我发封电子邮件。

# 最终项目：构建一个实时行程

尽管从我们之前的示例中自然的下一步是向我们已经在本章中构建的不断增长的社交地图添加额外的功能，但我们正在改变方向。

在我们的最终配方中，我们将构建一个交互式的 Google 地图，它将以我在写这本书时在南美洲的一位亲密朋友的旅行信息为动画。为了构建这个应用程序，我们将通过添加绘图和移动标记来为地图添加动画；我们将与旅行信息的外部源集成，并集成动画和文本片段来描述旅程。在下面的屏幕截图中，你可以看到一个非常小的纯路径快照：

![最终项目：构建实时行程](img/3707OT_10_06.jpg)

## 准备就绪

在这个配方中，我们将使用许多元素，这些元素都是基于我们在所有章节中所做的工作。因此，如果你没有和我们一起经历这段旅程，那么就不容易立即开始。没有先决条件。我们将从头开始，但不会专注于我们已经学过的东西。

当用户在世界地图上“旅行”时，如果数据源中有用户的消息，地图将变暗，并在用户继续旅行之前显示消息：

![准备就绪](img/3707OT_10_07.jpg)

## 如何做...

在这个配方中，我们将创建两个文件：一个 HTML 文件和一个 JavaScript 文件。让我们来看看它们，从 HTML 文件开始：

1.  创建 HTML 文件。

```js
<!DOCTYPE html>
<html>
  <head>
    <title>Google Maps Markers and Events</title>
    <meta charset="utf-8" />
    <meta name="viewport" content="initial-scale=1.0, user-scalable=no" />
    <link href='http://fonts.googleapis.com/css?family=Yellowtail' rel='stylesheet' type='text/css'>
    <style>
      html { height: 100% }
      body { height: 100%; margin: 0; padding: 0 }
      #map { height: 100%; width:100%; position:absolute; top:0px; left:0px }

      .overlay {
        background: #000000 scroll;
        height: 100%;
        left: 0;
        opacity: 0;
        position: absolute;
        top: 0;
        width: 100%;
        z-index: 50;
    }
    .overlayBox {
        left: -9999em;
        opacity: 0;
        position: absolute;
        z-index: 51;
        text-align:center;
        font-size:32px;
        color:#ffffff;
        font-family: 'Yellowtail', cursive;
    }
    </style>
  <script src="img/jquery.min.js"></script>
    <script src="img/js?key=AIzaSyBp8gVrtxUC2Ynjwqox7I0dxrqjtCYim-8&sensor=false"></script>
    <script src="img/jsapi"></script>

    <script src="img/10.05.travel.js"></script>
  </head>
  <body>
    <div id="map"></div>
  </body>
</html>
```

1.  现在是时候转到 JavaScript 文件`10.05.travel.js`了。我们将通过初始化可视化库并存储全局地图变量来开始。

```js
 google.load('visualization', '1.0');
google.setOnLoadCallback(init);
```

1.  var map; 当触发`init`函数时，地图被加载，并触发加载 Google 电子表格，我们将在其中存储所有朋友的旅行信息。

```js
function init() {
  var BASE_CENTER = new google.maps.LatLng(48.516817734860105,13.005318750000015 );

  map = new google.maps.Map(document.getElementById("map"),{
    center: BASE_CENTER,
    mapTypeId: google.maps.MapTypeId.SATELLITE,
    disableDefaultUI: true,

  });
  var query = new google.visualization.Query(
      'https://spreadsheets.google.com/tq?key=0Aldzs55s0XbDdERJVlYyWFJISFN3cjlqU1JnTGpOdHc');
    query.send(onTripDataReady);

}
```

1.  文档加载时，它将触发`onTripDataReady`监听器。当发生这种情况时，我们将想要创建一个新的`GoogleMapTraveler`对象（一个用于管理我们的体验的自定义类）。

```js
function onTripDataReady(response){
  var gmt = new GoogleMapTraveler(response.g.D,map);	
}
```

1.  `GoogleMapTraveler`对象的构造方法将准备我们的变量，创建一个新的`Animator`对象、一个`Traveler`对象和一个新的`google.maps.Polyline`对象，并通过调用`nextPathPoint`方法触发第一个旅行点的创建。

```js
function GoogleMapTraveler(aData,map){
  this.latLong; //will be used to store current location
  this.zoomLevel; //to store current zoom level
  this.currentIndex=0;
  this.data = aData; //locations
  this.map = map;

  //this.setPosition(0,2);
  this.animator = new Animator(30);

  this.pathPoints = [this.getPosition(0,1)]; //start with two points at same place.

  var lineSymbol = {
        path: 'M 0,-1 0,1',
        strokeOpacity: .6,
        scale: 2
      };

    this.lines = new google.maps.Polyline({
        path: this.pathPoints,
        strokeOpacity: 0,
        strokeColor: "#FF0000",
        icons: [{
          icon: lineSymbol,
          offset: '0',
          repeat: '20px'
        }],
        map: map
      });

  this.traveler = new Traveler(this.map,this.getPosition(0,1));
  this.nextPathPoint(1);

}
```

1.  `getPosition`方法是一个非常聪明、小巧的方法，它使我们能够每次调用时创建一个新的`google.maps.LatLng`对象，并根据点的平均值或一个项目创建一个点。

```js
  GoogleMapTraveler.prototype.getPosition = function (index,amount){
  var lat=0;
  var lng=0;
  for(var i=0; i<amount; i++){
    lat+= parseFloat(this.data[index+i].c[0].v);
    lng+= parseFloat(this.data[index+i].c[1].v);

  }
  var ll=new google.maps.LatLng(
            lat/amount,
            lng/amount);
  return ll;
}
```

1.  我们希望能够设置我们旅行者的位置，因此我们还希望创建一个`setPosition`方法。

```js
GoogleMapTraveler.prototype.setPosition = function(index,amount){
  this.currentFocus = index;

  var lat=0;
  var lng=0;
  for(var i=0; i<amount; i++){
    lat+= parseFloat(this.data[index+i].c[0].v);
    lng+= parseFloat(this.data[index+i].c[1].v);

  }
  var ll=new google.maps.LatLng(
            lat/amount,
            lng/amount);

  if(this.data[index].c[2])this.map.setZoom(this.data[index].c[2].v);
  this.map.setCenter(ll);

}
```

1.  我们的应用程序的核心是能够自动从一步移动到下一步。这种逻辑是使用我们的`Animator`对象结合`nextPathPoint`方法应用的：

```js
GoogleMapTraveler.prototype.nextPathPoint = function(index){
  this.setPosition(index-1,2);
  this.pathPoints.push(this.getPosition(index-1,1)); //add last point again
  var currentPoint = this.pathPoints[this.pathPoints.length-1];
  var point = this.getPosition(index,1);

  //console.log(index,currentPoint,point,this.getPosition(index,1));
  this.animator.add(currentPoint,"Za",currentPoint.Za,point.Za,1);
  this.animator.add(currentPoint,"Ya",currentPoint.Ya,point.Ya,1);
  this.animator.add(this.traveler.ll,"Za",this.traveler.ll.Za,point.Za,2,0.75);
  this.animator.add(this.traveler.ll,"Ya",this.traveler.ll.Ya,point.Ya,2,0.75);

 this.animator.onUpdate = this.bind(this,this.renderLine);
 this.animator.onComplete = this.bind(this,this.showOverlayCopy);//show copy after getting to destination
}
```

1.  有两个回调通过我们的`Animator`对象触发（它们在前面的代码片段中突出显示）。现在是时候创建更新`onUpdate`回调信息的逻辑了。让我们来看看`renderLine`方法。

```js
GoogleMapTraveler.prototype.renderLine = function(){
  this.lines.setPath(this.pathPoints);
  if(this.traveler.isReady)this.traveler.refreshPosition();
}
```

1.  在下一步中，当动画完成时，它会触发我们的覆盖逻辑。覆盖逻辑非常简单；如果在 Google 文档的第五列中有文本，我们将使屏幕变暗并输入文本。如果没有文本，我们将跳过这一步，直接进入触发`hideOverlayCopy`方法的下一步（电子表格中的下一行）。

1.  我们`GoogleMapTraveler`对象的先前方法是`bind`方法。我们已经在第六章*将静态事物变得生动*的*转向面向对象编程*配方中创建了这个`bind`方法；因此，我们不会进一步详细说明。

```js
GoogleMapTraveler.prototype.bind = function(scope, fun){
   return function () {
        fun.apply(scope, arguments);
    };
}
```

1.  创建`Traveler`类。`Traveler`类将基于本章中*自定义标记的外观和感觉*配方中的工作，只是这一次它将是一个动画标记。

```js
function Traveler(map,ll) {
  this.ll = ll;
    this.radius = 15;
    this.innerRadius = 10;
    this.glowDirection = -1;
    this.setMap(map);
    this.isReady = false;

  }

  Traveler.prototype = new google.maps.OverlayView();

  Traveler.prototype.onAdd = function() {
  this.div = document.createElement("DIV");
  this.canvasBG = document.createElement("CANVAS");
    this.canvasBG.width = this.radius*2;
  this.canvasBG.height = this.radius*2;
  this.canvasFG = document.createElement("CANVAS");
    this.canvasFG.width = this.radius*2;
  this.canvasFG.height = this.radius*2;

  this.div.style.border = "none";
  this.div.style.borderWidth = "0px";
  this.div.style.position = "absolute";

  this.canvasBG.style.position = "absolute";
  this.canvasFG.style.position = "absolute";

  this.div.appendChild(this.canvasBG);
  this.div.appendChild(this.canvasFG);

    this.contextBG = this.canvasBG.getContext("2d");
    this.contextFG = this.canvasFG.getContext("2d");

  var panes = this.getPanes();
    panes.overlayLayer.appendChild(this.div);

  }

  Traveler.prototype.draw = function() {
    var radius = this.radius;
    var context = this.contextBG;

  context.fillStyle = "rgba(73,154,219,.4)";
  context.beginPath();
    context.arc(radius,radius, radius, 0, Math.PI*2, true);
  context.closePath();
  context.fill();

  context = this.contextFG;
  context.fillStyle = "rgb(73,154,219)";
  context.beginPath();
    context.arc(radius,radius, this.innerRadius, 0, Math.PI*2, true);
  context.closePath();
  context.fill();

    var projection = this.getProjection();

    this.updatePosition(this.ll);
    this.canvasBG.style.opacity = 1;
    this.glowUpdate(this);
    setInterval(this.glowUpdate,100,this);
    this.isReady = true;

  };

  Traveler.prototype.refreshPosition=function(){
    this.updatePosition(this.ll);	
  }

  Traveler.prototype.updatePosition=function(latlng){
    var radius = this.radius;
    var projection = this.getProjection();
  var point = projection.fromLatLngToDivPixel(latlng);
    this.div.style.left = (point.x - radius) + 'px';
    this.div.style.top = (point.y - radius) + 'px';	
  }

  Traveler.prototype.glowUpdate=function(scope){ //endless loop
    scope.canvasBG.style.opacity = parseFloat(scope.canvasBG.style.opacity) + scope.glowDirection*.04;
    if(scope.glowDirection==1 && scope.canvasBG.style.opacity>=1) scope.glowDirection=-1;
    if(scope.glowDirection==-1 && scope.canvasBG.style.opacity<=0.1) scope.glowDirection=1;
  }
```

1.  我们将获取在第六章中创建的*Animating independent layers*食谱中创建的`Animator`类，并对其进行调整（代码片段中的更改已突出显示）。

```js
function Animator(refreshRate){
 this.onUpdate = function(){};
 this.onComplete = function(){};
  this.animQue = [];
  this.refreshRate = refreshRate || 35; //if nothing set 35 FPS
  this.interval = 0;
}

Animator.prototype.add = function(obj,property, from,to,time,delay){
  obj[property] = from;

  this.animQue.push({obj:obj,
            p:property,
            crt:from,
            to:to,
            stepSize: (to-from)/(time*1000/this.refreshRate),
            delay:delay*1000 || 0});

  if(!this.interval){ //only start interval if not running already
    this.interval = setInterval(this._animate,this.refreshRate,this);	
  }

}

Animator.prototype._animate = function(scope){
  var obj;
  var data;

  for(var i=0; i<scope.animQue.length; i++){
      data = scope.animQue[i];

      if(data.delay>0){
        data.delay-=scope.refreshRate;
      }else{
        obj = data.obj;
 if((data.stepSize>0 && data.crt<data.to) ||
 (data.stepSize<0 && data.crt>data.to)){

          data.crt = data.crt + data.stepSize;
          obj[data.p] = data.crt;
        }else{
          obj[data.p] = data.to;	
          scope.animQue.splice(i,1);
          --i;
        }
      }

  }
 scope.onUpdate();
  if(	scope.animQue.length==0){
    clearInterval(scope.interval);
    scope.interval = 0; //reset interval variable
 scope.onComplete();
  }
}

```

当您加载 HTML 文件时，您会发现一个全屏地图，它从电子表格中获取方向。它将以动画方式显示我朋友从以色列到南美洲再返回时所走过的路径。

## 它是如何工作的...

在这个例子中有许多组件，但我们主要关注我们在本书的其他部分中尚未涵盖的新步骤。

我们遇到的第一件新事物就在我们的 HTML 和 CSS 中：

```js
<link href='http://fonts.googleapis.com/css?family=Yellowtail' rel='stylesheet' type='text/css'>
```

我们从 Google 字体库[`www.google.com/webfonts`](http://www.google.com/webfonts)中选择了一种字体，并将其集成到文本覆盖中。

```js
.overlayBox {
       ...
        font-family: 'Yellowtail', cursive;
    }
```

现在是时候进入我们的 JavaScript 文件了，我们首先加载 Google 可视化库。这是我们在第八章中使用的相同库，*与 Google 图表一起玩*。加载完成后，将触发`init`函数。`init`函数启动我们的地图，并开始加载电子表格。

在第八章的*将数据源更改为 Google 电子表格*食谱中，*与 Google 图表一起玩*，我们首次使用了 Google 电子表格。在那里，您学会了准备和添加 Google 图表到 Google 可视化中所涉及的所有步骤。在我们的情况下，我们创建了一个包含我朋友走过的所有地区的线路的图表。

![它是如何工作的...](img/3707OT_10_08.jpg)

在这种情况下的例外是，我们不想将我们的 URL 提供给 Google 图表，而是想直接使用它。为此，我们将使用 Google 的 API 接口之一，`google.visualization.Query`对象：

```js
var query = new google.visualization.Query(
      'https://spreadsheets.google.com/tq?key=0Aldzs55s0XbDdERJVlYyWFJISFN3cjlqU1JnTGpOdHc');
    query.send(onTripDataReady);
```

下一步是创建我们的`GoogleMapTraveler`对象。Google 地图旅行者是我们与 Google 地图一起工作的一种新方式。它不扩展 Google 地图的任何内置功能，而是我们过去创建的所有其他想法的中心。它被用作标记管理中心，称为 Traveler，我们将很快创建的`google.maps.Polyline`对象，它使我们能够在地图上绘制线条。

我们不想让新添加到 Google 地图中的线条呈现出非常静态的外观，而是想为其创建一个揭示效果。为了实现这一点，我们需要一种方法，每隔几毫秒更新折线以创建动画。一开始，我就知道起点和终点，因为我从之前创建的 Google 电子表格中获取了这些信息。

这个想法非常简单，尽管在一个非常复杂的生态系统中。这个想法是创建一个数组，用于存储所有的纬度/经度点。然后，每当我们想要更新屏幕时，将其提供给`this.line`对象。

这个应用程序的逻辑核心存储在这行代码中：

```js
this.nextPathPoint(1);
```

它将开始一个递归的旅程，遍历图表中的所有点。

## 还有更多...

让我们更深入地了解`GoogleMapTraveler.prototype.nextPathPoint`方法背后的逻辑。在这个函数中，我们首先要做的事情是设置地图视图。

```js
this.setPosition(index-1,2);
```

`setPosition`方法执行了一些与根据发送的当前索引中的数据重新定位地图和缩放级别相关的操作。它比这更智能，因为它接受第二个参数，使其能够对两个点进行平均。当一个人在两点之间旅行时，最好是我们的地图位于两点的中心。通过将`2`作为第二个参数发送进去来实现。`setPosition`方法的内部逻辑很简单。它将循环遍历需要的项目，以平均出正确的位置。

接下来，我们向我们的`this.pathPoints`数组添加一个新点。我们首先复制已经在数组中的相同点，因为我们希望我们的新第二点从起点开始。这样，我们可以每次更新数组中的最后一个值，直到它达到终点（真正的下一个点）。

```js
this.pathPoints.push(this.getPosition(index-1,1)); //add last point again
```

我们创建了一些辅助变量。一个将指向我们刚刚创建并推送到我们的`pathPoints`数组中的新对象。第二个是我们希望在动画结束时达到的点。

```js
var currentPoint = this.pathPoints[this.pathPoints.length-1];
var point = this.getPosition(index,1);
```

### 注意

第一个变量不是一个新对象，而是对最后创建的点的引用，第二行是一个新对象。

我们的下一步将是开始并动画化我们的`currentPoint`的值，直到它达到`point`对象中的值，并更新我们的旅行者纬度/经度信息，直到它也到达目的地。我们给第二个动画延迟 0.75 秒，以使事情更有趣。

```js
this.animator.add(currentPoint,"Za",currentPoint.Za,point.Za,1);
this.animator.add(currentPoint,"Ya",currentPoint.Ya,point.Ya,1);
this.animator.add(this.traveler.ll,"Za",this.traveler.ll.Za,point.Za,2,0.75);
this.animator.add(this.traveler.ll,"Ya",this.traveler.ll.Ya,point.Ya,2,0.75);
```

在结束这个方法之前，我们想实际动画化我们的线。现在，我们正在动画化两个不可视的对象。为了开始动画化我们的可视元素，我们将监听更新，直到我们完成动画。

```js
this.animator.onUpdate = this.bind(this,this.renderLine);
  this.animator.onComplete = this.bind(this,this.showOverlayCopy);//show copy after getting to destination
```

每次动画发生时，我们在`renderLine`方法中更新我们的可视元素的值。

为了避免运行时错误，我们为旅行者标记添加了一个`isReady`布尔值，以指示我们的元素何时准备好绘制。

```js
this.lines.setPath(this.pathPoints);
if(this.traveler.isReady)this.traveler.refreshPosition();
```

当动画完成时，我们转到`showOverlayCopy`方法，在那里我们接管屏幕并以与之前相同的策略动画复制。这一次，当我们完成这个阶段时，我们将再次触发我们的初始函数，并以更新后的索引重新开始整个循环。

```js
GoogleMapTraveler.prototype.hideOverlayCopy = function(){
  //update index now that we are done with initial element
  this.currentIndex++;
  ...

  //as long as the slide is not over go to the next.
  if(this.data.length>this.currentIndex+1)this.nextPathPoint(this.currentIndex+1); 
}
```

这涵盖了我们构建的核心。现在是时候简要谈谈另外两个类，这些类将帮助创建这个应用程序。

### 理解旅行者标记

我们不会深入研究这个类，因为在很大程度上，它是基于我们在上一个配方中所做的工作，“自定义标记的外观和感觉”。最大的区别是，我们在我们的元素中添加了内部动画和一个`updatePosition`方法，使我们能够在需要移动时移动我们的标记。

```js
 Traveler.prototype.updatePosition=function(latlng){
    var radius = this.radius;
    var projection = this.getProjection();
  var point = projection.fromLatLngToDivPixel(latlng);
    this.div.style.left = (point.x - radius) + 'px';
    this.div.style.top = (point.y - radius) + 'px';	
  }
```

这个方法得到一个纬度和经度，并更新标记的位置。

由于我们正在动画化这个对象的实际`ll`对象在主类中，我们添加了第二个方法`refreshPosition`，每次更新动画时都会调用它。

```js
Traveler.prototype.refreshPosition=function(){
    this.updatePosition(this.ll);	
  }
```

在这个课程中还有更多可以探索和发现的东西，但我会留给你一些乐趣。

### 更新动画对象

我们对我们的`Animator`类进行了两个重大更新，这个类最初是在第六章的“独立层动画”配方中创建的，“让静态事物活起来”。第一个变化是集成回调方法。回调的概念与事件非常相似。回调使我们能够在发生某事时调用一个函数。这种工作方式使我们一次只能有一个监听器。为此，我们首先创建了以下两个变量，它们是我们的回调函数：

```js
function Animator(refreshRate){
  this.onUpdate = function(){};
  this.onComplete = function(){};
```

然后我们在`Animator`类中的相关位置（在更新或完成时）触发这两个函数。在我们的`GoogleMapTraveler`对象中，我们用`GoogleMapTraveler`对象内部的函数覆盖了默认函数。

我们对`Animator`对象的第二个也是最后一个重大更新是，我们添加了更智能、更详细的逻辑，使我们的动画师能够动画化正负区域。我们的原始动画没有考虑到动画化纬度/经度值，因此我们调整了核心动画逻辑。

这涵盖了我们在这个食谱中探索的一些重要新事物。这个食谱中还充满了我们在各章中学到的许多其他小东西。我真诚希望你喜欢和我一起走过这段旅程，因为这是我们书的结尾。请随时与我分享你的工作和见解。你可以在[`02geek.com`](http://02geek.com)找到我，我的电子邮件是`<ben@02geek.com>`。
