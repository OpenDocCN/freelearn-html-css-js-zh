# 第五章。Node.js 和正则表达式

到目前为止，我们已经乐在其中学习如何为不同情况创建正则表达式。然而，您可能想知道在现实世界的情况下应用正则表达式会是什么样子，比如读取一个日志文件并以用户友好的格式呈现其信息？

在本章中，我们将学习如何实现一个简单的**Node.js**应用程序，它可以读取一个日志文件，并使用正则表达式解析它。这样，我们可以从中检索特定信息，并以不同的格式输出它。我们将测试我们从本书前几章中获得的所有知识。

在本章中，我们将涵盖以下主题：

+   安装所需的软件来开发我们的示例

+   使用 Node.js 读取文件

+   分析 Apache 日志文件的解剖学

+   使用正则表达式创建解析器来读取 Apache 日志文件

# 设置 Node.js

由于我们将开发一个 Node.js 应用程序，第一步是安装 Node.js。我们可以从[`nodejs.org/download/`](http://nodejs.org/download)获取它。只需按照下载说明进行操作，我们就可以在计算机上设置好它。

### 注意

如果这是您第一次使用 Node.js，请阅读[`nodejs.org/`](https://nodejs.org/)上的教程。

为了确保我们已经安装了 Node.js，打开终端应用程序（如果您使用 Windows，则为命令提示符），然后输入`node –v`。安装的 Node.js 版本应该显示如下：

![设置 Node.js](img/2258OS_05_01.jpg)

我们现在可以开始了！

# 开始我们的应用程序

让我们开始用 Node.js 开发我们的示例应用程序，它将读取一个日志文件，并使用正则表达式解析其信息。我们将在一个名为`regex.js`的 JavaScript 文件中创建所有所需的代码。在我们开始编码之前，我们将进行一个简单的测试。在`regex.js`中添加以下内容：

```js
console.log('Hello, World!');
```

接下来，在终端应用程序中，从创建文件的目录中执行`regex.js`命令节点。**Hello, World!**消息应该显示如下：

![开始我们的应用程序](img/2258OS_05_02.jpg)

使用 Node.js 创建了 hello world 应用程序并且它可以工作了！我们现在可以开始编写我们的应用程序了。

## 使用 Node.js 读取文件

由于我们的应用程序的主要目标是读取一个文件，我们需要应用程序要读取的文件！我们将使用一个示例 Apache 日志文件。互联网上有很多文件，但我们将使用可以从[`fossies.org/linux/source-highlight/tests/access.log`](http://fossies.org/linux/source-highlight/tests/access.log)下载的日志文件。将文件放在创建`regex.js`文件的同一目录中。

### 注意

这个示例 Apache 日志文件也可以在本书的源代码包中找到。

要使用 Node.js 读取文件，我们需要导入 Node.js 文件系统模块。删除我们在`regex.js`文件中放置的`console.log`消息，并添加以下一行代码：

```js
var fs = require('fs');
```

### 注意

要了解更多关于 Node.js 文件系统模块的信息，请阅读其文档[`nodejs.org/api/fs.html`](http://nodejs.org/api/fs.html)。

下一步是打开文件并读取其内容。我们将使用以下代码来实现这一点：

```js
fs.readFile('access.log', function (err, data) {//#1

  if (err) throw err;//#2

  var text = data.toString();//#3

  var lines = text.split('\n');//#4

  lines.forEach(function(line) {//#5
    console.log(line);//#6
  });
});
```

根据 Node.js 文档，`readFile`函数（`#1`）可以接收三个参数：文件名（`access.log`），某些选项（在本例中我们没有使用），以及当文件内容加载到内存中时将执行的回调函数。

### 注意

要了解更多关于`readLine`函数的信息，请访问[`nodejs.org/api/fs.html#fs_fs_readfile_filename_options_callback`](http://nodejs.org/api/fs.html#fs_fs_readfile_filename_options_callback)。

回调函数接收两个参数。第一个是错误。如果出现问题，将抛出异常（`＃2`）。第二个参数是`data`，其中包含文件内容。我们将在名为`text`的变量中存储一个包含所有文件内容的字符串（`＃3`）。

然后，日志的每条记录都放在文件的一行中。因此，我们可以继续分隔文件记录并将其存储到一个数组中（`＃4`）。现在，我们可以迭代包含日志行的数组（`＃5`）并在每行执行一个操作。在这种情况下，我们现在只是在`console`（`＃6`）中输出每行的内容。我们将在下一节中用不同的逻辑替换代码的第`＃6`行。

如果我们执行`regex.js`命令节点，则所有文件内容应显示如下：

![使用 Node.js 读取文件](img/2258OS_05_03.jpg)

# Apache 日志文件的解剖

在创建将匹配 Apache 文件一行的正则表达式之前，我们需要了解它包含什么样的信息。

让我们看一下`access.log`中的一行：

```js
127.0.0.1 - jan [30/Jun/2004:22:20:17 +0200] "GET /cgi-bin/trac.cgi/login HTTP/1.1" 302 4370 "http://saturn.solar_system/cgi-bin/trac.cgi" "Mozilla/5.0 (X11; U; Linux i686; en-US; rv:1.7) Gecko/20040620 Galeon/1.3.15"
```

我们正在阅读的 Apache 访问日志遵循`％h％l％u％t“％r”％>s％b“％{Referer}i”“％{User-agent}i”`格式。让我们看看每个部分：

+   `％h`：日志的第一部分是（`127.0.0.1`）IP 地址

+   `％l`：在第二部分中，输出中的连字符表示所请求的信息部分不可用

+   `％u`：第三部分是请求（`jan`）文档的用户 ID。

+   `％t`：第四部分是请求接收的时间，例如（`[30/Jun/2004:22:20:17 +0200]`）。它的格式是`[day/month/year:hour:minute:second zone]`，其中：

+   `day` = 2*digit

+   `month` = 3*letter

+   `year` = 4*digit

+   `hour` = 2*digit

+   `minute` = 2*digit

+   `second` = 2*digit

+   `zone` =（``+' | `-'）4*digit

+   `\"％r\"`：第五部分是客户端的请求行，用双引号给出，例如（`"GET /cgi-bin/trac.cgi/login HTTP/1.1"`）

+   `％>s`：第六部分是服务器发送给（`302`）客户端的状态代码

+   `％b`：第七部分是返回给（`4370`）客户端的对象的大小

+   `\"％{Referer}i\"`：第八部分是客户端报告从中引用的站点，用双引号给出，例如（`"http://saturn.solar_system/cgi-bin/trac.cgi"`）

+   `\"％{User-agent}i\"`：第九部分也是用户代理 HTTP 请求标头，也用双引号给出，例如（`"Mozilla/5.0（X11；U；Linux i686；en-US；rv：1.7）Gecko/20040620 Galeon/1.3.15"`）

所有部分都用空格分隔。有了这些信息和之前提供的信息，我们可以开始创建我们的正则表达式。

### 注意

有关 Apache 日志格式的更多信息，请阅读[`httpd.apache.org/docs/2.2/logs.html`](http://httpd.apache.org/docs/2.2/logs.html)。

## 创建 Apache 日志正则表达式

在 Apache 访问日志文件中，我们要识别并从文件的每一行中提取九个部分。在创建正则表达式时，我们可以尝试两种方法：可以非常具体或更通用。如前所述，最强大的正则表达式是通用的。我们将在本章中尝试实现这些表达式。

例如，对于日志的第一部分，我们知道它是一个 IP 地址。我们可以具体使用正则表达式（`^\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b`）匹配 IP，或者，正如我们所知，日志以 IP 开头，我们可以使用`^(\S+)`，其中`^`表示它匹配输入的开头，`\S`匹配除空格之外的单个字符。`^(\S+)`表达式将精确匹配日志的第一部分，直到找到空格（例如 IP 地址）。此外，`^(\S+)`比使用`^\b\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}\b`更简单，但我们仍然实现了相同的结果。

让我们继续测试到目前为止创建的正则表达式：

![创建 Apache 日志正则表达式](img/2258OS_05_04.jpg)

为了总结我们在第一章中学到的内容，*正则表达式入门*，`exec`方法在字符串中执行匹配搜索。它返回一个信息数组，因为它是字符串第一个匹配的位置，然后是正则表达式的每个部分中的后续位置。

对于第二部分和第三部分，我们可以继续使用`^(\S+)`正则表达式。第二部分和第三部分可以包含特定信息（包括一组字母数字字符），或者可以包含连字符。我们对每个部分中出现的信息感兴趣，直到找到一个空格。因此，我们可以在我们的正则表达式中添加两个`^(\S+)`：`^(\S+) (\S+) (\S+)`并测试它：

![创建 Apache 日志正则表达式](img/2258OS_05_05.jpg)

日志行的前三部分被识别出来。

### 为时间部分创建正则表达式

第四部分是括号之间给定的时间。从日志中匹配时间的正则表达式是`\[([^:]+):(\d+:\d+:\d+) ([^\]]+)\]`。

让我们看看如何实现这个结果。

首先，我们有开放和关闭括号。我们不能简单地在正则表达式中使用`[]`，因为正则表达式中的括号代表一组字符（正如我们在第三章中学到的那样，*特殊字符*）。因此，我们需要在每个括号前使用（`\`）转义字符，以便我们可以将括号表示为正则表达式的一部分。

时间正则表达式的下一部分是`"([^:]+):"`。在开放括号之后，我们想匹配任何字符，直到找到（`:`）冒号。我们在第二章中学到了关于否定范围的知识，这正是我们要使用的。我们期望任何字符都可以出现，除了冒号，所以我们使用`[ˆ:]`来表示它。此外，它可以由一个或多个字符组成，比如（`+`）。接下来，我们期望有一个（`:`）冒号。有了这个正则表达式的一部分，我们就可以匹配`"[30/Jun/2004:"`，从`"[30/Jun/2004:22:20:17 +0200]"`中。

同样的正则表达式可以表示为`"(\d{2}\/\w{3}\/\d{4}):"`，因为日期以两位数字的形式给出，月份以三个字符给出，年份以四位数字给出，并用`\`分隔。

正则表达式的下一部分是`(\d+:\d+:\d+)`。它将从示例中匹配`22:20:17`。`\d`字符匹配任何数字（`+`匹配一个或多个），后跟一个（`:`）冒号。我们也可以使用（`\d{2}:\d{2}:\d{2}`），因为小时、分钟和秒分别由两位数字表示。

最后一部分是`([^\]]+)\]`。我们期望任何字符，除了`"]"（[^\]] - 否定]）`。这将匹配时区（+0200）。我们也可以使用`([\+|-]\d{4})`作为正则表达式，因为时区格式是`+`或`-`，后跟四位数字。

当我们测试正则表达式时，我们将得到这个结果：

![为时间部分创建正则表达式](img/2258OS_05_06.jpg)

### 注意

请注意，时间的每个部分都被一个子集分割（日期、时间和时区），由括号组（）分隔。如果我们想将时间作为一个单独的部分，我们可以删除子集：`\[(\d{2}\/\w{3}\/\d{4}:\d{2}:\d{2}:\d{2} [\+|-]\d{4})\]`。

### 为请求信息创建正则表达式

在我们分开的部分（在此之前的几个部分中），让我们来处理日志的第五部分，即请求信息。

让我们看一下"`GET /cgi-bin/trac.cgi/login HTTP/1.1`"的例子，这样我们就可以从中创建一个正则表达式。

请求用双引号括起来，这样我们知道要在`\" \"`内创建一个正则表达式。从前面的例子中，有三个部分（`GET`、`/cgi-bin/trac.cgi/login`和`HTTP/1.1`）。因此，`GET`可以用`(\S+)`表示。

接下来，我们有`/cgi-bin/trac.cgi/login`。我们将使用`(.*?)`，意思是，它可以是任何字符，或者什么都没有。我们将使用这个，因为我们不知道这个信息的格式。

然后，我们有`HTTP/1.1`协议，为了匹配它，我们还将使用`(\S+)`。

当我们尝试匹配正则表达式时，将会得到以下结果：

![为请求信息创建正则表达式](img/2258OS_05_07.jpg)

### 提示

如果我们想要分别检索请求的每个部分（比如方法、资源和协议），我们可以像之前一样使用`()`。

### 为状态码和对象大小创建正则表达式

日志的下两部分很简单。第一部分是状态，表示为`2xx`、`3xx`、`4xx`或`5xx`，因此基本上是三位数。我们可以用两种方式表示它：`(\S+)`，它将匹配任何字符，直到找到一个空格；或者`(\d{3})`。当然，我们还可以更具体一些，允许第一个数字只能是`2`、`3`、`4`或`5`，不过，让我们不要把它搞得太复杂。

一个数字也可以表示对象大小。然而，如果没有返回信息，它将用连字符表示，所以`(\S+)`表示得最好。或者我们也可以使用`([\d|-]+)`。

输出将会是以下内容：

![为状态码和对象大小创建正则表达式](img/2258OS_05_08.jpg)

### 为引荐者和用户代理创建正则表达式

这两部分都用双引号括起来。我们可以使用`"([^"]*)"`表达式来表示这些信息，这意味着包括除了`"`之外的任何字符。我们可以在这两部分中应用它。

通过添加日志的最后两部分，我们将得到以下输出：

![为引荐者和用户代理创建正则表达式](img/2258OS_05_09.jpg)

我们的最终正则表达式来匹配 Apache 访问日志的一行如下：

```js
^(\S+) (\S+) (\S+) \[(\d{2}\/\w{3}\/\d{4}:\d{2}:\d{2}:\d{2} [\+|-]\d{4})\] \"(\S+ .*? \S+)\" (\d{3}) ([\d|-]+) "([^"]*)" "([^"]*)"
```

一次性创建一个正则表达式可能会很棘手和复杂。然而，我们已经将每个部分分开并创建了一个正则表达式。在所有这些结束时，我们所要做的就是将所有这些部分组合在一起。

我们现在已经准备好继续编写我们的应用程序。

### 解析每个 Apache 日志行

我们现在知道了我们想要使用的正则表达式，所以我们需要做的就是将（`#1`）正则表达式添加到代码中，对每一行进行正则表达式匹配（`#2`），并获得结果（`#3`）。目前我们只需要在控制台中输出结果（`#4`）。代码如下所示：

```js
var fs = require('fs');

fs.readFile('access.log', function (err, logData) {

  if (err) throw err;

  var text = logData.toString(),
    lines = text.split('\n'),
    results = {},
    regex = /^(\S+) (\S+) (\S+) \[(\d{2}\/\w{3}\/\d{4}:\d{2}:\d{2}:\d{2} [\+|-]\d{4})\] \"(\S+ .*? \S+)\" (\d{3}) ([\d|-]+) "([^"]*)" "([^"]*)"/; //#1

  lines.forEach(function(line) {

    results = regex.exec(line); //#2

    for (i=0; i<results.length; i++){ //#3
      console.log(results[i]); //#4
    }

  });  //#5
});
```

### 提示

**这是唯一使正则表达式与 Node.js 配合工作的方法吗？**

在这个例子中，我们使用了 JavaScript 正则表达式，这是我们在整本书中学到的。然而，Node.js 还有其他可以在使用正则表达式时让我们的生活更轻松的包。`node-regexp`包是提供在 Node.js 中使用正则表达式的新方法的包之一。值得一看，并花一些时间在[`www.npmjs.com/package/node-regexp`](https://www.npmjs.com/package/node-regexp)上进行尝试。

我们将在接下来的两节中继续完成我们的代码。

#### 为每一行创建一个 JSON 对象

让我们尝试对 Apache 日志的每一行做一些更有用的事情。我们将为每一行创建一个**JavaScript 对象表示**（**JSON**）对象，并将其添加到一个数组中。为了包装我们的应用程序，我们将把 JSON 内容保存到一个文件中。

### 注意

要了解更多关于 JSON 的信息，请参考[`www.json.org/`](http://www.json.org/)。

所以在正则表达式声明之后（它在`var`声明内部），我们将添加一个新变量，用于保存我们将创建的**JSON**对象的集合：

```js
jsonObject = [],
row;
```

与前一节代码中的`#3`和`#4`行不同，我们将放置以下代码：

```js
if (results){
  row = {
    ip: results[1],
    available: results[2],
    userid: results[3],
    time: results[4],
    request: results[5],
    status: results[6],
    size: results[7],
    referrer: results[8],
    userAgent: results[9],
  }

  jsonObject.push(row);
}
```

这段代码将验证正则表达式的执行是否产生了任何结果，并将创建一个名为**row**的 JSON 对象。然后，我们只需要将 JSON 对象添加到`jsonObject`数组中。

接下来，我们将构建 Node.js 应用程序的最后一部分。我们将创建一个 JSON 文件，其中包含我们创建的 JSON 数组。我们需要将以下代码放在代码的第`#5`行，就像在前一节中看到的那样：

```js
var outputFilename = 'log.json';
fs.writeFile(outputFilename, JSON.stringify(jsonObject, null, 4), function(err) {
  if(err) {
    console.log(err);
  } else {
    console.log("JSON saved to " + outputFilename);
  }
});
```

### 注意

要了解更多关于`writeFile`函数的信息，请参考[`nodejs.org/api/fs.html#fs_fs_writefile_filename_data_options_callback`](http://nodejs.org/api/fs.html#fs_fs_writefile_filename_data_options_callback)。

结果将是一个类似以下内容的 JSON：

```js
[
  {
    "ip": "127.0.0.1",
    "available": "-",
    "userid": "jan",
    "time": "30/Jun/2004:22:20:17 +0200",
    "request": "GET /cgi-bin/trac.cgi/login HTTP/1.1",
    "status": "302",
    "size": "4370",
    "referrer": "http://saturn.solar_system/cgi-bin/trac.cgi",
    "userAgent": "Mozilla/5.0 (X11; U; Linux i686; en-US; rv:1.7) Gecko/20040620 Galeon/1.3.15"
  }
  //more content
]
```

#### 在表中显示 JSON

最后一步是创建一个简单的 HTML 页面来显示 Apache 日志内容。我们将创建一个 HTML 文件，并将以下代码放入其中：

```js
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Log</title>
    <link rel="stylesheet" href="http://netdna.bootstrapcdn.com/bootstrap/3.0.3/css/bootstrap.min.css">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/bootstrap-table/1.5.0/bootstrap-table.min.css">
    <script src="img/jquery.min.js"></script>
    <script src="img/bootstrap-table.min.js"></script>
    <style>
      body{
        margin-top: 30px;
        margin-right: 30px;
        margin-left: 30px;
      }
    </style>
  </head>
```

上述代码包含所需的 JavaScript 和 CSS 导入，以便我们可以显示 Apache 日志。

### 注意

此示例的表是使用 Bootstrap 表创建的。有关其用法和示例的更多信息，请访问[`wenzhixin.net.cn/p/bootstrap-table/docs/examples.html`](http://wenzhixin.net.cn/p/bootstrap-table/docs/examples.html)。

下一个也是最后一个代码片段是 HTML 的正文：

```js
  <body>
    <table data-toggle="table" data-url="log.json" data-cache="false" data-height="400" data-show-refresh="true" data-show-toggle="true" data-show-columns="true" data-search="true" data-select-item-name="toolbar1" >
      <thead>
        <tr>
          <th data-field="ip">IP</th>
          <th data-field="time">Time</th>
          <th data-field="request">Request Info</th>
          <th data-field="status">Status</th>
          <th data-field="size">Size</th>
          <th data-field="referrer">Referrer</th>
          <th data-field="userAgent">User Agent</th>
        </tr>
      </thead>
    </table>
  </body>
</html>
```

正文将包含一个表，该表将读取`log.json`文件的内容，解析并显示它。

为了能够在浏览器中打开 html 文件，我们需要一个服务器。这是因为我们的代码正在使用 Ajax 请求来加载 Node.js 应用程序创建的 JSON 文件。由于我们已经安装了 Node.js，我们可以使用它最简单的服务器来执行我们的代码。

在终端中，执行以下命令以安装服务器：

```js
npm install http-server –g

```

然后，将目录更改为为 HTML 文件创建的目录：

```js
cd chapter05

```

最后，启动服务器：

```js
http-server

```

您将能够在`http://localhost:8080/` URL 中看到结果。我们可以在以下图片中看到最终结果：

![在表中显示 JSON](img/2258OS_05_10.jpg)

我们还可以切换表中的结果并查看完整的数据：

![在表中显示 JSON](img/2258OS_05_11.jpg)

现在我们已经完成了我们的示例 Node.js 应用程序，它已经读取并解析了 Apache 日志文件，并可以以更友好的方式显示出来。

# 摘要

在本章中，我们学习了如何创建一个简单的 Node.js 应用程序，读取 Apache 日志文件并使用正则表达式提取日志信息。我们能够将我们在本书前几章中所学到的知识付诸实践。

我们还学到，要创建一个非常复杂的正则表达式，最好是分步进行。我们学到，创建正则表达式时可以非常具体，也可以更通用，从而达到相同的结果。

随着**EcmaScript**的新版本（EcmaScript 6）的创建（将为 JavaScript 添加许多新功能），熟悉与正则表达式相关的改进是很好的。有关更多信息，请访问[`www.ecmascript.org/dev.php`](http://www.ecmascript.org/dev.php)。

希望您喜欢这本书！玩得开心，创建正则表达式！
