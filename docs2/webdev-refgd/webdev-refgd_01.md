# 第一章：HTML 元素

**超文本标记语言**（**HTML**）是一种注释文本的语言。文本的注释是通过元素来完成的。以下面的 `p` 元素为例，我们将看到如何使用 HTML：

```js
<p>This is an example</p>
```

HTML 元素也有属性，这些属性将修改它们的渲染或解释方式。属性添加在起始标签内。下面是 `div` 元素中的 `class` 属性：

```js
<div class="example">This is an example</div>
```

到目前为止，HTML 已经有多个规范，但我们将只关注 HTML5 中最常用和最重要的元素。HTML5 是最新的官方规范，因此在撰写本书时，我们将尽可能保证其前瞻性。您将希望尽可能紧密地遵循 HTML 的规范。大多数浏览器都非常宽容，会渲染您的 HTML，但当你超出规范时，你可能会遇到奇怪的渲染问题。

### 注意

所有 HTML 元素都将具有全局属性。以下各节中列出的每个元素的属性都是超出全局属性的属性。

# DOCTYPE

`DOCTYPE` 元素定义了文件的文档类型，如下所示：

```js
<!DOCTYPE documentType>
```

## 属性

前面代码中可以看到的 `documentType` 属性让浏览器知道您将使用的文档类型。

## 描述

HTML5 有一个简单的文档类型声明，`<!DOCTYPE html>`。这会让浏览器知道文档是 HTML5。HTML 的先前版本需要一个正式的版本定义，但 HTML5 为了简化已经去除了这一点。

大多数浏览器将强制执行对声明的文档类型的严格遵循，并试图根据查看文档来推断它是什么。这可能会导致渲染问题，因此建议您遵循标准。

下面是一个 HTML5 声明：

```js
<!DOCTYPE html>
```

# html

`html` 元素是 HTML 文档的根元素：

```js
<html manifest></html>
```

## 属性

`manifest` 属性链接到一个资源清单，列出了哪些文件应该被缓存。

## 描述

`html` 元素必须直接跟在 `DOCTYPE` 元素之后。这是所有其他元素都必须是其后代的根元素。

`html` 元素必须有一个 `head` 元素和一个 `body` 元素作为其子元素。所有其他元素都将包含在这些标签内。

下面是一个简单的 HTML 页面的样子：

```js
<!DOCTYPE html>
<html manifest="offline.appcache"><head>
</head>
<body>
    Hey
</body>
</html>
```

# 文档元数据

下一个元素将提供有关文档的元数据。除了这些，您还可以包括对资源（如 CSS 和 JavaScript）的链接。

## head

`head` 元素是元数据父元素。所有其他元数据元素都将是这个元素的子元素：

```js
<head></head>
```

### 描述

`head` 元素通常必须包含一个 `title` 元素。可以放入 `head` 的元素有 `title`、`link`、`meta`、`style`、`script`、`noscript` 和 `base`。这些元素将在下面进行解释。

下面是一个定义了 `title` 和 `stylesheet` 元素的 `head` 元素的示例：

```js
<head>
    <title>Title that appears as the tab name</title>
    <link href="style.css" rel="stylesheet" type="text/css" media="all" />
</head>
```

### title

`title`元素显示文档的标题。这是在浏览器标签或浏览器窗口中显示的内容：

```js
<title></title>
```

#### 描述

`title`元素是一个恰如其分的元素。这是`head`中的必需元素，并且对于每个文档，应该只有一个`title`元素。以下是一个简单的`title`元素示例：

```js
<head>
    <title>Example</title>
</head>
```

### link

`link`元素将资源链接到当前文档：

```js
<link crossorigin href media rel sizes type></link>
```

#### 属性

在`link`元素中使用的属性如下：

+   `crossorigin`: 这告诉浏览器是否要发起**跨源资源共享**（**CORS**）请求

+   `href`: 这表示资源的 URL

+   `media`: 这选择此资源适用的媒体

+   `rel`: 这表示此资源与文档的关系

+   `sizes`: 当`rel`设置为`icons`时，此属性被使用

+   `type`: 这表示资源的内容类型

#### 描述

`link`元素有很多属性，但大多数情况下，它用于加载 CSS 资源。这意味着您至少需要使用`href`、`rel`、`type`和`media`属性。

您可以在`head`元素中拥有多个`link`元素。以下是一个加载 CSS 的简单示例：

```js
<link href="style.css" rel="stylesheet" type="text/css" media="all" />
<link href="style.css" media="screen" rel="styleshhet" sizes type="text/css"></link>
```

#### 参见

您还可以通过引用`crossorigin`、`href`、`media`、`rel`、`sizes`和`type`属性来获取更多关于`title`元素的信息。

### meta

`meta`元素包含关于文档的元数据。其语法如下：

```js
<meta content http-equiv name></meta>
```

#### 属性

在`meta`元素中使用的属性如下：

+   `content`: 这声明了`name`或`http-equiv`属性的价值。

+   `http-equiv`: 在 HTML5 中，此属性可以设置为`default-style`，以设置默认样式。或者，它可以设置为`refresh`，可以指定刷新页面的秒数，如果需要，还可以指定新页面的不同 URL，例如，`http-equiv="1;url=http://www.google.com"`。

+   `name`: 这声明了元数据的名称。

#### 描述

`meta`标签有许多非标准应用。标准化应用可以在第二章 *HTML 属性*中查看。

### 注意

苹果有许多`meta`标签，这些标签会将信息传递给 iOS 设备。例如，您可以设置对 App Store 应用的引用，设置图标，或以全屏模式显示页面，这只是其中的一些例子。所有这些标签都是非标准的，但在针对 iOS 设备时很有用。许多其他网站和公司也是如此。

您可以在`head`元素中使用多个`meta`标签。以下有两个示例。第一个示例将在每 5 秒刷新页面，而另一个示例将定义`author`元数据：

```js
<meta http-equiv="refresh" content="5" />
<meta name="author" content="Joshua" />
```

#### 参见

您还可以通过引用`name`属性来获取更多关于`meta`元素的信息。

### style

`style`元素包含样式信息。

**CSS**:

```js
<style media scoped type></style>
```

#### 属性

在`style`元素中使用的属性如下：

+   `media`: 这是一个媒体查询

+   `scoped`：此元素包含的样式仅应用于父元素

+   `type`：这设置了文档的类型；默认值是 `text`/`css`

#### 描述

`style` 元素通常位于 `head` 元素中。这允许你为当前文档定义 CSS 样式。

使用 CSS 的首选方法是创建一个单独的资源并使用 `link` 元素。这允许样式一次定义并在整个网站上使用，而不是在每一页上定义它们。这是一种最佳实践，因为它允许我们在一个地方定义样式。然后我们可以轻松地找到并更改样式。

这里是一个简单的内联样式表，将字体颜色设置为蓝色：

```js
<style media="screen" scoped type="text/css">
    body{
        color: #0000FF;
    }
</style>
```

#### 相关内容

你还可以参考全局属性和第 3-7 章，以了解更多有关 `style` 元素的信息。

### base

`base` 元素是文档的基本 URL。其语法如下：

```js
<base href target>
```

#### 属性

在 `base` 元素中使用的属性如下：

+   `href`：这表示要使用的基 URL

+   `target`：这表示与 URL 一起使用的默认目标

#### 描述

基 URL 在页面上使用相对路径或 URL 时使用。如果没有设置，浏览器将使用当前 URL 作为基 URL。

这里是如何设置基 URL 的一个例子：

```js
<base href="http://www.packtpub.com/">
```

#### 相关内容

