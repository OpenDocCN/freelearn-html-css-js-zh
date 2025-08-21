# 第三章：Nest.js 认证

Nest.js，在版本 5 中，`@nestjs/passport` 软件包允许您实现所需的认证策略。当然，您也可以使用 `passport` 手动执行此操作。

在本章中，您将看到如何通过将其集成到 Nest.js 项目中来使用 passport。我们还将介绍策略是什么，以及如何配置策略以与 passport 一起使用。

我们还将使用认证中间件来管理限制访问，并查看守卫如何在用户访问处理程序之前检查数据。此外，我们将展示如何使用 Nest.js 提供的 passport 软件包，以涵盖两种可能性。

作为示例，我们将使用以下存储库文件：

+   `/src/authentication`

+   `/src/user`

+   `/shared/middlewares`

+   `/shared/guards`

# Passport

Passport 是一个众所周知的流行且灵活的库。事实上，passport 是一种灵活的中间件，可以完全自定义。Passport 允许不同的方式来验证用户，如以下方式：

+   `本地策略` 允许您仅使用自己的数据 `email` 和 `password` 来验证用户，在大多数情况下。

+   `jwt 策略` 允许您通过提供令牌并使用 `jsonwebtoken` 验证此令牌来验证用户。这种策略被广泛使用。

一些策略使用社交网络或 Google 来验证用户的配置文件，如 `googleOAuth`、`Facebook`，甚至 `Twitter`。

为了使用 passport，您必须安装以下软件包：`npm i passport`。在了解如何实现认证之前，您必须实现 `userService` 和 `userModel`。

# 手动实现

在本节中，我们将使用 passport 手动实现认证，而不使用 Nest.js 软件包。

## 实施

为了配置 passport，需要配置三件事：

+   认证策略

+   应用程序中间件

+   可选的会话

Passport 使用策略来验证请求，并且凭据的验证被委托给一些请求中的策略。

在使用 passport 之前，您必须配置策略，在这种情况下，我们将使用 `passport-jwt` 策略。

在任何其他操作之前，您必须安装适当的软件包：

+   `npm i passport-jwt @types/passport-jwt`

+   `npm i jsonwebtoken @types/jsonwebtoken`

### 认证模块

为了有一个可工作的示例，您必须实现一些模块，我们将从 `AuthenticationModule` 开始。`AuthenticationModule` 将使用 jwt 策略配置策略。为了配置策略，我们将扩展 `passport-jwt` 软件包提供的 `Strategy` 类。

#### 策略

这是一个扩展 `Strategy` 类以配置并在 passport 中使用的策略的示例。

```js
@Injectable()  
export default class JwtStrategy extends Strategy {  
   constructor(private readonly authenticationService: AuthenticationService) {  
       super({  
            jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),  
            passReqToCallback: true,  
            secretOrKey: 'secret'  
        }, async (req, payload, next) => {  
            return await this.verify(req, payload, next);  
        });  
        passport.use(this);  
    }  

   public async verify(req, payload, done) {  
       const isValid = await this.authenticationService.validateUser(payload);  
        if (!isValid) {  
           return done('Unauthorized', null);  
        } else {  
           return done(null, payload);  
        }  
   }  
}

```

构造函数允许您向扩展的 `Strategy` 类传递一些配置参数。在这种情况下，我们只使用了三个参数：

+   `jwtFromRequest` 选项接受一个函数，以从请求中提取令牌。在我们的情况下，我们使用 `passport-jwt` 软件包提供的 `ExtractJwt.fromAuthHeaderAsBearerToken()` 函数。此函数将从请求的标头中提取令牌，使用 `Authorization` 标头，并选择跟随 `bearer` 词的令牌。

+   `passReqToCallback` 参数接受一个布尔值，以便告诉您是否要在稍后看到的验证方法中获取 `req`。

+   `secretOrKey` 参数接受一个字符串或缓冲区，以验证令牌签名。

还有其他参数可用于配置策略，但为了实现我们的认证，我们不需要它们。

此外，在传递不同的先前参数之后，我们传递了一个名为`verify`的回调函数。这个函数是异步的，其目的是验证传递的令牌以及从令牌获得的载荷是否有效。此函数执行我们的`verify`方法，该方法调用`authenticationService`以验证具有载荷作为参数的用户。

