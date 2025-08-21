# 第九章：微服务

使用 Nest.js 微服务，我们能够提取出应用程序业务逻辑的一部分，并在单独的 Nest.js 上下文中执行它。默认情况下，这个新的 Nest.js 上下文并不在新线程甚至新进程中执行。因此，“微服务”这个名称有点误导。实际上，如果您坚持使用默认的 TCP 传输，用户可能会发现请求完成的时间更长。然而，将应用程序的一些部分卸载到这个新的微服务上下文中也有好处。为了介绍基础知识，我们将坚持使用 TCP 传输，但在本章的高级架构部分中，我们将寻找一些现实世界的策略，Nest.js 微服务可以提高应用程序性能。要查看一个工作示例，请记住您可以克隆本书的附带 Git 存储库：

`git clone https://github.com/backstopmedia/nest-book-example.git`

# 服务器引导

要开始，请确保`@nestjs/microservices`已安装在您的项目中。该模块提供了客户端、服务器和所需的实用程序，以将 Nest.js API 应用程序转换为微服务应用程序。最后，我们将修改我们的博客应用程序的引导程序以启用微服务。

```js
async function bootstrap() {
    const app = await NestFactory.create(AppModule);
    app.connectMicroservice({
        transport: Transport.TCP,
        options: {
            port: 5667
        }
    });

    await app.startAllMicroservicesAsync();
    await app.listen(3001);
}

```

`connectMicroservice`方法指示 NestApplication 设置一个新的 NestMicroservice 上下文。该对象提供了设置 NestMicroservice 上下文的选项。在这里，我们保持简单，并使用 Nest.js 提供的标准 TCP 传输。调用`startAllMicroservicesAsync`启动 NestMicroservice 上下文。在调用 NestApplication 的`listen`之前，请务必这样做。

# 配置

传递给`connectMicroservice`的配置参数取决于我们使用的传输方式。传输是客户端和服务器的组合，它们协同工作以在 NestApplication 和 NestMicroservice 上下文之间传输微服务请求和响应。Nest.js 附带了许多内置传输，并提供了创建自定义传输的能力。可用的参数取决于我们使用的传输方式。现在，我们将使用 TCP 传输，但稍后会介绍其他传输方式。TCP 传输的可能选项包括：

+   **host**：运行 NestMicroservice 上下文的主机。默认值是假定为`localhost`，但如果 NestMicroservice 作为不同主机上的单独项目运行，例如不同的 Kubernetes pod，可以使用这个选项。

+   **port**：NestMicroservice 上下文正在侦听的端口。默认值是假定为`3000`，但我们将使用不同的端口来运行我们的 NestMicroservice 上下文。

+   **retryAttempts**：在 TCP 传输的上下文中，这是服务器在收到`CLOSE`事件后尝试重新建立自身的次数。

+   **retryDelay**：与`retryAttempts`一起工作，并延迟传输重试过程一定的毫秒数。

# 第一个微服务处理程序

对于我们的第一个微服务处理程序，让我们将 UserController 索引方法转换为微服务处理程序。为此，我们复制该方法并进行一些简单的修改。我们将不再使用`Get`来注释该方法，而是使用`MessagePattern`。

```js
@Controller()
export class UserController {

    @Get('users')
    public async index(@Res() res) {
        const users = await this.userService.findAll();
        return res.status(HttpStatus.OK).json(users);
    }

    @MessagePattern({cmd: 'users.index'})
    public async rpcIndex() {
        const users = await this.userService.findAll();
        return users;
    }
}

```

消息模式为 Nest.js 提供了确定要执行哪个微服务处理程序的手段。该模式可以是一个简单的字符串或一个复杂的对象。当发送新的微服务消息时，Nest.js 将搜索所有已注册的微服务处理程序，以找到与消息模式完全匹配的处理程序。

微服务方法本身可以执行与正常控制器处理程序几乎相同的业务逻辑来响应。与正常的控制器处理程序不同，微服务处理程序没有 HTTP 上下文。事实上，像`@Get`、`@Body`和`@Req`这样的装饰器在微服务控制器中没有意义，也不应该使用。为了完成消息的处理，处理程序可以返回一个简单的值、promise 或 RxJS Observable。

# 发送数据

之前的微服务处理程序非常牵强。更有可能的是，微服务处理程序将被实现为对数据进行一些处理并返回一些值。在正常的 HTTP 处理程序中，我们会使用`@Req`或`@Body`来从 HTTP 请求的主体中提取数据。由于微服务处理程序没有 HTTP 上下文，它们将输入数据作为方法参数。

```js
@Controller()
export class UserController {
    @Client({transport: Transport.TCP, options: { port: 5667 }})
    client: ClientProxy

    @Post('users')
    public async create(@Req() req, @Res() res) {
        this.client.send({cmd: 'users.index'}, {}).subscribe({
            next: users => {
                res.status(HttpStatus.OK).json(users);
            },
            error: error => {
                res.status(HttpStatus.INTERNAL_SERVER_ERROR).json(error);
            }
        });
    }

    @MessagePattern({cmd: 'users.create'})
    public async rpcCreate(data: any) {
        if (!data || (data && Object.keys(data).length === 0)) throw new Error('Missing some information.');

        await this.userService.create(data);
    }
}

```

在这个例子中，我们使用`@Client`装饰器为 Nest.js 依赖注入提供了一个注入微服务客户端实例的地方。客户端装饰器接受与在引导应用程序时传递给`connectMicroservice`相同的配置对象。客户端是 NestApplication 上下文与 NestMicroservice 上下文进行通信的方式。使用客户端，我们修改了原始的`@Post('users')` API，将创建新用户的处理过程转移到了 NestMicroservice 上下文中。

