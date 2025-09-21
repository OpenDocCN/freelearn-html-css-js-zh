

# 表单和数据提交

在上一章中，我们探讨了 SvelteKit 中加载数据的一些细节。虽然加载数据很重要，但同样重要的是我们要了解如何让用户提交数据。这就是为什么本章将探讨 SvelteKit 中表单和动作的一些细节。虽然并非所有应用程序都必须接受用户的数据，但那些以直观方式这样做的一般会脱颖而出。毕竟，一些最好的用户体验被认为是理所当然的，因为它们只是简单地工作。只有当事情出错时，用户才会开始关注它们。

在本章中，我们将学习如何利用`<form>`元素来保持我们的应用程序的可访问性和代码的简洁性。将这些表单与易于实现的动作集成，使我们能够根据需要处理提交的数据。最后，我们将探讨如何通过添加渐进式增强来软化围绕表单的标准用户体验的一些边缘。为了完成所有这些，我们将对之前开始的登录表单进行最后的润色。

本章将涵盖以下内容：

+   表单设置

+   动作分析

+   提高表单功能

完成本章后，你应该能够轻松地创建自己的登录表单，并且你会知道如何继续前进并接受你基于 SvelteKit 的应用程序的用户的所有类型的数据。

# 技术要求

本章的完整代码可在 GitHub 上找到：[`github.com/PacktPublishing/SvelteKit-Up-and-Running/tree/main/chapters/chapter06`](https://github.com/PacktPublishing/SvelteKit-Up-and-Running/tree/main/chapters/chapter06)。

# 表单设置

我们在*第四章*中简要地看了如何一起使用表单和动作。虽然在上一章中涵盖了`RequestEvent`，我们开始创建必要的代码以使用 cookie 在我们的应用程序中验证用户。然而，在那个例子中，我们从未给用户提供提供用户名或密码的手段。我们也从未在应用程序中创建 cookie。相反，我们选择使用浏览器的开发者工具手动创建一个。现在是时候将所有这些整合在一起了。由于我们已经涵盖了与`load()`相关的逻辑以及与`RequestEvent`相关的细节，我们可以继续构建之前的例子。一个不错的起点是登录表单本身。毕竟，如果没有提供一个地方让他们登录，我们就无法让用户登录。

但在我们创建表单之前，让我们先在我们的导航中添加一个指向登录页面的链接：

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
    <li><a href='/login'>Login</a></li>
  </ul>
</nav>
```

这个更改就像复制一个现有的`<li>`元素，并替换`<a>`内的路由和文本一样简单。这将使导航和测试我们的登录功能变得更加简单。

接下来，让我们从实际的表单开始。回顾一下我们创建的用于根据用户的 cookie 显示成功登录的文件，我们需要进行一些修改。首先，我们将从 `$app/forms` 中导入 `enhance` 模块。我们将在本章后面讨论这个背后的部分魔法，所以现在不用担心。接下来，我们将想要导出 `form` 变量，以便我们可以向用户指示他们的登录状态。最后，我们需要创建一个带有适当输入的 `<form>` 元素，并给它一些样式：

src/routes/login/+page.svelte

```js
<script>
  import { enhance } from '$app/forms';
  export let data;
  export let form;
</script>
{#if form?.msg}
  {form.msg}
{/if}
{#if data.user}
  <p>
    Welcome, {data.user.name}!
  </p>
{/if}
<form use:enhance method="POST" action="?/login">
  <label for="username">Username</label>
  <input name="username" id="username" type="text"/>
  <label for="password">Password</label>
  <input name="password" id="password" type="password"/>
  <button>Log In</button>
</form>
<style>
  form {
    display: flex;
    flex-direction: column;
    justify-content: center;
    width: 25%;
    margin: 0 auto;
  }
  input {
    margin: .25em 1em 1em;
    display: block;
  }
  label {
    margin: 0 .5em;
  }
</style>
```

现在您已经看到了所有的添加内容，让我们来讨论一下。除了新的导入和导出之外，您接下来会注意到的下一个变化是 Svelte 指令检查 `form?.msg` 是否已设置。如果已设置，我们将显示该消息。

数据与表单

`form` 属性来自我们的登录操作返回的数据（将在下一节中创建）。记住，我们包含 `export let data;` 以获取 `load()` 返回的数据属性。同样，我们包含 `export let form;` 以检索由表单操作返回的数据。返回的数据也可以通过应用程序中的 `$page.form` 存储在任何地方检索。

下一个重大变化是添加了 `<form>` 元素。它使用了我们之前导入的 `enhance` 模块，将 HTTP 方法设置为 POST，并将数据发送到位于 `src/routes/login/+page.server.js` 的 `login` 操作。我们必须将 HTTP 方法设置为 POST；否则，我们的表单将尝试通过 GET 请求提交数据，我们不希望在不安全的情况下发送密码。

我们随后包括了输入、标签和按钮的适当标记。目前，我们只引用了一个表单操作来管理用户的登录。如果我们想启用注册或密码重置功能，并保持后续操作与我们的登录操作在同一文件中，我们可以利用 `formaction` 属性。然而，`formaction` 属性旨在用于您有多个按钮引用同一 `<form>` 元素内的不同端点时。在密码重置场景中，我们可能需要另一个 `<form>` 元素来指定发送密码重置链接的电子邮件。同样，在注册的情况下，我们可能需要获取用户的电子邮件，以及他们的用户名和密码，因此在只接受用户名和密码详情的表单中包含这两者几乎没有意义。在这些情况下，为每个功能创建一个单独的表单并直接在 `<form>` 元素上指定操作会更有意义。出于项目组织的目的，仍然可能有必要将有关身份验证的逻辑保持在单个 `+page.server.js` 文件中。

创建表单的实际标记相对简单易行。我们刚刚看到，我们需要指定方法以及要调用的动作。为了获取从我们的动作返回的信息，我们需要在页面上包含`export let form;`并利用从动作返回的数据。现在你已经看到了它的几种变体，你应该能够轻松地创建表单以从用户那里获取数据。当然，如果我们不利用提交的数据，表单就没有什么用处。在下一节中，我们将创建一个动作来处理表单收集的数据。为了确保我们的动作能够顺利工作，我们需要设置一个数据库并讨论一些安全最佳实践。

# 分析动作

在*第四章*中，我们花了一部分时间来探讨动作是如何工作的。现在，是时候更深入地研究它们以及它们在底层是如何工作的了。但在我们开始之前，我们需要设置另一个模拟数据库并简要讨论安全问题。一旦我们完成这些，我们将完成向我们的应用程序添加逻辑并验证有效用户的工作。本节将涵盖以下内容：

+   数据库设置

+   密码和安全

+   登录动作

经过所有这些，你将大致了解如何最终为你的 SvelteKit 应用程序创建一个登录表单。

## 数据库设置

当然，这并不是一个真正的数据库。我们将利用另一个 JSON 文件来存储我们的用户数据，并帮助我们模拟查找用户及其散列密码。它看起来应该像这样：

src/lib/users.json

```js
[
  {
    "id": "1",
    "username": "dylan",
    "password": "$2b$10$7EakXlg...",
    "identity": "301b1118-3a11-...",
    "name": "Dylan"
  },
  {
    "id": "2",
    "username": "jimmy",
    "password": "$2b$10$3rdM9VQ...",
    "identity": "62e3e3cc-adbe-...",
    "name": "Jimmy"
  }
]
```

此文件是一个简单的数组，包含两个用户对象以及与我们的用户相关的各种属性。在这个演示中，这些属性的值是微不足道的，但`identity`和`password`的值对我们特别感兴趣，正如我们将在下一节中看到的。`identity`属性通常对应于存储在另一个表中的用户会话 ID。它应该使用一个唯一的标识符，并且不容易被猜到。如果它很容易被猜到，任何人都可以通过在他们的设备上创建带有有效会话 ID 的标识符 cookie 来认证我们的应用程序作为任何用户。在这个例子中，`identity`使用了**Crypto Web API**来生成一个随机的**通用唯一标识符**（**UUID**）。不应该使用 Crypto Web API 进行密码散列。在这个演示中，我们只会用它来创建一个将被保存在用于认证用户的 cookie 中的 UUID。为了测试目的，这个值可以是任何唯一的字符串，但这个例子旨在相对真实。为了使这些材料不偏离学习 SvelteKit 的指导方针，我们只需要包括这些内容来构建我们的模拟用户数据库。

## 密码和安全

由于身份验证是网络应用中非常常见的功能，如果不进一步阐述如何正确实现它，那就太不合适了。而且因为如果实现不当，可能会产生灾难性的后果，我们将学习如何安全地实现它。虽然我们目前还没有连接到真实的数据库，而是将用户密码存储在 JSON 文件中（除了演示目的之外，强烈建议不要这样做），但我们将观察如何使用另一个通过 npm 安装的包正确地哈希密码。

要继续前进，我们需要安装**bcrypt**。在你的终端中，在项目目录中运行以下命令：

```js
npm install bcrypt
```

完成这些后，我们可以使用以下代码生成哈希。这段代码将是临时的，因为它将为我们提供一个方便的方法来生成密码的哈希以及用户对象`identity`属性的 UUID。然后可以将这些添加到你的`users.json`文件中，模拟从数据库中查找用户密码。我们将在之后演示登录功能来利用它：

src/routes/login/+page.server.js

```js
import bcrypt from 'bcrypt';
export const actions = {
  login: async ({request}) => {
    const form = await request.formData();
    const hash = bcrypt.hashSync(form.get('password'), 10);
    console.log(hash);
    console.log(crypto.randomUUID());
  }
}
```

这个`+page.server.js`文件导入了我们刚刚使用 npm 安装的`bcrypt`模块。然后它创建了一个登录操作，我们的`<form>`元素会将数据提交到这个操作。它通过`crypto.randomUUID();`从开发者工具的控制台检索登录表单提交的表单数据。当你在浏览器中导航到`/login`，填写表单的密码字段，提交它，然后在你的终端中打开服务器控制台时，你将能够将哈希和随机生成的 UUID 复制到每个用户的`users.json`文件中相应的`password`和`identity`属性中。这样，你就可以为你的用户创建密码。如果你正在使用本书 GitHub 仓库中的代码，每个用户的哈希都是从以下字符串派生出来的：

1.  **password**

1.  **jimmy**

注意

不言而喻，这些被认为是糟糕的密码。在任何情况下都不应该尝试使用这些密码或它们的哈希，除非是在这个演示中。

在讨论安全性的话题时，我们应该抓住这个机会指出一些应避免的做法。重要的是我们开发者不要使用*共享变量*来存储敏感数据。为了澄清，这并不意味着不能使用变量来存储敏感数据。相反，我们应该避免在表单操作中设置变量，这样变量随后可能会在`load()`函数中可用。以一个不良做法的例子来说，考虑一个开发者在一个`+page.server.js`文件的最高作用域级别声明一个变量来存储聊天消息，在表单操作中将消息数据赋值给它，然后在同一文件的`load()`函数中返回相同的变量。这样做可能会让用户 B 查看用户 A 的聊天消息。通过立即将数据返回到页面，可以避免这种数据泄露。这些相同的指南也适用于 Svelte 存储。在服务器上管理数据时，我们永远不应该设置存储的状态，因为在服务器上这样做可能会使所有该服务器上的用户都能访问到。

现在我们已经知道了如何创建哈希密码和 UUID，我们可以遵循一些关于安全性的基本最佳实践。如果你有任何疑问，请查阅官方 SvelteKit 文档。随着技术的变化，最佳实践也可能随之改变。在下一节中，我们将看到如何通过创建一个将所有这些联系在一起的操作来最终完成登录表单。

## 登录操作

在完成所有这些设置之后，我们终于做到了。现在我们可以完成表单使用的操作来让我们的用户登录并在他们的浏览器中设置一个 cookie。我相信你现在已经准备好了，让我们深入探讨吧。

在这个文件中，我们之前创建了一些代码来生成密码的哈希值以供测试。我们可以去掉这些代码，用代码替换它，该代码将在我们的数据库中查找匹配的用户名，将提供的密码与找到的用户哈希密码进行比较，并在用户的设备上设置一个 cookie：

src/routes/login/+page.server.js

```js
import bcrypt from 'bcrypt';
import users from '$lib/users.json';
export const actions = {
  login: async ({request, cookies}) => {
    const form = await request.formData();
    const exists = users.filter(user => user.username === form.
      get('username'));
    const auth = exists.filter(user => bcrypt.compareSync(form.
      get('password'), user.password));
    if(!exists.length || !auth.length) {
      return {msg: 'Invalid login!'};
    }
    cookies.set('identity', auth[0].identity, {path: '/'});
    return {msg: 'success!'}
  }
}
```

在这个新版本中，我们仍然导入了`bcrypt`模块，但我们还添加了`user.json`的导入。然后我们向解构的`RequestEvent`参数中添加了`cookies`。在设置`login`操作后，我们获取由`<form>`元素提交的数据，并将其放入`form`常量中。接下来，我们使用`filter()`来检查`users`数组中每个元素的用户名。任何匹配项都被添加到`exists`常量中。然后我们再次使用`filter()`来检查提交的密码与`exists`中每个用户的哈希密码。如果找到匹配项，它将被添加到`auth`常量中。如果`exists`或`auth`的数组中没有项目，我们返回一条消息，表示登录尝试无效。

对抗账户枚举

我们绝不应该返回一个消息，告诉用户他们的用户名（或电子邮件）是正确的，但提供的密码失败了。这样做将允许恶意行为者枚举有效账户，本质上是在短时间内连续猜测用户名。一旦完成，攻击者就可以轻松地编制一个有效用户名的列表，并开始在真实账户上尝试暴力破解密码。由于用户并不以创建强密码而闻名，这可能导致多个账户的账户接管。这就是为什么我们只返回一个消息，告诉用户他们的登录尝试失败了。无论他们的用户名或密码是否正确，这是他们需要自己弄清楚的事情。

如果用户成功登录，我们使用`cookies.set()`发送`Set-Cookie`头部信息，告诉客户端将`identity`cookie 设置为用户的会话 ID，在域名的根目录下。我们必须在选项中指定根路径；否则，我们的 cookie 将默认只在设置的最高级别路由上工作——在这种情况下，仅在诸如`/login`之类的页面上工作。你可以想象这对用户来说是多么令人沮丧。然后我们可以检查用户是否有权访问应用程序中各个位置的功能。要删除相同的 cookie 并注销用户，我们可以使用`cookies.delete()`，同时传入 cookie 的名称以及我们的路径。

最后，为了向用户显示他们的登录尝试是否成功，我们需要对我们的根服务器布局进行一些调整。如果你还记得，我们之前只检查了`identity === '1'`。在实现了假数据库之后，我们可以改为与我们的用户 JSON 文件进行比对：

src/routes/+layout.server.js

```js
import users from '$lib/users.json';
export function load({ cookies }) {
  const data = {
    notifications: {
      count: 3,
      items: […]
    }
  };
  const exists = users.filter(user => user.identity === cookies.get('identity'));
  if(exists.length) {
    const {password, …user} = {...exists[0]};
    data.user = user;
  }
  return data;
}
```

由于我们需要将`identity`cookie 的值与`users.json`中存在的值进行比对，我们首先需要导入它。目前我们不需要对`data`常量进行任何更改，因此我们可以保留与通知相关的代码。然后我们必须使用`filter()`来查找是否有任何用户具有从身份 cookie 获取的值，并将找到的这些用户分配给`exists`常量。如果`exists`有值，我们就获取第一个找到的值，并利用解构赋值的强大功能来避免将用户的密码传递给`data.user`。这样做是为了防止包含敏感数据。

现在我们已经将所有这些放在一起，我们可以通过在浏览器中导航到`/login`并输入适当的详细信息来验证登录是否正常工作。如果你创建了自定义的散列，你需要使用你提供的字符串来成功进行身份验证。提交表单后，我们应该看到状态消息，以及来自`src/routes/login/+page.svelte`的欢迎消息。

回顾一下，在本节中，我们使用 JSON 文件创建了一个假的用户数据库。我们在该文件中包含了我们的安全密码散列以供检查。当从`src/routes/login/+page.svelte`中的`<form>`元素提交用户名和密码时，使用`src/routes/login/+page.server.js`检索这些数据。然后我们检查用户名以及该用户的散列密码；如果找到匹配项，我们通过`cookies.set()`发送响应中的`Set-Cookie`头，并发送一个*成功*状态消息。如果登录尝试与用户名或密码不匹配，我们返回一个*无效登录*状态消息。现在我们已经知道如何创建表单并将我们的数据提交到适当的操作，让我们来探讨一些可以改善我们应用程序用户体验的方法。

# 增强表单

为了减少网页表单中固有的摩擦，SvelteKit 为我们提供了一些选项。我们之前在设置登录表单的`use:enhance`时看到了这些选项中的第一个。这个选项非常适合防止页面重定向，因为它可以在后台提交表单数据，这意味着我们的页面不需要重新加载。我们还没有看到的是 SvelteKit 所说的**快照**。在本节中，我们将探讨这两个工具以及它们如何帮助改善您应用程序的体验：

+   `enhance`

+   快照

完成本节后，您将能够构建出直观且流畅的表单供您的用户使用，这将大大增加用户接受您应用程序的可能性。

## enhance

通过从`$app/forms`导入`enhance`，我们可以逐步增强`<form>`元素的处理流程。这意味着我们可以在不要求页面重新加载的情况下提交数据，这在提交`<form>`元素时通常是必须的。我们已经看到这个动作出现了几次，但在两种情况下我们都没有讨论它是如何工作的。

`enhance`采取的第一步是更新`form`属性。我们通过位于`src/routes/login/+page.svelte`中的 Svelte 指令观察到这一点，该指令检查`form?.msg`是否已设置。由于 Svelte 是响应式的，当`enhance`更新`form`时，我们可以立即查看更改并显示我们的消息。`enhance`还会更新`$page.form`和`$page.status`，这两个都是 Svelte `$page`存储的属性。这个存储提供了有关当前显示页面的信息。`$page.form`将包含来自表单操作的相同数据，而`$page.status`将包含 HTTP 状态码数据。我们第一次在*第四章*的*动态路由*部分看到了`$page`存储的例子。

在收到成功响应后，`enhance` 将重置 `<form>` 元素，并通过调用 `invalidateAll()` 强制适当的 `load()` 函数重新运行。然后，它将解析任何重定向，渲染最近的 `+error.svelte`（如果发生错误），并将焦点重置到正确的元素，就像页面是第一次被加载一样。

如果 `enhance` 应用于具有指向完全不同路由的动作的 `<form>` 元素，`enhance` `form` 或 `$page`。这是因为它的目的是模拟原生浏览器行为，并且像这样跨路由提交数据通常会触发页面刷新。要强制更新这些属性，您需要传递一个回调函数给 `enhance`。然后，回调函数可以使用 `applyAction` 来相应地更新存储。`applyAction()` 函数接受一个 SvelteKit `ActionResult` 类型，也可以从 `$app/forms` 中导入。

## 快照

用户经常遇到的一种令人沮丧的经历是在填写完一个大型表单后，但在提交该表单之前离开页面。无论原因如何，丢失花费了大量时间输入的数据都是痛苦的。

通过在快照中持久化 `<form>` 数据，我们可以使用户更容易地继续他们离开的地方。用户头痛的减少意味着使用我们的应用程序的体验更好。而且实现起来非常简单，我们只需要导出一个带有 `capture` 和 `restore` 属性设置的 `snapshot` 常量。要看到它的实际效果，让我们持久化我们之前构建的评论表单数据：

src/routes/comment/+page.svelte

```js
<script>
  import { enhance } from '$app/forms';
  export let form;
  let comment = '';
  export const snapshot = {
    capture: () => comment,
    restore: (item) => comment = item
  }
</script>
<div class='wrap'>
  {#if form && form.status === true}
    <p>{form.msg}</p>
  {/if}
  <form method='POST' action='?/create' use:enhance>
    <label>
      Comment
      <input name="comment" type="text" bind:value={comment}>
    </label>
    <button>Submit</button>
    <button formaction='?/star'>Star</button>
    <button formaction='?/reply'>Reply</button>
  </form>
</div>
```

在这个新版本中，我们只做了三项重要的更改：

1.  添加了 `let comment = '';` 以便我们可以捕获并恢复到我们的 JS 中的输入值。

1.  添加了带有 `export` 的快照对象 `const snapshot`：

    1.  当在导航离开页面之前更新页面时，`capture` 属性会调用一个匿名函数。这个函数只需要返回我们希望捕获并在稍后恢复的值——在这种情况下，与 `comment` 相关的值。

    1.  `restore` 在页面加载后立即调用，并将调用它的参数分配给 `comment`。

1.  我们将评论的 `<text>` 输入的值绑定到 `comment` 变量，以便在捕获时检索输入的值，并在恢复时设置。

实施这些更改后，您可以通过在浏览器中打开 `/comment` 路由，输入一个测试评论，然后导航到另一个页面来测试它们。当您在浏览器中点击 *后退* 时，您将观察到恢复的数据正如您离开时一样。因为快照将它们捕获的数据持久化到会话存储中，您可以通过打开浏览器开发者工具来观察这些数据。在 Firefox 中，您可以通过点击菜单中的链接找到它，但不会触发恢复，因为它被视为导航到新页面。

了解如何逐步且无缝地增强你的表单可以带来一个让用户不断回访的体验。通过在后台运行表单提交，我们可以利用 Svelte 的响应性为用户提供即时且有用的反馈。并且通过使用快照，我们可以保存用户在简单或复杂表单上的进度。有了这些知识，你现在可以着手将直观的体验构建到你的应用程序中。

# 摘要

我们本章开始时构建了一个简单的 `<form>` 元素，用于接受用户名和密码。在同一页面组件中，我们将身份验证尝试的状态反馈给用户。之后，我们创建了一个表单操作，用于查找用户名并比较提供的密码与哈希值。如果成功，我们在用户的设备上设置一个 cookie 来登录用户。如果失败，我们通知用户他们的尝试失败了。我们还简要讨论了围绕使用我们的应用程序对用户进行身份验证的一些安全最佳实践。然后我们考察了如何通过使用 `enhance` 和快照来改进 `<form>` 元素的体验。完成所有这些后，我们可以对在未来的 SvelteKit 项目中实施的任何表单充满信心。

到目前为止，我们已经涵盖了所有内容，你应该能够构建一个基本的网站或应用程序。在下一章中，我们将介绍更多高级功能，这些功能可以真正展示使用 SvelteKit 构建的力量。我们将探讨更高级的路由概念，指出它们是如何利用我们已经讨论过的功能，并解释一些尚未涉及的内容。

# 资源

以下为本章的资源：

+   bcrypt: [`github.com/kelektiv/node.bcrypt.js`](https://github.com/kelektiv/node.bcrypt.js)

+   Crypto Web API: [`developer.mozilla.org/en-US/docs/Web/API/Crypto`](https://developer.mozilla.org/en-US/docs/Web/API/Crypto)