如果用户有效，我们返回载荷，否则我们返回一个错误以指示载荷无效。

#### 身份验证服务

如前一节所示，为了验证从令牌中获取的载荷，调用`AuthenticationService`提供的`validateUser`方法。

事实上，该服务将实现另一种方法，以为已登录的用户生成令牌。该服务可以按照以下示例实现。

```js
@Injectable()  
export class AuthenticationService {  
   constructor(private readonly userService: UserService) { }  

   createToken(email: string, ttl?: number) {  
        const expiresIn = ttl || 60 * 60;  
        const secretOrKey = 'secret';  
        const user = { email };  
        const token = jwt.sign(user, secretOrKey, { expiresIn });  
        return {  
            expires_in: expiresIn,  
            access_token: token,  
        };  
   }  

   async validateUser(payload: { email: string; password: string }): Promise<boolean> {  
        const user = await this.userService.findOne({  
            where: { email: payload.email }  
        });  
        return !!user;  
   }  
}

```

服务注入了`UserService`，以便使用传递给`validateUser`方法的载荷来查找用户。如果载荷中的电子邮件允许您找到用户，并且该用户具有有效的令牌，她可以继续身份验证过程。

为了为尝试登录的用户提供令牌，实现`createToken`方法，该方法以`email`和可选的`ttl`作为参数。`ttl`（生存时间）将配置令牌在一段时间内有效。`ttl`的值以秒为单位表示，我们在`60 * 60`中定义了默认值，这意味着 1 小时。

#### 身份验证控制器

为了处理用户的身份验证，实现控制器并为登录端点提供处理程序。

```js
@Controller()  
export class AuthenticationController {  
   constructor(  
        private readonly authenticationService: AuthenticationService,  
        private readonly userService: UserService) {}  

   @Post('login')  
   @HttpCode(HttpStatus.OK)  
   public async login(@Body() body: any, @Res() res): Promise<any> {  
       if (!body.email || !body.password) {  
           return res.status(HttpStatus.BAD_REQUEST).send('Missing email or password.');  
       }  

       const user = await this.userService.findOne({  
           where: {  
               email: body.email,  
                password: crypto.createHmac('sha256', body.password).digest('hex')  
           }  
       });  
       if (!user) {  
           return res.status(HttpStatus.NOT_FOUND).send('No user found with this email and password.');  
       }  

       const result = this.authenticationService.createToken(user.email);  
       return res.json(result);  
    }  
}

```

控制器提供了登录处理程序，可通过在`POST /login`路由上调用来访问。该方法的目的是验证用户提供的凭据，以便在数据库中找到他。如果找到用户，则创建适当的令牌，并将其作为响应返回，其中`expiresIn`值对应于我们先前定义的`ttl`。否则，请求将被拒绝。

#### 模块

我们现在已经定义了我们的服务和策略，以配置 passport 并提供一些方法来创建令牌和验证载荷。让我们定义`AuthenticationModule`，它类似于以下示例。

```js
@Module({})  
export class AuthenticationModule {  
   static forRoot(strategy?: 'jwt' | 'OAuth' | 'Facebook'): DynamicModule {  
       strategy = strategy ? strategy : 'jwt';  
        const strategyProvider = {  
            provide: 'Strategy',  
            useFactory: async (authenticationService: AuthenticationService) => {  
                const Strategy = (await import (`./passports/${strategy}.strategy`)).default;  
                return new Strategy(authenticationService);  
            },  
            inject: [AuthenticationService]  
       };  
        return {  
            module: AuthenticationModule,  
            imports: [UserModule],  
            controllers: [AuthenticationController],  
            providers: [AuthenticationService, strategyProvider],  
            exports: [strategyProvider]  
        };  
    }  
}

```

如您所见，该模块不是作为普通模块定义的，因此在`@Module()`装饰器中没有定义组件或控制器。事实上，该模块是一个动态模块。为了提供多种策略，我们可以在类上实现一个静态方法，以便在另一个模块中导入时调用它。这个`forRoot`方法以您想要使用的策略的名称作为参数，并将创建一个`strategyProvider`，以便添加到返回模块的组件列表中。该提供程序将实例化策略并将`AuthenticationService`作为依赖项提供。

