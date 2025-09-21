

# 实现样式和主题

没有样式的情况下，组件内的 `h1` 元素将与其他组件中的 `h1` 元素看起来相同。Svelte 允许您使用 **层叠样式表** (**CSS**)，这是一种用于样式化和格式化网页内容的语言，来设置您的元素样式，使它们具有不同的外观和感觉。

在本章中，我们将首先讨论不同的 Svelte 组件样式设置方法。然后，我们将看到一些示例，包括将流行的 CSS 框架 **Tailwind CSS** 集成到 Svelte 中。

接下来，我们将讨论主题。当您在 Svelte 组件中一致地应用一组样式时，您将在组件中看到整体样式主题。我们将讨论如何跨组件同步样式，以及如何允许组件用户自定义样式。

到本章结束时，您将学会各种样式设置方法，并能够根据场景选择合适的方案和适用正确的方法。

本章包括以下内容：

+   设置 Svelte 组件样式的多种方法

+   使用 Tailwind CSS 设置 Svelte 组件样式的多种方法

+   将主题应用到 Svelte 组件

# 技术要求

您可以在 GitHub 上找到本章使用的代码：[`github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter02`](https://github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter02)。

# 以六种不同的方式设置 Svelte 组件的样式

在 Svelte 组件中，您有定义结构和内容的元素。通过样式设置，您可以改变元素的外观和感觉，超出浏览器的默认设置。

Svelte 组件可以通过六种不同的方式来设置样式。让我们来探索在 Svelte 组件内部应用样式的不同方法。

## 使用 `style` 属性进行样式设置

首先，您可以通过 `style` 属性向元素添加内联样式：

```js
<div style="color: blue;" />
```

前面的代码片段将 `div` 内文本的颜色变为蓝色。

与 HTML 元素中的 `style` 属性类似，您可以在 `style` 属性中添加多个 CSS 样式声明：

```js
<div style="color: blue; font-size: 2rem;" />
```

在 Svelte 中添加多个 CSS 样式声明的语法与在 HTML 中做的方式相同。在前面的代码片段中，我们将 `div` 内的文本颜色改为蓝色，并设置为 2 rem 大小。

`style` 属性的值是一个字符串。您可以使用动态表达式来构建 `style` 属性：

```js
<div style="color: {color};" />
```

当您在 `style` 属性中拥有多个 CSS 样式声明时，有时会变得杂乱无章：

```js
<div style="color: {color}; font-size: {fontSize}; background: {background}; border-top: {borderTop};" />
```

## 使用 `style:` 指令

Svelte 提供了 `style:` 指令，允许您将 `style` 属性拆分为多个属性，在添加换行和缩进后，这可能会使代码更易于阅读：

```js
<div
  style:color={color}
  style:font-size={fontSize}
  style:background={background}
  style:border-top={borderTop}
/>
```

`style:` 指令遵循以下语法：

```js
style:css-property-name={value}
```

CSS 属性名称可以是任何 CSS 属性，包括 CSS 自定义属性：

```js
<div style:--main-color={color} />
```

如果样式名称与它所依赖的值名称相匹配，那么您可以使用 `style:` 指令的简写形式：

```js
<div
  style:color
  style:font-size={fontSize}
  style:background
  style:border-top={borderTop}
/>
```

在 `style:` 指令中声明的样式具有比 `style` 属性更高的优先级。在下面的例子中，`h1` 文本颜色是红色而不是蓝色：

```js
<div style:color="red" style="color: blue;" />
```

除了逐个为元素添加内联样式来样式化元素外，接下来的方法允许我们编写 CSS 选择器来针对多个元素并将它们一起样式化。

## 添加 `<style>` 块

在每个 Svelte 组件中，你可以有一个顶级 `<style>` 块。

在 `<style>` 块内，你可以有 CSS 规则，针对组件内的元素：

```js
<div>First div</div>
<div>Second div</div>
<style>
  div {
    color: blue;
  }
</style>
```

当你想要在组件内的多个元素上应用相同的样式时，这种方法很有用。

在前面的代码中，CSS 规则会将文档中的所有 `div` 元素都变成蓝色吗？

不。`<style>` 块内的 CSS 规则的作用域限定在组件内，这意味着 CSS 规则只会应用于组件内的元素，而不会应用于应用中的其他地方的 `div` 元素。

因此，你不必担心 `<style>` 块内的 CSS 规则会改变组件外的元素。

但是，这是如何工作的？Svelte 如何确保 CSS 规则只应用于同一组件内的元素？

让我们探索一下。

### Svelte 如何作用域组件内的 CSS 规则

让我们稍微偏离一下，来理解 Svelte 如何确保组件内的 CSS 规则是作用域限定的。

当 Svelte 编译一个组件时，Svelte 编译器会遍历每个 CSS 规则，并尝试将每个元素与 CSS 规则的选择器匹配：

```js
<div>row</div>
<style>
  div { color: red; }
</style>
```

当一个元素匹配选择器时，Svelte 编译器将为组件生成一个唯一的 CSS 类名并将其应用于该元素。同时，Svelte 通过在选择器中包含生成的类名的类选择器来限制被 CSS 规则选择的元素的作用域。

元素和 CSS 规则的转换看起来像这样：

```js
<div class="svelte-q5jdbb">row</div>
<style>
  div.svelte-q5jdbb { color: red; }
</style>
```

在这里，`"svelte-g5jdbb"` 是由计算 CSS 内容的哈希值生成的唯一 CSS 类名。当 CSS 内容发生变化时，哈希值也会不同。由于 Svelte 编译器只将 CSS 类名应用于组件内的元素，因此不太可能将样式应用于组件外的其他元素。

这种转换默认发生在编译过程中。你不需要做任何额外的事情。这里的例子只是为了说明目的。

了解 `<style>` 块内的 CSS 规则是作用域限定的，当你想要将样式应用于具有相同节点名的所有元素时，使用 CSS 类型选择器作为你的 CSS 规则通常就足够了。然而，如果你只想样式化某些元素，比如在上一个例子中只样式化第二个 `div` 元素，你可以在第二个 `div` 元素上添加一个 `class` 属性来区分其他 `div` 元素。

是的，添加一个 `class` 属性是我们样式化 Svelte 组件的另一种方式。

## 添加 `class` 属性

CSS 类选择器在 Svelte 中的工作方式与在 CSS 中相同。

当你向元素添加 `class` 属性时，你可以使用 CSS 类选择器来定位它。

在下面的示例中，我们将 `highlight` 类添加到了第二个 `div` 元素上，因此只有第二个 `div` 元素具有黄色背景：

```js
<div>First div</div>
<div class="highlight">Second div</div>
<style>
  .highlight {
    background-color: yellow;
  }
</style>
```

`class` 属性的值可以是字符串或动态表达式。

你可以条件性地将类应用到元素上：

```js
<div class="{toHighlight ? "highlight" : ""} {toBold ? "bold" : ""}" />
```

在前面的示例中，当 `toHighlight` 和 `toBold` 的值都为 `true` 时，`class` 属性的值计算为 `"highlight bold"`。因此，两个类，`highlight` 和 `bold`，被应用到 `div` 元素上。

这种根据变量条件性地将类应用到元素上的模式非常常见，因此 Svelte 提供了 `class:` 指令来简化它。

## 使用 class: 指令简化类属性

在前面的示例中，我们当 `toHighlight` 变量为真值时条件性地应用了 `highlight` 类，当 `toBold` 变量为真值时应用了 `bold` 类。

这可以通过 `class:` 指令来简化，如下所示：

```js
class:class-name={condition}
```

使用 `class:` 指令简化前面的示例，我们得到以下代码：

```js
<div class:highlight={toHighlight} class:bold={toBold} />
```

就像 `style:` 属性一样，如果类的名称与条件变量的名称相同，你可以进一步简化为 `class:` 指令的缩写形式。

如果添加 `highlight` 类的条件是一个名为 `highlight` 的变量，那么前面的示例可以重写如下：

```js
<div class:highlight />
```

将所有这些放在一起，在下面的代码示例中，当 `highlight` 变量值为 `true` 时，`div` 元素具有黄色背景，否则具有透明背景：

```js
<script>
  export let highlight = true;
</script>
<div class:highlight />
<style>
  .highlight {
    background-color: yellow;
  }
</style>
```

我们迄今为止探索的所有将样式应用到元素的方法都涉及在 Svelte 组件内编写 CSS 声明。然而，可以在 Svelte 组件外部定义样式。

## 从外部 CSS 文件应用样式

假设你向应用程序的 HTML 中添加了一个样式元素，如下所示：

```js
<html>
  <head>
    <style>.title { color: blue }</style>
  </head>
</html>
```

或者，你可以在应用程序的 HTML 中包含外部 CSS 文件，如下所示：

```js
<html>
  <head>
    <link rel="stylesheet" href="external.css">
  <head>
</html>
```

在这两种情况下，它们中编写的 CSS 规则被应用到应用程序中的所有元素上，包括 Svelte 组件内的元素。

如果你正在使用构建工具，如 webpack、Rollup 或 Vite 来打包你的应用程序，通常需要配置你的构建工具以使用 `import` 语句导入 CSS 文件，就像导入任何 JS 文件一样（一些工具，如 Vite，甚至默认配置为允许像导入 JS 文件一样导入 CSS 文件！）：

```js
import './style.css';
```

### 导入 CSS 模块

在 Vite 中，当你将 CSS 文件命名为以 `.module.css` 结尾时，该 CSS 文件被视为 **CSS 模块** 文件。CSS 模块是一个 CSS 文件，其中定义的所有类名都具有局部作用域：

```js
/* filename: style.module.css */
.highlight {
  background-color: yellow;
}
```

这意味着在 CSS 模块中指定的 CSS 类名不会与任何其他地方指定的类名冲突，即使是与相同名称的类名也不会。

这是因为构建工具会将 CSS 模块中的类名转换成独特的东西，这不太可能与任何其他名称冲突。

以下是一个示例，说明前面的 CSS 模块在构建后 CSS 规则将如何呈现：

```js
/* the class name 'highlight' transformed into 'q3tu41d' */
.q3tu41d {
  background-color: yellow;
}
```

当从 JavaScript 模块导入 CSS 模块时，CSS 模块导出一个对象，其中包含原始类名到转换后类名的映射：

```js
import styles from './style.module.css';
styles.highlight; // 'q3tu41d'
```

在前面的代码片段中，导入的 `styles` 模块是一个对象，我们可以通过 `styles.highlight` 获取转换后的类名，即 `'q3tu41d'`。

这允许你在 Svelte 组件中使用转换后的类名：

```js
<script>
  import styles from './style.module.css';
</script>
<div class="{styles.highlight}" />
```

我们已经看到了六种不同的方式来设置 Svelte 组件的样式，但你是如何选择何时使用哪一种方式的呢？

## 选择为 Svelte 组件设置样式的技巧：

我们到目前为止看到的每种方法都有其优缺点。大多数时候，选择哪种方法来设置 Svelte 组件的样式取决于个人偏好和便利性。

在选择为我的 Svelte 组件设置样式的技巧时，以下是我的一些个人偏好：

+   `<style>` 块与内联样式：

    大多数时候，我发现我编写的样式与显示元素的逻辑关系不大，而更多的是关于元素的外观。因此，当样式与元素一起内联时，我发现这会干扰我阅读组件的流程。我更喜欢将所有样式放在 `<style>` 块的一个地方。

+   使用 `style:` 指令和 `class:` 指令控制样式：

    当元素的样式属性依赖于一个变量时，我会使用 `style:` 指令或 `class:` 指令而不是 `style` 属性或 `class` 属性。我发现这更易于阅读，同时也发现这是一个强烈的信号，告诉我元素的样式是动态的。

    当我仅基于一个变量更改一个样式属性时，我会使用 `style:` 指令。然而，当使用同一个变量更改多个样式属性时，我更倾向于声明一个 CSS 类来将所有样式组合在一起，并通过 `class:` 指令来控制它。

+   在多个 Svelte 组件中重用 CSS 声明的 CSS 模块：

    在撰写本书时，Svelte 中没有内置方法来在多个 Svelte 组件中共享相同的 CSS 声明。因此，你可能希望通过 CSS 模块来共享 CSS 声明。

    然而，更常见的情况是，当你需要在多个 Svelte 组件中为元素使用相同的 CSS 声明时，你可能会遇到一个不太完美的组件结构。可能可以将元素抽象成一个 Svelte 组件。

我们已经看到了如何在 Svelte 组件内部和外部定义自己的样式；然而，有时采用他人编写的样式比设计自己的样式要容易得多。

在下一节中，我们将探讨如何在 Svelte 中使用流行的 CSS 框架 Tailwind CSS。

# 使用 Tailwind CSS 样式化 Svelte

Tailwind CSS 是一个以实用类为首要的 CSS 框架。它包含预定义的类，如 `flex`、`pt-4` 和 `text-center`，你可以在你的标记中直接使用这些类：

```js
<div class="flex pt-4 text-center" />
```

我们将使用 Vite 的 Svelte 模板作为基础来设置 Tailwind CSS。如果你不熟悉设置 Vite 的 Svelte 模板，以下是一些快速设置步骤：

1.  运行 Vite 设置工具：

    ```js
    my-project-name containing the basic files necessary for a Svelte project.
    ```

1.  进入 `my-project-name` 文件夹并安装依赖项：

    ```js
    cd my-project-name
    npm install
    ```

1.  依赖项安装完成后，你可以启动开发服务器：

    ```js
    npm run dev
    ```

随着 Svelte 项目运行起来，让我们看看我们需要做什么来设置 Tailwind CSS。

## 设置 Tailwind CSS

Tailwind CSS 提供了一个 `tailwindcss` CLI 工具，使得设置变得更加简单。按照以下步骤操作：

1.  要在 Svelte + Vite 项目中设置 Tailwind CSS，我们首先安装所需的依赖项：

    ```js
    @tailwind directive – a Tailwind CSS directive, which you’ll see later.
    ```

1.  安装完 `tailwindcss` 后，运行命令以生成 `tailwind.config.js` 和 `postcss.config.js`：

    ```js
    tailwind.config.js file keeps the configuration for Tailwind CSS. Tailwind CSS works by scanning template files for class names and generating the corresponding styles into a CSS file.
    ```

1.  在 `tailwind.config.js` 中指定我们的 Svelte 组件的位置，以便 Tailwind CSS 知道在哪里扫描 Svelte 组件以获取类名：

    ```js
    module.exports = {
      content: [
        // all files ends with .svelte in the src folder
        "./src/**/*.svelte"
      ],
    };
    ```

    `postcss.config.js` 文件保存了 PostCSS 的配置。默认生成的配置目前是好的。

1.  在 `./src/main.css` 中创建一个 CSS 文件以添加 Tailwind 指令：

    ```js
    @tailwind base;
    @tailwind components;
    @tailwind utilities;
    ```

1.  在 `./src/main.js` 文件中导入新创建的 `./src/main.css`：

    ```js
    import './src/main.css';
    ```

    这与之前看到的导入外部 CSS 文件类似。

1.  启动 Vite `dev` 服务器：

    ```js
    text-center and text-sky-400 Tailwind CSS classes onto the elements in your Svelte component.For example, head over to `./src/App.svelte` and add the following: `<h1 class="text-center` `text-sky-400">Hello World</h1>`
    ```

你刚刚创建了一个带有两个 Tailwind CSS 类 `<h1>` 元素：`text-center` 和 `text-sky-400`。这将使 `<h1>` 元素中的文本居中对齐，并呈现天蓝色。当你使用 Tailwind CSS 实用类名称在你的 Svelte 组件中时，这些类的样式声明会被生成并替换掉 `@tailwind` `utilities` 指令。

Tailwind CSS 能够从 `class` 属性和 `class:` 指令中提取类名，允许你静态或条件性地将 Tailwind CSS 样式应用到元素上：

```js
<script>
  export let condition = false;
</script>
<h1 class="text-center">Center aligned title</h1>
<h1 class:text-center={condition}>
  Conditionally center aligned
</h1>
```

Tailwind CSS 包含了许多实用类：你可以在 [`tailwindcss.com/`](https://tailwindcss.com/) 上了解更多关于 Tailwind CSS 的信息。

由于 Tailwind CSS 类是全局可用的，因此你可以将相同的 CSS 类应用到同一应用程序内的 Svelte 组件上，以保持相同的视觉和感觉。然而，如果你使用自己的样式声明这样做，你可能必须在组件之间重新指定相同的颜色或尺寸以保持相同的视觉。将来更改值也会成为问题，因为我们必须通过值搜索用法并更新它们。

CSS 自定义属性是解决这个问题的方案。它们允许我们在一个地方定义一个值，然后在其他多个地方引用它。让我们看看我们如何在 Svelte 组件中使用 CSS 自定义属性。

# 使用 CSS 自定义属性为主题化 Svelte 组件

让我们对 CSS 自定义属性进行快速知识检查：

+   你可以像定义其他 CSS 属性一样定义 CSS 自定义属性，只是 CSS 自定义属性的名称以两个短横线开头：

    ```js
    --text-color: blue;
    ```

+   要引用 CSS 自定义属性，请使用 `var()` 函数：

    ```js
    color: var(--text-color);
    ```

+   CSS 自定义属性像任何其他 CSS 属性一样具有级联性：

    ```js
    .foo {
      --text-color: blue;
    }
    div {
      --text-color: red;
    }
    ```

    `<div>` 元素的 `--text-color` CSS 自定义属性的值是红色，除了具有名为 `foo` 的类的 `<div>` 元素。

+   CSS 自定义属性的值从其父元素继承：

    ```js
    <div>
      <span />
    </div>
    ```

    如果前一个示例中 `<div>` 的 `--text-color` 值是红色，那么如果没有其他 CSS 规则应用到 `<span>` 上，`<span>` 的 `--text-color` 值也是红色。

## 定义 CSS 自定义属性

要为 Svelte 组件指定一组尺寸和颜色，我们可以在应用程序的根组件中定义它们作为 CSS 自定义属性：

```js
<div>
 <ChildComponent />
</div>
<style>
 div {
  --text-color: #eee;
  --background-color: #333;
  --text-size: 14px;
 }
</style>
```

在前一个示例中，我们在组件的根元素（`<div>` 元素）处定义了 CSS 自定义属性。由于 CSS 自定义属性值从父元素继承，子元素和子组件内的元素从 `<div>` 元素继承值。

由于我们正在定义组件 `<div>` 元素的 CSS 自定义属性，因此不是 `<div>` 元素子代元素的元素将无法访问该值。

如果你希望为所有元素定义变量，即使是那些不是根组件根元素的子代元素，你可以在文档的根处使用 `:root` 选择器来定义 CSS 自定义属性：

```js
<style>
  :root {
    --text-color: #eee;
    --background-color: #333;
    --text-size: 14px;
  }
</style>
```

对于 `:root`，你不需要使用 `:global()` 伪选择器，因为它始终指向文档的根，并且永远不会作用域到组件。

在 CSS Modules 中，`:global()` 伪选择器用于定义全局样式，这些样式适用于局部模块作用域之外。在 Svelte 中，当在组件的 `<style>` 块中使用时，它允许你定义不会作用域到组件的 CSS 规则，使它们在整个 Svelte 应用程序中的元素可用并适用。

作为在`<style>`块中定义 CSS 自定义属性的替代方案，你可以通过`style`属性直接在元素本身上定义它们：

```js
<div style="--text-color: #eee; --background-color: #333; --text-size: 14px;">
  <ChildComponent />
</div>
```

如你所回忆，使用`style:`指令这样做可以使代码看起来更整洁：

```js
<div
  style:--text-color="#eee"
  style:--background-color="#333"
  style:--text-size="14px">
  <ChildComponent />
</div>
```

要在子组件中引用该值，你使用`var()`函数：

```js
<style>
  div {
   color: var(--text-color);
  }
</style>
```

使用 CSS 自定义属性的优点在于，我们可以动态地更改 CSS 自定义属性的值，引用该 CSS 自定义属性的元素样式将自动更新。

例如，我可以根据条件指定`<div>`的`--text-color`值为`#222`或`#eee`：

```js
<script>
  export let condition = false;
</script>
<div style:--text-color={condition ? '#222' : '#eee'} >
  <ChildComponent />
</div>
```

当条件为`true`时，`var(--text-color)`的值为`#222`，当条件变为`false`时，值变为`#eee`。

如你所见，CSS 自定义属性使得同步元素样式变得容易得多。

现在，让我们看看使用 CSS 自定义属性的实际情况：创建深色/浅色主题模式。

## 示例 – 实现深色/浅色主题模式

深色模式是一种颜色方案，其中文本颜色较浅，背景颜色较深。深色模式背后的理念是在保持颜色对比度比率的条件下减少屏幕发出的光线，从而使内容仍然可读。来自设备的较少光线使得阅读更加舒适，尤其是在低光环境中。

大多数操作系统允许用户设置是否使用深色或浅色主题的偏好，并且主要浏览器支持`prefers-color-scheme`媒体查询来指示用户的系统偏好：

```js
@media (prefers-color-scheme: dark) {
  /* Dark theme styles go here */
}
@media (prefers-color-scheme: light) {
  /* Light theme styles go here */
}
```

在我们开始之前，让我们确定所需的变量。

为了简化问题，我们只更改背景色和文本色，因此将是`--background-color`和`--text-color`。

可能你的应用程序还有其他颜色，例如强调色、阴影色和边框色，这些颜色在深色和浅色主题中需要不同的颜色。

由于这些颜色将被应用到各个地方，我们将在根元素上使用`:root`伪类来定义它们：

```js
<style>
  @media (prefers-color-scheme: dark) {
    :root {
      --background-color: black;
      --text-color: white;
    }
  }
  @media (prefers-color-scheme: light) {
    :root {
      --background-color: white;
      --text-color: black;
    }
  }
</style>
```

现在，在我们的 Svelte 组件中，我们需要开始设置文本颜色以使用`var(--text-color)`：

```js
<style>
  * {
    color: var(--text-color);
  }
</style>
```

就这样；当系统偏好设置为深色主题时，文本颜色将是白色，当设置为浅色主题时，文本颜色将是黑色。

由于 CSS 自定义属性的继承特性，CSS 自定义属性的值将由设置了该值的最近父元素确定。

这为允许组件用户指定组件样式而无需通过更高特异性的 CSS 规则覆盖样式声明打开了大门。

## 允许用户更改组件的样式

假设你使用 CSS 在`<style>`块中为组件设置样式，如下所示：

```js
<p>Hello World</p>
<style>
  p {
   color: red;
  }
</style>
```

如果你想从组件外部修改`p`元素的色彩，你需要了解以下内容：

+   使用`div :global(p)`选择器来覆盖颜色；然而，由于我们不知道组件的实现细节，我们无法确定我们的选择器是否具有更高的特定性：

    ```js
    <div>
      <Component />
    </div>
    <style>
      div :global(p) {
        color: blue;
      }
    </style>
    ```

+   **组件的元素结构**：

    要知道要覆盖哪个元素的色彩，我们需要了解组件的元素结构，以及我们想要更改颜色的文本所在的元素是否是段落元素。

    组件的 CSS 规则和元素结构不应成为组件的公共 API 的一部分。通过具有更高特定性的 CSS 规则覆盖组件的样式是不推荐的。对 CSS 规则或元素结构的小幅调整很可能会破坏我们的 CSS 规则覆盖。

    一种更好的方法是公开一个 CSS 自定义属性的列表，这些属性可以用来覆盖组件的样式：

    ```js
    <p>Hello World</p>
    <style>
      p {
       color: var(--text-color, red);
      }
    </style>
    ```

    `var()`函数接受一个可选的第二个参数，它是如果第一个参数中的变量名不存在时的后备值。

    如果你使用组件而没有定义`--text-color`，那么段落的颜色将回退到红色。

    在以下代码片段中，段落的颜色是红色：

    ```js
    <div>
      <Component />
    </div>

    ```

    这里，`"display:contents"`是为了确保额外的`div`不参与`<Component />`内容的布局。

    ```js

    ```

如果我们在使用 CSS 自定义属性时指定后备值，我们可能会发现自己在重复后备值几次。如果我们打算更改后备值，这将变得麻烦。让我们看看我们如何对齐后备值。

## 对齐后备值

如果我们在多个元素中使用`var(--text-color, red)`，你可能会很快意识到我们也应该定义一个 CSS 自定义属性作为后备值，以免我们重复多次使用该值，将来可能难以找到并替换所有这些值。

要定义另一个 CSS 自定义属性，你必须在组件的根元素上定义它。如果该值仅限于你的组件及其子组件，那么你不应该在文档根元素上通过`:root`定义 CSS 自定义属性：

```js
<p>Hello World</p>
<style>
  p {
    --fallback-color: red;
    color: var(--text-color, var(--fallback-color));
  }
</style>
```

然而，这种方法要求我们在使用`var(--text-color)`的任何地方都使用`var(--fallback-color)`。

一种稍微更好的方法是定义一个新的 CSS 自定义属性，如果已定义，则其值为`--text-color`，否则为红色作为后备：

```js
<style>
  p {
   --internal-text-color: var(--text-color, red);
   color: var(--internal-text-color);
  }
</style>
```

这样，`var(--internal-text-color)`的值将始终被定义，并且使用单个 CSS 自定义属性对后续元素进行操作更为方便。

# 摘要

在本章中，我们介绍了六种不同的方法来样式化 Svelte 组件。那么，你知道你打算使用哪种方法来样式化你的 Svelte 组件吗？你现在应该知道选择最适合场景的方法。

我们首先了解了如何在 Svelte 项目中使用 Tailwind CSS。在开始时，需要一些初始设置来让 Tailwind CSS 启动并运行，但像 Tailwind CSS 这样的 CSS 框架通常自带预定义的 CSS 类，大多数情况下，你使用 `class` 属性或 `class:` 指令来应用它们。

最后，我们介绍了如何使用 CSS 自定义属性来为主题 Svelte 组件，以及如何允许组件用户自定义组件的样式。现在，你可以在允许他人使用不同于你创建的默认样式的条件下创建和分享 Svelte 组件。

在下一章中，我们将探讨如何管理 Svelte 组件的 props 和 states。
