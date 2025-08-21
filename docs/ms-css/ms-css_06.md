# 第六章：变得响应式

到目前为止，我们几乎所有的东西都是固定尺寸。我们的布局有固定宽度，我们的图片有固定宽度，我们的菜单也有固定宽度。但是当使用手机、平板电脑和其他各种设备尺寸时，这样做并不会带来良好的体验。幸运的是，响应式网页设计可以将我们的静态网站转变为流体、设备友好的网站。

开创一切的书籍- *响应式网页设计*，作者*Ethan Marcotte*，*2011*年。他概述了响应式网页设计的三个主要技术支柱：

+   流体网格，

+   灵活的图片，以及

+   媒体查询。

我们将讨论响应式网页设计的三个基本 CSS 基础，然后讨论如何在较小的屏幕尺寸下构建主导航的适配，最后是`viewport`元标签。

# 流体网格

在这一部分，我们将讨论响应式网页设计的三个主要组成部分之一，即流体网格或基于百分比的布局。我们将看看如何将固定宽度布局转换为流体网格，为此，您需要学习将像素转换为百分比的公式。

# 将像素转换为百分比

现在，我们有一个固定宽度布局，如下面的屏幕截图所示：

![](img/00229.jpeg)

如果您缩小浏览器，您会看到它会分解成更小的尺寸，如下面的屏幕截图所示：

![](img/00230.jpeg)

创建流体网格是解决这个问题的第一步。目标是将所有基于像素的宽度以及左右边距和左右填充转换为百分比。我们现在先忽略主导航，但稍后我们会回到它。我们将从`div`标签开始，这是我用来包装大部分内容的`wrapper`类。让我们将属性`width`更改为`max-width`。这表示该元素可以比`960px`宽，但不能超过这个宽度。让我们还将宽度设置为`90%`：

```css
.wrapper {
 width:960px;
 width: 90%;
  margin: 0 auto;
}
```

因此，根据这段代码，我们将宽度设置为其父元素的 90%，而父元素没有宽度。因此，它将是浏览器窗口的 90%。这将使其在`960px`以下的宽度上两侧有 5%的间距。让我们在浏览器中查看网站。您可以刷新浏览器并再次缩小它。下面的屏幕截图显示了它没有显著的影响，看起来相当糟糕：

![](img/00231.jpeg)

我们还想在`wrapper`内部创建这些元素的百分比。由于我们从固定像素宽度开始，我们需要将所有像素转换为百分比。

# 计算百分比宽度

根据*Ethan Marcotte*的*响应式网页设计*，有一个将基于像素的布局转换为基于百分比的布局的公式：*目标/上下文=结果*。*目标*是元素的期望宽度。*上下文*通常是其父元素的宽度。*结果*是我们可以插入到我们的 CSS 中的百分比。

如果我们看看`intro-content`部分内的 HTML，我们可以看到`wrapper`类和其中的两个`div`标签，即`intro-content`和`go-premium`，如下面的代码片段所示：

```css
<section class="grouping">
    <div class="wrapper">
        <div class="intro-content">
            <h1>Old Chompy</h1>
            <h2>Dedicated to sharks and other aquatic species</h2>
            <p>Lorem ipsum dolor sit amet...</p>
        </div><!-- end intro-content -->
        <div class="go-premium">
            <a href="#" class="call-to-action">Go Premium</a>
            <p class="reasons">So many awesome features...
                <a href="#">Learn more &raquo;</a>
            </p>
        </div><!-- end go-premium -->
    </div><!-- end wrapper -->
</section><!-- end section -->
```

回到我们的 CSS，我们的第一个元素是`intro-content`，它是出现在包装器内部的部分，如下面的代码片段所示：

```css
.intro-content {
  width: 600px;
  margin-right: 60px;
  float: left;
}
```

这里的目标是 600 像素，上下文是 960 像素。因此我们的计算是 600 除以 960，等于 0.625。我们将这个值作为我们的宽度插入，并添加一个百分比作为我们的度量单位，并将小数点移动两位，使其变为 62.5%：

```css
.intro-content {
  width: 62.5%; /* 600/960 */
  margin-right: 60px;
  float: left;
}
```

正如您所看到的，声明结束时的我的注释告诉我，元素最初宽度为 600 像素，父元素最初宽度为 960 像素。

`margin-right`属性也需要是百分比。公式仍然是一样的——*目标*除以*上下文*等于*结果*。我们的目标是 60 像素，我们的上下文仍然是 960 像素——父元素，即`wrapper`类。60 除以 960 得到 0.0625。我们将这个转换为百分比，将小数点移动两位，得到`6.25%`：

```css
.intro-content {
  width: 62.5%; /* 600/960 */  margin-right: 6.25%; /* 60/960px */
  float: left;
}
```

接下来是我们的呼吁行动按钮的容器，`go-premium`：

```css
.go-premium {
  width: 300px;
  float: left;
  margin-top: 150px;
}
```

由于宽度是`300px`，它也需要转换为百分比。所以让我们做同样的事情，在这种情况下，300 除以 960——我们仍然有同样的父元素。这是 0.3125。将小数点移动两位，加上百分比，然后将其放在 CSS 注释中，以备将来使用：

```css
.go-premium {
  width: 31.25%; /* 300/960 */
  float: left;
  margin-top: 150px;
}
```

现在我认为我们准备好在浏览器中查看这个了。如果我稍微缩小浏览器窗口，布局不会立即破裂：

![](img/00232.jpeg)

但是如果我再稍微缩小浏览器窗口，那么最终它开始看起来非常糟糕：

![](img/00233.jpeg)

但是，我们确实取得了一些进展，因为我们的布局开始变得流体。介绍内容和呼吁行动按钮随着浏览器窗口的变小而变窄。最终，它们将开始重叠，但没关系；至少我们为这个顶部部分建立了一个流体基础。

现在让我们看看它下面的三列；它们在窗口变小时有点破碎：

![](img/00234.jpeg)

所以让我们看看`secondary-section`类中的 HTML。这三列在一个`div`标签中，类名为`wrapper`，最初也是 960 像素宽（但现在是最大宽度为 960px，宽度为 90%）：

```css
<section class="secondary-section grouping">
  <div class="wrapper">
    <div class="column">
      <figure>
        <img src="img/octopus-icon.png" alt="Octopus">
      </figure>
      <h2>The Octopus</h2>
      <p>...</p>
      <a href="#" class="button">Tenticals &raquo;</a>
    </div>
    <div class="column">
      <figure><img src="img/crab-icon.png" alt="Crab"></figure>
      <h2>The Crab</h2>
      <p>...</p>
      <a href="#" class="button">Crabby &raquo;</a>
    </div>
    <div class="column">
      <figure><img src="img/whale-icon.png" alt="Whale"></figure>
      <h2>The Whale</h2>
      <p>...</p>
      <a href="#" class="button">Stuart &raquo;</a>
    </div>
  </div><!-- end wrapper -->
</section>
```

