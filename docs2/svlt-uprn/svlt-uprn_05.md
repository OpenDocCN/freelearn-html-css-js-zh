

# 深入探讨数据加载

每个创建过的应用程序都是由数据驱动的。没有数据来处理，应用程序实际上是没有用的。这就是为什么开发者对如何管理他们应用程序的数据检索有一个牢固的理解非常重要。当使用 SvelteKit 时，这是通过在页面或布局文件中导出`load()`函数来完成的。

在上一章中，我们简要介绍了`load()`。在本章中，我们将通过讨论其工作原理以及查看更多实际、真实世界的示例来进一步分析它。我们将创建一个仅在客户端强制执行`load()`的示例，并介绍使用`load()`时需要记住的一些关键细节。我们还将使用`load()`在布局中展示它如何使数据在我们的应用程序中可移植。最后，我们将查看一个示例，展示如何利用服务器`load()`函数提供的一些数据，这些数据在通用`load()`函数中不可用。

本章我们将涵盖以下主题：

+   在客户端加载

+   在布局中加载

+   解构 RequestEvent

完成本章学习后，你将能够熟练地在 SvelteKit 应用程序中以各种方式加载数据。

# 技术要求

本章的完整代码可在 GitHub 上找到：[`github.com/PacktPublishing/SvelteKit-Up-and-Running/tree/main/chapters/chapter05`](https://github.com/PacktPublishing/SvelteKit-Up-and-Running/tree/main/chapters/chapter05).

# 在客户端加载

在上一章讨论*创建服务器页面*时，我们介绍了从`+page.js`导出的`load()`函数将在客户端和服务器上运行。当我们想要确保`load()`只在服务器上运行时，我们将它移动到`+page.server.js`。但如果你正在尝试构建一个离线准备好的应用程序呢？你可能正在构建一个仅在客户端上运行的`load()`函数，而不是在服务器上。当`+page.js`中的`load()`函数在两个环境中运行时，我们如何做到这一点？

再次回想一下上一章中*创建服务器页面*部分，我们讨论了页面选项，并且你会记得`ssr`选项。当导出时，这个常量将禁用或启用`+page.js`中的`load()`函数，使其仅在客户端运行，我们可以添加`export const ssr = false;`。让我们回到我们的`fetch`示例*第三章*并修改它以展示这一点。

在进行此调整之前，请确保`console.log('got response')`函数仍然存在。在浏览器中打开`/fetch`路由，并确认输出显示在浏览器控制台和你的开发服务器上。完成这些操作后，通过导出`ssr`页面选项来禁用页面上的 SSR：

src/routes/fetch/+page.js

```js
const key = 'DEMO_KEY'; // your API key here
export const ssr = false;
export function load() {
  const pic = fetch(`https://api.nasa.gov/planetary/apod?api_key=${key}`)
    .then(response => {
      console.log('got response');
      return response.json();
    });
  return {pic};
}
```

这个例子与我们之前看到的例子相同，只是在第 3 行，我们添加了 `export const ssr = false;`。这个页面选项有效地禁用了页面的 SSR（服务器端渲染），意味着 `load()` 只会在客户端运行。你会注意到 `console.log()` 调用不再输出到开发服务器，但会在浏览器控制台中显示。

从现在开始，我们将区分 `load()` 函数，即 `load()` 函数是在服务器上的 `+page.server.js` 中运行的，这意味着通用 `load()` 函数是从 `+page.js` 中运行的。从高层次来看，它们在功能上是相同的。但还有一些特殊之处需要提及：

+   通用和服务器 `load()` 函数都可以访问与调用它的请求相关的数据。

+   服务器 `load()` 函数将能够访问更多请求数据，例如 cookie 和客户端 IP 地址。

+   通用 `load()` 函数始终返回一个对象。该对象的值可以是几乎任何东西。

+   服务器 `load()` 函数*必须*返回可以被 `devalue` 包序列化的数据（基本上，任何可以转换为 JSON 的东西）。更多关于 `devalue` 的信息请访问 [`github.com/rich-harris/devalue`](https://github.com/rich-harris/devalue)。

通用负载时间

应该提到的是，在第一次渲染时，`load()` 将在服务器和客户端执行。然后，每个后续请求都将在客户端执行。为了演示这种行为，请将你的浏览器导航到一个从 `+page.js` 文件中运行 `load()` 的路由，例如我们来自 *第三章* 的 `/fetch` 示例。观察在浏览器中首次打开 `/fetch` 页面时服务器和客户端的输出控制台。导航到另一个路由然后再返回将只显示客户端的输出。

关于调用 `load()` 的最后一点说明；除非你指定页面应该通过页面选项预先渲染，否则它将在运行时始终被调用。如果你决定预先渲染页面，那么 `load()` 将在构建时被调用。记住，页面应该只在静态 HTML 对于每个访问页面的用户都相同的情况下才进行预先渲染。

我们刚刚介绍了如何强制 `load()` 只在客户端运行以及它的一些工作细节。结合所有这些新信息以及前几章的信息，你应该对 `load()` 的基础知识感到相对舒适。让我们进一步探讨，看看它如何在布局模板中使用。

# 加载布局

到目前为止，我们只看了 `load()` 在 `+page.js` 或 `+page.server.js` 文件中的使用，但它也可以在 `+layout.js` 或 `+layout.server.js` 文件中使用。虽然布局不能导出操作，但它们在其他方面与页面文件功能相同。这意味着之前提到的页面选项（如 `ssr`）和 `load()` 函数将适用于布局内部嵌套的任何组件。关于 `load()` 函数的另一个重要特性是，由于它们在 SvelteKit 中并发运行，单个页面将在所有请求完成之前不会渲染。在页面上以及布局上都有 `load()` 函数将防止渲染，直到两者都完成。但由于它们将同时运行，任何延迟都应该微不足道。

在布局中加载数据的最明显优势是能够访问在兄弟和子页面中的数据。这意味着布局加载的任何数据都可以在导出 `data` 变量的继承的 `+page.svelte` 文件中访问。SvelteKit 还会跟踪应用程序中加载的数据，并且只有在它认为绝对必要时才会触发 `load()`。在我们想要强制重新加载数据的情况下，我们可以导入 `$app/navigation` 提供的 `invalidate` 或 `invalidateAll` 模块。

为了演示这些概念，让我们创建一个与导航并列的组件，以便向用户提醒未读通知。该组件将持久存在于应用头部，以便易于访问。这为展示从布局中加载数据的理想场景提供了可能。我们还将创建另一个页面，显示通知的完整列表，以演示从布局加载数据如何在子组件中使用。

让我们从 `+layout.js` 中的 `load()` 函数开始。为了简单起见，我们将在函数调用中直接返回数据，而不是调用一个假想的数据库或 API：

src/routes/+layout.js

```js
export function load() {
  console.log('notifications loaded');
  return {
    notifications: {
      count: 3,
      items: [
        {
          type: `comment`,
          content: `Hi! I'm Dylan!`
        },
        {
          type: `comment`,
          content: `Hi Dylan. Nice to meet you!`
        },
        {
          type: `comment`,
          content: `Welcome to the chapter about load()!`
        }
      ]
    },
  }
}
```

此文件仅包含导出的 `load()` 函数，它返回一个包含另一个 `notifications` 对象的对象。请记住，通用的 `load()` 函数可以导出任何东西，只要它位于对象内部。`notifications` 对象相当简单，它由两个属性组成；一个值为 `3` 的 `count` 属性，另一个标记为 `items` 的属性，它只是一个包含三个其他对象的数组。为了显示数据不是每次导航到新页面时都会加载，我们包含了一个 `console.log()` 调用，输出文本 `notifications loaded`。

接下来，我们将对我们的根布局模板进行一些更改，以便它实际上可以使用我们刚刚加载的数据。大部分将保持不变，但我们需要添加一些可以显示数据的标记以及一些基本的样式来传达**通知徽章**的概念：

src/routes/+layout.svelte

```js
<script>
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
  <div class='content'>
    <slot></slot>
  </div>
  <div class='footer'>
    This is my footer
  </div>
