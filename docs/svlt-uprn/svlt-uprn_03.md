

# 第三章：兼容现有标准

一些框架试图通过为你提供工具和功能来简化你的开发工作，例如处理网络请求或管理表单提交的数据。虽然这种策略的意图是高尚的，但它可能产生意想不到的后果。例如，当学习一个新的框架时，开发者必须掌握其所有复杂性才能有效。阅读关于*另一种制作网络请求的方式*可能会减慢开发者的速度，因为阅读文档的时间就是没有在构建的时间。它也可能防止代码的可移植性。当为应用*A*编写的代码特定于框架*X*时，那么在将其用于使用框架*Y*构建的应用*B*之前，代码需要修改。

SvelteKit 针对这个问题有一个解决方案，那就是什么都不做。嗯，不是真的什么都不做，而是不提供需要你每次使用时查阅文档的包装器和函数，SvelteKit 鼓励使用现有的 Web **应用程序编程接口**（**APIs**）和标准。通过不重新发明轮子，更多的开发者可以快速开始使用 SvelteKit，因为他们不需要学习他们已经熟悉的标准的抽象。这不仅将阅读文档的时间降到最低，而且还需要更少的代码来驱动框架。

本章将介绍一些基本的**Web APIs**用法以及它们如何与 SvelteKit 交互。具体来说，我们将查看以下当前网络标准的示例：

+   获取

+   FormData

+   URL

对这些标准的深入探讨超出了本章的范围，但如果你想了解更多信息，相关信息将在本章末尾提供。在这些示例之后，你应该能够舒适地使用你对各种网络标准的现有知识来使用 SvelteKit。

# 技术要求