让我们继续创建一些需要保护的东西，比如`UserModule`。

### 用户模块

`UserModule`提供了一个服务、一个控制器和一个模型（请参阅 Sequelize 章节中的 User 模型）。我们在`UserService`中创建了一些方法，以便操作有关用户的数据。这些方法在`UserController`中使用，以向 API 的用户提供一些功能。

所有功能都不能被用户使用，或者在返回的数据中受到限制。

#### 用户服务

让我们来看一个`UserService`的例子和一些方法，以便访问和操作数据。本部分描述的所有方法将在控制器中使用，其中一些受身份验证限制。

```js
@Injectable()
export class UserService() {
    // The SequelizeInstance come from the DatabaseModule have a look to the Sequelize chapter
    constructor(@Inject('UserRepository') private readonly UserRepository: typeof User,
                @Inject('SequelizeInstance') private readonly sequelizeInstance) { }

    /* ... */
}

```

服务注入了我们在 Sequelize 章节中描述的`UserRepository`，以便访问模型和数据库中的数据存储。我们还注入了在 Sequelize 章节中描述的`SequelizeInstance`，以便使用事务。

`UserService`实现了`findOne`方法，以在`options`参数中传递条件查找用户。`options`参数可以如下所示：

```js
{
    where: {
        email: 'some@email.test',
        firstName: 'someFirstName'
    }
}

```

使用这些条件，我们可以找到相应的用户。该方法将只返回一个结果。

```js
@Injectable()
export class UserService() {
    /* ... */

    public async findOne(options?: object): Promise<User | null> {  
        return await this.UserRepository.findOne<User>(options);  
    }

    /* ... */
}

```

让我们实现`findById`方法，该方法以 ID 作为参数，以查找唯一的用户。

```js
@Injectable()
export class UserService() {
    /* ... */

    public async findById(id: number): Promise<User | null> {  
        return await this.UserRepository.findById<User>(id);  
    }  

    /* ... */
}

```

然后我们需要一种方法，在数据库中创建一个新用户，传递符合`IUser`接口的用户。正如您所看到的，该方法使用`this.sequelizeInstance.transaction`事务，以避免在一切完成之前读取数据。该方法将参数传递给`create`函数，该函数是`returning`，以获取已创建的用户实例。

```js
@Injectable()
export class UserService() {
    /* ... */

    public async create(user: IUser): Promise<User> {  
        return await this.sequelizeInstance.transaction(async transaction => {  
            return await this.UserRepository.create<User>(user, {  
                returning: true,  
                transaction,  
            });  
        });  
    }  

    /* ... */
}

```

当然，如果您可以创建用户，您也需要通过以下方法更新用户，遵循`IUser`接口。这个方法也将返回已更新的用户实例。

```js
@Injectable()
export class UserService() {
    /* ... */

    public async update(id: number, newValue: IUser): Promise<User | null> {  
        return await this.sequelizeInstance.transaction(async transaction => {  
            let user = await this.UserRepository.findById<User>(id, { transaction });  
            if (!user) throw new Error('The user was not found.');  

            user = this._assign(user, newValue);  
            return await user.save({  
                returning: true,  
                transaction,  
            });  
        });  
    }  

    /* ... */
}

```

为了在所有方法中进行一轮，我们将实现`delete`方法，从数据库中完全删除用户。

```js
@Injectable()
export class UserService() {
    /* ... */

    public async delete(id: number): Promise<void> {  
        return await this.sequelizeInstance.transaction(async transaction => {  
            return await this.UserRepository.destroy({  
                where: { id },  
                transaction,  
            });  
        });  
    }

    /* ... */
}

```

在所有先前的示例中，我们定义了一个完整的`UserService`，允许我们操作数据。我们有可能创建、读取、更新和删除用户。

#### 用户模型

如果您想查看用户模型的实现，可以参考 Sequelize 章节。

#### 用户控制器