</div>
<style>
  .wrapper {
    min-height: 100vh;
    display: grid;
    grid-template-rows: auto 1fr auto;
  }
  .footer {
    text-align: center;
    margin: 20px 0;
  }
  .nav {
    text-align: center;
  }
  .menu {
    display: inline-block;
  }
  .notifications {
    float: right;
  }
</style>
```

在这个版本的 `+layout.svelte` 中有一些重要的更改需要注意：

+   导入了新的`Notify`组件（如下所示）。

+   我们导出`data`变量以使用从`src/routes/+layout.js`返回的数据。

+   将`notifications`的`count`属性发送到`Notify`组件。

+   将`.menu`和`.notifications`元素的标记添加到`.nav`div 元素中。这允许我们在页面的右上角显示`Notify`组件。

+   为具有`.nav`、`.menu`和`.notifications`类的元素添加了新的样式，以装饰我们的新标记。

接下来，让我们看看我们刚刚导入的`Notify`组件。此组件将包含显示我们的通知计数和链接到`/notification`路由的标记：

src/lib/Notify.svelte

```js
<script>
  export let count = 0;
</script>
<a href='/notifications'>
  {count}
</a>
<style>
  a {
    padding: 15px;
    color: white;
    text-decoration: none;
    background-color: #ea6262;
  }
