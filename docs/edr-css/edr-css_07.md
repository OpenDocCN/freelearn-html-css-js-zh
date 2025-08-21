# 第七章：将 ECSS 应用于您的网站或应用程序

在本章中，我们将涵盖以下主题：

+   将 ECSS 应用于逻辑模块

+   将 ECSS 应用于视觉模块

+   组织模块、它们的组件和命名文件

+   使用 CMS 生成的内容

+   ECSS 和全局样式

ECSS 非常适用于复杂的 Web 应用程序。首先，让我们考虑如何在大型应用程序的逻辑周围应用 ECSS。

# 将 ECSS 应用于逻辑模块

通常，在 Web 应用程序中，某种编程语言（例如 JavaScript/TypeScript/Ruby/等）将生成*某个东西*。

使用该东西的文件名作为模块（或模块的组件）的名称通常是实用且可取的。因此，如果一个文件名为`Header.js`并生成了页眉的容器，那么页眉的任何组件部分都可以相应地命名。例如，在 ECSS 术语中，公司注册号可能会得到`sw-Header_Reg`作为其选择器。扩展一下，页眉内的搜索框组件可能具有类似`sw-HeaderSearch_Input`的选择器（由`HeaderSearch.js`文件创建的输入框）。

## 一个例子

让我们考虑一个更具体的例子。假设我们正在编写一个 JavaScript 客户端应用程序，并且有一个名为`ShoppingCartLines.js`的组件。它的任务是渲染出购物车中的行，然后显示在名为`ShoppingCart.js`的模块中。`ShoppingCart`模块渲染出与购物车本身有关的任何内容。到目前为止都很简单。

现在让我们通过建议，在某些情况下，我们的购物车将在模态视图中工作，在其他情况下，作为页面的一部分，正常文档流中，来使我们想象的情景稍微复杂化。

在这种情况下，我们有一个更广泛的模块：`ShoppingCart`和一个通常位于模块内的组件称为`ShoppingCartLines`。每个都将有自己的子节点。该模块和组件有两种可能的视图：在模态中和在页面中。让我们还想象一下，上下文的切换将由应用程序逻辑处理。

我们的常数是模块本身，我们可以使用命名空间为其提供上下文。在应用 ECSS 处理应用程序逻辑时，始终使用应用程序模块或组件的完整名称作为 ECSS 样式选择器的模块部分是有意义的。这样做的好处是，使 DOM 中的所有 HTML 类都能描述其来源和目的。

### 提示

在命名模块或组件的最外层容器的类时，不应该向类/选择器添加子扩展。只有模块或组件的子部分应该获得节点扩展。

好的，所以，目前为止，我们的选择器在样式表中可以这样命名：

```css
.mod-ShoppingCart {} /*Modal*/
.page-ShoppingCart {} /*Page*/
.mod-ShoppingCartLines {} /*Modal*/
.page-ShoppingCartLines {} /*Page*/
```

这样，我们的模块和组件通过命名空间切换隔离了它们的两个上下文。我们可以自由地根据需要为每个模块进行样式设置，而不会从一个模块泄漏到另一个模块。这是典型的情况，当组件和模块共享 HTML 类以实现抽象和重用时，通常会变得复杂。

让我们考虑一下这种情景的变化。假设我们不会在应用程序逻辑中切换上下文，而是在媒体查询中切换样式。在较小的视口上有一个模态实现，在较大的视口上有页面样式，正常文档流中。

在这种情况下，我们可以使用单个命名空间，例如`sc-ShoppingCart`（我使用`sc-`来指定上下文为`ShoppingCart`），并在 CSS 中使用媒体查询来提供视觉变化。

例如：

```css
.sc-ShoppingCart {
    /*Modal styles for smaller viewports*/
    @media (min-width: $M) {
        /* Page styles for larger viewports */
    }
}

.sc-ShoppingCartLines {
    /* Modal styles for smaller viewports */
    @media (min-width: $M) {
        /* Page styles for larger viewports */
    }
}
```

## 模块或组件的子节点

如前所述，模块或组件将有其自己的子节点元素。这些选择器应该使用子扩展进行命名。例如：

