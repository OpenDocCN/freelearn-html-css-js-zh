# 第八章：JavaScript 实现、语法基础和变量类型

JavaScript（JS）是一种主要用于生成动态和用户交互式网页的编程语言。大量网站和所有最新的互联网浏览器（客户端）都使用并支持 JavaScript。它是每个网络开发者必须学习的技术堆栈的一部分；这些包括 HTML（内容）、CSS（内容的呈现）和 JS（当用户与网页交互时内容的动作）。

以前，JavaScript 的使用仅限于输入验证。例如，如果用户输入无效信息或尝试提交缺少信息的表单，它会产生一个错误消息。现在，JS 被认可为一种完整的编程语言，能够处理复杂的计算，并处理客户端窗口的所有方面。JS 通过使其更具交互性来增强网页。例如，可以将散布的缩略图页面转换为漂亮的图片库，可以在不强制客户端反复重新加载的情况下加载网站内容，可以以优雅的方式处理各种错误处理，可以创建用户投票并在网站上查看结果，以及通过 JavaScript 的力量在移动中与 HTML 元素互动等等。

JavaScript 非常快——真的非常快。它对用户操作，如点击、双击、调整大小、拖动、滚动等，能立即做出响应。JavaScript 的语法与 Java 语言类似。*JavaScript*这个名字让很多人感到困惑。只有 JS 的语法与 Java 语言的语法相似。否则，它与 Java 编程语言根本没有任何关系。

以下是一些 Java（编程语言）和 JS（脚本语言）之间的关键区别：

+   JavaScript 嵌入到 HTML 中，并在浏览器中运行，而 Java 需要**Java 虚拟机**（**JVM**）来执行；然而，Java 小程序可以在浏览器中运行。

+   JavaScript 被称为客户端语言，因为它在客户端的浏览器中运行，但现在，Node.js 是 JS 进入服务器端的大门。另一方面，Java 在 Web 服务器上运行，例如，**Apache Tomcat**。

+   JavaScript 通常由客户端解释，而 Java 则由 JVM 编译然后执行。

+   JavaScript 是由**Netscape**（现在是 Mozilla）创建的，而 Java 是由**Sun Microsystems**（现在是 Oracle）开发的。

# JavaScript 的历史

Netscape（现在为 Mozilla）的 Brendan Eich 在 1995 年开发了 JavaScript。最初，它被命名为 **Mocha**，后来其名称改为 **Livescript**，最终更名为 JavaScript。在发布后，Microsoft 引入了 **Jscript**——他们自己的 JavaScript 版本，包含在 Internet Explorer 中。Netscape 将 JavaScript 提交给 **欧洲** **计算机制造商协会**（**ECMA**）进行标准化和规范。标准化版本被命名为 **ECMAScript**。ECMAScript 是由 ECMA 国际标准化的脚本语言规范。

ECMAScript V5 引入了一种新的模式，称为 **严格模式**；这引入了更好的、彻底的错误检查，以避免导致错误的构造。许多来自第三版本的不确定性都被移除，并添加了现实世界的实现。

ECMAScriptV6 添加了许多新的语法来编写复杂的程序。它引入了类和模块，扩展了 V5 的语义。迭代器也在此版本中添加，以及许多类似 Python 的语义，包括代理、箭头函数、生成器和生成器表达式、映射、集合等。

## JavaScript 的演变

在将 JavaScript 标准化为 ECMAScript 之后，下一步是实现对 **文档对象模型**（**DOM**）的完全控制。

网页浏览器通过 DOM API 将 HTML 解析为 DOM。DOM 是客户端对网页的理解。所有元素/节点都被转换为称为 **DOM 树**的树状结构。可以使用不同的方法（如 `getElementById()` 和 `getElementsByName()`）实时操作此树的结构成员。我们在开发者工具中看到的 HTML 源代码，如 **Inspect Element**、**Fire Bug** 等，都是 DOM 视图。这些开发工具也极大地帮助我们在瞬间修改我们的 DOM。

之后，**异步 JavaScript 和 XML**（**AJAX**）被引入以实现异步应用程序。数据被发送到并从网络服务器接收，而不需要重新加载或刷新整个 DOM 树（异步）。

随着 JavaScript 的发展，开发了不同的 JS 库以简化 DOM 操作。其中一些流行的库是 jQuery、Prototype 和 Mootools。这些库的主要功能是处理 DOM 操作、动画和 AJAX。最著名且广泛使用的 JS 库是 jQuery。此外，还开发了一个名为 **Node.js** 的服务器端 JavaScript 版本。

在这本书中，你将了解 JavaScript、其历史和演变。你将学习其实现和语法。还提供了函数引用和如何在不同情况下使用的完整示例。

# JavaScript 实现

由于 JavaScript 和 ECMAScript 在类似的应用场景中被使用，JavaScript 比 ECMAScript 提供了更多功能。它由以下三个部分实现：

+   核心 JavaScript（ECMAScript）

+   文档对象模型（DOM）

+   浏览器对象模型（BOM）

## 核心 JavaScript（ECMAScript）

JavaScript 支持移动设备和桌面计算机；这一特性使其成为一种跨平台脚本语言。然而，如果单独使用，它并不太有用，这就是为什么它通常与服务器端语言一起使用，以创建强大且交互式应用程序。它可以轻松地集成到网络浏览器环境中，使用户能够完全控制浏览器对象和事件。

JavaScript 的核心功能也称为 ECMAScript。ECMAScript 实际上不依赖于浏览器或环境。它是一组核心语言元素，用于不同的环境，例如 **ScriptEase** 和 **Flash** **动作脚本**。因此，我们可以说 ECMA Script 包含以下内容的定义：

+   语言语法

+   关键字/保留字

+   数据类型

+   语句

+   运算符

+   控制结构

+   对象（例如，数组、日期、数学等）

因此，ECMAScript 定义了脚本语言的所有函数、方法、属性和对象。其他脚本语言，如 JavaScript，是 ECMAScript 的实现。

ECMAScript 在不同的浏览器中有不同的实现，之后 DOM 和 BOM 才被包含。

## 文档对象模型 (DOM)

DOM 是 HTML 和 XML 文档的 **应用程序** **编程接口** (**API**)。它是网络文档的逻辑结构。它是文档中存在的节点的高层次树视图。浏览器将文档转换为 DOM，并且只理解它。DOM 允许开发者控制节点，并实时更改文档。

例如，这是一个简单的网页：

```js
<html>
  <head>
    <title>My Web Document</title>
  </head>
  <body>
    <p>Javascript Reference!<p>
  </body>
</html>
```

## 浏览器对象模型 (BOM)

当 DOM 处理文档时，**浏览器对象模型** (**BOM**) 处理浏览器窗口。BOM 允许程序员在浏览器窗口上执行多个操作，这些操作与 HTML 文档本身没有直接关系，也没有影响。BOM 还处理浏览器对象，例如历史记录、屏幕、导航器和位置。

### 注意

BOM 在不同的客户端中有不同的实现。

## 客户端 JavaScript

JavaScript 主要被认知为一种客户端脚本语言。任何用于编写在客户端（任何网络浏览器，如 Internet Explorer、Safari、Google Chrome、Mozilla Firefox、Opera 等）上执行程序或脚本的编程语言都称为 **客户端脚本语言**。这些脚本使你的 HTML 看起来交互式和动态。JavaScript 允许用户与网页内容以及客户端窗口本身进行交互。

### 将 JavaScript 添加到网页中

