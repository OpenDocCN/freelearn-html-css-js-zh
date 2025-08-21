# 第十三章：架构

现在您知道，Nest.js 基于与 Angular 相同的原则，因此将其结构与 Angular 类似是一个好主意。

在进入文件结构之前，我们将看到一些关于命名和如何结构化不同目录和文件的指南，以便使项目更易读和可维护。

我们将看一下两种不同类型项目的架构：

+   服务器应用程序

+   使用 `Angular universal` 与 Nest.js 和 Angular 6 创建更完整的应用程序

在本章结束时，您应该知道如何为服务器应用程序或具有客户端前端的完整应用程序结构化您的应用程序。 

# 命名约定的样式指南

在这部分，我们将看到可以使用的命名约定，以便具有更好的可维护性和可读性。对于每个装饰器，您应该使用带连字符的名称，后跟一个点和对应的装饰器或对象的名称。

## 控制器

控制器的命名应遵循以下原则：

***user.controller.ts***

```js
@Controller()
export class UserController { /* ... */ }

```

## 服务

服务的命名应遵循以下原则：

***user.service.ts***

```js
@Injectable()
export class UserService { /* ... */ }

```

## 模块

模块的命名应遵循以下原则：

***user.module.ts***

```js
@Module()
export class UserModule { /* ... */ }

```

## 中间件

中间件的命名应遵循以下原则：

***authentication.middleware.ts***

```js
@Injectable()
export class AuthenticationMiddleware { /* ... */ }

```

## 异常过滤器

异常过滤器的命名应遵循以下原则：

***forbidden.exception.ts***

```js
export class ForbiddenException { /* ... */ }

```

## 管道

管道的命名应遵循以下原则：

***validation.pipe.ts***

```js
@Injectable()
export class ValidationPipe { /* ... */ }

```

## 守卫

守卫的命名应遵循以下原则：

***roles.guard.ts***

```js
@Injectable()
export class RolesGuard { /* ... */ }

```

## 拦截器

拦截器的命名应遵循以下原则：

***logging.interceptor.ts***

```js
@Injectable()
export class LoggingInterceptor { /* ... */ }

```

## 自定义装饰器

自定义装饰器的命名应遵循以下原则：

***comment.decorator.ts***

```js
export const Comment: (data?: any, ...pipes: Array<PipeTransform<any>>) => {
    ParameterDecorator = createParamDecorator((data, req) => {
        return req.comment;
    }
};

```

## 网关

网关的命名应遵循以下原则：

***comment.gateway.ts***

```js
@WebSocketGateway()
export class CommentGateway {

```

## 适配器

适配器的命名应遵循以下原则：

***ws.adapter.ts***

```js
export class WsAdapter {

```

## 单元测试

单元测试的命名应遵循以下原则：

***user.service.spec.ts***

## 端到端测试

端到端测试的命名应遵循以下原则：

***user.e2e-spec.ts***

现在我们已经概述了 Nest.js 提供的工具，并制定了一些命名指南。我们现在可以进入下一部分了。

# 目录结构

拥有良好结构化的目录文件的项目非常重要，因为这样更易读、易懂且易于使用。

因此，让我们看看如何结构化我们的目录，以便更清晰。您将在以下示例中看到用于存储库的目录文件架构，该架构是使用前一节中描述的命名约定创建的。

## 服务器架构

对于服务器架构，您将看到一个用于存储库的建议架构，以便有清晰的目录。

### 完整概述

查看基本文件结构，不要深入细节：

```js
.
├── artillery/
├── scripts/
├── migrations/
├── src/
├── Dockerfile
├── README.md
├── docker-compose.yml
├── migrate.ts
├── nodemon.json
├── package-lock.json
├── package.json
├── tsconfig.json
├── tslint.json
└── yarn.lock

```

我们有四个文件夹用于存放服务器所需的所有文件：

+   `artillery` 目录，如果需要，可以包含所有用于测试 API 端点的场景。

