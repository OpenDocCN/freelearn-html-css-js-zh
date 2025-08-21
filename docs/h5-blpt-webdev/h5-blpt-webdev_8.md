# 附录 A.你现在是一名专家，接下来做什么？

我们已经准备好我们的网站了。我们学会了如何编写它的代码，使用构建脚本来构建它，以及将其部署到生产环境中，这样它就能顺利上线。你已经有效地掌握了 HTML5 Boilerplate 的学习。如果你对成为一个更好的网页开发者感兴趣，你可以花时间去了解网络相关的其他有用部分！让我们一起探索其中的一些。

# 为你的代码编写单元测试

我们为我们的网站编写了一些 JavaScript 代码。虽然浏览器会告诉我们代码是否编写错误，但没有办法告诉我们代码是否如预期工作。也许有些边缘情况我们没有考虑到。代码应该尽可能健壮，并处理所有预期的用例和大多数错误条件。你可以通过编写测试来测试你的代码调用的每个函数，从而确保这是可能的。

单元测试可以被认为是你的代码中最小的可测试部分。当你编写单元测试时，你确保代码的每个部分都正确运行。开始编写单元测试的最简单方法是使用测试套件。

`QUnit.js`是一个流行的基于浏览器的测试套件，用于在浏览器中测试你的代码。让我们在我们的为阳光与沙滩音乐节网站编写的代码中使用它。

## 创建测试环境

让我们在我们的项目中创建一个`tests`文件夹。

然后，我们从`code.jquery.com/qunit/qunit-1.9.0.js`下载`QUnit.js`，并从`code.jquery.com/qunit/qunit-1.9.0.css`下载相关的 CSS 文件`qunit.css`。这些文件的最新版本可以在`github.com/jquery/qunit`找到。

我们现在通过在`tests`文件夹中创建一个`tests.html`页面来创建一个测试环境，并具有以下代码：

```java
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>Tests for Sun n' Sand Festival Code</title>
<link rel="stylesheet" href="qunit-1.9.0.css">
</head>
<body>
<div id="qunit"></div>
<div id="qunit-fixture"></div>
<script src="img/jquery.min.js"></script>
<script>window.jQuery || document.write('<script src="img/jquery-1.7.2.min.js"><\/script>')</script>
<script src="img/qunit-1.9.0.js"></script>
<script src="img/main.js"></script>
<script src="img/test.js"></script>
</body>
</html>
```

在这段代码中，我们包括了我们的`main.js`文件，这个文件是我们网站所使用的。我们将测试我们为显示阵容而编写的标签代码。

现在，我们将创建一个`test.js`文件，我们将在其中编写我们代码的所有测试。

由于我们的测试依赖于用于标签的标记，让我们从`index.html`中复制不含内容的标记到`tests.html`。

如果我们按原样执行这个测试，我们会得到一个全局失败的错误。如果你打开浏览器开发者工具的控制台，你应该会看到以下的错误：

```java
Uncaught TypeError: Object [object Object] has no method 'smoothScroll'
```

这是因为我们从`main.js`中调用插件，但我们在这里没有包含这些插件，因为我们不是在测试它们。我们可以在 QUnit 没有被使用的情况下，只测试插件和框架的存在，如以下代码片段所示：

```java
if(window.QUnit == undefined) {
  $('.js-scrollitem').smoothScroll();
  if(Modernizr.svg === false) {
    $('img[src$=".svg"]').each(function() {
      this.src = /(.*)\.svg$/.exec(this.src)[1] + '.png';
    });
  }

  if (Modernizr.generatedcontent === false && window.onbeforeprint !== undefined) {
    window.onbeforeprint = printLinkURLs;
    window.onafterprint = hideLinkURLs;
  }

  Modernizr.load({
    test: Modernizr.audio,
    nope: {
      'mediaelementjs': 'js/vendor/mediaelement/mediaelement-and-player.min.js'
},
    callback: {
    'mediaelementjs': function() {
      $('audio').mediaelementplayer();
    }
  } 
 });
}
```

