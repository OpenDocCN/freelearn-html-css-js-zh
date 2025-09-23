# 第七章。访问设备传感器和位置 API

*使用设备传感器为复杂应用打开了大门，这些应用可能会改善用户体验并增强现代应用的功能。对于移动开发者来说，了解设备传感器的功能和限制，以便有效地使用 PhoneGap 框架提供的 API，是非常重要的。位置数据允许移动开发者将每条信息与设备的地理位置标记。在本章中，您还将学习如何将位置 API 与您的应用结合使用。*

在本章中，您将：

+   学习最常见的设备传感器及其如何用于增强用户体验

+   使用加速度计了解设备方向和设备运动事件概述

+   学习如何直接使用 JavaScript 与设备传感器协同工作

+   学习如何使用 PhoneGap 的指南针 API

+   了解地理位置及其数据在设备中的可用性

+   学习如何使用 PhoneGap 地理位置 API 以及如何在应用中集成 Google Maps API

位置数据允许移动开发者将每条信息与设备的地理位置标记。这种基于内容的标记使得使用非常具体的应用成为可能。PhoneGap 框架提供了一个简单易用、易于理解且功能强大的地理位置 API。

# 介绍设备传感器

人类有感官（触觉、听觉、嗅觉等）；手机有数字“感官”：触觉、地理位置、方向和运动。传感器是测量物理量并将其转换为软件可理解的信号的设备组件。现代智能手机配备了各种传感器，可以在用户完成日常任务时提供支持。通过访问设备传感器，您可以增强最终用户的使用体验并开发复杂的应用。

传感器可以是基于硬件的或基于软件的。基于硬件的传感器是嵌入到手机或平板电脑中的物理组件。它们通过直接测量特定的环境属性（如加速度、地磁场强度或角度变化）来获取数据。基于软件的传感器不是物理设备，尽管它们模仿基于硬件的传感器。

典型的设备传感器包括**加速度计**、**陀螺仪**、**指南针**、**气压计**、**方向传感器**等。

并非所有设备及其操作系统都支持相同的传感器，因此在考虑在您的应用中使用哪些传感器之前，您必须知道您想要针对哪些设备。设备传感器通常分为以下几类：

+   运动传感器

+   环境传感器

+   位置传感器

**运动传感器**测量沿三个轴的加速度力和旋转力。属于这一类别的硬件部件包括加速度计、重力传感器、陀螺仪和旋转矢量传感器。**环境传感器**测量各种环境参数，如环境空气温度和压力、照明和湿度。气压计、光度计和温度计属于这类传感器。**位置传感器**测量设备的物理位置。

如前所述，每个操作系统都提供不同的传感器。从开发者的角度来看，这意味着要在不同的平台上工作，你必须了解每个平台上传感器的工作方式。当使用 PhoneGap 时，你可以安全地在不同平台上使用**加速度计**和**指南针**API。此外，你可以依赖**内置浏览器**的功能来获取额外的传感器信息，例如设备方向。

加速计实际上由三个加速度计组成，每个加速度计都测量沿轴 *x*、*y* 和 *z* 的线性路径上的速度变化（即线性加速度）随时间的变化。结合三个加速度计的数据，你可以得到设备的运动和方向。

陀螺仪始终是运动传感器的一部分，它测量围绕三个轴的旋转速率，通常是**翻滚**、**俯仰**和**偏航**。

磁力计测量设备周围磁场以及在没有任何强局部场的情况下，这些测量将参照地球的磁场。这样，设备能够确定其相对于地磁北极的**航向**；使用航向值，也可以确定设备的**偏航**。即使用户在设置应用中关闭了位置更新，磁航向更新也是可用的；报告的值是从**0**到**360**的正数。当用户以横向模式握持设备时，用户的实际航向是报告的航向加上 90 度。

iOS 平台提供了开发者可以期待的所有常见传感器，例如加速度计、磁力计、陀螺仪和接近传感器。

Android 平台提供了四个额外的传感器，允许你监控各种环境属性：环境**湿度**、**亮度**、环境**压力**和环境**温度**。所有传感器都是基于硬件的，并且只有当制造商将它们集成到设备中时才可用。

### 注意

你可以在 Google Play 商店找到 Android 传感器的完整演示；只需搜索并安装*Android Sensor Box*应用程序。

