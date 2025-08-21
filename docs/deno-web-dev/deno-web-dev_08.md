# 第六章：添加认证并连接到数据库

在上一章中，我们在应用程序中添加了一个 HTTP 框架，极大地简化了我们的代码。之后，我们在应用程序中添加了用户的概念，并开发了注册端点。目前为止，我们的应用程序已经存储了一些东西，唯一的缺点是它存储在内存中。我们将在本章解决这个问题。

在实现 oak（HTTP 框架的选择）时，我们使用的另一个概念是中间件函数。我们将从学习中间件函数是什么以及为什么它们几乎是所有 Node.js 和 Deno 框架中代码重用的*标准*开始本章。

然后我们将使用中间件函数并实现登录和授权。除此之外，我们还将学习如何使用中间件添加诸如请求日志和计时等标准功能到应用程序中。

随着我们的应用程序在需求方面几乎完成，我们将用剩余的时间学习如何连接到一个真正的持久化引擎。在这本书中，我们将使用 MongoDB。我们将使用之前构建的抽象确保过渡顺利。然后我们将创建一个新的用户存储库，以便它可以像我们使用内存解决方案一样连接到数据库。

到本章结束时，我们将拥有一个完整的应用程序，支持注册和用户登录。登录后，用户还可以获取博物馆列表。所有这些都是通过 HTTP 和持久化实现的业务逻辑完成的。

在本章之后，我们将只回来添加测试并部署应用程序，从而完成构建应用程序的完整周期。

在本章中，我们将涵盖以下主题：

+   使用中间件函数

+   添加认证

+   使用 JWT 添加授权

+   连接 MongoDB

让我们开始吧！

## 技术要求