确保你移除生产代码中的条件—`if(window.QUnit == undefined)`。

现在，让我们写一个测试，以确认当一个导航标签被点击时，正确的类被应用于它，使用以下的代码片段：

```java
$('.js-tabitem').each(function() {
  var $this = $(this);
  $this.trigger('click');
  test( "navigation tabs", function() {
    ok($this.hasClass('t-tab__navitem--active'), 
   'The clicked navigation item has the correct active class applied');
  });
});
```

`test()`函数是 QUnit 测试套件中可用的函数。第一个参数是文本的标题，第二个参数是你想要执行的实际测试函数。

我们还使用`ok()`，这是 QUnit 测试套件中的一个断言。断言是单元测试的基本元素，在这里你测试你的代码执行结果是否返回期望的值。QUnit 有不同种类的断言，具体请参阅`api.qunitjs.com/category/assert/`。

在`ok()`函数中，我们传递给这个函数的第一个参数是一个表达式，该表达式计算结果为真或假。第二个参数是在断言执行时你希望显示的信息。

现在，让我们通过以下代码段来测试非活动导航项是否不包含使导航项显示为活动的类名：

```java
$('.js-tabitem').not(this).each(function() {
  ok(!$(this).hasClass('t-tab__navitem--active'),
    'Inactive item does not have active class');
});
```

现在让我们执行这些测试！在你的浏览器中打开`tests.html`页面。你应该会看到类似下面的截图：

![创建测试环境](img/8505_App_01.jpg)

你还可以执行更复杂的测试！详细了解 QUnit 请访问他们的在线食谱本`qunitjs.com/cookbook/`。

### 你应该知道的神秘默认设置

为了得出 HTML5 Boilerplate 中的默认设置，进行了大量的研究。了解不同浏览器的行为以及我们为何选择这些默认设置是非常有趣的。

## 元 UTF-8

`meta`元素代表页面的任何元数据信息。在`<head>`元素中设置`<meta charset="utf-8">`将确保在没有关于页面编码的其他信息时，浏览器以 UTF-8 编码解析页面。

需要注意的是，大多数浏览器只在页面的前 512 字节内寻找字符编码元数据。因此，如果你在`<head>`元素中有大量的数据，你需要确保这个元元素出现在其他一切之前。

