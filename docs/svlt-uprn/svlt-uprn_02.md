

# 第二章：配置和选项

在我看来，没有什么比在项目上辛勤工作几周，结果在生产环境中因为一个看似微不足道的配置选项而出现 bug 更令人沮丧的了。这就是为什么本章将努力在您深入研究之前向您介绍可用的各种选项。我知道您已经准备好学习 SvelteKit 了。我保证，我对“真正开始构建东西”的热情与您一样，但我向您保证，早期学习这些概念是有价值的。如果您希望快速开发高性能的 Web 应用程序，那么我相信在学习您特定的应用程序之前学习这些概念对您最有利。

为了开始本章，我们将首先查看如何使用`svelte.config.js`文件中的选项来管理您的项目配置。然后，我们将简要地查看一些使用基本适配器的配置，以便您开始。我们还将查看您在`vite.config.js`文件中可用的选项。在关于 SvelteKit 的书中如此多地讨论 Vite 可能看起来有些奇怪，但如果没有 Vite，SvelteKit 就不会是现在的工具。

在本章中，我们将涵盖以下主题：

+   配置 SvelteKit

+   配置 Vite

在我们讨论了配置及其如何影响您的工具之后，您应该会感到更自在地做出适当的更改，以满足您下一个 SvelteKit 项目的需求。

# 技术要求