你还可以参考 `target` 属性以获取有关 `base` 元素的更多详细信息。

### script

`script` 元素允许你引用或为文档创建脚本：

```js
<script async crossorigin defer src type><script>
```

#### 属性

在 `script` 元素中使用的属性如下：

+   `async`：这是一个布尔属性，告诉浏览器异步处理此脚本。这仅适用于引用的脚本。

+   `crossorigin`：这告诉浏览器是否进行 CORS 请求。

+   `defer`：这是一个布尔属性，告诉浏览器在文档解析后执行脚本。

+   `src`：这区分了脚本的 URL。

+   `type`：这定义了脚本的类型，如果省略了属性，则默认为 JavaScript。

#### 描述

`script` 元素是将 JavaScript 引入你的文档并添加增强交互性的方式。这可以通过添加 JavaScript 到元素中的裸 `script` 标签来完成。此外，你可以使用 `src` 属性来引用外部脚本。引用脚本文件被认为是一种最佳实践，因为它可以在此处重用。

此元素可以是 `head` 的子元素，也可以放置在文档的任何位置。根据 `script` 元素的位置，你可能或可能无法访问 DOM。

这里有两个使用 `script` 元素的例子。第一个例子将引用外部脚本，第二个将是一个内联 `script` 元素，最后一个将展示如何使用 `crossorigin` 属性：

```js
<script src="img/example.js" type="text/javascript"></script>
<script>
    var a = 123;
</script>
<script async crossorigin="anonymous" defer src="img/application.js" type="text/javascript"><script>
```

### noscript

如果在浏览器中关闭了脚本，则将解析 `noscript` 元素。其语法如下：

```js
<noscript></noscript>
```

#### 描述

如果启用了脚本，则此元素内的内容将不会显示在页面上，并且其中的代码将运行。如果禁用了脚本，它将被解析。

此元素主要用于让用户知道网站可能无法与 JavaScript 一起工作。今天几乎每个网站不仅使用 JavaScript，而且需要它。

这里是 `noscript` 元素的例子：

```js
<noscript>
    Please enable JavaScript.
</noscript>
```

# 语义内容部分

下一个元素是在向文档添加内容时使用的主要元素。例如，使用 `article` 元素而不是任意的 `div` 元素允许浏览器推断这是页面的主要内容。应使用这些元素来为文档提供结构，而不是用于样式目的。语义元素使我们的 HTML 文档能够通过不断增加的不同设备实现更高的可访问性。

## body

`body` 元素是文档的主要内容部分。必须只有一个主要元素，其语法如下：

```js
<body></body>
```

### 属性

`body` 元素的属性包括内联事件属性。

### 描述

`body` 元素是大多数文档的主要内容部分。它必须是 `html` 的第二个子元素，并且文档中只能有一个 `body` 元素。

这里是 `body` 元素的例子：

```js
<body>
    <span>Example Body</span>
</body>
```

## section

`section` 元素描述了文档的内容部分。例如，这可以是书的章节：

```js
<section></section>
```

### 描述

`section` 元素是 HTML5 中引入的一个新元素。`section` 元素应该将内容分组在一起。虽然不是必需的，但将 `heading` 元素作为代码中的第一个元素是一种最佳实践。应将部分视为文档大纲的另一个部分。它将相关项目组合成一个易于定位的区域。您可以在文档中多次使用此元素。

这里是 `section` 元素的例子：

```js
<section>
    <h2>Section Heading</h2>
    <p>Section content.</p>
</section>
```

## nav

`nav` 元素是导航元素：

```js
<nav></nav>
```

### 描述

`nav` 元素是 HTML5 中引入的另一个语义元素。这使浏览器知道此元素的内容是父元素，用于导航。`nav` 元素通过为屏幕阅读器提供导航地标来增强可访问性。此元素应包含任何用于简化导航的网站导航或其他链接。您可以使用此元素多次。

这里是一个使用 `nav` 元素的例子：

```js
<nav>
    <ul>
        <li><a href="#">Home</a></li>
    </ul>
</nav>
```

### article

`article` 元素的设计是为了包裹可以独立存在的内联内容：

```js
<article></article>
```

### 描述

`article` 元素是 HTML5 中引入的新元素。`article` 元素类似于 `section` 元素；在这一点上，它表示元素中的内容是页面的核心部分。`article` 元素应该是一个完整的作品，可以独立存在。例如，在博客中，实际的博客文章应该用 `article` 元素包裹。

可以使用`article`元素或`section`元素进一步拆分内容。没有使用哪个元素的标准规则。然而，两者都应该与外部的`article`元素中的内容相关。

这里是使用`article`元素的示例：

```js
<article>
    <header>
        <h1>Blog Post</h1>
    </header>
    <p>This post covers how to use an article element...</p>
    <footer>
        <address>
            Contact the author, Joshua
        </address>
    <footer>
</article>
```

## 标题

`heading`元素是用于指定不同标题级别的元素，根据其重要性来划分：

```js
<h1></h1>
<h2></h2>
<h3></h3>
<h4></h4>
<h5></h5>
<h6></h6>
```

### 描述

这些应该用于给不同的标题赋予相对的重要性。例如，`h1`应用于文档的标题。标题的重要性随着标题值的增加而降低，即，在下面的示例中`h6`是标题级别最低的。

这里是使用所有标题的示例：

```js
<h1>Heading Importance 1</h1>
<h2>Heading Importance 2</h2>
<h3>Heading Importance 3</h3>
<h4>Heading Importance 4</h4>
<h5>Heading Importance 5</h5>
<h6>Heading Importance 6</h6>
```

### 参见

你也可以参考全局属性来详细了解`heading`元素。

## header

`header`元素将一组被认为是特定内容组标题的内容组合在一起，其语法如下：

```js
<header></header>
```

### 描述

`header`元素通常是页面上第一个内容元素之一。它很可能包含导航选项、标志和/或搜索框。页眉通常在网站的多个页面上重复。每个部分或文章也可以包含页眉。这是 HTML5 中引入的新元素。

这里是`header`元素的示例：

```js
<header>
    <img src="img/logo.png" />
</header>
```

### 参见

你也可以参考全局属性来详细了解`header`元素。

## footer

`footer`元素为特定内容组提供页脚，其语法如下：

```js
<footer></footer>
```

### 描述

`footer`元素包含被认为是文档页脚的所有内容。通常，这包括版权、作者和/或社交媒体链接。当然，你决定放入页脚的内容是任意的。每个部分或文章也可以包含页脚。

这里是`footer`元素的示例：

```js
<footer>
    Written by: Joshua Johanan
</footer>
```

## 地址

`address`元素用于作者的联系方式或组织的联系地址，其语法如下：

```js
<address></address>
```

### 描述

当你有用户的联系信息时，请使用`address`元素。`address`元素将为内容添加语义值，与常规的`div`元素相反。

通常，这将被放置在页脚中，但也可以用于文章或主体部分。它将应用于最近的`article`元素或整个文档。不要在`address`元素中使用任何内容部分元素。

这里是`address`元素的使用示例：

```js
<footer>
    <address>
        Please contact me at my <a href="#">website</a>
    </address>
</footer>
```

## aside

`aside`元素用于补充内容：

```js
<aside></aside>
```

### 描述

使用`aside`元素来突出与主要文章相关的内容。在博客的上下文中，一些示例包括作者的简介、作者的其他帖子或甚至相关的广告。

这里是一个`aside`元素的示例：

```js
<aside>
    Peyton Manning is a 5-time MVP (2003, 2004, 2008, 2009, 2013)

</aside>
```

