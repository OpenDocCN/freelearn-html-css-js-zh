# 第十章：不要重复自己-让我们创建一个简单的 CSS 框架

你有多少次做了一些工作，只是在下一个项目中重复了一遍？你有多少次想到了可重复使用的元素？在编码时，你应该考虑下一次在同一个或另一个项目上工作时可以省略多少操作。这意味着你需要使用以下内容：

+   自动化

+   代码模板或框架

这一章是关于构建可重用代码以及如何最终将其用作项目基础的。在这一章中，我们将涵盖以下主题：

+   为一个小而简单的 CSS 框架制定计划

+   创建你自己的网格系统

+   创建可重用元素

请记住，这段代码可以并且应该被扩展。所示的过程应该让你更加清楚如何利用你已经创建的框架来帮助自己，但仍然可以随着你的代码发展而发展。当然，你也可以使用其他框架。

# 文件结构

当你计划一个系统/框架时，文件结构是非常重要的。当你开始创建某些东西时，它需要一个演变。所以根据开发过程，你的系统在不断演变。当你的系统在演变时，它会发生很多变化。所以，让我们创建一个简单的结构：

+   有用的 mixin：

+   有用元素的简化形式

+   内联列表

+   基本元素

+   清除浮动

+   简单的渐变生成器

+   网格 mixin：

+   n 列中的第 n 列

+   表单：

+   输入/文本区样式助手

+   输入占位符

+   按钮：

+   内联（自动宽度）

+   全宽度

+   标准导航：

+   一级

+   两级

我们将使用 mixin 而不是已经创建的类。为什么？我们希望尽量减少 CSS 代码，这样当我们生成完整的 12 列网格时，我们将在媒体查询中的每个断点产生 12 个类。作为前端开发人员，我们希望创建尽可能少的代码。当然，我们可以重用一些类并用 SASS 扩展它们，但这个框架的主要方法是简单和可重用的 mixin。

# 有用元素的简化形式

在 CSS 代码（不仅仅是 CSS），每次重复代码的一部分时，你希望更快地获得最终效果。那么为什么不为一些 CSS 声明创建简短的形式呢？让我们看看我们可以做些什么来缩短它：

```css
/* Text decoration */
=tdn
  text-decoration: none

=tdu
  text-decoration: underline

/* Text align */
=tac
  text-align: center

=tar
  text-align: right

=tal
  text-align: left

/* Text transform */
=ttu
  text-transform: uppercase

=ttl
  text-transform: lowercase

/* Display */
=di
  display: inline

=db
  display: block

=dib
  display: inline-block

/* Margin 0 auto */
=m0a
  margin: 0 auto
```

现在，每次你想将一些文本转换为大写时，只需使用以下代码：

```css
.sampleClass
  +ttu
```

这是编译后的 CSS：

```css
.sampleClass {
    text-transform: uppercase;
}
```

另一个使用短 mixin 的例子是一个元素，它将显示为块元素，文本将居中显示：

```css
.sampleClass
  +db
  +tac
```

这是编译后的 CSS：

```css
.sampleClass {
    display: block;
    text-align: center;
}
```

# 其他 mixin

还有其他对我们的框架有用的 mixin：

+   渐变

+   动画

+   清除浮动

让我们从渐变 mixin 开始：

