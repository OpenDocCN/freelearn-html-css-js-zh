

# 钩子和错误处理

虽然从任何页面发起 API 请求可能非常有用，但想象一下在 *每个* 页面上尝试对用户进行外部 API 验证的麻烦。可能可以创建一个自定义助手，为每个请求添加特定的头或 cookie 来协助这一过程。幸运的是，SvelteKit 提供了在整个框架中操作请求和响应对象的方法。它是通过所谓的 **钩子** 来实现的。这些钩子可以非常强大地管理进出我们应用的数据。它们还可以帮助管理错误。由于我们之前对错误处理的接触非常简短，所以我们将在介绍钩子之后更详细地研究错误处理。

在本章中，我们将涵盖以下主题：

+   使用钩子

+   错误处理

作为实际示例，我们将构建一个简单的界面，允许我们 *star* GitHub 上的官方 SvelteKit 仓库。您将需要验证您的个人账户，但如果您在 GitHub 上没有账户，请不要担心，因为概念足够简单，可以轻松跟随。到练习结束时，我们将涵盖 SvelteKit 中可用的钩子，以及如何利用它们来帮助管理错误。

# 技术要求

本章的完整代码可在 GitHub 上找到：[`github.com/PacktPublishing/SvelteKit-Up-and-Running/tree/main/chapters/chapter09`](https://github.com/PacktPublishing/SvelteKit-Up-and-Running/tree/main/chapters/chapter09)。

建立示例需要 GitHub 账户：[`github.com/signup`](https://github.com/signup)。

# 使用钩子

与其他未命名的 JS 框架不同，SvelteKit 将需要记住的钩子列表保持简短和简单。在撰写本文时，只有两种类型的钩子 – `+page.server.js` 只在服务器上运行，而 `+page.js` 则在服务器或客户端上运行。服务器和共享钩子都放在 `src/` 目录中，无论是在 `src/hooks.server.js` 还是 `src/hooks.client.js`，这取决于我们打算在哪个环境中运行钩子。我们将把这个部分分解成以下子部分：

+   服务器钩子

+   共享钩子

到本节结束时，您将能够修改所有进入和离开您的 SvelteKit 应用程序的请求。

## 服务器钩子

只能在服务器上运行的钩子有 `handleFetch()` 和 `handle()`。正如我们所期望的，`handleFetch()` 具有操作 SvelteKit 内置的 `fetch()` 方法所发起请求的能力，这个方法可以在 `load()` 或通过操作调用。另一个钩子 `handle()`；可以在 SvelteKit 的路由器接收请求时操作数据。可以这样理解，`handleFetch()` 操作的是即将离开应用的数据，而 `handle()` 操作的是进入应用的数据。作为服务器钩子，两者都可以添加到 `src/hooks.server.js`。

为了开始本章的示例，展示钩子有多么有用，让我们先设置一些东西。首先，我们需要一种方法来使用 GitHub 认证我们的个人账户。我们将通过向适当的 GitHub API 端点发送一个**个人访问令牌**来实现这一点。一旦生成，这个令牌就可以添加到 HTTP 请求头中。非常重要的一点是，我们要像对待密码一样对待令牌，因此我们将通过环境变量安全地导入这个令牌，这将在后面的章节中进一步讨论。

要生成你的令牌，请访问[`github.com`](https://github.com)并登录。然后，你可以通过点击右上角的个人头像并选择`SvelteKit star repo`、一个过期日期和完整的*repo*范围来导航到你的个人资料设置。完成操作后，点击**生成令牌**并复制提供的值。

然后，我们可以打开我们的 SvelteKit 项目，并在项目的根目录中创建一个`.env`文件。在这个文件中，我们将添加我们的令牌，如下所示：

`.env`

```js
GITHUB=YOUR_GITHUB_TOKEN_HERE
```

你可以保存并关闭文件。我们将在后面的章节中详细介绍如何管理秘密，但到目前为止，只需知道我们已经给我们的令牌起了一个名字，我们可以用这个名字将其导入到我们的代码中。

接下来，让我们创建一个新的路由并将其添加到`Nav.svelte`中：

`src/lib/Nav.svelte`

```js
<nav>
  <ul>
    <li><a href='/'>Home</a></li>
    ...
    <li><a href='/github'>GitHub</a></li>
  </ul>
</nav>
```

再次强调，这是一个简单的添加操作，我们为链接添加了标记，并将其添加到我们不断增长的链接列表中。一旦我们为新的路由添加了标记，我们就会创建一个适当的`+page.svelte`文件来渲染这个路由。这个页面将包含一个简单的表单，有按钮允许我们*星标*和*取消星标*GitHub 上的 SvelteKit 仓库，以及一些显示仓库当前有多少星标的文本：

`src/routes/(app)/github/+page.svelte`

```js
<script>
  import { enhance } from '$app/forms';
  import { invalidate } from '$app/navigation';
  export let data;
  export let form;
  const reload = () => {
    invalidate('https://api.github.com/repos/');
  };
</script>
{#if form && form.message }
  {form.status}
  {form.message}
{/if}
<p>
  Stargazers on the official SvelteKit repo: {data.repo.stargazers_count}
</p>
<form method='POST' use:enhance>
  <button formaction='?/star' on:click={reload}>Star</button>
  <button formaction='?/unstar' on:click={reload}>Unstar</button>
</form>
```

这里发生了很多事情，但我们之前都见过。从文件的末尾开始，我们可以看到这个页面包含一个`<form>`元素，由两个按钮组成。一个按钮调用`/github?/star`操作，另一个调用`/github?/unstar`操作。表单使用`enhance`模块，这意味着请求在后台进行，不会触发页面刷新。表单中的每个按钮都调用分配给常量`reload`的函数表达式。这个匿名函数利用了`invalidate`模块，它强制任何使用`fetch()`方法的`load()`函数重新运行指定的 URL。这很有用，因为显示仓库有多少星的段落标签将相应地更新。最后要注意的代码部分是 Svelte 的`{#if}`指令，它将通知我们请求的状态。

保存`+page.svelte`后，我们需要继续处理`+page.server.js`，因为那里将包含我们大部分的逻辑。重要的是，它不会包含任何与认证相关的代码，因为所有这些都将驻留在钩子中：

`src/routes/(app)/github/+page.server.js`

```js
const star_url = 'https://api.github.com/user/starred/sveltejs/kit';
export function load({ fetch }) {
  const repo = fetch( 'https://api.github.com/repos/sveltejs/kit' )
    .then( response => response.json() );
  return { repo };
}
export const actions = {
  star: async({ fetch }) => {
    const response = fetch(star_url, {
      method: 'PUT',
      headers: {
        'Content-Length': '0',
      }
    })
    .then(response => {
      const status = response.status;
      return {
        status: status,
        message: (status === 204 ? 'Success!' : 'Error')
      }
    });
    return response;
  },
  unstar: async({ fetch }) => {
    const response = fetch(star_url, {
      method: 'DELETE'
    })
    .then(response => {
      const status = response.status;
      return {
        status: status,
        message: (status === 204 ? 'Success!' : 'Error')
      }
    });
    return response;
  },
}
```

这里有很多事情在进行，但同样，这并不是我们之前没有见过的。第一行声明了一个常量 `star_url`，我们可以在之后的 `fetch()` 请求中引用它。这是我们用来通知 GitHub 我们想要 *star* 或 *unstar* 仓库的端点。紧接着的代码块创建了一个熟悉的 `load()` 函数。这个端点实际上不需要认证，并将返回有关指定仓库的一般信息。

从那里，我们可以检查我们创建的两个操作 – *star* 和 *unstar*。这两个操作都解构了 SvelteKit 附加的 `fetch()` 方法。然后它们都使用 `fetch()` 向 `star_url` 端点发送请求，但根据 GitHub API 规范，它们在使用的 HTTP 方法类型上有所不同。我们还为 *star* 请求添加了一个额外的头部，因为官方 GitHub API 文档规定，对于这个特定的请求，`Content-Body` 应该设置为 `0`。这两个操作随后返回一个对象，包含请求状态和一个消息，告知我们请求是否成功通过，或者是否收到了错误。

在这一点上，我们可以在浏览器中导航到 `http://localhost/github` 并查看官方 SvelteKit 仓库的星标数量。然而，尝试为其添加星标将会导致错误，因为我们没有使用 GitHub API 的适当权限。为了进行认证，我们需要在网络的请求头部提供我们的访问令牌。虽然我们可以在每个操作的 `+page.server.js` 文件中这样做，但如果我们要构建更多需要与 GitHub 进行认证的路由，这会很快变得麻烦。例如，如果我们想构建允许我们读取和评论问题的功能呢？相反，我们可以使用 `handleFetch()` 来捕获所有即将离开我们应用程序的 `fetch()` 请求，并在一个位置进行认证。为此，我们需要创建 `src/hooks.server.js`：

src/hooks.server.js

```js
import { GITHUB } from '$env/static/private';
export async function handleFetch( { request, fetch  } ) {
  if (request.url.startsWith('https://api.github.com/')) {
    request.headers.set('Accept', 'application/vnd.github+json');
    request.headers.set('Authorization', 'Bearer ' + GITHUB);
    request.headers.set('X-GitHub-Api-Version', '2022-11-28');
  }
  return fetch(request);
}
```

我们首先导入之前创建的个人访问令牌。再次强调，我们将在后面的章节中详细探讨其工作原理，所以现在只需确认我们已经导入了保存在 `.env` 中的令牌。然后，我们为 `handleFetch()` 函数创建函数定义。由于 Fetch 是一个异步 API，因此我们需要指定这个函数也将是 `async` 的。我们随后解构了 `RequestEvent` 对象以提取 `request` 和 `fetch`。在钩子中，我们检查请求的 URL 是否为 [`api.github.com/`](https://api.github.com/)。如果是，我们将设置必要的 `Accept`、`Authorization` 和 `X-GitHub-Api-Version` 头部。虽然 `Accept` 和 `X-GitHub-Api-Version` 是简单的预定字符串，但 `Authorization` 头部的值是连接字符串 *bearer* 以及导入的个人访问令牌。

现在，我们可以在浏览器中打开 `http://localhost/github` 并点击 *star* 按钮。因为我们已经在 GitHub API URL 上使用了 `invalidate()`，所以我们应该观察到计数增加一个。SvelteKit 的开发者们确实赢得了我们的点赞，但如果您对这个简单的慈善行为感到不舒服，尝试点击 *unstar* 按钮并再次观察变化。

需要注意的是，`handleFetch()` 只会钩入 SvelteKit 提供的 `fetch()` 函数，而不是标准的 `fetch()` 函数。这可以通过从 `src/routes/(app)/github/+page.server.js` 中的 `*star*` 和 `*unstar*` 动作中移除解构的 `fetch` 参数来观察到。然后，尝试标记仓库将返回 GitHub API 的 *未授权* 错误，因为除非我们使用 SvelteKit 提供的 `fetch()` 方法，否则 `handleFetch()` 钩子不会被调用。这也可以通过从 `+page.server.js` 中的 `load()` 函数中移除解构的 `fetch` 来观察到。假设您没有超过 GitHub 的速率限制，标准的 `fetch()` 函数将继续提供数据，因为调用端点不需要身份验证。正因为如此，强烈建议尽可能使用 SvelteKit 的 `fetch()`。

如我们所见，能够操纵离开我们应用程序的请求是非常强大的，但关于进入我们平台的请求呢？为此，我们需要利用 `handle()`。为了这个例子，我们将通过添加 `src/hooks.server.js` 的 `handle()` 钩子来演示：

`src/hooks.server.js`

```js
import { GITHUB } from '$env/static/private';
export async function handleFetch( { request, fetch  } ) {
  if (request.url.startsWith('https://api.github.com/')) {
    request.headers.set('Accept', 'application/vnd.github+json');
    request.headers.set('Authorization', 'Bearer ' + GITHUB);
    request.headers.set('X-GitHub-Api-Version', '2022-11-28');
  }
  return fetch(request);
}
export async function handle({ event, resolve }) {
  event.setHeaders({'X-NOT-FROM-GITHUB': 'our value'});
  const response = await resolve(event);
  response.headers.set('X-ANOTHER-HEADER', 'something else');
  return response;
}
```

在这个版本中，我们在文件末尾添加了 `handle()` 函数的定义。这个函数接受一个 `RequestEvent` 对象以及一个 `resolve()` 函数。在我们的版本中，我们添加了一个自定义的头部，该头部将被添加到我们应用程序内的所有请求中。这个头部是 `X-NOT-FROM-GITHUB`，并将包含 *我们的值*。一旦头部被添加到 `event` 对象中，我们就通过将事件传递给 `resolve()` 并返回承诺来完成请求。我们还添加了另一个方法来演示如何在请求完成后使用 `resolve()` 修改头部。可以通过在应用程序中的任何页面上打开浏览器，打开 **开发者工具**，导航到 **网络** 选项卡，并刷新页面来观察这些头部。打开第一个请求将显示我们在 **响应头部** 中的自定义头部和值。

当然，我们不仅限于设置头。选择头只是为了演示目的。我们还可以通过将 `event.locals` 设置为我们选择的任何对象来设置将在 `load()` 和服务器页面中可用的数据。我们甚至可以利用 `event.cookies` 来修改 cookie 值。然而，如果我们使用 SvelteKit 的内置 `fetch()` 方法，只要它位于有权访问 cookie 的域中，这些 cookie 将自动在应用程序中传递。最后，需要注意的是，`handle()` 和 `handleFetch()` 都将在渲染或预渲染过程中运行。如果你尝试使用 `adapter-static` 生成静态网站，这一点尤其重要要记住。

正如我们刚刚演示的，服务器钩子可以在需要自定义请求和响应头时成为一个强大的工具。虽然修改头数据只是它们的一个用例，但我们提供的示例应该突出它们可以解决的许多可能的解决方案。接下来，让我们看看我们可以在客户端和服务器上使用的钩子。

## 共享钩子

在刚刚介绍了服务器环境中的两个钩子之后，我们现在在客户端环境中只有 `handleError()` 可用。这个钩子对于捕获未通过 SvelteKit 的 `error` 模块抛出的错误非常有用——也就是说，任何关键的应用程序错误，或者使用 `throw new Error()` 抛出的错误。记录应用程序错误非常有用。它还允许我们控制用户在遇到应用程序的严重问题时会看到什么。为了演示共享钩子的用法，让我们将其添加到 `src/hooks.server.js` 和 `src/hooks.client.js` 中。在这个例子中，我们将服务器上的错误记录到文件中，并在客户端使用 `console.log()` 记录错误：

src/hooks.client.js

```js
export async function handleError({ error, event }) {
  console.log('client handled error' + error.message);
  console.log(event.url);
  return {message: 'Whoops, looks like you found an error! Sorry about that.'};
}
```

在这个例子中，我们使用 `console.log()` 在浏览器控制台中显示各种消息。如果 `handleError()` 能够在客户端遇到任何意外错误时运行，那将是非常好的；然而，实际情况并非如此，因为它只有在客户端的 `load()` 过程中或客户端渲染时遇到错误才会运行。因此，我们需要在另一个位置抛出错误以观察其效果。为了触发这个错误，我们将对 `src/routes/(site)/fetch/+page.js` 进行一些小的修改：

src/routes/(site)/fetch/+page.js

```js
import { browser } from '$app/environment';
const key = 'DEMO_KEY'; // your API key here
export const prerender = true;
export function load() {
  if(browser) {
    throw new Error('in the browser');
  }
  const pic = fetch(`https://api.nasa.gov/planetary/apod?api_key=${key}`)
    .then(response => {
      console.log('got response');
      return response.json();
    });
  return {pic};
}
```

我们做的第一个修改是从 `$app/environment` 中导入 `browser` 模块。一旦完成，我们检查正在运行的代码是否在浏览器中，然后抛出一个错误，其余的代码保持不变。当我们导航到应用程序中的 `/fetch` 路由时，我们将看到来自 `src/hooks.client.js` 的消息输出。通过这个钩子，我们可以确保在客户端错误期间不会输出任何敏感数据。我们还可以相应地调整返回的错误对象中的消息。

要在服务器上演示 `handleError()`，我们可以将错误消息写入日志文件，这样开发人员或安装在服务器上的守护进程可以在稍后轻松阅读：

src/hooks.server.js

```js
import { GITHUB } from '$env/static/private';
import fs from 'fs';
export async function handleFetch( { request, fetch  } ) {
  if (request.url.startsWith('https://api.github.com/')) {
    request.headers.set('Accept', 'application/vnd.github+json');
    request.headers.set('Authorization', 'Bearer ' + GITHUB);
    request.headers.set('X-GitHub-Api-Version', '2022-11-28');
  }
  return fetch(request);
}
export async function handle({ event, resolve }) {
  event.setHeaders({'X-NOT-FROM-GITHUB': 'our value'});
  const response = await resolve(event);
  response.headers.set('another', 'custom value');
  return response;
}
function today() {
  const current = new Date();
  return current.getDate() + "-" +
         current.getDay() + "-" +
         current.getFullYear() + " " +
         current.getHours() + ":" +
         current.getMinutes() + ":" +
         current.getSeconds();
}
export async function handleError({ error, event }) {
  const log = today() + ' ' + error.message + ' @ ' + event.request.url;
  fs.appendFile('./app.log', log + '\n', (err) => {
    if(err) {
      console.log(err);
    }
  });
  return {
    error: error.message
  };
}
```

在这个例子中，首先要注意的更改是我们导入了 `fs` 模块。这是一个用于访问文件系统的 Node 特定模块。这值得注意，因为包含它将阻止我们在其他环境中构建我们的应用程序。接下来要注意的更改是我们添加了 `today()` 函数，该函数返回当前日期和时间作为字符串。理想情况下，这个函数应该位于 `utilities` 文件夹中，可能位于 `src/lib/`，但在这个演示中，我们可以将其包含在这里。我们还添加了 `handleError()` 到文件的末尾。它接受一个 `error` 对象以及一个 `event` 对象。然后我们将 `today()` 函数的输出与错误消息以及错误发生的位置信息连接起来。然后使用 `fs.appendFile()` 将所有内容写入我们的日志文件。如果在写入文件时遇到错误，我们将将其输出到控制台。最后，我们返回一个包含错误消息的对象。

要触发 `handleError()`，我们可以在我们的操作或 `load()` 中抛出一个错误。精确的位置并不特别重要，但我们可以通过在 `+page.server.js` 中添加 `throw new Error('我们的自定义错误');` 来看到我们的日志功能在行动。实际上，尝试从我们迄今为止创建的各种 `load()` 函数或操作中抛出错误。你甚至可以尝试与请求提供的数据进行实验。例如，记录各种标题可能很有用，例如客户端 User-Agent，因为当调试浏览器兼容性时，这些信息可能很有帮助。一旦你抛出了几个错误，打开项目目录根目录下的 `app.log` 并观察输出。这个简单的日志机制可以根据项目的需求进行定制。

通过利用 `handleError()`，我们可以轻松地启动自己的日志机制。我们还可以自定义客户端渲染或 `load()` 期间显示的错误消息，这对于我们的应用程序是单页应用时特别有帮助。当然，`handleError()` 非常有用，但它只对意外错误有帮助。我们应该如何管理我们完全预期的错误呢？

# 错误处理

有时，讨厌的用户试图访问他们不应该访问的资源。或者，也许他们只是用户，没有注意到他们点击的地方。无论如何，我们每个人都至少遇到过这样的用户。作为开发者，我们最好确保我们的代码在用户不可避免地遇到错误时给出有意义的消息。为了我们的理智和未来的自己，我们应该在 SvelteKit 中讨论错误。

我们使用 `handleError()` 创建的所有错误都被视为 *意外错误*。这是因为我们没有使用 SvelteKit 的错误模块。使用此模块创建的错误被视为 *预期错误*。通过导入模块，我们可以向 SvelteKit 发送自定义的状态码和错误消息，然后这些消息可以被 `+error.svelte` 组件捕获和使用。这使我们能够对用户显示的消息有更大的控制权。

在 *第四章* 中，我们简要探讨了路由机制将如何与最近的 `+error.svelte` 模板交互。然而，我们并没有真正讨论 `error` 模块或如何使用它。首先，它需要从 `@sveltejs/kit` 中导入。一旦导入，我们就可以使用 `throw error()` 抛出错误，并将两个参数传递给函数。第一个参数是一个整数，表示分类错误的 HTTP 状态码。例如，`401` 代码表示一个 *客户端未授权* 错误。传递给 `error()` 的第二个参数是一个包含 `message` 属性的对象，其中包含我们想要传达的消息。

为了看一个例子，让我们对我们的先前代码做一些修改：

src/routes/(app)/github/+page.server.js

```js
import { error } from '@sveltejs/kit';
const star_url= 'https://api.github.com/user/starred/sveltejs/kit';
export function load({ fetch }) {
  throw error(401, {
    message: 'You don\'t have permission to see this!',
    id: crypto.randomUUID()
  });
  const repo = fetch( 'https://api.github.com/repos/sveltejs/kit' )
    .then( response => response.json() );
  return { repo };
}
export const actions = {
    // omitted for brevity
}
```

在这个例子中，我们从 `@sveltejs/kit` 中导入了 `error` 模块。然后在 `load()` 函数中，我们立即使用该模块抛出一个错误。对于我们的错误，我们提供了 `401` 状态码和一个包含 `message` 属性的对象，以及一个自定义的错误 `id`。同样，这有助于错误报告和日志记录，因为我们可以用我们想要提供的信息自定义错误对象。如果我们不想创建整个对象，我们可以提供一个包含错误消息的字符串，甚至只是 HTTP 状态码！我们将保持此文件中的其余代码不变。

而不是在错误周围构建传统的 `try catch` 语句，我们将让 SvelteKit 通过提供一个定制的 `+error.svelte` 组件来处理捕获：

src/routes/(app)/github/+error.server.js

```js
<script>
  import { page } from '$app/stores';
</script>
{$page.error.message}
{#if $page.error.id}
  <p>
    Error ID: {$page.error.id}
  </p>
{/if}
```

要显示我们的错误消息，我们必须利用 Svelte 存储库 `page` 中的数据。这个存储库是从 `$app/stores` 导入的，并且可以通过在存储库名称前加上 `$` 符号来访问其内部数据。`page` 存储库包含有关访问页面的各种属性和数据。然后我们显示提供的错误消息，接着是一个 Svelte `{#if}` 指令，用于显示错误 ID，如果存在的话。

然后，我们可以导航到 `http://localhost/github` 并查看我们新的错误消息以及错误 ID 号码。为了玩转这个示例并了解更多关于错误处理的知识，尝试将 `src/routes/(app)/github/+error.svelte` 移动到 `src/routes/(app)/+error.svelte`。如果你将它完全移动到 `src/routes/+error.svelte` 会发生什么？你应该在自己的项目中尝试这个操作，简而言之，Svelte 的路由机制足够强大，能够将错误传递到它找到的最近的 `+error.svelte` 组件。

在探索了错误处理之后，你现在应该能够向应用程序用户展示与他们遇到的错误相关的相关信息。以信息化的方式这样做应该会最小化处理支持票证所需的时间。

# 摘要

在探索了 SvelteKit 中可用的各种钩子以及错误处理之后，我们已经涵盖了大量的内容。我们首先看到了如何通过在 `handleFetch()` 钩子中更改熟悉的 `RequestEvent` 对象来修改利用 SvelteKit 内置 `fetch()` 的出站 `fetch()` 请求。我们还看到了如何通过 `handle()` 调整进入我们应用程序的数据。然后，我们探讨了 `handleError()` 共享钩子及其如何被用来构建丰富的日志机制或与另一个服务结合使用。最后，我们回到如何使用 SvelteKit 的错误路由设备来管理预期错误，这些设备允许我们通过自定义 Svelte 组件来自定义外观和感觉。

现在我们已经牢固掌握了如何管理错误，接下来我们将进入下一章，评估我们如何最好地管理静态资产。

# 资源

+   GitHub API 文档：[`docs.github.com/en/rest`](https://docs.github.com/en/rest)
