# 10

# 管理静态资源

当涉及到管理静态资源时，SvelteKit 与这个过程几乎没有关系。实际上，整个过程都交给了打包工具 Vite。通过利用 Vite 来管理静态资源，我们作为开发者无需再学习另一个框架特定的策略。相反，我们可以依赖 Vite 的高效打包和构建过程。因为 Vite 自动管理导入的资源，我们无需担心为缓存文件进行哈希处理。在本章中，我们将探讨如何利用 Vite 来管理静态资源，如图片、字体、音频、视频和 CSS 文件。一旦我们考察了*如何*实现这一点，我们还将讨论一些与静态资源相关的细节。

本章将分为以下部分：

+   导入资源

+   其他信息

到我们完成时，我们将对在 SvelteKit 应用程序中包含可以“原样”提供的服务文件的最佳实践有一个牢固的掌握。

# 技术要求

本章的完整代码可在 GitHub 上找到：[`github.com/PacktPublishing/SvelteKit-Up-and-Running/tree/main/chapters/chapter10`](https://github.com/PacktPublishing/SvelteKit-Up-and-Running/tree/main/chapters/chapter10)。

# 导入资源

如果你过去十年从事过 Web 开发，那么你可能会记得一个时期，那时样式要么是内联编写的，要么是在**层叠样式表**（**CSS**）中编写的。这些**CSS**文件对于创建应用程序的一致外观和感觉非常有帮助。当然，它们的集中化性质也带来了自己的缺点。它们通常变得很大且难以导航，这可能导致包含未使用的样式规则。当宝贵的毫秒可以决定用户转化或失去销售的时候，不向客户发送未使用的资源是非常重要的。此外，如果我们用 SvelteKit 构建 Web 应用程序，我们真的应该使用 Svelte 方法，并将样式隔离在每个组件中。但有时保留一个应用全局样式的样式表是有用的。例如，想象一下需要将特定样式应用到每个段落元素上。将相同的简单样式规则应用到应用程序中的每个组件可能会导致代码重复。甚至可能存在我们忘记包含规则的情况，导致应用程序中样式不一致。虽然像**Tailwind CSS**或**Bootstrap**这样的项目很棒，但它们可能并不适合每个项目，这就是为什么我们将介绍如何在 SvelteKit 应用程序中包含全局样式表。

首先，我们需要一些样式。请注意，这些样式将应用于整个应用程序。通常，在 Svelte 组件内创建样式时，这些样式会被隔离到特定的组件中，这意味着它们不会应用于父组件或子组件。许多现代浏览器会为 HTML 元素应用它们自己的默认样式，因此，为了统一应用程序的体验，通常的做法是创建一个 `reset.css` 文件。此文件通过将浏览器为常见元素应用的样式重置为可预测的样式，确保在不同浏览器之间的一致性体验。在我们的例子中，我们将使用由 *Josh W. Comeau* 编写的简洁而全面的 **自定义 CSS 重置** 的略微修改版本。请参阅本章末尾的资源，以获取解释其工作原理的文章链接：

src/reset.css

```js
/* https://www.joshwcomeau.com/css/custom-css-reset/ */
*, *::before, *::after {
  box-sizing: border-box;
}
* {
  margin: 0;
}
html, body {
  height: 100%;
  font-family: sans-serif;
}
body {
  line-height: 1.5;
  -webkit-font-smoothing: antialiased;
}
img, picture, video, canvas, svg {
  display: block;
  max-width: 100%;
}
input, button, textarea, select {
  font: inherit;
}
p, h1, h2, h3, h4, h5, h6 {
  overflow-wrap: break-word;
}
```

从本质上讲，此 CSS 文件的规则是为各种 HTML 元素设置更合理和可预测的默认样式。例如，`box-sizing` 被设置为 `border-box`，这适用于所有元素以及 `::before` 和 `::after` 伪元素。这条规则意味着元素的填充将在计算该元素宽度时被包含。这些 CSS 规则允许在浏览器之间提供一致和可靠的体验。当然，我们可以自由地向此 CSS 添加任何内容。为了使我们的更改更加明显，我们还对 `html` 和 `body` 元素设置了 `font-family: sans-serif;`。

要将此 CSS 包含到我们的应用程序中，我们需要打开 `src/routes/+layout.svelte` 并像导入 JS 模块一样导入它。如果你还记得*第二章*，我们曾使用相同的方法导入图像路径！

src/routes/+layout.svelte

```js
<script>
  import '/src/reset.css';
  import Nav from '$lib/Nav.svelte';
  import Notify from '$lib/Notify.svelte';
  export let data;
</script>
...
```

显然，我们在此文件中需要做的唯一更改是第一行，其中我们导入 `reset.css`。为了简洁起见，文件中的其余代码已被省略。导入 CSS 文件后，请注意我们的样式立即被应用。我们不需要创建 `<link>` 或 `<style>` 标签，因为 Vite 识别样式表并将其自动为我们应用。方便的是，文件导入路径可以是相对的或绝对的，因为 Vite 不会区分这两者。

为了突出使用此方法导入样式表的好处，让我们将其与另一种包含全局样式表的方法进行比较。这种方法应用于 SvelteKit 的 1.0 版本之前的预发布版本，它通过手动将 `<link>` 标签添加到 `src/app.html` 的 `<head>` 部分来实现。然后，它使用 `%sveltekit.assets%` 占位符引用 `static/` 目录中的文件。这种方法并不可取，但让我们分析它以考虑其缺陷：

src/app.html

```js
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <link rel="icon" href="%sveltekit.assets%/favicon.png" />
    <meta name="viewport" content="width=device-width" />
    <link rel="stylesheet" href="%sveltekit.assets%/global.css" />
      %sveltekit.head%
  </head>
  <body data-sveltekit-preload-data="hover">
    <div style="display: contents">%sveltekit.body%</div>
  </body>
</html>
```

如我们所见，这种方法将`static/global.css`文件包含在`src/app.html`的 head 标签中，这是应用程序的入口点。`app.html`文件作为整个应用程序构建的基础框架，因此我们可以合理地认为，我们可以在这里包含任何额外的脚本或外部资产，就像 favicon 一样。这种方法依赖于`%sveltekit.assets%`占位符来包含`static/`目录中的 CSS 文件。然而，这种方法没有考虑到 Vite 的 HMR 功能。每当对`static/global.css`进行更改时，整个开发服务器都需要重新启动以反映这些更改，因为 Vite 不会处理任何包含在静态资产中的文件。同时考虑一下常见的场景，即对`.css`文件进行压缩，对`.scss`、`.sass`或`.less`文件进行预处理。在这些情况下，我们需要 Vite 比从 SvelteKit 的`static/`目录中包含的文件采取更积极的做法。而且由于 Vite 可以通过在静态资产的文件名后附加哈希来管理缓存文件，很明显，像导入 JS 模块一样导入文件符合我们的最佳利益。

在*第二章*中，我们看到了如何导入一个图片 URL。现在我们也看到了 Vite 如何允许我们直接将全局 CSS 文件导入到我们的 Svelte 组件中。既然我们已经展示了如何动态地最佳利用静态资产，让我们讨论一下这个过程中的一些细节。

# 其他信息

我们现在知道*如何*导入静态文件，但在这样做时，还有一些细节需要注意。以下是我们需要覆盖的各个项目的分解：

+   图片与样式

+   自定义导入

+   文件路径

+   SvelteKit 配置选项

+   Vite 配置选项

现在让我们回顾一下关于我们每个导入背后发生的一些重要信息。

## 图片与样式

当我们在*第二章*中导入图片时，我们收到了 URL，然后我们在`<img>`标签的`src`属性中引用它。当我们导入 CSS 文件时，我们只需要导入它以应用样式。这是因为 Vite 预先配置为自动将 CSS 文件中的样式注入到执行导入的组件中。这就是为什么导入是在根`+layout.svelte`文件中执行的原因。Vite 还支持 CSS `@import`和`url()`语句以及 CSS 模块。CSS 模块可以在代码中将样式规则作为对象导入时很有用，而`@import`和`url()`允许开发者构建一个中心 CSS 文件，该文件可以引用位于其他地方的较小的 CSS 文件。当导入 CSS 文件时，除了导入之外不需要采取任何其他操作。当导入其他媒体，如字体、音频或视频文件时，我们需要将导入的资产 URL 设置为相应 HTML 元素的`src`属性。

## 自定义导入

当从 Vite 导入静态资源时，我们可以通过在文件导入名称后附加适当的后缀来自定义它们的导入方式。例如，要将文件作为字符串导入，我们可以在文件名后附加`?raw`。正如预期的那样，这将给我们导入文件的原始内容。对于前面显示的`reset.css`示例，它可以通过`import reset from '/src/reset.css?raw';`来包含，其中 reset 变量包含`reset.css`的内容。然后我们需要找到一种方法将此内容包含在`<style>`标签内。以类似的方式，如果我们想将文件作为不在标准媒体类型中找到的 URL 导入，我们可以在文件导入语句后附加`?url`。这有助于将来自`?worker`的文件包含到文件名中！

## 文件路径

当运行 Vite 的开发服务器时，我们可以在浏览器开发者工具中观察网络请求，并注意到导入的文件是从项目源代码中的位置提供的。例如，来自*第二章*的图片位于`src/lib/images/demo.svg`，并且也是从该位置提供的。然而，当我们运行`npm run build`然后运行`npm run preview`时，我们会观察到路径已更改。它被赋予的路径是`_app/immutable/assets/demo.dd76856a.svg`。此路径是构建的 SvelteKit 应用程序特有的，通常在应用程序构建后我们不会编辑它。但请花点时间注意，文件名中包含了一个哈希值。如果文件内容发生变化，我们会注意到构建的资产将包括附加到文件名上的不同哈希值。

我们也可以利用这个时刻来观察包含的 favicon 文件。我们会看到在`dev`和`build`/`preview`中，它都是从域名根目录`/`提供的。这是因为它位于`static/`目录中，Vite 提供和构建应用程序，使得任何位于那里的文件都将从应用程序的根级别提供。

## SvelteKit 配置选项

这些选项可以在`svelte.config.js`中进行自定义：

+   `files.assets` – 此选项指定了静态资源将被存储的目录。SvelteKit 会自动将此选项设置为`static`，并覆盖由`publicDir`指定的 Vite 同级设置（默认为`public`）。通常适合这里的文件有`robots.txt`或`favicon.ico`，因为它们很少改变。要在源代码中引用此处定位的文件，只需在文件名前加上`/`即可。例如，我们可以在任何组件中通过添加`<img src='/favicon.png' />`来显示默认的 favicon 图标。

+   `paths.assets` – 此选项接受一个字符串，指定从哪里提供应用程序文件的绝对路径。它默认为空字符串。

+   `paths.base` – 此选项也默认为空字符串。如果你的应用程序是从子目录中提供的，你可以在这里指定根相对路径。然后，你可以使用从 `$app/paths` 导入的 `base` 模块来适当地修改硬编码的路径。

+   `paths.relative` – 此选项接受一个布尔值。当 `true` 时，`$app/paths` 模块中提供的 `base` 和 `assets` 的值将相对于构建的资产。当 `false` 时，这些相同的值将是根相对的。

## Vite 配置选项

此选项可以在项目的 `vite.config.js` 中进行自定义：

+   `assetsInclude` – 许多常见的媒体类型都由 Vite 自动处理，但如果项目需要扩展默认列表以将不常见的文件类型视为资产，则此选项可能很有用。此选项允许自定义允许的静态资产文件类型。它可以是字符串、正则表达式或 **picomatch** 模式。

我们刚刚看到了如何自定义静态资产的导入。如果我们需要强制将资产导入为 URL，我们知道我们会在导入文件的末尾追加 `?url`。我们还了解到 CSS 文件会自动注入到导入到其中的组件中。结合一些配置选项，这些细节提供了关于如何在 SvelteKit 应用程序中使静态资产的管理无压力的见解。

# 摘要

当在 SvelteKit 应用程序中包含图像、CSS 文件或其他媒体类型时，很明显我们应该利用 Vite 来导入资产，就像我们导入 JS 模块一样。这样做的好处是简单，同时也允许优化缓存。它使我们的开发体验保持流畅，因为 Vite 的 HMR 会自动在浏览器中显示更改。它也非常灵活，允许通过 URL 或原始内容导入各种媒体类型。

现在我们知道了如何管理静态资产，我们应该回到如何管理秘密的话题。如果你还记得上一章，我们在 `.env` 文件中添加了一个个人访问令牌，这使得我们可以使用 GitHub API 进行身份验证。在下一章中，我们将进一步探讨这一点，并介绍使管理秘密变得轻松的各种模块。

# 资源

+   Tailwind CSS – [`tailwindcss.com/`](https://tailwindcss.com/)

+   Bootstrap – [`getbootstrap.com/`](https://getbootstrap.com/)

+   Josh W. Comeau 的自定义 CSS 重置 – [`www.joshwcomeau.com/css/custom-css-reset/`](https://www.joshwcomeau.com/css/custom-css-reset/)

+   Vite 配置选项 – [`vitejs.dev/config/`](https://vitejs.dev/config/)

+   CSS 模块 – [`github.com/css-modules/css-modules`](https://github.com/css-modules/css-modules)

+   picomatch – [`github.com/micromatch/picomatch`](https://github.com/micromatch/picomatch)
