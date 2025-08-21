# 第八章：测试 – 单元和集成

代码在相应的测试编写完成后才会创建。既然你正在阅读这一章，那么我可以假设我们可以同意这个观点。然而，你可能想知道，为什么我们一个测试都没有编写呢？这是可以理解的。

我们选择不这样做，因为我们认为这会让内容更难吸收。由于我们希望你在构建应用程序的同时专注于学习 Deno，所以我们决定不这样做。第二个原因是，我们确实希望有一个完整的章节专注于测试；即这一章。

测试是软件生命周期中的一个非常重要的部分。它可以用来节省时间，明确需求，或者只是因为你希望在以后重新编写和重构时感到自信。无论动机是什么，有一点是肯定的：你会编写测试。我也真心相信测试在软件设计中扮演着重要的角色。容易测试的代码很可能容易维护。

由于我们非常倡导测试的重要性，所以我们不能不学习它就认为这是一本完整的 Deno 指南。

在这一章中，我们将编写不同类型的测试。我们将从单元测试开始，这对于开发者和维护周期来说是非常有价值的测试。然后，我们将进行集成测试，在那里我们将运行应用程序并对其执行几个请求。最后，我们将使用在前一章中编写的客户端。在这个过程中，我们将向之前构建的应用程序添加测试，一步一步地进行，并确保我们之前编写的代码正常工作。

本章还将展示我们在这本书的开始时做出的某些架构决策将如何得到回报。这将是介绍我们如何使用 Deno 及其工具链编写简单的模拟和干净、专注的测试的入门。

在这一章中，我们将介绍以下主题：

+   在 Deno 中编写你的第一个测试

+   编写集成测试

+   测试网络服务器

+   为应用程序创建集成测试

+   一起测试 API 和客户端

+   基准测试应用程序的部分

让我们开始吧！

## 技术要求

