# 第九章：访问本机电话功能

将混合应用与普通的移动友好型 Web 应用区分开来的主要是与底层移动设备的操作系统和硬件进行交互的能力。现代设备为应用程序开发人员提供了大量的服务，从 GPS 和数据库功能到蓝牙、NFC 和其他通信技术。充分利用这些服务可以让我们构建出最好地满足移动用户需求的应用程序。

在本章中，我们将继续在上一章中简要介绍的移动服务的基础上进行构建，并根据需要进行一些刷新。我们的目标是使用 GPS 接收器，这是最普遍的智能手机功能之一，以构建一个简单的导航应用程序。在这样做的过程中，我们还将熟悉一个新的、基本的 AngularJS 组件，称为指令。

# 创建项目

我们将首先设置应用程序的基本结构。与以前一样，我们将选择一个空白项目，并从头开始构建我们的应用程序，以确保我们理解一切是如何工作的：

1.  为您的应用程序创建一个新的项目文件夹。接下来，进入该文件夹并从终端或命令行执行以下操作：

```js
ionic start superNav blank

```

1.  Ionic 现在将下载并配置一切您需要部署一个基本的应用程序（尽管不是一个非常有趣的应用程序）。您可以通过进入您的项目文件夹并执行以下命令来查看它的外观：

```js
ionic serve -l

```

该命令的输出如下截图所示：

![创建项目](img/B04653_09_01.jpg)

现在我们已经有了基本的结构，让我们开始添加一些基本功能。

## 创建基本应用程序结构

我们希望尽可能地保持我们的应用程序简单——一个带有地图的单个屏幕，以及一个工具栏，我们可以在其中放置各种实用程序的按钮，例如查找用户当前位置。

让我们创建一个满足这一要求的基本视图。

打开您的应用程序的`index.html`文件，并确保它看起来像以下内容：

```js
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="initial-scale=1, maximum-scale=1, user-scalable=no, width=device-width">
    <title></title>

    <link href="lib/ionic/css/ionic.css" rel="stylesheet">
    <link href="css/style.css" rel="stylesheet">

    <!-- ionic/angularjs js -->
    <script src="img/ionic.bundle.js"></script>

    <!-- cordova script (this will be a 404 during development) -->
    <script src="img/cordova.js"></script>

    <script src="img/js?key=AIzaSyB16sGmIekuGIvYOfNoW9T44377IU2d2Es&sensor=true"></script>

    <!-- your app's js -->
    <script src="img/app.js"></script>
  </head>

  <body ng-app="supernav" ng-controller="MapCtrl">
    <ion-header-bar class="bar-stable">
      <h1 class="title">SuperNav</h1>
    </ion-header-bar>

    <ion-content scroll="false">
      <div id="map"></div>
    </ion-content>

    <ion-footer-bar class="bar-stable">
    </ion-footer-bar>
  </body>
</html>
```

浏览器预览现在应该是这样的（如果您在上一步关闭了服务器，请随时重新启动并让其运行；它将自动加载对底层源代码所做的任何更改）：

![创建基本应用程序结构](img/B04653_09_02.jpg)

内容多了一点，但目前还没有什么激动人心的。请耐心等待；我们正在努力。

## 将谷歌地图集成到应用程序中

接下来，我们想要集成 UI 最基本的功能——地图。为此，我们将使用**谷歌地图**，这是最受欢迎的地图服务之一。

如果您曾经在移动设备上使用过谷歌地图应用程序或在谷歌上查找过某个地点的方向，您已经熟悉谷歌地图的外观和部分功能。实质上，谷歌地图是一个完整的包，提供了一切——可伸缩的地图、卫星图像、地理编码等等。对于我们的目的来说，它非常完美。

为了将谷歌地图集成到应用程序中，我们需要导入谷歌地图 JavaScript SDK，这是免费提供的。为此，请在`index.html`文件中添加以下导入：

```js
<script src="img/js?sensor=true"></script>
```

接下来，我们需要指定 UI 中应用程序应该绘制的区域。将现有的`div id`标签更改为以下内容：

```js
<div id="map" ng-controller="MapCtrl"></div>
```

为了正确渲染地图，我们需要添加一些自定义的 CSS 来强制地图填充其父容器。打开`www/css/style.css`文件并添加以下内容：

```js
#map {
  display: block;
  width: 100%;
  height: 100%;
}

.scroll {
  height: 100%;
}
```

还要注意，我们为地图的控制器添加了一个绑定。我们将使用这个绑定来执行渲染和处理地图所需的初始配置。因此，让我们继续添加它！在您的项目中创建`www/js/controllers.js`文件，并确保它包含以下内容：

