# 第六章：在 ECSS 中处理状态变化

在上一章中，我们考虑了项目组织以及如何理解和应用 ECSS 类命名约定。在本章中，我们将把重点转移到 ECSS 如何处理活动界面以及如何以一种合理和可访问的方式促进样式变化。

大多数 Web 应用程序需要处理状态。

首先让我们澄清一下我们所说的“状态”。考虑一些例子：

+   用户点击按钮

+   界面中的值已更新

+   界面的某个区域被禁用

+   界面中的小部件正在忙碌

+   输入的值超过了允许的值

+   应用程序的某个部分开始包含实时数据

所有这些情况都可以定义为状态变化。我们通常需要向用户传达的状态变化。因此，这些是需要传达给 DOM 的变化，随后我们的样式表需要一些合理的方式来满足这些需求。

我们如何以一致和深思熟虑的方式定义这些状态变化？

# ECSS 以前如何处理状态变化

在第三章中，我提到了我有多喜欢 SMACSS 方法来传达状态。例如：

```css

.has-MiniCartActive {}

```

表示在此节点上或下方的某处，*迷你购物车*处于活动状态。

另一个例子：

```css

.is-ShowingValue {}

```

这将传达组件或其中一个组件正在显示一些值（先前隐藏的）。

在历史上，这就是我在应用 ECSS 时传达状态的方式。我使用了微命名空间类，除了节点上的任何现有类来传达这个状态。例如：

![ECSS 以前如何处理状态变化](img/Warning-image-1.jpg)

```css

        .is-Suspended {} 
        .is-Live {} 
        .is-Selected {} 
        .is-Busy {}

```

在 DOM 中使用这些类的节点可能如下所示：

```css

<button class="co-Button is-Selected">Old Skool Button</button>

```

### 注意