现在我们已经创建了我们的服务和模型，我们需要实现控制器来处理来自客户端的所有请求。该控制器至少提供了一个创建、读取、更新和删除处理程序，应该像以下示例一样实现。

```js
@Controller()  
export class UserController {  
   constructor(private readonly userService: UserService) { }

   /* ... */
}

```

控制器注入了`UserService`，以使用`UserService`中实现的方法。

提供一个`GET users`路由，允许访问数据库中的所有用户，您将看到我们不希望用户访问所有用户的数据，只希望用户访问自己的数据。这就是为什么我们使用了一个守卫，只允许用户访问自己的数据。

```js
@Controller()  
export class UserController {  
    /* ... */

    @Get('users')  
    @UseGuards(CheckLoggedInUserGuard)
    public async index(@Res() res) {  
        const users = await this.userService.findAll();  
        return res.status(HttpStatus.OK).json(users);  
    }

    /* ... */
}

```

用户可以访问一个允许您创建新用户的路由。当然，如果您愿意，用户可以注册到已登录的应用程序中，我们必须允许那些没有限制的用户。

```js
@Controller()  
export class UserController {  
    /* ... */

    @Post('users')  
    public async create(@Body() body: any, @Res() res) {  
       if (!body || (body && Object.keys(body).length === 0)) throw new Error('Missing some information.');  

        await this.userService.create(body);  
        return res.status(HttpStatus.CREATED).send();  
    }  

    /* ... */
}

```

我们还提供了一个`GET users/:id`路由，允许您通过 ID 获取用户。当然，已登录用户不应该能够访问另一个用户的数据，即使通过这个路由。该路由也受到守卫的保护，以允许用户访问自己而不是其他用户。

```js
@Controller()  
export class UserController {  
    /* ... */

    @Get('users/:id')  
    @UseGuards(CheckLoggedInUserGuard)
    public async show(@Param() id: number, @Res() res) {  
       if (!id) throw new Error('Missing id.');  

        const user = await this.userService.findById(id);  
        return res.status(HttpStatus.OK).json(user);  
    }   

    /* ... */
}

```

用户可能想要更新自己的一些信息，这就是为什么我们通过以下`PUT users/:id`路由提供了一种更新用户的方式。这个路由也受到守卫的保护，以避免用户更新其他用户。

```js
@Controller()  
export class UserController {  
    /* ... */
    @Put('users/:id')  
    @UseGuards(CheckLoggedInUserGuard)
    public async update(@Param() id: number, @Body() body: any, @Res() res) {  
       if (!id) throw new Error('Missing id.');  

        await this.userService.update(id, body);  
        return res.status(HttpStatus.OK).send();  
    }

```

使用删除来完成最后一个处理程序。这个路由也必须受到守卫的保护，以避免用户删除另一个用户。唯一能够被用户删除的用户是他自己。

```js
    @Delete('users/:id')  
    @UseGuards(CheckLoggedInUserGuard)
    public async delete(@Param() id: number, @Res() res) {  
       if (!id) throw new Error('Missing id.');  

        await this.userService.delete(id);  
        return res.status(HttpStatus.OK).send();  
    }  
}

```

我们已经在这个控制器中实现了所有需要的方法。其中一些受到守卫的限制，以应用一些安全性，并防止用户操纵另一个用户的数据。

#### 模块

为了完成`UserModule`的实现，我们当然需要设置模块。该模块包含一个服务、一个控制器和一个提供者，允许您注入用户模型并提供一种操作存储数据的方式。

```js
@Module({  
    imports: [],  
    controllers: [UserController],  
    providers: [userProvider, UserService],
    exports: [UserService]  
})  
export class UserModule {}

```

该模块被导入到主`AppModule`中，就像`AuthenticationModule`一样，以便在应用程序中使用并可访问。

### 应用程序模块

`AppModule`导入了三个示例模块。

+   `DatabaseModule`访问 sequelize 实例并访问数据库。

+   `AuthenticationModule`允许您登录用户并使用适当的策略。

+   `UserModule`公开了一些可以由客户端请求的端点。

最后，该模块应该如以下示例所示。

