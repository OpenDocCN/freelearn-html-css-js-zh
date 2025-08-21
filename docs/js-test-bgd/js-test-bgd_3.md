# 第三章：语法验证

> 为了巩固我们之前学到的知识，我们现在将转向一个稍微困难的话题——验证 JavaScript。在本章中，你可以期待两个主要话题——围绕 JavaScript 代码验证和测试的问题，以及如何使用 JSLint 和 JavaScript Lint（这是一个免费的 JavaScript 验证器）来检查你的 JavaScript 代码，以及如何调试它们。我会明确地展示如何使用 JSLint 发现验证错误，然后，如何修复它们。
> 
> 我们将简要介绍验证和测试 JavaScript 之间的区别以及你在验证或测试代码时可能需要考虑的一些问题。你还将了解有效的 HTML 和 CSS 与 JavaScript 之间的关系，以及如何尝试编写高质量的代码以帮助你减少 JavaScript 代码中的错误。更重要的是，我们将学习到两个常用于验证 JavaScript 代码的免费工具，如何利用它们检查你的代码，以及最重要的是，如何修复检测到的验证错误。

在本章中，我们将学习以下主题：

+   验证和测试之间的区别

+   一个好的代码编辑器如何帮助你发现验证错误

+   什么使代码质量高

+   为什么在开始操作 JavaScript 之前需要确保 HTML 和 CSS 是有效的

+   为什么嵌入在 HTML 中的 JavaScript 可能会被报告为无效

+   通过验证检测到的常见 JavaScript 错误

+   JSLint 和 JavaScript Lint——如何使用它们检查你的代码

+   产生验证警告的有效代码结构

+   如何修复由 JSLint 发现的验证错误

那么，不再赘述，让我们开始讨论一个较轻松的话题——验证和测试之间的区别。

# 验证和测试之间的区别

有效验证和测试之间有一条细微的界限。如果你对集合（如数学中的集合）有一些了解，我会说验证可以导致更好的测试结果，而测试不一定导致有效代码。

让我们考虑这样一个场景——你编写了一个 JavaScript 程序，并在 Internet Explorer 和 Firefox 等主要浏览器上进行了测试，并且它运行正常。在这种情况下，你已经测试了代码，以确保它是功能性的。

然而，你所创建的同一代码可能有效也可能无效；有效代码类似于具有以下特点的代码：

+   格式良好

+   具有良好的编码风格（如适当的缩进、注释良好的代码、适当的间距）

+   符合语言规范（在我们这个案例中，是 JavaScript）

### 注意

在某个时间点，你可能会注意到良好的编码风格是非常主观的——有各种验证器可能对所谓的“良好编码风格”有不同的意见或标准。因此，如果你用不同的验证器来验证你的代码，当你看到不同的编码风格建议时，不要惊慌。

这并不意味着有效的代码会导致具有功能的代码（如你所见）以及具有功能的代码会导致有效的代码，因为两者有不同的比较标准。

然而，有效的代码通常会导致更少的错误，既有功能又是有效的代码通常是高质量代码。这是因为编写一段既有效又正确的 JavaScript 代码，比仅仅编写正确的代码要困难得多。

测试通常意味着我们试图让代码正确运行；而验证则是确保代码在语法上是正确的，有良好的风格，并且符合语言规范。虽然良好的编码风格可能是主观的，但通常有一种被大多数程序员接受的编码风格，例如，确保代码有适当的注释、缩进，并且没有全局命名空间的污染（尤其是在 JavaScript 的情况下）。

为了使情况更清晰，以下是三个你可以考虑的情况：

## 代码是有效的但却是错误的——验证并不能找到所有的错误。

这种错误形式很可能是由 JavaScript 中的逻辑错误引起的。考虑我们之前学到的内容；逻辑错误可能在语法上是正确的，但可能在逻辑上是错误的。

一个典型的例子可能是一个无限`for`循环或无限`while`循环。

## 代码是无效的但却是正确的

这很可能是大多数功能性代码的情况；一段 JavaScript 可能是功能上正确并且运行正常，但它可能是无效的。这可能是由于编码风格不良或有效代码中缺失的其他特征引起的。

在本章后面，你将看到一个完整的、有效的 JavaScript 代码示例。

## 无效且错误的代码——验证可以找到一些可能用其他方法难以发现的错误

在这种情况下，代码错误可能由第一章中提到的 JavaScript 错误的三个形式引起，*什么是 JavaScript 测试*，加载错误，运行时错误和逻辑错误。虽然由语法错误引起的错误可能更容易被好的验证器发现，但也有可能一些错误深深地隐藏在代码中，以至于使用手动方法很难发现它们。

既然我们已经对验证和测试有了共同的理解，那么让我们继续下一部分，讨论围绕高质量代码的问题。

# 代码质量

虽然关于高质量代码有很多观点，我个人认为有一些公认的标准。一些最常提到的标准可能包括代码的可读性，易于扩展，效率，良好的编码风格，以及符合语言规范等。

在这里，我们将关注使代码有效的一些因素——编码风格和符合规范。一般来说，良好的编码风格几乎可以保证代码高度可读（甚至对第三方也是如此），这将有助于我们手动发现错误。

最重要的是，良好的编码风格使我们能够快速理解代码，特别是如果我们需要团队合作或需要独立进行代码调试时。

你会注意到，我们将重点关注代码有效性对于测试目的的重要性。但现在，让我们从质量代码的第一个构建块开始——有效的 HTML 和 CSS。

## 在开始 JavaScript 之前，HTML 和 CSS 需要是有效的

在第一章中，我们有一个共同的理解，JavaScript 通过操作 HTML 文档的文档对象模型**（DOM）**为网页注入生命力。这意味着在 JavaScript 可以操作 DOM 之前，DOM 必须存在于代码中。

### 注意

这里有一个与 HTML、CSS 和浏览器直接相关的重要事实，相比 C 或 Python 等语言的编译器，浏览器对无效的 HTML 和 CSS 代码更加宽容。这是因为，所有浏览器需要做的就是解析 HTML 和 CSS，以便为用户渲染网页。另一方面，编译器通常对无效代码毫不留情。任何缺失的标签、声明等都会导致编译错误。因此，编写无效的或甚至是含有错误的 HTML 和 CSS 是可以的，但仍能得到一个“通常”外观的网页。

根据之前的解释，我们应该明白，为了创建高质量的 JavaScript 代码，我们需要有效的 HTML 和 CSS。

以下是我根据个人经验总结的，在开始学习 JavaScript 之前，为什么需要具备有效的 HTML 和 CSS 知识的原因：

+   有效的 HTML 和 CSS 有助于确保 JavaScript 按预期工作。例如，考虑这样一种情况，你可能有两个具有相同`id`的`div`元素（在之前的章节中，我们已经提到`div id`属性是给每个 HTML 元素提供唯一 ID 的），而你的 JavaScript 包含了一段预期对具有上面提到的 ID 的 HTML 元素工作的代码。这将导致意想不到的结果。