我们将继续使用它作为我们的上下文，同时将我们的`.column`宽度从像素转换为百分比。一直到我们的 CSS 底部，我们看到每列宽度为`300px`：

```css
/****************
3 columns
****************/
.column {
  float: left;
  width: 300px; 
  margin-left: 30px; 
}
.column:first-child {
  margin-left: 0;
}
```

让我们在这里应用我们的公式。我们已经知道 300 除以 960 等于 31.25%，因为这是我们刚刚使用的确切计算：

```css
.column {
  float: left;
  width: 31.25%; /* 300/960 */
  margin-left: 30px; 
}
```

`margin-left`属性是`30px`，所以我们实际上要复制并粘贴 31.25%到这里，但是我们会移动小数点一位，并添加一个注释说明 30 除以 960：

```css
.column {
  float: left;
  width: 31.25%; /* 300/960 */
 margin-left: 3.125%; /* 30/960 */
}
```

我们在第一列上有一个`margin-left`属性的值为`0`。我们不必将 0 更改为百分比，因为 0、0 像素和 0%都是完全相同的东西——什么都没有：

```css
.column:first-child {
  margin-left: 0;
}
```

顺便说一句，我从来没有改变过任何高度、顶部和底部边距或填充，因为这些对我们来说并不重要。所以现在，如果我们刷新这个部分并使其变小，我们会看到我们的三列会与浏览器窗口一起按比例缩小：

![](img/00235.jpeg)

现在我们主页的一切都是流体的，除了我们的导航，我现在打算保持原样。我想要完全不同的处理方式，所以我会将其保持为固定宽度。

# 将填充更改为百分比

我们从来不必将填充左右更改为百分比，因为我们没有任何填充，但是这样做的过程非常相似。您仍然使用相同的公式——目标除以上下文等于结果。但是上下文现在有点不同；它是元素本身的宽度，而不是父元素的宽度，就像宽度和边距一样。唯一的例外是，如果元素本身没有定义宽度，您可以使用其父元素的宽度或通过确定父元素的宽度来确定元素本身的宽度：

![](img/00236.jpeg)

如果您使用`box-sizing`属性和 border-box 值，填充将不再计入元素的框模型宽度。因此，您可以将其保留为像素长度，只需将宽度和边距转换为百分比，因此`box-sizing: border-box`肯定会很有帮助。

# 鲨鱼电影页面上的流体网格

让我们搜索一些其他非百分比宽度/边距/填充。所以我们不用担心任何与垂直距离相关的东西，比如`height`、`margin-top`、`margin-bottom`、`padding-top`或`padding-bottom`。我们也不用担心任何值为`0`的东西。

我们将在`wrapper`规则集中遇到`auto`的左右边距：

```css
.wrapper {
  max-width: 960px;
  width: 90%;
  margin: 0 auto;
}
```

这不需要转换成百分比，因为`auto`会根据可用空间自动计算宽度，所以它和百分比一样好。

我们担心以下声明块中的`margin`属性：

```css
.content-block .figure {
  float: left;
  margin: 30px;
  border: 15px solid #fff;
  overflow: hidden;
}
```

这个规则集有一个`margin`为`30px`；它使用了单值语法。这意味着上下左右的边距都是`30px`。我们只想改变左右边距。所以我们可以使用双值语法。第一个值是指上下边距，第二个值是指左右边距。

```css
margin: 30px 30px;
```

记住`content-block .figure`是围绕我们的图片的元素，如下图所示。所以我们实际上是在尝试将`margin-right`和`margin-left`转换为百分比：

![](img/00237.jpeg)

如果我们在`shark-movies.html`中查看，我们会发现图片在`wrapper`中：

```css
<section id="jaws" class="content-block style-1 wave-border grouping">
  <div class="wrapper">
    <a href="#" class="figure">
      <img src="img/jaws.jpg" alt="Jaws movie">
    </a>
    <h1>Jaws</h1>
    <p>...</p>
    <a href="" class="button button-narrow button-alt float-right">Learn More</a>
  </div><!-- end of wrapper -->
</section>
```

所以，我们知道`wrapper`是 960 像素。到目前为止，我们的上下文很容易确定，因为我们的上下文一直是`wrapper`的宽度。

30 除以 960 得到 03.125，即 3.125%，所以我们会保存这个：

```css
.content-block .figure {
  float: left;  margin: 30px 3.125%; /* 30/960 */
  border: 15px solid #fff;
  overflow: hidden;
}
```

在我们的网站上，除了导航之外，所有硬像素长度都是百分比！并不是所有东西都必须是百分比才能实现响应式网页设计。我们做出了判断，决定我们会处理导航而不使用百分比宽度。这对于流体网格和响应式网页设计的许多其他决定都是正确的；真的没有一种大小适合所有的解决方案。您网站的每个组件都需要从桌面到移动端进行彻底思考，甚至更好的是，从移动端到桌面。因此，第一步，创建流体网格，目前已经完成。这是一个重要的步骤，因为它确保我们的设计将开始在所有屏幕尺寸上很好地适应。在下一节中，我们将看一下灵活的图片。

# 灵活的图片

我们已经创建了一个流体网格，这是响应式网页设计的第一个基础。基础二是响应式图片或灵活图片。我们希望我们的图片，或者至少某些图片，能像我们的 divs 和 sections 一样行为。我们希望它们是流体的或灵活的。

从我们的网站来看，我们可以注意到`章鱼`、`螃蟹`和`鲸鱼`的三张图片随着它们所在的列变小。另一方面，顶部的鲨鱼无论浏览器宽度如何，大小似乎都保持不变：

![](img/00238.jpeg)

我们导航中的图片不是灵活的。我们列中的三张图片是灵活的。我们将看一下导航中的图片，看看原因。但首先，让我们来看看保证响应式图片的三个因素：

+   将`img`标签放在一个容器中。最语义化的容器通常是`figure`标签，但它当然可以是任何元素。

+   使容器变得流体；给它一个百分比宽度。

+   给所有`img`标签或至少要成为流体或灵活的`img`标签分配`max-width`属性为`100%`。

# 章鱼、螃蟹和鲸鱼的图片

现在让我们来看看我们 HTML 文件中的一张图片。我们可以看到章鱼图片在一个容器中。容器是`figure`元素：

