# 第十章：JSON 在移动平台上的应用

当今的移动应用程序非常流行——像平板电脑和智能手机这样的设备在世界许多地方都超过了 PC 的销量。这些设备由 iOS 和 Android 等平台提供支持，并包括用于创建和解析 JSON 的 API，作为平台的一部分，使得作为应用程序开发者的你的生活变得稍微容易一些。在本章中，有你需要的食谱：

+   在 Android 上解析 JSON

+   在 Android 上生成 JSON

+   使用 Objective-C 在 iOS 上解析 JSON

+   使用 Objective-C 在 iOS 上生成 JSON

+   使用 Swift 在 iOS 上解析 JSON

+   使用 Swift 在 iOS 上生成 JSON

+   使用 Qt 解析 JSON

+   使用 Qt 生成 JSON

# 简介

正如我们在前一章中讨论的那样，JSON 是一种出色的媒介，用于与 Web 服务和服务器通信，无论客户端是 Web 应用程序还是传统应用程序。这对于移动应用程序尤其正确，许多移动应用程序运行在低带宽广域网上，在这种情况下，JSON 与 XML 相比的简洁性使得整体数据负载更小，从而确保远程查询的响应时间更快。

当今领先的移动平台是 Android 和 iOS。Android 运行 Linux 的一个变体，支持用 Java 进行软件开发，并在`org.json`命名空间中包括一个 JSON 处理器。iOS 从 Mach 和 BSD 中衍生而来，支持使用 Objective-C、Swift、C 和 C++进行软件开发，尽管对于大多数应用程序开发，你使用 Objective-C 或 Swift，每种语言都包含对`NSJSONSerialization`类的绑定，该类实现 JSON 解析和 JSON 序列化。

移动开发者的另一个选择是使用跨平台工具包，如 Qt，进行应用程序开发。Qt 在多种平台上运行，包括 Android、iOS 和 BlackBerry。Qt 定义了`QJsonDocument`和`QJsonObject`类，你可以使用它们来实现映射和 JSON 之间的相互转换。Qt 是一个存在已久的开源框架，不仅运行在移动平台上，还运行在 Mac OS X、Windows 和 Linux 上，以及其他许多平台。

我们接下来要讨论的 JSON 与前几章使用的类似，并像这样看起来像一个文档：

```js
{
  'call': 'kf6gpe-7',
  'lat': 37.40150,
  'lng': -122.03683
  'result': 'ok'
}
```

在接下来的讨论中，我假设你已经正确地为目标平台设置了软件开发环境。描述为 Android、iOS 和 Qt 设置软件环境的流程会比这本书允许的空间要大。如果你对为特定的移动平台开发软件感兴趣，你可能想要查阅 Android 或 iOS 的开发者资源：