+   有效的 HTML 和 CSS 有助于提高你的网页行为的可预测性；试图用 JavaScript 修复错误的 HTML 或 CSS 是没有意义的。如果你从一开始就使用有效的 HTML 和 CSS，然后应用 JavaScript，你很可能会得到更好的结果。

+   无效的 HTML 和 CSS 可能会导致不同浏览器中出现不同的行为。例如，一个未闭合的 HTML 标签在不同浏览器中可能会有不同的显示效果。

总之，创建高质量 JavaScript 代码最重要的构建块之一是拥有有效的 HTML 和 CSS。

### 如果你不验证你的代码会发生什么

你可能会不同意我上文关于为什么 HTML 和 CSS 应该有效的观点。一般来说，验证有助于你防止与编码风格和规格相关的错误。然而，请注意，使用不同的验证器可能会给你不同的结果，因为验证器可能在代码风格方面有不同的标准。

如果你在想无效的代码是否会影响你的 JavaScript 代码，我会建议你尽可能让代码有效；无效的代码可能会导致棘手的问题，比如跨浏览器不兼容、代码难以阅读等。

无效代码意味着你的代码可能不是万无一失的；在互联网的早期，有些网站依赖于早期 Netscape 浏览器的怪癖。回想一下，当 Internet Explorer 6 被广泛使用时，也有许多网站以怪癖模式工作以支持 Internet Explorer 6。

现在，大多数浏览器都支持或正在支持网络标准（尽管有些细微的差异，但它们以微妙的方式支持），编写有效的代码是确保你的网站按预期工作和工作表现的最佳方式之一。

#### 如何通过验证简化测试

虽然无效的代码可能不会导致你的代码功能失效，但有效的代码通常可以简化测试。这是因为有效代码关注编码风格和规格，符合规格的有效代码通常更可能正确，且更容易调试。考虑以下风格上无效的代码：

```js
function checkForm(formObj){
alert(formObj.id)
//alert(formObj.text.value);
var totalFormNumber = document.forms.length;
// check if form elements are empty and are digits
var maxCounter = formObj.length; // this is for checking for empty values
alert(totalFormNumber);
// check if the form is properly filled in order to proceed
if(checkInput(formObj)== false){
alert("Fields cannot be empty and it must be digits!");
// stop executing the code since the input is invalid
return false;
}
else{
;
}
var i = 0;
var formID;
while(i < totalFormNumber){
if(formObj == document.forms[i]){
formID = i;alert(i);
}
i++;
}
if(formID<4){
formID++;
var formToBeChanged = document.forms[formID].id;
// alert(formToBeChanged);
showForm(formToBeChanged);
}
else{
// this else statement deals with the last form
// and we need to manipulate other HTML elements
document.getElementById("formResponse").style.visibility = "visible";
}
return false;
}

```

你熟悉之前的代码吗？还是没有意识到之前的代码片段来自第二章，《JavaScript 中的即兴测试与调试》？

之前的代码是代码风格不佳的一个极端例子，尤其是在缩进方面。想象一下，如果你必须手动调试你之前看到的第二个代码片段！我敢肯定，你会觉得检查代码很沮丧，因为你几乎无法从视觉上了解发生了什么。

更重要的是，如果你在团队中工作，你将被要求编写可读的代码；总之，编写有效的代码通常会导致代码更具可读性，更容易理解，因此错误更少。

#### 验证可以帮助你调试代码

如上文所述，浏览器通常对无效的 HTML 和 CSS 比较宽容。虽然这是真的，但可能会有一些错误没有被捕捉到，或者没有正确或优雅地渲染。这意味着虽然无效的 HTML 和 CSS 代码在某个平台或浏览器上可能看起来没问题，但在其他平台上可能不受支持。

这意味着使用有效的代码（有效的代码通常意味着由国际组织如 W3C 设定的标准代码集）将使你的网页在不同的浏览器和平台上正确渲染的概率大大增加。

有了有效的 HTML 和 CSS，您可以安全地编写您的 JavaScript 代码并期望它按预期工作，前提是您的 JavaScript 代码同样有效且无错误。

#### 验证帮助您使用好的编程实践

有效的代码通常需要使用好的编程实践。正如本章中多次提到的，好的实践包括适当闭合标签，合适的缩进以提高代码可读性等。