```css
<figure>
    <img src="img/octopus-icon.png" alt="Octopus">
</figure>
```

`figure`元素没有定义宽度，但它是一个占据整个容器宽度的块级元素。所以我们可以将`figure`宽度看作`100%`。它在`column` div 中：

```css
<div class="column">
    <figure>
        <img src="img/octopus-icon.png" alt="Octopus">
    </figure>
    <h2>The Octopus</h2>
    <p>...</p>
    <a href="#" class="button">Tenticals &raquo;</a>
</div>
```

如果我们在我们的 CSS 列中查看，我们会看到列宽为`31.25%`：

```css
/****************
3 columns
****************/
.column {
  float: left;
  width: 31.25%; /* 300/960 */  
  margin-left: 3.125%; /* 30/960 */
}
```

所以我们完成了第一步——我们的图像在容器内。然后我们有第二步——父元素是流动的。第三步是为所有图像分配最大宽度。

让我们滚动到我们的 CSS 文件的顶部。实际上，我在重置中有这个选择器，如下一个屏幕截图所示。它针对`img`、`iframe`、`video`和`object`，几乎所有类型的媒体。我已经为这个选择器分配了最大宽度为 100%：

```css
img, iframe, video, object {
  max-width: 100%; 
}
```

我将这个选择器作为我在每个项目中使用的重置或基本样式层的一部分。所以，玩得开心，让我们删除那个属性：

```css
img, iframe, video, object {
 /* max-width: 100%;*/
}
```

如果我们保存并查看我们的网站，当我们缩小浏览器窗口时，图像不会变小；它们将保持相同的大小并挤在一起：

![](img/00239.jpeg)

当我们重新添加`max-width: 100%`声明时，这些图像再次变得灵活。这表明任何图像的最大宽度只能是其容器的 100%。因此，随着容器变小，图像的宽度也会变小：

```css
img, iframe, video, object {
  max-width: 100%; 
}
```

![](img/00240.jpeg)

# 鲨鱼图片

鲨鱼图像不会变小有两个原因。让我们来检查一下。我们可以看到鲨鱼图像确实有一个直接的容器元素——一个`figure`标签。但是该容器是故意不是流动的：

![](img/00241.jpeg)

如果您点击图像容器，即`nav`标签，您会看到它会扩展到浏览器的全宽，说明容器不是流动的：

![](img/00242.jpeg)

如果我们检查图像本身，我们会看到它被分配了`160px`的宽度，这肯定会阻止它成为流动的：

![](img/00243.jpeg)

就我个人而言，我不喜欢在图片上设置宽度。即使我不希望这张图片是流动的，我也不希望它有宽度。在这种情况下，让我们做一些清理工作，并更改图片，使得图像元素的宽度为`160px`，这是图像的容器，而不是图像本身。

```css
nav img {
  width: 160px;
}
```

这更多地是我的个人偏好：

```css
nav figure {
  position: absolute;
  top: -20px;
  left: 50px;
  width: 160px;
  -webkit-animation: crazyShark 1s ease; 
  animation: crazyShark 1s ease;
}
/* nav img {
 width: 160px;
} */
```

我们故意将鲨鱼图像保留为固定宽度，因为它不一定需要缩小或增大以使设计具有响应性。我们将在本章后面单独处理页眉部分。

# 在鲨鱼电影页面上缩小图像

让我们看一下电影页面上的图像。当我们调整浏览器大小时，它们不会缩小。它们有固定的宽度：

![](img/00244.jpeg)

我认为它们应该缩小；它们在较小的浏览器尺寸下有点太大了。我们电影页面上的所有三个图像之所以不会缩小，是因为它们的父元素没有定义宽度。让我们使用 Ethan Marcotte 的公式-目标除以上下文等于结果。我们知道图像、标题 1、段落和了解更多按钮所占区域的上下文仍然是 960 像素宽，因为它在`wrapper`内部：

那么围绕图像的锚点标签的宽度是多少？如果我们查看我们的 CSS，我们有`.content-block .figure`，那里没有定义宽度：

```css
.content-block .figure {
  float: left;
  margin: 30px 3.125%; /* 30/960 */
  border: 15px solid #fff;
  overflow: hidden;
}
```

如果我们看一下`.figure`内部的图像，那里也没有定义宽度：

```css
.content-block .figure img {
  float: left;
  -webkit-transition: transform .25s ease-in-out;
  transition: transform .25s ease-in-out;
}
```

因此，我们必须利用 Chrome DevTools 的功能来确定围绕`img`元素的`a`元素的宽度是多少。如果我们悬停在图像本身上，我们会看到图像是 200 像素乘以 200 像素：

![](img/00245.jpeg)

如果我们实际上突出显示锚点，如下面的屏幕截图所示，DevTools 告诉我们宽度为 230 像素。您可以看到它在图像正上方的弹出气泡中。我们的宽度是 230 - 图像的 200 像素加上 15 像素的左边框和 15 像素的右边框。这是有道理的。

![](img/00246.jpeg)

现在我们要做的是，当我们将像素值转换为百分比时，我们要使用 230 作为我们的目标。我们还将不得不使用`box-sizing: border-box`。记住，正如你在﻿第一章中学到的*CSS 基础*中的*盒模型和块与内联元素*部分中所学到的，如果你将一个元素设置为`box-sizing: border-box`，那么`border`和`padding`会被计算到你定义的`width`中：

```css
.content-block .figure {
  float: left;
  margin: 30px 3.125%; /* 30/960 */
  border: 15px solid #fff;
  overflow: hidden;
  box-sizing: border-box;
}
```

230 除以 960 等于 0.23958333333333，所以我们将其转换为百分比，得到`23.98333333333%`：

```css
.content-block .figure {
  float: left;
  margin: 30px 3.125%; /* 30/960 */
  border: 15px solid #fff;
  overflow: hidden;
  box-sizing: border-box;
  width: 23.958333333333%; /*230/960*/
}
```

现在，如果我们刷新浏览器并将其缩小，我们会看到我们的图片变小了。现在可能看起来有点奇怪，但信不信由你，这正是我们想要的，所以很棒！

![](img/00247.jpeg)

现在让我们回到 CCS 代码。我们正在使用的这种技术非常有帮助，但我们可能会在整个代码中重复使用`box-sizing: border-box`。让我们从`.content-block .figure`规则集中完全删除它，并以稍微不同的方式将其添加到我们的重置中，应该在哪里。让我们将这个添加到我们样式表的重置部分：

```css
html {
  box-sizing: border-box;
}
*, *:before, *:after {
  box-sizing: inherit;
}
```