```css
.sc-ShoppingCart {
    /* The root of the component/module, no child
    extension needed */
}

.sc-ShoppingCart_Title {
    /* The 'title' child node of the Shopping Cart */
}

.sc-ShoppingCart_Close {
    /* A 'close' button child of the Shopping Cart for
    when the cart is modal */
}
```

每个子节点都会获得其父节点的命名空间和组件（或模块）名称。

### 提示

有关 ECSS 命名约定的详细信息，请参阅第五章，*文件组织和命名约定*。

因此，此时我们已经了解了在应用 ECSS 时如何命名我们的选择器，围绕应用模块和逻辑。现在我们将看看如何命名选择器并在纯粹的视觉模块周围应用 ECSS。但首先，关于使用类型选择器的一个简短但重要的离题。

## 关于类型选择器的说明

在编写 CSS 时，有时会诱人地使用类型选择器。通常情况下，这是在存在 HTML5 文本级元素时，比如`<i>`、`<b>`、`<em>`或`<span>`。例如，假设我们有一个句子，其中有几个需要加粗的单词。那么诱惑就会是这样做：

![关于类型选择器的说明](img/Warning-image-1.jpg)

```css
 <p class="ch-ShoppingCart_TextIntro">Here is the contents
 of your cart. You currently have <b>5 items</b>.</p>

```

并使用这些选择器来应用样式到`b`标签的内容：

![关于类型选择器的说明](img/Warning-image-1.jpg)

```css
 .ch-ShoppingCart_TextIntro { 
      /* Styles for the text */ 
      b { 
        /* Styles for the bold section within */ 
      } 
  }

```

这里有几个问题：

1.  我们已经对某些标记结构创建了依赖（它必须是一个子节点并且是一个`b`标签）。

1.  由于第 1 点，我们创建了一个比必要更具体的选择器。这使得任何未来的覆盖更难以理解和执行。

虽然这可能看起来过于冗长，但这就是应该处理该场景的方式：

```css
<p class="ch-ShoppingCart_TextIntro">Here is the contents of your cart. You currently have <b class="ch-ShoppingCart_TextIntroStrong">5 items</b>.</p>
```

并且这个 CSS：

```css
.ch-ShoppingCart_TextIntro {
    /* Styles for the text */
}

.ch-ShoppingCart_TextIntroStrong {
    /* Styles for the bold section within */
}
```

每个元素都有自己的选择器和规则。它们互不依赖。也不需要特定的标记来应用任何规则。

### 提示

应用于元素的每个规则都应尽可能地对其自身的外观持有意见。例如，如果您有一个包含两个文本节点的元素，似乎逻辑上将字体大小和行高应用于包装元素，以便两个文本节点将从中继承。但是，这会阻止该文本节点被移动到另一个位置并保持一致的渲染。相反，对每个节点应用颜色、字体大小和行高，即使它们最初非常相似（也许一开始只有颜色不同）。这起初似乎有违直觉，但可以防止未来可能的偏差（在 DOM 中移动，样式分歧等）。

# 将 ECSS 应用于视觉模块

*视觉*组件指的是不一定由特定应用逻辑生成的标记区域。