![微服务 TCP 流](img/MicroserviceFlow001.png)

这张图表展示了创建新用户时数据流的简化视图。客户端与微服务上下文建立 TCP 连接，并将数据库操作的处理过程转移到微服务上下文中。`rpcCreate`方法将返回一个成功的响应和一些数据，或者一个异常。在处理微服务消息的同时，正常的控制器处理程序将等待响应。

请注意，微服务客户端的`send`方法返回一个 Observable。如果你想等待来自微服务的响应，只需订阅 Observable 并使用响应对象发送结果。另外，Nest.js 将 Observables 视为一等公民，并且它们可以从处理程序中返回。Nest.js 会负责订阅 Observable。请记住，你会失去一些对响应状态码和主体的控制。但是，你可以通过异常和异常过滤器重新获得一些控制。

# 异常过滤器

异常过滤器提供了一种将从微服务处理程序抛出的异常转换为有意义对象的方法。例如，我们的`rpcCreate`方法目前抛出一个带有字符串的错误，但是当 UserService 抛出错误或者可能是 ORM 时会发生什么。这个方法可能会抛出许多不同的错误，而调用方法唯一知道发生了什么的方法是解析错误字符串。这是完全不可接受的，所以让我们来修复它。

首先创建一个新的异常类。注意我们的微服务异常扩展了 RpcException，并且在构造函数中没有传递 HTTP 状态码。这些是微服务异常和正常的 Nest.js API 异常之间唯一的区别。

```js
export class RpcValidationException extends RpcException {
    constructor(public readonly validationErrors: ValidationError[]) {
        super('Validation failed');
    }
}

```

现在我们可以改变`rpcCreate`方法，当数据无效时抛出这个异常。

```js
@MessagePattern({cmd: 'users.create'})
public async rpcCreate(data: any) {
    if (!data || (data && Object.keys(data).length === 0)) throw new RpcValidationException();

    await this.userService.create(data);
}

```

最后，创建一个异常过滤器。微服务异常过滤器与它们的正常 API 对应物不同，它们扩展了 RpcExceptionFilter 并返回一个 ErrorObservable。这个过滤器将捕获我们创建的 RpcValidationException，并抛出一个包含特定错误代码的对象。

**注意**`throwError`方法来自 RxJS 版本 6 包。如果你仍在使用 RxJS 版本 5，使用`Observable.throw`。

```js
@Catch(RpcValidationException)
export class RpcValidationFilter implements RpcExceptionFilter {
    public catch(exception: RpcValidationException): ErrorObservable {
        return throwError({
            error_code: 'VALIDATION_FAILED',
            error_message: exception.getError(),
            errors: exception.validationErrors
        });
    }
}

```

当新的异常发生时，我们所要做的就是采取行动。修改`create`方法以捕获从微服务客户端抛出的任何异常。在捕获中，检查`error_code`字段是否具有`VALIDATION_FAILED`的值。当它是这样时，我们可以向用户返回`400`的 HTTP 状态码。这将允许用户的客户端，即浏览器，以不同的方式处理错误，可能向用户显示一些消息并允许他们修复输入的数据。与将所有错误作为`500`的 HTTP 状态码返回给客户端相比，这提供了更好的用户体验。

```js
@Post('users')
public async create(@Req() req, @Res() res) {
    this.client.send({cmd: 'users.create'}, body).subscribe({
        next: () => {
            res.status(HttpStatus.CREATED).send();
        },
        error: error => {
            if (error.error_code === 'VALIDATION_FAILED') {
                res.status(HttpStatus.BAD_REQUEST).send(error);
            } else {
                res.status(HttpStatus.INTERNAL_SERVER_ERROR).send(error);
            }
        }
    });
}

```

# 管道

Nest.js 中最常用的管道是 ValidationPipe。然而，这个管道不能与微服务处理程序一起使用，因为它会抛出扩展 HttpException 的异常。在微服务中抛出的所有异常都必须扩展 RpcException。为了解决这个问题，我们可以扩展 ValidationPipe，捕获 HttpException，并抛出 RpcException。

```js
@Injectable()
export class RpcValidationPipe extends ValidationPipe implements PipeTransform<any> {
    public async transform(value: any, metadata: ArgumentMetadata) {
        try {
            await super.transform(value, metadata);
        } catch (error) {
            if (error instanceof BadRequestException) {
                throw new RpcValidationException();
            }

            throw error;
        }

        return value;
    }
}

```

在使用 ValidationPipe 之前，我们必须创建一个描述我们微服务方法期望的数据格式的类。

```js
class CreateUserRequest {
      @IsEmail()
      @IsNotEmpty()
      @IsDefined()
      @IsString()
      public email: string;

      @Length(8)
      @Matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)\S+$/)
      @IsDefined()
      @IsString()
      public password: string;

      @IsNotEmpty()
      @IsDefined()
      @IsString()
      public firstName: string;

      @IsNotEmpty()
      @IsDefined()
      @IsString()
      public lastName: string;
}

```

新的请求类使用`class-validator` NPM 包来验证从 Nest.js 微服务模块传递给我们微服务方法的对象。该类包含所有属性，并使用特定的装饰器描述这些属性应包含的内容。例如，`email`属性应该是一个电子邮件地址，不能是空的，必须被定义，并且必须是一个字符串。现在我们只需要将其连接到我们的`rpcCreate`方法。

```js
@MessagePattern({cmd: 'users.create'})
@UsePipes(new RpcValidationPipe())
@UseFilters(new RpcValidationFilter())
public async rpcCreate(data: CreateUserRequest) {
    await this.userService.create(data);
}

```

