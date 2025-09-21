# *第六章*：理解和使用 Bootstrap 5 实用 API

在本章中，我们将学习如何使用 **实用 API** 生成和添加新的简单和复杂实用类，以及覆盖、修改和删除现有实用类。

当您需要调整组件或整体布局时，了解如何使用实用 API 是有用的，例如，控制元素的空间或尺寸、添加上下文颜色、更改边框或创建基于 **flexbox** 的布局。

在本章中，我们将涵盖以下主题：

+   关于实用 API

+   理解实用 API 语法

+   使用实用 API

对于最后两个部分，将有大量的代码示例来帮助理解强大的实用 API。

# 技术要求

+   要预览示例，您需要一个代码编辑器和浏览器。所有代码示例的源代码可以在以下位置找到：[`github.com/PacktPublishing/The-Missing-Bootstrap-5-Guide`](https://github.com/PacktPublishing/The-Missing-Bootstrap-5-Guide)。

+   您需要一个 Sass 编译器来编译 Sass 文件到 CSS。请参阅 *第二章*，*使用和编译 Sass*，了解不同的方法。

# 关于实用 API

Bootstrap 5 特有的所谓实用 API，用于生成不同的实用类和/或 CSS 自定义属性（也称为 CSS 变量）。它的工作方式如下：

+   `_utilities.scss` 文件包含 `$utilities` 变量，这是一个 Sass 映射，包含由实用 API 使用的各种设置。实际上，它是多个映射，通过使用 `map-merge()` Sass 函数合并成一个包含嵌套映射的单个映射。共有 83 组实用类，它们按名称分组。它们的设置和值在每个组中都有所不同，但其中一些共享一些值（例如，一些弹性实用类共享 `flex` 类前缀）。

+   `$utilities` 变量在 `utilities/_api.scss` 文件中使用，该文件在其他所有 Bootstrap 5 元素导入之后。

+   在 `utilities/_api.scss` 文件中，`$utilities` 变量作为参数传递给位于 `mixins/_utilities.scss` 文件的 `generate-utility()` 混合。

+   `generate-utility()` 混合是 Bootstrap 5 在 `mixin` 子文件夹中找到的各种混合中最大和最先进的。当传递一个包含所需和可选值的 Sass 映射给它时，真正的魔法就发生了。

我们不会详细学习实用 API 中的 Sass 代码是如何工作的，而是将学习其语法，并查看不同示例，了解我们如何使用它来生成和添加简单和复杂的新实用类，以及覆盖、修改和删除现有实用类。

# 理解实用 API 语法

现在，让我们看看实用 API 的语法。首先是所有选项的语法概述，每行代码上方有一行注释。在注释中，你可以看到名称，在括号中，你可以看到选项是必需的还是可选的，在代码行中，你可以看到名称，后面跟着在方括号中接受的值的类型，以及（如果有）括号中的默认值：

```js
$utilities: (
```

```js
  // Name (required)
```

```js
  "[String]": (
```

```js
    // Property (required)
```

```js
    property: [name],
```

```js
    // Values (required)
```

```js
    values: [space-separated list or map],
```

```js
    // Class (optional)
```

```js
    class: [name (null)],
```

```js
    // CSS variable utilities (optional)
```

```js
    css-var: [Boolean (false)],
```

```js
    // CSS variable name (optional)
```

```js
    css-variable-name: [name (null)],
```

```js
    // Local CSS variables (optional)
```

```js
    local-vars: [map (null)],
```

```js
    // State (optional)
```

```js
    state: [space-separated list (null)],
```

```js
    // Responsive (optional)
```

```js
    responsive: [Boolean (false)],
```

```js
    // RFS (optional)
```

```js
    rfs: [Boolean (false)],
```

```js
    // Print (optional),
```

```js
    print: [Boolean (false)],
```

```js
    // RTL,
```

```js
    rtl: [Boolean (true)]
```

```js
  )
```

```js
);
```

我们将逐一介绍这些选项，并查看它们一起的作用，以下是一些示例。

## 关于代码示例

本章中所有代码示例的 Bootstrap 5 导入已减少，仅包括与实用 API 一起工作时所需的导入。对于本节中与实用 API 语法相关的示例，SCSS 文件顶部的必需导入如下：

style.scss

```js
// Required
```

```js
@import "../../../../../bootstrap/scss/functions";
```

```js
@import "../../../../../bootstrap/scss/variables";
```

```js
@import "../../../../../bootstrap/scss/maps";
```

```js
@import "../../../../../bootstrap/scss/mixins";
```

对于其他与使用实用 API 相关的示例，SCSS 文件顶部的必需导入如下：

style.scss

```js
// Required
```

```js
@import "../../../../../bootstrap/scss/functions";
```

```js
@import "../../../../../bootstrap/scss/variables";
```

```js
@import "../../../../../bootstrap/scss/maps";
```

```js
@import "../../../../../bootstrap/scss/mixins";
```

```js
@import "../../../../../bootstrap/scss/root";
```

```js
@import "../../../../../bootstrap/scss/reboot";
```

在必需的导入之后，我们导入默认的实用工具，添加我们自己的自定义代码，最后导入实用工具 API。它看起来像这样：

style.scss

```js
// Utilities
```

```js
@import "../../../../../bootstrap/scss/utilities";
```

```js
// Custom code here
```

```js
// Utilities API
```

```js
@import "../../../../../bootstrap/scss/utilities/api";
```

现在我们已经看到了本章 SCSS 文件的结构，我们可以省略仅导入 Bootstrap 5 部分的代码，专注于自定义代码。HTML 示例也已减少，仅显示必要的代码。记住，对于每个代码示例，都有指向包含所有导入的完整示例的文件引用。

## 名称

这是实用工具的名称，应该是一个带引号的字符串。它在生成的 CSS 中不被使用，所以它更多的是一种组织代码的方式：

```js
$utilities: (
```

```js
  "Utility name": (
```

```js
    // Options defined here
```

```js
  )
```

```js
);
```

## CSS 属性

CSS 属性应该写成不带引号的，并且必须是一个有效的 CSS 属性。它可以是一个 CSS 属性（最常见）或一个由空格分隔的 CSS 属性列表。它用于 CSS 规则，如果未定义`class`键，它也是默认类名。

### 一个 CSS 属性

这里有一个包含一个 CSS 属性的示例：

part-1/chapter-6/api-syntax/css-property-single/scss/style.scss

```js
// Required imports
```

```js
// Utilities
```

```js
$utilities: (
```

```js
  "Font weight": (
```

```js
    property: font-weight,
```

```js
    values: 300 400 700
```

```js
  )
```

```js
);
```

```js
// Utilities API
```

这编译成以下内容：

part-1/chapter-6/api-syntax/css-property-single/css/style.css

```js
.font-weight-300 { font-weight: 300 !important }
```

```js
.font-weight-400 { font-weight: 400 !important }
```

```js
.font-weight-700 { font-weight: 700 !important }
```

现在，我们可以通过以下 HTML 来查看这个实用工具的效果：

part-1/chapter-6/api-syntax/css-property-single/index.xhtml

```js
<p class="font-weight-300">Lorem ipsum...</p>
```

```js
<p class="font-weight-400">Lorem ipsum...</p>
```

```js
<p class="font-weight-700">Lorem ipsum...</p>
```

要查看这个示例的实际效果，请在一个浏览器中打开 HTML 文件。

### 多个 CSS 属性

这里有一个包含多个 CSS 属性的示例：

part-1/chapter-6/api-syntax/css-property-multiple/scss/style.scss

```js
// Required imports
```

```js
// Utilities
```

```js
$utilities: (
```

```js
  "Spacing": (
```

```js
    property: margin padding,
```

```js
    values: $spacers,
```

```js
    class: spacing
```

```js
  )
```

```js
);
```

```js
// Utilities API
```

这编译成以下内容：

part-1/chapter-6/api-syntax/css-property-multiple/css/style.css

```js
.spacing-0 {
```

```js
  margin: 0 !important;
```

```js
  padding: 0 !important
```

```js
}
```

```js
.spacing-1 {
```

```js
  margin: .25rem !important;
```

```js
  padding: .25rem !important
```

```js
}
```

```js
.spacing-2 {
```

```js
  margin: .5rem !important;
```

```js
  padding: .5rem !important
```

```js
}
```

```js
.spacing-3 {
```

```js
  margin: 1rem !important;
```

```js
  padding: 1rem !important
```

```js
}
```

```js
.spacing-4 {
```

```js
  margin: 1.5rem !important;
```

```js
  padding: 1.5rem !important
```

```js
}
```

```js
.spacing-5 {
```

```js
  margin: 3rem !important;
```

```js
  padding: 3rem !important
```

```js
}
```

现在，我们可以通过以下 HTML 来查看这个实用工具的效果：

part-1/chapter-6/api-syntax/css-property-multiple/index.xhtml

```js
<div style="margin: 1rem; padding: 1rem;" 
```

```js
  class="spacing-0">.spacing-0</div>
```

```js
<div class="spacing-1">.spacing-1</div>
```

```js
<div class="spacing-2">.spacing-2</div>
```

```js
<div class="spacing-3">.spacing-3</div>
```

```js
<div class="spacing-4">.spacing-4</div>
```

```js
<div class="spacing-5">.spacing-5</div>
```

要查看这个示例的实际效果，请在一个浏览器中打开 HTML 文件。

## 值

这指定了指定 CSS 属性的值，这些值在生成类名和 CSS 规则时使用。

它可以是一个空格分隔的值列表，用于类名和 CSS 规则：

```js
values: 300 400 700
```

或者，如果您不想让类名与值相同，您可以使用映射：

```js
values: (
```

```js
  light: 300,
```

```js
  normal: 400,
```

```js
  bold: 700
```

```js
)
```

让我们再次使用第一个示例中的一个 CSS 属性，并将值从列表更改为映射：

part-1/chapter-6/api-syntax/values/scss/style.scss

```js
// Required imports
```

```js
// Utilities
```

```js
$utilities: (
```

```js
  "Font weight": (
```

```js
    property: font-weight,
```

```js
    values: (
```

```js
      light: 300,
```

```js
      normal: 400,
```

```js
      bold: 700
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

```js
// Utilities API
```

这会编译成以下内容：

part-1/chapter-6/api-syntax/values/css/style.css

```js
.font-weight-light { font-weight: 300 !important }
```

```js
.font-weight-normal { font-weight: 400 !important }
```

```js
.font-weight-bold { font-weight: 700 !important }
```

还可以引用存储列表或映射的 Sass 变量：

```js
values: $font-weights
```

我们现在可以通过以下 HTML 来看到这个实用工具的效果：

part-1/chapter-6/api-syntax/values/index.xhtml

```js
<p class="font-weight-light">Lorem ipsum...</p>
```

```js
<p class="font-weight-normal">Lorem ipsum...</p>
```

```js
<p class="font-weight-bold">Lorem ipsum...</p>
```

要查看此效果，请在一个浏览器中打开 HTML 文件。

## 类

此选项用于更改编译 CSS 中使用的 `class` 前缀。这是可选的，并且前缀应不带引号。考虑我们之前的第一个示例：

part-1/chapter-6/api-syntax/class/scss/style.scss

```js
// Required imports
```

```js
// Utilities
```

```js
$utilities: (
```

```js
  "Font weight": (
```

```js
    property: font-weight,
```

```js
    values: 300 400 700,
```

```js
    class: fw
```

```js
  )
```

```js
);
```

```js
// Utilities API
```

这会编译成以下内容：

part-1/chapter-6/api-syntax/class/css/style.css

```js
.fw-300 { font-weight: 300 !important; }
```

```js
.fw-400 { font-weight: 400 !important; }
```

```js
.fw-700 { font-weight: 700 !important; }
```

我们现在可以通过以下 HTML 来看到这个实用工具的效果：

part-1/chapter-6/api-syntax/class/index.xhtml

```js
<p class="fw-300">Lorem ipsum...</p>
```

```js
<p class="fw-400">Lorem ipsum...</p>
```

```js
<p class="fw-700">Lorem ipsum...p>
```

要查看此效果，请在一个浏览器中打开 HTML 文件。

## CSS 变量实用工具

这是一个布尔选项，可以用来为选择器生成 CSS 变量而不是常规 CSS 规则。

让我们将第一个示例改为生成 CSS 变量而不是常规 CSS 规则：

part-1/chapter-6/api-syntax/css-variable-utilities/scss/style.scss

```js
// Required imports
```

```js
// Utilities
```

```js
$utilities: (
```

```js
  "Font weight": (
```

```js
    property: font-weight,
```

```js
    values: 300 400 700,
```

```js
    css-variable-name: fw,
```

```js
    css-var: true
```

```js
  )
```

```js
);
```

```js
// Utilities API
```

这会编译成以下内容：

part-1/chapter-6/api-syntax/css-variable-utilities/css/style.css

```js
.font-weight-300 { --bs-fw: 300 }
```

```js
.font-weight-400 { --bs-fw: 400 }
```

```js
.font-weight-700 { --bs-fw: 700 }
```

我们现在可以通过以下 HTML 来看到这个实用工具的效果：

part-1/chapter-6/api-syntax/css-variable-utilities/index.xhtml

```js
<p class="font-weight-300">Lorem ipsum...</p>
```

```js
<p class="font-weight-400">Lorem ipsum...</p>
```

```js
<p class="font-weight-700">Lorem ipsum...</p>
```

此 HTML 必须与以下 CSS 结合使用，该 CSS 位于我们示例中 HTML 文件的 `<style>` 标签中：

part-1/chapter-6/api-syntax/css-variable-utilities/index.xhtml

```js
p[class^="font-weight"] { font-weight: var(--bs-fw); }
```

CSS 选择器针对页面上所有以 `font-weight` 开头的段落元素，然后使用存储在各个类名中的 `--bs-fw` 变量设置字体粗细。

要查看此效果，请在一个浏览器中打开 HTML 文件。

## CSS 变量名称

此选项定义了在默认 `--bs-` 前缀之后生成的 CSS 变量实用工具的前缀，正如我们在上一个示例中所做的那样，与 `css-var: true` 一起使用，如下所示：`css-variable-name: fw`。完整的前缀，正如我们所见，变成了 `--bs-fw`。

## 本地 CSS 变量

使用此布尔选项，实用工具 API 仍然会生成 CSS 规则，但也会在这些规则内生成本地 CSS 变量。您必须指定一个 Sass 映射作为值，其中映射的键是 CSS 变量名称，映射的值是 CSS 变量值。

让我们将第一个示例也改为生成本地 CSS 变量：

part-1/chapter-6/api-syntax/local-css-variables/scss/style.scss

```js
// Required imports
```

```js
// Utilities
```

```js
$utilities: (
```

```js
  "Font weight": (
```

```js
    property: font-weight,
```

```js
    values: 300 400 700,
```

```js
    local-vars: (
```

```js
      font-weight-opacity: 0.5 
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

```js
// Utilities API
```

这会编译成以下内容：

part-1/chapter-6/api-syntax/local-css-variables/css/style.css

```js
.font-weight-300 {
```

```js
  --bs-font-weight-opacity: .5;
```

```js
  font-weight: 300 !important
```

```js
}
```

```js
.font-weight-400 {
```

```js
  --bs-font-weight-opacity: .5;
```

```js
  font-weight: 400 !important
```

```js
}
```

```js
.font-weight-700 {
```

```js
  --bs-font-weight-opacity: .5;
```

```js
  font-weight: 700 !important
```

```js
}
```

我们现在可以通过以下 HTML 来看到这个实用工具的效果：

part-1/chapter-6/api-syntax/local-css-variables/index.xhtml

```js
<p class="font-weight-300">Lorem ipsum...</p>
```

```js
<p class="font-weight-400">Lorem ipsum...</p>
```

```js
<p class="font-weight-700">Lorem ipsum...</p>
```

此 HTML 必须与以下 CSS 结合使用，该 CSS 位于我们示例中 HTML 文件的 `<style>` 标签中：

part-1/chapter-6/api-syntax/local-css-variables/index.xhtml

```js
p[class^="font-weight"] {
```

```js
  opacity: var(--bs-font-weight-opacity);
```

```js
}
```

CSS 选择器针对页面上所有以 `font-weight` 开头的类段元素，然后使用存储与各种类名一起的 `--bs-font-weight-opacity` 变量设置字体粗细。

要看到这个功能的效果，请在浏览器中打开 HTML 文件。

## 状态

此选项用于在正常实用工具类之外生成伪类变体。您可以指定单个伪类或以空格分隔的伪类列表。当使用 `state` 选项时，实用工具 API 将为该伪类生成类名，并将伪类的名称附加到类名中。

让我们再次更改我们的第一个示例：

part-1/chapter-6/api-syntax/state/scss/style.scss

```js
// Required imports
```

```js
// Utilities
```

```js
$utilities: (
```

```js
  "Font weight": (
```

```js
    property: font-weight,
```

```js
    values: 300 400 700,
```

```js
    state: hover
```

```js
  )
```

```js
);
```

```js
// Utilities API
```

这将编译成以下内容：

part-1/chapter-6/api-syntax/state/css/style.css

```js
.font-weight-300 { font-weight: 300 !important; }
```

```js
.font-weight-300-hover:hover { font-weight: 300 !important; }
```

```js
.font-weight-400 { font-weight: 400 !important; }
```

```js
.font-weight-400-hover:hover { font-weight: 400 !important; }
```

```js
.font-weight-700 { font-weight: 700 !important; }
```

```js
.font-weight-700-hover:hover { font-weight: 700 !important; }
```

我们现在可以通过使用以下 HTML 来看到这个实用工具的效果：

part-1/chapter-6/api-syntax/state/index.xhtml

```js
<p class="font-weight-300-hover">Lorem ipsum...</p>
```

```js
<p class="font-weight-400-hover">Lorem ipsum...</p>
```

```js
<p class="font-weight-700-hover">Lorem ipsum...</p>
```

要看到这个功能的效果，请在浏览器中打开 HTML 文件并将鼠标悬停在段落上。

## 响应式

此布尔选项用于在所有断点处生成响应式实用工具。它将设备缩写作为中缀插入到类名中。

让我们使我们的示例具有响应性：

part-1/chapter-6/api-syntax/responsive/scss/style.scss

```js
// Required imports
```

```js
// Utilities
```

```js
$utilities: (
```

```js
  "Font weight": (
```

```js
    property: font-weight,
```

```js
    values: 300 400 700,
```

```js
    responsive: true
```

```js
  )
```

```js
);
```

```js
// Utilities API
```

这将编译成以下内容：

part-1/chapter-6/api-syntax/responsive/css/style.css

```js
.font-weight-300 { font-weight: 300 !important; }
```

```js
.font-weight-400 { font-weight: 400 !important; }
```

```js
.font-weight-700 { font-weight: 700 !important; }
```

```js
@media (min-width: 576px) {
```

```js
  .font-weight-sm-300 { font-weight: 300 !important; }
```

```js
  .font-weight-sm-400 { font-weight: 400 !important; }
```

```js
  .font-weight-sm-700 { font-weight: 700 !important; }
```

```js
}
```

```js
@media (min-width: 768px) {
```

```js
  .font-weight-md-300 { font-weight: 300 !important; }
```

```js
  .font-weight-md-400 { font-weight: 400 !important; }
```

```js
  .font-weight-md-700 { font-weight: 700 !important; }
```

```js
}
```

```js
// and so on...
```

我们现在可以通过使用以下 HTML 来看到这个实用工具的效果：

part-1/chapter-6/api-syntax/responsive/index.xhtml

```js
<h3>All breakpoints</h3>
```

```js
<p class="font-weight-300">Lorem ipsum...</p>
```

```js
<p class="font-weight-400">Lorem ipsum...</p>
```

```js
<p class="font-weight-700">Lorem ipsum...</p>
```

```js
<h3>Breakpoint sm</h3>
```

```js
<p class="font-weight-sm-300">Lorem ipsum...</p>
```

```js
<p class="font-weight-sm-400">Lorem ipsum...</p>
```

```js
<p class="font-weight-sm-700">Lorem ipsum...</p>
```

```js
// and so on...
```

要看到这个功能的效果，请在浏览器中打开 HTML 文件并调整浏览器窗口的大小。

## RFS

此布尔选项将启用与 RFS Sass 插件一起的流体缩放。您将在*第十章**，使用 Bootstrap 5 和高级 Sass 及 CSS 功能*中了解更多关于此插件的信息。

## 打印

此布尔选项除了生成正常的实用工具类之外，还会为打印生成实用工具类。

让我们最后一次使用我们的示例：

part-1/chapter-6/api-syntax/print/scss/style.scss

```js
// Required imports
```

```js
// Utilities
```

```js
$utilities: (
```

```js
  "Font weight": (
```

```js
    property: font-weight,
```

```js
    values: 300 400 700,
```

```js
    print: true
```

```js
  )
```

```js
);
```

```js
// Utilities API
```

这将编译成以下内容：

part-1/chapter-6/api-syntax/print/css/style.css

```js
.font-weight-300 { font-weight: 300 !important; }
```

```js
.font-weight-400 { font-weight: 400 !important; }
```

```js
.font-weight-700 { font-weight: 700 !important; }
```

```js
@media print {
```

```js
  .font-weight-print-300 { font-weight: 300 !important; }
```

```js
  .font-weight-print-400 { font-weight: 400 !important; }
```

```js
  .font-weight-print-700 { font-weight: 700 !important; }
```

```js
}
```

我们现在可以通过使用以下 HTML 来看到这个实用工具的效果：

part-1/chapter-6/api-syntax/print/index.xhtml

```js
<p class="font-weight-print-300 ">Lorem ipsum...</p>
```

```js
<p class="font-weight-print-400 ">Lorem ipsum...</p>
```

```js
<p class="font-weight-print-700 ">Lorem ipsum...</p>
```

要看到这个功能的效果，请在浏览器中打开 HTML 文件，然后在浏览器菜单中选择**打印**。这将打开一个**打印**对话框，您可以在其中预览应用了打印样式的页面。

## RTL

此布尔选项指定实用工具是否应包含在编译样式的 RTL 版本中。这是 Bootstrap 5 的一个版本，它支持布局、组件和实用工具中的从右到左的文本。

## !important

使用工具 API 生成的所有工具类都添加了`!important`，以确保它们覆盖组件和修饰类的设计。可以通过将`$enable-important-utilities`选项设置为`false`来禁用此功能，这在*第四章*中提到过，*Bootstrap 5 全局选项和颜色*。

## 更多语法示例

要查看更多工具 API 语法的例子，您可以查看 Bootstrap 5 中的`_utilities.scss`文件。在这里，您将看到用于生成 Bootstrap 5 中所有默认工具类的语法。

# 使用工具 API

我们已经看到了工具 API 的语法，现在我们将看到一些如何用于不同目的的例子。首先，我们将看到如何生成和添加简单和复杂的新工具的例子，然后我们将看到如何覆盖、修改和删除现有工具的例子。为此，我们将使用以下两个 Sass 函数：`map-merge()`（合并现有映射）和`map-get()`（从指定的 Sass 映射中获取与键关联的值）。

## 添加一个简单工具

首先，我们将生成并添加一个简单的光标工具，它可以用来在悬停于特定元素时更改光标。我们指定了工具的名称（`"cursor"`），要使用的 CSS 属性（`cursor`），以及用于该 CSS 属性的值（`auto context-menu copy grab help pointer wait`）。这些是在使用工具 API 时必须指定的最小选项。

在使用工具 API 的语法之前，我们使用`map-merge()` Sass 函数将`$utilities`变量中的现有工具设置与这个新工具合并。这样做是为了避免覆盖`$utilities`变量并丢失所有默认工具。

代码看起来是这样的：

part-1/chapter-6/examples/add-simple-utility/scss/style.scss

```js
// Required imports
```

```js
// Utilities
```

```js
// Generate simple cursor utilities
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
    "cursor": (
```

```js
      property: cursor,
```

```js
      values: auto context-menu copy grab help pointer wait
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

```js
// Utilities API
```

这将编译所有默认工具，以及以下新工具：

part-1/chapter-6/examples/add-simple-utility/css/style.css

```js
.cursor-auto { cursor: auto !important; }
```

```js
.cursor-context-menu { cursor: context-menu !important; }
```

```js
.cursor-copy { cursor: copy !important; }
```

```js
.cursor-grab { cursor: grab !important; }
```

```js
.cursor-help { cursor: help !important; }
```

```js
.cursor-pointer { cursor: pointer !important; }
```

```js
.cursor-wait{ cursor: wait !important; }
```

我们现在可以通过使用以下 HTML 来看到这些工具的效果：

part-1/chapter-6/examples/add-simple-utility/index.xhtml

```js
<p class="cursor-auto">.cursor-auto</p>
```

```js
<p class="cursor-context-menu">.cursor-context-menu</p>
```

```js
<p class="cursor-copy">.cursor-copy</p>
```

```js
<p class="cursor-grab">.cursor-grab</p>
```

```js
<p class="cursor-help">.cursor-help</p>
```

```js
<p class="cursor-pointer">.cursor-pointer</p>
```

```js
<p class="cursor-wait">.cursor-wait</p>
```

要看到这个效果，请在一个浏览器中打开 HTML 文件，并用鼠标悬停在不同的元素上。

## 添加一个复杂工具

现在，这里有一个如何生成并添加一个具有许多选项的复杂不透明度工具的例子。我们指定了工具的名称（`"opacity"`），要使用的 CSS 属性（`opacity`），用于该 CSS 属性的值（作为 Sass 映射，以便类名可以与 CSS 值不同），要使用的类前缀（`o`），以及使此工具在`hover`上工作的状态，并且我们启用了`responsive`和`print`选项。

再次，我们使用`map-merge()`函数将现有工具与这个新工具合并。

代码看起来是这样的：

part-1/chapter-6/examples/add-complex-utility/scss/style.scss

```js
// Required imports
```

```js
// Utilities
```

```js
// Generate complex opacity utilities
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
    "opacity": (
```

```js
      property: opacity,
```

```js
      values: (
```

```js
        0: 0,
```

```js
        25: .25,
```

```js
        50: .5,
```

```js
        75: .75,
```

```js
        100: 1,
```

```js
      ),
```

```js
      class: o,
```

```js
      state: hover,
```

```js
      responsive: true,
```

```js
      print: true
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

```js
// Utilities API
```

这将编译所有默认工具，以及以下新工具：

part-1/chapter-6/examples/add-complex-utility/css/style.css

```js
.o-0 { opacity: 0 !important }
```

```js
.o-0-hover:hover { opacity: 0 !important }
```

```js
.o-25 { opacity: .25 !important }
```

```js
.o-25-hover:hover { opacity: .25 !important }
```

```js
.o-50 { opacity: .5 !important }
```

```js
.o-50-hover:hover { opacity: .5 !important }
```

```js
.o-75 { opacity: .75 !important }
```

```js
.o-75-hover:hover { opacity: .75 !important }
```

```js
.o-100 { opacity: 1 !important }
```

```js
.o-100-hover:hover { opacity: 1 !important }
```

```js
@media (min-width: 576px) {
```

```js
  .o-sm-0 { opacity: 0 !important }
```

```js
  .o-sm-0-hover:hover { opacity: 0 !important }
```

```js
  .o-sm-25 { opacity: .25 !important }
```

```js
  .o-sm-25-hover:hover { opacity: .25 !important }
```

```js
  .o-sm-50 { opacity: .5 !important }
```

```js
  .o-sm-50-hover:hover { opacity: .5 !important }
```

```js
  .o-sm-75 { opacity: .75 !important }
```

```js
  .o-sm-75-hover:hover { opacity: .75 !important }
```

```js
  .o-sm-100 { opacity: 1 !important }
```

```js
  .o-sm-100-hover:hover { opacity: 1 !important }
```

```js
}
```

```js
// and so on…
```

我们现在可以通过以下 HTML 来查看这些工具的效果：

part-1/chapter-6/examples/add-complex-utility/index.xhtml

```js
<h3>All breakpoints</h3>
```

```js
<p class="o-0">.o-0</p>
```

```js
<p class="o-0-hover">.o-0-hover</p>
```

```js
<p class="o-25">.o-25</p>
```

```js
<p class="o-25-hover">.o-25-hover</p>
```

```js
<p class="o-50">.o-50</p>
```

```js
<p class="o-50-hover">.o-50-hover</p>
```

```js
<p class="o-75">.o-75</p>
```

```js
<p class="o-75-hover">.o-75-hover</p>
```

```js
<p style="opacity: 0;" class="o-100">.o-100</p>
```

```js
<p style="opacity: 0;" class="o-100-hover">.o-100-hover</p>
```

```js
<h3>Breakpoint sm</h3>
```

```js
<p class="o-sm-0">.o-sm-0</p>
```

```js
<p class="o-sm-0-hover">.o-sm-0-hover</p>
```

```js
<p class="o-sm-25">.o-sm-25</p>
```

```js
<p class="o-sm-25-hover">.o-sm-25-hover</p>
```

```js
<p class="o-sm-50">.o-sm-50</p>
```

```js
<p class="o-sm-50-hover">.o-sm-50-hover</p>
```

```js
<p class="o-sm-75">.o-sm-75</p>
```

```js
<p class="o-sm-75-hover">.o-sm-75-hover</p>
```

```js
<p style="opacity: 0;" class="o-sm-100">.o-sm-100</p>
```

```js
<p style="opacity: 0;" class="o-sm-100-hover">
```

```js
 .o-sm-100-hover</p>
```

```js
// and so on...
```

要查看此功能的效果，请在一个浏览器中打开 HTML 文件并将鼠标悬停在不同的元素上。

## 覆盖一个工具

您可以通过使用相同的名称来覆盖现有的工具。此方法也可以用来修改工具，但那时您必须小心，并使用所有相同的选项。

以下是一个如何使用具有更多指定值的新的`"width"`工具来覆盖尺寸工具的示例：

Part-1/chapter-6/examples/overwrite-utility/scss/style.scss

```js
// Required imports
```

```js
// Utilities
```

```js
// Overwrite sizing utility for width
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
      class: w,
```

```js
      values: (
```

```js
        10: 10%,
```

```js
        20: 20%,
```

```js
        30: 30%,
```

```js
        40: 40%,
```

```js
        50: 50%,
```

```js
        60: 60%,
```

```js
        70: 70%,
```

```js
        80: 80%,
```

```js
        90: 90%,
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

```js
// Utilities API
```

这将编译所有默认工具，以及以下新工具：

part-1/chapter-6/examples/overwrite-utility/css/style.css

```js
.w-10 { width: 10% !important }
```

```js
.w-20 { width: 20% !important }
```

```js
.w-30 { width: 30% !important }
```

```js
.w-40 { width: 40% !important }
```

```js
.w-50 { width: 50% !important }
```

```js
.w-60 { width: 60% !important }
```

```js
.w-70 { width: 70% !important }
```

```js
.w-80 { width: 80% !important }
```

```js
.w-90 { width: 90% !important }
```

```js
.w-100 { width: 100% !important }
```

```js
.w-auto { width: auto !important }
```

我们现在可以通过以下 HTML 来查看这些工具的效果：

part-1/chapter-6/examples/overwrite-utility/index.xhtml

```js
<div class="w-10 bg-secondary text-white p-3 mb-2">.w-10
```

```js
</div>
```

```js
<div class="w-20 bg-secondary text-white p-3 mb-2">.w-20
```

```js
</div>
```

```js
<div class="w-30 bg-secondary text-white p-3 mb-2">.w-30
```

```js
</div>
```

```js
<div class="w-40 bg-secondary text-white p-3 mb-2">.w-40
```

```js
</div>
```

```js
<div class="w-50 bg-secondary text-white p-3 mb-2">.w-50
```

```js
</div>
```

```js
<div class="w-60 bg-secondary text-white p-3 mb-2">.w-60
```

```js
</div>
```

```js
<div class="w-70 bg-secondary text-white p-3 mb-2">.w-70
```

```js
</div>
```

```js
<div class="w-80 bg-secondary text-white p-3 mb-2">.w-80
```

```js
</div>
```

```js
<div class="w-90 bg-secondary text-white p-3 mb-2">.w-90
```

```js
</div>
```

```js
<div style="width: 50%;" class="w-100 bg-secondary 
```

```js
  text-white p-3 mb-2">.w-100</div>
```

```js
<div style="width: 50%;" class="w-auto bg-secondary
```

```js
  text-white p-3">.w-auto</div>
```

要查看此功能的效果，请在一个浏览器中打开 HTML 文件。

## 修改一个工具

可以修改现有的工具。在这个例子中，我们将`"margin-start"`工具的`class`前缀从`.ms-*`重命名为`.ml-*`。现在它将与 Bootstrap 4 中使用的相同前缀相同。这次，我们将使用`map-get()`函数从`$utilities`变量中获取`"margin-start"`属性的当前值，以便将其与`class: ml`属性合并，这将覆盖现有的`class: ms`属性。代码如下：

part-1/chapter-6/examples/modify-utility/scss/style.scss

```js
// Required imports
```

```js
// Utilities
```

```js
// Rename class for existing utility
```

```js
$utilities: map-merge(
```

```js
  $utilities, (
```

```js
    "margin-start": map-merge(
```

```js
      map-get($utilities, "margin-start"),
```

```js
      ( class: ml ),
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

```js
// Utilities API
```

这将编译所有默认工具，以及以下新工具：

part-1/chapter-6/examples/modify-utility/css/style.css

```js
.ml-0 { margin-left: 0 !important }
```

```js
.ml-1 { margin-left: .25rem !important }
```

```js
.ml-2 { margin-left: .5rem !important }
```

```js
.ml-3 { margin-left: 1rem !important }
```

```js
.ml-4 { margin-left: 1.5rem !important }
```

```js
.ml-5 { margin-left: 3rem !important }
```

```js
.ml-auto { margin-left: auto !important }
```

我们现在可以通过以下 HTML 来查看这些工具的效果：

part-1/chapter-6/examples/modify-utility/index.xhtml

```js
<div class="ml-1 bg-secondary text-white p-3 mb-2">.ml-1
```

```js
</div>
```

```js
<div class="ml-2 bg-secondary text-white p-3 mb-2">.ml-2
```

```js
</div>
```

```js
<div class="ml-3 bg-secondary text-white p-3 mb-2">.ml-3
```

```js
</div>
```

```js
<div class="ml-4 bg-secondary text-white p-3 mb-2">.ml-4
```

```js
</div>
```

```js
<div class="ml-5 bg-secondary text-white p-3 mb-2">.ml-5
```

```js
</div>
```

要查看此功能的效果，请在一个浏览器中打开 HTML 文件。重命名的`margin-left`工具类现在可以正常工作。

## 删除一个工具

可以从现有的工具集中删除一个工具。在这里，您可以看到如何通过将它们的值设置为`null`来删除`"user-select"`和`"pointer-events"`交互工具：

part-1/chapter-6/examples/remove-utility/scss/style.scss

```js
// Required imports
```

```js
// Utilities
```

```js
// Remove interaction utilities
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
    "user-select": null,
```

```js
    "pointer-events": null
```

```js
  )
```

```js
);
```

```js
// Utilities API
```

这将编译所有默认工具，除了`"user-select"`和`"pointer-events"`工具。

我们现在可以通过尝试以下 HTML 来查看这些工具确实已被删除：

part-1/chapter-6/examples/remove-utility/index.xhtml

```js
<h3>User select</h3>
```

```js
<h4>All</h4>
```

```js
<p class="user-select-all">Lorem ipsum...</p>
```

```js
<h4>Auto</h4>
```

```js
<p style="user-select: none;" class="user-select-auto">
```

```js
  Lorem ipsum...</p>
```

```js
<h4>None</h4>
```

```js
<p class="user-select-none">Lorem ipsum...</p>
```

```js
<h3>Pointer events</h3>
```

```js
<h4>None</h4>
```

```js
<p><a href="#" class="pe-none">Lorem ipsum dolor sit amet,
```

```js
  consectetur adipiscing elit</a>. In laoreet pellentesque
```

```js
  lorem sed elementum. Suspendisse maximus convallis ex.
```

```js
  Etiam eleifend velit leo.</p>
```

```js
<h4>Auto</h4>
```

```js
<p><a href="#" style="pointer-events: none;" 
```

```js
  class="pe-auto">Lorem ipsum dolor sit amet, consectetur
```

```js
    adipiscing elit</a>. In laoreet pellentesque lorem sed 
```

```js
    elementum. Suspendisse maximus convallis ex. Etiam 
```

```js
    eleifend velit leo.</p>
```

```js
<h4>None for parent, auto for second child</h4>
```

```js
<p class="pe-none"><a href="#">Lorem ipsum dolor sit amet,
```

```js
  consectetur adipiscing elit</a>. In laoreet pellentesque 
```

```js
  lorem sed elementum. Suspendisse maximus convallis ex.
```

```js
  <a href="#" class="pe-auto">Etiam eleifend velit leo</a>.
```

```js
</p>
```

要查看此功能的效果，请在一个浏览器中打开 HTML 文件。对于`"user-select"`和`"pointer-events"`的工具不再起作用。

# 摘要

我们现在已经了解了关于实用 API 的所有内容，这是一个内置在 Bootstrap 5 中的非常强大的工具。这也是关于自定义 Bootstrap 5 的最后一章，同时也是本书第一部分的最后一章。在本书的下一部分，我们将看到如何在构建一个完整的网站并使用 Sass 和一些自定义 JavaScript 以多种不同的方式对其进行定制时，如何将这些知识付诸实践。
