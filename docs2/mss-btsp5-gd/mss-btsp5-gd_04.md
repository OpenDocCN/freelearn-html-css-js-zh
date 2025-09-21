# *第三章*：下载和探索 Bootstrap 5 Sass 文件

在本章中，我们将首先看到两种不同的方式下载 Bootstrap 5 源代码，然后我们将更深入地查看源代码中的 Bootstrap 5 Sass 文件。我们将学习 Sass 代码是如何被分成各种文件，这些文件是如何结构的，以及这种结构意味着什么。

之后，我们将学习如何导入默认的 Bootstrap 5，最后，我们将了解 Bootstrap 5 的 Sass 变量。

在自定义样式之前熟悉 Bootstrap 5 的 Sass 文件和变量是有用的，因为您将知道在哪里搜索和查找满足您需求的特定变量。您还将了解某些变量和文件之间的依赖关系。

在本章中，我们将介绍以下主题：

+   下载 Bootstrap 5 源代码

+   探索 Bootstrap 5 的 Sass 文件

+   导入默认的 Bootstrap 5

+   探索 Bootstrap 5 变量

# 技术要求

+   为了预览示例，您需要一个代码编辑器和浏览器。

+   您需要一个 Sass 编译器来编译 Bootstrap 5 Sass 文件到 CSS。请参阅*第二章*，*使用和编译 Sass*，了解不同的方法。