```css
=linearGradientFromTop($startColor, $endColor)
  background: $startColor /* Old browsers */
  background: -moz-linear-gradient(top,  $startColor 0%, $endColor 100%)
  background: -webkit-gradient(linear, left top, left bottom, color-stop(0%, $startColor), color-stop(100%, $endColor))
  background: -webkit-linear-gradient(top,  $startColor 0%, $endColor 100%)
  background: -o-linear-gradient(top,  $startColor 0%, $endColor 100%)
  background: -ms-linear-gradient(top,  $startColor 0%, $endColor 100%)
  background: linear-gradient(to bottom,  $startColor 0%, $endColor 100%)
  filter: progid:DXImageTransform.Microsoft.gradient( startColorstr='#{$startColor}', endColorstr='#{$endColor}',GradientType=0 ) 

=linearGradientFromLeft($startColor, $endColor)
  background-color: $startColor
  background: -webkit-gradient(linear, left top, right top, from($startColor), to($endColor))
  background: -webkit-linear-gradient(left, $startColor, $endColor)
  background: -moz-linear-gradient(left, $startColor, $endColor)
  background: -ms-linear-gradient(left, $startColor, $endColor)
  background: -o-linear-gradient(left, $startColor, $endColor)
  background: linear-gradient(left, $startColor, $endColor)
  filter: progid:DXImageTransform.Microsoft.gradient(startColorStr='#{$startColor}', endColorStr='#{$endColor}', gradientType='1')
```

全部动画：

```css
=animateAll($time)
  -webkit-transition: all $time ease-in-out
  -moz-transition: all $time ease-in-out
  -o-transition: all $time ease-in-out
  transition: all $time ease-in-out
```

## 清除浮动

不要忘记在你的私人 SASS 框架中的 mixin 中添加`clearfix`。你将使用它作为 mixin 的调用或作为一个类，并且所有其他元素将扩展之前创建的类：

```css
=clearfix
  &:after
    content: " "
    visibility: hidden
    display: block
    height: 0
    clear: both
```

每次你想创建一个可重用的`clearfix`类时，可以这样做：

```css
.clearfix
  +clearfix
```

这是编译后的 CSS：

```css
.clearfix:after {
    content: " ";
    visibility: hidden;
    display: block;
    height: 0;
    clear: both;
}
```

或者可以写一个更简短的版本：

```css
.cf
  +clearfix
```

这是编译后的 CSS：

```css
.cf:after {
    content: " ";
    visibility: hidden;
    display: block;
    height: 0;
    clear: both;
}
```

现在，你可以在 SASS 代码中使用`@extend`来扩展它：

```css
.element
  @extend .cf
```

这是编译后的 CSS：

```css
.cf:after, .element:after {
    content: " ";
    visibility: hidden;
    display: block;
    height: 0;
    clear: both;
}
```

将绝对元素居中在另一个相对元素中：

```css
/* Absolute center vertically and horizontally */
=centerVH
  position: absolute
  top: 50%
  left: 50%
  -ms-transform: translate(-50%,-50%)
  -webkit-transform: translate(-50%,-50%)
  transform: translate(-50%,-50%)
```

# 媒体查询

在每个响应式网页项目中，你都需要创建媒体查询。你需要选择要实施的步骤，然后开始根据这些步骤创建项目。

## 媒体查询模板

媒体查询非常简单易用。媒体查询的主要问题是可重用的步骤，你可以将它们保存在一个地方。在一些项目中，你需要添加更多的查询，因为项目可见性问题或一些额外的代码会影响你的代码。让我们专注于如何通过一些设置来做一次，然后在我们的代码中使用它。

基本设置集中在以下内容上：

+   移动设备（手机）

+   移动设备（平板）

+   桌面设备

+   桌面设备（大屏）

在某些情况下，你可以扩展这个列表，加入移动设备位置（纵向和横向），但是更少的媒体查询对于维护来说更好更容易。那么我们如何保持这些尺寸呢？

+   `$small`：320 像素

+   `$medium`：768 像素

+   `$large`：1024 像素

# 网格

在标准的 HTML/CSS 项目中，最常见的元素是网格。当然，你可以使用别人的网格或从 CSS 框架（如 Bootstrap 或 Foundation）中获取。从头开始创建它很难吗？并不是真的。在本章中，我们将创建一个基本的网格系统，并将使用它来看看它是如何创建行和列的。

## 标准网格 16/12

标准网格是基于 16 列或 12 列系统的。这两种系统的优势是什么？这取决于你的结构。例如，分析布局后，假设你需要：

