# 第四章：创建可访问的体验

在本章中，我们将涵盖：

+   测试浏览器支持

+   添加跳转导航

+   添加元标记

+   在标签中使用语义描述供屏幕阅读器使用

+   提供替代站点视图

+   使用`hgroup`创建可访问的标题区域

+   为不支持的浏览器显示替代内容

+   使用 WAI-ARIA

# 介绍

> "良好的可访问性设计就是良好的网页设计。"

到目前为止，我们已经谈了很多关于语义网页编码以及 HTML5 如何使我们能够将这种命名方法提升到以前无法达到的新水平。我们的讨论大部分集中在语义网页编码如何使我们作为网页开发人员的工作更加轻松、快速和有意义。

在本章中，我们将关注语义网页编码如何改善我们的受众在线体验。现在，应用语义标签——有意义的标签而不仅仅是表现性的标签——对于屏幕阅读器和依赖它们来浏览我们创建的网站、应用程序和接口的人来说变得更加重要。

如果你曾经为军方、学术机构或者任何从美国联邦政府获得资金的人编写过网站、应用程序或接口，你一定听说过第 508 条。

与 HTML 或 CSS 验证不同，第 508 条验证方式不同。在 HTML 或 CSS 中，代码要么有效，要么无效。这是二进制的。但在第 508 条中，情况并非如此。在这种情况下，有三种不同级别的验证，每一级别都更难达到。

在本章中，我们将探讨如何使用 HTML5 测试浏览器支持，添加跳转导航和元标记，为屏幕阅读器使用标签提供语义描述，提供替代站点视图，使用新的 HTML5`hgroup`元素创建可访问的标题区域，为不支持的浏览器显示替代内容，并使用 WAI-ARIA。

现在，让我们开始吧！

# 测试浏览器支持

让我们从使用由 Faruk Ates 和 Paul Irish 开发的开源 Modernizr 项目开始。根据该网站，Modernizr 使用特性检测来测试当前浏览器对即将推出的特性的支持情况。

### 提示

Modernizr 概念旨在进行特性检测而不是浏览器检测。这是一个微妙但重要的区别。与进行广泛的假设不同，Modernizr 方法检测浏览器支持的特性。

![测试浏览器支持](img/1048_04_01.jpg)

## 如何做到...

下载 Modernizr JavaScript 文件并在你的标记的`head`部分引用它。然后将"no-js"类添加到你的`body`元素中，就像这样：

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Title</title>
<!--[if lt IE 9]><script src="img/html5.js"> </script>[endif]-->
<script src="img/modernizr-1.6.min.js"></script>
</head>
<body class="no-js">
</body>
</html>

