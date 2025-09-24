# 第七章：CSS 函数

CSS 函数在 CSS 中用于许多事情。它们可以用来创建特殊类型的进程，例如创建动画或使用自定义字体，或者创建透明度或变换元素在二维和三维平面上的视觉效果。

让我们看看 CSS 函数都是关于什么的。

# 过滤器

CSS 过滤器允许我们以不同的方式操纵元素的颜色。

## brightness()

`brightness()` CSS 函数与 `filter` 属性一起使用，其语法如下：

```js
filter: brightness(20%);
```

### 描述

`brightness()` 函数修改图像的照明。值可以声明为不带单位的百分比或数字，例如，`10%` 和 `0.5%`。

值为 `100%` 保持元素不变；值为 `0%` 使元素完全变黑。允许值超过 `100%`，这将产生更强烈的效果。值的范围没有限制。

值为 `1` 保持元素不变；值为 `0` 使元素完全变黑。允许值超过 `1`，这将产生更强烈的效果。值的范围没有限制。此外，对于百分比和数字，负值均无效。

**CSS**:

```js
.element {
  filter: brightness(20%);
}
```

## contrast()

`contrast()` CSS 函数与 `filter` 属性一起使用，其语法如下：

```js
filter: contrast(10);
```

### 描述

`contrast()` 函数修改元素的对比度。值可以声明为不带单位的百分比或数字，例如，`10%` 和 `0.5%`。

值为 `100%` 保持元素不变；值为 `0%` 使元素完全变黑。允许值超过 `100%`，这将产生更强烈的效果。值的范围没有限制。

值为 `1` 保持元素不变；值为 `0` 使元素完全变黑。允许值超过 `1`，这将产生更强烈的效果。值的范围没有限制。此外，负值无效，并且允许使用小数。

**CSS**:

```js
.element {
  filter: contrast(10);
}
```

## grayscale()

`grayscale()` CSS 函数与 `filter` 属性一起使用，其语法如下：

```js
filter: grayscale(.8);
```

### 描述

`grayscale()` 函数将元素转换为黑色调。值可以声明为不带单位的百分比或数字，例如，`10%` 和 `0.5%`。

值为 `0%` 保持元素不变；值为 `100%` 使元素变为灰度。不允许值超过 `100%`。

值为 `0` 保持元素不变；值为 `1` 使元素变为灰度。不允许值超过 `1`。此外，对于两种情况，负值均无效。允许使用小数。

## invert()

`invert()` CSS 函数与 `filter` 属性一起使用，其语法如下：

```js
filter: invert(1);
```

### 描述

`invert()` 函数反转元素的色彩。如果用于图像，它会使图像看起来像负片。

值为 `100%` 完全反转元素的色彩；值为 `0%` 保持元素不变。不允许值超过 `100%`。

值为 `1` 完全反转元素的颜色；值为 `0` 保持元素不变。不允许超过 `1` 的值。也不允许负值。两者都允许小数值。

## hue-rotate()

`hue-rotate()` CSS 函数与 `filter` 属性一起使用，看起来像这样：

```js
filter: hue-rotate(80deg);
```

### 描述

`hue-rotate()` 函数应用于元素，进行色调旋转。它接受一个 *角度* 值。

角度值定义了元素样本在色轮上修改的度数。

没有最大值。然而，如果值大于 `360deg`，旋转将只是绕一圈。例如，如果我们声明 `380deg`，那么它等同于 `20deg`。

## blur()

`blur()` CSS 函数与 `filter` 属性一起使用，看起来像这样：

```js
filter: blur(10px);
```

### 描述

`blur()` 函数产生 *模糊* 效果。值声明为 *长度* 值（`px`、`em`、`in`、`mm`、`cm`、`vw` 等）。值越高，模糊效果越强烈，反之亦然。

不允许百分比和负值，但允许小数值。

## saturate()

`saturate()` CSS 函数与 `filter` 属性一起使用，看起来像这样：

```js
filter: saturate(300%);
```

### 描述

它会影响元素的饱和度水平。值可以声明为 *百分比* 或不带单位的 *数字*，例如 `10%` 和 `0.5%`。

元素的默认饱和度值为 `100%`，或使用无单位数字时为 `1`。

值为 `0%` 完全去饱和元素（移除所有颜色，使元素变为灰度）；值为 `100%` 保持元素不变。允许超过 `100%` 的值，以创建更强烈的效果。

值为 `0` 完全去饱和元素（移除所有颜色，使元素变为灰度）；值为 `1` 保持元素不变。允许超过 `1` 的值，以创建更强烈的效果。

## sepia()

`sepia()` CSS 函数与 `filter` 属性一起使用，看起来像这样：

```js
filter: sepia(100%);
```

### 描述

`sepia()` 函数将元素转换为棕褐色。想象一下灰度图像，但以棕色色调呈现。

值为 `100%` 完全将元素转换为棕褐色；值为 `0%` 保持元素不变。不允许超过 `100%` 的值。

值为 `1` 完全将元素转换为棕褐色；值为 `0` 保持元素不变。不允许超过 `1` 的值。

此外，对于两者，负值都是无效的。

## 变换

CSS 变换已经变得如此流行，以至于现在在网站上几乎看不到没有某种变换的情况，例如按钮形状、动画和布局。

让我们看看变换 CSS 函数。

## matrix()

`matrix()` CSS 函数与 `transform` 属性一起使用，看起来像这样：

```js
matrix(0.5, 0, 0.0881635, 0.5, 0, 0);
```

### 描述

`matrix()` 函数是所有变换属性的缩写，因为它们可以在这里组合。此函数用于定义二维变换矩阵。

