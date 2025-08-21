# 第四章：增强博客外观

*在前一章中，我们使用 HTML5 元素从标题部分到页脚部分构建了博客标记。然而，博客目前还是没有样式的。如果你在浏览器中打开博客，你只会看到它是空的；我们还没有编写样式来完善它的外观。*

*在本章的过程中，我们将专注于使用 CSS 和 JavaScript 装饰博客。我们将使用 CSS3 来添加博客样式。CSS3 带来了许多新的 CSS 属性，如`border-radius`、`box-shadow`和`box-sizing`，使我们能够装饰网站而无需添加图片和额外的标记。*

*然而，如前所述，CSS 属性仅适用于最新的浏览器版本。Internet Explorer 6 到 8 无法识别这些 CSS 属性，并且无法在浏览器中输出结果。因此，作为补充，我们还将利用一些 polyfill 来使我们的博客在旧版 Internet Explorer 中呈现出色。*

*这将是一个充满冒险的章节。让我们开始吧。*

在本章中，我们将涵盖以下主题：

+   研究 CSS3 属性和 CSS 库，我们将在博客中使用

+   使用 Koala 编译和压缩样式表和 JavaScript

+   采用移动优先的方法撰写博客样式规则

+   优化博客以适应桌面

+   使用 polyfill 在 Internet Explorer 中修复博客

# 使用 CSS3

CSS3 配备了期待已久的属性，`border-radius`和`box-shadow`，简化了在 HTML 中呈现圆角和阴影的旧方法。除此之外，它还带来了一种新类型的伪元素，使我们能够通过 HTML5 的`placeholder`属性来样式化输入字段中显示的占位文本。

让我们看看它们是如何工作的。

## 使用 CSS3 创建圆角

