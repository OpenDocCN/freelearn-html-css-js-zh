# 第五章。CSS 属性 – Part 2

好的，我们已经完成了 CSS 属性的 Part 1 部分。确实，还有很多很多属性需要讨论。

现在我们继续到 Part 2。

# 字体

在设计的世界里，字体是我们拥有的最有力的资产之一，同时，它们也是被低估最多的资产之一。

排版功能如此强大，当我们正确使用它时，我们甚至可能在我们的项目中不用使用任何一张图片。

让我们来看看 CSS 字体属性，怎么样？

## font-family

`font-family` CSS 属性定义了元素想要使用的字体，其格式如下：

```js
font-family: "Times New Roman", Times, Baskerville, Georgia, serif;
```

### 描述

这个属性可以在其声明中包含一个或多个字体名称。它没有限制可以包含多少个字体名称；然而，实际上列出超过四到五个字体是非常不可能的，而且也是不必要的。

字体家族名称由逗号分隔。我们称之为*字体堆栈*。浏览器将读取字体堆栈，并使用堆栈中的第一个字体，如果找不到或下载不了，它将移动到下一个字体家族名称，依此类推，直到它能够使用堆栈中的一个字体。

字体家族名称有两种类型：字体名称和通用字体名称。

#### 字体名称

姓氏实际上是代表真实字体的名称，例如`Times`、`Arial`、`Verdana`、`Monaco`等等。它们应该始终在字体堆栈中列在通用字体名称之前。

#### 通用字体名称

这些是唯一代表系统字体的关键词。它们被称为**后备**字体。它们应该始终在字体堆栈中列在字体名称之后。通用字体名称可以是`monospace`、`san-serif`、`serif`、`cursive`和`fantasy`。

### 小贴士

当字体家族名称包含多个单词时，使用引号（单引号或双引号）不是强制的。例如，写作`font-family: "Times New Roman", serif;`与写作`font-family: Times New Roman, serif;`是相同的。注意在第二个例子中`Times New Roman`没有使用引号。

**CSS**:

```js
.element {
  font-family: "Times New Roman", Times, Baskerville, Georgia, serif;
}
```

## font-feature-settings

`font-feature-settings` CSS 属性提供了对 OpenType 字体中排版属性的控制的权限，其格式如下：

```js
font-feature-settings: "smcp";
```

### 描述

`font-feature-settings` CSS 属性允许我们控制和使用某些字体文件中包含的其他交替符号。

一个交替符号的例子是当我们输入分数 1/2 或 1/4 时，字体实际上包含了“小上标”版本，如½和¼。或者如果我们输入 H[2]O，它就会变成 H[2]O。

记住，并非所有字体都包含特殊符号（字体特征）。

要找出字体文件具有哪些字体特征，你可以使用以下两种工具中的任何一种：

