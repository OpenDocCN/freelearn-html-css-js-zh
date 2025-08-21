# 第一章：响应式网页设计

*我仍然记得，小时候，手机只有一个微小的单色屏幕。那时我们能做的只是打电话、发短信和玩简单的游戏。如今，移动设备在许多方面都有了巨大的进步。*

*新的移动设备具有不同的屏幕尺寸；一些甚至具有更高的 DPI 或分辨率。大多数新的移动设备现在配备了触摸屏，使我们可以方便地使用手指轻触或滑动与设备进行交互。屏幕方向可以在纵向和横向之间切换。与旧设备相比，软件也更加强大。特别是移动浏览器现在能够呈现和显示与桌面电脑浏览器一样好的网页。*

*此外，过去几年移动用户的数量激增。现在我们可以看到周围有很多人花费数小时使用他们的移动设备，手机或平板电脑，进行诸如在外出时经营业务或简单的互联网浏览等活动。移动用户的数量未来可能会继续增长，甚至可能超过桌面用户的总数。*

*也就是说，移动设备改变了网络，改变了人们使用互联网和享受网站的方式。移动设备的这些进步和不断增长的移动互联网使用量引发了一个新的构建网站的范式的问题，即如何在不同情况下构建可访问且功能良好的网站。这就是**响应式网页设计**的用武之地。*

在本章中，我们将涵盖以下主题：

+   简要了解响应式网页设计、视口 meta 标签和 CSS3 媒体查询的基础知识

+   在接下来的章节中，我们将使用响应式框架来构建响应式网站

# 简而言之，响应式网页设计

响应式网页设计是网页设计和开发社区中讨论最多的话题之一。因此，我相信你们很多人对它有一定程度的了解。

