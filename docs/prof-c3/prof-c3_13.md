# 第十三章：代码优化

本章是关于一般创建代码过程和与每个步骤相关的过程。这个过程有几个一般阶段，我们将研究如何在每个阶段优化代码。

在本章中，我们将涵盖以下主题：

+   在创建的每个步骤中进行代码优化

+   如何在你的代码库中保持代码

+   如何优化 SASS 代码

+   如何在 CSS/SASS 代码中使用简写形式

+   如何准备代码以用于生产

# 自我优化

优化过程是在你开始编写代码时开始的。在编写代码时意识到可以优化的内容以及它在编写代码时应该如何出现是至关重要的。在编写过程之后，当你开始优化时，重构和重组代码可能会非常困难。但是构建代码并自动附加优化过程是很容易的。在编写代码时可以执行哪些过程？

+   使用简写形式

+   省略使用`!important`

+   省略使用 ID

## 在将代码发布之前的几个步骤

在代码创建过程中，有一些可重复的步骤：

+   编写代码

+   测试代码

+   将代码发布

这些过程有时是可重复的，特别是当它们与 Eric Ries 的《精益创业》方法论和多阶段项目相关时。在将代码发布之前，你需要记住这几个步骤：

+   检查是否使用了简写形式

+   检查元素/声明是否重复

+   检查元素/声明是否在 HTML 中使用（僵尸选择器）

+   检查`!important`的出现（如果可能的话，尝试省略它们）

+   检查代码是否被压缩

这个列表非常基础。在接下来的章节中，我们将运行优化过程和用法，检查所有可能性。

## 使用简写形式

在编写和构建过程中，简写形式有助于压缩代码。在 CSS 中使用简写形式，你可以节省很多字符，使代码更加精简。让我们来看一下简写形式的概述。

### 填充/边距的简写形式

你写填充和边距时有多少次使用了完整形式？你有多少次看到别人的代码没有使用填充和边距的简写形式而感到紧张？是的！它可能会让你紧张，因为这是对 CSS 的浪费！让我们从 SASS 中对元素填充的简单描述开始：

```css
.element
  padding:
    top: 10px
    right: 20px
    bottom: 30px
    left: 40px
```

它会给你类似这样的 CSS 代码：

```css
.element {
    padding-top: 10px;
    padding-right: 20px;
    padding-bottom: 30px;
    padding-left: 40px;
}
```

以下是用 CSS 简要描述它的方法：

```css
.element 
  padding: 10px 20px 30px 40px
```

一般来说，填充可以描述如下：

```css
padding: top right bottom left
```

你也可以用同样的方法处理边距：

```css
.element
margin:
    top: 10px
    right: 20px
    bottom: 30px
    left: 40px
```

它会给你类似这样的 CSS 代码：

```css
.element {
    margin-top: 10px;
    margin-right: 20px;
    margin-bottom: 30px;
    margin-left: 40px;
}
```

以下是用 CSS 简要描述它的方法：

```css
.element
margin: 10px 20px 30px 40px
```

一般来说，边距可以描述如下：

```css
margin: top right bottom left
```

让我们用另一个例子：

```css
.element
  margin:
    top: 10px
    right: 20px
    bottom: 10px
    left: 20px
```

编译为 CSS：

```css
.element {
  margin-top: 10px;
  margin-right: 20px;
  margin-bottom: 10px;
  margin-left: 20px; 
}
```

正如你所看到的，有两对值。当上边距/填充的值在底部值中重复，并且左值等于右值时，你可以使用简写版本：

```css
.element
  margin: 10px 20px
```

当编译为 CSS 时，它看起来像这样：

```css
.element {
  margin: 10px 20px; 
}
```

如你所见，这个版本是被压缩的，最终基于这个模式：

```css
margin: top_bottom_value left_right_value
```

### 边框的简写形式

让我们从边框的基本描述开始，然后我们可以扩展它：

```css
.element
    border:
    style: solid
    color: #000
    width: 10px
```

这是编译后的 CSS：

```css
.element {
  border-style: solid;
  border-color: #000;
  border-width: 10px; 
}
```

这个类将在框周围创建一个边框，它将是实心的，宽度为`10px`，颜色为黑色。因此，让我们创建一个包括所有边框（上、右、下和左）的类，定义样式颜色和宽度：