您可以在 GitHub 上找到本章的代码文件，地址为[`github.com/PacktPublishing/The-Missing-Bootstrap-5-Guide`](https://github.com/PacktPublishing/The-Missing-Bootstrap-5-Guide)

# 下载 Bootstrap 5 源代码

您可以通过不同的方式下载 Bootstrap 5。在这里，我将向您展示三种方法。

## 从网站

您可以从[getbootstrap.com](http://getbootstrap.com)的官方 Bootstrap 网站，在**文档**部分的**入门**部分的**下载**页面下载 Bootstrap 文件：[getbootstrap.com/docs/5.2/getting-started/download](http://getbootstrap.com/docs/5.2/getting-started/download)。

从那个**下载**页面，您可以下载 Bootstrap 5 的编译版 CSS 和 JavaScript 文件，或者您可以下载源文件，其中包含所有的 Sass 代码、JavaScript 代码和文档文件。我们想要源文件，所以您应该点击紫色**下载源文件**按钮来下载一个包含我们所需所有内容的 ZIP 文件。

## 从 GitHub

Bootstrap 5 的源代码可以从官方 GitHub 仓库[github.com/twbs/bootstrap](http://github.com/twbs/bootstrap)下载。要从那里下载，您首先点击绿色的**代码**按钮来展开弹出窗口。从那里，您可以使用以下命令获取克隆它的 URL：

```js
git clone https://github.com/twbs/bootstrap.git
```

或者，您可以直接下载包含所有源代码的 ZIP 文件。

## 从 NPM

如果您在开发设置中使用 NPM，您可以使用以下命令下载和安装 Bootstrap 5 的最新版本：

```js
npm install bootstrap
```

它现在将被下载到你的项目的 `node_modules` 文件夹中。我们将在第十二章*优化 Bootstrap 5 CSS 和 JavaScript 代码*中看到如何使用 Node.js、NPM 和 Laravel Mix 设置构建过程。

现在我们已经下载了 Bootstrap 5，是时候看看 Sass 文件了。

# 探索 Bootstrap 5 Sass 文件

在本节中，我们将探索 Bootstrap 5 Sass 文件。这些文件位于下载的 Bootstrap 5 文件中的 `scss` 文件夹内。我们将查看此文件夹及其子文件夹的内容。

## 根目录

在 `scss` 文件夹的根目录中，我们总共有 41 个文件，其中 37 个是 Sass 部分文件，因为它们以下划线开头，剩下的 4 个文件是常规的 SCSS 文件。以下是每个文件的概述，每个文件旁边都有注释：

```js
_accordion.scss (component)
```

```js
_alert.scss (component)
```

```js
_badge.scss (component)
```

```js
_breadcrumb.scss (component)
```

```js
_button-group.scss (component)
```

```js
_buttons.scss (component)
```

```js
_card.scss (component)
```

```js
_carousel.scss (component)
```

```js
_close.scss (component)
```

```js
_containers.scss (layout)
```

```js
_dropdown.scss (component)
```

```js
_forms.scss (imports partials from the "forms" folder)
```

```js
_functions.scss (various custom Bootstrap 5 functions)
```

```js
_grid.scss (layout)
```

```js
_helpers.scss (imports partials from the "helpers" folder)
```

```js
_images.scss (content)
```

```js
_list-group.scss (component)
```

```js
_maps.scss (Sass maps for customization)
```

```js
_mixins.scss (imports partials from the "vendor" and 
```

```js
              "mixins" folder)
```

```js
_modal.scss (component)
```

```js
_nav.scss (component)
```

```js
_navbar.scss (component)
```

```js
_offcanvas.scss (component)
```

```js
_pagination.scss (component)
```

```js
_placeholders.scss (component)
```

```js
_popover.scss (component)
```

```js
_progress.scss (component)
```

```js
_reboot.scss (normalization of HTML elements)
```

```js
_root.scss (CSS custom properties)
```

```js
_spinners.scss (component)
```

```js
_tables.scss (content)
```

```js
_toasts.scss (component)
```

```js
_tooltip.scss (component)
```

```js
_transitions.scss (global transitions)
```

```js
_type.scss (content)
```

```js
_utilities.scss (settings for utilities)
```

```js
_variables.scss (Sass variables for customization)
```

```js
bootstrap-grid.scss (main SCSS file that imports grid and
```

```js
                     flex utilities only)
```

```js
bootstrap-reboot.scss (main SCSS file that imports reboot 
```

```js
                       styles only)
```

```js
bootstrap-utilities.scss (main SCSS file that imports 
```

```js
                          utilities only)
```

```js
bootstrap.scss (main SCSS file that imports everything)
```

如您从我的注释中看到的，大多数文件都与 Bootstrap 5 单个部分的样式相关：布局、内容和组件。以下是其他文件的快速说明。

### `_forms.scss`

此文件从 `forms` 文件夹导入所有 Sass 部分文件；它用于为各种表单元素提供样式。

### `_functions.scss`

此文件包含各种自定义 Bootstrap 5 函数。它主要用于评估变量、映射和混入之间的源代码，但除此之外，你还会找到 `tint-color()` 和 `shade-color()` 颜色函数，我们将在本章后面再次提到。

### `_helpers.scss`

此文件从 `helpers` 文件夹导入所有 Sass 部分文件；它用于为各种辅助类提供样式。

### `_maps.scss`

此文件包含重新分配的 Sass 映射，可以通过覆盖默认 Sass 映射来进行自定义。这将自动更新实用工具等。

### `_mixins.scss`

此文件从 `vendor` 和 `mixins` 文件夹导入所有 Sass 部分文件。`vendor` 文件夹包含 `mixins` 文件的样式，而 `mixins` 文件夹包含许多有用的混入（mixins），这些混入被其他文件在其他地方使用。我们将在第十章*使用 Bootstrap 5 与高级 Sass 和 CSS 功能*中了解更多关于如何充分利用 RFS Sass 插件和 Bootstrap 5 混入的方法。

### `_reboot.scss`

此文件包含用于标准化常见 HTML 元素样式的代码。它不是针对任何类，而是直接针对 HTML 元素。它是 **Normalize.css** 的分支，其中一些代码已被删除，而一些代码已被添加。Normalize.css 是一个重置样式表，它使浏览器更一致地渲染所有元素。更多关于它的信息可以在官方网站上找到：[necolas.github.io/normalize.css/](http://necolas.github.io/normalize.css/)。

### `_root.scss`

此文件用于生成根元素的 CSS 自定义属性，在这种情况下，意味着颜色和基本排版。它主要用于 `_reboot.scss` 文件，但也有一些在其他地方使用。

### `_transitions.scss`

此文件包含各种组件使用的过渡样式。

### _utilities.scss

此文件包含所有工具的设置。这些设置由工具 API 用于生成工具类。你将在*第六章*，*理解和使用 Bootstrap 5 工具 API*中了解更多关于工具 API 的内容。

### _variables.scss

此文件包含 Bootstrap 5 中跨文件使用的变量列表。你将在*第四章**，*Bootstrap 5 全局选项和颜色*，以及*第五章**，*定制各种 Bootstrap 5 元素*中了解更多关于此文件的内容。

### 主要 SCSS 文件

在所有的 Sass 部分文件之后，我们有四个主要的 SCSS 文件，它们用于导入 Bootstrap 5 的不同版本。以下是每个文件的快速概述以及它们导入的内容：

+   `bootstrap-grid.scss`: 仅包含网格系统和弹性工具

+   `bootstrap-reboot.scss`: 仅包含重置样式

+   `bootstrap-utilities.scss`: 仅包含辅助工具和工具

+   `bootstrap.scss`: 所有内容

在这四个文件中，我们只会使用 `bootstrap.scss` 来导入所有内容，但我们也会使用 `bootstrap.scss` 内部的修改版本来进行特定的定制和优化。

`scss` 文件夹还包含以下子文件夹：`forms`、`helpers`、`mixins`、`utilities` 和 `vendor`。这些文件夹包含更多的 Sass 部分文件。我已经在描述根文件夹中的 Sass 部分文件时提到了这些文件夹，所以不会进一步详细介绍。通常，所有 Sass 部分文件都应该保持不变，除非你有非常高级和具体的要求。我们在自己的文件中定制 Bootstrap 5，并按原样导入 Bootstrap 5 文件。

# 导入默认的 Bootstrap 5

在上一章中，我们学习了如何将 Sass 编译为 CSS。为了导入没有定制的默认 Bootstrap 5，我们只需将 `bootstrap.scss` 文件导入到我们自己的 SCSS 文件中，如下所示：

part-1/chapter-3/import-compile-default-bootstrap/scss/bootstrap.scss

```js
@import "../../../../bootstrap/scss/bootstrap";
```

在下一章中，我们将学习如何导入一个更高级的定制版本的 Bootstrap 5，其中我们导入 Bootstrap 5 的单个 SCSS 文件，而不是像前面代码中那样导入主文件。

在学习如何以不同的方式定制 Bootstrap 5 之前，我们将更仔细地查看 `bootstrap.scss` 文件。

## 探索 bootstrap.scss

`bootstrap.scss` 文件包含所有将导入各种元素的单个 Sass 部分文件的代码。现在让我们看看这个文件包含什么：

bootstrap.scss

```js
@import "mixins/banner";
```

```js
@include bsBanner("");
```

```js
// scss-docs-start import-stack
```

```js
// Configuration
```

```js
@import "functions";
```

```js
@import "variables";
```

```js
@import "maps";
```

```js
@import "mixins";
```

```js
@import "utilities";
```

```js
// Layout & components
```

```js
@import "root";
```

```js
@import "reboot";
```

```js
@import "type";
```

```js
@import "images";
```

```js
@import "containers";
```

```js
@import "grid";
```

```js
@import "tables";
```

```js
@import "forms";
```

```js
@import "buttons";
```

```js
@import "transitions";
```

```js
@import "dropdown";
```

```js
@import "button-group";
```

```js
@import "nav";
```

```js
@import "navbar";
```

```js
@import "card";
```

```js
@import "accordion";
```

```js
@import "breadcrumb";
```

```js
@import "pagination";
```

```js
@import "badge";
```

```js
@import "alert";
```

```js
@import "progress";
```

```js
@import "list-group";
```

```js
@import "close";
```

```js
@import "toasts";
```

```js
@import "modal";
```

```js
@import "tooltip";
```

```js
@import "popover";
```

```js
@import "carousel";
```

```js
@import "spinners";
```

```js
@import "offcanvas";
```

```js
@import "placeholders";
```

```js
// Helpers
```

```js
@import "helpers";
```

```js
// Utilities
```

```js
@import "utilities/api";
```

```js
// scss-docs-end import-stack
```

如您从前面的代码中可以看到，`bootstrap.scss` 文件将导入根 `scss` 文件夹中的所有 Sass 部分文件，而其中一些文件将导入子文件夹中的 Sass 部分文件。对于某些自定义，在定义自定义更改后导入此文件就足够了，但对于其他自定义，我们需要单独导入 Bootstrap 5 部分文件，并在这些导入之间添加我们自己的自定义更改。在接下来的章节中，我将向您展示何时需要导入整个 `bootstrap.scss` 文件或单独导入 Bootstrap 5 部分文件。

接下来，让我们简单谈谈 Bootstrap 5 的变量。

# 探索 Bootstrap 5 的变量

要自定义 Bootstrap 5 的任何元素，您将首先查看 `_variables.scss` 文件。此文件包含 Bootstrap 5 所使用的所有变量，包括全局选项、颜色、布局、内容（包括排版）、表单、组件、辅助工具和实用工具的变量。因此，例如，要了解组件的哪个部分可以自定义，您只需在 `_variables.scss` 文件中搜索组件名称。然后，您将找到所有相关变量及其默认值分组在一起。现在，您需要在自己的 Sass 文件中使用这些相同的变量名来更改它。您将在下一章中学习如何确切地做到这一点。

Bootstrap 团队已经仔细地排列了这些变量，因为一些变量引用了其他变量。不同组件的变量出现顺序不是按字母顺序排列的，因此我最好的建议是滚动浏览文件或搜索组件名称。

Bootstrap 5 使用了总共 886 个变量，其中一些是包含多个值的 Sass 映射。所有这些变量及其值都用于构建 Bootstrap 5 的不同元素，并且它们都可以进行自定义。

我们不会看到如何自定义 Bootstrap 5 所有元素（无论是布局、内容、表单、组件、辅助工具还是实用工具）的示例，但我们将从每个类别中通过足够的示例来学习如何更改其他所有内容。

查找变量名的一种替代方法

您还可以通过查看 `scss` 文件夹根目录下的部分文件来找到 Bootstrap 5 各个元素的变量名。例如，如果您进入 `_breadcrumb.scss` 文件，您可以看到该组件在不同 CSS 声明中使用的变量。在那里，您可以查看变量作为值被用于哪个属性，但通常，只需在 `_variables.scss` 文件中查找变量就足够简单、快捷且足够了。

## 带有 !default 标志的默认值

所有 Bootstrap 5 变量都使用`!default`标志。这个标志的功能在上一章的*默认值*子节中进行了描述。简而言之，`!default`标志用于在导入 Bootstrap 5 Sass 文件之前预先设置变量值。这就是 Bootstrap 5 定制工作的方式。

## 空值默认变量

在 886 个变量中，有 44 个具有`null`值。当这些变量在 Bootstrap 5 源代码的其他地方用作 CSS 声明时，该声明将无法编译。这是一种为用户提供更多定制选项的智能方式，而不会因为不必要的样式而膨胀代码库。如果用户选择在导入 Bootstrap 5 文件之前定义这些变量中的一个的值，那么`null !default`的值将被覆盖，CSS 声明将可以编译。

以第`591`行的`$headings-font-style`变量为例，它有一个`null !default`的默认值。这个变量在`_reboot.scss`文件中被用于一些标题的一般样式，如下所示：

```js
font-style: $headings-font-style;
```

当值为`null`时，这个 CSS 声明将无法编译。然而，如果我们给变量赋一个值，声明*将*可以编译。这样，标题将不会有任何特定的字体样式，但由于`$headings-font-style`变量已经可供开发者使用，因此可以很容易地赋予它们一个样式。

# 摘要

你现在已经学会了如何从官方网站或 NPM 下载 Bootstrap 5 源代码。你也了解了`scss`文件夹及其子文件夹中的 Bootstrap 5 Sass 文件和变量，以及这些文件和变量是如何相互关联的。最后，你学习了如何导入和编译 Bootstrap 5。

在下一章中，我们将开始通过关注全局选项和调色板来定制 Bootstrap 5 样式。