+   3 列组合

+   2 列组合

+   6 列组合

因此，你可以使用 12 列系统。但是，正如你所看到的，你需要坚持这个系统，那么你如何创建自己的代码，使其更有弹性呢？你可以使用以下命名约定：

```css
.grid-NofK
```

这里，`N`是列数，`K`是分隔符，例如：

```css
.grid-3of12
.grid-5of6
```

当你在处理网格时，你需要记住有时你需要从左边推一些列。这种情况是当你需要创建`.push`类时：

```css
.push-NofK
```

这种命名约定的优点是什么？没有静态分隔符。在经典网格中，你有一个有 12 列或 16 列及其组合的网格。这里是逐个类写的网格示例：

12 列网格：

```css
.grid-1of12 {
    width: 8.33%
}

.push-1of12 {
    margin-left: 8.33%
}

.grid-2of12 {
    width: 16.66%
}

.push-2of12 {
    margin-left: 16.66%
}

.grid-3of12 {
    width: 25%
}

.push-3of12 {
    margin-left: 25%
}

.grid-4of12 {
    width: 33.33%
}

.push-4of12 {
    margin-left: 33.33%
}

.grid-5of12 {
    width: 41.66%
}

.push-5of12 {
    margin-left: 41.66%
}

.grid-6of12 {
    width: 50%
}

.push-6of12 {
    margin-left: 50%
}

.grid-7of12 {
    width: 58.33%
}

.push-7of12 {
    margin-left: 58.33%
}

.grid-8of12 {
    width: 66.66%
}

.push-8of12 {
    margin-left: 66.66%
}

.grid-9of12 {
    width: 75%
}

.push-9of12 {
    margin-left: 75%
}

.grid-10of12 {
    width: 83.33%
}

.push-10of12 {
    margin-left: 83.33%
}

.grid-11of12 {
    width: 91.66%
}

.push-11of12 {
    margin-left: 91.66%
}

.grid-12of12 {
    width: 100%
}

.push-12of12 {
    margin-left: 100%
}
```

16 列网格：

```css
.grid-1of16 {
    width: 6.25%
}

.push-1of16 {
    margin-left: 6.25%
}

.grid-2of16 {
    width: 12.5%
}

.push-2of16 {
    margin-left: 12.5%
}

.grid-3of16 {
    width: 18.75%
}

.push-3of16 {
    margin-left: 18.75%
}

.grid-4of16 {
    width: 25%
}

.push-4of16 {
    margin-left: 25%
}

.grid-5of16 {
    width: 31.25%
}

.push-5of16 {
    margin-left: 31.25%
}

.grid-6of16 {
    width: 37.5%
}

.push-6of16 {
    margin-left: 37.5%
}

.grid-7of16 {
    width: 43.75%
}

.push-7of16 {
    margin-left: 43.75%
}

.grid-8of16 {
    width: 50%
}

.push-8of16 {
    margin-left: 50%
}

.grid-9of16 {
    width: 56.25%
}

.push-9of16 {
    margin-left: 56.25%
}

.grid-10of16 {
    width: 62.5%
}

.push-10of16 {
    margin-left: 62.5%
}

.grid-11of16 {
    width: 68.75%
}

.push-11of16 {
    margin-left: 68.75%
}

.grid-12of16 {
    width: 75%
}

.push-12of16 {
    margin-left: 75%
}

.grid-12of16 {
    width: 81.25%
}

.push-12of16 {
    margin-left: 81.25%
}

.grid-12of16 {
    width: 87.5%
}

.push-12of16 {
    margin-left: 87.5%
}

.grid-12of16 {
    width: 93.75%
}

.push-12of16 {
    margin-left: 93.75%
}

.grid-12of16 {
    width: 100%
}

.push-12of16 {
    margin-left: 100%
}
```

写了很多东西……