```js
angular.module('supernav.controllers', [])
.controller('MapCtrl', function ($scope) {
  $scope.mapCreated = function (map) {
    $scope.map = map;
  };

  function initialize() {
    var mapOptions = {
      center: new google.maps.LatLng(57.661577, 11.914768),
      zoom: 16,
      mapTypeId: google.maps.MapTypeId.TERRAIN
    };

    $scope.map = new google.maps.Map(
      document.getElementById('map'), mapOptions
    );

    $scope.onCreate({map: map});

    }

    if (document.readyState === "complete") {
      initialize();
    } else {
    google.maps.event.addDomListener(window, 'load', initialize);
    }
});
```

在这里，我们定义了一个新的`supernav.controllers`模块，它将包含我们应用程序的控制器。目前，它只有一个这样的控制器——`MapCtrl`。让我们来看看它，并考虑它的作用：

1.  我们首先定义了地图作用域变量，它将用于简单地引用我们正在处理的地图。我们还定义了一个作用域函数，以便将一个值绑定到这个变量上。

1.  我们定义了`initialize`本地函数，它将用于设置和配置 Google Maps 实例，如下所示：

+   在这里，我们定义了`mapOptions`对象，它提供了要创建的地图的初始设置。它具有以下属性：

`center`：此属性包括地图最初将居中显示的地球表面上的点的纬度和经度坐标。坐标对作为 Google Maps 自己的`LatLng`对象的实例传递。

`zoom`：这是需要应用于地图的位置的缩放程度。

`mapTypeId`：这是我们想要显示的地图类型。Google Maps 支持多种不同的视图模式，从原始卫星图像到道路、企业等的详细视图。

+   然后，我们创建了实际的`Map`对象。我们将以下内容传递给它的构造函数：

这是我们想要绑定地图的**文档对象模型**（**DOM**）的一部分。在我们的情况下，它是具有`map`ID 的`div`，这在我们的视图中早已定义。我们使用了标准的`getElementById` DOM 函数来检索引用。

然后我们有之前定义的`mapOptions`。

最后，在创建了`map`之后，我们将其绑定到之前定义的`map`作用域对象上。

1.  最后，如果 DOM 已完全加载，我们尝试执行刚刚定义的`initialize`函数。如果 DOM 尚未准备好，我们将其注册为需要在准备好时运行的回调函数。

1.  现在我们需要做的就是确保控制器被正确加载并负责地图。为此，首先确保通过将以下内容添加到您的`index.html`文件中来导入 JavaScript 文件：

```js
<script src="img/controllers.js"></script>
```

1.  接下来，修改`app.js`文件，以确保该模块被列为依赖项，如下所示：

```js
angular.module('supernav', ['ionic', 'supernav.controllers'])
```

就是这样！浏览器预览现在应该是这样的：

![将 Google 地图集成到应用程序中](img/B04653_09_03.jpg)

我们已经走得相当远了。虽然我们还没有任何高级的导航功能，但我们已经成功地构建了一个基本的应用程序，人们可以用来浏览世界地图。对于只花了 10 分钟的工作来说，这还不错！

然而，在我们继续之前，值得停下来考虑一下我们迄今为止走过的架构之路。我们在这里所做的一切都是标准的 AngularJS 实践——为我们想要显示的元素（在本例中是`map`）创建一个`view`，为其创建一个`controller`和一些逻辑，并通过将地图集成到应用程序中来激活控制器。然而，您可能还记得我们在本书前面提到过的 AngularJS 最大的优势是创建增强的 HTML。我们已经看到了数据绑定、实时 DOM 更新和其他功能是如何工作的。然而，AngularJS 还提供了定义自定义 HTML 标签的能力，以便在应用程序的多个部分中重用。我们在这里创建的地图就是一个很好的例子。如果我们能将它封装在`<map>`标签中会怎样？我们可以，为了实现这一点，我们需要谈论指令。

# Angular 指令

简而言之，指令是自定义的 HTML 元素。您可以像编写普通的 HTML 元素一样编写它们，但它们的功能完全是以编程方式定义的。因此，它们通过让我们添加任何我们需要的内容来扩展标准的 HTML 语法，以便构建真正动态的页面。

## 创建指令

与我们已经看到的服务和控制器一样，指令被定义为模块的组件，AngularJS 为我们提供了创建它们所需的工具。我们将通过为之前创建的地图创建一个指令来探索这个过程。

我们需要做的第一件事是创建一个名为`www/js/directives.js`的新文件，用于我们项目的指令。创建这个文件并添加以下内容：

```js
angular.module('supernav.directives', [])
.directive('map', function () {
  return {}
});
```

`module`函数用于为模块定义一个指令，正如你可能已经猜到的那样，它的第一个参数是`directive`本身的名称，而第二个参数是一个`factory`函数，它给我们提供了一个描述指令如何工作的对象。在这个意义上，指令类似于我们之前学习的服务。

## 限制指令