回到 90 年代，创建圆角是很复杂的。添加大量的 HTML 标记，切割图像，并制定多行样式规则是不可避免的，正如 Ben Ogle 在[`benogle.com/2009/04/29/css-round-corners.html`](http://benogle.com/2009/04/29/css-round-corners.html)的文章中所述。

CSS3 使得使用`border-radius`属性创建圆角变得更加简单，以下是一个例子：

```html
div {
  width: 100px; height: 100px;
  border-radius: 30px;
}
```

上述样式规则将使盒子的角变圆（阅读第一章中的*关于 CSS 盒模型的一些话*部分，*响应式网页设计*），每个角为`30px`，如下图所示：

![使用 CSS3 创建圆角](img/image00254.jpeg)

此外，我们还可以只将特定的角进行圆角处理。例如，以下代码片段将只圆角处理右上角：

```html
div {
  width: 100px; height: 100px;
  border-top-right-radius: 30px;
}
```

## 创建阴影

与创建圆角类似，过去在网站中创建阴影效果时不可避免地需要使用图片。现在，通过引入`box-shadow`属性，添加阴影效果变得更加容易。`box-shadow`属性由五个参数（或值）组成：

第一个参数指定了阴影的位置。这个参数是可选的。将值设置为`inset`，让阴影出现在盒子内部，或者留空以在外部显示阴影。

第二个参数指定了盒子的**阴影垂直**和**水平距离**。

第三个参数指定了**阴影模糊**，使阴影变淡；数字越大，产生的阴影就越大但也更加淡化。

第四个参数指定了**阴影扩展**；这个值与阴影模糊值略有矛盾。这个值会放大，同时也加强阴影的深度。

最后一个参数指定了颜色。颜色可以是任何网络兼容的颜色格式，包括 Hex、RGB、RGBA 和 HSL。

延续前面的例子，我们可以添加`box-shadow`，如下所示：

```html
div {
  width: 100px;
  height: 100px;
  border-radius: 30px;
  box-shadow: 5px 5px 10px 0 rgba(0,0,0,0.5);
}
```

上述代码将输出阴影，如下图所示：

![使用 CSS3 创建阴影](img/image00255.jpeg)

如果要在框内显示阴影，请在开头添加`inset`，如下：

```html
div {
  width: 100px;
  height: 100px;
  border-radius: 30px;
  box-shadow: inset 5px 5px 10px 0 rgba(0,0,0,0.5);
}
```

### 提示

CSS3 的`box-shadow`属性可以以许多创造性的方式应用，以下是 Paul Underwood 的一个示例，供您参考：

[`www.paulund.co.uk/creating-different-css3-box-shadows-effects`](http://www.paulund.co.uk/creating-different-css3-box-shadows-effects)

## CSS3 浏览器支持和厂商前缀的使用

`border-radius`和`box-shadow`属性在许多浏览器中得到了很好的实现。从技术上讲，如果我们只针对最新的浏览器版本，就不需要包括所谓的厂商前缀。

然而，如果我们打算在最早期的浏览器版本中启用这两个属性`border-radius`和`box-shadow`，在这些浏览器版本中，如 Safari 3、Chrome 4 和 Firefox 3，它们仍然被浏览器供应商标记为实验性的，需要添加厂商前缀。每个浏览器的前缀如下：

+   `-webkit-`：这是基于 WebKit 的浏览器前缀，目前包括 Safari、Chrome 和 Opera。

+   `-moz-`：这是 Mozilla Firefox 的前缀。

+   `-ms-`：这是 Internet Explorer 的前缀。但自 Internet Explorer 9 以来，Internet Explorer 已经支持`border-radius`和`box-shadow`，无需添加此前缀。

让我们继续我们之前的例子（再次）。通过添加厂商前缀以适应 Chrome、Safari 和 Firefox 的最早版本，代码如下：

```html
div {
  width: 100px;
  height: 100px;
  -webkit-border-radius: 30px;
  -moz-border-radius: 30px;
  border-radius: 30px;
  -webkit-box-shadow: 5px 5px 10px 0 rgba(0,0,0,0.5);
  -moz-box-shadow: 5px 5px 10px 0 rgba(0,0,0,0.5);
  box-shadow: 5px 5px 10px 0 rgba(0,0,0,0.5);
}
```

代码可能会变得有点长；但这仍然比应对复杂的标记和多个样式规则要好。

### 注意

**Chrome 及其新的浏览器引擎 Blink**

Chrome 决定分叉 WebKit，并在其上构建自己的浏览器引擎，名为 Blink（[`www.chromium.org/blink`](http://www.chromium.org/blink)）。Opera 之前放弃了其初始引擎（Presto）以使用 WebKit，现在也跟随 Chrome 的动向。Blink 取消了厂商前缀的使用，因此我们不会找到`-blink-`前缀或类似的前缀。在 Chrome 的最新版本中，Chrome 默认禁用实验性功能。但是，我们可以通过`chrome://flags`页面中的选项来启用它。

## 自定义占位符文本样式

随着 HTML5 的加入，占位符属性带来了如何自定义占位符文本的问题。默认情况下，浏览器以浅灰色显示占位符文本。例如，我们如何更改颜色或字体大小？

在撰写本文时，每个浏览器在这方面都有自己的方式。基于 WebKit 的浏览器，如 Safari、Chrome 和 Opera，使用`::-webkit-input-placeholder`。而 Internet Explorer 10 使用`:-ms-input-placeholder`。另一方面，Firefox 4 到 18 使用`伪类` `:-moz-placeholder`，但自 Firefox 19 以来已被伪元素`::-moz-placeholder`（注意双冒号）所取代，以符合标准。

这些选择器不能在单个样式规则中同时使用。因此，以下代码片段将不起作用：

```html
input::-webkit-input-placeholder,
input:-moz-placeholder,
input::-moz-placeholder,
input:-ms-input-placeholder { 
  color: #fbb034;
}
```

它们必须在单个样式规则声明中声明，如下：

```html
input::-webkit-input-placeholder {
  color: #fbb034; 
}
input:-moz-placeholder {
  color: #fbb034;
}
input::-moz-placeholder {
  color: #fbb034;
}
input:-ms-input-placeholder { 
  color: #fbb034;
}
```

这绝对是低效的；我们只是为了达到相同的输出而添加了额外的行。目前还没有其他可行的选择。对于样式化占位符的标准仍在讨论中（请参阅[`wiki.csswg.org/ideas/placeholder-styling`](http://wiki.csswg.org/ideas/placeholder-styling)和[`wiki.csswg.org/spec/css4-ui#more-selectors`](http://wiki.csswg.org/spec/css4-ui#more-selectors)中的 CSSWG 讨论了解更多细节）。

## 使用 CSS 库

区分 CSS 库和 CSS 框架的基本事情是它所解决的问题。例如，CSS 框架，如 Blueprint ([`www.blueprintcss.org/`](http://www.blueprintcss.org/))，旨在作为新网站的基础或起点。它通常附带各种库的组件，以涵盖许多情况。另一方面，CSS 库解决的是非常具体的问题。一般来说，CSS 库也不受限于特定的框架。`Animate.css` ([`daneden.github.io/animate.css/`](http://daneden.github.io/animate.css/)) 和 `Hover.css` ([`ianlunn.github.io/Hover/`](http://ianlunn.github.io/Hover/)) 就是两个完美的例子。它们都是 CSS 库。它们可以与任何框架一起使用。

在这里，我们将把两个 CSS 库整合到博客中，即`Normalize` ([`necolas.github.io/normalize.css/`](http://necolas.github.io/normalize.css/)) 和 `Formalize` ([`formalize.me/`](http://formalize.me/))。这些 CSS 库将标准化不同浏览器中的基本元素样式，并最小化可能意外发生的样式错误。

# 使用 Koala

一旦我们探索了这个项目中要包含的所有内容，让我们设置工具将它们整合在一起。在第一章中，*响应式网页设计*，我们已经安装了 Koala。Koala 是一个免费的开源开发工具，具有许多功能。在这个第一个项目中，我们将使用 Koala 来将样式表和 JavaScript 编译成一个文件，并将代码压缩以得到更小的文件大小。

我们将在博客中包含大约五个样式表。如果我们分别加载所有这些样式表，浏览器将不得不发出五个 HTTP 请求，如下面的截图所示：

![使用 Koala 工作](img/image00256.jpeg)

如前面的截图所示，浏览器执行了五个 HTTP 请求来加载所有样式表，总共大小为 24.4 KB，需要大约 228 毫秒来加载。

将这些样式表合并成一个文件并压缩其中的代码将加快页面加载速度。样式表也会变得显著更小，最终也会节省带宽消耗。

如下面的截图所示，浏览器只执行了一个 HTTP 请求；样式表大小减小到 13.5KB，加载时间只需 111 毫秒。与前面的例子相比，页面加载速度提高了约 50%：

![使用 Koala 工作](img/image00257.jpeg)

### 提示

**加快网站性能的最佳实践：**

前往 YSlow！性能规则 ([`developer.yahoo.com/performance/rules.html`](https://developer.yahoo.com/performance/rules.html)) 或 Google PageSpeed Insight 规则 ([`developers.google.com/speed/docs/insights/rules`](https://developers.google.com/speed/docs/insights/rules))，了解除了合并样式表和 JavaScript 之外，使网站加载更快的进一步步骤。

# 行动时间 - 将项目目录整合到 Koala 并合并样式表

在本节中，我们将整合配置好的 Koala 来编译和输出样式表，执行以下步骤：

1.  在`css`文件夹中创建一个名为`main.css`的新样式表。这是主要的样式表，我们将在其中为博客编写自己的样式规则。

1.  创建一个名为`style.css`的新样式表。

1.  下载`normalize.css` ([`necolas.github.io/normalize.css/`](http://necolas.github.io/normalize.css/))，并将其放入项目目录的`css`文件夹中。

1.  下载`formalize.css` ([`formalize.me/`](http://formalize.me/))，并将其放入项目目录的`css`文件夹中。

1.  在 Sublime Text 中打开`style.css`。

1.  使用`@import`规则按以下顺序导入支持的样式表，如下所示：

```html
@import url("css/normalize.css");
@import url("css/formalize.css");
@import url("css/responsive.gs.12col.css");
@import url("css/main.css");
@import url("css/responsive.css");
```

1.  启动 Koala。然后，将项目目录拖放到 Koala 侧边栏中。Koala 将显示并列出可识别的文件，如下截图所示：![行动时间-将项目目录整合到 Koala 中并合并样式表](img/image00258.jpeg)

1.  选择`style.css`并选中**自动编译**，以便在 Koala 检测到其中的更改时自动编译`style.css`。查看以下截图：![行动时间-将项目目录整合到 Koala 中并合并样式表](img/image00259.jpeg)

1.  选择**合并导入**选项，让 Koala 将样式表中的内容（包括`style.css`中包含的内容）与`@import`规则合并。查看以下截图：![行动时间-将项目目录整合到 Koala 中并合并样式表](img/image00260.jpeg)

1.  将**输出样式**设置为**压缩**。查看以下截图：![行动时间-将项目目录整合到 Koala 中并合并样式表](img/image00261.jpeg)

这将把样式规则压缩成一行，最终会使`style.css`文件大小变小。

1.  单击**编译**按钮。查看以下截图：![行动时间-将项目目录整合到 Koala 中并合并样式表](img/image00262.jpeg)

这将编译`style.css`并生成一个名为`style.min.css`的新文件作为输出。

1.  打开`index.html`并链接`style.min.css`，使用以下代码：

```html
<link href="style.min.css" rel="stylesheet">
```

## *刚刚发生了什么？*

我们刚刚在 Koala 中整合了项目目录。我们还创建了两个新的样式表，分别是`main.css`和`style.css`。我们还使用`@import`规则将五个样式表，包括`main.css`，合并到了`style.css`文件中。我们合并了这些文件，并生成了一个名为`style.min.css`的新样式表，它可以在`style.css`中找到，如下截图所示：

![刚刚发生了什么？](img/image00263.jpeg)

最后，我们在`index.html`中链接了压缩后的样式表`style.min.css`。

## 英雄试试看-重命名输出

`style.min.css`是 Koala 设置的默认名称；它会在每个压缩输出中插入后缀`min`。虽然这是压缩的 Web 源文件、样式表和 JavaScript 最流行的命名约定，但 Koala 允许您重命名输出以匹配您的个人喜好。要这样做，请单击以下截图中用圆圈标出的编辑图标：

![英雄试试看-重命名输出](img/image00264.jpeg)

以下是一些您可以尝试的替代命名想法：

+   `style-min.css`（带有破折号）

+   `styles.min.css`（带有`s`）

+   `blog.css`（指的是网站名称）

然而，如果您决定重命名输出而不是像我们在前面的步骤中那样管理`style.min.css`，请不要忘记更改`<link>`元素中指向样式表的名称。

## 快速测验-网站性能规则

Q1. 以下哪条规则不是用于改善网站性能的规则？

1.  压缩资源，如 CSS 和 JavaScript。

1.  压缩图像文件。

1.  利用浏览器缓存。

1.  使用 CSS 简写属性。

1.  使用 CDN 传递 Web 资源。

# 首先考虑移动端

在我们动手编写代码之前，让我们谈谈移动优先的方法，这将驱动我们在写部分博客样式规则时的决定。

移动优先是 Web 设计社区中的热词之一。移动优先是一种新的思维方式，用于构建今天的网站，也指导着构建针对移动使用优化的网站的模式。正如第一章中所述，*响应式 Web 设计*，移动用户正在增加，桌面不再是用户访问 Web 的主要平台。

移动优先的概念驱使我们在构建网站块时考虑和优先考虑移动使用，包括我们如何组合样式规则和媒体查询。此外，采用移动优先思维，正如 Brad Frost 在他的博客文章中所展示的（[`bradfrostweb.com/blog/post/7-habits-of-highly-effective-media-queries/`](http://bradfrostweb.com/blog/post/7-habits-of-highly-effective-media-queries/)），允许生成比另一种方式（从桌面到移动）更精简的代码。在这里，我们将首先优化和处理移动端的博客，然后再增强到桌面版本。

移动优先不在本书的范围之内。以下是一些我推荐的进一步了解这个主题的来源：

+   Luke Wroblewski 的《Mobile First》（[`www.abookapart.com/products/mobile-first`](http://www.abookapart.com/products/mobile-first)）

+   Brad Frost 的《Mobile First Responsive Web Design》（[`bradfrostweb.com/blog/web/mobile-first-responsive-web-design/`](http://bradfrostweb.com/blog/web/mobile-first-responsive-web-design/)）

+   Jeremy Girard 的《Building a Better Responsive Website》（[`www.smashingmagazine.com/2013/03/05/building-a-better-responsive-website/`](http://www.smashingmagazine.com/2013/03/05/building-a-better-responsive-website/)）

# 组合博客样式

在前面的章节中，我们添加了第三方样式，奠定了博客外观的基础。从本节开始，我们将为博客编写自己的样式规则。我们将从页眉开始，然后逐步到页脚。

# 采取行动-组合基本样式规则

在这一部分，我们将编写博客的基本样式。这些样式规则包括内容字体系列，字体大小，以及其中的许多元素。

首先，我个人认为使用默认系统字体如 Arial 和 Times 非常无聊。

### 注意

由于浏览器支持和字体许可限制，我们只能使用用户操作系统中安装的字体。因此，十多年来，我们在网页上只能使用非常有限的字体选择，许多网站使用相同的一组字体，如 Arial，Times，甚至 Comic Sans。所以，是的，这些都是无聊的字体。

如今，随着`@font-face`规范的进步，以及在网页上使用字体的许可，我们现在能够在网站上使用用户计算机字体选择之外的字体。现在也有更大的免费字体集合可以嵌入到网页中，比如我们可以在 Google Font（[`www.google.com/fonts`](http://www.google.com/fonts)）、Open Font Library（[`openfontlibrary.org/`](http://openfontlibrary.org/)）、Font Squirrel（[`www.fontsquirrel.com`](http://www.fontsquirrel.com)）、Fonts for Web（[`fontsforweb.com/`](http://fontsforweb.com/)）和 Adobe Edge Web Font（[`edgewebfonts.adobe.com/`](https://edgewebfonts.adobe.com/)）中找到的字体。

我真的鼓励网页设计师更多地探索使用自定义字体在他们的网站上构建更丰富的网站的可能性。

执行以下步骤来组合基本样式规则：

1.  为了使我们的博客看起来更加清新，我们将从 Google Font 库中使用一些自定义字体。Google Font 让我们能够在网页上使用字体变得更加容易。Google 已经解决了编写语法的麻烦，同时确保字体格式在所有主要浏览器中兼容。

### 注意

说到这一点，可以参考 Paul Irish 的文章，“Bulletproof @font-face syntax”（[`www.paulirish.com/2009/bulletproof-font-face-implementation-syntax/`](http://www.paulirish.com/2009/bulletproof-font-face-implementation-syntax/)），以获取有关在所有浏览器中工作的 CSS3 `@font-face` 语法的进一步帮助。

1.  此外，我们不会被字体许可证所困扰，因为 Google 字体是完全免费的。我们所要做的就是按照此页面中的说明添加一个特殊的样式表[`developers.google.com/fonts/docs/getting_started#Quick_Start`](https://developers.google.com/fonts/docs/getting_started#Quick_Start)。在我们的情况下，在主要样式表链接之前添加以下链接：

```html
<link href='http://fonts.googleapis.com/css?family=Droid+Serif:400,700,400italic,700italic|Varela+Round' rel='stylesheet'>
```

这样做后，我们将能够使用 Droid Serif 字体系列，以及 Varela Round；请在以下网页中查看这些字体样本和字符：

+   Droid Serif（[`www.google.com/fonts/specimen/Droid+Serif`](http://www.google.com/fonts/specimen/Droid+Serif)）

+   Varela Round（[`www.google.com/fonts/specimen/Varela+Round`](http://www.google.com/fonts/specimen/Varela+Round)）

1.  将整个元素框大小设置为`border-box`。在`main.css`中添加以下行（以及下一步中的其他行）：

```html
* { 
  -webkit-box-sizing: border-box;
  -moz-box-sizing: border-box;
  box-sizing: border-box;
  *behavior: url(/scripts/boxsizing.htc);
}
```

1.  我们将设置博客的主要字体，即适用于博客整个内容的字体。在这里，我们将使用 Google 字体的 Droid Serif。在`@import`样式表列表之后添加以下样式规则：

```html
body {
  font-family: "Droid Serif", Georgia, serif;
  font-size: 16px;
}
```

1.  我们将为标题（`h1`，`h2`，`h3`，`h4`，`h5`和`h6`）应用不同的字体系列，以使其与正文内容区分开来。在这里，我们将应用从 Google 字体收集中带来的第二个自定义字体系列，Varela Round。

1.  将以下行添加到标题应用 Varela Round：

```html
h1, h2, h3, h4, h5, h6 {
    font-family: "Varela Round", Arial, sans-serif;
    font-weight: 400;
}
```

### 注意

默认情况下，浏览器将标题的粗细设置为`bold`或`600`。然而，Varela Round 只有普通字重，相当于`400`。因此，如前面的代码片段所示，我们还将`font-weight`设置为`400`，以防止所谓的*faux-bold*。

有关 faux-bold 的更多信息，请参阅 A List Apart 文章*Say No to Faux Bold*（[`alistapart.com/article/say-no-to-faux-bold`](http://alistapart.com/article/say-no-to-faux-bold)）。

1.  在这一步中，我们还将自定义默认的锚标签或链接样式。我个人偏好去掉默认链接样式的下划线。

### 注意

即使 Google 也删除了其搜索结果的下划线（[`www.theverge.com/2014/3/13/5503894/google-removes-underlined-links-site-redesign`](http://www.theverge.com/2014/3/13/5503894/google-removes-underlined-links-site-redesign)）。

此外，我们还将链接颜色更改为`#3498db`。它是蓝色的，但比默认链接样式中应用的蓝色更为柔和，如下面的截图所示：

![行动时间-撰写基本样式规则](img/image00265.jpeg)

1.  添加以下行以更改默认链接颜色：

```html
a {
    color: #3498db;
    text-decoration: none;
}
```

1.  我们还将设置链接的悬停状态颜色。当鼠标光标悬停在链接上时，将显示这种颜色。在这里，我们将链接悬停颜色设置为`#2a84bf`，这是我们在第 4 步中设置的颜色的较暗版本。请看下面的截图：![行动时间-撰写基本样式规则](img/image00266.jpeg)

1.  添加以下行以设置链接在悬停状态时的颜色，如下所示：

```html
a:hover {
    color: #2a84bf;
}
```

1.  使用以下样式规则使图像具有流体性，如下所示：

```html
img {
  max-width: 100%;
  height: auto;
}
```

此外，这些样式规则将防止图像在实际图像宽度大于容器时超出容器。

### 注意

有关流体图像的更多详细信息，请参阅 A List Apart 文章*Fluid Images*（[`alistapart.com/article/fluid-images`](http://alistapart.com/article/fluid-images)）。

## *刚刚发生了什么？*

我们刚刚添加了一些样式规则，用于处理博客中的一些元素，即标题、链接和图像。在这个阶段，除了内容和标题中的字体系列更改以及链接颜色之外，博客中还没有出现明显的差异。请看下面的截图：

![刚刚发生了什么？](img/image00267.jpeg)

## 尝试一下-自定义链接颜色

请注意，链接颜色`#2a84bf`是我个人的选择。在选择颜色时有许多考虑因素，例如品牌、受众和内容。链接不一定要是`#2a84bf`。例如，星巴克网站（[`www.starbucks.co.id/about-us/pressroom`](http://www.starbucks.co.id/about-us/pressroom)）中的链接颜色是绿色，这与其品牌身份有关。

因此，不要害怕探索和尝试新的颜色。以下是一些颜色想法：

![尝试一下英雄-自定义链接颜色](img/image00268.jpeg)

接下来，我们将组合博客标题和导航样式规则。样式规则将主要通过元素的类应用。因此，在继续之前，请参考第二章*网页开发工具*，查看我们在元素中添加的类名和 ID。

# 行动时间-使用 CSS 增强标题和导航外观

步骤如下：

1.  打开`main.css`。

1.  使用`padding`在标题周围添加一些空白，并将标题颜色设置为`#333`，如下所示：

```html
.blog-header {
padding: 30px 15px;
background-color: #333;
}
```

1.  为了使标志看起来突出，我们将使用 Varela Round 字体，这是我们用于标题的相同字体系列。我们还会使它变大，并将所有字母转换为大写，如下所示：

```html
 .blog-name {
  font-family: "Varela Round", Arial, sans-serif;
  font-weight: 400;
  font-size: 42px;
  text-align: center;
  text-transform: uppercase;
}
```

1.  标志链接颜色目前为`#2a84bf`，这是我们为链接`<a>`设置的常用颜色。这种颜色与背景颜色不搭配。让我们将颜色改为白色，如下所示：

```html
.blog-name a {
    color: #fff;
}
```

1.  设置搜索输入样式，如下所示：

```html
.search-form input {
  height: 36px;
  background-color: #ccc;
  color: #555;
  border: 0;
  padding: 0 10px;
  border-radius: 30px;
}
```

这些样式规则设置了输入颜色、边框颜色和背景颜色。它将输入变成了如下所示的东西：

![行动时间-使用 CSS 增强标题和导航外观](img/image00269.jpeg)

1.  如前面的屏幕截图所示，占位文本几乎无法阅读，因为颜色与输入背景颜色融为一体。因此，让我们将文本颜色变得更深一些，如下所示：

```html
.search-form input::-webkit-input-placeholder {
  color: #555;
}
.search-form input:-moz-placeholder {
  color: #555;  
}
.search-form input::-moz-placeholder {
  color: #555;  
}
.search-form input:-ms-input-placeholder {  
  color: #555;
}
```

如果您使用 OS X 或 Ubuntu，您将看到突出显示当前目标时输入的发光颜色，如下面的屏幕截图所示：

![行动时间-使用 CSS 增强标题和导航外观](img/image00270.jpeg)

在 OS X 中，发光颜色是蓝色。在 Ubuntu 中，它将是橙色。

1.  我想要去掉这种发光效果。发光效果在技术上通过`box-shadow`显示。因此，为了去除这种效果，我们只需将输入的`box-shadow`设置为`none`，如下所示：

```html
.search-form input:focus {
  -webkit-box-shadow: none;
  -moz-box-shadow: none;
  box-shadow: none;
}
```

值得注意的是，发光效果是**用户体验**（**UX**）设计的一部分，告诉用户他们当前正在输入字段中。如果用户只能用键盘浏览网站，这种 UX 设计尤其有帮助。

1.  因此，我们将不得不创建一种效果，以带来类似的用户体验作为替代。在这里，我们将通过使输入背景颜色变浅来替换我们去除的发光效果。以下是此步骤的完整代码：

```html
.search-form input:focus {
  -webkit-box-shadow: none;
  -moz-box-shadow: none;
  box-shadow: none;
  background-color: #bbb;
}
```

如下面的屏幕截图所示，当焦点在输入时，输入背景颜色会变浅：

![行动时间-使用 CSS 增强标题和导航外观](img/image00271.jpeg)

1.  我们将为导航编写样式。首先，将菜单对齐到中心，并在导航的顶部和底部添加一些空白，使用`margin`。看一下以下代码：

```html
.blog-menu {
  margin: 30px 0;
  text-align: center;
}
```

1.  删除`<ul>`的左侧填充，如下所示：

```html
.blog-menu ul {
  padding-left: 0;
}
```

1.  在菜单之间添加一些空白，并删除列表符号，如下所示：

```html
.blog-menu li {
  margin: 15px;
  list-style-type: none;
}
```

1.  自定义菜单颜色和字体，如下所示：

```html
.blog-menu a {
  color: #7f8c8d;
  font-size: 18px;
   text-transform: uppercase;
   font-family: "Varela Round", Arial, sans-serif;
}
.blog-menu a:hover {
    color: #3498db;
}
```

## *刚刚发生了什么？*

我们刚刚装饰了标题和导航。与我们在本节前面讨论的以移动设备为先的思维方式相对应，我们首先将样式定位到优化博客在移动设备上的呈现。

激活 Chrome 移动模拟器，您会发现博客已经针对较小的屏幕尺寸进行了优化；标志和菜单，如下截图所示，与其左对齐相比，已对齐到中心：

![刚刚发生了什么？](img/image00272.jpeg)

## 尝试一下-自定义页眉

博客页眉给予了深色，`#333`。我完全理解这种颜色可能对你们中的一些人来说很无聊。因此，可以自由定制颜色以及标志和搜索输入字段的样式。以下是一些想法：

+   使用 CSS3 渐变或图像作为页眉背景

+   通过 CSS 图像替换方法用图像替换标志

+   减少搜索输入边框半径，更改背景颜色，并调整占位符文本颜色

在处理了博客页眉和导航之后，我们继续处理博客内容部分。内容部分包括博客文章项目和博客分页。

# 行动时间-使用 CSS 增强内容部分的外观

执行以下步骤来设置博客内容的样式：

1.  在内容部分的所有四周添加空白，使用`padding`和`margin`，如下所示

```html
.blog-content {
  padding: 15px;
  margin-bottom: 30px;
} 
```

1.  使用一些空白和边框来分隔每篇博客文章，如下所示：

```html
.post {
  margin-bottom: 60px;
  padding-bottom: 60px;
  border-bottom: 1px solid #ddd;
}
```

1.  将标题对齐到中心，稍微调整标题字体大小，并使用以下样式规则更改颜色：

```html
.post-title {
  font-size: 36px;
  text-align: center;
  margin-top: 0;
}
.post-title a {
  color: #333;
}
.post-title a:hover {
  color: #3498db;
}
```

1.  在标题下面，我们有`post-meta`，其中包括文章作者姓名和文章发布日期。与标题类似，我们还调整字体大小和空白，并更改字体颜色，如下所示：

```html
.post-meta {
  font-size: 18px;
  margin: 20px 0 0;
  text-align: center;
  color: #999;
}
.post-meta ul {
  list-style-type: none;
  padding-left: 0;
}
.post-meta li {
  margin-bottom: 10px;
}
```

1.  正如您在下面的截图中所看到的，由于所有边缘的 margin，文章缩略图看起来又小又挤：![行动时间-使用 CSS 增强内容部分的外观](img/image00273.jpeg)

1.  让我们移除这些 margin，如下所示：

```html
.post-thumbnail {
  margin: 0;
}
```

如下截图所示，一些图片有标题：

![行动时间-使用 CSS 增强内容部分的外观](img/image00274.jpeg)

1.  让我们对其进行样式设置，使其看起来与其他内容有所不同，并且显示它是一个图片标题。添加以下代码来设置标题的样式：

```html
.post-thumbnail figcaption {
  color: #bdc3c7;
  margin-top: 15px;
  font-size: 16px;
  font-style: italic;
}
```

1.  调整文章摘录的字体大小、颜色和行高，如下所示：

```html
.post-excerpt {
  color: #555;
  font-size: 18px;
  line-height: 30px;
}
```

1.  从这一步开始，我们将编写博客分页的样式。首先，让我们对字体大小、字体系列、空白、位置和对齐进行一些调整，如下所示：

```html
.blog-pagination {
  text-align: center;
  font-size: 16px;
  position: relative;
  margin: 60px 0;
}
.blog-pagination ul {
  padding-left: 0;
}
.blog-pagination li,
.blog-pagination a {
  display: block;
  width: 100%;
}
.blog-pagination li {
  font-family: "Varela Round", Arial, sans-serif;
  color: #bdc3c7;
  text-transform: uppercase;
  margin-bottom: 10px;
}
```

1.  将分页链接装饰成圆角边框，如下所示：

```html
.blog-pagination a {
  -webkit-border-radius: 30px;
  -moz-border-radius: 30px;
  border-radius: 30px;
  color: #7f8c8d;
  padding: 15px 30px;
  border: 1px solid #bdc3c7;
}
```

1.  当鼠标悬停在链接上时指定链接的装饰，如下所示：

```html
.blog-pagination a:hover {
  color: #fff;
  background-color: #7f8c8d;
  border: 1px solid #7f8c8d;
}
```

1.  最后，使用以下样式规则将页面编号指示器放置在分页链接的顶部：

```html
.blog-pagination .pageof {
  position: absolute;
  top: -30px;
}
```

## *刚刚发生了什么？*

我们刚刚对博客内容部分进行了样式设置，包括页面导航（分页），以下截图显示了内容部分的外观：

![刚刚发生了什么？](img/image00275.jpeg)

## 尝试一下-改进内容部分

我们在内容部分应用的大多数样式规则都是装饰性的。这不是您必须强制遵循的东西。请随意改进样式以符合您的个人品味。

您可以进行以下修改：

+   自定义文章标题字体系列和颜色

+   为文章图片应用边框颜色或圆角

+   更改分页边框颜色，或使背景更加丰富多彩

接下来，我们将为博客的最后一部分-页脚进行样式设置。

# 行动时间-使用 CSS 增强页脚部分的外观

执行以下步骤来增强页脚的样式：

1.  调整页脚字体、颜色和 margin，如下所示：

```html
.blog-footer {
  background-color: #ecf0f1;
  padding: 60px 0;
  font-family: "Varela Round", Arial, sans-serif;
  margin-top: 60px;
}
.blog-footer,
.blog-footer a {
  color: #7f8c8d;
}
```

1.  页脚包含社交媒体链接。让我们调整包括 margin、padding、对齐、颜色和空白的样式，如下所示：

```html
.social-media {
  margin: 0 0 30px;
}
.social-media ul {
  margin: 0;
  padding-left: 0;
}
.social-media li {
  margin: 0 8px 10px;
  list-style: none;
}
.social-media li,
.social-media a {
  font-size: 18px;
}
.social-media a:hover {
  color: #333;
}
```

1.  将 margin-top 设置为版权容器之外。

```html
.copyright {
  margin-top: 0;
}
```

1.  将页脚内容对齐到中心，如下所示：

```html
.social-media,
.copyright {
  text-align: center;
}
```

## *刚刚发生了什么？*

我们刚刚为页脚部分添加了样式规则，下面的截图显示了博客页脚的外观：

![刚刚发生了什么？](img/image00276.jpeg)

# 优化博客以适应桌面

目前博客已经针对移动端或窄视口大小进行了优化。如果你在更大的视口大小下查看，你会发现一些元素位置不正确或未正确对齐。例如，博客标志和导航目前对齐到中间，如下截图所示：

![优化博客以适应桌面](img/image00277.jpeg)

根据我们在第三章中展示的蓝图，*使用 Responsive.gs 构建简单响应式博客*，标志应该对齐到左侧，每个菜单链接应该内联显示。在接下来的步骤中，我们将通过媒体查询来修复这些问题；我们将优化博客以适应桌面视图。

# 行动时间-为桌面编写样式规则

执行以下步骤为桌面编写样式规则：

1.  在 Sublime Text 中打开`responsive.css`。

1.  添加以下媒体查询：

```html
@media screen and (min-width: 640px) {
  // add style rules here
}
```

我们将在接下来的步骤中添加所有的样式规则到这个媒体查询中。这个媒体查询规定将应用样式规则在视口宽度从 640 像素及以上的范围内。

1.  将博客标志对齐到左侧，如下所示：

```html
.blog-name {
  text-align: left;
  margin-bottom: 0;
}
```

1.  将导航菜单、文章元数据和社交媒体的列表项内联显示，如下所示：

```html
.blog-menu li,
.post-meta li,
.social-media li {
      display: inline;
}
```

1.  增加文章标题的大小，如下所示：

```html
.post-title {
  font-size: 48px;
}
```

1.  同时，将博客分页链接内联显示，如下所示：

```html
.blog-pagination li,
.blog-pagination a {
  display: inline;
}
```

1.  将分页页码指示器放在初始位置——与博客分页链接一起，如下所示：

```html
.blog-pagination .pageof {
  position: relative;
  top: 0;
  padding: 0 20px;
}
```

1.  将页脚中的社交媒体链接对齐到左侧，版权声明对齐到右侧，如下所示：

```html
.social-media {
  text-align: left;
}
.copyright {
  text-align: right;
}
```

## *刚刚发生了什么？*

我们刚刚添加了适用于桌面视图的样式规则。如果你现在在大于 640 像素的视口宽度下查看博客，你应该会发现博客中的元素，如标志和导航菜单，处于它们通常的位置，如下截图所示：

![刚刚发生了什么？](img/image00278.jpeg)

# 使用 polyfills 使 Internet Explorer 更强大

使用辉煌的 CSS3 和 HTML5 功能会带来一个后果：布局在旧的 Internet Explorer 中失败并且破碎，如下截图所示：

![使用 polyfills 使 Internet Explorer 更强大](img/image00279.jpeg)

如果你对此满意，可以跳过这一部分，立即转到下一个项目。然而，如果你感到有冒险精神，让我们继续这一部分并修复这些错误。

# 行动时间-使用 polyfills 修补 Internet Explorer

执行修补 Internet Explorer 的步骤：

1.  我们在 scripts 文件夹中有一些 polyfills，分别是`html5shiv.js`，`respond.js`和`placeholder.js`。让我们将这些脚本合并成一个文件。

1.  首先，创建一个名为`polyfills.js`的新 JavaScript 文件，用于保存这些 polyfill 脚本的内容。

1.  在 Sublime Text 中打开`polyfills.js`。

1.  添加以下行来导入 polyfill 脚本：

```html
// @koala-prepend "html5shiv.js"
// @koala-prepend "respond.js"
// @koala-prepend "placeholder.js"
```

### 注意

`@koala-prepend`指令是 Koala 专有的导入 JavaScript 文件的指令。在 Koala 文档页面[`github.com/oklai/koala/wiki/JS-CSS-minify-and-combine`](https://github.com/oklai/koala/wiki/JS-CSS-minify-and-combine)中了解更多信息。

1.  在 Koala 中，选择`polyfills.js`，并点击**Compile**按钮，如下截图所示：![行动时间-使用 polyfills 修补 Internet Explorer](img/image00280.jpeg)

通过这一步，Koala 将生成名为`polyfills.min.js`的压缩文件。

1.  在`index.html`中，在`</head>`之前链接`polyfills.js`，如下所示：

```html
<!--[if lt IE 9]>
<script type="text/javascript" src="img/polyfills.min.js"></script>
<![endif]-->
```

### 注意

由于这个脚本只在 Internet Explorer 8 及以下版本中需要，我们用 Internet Explorer 条件注释`<!--[if lt IE 9]>`将它们封装起来，如你在前面的代码片段中所见。

有关 Internet Explorer 条件注释的更多信息，请参考 QuirksMode 文章[`www.quirksmode.org/css/condcom.html`](http://www.quirksmode.org/css/condcom.html)。

## *发生了什么？*

我们刚刚在博客中应用了 polyfills 来修复 Internet Explorer 在 HTML5 和媒体查询中的渲染问题。这些 polyfills 可以立即使用。刷新 Internet Explorer，就完成了！请看下面的屏幕截图：

发生了什么？

样式规则已经应用，布局已经就位，占位文本也在那里。

## 来吧英雄-为 Internet Explorer 完善博客

我们将结束这个项目。但是，正如您从前面的屏幕截图中所看到的，仍然有许多问题需要解决，以使博客在旧版 Internet Explorer 中的外观与最新浏览器一样好。例如：

+   参考前面的屏幕截图，占位文本目前是对齐到顶部的。您可以修复它，使其垂直居中对齐。

+   您还可以应用一个名为 CSS3Pie 的 polyfill（[`css3pie.com/`](http://css3pie.com/)），它可以在 Internet Explorer 中实现 CSS3 边框半径，使搜索输入字段的外观与最新的浏览器版本一样圆角。

# 总结

我们完成了第一个项目；我们使用 Responsive.gs 构建了一个简单的响应式博客。博客的最终结果可能对您来说并不那么吸引人。特别是在旧版 Internet Explorer 中，它也远非完美；正如前面提到的，仍然有许多问题需要解决。不过，我希望您能从这个过程中获得一些有用的东西，包括其中的技术和代码。

总之，在本章中，我们已经增强和完善了博客的 CSS3，使用 Koala 来合并和最小化样式表和 JavaScript 文件，并应用了 polyfills 来修复 Internet Explorer 在 HTML5 和 CSS3 中的问题。

在下一章中，我们将开始第二个项目。我们将探索另一个框架，以构建一个更广泛和响应式的网站。
