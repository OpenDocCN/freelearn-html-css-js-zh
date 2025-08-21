# 第九章：ECSS 方法的工具

在这最后一章中，我们将看一些免费和开源的工具，以便编写合理和可维护的样式表。

在为持久项目编写 CSS 时，用于生成 CSS 的技术应该基本上是无关紧要的。我们应该始终意识到可能会有更好或更有效的工具可用来实现我们的目标，如果可能的话，应该加以采纳。

因此，无论是 Sass、PostCSS、LESS、Stylus、Myth 还是其他任何 CSS 处理器，都不应该成为编写样式表的障碍。如果需要的话，编写的样式表应尽可能容易迁移到另一种元语言。

此外，所采用的 CSS 处理器应该最好能满足整个项目的需求，而不仅仅是个别作者的偏好。也就是说，CSS 处理器应具备一些必要的功能，接下来我们将简要介绍这些功能。

# CSS 处理器的 CSS 要求

我认为 CSS 处理器对于样式表编写是必不可少的。这允许区分*编写*的样式表（作者在其选择的 CSS 处理器中编写的样式表）和*结果*的 CSS（编译和压缩后提供给用户的 CSS）。

尽管声明 CSS 处理器是必不可少的，但所需的功能相当微不足道：

+   **变量**：减少人为错误，如颜色选择和指定常量如网格尺寸

+   **部分文件**：为了方便作者编写与特性分支、模板或逻辑文件相对应的样式表

+   **颜色操作**：允许对上述变量进行一致的操作，例如能够调整颜色的 alpha 通道或轻松调整颜色

+   所有其他能力被认为是非必要的，应根据项目的特定需求进行评估

# 从编写的样式表构建 CSS

需要某种构建系统将编写的样式表编译成纯 CSS。

### 提示

有许多工具可用于执行此任务，例如 Grunt、Gulp 和 Brocolli 等。然而，就像没有普遍*正确*的 CSS 处理器或 CSS 方法论一样，也没有普遍*正确*的构建工具。

除了将编写的样式表编译成 CSS 之外，良好的工具还可以提供进一步的好处。

+   **Linting**：启用代码一致性并防止非工作代码达到部署

+   **Aggressive minification**：重新定位 z-index，将长度值转换为更小的长度值，例如（虽然*1pt*等同于*16px*，但字符数少了一个），合并相似的选择器

+   **Autoprefixer**：启用快速准确的供应商前缀，并防止供应商前缀出现在编写的样式表中

### 提示

对于样式表编写中被认为是必不可少的语法方面的考虑，请参阅第八章，“合理样式表的十诫”。

## 保存编译，ECSS 样式表的旅程

就工具而言，在撰写本文时，我目前使用 Gulp 和 PostCSS 以及其众多插件来编写 ECSS。这是一个运作良好的过程，所以我会在这里简要记录一下。

### 注意

