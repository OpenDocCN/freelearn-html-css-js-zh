

# 提高可访问性和优化 SEO

当涉及到构建 Web 应用程序时，我们不能忽视确保应用程序对所有用户可访问的重要性。通过赋权依赖于屏幕阅读器等辅助技术的用户，我们可以进一步扩大我们应用程序的影响。不仅使应用程序对更广泛的受众可用可以带来更多用户，而且还可以影响**搜索引擎优化**（**SEO**）。因此，忽视 SvelteKit 如何帮助我们从一开始就使应用程序可访问将是疏忽的。

为了了解我们如何最好地赋权我们的用户，以及这样做如何帮助我们提高 SEO，我们应该检查几个概念。首先，我们将看到内置的编译时检查如何通过我们端的小配置来提高我们应用程序的可访问性。我们还将了解如何最好地宣布路由更改，这可以惠及屏幕阅读器等工具。然后，我们将简要介绍一些可以惠及可访问性的技巧，并以一些简单的 SEO 优化技巧结束。我们将所有内容分解为以下部分：

+   编译时检查

+   宣布路由

+   可访问性增强

+   SEO 技巧

完成本章后，我们将涵盖确保您的 SvelteKit 应用程序对广大受众可访问性的基本要素。遵循这里概述的最佳实践将带来提高 SEO 排名的额外好处。

# 技术要求