本章的完整代码可在 GitHub 上找到：[`github.com/PacktPublishing/SvelteKit-Up-and-Running/tree/main/chapters/chapter02`](https://github.com/PacktPublishing/SvelteKit-Up-and-Running/tree/main/chapters/chapter02)

# 配置 SvelteKit

SvelteKit 项目的核心配置位于`svelte.config.js`文件中。了解您可用的各种选项将使您能够充分利用 SvelteKit。虽然我们无法在如此简短的章节中涵盖所有可用的选项，但我们的目标是涵盖您可能认为有用的选项。有关更多配置选项，请参阅本章末尾的*进一步阅读*部分以获取更多资源。

要开始，请打开编辑器中骨架项目中的`svelte.config.js`文件。请注意，目前它相当简单。基本上，它从`@sveltejs/adapter-auto`包中导入`adapter`函数，在`config`常量的`kit`属性中指定该函数，并最终导出`config`对象。我们还通过`config.kit`属性提供了一个类型注解，我们将在这里添加各种其他属性来自定义配置。它应该看起来像这样：

```js
import adapter from '@sveltejs/adapter-auto';
/** @type {import('@sveltejs/kit').Config} */
const config = {
      kit: {
           adapter: adapter()
       }
};
export default config;
```

`adapter-auto` 包在通过 `npm create svelte@latest` 安装时默认包含。它将在构建时自动适应你的项目，以便可以轻松部署到 Vercel、Cloudflare Pages、Netlify 或 Azure Static Web Apps。因此，`adapter-auto` 被视为 `devDependencies`。这样做将允许你自定义特定于该环境的选项，因为 `adapter-auto` 不接受任何选项。

我们将在后面的章节中详细讨论适配器；我们将介绍如何使用 `adapter-static` 包生成静态网站，以及如何使用 `adapter-node` 在 Node.js 服务器上运行 SvelteKit 应用程序。现在，让我们看看你可能在以下章节中需要的一些其他潜在配置选项。

## alias

虽然通过 `$lib/` 别名访问 `src/lib/` 路径非常有用，但有时你可能想访问其外部的内容。为此，你可以在 `config.kit` 中添加 `alias` 属性。例如，如果你的项目有一个用于连接和管理数据库的特定文件，你可以创建一个别名以简化导入。或者，你可能想轻松地从常用文件夹中访问图像。以下代码片段展示了这些概念：

src/db.js

```js
const db = {
     connection: 'DB connection successful!',
     // database logic goes here…
};
export default db;
```

svelte.config.js

```js
import adapter from '@sveltejs/adapter-auto';
/** @type {import('@sveltejs/kit').Config} */
const config = {
      kit: {
           adapter: adapter(),
           alias: {
                 db: '/src/db.js',
                 img: '/src/images'
                 }
       },
};
export default config;
```

src/routes/+page.svelte

```js
<script>
      import db from 'db';
      import url from 'img/demo.png';
      let status = db.connection;
</script>
<p>{status}</p>
<img src={url}>
```

在这个例子中，我创建了一个名为 `connection` 的模拟数据库对象。在 `svelte.config.js` 文件中，我添加了两个别名——一个用于模拟数据库对象，另一个指向一个图像文件夹。然后，在 `+page.svelte` 文件中，从这两个别名中进行了导入，以说明如何使用别名导入单个文件，以及如何从目录中导入特定文件。

此示例未能考虑通常用于数据库操作所需的任何异步代码。直接从路由调用数据库也不是最佳实践，但我们在后面的章节中会回到如何做到这一点。虽然这个例子相当基础，但它的目的是展示能够为特定文件和目录添加自己的别名是有用的。

## appDir

此属性默认为 `_app` 值，并用于确定导入的资产（如 CSS 和 JS）将从构建的应用程序中提供。

## csp

为了帮助保护你的应用程序及其用户免受恶意 `config.kit.csp` 的侵害，以下是一些措施：

+   `mode: 'hash' | 'nonce' | 'auto'`：这指定了是否应使用哈希或非 ces 来强制执行 CSP。使用 `auto` 将为动态渲染的页面使用非 ces，为预渲染的页面使用哈希。

+   `directives`: 在此处指定的指令将被添加到 `Content-Security-Policy` 头部。有关可用选项的完整列表，请参阅官方 SvelteKit 文档。

+   `reportOnly`：这个选项用于报告错误，但仍然允许脚本加载。这里指定的指令将被添加到`Content-Security-Policy-Report-Only`头中。有关可用选项的完整列表，请参阅官方 SvelteKit 文档。

虽然这可以帮助防止从外部来源加载脚本，但您可能故意将来自`%sveltekit.nonce%`的脚本包含到包含脚本的`nonce`属性中，如下所示：

```js
<script src='YOUR_EXT_URL/script.js' nonce='%sveltekit.nonce%'>
```

## CSRF

为了帮助保护您的应用程序免受来自另一个源对您的应用程序的`POST`请求。如果您需要禁用它，可以通过将`config.kit.csrf.checkOrigin`设置为`false`来实现，如下所示：

```js
const config = {
  kit: {
    adapter: adapter(),
    csrf: {
      checkOrigin: false
    }
  },
};
```

## 环境变量

如果您在项目中包含了`.env`文件，您可以使用以下选项进一步指定哪些可以公开以及它们存储在哪个目录中：

+   `dir`：一个默认为“`.`”（项目根目录）的字符串值。

+   `publicPrefix`：一个默认为`PUBLIC_`的字符串值。这个前缀表示`.env`文件中定位的环境变量是否安全地暴露给客户端代码。这些环境变量随后可以通过从整个应用程序中的`$env/static/public`模块导入来访问。例如，假设我们正在处理项目`base`目录中的一个`.env`文件。添加`PUBLIC_EXTERNAL_API=https://api.nasa.gov/planetary/apod`值将在客户端代码中可导入，而`INTERNAL_API=AN_INTERNAL_IP_ADDRESS`值则只能在服务器代码中导入。

## 预渲染

在您的应用程序中，很可能至少有一部分内容是可以预渲染的——也就是说，它将为每个访问它的用户显示完全相同的 HTML、CSS 和 JS。一个常见的例子是“关于”页面。是否希望特定页面进行预渲染将在后面的章节中讨论，但为了进一步自定义预渲染，请考虑以下可以添加到`config.kit.prerender`中的选项：

+   `entries`：一个字符串数组，用于指定每个应该进行预渲染的路由。默认值是`*`特殊字符，将包括所有非动态路由。

+   `concurrency`：可以同时预渲染的页面数量。由于 JavaScript 是单线程的，因此只有在您的项目需要在预渲染过程中进行网络请求时，此选项才会生效。

+   `crawl`：此选项默认为`true`。它告诉 SvelteKit 是否应该预渲染`entries`数组中找到的页面。

+   `handleHttpError`：此选项默认为`fail`。其他选项包括`ignore`、`warn`或一个特定的 SvelteKit `details`对象，该对象实现了`PrerenderHttpErrorHandler`。如果在预渲染过程的任何点上失败，此选项将确定如何处理它。

+   `handleMissingId`：这与 `handleHttpError` 相同，但它接受的 `details` 对象是从 `PrerenderMissingIdHandler` 实现的。链接到另一页上的特定点是通过 URL 中的哈希实现的——一个 `#` 字符后跟目标页面上元素的 ID。如果目标页面上找不到指定 ID 的元素，那么此选项将决定构建过程应该如何行为。

+   `origin`：默认为 [`kit.svelte.dev/docs/page-options`](https://kit.svelte.dev/docs/page-options)。如果你想在渲染内容中使用应用程序的 URL，在这里指定它可能会有所帮助。

希望这个选项列表不会让你感到过于繁重，并为你提供了如何根据需要自定义 SvelteKit 的见解。SvelteKit 将继续发展，因此始终建议查看官方文档，因为新功能可能会在未来的版本中添加或删除。

现在我们已经查看了一些可用于自定义 SvelteKit 的选项，让我们来看看如何修改 Vite 的行为。与 SvelteKit 一样，Vite 对项目应该如何配置有自己的看法。这些看法对于广泛的用例非常有用，但总会有需要调整的情况。

# 配置 Vite

如前所述，Vite 是使 SvelteKit 成为可能的项目构建工具，因此了解如何配置 Vite 与了解 SvelteKit 一样重要。话虽如此，Vite 具有高度的可配置性，因此为了简洁（以及您的注意力集中），我们将仅限于提供一个对可用选项的高级概述。本节的目的不是提供一个详尽的列表，而是快速浏览一下您可用的选项。如需进一步阅读，请参阅本章末尾的资源。

在其核心，SvelteKit 只是一个 Vite 插件。显然，它不仅仅是一个插件，但当你在一个新创建的 SvelteKit 项目中打开 `vite.config.js` 文件时，你就会明白我的意思。类似于 `svelte.config.js`，这个配置导出了一个 `config` 常量，其中设置了一些属性——导入的 SvelteKit 插件和要包含的测试。如果你在 `create@svelte` 提示中回答了“否”关于测试的问题，你可能看不到 `test` 属性。以下代码片段展示了骨架应用程序的配置：

```js
import { sveltekit } from '@sveltejs/kit/vite';
const config = {
      plugins: [sveltekit()],
      test: {
           include: ['src/**/*.{test,spec}.{js,ts}']
     }
};
export default config;
```

## 插件

从我们的 Vite 配置的`plugins`属性开始似乎很合理，因为这本书的整个前提都是基于一个特定的插件。随着 Vite 变得越来越受欢迎，其采用率也在上升，其生态系统正在扩展，随之而来的是更多的插件。许多这些插件都是由社区开发的，用于解决常见问题。由于 Vite 扩展了 Rollup，许多 Rollup 插件也可以与 Vite 无缝工作，尽管并非所有。Vite 开发团队维护了几个官方的 Vite 插件，包括一个支持旧版浏览器支持的插件。许多功能和功能可以通过这些插件添加到你的应用程序中。一旦你找到了满足你要求的插件并安装了它，你将需要将其导入并包含在`config.plugins`数组中，就像之前展示的 SvelteKit 一样。插件的配置各不相同，但通常，选项是通过导入函数调用的参数传递的。要查找更多可用的 Vite 插件，请查看 GitHub 上的 Awesome Vite 项目[`github.com/vitejs/awesome-vite`](https://github.com/vitejs/awesome-vite)。

## server

由于 Vite 正在运行你用 SvelteKit 工作的开发服务器，你可能需要更改该服务器的一些操作方式。例如，你可能需要更改默认端口，通过另一个服务器代理，添加对 HTTPS 的支持，或者甚至禁用 HMR 以使用旧浏览器进行测试。在这些情况下以及更多情况下，你将想要相应地调整`config.server`。

## build

当你在生产环境中构建应用程序时，你可能需要做一些调整。无论你需要确保应用程序满足特定的浏览器兼容性，还是你想自定义 Rollup 的选项，这些设置都可以在`config.build`下找到。

## preview

一旦你为你的生产环境构建了应用程序，你应该使用 Vite 的预览功能对其进行测试。可以通过管理`config.preview`来更改预览服务器的选项。

## optimizeDeps

有时，你可能想测试多个依赖项或依赖项的版本与你的代码一起工作，以查看它们对你的项目有多好。要包括或排除依赖项以进行预捆绑，或进一步修改`esbuild`使用的选项，请开始配置`config.optimizeDeps`。

## ssr

如果你需要防止依赖在你的服务器环境中运行，你可能需要管理在`config.ssr`下可用的选项。

# 摘要

在本章中，我们介绍了 SvelteKit 可用的许多配置选项。我们讨论了`config.kit.alias`可能如何用于导入频繁访问的文件的一个例子。虽然我们没有涵盖所有选项，但我们简要地看了`appDir`、`csp`、`csrf`、`env`和`prerender`选项以及它们如何影响我们的应用程序。

我们也浏览了一些在 `vite.config.js` 中可用的配置选项。大部分情况下，SvelteKit 会根据安装提示中的选择设置一个适合我们的配置，但对于进一步的调整和插件，我们现在知道在哪里开始查找官方 Vite 文档。

在下一章中，我们将讨论一些现有的网络标准以及 SvelteKit 的设计如何考虑到这些标准。存在于网络浏览器中的现有标准正迅速在其他环境中得到支持，包括基于 Node 的环境，甚至是 Deno 和 Cloudflare Workers。通过利用这些标准而不是在它们之上构建新的标准，SvelteKit 对更多开发者变得可访问。

# 进一步阅读

+   **SvelteKit 文档** – 在你的故障排除之旅中，官方 SvelteKit 文档应该是你的第一站：[`kit.svelte.dev/docs`](https://kit.svelte.dev/docs)

+   **出色的 Vite** – 一系列官方和社区维护的资源或插件，使使用 Vite 进行开发变得愉快：[`github.com/vitejs/awesome-vite`](https://github.com/vitejs/awesome-vite)

+   **Vite 文档** – 需要修改你的开发服务器或构建过程？请在此查看官方推荐：[`vitejs.dev/config/`](https://vitejs.dev/config/)