由于微服务处理程序不使用`@Body`装饰器，我们需要使用`@UsePipes`来使用我们的新的 RpcValidationPipe。这将指示 Nest.js 根据其类类型验证输入数据。就像对 API 一样，使用验证类和 RpcValidationPipe 来将输入验证从控制器或微服务方法中卸载出来。

# 守卫

在微服务中，守卫的作用与普通 API 中的作用相同。它们确定特定的微服务处理程序是否应该处理请求。到目前为止，我们已经使用守卫来保护 API 处理程序免受未经授权的访问。我们应该对我们的微服务处理程序做同样的事情。尽管在我们的应用程序中，我们的微服务处理程序只从我们已经受保护的 API 处理程序中调用，但我们永远不应该假设这将始终是这种情况。

```js
@Injectable()
export class RpcCheckLoggedInUserGuard implements CanActivate {
    canActivate(context: ExecutionContext): boolean | Promise<boolean> | Observable<boolean> {
        const data = context.switchToRpc().getData();
        return Number(data.userId) === data.user.id;
    }
}

```

新的守卫看起来与 API 的`CheckLoggedInUserGuard`守卫完全相同。不同之处在于传递给`canActivate`方法的参数。由于这个守卫是在微服务的上下文中执行的，它将获得一个微服务`data`对象，而不是 API 请求对象。

我们使用新的微服务守卫与我们之前使用 API 守卫的方式相同。只需在微服务处理程序上添加`@UseGuards`装饰器，我们的守卫现在将保护我们的微服务免受滥用。让我们为检索当前用户信息创建一个新的微服务。

```js
@Get('users/:userId')
@UseGuards(CheckLoggedInUserGuard)
public async show(@Param('userId') userId: number, @Req() req, @Res() res) {
    this.client.send({cmd: 'users.show'}, {userId, user: req.user}).subscribe({
        next: user => {
            res.status(HttpStatus.OK).json(user);
        },
        error: error => {
            res.status(HttpStatus.INTERNAL_SERVER_ERROR).send(error);
        }
    });
}

@MessagePattern({cmd: 'users.show'})
@UseGuards(RpcCheckLoggedInUserGuard)
public async rpcShow(data: any) {
    return await this.userService.findById(data.userId);
}

```

`show` API 处理程序现在将访问数据库的繁重工作交给了 NestMicroservice 上下文。微服务处理程序上的守卫确保，如果处理程序以某种方式在`show`API 处理程序之外被调用，它仍将保护用户数据免受未经授权的请求。但仍然存在一个问题。这个示例从数据库返回整个用户对象，包括散列密码。这是一个安全漏洞，最好通过拦截器来解决。

# 拦截器

微服务拦截器的功能与普通 API 拦截器没有任何不同。唯一的区别是拦截器接收到的是发送到微服务处理程序的数据对象，而不是 API 请求对象。这意味着您实际上可以编写一次拦截器，并在两种情境下使用它们。与 API 拦截器一样，微服务拦截器在微服务处理程序之前执行，并且必须返回一个 Observable。为了保护我们的`rpcShow`微服务端点，我们将创建一个新的拦截器，该拦截器将期望一个`User`数据库对象并移除`password`字段。

```js
@Injectable()
export class CleanUserInterceptor implements NestInterceptor {
    intercept(context: ExecutionContext, stream$: Observable<any>): Observable<any> {
        return stream$.pipe(
            map(user => JSON.parse(JSON.stringify(user))),
            map(user => {
                return {
                    ...user,
                    password: undefined
                };
            })
        );
    }
}

```

```js
@MessagePattern({cmd: 'users.show'})
@UseGuards(RpcCheckLoggedInUserGuard)
@UseInterceptors(CleanUserInterceptor)
public async rpcShow(data: any) {
    return await this.userService.findById(data.userId);
}

```

`rpcShow`微服务处理程序的响应现在将删除`password`字段。请注意，在拦截器中，我们必须将`User`数据库对象转换为 JSON 格式。这可能会因您使用的 ORM 而有所不同。使用 Sequelize，我们需要从数据库响应中获取原始数据。这是因为 ORM 的响应实际上是一个包含许多不同 ORM 方法和属性的类。通过将其转换为 JSON 格式，然后使用`password: undefined`的扩展运算符来删除`password`字段。

# 内置传输

TCP 传输只是 Nest.js 内置的几种传输方式之一。使用 TCP 传输，我们必须将 NestMicroservice 上下文绑定到另一个端口，占用服务器上的另一个端口，并确保 NestMicroservice 上下文在启动 NestApplication 上下文之前运行。其他内置传输可以克服这些限制并增加额外的好处。

## Redis