</style>
```

此组件相对简单。首先，它导出`count`变量并为其设置默认值`0`。这是必要的，因为虽然这个组件在布局中使用，但它并不位于我们之前创建的`+layout.js`文件之下或旁边，因此它无法访问布局`load()`函数提供的信息。接下来，这个组件创建一个链接标签来包含`count`变量。最后，它包含了一些基本的样式来装饰我们的通知徽章。

最后，让我们看看通知页面。因为这个文件位于`+layout.js`的层次结构之下，我们可以像从与其并存的`+page.js`文件加载的数据一样访问`data`：

src/routes/notifications/+page.svelte

```js
<script>
  export let data;
</script>
{#if data.notifications.count > 0}
  <ul>
  {#each data.notifications.items as item}
    <li>{item.content}</li>
  {/each}
  </ul>
{/if}
```

此页面使用了 Svelte 指令：`{#if}`和`{#each}`。由于我们在组件顶部导出了`data`变量，因此我们可以在此组件中使用从`src/routes/+layout.js`加载的数据。如果`notifications`对象的`count`属性大于零，它将创建必要的标记来生成一个无序列表。然后，它在一个列表项中输出每个评论项的`content`属性。

现在，当你用浏览器打开你的项目时，你应该在应用的右上角看到一个新通知徽章，显示`notification`对象中`count`属性的值。尝试选择导航菜单中的某些项，看看每次点击链接时文本**notifications loaded**是否都会输出。它在开发服务器和浏览器控制台中首次加载时显示，但不会再次运行。这是因为正在加载的数据尚未更改，SvelteKit 能够识别这一点。

让我们看看如何在点击通知徽章时强制重新加载数据。我们可以通过使用从`$app/navigation`导入的`invalidateAll`来实现这一点。如果`load()`函数使用了`fetch()`，那么使用`invalidate`模块是有意义的。在那个例子中，我们会通过将`fetch()`调用中指定的 URL 传递给`invalidate()`来强制重新加载。由于我们只是返回一个对象，我们需要使用`invalidateAll()`来触发重新加载：

src/lib/Notify.svelte

```js
<script>
  import { invalidateAll } from '$app/navigation';
  export let count = 0;
</script>
<a href='/notifications' on:click={() => invalidateAll()}>
  {count}
</a>
```

在`Notify.svelte`组件中，我们添加了对`invalidateAll`的导入。当点击通知链接徽章时，它会调用`invalidateAll()`，通知 SvelteKit 重新运行上下文中的所有`load()`函数。现在，当你点击页面顶部的通知链接时，你应该会在浏览器控制台看到输出**notifications loaded**。导航到其他页面，如**关于**、**新闻**或**主页**，则不会产生输出。

在未来，如果你发现自己正在构建将在应用程序界面中显示动态数据的组件，请考虑我们刚刚讨论的概念。通过在布局文件中加载数据，你可以减少 HTTP 请求或数据库查询的数量，这可以显著提高用户的应用程序体验。如果你需要强制重新加载数据，你将知道如何使数据无效，以便 SvelteKit 重新运行适当的`load()`函数。接下来，让我们看看如何进一步利用`load()`来构建更高级的功能。

# 解构`RequestEvent`

当涉及到`load()`函数时，服务器似乎比客户端拥有更多信息。在如此多的数据中，很难确切知道所有可用的信息。简而言之，服务器的`load()`函数使用 SvelteKit 特定的`RequestEvent`进行调用。以下是该对象中可用的属性（`prop`）和函数（`fn`）的快速概述：

+   `cookies` (`prop`) – 请求期间发送的 cookie。

+   `fetch` (`fn`) – 在*第三章*中讨论的 Web API `fetch()`函数的兼容变体。它带来了额外的优势，允许基于相对路由进行请求，并在同一服务器上传递 cookie 和头部信息。

+   `getClientAddress` (`fn`) – 返回客户端的 IP 地址。

+   `locals` (`prop`) – 通过 SvelteKit 的`handle()`钩子插入到请求中的任何自定义数据。我们将在后面的章节中介绍。

+   `params` (`prop`) – 当前路由特有的参数，例如上一章新闻示例中传递给文章的 slug。

+   `platform` (`prop`) – 环境适配器添加的数据。

+   `request` (`prop`) – 实际请求数据，表示为一个对象。

+   `route` (`prop`) – 请求路由的标识符。

+   `setHeaders` (`fn`) – 允许在返回的`Response`对象中操作头部。

+   `url` (`prop`) – 关于请求 URL 的数据，我们在*第三章*中讨论过。

请求事件演示

要亲自查看这些信息，请创建一个包含`console.log()`函数的`src/routes/+layout.server.js`文件，该函数输出传递给`load()`的单个参数。通过在根布局中创建它，你将能够看到基于从浏览器访问的不同路由，属性是如何变化的。然后，数据将在你的开发控制台中显示。

一个你可能需要利用这些数据的实际例子是在用户认证的情况下。通常，用户认证后，他们会被分配一个 cookie（*因为他们输入密码做得很好 – 这是个双关语*）存储在他们的设备上，这确保了他们的认证将在访问期间持续有效。如果他们离开应用程序，它可以在以后用来确认他们的身份，这样他们就不需要再次进行认证。让我们观察一下 SvelteKit 是如何实现这一点的。由于本章是关于`load()`，我们将构建实际的表单并在下一章讨论如何设置 cookie。现在，我们只需检查用户是否设置了 cookie，并在浏览器中手动设置一个。

首先，让我们将`src/routes/+layout.js`重命名为`src/routes/+layout.server.js`。如果我们打算访问 cookie 数据，我们需要访问`RequestEvent`提供的数据。通过将逻辑添加到我们的根服务器布局中，我们有一个额外的优势，即在整个应用程序中保持认证检查：

`src/routes/+layout.server.js`

```js
export function load({ cookies }) {
  const data = {
    notifications: {
      count: 3,
      items: [
        {
          type: `comment`,
          content: `Hi! I'm Dylan!`
        },
         …
      ]
    }
  };
  if(cookies.get('identity') === '1') {
    // lookup user ID in database
    data.user = {
      id: 1,
      name: 'Dylan'
    }
  }
  return data;
}
```

在这个新的根布局逻辑版本中，我们解构了传递给`load()`的参数，因为我们目前只需要访问`cookies`属性。我们保留了之前创建的`notifications`对象，但将其放入一个名为`data`的新变量中。为了简洁起见，这段文本省略了一些条目。从那里，我们检查发送到我们应用程序的请求是否包含一个名为`user`且值为`1`的 cookie。如果是，我们将一些假用户信息插入到`data`对象的`user`属性中。通常在这种情况下，我们会将 cookie 值与数据库中的有效会话进行比对，如果找到，则检索相应的用户数据并将其发送回客户端，但我们正在尝试保持简单。完成所有这些后，`data`对象从`load()`返回。

接下来，我们需要实际展示用户已经成功认证。为此，我们将创建一个新的路由，让我们的用户可以登录：

`src/routes/login/+page.svelte`

```js
<script>
  export let data;
