# 第八章：使用 JSON 进行二进制数据传输

在本章中，我们将讨论 JSON 和二进制数据之间的交集。在这里，您会找到以下菜谱：

+   使用 Node.js 将二进制数据编码为 base64 字符串

+   使用 Node.js 从 base64 字符串解码二进制数据

+   在浏览器中使用 JavaScript 将二进制数据编码为 base64 字符串

+   使用 Json.NET 将数据编码为 BSON

+   使用 Json.NET 解码 BSON 数据

+   使用`DataView`访问`ArrayBuffer`

+   使用`ArrayBuffer`进行 base64 的编码和解码

+   使用 express 模块构建的 Node.js 服务器上压缩对象体内容

# 引言

使用 JSON 时考虑二进制表示通常有两个原因：要么是因为你需要将在应用程序的一个部分与另一个部分之间传输二进制数据，要么是因为你担心传输的 JSON 数据的大小。

在第一种情况下，你实际上有点束手无策，因为现有的 JSON 规范没有为二进制数据提供容器格式，因为 JSON 在本质上是一种基于文本的数据表示。你可以选择将二进制数据编码为另一种格式，如 base64，将二进制数据表示为可打印的字符串，或者使用支持二进制数据的 JSON 扩展，如二进制 JSON（BSON）。

