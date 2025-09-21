# *第十二章*：优化 Bootstrap 5 CSS 和 JavaScript 代码

在本章中，我们不会使用 Bootstrap 5 创建任何新的用户界面或自定义现有的界面。我们甚至不会查看 Bootstrap 5 文件中包含的任何源代码。相反，我们将看到在完成我们的网站创建后，我们如何优化我们的编译 Bootstrap 5 CSS 和 JavaScript 代码。

我们首先将专注于通过仅包含我们实际使用的 Bootstrap 5 Sass 部分来优化我们的样式表，并移除所有我们不使用的辅助工具和实用工具。然后我们将使用 Node.js、NPM 和 Laravel Mix 来设置一个构建过程来自动化一些任务，这将帮助我们捆绑我们组件实际使用的 JavaScript，并压缩我们的编译 CSS 和捆绑的 JavaScript。

在本章中，我们将涵盖以下主要主题：

+   仅包含使用组件的 Sass 部分

+   移除未使用的辅助工具和实用工具

+   使用 Node.js、npm 和 Laravel Mix 设置构建过程

+   使用模块打包器优化 JavaScript

+   压缩我们的编译 CSS 和捆绑的 JavaScript

# 技术要求

+   要预览示例，您需要一个代码编辑器和浏览器。所有代码示例的源代码都可以在这里找到：[`github.com/PacktPublishing/The-Missing-Bootstrap-5-Guide`](https://github.com/PacktPublishing/The-Missing-Bootstrap-5-Guide)。

+   要编译 Sass 到 CSS，您需要以下任何一个：

    +   **Node.js**，如果您更喜欢使用终端（Mac）或命令提示符（Windows）的**命令行界面**（**CLI**）

    +   **Scout-App**，如果您更喜欢**图形用户界面**（**GUI**）

    +   **Visual Studio Code**，如果您更喜欢使用 Visual Studio Code Marketplace 中的扩展

所有这些方法都在*第二章*中解释，*使用和编译 Sass*。

要使用 Laravel Mix，您需要在您的计算机上安装**Node.js**，然后按照说明从 npm 下载所需的包。

# 仅包含使用组件的 Sass 部分

在本节中，我们将看到如何通过仅包含我们 HTML 代码中实际使用的 Bootstrap 5 Sass 部分来优化我们的编译 CSS。在我们这本书中早期创建的网站上，我们使用了 Bootstrap 5 的所有组件，除了 Placeholder、Popover 和 Scrollspy 组件。Scrollspy 是一个仅使用 JavaScript 的组件，因此我们将移除 Placeholder 和 Popover 组件的 Sass 部分。在我们的`styles.scss`文件中，有一个长长的`// Optional Bootstrap CSS`列表，其中包含所有组件的 Sass 部分。所以，这就是我们需要仔细考虑可以移除哪些部分文件的地方。在下面的代码片段中，您将看到长长的`// Optional Bootstrap CSS`列表，其中导入 Placeholder 和 Popover 组件 Sass 部分的行已被注释掉。您也可以选择删除这些行，但通过将它们注释掉，如果稍后添加这些组件到网站上，很容易取消注释它们，并且当 Sass 编译时，它们将自动被移除：

part-3/chapter-12/website/scss/style.scss

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
// @import "../../../../bootstrap/scss/popover";
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
// @import "../../../../bootstrap/scss/placeholders";
```

当我们编译 Sass 代码时，编译后的 CSS 样式表现在会稍微小一些，因为 Popover 和 Placeholder 组件的 CSS 已经被移除。

Sass 部分之间的依赖关系

在各种 Sass 部分文件之间存在一些依赖关系，因此简单地移除特定组件的文件可能并不容易。请确保仔细测试您的网站。

# 移除未使用的辅助器和实用工具

Bootstrap 5 的辅助器和实用工具是我们除了组件之外使用的，有时是在组件的特定代码中，有时是我们自己的 HTML 代码的一部分。因此，也可以考虑我们实际使用了哪些，并确保不包含我们未使用的那些。

## 移除未使用的辅助器

在我们为网站编写的代码中，我们使用了以下辅助器：比例、位置、视觉隐藏和拉伸链接。这些辅助器的 Bootstrap 5 代码通过以下代码导入到我们的`style.scss`文件中：

part-3/chapter-12/website/scss/style.scss

```js
// Helpers
```

```js
@import "../../../../bootstrap/scss/helpers";
```

这将导入包含对单个辅助器`@import`语句的`_helpers.scss`文件。该文件的内容如下：

part-3/chapter-12/website/bootstrap/scss/_helpers.scss

```js
@import "helpers/clearfix";
```

```js
@import "helpers/colored-links";
```

```js
@import "helpers/ratio";
```

```js
@import "helpers/position";
```

```js
@import "helpers/stacks";
```

```js
@import "helpers/visually-hidden";
```

```js
@import "helpers/stretched-link";
```

```js
@import "helpers/text-truncation";
```

```js
@import "helpers/vr";
```

我们现在可能认为我们只需要取消注释我们不需要的辅助工具的`@import`语句，就像我们在上一节中对组件所做的那样。这在这里和现在会有效，但如果我们在以后选择更新 Bootstrap 5，我们的更改将会消失，因为`_helpers.scss`文件将被替换。这是因为这个文件是 Bootstrap 5 源代码的一部分，当你更新 Bootstrap 5 时会被替换。如果你知道你以后不会更新 Bootstrap 5，或者知道你会在更新后记得再次对`_helpers.scss`文件进行相同的编辑，你可以这样做：

part-3/chapter-12/website/bootstrap/scss/_helpers.scss

```js
// @import "helpers/clearfix";
```

```js
// @import "helpers/colored-links";
```

```js
@import "helpers/ratio";
```

```js
@import "helpers/position";
```

```js
// @import "helpers/stacks";
```

```js
@import "helpers/visually-hidden";
```

```js
@import "helpers/stretched-link";
```

```js
// @import "helpers/text-truncation";
```

```js
// @import "helpers/vr";
```

但为了以更好的和更可维护的方式来做这件事，我们可以在`styles.scss`文件中更改我们的`@import`语句，只导入我们需要的单个辅助部分文件，而不是导入`_helpers.scss`文件。所以，我们将忽略那个文件（即使它是 Bootstrap 5 源代码的一部分），而是实现我们自己的导入助手的方式。

代码将看起来像这样：

part-3/chapter-12/website/scss/style.scss

```js
// Helpers
```

```js
// @import "../../../../bootstrap/scss/helpers/clearfix";
```

```js
// @import "../../../../bootstrap/scss/helpers/colored-links";
```

```js
@import "../../../../bootstrap/scss/helpers/ratio";
```

```js
@import "../../../../bootstrap/scss/helpers/position";
```

```js
// @import "../../../../bootstrap/scss/helpers/stacks";
```

```js
@import "../../../../bootstrap/scss/helpers/visually-hidden";
```

```js
@import "../../../../bootstrap/scss/helpers/stretched-link";
```

```js
// @import "../../../../bootstrap/scss/helpers/text-truncation";
```

```js
// @import "../../../../bootstrap/scss/helpers/vr";
```

我选择保留我们文件中的未使用导入，但将它们注释掉。这样，在需要时更容易取消注释它们，而且无论如何，在编译 Sass 时它们会被移除，因此不会占用任何空间。就像我们处理组件部分文件的导入一样。

## 移除未使用的工具

要移除未使用的工具，我们需要采取与组件和辅助工具不同的方法。我们现在需要使用我们之前在*第六章*中学习的工具 API。在 Bootstrap 5 的`_utilities.scss`文件中，我们看到总共定义了 80 个具有不同设置和值的工具类。对于每一个，我们都可以看到类或类前缀将是什么，我们可以使用这个来搜索我们的代码，看看我们实际上使用了哪些。如果我们这样做，我们会发现我们大约使用了其中的一半。为了更精确，我们需要移除以下 37 个未使用的工具类：

```js
align, opacity, overflow, shadow, start, translate-middle, max-width, viewport-width, min-viewport-width, max-height, viewport-height, min-viewport-height, flex, flex-grow, flex-shrink, flex-wrap, gap, align-content, align-self, order, margin, margin-start, padding-x, padding-end, padding-start, font-family, word-wrap, text-opacity, bg-opacity, gradient, user-select, pointer-events, rounded-top, rounded-end, rounded-bottom, rounded-start, visibility
```

负边距工具（`negative-margin`、`negative-margin-x`、`negative-margin-y`、`negative-margin-top`、`negative-margin-end`、`negative-margin-bottom`和`negative-margin-start`）不包括在列表中，因为它们默认是禁用的，而且我们没有通过我们自己的 Sass 代码中的`$enable-negative-margins`选项启用它们。

为了移除所有未使用的工具，以便它们不会通过工具 API 生成，我们使用以下代码：

part-3/chapter-12/website/scss/_utilities.scss

```js
// Remove unused utilities
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
    "align": null,
```

```js
    "opacity": null,
```

```js
    "overflow": null,
```

```js
    "shadow": null,
```

```js
    "start": null,
```

```js
    "translate-middle": null,
```

```js
    "max-width": null,
```

```js
    "viewport-width": null,
```

```js
    "min-viewport-width": null,
```

```js
    "max-height": null,
```

```js
    "viewport-height": null,
```

```js
    "min-viewport-height": null,
```

```js
    "flex": null,
```

```js
    "flex-grow": null,
```

```js
    "flex-shrink": null,
```

```js
    "flex-wrap": null,
```

```js
    "gap": null,
```

```js
    "align-content": null,
```

```js
    "align-self": null,
```

```js
    "order": null,
```

```js
    "margin": null,
```

```js
    "margin-start": null,
```

```js
    "padding-x": null,
```

```js
    "padding-end": null,
```

```js
    "padding-start": null,
```

```js
    "font-family": null,
```

```js
    "word-wrap": null, 
```

```js
    "text-opacity": null,
```

```js
    "bg-opacity": null,
```

```js
    "gradient": null,
```

```js
    "user-select": null,
```

```js
    "pointer-events": null,
```

```js
    "rounded-top": null,
```

```js
    "rounded-end": null,
```

```js
    "rounded-bottom": null,
```

```js
    "rounded-start": null,
```

```js
    "visibility": null,
```

```js
  )
