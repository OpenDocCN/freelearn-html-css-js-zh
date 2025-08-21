# 第七章：使用 Foundation 为企业构建响应式网站

在这个时代，许多人都与互联网连接，拥有一个网站对于任何规模的公司都变得至关重要——无论是小公司还是拥有数十亿美元业务的财富 500 强公司。因此，在本书的第三个项目中，我们将为企业构建一个响应式网站。

构建网站时，我们将采用一个名为 Foundation 的新框架。Foundation 是由总部位于加利福尼亚的 Web 开发机构 ZURB 开发的。它是一个精心打造的框架，具有一系列交互式小部件。在技术方面，Foundation 样式是建立在 Sass 和 SCSS 之上的。因此，在项目进行过程中，我们也将学习这个主题。

为了开展这个项目，首先让我们假设你有一个商业理念。这可能有点夸张，但这是一个可能会变成数十亿美元业务并改变世界的杰出理念。你有一个很棒的产品，现在是建立网站的时候了。你非常兴奋，迫不及待地想要改变世界。

所以，话不多说，让我们开始这个项目。

本章将主要围绕 Foundation 展开，我们将在此讨论的主题包括：

+   在线框架中检查网站设计和布局

+   研究 Foundation 的特性、组件和附加组件

+   管理项目目录和资产

+   通过 Bower 获取 Foundation 包

+   构建网站 HTML 结构

# 检查网站布局

首先，与我们之前做的两个项目不同，我们将在本章中进一步研究网站布局的线框图。在检查完之后，我们将发现网站所需的 Foundation 组件，以及 Foundation 包中可能不包含的组件和资产。以下是正常桌面屏幕尺寸下的网站布局：

![检查网站布局](img/image00342.jpeg)

前面的线框图显示，网站将有五个部分。第一部分，显而易见的是页眉。页眉部分将包含网站标志、菜单导航、几行标语和一个按钮——许多人称之为行动号召按钮。

### 注意

以下是关于行动号召按钮的指南、最佳实践和示例的一些参考资料。这些是旧的帖子，但其中的指南、技巧和原则是永恒的；至今仍然有效和相关。

