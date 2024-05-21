# 第一章. 正则表达式入门

**正则表达式**是用来以语法方式表示模式的特殊工具。在处理任何类型的文本输入时，你并不总是知道值是什么，但你通常可以假设（甚至要求）你将接收到的格式。当你创建一个正则表达式来提取和操作这个输入时，就会出现这种情况。

因此，要匹配一个特定的模式需要一个非常机械的语法，因为即使是一个或两个字符的变化也会大大改变正则表达式的行为，结果也会相应地改变。

正则表达式本身（或**Regex**，简称）并不特定于任何单一的编程语言，你绝对可以在几乎所有现代语言中直接使用它们。然而，不同的语言使用不同的功能集和选项实现了正则表达式；在本书中，我们将通过**JavaScript**来看正则表达式及其特定的实现和功能。

# 一切都是关于模式

正则表达式是用专门的字符语法描述模式的字符串，本书中我们将学习这些不同字符和代码，这些字符和代码用于以模糊的方式匹配和操作不同的数据片段。现在，在我们尝试创建正则表达式之前，我们需要能够发现和描述这些模式（用英语）。让我们看一些不同和常见的例子，稍后在本书中，当我们对语法有更牢固的掌握时，我们将看到如何在代码中表示这些模式。

## 分析电话号码

让我们从简单的事情开始，看一下一个单独的电话号码：

```js
123-123-1234
```

我们可以描述这个模式为三个数字，一个破折号，然后另外三个数字，接着是第二个破折号，最后是四个数字。这很简单；我们看着一个字符串描述它是如何组成的，如果你的所有数字都遵循给定的模式，前面的描述就会完美地起作用。现在，假设我们将以下三个电话号码添加到这个集合中：

```js
123-123-1234
(123)-123-1234
1231231234
```

这些都是有效的电话号码，在你的应用程序中，你可能希望能够匹配它们所有，让用户可以以他们感觉最舒适的方式写入。所以，让我们再试一次我们的模式。现在，我会说我们有三个数字，可选地在括号内，然后是一个可选的破折号，另外三个数字，接着是另一个可选的破折号，最后是四个数字。在这个例子中，唯一强制的部分是这十个数字：破折号和括号的放置完全取决于用户。

还要注意的是，我们并没有对实际数字做任何限制，事实上，我们甚至不知道它们将是什么，但我们知道它们必须是数字（与字母相反），所以我们只放置了这个约束：

![分析电话号码](img/2258OS_01_09.jpg)

## 分析一个简单的日志文件

有时，我们可能有比数字或字母更具体的约束；在其他情况下，我们可能希望有一个特定的单词，或者至少是来自特定组的单词。在这些情况下（大多数情况下都是如此），你越具体，越好。让我们看下面的例子：

```js
[info] – App Started
[warning] – Job Queue Full
[info] – Client Connected
[error] – Error Parsing Input
[info] – Application Exited Successfully
```

这当然是某种日志的示例，我们可以简单地说每一行都是一个单独的日志消息。然而，如果我们想更具体地操作或提取数据，这并没有帮助。另一个选择是说我们在括号中有某种单词，它指的是日志级别，然后是破折号后面的消息，它将由任意数量的单词组成。同样，这并不太具体，我们的应用程序可能只知道如何处理前面三个日志级别，所以你可能想忽略其他一切或引发错误。

为了最好地描述前面的模式，我们可以说你有一个单词，它可以是 info、warning 或 error，位于一对方括号内，然后是一个破折号，然后是一些句子，构成了日志消息。这将允许我们更准确地捕获日志中的信息，并确保我们的系统准备好在发送之前处理数据：

![分析简单的日志文件](img/2258OS_01_10.jpg)

## 分析 XML 文件

我想讨论的最后一个例子是当你的模式依赖于自身时；XML 就是一个完美的例子。在 XML 中，你可能有以下标记：

```js
<title>Demo</title>
<size>45MB</size>
<date>24 Dec, 2013</date>
```