本章所需的代码可在以下 GitHub 链接中找到：[`github.com/PacktPublishing/Deno-Web-Development/tree/master/Chapter06`](https://github.com/PacktPublishing/Deno-Web-Development/tree/master/Chapter06)。

# 使用中间件函数

如果您使用过任何 HTTP 框架，无论是 JavaScript 还是其他框架，您可能都熟悉中间件函数的概念。如果您不熟悉，也没关系——这就是我们将在本节解释的内容。

让我们从 Express.js 文档中借用的一个定义开始：[`expressjs.com/en/guide/writing-middleware.html`](http://expressjs.com/en/guide/writing-middleware.html)

“中间件函数是具有访问请求对象(req)、响应对象(res)以及应用程序请求-响应周期中下一个中间件函数的函数。下一个中间件函数通常由一个名为 next 的变量表示。”

中间件函数拦截请求并具有对它们进行操作的能力。它们可以在许多不同的用例中使用，如下所示：

+   更改请求和响应对象

+   结束请求-响应生命周期（回答请求或跳过其他处理程序）

+   调用下一个中间件函数

中间件函数通常用于诸如检查认证令牌、根据结果自动响应、记录请求、向请求中添加特定头、用上下文丰富请求对象和错误处理等任务。

我们将在应用程序中实现一些这些示例。

## 中间件是如何工作的？

中间件作为堆栈处理，每个函数都可以通过运行代码在堆栈执行前后控制响应流程。

在 oak 框架中，中间件函数是通过`use`函数进行注册的。这个时候，你可能还记得我们之前是如何使用 oak 的路由器的。oak 的`Router`对象所做的就是为注册的路由创建处理程序，并导出带有这种行为的中间件函数，以便在主应用程序上注册。这些被称为`routes`和`allowedMethods` ([`github.com/PacktPublishing/Deno-Web-Development/blob/43b7f7a40157212a3afbca5ba0ae20f862db38c4/ch5/sections/2-2-handling-routes-in-an-oak-application/museums-api/src/web/index.ts#L38`](https://github.com/PacktPublishing/Deno-Web-Development/blob/43b7f7a40157212a3afbca5ba0ae20f862db38c4/ch5/sections/2-2-handling-routes-in-an-oak-application/museums-api/src/web/index.ts#L38)).

为了更好地理解中间件函数，我们将实现它们中的几个。我们将在下一节中这样做。

## 通过中间件添加请求计时

让我们在请求中添加一些基本日志记录。oak 中间件函数（[`github.com/oakserver/oak#application-middleware-and-context`](https://github.com/oakserver/oak#application-middleware-and-context)）接收两个参数。第一个是上下文对象，这是所有路由都得到的一个对象，而第二个是`next`函数。这个函数可以用来执行堆栈中的其他中间件，允许我们控制应用程序流程。

我们首先要添加一个中间件，为响应添加`X-Response-Time`头。按照以下步骤操作：

1.  打开`src/web/index.ts`，并注册一个通过调用`next`执行剩余堆栈的中间件。

    这为响应添加了一个头，其值为从请求开始到处理完毕的毫秒差：

    ```js
    const app = new Application();
    .use calls; this way, all the other middleware functions will run once this has been executed.The first lines are executed before the route handler (and other middleware functions) starts handling the request. Then, the call to `next` makes sure the route handlers execute; only then is the rest of the middleware code executed, thus calculating the difference from the initial value and the current date and adding it as a header.
    ```

1.  执行以下代码以启动服务器：

    ```js
    $ deno run --allow-net src/index.ts
    Application running at http://localhost:8080
    ```

1.  发起一个请求，并检查是否有了所需的头：

    ```js
    x-response-time header there. Note that we've used the -i flag so that we're able to see the response headers on curl. 
    ```

有了这个，我们首次完全理解后使用了中间件函数。我们用它们来控制应用程序的流程，通过使用`next`，并为请求添加了一个头。

接下来，我们将对刚刚创建的中间件进行组合并添加逻辑，以记录向服务器发起的请求。

## 通过中间件添加请求日志

现在我们已经构建了计算请求时间的逻辑，我们处于向应用程序添加请求日志的好位置。

最终目标是让每个向应用程序发起的请求都记录在控制台上，包括其路径、HTTP 方法和响应时间；像以下示例一样：

```js
GET http://localhost:8080/api/museums - 65ms
```

当然，我们也可以每个请求分别处理，但由于这是一件需要跨应用程序做的事情，我们将把它作为中间件添加到`Application`对象中。

我们在上一节编写的 middleware 要求处理程序（以及中间件函数）运行，以便添加响应时间（它在执行部分逻辑之前调用 next 函数）。我们需要在之前注册当前的日志中间件，它将请求时间添加到请求中。让我们开始：

1.  打开`src/web/index.ts`并在控制台上添加记录请求方法、路径和时间戳的代码：

    ```js
    X-Response-Time header, which is going to be set by the previous middleware to log the request to the console. We're also using next to make sure all the handlers (and middleware functions) run before we log to the console. We need this specifically because the header is set by another piece of middleware.
    ```

1.  执行以下代码以启动服务器：

    ```js
    $ deno run --allow-net src/index.ts
    Application running at http://localhost:8080
    ```

1.  对端点执行请求：

    ```js
    $ curl http://localhost:8080/api/museums
    ```

1.  检查服务器进程是否将请求记录到控制台：

    ```js
    $ deno run --allow-net src/index.ts
    Application running at http://localhost:8080
    GET http://localhost:8080/api/museums - 46ms
    ```

这样一来，我们的中间件函数就可以协同工作了！

在这里，我们在主要的应用程序对象上注册了中间件函数。然而，也可以通过调用相同的`use`方法在特定的 oak 路由上执行此操作。

为了给您一个例子，我们将注册一个只会在`/api`路由来执行的中间件。我们将做与之前完全相同的事情，但这次调用的是 API`Router`对象的`use`方法，如下例所示：

```js
const apiRouter = new Router({ prefix: "/api" })
apiRouter.use(async (_, next) => {
  console.log("Request was made to API Router");
  await next();
}))
…
app.use(apiRouter.routes());
app.use(apiRouter.allowedMethods());
```

想要应用程序流程正常进行的中间件函数*必须调用*`next`函数。如果这种情况没有发生，堆栈中的其余中间件和路由处理程序将不会被执行，因此请求将无法得到响应。

使用中间件函数的另一种方法是将它们直接添加到请求处理程序之前。

假设我们想要创建一个添加`X-Test`头的中间件。我们可以在应用程序对象上编写该中间件，或者我们可以在路由本身上直接使用它，如下代码所示：

```js
import { Application, Router, RouterMiddleware } from
  "../deps.ts";
…
const addTestHeaderMiddleware: RouterMiddleware = async (ctx,
   next) => {
  ctx.response.headers.set("X-Test", "true");
  await next();
}
apiRouter.get("/museums", addTestHeaderMiddleware, async (ctx)
  => {
  ctx.response.body = {
    museums: await museum.getAll()
  }
});
```

为了让之前的代码运行，我们需要在`src/deps.ts`中导出`RouterMiddleware`类型：

```js
export type { RouterMiddleware } from
  "https://deno.land/x/oak@v6.3.1/mod.ts";
```

使用这个中间件，无论何时我们想要添加`X-Test`头，只需要在路由处理程序之前包含`addTestHeaderMiddleware`。它会在处理程序代码之前执行。这不仅仅适用于一个中间件，因为可以注册多个中间件函数。

中间件函数就到这里结束！

我们已经学习了使用这种非常常见的 web 框架特性来创建和共享功能的基本知识。在我们进入下一部分时，我们将继续使用它们，在那里我们将处理认证、验证令牌和授权用户。

让我们来实现我们应用程序的认证！

# 添加认证

在上一章中，我们向应用程序添加了创建新用户的功能。这个功能本身很酷，但如果我们不能用它来进行认证，那么它就值不了多少。这就是我们在这里要做的。

我们先来创建检查用户名和密码组合是否正确的逻辑，然后实现一个端点来完成这个任务。

之后，我们将通过从登录端点返回令牌来过渡到授权主题，然后使用该令牌来检查用户是否已认证。

让我们一步一步来，从业务逻辑和持久性层开始。

## 创建登录业务逻辑

我们的一种实践是，在编写新功能时，首先从业务逻辑开始。我们认为这是直观的，因为你首先考虑“业务”和用户，然后才进入技术细节。这就是我们要在这里做的。

我们首先在`UserController`中添加登录逻辑：

1.  在开始之前，让我们在`src/users/types.ts`中为`UserController`接口添加`login`方法：

    ```js
    export type RegisterPayload = { username: string;
      password: string };
    export type LoginPayload = { username: string; password:
      string };
    export interface UserController {
      register: (payload: RegisterPayload) =>
        Promise<UserDto>;
      login: (
        { username, password }: LoginPayload,
      ) => Promise<{ user: UserDto }>;
    }
    ```

1.  在控制器上声明`login`方法；它应该接收一个用户名和密码：

    ```js
    public async login(payload: LoginPayload) {
    }
    ```

    让我们停下来思考一下登录流程应该是什么样子：

    +   用户发送他们的用户名和密码。

    +   应用程序通过用户名从数据库中获取用户。

    +   应用程序使用数据库中的盐对用户发送的密码进行编码。

    +   应用程序比较两个加盐密码。

    +   应用程序返回一个用户和一个令牌。

        现在我们不担心令牌。然而，流程的其余部分应该为当前部分设置要求，帮助我们思考`login`方法的代码。

        单从这些要求来看，我们就可以理解我们需要在`UserRepository`上有一个通过用户名获取用户的方法。让我们来看看这个。

1.  在`src/users/types.ts`中，向`UserRepository`添加一个`getByUsername`方法；它应该通过用户名从数据库中获取用户：

    ```js
    export interface UserRepository {
      create: (user: CreateUser) => Promise<User>;  
      exists: (username: string) => Promise<boolean>
      getByUsername: (username: string) => Promise<User>
    }
    ```

1.  在`src/users/repository.ts`中实现`getByUsername`方法：

    ```js
    export class Repository implements UserRepository {
    …
    UserController and use the recently created method to get a user from the database.
    ```

1.  在`UserController`的`login`方法内部使用来自仓库的`getByUsername`方法：

    ```js
    public async login(payload: LoginPayload) {
      hashPassword in the previous chapter when we implemented the register logic, so let's use that.
    ```

1.  在`UserController`内部创建一个`comparePassword`方法。

    它应该接收一个密码和一个`user`对象。然后，它应该将用户发送的密码一旦被加盐和哈希与数据库中存储的密码进行比较：

    ```js
    import {
      LoginPayload,
      RegisterPayload,
      User,
      UserController,
      UserRepository,
    } from "./types.ts";
    import { hashWithSalt } from "./util.ts"
    …
    private async comparePassword(password: string, user:
      User) {
      const hashedPassword = hashWithSalt (password,
        user.salt);
      if (hashedPassword === user.hash) {
        return Promise.resolve(true);
      }
      return Promise.reject(false);
    }
    ```

1.  在`UserController`的`login`方法上使用`comparePassword`方法：

    ```js
    public async login(payload: LoginPayload) {
      try {
        const user = await
         this.userRepository.getByUsername(payload.username);
        await this.comparePassword(payload.password, user);
        return { user: userToUserDto(user) };
      } catch (e) {
        console.log(e);
        throw new Error('Username and password combination is
          not correct')
      }
    }
    ```

有了这个，我们就有了`login`方法的工作！

它接收一个用户名和一个密码，通过用户名获取用户，比较哈希密码，如果一切按计划进行，则返回用户。

现在我们应该准备好实现登录端点——一个将使用我们刚刚创建的登录方法。

## 创建登录端点

既然我们已经创建了业务逻辑和数据获取逻辑，我们就可以开始在我们的网络层中使用它。让我们创建一个`POST /api/login`路由，该路由应该允许用户使用他们的用户名和密码登录。按照以下步骤操作：

1.  在`src/web/index.ts`中创建登录路由：

    ```js
    apiRouter.post("/login", async (ctx) => {
    })
    ```

1.  使用`request.body`函数获取请求体（[`doc.deno.land/https/raw.githubusercontent.com/oakserver/oak/main/request.ts#Request`](https://doc.deno.land/https/raw.githubusercontent.com/oakserver/oak/main/request.ts#Request))，然后将用户名和密码发送到`login`方法：

    ```js
    apiRouter.post("/login", async (ctx) => {
      400 Bad Request) if things didn't go well.
    ```

1.  如果登录成功，它应该返回我们的`user`：

    ```js
    …
    const { user: loginUser } = await user.login({ username,
      password });
    ctx.response.body = { user: loginUser };
    ctx.response.status = 201;
    …
    ```

    有了这些，我们应该拥有登录用户所需的一切！让我们试一试。

1.  运行应用程序，通过运行以下命令：

    ```js
    $ deno run --allow-net src/index.ts
    Application running at http://localhost:8080
    ```

1.  向`/api/users/register`发送请求以注册用户，然后尝试使用创建的用户登录到`/api/login`：

    ```js
    $ curl -X POST -d '{"username": "asantos00", "password": "testpw" }' -H 'Content-Type: application/json' http://localhost:8080/api/users/register
    {"user":{"username":"asantos00","createdAt":"2020-10-19T21:30:51.012Z"}}
    ```

1.  现在，尝试使用创建的用户登录：

    ```js
    $ curl -X POST -d '{"username": "asantos00", "password": "testpw" }' -H 'Content-Type: application/json' http://localhost:8080/api/login 
    {"user":{"username":"asantos00","createdAt":"2020-10-19T21:30:51.012Z"}}
    ```

而且它有效！我们在注册表上创建用户，并能够在之后使用他们登录。

在本节中，我们学习了如何向我们的应用程序添加认证逻辑，并实现了`login`方法，该方法允许用户使用注册的用户登录。

在下一节中，我们将学习如何使用我们创建的认证来获取一个令牌，该令牌将允许我们处理授权。我们将使博物馆路线只对认证用户可用，而不是公开可用。为此，我们需要开发授权功能。让我们深入了解一下！

# 使用 JWT 添加授权

现在，我们有一个允许我们登录并返回已登录用户的应用程序。然而，如果我们想要在 API 中使用登录，我们必须创建一个授权机制。这个机制应该启用 API 的用户进行认证，获取一个令牌，并使用这个令牌来标识自己并访问资源。

我们这样做是因为我们希望关闭应用程序的某些路由，使它们只对认证用户可用。

我们将开发所需内容，通过使用**JSON Web Tokens**（**JWT**），这是一种在 API 中相当标准的认证方式。

如果你不熟悉 JWT，我将留下一个来自[jwt.io](http://jwt.io)的解释：

"JSON Web Tokens 是一种开放、行业标准的 RFC 7519 方法，用于在两个方之间安全地表示声明。"

它主要用于当你希望你的客户端连接到一个认证服务，然后提供你的服务器验证该认证是否由一个你信任的服务发出。

为了避免重复[jwt.io](http://jwt.io)已经很好地解释过的风险，我将给你一个链接，完美地解释了这个标准是什么：`[`jwt.io/introduction/`](https://jwt.io/introduction/)`。确保阅读它；我相信你们都有足够的知识来理解我们接下来如何使用它。

在本节中，由于本书的范围，我们将不会实现生成和验证 JWT 令牌的全部逻辑。这段代码可以在本书的 GitHub 仓库中找到([`github.com/PacktPublishing/Deno-Web-Development/tree/master/Chapter06/jwt-auth`](https://github.com/PacktPublishing/Deno-Web-Development/tree/master/Chapter06/jwt-auth))。

我们将要在这里做的是将我们当前的应用程序与一个具有生成和验证 JWT 令牌功能的模块集成，这对我们的应用程序至关重要。然后，我们使用该令牌来决定是否允许用户访问博物馆路线。

让我们开始吧！

## 从登录返回令牌

在前一节中，我们实现了登录功能。我们开发了一些逻辑来验证用户名和密码的组合，如果成功就返回用户。

为了授权一个用户并让他们访问私有资源，我们需要知道认证的用户是谁。一个常见的做法是通过令牌来实现。我们有各种方法可以做到这一点。它们包括基本 HTTP 认证、会话令牌、JWT 令牌等替代方案。我们选择 JWT，因为我们认为它是业界广泛使用的解决方案，你们可能会在工作中遇到。如果你们没有遇到过，也不要担心；它是足够简单的。

我们需要做的第一件事是在用户登录时向用户返回令牌。我们的`UserController`将不得不在与`userDto`结合时返回该令牌。

在提供的`jwt-auth`模块中([`github.com/PacktPublishing/Deno-Web-Development/tree/master/Chapter06/jwt-auth`](https://github.com/PacktPublishing/Deno-Web-Development/tree/master/Chapter06/jwt-auth)),你可以检查我们导出了一个仓库。

如果我们访问文档，使用 Deno 的文档网站在[`doc.deno.land/https/raw.githubusercontent.com/PacktPublishing/Deno-Web-Development/master/Chapter06/jwt-auth/repository.ts`](https://doc.deno.land/https/raw.githubusercontent.com/PacktPublishing/Deno-Web-Development/master/Chapter06/jwt-auth/repository.ts)，我们可以看到它导出了两个方法：`getToken`和`generateToken`。

阅读方法的文档，我们可以理解，一个为用户 ID 获取令牌，另一个生成新令牌。

让我们使用这个方法，按照以下步骤在我们的登录用例中生成新令牌：

1.  首先，在`src/users/types.ts`中的`UserController`返回类型中添加令牌：

    ```js
    export interface UserController {
      register: (payload: RegisterPayload) =>
        Promise<UserDto>
      login: ({ username, password }: LoginPayload) =>
        Promise<{ user: UserDto, UserController knows how to return a token. Looking at its logic, we can see that it should be able to delegate that responsibility by calling a method that will return that token. From the previous chapters, we know that we don't want to import our dependencies directly; we'd rather have them injected into our `constructor`. That's what we'll do here. Another thing we know is that we want to use this "third-party module" that deals with authentication. We'll need to add it to our dependencies file.
    ```

1.  前往`src/deps.ts`，为`jwt-auth`模块添加导出，运行`deno cache`以更新锁文件并下载依赖项：

    ```js
    export type {
      Algorithm,
    } from "https://raw.githubusercontent.com/PacktPublishing/
     Deno-Web-Development/master/Chapter06/jwt-auth/mod.ts";
    export {
      Repository as AuthRepository,
    } from "https://raw.githubusercontent.com/PacktPublishing/
      Deno-Web-Development/master/Chapter06/jwt-auth/mod.ts";
    ```

1.  使用`AuthRepository`类型定义`UserController`构造函数的依赖项：

    ```js
    authRepository, which we've just imported. We previously discovered that it exposes a generateToken method, which will be of use to the login of UserController.
    ```

1.  打开`src/users/controller.ts`中的登录方法，并使用`authRepository`中的`generateToken`方法来获取令牌并返回它：

    ```js
    public async login(payload: LoginPayload) {
        try {
          const user = await
            this.userRepository.getByUsername
              (payload.username);
          await this.comparePassword(payload.password, user);
    authRepository to get a token. If we try to run this code, we know it will fail. In fact, we just need to open `src/index.ts` to see our editor's warnings. It is complaining that we're not sending `authRepository` to `UserController`, and we should.
    ```

1.  回到`src/index.ts`，从`jwt-auth`实例化`AuthRepository`：

    ```js
    import { AuthRepository } from "./deps.ts";
    …
    const authRepository = new AuthRepository({
      configuration: {
        algorithm: "HS512",
        key: "my-insecure-key",
        tokenExpirationInSeconds: 120
      }
    });
    ```

    你也可以通过模块的文档来检查，因为它需要一个带有三个属性的`configuration`对象发送，即`algorithm`、`key`和`tokenExpirationInSeconds`。

    `key`应该是一个秘密值，用于创建和验证 JWT，`algorithm`是令牌将编码的加密算法（支持 HS256、HS512 和 RS256），`tokenExpirationInSeconds`是令牌过期的时间。

    关于我们刚刚提到的`key`变量等不应存在于代码中的秘密值，我们将在下一章学习如何处理它们，那里我们将讨论应用程序配置。

    我们现在有一个`AuthRepository`的实例！我们应该能够将其发送到我们的`UserController`并使其工作。

1.  在`src/index.ts`中，将`authController`发送到`UserController`构造函数中：

    ```js
    const userController = new UserController({
      userRepository, authRepository });
    ```

    现在，你应该能够运行应用程序！

    现在，如果你创建几个请求来测试它，你会注意到`POST /login`端点仍然没有返回令牌。让我们解决这个问题！

1.  打开`src/web/index.ts`，在`login`路线上，确保我们从响应中的`login`方法返回`token`：

    ```js
    apiRouter.post("/login", async (ctx) => {
      const { username, password } = await
        ctx.request.body().value;
      try {
        const { user: loginUser, token } = await user.login({
          username, password });
        ctx.response.body = { user: loginUser, token };
        ctx.response.status = 201;
      } catch (e) {
        ctx.response.body = { message: e.message };
        ctx.response.status = 400;
      }
    })
    ```

我们几乎完成了！我们成功完成了第一个目标：使`login`端点返回一个令牌。

我们接下来要实现的是确保用户在尝试访问认证路线时始终发送令牌的逻辑。

我们继续完善认证逻辑。

## 创建一个认证路线

有了向用户获取令牌的能力，我们现在希望确保只有登录的用户能够访问博物馆路线。

用户必须将令牌发送到`Authorization`头中，正如 JWT 令牌标准所定义的。如果令牌无效或不存在，用户应显示`401 Unauthorized`状态码。

验证用户在请求中发送的令牌是中间件函数的一个很好的用例。

为了做到这一点，既然我们正在使用`oak`，我们将使用一个名为`oak-middleware-jwt`的第三方模块。这只是一个自动验证 JWT 令牌的中间件，基于密钥，并提供对我们有用的功能。

你可以查看其文档在[`nest.land/package/oak-middleware-jwt`](https://nest.land/package/oak-middleware-jwt)。

让我们在我们的网络代码中使用这个中间件，使博物馆路线只对认证用户可用。按照以下步骤操作：

1.  在`deps.ts`文件中添加`oak-middleware-jwt`，并导出`jwtMiddleware`函数：

    ```js
    export {
      jwtMiddleware,
    } from "https://x.nest.land/
       oak-middleware-jwt@2.0.0/mod.ts";
    ```

1.  回到`src/web/index.ts`，在博物馆路由中使用`jwtMiddleware`，在那里发送密钥和算法。

    不要忘记我们在上一节中提到的内容——中间件函数可以通过在路由处理程序之前发送它，在任何路由中使用：

    ```js
    import { Application, src/index.ts and forget to change this.This is exactly why we should extract this and expect it as a parameter to the `createServer` function.
    ```

1.  在`createServer`函数中向`configuration`内部添加`authorization`作为参数：

    ```js
    import { Algorithm type from the deps.ts file, which exports it from the jwt-auth module. We're doing this so that we can ensure, via types, that the algorithms that are sent are only the supported ones.
    ```

1.  现在，仍然在`src/web/index.ts`中，使用`authorization`参数发送将被注入到`jwtMiddleware`中的值：

    ```js
    const authenticated = jwtMiddleware(authorization)
    ```

    我们唯一缺少的是实际上将`authorization`值发送到`createServer`函数的能力。

1.  在`src/index.ts`中，将认证配置提取到一个变量中，以便我们可以重复使用：

    ```js
    import { AuthRepository, Algorithm } from "./deps.ts";
    …
    const authConfiguration = {
      algorithm: "HS512" as Algorithm,
      key: "my-insecure-key",
      tokenExpirationInSeconds: 120
    }
    const authRepository = new AuthRepository({
      configuration: authConfiguration
    });
    ```

1.  让我们重复使用那个相同的变量来发送发送到`createServer`所需参数：

    ```js
    createServer({
      configuration: {
        port: 8080,
        authorization: {
          key: authConfiguration.key,
          algorithm: authConfiguration.algorithm
        }
      },
      museum: museumController,
      user: userController
    })
    ```

    大功告成！让我们测试一下我们的应用程序，看看它是否按预期工作。

    请注意，期望的行为是只有认证用户才能访问博物馆路由并看到所有的博物馆。

1.  让我们通过运行以下命令来运行应用程序：

    ```js
    $ deno run --allow-net src/index.ts
    Application running at http://localhost:8080
    ```

1.  让我们注册一个用户，这样我们就可以登录了：

    ```js
    $ curl -X POST -d '{"username": "asantos00", "password": "testpw1" }' -H 'Content-Type: application/json' http://localhost:8080/api/users/register
    {"user":{"username":"asantos00","createdAt" :"2020-10-27T19:14:01.984Z"}}
    ```

1.  现在，让我们登录，这样我们就可以获得我们的令牌：

    ```js
    $ curl -X POST -d '{"username": "asantos00", "password": "testpw1" }' -H 'Content-Type: application/json' http://localhost:8080/api/login
    {"user":{"username":"asantos00","createdAt":"2020-10-27T19:14:01.984Z"},"token":"eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJtdXNldW1zIiwiZXhwIjoxNjAzODI2NTEzLCJ1c2VyIjoi YXNhbnRvczAwIn0.XV1vaHDpTu2SnavFla5q8eIPKCRIfDw_Kk-j8gi1 mqcz5UN3sVnk61JWCapwlh0IJ46fJdc7cw2WoMMIh-ypcg"}
    ```

1.  最后，让我们尝试使用从前一个请求返回的令牌访问博物馆路由：

    ```js
    Authentication header with Bearer as a prefix, as specified by the JWT specification.
    ```

1.  为了确保它按预期工作，让我们尝试在不带`Authorization`头的相同请求中，期望一个`unauthorized`响应：

    ```js
    -i flag with curl so that it logs the request status code and headers.
    ```

就这些！现在我们已经成功创建了一个仅限认证用户访问的路由。这在任何包含用户的应用程序中都非常常见。

如果我们更深入地了解这个问题，我们可以探索 JWT `refreshToken`，或者甚至如何从 JWT 令牌中读取用户信息，但这些都超出了本书的范围。这是我要让您自己探索的东西。

在本节中，我们实现了我们的目标，并查看了 API 的许多不同部分。

不过还有一件事缺失：与真实持久化引擎的连接。这就是我们接下来要做的——将我们的应用程序连接到 NoSQL 数据库！

# 连接到 MongoDB

到目前为止，我们已经实现了一个列出博物馆的应用程序，并包含用户，允许他们进行认证。这些功能已经就位，但它们都有一个缺点：它们都在内存数据库上运行。

我们选择这种方式是为了简化问题。然而，由于我们的大部分实现都不依赖于交付机制，如果数据库发生变化，它应该不会有多大变化。

从这一节的标题中，您可能已经猜到，我们将学习如何将应用程序的一个实体移动到数据库。我们将利用我们已经创建的抽象来实现这一点。这个过程将与所有实体非常相似，因此我们决定学习如何连接数据库，只为了用户模块。

稍后，如果您好奇如果所有应用程序都连接到数据库，这会怎样工作，您将有机会检查这本书的 GitHub 仓库。

为了确保我们都对类似的数据库进行操作，我们将使用 MongoDB Atlas。Atlas 是一个提供免费 MongoDB 集群的产品，我们可以用来连接我们的应用程序。

如果你不熟悉 MongoDB，这里有一个来自他们网站的“一句话解释”（[`www.mongodb.com/`](https://www.mongodb.com/)）。请随意去那里了解更多：

"MongoDB 是一个通用目的、基于文档、分布式数据库，为现代应用程序开发人员和云时代而构建。"

准备好了吗？让我们开始吧！

## 创建一个用户 MongoDB 存储库

我们当前的`UserRepository`是负责将用户与数据库连接的模块。这就是我们想要更改的，以便我们的应用程序连接到一个 MongoDB 实例，而不是一个内存数据库。

我们将通过创建新的 MongoDB 存储库、将其暴露给世界、并将我们应用程序的其余部分连接到它的步骤。

首先，通过重新组织用户模块的内部文件结构，为新的用户存储库创建空间。

### 重新排列我们的用户模块

我们的用户模块最初设想只有一个存储库，因此它没有相应的文件夹；只是一个`repository.ts`文件。现在我们考虑将用户保存到数据库的更多方法，我们需要创建它。

记得我们第一次谈到架构时，提到了它将不断进化吗？这就是正在发生的事情。

让我们重新排列用户模块，以便它可以处理多个存储库并添加一个 MongoDB 存储库，遵循我们之前创建的`UserRepository`接口：

1.  在`src/users`内创建一个名为`repository`的文件夹，并将实际的`src/users/repository.ts`移动到那里，将其重命名为`inMemory.ts`：

    ```js
    └── src
        ├── museums
        ├── users
        │   ├── adapter.ts
        │   ├── controller.ts
        │   ├── index.ts
        │   ├── repository
    │ │   ├── inMemory.ts
        │   ├── types.ts
        │   └── util.ts
    ```

1.  记得修复`src/users/repository/inMemory.ts`内的模块导入：

    ```js
    import { User, UserRepository } from "../types.ts";
    import { generateSalt, hashWithSalt } from "../util.ts";
    ```

1.  为了保持应用程序的运行，让我们前往`src/users/index.ts`并导出正确的存储库：

    ```js
    export { Repository } from './repository/inMemory.ts'
    ```

1.  现在，让我们创建一个 MongoDB 存储库。将其命名为`mongoDb.ts`，并将其放入`src/users/respository`文件夹内：

    ```js
    import { UserRepository } from "../types.ts";
    export class Repository implements UserRepository {
      storage
      async create(username: string, password: string) {
      }
      async exists(username: string) {
      }
      async getByUsername(username: string) {
      }
    }
    ```

    确保它实现了我们之前定义的`UserRepository`接口。

这里就是所有乐趣开始的地方！现在我们有了 MongoDB 存储库，我们将开始编写它并将其连接到我们的应用程序。

## 安装 MongoDB 客户端库

我们已经有了一个我们存储库需要实现的方法列表。遵循接口，我们可以保证我们的应用程序会工作，不管实现方式如何。

有一件事我们可以肯定，因为我们不想一直重新发明轮子：我们将使用第三方包来处理与 MongoDB 的连接。

我们将使用`deno-mongo`包进行此操作（[`github.com/manyuanrong/deno_mongo`](https://github.com/manyuanrong/deno_mongo)）。

重要提示

Deno 的 MongoDB 驱动程序使用 Deno 插件 API，该 API 仍处于不稳定状态。这意味着我们将不得不以`--unstable`标志运行我们的应用程序。由于它目前正在使用尚未被认为是稳定的 API，因此暂时不应在生产环境中使用。

让我们看看文档中的示例，其中建立了与 MongoDB 数据库的连接：

```js
import { MongoClient } from
  "https://deno.land/x/mongo@v0.13.0/mod.ts";
const client = new MongoClient();
client.connectWithUri("mongodb://localhost:27017");
const db = client.database("test");
const users = db.collection<UserSchema>("users");
```

在这里，我们可以看到我们将需要创建一个 MongoDB 客户端并使用包含主机（可能包含主机的用户名和密码）的连接字符串连接到数据库。

之后，我们需要让客户端访问一个特定的数据库（在这个例子中是`test`）。只有这样，我们才能拥有允许我们与集合（在这个例子中是`users`）交互的处理程序。

首先，让我们将`deno-mongo`添加到我们的依赖列表中：

1.  前往你的`src/deps.ts`文件，并在那里添加`MongoClient`的导出：

    ```js
    export { MongoClient } from
      "https://deno.land/x/mongo@v0.13.0/mod.ts";
    ```

1.  现在，确保运行`cache`命令以安装模块。我们将不得不使用`--unstable`标志运行它，因为我们要安装的插件在安装时也需要不稳定的 API：

    ```js
    $ deno cache --lock=lock.json --lock-write --unstable src/deps.ts
    ```

有了这个，我们已经用我们刚刚安装的包更新了`deps.ts`文件！

让我们继续使用这个包来开发我们的仓库。

## 开发 MongoDB 仓库

从我们从文档中获得的示例中，我们学会了如何连接到数据库并创建我们想要的用户集合的处理程序。我们知道我们的仓库需要访问这个处理程序，以便它可以与集合交互。

再次，我们可以在仓库内部直接创建 MongoDB 客户端，但这将使我们无法在没有尝试连接到 MongoDB 的情况下测试该仓库。

由于我们尽可能希望将依赖项注入到模块中，我们将通过其构造函数将 MongoDB 客户端传递给我们的仓库，这在代码的其他部分非常类似于我们做的。

让我们回到我们的 MongoDB 仓库，并按照这些步骤进行操作：

1.  在 MongoDB 仓库内创建`constructor`方法。

    确保它接收一个具有名为`storage`的`Database`类型的属性的对象，该属性是由`deno-mongo`包导出的：

    ```js
    import { User, UserRepository } from "../types.ts";
    collection method on it, to get access to the users' collection. Once we've done that, we must set it to our storage class property. Both the method and the type require a generic to be passed in. This should represent the type of object present in that collection. In our case, it is the User type.
    ```

1.  现在，我们必须进入`src/deps.ts`文件，并从`deno-mongo`中导出`Database`和`Collection`类型：

    ```js
    export { MongoClient, Collection, Database } from
      "https://deno.land/x/mongo@v0.13.0/mod.ts";
    ```

现在，这只是开发满足`UserRepository`接口的方法的问题。

这些方法将非常类似于我们为内存数据库开发的那些方法，区别在于我们现在在与 MongoDB 集合交互，而不是我们之前使用的 JavaScript Map。

现在，我们只需要实现一些方法，这些方法将创建用户、验证用户是否存在，并通过用户名获取用户。这些方法在插件文档中可用，非常接近 MongoDB 的本地 API。

这是最终类的样子：

```js
import { CreateUser, User, UserRepository } from
 "../types.ts";
import { Collection, Database } from "../../deps.ts";
export class Repository implements UserRepository {
  storage: Collection<User>
  constructor({ storage }: RepositoryDependencies) {
    this.storage = storage.collection<User>("users");
  }
  async create(user: CreateUser) {
    const userWithCreatedAt = { ...user, createdAt: new Date() }
    this.storage.insertOne({ ...user })
    return userWithCreatedAt;
  }
  async exists(username: string) {
    return Boolean(await this.storage.count({ username }));
  }
  async getByUsername(username: string) {
    const user = await this.storage.findOne({ username });
    if (!user) {
      throw new Error("User not found");
    }
    return user;
  }
}  
```

我们突出了使用`deno-mongo`插件访问数据库的方法。注意逻辑与我们之前做的非常相似。我们在`create`方法中添加了创建日期，然后从 mongo 调用`create`方法。在`exists`方法中，我们调用`count`方法，并将其转换为`boolean`。对于`getByUsername`方法，我们使用 mongo 库中的`findOne`方法，发送用户名。

如果你对如何使用这些 API 有任何疑问，请查看 deno-mongo 的文档 ([`github.com/manyuanrong/deno_mongo`](https://github.com/manyuanrong/deno_mongo)).

## 将应用程序连接到 MongoDB

现在，为了暴露我们创建的 MongoDB 仓库，我们需要进入`src/users/index.ts`并将其作为`Repository`暴露（删除高亮显示的行）：

```js
export { Repository } from "./repository/mongoDb.ts";
export { Repository } from "./repository/inMemory.ts";
```

现在，我们应该在我们的编辑器和 typescript 编译器中看到抱怨，抱怨我们在`src/index.ts`中实例化`UserRepository`时没有发送正确的依赖关系，这是正确的。所以，让我们去那里修复它。

在将数据库客户端发送到`UserRepository`之前，它需要被实例化。通过查看`deno-mongo`的文档，我们可以看到以下示例：

```js
const client = new MongoClient();
client.connectWithUri("mongodb://localhost:27017");
```

我们没有连接到 localhost，所以我们需要稍后更改连接 URI。

让我们按照文档的示例，编写连接到 MongoDB 实例的代码。按照以下步骤操作：

1.  在将`MongoClient`的导出添加到`src/deps.ts`文件后，在`src/index.ts`中导入它：

    ```js
    import { MongoClient } from "./deps.ts";
    ```

1.  然后，调用`connectWithUri`：

    ```js
    const client = new MongoClient();
    client.connectWithUri("mongodb://localhost:27017");
    ```

1.  然后，通过在客户端上调用`database`方法来获取一个数据库处理器：

    ```js
    const db = client.database("getting-started-with-deno");
    ```

这应该是我们连接到 MongoDB 所需的所有内容。唯一缺少的是将数据库处理器发送到`UserRepository`的代码。所以，让我们添加这个：

```js
const client = new MongoClient();
client.connectWithUri("mongodb://localhost:27017");
const db = client.database("getting-started-with-deno");
...
const userRepository = new UserRepository({ storage: db });
```

不应该有任何警告出现，我们应该现在能够运行我们的应用程序了！

然而，我们仍然没有一个数据库可以连接。我们接下来会看看这个问题。

## 连接到 MongoDB 集群

现在，我们需要连接到一个真实的 MongoDB 实例。在这里，我们将使用一个名为 Atlas 的服务。Atlas 是 MongoDB 提供的一个云 MongoDB 数据库服务。他们的免费层非常慷慨，非常适合我们的应用程序。在那里创建一个账户。完成后，我们可以创建一个 MongoDB 集群。

重要提示

如果你有其他任何 MongoDB 实例，无论是本地的还是远程的，都可以跳过下一段，直接将数据库 URI 插入代码中。

以下链接包含创建一个集群所需的所有说明：[`docs.atlas.mongodb.com/tutorial/create-new-cluster/`](https://docs.atlas.mongodb.com/tutorial/create-new-cluster/)。

一旦集群被创建，我们还需要创建一个可以访问它的用户。前往[`docs.atlas.mongodb.com/tutorial/connect-to-your-cluster/index.html#connect-to-your-atlas-cluster`](https://docs.atlas.mongodb.com/tutorial/connect-to-your-cluster/index.html#connect-to-your-atlas-cluster)了解如何获取连接字符串。

它应该看起来像下面这样：

```js
mongodb+srv://<username>:<password>@clustername.mongodb.net/
  test?retryWrites=true&w=majority&useNewUrlParser=
    true&useUnifiedTopology=true
```

现在我们有了连接字符串，我们只需要将其传递给之前在`src/index.ts`中创建的代码：

```js
const client = new MongoClient();
client.connectWithUri("mongodb+srv://<username>:<password>
  @clustername.mongodb.net/test?retryWrites=true&w=
    majority&useNewUrlParser=true&useUnifiedTopology=true");
const db = client.database("getting-started-with-deno");
```

应该就是我们所需要的全部内容了，让我们开始吧！

记住，由于我们使用插件 API 连接到 MongoDB，而且它仍然不稳定，所以需要以下权限以及`--unstable`标志：

```js
$ deno run --allow-net --allow-write --allow-read --allow-plugin --allow-env --unstable src/index.ts
Application running at http://localhost:8080
```

现在，为了测试我们的`UserRepository`是否运行正常并且与数据库连接，让我们尝试注册并登录看看是否可行：

1.  向`/api/users/register`发送一个`POST`请求来注册我们的用户：

    ```js
    $ curl -X POST -d '{"username": "asantos00", "password": "testpw1" }' -H 'Content-Type: application/json' http://localhost:8080/api/users/register
    {"user":{"username":"asantos00","createdAt":"2020-11-01T23:21:58.442Z"}}
    ```

1.  现在，为了确保我们连接到永久存储，我们可以停止应用程序然后再次运行它，在尝试登录之前：

    ```js
    $ deno run --allow-net --allow-write --allow-read --allow-plugin --allow-env --unstable src/index.ts
    Application running at http://localhost:8080
    ```

1.  现在，让我们用刚才创建的同一个用户登录：

    ```js
    $ curl -X POST -d '{"username": "asantos00", "password": "testpw1" }' -H 'Content-Type: application/json' http://localhost:8080/api/login
    {"user":{"username":"asantos006"},"token":"eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJtdXNl dW1zIiwiZXhwIjoxNjA0MjczMDQ1LCJ1c2VyIjoiYXNhbnRvczAwNi J9.elY48It-DHse5sSszCAWuE2PzNkKiPsMIvif4v5klY1URq0togK 84wsbSskGAfe5UQsJScr4_0yxqnrxEG8viw"}
    ```

我们得到了响应！我们成功地将之前连接到内存数据库的应用程序连接到了一个真实的 MongoDB 数据库。如果你使用了 MongoDB，你可以在 Atlas 界面的**集合**菜单中查看那里创建的用户。

你注意到我们为了更改持久性机制并没有触及到任何业务或网络逻辑了吗？这证明了我们最初创建的层和抽象现在正在发挥作用，通过允许应用程序不同部分之间的解耦。

有了这些，我们完成了这一章节并把我们的用户迁移到了一个真实的数据库。我们也可以对其他模块做同样的事情，但那将会是几乎相同的工作，并且不会为你的学习体验增加太多。我想挑战你编写其他模块的逻辑，使其能够连接到 MongoDB。

如果你想要跳过这部分但是好奇它会是怎样的，那么去看看这本书的 GitHub 仓库吧。

# 总结

这一章节基本上已经涵盖了我们在逻辑方面对应用程序的封装。我们稍后会在第八章 *测试 - 单元和集成* 中添加测试以及我们所缺少的一个特性——对博物馆进行评分的能力。然而，这部分大多数已经完成。在其当前状态下，我们有一个应用程序，它的领域被划分为可以独立使用且彼此不依赖的模块。我们相信我们已经实现了一些既易于在代码中导航又可扩展的东西。

这一过程结束了不断重构和精炼架构、管理依赖项以及调整逻辑以确保代码尽可能解耦，同时尽可能容易地在未来进行更改。在完成所有这些工作时，我们设法创建了一个具有几个功能的应用程序，同时尝试绕过行业标准。

我们通过学习中间件函数开始了这一章，这是我们之前使用过，尽管我们还没有学习过它们的东西。我们理解了它们是如何工作的，以及它们如何被利用来在应用程序和路线中添加逻辑。为了更具体一点，我们进入了具体的例子，并以在应用程序中实现几个为例结束。在这里，我们添加了诸如基本日志记录和请求计时等常见功能。

然后，我们继续完成认证的旅程。在上一章中添加了用户和注册功能后，我们开始实现认证功能。我们依赖一个外部包来管理我们的 JWT 令牌，我们稍后用于我们的授权机制。在向用户提供令牌后，我们必须确保令牌有效，然后才允许用户访问应用程序。我们在博物馆路线上添加了一个认证路线，确保它只能被认证用户访问。再次使用中间件来检查令牌的有效性并在错误情况下回答请求。

我们通过向应用程序添加一个新功能来结束这一章：连接到真实数据库。在我们这样做之前，我们所有的应用程序模块都依赖于内存中的数据库。在这里，我们将其中一个模块，“用户”，移动到 MongoDB 实例。为了做到这一点，我们利用了之前创建的层来将业务逻辑与我们的持久化和交付机制分离。在这里，我们创建并实现了我们所谓的 MongoDB 存储库，确保应用程序运行顺利，但具有真正的持久化机制。我们为此示例使用了 MongoDB Atlas。

在下一章中，我们将向我们的网络应用程序添加一些内容，具体包括管理代码之外的秘密和配置的能力，这是一个众所周知的好实践。我们还将探索 Deno 在运行浏览器代码等方面的可能性，等等。下一章将结束这本书的这一部分；也就是说，构建应用程序的功能。让我们开始吧！