现在，我们需要创建一个可以在媒体查询和响应式网站上使用的代码。在最流行的 CSS 框架（如 Bootstrap 和 Foundation）中，你可以为手机/平板/桌面使用类：

```css
<div class="small-2 medium-4 large-5">
</div>
```

例如，当分隔符设置为`12`时，你将在小设备上看到这个框是`2`列宽，中等设备上是`4`列宽，大文档上是`5`列宽。我们可以创建所有这些类，但我建议你创建一个 mixin，我们可以在 CSS 中描述的每个元素中调用它。

SASS 代码将如下所示：

```css
=grid($columns, $divider)
  width: percentage($columns/$divider)

=push($columns, $divider)
  margin-left: percentage($columns/$divider)
```

我们如何在 SASS 代码中使用它？假设我们有一个基于网格`16`的块，并且我们想要给它宽度为`12`的`16`，并用`2`的`16`推动它：

```css
.gridElement
  +grid(12, 16)
  +push(2, 16)
```

这是编译后的 CSS：

```css
.gridElement {
    width: 75%;
    margin-left: 12.5%;
}
```

# 标准可重用结构

作为前端开发人员，你总是在努力处理可重复的元素。在几乎所有情况下，你会觉得自己在试图重复造轮子，那么你可以做些什么来避免重复呢？让我们创建一些标准和可重用的结构。

## 可重用的多级菜单

多级菜单是最可重用的代码。所有更大的网站都有一个你可以描述为可重用代码的菜单。

让我们从 HTML 代码开始：

```css
<ul class="menu-multilevel">
    <li>
        <a href="#">Level one - item one</a>
        <ul>
            <li><a href="#">Level two - item one</a></li>
            <li><a href="#">Level two - item two</a></li>
            <li><a href="#">Level two - item three</a></li>
            <li><a href="#">Level two - item four</a></li>
        </ul>
    </li>
    <li>
        <a href="#">Level two - item one</a>
        <ul>
            <li><a href="#">Level two - item one</a></li>
            <li><a href="#">Level two - item two</a></li>
            <li><a href="#">Level two - item three</a></li>
            <li><a href="#">Level two - item four</a></li>
        </ul>
    </li>
    <li>
        <a href="#">Level one - item three</a>
        <ul>
            <li><a href="#">Level three - item one</a></li>
            <li><a href="#">Level three - item two</a></li>
            <li><a href="#">Level three - item three</a></li>
            <li><a href="#">Level three - item four</a></li>
        </ul>
    </li>
</ul>
```

SASS 代码：

```css
ul.menu-multilevel
  list-style: none
  padding: 0

ul.menu-multilevel > li
  float: left
  display: inline-block
  position: relative
  margin-right: 10px

  &:hover
    ul
      display: block
      width: 200px

ul.menu-multilevel ul
  display: none
  position: absolute
  left: 0

  li
    display: block
```

这是编译后的 CSS：

```css
ul.menu-multilevel {
    list-style: none;
    padding: 0;
}

ul.menu-multilevel > li {
    float: left;
    display: inline-block;
    position: relative;
    margin-right: 10px;
}

ul.menu-multilevel > li:hover ul {
    display: block;
    width: 200px;
}

ul.menu-multilevel ul {
    display: none;
    position: absolute;
    left: 0;
}

ul.menu-multilevel ul li {
    display: block;
}
```

现在，让我们稍微重建这个代码，以在 SASS 中创建一个可重用的 mixin：

```css
=memuMultilevel
  list-style: none
  padding: 0

  & > li
    float: left
    display: inline-block
    position: relative
    margin-right: 10px

    &:hover
      ul
        display: block
        width: 200px

  & ul
    display: none
    position: absolute
    left: 0

    li
      display: block
```

要使用它，你需要像这样调用一个 mixin：

```css
ul.menu-multilevel
  +memuMultilevel
```

生成的 CSS：