1.  Fontdeck.com ([`fontdeck.com/`](http://fontdeck.com/)) - 找到你想要的字体，在“关于这个字体”部分，查找 OPENTYPE 行，那里将列出该特定字体的所有字体特征。

1.  测试 OpenType 功能（文本有链接 [`www.impallari.com/testing/index.php`](http://www.impallari.com/testing/index.php)）- 只需拖放您的字体文件并点击左上角的 OpenType Features 链接，一个大型面板将滑动出来，您可以选择查看哪些功能。

这里有一些最常见的特征标签：

+   `dlig`: 可选连字符

+   `kern`: 字距调整

+   `liga`: 标准连字符

+   `lnum`: 基线数字

+   `onum`: 旧式数字

+   `tnum`: 表格数字

+   `smcp`: 小写字母大写

+   `ss01, ss02, ss03, ss04… ss20`: 风格集（字体相关）

+   `swsh`: 波浪形

更多信息，请查看 MDN 网站：[`tiny.cc/mdn-font-feature-settings`](http://tiny.cc/mdn-font-feature-settings)

要查看所有字体特征的完整列表，请访问微软网站 [`tiny.cc/msn-font-features-list`](http://tiny.cc/msn-font-features-list)

**CSS**:

```js
/*Small capitals*/
.element {
  font-feature-settings: "smcp";
}
```

## font-size

`font-size` CSS 属性定义了元素的文本大小，其看起来如下：

```js
font-size-settings: "smcp";
```

### 描述

此属性也可以用来改变其他元素的大小，因为我们还可以计算`em`、`rem`和`ex`长度单位的价值。

我们可以使用几种不同的值类型与`font-size` CSS 属性一起使用：绝对关键词/大小、相对大小、长度和百分比。

#### 绝对关键词/大小

定义的大小直接关联到特定的字体大小。它们也可以用来根据父元素的字体大小设置元素的字体大小。值如下：

+   `xx-small`

+   `x-small`

+   `small`

+   `medium`（默认值）

+   `large`

+   `x-large`

+   `xx-large`

#### 相对大小关键词

这些大小根据其父容器字体大小增加或减少元素的字体大小。值如下：

+   `smaller`

+   `larger`

#### 长度

负值无效。当使用`px`时，字体大小是绝对的；它不相对于父容器字体大小。当使用`em`、`ex`和`ch`时，字体大小相对于元素的父容器字体大小。当使用`rem`时，字体大小相对于根元素，即`<html>`元素。当使用`vw`、`vh`、`vmax`和`vmin`时，字体大小相对于视口。

要查看此属性的可用值，请参阅*绝对长度单位*部分。

最流行的单位有：

+   `px`

+   `em`

+   `rem`

#### 百分比

`percentage`属性指的是父元素字体大小的百分比。其单位是`%`。

**CSS**:

```js
/*Absolute keywords/size*/
.element { font-size: x-small; }
/*Relative size*/
.element { font-size: smaller; }
/*Length value*/
.element { font-size: 18px; }
/*Percentage value*/
.element { font-size: 62.5%; }
```

## font-size-adjust

`font-size-adjust` CSS 属性帮助我们根据小写字母*x*和上标字母*X*的大小差异来定义字体宽高比，其看起来如下：

```js
font-size-adjust: .5";
```

### 描述

在字体堆栈中，字体大小可以不同，因此文本的样式可能会与预期设计有相当大的差异。使用`font-size-adjust` CSS 属性，我们可以确保当浏览器使用字体堆栈中的任何字体时，字体大小是一致的。

此属性接受一个不带单位的*数字*值。它也可以接受十进制值。

一个可以为我们完成这项工作的优秀在线工具是 Fontdeck 的字体大小调整 web 应用：[`fontdeck.com/support/fontsizeadjust`](http://fontdeck.com/support/fontsizeadjust)。

### 提示

尽管在撰写本节时 Firefox 是唯一支持`font-size-adjust`属性的浏览器，但我决定仍然包括它，因为一旦其他浏览器开始支持它，它将非常有价值。

**CSS**:

```js
.element {
  font-size-adjust: .5;
}
```

## font-stretch

`font-stretch` CSS 属性允许我们从相关的字体族中选择`condensed`、`normal`或`expanded`面，其外观如下：

```js
font-stretch: condensed;
```

### 描述

`font-stretch`属性不仅将字体拉伸到我们告诉它的任何地方。它会在字体文件中寻找与声明的样式相匹配的实际面；或者尽可能接近。

此属性支持以下值：

+   `ultra-condensed`

+   `extra-condensed`

+   `condensed`

+   `semi-condensed`

+   `normal`

+   `semi-expanded`

+   `expanded`

+   `extra-expanded`

+   `ultra-expanded`

**CSS**:

```js
.element {
  font-stretch: condensed;
}
```

## font-style

`font-style` CSS 属性指定了元素的字体样式，其外观如下：

```js
font-style: italic;
```

### 描述

`font-style`属性接受以下值：`normal`、`italic`和`oblique`。

让我们澄清`italic`和`oblique`之间的区别。

根据规范：

> *"斜体形式通常是草书的，而斜体面通常是常规面的倾斜版本。"*

这意味着当我们声明字体样式为`italic`时，浏览器将寻找字体的斜体面并使用该面。一个很好的例子是字体族*Georgia*；当我们使用`italic`属性时，我们可以清楚地看到它是一个真正的斜体面，而不是使正常面倾斜。

斜体使正常面倾斜或倾斜，以*模拟*斜体。

**CSS**:

```js
.element {
  font-style: italic;
}
```

## font-variant

`font-variant` CSS 属性将目标文本转换为小写字母，其外观如下：

```js
font-variant: small-caps;
```

### 描述

`font-variant`属性在 CSS3 中被视为缩写，并已通过新值进行了扩展，但开发者很少使用。

需要注意的一点是，如果文本已经是全大写，并且我们对其应用了`small-caps`属性，文本将不会改变；它将继续保持全大写。

最常用的值是：`normal`和`small-caps`。CSS3 中的一些附加值包括`small-caps`、`all-small-caps`、`petite-caps`、`all-petite-caps`、`unicase`和`titling-caps`。

想要了解更多信息，请访问 MDN 网站：[`tiny.cc/mdn-font-variant`](http://tiny.cc/mdn-font-variant)

**CSS**:

```js
.element {
  font-variant: small-caps;
}
```

## font-variant-ligatures

`font-variant-ligatures` CSS 属性在文本中定义连字符，指定了如何组合符号以产生更和谐的文本，其外观如下：

```js
font-variantligatures: discretionary-ligatures;
```

### 描述

`font-variant-ligatures`在 OpenType 字体中很常见。

`font-variant-ligatures`属性使用以下值：`common-ligatures`、`no-common-ligatures`、`discretionary-ligatures`、`no-discretionary-ligatures`、`historical-ligatures`、`no-historical-ligatures`、`contextual`、`no-contextual`和`contextual`。

更多信息，请查看 MDN 网站：[`tiny.cc/mdnfont-variant-ligatures`](http://tiny.cc/mdnfont-variant-ligatures)

**CSS**:

```js
.element {
  font-variant-ligatures: discretionary-ligatures;
}
```

## font-weight

`font-weight` CSS 属性定义了字体的厚度（重量），其语法如下：

```js
font-weight: bold;
```

### 描述

这个属性接受两种类型的值：一个*数字*值和一个*关键字*值。

#### 数字值

这个属性接受`100`、`200`、`300`、`400`、`500`、`600`、`700`、`800`和`900`这样的数字值。

#### 关键字值

这个属性也接受关键字值，如`normal`、`bold`、`bolder`和`lighter`。

### 小贴士

`normal`关键字等同于`400`数字值，而`bold`关键字等同于`700`。

有一点需要注意，关键字`bolder`和`lighter`取决于父元素的字体粗细。

**CSS**:

```js
/*Numeric value*/
.element { font-weight: 200; }
/*Keyword value*/
.element { font-weight: bold; }
```

## font

`font` CSS 属性是`font-style`、`font-variant`、`font-weight`、`font-size`、`line-height`和`font-family`属性的简写，其语法如下：

```js
font: italic small-caps bold 18px/2 "Times New Roman", Times, Baskerville, Georgia, serif;
```

### 描述

在使用`font`简写属性时，有一些事情需要考虑，以确保其正常工作：

+   至少需要声明`font-size`和`font-family`属性

+   如果省略了上述两个属性中的任何一个，则声明将被忽略

+   如果要声明这两个属性之外的更多属性，则必须遵守以下顺序：

    +   `font-style`

    +   `font-variant`

    +   `font-weight`

    +   `font-size`

    +   `line-height`

    +   `font-family`

    ### 小贴士

    在简写中声明`line-height`值时，它必须始终跟在`font-size`属性之后，并用斜杠"`/`"字符分隔，例如，`.element { font: 12px/1.6 Arial; }`。

+   **CSS**:

    ```js
    /*All font properties in a single declaration*/
    .element { font: italic small-caps bold 18px/2 "Times New Roman", Times, Baskerville, Georgia, serif; }
    /*font-style*/
    .element { font-style: italic; }
    /*font-variant*/
    .element { font-variant: small-caps; }
    /*font-weight*/
    .element { font-weight: bold; }
    /*font-size*/
    .element { font-size: 18px; }
    /*line-height*/
    .element { line-height: 2; }
    /*font-family*/
    .element { font-family: "Times New Roman", Times, Baskerville, Georgia, serif; }
    ```

# Transform

CSS 转换已经变得如此流行，以至于现在很少看到网站上没有某种转换——从按钮形状和动画到布局。

让我们深入探讨。

## transform

`transform` CSS 属性允许我们在 2D 和 3D 空间中对元素进行缩放、旋转、倾斜和转换，其语法如下：

```js
transform: translate(-10px, 10%);
```

### 描述

这个属性支持以下值：`scale()`、`skewX()`和`skewY()`、`translate()`、`rotate()`、`matrix()`和`perspective()`。

注意，X 轴等于*水平*，Y 轴等于*垂直*。

### 小贴士

记住哪个轴是哪个的一个简单方法是说：“*x 是一个十字，所以 x 轴是横跨的*”。[`tiny.cc/xy-axis`](http://tiny.cc/xy-axis)

#### scale()

`scale()`函数用于缩放元素。它也是`scaleX()`和`scaleY()`函数的简写。它接受一个不带单位的*数字*值。*数字*值表示元素缩放的比例。例如，`2`表示元素将被缩放至其大小的两倍。负值也是有效的。

#### skew()

`skew()`函数使元素倾斜。如果声明了一个单个值，那么它只会使元素在*x*轴上倾斜。如果声明了两个值，那么元素将在*x*轴和*y*轴上倾斜。`skew()`函数接受一个*数值*后跟`deg`、`grad`、`rad`或`turn`单位。然而，`deg`单位是最常用的单位。负值是有效的。

##### skewX()和 skewY()

`skewX()`和`skewY()`函数仅使元素水平或垂直倾斜。

#### translate()

`translate()`函数改变元素的位置。如果声明了一个单个值，那么它只会沿 X 轴平移元素。如果声明了两个值，那么元素将在 X 轴和 Y 轴上平移。负值是有效的。

##### translateX()和 translateY()

`translateX()`和`translateY()`函数改变元素的水平或垂直位置。

#### rotate()

`rotate()`函数在 2D 空间中围绕一个固定点旋转一个元素。它接受一个*数值*后跟`deg`、`grad`、`rad`或`turn`单位。尽管如此，`deg`单位是最常用的。负值是有效的。

#### matrix()

`matrix()`函数是所有变换值的简写，因为它们可以在这里组合。鉴于`matrix()`函数的复杂性，这需要扎实的数学基础。

#### perspective()

此值给元素一个 3D 视角；一旦设置了视角，我们就可以使用其他任何值。相关元素将在 3D 平面上反应。它接受一个带有*长度单位*的*数值*。

本书不涉及`matrix()`函数的高级数学解释。然而，对于非常详细的解释，你可以参考这两篇文章中的任意一篇：

+   *《理解 CSS 变换矩阵》*，作者：Tiffany Brown，链接：[`tiny.cc/css-matrix-1`](http://tiny.cc/css-matrix-1)

+   *《CSS3 矩阵变换，让数学挑战者也能理解》*，作者：Zoltan Hawryluk，链接：[`tiny.cc/css-matrix-2`](http://tiny.cc/css-matrix-2)

**CSS**:

```js
/*Scale the same value in both X an Y-axis*/
.element { transform: scale(1.1); }
/*Scale different values for X and Y-axis*/
.element { transform: scale(1.1, 1.5); }
/*Tilt the element on X-axis only*/
.element { transform: skew(10deg); }
/*Tilt the element on both X and Y-axis*/
.element { transform: skew(10deg, -20deg); }
/*Move the element 10px to the right*/
.element { transform: translate(10px); }
/*Move the element 10px to the right and 10% down*/
.element { transform: translate(-10px, 10%); }
/*Rotate in a 2D plane*/
.element { transform: rotate(10deg); }
/*Matrix*/
.element { transform: matrix(2, 2, 1, 2, 0, 0); }
/*Add perspective to the element to make it rotate in a 3D plane*/
.element { transform: perspective(100px) rotateX(40deg); }
```

## transform-origin

`transform-origin` CSS 属性允许我们改变变换元素的基点，它看起来像这样：

```js
transform-origin: 50px;
```

### 描述

只有当声明了`transform`属性时，`transform-origin`属性才起作用。

2D 变换可以影响*x*和*y*轴。3D 变换可以改变这两个轴，以及*z*轴。

对于 2D 变换，最多可以声明两个值；第一个是 X 轴（水平），第二个是 Y 轴（垂直）。

3D 变换可以接受最多三个值，代表 X、Y 和 Z 轴。

该属性接受的键值有：`top`、`right`、`bottom`、`left`和`center`。

**CSS**:

```js
/*Single value affects both X and Y-axis*/
.element {
  transform: scale(2, 4);
  transform-origin: 50px;
}
/*Two values one for each axis*/
.element {
  transform: scale(2, 4);
  transform-origin: 50px -100px;
}
/*Keyword value*/
.element {
  transform: scale(2, 4);
  transform-origin: bottom;
}
/*Length and keyword values*/
.element {
  transform: scale(2, 4);
  transform-origin: 50px bottom;
}
/*Both keyword values*/
.element {
  transform: scale(2, 4);
  transform-origin: right bottom;
}
```

## transform-style

`transform-style` CSS 属性定义了元素是在 3D 空间中定位还是在 2D 空间（平坦）中，它看起来像这样：

```js
transform-style: preserve-3d;
```

### 描述

该属性只接受两个值：`flat`和`preserve-3d`。

当应用 `preserve-3d` 属性时，元素在 *z* 轴上的堆叠可以通过 `translate()` 函数进行更改，因此元素可以出现在不同的平面上，而不管它们在源 HTML 中出现的顺序如何。

当应用 `flat` 属性时，元素遵循它们在源 HTML 中出现的顺序。

### 小贴士

注意，此属性应用于父元素，而不是子元素。

**CSS**:

```js
/*Perspective*/
.parent-container {
  transform-style: preserve-3d;
  perspective: 500px;
}
.element-1 {
  transform: translateZ(1px) rotateX(60deg);
}
.element-2 {
  transform: translateZ(-2px);
}
```

## 过渡

CSS 过渡使我们能够对我们的动画有非常细粒度的控制。

让我们来看看这些属性。

### transition

`transition` CSS 属性是所有过渡属性的缩写：`transition-delay`、`transition-duration`、`transition-property` 和 `transition-timing-function`。它看起来像这样：

```js
transition: width 400ms ease-out 1s;
```

#### 描述

此属性允许我们通过 `:hover` 或 `:active` 伪类定义元素两种状态之间的过渡。

需要考虑的一点是，这些属性的顺序并不重要。然而，由于 `transition-delay` 和 `transition-duration` 使用相同的值单位，`transition-delay` 将始终首先考虑，然后是 `transition-duration`。

**CSS**:

```js
/*Shorthand with all properties in a single declaration*/
.element {
  width: 100px;
  /*property - duration - timing function - delay*/
  transition: width 400ms ease-out 1s;
}
/*Longhand. Each property is declared separately*/
.element {
  transition-property: width;
  transition-duration: 400ms;
  transition-timing-function: ease-out;
  transition-delay: 1s;
}
.element:hover {
  width: 300px;
}
```

### transition-delay

`transition-delay` CSS 属性允许我们设置一个 *计时器*。当计时器达到零时，过渡开始。它看起来像这样：

```js
transition-delay: 1s;
```

#### 描述

此属性接受一个 *数字* 值，后跟 `s` 或 `ms`，分别代表 *秒* 和 *毫秒*。

**CSS**:

```js
.element {
  transition-delay: 1s;
}
```

### transition-duration

`transition-duration` CSS 属性允许我们定义过渡从开始到结束需要多长时间。这也被称为 **周期**，它看起来像这样：

```js
transition-duration: 400ms;
```

#### 描述

`transition-duration` 属性接受一个 *数字* 值，后跟 `s` 或 `ms`，分别代表秒和毫秒。

**CSS**:

```js
.element {
  transition-duration: 400ms;
}
```

### transition-property

`transition-property` CSS 属性指定了哪些 CSS 属性或属性将进行过渡。

虽然并非所有属性都可以动画化。W3C 有一个 animatable CSS 属性的不错列表，可以在 [`tiny.cc/w3c-animatable-css-props`](http://tiny.cc/w3c-animatable-css-props) 找到

#### 描述

`transition-property` CSS 属性接受以下值：

+   `none`: 这意味着不会发生任何过渡

+   `all`: 这意味着所有属性都将进行过渡

+   `属性名`: 这意味着指定的属性或属性将进行过渡

**CSS**:

```js
.element {
  transition-property: width;
}
```

### transition-timing-function

`transition-timing-function` CSS 属性定义了动画的速度如何在其周期内进展。

`transition-timing-function` 和 `animation-timing-function` 属性接受相同的五个预定义值，也称为 **贝塞尔** 曲线的缓动函数：`ease`、`ease-in`、`ease-out`、`ease-in-out` 和 `linear`。

请参考 *animation-timing-function* 部分，以获取所有值的详细解释。

**CSS**:

```js
.element {
  transition-timing-function: ease-out;
}
```

# 定位

在构建网站和应用时，定位元素是我们花费大量时间的事情，因此了解如何将元素放置在布局中至关重要，尤其是在元素可以根据可用空间具有不同位置时。

让我们看看定位究竟是怎么回事。

## position

`position` CSS 属性定义了元素的位置。

### 描述

`position` 属性有五个关键字值：`static`、`absolute`、`relative`、`fixed` 和 `sticky`。

#### static

这是 `position` 属性的默认值。元素保持在文档的流中，并出现在其标记中实际位置。

#### absolute

元素从文档流中移除，并相对于其最近的定位祖先元素进行定位。

#### relative

除非声明了 `top`、`right`、`bottom` 或 `left` 中的一个或多个属性，否则元素不会改变位置。它还为 `absolute` 定位子元素创建一个 *参考位置*。

#### fixed

元素就像 `absolute` 定位元素一样从文档流中移除。然而，与相对于祖先元素的 `absolute` 定位元素不同，`fixed` 元素始终相对于文档。

#### sticky

此值是 `relative` 和 `fixed` 定位的混合。元素被视为 `relative`，直到达到一个特定的点或阈值，此时元素被处理为 `fixed`。在撰写本文时，只有 Firefox 支持此属性。

**CSS**:

```js
/*Position relative*/
.element {
  position: relative;
}
/*When the element hits the top of the viewport, it will stay fixed at 20px from the top*/
.element {
  position: sticky;
  top: 20px;
}
```

## top

CSS 的 `top` 属性与 `position` 属性紧密相关。

如果元素声明了 `position: relative;`，则此属性指定元素从其当前位置的顶部距离；如果祖先具有 `position: relative;` 且元素声明了 `position: absolute;`，则指定元素从最近的祖先顶部距离。

如果没有祖先声明了 `position: relative;`，则绝对定位的元素将遍历 DOM，直到它到达 `<body>` 标签，此时它将将自己定位在页面的顶部，无论其在源 HTML 中的位置如何。

负值是有效的。

### 描述

它支持以下值：

+   `auto`：`auto` 关键字是默认值。浏览器计算顶部位置。

+   `长度值`：对于长度值，我们使用以下单位之一：`px`、`em`、`in`、`mm`、`cm`、`vw` 等。

+   `百分比值`：对于百分比值，我们使用像 `50%`、`85%` 这样的百分比。

**CSS**:

```js
/*auto*/
.element { top: auto; }
/*Length value*/
.element { top: 20px; }
/*Percentage value*/
.element { top: 15%; }
```

## bottom

CSS 的 `bottom` 属性与 `position` 属性紧密相关。

如果元素本身声明了 `position: relative;`，则此属性指定元素从其当前位置的底部距离；如果祖先具有 `position: relative;` 且元素声明了 `position: absolute;`，则指定元素从最近的祖先底部距离。

如果没有祖先元素声明了 `position: relative;`，则绝对定位的元素将遍历 DOM，直到它到达 `<body>` 标签，此时它将将自己定位在页面的底部，而不管它在源 HTML 中的位置如何。

负值是有效的。

### 描述

它支持以下值：

+   `auto`: `auto` 关键字是 `bottom` 属性的默认值。浏览器计算顶部位置。

+   `长度值`: 对于长度值，我们使用以下单位之一：`px`、`em`、`in`、`mm`、`cm`、`vw`等等。

+   `百分比值`: 对于百分比值，我们使用类似 `50%`、`85%` 等的百分比。

**CSS**:

```js
/*auto*/
.element { bottom: auto; }
/*Length value*/
.element { bottom: 20px; }
/*Percentage value*/
.element { bottom: 15%; }
```

## left

`left` CSS 属性与 `position` 属性紧密相关。

该属性指定了元素从其当前位置左侧的距离，如果元素声明了 `position: relative;` 或者在祖先元素有 `position: relative;` 且元素有 `position: absolute;` 声明时，从最近的祖先元素的右侧开始计算。

如果没有祖先元素声明了 `position: relative;`，则绝对定位的元素将遍历 DOM，直到它到达 `<body>` 标签，此时它将将自己定位在页面的左侧，而不管它在源 HTML 中的位置如何。

负值是有效的。

### 描述

`left` 属性支持以下值：

+   `auto`: `auto` 关键字是 `left` 属性的默认值。浏览器计算顶部位置。

+   `长度值`: 对于长度值，我们使用以下单位之一：`px`、`em`、`in`、`mm`、`cm`、`vw`等等。

+   `百分比值`: 对于百分比值，我们使用类似 `50%`、`85%` 等的百分比。

**CSS**:

```js
/*auto*/
.element { left: auto; }
/*Length value*/
.element { left: 20px; }
/*Percentage value*/
.element { left: 15%; }
```

## right

`right` CSS 属性与 `position` 属性紧密相关。

该属性指定了元素从其当前位置右侧的距离，如果元素声明了 `position: relative;` 或者在祖先元素有 `position: relative;` 且元素有 `position: absolute;` 声明时，从最近的祖先元素的右侧开始计算。

如果没有祖先元素声明了 `position: relative;`，则绝对定位的元素将遍历 DOM，直到它到达 `<body>` 标签，此时它将将自己定位在页面的右侧，而不管它在源 HTML 中的位置如何。

负值是有效的。

### 描述

它支持以下值：

+   `auto`: `auto` 关键字是 `right` 属性的默认值。浏览器计算顶部位置。

+   `长度值`: 对于长度值，我们使用以下单位之一：`px`、`em`、`in`、`mm`、`cm`、`vw`等等。

+   `百分比值`: 对于百分比值，我们使用类似 `50%`、`85%` 等的百分比。

**CSS**:

```js
/*auto*/
.element { right: auto; }
/*Length value*/
.element { right: 20px; }
/*Percentage value*/
.element { right: 15%; }
```

## vertical-align

`vertical-align` CSS 属性控制元素在垂直方向上的定位，以便将其与旁边的另一个（或多个）元素对齐。

### 描述

它接受以下值：

+   `baseline`：这是默认值。它将元素与底部对齐，正好在文本的最后一行，无论元素的行高如何。换句话说，它将文本的最后一行与元素的基线对齐。

+   `bottom`：此属性将元素的容器与底部对齐。不考虑元素的文本和行高，只考虑元素的容器与底部对齐。

+   `长度值`：对于长度值，我们使用以下单位之一：`px`、`em`、`in`、`mm`、`cm`、`vw` 等。负值是有效的。

+   `middle`：此属性根据元素的垂直中点将元素水平居中对齐。

+   `百分比值`：对于百分比值，我们使用类似 `50%`、`85%` 等的百分比。负值是有效的。

+   `sub`：此属性将元素与父容器的下标基线对齐。

+   `super`：此属性将元素与父容器的上标基线对齐。

+   `top`：此属性将元素的容器与顶部对齐。不考虑元素的文本和行高。

+   `text-bottom`：此属性根据父元素的文本基线将元素对齐到底部。不考虑元素的行高，只考虑其容器的底部。

+   `text-top`：此属性根据父元素的文本基线将元素对齐到顶部。不考虑元素的行高，但考虑其容器的顶部。

# 文本

字体排印是一个极其强大的设计功能，实际上，使用 CSS 样式化文本实际上非常简单。

让我们看看怎么做。

## 颜色

`color` CSS 属性定义文本的颜色，其外观如下：

```js
color: red;
```

或者，它也可以是这样的：

```js
color: #f00;
```

### 描述

`color` 属性支持所有颜色模式，包括 `HEX`、`RGB`、`RGBa`、`HSL`、`HSLs` 和 *颜色名称*。

**CSS**：

```js
/*Color Name*/
.element { color: red; }
/*HEX*/
.element { color: #f00; }
/*RGB*/
.element { color: rgb(255, 0, 0); }
/*RGBa - Color has 50% opacity*/
.element { color: rgba(255, 0, 0, .5); }
/*HSL*/
.element { color: hsl(0, 100%, 50%); }
/*HSLa - Color has 50% opacity*/
.element { color: hsla(0, 100%, 50%, .5); }
```

## text-align

`text-align` CSS 属性定义了文本的对齐方式，其外观如下：

```js
text-align: center;
```

### 描述

文本可以居中、左对齐、右对齐和完全对齐。

`text-align` 属性仅适用于内联元素。如果将此属性应用于块级元素，则它不会应用于该元素本身，但会应用于其内容。

**CSS**：

```js
/*Centered text*/
.element { text-align: center; }
/*Left aligned text*/
.element { text-align: left; }
/*Right aligned text*/
.element { text-align: right; }
/*Fully justified text*/
.element { text-align: justify; }
```

## text-decoration

`text-decoration` CSS 属性定义了文本的几个格式化功能，其外观如下：

```js
text-decoration: underline;
```

### 描述

`text-decoration` 属性接受以下关键字值：`underline`、`overline`、`line-through`、`none` 和 `blink`。

此属性也是 `text-decoration-line`、`text-decoration-color` 和 `text-decoration-style` 属性的缩写。

如果用作缩写，则可以在同一声明中接受一个、两个或三个值。

**CSS**：

```js
/*Line above the text*/
.element { text-decoration: overline; }
/*Line under the text*/
.element { text-decoration: underline; }
/*Line across the text*/
.element { text-decoration: line-through; }
/*No underline*/
.element { text-decoration: none; }
/*Blinking text*/
.element { text-decoration: blink; }
/*Shorthand*/

/*Two values*/
.element { text-decoration: underline wavy; }
/*Three values*/
.element { text-decoration: overline dashed yellow; }
```

## text-decoration-line

`text-decoration-line` CSS 属性定义文本可以具有的装饰线类型，其外观如下：

```js
text-decoration-line: overline;
```

### 描述

`text-decoration-line` 属性可以在单个声明中接受一个、两个或三个值。关键字值与 `text-decoration` 属性中的相同：`underline`、`overline`、`line-through`、`none` 和 `blink`。

**CSS**:

```js
/*One value*/
.element { text-decoration-line: overline; }
/*Two values*/
.element { text-decoration-line: overline underline; }
/*Three values*/
.element { text-decoration-line: overline underline blink; }
```

## text-decoration-color

`text-decoration-color` CSS 属性定义了 `text-decoration` 属性可以具有的颜色类型，其外观如下：

```js
text-decoration-color: red;
```

### 描述

它支持所有颜色模式：`HEX`、`RGB`、`RGBa`、`HSL`、`HSLs` 和 *颜色名称*。

**CSS**:

```js
/*Color Name*/
.element { text-decoration-color: red; }
/*HEX*/
.element { text-decoration-color: #f00; }
/*RGB*/
.element { text-decoration-color: rgb(255, 0, 0); }
/*RGBa - Color has 50% opacity*/ 
.element { text-decoration-color: rgba(255, 0, 0, .5); }
/*HSL*/
.element { text-decoration-color: hsl(0, 100%, 50%); }
/*HSLa - Color has 50% opacity*/
.element { text-decoration-color: hsla(0, 100%, 50%, .5); }
```

## text-decoration-style

`text-decoration-style` CSS 属性定义了 `text-decoration` 属性可以具有的线类型，其外观如下：

```js
text-decoration-style: dotted;
```

### 描述

`text-decoration-style` 属性支持以下关键字值：`dashed`、`dotted`、`double`、`solid` 和 `wavy`。

**CSS**:

```js
.element {
  text-decoration-style: wavy;
}
```

## text-indent

`text-indent` CSS 属性定义了元素中第一行文本的左侧空间（缩进），其外观如下：

```js
text-indent: red;
```

### 描述

它接受 `length` 和 `percentage` 值。负值是有效的。

**CSS**:

```js
.element {
  text-indent: 50px;
}
```

## text-overflow

`text-overflow` CSS 属性定义了超出容器外部的文本应该如何被裁剪，其外观如下：

```js
text-overflow: ellipsis;
```

### 描述

为了使此属性生效，应该声明另外两个属性：`overflow: hidden;` 和 `white-space: nowrap;`。

有两个关键字值：`clip` 和 `ellipsis`。

#### clip

这会在父容器的边缘处精确切断文本。这可能会导致最后一个字符在任何时刻被切断，只显示其一部分。

#### ellipsis

这在文本行的末尾添加一个省略号字符 "…"。

CSS:

```js
.element {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
```

## text-rendering

`text-rendering` CSS 属性允许我们定义文本的质量相对于速度/性能，其外观如下：

```js
text-rendering: optimizeLegibility;
```

### 描述

根据字体，当应用此属性时，我们可以看到诸如更好的字距调整和更好的连字符等好处。

然而，由于这个 CSS 属性实际上是一个 SVG 属性，并且不属于任何 CSS 标准，因此浏览器和操作系统会根据自己的判断来应用这个属性，这可能会导致在不同的浏览器和/或平台之间无法达到预期的改进。

此外，一些小屏幕设备在遇到 `text-rendering` CSS 属性时存在严重的性能问题，尤其是较旧的 iOS 和 Android 设备。

### 小贴士

使用 `text-rendering` CSS 属性时要格外小心，并确保运行所有相关测试。

此属性支持四个值：`auto`、`optimizeSpeed`、`optimizeLegibility` 和 `geometricPrecision`。

#### auto

这是默认值。浏览器会尽力做出最佳猜测，以优化文本的渲染，以实现速度、可读性和几何精度。

记住，每个浏览器对此属性的解析都不同。

#### optimizeSpeed

浏览器更倾向于渲染速度而不是可读性和几何细节。它禁用了字距调整和连字符。

#### optimizeLegibility

浏览器更倾向于可读性而非渲染速度和几何细节。它启用了字距调整和一些可选的连字符。请注意，如果浏览器尝试使用任何字距调整和连字符，这些信息需要包含在字体文件中，否则我们不会看到这些功能的效果。

#### geometricPrecision

浏览器更倾向于几何细节而非可读性和渲染速度。此属性在缩放字体时很有帮助。

例如，某些字体中的字距调整可能无法正确缩放，因此当我们应用此值时，浏览器能够保持文本看起来很漂亮。

**CSS**:

```js
.element {
  text-rendering: geometricPrecision;
}
```

## text-shadow

`text-shadow` CSS 属性将阴影应用到文本上，它看起来像这样：

```js
text-shadow: 5px -2vw .2cm black;
```

### 描述

此属性可以在同一声明中接受两个、三个或四个值。声明中的第一个长度值始终用于`x-offset`值，第二个长度值用于`y-offset`值。

它支持以下四个值：`x-offset`、`y-offset`、`color`和`blur`。

+   `x-offset`: 这设置了阴影与文本的水平距离。它声明为一个长度值（`px`、`em`、`in`、`mm`、`cm`、`vw`等）。负值是有效的。此值是必需的。

+   `y-offset`: 这设置了阴影与文本的垂直距离。它声明为一个长度值（`px`、`em`、`in`、`mm`、`cm`、`vw`等）。负值是有效的。此值是必需的。

+   `color`: 这是文本阴影的颜色。它支持所有颜色模式：`HEX`、`RGB`、`RGBa`、`HSL`、`HSLs`和*颜色名称*。此值是可选的。如果没有指定，默认颜色将与文本本身的颜色相同。

+   `blur`: 这是**模糊**效果。它声明为一个长度值（`px`、`em`、`in`、`mm`、`cm`、`vw`等）。它支持所有颜色模式：`HEX`、`RGB`、`RGBa`、`HSL`、`HSLs`和*颜色名称*。此值是可选的。如果没有指定，默认值是零（`0`）。

**CSS**:

```js
/*2 values: x-offset and y-offset*/
.element { text-shadow: 5px 10px; }
/*3 values: x-offset, y-offset and blur. Color value defaults to the font color*/
.element { text-shadow: 5px 2vw 5px; }
/*4 values: x-offset, y-offset, blur and color name*/
.element { text-shadow: 5px -2vw .2cm black; }
```

## text-transform

`text-transform` CSS 属性控制文本的大小写，它看起来像这样：

```js
text-transform: capitalize;
```

### 描述

`text-transform`属性支持以下四个关键字值：`none`、`capitalize`、`uppercase`和`lowercase`。

+   `none`: 这是默认值。不应将任何大小写应用于元素。

+   `capitalize`: 这会将每个单词的首字母大写。此属性足够智能，可以忽略行首的任何特殊字符或符号，并将第一个单词的首字母大写。

+   `uppercase`: 这会将所有文本转换为大写/首字母大写。此属性还可以忽略行首的特殊字符或符号。

+   `lowercase`: 这会将所有文本转换为小写。此属性还可以忽略行首的特殊字符或符号。

## text-underline-position

`text-underline-position` CSS 属性定义了具有声明`text-decoration`属性的元素的下划线位置，它看起来像这样：

```js
text-underline-position: left;
```

### 描述

`text-underline-position` 属性支持四个关键字值：`auto`、`left`、`right` 和 `under`。

#### auto

这是默认值。此属性允许浏览器决定下划线的位置，是位于文本的基线还是其下方。

#### left

这仅适用于垂直书写模式。它将下划线放置在文本的左侧。

#### right

这仅适用于垂直书写模式。它将下划线放置在文本的右侧。

#### under

此值将下划线放置在文本的基线之下，因此它不会跨越任何下降线（它支持如 `q`、`p`、`y` 等值）。这对于包含大量下标的数学和化学公式的文本很有帮助，这样下划线就不会干扰某些字符，使这些公式变得混乱或难以阅读。

**CSS**:

```js
.element {
  text-underline-position: left;
}
```

## direction

`direction` CSS 属性设置文本的方向。对于西方语言和其他类似语言，是从左到右，对于阿拉伯语或希伯来语等语言，是从右到左。其外观如下：

```js
direction: rtl;
```

### 描述

文本方向通常通过 HTML 中的 `dir` 属性而不是通过 CSS 定义。

`direction` CSS 属性不会由表格单元格继承。因此，除了这个之外，如果 CSS 文件没有加载，建议通过 HTML 文档中的 `dir` 属性设置文本方向，以保持完整的级联支持。

此属性接受两个关键字值，`ltr` 和 `rtl`，分别表示 *从左到右* 和 *从右到左*。

**CSS**:

```js
.element {
  direction: rtl;
}
```

# 表格

表格，表格，表格！与图像一起，HTML 表格是网页设计的黑羊。

不论是黑羊还是白羊，表格的力量都是惊人的。而且，如果有一个 HTML 元素能很好地完成其工作，那么它就是表格。

让我们深入探讨。

## table-layout

`table-layout` CSS 属性允许我们定义 HTML 表格在文档中的布局方式，其外观如下：

```js
table-layout: fixed;
```

### 描述

有两个关键字值：`auto` 和 `fixed`。

+   `auto`：这是默认值。浏览器会以这种方式自动布局表格，而不需要在 CSS 中声明任何内容。表格单元格会适应其内部的内容；表格的行为有时可能是不可预测的。

+   `fixed`：通过从第一行声明表格单元格的宽度，整个表格的渲染实际上可以更快；我们为提高性能所做的任何努力都是对每个人来说都是一大胜利。

由于表格单元格具有固定宽度，根据单元格中的数据，一些信息可能会溢出单元格。通过组合使用 `overflow` 属性和 `text-overflow: ellipsis;`，我们可以解决这个问题。

**CSS**:

```js
table {
  table-layout: fixed;
}
```

## border-collapse

`border-collapse` CSS 属性告诉表格单元格保持分离或靠近（合并），其外观如下：

```js
border-collapse: collapse;
```

### 描述

此属性支持两个关键字值：`separate` 和 `collapse`。

+   `separate`：这是默认值。表格单元格之间有空隙，并且每个单元格都有自己的边框。

+   `collapse`：此值将单元格聚集在一起，因此空间丢失，单元格共享边框。

**CSS**:

```js
table {
  border-collapse: collapse;
}
```

## border-spacing

`border-spacing` CSS 属性在表格单元格之间创建空间，它看起来像这样：

```js
border-spacing: 10px;
```

### 描述

`border-spacing` 属性仅在存在 `border-collapse: separate;` 声明时才起作用。

**CSS**:

```js
table {
  border-collapse: separate;
  border-spacing: 10px;
}
```

## empty-cells

`empty-cells` CSS 属性允许我们定义浏览器应该如何渲染没有内容的单元格的边框和背景，它看起来像这样：

```js
empty-cells: hide;
```

### 描述

`empty-cells` 属性支持两个关键字值：`show` 和 `hide`，它们确定是否渲染边框和背景。

**CSS**:

```js
table {
  empty-cells: hide;
}
```

## caption-side

`caption-side` CSS 属性定义了表格标题的位置，它看起来像这样：

```js
caption-side: bottom;
```

### 描述

它支持两个关键字值：`top` 和 `bottom`。

在 CSS 2 中，支持了其他关键字值，如 `left` 和 `right`，但在 CSS 2.1 中被删除。

**CSS**:

```js
caption {
  caption-side: bottom;
}
```

# 单词和段落

继续介绍排版功能，我们现在进入 `word` 和 `paragraph` 属性。

让我们开始吧。

## hyphens

`hyphens` CSS 属性告诉浏览器在文本行换行时如何处理连字符，它看起来像这样：

```js
hyphens: auto;
```

### 描述

我们可以控制何时连字符化以及是否允许它或在某些条件下发生。

连字符是语言相关的。文档中的语言由 `lang` 属性定义；如果浏览器支持该语言并且有适当的连字符词典，则会进行连字符化。每个浏览器对连字符化的支持方式不同。

### 提示

确保在 `<html>` 标签上全局声明或在每个应进行连字符化的特定元素上声明 `lang` 属性。

该属性支持三个关键字值：`none`、`manual` 和 `auto`。

+   `none`：即使有字符建议在哪里引入行中断，文本行也不会在行中断处中断。行仅在空白处中断。

+   `manual`：文本行在存在字符建议行中断的地方中断。

+   `auto`：浏览器根据需要决定引入行中断。它的决定基于连字符字符的存在或语言特定的信息。

#### Unicode 字符用于建议行中断机会

我们可以使用两个 Unicode 字符来手动在文本中设置潜在的行中断：

```js
U+2010 (HYPHEN)
```

这个字符被称为**硬连字符**。它是一个实际的连字符 "-"，并且在文本中是可见的。浏览器可能会也可能不会在该特定字符处断行。

```js
U+00AD (SHY)
```

这个字符被称为**软连字符**。尽管它在标记中存在，但在文本中不会渲染。然而，浏览器会看到这个字符，如果它可以利用它来创建行中断，它将会这样做。要插入软连字符，我们使用 `&shy;`。

**CSS**:

```js
.element {
  hyphens: auto;
}
```

## word-break

`word-break` CSS 属性用于指定是否应在单词内发生换行而不是在连字符或单词之间的空格处断行，其外观如下：

```js
word-break: break-all;
```

### 描述

`word-break` 属性在长字符串（如 URL）比其父容器宽时非常有用，这会破坏布局或溢出到边缘。应用 `word-break` 属性会使 URL 在字符串的某个位置断开。

该属性支持三个关键字值：`normal`、`break-all` 和 `keep-all`。

#### normal

这是默认值。换行将根据默认的换行规则发生。

#### break-all

这允许在任何地方发生换行，包括两个字母之间。

#### keep-all

这仅影响 CJK（中文、日文和韩文）文本。在这里，文本单词不会断开。

**CSS**:

```js
.element {
  word-break: break-all;
}
```

## word-spacing

`word-spacing` CSS 属性控制单词之间的空间，其外观如下：

```js
word-spacing: .2em;
```

### 描述

它支持以下值：一个 `normal` 值、一个 `length` 值和一个 `percentage` 值。

#### normal

这是默认值。它由当前字体或浏览器本身定义，并重置为默认间距。

#### 长度值

当我们使用长度值时，我们使用以下单位之一：`px`、`em`、`in`、`mm`、`cm`、`vw` 等

#### 百分比值

对于百分比值，我们使用像 `50%`、`85%` 这样的百分比。

**CSS**:

```js
.element {
  word-spacing: .2em;
}
```

## word-wrap

`word-wrap` CSS 属性允许在长单词中没有建议的断点时在任意位置断开，其外观如下：

```js
empty-cells: hide;
```

### 描述

该属性支持两个关键字值：`normal` 和 `break-word`。

+   `normal`：它使长单词在正常断点处断开

+   `break-word`：它表示原本不可断开单词现在可以在任意位置断开

**CSS**:

```js
.element {
  word-wrap: break-word;
}
```

## line-height

`line-height` CSS 属性定义了文本行之间的距离。在排版中，行高被称为 **leading**。

### 描述

它支持以下值：一个 `normal` 值、一个 `number` 值、一个 `length` 值和一个 `percentage` 值。

+   `normal`：这是默认值。行高由浏览器定义。

+   `数字值`：这是一个没有单位的数字。这是推荐的方法。推荐无单位值的原因是这个值可以级联到子元素。子元素可以根据它们的字体大小调整行高。

+   `长度值`：当我们使用长度值时，我们使用以下单位之一：`px`、`em`、`in`、`mm`、`cm`、`vw` 等。我们也可以使用小数。

+   `百分比值`：对于百分比值，我们使用像 `50%`、`85%` 这样的百分比。

**CSS**:

```js
.element {
  line-height: 1.6;
}
```

## orphans

让我们先了解术语 **孤行**。孤行是指当文本块跨越两页时，留在上一页的段落行。在 Web 世界中，这体现在跨越多个列的文本中，其中段落的第一个行留在前一列。

从编辑的角度来看，这非常不好。推荐的孤儿处理方法是至少在上一页或列上留下三行。行数越多越好。

`orphans` CSS 属性控制上一页上剩余的行数，其格式如下：

```js
orphans: 3;
```

### 描述

这个属性在打印样式表中非常有用，但在使用 CSS 列时也可以工作。

它支持不带单位的数值。这个数值定义了上一页或列上剩余的行数。

**CSS**:

```js
/*Print stylesheet*/
@media print {
  .element {
    orphans: 3;
  }
}
/*Screen stylesheet*/
.element {
  orphans: 4;
}
```

## 引号

`quotes` CSS 属性控制要使用的引号类型，其格式如下：

```js
quotes: """ """ "'" "'";
```

### 描述

引号通过 `:before` 和 `:after` 伪元素添加。

添加引号最简单的方法是让浏览器使用 `content` 属性和 `open-quote` 以及 `close-quote` 值来完成：`content: open-quote;` 和 `content: close-quote;`。

我们也可以声明我们想要使用的引号类型，而不是让浏览器决定。我们将在下面的 CSS 示例中看到这一点。

引号始终以双字符符号开始和结束，例如 `" "` 或例如法语中的 `« »`。但是，如果有嵌套引号，这个嵌套引号使用单字符符号，例如 `' '` 或例如法语中的 `‹ ›`。

这个属性支持两个值：`none` 和 `[string string]` 值。

#### none

使用 `content` 属性时不会生成引号。

#### [string string +] 值

每个 `string` 代表一对引号。第一个 `string` 代表外层引号，第二个 `string` 代表嵌套层引号。

`+` 符号表示我们可以添加更深层次的嵌套引号，但实际上并不必要，两层通常适用于大多数情况。

考虑到这些考虑因素，我们可以使用 `quotes` 属性创建以下引号：

```js
CSS:/*Quotation marks inserted by the browser " " and ' '*/
p:before { content: open-quote; }
p:after { content: close-quote; }
/*Custom declared quotation marks*/
p { quotes: """ """ "'" "'"; }
/*French quotation marks*/
p { quotes: "«" "»" "‹" "›"; } 
```

## widows

让我们先澄清一下术语 **widows**。孤儿是指当文本块跨越两页时，出现在下一页的段落行。在 Web 世界中，这体现在跨越几个列的文本中，其中段落的最后一行出现在下一列。

从编辑的角度来看，这非常不好。推荐的孤儿处理方法是至少在下一页或列上留下三行。行数越多越好。

CSS 的 `widows` 属性控制下一页将出现的行数，其格式如下：

```js
widows: 3;
```

### 描述

`widows` 属性在打印样式表中非常有用，但在使用 CSS 列时也可以工作。

它支持不带单位的数值。这个数值定义了下一页或列将出现的行数。

**CSS**:

```js
/*Print stylesheet*/
@media print {
  .element {
    widows: 3;
  }
}
/*Screen stylesheet*/
.element {
  widows: 4;
}
```

## writing-mode

`writing-mode` CSS 属性定义文本行或甚至块级元素布局的方向，无论是垂直还是水平，其格式如下：

```js
writing-mode: vertical-rl;
```

### 描述

这个属性支持三个关键字值：`horizontal-tb`、`vertical-rl` 和 `vertical-lr`。

#### horizontal-tb

这意味着*水平*，*从上到下*。内容从左到右，从上到下流动。

#### vertical-rl

这意味着*垂直*，*从右到左*。内容被旋转 90 度并垂直流动。

为了更好地理解，可以这样想：将你的头向右肩倾斜，在这个时候，你将能够从顶部（在你倾斜头部之前在右边）读到文本流动到底部（在你倾斜头部之前在左边）。

#### vertical-lr

这意味着*垂直*，*从左到右*。内容被旋转 90 度并垂直流动。

然而，与 `vertical-rl` 不同，当你将头靠在右肩上时，内容从底部（在你倾斜头部之前在左边）流向顶部（在你倾斜头部之前在右边），并且文本行是颠倒的。

**CSS**:

```js
.element {
  writing-mode: vertical-rl;
}
```

## letter-spacing

`letter-spacing` CSS 属性定义了文本行中字符之间的空间，它看起来像这样：

```js
letter-spacing: 5px;
```

### 描述

`letter-spacing` 属性支持以下关键字值：`normal` 和 `length`。

负值是有效的。

### 小贴士

除非你理解可读性和设计影响，否则大多数字体中的默认字母间距是理想的，很少需要更改。

**CSS**:

```js
.element {
  letter-spacing: 5px;
}
```

## white-space

`white-space` CSS 属性定义了元素内部空白将被如何处理，它看起来像这样：

```js
white-space: pre-wrap;
```

### 描述

这个属性支持五个关键字值：`normal`、`nowrap`、`pre`、`pre-wrap` 和 `pre-line`。

#### normal

这是默认值。如果有两个或更多空格连在一起，它们将合并为一个空格。这是 HTML 文档的正常行为。它会在必要时换行文本。

#### nowrap

多个空格将合并为一个空格，除非在标记中有 `<br>` 标签，否则行永远不会自动换行。

#### pre

它与 HTML 元素 `<pre>` 相同，意味着**预格式化**。这个值将尊重文本中的所有空格，但除非在标记中有 `<br>` 标签，否则不会换行，就像 `nowrap` 一样。这个值非常适合显示短段代码。

#### pre-wrap

这个值尊重所有空格，并在必要时换行。这个值非常适合显示长段代码。

#### pre-line

这个值将多个空格合并为一个空格，并在必要时换行。

## tab-size

`tab-size` CSS 属性允许我们定义制表符可以有多少个空格。

正如我们在之前的 `white-space` 属性描述中看到的，默认情况下多个空格会被合并为一个空格。因此，这个属性只会在 `<pre>` 标签内的元素或具有尊重多个空格的 `white-space` 属性（`pre` 和 `pre-wrap`）的元素上起作用。

这个属性非常适合显示代码。

制表符字符的默认空格数是 8。但我们，作为网页设计师和开发者，很挑剔，喜欢四格或有时是两格。使用 `tab-size` 属性，我们可以将其默认值修改为我们想要的任何值。

### 描述

`tab-size` 属性支持两个值：一个数值和一个长度值。

+   **数值**：它只是一个没有单位的数字。负值无效。

+   **长度值**：当我们使用长度值时，我们使用以下单位之一：`px`、`em`、`in`、`mm`、`cm`、`vw` 等。

**CSS**：

```js
pre {
  white-space: pre-wrap;
  tab-size: 4;
}
```

# 分页

尽管我们为网络构建网站和应用，但有时我们制作的内容也会被打印出来。

以下属性将帮助我们使我们的内容在打印时更具可读性和跨页处理得更好。

让我们来检查这些分页属性。

## page-break-after

`page-break-after` CSS 属性定义了在特定元素之后分页的位置，看起来是这样的：

```js
page-break-after: always;
```

### 描述

这意味着当创建新的分页符时，将打印新的一页。

它仅适用于块级元素。此外，由于此属性用于打印，通常在 `@print` 媒体查询内看到它。

此属性支持五个关键字值：`always`、`auto`、`avoid`、`left` 和 `right`。

+   `always`：此值将在元素后强制分页。

+   `auto`：这是默认值。它创建自动分页符。

+   `avoid`：如果可能，这将不允许在元素后有任何分页。

+   `left`：这将强制在元素后进行一个或两个分页，以便使下一页成为左侧页面。

+   `right`：这将强制在元素后进行一个或两个分页，以便使下一页成为右侧页面。

**CSS**：

```js
@media print {
  .element {
    page-break-after: always;
  }
}
```

## page-break-before

`page-break-before` CSS 属性与 `page-break-after` CSS 属性类似。然而，它定义了在特定元素之前分页的位置，看起来是这样的：

```js
page-break-before: always;
```

### 描述

当创建新的分页符时，将打印新的一页。

它仅适用于块级元素。此外，由于此属性用于打印，通常在 `@print` 媒体查询内看到它。

此属性支持与 `page-break-before` 属性相同的五个关键字值：`always`、`auto`、`avoid`、`left` 和 `right`。

请参阅 `page-break-before` CSS 属性部分，了解每个关键字值的描述。

**CSS**：

```js
@media print {
  .element {
    page-break-before: always;
  }
}
```

## page-break-inside

`page-break-inside` CSS 属性与 `page-break-before` 和 `page-break-after` 属性类似。然而，它定义了在特定元素内部分页的位置，看起来是这样的：

```js
page-break-inside: avoid;
```

### 描述

这意味着当创建新的分页符时，将打印新的一页。

尽管这个属性只支持两个关键字值，即 `auto` 和 `avoid`。

+   `auto`：这是默认值。它创建自动分页符。

+   `avoid`：如果可能，这将不允许在元素内部有任何分页。

**CSS**：

```js
@media print {
  .element {wdwqd
    page-break-before: avoid;
  }
}
```
