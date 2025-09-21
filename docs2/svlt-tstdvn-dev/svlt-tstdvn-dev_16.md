

# 第十六章：测试驱动服务工作者

本章探讨了**服务工作者**，这些代码片段被安装在浏览器上，并在任何 HTTP 操作之前被调用。这使得它们对于一组特定的功能非常有用，例如启用对您的应用程序的离线访问。本章实现的服务工作者正好提供了这个功能。

通常来说，使用现成的服务工作者而不是自己编写是一个好主意。但是，了解如何测试自己的服务工作者是有教育意义的，因此本书中包含了这部分内容。

术语**可测试性**用来描述编写应用程序代码测试的简单程度。我们构建组件和模块的方式对它们的可测试性有很大影响。服务工作者是这样一个很好的例子，即一开始看起来非常复杂难以测试的东西，通过重构其实现，使得测试变得几乎微不足道。

本章涵盖了以下关键主题：

+   添加用于离线访问的 Playwright 测试

+   实现服务工作者

到本章结束时，你将学会如何测试驱动服务工作者，并且你还将学会一种使你的应用程序代码更容易测试的新技术。

# 技术要求

本章的代码可以在网上找到，地址是 [`github.com/PacktPublishing/Svelte-with-Test-Driven-Development/tree/main/Chapter16/Complete`](https://github.com/PacktPublishing/Svelte-with-Test-Driven-Development/tree/main/Chapter16/Complete)。

# 添加用于离线访问的 Playwright 测试

服务工作者通常具有特定的意图。在我们的例子中，服务工作者使应用程序能够离线使用：加载应用程序会导致它被缓存。如果网络不再可访问，下一次页面加载将从这个缓存中提供，这是服务工作者提供的。

因此，Playwright 测试需要测试在没有网络连接时应用程序的行为。

在编写本文时，Playwright 对服务工作者事件的支撑是实验性的，因此需要通过在 `package.json` 文件中使用 `PW_EXPERIMENTAL_SERVICE_WORKER_NETWORK_EVENTS` 标志来启用：

```js
{
  "scripts": {
    ...,
  "test": "PW_EXPERIMENTAL_SERVICE_WORKER_NETWORK_EVENTS=1
    playwright test",
  ...
  }
}
```

完成这些后，我们就准备好编写测试了。我们需要两个辅助函数。第一个，`waitForServiceWorkerActivation`，可以被任何 Playwright 测试调用，以确保后续命令在服务工作者正在积极缓存新请求之前不会运行。

这段代码可以在 `tests/offline.test.js` 中找到。我把它放在了使用它的单个测试旁边：没有必要将其移动到另一个文件，因为我不会在测试套件的其他地方重用这个函数：

```js
const waitForServiceWorkerActivation = (page) =>
  page.evaluate(async () => {
    const registration =
      await
        window.navigator.serviceWorker.getRegistration();
    if (registration.active?.state === 'activated')
      return;
    await new Promise((res) =>
      window.navigator.serviceWorker.addEventListener(
        'controllerchange',
        res
      )
    );
  });
```

接下来，我们需要一个 `disableNetwork` 函数，它将导致任何网络请求返回网络错误：

```js
const disableNetwork = (context) =>
  context.route('', (route) => route.abort());
```

然后，我们就可以编写测试了：

```js
test('site is available offline', async ({
  page,
  context,
  browser
}) => {
  await page.goto('/birthdays');
  await waitForServiceWorkerActivation(page);
  await disableNetwork(context);
  await page.goto('/birthdays');
  await expect(
    page.getByText('Birthday list')
  ).toBeVisible();
});
```

没有服务工作者，这个测试将会失败，因为第二个 `page.goto` 调用将会出错。在下一节中，我们将看到如何实现这个服务工作者。

# 实现服务工作者

服务工作者有一个奇怪的用户界面。一方面，它们需要依赖于浏览器上下文提供的名为`self`的变量。然后它们需要将监听器附加到某些事件上，并且它们需要使用`event.waitUntil`函数来确保浏览器在假设工作者准备好之前等待其操作完成。

结果表明，在 Vitest 测试中设置`self`的值以及伪造事件相当困难。并非不可能，但困难且费时。

考虑到这种复杂性，实现可测试的服务工作者的技巧是将大部分功能移入另一个模块：每个事件都变成一个简单的函数调用，我们可以测试这个函数调用而不是事件。

然后，我们不对服务工作者模块进行测试。我们仍然有 Playwright 测试为我们提供覆盖率，并且一旦完成，这段代码不太可能发生变化，所以这个特定文件没有单元测试并不是什么大问题。

这个示例，如下代码块所示，存储在`src/service-worker.js`文件中。它几乎将所有功能推入`addFilesToCache`、`deleteOldCaches`和`fetchWithCacheOnError`函数中：

```js
import {
  build,
  files,
  version
} from '$service-worker';
import {
  addFilesToCache,
  deleteOldCaches,
  fetchWithCacheOnError
} from './lib/service-worker.js';
const cacheId = `cache-${version}`;
const appFiles = ['/birthdays'];
const assets = [...build, ...files, ...appFiles];
self.addEventListener('install', (event) => {
  event.waitUntil(addFilesToCache(cacheId, assets));
});
self.addEventListener('activate', (event) => {
  event.waitUntil(deleteOldCaches(cacheId));
  event.waitUntil(self.clients.claim());
});
self.addEventListener('fetch', (event) => {
  if (event.request.method !== 'GET') return;
  event.respondWith(
    fetchWithCacheOnError(cacheId, event.request)
  );
});
```

让我们看看位于`src/lib/service-worker.js`文件中的三个函数的实现。这些函数都使用了 Cache API，我们将通过设置间谍来测试它。

这是`addFilesToCache`，它只是简单地打开相关的缓存并插入所有给定的资产：

```js
export const addFilesToCache = async (
  cacheId,
  assets
) => {
  const cache = await caches.open(cacheId);
  await cache.addAll(assets);
};
```

要开始测试这一点，首先我们需要为`caches`定义一个默认值。在示例仓库中，以下代码位于`src/lib/service-worker.test.js`测试套件中，但您也可以将其放置在 Vitest 设置文件中，如果您有多个测试套件使用 Cache API，这会更有意义：

```js
global.caches = {
  open: () => {},
  keys: () => {},
  delete: () => {}
};
```

现在我们从`addFilesToCache`函数的`describe`块开始。它只需要调用`vi.spyOn`以及一个手工制作的缓存对象。`caches`间谍和`cache`存根仅实现了测试`addFilesToCache`函数所需的功能，不多也不少：

```js
describe('addFilesToCache', () => {
  let cache;
  beforeEach(() => {
    cache = {
      addAll: vi.fn()
    };
    vi.spyOn(global.caches, 'open');
    caches.open.mockResolvedValue(cache);
  });
});
```

然后，测试本身很简单：

```js
it('opens the cache with the given id', async () => {
  await addFilesToCache('cache-id', []);
  expect(global.caches.open).toBeCalledWith(
    'cache-id'
  );
});
it('adds all provided assets to the cache', async () => {
  const assets = [1, 2, 3];
  await addFilesToCache('cache-id', assets);
  expect(cache.addAll).toBeCalledWith(assets);
});
```

接下来，这是`deleteOldCaches`的定义，它稍微复杂一些：

```js
export const deleteOldCaches = async (cacheId) => {
  for (const key of await caches.keys()) {
    if (key !== cacheId) await caches.delete(key);
  }
};
```

结果表明，我们为这个设置的反间谍活动非常简单：

```js
describe('deleteOldCaches', () => {
  beforeEach(() => {
    vi.spyOn(global.caches, 'keys');
    vi.spyOn(global.caches, 'delete');
  });
```

然后，测试本身相当简单。注意每个测试都是自包含的，它们为`cache.keys`函数有自己的存根值：

```js
it('calls keys to retrieve all keys', async () => {
  caches.keys.mockResolvedValue([]);
  await deleteOldCaches('cache-id');
  expect(caches.keys).toBeCalled();
});
it('delete all caches with the provided keys', async () => {
  caches.keys.mockResolvedValue([
    'cache-one',
    'cache-two'
  ]);
  await deleteOldCaches('cache-id');
  expect(caches.delete).toBeCalledWith('cache-one');
  expect(caches.delete).toBeCalledWith('cache-two');
});
it('does not delete the cache with the provided id', async () => {
  caches.keys.mockResolvedValue(['cache-id']);
  await deleteOldCaches('cache-id');
  expect(caches.delete).not.toBeCalledWith(
    'cache-id'
  );
});
```

最后，我们来到`fetchWithCacheOnError`函数，这是三个中最复杂的。这涉及到 Cache API 和 Fetch API，因此我们的测试将需要处理`request`和`response`对象：

```js
export const fetchWithCacheOnError = async (
  cacheId,
  request
) => {
  const cache = await caches.open(cacheId);
  try {
    const response = await fetch(request);
    if (response.status === 200) {
      cache.put(request, response.clone());
    }
    return response;
  } catch {
    return cache.match(request);
  }
};
```

让我们来看看测试设置。除了`caches.open`间谍和`cache`存根；还有一个`successResponse`对象和一个`request`对象。这些有虚拟值：调用`successResponse.clone()`不会返回一个响应，而`request`不是一个真正的请求对象。它们只是字符串。但这就是我们测试所需要的：

```js
describe('fetchWithCacheOnError', () => {
  const successResponse = {
    status: 200,
    clone: () => 'cloned response'
  };
  const request = 'request';
  let cache;
  beforeEach(() => {
    cache = {
      put: vi.fn(),
      match: vi.fn()
    };
    vi.spyOn(global.caches, 'open');
    caches.open.mockResolvedValue(cache);
    vi.spyOn(global, 'fetch');
    fetch.mockResolvedValue(successResponse);
  });
});
```

现在，让我们来看看四个快乐的路径测试。这些测试假设有一个正常工作的网络连接和一个有效的 HTTP 响应，状态码为`200`：

```js
it('opens the cache with the given id', async () => {
  await fetchWithCacheOnError('cache-id', request);
  expect(global.caches.open).toBeCalledWith(
    'cache-id'
  );
});
it('calls fetch with the request', async () => {
  await fetchWithCacheOnError('cache-id', request);
  expect(global.fetch).toBeCalledWith(request);
});
it('caches the response after cloning', async () => {
  await fetchWithCacheOnError('cache-id', request);
  expect(cache.put).toBeCalledWith(
    request,
    'cloned response'
  );
});
it('returns the response', async () => {
  const result = await fetchWithCacheOnError(
    'cache-id',
    request
  );
  expect(result).toEqual(successResponse);
});
```

然后我们对除了`200`以外的 HTTP 状态码进行了测试：

```js
it('does not cache the response if the status code is not 200', async () => {
  fetch.mockResolvedValue({ status: 404 });
  await fetchWithCacheOnError('cache-id', request);
  expect(cache.put).not.toBeCalled();
});
```

最后，我们有一个嵌套的网络错误上下文。注意使用`mockRejectedValue`而不是`mockResolvedValue`，这将抛出异常并导致`catch`块执行：

```js
describe('when fetch returns a network error', () => {
  let cachedResponse = 'cached-response';
  beforeEach(() => {
    fetch.mockRejectedValue({});
    cache.match.mockResolvedValue(cachedResponse);
  });
  it('retrieve the cached value', async () => {
    await fetchWithCacheOnError(
      'cache-id',
      request
    );
    expect(cache.match).toBeCalledWith(request);
  });
  it('returns the cached value', async () => {
    const result = await fetchWithCacheOnError(
      'cache-id',
      request
    );
    expect(result).toEqual(cachedResponse);
  });
});
```

就这样：我们使用 Playwright 和 Vitest 测试的组合，实现了一个完全经过测试的服务工作者。

# 摘要

我们通过观察即使是像服务工作者这样的复杂浏览器功能也可以被测试覆盖完全，完成了这本书的编写。

你已经看到了 Playwright 测试应该如何始终测试实现带来的好处——在这个例子中，检查页面是否可以离线访问——而不是测试实现细节，比如服务工作者是否可用。

你也看到了 Vitest 测试如何通过将大部分实现推入纯 JavaScript 函数来避免检查尴尬的服务工作者接口。

就这样，我们对测试驱动 Svelte 的探索告一段落。现在轮到你了，将你所学的应用到实践中。

正如这本书所展示的，你的测试实践有很多途径可以遵循。我鼓励你进行实验，找到适合你的方法。寻找那些使你的生活更轻松并允许你以稳定的速度构建更高品质软件的实践。

感谢你选择花时间阅读这本书。如果你有任何反馈，无论是好是坏，我都非常乐意听到。你可以通过书的 GitHub 仓库或通过我的网站[www.danielirvine.com](http://www.danielirvine.com)联系我。