```css
ul.menu-multilevel {
    list-style: none;
    padding: 0;
}

ul.menu-multilevel > li {
    float: left;
    display: inline-block;
    position: relative;
    margin-right: 10px;
}

ul.menu-multilevel > li:hover ul {
    display: block;
    width: 200px;
}

ul.menu-multilevel ul {
    display: none;
    position: absolute;
    left: 0;
}

ul.menu-multilevel ul li {
    display: block;
}
```

## 如何创建可重用的按钮

按钮是下一个你可以看到和重复使用的元素。让我们考虑一下按钮参数。当然，我们需要有机会设置背景和字体颜色。我们需要有机会改变边框颜色和填充。

让我们从一个简单的 CSS 定义开始：

```css
.button {
    padding: 5px 10px;
    background: #ff0000;
    color: #fff;
}
```

因此，基于此，mixin 在 SASS 中可以如下所示：

```css
=button($bgc, $fc)
  display: inline-block
  background: $bgc
  color: $fc
```

这里：

+   `$bgc`：背景颜色

+   `$fc`：字体颜色

要使用这个 mixin，你只需要执行这个：

```css
.button
  padding: 5px 10px
  +button(#ff0000, #fff)
```

这是编译后的 CSS：

```css
.button {
    padding: 5px 10px;
    display: inline-block;
    background: #ff0000;
    color: #fff;
}
```

你如何扩展这个 mixin？让我们考虑一下其他可以参数化的值。当然，边框半径。所以，让我们添加一个新的 mixin：

```css
=roundedButton($bgc, $fc, $bc, $br)
  background: $bgc
  color: $fc
  border-color: $bc
  border-radius: $br
```

这里：

+   `$bc`：边框颜色

+   `$br`：边框半径

让我们使用这个 mixin：

```css
.roundedButton
  +roundedButton(black, white, red, 5px)
```

这是编译后的 CSS：

```css
.roundedButton {
    background: black;
    color: white;
    border-color: red;
    border-radius: 5px;
}
```

如果你需要创建一堆有三种尺寸的按钮，你可以这样做：

```css
.button
  +button(#ff0000, #fff)

  .small
    padding: 5px 10px

  .medium
    padding: 10px 20px

  .large
    padding: 15px 30px
```

这是编译后的 CSS：

```css
.button {
    display: inline-block;
    background: #ff0000;
    color: #fff;
}

.button .small {
    padding: 5px 10px;
}

.button .medium {
    padding: 10px 20px;
}

.button .large {
    padding: 15px 30px;
}
```

# 收集其他可重用的 mixin

我们需要一堆有用的可重用的 mixin。还有什么可以额外帮助的？让我们想一想：

+   基本元素

+   内联列表

## 基本元素

正如你可能还记得之前的章节中所提到的，我们一直在使用基元。创建基元的 mixin 列表可以成为我们框架非常有用和有帮助的一部分。我们将为以下内容创建 mixin：

+   矩形（带或不带填充）

+   圆/环

+   三角形

让我们快速回顾一下：

```css
=rectangle($w, $h, $c)
  width: $w
  height: $h
  background: $c

=square($w, $c)
  width: $w
  height: $w
  background: $c

=circle($size, $color)
  width: $size
  height: $size
  border-radius: 50%
  background: $color

=ring($size, $color, $width)
  width: $size
  height: $size
  border-radius: 50%
  border: $width solid $color
  background: none

=triangleRight($width, $height, $color)
  width: 0
  height: 0
  border-style: solid
  border-width: $height/2 0 $height/2 $width
  border-color: transparent transparent transparent $color

=triangleLeft($width, $height, $color)
  width: 0
  height: 0
  border-style: solid
  border-width: $height/2 $width $height/2 0
  border-color: transparent $color transparent transparent

=triangleTop($width, $height, $color)
  width: 0
  height: 0
  border-style: solid
  border-width: 0 $width/2 $height $width/2
  border-color: transparent transparent $color transparent

=triangleBottom($width, $height, $color)
  width: 0
  height: 0
  border-style: solid
  border-width: $height $width/2 0 $width/2
  border-color: $color transparent transparent transparent
```