```

```js
);
```

编译 Sass 代码后，生成的 CSS 文件（在扩展的 Sass 输出格式中）现在将减少 1,513 行，这相当于节省了 25,485 字节。

### 指定使用的工具

优化工具使用的一种另一种方法是指定正在使用的工具。在我们的项目中总共使用了 39 个工具，我们可以通过使用 Bootstrap 5 在 `_functions.scss` 文件中提供的 `map-get-multiple()` Sass 函数来生成这些工具。使用此函数，我们可以从 Sass 映射中获取多个键，从而按照以下方式覆盖 `$utilities` 变量：

SCSS

```js
$utilities: map-get-multiple(
```

```js
  $utilities,
```

```js
  (
```

```js
    "float",
```

```js
    "display",
```

```js
    "position",
```

```js
    "top",
```

```js
    "bottom",
```

```js
    "end",
```

```js
    "border",
```

```js
    "border-top",
```

```js
    "border-end",
```

```js
    "border-bottom",
```

```js
    "border-start",
```

```js
    "border-color",
```

```js
    "border-width",
```

```js
    "width",
```

```js
    "height",
```

```js
    "flex-direction",
```

```js
    "justify-content",
```

```js
    "align-items",
```

```js
    "margin-x",
```

```js
    "margin-y",
```

```js
    "margin-top",
```

```js
    "margin-end",
```

```js
    "margin-bottom",
```

```js
    "padding",
```

```js
    "padding-y",
```

```js
    "padding-top",
```

```js
    "padding-bottom",
```

```js
    "font-size",
```

```js
    "font-style",
```

```js
    "font-weight",
```

```js
    "line-height",
```

```js
    "text-align",
```

```js
    "text-decoration",
```

```js
    "text-transform",
```

```js
    "white-space",
```

```js
    "color",
```

```js
    "background-color",
```

```js
    "rounded",
```

```js
    "utility",
```

```js
  )
```

```js
);
```

无论你选择指定要删除的工具还是正在使用的工具，结果都将相同。

## 剪切使用中的工具

如果你想要走得更远，也可以剪切任何使用中的工具。我的意思是，通过编辑该工具的值来删除我们不使用的工具的任何变体。

假设我们想要编辑 `text-align` 工具，因为我们项目中只使用了 `.text-center` 类。所以，我们想要移除 `.text-start` 和 `.text-end` 变体，以及禁用此工具的响应式变体。为了实现这一点，我们可以使用以下代码：

SCSS

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
    "text-align": map-merge(
```

