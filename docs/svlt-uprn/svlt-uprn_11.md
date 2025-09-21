

# 第十一章：模块和秘密

虽然路由对于 SvelteKit 来说确实很重要，但它远不止于此。在这本书的整个过程中，我们使用了多个不同的 SvelteKit 模块。例如，我们已从 `$app/forms`、`$app/environment` 和 `$app/stores` 等模块中导入绑定，仅举几个例子。但我们尚未探讨这些模块是什么，以及它们是如何工作的。在本章中，我们将简要概述我们之前看到的一些模块以及我们尚未看到的一些模块。我们还将涵盖一些用于管理秘密的模块以及何时使用哪个模块。

在本章中，我们将检查以下内容：

+   SvelteKit 模块摘要

+   保护秘密安全

在涵盖了各种模块，并检查了我们之前存储 GitHub API 秘密的示例之后，我们将清楚地了解何时最好利用每个单独模块的力量。

# 技术要求

本章的完整代码可在 GitHub 上找到：[`github.com/PacktPublishing/SvelteKit-Up-and-Running/tree/main/chapters/chapter11`](https://github.com/PacktPublishing/SvelteKit-Up-and-Running/tree/main/chapters/chapter11)。

# SvelteKit 模块摘要

在前面的章节中，我们使用了几个不同的模块，但只提供了简短的说明。虽然本节的分析也将是简短的，但它应该提供足够广泛的洞察，让潜在的 SvelteKit 开发者熟悉现有模块的工作原理。我们已经遇到了这里列出的几个，但还有一些我们尚未涉及。对于更深入的解释，请参阅本章末尾的资源。

## $app/environment

为了开始对模块的分析，让我们从一个我们相对近期使用过的模块开始。在 *第九章* 中，当我们试图在客户端抛出错误时，我们使用了 `$app/environment` 模块并导入了 `browser`。正如我们预期的那样，根据 SvelteKit 命名约定，从这个模块导出的所有绑定都与应用程序环境相关。这使得通过它们的名称识别每个绑定的目的变得非常简单。例如，我们看到 `browser` 返回一个布尔值，基于它正在运行的运行环境是否是客户端。同样，`building` 和 `dev` 将根据代码是否在构建过程中运行或在我们的开发环境中运行分别返回布尔值。`$app/environment` 的最后一个导出是 `version`。虽然 `version` 的命名仍然很合适，但重要的是要注意，它并不指代 SvelteKit 的版本。相反，它指的是应用程序构建的版本。`version` 的值由构建时的 Unix 时间戳确定。SvelteKit 使用这个字符串值在客户端导航遇到错误时检查新版本，并通过执行完整页面加载来默认使用标准导航实践。

## $app/forms

当我们在 *第四章* 中使用 `$app/forms` 的 `enhance` 时，我们看到了它如何通过在提交表单时在后台发送数据而不是要求整个页面重新加载来 *渐进式增强* 我们的表单提交。如果 `enhance` 的默认行为没有完全满足我们的需求，它可以被自定义。如果我们决定改变其默认行为并提供一个自定义的回调函数，可以使用 `applyAction` 来更新组件 `form` 属性中的数据。我们可以从 `$app/forms` 中导入的最后一个绑定是 `deserialize`，这是一个在反序列化表单提交的响应数据时非常有用的函数。

## `$app/navigation`

如预期，`$app/navigation` 提供了与导航相关的工具。我们之前从这个模块中使用了 `invalidateAll` 在 *第五章* 和 `invalidate` 在 *第九章*。在这两个实例中，我们这样做是为了强制 `load()` 再次运行。从这个模块中还有其他一些有用的绑定，如 `afterNavigate` 和 `beforeNavigate`。这两个绑定在 SvelteKit 生命周期中的特定时间运行回调函数。使用 `afterNavigate`，提供的回调函数在当前组件挂载后以及用户导航到新 URL 后执行。`beforeNavigate` 的回调函数在导航到新 URL 前运行。

从这个特定模块中可用的其他绑定包括 `disableScrollHandling`、`goto`、`preloadCode` 和 `preloadData`。如果我们想改变 SvelteKit 管理浏览器窗口滚动位置的方式，我们可以通过调用 `disableScrollHandling()` 完全禁用它。`goto()` 函数为开发者提供了在导航到另一个 URL 的同时，包括允许管理浏览器历史记录、保持特定元素的关注焦点以及触发 `load()` 函数重新运行的功能。`preloadCode()` 函数允许开发者在使用特定路由之前导入该路由的代码，而 `preloadData()` 将执行相同的功能，同时也会调用路由的 `load()` 函数。当 `<a>` 元素被赋予 `data-sveltekit-preload-data` 属性时，也会发现 `preloadData()` 的相同行为。

## `$app/paths`

当我们需要操作文件路径或为位于子目录中的应用程序创建路由链接时，我们可以依赖 `$app/paths`。`assets` 和 `base` 都将返回一个字符串值，该值可以在 `svelte.config.js` 中的 `config.kit.paths` 下自定义。如果应用程序是从子目录中提供的，我们可以手动将 `{base}` 文本添加到路由之前，以确保我们的应用程序从正确的目录中提供路由，而不是尝试从域名根目录提供路由。`base` 提供的值必须始终是一个以 `/` 开始但永不以 `/` 结尾的根相关路径。相反，`assets` 将始终是应用程序文件的绝对路径。

## $app/stores

SvelteKit 开发者非常慷慨，为开发者提供了一些可读的存储，这些存储可以为我们提供有用的信息。当我们从 `$app/stores` 导入时，我们将能够访问 `navigating`、`page`、`updated` 和 `getStores`。在导航期间，`navigating` 将填充一个包含元数据的对象，例如导航事件开始的位置、目的地以及导航类型。类型可以从表单提交到链接点击，甚至浏览器的前进和后退事件。一旦导航完成，此存储的值将返回 `null`。同时，`page` 作为包含与当前查看页面相关信息的存储。我们曾在 *第四章* 中使用它来显示当前页面的错误消息。`updated` 存储在 SvelteKit 检查是否检测到应用程序的新版本后，将返回一个布尔值。所有这些存储都可以通过调用 `getStores()` 来检索，但这样做并不推荐，并且仅在需要暂停存储订阅直到组件挂载之后时才应这样做。否则，可以通过在存储名称前加上 `$` 符号来自动处理订阅过程来访问所有存储值。

## $service-worker

当构建一个 `$service-worker` 模块时，它只能在服务工作者中使用。当导入绑定时，我们将能够访问 `base`、`build`、`files`、`prerendered` 和 `version`。再次强调，合理的命名约定是我们的救星，并且主要解释了从每个绑定返回的数据类型。我们可以从 `base` 接收一个字符串值，它的工作方式与 `$app/paths` 中的 `base` 类似。即使应用程序位于子目录中，它也会为我们提供应用程序的基本路径。`build` 返回的字符串数组将由 Vite 在构建过程中生成的 URL 组成。另一个字符串数组 `files` 将提供有关静态文件或包含在 `config.kit.files.assets` 中的文件的详细信息。对于预渲染的路径名，我们可以从 `prerendered` 数组中检索字符串。最后，我们可以使用 `version` 在我们的服务工作者代码中确定应用程序版本。

这个模块列表绝对不是全面的。在过度依赖它们之前，建议阅读本章末尾列出的官方 SvelteKit 文档。现在我们已经检查了一些这些模块，让我们继续探讨那些帮助我们保护敏感信息的模块。

# 保护机密信息

当涉及到保护像 API 密钥或数据库凭证这样的机密信息时，SvelteKit 为我们提供了保障。正如我们在 *第九章* 中所看到的，我们可以从 `$env/static/private` 导入 `.env` 机密信息。但这并不是唯一允许我们导入环境变量的模块。在本节中，我们将探讨更多模块以及它们如何帮助我们导入环境变量以及机密信息。

## `$env/static/private`

从我们已使用的模块开始，`$env/static/private` 非常适合导入像 `.env` 文件中指定的或启动运行时设置的变量。例如，如果我们使用 `API_KEY="" node index.js` 命令启动我们的应用程序，`API_KEY` 变量将可用，就像我们在 *第九章* 中导入 `$env/static/private` 后 `GITHUB` 可用一样。在启动应用程序时提供的任何环境变量都将覆盖 `.env` 文件中提供的值。这个模块中的变量在构建时由 Vite 静态捆绑到应用程序代码中。这意味着捆绑的代码更少，但也意味着机密信息将包含在应用程序构建中。我们可以通过以下构建命令来确认这一点：

```js
npm run build
grep -r 'YOUR_API_KEY' .svelte-kit/
```

将 `YOUR_API_KEY` 替换为 GitHub 提供的个人访问令牌将显示我们的构建中注入了 API 键的文本和文件。因此，我们应该谨慎不要将构建提交到版本控制，因为这样做可能会暴露机密信息。幸运的是，这个模块只会在服务器端代码中工作，如果我们尝试将其导入客户端代码，它会导致构建失败。

## `$env/static/public`

我们可以将 `$env/static/public` 视为 `$env/static/private` 的兄弟模块。主要区别在于，这个模块只会导入以 `config.kit.env.publicPrefix` 中设置的值开头的变量。默认情况下，这个值是 `PUBLIC_`。这允许我们将所有机密信息都保存在同一个 `.env` 文件中，同时允许我们指定哪些是安全地暴露给客户端的，哪些不是。例如，以下 `.env` 文件将允许将 `PUBLIC_GITHUB_USER` 导入客户端代码，但不允许 `GITHUB_TOKEN`。尽管这两个值都存在于同一个文件中，但我们可以放心，SvelteKit 不会将我们的机密信息告诉除我们信任的服务器之外的任何人：

`.env`

```js
GITHUB_TOKEN=YOUR_GITHUB_PERSONAL_ACCESS_TOKEN
PUBLIC_GITHUB_USER=YOUR_USERNAME_HERE
```

## `$env/dynamic/private`

正如我们不能将 `$env/static/private` 导入客户端代码中一样，我们也不能将 `$env/dynamic/private` 导入客户端代码中。这两个模块之间的区别在于，虽然 `$env/static` 模块可以访问 `.env` 文件，但 `$env/dynamic` 可以访问适配器指定的环境变量。回想一下*第八章*，当我们将应用程序部署到 Cloudflare 时。当我们这样做时，我们指定了环境变量 `NODE_VERSION`。获取 Node 版本可能对我们来说并不非常有用，但使用 `$env/dynamic/private` 允许我们访问在 Cloudflare Pages 项目设置中设置的其它秘密。这在团队合作中非常有用，因为可以在平台上共享秘密，而不是传递 `.env` 文件。当项目将要部署的平台有自己的设置环境变量的方式时，你很可能要依赖 `$env/dynamic` 来访问它们。

## $env/dynamic/public

按照既定的模式，本模块应该只需要很少的解释。使用 `$env/dynamic/public`，我们只能导入以 `PUBLIC_` 或者在 `config.kit.env.publicPrefix` 中指定的任何值开头的环境变量。这与 `$env/static/public` 的工作方式相同。当然，它与 `$env/static/public` 不同，因为这是一个另一个动态模块，这意味着它是特定于配置的环境适配器的。例如，如果我们把 `PUBLIC_GITHUB_USER` 添加到我们的 Cloudflare Pages 环境变量中，那么我们就可以使用这个模块来访问该环境变量的值。

# 摘要

在 SvelteKit 中有大量的模块可用。希望这个简要概述已经提供了足够的洞察力，以便我们知道如何开始使用每一个模块。在下一章中，我们将探讨一些增强可访问性的最佳实践，这可以带来提高**搜索引擎优化**（**SEO**）的额外好处。

# 资源

+   SvelteKit 模块文档 – [`kit.svelte.dev/docs/modules`](https://kit.svelte.dev/docs/modules)