+   你可以访问苹果公司为 iOS 开发者提供的开发者网站[`developer.apple.com`](https://developer.apple.com)。

+   你可以访问谷歌公司为 Android 开发者提供的开发者网站[`developer.android.com/index.html`](http://developer.android.com/index.html)。

+   你可以访问[`www.qt.io`](http://www.qt.io)获取关于 Qt 的信息。

# 在 Android 上解析 JSON

Android 提供了 `JSONObject` 类，该类让你通过一个与地图概念上相似的接口来表示 JSON 文档的键值对，并包括通过访问器方法进行序列化和反序列化，这些方法可以访问 JSON 对象的命名字段。

## 如何做到…

你首先通过用你想要解析的 JSON 初始化 `JSONObject`，然后使用其各种 `get` 方法来获取 JSON 字段的值：

```js
Import org.json.JSONObject;

String json = "…";
JSONObject data = new JSONObject(data);

String call = data.getString("call");
double lat = data.getDouble("lat");
double lng = data.getDouble("lng");
```

## 它是如何工作的…

`JSONObject` 构造函数接收要解析的 JSON，并提供访问器方法来访问 JSON 字段。在这里，我们使用 `getString` 和 `getDouble` 访问器分别访问 JSON 的 `call`、`lat` 和 `lng` 字段。

`JSONObject` 类定义了以下访问器：

+   `get` 方法，返回一个子类 `java.lang.Object`，其中包含命名槽中的值。

+   `getBoolean` 方法，如果槽中包含 `Boolean` 类型数据，则返回一个 `Boolean` 类型数据。

+   `getDouble` 方法，如果槽中包含 `double` 类型数据，则返回一个 `double` 类型数据。

+   `getInt` 方法，如果槽中包含 `int` 类型数据，则返回一个 `int` 类型数据。

+   `getJSONArray` 方法，如果槽中包含数组，则返回一个 `JSONArray` 实例，这是一个处理数组的 JSON 解析类。

+   `getJSONObject` 方法，如果槽中包含另一个映射，则返回一个 `JSONObject` 实例。

+   `getLong` 方法，如果槽中包含 `long` 类型数据，则返回一个 `long` 类型数据。

+   `getString` 方法，如果槽中包含 `String` 类型数据，则返回一个 `String` 类型数据。

该类还定义了 `has` 和 `isNull` 方法。这些方法接收一个槽的名字，如果字段中有值，或者没有该字段名或者值为 `null`，则返回 `true`。

`JSONArray` 与 `JSONObject` 类似，只不过它处理数组而不是映射。它有相同的访问器方法，这些方法采用集合中的整数索引，返回对象、布尔值、字符串、数字等等。

## 还有更多内容…

`JSONObject` 类还定义了 `keys` 方法，该方法返回 JSON 中的键的 `Iterator<String>`。你还可以通过调用 `names` 获得 JSON 中的名称的 `JSONArray`，或者通过调用 `length` 获得 JSON 中的键值对的数量。

## 也见

关于 `JSONObject` 的更多信息，请参阅 Android 文档中的 [`developer.android.com/reference/org/json/JSONObject.html`](http://developer.android.com/reference/org/json/JSONObject.html)。关于 `JSONArray` 的更多信息，请参阅 [`developer.android.com/reference/org/json/JSONArray.html`](http://developer.android.com/reference/org/json/JSONArray.html)。

# 在 Android 上生成 JSON

`JSONObject` 类还支持设置器方法来初始化 JSON 映射中的数据。利用这些方法，你可以将数据分配给一个 JSON 对象，然后通过调用其 `toString` 方法来获取 JSON 表示形式。

## 如何做到…

下面是一个简单示例：

```js
import org.JSON.JSONObject;

JSONObject data = new JSONObject();
data.put("call", "kf6gpe-7");
data.put("lat", 37.40150);
data.put("lng", -122.03683);
String json = data.toString();
```

## 它是如何工作的…

多态的 put 方法可以接受整数、长整数、对象、布尔值或双精度浮点数，将你命名的槽分配给你指定的值。

`JSONObject`类定义了`toString`方法，它接受一个可选的空格数来缩进嵌套结构以进行美观的 JSON 打印。如果你不传递这个缩进，或者传递 0，实现尽可能地将 JSON 编码成最紧凑的形式。

## 还有更多…

还有一个`putOpt`方法，它接受任何`Object`的子类，如果名称和非空值都不为空，则将值放入名称中。

你可以通过传递`JSONArray`或将另一个`JSONObject`作为要设置的值来将槽分配给值数组。`JSONArray`定义了一个类似的 put 方法，它以数组中的整数索引作为第一个参数，而不是槽名。例如，使用前面示例中的数据对象，我可以使用以下代码添加一个在站点的测量电压数组（也许来自无线电的电池）：

```js
import org.JSON.JSONObject;

JSONArray voltages = new JSONArray();
voltages.put(3.1);
voltages.put(3.2);
voltages.put(2.8);
voltages.put(2.6);
data.put("voltages", voltages);
```

你也可以直接传递`java.util.Collection`和`java.util.Map`实例，而不是传递`JSONArray`或`JSONObject`实例。之前的代码也可以写成：

```js
import org.JSON.JSONObject;
import org.JSON.JSONArray;
import java.util.Collection;

Collection<double> voltages = new Collection<double>();
voltages.put(3.1);
voltages.put(3.2);
voltages.put(2.8);
voltages.put(2.6);
data.put("voltages", voltages);
```

这使得在构建更复杂的 JSON 对象时稍微容易一些，因为你不必将每个 Java 集合或映射包装在一个相应的 JSON 对象中。

## 另见

关于`JSONObject`的更多信息，请参阅 Android 文档中的[`developer.android.com/reference/org/json/JSONObject.html`](http://developer.android.com/reference/org/json/JSONObject.html)。关于`JSONArray`的更多信息，请参阅[`developer.android.com/reference/org/json/JSONArray.html`](http://developer.android.com/reference/org/json/JSONArray.html)。

# 在 iOS 上用 Objective-C 解析 JSON

对象 C 的类库定义了`NSJSONSerialization`类，它可以进行 JSON 的序列化和反序列化。它将 JSON 转换为`NSDictionary`对象的值，这些对象的键是 JSON 中的槽名，值是它们的 JSON 值。它在 iOS 5.0 及以后的版本中可用。

## 如何做到…

这是一个简单的例子：

```js
NSError* error;
NSDictionary* data = [ NSJSONSerialization
  JSONObjectWithData: json
  options: kNilOptions
  error: &error ];

NSString* call = [ data ObjectForKey: @"call" ];
```

## 它是如何工作的…

`NSJSONSerialization`类有一个方法`JSONObjectWithData:options:error`，它接受一个`NSString`、解析选项和一个记录错误的地方，并执行 JSON 解析。它可以接受顶层为数组或字典的 JSON，分别返回`NSArray`或`NSDictionary`结果。所有值必须是`NSString`、`NSNumber`、`NSArray`、`NSDictionary`或`NSNull`的实例。如果顶层对象是数组，该方法返回`NSArray`；否则，它返回`NSDictionary`。

## 还有更多…

默认情况下，此方法返回的数据是不可变的。如果你需要可变的数据结构，你可以传递选项`NSJSONReadingMutableContainers`。要解析不是数组或字典的顶层字段，请传递选项`NSJSONReadingAllowFragments`。

## 另见

关于类的 Apple 文档位于 [`developer.apple.com/library/ios/documentation/Foundation/Reference/NSJSONSerialization_Class/index.html`](https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSJSONSerialization_Class/index.html)。

# 使用 Objective-C 在 iOS 上生成 JSON

您还可以使用 `NSJSONSerializer` 类序列化 `NSDictionary` 或 `NSArray`；只需使用 `dataWithJSONObject` 方法。

## 如何进行…

这是一个简单的例子，假设数据是您想要转换为 JSON 的`NSDictionary`：

```js
NSError *error;
NSData* jsonData = [NSJSONSerialization
dataWithJSONObject: data
options: NSJSONWritingPrettyPrinted
error: &error];
```

## 它是如何工作的…

`dataWithJSONObject:options:error` 方法可以接受 `NSArray` 或 `NSDictionary` 并返回一个包含您传递的集合编码 JSON 的 `NSData` 数据块。如果您传递 `kNilOptions`，JSON 将以紧凑的方式编码；要获取格式化的 JSON，请改为传递 `NSJSONWritingPrettyPrinted` 选项。

## 参见

关于 `NSJSONSerialization` 类的 Apple 文档位于 [`developer.apple.com/library/ios/documentation/Foundation/Reference/NSJSONSerialization_Class/index.html`](https://developer.apple.com/library/ios/documentation/Foundation/Reference/NSJSONSerialization_Class/index.html)。

# 使用 Swift 在 iOS 上解析 JSON

相同的 `NSJSONSerialization` 类在 Swift 中可用，这是 Apple 为 iOS 开发推出的新语言。

## 如何进行…

这是一个如何调用 `NSJSONSerialization` 类的 `JSONObjectWithData` 方法的 Swift 示例：

```js
import Foundation
var error: NSError?
Let json: NSData = /* the JSON to parse */
let data = NSJSONSerialization.JSONObjectWithData(json, 
  options: nil, 
  error: &error);
```

## 它是如何工作的…

Swift 中的方法调用看起来像函数调用，参数以可选命名的逗号分隔的形式传递，这与 C++或 Java 中的调用类似。`JSONObjectWithData` 方法的参数与 Objective-C 版本中的方法参数相同。

# 使用 Swift 在 iOS 上生成 JSON

当然，您也可以从 Swift 调用 `NSJSONSerialization.dataWithJSONObject` 方法，该方法返回一个 `NSData` 对象，您可以将其转换为字符串。

## 如何进行…

这是一个简单的例子：

```js
var error: NSError?
var data: NSJSONSerialization.dataWithJSONObject(
  dictionary, 
  options: NSJSONWritingOptions(0),
  error: &error);
var json: NSString(data: data, encoding: NSUTF8StringEncoding); 
```

## 它是如何工作的…

`dataWithJSONObject` 方法的工作方式与其 Objective-C 对应方法相同。一旦我们收到包含字典 JSON 编码版本的 `NSData`，我们使用 `NSString` 构造函数将其转换为 `NSString`。

# 使用 Qt 解析 JSON

Qt 实现的 JSON 解析在其接口上实际上与 Android 版本非常相似。Qt 定义了 `QJsonObject` 和 `QJsonArray` 类，分别可以包含 JSON 映射和 JSON 数组。解析本身是由 `QJsonDocument` 类完成的，该类有一个静态的 `fromJson` 方法，接受 JSON 并执行必要的解析。

## 如何进行…

这是一个简单的例子：

```js
QString json = "{ 'call': 'kf6gpe-7', 'lat': 37.40150, 'lng': -122.03683, 'result': 'ok'}";
QJsonDocument document = QJsonDocument.fromJson(json);
QJsonObject data = document.object;
QString call = data["call"].toString();
```

## 它是如何工作的…

解析是两步操作：首先，代码使用 `QJsonDocument` 解析 JSON，然后使用结果的 `QJsonObject` 访问数据。

`QJsonObject` 类作为 `QJsonValue` 对象的映射，每个对象都可以使用以下方法之一转换为其基本类型：

+   `toArray`: 这个方法转换为 `QJsonArray`。

+   `toBool`：这个方法转换为布尔值

+   `toDouble`：这个方法转换为双精度浮点数

+   `toInt`：这个方法转换为整数

+   `toObject`：这个方法转换为另一个`QJsonObject`，让你嵌套`QJsonObject`的映射

+   `toString`：这个方法转换为`QString`

## 还有更多内容…

你还可以使用 Qt 的`foreach`宏或者`begin`、`constBegin`和`end`迭代方法遍历`QJsonObject`中的键。此外，还有一个`contain`方法，它接受一个槽的名字，如果映射中包含你寻找的槽，则返回 true。

## 参见

参见 Qt 文档中关于 JSON 解析的部分：[`doc.qt.io/qt-5/json.html`](http://doc.qt.io/qt-5/json.html)。

# 使用 Qt 生成 JSON

`QJsonDocument`类还有一个`toJson`方法，该方法将引用的对象转换为 JSON。

## 如何做到…

以下是一个示例，它将 JSON 转换为另一种 JSON，并在转换过程中格式化 JSON：

```js
QString json = "{ 'call': 'kf6gpe-7', 'lat': 37.40150, 'lng': -122.03683, 'result': 'ok'}";
QJsonDocument document = QJsonDocument.fromJson(json);
QJsonObject data = document.object;
QByteArrayprettyPrintedJson = 
document.toJson(QJsonDocumented::Indented);    
```

## 它是如何工作的…

`QJsonDocument`类有一个`toJson`方法，该方法将引用的文档或数组转换为 JSON。你可以通过传递`QJsonDocument::Indented`来请求格式化后的 JSON 版本，或者通过传递`QJsonDocument::Compact`来请求紧凑的 JSON 版本。

## 参见

关于`QJsonDocument`的更多信息，请参阅 Qt 文档：[`doc.qt.io/qt-5/qjsondocument.html`](http://doc.qt.io/qt-5/qjsondocument.html)。