作为我们的重置的一部分，每个元素都将获得`box-sizing: border-box`。我们可以看到我们已经将它添加到我们的 HTML 元素中，并使用了通用（星号）选择器，正如你从我们的特异性规则部分中记得的那样，它适用于所有元素。我们只在`html`元素上应用`box-sizing: border-box`，但其他所有东西都将获得`box-sizing: inherit`。`html`是每个元素的父元素；因此，你正在继承`border-box`属性到每个元素。好了，我们从灵活的图片中稍微偏离了一下，但我们需要这样做来创造一个积极的前进路径。

因此，总结一下，我们的图片现在是灵活的，但在非常小的浏览器宽度下（比如手机或平板电脑等更小的设备），我们的网站并不完美。在下一节中，我们将找出如何使用媒体查询来处理这个问题。

# 媒体查询

响应式网页设计的前两个基础只能让你走得更远。最重要的基础是媒体查询。媒体查询基本上是你的 CSS 中的“if”语句或条件逻辑。例如，*如果*浏览器的宽度小于 500 像素，我们可以根据这些条件应用不同的规则集。媒体查询非常强大，因为在某些点上，我们的网站真的会崩溃并且看起来很糟糕，我们需要用它来修复这个问题。在本节中，我们将找出什么是媒体查询，并使用它来修复网站的剩余问题，特别是在更窄的宽度下。

还要考虑的一件事是，由于我们要缩小浏览器窗口来模拟平板电脑或移动设备，我们将没有太多的空间来查看 DevTools。你可以点击 3 个垂直点的图标，打开下拉菜单将 DevTools 移到右边：

![](img/00248.jpeg)

Chrome 会不时地更新 UI，所以图标可能对你来说看起来不一样。

现在我们可以将浏览器窗口缩小到任何更小的宽度，仍然有足够的空间来使用开发者工具：

![](img/00249.jpeg)

快速提示

如果你使用 Chrome DevTools 并缩小浏览器窗口，在右上角它会显示浏览器视口的宽度和高度。因此，在下面的图片中，你可以看到它提供了关于宽度的信息，为**691px**。

![](img/00250.jpeg)

我们网站目前的问题是，如果达到更窄的宽度，导航栏将会在鲨鱼后面，呼吁行动按钮将被挤压到网站标题中，三列也太窄了：

![](img/00249.jpeg)

所以我们将跳过导航栏，最后再回来处理。主要的网站标题和呼吁行动按钮是浮动的。在 1023 像素时，让我们使用媒体查询“取消浮动”这两个部分，并将它们堆叠在一起。

# 媒体查询的解剖

我将把所有媒体查询放在样式表的底部。您不必这样做，但我会这样做。媒体查询始终以`@media`开头。然后，它们有两个部分。第一部分是*媒体类型*。例如：

```css
@media screen
```

您还可以插入`print`，`screen`，`all`和其他一些选项。`print`仅适用于打印样式表，而`screen`仅适用于计算机屏幕（但不适用于打印输出）。`all`将适用于两者。如果我们完全省略`媒体类型` - 这是完全可以接受的 - 它将默认为`all`。

媒体查询的第二部分称为*媒体特征*，并确定何时使用媒体查询。我们需要将其与`@media screen`分开，使用“and”一词。例如：

```css
@media screen and (max-width: 1023px) { }
```

同样，这是类似于 JavaScript 中的 if 语句的条件逻辑。如果*媒体类型*和*媒体特征*都评估为 true，那么媒体查询内部的内容将被应用（是的，我们很快将在媒体查询的大括号内放置内容）。

`(max-width: 1023px)`表示如果浏览器窗口为 1,023px 或更低，则此媒体查询将应用（或评估为 true）。一旦屏幕宽度超过 1,023px，那么媒体查询将不再应用（或评估为 false）。您还可以使用`min-width`；它具有相反的效果，适用于所有大于 1,023px 的内容：

```css
@media screen (min-width: 1023px) { }
```

实际上，您也可以使用任何长度值以及许多其他值。通常，`max-width`和`min-width`与我们要做的事情非常契合，我们现在将坚持使用`max-width`。注意末尾的大括号。这几乎就像我们刚刚构建了一个 CSS 规则集：

```css
@media screen (max-width: 1023px) { }
```

在媒体查询的大括号内，我们可以开始编写纯粹的 CSS，只有在浏览器窗口同时（1）是屏幕和（2）1,023px 或更少时才会应用。让我们将`float: none`和`width: auto`都添加到`intro-content`和`go-premium`：

```css
@media screen and (max-width: 1023px){
 .intro-content {
 float: none;
 width: auto;
 }
 .go-premium{
 float: none;
 width: auto;
 }
}
```

`auto`值是`width`属性的默认值，因此它实际上使这些块元素跨越整个可用宽度，这将是`wrapper`的 100％。`auto`关键字意味着它将自动计算值。`auto`根据其配对的属性具有不同的值。在这种情况下，它基本上与“100％”相同。

现在我们可以看到我们的介绍内容不再浮动，宽度是全宽，而`go-premium`按钮也是一样的：

![](img/00251.jpeg)

但是，`go-premium`按钮和`intro-content`之间有一个很大的空间，我们需要摆脱它：

![](img/00252.jpeg)

为了解决这个问题，我们将添加：

```css
@media screen and (max-width: 1023px){
  .intro-content {
    float: none;
    width: auto;
 padding-bottom: 0;
 margin-bottom: 30px;
  }
  .go-premium {
    float: none;
    width: auto;
  }
}
```

在`go-premium`按钮本身上，我们将删除顶部边距：

```css
@media screen and (max-width: 1023px){
  .intro-content {
    float: none;
    width: auto;
    padding-bottom: 0;
    margin-bottom: 30px;
  }
  .go-premium {
    float: none;
    width: auto;
    margin-top: 0;
  }
}
```

我们已经有了我们的标题，副标题，文本和按钮，一切看起来都很好：

![](img/00253.jpeg) ![](img/00254.jpeg)

# 考虑 iPad 和其他平板电脑尺寸

我们选择 1,023 作为断点，因为这只是低于横向放置的 iPad 的宽度的一个像素。

![](img/00255.jpeg)

这样，我们的媒体查询将适用于所有小于 1,024p 的宽度和设备。截至 2017 年，我会*猜测* iPad 是 - 如果不是最受欢迎的平板电脑 - 其中一个最受欢迎的平板电脑。我可以更肯定地说 iPad 绝对不是唯一受欢迎的平板电脑。事实上，令人惊讶的是，有多少不同的平板设备和宽度，因此您可能不希望分别使用 1,024 和 768 作为媒体查询的基础。找出您的布局通常从何处开始中断或看起来有趣，并确定媒体查询的逻辑放置位置。然后，在 iPad 和任何其他设备或模拟器上测试您的网站，以确保您的网站看起来不错。在我们的情况下，我们将使用 1,023 作为基线，因为在 1,024 时，布局仍然看起来不错。