</script>
{#if data.user}
  <p>
    Welcome, {data.user.name}!
  </p>
{/if}
```

如我们在上一节中所述，从`+layout.js`或`+layout.server.js`返回的数据通过导出`data`变量在子组件中可用。一旦完成，我们使用 Svelte 的`{#if}`指令检查是否设置了`user`属性。如果找到，我们则显示`data.user`的`name`属性。

当然，在这个例子中，我们从未设置任何 cookie。我们将在下一章中介绍这一点，所以现在，让我们手动在我们的浏览器中创建 cookie。在这样做之前，导航到`/login`路由并验证页面上没有显示任何内容。一旦确认它是一个空白页面，请按照以下步骤为您的浏览器创建 cookie：

+   `identity`。

+   双击`1`。

+   使用相同的步骤确保`/`。

+   `identity`。

+   选择`1`。

+   使用相同的步骤确保`/`。

按照这些步骤操作后，你现在应该已经在浏览器中拥有了正确的 cookie。完成此操作后，在浏览器中刷新`/login`页面，你会看到一个欢迎用户的消息，其中包含由`name`属性指定的值。这个例子相当简单，而实际的基于 cookie 的登录系统在功能上稍微复杂一些；然而，概念是相同的。

尽管我们覆盖的例子只使用了`RequestEvent`中的`cookies`属性，但我们看到了如何轻松地访问其他任何属性，例如`url`和`params`，或者甚至使用`setHeaders`函数设置我们自己的头部信息。有了所有这些数据可供我们使用，我们可以在应用程序中构建的内容的可能性几乎是无限的。

# 摘要

在本章中，我们讨论了关于`load()`的大量信息。我们首先讨论了它只能在客户端完成，然后转向一些关于其工作原理的更详细的内容。之后，我们探讨了在布局中使用`load()`以最小化每个页面加载时发出的请求数量，并最大化方便地访问可能需要应用范围内的数据。我们还探讨了在需要重新加载数据的情况下如何使数据失效。最后，我们介绍了服务器`load()`函数是如何通过`RequestEvent`调用的，这为我们提供了访问大量有价值信息的机会。这些信息可以让我们为我们的应用程序构建基于 cookie 的登录功能。

在本章中，我们学习了关于`load()`背后的一些更详细的细节，你应该感到轻松，可以休息一下。如果你手头有烤制的饼干，我建议你从书中休息一下，给自己一些奖励。你已经应得它了。

但请务必回来，因为在下一章中，我们将探讨更多关于通过使用表单从用户那里接收数据背后的细节，使表单变得有趣，并通过利用快照来减少数据输入的摩擦。

# 资源

devalue: [`github.com/rich-harris/devalue`](https://github.com/rich-harris/devalue)
