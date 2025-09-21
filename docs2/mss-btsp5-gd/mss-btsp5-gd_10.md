# *第八章*：使用 Bootstrap 5 变量、工具 API 和 Sass 定制网站

在本章中，我们将开始定制在前一章中描述和预览的网站。我们将主要通过更改一些用于样式化 Bootstrap 5 所有元素的 Bootstrap 5 变量来实现这一点。在这个过程中，我们将更改一些正在使用的网格和间距工具类，以更好地匹配新样式。我们还将使用工具 API 修改一些现有工具类，并通过使用这些新工具类来优化布局。最后，我们还将添加我们自己的 Sass 来进一步定制网站。

在本章中，将会有许多代码示例和一些截图，但我建议你在不同设备尺寸的浏览器中探索这个定制的网站版本，以真正看到与之前的不同。

在本章中，我们将介绍以下主要内容：

+   导入 Bootstrap 5 文件和自定义样式

+   默认变量覆盖

+   使用现有变量设置变量值

+   工具类

+   其他自定义样式

# 技术要求

+   预览示例需要代码编辑器和浏览器。所有代码示例的源代码都可以在这里找到：[`github.com/PacktPublishing/The-Missing-Bootstrap-5-Guide`](https://github.com/PacktPublishing/The-Missing-Bootstrap-5-Guide)。

+   要将 Sass 编译为 CSS，你需要以下任何一个：

    +   **Node.js**，如果你更喜欢使用终端（macOS）或命令提示符（Windows）的**命令行界面**（CLI）

    +   **Scout-App**，如果你更喜欢使用**图形用户界面**（GUI）

    +   **Visual Studio Code**，如果你更喜欢使用 Visual Studio Code 市场上的扩展

所有这些方法都在 *第二章* *使用和编译 Sass* 中解释。

# 导入 Bootstrap 5 文件和自定义样式

在 *第四章* *Bootstrap 5 全局选项和颜色* 中，我们学习了 `bootstrap.scss` 文件，以及它包含所有将导入 Bootstrap 5 各个元素单独 Sass 部分文件的代码。我们将使用这些 `@import` 语句的不同顺序，这将使定制更容易。我们还将添加四个我们自己的 SCSS 文件的 `@import` 语句，我们使用这些文件来覆盖变量（两个文件用于此目的）、定义工具类和添加自定义样式，在 Bootstrap 5 的 `@import` 语句之间。

`@import` 语句的顺序如下：

+   必须导入 Bootstrap 5 的 `_functions.scss` 文件

+   导入我们自己的 `_default-variable-overrides.scss` 文件，该文件用于默认变量覆盖

+   必须导入 Bootstrap 5 的 `_variables.scss` 文件

+   导入我们自己的 `_variable-value-using-variable.scss` 文件，该文件用于使用现有变量覆盖变量值

+   必须导入 Bootstrap 5 的 `_maps.scss`、`_mixins.scss` 和 `_root.scss` 文件

+   Bootstrap 5 可选部分文件的导入，用于布局、内容、表单和组件

+   Bootstrap 5 的`_helpers.scss`文件导入

+   Bootstrap 5 的`_utilities.scss`文件导入

+   导入我们自己的`_utilities.scss`文件，该文件用于定义我们想要使用实用 API 生成的实用工具

+   Bootstrap 5 的`utilities/_api.scss`文件导入，用于生成实用类

+   导入我们自己的`_custom-styles.scss`文件，该文件用于添加任何额外的自定义 SCSS

接下来，您将看到所有`@import`语句的代码（有关我们自己的文件的注释和`@import`语句的代码已被突出显示）：

part-2/chapter-8/website/scss/style.scss

```js
// Required
```

```js
@import "../../../../bootstrap/scss/functions";
```

```js
// Default variable overrides
```

```js
@import "default-variable-overrides";
```

```js
// Required
```

```js
@import "../../../../bootstrap/scss/variables";
```

```js
// Variable value using existing variable
```

```js
@import "variable-value-using-variable";
```

```js
// Required
```

```js
@import "../../../../bootstrap/scss/maps";
```

```js
@import "../../../../bootstrap/scss/mixins";
```

```js
@import "../../../../bootstrap/scss/root";
```

```js
// Optional Bootstrap CSS
```

```js
@import "../../../../bootstrap/scss/reboot";
```

```js
@import "../../../../bootstrap/scss/type";
```

```js
@import "../../../../bootstrap/scss/images";
```

```js
@import "../../../../bootstrap/scss/containers";
```

```js
@import "../../../../bootstrap/scss/grid";
```

```js
@import "../../../../bootstrap/scss/tables";
```

```js
@import "../../../../bootstrap/scss/forms";
```

```js
@import "../../../../bootstrap/scss/buttons";
```

```js
@import "../../../../bootstrap/scss/transitions";
```

```js
@import "../../../../bootstrap/scss/dropdown";
```

```js
@import "../../../../bootstrap/scss/button-group";
```

```js
@import "../../../../bootstrap/scss/nav";
```

```js
@import "../../../../bootstrap/scss/navbar";
```

```js
@import "../../../../bootstrap/scss/card";
```

```js
@import "../../../../bootstrap/scss/accordion";
```

```js
@import "../../../../bootstrap/scss/breadcrumb";
```

```js
@import "../../../../bootstrap/scss/pagination";
```

```js
@import "../../../../bootstrap/scss/badge";
```

```js
@import "../../../../bootstrap/scss/alert";
```

```js
@import "../../../../bootstrap/scss/progress";
```

```js
@import "../../../../bootstrap/scss/list-group";
```

```js
@import "../../../../bootstrap/scss/close";
```

```js
@import "../../../../bootstrap/scss/toasts";
```

```js
@import "../../../../bootstrap/scss/modal";
```

```js
@import "../../../../bootstrap/scss/tooltip";
```

```js
@import "../../../../bootstrap/scss/popover";
```

```js
@import "../../../../bootstrap/scss/carousel";
```

```js
@import "../../../../bootstrap/scss/spinners";
```

```js
@import "../../../../bootstrap/scss/offcanvas";
```

```js
@import "../../../../bootstrap/scss/placeholders";
```

```js
// Helpers
```

```js
@import "../../../../bootstrap/scss/helpers";
```

```js
// Utilities
```

```js
@import "../../../../bootstrap/scss/utilities";
```

```js
// Define utilities
```

```js
@import "utilities";
```

```js
// Utilities API
```

```js
@import "../../../../bootstrap/scss/utilities/api";
```

```js
// Own custom styles
```

```js
@import "custom-styles";
```

我们现在准备好开始定制我们的网站，并将自定义代码添加到我们自己的四个不同文件中。

我们将从`_default-variable-overrides.scss`文件中的默认变量覆盖代码开始。

# 默认变量覆盖

`_default-variable-overrides.scss`

在此文件中，我们将添加有关颜色、全局选项、全局变量和布局的所有自定义代码，以及一些有关表单和组件的自定义代码。在`_variable-value-using-variable.scss`文件中也会有关于表单和组件的自定义代码（以及其他自定义代码），我们将在本章稍后更详细地研究这个文件。

## 调色板

我们首先想做的事情是定义我们自己的调色板。与我们在本章稍后将要定制的排版一起，这是定义品牌视觉设计的一个重要方面。

我们通过分配新的十六进制颜色值来覆盖所有上下文颜色类，但可以使用其他任何方式来定义颜色。注释中提到的`$secondary`、`$light`和`$dark`变量被分配了 Bootstrap 5 中定义的灰色颜色列表中的颜色值。灰色颜色的变量尚未导入，因此我们无法将它们用作`$secondary`、`$light`和`$dark`的值：

part-2/chapter-8/website/scss/_default-variable-overrides.scss 第 1-10 行

```js
// Color palette
```

```js
$primary: #0464a1;
```

```js
$secondary: #adb5bd; // = $gray-500
```

```js
$success: #2A9D8F;
```

```js
$info: #E9C46A;
```

```js
$warning: #F4A261;
```

```js
$danger: #E76F51;
```

```js
$light: #dee2e6; // = $gray-300
```

```js
$dark: #343a40; // $gray-800
```

在定义了我们自己的调色板之后，我们现在将转到更改全局选项之一。

## 全局选项

对于这个网站，我们只想更改 Bootstrap 5 的阴影选项。我们将它设置为`true`以向各种组件添加阴影：

part-2/chapter-8/website/scss/_default-variable-overrides.scss 第 12-14 行

```js
// Shadows
```

```js
$enable-shadows: true;
```

## 全局变量

全局变量是用于在其他许多地方作为其他变量值的变量。影响最大的全局变量是间距和边框半径的变量，我们现在将更改这些变量。

### 间距

我们可以通过修改`$spacer`变量来更改许多 Bootstrap 5 元素的默认样式，并且我们可以通过修改`$spacers`映射来控制间距实用工具的值。

我们将使用以下代码中看到的新的间距值：

part-2/chapter-8/website/scss/_default-variable-overrides.scss 行 17-26

```js
// Spacer
```

```js
$spacer: 1.5rem;
```

```js
$spacers: (
```

```js
  0: 0,
```

```js
  1: $spacer / 4,
```

```js
  2: $spacer / 2,
```

```js
  3: $spacer,
```

```js
  4: $spacer * 2,
```

```js
  5: $spacer * 4,
```

```js
);
```

增加间隔会影响 Bootstrap 5 的许多元素，包括与响应式网格系统一起使用的网格间隔实用工具。为了在使用嵌套网格系统时在列之间获得更多一致的间距，我们将主页（在热门产品模块中）、商店页面（在产品概览模块中）、产品页面（在相关产品模块中）和愿望单页面（在产品概览模块中）上使用的`.g-4`类更改为`.g-3`。

另一项我们需要进行的微小调整是将产品页面中`<blockquote>`元素中包裹引用图标的三 `<div>` 元素上的`.mt-2`类更改为`.mt-1`。请参见此处代码更改：

part-2/chapter-8/website/product.xhtml

```js
// Original code
```

```js
<div class="text-secondary float-start fs-1 lh-1 mt-2 me-2 border-top border-start border-2 border-secondary p-1"><i class="bi-quote"></i></div>
```

```js
// Updated code
```

```js
<div class="text-secondary float-start fs-1 lh-1 mt-1 me-2 border-top border-start border-2 border-secondary p-1"><i class="bi-quote"></i></div>
```

### 边框半径

我们还可以通过修改与边框半径相关的三个变量来更改许多 Bootstrap 5 元素的默认样式：`$border-radius`、`$border-radius-sm`和`$border-radius-lg`。最后一个边框半径变量`$border-radius-pill`仅用于边框实用工具，因此我们不会更改它。

最激进的改变将是将所有变量设置为`0`，以从所有元素中移除边框半径并拥有直角，但我们将加倍默认边框半径并对其他两个变量进行轻微调整：

part-2/chapter-8/website/scss/_default-variable-overrides.scss 行 28-31

```js
// Border radius
```

```js
$border-radius: .5rem;
```

```js
$border-radius-sm: .25rem;
```

```js
$border-radius-lg: .75rem;
```

## 布局

可以调整布局的样式，包括断点、容器和网格系统。如果我们更改网格系统中的列数，将需要在我们的 HTML 页面中的网格系统标记上进行大量更改，因此我们将坚持更改网站中的断点和容器。

### 断点

断点设置用于网格系统和容器，以及以下具有响应式变化的元素：下拉菜单、列表组、模态框、导航栏、表格和位置辅助工具。

我们将每个断点增加 40 像素，使我们的网格系统与默认 Bootstrap 5 设置略有不同：

part-2/chapter-8/website/scss/_default-variable-overrides.scss 行 34-42

```js
// Breakpoints
```

```js
$grid-breakpoints: (
```

```js
  xs: 0,
```

```js
  sm: 616px,
```

```js
  md: 808px,
```

```js
  lg: 1032px,
```

```js
  xl: 1240px,
```

```js
  xxl: 1440px
```

```js
);
```

### 容器

由于我们增加了断点大小 40 像素，我们还将增加容器最大宽度大小 40 像素。没有必要用相同的值同时更改网格断点和容器最大宽度，但你应该确保在更改任何这些值时进行测试：

part-2/chapter-8/website/scss/_default-variable-overrides.scss 44-51

```js
// Container
```

```js
$container-max-widths: (
```

```js
  sm: 580px,
```

```js
  md: 760px,
```

```js
  lg: 1000px,
```

```js
  xl: 1180px,
```

```js
  xxl: 1360px
```

```js
);
```

我们现在已经对 Bootstrap 5 的布局进行了几项更改，并将继续自定义表单。

## 表单

输入框和按钮已经受到我们之前更改的全球边框半径变量的影响，但不受全球填充变量的影响。因此，我们将调整输入框和按钮的垂直和水平填充。它们共享相同的填充、字体、焦点状态和边框宽度变量，所以我们只需要在一个地方覆盖填充即可影响输入框和按钮：

part-2/chapter-8/website/scss/_default-variable-overrides.scss 行 53-56

```js
// Inputs and buttons
```

```js
$input-btn-padding-y: .5rem;
```

```js
$input-btn-padding-x: 1em;
```

## 组件

在以下内容中，我们将更改我们网站中使用的某些组件的各种属性。

### 面包屑

对于面包屑组件，我们将更改面包屑分隔符的符号：

part-2/chapter-8/website/scss/_default-variable-overrides.scss 行 59-60

```js
// Breadcrumb
```

```js
$breadcrumb-divider: quote("–");
```

### 轮播

对于轮播组件，我们将增加每侧的控制图标宽度，并增加指示器的高度和间距：

part-2/chapter-8/website/scss/_default-variable-overrides.scss 行 62-65

```js
// Carousel
```

```js
$carousel-control-icon-width: 4rem;
```

```js
$carousel-indicator-height: 10px;
```

```js
$carousel-indicator-spacer: 10px;
```

### 下拉菜单

对于下拉菜单组件，我们将设置最小宽度为 `0`，这样当下拉菜单项不包含太多文本时，它不会占用额外的水平空间：

part-2/chapter-8/website/scss/_default-variable-overrides.scss 行 67-68

```js
// Dropdown
```

```js
$dropdown-min-width: 0;
```

### 模态框

对于模态框组件，我们将增加大型模态框的大小，增加背景的透明度，并改变其打开和关闭时的过渡效果：

part-2/chapter-8/website/scss/_default-variable-overrides.scss 行 70-73

```js
// Modal
```

```js
$modal-lg: 900px;
```

```js
$modal-backdrop-opacity: .75;
```

```js
$modal-fade-transform: rotate(5deg) scale(.9);
```

### 导航栏

对于导航栏组件，我们将增加导航链接的垂直填充，使其更大：

part-2/chapter-8/website/scss/_default-variable-overrides.scss 行 75-76

```js
// Navs
```

```js
$nav-link-padding-y: 1rem;
```

### 进度条

对于进度条组件，我们将增加其高度，减少动画持续时间，并减少过渡持续时间：

part-2/chapter-8/website/scss/_default-variable-overrides.scss 行 78-81

```js
// Progress
```

```js
$progress-height: 2rem;
```

```js
$progress-bar-animation-timing: .5s linear infinite;
```

```js
$progress-bar-transition: width .4s ease;
```

### 旋转器

对于旋转器组件，我们将增加动画持续时间，使其旋转速度稍微慢一些：

part-2/chapter-8/website/scss/_default-variable-overrides.scss 行 83-84

```js
// Spinner
```

```js
$spinner-animation-speed: 1s;
```

我们现在已经完成了所有关于默认变量覆盖的自定义设置，因此我们将继续使用现有变量进行进一步的变量值自定义。

# 使用现有变量设置变量值

`_variable-value-using-variable.scss`

在此文件中，我们将添加关于内容的所有自定义代码，关于表单和组件的一些自定义代码，以及一个辅助类的自定义代码。我们之所以在导入所需的 Bootstrap 5 `_variables.scss` 文件之后导入此文件，是因为我们希望使用 `_variables.scss` 中的现有变量作为覆盖与内容、某些表单和组件变量以及一个辅助类变量相关联的新值。这在我们需要将颜色变量分配给其他变量时特别有用。

`_variable-value-using-variable.scss`文件中的某些代码不必放在那里，也可以放在`_default-variable-overrides.scss`文件中。但是，我选择尽可能地将相关的变量分组在一起，这就是为什么所有与内容相关的变量都集中在这里。

## 内容

Bootstrap 5 的内容类别包括排版、图片、表格和图表。我们将对其中所有内容进行更改，除了表格。

### 排版

如前所述，排版是视觉设计的一个重要方面。因此，我们将对整体排版进行重大更改。

首先，我们将使用这两个来自 Google Fonts 的免费字体：Roboto 和 Comfortaa。这可以通过在所有 HTML 文件的`</head>`标签之前添加以下三个`<link>`标签来实现：

*.xhtml 行 9-11

```js
<link rel="preconnect" href="https://fonts.googleapis.com">
```

```js
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
```

```js
<link href="https://fonts.googleapis.com/css2?family=Comfortaa:wght@300;500&family=Roboto:wght@300;400;700&display=swap" rel="stylesheet">
```

现在可以使用两个 Google Fonts 字体，它们将被添加到下一个代码块中的`$font-family-base`和`$headings-font-family`变量中。

这里是我们对排版所做的所有更改的概述：

+   `<body>`的背景颜色和文本颜色

+   默认字体家族、字体大小和行高

+   链接的样式

+   正常标题的字体家族、字体大小、字体粗细、行高和底部外边距

+   显示标题的字体大小和行高

+   首段文字的字体大小

看看以下代码中是如何实现所有这些更改的：

part-2/chapter-8/website/scss/_variable-value-using-variable.scss 行 2-37

```js
// Typograhy
```

```js
// Body styles
```

```js
$body-bg: $gray-100;
```

```js
$body-color: $gray-700;
```

```js
// Basic typography styles
```

```js
$font-family-base: 'Roboto', sans-serif;
```

```js
$font-size-base: 1.125rem;
```

```js
$line-height-base: 1.4;
```

```js
$line-height-sm: 1.2;
```

```js
// Link styles
```

```js
$link-decoration: none;
```

```js
$link-hover-color: shift-color($link-color, $link-shade-percentage);
```

```js
$link-hover-decoration: underline;
```

```js
// Heading styles
```

```js
$headings-font-family: 'Comfortaa', cursive;
```

```js
$h1-font-size: $font-size-base * 3;
```

```js
$h2-font-size: $font-size-base * 2.5;
```

```js
$h3-font-size: $font-size-base * 2;
```

```js
$h4-font-size: $font-size-base * 1.75;
```

```js
$h5-font-size: $font-size-base * 1.5;
```

```js
$h6-font-size: $font-size-base * 1.25;
```

```js
$headings-font-weight: $font-weight-bold;
```

```js
$headings-line-height: $line-height-sm;
```

```js
$headings-margin-bottom: $spacer * 0.25;
```

```js
// Display styles
```

```js
$display-font-sizes: (
```

```js
  1: $h1-font-size * 1.5,
```

```js
  2: $h2-font-size * 1.5,
```

```js
  3: $h3-font-size * 1.5,
```

```js
  4: $h4-font-size * 1.5,
```

```js
  5: $h5-font-size * 1.5,
```

```js
  6: $h6-font-size * 1.5
```

```js
);
```

```js
$display-line-height: $headings-line-height;
```

```js
// Lead styles
```

```js
$lead-font-size: $font-size-base * 1.375;
```

### 图片

对于图片，我们将对图像缩略图进行一些更改。我们将增加填充，更改背景颜色，更改边框颜色，并减小边框半径：

part-2/chapter-8/website/scss/_variable-value-using-variable.scss 行 39-43

```js
// Images
```

```js
$thumbnail-padding: .5rem;
```

```js
$thumbnail-bg: $body-bg;
```

```js
$thumbnail-border-color: $gray-500;
```

```js
$thumbnail-border-radius: $border-radius-sm;
```

注意，我们刚刚将图像缩略图的背景颜色设置为我们在前面的排版代码块中更改的`<body>`的背景颜色。我们将使用该变量（`$body-bg`）或相关的变量（用于`<body>`文本颜色的`$body-color`）来对所有剩余的此文件中的自定义进行修改。

### 图表

对于图表，我们将增加字体大小并更改图表标题的文本颜色：

part-2/chapter-8/website/scss/_variable-value-using-variable.scss 行 45-47

```js
// Figures
```

```js
$figure-caption-font-size: $font-size-lg;
```

```js
$figure-caption-color: $body-color;
```

## 表单

我们将设置输入元素、复选框、单选按钮和选择元素的背景颜色。我们首先将`$body-bg`值分配给`$input-bg`，然后使用该变量（`$input-bg`）作为`$form-check-input-bg`和`$form-select-bg`变量的值：

part-2/chapter-8/website/scss/_variable-value-using-variable.scss 行 49-53

```js
// Form controls
```

```js
$input-bg: $body-bg;
```

```js
$form-check-input-bg: $input-bg;
```

```js
$form-select-bg: $input-bg;
```

## 组件

如您接下来所见，我们将放置那些需要`$body-bg`变量作为背景颜色值的组件变量（在此文件和内容自定义之后）。

### 卡片

对于卡片组件，我们将更改背景颜色：

part-2/chapter-8/website/scss/_variable-value-using-variable.scss 行 56-57

```js
// Card
```

```js
$card-bg: $body-bg;
```

### 列表组

对于列表组组件，我们将更改背景颜色：

part-2/chapter-8/website/scss/_variable-value-using-variable.scss 行 59-60

```js
// List group
```

```js
$list-group-bg: $body-bg;
```

## 辅助类

由于辅助类不能使用 `_variables.scss` 文件中的变量进行自定义，因为它们没有与 Bootstrap 5 元素相同的视觉属性，所以不能使用变量进行自定义。例外的是比率辅助类，我们将在下一节中看到。

### 比率

比率辅助类是从 `$aspect-ratios` Sass 映射生成的。我们可以通过将其与自定义映射合并来向此映射添加更多值：

part-2/chapter-8/website/scss/_variable-value-using-variable.scss 行 62-69

```js
// Ratio
```

```js
// Create custom ratio map
```

```js
$custom-ratio: (
```

```js
  "5x3": calc(3 / 5 * 100%)
```

```js
);
```

```js
// Merge "$aspect-ratios" and "$custom-ratio"
```

```js
$aspect-ratios: map-merge($aspect-ratios, $custom-ratio);
```

在前面的代码中，通过新的属性和宽高比 `5` 比 `3` 创建了新的 `$custom-ratio` Sass 映射。该值基于与 `$aspect-ratios` Sass 映射中现有值相同的计算。当这些映射合并时，我们现在得到了新的 `.ratio-5x3` 辅助类。在此更改之后，我们需要更新已经在我们的联系页面使用的比率辅助类，如下所示：

part-2/chapter-8/website/contact.xhtml

```js
// Original code
```

```js
<div class="ratio ratio-21x9">
```

```js
  <iframe src="img/[URL for Google Maps embed]" 
```

```js
    allowfullscreen></iframe>
```

```js
</div>
```

```js
// Updated code
```

```js
<div class="ratio ratio-5x3">
```

```js
  <iframe src="img/[URL for Google Maps embed]" 
```

```js
    allowfullscreen></iframe>
```

```js
</div>
```

我们现在已经完成了所有辅助类的自定义，因此我们将继续自定义实用程序。

# 实用程序

`_utilities.scss`

在此文件中，我们将添加所有使用实用程序 API 的自定义代码。一些实用程序已经进行了自定义，因为它们的值基于 `_variables.scss` 文件中的变量，我们已经在本章前面覆盖了这些变量。这包括间距实用程序和各种文本实用程序。额外的自定义需要使用实用程序 API 的代码。

在使用实用程序 API 修改任何实用程序之前，我们应该考虑我们已经在使用的哪些实用程序在当前形式下不足或不灵活，然后进行修改。

对于我们的网站，我们将在购物车页面改进两个方面。首先，我们希望通过使用更灵活的边框实用程序来改进**摘要**部分**运输详情**标签页和**支付选项**标签页中围绕我们的产品概览的 `<div>` 元素中元素的响应式布局。

由于我们不仅修改了一些实用程序的值，而且使它们更加灵活，从而生成了我们可以使用的更多实用程序类，因此我们还需要相应地更新 HTML。

### 尺寸

我们想改变 `width` 的尺寸实用程序，以便更好地控制断点之间的一些元素的布局。我们将使 `width` 的实用程序响应式，并使用以下代码添加额外的属性和值 `33: 33%` 和 `67: 67%`：

part-2/chapter-8/website/scss/_utilities.scss 行 1-20

```js
// Make sizing utilities for width responsive and add extra 
```

```js
// values
```

```js
$utilities: map-merge(
```

```js
  $utilities,
```

```js
  (
```

```js
    "width": (
```

```js
      property: width,
```

```js
      responsive: true,
```

```js
      class: w,
```

```js
      values: (
```

```js
        25: 25%,
```

```js
        33: 33%,
```

```js
        50: 50%,
```

```js
        67: 67%,
```

```js
        75: 75%,
```

```js
        100: 100%,
```

```js
        auto: auto
```

```js
      )
```

```js
    )
```

```js
  )
```

```js
);
```

这将生成大量的新响应式 `width` 实用程序类以及我们添加的新属性和值的类，因此我们现在将按以下方式更新我们的 HTML：

part-2/chapter-8/website/cart.xhtml

```js
// Original code
```

```js
<div class="d-flex align-items-start me-3 mb-3 mb-md-0">[code for product]</div>
```

```js
<div class="d-flex">
```

```js
  <div class="input-group w-auto me-3">[code for input 
```

```js
    group]</div>
```

```js
  [code for remove button]
```

```js
</div>
```

```js
// Updated code
```

```js
<div class="d-flex align-items-start me-3 mb-3 mb-md-0 w-md-67 w-lg-75 w-xl-67 w-xxl-75">[code for product]</div>
```

```js
<div class="d-flex w-75 w-sm-50 w-md-33 w-lg-25 w-xl-33 w-xxl-25">
```

```js
  <div class="input-group w-auto me-3">[code for input 
```

```js
    group]</div>
```

```js
  [code for remove button]
```

```js
</div>
```

在前面的代码中，我们现在分别为我们布局的两个不同部分使用 `4` 和 `6` 个宽度实用工具类。如果我们看第二部分，我们可以看到我们在调整每个现有断点的宽度。这在处理响应式设计时有时是必要的，以获得最佳可能的布局。

### 边框

我们希望改进 `<div>` 元素在 `xs`、`sm`、`xl` 和 `xxl` 断点上的边框使用，并在 `md` 和 `lg` 断点之间转换为两列网格。我们希望对于 `md` 和 `lg` 断点，边框放置在右侧，对于所有其他断点，放置在底部。

为了实现这一点，我们需要使 `border-width`、`border-bottom` 和 `border-end` 的边框实用工具响应式，我们可以通过以下代码来完成：

part-2/chapter-8/website/scss/_utilities.scss 行 22-38

```js
// Make border utilities for border-width, border-bottom 
```

```js
// and border-end responsive
```

```js
$utilities: map-merge(
```

```js
  $utilities, (
```

```js
    "border-width": map-merge(
```

```js
      map-get($utilities, "border-width"),
```

```js
      ( responsive: true ),
```

```js
    ),
```

```js
    "border-bottom": map-merge(
```

```js
      map-get($utilities, "border-bottom"),
```

```js
      ( responsive: true ),
```

```js
    ),
```

```js
    "border-end": map-merge(
```

```js
      map-get($utilities, "border-end"),
```

```js
      ( responsive: true ),
```

```js
    ),
```

```js
  )
```

```js
);
```

这将生成大量的新响应式边框实用工具类，因此我们现在需要更新我们的 HTML。我们还将使用间距实用工具来优化页面该部分的填充和边距布局，结果如下所示：

part-2/chapter-8/website/cart.xhtml

```js
// Original code
```

```js
<div class="border-bottom border-2 border-light mb-4 pb-4">[code for products]</div>
```

```js
// Updated code
```

```js
<div class="border-bottom border-2 border-light mb-4 pb-4 border-bottom-md-0 border-end-md border-md-2 pb-md-0 mb-md-0 border-bottom-xl border-end-xl-0 border-xl-2 pb-xl-4 mb-xl-4">[code for products]</div>
```

我们已经为实用工具完成了所有自定义设置，因此我们将继续进行其他自定义样式的设置。

# 其他自定义样式

`_custom-styles.scss`

在此文件中，我们将添加所有无法通过覆盖 `_variables.scss` 文件中现有变量的组件的自定义代码。相反，我们将使用 CSS 类选择器针对组件，并使用常规 CSS 和 Sass 代码的组合来修改样式。即使我们无法覆盖现有变量，我们仍然可以将它们用作 CSS 属性的值，我们将为此部分中的两个组件这样做。

## 警报

我们希望在 `alert` 组件的左侧添加一个粗边框，边框颜色继承文本颜色，厚度是 `$spacer` 变量值的一半：

part-2/chapter-8/website/scss/_custom-styles.scss 行 1-4

```js
// Customize the left border of the alert component
```

```js
.alert {
```

```js
  border-left: ($spacer * .5) solid;
```

```js
}
```

前面代码中使用的 `border-left` 简写等于以下内容：

```js
border-left-width: $spacer * .5;
```

```js
border-left-style: solid;
```

```js
border-left-color: currentColor;
```

由于默认的 `currentColor` 边框颜色值，左侧边框的颜色将继承警报文本颜色的颜色，这取决于所使用的警报类型。在我们的网站上，我们目前使用 `$info` 颜色上的信息警报。

## 抽屉式布局

我们希望将 `_variables.scss` 文件的头信息作为我们的 CSS 属性的值。这导致了以下代码：

part-2/chapter-8/website/scss/_custom-styles.scss 行 6-10

```js
// Customize the background color and bottom border of the 
```

```js
// header of the offcanvas component
```

```js
.offcanvas-header {
```

```js
  background-color: $card-cap-bg;
```

```js
  border-bottom: $card-border-width solid $card-border-color;
```

```js
}
```

# 摘要

在本章中，我们学习了与真实网站相关的 Bootstrap 5 定制在实际中的工作方式。首先，我们学习了如何混合和排序 Bootstrap 5 Sass 部分和我们的自定义 Sass 部分的`@import`语句，以实现最大的定制性。然后，我们学习了如何使用 Bootstrap 5 变量来定制布局、内容、表单、组件和辅助元素。接着，我们学习了如何使用实用 API 修改现有实用工具，使它们更加灵活并改善我们网站的布局。最后，我们学习了如何使用我们自己的 Sass 代码以不同于默认方式的方式来定制组件。

在下一章中，我们将看到如何进一步改进和扩展我们的网站，包括如何添加自定义组件和添加交互功能。