# 将我们的三列添加到媒体查询

现在，我们只需要将所有的 CSS 添加到我们的媒体查询中，以使网站的其余部分看起来不错，从三列区域开始。正如您在下面的截图中所看到的，这些区域太紧凑了：

![](img/00256.jpeg)

为较小的设备创建内容管道是一种标准做法，摆脱任何多列浮动布局。因此，我们将再次从`.column`中删除浮动，并通过指定`auto`关键字使宽度成为父元素的全宽。让我们转到 CSS 的底部并更新媒体查询：

```css
@media screen and (max-width: 1023px){
  .intro-content {
    float: none;
    width: auto;
    padding-bottom: 0;
    margin-bottom: 30px;
  }
  .go-premium {
    float: none;
    width: auto;
    margin-top: 0;
  }
 .column {
 float: none;
 width: auto;
 }
}
```

现在当我们转到我们的网站时，这三列中的每一列都将占满整个宽度，不会浮动：

![](img/00257.jpeg)

问题是，我们可能应该将这段文字和图像本身居中。因此，首先我们将在媒体查询中首先针对`.column figure`进行定位以居中图像：

```css
@media screen and (max-width: 1023px){
  .intro-content {
    float: none;
    width: auto;
    padding-bottom: 0;
    margin-bottom: 30px;
  }
  .go-premium {
    float: none;
    width: auto;
    margin-top: 0;
  }
  .column {
    float: none;
    width: auto;
  }
 .column figure {
 margin: 0 auto;
 max-width: 250px;
 width: 100%;
 }
}
```

通过使用`auto`关键字设置左右边距，并且不仅设置`width: 100%`而且还设置`max-width: 25px`，我们能够使图像居中：

![](img/00258.jpeg)

添加`width: 100%`确保如果`wrapper`容器小于 250px（图像的最大尺寸），则图像宽度将是其容器的`100%`；在非常窄的宽度下起到一种安全保障作用。

接下来，让我们简单地使用`text-align: center`来居中标题：

```css
@media screen and (max-width: 1023px){
  .intro-content {
    float: none;
    width: auto;
    padding-bottom: 0;
    margin-bottom: 30px;
  }
  .go-premium {
    float: none;
    width: auto;
    margin-top: 0;
  }
  .column {
    float: none;
    width: auto;
  }
  .column figure {
    margin: 0 auto;
    max-width: 250px;
    width: 100%;
  }  .column h2 {
 text-align: center;
 }
}
```

看起来不错：

![](img/00259.jpeg)

我们的响应式设计还可以采用另一种方法-移动优先方法。

# 移动优先方法

通常，这是最佳实践。我们希望在同一时间或在桌面体验之前考虑较小的显示屏，正如名称所示。这样，内容和设计可以在设计和构建过程中充分考虑移动设备及其所有约束，因此可以完全实现。Mobile first 不仅涉及我们如何编写 CSS。但这是移动优先的代码部分的一般思路：将所有针对最小设备的 CSS 放在任何媒体查询之外。然后使用媒体查询来针对越来越大的设备。

![](img/00260.jpeg)

这意味着使用`min-width`媒体查询为较大的显示屏添加额外的 CSS。例如，我们的布局默认情况下不会有任何浮动；相反，它将是一个单一的内容隧道，这通常是移动设备的标准。然后我们的媒体查询，使用`min-width`而不是`max-width`，将添加浮动，这些浮动将应用于更宽的屏幕宽度以创建多列布局。

