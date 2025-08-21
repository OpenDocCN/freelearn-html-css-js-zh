# 第五章：使用 Bootstrap 开发投资组合网站

*Bootstrap ([`getbootstrap.com/`](http://getbootstrap.com/)) 是最坚固的前端开发框架之一。它具有令人惊叹的功能，如响应式网格、用户界面组件和 JavaScript 库，让我们能够快速构建响应式网站。*

*Bootstrap 如此受欢迎，以至于 Web 开发社区积极支持它，通过开发各种形式的扩展来添加额外功能。如果 Bootstrap 提供的标准功能不足够，可以有扩展来满足特定需求。*

*在本章中，我们将开始第二个项目。我们将使用 Bootstrap 构建一个响应式的投资组合网站。因此，本章显然对那些从事摄影、平面设计和插图等创意领域工作的人很有用。*

*在这里，我们还将使用 Bootstrap 扩展来为投资组合网站提供侧栏导航。在使用 Bootstrap 之后，我们将把 LESS 作为网站样式表的基础。*

*让我们继续。*

本章我们将讨论以下内容：

+   探索 Bootstrap 组件

+   研究 Bootstrap 扩展以实现侧栏导航

+   检查投资组合网站蓝图和设计

+   使用 Bower 和 Koala 设置和组织项目目录和资产

+   构建投资组合网站 HTML 结构

# Bootstrap 组件

与我们在第一个项目中使用的 Responsive.gs 框架不同，Bootstrap 附带了额外的组件，这些组件在 Web 上通常使用。因此，在我们进一步开发投资组合网站之前，首先让我们探索这些组件，主要是我们将在网站中使用的响应式网格、按钮和表单元素等。

### 注意

坦率地说，官方的 Bootstrap 网站([`getbootstrap.com/`](http://getbootstrap.com/))始终是与 Bootstrap 相关的任何内容保持最新的最佳来源。因此，在这里，我想指出一些直接的关键事项。

## Bootstrap 响应式网格

Bootstrap 配备了一个响应式网格系统，以及形成列和行的支持类。在 Bootstrap 中，我们使用这些前缀类来构建列：`col-xs-`、`col-sm-`、`col-md-`和`col-lg-`。然后是列号，范围从`1`到`12`，用于定义列的大小以及针对特定视口大小的列。请参阅以下表格以获取有关前缀的更多详细信息：

| 前缀 | 描述 |
| --- | --- |
| `col-xs-` | 这指定了 Bootstrap 定义的最小（超小）视口大小的列，即小于或等于 768 像素 |
| `col-sm-` | 这指定了 Bootstrap 定义的小视口大小的列，即大于或等于 768 像素。 |
| `col-md-` | 这指定了 Bootstrap 定义的中等视口大小的列，即大于或等于 992 像素 |
| `col-lg-` | 这指定了 Bootstrap 定义的大视口大小的列，即大于或等于 1,200 像素 |

在下面的示例中，我们在一行中设置了三列，每列分配了一个`col-sm-4`类：

```html
<div class="row">
  <div class="col-sm-4"></div>
  <div class="col-sm-4"></div>
  <div class="col-sm-4"></div>
</div>
```

因此，每列的大小将相同，并且它们会缩小到 Bootstrap 定义的小视口大小（≥ 768px）。以下屏幕截图显示了在浏览器中添加一些样式后的标记效果：

![Bootstrap 响应式网格](img/image00282.jpeg)

查看小于 768 像素的视口大小的示例，所有这些列将开始堆叠——第一列在顶部，第三列在底部，如下图所示：

![Bootstrap 响应式网格](img/image00283.jpeg)

此外，我们可以添加多个类来指定多个视口大小内的列比例，如下所示：

```html
<div class="row">
  <div class="col-sm-6 col-md-2 col-lg-4"></div>
  <div class="col-sm-3 col-md-4 col-lg-4"></div>
  <div class="col-sm-3 col-md-6 col-lg-4"></div>
</div>
```

根据上述示例，在 Bootstrap 定义的大视口大小（≥ 1,200 像素）内，列的大小将相同，如下截图所示：

![Bootstrap 响应式网格](img/image00284.jpeg)

然后，当我们在中等视口大小下查看时，根据每个列上分配的类，列的比例将开始变化。第一列的宽度将变小，第二列将保持相同比例，而第三列将变大，如下截图所示：

![Bootstrap 响应式网格](img/image00285.jpeg)

当网站处于 Bootstrap 定义的中等和小视口大小的临界点时（大约为 991 像素），列的比例将再次开始变化，如下截图所示：

![Bootstrap 响应式网格](img/image00286.jpeg)

### 注意

要了解如何构建 Bootstrap 网格，请前往 Bootstrap 官方网站的网格系统部分（[`getbootstrap.com/css/#grid`](http://getbootstrap.com/css/#grid)）。

## Bootstrap 按钮和表单

我们将在网站中加入其他组件，如按钮和表单。我们将创建一个在线联系方式，用户可以通过该方式与我们联系。在 Bootstrap 中，按钮由`btn`类和`btn-default`组成，以应用 Bootstrap 默认样式，如下代码所示：

```html
<button type="button" class="btn btn-default">Submit</button>
<a class="btn btn-default">Send</a>
```

将`btn-default`类替换为`btn-primary`、`btn-success`或`btn-info`，以给按钮指定颜色，如下代码所示：

```html
<button type="button" class="btn btn-info">Submit</button>
<a class="btn btn-success">Send</a>
```

以下代码片段使用这些类定义按钮大小：`btn-lg`使按钮变大，`btn-sm`使其变小，`btn-xs`使按钮变得更小，如下代码所示：

```html
<button type="button" class="btn btn-info btn-lg">Submit</button>
<a class="btn btn-success btn-sm">Send</a>
```

以下截图显示了在添加前述类时按钮大小的变化：

![Bootstrap 按钮和表单](img/image00287.jpeg)

Bootstrap 允许我们以多种方式显示按钮，例如将一系列按钮内联显示或在按钮中添加下拉切换。要了解如何构建这些类型的按钮，请前往 Bootstrap 官方网站的按钮组（[`getbootstrap.com/components/#btn-groups`](http://getbootstrap.com/components/#btn-groups)）和按钮下拉切换（[`getbootstrap.com/components/#btn-dropdowns`](http://getbootstrap.com/components/#btn-dropdowns)）部分获取更多帮助和详细信息。

![Bootstrap 按钮和表单](img/image00288.jpeg)

Bootstrap 按钮组和带下拉切换的按钮

Bootstrap 还提供了一些可重复使用的类来为表单元素（如`<input>`和`<textarea>`）添加样式。Bootstrap 使用`form-control`类来为表单元素添加样式。样式轻巧得体，如下截图所示：

![Bootstrap 按钮和表单](img/image00289.jpeg)

有关在 Bootstrap 中对表单元素进行样式和排列的更多信息，请参阅 Bootstrap 官方页面的表单部分（[`getbootstrap.com/css/#forms`](http://getbootstrap.com/css/#forms)）。

## Bootstrap Jumbotron

Bootstrap 将 Jumbotron 描述如下：

> *“一种轻量灵活的组件，可以选择性地扩展整个视口，以展示站点上的关键内容”（[`getbootstrap.com/components/#jumbotron`](http://getbootstrap.com/components/#jumbotron)）*

Jumbotron 是一个特殊的部分，用于显示网站的首行消息，如营销文案、口号或特别优惠，另外还有一个按钮。Jumbotron 通常放置在折叠区域上方和导航栏下方。要在 Bootstrap 中构建 Jumbotron 部分，请应用`jumbotron`类，如下所示：

```html
<div class="jumbotron">
  <h1>Hi, This is Jumbotron</h1>
<p>Place the marketing copy, catchphrases, or special offerings.</p>
  <p><a class="btn btn-primary btn-lg" role="button">Got it!</a></p>
</div>
```

使用 Bootstrap 默认样式，Jumbotron 的外观如下：

![Bootstrap Jumbotron](img/image00290.jpeg)

这是默认样式下的 Jumbotron 外观

### 注意

有关 Bootstrap Jumbotron 的更多细节可以在 Bootstrap 组件页面找到（[`getbootstrap.com/components/#jumbotron`](http://getbootstrap.com/components/#jumbotron)）。

## Bootstrap 第三方扩展

无法满足每个人的需求，Bootstrap 也是如此。有许多形式的扩展被创建出来，包括 CSS、JavaScript、图标、起始模板和主题，以扩展 Bootstrap。在这个页面上找到完整的列表（[`bootsnipp.com/resources`](http://bootsnipp.com/resources)）。

在这个项目中，我们将包括一个名为 Jasny Bootstrap（[`jasny.github.io/bootstrap/`](http://jasny.github.io/bootstrap/)）的扩展，由 Arnold Daniels 开发。我们将主要用它来整合 off-canvas 导航。off-canvas 导航是响应式设计中的一种流行模式；菜单导航首先设置在网站的可见区域之外，通常只有在点击或轻触时才会滑入，如下面的截图所示：

![Bootstrap 第三方扩展](img/image00291.jpeg)

当用户点击三条杠图标时，off-canvas 部分滑入

### Jasny Bootstrap off-canvas

Jasny Bootstrap 是一个为原始 Bootstrap 添加额外构建块的扩展。Jasny Bootstrap 是以 Bootstrap 为设计基础的；它几乎在每个方面都遵循 Bootstrap 的约定，包括 HTML 标记、类命名、JavaScript 函数以及 API。

如前所述，我们将使用这个扩展在作品集网站中包含 off-canvas 导航。以下是一个构建 off-canvas 导航的 Jasny Bootstrap 示例代码片段：

```html
<nav id="offcanvas-nav" class="navmenu navmenu-default navmenu-fixed-left offcanvas" role="navigation">
  <ul class="nav navmenu-nav">
    <li class="active"><a href="#">Home</a></li>
    <li><a href="#">Link</a></li>
    <li><a href="#">Link</a></li>
  </ul>
</nav>
<div class="navbar navbar-default navbar-fixed-top">
<button type="button" class="navbar-toggle" data-toggle="offcanvas" data-target="#offcanvas-nav" data-target="body">
    <span class="icon-bar"></span>
    <span class="icon-bar"></span>
    <span class="icon-bar"></span>
  </button>
</div>
```

从上面的代码片段可以看出，构建 off-canvas 导航需要大量的 HTML 元素、类和属性。首先，我们需要两个元素，`<nav>`和`<div>`，分别包含菜单和切换导航菜单的按钮。`<nav>`元素被赋予一个 ID，作为唯一的参考，通过`<button>`中的`data-target`属性来定位目标菜单。

在这些元素中添加了一些类和属性，用于指定颜色、背景、位置和功能：

+   `navmenu`：Jasny Bootstrap 有一种新类型的导航，称为 navmenu。`navmenu`类将垂直显示导航，并放置在网站内容的侧面——右侧或左侧，而不是顶部。

+   `navmenu-default`：这个类将使用默认样式设置`navmenu`类，主要是浅灰色。如果你喜欢深色，可以使用`navmenu-inverse`类。看一下下面的截图：![Jasny Bootstrap off-canvas](img/image00292.jpeg)

off-canvas 导航的两种默认颜色

+   `navmenu-fixed-left`类将导航菜单定位在左侧。使用`navmenu-fixed-right`类将其设置在右侧。

+   `offcanvas`类是将导航菜单设置在画布之外的主要类。

+   `<button>`中的`data-target="#offcanvas-nav"`代码作为一个选择器，指向具有给定 ID 的特定导航菜单。

+   `data-toggle="offcanvas"`代码告诉按钮切换 off-canvas 导航。此外，原始的 Bootstrap 还附带了几种`data-toggle`类型，用于连接不同的小部件，比如模态框（`data-toggle="modal"`）、下拉菜单（`data-toggle="dropdown"`）和选项卡（`data-toggle="tab"`）。

+   `data-target="body"`让网站主体在 off-canvas 导航被切换时同时滑动。Jasny Bootstrap 称之为推动菜单；访问这个页面（[`jasny.github.io/bootstrap/examples/navmenu-push/`](http://jasny.github.io/bootstrap/examples/navmenu-push/)）来看它的实际效果。

### 注意

此外，Jasny Bootstrap 提供了两种额外类型的画布导航，名为滑入菜单（[`jasny.github.io/bootstrap/examples/navmenu/`](http://jasny.github.io/bootstrap/examples/navmenu/)）和展示菜单（[`jasny.github.io/bootstrap/examples/navmenu-reveal/`](http://jasny.github.io/bootstrap/examples/navmenu-reveal/)）-请访问包含的 URL 以查看它们的运行情况。

# 深入了解 Bootstrap

探索 Bootstrap 组件的每一寸都超出了本书的能力范围。因此，我们只讨论了 Bootstrap 中对项目至关重要的一些内容。除了 Bootstrap 官方网站（[`getbootstrap.com/`](http://getbootstrap.com/)）之外，以下是一些深入研究 Bootstrap 的专门参考资料，您可以查看：

+   初学者的 Bootstrap 教程由 Coder's Guide 提供（[`www.youtube.com/watch?v=YXVoqJEwqoQ`](http://www.youtube.com/watch?v=YXVoqJEwqoQ)），这是一系列视频教程，帮助初学者快速上手 Bootstrap

+   Twitter Bootstrap Web Development How-To, David Cochran, Packt Publishing ([`www.packtpub.com/web-development/twitter-bootstrap-web-development-how-instant`](http://www.packtpub.com/web-development/twitter-bootstrap-web-development-how-instant))

+   Mobile First Bootstrap, Alexandre Magno, Packt Publishing ([`www.packtpub.com/web-development/mobile-first-bootstrap`](http://www.packtpub.com/web-development/mobile-first-bootstrap))

# 使用字体图标

Retina 或高清（HD）显示使屏幕上的所有内容看起来更清晰，更生动。但问题在于在高清显示出现之前带来的传统图像或网络图标。这些图像通常作为位图或光栅图像提供，并且在这个屏幕上变得模糊，如下面的屏幕截图所示：

![使用字体图标](img/image00293.jpeg)

一系列在视网膜显示器上模糊的图标

我们不希望这种情况发生在我们的网站上，因此我们将不得不使用更可伸缩并在高清屏幕上保持清晰的字体图标。

说实话，Bootstrap 附带了一个名为 Glyphicon 的字体图标集。遗憾的是，它没有我们需要的社交媒体图标。在浏览了许多字体图标集之后，我最终选择了 Ionicons（[`ionicons.com/`](http://ionicons.com/)）。在这里，我们将使用由 Lance Hudson 开发的带有 LESS 的替代版本（[`github.com/lancehudson/ionicons-less`](https://github.com/lancehudson/ionicons-less)），因此我们将能够与 Bootstrap 无缝集成，后者也使用 LESS。

# 检查投资组合网站布局

在我们开始构建网站的块和边缘之前，让我们看一下网站线框图。这个线框图将成为参考，并让我们了解网站布局在移动和桌面视图中将如何组织。

![检查投资组合网站布局](img/image00294.jpeg)

上面的屏幕截图显示了桌面或技术上说是宽视口大小的网站布局。

网站将在网站的左上方放置一个按钮，带有所谓的汉堡图标，以滑入画布菜单。然后是网站的第一行，显示网站名称和一行口号。接下来的部分将包含投资组合图像，而最后一部分将包含在线表单和社交媒体图标。

移动视图看起来更简化，但保持了与桌面视图布局相同的逻辑结构，如下面的屏幕截图所示：

![检查投资组合网站布局](img/image00295.jpeg)

# 项目目录、资产和依赖项

让我们通过组织项目目录和包括依赖项、图像和字体图标在内的资产来开始项目。

### 注意

什么是依赖关系？这里的依赖关系是指运行项目和构建网站所需的文件或文件包，如 CSS 和 JavaScript 库。

在这个项目中，我们将实践使用 Bower（[`bower.io/`](http://bower.io/)）来组织项目的依赖关系。Bower，正如我们在第一章中简要提到的那样，*响应式 Web 设计*，是一个前端包管理器，简化了安装、移除和更新前端开发库（如 jQuery、Normalize 和 HTML5Shiv）的方式。

# 进行操作-组织项目目录、资产和使用 Bower 安装项目依赖关系

在本节中，我们将添加包括 Bootstrap、Jasny Bootstrap、Ionicons 和 HTML5Shiv 在内的项目依赖关系。我们将使用 Bower 安装它们，以便将来更轻松地维护它们——移除和更新它们。

此外，由于这可能是您中的许多人第一次使用 Bower，我将以缓慢的速度逐步为您讲解整个过程。请仔细执行以下步骤：

1.  在`htdocs`文件夹中，创建一个新文件夹，并将其命名为`portfolio`。这是项目目录，我们将在其中添加所有项目文件和文件夹。

1.  在`portfolio`文件夹中，创建一个名为`assets`的新文件夹。我们将把项目资产，如图像、JavaScript 和样式表放在这个文件夹中。

1.  在资产文件夹中，创建以下文件夹：

+   `img`用于包含网站图像和基于图像的图标

+   `js`用于包含 JavaScript 文件

+   `fonts`用于包含字体图标集

+   `less`用于包含 LESS 样式表

+   `css`作为 LESS 的输出文件夹

1.  创建`index.html`作为网站的主页。

1.  在`img`文件夹中添加网站的图像；这包括作品集图像和移动设备的图标，如下面的屏幕截图所示：![进行操作-组织项目目录、资产和使用 Bower 安装项目依赖关系](img/image00296.jpeg)

### 注意

这个网站大约有 14 张图片，包括移动设备的图标。我要感谢我的朋友 Yoga Perdana（[`dribbble.com/yoga`](https://dribbble.com/yoga)）允许我在这本书中使用他的精彩作品。您可以在本书中找到这些图像。但是，当然，您也可以用您自己的图像替换它们。

1.  我们将通过 Bower 安装依赖关系——运行项目和构建网站所需的包、库、JavaScript 或 CSS——但在运行任何 Bower 命令来安装依赖关系之前，我们希望使用`bower init`命令将项目设置为 Bower 项目，以定义`bower.json`中的项目规范，如项目名称、版本和作者。

1.  首先，打开终端或命令提示符（如果您使用 Windows）。然后，使用`cd`命令导航到项目目录，如下所示：

+   在 Windows 中：`cd \xampp\htdocs\portfolio`

+   在 OS X 中：`cd /Applications/XAMPP/htdocs/portfolio`

+   在 Ubuntu 中：`cd /opt/lampp/htdocs/portfolio`

1.  输入`bower init`，如下面的屏幕截图所示：![进行操作-组织项目目录、资产和使用 Bower 安装项目依赖关系](img/image00297.jpeg)

### 注意

这个命令`bower init`将我们的项目初始化为一个 Bower 项目。这个命令还会引导我们填写一些提示，以描述项目，比如项目名称、项目版本、作者等。

1.  首先，我们指定项目名称。在这种情况下，我想将项目命名为`responsive-portfolio`。输入名称如下，并按*Enter*继续。请看下面的屏幕截图：![进行操作-组织项目目录、资产和使用 Bower 安装项目依赖关系](img/image00298.jpeg)

1.  指定项目版本。由于项目是新的，让我们简单地将其设置为`1.0.0`，如下面的屏幕截图所示：![进行操作-组织项目目录，资产，并使用 Bower 安装项目依赖项](img/image00299.jpeg)

1.  按下*Enter*键继续。

1.  指定项目描述。此提示完全是可选的。如果您认为对于您的项目不需要，可以将其留空。在这种情况下，我将描述项目为`使用 Bootstrap 构建的响应式作品集网站`，如下面的屏幕截图所示：![进行操作-组织项目目录，资产，并使用 Bower 安装项目依赖项](img/image00300.jpeg)

1.  指定项目的主文件。这肯定会根据项目而变化。在这里，让我们将主文件设置为`index.html`，网站的首页，如下面的屏幕截图所示：![进行操作-组织项目目录，资产，并使用 Bower 安装项目依赖项](img/image00301.jpeg)

1.  这提示了一个问题，“这个软件包暴露了什么类型的模块？”它指定了软件包的用途。在这种情况下，只需选择全局选项，如下面的屏幕截图所示：![进行操作-组织项目目录，资产，并使用 Bower 安装项目依赖项](img/image00302.jpeg)

1.  按空格键选择它，然后按*Enter*继续。

### 注意

此提示描述了项目中的模块技术（我们的项目）的用途。我们的项目没有附属于特定技术或模块；它只是一个纯静态的网站，包括 HTML、CSS 和几行 JavaScript。我们不构建 Node、YUI 或 AMD 模块。因此，最好选择`globals`选项。

1.  **关键字**提示告诉项目的关系。在这种情况下，我想将其填写为`作品集`，`响应式`，`bootstrap`，如下面的屏幕截图所示。按*Enter*继续：![进行操作-组织项目目录，资产，并使用 Bower 安装项目依赖项](img/image00303.jpeg)

**关键字**提示是可选的。如果您愿意，可以将其留空，然后按*Enter*键。

1.  **作者**提示指定了项目的作者。此提示预填了您在系统中注册的计算机用户名和电子邮件。然而，您可以通过指定一个新名称并按*Enter*继续来覆盖它，如下面的屏幕截图所示：![进行操作-组织项目目录，资产，并使用 Bower 安装项目依赖项](img/image00304.jpeg)

### 提示

如果项目有多个作者，您可以使用逗号分隔符指定每个作者，如下所示：

**作者:** `约翰·多, 简·多`。

1.  指定项目的许可证。在这里，我们将简单地将其设置为`MIT`许可证。`MIT`许可证允许任何人对项目中的代码做任何他或她想做的事情，包括修改、转让和商业使用。请看下面的屏幕截图：![进行操作-组织项目目录，资产，并使用 Bower 安装项目依赖项](img/image00305.jpeg)

### 注意

参考选择许可证（[`choosealicense.com/`](http://choosealicense.com/)）以找到其他类型的许可证。

1.  指定项目的主页。这可以是您自己的网站存储库。在这种情况下，我想将其设置为我的个人域名`creatiface.com`，如下面的屏幕截图所示：![进行操作-组织项目目录，资产，并使用 Bower 安装项目依赖项](img/image00306.jpeg)

1.  在**将当前安装的组件设置为依赖项？**命令中，输入`n`（否），因为我们还没有安装任何依赖项或软件包，如下面的屏幕截图所示：![进行操作-组织项目目录，资产，并使用 Bower 安装项目依赖项](img/image00307.jpeg)

1.  **将常见的忽略文件添加到忽略列表中？**命令将创建包含要从 Git 存储库中排除的常见文件列表的`.gitignore`文件。键入`Y`以确认。请查看下面的屏幕截图：![行动时间-组织项目目录，资产，并使用 Bower 安装项目依赖](img/image00308.jpeg)

### 注意

我将使用 Git 来管理代码修订，并将其上传到 Git 存储库，如 Github 或 Bitbucket，因此我选择了`Y`（是）。但是，如果您尚未熟悉 Git，并且不打算在 Git 存储库中托管项目，您可以忽略此提示并键入`n`。 Git 超出了本书讨论的范围。要了解有关 Git 的更多信息，我推荐以下最佳参考资料：

通过 GitTower 学习初学者的 Git ([`www.git-tower.com/learn/`](http://www.git-tower.com/learn/))。

1.  对于**您是否想将此软件包标记为私有，以防止意外发布到注册表中？**命令，键入`Y`，因为我们不会将项目注册到 Bower 注册表中。请查看下面的屏幕截图：![行动时间-组织项目目录，资产，并使用 Bower 安装项目依赖](img/image00309.jpeg)

1.  检查输出。如果看起来不错，请在`bower.json`文件中键入`Y`以生成输出，如下面的屏幕截图所示：![行动时间-组织项目目录，资产，并使用 Bower 安装项目依赖](img/image00310.jpeg)

1.  有许多库我们想要安装。首先，让我们使用`bower install bootstrap ––save`命令安装 Bootstrap，如下面的屏幕截图所示：![行动时间-组织项目目录，资产，并使用 Bower 安装项目依赖](img/image00311.jpeg)

在命令后面的`--save`参数将在`bower.json`中注册 Bootstrap 作为项目依赖项。如果您打开它，您应该会发现它记录在依赖项下，如下面的屏幕截图所示：

![行动时间-组织项目目录，资产，并使用 Bower 安装项目依赖](img/image00312.jpeg)

您还应该在新文件夹`bower_components`中找到保存的 Bootstrap 软件包，以及作为 Bootstrap 依赖项的 jQuery，如下面的屏幕截图所示：

![行动时间-组织项目目录，资产，并使用 Bower 安装项目依赖](img/image00313.jpeg)

1.  使用`bower install jasny-bootstrap –save`命令安装 Bootstrap 扩展 Jasny Bootstrap。

1.  使用`bower install ionicons-less –save`命令安装带有 LESS 样式表的 Ionicons。

1.  Ionicons 软件包附带字体文件。将它们移动到项目目录的`fonts`文件夹中，如下面的屏幕截图所示：![行动时间-组织项目目录，资产，并使用 Bower 安装项目依赖](img/image00314.jpeg)

1.  最后，使用`bower install html5shiv ––save`命令安装 HTML5Shiv，以在 Internet Explorer 8 及以下版本中启用 HTML5 的新元素。

## *刚刚发生了什么？*

我们刚刚创建了文件夹和网站首页文档`index.html`。还准备了将显示在网站上的图像和图标。我们还在`bower.json`中记录了项目规范。通过这个文件，我们可以知道项目的名称是`responsive-portfolio`，当前版本为 1.0.0，并且有一些依赖项，如下所示：

+   Bootstrap ([`github.com/twbs/bootstrap`](https://github.com/twbs/bootstrap))

+   Jasny Bootstrap ([`jasny.github.io/bootstrap/`](http://jasny.github.io/bootstrap/))

+   带有 LESS 的 Ionicons ([`github.com/lancehudson/ionicons-less`](https://github.com/lancehudson/ionicons-less))

+   HTML5Shiv ([`github.com/aFarkas/html5shiv`](https://github.com/aFarkas/html5shiv))

我们通过`bower install`命令下载了这些库，这比下载和解压`.zip`包要简洁。所有的库应该已经添加到一个名为`bower_components`的文件夹中，如下面的截图所示：

![刚刚发生了什么？](img/image00315.jpeg)

## 尝试自定义 Bower 目录

默认情况下，Bower 会创建一个名为`bower_components`的新文件夹。Bower 允许我们通过 Bower 配置文件`.bowerrc`来配置文件夹名称。通过创建`.bowerrc`文件，根据您的喜好更改文件夹名称。请参考此参考链接（[`bower.io/docs/config/`](http://bower.io/docs/config/)）来配置 bower。

## 小测验-测试您对 Bower 命令的理解

Q1\. 我们已经向您展示了如何使用 Bower 安装和更新库。现在的问题是：如何删除已安装的库？

1.  运行`bower remove`命令。

1.  运行`bower uninstall`命令。

1.  运行`bower delete`命令。

Q2\. 除了安装和删除库之外，我们还可以通过 Bower 注册表搜索库的可用性。如何通过 Bower 注册表搜索库？

1.  运行`bower search`，然后跟上关键字。

1.  运行`bower search`，然后跟上库名称。

1.  运行`bower browse`，然后跟上关键字。

Q3\. Bower 还允许我们查看包属性的详细信息，例如包版本、依赖关系、作者等。我们执行什么命令来查看这些详细信息？

1.  `bower info`。

1.  `bower detail`。

1.  `bower property`。

## 更新 Bower 组件

由于依赖项是通过 Bower 安装的，因此项目的维护将更加简化。这些库可以在以后更新到新版本。通过使用 Bower 命令，更新我们刚刚安装的库实际上比下载`.zip`包并手动将文件移动到项目目录中更加简洁。

运行`bower list`命令以查看所有已安装的 Bower 包，并检查包的新版本是否可用，如下面的截图所示：

![更新 Bower 组件](img/image00316.jpeg)

然后，使用`bower install`命令安装新版本，后面跟着 Bower 包名称和版本号。例如，要安装 Bootstrap 版本 3.2.0，运行`bower install bootstrap#3.2.0 ––save`命令。

### 注意

实际上，我们应该能够使用`bower update`命令来更新包。然而，根据 Bower Issue 线程中的一些报告，这个命令似乎并不按预期工作（[`github.com/bower/bower/issues/1054`](https://github.com/bower/bower/issues/1054)）。因此，目前使用先前展示的`bower install`命令是正确的方法。

# 作品集网站 HTML 结构

现在我们已经准备好构建网站所需的基本内容。让我们开始构建网站的 HTML 结构。与上一个项目一样，在这里，我们将使用一些新的 HTML5 元素来构建语义结构。

# 行动时间-构建网站 HTML 结构

在这一部分，我们将构建网站的 HTML 结构。您会发现，我们将在这里添加的一些元素与我们在第一个网站中添加的元素相似。因此，以下步骤将是直接的。如果您已经从头到尾地跟随了第一个项目，那么这些步骤也应该很容易跟随。让我们继续。

1.  打开`index.html`。然后，添加基本的 HTML 结构，如下所示：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Portfolio</title>
</head>
<body>

</body>
</html>
```

1.  在`<meta charset="UTF-8">`下面，添加一个 meta 标签来解决 Internet Explorer 的渲染兼容性问题：

```html
<meta http-equiv="X-UA-Compatible" content="IE=edge">
```

前面的 meta 标签规范将强制 Internet Explorer 使用其中的最新引擎版本来渲染页面。

### 注意

有关`X-UA-Compatible`的更多信息，请参考 Modern.IE 文章，*如何使用 X-UA-Compatible*（[`www.modern.ie/en-us/performance/how-to-use-x-ua-compatible`](https://www.modern.ie/en-us/performance/how-to-use-x-ua-compatible)）。

1.  在`http-equiv`meta 标记下方，添加 meta 视口标记：

```html
<meta name="viewport" content="width=device-width, initial-scale=1">
```

前面的视口 meta 标记规定了网页视口宽度要遵循设备视口大小，并在首次打开网页时以 1:1 的比例缩放页面。

1.  在视口 meta 标记下方，添加到 favicon 和 apple-touch-icon 的链接，这将在苹果设备（如 iPhone、iPad 和 iPod）上显示网站的图标：

```html
<link rel="apple-touch-icon" href="assets/img/apple-icon.png">
<link rel="shortcut icon" href="assets/img/favicon.png" type="image/png">
```

1.  在`<title>`下方添加网站的 meta 描述：

```html
<meta name="description" content="A simple portoflio website built using Bootstrap">
```

在这个 meta 标记中指定的描述将显示在**搜索引擎结果页面**（**SERP**）中。

1.  您还可以按照以下方式在 meta 描述标记下方指定页面的作者。

```html
<meta name="author" content="Thoriq Firdaus">
```

1.  在`<body>`内，按照以下方式添加网站的侧栏导航 HTML：

```html
<nav id="menu" class="navmenu navmenu-inverse navmenu-fixed-left offcanvas portfolio-menu" role="navigation">
        <ul class="nav navmenu-nav">
            <li class="active"><a href="#">Home</a></li>
            <li><a href="#">Blog</a></li>
            <li><a href="#">Shop</a></li>
            <li><a href="#">Speaking</a></li>
            <li><a href="#">Writing</a></li>
            <li><a href="#">About</a></li>
        </ul>
    </nav>
```

除了本章中 Jasny Bootstrap 侧栏部分提到的基本类之外，我们还在`<nav>`元素中添加了一个名为`portfolio-menu`的新类，以应用我们自己的样式到侧栏导航。

1.  添加 Bootstrap `navbar`结构，以及用于滑动侧栏的`<button>`：

```html
<div class="navbar navbar-default navbar-portfolio portfolio-topbar">
<button type="button" class="navbar-toggle" data-toggle="offcanvas" data-target="#menu" data-canvas="body">
        <span class="icon-bar"></span>
<span class="icon-bar"></span>
<span class="icon-bar"></span>
</button>
</div>
```

1.  在“导航栏”下方，添加`<main>`元素，如下所示：

```html
<main class="portfolio-main" id="content" role="main">
</main>
```

正如 W3C（[`www.w3.org/TR/html-main-element/`](http://www.w3.org/TR/html-main-element/)）中所描述的，`<main>`元素定义了网站的主要内容。因此，这就是我们将放置网站内容的地方，包括作品集图片。

1.  添加 Bootstrap Jumbotron，包含作品集网站名称和一行标语。由于我将展示一个朋友的作品，Yoga Perdana 的作品，我希望展示他的名字，以及他在 Dribbble 页面个人资料中显示的标语（[`dribbble.com/yoga`](https://dribbble.com/yoga)），如下所示：

```html
<main class="portfolio-main" id="content" role="main">
<section class="jumbotron portfolio-about" id="about">
<h1 class="portfolio-name">Yoga Perdana</h1>
<p class="lead">Illustrator &amp; Logo designer. I work using digital tools, specially vector.</p>
</section>
</main>
```

您可以自由地在此处添加您的姓名或公司名称。

1.  在 Bootstrap Jumbotron 部分下方，使用 HTML5 `<section>`元素添加一个新的部分，并包含定义此部分的标题，如下所示：

```html
...
<section class="jumbotron portfolio-about" id="about">
<h1 class="portfolio-name">Yoga Perdana</h1>
<p class="lead">Illustrator &amp; Logo designer. I work using digital tools, specially vector.</p>
</section>
<section class="portfolio-display" id="portfolio">
  <h2>Portfolio</h2>
</section>
```

1.  在包含以下代码的标题下方添加 Bootstrap 容器（[`getbootstrap.com/css/#overview-container`](http://getbootstrap.com/css/#overview-container)），该容器将包含作品集图片：

```html
<section class="portfolio-display" id="portfolio">
<h2>Portfolio</h2>
   <div class="container">
</div>
</section>
```

1.  将作品集图片排列成列和行。我们有 12 张作品集图片，这意味着我们可以在一行中有四张图片/列。以下是第一行：

```html
...
<div class="container">
<div class="row">
<div class="col-md-3 col-sm-6 portfolio-item">
 <figure class="portfolio-image">
<img class="img-responsive" src="img/6layers.jpg" height="300" width="400" alt="">
<figcaption class="portfolio-caption">6 Layers</figcaption>
 </figure>
 </div>
<div class="col-md-3 col-sm-6 portfolio-item">
 <figure class="portfolio-image">
<img class="img-responsive" src="img/blur.jpg" height="300" width="400" alt="">
<figcaption class="portfolio-caption">Blur</figcaption>
</figure>
 </div>
<div class="col-md-3 col-sm-6 portfolio-item">
 <figure class="portfolio-image">
<img class="img-responsive" src="img/brain.jpg" height="300" width="400" alt="">
<figcaption class="portfolio-caption">Brain</figcaption>
</figure>
 </div>
 <div class="col-md-3 col-sm-6 portfolio-item">
 <figure class="portfolio-image">
<img class="img-responsive" src="img/color.jpg" height="300" width="400" alt="">
<figcaption class="portfolio-caption">Color</figcaption>
</figure>
 </div>
</div>
</div>
```

每列都分配了一个特殊的类，以便我们可以应用自定义样式。我们还在包裹图片的`<figure>`中添加了一个类，以及包裹图片标题的`<figcaption>`元素，以达到同样的目的。

1.  将剩余的图片添加到列和行中。在这种情况下，我们有 12 张图片，因此网站上应该显示三行。每行包含四张图片，包括我们在第 13 步中添加的一行。

1.  在作品集部分下方，添加包含三个表单字段和一个按钮的网站留言表单，如下所示：

```html
...
</section>
<div class="portfolio-contact" id="contact">
 <div class="container">
 <h2>Get in Touch</h2>
<form id="contact" method="post" class="form" role="form">
 <div class="form-group">
<input type="text" class="form-control input-lg" id="input-name" placeholder="Name">
</div>
 <div class="form-group">
<input type="email" class="form-control input-lg" id="input-email" placeholder="Email">
 </div>
 <div class="form-group">
<textarea class="form-control" rows="10"></textarea>
 </div>
 <button type="submit" class="btn btn-lg btn-primary">Submit</button>
 </form>
</div>
</div>

```

在这里，我们只用了三个表单字段来简化网站表单。但是，您可以根据自己的需求添加额外的表单字段。

1.  最后，我们将使用 HTML5 `<footer>`元素添加网站页脚。页脚，正如我们从网站线框图中看到的那样，包含社交媒体图标和网站版权声明。

1.  在网站的主要内容下方添加以下 HTML 标记：

```html
... 
</main>
<footer class="portfolio-footer" id="footer">
        <div class="container">
          <div class="social" id="social">
            <ul>
<li class="twitter"><a class="icon ion-social-twitter" href="#">Twitter</a></li>
<li class="dribbble"><a class="icon ion-social-dribbble-outline" href="#">Dribbble</a></li>
                </ul>
          </div>
<div class="copyright">Yoga Perdana &copy; 2014</div>
        </div>
    </footer>
```

## *刚刚发生了什么？*

我们刚刚使用了一些 HTML5 元素和 Bootstrap 可重用类构建了投资组合网站的 HTML 结构。您应该能够通过以下地址`http://localhost/portfolio/`或`http://{computer-username}/portfolio/`来查看网站，如果您使用的是 OS X。在这个阶段，网站还没有应用任何样式；我们还没有在页面中链接任何样式表。因此，接下来的提示后面的屏幕截图是网站当前的外观。

### 提示

在前面步骤中显示的完整代码也可以从以下 Gist [`git.io/oIh31w`](http://git.io/oIh31w) 获取。

![刚刚发生了什么？](img/image00317.jpeg)

## 尝试一下 - 扩展投资组合网站

Bootstrap 提供了各种组件。然而，我们只使用了一些，包括网格、Jumbotron、按钮和表单。通过添加额外的 Bootstrap 组件来扩展网站，如下所示：

+   分页（[`getbootstrap.com/components/#pagination`](http://getbootstrap.com/components/#pagination)）

+   面包屑（[`getbootstrap.com/components/#breadcrumbs`](http://getbootstrap.com/components/#breadcrumbs)）

+   响应式嵌入（[`getbootstrap.com/components/#responsive-embed`](http://getbootstrap.com/components/#responsive-embed)）

+   面板（[`getbootstrap.com/components/#panels`](http://getbootstrap.com/components/#panels)）

+   Wells（[`getbootstrap.com/components/#wells`](http://getbootstrap.com/components/#wells)）

此外，尝试创建更多的网页，并通过侧栏导航菜单将它们链接起来。

## 弹出测验 - Bootstrap 按钮类

Bootstrap 指定了许多可重用类，可以快速地为元素设置预设样式。

Q1. 以下哪个类在 Bootstrap 网格中没有被使用？

1.  `col-sm-pull-8`

1.  `col-md-push-3`

1.  `col-xs-offset-5`

1.  `col-lg-6`

1.  `col-xl-7`

Q2. 以下哪个类用于样式化按钮？

1.  `btn-link`

1.  `btn-submit`

1.  `btn-send`

1.  `btn-cancel`

1.  `btn-enter`

# 摘要

本章开始了本书的第二个项目。我们正在使用最流行的前端开发框架之一 Bootstrap 构建一个投资组合网站。我们还探索了一个名为 Bower 的新的引人注目的网页开发工具，它简化了网站依赖管理。

它们都是工具的绝佳组合。Bootstrap 让我们可以快速使用模块化组件和可重用类构建响应式网站，而 Bower 使项目更易于维护。

在下一章中，我们将更多地处理 LESS 和 JavaScript 来装饰网站。