```js
      map-get($utilities, "text-align"),
```

```js
      ( values: (
```

```js
          center: center,
```

```js
        ),
```

```js
        responsive: false
```

```js
      ),
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

现在，我们只生成 `.text-center` 类，这是我们用于文本对齐的唯一一个类。

或者，这也可以通过以下代码来完成，其中我们覆盖了文本对齐工具的代码，因此还必须指定 `property` 和 `class`：

SCSS

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
    "text-align": (
```

```js
      property: text-align,
```

```js
      responsive: false,
```

```js
      class: text,
```

```js
      values: (
```

```js
        center: center,
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

这只是一个例子。我没有在项目中使用这种方法来剪切使用中的工具，因为这是一项相当大的任务。但如果你走得更远，就有可能节省一些代码行和字节。

在下一节中，我们将使用 Node.js、npm 和 Laravel Mix 设置构建工具。我们将这样做是为了能够优化我们的 JavaScript 使用，并自动化 Sass 编译和一般压缩的任务。

# 使用 Node.js、npm 和 Laravel Mix 设置构建过程

在*第二章**，*使用和编译 Sass*中，我们已经学习了如何使用 Node.js 和 npm 设置一个脚本，该脚本将编译我们的 Sass 为 CSS。在这个设置中，我们将使用 Laravel Mix，它是围绕 Webpack 的包装器。简而言之，Webpack 是一个模块打包器，它为浏览器准备 JavaScript 和资源。通过使用 Laravel Mix，我们得到了一个简单的 API 来设置 Webpack 配置，而不需要任何先前的 Webpack 经验。

如果你还没有安装 Node.js，那么请返回并查看*第二章*和*编译 Sass*部分。现在，这里有一个使用 Node.js 和 npm 设置 Laravel Mix 的逐步指南。这将最终生成一个 `package.json` 文件、一个 Laravel Mix 配置文件、一个文件夹和一些空文件。

跳过设置 Laravel Mix

如果你想，你可以选择跳过 Laravel Mix 的手动设置，直接使用本书提供的 Laravel Mix 模板中的相关代码。然后，简单地进入下一节。

在我们开始设置 Laravel Mix 之前，我们必须打开终端（Mac）或命令提示符（Windows）并导航到我们的项目文件夹。然后，我们可以按照以下八个步骤进行操作：

1.  使用默认设置生成一个空的 npm 项目：

    ```js
    npm init -y
    ```

1.  安装 Bootstrap 5 的最新版本：

    ```js
    npm install bootstrap@5.2.0 --save-dev
    ```

使用 `--save-dev` 来指定这是一个开发依赖项。如果你想安装其他版本，请更改版本号。

1.  安装 Popper，它是 Dropdown、Popover 和 Tooltip 组件的依赖项：

    ```js
    npm install @popperjs/core --save-dev
    ```

1.  安装 Laravel Mix：

    ```js
    npm install laravel-mix --save-dev
    ```

1.  在项目的根目录中创建 `webpack.mix.js` 文件。这是 Laravel Mix 的配置文件。

1.  在项目的根目录中，创建一个包含空 `js` 和 `scss` 文件夹的 `src` 文件夹。在 `js` 文件夹中创建 `script.js` 文件，在 `scss` 文件夹中创建 `style.scss` 文件。文件和文件夹结构应如下所示：

    ```js
    src
    |- js
       |- script.js
    |- scss
       |- style.scss
    ```

我们将在这些文件中稍后添加一些代码。

1.  将以下代码添加到配置文件（`webpack.mix.js`）中，以导入 `mix` 并定义我们的构建：

    ```js
    let mix = require('laravel-mix');
    mix.js('src/js/script.js', 'js/').sass(
      'src/scss/style.scss', 'css/');
    ```

Laravel Mix 现在已被指示执行以下操作：

1.  将 `src/js/script.js` 中的 JavaScript 打包并输出到 `js` 文件夹。

1.  编译 `src/scss/style.scss` 中的 Sass 并将其输出到 `css` 文件夹。

1.  运行以下命令来安装额外的依赖项：

    ```js
    npx mix
    ```

此命令通常只需运行一次。

## 使用 Laravel Mix 模板

如果您没有按照手动步骤设置 Laravel Mix，您也可以从 `part-3/chapter-12/laravel-mix/template` 文件夹中获取 Laravel Mix 模板。在这种情况下，您只需将模板文件夹的内容放置到您的项目文件夹中，并运行以下命令：

```js
npm install
```

这将安装 Bootstrap 5、Popper 和 Laravel Mix，以及所有依赖项。您不需要运行 `npx mix` 命令来安装额外的依赖项。

## 使用 Laravel Mix

我们项目中的代码现在应该看起来像这样：

```js
src
```

```js
|- js
```

```js
   |- script.js
