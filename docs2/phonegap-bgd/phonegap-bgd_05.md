# 第五章。使用设备存储和 Files API

*你的 PhoneGap 知识正在逐步积累。现在是时候添加一些与外部数据源以及与设备本身的交互了。本章的主要目标是指导你了解 PhoneGap 的离线存储功能，并帮助你理解如何与 Files API 交互。*

在本章中，你将：

+   学习如何使用`localStorage`对象在设备上读取和写入数据

+   学习如何根据特定平台实现处理本地数据库的存储

+   理解数据库存储限制并学习如何处理它们

+   了解 Files API 的工作原理，以及如何组织代码以保持其清晰和可维护性

+   使用 Files API 来探索设备文件系统

+   学习如何在文件内部读取和渲染数据

+   学习如何将文件加载和保存到设备的持久存储中

# 应用数据存储

每个应用程序（桌面、网页或移动）都需要存储（并访问）一些数据以正常工作。数据如何存储取决于应用程序将处理的信息类型以及应用程序将在其中运行的环境。例如，一个网页应用程序可以主要依赖服务器存储，因为它在互联网上运行。大多数高级网页应用程序实现离线策略，并在用户机器上本地存储一些数据。

现代网络开发提供了几个工具，以便让用户在未连接时也能与应用程序交互：