[Redis](https://redis.io/)是一个简单的内存数据存储，可以用作发布-订阅消息代理。Redis 传输利用了[redis](https://github.com/NodeRedis/node_redis) NPM 包和 Redis 服务器之间传递消息的 NestApplication 和 NestMicroservice 上下文。要使用 Redis 传输，我们需要更新我们的`bootstrap`方法以使用正确的 NestMicroservice 配置。

```js
async function bootstrap() {
    const app = await NestFactory.create(AppModule);
    app.connectMicroservice({
        transport: Transport.REDIS,
        options: {
            url: process.env.REDIS_URL
        }
    });

    await app.startAllMicroservicesAsync();
    await app.listen(3001);
}

```

您还必须更新所有使用`@Client`装饰器的位置，以使用相同的设置。相反，让我们将此配置集中化，这样我们就不会重复代码，并且可以更轻松地切换传输方式。

```js
export const microserviceConfig: RedisOptions = {
    transport: Transport.REDIS,
    options: {
        url: process.env.REDIS_URL
    }
};

```

Redis 传输可以采用以下选项：

+   **url**：Redis 服务器的 URL。默认值为`redis://localhost:6379`。

+   **retryAttempts**：当连接丢失时，微服务服务器和客户端将尝试重新连接到 Redis 服务器的次数。这用于为`redis` NPM 包创建`retry_strategy`。

+   **retryDelay**：与`retryAttempts`配合使用，以毫秒为单位延迟传输的重试过程。

现在我们可以更新应用程序的`bootstrap`以使用我们创建的`microserviceConfig`对象。

```js
async function bootstrap() {
    const app = await NestFactory.create(AppModule);
    app.connectMicroservice(microserviceConfig);

    await app.startAllMicroservicesAsync();
    await app.listen(3001);
}

```

最后，在 UserController 中更新`@Client`装饰器。

```js
@Controller()
export class UserController {
    @Client(microserviceConfig)
    client: ClientProxy
}

```

启动 Redis 服务器，例如[redis docker image](https://hub.docker.com/_/redis/)和应用程序，所有我们的微服务事务现在将通过 Redis 服务器进行处理。下面的图表显示了在创建新用户并使用 Redis 传输时的数据流的简化视图。

![微服务 Redis 流程](img/MicroserviceFlow002.png)

客户端和服务器都与 Redis 服务器建立连接。当调用`client.send`时，客户端会即时修改消息模式以创建发布和订阅通道。服务器消费消息并移除消息模式修改以找到正确的微服务处理程序。一旦微服务处理程序完成处理，模式再次被修改以匹配订阅通道。客户端消费这条新消息，取消订阅订阅通道，并将响应传递回调用者。

## MQTT

[MQTT](http://mqtt.org/)是一种简单的消息协议，旨在在网络带宽有限时使用。MQTT 传输利用[mqtt](https://github.com/mqttjs/MQTT.js) NPM 软件包和远程 MQTT 服务器在 NestApplication 和 NestMicroservice 上下文之间传递消息。数据流和微服务客户端和服务器的操作方式几乎与 Redis 传输相同。要使用 MQTT 传输，让我们更新 microserviceConfig 配置对象。

```js
export const microserviceConfig: MqttOptions = {
    transport: Transport.MQTT,
    options: {
        url: process.env.MQTT_URL
    }
};

```

MQTT 传输可以采用几种选项，所有这些选项都在`mqtt` NPM 软件包的 Github 存储库中详细说明。最值得注意的是，传输将`url`选项默认设置为`mqtt://localhost:1883`，并且没有连接重试。如果与 MQTT 服务器的连接丢失，微服务消息将不再传递。

启动 MQTT 服务器，例如[eclipse-mosquitto docker image](https://hub.docker.com/_/eclipse-mosquitto/)，现在应用程序和所有微服务事务将通过 MQTT 服务器进行处理。

## NATS

[NATS](https://nats.io/)是一个自称具有极高吞吐量的开源消息代理服务器。NATS 传输利用[nats](https://github.com/nats-io/node-nats) NPM 软件包和远程 NATS 服务器在 NestApplication 和 NestMicroservice 上下文之间传递消息。

```js
export const microserviceConfig: MqttOptions = {
    transport: Transport.NATS,
    options: {
        url: process.env.NATS_URL
    }
};

```

NATS 传输可以采用以下选项：

+   **url**：NATS 服务器的 URL。默认值为`nats://localhost:4222`。

+   **name**/**pass**：用于将 Nest.js 应用程序与 NATS 服务器进行身份验证的用户名和密码。

+   **maxReconnectAttempts**：当连接丢失时，服务器和客户端尝试重新连接到 NATS 服务器的次数。默认值是尝试重新连接 10 次。

+   **reconnectTimeWait**：与`maxReconnectAttempts`配合使用，以毫秒为单位延迟传输的重试过程。

+   **servers**：一组`url`字符串，所有这些字符串都是 NATS 服务器。这允许传输利用 NATS 服务器集群。

+   **tls**：一个布尔值，指示连接到 NATS 服务器时是否应使用 TLS。**注意**，默认值为 false，这意味着所有消息都以明文传递。也可以提供对象而不是布尔值，并且可以包含标准的 Node TLS 设置，如客户端证书。

启动 NATS 服务器，例如[nats docker image](https://hub.docker.com/_/nats/)，现在应用程序和所有微服务事务将通过 NATS 服务器进行处理。下面的图表显示了在创建新用户并使用 NATS 传输时数据流的简化视图。

![微服务 NATS 流程](img/MicroserviceFlow003.png)

客户端和服务器都与 NATS 服务器建立连接。当调用`client.send`时，客户端会即时修改消息模式以创建发布和订阅队列。Redis 传输和 NATS 传输之间最显着的区别之一是 NATS 传输使用队列组。这意味着现在我们可以有多个 NestMicroservice 上下文，并且 NATS 服务器将在它们之间负载平衡消息。服务器消耗消息并移除消息模式修改以找到正确的微服务处理程序。一旦微服务处理程序完成处理，模式将再次修改以匹配订阅通道。客户端消耗这条新消息，取消订阅订阅通道，并将响应传递给调用者。

## gRPC

[gRPC](https://grpc.io)是一个远程过程调用客户端和服务器，旨在与 Google 的[Protocol Buffers](https://developers.google.com/protocol-buffers/)一起使用。gRPC 和协议缓冲区是值得拥有自己的书籍的广泛主题。因此，我们将继续讨论在 Nest.js 应用程序中设置和使用 gRPC。要开始，我们需要[grpc](https://github.com/grpc/grpc-node) NPM 包。在我们可以为 Nest.js 应用程序编写任何代码之前，我们必须编写一个协议缓冲区文件。

```js
syntax = "proto3";

package example.nestBook;

message User {
    string firstName = 1;
    string lastName = 2;
    string email = 3;
}

message ShowUserRequest {
    double userId = 1;
}

message ShowUserResponse {
    User user = 1;
}

service UserService {
    rpc show (ShowUserRequest) returns (ShowUserResponse);
}

```

上面的代码片段描述了一个名为`UserService`的单个 gRPC 服务。这通常将映射到您自己项目中的一个服务或控制器。该服务包含一个名为`show`的方法，该方法接受一个带有`userId`的对象，并返回一个带有`user`属性的对象。`syntax`值指示 gRPC 包我们使用的协议缓冲区语言的格式。`package`声明充当我们在 proto 文件中定义的所有内容的命名空间。在导入和扩展其他 proto 文件时，这是最有用的。

**注意：**我们保持了 proto 文件的简单，以便我们可以专注于配置 Nest.js 以使用 gRPC 微服务。

与所有其他传输方式一样，我们现在需要在我们的控制器中配置 NestMicroservice 上下文和微服务客户端。

```js
export const microserviceConfig: GrpcOptions = {
    transport: Transport.GRPC,
    options: {
        url: '0.0.0.0:5667',
        protoPath: join(__dirname, './nest-book-example.proto'),
        package: 'example.nestBook'
    }
};

```

gRPC 传输可以采用以下选项：

+   **url**：gRPC 服务器的 URL。默认值为`localhost:5000`。

+   凭证：来自`grpc` NPM 包的`ServerCedentials`对象。默认情况下，使用`grpc.getInsecure`方法来检索默认凭证对象。这将禁用 TLS 加密。为了建立安全的通信通道，请使用`grpc.createSsl`并提供根 CA、私钥和公钥证书。有关凭证的更多信息可以在[这里](https://grpc.io/grpc/node/grpc.credentials.html)找到。

+   **protoPath**：proto 文件的绝对路径。

+   **root**：所有 proto 文件所在位置的绝对路径。这是一个可选选项，如果您不在自己的项目中导入其他 proto 文件，则很可能不需要。如果定义了此选项，它将被预置到`protoPath`选项之前。

+   **package**：用于客户端和服务器的包的名称。这应该与 proto 文件中给出的包名称匹配。

在我们真正使用 gRPC 传输之前，我们需要对我们的控制器进行一些更改。

```js
@Controller()
export class UserController implements OnModuleInit {
    @Client(microserviceConfig)
    private client: ClientGrpc;
    private protoUserService: IProtoUserService;

    constructor(
        private readonly userService: UserService
    ) {
    }

    public onModuleInit() {
        this.protoUserService = this.client.getService<IProtoUserService>('UserService');
    }
}

```

请注意，我们仍然使用`@Client`装饰的`client`属性，但我们有一个新类型`ClientGrpc`和一个新属性`protoUserService`。使用 gRPC 传输时注入的客户端不再包含`send`方法。相反，它具有一个`getService`方法，我们必须使用它来检索我们在 proto 文件中定义的服务。我们使用`onModuleInit`生命周期钩子，以便在 Nest.js 实例化我们的模块之后立即检索 gRPC 服务，而在任何客户端尝试使用控制器 API 之前。`getService`方法是一个通用方法，实际上并不包含任何方法定义。相反，我们需要提供我们自己的方法。

```js
import { Observable } from 'rxjs';

export interface IProtoUserService {
    show(data: any): Observable<any>;
}

```

我们可以对我们的接口更加明确，但这可以传达要点。现在我们控制器中的`protoUserService`属性将具有一个`show`方法，允许我们调用`show` gRPC 服务方法。

```js
@Get('users/:userId')
@UseGuards(CheckLoggedInUserGuard)
public async show(@Param('userId') userId: number, @Req() req, @Res() res) {
    this.protoUserService.show({ userId: parseInt(userId.toString(), 10) }).subscribe({
        next: user => {
            res.status(HttpStatus.OK).json(user);
        },
        error: error => {
            res.status(HttpStatus.INTERNAL_SERVER_ERROR).json(error);
        }
    });
}

@GrpcMethod('UserService', 'show')
public async rpcShow(data: any) {
    const user =  await this.userService.findById(data.userId);
    return {
        user: {
            firstName: user.firstName,
            lastName: user.lastName,
            email: user.email
        }
    };
}

```

控制器的`show` API 方法已更新为使用`protoUserService.show`。这将调用`rpcShow`方法，但通过 gRPC 微服务传输。`rpcShow`方法包含不同的装饰器`@GrpcMethod`，而不是`@MessagePattern`。这对于所有 gRPC 微服务处理程序是必需的，因为微服务不再匹配模式，而是调用定义的 gRPC 服务方法。实际上，这是`@GrpcMethod`装饰器的两个可选参数的映射：服务名称和服务方法。

```js
export class UserController implements OnModuleInit {
    @GrpcMethod()
    public async rpcShow(data: any) {
    }
}

```

在上面的例子中，我们在调用`@GrpcMethod`装饰器时没有定义服务名称和服务方法。Nest.js 将自动将这些值映射到方法和类名。在这个例子中，这相当于`@GrpcMethod('UserController', 'rpcShow')`。

您可能已经注意到我们将`0.0.0.0:5667`作为我们 gRPC 服务器的 URL。当我们启动 Nest.js 应用程序时，它将在本地主机上创建一个 gRPC 服务器，并在端口`5667`上进行监听。从表面上看，这可能看起来像 TCP 传输的更复杂版本。但是，gRPC 传输的强大之处直接源自协议缓冲区的语言和平台不可知性。这意味着我们可以创建一个使用 gRPC 公开微服务的 Nest.js 应用程序，该微服务可以被任何其他语言或平台使用，只要它也使用协议缓冲区连接到我们的微服务。我们还可以创建 Nest.js 应用程序，连接到可能在其他语言（如 Go）中公开的微服务。

![微服务 gRPC 流程](img/MicroserviceFlow004.png)

当使用 gRPC 传输连接到两个或更多不同 URL 的服务时，我们需要创建相等数量的 gRPC 客户端连接，每个服务器一个。上面的图表显示了如果我们将示例博客应用程序中的评论的 crud 操作转移到 Go 服务器中，处理将会是什么样子。我们使用 gRPC 客户端连接到 Nest.js 应用程序中托管的用户微服务，另外一个连接到 Go 应用程序中托管的评论微服务。

使用任何其他传输都可以获得相同的设置。但是，您需要编写额外的代码来序列化和反序列化 Nest.js 应用程序和托管微服务的 Go 应用程序之间的消息。通过使用 gRPC 传输，协议缓冲区会为您处理这些问题。

# 自定义传输

自定义传输允许您为 NestApplication 和 NestMicroservice 上下文之间的通信定义新的微服务客户端和服务器。您可能出于多种原因想要创建自定义传输策略：您或您的公司已经有一个没有内置 Nest.js 传输的消息代理服务，或者您需要自定义内置传输的工作方式。在我们的例子中，我们将通过实现一个新的 RabbitMQ 传输来工作。

```js
export class RabbitMQTransportServer extends Server implements CustomTransportStrategy {
    private server: amqp.Connection = null;
    private channel: amqp.Channel = null;

    constructor(
        private readonly url: string,
        private readonly queue: string
    ) {
        super();
    }
}

```

Nest.js 要求所有自定义传输都实现`CustomTransportStrategy`接口。这迫使我们定义自己的`listen`和`close`方法。在我们的例子中，我们连接到 RabbitMQ 服务器并监听特定的频道。关闭服务器就像从 RabbitMQ 服务器断开连接一样简单。

```js
public async listen(callback: () => void) {
    await this.init();
    callback();
}

public close() {
    this.channel && this.channel.close();
    this.server && this.server.close();
}

private async init() {
    this.server = await amqp.connect(this.url);
    this.channel = await this.server.createChannel();
    this.channel.assertQueue(`${this.queue}_sub`, { durable: false });
    this.channel.assertQueue(`${this.queue}_pub`, { durable: false });
}

```

通过扩展 Nest.js 的`Server`类，我们的自定义传输预先配备了 RxJS 处理消息的功能，这使得 Nest.js 非常出色。然而，我们的自定义传输目前并没有真正处理消息。我们需要添加逻辑，以确定消息将如何通过 RabbitMQ 发送和接收到我们的自定义传输。

```js
public async listen(callback: () => void) {
    await this.init();
    this.channel.consume(`${this.queue}_sub`, this.handleMessage.bind(this), {
        noAck: true,
    });
    callback();
}

private async handleMessage(message: amqp.Message) {
    const { content } = message;
    const packet = JSON.parse(content.toString()) as ReadPacket & PacketId;
    const handler = this.messageHandlers[JSON.stringify(packet.pattern)];

    if (!handler) {
        return this.sendMessage({
            id: packet.id,
            err: NO_PATTERN_MESSAGE
        });
    }

    const response$ = this.transformToObservable(await handler(packet.data)) as Observable<any>;
    response$ && this.send(response$, data => this.sendMessage({
        id: packet.id,
        ...data
    }));
}

private sendMessage(packet: WritePacket & PacketId) {
    const buffer = Buffer.from(JSON.stringify(packet));
    this.channel.sendToQueue(`${this.queue}_pub`, buffer);
}

```

自定义传输现在将在`sub`频道上监听传入的消息，并在`pub`频道上发送响应。`handleMessage`方法解码消息的内容字节数组，并使用嵌入的模式对象找到正确的微服务处理程序来处理消息。例如，`{cmd: 'users.create'}`将由`rpcCreate`处理程序处理。最后，我们调用处理程序，将响应转换为 Observable，并将其传递回 Nest.js 的`Server`类。一旦提供了响应，它将通过我们的`sendMessage`方法传递，并通过`pub`频道传出。

由于服务器没有客户端是无用的，我们也需要创建一个客户端。RabbitMQ 客户端必须扩展 Nest.js 的`ClientProxy`类，并为`close`、`connect`和`publish`方法提供重写。

```js
export class RabbitMQTransportClient extends ClientProxy {
    private server: amqp.Connection;
    private channel: amqp.Channel;
    private responsesSubject: Subject<amqp.Message>;

    constructor(
        private readonly url: string,
        private readonly queue: string) {
        super();
    }

    public async close() {
        this.channel && await this.channel.close();
        this.server && await this.server.close();
    }

    public connect(): Promise<void> {
        return new Promise(async (resolve, reject) => {
            try {
                this.server = await amqp.connect(this.url);
                this.channel = await this.server.createChannel();

                const { sub, pub } = this.getQueues();
                await this.channel.assertQueue(sub, { durable: false });
                await this.channel.assertQueue(pub, { durable: false });

                this.responsesSubject = new Subject();
                this.channel.consume(pub, (message) => { this.responsesSubject.next(message); }, { noAck: true });
                resolve();
            } catch (error) {
                reject(error);
            }
        });
    }

    protected async publish(partialPacket: ReadPacket, callback: (packet: WritePacket) => void) {
    }

    private getQueues() {
        return { pub: `${this.queue}_pub`, sub: `${this.queue}_sub` };
    }
}

```

在我们的示例中，我们创建了一个新的连接到 RabbitMQ 服务器，并指定了`pub`和`sub`通道。客户端与服务器相比，使用了相反的通道配置。客户端通过`sub`通道发送消息，并在`pub`通道上监听响应。我们还利用了 RxJS 的强大功能，通过将所有响应导入 Subject 来简化`publish`方法中的处理。让我们实现`publish`方法。

```js
protected async publish(partialPacket: ReadPacket, callback: (packet: WritePacket) => void) {
    if (!this.server || !this.channel) {
        await this.connect();
    }

    const packet = this.assignPacketId(partialPacket);
    const { sub } = this.getQueues();

    this.responsesSubject.asObservable().pipe(
        pluck('content'),
        map(content => JSON.parse(content.toString()) as WritePacket & PacketId),
        filter(message => message.id === packet.id),
        take(1)
    ).subscribe(({err, response, isDisposed}) => {
        if (isDisposed || err) {
            callback({
                err,
                response: null,
                isDisposed: true
            });
        }

        callback({err, response});
    });

    this.channel.sendToQueue(sub, Buffer.from(JSON.stringify(packet)));
}

```

`publish`方法首先为消息分配一个唯一的 ID，并订阅响应主题以将响应发送回微服务调用者。最后，调用`sendToQueue`将消息作为字节数组发送到`sub`通道。一旦收到响应，就会触发对响应主题的订阅。订阅流的第一件事是提取响应的`content`并验证消息 ID 是否与最初调用`publish`时分配的 ID 匹配。这可以防止客户端处理不属于特定`publish`执行上下文的消息响应。简而言之，客户端将接收每个微服务响应，甚至可能是针对不同微服务或相同微服务的不同执行的响应。如果 ID 匹配，客户端会检查错误并使用`callback`将响应发送回微服务调用者。

在我们可以使用新传输之前，我们需要更新之前创建的微服务配置对象。

```js
export const microserviceConfig = {
    url: process.env.AMQP_URL
};

export const microserviceServerConfig: (channel: string) => CustomStrategy = channel => {
    return {
        strategy: new RabbitMQTransportServer(microserviceConfig.url, channel)
    }
};

```

现在我们有了一个方法，可以实例化我们的自定义传输服务器。这在我们应用程序的`bootstrap`中用于将我们的 NestMicroservice 上下文连接到 RabbitMQ 服务器。

```js
async function bootstrap() {
    const app = await NestFactory.create(AppModule);
    app.connectMicroservice(microserviceServerConfig('nestjs_book'));

    await app.startAllMicroservicesAsync();
    await app.listen(3001);
}

```

我们自定义传输的最后一部分在我们的控制器中。由于我们使用自定义传输，我们不能再使用`@ClientProxy`装饰器。相反，我们必须自己实例化我们的自定义传输。你可以在构造函数中这样做：

```js
@Controller()
export class UserController {
    client: ClientProxy;

    constructor(private readonly userService: UserService) {
        this.client = new RabbitMQTransportClient(microserviceConfig.url, 'nestjs_book');
    }
}

```

等等！你现在在控制器和自定义传输客户端之间创建了一个硬绑定。这会使将来迁移到不同策略变得更加困难，而且非常难以测试。相反，让我们利用 Nest.js 的依赖注入来创建我们的客户端。首先创建一个新模块来容纳和公开我们的自定义传输客户端。

```js
const ClientProxy = {
  provide: 'ClientProxy',
  useFactory: () => new RabbitMQTransportClient(microserviceConfig.url, 'nestjs_book')
};

@Module({
    imports: [],
    controllers: [],
    components: [ClientProxy],
    exports: [ClientProxy]
})
export class RabbitMQTransportModule {}

```

在我们的示例中，我们给我们的组件注入了标记为`'ClientProxy'`的注入令牌。这只是为了保持简单，你可以随意更改它。重要的是确保用于注册组件的注入令牌也是我们在控制器构造函数中放置`@Inject`装饰器时使用的注入令牌。

```js
@Controller()
export class UserController {

    constructor(
        private readonly userService: UserService,
        @Inject('ClientProxy')
        private readonly client: ClientProxy
    ) {
    }

```

我们的控制器现在将在运行时注入一个微服务客户端，允许 API 处理程序与微服务处理程序进行通信。更好的是，客户端现在可以在测试中被模拟重写。启动一个 RabbitMQ 服务器，比如[rabbitmq docker image](https://hub.docker.com/_/rabbitmq/)，并设置`AMQP_URL`环境变量，即`amqp://guest:guest@localhost:5672`，所有微服务请求将通过 RabbitMQ 服务器进行处理。

在我们的 RabbitMQ 示例中，微服务客户端和服务器的数据流以及操作方式几乎与 NATS 传输相同。就像 NATS 一样，RabbitMQ 提供了多个 NestMicroservice 上下文消费消息的能力。RabbitMQ 将在所有消费者之间进行负载均衡。

# 混合应用程序

当我们在本章开始实现微服务时，我们修改了启动方法来调用`connectMicroservice`。这是一个特殊的方法，将我们的 Nest.js 应用程序转换为混合应用程序。这意味着我们的应用程序现在包含多种上下文类型。这很简单，但这有一些影响，你应该意识到。具体来说，使用混合应用程序方法，你将无法再为 NestMicroservice 上下文附加全局过滤器、管道、守卫和拦截器。这是因为 NestMicroservice 上下文会立即启动，但在混合应用程序中不会连接。为了解决这个限制，我们可以独立地创建我们的两个上下文。

```js
async function bootstrap() {
    const app = await NestFactory.create(AppModule);
    const rpcApp = await NestFactory.createMicroservice(AppModule, microserviceServerConfig('nestjs_book'));
    rpcApp.useGlobalFilters(new RpcValidationFilter());

    await rpcApp.listenAsync();
    await app.listen(process.env.PORT || 3000);
}

```

现在我们正在独立创建两个应用程序上下文，我们可以利用 NestMicroservice 上下文的全局变量。为了测试这一点，我们可以更新`rpcCreate`处理程序以删除`RpcValidationFilter`。在这一点上执行应用程序仍然应该导致在请求`create`API 时不包含必需字段时返回验证错误。

```js
@MessagePattern({cmd: 'users.create'})
public async rpcCreate(data: CreateUserRequest) {
    if (!data || (data && Object.keys(data).length === 0)) throw new RpcValidationException();
    await this.userService.create(data);
}

```

我们可以扩展这种启动应用程序的方法，将更多的应用程序拆分为独立的上下文。这仍然不使用多个进程或线程，但通过使用一些更高级的架构设计，我们可以获得这些好处。

# 高级架构设计

到目前为止，我们已经涵盖了在 Nest.js 中设置和开始编写和使用微服务所需的一切。在这一过程中，我们描述了 Nest.js 微服务的一些缺点。特别是，由于微服务不在单独的线程或进程中运行，使用 Nest.js 微服务时可能在性能方面并没有太多收益。

然而，并不是说你不能获得这些好处。Nest.js 只是没有提供开箱即用的工具。在大多数关于在生产环境中运行 NodeJS 应用程序的资料中，通常总是涵盖并推荐使用 NodeJS 的`cluster`模块。我们可以在我们的 Nest.js 应用程序中做同样的事情。

```js
async function bootstrapApp() {
    const app = await NestFactory.create(AppModule);

    await app.listen(process.env.PORT || 3000);
}

async function bootstrapRpc() {
    const rpcApp = await NestFactory.createMicroservice(AppModule, microserviceServerConfig('nestjs_book'));
    rpcApp.useGlobalFilters(new RpcValidationFilter());

    await rpcApp.listenAsync();
}

if (cluster.isMaster) {
    const appWorkers = [];
    const rpcWorkers = [];

    for (let i = 0; i < os.cpus().length; i++) {
        const app = cluster.fork({
            APP_TYPE: 'NestApplication'
        });
        const rpc = cluster.fork({
            APP_TYPE: 'NestMicroservice'
        });

        appWorkers.push(app);
        rpcWorkers.push(rpc);
    }

    cluster.on('exit', function(worker, code, signal) {
        if (appWorkers.indexOf(worker) > -1) {
            const index = appWorkers.indexOf(worker);
            const app = cluster.fork({
                APP_TYPE: 'NestApplication'
            });
            appWorkers.splice(index, 1, app);
        } else if (rpcWorkers.indexOf(worker) > -1) {
            const index = rpcWorkers.indexOf(worker);
            const rpc = cluster.fork({
                APP_TYPE: 'NestMicroservice'
            });
            rpcWorkers.splice(index, 1, rpc);
        }
    });
} else {
    if (process.env.APP_TYPE === 'NestApplication') {
        bootstrapApp();
    } else if (process.env.APP_TYPE === 'NestMicroservice') {
        bootstrapRpc();
    }
}

```

现在，我们的 NestApplication 和 NestMicroservice 上下文不仅在自己的线程上运行，而且根据服务器上可用的 CPU 数量进行集群化。对于每个 CPU，将创建一个单独的 NestApplication 和 NestMicroservice 上下文。NestApplication 上下文线程将共享主应用程序端口。最后，由于我们使用 RabbitMQ，运行多个 NestMicroservice 上下文，我们有多个订阅者等待微服务消息。RabbitMQ 将负责在所有 NestMicroservice 实例之间负载平衡消息分发。我们使我们的应用程序更具弹性，更能够处理比本章开始时更多的用户负载。

# 摘要

在本章开始时，我们说“微服务”是 Nest.js 的一个误导性名称。事实上，情况可能仍然如此，但这实际上取决于许多因素。我们最初使用 TCP 传输的示例几乎无法符合所有传统定义的微服务。NestApplication 和 NestMicroservice 上下文都是从同一个进程中执行的，这意味着一个的灾难性故障可能会导致两者都崩溃。

在突出 Nest.js 开箱即用的所有传输方式之后，我们在示例博客应用程序中使用自定义的 RabbitMQ 传输重新实现了我们的微服务。我们甚至将 NestApplication 和 NestMicroservice 上下文运行在自己的线程中。这是朝着实现“微服务”名称的正确方向迈出的重要一步。

尽管我们在本书中没有涵盖具体细节，但现在显而易见的是，您不仅限于在同一个 Nest.js 项目或存储库中使用微服务。使用诸如 Redis 和 RabbitMQ 之类的传输方式，我们可以创建并使用多个 Nest.js 项目，其唯一目的是执行 NestMicroservice 上下文。所有这些项目都可以独立在 Kubernetes 集群中运行，并通过 Redis 或 RabbitMQ 传递消息进行访问。更好的是，我们可以使用内置的 gRPC 传输与其他语言编写的微服务进行通信，并部署到其他平台上。

在下一章中，我们将学习 Nest.js 中的路由和请求处理。