```css
.element
  border:
    top:
      style: solid
      color: #000
      width: 1px

    right:
      style: solid
      color: #f00
      width: 2px

    bottom:
      style: solid
      color: #0f0
      width: 3px

    left:
      style: solid
      color: #00f
      width: 4px
```

CSS：

```css
.element {
  border-top-style: solid;
  border-top-color: #000;
  border-top-width: 1px;

  border-right-style: solid;
  border-right-color: #f00;
  border-right-width: 2px;

  border-bottom-style: solid;
  border-bottom-color: #0f0;
  border-bottom-width: 3px;

  border-left-style: solid;
  border-left-color: #00f;
  border-left-width: 4px; 
}
```

因此，如果你想让这个代码变得更短一点，你可以使用全局定义的边框简写。代码如下：

```css
.element
  border: 1px solid #000
```

CSS：

```css
.element {
  border: 1px solid #000; 
}
```

和方向。代码将看起来像这样：

```css
.element
border:
    top: 1px dotted #000
    right: 2px solid #f00
    bottom: 3px dashed #0f0
    left: 4px double #00f
```

编译：

```css
.element {
    border-top: 1px dotted #000;
    border-right: 2px solid #f00;
    border-bottom: 3px dashed #0f0;
    border-left: 4px double #00f;
}
```

有一种方法可以以与我们定义填充和边框相同的方式来描述样式/宽度/颜色：

```css
.element
  border:
    style: dotted solid dashed double
    width: 1px 2px 3px 4px
    color: #000 #f00 #0f0 #00f
```

编译：

```css
.element {
    border-style: dotted solid dashed double;
    border-width: 1px 2px 3px 4px;
    border-color: #000 #f00 #0f0 #00f;
}
```

现在，让我们收集关于`border-radius`的信息。边框半径的全局定义如下：

SASS：

```css
.element
  border-radius: 5px
```

CSS：

```css
.element {
    border-radius: 5px;
}
```

在另一行和另一个值中描述每个角：

```css
.element
  border:
    top:
      left-radius: 5px
      right-radius: 6px
    bottom:
      left-radius: 7px
      right-radius: 8px
```

CSS：

```css
.element {
  border-top-left-radius: 5px;
  border-top-right-radius: 6px;
  border-bottom-left-radius: 7px;
  border-bottom-right-radius: 8px; 
}
```

现在，前面的代码可以用以下方式描述得更短：

```css
.element
  border-radius: 5px 6px 7px 8px
```

CSS：

```css
.element {
  border-radius: 5px 6px 7px 8px;
}
```

### 字体样式中的简短形式

字体在每个段落标题链接中都有描述。如您所见，在代码中这么多重复出现的情况下使用简写是很好的。在这里，我们对一个示例元素的字体和行高进行了简单描述：

```css
.element
font:
    size: 12px
    family: Arial
    weight: bold
  line-height: 1.5
```

CSS：

```css
.element {
  font-size: 12px;
  font-family: Arial;
  font-weight: bold;
  line-height: 1.5; 
}
```

让我们基于模式使用简短形式：

```css
font: font_family font_size/line_height font_weight
```

通过这种简短的形式，我们在 SASS 中的五行（在 CSS 中的四行）被改为了一行：

```css
.element
  font: Arial 12px/1.5 bold
```

编译后，代码如下：

```css
.element {
  font: Arial 12px/1.5 bold; 
}
```

### 背景的简短形式

背景是最常用的 CSS 特性之一。背景的主要用途是：

```css
.element
  background:
    color: #000
    image: url(path_to_image.extension)
    repeat: no-repeat
    attachment: fixed
    position: top left
```

这段代码将给我们以下输出：

```css
.element {
  background-image: url(path_to_image.extension);
  background-repeat: no-repeat;
  background-attachment: fixed;
  background-position: top left; 
}
```

这是很多代码！简短形式按照这个顺序描述：

```css
background-color
background-image
background-repeat
background-attachment
background-position
```

例子：

```css
background: url color repeating attachment position-x position-y;
```

如果我们想用这种简短形式描述我们的元素，我们只需要这样做：

```css
.element
  background: #000 url(path_to_image.extension) no-repeat fixed top left
```

在 SASS 编译成 CSS 后，我们将得到以下结果：

```css
.element {
  background: #000 url(path_to_image.extension) no-repeat fixed top left; 
}
```

## 检查重复

