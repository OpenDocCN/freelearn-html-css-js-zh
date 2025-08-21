# 第一章：介绍

每个 Web 开发人员都严重依赖于一个或多个 Web 框架（有时如果他们的服务有不同的要求，会使用更多），而公司将依赖于许多框架，但每个框架都有其优缺点。这些框架正是为开发人员提供一个框架，提供基本功能，任何 Web 框架必须提供这些功能，才能被认为是开发人员或公司在其技术栈中使用的一个好选择。在本书中，我们将讨论您期望在像 Nest 这样的先进框架中看到的框架的许多部分。这些包括：

1.  依赖注入

1.  认证

1.  ORM

1.  REST API

1.  Websockets

1.  微服务

1.  路由

1.  Nest 特定工具的解释

1.  OpenApi（Swagger）文档

1.  命令查询责任分离（CQRS）

1.  测试

1.  使用 Universal 和 Angular 进行服务器端渲染。

Nest 提供了更多这些功能，因为它是建立在 Node.js Express 服务器之上的现代 Web 框架。通过利用现代 ES6 JavaScript 的弹性和 TypeScript 在编译时强制类型安全，Nest 在设计和构建服务器端应用程序时将可扩展的 Node.js 服务器提升到一个全新的水平。Nest 将三种不同的技术结合成一个成功的组合，允许高度可测试、可扩展、松散耦合和可维护的应用程序。这些技术包括：

1.  面向对象编程（OOP）：一个围绕对象而不是动作和可重用性而不是利基功能构建的模型。

1.  函数式编程（FP）：设计不依赖于全局状态的确定功能，即函数 f(x)对于一些不变的参数每次返回相同的结果。

1.  函数式响应式编程（FRP）：是上述 FP 和响应式编程的扩展。函数式响应式编程在其核心是考虑时间流的函数式编程。它在 UI、模拟、机器人和其他应用程序中非常有用，其中特定时间段的确切答案可能与另一个时间段的答案不同。

# 讨论的主题

以下每个主题将在接下来的章节中详细讨论。

# Nest CLI

在 Nest 的第 5 版中，有一个 CLI 可以允许通过命令行生成项目和文件。可以通过以下命令全局安装 CLI：

```js
npm install -g @nestjs/cli

```

或者通过 Docker：

```js
docker pull nestjs/cli:[version]

```

可以使用以下命令生成新的 Nest 项目：

```js
nest new [project-name]

```