从历史上看，在 DOM 中更改类，特别是在 DOM 的根附近，是不鼓励的。这样做会使渲染树无效，这意味着浏览器必须执行大量的重新计算。然而，情况正在改善。WebKit（用于 iOS 和 Safari 浏览器）的 Antii Koivisto 最近的工作意味着这样的更改现在几乎是最佳的。有兴趣的人可以在这里阅读有关类更改的 WebKit 变更集：[`trac.webkit.org/changeset/196383`](http://trac.webkit.org/changeset/196383)，以及属性选择器，例如`aria-*`在这里：[`trac.webkit.org/changeset/196629`](http://trac.webkit.org/changeset/196629)

# 转向 WAI-ARIA

然而，通过为另一本书研究 ARIA 一点（如果您感兴趣的话）,我发现如果这些信息纯粹是为了样式钩子而进入 DOM，它可能会在那里承担更多的重量。

这些相同的样式钩子实际上可以放在 DOM 中作为 WAI-ARIA（[`www.w3.org/TR/wai-aria/`](https://www.w3.org/TR/wai-aria/)）状态。WAI-ARIA 的[状态和属性](http://www.w3.org/TR/wai-aria/states_and_properties)部分描述了 W3C 标准化的方式，用于在应用程序中向辅助技术传达状态和属性。在 WAI-ARIA 的摘要描述的开头部分中，包含了这样的内容：

> 这些语义是为了让作者能够在文档级标记中正确传达用户界面行为和结构信息给辅助技术。

虽然规范旨在帮助通过辅助技术向残障用户传达状态和属性，但它也很好地满足了 ECSS 的需求。这是一个很好的结果！我们可以提高 Web 应用程序的可访问性，同时还获得了一个清晰定义、深思熟虑的词汇表，用于传达我们在应用程序逻辑中需要的状态。以下是使用 aria 重新编写的先前示例来传达状态：

```css

<button class="co-Button" aria-selected="true">Old Skool Button</button>

```

类处理按钮的美学。`aria-*`属性传达该节点或其后代的状态（如果有）。

在 JavaScript 应用程序领域，唯一需要的更改是从`classList`修改为`setAttribute`修改。例如，设置我们的`button`属性：

```css

button.setAttribute("aria-selected", "true");

```

### 提示

请注意，以这种方式分离关注点确实需要在 JavaScript 中添加一个*触点*。如果您绝对，绝对希望以最快/最简单的方式处理状态更改，那么使用`classList`更新一次将更快。

# ARIA 属性作为 CSS 选择器

在我们首选的 CSS 语法中，将该更改写在一组大括号内将如下所示：

```css
.co-Button {
    background-color: $color-button-passive;
    &[aria-selected="true"] {
        background-color: $color-button-selected;
    }
}

```

我们使用和符号（`&`）作为父选择器和属性选择器，以利用节点上的 aria 属性提供的增强特异性。然后我们可以根据需要对更改进行样式设置。

以这种方式嵌套状态更改在规则中提供了更高的开发人员人体工程学。意图是规则仅在整个应用程序样式中的根级别定义一次。这提供了一个真理的来源，以定义与该类相关的所有可能的情况。有关更多信息，请确保阅读第八章，*理智样式表的十诫*。

### 提示

作为相关说明，*CSS 选择器级别 4 规范*（[`drafts.csswg.org/selectors-4/#attribute-case`](https://drafts.csswg.org/selectors-4/#attribute-case)）通过在右方括号之前使用`i`标志来提供不区分大小写。例如：

`css .co-Button { background-color: $color-button-passive; &[aria-selected="true" i] { background-color: $color-button-selected; } }` 这允许通过属性的任何情况值变体（默认情况下它区分大小写）

## 使用 ARIA 重新设计状态和属性

WAI-ARIA 规范的这一部分描述了*Widget Attributes*，这些属性包含了在处理 Web 应用程序和快速更改数据时所需的许多常见状态。

以下是使用 ARIA 重新编写的本章开头给出的示例：

+   `aria-disabled="true"`（用于替代`is-Suspended`）

+   `aria-live="polite"`（用于替代`.is-Live`，礼貌值是*三个可能值*之一（[`www.w3.org/TR/wai-aria/states_and_properties#aria-live`](https://www.w3.org/TR/wai-aria/states_and_properties#aria-live)）用于描述应如何传达更新）

+   `aria-current="true"`（此版本目前提议用于 WAI-ARIA 1.1 [`www.w3.org/TR/wai-aria-1.1/#aria-current`](http://www.w3.org/TR/wai-aria-1.1/#aria-current)）

+   `aria-busy="true"`（用于替代`is-Busy`，表示元素及其子树（如果有）正在忙碌）

还有很多，根据 W3C 规范标准，规范很容易理解。

## 如果无法使用 ARIA

如果由于任何原因，您无法使用`aria-*`属性来在站点或应用程序中传达状态。我现在倾向于在不使用微命名空间来指定状态的情况下命名选择器。例如，而不是：

![如果无法使用 ARIA](img/Warning-image-1.jpg)

```css
 <button class="co-Button is-Selected">Old Skool
       Button</button>

```

我建议改用这样的选择器的变体版本：

```css

<button class="co-Button co-Button-selected">Old Skool Button</button>

```

这保持了模块的上下文，并仅指示正在应用此相同类的变体。

### 注意

您应该知道使用属性选择器来传达状态时存在一个*陷阱*。某些较旧版本的 Android（例如 Android 4.0.3 原生浏览器）在属性值更改时不会强制重新计算样式。这样做的结果是，依赖于属性的任何样式都不会动态工作（例如在 JavaScript 中切换时）。有两种可能的解决方法。首先，您可以在 DOM 的某个地方切换一个类，同时更改属性。或者，您可以通过在 CSS 中的某个地方列出每个空规则来启动属性选择器。甚至链接在一起也可以工作，例如`[aria-thing][aria-thing2]{}`。任何一种选择都会给程序添加一个不必要的复杂性。关于这种行为的错误报告可以在此处找到：[`bugs.webkit.org/show_bug.cgi?id=64372`](https://bugs.webkit.org/show_bug.cgi?id=64372)，提到的解决方法来自于这个 Stack Overflow 问题：[`stackoverflow.com/questions/6655364/css-attribute-selector-descendant-gives-a-bug-in-webkit/`](http://stackoverflow.com/questions/6655364/css-attribute-selector-descendant-gives-a-bug-in-webkit/)

# 总结

使用 WAI-ARIA 状态来传达 DOM 中的变化提供了与标准 HTML 类一样有用且易于使用的样式钩子。尽管这纯粹是个人偏好，但我也喜欢在样式表中使用完全不同的选择器来传达状态；这样在规则中更容易识别。

这些先前的因素都没有给你带来任何新东西。使用 WAI-ARIA 状态将几乎免费地开始为辅助技术用户提供更好的通信方式。如果金钱有话说，还要考虑，通过使用 WAI-ARIA，您可以将产品扩展到更多的用户（请参见下面的附加信息）。

因此，使用 WAI-ARIA 状态和属性是采用 ECSS 方法的项目中传达状态的推荐方式。

# 来自英国皇家国家盲人协会（RNIB）的附加信息和统计数据

*RNIB*（[`rnib.org.uk/`](http://rnib.org.uk/)）很友好地提供了一些关于英国盲人数量的数据。在为您的项目使用 ARIA 状态进行辩论/考虑时，这些数据可能会有所帮助。

+   在英国，有超过 84,000 名注册的盲人和部分视力受损的工作年龄人口（估计总人口约为 6400 万）。

然而，根据政府的劳动力调查，英国有大约 185,000 名工作年龄人口自报有*视力困难*。这包括那些视力损失不符合注册条件但仍足以影响他们日常生活的人。也包括那些不认为自己是残疾人的人。在这 185,000 人中：

+   113,000 人认为自己有*视力困难*并认为自己是长期残疾人

+   72,000 人认为自己有*视力困难*但不认为自己是残疾人

+   2011 年英国视力受损人口估计人数-1,865,900

+   2020 年英国预计患有视力受损的人数-2,269,700

+   2012 年英国成年糖尿病患者人数，这是视力受损的主要原因-3,866,980
