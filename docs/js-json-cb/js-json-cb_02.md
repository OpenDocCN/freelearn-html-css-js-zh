# 第二章：服务器端的读写 JSON

在前一章中，我们查看了一些最常见的客户端环境中的 JSON 处理。在本章中，我们将注意力转向服务器端的 JSON 编码和解码。我们将查看以下环境中的食谱：

+   在 Clojure 中读写 JSON

+   在 F#中读写 JSON

+   在 Node.js 中读写 JSON

+   在 PHP 中读写 JSON

+   在 Ruby 中读写 JSON

一些语言，如 C++和 Java，既用于客户端又用于服务器端；对于这些语言，请参考第一章，*客户端的读写 JSON*（一个例外是关于 Node.js 中的 JSON 讨论，因为 Node.js 在这本书的后续章节中扮演重要角色）。

# 在 Clojure 中读写 JSON

Clojure 是一种运行在 Java 和 Microsoft **公共** **语言运行时**（**CLR**）平台之上的现代 Lisp 变体。因此，你可以使用我们在第一章中讨论的设施，在本地运行时将 JSON 和对象之间进行转换，但有一个更好的方法，那就是 Clojure 的`data.json`模块，可在[`github.com/clojure/data.json`](https://github.com/clojure/data.json)找到。

## 准备

首先，你需要在你`data.json`模块中指定你的依赖。你可以用以下依赖在你的 Leiningen 文件中这样做：

```js
[org.clojure/data.json "0.2.5"]
```

如果你使用 Maven，你会需要这个：

```js
<dependency>
<groupId>org.clojure</groupId>
<artifactId>data.json</artifactId>
<version>0.2.5</version>
</dependency>
```

### 提示

当然，`data.json`的版本可能会在我写这篇和你在项目中作为依赖包含它之间发生变化。查看 data.json 项目以获取当前版本。

最后，你需要在你的代码中包含`data.json`模块，在一个像`json`这样的命名空间中：

```js
(ns example
  (:require [clojure.data.json :as json])
```

这使得`data.json`模块的实现通过`json`命名空间可用。

## 如何做到...

将 Clojure 映射编码为 JSON 很容易，只需调用`json/write-str`。例如：

```js
(json/write-str {:call "KF6GPE",:type "l",:time 
"1399371514":lasttime"1418597513",:lat 37.17667,:lng
-122.14650: :result "ok"})
;;=>"{\"call\": \"KF6GPE\", \"type\": \"l\", \"time\":
\"1399371514\", \"lasttime\": \"1418597513\", \"lat\": 37.17667,
\"lng\": -122.14650,\"result\":\"ok\"}"
```

如果你有一个实现`java.io.Writer`的流，你想向其写入 JSON，你也可以使用`json/write`：

```js
(json/write {:call "KF6GPE",:type "l", :time 
"1399371514":lasttime "1418597513",:lat 37.17667, :lng 
-122.14650: result "ok" }  stream)
```

阅读是写作的相反过程，它将 JSON 读取到关联数组中，你可以进一步处理：

```js
(json/read-str "{\"result\":\"ok\"}")
;;=> {"result" "ok"}
```

还有`json/read`，`json/write`的对应物，它接收一个流，你可以从中读取并返回解析后的 JSON 映射。

## 还有更多...

这些方法都接受两个可选参数，一个`:key-fn`参数，模块将其应用于每个 JSON 属性名称，和一个`:value-fn`参数，模块将其应用于属性值。例如，你可以使用`:key-fn keyword`将 JSON 转换为更传统的 Clojure 关键词映射，像这样：

```js
(json/read-str "{\"call\": \"KF6GPE\", \"type\": \"l\", \"time\":
\"1399371514\", \"lasttime\": \"1418597513\", \"lat\": 37.17667,
\"lng\": -122.14650,\"result\":\"ok\"}:key-fn keyword)
;;=> {:call "KF6GPE",:type "l", :time 
"1399371514":lasttime "1418597513",:lat 37.17667, :lng 
-122.14650: :result "ok"}
```

或者，你可以提供一个 lambda，比如以下这个，它将键转换为大写：

```js
(json/write-str {:result "OK"}
                :key-fn #(.toUpperCase %))
;;=> "{\"RESULT\":"OK"}"
```

这里有一个来自 `data.json` 文档的很好例子，它使用 `value-fn` 将 ISO 日期字符串转换为 Java `Date` 对象，当你解析 JSON 时：

```js
(defn my-value-reader [key value]
  (if (= key :date)
    (java.sql.Date/valueOf value)
    value))

(json/read-str "{\"result\":\"OK\",\"date\":\"2012-06-02\"}"
               :value-fn my-value-reader
               :key-fn keyword) 
;;=> {:result"OK", :date #inst "2012-06-02T04:00:00.000-00:00"}
```

上述代码执行以下操作：

1.  定义了一个辅助函数 `my-value-reader`，它使用 JSON 键值对的关键字来确定其类型。

1.  给定一个 JSON 键值 `:date`，它将该值视为一个字符串，传递给 `java.sql.Date` 方法的 `valueOf`，该方法返回一个从它解析的字符串中获取值的 `Date` 实例。

1.  调用 `json/read-str` 来解析一些简单的 JSON 数据，它包括两个字段：一个 `result` 字段和一个 `date` 字段。

1.  **JSON 解析器**解析 JSON 数据，将 JSON 属性名转换为关键字，并使用我们之前定义的值转换器将日期值转换为它们的 `java.sql.Date` 表示形式。

# 在 F# 中读写 JSON

F# 是一种运行在 CLR 和 .NET 之上的语言，擅长于函数式和面向对象编程任务。因为它建立在 .NET 之上，所以你可以使用诸如 `Json.NET`（在 第一章，*客户端的 JSON 读写* 中提到）等第三方库来将 JSON 与 CLR 对象进行转换。然而，还有一个更好的方法：开源库 F# Data，它创建了本地的数据类型提供程序来处理多种不同的结构化格式，包括 JSON。

## 准备开始

首先，获取库的副本，可在 [`github.com/fsharp/FSharp.Data`](https://github.com/fsharp/FSharp.Data) 找到。下载后，你需要构建它；你可以通过运行随分发提供的 `build.cmd` 构建批处理文件来完成此操作（有关详细信息，请参见 F# Data 网站）。或者，你可以在 NuGet 上找到相同的包，通过从 **项目** 菜单中选择 **管理 NuGet 包** 并搜索 F# Data。找到后，点击 **安装**。我更喜欢使用 NuGet，因为它会自动将 `FSharp.Data` 程序集添加到你的项目中，省去了你自己构建源代码的麻烦。另一方面，源分发可以让你离线阅读文档，这也很有用。

一旦你获得了 F# 数据，你只需要在源文件中打开它，你打算用它时使用 `open` 指令，像这样：

```js
open FSharp.Data
```

## 如何进行...

以下是一些示例代码，它实现了 JSON 与 F# 对象之间的转换，然后又从另一个 F# 对象创建了新的 JSON 数据：

```js
open FSharp.Data

type Json = JsonProvider<""" { "result":"" } """>
let result = Json.Parse(""" { "result":"OK" } """)
let newJson = Json.Root( result = "FAIL")

[<EntryPoint>]
let main argv = 
    printfn "%A" result.Result
    printfn "%A" newJson
    printfn "Done"
```

让我们看看它是如何工作的。

## 它是如何工作的...

首先，重要的是要记住 F#是强类型的，并且从数据中推断类型。理解这一点对于理解 F# Data 库是如何工作的至关重要。与我们在前面章节中看到的例子不同，在那里转换器将 JSON 映射到键值对，F# Data 库从您提供的 JSON 中推断出整个数据类型。在许多方面，这既结合了其他转换器在转换 JSON 时采取的动态集合导向方法，也结合了我在第七章 *以类型安全的方式使用 JSON*中要展示的类型安全方法。这是因为您不需要辛苦地制作要解析的 JSON 的类表示，而且您还能在编写的代码中获得编译时类型安全的所有优势。更妙的是，F# Data 创建的类都支持 Intellisense，所以您在编辑器中就能直接获得工具提示和名称补全！

让我们逐段看看之前的示例并了解它做了什么：

```js
open FSharp.Data
```

第一行使 F# Data 类可用于您的程序。 among other things, 这定义了`JsonProvider`类，该类从 JSON 源创建 F#类型：

```js
type Json= JsonProvider<""" { "result":"" } """>
```

这行代码定义了一个新的 F#类型`Json`，该类型的字段和字段类型是从您提供的 JSON 中推断出来的。在底层，这做了很多工作：它推断出成员名称、成员的类型，甚至处理诸如混合数值（比如说您有一个同时包含整数和浮点数的数组，它能正确推断出类型为数值类型，这样你就可以表示任一类型）以及复杂的记录和可选字段等事情。

您可以向`JsonProvider`传递以下三个之一：

1.  一个包含 JSON 的字符串。这是最简单的情况。

1.  一个包含 JSON 文件的路径。库将打开文件并读取内容，然后对内容进行类型推断，最后返回一个能够表示文件中 JSON 的类型。

1.  一个 URL。库将获取 URL 处的文档，解析 JSON，然后对内容进行相同的类型推断，返回一个代表 URL 处 JSON 的类型。

下一行解析一个单独的 JSON 文档，如下所示：

```js
let result = Json.Parse(""" { "result":"OK" } """)
```

这可能一开始看起来有点奇怪：为什么我们要把 JSON 同时传递给`JsonProvider`和`Parse`方法呢？回想一下，`JsonProvider`是从您提供的 JSON 中创建一个类型。换句话说，它不是为了解析 JSON 的值，而是为了解析它所表示的数据类型，以便制作一个能够模拟 JSON 文档本身的类。这一点非常重要；对于`JsonProvider`来说，您想要传递一个具有字段和值的代表性 JSON 文档，这个文档是您的应用程序可能遇到的某一特定类型的所有 JSON 文档的共通之处。然后，您将一个特定的 JSON 文档（比如说，一个 web 服务结果）传递给`JsonProvider`创建的类的`Parse`方法。相应地，`Parse`返回了一个您调用`Parse`的方法的实例。

现在你可以访问类`Parse`返回实例中的字段；例如，稍后，我将在应用程序的`main`函数中打印出`result.Result`的值。

创建 JSON，你需要一个表示要序列化数据的类型的实例。在下一行，我们使用刚刚创建的`Json`类型来创建一个新的 JSON 字符串：

```js
let newJson = Json.Root( result = "FAIL")
```

这创建了一个`Json`类型的实例，并将结果字段设置为字符串`FAIL`，然后将该实例序列化为一个新的字符串。

最后，程序的其余部分是程序的入口点，并只是打印出解析的对象和创建的 JSON。

## 还有更多...

F#数据库支持远不止只是 JSON；它还支持**逗号分隔值**（**CSV**）、HTML 和 XML。这是一个非常适合进行各种结构化数据访问的优秀库，如果你在 F#中工作，那么熟悉它绝对是件好事。

# 使用 Node.js 读写 JSON

Node.js 是一个基于与 Google 为 Chrome 构建的高性能 JavaScript 运行时相同的高性能和异步编程模型的 JavaScript 环境，由 Joyent 支持。它高性能和异步编程模型使它成为一个优秀的自定义 Web 服务器环境，并且它被包括沃尔玛在内的许多大型公司用于生产环境。

## 准备

因为我们在接下来的两章中也会使用 Node.js，所以值得向你指出如何下载和安装它，即使你的日常服务器环境更像 Apache 或 Microsoft IIS。你需要访问[`www.nodejs.org/`](http://www.nodejs.org/)，并从首页下载安装程序。这将安装运行 Node.js 和 npm（Node.js 使用的包管理器）所需的一切。

### 提示

在 Windows 上安装后，我必须重新启动计算机，才能使 Windows 外壳正确找到 Node.js 安装程序安装的 node 和 npm 命令。

一旦你安装了 Node.js，我们可以通过在 Node.js 中启动一个简单的 HTTP 服务器来测试安装。为此，将以下代码放入一个名为`example.js`的文件中：

```js
var http = require('http');
http.createServer(function(req, res) {
   res.writeHead(200, {'Content-Type': 'text/plain'});
   res.end('Hello world\n');
}).listen(1337, 'localhost');
console.log('Server running at http://localhost:1337');
```

这段代码加载了 Node.js 的`http`模块，然后创建了一个绑定在本地机器上端口`1337`的 Web 服务器。你可以在创建文件的同一目录下，通过命令提示符输入以下命令来运行它：

```js
node example.js
```

一旦这样做，将你的浏览器指向 URL`http://localhost:1337/`。如果一切顺利，你应该在网页浏览器中看到“Hello world”的消息。

### 提示

你可能需要告诉你的系统防火墙，以启用对由`node`命令服务的端口的访问。

## 怎么做...

由于 Node.js 使用 Chrome 的 V8 JavaScript 引擎，因此 Node.js 中处理 JSON 与 Chrome 中的处理方式相同。JavaScript 运行时定义了`JSON`对象，该对象为您提供了 JSON 解析器和序列化器。

解析 JSON，你所需要做的就是调用`JSON.parse`方法，像这样：

```js
var json = '{ "call":"KF6GPE","type":"l","time":
"1399371514","lasttime":"1418597513","lat": 37.17667,"lng":
-122.14650,"result" : "ok" }';
var object = JSON.parse(json);
```

这将解析 JSON，返回包含数据的 JavaScript 对象，我们在这里将其分配给变量 object。

当然，你可以做相反的操作，使用`JSON.stringify`，像这样：

```js
var object = { 
call:"KF6GPE",
type:"l",
time:"1399371514",
lasttime:"1418597513",
lat:37.17667,
lng:-122.14650,
result: "ok"
};

var json = JSON.stringify(object);
```

## 也见

关于在 JavaScript 中解析和创建 JSON 的更多信息，请参阅第一章中的*在客户端读写 JSON*一节，以及*读写客户端 JSON*。

# 在 PHP 中读写 JSON

PHP 是一个流行的服务器端脚本环境，可以轻松与 Apache 和 Microsoft IIS 网络服务器集成。它具有内置的简单 JSON 编码和解码支持。

## 如何做到...

PHP 提供了两个函数`json_encode`和`json_decode`，分别用于编码和解码 JSON。

你可以将原始类型或自定义类传递给`json_encode`，它将返回一个包含对象 JSON 表示的字符串。例如：

```js
$result = array(
"call" =>"KF6GPE",
"type" =>"l",
"time" =>"1399371514",
"lasttime" =>"1418597513",
"lat" =>37.17667,
"lng" =>-122.14650,
"result" =>"ok");
$json = json_encode($result);
```

这将创建一个包含我们关联数组的 JSON 表示的字符串`$json`。

`json_encode`函数接受一个可选的第二个参数，让你指定编码器的参数。这些参数是标志，所以你可以用二进制或`|`操作符来组合它们。你可以传递以下标志的组合：

+   `JSON_FORCE_OBJECT`：这个标志强制编码器将 JSON 编码为对象。

+   `JSON_NUMERIC_CHECK`：这个标志检查传入结构中的每个字符串的内容，如果它包含一个数字，则在编码之前将字符串转换为数字。

+   `JSON_PRETTY_PRINT`：这个标志将 JSON 格式化为更容易供人类阅读的形式（不要在生产环境中这样做，因为这会使 JSON 变大）。

+   `JSON_UNESCAPED_SLASHES`：这个标志指示编码器不要转义斜杠字符。

最后，你可以传递一个第三个参数，指定编码器在编码你传递的值时应遍历表达式的深度。

`json_encode`的补数是`json_decode`，它接受要解码的 JSON 和一个可选参数集合。其最简单的用法可能像这样：

```js
$json = '{ "call":"KF6GPE","type":"l","time":
"1399371514","lasttime":"1418597513","lat": 37.17667,"lng":
-122.14650,"result" : "ok" }';
$result = json_decode($json);
```

`json_decode`函数最多接受三个可选参数：

+   第一个参数，当为真时，指定结果应该以关联数组的形式返回，而不是`stdClass`对象。

+   第二个参数指定一个可选的递归深度，以确定解析器应深入解析 JSON 的深度。

+   第三个参数可能是选项`JSON_BIGINT_AS_STRING`，当设置时表示应该将溢出整数值的整数作为字符串返回，而不是转换为浮点数（这可能会失去精度）。

这些函数在成功时返回`true`，在出错时返回`false`；你可以通过检查`json_last_error`的返回值来使用 JSON 确定上一次错误的的原因。

# 在 Ruby 中读写 JSON

Ruby 提供了`json`宝石用于处理 JSON。在 Ruby 的早期版本中，你必须自己安装这个宝石；从 Ruby 1.9.2 及以后版本开始，它成为基础安装的一部分。

## 准备

如果你正在使用比 Ruby 1.9.2 更早的版本，首先需要使用以下命令安装 gem：

```js
gem install json

```

请注意，Ruby 的实现是 C 语言，因此安装 gem 可能需要 C 编译器。如果您系统上没有安装，您可以使用以下命令安装 gem 的纯 Ruby 实现：

```js
gem install json_pure

```

无论你是否需要安装 gem，你都需要在代码中包含它。要做到这一点，请包含`rubygems`和`json`或**json/pure**，具体取决于你安装了哪个 gem；使用`require`，像这样：

```js
require 'rubygems'
require 'json'
```

前面的代码处理了前一种情况，而下面的代码处理了后一种情况：

```js
require 'rubygems'
require 'json/pure'
```

## 如何做到...

该 gem 定义了 JSON 对象，其中包含`parse`和`generate`方法，分别用于序列化和反序列化 JSON。使用它们正如你所期望的那样。创建一个对象或一些 JSON，调用相应的函数，然后查看结果。例如，要使用 JSON.generate 创建一些 JSON，你可以执行以下操作：

```js
require 'rubygems'
require 'json'
object = { 
"call" =>"KF6GPE",
"type" =>"l",
"time" =>"1399371514",
"lasttime" =>"1418597513",
"lat" => 37.17667,
"lng" => -122.14650,
"result" =>"ok"
}
json = JSON.generate(object)
```

这包括必要的模块，创建一个具有单个字段的关联数组，然后将其序列化为 JSON。

反序列化工作方式与序列化相同：

```js
require 'rubygems'
require 'json'
json = '{ "call":"KF6GPE","type":"l","time":
"1399371514","lasttime":"1418597513","lat": 37.17667,"lng":
-122.14650,"result" : "ok" }'
object = JSON.parse(object)
```

`parse`函数可以接受一个可选的第二个参数，一个具有以下键的哈希，表示解析器的选项：

+   `max_nesting`表示允许在解析的数据结构中嵌套的最大深度。它默认为 19，或者可以通过传递`:max_nesting => false`来禁用嵌套深度检查。

+   `allow_nan`，如果设置为真，则允许 NaN、Infinity 和-Infinity，这与 RFC 4627 相悖。

+   `symbolize_names`，当为真时，返回 JSON 对象中属性名的符号；否则，返回字符串（字符串是默认值）。

## 另见

JSON Ruby gem 的文档可以在网上找到，网址为[`flori.github.io/json/doc/index.html`](http://flori.github.io/json/doc/index.html)。