本章的完整代码可在 GitHub 上找到：[`github.com/PacktPublishing/SvelteKit-Up-and-Running/tree/main/chapters/chapter12`](https://github.com/PacktPublishing/SvelteKit-Up-and-Running/tree/main/chapters/chapter12)

# 编译时检查

当我们安装我们的 SvelteKit 项目时，它自带了一些有见地的增强功能。在这些增强功能中，编译时检查特别有用，可以警告我们有关格式不佳或缺少属性的元素。在做出建议的更改后，我们会注意到这些警告消失了。

如果你注意到了`eslint`的建议或构建输出的结果，你可能已经注意到了一些关于`A11y`的警告。这是指代*可访问性*的缩写术语。它指的是字母*A*，接下来的 11 个字符，以及字母*y*。在认识到使应用程序可访问的重要性时，Svelte 开发者选择默认包含合理的默认行为，因为这有助于构建一个更加开放的互联网。在因常规警告而感到沮丧之前，考虑一下在无需寻求我们自己的解决方案的情况下，让应用程序检查 a11y 错误带来的便利。不仅考虑 a11y 构建有助于用户，而且有助于开发者通过识别哪些模式是可访问的，哪些不是，来变得更好。

如果你还没有看到这些问题，我们可以回到我们最早的例子之一，并从`<img>`元素中移除`alt`属性：

src/routes/+page.svelte

```js
<script>
  import db from 'db';
  import url from 'img/demo.svg';
  let status = db.connection;
  let name = 'World';
</script>
<form>
  <label for="name" >What is your name?</label>
  <input type="text" class='name' bind:value={name} />
</form>
<h1>Hello, {name}!</h1>
<p>{status}</p>
<img src={url}>
// A11y: <img> element should have an alt attribute
```

在这个例子中，所做的唯一更改是在`<img>`标签的最后移除`alt`属性。大多数现代编辑器应该会直接在文件中提醒您，但如果您没有看到这个警告，您可以在终端中运行`npm run build`来直接查看构建输出。在观察`build`输出后，我们将能够确定问题的确切位置并查看推荐的修复方案。

这些警告不仅适用于 HTML 元素的缺失属性。如果表单标签与控件相关联，如果某些媒体类型有字幕，如果属性赋予不正确的值，以及更多无法合理列出的警告，我们也会收到警告。有关完整列表，请参阅本章末尾的资源。正如我们所见，编译时无障碍性检查可以在帮助开发者向尽可能多的用户提供可访问的应用程序方面非常有用。

遵循编译时检查警告的建议并不是我们提高应用程序无障碍性的唯一方法。我们还可以通过更新每个页面的标题来通知用户导航事件。通过以下方式宣布路由更改，我们可以在保持客户端导航的同时，提醒屏幕阅读器用户。

# 宣布路由

确保我们应用程序的无障碍性的另一种策略是宣布我们的路由。这意味着我们的所有页面都包含一个标题，以便屏幕阅读器可以向用户宣布新页面。在典型的 SSR 应用程序中，导航包括在导航到时加载每个新页面。在 SvelteKit 中，导航由客户端处理，因此并不总是需要完整的页面重新加载。这对屏幕阅读器来说是一个困境，因为它们依赖于每个链接点击时都存在新的标题元素，以便向用户宣布页面。

为了更好地与屏幕阅读器协同工作，我们可以使用`<svelte:head>`指令将标题插入到我们创建的每个新页面中：

src/routes/+page.svelte

```js
<script>
  import db from 'db';
  import url from 'img/demo.svg';
  let status = db.connection;
  let name = 'World';
</script>
<svelte:head>
  <title>Home</title>
</svelte:head>
<form>
  <label for="name" >What is your name?</label>
  <input type="text" class='name' bind:value={name} />
</form>
<h1>Hello, {name}!</h1>
<p>{status}</p>
<img src={url} alt='demo'>
```

在我们的应用程序着陆页面上，我们已经重新添加了`alt`属性到我们的`<img>`标签，但更重要的是，我们已经将页面标题设置为`<svelte:head>`标签。让我们在另一个文件中也进行类似的更改，以便我们可以观察这如何影响浏览体验：

src/routes/(site)/about/+page.svelte

```js
<svelte:head>
  <title>About</title>
</svelte:head>
<div class='wrapper'>
  <h1>About</h1>
  <p>
    Lorem ipsum dolor ...
  </p>
</div>
```

在我们应用程序的`<title>`元素中，使用适当的文本，并用`<svelte:head>`标签包围它。这些标签将内容放置在文档的头部。为了了解这如何影响浏览体验，请打开应用程序的开发版本，并注意在浏览器中显示的主页页面标题。然后，点击**关于**，观察浏览器标签中显示的标题如何变化。

对于屏幕阅读器应用程序，这个小小的改变允许它们提醒用户他们已经导航到一个新的路由。即使对于不使用屏幕阅读器的用户，这也是对之前只显示站点名称的文本的一个明显的改进。如果用户在这个网站上标记了一个页面，现在的默认文本将更准确地反映该页面。这不仅对所有人类用户有帮助，而且还可以大大提高我们应用程序的 SEO，因为许多搜索引擎在索引帖子时会考虑页面标题。

通过宣布路由，我们可以极大地改善我们应用程序的用户体验。接下来，让我们看看一些其他的小调整，这些调整可以带来很大的改进。

# 无障碍性增强

并非所有开发人员以及所有使用屏幕阅读器的用户都将英语作为他们的母语。因此，我们应该能够相应地调整我们的应用程序。这样做相当直接，通过在内容提供语言上做一小笔记，我们可以极大地改善全球辅助技术用户的体验。默认情况下，SvelteKit 将语言设置为英语，但我们可以通过更改`src/app.html`中的`<html>`元素的`lang`属性来快速调整：

src/app.html

```js
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="utf-8" />
        <link rel="icon" href="%sveltekit.assets%/favicon.png" />
        <meta name="viewport" content="width=device-width" />
        %sveltekit.head%
    </head>
    <body data-sveltekit-preload-data="hover">
         <div style="display: contents">%sveltekit.body%</div>
    </body>
</html>
```

注意文件顶部的`<html>`元素，其中设置了`lang`属性。将`lang`属性设置为适当的语言，例如`fr`代表法语或`ar`代表阿拉伯语，确保屏幕阅读器可以正确发音或翻译内容。

我们可以在我们的应用程序中实现的最后一个无障碍性改进是允许 SvelteKit 操纵 HTML 元素上的焦点。通常，当应用程序在服务器上渲染时，每个新的导航事件都会重置焦点。但在客户端渲染的应用程序中，浏览器可能无法检测到导航事件的发生，因此，焦点将保持在当前聚焦的元素上。为了考虑到无障碍性，SvelteKit 将焦点重置到`<body>`元素上——也就是说，除非某个元素设置了`autofocus`属性，在这种情况下，该元素将被给予焦点。让 SvelteKit 的行为控制焦点的好处是让屏幕阅读器的用户知道已经发生了导航事件。

当谈到使我们的应用程序更容易被更广泛的受众使用时，我们不需要付出太多努力。这样做可以改善用户体验，上述所有改进也可以提高 SEO 排名。

# SEO 技巧

除了在我们的应用程序中做一些小的 a11y 改进外，我们还可以记住一些其他建议。首先，我们应该尽可能使用 SvelteKit 的 **服务器端渲染**（**SSR**）。这样做可以确保应用程序快速交付，同时使内容更容易被搜索引擎解析。当然，现在许多搜索引擎都有索引客户端渲染内容的能力，但 SSR 的速度和可靠性不容忽视。我们只有在有充分理由的情况下才应该禁用 SSR。

另一个值得考虑的有用技巧是考虑我们应用程序的性能。在大多数情况下，我们可以依赖 Vite 从我们的构建中摇树干未使用的代码。更小的包大小意味着需要发送给客户端的代码行数更少，许多搜索引擎根据资产交付时间来排名结果。请参阅本章末尾的 *资源* 部分，了解可以提供关于页面速度洞察力的工具。

提高搜索引擎优化（SEO）的最后一个有用技巧是在路由名称中省略尾部斜杠。额外的斜杠可能会对页面排名产生负面影响，所以除非你有充分的理由，否则考虑让页面选项 `trailingSlash` 属性保持不变。遵循这些少数几个技巧，我们可以确保我们的 SvelteKit 应用程序在搜索引擎结果中排名很高。

# 摘要

当谈到构建一个成功的网络应用程序时，我们必须努力使其对所有用户都易于访问。这样做的原因可能是出于纯粹的自私，试图尽可能多地占领市场，或者出于平等主义，试图包括来自网络各个领域的用户。可能你只是想在高搜索引擎结果中排名很高。无论原因如何，使用 SvelteKit 的过程相当直接。我们已经看到了编译时提供的警告，并且我们已经了解了在创建独特的页面标题时，SEO 和 a11y 的好处。只要记住一些 SEO 小技巧，我们的可访问应用程序就很容易为世界所知。

在完成本章后，我们几乎涵盖了关于 SvelteKit 所有的讨论内容。然而，技术发展迅速，所以我们永远无法真正停止学习。到这本书出版时，SvelteKit 很可能已经引入了更多的改进和变化。为了确保您拥有最新的信息，下一章将提供更多资源、社区和值得探索的示例。

# 资源

+   *可访问性项目* – [`www.a11yproject.com/`](https://www.a11yproject.com/)

+   *MDN 网络文档：可访问性* – [`developer.mozilla.org/en-US/docs/Web/Accessibility`](https://developer.mozilla.org/en-US/docs/Web/Accessibility)

+   *Svelte 可访问性警告* – [`svelte.dev/docs#accessibility-warnings`](https://svelte.dev/docs#accessibility-warnings)

+   *页面速度洞察力* – [`pagespeed.web.dev/`](https://pagespeed.web.dev/)

# 附录：示例和支持

由于学习是一个永无止境的过程，而且技术发展迅速，本章的最终目的是为你提供继续使用 SvelteKit 旅程所需的资源。在 Web 开发的世界里，几乎每个项目都会集成多个工具和技术，因此我们将探讨如何轻松地将 SvelteKit 与其他前端工具集成。我们还将看到一些官方和社区资源，这些资源在解决问题、提升知识或发现新组件时非常有价值。之后，我们将以作者的感谢语结束。让我们以下列章节结束这本书：

+   集成

+   更多阅读和资源

+   总结

之后，你将拥有所有必要的工具和知识，可以继续构建酷炫的 SvelteKit 项目。

# 技术要求

本章的完整代码可在 GitHub 上找到：[`github.com/PacktPublishing/SvelteKit-Up-and-Running/tree/tailwind`](https://github.com/PacktPublishing/SvelteKit-Up-and-Running/tree/tailwind)。

# 集成

当谈到构建现代 Web 应用程序时，使用大量技术并不罕见。每个工具都有其位置，并且开发者可能更熟悉特定的前端框架，而不是标准 CSS。这是可以的，只要这些工具能够很好地与其他工具集成，它就可以加快开发速度。幸运的是，SvelteKit 与其他工具配合得相当好。

在撰写本文时，Tailwind CSS 已经变得非常流行。Tailwind CSS 的目标是减少发送的 CSS 量，只提取使用的那部分。这对于减少发送给客户端的资产数量和加快加载时间非常有用。为了展示如何简单地将 Tailwind CSS 这样的工具集成到我们现有的 SvelteKit 项目中，让我们来操作一下。这些步骤也可以在官方 Tailwind CSS 文档中找到。建议在开始此过程之前在你的仓库中创建一个新的分支，因为它将破坏我们的一些现有样式。如果你正在跟随这本书的仓库，这些示例可以在 `tailwind` 分支上找到。首先，我们可以使用以下命令安装 Tailwind 以及其他几个依赖项：

```js
npm install -D tailwindcss postcss autoprefixer
```

当然，`tailwindcss` 将包含在项目中使用 Tailwind CSS 所需的工具。`postcss` 依赖项将允许我们操作 CSS 文件，而 `autoprefixer` 是一个 `postcss` 插件，它将自动将适当的供应商前缀注入到我们生成的 CSS 中。一旦我们将依赖项添加到我们的开发环境中，我们可以使用以下命令来初始化我们的 Tailwind 项目。它将创建必要的 `tailwind.config.js` 和 `postcss.config.js` 文件：

```js
npx tailwindcss init -p
```

在初始化 `tailwindcss` 之后，我们可以打开 `svelte.config.js` 并导入 `vitePreprocess` 模块。这将使我们能够处理 Svelte 组件中的 `<style>` 标签：

svelte.config.js

```js
import adapter from '@sveltejs/adapter-auto';
import { vitePreprocess } from '@sveltejs/kit/vite';
const config = {
    kit: {
        adapter: adapter(),
        alias: {
            db: '/src/db.js',
            img: '/src/lib/images'
        }
    },
    preprocess: vitePreprocess()
};
export default config;
```

现在我们已经导入了 `vitePreprocess`，我们可以确保 Tailwind CSS 了解我们的组件路径。我们可以通过更新 `tailwind.config.js` 来做到这一点，如下所示：

tailwind.config.js

```js
/** @type {import('tailwindcss').Config} */
export default {
  content: ['./src/**/*.{html,js,svelte,ts}'],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

显然，我们只需要更改 `content` 数组属性中可用的路径，指向我们的 `src/` 目录，并确保 `.svelte` 文件类型被识别，以及其他标准文件类型。

然后，我们可以创建一个单独的 `app.css` 文件，在那里我们可以使用 `@tailwind` 指令导入 Tailwind 的所有功能：

src/app.css

```js
@tailwind base;
@tailwind components;
@tailwind utilities;
```

如果你一直很注意，下一步应该很简单。然后我们将 `src/app.css` 导入到我们的根布局组件中：

src/routes/+layout.svelte

```js
<script>
  import '/src/reset.css';
  import '/src/app.css';
  import Nav from '$lib/Nav.svelte';
  import Notify from '$lib/Notify.svelte';
  export let data;
</script>
<div class='wrapper'>
  <div class='nav'>
    <div class='menu'>
      <Nav />
    </div>
    <div class='notifications'>
      <Notify count={data.notifications.count}/>
    </div>
  </div>
  <div class='content bg-orange-300'>
    <slot />
  </div>
  <div class='footer'>
    This is my footer
  </div>
</div>
<!-- <style> omitted for brevity -->
```

当然，我们已导入 `reset.css`，因此我们的项目中的现有 CSS 将会有冲突。确保你的开发环境正在运行 `npm run dev`。为了防止完全破坏我们的项目，我们只将 `.content` 元素的背景设置为 Tailwind CSS 提供的浅橙色，但我们肯定会注意到项目中的其他变化。如果你还没有这样做，现在是一个探索以实用为先的 CSS 实践的绝佳时机。

我们看到了手动集成另一个工具（如 Tailwind CSS）的方法，但我们谈论的是 SvelteKit，在那里事情总是按预期进行。如果这些步骤太难记住，有一个更简单的方法。尝试在你的项目仓库的 `main` 分支上创建另一个分支，并使用以下命令执行我们刚才所做的基本相同的事情。再次提醒，如果你正在跟随本书的仓库，这段代码可以在 `tailwind-add` 分支中找到：

```js
npx svelte-add@latest tailwindcss
```

我们可以按照提示进行操作，一旦我们使用 `npm install` 安装了依赖项，我们的项目就会集成 Tailwind CSS！通过使用社区维护的 `svelte-add` 项目，我们可以快速轻松地导入与我们的 SvelteKit 项目集成的模板。例如，如果你在编写 CSS 时更喜欢使用 SCSS/Sass 风格，你可以使用 `scss` 自定义添加器，如下所示：

```js
npx svelte-add@latest scss
```

如我们所见，使用 SvelteKit 集成不同的技术并不困难。虽然我们可以手动集成这些其他工具链，但使用社区提供的资源也同样容易。让我们看看更多的社区资源，看看还有哪些其他选择！

# 更多阅读和资源

如前所示，围绕 SvelteKit 的社区资源可以非常有效地节省我们的时间和精神负担，使我们能够专注于构建我们的应用程序。如果没有 SvelteKit 的社区，这本书就不可能完成。如果你想要扩展你的 SvelteKit 知识，帮助他人，或者创建你自己的 SvelteKit 扩展，请考虑以下列出的各种资源！

## SvelteKit 文档

在官方 SvelteKit 网站上提供的文档可能是你找到有关框架信息的最佳资源。它非常详尽，并且不断更新以反映框架内的变化。确保在有任何关于 SvelteKit 的问题时从这里开始：

[`kit.svelte.dev`](https://kit.svelte.dev)

## SvelteKit 教程

为了彻底测试你的 SvelteKit 知识并学习这本书无法涵盖的内容，请查看官方 SvelteKit 教程：

[`learn.svelte.dev/tutorial`](https://learn.svelte.dev/tutorial)

## Svelte 和 SvelteKit 聊天

有问题或只是想与其他使用 SvelteKit 的人聊天？官方 Discord 服务器是你要去的地方：

[`svelte.dev/chat`](https://svelte.dev/chat)

## 独立创作者

有太多优秀的作家和创作者在与 SvelteKit 合作，这里无法一一列举，但作者最喜欢的两位是罗德尼·约翰逊和乔希·柯林斯沃斯。柯林斯沃斯提供了我们在*第八章*中看到的出色的 SvelteKit 静态博客起始模板，而约翰逊则创作了信息丰富的教程视频和文章：

+   罗德尼·约翰逊：[`rodneylab.com/`](https://rodneylab.com/)

+   乔希·柯林斯沃斯：[`joshcollinsworth.com/`](https://joshcollinsworth.com/)

## Svelte Society

当谈到寻找 Svelte 和 SvelteKit 社区资源时，Svelte Society 会为你提供全方位的支持，无论你是寻找模板、组件、插件，还是更多。他们甚至还会组织 Svelte 活动，所以如果你想在你的地区遇见其他 Svelte 开发者，你应该从这里开始：

[`sveltesociety.dev/`](https://sveltesociety.dev/)

## SvelteKit 仓库

就像许多开源项目一样，SvelteKit 背后的代码在 GitHub 上免费提供供查看。如果你认为你发现了一个针对框架的特定错误，请在这里搜索问题，如果你没有看到你的问题被列出，请通过提交它来贡献！SvelteKit 开发者不断接受拉取请求，并欣赏他们能得到的任何帮助：

[`github.com/sveltejs/kit`](https://github.com/sveltejs/kit)

就像许多开源项目一样，社区和文档可以成就或毁掉一个项目。由于 SvelteKit 背后出色的支持，很难想象一个人们不会不断宣扬 SvelteKit 和 Svelte 的未来。

# 总结

如果你已经读到这儿，那么感谢你一直陪伴着我。我希望这里提供的材料和知识能够对你的 SvelteKit 项目有所帮助。如果你喜欢这本书，那么请与对学习新 JS 框架感兴趣的朋友、同事和熟人分享。作为我的第一本书，这无疑是一次旅程，我在写作过程中学到了很多。如果你对我的更多技术性文本感兴趣，我会在 https://www.closingtags.com 上写关于网页开发和相关技术的文章。如果你用 SvelteKit 做出了什么酷炫的东西，我很乐意听到你的分享。你可以通过我网站上的联系表单联系到我。再次感谢，并期待看到你创造的作品。

# 摘要

就这样，你已经完成了这本书！如果你对 SvelteKit 的各种功能还有疑问，可以查看之前提供的社区资源。你将找到所有扩展你的 SvelteKit 知识和了解社区其他成员所做的事情所需的一切。由于 SvelteKit 与许多其他工具集成得很好，将其融入现有的工作流程应该会非常容易。我期待看到你用它创造的作品。再次感谢！

# 资源

+   Tailwind CSS: [`tailwindcss.com`](https://tailwindcss.com)

+   Svelte Add: [`github.com/svelte-add/svelte-add`](https://github.com/svelte-add/svelte-add)