```

```js
|- scss
```

```js
   |- style.scss
```

```js
package-lock.json
```

```js
package.json
```

```js
webpack.mix.js
```

请注意，我们还没有在 `script.js` 和 `style.scss` 文件中放入任何代码，也没有向项目中添加任何 HTML 文件或其他资源。我们将在稍后进行。首先，让我们看看两个最有用的命令。

触发 Webpack 构建，我们在 `webpack.mix.js` 文件中定义了它，我们只需运行以下命令：

```js
npx mix
```

这将现在编译文件并构建包。

要监视文件系统中的更改并自动重新编译文件和重新构建包，我们可以使用以下命令：

```js
npx mix watch
```

每次我们在项目文件夹中的 JavaScript 或 Sass 文件保存更改时，Webpack 都会运行。

`script.js` 和 `style.scss` 文件中还没有添加任何代码

由于我们还没有在 `src` 文件夹中的 `script.js` 和 `style.scss` 文件中添加任何代码，Laravel Mix 的命令目前不会产生任何结果。我们现在将这样做。

## 将网站代码添加到构建过程中

由于我们的项目与 Laravel Mix 的使用方式不兼容，我们必须对其进行一些更改。因此，我们不会在 `part-3/chapter-12/website` 中的现有项目中做出更改，而是将其复制到 `part-3/chapter-12/laravel-mix/setup` 并在那里进行更改。请注意，本节中描述的所有更改已应用于 GitHub 上的代码。

为了重构和更新我们的项目，我们需要采取以下步骤：

1.  `src` **文件夹**:

首先，我们必须将我们的 `js` 和 `scss` 文件夹移动到新的 `src` 文件夹中，并删除 `bootstrap.bundle.min.js` 文件。Bootstrap 5 JavaScript 代码将由 Laravel Mix 编译和打包，然后放置在另一个文件夹中。

1.  **更新 HTML**:

由于我们在上一步中已删除 `bootstrap.bundle.min.js` 文件，我们现在可以从所有 HTML 文件中删除以下代码行：

HTML

```js
<script src="img/bootstrap.bundle.min.js"></script>
```

然后，在之前仅包含 `bootstrap.bundle.min.js` 文件而没有 `script.js` 文件的文件中，我们必须添加此文件，以便最终所有 HTML 页面都只包含以下 `script` 标签：

```js
<script src="img/script.js"></script>
```

因此，每个 HTML 文件的底部部分应该看起来像这样，但有少数例外：

```js
      </footer>
    <script src="img/script.js"></script>
  </body>