+   **LocalStorage** API（[`www.w3.org/TR/webstorage/#the-localstorage-attribute`](http://www.w3.org/TR/webstorage/#the-localstorage-attribute)）

+   **SessionStorage** API（[`www.w3.org/TR/webstorage/#the-sessionstorage-attribute`](http://www.w3.org/TR/webstorage/#the-sessionstorage-attribute)）

+   **ApplicationCache**接口（[`www.w3.org/TR/2011/WD-html5-20110525/offline.html`](http://www.w3.org/TR/2011/WD-html5-20110525/offline.html)）

+   **IndexedDB** API（[`www.w3.org/TR/IndexedDB/`](http://www.w3.org/TR/IndexedDB/)）

所有现代移动浏览器都允许开发者通过 navigator 对象处理在线和离线事件（[`developer.mozilla.org/en/docs/Online_and_offline_events`](https://developer.mozilla.org/en/docs/Online_and_offline_events)）。

### 注意

重要的是要记住，关于`OnLine`属性的规范报告称，该属性本身是不可靠的，因为一台计算机可以连接到网络而没有互联网访问权限。

本书范围之外是对先前 API、事件和界面的全面概述。在以下章节中，我将仅讨论对构建 PhoneGap 应用程序最相关的部分：LocalStorage 和 IndexedDB。

## 探索 PhoneGap LocalStorage API

主要有两种网络存储类型：**本地存储**和**会话存储**。LocalStorage API 是 W3C 定义的**WebStorage** API 的一部分，旨在为网络客户端的键值对数据持久存储提供指导。LocalStorage API 旨在支持需要在会话之间可用的数据。换句话说，使用 LocalStorage API 保存的应用程序数据将在应用程序下次运行时再次可用。

为了访问 LocalStorage API，你必须将`window`对象引用到其`localStorage`属性。如果你在浏览器控制台中输入以下代码片段，你可以查看`localStorage`对象的方法和属性：

```js
console.log(window.localStorage);
```

以下列表总结了可用的方法和属性：

+   `key`: 这返回存储在特定位置的键名；你可以通过键或索引访问`localStorage`数据

+   `getItem`: 这返回由键标识的值

+   `setItem`: 这将值保存到`localStorage`对象的特定键（即字符串）中；该方法需要一个字符串作为键和一个要存储在特定键中的值

+   `removeItem`: 这将从`localStorage`对象中删除由键标识的项

+   `clear`: 这将从`localStorage`对象中删除所有键值对

+   `length`: 这返回存储在`localStorage`对象中的项的总数

`localStorage`对象允许你仅以键值对的形式存储简单的字符串数据。如果你想存储更复杂的数据，你必须使用 JSON 或其他数据的字符串表示形式。当你使用 JSON 格式的字符串来存储值时，当你想要使用它时，你需要将字符串转换回 JSON。每次`localStorage`对象更新时，都会触发`StorageEvent`。此事件不能取消，并包含以下属性：

+   `key`: 这是一个表示已添加、删除或修改的存储中的命名键的字符串

+   `oldValue`: 如果更新了命名键，则这是该键的先前值；如果向`localStorage`对象中添加了新项，则为 null

+   `newValue`: 这是键的新值，如果移除了项则为 null

+   `url`: 这是调用触发此数据更改的方法的 HTML 页面的地址

### 小贴士

如果可以防止事件的默认操作，则 JavaScript 事件是可取消的。

请记住，存储事件不会在同一个窗口或标签页上工作；它们只为使用相同`localStorage`对象的其他窗口或标签页触发。

PhoneGap 的`localStorage`功能允许你作为开发者编写类似于浏览器的代码；它是为你处理不同平台（Android、BlackBerry WebWorks OS 6.0 及以上、iOS、Windows Phone 7 和 8、以及 Tizen）的框架。

使用`localStorage`对象时存在一些缺点。由于`localStorage` API 是同步的，访问`localStorage`对象所需的时间大于访问内存中对象所需的时间。因此，应用可能看起来响应较慢。此外，如前所述，复杂数据需要序列化和反序列化。这个过程可能会进一步影响应用的响应速度。您可以使用 JavaScript **WebWorker**来避免任何性能下降，但支持取决于平台浏览器的实现，而不是 PhoneGap。

然而，这些缺点不应该阻止你使用 LocalStorage API，因为，就像大多数性能指标一样，当你连续多次执行相同的操作时，这些性能影响真的很重要。更具体地说，由于访问`localStorage`对象所需的时间导致的性能下降发生在不同的标签页/窗口访问同一对象时。

当基于 PhoneGap 构建的移动应用运行时，几乎不可能同时由其他标签页/窗口访问`localStorage`（只有当你的应用开始使用`UIWebView`中的多个`InAppBrowser`实例时，你才应该遇到问题）。

现在我们将创建一个示例应用，以便熟悉使用`localStorage`对象及其支持的所有方法。

# 行动时间 - 在 LocalStorage 上读取和写入数据

1.  打开命令行工具，使用您之前安装的 PhoneGap CLI 创建一个新的 PhoneGap 项目。这将在您的当前工作目录中创建一个名为`DeviceApi`的新目录；这可以通过以下命令完成：

    ```js
    $ phonegap create DeviceApi

    ```

1.  移动到您刚刚创建的目录：

    ```js
    $ cd DeviceApi

    ```

1.  在设备 API 上添加您想要测试的平台。例如，我们添加 Android 平台：

    ```js
    $ cordova platform add android

    ```

1.  删除`www`目录内除`index.html`之外的所有文件和子目录。打开您在`www`根目录中找到的`index.html`文件，并添加以下 HTML 代码片段：

    ```js
    <button type="button" onclick="addCountry()">Add Canada</button> <br/><br/>
    <button type="button" onclick="get1Data()">Get USA Data</button> <br/><br/>
    <button type="button" onclick="getAllData()">Get All Data</button> <br/><br/>
    <button type="button" onclick="removeAllData()">Remove All Countries</button> <br/><br/>
    ```

    此代码将添加四个按钮，这些按钮将添加新的`localStorage`数据，删除数据，检索数据和清除所有`localStorage`数据。请注意，每个按钮都有一个`onclick`事件，指向一个函数。

1.  现在让我们向页面添加所需的 JavaScript。首先，我们将`onDeviceReady`函数绑定到`deviceready`事件。当页面加载时，此函数将添加 3 个`localStorage`数据。使用`localStorage`对象的`setItem`方法来添加新的键值对。如下所示：

    ```js
    document.addEventListener("deviceready", onDeviceReady, false);

    function onDeviceReady() {
              localStorage.setItem("IN", "India");
              localStorage.setItem("FR", "France");
              localStorage.setItem("USA", "U.S.A");
    }
    ```

1.  当用户点击**添加加拿大**按钮时，我们需要向`localStorage`对象添加一个新的国家。我们将再次使用`setItem`来添加新项。完成后，我们将显示一个警告。请注意，您不能再次添加相同的键到存储中，如下所示：

    ```js
    function addCountry() {
       localStorage.setItem("CA", "Canada");
       alert("Canada added successfully");
    }
    ```

1.  现在，要获取存储中的特定对象，我们应该使用`getItem`方法。当存在匹配的键时，返回值。如果没有，则返回 null。在这个例子中，我们将获取`USA`键的值并在 alert 中显示它：

    ```js
    function get1Data() {
       var usa = localStorage.getItem("USA");
       alert("Country Name is " + usa);
    }
    ```

1.  要获取存储中的所有项，我们应该遍历`localStorage`对象，并逐个获取键，如下面的代码所示：

    ```js
    function getAllData() {

       for(var i in localStorage){
          alert(localStorage.getItem(i));
       }

    }
    ```

1.  `clear`方法用于清除应用中`localStorage`的所有值：

    ```js
    function removeAllData() {
       window.localStorage.clear();
       alert("All Countries removed successfully");
    }
    ```

如`removeAllData`函数所示，我们可以使用`window.localStorage`而不是`localStorage`来操作`localStorage`对象。

本例的完整代码如下提供供您参考：

```js
<!DOCTYPE html>
<html>
    <head>
        <!--Other section removed for sake of simplicity -->
        <title>Hello World</title>
    </head>
<body>
<button type="button" onclick="addCountry()">Add Canada</button> <br/><br/>
<button type="button" onclick="get1Data()">Get USA Data</button> <br/><br/>
<button type="button" onclick="getAllData()">Get All Data</button> <br/><br/>
<button type="button" onclick="removeAllData()">Remove All Countries</button> <br/><br/>

<script type="text/javascript" src="img/cordova.js"></script>
<script type="text/javascript">
document.addEventListener("deviceready", onDeviceReady, false);

function onDeviceReady() {
     localStorage.setItem("IN", "India");
     localStorage.setItem("FR", "France");
     localStorage.setItem("USA", "U.S.A");
}

function addCountry() {
     localStorage.setItem("CA", "Canada");
     alert("Canada added successfully");
}

function get1Data() {
      var usa = localStorage.getItem("USA");
      alert("Country Name is " + usa);
}

function getAllData() {
   for(var i in localStorage){
      alert(localStorage.getItem(i));
   }
}

function removeAllData() {
      window.localStorage.clear();
      alert("All Countries removed successfully");
}

</script>
</body>
</html>
```

## *刚才发生了什么？*

您使用 JavaScript 在设备上保存了持久数据，这些 JavaScript 也可以在所有主流桌面浏览器中运行。请注意，每个平台都会将数据存储在不同的位置，并且这些数据可能会被设备清除。根据平台的不同，这些数据可能在应用关闭或设备重启时被删除。因此，强烈建议不要使用`localStorage`对象来存储关键信息。

## 探索 PhoneGap SQL 存储

目前，移动设备上不同浏览器中的客户端存储实现相当不一致。在 PhoneGap 项目中工作时，对每个浏览器实现进行分析非常重要，因为应用是通过 WebView 渲染的；在 iOS 上，这是 Objective-C 的`UIWebView`类；在 Android 上，它是`android.webkit.WebView`，在其他所有支持的平台上也有所不同。WebView 只是暴露了底层平台浏览器。因此，了解目标平台移动浏览器的客户端存储选项是否受支持非常重要。

以下表格总结了在撰写本文时主要浏览器移动版本的存储支持；X 标志表示该功能是否受支持。

|   | Android 浏览器 | Firefox OS | iOS Safari | IE 11 移动版 | Chrome for Android |
| --- | --- | --- | --- | --- | --- |
| IndexedDB | X | X | X | X | X |
| Web SQL | X | --- | X | --- | X |

IndexedDB 是一个简单的平面文件数据库，具有分层键值持久性和基本索引。**Web SQL**基本上是浏览器中嵌入的**SQLite**。SQLite 是一个包含在用 C 编写的约 350 KB 的小型库中的关系数据库，被 Skype 或 Photoshop Lightroom 等软件使用。这些存储选项之间的主要区别在于，IndexedDB 是一个 NoSQL 数据库，允许您根据应用程序需求与 JavaScript 对象和索引一起工作，而 Web SQL 是一个真正的关系型客户端数据库实现。使用 Web SQL 的一个优点是您可以在后端和前端之间共享相同的查询。在运行一些测试时，您可以看到 Web SQL 可以有多快。

### 注意

为了在您的机器上运行相同的测试，您可以克隆位于[`github.com/scaljeri/indexeddb-vs-websql`](https://github.com/scaljeri/indexeddb-vs-websql)的 GitHub 仓库，并在您的网络浏览器中打开文件`test.html`。

W3C 于 2010 年 11 月 18 日停止了对 Web SQL 的支持，使 IndexedDB 成为*事实上的*标准。从开发者的角度来看，IndexedDB 可能看起来是一个巨大的倒退，但实际上并非如此。多年来，开发者使用键值对在客户端存储数据，并且大多数时候他们使用**非结构化查询语言**（**UQL**）或 NoSQL 来查询对象。因此，IndexedDB 应被视为客户端存储的自然演变。

PhoneGap 提供基于已弃用的 Web SQL 数据库规范的存储 API。当设备支持 Web SQL 时，应用程序将使用它。如果不支持，应用程序将使用 PhoneGap 的。对于开发者来说，将不会有任何区别，因为我们不需要更改任何代码行。

## 在 PhoneGap 中处理数据库存储

要在 PhoneGap 中使用`Database`对象，只需使用以下所示的`openDatabase`方法：

```js
var size = (1024 * 1024 * 2);
var db = window.openDatabase("name", "1.0", "Test DB", size);
```

`openDatabase`方法接受以下四个（自解释）参数：

+   数据库**名称**

+   数据库**版本**

+   数据库的**显示名称**

+   数据库的**估计大小**

请记住，应用程序可以查询数据库的版本号，以了解是否需要升级数据库模式。`openDatabase`方法返回当前打开数据库的引用。

在前面的代码片段中，我分配了 2MB 的字节空间。在分配空间时，请考虑移动设备可能对其支持的数据库大小的限制。例如，iOS 仅允许基于 Web 的应用程序使用最多 5MB；当使用 PhoneGap 作为包装器时也是如此。

返回的数据库对象公开了两个方法，这些方法接受一个到三个参数：`transaction()`和`readTransaction()`。主要区别在于`readTransaction()`方法必须用于只读模式。可以传递给这些方法的参数包括：

+   一个执行一个或多个 SQL 语句的函数

+   一个处理在打开数据库时由应用程序抛出的异常的函数

+   一个处理数据库成功打开的函数

注意，`transaction()`和`readTransaction()`异步方法是 PhoneGap 框架中唯一需要在成功处理函数之前想要失败处理函数的方法。此外，成功处理函数是唯一不接收任何参数的；其他处理函数接收一个`SQLTransaction`对象。

`SQLTransaction`对象公开了`executeSql`方法。使用此方法，可以运行多个 SQL 语句，并将一些参数传递给语句以处理成功的 SQL 查询执行和 SQL 错误。

### 注意

你可以使用多个标记来将参数传递给 SQL 语句。有关可用标记的完整概述，请参阅在线文档中的[`www.sqlite.org/lang_expr.html`](http://www.sqlite.org/lang_expr.html)。

成功处理程序接收两个参数：第一个是事务本身的引用，第二个是包含执行查询的信息和可选结果的 `SQLResultSet` 对象。

### 小贴士

当执行 `SELECT` 语句时，`SQLResultSet` 对象的 `insertId` 属性返回一个 `Exception: DOMException` 值。你可以安全地忽略它，因为它不会影响应用程序。

# 实践时间 - 填充本地数据库

为了巩固你刚刚学到的知识，你将创建一个新的本地数据库，向其中添加一个表，并写入和读取一些数据。

1.  返回到之前示例中创建的项目，或者创建一个新的项目。清空 `index.html` 文件的内容，并添加以下按钮标记到其中。此按钮将用于从数据库中查询数据：

    ```js
    <button type="button" onclick="queryEmployees()">Fetch All Rows</button>
    ```

1.  创建一个 JavaScript 标签和 `deviceready` 事件监听器：

    ```js
    document.addEventListener("deviceready", onDeviceReady, false);
    ```

1.  在 `onDeviceReady` 函数的主体中，我们将创建一个名为 `EMP` 的新数据库，版本号为 `1.0`，命名为 `Employee Details`，估计大小为 2 MB：

    ```js
    var size = (1024 * 1024 * 2);

    function onDeviceReady() {
       var database = window.openDatabase('employee', '1.0', 'Employee Details', size);
       database.transaction(populateDB, onError, onSuccess);
    }
    ```

    使用 `openDatabase` 方法创建数据库后，我们将填充数据库。我们必须为需要执行的每个操作创建一个事务，因此我们将 `populateDB` 方法作为事务的参数提供：

    ```js
    function populateDB(tx) {
       tx.executeSql('DROP TABLE IF EXISTS EMP');
       tx.executeSql('CREATE TABLE IF NOT EXISTS EMP (id unique, name)');
       tx.executeSql('INSERT INTO EMP (id, name) VALUES (1, "John Doe")');
       tx.executeSql('INSERT INTO EMP (id, name) VALUES (2, "Jane Doe")');
    }

    function onError(err) {
       alert("SQL Error: " + err.code);
    }

    function onSuccess() {
       alert("Success!");
    }
    ```

    `executeSql` 方法可以接受任何标准 SQL 语句并执行它。

1.  现在，我们将创建 `queryEmployees` 函数，该函数绑定到按钮的 `onclick` 事件。此函数将打开数据库并创建一个事务以查询数据库：

    ```js
    function queryEmployees() {
       var database = window.openDatabase('employee', '1.0', 'Employee Details', size);
       database.transaction(queryData, onError);
    }
    ```

1.  作为事务参数提供的 `queryData` 函数执行 SQL 查询。这需要事务参数 `tx`，并将结果集传递给 `onSelectSuccess` 函数：

    ```js
    function queryData(tx) {
       tx.executeSql('SELECT * FROM EMP', [], onSelectSuccess, onError);
    }
    ```

1.  `onSelectSuccess` 函数将事务和结果集作为参数，并执行实际的解析工作：

    ```js
    function onSelectSuccess(tx, results) {

       var len = results.rows.length;
       alert("Total Employees : " + len);

       for (var i=0; i<len; i++){
          alert(results.rows.item(i).id + " - "
                + results.rows.item(i).name);
       }
    }
    ```

本示例的完整代码提供给你快速参考。你还可以在浏览器开发者工具中验证数据库记录：

```js
<!DOCTYPE html>
<html>
    <head>
        <!--Other section removed for sake of simplicity -->
        <title>Database Example</title>
    </head>
<body>
   <button type="button" onclick="queryEmployees()">Fetch All Rows</button>
   <script type="text/javascript" src="img/cordova.js"></script>

   <script type="text/javascript">
      var size = (1024 * 1024 * 2);

      document.addEventListener("deviceready", onDeviceReady, false);

      function onDeviceReady() {
        var database = window.openDatabase('employee', '1.0', 'Employee Details', size);
        database.transaction(populateDB, onError, onSuccess);
      }

      function populateDB(tx) {
        tx.executeSql('DROP TABLE IF EXISTS EMP');
        tx.executeSql('CREATE TABLE IF NOT EXISTS EMP (id unique, name)');
        tx.executeSql('INSERT INTO EMP (id, name) VALUES (1, "John Doe")');
        tx.executeSql('INSERT INTO EMP (id, name) VALUES (2, "Jane Doe")');
      }

      function onError(err) {
        alert("SQL Error: " + err.code);
      }

      function onSuccess() {
        alert("Success!");
      }

      function queryEmployees() {
        var database = window.openDatabase('employee', '1.0', 'Employee Details', size);
        database.transaction(queryData, onError);
      }

      function queryData(tx) {
        tx.executeSql('SELECT * FROM EMP', [], onSelectSuccess, onError);
      }

      function onSelectSuccess(tx, results) {

        var len = results.rows.length;
        alert("Total Employees : " + len);

        for (var i=0; i<len; i++){
          alert(results.rows.item(i).id + " - " + results.rows.item(i).name);
        }
      }
   </script>
</body>
</html>
```

## *刚才发生了什么？*

你创建了一个本地数据库，用于存储和恢复与你的应用程序及其用户相关的信息。当应用程序被卸载时，数据库将自动删除。

# 数据库限制

在使用 PhoneGap 的 WebSQL 实现时，有一些限制需要注意（有关完整概述，请参阅 SQLite 文档中的[`www.sqlite.org/limits.html`](http://www.sqlite.org/limits.html)）。这些限制与框架本身无关，而是由于每个目标平台的 web 视图实现造成的。

在开发应用程序时，您很容易遇到的限制是数据库文件的大小限制。例如，在 WebKit 中，它取决于操作系统，从 5 MB 到 25 MB 不等。另一个您可能会遇到的限制是，自 iOS 5.1 以来，`localStorage`和 Web SQL 数据库已从`~/Library/WebKit`文件夹移动到`~/Library/Caches`文件夹。实际上，这个变化意味着存储的信息不再备份，并且当需要更多空间时，操作系统可以随意删除（有关 iOS 数据管理的更多信息，请参阅 Apple 开发者指南[`developer.apple.com/technologies/ios/data-management.html`](https://developer.apple.com/technologies/ios/data-management.html)）。

为了避免描述中的问题，您可以使用 GitHub 上可用的 Sqlite 插件，适用于 Android 和 iOS ([`github.com/litehelpers/Cordova-sqlite-storage`](https://github.com/litehelpers/Cordova-sqlite-storage))。

### 注意

您将在本书中了解更多关于 PhoneGap 插件的内容；目前，您只需要知道插件通常是 HML/CSS/JavaScript 和本地代码的组合，用于扩展 PhoneGap 的功能。

使用此插件时，您获得的主要优势是您可以保持 SQLite 数据库在已知并可重新配置的用户数据位置，没有更多的大小限制，并且可以使用**SQLcipher**对数据库进行加密（有关完整参考，请参阅在线文档[`sqlcipher.net/documentation/`](http://sqlcipher.net/documentation/)）。

从开发者的角度来看，API 没有变化，只是前缀有所不同；这意味着您不需要像这样通过`window`对象打开数据库：

```js
window.openDatabase();
```

您必须像这样引用插件：

```js
sqlitePlugin.openDatabase();
```

# 理解文件 API

PhoneGap 文件 API 是两个不同 W3C API 的实现，即目录和系统 API 以及文件 API（您可以在 W3C 网站上找到完整的规范[`www.w3.org/TR/file-system-api/`](http://www.w3.org/TR/file-system-api/)和[`www.w3.org/TR/file-upload`](http://www.w3.org/TR/file-upload)）。PhoneGap 文件 API 并不是 W3C 规范的完整实现；缺失的部分是同步文件系统接口的实现。异步 JavaScript API 使用起来稍微复杂一些，因为您必须与多个嵌套函数一起工作，但这不应成为大问题；实际上，这是所有网络开发者都非常熟悉的事情。

### 注意

异步与同步 JavaScript 执行的主要区别在于，在前者的情况下，你可以同时运行多个进程，从而避免用户界面“冻结”。随着 JavaScript 中 Web Workers 的引入，可以避免这个问题，但这完全超出了本书的范围；你可以在 Mozilla 网站上找到更多关于 Web Workers 的信息，网址为[`developer.mozilla.org/en-US/docs/DOM/Using_web_workers`](https://developer.mozilla.org/en-US/docs/DOM/Using_web_workers)。

为了访问设备文件系统，你可以使用`LocalFileSystem`对象的`requestFileSystem`方法；此对象的所有方法都在`window`对象中定义。该方法接受以下四个参数：

+   存储类型（临时或持久）

+   在设备存储上要分配的字节数量

+   成功处理程序

+   错误处理程序

当你想访问设备文件系统时，生成的代码看起来如下所示：

```js
window.requestFileSystem(/*storage*/, /*size*/, onSuccess, onError);
```

对于`storage`参数，你需要指定`LocalFileSystem`对象中定义的以下两个伪常量之一：

+   `LocalFileSystem.PERSISTENT`：这表示存储无法在没有应用或用户许可的情况下被用户代理删除。

+   `LocalFileSystem.TEMPORARY`：这表示存储在请求空间中的文件可以被用户代理或系统删除，无需应用或用户的许可

请求的沙盒存储大小以字节为单位表示；例如，为了使代码更易读，可以使用（4 x 1024 x 1024）的语法来分配 4 KB，而不是使用 4,194,304 字节的数字。

### 注意

设备硬盘并未完全对应用程序视图开放。硬盘的一部分仅被分配给单个应用；这就是应用的**沙盒**。应用沙盒背后的理念是每个应用只能访问其自己的沙盒以及操作系统拥有的某些高级目录。高级目录的结构取决于操作系统。

`onSuccess`处理程序接收一个`FileSystem`对象作为参数。为此对象定义的两个属性是`name`和`root`。访问对象的`name`属性可以读取文件系统的名称；访问`root`属性允许你获取应用沙盒的`root`目录的引用。这在此处显示：

```js
function onSuccess(fileSystem){

    console.log(fileSystem.name);
    var currentRoot = fileSystem.root;

}
```

`onError`处理程序接收一个`FileError`对象作为参数；此对象使用对象本身定义的几个伪常量来表示不同的错误，如下所示：

```js
function onError(fileError){

    console.log(fileError.code);

}
```

如果已知文件或目录的位置，可以使用`LocalFileSystem`对象的`resolveLocalFileSystemURI`方法来访问它。此方法接受以下三个参数：

+   文件或目录的 URI

+   成功处理程序（`onSuccess`）

+   错误处理程序（`onError`）

如果你想访问例如 Android 设备的外部存储，可以使用以下语法：

```js
window.resolveLocalFileSystemURI('file:///mnt/sdcard', onSuccess, onError);.
```

`onSuccess`函数接收一个`DirectoryEntry`或`FileEntry`对象作为参数，这取决于输入路径的类型（即目录或文件）；`onError`处理程序接收一个`FileError`对象作为参数。

`code`属性的值由以下伪常量总结：

+   `FileError.NOT_FOUND_ERR`（返回值`1`）：这意味着应用所需的文件或目录无法找到

+   `FileError.SECURITY_ERR`（返回值`2`）：这意味着文件或目录位于应用沙盒之外，或者应用没有访问它的权限

+   `FileError.ABORT_ERR`（返回值`3`）：这是在调用读取器或写入器的`abort`方法时抛出的

+   `FileError.NOT_READABLE_ERR`（返回值`4`）：这意味着应用所需的文件或目录无法读取

+   `FileError.ENCODING_ERR`（返回值`5`）：这意味着在`LocalFileSystem`对象的`resolveLocalFileSystemURI`方法中用作参数的路径或本地 URI 格式不正确

+   `FileError.NO_MODIFICATION_ALLOWED_ERR`（返回值`6`）：这意味着应用尝试写入一个由于文件系统的实际状态而无法修改的文件或目录

+   `FileError.INVALID_STATE_ERR`（返回值`7`）：这意味着应用访问了一个由另一个进程使用的文件

+   `FileError.SYNTAX_ERR`（返回值`8`）：这是不言自明的；这是由于语法错误导致的

+   `FileError.INVALID_MODIFICATION_ERR`（返回值`9`）：这意味着应用请求的修改无效；此类错误的例子是将目录移动到其自身的子目录中

+   `FileError.QUOTA_EXCEEDED_ERR`（返回值`10`）：这意味着应用请求的存储量超过了允许的存储配额

+   `FileError.TYPE_MISMATCH_ERR`（返回值`11`）：这意味着应用尝试访问一个文件或目录，但条目不是预期的类型（即返回的是目录而不是文件）

+   `FileError.PATH_EXISTS_ERR`（返回值`12`）：这意味着由于存在具有相同路径的文件或目录，应用未能创建文件或目录

## 读取目录和文件

只有在获得对文件系统的访问权限后，才能读取设备目录、子目录和内容。再次强调，用作`requestFileSystem`方法参数的`onSuccess`处理程序接收一个`FileSystem`对象。通过此对象的`root`属性，可以访问一个`DirectoryEntry`对象，然后创建一个能够读取当前目录中所有条目的`DirectoryReader`对象。这在此处展示：

```js
function onSuccess(fileSystem){

   var currentRoot = fileSystem.root;
   var reader = currentRoot.createReader();

}
```

`DirectoryReader` 对象公开了一个名为 `readEntries` 的方法。它可以用来读取条目，由于文件 API 的异步特性，它接受一个成功处理程序和一个失败处理程序。类似于 `resolveLocalFileSystemURI` 方法所发生的情况，成功处理程序根据所列项目的类型（即目录或文件）接收一个 `DirectoryEntry` 或 `FileEntry` 对象的数组。

# 行动时间 - 列出文件夹

准备探索设备的持久存储文件夹。使用以下步骤：

1.  打开命令行工具，使用您之前安装的 Cordova CLI 工具创建一个新的项目。这将在一个名为 `FileSystem` 的新目录中创建一个新目录，位于您当前的工作目录中：

    ```js
    $ cordova create FileSystem

    ```

1.  移动到您刚刚创建的目录：

    ```js
    $ cd FileSystem

    ```

1.  在设备 API 上添加您想要测试的平台。例如，我们添加了 Android 平台：

    ```js
    $ cordova platform add android

    ```

1.  使用以下命令添加 File API 插件：

    ```js
    $ cordova plugin add cordova-plugin-file

    ```

1.  前往 `www` 文件夹，打开 `index.html` 文件，并在应用程序的主体中添加一个具有 `id` 值为 `fileslist` 的 `div` 元素：

    ```js
    <div id="fileslist"></div>
    ```

1.  我们现在将在 JavaScript 部分添加一个 `deviceready` 事件监听器。`onDeviceReady` 函数必须在 `deviceready` 事件触发后调用：

    ```js
    document.addEventListener("deviceready", onDeviceReady, false);
    ```

1.  在 `onDeviceReady` 函数的主体中，请求访问设备文件系统，指定您将要定义的成功和失败处理程序，并请求 0 KB 的持久存储；您只需要在写入设备文件系统时指定配额：

    ```js
    function onDeviceReady() {
       var size = 0;
       window.requestFileSystem(LocalFileSystem.PERSISTENT, size, onFileSystemSuccess, onFileSysError);
    }
    ```

1.  定义错误处理程序，当代码抛出错误时，它会通知您：

    ```js
    function onFileSysError(error){
       alert("Error Code: " + error.code);
    }
    ```

1.  定义成功处理程序，并在其主体内部创建一个新的 `DirectoryReader` 对象，并使用此对象来读取所有目录内容。我们需要传递一个新的函数 `parseDirectories`，该函数将实际遍历目录列表。目录条目将自动作为参数传递给此函数：

    ```js
    function onFileSystemSuccess(fileSystem){
       var reader = fileSystem.root.createReader();
       reader.readEntries(parseDirectories,onFileSysError);
    }
    ```

1.  定义 `parseDirectories` 函数，并将其以下片段添加到其中：

    ```js
    function parseDirectories(entries){
        var str = '<ul>';

        for (var i = 0, len = entries.length; i < len; i++) {
            if (entries[i].isDirectory) {
                 str += '<li>' + entries[i].fullPath + '</li>';
            }
        }

        str += "</ul>";

        document.getElementById("fileslist").innerHTML+= str;

    }
    ```

    我们读取条目，并在列表中的每个条目上检查它是否是目录或文件。如果是目录，我们将目录名称追加到无序列表，如果是文件则忽略它。最后，我们将生成的无序列表（`ul`）添加到我们之前创建的名为 `fileslist` 的 `div` 中。

    当此代码在实际设备上运行时，它将列出持久存储中的所有目录。

本例的整个源代码如下所示：

```js
<!DOCTYPE html>

<html>
    <head>
        <!--Other section removed for sake of simplicity -->
        <title>File Example</title>
    </head>

<body>

<div id="fileslist"></div>

<script type="text/javascript" src="img/cordova.js"></script>

<script type="text/javascript">

document.addEventListener("deviceready", onDeviceReady, false);

function onDeviceReady() {
   var size = 0;
   window.requestFileSystem(LocalFileSystem.PERSISTENT, size, onFileSystemSuccess, onFileSysError);
}

function onFileSysError(error){
   alert("Error Code: " + error.code);
}

function onFileSystemSuccess(fileSystem){
   var reader = fileSystem.root.createReader();
   reader.readEntries(parseDirectories,onFileSysError);
}

function parseDirectories(entries){
   var str = '<ul>';

   for (var i = 0, len = entries.length; i < len; i++) {
        if (entries[i].isDirectory) {
              str += '<li>' + entries[i].fullPath + '</li>';
        }
   }

   str += "</ul>";

   document.getElementById("fileslist").innerHTML+= str;

}

</script>

</body>
</html>
```

## *发生了什么？*

应用程序现在可以使用异步的 Files API 从设备的持久存储中读取目录列表。

## 写入和读取文件数据

要将数据写入文件，应用程序只需使用 `FileWriter` 对象访问文件即可。为了获取 `FileWriter` 对象，您首先必须使用 `LocalFileSystem` 对象的 `requestFileSystem` 方法访问 `DirectoryEntry` 对象或 `FileEntry` 对象。

一旦成功访问文件系统，你可以请求一个文件，指定你想要使用`create`标志创建它：

```js
Function onFileSystemSuccess(fileSystem){

    var root = fileSystem.root;
    root.getFile('data.txt', {create: true}, 
                  onGetFile, onGetFileError);

}
```

### 注意

文件 API 中有两个标志可以作为`getFile`和`getDirectory`方法的参数使用：`create`和`exclusive`。`create`标志用于指示应创建文件或目录；当`create`标志设置为`true`时，`exclusive`标志才会生效，并且如果文件或目录已存在，它会导致创建失败。

与其他 Files API 一样，`getFile`方法也是异步的，需要成功和失败处理程序。一旦在成功处理程序中，就可以使用接收到的`FileEntry`对象的`createWriter`方法创建一个`FileWriter`对象。`createWriter`方法也需要成功和失败处理程序，如下所示：

```js
function onGetFile(file){

    file.createWriter(onGetWriter, onGetWriterError);

}
```

再次强调，你有另外两个处理程序，这意味着只有在你调用了三个回调函数之后，你才能将一些内容写入你刚刚创建的文件：

```js
function onGetWriter(writer){

    writer.write('Hello PhoneGap Files API!');

}
```

### 小贴士

在 PhoneGap 中使用`FileWriter`对象无法从 JavaScript 写入二进制数据；这是框架的限制，因为它将数据作为字符串在原生和 JavaScript 层之间传递。一个可能的解决方案是编写一个插件，将 Base64 字符串转换为二进制数据。

当执行写入操作时，会发生几个事件。对于每个此类事件，`FileWriter`对象上都有一个相应的属性：

+   当`FileWriter`对象开始写入文件时，会调用`onwritestart`事件；它接收一个`ProgressEvent`对象作为参数。

+   当`FileWriter`对象成功完成写入操作时，会调用`onwrite`事件；它接收一个`ProgressEvent`对象作为参数。

+   当通过调用`FileWriter`的`abort`方法中断写入操作时，会调用`Onabort`事件；它接收一个`ProgressEvent`对象作为参数。

+   当写入操作失败时，会调用`Onerror`事件；它接收一个`ProgressEvent`对象作为参数。为了了解错误发生的原因，你可以访问存储在`event`对象的`target.error`属性中的`FileError`对象。

`FileWriter`对象还包含其他属性。要获取完整概述，请参考可在[`docs.phonegap.com/en/edge/cordova_file_file.md.html#FileWriter`](http://docs.phonegap.com/en/edge/cordova_file_file.md.html#FileWriter)找到的在线指南。

使用`ProgressEvent`对象，你可以通过`loaded`、`total`和`type`属性访问已加载的字节、总字节和事件的性质（即`abort`、`writeend`等）：

```js
function onGetWriter(writer){

    writer.onwrite = function(evt){

        console.log(evt.loaded, evt.total, evt.type);

    }

    writer.write('Hello PhoneGap Files API!');

}
```

### 小贴士

如果你尝试依次调用`FileWriter`对象的`write`方法，只有第一个字符串将被添加到文件中。你必须等待`writeend`事件触发，才能将其他数据写入文件。

当你想读取文件时，你可以使用一个`FileReader`对象。此对象与`FileWriter`对象类似。使用它时，会发生几个事件：`FileWriter`对象的`onabort`和`onerror`属性与`FileReader`的类似。仅与`FileReader`对象相关的属性如下：

+   `onloadstart`: 当`FileReader`对象开始读取文件时，存储在此属性的函数将被调用；它接收一个`ProgressEvent`对象作为参数。

+   `onload`: 当读取操作成功完成后，存储在此属性的函数将被调用；它接收一个`ProgressEvent`对象作为参数。

+   `onloadend`: 当读取操作完成时（无论成功与否），存储在此属性的函数将被调用；它接收一个`ProgressEvent`对象作为参数。

`FileReader`对象允许你以以下四种不同的方式读取文件数据：

+   `readAsDataURL`: 这将读取文件，并将指定文件的 内容作为 Base64 编码的数据 URL 返回

+   `readAsText`: 这将读取文件，并以默认 UTF-8 编码将数据作为字符串返回

+   `readAsBinaryString`: 这将以二进制形式读取文件，并将数据作为二进制字符串返回

+   `readAsArrayBuffer`: 这将读取文件，并将数据作为`ArrayBuffer`返回

为了将你刚刚学到的知识付诸实践，现在你将看到如何解析设备的持久存储，恢复第一个可用的图像，并在 app 的 web 视图中渲染它。

这是将文本文件写入手机存储的完整源代码：

```js
<!DOCTYPE html>
<html>
    <head>
        <!--Other section removed for sake of simplicity -->
        <title>File Example</title>
    </head>
<body>
<script type="text/javascript" src="img/cordova.js"></script>

<script type="text/javascript">

document.addEventListener("deviceready", onDeviceReady, false);

function onDeviceReady() {
   var size = 0;
   window.requestFileSystem(LocalFileSystem.PERSISTENT, size, onFileSystemSuccess, onFileSysError);
}

function onFileSystemSuccess(fileSystem){
   var root = fileSystem.root;
   root.getFile('data.txt', {create: true}, onGetFile, onFileSysError);
}

function onGetFile(file){
   file.createWriter(onGetWriter, onFileSysError);
}

function onGetWriter(writer){

   writer.onwrite = function(evt){
      console.log(evt.loaded, evt.total, evt.type);
   }

   writer.write('Hello PhoneGap Files API!');
}

function onFileSysError(error){
   alert("Error Code: " + error.code);
}

</script>
</body>
</html>
```

# 行动时间 - 读取和渲染图像

准备将设备存储中第一个可用的图像渲染到 PhoneGap 默认 app 模板中。请参考以下步骤：

1.  打开命令行工具，创建一个名为`ReadingFile`的新 PhoneGap 项目。

1.  使用以下命令行添加 File API 插件：

    ```js
    $ cordova plugin add cordova-plugin-file

    ```

1.  进入`www`文件夹，打开`index.html`文件，在紧随`deviceready`之后的 app 主`div`内添加一个`img`标签，其`id`值为`firstImage`：

    ```js
    <img id='firstImage' />
    ```

1.  进入`www/js`文件夹，打开`index.js`文件，并定义一个名为`requestFileSystem`的新函数：

    ```js
    Function requestFileSystem() {
        // The request of access to the file system will go here

    }
    ```

1.  定义错误处理程序以获取每个可能错误的代码：

    ```js
    Function onError(error){

         alert(error.code);

    }
    ```

1.  在`requestFileSystem`函数的主体中，使用`LocalFileSystem`对象的`requestFileSystem`函数访问设备文件系统，定义成功和失败处理程序，并在成功处理程序中访问`root`文件系统：

    ```js
    window.requestFileSystem(LocalFileSystem.PERSISTENT, 0,

                       onFileSystemSuccess, onError);
    ```

    ### 提示

    你请求的是`0`字节的配额，因为你只是读取文件；你只需要在写入设备文件系统时指定配额。

1.  一旦你获得对`root`文件系统的访问权限，你可以在成功处理程序中创建一个`DirectoryReader`对象，并开始使用该对象的`readEntries`异步方法探索根文件系统：

    ```js
    var root = fileSystem.root;
    var reader = root.createReader();

    reader.readEntries(function(entries){

        for (var i = 0, entry; entry = entries[i]; i++){

            // Here The logic to check if the file is an image

        }

    }, onError);
    ```

1.  为了在`for`循环中确定一个文件是否是图片，你可以首先检查条目的`isFile`属性，然后使用一个简单的正则表达式；当条件满足时，使用根`DirectoryEntry`对象的`getFile`方法访问文件，指定成功和失败处理程序：

    ```js
    if (entry.isFile && (/\.(gif|jpg|jpeg|png)$/i).test(entry.name)){

        root.getFile(entry.name, {create: false}, 
                     onGetFile, onError);
        break;

    }
    ```

1.  在 JavaScript 部分，定义`onGetFile`函数，并在其主体中，通过使用`FileEntry`对象的`file`方法访问实际的文件。一旦你获得了对文件的访问权限，指定`onload`和`onerror`处理程序，并使用`readAsDataURL`方法读取文件，以便将结果分配给`img`标签的`src`属性，如下所示：

    ```js
    function onGetFile(fileEntry){

        fileEntry.file(function(file){

            var reader = new FileReader();

            reader.onload = function(evt){

                var img = document.querySelector('#firstImage');
                img.src = evt.target.result;

            };

            reader.onerror = function(evt){

                alert(evt.target.error.code, null);

            };

            reader.readAsDataURL(file);

        }, onError);

    }
    ```

现在在一个真实设备上测试项目。查看以下提供的完整代码，以便快速审查：

```js
<!DOCTYPE html>
<html>
    <head>
        <!--Other section removed for sake of simplicity -->
        <title>File Example</title>
    </head>
<body>

<img id="firstImage" />

<script type="text/javascript" src="img/cordova.js"></script>

<script type="text/javascript">

document.addEventListener("deviceready", onDeviceReady, false);

function onDeviceReady() {
   var size = 0;
   window.requestFileSystem(LocalFileSystem.PERSISTENT, size, onFileSystemSuccess, onError);
}

function onFileSystemSuccess(fileSystem){
   var root = fileSystem.root;
   var reader = root.createReader();

   reader.readEntries(function(entries){
      for (var i = 0, entry; entry = entries[i]; i++){
        if (entry.isFile &&
        (/\.(gif|jpg|jpeg|png)$/i).test(entry.name)){
          root.getFile(entry.name, {create: false}, onGetFile, onError);
          break;
        }
      }
   }, onError);

}

function onGetFile(fileEntry){

   fileEntry.file(function(file){
      var freader = new FileReader();

      freader.onload = function(evt){
        var img = document.querySelector('#firstImage');
        img.src = evt.target.result;
      };

      freader.onerror = function(evt){
        alert(evt.target.error.code);
      };

      freader.readAsDataURL(file);
   }, onError);

}

function onError(error){
   alert("Error Code: " + error.code);
}
</script>
</body>
</html>
```

## *刚才发生了什么？*

你已经探索了设备的文件系统，并在你的应用中将找到的第一张图片作为 Base64 数据流渲染。现在，你对 File API 有了一定的了解，是时候学习如何从设备传输文件到设备了。

## 文件传输

PhoneGap File API 还包括`FileTransfer`对象。正如其名所示，这个对象允许你开发应用程序，通过互联网下载和上传文件。`FileTransfer`对象公开的方法是自解释的：`upload`、`download`和`abort`。

`upload`方法接受多个参数：设备上文件的路径、接收文件的 URL、成功和失败处理程序、选项对象以及一个布尔值，用于强制方法接受所有安全证书。（我在下一个片段中省略了布尔值，因为在生产中使用它不被推荐；应用程序应该只接受它被设计用来处理的协议。）

```js
var fileTransfer = new FileTransfer();
fileTransfer.upload(fileURI, URL, onSuccess, onError, options);
```

`options`参数是一个`FileUploadOptions`对象。该对象允许你使用以下属性提供额外的信息：

+   `chunkedMode`: 这是一个布尔值，表示是否在不进行内部缓冲的情况下执行 HTTP 请求的流式传输。（对于分块传输编码的更详细描述，你可以参考[`en.wikipedia.org/wiki/Chunked_transfer_encoding`](http://en.wikipedia.org/wiki/Chunked_transfer_encoding)）。

+   `fileKey`: 这是一个字符串，表示文件上传到服务器上表单元素下的名称；默认值是`file`。

+   `fileName`: 这是一个表示上传文件名称的字符串；默认值是`image.jpg`。

+   `mimeType`: 这是一个表示将要上传的文件 MIME 类型的字符串；默认值是`image/jpg`。

+   `params`: 这是一个对象，表示要包含在 HTTP 请求头中的键值对。

`onSuccess`处理程序接收一个`FileEntry`对象作为参数，这样你就可以立即访问信息，例如设备上的文件名和完整路径。`onError`处理程序接收一个`FileTransferError`对象；该对象的属性如下：

+   `code`：这是一个表示存储在`FileTransferError`伪常量中的四个可能错误代码之一的数字（即`FILE_NOT_FOUND_ERR`、`INVALID_URL_ERR`、`CONNECTION_ERR`和`ABORT_ERR`）。

+   `source`：这是一个表示源文件 URI 的字符串。

+   `target`：这是一个表示目标文件 URI 的字符串。

+   `http_status`：这是一个表示 HTTP 状态码的数字。

`download`方法的工作方式类似；唯一的区别是前两个参数被交换，分别是：下载文件的 URL 和系统 URI（即路径），以便在设备上存储；此外，`options`参数只接受 HTTP 头。如下所示：

```js
var fileTransfer = new FileTransfer();
fileTransfer.download(URL, filePath, onSuccess, onError, options);
```

可以使用`abort`方法停止下载或上传操作，一旦调用`onError`处理程序，其参数的`code`属性值是伪常量，`FileTransferError.ABORT_ERR`。

### 注意

此外，当你设置错误的文件下载路径时，`FileTransferError`对象的`code`属性等于`FileTransferError.FILE_NOT_FOUND_ERR`（即值`1`）。

在`FileTransfer`对象中只定义了`onprogress`属性。正如其名所示，此属性用于存储一个函数，每当从或向设备传输数据块时都会调用该函数。

接下来，为了将你刚刚学到的知识付诸实践，你将下载一个文件，显示下载进度，并在下载完成后添加一个文件链接。

# 行动时间 - 下载并保存文件

准备下载一个文件并在 PhoneGap 默认应用模板中显示进度条和文件链接。请参考以下步骤：

1.  打开命令行工具并创建一个名为`DownloadFile`的新 PhoneGap 项目：

    ```js
    $ cordova create DownloadFile

    ```

1.  切换到`DownloadFile`目录：

    ```js
    $ cd DownloadFile

    ```

1.  使用以下命令添加 File 和 FileTransfer API 插件：

    ```js
    $ cordova plugin add cordova-plugin-file-transfer
    $ cordova plugin add cordova-plugin-file

    ```

1.  前往`www`文件夹，打开`index.html`文件，在`deviceready`标签下面的应用主`div`元素内添加一个带有`progress`值的`progress`标签；将`value`属性赋值为`1`，将`max`属性赋值为`100`：

    ```js
    <progress id='progress' value='1' max='100'></progress>
    ```

1.  定义一个名为`onDeviceReady`的新 JavaScript 函数，并将其添加到`deviceready`监听器中：

    ```js
    document.addEventListener("deviceready", onDeviceReady, false);

       function onDeviceReady() {
          var size = 0;
          window.requestFileSystem(LocalFileSystem.PERSISTENT, size, onFileSystemSuccess, onError);
       }
    ```

1.  对于`onFileSystemSuccess`方法，一旦你获得对文件系统的访问权限，创建一个新的`FileTransfer`对象，并调用`download`方法，指定远程 URL、系统根 URI 以及成功和失败处理程序：

    ```js
    function onFileSystemSuccess(fileSystem){
       var fileTransfer = new FileTransfer();
       var url = 'http://s3.amazonaws.com/mislav/Dive+into+HTML5.pdf';

       fileTransfer.download(encodeURI(url),
               fileSystem.root.toURL() + '/' + 'html5.pdf',
               fileDownloaded,
               onError);

       fileTransfer.onprogress = function(evt){

          if (evt.lengthComputable){

            var tot = (evt.loaded / evt.total) * 100;

            var element = document.querySelector('#progress');
            element.value = Math.round(tot);
          }
       }
    }
    ```

1.  定义成功和失败事件回调函数。对于任何错误，我们将以 JSON 字符串的形式将整个`error`对象警报整个系统，如下所示：

    ```js
    function fileDownloaded(entry){
       alert("Success");
    }

    function onError(error){
       alert(JSON.stringify(error));
    }
    ```

1.  现在，在真实设备上运行你的项目。文件将被下载，进度将在进度条中显示。一旦完成，你可以在设备上验证下载的文件。

本例的完整源代码如下所示：

```js
<!DOCTYPE html>
<html>
    <head>
        <!--Other section removed for sake of simplicity -->
        <title>File Example</title>
    </head>
    <body>

<progress id='progress' value='1' max='100'></progress>

<script type="text/javascript" src="img/cordova.js"></script>

<script type="text/javascript">

   document.addEventListener("deviceready", onDeviceReady, false);

   function onDeviceReady() {
      var size = 0;
      window.requestFileSystem(LocalFileSystem.PERSISTENT, size, onFileSystemSuccess, onError);
}

      function onFileSystemSuccess(fileSystem){
         var fileTransfer = new FileTransfer();
         var url = 'http://s3.amazonaws.com/mislav/Dive+into+HTML5.pdf';

         fileTransfer.download(encodeURI(url),
               fileSystem.root.toURL() + '/' + 'html5.pdf',
                 fileDownloaded,
                 onError);

         fileTransfer.onprogress = function(evt){

            if (evt.lengthComputable){

            var tot = (evt.loaded / evt.total) * 100;

            var element = document.querySelector('#progress');
            element.value = Math.round(tot);
         }
      }
   }

   function fileDownloaded(entry){
      alert("Success");
   }

   function onError(error){
      alert(JSON.stringify(error));
   }

</script>
</body>
</html>
```

## *发生了什么？*

你启动了一个文件下载，显示了一个进度条，并渲染了一个指向文件的链接。你将注意到一个问题，因为大多数平台在 WebView 中不提供 PDF 阅读器。简而言之，用户既无法在应用中也无法在外部浏览器中阅读该文件。为了打开一个原生应用来阅读文件，你必须使用一个外部插件。你将在下一章中了解到如何在你的应用中集成插件以及如何解决这个问题。

# 摘要

在本章中，你学习了如何在设备上保存数据以及如何处理最常见的限制。你还学习了 Files API 的工作原理，并了解了它的功能。

在下一章中，你将学习如何使用 Contact API 来处理设备中的联系人以及 Camera API，以便使用设备摄像头捕捉图像。