+   Call to Action Buttons: Examples and Best Practices ([`www.smashingmagazine.com/2009/10/13/call-to-action-buttons-examples-and-best-practices/`](http://www.smashingmagazine.com/2009/10/13/call-to-action-buttons-examples-and-best-practices/)).

+   "Call To Action" Buttons: Guidelines, Best Practices And Examples ([`www.hongkiat.com/blog/call-to-action-buttons-guidelines-best-practices-and-examples/`](http://www.hongkiat.com/blog/call-to-action-buttons-guidelines-best-practices-and-examples/)).

+   How To Design Call to Action Buttons That Convert ([`unbounce.com/conversion-rate-optimization/design-call-to-action-buttons/`](http://unbounce.com/conversion-rate-optimization/design-call-to-action-buttons/)).

通常，人们在决定购买之前需要尽可能多地了解优缺点。因此，在页眉下，我们将显示产品的项目列表或提供的关键功能。

除了功能列表，我们还将在滑块中显示客户的推荐。根据[www.entrepreneur.com](http://www.entrepreneur.com) ([`www.entrepreneur.com/article/83752`](http://www.entrepreneur.com/article/83752))，显示客户的推荐是推动更多客户或销售的有效方式之一，这对业务最终是有利的。

在推荐部分下方，网站将显示计划和价格表。最后一个部分将是包含次要网站导航和指向 Facebook 和 Twitter 的链接的页脚。

现在让我们看看网站在较小的视口大小下的布局，如下所示：

![检查网站布局](img/image00343.jpeg)

与我们在之前的项目中构建的网站类似，所有内容都将被堆叠。口号和行动号召按钮都居中对齐。导航中的菜单现在被描述为汉堡图标。接下来，我们将看看 Foundation 在其套件中提供了什么来构建网站。

# 深入了解 Foundation

Foundation ([`foundation.zurb.com/`](http://foundation.zurb.com/))是最流行的前端开发框架之一。它被许多知名公司使用，如 Pixar、华盛顿邮报、Warby Parker ([`www.warbyparker.com/`](https://www.warbyparker.com/))等。正如前面提到的，Foundation 附带了常见的网页组件和交互式小部件。在这里，我们将研究我们将用于网站的组件和小部件。

## 网格系统

网格系统是框架的一个组成部分。它是使管理网页布局感觉轻松的一件事。Foundation 的网格系统包括可以通过提供的类适应窄视口大小的十二列。与我们在前几章中探讨的两个框架类似，网格由行和列组成。每个列都必须包含在行内，以使布局正确跨越。

在 Foundation 中，应用`row`类来定义一个元素作为行，并将元素应用`columns`或`column`类来定义为列。例如：

```html
<div class="row">
<div class="columns">
</div>
<div class="columns">
</div>
</div>
```

您还可以从`columns`中省略*s*，如下所示：

```html
<div class="row">
<div class="column">
</div>
<div class="column">
</div>
</div>
```

列的大小通过以下一系列类来定义：

+   `small-{n}`：这指定了小视口大小范围（大约 0 像素-640 像素）中的网格列宽度。

+   `medium-{n}`：这指定了中等视口大小范围（大约 641 像素-1,024 像素）中的网格列宽度。

+   `large-{n}`：这指定了大视口大小范围（大约 1,025 像素-1,440 像素）中的网格列宽度。

### 注意

我们在前面的类名中给出的`{n}`变量代表从`1`到`12`的数字。一行中的列数之和不应超过`12`。

这些类可以在单个元素中结合应用。例如：

```html
<div class="row">
<div class="small-6 medium-4 columns"></div>
<div class="small-6 medium-8 columns"></div>
</div>
```

上面的示例在浏览器中产生以下结果：

![网格系统](img/image00344.jpeg)

调整视口大小，使其足够小，以便列的宽度根据分配的类进行调整。在这种情况下，每个列的宽度相等，因为它们都分配了`small-6`类：

![网格系统](img/image00345.jpeg)

### 注意

通常，您可以通过拖动浏览器窗口来调整视口大小。如果您使用 Chrome，可以激活设备模式和移动模拟器([`developer.chrome.com/devtools/docs/device-mode`](https://developer.chrome.com/devtools/docs/device-mode))。或者，如果您使用 Firefox，可以启用响应式设计视图([`developer.mozilla.org/en-US/docs/Tools/Responsive_Design_View`](https://developer.mozilla.org/en-US/docs/Tools/Responsive_Design_View))，这将允许您调整视口大小，而无需拖动 Firefox 窗口。

## 按钮

按钮对于任何类型的网站都是必不可少的，我们肯定会在网站的某些地方添加按钮。Foundation 使用`button`类来定义一个元素作为按钮。您可以将该类分配给元素，例如`<a>`和`<button>`。该类应用了默认的按钮样式，如下面的屏幕截图所示：

![按钮](img/image00346.jpeg)

此外，您可以包含其他类来定义按钮的颜色或上下文。使用`secondary`、`success`、`alert`中的一个类来设置按钮颜色：

![按钮](img/image00347.jpeg)

您还可以使用`tiny`、`small`或`large`中的一个类来指定按钮大小：

![按钮](img/image00348.jpeg)

使用`radius`和`round`中的一个类使按钮更漂亮，带有圆角：

![按钮](img/image00349.jpeg)

### 注

还有一些类来形成按钮。此外，Foundation 还提供多种类型的按钮，如按钮组、分割按钮和下拉按钮。因此，您可以转到 Foundation 文档的**按钮**部分，了解更多信息。

## 导航和顶部栏

网站上一个重要的部分是导航。导航帮助用户从一个页面浏览到另一个页面。在这种情况下，Foundation 提供了几种导航类型，其中之一称为顶部栏。Foundation 的顶部栏将位于网站的顶部，任何内容或部分之前。以下是 Foundation 默认样式下顶部栏的外观：

![导航和顶部栏](img/image00350.jpeg)

顶部栏是响应式的。尝试调整浏览器的视口大小，使其足够小，您会发现导航隐藏在菜单中，需要点击**菜单**才能显示完整的菜单项列表：

![导航和顶部栏](img/image00351.jpeg)

Foundation 顶部栏主要由`top-bar`类形成应用样式，`data-topbar`属性运行与顶部栏相关的 JavaScript 函数，最后`role="navigation"`以提高可访问性。

因此，以下代码是我们开始在 Foundation 中构建顶部栏的方式：

```html
<nav class="top-bar" data-topbar role="navigation">      
  ...
</nav>
```

Foundation 将顶部栏内容分为两个部分。左侧区域称为标题区域，包括网站名称或标志。Foundation 使用列表元素构建此部分，如下所示：

```html
<ul class="title-area">
<li class="name">
    <h1><a href="#">Hello</a></h1>
  </li>
  <li class="toggle-topbar menu-icon">
<a href="#"><span>Menu</span></a>
</li>
</ul>
```

第二部分简称为顶部栏部分。通常，此部分包含菜单、按钮和搜索表单。Foundation 使用`top-bar-section`类设置此部分，以及`left`和`right`类来指定对齐方式。因此，将所有内容放在一起，以下是构建基本 Foundation 顶部栏的完整代码，如前面的屏幕截图所示：

```html
<nav class="top-bar" data-topbar role="navigation">
  <ul class="title-area">
    <li class="name">
      <h1><a href="#">Hello</a></h1>
    </li>
    <li class="toggle-topbar menu-icon">
<a href="#"><span>Menu</span></a>
</li>
  </ul>
<section class="top-bar-section">
    <ul class="right">
      <li class="active"><a href="#">Home</a></li>
      <li><a href="#">Blog</a></li>
      <li><a href="#">About</a></li>
      <li><a href="#">Contact</a></li>
    </ul>
  </section>
</nav>
```

当然，您需要在文档中预先链接 Foundation CSS 样式表，以查看顶部栏的外观。

## 价格表

无论您是销售产品还是服务，您都应该命名您的价格。

由于我们将为企业构建网站，因此需要显示价格表。幸运的是，Foundation 已经将此组件包含在其核心中，因此我们不需要第三方扩展。为了灵活性，Foundation 使用列表元素结构化价格表，如下所示：

```html
<ul class="pricing-table pricing-basic">
   <li class="title">Basic</li>
   <li class="price">$10<small>/month</small></li>
   <li class="description">Perfect for personal use.</li>
   <li class="bullet-item">1GB Storage</li>
   <li class="bullet-item">1 User</li>
   <li class="bullet-item">24/7 Support</li>
<li class="cta-button">
<a class="button success round" href="#">Sign Up</a>
</li>
</ul>
```

列表中的每个项目都使用一个类进行设置，我相信类名已经解释了它自己。鉴于前面的 HTML 结构和 Foundation CSS 提供的默认样式，输出结果非常好，如下面的屏幕截图所示：

![价格表](img/image00352.jpeg)

## 在 Orbit 中移动

轮播或滑块是网络上最流行的设计模式之一。尽管在可访问性方面存在争议，但许多人仍然喜欢在他们的网站上使用它，我们也是如此。在这里，我们希望使用 Orbit ([`foundation.zurb.com/docs/components/orbit.html`](http://foundation.zurb.com/docs/components/orbit.html))，Foundation jQuery 插件来显示内容滑块。

Orbit 是可定制的，我们可以完全控制输出，以及通过类、属性或 JavaScript 初始化幻灯片的行为。我们还可以在 Orbit 幻灯片中添加几乎任何内容，包括文本内容、图像、链接和混合内容。不用说，我们可以为其大部分部分设置样式。

### 轨道是如何构建的？

Foundation 使用`list`元素来构建幻灯片容器，以及幻灯片，并使用 HTML5 的`data-`属性`data-orbit`来启动功能。以下是 Orbit 滑块结构的基本示例，包含两张图片幻灯片：

```html
<ul class="example-orbit" data-orbit>
<li><img src="img/image.jpg" alt="" /></li>
<li class="active"><img src="img/image2.jpg" alt="" /></li>
</ul>
```

部署 Orbit 非常简单，从技术上讲，它几乎可以包含任何类型的内容在幻灯片中，而不仅仅是图片。随着我们构建网站，我们将更多地关注这一点。

### 注意

目前，可以随意探索 Foundation 官方网站的 Orbit 滑块部分（[`foundation.zurb.com/docs/components/orbit.html`](http://foundation.zurb.com/docs/components/orbit.html)），在我看来，这是了解 Orbit 滑块的最佳地方。

## 添加附加组件，字体图标

Foundation 还提供了一些附加组件，其中之一是 Webicons（[`zurb.com/playground/social-webicons`](http://zurb.com/playground/social-webicons)）。毋庸置疑，我们需要社交图标，由于这些图标基本上是矢量图，因此在任何屏幕分辨率（普通或高清）下都可以无限缩放，因此将保持清晰和锐利。请查看以下图标集：

![添加附加组件，字体图标](img/image00353.jpeg)

图标集中的一些字形

除了这个图标集，您还可以找到以下内容：

+   一系列起始模板（[`foundation.zurb.com/templates.html`](http://foundation.zurb.com/templates.html)），这些模板对于启动新网站和网页非常有用。

+   响应式表格（[`foundation.zurb.com/responsive-tables.html`](http://foundation.zurb.com/responsive-tables.html)）

+   模板（[`foundation.zurb.com/stencils.html`](http://foundation.zurb.com/stencils.html)），您会发现对于勾画和原型设计新网站很有用

## 关于 Foundation 的更多信息

详细介绍 Foundation 的每个角落和方面超出了本书的范围。这些是我们将在项目和网站中使用的框架的最基本组件。

幸运的是，Packt Publishing 出版了几本专门介绍 Foundation 的书籍。如果您有兴趣进一步探索这个框架，我建议您看一下以下书籍：

+   *学习 Zurb Foundation*，*Kevin Horek*，*Packt Publishing*（[`www.packtpub.com/web-development/learning-zurb-foundation`](https://www.packtpub.com/web-development/learning-zurb-foundation)）

+   *ZURB Foundation Blueprints*，*James Michael Stone*，*Packt Publishing*（[`www.packtpub.com/web-development/zurb-foundation-blueprints`](https://www.packtpub.com/web-development/zurb-foundation-blueprints)）

# 额外所需资产

除了 Foundation 自己的组件之外，我们还需要一些文件。这些文件包括网站页眉的图像封面，将代表网站功能列表部分的图标，favicon 图像以及 Apple 图标，显示在推荐部分的头像图像，以及最后（也很重要）网站标志。

在页眉图像方面，我们将使用 Alejandro Escamilla 拍摄的以下图像，该图像显示了一名男子正在使用他的 Macbook Air；尽管屏幕似乎关闭了（[`unsplash.com/post/51493972685/download-by-alejandro-escamilla`](http://unsplash.com/post/51493972685/download-by-alejandro-escamilla)）：

![额外所需资产](img/image00354.jpeg)

用于显示功能列表项旁边的图标是由 Ballicons 的 Nick Frost 设计的（[`ballicons.net/`](http://ballicons.net/)）。我们将在网站中包含该系列中的以下图标：

![额外所需资产](img/image00355.jpeg)

以下是使用 Photoshop 动作 AppIconTemplate 生成的 favicon 和 Apple 图标：

![额外所需资产](img/image00356.jpeg)

Favicon 和 Apple 图标

我们将使用 WordPress 的神秘人作为默认头像。此头像图像将显示在推荐语句上方，如下线框所示：

![额外所需资产](img/image00357.jpeg)

神秘人

该网站的标志是用 SVG 制作的，以确保清晰度和可伸缩性。标志显示在以下截图中：

![额外所需资产](img/image00358.jpeg)

您可以从随本书提供的源文件中获取所有这些资产。否则，可以从我们在前面段落中显示的 URL 中获取它们。

# 项目目录、资产和依赖项

一旦我们评估了网站布局、框架功能和所有所需的资产，我们将开始着手项目。在这里，我们将开始整理项目目录和资产。此外，我们将通过 Bower 获取并记录所有项目依赖项，第二个项目使用 Bootstrap。所以，是时候行动了。

# 行动时间-组织项目目录、资产和依赖项

1.  在`htdocs`文件夹中，创建一个新文件夹，命名为`startup`。这是网站将驻留的文件夹。

1.  在`startup`文件夹中，创建一个名为`assets`的文件夹，以包含所有资产，如样式表、JavaScript 文件、图像等。

1.  在`assets`文件夹内创建文件夹以对这些资产进行分组：

+   `css`用于样式表。

+   `js`包含 JavaScript 文件。

+   `scss`包含 SCSS 样式表（关于 SCSS 的更多内容请参见下一章）。

+   `img`包含图像。

+   `fonts`包含字体图标。

1.  添加图像，包括网站标志、页眉图像、图标和头像图像，如下截图所示：![行动时间-组织项目目录、资产和依赖项](img/image00359.jpeg)

1.  现在，我们将下载项目依赖项，其中包括 Foundation 框架、图标、jQuery 和其他几个库。因此，让我们打开终端或命令提示符（如果您使用 Windows）。然后，使用`cd`命令导航到项目目录：

+   在 Windows 中：`cd \xampp\htdocs\startup`

+   在 OSX 中：`cd /Applications/XAMPP/htdocs/startup`

+   在 Ubuntu 中：`cd /opt/lampp/htdocs/startup`

1.  与第二个项目一样，键入命令，填写提示以设置项目规范，包括项目名称和项目版本，如下截图所示：![行动时间-组织项目目录、资产和依赖项](img/image00360.jpeg)

当所有提示都填写并完成时，Bower 将生成一个名为`bower.json`的新文件，以存放所有信息。

1.  在安装项目依赖项之前，我们将设置依赖项文件夹的目标位置。为此，请创建一个名为`.bowerrc`的点文件。将以下行保存在文件中：

```html
{
  "directory": "components"
}
```

这行将告诉 Bower 将文件夹命名为`components`而不是`bower_components`。一旦配置设置完成，我们就可以安装库，首先安装 Foundation 包。

1.  要通过 Bower 安装 Foundation 包，请键入`bower install foundation ––save`。确保包括`--save`参数以记录 Foundation 在`bower.json`文件中。

### 注意

除了 Foundation 主要包（例如样式表和 JavaScript 文件）外，此命令还将获取与 Foundation 相关的库，即：

Fastclick ([`github.com/ftlabs/fastclick`](https://github.com/ftlabs/fastclick))

jQuery ([`jquery.com/`](http://jquery.com/))

jQuery Cookie ([`github.com/carhartl/jquery-cookie`](https://github.com/carhartl/jquery-cookie))

jQuery Placeholder ([`github.com/mathiasbynens/jquery-placeholder`](https://github.com/mathiasbynens/jquery-placeholder))

Modernizr ([`modernizr.com/`](http://modernizr.com/))

1.  Foundation 字体图标设置在一个单独的存储库中。要安装它，请键入`bower install foundation-icons --save`命令。

1.  Foundation 图标包带有样式表，通过 HTML 类指定和呈现图标文件。在这里，我们需要将包文件夹中的字体复制到我们自己的`fonts`文件夹中。请看下面的屏幕截图：![行动时间-组织项目目录、资产和依赖关系](img/image00361.jpeg)

## *刚刚发生了什么？*

我们刚刚创建了项目目录，以及用于组织项目资产的文件夹。此外，我们还通过 Bower 安装了构建网站所需的库，其中包括 Foundation 框架。

在添加了图像和库之后，我们将在下一节中构建网站的主页标记。因此，不用再多说，让我们继续前进，再次行动起来。

# 行动时间-构建网站的 HTML 结构

1.  创建一个名为`index.html`的新 HTML 文件。然后，在 Sublime Text 中打开它，这是本书中我们选择的代码编辑器。

1.  让我们按照以下方式添加基本的 HTML5 结构：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Startup</title>
</head>
<body>

</body>
</html>
```

1.  添加 meta`X-UA-Compatible`变量，内容值为`IE=edge`，以允许 Internet Explorer 使用其最新的渲染版本：

```html
<meta http-equiv="X-UA-Compatible" content="IE=edge">
```

1.  不要忘记 meta`viewport`标签，以使网站响应式；将其添加到`<head>`中，如下所示：

```html
<meta name="viewport" content="width=device-width, initial-scale=1">
```

1.  在 meta 视口标签下方添加网站图标以及苹果图标，如下所示：

```html
<link rel="apple-touch-icon" href="assets/img/apple-icon.png">
<link rel="shortcut icon" href="assets/img/favicon.png" type="image/png">
```

1.  为了搜索引擎结果的目的，添加 meta 描述：

```html
<meta name="description" content="A startup company website built using Foundation">
```

1.  内容的 HTML 标记将遵循本书早期部分讨论的 Foundation 指南。此外，我们可能会在元素中添加额外的类来自定义样式。让我们从添加网站`<header>`开始，为此在`<body>`中添加以下行：

```html
<header class="startup-header">
...
</header>
```

1.  接下来，在标题中添加网站导航栏，如下所示：

```html
<header class="startup-header">
<div class="contain-to-grid startup-top-bar">
<nav class="top-bar" data-topbar>
 <ul class="title-area">
 <li class="name startup-name">
 <h1><a href="#">Startup</a></h1>
 </li>
<li class="toggle-topbar menu-icon">
 <a href="#"><span>Menu</span></a>
</li>
</ul>
 <section class="top-bar-section">
 <ul class="right">
 <li><a href="#">Features</a></li>
<li><a href="#">Pricing</a></li>
<li><a href="#">Blog</a></li>
<li class="has-form log-in"><a href="" class="button secondary round">Log In</a></li>
<li class="has-form sign-up"><a href="#" class="button round">Sign Up</a></li>
 </ul>
</section>
</nav>
</div>
</header>
```

1.  在导航栏 HTML 标记下方，按照以下方式添加标语和行动号召按钮：

```html
<header class="startup-header"> 
  ...
<div class="panel startup-hero">
 <div class="row">
<h2 class="hero-title">Stay Cool and be Awesome</h2>
<p class="hero-lead">The most awesome web application in the galaxy.</p>
</div>
 <div class="row">
<a href="#" class="button success round">Signup</a>
 </div>
</div>
</header>
```

1.  接下来，我们将添加包含产品功能列表部分、推荐部分和计划价格表的网站正文内容。首先，在标题下方添加一个包裹正文内容部分的`<div>`，如下所示：

```html
<div class="startup-body">
  ...
</div>
```

1.  在`<div>`中，按照以下方式添加功能列表部分的 HTML 标记：

```html
<div class="startup-body">
<div class="startup-features">
<div class="row">
 <div class="medium-6 columns">
 <div class="row">
 <div class="small-3 medium-4 columns">
 <figure>
<img src="img/analytics.png" height="128" width="128" alt="">
 </figure>
</div>
 <div class="small-9 medium-8 columns">
 <h4>Easy</h4>
<p>This web application is super easy to use. No complicated setup. It just works out of the box.</p>
 </div>
 </div>
 </div>
 <div class="medium-6 columns">
 <div class="row">
<div class="small-3 medium-4 columns">
 <figure>
 <img src="img/clock.png" height="128" width="128" alt="">
 </figure>
 </div>
 <div class="small-9 medium-8 columns">
 <h4>Fast</h4>
 <p>This web application runs in a blink of eye. There is no other application that is on par with our application in term of speed.</p>
 </div>
 </div>
 </div>
 </div>
 <div class="row">
 <div class="medium-6 columns">
 <div class="row">
<div class="small-3 medium-4 columns">
 <figure>
<img src="img/target.png" height="128" width="128" alt="">
</figure>
 </div>
<div class="small-9 medium-8 columns">
 <h4>Secure</h4>
<p>Your data is encyrpted with the latest Kryptonian technology. It will never be shared to anyone. Rest assured, your data is totally safe.</p>
 </div>
 </div>
 </div>
 <div class="medium-6 columns">
 <div class="row">
 <div class="small-3 medium-4 columns">
 <figure>
 <img src="img/bubbles.png" height="128" width="128" alt="">
 </figure>
 </div>
 <div class="small-9 medium-8 columns">
 <h4>Awesome</h4>
 <p>It's simply the most awesome web application and make you the coolest person in the galaxy. Enough said.</p>
 </div>
</div>
 </div>
 </div>
</div>
</div> 
```

此部分的列划分是指网站线框图中显示的布局。因此，正如您从刚刚添加的代码中看到的那样，每个功能列表项都分配了`medium-6`列，因此每个项目的列宽将相等。

1.  在功能列表部分下方，我们按照以下方式添加了推荐部分的 HTML 标记：

```html
<div class="startup-body">
...
<div class="startup-testimonial">
 <div class="row">
 <ul class="testimonial-list" data-orbit>
 <li data-orbit-slide="testimonial-1">
 <div>
 <blockquote>Lorem ipsum dolor sit amet, consectetur adipisicing elit. Dolor numquam quaerat doloremque in quis dolore enim modi cumque eligendi eius.</blockquote>
 <figure>
 <img class="avatar" src="img/mystery.png" height="128" width="128" alt="">
 <figcaption>John Doe</figcaption>
 </figure>
 </div>
 </li>
 <li data-orbit-slide="testimonial-2">
 <div>
 <blockquote>Lorem ipsum dolor sit amet, consectetur adipisicing elit.</blockquote>
 <figure>
 <img class="avatar" src="img/mystery.png" height="128" width="128" alt="">
 <figcaption>Jane Doe</figcaption>
 </figure>
 </div>
 </li>
 </ul>
 </div>
 </div>
</div>
```

1.  根据线框布局，我们应该在推荐部分下方添加计划价格表，如下所示：

```html
<div class="startup-body">
<!-- ... feature list section … -->
<!-- ... testimonial section … --> 
<div class="startup-pricing">
 <div class="row">
 <div class="medium-4 columns">
 <ul class="pricing-table pricing-basic">
 <li class="title">Basic</li>
 <li class="price">$10<small>/month</small></li>
 <li class="description">Perfect for personal use.</li>
 <li class="bullet-item">1GB Storage</li>
 <li class="bullet-item">1 User</li>
 <li class="bullet-item">24/7 Support</li>
 <li class="cta-button"><a class="button success round" href="#">Sign Up</a></li>
 </ul>
 </div>
 <div class="medium-4 columns">
 <ul class="pricing-table pricing-team">
 <li class="title">Team</li>
 <li class="price">$50<small>/month</small></li>
 <li class="description">For a small team.</li>
 <li class="bullet-item">50GB Storage</li>
 <li class="bullet-item">Up to 10 Users</li>
 <li class="bullet-item">24/7 Support</li>
 <li class="cta-button"><a class="button success round" href="#">Sign Up</a></li>
 </ul>
 </div>
 <div class="medium-4 columns">
 <ul class="pricing-table pricing-enterprise">
 <li class="title">Enterprise</li>
 <li class="price">$300<small>/month</small></li>
 <li class="description">For large corporation</li>
 <li class="bullet-item">Unlimited Storage</li>
 <li class="bullet-item">Unlimited Users</li>
 <li class="bullet-item">24/7 Priority Support</li>
 <li class="cta-button"><a class="button success round" href="#">Sign Up</a></li>
 </ul>
 </div>
 </div>
 </div>
</div>
```

1.  最后，在正文内容下方添加网站页脚，如下所示：

```html
</div> <!—the body content end --> 
<footer class="startup-footer">
 <div class="row footer-nav">
 <ul class="secondary-nav">
 <li><a href="#">About</a></li>
 <li><a href="#">Contact</a></li>
 <li><a href="#">Help</a></li>
 <li><a href="#">Careers</a></li>
 <li><a href="#">Terms</a></li>
 <li><a href="#">Privacy</a></li>
 </ul>
 <ul class="social-nav">
 <li><a class="foundicon-facebook" href="#">Facebook</a></li>
 <li><a class="foundicon-twitter" href="#">Twitter</a></li>
 </ul>
 </div>
 <div class="row footer-copyright">
 <p>Copyright 2014 Super Awesome App. All rights reserved.</p>
 </div>
 </footer>
</body> 
```

## *刚刚发生了什么？*

我们只是按照 Foundation 指南构建了网站内容和部分的 HTML 标记。我们还在途中添加了额外的类，以便稍后自定义 Foundation 默认样式。

自从构建 HTML 标记以来，我们还没有添加任何样式；此时的网站看起来是白色和简单的，如下面的屏幕截图所示：

![刚刚发生了什么？](img/image00362.jpeg)

### 提示

我们刚刚添加的 HTML 的完整代码也可以在[`git.io/qvdupQ`](http://git.io/qvdupQ)找到。

# 摘要

本章有效地开始了我们的第三个项目。在这个项目中，我们使用 Foundation 为一家初创公司构建网站。我们浏览了 Foundation 的特性，并将其中一些特性应用到了网站中。不过，本章中我们只添加了网站的 HTML 结构。此时的网站看起来仍然是白色和简单的。我们需要编写样式来定义网站的外观和感觉，这正是我们将在下一章中做的事情。

我们将使用 Sass 来编写网站样式，Sass 是 CSS 预处理器，也定义了 Foundation 基本样式。因此，在下一章的开始，我们将首先学习如何使用 Sass 变量、混合、函数和其他 Sass 特性，然后再编写网站样式。

看起来还有很多工作要做才能完成这个项目。因此，话不多说，让我们继续下一章吧。