Windows Phone 7.5/8 平台对传感器提供了广泛的支持。你可以使用**倾斜仪**传感器来检测设备的俯仰、滚转和偏航，或者你可以使用**四元数**传感器（四元数是三维空间中两条有向线的商）来创建复杂的 3D 应用程序。有关 Windows Phone 传感器 API 的完整概述，请参阅[`msdn.microsoft.com/library/windows/apps/windows.devices.sensors`](http://msdn.microsoft.com/library/windows/apps/windows.devices.sensors)在线文档。

设备的位置能力依赖于几个被称为位置传感器的传感器。设备通常使用多种定位方法来提供不同粒度的位置数据。位置数据的来源在准确性、启动时间和功耗特征方面有所不同，包括以下内容：

+   GPS

+   A-GPS

+   细胞塔三角测量

+   Wi-Fi 三角测量

+   IP 地址

随着传感器的持续发展，最终用户的需求在增长，市场上可用的应用程序质量也在提高。

### 注意

**Lapka Electronics**发布了一套传感器和应用程序，能够将环境数据转换为易于读取的数值。使用他们的传感器和应用程序，你可以测量电磁污染、湿度、原始食品和饮用水中硝酸盐的含量等。有关这些传感器的更多信息可在[`mylapka.com/`](http://mylapka.com/)在线获取。

## 传感器和人与计算机的交互

传感器的发展非常迅速，并且仍在快速发展，影响着创意人士如何设计应用程序。新一代应用程序依赖于语音命令、手势等，以便用户能够以更直观的方式控制应用程序。现在，应用程序能够根据收集到的传感器数据感知用户意图。使用传感器使应用程序（和计算机）更易于控制的做法被称为**感知计算**。这一倡议由英特尔领导，包括视频会议、游戏等多种应用。

相比之下，**增强现实**是关于通过计算机扩展人类与物理世界的交互方式。使用增强现实界面，你可以向外部环境添加更多信息，并创建令人惊叹且有用的应用程序。在移动设备上，增强现实应用程序的实现高度依赖于设备上的传感器，例如视频摄像头和方向传感器。

通过传感器可以实现的交互类型的一个很好的例子是 iOS 上名为**Car Finder**的应用程序。当用户拍照时，应用程序会存储汽车的位置，然后向用户提供找到他/她停车位置所需的信息。

对于移动开发者来说，使用传感器及其返回的数据的能力越来越重要。PhoneGap 支持的传感器有限，但 PhoneGap 只是一个包装器，使得将你的表示层与原生设备代码分离变得更容易。因此，你可以在 PhoneGap 包装器周围开始编写额外的原生代码来扩展其功能。

### 注意

在 Microsoft 网站上有一个关于传感器开发的有趣资源，网址为[`research.microsoft.com/en-us/groups/sendev/`](http://research.microsoft.com/en-us/groups/sendev/)，在那里你可以找到帮助你开始使用传感器的论文和资源。

# 加速度计

PhoneGap 加速度计 API 允许你检测设备相对于设备方向的运动变化值。请注意，加速度计将值检测为相对于当前设备位置的增量运动。更重要的是，它考虑了重力效应（即*9.81 m/s*^(*2*)），因此当设备平放在桌子上朝上时，返回的值应该是*x = 0*，*y = 0*，和*z = 9.81*。

就像任何其他插件一样，在使用项目之前，你必须先安装该插件。可以使用以下命令将插件添加到项目中：

```js
$ cordova plugin add cordova-plugin-device-motion

```

一旦插件安装到项目中，就会创建一个`navigator.Accelerometer`全局对象，并在`deviceready`事件触发后可用。然而，建议在使用之前仍然检查其存在：

```js
document.addEventListener("deviceready", onDeviceReady, false);

function onDeviceReady() {
    if( typeof navigator.accelerometer === "undefined"){
        //plugin is ready now
    }
}
```

你可以使用`getCurrentAcceleration`方法或通过`watchAcceleration`方法设置监视器来检测设备加速度数据。这两种方法都在`navigator.accelerometer`对象上可用，并接受类似的参数。

`getCurrentAcceleration`方法接受一个成功和失败回调函数作为参数，并且不返回任何内容。`watchAcceleration`方法接受一个额外的参数，用于定义选项并返回当前监视器的引用。

为了持续监控加速度数据，你必须定义你想要恢复数据的频率，并将`watchAcceleration`方法返回的值存储在一个变量中：

```js
var options = {frequency: 300};
var currentAcceleration = navigator.accelerator.watchAcceleration 
                          (onSuccess, onFailure, options);
```

`onSuccess`处理程序接收一个`Acceleration`对象作为参数，访问其属性，使得能够读取每个轴上的加速度：

```js
function onSuccess(acceleration) {

    console.log('Acceleration X: ' + acceleration.x );
    console.log('Acceleration Y: ' + acceleration.y);
    console.log('Acceleration Z: ' + acceleration.z );

};
```

失败处理程序不接收任何参数，但在处理访问设备加速度计时可能出现的错误时非常有用：

```js
function onError() {

    console.log('Error accessing the accelerometer');

};
```

为了停止监控加速度计数据，只需调用定义在`accelerator`对象上的`clearWatch`方法，并传递之前用于存储`watchAcceleration`方法结果的变量引用即可：

```js
navigator.accelerometer.clearWatch(currentAcceleration);
```

此方法不接受任何额外的处理程序。

### 小贴士

PhoneGap 的所有传感器 API 都以类似的方式工作；您将始终需要使用 `getCurrentSENSOR` 和 `watchSENSOR` 方法（其中 `SENSOR` 是传感器的名称）从传感器获取数据。为了停止监视传感器，您将始终使用 `clearWatch` 方法。

## 检测晃动

使用从加速度计 API 恢复的信息，可以了解用户是否在晃动设备。

## 设备方向事件

cordova-plugin-device-motion 插件仅支持访问加速度信息。为了处理方向变化，您必须依赖于目标平台浏览器的 JavaScript API。当您想要在设备方向改变时更新用户界面时，您必须使用 CSS 媒体查询；任何其他业务逻辑都可以使用 JavaScript 处理，因为 PhoneGap 使用网页视图来渲染应用的用户界面。

使用 JavaScript，您可以设置一个监听器来处理 `orientationchange` 事件，并设置另一个监听器来处理 `deviceorientation` 事件，以处理设备方向。第一个事件在设备方向每次改变时触发；第二个事件在设备物理方向改变时触发。两个监听器都必须注册到 `window` 对象：

```js
window.addEventListener('orientationchange', EVENT_HANDLER);
window.addEventListener('deviceorientation', EVENT_HANDLER);
```

`orientationchange` 事件处理器通常用于检测屏幕方向改变后的屏幕方向。一旦方向改变，应用会接收到几个事件的通知。以下表格总结了这些事件和方向属性值：

| 设备和用户手势 | 触发的事件 | 方向 |
| --- | --- | --- |
| iPad 横屏 | `resize` `orientationchange` | 090 |
| iPad 竖屏 | `resize` `orientationchange` | 900 |
| iPhone 横屏 | `resize` `orientationchange` | 090 |
| iPhone 竖屏 | `resize` `orientationchange` | 900 |
| 安卓手机横屏 | `orientationchange` `resize` | 9090 |
| 安卓手机竖屏 | `orientationchange` `resize` | 00 |

`deviceorientation` 事件非常强大。它将包含以下信息的 `DeviceOrientationEvent` 事件实例返回给处理器：

+   `alpha`：这返回设备绕 *z* 轴的旋转

+   `beta`：这返回设备绕 *x* 轴的旋转

+   `gamma`：这返回设备绕 *y* 轴的旋转

### 提示

为了提高您应用的性能，考虑使用 `event-handler` 函数仅保存当前从传感器数据到变量的值。然后，将您的计算或 DOM 操作移动到在固定时间执行的新函数中。

## 使用 JavaScript 处理方向

是时候将您所学的设备方向事件知识付诸实践了。让我们来处理一个非常基础的示例，该示例能够根据设备的物理方向在 `div` 元素中显示屏幕方向。

# 行动时间 - 使用 JavaScript 处理设备方向

执行以下步骤：

1.  打开命令行工具，创建一个名为 `orientationevents` 的新 PhoneGap 项目，并添加你想要针对此示例的目标平台。

1.  将插件安装到你的项目中：

    ```js
    $ phonegap plugin add cordova-plugin-device-motion

    ```

1.  前往 `www` 文件夹，打开 `index.html` 文件，并在应用下方的 `#deviceready` 主要 `div` 内添加带有 `#orientation` ID 的 `div`：

    ```js
    <div class="app">
        <h1>Apache Cordova</h1>
        <div id="deviceready">
            ......
        </div>
        <div id="orientation">
        </div>
    </div>
    ```

1.  前往 `css` 文件夹，并在 `index.css` 文件内定义两个新规则，为 `div` 及其内容添加边框和更大的字体大小。你甚至可以直接将 CSS 样式添加到 HTML 页面的 `head` 部分：

    ```js
    #orientation{

        width: 230px;
        border: 1px solid rgb(10, 1, 1);

    }

    #orientation p{
        font-size: 36px;
        font-weight: bold;
        text-align: center;

    }
    ```

1.  前往 `js` 文件夹，打开 `index.js` 文件，并定义一个新函数，以便轻松检测设备是否可以处理 `orientationchange` 和 `deviceorientation` 事件。或者，你甚至可以将脚本直接嵌入到你的 HTML 页面中：

    ```js
    orientationSupported: function(){

        try {
            return 'DeviceOrientationEvent' in window &&
            window['DeviceOrientationEvent'] !== null;
        } catch (e) {
            return false;
        }

    }
    ```

1.  在 `deviceready` 函数中，如果设备支持 `orientationchange` 和 `deviceorientation` 事件，则添加两个监听器：

    ```js
    if(orientationSupported){

        window.addEventListener('orientationchange', orientationChanged);
        window.addEventListener('deviceorientation', updateOrientation);

    }else{

         alert('Orientation not supported!');

    }
    ```

1.  定义 `orientationChanged` 事件处理程序，并使用它来在屏幕上打印当前设备方向：

    ```js
    orientationChanged: function(){

        var element = document.querySelector('#orientation');
        element.innerHTML = '<p>' + window.orientation + '</p>';

    }
    ```

1.  定义 `deviceorientation` 事件的处理程序，并使用设备传感器的信息来改变 `div` 方向的 3D 变换：

    ```js
    updateOrientation: function(event){

        var alpha = event.alpha,
        beta = event.beta,
        gamma = event.gamma;

        var element = document.querySelector('#orientation');
        var rotation = 'rotateZ(' + alpha + 'deg) rotate(' + beta + 'deg) rotateY(' + gamma + 'deg)';
    // For brevity the browser prefixes have been removed
        element.style.transform = rotation;

    }
    ```

1.  再次打开命令行工具，定位主项目文件夹，然后编译应用并在你之前添加的每个平台上进行测试。

下面是这个示例的完整代码：

```js
<!DOCTYPE html>
<html>
    <head>
        <!--Other section removed for sake of simplicity -->
        <title>Media Capture Example</title>
        <style>
          #orientation{
            width: 230px;
            border: 1px solid rgb(10, 1, 1);
          }

          #orientation p{
            font-size: 36px;
            font-weight: bold;
            text-align: center;
          }
        </style>
    </head>
    <body>
      <div class="app">
        <h1>Apache Cordova</h1>
        <div id="deviceready"></div>
        <div id="orientation"></div>
      </div>

      <script type="text/javascript" src="img/cordova.js"></script>
      <script type="text/javascript">

        document.addEventListener("deviceready", onDeviceReady, false);

        function onDeviceReady() {
          if(orientationSupported){
            window.addEventListener('orientationchange', orientationChanged);
            window.addEventListener('deviceorientation', updateOrientation);
          }else{
            alert('Orientation not supported!');
          }
        }

        orientationSupported: function(){
          try {
            return 'DeviceOrientationEvent' in window &&
            window['DeviceOrientationEvent'] !== null;
          } catch (e) {
            return false;
          }
        }
        orientationChanged: function(){
          var element = document.querySelector('#orientation');
          element.innerHTML = '<p>' + window.orientation + '</p>';
        }

        updateOrientation: function(event){

          var alpha = event.alpha,
          var beta = event.beta,
          var gamma = event.gamma;

          var element = document.querySelector('#orientation');
          var rotation = 'rotateZ(' + alpha + 'deg) rotate(' + beta + 'deg) rotateY(' + gamma + 'deg)';
          // For brevity the browser prefixes have been removed
          element.style.transform = rotation;
        }

      </script>
    </body>
</html>
```

## *发生了什么？*

你使用 JavaScript 处理了方向事件，并通过 PhoneGap 将结果部署到设备上。该应用能够实时获取设备屏幕方向和当前位置。

# Compass

PhoneGap Compass API 允许你获取设备指向的方向。罗盘是一个传感器，可以检测设备指向的方向或航向，并使用 `0` 到 `359.99` 的值返回设备的航向（以度为单位）。Compass API 与加速度计 API 的工作方式类似；实际上，你可以读取设备的当前航向，或者定义一个监视器以连续读取航向值。

Compass API 可在 `navigator` 对象的 `compass` 属性上找到，并公开以下函数：

+   `compass.getCurrentHeading`: 这通过处理程序读取当前的罗盘方向

+   `compass.watchHeading`: 这通过处理程序在特定时间间隔读取罗盘方向，并返回对其的引用

+   `compass.clearWatch`: 这将停止之前定义的时间间隔读取处理程序

`getCurrentHeading` 和 `watchHeading` 函数接受非常相似的参数；唯一的区别是 `watchHeading` 函数的最后一个参数，允许你配置它。为了读取设备的当前航向，只需执行 `getCurrentHeading` 函数，指定成功和错误处理程序即可：

```js
navigator.compass.getCurrentHeading(onSuccess, onError);
```

`onSuccess`处理器接收一个`CompassHeading`对象作为参数，具有以下属性：

+   `magneticHeading`：这是从`0`到`359.99`度的航向

+   `trueHeading`：这是相对于地理北极的航向，以度为单位

+   `headingAccuracy`：这是报告航向与真实航向之间的偏差，以度为单位

+   `timestamp`：这是确定此航向的时间

错误处理器接收一个`CompassError`对象作为参数；`CompassError`对象有一个名为`code`的属性，它返回两个可能的值，例如`CompassError.COMPASS_INTERNAL_ERR`或`CompassError.COMPASS_NOT_SUPPORTED`：

```js
function onError (error) {
    switch(true){

        case error.code == CompassError.COMPASS_INTERNAL_ERR:
        navigator.notification.alert('Compass Error!', null, 'Info', 'OK');
        break;

        case error.code == CompassError.COMPASS_NOT_SUPPORTED:
        navigator.notification.alert('Compass Unavailable!', null, 'Info', 'OK');
        break;

        default:
        navigator.notification.alert('Generic Error!', null,'Info', 'OK');

    }
}
```

`watchHeading`函数与`getCurrentHeading`函数类似。唯一的区别是它接受一个额外的`CompassOption`对象，允许您设置以毫秒（即频率）为单位检索指南针航向的频率以及触发成功处理器的度数变化（即过滤器）：

```js
var options = {frequency: 300};
var currentHeading = navigator.compass.watchHeading(
                     onSuccess, onError, options);
```

为了停止监视航向值的变化，只需使用`clearWatch`函数和当前航向监视器的引用即可：

```js
clearWatch(currentHeading);
```

### 注意

`CompassHeading`对象的`trueHeading`属性在 Android 上不受支持。它返回与`magneticHeading`相同的值，并且`headingAccuracy`值始终为`0`，因为在`magneticHeading`和`trueHeading`之间没有差异。

在 iOS 上，只有当使用`watchLocation`函数运行位置服务时，才会返回`trueHeading`属性。

## 创建指南针

读取设备的当前航向是开发者在多个用例中的常见任务，例如交通应用、增强现实应用或任何包含方向感的应用。让我们看看如何使用 PhoneGap 创建一个完整的指南针：

### 注意

用于渲染指南针的图像可在 Creative Commons 许可证下在[`commons.wikimedia.org/wiki/File:Compass.svg`](http://commons.wikimedia.org/wiki/File:Compass.svg)找到。在开始此示例之前，请下载该图像并创建三个单独的 PNG 文件，分别用于背景、表盘和箭头。由于这是一个 SVG 矢量文件，您可以处理图像的每一层，并按需编辑它。要编辑图像，您可以使用任何矢量编辑应用程序，例如 Adobe Illustrator 或免费应用程序 Inkscape。

# 动手时间 - 使用指南针 API

执行以下步骤：

1.  打开命令行工具，创建一个名为`compass`的新 PhoneGap 项目，并添加您想要针对此示例的目标平台。

1.  使用以下命令添加 Compass API 插件：

    ```js
    $ cordova plugin add cordova-plugin-device-orientation

    ```

1.  前往`www`文件夹，打开`index.html`文件。三个`div`标签用于处理指南针箭头和背景：

    ```js
    <section id="compass">
        <div id="compassbg"></div>
        <div id="north"></div>
        <div id="arrow"></div>
    </section>
    ```

1.  前往`css`文件夹，打开`index.css`文件，并定义指南针每个元素所需的单独背景规则：

    ```js
    #compassbg {
        background-image: url(../img/Compass.png);
    }
    #north {
        background-image: url(../img/arrow_direction.png);
    }

    #arrow {
        background-image: url(../img/arrow_beta.png);
    }

    #compass, #arrow, #north, #compassbg  {
        background-repeat: no-repeat;
        background-size: cover;
        position: fixed;
        width: 286px;
        height: 286px;

    }
    ```

1.  前往`js`文件夹，打开`index.js`文件，添加一个新变量以存储你将定义的用于监控设备航向的监视器的引用。或者，你也可以直接在页面上嵌入脚本：

    ```js
    var currentHeading = null;
    ```

1.  定位到`deviceready`函数，并在其中添加每 150 毫秒检查设备航向所需的代码片段：

    ```js
    var options = {frequency: 150};
    currentHeading = navigator.compass.watchHeading (onCompassSuccess,onCompassError, options);
    ```

1.  创建一个名为`onCompassSuccess`的新函数，并在其主体内部开始读取存储在接收到的参数中的航向数据；使用它来旋转指南针元素：

    ```js
    function onCompassSuccess(heading){
      var magneticHeading = heading.magneticHeading;
      var trueHeading = heading.trueHeading;

      var compass = document.querySelector('#compassbg');
      var north = document.querySelector('#north');

      var compassRotation = 'rotate(' + magneticHeading + 'deg)';
      var northRotation = 'rotate(' + trueHeading + 'deg)';
      var compassSytle = compass.style;
      var northStyle = north.style;

      compassStyle.transform = compassRotation;
      northStyle.transform = northRotation;
    }
    ```

1.  定义一个函数来捕获可能出现的失败：

    ```js
    function onCompassError(error){
      alert("Error with Compass");
    }
    ```

1.  再次打开命令行工具，定位到主项目文件夹，编译应用程序，并在你之前添加的每个平台上进行测试。

对于我们刚才看到的例子，你可以在这里找到完整的代码：

```js
<!DOCTYPE html>
<html>
    <head>
        <!--Other section removed for sake of simplicity -->
        <title>Media Capture Example</title>
        <style>
          #compassbg {
            background-image: url(../img/Compass.png);
          }
          #north {
            background-image: url(../img/arrow_direction.png);
          }

          #arrow {
            background-image: url(../img/arrow_beta.png);
          }

          #compass, #arrow, #north, #compassbg  {
            background-repeat: no-repeat;
            background-size: cover;
            position: fixed;
            width: 286px;
            height: 286px;
          }

        </style>

    </head>
    <body>

      <section id="compass">
        <div id="compassbg"></div>
        <div id="north"></div>
        <div id="arrow"></div>
      </section>

      <script type="text/javascript" src="img/cordova.js"></script>

      <script type="text/javascript">
        var currentHeading = null;

        document.addEventListener("deviceready", onDeviceReady, false);

        function onDeviceReady() {
          var options = {frequency: 150};

          currentHeading = navigator.compass.watchHeadingonCompassSuccess, onCompassError, options);
        }

        function onCompassSuccess(heading){
          var magneticHeading = heading.magneticHeading;
          var trueHeading = heading.trueHeading;

          var compass = document.querySelector('#compassbg');
          var north = document.querySelector('#north');

          var compassRotation = 'rotate(' + magneticHeading + 'deg)';
          var northRotation = 'rotate(' + trueHeading + 'deg)';
          var compassSytle = compass.style;
          var northStyle = north.style;

          compassStyle.transform = compassRotation;
          northStyle.transform = northRotation;
        }

        function onCompassError(error){
          alert("Error with Compass");
        }
      </script>
    </body>
</html>
```

## *发生了什么？*

你使用 PhoneGap API 实现了一个真正的、跨平台的指南针。在这个过程中，你学习了如何使用移动设备传感器的一个相当复杂的功能。

# 地理定位简介

术语**地理定位**用于指代识别物体在现实世界中的地理位置的过程。能够检测用户位置的设备越来越常见，我们现在已经习惯了根据我们的位置获取内容（**地理定位**）。

使用**全球定位系统**（**GPS**）——一个基于空间的卫星导航系统，它在全球范围内持续提供位置和时间信息——你现在可以获取设备的准确位置。在 20 世纪 70 年代初，美国军方创建了 Navstar，这是一个防御导航卫星系统。Navstar 是今天数十亿设备使用的 GPS 基础设施的基础。截至 2014 年 10 月，已有 68 颗 GPS 卫星成功送入地球轨道（有关过去和计划发射的详细报告，请参阅[`en.wikipedia.org/wiki/List_of_GPS_satellite_launches`](http://en.wikipedia.org/wiki/List_of_GPS_satellite_launches)）。

设备的位置通过一个点来表示。这个点由两个组成部分组成：纬度和经度。现代设备有许多确定位置信息的方法；以下是一些方法：

+   GPS

+   IP 地址

+   GSM/CDMA 小区 ID

+   Wi-Fi 和蓝牙 MAC 地址

每种方法都提供相同的信息；变化的是设备位置的准确性。GPS 卫星持续传输可解析的信息；例如，GPS 阵列的一般健康状况，所有卫星的大致轨道位置，传输卫星的精确轨道或路径信息，以及传输时间。接收器通过计算阵列中任何可见卫星发送的信号的传输时间来计算自己的位置。

### 注意

测量从一点到一组卫星的距离以确定位置的过程称为 **三角测量法**。距离是通过使用光速作为常数以及信号离开卫星的时间来确定的。

移动开发中出现的趋势是基于 GPS 的“人脉发现”应用，如 Highlight、Sonar、Banjo 和 Foursquare。每个应用都有不同的功能，并且为不同的目的而构建，但它们都共享同一个杀手级功能：使用位置作为元数据的一部分，以便根据用户的需求过滤信息。

# PhoneGap 定位 API

定位 API 不是 HTML5 规范的一部分，但它与移动开发紧密集成。PhoneGap 定位 API 和 W3C 定位 API 相互映射；两者定义了相同的方法和相对参数。有几款设备已经实现了 W3C 定位 API；对于这些设备，你可以使用原生支持而不是 PhoneGap API。

### 注意

根据 HTML 规范，用户必须明确允许网站或应用使用设备的当前位置。

定位 API 通过 `navigator` 对象的 `geolocation` 子对象公开，并包含以下三个方法：

+   `getCurrentPosition()`: 这将返回设备位置

+   `watchPosition()`: 这将监视设备位置的变化

+   `clearWatch()`: 这将停止设备位置变化的监视器

`watchPosition()` 和 `clearWatch()` 方法的工作方式与 `setInterval()` 和 `clearInterval()` 方法相同；实际上，第一个方法返回一个标识符，该标识符传递给第二个方法。`getCurrentPosition()` 和 `watchPosition()` 方法相互映射，并接受相同的参数：一个成功回调函数和一个失败回调函数，以及一个可选的 `configuration` 对象。`configuration` 对象用于指定设备位置缓存的最高年龄，以便在方法失败后设置超时，并指定应用程序是否只需要精确结果：

```js
var options = {maximumAge: 3000, timeout: 5000, 
               enableHighAccuracy: true };
navigator.geolocation.watchPosition(onSuccess, onFailure, options);
```

### 小贴士

只有第一个参数是必需的，但建议始终处理失败用例。

成功处理函数接收一个 `Position` 对象作为参数。访问其属性，你可以读取设备的坐标和存储坐标的对象的创建时间戳：

```js
function onSuccess(position) {

    console.log('Coordinates: ' + position.coords);
    console.log('Timestamp: ' + position.timestamp);

}
```

`Position` 对象的 `coords` 属性包含一个 `Coordinates` 对象；到目前为止，该对象最重要的属性是 `longitude`（经度）和 `latitude`（纬度）。使用这些属性，你可以开始将定位信息作为相关元数据集成到你的应用中。

失败处理函数接收一个 `PositionError` 对象作为参数。使用该对象的 `code` 和 `message` 属性，你可以优雅地处理每个可能出现的错误：

```js
function onError(error) {
    console.log('message: ' + error.message);
    console.log ('code: ' + error.code);

}
```

`message`属性返回错误的详细描述，而`code`属性返回一个整数；可能值通过以下伪常量表示：

+   `PositionError.PERMISSION_DENIED`：这表示用户拒绝了应用使用设备的当前位置

+   `PositionError.POSITION_UNAVAILABLE`：这表示无法确定设备的位置

    ### 注意

    如果你想在返回`POSITION_UNAVAILABLE`错误时恢复最后可用的位置，你必须编写一个使用平台特定 API 的自定义插件。Android 和 iOS 都有这个功能。你可以在[`stackoverflow.com/questions/10897081/retrieving-last-known-geolocation-phonegap`](http://stackoverflow.com/questions/10897081/retrieving-last-known-geolocation-phonegap)找到详细的示例。

+   `PositionError.TIMEOUT`：这表示在实现成功获取新的`Position`对象之前，指定的超时时间已过。

### 注意

JavaScript 不支持 Java 和其他面向对象编程语言中的常量。术语“伪常量”指的是在 JavaScript 应用中不应更改的值。

使用设备位置信息执行的最常见任务之一是在地图上显示设备位置。你可以通过在应用中集成 Google Maps 快速完成此任务；唯一的要求是有效的 API 密钥。要获取密钥，请按照以下步骤操作：

1.  访问 API 控制台[`code.google.com/apis/console`](https://code.google.com/apis/console)，并使用您的 Google 账户登录。

1.  在左侧菜单中选择**APIs & auth**。

1.  选择**Google Maps JavaScript API**并激活该服务。

# 行动时间 - 使用 Google Maps 显示设备位置

准备向 PhoneGap 默认应用模板添加地图渲染器。执行以下步骤：

1.  打开命令行工具并创建一个名为`MapSample`的新 PhoneGap 项目：

    ```js
    $ cordova create MapSample

    ```

1.  将工作目录更改为新创建的项目：

    ```js
    $ cd MapSample

    ```

1.  将所需的平台添加到项目中。例如，我们将向此项目添加 Android：

    ```js
    $ cordova platform add android

    ```

1.  使用以下命令添加 Geolocation API 插件：

    ```js
    $ cordova plugin add cordova-plugin-geolocation

    ```

1.  前往`www`文件夹，打开`index.html`文件，删除所有现有内容，并在`body`标签内添加一个`div`元素，其`id`值为`#map`：

    ```js
    <div id='map'></div>
    ```

1.  包含将在运行时添加到应用的`cordova.js`文件：

    ```js
    <script type="text/javascript" src="img/cordova.js"></script>
    ```

1.  添加一个新的`script`标签以包含 Google Maps JavaScript 库。将`YOUR_API_KEY`值替换为您的实际 Google Maps API 密钥：

    ```js
    <script type="text/javascript"
       src="img/strong>&sensor=true">
    </script>
    ```

1.  创建一个新的 CSS 样式以给`div`元素及其内容一个适当的大小：

    ```js
    #map{

        width: 280px;
        height: 230px;
        display: block;
        margin: 5px auto;
        position: relative;

    }
    ```

1.  在 JavaScript 部分，定义一个名为`initMap`的新函数：

    ```js
    function initMap(lat, long){

        // The code needed to show the map and the
        // device position will be added here

       }
    ```

1.  在函数体中，定义一个`options`对象以指定地图的渲染方式：

    ```js
    var options = {

        zoom: 8,
        center: new google.maps.LatLng(lat, long),
        mapTypeId: google.maps.MapTypeId.ROADMAP

    };
    ```

1.  在`initMap`函数体中添加初始化地图渲染的代码以及显示代表当前设备位置的标记的代码：

    ```js
    var map = new google.maps.Map(document.getElementById('map'), options);

    var markerPoint = new google.maps.LatLng(lat, long);

    var marker = new google.maps.Marker({

        position: markerPoint,
        map: map,
        title: 'Device\'s Location'

    });
    ```

1.  定义一个函数作为成功处理程序使用，并在其主体中调用之前定义的`initMap`函数：

    ```js
    function onSuccess(position){

        var coords = position.coords;
        initMap(coords.latitude, coords.longitude);

    }
    ```

1.  定义另一个函数，以便有一个能够通知用户出错的失败处理程序：

    ```js
    function onFailure(error){

        alert(error.message);

    }
    ```

1.  进入`deviceready`函数，并将调用 Geolocation API 以恢复设备位置的调用作为最后一条语句添加：

    ```js
    navigator.geolocation.getCurrentPosition(app.onSuccess, app.onFailure, {timeout: 5000, enableAccuracy: false});
    ```

1.  打开命令行工具，构建应用，然后在测试设备上运行它：

    ```js
    $ cordova build
    $ cordova run android

    ```

这里是项目的完整代码，供你参考：

```js
<!DOCTYPE html>
<html>
<head>
<title>GeoLocation Example</title>

<style>
  #map{
    width: 280px;
    height: 230px;
    display: block;
    margin: 5px auto;
    position: relative;
  }
</style>

</head>
<body>

<div id='map'></div>

<script type="text/javascript" src="img/cordova.js"></script>

<script type="text/javascript" src="img/js?key=YOUR_API_KEY&sensor=true"></script>

<script type="text/javascript">
  var currentHeading = null;

  document.addEventListener("deviceready", onDeviceReady, false);

  function onDeviceReady() {
    navigator.geolocation.getCurrentPosition(onSuccess, onFailure, {timeout: 5000, enableAccuracy: false});
  }

  function onSuccess(position){
    var coords = position.coords;
    initMap(coords.latitude, coords.longitude);
  }

  function initMap(lat, long){
    var options = {
      zoom: 8,
      center: new google.maps.LatLng(lat, long),
      mapTypeId: google.maps.MapTypeId.ROADMAP
    };

    var map = new google.maps.Map(document.getElementById('map'), options);

    var markerPoint = new google.maps.LatLng(lat, long);

    var marker = new google.maps.Marker({
      position: markerPoint,
      map: map,
      title: 'Device\'s Location'
    });
  }

  function onFailure(error){
    alert(error.message);
  }

</script>
</body>
</html>
```

## *发生了什么？*

你在应用中集成了 Google Maps。这个地图是大多数用户都熟悉的交互式地图——最常见的手势已经生效，Google 街景控制也已经启用。

### 提示

要在 iOS 上成功加载 Google Maps API，必须将`googleapis.com`和`gstatic.com`域名列入白名单。将项目的`.plist`文件作为源代码打开（右键单击文件，然后选择**打开方式** | **源代码**)，并添加以下域名数组：

```js
<key>ExternalHosts</key>
<array>
    <string>*.googleapis.com</string>
    <string>*.gstatic.com</string>
</array>
```

## 其他地理位置数据

在上一个示例中，你只使用了接收到的`position`对象中的`latitude`和`longitude`属性。还有其他属性可以作为`Coordinates`对象的属性访问：

+   `海拔`: 这表示设备相对于海平面的高度，单位为米

+   `accuracy`: 这表示纬度和经度的精度级别，单位为米；它可以用来在映射设备位置时显示精度半径

+   `altitudeAccuracy`: 这表示海拔的精度，单位为米

+   `heading`: 这表示设备相对于真北顺时针方向的方位角

+   `速度`: 这表示设备当前的地速，单位为每秒米

`latitude`和`longitude`属性是这些属性中支持最好的，也是在与远程 API 通信时最有用的属性。其他属性主要在你开发的应用中地理位置是其标准功能的核心组件时有用，例如使用这些数据创建与地理位置数据相关的信息流的应用。`accuracy`属性是这些附加功能中最重要的，因为作为应用程序开发者，你通常不知道哪个特定的传感器提供了位置信息，你可以使用`accuracy`属性作为查询外部服务时的范围。

有几个 API 允许你发现与地点相关的有趣数据；在这些 API 中，最有趣的是 Google Places API 和 Foursquare API。

### 注意

Google 地点（Google Places）和 Foursquare 在线文档组织得非常好，如果你想要深入了解这些主题，这是一个很好的起点。你可以通过[`developers.google.com/maps/documentation/javascript/places`](https://developers.google.com/maps/documentation/javascript/places)访问 Google 地点文档，以及[`developer.foursquare.com/`](https://developer.foursquare.com/)访问 Foursquare。

# 摘要

在本章中，你学习了如何使用设备传感器来增强你应用程序的功能。你还学习了如何从设备获取地理位置信息，以及如何在应用程序中集成外部地理位置服务。此外，你继续了解 PhoneGap API，这些 API 允许你创建强大的原生应用程序。

在下一章中，你将开始学习使用 PhoneGap 的一些高级概念。