您仍然可以将区域分成逻辑**视觉**区域，并对其应用 ECSS。这是[`ecss.io`](http://ecss.io)网站采用的方法。

没有硬性规定。例如，我们可以将设计分为结构、菜单、页脚、导航、快速跳转菜单、主图等视觉区域。

在这种情况下，我们的选择器看起来像这样：

```css
.st-Header {
    /* Structural container for header */  
}

.st-Footer {
    /* Structural container for footer */
}
```

然而，我们也可以这样做：

```css
.hd-Outer {
    /* Structural container for header */ 
}

.ft-Outer {
    /* Structural container for footer */
}
```

或者如果是模块的话，也可以这样：

```css
.hd-Header {
    /* Structural container for the Header module */
}

.ft-Footer {
    /* Structural container for the footer module */
}
```

这些方法都没有对错之分。只要子节点/选择器遵循相同的命名约定，样式就会被隔离到特定区域。

事实是，在较小的网站上，您可以使用几乎任何类命名方法，碰撞的危险将是最小的。但是，一旦项目开始增长，命名空间和严格的命名约定的好处将开始丰厚地回报您。只需做出决定，并一致地应用该选择。

# 组织模块、它们的组件和命名文件

在这一点上，我认为考虑一个更详细的示例模块结构将是有用的。它类似于我习惯使用 ECSS 的结构。它比我们之前的例子更复杂，提供了另一种略有不同的文件组织和选择器命名变化。从我们的 CSS 角度来看，我们的目标是隔离、一致性和良好的开发人员人体工程学。让我们来看看。

假设我们有一个模块。它的工作是加载我们网站的侧边栏区域。目录结构可能最初看起来像这样：

```css
SidebarModule/ => everything SidebarModule related lives in here
  /assets => any assets (images etc) for the module
  /css => all CSS files
  /min => minified CSS/JS files
  /components => all component logic for the module in
  here
  css-namespaces.json => a file to define all namespaces
  SidebarModule.js => logic for the module
  config.json => config for the module
```

就示例标记结构而言，我们期望这个模块应该产生类似这样的东西：

```css
<div class="sb-SidebarModule">

</div>
```

样式化此初始元素的 CSS 应该放在`css`文件夹中，就像这样：

```css
SidebarModule/ 
  /assets 
  /css 
    /components 
    SidebarModule.css 
/min 
/components 
css-namespaces.json 
SidebarModule.js 
config.json
```

现在，假设我们在`SidebarModule`内有一个组件，它为`SidebarModule`创建一个标题。我们可能会将组件命名为一个名为`Header.js`的文件，并将其存储在`SidebarModule`的`components`子文件夹中，就像这样：

```css
SidebarModule/ 
  /assets 
  /css
    /components 
    SidebarModule.css 
/min 
/components 
  Header.js 
css-namespaces.json 
SidebarConfig.js 
SidebarModule.js 
config.json
```

有了这个，`Header.js`可能会呈现如下标记：

```css
<div class="sb-SidebarModule">
    <div class="sb-Header">
        <div class="sb-Header_Logo"></div>
    </div>
</div>

```

注意`Header`组件，由于在`SidebarModule`的上下文中，携带`sb-`微命名空间来指定其父级。并且由这个新组件创建的节点根据创建它们的逻辑进行命名。

就一般约定而言：

组件应携带原始逻辑的微命名空间。如果您正在创建一个位于模块内的组件，它应携带原始模块的命名空间（模块的可能命名空间在`css-namespaces.json`中定义）。

HTML 类/CSS 选择器应根据生成它们的文件名/组件进行命名。例如，如果我们在我们的模块内创建了另一个名为`HeaderLink.js`的组件，它将在`Header.js`组件的子级内呈现其标记，那么它生成的标记和适用的 CSS 选择器应与此文件名匹配。

例如：

```css
<div class="sb-SidebarModule">
    <div class="sb-HeaderPod">
        <div class="sb-HeaderPod_Logo"></div>
    </div>
    <div class="sb-HeaderPod_Nav">
        <div class="sb-HeaderLink">Node Value</div>
        <div class="sb-HeaderLink">Node Value</div>
        <div class="sb-HeaderLink">Node Value</div>
        <div class="sb-HeaderLink">Node Value</div>
    </div>
</div>
```

就文件夹结构而言，现在看起来是这样的：

```css
SidebarModule/ 
  /assets 
  /css 
    /components 
      Header.css 
      HeaderLink.css 
    SidebarModule.css 
  /min 
  /components 
    Header.js
    HeaderLink.js 
  css-namespaces.json 
  SidebarConfig.js 
  SidebarModule.js
  tsconfig.json
```

注意组件逻辑（`*.js`文件）和相关样式（`*.css`文件）之间存在一对一的对应关系-两者都位于`components`子文件夹中。尽管逻辑和样式不共享相同的直接父文件夹，但它们都位于同一个模块文件夹中，如果需要，可以轻松删除整个模块。

## 组件内的节点

总之。以这种方式使用，组件内节点的 ECSS 命名约定应始终是：

```css
ns-Component_Node-variant
```

+   `ns`：微命名空间（始终小写）

+   `-Component`：组件名称（始终使用大驼峰命名法）

+   `_Node`：组件的子节点（始终大驼峰，前面有下划线）

+   `-variant`：节点的可选变体（始终小写，并在连字符之前）

### 变体

请注意，组件内节点的`-variant`部分是可选的，只应用于表示细微差异的项目。例如，除了不同的背景图像之外完全相同的多个标题可能会呈现如下：

```css
<div class="sb-Classification_Header sb-Classification_Header-2"></div>
```

请记住，我们在第五章中更详细地讨论了变体选择器，*文件组织和命名约定*。

# 从 CMS 生成的内容中工作

很可能，如果您使用 ECSS 与任何类型的内容管理系统（Wordpress、Ghost、Drupal 等），您将遇到一种情况，即不可能向每个元素添加类。例如，在 Wordpress 页面或帖子中，期望用户输入内容并记住要添加到每个段落标记的正确类是不现实的。在这些情况下，我认为务实必须取胜。

为封闭元素设置一个 ECSS 类，并（勉强）接受所有嵌套元素都将使用类型选择器进行设置。以下是一些示例标记：

```css
<main class="st-Main">
    <h1>How to survive in South Central?</h1>
    <p>A place where bustin' a cap is fundamental. </p>
    <ul>
        <li>Rule number one: get yourself a gun. A nine in yo' ass'll be fine</li>
        <li>Rule number two: don't trust nobody.</li>
    </ul>    

</main>
```

以下是您可能编写的 CSS 来处理选择这些元素的方式：

```css
.st-Main {
    h1 {
        /* Styles for h1 */
    }
    p {
        /* Styles for p */
    }
    ul {
        /* Styles for ul */
    }
    li {
        /* Styles for li */
    }
}
```

我对此并不疯狂。我们正在嵌套选择器，将我们的样式与元素绑定，基本上是我们通常希望避免的 ECSS。但是，我很诚实。现实情况是，这可能是我们能够做出的最好妥协。在可能向元素添加类的情况下，我们绝对应该这样做。但是，会有一些情况下这根本不可能，任何象牙塔的理想主义都无法在这些情况下帮助。记住*平青哥*！

# ECSS 和全局样式

虽然网页应用中大部分的 CSS 可以描述为基于模块的，但我们需要处理的全局 CSS 也是不可避免的。从 ECSS 的角度来看，我们应该尽量保持全局 CSS 的最小化。通常，除了任何必需的*重置*样式之外，还会有默认的字体大小、字体系列，也许还有一些默认的颜色。这些样式通常应用于类型选择器。除非你在根 HTML 元素或者 body 上有类。

### 注意

如果你正在寻找一个网页应用的基本重置样式集，你可能会发现我的*App Reset* CSS 很有用。你可以在 GitHub 上找到它：[`github.com/benfrain/app-reset`](https://github.com/benfrain/app-reset)，或者通过 NPM 安装`npm install app-reset`。

可能还需要一些全局结构。例如，如果你的应用程序中有一个常见的结构（头部、页脚、侧边栏等），你可能希望创建一些选择器来反映这一点。过去，我曾经使用`.st-`或`.sw-`微命名空间来定义*结构*或*站点范围*，但你可以使用最适合你的任何东西。然而，我的建议是，这些选择器实际上不应该有很多，因为这些通常涉及到应用程序的所有模块应该存在的非常广泛的领域。

在组织全局 CSS 方面，我目前更倾向于在任何项目的根目录下创建一个名为`globalCSS`的文件夹。在那个文件夹里会有任何变量、混合、全局图像资源、任何字体或图标字体文件，一个基本的 CSS 重置文件和任何需要的全局 CSS。

# 总结

在本章中，我们已经看过了你可能在项目中应用 ECSS 的两种主要方式。我们还考虑了一个完整和更复杂的模块可能的文件夹结构。我希望到这一点，你已经对如何在你的项目中应用 ECSS 有了一个大致的想法。

与实施 CSS 的架构方法相辅相成的是实际编写样式表的实践。你知道，在编辑器中代码实际上是什么样子的。本书中的代码示例一直在演示这种语法，但现在是时候更详细地深入研究了。

如何最好地编写样式表来实践所有这些 ECSS 的花哨东西，这将是我们在下一章中要讨论的内容。