伊桑·马科特是首次提出“响应式网页设计”这个术语的人。他在他的文章*响应式网页设计*（[`alistapart.com/article/responsive-web-design/`](http://alistapart.com/article/responsive-web-design/)）中建议，网页应该无缝地调整和适应用户查看网站的环境，而不是专门针对特定平台进行处理。换句话说，网站应该是响应式的，无论在哪种屏幕尺寸上查看，都应该能够呈现。

以时代网站（[`time.com/`](http://time.com/)）为例。网页在桌面浏览器上的大屏幕上和移动浏览器上的有限可视区域上都能很好地适应。布局会随着视口大小的变化而变化和适应。如下截图所示，在移动浏览器上，标题的背景颜色是深灰色，图片按比例缩小，并且出现了一个“点击”栏，时代隐藏了最新新闻、杂志和视频栏目：

![响应式网页设计简介](img/image00219.jpeg)

构建响应式网站有两个组成部分，即**视口 meta 标签**和**媒体查询**。

## 视口 meta 标签

在智能手机如 iPhone 成为主流之前，每个网站都是建立在大约 1000 像素宽或 980 像素宽的基础上，然后缩小以适应手机屏幕，最终导致网站无法阅读。因此，`<meta name="viewport">`被创建出来。

简而言之，视口`meta`标签用于定义浏览器中网页的比例和可见区域（视口）。以下代码是视口 meta 标签的示例：

```html
<meta name="viewport" content="width=device-width, initial-scale=1">
```

上述视口`meta`标签规范将网页视口宽度定义为跟随设备。它还定义了在首次打开网页时网页缩放为 1:1，使得网页内容的大小和尺寸保持不变；它们不应该被放大或缩小。

为了更好地理解视口 meta 标签对网页布局的影响，我创建了两个网页进行比较；一个添加了视口 meta 标签，另一个没有。您可以在以下截图中看到差异：

![视口 meta 标签](img/image00220.jpeg)

在上图中显示的第一个网站使用了与我们之前代码示例中完全相同的视口 meta 标签规范。由于我们指定了`width=device-width`，浏览器会认识到网站视口与设备屏幕大小相同，因此不会压缩网页以适应整个屏幕。`initial-scale=1`将保持标题和段落的原始大小。

在第二个网站的示例中，由于我们没有添加视口`meta`标签，浏览器假定网页应该完全显示。因此，浏览器强制整个网站适应整个屏幕区域，使得标题和文本完全不可读。

### 关于屏幕大小和视口的说明

您可能在许多网页设计论坛或博客上发现视口和屏幕大小经常可以互换地提到。但实际上，它们是两个不同的东西。

屏幕大小指的是设备的实际屏幕大小。例如，13 英寸的笔记本电脑通常具有 1280*800 像素的屏幕大小。而视口描述的是浏览器中显示网站的可视区域。以下图示说明了这一点：

![关于屏幕大小和视口的说明](img/image00221.jpeg)

## 媒体查询

CSS 中的媒体类型模块使我们能够将样式规则定位到特定的媒体上。如果您以前创建过打印样式表，那么您肯定熟悉媒体类型的概念。CSS3 引入了一个称为媒体查询的新媒体类型，它允许我们在指定的视口宽度范围内应用样式，也被称为断点。

以下是一个简单的例子；当网站的视口大小为`480px`或更小时，我们将网站的`p`字体大小从`16px`减小到`14px`。

```html
p { 
font-size: 16px;
}
@media screen and (max-width: 480px) {
p {
    font-size: 14px;
}
} 
```

以下图示说明了上述代码：

![媒体查询](img/image00222.jpeg)

我们还可以使用`and`运算符结合多个视口宽度范围。根据我们之前的例子，我们可以将`p`字体大小设置为`14px`，当视口大小在`480px`和`320px`之间时，如下所示：

```html
@media screen and (min-width: 320px) and (max-width: 480px) {
p {
font-size: 11px;
  }
}
```

### 注意

**视口和媒体查询参考**

在构建响应式网站时，我们将处理视口 meta 标签和媒体查询。Packt Publishing 出版了一本专门的书籍，*使用 HTML5 和 CSS3 构建响应式网页设计*，*Ben Frein*，*Packt Publishing*，其中更详细地介绍了这两个方面。我建议将其作为本书的伴读书籍。

# 对响应式框架的一瞥

构建响应式网站可能是非常繁琐的工作。在构建响应式网站时需要考虑许多测量标准，其中之一就是创建响应式网格。

网格帮助我们构建具有正确对齐的网站。如果您以前使用过 960.gs（[`960.gs/`](http://960.gs/)），这是一个流行的 CSS 框架之一，您可能已经体验过通过在元素中添加预设类（如`grid_1`或`push_1`）来组织网页布局是多么容易。

然而，960.gs 网格是以固定单位像素（`px`）设置的，这在构建响应式网站时是不适用的。我们需要一个以百分比（`%`）单位设置网格的框架来构建响应式网站；我们需要一个响应式框架。

响应式框架提供了构建响应式网站的基本组件。通常，它包括用于组装响应式网格的类、用于排版和表单输入的基本样式，以及一些样式来解决各种浏览器的怪癖。一些框架甚至通过一系列样式进一步创建常见的设计模式和网页用户界面，如按钮、导航栏和图像滑块。这些预定义的样式使我们能够更快地开发响应式网站，减少麻烦。以下是使用响应式框架构建响应式网站的几个其他原因：

+   **浏览器兼容性**: 确保网页在不同浏览器上的一致性真的比开发网站本身更令人痛苦和苦恼。然而，有了框架，我们可以最小化处理浏览器兼容性问题的工作。框架开发人员很可能在公开发布之前在各种桌面浏览器和移动浏览器上测试了框架，这些浏览器环境最受限制。

+   **文档**: 一般来说，框架还附带有全面的文档，记录了使用框架的方方面面。文档对于初学者开始学习框架非常有帮助。当我们与团队合作时，文档也是一个很大的优势。我们可以参考文档让每个人都在同一页面上，并遵循标准的编码规范。

+   **社区和扩展**: 一些流行的框架，如 Bootstrap 和 Foundation，有一个活跃的社区，帮助解决框架中的错误并扩展功能。jQuery UI Bootstrap ([`jquery-ui-bootstrap.github.io/jquery-ui-bootstrap/`](http://jquery-ui-bootstrap.github.io/jquery-ui-bootstrap/))可能是一个很好的例子。jQuery UI Bootstrap 是一组 jQuery UI 小部件的样式，以匹配 Bootstrap 原始主题的外观和感觉。现在很常见的是找到基于这些框架的免费 WordPress 和 Joomla 主题。

在本书的过程中，我们将使用三种不同的响应式框架 Responsive.gs、Bootstrap 和 Foundation 构建三个响应式网站。

## 响应式.gs 框架

Responsive.gs ([`responsive.gs/`](http://responsive.gs/))是一个轻量级的响应式框架，压缩后仅为 1KB。Responsive.gs 基于宽度为 940px，并以 12、16 和 24 列的三种变体构建。此外，Responsive.gs 还附带了 box-sizing polyfill，它在 Internet Explorer 6、7 和 8 中启用了 CSS3 的 box-sizing，并使其在这些浏览器中表现得体面。

### 注意

Polyfill 是一段代码，它使某些 Web 功能和能力在浏览器中不是原生内置的情况下能够使用。通常，它解决了旧版本的 Internet Explorer 的问题。例如，您可以使用 HTML5Shiv ([`github.com/aFarkas/html5shiv`](https://github.com/aFarkas/html5shiv))，以便在 Internet Explorer 6、7 和 8 中识别新的 HTML5 元素，如`<header>`、`<footer>`和`<nav>`。

### 关于 CSS 框模型的说明

被归类为块级元素的 HTML 元素本质上是通过 CSS 绘制的具有内容宽度、高度、边距、填充和边框的框。在 CSS3 之前，我们在指定框时面临着一些限制。例如，当我们指定一个`<div>`标签的宽度和高度为`100px`时，如下所示：

```html
div { 
  width: 100px;
  height: 100px;
}
```

浏览器将`div`呈现为`100px`的正方形框，如下图所示：

![关于 CSS 框模型的说明](img/image00223.jpeg)

然而，只有在没有添加填充和边框的情况下才会成立。由于框有四个边，填充为 10px（`padding: 10px;`）实际上会为宽度和高度增加 20px——每个边增加 10px，如下图所示：

![关于 CSS 框模型的说明](img/image00224.jpeg)

虽然它占据页面上的空间，但元素的边距空间是在元素外部保留的，而不是作为元素本身的一部分；因此，如果我们给元素设置背景颜色，边距区域将不会采用该颜色。

### CSS3 盒模型

CSS3 引入了一个名为`box-sizing`的新属性，它允许我们指定浏览器应如何计算 CSS 盒模型。我们可以在`box-sizing`属性中应用几个值。

| 值 | 描述 |
| --- | --- |
| `content-box` | 这是盒模型的默认值。此值指定填充和边框框的厚度在指定的宽度和高度之外，正如我们在前面的部分中所演示的那样。 |
| `border-box` | 此值将执行与`content-box`相反的操作；它将包括填充和边框框作为盒子的宽度和高度。 |
| `padding-box` | 在撰写本书时，此值是实验性的，并且最近才被添加。此值指定了盒子的尺寸。 |

在本书的每个项目中，我们将使用`border-box`值，以便我们可以轻松确定网站的盒子尺寸。让我们以前面的例子来理解这一点，但这次我们将`box-sizing`模型设置为`border-box`。如前表所述，`border-box`值将保留盒子的宽度和高度为`100px`，而不管填充和边框的添加。下图显示了两种不同值的输出之间的比较，`content-box`（默认值）和`border-box`：

![CSS3 box sizing](img/image00225.jpeg)

在本书中，我们将使用 Responsive.gs，并在接下来的两章中更多地探索它，以构建一个简单的响应式博客。

## Bootstrap 框架

Bootstrap（[`getbootstrap.com/`](http://getbootstrap.com/)）最初是由 Mark Otto（[`markdotto.com/`](http://markdotto.com/)）开发，并最初仅用于 Twitter 内部使用。简而言之，Bootstrap 随后免费向公众发布。

### 注意

Bootstrap 长期以来一直与 Twitter 相关联，但自作者离开 Twitter 并且 Bootstrap 本身已经超出了他的预期，Bootstrap 现在成为了自己的品牌（[`blog.getbootstrap.com/2012/09/29/onward/`](http://blog.getbootstrap.com/2012/09/29/onward/)）。

如果参考最初的开发，响应式功能尚未添加。由于对创建响应式网站的需求不断增加，它在版本 2 中被添加。

Bootstrap 相比 Responsive.gs 还带有许多其他功能。它内置了预设的用户界面样式，包括网站上常用的按钮、导航栏、分页和表单等常见用户界面，因此在启动新项目时无需从头开始创建它们。此外，Bootstrap 还配备了一些自定义的 jQuery 插件，如图像滑块、轮播、弹出框和模态框。

您可以以多种方式使用和自定义 Bootstrap。您可以直接通过 CSS 样式表自定义 Bootstrap 主题及其组件，通过 Bootstrap 自定义和下载页面（[`getbootstrap.com/customize/`](http://getbootstrap.com/customize/)），或者使用 Bootstrap LESS 变量和混合，用于生成样式表。

在本书中，我们将在第五章中深入了解 Bootstrap，*使用 Bootstrap 开发投资组合网站*，以及第六章中，*使用 LESS 打磨响应式投资组合网站*，来构建一个响应式投资组合网站。

## Foundation 框架

Foundation（[`foundation.zurb.com/`](http://foundation.zurb.com/)）是由总部位于加利福尼亚的设计机构 ZURB 创建的框架。与 Bootstrap 类似，Foundation 不仅是一个响应式 CSS 框架；它还附带了预设的网格、组件和许多 jQuery 插件，以呈现交互式功能。

一些知名品牌，如 McAfee（[`www.mcafee.com/common/privacy/english/slide.html`](http://www.mcafee.com/common/privacy/english/slide.html)），这是最受尊敬的计算机防病毒品牌之一，已经使用 Foundation 构建了他们的网站。

Foundation 样式表由 Sass 提供支持，Sass 是基于 Ruby 的 CSS 预处理器。我们将在本书的最后两章中更多地讨论 Sass 以及 Foundation 的特性；在那里，我们将为一家初创公司开发一个响应式网站。

### 提示

有许多人抱怨响应式框架中的代码过多；由于 Bootstrap 等框架被广泛使用，它必须涵盖每种设计场景，因此会带有一些您网站可能不需要的额外样式。幸运的是，我们可以通过使用正确的工具（如 CSS 预处理器）和遵循适当的工作流程来轻松解决这个问题。

坦率地说，并没有完美的解决方案；使用框架并不适合每个人。一切都取决于您的需求、您网站的需求，特别是您客户的需求和预算。实际上，您将不得不权衡这些因素，以决定是否使用响应式框架。Jem Kremer 在她的文章*Responsive Design Frameworks: Just Because You Can, Should You?*（[`www.smashingmagazine.com/2014/02/19/responsive-design-frameworks-just-because-you-can-should-you/`](http://www.smashingmagazine.com/2014/02/19/responsive-design-frameworks-just-because-you-can-should-you/)）中对此进行了广泛讨论。

## CSS 预处理器简介

Bootstrap 和 Foundation 都使用 CSS 预处理器来生成它们的样式表。Bootstrap 使用 LESS（[`lesscss.org/`](http://lesscss.org/)）——尽管官方最近才发布了对 Sass 的支持。相反，Foundation 只使用 Sass 来生成其样式表（[`sass-lang.com/`](http://sass-lang.com/)）。

CSS 预处理器并不是一种全新的语言。如果您了解 CSS，您应该立即适应 CSS 预处理器。CSS 预处理器通过允许使用变量、函数和操作等编程功能来简单地扩展 CSS。

以下是我们使用 LESS 语法编写 CSS 的示例：

```html
@color: #f3f3f3;

body {
  background-color: @color;
}
p {
  color: darken(@color, 50%);
}
```

### 提示

**下载示例代码**

您可以从您在[`www.packtpub.com`](http://www.packtpub.com)的帐户中下载您购买的所有 Packt Publishing 图书的示例代码文件。如果您在其他地方购买了本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便直接通过电子邮件接收文件。

当编译前面的代码时，它会取出我们定义的`@color`变量并将其值放入输出中，如下所示：

```html
body {
  background-color: #f3f3f3;
}
p {
  color: #737373;
}
```

该变量可在整个样式表中重复使用，这使我们能够保持样式一致性并使样式表更易于维护。

在构建响应式网站的过程中，我们将进一步使用和探索 CSS 预处理器 LESS 和 Sass，以及 Bootstrap(第五章, *使用 Bootstrap 开发作品集网站*和第六章, *使用 LESS 打磨作品集网站*)和 Foundation(第七章, *使用 Foundation 创建企业响应式网站*和第八章, *扩展 Foundation*)。

## 尝试一下——深入了解响应式网页设计

我们在这里对响应式网页设计的讨论虽然重要，但只是冰山一角。关于响应式网页设计，有很多内容超出了我们最近在前几节中涵盖的内容。我建议你花些时间更深入地了解响应式网页设计，包括概念、技术细节和一些限制，以消除任何疑虑。

以下是一些最佳参考资料的推荐：

+   Ethan Martcotte 的《Responsive Web Design》([`alistapart.com/article/responsive-web-design`](http://alistapart.com/article/responsive-web-design))，这是一切的开始

+   另一个好的起点是 Rachel Shillcock 的《Responsive Web Design》([`webdesign.tutsplus.com/articles/responsive-web-design--webdesign-15155`](http://webdesign.tutsplus.com/articles/responsive-web-design--webdesign-15155))

+   Ian Yates 的《Don't Forget the Viewport Meta Tag》([`webdesign.tutsplus.com/articles/quick-tip-dont-forget-the-viewport-meta-tag--webdesign-5972`](http://webdesign.tutsplus.com/articles/quick-tip-dont-forget-the-viewport-meta-tag--webdesign-5972))

+   Rachel Andrew 的《How To Use CSS3 Media Queries To Create a Mobile Version of Your Website》([`www.smashingmagazine.com/2010/07/19/how-to-use-css3-media-queries-to-create-a-mobile-version-of-your-website/`](http://www.smashingmagazine.com/2010/07/19/how-to-use-css3-media-queries-to-create-a-mobile-version-of-your-website/))

+   阅读有关使用 HTML5 图片元素的响应式图片的未来标准的文章《Responsive Images Done Right: A Guide To <picture> And srcset》作者是 Eric Portis ([`www.smashingmagazine.com/2014/05/14/responsive-images-done-right-guide-picture-srcset/`](http://www.smashingmagazine.com/2014/05/14/responsive-images-done-right-guide-picture-srcset/))

+   一些使数据表响应式的方法的总结 ([`css-tricks.com/responsive-data-table-roundup/`](http://css-tricks.com/responsive-data-table-roundup/))

## 快速测验——响应式网页设计的主要组件

Q1. 在他的文章中，我们在本章中已经提到了两次，Ethan Marcotte 提到了构成响应式网站的主要技术要素。这些主要组件是什么？

1.  视口元标记和 CSS3 媒体查询。

1.  流体网格、灵活图片和媒体查询。

1.  响应式图片、断点和 polyfills。

Q2. 什么是视口？

1.  设备的屏幕尺寸。

1.  网页呈现的区域。

1.  设置网页视口大小的元标记。

Q3. 以下哪一种是声明 CSS3 媒体查询的正确方式？

1.  `@media (max-width: 320px) { p{ font-size:11px; }}`

1.  `@media screen and (max-device-ratio: 320px) { div{ color:white; }}`

1.  `<link rel="stylesheet" media="(max-width: 320px)" href="core.css" />`

# 响应式网页设计的灵感来源

现在，在我们跳入下一章并开始构建响应式网站之前，花些时间寻找响应式网站的想法和灵感可能是个好主意；看看它们是如何构建的，以及在桌面浏览器和移动浏览器上布局是如何组织的。

网站经常重新设计以保持新鲜是很常见的事情。因此，我们最好直接去策划网站的网站，而不是制作一堆网站截图，因为由于重新设计，这些截图可能在接下来的几个月内就不再相关了，以下是去的地方：

+   MediaQueries ([`mediaqueri.es/`](http://mediaqueri.es/))

+   Awwwards ([`www.awwwards.com/websites/responsive-design/`](http://www.awwwards.com/websites/responsive-design/))

+   CSS Awards ([`www.cssawards.net/structure/responsive/`](http://www.cssawards.net/structure/responsive/))

+   WebDesignServed ([`www.webdesignserved.com/`](http://www.webdesignserved.com/))

+   Bootstrap Expo ([`expo.getbootstrap.com/`](http://expo.getbootstrap.com/))

+   Zurb Responsive ([`zurb.com/responsive`](http://zurb.com/responsive))

# 摘要

在本章中，我们简要介绍了响应式网页设计背后的故事，以及视口元标签和 CSS3 媒体查询，这些构成了响应式网站。本章还总结了我们将使用以下框架来进行三个项目的工作：Responsive.gs，Bootstrap 和 Foundation。

使用框架是快速建立响应式网站的更简单的方法，而不是从头开始构建所有内容。然而，正如前面提到的，使用框架也有一些负面影响。如果做得不好，最终结果可能会出现问题。网站可能会被填充并卡在不必要的样式和 JavaScript 中，最终导致网站加载缓慢且难以维护。

我们需要设置合适的工具；它们不仅会促进项目的进行，还将帮助我们使网站更易于维护，这就是我们将在下一章中要做的事情。