让我们开始为我们的`map`指令构建工厂函数。我们应该首先在`directive`中添加一个限制，以告诉 AngularJS 解析器这个特定指令可以出现在哪种 HTML 元素中：

```js
angular.module('supernav.directives', [])
.directive('map', function () {
  return {
    restrict: 'E'
  }
});
```

现在，你可能会惊叹，“E？你说的是什么 E？”嗯，AngularJS 允许我们将指令限制在以下三种不同类别的元素中：

+   **E（元素）**：这些是您标准的 HTML 标签，比如`<map></map>`

+   **A（属性）**：这些是元素属性，比如`<div map></div>`

+   **C（类）**：这些是映射到指令的自定义元素类属性，比如`<div class="map"></div>`

你不必局限于一个限制。例如，你也可以写以下内容以将其限制为元素和属性：

```js
angular.module('supernav.directives', [])
.directive('map', function () {
  return {
    restrict: 'EA'
  }
});
```

因此，如果你写`<map></map>`或`<div map></map>`，Angular 解析器将会检测到指令。

### 注意

你会经常发现，将指令限制在单一类型的元素中是有意义的。这是一个好的实践，因为它减少了你的应用程序的复杂性。

## 作用域隔离

就像控制器一样，指令能够访问它们所在的作用域。然而，为指令创建一个隔离作用域也是可能的（并且通常被认为是良好的实践）。这个作用域将包含一组只有当前指令实例知道的数据。除此之外，作用域隔离还有助于创建可重用的小部件，从而提高代码质量。

我们通过在`directive`中定义作用域注入点来实现这一点，这些注入点将采用标准 HTML 属性的形式：

```js
angular.module('supernav.directives', [])
.directive('map', function () {
  return {
    restrict: 'E',
    scope: {
      onCreate: '&'
    }
});
```

在这里，我们定义了一个名为`onCreate`的注入点，它将指令映射到我们正在隔离的父作用域中的一个`function`（`&`符号表示委托绑定）。例如，假设我们想要将`MapCtrl`中的`onCreate`方法注入到隔离作用域中。那么我们将这样写我们的指令：

```js
<map on-create="mapCreated(map)"></map>
```

此时，`map`参数尚未绑定。稍后，我们将看到如何在`directive`内部定义并传递它到函数中。

然而，在我们继续之前，你是否注意到，尽管我们将我们的注入点命名为`onCreate`，但在实际的 HTML 中我们将其写为`on-create`？这是由于 AngularJS 的一个叫做**规范化**的过程。通过这个过程，属性和标签被转换成更简洁的形式。这个过程的一部分包括用**驼峰命名法**替换**连字符绑定**的单词。如果你想了解更多关于它是如何工作的，我们会给你文档的参考，尽管理解它对于我们在这里开发指令并不是至关重要的。

## DOM 操作

最终，我们希望我们的地图指令能够扩展并在 DOM 中显示地图。为了做到这一点，我们需要允许它实际操作 DOM 本身。

通常实现这一点的方法是为`directive`提供一个`link`函数，它允许它查看 DOM 更新过程。让我们为我们的`map`指令添加一个链接，如下所示：

```js
angular.module('supernav.directives', [])
.directive('map', function () {
  return {
    restrict: 'E',
    scope: {
      onCreate: '&'
    },
    link: function ($scope, $element, $attr) {
      function initialize() {
        var mapOptions = {
          center: new google.maps.LatLng(43.07493, -89.381388),
          zoom: 16,
          mapTypeId: google.maps.MapTypeId.ROADMAP
        };
        var map = new google.maps.Map($element[0], mapOptions);

        $scope.onCreate({map: map});

        google.maps.event.addDomListener(
          $element[0], 'mousedown', function (e) {
            e.preventDefault();
            return false;
        });
      }

      if (document.readyState === "complete") {
        initialize();
      } else {
        google.maps.event.addDomListener(window, 'load', initialize);
      }
    }
  }
});
```

看起来很奇怪，不是吗？这是我们之前在`controller`中定义的`initialize`函数和相关地图设置过程，尽管有一些细微的修改。我们已经讨论过这是如何工作的。所以，让我们看看它如何在`link`函数的上下文中发挥作用：

`link`函数接受以下三个参数：

+   `$scope`：这是指令渲染的作用域。

+   `$element`：这是指令绑定的标签，在我们的例子中是`<map>`。该标签被包裹在类似于 JQuery 的 jqLite 库中，这使我们能够直接对其进行操作。

+   `$attr`：这定义了指令元素的属性及其关联的值。

在`initialize`函数内部，我们现在使用`$element[0]`来获取标签本身的名称（在我们的例子中是`map`）。我们还使用`$scope`参数来调用父作用域中的`onCreate`委托（请注意，我们在这种情况下需要明确定义参数名称及其关联的值）。