对于非常好奇的人，可以在这里找到更多关于我从 Sass 到 PostCSS 的*经历*（[`benfrain.com/breaking-up-with-sass-postcss/`](https://benfrain.com/breaking-up-with-sass-postcss/)）。

样式表作者将样式表写入一个部分 CSS 文件（带有`*.css`文件扩展名），使用的语法与 Sass 非常相似。

在保存作者样式表时，Gulp watch 任务会注意到文件的更改，并首先运行 linting 任务。然后，如果一切正常，它会将部分作者样式表编译为 CSS 文件，然后自动添加前缀，最后 BrowserSync 将更改的 CSS 直接注入到我正在工作的网页中。通常，在我可以 *Alt* + *Tab* 到浏览器窗口之前，或者甚至在我从文本编辑器移动到浏览器窗口之前，都会创建一个源映射文件，因为一些作者发现在开发者工具中使用源映射更容易进行调试。所有这些都发生在我可以 *Alt* + *Tab* 到浏览器窗口之前，甚至在我可以从文本编辑器移动到浏览器窗口之前。

这是一个演示如何在基于 Gulp 的构建工具中设置 PostCSS 的 `gulpfile.js` 示例：

```css
//PostCSS related
var postcss = require("gulp-postcss");
var postcssImport = require("postcss-import");
var autoprefixer = require("autoprefixer");
var simpleVars = require("postcss-simple-vars");
var mixins = require("postcss-mixins");
var cssnano = require("cssnano");
var reporter = require("postcss-reporter");
var stylelint = require("stylelint");
var stylelinterConfig = require("./stylelintConfig.js");
var colorFunction = require("postcss-color-function");
var nested = require("postcss-nested");
var sourcemaps = require("gulp-sourcemaps");

// Create the styles
gulp.task("styles", ["lint-styles"], function () {

    var processors = [
        postcssImport({glob: true}),
        mixins,
        simpleVars,
        colorFunction(),
        nested,
        autoprefixer({ browsers: ["last 2 version", "safari 5", "opera 12.1", "ios 6", "android 2.3"] }),
        cssnano
    ];

    return gulp.src("preCSS/styles.css")

    // start Sourcemaps
    .pipe(sourcemaps.init())

    // We always want PostCSS to run
    .pipe(postcss(processors).on( class="st">"error", gutil.log))

    // Write a source map into the CSS at this point
    .pipe(sourcemaps.write())

    // Set the destination for the CSS file
    .pipe(gulp.dest("./build"))

    // If in DEV environment, notify user that styles have been compiled
    .pipe(notify("Yo Mofo, check dem styles!!!"))

    // If in DEV environment, reload the browser
    .pipe(reload({stream: true}));
});

```

对于 Gulp，构建选择是相当无限的，这只是一个示例。但是，请注意 `styles` 任务的第一步是运行 `lint-styles` 任务。

如前几章所述，样式表的 linting 是一个非常重要的步骤，特别是在涉及多个样式表作者的项目中。让我们接下来更深入地了解一下。

### Stylelint

Stylelint 是一个基于 Node 的静态分析样式表的 linting 工具。通俗地说，它会分析你的样式表，找出你特别关心的问题，并警告你任何问题。

### 提示

如果你使用 Sass，你应该查看 *scss-lint*（[`github.com/brigade/scss-lint`](https://github.com/brigade/scss-lint)），它为 Sass 文件提供了类似的功能。

如果发现任何作者错误，linting 任务会导致构建失败。通常情况下，在两个地方运行 linting 是最有益的。在文本编辑器（例如 Sublime）和构建工具（例如 Gulp）中。这样，如果作者有必要的文本编辑器，那么 *基于编辑器的 linting* ([`github.com/kungfusheep/SublimeLinter-contrib-stylelint`](https://github.com/kungfusheep/SublimeLinter-contrib-stylelint)) 就会在作者点击 *保存* 之前指出问题。

即使用户没有编辑器中的 linting 功能，保存时 linting 任务也会通过 Gulp 运行。构建步骤可以防止编译后的代码进入生产环境（因为持续集成软件也会导致构建失败）。

这是一个巨大的时间节省，对于代码的同行审查和质量保证测试来说是非常宝贵的。

这是一个 Stylelint 的 `.stylelintrc` 配置示例（这是针对 Stylelint 的 v5 版本的，所以未来/之前的版本可能会有些许不同）：

```css
{
    "rules": {
        "color-hex-case": "lower",
        "color-hex-length": "long",
        "color-named": "never",
        "color-no-invalid-hex": true,
        "font-family-name-quotes": "always-where-
        required",
        "font-weight-notation": "numeric",
        "function-comma-newline-before": "never-multi-
        line",
        "function-comma-newline-after": "never-multi-
        line",
        "function-comma-space-after": "always",
        "function-comma-space-before": "never",
        "function-linear-gradient-no-nonstandard-
        direction": true,
        "function-max-empty-lines": 0,
        "function-name-case": "lower",
        "function-parentheses-space-inside": "never",
        "function-url-data-uris": "never",
        "function-url-quotes": "always",
        "function-whitespace-after": "always",
        "number-leading-zero": "never",
        "number-no-trailing-zeros": true,
        "string-no-newline": true,
        "string-quotes": "double",
        "length-zero-no-unit": true,
        "unit-case": "lower",
        "unit-no-unknown": true,
        "value-keyword-case": "lower",
        "value-no-vendor-prefix": true,
        "value-list-comma-space-after": "always",
        "value-list-comma-space-before": "never",
        "shorthand-property-no-redundant-values": true,
        "property-case": "lower",
        "property-no-unknown": true,
        "property-no-vendor-prefix": true,
        "declaration-bang-space-before": "always",
        "declaration-bang-space-after": "never",
        "declaration-colon-space-after": "always",
        "declaration-colon-space-before": "never",
        "declaration-empty-line-before": "never",
        "declaration-block-no-duplicate-properties": true,
        "declaration-block-no-ignored-properties": true,
        "declaration-block-no-shorthand-property-
        overrides": true,
        "declaration-block-semicolon-newline-after":
        "always",
        "declaration-block-semicolon-newline-before":
        "never-multi-line",
        "declaration-block-single-line-max-declarations":
        1,
        "declaration-block-trailing-semicolon": "always",
        "block-closing-brace-empty-line-before": "never",
        "block-no-empty": true,
        "block-no-single-line": true,
        "block-opening-brace-newline-after": "always",
        "block-opening-brace-space-before": "always",
        "selector-attribute-brackets-space-inside":
        "never",
        "selector-attribute-operator-space-after":
        "never",
        "selector-attribute-operator-space-before":
        "never",
        "selector-attribute-quotes": "always",
        "selector-class-pattern": "^[a-z{1,3}-[A-Z][a-zA-Z0-9]+(_[A-Z][a-zA-Z0-9]+)?(-([a-z0-9-]+)?[a-z0-9])?$", { "resolveNestedSelectors": true }],
        "selector-combinator-space-after": "always",
        "selector-combinator-space-before": "always",
        "selector-max-compound-selectors": 3,
        "selector-max-specificity": "0,3,0",
        "selector-no-id": true,
        "selector-no-qualifying-type": true,
        "selector-no-type": true,
        "selector-no-universal": true,
        "selector-no-vendor-prefix": true,
        "selector-pseudo-class-case": "lower",
        "selector-pseudo-class-no-unknown": true,
        "selector-pseudo-class-parentheses-space-inside":
        "never",
        "selector-pseudo-element-case": "lower",
        "selector-pseudo-element-colon-notation":
        "single",
        "selector-pseudo-element-no-unknown": true,
        "selector-max-empty-lines": 0,
        "selector-list-comma-newline-after": "always",
        "selector-list-comma-newline-before": "never-
        multi-line",
        "selector-list-comma-space-before": "never",
        "rule-nested-empty-line-before": "never",
        "media-feature-colon-space-after": "always",
        "media-feature-colon-space-before": "never",
        "media-feature-name-case": "lower",
        "media-feature-name-no-vendor-prefix": true,
        "media-feature-no-missing-punctuation": true,
        "media-feature-parentheses-space-inside": "never",
        "media-feature-range-operator-space-after":
        "always",
        "media-feature-range-operator-space-before": "always",
        "at-rule-no-unknown": [true, {"ignoreAtRules": ["mixin"]}],
        "at-rule-no-vendor-prefix": true,
        "at-rule-semicolon-newline-after": "always",
        "at-rule-name-space-after": "always",
        "stylelint-disable-reason": "always-before",
        "comment-no-empty": true,
        "indentation": 4,
        "max-empty-lines": 1,
        "no-duplicate-selectors": true,
        "no-empty-source": true,
        "no-eol-whitespace": true,
        "no-extra-semicolons": true,
        "no-indistinguishable-colors": [true, {
            "threshold": 1,
            "whitelist": [ [ "#333333", "#303030" ] ]
        }],
        "no-invalid-double-slash-comments": true
    }
}
```

这只是一个示例，你可以从 *不断扩展的列表* ([`stylelint.io/user-guide/rules/`](http://stylelint.io/user-guide/rules/)) 中设置你关心的任何规则。如果是第一次使用这类工具，你可能还会发现下载/克隆 *ecss-postcss-shell* ([`github.com/benfrain/ecss-postcss-shell`](https://github.com/benfrain/ecss-postcss-shell)) 也很有用。这是一个基本的 Gulp 设置，用于通过 PostCSS 运行作者样式表，并使用 Stylelint 对样式进行 linting。

### 注意

我甚至为 Stylelint 项目贡献了一点代码，帮助添加了一个名为 `selector-max-specificity` 的规则，用于控制任何选择器的最大特异性级别。如果你参与控制 CSS 代码库，这是一个很好的项目可以参与。

如果这还不够，Stylelint 是可扩展的。很容易添加额外的功能。对于我工作中的当前构建 ECSS 项目，我们有额外的 Stylelint 规则：

+   确保只有覆盖和媒体查询可以嵌套（防止不使用父（`&`）选择器的嵌套）

+   确保关键选择器与 ECSS 命名约定匹配（Stylelint 现在有一个 `selector-class-pattern` 规则来帮助解决这个问题）

+   防止关键选择器成为复合选择器（例如 `.ip-Selector.ip-Selector2 {}`）

+   确保关键选择器是单数（例如 `.ip-Thing` 而不是 `.a-Parent .ip-Thing {}`）

这些提供了定制的质量保证，如果手动执行将会耗费大量时间并容易出错。

如果我没有表达清楚，我想让你知道我喜欢 Stylelint，并认为 linting 是大型 CSS 项目中不可或缺的工具，有多个作者。我简直无法推荐它。

### 注意

关于 Stylelint 还有更多信息，请参阅*这篇博客文章* ([`benfrain.com/floss-your-style-sheets-with-stylelint/`](https://benfrain.com/floss-your-style-sheets-with-stylelint/)) 或者官方的*Stylelint* ([`stylelint.io/`](http://stylelint.io/)) 网站。

# 优化

当 CSS 即将投入生产时，它需要通过*cssnano* ([`cssnano.co/`](http://cssnano.co/)) 进行额外的处理。这是一个由非常有才华的 Ben Briggs 开发的出色且模块化的 CSS 缩小器。强烈推荐。

除了 cssnano 提供的更明显的缩小步骤外，您还可以通过在 PostCSS 生态系统中使用插件执行一些微观优化。例如，通过一致地对 CSS 声明进行排序，Gzip 可以更有效地压缩样式表。这不是我想要手动完成的工作，但*postcss-sorting* ([`github.com/hudochenkov/postcss-sorting`](https://github.com/hudochenkov/postcss-sorting)) 插件可以免费完成。以下是使用各种声明排序配置的 Gzip 文件大小的比较。

举例来说，我拿了一个大型的测试 CSS 文件，未排序时 Gzip 后大小为 37.59 kB。这是同一个文件在使用其他声明排序配置后 Gzip 后的文件大小：

+   postcss-sorting: 37.54

+   CSSComb: 37.46

+   Yandex: 37.48

+   Zen: 37.41

所以，我们最多只能节省原始大小的不到 1%。虽然是微小的节约，但你可以免费有效地获得它。

还有其他一些类似的优化，比如将类似的媒体查询分组，但我会留下这些微观优化供您探索，如果您对它们感兴趣的话。

# 总结

在本章中，我们已经介绍了工具，以促进不断的代码质量和改进的样式表编写体验。然而，您应该知道，我们所涵盖的所有内容中，这里列出的具体工具可能是最短命的。工具技术发展迅速。仅仅三年时间，我从普通的 CSS，转到了 Sass（带有*scss-lint*（[`github.com/brigade/scss-lint`](https://github.com/brigade/scss-lint)）），再到了 PostCSS 和 Stylelint，同时也从 CodeKit 这样的 GUI 构建工具转到了 JavaScript 构建工具 Grunt，然后是 Gulp，现在是 NPM 脚本。

我不知道在 6 个月后最好的选择是什么，所以要记住的是要考虑工具和方法如何改进团队的样式表编写体验，而不是当前的工具是什么。

|   | *在你的个人关系中要忠诚，但在你选择的工具和技术上要多变* |   |
| --- | --- | --- |
|   | --*务实编码之道 ([`benfrain.com/be-better-front-end-developer-way-of-pragmatic-coding/`](https://benfrain.com/be-better-front-end-developer-way-of-pragmatic-coding/))* |

# 结束的右花括号

现在，朋友们，我们已经到达了这本小书的结尾。

虽然我希望你们中的一些人能够接受 ECSS 并开始全面实施它，但如果它只是激发了你自己的探索之旅，我同样会很高兴。

一开始，我试图找到一种处理以下问题的 CSS 扩展方法：

+   允许随着时间的推移轻松维护大型的 CSS 代码库

+   允许从代码库中删除 CSS 代码的部分，而不影响其余的样式

+   应该能够快速迭代任何新设计

+   更改应用于一个视觉元素的属性和值不应无意中影响其他元素

+   任何解决方案都应该需要最少的工具和工作流程更改来实施

+   在可能的情况下，应使用 W3C 标准，如 ARIA，来传达用户界面中的状态变化

ECSS 解决了所有这些问题：

+   将 CSS 分隔成模块可以轻松删除已弃用的功能

+   独特的命名约定避免了全局命名冲突，降低了特异性，并防止了对不相关元素的不必要更改

+   由于所有新模块都是*greenfield*，因此可以轻松构建新设计

+   尽管有一些工具可以适应全局导入和 linting，我们仍然在 CSS 文件中编写 CSS，这使得开发人员的入职过程变得更加容易

+   我们也可以接受 ARIA 作为控制和传达状态变化的手段，不仅仅是为了辅助技术，而且在更广泛的意义上也是如此

考虑到 CSS 的扩展是一种有点小众的追求。在未来，我们将拥有诸如*CSS Scoping* ([`www.w3.org/TR/css-scoping-1/#scope-atrule`](http://www.w3.org/TR/css-scoping-1/#scope-atrule))之类的东西，但在那之前，我们必须利用手头的工具和技术来弯曲现有技术以符合我们的意愿。

我已经多次提到过，有很多方法可以解决这个问题。其他方法可能更可取。以下是一些人和资源的列表，没有特定顺序，可能有助于你自己的探索。

亲爱的读者，直到下次，祝你探险愉快。

|   | *吸收有用的东西，拒绝无用的东西，添加特别属于你自己的东西。* |   |
| --- | --- | --- |
|   | --*李小龙* |

## 资源

以下是一些经常谈论或写作关于 CSS 架构/扩展的人：

+   Thierry Koblentz：[`cssmojo.com/`](http://cssmojo.com/)

+   Nicolas Gallagher：[`nicolasgallagher.com/`](http://nicolasgallagher.com/)

+   Kaelig Deloumeau-Prigent：[`kaelig.fr/`](http://kaelig.fr/)

+   Nicole Sullivan：[`www.stubbornella.org/content/`](http://www.stubbornella.org/content/)

+   哈里·罗伯茨：[`csswizardry.com/`](http://csswizardry.com/)

+   乔纳森·斯努克：[`snook.ca/`](https://snook.ca/)

+   Micah Godbolt：[`www.godbolt.me/`](http://www.godbolt.me/)

关于使用 JavaScript 内联样式的讨论：*Shop Talk show #180* ([`shoptalkshow.com/episodes/180-panel-on-inline-styles/`](http://shoptalkshow.com/episodes/180-panel-on-inline-styles/))

围绕 CSS 的有趣方法/项目：

+   React 的 Radium：[`github.com/FormidableLabs/radium`](https://github.com/FormidableLabs/radium)

+   React Native for Web：[`github.com/necolas/react-native-web`](https://github.com/necolas/react-native-web)

+   CSS 模块：[`github.com/css-modules`](https://github.com/css-modules)

+   原子 CSS：[`acss.io/`](http://acss.io/)
