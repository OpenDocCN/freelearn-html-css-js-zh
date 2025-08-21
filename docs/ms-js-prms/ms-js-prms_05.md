# 第五章：WinRT 中的承诺

在过去的四章中，我们花费时间加强我们的概念并确保我们的思维基础与承诺保持一致。从这一章开始，我们将探索在不同技术中的承诺。我们将了解这些技术是如何采用该概念的，它们为何采用它，以及承诺与这些技术有何关联。我们将查看一些相关技术的代码库，以便获得第一手知识，了解如何在实际环境中实现承诺。

# WinRT 简介

我们对技术的第一次关注是 WinRT。WinRT 是什么？它是 Windows Runtime 的简称。这是由微软提供的一个平台，用于构建适用于 Windows 8+操作系统的应用程序。它支持在 C++/ICX、C#（C#）、VB.NET、TypeScript 和 JavaScript 中进行应用程序开发。

微软将 JavaScript 作为其主要的、一等公民工具之一，用于开发跨浏览器应用程序以及开发相关设备。我们现在完全了解使用 JavaScript 的利弊，这使我们来到了实现承诺使用的地方。

您猜对了！在本章中，我们将重点介绍如何在 WinRT 上实现承诺，这是实现的需要，以及如何实现。我们还将根据需要查看一些代码，以了解承诺在这些平台上表现如何，以及如何实际使用它。

# WinRT 的演变

随着 Windows 8 的发布，微软发布了其著名且最常使用的操作系统 Windows 的全新架构。这个架构适用于所有设备和平台，包括手机、平板电脑、可穿戴设备等。由于这种单例方法，应用程序开发需要一个统一的方法，因此微软在其平台上添加了更多的工具和语言，从这个角度来看，JavaScript for Windows，或者 Win，就出现在了舞台上。

您可能会问，为什么它们采用了 JavaScript，而不是其他语言来扩展其网络编程的武器库？答案在于 JavaScript 的结构。在前几章中，我们了解到 JavaScript 被认为是基于网络编程的最佳工具，并且在许多场景中非常有用。微软已经利用了这种力量，并将其嵌入到 WinRT 平台中。通过添加这个平台，微软在其竞争对手中占据了优势，因为它现在可以接触到更多知道可以使用 JavaScript 为微软编程并且可以将作品展示给大量用户的程序员。

# 关于 WinJS 的一些细节

WinJS 作为一款开源的 JavaScript 库发布，由微软根据 Apache 许可证发布。最初，它旨在用于构建 Windows 应用商店的软件，但后来，它被广泛接受用于所有浏览器上。现在，它与 HTML5 结合使用，为基于 Brewers 的以及 Windows 应用商店的应用程序构建。

它最初是在 2014 年 4 月 4 日的 2014 年微软 Build 开发者大会上宣布的，自那时以来，它经历了从 1.0 到 3.0 版本的演变，其 SDK 内部功能和实现满满。

# WinJS – 它的目的和发行历史

WinJS 1.0 最初与 Windows 8.0 一起发布。以下是到目前为止值得注意的发行版本。发行历史如下：

| 发行名称 | 目的/重点领域 |
| --- | --- |
| WinJS 1.0 | 这是作为 Windows 8.0 的 JavaScript 库发布的。 |
| WinJS 2.0 for Windows 8.1 | 这是更新版本，在 GitHub 上发布为 Apache License。 |
| 为 Windows 的 WinJS Xbox 1.0 | 这是专门为 Xbox one for Windows 发布的。 |
| WinJS Phone 2.1 for Windows Phone 8.1 | 这是为 Windows phone 开发平台发布的。 |
| WinJS 3.0 | 这是于 2014 年 9 月发布的，用于改进跨平台功能、JavaScript 模块化和通用控件设计。 |

## WinJS on GitHub