</html>
```

1.  **更新 Sass**:

现在，我们需要更新 `style.scss` 文件中 Bootstrap 5 Sass 部分的导入文件路径。我们需要将 `../bootstrap/scss/` 替换为 `~bootstrap/scss/`（或简单地用 `../` 替换 `~`），以确保直接从 `node_modules` 文件夹获取源代码。波浪字符 `~` 是对该文件夹的引用。

我们 `style.scss` 文件的上部现在应该看起来像这样：

part-3/chapter-12/laravel-mix/website/src/scss/style.scss

```js
// Required
@import "~bootstrap/scss/functions";
// Default variable overrides
@import "default-variable-overrides";
// Required
@import "~bootstrap/scss/variables";
// Variable value using existing variable
@import "variable-value-using-variable";
// Required
@import "~bootstrap/scss/maps";
@import "~bootstrap/scss/mixins";
@import "~bootstrap/scss/root";
```

我们将保持其余的 Sass 文件不变。

1.  **更新 JavaScript**:

当我们通过 `<script>` 标签加载 Bootstrap 5 时，我们在初始化 JavaScript 组件时使用 `bootstrap` 命名空间，如下所示：

JavaScript

```js
new bootstrap.component
```

为了将 Bootstrap 5 JavaScript 源代码导入以使其与我们的当前 JavaScript 代码兼容，我们必须在 `script.js` 文件的顶部以以下方式执行：

part-3/chapter-12/laravel-mix/setup/src/js/script.js

```js
import * as bootstrap from 'bootstrap';
```

这将导入所有 Bootstrap 5 JavaScript 源文件，这与我们之前在 `<script>` 标签中使用 `bootstrap.bundle.min.js` 文件时的情况类似。

现在，为了实际触发 Webpack 构建、编译 Sass 和打包 JavaScript，我们可以使用之前的 Laravel Mix 命令运行一次：

```js
npx mix
```

或者，我们可以使用以下命令在每次保存项目文件夹中 JavaScript 或 Sass 文件更改时运行它：

```js
npx mix watch
```

我们现在已经学会了如何使用 Node.js、npm 和 Laravel Mix 设置构建过程。然而，尽管我们没有使用所有 JavaScript 代码，但我们仍然导入了所有 JavaScript 代码。接下来，我们将看到如何优化 JavaScript 导入。

# 使用模块打包器优化 JavaScript

当使用模块打包器时，我们可以优化我们对 JavaScript 文件的使用。我们不必像之前那样导入所有 Bootstrap 5 的 JavaScript 源文件，而只需导入我们需要的文件。

在 Bootstrap 5 的源代码中，我们有一个 `bootstrap/js/dist` 文件夹，其中包含单个 JavaScript 组件的源代码，以及 `base-component.js` 文件和 `dom` 文件夹，其中包含一些与操作 DOM（页面文档对象模型）相关的一般代码。

我们可能会想象，我们只需在我们的 HTML 中的 `<script>` 标签中添加这些文件之一，如下所示：

HTML

```js
<script src="img/modal.js"></script>
```

然而，这不会起作用。我们将在浏览器控制台中收到错误。我们必须在我们的 `script.js` 文件中引用这些文件并使用模块打包器。

例如，如果我们只导入 `bootstrap/js/dist/modal.js` 文件，它也将导入 `bootstrap/js/dist/base-component.js` 文件以及 `bootstrap/js/dist/dom` 文件夹中所有与 DOM 相关的文件。

如果我们只导入 `bootstrap/js/dist/tooltip.js` 文件，它也将导入 `bootstrap/js/dist/base-component.js` 文件，`bootstrap/js/dist/dom` 文件夹中所有与 DOM 相关的文件，以及 Popper npm 包的所有必要文件，因为工具提示组件依赖于这个库进行定位。

在我们继续之前，请注意，我们不会对 `part-3/chapter-12/laravel-mix/setup` 中的代码进行更多修改，而是将其复制到 `part-3/chapter-12/laravel-mix/optimize-js` 并在那里进行修改。请注意，本节中描述的所有更改已应用于 GitHub 上的代码。

在我们的网站上，我们使用所有 JavaScript 组件，除了按钮、弹出框和滚动位置（然而，我们仍然使用按钮作为使用 HTML 和 CSS 的常规组件）。现在，为了优化 JavaScript 文件的使用，我们将向我们的 `script.js` 文件添加以下代码行，仅包括我们实际使用的组件：

part-3/chapter-12/laravel-mix/optimize-js/src/js/script.js

```js
import Alert from 'bootstrap/js/dist/alert';
```

```js
import Carousel from 'bootstrap/js/dist/carousel';
```

```js
import Collapse from 'bootstrap/js/dist/collapse';
```

```js
import Dropdown from 'bootstrap/js/dist/dropdown';
```

```js
import Modal from 'bootstrap/js/dist/modal';
```

```js
import Offcanvas from 'bootstrap/js/dist/offcanvas';
```

```js
import Tab from 'bootstrap/js/dist/tab';
```

```js
import Toast from 'bootstrap/js/dist/toast';
```

```js
import Tooltip from 'bootstrap/js/dist/tooltip';
```

如前所述，当我们通过 `<script>` 标签加载 Bootstrap 5 时，我们在初始化 JavaScript 组件时使用 `bootstrap` 命名空间，如下所示：

JavaScript

```js
new bootstrap.component
```

由于我们现在正在使用打包器，window 对象将不会被定义，因此我们必须以以下方式初始化组件：

JavaScript

```js
new component
```

因此，我们需要对我们的 `script.js` 文件进行一些修改。我们需要将所有使用的地方的 `new bootstrap.[component]` 代码替换为 `new [component]`。例如，在初始化所有工具提示的代码中，我们必须进行以下更改：

part-3/chapter-12/laravel-mix/optimize-js/src/js/script.js

```js
const tooltipTriggerElements = 
```

```js
  document.querySelectorAll('[data-bs-toggle="tooltip"]');