```

## 它是如何工作的...

在你的标记中包含该脚本和简单的 body 类，可以使 Modernizr 检测浏览器支持的项目。然后它将添加类和 JavaScript API 来检测对某些功能的支持。如果给定浏览器不支持这些功能，Modernizr 就不会添加它们。

+   @font-face

+   画布

+   画布文本

+   HTML5 音频

+   HTML5 视频

+   rgba()

+   hsla()

+   边框图像：

+   边框半径：

+   盒阴影：

+   文本阴影：

+   不透明度：

+   多重背景

+   灵活的盒模型

+   CSS 动画

+   CSS 列

+   CSS 渐变

+   CSS 反射

+   CSS 2D 变换

+   CSS 3D 变换

+   CSS 过渡

+   地理位置 API

+   localStorage

+   sessionStorage

+   SVG

+   SMIL

+   SVG 裁剪

+   内联 SVG

+   拖放

+   哈希变化

+   X 窗口消息

+   历史管理

+   应用缓存

+   触摸事件

+   Web 套接字

+   Web Workers

+   Web SQL 数据库

+   WebGL

+   IndexedDB

+   输入类型

+   输入属性

## 还有更多...

在 Modernizr 2 Beta 中新增了自定义 JavaScript 下载的功能。现在，如果你不关心某个功能（比如拖放），你可以取消选择它，Modernizr 就不会检查它。详细信息请访问：[`modernizr.github.com/Modernizr/2.0-beta`](http://modernizr.github.com/Modernizr/2.0-beta)。

### 展望未来

来自网站：

> "Modernizr 是一个小巧简单的 JavaScript 库，它可以帮助您充分利用新兴的 Web 技术（CSS3、HTML5），同时仍然对尚未支持这些新技术的旧浏览器保持良好的控制。"

### Modernizr 真正做的是

Modernizr 不做的是在浏览器中添加或启用本来不存在的功能。例如，如果您的浏览器不支持输入属性，Modernizr 不会自动为您的浏览器添加该功能。这是不可能的。它只是让您作为开发人员知道您可以使用什么。

### 出于正确的原因而这样做

有些 Web 开发人员使用 Modernizr，因为他们在某篇文章中读到他们应该使用它。这没问题，但你比他们聪明。您知道在浏览器中检测这些功能如何更好地告知您如何为不支持某些属性的浏览器提供可访问的体验。聪明又英俊的你！

## 参见

进一步阅读，作者 Gil Fink 为微软撰写了简单而简洁的["使用 Modernizr 检测 HTML5 功能"](http://"Detecting)文章，网址为[`blogs.microsoft.co.il/blogs/gilf/archive/2011/01/09/detecting-html5-features-using-modernizr.aspx`](http://blogs.microsoft.co.il/blogs/gilf/archive/2011/01/09/detecting-html5-features-using-modernizr.aspx)。

# 添加跳过导航

跳过重复元素如导航的能力对于使用屏幕阅读器的人非常有益。想象一下，当访问网站时，您需要读取每个导航元素，然后才能继续查看主要内容。那会很烦人，不是吗？对于使用屏幕阅读器的人来说也是如此。让我们看看如何轻松地不让我们的一部分受众感到烦恼。

## 准备工作

在这个例子中，我们要做的是创建一个简单但特殊的不可见锚点，这将使我们的屏幕阅读器朋友有选择地跳过我们的导航，直接进入好东西：我们的网站内容。

## 如何做…

如果您在 HTML 领域有一段时间，毫无疑问您曾经创建过跳过导航。它可能看起来像这样：

```html
<a class="skip" href="#content">Skipnav</a>

```

您的 CSS 可能包括以下内容，以使锚点不可见：

```html
.skip {display: none}

```

包含您主要内容的第一个`div`然后包含了另一个看起来像这样的不可见锚点：

```html
<h2><a name="content"></a></h2>

```

多年来一切都很好。它运行得很好。在 HTML5 中也应该有效，对吧？嗯，猜怎么着？它不行。让我们看看为什么。

## 它是如何工作的…

在 HTML5 中，锚点标签的`name`属性不再有效。还记得我在第一章中说的关于创建 HTML5 规范的方法是“铺平牛路”的吗？嗯，这次不是。这次牛路已经被移除了。所以现在我们要做的是：

我们将保留初始的标记：

```html
<a class="skip" href="#content">Skipnav</a>

```

我们将保留隐藏锚点的初始 CSS：

```html
.skip {display: none}

```

但这是我们将用第二个标记做出不同的地方：

```html
<h2 id="content"></h2>

```

当我们移除了锚点时，我们将其`name`属性从`ID`重命名并添加到`h2`中。现在它有效并符合 HTML5 规范。简单！

## 还有更多…

跳过导航的能力是我们开发人员可以为不同能力的受众提供支持的最常见且易于实现的事情之一。考虑重新访问您开发过的旧网站，更新（或添加）跳过导航，将`DOCTYPE`切换到 HTML5，您就可以使用最新技术，同时支持可访问性。

### 完整的浏览器支持

跳过导航是一项改变，幸运的是所有主要的网络浏览器都支持。作者在这本书中很少有机会这样说，所以这真是一种解脱！

### 少即是多

在不久的将来，当屏幕阅读器更新时，我们将能够使用 Web 可访问性倡议-可访问丰富互联网应用程序角色，并使用新的`nav`元素来实现跳过导航的功能，而不是通过显式链接。更少的标记意味着更好的效果！

Web 标准项目网站[`webstandards.org`](http://webstandards.org)采取了一种有趣的方法，只有在视力用户悬停在上面时才显示跳过`nav`。

![Less equals more](img/1048_04_02.jpg)

## 另请参阅

[`html5accessibility.com`](http://html5accessibility.com)是一个很棒的资源，提供了关于哪些新的 HTML5 元素在 Web 浏览器中得到了支持，以及对使用辅助技术的人有用的信息。

# 添加元标记

> “语言标记标识了人类为了向其他人传达信息而口头、书面或以其他方式传达的自然语言。计算机语言明确排除在外。HTTP 在 Accept-Language 和 Content-Language 字段中使用语言标记。”- 万维网联盟的超文本传输协议规范

## 做好准备

如果您正在考虑为您或您的客户的网站考虑可访问性（您应该考虑！），您将希望确保使用屏幕阅读器的人能够以您打算的语言阅读您的信息。我们将看看如何做到这一点。

## 如何做...

首先确定您希望网站阅读的语言。它可以是英语、法语、克林贡语或任何组合。请参阅最受欢迎的内容语言列表：[`devfiles.myopera.com/articles/554/httpheaders-contentlang-url.htm`](http://devfiles.myopera.com/articles/554/httpheaders-contentlang-url.htm)。

## 它是如何工作的...

我们已经在我们的通用模板中有一个英语内容语言元标记：

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Title</title>
<!--[if lt IE 9]><script src="img/html5.js"> </script>[endif]-->
<script src="img/modernizr-1.6.min.js"></script>
</head>
<body class="no-js">
</body>
</html>

```