## p

`p`元素被称为段落元素。这是一个块级元素，其语法如下：

```js
<p></p>
```

### 描述

应该使用 `p` 元素来区分文档中的单独段落。此元素与 `text` 和 `inline` 元素相关联。你不会想使用 `div` 元素，例如。如果你发现自己想要这样做，你可能需要以不同的方式构建你的文档。

这里是几个段落：

```js
<p>This is an intro paragraph.</p>
<p>This paragraph will build upon the opening.</p>
```

# 内容部分

`content` 部分与语义内容部分非常相似。主要区别在于，所有给定元素的使用不是由文档的轮廓或目的驱动的，就像语义部分那样。

## 水平线

`hr` 元素是水平线元素，其语法如下：

```js
<hr>
```

### 描述

默认情况下，`hr` 元素将在内容中绘制一条水平线。你可以通过 CSS 改变此元素的外观。

此元素内部不应包含任何内容：

```js
<p>This is a paragraph.</p>
<hr/>
<p>This paragraph goes in another direction.</p>
```

## pre

`pre` 元素是预格式化文本：

```js
<pre></pre>
```

### 描述

HTML 文档中的文本通常在浏览器中不会以与文本文档相同的空白或换行符显示。`pre` 元素允许你以与文档中相同的方式显示文本。空白和换行符将被保留。

这里是使用换行符的示例。

```js
<pre>This text
has some
line breaks.</pre>
```

## blockquote

`blockquote` 元素的语法如下：

```js
<blockquote cite></blockquote>
```

### 属性

在 `blockquote` 元素中使用 `cite` 属性来指向被引用文档的 URL。

### 描述

当从文档或文本中提取引用时，使用 `blockquote` 元素。

这里有一个示例：

```js
<blockquote cite="https://www.packtpub.com/">
    <p>Contact Us</p>
</blockquote>
```

## 有序列表

`ol` 元素是有序列表元素，其语法如下：

```js
<ol reversed start type></ol>
```

### 属性

在 `ol` 元素中使用的属性如下：

+   `reversed`：这是一个布尔值。表示列表是倒序的。

+   `start`：这个属性接受一个数值作为起始值。

+   `type`：这将改变编号元素的类型。默认情况下，我们可以有一个编号列表（`1`），但我们可以更改为其他类型，例如小写字母（`a`）、大写字母（`A`）、小写罗马数字（`i`）和大写罗马数字（`I`）。

### 描述

`ol` 元素可以在与 `ul` 元素相同的情况下使用，除了 `ol` 元素是编号的而不是带点的。例如，你会使用 `ul` 元素来创建购物清单，而使用 `ol` 元素来创建编号指令集。你可以在每个元素内部嵌套多个 `ul` 或 `ol` 元素。

列表中的项将是 `li` 元素。

这里是一个使用罗马数字并从 `10` 开始的列表示例。

```js
<ol start="10" type="i">
    <li>Roman numeral 10</li>
    <li>Roman numeral 11</li>
</ol>
```

### 相关内容

你也可以参考 `ul` 和 `li` 元素来了解更多关于 `ol` 元素的信息。

## 无序列表

`ul` 元素是无序列表元素：

```js
<ul></ul>
```

### 描述

`ul` 元素可以在与 `ol` 元素相同的情况下使用，但 `ul` 元素将带点，而 `ol` 元素将编号。

当你为这个列表设置样式时，你应该使用 CSS 而不是旧的 HTML 4 属性。

你可以多次嵌套 `ul` 和 `ol` 元素。

这里是 `ul` 元素的示例：

```js
<ul>
    <li>Items in</li>
    <li>no particular</li>
    <li>order</li>
</ul>
```

### 参考信息

您还可以参考 `ol` 和 `li` 元素，以了解更多关于 `ul` 元素的信息。

## li

`li` 元素是列表项元素：

```js
<li value></li>
```

### 属性

`value` 属性在 `li` 元素和 `ol` 元素中使用，它是有序列表中项目的值。

### 描述

您将使用 `li` 元素为列表中的每个项目。

这里有一个例子：

```js
<ul>
    <li>Item 1</li>
    <li>Item 2</li>
    <li>Item 3</li>
</ul>
```

### 参考信息

您还可以参考 `ol` 和 `ul` 元素，以了解更多关于 `li` 元素的信息。

## dl

`dl` 元素是定义列表元素：

```js
<dl></dl>
```

### 描述

`dl` 元素是一个列表，其中项目有术语和定义；然而，`dl` 元素不仅可以用于术语和定义。

在构建 `dl` 元素的列表时，您必须使用 `dt` 元素后跟 `dd` 元素。每个 `dt` 元素后面可以有多个 `dd` 元素。

这里有一个例子：

```js
<dl>
    <dt>PactPub</dt>
    <dd>Packt Publishing</dd>
</dl>
```

### 参考信息

您还可以参考 `dt` 和 `dd` 元素，以了解更多关于 `dl` 元素的信息。

## dt

`dt` 元素是定义术语元素：

```js
<dt></dt>
```

### 描述

`dt` 元素是定义列表中的第一个项目，另一个项目是 `dd` 元素。

这里有一个示例：

```js
<dl>
    <dt>PactPub</dt>
    <dd>Packt Publishing</dd>
</dl>
```

### 参考信息

您还可以参考 `dl` 和 `dd` 元素，以了解更多关于 `dt` 元素的信息。

## dd

`dd` 元素是定义描述元素：

```js
<dd></dd>
```

### 描述

`dd` 元素是定义列表中的第二个项目，另一个是 `dt` 元素。

这里有一个示例：

```js
<dl>
    <dt>PactPub</dt>
    <dd>Packt Publishing</dd>
</dl>
```

### 参考信息

您还可以参考 `dl` 和 `dd` 元素，以了解更多关于 `dd` 元素的信息。

## figure

`figure` 元素的语法如下：

```js
<figure></figure>
```

### 描述

`figure` 元素是 HTML5 中引入的新元素。与文章元素在无意义的地方添加一些意义的方式类似，图元素也添加了意义。图是一个图像或其他与文档相关的信息项。这比仅仅使用 `img` 元素更有意义。

这里有一个例子：

```js
<figure>
     <img src="img/figure1.jpg" title="Figure 1" />
     <figcaption>Figure One</figcaption>
</figure>
```

### 参考信息

您还可以参考 `figcaption` 元素，以了解更多关于 `figure` 元素的信息。

## figcaption

`figcaption` 元素是图注元素：

```js
<figcaption></figcaption>
```

### 描述

`figcaption` 元素是在 HTML5 中与 `figure` 元素一起引入的。此元素为图提供标题。此元素必须位于 `figure` 元素内部，并且必须是 `figure` 元素的第一或最后一个子元素。

这里是 `figcaption` 元素的简单示例：

```js
<figure>
     <img src="img/figure1.jpg" title="Figure 1" />
     <figcaption>Figure One</figcaption>
</figure>
```

### 参考信息

您还可以参考 `figure` 元素，以了解更多关于 `figcaption` 元素的信息。

## div

`div` 元素是分割元素：

```js
<div></div>
```

### 描述

`div` 元素是今天 HTML 中最常用的元素之一。它是用来将您的文档分割成任意部分使用的元素。`div` 元素没有默认样式。这些分割可以是用于定位、样式或其他任何原因。`div` 元素不会影响文档的语义意义。它应该只在没有其他元素满足您的要求时使用。

这里是一个示例：

```js
<div>
    You can put whatever you want in here!
    <div>
        More elements.
    </div>
</div>
```

### main

`main` 元素的语法如下：