# 让我们测试和使用我们的框架

为了检查我们的框架是如何工作的，以及添加所有内容有多容易，让我们创建一个博客模板。在这个模板中，让我们包括视图：

+   帖子列表

+   单个帖子

+   单页

让我们创建区域：

+   页眉

+   页脚

+   内容

这是我们简化的设计：

！[让我们测试和使用我们的框架]（img/00146.jpeg）

让我们从博客页面（主页）的简单结构开始：

```css
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title></title>
    <link rel="stylesheet" href="css/master.css" media="screen" title="no title" charset="utf-8">
</head>
<body>
<header>
    <h1>FEDojo.com</h1>
    <h2>Front End Developers Blog</h2>
</header>

<nav>
    <ul>
        <li><a href="#">Home</a></li>
        <li><a href="#">About</a></li>
        <li><a href="#">Contact</a></li>
    </ul>
</nav>

<main>
    <article class="main--article">
        <a href="#">
            <img src="img/error_log.png" alt=""/>
            <span class="comments"></span>
        </a>
        <h3>Lorem ipsum dolor sit amet, consectetur adipisicing elit</h3>
        <p>
            sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud
            exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit
            in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non
            proident, sunt in culpa qui officia deserunt mollit anim id est laborum.
        </p>
        <a href="#" class="readmore">Read more</a>
    </article>

</main>

<footer>
    <div class="wrapper">
        <div class="column">
            Left column
        </div>
        <div class="column">
            Right column
        </div>
    </div>
</footer>
</body>
</html>
```

正如你所看到的，我们有一个基于标签的结构：

+   页眉

+   导航

+   主要

+   页脚

这是我们的文件结构：

！[让我们测试和使用我们的框架]（img/00147.jpeg）

让我们描述页眉：

```css
header
  h1
    +tac
    margin-bottom: 0

  h2
    +tac
    font-size: 16px
    margin-top: 0
    margin-bottom: 30px
```

描述页脚：

```css
footer
  width: 100%
  background: #d3d3d3
  padding: 50px 0

  .wrapper
    +m0a /* margin 0 auto */
    +clearfix
    max-width: $wrapper

  .column
    width: 50%
    float: left
```

描述导航：

```css
nav
  background: black
  text-align: center

  ul
    +navigation

  a
    color: white
    +ttu
    padding: 10px
```

在`fed`目录中，我们存储可重复使用的代码（我们的框架）。在其余的目录中，我们存储与项目相关的代码。在描述结构中，我们存储在所有视图上重复的元素的样式。在视图目录中，我们将保留与特定视图相关的元素的样式。

# 记住！

当你创建一些可重复使用的代码甚至任何其他代码时，你需要留下评论。出于某种原因，程序员们当前（并且不礼貌地）趋向于不添加评论“他们的代码不需要额外的描述”。另一种思路是，“那是我的代码。我知道我在写什么”。你认为把它留下是公平的吗？当然，答案是否定的！即使你的记忆也不完美。你可能会忘记你在代码中的意思和目的是什么。建议你至少为自己和将来在项目上工作的其他人写一些简短的评论。

在 Github 和 Bitbucket 的黄金时代，你可以在几秒钟内分享你的代码，并与来自世界另一部分的另一位程序员一起工作，他可以 fork 你的代码或为你的项目做出贡献。

# 摘要

正如你所看到的，有很多可重复使用的结构，每次创建新项目时都可以装饰。最好是写一次东西，然后添加一些新功能，而不是每次都写一些东西并描述可重复的元素。

在下一章中，我们将尝试创建一个简单的 CSS 框架，其中包含准备好使用的组件！