本章中将使用的代码可以在 [`github.com/PacktPublishing/Deno-Web-Development/tree/master/Chapter08/sections`](https://github.com/PacktPublishing/Deno-Web-Development/tree/master/Chapter08/sections) 找到。

# 在 Deno 中编写你的第一个测试

在我们开始编写测试之前，记住一些事情是很重要的。其中最重要的原因是，我们为什么要测试？

对于这个问题，可能会有多个答案，但大多数都会指向保证代码正在运行。你也可能说，你使用它们以便在重构时具有灵活性，或者你重视在实施时拥有短暂的反馈周期——我们可以同意这两点。由于我们在实现这些功能之前没有编写测试，所以后者对我们来说并不适用。

在本章中，我们将保持这些目标。在本节中，我们将编写我们的第一个测试。我们将使用在前几章中编写的应用程序并为其添加测试。我们将编写两种类型的测试：集成和单元测试。

集成测试将测试应用程序不同组件之间的交互。单元测试测试隔离的层。如果我们把它看作是一个光谱，那么单元测试更接近代码，而集成测试更接近用户。在用户端的尽头，还有端到端测试。这些测试通过模拟用户行为来测试应用程序，我们将在本章不涉及这些内容。

我们在开发实际应用程序时使用的部分模式，如依赖注入和控制反转，在测试时非常有用。由于我们的代码通过注入其所有依赖关系来开发，现在，只需在测试中模拟这些依赖关系即可。记住：易于测试的代码通常也易于维护。

我们首先要做的是为业务逻辑编写测试。目前，由于我们的 API 相当简单，所以它没有太多的业务逻辑。大部分都存在于`UserController`中，因为`MuseumController`非常简单。我们从后者开始。

为了在 Deno 中编写测试，我们需要使用以下内容：

+   在第二章，*工具链*中介绍的 Deno 测试运行器

+   来自 Deno 命名空间的`test`方法([`doc.deno.land/builtin/stable#Deno.test`](https://doc.deno.land/builtin/stable#Deno.test))

+   来自 Deno 标准库的断言方法(`doc.deno.land/https/deno.land/std@0.83.0/testing/asserts.ts`)

这些都是 Deno 的组成部分，由核心团队分发和维护。社区中还有许多其他可以在测试中使用的库。我们将使用 Deno 中提供的默认设置，因为它工作得很好，并允许我们编写清晰易读的测试。

让我们去学习我们如何定义一个测试！

## 定义测试

Deno 提供了一个定义测试的 API。这个 API，`Deno.test` ([`doc.deno.land/builtin/stable#Deno.test`](https://doc.deno.land/builtin/stable#Deno.test))，提供了两种不同的定义测试的方法。

其中一个是我们在第二章*中所展示的*，*工具链*，由两部分组成；也就是说，测试名称和测试函数。这可以在以下示例中看到：

```js
Deno.test("my first test", () => {})
```

我们可以这样做另一种方式是调用相同的 API，这次发送一个对象作为参数。 你可以发送函数和测试名称，以及几个其他选项，到这个对象，如你在以下示例中所见：

```js
Deno.test({
  name: "my-second-test",
  fn: () => {},
  only: false,
  sanitizeOps: true,
  sanitizeResources: true,
});
```

这些标志行为在文档中解释得非常清楚([`doc.deno.land/builtin/stable#Deno.test`](https://doc.deno.land/builtin/stable#Deno.test))，但这里有一个总结供您参考：

+   `only`：只运行设置为`true`的测试，并使测试套件失败，因此这应该只用作临时措施。

+   `sanitizeOps`：如果 Deno 的核心启动的所有操作都不成功，则测试失败。这个标志默认是`true`。

+   `sanitizeResources`：如果测试结束后仍有资源在运行，则测试失败（这可能表明内存泄漏）。这个标志确保测试必须有一个清理阶段，其中资源被停止，默认情况下是`true`。

既然我们知道了 API，那就让我们去编写我们的第一个测试——对`MuseumController`函数的单元测试。

## 对 MuseumController 的单元测试

在本节中，我们将编写一个非常简单的测试，它将只涵盖我们在`MuseumController`中编写的功能，不多不少。

它列出了应用程序中的所有博物馆，尽管目前它还没有做什么，只是作为`MuseumRepository`的代理工作。我们可以通过以下步骤创建这个简单功能的测试文件和逻辑：

1.  创建`src/museums/controller.test.ts`文件。

    测试运行器将自动将名称中包含`.test`的文件视为测试文件，以及其他在第二章《工具链》中解释的约定，*章节目录*.

1.  使用`Deno.test`([`doc.deno.land/builtin/stable#Deno.test`](https://doc.deno.land/builtin/stable#Deno.test))声明第一个测试：

    ```js
    Deno.test("it lists all the museums", async () => {});
    ```

1.  现在，从标准库中导出断言方法，并将其命名空间命名为`t`，这样我们就可以在测试文件中使用它们，通过在`src/deps.ts`中添加以下内容：

    ```js
    export * as t from
      "https://deno.land/std@0.83.0/testing/asserts.ts";
    ```

    如果您想了解标准库中可用的断言方法，请查看`doc.deno.land/https/deno.land/std@0.83.0/testing/asserts.ts`。

1.  现在，您可以使用标准库中的断言方法来编写一个测试，该测试实例化`MuseumController`并调用`getAll`方法：

    ```js
    import { t } from "../deps.ts";
    import { Controller } from "./controller.ts";
    Deno.test("it lists all the museums", async () => {
      const controller = new Controller({
    MuseumController and sending in a mocked version of museumRepository, which returns a static array. This is how we're sure we're testing only the logic inside MuseumController, and nothing more. Closer to the end of the snippet, we're making sure the getAll method's result is returning the museum being returned by the mocked repository. We are doing this by using the assertion methods we exported from the dependencies file.
    ```

1.  让我们运行测试并验证它是否正常工作：

    ```js
    $ deno test --unstable --allow-plugin --allow-env --allow-read –-allow-write --allow-net src/museums
    running 1 tests
    test it lists all the museums ... ok (1ms)
    test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out (1ms)
    ```

我们的第一个测试成功了！

注意测试输出如何列出测试的名称、状态以及运行所需的时间，同时还包括测试运行的摘要。

`MuseumController`内部的逻辑相当简单，因此这也一个非常简单的测试。然而，它隔离了控制器的行为，允许我们编写一个非常专注的测试。如果您对为应用程序的其他部分创建单元测试感兴趣，它们可以在本书的存储库中找到([`github.com/PacktPublishing/Deno-Web-Development/tree/master/Chapter08/sections/7-final-tested-version/museums-api`](https://github.com/PacktPublishing/Deno-Web-Development/tree/master/Chapter08/sections/7-final-tested-version/museums-api)).

在接下来的几节中，我们将编写更多有趣的测试。这些测试将教会我们如何检查应用程序不同模块之间的集成。

# 编写集成测试

我们在上一节创建的第一个单元测试依赖于仓库的模拟实例，以保证我们的控制器正在工作。这个测试在检测`MuseumController`中的错误时增加了很大的价值，但它在了解控制器是否与仓库良好工作时并不重要。

这就是集成测试的目的：它们测试多个组件如何相互集成。

在本节中，我们将编写一个集成测试，用于测试`MuseumController`和`MuseumRepository`。这些测试将紧密模仿应用程序运行时发生的情况，并有助于我们后来在检测这两个类之间的任何问题时提供帮助。

让我们开始：

1.  在`src/museums`中为这个模块的集成测试创建一个文件，称为`museums.test.ts`，并在其中添加第一个测试用例。

    它应该测试是否可以获取所有博物馆，这次使用仓库的实例而不是模拟的一个：

    ```js
    Deno.test("it is able to get all the museums from
      storage", async () => {});
    ```

1.  我们将首先实例化仓库并在其中添加几个测试用例：

    ```js
    import { t } from "../deps.ts";
    import { Controller, Repository } from "./index.ts";
    Deno.test("it is able to get all the museums from
      storage", async () => {
      const repository = new Repository();
      repository.storage.set("0", {
        description: "museum with id 0",
        name: "my-museum",
        id: "0",
        location: { lat: "123", lng: "321" },
      });
      repository.storage.set("1", {
        description: "museum with id 1",
        name: "my-museum",
        id: "1",
        location: { lat: "123", lng: "321" },
      });
    …
    ```

1.  现在我们已经有了一个仓库，我们可以用它来实例化控制器：

    ```js
    const controller = new Controller({ museumRepository:
      repository });
    ```

1.  现在我们可以编写我们的断言，以确保一切正常工作：

    ```js
    const allMuseums = await controller.getAll();
    t.assertEquals(allMuseums.length, 2);
    t.assertEquals(allMuseums[0].name, "my-museum", "has
      name");
    t.assertEquals(
      allMuseums[0].description,
      "museum with id 0",
      "has description",
    );
    t.assertEquals(allMuseums[0].id, "0", "has id");
    t.assertEquals(allMuseums[0].location.lat, "123", "has
      latitude");
    t.assertEquals(allMuseums[0].location.lng, "321", assertEquals, allowing us to get a proper message when this assertion fails. This is something that all assertion methods support.
    ```

1.  让我们运行测试并查看结果：

    ```js
    $ deno test --unstable --allow-plugin --allow-env --allow-read –-allow-write --allow-net src/museums
    running 2 tests
    test it lists all the museums ... ok (1ms)
    test it is able to get all the museums from storage ... ok (1ms)
    test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out (2ms)
    ```

它通过了！这就是我们需要的仓库和控制器集成测试的全部！当我们要更改`MuseumController`或`MuseumRepository`中的代码时，这个测试很有用，因为它确保它们在一起工作时没有问题。

如果你对应用程序其他部分的集成测试如何工作感到好奇，我们在这本书的仓库中提供了它们([`github.com/PacktPublishing/Deno-Web-Development/tree/master/Chapter08/sections/7-final-tested-version/museums-api`](https://github.com/PacktPublishing/Deno-Web-Development/tree/master/Chapter08/sections/7-final-tested-version/museums-api)).

在第一部分，我们创建了一个单元测试，在这里，我们创建了一个集成测试，但我们还没有为应用程序的界面编写任何测试——Web 部分，它使用 HTTP。那就是我们下一节要做的。我们将学习如何孤立地测试 Web 层中的逻辑，不使用任何其他模块。

# 测试 Web 服务器

到目前为止，我们已经学习了如何测试应用程序的不同部分。我们始于业务逻辑，它测试如何与与持久性（仓库）交互的模块集成，但 Web 层仍然没有测试。

确实，那些测试非常重要，但我们可以说，如果 Web 层失败，用户将无法访问任何逻辑。

这就是我们将在本节中做的事情。我们将启动我们的 web 服务器，模拟其依赖项，并向其发送几个请求以确保 web*单元*正在工作。

让我们通过以下步骤创建 web 模块的单元测试：

1.  前往`src/web`，并创建一个名为`web.test.ts`的文件。

1.  现在，为了测试 web 服务器，我们需要回到`src/web/index.ts`中的`createServer`函数，并导出它创建的`Application`对象：

    ```js
    const app = new Application();
    …
    return { app };
    ```

1.  我们还希望能够在任何时候停止应用程序。我们还没有实现这一点。

    如果我们查看 oak 的文档，我们会看到它非常完善([`github.com/oakserver/oak#closing-the-server`](https://github.com/oakserver/oak#closing-the-server))。

    要取消由`listen`方法启动的应用程序，我们还需要返回`AbortController`。所以，让我们在`createServer`函数的最后这样做。

    如果你不知道`AbortController`是什么，我将留下一个来自 Mozilla 开发者网络的链接([`developer.mozilla.org/en-US/docs/Web/API/AbortController`](https://developer.mozilla.org/en-US/docs/Web/API/AbortController)),它解释得非常清楚。简而言之，它允许我们取消一个进行中的承诺：

    ```js
    const app = new Application();
    …
    const controller = new AbortController();
    const { signal } = controller;
    …
    return { app, controller };
    ```

    注意我们是如何实例化`AbortController`的，与文档中的示例类似，并在最后返回它，以及`app`变量。

1.  回到我们的测试中，让我们创建一个测试，以检查服务器是否响应`hello world`：

    ```js
    Deno.test("it responds to hello world", async () => {})
    ```

1.  让我们用之前创建的函数来启动服务器的实例；也就是说，`createServer`。记住，要调用这个函数，我们必须发送它的依赖项。在这里，我们需要模拟它们：

    ```js
    import { Controller as UserController } from
      "../users/index.ts";
    import { Controller as MuseumController } from
      "../museums/index.ts";
    import { createServer } from "./index.ts";
    …
    const server = await createServer({
      configuration: {
        allowedOrigins: [],
        authorization: {
          algorithm: "HS256",
          key: "abcd",
        },
        certFile: "",
        keyFile: "",
        port: 9001,
        secure: false,
      },
    9001 and with HTTPS disabled, along with some random algorithm and key.Note how we're using TypeScript's `as` keyword to pass mocked types into the `createServer` function without TypeScript warning us about the type.
    ```

1.  现在我们可以创建一个测试，通过响应 hello world 请求来检查 web 服务器是否正常工作：

    ```js
    import { t } from "../deps.ts";
    …
    const response = await fetch(
      "http://localhost:9001/",
      {
        method: "GET",
      },
    ).then((r) => r.text());
    t.assertEquals(
      response,
      "Hello World!",
      "responds with hello world",
    );
    ```

1.  我们需要做的最后一件事是在测试运行后关闭服务器。Deno 默认让我们测试失败，如果我们不做这件事（因为`sanitizeResources`默认是`true`），这可能会导致内存泄漏：

    ```js
      server.controller.abort();
    ```

这标志着我们 web 层的第一个测试结束！这是一个单元测试，它测试了启动服务器的逻辑，并确保 Hello World 运行正常。接下来，我们将为端点编写更完整的测试，包括业务逻辑。

在下一节中，我们将开始为登录和注册功能编写集成测试。这些测试比我们为博物馆模块编写的测试要复杂一些，因为它们将测试整个应用程序，包括其业务逻辑、持久性和 web 逻辑。

# 为应用程序创建集成测试

我们迄今为止编写的三个测试都是针对单一模块的单元测试以及两个不同模块之间的集成测试。然而，为了确信我们的代码正在工作，如果我们可以测试整个应用程序的话，那将会很酷。那就是我们在这里要做的。我们将用测试配置设置我们的应用程序，并对它运行一些测试。

我们首先调用用于初始化 Web 服务器的同一个函数，然后创建所有其依赖项（控制器、存储库等）的实例。我们会确保使用诸如内存持久化之类的东西来做到这一点。这将确保我们的测试是可复制的，并且不需要复杂的拆卸阶段或连接到真实数据库，因为这将减慢测试速度。

我们将从创建一个测试文件开始，这个文件现在将包含整个应用程序的集成测试。随着应用程序的发展，可能很有必要在每个模块内部创建一个测试文件夹，但现在，这个解决方案将完全没问题。

我们将使用与生产环境中运行的非常接近的设置实例化应用程序，并对它进行一些请求和断言：

1.  创建`src/index.test.ts`文件，与`src/index.ts`文件并列。在它里面，创建一个测试声明，测试用户是否可以登录：

    ```js
    Deno.test("it returns user and token when user logs
      in", async () => {})
    ```

1.  在我们开始编写这个测试之前，我们将创建一个帮助函数，该函数将为测试设置 Web 服务器。它将包含实例化控制器和存储库的所有逻辑，以及向应用程序发送配置。它看起来像这样：

    ```js
    import { CreateServerDependencies } from
      "./web/index.ts";
    …
    function createTestServer(options?: CreateServerDependencies) {
      const museumRepository = new MuseumRepository();
      const museumController = new MuseumController({
        museumRepository });
      const authConfiguration = {
        algorithm: "HS256" as Algorithm,
        key: "abcd",
        tokenExpirationInSeconds: 120,
      };
      const userRepository = new UserRepository();
      const userController = new UserController(
        {
          userRepository,
          authRepository: new AuthRepository({
            configuration: authConfiguration,
          }),
        },
      );
      return createServer({
        configuration: {
          allowedOrigins: [],
          authorization: {
            algorithm: "HS256",
            key: "abcd",
          },
          certFile: "abcd",
          keyFile: "abcd",
          port: 9001,
          secure: false,
        },
        museum: museumController,
        user: userController,
        ...options,
      });
    }
    ```

    我们在这里所做的是非常类似于我们在`src/index.ts`中做的布线逻辑。唯一的区别是，我们将显式导入内存存储库，而不是 MongoDB 存储库，如下面的代码块所示：

    ```js
    import {
      Controller as MuseumController,
      InMemoryRepository as MuseumRepository,
    } from "./museums/index.ts";
    import {
      Controller as UserController,
      InMemoryRepository as UserRepository,
    } from "./users/index.ts";
    ```

    为了让我们能够访问`Museums`和`Users`模块的内存存储库，我们需要进入这些模块并将它们导出。

    这就是`src/users/index.ts`文件应该看起来像的样子：

    ```js
    export { Repository } from "./repository/mongoDb.ts";
    Repository but also exporting InMemoryRepository at the same time.Now that we have a way to create a test server instance, we can go back to writing our tests.
    ```

1.  使用我们刚刚创建的帮助函数`createTestServer`创建一个服务器实例，并使用`fetch`向 API 发送注册请求：

    ```js
    Deno.test("it returns user and token when user logs
      in", async () => {
      const jsonHeaders = new Headers();
      jsonHeaders.set("content-type", "application/json");
      const server = await createTestServer();
      // Registering a user
      const { user: registeredUser } = await fetch(
        "http://localhost:9001/api/users/register",
        {
          method: "POST",
          headers: jsonHeaders,
          body: JSON.stringify({
            username: "asantos00",
            password: "abcd",
          }),
        },
      ).then((r) => r.json())
    …
    ```

1.  由于我们可以访问注册的用户，我们可以尝试使用同一个用户登录：

    ```js
      // Login in with the createdUser
      const response = await
        fetch("http://localhost:9001/api/login", {
          method: "POST",
          headers: jsonHeaders,
          body: JSON.stringify({
          username: registeredUser.username,
          password: "abcd",
        }),
      }).then((r) => r.json())
    ```

1.  我们现在准备开发一些断言来检查我们的登录响应是否是我们预期的那样：

    ```js
      t.assertEquals(response.user.username, "asantos00",
        "returns username");
      t.assert(!!response.user.createdAt, "has createdAt
        date");
      t.assert(!!response.token, "has token");
    ```

1.  最后，我们需要在我们的服务器上调用`abort`函数：

    ```js
    server.controller.abort();
    ```

这是我们第一次进行应用程序集成测试！我们让应用程序运行起来，对它执行注册和登录请求，并断言一切按预期进行。在这里，我们逐步构建了测试，但如果你想要查看完整的测试，它可在本书的 GitHub 仓库中找到（[`github.com/PacktPublishing/Deno-Web-Development/blob/master/Chapter08/sections/7-final-tested-version/museums-api/src/index.test.ts`](https://github.com/PacktPublishing/Deno-Web-Development/blob/master/Chapter08/sections/7-final-tested-version/museums-api/src/index.test.ts)）。

为了结束本节，我们将再写一个测试。还记得在前一章节中，我们创建了一些授权逻辑，只允许已登录的用户访问博物馆列表吗？让我们用另一个测试来检查这个逻辑是否生效：

1.  在`src/index.test.ts`中创建另一个测试，用于检测带有有效令牌的用户是否可以访问博物馆列表：

    ```js
    Deno.test("it should let users with a valid token
      access the museums list", async () => {})
    ```

1.  由于我们想要再次登录和注册，我们将提取这些功能到一个我们可以用于多个测试的实用函数中：

    ```js
    function register(username: string, password: string) {
      const jsonHeaders = new Headers();
      jsonHeaders.set("content-type", "application/json");
      return
       fetch("http://localhost:9001/api/users/register", {
         method: "POST",
         headers: jsonHeaders,
         body: JSON.stringify({
          username,
          password,
        }),
      }).then((r) => r.json());
    }
    function login(username: string, password: string) {
      const jsonHeaders = new Headers();
      jsonHeaders.set("content-type", "application/json");
      return fetch("http://localhost:9001/api/login", {
        method: "POST",
        headers: jsonHeaders,
        body: JSON.stringify({
          username,
          password,
        }),
      }).then((r) => r.json());
    }
    ```

1.  有了这些函数，我们现在可以重构之前的测试，使其看起来更简洁，如下面的代码段所示：

    ```js
    Deno.test("it returns user and token when user logs
      in", async () => {
      const jsonHeaders = new Headers();
      jsonHeaders.set("content-type", "application/json");
      const server = await createTestServer();
      // Registering a user
      await register("test-user", "test-password");
      const response = await login("test-user", "test-
      password");
      // Login with the created user
      t.assertEquals(response.user.username, "test-user",
        "returns username");
      t.assert(!!response.user.createdAt, "has createdAt
        date");
      t.assert(!!response.token, "has token");
      server.controller.abort();
    });
    ```

1.  让我们回到我们正在编写的测试——那个检查已认证用户是否可以访问博物馆的测试——并使用`register`和`login`函数来注册和认证一个用户：

    ```js
    Deno.test("it should let users with a valid token
      access the museums list", async () => {
      const jsonHeaders = new Headers();
      jsonHeaders.set("content-type", "application/json");
      const server = await createTestServer();
      // Registering a user
      await register("test-user", "test-password");
      const { token } = await login("test-user", "test-
        password");
    ```

1.  现在，我们可以使用`login`函数返回的令牌，在`Authorization`头中进行认证请求：

    ```js
      const authenticatedHeaders = new Headers();
      authenticatedHeaders.set("content-type",
        "application/json");
      login function and sending it with the Authorization header in the request to the museums route. Then, we're checking if the API responds correctly to the request with the 200 OK status code. In this case, since our application doesn't have any museums, it is returning an empty array, which we're also asserting.Since we're testing this authorization feature, we can also test that a user with no token or an invalid token can't access this same route. Let's do it.
    ```

1.  创建一个测试，检查用户是否可以在没有有效令牌的情况下访问`museums`路由。它应该与之前的测试非常相似，只是我们现在发送一个无效的令牌：

    ```js
    Deno.test("it should respond with a 401 to a user with
      an invalid token", async () => {
      const server = await createTestServer();
      const authenticatedHeaders = new Headers();
      authenticatedHeaders.set("content-type",
        "application/json");
    authenticatedHeaders.set("authorization", 
       `Bearer invalid-token`);
      const response = await
        fetch("http://localhost:9001/api/museums", {
          headers: authenticatedHeaders,
          body: JSON.stringify({
          username: "test-user",
          password: "test-password",
        }),
      });
      t.assertEquals(response.status, 401);
      t.assertEquals(await response.text(),
       "Authentication failed");
      server.controller.abort();
    });
    ```

1.  现在，我们可以运行所有测试并确认它们都通过了：

    ```js
    $ deno test --unstable --allow-plugin --allow-env --allow-read –-allow-write --allow-net src/index.test.ts      
    running 3 tests
    test it returns user and token when user logs in ... Application running at http://localhost:9001
    POST http://localhost:9001/api/users/register - 3ms
    POST http://localhost:9001/api/login - 3ms
    ok (24ms)
    test it should let users with a valid token access the museums list ... Application running at http://localhost:9001
    POST http://localhost:9001/api/users/register - 0ms
    POST http://localhost:9001/api/login - 1ms
    GET http://localhost:9001/api/museums - 8ms
    ok (15ms)
    test it should respond with a 400 to a user with an invalid token ... Application running at http://localhost:9001
    An error occurred Authentication failed
    ok (5ms)
    test result: ok. 3 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out (45ms)
    ```

本书中我们将要编写的应用程序集成测试就到这里为止！如果你想要了解更多，请不要担心——关于测试的所有代码都可在本书的 GitHub 仓库中找到（[`github.com/PacktPublishing/Deno-Web-Development/tree/master/Chapter08/sections/7-final-tested-version/museums-api`](https://github.com/PacktPublishing/Deno-Web-Development/tree/master/Chapter08/sections/7-final-tested-version/museums-api)）。

我们现在对代码的信心大大增强。我们创造了机会，可以在以后更少的担忧下重构、扩展和维护代码。在隔离测试代码方面，我们所做的架构决策越来越显示出其价值。

在上一章中，当我们创建我们的 JavaScript 客户端时，我们提到了将其保存在 API 代码库中的一个优点是，我们可以轻松地为客户端和 API 编写测试，以确保它们能很好地一起工作。在下一节中，我们将展示如何做到这一点。这些测试将与我们在这里所做的非常相似，唯一的区别是，我们使用的是我们创建的 API 客户端，而不是使用`fetch`进行原始请求。

# 与应用和 API 客户端一起测试

当你向用户提供 API 客户端时，你有责任确保它与你的应用程序完美配合。确保这种配合的一种方法是拥有一个完整的测试套件，不仅测试客户端本身，还测试它与 API 的集成。在这里我们将处理后者。

我们将使用 API 客户端的一个特性，并创建一个测试，确保它正在工作。再次，你会注意到这些测试与我们在上一部分末尾编写的测试有一些相似之处。我们将复制之前测试的逻辑，但这次我们将使用客户端。让我们开始吧：

1.  在同一个`src/index.test.ts`文件中，为登录功能创建一个新的测试：

    ```js
    Deno.test("it returns user and token when user logs in
      with the client", async () => {})
    ```

    为了这次测试，我们知道我们需要访问 API 客户端。我们需要从`client`模块中导入它。

1.  从`src/client/index.ts`导入`getClient`函数：

    ```js
    import { getClient } from "./client/index.ts"
    ```

1.  让我们回到`src/index.test.ts`测试，导入`client`，从而创建一个它的实例。记住，它应该使用测试网络服务器创建的相同地址：

    ```js
    Deno.test("it returns user and token when user logs in
      with the client", async () => {
      const server = await createTestServer();
      const client = getClient({
    createTestServer function and this test, but for simplicity, we won't do this here.
    ```

1.  现在，只需编写调用使用`client`的`register`和`login`方法的逻辑即可。最终测试将如下所示：

    ```js
    Deno.test("it returns user and token when user logs in
      with the client", async () => {
    …
      // Register a user
      await client.register(
        { username: "test-user", password: "test-password"
           },
      );
      // Login with the createdUser
      const response = await client.login({
        username: "test-user",
        password: "test-password",
      });
      t.assertEquals(response.user.username, "test-user",
        "returns username");
      t.assert(!!response.user.createdAt, "has createdAt
        date");
      t.assert(!!response.token, "has token");
    …
    });
    ```

    注意我们是如何使用客户端的方法进行登录和注册，同时保留来自先前测试的断言。

遵循相同的指南，我们可以为客户端的所有功能编写测试，确保它与 API 一起正常工作，从而使我们能够自信地维护它。

为了简洁起见，而且因为这些测试类似于我们之前编写的测试，我们在这里不会提供为客户端所有功能编写测试的逐步指南。然而，如果你感兴趣，你可以在本书的 GitHub 存储库中找到它们([`github.com/PacktPublishing/Deno-Web-Development/blob/master/Chapter08/sections/7-final-tested-version/museums-api/src/index.test.ts`](https://github.com/PacktPublishing/Deno-Web-Development/blob/master/Chapter08/sections/7-final-tested-version/museums-api/src/index.test.ts)).

在下一节中，我们将简要介绍一个可能位于应用程序路径末端的特性。总有一天，你会发现应用程序的某些部分似乎变得很慢，你希望追踪它们的性能，这时性能测试就派上用场了。因此，我们将引入基准测试。

# 基准测试应用程序的部分

当涉及到在 JavaScript 中编写基准测试时，该语言本身提供了一些函数，所有这些函数都包含在高级分辨率时间 API 中。

由于 Deno 完全兼容 ES6，这些相同的功能都可以使用。如果你有时间查看 Deno 的标准库或官方网站，你会发现人们对基准测试给予了大量的关注，并且跟踪了 Deno 各个版本中的基准测试([`deno.land/benchmarks`](https://deno.land/benchmarks))。在检查 Deno 的源代码时，你会发现有关如何编写它们的非常不错的示例集。

对于我们的应用程序，我们可以轻松地使用浏览器上可用的 API，但 Deno 本身在其标准库中提供了功能，以帮助编写和运行基准测试，因此我们将在这里使用它。

首先，我们需要了解 Deno 的标准库基准测试工具，这样我们才知道我们可以做什么（[`github.com/denoland/deno/blob/ae86cbb551f7b88f83d73a447411f753485e49e2/std/testing/README.md#benching`](https://github.com/denoland/deno/blob/ae86cbb551f7b88f83d73a447411f753485e49e2/std/testing/README.md#benching)）。在本节中，我们将使用两个可用的函数编写一个非常简单的基准测试；即`bench`和`runBenchmarks`。第一个将定义一个基准测试，而第二个将运行它并将结果打印到控制台。

记得我们在第五章《添加用户和迁移到 Oak》中写的函数吗？该函数用于生成一个散列和一个盐，使我们能够将用户凭据安全地存储在数据库上。我们将按照以下步骤为此编写一个基准测试：

1.  首先，在`src/users/util.ts`旁边创建一个名为`utilBenchmarks.ts`的文件。

1.  导入我们要测试的`util`中的两个函数，即`generateSalt`和`hashWithSalt`：

    ```js
    import { generateSalt, hashWithSalt } from "./util.ts"
    ```

1.  是时候将基准测试工具添加到我们的`src/deps.ts`文件中，并运行`deno cache`命令（我们在第二章《工具链》中了解到它），在此处导入它。我们将把它作为`benchmark`导出到`src/deps.ts`中，以避免命名冲突：

    ```js
    export * as benchmark from
      "https://deno.land/std@0.83.0/testing/bench.ts";
    ```

1.  将基准测试工具导入到我们的基准文件中，并为`generateSalt`函数编写第一个基准测试。我们希望它运行 1000 次：

    ```js
    import { benchmarks } from "../deps.ts";
    benchmarks.bench({
      name: "runsSaltFunction1000Times",
      runs: 1000,
      func: (b) => {
        bench function (as stated in the documentation). Inside this object, we're defining the number of runs, the name of the benchmark, and the test function. That function is what will run every time, since an argument is an object of the BenchmarkTimer type with two methods; that is, start and stop. These methods are used to start and stop the timings of the benchmarks, respectively.
    ```

1.  我们所缺少的就是在基准测试定义之后调用`runBenchmarks`：

    ```js
    benchmarks.bench({
      name: "runsSaltFunction1000Times",
      …
    });
    benchmarks.runBenchmarks();
    ```

1.  是时候运行这个文件并查看结果了。

    记住，由于我们希望我们的基准测试精确，所以我们正在处理高级分辨率时间。为了让这段代码访问这个系统特性，我们需要以`--allow-hrtime`权限运行这个脚本（如第二章《工具链》中所解释）：

    ```js
    $ deno run --unstable --allow-plugin --allow-env --allow-read --allow-write --allow-hrtime src/users/utilBenchmarks.ts
    running 1 benchmarks ...
    benchmark runsSaltFunction1000Times ...
        1000 runs avg: 0.036691561000000206ms
    benchmark result: DONE. 1 measured; 0 filtered
    ```

1.  让我们为第二个函数编写基准测试，即`hashWithSalt`：

    ```js
    benchmarks.bench({
      name: "runsHashFunction1000Times",
      runs: 1000,
      func: (b) => {
        b.start();
        hashWithSalt("password", "salt");
        b.stop();
      },
    });
    benchmarks.runBenchmarks();
    ```

1.  现在，让我们运行它，以便我们得到最终结果：

    ```js
    $ deno run --allow-hrtime --unstable --allow-plugin --allow-env –-allow-write --allow-read src/users/utilBenchmarks.ts     
    running 2 benchmarks ...
    benchmark runsSaltFunction100Times ...
        1000 runs avg: 0.036691561000000206ms
    benchmark runsHashFunction100Times ...
        1000 runs avg: 0.02896806399999923ms
    benchmark result: DONE. 2 measured; 0 filtered
    ```

就是这样！现在您可以随时使用我们刚刚编写的代码来分析这些函数的性能。您可能需要这样做，是因为您已经更改了此代码，或者只是因为您想对其进行严格跟踪。您可以将其集成到诸如持续集成服务器之类的系统中，这样您就可以定期检查这些值并保持其正常运行。

这部分结束了本书的基准测试部分。我们决定给它一个简短的介绍，并展示从 Deno 获取的哪些 API 可以促进基准测试需求。我们相信，这里介绍的概念和示例将允许您跟踪应用程序的运行情况。

# 总结

随着这一章的结束，我们已经完成了我们一直在构建的应用程序的开发周期。我们开始时编写了一些简单的类和业务逻辑，编写了 web 服务器，最后将其与持久化集成。我们通过学习如何测试我们编写的功能来结束这一部分，这就是我们在这章所做的。我们决定使用几种不同类型的测试，而不是深入每个模块编写所有测试，因为我们认为这样做会带来更多的价值。

我们首先为业务逻辑编写了一个非常简单的单元测试，然后进行了一个带有多个类的集成测试，后来编写了一个针对 web 服务器的测试。这些测试只能通过利用我们创建的架构、遵循依赖注入原则，并尽可能使代码解耦来编写。

随着章节的进展，我们转向了集成测试，这些测试紧密地模仿了将在生产环境中运行的队列应用程序，使我们能够提高对我们刚刚编写的代码的信心。我们创建了测试，这些测试通过测试环境实例化了应用程序，使我们能够启动带有所有应用程序层（业务逻辑、持久化和网络）的 web 服务器，并对它进行断言。在这些测试中，我们可以非常有信心地断言登录和注册行为是否正常工作，因为我们向 API 发送了真实的请求。

为了结束这一章，我们将它与前一章连接起来，我们在那一章为 API 编写了一个 JavaScript 客户端。我们利用了客户端与 API 位于同一代码库中的一个巨大优势，并一起测试了客户端及其应用程序。这是确保一切按预期工作，并在发布 API 和客户端更改时保持信心的一种很好的方式。

这一章节试图展示如何在 Deno 中使用测试来提高我们对所编写代码的信心，以及当它们用于关注简单结果时所体现的价值。这类测试在应用更改时将非常有用，因为我们可以使用它们来添加更多功能或改进现有功能。在这里，我们了解到 Deno 提供的测试套件足以编写清晰、可读的测试，而无需任何第三方包。

下一章节将关注应用开发过程中最重要的阶段之一，那就是部署。我们将配置一个非常简单的持续集成环境，在该环境中我们可以将应用部署到云端。这是一个非常重要的章节，因为我们还将体验到 Deno 在部署方面的某些优势。

迫不及待地想让你的应用供用户使用吗？我们也是——让我们开始吧！