```js
<main></main>
```

### 描述

`main` 元素应该包含文档的主要内容。您不能将此元素作为 `article`、`aside`、`footer`、`header` 或 `nav` 元素的子元素。这与文章不同，因为文章应该是一个自包含的元素。

这里是 `main` 元素的使用示例：

```js
<main>
    This is the main content of the document.
    <article>
        Here is the article of the document.
    </article>
</main>
```

# 内联元素

以下元素都可以包裹文本和块级元素，以赋予它们功能、样式和意义。

## a

`a` 元素是锚元素。这是 HTML 获得**超文本**（**HT**）的地方，语法如下：

```js
<a download href media ping rel target type></a>
```

### 属性

这里列出了在 `a` 元素中使用的属性：

+   `download`: 此属性让浏览器知道该项应该被下载。对话框将默认为此属性中的文件名。

+   `href`: 这是链接目标。

+   `media`: 这根据媒体查询指定了样式表应该应用到的媒体。

+   `ping`: 这会创建一个用于 ping 和通知的 URL，如果链接被跟随。

+   `rel`: 这指定了被链接文档的关系。

+   `target`: 这指定了目标链接应该显示的位置。

+   `type`: 这指定了链接资源的 MIME 类型。

### 描述

`a` 元素是最重要和最有用的元素之一。它允许您将文档链接在一起，并且您可以轻松地在元素之间跳转。我们可以这样说，如果没有这个非常易于使用的元素，互联网可能不会像现在这样受欢迎。

链接可以是文档中的 `anchor` 标签、相对 URL 或任何外部资源。当链接到当前文档中的 `anchor` 标签时，请使用 `a` 标签和 `id` 属性。

您在 `a` 元素内部放置的内容将成为用户可以点击以跟随链接的部分。这包括 `text`、`img` 和 `div` 元素等。

这里是一个带有图像的 `img` 元素的示例：

```js
<a href="http://www.packtpub.com">
    <img src="img/packt_logo.png" />
</a>
```

这里是一个将要下载的 PDF 文档的示例；这将跟踪每个点击：

```js
<a download="report.pdf" href="assests/report.pdf" media="min-width: 1024px" ping="track/click" rel="alternate" target="_blank" type=" application/pdf"></a>
```

## abbr

`abbr` 元素是缩写元素：

```js
<abbr></abbr>
```

### 描述

`abbr` 元素用于显示缩写代表的内容。您应该在 `title` 属性中放置完整的单词。大多数浏览器在您将鼠标悬停在此元素上时将显示此内容作为工具提示。

这里是一个示例：

```js
<abbr title="abbreviation">abbr</abbr>
```

## bdo

`bdo` 元素是双向覆盖元素：

```js
<bdo dir></bdo>
```

### 属性

`dir` 属性用于 `bdo` 元素中，它给出了文本的方向。其值可以是 `ltr`、`rtl` 和 `auto`。

### 描述

`bdo` 元素将覆盖元素中定义的方向的当前文本方向。

这里是一个示例：

```js
<bdo dir="rtl">Right to Left.</bdo>
```

## br

`br` 元素是换行元素：

```js
<br>
```

### 描述

`br` 元素添加一个换行。由于在浏览器中渲染文本时忽略行断，因此需要此元素。不应使用它来帮助放置元素，因为这应该是 CSS 的工作。

这里是一个示例：

```js
First Name<br>
LastName
```

## cite

`cite` 元素是引用元素：

```js
<cite></cite>
```

### 描述

`cite`元素用于引用另一个来源。大多数浏览器将以斜体显示。

这里是一个例子：

```js
This quote is from <cite>Web Developer's Reference</cite>
```

## code

`code`元素的语法如下：

```js
<code></code>
```

### 描述

`code`元素用于在文档中显示编程代码。浏览器将使用等宽字体显示它。

这里是一个例子：

```js
Here is some JavaScript: <code>var a = 'test'</code>
```

## dfn

`dfn`元素是定义实例元素：

```js
<dfn></dfn>
```

### 描述

`dfn`元素用于创建定义实例或首次引入并解释特定单词。

这里是`dfn`元素的例子：

```js
<dfn>HTML</dfn>, or HyperText Markup Language.
```

## em

`em`元素是强调元素：

```js
<em></em>
```

### 描述

`em`元素用于给特定的单词或短语添加更多强调。默认情况下，浏览器将以斜体字体显示，但不应该仅仅用于斜体。

## kbd

`kbd`元素是键盘输入元素：

```js
<kbd></kbd>
```

### 描述

`kbd`元素用于用户应输入的文本。这并不意味着用户将数据输入到元素中，而是他们将在窗口、控制台或他们电脑上的某些应用程序中输入它。

这里是一个例子：

```js
Press <kbd>Win + R</kbd> to open the Run dialog.
```

## mark

`mark`元素的语法如下：

```js
<mark></mark>
```

### 描述

`mark`元素用于突出显示文本。

这里是一个例子：

```js
<mark>This</mark> will be highlighted
```

## q

`q`元素是引用元素：

```js
<q cite></q>
```

### 属性

`q`元素中使用的`cite`属性声明了引用的来源 URL。

### 描述

`q`元素用于短引用。对于较长的引用，请使用`blockquote`。

这里是一个例子：

```js
<q cite="http://en.wikiquote.org/">Don't quote me on this.</q>
```

### 相关内容

您还可以参考`blockquote`属性来了解更多关于`q`元素的信息。

## s

`s`元素是删除线元素：

```js
<s></s>
```

### 描述

当文档中的某条信息不再准确时，应使用`s`元素。这与对文档进行的修订不同。

这里是一个例子：

```js
Today is the <s>twenty-fifth<s> twenty-sixth.
```

## samp

`samp`元素是示例输出元素：

```js
<samp></samp>
```

### 描述

`samp`元素用于显示命令或程序的示例输出。

这里是一个例子：

```js
The command should output <samp>Done!</samp>
```

## small

`small`元素的语法如下：

```js
<small></small>
```

### 描述

`small`元素用于使文本变小。这通常用于版权或法律文本等文本。

这里是一个例子：

```js
<small>Copyright 2014</small>
```

## span

`span`元素的语法如下：

```js
<span></span>
```

### 描述

`span`元素与`div`元素类似；即，它只是一个任意的容器。`div`元素是一个块级元素，而`span`元素是一个内联元素。该元素不会向文本或文档添加任何语义意义。通常，它用于向文本添加 CSS 样式：

```js
<span>This text is in the span element.</span>
```

## strong

`strong`元素的语法如下：

```js
<strong></strong>
```

### 描述

当某些文本需要更多重要性时，应使用`strong`元素。这具有一些语义意义。在大多数浏览器中，`strong`元素的默认样式是粗体。因此，它不应与`b`元素互换，因为`b`元素不携带任何语义意义。

这里是一个例子：

```js
<strong>Warning!</strong> JavaScript must be enabled.
```

## sub

`sub`元素是下标元素：

```js
<sub></sub>
```

### 描述

`sub`元素会将文本渲染为下标。

这里有一个例子：

```js
H<sub>2</sub>O
```

## sup

`sup`元素是上标元素：

```js
<sup></sup>
```

### 描述

`sup`元素会将文本渲染为上标。

这里有一个例子：

```js
x<sup>2</sup> is what x squared should look like
```

## time

`time`元素的语法如下：

```js
<time datetime></time>
```

### 属性

在`time`元素中使用的`datetime`属性提供了一个日期和时间值的字符串。

### 描述

