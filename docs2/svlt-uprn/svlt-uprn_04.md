# 4

# 有效的路由技术

我们已经花费了很多时间来介绍背景信息。我知道这些主题并不总是最令人兴奋的，但现在我们已经介绍了它们，我们可以进入真正的乐趣。到目前为止，我们只是简要地提到了通过在`src/routes/`内部创建一个名为所需路由名的目录来添加新路由，并在其中添加一个`+page.svelte`文件。我们还简要地探讨了创建服务器页面。但是，当然，路由并不总是这么简单。我们如何构建**应用程序编程接口**（**API**）？我们如何在不重复每个页面的样式的情况下，在整个应用程序中创建一个一致的**用户界面**（**UI**）？当我们的应用程序抛出错误时会发生什么？

在本章中，我们将通过讨论 SvelteKit 路由的某些核心点来回答这些问题。首先，我们将看看我们如何创建具有动态内容的新页面。然后，我们将更详细地研究`+page.server.js`文件的工作方式。然后，我们将展示如何创建可以接受各种类型 HTTP 请求的 API 端点。最后，我们将介绍如何使用布局在整个应用程序中构建一个一致的 UI。

在本章中，我们将涵盖以下主题：

+   创建动态页面

+   创建服务器页面

+   创建 API 端点

+   创建布局

到本章结束时，你应该对 SvelteKit 的基于文件的路由机制的路由概念有一个舒适的理解。

# 技术要求