我们可以说模式由一个标签、一些文本和一个闭合标签组成。这对于它是一个有效的 XML 来说并不够具体，因为闭合标签必须与开放标签匹配。因此，如果我们重新定义模式，我们会说它包含左侧由开放标签包裹的一些文本，右侧是匹配的闭合标签：

![分析 XML 文件](img/2258OS_01_11.jpg)

最后三个例子只是用来让我们进入正则表达式的思维方式；这些只是一些常见类型的模式和约束，你可以在自己的应用程序中使用。

现在我们知道可以创建什么样的模式，让我们花一点时间讨论一下我们可以用这些模式做什么；这包括 JavaScript 提供的实际功能和函数，允许我们在创建模式后使用它们。

# JavaScript 中的正则表达式

在 JavaScript 中，正则表达式被实现为它们自己的类型的对象（比如`RegExp`对象）。这些对象存储模式和选项，然后可以用于测试和操作字符串。

要开始使用正则表达式，最简单的方法是启用 JavaScript 控制台并尝试不同的值。获取控制台的最简单方法是打开浏览器，比如**Chrome**，然后在任何页面上打开 JavaScript 控制台（在 Mac 上按*command* + *option* + *J*，在 Windows 上按*Ctrl* + *Shift* + *J*）。

让我们从创建一个简单的正则表达式开始；我们还没有深入讨论涉及的不同特殊字符的具体内容，所以现在，我们将创建一个只匹配一个单词的正则表达式。例如，我们将创建一个匹配`hello`的正则表达式。

## RegExp 构造函数

在 JavaScript 中，正则表达式可以以两种不同的方式创建，类似于字符串中使用的方式。有一个更明确的定义，你可以调用构造函数并传递你选择的模式（以及可选的任何设置），然后有一个文字定义，这是同样过程的简写。这是两种方式的例子（你可以直接在 JavaScript 控制台中输入）：

```js
var rgx1 = new RegExp("hello");
var rgx2 = /hello/;
```

这两个变量本质上是相同的，你可以根据个人喜好选择使用哪一个。唯一的区别是，使用构造函数方法时，你使用一个字符串来创建一个表达式：因此，你必须确保提前转义任何特殊字符，以便它传递到正则表达式。

除了模式，正则表达式的两种形式的构造函数都接受第二个参数，这是一个标志字符串。**标志**就像设置或属性，它们应用于整个表达式，因此可以改变模式及其方法的行为。

### 使用模式标志

我想要介绍的第一个标志是**忽略大小写**或**i**标志。标准模式是区分大小写的，但如果你有一个可以是任何大小写的模式，这是一个很好的选项，允许你只指定一个大小写，并且让修改器为你调整，使模式简短和灵活。

接下来的标志是**多行**或**m**标志，这使 JavaScript 将字符串中的每一行基本上视为新字符串的开始。例如，您可以说字符串必须以字母**a**开头。通常，JavaScript 会测试整个字符串是否以字母 a 开头，但使用 m 标志，它将针对每行单独测试此约束，因此任何行都可以通过以 a 开头的测试。

最后一个标志是**全局**或**g**标志。没有这个标志，`RegExp`对象只检查字符串中是否有匹配，只返回找到的第一个匹配；然而，在某些情况下，您不只是想知道字符串是否匹配，您可能想要了解所有特定的匹配。这就是全局标志的作用，当使用时，它将修改不同`RegExp`方法的行为，允许您获取所有匹配，而不仅仅是第一个。

因此，继续从前面的例子，如果我们想创建相同的模式，但这次，大小写不敏感并使用全局标志，我们会写类似于这样的内容：

```js
var rgx1 = new RegExp("hello", "gi");
var rgx2 = /hello/gi;
```

## 使用 rgx.test 方法

现在我们已经创建了我们的正则表达式对象，让我们使用它最简单的函数，即`test`函数。`test`方法只根据字符串是否与模式匹配返回`true`或`false`。这是它的一个示例：

```js
> var rgx = /hello/;
undefined
> rgx.test("hello");
true
> rgx.test("world");
false
> rgx.test("hello world");
true
```