`datetime`元素允许浏览器轻松解析文档中的日期。你可以包裹一个日期或日期的描述（例如明天或 7 月 4 日），浏览器仍然可以读取确切的日期。

这里有一个例子：

```js
The party is on <time datetime="2014-11-27 14:00">Thanksgiving @ 2PM</time>
```

## var

`var`元素是变量元素：

```js
<var></var>
```

### 描述

`var`元素用于数学表达式中的变量或编程中的变量。

这里有一个例子：

```js
The variable <var>x</var> is equal to the string test in this example.
```

## wbr

`wbr`元素是单词断行机会元素：

```js
<wbr>
```

### 描述

`wbr`元素是 HTML5 中引入的新元素。我们使用这个元素来让浏览器知道在单词之间一个好的断点。这不会强制断行，但如果需要断行，浏览器将尊重这个元素。

它是一个空标签，这意味着它不应该有结束标签。

这里有一个例子：

```js
If you have a really short width <wbr>then you <wbr>could have breaks.
```

# 嵌入内容

以下元素用于将媒体或其他对象嵌入到文档中。

## img

`img`元素是图片元素：

```js
<img alt crossorigin height ismap sizes src srcset width />
```

### 属性

在`img`元素中使用的属性如下：

+   `alt`：这是图片的替代文本。用于描述图片。这用于提高可访问性。

+   `crossorigin`：这会让浏览器知道是否应该使用 CORS 请求来获取此图片。如果图片将在画布元素中修改，而不是来自同一域名，则必须使用 CORS 请求。

+   `height`：这是一个设置图片高度的属性。

+   `ismap`：这会让浏览器知道图片是否用于服务器端映射。

+   `sizes`：这是一个将映射到大小的媒体条件列表。这用于帮助浏览器确定使用哪个图片。默认情况下，这将设置为 100 VW，即视口宽度的 100%。

+   `src`：这是最重要的属性，它是图片的 URL。

+   `srcset`：这是一个可以用于显示在我们网页上的多个图片列表。这用于针对不同的屏幕尺寸或像素密度。

+   `width`：这是一个设置图片宽度的属性。

### 描述

当你希望在文档中插入图片时，会使用`img`元素。这个元素有许多属性，但`src`和`alt`属性是唯一必需的属性。在几乎所有情况下，都应该使用`alt`属性来描述图片。主要的例外是当图片仅用作装饰性图片时，例如，当图片被用作水平线代替时。如果图片的大小与页面上所需的大小不同，可以使用宽度和高度；否则，它将默认为图片的大小。

`crossorigin`元素可能会令人困惑。它用于确保在你在画布元素中修改图像之前拥有该图像的所有权。图像必须来自相同的完全限定域名，或者服务器的响应必须让浏览器知道当前域名是否可以使用该图像。

最后，`srcset`用于向浏览器提供一个它可以使用图像的列表。这是通过逗号分隔的 URL 列表和一个描述符来完成的。描述符可以是宽度描述符，它是一个数字后跟`w`，或者像素描述符，它是一个数字后跟`x`。宽度描述符告诉浏览器图像的宽度。像素描述符告诉浏览器应该为图像使用的像素密度。

### 注意

当像素密度变化时，浏览器也可以使用宽度描述符。例如，如果你有一个分辨率是双倍的图像，像素密度也加倍，浏览器将选择更高的分辨率。

`sizes`元素与`srcset`一起使用，以帮助浏览器识别一个断点。这是通过使用媒体条件来完成的，例如，`"(min-width: 1600px) 25vw, 100vw"`。这表示如果页面宽度至少为`1600`像素，则图像将是视口宽度的 25%，否则视口宽度是 100%。这有助于浏览器知道你想要在何处设置断点以及你想要图像有多大。

### 注意

关于`srcset`的最佳思考方式是，你正在让浏览器知道可以在特定的`img`元素中使用的所有图像。你包含浏览器最关心的信息——宽度和像素密度——并让浏览器选择。

这里有一些示例。第一个示例是一个简单的`img`标签，下一个示例使用了`crossorigin`：

```js
<img src="img/example.png" alt="This is an example image"/>
<img src="img/example.png" crossorigin="anonymous" alt="This is an example image"/>
```

这里是一个`srcset`示例，它允许浏览器根据像素密度选择图像：

```js
<img src="img/normal.jpg" srcset="normal.jpg 1x, retina.jpg 2x" alt="Image of Person in article" />
```

以下是一个使用`srcset`和宽度的示例：

```js
<img src="img/regular.jpg" srcset="double_size.jpg 1000w, regular.jpg 500w" sizes="100vw" alt="A bird"/>
```

### iframe

`iframe`元素是内联框架元素：

```js
<iframe height name seamless src width></iframe>
```

### 属性

在`iframe`元素中使用的属性如下：

+   `height`：这是一个设置高度的属性。

+   `name`：这表示一个可以在目标属性中使用的名称。

+   `seamless`：这使得`iframe`看起来像是文档内容的一部分。这将应用外部上下文 CSS，并允许我们在外部上下文中打开链接。

+   `src`：这是嵌入文档的 URL。

+   `width`：这是一个设置宽度的属性。

### 描述

`iframe`元素用于在当前文档中嵌入另一个完整的 HTML 文档。

这里有一个加载 Google 主页的示例，另一个加载 Packt Publishing 页面的示例：

```js
<iframe src="img/www.google.com"></iframe>
<iframe height="100px" name="remote-document" seamless src="img/" width="100px"></iframe>
```

## embed

`embed`元素的语法如下：

```js
<embed height src type width/>
```

### 属性

在`embed`元素中使用的属性如下：

+   `height`：这是一个设置高度的属性

+   `src`：这是要嵌入的对象的 URL

+   `type`：这是对象的 MIME 类型。

+   `width`：这是一个设置宽度的属性

### 描述

`embed`元素用于在文档中嵌入其他对象。根据对象类型，还有其他用于嵌入对象的元素。例如，您可以使用视频元素嵌入视频，如下所示：

```js
<embed src="img/example.mp4" type="video/mp4"/>
```

### 参见

您还可以参考`audio`、`video`和`object`元素以了解更多关于`embed`元素的信息。

## object

`object`元素的语法如下：

```js
<object data height type width></object>
```

### 属性

这里是`object`元素使用的属性：

+   `data`: 这是要嵌入的对象的 URL

+   `height`: 这是设置高度的属性

+   `type`: 这是对象的 MIME 类型

+   `width`: 这是设置宽度的属性

### 描述

`object`元素可以非常类似于`embed`元素使用。这历史上被用于`Flash`对象。

这里是一个示例：

```js
<object data="example.swf" type="application/x-shockwave-flash"></object>
```

### 参见

您还可以参考`audio`、`video`、`embed`和`param`属性以了解更多关于`object`元素的信息。

## `param`

`param`元素是参数元素：

```js
<param name="movie" value="video.swf"/>
```

### 属性

在`param`元素中使用的属性如下：

+   `name`: 这是参数的名称

+   `value`: 这是参数的值

### 描述

`param`元素为`object`元素定义了一个参数。此元素的父元素应该是`object`元素。

这里是一个示例。此示例在旧版浏览器中使用对象时很有用：

```js
<object data="example.swf" type="application/x-shockwave-flash">
    <param name="movie" value="example.swf" />
</object>
```

## 视频

`video`元素的语法如下：

```js
<video autoplay buffered controls crossorigin height loop muted played poster src width></video>
```

### 属性

在`video`元素中使用的属性如下：

+   `autoplay`: 这是一个布尔属性，告诉浏览器在加载停止后尽可能快地开始播放视频

+   `buffered`: 这是一个`read`对象，用于说明视频缓冲了多少

