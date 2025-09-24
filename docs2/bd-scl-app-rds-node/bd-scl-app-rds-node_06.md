# 第七章

# API 缓存

# 简介

用户打开浏览器并尝试访问应用程序的不同部分。这会导致对后端进行许多 API 调用。通常，用户数量较少时，响应会很快。然而，当应用程序中的数据增长并且大量用户同时访问数据时，响应时间会增加。这可能会导致用户体验不佳。

通常，对于一个系统，当数据的状态不经常变化时，对于给定的输入，输出将大部分相同，除非数据发生变化。在数据没有变化之前，如果你可以在某处保留结果的副本，那么就不需要重复从数据库中获取相同的数据。将此结果存储起来并在需要时访问的过程是缓存的基本概念。

在本章中，我们将学习如何缓存数据，并使用缓存的数据来服务 API 或保存查询数据，这样我们就不必在一定时间内或直到数据发生变化时查询数据库。

# 结构

在本章中，我们将讨论以下主题：

+   理解缓存

+   Redis 简介

+   设置 Redis 服务器

+   Redis/Caching 的优缺点

+   使用 Redis 缓存数据

# 理解缓存

假设你正在尝试登录应用程序。在输入用户名和密码后，应用程序需要几秒钟来验证并导航到主页。在主页上，有许多内容：你参与的项目列表、项目任务上的团队活动、你的高优先级任务列表等等。如果系统已经忙于处理过多的请求，那么收集每个部分的数据将需要相当多的时间。

主页上的所有这些内容都需要从数据库中获取一些数据。每次你或其他用户打开应用程序时，数据库查询都会花费时间。这些数据库查询结果可以被*缓存*。如果被缓存，那么每次请求主页时，就可以避免数据库查询，直接从缓存中读取数据并发送响应。这种缓存-检索数据的过程会使响应更快，并提高用户体验。

根据定义，缓存是一种将获取的数据或计算结果存储在*缓存*中的技术，以便任何未来请求该数据时可以更快地提供服务。当我们需要访问数据时，首先会检查缓存以查看所需数据是否已缓存。如果是的话，则从缓存中提供数据。如果不是，则获取数据，如果需要则处理它，然后将其存储在缓存中，以便下一次请求可以更快地提供服务。

缓存是系统性能优化的关键组件。它有助于降低响应时间或延迟，并使系统具有高度的可扩展性。缓存可以被视为高速数据存储系统。有许多类型的缓存：内存缓存、磁盘缓存、浏览器缓存、数据库缓存等等。对于我们的用例，我们将缓存 API 响应所需的数据和数据库查询结果。对于缓存，我们将使用名为 Redis 的软件。

# Redis 简介

引用 Redis.io 网站的内容：

*“被数百万开发者用作数据库、缓存、流式引擎和消息代理的开源内存数据存储。”*

简而言之，Redis 可以存储各种类型的数据。其中大多数是简单的键值对。它支持各种数据结构，如字符串、散列、列表、集合、有序集合、位图等。我们可以在 Redis 中存储这些类型的数据并通过键访问。

由于本质上是内存中的，Redis 在读写操作上非常快，因此成为缓存的热门选择。

除了帮助我们存储各种类型数据的核心数据结构外，Redis 还提供了包括数据复制、持久化、分片和事务功能在内的特性。Redis 被开发者用于各种应用。Redis 被 GitHub、Twitter、snapchat、stackoverflow 以及许多其他公司使用。Techstacks.io 维护了一个使用 Redis 进行用例的流行网站列表。