```js
@Module({  
   imports: [  
        DatabaseModule,  
        // Here we specify the strategy
        AuthenticationModule.forRoot('jwt'),  
        UserModule  
    ]
})  
export class AppModule implements NestModule {  
   public configure(consumer: MiddlewaresConsumer) {  
       consumer  
           .apply(AuthenticationMiddleware)  
           .with(strategy)  
           .forRoutes(  
               { path: '/users', method: RequestMethod.GET },  
                { path: '/users/:id', method: RequestMethod.GET },  
                { path: '/users/:id', method: RequestMethod.PUT },  
                { path: '/users/:id', method: RequestMethod.DELETE }  
           );  
    }  
}

```

如你在这个例子中所看到的，我们已经将`AuthenticationMiddleware`应用到了我们想要保护不被未登录用户访问的路由上。

这个中间件的目的是应用 passport 中间件`passport.authenticate`，它验证用户提供的令牌，并将请求存储在头部作为`Authorization`值。这个中间件将使用策略参数来对应应该应用的策略，对我们来说是`strategy = 'jwt'`。

这个中间件应用于`UserController`的几乎所有路由，除了允许你创建新用户的`POST /users`。

## 身份验证中间件

如前一节所示，我们已经应用了`AuthenticationMiddleware`，并且我们已经看到 passport 是用于验证用户的中间件。这个中间件将使用策略`jwt`执行`passport.authenticate`方法，使用一个回调函数来返回验证方法的结果。因此，我们可以接收对应于令牌的有效负载，或者在验证不起作用的情况下收到错误。

```js
@Injectable()
export class AuthenticationMiddleware implements NestMiddleware {
    constructor(private userService: UserService) { }

    async resolve(strategy: string): Promise<ExpressMiddleware> {
        return async (req, res, next) => {
            return passport.authenticate(strategy, async (...args: any[]) => {
                const [,  payload, err] = args;
                if (err) {
                    return res.status(HttpStatus.BAD_REQUEST).send('Unable to authenticate the user.');
                }

                const user = await this.userService.findOne({ where: { email: payload.email }});
                req.user = user;
                return next();
            })(req, res, next);
        };
    }
}

```

如果身份验证成功，我们将能够将用户存储在请求`req`中，以便控制器或守卫使用。中间件实现了`NestMiddleware`接口，以实现解析函数。它还注入了`UserService`，以便找到已验证的用户。

## 使用守卫管理限制

Nest.js 带有一个守卫概念。这个可注入的守卫有一个单一的责任，就是确定请求是否需要由路由处理程序处理。

守卫用于实现`canActivate`接口的类，以实现`canActivate`方法。

守卫在每个中间件之后和任何管道之前执行。这样做的目的是将中间件的限制逻辑与守卫分开，并重新组织这个限制。

想象一下使用守卫来管理对特定路由的访问，并且你希望这个路由只能被已登录的用户访问。为此，我们实现了一个新的守卫，如果访问路由的用户与想要访问资源的用户相同，它必须返回`true`。使用这种类型的守卫，可以避免用户访问其他用户。

```js
@Injectable()
export class CheckLoggedInUserGuard implements CanActivate {
    canActivate(context: ExecutionContext): boolean | Promise<boolean> | Observable<boolean> {
        const request = context.switchToHttp().getRequest();
        return Number(req.params.userId) === req.user.id;
    }
}

```

正如你所看到的，你可以从上下文中获取处理程序，该上下文对应于应用守卫的控制器上的路由处理程序。你还可以从请求参数中获取`userId`，并将其与请求中注册的已登录用户进行比较。如果想要访问数据的用户是相同的，那么他可以访问请求参数中的引用，否则他将收到`403 Forbidden`。

要将守卫应用到路由处理程序，请参见以下示例。

```js
@Controller()
@UseGuards(CheckLoggedInUserGuard)  
export class UserController {/*...*/}

```

现在我们已经保护了所有的用户控制器的路由处理程序，它们都是可访问的，除了`delete`，因为用户必须是`admin`才能访问。如果用户没有适当的角色，他们将收到`403 Forbidden`的响应。

# Nest.js passport 包

`@nestjs/passport`包是一个可扩展的包，允许你在 Nest.js 中使用 passport 的任何策略。如前一节所示，可以手动实现身份验证，但如果想要更快地实现并包装策略，那么就使用这个好的包。