+   `controls`: 这是一个布尔属性，用于决定是否显示控件

+   `crossorigin`: 如果您计划在画布中修改视频且视频不在同一完全限定的域名上托管，则使用此属性进行 CORS 请求

+   `height`: 这是设置高度的属性

+   `loop`: 这表示是否循环视频

+   `muted`: 这表示是否静音音频

+   `played`: 这是一个`read`对象，用于读取视频播放了多少

+   `poster`: 这是将在视频可以播放之前显示的图像的 URL

+   `src`: 这是视频的 URL

+   `width`: 这是设置宽度的属性

### 描述

`video`元素是 HTML5 中引入的新元素。您可以使用它直接在浏览器中播放视频。这对于用户来说非常有用，因为用户不需要插件或特殊播放器来查看视频。此外，您还可以将`video`元素用作`canvas`元素的源。

如果浏览器只能播放某种类型的文件，您还可以使用`source`元素包含多个源。如果浏览器不支持`video`元素或文件类型，您可以将回退内容放入元素中。

这里是使用 `video` 元素的示例，以及另一个演示所有属性的可能值的示例：

```js
<video src="img/example.mp4" autoplay poster="example-poster.png"></video>
<video autoplay buffered controls crossorigin="anonymous" height="100px" loop muted played poster="cover.jpg" src="img/video.ogg" width="100px"></video>
```

### 参见

你也可以参考 `source` 和 `track` 属性来了解更多关于 `video` 元素的信息。

## audio

`audio` 元素的语法如下：

```js
<audio autoplay buffered controls loop muted played src volume></audio>
```

### 属性

`audio` 元素中使用的属性如下：

+   `autoplay`: 这是浏览器在可能的情况下立即开始播放音频，而不需要加载的属性

+   `buffered`: 这是缓冲时间范围的属性

+   `controls`: 这是浏览器显示控制按钮的属性

+   `loop`: 这是决定是否循环音频的属性

+   `muted`: 这是决定是否静音音频的属性

+   `played`: 这是音频播放时间范围的属性

+   `src`: 这是音频资源的 URL

+   `volume`: 这是范围在 0.0 到 1.0 之间的音量属性

### 描述

`audio` 元素是在 HTML5 中引入的。你可以在页面上添加音频，并让浏览器播放它。

你还可以使用 `source` 元素来包含多个源，以防浏览器可以播放某种类型的文件。如果浏览器不支持音频元素或文件类型，你可以在元素中放入回退内容。

这里是使用 `audio` 元素的示例：

```js
<audio src="img/test.mp3" autoplay loop>
    Your browsers does not support the audio element.
</audio>
```

### 参见

你也可以参考 `source` 和 `track` 属性来了解更多关于 `audio` 元素的信息。

## source

`source` 元素的语法如下：

```js
<source src type />
```

### 属性

`source` 元素中使用的属性如下：

+   `src`: 这是资源的 URL

+   `type`: 这是资源的 MIME 类型

### 描述

`source` 元素用于向 `audio`、`picture` 和 `video` 元素添加多个源。它必须是这些元素之一的孩子。你可以使用它来指定同一资源的多个格式。例如，你可以为视频提供两种不同的视频编码。如果浏览器不能播放第一个，它将回退到另一个。

这里是一个使用 `audio` 元素的示例：

```js
<audio autoplay controls>
    <source src="img/test.ogg" type="audio/ogg" />
    <source src="img/test.mp3" type="audio/mpeg">
</audio>
```

### 参见

你也可以参考 `audio` 和 `video` 属性来了解更多关于 `audio` 元素的信息。

## track

`track` 元素的语法如下：

```js
<track default kind label src />
```

### 属性

这里是 `track` 元素中使用的属性：

+   `default`: 这表示所选轨道是否为默认轨道。

+   `kind`: 这表示可以加载的不同类型的轨道。以下是值：`subtitles`、`captions`、`descriptions`、`chapters` 或 `metadata`。

+   `label`: 这是轨道的标题。

+   `src`: 这是资源的 URL。

### 描述

你主要会使用 `track` 元素来为视频添加字幕或子标题。

这里是一个带有字幕的示例视频：

```js
<video src="img/test.mp4">
    <track label="English" kind="captions" src="img/en.vtt" default>
    <track label="Spanish" kind="captions" src="img/sp.vtt">
</video>
```

# 表格

表格用于展示数据。它们使得定义行和列变得非常容易。在过去，表格用于创建布局，但如今这通过 CSS 来完成。它们应该仅用于显示表格数据。

## 表格

`table`元素的语法如下：

```js
<table></table>
```

### 描述

`table`元素是创建表格的根元素。本节中所有其他元素都必须是此元素的子元素。

下面是`table`元素的简单例子：

```js
<table>
    <tr>
        <td>Column in Row 1</td>
    </tr>
</table>
```

## caption

`caption`元素的语法如下：

```js
<caption></caption>
```

### 描述

`caption`元素将是表格的标题。此元素必须是`table`元素的第一个子元素。

下面是一个简单的例子：

```js
<table>
    <caption>Caption for the table</caption>
    <tr>
        <td>Column in Row 1</td>
    </tr>
</table>
```

## colgroup

`colgroup`元素是列分组元素：

```js
<colgroup span></colgroup>
```

### 属性

`span`属性表示该组跨越的列数。

### 描述

`colgroup`元素用于定义适用于所有列或列组的样式。由于新的 CSS 选择器可以针对所有列甚至某些特定列，因此此元素不如以前有用。

## tbody

`tbody`属性是表格主体元素：

```js
<tbody></tbody>
```

### 描述

`tbody`属性是表格的主体部分。所有数据行和列都应该放在这个元素中。此元素应有一个或多个`tr`元素作为其子元素。

下面是一个例子：

```js
<table>
    <tbody>
        <tr>
            <td>Column in Row 1</td>
        </tr>
    </tbody>
</table>
```

## thead

`thead`元素是表格表头元素：

```js
<thead></thead>
```

### 描述

`thead`元素是包含所有列标题的行。它必须出现在`tbody`或`tfoot`元素之前。

下面是一个例子：

```js
<table>
    <thead>
        <tr>
            <th>Heading 1</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>Column in Row 1</td>
        </tr>
    </tbody>
</table>
```

## tfoot

`tfoot`元素是表格的页脚元素：

```js
<tfoot></tfoot>
```

### 描述

`tfoot`元素是表格的页脚。它必须在任何`thead`元素之后使用，但可以在`tbody`之前或之后。`tfoot`元素的位置不会影响其渲染位置，它始终位于底部。

下面是一个例子：

```js
<table>
    <tbody>
        <tr>
            <td>Column in Row 1</td>
        </tr>
    </tbody>
    <tfoot>
        <tr>
            <td>Footer 1</td>
        </tr>
    </tfoot>
</table>
```

## tr

`tr`元素是表格行元素：

```js
<tr></tr>
```

### 描述

`tr`元素是行元素。每次需要在表格中添加另一行时，请使用此元素。此元素可以是`table`、`tbody`、`thead`或`tfoot`元素的子元素。您必须使用`td`或`th`作为其子元素。

下面是一个例子：

```js
<table>
    <tbody>
        <tr>
            <td>Column in Row 1</td>
        </tr>
    </tbody>
</table>
```

## td

`td`元素是表格单元格元素：

```js
<td colspan headers rowspan></td>
```

### 属性

在`td`元素中使用的属性如下：

+   `colspan`：这表示它将跨越多少列，以整数形式表示

+   `rowspan`：这表示`rowspan`属性将跨越多少行，以整数形式表示