+   `scripts` 目录将包含您在应用程序中需要使用的所有脚本。在我们的情况下，等待 `RabbitMQ` 使用的端口打开的脚本，以便 Nest.js 应用程序在启动之前等待。

+   `migrations` 目录存在是因为我们使用了 `sequelize`，并且编写了一些迁移文件，这些文件存储在该目录中。

+   `src` 目录，其中包含我们服务器应用的所有代码。

在存储库中，我们还有一个 `client` 目录。但在这种情况下，它仅用作 WebSocket 使用的示例。

### `src` 目录

`src`目录将包含所有应用程序模块、配置、网关等。让我们来看看这个目录：

```js
src
├── app.module.ts
├── main.cluster.ts
├── main.ts
├── gateways
│   ├── comment
│   └── user
├── modules
│   ├── authentication
│   ├── comment
│   ├── database
│   ├── entry
│   ├── keyword
│   └── user
└── shared
    ├── adapters
    ├── config
    ├── decorators
    ├── exceptions
    ├── filters
    ├── guards
    ├── interceptors
    ├── interfaces
    ├── middlewares
    ├── pipes
    └── transports

```

这个目录也必须被良好地构建。为此，我们创建了三个子目录，对应于所有放在`gateways`目录中的 Web 套接字网关。`modules`将包含应用程序所需的所有模块。最后，`shared`将包含所有共享内容，如其名称所示，与所有`adapters`、`config`文件和自定义装饰器的元素对应，这些元素可以在任何模块中使用而不属于特定的模块。

现在我们将深入研究模块目录。

#### 模块

您的应用程序的主要部分将被构建为一个模块。这个模块将包含许多不同的文件。让我们看看一个模块可以如何构建：

```js
src/modules
├── authentication
│   ├── authentication.controller.ts
│   ├── authentication.module.ts
│   ├── authentication.service.ts
│   ├── passports
│   │   └── jwt.strategy.ts
│   └── tests
│       ├── e2e
│       │   └── authentication.controller.e2e-spec.ts
│       └── unit
│           └── authentication.service.spec.ts
├── comment
│   ├── comment.controller.ts
│   ├── comment.entity.ts
│   ├── comment.module.ts
│   ├── comment.provider.ts
│   ├── comment.service.ts
│   ├── interfaces
│   │   ├── IComment.ts
│   │   ├── ICommentService.ts
│   │   └── index.ts
│   └── tests
│       ├── unit
│       │   └── comment.service.spec.ts
│       └── utilities.ts
├── database
│   ├── database-utilities.service.ts
│   ├── database.module.ts
│   └── database.provider.ts
├── entry
│   ├── commands
│   │   ├── handlers
│   │   │   ├── createEntry.handler.ts
│   │   │   ├── deleteEntry.handler.ts
│   │   │   ├── index.ts
│   │   │   └── updateEntry.handler.ts
│   │   └── impl
│   │       ├── createEntry.command.ts
│   │       ├── deleteEntry.command.ts
│   │       └── updateEntry.command.ts
│   ├── entry.controller.ts
│   ├── entry.entity.ts
│   ├── entry.model.ts
│   ├── entry.module.ts
│   ├── entry.provider.ts
│   ├── entry.service.ts
│   ├── interfaces
│   │   ├── IEntry.ts
│   │   ├── IEntryService.ts
│   │   └── index.ts
│   └── tests
│       ├── unit
│       │   └── entry.service.spec.ts
│       └── utilities.ts
├── keyword
│   ├── commands
│   │   ├── handlers
│   │   │   ├── index.ts
│   │   │   ├── linkKeywordEntry.handler.ts
│   │   │   └── unlinkKeywordEntry.handler.ts
│   │   └── impl
│   │       ├── linkKeywordEntry.command.ts
│   │       └── unlinkKeywordEntry.command.ts
│   ├── events
│   │   ├── handlers
│   │   │   ├── index.ts
│   │   │   └── updateKeywordLinks.handler.ts
│   │   └── impl
│   │       └── updateKeywordLinks.event.ts
│   ├── interfaces
│   │   ├── IKeyword.ts
│   │   ├── IKeywordService.ts
│   │   └── index.ts
│   ├── keyword.controller.ts
│   ├── keyword.entity.ts
│   ├── keyword.module.ts
│   ├── keyword.provider.ts
│   ├── keyword.sagas.ts
│   ├── keyword.service.ts
│   └── keywordEntry.entity.ts
└── user
    ├── interfaces
    │   ├── IUser.ts
    │   ├── IUserService.ts
    │   └── index.ts
    ├── requests
    │   └── create-user.request.ts
    ├── tests
    │   ├── e2e
    │   │   └── user.controller.e2e-spec.ts
    │   ├── unit
    │   │   └── user.service.spec.ts
    │   └── utilities.ts
    ├── user.controller.ts
    ├── user.entity.ts
    ├── user.module.ts
    ├── user.provider.ts
    └── user.service.ts

```