在没有`charset`编码信息的情况下，浏览器必须猜测应应用哪种`charset`编码。HTML5 规范概述了所有浏览器必须实现的嗅探算法，具体请参阅[www.whatwg.org/specs/web-apps/current-work/multipage/parsing.html#encoding-sniffing-algorithm](http://www.whatwg.org/specs/web-apps/current- work/multipage/parsing.html#encoding-sniffing-algorithm)。不幸的是，旧版浏览器有自己猜测字符编码的机制。

在 Internet Explorer 7 及以下版本的情况下，默认的字符编码偏好通常设置为`Auto Select`。这意味着浏览器扫描页面的内容以检测最适合的字符编码。在 Internet Explorer 中，如果在页面的前 4096 个字符内找到一个 UTF-7 字符串，它将假设页面使用 UTF-7 编码，你的页面将变得容易受到使用 UTF-7 编码的跨站脚本攻击。因此，在`index.html`页面顶部使用`meta`元素声明。

请注意，如果你的服务器发送了一个编码不同的 HTTP 头，那么这将优先考虑。确保你的服务器设置为在 HTTP 头中提供正确的`charset`编码。

## HTML Doctype

在 HTML 和 CSS 标准化之前，大多数标记和样式在任何一个浏览器中都无法一致地渲染。但是当我们有了关于标记应该如何编写的标准，越来越多的开发者开始采用这些标准时，浏览器不得不面对的问题是在互联网上的哪些页面符合这些标准，哪些页面不符合。

文档类型（Doctype）的发明是为了让开发者能够通知浏览器使用较新的标准模式来渲染页面。没有 Doctype 声明，浏览器将使用所谓的**怪异模式**（浏览器以前在标准成为可接受做法之前渲染页面的方式）来渲染页面。在 IE6 中，在 Doctype 上方有一个注释或 XML 命名空间声明会导致页面也以怪异模式渲染。在 2000 年代初建议使用带有 XML 命名空间声明的 XHTML Doctype 时，这将在 Internet Explorer 中引起重大问题。

并非所有的 Doctype 声明都在标准模式下渲染。使用标准模式的最简单方法是使用最小的推荐 Doctype，`<!doctype html>`。在 Doctype 声明中可以使用任何大写或小写的混合（例如，`<!DoCtYpE hTmL>`）。

## 清除解决方案的详细信息

`clearfix`CSS 类用于确保浮动元素适合其父容器。这个想法的第一次探索发生在 2002 年，并在[www.positioniseverything.net/easyclearing.html](http://www.positioniseverything.net/easyclearing.html)的文章中进一步阐述。

`clearfix`选择器按照以下方式工作：

```java
.clearfix:after {
  content: ".";
  display: block;
  height: 0;
  clear: both;
  visibility: hidden;
}
.clearfix { zoom: 1; } /* IE 5.5/6/7 */
```

这种方法最大的问题是，边距在所有浏览器上的一致性坍缩。Theirry Koblentz 在[www.tjkdesign.com/lab/clearfix/new-clearfix.html](http://www.tjkdesign.com/lab/clearfix/new-clearfix.html)上写了更多关于它。

蒂埃里·科布伦茨在 2010 年更新了这种方法，引入了`:before`和`:after`伪元素的使用，在[www.yuiblog.com/blog/2010/09/27/clearfix-reloaded-overflowhidden-demystified/](http://www.yuiblog.com/blog/2010/09/27/clearfix-reloaded-overflowhidden-demystified/)的一篇文章中进行了更新。这两个伪元素如下所示：

```java
.clearfix:before,
.clearfix:after {
  content: ".";
  display: block;
  height: 0;
  overflow: hidden;
}
.clearfix:after {clear: both;}
.clearfix {zoom: 1;} /* IE < 8 */
```

使用两个伪元素防止了在使用`clearfix`类时不一致的边距合并问题。

2011 年，尼古拉斯·加利亚尔发现了一种替代方法，如果我们的目标浏览器是 IE6 及以上版本和其他现代浏览器，将大大减少`clearfix`类的代码行数，他在`nicolasgallagher.com/micro-clearfix-hack/`的文章中解释了这一点。尼古拉斯的代码如下所示：

```java
.cf:before,
.cf:after {
    content: " ";
    display: table;
}

.cf:after {
    clear: both;
}

/**
 * For IE 6/7 only
 * Include this rule to trigger hasLayout and contain floats.
 */
.cf {
    *zoom: 1;
}
```

在这种方法中，使用`display: table`会在伪元素内创建一个匿名表格单元（有关这意味着什么的更多信息可以在规范[www.w3.org/TR/CSS2/tables.html#anonymous-boxes](http://www.w3.org/TR/CSS2/tables.html#anonymous-boxes)中找到），这防止了顶端边距的合并。`content`属性不需要任何内容在其中工作，但这种方法使用一个空格字符来克服当在可编辑元素上使用时的 Opera 错误。

这就是`clearfix`类的发展过程！正如你所看到的，为了制作可能跨越主要浏览器平台的最佳`clearfix`类，进行了大量的研究和开发。

## 打印样式做什么？

HTML5 Boilerplate 样式表带有一组在用户打印您的页面时非常有用的默认样式。设计一个页面在打印时的外观是我们在设计网页时大多数不会考虑的事情，而 HTML5 Boilerplate 为您提供了一组良好的默认值，因此您大多数时候不需要考虑（然而，这样做是一个好的实践）。

### 打印媒体查询

我们将所有的打印样式内嵌在一个名为“print”的 CSS 媒体查询中。当用户选择打印页面时，这个媒体查询会被匹配，在这种情况下应用这些样式规则。我们在下面的代码片段中展示了所有的规则都声明在`@media print`查询内：

```java
@media print {
  a, a:visited { text-decoration: underline; }
  /* More Styles below */
}
```

### 优化颜色和背景

然后我们确保优化页面，使其在打印时最易读，并确保我们不会浪费太多的打印墨水打印不必要的图片、颜色和文字。这意味着我们确保移除所有背景图片或图片，这些图片对于所有元素来说只是稍微不同的白色或透明色。我们还确保所有的颜色都是黑色，因为这意味着打印机不需要混合任何墨水，因此可以更快地打印。我们还移除了阴影，因为这会使文字更难读。

我们针对这些更新的最后一条规则如下：

```java
* {
        background: transparent !important;
        color: #000 !important; /* Black prints faster: h5bp.com/s */
        box-shadow:none !important;
        text-shadow: none !important;
    }
```

### 更好的链接

现在很少有设计师使用 `text-decoration: underline` 来为页面上的链接设置样式。通常，人们使用颜色来指示某物是链接。然而，在打印时，下划线更容易辨认，尤其是当你无法控制打印机和渲染它们的颜色时。因此，我们让所有链接（活动或已访问）通过以下代码片段使用下划线样式：

```java
    a,
    a:visited {
        text-decoration: underline;
    }
```

在打印时提供实际链接的参考也会很有帮助，因为用户如果从打印的页面阅读并希望访问链接，没有办法导航到该链接。我们通过在 CSS 中使用 `attr()` 函数来实现。`attr()` 返回当前规则将应用于的元素的属性的值。在这种情况下，由于我们将其应用于链接，我们可以使用 `attr()` 来获取链接的 `href` 属性的值并打印它们。当它们作为 `content` 属性的值使用时，使用空格字符将字符串连接在一起。我们还希望确保如果链接有标题，我们也打印出来，因为标题只有在悬停在链接上时才可见。所有这些在 CSS 中表达出来就像以下的代码片段：

```java
    a[href]:after {
        content: " (" attr(href) ")";
    }

    abbr[title]:after {
        content: " (" attr(title) ")";
    }
```

但是，这意味着即使是链接到同一页面其他位置的链接或用于 JavaScript 操作（带有 `javascript:` 前缀）的链接也会以同样的方式呈现！所以，我们需要确保我们不对这些链接这样做。

为此，我们使用属性选择器，它允许我们选择具有以特定值开始、结束或包含的属性的元素。通过使用选择器 `a[href^="javascript:"]:after`，我们确保我们只选择具有 `href` 属性的链接的 `:after` 伪元素，该属性的值以 `javascript:` 字符串开头。

同样，我们也选择所有 `href` 属性以 `#` 字符开头的链接，因为这意味着这样的链接是内联链接，链接到同一页面内的另一个位置。

然后我们确保对 these links 中的 pseudo-elements 不渲染任何内容。规则看起来像以下的代码片段：

```java
    .ir a:after,
    a[href^="javascript:"]:after,
    a[href^="#"]:after {
        content: "";
    }
```

请注意，这些规则不适用于 IE6，如果必须在 IE6 中提供此功能，您可能需要使用提供此功能的 JavaScript。

### 在同一页面内渲染所有代码和引用

有时，您的打印页面可能包含引用或代码，作为读者，当代码（或引用）本可以在一个页面内无任何中断地包含时，需要不断参考之前的页面是很烦人的。为此，我们可以使用 CSS `page-break-inside` 属性，它允许您告诉浏览器您更愿意让这些元素在两页之间断开还是保持在同一页面上。下面的代码片段显示了此代码：

```java
pre,
    blockquote {
        border: 1px solid #999;
        page-break-inside: avoid;
    }
```

请注意，Firefox 不支持 `page-break-inside`，但在所有其他浏览器中都可以使用。

### 更好地渲染表格

默认情况下，将标题放在`thead`标签内将确保当表格跨两页时，标题会重复。然而，目前只有 Firefox 和 Opera 支持这一点。在 IE 中，你可以这样做，但你必须明确指出，如下面的代码片段所示：

```java
    thead {
        display: table-header-group; /* h5bp.com/t */
    }
```

### 更好地渲染图像

理想情况下，我们想要防止表格行和图像跨页断裂，因此我们使用现在熟悉的`page-break-inside`属性来告诉浏览器我们的偏好，如下面的代码片段所示：

```java
    tr,
    img {
        page-break-inside: avoid;
    }
```

当图像超出页面或打印时被裁剪而在网站上以完整形式显示时，它也不太好看。因此，我们将最大宽度限制为与页面本身一样宽，不超过此宽度，如下面的代码片段所示：

```java
    img {
        max-width: 100% !important;
    }
```

### 页面边距

`@page`规则允许你在打印时修改页面的属性。除了 Firefox 之外，所有浏览器都支持这个规则。这个规则将页边距设置为每页`0.5 cm`，如下面的代码片段所示：

```java
    @page {
        margin: 0.5cm;
    }
```

### 孤儿和寡妇的最优设置

**孤儿**是指出现在页面底部的文本行。**寡妇**是指出现在页面顶部的那些。我们确保文本行不要以留下比所需更少的行在底部或顶部的方式断裂。这将创造一个更易读的体验。以下代码片段用于此目的：

```java
    p,
    h2,
    h3 {
        orphans: 3;
        widows: 3;
    }
```

### 保持标题与内容在一起

如果标题出现在一页的底部，而其对应的内容却出现在下一页，这将使得内容难以阅读。为了告诉浏览器避免这样做，我们可以使用`page-break-after`设置，如下面的代码片段所示：

```java
    h2,
    h3 {
        page-break-after: avoid;
    }
}
```

## 协议相关 URL 是什么？

在 HTML5 Boilerplate 中，当我们提到 jQuery 时，我们是这样提到的：

```java
<script src="img/jquery.min.js">
</script>
```

请注意，我们没有在 URL 前面加上`http`或`https`；相反，它以`//`开头。这些被称为协议相关 URL，当你想在 HTTP 或 HTTPS 环境中使用一个协议无关的资源时，它们很有用。

当你使用 HTTPS 提供页面时，浏览器在页面加载使用 HTTP 协议的资产和资源时会抛出警告和错误。为了防止这种情况，你需要确保你使用 HTTPS 协议来请求所有资产。如果你使用相对 URL 来引用页面所在的父文件夹中的资产，这通常不是问题。然而，如果你在引用像 jQuery 的 CDN URL（前面提到）这样的外部 URL，那么你需要确保在页面使用 HTTPS 协议时使用`https`，而在页面使用 HTTP 协议时使用`http`前缀。

而不是用 JavaScript 来做这个决定，简单地省略协议可以确保浏览器在请求那个外部 URL 时使用当前的协议。在这种情况下，如果这个页面以 HTTPS 的形式提供，即`https://example.com`，那么请求的 URL 将是[`ajax.googleapis.com/ajax/libs/jquery/1.8.1/jquery.min.js`](https://ajax.googleapis.com/ajax/libs/jquery/1.8.1/jquery.min.js)。

您可以在`paulirish.com/2010/the-protocol-relative-url/`了解更多关于这个内容。

## 使用条件注释

历史上，IE6、IE7 和 IE8 是拥有最多 bug 和样式渲染不一致的浏览器。有多种方法可以向 IE8 及以下版本提供样式，以下是其中几种。

### 浏览器样式 hack

最常用的技术是在 CSS 样式规则中使用 hack，这些规则只针对一个浏览器。

对于 IE6 及以下版本，使用以下代码片段：

```java
* html #uno  { color: red }
```

对于 IE7，使用以下代码片段：

```java
*:first-child+html #dos { color: red }
```

对于 IE8，使用以下代码片段：

```java
@media \0screen {
   #tres { color: red }
}
```

还有更多针对两个或更多浏览器（或排除两个或更多浏览器）的 hack，全部列在`paulirish.com/2009/browser-specific-css-hacks/`这篇文章中。

这些 hack 的问题在于，首先它们利用浏览器解析技术的漏洞。如果浏览器修复了这些解析错误，它们可能就无法工作。幸运的是，我们不必担心 IE6 和 IE7 等旧浏览器。

这些 hack 也不易读，如果没有注释，就无法理解它们针对哪些浏览器。

这些方法的优势在于你可以保持你的样式规则在一起，而且你不需要为需要 hack 的浏览器提供单独的样式表。

### 服务器端浏览器检测

当它们向服务器发起请求时，浏览器会随请求发送一个 User Agent 字符串。服务器可以根据它们对 User Agent 字符串的解释提供不同的资源。例如，如果一个浏览器用以下的 User Agent 字符串将自己标识为 IE6：

```java
Mozilla/4.0 (compatible; MSIE 6.0; Windows XP)
```

然后，服务器可以回传一个不同的样式表给 IE6。虽然这看起来是一个简单、容易的解决方案，但问题出现在浏览器撒谎的时候。历史上，浏览器从未准确声称自己是哪个浏览器，因此，很可能会向一个浏览器发送错误的样式表。

这也涉及到一点服务器端的处理开销，以根据浏览器的 User Agent 设置处理请求，因此这不是向 IE8 及以下版本提供不同样式表的理想方式。

### 基于条件注释的样式表

条件注释是 IE9 及以下版本能理解的具有特殊语法的 HTML 注释。以下是一个条件注释的示例：

```java
<!--[if lt IE 9]>
<p>HTML Markup here</p>
<!--<![endif]-->
```

所有浏览器（除了 IE9 及以下版本）都会忽略这些条件注释内的内容。IE9 及以下版本会尝试解释这些注释内的`if`条件，并根据 IE 浏览器的版本号是否与`if`条件中的版本号匹配来选择性地渲染内容。

之前的示例将在所有 8、7、6 及以下版本的 IE 上渲染`p`标签。

条件注释完美地针对老版本的 IE，HTML5 Boilerplate 就是这么做的。使用它们有两种方法。第一种是基于匹配条件注释输出一个单独的样式表，如下面的代码片段所示：

```java
<!--[if lt IE 9]>
<link rel="stylesheet" href="/css/legacy.css">
<![endif]-->
```

这将使 IE8 及以下使用`legacy.css`，其他浏览器将忽略这段代码。

独立样式表的问题在于，在开发样式时，您需要针对两个不同的样式表，偶尔 IE 特定的样式表可能会被遗忘。

有些人只为 IE8 及以下提供一个非常基础的体验，如下面的代码片段所示：

```java
<!--[if ! lte IE 6]><!-->
/* Stylesheets for browsers other than Internet Explorer 6 */
<!--<![endif]-->
<!--[if lte IE 6]>
<link rel="stylesheet" href="http://universal-ie6-css.googlecode.com/files/ie6.1.1.css" media="screen, projection">
<![endif]-->
```

但 HTML5 Boilerplate 更喜欢一个更可读且针对性的方法，使用类名向所有浏览器提供最佳可能的样式，我们接下来会看到。

### 基于条件注释的类名

前一个条件注释方法的迭代将是基于条件注释在根元素上附加类名，如下面的代码片段所示：

```java
<!--[if IE 8]>
<html class="no-js lt-ie9">
<![endif]-->
```

然后，在您的样式表中，您可以使用它在 IE8 及以下设置样式，如下所示：

```java
.lt-ie9 h1 { color: red }
```

您可以在`paulirish.com/2008/conditional-stylesheets-vs-css-hacks-answer-neither/`上阅读更多关于这个解决方案的信息。

这个解决方案不需要单独的样式表，但允许你编写可读的类名，表明样式表中为何存在该样式规则。这是我们在 HTML5 Boilerplate 中采用的解决方案，并推荐使用。

## 什么是元标签 x-ua-compatible？

`x-ua-compatible`是一个头部标签，用于定义 Internet Explorer 如何渲染您的页面。它声明了 Internet Explorer 应使用哪种模式来渲染您的页面。这主要针对那些在 Internet Explorer 9 及以后版本中因对标准支持更好而断裂的老网站。它可以以两种方式设置。

### HTML 页面中的元标签

在这种情况下，我们只需在 HTML 页面的`<head></head>`标签之间添加一个`meta`标签，如下所示：

```java
<head>
<meta http-equiv="X-UA-Compatible" content="IE=EmulateIE7" >
</head>
```

### HTTP 头响应自服务器

在 Apache 中，在`.htaccess`文件中编写以下内容，会使服务器对那个父文件夹的任何请求发送`X-UA-Compatible` HTTP 头作为响应：

```java
LoadModule headers_module modules/mod_headers.so
Header set X-UA-Compatible "IE=EmulateIE7"
```

我们推荐这种设置其值的方法，因为 HTTP 头值会覆盖通过`meta`标签设置的任何值。此外，在`html`元素上使用 IE 条件注释的`meta`标签会导致忽略这个`meta`标签。`X-UA-Compatible`头可以有以下值。

#### Edge

这将使用可用的最新渲染模式。例如，在 Internet Explorer 10 中，它将是 IE10。我们总是希望使用最新的渲染模式，因为这意味着我们能够访问最新的、最符合标准的浏览器版本。这就是为什么它是 HTML5 Boilerplate 中的默认选项。

#### IE9

这将只使用 IE9 模式来渲染页面。例如，当您使用这种模式时，如果这个页面在 Internet Explorer 10 中被查看，它将使用 IE9 模式来渲染页面。

#### IE8

这将渲染页面，好像它是在 Internet Explorer 8 上查看的一样。

#### IE7

这种模式会以 Internet Explorer 7 以标准模式渲染内容的方式渲染页面。

#### 模拟 IE9

这种模式告诉 Internet Explorer 使用`<!DOCTYPE>`指令来确定如何渲染内容。标准模式下的指令在 IE9 模式下显示，怪异模式下的指令在 IE5 模式下显示。所有模拟模式与之前的模式不同，都尊重`<!DOCTYPE>`指令。

#### 模拟 IE8

这种模式告诉 Internet Explorer 使用`<!DOCTYPE>`指令来确定如何渲染内容。标准模式下的指令在 IE8 模式下显示，怪异模式下的指令在 IE5 模式下显示。与 IE8 模式不同，模拟 IE8 模式尊重`<!DOCTYPE>`指令。

#### 模拟 IE7

这种模式告诉 Internet Explorer 使用`<!DOCTYPE>`指令来确定如何渲染内容。标准模式下的指令在 Internet Explorer 7 的标准模式下显示，而在 IE5 模式下显示怪异模式下的指令。与 IE7 模式不同，模拟 IE7 模式尊重`<!DOCTYPE>`指令。对于许多网站来说，这是首选的兼容性模式。

#### IE5

这种模式会以 Internet Explorer 7 在怪异模式下显示内容的方式渲染内容。您可以在 MSDN 文档`msdn.microsoft.com/en-us/library/cc288325(v=VS.85).aspx`上了解这些模式。

# 贡献

如果你喜欢这个项目到目前为止所看到的内容，你可能想要贡献！为 HTML5 Boilerplate 做出贡献在你的学习和理解中即使做出最小的更改也是有益的。贡献有两种方式，如下所述：

+   报告问题

+   提交拉取请求

## 报告问题

如果您在 HTML5 Boilerplate 的文件中发现了一个错误或者是不正确的内容，那么您可以提交一个问题，任何贡献者都可以查看并看看是否可以解决。

诀窍是找出是否是 HTML5 Boilerplate 的问题，或者是项目代码引起的。您可以通过开始安装 HTML5 Boilerplate 的干净版本并验证错误是否仍然发生来验证这是否是 HTML5 Boilerplate 的问题。

如果遇到 HTML5 Boilerplate 的问题，在提交问题时，请确保它没有被报告过。GitHub 问题页面`github.com/h5bp/html5-boilerplate/issues`列出了所有开放的问题。在顶部的**搜索**栏中搜索您遇到的问题。很可能它已经被修复，但修复还没有推送到稳定分支。

如果问题全新的，那么确保你通过一个减少的测试用例以一种明显的方式隔离问题（Chris Coyier 在`css-tricks.com/reduced-test-cases/`中撰写了关于减少测试用例的内容）。当你提交一个 bug 报告时，确保它易于理解，这样我们才能找到一个快速的解决方案。理想情况下，你的 bug 报告应该包含以下内容：

+   一个简短且描述性的标题

+   问题的概述以及此 bug 发生的浏览器/操作系统

+   如果可能的话，重现 bug 的步骤

+   一个减少的测试用例的 URL（你可以在`jsfiddle.net`或`codepen.io`上托管一个）

+   可能引起 bug 的代码行以及其他与 bug 相关的信息

理想情况下，一个 bug 报告应该是自包含的，这样贡献者不需要再次与你联系以了解关于 bug 的更多信息，而可以专注于解决它。

按照这个流程提交一个 bug 报告本身就是了解如何找出你编写的标记、样式或脚本中错误的的学习过程。

## 拉取请求

如果你有关于如何改进 HTML5 Boilerplate 的想法，或者有修补现有问题的补丁，改进或新功能，你可以提交所谓的**pull 请求**。Pull 请求是一组你可以提交给 HTML5 Boilerplate GitHub 存储库进行审查的更改，以便可以让核心贡献者审查并如果认为有用的话将其合并到 HTML5 Boilerplate 中。

开始贡献的一个好方法是找到一个你认为可以解决的小问题，分叉 GitHub 项目（在`help.github.com/articles/fork-a-repo`上了解这意味着什么），在你的更改上工作并提交一个 pull 请求。

如果你的贡献改变了大量代码并极大地改变了项目的性质，首先考虑在 GitHub 项目上打开一个 issues。

以下是要开始创建拉取请求的步骤：

+   分叉项目。

+   克隆你的分叉（在终端中，输入`git clone https://github.com/<your-username>/html5-boilerplate.git`并按*Enter*）。

+   在终端中添加一个上游远程（输入`git remote add upstream https://github.com/h5bp/html5-boilerplate.git`并按*Enter*）。

+   从上游获取最新更改（例如，通过在终端中输入`git pull upstream master`并按*Enter*）。

+   创建一个新的主题分支来包含你的功能、更改或修复（`git checkout -b <topic-branch-name>`）。

+   确保你的更改遵守项目整个中使用的当前编码约定；也就是说，缩进、准确的注释等。

+   将你的更改逻辑上分组提交；使用 Git 的交互式重置功能（关于此功能的更多信息请访问 `help.github.com/articles/interactive-rebase`）来在公开之前整理你的提交。请遵守这些 Git 提交信息指南（访问 `tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html`），否则你的拉取请求很可能不会被合并到主项目。

+   将上游分支合并（或重置）到你的主题分支。

+   将你的主题分支推送到你的分叉（`git push origin <topic-branch-name>`）。

+   用清晰的标题和描述打开一个拉取请求。请提到你测试了哪些浏览器。

这可能看起来像是很多工作，但它使你的拉取请求显著更容易理解且更快合并。此外，你的代码成为了你所做工作的文档，任何想要知道为什么某个部分是这样的都可以回溯到你的提交并确切了解原因。

在 HTML5 样板代码上工作将帮助你开始采用协作开发的最佳实践，你可以将这些实践带回你的工作场所或任何其他协作工作中。