大多数情况下，JavaScript 代码被嵌入到 HTML 中的`<script>`标签内，可以通过查看页面源代码来查看。将 JavaScript 嵌入到页面中的良好实践是将所有脚本放在一个单独的文件中（一个具有`.js`扩展名的 JavaScript 文件）。然后，此文件可以包含在页面中。当页面被解释时，它将被视为与在同一页面上嵌入的相同。

### 小贴士

JavaScript 文件必须具有`.js`扩展名。

由于客户端始终期望 HTML，因此任何其他内容，如样式或脚本，都必须包含在其特定的标签内；例如：

```js
<style>
  /* all css code */
</style>
<script>
  //all scripts
</script>
```

当浏览器的渲染引擎发现任何`<script>`标签时，JavaScript 解释器就会被调用。

大多数时候，JavaScript 位于网络文档的`<head>`标签内：

```js
<html>
  <head>
    <title>My First JavaScript Example</title>
    <script type="text/javascript">
      alert("Hello earthlings!");
    </script>
  </head>
  <body>
  </body>
</html>
```

`type`属性指定这些`<script>`标签内封装的代码是纯 JavaScript 代码。不强制提及`type`属性。前面的代码也可以不使用它来编写，并且将按完全相同的方式工作。

虽然可以在 HTML 文档的任何位置放置`<script>`标签，但另一个良好的实践是在文档末尾关闭`</body>`标签之前将其放置，以确保在执行任何脚本之前所有页面都已加载。

## 加载外部 JavaScript 文件

就像外部 CSS 文件被包含到网页中一样，JavaScript 也可以从外部 JavaScript 文件中包含到网页中。让我们看看以下 HTML 代码：

```js
<html>
  <head>
    <title> My First JavaScript Example </title>
    <script src="img/external_javascript_Planets.js"></script>
  </head>
  <body>
  </body>
</html>
```

`src`属性与用于图像`<img>`标签的属性非常相似。它只是提到了外部 JavaScript 文件的引用路径，这些文件稍后将被包含到页面中。不要在导入外部 JS 文件的`<script>`标签中添加任何其他 JavaScript 代码。如果需要，请放置另一个`<script>`标签。

也可以包含多个外部 JS 文件，如下所示：

```js
<html>
  <head>
    <title> My First JavaScript Example </title>
    <script src="img/external_javascript_Moon.js"></script>
    <script src="img/external_javascript_Sun.js"></script>
    <script src="img/external_javascript)Stars.js"></script>
  </head>
  <body>
  </body>
</html>
```

### 小贴士

在使用外部 JavaScript 库（如 jQuery 和 MooTools）的情况下，必须注意文件包含的顺序。

### 在 JavaScript 中编写我们的第一个程序

别担心，这绝对不是一门陌生的语言。它是一种高级语言，由于其语句、关键字和其他语法基于基本英语，因此可以很容易地理解。学习任何编程语言的最简单方法是跳进去并玩其代码。为了在 JavaScript 中入门，我们将编写一个非常基础的 JavaScript 程序。

#### 前提条件

以下是我们编写第一个 JavaScript 程序所需的前提条件：

+   一个网络浏览器

+   一个文本/代码编辑器

+   一个网络文档

#### 如何操作

1.  在您舒适的任何代码编辑器中创建一个`index.html`文件。将以下代码复制并粘贴到您的文档中。这是一个简单的 HTML 代码：

    ```js
    <html>
      <head>
        <title> My First JavaScript Example </title>
      </head>
      <body>
        <h1>The Planetary System</h1>
      </body>
    </html>
    ```

1.  在头部部分在关闭`</head>`标签之前创建一个`<script>`标签：

    ```js
    <html>
      <head>
        <title> My First JavaScript Example </title>
        <script type="text/javascript"></script>
      </head>
      <body>
        <h1>The Planetary System</h1>
      </body>
    </html>
    ```

1.  在脚本标签内插入`alert("hello world"); document.write('<h2>This text has been added by JavaScript!</h2>');`。

1.  `alert`属性是一个 JavaScript 函数，它创建一个带有内部引号和括号中消息的弹出窗口。该消息被视为传递给警告函数的字符串。同样，`document.write`是一个 JavaScript 函数，它将给定的字符串输出到网页上。

    ```js
    <html>
      <head>
        <title> My First JavaScript Example </title>
        <script type="text/javascript">
          alert("Hello Earthlings");
          document.write('<h2>Greetings from Mars!</h2>');
        </script>
      </head>
      <body>
        <h1>The Planetary System</h1>
      </body>
    </html>
    ```

1.  双击`index.html`文件以执行并在浏览器中打开。将出现一个带有警告信息的弹出窗口。

点击**确定**以关闭警告框，网页将显示。

这是你进入 JavaScript 世界的第一步，尽管它非常基础，并没有展示 JavaScript 的真实力量。我们将在接下来的章节中练习高级 JavaScript 代码。

## 服务器端 JavaScript

服务器负责以编程语言提供网页。客户端向服务器发送带有命令的请求，服务器响应该客户端请求。服务器端编程是指运行在服务器上的程序。我们将在第十一章中进一步详细研究，*扩展 JavaScript 和 ECMA Script 6*。

术语*服务器端*意味着网页的控制由 Web 服务器而不是网页处理。Web crossing 运行该脚本，并以 HTML 的形式将信息发送到每个用户的浏览器。

**Rhino**和**Node**都常用于创建服务器。服务器端脚本不会下载到客户端浏览器。

### 服务器端

服务器端指的是在客户端-服务器关系中的计算机网络中由服务器执行的操作。术语*服务器端*也可以理解为浏览器之外的一切。

### 客户端

任何在*客户端*运行的东西意味着它在 Web 浏览器内部运行。

### 使用 Rhino 进行脚本编写