+   `headers`：这是一个以空格分隔的字符串列表，与任何`th`元素的 ID 相匹配

### 描述

`td`元素是基本的表格列元素。`colspan`和`rowspan`属性允许您分别使列更宽和更高。

下面是一个例子：

```js
<table>
    <tbody>
        <tr>
            <td>Column in Row 1</td>
        </tr>
    </tbody>
</table>
```

## th

`th`元素是表格表头单元格元素：

```js
<th colspan rowspan></th>
```

### 属性

在`th`元素中使用的属性如下：

+   `colspan`：这表示`colspan`属性将跨越多少列，以整数形式表示

+   `rowspan`: 这表示`rowspan`属性将跨越的行数，以整数形式表示

### 描述

当我们在`thead`元素中添加列时，会使用`th`元素。

下面是一个示例：

```js
<table>
    <thead>
        <tr>
            <th>Header</th>
        </tr>
    </thead>
</table>
```

# 表单

表单非常适合从用户那里获取信息。它们通常包含多个元素，这些元素可以接受用户的输入。从用户输入中获取的数据随后被发送到服务器进行处理。

## 表单

`form`元素的语法如下：

```js
<form accept-charset action autocomplete enctype method name novalidate target></form>
```

### 属性

在`form`元素中使用的属性如下：

+   `accept-charset`: 这是服务器接受的字符编码列表。这可以是一个空格或逗号分隔的列表。

+   `action`: 这是处理表单的 URL。

+   `autocomplete`: 这让浏览器知道是否可以使用之前输入的值自动完成此表单。

+   `enctype`: 这设置了发送到服务器的内容的 MIME 类型。

+   `method`: 这是指提交表单时将使用的 HTTP 方法。可以是`Post`或`Get`。

+   `name`: 这是表单的名称。

+   `novalidate`: 这告诉浏览器不要验证表单。

+   `target`: 这表示响应将在何处显示。可以是：`_self`, `_blank`, `_parent`, 或 `_top`。

### 描述

`form`元素是文档中表单的根元素。当提交时，表单将包含所有输入到表单内部不同元素中的数据。

这里是语法的一个简单示例：

```js
<form action="processForm" method="post">
    <input type="text" name="text-input"/>
    <button type="submit">Submit!</button>
</form>
```

## 字段集

`fieldset`元素的语法如下：

```js
<fieldset disabled form name></fieldset>
```

### 属性

在`fieldset`元素中使用的属性如下：

+   `disabled`: 这将禁用字段集中的所有元素

+   `form`: 这是`form`属性所属的表单的 ID

+   `name`: 这是字段集的名称

### 描述

`fieldset`元素允许我们将相关的输入项组合在一起。大多数浏览器的默认样式是在字段集周围添加边框。

如果第一个元素是`legend`元素，则字段集将使用该元素作为其标签。

下面是使用`fieldset`元素的示例：

```js
<form action="processForm" method="post">
    <fieldset>
        <legend>This is a fieldset</legend>
        <input type="text" name="text-input" />
    </fieldset>
</form>
```

### 相关内容

您还可以参考`legend`属性以了解更多关于`fieldset`元素的信息。

## 图例

`legend`元素的语法如下：

```js
<legend></legend>
```

### 描述

`legend`元素将成为其子元素的`fieldset`元素的标签。

### 相关内容

您还可以参考`fieldset`元素以了解更多关于`legend`元素的信息。

## 标签

`label`元素的语法如下：

```js
<label accesskey for form></label>
```

### 属性

在`label`元素中使用的属性如下：

+   `accesskey`: 这是`accesskey`元素的快捷键

+   `for`: 这是表单元素的 ID，也是其标签的 ID

+   `form`: 这是与`form`属性关联的表单的 ID

### 描述

`label`元素用于标记输入。您可以将元素放入标签中或使用`for`属性。当标签正确链接到输入时，您可以点击标签，光标将定位到输入框。

下面是一个示例，涵盖了标记元素的不同方式：

```js
<form action="processForm" method="post">
    <label>First name: <input type="text" name="firstName" /></label>
    <label for="lastNameInput">Last name: </label><input id="lastNameInput" type="text" name="lastName" />
</form>
```

## 输入

`input`元素的语法如下所示：

```js
<input accept autocomplete autofocus checked disabled form formaction formenctype formmethod formnovalidate formtarget height inputmode max maxlength min minlength multiple name placeholder readonly required size spellcheck src step tabindex type value width></input>
```

### 属性

在`input`元素中使用的属性如下：

+   `accept`: 这用于指定网页接受的文件类型。

+   `autocomplete`: 这表示浏览器是否可以根据先前值自动完成此输入。

+   `autofocus`: 这允许浏览器自动将焦点放在元素上。这应该只用于一个元素。

+   `checked`: 这与单选按钮或复选框一起使用。这将选择页面加载时的值。

+   `disabled`: 这表示是否禁用元素。

+   `form`: 这表示表单的 ID。

+   `formaction`, `formenctype`, `formmethod`, `formnovalidate`, 和 `formtarget`: 如果这些属性与按钮或图像相关联，将覆盖表单的值。

+   `height`: 这用于设置图像的高度。

+   `inputmode`: 这为浏览器提供了显示哪个键盘的提示。例如，您可以使用数字来指定仅键盘。

+   `max`: 这是系统的最大数字或日期时间。

+   `maxlength`: 这是可以在网页中接受的字符的最大数量。

+   `min`: 这是系统的最小数字或日期时间。

+   `minlength`: 这是字符的最小数量。

+   `multiple`: 这表示是否可以有多个值。这与`email`或`file`一起使用。

+   `placeholder`: 这是当没有为该属性分配值时在元素中显示的文本。

+   `readonly`: 这使元素为只读格式。

+   `required`: 这是必须分配值且不能为空的元素。

+   `size`: 这是元素的大小。

+   `src`: 如果它是`img`类型，这将图像的 URL。

+   `step`: 这与`min`和`max`属性一起使用，以确定增量步骤。

+   `tabindex`: 这是使用 tab 键时元素的顺序。

+   `type`: 请参阅下一节以获取描述。

+   `value`: 这是元素的初始值。

+   `width`: 这是设置宽度的属性。

### 描述

`input`元素是获取用户数据的主要方式。根据使用的类型，该元素可能会有很大的变化。HTML5 增加了一些输入，同时也提供了验证。例如，`email`类型将验证电子邮件是否为有效的`email`。此外，类型还可以向浏览器提供有关要显示哪个键盘的提示。这对于拥有许多不同虚拟键盘的移动设备来说很重要。例如，`tel`类型将显示数字键盘而不是常规键盘。以下是不同类型键盘及其描述的概述：

+   `button`: 这是一个按钮。

+   `checkbox`: 这是一个复选框。

+   `color`: 对于大多数浏览器，这将创建一个颜色选择器；然而，HTML5 并不要求必须使用。

+   `date`: 这创建了一个日期选择器。

+   `datetime`: 这使用时区创建日期和时间选择器。

+   `datetime-local`: 这创建了一个没有时区的日期和时间选择器。

+   `email`: 这是一个用于电子邮件地址的文本输入。此类型会验证电子邮件。

+   `file`: 这选择一个文件。

+   `hidden`: 此属性不会显示，但值仍然是表单的一部分。

+   `image`: 这实际上创建了一个图像按钮。

+   `month`: 这可以输入月份和年份。

+   `number`: 这用于浮点数。

+   `password`: 这是一个文本输入，其中文本不会显示。