```

```js
for (let i = 0; i < tooltipTriggerElements.length; i++) {
```

```js
  new bootstrap.Tooltip(tooltipTriggerElements[i])
```

```js
}
```

现在，我们可以再次运行之前提到的相同的 Laravel Mix 命令。

请注意，我们还必须运行以下命令：

```js
npm install
```

如果它还没有运行过。此命令将创建 `node_modules` 文件夹并下载所有必要的依赖项（包括 Bootstrap 5 的源代码）到那里。

# 压缩我们的编译 CSS 和打包 JavaScript

为了进一步优化我们的代码，我们希望压缩编译后的 CSS 和打包的 JavaScript。我们可以通过运行以下命令使用 Laravel Mix 来实现：

```js
npx mix --production
```

`--production` 标志用于指示我们想要压缩编译和打包的文件。

# 摘要

我们现在已经学会了如何优化编译后的 Bootstrap 5 CSS 和 JavaScript 代码。首先，我们学习了如何通过仅包含我们实际使用的 Bootstrap 5 Sass 部分并移除所有未使用的辅助工具和实用工具来优化我们的样式表。然后，我们学习了如何使用 Node.js、npm 和 Laravel Mix 设置构建过程来自动化一些任务，这有助于我们仅打包我们的组件实际使用的 JavaScript，并压缩我们的编译 CSS 和打包 JavaScript。

尽管本章中描述的方法和技术不会改变网站的外观和感觉，但它们将优化您的最终代码，从而为用户提供更好的性能和更快的加载时间。

在本书中，我们学习了以各种新的方式使用 Bootstrap 5。从只知道关于默认样式下各种 Bootstrap 5 组件的 HTML 结构和类名，我们学习了如何更改全局选项和调色板，如何理解并导航 Sass 源代码，如何使用 Sass 变量自定义各种 Bootstrap 5 元素，以及如何使用强大的实用 API。在此基础上，我们还学习了如何使用更高级的 Sass、CSS 和 JavaScript 功能与 Bootstrap 5 结合，最后学习了如何优化编译后的 CSS 和 JavaScript（包括使用模块打包器）。现在，作为 Bootstrap 5 开发者，我们已经具备了解决更高级任务和更富有创意地使用 Bootstrap 5 源代码的能力。

感谢阅读！