有关移动优先方法的更多信息，请查看 Luke Wroblewski 关于该主题的定义书籍-*Mobile First*-可在[abookapart.com](https://abookapart.com/)网站上找到：

![](img/00261.jpeg)

# 解决导航问题

现在让我们谈谈这个导航。解决导航问题并不像使用媒体查询解决我们刚刚解决的问题那么容易。在某些宽度下，我们的鲨鱼会挡住导航，而且在某个时候，我们还需要对下拉菜单做些处理。此外，如果我们要添加更多的导航项，那么在更宽的宽度下，我们将遇到这个问题。

![](img/00262.jpeg)

我们遇到了解决导航的第一个真正挑战。出现了一些特定的响应式设计模式。让我们谈谈 Brad Frost 策划的网站-[bradfrost.github.io/this-is-responsive](http://bradfrost.github.io/this-is-responsive/)：

![](img/00263.jpeg)

这基本上是一组用于响应式网页设计的模式。它有很多不同的处理导航的方式供您探索。让我们看看*模式*下的第一个，称为切换。在较宽的宽度下，菜单看起来有点像我们的；顶部只是一些导航项：

![](img/00264.jpeg)

在较窄的浏览器宽度下，导航项被替换为菜单链接：

![](img/00265.jpeg)

当您单击链接时，它会展开并隐藏菜单：

![](img/00266.jpeg)

这通常是我们想要在我们的网站上做的事情；这将需要更多地使用媒体查询和一些 JavaScript 或 jQuery 来显示和隐藏导航点击。准备好挑战了吗？好的，让我们做到！

我们已经使用媒体查询根据屏幕宽度更改了我们的 CSS。这非常有用，因为它修复了我们在较窄宽度下的大部分问题。我们仍然需要在较小宽度下修复导航。在下一节中，我们将使用媒体查询大幅改变我们的导航，使其隐藏，除了一个可以点击以显示带有 jQuery 的导航的菜单图标。

# 移动菜单

到目前为止，在本章中，您已经了解了 Ethan Marcotte 关于响应式网页设计的三个基本原则-流体网格，可伸缩的图像和媒体查询。在某种程度上，这在某种程度上是容易的部分。困难的部分是弄清楚多种棘手的设计挑战；例如，在移动设备上该怎么处理我们的菜单，特别是当我们决定添加更多菜单时。幸运的是，这是一个体面的设计模式已经出现的领域。我们将放弃水平菜单，而是显示导航栏上的每个按钮的隐藏菜单，而是通过点击或触摸来激活的隐藏菜单。被点击或触摸后，隐藏菜单将垂直滑下并显示所有菜单选项。我们将通过为移动导航设置样式来实现这一点。然后，我们将隐藏导航并添加触发移动菜单打开和关闭的菜单图标。最后，我们将将我们的 HTML 链接到 jQuery CDN 和我们的脚本文件，并编写一些基本的 jQuery 来实现这一点。

# 在打开状态下为移动导航设置样式

在样式表的最底部，我将添加一个针对`900px`或更小的新媒体查询：

```css
@media screen and (max-width: 1023px){
  ...
}/* end of media query */

@media screen and (max-width: 900px) {

}
```

请注意，我还在第一个媒体查询的结束大括号处添加了一个注释。这很有用，这样我们就不会迷失我们的第一个媒体查询的结束位置。新的移动导航只会在新的断点`900px`宽度时触发。那时它开始看起来很奇怪并且开始破损。首先，我不想导航固定在顶部，所以让我们摆脱固定定位并将其替换为默认的静态位置：

```css
@media screen and (max-width: 900px) {
 nav {
 position: static;
 }
}
```

静态，你可能还记得之前的部分，是`position`属性的默认值，因此它基本上关闭了`fixed`定位，并将其返回为根据*正常流*行为的元素。

接下来，让我们告诉`primary-nav`的所有直接列表项向右浮动，而不是向左。我们还将给它一个完整的宽度，因为这些浮动元素只会占用所需的宽度，类似于内联元素，因此我们将告诉它们通过将宽度设置为`100％`来占用整个可用宽度：

```css
@media screen and (max-width: 900px) {
  nav {
    position: static;
  }
 .primary-nav > li {
 float: right;
 width: 100%;
 }
}/* close media query */
```

我还在这个媒体查询的结束大括号处添加了一个注释，以便不会迷失。

现在让我们专注于锚点。让我们将文本对齐到右侧，标准化填充，移除左侧的边框并在底部添加边框，并将字体大小调整为 13px，并将宽度设置为 100％：

```css
@media screen and (max-width: 900px) {
  nav {
    position: static;
  }
  .primary-nav > li {
    float: right;
    width: 100%;
  }
 .primary-nav li a {
 text-align: right;
 padding: 15px 25px 15px 0;
 width: 100%;
 border-bottom: 1px solid #e7e7e7;
 border-left: none;
 font-size: 13px;
 }
}/* close media query */
```

好的，这是在应用此 CSS 之前导航的样子：

![](img/00267.jpeg)

这是刷新后的样子。现在它开始看起来像移动导航了：

![](img/00268.jpeg)

问题在于 100％的宽度实际上并没有起作用。嗯，实际上它起作用了；锚点是其容器的 100％，但它们的容器也需要是 100％。因此，让我们将`primary-nav`的宽度设置为 100％：

```css
@media screen and (max-width: 900px) {
  nav {
    position: static;
  }
  .primary-nav > li {
    float: right;
    width: 100%;
  }
  .primary-nav li a {
    text-align: right;
    padding: 15px 25px 15px 0;
    width: 100%;
    border-bottom: 1px solid #e7e7e7;
    border-left: none;
    font-size: 13px;
  }  .primary-nav {
 width: 100%;
 }
}/* close media query */
```

我们可以在待办清单上划掉这一项：

![](img/00269.jpeg)

这确实像是一个菜单。房间里的大象是鲨鱼只是占用了太多的空间。让我们来修复一下。

让我们在针对鲨鱼的媒体查询的底部添加一个新的选择器。我们将使它变得更小，向上移动，并使用顶部和左侧位置的偏移属性将其紧贴到左侧，因为这已经是一个绝对定位的元素：

```css
@media screen and (max-width: 900px) {
  nav {
    position: static;
  }
  .primary-nav > li {
    float: right;
    width: 100%;
  }
  .primary-nav li a {
    text-align: right;
    padding: 15px 25px 15px 0;
    width: 100%;
    border-bottom: 1px solid #e7e7e7;
    border-left: none;
    font-size: 13px;
  }  .primary-nav {
    width: 100%;
  }
  nav figure {
 width: 100px;
 top: 0;
 left: 20px;
 }
}/* close media query */
```

看起来不错：

![](img/00270.jpeg)

接下来要修复的是下拉菜单。这样不行：

![](img/00271.jpeg)

我们现在需要做一个设计决定。我们应该隐藏下拉菜单，不让任何移动用户访问它吗？我们可以。但这对移动用户不公平。我们显然可以保持它如上所示，但我认为把它融入到`primary-nav`中会更好地为移动用户服务。我想让它看起来像其他主要菜单。所以我们要针对`.has-submenu ul`选择器。我们将在媒体查询的底部添加这个规则集。我们将把`position`属性从`absolute`改为`static`，把`display`属性改为`block`，移除`border`和`border-radius`属性，并让`width`横跨整个宽度：

```css
@media screen and (max-width: 900px) {
  nav {
    position: static;
  }
  .primary-nav > li {
    float: right;
    width: 100%;
  }
  .primary-nav li a {
    text-align: right;
    padding: 15px 25px 15px 0;
    width: 100%;
    border-bottom: 1px solid #e7e7e7;
    border-left: none;
    font-size: 13px;
  }  .primary-nav {
    width: 100%;
  }
  nav figure {
    width: 100px;
    top: 0;
    left: 20px;
  }  .has-submenu ul {
 position: static;
 display: block;
 border: none;
 border-radius: 0;
 width: 100%;
 }
}/* close media query */
```

现在我们有这个：

![](img/00272.jpeg)

哇！看起来很不错。我们也可以取消动画，因为它不再需要。我们将在媒体查询内部添加一个新的选择器，并将`-webkit-animation`和`animation`设置为`none`；这个关键字将取消动画：

```css
@media screen and (max-width: 900px) {
  nav {
    position: static;
  }
  .primary-nav > li {
    float: right;
    width: 100%;
  }
  .primary-nav li a {
    text-align: right;
    padding: 15px 25px 15px 0;
    width: 100%;
    border-bottom: 1px solid #e7e7e7;
    border-left: none;
    font-size: 13px;
  }  .primary-nav {
    width: 100%;
  }
  nav figure {
    width: 100px;
    top: 0;
    left: 20px;
  }  .has-submenu ul {
    position: static;
    display: block;
    border: none;
    border-radius: 0;
    width: 100%;
  }  .has-submenu:hover ul {
 -webkit-animation: none;
 animation: none;
 }
}/* close media query */
```

我们不再有动画效果。"电影"菜单的悬停状态以一种奇怪的方式覆盖了鲨鱼，但很快在我们添加汉堡菜单图标时就会得到修复：

![](img/00273.jpeg)

移动导航在打开状态下已经完成；现在我们需要隐藏它，并添加汉堡图标来触发它的打开和关闭。

# 添加汉堡菜单图标

让我们在`index.html`文件和`shark-movies.html`文件中的`primary-nav`正上方添加一个`div`标签。我们给它一个类名`mobile-menu-icon`；这很重要：

```css
<!--
===============
Nav
===============
-->
<nav class="grouping">
    <figure>
        <img src="img/sharky.png" alt="Shark">
    </figure>
    <div class="mobile-menu-icon"></div>
    <ul class="primary-nav grouping">
        <li><a href="#">Home</a></li>
        <li class="has-submenu"><a href="shark-movies.html">Movies</a>
            <ul>
                <li class=""><a href="">Jaws</a></li>
                <li class=""><a href="">Sharknado</a></li>
                <li class=""><a href="">Open Water</a></li>
            </ul>
       </li>
        <li><a href="#">Species</a></li>
        <li><a href="#">Chum</a></li>
    </ul>
</nav>
```

当我们应用这个时，浏览器中什么都没有显示出来，因为这只是一个空的`div`标签。让我们使用背景图片添加图标。我们不会把这个放在媒体查询中；实际上，我们会移动到 CSS 中原始的 nav 所在的位置，并添加这个规则集：

```css
/****************
nav
****************/
nav {
  background-color: #fff;
  position: fixed;
  left: 0;
  right: 0;
  top: 0;
  z-index:1;
  box-shadow: 0 8px 15px rgba(0, 0, 0, 0.1);
}
nav figure {
  width: 160px;
  position: absolute;
  top: -20px;
  left: 50px;
  -webkit-animation: crazyShark 1s ease; 
  animation: crazyShark 1s ease;
}
.mobile-menu-icon {
 background: url('../images/mobile-menu-icon.png') 0 0 no-repeat;
}
```

我们已经在我们的图片文件夹中有这张图片。我们使用零和零作为我们的背景位置和 no-repeat，以确保这张图片不会自动重复。除非我们添加宽度和高度，否则在浏览器中什么都不会显示。我们知道这张图片宽 30 像素，高 26 像素，所以我们将使用这些确切的尺寸：

```css
.mobile-menu-icon {
  background: url('../images/mobile-menu-icon.png') 0 0 no-repeat;
 width: 30px;
 height: 26px;
}
```

现在当我们保存并刷新时，我们可以看到三条杠图标位于浏览器窗口的顶部：

![](img/00274.jpeg)

我们还想把它移到右边，并给它一些距离，使用一些上、右和下的 margin。我们还可以改变光标，这样它看起来就有了不同的外观。让我们把这些属性添加到`mobile-menu-icon`中：

```css
.mobile-menu-icon {
  background: url('../images/mobile-menu-icon.png') 0 0 no-repeat;
  width: 30px;
  height: 26px;
  float: right;
 margin: 10px 15px 10px 0;
 cursor: pointer;
}
```

在刷新浏览器之前，我们可以看到我们只是在图标上方普通的光标悬停：

![](img/00275.jpeg)

刷新后，它移到右边，现在有一个指针类型的光标，表示它是可点击的：

![](img/00276.jpeg)

显然，我们在移动设备上看不到这个，但在桌面设备或桌面浏览器上，这是一个小小的好东西。

# 隐藏菜单

现在让我们默认隐藏菜单。我们不再使用`display: none`，因为如我之前提到的，这对无障碍原因来说并不好。让我们探索另一种更有创意地隐藏内容的技术，以便屏幕阅读器仍然可以找到并宣布它。我们将回到媒体查询内部，`.primary-nav`内部。我们将说这个元素的高度是零：

```css
@media screen and (max-width: 900px) {
  .intro-content {
    margin-top: 50px;
  }
  nav {
    position: static;
  }
  .primary-nav {
    width: 100%;
    max-height: 0;
 overflow: hidden;
    -webkit-transition: all ease-out .35s;
    -moz-transition: all ease-out .35s;
    -o-transition: all ease-out .35s;
    transition: all ease-out .35s;
  }
...
```

以下是前面代码的输出：

![](img/00277.jpeg)

就是这样。现在我们要做的是在点击时激活菜单。

# 使用 jQuery 在点击时触发菜单

在这个阶段，我们需要链接到 jQuery 和我们自己的 JavaScript 文件。我们将在 HTML 的底部，在闭合的`</body>`和`</html>`标签之前做这个。复制一个链接到托管在谷歌网站上的 jQuery CDN。在此之下，添加一个链接到我们自己的 JS 文件。我们将把这个文件放在`js`文件夹中，并将文件命名为`scripts.js`：

```css
<script src="img/jquery.min.js"></script>
<script src="img/scripts.js"></script>
</body>
</html>
```

另外，让我们将这个复制到`shark-movies.html`的相同位置。我们也要创建一个新的 JavaScript 文件。

在 Sublime Text 中创建新文件的简单方法是使用*Cmd* + *N*（在 Mac 上）或*Ctrl* + *N*（在 Windows 上）。*Cmd* + *S*（在 Mac 上）或*Ctrl* + *S*（在 Windows 上）将让您保存名称并保存文件。

![](img/00278.jpeg)

我们将其保存在`js`文件夹中：

![](img/00279.jpeg)

我们将文件命名为`scripts.js`。

好的，现在让我们写一些 jQuery。如果这里的一切对你来说都不太有意义，不要担心。我不会详细讨论这个话题，因为这超出了本书的范围，但这对我们的响应式设计是必需的。我们要粘贴到我们的新`scripts.js`文件中的是一个在 DOM 准备就绪时触发的函数：

```css
$(document).ready(function() {

});//end doc ready
```

我们想把我们写的代码放在这个函数里。这只是告诉脚本在网页准备就绪之前等待，然后再执行其中的代码。这对于 jQuery 来说是标准的。因此，让我们将其粘贴到我们的函数中：

```css
$(document).ready(function() {
  $(".mobile-menu-icon").on("click", function(){
    $(".primary-nav").toggleClass("active");
    $(this).toggleClass("open");
  });
});//end doc ready
```

首先，我们在这里有一个 jQuery 函数，它专门针对`mobile-menu-icon`类，当你点击该元素时：

```css
$(".mobile-menu-icon").on("click", function(){
```

一旦点击了该元素，就会执行两行代码。我们首先要关注的是`primary-nav`，并切换一个名为`active`的类：

```css
$(".primary-nav").toggleClass("active");
$(this).toggleClass("open");
```

因此，如果您点击汉堡菜单，它将向`primary-nav`添加一个`active`类。如果您再次点击它，它将删除它，并为我们保持这样做，这很好。下一行是针对`$(this)`。在这里，`$(this)`指的是我们点击的任何东西。在这种情况下，我们点击的是`mobile-menu-icon`，并在其上切换一个名为`open`的类。查看 DevTools 中的移动菜单图标和`primary-nav`：

![](img/00280.jpeg)

DevTools 中的代码行在以下截图中突出显示：

![](img/00281.jpeg)

这两者都应该添加类。当我们点击汉堡菜单图标时，我们会看到`mobile-menu-icon`获得`open`类，`primary-nav`获得`active`类：

![](img/00282.jpeg)

当我们再次点击它时，它们两者都消失了。所以现在我们可以继续以我们需要的方式定位这些类。让我们回到我们的 CSS。我们想要定位`mobile-menu-icon`在打开状态下。因此，我们将该选择器添加到 CSS 的 nav 部分中：

```css
@media screen and (max-width: 900px) {
  ...
  .mobile-menu-icon {
    background: url('../images/mobile-menu-icon.png') 0 0 no-repeat;
    width: 30px;
    height: 26px;
    float: right;
    margin: 10px 15px 10px 0;
    cursor: pointer;
  }
  .mobile-menu-icon.open { }
}/* end of media query */
```

我们要做的就是改变背景图片：

```css
.mobile-menu-icon.open { 
  background-image: url('../images/mobile-menu-close-icon.png');
}
```

现在当我们点击汉堡图标时，我们得到 x 图标，当我们再次点击它时，我们得到菜单图标。所以很好：

![](img/00283.jpeg)

现在我们想要定位`primary-nav.active`选择器，因此让我们将其添加到我们的 CSS 中，并给它一些高度：

```css
@media screen and (max-width: 900px) {
  .intro-content {
    margin-top: 50px;
  }
  nav {
    position: static;
  }
  .primary-nav {
    width: 100%;
    max-height: 0;
    overflow: hidden;
  }
  .primary-nav.active {
 max-height: 350px;
 }
  ...
}
```

现在当我们点击图标时，我们得到我们的菜单：

![](img/00284.jpeg)

当我们再次点击它时，它消失了：

![](img/00285.jpeg)

目前，图像出现并立即消失，所以我们想为其添加一个过渡效果。让我们去`.primary-nav`并添加一个过渡效果：

```css
@media screen and (max-width: 900px) {
  .intro-content {
    margin-top: 50px;
  }
  nav {
    position: static;
  }
  .primary-nav {
    width: 100%;
    max-height: 0;
    overflow: hidden;
 -webkit-transition: all ease-out .35s;
 transition: all ease-out .35s;
  }
  .primary-nav.active {
    max-height: 350px;
  }
  ...
}
```

现在我们应该有一个丝般顺滑的过渡效果，当我们点击菜单图标时，隐藏的导航会向下滑动和向上滑动。

我们的移动导航已经完成，有相当多的 CSS 和一点 JavaScript，并且我们的网站现在具有广泛的响应性。我们只需要做一件事 - 我们需要在移动设备上测试我们的网站。我们会注意到，结果与我们将浏览器调整为手机或平板电脑宽度时非常不同。幸运的是，解决方案非常简单 - `viewport` meta 标签。

# 视口 meta 标签

我们的响应式网站几乎完成了。除了我们还没有在移动设备上测试过它。在本节中，让我们使用 Chrome 的移动设备模拟器来测试我们的设计，然后看看并尝试理解`viewport` meta 标签。

# 在移动设备上测试我们的响应式设计

在手机上测试的一种方法是这样的-让您的网站上线并在实际手机或平板上测试。一个更简单的方法是在手机上进行简单测试（但可能略微不太准确）是使用 Chrome 的设备模拟器。在 DevTools 中有一个设备图标：

![](img/00286.jpeg)

一旦你点击了那个，你就可以选择一个手机。我们可以看到我们的网站，但它看起来并不像我们刚刚将浏览器窗口最小化到手机大小时那样：

![](img/00287.jpeg)

发生的情况是，大多数移动设备会尝试缩小您的网站以适应手机屏幕，然后如果您的网站不具有响应性，它将看起来像桌面版本，只是小得多。所以一个显而易见的问题是我没有看到移动导航。有一个非常简单的解决方案-`viewport`元素。我将把它复制粘贴到`index`和`shark-movies`页面中：

```css
<!doctype html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">

<!-- description -->
  <title>Section 6-Becoming Responsive - Mastering CSS</title>

<!-- stylesheets -->
  <link rel="stylesheet" href="css/style.css">

<!-- mobile -->
 <meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0">

<!-- stylesheets for older browsers --> 
  <!-- ie6/7 micro clearfix -->
  <!--[if lte IE 7]>
    <style>
    .grouping {
        *zoom: 1;
    }
    </style>
  <![endif]-->
  <!--[if IE]>
    <script src="img/html5.js"></script>
  <![endif]-->
</head>
```

这只是一个带有`viewport`名称的`meta`元素；我们稍后会回到这个问题。现在，让我们看看当我们刷新浏览器时会发生什么：

![](img/00288.jpeg)

实际上我们得到了移动版本，所以这看起来好多了，解决了我们遇到的问题。

# viewport 元标签的解剖

让我们来看看这个元标签：

```css
<meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0">
```

在这里，我提供了一个带有值`viewport`的名称属性。然后，我有一个`content`属性，里面提供了一些不同的东西。第一件事是`width=device-width`。这基本上意味着“请不要在移动设备上缩小我的页面，因为我正在使用媒体查询来处理这个问题。谢谢！”第二件事是`initial-scale=1.0`，基本上是说这个-将其大小调整到设备的宽度，而不多。最后，我们有`minimum-scale=1.0`。这在您旋转手机时会有所帮助，因此网站在设备的宽度从*纵向*模式更改为*横向*模式以及反之后仍然保持在设备的宽度。`viewport`元标签还有更多内容。我们可以添加`user-scalable=no`：

`user-scalable=no`这个术语不允许用户在手机上放大或缩小。拥有这个属性的网站可能会非常恼人，这就是为什么我们不会在我们的网站上包含它的原因。

总之，我建议将`viewport`元标签添加到您的网站的模板中，以便在每个网站上使用。实际设备测试没有替代品，因为实际手机和模拟器永远不会完全相同。

# 总结

在这一章中，我们涵盖了响应式网页设计的核心概念，这将使您的网站在任何设备上都能看起来很好。您学到了流体网格和灵活图片是使网站适应所有屏幕尺寸的第一步。我们现在了解了媒体查询如何确保网站在较窄的宽度下看起来很好。我们还使用 jQuery 创建了一个移动菜单，以便在点击时触发菜单。最后，我们在 Chrome 的移动设备模拟器上测试了我们的设计，并学会了如何使用`viewport`元标签来确保我们的网站在移动设备上具有响应性。我强烈建议您在自己使用这些技术时，从设计过程的一开始就考虑移动体验。在下一章中，我们将讨论网络字体。