+   `radio`: 这是一个使用相同名称属性对多个元素进行分组控制的控件。只能从提供的列表中选择一个。

+   `range`: 这是一种选择数字范围的方式。

+   `reset`: 这将重置表单。

+   `search`: 这是一个文本输入。

+   `submit`: 这是一个提交表单的按钮。

+   `tel`: 这是一个输入电话号码的字段。

+   `text`: 这是你基本的文本输入。

+   `time`: 这是没有时区的时间。

+   `url`: 这是一个输入 URL 的字段。这将进行验证。

+   `week`: 这是输入周数的字段。

下面是 `text`、`e-mail` 和 `tel` 输入的示例：

```js
<input type="text" name="name" placeholder="enter email"/>
<input type="email" />
<input type="tel" />
```

## button

`button` 元素的语法如下：

```js
<button autofocus disabled form formaction formenctype formmethod formnovalidate formtarget name type value></button>
```

### 属性

在 `button` 元素中使用的属性如下：

+   `autofocus`: 这允许浏览器自动聚焦于具有此属性的元素。这应仅用于一个元素。

+   `disabled`: 这表示是否禁用元素。

+   `form`: 这是表单的 ID。

+   `formaction`、`formenctype`、`formmethod`、`formnovalidate` 和 `formtarget`：如果这是一个按钮或图像，这些将覆盖表单的值。

+   `name`: 这是表单按钮的名称。

+   `type`: 这会改变按钮的功能。值包括 `submit`——提交表单，`reset`——重置表单，`button`——默认不执行任何操作。

+   `value`: 这是元素的初始值。

### 描述

`button` 元素创建一个可点击的按钮。更改 `type` 属性将改变其行为。

下面是 `reset` 和 `submit` 按钮的示例：

```js
<button type="reset">Reset</button>
<button type="submit">Submit</button>
```

## select

`select` 元素的语法如下：

```js
<select autofocus disabled form multiple name required size ></select>
```

### 属性

在 `button` 元素中使用的属性如下：

+   `autofocus`: 这允许浏览器自动聚焦于此元素。这应仅用于一个元素。

+   `disabled`: 这表示是否禁用元素。

+   `form`: 这是表单的 ID。

+   `multiple`: 这表示是否可以选择多个项目。

+   `name`: 这是元素的名称。

+   `required`: 这用于检查选项是否需要值。

+   `size`: 这确定元素的行数。

### 描述

`button` 元素与 `option` 元素一起使用。可以将多个 `option` 元素添加到列表中，以选择或选择。当表单提交时，将使用所选选项的值。

下面是一个示例：

```js
<select name="select">
    <option value="1">One</option>
    <option value="2">Two</option>
</select>
```

### 参见

你也可以参考`optgroup`和`option`属性来了解更多关于`button`元素的信息。

## optgroup

`optgroup`元素是选项组元素：

```js
<optgroup disabled label></optgroup>
```

### 属性

在`optgroup`元素中使用的属性如下：

+   `disabled`: 这将禁用该组

+   `label`: 这是下拉菜单中的标题

### 描述

`optgroup`元素允许你将选项分组在一起。此元素的子元素需要是`option`元素。它们不可选择，也没有值。

这里是`outgroup`元素的一个例子，包含汽车品牌和型号：

```js
<select name="cars">
   <optgroup label="Ford">
       <option value="Fiesta">Fiesta</option>
       <option value="Taurus">Taurus</option>
   </optgroup>
    <optgroup label="Honda">
        <option value="Accord">Accord</option>
        <option value="Fit">Fit</option>
    </optgroup>
</select>
```

### 参见

你也可以参考`option`和`select`元素来了解更多关于`optgroup`元素的信息。

## option

`option`元素的语法如下：

```js
<option disabled selected value></option>
```

### 属性

在`option`元素中使用的属性如下：

+   `disabled`: 这表示元素是否禁用。

+   `selected`: 这表示选项是否被选中。每个选中的元素只能设置一个选项。

+   `value`: 这表示`option`的值。

### 描述

`option`元素是`select`元素中的实际项目。它们可以是`select`或`optgroup`元素的子元素。

这里有一个例子：

```js
<select name="select">
    <option value="1">One</option>
    <option value="2">Two</option>
</select>
```

### 参见

你也可以参考`select`和`optgroup`元素来了解更多关于`option`元素的信息。

## textarea

`textarea`元素的语法如下：

```js
<textarea autocomplete autofocus cols disabled form maxlength minlength name placeholder readonly required rows spellcheck wrap></textarea>
```

### 属性

在`textarea`元素中使用的属性如下：

+   `autocomplete`: 这表示浏览器是否可以根据先前值自动完成此输入。

+   `autofocus`: 这允许浏览器自动聚焦到该元素。这应该只用于一个元素。

+   `cols`: 这表示`textarea`元素的字数宽度。

+   `disabled`: 这表示是否禁用元素。

+   `form`: 这是表单的 ID。

+   `maxlength`: 这是最大字符数。

+   `minlength`: 这是最小字符数。

+   `name`: 这是元素的名称。

+   `placeholder`: 这是元素中没有值时显示的文本。

+   `readonly`: 这使元素只读。

+   `required`: 这表示元素是必需的，不能为空。

+   `rows`: 这表示`textarea`的行数。

+   `spellcheck`: 这表示元素是否应该进行拼写检查。

+   `wrap`: 这表示行如何换行。

### 描述

当你需要比单行更多的文本时，你会使用这个。

这里有一个例子：

```js
<textarea cols="20" rows="10" placeholder="Input text here"></textarea>
```

# 绘图元素

在 HTML 的早期版本中，如果你想要一个图形或图像，你必须在其他应用程序中创建它，并使用`img`元素将其拉入你的文档。HTML5 引入了一些新的元素和功能，以替换旧元素，允许你在浏览器中绘制自己的图像。

## canvas

`canvas`元素的语法如下：

```js
<canvas height width></canvas>
```

### 属性

在`canvas`元素中使用的属性如下：

+   `height`：这是一个设置高度的属性

+   `width`：这是一个设置宽度的属性

### 描述

`canvas` 元素用于绘图。你可以使用 JavaScript 来绘制线条、形状和图像；从视频中提取帧；以及使用 WebGL 等功能。HTML 元素只是你用来进行绘图的画布（恰如其名！）。所有的交互都在 JavaScript 中进行。

下面是一个小 `canvas` 元素的示例：

```js
<canvas height="400" width="400">
    Your browser does not support the canvas element.
</canvas>
```

## svg

`svg` 元素是 **可缩放矢量图形**（**SVG**）元素：

```js
<svg height viewbox width ></svg>
```

### 属性

在 `svg` 元素中使用到的属性如下：

+   `height`：这是设置高度的属性。

+   `viewbox`：这个属性设置了元素的范围。它接受四个数字，分别对应 `min-x`、`min-y`、`width` 和 `height`。

+   `width`：这是设置宽度的属性。

### 描述

SVG 并不是一个真正的 HTML 元素。它是一个拥有许多元素和属性的独立规范。有关于 SVG 的书籍完全就是写的这个。这个元素之所以存在，是因为你现在可以在 HTML 文档中创建内联 SVG。这为你提供了在二维绘图方面的很多灵活性，这是图像所不具备的。

下面是一个示例，展示了高度、宽度和视口之间的区别。视口占据了元素的范围，而高度和宽度则决定了元素的大小：

```js
<svg  preserveAspectRatio=""
    width="200" height="100" viewBox="0 0 400 200">
    <rect x="0" y="0" width="400" height="200" fill="yellow" stroke="black" stroke-width="3" />
</svg>
```