在本节中，你将看到使用`jwt`的包的用法，就像前一节所示的那样。要使用它，你必须安装以下包：

`npm install --save @nestjs/passport passport passport-jwt jsonwebtoken`

要使用这个包，你将有可能使用与前一节中实现的完全相同的`AuthenticationService`，但记得遵循下面的代码示例。

```js
@Injectable()  
export class AuthenticationService {  
   constructor(private readonly userService: UserService) { }  

   createToken(email: string, ttl?: number) {  
        const expiresIn = ttl || 60 * 60;  
        const secretOrKey = 'secret';  
        const user = { email };  
        const token = jwt.sign(user, secretOrKey, { expiresIn });  
        return {  
            expires_in: expiresIn,  
            access_token: token,  
        };  
   }  

   async validateUser(payload: { email: string; password: string }): Promise<boolean> {  
        const user = await this.userService.findOne({  
            where: { email: payload.email }  
        });  
        return !!user;  
   }  
}

```

要实例化 jwt 策略，你还需要实现 `JwtStrategy`，但现在你只需要传递选项，因为 passport 被包装在这个包中，并且会在幕后自动将策略应用于 passport。

```js
@Injectable()
export default class JwtStrategy extends PassportStrategy(Strategy) {  
   constructor(private readonly authenticationService: AuthenticationService) {  
       super({  
            jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),  
            passReqToCallback: true,  
            secretOrKey: 'secret'  
        });
    }  

   public async validate(req, payload, done) {  
       const isValid = await this.authenticationService.validateUser(payload);  
        if (!isValid) {  
           return done('Unauthorized', null);  
        } else {  
           return done(null, payload);  
        }  
   }  
}

```

正如你所看到的，在这个新的 `JwtStrategy` 实现中，你不再需要实现回调。这是因为现在你扩展了 `PassportStrategy(Strategy)`，其中 `Strategy` 是从 `passport-jwt` 库中导入的成员。此外，`PassportStrategy` 是一个混合类，将调用我们实现并根据这个混合类的抽象成员命名的 `validate` 方法。该方法将被策略调用作为有效载荷的验证方法。

该包提供的另一个功能是 `AuthGuard`，它可以与 `@UseGuards(AuthGuard('jwt'))` 一起使用，以在特定控制器方法上启用身份验证，而不是使用我们在上一节中实现的中间件。

`AuthGuard` 接受策略名称作为参数，我们的示例中是 `jwt`，还可以接受遵循 `AuthGuardOptions` 接口的其他一些参数。该接口定义了三个可用的选项：

+   - `callback` 作为允许你实现自己逻辑的函数

+   - `property` 作为一个字符串，用于定义要添加到请求中并附加到经过身份验证的用户的属性的名称

+   你还看到了新的 `@nestjs/passport` 包，它允许你以更快的方式实现一些类，如 `AuthenticationService` 和 `JwtStrategy`，并能够使用该包提供的 `AuthGuard` 在任何控制器方法上验证任何用户。

默认情况下，`session` 被设置为 false，`property` 被设置为 user。默认情况下，回调将返回 `user` 或 `UnauthorizedException`。就是这样，现在你可以在任何控制器方法上验证用户并从请求中获取用户。

你唯一需要做的就是创建以下示例中的 `AuthModule`：

```js
@Module({
  imports: [UserModule],
  providers: [AuthService, JwtStrategy],
})
export class AuthModule {}

```

正如你所看到的，现在不需要创建提供者来实例化策略，因为它现在被包装在这个包中。

# 摘要

在本章中，你已经学会了什么是 passport 以及配置 passport 不同部分的策略，以便验证用户并将其存储到请求中。你还学会了如何实现不同的模块，`AuthenticationModule` 和 `UserModule`，以便用户登录并提供一些用户可访问的端点。当然，我们已经通过 `AuthenticationMiddleware` 和 `CheckLoggedInUserGuard` 限制了对一些数据的访问，以提供更多安全性。

在下一章中，你将学习关于依赖注入模式的内容。

- `session` 作为布尔值