简单的`<html lang="en">`就足以确保我们的网站以英语阅读。将其更改为其他语言也同样简单。对于法语，请使用`<html lang="fr">`，如果您是《星际迷航》的忠实粉丝，请使用`<html lang="x-klingon">`。请注意，克林贡语示例中的`x-前缀`表示实验性语言。

## 还有更多...

您还可以通过使用类似以下内容的方式指定多种语言：`<html lang="en, fr">`，用于英语和法语。请注意，由于我们引用了多个值，因此在值周围使用引号。

### 你在说什么？

> 注意：“如果未指定内容语言，则默认情况下内容适用于所有语言受众。这可能意味着发送者不认为它是特定于任何自然语言，或者发送者不知道它适用于哪种语言。”- 万维网联盟的超文本传输协议规范

### 一切都归结为 SEO

指定内容语言对搜索引擎也有好处，并使它们能够解析我们打算使用的语言的内容。我们当中谁不需要更多的搜索引擎优化呢？

### 我做到了吗？

如果您不指定元语言，最糟糕的情况会是什么？什么都不会发生，对吗？错。事实证明，如果我们不指定元语言，我们的宿敌 Internet Explorer 的旧版本将尝试猜测您打算使用的语言。正如我们已经看到的，有时 IE 的猜测是错误的。根据这篇文章，无害的用户输入可能会变成活动的 HTML 并执行，导致安全漏洞：[`code.google.com/p/doctype/wiki/ArticleUtf7`](http://code.google.com/p/doctype/wiki/ArticleUtf7)。

## 另请参阅

[`section508.gov`](http://section508.gov)是美国法典第 508 条规定的官方网站。尽管我们网页开发人员主要关注第 508 条如何适用于网络，但实际上它是一个更广泛的法律体系，定义了我们美利坚合众国如何适应那些在虚拟世界和现实世界中具有不同能力的人。

# 在标签中为屏幕阅读器提供语义描述

语义网页开发的方法不仅对我们这些开发网站、应用程序和界面的人有意义，对那些使用和与我们创建的体验进行交互的人也有意义。

## 准备好了

让我们回顾一下 HTML5 规范中的一些新的、更语义化的标签。

## 如何做...

新的 HTML5 标签包括：

+   <article>

+   <aside>

+   <audio>

+   <canvas>

+   <datalist>

+   <details>

+   <embed> - 不是一个新标签，但它最终在 HTML5 中验证

+   <figcaption>

+   <figure>

+   <footer>

+   <header>

+   <hgroup>

+   <keygen>

+   <mark>

+   <meter>

+   <nav>

+   <output>

+   <progress>

+   <rp>

+   <rt>

+   <ruby>

+   <section>

+   <source>

+   <summary>

+   <time>

+   <video>

+   <wbr>

## 它是如何工作的...

在列表中，以下新标签可以支持文本：

+   <article>

+   <aside>

+   <datalist>

+   <details>

+   <figcaption>

+   <figure>

+   页脚

+   <header>

+   <hgroup>

+   <keygen>

+   <mark>

+   <nav>

+   <output>

+   <section>

+   <source>

+   <summary>

+   <time>

+   <wbr>

该列表代表了绝大多数可用的新 HTML5 标签。使用这些更语义化的标签将为屏幕阅读器增加额外的含义和智能。

## 还有更多...

以下新标签还为我们提供了创造更丰富和更有意义的体验的机会：

+   <audio>

+   <embed>

+   <progress>

+   <video>

### 始终改进

调查一下您已经发布的具有辅助功能要求的项目。如果您仍然能够更新它们，这是一个重温它们并添加更多语义上有意义的标记的绝佳机会。记住：只是因为一个站点、应用程序或界面已经发布，这并不意味着您以后不能再次访问它。如果一个项目在发布时是一个失败，那么现在是更新它然后重新发布的绝佳时机。谁知道呢？它可能会变成一个完美的作品集，让您得到另一份工作！得分！

### 良好 SEO 的语义

使用越来越多的语义和有意义的标签不仅对使用屏幕阅读器的人有益，而且对搜索引擎优化也有益，因为搜索引擎将能够更智能地解析和理解您的代码。

### 格雷格终于学会了

语义网页开发对其他开发者也有好处。如果你用<nav>标签编码一个区域，那么你团队中的另一个开发者或将来在你的项目上工作的开发者将立即理解你的意图。作者曾经与一个开发者合作，他用毫无意义的命名，比如`<div id="banana">`，用于与香蕉无关的东西。那个开发者认为这是一种通过成为唯一知道某些标签含义的方式来确保工作。不幸的是，当他几年后编辑他之前创建的东西时，这种方法对他来说变得痛苦，他无法记住它的含义。教训是什么？不要惹自己以后生气！

## 另请参阅

[`caniuse.com`](http://caniuse.com)提供 HTML5、CSS3、SVG 等在台式机和移动浏览器中的兼容性表。该网站是了解哪些新标签可以被支持的宝贵帮助。它不断更新，不仅值得收藏，而且值得一遍又一遍地参考。

# 提供备用站点视图

驻剑桥，马萨诸塞州的网站开发者伊桑·马科特创建了一种他称之为“响应式网页设计”的方法，以支持具有不同尺寸显示屏的台式电脑以及移动设备 - 所有这些都使用一个代码库。虽然这种方法并非必需，但可以视为朝着创建可访问体验的另一步。让我们更仔细地看看他的方法。

## 准备好了

马科特在 2010 年 5 月 25 日的《A List Apart》杂志上发表了这篇文章：[`alistapart.com/articles/responsive-web-design`](http://alistapart.com/articles/responsive-web-design)。阅读这篇文章将让你提前了解本节的其余内容。

## 如何做...

让我们仔细看看 Jon Hicks 的作品集，网址为[`hicksdesign.co.uk`](http://hicksdesign.co.uk)，这是 Marcotte 方法的一个出色示例。

Hicks 在 27 英寸显示器上以全宽度看到的作品集。

![如何做...](img/1048_04_03.jpg)

调整窗口大小会导致网站从四列变为三列：

![如何做...](img/1048_04_04.jpg)

进一步调整窗口大小会导致网站从三列变为两列：

![如何做...](img/1048_04_05.jpg)

进一步调整窗口大小会导致网站从两列变为一列：

![如何做...](img/1048_04_06.jpg)

## 它是如何工作的...

通过在样式表中使用灵活网格以及`min-width`和`max-width`值，Hicks 创建了一个可以轻松适应不同显示尺寸的体验。让我们看看它是如何做到的。

## 还有更多...

这种新的灵活的前端网页开发方式使我们能够创建体验，无论设备分辨率如何都能正常工作。多亏了 Marcotte，我们不再被迫为每个设备创建单独的体验。当前状态：一次编码，到处显示。

我们从一个流动网格开始，列可以适应任何可用的屏幕空间，灵活的图像，并让媒体查询根据分辨率和视口提供独特的样式表。

这是一个样本媒体查询：

```html
@media screen and (max-width: 600px) {
body {
font-size: 80%;
}
#content-main {float: none; width: 100%;}
}

```

你可以很容易地看到，我们说的是当设备的最大宽度为 600 像素时，我们告诉`body`以 80%的高度显示字体。我们还指定 content-main `div`将是一个 100%宽度的单列。如果设备的最大宽度是 601 像素或更多，这些规则将被忽略。请注意，由于这个媒体查询指定了屏幕，如果用户打印页面，这些规则也将被忽略。

### 最小宽度

正如你可以想象的，如果我们能够为这样的窄宽度指定样式，你也可以为更宽的宽度指定其他样式，比如：

```html
@media screen and (min-width: 1024px)

```

请注意，我们仍然在媒体查询中针对屏幕进行定位，但现在我们说的是只有视口*大于*1024 像素时才应用一些样式。

### 我的数学老师是对的

那些习惯于使用像 Grid960 这样的固定网格布局系统的人可能会发现使用灵活网格一开始是一种心理挑战。正如 Marcotte 所解释的：

> “网格的每个方面，以及放置在其上的元素，都可以相对于其容器表达为比例。”

我们可以将基于像素的宽度转换为百分比，以保持比例不变，无论显示的大小如何。让我们从一个简单的例子开始：假设我们的`header`宽度为 600 像素，我们想在一个宽度为 1200 像素的设备上显示它。如果方程式是目标÷上下文=结果，那么 600÷1200= .5。我们的 CSS 看起来像这样：

```html
header {width: 50%; /* 600px / 1200px = .5 */}

```

那很容易。但是如果你想在 960 像素宽度上显示一个不同的`header`，宽度为 510 像素呢？简单的除法给我们留下了这个：510÷960= .53125。我们可以这样调整我们的 CSS：

```html
header {width: 53.125%; /* 510px / 960px = .53125 */}

```

通过将每个宽度定义为整体的一部分来重复这个过程，你将在通向多种设备的响应式显示的道路上取得很大进展。

### 更大总是更好吗？

流动图像更容易，因为没有数学可做。相反，只需包括：

```html
img {max-width: 100%;}

```

在你的样式表中，这些图像将永远不会超出你的显示或设备的宽度。

将这三种技术结合在一起可以为多个平台、浏览器甚至屏幕阅读器上的用户创建一个几乎无缝的网站体验。

## 另请参阅

2011 年，Marcotte 在[`books.alistapart.com/products/responsive-web-design`](http://books.alistapart.com/products/responsive-web-design)上发表了关于响应式网页设计的权威书籍。

# 使用 hgroup 创建可访问的标题区域

记得`hgroups`吗？当然记得。我们在前一章中看过它们，作为一种逻辑上组合相关标题标签的方法。现在，我们将看看这个新的 HTML5 元素如何具有附加的可访问性益处。

## 准备工作

之前，我们以 Roxane 的作品集为例：

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Roxane</title>
<!--[if lt IE 9]><script src="img/html5.js"> </script>[endif]-->
</head>
<body>
<header>
<hgroup>
<h1>Roxane is my name.</h1>
<h2>Developing websites is my game.</h2>
</hgroup>
</header>
</body>
</html>

```

在过去，我们可能会这样编写她网站的主体区域：

```html
<body>
<div>
<div>
<h1>Roxane is my name.</h1>
<h2>Developing websites is my game.</h2>
</div>
</div>
</body>

```

## 如何做...

对于屏幕阅读器来说，过时的第二个示例在语义上毫无意义。它不知道那些标题标签除了在同一个`div`中之外，还有其他任何相关性。

## 它是如何工作的...

现在，由于 WHATWG 的 HTML5 草案标准在[`whatwg.org/html5`](http://whatwg.org/html5)，我们和屏幕阅读器都明白`hgroup`是相关标题的分组。那又怎样，你会问？如果你不能依靠视力知道这些标题是相关的，难道你不希望有其他机制 - 比如屏幕阅读器 - 告诉你它们是相关的吗？

## 另请参阅

[`diveintoaccessibility.org`](http://diveintoaccessibility.org) 是一个了不起的 30 天逐步资源，可以更多地了解 508 部分和可访问性标准。作者 Mark Pilgrim（也是[`diveintohtml5.org`](http://diveintohtml5.org)的* Dive Into HTML5*作者）提供了易于理解的技巧，可以按人员，残疾，设计原则，Web 浏览器和发布工具分类。该网站已经存在几年了，但由于多年来可访问性标准并没有发生太大变化，因此仍然是一个宝贵的免费资源。

# 为不支持的浏览器显示替代内容

一些新的 HTML5 元素是如此新，以至于并非所有桌面浏览器都支持它们。那么我们怎么能假设所有屏幕阅读器都会支持它们呢？

## 准备就绪

幸运的是，我们可以放心地认为屏幕阅读器将支持常见的文本标签，如：

+   `<h1>`

+   `<h2>`

+   `<h3>`

+   `<h4>`

+   `<h5>`

+   `<h6>`

+   `<p>`

+   `<ul>`

+   `<ol>`

+   `<li>`

+   `<dl>`

+   `<dt>`

+   `<dd>`

以及其他预期的内容。但是对于那些新的 HTML5 元素呢，比如：

+   `<article>`

+   `<aside>`

+   `<audio>`

+   `<canvas>`

+   `<datalist>`

+   `<details>`

+   `<figcaption>`

+   `<figure>`

+   `<footer>`

+   `<header>`

+   `<hgroup>`

+   `<mark>`

+   `<meter>`

+   `<nav>`

+   `<output>`

+   `<progress>`

+   `<section>`

+   `<summary>`

+   `<time>`

+   `<video>`

这些是否会按我们的意图传达给用户？如果是，那太棒了。但如果不是，用户会得到什么信息？这是否有意义？当然，我们会同意我们最不想做的事情就是通过我们的新标签提供更少的含义。即使盲人也能看到这将是一个史诗般的失败。

## 如何做...

在撰写本文时，许多 HTML5 元素至少提供了与屏幕阅读器相同数量的语义信息。让我们更具体地看看每个新元素：

+   `<article>` - 与`div`具有相同的语义信息。

+   `<aside>` - 与`div`具有相同的语义信息。

+   `<details>` - 与`div`具有相同的语义信息。

+   `<figcaption>` - 与`div`具有相同的语义信息。

+   `<figure>` - 与`div`具有相同的语义信息。

+   `<footer>` - 与`div`具有相同的语义信息。

+   `<header>` - 与`div`具有相同的语义信息。

+   `<hgroup>` - 与`div`具有相同的语义信息。

+   `<meter>` - 与`div`具有相同的语义信息。

+   `<nav>` - 与`div`具有相同的语义信息。

+   `<output>` - 与`div`具有相同的语义信息。

+   `<progress>` - 与`div`具有相同的语义信息。

+   `<section>` - 与`div`具有相同的语义信息。

+   `<summary>` - 与`div`具有相同的语义信息。

然而，对于其他新的 HTML5 元素，它们的含义对于屏幕阅读器来说并不那么清晰：

+   `<audio>` - 语义信息似乎是一致的，但 Firefox 在内置滑块控件上存在问题，Internet Explorer 9 只有部分播放/暂停支持，Opera 具有良好的键盘支持，但没有实际的辅助技术支持。

+   `<canvas>` - 几乎不提供可用的语义信息来辅助技术。任何依赖新的 HTML `<canvas>`元素传达信息的人都必须极度谨慎。故意使用它会让观众无法进入。

+   `<datalist>` - 仅在 Opera 中可通过键盘访问。

+   `<mark>` - 不提供额外的语义信息。

+   `<time>` - 仅在 Opera 中存在键盘访问的问题。

+   `<video>`-语义信息似乎是一致的，但 Firefox 在内置滑块控件上有问题，Internet Explorer 9 只有部分播放/暂停支持，Opera 具有良好的键盘支持，但没有实际的辅助技术支持。

## 它是如何工作的...

目前，直到屏幕阅读器能够跟上所有新的 HTML5 元素，我们在决定使用哪些新标签以及我们打算用它们传达什么含义给使用辅助技术的人时必须小心。

## 另请参阅

艾米莉·刘易斯对 HTML 和 CSS 以及可用性、语义和可访问性都感到兴奋。她是我们需要更多的热情倡导者，以使前端网页开发世界蓬勃发展。请查看她出色的“Web Accessibility and WAI-ARIA Primer”，了解如何开始思考可访问性的未来，网址为[`msdn.microsoft.com/en-us/scriptjunkie/ff743762.aspx`](http://msdn.microsoft.com/en-us/scriptjunkie/ff743762.aspx)。

# 使用 WAI-ARIA

通过使用技术，我们已经开发出了在浏览器窗口中动态更新信息的方法，而无需手动从服务器刷新页面。对于有视力的人来说，这是一个福音，可以更快地检索信息并以更有用的方式呈现。但是当一个人看不见时会发生什么？他们如何知道页面上的信息已经以任何方式更新，而无需刷新页面，重新显示其内容，并让辅助技术再次完整地读给他们听？

## 准备就绪

可访问的富互联网应用程序（WAI-ARIA）是一项新兴的技术规范，与 HTML5 的许多新语义标签一样，迫使我们真正思考我们的内容以及我们想要如何向受众呈现它。我们可以使用 WAI-ARIA 来定义角色、属性和状态，以帮助我们定义我们的元素应该做什么。

基于 Marcotte 的响应式 Web 设计方法的 WAI-ARIA 概述，网址为[`w3.org/WAI/intro/aria`](http://w3.org/WAI/intro/aria)。

![准备就绪](img/1048_04_07.jpg)

## 如何做到...

还记得我们在 Roxane 的作品集中包含了新的 HTML5 `nav`元素吗？以下是我们如何使用 WAI-ARIA 添加额外含义的方法：

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Roxane</title>
<!--[if lt IE 9]><script src="img/html5.js"> </script>[endif]-->
</head>
<body>
<header>
<hgroup>
<h1>Roxane is my name.</h1>
<h2>Developing websites is my game.</h2>
</hgroup>
</header>
<nav role="navigation">
<ul>
<li><a href="#About">About</a></li>
<li><a href="#Work">Work</a></li>
<li><a href="#Contact">Contact</a></li>
</ul>
</nav>
</body>
</html>

```

因为并非每个浏览器都理解`nav`标签，也并非每个浏览器都理解它的角色。WAI-ARIA 为我们提供了这样做的方法。`role="navigation"`被称为“地标角色”，对于一个无视力的人来说，它就像现实世界中的实际地标一样：它让他们知道自己在哪里。

## 它是如何工作的...

现在，即使没有视力的人也可以通过这些地标角色知道页面上的变化。WAI-ARIA 通过“监视”更新来通知用户这些动态页面变化。而不是重新阅读整个屏幕，只呈现新信息。

## 还有更多...

WAI-ARIA 允许我们创建复杂的、数据驱动的对象，如下拉菜单、选项卡、滑块、文件树等，这些是有视力的用户多年来习惯的，并且通过使用这些角色，即使那些无法看到内容更新的人也会被通知它已经这样做了。

### 仍在等待浏览器支持

总是有一个陷阱，对吧？在这种情况下的陷阱是，与本书中描述的许多其他 HTML5 功能一样，WAI-ARIA 并不是所有辅助技术浏览器完全支持的。无论我们作为开发人员多么勤奋和善意，如果我们的目标受众没有具有 WAI-ARIA 支持的最新浏览器，他们将无法获得我们希望他们获得的信息。

### 这就是我的角色

还有一个问题：错误地使用 WAI-ARIA 实际上可能会使情况变得更糟。如果比你水平低的开发人员将`role="navigation"`分配给一些经常更新但允许用户跳过导航的内容，那么用户将永远不会知道信息正在更新。幸运的是，这可能永远不会发生在你身上，因为你会在同行代码审查中发现这个错误。伟大的力量伴随着巨大的责任。

### 首要考虑无障碍

如果您正在开发网站，并且需要支持残障人士，就必须从最初的启动会议开始非常小心，以确保满足他们的需求。最具可访问性的成功项目是从一开始就考虑人们的需求。试图在最后添加一系列功能只会让自己和项目失败。我们要么让人们和项目成功，要么让它们失败。你会选择哪一个？

## 另请参阅

要了解如何使用 WAI-ARIA 构建更具可访问性的网络，请阅读 Scott Gilbertson 的这篇出色的 WebMonkey 文章：[`webmonkey.com/2010/11/can-wai-aria-build-a-more-accessible-web`](http://webmonkey.com/2010/11/can-wai-aria-build-a-more-accessible-web)。

吉尔伯特森后来写了一篇关于使用 ARIA 角色为网站设计样式的绝妙资源：[`webmonkey.com/2011/01/styling-webpages-with-arias-landmark-roles`](http://webmonkey.com/2011/01/styling-webpages-with-arias-landmark-roles)。