如您所见，第一个字符串匹配并返回 true，第二个字符串不包含`hello`，因此返回`false`，最后一个字符串*匹配模式*。在模式中，我们没有指定字符串必须只包含`hello`，因此它匹配最后一个字符串并返回`true`。

## 使用 rgx.exec 方法

`RegExp`对象的下一个方法是`exec`函数，它不仅仅是检查模式是否与文本匹配，`exec`还返回有关匹配的一些信息。例如，让我们创建另一个正则表达式，并获取模式的起始`index`；

```js
> var rgx = /world/;
undefined
> rgx.exec("world !!");
[ 'world' ]
> rgx.exec("hello world");
[ 'world' ]
> rgx.exec("hello");
null
```

如您在这里所见，函数的结果包含实际匹配作为第一个元素（`rgx.exec("world !!")[0];`），如果您`console.dir`结果，您还会看到它还包含两个属性：`index`和`input`，分别存储起始`index`属性和完整的`input`文本。如果没有匹配，函数将返回`null`：

![使用 rgx.exec 方法](img/2258OS_01_01.jpg)

## 字符串对象和正则表达式

除了`RegExp`对象本身的这两种方法之外，字符串对象上还有一些接受`RegExp`对象作为参数的方法。

### 使用 String.replace 方法

最常用的方法是`replace`方法。例如，假设我们有`foo foo`字符串，我们想将其更改为`qux qux`。使用带有字符串的`replace`只会切换第一次出现，如下所示：

![使用 String.replace 方法](img/2258OS_01_02.jpg)

为了替换所有出现的情况，我们需要提供一个带有`g`标志的`RegExp`对象，如下所示：

![使用 String.replace 方法](img/2258OS_01_03.jpg)

### 使用 String.search 方法

接下来，如果您只想在字符串中找到第一个匹配的（从零开始的）索引，您可以使用`search`方法：

```js
> str = "hello world";
"hello world"
> str.search(/world/);
6
```

### 使用 String.match 方法

我现在要谈论的最后一个方法是`match`函数。当设置了`g`标志时，此函数返回与我们之前看到的`exec`函数相同的输出（包括`index`和`input`属性），但返回了所有匹配的常规`Array`。这是一个例子：

![使用 String.match 方法](img/2258OS_01_04.jpg)

我们已经快速浏览了 JavaScript 中正则表达式的最常见用法（代码方面），所以现在我们准备构建我们的`RegExp`测试页面，这将帮助我们探索实际的正则表达式语法，而不将其与 JavaScript 代码结合在一起。

# 构建我们的环境

为了测试我们的正则表达式模式，我们将构建一个**HTML**表单，该表单将处理提供的模式并将其与字符串匹配。

我将把所有的代码放在一个文件中，所以让我们从 HTML 文档的头部开始：

```js
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>Regex Tester</title>
    <link rel="stylesheet" href="http://netdna.bootstrapcdn.com/bootstrap/3.0.3/css/bootstrap.min.css">
    <script src="img/jquery.min.js"></script>
    <style>
      body{
        margin-top: 30px;
      }
      .label {
         margin: 0px 3px;
      }
    </style>
  </head>
```

### 提示

**下载示例代码**

