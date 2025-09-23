# 第八章. 高级 PhoneGap

*如果您希望您的应用程序能够触及多个国家和文化的用户，请输入全球化与本地化。您的应用程序应能够支持多种语言/区域，这可以通过使用全球化 API 和相关本地化库来实现。如果您希望应用程序更先进且用户友好，请考虑支持多种手势。*

在本章中，您将：

+   学习如何使用全球化 API 来支持移动应用程序用户的区域设置

+   学习如何根据用户的喜好以多种语言显示内容来本地化应用程序

+   获取提供手势支持的库的概述

+   学习如何实现多个触摸手势以增强用户体验

+   学习如何移除点击事件中引入的 300 毫秒延迟

# 使用全球化 API

在计算机科学中，全球化、国际化、本地化是适应不同语言、区域差异和目标市场技术要求的手段。让我们详细看看这些。

术语**本地化**指的是在您的应用程序能够在不同语言和当地文化规范下部署之前所需的所有活动。在开始本地化应用程序之前，您必须通过移除任何语言和文化依赖项并设计代码以便它能够适应各种语言而无需进行工程更改来**国际化**您的代码。然后，您可以本地化应用程序，翻译面向客户端的内容和标签，并对其进行其他适应，以便在特定区域中良好运行。

术语**区域**指的是用于本地化的设置或首选项的集合。区域通常描述为语言和国家对的组合，如 en-US、de-AT、it-IT 等。

术语**全球化**代表国际化与本地化的结合。

有些看起来奇怪的缩写中使用了数字来表示用于指代国际化、本地化和全球化的第一个和最后一个字母之间的字母数：

+   `i18n`: 这代表国际化

+   `l10n`: 这代表本地化

+   `g11n`: 这代表全球化

### 注意

