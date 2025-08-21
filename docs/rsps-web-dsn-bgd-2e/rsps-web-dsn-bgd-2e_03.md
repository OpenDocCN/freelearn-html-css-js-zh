# 第三章：使用 Responsive.gs 构建一个简单的响应式博客

*在上一章中，我们安装了一些软件，这些软件将为我们的项目提供便利。在这里，我们将开始我们的第一个项目。在这个项目中，我们将构建一个响应式博客。*

*拥有博客对于一家公司来说是必不可少的。甚至一些财富 500 强公司，如联邦快递([`outofoffice.van.fedex.com/`](http://outofoffice.van.fedex.com/))，微软([`blogs.windows.com/`](https://blogs.windows.com/))和通用汽车([`fastlane.gm.com/`](http://fastlane.gm.com/))都有官方企业博客。博客是公司发布官方新闻以及与客户和大众联系的重要渠道。使博客具有响应式设计是使博客更易于读者访问的途径，这些读者可能通过手机或平板等移动设备访问网站。*

*由于我们在这个第一个项目中要构建的博客不会那么复杂，所以这一章对于刚接触响应式网页设计的人来说是一个理想的章节。*

*那么让我们开始吧。*

总之，在本章中，我们将涵盖以下主题：

+   深入了解 Responsive.gs 组件

+   审查博客蓝图和设计

+   整理网站文件和文件夹

+   查看 HTML5 元素以进行语义标记

+   构建博客标记

# Responsive.gs 组件

正如我们在第一章中提到的，*响应式网页设计*，Responsive.gs 是一个轻量级的 CSS 框架。它只包含构建响应式网站的最基本要求。在本节中，我们将看到 Responsive.gs 包含了什么。

## 类

Responsive.gs 附带了一系列可重复使用的类，用于形成响应式网格，使网页设计师更容易更快地构建网页布局。这些类包含经过精心校准和测试的预设样式规则。因此，我们可以简单地将这些类放入 HTML 元素中以构建响应式网格。以下是 Responsive.gs 中的类列表：

| 类名 | 用法 |
| --- | --- |
| `container` | 我们使用这个类来设置网页容器并将其对齐到浏览器窗口的中心。然而，这个类并不给出元素的宽度。Responsive.gs 给了我们根据需要设置宽度的灵活性。 |

| `row`，`group` | 我们使用这两个类来包装一组列。这两个类都设置了所谓的自清除浮动，以解决由 CSS `float`属性引起的一些布局问题。查看以下参考资料，了解有关 CSS `float`属性以及它可能对网页布局造成的问题的更多信息：

+   Louis Lazaris 的*CSS 浮动属性之谜*([`www.smashingmagazine.com/2009/10/19/the-mystery-of-css-float-property/`](http://www.smashingmagazine.com/2009/10/19/the-mystery-of-css-float-property/))

+   Chris Coyier 的*关于浮动的一切*([`css-tricks.com/all-about-floats/`](http://css-tricks.com/all-about-floats/))

|

| `col` | 我们使用这个类来定义网页的列。这个类设置了 CSS `float`属性。因此，任何设置了这个类的元素都必须包含在设置了`row`或`group`类的元素中，以避免 CSS `float`属性引起的问题。 |
| --- | --- |
| `gutters` | 我们使用这个类来在前面设置了`col`类的列之间添加空间。 |
| `span_{x}` | 这个类定义了列宽。因此，我们与`col`类一起使用这个类。Responsive.gs 有三种网格变体，这使我们在组织网页布局时更加灵活。Responsive.gs 有 12、16 和 24 列格式。这些变体设置在三个单独的样式表中。如果你下载了 Responsive.gs 包，然后解压缩，你会发现三个名为`responsive.gs.12col.css`、`responsive.gs.16col.css`和`responsive.gs.24col.css`的样式表。这些样式表之间唯一的区别是其中定义的`span_`类的数量。显然，24 列格式的样式表有最多的`span_{x}`类；该类从`span_1`到`span_24`。如果你需要更大的灵活性来划分你的页面，那么使用 Responsive.gs 的 24 列格式是一个好选择。尽管每列可能太窄。 |
| `clr` | 此类用于解决浮动问题。我们在使用行类不合适的情况下使用此类。 |

现在，让我们看看如何在示例中应用它们，以发现它们真正的作用。很多时候，你会看到一个网页被分成多列结构。以我们的示例为例；我们可以这样构建一个包含两列内容的网页：

```html
<div class="container">
<div class="row gutters">
  <div class="col span_6">
    <h3>Column 1</h3>
    <p>Lorem ipsum dolor sit amet, consectetur adipisicing    
elit. Veniam, enim.</p>
  </div>
  <div class="col span_6">
    <h3>Column 2</h3>
    <p>Lorem ipsum dolor sit amet, consectetur adipisicing
elit. Reiciendis, optio.</p>
  </div>
</div>
</div>
```

从上面的代码片段中可以看出，我们首先添加了包裹所有内容的`container`。然后，紧随其后的是带有`row`类的`div`，用于包裹列。同时，我们还添加了`gutters`类，以便在两列之间留出空白空间。在这个例子中，我们使用了 12 列格式。因此，为了将页面分成两个相等的列，我们为每个列添加了`span_6`类。这意味着`span_{x}`类的数量应该等于 12、16 或 24，以便根据我们使用的变体使列覆盖整个容器。因此，如果我们使用了 16 列格式，我们可以添加`span_8`。

在浏览器中，我们将看到以下输出：

![The classes](img/image00242.jpeg)

# 使用 HTML5 元素进行语义标记

保罗·博格在他的文章《语义代码：什么？为什么？怎么？》中写道：（[`boagworld.com/dev/semantic-code-what-why-how/`](http://boagworld.com/dev/semantic-code-what-why-how/)）

> *HTML 最初是用来描述文档内容的手段，而不是为了使其外观上令人愉悦。*

与传统的报纸或杂志等明显是为人类而设计的内容发布不同，网络既被人类阅读，也被搜索引擎和屏幕阅读器等机器阅读，这些机器帮助视觉受损的人浏览网站。因此，使我们的网站结构具有语义是非常鼓励的。语义标记使这些机器更好地理解内容，也使内容在不同格式下更易访问。

因此，HTML5 在使网络更具语义的使命中引入了一堆新元素。以下是我们将用于博客的元素列表：

| 元素 | 描述 |
| --- | --- |
| `<header>` | `<header>`元素用于指定一个部分的头部。虽然这个元素通常用于指定网站的头部，但也适合用于指定文章头部，例如我们放置标题和其他支持文章的部分。我们可以在单个页面中多次使用`<header>`。 |
| `<nav>` | `<nav>`元素用于表示网站的主要导航或页面部分的一组链接。 |
| `<article>` | `<article>`元素相当不言自明。该元素指定网站的文章，如博客条目或主页内容。 |
| `<main>` | `<main>`元素定义了部分的主要部分。这个元素可以用来做一些事情，比如包装文章内容。 |
| `<figure>` | `<figure>`元素用于指定文档中的图表、插图和图片。`<figure>`元素可以与`<figcaption>`一起使用，以添加图表的标题（如果需要的话）。 |
| `<figcaption>` | 如前所述，`<figcaption>`表示文档图表的标题。因此，它必须与`<figure>`元素一起使用。 |
| `<footer>` | 与`<header>`元素类似，`<footer>`元素通常用于指定网站页脚。但它也可以用来表示部分的结束或最低部分。 |

### 提示

参考这个速查表[`websitesetup.org/html5-cheat-sheet/`](http://websitesetup.org/html5-cheat-sheet/)，了解 HTML5 中的更多新元素。

## HTML5 搜索输入类型

除了新元素，我们还将在博客中添加一种特定的新输入类型，搜索。顾名思义，搜索输入类型用于指定搜索输入。在桌面浏览器中，您可能看不到明显的区别。您可能也不会立即看到搜索输入类型如何为网站和用户带来优势。

搜索输入类型将提升移动用户的体验。iOS、Android 和 Windows Phone 等移动平台已经配备了上下文屏幕键盘。键盘会根据输入类型而改变。您可以在下面的屏幕截图中看到，键盘显示了**搜索**按钮，让用户更方便地进行搜索：

![HTML5 搜索输入类型](img/image00243.jpeg)

### HTML5 placeholder 属性

HTML5 引入了一个名为`placeholder`的新属性。规范描述了这个属性作为一个短提示（一个词或短语），旨在在控件没有值时帮助用户进行数据输入，如下面的例子所示：

```html
<input type="search" name="search_form " placeholder="Search here…"> 
```

您会看到`placeholder`属性中的**在这里搜索…**显示在输入字段中，如下面的屏幕截图所示：

![HTML5 placeholder 属性](img/image00244.jpeg)

过去，我们依赖 JavaScript 来实现类似的效果。如今，有了`placeholder`属性，应用程序变得简单得多。

## HTML5 在 Internet Explorer 中

这些新的 HTML 元素使我们的文档标记更具描述性和意义。不幸的是，Internet Explorer 6、7 和 8 将无法识别它们。因此，无法应用到这些元素的选择器和样式规则；就好像这些新元素没有包含在 Internet Explorer 字典中一样。

这就是一个名为 HTML5Shiv 的 polyfill 发挥作用的地方。我们将包括 HTML5Shiv（[`github.com/aFarkas/html5shiv`](https://github.com/aFarkas/html5shiv)）来使 Internet Explorer 8 及更低版本认识这些新元素。阅读 Paul Irish 的以下帖子（[`paulirish.com/2011/the-history-of-the-html5-shiv/`](http://paulirish.com/2011/the-history-of-the-html5-shiv/)）了解 HTML5Shiv 背后的历史；它是如何发明和发展的。

此外，旧版的 Internet Explorer 无法渲染 HTML5 中的`placeholder`属性中的内容。幸运的是，我们可以使用一个 polyfill（[`github.com/UmbraEngineering/Placeholder`](https://github.com/UmbraEngineering/Placeholder)）来模拟旧版 Internet Explorer 中`placeholder`属性的功能。我们以后也会在博客中使用它。

## 在 Responsive.gs 包中查看 polyfills

Responsive.gs 还附带了两个 polyfills，以启用 Internet Explorer 6、7 和 8 中不支持的某些功能。从现在开始，让我们把这些浏览器版本称为“旧版 Internet Explorer”，好吗？

### 盒模型 polyfills

第一个 polyfill 是通过名为`boxsizing.htc`的**HTML 组件**（**HTC**）文件提供的。

HTC 文件与 JavaScript 非常相似，通常与 Internet Explorer 专有的 CSS 属性`behavior`一起使用，以为 Internet Explorer 添加特定功能。Responsive.gs 附带的`boxsizing.htc`文件将应用与 CSS3 `box-sizing`属性类似的功能。

Responsive.gs 在样式表中包含了`boxsizing.htc`文件，如下所示：

```html
* { 
  -webkit-box-sizing: border-box;
  -moz-box-sizing: border-box;
  box-sizing: border-box;
  *behavior: url(/scripts/boxsizing.htc); 
}
```

如上面的代码片段所示，Responsive.gs 应用了`box-sizing`属性，并在通配符选择器中包含了`boxsizing.htc`文件。这个通配符选择器也被称为通配符选择器；它选择文档中的所有元素，也就是说，在这种情况下，`box-sizing`将影响文档中的所有元素。

### 注意

`boxsizing.htc`文件路径必须是绝对路径或相对于 HTML 文档的路径，以使填充脚本起作用。这是一个技巧。这是我们强制使用的东西，以使旧版 Internet Explorer 表现得像现代浏览器一样。像上面的 HTC 文件一样使用并不符合 W3C 标准。 

请参考微软关于 HTC 文件的页面（[`msdn.microsoft.com/en-us/library/ms531018(v=vs.85).aspx`](http://msdn.microsoft.com/en-us/library/ms531018(v=vs.85).aspx)）。

### CSS3 媒体查询填充

Responsive.gs 附带的第二个填充脚本是`respond.js`（[`github.com/scottjehl/Respond`](https://github.com/scottjehl/Respond)），它将"神奇地启用"CSS3 `respond.js`以使其立即可用。无需配置；我们可以简单地在`head`标签中链接脚本，如下所示：

```html
<!--[if lt IE 9]>
<script src="img/respond.js"></script>
<![endif]-->
```

在上面的代码中，我们将脚本封装在`<!--[if lt IE 9]>`中，以便脚本只在旧版 Internet Explorer 中加载。

# 检查博客的线框图

建立网站与建造房子很相似；我们需要在堆砌所有砖块之前检查每个角落的规格。因此，在我们着手建立博客之前，我们将检查博客的线框图，看看博客的布局是如何排列的，以及将在博客上显示的内容。

让我们来看一下下面的线框图。这个线框图展示了在桌面屏幕上查看博客布局时的情况：

![检查博客的线框图](img/image00245.jpeg)

正如在上面的截图中所示，博客将是简单明了的。在页眉中，博客将有一个标志和一个搜索表单。在页眉下方，依次放置菜单导航、博客文章、用于导航到下一个或上一个文章列表的分页和页脚。

博客文章通常包括标题、发布日期、文章的特色图片和文章摘要。这个线框图是博客布局的抽象。我们将用它作为博客布局将如何排列的视觉参考。因此，尽管我们在前面的线框图中只显示了一篇文章，但实际上我们稍后会在实际博客中添加更多的文章。

当视口宽度被挤压时，以下是博客布局：

检查博客的线框图

当视口宽度变窄时，博客布局会自适应。值得注意的是，当我们改变布局时，我们不应该改变内容流和 UI 层次结构。确保桌面版和移动版之间的布局一致性将帮助用户快速熟悉网站，无论他们在哪里查看网站。如上面的线框图所示，我们仍然按照相同的顺序设置了 UI，尽管它们现在是垂直堆叠以适应有限的区域。

这个线框图值得一提的是，导航变成了一个 HTML 下拉选择。在建立博客的过程中，我们将讨论如何做到这一点。

现在，我们已经准备好工具并检查了博客布局，我们准备开始项目。我们将从创建和组织项目目录和资产开始。

# 组织项目目录和文件

通常，我们将不得不链接到某些文件，比如样式表和图片。不幸的是，网站并不聪明；它们无法自行找到这些文件。因此，我们必须正确设置文件路径，以避免链接错误。

这就是为什么在构建网站时拥有有组织的目录和文件是至关重要的。当我们在一个由许多人组成的团队中进行非常大型的项目，并且需要处理数十到数百个文件时，这将变得非常重要。管理不善的目录可能会让团队中的任何人都发疯。

有组织良好的目录将帮助我们最小化潜在的链接错误。这也将使项目在未来更易于维护和扩展。

# 创建和组织项目目录和资产的行动时间

执行以下步骤来设置项目的工作目录：

1.  转到`htdocs`文件夹。作为提醒，这个文件夹是位于本地服务器中的文件夹：

+   在 Windows 中的`C:\xampp\htdocs`

+   在 OSX 中的`/Applications/XAMPP/htdocs`

+   在 Ubuntu 中的`/opt/lampp/htdocs`

1.  创建一个名为`blog`的新文件夹。从现在开始，我们将把这个文件夹称为项目目录。

1.  创建一个名为`css`的新文件夹来存储样式表。

1.  创建一个名为`image`的新文件夹来存储图片。

1.  创建一个名为`scripts`的新文件夹来存储 JavaScript 文件。

1.  创建一个名为`index.html`的新文件；这个 HTML 文件将是博客的主页。从[`responsive.gs/`](http://responsive.gs/)下载 Responsive.gs 软件包。该软件包以`.zip`格式提供。解压缩软件包以释放其中的文件。在那里，你会发现许多文件，包括样式表和 JavaScript 文件，正如你从以下截图中所看到的：![Time for action – creating and organizing project directories and assets](img/image00247.jpeg)

Responsive.gs 中的文件

1.  将`responsive.gs.12col.css`移动到项目目录的`css`文件夹中；这是我们需要的 Responsive.gs 唯一的样式表。

1.  将`boxsizing.htc`移动到项目目录的`scripts`文件夹中。

1.  Responsive.gs 软件包中的`respond.js`文件已过时。让我们从 GitHub 存储库（[`github.com/scottjehl/Respond/blob/master/src/respond.js`](https://github.com/scottjehl/Respond/blob/master/src/respond.js)）下载最新版本的 Respond.js，并将其放在项目目录的`scripts`文件夹中。

1.  从[`github.com/aFarkas/html5shiv`](https://github.com/aFarkas/html5shiv)下载 HTML5Shiv。将 JavaScript 文件`html5shiv.js`放在`scripts`文件夹中。

1.  我们还将使用由 James Brumond 开发的占位符填充（[`github.com/UmbraEngineering/Placeholder`](https://github.com/UmbraEngineering/Placeholder)）。James Brumond 为不同的场景开发了四种不同的 JavaScript 文件。

1.  我们将在这里使用的脚本是`ie-behavior.js`，因为这个脚本专门针对 Internet Explorer。下载脚本（[`raw.githubusercontent.com/UmbraEngineering/Placeholder/master/src/ie-behavior.js`](https://raw.githubusercontent.com/UmbraEngineering/Placeholder/master/src/ie-behavior.js)）并将其重命名为`placeholder.js`，以使其更明显地表明这个脚本是一个占位符填充。将其放在项目目录的`scripts`文件夹中。

1.  博客将需要一些图像作为帖子的特色图像。在本书中，我们将使用以下屏幕截图中显示的图像，这些图像是由 Levecque Charles（[`twitter.com/Charleslevecque`](https://twitter.com/Charleslevecque)）和 Jennifer Langley（[`jennifer-langley.squarespace.com/photography/`](https://jennifer-langley.squarespace.com/photography/)）连续拍摄的：![行动时间-创建和组织项目目录和资产](img/image00248.jpeg)

### 提示

在 Unsplash（[`unsplash.com/`](http://unsplash.com/)）上找到更多免费的高清图像。

1.  我们将为博客添加一个网站图标。网站图标是一个小图标，出现在浏览器标签旁边的标题旁边，这对于读者快速识别博客将会很有帮助。以下是一个屏幕截图，显示了 Chrome 中的一些固定标签。我敢打赌，你仍然能够通过查看网站图标来识别这些标签中的网站：![行动时间-创建和组织项目目录和资产](img/image00249.jpeg)

Google Chrome 固定标签

1.  此外，我们还将添加 iOS 图标。在 iPhone 和 iPad 等苹果设备上，我们可以将网站固定在主屏幕上，以便快速访问网站。这就是苹果图标派上用场的地方。iOS（iPhone/iPad 操作系统）将显示我们提供的图标，如下面的屏幕截图所示，就像是一个本地应用程序：![行动时间-创建和组织项目目录和资产](img/image00250.jpeg)

将网站添加到 iOS 主屏幕

1.  这些图标包含在随本书提供的源文件中。将这些图标复制并粘贴到我们刚刚在步骤 5 中创建的图像文件夹中，如下面的屏幕截图所示：![行动时间-创建和组织项目目录和资产](img/image00251.jpeg)

### 提示

使用 AppIconTemplate 快速轻松地创建网站图标和 iOS 图标。

AppIconTemplate（[`appicontemplate.com/`](http://appicontemplate.com/)）是一个 Photoshop 模板，可以让我们轻松设计图标。该模板还附带了 Photoshop 操作，可以通过几次点击生成图标。

## *刚刚发生了什么？*

我们刚刚为这个项目创建了一个目录，并将一些文件放入了该目录。这些文件包括 Responsive.gs 样式表和 JavaScript 文件、图像和图标，以及一些 polyfill。我们还创建了一个`index.html`文件，这将是博客的主页。此时，项目目录应包含如下屏幕截图中显示的文件：

![刚刚发生了什么？](img/image00252.jpeg)

当前的工作目录中的文件和文件夹

## 尝试使目录结构更有组织性

许多人对如何组织他们项目的目录结构有自己的偏好。在上一节中显示的只是一个例子，是我个人管理该项目目录的方式。

尝试进一步使目录更有组织，并满足您对组织的个人偏好。一些常见的想法如下：

+   缩短文件夹名称，即`js`和`img`，而不是 JavaScript 和 Image

+   将`js`、`img`和`css`文件夹全部放在一个名为`assets`的新文件夹中

## 小测验-使用 polyfill

在本书的早些时候，我们讨论了 polyfill，并提到了一些我们将在博客中实现的 polyfill 脚本。

Q1.你认为何时使用 polyfill 会更合适？

1.  当博客在 Internet Explorer 6 中查看时。

1.  当浏览器不支持该功能时。

1.  当我们需要在网站上添加新功能时。

1.  我们可以随时使用它。

# 博客 HTML 结构

我们在上一节中已经介绍了项目目录和文件的结构。现在让我们开始构建博客标记。正如我们提到的，我们将使用一些 HTML5 元素来形成更有意义的 HTML 结构。

# 行动时间-构建博客

执行以下步骤来构建博客：

1.  打开我们在上一节*行动时间-创建和组织项目目录和资产*的第 6 步中创建的`index.html`文件。让我们从添加最基本的 HTML5 结构开始，如下所示：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Blog</title>
</head>
<body>

</body>
</html>
```

在这里，设置`DOCTYPE`，它已经被简化到最低形式。HTML5 中的`DOCTYPE`格式现在比其 HTML4 对应格式更短更干净。然后，我们设置页面的语言，这里设置为`en`（英语）。您可以根据自己的语言更改它；在[`en.wikipedia.org/wiki/List_of_ISO_639-1_codes`](http://en.wikipedia.org/wiki/List_of_ISO_639-1_codes)上找到您本地语言的代码。

我们还将字符编码设置为`UTF-8`，以使浏览器能够将 Unicode 字符（如`U+20AC`）呈现为可读格式`€`。

1.  在`head`标签中的`charset`元标签下方，添加以下 meta：

```html
<meta http-equiv="X-UA-Compatible" content="IE=edge">
```

Internet Explorer 有时会表现得很奇怪，突然切换到兼容模式，并以 Internet Explorer 8 和 7 中查看的方式呈现页面。这个 meta 标签的添加将阻止这种情况发生。它将强制 Internet Explorer 以最新标准的最高支持度呈现页面。

1.  在`http-equiv`元标签下方，添加以下 meta 视口标签：

```html
<meta name="viewport" content="width=device-width, initial-scale=1">
```

正如我们在第一章中提到的*响应式网页设计*，前面的视口 meta 标签规范定义了网页视口宽度，以跟随设备视口大小。它还定义了在首次打开网页时的网页比例为 1:1。

1.  使用`link`标签将苹果图标链接如下：

```html
<link rel="apple-touch-icon" href="image/icon.png">
```

根据苹果的官方说明，通常需要包含多个图标源，以满足 iPhone、iPad 和具有 Retina 屏幕的设备。这对我们的博客实际上并不是必要的。诀窍在于，我们通过单一来源提供所需的最大尺寸，即 512 像素，如前面的屏幕截图所示。

### 注意

前往苹果文档，为 Web Clip 指定网页图标（[`developer.apple.com/library/ios/documentation/AppleApplications/Reference/SafariWebContent/ConfiguringWebApplications/ConfiguringWebApplications.html`](https://developer.apple.com/library/ios/documentation/AppleApplications/Reference/SafariWebContent/ConfiguringWebApplications/ConfiguringWebApplications.html)），以供进一步参考。

1.  在标题下方添加描述 meta 标签，如下所示：

```html
<meta name="description" content="A simple blog built using Responsive.gs">
```

1.  博客的描述将显示在**搜索引擎结果页面**（**SERP**）中。在这一步中，我们将构建博客标题。首先，让我们添加 HTML5 的`<header>`元素以及用于样式的类，以包装标题内容。在`body`标签中添加以下内容：

```html
<header class="blog-header row">

</header>
```

1.  在我们在第 9 步中添加的`<header>`元素中，添加一个新的`<div>`元素，带有`container`和`gutters`类，如下所示：

```html
<header class="blog-header row">
<div class="container gutters">

</div>
</header>
```

参考本章前面显示的表格，`container`类将使博客标题内容居中于浏览器窗口，而`gutters`类将在下一步中添加的列之间添加间距。

1.  在新的列中创建一个包含博客标志/名称的`<div>`元素，以及 Responsive.gs 的`col`和`span_9`类，将`<div>`元素设置为列并指定宽度。不要忘记添加类以添加自定义样式：

```html
<header class="blog-header row">
<div class="container gutters">
       <div class="blog-name col span_9">
<a href="/">Blog</a>
</div>
</div>
</header>
```

1.  参考博客线框图，我们将在博客标志/名称旁边放置一个搜索表单。因此，使用 Responsive.gs 的`col`和`span_3`类以及输入搜索类型创建另一个新的列。在标志标记下方添加以下`<div>`元素：

```html
<header class="blog-header row">
<div class="container gutters">
       <div class="blog-name col span_9">
   <a href="/">Blog</a>
</div>
<div class="blog-search col span_3">
           <div class="search-form">
             <form action="">
 <input class="input_full" type="search"      placeholder="Search here...">
  </form>
            </div>
   </div>
</div>
</header>
```

正如我们在本章前面提到的，我们使用了一个输入搜索类型来提供更好的用户体验。这个输入将显示移动屏幕键盘，带有一个特殊的键，允许用户点击**搜索**按钮并立即运行搜索。我们还使用 HTML5 的`placeholder`属性添加占位文本，向用户显示他们可以通过输入框在博客中进行搜索。

1.  构建完头部博客后，我们将构建博客导航。在这里，我们将使用 HTML5 的`nav`元素来定义一个新的导航部分。创建一个带有支持样式的`nav`元素。在头部构建下方添加`nav`元素如下：

```html
...     
</div>
</header> 
<nav class="blog-menu row">

</nav>
```

1.  在`nav`元素内，创建一个带有`container`类的`div`元素，将导航内容对齐到浏览器窗口的中心：

```html
<nav class="blog-menu">
<div class="container">
</div>
</nav>
```

1.  根据线框图，博客将在链接菜单上有五个项目。我们将使用`ul`元素布置这些链接。在容器内添加链接，如下所示的代码片段：

```html
<nav class="blog-menu row">
  <div class="container">
       <ul class="link-menu">
         <li><a href="/">Home</a></li>
         <li><a href="#">Archive</a></li>
         <li><a href="#">Books</a></li>
         <li><a href="#">About</a></li>
         <li><a href="#">Contact</a></li>
      </ul>
</div>
</nav>
```

1.  完成导航构建后，我们将构建博客内容部分。根据线框图，内容将包括一系列文章。首先，让我们添加 HTML5 的`<main>`元素，将内容包裹在导航下方如下：

```html
...
</ul>  
</nav>
<main class="blog-content row">

</main>
```

我们使用`<main>`元素，因为我们认为文章是博客的主要部分。

1.  与其他博客部分一样——头部和导航——我们添加一个容器`<div>`将博客文章对齐到中心。在`<main>`元素内添加这个`<div>`元素：

```html
<main class="blog-content row">
   <div class="container">

</div>
</main>
```

1.  现在我们将创建博客文章的标记。把博客文章想象成一篇文章。因此，在这里我们将使用`<article>`元素。在第 17 步中添加的容器`<div>`内添加`<article>`元素如下：

```html
<main class="blog-content row">
<div class="container">
  <article class="post row">

  </article>
</div>
</main>
```

1.  如前所述，`<header>`元素不仅限于定义头部。博客也可以用来定义一个部分的头部。在这种情况下，除了博客头部，我们将使用`<header>`元素来定义包含文章标题和发布日期的文章头部部分。

1.  在文章元素内添加`<header>`元素：

```html
<article class="post row">
<header class="post-header">
<h1 class="post-title">
<a href="#">Useful Talks &amp; Videos for Mastering CSS</a>
  </h1>
      <div class="post-meta">
     <ul>
        <li class="post-author">By John Doe</li>
        <li class="post-date">on January, 10 2014</li>
     </ul>
     </div>
</header>
</article>
```

1.  一图胜千言。因此，使用图片来支持文章是常态。在这里，我们将在文章头部下方显示图片。我们将特色图片与文章摘要一起分组，作为文章摘要，如下所示：

```html
...
 </header>
 <div class="post-summary">
<figure class="post-thumbnail">
<img src="img/village.jpg" height="1508" width="2800" alt="">
</figure>
<p class="post-excerpt">Lorem ipsum dolor sit amet,   consectetur adipisicing elit. Aspernatur, sequi, voluptatibus, consequuntur vero iste autem aliquid qui et rerum vel ducimus ex enim quas!...<a href="#">Read More...</a></p>
  </div>
</article>
```

随后添加几篇文章。可选地，你可以在其他文章中排除特色图片。

1.  添加了一堆文章后，我们现在将添加文章分页。分页是一种常见的页面导航形式，允许我们跳转到下一个或上一个文章列表。通常，分页位于最后一篇文章项之后的页面底部。

博客的分页包括两个链接，用于导航到下一页和上一页，以及一个小节用于放置页面数字，显示用户当前所在的页面。

1.  在最后一篇文章后添加以下代码：

```html
...
</article>
<div class="blog-pagination">
<ul>
  <li class="prev"><a href="#">Prev. Posts</a></li>
  <li class="pageof">Page 2 of 5</li>
  <li class="next"><a href="#">Next Posts</a></li>
</ul>
</div>
```

1.  最后，我们将构建博客页脚。我们可以使用 HTML5 的`<footer>`元素来定义博客页脚。页脚结构与头部相同。页脚将有两列；分别包含博客页脚链接（或者，我们称之为次要导航）和版权声明。这些列将被包裹在一个`<div>`容器内。在主要部分添加以下页脚，如下所示：

```html
      …
</main> 
<footer class="blog-footer row">
   <div class="container gutters">
     <div class="col span_6">
<nav id="secondary-navigation"  class="social-   media">
          <ul>
           <li class="facebook">
<a href="#">Facebook</a>
  </li>
           <li class="twitter">
<a href="#">Twitter</a></li>
           <li class="google">
<a href="#">Google+</a>
   </li>
         </ul>
       </nav>
     </div>
   <div class="col span_6">
<p class="copyright">&copy; 2014\. Responsive  Blog.</p>
   </div>
   </div>
</footer>
```

## *刚刚发生了什么？*

我们刚刚完成了博客的 HTML 结构——头部、导航、内容和页脚。假设你一直在密切关注我们的指示，你可以在`http://localhost/blog/`或`http://{coputer-username}.local/blog/`中访问博客。

然而，由于我们还没有应用任何样式，你会发现博客看起来很简单，布局还没有组织好：

![刚刚发生了什么？](img/image00253.jpeg)

当前阶段的博客外观

我们将在下一章中为博客添加样式。

## 英雄试试看-创建更多博客页面

在本书中，我们只构建了博客的主页。但是，您可以通过创建更多页面来扩展博客，例如添加关于页面、单篇文章内容页面和带有联系表单的页面。您可以重用本章中构建的 HTML 结构。删除`<main>`元素内的任何内容，并根据需要替换为内容。

## 快速测验-HTML5 元素

让我们以有关 HTML5 的简单问题结束本章：

Q1. `<header>`元素用于什么？

1.  它用于表示网站页眉。

1.  它用于表示一组介绍和导航辅助。

Q2. `<footer>`元素用于什么？

1.  它用于表示网站页脚。

1.  它用于表示部分的结束或最低部分。

Q3. 在单个页面内允许多次使用`<header>`和`<footer>`元素吗？

1.  是的，只要语义上合乎逻辑。

1.  不，这被认为是多余的。

# 总结

在本章中，我们开始了我们的第一个项目。在本章的早些时候，我们探讨了 Responsive.gs 组件，了解了 Responsive.gs 如何构建响应式网格，以及用于塑造网格的类。

我们还讨论了 HTML5，包括新元素，即在不支持特定功能的浏览器中模仿 HTML5 功能的 polyfills。然后，我们使用 HTML5 构建博客标记。

在下一章中，我们将更多关注使用 CSS3 标记博客，并添加一些 JavaScript。我们还将调试博客，以解决在旧版 Internet Explorer 中出现的错误。