在创建 CSS 代码时，您需要注意代码的重复。对于专业开发人员来说，代码可能看起来有点奇怪，但我们可以把它当作代码审查过程的一个很好的样本。让我们来分析一下。

HTML：

```css
<section>
    <a class="button">click it</a>
    <a class="buttonBlue">click it</a>
    <a class="buttonGreen">click it</a>
</section>
```

CSS：

```css
section .button {
    padding: 5px 10px; /* Repeated padding */
    font-size: 12px; /* Repeated font size */
    color: white; /* Repeated color */
    background: black;
}

section .buttonBlue {
    padding: 5px 10px; /* Repeated padding */
    font-size: 12px; /* Repeated font size */
    color: white; /* Repeated color */
    background: blue;
}

section .buttonGreen {
    padding: 5px 10px; /* Repeated padding */
    font-size: 12px; /* Repeated font size */
    color: white; /* Repeated color */
    background: green;
}
```

如您所见，重复部分已经被注释，现在我们将创建一个通用类：

```css
section .button {
    padding: 5px 10px;
    font-size: 12px;
    color: white;
    background: black;
}

section button.button_blue {
    background: blue;
}

section button.button_green {
    background: green;
}
```

我们需要在 HTML 代码中追加一些小的改变：

```css
<section>
    <a class="button">click it</a>
    <a class="button button_blue">click it</a>
    <a class="button button_green">click it</a>
</section>
```

在 SASS 中将其最小化：

```css
section
  .button
    padding: 5px 10px
    font-size: 12px
    color: white
    background: black

    &.button_blue
      background: blue

    &.button_green
      background: green
```

这是另一种处理重复的方法，而不改变标记：

```css
article .h1 {
    font-family: Arial; /* Repeated font family */
    padding: 10px 0 15px 0; /* Repeated padding */
    font-size: 36px;
    line-height: 1.5; /* Repeated line height */
    color: black; /* Repeated color */
}

article .h2 {
    font-family: Arial; /* Repeated font family */
    padding: 10px 0 15px 0; /* Repeated padding */
    font-size: 30px;
    line-height: 1.5; /* Repeated line height */
    color: black; /* Repeated color */
}

article .h3 {
    font-family: Tahoma; /* Oryginal font family */
    padding: 10px 0 15px 0; /* Repeated padding */
    font-size: 24px;
    line-height: 1.5; /* Repeated line height */
    color: black; /* Repeated color */
}
```

让我们收集重复的部分：

```css
font-family: Arial; 
padding: 10px 0 15px 0; 
line-height: 1.5; 
color: black; 
```

让我们添加一个将在自定义元素`.h3`中被覆盖的值：

```css
font-family: Tahoma;
```

现在，让我们描述选择器并在单独的选择器中覆盖值：

```css
article .h1,
article .h2,
article .h3 {
    padding: 10px 0 15px 0;
    line-height: 1.5;
    color: black;
    font-family: Arial;
}

article .h1 {
    font-size: 36px;
}

article .h2 {
    font-size: 30px;
}

article .h3 {
    font-size: 24px;
    font-family: Tahoma;
}
```

让我们将其改为 SASS 代码：

```css
article
.h1,
  .h2,
  .h3
    padding: 10px 0 15px 0
    line-height: 1.5
      color: black
    font:
      family: Arial

  .h1
    font:
      size: 36px

  .h2
    font:
      size: 30px

  .h3
    font:
      size: 24px
      family: Tahoma
```

让我们用`@extend`做同样的事情：

```css
article
  .h1
    padding: 10px 0 15px 0
    line-height: 1.5
    color: black
    font:
      family: Arial
      size: 36px

  .h2
    @extend .h1
    font:
      size: 30px

  .h3
    @extend .h1
    font:
      size: 24px
      family: Tahoma
```

当您自己创建代码时，检查重复的过程很容易，但当您与其他开发人员一起工作或者在一个由其他人开始的项目上工作时，这可能会更难。这个过程使代码变得更短，因此它可以被视为代码优化的过程。通过这些技术，您可以对代码进行追加更改。

# 总结

在本章中，我们讨论了 CSS 代码优化的过程。有了这些知识，您可以最小化您的代码，并且在创建代码时考虑优化过程。这些知识将使您成为一个更加明智的前端开发人员，了解代码如何可以在瞬间被最小化。

在下一章中，我们将讨论您可以在 CSS 和前端项目中使用的最终自动化！