本章的完整代码可在 GitHub 上找到：[`github.com/PacktPublishing/SvelteKit-Up-and-Running/tree/main/chapters/chapter03`](https://github.com/PacktPublishing/SvelteKit-Up-and-Running/tree/main/chapters/chapter03)。

# Fetch

首先，让我们看看最常用的 Web APIs 之一，`fetch`。假设你的开发环境运行在 Node.js（v18+）的最新 LTS 版本上，你将无需安装额外的包如`node-fetch`就能在服务器上使用`fetch`。那个包是最广泛使用的包，它提供了功能，允许开发者在`fetch`被纳入 Node.js 的核心功能之前进行网络请求。由于`fetch`现在也支持每个主要浏览器，我们可以在客户端和服务器端环境中安全地使用它来发起外部和内部请求。因为 SvelteKit 可以在浏览器和服务器环境中运行，所以我们可以说，我们实际上“让 fetch 发生了。”

为了说明`fetch`在浏览器和服务器上如何工作，让我们看看一个例子。我们将通过添加`src/routes/fetch/+page.svelte`文件来创建一个新的路由，作为我们的演示页面。我们还将创建`src/routes/fetch/+page.js`来从实际的`fetch`请求中获取数据。在这个例子中，我们将利用位于[`api.nasa.gov/`](https://api.nasa.gov/)的 NASA 免费“每日天文图片”API。建议获取一个 API 密钥以供常规使用；然而，以下示例中提供的演示 API 密钥足以满足我们的需求。这个例子向 NASA API 发出网络请求，并在我们的页面上显示接收到的数据。

请注意，样式绝对不是必需的，只是提供以使我们的示例在浏览器中查看时更容易理解。我们可以添加很多样式，使这个例子看起来很棒，但这会占用我们在这页面上有限的更多空间。此外，你在这里是为了学习 SvelteKit，而不是 CSS：

`src/routes/fetch/+page.svelte`

```js
<script>
  export let data;
</script>
<h1>Astronomy Picture of the Day</h1>
<h2>{data.pic.title}</h2>
<div class='wrap'>
  <a href={data.pic.hdurl}><img src={data.pic.url}></a>
</div>
<p>{data.pic.explanation}</p>
<style>
  h1, h2 {
    text-align: center;
  }
  .wrap {
    max-width: 75%;
    margin: 0 auto;
    text-align: center;
  }
  img {
    max-width: 100%;
  }
</style>
```

在这个文件中，我们只是设置了数据的标记，以便填充。要访问由兄弟文件`+page.js`提供的数据，我们必须包含`export let data;`。如果你熟悉 React 或 Vue，可以将这视为 SvelteKit 在组件之间传递属性的方法。如果你熟悉 React 或 Vue，可以将这视为 SvelteKit 在组件之间传递属性的方法。完成这些后，我们可以使用对象中提供的数据来填充图像标题、链接到高分辨率文件、链接到外部托管图像，并显示图像描述。当然，我们还添加了一些最小化样式。真正的魔法，以及我们的`fetch`请求，发生在下一个`+page.js`片段中，我们在那里调用外部 API：

`src/routes/fetch/+page.js`

```js
const key = 'DEMO_KEY'; // your API key here
export function load() {
  const pic = fetch(`https://api.nasa.gov/planetary/apod?api_key=${key}`)
  .then(response => {
    console.log('got response');
    return response.json();
  });
  return {pic};
}
```

在`+page.js`中，我们创建了一个常量，你可以在这里放置你自己的 API 密钥。然后，我们导出了`load`函数，这是告诉 SvelteKit 在渲染兄弟文件`+page.svelte`之前必须运行的函数。它可以`async`也可以不是；SvelteKit 并不关心，会相应地处理每种情况。然后我们创建了一个常量`pic`，我们可以将`fetch`调用返回的 promise 分配给它。在`fetch`调用中，我们提供了带有附加 API 密钥的 URL，作为第一个也是唯一的参数。如果你的 API 需要在头部指定选项、设置方法或可能需要一个认证 cookie，你可以通过在第二个参数中提供一个对象来实现。记住，SvelteKit 旨在与现有的网络标准兼容，所以实现将至少包含标准化的功能。有关如何以这种方式利用`fetch`的更多信息，请参阅本章末尾的资源。

继续使用`fetch`调用接收到的承诺，我们运行`console.log()`来演示代码在浏览器和服务器环境中都会运行。如果你的开发环境还没有运行，你可以使用`npm run dev`命令启动它。记住，它将在提供的 URL 中在浏览器中可用。你可以通过检查你的浏览器控制台输出（在开发者工具中）以及你的终端中的 Vite 开发服务器输出来确认这一点。在这两种情况下，你应该看到显示的“*收到响应*”输出。因为我们的请求不需要任何认证，所以在客户端运行是安全的。

最后，我们将转换`load`函数，这些函数在服务器和浏览器上都会运行。对于通用`load`函数，它们**必须**返回一个对象。

加载数据

我们已经确定从`+page.js`运行的代码实际上在浏览器和服务器上都会运行。如果我们想让代码只在客户端运行，我们就可以将其放在`+page.svelte`中。同样，如果我们想让这段代码只在服务器上运行，我们可以将文件名更改为`+page.server.js`。第二种用法更适合进行认证数据库调用或访问环境变量，例如 API 密钥（就像我们在之前的例子中所做的那样）。SvelteKit 将识别这个文件是打算只在服务器环境中运行的，并将采取适当的步骤来分离该逻辑。服务器`load`函数与通用`load`函数的工作方式略有不同，但我们将更深入地探讨这一点，在后面的章节中。

在完成所有这些操作之后，我们现在可以在浏览器中导航到 `/fetch`，查看图片、标题和描述，甚至可以点击图片查看全分辨率文件。调用各种内部或外部 API 非常有帮助，但我们的应用程序可能需要在某个时候从用户那里接收数据。在下一节中，我们将介绍如何访问和操作`Requests`中提供的`FormData`对象。

# FormData

在构建 Web 应用程序时，通常通过使用表单来接受用户数据。可以通过`+page.svelte`和`+page.server.js`访问这些数据，这些都在新的*评论*路由下。和之前一样，`+page.svelte`文件包含用于构建表单的 HTML 和最小样式。为了获取从服务器发送的数据，我们必须在我们的客户端`<script>`代码中包含读取`export let form;`的行。这让我们可以查看从`+page.server.js`返回的对象的状态，并通过 Svelte 提供的模板系统将状态反馈给用户：

src/routes/comment/+page.svelte

```js
<script>
  export let form;
</script>
<div class='wrap'>
  {#if form && form.status === true}
    <p>{form.msg}</p>
  {/if}
  <form method='POST'>
    <label>
      Comment
      <input name="comment" type="text">
    </label>
    <button>Submit</button>
  </form>
</div>
<style>
  .wrap {
    width: 50%;
    margin: 0 auto;
    text-align: center;
  }
  p {
    color: green;
  }
  label {
    display: block;
    margin: 1rem;
  }
</style>
```

src/routes/comment/+page.server.js

```js
export const actions = {
  default: async (event) => {
    const form = await event.request.formData();
    const comment = form.get('comment');
    if(comment) {
      // save comment to database
      return {
        status: true,
        msg: `Your comment has been received!`
      }
    }
  }
}
```

如前所述，`+page.server.js`与`+page.js`文件的工作方式相同，但它在服务器上才会运行。就像我们可以在`fetch`示例中导出`load()`一样，服务器端文件也可以导出`actions`。这些导出的`actions`只能通过`POST`请求访问。我们将在后面的章节中探讨如何进一步利用`actions`，但到目前为止，请注意我们正在创建一个单独的动作，`default`。这意味着当表单提交时，`POST`请求将由`/comment`端点处理。

从`default`动作中，这个例子访问了位于`event`中的`Request`对象，然后访问该`Request`中的`FormData`。一旦`FormData`对象被分配给`form`变量，我们使用`get()`通过名称从表单输入中检索值。通常，我们接下来会进行类似数据库调用的操作，以保存评论和唯一的用户标识符。为了简洁起见，我们只需返回一个包含`status`和`msg`属性的简单对象。

当页面重新加载时，这些属性将通过 Svelte 模板语法进行检查，并向用户显示他们的评论已成功提交的消息。尽管这个例子只使用了`FormData`的`get()`方法，但官方标准中记录的所有方法对我们也都是可用的。如果我们想遍历所有提交的值和键，我们可以使用`for...of`循环，并通过`form.values()`和`form.keys()`访问值。

现在我们知道了如何从提交的表单中获取数据，我们也应该看看另一种可能被忽视的从用户那里接收数据的方式——URL。

# URL

`event`对象，就像`request`对象一样。让我们继续使用之前的评论示例。如果我们想在其他网站的网络中构建一个评论服务，我们可能想向用户报告他们刚刚在哪个网站上发表了评论。让我们扩展之前的例子并做到这一点。我们不需要对`src/routes/comment/+page.svelte`做任何修改。相反，我们将相应地调整我们的服务器动作。

这个例子与上一个例子相同，只是做了一些改动。通过`event.url`访问 URL API，并将其分配给`url`常量。然后使用`console.log(url)`将其输出到服务器，以显示你可以访问的各种只读属性。为了演示，我们通过`url.hostname`获取主机名，并在模板字面量中使用它，这被分配给返回对象的`msg`属性：

src/routes/comment/+page.server.js

```js
export const actions = {
  default: async (event) => {
    const form = await event.request.formData();
    const url = event.url;
    console.log(url);
    const comment = form.get('comment');
    if(comment) {
      // save comment to DB here
      return {
        status: true,
        msg: `Your comment at ${url.hostname} has been 
          received!`
      }
    }
  }
}
```

如果你切换到浏览器窗口并发表评论，你现在会看到它报告了`hostname`属性。在生产环境中，这应该是你的域名，但在我们的开发环境中，你会看到消息`http://localhost`，你将看到`URL`对象：

示例事件.url 对象

```js
URL {
  href: 'http://127.0.0.1:5173/comment',
  origin: 'http://127.0.0.1:5173',
  protocol: 'http:',
  username: '',
  password: '',
  host: '127.0.0.1:5173',
  hostname: '127.0.0.1',
  port: '5173',
  pathname: '/comment',
  search: '',
  searchParams: URLSearchParams {},\
  hash: ''
}
```

这个`URL`对象展示了获取其信息是多么简单。所有显示的只读属性都可以通过点符号轻松访问。不再需要使用正则表达式解析整个`URL`以提取所需的部分。现在，如果您想获取查询字符串中设置的值，可以轻松地使用`for...of`循环遍历`event.url.searchParams`。在您的浏览器中添加一些 URL 选项。一个例子可能看起来像`http://127.0.0.1:5173/comment?id=5&name=YOUR_NAME_HERE`。现在，`console.log(url)`函数调用将输出第一个`?`之后和每个随后的`&`设置的名字和值。

# 摘要

在本章中，我们介绍了一个`fetch`的示例用例，以及一个两部分的演示，展示了如何使用`URL`和`FormData`。虽然这里提供的示例并不代表您将能够访问的所有各种 Web API 的全貌，但它们应该说明了使用 SvelteKit 使用它们是多么简单。如果您希望使用 SvelteKit 构建应用程序，那么熟悉这些现代 Web API 非常重要，因为它们在 SvelteKit 的开发过程中被广泛使用。SvelteKit 鼓励您依赖现有的知识。通过这样做，SvelteKit 可以向您提供更少的代码，从而使您可以向用户提供更少的代码。

在下一章中，我们将从使用 SvelteKit 所必需的背景信息中脱离出来，开始构建一个类似应用程序的东西。我们将介绍各种路由技术以及如何在应用程序中构建一致的用户界面。

# 资源

+   MDN Web 文档 – 要了解`fetch()`、`formData()`、`URL`等更多信息，请访问[`developer.mozilla.org`](https://developer.mozilla.org)。

# 第二部分 – 核心概念

网络导航是一种非常典型的体验，SvelteKit 的开发者已经将路由作为框架的核心。在本部分中，我们将更详细地检查之前介绍的路线技术。之后，我们将看到 SvelteKit 如何将数据移动到组件中，并通过 HTML 表单元素从它们那里接受数据。最后，我们将看到一些更高级的路由技术，这些技术承诺可以覆盖在路由中遇到的几乎所有边缘情况。

本部分包含以下章节：

+   *第四章*, *有效路由技术*

+   *第五章*, *深入数据加载*

+   *第六章*, *表单和数据提交*

+   *第七章*, *高级路由技术*