在我们的存储库中，有许多模块。其中一些还实现了`cqrs`，它与模块位于同一个目录，因为它涉及到模块并且是其一部分。`cqrs`部分分为`commands`和`events`目录。模块还可以定义一些接口，这些接口放在单独的`interfaces`目录中。单独的目录使我们能够更清晰地阅读和理解，而不必将许多不同的文件混在一起。当然，所有涉及模块的测试也包括在它们自己的`tests`目录中，并分为`unit`和`e2e`。

最后，定义模块本身的主要文件，包括可注入对象、控制器和实体，都在模块的根目录中。

在本节中，我们已经看到了如何以更清晰和更易读的方式构建我们的服务器应用程序的结构。现在您知道应该把所有模块放在哪里，以及如何构建一个模块，以及如果使用它们，应该把网关或共享文件放在哪里。

## Angular Universal 架构

存储库的 Angular Universal 部分是一个独立的应用程序，使用 Nest.js 服务器和 Angular 6。它将由两个主要目录组成：`e2e`用于端到端测试，以及包含服务器和客户端的`src`。

让我们首先看一下这个架构的概述：

```js
├── e2e/
├── src/
├── License
├── README.md
├── angular.json
├── package.json
├── tsconfig.json
├── tslint.json
├── udk.container.js
└── yarn.lock

```

### `src`目录

这个目录将包含`app`目录，以便使用模块化的 Angular 架构放置我们的客户端内容。此外，我们还将找到`environments`，它定义了我们是否处于生产模式，并导出常量。这个环境将被生产环境配置替换为生产模式，然后是`server`和`shared`目录。共享目录允许我们共享一些文件，例如接口，而服务器目录将包含所有服务器应用程序，就像我们在前一节中看到的那样。

但在这种情况下，服务器有些变化，现在看起来是这样的：

```js
├── main.ts
├── app.module.ts
├── environments
│   ├── environment.common.ts
│   ├── environment.prod.ts
│   └── environment.ts
└── modules
    ├── client
    │   ├── client.constants.ts
    │   ├── client.controller.ts
    │   ├── client.module.ts
    │   ├── client.providers.ts
    │   ├── interfaces
    │   │   └── angular-universal-options.interface.ts
    │   └── utils
    │       └── setup-universal.utils.ts
    └── heroes
        ├── heroes.controller.ts
        ├── heroes.module.ts
        ├── heroes.service.ts
        └── mock-heroes.ts

```

modules 目录将包含所有的 Nest.js 模块，就像我们在前一节中看到的那样。其中一个模块是`client`模块，将为 Universal 应用程序提供所有必需的资产，并设置初始化程序以设置引擎并提供一些 Angular 配置。

关于`environments`，这个目录将包含与 Angular 应用程序相关的所有配置路径。这个配置引用了在前一节项目的基础中看到的`angular.json`文件中配置的项目。

# 总结

这一章让您以更易理解、易读和易于处理的方式设置应用程序的架构。我们已经看到了如何为服务器应用程序定义架构目录，以及如何使用 Angular Universal 构建完整应用程序。通过这两个示例，您应该能够以更清晰的方式构建自己的项目。

下一章将展示如何在 Nest.js 中使用测试。