# 把所有东西整合在一起

现在我们有了一个完整的指令，是时候将其整合到我们的应用程序中了。首先确保新的指令已经正确加载。`app.js`文件的第一行应该是这样的：

```js
angular.module('supernav', ['ionic', 'supernav.controllers', 'supernav.directives'])
```

同样，`index.html`文件应该包含以下导入：

```js
<script src="img/directives.js"></script>
```

接下来，确保`index.html`中的`ion-content`部分现在看起来像这样：

```js
<ion-content scroll="false">
  <map on-create="mapCreated(map)"></map>
</ion-content>
```

接下来，由于我们将`map`的渲染逻辑移到了指令中，所以从`controller.js`文件中将其移除，现在它应该是这样的：

```js
angular.module('supernav.controllers', [])
.controller('MapCtrl', function ($scope) {
  $scope.mapCreated = function (map) {
    $scope.map = map;
  };
});
```

最后，我们需要对`style.css`进行一些小的修改，以确保`map`指令能够正确渲染在应用程序上。确保它看起来像这样：

```js
map {
  display: block;
  width: 100%;
  height: 100%;
}

.scroll {
  height: 100%;
}
```

就是这样！如果需要的话，请确保在浏览器中重新加载预览。它应该看起来和我们不使用指令时一样。我们已经成功地把所有东西都整合在一起了！

# 通过 GPS 添加地理定位

现在我们有了一个更好的工作地图视图，让我们继续为其添加一些基本的导航功能，即，将地图聚焦在我们当前的物理位置上。

首先，让我们暂时在页脚工具栏中添加一个`button`。确保`ion-footer-bar`标签看起来像下面这样：

```js
<ion-footer-bar class="bar-stable">
  <a ng-click="centerOnUser()" class="button button-icon icon ion-navigate"></a>
</ion-footer-bar>
```

您的预览现在应该是这样的：

![通过 GPS 添加地理定位](img/B04653_09_04.jpg)

现在，我们需要利用本机设备的位置功能，以找到用户当前的位置。幸运的是，这可以直接通过 HTML5 地理位置接口来实现。为了看到它的效果，让我们在`controller.js`文件中`MapCtrl`控制器内添加以下内容：

```js
$scope.centerOnUser = function () {
  console.log("Centering on user");
  if (!$scope.map) {
    return;
  }

  navigator.geolocation.getCurrentPosition(function (pos) {
    console.log('Got pos', pos);
    $scope.map.setCenter(
      new google.maps.LatLng(pos.coords.latitude, pos.coords.longitude));
  }, function (error) {
    alert('Unable to get location: ' + error.message);
  });
};
```

注意我们在这里做了什么。

我们定义了`centerOnUser()`作用域函数。这个函数又绑定到我们刚刚在`index.html`文件中定义的位置按钮上。

在这个函数中，我们使用`navigator.geolocation.getCurrentPosition`函数，这是 HTML5 标准的一部分，以获取用户当前位置。这个函数接受一个回调作为参数，回调又以一个位置对象`pos`作为参数。

在回调函数中，我们使用存储在`pos`中的纬度和经度值，以便使用 Google Maps 提供的`setCenter()`方法来重新设置地图的中心。

现在您可以在预览中尝试导航功能。点击位置按钮应该会将地图的焦点切换到您当前的位置。就是这样。我们完成了！

# 总结

在本章中，我们直接使用了 Google Maps API 来渲染地图并处理地图。虽然这很好，但我们还建议您查看可以在[`angular-ui.github.io/angular-google-maps/#!/`](http://angular-ui.github.io/angular-google-maps/#!/)找到的模块，它使得在 AngularJS 中使用 Google Maps 变得更加容易。

除了我们在这里介绍的导航功能之外，移动设备当然还提供了大量的服务。幸运的是，几乎所有的服务都有相应的 Cordova 插件，这将让您能够使用这些服务。我们建议您在需要找到插件时参考 Cordova 插件注册表，网址为[`plugins.cordova.io/#/`](http://plugins.cordova.io/#/)。

您可以在谷歌地图上做的远远不止我们在这里探讨的内容。我们强烈建议您探索谷歌地图的开发者页面，该页面位于[`developers.google.com/maps/`](https://developers.google.com/maps)，以更好地了解有关谷歌地图的可用可能性。

我们在这里构建的导航应用实际上是 Ionic 中的`map`起始应用程序。您可以在[`github.com/driftyco/ionic-starter-maps`](https://github.com/driftyco/ionic-starter-maps)找到它的完整源代码。

在本章中，您学习了如何利用手机的原生功能和谷歌地图来构建一个简单的导航应用程序。最重要的是，您还学习了 AngularJS 中的指令，这是一个强大的功能，可以让您创建自定义的 HTML 元素。
