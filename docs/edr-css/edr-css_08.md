# 第八章：理智样式表的十诫

1.  你应该有一个所有关键选择器的单一真相来源

1.  除非你正在嵌套媒体查询或覆盖，否则不应该嵌套

1.  即使你认为你必须使用 ID 选择器，也不应该使用 ID 选择器

1.  在编写样式表中不应该写厂商前缀

1.  你应该使用变量来设置大小、颜色和 z-index

1.  你应该总是首先编写移动规则（避免使用 max-width）

1.  节制使用混合，并避免 @extend

1.  你应该注释所有魔术数字和浏览器黑客

1.  不应该内联图片

1.  当简单的 CSS 能够正常工作时，你不应该编写复杂的 CSS

遵循这些规则的人是有福的，因为他们将继承理智的样式表。

阿门。

# 为什么是十诫？

以下是一套高度主观的规则，是为了在开发团队中编写可预测的样式表而产生的。每个规则都可以通过工具强制执行。当一个项目中只有一个 CSS 开发人员时，花时间开发或集成工具可能看起来是多余的。然而，在超过两个活跃的开发人员之后，工具将一次又一次地赚取它的时间投资。我们将在下一章中处理工具来“监督”规则。现在，让我们考虑语法和规则本身。

## 工具

为了实现更易维护的样式表，我们可以依赖于 PostCSS，这是一个允许使用 JavaScript 操作 CSS 的 CSS 工具。有兴趣的人可以在这里查看更多信息：[`github.com/postcss/postcss`](https://github.com/postcss/postcss)

PostCSS 促进了扩展 CSS 语法的使用。为了编写，所使用的语法大量借鉴了 Sass ([`sass-lang.com/`](http://sass-lang.com/))。这提供了使我们的编写样式表更易于维护的功能。

使用 PostCSS，我们能够利用：

+   变量

+   混合（如宏，用于某些设置，如字体系列）

+   使用和符号（`&`）引用**关键选择器**

实际上，PostCSS 可以实现类似于 CSS 预处理器（如 Sass、LESS 或 Stylus）的功能。

它的不同之处在于它的模块化和可扩展性。与前面提到的预处理器需要“吞下整颗药丸”不同，使用 PostCSS 允许我们更加选择我们使用的功能集。它还允许我们轻松地根据需要扩展我们的功能集，可以使用任意数量的“现成的”插件，或者使用 JavaScript 编写我们自己的插件。

例如，Sass 允许编写循环，我们选择阻止该功能。例如，如果需要循环来解决特定问题（例如，100 种不同颜色的标题变体），我们仍然可以通过使用 JavaScript 编写的 PostCSS 插件来实现。

此外，由于 PostCSS 生态系统，我们可以对编写的样式进行静态分析和 linting；当编写不良代码时，会导致构建失败和代码提交。

### 注意

如果术语“linting”对你来说是陌生的，那么它是静态分析的另一个术语。它查看编写的代码，并根据任意数量的预定义规则提出建议。例如，如果你使用浮动，或者没有在需要的地方放置空格或分号，它可能会发出警告。一般来说，你可以使用 linters 来强制执行任何你喜欢的编码约定，而且在独自工作时非常有用，但在团队中工作时可能是无价的：在那里，许多（粗心的）人可能会触及代码。

## 原理

当我们编写 ECSS 时，我们希望避免产生 CSS，这些 CSS 遭受过于具体、充斥着不需要的前缀、缺乏注释和充满“魔法”数字的问题。

以下 10 条规则阐明了被认为是实现此目标最重要的规则。

## Throughout 使用的定义：

+   **覆盖**：根据继承有意修改关键选择器的值的情况

+   **关键选择器**：任何 CSS 规则中最右边的选择器

+   **前缀**：供应商特定前缀，例如`-webkit-transform:`

+   **编写样式表**：我们在其中编写样式规则的文件

+   **CSS**：工具生成的结果 CSS 文件，最终由浏览器消耗

现在让我们考虑每个规则及其旨在解决的问题。

# 1. 所有关键选择器都应该有一个单一的真相来源

在编写样式表中，关键选择器应该只写一次。

这使我们能够在代码库中搜索关键选择器并找到我们选择器的*单一真相来源*。由于使用了扩展的 CSS 语法，发生在关键选择器上的一切都可以封装在一个规则块中。

通过嵌套和使用*父*选择器引用关键选择器来处理对关键选择器的覆盖。稍后会详细讨论。

考虑这个例子：

```css
.key-Selector {
    width: 100%;
    @media (min-width: $M) {
        width: 50%;
    }
    .an-Override_Selector & {
        color: $color-grey-33;
    }
}

```

这将在 CSS 中产生以下结果：

```css
.key-Selector {
  width: 100%;
}

@media (min-width: 768px) {
  .key-Selector {
    width: 50%;
  }
}

.an-Override_Selector .key-Selector {
  color: #333;
}

```

在编写样式表中，关键选择器（`.key-Selector`）在根级别永远不会重复。因此，从维护的角度来看，我们只需要在代码库中搜索`.key-Selector`，就能找到关于该关键选择器的一切内容，这是一个单一的真相来源。

+   如果我们需要在不同的视口大小下以不同的方式显示，会发生什么？

+   当它存在于 containerX 中时会发生什么？

+   当通过 JavaScript 向其添加这个或那个类时会发生什么？

在所有这些情况下，关键选择器的可能性都嵌套在同一个规则块中。这意味着任何可能的特异性问题都完全隔离在一个大括号集合内。

让我们接下来更详细地看一下覆盖。

## 处理覆盖

在先前的例子中，演示了如何处理对关键选择器的覆盖。我们将覆盖选择器嵌套在关键选择器的规则块内，并使用`&`符号引用父级。`&`符号在 Sass 语言中是父选择器。你可以把它想象成 JavaScript 中的`this`。

### 提示

要使用父选择器测试规则，我建议使用[`sassmeister.com`](http://sassmeister.com)

### 标准覆盖

考虑这个例子：

```css
.ip-Carousel {
    font-size: $text13;
    /* The override is here for when this key-selector sits within a ip-HomeCallouts element */
    .ip-HomeCallouts & {
        font-size: $text15;
    }
}
```

这将产生以下 CSS：

```css
.ip-Carousel {
  font-size: 13px;
}

.ip-HomeCallouts .ip-Carousel {
  font-size: 15px;
}

```

这会导致`ip-Carousel`在具有`ip-HomeCallouts`类的元素内部时`font-size`增加。

### 在同一元素上使用额外的类进行覆盖

让我们考虑另一个例子，如果我们需要在此元素获得额外类时提供覆盖？我们应该这样做：

```css
.ip-Carousel {
    background-color: $color-green;
    &.ip-ClassificationHeader {
        background-color: $color-grey-a7;
    }
}
```

这将产生以下 CSS：

```css
.ip-Carousel {
  background-color: #14805e;
}

.ip-Carousel.ip-ClassificationHeader {
  background-color: #a7a7a7;
}

```

同样，覆盖包含在关键选择器的规则块内。

### 在另一个类中覆盖并且还有额外的类

最后让我们考虑一种情况，我们需要为另一个元素内部的关键选择器提供覆盖，该元素还有额外的类：

```css
.ip-Carousel {
    background-color: $color-green;
    .home-Container &.ip-ClassificationHeader {
        background-color: $color-grey-a7;
    }
}

```

这将产生以下 CSS：

```css
.ip-Carousel {
  background-color: #14805e;
}

.home-Container .ip-Carousel.ip-ClassificationHeader {
  background-color: #a7a7a7;
}

```

在这里，我们使用父选择器来引用我们的关键选择器，介于上面的覆盖（`.home-Container`）和另一个类（`.ip-ClassificationHeader`）之间。

### 使用媒体查询进行覆盖

最后，让我们考虑使用媒体查询进行覆盖。考虑这个例子：

```css
.key-Selector {
    width: 100%;
    @media (min-width: $M) {
        width: 50%;
    }
}
```

这将产生以下 CSS：

```css
.key-Selector {
  width: 100%;
}

@media (min-width: 768px) {
  .key-Selector {
    width: 50%;
  }
}
```

再次，所有可能性都包含在同一个规则内。注意媒体查询宽度的变量使用？我们很快会讨论到这一点。

任何和所有媒体查询都应以相同的方式包含。以下是一个更复杂的例子：

```css
.key-Selector {
    width: 100%;
    @media (min-width: $M) and (max-width: $XM) and (orientation: portrait) {
        width: 50%;
    }
    @media (min-width: $L) {
        width: 75%;
    }
}
```

这将产生以下 CSS：

```css
.key-Selector {
  width: 100%;
}

@media (min-width: 768px) and (max-width: 950px) and (orientation: portrait) {
  .key-Selector {
    width: 50%;
  }
}

@media (min-width: 1200px) {
  .key-Selector {
    width: 75%;
  }
}
```

通过刚刚查看的所有覆盖的嵌套，你可能会认为嵌套子元素也是有道理的？你错了。非常错误。这将是一件非常非常糟糕的事情。接下来我们将看看为什么。

# 2. 除非你是嵌套媒体查询或覆盖，否则不得嵌套

CSS 中的关键选择器是任何规则中最右边的选择器。它是应用封闭属性/值的选择器。

我们希望我们的 CSS 规则尽可能*扁平*。我们**不希望**在关键选择器（或任何 DOM 元素）之前有其他选择器，除非我们绝对需要它们来覆盖默认的关键选择器样式。

原因是添加额外的选择器并使用元素类型（例如`h1.yes-This_Selector`）：

+   创建额外不需要的特异性

+   使维护变得更加困难，因为随后的覆盖需要更加具体

+   向结果 CSS 文件添加不必要的膨胀

+   在元素类型的情况下，将规则绑定到特定元素和/或标记结构

例如，假设我们有这样的 CSS 规则：

![2. 除非你在嵌套媒体查询或覆盖，否则不要嵌套](img/Warning-image-1.jpg)

```css
 #notMe .or-me [data-thing="nope"] .yes-This_Selector {
 width: 100%; 
 }

```

在上面的例子中，`yes-This_Selector`是关键选择器。如果这些属性/值应该在所有情况下添加到关键选择器中，我们应该制定一个更简单的规则。

简化前面的例子，如果我们只想针对关键选择器进行目标定位，我们会希望有这样的规则：

```css
.yes-This_Selector {
    width: 100%;
}
```

## 不要在规则中嵌套子元素

假设我们有这样的情况，我们在包装元素内部有一个视频播放按钮。考虑这个标记：

```css
<div class="med-Video">
    <div class="med-Video_Play">Play</div>
</div>
```

让我们为包装器设置一些基本样式：

```css
.med-Video {
    position: absolute;
    background-color: $color-black;
}
```

现在我们想要在包装元素内部定位播放元素。你可能会想这样做：

![不要在规则中嵌套子元素](img/Warning-image-1.jpg)

```css
 .med-Video {
 position: absolute; 
 background-color: $color-black; 
 /* Center the play button */
 .med-Video_Play { 
 position: absolute; 
 top: 50%; 
 left: 50%;
 transform: translate(-50%, -50%); 
 } 
 }

```

这将产生以下 CSS（为简洁起见删除了供应商前缀）：

![不要在规则中嵌套子元素](img/Warning-image-1.jpg)

```css
 .med-Video { 
 position: absolute; 
 background-color: #000;
 /* Center the play button */ 
 } 
 .med-Video .med-Video_Play { 
 position: absolute; 
 top: 50%; 
 left: 50%; 
 transform: translate(-50%, -50%); 
 }

```

你看到问题了吗？当完全不需要时，我们为`.med-Video_Play`元素引入了额外的特异性。

这是一个微妙的例子。然而，重要的是要意识到这一点，并避免这样做，以免最终得到这样的规则：

![不要在规则中嵌套子元素](img/Warning-image-1.jpg)

```css
 .MarketGrid > .PhoneOnlyContainer > .ClickToCallHeader >
  .ClickToCallHeaderMessage > .MessageHolder > span { 
 font-weight: bold; 
 padding-right: 5px; 
 }

```

相反，记住*每个关键选择器都有自己的规则块*。覆盖是嵌套的，子元素不是。这是正确重写的示例：

```css
.med-Video {
    position: absolute;
    background-color: $color-black;
}

/* Center the play button */
.med-Video_Play {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
}
```

这将产生以下 CSS：

```css
.med-Video {
  position: absolute;
  background-color: #000;
}

/* Center the play button */
.med-Video_Play {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

```

每个关键选择器只有在需要时才会具体化，而不会更多。

# 3. 即使你认为你必须使用 ID 选择器，也不要使用

在复杂 UI 中 ID 的限制已经有了详细的记录。简而言之，它们比类选择器更具体 - 因此使覆盖更加困难。此外，它们在页面中只能使用一次，因此它们的有效性有限。

### 提示

记住我们在第二章中详细讨论了特异性，*CSS 在规模上的问题*。

使用 ECSS 时，我们不在 CSS 中使用 ID 选择器。它们与基于类的选择器相比没有任何优势，并引入了不需要的问题。

在几乎难以置信的情况下，如果您必须使用 ID 来选择元素，请在属性选择器中使用它以保持特异性较低：

```css
[id="Thing"] {
    /* Property/Values Here */
}
```

# 4. 不要在编写样式表中写供应商前缀

多亏了 PostCSS，我们现在有了工具，这意味着在编写样式表中不需要为 W3C 指定的属性/值编写供应商前缀。这些前缀由*Autoprefixer*（[`github.com/postcss/autoprefixer`](https://github.com/postcss/autoprefixer)）工具自动处理，可以配置为为所需的平台/浏览器支持级别提供供应商前缀。

例如，不要这样做：

![4. 不要在编写样式表中写供应商前缀](img/Warning-image-1.jpg)

```css
 .ip-Header_Count { 
 position: absolute; 
 right: $size-full;
 top: 50%; 
 -webkit-transform: translateY(-50%); 
 -ms-transform: translateY(-50%); 
 transform: translateY(-50%); 
 }

```

相反，你应该只写这个：

```css
.ip-Header_Count {
    position: absolute;
    right: $size-full;
    top: 50%;
    transform: translateY(-50%);
}
```

这不仅使编写样式表更易于阅读和处理，而且意味着当我们想要改变我们的支持级别时，我们可以对构建工具进行单一更改，添加的供应商前缀将自动更新。

唯一的例外是可能仍然需要非 W3C 属性/值的情况。例如，在 WebKit 设备中的触摸惯性滚动面板，仍然需要在作者样式中添加某些供应商前缀的属性，因为它们不是 W3C。例如：

```css
.ui-ScrollPanel {
    -webkit-overflow-scrolling: touch;
}
```

或者在移除 WebKit 的滚动条时：

```css
.ui-Component {
    &::-webkit-scrollbar {
      -webkit-appearance: none;
    }
}
```

# 5. 使用变量进行尺寸、颜色和 z-index 设置

对于任何规模的项目，设置尺寸、颜色和 z-index 的变量是必不可少的。

UI 通常基于某种网格或尺寸比例。因此，尺寸应该基于固定尺寸，并对这些尺寸进行合理的划分。例如，这里是基于`11px`的尺寸和变量的变体：

```css
$size-full: 11px;
$size-half: 5.5px;
$size-quarter: 2.75px;
$size-double: 22px;
$size-treble: 33px;
$size-quadruple: 44px;
```

对于开发人员来说，使用变量还提供了额外的经济效益。例如，它可以节省从复合中选择颜色值。它还有助于规范化设计。

例如，如果一个项目只使用 13px、15px 和 22px 的字体大小，并且有一个变化要求 14px 的字体大小，那么变量提供了一些标准化的参考。在这种情况下，如果字体是 13px 或 15px，因为 14px 在其他地方都没有使用？这使开发人员可以向设计人员反馈可能存在的设计不一致之处。

颜色值也是如此。例如，假设我们有一个十六进制`#333`的变量。我们可以这样写一个变量：

```css
$color-grey-33: #333333;
```

表面上看，当十六进制值更短时，写变量名似乎是荒谬的。然而，再次强调，使用变量可以防止不需要的变体渗入代码库（例如`#323232`），并有助于识别代码中的*红旗*。

在对颜色进行修改时，仍然使用变量是很重要的。使用颜色函数对变量进行操作以实现你的目标。例如，假设我们想要一个半透明的`#333`颜色。

应该在作者样式表中实现这样：

```css
.ip-Header {
    background-color: color($color-grey-33 a(.5));
}
```

PostCSS 可以为 W3C 颜色函数提供一个 polyfill：[`drafts.csswg.org/css-color/#modifying-colors`](https://drafts.csswg.org/css-color/#modifying-colors)，上面的示例产生了以下 CSS：

```css
.ip-Header {
    background-color: rgba(51, 51, 51, 0.5);
}
```

在这个例子中，我们使用了 alpha CSS 颜色函数。我们使用`color()`函数，传入我们想要操作的颜色，然后进行操作（在这种情况下是 alpha）。

最初使用变量可能看起来更复杂，但这样可以让未来的作者更容易理解正在操作的颜色是什么。

### 提示

我还鼓励你看看*CSS Color Guard*（[`github.com/SlexAxton/css-colorguard`](https://github.com/SlexAxton/css-colorguard)），这是一个用于警告代码库中颜色在视觉上难以区分的工具。

同样重要的是使用变量来设置 z-index。这在堆叠上下文方面是很重要的。不应该需要`z-index: 999`或类似的东西。而是使用几个默认值（设置为变量）中的一个。这里有一些与 z-index 相关的变量：

```css
$zi-highest: 50;
$zi-high: 40;
$zi-medium: 30;
$zi-low: 20;
$zi-lowest: 10;
$zi-ground: 0;
$zi-below-ground: -1;

```

# 6. 总是首先编写移动端规则（避免使用 max-width）

对于任何响应式工作，我们希望在样式中采用移动优先的思维方式。因此，规则的根部应该是适用于最小视口（例如移动设备）的属性和值。然后我们使用媒体查询来覆盖或添加这些样式，根据需要。

考虑这个：

```css
.med-Video {
    position: relative;
    background-color: $color-black;
    font-size: $text13;
    line-height: $text15;
    /* At medium sizes we want to bump the text up */
    @media (min-width: $M) {
        font-size: $text15;
        line-height: $text18;
    }
    /* Text and line height changes again at larger
    viewports */
    @media (min-width: $L) {
        font-size: $text18;
        line-height: 1;
    }
}
```

这将产生以下 CSS：

```css
.med-Video {
  position: relative;
  background-color: #000;
  font-size: 13px;
  line-height: 15px;
}

@media (min-width: 768px) {
  .med-Video {
    font-size: 15px;
    line-height: 18px;
  }
}

@media (min-width: 1200px) {
  .med-Video {
    font-size: 18px;
    line-height: 1;
  }
}
```

我们只需要在不同的视口更改`font-size`和`line-height`，所以这就是我们要修改的全部内容。通过在媒体查询中使用`min-width`（而不是`max-width`），如果在更大的视口尺寸上`font-size`和`line-height`需要保持不变，我们就不需要额外的媒体查询。只有当视口尺寸变化时，我们才需要媒体查询。因此，不建议将`max-width`作为媒体查询的单个参数。

底线：使用`min-width`而不是`max-width`编写媒体查询。这里唯一的例外是如果您想将一些样式隔离到中等范围。例如：在中等和大型视口之间。

```css
.med-Video {
    position: relative;
    background-color: $color-black;
    font-size: $text13;
    line-height: $text15;
    /* Between medium and large sizes we want to bump the
    text up */
    @media (min-width: $M) and (max-width: $L) {
        font-size: $text15;
        line-height: $text18;
    }
}
```

# 7. 谨慎使用 mixins（避免@extend）

避免将代码抽象为 mixins 的诱惑。有一些领域是 mixins 非常适合的。CSS 文本截断的代码（例如`@mixin Truncate`）或 iOS 风格的惯性滚动面板，其中有许多伪选择器需要针对不同的浏览器进行正确设置。另一个很好的用例可能是复杂的字体堆栈。

### 提示

字体堆栈很难设置，而且很烦人。我发现处理字体的最理智的方法是让`body`使用最常见的字体堆栈，然后只在需要时用不同的字体堆栈覆盖它。

例如：

```css
.med-Video {
    position: relative;
    background-color: $color-black;
    font-size: $text13;
    line-height: $text15;
    /* At medium sizes we want to bump the text up */
    @media (min-width: $M) {
        @mixin FontHeadline;
        font-size: $text15;
        line-height: $text18;
    }
}
```

对于更简单的字体堆栈，变量可以轻松处理这个需求，因此可能更可取。然而，对于更复杂的字体堆栈，混合很适合，其中在某些情况下最好使用某些字体堆栈。例如，也许 LoDPI 需要一个字体，而 HiDPI 需要另一个字体。这些情况不能仅通过使用变量来处理，因此需要根据需要使用混合。

最终，一个项目中应该有十个或更少的 mixins。如果超过这个数量，那么可能是滥用 mixins 来无谓地抽象代码。

## 避免@extend

我第一次接触`@extend`是在使用 Sass 时（[`sass-lang.com/documentation/file.SASS_REFERENCE.html#extend`](http://sass-lang.com/documentation/file.SASS_REFERENCE.html#extend)）。`@extend`指令使一个选择器继承另一个选择器的样式。虽然这可能会带来一些文件大小的好处，但它可能会使调试变得更加困难，因为规则以一种在编写时不一定能够预测的方式组合在一起。

为了确定使用`@extend`是否值得，我在当时正在使用的 Sass 代码库上进行了一个简短的实验。有 73 个实例需要一个 Headline 字体堆栈，37 个实例需要一个 headline condensed 字体堆栈（因此，如果选择 mixins 路线，那就是 73 个`@include Headline`和 37 个`@include HeadlineCondensed`）。

让我们看看没有任何字体引用的文件大小，字体引用定义为 mixins/@includes，然后字体引用为@extend

### 没有字体引用

没有任何字体声明：

105.5 KB（最小化），14.2 KB（Gzipped）

这是我们的*base*或者说控制。让我们看看通过 mixins/@includes 添加所有字体的区别。

### 使用@includes

使用 mixins（Sass 中的@include）*Headline*和*Headline Condensed*的结果 CSS 文件大小为：

146.9 KB（最小化），15.4 KB（Gzipped）

因此，增加了 1.2 KB。`@extend`的表现如何？

### 使用@extend

使用`@extend`而不是`@include`：

106.9 KB（最小化），14.5（Gzipped）；只增加了 0.3 KB 的文件大小。

从这些轶事数据中得出什么结论？对我来说，其他一切都相等的话，如果你绝对想要最小的文件大小，也许`@extend`是一个好选择。虽然节省不多，但确实有一些。

然而，务实地说，如果使用`@include`而不是`@extend`可以获得任何可维护性的收益，我肯定不会担心文件大小。

就我个人而言，我不允许在项目中使用`@extend`功能。它增加了调试的复杂性，而好处很少。

# 8. 一切魔数和浏览器 hack 都应该有注释

每个项目都应该有一个包含与项目相关的所有变量的变量文件。

### 提示

PostCSS 可以在 CSS 文件或 JavaScript 对象中定义变量和 mixins。您可以在[`benfrain.com/creating-and-referencing-javascript-mixins-and-variables-with-postcss/`](https://benfrain.com/creating-and-referencing-javascript-mixins-and-variables-with-postcss/)了解更多关于后者的信息。

如果出现需要在编写样式表中输入基于像素的值，而这些值在变量中尚未定义，那么这应该对你构成一个警示。这种情况也在上面有所涉及。在需要在编写样式表中输入*魔术*数字的情况下，请确保在上一行添加注释以解释其相关性。这可能在当时看起来多余，但请考虑其他人和自己在 3 个月后。为什么你要给那个元素添加一个负边距 17 像素？

例子：

```css
.med-Video {
    position: relative;
    background-color: $color-black;
    font-size: $text13;
    line-height: $text15;
    /*We need some space above to accommodate the
    absolutely positioned icon*/
    margin-top: 20px;
}
```

对于任何设备/浏览器的 hack 也是一样。你可能有自己的语法，但我在 hack 代码的开始上方使用`/*HHHack:*/`前缀添加注释，当我不得不添加代码来满足特定情况时。考虑一下：

```css
.med-Video {
    background-color: $color-black;
    font-size: $text13;
    line-height: $text15;
    /*HHHack needed to force Windows Phone 8.1 to render the full width, reference ticket SMP-234 */
    width: 100%;
}
```

这种覆盖应该尽可能放在规则的最下面。但是，请确保添加注释。否则，未来的作者可能会查看你的代码并认为这行（行）是多余的，然后将其删除。

### 提示

如果你发现你有很多代码，纯粹是为了服务特定的浏览器，你可以考虑将这些规则（手动或使用工具）提取到一个只在需要时提供的单独文件中。

# 9. 不要在编写样式表中放置内联图像

虽然我们继续支持基于 HTTP 的用户（而不是 HTTP2），但内联资源的做法提供了一些优势；主要是减少了为向用户提供页面所需的 HTTP 请求的数量。然而，不鼓励将内联资源放在编写样式表中。

考虑一下：

```css
.rr-Outfit {
    min-height: $size-quadruple;
    background-image:
url(data:image/gif;base64,iVBORw0KGgoAAAANSUhEUgAAABAAAAAQCAMAAAAoLQ9TAAAAA3NCSVQICAjb4U/gAAAAw1BMVEX///8AAACEhIQAAABYWFgAAAAAAAAAAAB6enomJiYAAAAiIiJ4eHgAAABiYmJgYGAAAACJiYkAAAC0tLR/f3/IyMjHx8e/v7+vr6+fn5+ZmZmUlJSBgYFmZmYiIiIQEBAAAAD////w8PDv7+/i4uLh4eHf39/W1tbS0tLR0dHMzMzHx8fFxcXCwsK/v7+4uLi0tLSvr6+rq6ulpaWfn5+ZmZmQkJCPj498fHxwcHBgYGBAQEAwMDAgICAfHx8QEBAAAAAphWZmAAAAQXRSTlMAESIiMzNEVWZmZneIiKqqu8zM3d3u7u7u7u7u7u7u7u7//////////////////////////////////////////wP/q/8AAAAJcEhZcwAACxIAAAsSAdLdfvwAAAAVdEVYdENyZWF0aW9uIFRpbWUAMjEvNS8xMpVX8IQAAAAcdEVYdFNvZnR3YXJlAEFkb2JlIEZpcmV3b3JrcyBDUzVxteM2AAAAw0lEQVQYlU2P6VLCUAyFD1KvWFQ2F1ChnLLW2oIs1ZZC8/5PRW4rM+RH5uSbLCeARt2YO2Pq+I+aafUjRs+PplbVzfeQcbyZzoZuSZoBdyKSkQxdOx+SuQKZK/m8BZ7Iia3lT8m6BQy4/NodpYjpb9OiC3i/U+1NfE08iAdIamXACggwzq7BCGgfrIxt8pOsDbjpRu/k+Q/DBWf3auSVq/1Rz55074d16gT0i8pq4JTPOC9MRIqEbzeXfx860eS71yj1GWENHluGjvJwAAAAAElFTkSuQmCC);
}
```

未来的作者应该如何理解那个资产是什么？

### 提示

如果你在样式表中遇到现有的内联图像，可以复制并粘贴数据到浏览器地址栏中来确定图像是什么。

相反，让工具内联图像。这意味着编写样式表可以提供图像可能是什么的线索，但也使得该图像更容易被替换。如果使用 *postcss-assets* ([`github.com/assetsjs/postcss-assets`](https://github.com/assetsjs/postcss-assets)) 插件，你可以使用内联命令内联图像。下面是之前的例子重写：

```css
.rr-Outfit {
    min-height: $size-quadruple;
    background-image: inline("/path/to-image/relevant-image-name.jpg");
}
```

这不仅更容易阅读，还指定了现有资产的位置。这是一种更好的方法。

# 10. 当简单的 CSS 能够正常工作时，不要写复杂的 CSS

尽量编写尽可能简单的 CSS 代码，以便其他人在未来能够理解。循环、混合和函数很少需要编写。一般规则是，如果一个规则少于 10 个变体，就手动编写。另一方面，如果你需要为一个包含 30 个图像的精灵表创建背景位置，这就是工具应该使用的东西。

这种追求简单性应该延伸到布局的实现方式。如果一个更好支持的布局机制以与较少支持的机制相同的 DOM 节点数量实现了相同的目标，那就使用前者。然而，如果不同的布局机制减少了所需的 DOM 节点数量或提供了额外的好处，但只是不熟悉（例如 Flexbox），花时间了解它可能提供的好处。

# 总结

规则没有执行力就什么都不是。当许多人触及 CSS 代码库时，无论教育、强硬的话语还是文档都无法阻止你的代码库质量被稀释。提供*胡萝卜*只能走得那么远，通常也需要使用一点*棍子*！

在这种情况下，*棍子*将采取静态分析*linting*工具的形式，这些工具可以在作者编写代码时检查和强制执行代码。这种方法可以防止不符合规范的代码进一步传播到有问题的开发者的本地机器之外。在下一章中，我们将讨论如何处理这个问题，以及工具的一般情况。

*乐警*来了！