此过程将从[typescript-starter](https://github.com/nestjs/typescript-starter)创建项目，并将要求输入`name`，`description`，`version`（默认为 0.0.0）和`author`（这将是您的名字）。完成此过程后，您将拥有一个完全设置好的 Nest 项目，并且依赖项已安装在您的`node_modules`文件夹中。`new`命令还将询问您想要使用哪种包管理器，就像`yarn`或`npm`一样。Nest 在创建过程中为您提供了这个选择。

CLI 中最常用的命令将是`generate`（g）命令，这将允许您创建 Nest 支持的新的`controllers`，`modules`，`servies`或任何其他组件。可用组件的列表如下：

1.  `class`（cl）

1.  `controller`（co）

1.  `decorator`（d）

1.  `exception`（e）

1.  `filter`（f）

1.  `gateway`（ga）

1.  `guard`（gu）

1.  `interceptor`（i）

1.  `middleware`（mi）

1.  `module`（mo）

1.  `pipe`（pi）

1.  `provider`（pr）

1.  `service`（s）

请注意，括号中的字符串是该特定命令的别名。这意味着您可以输入：

```js
nest generate service [service-name]

```

在控制台中，您可以输入：

```js
nest g s [service-name]

```

最后，Nest CLI 提供了`info`（i）命令来显示关于您的项目的信息。此命令将输出类似以下内容的信息：

```js
[System Information]
OS Version     : macOS High Sierra
NodeJS Version : v8.9.0
YARN Version    : 1.5.1
[Nest Information]
microservices version : 5.0.0
websockets version    : 5.0.0
testing version       : 5.0.0
common version        : 5.0.0
core version          : 5.0.0

```

# 依赖注入

依赖注入是一种技术，它通过将依赖对象（如服务）注入到组件的构造函数中，从而将依赖对象（如模块或组件）提供给依赖对象。下面是来自 sequelize 章节的一个示例。在这里，我们将`UserRespository`服务注入到`UserService`的构造函数中，从而在`UserService`组件内部提供对用户数据库存储库的访问。

```js
@Injectable()
export class UserService implements IUserService {
    constructor(@Inject('UserRepository') private readonly UserRepository: typeof User) { }
    ...
}

```

反过来，这个`UsersService`将被注入到`src/users/users.controller.ts`文件中的`UsersController`中，这将为指向该控制器的路由提供对`UsersService`的访问。更多关于路由和依赖注入的内容将在后面的章节中讨论。

# 认证

认证是开发中最重要的方面之一。作为开发人员，我们始终希望确保用户只能访问他们有权限访问的资源。认证可以采用多种形式，从展示您的驾驶执照或护照到为登录门户提供用户名和密码。近年来，这些认证方法已经扩展到变得更加复杂，但我们仍然需要相同的服务器端逻辑，以确保这些经过认证的用户始终是他们所说的那个人，并保持这种认证，这样他们就不需要为每次对 REST API 或 Websocket 的调用重新进行认证，因为那将提供非常糟糕的用户体验。选择的库恰好也被命名为 Passport，并且在 Node.js 生态系统中非常知名和使用。在 Nest 中集成时，它使用 JWT（JSON Web Token）策略。Passport 是一个中间件，HTTP 调用在到达控制器端点之前会经过它。这是为示例项目编写的`AuthenticationMiddleware`，它扩展了`NestMiddleware`，根据请求负载中的电子邮件对每个用户进行认证。

```js
@Injectable()  
export class AuthenticationMiddleware implements NestMiddleware {  
   constructor(private userService: UserService) { }  

   async resolve(strategy: string): Promise<ExpressMiddleware> {  
       return async (req, res, next) => {  
           return passport.authenticate(strategy, async (/*...*/args: any[]) => {  
               const [, payload, err] = args;  
                if (err) {  
                    return res.status(HttpStatus.BAD_REQUEST).send('Unable to authenticate the user.');  
                }  

               const user = await this.userService.findOne({
                    where: { email: payload.email }
               });  
                req.user = user;  
                return next();  
            })(req, res, next);  
        };  
    }  
}

```

Nest 还实现了守卫，它们与其他提供者一样使用`@Injectable()`进行装饰。守卫基于经过认证的用户所拥有的访问权限来限制某些端点。守卫将在认证章节中进一步讨论。

# ORM

ORM 是对象关系映射，是处理服务器和数据库之间通信时最重要的概念之一。ORM 提供了内存中对象（如`User`或`Comment`这样的定义类）与数据库中的关系表之间的映射。这使您可以创建一个数据传输对象，它知道如何将存储在内存中的对象写入数据库，并从 SQL 或其他查询语言中读取结果，再次存入内存。在本书中，我们将讨论三种不同的 ORM：两种关系型数据库和一种 NoSQL 数据库。TypeORM 是 Node.js 中最成熟和最流行的 ORM 之一，因此具有非常广泛和完善的功能集。它也是 Nest 提供自己的包之一：`@nestjs/typeorm`。它非常强大，并支持许多数据库，如 MySQL、PostgreSQL、MariaDB、SQLite、MS SQL Server、Oracle 和 WebSQL。除了 TypeORM，Sequelize 也是另一个用于关系数据的 ORM。

如果 TypeORM 是最受欢迎的 ORM 之一，那么 Sequelize 就是 Node.js 世界中最受欢迎的 ORM。它是用纯 JavaScript 编写的，但通过`sequelize-typescript`和`@types/sequelize`包具有 TypeScript 绑定。Sequelize 拥有强大的事务支持、关系、读取复制和许多其他功能。本书涵盖的最后一个 ORM 是处理非关系型或 NoSQL 数据库的 ORM。包`mongoose`处理了 MongoDB 和 JavaScript 之间的对象关系。实际的映射比与关系数据库更接近，因为 MongoDB 以 JSON 格式存储其数据，JSON 代表 JavaScript 对象表示法。Mongoose 也是具有`@nestjs/mongoose`包的包之一，并提供通过查询链接查询数据库的能力。

# REST API

REST 是创建 API 的主要设计范式之一。它代表着表现状态转移，并使用 JSON 作为传输格式，这与 Nest 存储对象的方式一致，因此它是用于消费和返回 HTTP 调用的自然选择。REST API 是本书讨论的许多技术的组合。它们以一定的方式组合在一起；客户端向服务器发起 HTTP 调用。服务器将根据 URL 和 HTTP 动词路由调用到正确的控制器，可选择性地通过一个或多个中间件传递到控制器之前。控制器然后将其交给服务进行处理，这可能包括通过 ORM 与数据库通信。如果一切顺利，服务器将向客户端返回一个 OK 响应，如果客户端请求资源（GET 请求），则可能包含一个可选的主体，或者如果是 POST/PUT/DELETE，则只返回一个 200/201 HTTP OK，而没有响应主体。

# WebSockets

WebSockets 是连接到服务器并发送/接收数据的另一种方式。使用 WebSockets，客户端将连接到服务器，然后订阅特定的频道。然后客户端可以将数据推送到已订阅的频道。服务器将接收这些数据，然后将其广播给订阅了特定频道的每个客户端。这允许多个客户端都实时接收更新，而无需手动进行 API 调用，可能会通过 GET 请求向服务器发送大量请求。大多数聊天应用程序使用 WebSockets 来实现实时通信，群组消息中的每个成员发送消息后，所有成员都会立即收到消息。Websockets 允许更多地以流式传输数据的方式来传输数据，而不是传统的请求-响应 API，因为 Websockets 会在接收到数据时广播数据。

# 微服务

微服务允许 Nest 应用程序以一组松散耦合的服务的形式进行结构化。在 Nest 中，微服务略有不同，因为它们是使用除 HTTP 之外的不同传输层的应用程序。这一层可以是 TCP 或 Redis pub/sub 等。Nest 支持 TCP 和 Redis，尽管如果您使用其他传输层，可以通过使用`CustomTransportStrategy`接口来实现。微服务很棒，因为它们允许团队在全局项目中独立于其他团队的微服务进行工作，并对服务进行更改，而不会影响项目的其他部分，因为它是松散耦合的。这允许持续交付和持续集成，而不受其他团队微服务的影响。

# GraphQL

正如我们在上面看到的，REST 是设计 API 时的一种范式，但现在有一种新的方式来考虑创建和使用 API：GraphQL。使用 GraphQL，每个资源都不再有自己指向它的 URL，而是 URL 将接受一个带有 JSON 对象的查询参数。这个 JSON 对象定义了要返回的数据的类型和格式。Nest 通过`@nestjs/graphql`包提供了这方面的功能。这将在项目中包括`GraphQLModule`，它是 Apollo 服务器的包装器。GraphQL 是一个可以写一整本书的主题，所以我们在本书中不再深入讨论它。

# 路由

路由是讨论 Web 框架的核心原则之一。客户端需要知道如何访问服务器的端点。这些端点中的每一个描述了如何检索/创建/操作存储在服务器上的数据。描述 API 端点的每个`Component`必须具有一个`@Controller('prefix')`装饰器，用于描述此组件端点集的 API 前缀。

```js
@Controller('hello')
export class HelloWorldController {
  @Get(‘world’)
  printHelloWorld() {
    return ‘Hello World’;
  }
}

```

上述控制器是`GET /hello/world`的 API 端点，将返回一个带有`Hello World`的`HTTP 200 OK`。这将在路由章节中进一步讨论，您将了解如何使用 URL 参数、查询参数和请求对象。

# Nest 特定工具

Nest 提供了一组特定于 Nest.js 的工具，可以在整个应用程序中使用，帮助编写可重用的代码并遵循 SOLID 原则。这些装饰器将在后续的每一章中使用，因为它们定义了特定的功能：

1.  @Module：项目中可重用代码的定义，它接受以下参数来定义其行为。⋅⋅ *导入：这些是包含在此模块中使用的组件的模块。⋅⋅*导出：这些是将在其他模块中使用的组件，导入此模块的模块。⋅⋅ *组件：这些组件将可供至少通过 Nest 注入器共享此模块。⋅⋅*控制器：在此模块中创建的控制器，这些控制器将根据定义的路由定义 API 端点。

1.  @Injectable：Nest 中几乎所有东西都是可以通过构造函数注入的提供者。提供者使用`@Injectable()`进行注释。.. *中间件：在请求传递到路由处理程序之前运行的函数。在本章中，我们将讨论中间件、异步中间件和功能中间件之间的区别。..*拦截器：类似于中间件，它们在方法执行前后绑定额外的逻辑，并且可以转换或完全覆盖函数。拦截器受面向方面的编程（AOP）的启发。.. *管道：类似于拦截器功能的一部分，管道将输入数据转换为所需的输出。..*守卫：更智能、更专业的中间件，守卫的唯一目的是确定请求是否应该由路由处理程序处理。..*Catch：告诉`ExceptionFilter`要查找的异常，然后将数据绑定到它。

1.  @Catch：将元数据绑定到异常过滤器，并告诉 Nest 过滤器仅寻找`@Catch`中列出的异常。

注意：在 Nest 版本 4 中，上面列出的`@Injectable()`下的不是所有东西都使用`@Injectable()`装饰器。组件、中间件、拦截器、管道和守卫各自都有自己的装饰器。在 Nest 版本 5 中，这些都已合并为`@Injectable()`，以减少 Nest 和 Angular 之间的差异。

# OpenAPI（Swagger）

在编写 Nest 服务器时，文档非常重要，特别是在创建将被其他人使用的 API 时，否则最终将使用 API 的客户端的开发人员不知道该发送什么或者他们会得到什么。其中最流行的文档引擎之一是 Swagger。与其他文档引擎一样，Nest 提供了专门用于 OpenAPI（Swagger）规范的模块`@nestjs/swagger`。该模块提供装饰器来描述 API 的输入/输出和端点。然后可以通过服务器上的端点访问此文档。

# 命令查询责任分离（CQRS）

命令查询责任分离（CQRS）是每个方法应该是执行操作（命令）或请求数据（查询）的想法，但不能两者兼而有之。在我们示例应用程序的上下文中，我们不会在端点的控制器中直接使用数据库访问代码，而是创建一个组件（数据库服务），该组件具有诸如`getAllUsers()`的方法，该方法将返回控制器服务可以调用的所有用户，从而将问题和答案分离到不同的组件中。

# 测试

测试您的 Nest 服务器将是至关重要的，以便一旦部署，就不会出现意外问题，并且一切都能顺利运行。在本书中，您将了解两种不同类型的测试：单元测试和 E2E 测试（端到端测试）。单元测试是测试小片段或代码块的艺术，这可能是测试单个函数或为`Controller`、`Interceptor`或任何其他`Injectable`编写测试。有许多流行的单元测试框架，`Jasmine`和`Jest`是其中两个流行的框架。Nest 提供了专门的包，特别是`@nestjs/testing`，用于在`*.spec.ts`和`*.test.ts`类中编写单元测试。

E2E 测试是常用的另一种测试形式，与单元测试不同之处在于它测试的是整个功能，而不是单个函数或组件，这就是所谓的端到端测试的由来。最终，应用程序会变得如此庞大，以至于很难测试每一行代码和端点。在这种情况下，您可以使用 E2E 测试来测试应用程序从开始到结束，以确保一切顺利进行。对于 E2E 测试，Nest 应用程序可以再次使用`Jest`库来模拟组件。除了`Jest`，您还可以使用`supertest`库来模拟 HTTP 请求。

测试是编写应用程序的非常重要的一部分，不应被忽视。无论您最终使用什么语言或框架，这都是一个相关的章节。大多数大型开发公司都有专门的团队负责为推送到生产应用程序的代码编写测试，这些团队被称为 QA 开发人员。

# 使用 Angular Universal 进行服务器端渲染

Angular 是一个客户端应用程序开发框架，而 Angular Universal 是一种技术，允许我们的 Nest 服务器预渲染网页并将其提供给客户端，这有许多好处，将在“使用 Angular Universal 进行服务器端渲染”章节中讨论。Nest 和 Angular 非常搭配，因为它们都使用 TypeScript 和 Node.js。许多可以在 Nest 服务器中使用的包也可以在 Angular 应用程序中使用，因为它们都编译为 JavaScript。

# 总结

在本书中，您将更详细地了解上述每个主题，不断构建在先前概念的基础上。Nest 提供了一个清晰、组织良好的框架，以简单而高效的方式实现每个概念，这是因为框架的模块化设计在所有模块中都是一致的。
