# 第六章：使用联系人及摄像头 API

*在上一章中，我们看到了如何充分利用文件和存储插件。现在，随着你对 PhoneGap 的了解越来越深入，是时候给设备本身添加一些交互了。本章的主要目标是帮助你理解 PhoneGap 的联系人 API 的使用，并使用摄像头和媒体捕获 API 与设备媒体进行交互。*

在本章中，你将：

+   获取 PhoneGap 联系人 API 及其对象和属性的概览

+   学习如何使用联系人 API 读取和过滤设备上存储的联系人

+   理解摄像头和捕获 API 之间的区别

+   学习如何使用摄像头 API 从设备摄像头捕获图像

+   学习如何使用媒体捕获 API 处理各种媒体类型，如图像、音频和视频

# 联系人 API

你可以使用 PhoneGap API 轻松访问设备上存储的联系人信息。Contacts API 是 W3C 的 Pick Contacts Intent API（一个允许在 Web 应用程序内部访问用户地址簿服务的 intent）的实现。你可以在[`www.w3.org/TR/contacts-api/`](http://www.w3.org/TR/contacts-api/)上了解更多关于 W3C 规范的信息。

Contacts API 所需的功能由名为`cordova-plugin-contacts`的 Contacts 插件提供。此插件支持主要平台。

对于所有支持平台的完整列表，请参阅[`docs.phonegap.com/en/edge/cordova_contacts_contacts.md.html#Contacts`](http://docs.phonegap.com/en/edge/cordova_contacts_contacts.md.html#Contacts)上的文档。

为了开始与设备联系人的交互，你可以使用存储在`navigator`对象中的`contacts`对象中定义的`create`或`find`方法：

```js
var contact = navigator.contacts.create(properties);
navigator.contacts.find(contactFields, contactSuccess, contactError, contactFindOptions);
```

为了更好地理解这些方法是如何工作的，让我们探索与之相关的最相关对象：`Contact`、`ContactName`、`ContactField`、`ContactFindOptions`和`ContactError`。

## 联系人名称对象

`ContactName`对象用于在 PhoneGap 框架中存储联系人名称的所有详细信息。该对象存储在`Contact`对象的`name`属性中。

`ContactName`对象的属性都是字符串，且具有自解释性：

+   `formatted`：这代表联系人的完整名称

+   `familyName`：这代表联系人的姓氏

+   `givenName`：这代表联系人的名字

+   `middleName`：这代表联系人的中间名

+   `honorificPrefix`：这代表联系人的前缀（例如，先生或博士）

+   `honorificSuffix`：这代表联系人的后缀（例如，Esq.）

## 联系人字段对象

`ContactField`对象是 PhoneGap 框架中用于表示`Contact`对象字段的通用对象。该对象的通用性质使其可以在多个字段中重复使用。

`ContactField`对象的属性如下：

+   `type`: 这是一个字符串，表示字段的类型；可能的值有家庭、工作、手机等

+   `value`: 这是一个字符串，表示字段的值，例如电话号码或电子邮件地址

+   `pref`: 这是一个布尔值，表示是否在特定字段中返回用户的首选值。

    ### 注意

    当`ContactField`对象用于`Contact`对象的`photos`属性时，`type`属性表示返回图像的类型（例如，URL 或 Base64 编码的字符串）。

## `ContactAddress`对象

`ContactAddress`对象是存储在`Contact`对象的`addresses`属性中的对象。`addresses`属性是一个数组，其中可以与每个联系人关联多个地址。

`ContactAddress`对象的属性如下：

+   `pref`: 这是一个布尔值，表示返回的`ContactAddress`对象是否是用户为`ContactAddress`对象指定的首选值

+   `type`: 这是一个字符串，表示存储在`ContactAddress`对象中的地址类型（例如，家庭和办公室）

+   `formatted`: 这是一个字符串，表示用于显示的完整地址

+   `streetAddress`: 这是一个字符串，表示完整的街道地址

+   `locality`: 这是一个字符串，表示`ContactAddress`对象中包含的城市或地区

+   `region`: 这是一个字符串，表示`ContactAddress`对象中包含的州或地区

+   `postalCode`: 这是一个字符串，表示与`ContactAddress`对象中存储的地区相关的邮政编码或邮政编码

+   `country`: 这是一个字符串，表示存储在`ContactAddress`对象中的国家名称

    ### 提示

    在 Android 2.x（即`pref`属性不受支持）和 Android 1.x 中存在一些限制。此外，iOS 不支持`formatted`属性。请始终参考在线文档以验证对`ContactAddress`对象的支持状态。

## `ContactOrganization`对象

`ContactOrganization`对象代表存储联系人的公司、组织等所有详细信息。该对象存储在`Contact`对象的`organizations`属性包含的数组中。

`ContactAddress`对象的属性如下：

+   `pref`: 这是一个布尔值，表示返回的`ContactOrganization`对象是否是用户为`ContactOrganization`对象指定的首选值

+   `type`: 这是一个字符串，表示存储在`ContactOrganization`对象中的地址类型（例如，工作和其他）

+   `name`: 这是一个字符串，表示存储在`ContactOrganization`对象中的组织名称

+   `department`: 这是一个字符串，表示联系人为之工作的组织部门

+   `title`：这是一个表示联系人在组织中的头衔的字符串

`name`、`department`和`title`属性在 iOS 上部分支持；`pref`和`type`属性在 Android 1.x 和 Android 2.x 上支持不佳。

## 联系人对象

`Contact`对象代表存储在设备数据库中的联系人的所有详细信息。可以使用对象本身定义的`save`、`remove`和`clone`方法将`Contact`对象保存、删除和从设备联系人数据库中复制。`save`和`remove`方法接受两个参数，以便处理保存或删除操作的成功或失败：

```js
var contact = navigator.contacts.create({‘displayName': ‘Giorgio'});

contact.save(onContactSaved, onContactSavedError);
contact.remove(onContactRemoved, onContactRemovedError);
```

错误处理程序接收相同的`ContactError`对象作为参数；成功处理程序在联系人成功删除时接收保存的联系人或当前数据库的快照。

`ContactError`对象包含关于发生错误的`code`属性中的信息。可以返回的值如下：

+   `ContactError.UNKNOWN_ERROR`

+   `ContactError.INVALID_ARGUMENT_ERROR`

+   `ContactError.TIMEOUT_ERROR`

+   `ContactError.PENDING_OPERATION_ERROR`

+   `ContactError.IO_ERROR`

+   `ContactError.NOT_SUPPORTED_ERROR`

+   `ContactError.PERMISSION_DENIED_ERROR`

当创建一个新的`Contact`对象时，你可以在调用`create`方法时定义联系人的属性或逐个传递它们。`Contact`对象的属性如下：

+   `id`：这是一个用作全局唯一标识符的字符串

+   `displayName`：这是一个表示用于向最终用户显示的`Contact`对象名称的字符串

+   `name`：这是一个包含联系人姓名所有信息的对象；用于存储此信息的对象是`ContactName`

+   `nickname`：这是一个表示联系人非正式名称的字符串

+   `phoneNumbers`：这是一个包含所有联系人电话号码的数组；数组项是`ContactField`对象的实例

+   `emails`：这是一个包含所有联系人电子邮件地址的数组；数组项是`ContactField`对象的实例

+   `addresses`：这是一个包含所有联系人地址的数组；数组项是`ContactAddresses`对象的实例

+   `ims`：这是一个包含所有联系人即时消息账户的数组；数组项是`ContactField`对象的实例

+   `organizations`：这是一个包含联系人所属的所有组织的数组；数组项是`ContactOrganization`对象的实例

+   `birthday`：这是一个表示联系人出生日期的`Date`对象

+   `note`：这是一个表示关于联系人的注释的字符串

+   `photos`：这是一个包含所有联系人照片的数组；数组项是`ContactField`对象的实例

+   `categories`：这是一个包含所有联系人定义类别的数组；数组项是`ContactField`对象的实例

+   `urls`：这是一个包含与联系人相关联的所有网页的数组；数组项是`ContactField`对象的实例

`Contact` 对象的属性在所有平台上并不完全受支持。实际上，操作系统的碎片化使得一致地处理这些信息变得困难。

例如，`name`、`nickname`、`birthday`、`photos`、`categories` 和 `urls` 属性在 Android 1.x 上不受支持。同样，`categories` 属性在 Android 2.x 和 iOS 上也不受支持。

在 iOS 上，`photos` 数组中返回的项目包含一个指向应用临时文件夹的 URL。这意味着当应用退出时，此内容将被删除，并且如果你想让用户以相同的状态找到应用，你必须处理它。`displayName` 属性在 iOS 上不受支持，除非定义了 `ContactName`，否则将返回 null。如果定义了 `ContactName` 对象，则返回一个复合名称或昵称。

## 过滤联系人数据

我们已经提到了 `navigator.contacts` 对象中可用的 `find` 方法。使用此方法，一个应用可以在设备的联系人数据库中查找一个或多个联系人。`find` 方法接受四个参数。第一个是一个包含必须返回的 `Contact` 对象字段名称的数组。第二个和第三个是成功和错误处理程序。最后一个代表你可能希望应用于当前搜索的过滤选项。

为了对当前搜索应用过滤器，你可以实例化一个新的 `ContactFindOptions` 对象，并填充 `filter` 和 `multiple` 属性。`filter` 属性是一个不区分大小写的字符串，它将在 `find` 方法返回的 `Contact` 对象的字段上作为过滤器。`multiple` 属性默认为 `false`，并且是在成功处理程序中接收多个 `Contact` 对象时应该使用的。尝试以下示例以了解这些属性如何工作。

# 行动时间 - 搜索设备联系人

现在我们将看到一个从设备获取所有联系人并将其列出的例子：

1.  使用 PhoneGap CLI 创建一个新项目：

    ```js
    $ phonegap create ContactsApi

    ```

1.  将当前工作目录更改为新创建的目录：

    ```js
    $ cd ContactsApi

    ```

1.  将所需的平台添加到项目中。在这个例子中，我们将添加 Android：

    ```js
    $ phonegap platform add android

    ```

1.  将联系人插件安装到项目中：

    ```js
    $ phonegap plugin add cordova-plugin-contacts

    ```

1.  在 `www/index.html` 文件中，你需要替换内容并添加以下代码。首先，让我们创建一个事件监听器并将其绑定到一个函数上。在下面的代码中，当设备准备好时，`OnDeviceReady` 方法将被触发：

    ```js
    <script type=”text/javascript” charset=”utf-8”>
        document.addEventListener(“deviceready”, onDeviceReady, false);
    </script>
    ```

1.  现在，让我们为 `OnDeviceReady` 方法添加内容：

    ```js
    function onDeviceReady() {
        var options = new ContactFindOptions();
        options.filter = “”;
        options.multiple=true;
        var fields = [“displayName”, “name”, “addresses”];
        navigator.contacts.find(fields, onSuccess, onError, options);
    }
    ```

    使用前面的代码，你正在创建一个新的 `ContactFindOptions` 对象，并将过滤器/属性设置到该对象上。根据你设置的过滤器，该函数将返回所有匹配的联系人。在这个例子中，我们提供了一个空过滤器，这意味着我们将从设备中获取所有联系人。

    如果您只想限制搜索特定名称，您可以提供如所示的过滤器：

    ```js
    options.filter = “John”;
    ```

    您还可以限制函数将要返回的字段列表。如果您觉得只需要输出几个字段就足够了，您可以在字段列表中提及它们。例如，我们已将输出限制为仅显示名称、`name`对象和联系人的地址。

1.  当函数执行成功时，它将返回值到`OnSuccess`事件，并在出现问题时触发`OnError`事件。因此，让我们定义`OnSuccess`和`OnError`事件来定义我们将如何处理输出：

    ```js
    function onSuccess(contacts) {
        for (var i = 0; i < contacts.length; i++) {
            console.log(“Display Name = “ + contacts[i].displayName);
        }
    };

    function onError(contactError) {
        alert(‘onError!');
    };
    ```

1.  当我们获取所有联系信息后，我们遍历`contacts`对象，并在控制台中显示联系人的姓名。

以下完整代码已提供供您参考。当您在实际设备上测试代码时，您可以在控制台中看到联系人名称：

```js
<script type=”text/javascript” charset=”utf-8”>
document.addEventListener(“deviceready”, onDeviceReady, false);

function onDeviceReady() {
    var options = new ContactFindOptions();
    options.filter = “”;
    options.multiple=true;
    var fields = [“displayName”, “name”, “addresses”];
    navigator.contacts.find(fields, onSuccess, onError, options);
}

function onSuccess(contacts) {
    for (var i = 0; i < contacts.length; i++) {
        console.log(“Display Name = “ + contacts[i].displayName);
    }
};

function onError(contactError) {
    alert(‘onError!');
};
</script>
```

## *发生了什么？*

您根据`ContactFindOptions`对象定义的选项过滤了联系数据库，并使用 PhoneGap 框架提供的 API 对结果进行了细化。

# 动手时间 – 添加新联系人

我们将直接通过 Contacts API 创建一个新的联系条目示例。此代码如下：

```js
<script type=”text/javascript” charset=”utf-8”>
    // Binding to the events
    document.addEventListener(“deviceready”, onDeviceReady, false);

    // device APIs are ready and available to use
    function onDeviceReady() {
        var myContact = navigator.contacts.create({“displayName”: “Purus Test”});

        var phoneNumbers = [];

        phoneNumbers[0] = new ContactField(‘work', ‘1234567890', false);
        phoneNumbers[1] = new ContactField(‘mobile', ‘2345678901', true); // preferred
        phoneNumbers[2] = new ContactField(‘home', ‘3456789012', false);

        // You can add any other values of Contact object with name and phone numbers
        myContact.phoneNumbers = phoneNumbers;

        myContact.save(onSaveContactSuccess,onSaveContactError);

    }

    function onSaveContactSuccess(contact) {
        alert(‘saved successfully');
    };

    function onSaveContactError(error) {
        alert(error.code);
    };

</script>
```

`ContactField`的最后一个参数期望一个布尔值，表示该字段是否为首选字段。在上一个示例中，值为`true`的第二个电话号码被设置为首选号码。

## *发生了什么？*

我们已经看到如何创建一个新的`Contact`对象并将其使用 Contacts API 保存到设备上。

# Camera API 还是 Capture API？

PhoneGap 框架实现了两个不同的 API 来访问设备上的媒体：Camera API 和 Capture API。这两个 API 之间的主要区别是，Camera API 只能访问默认的设备相机应用程序，而 Capture API 可以使用默认的音频和视频录制应用程序录制音频或视频。另一个重要区别是，Capture API 允许通过单个 API 调用进行多次捕获。

### 注意

Capture API 是已废弃的 W3C 标准草案的一个实现。如您所见，该草案与实际的 PhoneGap 实现有几种相似之处。

# 使用 Camera API 访问相机

Camera API 通过`cordova-plugin-camera`键标识的 Camera 插件提供对设备相机应用程序的访问。安装此插件后，应用程序可以拍照或访问用户在设备上创建的照片库和相册中存储的媒体文件。Camera API 公开了在`navigator.camera`对象中定义的以下两个方法：

+   `getPicture`：此方法打开默认的相机应用程序或允许用户根据方法接受的`configuration`对象中指定的选项浏览媒体库

+   `cleanup`: 这将清理临时存储位置中可用的任何中间照片文件（仅在 iOS 上受支持）

作为参数，`getPicture`方法接受一个成功处理程序、一个失败处理程序，以及可选地使用其属性指定多个相机选项的对象，如下所示：

+   `quality`: 这是一个介于`0`和`100`之间的数字，用于指定保存图像的质量。

+   `destinationType`: 这是一个用于定义成功处理程序中返回的值格式的数字。可能的值存储在以下`Camera.DestinationType`伪常量中：

    +   `DATA_URL(0)`: 这表示`getPicture`方法将以 Base64 编码的字符串形式返回图像

    +   `FILE_URI(1)`: 这表示该方法将返回文件 URI

    +   `NATIVE_URI(2)`: 这表示该方法将返回一个平台相关的文件 URI（例如，iOS 上的`assets-library://`或 Android 上的`content://`）

+   `sourceType`: 这是一个用于指定`getPicture`方法可以访问图像的位置的数字。以下可能的值存储在`Camera.PictureSourceType`伪常量中：`PHOTOLIBRARY (0)`、`CAMERA (1)`和`SAVEDPHOTOALBUM (2)`：

    +   `PHOTOLIBRARY`: 这表示该方法将从设备的库中获取图像

    +   `CAMERA`: 这表示该方法将从相机获取图片

    +   `SAVEDPHOTOALBUM`: 这表示在用户选择图像之前，将提示用户选择相册

+   `allowEdit`: 这是一个布尔值（默认值为`true`），用于指示用户在确认选择之前可以对图像进行小幅度编辑；此功能仅在 iOS 上有效。

+   `encodingType`: 这是一个用于指定返回文件编码的数字。可能的值存储在`Camera.EncodingType`伪常量中：`JPEG (0)`和`PNG (1)`。

+   `targetWidth`和`targetHeight`：这些是像素宽度和高度，您希望捕获的图像缩放到这些值；可以指定这两个选项中的任意一个。当两者都指定时，图像将缩放到产生最小宽高比（图像的宽高比描述了其宽度和高度之间的比例关系）的值。

+   `mediaType`: 这是一个用于指定当使用`Camera.PictureSourceType.PHOTOLIBRARY`或`Camera.PictureSourceType.SAVEDPHOTOALBUM`伪常量作为`sourceType`时，`getPicture`方法必须返回哪种类型的媒体文件。可能的值存储在`Camera.MediaType`对象中作为伪常量，并且是`PICTURE (0)`、`VIDEO (1)`和`ALLMEDIA (2)`。

+   `correctOrientation`: 这是一个布尔值，它强制设备相机在捕获过程中纠正设备方向。

+   `cameraDirection`: 这是一个用于指定在捕获过程中必须使用哪个设备相机的数字。这些值存储在`Camera.Direction`对象中作为伪常量，并且是`BACK (0)`和`FRONT (1)`。

+   `popoverOptions`：这是一个在 iOS 上支持的对象，用于指定在从图库或相册选择图片时使用的弹出视图的锚点元素位置和箭头方向。

+   `saveToPhotoAlbum`：这是一个布尔值（默认值为 `false`），用于将捕获的图片保存到设备默认的照片相册中。

成功处理程序接收一个参数，该参数包含文件或数据的 URI，该数据存储在文件的 Base64 编码字符串中，具体取决于 `options` 对象中 `encodingType` 属性的值。失败处理程序接收一个字符串作为参数，该字符串包含设备的原生代码错误信息。

同样，`cleanup` 方法接受一个成功处理程序和一个失败处理程序。这两个方法之间的唯一区别是成功处理程序不接收任何参数。`cleanup` 方法仅在 iOS 上受支持，可以在 `sourceType` 属性值为 `Camera.PictureSourceType.CAMERA` 且 `destinationType` 属性值为 `Camera.DestinationType.FILE_URI` 时使用。

为了将关于 Camera API 的所学知识付诸实践，让我们创建一个可以从设备默认相机应用程序中拍照的应用程序。

# 行动时间 - 访问设备相机

准备访问设备的相机并向用户展示捕获的图片。请参考以下步骤：

1.  打开命令行工具并创建一个名为 `camera` 的新 PhoneGap 项目：

    ```js
    $ cordova create camera

    ```

1.  切换到创建的目录：

    ```js
    $ cd camera

    ```

1.  使用命令行工具，将 Android 和 iOS 平台添加到项目中：

    ```js
    $ cordova platforms add android
    $ cordova platforms add ios

    ```

1.  使用以下命令行添加 Camera API 插件：

    ```js
    $ cordova plugin add cordova-plugin-camera

    ```

1.  前往 `www` 文件夹，打开 `index.html` 文件，并用以下代码替换其内容；此代码是自解释的：

    ```js
    <!DOCTYPE html>
    <html>
      <head>
        <script type=”text/javascript” charset=”utf-8” src=”cordova.js”></script>
        <script type=”text/javascript” charset=”utf-8”>

        // Wait for device API libraries to load
        document.addEventListener(“deviceready”, onDeviceReady,false);

        // device APIs are available
        function onDeviceReady() {
        }

        function getCameraImage() {
          // Take picture using camera and get image source as base64-encoded string
          navigator.camera.getPicture(onCaptureSuccess, onError, { quality: 20, allowEdit: false, destinationType: navigator.camera.DestinationType.DATA_URL });
        }

        // Called when a photo is successfully retrieved
        function onCaptureSuccess(imageData) {
          var smallImage = document.getElementById(‘cameraImage');
          smallImage.src = “data:image/jpeg;base64,” + imageData;
        }

        // Capture any failures
        function onError(message) {
          alert(‘Error: ‘ + message);
        }
        </script>
      </head>
      <body>
        <button onclick=”getCameraImage();”>Capture Photo</button> <br>
        <img style=”display:none;width:60px;height:60px;” id=“cameraImage” src=”” />
      </body>
    </html>
    ```

    当用户点击 **捕获照片** 按钮时，`getCameraImage()` 函数将被触发，进而使用 `navigator.camera.getPicture()` 方法从设备相机捕获图像，并在捕获成功的情况下将图像数据传递给 `onCaptureSuccess()` 方法。稍后，将编码字符串格式的数据源分配给 `<img>` 标签的源。

1.  如果你想启用照片编辑功能，将 `allowEdit` 属性设置为 `true`，如下所示：

    ```js
    navigator.camera.getPicture(onCaptureSuccess, onError, { quality: 20, allowEdit: true, destinationType: navigator.camera.DestinationType.DATA_URL });
    ```

1.  一旦你进行了更改，你需要构建项目以在你的实际设备上尝试应用程序。为此，打开命令行工具并从项目的根目录启动 `prepare` 命令，然后启动 `compile` 命令：

    ```js
    $ cordova prepare
    $ cordova compile

    ```

    ### 小贴士

    `prepare` 和 `compile` 命令可以从项目的任何文件夹中执行。`build` 命令是 `prepare` 和 `compile` 命令的简写。

1.  在每个目标平台上运行项目（遗憾的是，模拟器不支持相机）。为了在实际设备平台上测试应用程序，你可以使用如下所示的 `run` 命令：

    ```js
    $ cordova run android

    ```

    ### 注意

    在运行此代码之前，您需要使您的设备准备好测试，并且每个平台的步骤都不同。例如，您需要在您的 Android 设备上启用 USB 调试选项。有关每个平台的设置信息，您可以访问 [`cordova.apache.org/docs/en/edge/index.html`](https://cordova.apache.org/docs/en/edge/index.html)。

如前几节所述，`cleanup` 方法仅在 iOS 上有效。这意味着您的应用程序的外观和工作方式应根据目标平台的不同而有所不同。

使用 Cordova 命令行工具创建新项目时，项目根目录下会创建一个名为 `merges` 的文件夹。此文件夹包含为项目添加的每个平台的一个单独文件夹；PhoneGap 项目的根文件夹如下所示：

```js
├── config.xml
├── hooks
├── merges
├── platforms
├── plugins
├── www
   └── css
   └── img
   └── js
   └── index.html
```

当您必须处理特定目标平台的不同用户界面元素或业务逻辑时，您可以将要合并的文件放置在 `merges/TARGET_PLATFORM` 文件夹中。您可以通过实现 `cleanup` 方法创建 `index.html` 和 `index.js` 文件，专门针对 iOS 平台。

## *发生了什么？*

您已访问设备相机并学习了如何使用 Cordova 的合并功能来处理 Android 和 iOS 上不同的 Camera API 实现。一旦构建了应用程序，如果您转到特定平台的文件夹（`platforms/android/assets/www` 和 `platforms/ios/www`）并打开 `index.js` 文件，您会发现它们彼此不同。

## 控制相机弹出窗口

iOS 中的 `getPicture` 方法（特别是在 iPad 上）当 `sourceType` 属性值为 `Camera.PictureSourceType` 对象中定义的以下伪常量之一时返回 `CameraPopoverHandle` 对象：`SAVEDPHOTOALBUM` 或 `PHOTOLIBRARY`。使用此对象，可以控制在调用 `getPicture` 方法时创建的弹出对话框的位置。

`CameraPopoverHandle` 对象仅公开 `setPosition` 方法，该方法需要一个 `CameraPopoverOptions` 对象作为参数。此对象允许您指定此对话框的坐标、尺寸和箭头位置：

```js
var popoverOptions = new CameraPopoverOptions();
popoverOptions.x = 220;
popoverOptions.y = 600;
popoverOptions.width = 320;
popoverOptions.height = 480;
popoverOptions.arrowDir = Camera.PopoverArrowDirection.ARROW_DOWN;
```

您可以使用更紧凑的语法达到相同的结果，在 `CameraPopoverOptions` 对象的构造函数中指定属性：

```js
var popoverOptions = new CameraPopoverOptions(220, 600, 320, 480, Camera.PopoverArrowDirection.ARROW_DOWN);
```

还可以使用 `cameraOptions` 对象的 `popoverOptions` 属性来指定坐标、位置和箭头方向：

```js
cameraOptions.popoverOptions = {
                x : 220,
                y :  600,
                width : 320,
                height : 480,
                arrowDir : Camera.PopoverArrowDirection.ARROW_DOWN

};
```

`PopoverArrowDirection` 对象中定义的伪常量与在 `UIPopoverArrowDirection` 类中定义的本地 iOS 常量相匹配：

```js
Camera.PopoverArrowDirection = {
                                ARROW_UP: 1,
                                ARROW_DOWN: 2,
                                ARROW_LEFT: 4,
                                ARROW_RIGHT: 8,
                                ARROW_ANY: 15
};
```

当您想要在设备方向改变时控制对话框的位置时，此选项非常有用。在下一个示例中，您将控制对话框的位置和大小以覆盖默认行为。

### 注意

如果您想使用不同的 iOS 设备（例如，iPhone 5 和 4S 以测试不同的屏幕尺寸）进行开发，您必须创建一个配置文件并将其添加到设备中。您可以为所有项目创建一个配置文件，或者为每个项目使用一个文件。为了为您的设备创建新文件，您必须执行以下操作：

1.  通过点击在连接设备时在 iTunes 中看到的序列号来读取设备 ID。

1.  前往 Apple 开发者门户，登录，并将新的开发设备添加到您的账户。

1.  创建并下载一个新的配置文件（您应该已经有证书可用，否则请阅读 Apple 开发者门户中可用的信息[`developer.apple.com/support/technical/certificates/`](https://developer.apple.com/support/technical/certificates/)）。

1.  使用 Xcode 组织者窗口（在 Xcode 中，导航到**Window** | **Organizer**）将配置文件添加到您的设备。

现在也可以直接将设备连接到 Xcode，让 Xcode 为您注册和管理配置文件。

# 行动时间 - 控制相册的位置

使用以下步骤更改 iPad 上默认相册对话框的位置：

1.  前往您之前创建的 PhoneGap 项目的`merges/ios/js`文件夹，打开`index.js`文件，并添加一个名为`initAdditionalOptions`的新函数：

    ```js
    initAdditionalOptions: function(){

        // Additional options will be defined here

    }
    ```

1.  在`initAdditionalOptions`函数的主体中，指定设备相册作为`getCamera`方法的数据源，以及弹出对话框的大小和位置：

    ```js
    app.cameraOptions.sourceType = Camera.PictureSourceType.SAVEDPHOTOALBUM;

    app.cameraOptions.popoverOptions = new CameraPopoverOptions(220, 600, 320, 480, Camera.PopoverArrowDirection.ARROW_DOWN);
    ```

1.  在`deviceready`事件处理器的末尾添加对`initAdditionalOptions`函数的调用：

    ```js
    deviceready: function() {
        // All the events handling initializations are here
        app.initAdditionalOptions();

    }
    ```

1.  打开命令行工具，运行`prepare`和`compile`命令（或`build`命令），以便在真实设备上测试项目。

## *刚才发生了什么？*

您仅使用 JavaScript 就在 iOS 的 iPad 上处理了默认对话框的大小和位置。

# 媒体捕获 API

现代设备为用户提供了丰富的媒体功能；目前，人们可以录制视频、录制音频、拍照，并在他们的通信流程中使用所有这些媒体。

媒体捕获 API 与大多数 PhoneGap API 一样异步工作，并提供对设备音频、图像和视频捕获功能的访问。为了开始使用此 API，您必须像下面这样将插件安装到项目中：

```js
$ phonegap plugin add cordova-plugin-media-capture

```

完成后，您可以通过`navigator.device`对象访问存储的`capture`对象：

```js
var capture = navigator.device.capture;
```

一旦您获得对`capture`对象的访问权限，您可以通过以下属性检测设备支持的哪些视频、音频和图像格式：

+   `supportedAudioModes`

+   `supportedImageModes`

+   `supportedVideoModes`

每个属性都返回一个`ConfigurationData`对象的数组；数组中的每个项目代表一个支持的媒体类型。在`ConfigurationData`对象中定义了三个属性，您可以使用这些属性清楚地识别设备支持的媒体类型：

+   `type`：这是一个小写字符串，表示根据在[`www.ietf.org/rfc/rfc2046.txt`](http://www.ietf.org/rfc/rfc2046.txt)中解释的 RFC2046 标准所支持的媒体类型（即，`video/3gpp`、`video/quicktime`、`image/jpeg`、`audio/amr`等）

+   `height`：这是一个表示支持的图像或视频高度的数字（当对象表示支持的音频格式时，该属性返回`0`）

+   `width`：这是一个表示支持的图像或视频宽度的数字（当对象表示支持的音频格式时，该属性返回`0`）

    ### 注释

    在撰写本文时，`ConfigurationData`对象在任何支持的平台上都没有实现。实际上，存储在`supportedAudioModes`、`supportedImageModes`和`supportedVideoModes`属性中的每个数组都是空的。

`capture`对象公开了三种方法，以便访问设备的视频、音频和图像捕获功能：

+   `captureVideo`

+   `captureAudio`

+   `captureImage`

这些方法具有相同的语法：每个方法都接受一个成功处理程序、一个失败处理程序和一个`option`对象作为参数。成功处理程序在媒体捕获操作成功时被调用，并接收一个`MediaFile`对象的数组作为参数，该数组描述了每个捕获的文件。如果媒体捕获操作期间发生错误或用户取消操作，则调用错误处理程序，并接收一个`CaptureError`对象作为参数。以下示例使用设备相机捕获图像：

```js
var capture = navigator.device.capture;

capture.captureImage(function(files){

                         console.log(files);

                     }, function(error){

                         console.log(error);

                     });
```

成功处理程序返回的文件数组中存储的`MediaFile`对象描述了捕获的媒体。`MediaFile`对象的属性如下：

+   `fullPath`：这是一个表示设备上文件路径（包括文件名）的字符串。

+   `lastModifiedDate`：这是自 1970 年 1 月 1 日以来的文件修改日期（有关 JavaScript 中`Date`对象的更多信息，请参阅[`developer.mozilla.org/en/docs/JavaScript/Reference/Global_Objects/Date`](https://developer.mozilla.org/en/docs/JavaScript/Reference/Global_Objects/Date)）

+   `name`：这是一个表示文件名的字符串。该名称由`lastModificationDate`值和文件扩展名组成。

+   `size`：这是一个表示文件大小的数字（以字节为单位）。

+   `type`：这是一个表示捕获文件的 MIME 类型的字符串（例如，`image/jpeg`）。

返回给错误处理程序的`CaptureError`对象只有一个属性，即`code`。该属性包含一个整数，等于`CaptureError`对象中定义的以下伪常量之一：

+   `CaptureError.CAPTURE_INTERNAL_ERR` (返回值 `0`): 这表示设备未能捕获视频、图像或声音

+   `CaptureError.CAPTURE_APPLICATION_BUSY` (返回值 `1`): 这表示捕获应用程序目前正在处理另一个捕获请求

+   `CaptureError.CAPTURE_INVALID_ARGUMENT` (返回值 `2`): 这表示在调用 API 时应用使用了无效的参数（例如，`limit`参数的值小于 1）

+   `CaptureError.CAPTURE_NO_MEDIA_FILES` (返回值 `3`): 这表示用户在捕获任何内容之前退出了相机应用程序或音频捕获应用程序

+   `CaptureError.CAPTURE_NOT_SUPPORTED` (返回值 `20`): 这表示请求的捕获操作不受支持

`option`对象因方法而异。实际上，捕获 API 为每种捕获定义了一个不同的对象：`CaptureVideoOptions`、`CaptureAudioOptions`和`CaptureImageOptions`。所有这些对象都具有相同的属性、限制和模式；`duration`属性仅在`CaptureVideoOptions`和`CaptureAudioOptions`对象中定义。`limit`属性的默认值是`1`，用于指定用户在返回应用之前可以进行的捕获次数。`duration`属性是捕获的最大长度（以秒为单位）。`mode`属性表示选定的视频或音频模式。

### 注意

对配置选项的支持非常零散。例如，`CaptureImageOptions`和`CaptureVideoOptions`对象的`limit`属性在 iOS 中不受支持。请参阅[`docs.phonegap.com/en/edge/cordova_media_capture_capture.md.html#Capture`](http://docs.phonegap.com/en/edge/cordova_media_capture_capture.md.html#Capture)文档以检查实现的实际状态。

操作图像最高效的方式是通过原生代码；然而，您也可以通过 JavaScript 执行简单的图像操作。为了避免使以下示例过于复杂，我们使用了 HTML Canvas 来渲染带有一些效果的图像。

HTML 的`<canvas>`标签是一个容器，用于使用 JavaScript 等脚本语言动态绘制图形。它有几个内置函数，我们将在下面使用它们。

### 注意

有关 HTML5 Canvas 的更多信息，请参阅[`www.html5canvastutorials.com/`](http://www.html5canvastutorials.com/)。

# 行动时间 - 使用 canvas 操作图像

准备使用媒体捕获 API 获取的图像应用棕褐色效果。执行以下步骤：

1.  打开命令行工具并创建一个名为`ImageEffect`的新 PhoneGap 项目：

    ```js
    $ cordova create ImageEffect

    ```

1.  将路径更改为新创建的项目：

    ```js
    $ cd ImageEffect

    ```

1.  使用以下命令添加媒体捕获 API 插件：

    ```js
    $ cordova plugin add cordova-plugin-media-capture

    ```

1.  在现有标记中添加一个`canvas`标签，其`id`值为`#manipulatedImage`，以便使用它来渲染操作后的图像：

    ```js
    <canvas id='manipulatedImage' />
    ```

1.  一旦`deviceready`事件被触发，访问设备相机，这允许用户仅访问一个图像：

    ```js
    var capture = navigator.device.capture;
    capture.captureImage(onGetImage, onImageError, {limit: 1});
    ```

1.  定义成功处理器并访问作为参数返回的数组中存储的文件信息：

    ```js
    function onGetImage(files){

        var currentFile = files[0];

        // Canvas access logic will go here
    }
    ```

1.  获取对画布的访问权限，并存储画布`2d`上下文的引用，以便能够在其上绘制内容：

    ```js
    var canvas = document.querySelector(‘#manipulatedImage');
    var context = canvas.getContext(‘2d');

    // Image object definition and load will go here
    ```

1.  创建一个`image`对象，将处理器分配给`onload`属性，并使用`MediaFile`对象的`fullPath`属性定义对象的`src`属性：

    ```js
    var image = new Image();

    image.onload = function(evt) {
        // The image manipulation logic will go here
    };

    image.src = currentFile.fullPath;
    ```

1.  在存储在`onload`属性中的函数中，在画布上绘制图像，获取像素，操作它们，并将像素重新分配到画布：

    ```js
    var width = canvas.width;
    var height = canvas.height;

    context.drawImage(this, 0, 0, width, height);

    var imgPixels = context.getImageData(0, 0, width, height);
    context.putImageData(grayscale(imgPixels), 0, 0, 0, 0, width, height);
    ```

本例的完整代码在此提供，供你理解：

```js
<!DOCTYPE html>
<html>
    <head>
        <!--Other section removed for sake of simplicity -->
        <title>Media Capture Example</title>
    </head>
<body>
<canvas id='manipulatedImage' />

<script type=”text/javascript” src=”cordova.js”></script>

<script type=”text/javascript”>

   document.addEventListener(“deviceready”, onDeviceReady, false);

   function onDeviceReady() {
      var capture = navigator.device.capture;
      capture.captureImage(onGetImage, onError, {limit: 1});
         }

   function onGetImage(files){
      var currentFile = files[0];

      var canvas = document.querySelector(‘#manipulatedImage');
      var context = canvas.getContext(‘2d');
      var image = new Image();

      image.onload = function(evt) {
         var width = canvas.width;
         var height = canvas.height;

         context.drawImage(this, 0, 0, width, height);
         var imgPixels = context.getImageData(0, 0, width, height);
         context.putImageData(grayscale(imgPixels), 0, 0, 0, 0, width, height);
      };
      image.src = currentFile.fullPath;
   }

   function onError(error){
      alert(JSON.stringify(error));
   }
</script>
</body>
</html>
```

你可以在[`www.html5rocks.com/en/tutorials/canvas/imagefilters/`](http://www.html5rocks.com/en/tutorials/canvas/imagefilters/)找到`grayscale`函数和几个图像效果。

### 注意

当你引用任何插件或 PhoneGap 特定的对象，如`CameraPopoverOptions`时，你必须等待`deviceready`事件被触发，以便所需的对象可用于使用：

```js
if (typeof window.plugins.CameraPopoverOptions !== ‘undefined'){
    // plugin object is available to be used
}
```

一些插件定义全局对象，并且可以使用它们来验证插件是否已加载并可使用：

```js
function onDeviceReady() {
   console.log(navigator.device.capture);
}
```

# 摘要

在本章中，你学习了关于联系人 API 的内容，如何从设备中查找联系人，如何过滤联系人，以及如何将新联系人保存到设备中。你还学习了相机和媒体捕获 API 之间的区别以及如何在你的应用程序中使用它们。

在下一章中，你将学习如何访问设备传感器 API 以确定设备方向、位置，以及如何实现位置 API。