您可以从[`www.packtpub.com`](http://www.packtpub.com)的帐户中下载您购买的所有 Packt Publishing 图书的示例代码文件。如果您在其他地方购买了本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便直接通过电子邮件接收文件。

这是一个相当标准的文档头，包含了标题和一些样式。除此之外，我还包括了用于设计的 bootstrap **CSS**框架和 jQuery 库来帮助进行**DOM**操作。

接下来，让我们在页面中创建表单和结果区域：

```js
<body>
  <div class="container">
    <div class="row">
      <div class="col-sm-12">
        <div class="alert alert-danger hide" id="alert-box"></div>
          <div class="form-group">
            <label for="input-text">Text</label>
            <input 
                    type="text" 
                    class="form-control" 
                    id="input-text" 
                    placeholder="Text"
            >
          </div>
          <label for="inputRegex">Regex</label>
          <div class="input-group">
            <input 
                   type="text" 
                   class="form-control" 
                   id="input-regex" 
                   placeholder="Regex"
            >
            <span class="input-group-btn">
              <button 
                      class="btn btn-default" 
                      id="test-button" 
                      type="button">
                             Test!
              </button>
            </span>
          </div>
        </div>
      </div>
      <div class="row">
        <h3>Results</h3>
        <div class="col-sm-12">
          <div class="well well-lg" id="results-box"></div>
        </div>
      </div>
    </div>
    <script>
      //JS code goes here
    </script>
  </body>
</html>
```

大部分代码是 Bootstrap 库所需的样式的样板 HTML；然而，要点是我们有两个输入：一个用于一些文本，另一个用于匹配的模式。我们有一个提交表单的按钮（`Test!`按钮）和一个额外的`div`来显示结果。

在浏览器中打开此页面应该会显示类似于这样的内容：

![构建我们的环境](img/2258OS_01_05.jpg)

## 处理提交的表单

我们需要做的最后一件事是处理表单的提交并运行正则表达式。我将代码分成了辅助函数，以帮助我们在浏览代码时进行代码流。首先，让我们为提交（`Test!`）按钮编写完整的点击处理程序（应该放在脚本标签中的注释处）：

```js
var textbox = $("#input-text");
var regexbox = $("#input-regex");
var alertbox = $("#alert-box");
var resultsbox = $("#results-box");

$("#test-button").click(function(){
  //clear page from previous run
  clearResultsAndErrors()

  //get current values
  var text = textbox.val();
  var regex = regexbox.val();

  //handle empty values
  if (text == "") {
    err("Please enter some text to test.");
  } else if (regex == "") {
    err("Please enter a regular expression.");
  } else {
    regex = createRegex(regex);

    if (!regex) {
      return;
    }

    //get matches
    var results = getMatches(regex, text);

    if (results.length > 0 && results[0] !== null) {
      var html = getMatchesCountString(results);
      html += getResultsString(results, text);
      resultsbox.html(html);
    } else {
      resultsbox.text("There were no matches.");
    }
  }
});
```

前四行使用 jQuery 从页面中选择相应的 DOM 元素，并将它们存储以供整个应用程序使用。这是一种最佳实践，当 DOM 是静态的时候，而不是每次使用时选择元素。

其余的代码是提交（`Test!`）按钮的点击处理程序。在处理`Test!`按钮的函数中，我们首先清除上一次运行的结果和错误。接下来，我们从两个文本框中获取值，并使用一个名为`err`的函数处理它们为空的情况，我们稍后会看一下这个函数。如果两个值都正常，我们尝试创建一个新的`RegExp`对象，并使用我编写的另外两个函数`createRegex`和`getMatches`来获取它们的结果。最后，最后一个条件块检查是否有结果，并显示**未找到匹配**消息或页面上的一个元素，该元素将使用`getMatchesCountString`显示找到了多少个匹配项，并使用`getResultsString`显示实际的匹配项。

## 重置匹配和错误

现在，让我们来看一下一些辅助函数，从`err`和`clearResultsAndErrors`开始：

```js
function clearResultsAndErrors() {
  resultsbox.text("");
  alertbox.addClass("hide").text("");
}

function err(str) {
  alertbox.removeClass("hide").text(str);
}
```

第一个函数清除结果元素中的文本，然后隐藏先前的错误，第二个函数取消隐藏警报元素，并将传递的错误添加为参数。

## 创建正则表达式

我想要看的下一个函数负责从文本框中给定的值创建实际的`RegExp`对象。

```js
function createRegex(regex) {
  try {
    if (regex.charAt(0) == "/") {
      regex = regex.split("/");
      regex.shift();

      var flags = regex.pop();
      regex = regex.join("/");

      regex = new RegExp(regex, flags);
    } else {
      regex = new RegExp(regex, "g");
    }
    return regex;
  } catch (e) {
    err("The Regular Expression is invalid.");
    return false;
  }
}
```

如果尝试使用不存在的标志或无效参数创建`RegExp`对象，它将抛出异常。因此，我们需要将`RegExp`的创建包装在`try`/`catch`块中，以便我们可以捕获错误并显示错误。

在`try`部分内，我们将处理两种不同的`RegExp`输入，第一种是当您在表达式中使用斜杠时。在这种情况下，我们通过斜杠分割表达式，移除第一个元素（即空字符串，它之前的文本是第一个斜杠），然后弹出最后一个元素，它应该是标志的形式。

然后我们将剩余的部分重新组合成一个字符串，并将其与标志一起传递给`RegExp`构造函数。我们正在处理的另一种情况是，您输入了一个字符串，然后我们将只使用`g`标志将此模式传递给构造函数，以便获得多个结果。

## 执行 RegExp 并提取其匹配项

接下来的函数是用于实际循环遍历`regex`对象并从不同的匹配中获取`results`的函数：

```js
function getMatches(regex, text) {
  var results = [];
  var result;

  if (regex.global) {
    while((result = regex.exec(text)) !== null) {
      results.push(result);
    }
  } else {
    results.push(regex.exec(text));
  }

  return results;
}
```

我们已经看到了之前的`exec`命令以及它如何为每个匹配返回一个`results`对象，但`exec`方法实际上根据是否设置了全局标志（`g`）而有所不同。如果没有设置，它将始终只返回第一个匹配，无论您调用多少次，但如果设置了，该函数将循环遍历结果，直到最后一个匹配返回`null`。在该函数中，如果设置了全局标志，我使用 while 循环来循环遍历`results`并将每个匹配推入`results`数组中，而如果没有设置，我只调用一次`function`并且只在第一个匹配时推入。

接下来，我们有一个函数，它将创建一个显示我们有多少匹配项（一个或多个）的字符串：

```js
function getMatchesCountString(results) {
  if (results.length === 1) {
    return "<p>There was one match.</p>";
  } else {
    return "<p>There are " + results.length + " matches.</p>";
  }
}
```

最后，我们有一个`function`，它将循环遍历`results`数组并创建一个 HTML 字符串以在页面上显示：

```js
function getResultsString(results, text) {
  for (var i = results.length - 1; i >= 0; i--) {
    var result = results[i];
    var match  = result.toString();
    var prefix = text.substr(0, result.index);
    var suffix = text.substr(result.index + match.length);
    text = prefix 
      + '<span class="label label-info">' 
      + match 
      + '</span>' 
      + suffix;
  }
  return "<h4>" + text + "</h4>";
}
```

在`function`内部，我们循环遍历匹配列表，对于每一个匹配，我们会切割字符串并将实际匹配内容包裹在标签中以进行样式设置。我们需要按照相反的顺序循环遍历列表，因为我们正在通过添加标签来改变实际文本，同时也需要改变索引。为了与`results`数组中的索引保持同步，我们从末尾修改`text`，保持其之前的部分不变。

## 测试我们的应用程序

如果一切顺利，我们现在应该能够测试应用程序。例如，假设我们输入`Hello World`字符串作为文本，并添加`l`模式（如果您记得的话，这将类似于在我们的应用程序中输入`/l/g`），您应该会得到类似于这样的结果：

![测试我们的应用程序](img/2258OS_01_06.jpg)

而如果我们指定相同的模式，但没有全局标志，我们将只得到第一个匹配：

![测试我们的应用程序](img/2258OS_01_07.jpg)

当然，如果您遗漏了某个字段或指定了无效的模式，我们的错误处理将会启动并提供适当的消息：

![测试我们的应用程序](img/2258OS_01_08.jpg)

现在一切都按预期工作，我们现在准备开始学习正则表达式本身，而不必担心旁边的 JavaScript 代码。

# 总结

在本章中，我们看了一下模式实际上是什么，以及我们能够表示的数据类型。正则表达式只是表达这些模式的字符串，结合 JavaScript 提供的函数，我们能够匹配和操作用户数据。

我们还介绍了构建一个快速的`RegExp`构建器，它使我们能够第一手了解如何在实际环境中使用正则表达式。在下一章中，我们将继续使用这个测试工具来开始探索`RegExp`语法。