由于 WinJS 是一个开源软件，它托管在 GitHub 上，作为 MSTF，微软开放技术。我假设你知道 GitHub 是什么以及它的用途；如果不知道，请查看[`github.com/`](https://github.com/)。

WinJS 的在线仓库分为三个基本部分：

+   WinJS 是用 TypeScript 编写的，可以在[`github.com/winjs/winjs`](https://github.com/winjs/winjs)找到。

+   用 JS 编写的 WinJS 模块，可以在[`github.com/winjs/winjs-modules`](https://github.com/winjs/winjs-modules)看到。

+   WinJS bower 是用 JS 编写的，可以在[`github.com/winjs/winjs-bower`](https://github.com/winjs/winjs-bower)找到。

这些仓库中的代码不断更新，全球程序员 24/7 提交错误修复，这是开源项目的唯一魅力。基础仓库位于[`github.com/winjs`](https://github.com/winjs)。

使用在线模拟器，你可以在[`try.buildwinjs.com`](http://try.buildwinjs.com)尝试 WinJS。

# HTML5，CSS3 和 JavaScript

HTML5、CSS3 和 JS 是 Web 应用程序开发的默认模型，原因之一非常强大。它们都不仅仅是技术；它们是标准。曾经有过这样的时代，公司习惯于推出自己的平台，并为程序员使用他们的平台提供许多赏金。在那个时代，对于开发者来说，在所有浏览器上保持应用程序的标准是一场噩梦，因此，很多时间都花在了项目的兼容性上，而不是实际特性的开发上。W3C 和其他标准维护机构解决了这一挫折，并开始研究行业主要玩家都能接受的标准。他们将使用这些作为基础，而不是为每个微小需求开发自己的标准。这导致了 HTML5 和 CSS3 的演变。由于 JavaScript 已经存在，并且被认为是浏览器的语言，因此它与剩下的两种技术结合，成为专有和开源项目的默认技术包。

现在，每个平台都可以使用这些，但语法上有一些微小的区别。这对程序员和工程师来说是一个福音，因为他们现在可以专注于解决业务问题，而不是兼容性。

# HTML5、CSS3 和 JavaScript 的 WT

WT 平台上的 JavaScript 允许程序员使用 HTML 和 CSS 构建应用程序。许多使用 JavaScript 的 WT 应用程序类似于为网站编写标记。除此之外，WT 上的 JavaScript 提供了一些附加功能，并引入了在这个平台上可以使用的一些不同方式。由于 WT 上 JavaScript 的实现平台之间有所不同，它在很大程度上采用了微软的风格，其中，利用默认的 JS 属性，WT 为 JavaScript 增加了一些额外功能。这提供了对触摸的增强支持，以及对 UI（用户界面）的外观和感觉更多的控制。这也提供了诸如`DatePicker`、`TimePicker`和`ListView`的控制器，以及对 WinJS 的专有访问。

# 需要将承诺与 WT 集成

JS 是 WT 平台上的主要语言之一。除了 JS 的好处，还有一些缺点。正如我们在上一章的讨论中所知，回调地狱是添加承诺的核心原因，这里也是如此。WT 也面临了同样的问题，但它通过将其实现为承诺来解决得足够快。对于 WT 的 JS，承诺是游戏规则的改变者，因为它使得编写健壮、可扩展、可维护的 Windows 平台应用程序变得容易。虽然 WT 不是第一个实现承诺的，但它是最快采用该概念并实现它的之一。

事实上，JavaScript 程序员开始使用 WT JS for Windows，由于它的高度可采用性，许多专业人士加入了这个社区。

# 使用异步编程时遇到的问题

只是为了刷新你的记忆，在第二章，*JavaScript 异步模型*中，我们学习了关于异步编程的大量知识，它是什么，以及 JS 是如何实现它的。我们都知道使用 JS 的问题在于它已经发展出了很高的复杂性，因为它在大多数操作中严重依赖于回调。在第二章，*JavaScript 异步模型*的*处理回调地狱*部分，我们看到如果回调变得无法控制，几乎不可能调试代码。那时提出了承诺范式来解决这个问题。当应用于 WT 时，JS 也有同样的情况。

# 启动承诺

Windows 库中为 JS 提供的异步 API 表示为承诺，如 common JS 承诺/提案所定义。一个人可以通过包括一个错误处理程序使他的代码更加健壮，这被认为是调试最重要的方面，正因为如此，许多 JavaScript 开发者更愿意使用承诺。

启动此快速入门的先决条件。

# 编写一个返回承诺的函数

以下是使用该示例代码，你可以有效地理解如何在 WT 中实现承诺。按照以下步骤操作：

1.  创建一个名为`IamPromise`的空白 Windows 运行时应用。

1.  添加一个`input`元素。

1.  添加一个显示 URL 结果的`div`元素。

1.  在`default.css`中添加样式说明，为应用程序添加一些呈现。

1.  为`input`元素添加一个更改处理程序。

1.  在更改处理程序中调用`xhr`。

1.  构建和调试应用程序，然后输入一个网址。

1.  在 VS2013 中用 JS 创建一个 WT 应用。

1.  添加一个`input`元素。

1.  在 HTML 中，使用以下代码创建一个`input`元素：

    ```js
    <div>
    <input id="inputUrl" />
    <!—the input id above is called input URL -- >
    </div>

    Add a DIV element that displays the result for the URL.

    <div id="ResultDiv">Result</div>
    <!—the div id named here as ResultDiv -- >

    Add the styling instructions to "default.css".

    input {
      // add your style statements here
    }
    ```

## 为输入元素添加一个更改处理程序。

使用以下代码了解`WinJS.Utilities.Ready`函数，该函数在 DOM 内容加载后的立即事件被调用。这是在页面解析后，但在所有资源都被加载之前的：

```js
WinJS.Utilities.ready(function () {

  // get the element by id
  var input = document.getElementById("inputUrl");
  // add our event listener here
  input.addEventListener("change", changeHandler);

}, false);
```

在更改处理程序中调用`xhr`。

在更改处理程序中通过传递用户输入的 URL 来调用`xhr`。之后，用结果更新`div`元素。`xhr`函数是返回承诺的函数。我们还可以使用承诺的`then`或`done`函数来更新 UI（用户界面），但在 WT 规范中`then()`和`done()`的使用之间有一个区别。`then()`函数在`xhr`函数成功返回或通过`XmlHttpRequest`产生错误时立即执行。相反，`done()`函数除了保证抛出未在函数内部处理的任何错误外，与`then()`函数相同：

```js
function changeHandler(e) {
  var input = e.target;
  var resDiv = document.getElementById("ResultDiv");

  WinJS.xhr({ url: e.target.value }).then(function completed(result) {
    if (result.status === 200) {
      resDiv.style.backgroundColor = "lightGreen";
      resDiv.innerText = "Success";
    }
  });
}
```

最后，是时候测试你的代码了。构建和调试应用程序，然后输入一个 URL。如果 URL 有效，那么在我们这个例子中名为`ResultDiv`的`div`元素应该变绿并显示**成功**消息。如果输入了错误的 URL，代码将不会做任何事情。

### 提示

在这里要记住的一点是，在输入 URL 后，可能需要在输入控件外部进行点击，以便发生更改事件。这通常不是情况，但作为一个提示，这是获取 promise 未来值的一个更简单的方法。

现在，第二好的部分来了——处理错误。

# 错误处理

使用 promise 的最佳部分是，错误处理和调试变得更加简单。仅仅通过添加几个函数，你不仅可以精确定位代码中的错误位置，还可以在控制台或浏览器上获取相关的错误日志。你不必总是添加`alert()`来调查错误的性质和位置。

同样的规则适用于我们之前的代码，我们可以在`then()`内部添加一个错误函数。记得在之前的代码中，当发生错误时，并没有显示错误吗？但是这次不同了。我们将添加一个错误处理程序，如果发现任何错误，它将把成功的背景颜色改为红色：

```js
function changeHandler(e) {
  var input = e.target;
  var resDiv = document.getElementById("ResultDiv");

  WinJS.xhr({url: e.target.value}).then(
    function fulfilled (result) {
      if (result.status === 200) {
        resDiv.style.backgroundColor = "lightGreen";
        resDiv.innerText = "Successfully returned the Promise ";
      }
    },
    // our error handler Function

    function error(e) {
      resDiv.style.backgroundColor = "red";

      if (e.message != undefined) {  // when the URL is incorrect or blank.
        resDiv.innerText = e.message;
      }

      else if (e.statusText != undefined) { // If  XmlHttpRequest was made.
        resDiv.innerText = e.statusText;
      }

      else {
        resDiv.innerText = "Error";
      }
    });
}
```

构建和调试应用程序，并输入 URL。如果 URL 正确，将显示成功，否则按钮将变红并显示错误消息。

### 提示

请注意，在前面的函数中，`error(e)`，我们将`e`参数与一个消息连接起来。采用这种做法将错误转换为字符串，因为它将显示更易理解的消息，这将帮助你调试和排除错误。

# 使用 then()和 done()函数链式调用 promise

就像规格一样，你不仅可以使用`then`和`done`函数来完成一个任务，还可以将它作为链式调用。这样，你也可以在代码内部创建自己的条件，使你的代码更加强大、优化和逻辑性。尽管如此，还是有一些限制，这也是逻辑上的限制。你可以添加多个`then()`，比如`then().then().then()`，但你不能这样做：`then().done().then()`。你可能会想知道背后的逻辑。每次`then()`都会返回一个 promise，你可以输入到下一个`then()`函数中，但当你添加`done()`时，它返回`undefined`，这打破了 promise 的逻辑，然而你从这样的链中得不到任何东西。

所以，简而言之，你可以这样做：`then().then().done()`。

然而，你不能这样做：`then().done().then()`。

这样的操作的通用示例可能如下所示：

```js
FirstAsync()
    .then(function () { return SecondAsync(); })
    .then(function () { return ThirdAsync(); })
    .done(function () { finish(); });
```

同时，要记住的是，如果你没有在`done()`中添加错误处理程序，而且操作出现错误，它会抛出异常，这将消耗整个事件循环。即使写在`try catch`块中，你也无法捕获这样的异常，而唯一能获取它的方式是通过`window.onerror()`。

然而，如果你不使用`then()`添加错误处理程序，情况就不会这样。它不会抛出异常，因为它不是这样设计的，相反，它只会返回一个处于错误状态的承诺，这可能会对后续的链式操作或处理输出造成更大的损害。所以，无论是使用`then()`还是`done()`，都要添加错误处理程序。

## 示例 1A —— 使用两个异步函数将网页下载到文件中

使用这个示例，我们将能够将网页下载到文件中。有几种方法可以做到这一点。最简单的方法是让浏览器为您保存文件，但这将是浏览器根据我们的指令行事的能力，而不是我们代码的能力。此外，您可以想象这个简单的操作如何很容易地解释如何使用两个异步方法来完成。

现在，来看看下面的代码：

```js
//WinJs code

WinJS.Utilities.startLog();

// Allocate the URI where to get the download the file.
var AllocatedUri = new Windows.Foundation.UriExample("http://www.packt.com");

// Get the folder for temporary files.
var temporaryFolder = Windows.Storage.ApplicationData.current.temporaryFolder;

// Create the temp file asynchronously.
temporaryFolder.createFileAsync("temporary.text", Windows.Storage.CreationCollisionOption.replaceExisting)
  .then(function (tempFile) {

    // lets start the download operation if the createFileAsync call succeeded

    var Iamdownloader = new Windows.Networking.BackgroundTransfer.BackgroundDownloader();
    var transfer = Iamdownloader.createDownload(uriExample, tempFile);
      return transfer.startAsync();
    })
    .then(

      //Define the function to use when the download completes successfully
      function (result) {
        WinJS.log && WinJS.log("File was download successfully ");
      });
```

再次，我们现在将解释每一行代码的作用。

有三个主要方法需要强调：`createFileAsync`、`startAsync`和`then`。

第一个`then`函数获取结果。然后将结果传递给处理函数。`BackgroundDownloader`方法创建下载操作，`startAsync`创建启动下载的例程。在这里，你可以看到`startAsync`是返回承诺的那个，我们将通过在第一个完成中返回`startAsync()`的值来将其与第二个`then()`链接起来。第二个`then()`负责完成处理程序，其参数包含下载操作。

## 示例 1B —— 使用 startAsync 将网页下载到文件中

另一种链式调用`then()`和`done()`函数的方法是，通过编写进度函数来跟踪异步操作的进度。由于这一点，我们不仅可以跟踪进度，还可以通过添加错误函数来获得很多关于错误条件的信息。

在下一个示例中，我们将看到如何使用`startAsync`函数和错误处理程序异步地将网页下载到文件中。这个示例的输出将与前一个相同，但机制稍有不同：

```js
// Allocate the URI where to get the download the file.
var AllocatedUri = new Windows.Foundation.Uri("http://www.packt.com");

// Get the folder for temporary files.
var temporaryFolder = Windows.Storage.ApplicationData.current.temporaryFolder;

// Create the temp file asynchronously.
temporaryFolder.createFileAsync("tempfile.txt", Windows.Storage.CreationCollisionOption.replaceExisting)
  .then(function (tempFile) {

    // lets start the download operation if the createFileAsync call succeeded

    var Iamdownloader = new Windows.Networking.BackgroundTransfer.BackgroundDownloader();
    var transfer = Iamdownloader.createDownload(uriExample, tempFile);
    return transfer.startAsync();
  })
  .then(
    //Define the function to use when the download completes successfully
    function (result) {
      WinJS.log && WinJS.log("File was download successfully ");
    },

    // this is where we add the error handlers which displays
    function (err) {
      WinJS.log && WinJS.log("File download failed.");
    },
    // Define the progress handling function.
    function (progress) {
      WinJS.log && WinJS.log("Bytes retrieved: " + progress.progress.bytesReceived);
    });
```

这段代码的唯一区别是正确添加了错误处理程序，这使得错误处理变得简单且易于阅读。

# 总结

在本章中，我们学习了承诺如何在 WinRT 中实现。我们看到了承诺在 Windows 平台上的演变以及它如何为不同的 Windows 设备做出贡献。我们还看到了它如何帮助 Windows 游戏机以及为 Windows 商店创建基于 Windows 的应用程序。

正是承诺的适应性使其在所有主要的前沿技术中找到了一席之地。即使是像微软这样的技术巨头也无法忽视它的存在，并且能够在当前和未来的技术中给予充分的关注和范围。

在下一章中，我们将学习承诺是如何在增长最快的服务器端 JavaScript 之一——Node.js 中实现的。