BSON 使用 JSON 的语义，但以二进制形式表示数据。因此，同样的基本结构是可用的：一个（可能嵌套的）键值对映射，其中值可以是其他键值对、数组、字符串，甚至是二进制数据。然而，代替使用纯文本编码，该格式是二进制的，这产生了更小的数据大小并支持原生二进制对象（您可以在[`bsonspec.org/`](http://bsonspec.org/)了解更多关于 BSON 的信息）。BSON 的缺点是它不是原生支持 JavaScript，而且作为一种二进制格式，不容易进行检查。为了激发你的兴趣，我将在本章讨论如何使用流行的 Json.NET 库与 BSON 一起使用。

第二个方法是取任何二进制数据，并将其编码为与文本兼容的格式。Base64 就是这样一种编码机制，多年来在互联网上用于各种目的，并且在现代浏览器和 Node.js 中都有支持。在本章中，我展示了使用现代浏览器接口和 Node.js 与 base64 相互转换的菜谱。请注意，这意味着数据膨胀，因为将二进制信息表示为文本会增加传输的数据大小。

人们在考虑为他们的应用程序使用 JSON 时经常表达的一个担忧是，JSON 包的大小与二进制格式（如 BSON、协议缓冲区或手工调优的二进制表示）相比。虽然 JSON 可能比二进制表示大，但您获得了可读性（特别有助于调试）、清晰的语义，以及大量可用的库和实施实例。减少空白字符和使用简短的关键字名称可以帮助减小 JSON 的大小，压缩也可以——在我最近的一个项目中，我的测试显示，使用标准的 HTTP 压缩对 JSON 进行压缩，比全部二进制表示节省的内存更多，当然在服务器和客户端实现起来也更简单。

请记住，为了节省内存而转换为二进制格式——无论是 BSON、压缩还是自定义格式——都会抵消 JSON 的一个最有用的属性，即其自文档化属性。

# 使用 Node.js 将二进制数据编码为 base64 字符串

如果您有二进制数据需要编码以作为 JSON 传递给客户端，您可以将其转换为 base64，这是在互联网上表示八位值的一种常见方式，仅使用可打印字符。Node.js 提供了`Buffer`对象和`base64`编码器和解码器来完成这项任务。

## 如何做到…

首先，您会分配一个缓冲区，然后将其转换为字符串，指示您想要的字符串应该是 base64 编码的，如下所示：

```js
var buffer = newBuffer('Hello world');
var string = buffer.toString('base64');
```

## 它是如何工作的…

`Node.js`的`Buffer`类包装了一组八位字节，位于 Node.js V8 运行时堆之外。当您需要在 Node.js 中处理纯二进制数据时，它会用到。我们示例的第一行创建了一个缓冲区，用字符串`Hello world`填充它。

`Buffer`类包含`toString`方法，该方法接受一个参数，即编码缓冲区的手段。这里，我们传递了`base64`，表示我们希望`s`包含`b`的`base64`表示，但我们可以同样容易地传递以下值之一：

+   `ascii`：这个值表示应该移除高位比特，并将每个八位字节剩余的 7 位转换为其 ASCII 等效值。

+   `utf8`：这个值表示它应该作为多字节 Unicode 编码。

+   `utf16le`：这些是 2 个或 4 个字节的小端 Unicode 字符。

+   `hex`：这个值是将每个八位字节编码为两个字符，八位字节的`hex`值。

## 也见

有关 Node.js 的`Buffer`类的文档，请参阅[`nodejs.org/api/buffer.html`](https://nodejs.org/api/buffer.html)。

# 从 base64 字符串解码二进制数据使用 Node.js

在 Node.js 中，没有`Buffer.toString`的逆操作；相反，您直接将 base64 数据传递给缓冲区构造函数，并附上一个标志，表示数据是 base64 编码的。

## 准备

如果你想要像这里显示的那样运行示例，你需要安装`buffertools`模块，以获取`Buffer.compare`方法。为了获得这个模块，请在命令提示符下运行`npm`：

```js
npm install buffertools

```

如果你只是要使用 Node.js 的`Buffer`构造函数来解码 base64 数据，你不需要做这个。

## 如何做到…

在这里，我们将我们的原始缓冲区与另一个用原始 base64 初始化的缓冲区进行比较，这是为了第一个消息：

```js
require('buffertools').extend();

var buffer = new Buffer('Hello world');
var string = buffer.toString('base64');
console.log(string);

var another = new Buffer('SGVsbG8gd29ybGQ=', 'base64');
console.log(b.compare(another) == 0);
```

## 它是如何工作的…

代码的第一行包含了`buffertools`模块，它扩展了`Buffer`接口。这只是为了在最后一行使用缓冲区工具的`Buffer.compare`方法，不是因为 base64 需要自我解码。

接下来的两行创建了一个`Buffer`对象并获取其`base64`表示，接下来的行将这个表示输出到控制台。

最后，我创建了第二个`Buffer`对象，用一些 base64 数据初始化它，传递 base64 以表示初始化数据应该被解码到缓冲区中。我在最后一行比较这两个缓冲区。注意，缓冲区工具的`compare`方法是一个序数比较，意味着如果两个缓冲区包含相同的数据，它返回 0，如果第一个包含小于数据的序数排序，它返回-1，如果第一个包含序数排序更大的数据，它返回 1。

## 也见

关于`buffertools`模块及其实现的信息，请参阅[`github.com/bnoordhuis/node-buffertools#`](https://github.com/bnoordhuis/node-buffertools#)。

# 在浏览器中使用 JavaScript 对二进制数据进行 base64 字符串编码

JavaScript 的基本实现不包括 base64 编码或解码。然而，所有现代浏览器都包括了`atob`和`btoa`方法来分别解码和编码 base64 数据。这些方法是 window 对象的方法，由 JavaScript 运行时定义。

## 如何做到…

这只是方法调用的简单：

```js
var encodedData = window.btoa("Hello world"); 
var decodedData = window.atob(encodedData);
```

## 它是如何工作的…

`btoa`函数接收一个字符串并返回该字符串的 base64 编码。它是 window 对象的方法，并调用原生浏览器代码。`atob`函数做相反的事情，接收一个包含 base64 的字符串并返回一个包含二进制数据的字符串。

## 也见

关于`btoa`和`atob`的总结，请参阅 Mozilla 开发者网站上的[`developer.mozilla.org/en-US/docs/Web/API/WindowBase64/Base64_encoding_and_decoding`](https://developer.mozilla.org/en-US/docs/Web/API/WindowBase64/Base64_encoding_and_decoding)（注意虽然这些文档来自 Mozilla，但这些`window`的方法由大多数现代浏览器定义）。

# 使用 Json.NET 对数据进行 BSON 编码

BSON 编码是如果你在连接的每一边都有一个编码器和解码器的实现，那么它是 JSON 的一个合理替代方案。不幸的是，目前还没有适合 JavaScript 的好编码器和解码器，但是有包括.NET 和 C++在内的许多其他平台上的实现。让我们看看如何使用 Json.NET 在 C#中对 BSON 进行编码。

## 准备开始

首先，你需要让你的应用程序能够访问 Json.NET 程序集。正如你在上一章中看到的，在食谱*如何使用 Json.NET 反序列化一个对象*中，最容易的方法是使用 NuGet。如果你还没有这么做，按照那个食谱的步骤将 Json.NET 程序集添加到你的解决方案中。

## 如何做到…

使用 Json.NET 来编码 BSON 相对简单，一旦你有了想要编码的类：

```js
public class Record {
  public string Callsign { get; set; }
  public double Lat { get; set; }
  public double Lng { get; set; }
} 
…
var r = new Record {
  Callsign = "kf6gpe-7",
  Lat = 37.047,
  Lng = 122.0325
};

var stream = new MemoryStream();
using (var writer = new Newtonsoft.Json.Bson.BsonWriter(ms))
{
  var serializer = new Newonsoft.Json.JsonSerializer();
  serializer.Serialize(writer, r);
}
```

## 它是如何工作的…

最容易的方法是从一个具有你想要转换的场的类开始，正如你为其他类型的 JSON 安全转换所做的那样。在这里，我们为了这个目的定义了一个简单的`Record`类，然后创建一个记录来编码。

接下来，我们创建一个`MemoryStream`来包含编码后的数据，以及一个`BsonWriter`对象来将数据写入内存流。当然，任何实现.NET 流接口的东西都可以与`BsonWriter`实例一起使用；如果你愿意，你可以写入文件而不是内存流。在那之后，我们创建一个实际的序列化器来完成工作，`JsonSerializer`的一个实例，并使用它本身来序列化我们使用编写器创建的记录。我们将实际的序列化包裹在一个 using 块中，这样在操作结束时，写入器使用的资源（但不是流）会立即被.NET 运行时清理。

## 参见 also

关于 BsonWriter 类的文档可以从 NewtonSoft 处获得，网址为[`www.newtonsoft.com/json/help/html/T_Newtonsoft_Json_Bson_BsonWriter.htm`](http://www.newtonsoft.com/json/help/html/T_Newtonsoft_Json_Bson_BsonWriter.htm)。

# 使用 Json.NET 从 BSON 中解码数据

使用 Json.NET，解码 BSON 与编码相反；给定一个描述要解码数据的类和一个二进制数据块，调用一个读取器来读取数据。

## 准备开始

当然，为了做到这一点，你需要在你项目中有一个 Json.NET 程序集的引用。参见第七章*使用类型安全的方式使用 JSON*中的食谱*如何使用 Json.NET 反序列化一个对象*，了解如何使用 NuGet 在你的应用程序中添加 Json.NET 的引用。

## 如何做到…

从一个流开始，你将使用`BsonReader`和`JsonSerializer`来反序列化 BSON。假设数据是 BSON 数据的`byte[]`：

```js
MemoryStream ms = new MemoryStream(data);
using (var reader = new Newtonsoft.Json.Bson.BsonReader(ms))
{
  var serializer = new Newtonsoft.Json.JsonSerializer();
  var r = serializer.Deserialize<Record>(reader);

  // use r
}
```

## 它是如何工作的…

我们从传入的数据中创建`MemoryStream`，然后使用`BsonReader`实际从流中读取数据。读取工作由`JsonSerializer`完成，它使用读取器将数据反序列化为`Record`类的新实例。

## 还有更多…

你可能没有代表反序列化数据的类的应用；这在开发初期很常见，当时你仍在定义数据传输的语义。你可以使用`Deserialize`方法反序列化一个`JsonObject`实例，然后使用`JsonObject`的接口获取各个字段值。关于`JsonObject`的信息，请参阅 Json.NET 文档[`www.newtonsoft.com/json/help/html/T_Newtonsoft_Json_JsonObjectAttribute.htm`](http://www.newtonsoft.com/json/help/html/T_Newtonsoft_Json_JsonObjectAttribute.htm)。

## 参见

`BsonReader`的文档在[`www.newtonsoft.com/json/help/html/T_Newtonsoft_Json_Bson_BsonReader.htm`](http://www.newtonsoft.com/json/help/html/T_Newtonsoft_Json_Bson_BsonReader.htm)。

# 使用`DataView`访问`ArrayBuffer`

有时，你不想与 JSON 一起工作，而是想与纯二进制数据一起工作。JavaScript 提供了`DataView`抽象，它让你可以在一个数组缓冲区的内存上进行类型化的访问，比如从一个`XMLHttpRequest`对象获得的内存。

## 准备中

开始之前，你需要你的数据在一个`ArrayBuffer`中，比如`XMLHttpRequest`对象返回的那个。有了这个，你可以创建一个`DataView`，然后使用那个`DataArray`，在数据视图上创建一个类型数组，以提取你感兴趣的字节。让我们看一个例子。

## 如何做到…

这是一个简单的示例：

```js
var req = new XMLHttpRequest();
req.open("GET", url, true);
req.responseType = "arraybuffer";
req.onreadystatechange = function () {
  if (req.readyState == req.DONE) {
    var arrayResponse = req.response;
    var dataView = new DataView(arrayResponse);
    var ints = new Uint32Array(dataView.byteLength / 4);

    // process each int in ints here.

    }
}
req.send();
```

## 它是如何工作的…

首先要注意到的是`XMLHttpRequest`对象的`responseType`。在这个例子中，我们将它设置为`arraybuffer`，表示我们想要一个以`ArrayBuffer`类实例表示的原始字节缓冲区。我们发起请求，在完成处理程序上创建`DataView`的响应。

`DataView`是一个抽象对象，从这个对象中我们可以创建不同的视图来读写`ArrayBuffer`对象中的二进制数据。

`DataView`支持将`ArrayBuffer`对象视为以下内容：

+   `Int8Array`: 这是一个 8 位补码有符号整数数组

+   `Uint8Array`: 这是一个 8 位无符号整数数组

+   `Int16Array`: 这是一个 16 位补码有符号整数数组

+   `Uint16Array`: 这是一个 16 位无符号整数数组

+   `Int32Array`: 这是一个 32 位补码有符号整数数组

+   `Uint32Array`: 这是一个 32 位无符号整数数组

+   `Float32Array`: 这是一个 32 位浮点数数组

+   `Float64Array`: 这是一个 64 位浮点数数组

除了从一个`DataView`构造这些数组之外，你还可以从一个`DataView`访问单个 8 位、16 位、32 位整数或 32 位或 64 位浮点数，使用相应的获取函数，传递你想获取的偏移量。例如，`getInt8`返回指定位置的`Int8`，而`getFloat64`获取你指定偏移量处的相应的 64 位浮点数。

## 参见

尽管 `ArrayBuffer` 和 `DataView` 并不仅限于 Microsoft Internet Explorer，但 Microsoft 的 MSDN 网站上的文档非常清晰。有关 `DataView` 方法的信息，请参阅 [`msdn.microsoft.com/en-us/library/br212463(v=vs.94).aspx`](https://msdn.microsoft.com/en-us/library/br212463(v=vs.94).aspx)，或者参见 [`msdn.microsoft.com/library/br212485(v=vs.94).aspx`](https://msdn.microsoft.com/library/br212485(v=vs.94).aspx) 以获取关于类型数组的概述。

# 使用 ArrayBuffer 进行 base64 编码和解码

如果你打算使用 `ArrayBuffer` 和 `DataView` 为你 的二进制数据，并将二进制数据作为 base64 字符串携带，你可以使用由 Mozilla 编写的函数，位于 [`developer.mozilla.org/en-US/docs/Web/API/WindowBase64/Base64_encoding_and_decoding#Solution_.232_.E2.80.93_rewriting_atob%28%29_and_btoa%28%29_using_TypedArrays_and_UTF-8`](https://developer.mozilla.org/en-US/docs/Web/API/WindowBase64/Base64_encoding_and_decoding#Solution_.232_.E2.80.93_rewriting_atob%28%29_and_btoa%28%29_using_TypedArrays_and_UTF-8) 进行如此操作。他们提供了 `strToUTF8Arr` 和 `UTF8ArrToStr` 函数来执行 UTF-8 编码和解码，以及 `base64EncArr` 和 `base64DecToArr` 函数来在 base64 字符串和数组缓冲区之间进行转换。

## 如何做到…

这是一个相互转换示例，它将文本字符串编码为 UTF-8，然后将文本转换为 base64，然后显示 base64 结果，最后将 base64 转换为 UTF-8 数据的 `ArrayBuffer`，然后再将 UTF-8 转换回普通字符串：

```js
var input = "Base 64 example";

var inputAsUTF8 = strToUTF8Arr(input);

var base64 = base64EncArr(inputAsUTF8);

alert(base64);

var outputAsUTF8 = base64DecToArr(base64);

var output = UTF8ArrToStr(outputAsUTF8);

alert(output);
```

## 它是如何工作的…

Mozilla 在他们的网站文件中定义了四个函数：

+   `base64EncArr` 函数将字节 `ArrayBuffer` 编码为 base64 字符串

+   `base64DecToArr` 函数将 base64 字符串解码为字节 `ArrayBuffer`

+   `strToUTF8Arr` 函数将字符串编码为 `ArrayBuffer` 中的 UTF-8 编码字符数组

+   `UTF8ArrToStr` 函数接受 `ArrayBuffer` 中的 UTF-8 编码字符数组，并返回它所编码的字符串

# 压缩 Node.js 服务器中使用 express 模块构建的对象体内容

如果你在使用 JSON 时有空间方面的主要考虑，让你在考虑二进制表示时，你应该认真考虑使用压缩。压缩可以带来与二进制表示相似的节省，在大多数服务器和 HTTP 客户端中使用 `gzip` 实现，并且可以在调试完你的应用程序后作为透明层添加。在这里，我们讨论为流行的基于 Node.js 的 express 服务器发送的 JSON 和其他对象添加对象体压缩。

## 准备好了

首先，你需要确保已经安装了 express 和 compress 模块：

```js
npm install express
npm install compression

```

如果你想要它在你的工作区中的所有 Node.js 应用程序中可用，你也可以 `npm install –g` 它。

## 如何做到…

在你服务器的入口点初始化 `express` 模块时，需要 require 压缩，并告诉 `express` 使用它：

```js
var express = require('express')
var compression = require('compression')
var app = express()
app.use(compression())

// further express setup goes here.
```

关于如何使用`express`模块来设置服务器的更多信息，请参阅第五章中的菜谱“为 Node.js 安装 express 模块”，*使用 JSON 与 MongoDB*.

## 它是如何工作的…

HTTP 头支持客户端指示它是否能够解压缩通过 HTTP 发送的对象体，并且现代浏览器都接受`gzipped`对象体。通过在基于 express 的服务器中包含 compress，你使得客户端可以请求压缩后的 JSON 作为其 Web API 请求的一部分，并且响应中也返回压缩后的 JSON。在大多数情况下，大多数客户端不需要进行任何更改，尽管如果你正在编写带有自己 HTTP 实现的本地客户端，你可能需要查阅文档以确定如何通过 HTTP 启用`gzip`解压缩。

代码首先需要引入 express 模块和压缩模块，然后配置 express 模块，在客户端请求压缩时，可选地使用压缩功能来发送响应。
