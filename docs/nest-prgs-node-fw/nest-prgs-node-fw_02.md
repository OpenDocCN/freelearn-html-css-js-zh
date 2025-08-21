# 第二章：概述

在本章中，我们将概述 Nest.js，并查看构建 Nest.js 应用程序所需的核心概念。

# 控制器

Nest 中的控制器负责处理传入的请求并向客户端返回响应。Nest 将传入的请求路由到控制器类中的处理程序函数。我们使用`@Controller()`装饰器来创建控制器类。

```js
import { Controller, Get } from '@nestjs/common';

@Controller('entries')
export class EntryController {
    @Get()
    index(): Entry[] {
        const entries: Entry[] = this.entriesService.findAll();
        return entries;
    }

```

我们将在**路由和请求处理**章节中详细讨论路由和处理请求的细节。

# 提供者

Nest 中的提供者用于创建服务、工厂、助手等，这些可以被注入到控制器和其他提供者中，使用 Nest 内置的依赖注入。`@Injectable()`装饰器用于创建提供者类。

例如，我们博客应用程序中的`AuthenticationService`是一个提供者，它注入并使用`UsersService`组件。

```js
@Injectable()
export class AuthenticationService {
    constructor(private readonly userService: UserService) {}

    async validateUser(payload: {
        email: string;
        password: string;
    }): Promise<boolean> {
        const user = await this.userService.findOne({
            where: { email: payload.email }
        });
        return !!user;
    }
}

```

我们将在**依赖注入**章节中更多地讨论依赖注入。

# 模块

Nest.js 应用程序被组织成模块。如果您熟悉 Angular 中的模块，那么 Nest 使用的模块语法将看起来非常熟悉。

每个 Nest.js 应用程序都将有一个**根模块**。在一个小应用程序中，这可能是唯一的模块。在一个较大的应用程序中，将应用程序组织成多个模块是有意义的，这些模块将您的代码分割成功能和相关功能。

Nest.js 中的模块是带有`@Module()`装饰器的类。`@Module()`装饰器接受一个描述模块的单个对象，使用以下属性。

| 属性 | 描述 |
| --- | --- |
| `components` | 要实例化的组件，可以在此模块中共享，并导出以供其他模块使用 |
| `controllers` | 由此模块创建的控制器 |
| `imports` | 要导入的模块列表，这些模块导出了此模块中需要的组件 |
| `exports` | 从此模块导出的组件列表，可供其他模块使用 |

在我们的示例应用程序中，根模块名为`AppModule`，应用程序分为多个子模块，这些子模块处理应用程序的主要部分，如身份验证、评论、数据库访问、博客条目和用户。

```js
@Module({
    components: [],
    controllers: [],
    imports: [
        DatabaseModule,
        AuthenticationModule.forRoot('jwt'),
        UserModule,
        EntryModule,
        CommentModule,
        UserGatewayModule,
        CommentGatewayModule
    ],
    exports: [],
})
export class AppModule implements NestModule {}

```

AppModule 导入应用程序所需的模块。我们的应用程序中的根模块不需要有任何`exports`，因为没有其他模块导入它。

根模块也没有任何`components`或`controllers`，因为这些都是在它们相关的子模块中组织的。例如，`EntryModule`包括与博客条目相关的`components`和`controllers`。

```js
@Module({
    components: [entryProvider, EntryService],
    controllers: [EntryController],
    imports: [],
    exports: [EntryService],
})
export class EntryModule implements NestModule {}

```

在 Nest.js 中，模块默认是单例的。这意味着您可以在模块之间共享导出组件的相同实例，例如上面的`EntryService`，而无需任何努力。

# 引导

每个 Nest.js 应用程序都需要进行引导。这是通过使用`NestFactory`创建根模块并调用`listen()`方法来完成的。

在我们的示例应用程序中，入口点是`main.ts`，我们使用 async/await 模式创建`AppModule`并调用`listen()`：

```js
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();

```

# 中间件

Nest.js 中间件可以是一个函数，也可以是一个使用`@Injectable()`装饰器实现`NestMiddleware`接口的类。中间件在路由处理程序**之前**被调用。这些函数可以访问**请求**和**响应**对象，并且可以对请求和响应对象进行更改。