Rhino 是一个用 Java 语言开发和编写的 JavaScript 引擎/解释器，由 Mozilla 基金会管理，作为开源软件([`www.mozilla.org/rhino`](http://www.mozilla.org/rhino))。它使 JavaScript 程序元素能够访问完整的 Java API。

#### 描述

Rhino 是用纯 Java 编写的 JavaScript 开源实现。Rhino 用于为最终用户提供脚本。基本上，Rhino 将 JavaScript 脚本转换为类。它用于 JavaScript 的服务器端编程。

MDN 提到以下内容：“使用 Java 脚本有许多用途。它允许我们通过利用许多可用的 Java 库来快速编写强大的脚本。”关于`this`语句等内容的更多信息将扩展一般性陈述，并给出`this`语句实现和目的的见解。

我们可以将 Rhino 脚本用作类似于调试器的壳。这个壳以批处理模式运行代码。批处理模式指的是批处理，意味着无需人工干预的自动化处理。批处理是交互式的对立面。

### 注意

由于是开源的，Rhino 是由 Mozilla 提供的免费程序，可以从 [`www.mozilla.org/rhino`](http://www.mozilla.org/rhino) 下载。Rhino 以 JAR 文件的形式分发。要使用命令行启动它，您可以在命令行界面中执行 Rhino JAR 文件：

```js
java -jar rhino1_7R3/js.jar programfiles.js

```

Rhino 1.7R2 版本使用 ECMAScript 3，而 1.7R3 版本的 Rhino 部分使用 ECMAScript 5。Rhino 的最新稳定版本是 1.7R5，该版本于 2015 年 1 月 29 日发布。

这里有一些函数及其用法和描述：

| 函数 | 用法 | 描述 |
| --- | --- | --- |
| `print` | `print(x)` | 这是一个全局的打印函数，用于在控制台打印 |
| `version` | `version(170)` | 这用于告诉 Rhino 我们想要 JS 1.7 的功能 |
| `load` | `load(file1,file2…)` | 这将加载并执行一个或多个 JavaScript 代码文件 |
| `readFile` | `readFile(file)` | 这将读取一个文本文件，并返回其内容作为字符串 |
| `readUrl` | `readUrl(url)` | 这将读取 URL 的文本内容，并将输出作为字符串返回 |
| `spawn` | `spawn(f)` | 这将运行 `f()` 或 `load` 并在新线程中执行 `f` 文件 |
| `runCommand` | `runCommand(cmd,[args..])` | 这将运行一个系统命令，带有零个或多个命令行参数 |
| `quit` | `quit()` | 这将使 Rhino 退出 |

这里是一个简单的示例，它显示 `strNote` 字符串中的消息：

```js
Dim strNote
strNote = "Welcome to Rhino"
Rhino.printstrNote
```

您可以在 [`www.rhinoscript.org/`](http://www.rhinoscript.org/) 找到一些有用的脚本。

### Node.js

Node.js 是 JavaScript 的一个实现，允许 JavaScript 在浏览器之外运行并执行基于 OS 和网络的任务。它是一个 JavaScript 的运行时接口。

Node 使用 Google V8，它实现了 ECMAScript 5 标准，这意味着 JavaScript 和 Node 的语法之间没有太大差异。例如，如果您想在控制台打印出 "Hello World"，您将编写以下代码：

```js
console.log("Hello from Mars!"); //This is same for JavaScript as well.
```

#### 描述

Node.js 是一个运行在 Google Chrome V8 引擎上的超级快速接口。Node.js 易于使用，快速且可扩展。像 JavaScript 客户端编程一样，Node.js 提供了抽象。因此，通过这种抽象，Node.js 可以处理大量的代码。

这里是一个 Node.js 的服务器端编码示例：

```js
var http=require("http");
http.createServer(function(request,response){
  response.writeHead(200,{"content-type":" text/plain"});
  response.write("Hello from Mars!");
  response.end();
}).listen(8080)
```

# 语言语法

语言语法基本上是一种通信手段。在编程语言中，这是一种与算法正式通信的方式，无论是从算法到程序员还是从程序员到机器。这是因为机器根据给定的指令工作。

这些指令旨在以特定格式编写，以便机器能够正确理解并编译它。这种特定格式由编程语言的某些一般规则定义，称为语言语法。

计算机包含要执行的指令列表。每种计算机语言都有不同的语法和规则。要使用不同的语言，我们必须了解它们的语言语法，如下所示：

+   语法（一组符号和规则）

+   语义学（用于术语到术语的转换）

+   语义学（语言的具体结构）

类似地，JavaScript 中的其他语言也有独特的语法。JavaScript 不是 Java 语言；这两个是不同的语言，两种语言的语法也不同。JavaScript 是一种强大且表达性强的语言。JavaScript 的语句以分号结尾进行分隔。

对于有面向对象编程概念的初学者来说，JavaScript 的语言语法非常容易。JavaScript 包含在`<scripts>…</script>`标签之间。在任何网络浏览器中，这些都被视为 HTML 标签。

你可以将你的 JavaScript 代码放置在网页的任何位置，但我更喜欢使用`<head>`标签来定义你的脚本。因此，你的脚本结构如下所示：

```js
<head>
  <script>
    ……………..
  </script>
</head>
```

基本上，当在`<head>`标签中写入`<script>`标签时，它告诉浏览器在网页加载时需要首先执行这些脚本。尽管如此，将 JavaScript 放在网页底部是一个好的实践。这个`script`标签有两个重要的属性。

## 语言

`language`属性将告诉您正在使用哪种语言属性；通常，它是 JavaScript。如果该属性不存在，则默认值为 JavaScript。

## 类型

`type`属性与前面的解释相同。这不是一个必需的属性。其值设置为`text/javascript`，这表明正在使用这种脚本语言。

```js
<script language=" javascript" type="text/javascript">
  .Script code.
</script>
```

# 字符集

字符集基本上是一组由计算机硬件和软件识别的字符。它用一个数字表示。早期 ASCII 被用作网页的标准字符集编码。在 HTML5 和 XML 出现之后，解决了许多字符编码问题。JavaScript 为不同类型的语言及其字符提供支持。字符集属性显示外部文件中的字符编码。对于 HTML5，默认字符集编码是 UTF-8。

字符集编码有一些常见的值，如下所示：

+   **ISO-8859-1**：这是用于编码拉丁字母

+   **UTF-8**：这是用于与 ASCII 兼容的 Unicode 编码

例如，如果你的前端页面是西班牙语，而你未在页面中使用字符集属性，那么它将无法清楚地显示一些特殊的西班牙语字符。为此，你必须在页面的顶部`<head>`标签中声明一个字符集属性，如下所示：

```js
<meta charset="utf-8">?
```

### 注意

对于这一点，您也可以更改您的网络服务器配置以作为 UTF-8 服务。或者，我们可以向服务器端脚本发送内容类型头。

在 JavaScript 中，`charset` 属性返回当前文档的字符编码。从文档中读取 `characterSet` 属性的语法是 `document.characterSet;`。

我们可以在我们网站的父页面的 `<script>` 标签中定义 `characterSet` 属性，如下所示：

```js
<script type="text/javascript"charset="utf-8"></script>
```

### 注意

另一种方法是将 UTF-8 字符集添加到服务器配置文件（`.htaccess`）中，如下所示：

```js
AddCharset UTF-8

```

因此，字符集编码将应用于您应用程序中的所有 JavaScript 页面。

## 大小写敏感性

HTML 不是大小写敏感的语言，但 XML 和 JavaScript 是大小写敏感的语言。

### 描述

JavaScript 是大小写敏感的语言。JavaScript 中的大小写敏感性不仅适用于变量名，还适用于 JavaScript 关键字和 JavaScript 的事件处理器。例如，如果您有 `firstname` 和 `firstName` 变量，那么这些将是两个不同的变量。在 JavaScript 中，在调用函数时，您必须按照定义时的确切方式写出其名称，匹配字母大小写。

在 JavaScript 的原始方法中常用的一个流行约定是使用驼峰式命名法，其中短语以第一个单词的首字母小写开始，每个后续单词的首字母大写，这使得阅读更加容易。

一些函数的示例如下：

+   `toUpper();`

+   `toArray();`

### 注意

JavaScript 中的关键字都是小写的，例如 `while`、`for`、`if` 等等。

JavaScript 有一些内置的函数；它们接受字符串并将它们转换为大写或小写。这可以在输入具有不同大小写时使字符串的处理更容易操作。以下是可以由这些函数接受的两个参数：

+   `.toUpperCase()`

+   `.toLowerCase()`

`.toUppercase()` 函数将字母转换为大写，而 `.toLowerCase()` 函数将字母转换为小写。这些都是 JavaScript 的内置且大小写敏感的函数。

## 空白和换行符

空白和换行符用于以整洁和一致的方式格式化和缩进代码，以便代码易于阅读和理解。

### 描述

空格、制表符、不是字符串一部分的换行符称为空白。JavaScript 在程序中的标记之间删除空白和换行符，当脚本执行时，字符串中的空格和换行符不会被删除。

在任何文本中，都有三种类型的换行符：`\r\n` 或 `\r` 或 `\n`。

这些换行符在不同的操作系统中出现：

+   `\r\n` 通常在 Windows 上创建

+   `\n` 通常在 OS X 上创建

如果您想从任何文本中删除换行符，那么我们必须处理所有类型的换行符，因为文本可能来自这三个来源：Windows、Linux 和 Mac。

我们有几种方法可以去除任何文本中的换行符。例如，我们有以下方法：

```js
varabc = abc.replace(/(\r\n|\n|\r)/gm,"");
```

在此代码中，我们使用了正则表达式来从我们的文本中去除换行符。这将去除换行符 `\r\n`，然后是 `\n`，最后是 `\r`。我们在正则表达式的末尾使用了后缀 `gm`，因为 `m` 表示应该从所有文本中去除换行符，而 `g` 表示应该多次执行此操作。

正则表达式在 第八章 中详细讨论，*JavaScript 表达式、运算符、语句和数组*。

现在，文本之间存在一些空格；您可以使用 JavaScript 来去除这些空格。JavaScript 有不同的函数可以用来从字符串中去除所有前导空格；您可以使用以下函数：

```js
str = str.replace(/^\s+|\s+$/g,'');
```

这段代码使用正则表达式，检查字符串开头和结尾的多个空格。

在这个正则表达式中，`g` 表示在检查文本时应执行全局搜索。还有更多方法，例如以下方法：

+   `string.replace()` 函数用于将所有前导空格替换为空字符串

+   `str.trim()` 函数用于从字符串的两端去除空格

## Unicode 转义序列

每个 Unicode 转义序列由六个字符组成，具有明确的语法：四个字符跟在 `\u` 之后。较小的代码点前面有前导零，但序列的长度保持不变。例如，`\u00a9` 有两个前导零。同样，版权符号可以表示为 `\u00A9`。

任何小于 65536 的代码点的字符都可以使用其代码点的十六进制值进行转义，前面加上 `\u`。

### 注意

代码点（也称为 **字符代码**）是特定 Unicode 字符的数值表示。

### 描述

JavaScript Unicode 转义序列允许您在字符串中放置特殊字符。支持 JavaScript 的浏览器可以使用 escape 函数。

要在同一个字符串中添加下一行，我们可以使用 `\n`。让我们看看以下代码：

```js
alert("Welcome to JavaScript. \n We hope you love it!")
```

要在字符串中添加引号，我们可以使用如下方式：

```js
var myquote="Jinnah said \"Impossible is a word unknown to me.\""
```

这个 Unicode 转义序列以反斜杠 (`\`) 开头，后跟字符。这个反斜杠是转义序列字符。要插入一个反斜杠本身，只需在下一个反斜杠之前添加另一个反斜杠 "`\`"。也称为双反斜杠 (`\\`)。

您可以通过 `\uaaaa` 指定一个 Unicode 字符，其中 `aaaa` 是一个十六进制数。因此，Unicode 转义字符可以表示一个 16 位字符。

Unicode 转义序列表示一个 Unicode 序列，如下所示：

+   反斜杠 (`\`)

+   字符 (`u`)

+   十六进制数 (`0-9`)(`a-f`)

例如，单词 cat 将表示为 `\u732b`。

Unicode 转义序列也可以用于注释和字面量。考虑以下示例：

```js
var x = "http\\u00253A\\u00252F\\u00252Fexample.com";
```

### 注意

这种字符转义序列的十六进制部分不区分大小写，这意味着 `\u041a` 和 `\u041A` 是等效的。

你也可以表示一个转义序列来表示一个 Unicode 字符 ["`\u03b1`"] 作为 ["`a`"]。例如，字符 `é` 的 Unicode 转义是 `\u00E9`，以下两个 JavaScript 字符串是相同的：

```js
"cafn" === "caf\u00e9" // => true
```

`==` 操作符在测试它们是否相同之前会尝试将值转换为相同的类型；而 `===` 则不会这样做。它要求对象类型相同才能相等。

这里有一些字符转义序列的示例：

| Unicode | 转义序列 | 含义 |
| --- | --- | --- |
| `\u000B` | `\v` | 垂直制表符 |
| `\u000A` | `\n` | 换行 |
| `\u0009` | `\t` | 空白字符 |
| `\u000C` | `\f` | 表单字段 |
| `\u000D` | `\r` | 行终止符 |
| `\u0020` |   | 空白字符 |

## 规范化

Unicode 标准（可在 [`unicode.org/standard/standard.html`](http://unicode.org/standard/standard.html) 找到）定义了编码字符的最可接受模式以及规范化方法。JavaScript 假设它读取和解释的代码具有所有 Unicode 表示的规范化。ECMAScript V6 有一个字符串原型函数 (`string.prototype.normalize()`) 来修复任何未处理的编码。

### 注意

ECMAScript 6 引入了 `String.prototype.normalize()` 方法，负责处理 Unicode 规范化。

## 标识符

标识符用于给函数和变量命名。这些标识符有一些规则，并且这些规则对所有编程语言都是相同的。标识符可以包含字母和完整的 Unicode 字符集的摘要。这些 JavaScript 标识符包括变量、对象、函数、属性、方法、事件等。

这里是标识符的规则：

+   标识符不能以数字开头

+   标识符可以包含数字、字母、下划线和美元符号 (`$`)

+   它可以是任何长度

+   这些是区分大小写的

+   我们不能使用保留词作为标识符

### 小贴士

我们还可以使用 Unicode 字符作为标识符。例如，Unicode 转义序列 `\uaaaa` 也可以用作标识符。我们必须避免使用全局方法或属性作为标识符。

在编写标识符时，一个最佳实践是使用一个单词并使用驼峰式命名法。考虑以下示例：

+   myNote

+   myNewNotebook

+   $sum

`var` 变量是一个有效的标识符名称，但由于它是保留词，因此不是一个有效的标识符。如果你使用一个有效的标识符，那么你的浏览器将正确处理它；然而，如果你使用一个无效的标识符，你的浏览器将显示警告，将其识别为错误。

## 保留关键字

保留字是 JavaScript 中的关键字。有一些字不能用作函数或变量名。这些保留字也是标识符。这些保留字是为 JavaScript 引擎保留的。如果你将这些保留字用作函数、变量或方法名，那么你的浏览器将显示警告，并且你的脚本可能会失败。

保留字基本上是具有特殊意义的 JavaScript 关键字，例如`break`、`case`、`do`、`delete`、`else`等等。

有三种类型的保留字：

+   受保护的保留字

+   新保留字

+   未来保留字

### 受保护的保留字

受保护的保留字不能用作变量或函数名。如果我们使用这些字，那么在编译时将会出现错误。以下是一些受保护的保留字的例子：

| `Break` | `case` | `catch` |
| --- | --- | --- |
| `class` | `const` | `Continue` |
| `Debugger` | `Default` | `Delete` |
| `do` | `Else` | `Export` |
| `Extends` | `False` | `If` |
| `Import` | `in` | `var` |

### 新保留字

JavaScript 还发布了新的关键字，这些保留字在当前版本的 JavaScript 中具有特殊意义。这些保留字也可以用作标识符。一旦你将其声明为标识符，那么它就会忘记它曾经是一个关键字。以下是一些新保留字的例子：

| `Abstract` | `Boolean` | `byte` |
| --- | --- | --- |
| `char` | `enum` | `final` |
| `doubke` | `implements` | `int` |
| `interface` | `internal` | `long` |
| `set` | `short` | `static` |
| `uint` | `ulong` | `ushort` |

### 未来保留字

未来保留字是为 JavaScript 未来的扩展而提出的。这些字在当前版本的 JavaScript 中也被用作标识符。当你选择一个字作为标识符时，也很重要的是要注意它是否已经是 JavaScript 保留字的名字。以下是一些未来保留字的例子：

| `asset` | `event` |
| --- | --- |
| `namespace` | `require` |
| `transient` | `violate` |
| `ensure` | `goto` |
| `native` | `synchronized` |
| `use` | `Invariant` |

## 注释

在任何编程语言中，注释都用于使你的代码可读。在 JavaScript 中，注释用于解释 JavaScript 代码。

注释的另一个用途是写入它们，以在我们的 JavaScript 代码中添加提示、建议和警告，这样如果除了开发者之外的其他人想要更改或修改你的脚本，他们可以很容易地进行修改。在脚本中使用注释被认为是最佳实践。注释还用于禁用代码的一些部分执行。在调试中，注释非常有帮助，因此它是开发者的宝贵调试工具。

你可以在你的 JavaScript 脚本中放入三种不同类型的注释：

+   多行注释

+   单行注释

+   HTML 注释的起始序列

### 多行注释

我们可以在注释边界之间写多行；这段代码将不会被执行：

```js
/* this is a comment */
```

### 单行注释

在行首添加两个正斜杠来注释它。`//`和行尾之间的文本将被注释，并且不会编译。双正斜杠用于一次注释一行：

```js
// this is comment
```

### HTML 注释的开头序列

JavaScript 将 HTML 注释的开头序列，即`<!--`视为`//`注释。它不识别 HTML 注释的结尾序列，即`-->`。HTML 样式的注释通常不在 JavaScript 代码块中使用，因为`//`更为方便。建议不要在 JS 中使用 HTML 注释，因为这已经是一种旧习惯了。它们被用来隐藏与旧浏览器不兼容的 JavaScript 代码。

## 字面量

字面量用于表示 JavaScript 中不同数据结构的值；例如：

+   `3.14`

+   `to be or not to be`

在接下来的章节中，我们将介绍 JavaScript 的字面量类型。

### 对象字面量

对象字面量用于在对象中保存值。

#### 描述

对象字面量包含逗号分隔的属性名和关联值的列表对，并且它们被括在`{}`内。对象字面量也可以没有值。在对象字面量中，每个`name:value`对由逗号分隔。

下面是一个对象字面量的例子：

```js
varbookObj={
  bookName="The Kite Runner",
  CoverImage="KiteRunner1.png",
  Author="KhaledHoesenni",
  bookCategory ="Bestseller"
};
```

### 数组字面量

数组字面量包含表示数组元素的列表表达式。

#### 描述

数组字面量包含数组内的值。它们放在数组括号`[]`内，每个元素由逗号（`,`）分隔。就像对象字面量一样，数组可以是空的，包含`0`个元素。当你创建一个数组字面量时，它指定了值作为其元素，并且其长度由这个数组的参数数量指定。下面是一个数组字面量的例子：

```js
var Planets= ["Earth", "Neptune", "Saturn", "Mars"];
```

### 布尔字面量

如其名所示，布尔字面量具有布尔值，因此其值要么是`true`要么是`false`。

### 整数

整数字面量必须包含仅限于整数的值。

#### 描述

在 JavaScript 中，整数字面量必须包含从 0 到 9 的数字。整数字面量不允许使用逗号和括号。整数可以是负数或正数。如果没有符号，则默认为正数。

### 浮点字面量

浮点字面量用于包含浮点数——具有小数点、分数或指数的数字。

#### 描述

浮点字面量的例子如下：

```js
22.56
-26.35689
18.4e2 //Equivalent to [18.4 x 10²] 
-17.22E3 //Equivalent to [17.22 x 10^-3]
```

### 字符串字面量

字符串字面量，正如其名所示，只包含字符串值，即放在一对引号内的值。

#### 描述

字符串字面量包含放在引号内的字符。您还可以创建一个空字符串，不包含任何内容。可以使用`+`符号将两个或多个字符串连接在一起。可以在字符串中插入特殊字符。一些特殊字符包括`\n`、`\r`、`\b`、`\t`等。

一些字符串字面量的例子如下：

```js
strFirstName= "Jamie";
strLastName="Down";
strMarksObtained="200";
strClassesJoined="Microprocessors, "+ "Microcontrollers";
strAddress= "Michigan University \n United States";
```

## 语句

JavaScript 还支持一组用于识别一组指令并执行任务的语句。JavaScript 中有不同类型的语句，如`if`、`for`、`while`等。使用这些语句的基本目的是评估或检查是否满足某些条件。这些语句对所有编程语言都是通用的。

### 条件语句

如其名所示，条件语句基于条件和`if`以及`then`规则。例如，“如果你努力工作，你会得到回报”。在这里，努力工作是得到回报的条件。

考虑以下示例：

```js
if (age < 20 && age > 12) {
  person = "Teen ager";
}
else {
  person = "Not a Teenager";
}
```

### 循环语句

在循环语句中，在括号内提供一个条件。如果条件评估为`true`，则循环将继续工作；如果条件等于`false`，程序将跳转到循环之后的下一行。一个例外是`do while`循环，它将始终至少执行一次。编程语言中有不同类型的循环；它们如下：

+   `for`循环

+   `for`/`in`循环

+   `while`循环

+   `do/while`循环

这里有一个示例，展示了如何使用`while`循环：

```js
var number = "";
var n = 0;
while (n < 10) {
  number += "\nThe number is " + n;
  n++;
}
console.log(number);
```

循环在第八章中详细介绍，*JavaScript 表达式、运算符、语句和数组*。

### 对象操作语句

JavaScript 中的对象操作语句用于在运行时访问对象的属性和方法以执行特定功能。JavaScript 使用`for...in`和`with`语句来操作对象。

这里有一个示例，展示了如何操作对象`car`：

```js
<script type="text/javascript">
  var x;
  var car = new Object();
  car.brand = "Porche";
  car.model = "2014";
  car.colour = "Black";
  for (x in car) {
    console,log(x + " --- " + car[x] + "<br />");
  }
</script>
```

输出：

```js
brand — Porche
model — 2014
color — Black
```

### 异常处理语句

`try`、`catch`、`throw`和`finally`语句用于 JavaScript 异常处理。让我们看看以下描述：

+   `try`语句用于测试代码中的错误。

+   `catch`语句通常跟在`try`语句后面，捕获并处理`try`块中找到的错误。

+   `throw`语句创建自定义错误或警报。

+   `finally`语句在`try`和`catch`之后执行代码，无论结果如何。通常，这个代码块用于清理和释放资源。

这里有一个示例，展示了如何使用`try`和`catch`块来处理异常：

```js
<!DOCTYPE html>
<html>

  <head>

    <script type="text/javascript">
      <!--
        function Calculate() {
          x = document.getElementById("Val1").value;
          y = document.getElementById("Val2").value;

          try{
            if ( x == 0 ||y==0 ) {
            throw("Divide by zero error." ); 
          }
          else {
            var a = x / y;
            alert("The answer is: " + a );
          }

          catch ( err ) {
            alert("Error: " + err );
          }
        }
      //-->
    </script>

  </head>

  <body>
    <form>
      <p>Enter first value</p>

      <input id="Val1" type="text">
      <p>Enter second value</p>

      <input id="Val2" type="text">
      <p><input type="button" value="Calculate" onclick="Calculate();" /></p>
    </form>

  </body>
</html>
```

## 可选分号

JavaScript 中的分号用于分隔语句。比如说，如果你有两个不同的语句，你只是把它们写下来，那么它们将不会容易理解。所以，JavaScript 使用分号来分隔语句。这将使你的代码更清晰、更有意义。如果你在 JavaScript 中不使用分号，那么它会使你的语句变得复杂，意味着你语句的结尾可能是你语句的开始。

### 注意

在你的代码中放置分号可以帮助你防止许多错误。代码块中缺少分号，在编译时将视为单行代码，从而导致多个错误。

JavaScript 不将换行符视为分号。以下是在 JavaScript 中使用分号的示例：

```js
a=10;
b=20;
a=10; b=20;
```

### 注意

JavaScript 中的分号不是一个终止符；这些只是用于分隔语句的分隔符。

JavaScript 会自动在语句的末尾插入分号。你不应该在闭合的大括号后加分号。同样，有时在圆括号 `()` 后加分号也是一个坏主意，更重要的是，代码将无法运行。

如果你的脚本中有一个 `for` 循环，那么在第一个和第二个语句后加上分号，在第三个语句后不要加分号。这将产生语法错误。例如，如果你有一个这样的 `for` 循环：

```js
for(i=0;i<5;i++;){}
```

上述代码将导致语法错误。正确的方法如下：

```js
for(i=0;i<5;i++){}
```

这里有一些关于分号的规则：

+   当使用赋值运算符时插入分号

+   在赋值运算符的右侧操作数后插入分号

+   在函数的闭合圆括号后插入分号

+   在关键字后插入分号

+   当你声明一个变量时插入分号

# 数据类型

计算机根据一组给定的指令工作；它不能区分数字和字符。例如，如果你写了一些数字，如 `12345`，和一些字符，如 `abcsd`，它不能区分整数和字符。所以，数据类型被用来做这个。数据类型告诉在语句中使用或引用的数据类型。

在所有编程语言（C、C++、Java、JavaScript）中，都使用数据类型。每个数据类型都有存储数据的具体功能。许多经典的计算机编程语言要求你在声明数据对象时指定数据类型。JavaScript 没有这样的要求。同样，一些数据库要求声明存储数据的数据类型。

JavaScript 是一种动态语言。这意味着如果你声明了一个对象，那么它的数据类型可以动态地改变，例如：

```js
var a=16;
a="abc";
a=true;
```

在第一个语句中，数据类型是整数；在第二个语句中，数据类型是字符；在第三个语句中，数据类型是布尔值。所以，在 JavaScript 中，数据类型在程序处理过程中会动态改变。

ECMAScript 标准中定义了六种主要的数据类型。

+   `Null`

+   `Undefined`

+   `Boolean`

+   `String`

+   `Symbol`

+   `Number`

### 注意

JavaScript 将整数和浮点数视为相同。它不区分这两者。JavaScript 中的所有数字都表示为浮点类型值。

在这里，`Null` 和 `undefined` 是 JavaScript 中的简单数据类型，因为每个都定义了单个值。JavaScript 还支持一种复合数据类型，称为对象。JavaScript 中的对象被视为关联数组。

JavaScript 中的函数也被视为对象。因此，函数也可以作为变量存储。

## `typeof operator`

运算符用于查找 JavaScript 变量的类型。例如，你有两个数字变量，如下所示：

```js
5+1=6
```

在这里，`5` 和 `1` 是操作数，`+` 是运算符。

该运算符的语法如下：

```js
Typeof(operand)
```

### 描述

`Typeof` 运算符是一个一元运算符；它返回一个字符串，指示 typeof 表达式的类型。`typeof` 运算符以字符串的形式返回信息。它可以返回六个可能的字符串值。它们是 `string`、`number`、`object`、`function`、`Boolean` 和 `undefined`。

这里有一些 `typeof` 运算符的示例：

+   `typeof("5"+"1")`: 这将返回一个字符串

+   `typeof(5+1)`: 这将返回一个数字

+   `typeof(5+"1")`: 这将返回一个字符串

+   `typeof(5*"1")`: 这将返回一个数字

### 注意

`typeof` 运算符不是一个函数。操作数用圆括号括起来，所以看起来像是一个函数，但它们不是函数。当你使用 `typeofnull` 运算符时，它返回一个对象。

## `The undefined type`

在 JavaScript 中，一个空变量或没有值的变量有一个未定义的值。并且，`typeof` 也是未定义的。如果你想清空一个变量，你可以给它赋予一个未定义的值；例如：

```js
var employee = undefined;
```

### 描述

在 JavaScript 中，`undefined` 是 `global` 对象的一个属性。这意味着值是未定义的，我们在浏览器的全局范围内声明它。

`undefined` 的基本有三个 `property` 属性：

+   `Writeable`

+   `Enumerable`

+   `Configurable`

这三个 `property` 属性表明，对于现代浏览器，这些属性是不可写的、可枚举的和可配置的。因此，我们应该避免覆盖这些属性。`undefined` 类型是 JavaScript 中的一个原始类型。

### 注意

`Undefined` 类型表示一个变量尚未被赋予值。所有浏览器，如 Firefox、Google Chrome、Opera 和 Safari，都根据 ECMAScript 支持该 `undefined` 属性。

`Undefined` 是 JavaScript 库中的一个内置类型。对于任何值未定义的函数类型，都被视为 `undefined` 类型。例如，如果你有一个没有 `return` 语句的函数，该函数的结果将被视为未定义。

作为全局对象的属性，它的值最初是未定义的。我们可以将其作为全局变量访问，尽管它是一个全局属性。一个简单的 `undefined` 类型想法是，当你的脚本中没有定义变量，但它确实存在，并且你将其视为全局变量时。

每当函数在脚本中执行，并且执行完毕没有返回值时，它将返回 `undefined` 类型，例如：

```js
var abc,
f1=function(){}
f2=function() {
  var hello;
  return hello;
}
typeof(abc) ; // undefined
typeof(f1); // function
typeof(f2); // function
```

在这个例子中，变量 `abc` 被定义了，但没有给它赋值。请注意，`undefined` 类型是全局作用域中声明对象的原生属性。在这里，将其定义为全局对象并不意味着它不能被重新定义。它可以按照 ECMAScript 的规则被重新定义。因此，`undefined` 不是一个保留字；它可以在其全局作用域之外的其他作用域中用作变量名。

当你使用比较操作符时，你还可以在那里声明 `undefined` 类型，以了解变量是否有值。

## 空类型

在 JavaScript 中，`null` 表示一个空值。

以下是语法：

```js
varabc; // Here abc does not exist and it has been not initialized

varabc=null; // Here abc exist but it does not has a value or a type
```

### 描述

在 JavaScript 中，`null` 的数据类型是对象。`Null` 表示在脚本中该值不存在。当你想要清空一个值时，你可以将其声明为 `null`。在期望某些值的地方，但发现什么也没有时，可以预期会出现 `null`。

### 注意

`null` 和 `undefined` 类型之间有很大的区别。`null` 的数据类型是对象，而 `undefined` 的数据类型是 `undefined`。在 `identity` 操作符中，`null` 不能是 `undefined`，但在 `equality` 操作符中，`null` 可以是 `undefined`。这是因为 `equality` 操作符在比较之前会将左侧的值转换为相同的类型。

`undefined` 表示变量已声明但未赋值。例如，`var abc`：

```js
console.log(abc); //undefined
console.log(typeof abc); //undefinednull  is an empty value. It can be assigned to a variable which means variable is defined and an empty value has been assigned to it. For instance,var abc = null;
console.log(abc); //null
console.log(typeof abc); //returns object
```

考虑以下示例：

```js
Null===undefined;
//It's a false because identity operator always verify data type //equality

Null==undefined; //It's true because of equality operator
```

> *"你可能想知道为什么 typeof 操作符对于值为 null 的值返回 object。这实际上是原始 JavaScript 实现中的一个错误，后来被 ECMAScript 复制。今天，人们认为 null 被视为对象的占位符，尽管技术上它是一个原始值。"*
> 
> *- 专业 JavaScript 网页开发者指南，Wrox 出版社*

## 数值类型

在 JavaScript 中只有一种数字类型。数字可以带小数或不带小数书写。可以用科学记数法书写更大的或更小的数字。

### 描述

与其他编程语言不同，我们有许多数值数据类型，如整数、浮点数、双精度浮点数等。JavaScript 有一个名为 Number 的数据类型来表示所有数值。JavaScript 为此类型提供了三个符号值：无穷大、NaN（不是一个有效的数字）。在 JavaScript 的数值类型中定义的 NaN 显示它是一个保留字，实际上它的值在现实中不是一个数字。

### 注意

在 JavaScript 中，整数和浮点值之间没有区别。

整数可以是正数、0 或负数。它们可以是任何基数，如十进制、十六进制或八进制。但在 JavaScript 中，数字通常以十进制书写。要检查 JavaScript 中可表示或可用的最大或最小数字，可以使用最大和最小值常量：

```js
Number.maxValue
Number.minValue
```

### 注意

在 Firefox 或 Chrome 中，它是 `Number.MAX_VALUE`。

在数字类型中，你只需检查具有两种表示形式的数字，例如，如果你有数字 0，那么它有两种表示形式——一个是-0，另一个是+0。JavaScript 数字可以带小数点或不带小数点书写；例如：

```js
A non- decimal number
var a=10
A number with decimal
var b=10.0123
```

如果数字太大或太小，我们则使用科学记数法的指数来表示它。考虑以下示例：

```js
If we have a number 100000000
var a=1000e5
And 0.00111
var b=111e-5
```

限制为`1e+21`（控制台从 10 的幂次方显示科学记数法）。

### 注意

在其他编程语言中，我们可以将数字定义为`float`、`integer`、`long`和`short`。但在 JavaScript 中，数字只有一个数据类型，即`numbers`。这些数字遵循 IEEE 标准，并且这些数字是双精度。

## `Boolean`类型

在编程语言中，`Boolean`类型基于布尔逻辑工作，其中变量返回`true`或`false`的结果。这些`true`和`false`基本上是字面量。布尔逻辑通常用于比较运算符，例如小于或大于等。

任何变量都可以被赋予一个以小写`true`或`false`书写的`Boolean`值。

### 描述

在 JavaScript 中，当你将`Boolean`数据类型声明给一个变量时，该变量不应有任何引号。以下是如何使用布尔赋值：`var abc = "true"; //字符串赋值`和`var abc = true; //布尔赋值`。`Boolean`变量也用于`if`、`if...else`和`while`循环中。

下面是一个展示如何使用布尔变量的示例：

```js
var x=true;
if(x==true) {
  document.write("this true");
}
```

当存在或不存在值时，也会使用`Boolean`运算符。例如，如果我们脚本中有一个复选框，那么它的值取决于它是否被选中。

在其他数据类型中，变量的值可以有无限多种，但对于`Boolean`数据类型，只有两种——`true`和`false`。这种数据类型通常用于我们有一个受控结构，如比较、`if...else`循环和`while`循环。这些都是受控结构。`If`和`else`语句在值为`true`时执行一个动作；否则，将执行另一个动作，即`false`。

你还可以将`Boolean`数据类型用作比较表达式。比较表达式是一个评估`0`、`null`和`undefined`为`false`，其他值为`true`的表达式。考虑以下示例：

```js
C=A+B
```

在这个例子中，`A+B`的结果首先存储在`C`中，然后评估这个表达式。如果你想检查`C`的值是否等于`A+B`，你必须使用比较运算符来完成，如下所示：

```js
C==A+B
```

## 字符串类型

在 JavaScript 中，任何在引号之间书写的都被视为字符串。可以使用双引号或单引号。

字符串字面量可以通过两种方式定义。一种方式是使用双引号编写字符串，另一种方式是使用单引号编写字符串。

### 注意

在 JavaScript 中，没有特殊类型来表示单个字符。如果你想用 JavaScript 表示单个字符，你必须定义一个单字符字符串，例如，字符`a`。空字符串是一个零长度的字符串，例如""。

### 描述

在 JavaScript 中，文本数据以`string`数据类型的形式表示。在编程语言中，字符串是字符和数字的组合。JavaScript 字符串是零个或多个字符的有序序列。

编写这些字符串的限制是，你必须使用相同的引号在开头和结尾。这意味着以单引号开始的字符串必须以单引号结束，双引号也是如此。考虑以下示例：

```js
var x="xyz" //double quotes
var y='xyz' //single quotes
var z="" //empty string
```

如果字符串中存在问题，它无法显示某些字符。对于这些字符的表示，JavaScript 中有一个技术——我们定义转义序列来表示。这意味着我们无法直接写入的字符，可以使用转义序列来表示。例如，如果我们想在字符串中插入一个引号，我们可以通过在引号前加上反斜杠来实现。这被称为转义引号。同样，字符串字面量无法表示非打印字符，如制表符、换行符等。

转义序列是以反斜杠开始的字符组合。以下是一些转义序列：

+   \b: 退格删除前一个字符并将光标向后移动一个空格。

+   \n: 换行符结束当前行并将光标移动到新行。如果在字符串中间使用，它将把`\n`后面的文本移动到新行。

+   \f: 表格馈送它是一个页面换行 ASCII 控制字符。它告诉打印机在新页面上继续打印。

+   \v: 垂直制表符它曾用于控制页面上的垂直移动。现在很少使用。

+   \t: 水平制表符控制页面上的文本水平插入。通常使用键盘上的*Tab*键插入。

+   \" : 双引号双引号前面需要加上反斜杠才能在字符串中插入。

+   \' : 单引号单引号前面需要加上反斜杠才能在字符串中插入。

+   \\: 反斜杠在字符串中插入反斜杠时，我们用另一个反斜杠来转义它。

这里是这个示例：

```js
var abc = "hello \n Morning"
```

其输出将如下所示：

```js
Hello
Morning
```

## 对象类型

对象是原始数据类型和有时是引用数据类型的序列，存储为名称-值对。

语法如下：

```js
varsomeObj= {property:value};
```

### 描述

在 JavaScript 中，对象是用花括号书写的。对象中的每个项目被称为**属性**。属性值可以是任何有效的数据类型，也可以是函数或方法。考虑以下示例：

```js
varobj= {book_name:"xyz", Author:"abc"};
```

我们可以说对象是项的列表，列表中的每个项都存储了一些作为其名称-值对的函数和方法：

在 JavaScript 中，一切都是对象，例如：

+   正则表达式是对象

+   数字是对象

+   字符串是对象

+   数学符号是对象

+   数组和函数是对象

+   对象就是对象

任何项的属性名可以是字符串或数字。当一个属性是数字时，写这个属性的方式就不同；我们将这样写：

```js
var age = {"30": "abc"};
```

在 JavaScript 中，对象用于存储数据和编写自定义方法和函数。原始值例如 `null`、`undefined`、`true` 和 `false`。

对象属性基本上是用于方法和函数内部的变量。它们也可以是整个页面的全局变量。方法和函数之间有一个区别。函数是对象的一个独立单元，而方法则附加到对象上，并使用 `this` 关键字：

```js
Document.write ("hello world");
```

# 变量、作用域和内存

当我们在 JavaScript 中描述一个变量时，这个变量可以存储的数据量没有限制。最大值由浏览器的容量决定，这取决于它可以存储多少数据。让我们看看以下代码：

```js
var abc="xyz";
```

我们可以使用 `var` 关键字在 JavaScript 中定义一个变量。我们可以立即定义变量的值，或者稍后定义。我们定义在函数边界之外的变量是全局变量。

### 提示

在 JavaScript 中创建过多的全局变量是一个非常糟糕的方法。JavaScript 中不建议使用全局变量的主要原因是因为在 JavaScript 中，所有代码共享一个单一的全球命名空间，而且 JavaScript 有隐式全局变量，即未在局部作用域中显式声明的变量会自动包含在全局命名空间中。过度依赖全局变量可能导致同一页面上的各种脚本之间发生冲突。

脚本中的每个函数都可以访问全局变量。当你在函数中定义一个变量时，其作用域就在该函数内部。让我们看看以下代码片段：

```js
Function abc() {
  var xyz="abc";
}
```

### 注意

全局变量的作用域在整个页面的执行过程中永远不会停止，即使它不再需要。

如果你没有在函数内部写 `var`，那么这个变量被认为是全局变量。如果你用相同的全局变量名定义了一个变量，那么它将被认为是全局变量。

JavaScript 动态为变量分配内存。当这些变量的作用域被取消引用或完成时，垃圾回收器会自动将其从内存中移除。因此，其缺点是不需要的变量仍然留在内存中。

保存内存有两种方法：

+   使用函数作用域

+   手动取消引用这些变量

使用函数作用域更为合适，因为当函数结束时，变量会被取消引用，从而节省内存：

```js
(Function (window, document, undefined) {
  varabc="abc";
}
(window, document);
```

## 变量声明

JavaScript 变量可以通过唯一名称识别。这些唯一名称被称为 **标识符**。JavaScript 中的变量用于存储信息。标识符可以是任何长度和大小，例如，`a`、`b`、`myName`。

当你在 JavaScript 中声明一个变量时，机器会将其保存在内存中，以便你可以在需要时从内存中存储或检索数据。变量名在脚本中应该是唯一的。

在 JavaScript 中声明变量名的规则与之前讨论的标识符规则相同。

当代码开始执行时，它首先处理变量声明。这些变量可以全局声明，也可以在函数内部声明。我们可以声明多个以逗号分隔的变量，例如：

```js
var myName,myAddress,myContact;
```

当我们将数据存储到变量中时，这个过程称为**变量初始化**。我们可以在创建变量时或稍后当我们想要使用它时初始化变量，例如：

```js
var a=1;
var b=2;
var c;
c=a+b;
```

在这个例子中，变量`a`存储了值`1`，而`b`存储了`2`，然后我们创建了变量`c`并在稍后定义其值。

## 变量作用域

变量作用域是变量在程序中存在的框架。我们可以在程序中定义变量的作用域。基本上，作用域是我们能够访问的方法、函数和对象集合。

在 JavaScript 中，变量有两种作用域：

+   局部作用域变量

+   全局作用域变量

### 局部作用域变量

在函数内部定义的变量称为局部作用域变量，因为这个变量只能在这个函数内部访问。函数变量将只对这个变量是局部的。

#### 描述

对于在局部作用域中定义的变量，可以在不同的函数中使用具有相同名称的变量。当函数开始时，局部变量被创建，当函数结束时，局部变量被删除。在 JavaScript 中，始终在使用变量之前声明它们；未能这样做将抛出一些异常。在一个函数中，如果你有一个与全局变量具有相同名称的局部变量，那么局部变量将具有优先权。因此，避免使用相同的变量名。让我们看一下以下代码片段：

```js
function xyz() {
  varabc="xyz";
  document.write(abc);
}
```

### 全局作用域变量

全局变量可以在 JavaScript 程序的任何地方声明。它们可以从任何地方访问。全局作用域变量在整个页面上脚本执行期间都保留在内存中。这会导致内存耗尽。

#### 描述

基本上，全局变量是窗口对象。全局变量通常在函数外部定义，并且网页中的所有函数都可以访问这个全局变量。如果你给一个未声明的变量赋值，那么这个变量将被视为全局变量。当你关闭网页时，全局变量将被删除。对于 HTML 的前端开发者来说，窗口对象是一个全局变量。

```js
var abc="xyx";
functionmy Function() {
  //some code
}
```

## 原始类型值和引用类型值

变量可以有两种类型的值：

+   原始类型值

+   引用类型值

### 原始类型值

通常对象是从属性中聚合的，这些属性可以用来引用对象。原始类型（值，数据类型）不是对象，并且没有与之关联的方法。

#### 描述

在 JavaScript 中，有五种原始类型：`string`、`number`、`null`、`undefined` 和 `Boolean`。原始值直接存储在栈上。这些值存储在它们被访问的相同位置。由于其固定大小，它可以很容易地进行操作。

### 引用类型值

引用类型值是类似于数组和函数的特殊对象，存储在堆中。使用指针来定位它们在内存中的位置。

#### 描述

在 JavaScript 中，存储引用类型值的变量存储在堆内存中。这些对象不能轻易操作，因为它们包含任意元素和属性。当引用值传递给函数时，函数会修改其值或内容，这种变化可以通过调用函数的对象以及有对象引用的其他函数看到。

## 执行上下文和范围

范围和上下文不是同一件事。JavaScript 中的每个函数都有一个范围和上下文。当我们用 JavaScript 声明一个函数时，这个函数可以在不同的上下文和范围内被访问。JavaScript 遵循范围和上下文的设计模式。

范围和上下文之间的主要区别在于，范围是基于函数的，而上下文是基于对象的。范围用于在调用时访问函数。而上下文与 `this` 关键字一起使用，因此它基本上是一个引用类型。

范围与执行上下文在执行上下文开始时相关联。与之相关联的是一个范围链，例如：

```js
function first() {
  second();
  function second() {
    third();
    function third() {
      fourth();
      function fourth() {
        // do something
      }
    }
  }
}
first();
```

在 JavaScript 中，代码执行的 环境 被分类为以下几种：

+   全局代码

+   函数代码

+   `eval` 代码

### 全局代码

全局代码是一个默认环境，代码首次执行时，即外部 `.js` 文件和本地内联代码被加载。

### 函数代码

函数代码是代码执行期间流程控制进入函数体的环境。每次函数返回都会退出当前执行上下文。

### `eval` 代码

代码被提供给内置的 `eval` 函数。`eval` 代码调用上下文并创建一个执行上下文。

例如：

```js
var eval_context = { a: 10, b: 20, c: 30 };
function test() {
   console.log(this);
}
function evalInContext() {
   console.log(this);        // { a: 10, b: 20, c: 30 }
   eval("test()");      // { a: 10, b: 20, c: 30 } inside test()
}
evalInContext.call(eval_context);
```

在这个例子中，你使用你想要的上下文调用函数，并在该函数内部运行 `eval`。

## 垃圾回收

JavaScript 中没有单独的内存管理。浏览器决定何时需要清理。垃圾回收基本上是知道变量在整个程序执行过程中是否仍在使用或将来会被使用，如果不使用则收集并删除它们。简单来说，对对象的引用跟踪是在后台进行的。一旦它变得空闲或达到零，就可以由 GC 收集。

我们可以手动清理内存，但在某些情况下，没有必要手动清理内存。我们无法强制 JavaScript 清理内存，因为这项任务是由垃圾回收器在运行时完成的。一些重型应用程序，如计算机游戏，需要大量内存并减慢你的系统；因此，在这种情况下，这完全取决于你的代码以及你如何结构化你的应用程序代码以使用计算机内存。

你可以通过以下步骤来结构化你的代码：

### 对象

当你声明一个对象时，尝试通过删除其属性并将其恢复为空对象（如`{}`）来重用该对象。这个过程被称为对象的回收。当对象第一次创建时（`new foo();`），会为其分配内存。因此，如果已经声明了一个对象，我们可以在脚本中重用它。

### 数组

当你在脚本中使用数组时，使用完毕后请清除该数组。将`[]`分配给数组通常用作清除它的快捷方式，但实际上它会创建一个新的空数组并废弃旧的数组。你可以将数组长度设置为`0`（`arr.length = 0;`），这将清除数组，同时重用相同的数组对象。

### 函数

当你需要多次调用一个函数时，你可以通过将一个永久变量分配给该函数而不是反复调用它来优化我们的代码。
