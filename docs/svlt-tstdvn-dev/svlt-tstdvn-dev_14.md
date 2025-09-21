

# 第十四章：测试认证

许多网络应用程序将涉及用户认证。本章展示了如何为这种功能编写测试。这些测试涵盖了登录、注销，并确保你的应用程序仅对已登录用户可访问。

本章不是教程，只包含少量关于实现认证所需的应用程序代码的细节。本书仓库使用 **Auth.js** 库，但相同的测试技术无论采用何种实现方法都适用。

本章涵盖了以下关键主题：

+   使用 Playwright 测试认证

+   使用 Vitest 测试认证

到本章结束时，你将看到如何编写涵盖所有认证方面的测试。

# 技术要求

本章的代码可以在网上找到，地址为 [`github.com/PacktPublishing/Svelte-with-Test-Driven-Development/tree/main/Chapter14/Complete`](https://github.com/PacktPublishing/Svelte-with-Test-Driven-Development/tree/main/Chapter14/Complete)。

如果你想运行这个示例中的代码，你需要创建一个包含一些环境变量的 `.env` 文件。有一个名为 `.env.example` 的示例文件，你可以复制并保存为 `.env`，这应该可以工作，但如果你想尝试 GitHub OAuth 集成，你需要在 GitHub 内进行一些配置。

你可以在仓库的 `README.md` 文件中找到更详细的信息。

# 使用 Playwright 测试认证

本节详细说明了使用 Playwright 测试进行有效测试所需的基础工作。首先，我们看看如何为端到端测试提供硬编码的认证凭据。然后我们使用这些凭据来验证用户能否登录和注销应用程序。最后，我们将更新现有的测试，以确保在尝试测试应用程序功能之前用户已经登录。

Cucumber 呢？

如果你使用 Cucumber 测试与 Playwright，那么这里展示的相同技术也适用于你。

## 为开发和测试模式创建一个认证配置文件

使用 OAuth 认证策略将认证责任委托给第三方提供者，如 Google 或 GitHub，这在很大程度上是典型的。然而，当涉及到编写端到端测试时，仅为了测试目的维护这些第三方用户账户是不切实际的。一方面，账户密码会过期并需要定期重置。这类工作需要被记录、跟踪和安排。

另一种解决方案是提供一个特殊的硬编码凭据，当应用程序服务器以特定测试模式运行时，可以使用该凭据登录。然后你的 Playwright 测试可以使用此凭据进行登录。

我们的解决方案如下：

1.  Playwright 使用一个额外的环境变量 `VITE_ALLOW_CREDENTIALS` 启动服务器，该变量设置为 `true`。

1.  `Auth.js` 初始化代码寻找这个凭证，如果找到，将通过其凭证机制启用登录。有一个单一的用户 `api`，与之关联的密码为空。

1.  这个用户可以由 API 测试和您的应用程序开发模式使用。

为了确保 Playwright 以正确的环境变量启动，`playwright.config.js` 文件需要进行如下更改：

```js
webServer: {
  command:
    'npm run build && npm run preview',
  port: 4173,
  env: {
    PATH: process.env.PATH,
    VITE_ALLOW_CREDENTIALS: true
  }
},
```

然后，应用程序有一个名为 `src/authProviders.js` 的文件，该文件检查凭证，如下面的代码片段所示。这个样本是针对 `Auth.js` 的，但其他身份验证库也可以类似地初始化。关键点是预期的 `authProviders` 是一个可能包含也可能不包含特殊凭证提供程序的数组，这取决于环境变量的值：

```js
import GitHubProvider from '@auth/core/providers/github';
import CredentialsProvider from '@auth/core/providers/credentials';
import {
  GITHUB_ID,
  GITHUB_SECRET
} from '$env/static/private';
const allowCredentials =
  import.meta.env.VITE_ALLOW_CREDENTIALS === 'true';
const GitHub = GitHubProvider({
  clientId: GITHUB_ID,
  clientSecret: GITHUB_SECRET
});
const credentials = CredentialsProvider({
  credentials: {
    username: { label: 'Username', type: 'text' }
  },
  async authorize({ username }, req) {
    if (username === 'api')
      return { id: '1', name: 'api' };
  }
});
const devAuthProviders = {
  GitHub,
  credentials
};
const prodAuthProviders = { GitHub };
export const authProviders = allowCredentials
  ? devAuthProviders
  : prodAuthProviders;
```

应用程序准备就绪后，即可在我们的测试中使用。

## 编写登录测试

拥有一个成功登录流程的测试和一个失败的测试非常重要。拥有这些测试确保了您对这些页面的自动化测试覆盖率。

这里是一个成功登录测试的示例。它可以在 `tests/login.test.js` 仓库文件中找到。它导航到常规的 `/birthdays` 路由，然后寻找名为 `api` 用户的按钮，并再次点击该按钮（这次，它起到提交登录信息的作用）。然后检查用户是否被重定向到主页并可以看到 **生日** **列表** 标题：

```js
import { expect, test } from '@playwright/test';
test('logs in and returns to the application', async ({
  page
}) => {
  await page.goto('/birthdays');
  await page.waitForLoadState('networkidle');
  await page
    .getByRole('button', { name: /Sign in with
      credentials/i })
    .click();
  await page.getByRole('textbox').fill('api');
  await page
    .getByRole('button', { name: /Sign in with
      credentials/i })
    .click();
  await expect(
    page.getByText('Birthday list')
  ).toBeVisible();
});
```

注意使用 `page.waitForLoadState` Playwright 函数。这是必要的，以确保所有相关的 `Auth.js` 代码都已运行，并最终渲染登录按钮。

接下来是测试失败的登录。为此，我们可以提供除 `api` 之外任何用户名，因此这个测试提供了 `未知用户` 文本，为数据提供了一些有用的上下文：

```js
test('does not log in if log in fails', async ({
  page
}) => {
  await page.goto('/birthdays');
  await expect(
    page.getByText('Please login')
  ).toBeVisible();
  await page.waitForLoadState('networkidle');
  await page
    .getByRole('button', { name: /Sign in with
      credentials/i })
    .click();
  await page
    .getByRole('textbox')
    .fill('unknown user');
  await page
    .getByRole('button', { name: /Sign in with
      credentials/i })
    .click();
  await expect(
    page.getByText(
      'Sign in failed. Check the details you provided are
        correct.'
    )
  ).toBeVisible();
});
```

这涵盖了新登录功能的测试。接下来，我们需要更新现有测试以确保它们继续工作。

## 更新现有测试以验证用户身份

我们在 `tests/birthdays.test.js` 中现有的测试需要更新，以便每个测试都以登录用户开始。我们可以使用 `beforeEach` 块来完成此操作，它具有原始测试不需要修改的优点。

Auth.js 提供了一个整洁的 API 类型的端点，我们可以直接调用。这意味着我们不需要为每个测试导航通过网页表单，这减少了每个测试需要完成的工作量。

`login` 函数在下面的代码片段中定义。它模仿了点击 `api` 的 `username` 字段值的行为。发送此请求时，还重要地发送 `origin` 标头；否则，它将被拒绝：

```js
const login = async ({ context, baseURL }) => {
  const response = await context.request.get(
    '/auth/csrf'
  );
  const { csrfToken } = await response.json();
  const response2 = await context.request.post(
    '/auth/callback/credentials',
    {
      form: {
        username: 'api',
        csrfToken
      },
      headers: {
        origin: baseURL
      }
    }
  );
};
```

然后，可以通过将其发送到 `test.beforeEach` 来为 `tests/birthday.test.js` 中的每个测试触发它，如下所示：

```js
test.beforeEach(login);
```

这就完成了 Playwright 测试。你会注意到我省略了任何检查当你未认证访问`/birthday`路由时会发生什么的测试。我们将在下一节的 Vitest 测试中涵盖这一点。

# 使用 Vitest 测试认证

现在我们深入一层，具体来看。我们的测试将集中在`/birthdays`路由以及它如何根据认证数据呈现。

Auth.js 库利用 SvelteKit 的会话机制将认证信息传递到组件中，所以我们通过`parent.session`对象和`locals.getSession`函数来实现这一点。我们只需要使用测试双胞胎来模拟我们想要的响应。

我们首先定义一个会话工厂，它可以用来设置这些会话测试双胞胎。然后我们将更新页面加载测试以包含新的认证功能，最后，我们将更新表单操作测试。

## 定义会话工厂

这里是`src/factories/session.js`的定义，它定义了四个导出，这些导出在随后的测试中使用：

```js
import { vi } from 'vitest';
const validSession = { user: 'api ' };
export const loggedInSession = () => ({
  session: validSession
});
export const loggedOutSession = () => ({
  session: null
});
export const loggedInLocalsSession = () => ({
  getSession: vi.fn().mockResolvedValue(validSession)
});
export const loggedOutLocalsSession = () => ({
  getSession: vi.fn().mockResolvedValue(null)
});
```

可以使用`loggedInSession`对象作为传递给页面加载的`parent`属性。Auth.js 认证过程将在你的路由加载之前运行，并将其合并到这个`parent`值中。所以，`loggedInSession`只是一个占位符对象：在我们的测试上下文中，任何值都构成一个有效的、已登录的用户。

`loggedOutSession`对象类似：这次，`session`的`null`值表示用户未认证。

`loggedInLocalsSession`和`loggedOutLocalsSession`值将用于替换传递给表单操作的 SvelteKit 的`locals`属性。这个属性是一组函数，表单操作可以使用这些函数。

接下来，我们将看到如何使用这些测试。

## 更新现有的页面加载函数测试

现在，让我们更新页面加载函数，使其具有必要的加载函数。我们还将编写一个测试来确保如果用户未认证，页面不会加载任何数据。

在`src/routes/birthdays/page.server.test.js`中需要一个新的导入，如下面的代码块所示。这些是两个工厂，将用于为传递给`load`函数的`parent`属性提供值：

```js
import {
  loggedInSession,
  loggedOutSession,
} from 'src/factories/session.js';
```

然后，`describe`块被更新以创建一个默认为已登录用户的`parent`变量：

```js
describe('/birthdays - load', () => {
  const parent = vi.fn();
  beforeEach(() => {
    parent.mockResolvedValue(loggedInSession());
  });
  ...
});
```

每个测试也需要更新，以便将这个新的`parent`属性传递给`load`函数：

```js
it('calls fetch with /api/birthdays', async () => {
  const fetch = vi.fn();
  fetch.mockResolvedValue(fetchResponseOk());
  const result = await load({ fetch, parent });
  expect(fetch).toBeCalledWith('/api/birthdays');
});
```

你还想要添加一个测试来检查端点在没有正确认证的情况下是否无法工作。在下面的示例中，我们用`loggedOutSession`覆盖了默认的`parent`属性，然后测试返回的页面有一个`303`状态，这意味着浏览器正在被重定向。它还检查被重定向到的页面是`/login`路由：

```js
it('redirects if the request is not authorised', async () => {
  parent.mockResolvedValue(loggedOutSession());
  expect.hasAssertions();
  try {
    await load({ parent });
  } catch (error) {
    expect(error.status).toEqual(303);
    expect(error.location).toEqual('/login');
  }
});
```

这就完成了`load`函数的测试。

## 更新现有的表单操作测试

对于表单动作测试，`import` 语句被更新为剩余的两个工厂函数：

```js
import {
  loggedInSession,
  loggedOutSession,
  loggedInLocalsSession,
  loggedOutLocalsSession
} from 'src/factories/session.js';
```

然后，`describe` 块被更新以包含在 `beforeEach` 中设置的 `locals` 变量。注意这次变量被定义为 `let`，因为 `loggedInLocalsSession` 工厂负责设置 `vi.fn` 间谍函数：

```js
describe('/birthdays - default action', () => {
  const fetch = vi.fn();
  let locals;
  beforeEach(() => {
    fetch.mockResolvedValue(fetchResponseOk());
    locals = loggedInLocalsSession();
  });
  ...
});
```

然后 `performFormAction` 被更新以包含 `locals` 属性。由于这是我们测试调用来触发表单动作的函数，因此测试本身不需要更改：

```js
  const performFormAction = (formData) =>
    actions.default({
      request: createFormDataRequest(formData),
      fetch,
      locals
    });
```

最后，我们需要一个测试来检查当用户未认证时提交表单会发生什么。在这种情况下，我们返回 `300` 状态码而不是重定向，但您可以选择回到上一个表单页面，就像处理验证错误时一样。这有助于用户会话已过期的场景。以下是更简单的版本：

```js
describe('when not authorised', () => {
  beforeEach(() => {
    locals = loggedOutLocalsSession();
  });
  it('returns a failure', async () => {
    const result = await performFormAction({});
    expect(result.status).toEqual(300);
  });
});
```

这完成了支持我们的认证实现所需的 Vitest 变更。

# 摘要

本章简要介绍了测试认证所需的测试类型。

Playwright 端到端测试应检查登录流程，无论是成功还是失败。它们还应尽可能使用假凭据，并确保任何路由都只能由使用 `test.beforeEach` 调登录的认证用户访问。

Vitest 对认证路由的测试工作方式不同。它们关注 SvelteKit 返回的 `session` 对象：它是否有有效的值？

虽然这涵盖了基础知识，但大多数应用程序将具有更复杂的需求：例如，您的应用程序可能为每个用户都有单独的数据存储，而不仅仅是我们的生日应用程序中的一个全局数据存储。

Playwright 网站上有关于如何测试特定模式的好文档，例如在单个测试中让多个用户角色交互。可以在 [`playwright.dev/docs/auth`](https://playwright.dev/docs/auth) 找到。

在下一章中，我们将探讨测试 Svelte 存储。