软件国际化是一个很大的话题，因为它涵盖了复数、日期、特殊字符等。讨论所有这些内容超出了本书的范围。如果您想了解更多关于国际化的知识，请查看 GNU 项目 gettext ([`www.gnu.org/software/gettext/manual/`](http://www.gnu.org/software/gettext/manual/))、globalize 项目 ([`github.com/jquery/globalize`](https://github.com/jquery/globalize)) 或 Jed 项目 ([`slexaxton.github.io/Jed/`](http://slexaxton.github.io/Jed/))。

PhoneGap 通过`globalization`对象提供的全球化 API 提供了强大的本地化支持。您可以将插件安装到项目中，如下所示：

```js
$ cordova plugin add cordova-plugin-globalization

```

`globalization`对象是`navigator`对象的子对象，因此具有全局作用域。为了访问`globalization`对象，只需输入以下代码行即可：

```js
var globalization = navigator.globalization;
```

`globalization`对象暴露了几个具有类似签名的异步方法。实际上，通常大多数方法接受一个参数、一个成功处理程序和一个失败处理程序，以及可选的`options`对象：

```js
globalization.methodName(argument, onSuccess, onError, options);
```

并非所有方法都接受参数和`options`对象；其中一些方法仅接受成功和失败处理程序。失败处理程序接收一个`GlobalizationError`对象作为参数。该对象上定义了两个属性：`message`和`code`。第一个属性包含描述错误详细信息的字符串；第二个属性包含一个整数，等于在`GlobalizationError`对象中定义的以下伪常量之一：

+   `GlobalizationError.UNKNOWN_ERROR`（返回值`0`）：这意味着发生了通用错误

+   `GlobalizationError.FORMATTING_ERROR`（返回值`1`）：这意味着在格式化操作期间发生错误

+   `GlobalizationError.PARSING_ERROR`（返回值`2`）：这意味着在解析操作期间发生错误

+   `GlobalizationError.PATTERN_ERROR`（返回值`3`）：这意味着在恢复货币、日期或数字模式时发生错误

Globalization API 暴露了在`navigator.globalization`对象中定义的以下方法：

+   `getPreferredLanguage`：此方法返回设备当前语言的字符串标识符；字符串存储在成功处理程序接收的对象的`value`属性中（例如，`{value: ‘English'}`）

+   `getLocaleName`：此方法根据设备的当前语言返回区域标识符；字符串存储在成功处理程序接收的对象的`value`属性中（例如，`{value: ‘en'}`）

+   `dateToString`：此方法根据客户端的区域设置和时区将日期格式化为字符串；该方法接受一个`Date`对象作为第一个参数，以及一个可选的`options`对象作为最后一个参数：

    ```js
    var globalization = navigator.globalization;

    var today = new Date();
    globalization.dateToString(today, onSuccess, onError);
    ```

    返回的结果存储在成功处理程序接收的对象的`value`属性中（例如，`{value: ‘06/14/2013 12:49 PM'}`）。

+   `stringToDate`：此方法解析格式为字符串的日期，并根据设备的偏好和日历返回相应的`Date`对象作为成功处理程序中的参数。

+   `getDatePattern`：此方法返回一个对象，作为成功处理程序中的参数接收，包含以下内容：

    +   一个用于根据设备偏好格式化和解析日期的模式字符串

    +   设备的时区

    +   设备时区与通用时间之间的秒数差以及设备非夏令时时区与通用时间之间的秒数差

    +   客户端的夏令时时区（例如，`{pattern: ‘dd/MM/yyyy HH:mm‘, timezone: ‘CEST‘, utc_offset: 3600, dst_offset: 3600}`）

    此方法通过可选的`options`对象接受，可以通过它指定格式长度（即`short`、`medium`、`long`或`full`）和要返回的数据（即`date`、`time`或`date and time`）

+   `getDateNames`: 该函数返回一个包含月份或星期名称的数组，具体取决于设备的设置；数组存储在成功处理程序接收到的对象中的`value`属性中（即，`{value: Array[12]}`）

+   `isDayLightSavingsTime`: 该函数返回一个布尔值，表示是否对作为第一个参数传入的`Date`对象使用设备的时区和日历实施夏令时；值存储在成功处理程序接收到的对象中的`dst`属性中（例如，`{dst: true}`）

+   `getFirstDayOfWeek`: 该函数返回一周的第一天作为数字，取决于设备的用户偏好和日历，假设一周的天数从`1`开始编号（即星期日）；字符串存储在成功处理程序接收到的对象中的`value`属性中（例如，`{value: 1}`）

+   `numberToString`: 该函数将作为第一个参数传入的数字格式化为字符串，格式根据客户端的区域设置和偏好；数字存储在成功处理程序接收到的对象中的`value`属性中（例如，`{value: ‘12,456,246'}`）

+   `stringToNumber`: 该函数将作为第一个参数传入的字符串格式化为数字，格式根据客户端的区域设置和偏好；数字存储在成功处理程序接收到的对象中的`value`属性中（例如，`{value: 1250.04}`）

+   `getNumberPattern`: 该函数返回一个对象，作为成功处理程序中的参数，包含以下内容：

    +   用于格式化和解析数字的格式字符串，根据设备的偏好

    +   解析和格式化数字时使用的分数位数，解析和格式化时使用的舍入增量等（例如，`{decimal: ‘.', fraction: 0, grouping: ‘,’ negative: ‘-‘, pattern: ‘#,##0.###‘, positive: ‘‘, rounding: 0, symbol: ‘.’}`）

+   `getCurrencyPattern`：此函数返回一个对象，该对象作为成功处理程序中的参数接收，包含一个用于根据传递给第一个参数的货币代码格式化和解析货币的模式字符串，设备的偏好设置，解析和格式化数字时使用的分数位数，解析和格式化时使用的舍入增量，用于模式的 ISO 4217 货币代码等（例如，`{code: ‘EUR', decimal: ‘.', fraction: 2, grouping: ‘,’ pattern: ‘$#,##0.00;(¤#,##0.00)’, rounding: 0}`）。

    ### 小贴士

    `numberToString`和`stringToNumber`方法都接受一个可选的`options`对象；通过此对象的`type`属性，您可以指定数字的格式（例如，`decimal`、`percent`或`currency`）。

通过`globalization`对象的方法提供的数据的组合，可以处理非常复杂的情况，并为最终用户提供高度本地化的应用。全球化 API 是一个非常强大的工具，它允许您与其他 JavaScript 库协同工作。

# 本地化您的应用

从开发角度来看，常见的做法是将文本放置在资源字符串中，这些字符串在执行时根据用户设置加载。您可以使用几种技术来全球化您的应用，例如将翻译存储在**可移植对象**（**PO**）文件中，创建包含所有内容的 JSON 对象，或者在应用启动时动态加载本地化文件。目标是部署一个能够在运行时选择相关语言资源文件，并处理文化感知的数字和日期解析与格式化、复数、货币、特殊字符、验证等的应用。

在以下示例中，我们将学习如何使用简单的 JavaScript 库在应用中加载不同的语言字符串。

# 动作时间 - 渲染本地化消息

根据设备的语言设置，按照以下步骤在您的应用中渲染不同的消息：

1.  打开命令行工具并创建一个名为`Globalization`的新 PhoneGap 项目：

    ```js
    $ cordova create Globalization

    ```

1.  使用以下命令添加全球化 API 插件：

    ```js
    $ cordova plugin add cordova-plugin-globalization

    ```

1.  使用命令行工具，添加您想用于此测试的平台（Android、Blackberry、iOS 或 Windows Phone 8）：

    ```js
    $ cordova platforms add android

    ```

1.  下载并保存位于[`github.com/marcelklehr/html10n.js`](https://github.com/marcelklehr/html10n.js)的`l10n.js`文件到`www/js`文件夹。

1.  前往`www`文件夹，创建一个名为`langs.json`的 JSON 文件以存储所有必需的语言字符串，如下所示。JSON 文件格式是存储数据如 XML 的简单方式。该文件将为每种语言重复相同的字面量：

    ```js
    {
      “en”: {
        “welcome”: “Welcome”,
        “english”: “English”,
        “french”: “French”,
        “alert”: “It's me!”
      },
      “fr”: {
        “welcome”: “Bienvenu”,
        “english”: “Anglais”,
        “french”: “Français”,
        “alert”: “C'est moi!”
      }
    }
    ```

1.  在`www`目录中，编辑`index.html`文件，并在`head`部分添加以下代码行以加载用于处理多种语言的 JavaScript 文件：

    ```js
    <script type=”text/javascript” src=”js/l10n.js”></script>
    ```

1.  按照以下示例加载`langs.json`文件：

    ```js
    <link rel=”localizations” href=”langs.json” type=“application/l10n+json”/>
    ```

1.  为`body`标签定义`onload`函数，以正确语言值初始化加载文档：

    ```js
    <body onload=”onLoad();”>
    ```

1.  创建一些 HTML 元素来尝试使用多种语言。在此示例中，我们将创建两个按钮、一个警告框和一个标题标签。当更改语言时，所有字符串值也会相应更改：

    ```js
    <h1 data-l10n-id=”welcome”>Welcome</h1>
    <button data-l10n-id=”french” onclick=”changeToFrench()”>French</button>
    <button data-l10n-id=”english” onclick=”changeToEnglish()”>English</button>
    <button data-l10n-id=”alert” onclick=”showAlert()”>Test Alert</button>
    ```

1.  创建一个用于初始化语言设置的函数。使用`localize`方法加载提供语言的应用程序。最初，我们使用`index`方法用英语语言加载页面：

    ```js
    function onLoad() {
      html10n.index();
      html10n.localize(‘en');
    }
    ```

1.  创建两个函数，当用户点击按钮时加载英语和法语语言：

    ```js
    function changeToFrench() {
      html10n.localize(‘fr');
    }

    function changeToEnglish() {
      html10n.localize(‘en');
    }
    ```

1.  定义一个函数，当点击**测试警告**按钮时，显示包含翻译字符串的警告窗口：

    ```js
    function showAlert() {
      var message = html10n.get(‘alert');
      alert(message);
    }
    ```

1.  除了这些，您还可以为其他 PhoneGap/Cordova 相关活动定义`deviceready`事件函数。在此示例中，它被留空：

    ```js
    document.addEventListener(“deviceready”, onDeviceReady, false);

    function onDeviceReady() {
      // Other Stuffs
    }
    ```

1.  打开命令行工具，进入`Globalization`文件夹，并在真实设备或模拟器上构建和运行应用程序：

    ```js
    $ cordova build
    $ cordova run

    ```

此处提供了示例的完整源代码供您参考：

```js
<!DOCTYPE html>
<html>
<head>
  <script type=”text/javascript” src=”js/l10n.js”></script>
  <link rel=”localizations” href=”langs.json” type=”application/l10n+json”/>
</head>
<body onload=”onLoad();”>
  <h1 data-l10n-id=”welcome”>Welcome</h1>
  <button data-l10n-id=”french” onclick=”changeToFrench()”>French</button>
  <button data-l10n-id=”english” onclick=”changeToEnglish()”>English</button>
  <button data-l10n-id=”alert” onclick=”showAlert()”>Test Alert</button>

  <script type=”text/javascript”>
    document.addEventListener(“deviceready”, onDeviceReady, false);

    function onDeviceReady() {
      // Other Stuffs
    }

    function onLoad() {
      html10n.index();
      html10n.localize(‘en');
    }

    function changeToFrench() {
      html10n.localize(‘fr');
    }

    function changeToEnglish() {
      html10n.localize(‘en');
    }

    function showAlert() {
      var message = html10n.get(‘alert');
      alert(message);
    }

  </script>
</body>
</html>
```

## *发生了什么？*

您开发了一个能够根据用户所需的语言渲染不同文本消息的应用程序。当用户点击**法语**/**英语**按钮时，应用程序中的所有其他文本将自动更改。

# 添加多点触控手势支持

对于任何混合移动应用程序，触摸手势是一个重要的功能，它使得应用程序在用户手中表现得更好。

以下是可以使您的应用启用多点触控手势处理的 JavaScript 库列表。每个库都有其优点和局限性。一些与其他库有依赖关系，而一些则没有。在选择库时必须小心，因为它可能会向您的应用程序引入新的依赖项：

| 库名称 | 依赖关系 | 网址 |
| --- | --- | --- |
| ZeptoJS | No | [`zeptojs.com/`](http://zeptojs.com/) |
| EventJS | No | [`github.com/mudcube/Event.js`](https://github.com/mudcube/Event.js) |
| QuoJS | No | [`quojs.tapquo.com/`](http://quojs.tapquo.com/) |
| Hammer | No | [`hammerjs.github.io/`](http://hammerjs.github.io/) |
| ThumbsJS | No | [`mwbrooks.github.io/thumbs.js/`](http://mwbrooks.github.io/thumbs.js/) |
| jGestures | jQuery | [`jgestures.codeplex.com/`](http://jgestures.codeplex.com/) |
| DoubleTab | jQuery | [`github.com/technoweenie/jquery.doubletap`](https://github.com/technoweenie/jquery.doubletap) |
| Touchable | jQuery | [`github.com/dotmaster/Touchable-jQuery-Plugin`](https://github.com/dotmaster/Touchable-jQuery-Plugin) |
| TouchyJS | jQuery | [`github.com/HotStudio/touchy`](https://github.com/HotStudio/touchy) |

虽然还有更多库，但 Hammer 在移动应用的手势支持方面被广泛信任。Hammer 是一个开源库，可以识别由触摸、鼠标和 pointerEvents 制作出的手势。它没有任何依赖，并且压缩后的总大小小于 4 KB。

我们将使用 Hammer 举例说明如何在应用中实现触摸事件。

# 实施手势支持的时间

按照以下步骤使用 Hammer JavaScript 库在你的应用中实现多点触摸手势：

1.  打开命令行工具并创建一个名为 `hammer` 的新 PhoneGap 项目：

    ```js
    $ cordova create hammer

    ```

1.  使用命令行工具，添加你想要用于此测试的平台（Android、Blackberry、iOS 或 Windows Phone 8）：

    ```js
    $ cordova platform add android

    ```

1.  从 [`github.com/hammerjs/hammer.js`](https://github.com/hammerjs/hammer.js) 下载与 Hammer 相关的文件，并将其保存到 `www/js` 文件夹。你可以在 GitHub 的下载包中找到 `hammer.min.js` 和 `hammer.js` 文件。你可以只保留 `hammer.min.js` 文件，删除其余的。

1.  在你的 `index.html` 文件中包含 `hammer.min.js` 文件以开始使用 Hammer：

    ```js
    <script src=”js/hammer.min.js”></script>
    ```

1.  在页面主体中添加一个名为 `touch` 的 `div` 元素。此元素将附加到我们将要创建的所有触摸事件：

    ```js
    <div id=”touch”>Try Me</div>
    ```

1.  现在，让我们为“按下”手势创建事件监听器并将其附加到我们刚刚创建的 `div` 元素：

    ```js
    <script type=”text/javascript”>
      var touchId = document.getElementById(‘touch');

      var hammer = new Hammer(touchId);

      hammer.on(“press”, function(ev) {
        touchId.textContent = ev.type +” gesture detected.”;
      });

    </script”>
    ```

1.  打开命令行工具，进入项目文件夹，并在真实设备或模拟器上构建和运行应用：

    ```js
    $ cordova build
    $ cordova run

    ```

当你尝试在元素上应用“按下”动作时，你会看到 Hammer 已经检测到你的动作并改变了元素的文字内容。

### 注意

注意到 Hammer 库对开发者来说非常灵活。你可以在单个事件处理器中处理多个手势。在以下示例中，我们在单个函数中处理了平移、滑动、拖动和触摸手势。然而，过度使用它们可能会损害你的应用性能：

```js
<script type=”text/javascript”>
  var touchId = document.getElementById(myDiv);

  var hammer = new Hammer(touchId);

  //list of events to be handled
  hammer.on(“pan swipe drag touch press”, function(ev) {
    touchId.textContent = ev.type +” gesture detected.”;
  });
</script”>
```

默认情况下禁用了捏合和旋转识别器，因为它们可能会干扰应用的行为。但是，如果你想在应用中处理这两个手势，你可以通过以下方式启用它们：

```js
hammertime.get(‘pinch').set({ enable: true });
hammertime.get(‘rotate').set({ enable: true });
```

你还可以启用垂直或所有方向的拖动和滑动识别器，如以下所示：

```js
hammertime.get(‘pan').set({ direction: Hammer.DIRECTION_ALL });
hammertime.get(‘swipe').set({ direction: Hammer.DIRECTION_VERTICAL });
```

### 注意

有关所有手势识别器和自定义选项的更多详细信息，请参阅[`hammerjs.github.io/getting-started/`](http://hammerjs.github.io/getting-started/)。阅读此文档将帮助你以最高效的方式使用 Hammer。

# 处理点击延迟

我们已经看到移动应用可以支持各种触摸手势。随着新技术的引入，Web 应用和原生移动应用之间的差异越来越接近于零，但实际上并不是零。其中一个差异是移动应用中如何处理点击事件。

当您点击按钮时，移动浏览器会等待 300 毫秒来实际触发您所点击的按钮的点击事件。实际原因是移动浏览器等待查看用户是否想要执行点击或双击操作。在 300 毫秒的延迟之后，如果没有其他轻触，则被视为单次点击。然而，如果没有双击的事件处理器，这种延迟将是不必要的。通过克服这些延迟，您可以使得您的应用程序更加响应迅速，减少卡顿。

我们将介绍一种使用名为 **FastClick** 的库来避免这种 300 毫秒延迟的方法，该库可在 [`github.com/ftlabs/fastclick`](https://github.com/ftlabs/fastclick) 获取。步骤如下：

1.  将 `fastclick.js` 包含在您的 JavaScript 包中，或如以下所示将其添加到您的 HTML 页面中：

    ```js
    <script type='application/javascript' src=‘/path/to/fastclick.js'></script>
    ```

1.  按照您通常为其他 PhoneGap/Cordova 功能性所做的方式定义 `deviceready` 事件监听器：

    ```js
    function onBodyLoad(){

      document.addEventListener(“deviceready”, onDeviceReady, false);

    }
    ```

1.  现在，按照以下所示附加 FastClick 功能性：

    ```js
    function onDeviceReady()
    {
      FastClick.attach(document.body);
    }
    ```

在某些情况下，您可能需要 FastClick 忽略一些可能发生双击的元素。在这些情况下，您可以将 `needsclick` 类添加到元素中，如以下所示：

```js
<a class=”needsclick”>Ignored by FastClick</a>
```

### 注意

对于与该主题相关的更多高级讨论，请参阅 FastClick 的 GitHub 页面。如果您正在查看 Google 提出的替代方法，请查看 [`developers.google.com/mobile/articles/fast_buttons`](https://developers.google.com/mobile/articles/fast_buttons)。

# 摘要

在本章中，您学习了如何使用 PhoneGap 的 Globalization API 创建支持用户语言的本地化应用程序，以及如何创建支持多种用户语言的本地化应用程序。更重要的是，我们看到了如何更好地处理应用程序中的多点触控手势。

在下一章中，我们将介绍一些有助于使应用程序准备公开发布的主题。