本章的完整代码可在 GitHub 上找到：[`github.com/PacktPublishing/SvelteKit-Up-and-Running/tree/main/chapters/chapter04`](https://github.com/PacktPublishing/SvelteKit-Up-and-Running/tree/main/chapters/chapter04)。

# 创建动态页面

在前面的章节中，我们已经介绍了创建新页面的过程。为了刷新你的记忆，它就像在`src/routes/`内部创建一个名为所需路由名的新目录一样简单。在那个目录内部，我们创建`+page.svelte`，它只是一个 Svelte 组件，然后作为在浏览器中显示的页面进行渲染。一个令人尴尬的简单的**关于**页面可能看起来像这样：

src/routes/about/+page.svelte

```js
<div class='wrapper'>
  <h1>About</h1>
  <p>
    Lorem ipsum dolor sit amet...
  </p>
</div>
```

这个例子仅仅说明了添加新页面是多么简单。在其中，我们看到一个`div`，一个`h1`标题标签，和一个带有`lorem ipsum`样本文本的段落`p`标签。当然，在现实世界的场景中，它会有更多内容以及一些样式。这个例子只存在是为了展示添加新静态页面是多么简单。

但如果你需要创建一个你不知道内容是什么的页面呢？如果我们想创建一个具有动态 URL 的页面呢？例如，当在线查看新闻文章时，用户通常可以直接分享到文章的链接。这意味着每篇文章都有自己的独特 URL。如果我们想创建一个显示从数据库中提取的文章的单个模板，我们就需要找到一种方法来管理每个 URL。 

在这些情况下，SvelteKit 的基于文件的路由机制有一个特殊的语法可以使用。当创建显示内容的模板时，我们在一个目录中创建它，其中路由的动态部分被方括号`[ ]`包围。方括号内的名称将成为一个参数，允许我们动态加载数据。为了使此参数可选，请使用双方括号`[[ ]]`。

这可能一开始听起来有些令人困惑，所以让我们看看一个示例，展示您如何管理新闻文章或博客文章。在这个示例中，我们需要创建一些文件。我们不会连接到实际的数据库，而是使用 JSON 文件来存储一些示例数据并直接从该文件中提取：

src/lib/articles.json

```js
{
  "0": {
    "title": "First Post",
    "slug": "first-post",
    "content": "Lorem ipsum dolor…"
  },
  "1": {
    "title": "Effective Routing Techniques",
    "slug": "effective-routing-techniques",
    "content": "Lorem ipsum dolor…"
  }
}
```

此文件基本上是一个包含两个对象的单一对象，每个对象都具有`title`、`slug`和`content`属性。您可以随意调整此文件或添加您想看到的条目。其目的是作为一个占位符数据库来展示以下示例。

接下来，新闻页面通常有一个着陆页，用户可以滚动查看最新的文章。根据既定惯例，我们将创建一个`+page.server.js`文件来加载数据并将其提供给作为页面渲染数据的`+page.svelte`模板：

src/routes/news/+page.server.js

```js
import json from '$lib/articles.json';
export function load() {
  return { json };
}
```

由于它不包含任何环境变量或秘密，也不调用真实的数据库，这个文件也可以是一个`+page.js`文件。它仅用于从 JSON 示例文件加载数据。本质上，它导入该文件，然后在导出的`load()`函数中将其作为对象返回，使其对下一个显示的 Svelte 模板可用。实际上，这样的事情也可以在 Svelte 模板的`<script>`标签中完成，但请记住，这个示例旨在作为一个真实数据库的替代品。

现在我们已经加载了我们的文章，我们需要一个地方来展示它们，所以现在让我们创建一个用于新闻的着陆页。为了简洁起见，省略了样式：

src/routes/news/+page.svelte

```js
<script>
  export let data;
</script>
<h1>
  News
</h1>
<ul>
  {#each Object.entries(data.json) as [key, value]}
    <li>
      <a href='/news/{value.slug}'>{value.title}</a>
    </li>
  {/each}
</ul>
```

这个 Svelte 组件导出`data`变量，使我们能够访问从我们的模拟数据库在`load()`中返回的数据。然后它添加一个`h1`标题标签，接着是一个无序列表，列出我们模拟数据中的每个条目。之后，它使用`{#each}` Svelte 模板语法遍历从`Object.entries(data.json)`返回的数组中的每个条目。对于每个条目（我们的两个示例对象），我们用列表项标签将其包围，并在`<a>`标签中显示`title`属性，通过`slug`属性链接到文章。

接下来，我们需要创建一个页面来显示文章内容，但请注意，我们希望路由中包含一个参数，所以我们将使用方括号`[ ]`来包围文章的 slug：

src/routes/news/[slug]/+page.svelte

```js
<script>
  export let data;
</script>
<h1>News</h1>
<h2>{data.title}</h2>
<div class='content'>
  {data.content}
</div>
<a href='/news'>Back to news</a>
```

此文件导出`data`变量，以便我们可以访问关于我们的文章的信息。然后，它在一个`<h1>`标题标签内显示`News`，接着是一个带有文章标题的`<h2>`标题标签，一个包含文章内容的`div`，以及一个返回新闻页面的链接。所有文章信息都是通过在下一个文件中对我们的*数据库*进行另一次调用而加载的：

src/routes/news/[slug]/+page.server.js

```js
import json from '$lib/articles.json';
import { error } from '@sveltejs/kit';
export function load({ params }) {
  let found = {};
  Object.keys(json).forEach((id) => {
    if(json[id].slug === params.slug) {
      found = json[id];
    }
  });
  if(Object.keys(found).length !== 0) {
    return found;
  }
  throw error(404, 'Whoops! That article wasn\'t found!');
}
```

就像之前一样，我们已经导入我们的模拟数据库来访问完整的文章内容。此代码还导入了 SvelteKit 的`error`模块，这将在稍后很有用。然后我们导出了`load()`函数，以便我们可以将加载的数据返回到渲染的页面。在`load()`函数内部，代码初始化一个名为`found`的空变量，然后开始遍历 JSON 数据中的每个对象。在循环中，它检查我们数据中的任何 slugs 是否与 URL 中给出的 slug 匹配。如果找到匹配项，它将被分配给`found`变量。循环结束后，我们检查`found`不是一个空对象。如果不是空的，我们返回一个包含`found`变量的对象。如果是空的，我们抛出一个**404 Not Found**错误。

当你在浏览器中打开你的开发站点并导航到`/news`时，你应该看到列出两个文章标题。点击它们将使用户重定向到相应的文章。这个例子以简单的方式说明了参数化路由，这在大多数情况下是有效的。但是，当它不起作用时我们该怎么办？你尝试导航到一个不存在的文章了吗？现在就试试看；我会在这里等着：

![](img/B19024_04_01.jpg)

图 4.1 – 在 SvelteKit 中抛出错误时显示一个通用错误页面

文章不存在，用户会看到一个通用的错误页面。如果我们完全不抛出错误，页面将渲染显示`undefined`值。相反，我们应该向用户展示一个合适的错误页面。正如 SvelteKit 为我们提供了`+page.svelte`、`+page.js`和`+page.server.js`文件，我们也可以创建一个`+error.svelte`模板，当应用程序抛出错误时可以使用。通过在`src/routes/news/[slug]/`目录中指定它，错误页面模板将被本地化到该特定路由。如果我们想构建一个在整个应用程序中通用的错误页面，我们可以在应用程序的根路由（`src/routes/+error.svelte`）放置一个`+error.svelte`模板。让我们在`src/routes/news/[slug]/`中创建一个模板，这样用户就不会因为我们的沟通不清而感到困惑：

src/routes/news/[slug]/+error.svelte

```js
<script>
  import { page } from '$app/stores';
</script>
<h1>News</h1>
<h2>{$page.status}</h2>
<div class='content'>
  {$page.error.message}
</div>
<style>
  * {
    font-family: sans-serif;
  }
  h1, h2, .content {
    text-align: center;
    color: #358eaa;
  }
</style>
```

对于这个错误模板，我们导入`page`存储模块来访问有关此特定请求的信息。该模块以及许多其他模块在整个应用程序中都可以使用。由于这个模块使用了 Svelte 的存储，我们可以通过在前面加上美元符号（`$`）来访问它包含的关于页面的值。这个模板的其余部分相当直接。它包括标记为`News`的`<h1>`标题标签，后面跟着我们在`+page.server.js`中抛出错误时传递的状态码和错误消息。一些样式被包括进来，以显示这个模板与 SvelteKit 中显示的默认模板的不同。将通用 SvelteKit 错误模板的*图 4.1*与我们的自定义版本*图 4.2*进行比较：

![](img/B19024_04_02.jpg)

图 4.2 – 定制的错误页面模板

到目前为止，你应该能够舒适地创建应用程序的基本路由，无论是静态的还是动态的。你也应该能够根据路由显示错误页面。关于高级路由的内容将在稍后介绍，但现在，让我们更仔细地看看`+page.server.js`文件是如何工作的。

# 创建服务器页面

在之前的示例中，我们使用了`+page.js`和`+page.server.js`文件来加载数据。通常，它们可以互换使用，但何时使用哪一个才是最佳时机？在本节中，我们将分析这两个文件之间的部分差异，并讨论`+page.server.js`文件中可用的各种功能。我们将将其分解为以下主题：

+   `load()`

+   页面选项

+   操作

## load()

正如我们在之前的示例中所见，数据可以通过在页面中导出`data`属性来加载到`+page.svelte`组件中。`+page.js`和`+page.server.js`都可以用来将数据加载到该页面模板中，因为它们都可以导出`load()`函数。何时使用哪个文件取决于你计划如何加载数据。当在`+page.js`文件中运行时，`load()`将在**客户端和服务器**上同时运行。如果你能够在这里加载数据，这是推荐的，因为 SvelteKit 可以管理通过`fetch()`调用获取数据。当预加载数据（预测用户可能的行为并在他们实际操作前开始处理）时，这尤其有用。

然而，有时这是不可能的。例如，如果你需要调用需要身份验证或数据库的 API，你很可能不希望你的连接秘密暴露给客户端。这样，任何有权访问你的应用程序的人都可以下载你的秘密并代表你向该 API 或数据库发出请求。在这些情况下，你需要在服务器上存储这些秘密。由于这些秘密在服务器上，你需要访问服务器的文件系统。服务器需要调用以获取适当的数据，并将这些数据传递给客户端。在这些情况下，最好使用`+page.server.js`文件来满足数据加载需求。

## 页面选项

`+page.js`和`+page.server.js`文件不仅用于加载数据。它们还可以导出针对其同级页面的特定选项。某些选项允许您配置与页面渲染相关的功能。这些特定的选项是布尔值，这意味着可以通过将它们设置为`true`来启用，或通过将它们设置为`false`来禁用。它们如下所示：

+   预渲染

+   ssr

+   csr

### 预渲染

虽然可以在`svelte.config.js`项目中自定义预渲染，但您可能会发现自己需要根据页面逐个显式启用或禁用它。要在页面上启用预渲染，您可以设置`export const prerender = true;`。相反，将其值设置为`false`将禁用该页面的预渲染。何时预渲染页面应根据页面的 HTML 内容是否静态来决定。如果页面上显示的 HTML 应该对任何查看者都相同，则该页面被认为是安全的预渲染。通过预渲染页面，HTML 将在构建时生成，并且对于每个特定路由的请求，将静态 HTML 发送到客户端。这导致最终用户的加载时间更快，从而提供更好的体验。

### 服务器端渲染

在`+page.js`文件或`+page.server.js`文件中，`export const ssr = true;`。SvelteKit 默认启用此选项，因此如果您需要更改它，您可能会发现自己通过将其值设置为`false`来禁用它。

### 客户端渲染

而不是在服务器上渲染页面并将其发送到客户端，通过启用`export const csr = false;`，就像您对其他可用的页面选项所做的那样。

如果您发现自己正在构建一个 SPA、一个静态 HTML 网站，或者尝试在客户端渲染静态内容，您可能会发现这些渲染相关的选项很有用。

## 行为

因为`+page.js`文件也在客户端运行，所以它们不能导出行为。行为允许您接收通过 POST HTTP 方法发送的`form`元素提交的数据。我们在*第三章*中看到了一个例子，我们讨论了 SvelteKit 与`FormData` API 的兼容性。在那个例子中，我们导出了一个默认行为。默认行为将由未指定`action`属性的表单元素提交时触发。然而，我们不仅限于只有默认行为，因为 SvelteKit 还允许我们创建命名行为。命名行为将在相同的路由上工作，但通过在查询字符串中提供路由后跟行为名称来区分彼此。默认行为可能不会与命名行为共存，因此您必须删除它或更改其名称，并在使用它的`form`元素上设置`action`属性。

基于我们之前的示例，让我们看看我们如何实现一些与在线评论相关的更多操作。我们可能想要创建的一些附加功能包括允许用户对评论进行`star`或回复。由于我们将添加更多命名操作，我们将`default`操作更改为`create`：

src/routes/comment/+page.server.js

```js
export const actions = {
  create: async (event) => {
    const form = await event.request.formData();
    …
  },
  star: async () => {
    return {
      status: true,
      msg: 'You starred this comment!'
    }
  },
  reply: async () => {
    return {
      status: true,
      msg: 'You replied!'
    }
  }
}
```

注意`default`操作已被更改为`create`。`create`中的代码已被省略，因为自上次示例以来它没有变化。我们还添加了`star`和`reply`操作。目前，它们除了返回一个对象，该对象将输出我们的消息，展示当点击相应的按钮时它们都会被调用。在现实世界的场景中，这些操作可能会调用数据库，增加“star count”，或保存回复评论的内容和被回复评论的唯一标识符。

关于表单本身，我们可以为每个新功能创建单独的表单，并指定`POST`方法和每个新功能的`action`属性。然而，一个更直观的用户体验会将功能整合在一起，并将它们都保留在一个统一的组件中。我们不会创建多个表单，而是为新的功能创建一个按钮，并为每个按钮指定一个`formaction`属性。这样做将保持父表单中指定的 HTTP 方法，但允许根据点击的按钮发送到不同的操作：

src/routes/comment/+page.svelte

```js
<script>
  import { enhance } from '$app/forms';
  export let form;
</script>
<div class='wrap'>
  {#if form && form.status === true}
    <p>{form.msg}</p>
  {/if}
  <form method='POST' action='?/create' use:enhance>
    <label>
      Comment
      <input name="comment" type="text">
    </label>
    <button>Submit</button>
    <button formaction='?/star'>Star</button>
    <button formaction='?/reply'>Reply</button>
  </form>
</div>
<style>
…
</style>
```

从我们上次遇到这个示例时，首先要注意的变化是我们添加了`import { enhance } from '$app/forms';`。这个添加将允许我们使用 JavaScript 逐步增强我们的表单。然后，在每次表单提交后，页面将不需要重新加载。这个模块在下面的`<form>`元素中使用 Svelte 的`use:`指令。尝试运行没有它的示例，并观察现在 URL 将根据点击的哪个按钮包含查询字符串。

说到按钮，我们在本例中添加了两个。每个按钮都设置了`formaction`属性，这允许我们指定从`+page.server.js`中调用哪个命名操作。请注意，我们必须通过指定一个查询参数后跟一个`/`字符来调用这些操作。我们还设置了表单操作为`?/create`。由于我们导出了命名操作，我们不能再有名为`default`的操作，必须在`form`元素上指定要调用的操作。如果我们想调用位于另一个路由的操作，我们可以通过将`formaction`设置为所需的路由名称后跟`?/`，然后是操作名称来轻松实现。

你现在应该对何时使用`+page.server.js`而不是`+page.js`、如何自定义页面渲染以及如何轻松地从`form`元素接受数据有信心。在下一节中，我们将介绍如何创建接受不仅仅是 POST 请求的 API 端点。

# API 端点

我们已经介绍了`+page.svelte`、`+page.js`和`+page.server.js`文件，但我们还没有讨论`+server.js`文件。这些文件使我们能够接受不仅仅是`POST`请求。作为网络应用开发者，我们可能需要支持各种平台。拥有一个 API 简化了我们服务器与其他平台之间数据的传输。许多 API 可以接受 GET 和 POST 请求，以及 PUT、PATCH、DELETE 或 OPTIONS。

一个`+server.js`文件通过导出一个函数来创建一个 API 端点，该函数的名称是你希望它接受的 HTTP 请求方法。导出的函数将接受 SvelteKit 特定的`RequestEvent`参数，并返回一个`Response`对象。例如，我们可以创建一个简单的端点，允许我们为博客创建帖子。如果我们使用移动应用来撰写和发布，这可能很有用。请注意，`+server.js`文件不应与页面文件一起存在，因为它旨在处理所有 HTTP 请求类型：

src/routes/api/post/+server.js

```js
import { json } from '@sveltejs/kit';
export function POST({ request }) {
  // save post to DB
  console.log(request);
  return json({
    status: true,
    method: request.method
  });
}
export function GET({ request }) {
  // retrieve post from DB
  console.log(request);
  return json({
    status: true,
    method: request.method
  });
}
```

此文件从`@sveltejs/kit`包中导入`json`模块，这对于发送 JSON 格式的`Response`对象很有用。然后我们导出名为 POST 和 GET 方法的函数，每个函数都只将`Request`对象输出到控制台，然后返回一个 JSON 格式的`Response`。如果我们愿意，我们还可以导出其他 HTTP 动词（如 PUT、PATCH 或 DELETE）的函数。

你应该通过在浏览器中导航到`api/post/`路由来演示这个示例。打开页面后，观察输出到你的开发服务器的`Request`对象中可用的其他属性。如果你不理解你看到的内容，那没关系，因为我们在下一章中会更深入地探讨它。回到你的浏览器，打开返回的 JSON 格式对象中设置为 POST 的`method`属性。如果你的浏览器不允许你编辑请求，OWASP ZAP、Burp Suite、Postman 或 Telerik Fiddler 等代理工具将允许你自定义 HTTP 请求。请参阅本章末尾的资源链接。

现在你已经知道了如何创建你自己的 API，让我们看看如何通过布局来统一你应用程序的用户体验。

# 创建布局

到目前为止，我们已经在本章中介绍了很多内容，但我们仍然只为每个特定页面添加了样式和标记。这是重复的，并且不是我们时间的高效利用。为了减少重复，我们可以利用布局。一个`+layout.svelte`组件可以通过利用 Svelte 的`<slot>`指令来统一用户体验。布局文件将任何兄弟页面组件和子路由嵌套在其中，使我们能够在整个应用程序中显示持久标记。就像`+page.svelte`一样，我们可以在我们的路由层次结构的任何级别包含`+layout.svelte`组件，允许布局嵌套在布局中。因为每个布局也是一个 Svelte 组件，样式将局部化到该特定组件，并且不会传播到嵌套在其内的组件。让我们看看我们如何使用布局来为现有的代码创建一个一致的布局和导航菜单：

src/routes/+layout.svelte

```js
<script>
  import Nav from '$lib/Nav.svelte';
</script>
<div class='wrapper'>
  <div class='nav'>
    <Nav />
  </div>
  <div class='content'>
    <slot />
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
</style>
```

由于此`+layout.svelte`组件位于我们路由的根级别，它将应用于所有子路由。我们的组件首先做的事情是导入我们的自定义导航组件（如下所示）。其次，它创建将容纳我们应用程序其余部分的标记，包括此文件的兄弟`+page.svelte`。其标记由几个具有不同类名的`<div>`元素组成，这些元素表示功能。具有`.wrapper`类的`<div>`元素包裹了所有其他元素，这样我们就可以应用在组件的`<style>`部分中找到的粘性页脚样式。具有`.nav`类的`<div>`包含我们的自定义`Nav`组件，`.content` `<div>`包含我们的 Svelte `<slot>`指令，而`.footer`是我们放置网站页脚信息的地方。现在让我们看看我们在根布局中导入的自定义`Nav`组件：

src/lib/Nav.svelte

```js
<nav>
  <ul>
    <li><a href='/'>Home</a></li>
    <li><a href='/news'>News</a></li>
    <li><a href='/fetch'>Fetch</a></li>
    <li><a href='/comment'>Comment</a></li>
    <li><a href='/about'>About</a></li>
    <li><a href='/api/post'>API</a></li>
  </ul>
</nav>
<style>
  ul {
    list-style: none;
    text-align: center;
  }
  ul li {
    display: inline-block;
    padding: 0;
    margin: 1em .75em;
  }
</style>
```

此组件仅包含 HTML，其中包含指向我们已创建的所有路由的链接和一些基本的样式。它由具有`href`属性设置为相对路由的`<a>`元素组成，这些元素嵌套在一个无序列表的列表项中，该列表项位于`<nav>`元素内。再次强调，这个例子过于简单，但就我们的目的而言，它是有效的。现在，您可以将本书后面创建的任何新路由添加到导航菜单中，以便在浏览器中测试时可以轻松访问。

相对路由

注意提供的路由是相对的，并且不以域名开头。如果我们要将我们的生产应用程序部署到子目录，而不是域名的根目录，这些路由将失败。相反，我们可以在 `svelte.config.js` 中设置应用程序的基本路径。具体来说，我们将设置 `config.kit.paths.base` 为我们的子目录路径，以 `/` 开头。然后在组件和路由中，我们可以使用 `import { base } from $app/paths` 并在所有路由前加上 `{base}/`。这样，我们的应用程序就会知道它存在于一个子目录中。尝试在你的开发项目中这样做，并观察 Vite 和 SvelteKit 如何自动从这个目录中提供服务！

为了进一步练习与布局相关的概念，尝试创建 `src/routes/news/[slug]/+layout.svelte` 以使文章具有一致的外观。就像我们看到的 `+page.svelte` 文件一样，`+layout.svelte` 文件可以与 `+layout.js` 或 `+layout.server.js` 文件一起使用。它们的功能与页面对应文件相同，但它们返回的数据将可用于 `+layout.svelte` 以及任何并存的 `+page.svelte` 页面。页面选项也可以在布局文件中设置，并且这些选项将“渗透”到嵌套组件中。

根据提供的信息，你现在应该具备为你的 SvelteKit 应用程序生产一致且健壮的 UI 所需的技能。布局在创建各种 UI 元素时非常有用，尤其是那些需要在应用程序的各个部分保持一致性的元素。

# 摘要

在本章中，我们介绍了如何创建静态和动态路由以及管理这些路由的自定义错误模板。我们还看到了开发者如何通过单个表单调用多个命名操作来接受通过 `<form>` 元素提交的数据。我们学习了如何利用 SvelteKit 的路由机制构建 API，这对于需要从除网络浏览器以外的平台访问应用程序尤其有用。然后我们通过布局统一了应用程序的 UI。有了这些布局，我们看到了它们如何被利用来在应用程序的各个部分保持导航元素的可预测位置。在这几页中要吸收这么多信息确实很多，所以我们将在这接下来的几章中更详细地探讨一些这些概念。

在下一章中，我们将学习更多关于管理我们加载到页面上的数据的方法。我们还将介绍更多高级的数据加载方法。

# 资源

**HTTP** **代理/发送工具**:

+   OWASP ZAP (Web 应用渗透测试和代理): [`www.zaproxy.org/`](https://www.zaproxy.org/).

+   Burp Suite (Web 应用渗透测试和代理): [`portswigger.net/burp`](https://portswigger.net/burp).

+   Postman (API 测试工具): [`www.postman.com/`](https://www.postman.com/)

+   Telerik Fiddler (Web 调试和代理工具): [`www.telerik.com/fiddler`](https://www.telerik.com/fiddler).