如果您需要更多关于使用 JavaScript 的良好实践的信息，请随时查看 JSLint 的创建者，Douglas Crockford，在 [`crockford.com`](http://crockford.com)。或者您可以阅读 John Resigs 的博客（JQuery 的创建者）在 [`ejohn.org/`](http://ejohn.org/)。他们都是很棒的人，知道什么是伟大的 JavaScript。

#### 验证

总结以上各节，DOM 由 HTML 提供，CSS 和 JavaScript 都应用于 DOM。这意味着如果存在无效的 DOM，那么操作 DOM 的 JavaScript（有时包括 CSS）可能会导致错误。

带着这个总结在心中，我们将重点关注如何使用颜色编码编辑器来发现验证错误。

## 颜色编码编辑器—您的编辑器如何帮助您发现验证错误

如果您是一个有经验的程序员，您可以跳过这一节；如果不是，您可能想了解一个好的编程编辑器的价值。

总的来说，一个好的编辑器可以帮助您防止验证错误。基于我们对验证的理解，您应该明白，您的编辑器应该执行以下活动：

+   突出显示匹配的括号

+   多种语法高亮

+   关键字、括号等之后的自动缩进

+   自动补全语法

+   自动补全您已经输入的单词

您可能注意到了，我遗漏了一些要点，或者增加了一些要点，关于好的编辑器应该做什么。但总的来说，前面列出的要点是为了帮助您防止验证错误。

### 注意

作为一个开始，您可以考虑使用微软的 SharePoint Designer 2007，这是一个免费、功能丰富的 HTML、CSS 和 JavaScript 编辑器，可在 [`www.microsoft.com/downloads/details.aspx?displaylang=en&FamilyID=baa3ad86-bfc1-4bd4-9812-d9e710d44f42`](http://www.microsoft.com/downloads/details.aspx?displaylang=en&FamilyID=baa3ad86-bfc1-4bd4-9812-d9e710d44f42)获得

例如，突出显示匹配的括号是为了确保您的代码用括号正确闭合，自动缩进是为了确保您为代码块使用了统一的空格。

尽管 JavaScript 代码块通常用花括号表示，但使用缩进来 visually 显示代码结构非常重要。考虑以下代码片段：

```js
function submitValues(elementObj){
var digits = /^\d+$/.test(elementObj.value);
var characters = /^[a-zA-Z\s]*$/.test(elementObj.value);
if(elementObj.value==""){
alert("input is empty");
return false;
}
else if(elementObj.name == "enterNumber" && digits == false){
alert("the input must be a digit!");
debuggingMessages(arguments.callee.name, elementObj, "INPUT must be digit");
return false;
}
else if(elementObj.name == "enterText" && characters == false){
alert("the input must be characters only!");
return false;
}
else{
elementObj.disabled = true;
return true;
}
}

```

下一个代码片段如下：

```js
function submitValues(elementObj)
{
var digits = /^\d+$/.test(elementObj.value);
var characters = /^[a-zA-Z\s]*$/.test(elementObj.value);
if(elementObj.value=="")
{alert("input is empty");
return false;
}
else if(elementObj.name == "enterNumber" && digits == false)
{alert("the input must be a digit!");
return false;
}else if(elementObj.name == "enterText" && characters == false)
{alert("the input must be characters only!");
return false;
}
else
{
elementObj.disabled = true;
return true;
}
}

```

我非常确信，您会认为第二个代码片段很乱，因为它的缩进不一致，您可能会遇到分辨哪个语句属于哪个条件块的问题。

从风格上讲，第二个代码示例就是我们所说的“糟糕的代码风格”。您可能会惊讶这可能会导致验证错误。

### 注意

如果您想知道`/^[a-zA-Z\s]*$/`和`/^\d+$/`是什么，它们实际上是正则表达式对象。正则表达式起源于 Perl（一种编程语言），由于它们的实用性，许多编程语言现在都有自己的正则表达式形式。大多数它们的工作方式相同。如果您想了解更多关于 JavaScript 正则表达式的信息，请随时访问[`www.w3schools.com/jsref/jsref_obj_regexp.asp`](http://www.w3schools.com/jsref/jsref_obj_regexp.asp)以了解正则表达式是如何工作的简要介绍。

# JavaScript 中常见的错误，将由验证工具检测到

我会简要提到一些由验证器检测到的最常见的验证错误。以下是它们的简短列表：

+   不一致的空格或缩进

+   缺少分号

+   缺少闭合括号

+   使用在调用或引用时未声明的函数或变量

### 注意

您可能已经注意到，一些验证错误并不是 exactly "错误"——就像语法错误——而是风格上的错误。如前所述，编码风格上的差异不一定导致功能错误，而是导致风格错误。但良好的编码风格的一个好处是，它通常会导致更少的错误。

至此，您可能很难想象这些常见错误实际上看起来是什么样子。但不要担心，当我们引入 JavaScript 验证工具时，您将能看到这些验证错误的实际操作。

# JSLint—在线验证器

JSLint 是我们将重点介绍的第一种 JavaScript 验证代码。通过访问这个 URL：[`www.jslint.com`](http://www.jslint.com)，您可以访问 JSLint。JSLint 在线验证器是由道格拉斯·克罗克福德创建的工具。

### 注意

道格拉斯·克罗克福德（Douglas Crockford）在雅虎！担任 JavaScript 架构师。他还是设计 JavaScript 未来版本的委员会成员。他在 JavaScript 风格和编程实践方面的观点普遍受到认可。您可以在他的网站上了解更多关于他和他的想法：[`www.crockford.com`](http://www.crockford.com)。

总的来说，JSLint 是一个在线 JavaScript 验证器。它帮助验证您的代码。同时，JSLint 足够智能，可以检测到一些代码错误，比如无限循环。JSLint 网站并不是一个特别大的网站，但无论如何，您必须阅读的两个重要链接如下：

+   基本操作说明，请访问[`www.jslint.com/lint.html`](http://www.jslint.com/lint.html)

+   要查看消息列表，请访问[`www.jslint.com/msgs.html`](http://www.jslint.com/msgs.html)

我不会试图向你描述 JSLint 是关于什么以及如何使用它；我个人认为应该亲自动手试试。因此，首先，我们将测试我们在第二章中编写的代码，*Ad Hoc Testing and Debugging in JavaScript*，看看会出现哪些验证错误（如果有的话）。

# 是行动的时候了——使用 JSLint 查找验证错误

正如前面提到的，我们将测试我们在第二章中编写的代码，*Ad Hoc Testing and Debugging in JavaScript*，看看我们会得到哪些验证错误。请注意，这个示例的完整验证代码可以在`source code`文件夹的`第三章`中找到，文件名为`perfect-code-for-JSLint.html`。

1.  打开你的网页浏览器，导航至[`www.jslint.com`](http://www.jslint.com)。你应该会看到一个带有巨大文本区域的首页。这就是你将要复制和粘贴你的代码的地方。

1.  请前往`第二章`的`source code`文件夹，打开名为：`getting-values-in-right-places-complete.html`的文件。然后，将源代码复制并粘贴到步骤 1 中提到的文本区域。

1.  现在点击名为**JSLint**的按钮。

    你的页面应该会立即刷新，并且你会收到一些形式的反馈。你可能会注意到你收到了很多（是的，很多）验证错误。而且，很可能会有一些对你来说是不懂的。然而，你应该能够识别出一些验证错误是在关于常见 JavaScript 验证错误的章节中引入的。

    现在，向下滚动，你应该在反馈区域看到以下短语：

    ```js
    xx % scanned
    too many errors

    ```

    这告诉你 JSLint 只是扫描了代码的一部分，并停止了扫描代码，因为错误太多了。

    我们能对此做些什么呢？如果验证错误太多，你一次无法找出所有的错误怎么办？

    不要担心，因为 JSLint 是健壮的，并且有选项设置，这些设置可以在[`www.jslint.com/#JSLINT_OPTIONS`](http://www.jslint.com/#JSLINT_OPTIONS)找到（这实际上位于 JSLint 主页的底部）。需要你输入的一个选项是**最大错误数**。在我们的例子中，你可能想输入一个巨大的数字，比如 1,000,000。

1.  输入一个巨大的数字作为**最大错误数**的输入框后，点击**The good parts**按钮。你会看到有几个复选框被选中了。

    在步骤 4 之后，你现在正式选择了被称为“The Good Parts”的选项，这是由本工具的作者设定的。这是一个设置，它会自动设置作者认为最重要的验证检查。

    这些选项包括：严格空格，每个函数允许一个 var 声明等等。

1.  现在点击**JSLint**按钮。你的浏览器将显示新的验证结果。现在你可以看看 JSLint 检测到的验证错误类型。

## 刚才发生了什么？

你刚才使用了 JSLint 来查找验证错误。这是 JSLint 的一个简单过程：将你的代码复制粘贴到文本区域，然后点击**JSLint**。不要对出现的这么多验证错误感到惊讶；我们才刚刚开始，我们将学习如何修复和避免这些验证错误。

### 注意

你可能注意到了，嵌入在 HTML 表单中的 JavaScript 导致了一个错误，提示**缺少 use strict 语句**。这个错误源于 JSLint 相信使用**use strict**语句，这使得代码能够在严格条件下运行。你将在本章的后部分学习如何修复和避免这类问题。

你将继续看到许多错误。在我看来，这是验证代码不容易实现的一个证据；但这是我们将在下一节实现的内容。

正如你所看到的，有各种各样的验证选项，在这个阶段，我们只要能让代码通过**好部分**的设定要求就足够了。因此，我们接下来将重点放在如何修复这些验证错误上。但在那之前，我会简要讨论产生验证警告的有效代码结构。

# 产生验证警告的有效代码结构

你可能注意到了，尽管我们的代码结构是有效的，但它产生了验证警告。你可能在想是否应该修复这些问题。以下是一些基本讨论，帮助你做出决定。

## 你应该修复产生验证警告的有效代码结构吗？

这取决于你的目标。如我在第一章中提到的，《什么是 JavaScript 测试？》，代码至少应该是正确的，按我们的意图工作。因此，如果你的目标是仅创建功能上正确的代码，那么你可能不想花时间和精力来修复这些验证警告。

然而，因为你正在阅读这本书，你很可能会想知道如何测试 JavaScript，正如你在本章后面将看到的，验证是测试 JavaScript 的重要部分。

## 如果你不修复它们会发生什么

无效代码的主要问题是，它将使代码的维护变得更为困难，在可读性和可扩展性方面。当团队合作时，这个问题会变得更严重，因为其他人必须阅读或维护你的代码。

有效的代码促进了良好的编程实践，这将帮助你避免将来出现的问题。

# 如何修复验证错误

本节将继续讨论上一节中提到的错误，并尝试一起修复它们。在可能的情况下，我会解释为什么某段代码会被认为是无效的。同时，编写有效且功能性的代码整个过程可能会很繁琐。因此，我会先从更容易修复的校验错误开始，然后再逐步转向更难的错误。

### 注意

在我们修复上一节中看到的校验错误的过程中，你可能会意识到修复校验错误可能需要在编写代码的方式上做出一些妥协。例如，你会了解到在代码中谨慎使用`alert()`并不是一种好的编程风格，至少根据 JSLint 的说法是这样的。在这种情况下，你必须将所有的`alert()`声明合并到一个函数中，同时仍保持代码的功能性。更重要的是，你也会意识到（或许）编写有效代码的最佳方式是从代码的第一行开始就编写有效的代码；你会看到修复无效代码是一个极其繁琐的过程，有时你只能尽量减少校验错误。

在修复代码的过程中，你将有机会练习重要的 JavaScript 函数，并学习如何编写更好的代码风格。因此，这可能是本章最重要的部分，我鼓励你和我一起动手实践。在开始修复代码之前，我首先总结一下 JSLint 发现的错误类型。

+   缺少“use `strict`”声明。

+   意外使用了`++`。

+   在`), value, ==, if, else, +`后面缺少空格。

+   函数名（如 debuggingMessages）未定义或在使用定义之前就使用了函数。

+   `var`声明过多。

+   使用了`===`而不是`==`。

+   `alert`未定义。

+   使用了`<\/`而不是`</`。

+   使用了 HTML 事件处理程序。

不再多说，我们直接开始讲解第一个校验错误：`use strict`。

## 缺少“use strict”声明的错误。

`use strict`语句是 JavaScript 中相对较新的特性，它允许我们的 JavaScript 在严格环境中运行。通常，它会捕获一些鲜为人知的错误，并“强制”你编写更严格、有效的代码。JavaScript 专家 John Resig 就此话题写了一篇很好的总结，你可以通过这个链接阅读它：[`ejohn.org/blog/ecmascript-5-strict-mode-json-and-more/`](http://ejohn.org/blog/ecmascript-5-strict-mode-json-and-more/)。

# 行动时间——修复"use strict"错误。

这个错误非常容易修复。但小心；如果代码无效，启用`use strict`可能会导致你的代码无法正常工作。以下是修复这个校验错误的方法：

1.  打开你的文本编辑器，复制并粘贴我们一直在使用的相同代码，并在你的 JavaScript 代码的第一行添加以下代码片段：

    ```js
    "use strict";

    ```

1.  保存你的代码并在 JSLint 上测试它。你会发现，现在错误已经消失了。

你可能会注意到还有一个关于你的 HTML 表单的另一个缺失的`use strict`错误；不要担心，我们会在本章的稍后部分解决这个问题。现在让我们继续下一个错误。

## 错误—意外使用++

这段代码在程序上没有问题。我们使用`++`的目的是在调用函数`addResponseElement()`时递增`globalCounter`。

然而，JSLint 认为使用`++`有问题。以下代码片段为例：

```js
var testing = globalCounter++ + ++someValues;
var testing2 = ++globalCounter + someValues++;

```

之前的陈述对大多数程序员来说可能看起来很困惑，因此被认为是坏的风格。更重要的是，这两个陈述在程序上是不同的，产生不同的结果。出于这些原因，我们需要避免使用`++`、`--`等这样的语句。

# 行动时间—修复"意外使用++"的错误

这个错误相对容易修复。我们只需要避免使用`++`。所以，导航到`addResponseElement()`函数，寻找`globalCounter++`。然后将`globalCounter++`更改为`globalCounter = globalCounter + 1`。所以，现在你的函数已经从这个样子：

```js
function addResponseElement(messageValue, idName){
globalCounter++; 
var totalInputElements = document.testForm.length;
debuggingMessages( addResponseElement","empty", "object is a value");
var container = document.getElementById('formSubmit');
container.innerHTML += "<input type=\"text\" value=\"" +messageValue+ "\"name=\""+idName+"\" /><br>";
if(globalCounter == totalInputElements){
container.innerHTML += "<input type=\"submit\" value=\"Submit\" />";
}
}

```

变成了这个样子：

```js
function addResponseElement(messageValue, idName) {
globalCounter = globalCounter + 1; 
debuggingMessages( "addResponseElement", "empty", "object is a value");
document.getElementById('formSubmit').innerHTML += "<input type=\"text\" value=\"" + messageValue + "\"name = \"" + idName + "\" /><br>";
if (globalCounter === 7) {
document.getElementById('formSubmit').innerHTML += "<input type=\"submit\" value=\"Submit\" />";
}
}

```

比较突出显示的行，你会看到代码的变化。现在让我们继续下一个错误。

## 错误—函数未定义

这个错误是由 JavaScript 引擎和网页在浏览器中渲染的方式引起的。我们在第一章*什么是 JavaScript 测试？*中简要提到，网页（和 JavaScript）在客户端从上到下解析。这意味着出现在顶部的东西将被首先阅读，然后是底部的东西。

# 行动时间—修复"函数未定义"的错误

1.  由于这个错误是由 JavaScript 函数的不正确流程引起的，我们需要改变函数的顺序。我们在第二章*即兴测试和调试 JavaScript*中做的是先写了我们将要使用的函数。这可能是错误的，因为这些函数可能需要的是只在 JavaScript 代码的后部分定义的数据或函数。这是一个非常简单的例子：

    ```js
    <script>
    function addTWoNumbers() {
    return numberOne() + numberTwo();
    }
    function numberOne(x, y) {
    return x + y;
    }
    function numberTwo(a, b){
    return a + b;
    }
    </script>

    ```

    基于之前的代码片段，你将意识到`addTwoNumbers()`需要从`numberOne()`和`numberTwo()`返回的数据。这里的问题在于，JavaScript 解释器在阅读`addTwoNumbers()`之前会先阅读`numberOne()`和`numberTwo()`。然而，`numberOne()`和`numberTwo()`都是由`addTwoNumbers()`调用的，导致代码流程不正确。

    这意味着，为了使我们的代码正确运行，我们需要重新排列函数的顺序。继续使用之前的例子，我们应该这样做：

    ```js
    <script>
    function numberOne(x, y) {
    return x + y;
    }
    function numberTwo(a, b){
    return a + b;
    }function addTWoNumbers() {
    return numberOne() + numberTwo();
    }
    </script>

    ```

    在之前的代码片段中，我们已经重新安排了函数的顺序。

1.  现在，我们将重新排列函数的顺序。对我们来说，我们只需要将我们的函数安排成这样：原来在代码中的第一个函数现在是最后一个，最后一个函数现在是第一个。同样，原来在 JavaScript 代码中出现的第二个函数现在是倒数第二个。换句话说，我们将反转代码的顺序。

1.  一旦您改变了函数的顺序，保存文件并在 JSLint 上测试代码。您应该注意到与函数未定义相关的验证错误现在消失了。

现在，让我们继续下一个验证错误。

## 太多的 var 声明

根据 JSLint，我们使用了太多的`var`声明。这意味着什么？这意味着我们在每个函数中使用了不止一个`var`声明；在我们的案例中，我们显然在每个函数中都使用了不止一个`var`声明。

这是怎么发生的呢？如果你滚动到底部并检查 JSLint 的设置，你会看到一个复选框被选中，上面写着**每个函数只允许一个 var 声明**。这意味着我们最多只能使用一个`var`。

为什么这被认为是好的风格呢？虽然许多程序员可能认为这是繁琐的，但 JSLint 的作者可能认为一个好的函数应该只做一件事。这通常意味着只操作一个变量。

当然，有很多讨论的空间，但既然我们都在这里学习，让我们动手修复这个验证错误。

# 行动时间——修复使用太多 var 声明的错误

为了修复这个错误，我们将需要进行某种代码重构。尽管代码重构通常意味着使您的代码更加简洁（即，更短的代码），您可能意识到将代码重构以符合验证标准是一项艰巨的工作。

1.  在本节中，我们将更改（几乎）所有将值保存到函数中的单个`var`声明。

    负责这个特定验证错误的代码是在`checkForm`函数中找到的。我们需要重构的语句如下：

    ```js
    var totalInputElements = document.testFormResponse.length;
    var nameOfPerson = document.testFormResponse.nameOfPerson.value;
    var birth = document.testFormResponse.birth.value;
    var age = document.testFormResponse.age.value;
    var spending = document.testFormResponse.spending.value;
    var salary = document.testFormResponse.salary.value;
    var retire = document.testFormResponse.retire.value;
    var retirementMoney = document.testFormResponse.retirementMoney.value;
    var confirmedSavingsByRetirement;
    var ageDifference = retire - age;
    var salaryPerYear = salary * 12;
    var spendingPerYear = spending * 12;
    var incomeDifference = salaryPerYear - spendingPerYear;

    ```

1.  现在我们将开始重构我们的代码。对于每个定义的变量，我们需要定义一个具有以下格式的函数：

    ```js
    function nameOfVariable(){
    return x + y; // x + y represents some form of calculation
    }

    ```

    我将从一个例子开始。例如，对于`totalInputElements`，我将这样做：

    ```js
    function totalInputElements() {
    return document.testFormResponse.length;
    }

    ```

1.  基于之前的代码，对您即将看到的内容做类似的事情：

    ```js
    /* here are the function for all the values */
    function totalInputElements() {
    return document.testFormResponse.length;
    }
    function nameOfPerson() {
    return document.testFormResponse.nameOfPerson.value;
    }
    function birth() {
    return document.testFormResponse.birth.value;
    }
    function age() {
    return document.testFormResponse.age.value;
    }
    function spending() {
    return document.testFormResponse.spending.value;
    }
    function salary() {
    return document.testFormResponse.salary.value;
    }
    function retire() {
    return document.testFormResponse.retire.value;
    }
    function retirementMoney() {
    return document.testFormResponse.retirementMoney.value;
    }
    function salaryPerYear() {
    return salary() * 12;
    }
    function spendingPerYear() {
    return spending() * 12;
    }
    function ageDifference() {
    return retire() - age();
    }
    function incomeDifference() {
    return salaryPerYear() - spendingPerYear();
    }
    function confirmedSavingsByRetirement() {
    return incomeDifference() * ageDifference();
    }
    function shortChange() {
    return retirementMoney() - confirmedSavingsByRetirement();
    }
    function yearsNeeded() {
    return shortChange() / 12;
    }
    function excessMoney() {
    return confirmedSavingsByRetirement() - retirementMoney();
    }

    ```

现在，让我们继续下一个错误。

## 期待<\/而不是<\

对我们大多数人来说，这个错误可能是最有趣的。我们之所以有这个验证错误，是因为 HTML 解析器与 JavaScript 解释器略有不同。通常，额外的反斜杠被 JavaScript 编译器忽略，但不被 HTML 解析器忽略。

这样的验证错误可能看起来不必要，但 Doug Crockford 知道这对我们的网页有一定的影响。因此，让我们继续解决这个错误的方法。

# 行动时间—解决期望的'<\/'而不是'</'错误

尽管这个错误是最引人入胜的，但它是最容易解决的。我们所需要做的就是找到所有包含`</`的 JavaScript 语句，并将它们更改为`<\/`。主要负责这个错误的功能是`buildFinalResponse()`。

1.  滚动到`buildFinalResponse()`函数，将所有含有`</`的语句更改为`<\/`。完成后，你应该有以下代码：

    ```js
    function buildFinalResponse(name, retiring, yearsNeeded, retire, shortChange) {
    debuggingMessages( buildFinalResponse", -1, "no messages");
    var element = document.getElementById("finalResponse");
    if (retiring === false) {
    element.innerHTML += "<p>Hi <b>" + name + "<\/b>,<\/p>";
    element.innerHTML += "<p>We've processed your information and we have noticed a problem.<\/p>";
    element.innerHTML += "<p>Base on your current spending habits, you will not be able to retire by <b>" + retire + " <\/b> years old.<\/p>";
    element.innerHTML += "<p>You need to make another <b>" + shortChange + "<\/b> dollars before you retire inorder to acheive our goal<\/p>";
    element.innerHTML += "<p>You either have to increase your income or decrease your spending.<\/p>";
    }
    else {
    // able to retire but....
    //alertMessage("retiring === true");
    element.innerHTML += "<p>Hi <b>" + name + "<\/b>,<\/p>";
    element.innerHTML += "<p>We've processed your information and are pleased to announce that you will be able to retire on time.<\/p>";
    element.innerHTML += "<p>Base on your current spending habits, you will be able to retire by <b>" + retire + "<\/b>years old.<\/p>";
    element.innerHTML += "<p>Also, you'll have' <b>" + shortChange + "<\/b> amount of excess cash when you retire.<\/p>";
    element.innerHTML += "<p>Congrats!<\/p>";
    }
    }

    ```

注意所有`</`都已更改为`<\/`。你可能还想检查代码，看看是否有这样的错误残留。

现在，解决了这个错误后，我们可以继续解决下一个验证错误。

## 期望'==='但找到'=='

在 JavaScript 和大多数编程语言中，`==`和`===`有显著的区别。一般来说，`===`比`==`更严格。

### 注意

JavaScript 中`===`和`==`的主要区别在于，`===`是一个严格等于运算符，只有当两个操作数相等且类型相同时，它才会返回布尔值`true`。另一方面，`==`运算符如果两个操作数相等，即使它们类型不同，也会返回布尔值`true`。

根据 JSList，当比较变量与真理值时应使用`===`，因为它比`==`更严格。在代码严格性方面，JSLint 确保代码质量是正确的。因此，现在让我们纠正这个错误。

# 行动时间—将`==`更改为`===`

由于前面提到的原因，我们现在将所有`==`更改为`===`，用于需要比较的语句。尽管这个错误相对容易解决，但我们需要了解这个错误的重要性。`===`比`==`严格得多，因此使用`===`而不是`==`更为安全和有效。

回到你的源代码，搜索所有包含`==`的比较语句并将它们更改为`===`. `==`主要出现在`if`和`else-if`语句中，因为它们用于比较。

完成后，你可能想测试一下你的更新后的代码是否通过了 JSLint，并看看你是否清除了所有这类错误。

现在，让我们继续处理另一个繁琐的错误："Alert is not defined"。

## Alert is not defined

通常，单独使用`alert`会导致全局命名空间的"污染"。尽管它很方便，但根据 JSLint，这是一种不好的代码实践。因此，我们将要使用的解决这个验证错误的方法是再次进行某种代码重构。

在我们的代码中，你应该注意到我们主要使用`alert()`来提供关于函数名、错误消息等方面的反馈。我们需要使我们的`alert()`能够接受各种形式的消息。

# 行动时间—解决"Alert is not defined"错误

我们将做的是将所有的`alert()`语句合并到一个函数中。我们可以向该函数传递一个参数，以便根据情况改变警告框中的消息。

1.  回到你的代码，在`<script>`标签的顶部定义以下函数：

    ```js
    function alertMessage(messageObject) {
    alert(messageObject);
    return true;
    }

    ```

    +   `messageObject`是我们将用来保存我们消息的参数。

1.  现在，将所有的`alert()`语句更改为`alertMessage()`，以确保`alert()`的消息与`alertMessage()`的消息相同。完成后，保存文件并再次运行 JSLint 代码。

如果你尝试运行你的代码在 JSLint 中，你应该看到`alert()`造成的“损害”已经减少到只有一次，而不是十到二十次。

在这种情况下，我们可以做的是最小化`alert()`的影响，因为对于我们来说，没有一个现成的替代方案可以在警告框中显示消息。

现在是避免 HTML 事件处理程序的下一个错误的时候了。

## 避免使用 HTML 事件处理程序

良好的编码实践通常指出需要将编程逻辑与设计分离。在我们的案例中，我们在 HTML 代码中嵌入了事件处理程序（JavaScript 事件）。根据 JSLint，这种编码可以通过完全避免 HTML 事件处理程序来改进。

### 注意

尽管理想情况下是将编程逻辑与设计分离，但在使用 HTML 内置事件处理程序方面并没有什么功能上的问题。你可能需要考虑这是否值得（在时间、可维护性和可扩展性方面），坚持（几乎）完美的编码实践。在本节的后部分，你会发现尝试验证（并功能正确）代码可能会很繁琐（甚至令人烦恼）。

为了解决这个验证错误，我们将需要使用事件监听器。然而，由于事件监听器的兼容性问题，我们将使用 JavaScript 库来帮助我们处理事件监听器支持的不一致性。在这个例子中，我们将使用 JQuery。

JQuery 是一个由 John Resig 创建的 JavaScript 库。你可以通过访问这个链接下载 JQuery：[`jquery.com`](http://jquery.com)。正如这个网站所描述的，“JQuery 是一个快速和简洁的 JavaScript 库，它简化了 HTML 文档遍历、事件处理和动画，以及 Ajax 交互，用于快速网页开发。”在我的个人经验中，JQuery 确实通过修复许多棘手问题（如 DOM 不兼容性、提供内置方法创建动画等）使生活变得更容易。我当然建议你通过访问以下链接跟随一个入门教程：[`docs.jquery.com/Tutorials:Getting_Started_with_jQuery`](http://docs.jquery.com/Tutorials:Getting_Started_with_jQuery)

# 行动时间—避免 HTML 事件处理程序

在本节中，你将学习如何通过不同的编码方式避免 HTML 事件处理程序。在这种情况下，我们不仅删除了每个 HTML 输入元素中嵌入的 JavaScript 事件，还需要为我们的 JavaScript 应用程序编写新函数，使其以相同的方式工作。此外，我们将使用一个 JavaScript 库，帮助我们删除与事件处理和事件监听器相关的所有复杂内容。

1.  打开同一文档，滚动到`<body>`标签。删除在表单中找到的所有 HTML 事件处理程序。在你删除了所有的 HTML 事件处理程序后，你的表单源代码应该看起来像这样：

    ```js
    <form name="testForm" >
    <input type="text" name="enterText" id="nameOfPerson" size="50" value="Enter your name"/><br>
    <input type="text" name="enterText" id="birth" size="50" value="Enter your place of birth"/><br>
    <input type="text" name="enterNumber" id="age" size="50" maxlength="2" value="Enter your age"/><br>
    <input type="text" name="enterNumber" id="spending" size="50" value="Enter your spending per month"/><br>
    <input type="text" name="enterNumber" id="salary" size="50" value="Enter your salary per month"/><br>
    <input type="text" name="enterNumber" id="retire" size="50" maxlength="3" value="Enter your the age you wish to retire at" /><br>
    <input type="text" name="enterNumber" id="retirementMoney" size="50" value="Enter the amount of money you wish to have for retirement"/><br>
    </form>

    ```

1.  现在滚动到`</style>`标签。在`</style>`标签之后，输入以下代码片段：

    ```js
    <script src="img/jquery.js">
    </script>

    ```

    在前一行中，你正在使 JQuery 在你的代码中生效。这将允许你在修复代码时使用 JQuery 库。现在该写一些 JQuery 代码了。

1.  为了维持我们代码的功能，我们将需要使用 JQuery 提供的`.blur()`方法。滚动到你的 JavaScript 代码的末尾，添加以下代码片段：

    ```js
    jQuery(document).ready(function () {
    jQuery('#nameOfPerson').blur(function () {
    submitValues(this);
    });
    jQuery('#birth').blur(function () {
    submitValues(this);
    });
    jQuery('#age').blur(function () {
    submitValues(this);
    });
    jQuery('#spending').blur(function () {
    submitValues(this);
    });
    jQuery('#salary').blur(function () {
    submitValues(this);
    });
    jQuery('#retire').blur(function () {
    submitValues(this);
    });
    jQuery('#retirementMoney').blur(function () {
    submitValues(this);
    });
    jQuery('#formSubmit').submit(function () {
    checkForm(this);
    return false;
    });
    });

    ```

    以下是对 JQuery 工作方式的简短解释：`jQuery(document).ready(function ()`用于启动我们的代码；它允许我们使用 JQuery 提供的方法。为了选择一个元素，我们使用`jQuery('#nameOfPerson')`。如前所述，我们需要保持代码的功能，所以我们将使用 JQuery 提供的`.blur()`方法。为此，我们将`.blur()`添加到`jQuery('#nameOfPerson')`中。我们需要调用`submitValues()`，因此我们需要将`submitValues()`包含在`.blur()`中。因为`submitValues()`是一个函数，所以我们将它这样包含：

    ```js
    jQuery('#nameOfPerson').blur(function () {
    submitValues(this);
    });

    ```

+   在此时，我们应该已经完成了必要的修正，以实现有效和功能的代码。我在下一节中简要总结一下这些修正。

## 我们所做修正的总结

现在，我们将通过快速回顾我们所做以修复验证错误的步骤来刷新我们的记忆。

首先，我们将原始代码粘贴到 JSLint 中，并注意到我们有大量的验证错误。幸运的是，这些验证错误可以分组，这样相同的错误可以通过修正一个代码错误来解决。

接下来，我们开始了修正过程。一般来说，我们试图从那些看起来最容易的验证错误开始修复。我们修复的第一个验证错误是缺少`use strict`声明的错误。我们所做的是在我们的 JavaScript 代码的第一行输入`use strict`，这样就修复了错误。

我们修复的第二个验证错误是“函数未定义错误”。这是由于 JavaScript 函数的不正确流程造成的。因此，我们将函数的流程从这：

```js
function submitValues(elementObj){
/* some code omitted */
}
function addResponseElement(messageValue, idName){
/* some code omitted */
function checkForm(formObj){
/* some code omitted */
}
function buildFinalResponse(name,retiring,yearsNeeded,retire, shortChange){
/* some code omitted */
}
function debuggingMessages(functionName, objectCalled, message){
/* some code omitted */
}

```

到这个程度：

```js
function debuggingMessages(functionName, objectCalled, message) {
/* some code omitted */
}
function checkForm(formObj) {
/* some code omitted */
function addResponseElement(messageValue, idName) {
/* some code omitted */
}
function submitValues(elementObj) {
/* some code omitted */
}

```

请注意，我们只是简单地反转了函数的顺序来修复错误。

然后我们转向了一个非常耗时的错误——在函数内使用太多的`var`声明。总的来说，我们的策略是将几乎所有的`var`声明重构为独立的函数。这些独立函数的主要目的是返回一个值，仅此而已。

接下来，我们又转向了另一个耗时的验证错误，那就是"expected`<\/` instead of`</`。一般来说，这个错误是指闭合的 HTML 标签。所以我们所做的就是将所有的闭合 HTML 标签中的`/>`更改为`\/>`。例如，我们将以下代码更改为：

```js
function buildFinalResponse(name,retiring,yearsNeeded,retire, shortChange){
debuggingMessages( buildFinalResponse", -1,"no messages");
var element = document.getElementById("finalResponse");
if(retiring == false){
//alert("if retiring == false");
element.innerHTML += "<p>Hi <b>" + name + "</b>,<p>";
element.innerHTML += "<p>We've processed your information and we have noticed a problem.</p>";
element.innerHTML += "<p>Base on your current spending habits, you will not be able to retire by <b>" + retire + " </b> years old.</p>";
element.innerHTML += "<p>You need to make another <b>" + shortChange + "</b> dollars before you retire inorder to acheive our goal</p>";
element.innerHTML += "<p>You either have to increase your income or decrease your spending.</p>";
}
else{
// able to retire but....
alert("retiring == true");
element.innerHTML += "<p>Hi <b>" + name + "</b>,</p>";
element.innerHTML += "<p>We've processed your information and are pleased to announce that you will be able to retire on time.</p>";
element.innerHTML += "<p>Base on your current spending habits, you will be able to retire by <b>" + retire + "</b>years old.
</p>";
element.innerHTML += "<p>Also, you'll have' <b>" + shortChange + "</b> amount of excess cash when you retire.</p>";
element.innerHTML += "<p>Congrats!<p>";
}
}

```

到这个：

```js
function buildFinalResponse(name, retiring, yearsNeeded, retire, shortChange) {
debuggingMessages( buildFinalResponse", -1, "no messages");
var element = document.getElementById("finalResponse");
if (retiring === false) {
element.innerHTML += "<p>Hi <b>" + name + "<\/b>,<\/p>";
element.innerHTML += "<p>We've processed your information and we have noticed a problem.<\/p>";
element.innerHTML += "<p>Base on your current spending habits, you will not be able to retire by <b>" + retire + " <\/b> years old.<\/p>";
element.innerHTML += "<p>You need to make another <b>" + shortChange + "<\/b> dollars before you retire inorder to achieve our goal<\/p>";
element.innerHTML += "<p>You either have to increase your income or decrease your spending.<\/p>"; 
}
else {
// able to retire but....
//alertMessage("retiring === true");
element.innerHTML += "<p>Hi <b>" + name + "<\/b>,<\/p>";
element.innerHTML += "<p>We've processed your information and are pleased to announce that you will be able to retire on time.<\/p>";
element.innerHTML += "<p>Base on your current spending habits, you will be able to retire by <b>" + retire + "<\/b>years old.<\/p>";
element.innerHTML += "<p>Also, you'll have' <b>" + shortChange + "<\/b> amount of excess cash when you retire.<\/p>";
element.innerHTML += "<p>Congrats!<\/p>"; 
}
}

```

请注意，高亮的行是我们将`/>`更改为`\/>`的地方。

在修复上一个错误后，我们转向了一个概念上更难以理解，但容易解决的错误。那就是，"expected `===` instead of saw `==`"。根据 JSLint，使用`===`比使用`==`更严格，更安全。因此，我们需要将所有的`==`更改为`===`。

下一个错误，"Alert is not defined"，在概念上与"Too many `var` statement"错误相似。我们需要做的是将所有的`alert()`声明重构为接受参数`messageObject`的`alertMessage()`函数。这使我们能够在几乎整个 JavaScript 程序中只使用一个`alert()`。每当我们需要使用一个警告框时，我们只需要向`alertMessage()`函数传递一个参数。

最后，我们转向修复一个最棘手的验证错误："避免使用 HTML 事件处理程序"。由于事件监听器的复杂性，我们得到了流行的 JavaScript 库 JQuery 的帮助，并编写了一些 JQuery 代码。首先，我们从我们的 HTML 表单中移除了所有的 HTML 事件处理程序。我们的 HTML 表单从这样变成了：

```js
<form name="testForm" >
<input type="text" name="enterText" id="nameOfPerson" onblur="submitValues(this)" size="50" value="Enter your name"/><br>
<input type="text" name="enterNumber" id="age" onblur="submitValues(this)" size="50" maxlength="2" value="Enter your age"/><br>
<input type="text" name="enterText" id="birth" onblur="submitValues(this)" size="50" value="Enter your place of birth"/><br>
<input type="text" name="enterNumber" id="spending" onblur="submitValues(this)" size="50" value="Enter your spending per month"/><br>
<input type="text" name="enterNumber" id="salary" onblur="submitValues(this)" size="50" value="Enter your salary per month"/><br>
<input type="text" name="enterNumber" id="retire" onblur="submitValues(this)" size="50" maxlength="3" value="Enter your the age you wish to retire at" /><br>
<input type="text" name="enterNumber" id="retirementMoney" onblur="submitValues(this)" size="50" value="Enter the amount of money you wish to have for retirement"/><br>
</form>

```

到这个：

```js
<form name="testForm" >
<input type="text" name="enterText" id="nameOfPerson" size="50" value="Enter your name"/><br>
<input type="text" name="enterText" id="birth" size="50" value="Enter your place of birth"/><br>
<input type="text" name="enterNumber" id="age" size="50" maxlength="2" value="Enter your age"/><br>
<input type="text" name="enterNumber" id="spending" size="50" value="Enter your spending per month"/><br>
<input type="text" name="enterNumber" id="salary" size="50" value="Enter your salary per month"/><br>
<input type="text" name="enterNumber" id="retire" size="50" maxlength="3" value="Enter your the age you wish to retire at" /><br>
<input type="text" name="enterNumber" id="retirementMoney" size="50" value="Enter the amount of money you wish to have for retirement"/><br>
</form>

```

为了支持新的 HTML 表单，我们链接了 JQuery 库，并添加了一些代码来监听 HTML 表单事件，像这样：

```js
<script type="text/javascript"> src="img/jquery.js"></script>
<script type="text/javascript">
/* some code omitted */
jQuery(document).ready(function () {
jQuery('#nameOfPerson').blur(function () {
submitValues(this);
});
jQuery('#birth').blur(function () {
submitValues(this);
});
jQuery('#age').blur(function () {
submitValues(this);
});
jQuery('#spending').blur(function () {
submitValues(this);
});
jQuery('#salary').blur(function () {
submitValues(this);
});
jQuery('#retire').blur(function () {
submitValues(this);
});
jQuery('#retirementMoney').blur(function () {
submitValues(this);
});
jQuery('#formSubmit').submit(function () {
checkForm(this);
return false;
});
});
</script>

```

完成的代码可以在`source code`文件夹下的`Chapter 3`中找到，文件名为`perfect-code-for-JSLint.html`。你可以将这个与你编辑的代码进行比较，看看你是否理解了我们试图做什么。现在，你可能想将代码复制粘贴到 JSLint 中看看效果如何。你将只会看到与 Jquery 使用相关的错误，一个关于使用`alert()`的验证错误，以及另一个关于使用太多`var`声明的错误。

## 刚才发生了什么？

我们已经纠正了大部分的验证错误，从极其大量的验证错误减少到少于十个验证错误，其中只有两个或三个与我们的代码相关。

### 注意

你可能会注意到`jQuery not defined`错误。尽管 JSLint 捕获了外部链接的 JQuery 库，但它并不显式阅读代码，因此导致了`jQuery not defined`错误。

现在我们已经修复了验证错误，让我们接下来使用另一个免费的验证工具——JavaScript Lint。

# JavaScript Lint—一个你可以下载的工具。

JavaScript Lint 可以在[`www.javascriptlint.com`](http://www.javascriptlint.com)下载，其工作方式与 JSLint 类似。主要区别在于，JavaScript Lint 是一个可下载的工具，而 JSLint 作为一个基于网页的工具运行。

与 JSLint 一样，JavaScript Lint 能够找出以下常见错误：

+   行尾缺少分号：在每行末尾都要加上分号。

+   没有`if, for`和`while`的括号。

+   不执行任何操作的语句。

+   在 switch 语句中的 case 语句将小数点转换为数字。

您可以通过访问其主页[`www.javascriptlint.com`](http://www.javascriptlint.com)了解更多关于它的功能。

要了解如何使用 JavaScript Lint，您可以跟随网站上找到的教程。

+   如果您使用 Windows，您可能需要阅读在[`www.javascriptlint.com/docs/running_from_windows_explorer.htm`](http://www.javascriptlint.com/docs/running_from_windows_explorer.htm)找到的设置说明。

+   如果您使用基于 Linux 的操作系统，您可以查看在[`www.javascriptlint.com/docs/running_from_the_command_line.htm`](http://www.javascriptlint.com/docs/running_from_the_command_line.htm)找到的说明。

+   最后，如果您希望将 JavaScript Lint 集成到您的 IDE（如 Visual Studio）中，您可以通过访问[`www.javascriptlint.com/docs/running_from_your_ide.htm`](http://www.javascriptlint.com/docs/running_from_your_ide.htm)了解更多有关如何执行此操作的信息。

我们不会讨论如何修复由 JavaScript Lint 发现的验证错误，因为这些原则与 JSLint 相似。然而，我们挑战你修复剩余的错误（除了由 JQuery 引起的错误）。

## 挑战自己——修复 JSLint 发现的剩余错误。

好的，这是我将向您提出的第一个挑战。修复 JSLint 发现的剩余错误，具体如下：

+   **"alert is not defined"：** 此错误在`alertMessage()`函数中找到。

+   **太多的 var 声明：** 此错误在`submitValues()`函数中找到。

以下是一些供您开始的想法：

+   在我们的 JavaScript 应用程序中，有没有办法避免使用`alert()`？我们如何显示能吸引观众注意力的信息，同时又是有效的？

+   对于在`submitValues()`函数中发现的错误，我们如何重构代码，使得函数中只有一个`var`声明？我们可以将`var`声明重构为一个函数，并让它返回一个布尔值吗？

好的，现在你可能想试试，但要注意，你所提出的或打算使用的解决方案可能会导致其他验证错误。所以你可能会在实施之前考虑一下你的解决方案。

# 总结

我们终于完成了这一章的结尾。我首先开始总结我们用来编写有效代码的一些策略和小贴士，然后概述了章节的其余部分。

我们用来编写有效代码（根据 JSLint）的某些策略如下：

+   适当间距你的代码，特别是在数学符号后，`if, else, ( )`等地方

+   每个函数中只使用一个`var`声明

+   考虑你的程序流程；编写代码时，确保所需数据或函数位于程序顶部

+   谨慎使用`alert()`函数。相反，将你的`alert()`函数合并成一个函数

+   使用`===`而不是`==`；这确保了你的比较语句更加准确

+   避免使用 HTML 事件处理程序，而是使用监听器。另外，你可以借助 JavaScript 库（如 JQuery）提供事件监听器给你的代码。

最后，我们讨论了以下主题：

+   测试与验证之间的区别

+   验证如何帮助我们写出好代码

+   如果我们不验证代码，可能会出现什么问题——如果我们不验证代码，它可能不具备可扩展性，可读性较差，并可能导致意外错误

+   如何使用 JSLint 和 JavaScript Lint 验证我们的代码

既然我们已经学习了如何通过验证工具测试 JavaScript，你可能想考虑一下当我们打算测试代码时可以采用的策略。正如本章中的示例所示，编写有效代码（或修正无效代码）是一个非常繁琐的过程。更重要的是，有些验证警告或错误并不影响我们程序的整个运行。在这种情况下，你认为验证代码值得花费精力吗？还是认为我们应该追求完美，写出完美的代码？这很大程度上取决于我们的测试计划，这将决定测试的范围、要测试的内容以及其他许多内容。这些主题将在下一章第四章，*计划测试*中介绍。所以，我将在本章结束，下章再见。