更多关于 Redis 的信息可以在 - [`redis.io`](https://redis.io) 上学习。

# 设置 Redis 服务器

让我们先设置 Redis 服务器。本节将涵盖 MacOS、Ubuntu(debian) 和 Rocky Linux 的安装。

# 在 Mac OS 上安装 Redis 服务器

在 Mac 上安装大多数软件最简单的方法是使用 `**homebrew**`。如果 `**homebrew**` 没有安装，可以通过运行以下命令进行安装：

`/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`

这将使一个名为 `**brew**` 的命令可用，我们可以使用它来安装 Redis 服务器。

`brew install redis`

一旦完成，可以使用以下命令启动服务器：

`brew services start redis`

要验证 Redis 是否已安装，我们可以尝试运行 `**redis-cli**`，这是访问 Redis 的命令行界面。

`redis-cli`

如果 Redis 安装成功，它将打开一个 Redis 提示符，如下所示：

`127.0.0.1:6379>`

如果提示符可见，Redis 已正确安装并准备好使用。

此外，我们可以执行一个 `**ping**` 命令，它应该响应为 `**PONG**`。如果这样做，那么一切正常。

`127.0.0.1:6379> ping`

`PONG`

# 在 Ubuntu / Linux 上安装 Redis 服务器

要在 Ubuntu 上安装 Redis，我们需要 `**lsb-release**`、`**curl**` 和 `**gpg**`。如果这些尚未安装，可以使用 `**apt**` 进行安装。

`sudo apt install lsb-release curl gpg`

现在，我们需要添加一个仓库，其中包含可用的 `**redis**` 二进制文件。

`curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg`

`echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list`

`一旦添加了仓库，我们就可以安装 Redis：`

`sudo apt-get update`

`sudo apt-get install redis`

之后，可以使用以下方式启动服务器：

`sudo systemctl start redis`

服务器启动后，类似于 Mac OS，我们可以使用 `**redis-cli**` 来验证。

# 在 Rocky 上安装 Redis 服务器（基于 RHEL）

我们可以使用 `**dnf**` 软件包管理器简单地安装 Redis。

`sudo dnf update -y`

`sudo dnf install -y redis`

安装完成后，服务可以启动如下：

`sudo systemctl status redis`

最后，我们可以使用 `**redis-cli**` 来验证。

# 缓存的优缺点

在我们继续使用 Redis 来实现我们的目的之前，了解缓存带来的优缺点是很重要的。缓存是一项关键技术，但它带来了一定的优点和缺点，我们在将其应用于应用时应该牢记在心。

# 缓存的优点

缓存的一些主要好处如下：

+   **性能提升**：缓存显著减少了访问数据所需的时间，这使得对最终用户的响应更快。因此，它提供了更好的应用性能和改进的用户体验。

+   **降低负载**：如果正确实施，大部分数据现在都从缓存中提供服务，这减少了应用上的负载。这样，应用可以处理更多的请求并更高效地运行。

+   **成本效益**：有了缓存，单个服务器可以高效地处理更多的请求。考虑一下，如果没有缓存的服务器每分钟可以处理 100 个请求，而有了缓存，它可以每分钟处理 1000 个请求。当用户负载增加时，我们不需要添加更多的服务器。不仅服务器成本降低了，由于负载减少和带宽消耗降低，运营成本也降低了。

+   **减少网络流量**：缓存服务器可以安装到应用所在的本地服务器上，或者安装到另一台服务器上。如果本地安装，它可以减少网络流量。然而，必须看到应用和缓存服务器是否可以保持在同一台机器上，而不会使资源消耗过高，例如内存和 CPU。

# 缓存的缺点

在开发过程中需要注意的一些缺点和要小心的事项如下：

+   **过时数据**：确保缓存一致性很重要。如果数据过时，应该有有效的缓存失效策略来从缓存中删除旧的和无效的数据。考虑这样一个例子，你通过更新电话号码来更新你的个人资料。应用程序已经缓存了用户实体，它将你的个人资料数据保存在缓存中。当缓存的实体有更新时，旧数据必须从缓存中删除，并且新的数据必须放入其中。

    在缓存中存储数据时非常重要的是要小心。在所有数据可能不再有效的地方，应该将其删除或替换为更新后的数据。有时，当数据更新并不总是可能时，可以利用 TTL 这类功能。TTL 代表生存时间。通常，所有缓存系统都提供这一功能。这允许我们设置数据在缓存中应该保留的时间。一旦时间到期，数据将自动失效。

+   **资源消耗**：如果应用程序中有大量数据，重要的是要决定我们缓存什么。只有最常需要的数据才应该被缓存。否则，在资源受限的环境中，内存、磁盘空间和其他资源可能会成为问题。

+   **缓存未命中**：当我们尝试访问某些数据但未在缓存中找到时，这被称为缓存未命中。了解如何处理它很重要。如果数据在缓存中找不到，应用程序必须回退到原始数据源。此外，当从原始源获取数据时，它应该被放入缓存以更快地服务未来的请求。

    处理像缓存未命中这样的情况至关重要，因为它在时间和资源方面可能代价高昂。如果有太多的请求，并且有太多的缓存未命中，这将导致应用程序效率低下。

+   **数据同步问题**：对于分布式环境，保持缓存数据和原始数据源同步可能是一个挑战。如果有多个缓存，这个问题会更大。

+   **复杂性**：总体而言，实现缓存可能会给系统增加更多的复杂性。它也可能导致额外的开发工作以及测试和维护工作。

# 使用 Redis 进行缓存

在我们的项目管理系统中，到目前为止，我们已经实现了几个模块：用户、角色、项目和任务。假设这个应用程序被一个拥有 10 万名用户的组织使用。在办公时间开始时，通常上午 9 点，人们将登录并访问他们的项目，了解他们的任务并取得进展。如果我们缓存一些东西，响应时间将得到改善，用户体验将更好。

缓存有许多策略，例如按需缓存和主动缓存。主动缓存是指在应用程序启动时缓存某些内容，而不需要任何用户请求。这种缓存是在预期未来请求的情况下进行的。另一方面，每当访问某个内容并且是缓存未命中情况时，此时的缓存就是按需的。在按需缓存的情况下，最初不会有任何记录，只有在缓存未命中后才会缓存对象。

在我们的应用程序中，我们可以混合使用这两种策略。当应用程序启动时，我们可以缓存所有用户、角色、项目及其任务。可能会有一些特定的情况，当我们想要缓存的数据不是直接的数据库查询结果时，例如，如果你想要所有项目的项目计数和相应地所有项目的任务计数。这些值将是某些函数的结果，我们可以将其缓存。在本节中，我们将看到如何缓存我们感兴趣缓存的两类数据。

总体来说，我们希望以以下方式存储数据：

+   **对象**：本质上是指项目的、任务的、用户的等 JSON 对象。

+   **数字**：项目、任务、用户等的计数。

可能还有更多。

# 更新项目依赖

首先，我们需要更新项目依赖，即我们案例中的节点模块。我们只需要一个包，那就是 Redis。让我们使用`**npm**` `**install**`来安装 Redis：

`npm install redis`

在开发过程中，安装 Redis 的类型定义会更好，这样我们就能得到适当的代码提示。

`npm install --save-dev @types/redis`

当 Redis 包可用后，我们可以像以下示例那样使用它：

`import { createClient, RedisClient } from 'redis';`

`const client: RedisClient = createClient();`

`client.on('connect', () => {`

`console.log('Connected to Redis');`

`});`

`client.set('key', 'value', (err, reply) => {`

`if (err) throw err;`

`console.log(reply); // OK`

`});`

`client.get('key', (err, reply) => {`

`if (err) throw err;`

`console.log(reply); // value`

`});`

`client.quit();`

在前面的例子中，我们导入了`**createClient**`函数和`**RedisClient**`类型。接下来，我们使用`**createClient()**`函数调用创建了一个客户端，并使用`**client.on()**`函数尝试连接到 Redis 服务器。

在讨论的例子中，我们使用了回调，但我们也可以使用 async-await。

使用`**client.set()**`，我们可以设置一个键值对。这将把键值对存储到作为缓存服务器的 Redis 服务器上。当需要时，我们可以使用`**client.get()**`获取键的值。

在这里，我们存储了一个简单的字符串 `**'value'**`，如果我们想存储一个对象，我们必须使用 `**JSON.stringify()**` 将对象转换为字符串，并将其作为字符串存储。所有这些都是因为 Redis 不直接支持 JSON 对象作为值。为了使 Redis 存储 JSON 对象，我们需要设置一个名为 `**RedisJSON**` 的 Redis 模块。此模块为 Redis 提供了本地的 JSON 功能。

`**RedisJSON**` 模块可在 GitHub 上找到：`**https://github.com/RedisJSON/RedisJSON**`。

我们需要克隆此仓库或下载仓库的 zip 文件。在继续之前，请确保已安装 Rust。

如果不可用，可以使用以下方式安装 rust：

`curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh`

此步骤完成后，我们可以通过检查 rust 版本来验证安装。

`rustc --version`

安装完成后，我们可以开始构建 `**RedisJSON**`。导航到下载的仓库并运行以下命令：

`cargo build –release`

这将在 `**target/release/librejson.so**` 中创建模块。如果文件缺失，则表示出了问题，请再次尝试构建。

我们需要修改 Redis 配置，通常位于 `**/etc/redis/redis.conf**` 或 `**/usr/local/etc/redis.conf**`，并添加对 `**redisjson.so**` 模块的启用。

**在 Linux 上**

`loadmodule /path/to/redisjson.so`

**在 Mac 上**

`loadmodule /path/to/rejson.dylib`

模块加载后，重新启动 Redis 以启用它。

# 缓存工具

由于我们需要缓存大量实体，使用缓存工具类可能是有益的。这个类理想情况下应该提供设置和检索缓存值的函数。

一旦我们有了缓存工具类，我们就可以直接在需要的地方导入它并使用其功能。让我们创建一个名为 `**cache_util.ts**` 的文件，其中包含以下代码：

`// cache_util.ts`

`import * as redis from 'redis';`

`export class CacheUtil {`

`// redis client instance`

`private static client = redis.createClient();`

`constructor() {`

`CacheUtil.client.connect();`

`}`

`public static async get(cacheName: string, key: string) {`

`try {`

``const data = await CacheUtil.client.json.get(`${cacheName}:${key}`);``

`return data;`

`} catch (err) {`

``console.error(`Error getting cache: ${err}`);``

`return null;`

`}`

`}`

`public static async set(cacheName: string, key: string, value) {`

`try {`

``await CacheUtil.client.json.set(`${cacheName}:${key}`, '.', value);``

`} catch (err) {`

``console.error(`Error setting cache: ${err}`);``

`}`

`}`

`public static async remove(cacheName: string, key: string) {`

`try {`

``await CacheUtil.client.del(`${cacheName}:${key}`);``

`} catch (err) {`

``console.error(`Error deleting cache: ${err}`);``

`}`

`}`

`}`

函数 `**set()**` 可以用于在缓存中设置键值对，而 `**get()**` 函数可以用于检索值。注意我们是如何使用 `**client.json.get()**` 和 `**client.json.set()**` 实际上获取和设置值的。这两个函数都被设置为静态的，这样我们就可以直接使用它们，而无需每次都初始化类。

我们现在可以初始化类一次并在任何地方使用它。我们需要这样做是为了将 Redis 客户端连接到服务器。如果没有调用 `**CacheUtil.client.connect()**`，应用程序将抛出错误：客户端已关闭。

现在，当缓存工具准备就绪时，让我们在 `**main.ts**` 中初始化它。

`// main.ts`

`// 初始化缓存工具`

`new CacheUtil();`

这将调用构造函数并将客户端连接到 Redis 服务器。

# 缓存实体

我们之前讨论了两种实体缓存方法：按需和主动。对于按需缓存，我们可以更新所有控制器的函数，首先检查所需数据是否在缓存中。以下示例显示了 `**users_controller**` 的 `**getOneHandler**`，它负责根据给定的用户 ID 返回用户：

`// users_controller.ts`

`public async getOneHandler(req: Request, res: Response): Promise<void> {`

`if (!hasPermission(req.user.rights, 'get_details_user')) {`

`res.status(403).json({ statusCode: 403, status: 'error', message: 'Unauthorised' });`

`return;`

`}`

`// 检查用户是否在缓存中`

`**const userFromCache = await CacheUtil.get(**'**User**'**, req.params.id);**`

`if (userFromCache) {`

`res.status(200).json({ statusCode: 200, status: 'success', data: userFromCache });`

`return;`

`} else {`

`// 从数据库获取用户`

`const service = new UsersService();`

`const result = await service.findOne(req.params.id);`

`if (result.statusCode === 200) {`

`delete result.data.password;`

`// 在缓存中设置用户`

`**CacheUtil.set(**'**User**'**, req.params.id, result.data);**`

`}`

`res.status(result.statusCode).json(result);`

`return;`

`}`

`}`

在该函数中，我们调用 `**CacheUtil.get()**` 以 `**'User'**` 作为缓存名称，键是请求参数中提供的 `**user_id**`。有可能请求的用户不在缓存中，因此必须检查返回的值 `**userFromCache**` 是否为 null。如果是 null，则应采取常规流程并从数据库中获取用户。如果用户存在于数据库中，则还应将其保存到缓存中。以下调用 `**CacheUtil.set()**` 的行为我们做了这件事：

`**CacheUtil.set('User', req.params.id, result.data);**`

这样，所有函数都可以更新为在实际数据库查询之前检查缓存。对于删除 API 被调用的案例，缓存中的值也必须被删除。如果值没有被删除，未来通过`**user_id**`获取用户的 API 调用将返回一个在数据库中不再存在的值。

`The `**delete**` function can be updated as:

`public async deleteHandler(req: Request, res: Response): Promise<void> {`

`if (!hasPermission(req.user.rights, 'delete_user')) {`

`res.status(403).json({ statusCode: 403, status: 'error', message: 'Unauthorised' });`

`return;`

`}`

`const service = new UsersService();`

`const result = await service.delete(req.params.id);`

`// remove user from cache`

`**CacheUtil.remove(**'**User**'**, req.params.id);**`

`res.status(result.statusCode).json(result);`

`return;`

`}`

调用`**CacheUtil.remove()**`将从缓存中删除用户。

使用这种按需方法进行缓存，确保常用数据保持在缓存中，而未使用的数据不会填满缓存。然而，在这种情况下，当数据第一次被请求时，总会发生缓存未命中。

# Building Cache at Startup

有时，在应用程序启动时填充缓存是好的。这有助于防止一些缓存未命中，因为请求的数据将存在于缓存中。让我们更新`**UsersUtil**`以添加一个函数`**putAllUsersInCache()**`，该函数将从数据库中检索所有用户并将它们放入缓存。

`// function to put all users in cache`

`public static async putAllUsersInCache() {`

`const userService = new UsersService();`

`const result = await userService.findAll({});`

`if (result.statusCode === 200) {`

`const users = result.data;`

`users.forEach(i => {`

`CacheUtil.set('User', i.user_id, i);`

`});`

``console.log(`All users are put in cache`);``

`}else{`

``console.log(`Error while putAllUsersInCache() => ${result.message}`);``

`console.log(result);`

`}`

`}`

此函数正在使用`**userService**`调用`**findAll()**`来从数据库中获取所有用户，如果查询结果成功，则使用`**forEach**`将用户放入缓存。

我们可以从`**main.ts**`调用此函数。

`// Proactive cache update`

`setTimeout(() => {`

`UsersUtil.putAllUsersInCache();`

`}, 1000 * 10 );`

当一个应用程序启动时，它可能需要时间来建立与数据库、缓存服务器的连接，并执行其他初始化操作。在开始其他操作之前，为这些连接和初始化操作留出足够的时间是明智的，以确保应用程序的平稳运行。因此，我们使用`**setTimeout**`添加了一些延迟（这里为 10 秒）。

类似于`**UsersUtil**`，其他实体工具也可以修改以添加一个功能，然后我们可以从`**main.ts**`中调用这些功能。这些功能不一定要从`**main.ts**`文件中调用。我们可以将它们放在其他地方，例如`**CacheUtil**`，然后从`**main.ts**`中调用`**CacheUtil**`来`**init**`缓存。同样，它可以是`**putAllXXToCache**`之外的另一个功能。

# 使用 Redis 时的注意事项

尽管 Redis 是一个强大的内存数据存储，但在使用 Redis 时仍有一些挑战。以下是一些挑战的讨论点：

+   **内存限制**

    由于它是一个内存数据存储，它提供了更快的访问速度，但受限于系统容量。如果您有一个大数据集，目标是尽量减少成本，那么这可能会是一个缺点。

+   **数据安全**

    默认情况下，Redis 未加密。尽管它支持基于密码的简单认证，但没有内置的 SSL/TLS 加密支持。为了保护 REST API 的数据，需要额外的工具。

+   **查询数据的方式有限**

    与传统的数据库系统相比，Redis 的查询能力有限。然而，通过在应用层实现一些额外的逻辑，可以执行复杂的查询。

+   **单线程特性**

    Redis 服务器本质上是单线程的。如今的大多数机器都是多核的，Redis 无法利用超过一个核心。然而，较新的 Redis 版本将慢查询放到单独的线程中，但 Redis 服务器的请求仍然只由一个线程处理。

+   **云上的托管成本**

所有主要云平台都提供 Redis 作为服务。例如，AWS 的`**Elasticache**`、Azure 的 Redis 缓存都是一些例子。对于大数据集，这些服务可能很昂贵。

如果 Redis 不适合您的用例，还可以探索其他选项，如 Memcached、Apache Kafka 等。

# 结论

缓存可以提高用户体验，极大地提升应用程序的性能，同时使其更加稳定和可扩展。在本章中，我们熟悉了缓存的概念，并设置了 Redis，它是任何规模应用程序缓存的流行选择。我们学习和实现了两种缓存策略：按需缓存和主动缓存。

在下一章中，我们将实现通知模块，同时学习 Redis 的另一个方面：消息队列。

# 多项选择题

1.  实现应用程序中缓存的 主要目标是什么？

    1.  为了增加数据处理时间。

    1.  为了安全地存储用户密码。

    1.  为了减少访问数据所需的时间。

    1.  为了增加网络流量。

1.  缓存处理数据库查询的显著好处是什么？

    1.  它增加了应用服务器的负载。

    1.  它通过从缓存中获取数据来帮助避免数据库查询。

    1.  它降低了应用程序的性能。

    1.  它消耗更多的带宽。

1.  以下哪项不是缓存的优点？

    1.  性能提升。

    1.  减轻应用程序的负载。

    1.  增加网络流量。

    1.  成本效益。

1.  在缓存的环境中，TTL 代表什么？

    1.  启动时间

    1.  存活时间

    1.  总时间限制

    1.  加载时间

1.  什么是“*缓存未命中*”？

    1.  当数据成功从缓存中检索时。

    1.  当数据在缓存中未找到时。

    1.  当缓存被完全利用时。

    1.  当缓存无法保存数据时。

1.  在分布式环境中缓存的关键挑战是什么？

    1.  简化用户界面。

    1.  减少用户数量。

    1.  保持缓存数据和原始数据源同步。

    1.  降低服务器成本。

1.  在启动时构建缓存的优点是什么？

    1.  它不必要地增加了缓存大小。

    1.  它确保频繁请求的数据立即可用。

    1.  它会减慢应用程序的启动速度。

    1.  它需要较少的开发工作量。

1.  在描述的场景中，哪种软件用于缓存？

    1.  MySQL

    1.  Redis

    1.  MongoDB

    1.  Oracle

# 答案

1.  c

1.  b

1.  c

1.  b

1.  b

1.  c

1.  b

1.  b

# 进一步阅读

[`www.geeksforgeeks.org/caching-system-design-concept-for-beginners/`](https://www.geeksforgeeks.org/caching-system-design-concept-for-beginners/)

[`redis.io`](https://redis.io)

[`redis.io/docs/data-types/json/`](https://redis.io/docs/data-types/json/)

[`www.npmjs.com/package/redis`](https://www.npmjs.com/package/redis)