这个函数需要扎实的数学理解，但在现实中，这个函数并不是手动完成的。相反，我们可以使用像 Eric Meyer 和 Aaron Gustafson 的《矩阵解析》这样的工具([`tiny.cc/eric-meyer-matrix`](http://tiny.cc/eric-meyer-matrix))。

`matrix()` 函数的高级数学解释超出了本书的范围。然而，对于非常详细的解释，你可以参考以下两篇文章中的任何一篇：

+   《理解 CSS 转换矩阵》由 Tiffany Brown 撰写([`tiny.cc/css-matrix-1`](http://tiny.cc/css-matrix-1))

+   《CSS3 矩阵变换对数学挑战者的解释》由 Zoltan Hawryluk 撰写([`tiny.cc/css-matrix-2`](http://tiny.cc/css-matrix-2))

**CSS**:

```js
/*This*/
.element { transform: skew(10deg) scale(.5); }
/*Is the same as this*/
.element { transform: matrix(0.5, 0, 0.0881635, 0.5, 0, 0); }
```

## matrix3d()

`matrix3d()` CSS 函数与`transform`属性一起使用，其形式如下：

```js
matrix3d(0.852825, 0.195593, -0.484183, 0, 0.0958426, 0.852825, 0.513326, 0, 0.513326, -0.484183, 0.708564, 0, 0.948667, 1.04842, 0.0291436, 1);
```

### 描述

就像二维的 `matrix()` 函数一样，`matrix3d()` 函数是一个简写，但这个简写适用于 4 x 4 网格中所有 3D 转换属性。

这个函数需要扎实的数学理解，但在现实中，这个函数并不是手动完成的。相反，我们可以使用像 Eric Meyer 和 Aaron Gustafson 的《矩阵解析》这样的工具([`tiny.cc/eric-meyer-matrix`](http://tiny.cc/eric-meyer-matrix))。

**CSS**:

```js
/*This*/
.element { transform: rotate3d(10, 10, 1, 45deg) translate3d(1px, 1px, 0); }
/*Is the same as this*/
.element { transform: matrix3d(0.852825, 0.195593, -0.484183, 0, 0.0958426, 0.852825, 0.513326, 0, 0.513326, -0.484183, 0.708564, 0, 0.948667, 1.04842, 0.0291436, 1); }
```

## rotate()

`rotate()` CSS 函数与`transform`属性一起使用，其形式如下：

```js
rotate(10deg);
```

### 描述

`rotate()` 函数在二维空间中围绕一个固定点旋转元素。它接受一个使用`deg`、`grad`、`rad`或`turn`单位的 *角度* 值。`deg`单位最常用。负值是有效的。

**CSS**:

```js
.element {
  transform: rotate(10deg);
}
```

## rotate3d()

`rotate3d()` CSS 函数与`transform`属性一起使用，其形式如下：

```js
rotate3d(1, 2, 1, 10deg);
```

### 描述

`rotate3d()` 函数通过 *X*、*Y* 和 *Z* 轴在三维平面上围绕一个固定位置旋转元素。它接受四个值：对应于 *X*、*Y* 和 *Z* 轴的三个无单位的 *数字* 值，以及一个定义旋转量的 *角度* 值。

正值在相应轴上按顺时针方向旋转元素。负值按逆时针方向旋转元素。

**CSS**:

```js
.element {
  transform: rotate3d(1, 2, .5, 10deg);
}
```

## rotateX()

`rotateX()` CSS 函数与`transform`属性一起使用，其形式如下：

```js
transform: rotateX(25deg);
```

之前的代码与以下代码类似：

```js
transform: rotate3d(1, 0, 0, 25deg);
```

### 描述

`rotateX()` 函数在三维平面上沿 *X* 轴旋转元素。它接受一个 *角度* 值。

正值按顺时针方向旋转元素。负值按逆时针方向旋转元素。

**CSS**:

```js
/*This*/
.element { transform: rotateX(25deg); }
/*Is the same as this*/
.element { transform: rotate3d(1, 0, 0, 25deg); }
```

## rotateY()

`rotateY()` CSS 函数与`transform`属性一起使用，其形式如下：

```js
transform: rotateY(75deg);
```

上面的行与这一行相同：

```js
transform: rotate3d(0, 1, 0, 75deg);
```

### 描述

`rotateY()` 函数在三维平面上沿 *Y* 轴旋转元素。它接受一个 *角度* 值。

正值按顺时针方向旋转元素。负值按逆时针方向旋转元素。

**CSS**:

```js
/*This*/
.element { transform: rotateY(75deg); }
/*Is the same as this*/
.element { transform: rotate3d(0, 1, 0, 75deg); }
```

## rotateZ()

`rotateY()` CSS 函数与 `transform` 属性一起使用，其形式如下：

```js
transform: rotateZ(33deg);
```

这与以下内容相同：

```js
transform: rotate3d(0, 0, 1, 33deg);
```

### 描述

`rotateY()` 函数用于在三维平面上沿 *Z* 轴旋转元素。它接受一个 *角度* 值。

正值会使元素顺时针旋转。负值会使元素逆时针旋转。

**CSS**:

```js
/*This*/
.element { transform: rotateZ(33deg); }
/*Is the same as this*/
.element { transform: rotate3d(0, 0, 1, 33deg); }
```

## scale()

`scale()` CSS 函数与 `transform` 属性一起使用，其形式如下：

```js
.element { transform: scale(2); }
```

或者：

```js
.element { transform: scale(2, 3); }
```

### 描述

`scale()` 函数用于在二维平面上改变元素的大小，使其变大或变小。它支持一个或两个无单位的数值，其中第二个值是可选的。数值表示元素应该缩放多少次。例如，值为 `2` 表示元素被缩放（放大）到 `200%`；值为 `0.5` 表示元素应该缩放（缩小）到 `50%`。

第一个值代表水平缩放，第二个值代表垂直缩放。如果只声明一个值，则表示两个方向将使用相同的值。

允许使用负值。然而，当使用负值时，元素会被翻转。

### 提示

当元素被缩放时，它不会影响布局；它将根据源顺序简单地重叠或出现在其他元素下方。

**CSS**:

```js
/*Element is flipped in both directions and scaled to 200% its size*/
.element { transform: scale(-2); }
/*Element is scaled to 200% horizontally and 300% vertically*/
.element { transform: scale(2, 3); }
```

## scale3d()

`scaled3d()` CSS 函数与 `transform` 属性一起使用，其形式如下：

```js
transform: scale3d(2, 2, 2);
```

### 描述

`scaled3d()` 函数通过 *X*、*Y* 和 *Z* 轴改变元素在三维平面上的大小，使其变大或变小。

它支持三个无单位的数值，这些数值是必需的。允许使用负值。然而，当使用负值时，元素会被翻转。

**CSS**:

```js
/*The element is twice the size*/
.element { transform: scale3d(2, 2, 2); }
/*Flipped element in both X and Y-axis while scaled to 300%, and 200% on the Z-axis*/
.element { transform: scale3d(-3, -3, 2); }
```

## scaleX()

`scaleX()` CSS 函数与 `transform` 属性一起使用，其形式如下：

```js
transform: scaleX(-2);
```

### 描述

`scaleX()` 函数用于在二维平面上沿 *X* 轴改变元素的大小。它支持一个无单位的数值。

允许使用负值。然而，当使用负值时，元素会被翻转。

**CSS**:

```js
.element {
  transform: scaleX(-2);
}
```

## scaleY()

`scaleY()` CSS 函数与 `transform` 属性一起使用，其形式如下：

```js
transform: scaleY(4);
```

### 描述

`scaleY()` 函数用于在二维平面上沿 *Y* 轴改变元素的大小。它支持一个无单位的数值。

允许使用负值。然而，当使用负值时，元素会被翻转。

**CSS**:

```js
.element {
  transform: scaleY(4);
}
```

## scaleZ()

`scaleZ()` CSS 函数与 `transform` 属性一起使用，其形式如下：

```js
transform: scaleZ(3);
```

### 描述

`scaleZ()` 函数用于在二维平面上沿 *Y* 轴改变元素的大小。它支持一个无单位的数值。

允许使用负值。然而，当使用负值时，元素会被翻转。

**CSS**:

```js
.element {
  transform: scaleY(4);
}
```

## skew()

`skew()` CSS 函数与 `transform` 属性一起使用，其形式如下：

```js
transform: skew(20deg);
```

或者您也可以使用以下代码：

```js
transform: skew(20deg, 25deg);
```

### 描述

`skew()` 函数在二维平面上对元素进行倾斜或倾斜 *X* 轴或 *X* 和 *Y* 轴。例如，平行四边形是一个倾斜的矩形。

它支持一个或两个角度值：第一个对应于 *X* 轴，第二个对应于 *Y* 轴。如果只声明了一个值，则元素仅在 *X* 轴上倾斜。允许使用负值。

建议您使用 `skewX()` 或 `skewY()` 函数而不是 `skew()`，因为 `skew()` 已经从规范中移除（尽管大多数浏览器仍然支持它）。

**CSS**:

```js
/*One value only affects the element on the X-axis*/
.element { transform: skew(20deg); }
/*Two values affects the element on both X and Y-axis*/
.element { transform: skew(20deg, 25deg); }
```

## skewX()

`@skewX()` CSS 函数与 `transform` 属性一起使用，看起来像这样：

```js
transform: skewX(40deg);
```

### 描述

`@skewX()` 函数在二维平面上对元素进行倾斜或倾斜 *X* 轴。

它支持一个角度值。允许使用负值。

**CSS**:

```js
.element {
  transform: skewX(40deg);
}
```

## skewY()

`@skewY()` CSS 函数与 `transform` 属性一起使用，看起来像这样：

```js
transform: skewY(36deg);
```

### 描述

`@skewY()` 函数在二维平面上对元素进行倾斜或倾斜 *Y* 轴。

它支持一个角度值。允许使用负值。

**CSS**:

```js
.element {
  transform: skewY(36deg);
}
```

## steps()

`steps()` 时间函数与 `transition-timing-function` 或 `animation-timing-function` 属性一起使用，看起来像这样：

```js
transition-timing-function: steps(3);
animation-timing-function: steps(3);
```

### 描述

`steps()` 时间函数将过渡或动画分成相等大小的间隔。我们还可以指定过渡或动画的步骤是在间隔的 `start` 还是 `end` 处发生。如果没有声明参数，则默认为 `end` 值。

它支持一个数值值，或者一个数值值和一个可选的 `start` 或 `end` 值。

通过一个例子来理解 `start` 或 `end` 的工作方式是最好的：使用 `start` 时，动画将立即开始，使用 `end` 时，将稍微延迟。

**CSS**:

```js
/*The transition is divided in 3 equal size intervals*/
.element { transition-timing-function: steps(3); }
/*The transition is divided in 3 equal size intervals but it will delay a bit before it starts*/
.element { transition-timing-function: steps(3, end); }
```

## translate()

`translate()` CSS 函数与 `transform` 属性一起使用，看起来像这样：

```js
transform: translate(20px);
```

或者像这样：

```js
transform: translate(20px, -50%);
```

### 描述

`translate()` 函数影响元素在二维平面上的位置，在 *X* 轴或 *X* 和 *Y* 轴上。

它支持长度和百分比值。允许使用负值。它支持一个或两个长度和百分比值；第一个对应于 X 轴，第二个对应于 Y 轴。如果只声明了一个值，则元素仅在 X 轴上移动。允许使用负值。

**CSS:**

```js
/*One value, the element is moved on the X-axis only*/
.element { transform: translate(20px); }
/*Two values, the element is moved on the X and Y-axis*/
.element { transform: translate(20px, -50%); }
```

## translate3d()

`translate3d()` CSS 函数与 `transform` 属性和 `perspective` 函数一起使用，看起来像这样：

```js
transform: perspective(100px)  translate3d(75px, 75px, -200px);
```

### 描述

`translate3d()` 函数用于在三维平面上沿 *X*、*Y* 和 *Z* 轴移动元素。

它支持长度和百分比值。允许使用负值。

为了能够看到这个函数的工作效果，我们需要给相关元素使用`perspective`函数创建一个三维平面，否则`translate3d()`声明将没有任何效果。

**CSS**:

```js
.element {
  transform: perspective(100px) translate3d(75px, 75px, -200px);
}
```

## translateX()

`translateX()` CSS 函数与`transform`属性一起使用，其语法如下：

```js
transform: translateX(99%);
```

### 描述

`translateX()`函数用于在二维平面上沿*X*轴移动元素。

它支持长度和百分比值。允许使用负值。

**CSS**:

```js
.element {
  transform: translateX(99%);
}
```

## translateY()

`translateY()` CSS 函数与`transform`属性一起使用，其语法如下：

```js
transform: translateY(55px);
```

### 描述

这用于在二维平面上沿*Y*轴移动元素。它支持长度和百分比值。允许使用负值。

**CSS**:

```js
.element {
  transform: translateY(55px);
}
```

## translateZ()

`translateZ()` CSS 函数与`transform`属性和`perspective`函数一起使用，其语法如下：

```js
transform: perspective(100px) translateZ(77px);
```

### 描述

这用于在三维平面上沿*Z*轴移动元素。它支持长度和百分比值。允许使用负值。

为了能够看到这个函数的工作效果，我们需要给相关元素使用`perspective`函数创建一个三维平面；否则，`translateZ()`声明将没有任何效果。

**CSS**:

```js
.element {
  transform: perspective(100px) translateZ(77px);
}
```

# 颜色

颜色可以成就或毁掉一个设计，有很多方法可以创建调色板和所有那些好东西。

让我们看看*HSL(a)*和*RGB(a)*。

## hsl()和 hsla()

`hsl()`和`hsla()` CSS 函数表示法以 HSL/HSLa 格式设置颜色，其语法如下：

```js
background-color: hsl(10, 20%, 30%);
background-color: hsla(10, 20%, 30%, .5);
```

### 描述

**HSL**代表**色调、饱和度和亮度**（或亮度）。**a**代表**Alpha**，它是 alpha 通道，我们用它来声明颜色的透明度。

`hsl()`函数支持用逗号分隔的三个或四个值。第一个值是色调，它是基础颜色。这个值以无单位的*数字*表示。这个数字代表在色轮上的角度（*10 = 10º*），范围从 0 到 360。因此，0 和 360 是红色，90 是黄绿色，180 是青色，270 是蓝紫色。

第二个值是饱和度，这基本上是基础颜色的量。这个值以百分比表示。`0%`表示完全没有基础颜色，显示灰色。`100%`表示基础颜色是完整的。

第三个值是亮度，也称为亮度。这基本上是基础颜色的亮度。`0%`表示没有亮度，因此是黑色。`100%`表示完全亮度，因此看起来是白色。`50%`表示基础颜色是完整的。

第四个值是 alpha 通道。这是颜色的透明度。它以无单位的*数字*小数形式从`0`到`1`声明。完全透明是`0`，`1`是完全不透明。

HSL 颜色命名系统相对于 RGB 的一个巨大优势是它更直观。一旦我们选择了一个基础颜色，我们就可以通过只改变饱和度和亮度值来轻松地创建基于该颜色的调色板。

你可以在 CodePen 中看到 HSL 颜色轮：[`tiny.cc/hsl-color-wheel`](http://tiny.cc/hsl-color-wheel)

**CSS**:

```js
/*HSL*/
.element { background-color: hsl(240, 100%, 50%); }
/*HSLa*/
.element { background-color: hsla(10, 20%, 30%, .5); }
```

## `rgb()`和`rgba()`

`rgb()`和`rgba()` CSS 功能符号设置 RGB/RGBa 格式的颜色，它们看起来是这样的：

```js
background-color: rgb(240, 100, 50);
background-color: rgba(10, 20, 30, .5);
```

### 描述

**RGB**代表**红色、绿色和蓝色**。*a*代表**Alpha**，这是我们声明颜色透明度的 alpha 通道。

这支持三个或四个由逗号分隔的无单位*数值*，或者三个*百分比*值和一个无单位*数值*。最后一个值用于 alpha 通道。

*数值*的范围是 0 到 255。*百分比*值的范围是 0%到 100%。例如，我们可以将绿色表示为`rgb(0, 255, 0)`或`rgb(0, 100%, 0)`。

正如我刚才提到的，第四个值是 alpha 通道。这是颜色的透明度。它用一个无单位的*数值*十进制值从`0`到`1`声明。完全透明是`0`，`1`是完全不透明。

**CSS**:

```js
/*RGB*/
.element { background-color: rgb(240, 100, 50); }
/*RGBa*/
.element { background-color: rgba(10, 20, 30, .5); }
```

# 渐变

对于那些不知道的人来说，CSS 渐变实际上是图像。但这些图像是在浏览器看到声明渐变颜色时由浏览器创建的。这些图像的特点是它们是即时创建的，不会引起任何 HTTP 请求。

CSS 渐变功能非常强大，我们不仅可以创建任何方向和各种形状的渐变，还可以创建令人惊叹的图案。

话虽如此，Lea Verou 有一个用渐变创建的 CSS 图案库，每个阅读这本书的人都应该将其收藏。在这里查看：[`tiny.cc/leave-verou-css3-patterns`](http://tiny.cc/leave-verou-css3-patterns)

让我们看看如何在 CSS 中创建渐变。

## `linear-gradient()`

`linear-gradient()` CSS 函数创建一个从一种颜色过渡到另一种颜色的*线*形渐变。它的最简单形式如下所示：

```js
background-image: linear-gradient(red, blue);
```

### 描述

我们可以创建遵循任何方向的线性渐变，称为*渐变线*：从左到右，从右到左，从上到下，从下到上，对角线，以及在任何 360º半径内的任何角度。

如果没有指定渐变线的方向，则默认值是从上到下。

渐变线中可以声明任意数量的颜色。从技术上来说，没有限制，但从设计角度来看，我们应始终尝试保持简单。至少需要两个颜色值。

`linear-gradient()`函数支持所有颜色模式：`HEX`，`RGB`，`RGBa`，`HSL`，`HSLa`，和*颜色名称*。

#### 方向

我们还可以通过一个*角度*值或四个*关键字*值来声明渐变线的方向：`to top`，`to bottom`，`to left`，和`to right`。

+   `to top`：渐变将从底部开始，到顶部结束

+   `to bottom`: 渐变将从顶部开始，并在底部结束

+   `to left`: 渐变将从右侧开始，并在左侧结束

+   `to right`: 渐变将从左侧开始，并在右侧结束

*角度* 值在声明的开始处定义，范围从 0 到 360。较大的值会绕圆周循环。

#### 颜色停止

我们还可以定义颜色在渐变中的停止位置。颜色停止是颜色值和 *停止位置* 的组合，其中停止位置是可选的。

停止位置可以用任何 *长度* 值或 *百分比* 值声明，并跟在颜色值之后。

### 小贴士

百分比值更常用，因为它们可以与元素一起缩放。像素值也可以，但它们没有相对单位那样灵活。

颜色停止非常灵活，因为它们允许我们在颜色之间创建平滑的过渡。这对于制作需要平滑颜色过渡的图案或其他类型的图形非常有用，例如国旗。

当没有声明 *停止位置* 时，浏览器会在渐变线上均匀地分布渐变颜色。

**CSS**:

```js
/*Basic gradient. Colors are distributed evenly along the gradient line*/
.element { background-image: linear-gradient(red, blue); }
/*Gradient goes from right to left and starts with color red*/
.element { background-image: linear-gradient(to left, red, blue); }
/*Gradient line is diagonal; inclined 170 degrees*/
.element { background-image: linear-gradient(170deg, red, blue); }
/*Gradient with stop positions in percentages*/
.element { background-image: linear-gradient(red 50%, blue 75%); }
/*Gradient with stop positions in pixels*/
.element { background-image: linear-gradient(red 100px, blue 150px); }
/*Colombian flag (yellow, blue and red bands) made with solid color transitions using stop positions*/
.element { background-image: linear-gradient(#fcd116 50%, #003893 50%, #003893 75%, #ce1126 75%); }
```

## radial-gradient()

CSS 函数 `radial-gradient()` 创建的渐变从一种颜色过渡到另一种颜色，但以圆形或椭圆形的形式，其最简单的形式如下：

```js
background-image: radial-gradient(orange, green);
```

### 描述

径向渐变有三个部分：其 *中心*、其 *结束形状* 和 *颜色停止*。

*中心* 定义了从元素中开始径向渐变的点；径向渐变不必从元素的中心开始。*结束形状* 定义了径向渐变是圆形还是椭圆形。如果没有声明 `circle` 关键字，则默认形状是椭圆形。*颜色停止* 是构成渐变的颜色，如果声明了，任何可选的 *停止位置*。记住，停止位置可以用任何 *长度* 值或 *百分比* 值声明，并跟在颜色值之后。

至少需要两种颜色来创建径向渐变，或者任何类型的渐变。

#### 位置

我们可以定义径向渐变的中心在元素内的位置。正如我之前提到的，默认位置是在元素的中心。

要声明一个特定位置，我们使用关键字 `at` 并定义 *X* 和 *Y* 轴坐标。这个值应该在声明任何颜色值之前，但在 *结束形状* 之后。

*X* 和 *Y* 轴坐标可以用任何 *长度* 值、*百分比* 值或任何关键字值（`top`、`right`、`bottom` 和 `left`）声明。这基本上与我们声明元素上的 `background-position` 的方式相同。

位置需要声明一个 *结束形状*，可以是 `circle` 或 `ellipse`；否则，声明无效。

#### 尺寸

我们还可以改变径向渐变的大小。渐变的*大小*在*位置*之前声明，但它可以在*结束形状*之前或之后。它可以接受一个或两个值，用于*宽度*和*高度*。如果声明了一个值，它将用于两者。

大小可以用*长度*值、*百分比*值或四个*关键字*值之一来定义：`closest-corner`、`farthest-corner`、`closest-side`和`farthest-side`。

+   `closest-corner`：渐变的大小取决于离中心最近的角。

+   `farthest-corner`：渐变的大小取决于离中心最远的角。

+   `closest-side`：渐变的大小取决于离中心最近的边。

+   `farthest-side`：渐变的大小取决于离中心最远的边。

**CSS**：

```js
/*Basic gradient. Colors are distributed evenly in the ellipse*/
.element { background-image: radial-gradient(red, blue); }
/*Ending shape declared as circle*/
.element { background-image: radial-gradient(circle, red, blue); }
/*Position declared (only one value). Gradient will start at the left and center*/
.element { background-image: radial-gradient(circle at left, red, blue); }
/*Position declared (two values)*/
.element { background-image: radial-gradient(circle at left top, red, blue); }
/*Position declared with percentages. An ending shape value is required*/
.element { background-image: radial-gradient(circle at 25% 75%, red, blue); }
/*Size of the gradient declared in pixels*/
.element { background-image: radial-gradient(100px 50px ellipse at 25% 75%, red, blue); }
/*Size of the gradient relative to the farthest side of the element. Ending shape can be after or before size*/
.element { background-image: radial-gradient(circle farthest-side at 25% 75%, red, blue); }
```

## repeating-linear-gradient()

`repeating-linear-gradient()` CSS 函数用于重复渐变图像，其外观如下：

```js
background-image: repeating-linear-gradient(orange 50px, green 75px);
```

### Description

`repeating-linear-gradient()`函数使用与`linear-gradient()` CSS 函数相同的语法和值，因此请参阅该函数以获取所有可用值的详细说明。

为了使`repeating-linear-gradient()`函数正常工作，我们需要在颜色上定义*停止位置*。否则，重复的渐变看起来就像我们只是使用了`linear-gradient()`。

**CSS**：

```js
/*Basic repeating linear gradient*/
.element { background-image: repeating-linear-gradient(orange 50px, green 75px); }
/*Repeating gradient goes from right to left and starts with color orange*/
.element { background-image: repeating-linear-gradient(to left, orange 50px, green 75px); }
/*Repeating gradient line is diagonal; inclined 170 degrees*/
.element { background-image: repeating-linear-gradient(170deg, orange 50px, green 75px); }
/*Repeating gradient with stop positions in percentages*/
.element { background-image: repeating-linear-gradient(orange 25%, green 50%); }
```

## repeating-radial-gradient()

`repeating-radial-gradient()` CSS 函数用于重复渐变图像，其外观如下：

```js
background-image: repeating-radial-gradient(navy 50px, gray 75px);
```

### Description

`repeating-linear-gradient()`函数使用与`radial-gradient()` CSS 函数相同的语法和值，因此请参阅该函数以获取所有可用值的详细说明。

为了使`repeating-radial-gradient()`函数正常工作，我们需要在颜色上定义*停止位置*。否则，重复的渐变看起来就像我们只是使用了`radial-gradient()`。

**CSS**：

```js
/*Basic repeating linear gradient*/
.element { background-image: repeating-radial-gradient(navy 50px, gray 75px); }
/*Ending shape declared as circle*/
.element { background-image: repeating-radial-gradient(circle, navy 50px, gray 75px); }
/*Position declared (only one value). Gradient will start at the left and center*/
.element { background-image: repeating-radial-gradient(circle at left, navy 50px, gray 75px); }
/*Position declared (two values)*/
.element { background-image: repeating-radial-gradient(circle at left top, navy 50px, gray 75px); }
/*Position declared with percentages. Defaults to ellipse shape unless 'circle' is specified*/
.element { background-image: repeating-radial-gradient(at 25% 75%, navy 50px, gray 75px); }
/*Size of the gradient declared in pixels*/
.element { background-image: repeating-radial-gradient(200px 25px at 25% 75%, navy 50px, gray 75px); }
/*Size of the gradient relative to the farthest side of the element. Ending shape can be after or before size*/
.element { background-image: repeating-radial-gradient(circle farthest-side at 25% 75%, navy 50px, gray 75px); }
```

# Values

以下 CSS 函数允许我们为各种结果声明许多自定义值。让我们来看看它们。

## attr()

`attr()` CSS 函数允许我们针对任何 HTML 属性的值并在 CSS 中使用它，其外观如下：

```js
attr(href);
```

### Description

术语**attr**是单词**attribute**的缩写。这个 CSS 函数针对一个*HTML 属性*，并使用其值通过 CSS 完成不同的事情。

在 CSS 中，`attr()`函数最常与`content`属性以及`:after` CSS 伪元素一起使用，以将内容注入文档中，但事实上，`attr()`函数可以与*任何*其他 CSS 属性一起使用。

在 HTML 中，使用`attr()` CSS 函数针对 HTML5 的`data-`或`href`属性是非常常见的。`attr()`函数可以用来针对*任何*HTML 属性。

在 CSS3 中，`attr()` CSS 函数的语法略有不同。它不仅接受属性值，还接受两个额外的参数，即*类型或单位*参数和*属性回退*参数。*类型或单位*参数是可选的。它告诉浏览器哪种类型的属性正在使用，以便解释其值。*属性回退*参数定义了在解析元素的主要属性时出现错误的情况下的回退值。

### 小贴士

新的 CSS3 语法包括*类型或单位*和*属性回退*参数，尚不稳定，并且可能从规范中删除。在决定使用新语法之前，请做好研究。

打印网页文档的一个良好实践是将 URL 打印在链接元素旁边。另一个常见做法是结合使用`attr()` CSS 函数、`content`属性和 HTML5 的`data-`属性在响应式表格中注入单元格（通常是标题）的内容，从而节省空间。

**CSS**:

```js
/*Print the links from the content*/
@media print {
   main a[href]:after {
     content: attr(href);
  }
}
```

## 响应式表格

当视口宽度为 640 像素或更小时，表格将变为响应式。这是通过结合使用`attr()` CSS 函数、`content`属性和 HTML5 的`data-`属性来实现的。

**HTML**:

```js
<table>
  <tr class="headings">
    <th>Plan</th>
    <th>Price</th>
    <th>Duration</th>
  </tr>
  <tr>
    <td data-label="Plan:">Silver</td>
    <td data-label="Price:">$50</td>
    <td data-label="Duration:">3 months</td>
  </tr>
</table>
```

**CSS**:

```js
/*40em = 640÷16*/
@media (max-width:40em) {
  /*Behave like a "row"*/
  td, tr {
    display: block;
  }
  /*Hide the headings but not with display: none; for accessibility*/
  .headings {
    position: absolute;
    top: -100%;
    left: -100%;
    overflow: hidden;
  }
  /*Inject the content from the data-label attribute*/
  td:before {
    content: attr(data-label);
    display: inline-block;
    width: 70px;
    padding-right: 5px;
    white-space: nowrap;
    text-align: right;
    font-weight: bold;
  }
}
```

## calc()

`calc()` CSS 函数允许我们执行数学计算，其格式如下：

```js
width: calc(100% / 2 + 25px);
```

或者像这样：

```js
padding: calc(5 * 2px - .25em);
```

### 描述

我们可以使用加法（`+`）、减法（`-`）、除法（`/`）和乘法（`*`）来进行这些计算。它通常用于计算`width`和`height`的相对值，但如您所见，我们可以使用此函数与任何 CSS 属性一起使用。

需要考虑的一些事情是，在加法（`+`）和减法（`-`）运算符前后都需要有空格，否则，例如，减法可以被认为是具有负值，例如`calc(2.5em -5px)`。这个`calc()`函数是无效的，因为第二个值被认为是负值。减法运算符后面需要有空格。然而，除法（`/`）和乘法（`*`）运算符不需要空格。

现在，在进行除法（`/`）时，右侧的值必须是*数字*值。对于乘法（`*`）操作，至少有一个值必须是*数字*值。

**CSS**:

```js
/* The element's width is half its intrinsic width plus 25px*/
.element { width: calc(100% / 2 + 25px); }
/*The element's padding is 10px minus .25em of the result*/
.element { padding: calc(5 * 2px - .25em); }
```

## url()

`url()` CSS 函数用于指向外部资源，其格式如下：

```js
background-image: url(..images/sprite.png);
```

### 描述

`url()`函数使用 URL 值来指向或链接到资源。**URL**代表**统一资源定位符**。

此函数通常与`background`或`background-image`属性一起使用，但可以与任何接受 URL 作为值的属性一起使用，如`@font-face`、`list-style`、`cursor`等。

URL 可以用单引号（`'` `'`）或双引号（`"` `"`）引用，或者根本不引用。然而，不能有引号样式的组合，例如以单引号开头并以双引号结尾。

此外，在单引号使用的 URL 中的双引号和双引号使用的 URL 中的单引号 *必须* 用反斜杠（`\`）转义。否则，它将破坏 URL。

指向资源的 URL 可以是绝对路径或相对路径。如果是相对路径，它相对于样式表在文件夹结构中的位置，而不是网页本身的位置。

`url()` CSS 函数还支持 Data URI，这基本上是图像的代码。因此，我们不必将选择器指向 `/images` 文件夹中的图像下载，而是可以将实际图像嵌入到 CSS 中。

注意这一点，因为尽管我们减少了 HTTP 请求（这是一个 *巨大的* 成功），但如果图像发生变化，我们可能会使 CSS 文件变得更大，并且更难维护。也可能存在潜在的性能和渲染阻塞问题。

关于 Data URI 的更多信息，您可以阅读尼古拉斯·扎卡斯的这篇文章：*Data URIs Explained* ([`www.nczonline.net/blog/2009/10/27/data-uris-explained/`](https://www.nczonline.net/blog/2009/10/27/data-uris-explained/)).

**CSS**:

```js
/*Colombian flag icon as Data URI. No quotes in URL*/
.element { background: url(data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAALCAYAAAB24g05AAAABHNCSVQICAgIfAhkiAAAAAlwSFlzAAALEgAACxIB0t1+/AAAABx0RVh0U29mdHdhcmUAQWRvYmUgRmlyZXdvcmtzIENTNui8sowAAAAkSURBVCiRY3y7leE/AwWAiRLNw8QARgbO40M8EBnvMgz1dAAAkeoGYBluZXgAAAAASUVORK5CYII=) #ccc no-repeat center; }

/*Custom cursor. Single quotes in URL*/
.element { cursor: url('../images/cursor.cur') 10 10, default; }

/*Web font. Double quotes in URL*/
@font-face {
  font-family: Franchise; 
  src: url("../fonts/franchise.woff") format("woff");
}
```

## `cubic-bezier()`

`cubic-bezier()` 函数允许我们创建自定义的加速度曲线，其外观如下：

```js
animation-timing-function: cubic-bezier(.42, 0, 1, 1);
```

### 描述

`cubic-bezier()` 函数与 `animation-timing-function` 和 `transition-timing-function` CSS 属性一起使用。大多数用例都可以从我们在第四章中提到的已定义缓动函数中受益，*CSS 属性 - 第一部分*（`ease`、`ease-in`、`ease-out`、`ease-in-out` 和 `linear`）；如果你喜欢冒险，`cubic-bezier()` 是你的最佳选择。

请参考第四章中关于 `animation-timing-function` CSS 属性的说明，以了解 **贝塞尔** 曲线的外观。`cubic-bezier()` 函数以以下形式接受四个参数：

```js
animation-timing-function: cubic-bezier(a, b, a, b);
```

让我们用 `cubic-bezier()` 函数表示所有五个预定义的缓动函数：

+   `ease`: `animation-timing-function: cubic-bezier(.25, .1, .25, 1);`

+   `ease-in`: `animation-timing-function: cubic-bezier(.42, 0, 1, 1);`

+   `ease-out`: `animation-timing-function: cubic-bezier(0, 0, .58, 1);`

+   `ease-in-out`: `animation-timing-function: cubic-bezier(.42, 0, .58, 1);`

+   `linear`: `animation-timing-function: cubic-bezier(0, 0, 1, 1);`

我不确定你是否也是这样，但我更喜欢使用预定义的值。

现在，我们可以开始调整和测试每个值到小数点后，保存，并等待实时刷新完成其工作。但如果你问我，太多的测试时间是被浪费的。

惊人的 Lea Verou 创建了最好的用于处理贝塞尔曲线的 Web 应用程序：[www.cubic-bezier.com](http://www.cubic-bezier.com)。这是处理贝塞尔曲线最简单的方法。我强烈推荐这个工具。

之前展示的贝塞尔曲线图像是从[www.cubic-bezier.com](http://www.cubic-bezier.com)网站获取的。

**CSS**:

```js
.element {
  width: 300px;
  height: 300px;
  animation: fadingColors 2s infinite alternate 3s none running cubic-bezier(.42, 0, 1, 1);
}
```

# 杂项

以下 CSS 函数没有特定的类别，因此我们将它们分组在这里的杂项部分。

让我们看看我们有什么。

## drop-shadow()

`drop-shadow()` CSS 函数与`filter`属性一起使用，在元素下方添加阴影，其外观如下：

```js
drop-shadow(5px 5px 3px rgba(0, 0, 0, .5));
```

### 描述

`drop-shadow()`函数几乎与`box-shadow`属性完全相同，但有两大区别：`drop-shadow()`函数不支持`spread-radius`或`inset`值。

请参考`box-shadow`属性以获取所有值的详细描述。此外，一些浏览器在启用此功能时实际上会提供硬件加速，这最终会提高性能。你知道的；任何能提高性能的方法都是加分项。

**CSS**:

```js
.element {
  filter: drop-shadow(5px 5px 3px rgba(0, 0, 0, .5));
}
```

## element()

`element()` CSS 函数允许我们使用任何 HTML 元素作为另一个 HTML 元素的背景，其外观如下：

```js
background: element(#other-element);
```

### 描述

`element()`函数的使用案例很少，但无论如何，它对我们是可用的（当然，浏览器的支持还不是很好）。

**CSS**:

```js
.element {
  background: element(#other-element);
}
```

## image()

`image()` CSS 函数允许我们指定一个图像文件作为背景使用，其外观如下：

```js
image(../images/sprite.png);
```

### 描述

`image()`函数实际上与`url()`函数相同，但它被认为更加灵活，是声明背景图像而不是使用众所周知的`url()`函数的理想选择。然而，由于浏览器支持不足，`image()` CSS 函数有被从规范中删除的风险。

**CSS**:

```js
.element {
  background-image: image(../images/sprite.png);
}
```

## opacity()

`opacity()` CSS 函数与`filter`属性一起工作。它定义了元素的不透明度（透明度），其外观如下：

```js
filter: opacity(.2);
```

### 描述

当此功能应用于一个元素时，该元素及其子元素都会受到影响。此功能支持从`0`（零）到`1`的数值范围，这是默认值。`0`的值是完全透明的，就像`0%`不透明，而`1`是`100%`不透明，完全没有透明度。允许使用小数，但不允许使用负值。

**CSS**:

```js
.element {
  filter: opacity(.2);
}
```

## perspective()

`perspective()` CSS 函数与`transform` CSS 属性一起使用，其外观如下：

```js
perspective(300px);
```

### 描述

此值给元素提供了三维透视效果。相关的元素将在三维平面上反应。

此函数的工作方式与 `perspective` 属性类似，区别在于 `perspective()` 函数用于给单个元素添加透视效果。因此，它应用于**元素本身**。`perspective` 属性适用于一次性给多个元素添加透视效果，因此它应用于父元素。

例如，如果我们将 `perspective()` 函数应用于列表上的每个元素，每个元素都会有自己的消失点。但如果我们将 `perspective` 属性应用于该列表的父容器，所有元素将共享相同的消失点。

`perspective()` 函数本身并没有什么作用，因此为了看到它的实际效果，我们必须将其与其他转换函数（如 `rotate()`、`rotateX()` 或 `rotateY()`）结合使用。

它接受一个带有长度单位的**数值**。不允许使用负值。该值定义了从用户到 *Z* 轴的距离。

值越高，透视效果越不强烈。这是因为元素离我们越远。然而，值越低，透视效果看起来越明显。这是因为元素离我们越近。

**CSS**:

```js
.element {
  transform: perspective(300px) rotateY(45deg);
}
```

## rect()

`rect()` CSS 函数用于使用 `clip` 属性创建矩形形状的裁剪遮罩，其格式如下：

```js
clip: rect(0, 100px, 200px, 0);
```

### 小贴士

由于与 SVG 相关的功能和限制不佳，`clip` CSS 属性现在已被弃用。当前广泛支持的 `clip-path` 属性是 SVG 规范的一部分，并且已被 CSS 遮罩模块采用。

### 描述

此功能**仅**与 `clip` 属性一起使用，正如我提到的，此属性现在已被弃用。此外，此 CSS 函数**不**与更现代的 `clip-path` CSS 属性一起使用，因此建议使用 `inset()` CSS 函数代替。

请参考第六章中的 `inset()` CSS 函数，*CSS 属性 – 第三部分*。

# 规则

CSS **规则**以 `@` 字符开头，后跟一个关键字或标识符。它们必须以分号 (`;`) 字符结尾。

一些最流行的规则包括 `@font-face`，用于声明自定义字体；`@import` 用于导入外部 CSS 文件（不过出于性能考虑并不推荐使用），它也被一些 CSS 预处理器用来引入外部部分文件，这些文件最终会被编译成一个单一的 CSS 文件（推荐方法）；`@media` 用于在响应式项目中声明媒体查询或打印样式表等；`@keyframes` 用于创建动画等。

规则，让我们看看它们在哪里**应用**。

## @charset

`@charset()` 规则定义了样式表所使用的字符编码，其格式如下：

```js
@charset "UTF-8";
```

### 描述

只要 HTML 中已经定义了字符编码，我们就很少需要在样式表中定义字符编码。当浏览器检测到 HTML 中的字符编码时，它意味着 CSS 文件(们)使用相同的字符编码。

如果你喜欢在 CSS 文件中声明字符编码，那也行。如果你计划在样式表中使用它，它应该是文件顶部第一件事。它不能在`@`符号之前有空格字符，或者在其上方有空行。*字符编码名称*应始终在引号内，无论是单引号(`'`)还是双引号(`"`).

**CSS**:

```js
/*Correct character encoding directive*/
@charset "UTF-8";
/*Character encoding of the CSS is set to Latin-9 (Western European languages, with Euro sign)*/
@charset 'iso-8859-15';
/*This is invalid, there is a space before the @ symbol*/
 @charset "UTF-8";
/*This is invalid, character encoding name should be inside single [''] or double quotes [""]*/
@charset UTF-8;
```

## @document()

`@document()` 规则允许定义仅适用于网站特定页面的样式，其一种形式如下所示：

```js
@document url('http://website.com/page.html') { ... }
```

### 描述

有四个 CSS 函数是`@document()`规则独有的：`url()`、`url-prefix()`、`domain()`和`regexp("")`。可以在单个声明中定义多个函数。

函数内的值可以不使用引号声明，或者使用单引号(`'`)或双引号(`"`).只有`regexp("")`函数需要使用双引号(`"`).

+   `url()`: 这将样式限制在匹配 URL 的文档中

+   `url-prefix()`: 这将样式限制在以指定 URL 开始的文档中

+   `domain()`: 这将样式限制在文档的特定域名中

+   `regexp("")`: 这将样式限制在匹配正则表达式的文档中

**CSS**:

```js
/*url() function*/
@document url('http://website.com/page.html') { ... }
/*url-prefix() function*/
@document url-prefix("http://website.com/about/") { ... }
/*domain() function*/
@document domain(website.com) { ... }
/*regexp() function*/
@document regexp("https:.*")
/*Multiple functions in a single declaration*/
@document url('http://website.com/page.html') { ... },
  url-prefix("http://website.com/about/") { ... },
  domain(website.com) { ... },
  regexp("https:.*") { ... }
```

## @font-face

`@font-face()` 规则用于定义在文档中使用的自定义字体，其最简单的形式如下：

```js
@font-face {
  font-family: Franchise; 
  src: url("../fonts/franchise.woff") format("woff");
}
```

### 描述

`@font-face()` 规则实际上比许多人认为的还要存在得更久，因此我们的朋友 IE6 支持这个功能。使用 `@font-face()` 规则，我们可以针对网站/webapp 上使用的自定义字体文件进行定位，并将设计和品牌推广的可能性扩展到系统字体之外。

自定义字体有一个特性，即每个浏览器的不同版本支持一种格式，但不支持另一种，甚至有自己的专有字体格式。

Paul Irish 的文章《Bulletproof @font-face Syntax》，其中*笑脸*技术起源于此，是所有网页设计师和开发人员必读的`@font-face`文章([`tiny.cc/paul-irish-font-face`](http://tiny.cc/paul-irish-font-face))。

我们需要考虑的五种字体格式是：WOFF/WOFF2、EOT、TTF、OTF 和 SVG。

#### WOFF/WOFF2

**WOFF** 代表 **Web 开放字体格式**，由 Mozilla 创建。WOFF 格式是 OTF/TTF 字体格式的包装器，并且比任何其他格式都提供了更好的字体数据压缩，从而使得文件(们)更小。

**WOFF2** 实际上是增强版的 WOFF。它提供了更多的压缩，平均约 30%，在某些情况下甚至高达 50%。

所有现代浏览器都支持这两种格式。

#### EOT

**EOT**代表**嵌入式 OpenType**，由微软创建。只有旧版本的 IE（IE6 到 IE8）需要使用这种字体格式。其他浏览器不支持此格式，因此如果我们不需要支持旧版浏览器，我们就不需要在`@font-face()`规则声明中声明对此字体格式的链接。

#### OTF 和 TTF

**OTF**和**TTF**代表**OpenType 字体**和**TrueType 字体**。这些字体格式是跨平台兼容的，包括高级布局功能和专家排版控制的信息。OTF 是一种较新的格式，比 TTF 具有更多功能，例如小写字母、连字符、分数等。

#### SVG

**SVG**代表**可缩放矢量图形**。SVG *字体文件*实际上并没有字体；它有字体的矢量表示。此类字体文件用于需要支持旧版 iOS 设备的情况。然而，如果未声明此类字体，旧版 iOS 设备将简单地使用系统字体，如果问我，我完全没问题。

`@font-face`括号内的值被称为**字体描述符**。在其中，我们可以声明几个值：`font-family`、`src`、`font-variant`、`font-stretch`、`font-style`、`font-weight`和`unicode-range`。

#### font-family

这是一个必需的值。它定义了在样式表中使用的字体名称。

#### src

这是一个必需的值。它定义了字体文件的位置或 URL。可以在同一个`src`声明块中定义多个 URL，以适应每个浏览器支持的字体类型。然而，如果需要支持旧版 IE，当它发现同一个`src`声明块中有多个 URL 时，旧版 IE 会崩溃，因此如果需要支持旧版 IE，需要声明独立的`src`声明块。

除了针对带有 URL 的外部文件，我们还可以使用`local()`函数针对本地安装的文件进行定位。

#### font-variant

`font-variant` CSS 属性将目标文本转换为小写字母。在 CSS3 中，它被视为一个简写，并扩展了新的值，但开发者很少使用。有关更多信息，请参阅第五章，*CSS 属性 - 第二部分*。

#### font-stretch

`font-stretch` CSS 属性允许我们从相关的字体族中选择一个*紧凑型*、*正常型*或*扩展型*面。有关更多信息，请参阅第五章，*CSS 属性 - 第二部分*。

#### font-weight

`font-weight` CSS 属性定义了字体的粗细（重量）。有关更多信息，请参阅第五章，*CSS 属性 - 第二部分*。

#### unicode-range

`unicode-range` CSS 属性描述符定义了应从 `@font-face` 声明中下载的特定字符或符号的范围。这在处理涉及不同语言的网站时非常有用。通过声明 `unicode-range`，浏览器只为该页下载该语言的特定字符，从而节省带宽并优化性能。

这个属性很少使用。

### Google 字体

我们在谈论 `@font-face` 时不能不提到 **Google Fonts**。Google Fonts 是一个免费的网页字体服务，它通过给我们一个指向所选字体（或字体组）的 HTML `<link>` 标签，使我们能够跳过在 CSS 文件中创建 `@font-face` 声明块的繁琐工作。

查看 Google Fonts，链接为 [`tiny.cc/google-fonts`](http://tiny.cc/google-fonts)。

**HTML**:

```js
<!-- Google Fonts link snippet -->
<link href='https://fonts.googleapis.com/css?family=Roboto' rel='stylesheet' type='text/css'>
```

**CSS**:

```js
/*Full descriptive @font-face declaration*/
@font-face {
  font-family: Roboto;
    /*IE9 Compat Modes*/
  src: url("../fonts/roboto.eot");
    /*Locally installed font file*/
  src: local("roboto"),
    /*IE6-IE8*/
    url("../fonts/roboto.eot?#iefix") format("embedded-opentype"),
    /*Modern Browsers*/
    url("../fonts/roboto.woff2") format("woff2"),
    url("../fonts/roboto.woff") format("woff"),
    /*Safari, Android, iOS*/
    url("../fonts/roboto.ttf") format("truetype"),
    /*Old iOS devices*/
    url("../fonts/roboto.svg#roboto") format("svg");
  /*Unicode range*/
  unicode-range: U+0400-045F, U+0490-0491, U+04B0-04B1, U+2116;
  /*Font properties*/
  font-weight: normal;
  font-style: normal;
}

/*Recommended @font-face syntax*/
@font-face {
  font-family: Roboto;
  src: url("../fonts/roboto.woff2") format("woff2"),
       url("../fonts/roboto.woff") format("woff");
  font-weight: normal;
  font-style: normal;
}

/*Usage*/
.element {
  font-family: Roboto, Arial, Helvetica, san-serif;
}
```

## @import

`@import()` 规则用于将样式表导入另一个样式表中，其语法如下：

```js
@import "other-style-sheet.css";
```

### 描述

之前的例子使用一个 `string` 值来指向另一个样式表。但样式表也可以通过 `url()` 函数导入。

`@import` 规则应该始终位于除 `@charset` 规则之外的所有其他规则之前，否则浏览器将忽略它。

需要考虑的一点是层叠。导入的样式表按导入的顺序层叠。还可以通过媒体查询指定特定导入样式表针对的媒体。如果声明了多个媒体查询，它们需要用逗号分隔。

### 提示

众所周知，使用 `@import` 由于按顺序下载而不是并行下载以及多次 HTTP 请求，会对性能产生负面影响。关于此问题的更多信息，请参阅 Steve Souders 的文章 *不要使用 @import*，链接为 [`tiny.cc/steve-souders-avoidimport`](http://tiny.cc/steve-souders-avoidimport)。

**CSS**:

```js
/*Import a style sheet in the same directory*/
@import "other-style-sheet.css";
/*Import a style sheet from a relative path*/
@import url(../other-stylesheets/other-style-sheet.css);
/*Import a style sheet with an absolute path*/
@import url(http://website.com/other-stylesheets/other-style-sheet.css);
/*Define a print style sheet with the 'print' media query*/
@import "print.css" print;
/*Declare multiple media queries*/
@import "screen.css" screen, projection;
/*Use more a commonly known media query*/
@import url("portrait.css") screen and (orientation: portrait);
```

## @keyframes

`@keyframes()` 规则用于列出要动画化的 CSS 属性，其语法如下：

```js
@keyframes animationName {
  from {
    /*Animation properties START here*/
  }
  to {
    /*Animation properties END here*/
  }
}
```

### 描述

使用 `@keyframes` 规则创建的动画只运行一个周期。如果我们想让动画反复播放，实现渐入渐出，或者展示其他行为，我们需要在元素本身中声明这些属性，而不是在 `@keyframes` 规则之外。

请参考第四章（ch04.html "第四章. CSS 属性 – 第一部分"）中的动画部分，以获取有关这些 CSS 属性的详细解释。

动画名称（也称为 **标识符**）始终位于 `@keyframes` 关键字之后，两者之间用空格分隔。这个动画名称将在稍后通过 `animation-name` 或 `animation` 简写 CSS 属性进行引用。

动画的开始和结束可以使用两个选择器关键字 `from` 和 `to`，或者使用两个关键帧选择器 `0%` 和 `100%` 来声明。不允许使用负值。

显然，我们可以声明中间路径点，但我们只能使用 *关键帧选择器* 来做到这一点。在 `@keyframes` 规则中声明的属性数量没有限制。然而，一些浏览器会动画化规范中说不应该动画化的属性，而其他浏览器则正确地遵循规范。当然，规范对一些定义不够明确，所以请确保运行适当的测试。

现在使用 `@keyframes` 规则和 `transition` 属性之间的区别在于，使用 `@keyframes` 规则，我们可以定义中间路径点发生的事情，而不是让浏览器为我们找出它。

基本上，如果我们有一个简单的动画，我们可以直接使用 `transition` 属性。如果我们有更复杂和精细的动画，则使用 `@keyframes`。

**CSS**:

```js
@keyframes diamond {
  from {
    top: 0;
    left: 0;
  }
  25% {
    top: 200px;
    left: -200px;
  }
  50% {
    top: 400px;
    left: 0;
  }
  75% {
    top: 200px;
    left: 200px;
  }
  to {
    top: 0;
    left: 0;
  }
}

.element {
  position: relative;
  animation: diamond 3s infinite ease-in-out;
}
```

## @media

`@media()` 规则允许我们定义一组应用于特定媒体类型的 CSS 样式，其外观如下：

```js
@media (min-width: 40em) { ... }
```

您可以使用前面的代码或以下代码：

```js
@media print { ... }
```

### 描述

这段 CSS 允许我们为特定的媒体类型声明任何一组样式。

在 `@media()` 规则之后出现的两种指令类型，例如 `print` 和 `screen`，甚至是 `(min-width: 40em)` 或 `(max-width: 40em)`，它们被称为 **媒体查询**。

关键字（`print` 和 `screen`）被称为 **媒体类型**。而那些测试特定 **用户代理**（**UA**）或显示功能的被称为 **媒体特性**。

#### 媒体类型

媒体类型区分大小写。我们可以使用 `@media()` 规则与以下 10 种媒体类型一起使用：

+   `all`: 这是为了在所有设备上工作

+   `braille`: 这是为了与盲文触觉反馈设备一起工作

+   `embossed`: 这是为了在盲文打印机上工作

+   `handheld`: 这是为了与手持“移动”设备一起工作

+   `print`: 这是为了打印文档

+   `projection`: 这是为了与投影仪一起工作

+   `screen`: 这是为了与所有尺寸的计算机屏幕一起工作

+   `speech`: 这是为了与语音合成器一起工作

+   `tty`: 这是为了与电传打字机一起工作

+   `tv`: 这是为了与电视一起工作

**CSS**:

```js
/*Viewport width media query*/
@media (min-width: 40em) {
  header { background: red; }
}
/*Print media query*/
@media print {
  *, *:before, *:after { background-image: none; }
}
```

## @namespace

`@namespace()` 规则用于在样式表中定义 XML 命名空间，其外观如下：

```js
@namespace url(http://www.w3.org/1999/xhtml);
```

您可以使用前面的代码或以下代码：

```js
@namespace svg url(http://www.w3.org/2000/svg);
```

### 描述

当我们需要在 CSS 文档中使用应用于或针对特定命名空间中元素的选择器时，我们使用 `@namespace()` 规则。例如，我们可以在我们的 HTML 文档中嵌入 SVG 文件。问题是 SVG 与 HTML 和 XML 共享一些常见元素，例如 `<a>` 元素。因此，我们不必创建单独的样式表来针对 SVG 元素，我们可以使用 `@namespace()` 规则声明 SVG 命名空间，以针对同一 HTML 文档中的 `<a>` 元素，这样我们只需要在一个样式表中工作，而不是两个（或更多）。

**现在，`@namespace()` 规则主要用于需要声明 `<html>` 元素中的命名空间的旧版 XHTML 文档：**

```js
<html  xml:lang="en" lang="en">
```

**使用 `xmlns` 指令后，我们现在可以在 CSS 中声明命名空间。最后，我们可以针对 SVG 块中的 `<a>` 元素进行定位，而不会影响 HTML `<a>` 元素。**

**URLs 仅用于使标记更易于阅读和理解，当有人阅读时。**

****XHTML**:**

```js
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html ** xml:lang="en" lang="en">**
```

******CSS**:****

```js
**@namespace "http://www.w3.org/1999/xhtml";
@namespace svg "http://www.w3.org/2000/svg";

svg|a {
  background: orange;
}**
```

## ****@page****

****`@page()` 规则用于修改页面的某些属性以准备打印，其语法如下：****

```js
**@page {
  margin: 2cm;
}**
```

### ****描述****

****当使用 `@page()` 规则时，只有页面的少数属性可以更改：`margins`、`widows`、`orphans` 和 *页面分页*。声明任何其他类型的属性都将被忽略。****

****我们还可以声明是否只想针对 *第一页*、*所有左页* 或 *所有右页* 使用 `:first`、`:left` 和 `:right` 伪类。`@page()` 规则最常用于更改边距。****

******CSS**:****

```js
**/*Affects all pages*/
@page {
  margin: 2cm;
}
/*Affects all left pages only*/
@page :left {
  margin: .8in;
  orphans: 3;
}
/*Affects all right pages only*/
@page :right {
  margin: 15mm;
}**
```

## ****@supports****

****`@supports()` 规则用于检测浏览器上的 *功能*，其语法如下：****

```js
**@supports (display: flex) { ... }**
```

### ****描述****

*****功能检测* 通常使用类似 **Modernizr** 的 polyfills 来实现。通过 `@supports()` 规则，我们可以仅使用 CSS 就达到类似的效果。****

****为了使此函数正常工作，我们需要指定一个 *属性* 和一个 *值*。我们可以与 `@supports()` 规则一起使用三个关键字操作符：`not`、`and` 和 `or`。****

#### ****not 操作符****

****就像我们可以检查浏览器支持的功能一样，我们也可以使用 `not` 操作符检查浏览器 *不支持* 的功能。****

#### ****and 操作符****

****`and` 操作符允许我们在同一个声明中检查多个 CSS 属性。****

#### ****or 操作符****

****此操作符的一个好例子是当我们需要检查带有供应商前缀的 CSS 属性以支持旧版浏览器时。当使用此操作符时，如果其中一个表达式为 `true`，则所有其他表达式也将有效。此外，我们还可以在必要时组合操作符。****

******CSS**:****

```js
**/*Detect a feature that is supported*/
@supports (transform: translateX(-50%)) {
  .element {
    transform: translateY(-50%)
  }
}
/*Detect a feature that is NOT supported*/
@supports not (transform: translateX(-50%)) {
  .element {
    margin-left: 120px;
  }
}
/*Detect multiple features*/
@supports (transform: translateX(-50%)) and (display: flex) {
  .element {
    transform: translateY(-50%);
    justify-content: space-between;
  }
}
/*Detect at least one feature*/
@supports (-webkit-box-pack: justify) or (-webkit-justify-content: space-between) or (-ms-flex-pack: justify) or (justify-content: space-between) {
  .element {
    justify-content: space-between;
  }
}
/*Combining conditions*/
@supports (transform: translateX(-50%)) and ( (-webkit-justify-content: space-between) or (justify-content: space-between) ) {
  transform: translateY(-50%);
  justify-content: space-between;
}**
```

****# 全局 CSS 关键字值

**以下关键字值对网页设计师和开发者来说是普遍存在的，但你是否曾想过它们的确切含义和作用？**

## auto

**auto** CSS 关键字值指示浏览器自动计算 CSS 属性的值，其语法如下：

```js
margin: auto;
```

**auto** 这个术语是 **自动** 的简称。它与说 100% 不同，因为 100% 是一个实际定义的值；`auto` 是由浏览器计算的。

**`auto` 关键字最常见的应用位置之一是使用 `margin` CSS 属性水平居中元素。**

**CSS**:

```js
.element {
  margin: auto;
}
```

### 小贴士

我看到大多数人使用 `margin: 0 auto;` 来居中元素。这没问题，但值零（`0`）可以省略。`margin: auto;` 就足够了，并且会产生相同的结果。

## 继承

`inherit` CSS 关键字值使元素从其父容器继承值。

**CSS**:

```js
/*All <h1>'s are red*/
h1 { color: red; }
/*All text in .element is blue*/
.element { color: blue; }
/*Make all <h1>'s inside .element blue*/
.element h1 { color: inherit; }
```

## 默认

`initial` CSS 关键字值将 CSS 属性设置为规范中定义的默认值。

**CSS**:

```js
/*All <h1>'s are red*/
h1 { color: red; }
/*Set the color of all <h1>'s inside .element to the default (black)*/
.element h1 { color: initial; }
```

## 无

`none` CSS 关键字值定义了没有特定的样式。

**CSS**:

```js
.element {
  border: none;
}
```

## 标准

`normal` CSS 关键字值定义了一个*标准*值。

**CSS**:

```js
.element {
  font-weight: normal;
}
```

## 未设置

`unset` CSS 关键字是 `inherit` 和 `initial` 关键字的组合，看起来是这样的：

```js
color: unset;
```

通过组合 `inherit` 和 `initial` 关键字，`unset` CSS 关键字值重置了属性的值。

如果一个元素正在从其父容器继承值，并且声明了 `unset` 关键字，那么属性的值将重置为父容器的值（因为它正在继承）。但如果一个元素没有父容器，并且声明了 `unset` 关键字，那么其属性的值将重置为规范中定义的默认值（因为它没有继承）。

**CSS**:

```js
/*All <h1>'s are set to the default color per the spec*/
h1 { color: unset; }
/*All <h1>'s are set to the default color of the parent*/
.element { color: green; }
.element h1 { color: unset; }
```

## 回退

`revert` CSS 关键字值就像 CSS 中的 *撤销*，它会将层叠回滚到之前的状态，并将属性重置为用户代理定义的默认值。它看起来是这样的：

```js
display: revert;
```

这与 `initial` CSS 关键字不同，因为 `revert` 会回滚层叠并重置属性值为用户代理的 *样式表* 值。而 `initial` 则将值重置为其规范中定义的默认值。

例如，规范说明 `display` 的默认值是 `inline`。然而，大多数用户代理将 `<div>` 的默认值设置为 `display: block;`，或者将 `<table>` 的默认值设置为 `display: table;`。

**CSS**:

```js
/*Default value per the spec*/
display: inline;
/*Default value per the UA style sheet*/
div { display: block; }
/*Style defined by the developer/designer*/
.element { display: inline-block; }
/*Style sheet is rolled back and DIV behaves as display: block;*/
div.element { display: revert; }
```

# 摘要

这就结束了关于 CSS 的章节，很有趣吧？

我们学习了 CSS 过滤器以及我们如何在不依赖图像编辑工具的情况下修改元素的色彩。这也适用于 CSS 变换，因为我们可以使用 CSS 很容易地修改元素的形状和方向，至少在一定程度上。

同时，我们还了解了在 CSS 中创建颜色的不同方法，以及 HSL 模式比其他任何颜色模式都更直观和灵活。

使用 `attr()` 或 `calc()` 函数计算和声明不同的值，为我们的 CSS 工具箱打开了新的可能性，例如，如何制作响应式表格。

我们现在知道，为了提高阴影效果的性能，我们可以使用 `drop-shadow()` 函数；或者要修改元素的透明度，我们可以使用 `opacity()` 函数；或者使用 `perspective()` 函数来修改元素的透视。

现在，@规则更有意义了。此外，我们还讨论了不同的字体格式，并了解到如果我们不需要支持旧版 IE，我们就可以直接使用 WOFF 和 WOFF2。

最后，我们终于弄清楚了所有常用的全局 CSS 关键字值，比如 auto 或 inherit，我们经常使用它们，却从未真正质疑过它们是什么以及它们是如何工作的。

注意，你不需要知道和记住所有的 CSS 函数，你需要知道在哪里查找——这本书。****