可以为路由配置一个或多个中间件函数，并且中间件函数可以选择将执行传递给堆栈上的下一个中间件函数，或者结束请求-响应周期。

如果中间件函数没有结束请求-响应周期，它必须调用`next()`将控制权传递给堆栈上的下一个中间件函数，或者如果它是堆栈上的最后一个函数，则传递给请求处理程序。未能这样做将使请求挂起。

例如，在我们的博客应用程序中，`AuthenticationMiddleware`负责对访问博客的用户进行身份验证。

```js
import {
    MiddlewareFunction,
    HttpStatus,
    Injectable,
    NestMiddleware
} from '@nestjs/common';
import * as passport from 'passport';
import { UserService } from '../../modules/user/user.service';

@Injectable()
export class AuthenticationMiddleware implements NestMiddleware {
    constructor(private userService: UserService) {}

    async resolve(strategy: string): Promise<MiddlewareFunction> {
        return async (req, res, next) => {
            return passport.authenticate(strategy, async (...args: any[]) => {
                const [, payload, err] = args;
                if (err) {
                    return res
                        .status(HttpStatus.BAD_REQUEST)
                        .send('Unable to authenticate the user.');
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

如果身份验证失败，将向客户端发送 400 响应。如果身份验证成功，那么将调用`next()`，并且请求将继续通过中间件堆栈，直到到达请求处理程序。

中间件是在 Nest.js 模块的`configure()`函数中配置在路由上的。例如，上面的`AuthenticationMiddle`在`AppModule`中配置如下所示。

```js
@Module({
    imports: [
        DatabaseModule,
        AuthenticationModule.forRoot('jwt'),
        UserModule,
        EntryModule,
        CommentModule,
        UserGatewayModule,
        CommentGatewayModule,
        KeywordModule
    ],
    controllers: [],
    providers: []
})
export class AppModule implements NestModule {
    public configure(consumer: MiddlewareConsumer) {
        const userControllerAuthenticatedRoutes = [
            { path: '/users', method: RequestMethod.GET },
            { path: '/users/:id', method: RequestMethod.GET },
            { path: '/users/:id', method: RequestMethod.PUT },
            { path: '/users/:id', method: RequestMethod.DELETE }
        ];

        consumer
            .apply(AuthenticationMiddleware)
            .with(strategy)
            .forRoutes(
                ...userControllerAuthenticatedRoutes,
                EntryController,
                CommentController
            );
    }
}

```

您可以将中间件应用到控制器上的所有路由，就像`EntryController`和`CommentController`中所做的那样。您还可以根据路径将中间件应用到特定路由上，就像从`UserController`中的子集路由中所做的那样。

# 守卫

守卫是用`@Injectable()`装饰器修饰并实现`CanActivate`接口的类。守卫负责确定请求是否应该由路由处理程序或路由处理。守卫在每个中间件之后执行，但在管道之前执行。与中间件不同，守卫可以访问`ExecutionContext`对象，因此它们确切地知道将要评估的内容。

在我们的博客应用程序中，我们在`UserController`中使用`CheckLoggedInUserGuard`，只允许用户访问和访问自己的用户信息。

```js
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
import { Observable } from 'rxjs';

@Injectable()
export class CheckLoggedInUserGuard implements CanActivate {
    canActivate(
        context: ExecutionContext
    ): boolean | Promise<boolean> | Observable<boolean> {
        const req = context.switchToHttp().getRequest();
        return Number(req.params.userId) === req.user.id;
    }
}

```

`@UseGuards`装饰器用于将守卫应用到路由上。这个装饰器可以用在控制器类上，将守卫应用到该控制器的所有路由上，也可以用在控制器中的单个路由处理程序上，就像`UserController`中所示的那样：

```js
@Controller('users')
export class UserController {
    constructor(private readonly userService: UserService) { }

    @Get(':userId')
    @UseGuards(CheckLoggedInUserGuard)
    show(@Param('userId') userId: number) {
        const user: User = this.userService.findById(userId);
        return user;
    }

```

# 总结

在本章中，我们介绍了 Nest.js 控制器、提供者、模块、引导和中间件。在下一章中，我们将介绍 Nest.js 身份验证。
