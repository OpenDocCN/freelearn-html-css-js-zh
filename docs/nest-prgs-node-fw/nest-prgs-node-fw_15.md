# 第十五章：使用 Angular Universal 进行服务器端渲染

如果您对用于客户端应用程序开发的 Angular 平台不熟悉，值得一看。Nest.js 与 Angular 有着独特的共生关系，因为它们都是用 TypeScript 编写的。这允许在 Nest.js 服务器和 Angular 应用程序之间进行一些有趣的代码共享，因为 Angular 和 Nest.js 都使用 TypeScript，可以在两者之间创建一个共享的包中的类。然后可以将这些类包含在任一应用程序中，并帮助保持在客户端和服务器之间通过 HTTP 请求发送和接收的对象一致。当我们引入 Angular Universal 时，这种关系被提升到另一个层次。Angular Universal 是一种技术，允许在服务器上预渲染您的 Angular 应用程序。这有许多好处，比如：

1.  为了便于 SEO 目的的网络爬虫。

1.  提高网站的加载性能。

1.  提高低性能设备和移动设备上网站的性能。

这种技术称为服务器端渲染，可以非常有帮助，但需要对项目进行一些重构，因为 Nest.js 服务器和 Angular 应用程序是按顺序构建的，当请求获取网页时，Nest.js 服务器实际上会运行 Angular 应用程序本身。这本质上模拟了浏览器中的 Angular 应用程序，包括 API 调用和加载任何动态元素。这个在服务器上构建的页面现在作为静态网页提供给客户端，动态的 Angular 应用程序在后台静默加载。

如果您现在刚开始阅读本书，并希望跟随示例存储库进行操作，可以使用以下命令进行克隆：

```js
git clone https://github.com/backstopmedia/nest-book-example

```

Angular 是另一个可以写一整本书的主题。我们将使用一个已经由作者之一改编用于本书的 Angular 6 应用程序。原始存储库可以在这里找到。

```js
https://github.com/patrickhousley/nest-angular-universal.git

```

这个存储库使用了 Nest 5 和 Angular 6，所以进行了一些更改，因为这本书是基于 Nest 4 的。不过不用担心，我们在本章开头展示的主要存储库中包含了一个 Angular Universal 项目。它可以在项目的根目录下的`universal`文件夹中找到。这是一个独立的 Nest + Angular 项目，而不是将主要存储库适应这本书的 Angular 应用，我们将其隔离出来，以提供一个清晰简洁的示例。

# 使用 Nest.js 为 Angular Universal 应用提供服务

现在我们将使用 Nest.js 服务器来提供 Angular 应用程序，我们需要将它们编译在一起，这样当我们运行 Nest.js 服务器时，它就知道在哪里查找 Universal 应用程序。在我们的`server/src/main.ts`文件中，有一些关键的东西需要在那里。在这里我们创建一个`bootstrap()`函数，然后从下面调用它。

```js
async function bootstrap() {
  if (environment.production) {
    enableProdMode();
  }

  const app = await NestFactory.create(ApplicationModule.moduleFactory());

  if (module.hot) {
    module.hot.accept();
    module.hot.dispose(() => app.close());
  }

  await app.listen(environment.port);
}

bootstrap()
  .then(() => console.log(`Server started on port ${environment.port}`))
  .catch(err => console.error(`Server startup failed`, err));

```

让我们逐行分析这个函数。

```js
if (environment.production) {
    enableProdMode();
  }

```

这告诉应用程序为应用程序启用生产模式。在编写 Web 服务器时，生产模式和开发模式之间有许多不同之处，但如果您想在生产环境中运行 Web 服务器，这是必需的。

```js
const app = await NestFactory.create(ApplicationModule.moduleFactory());

```

这将创建类型为`INestApplication`的 Nest 应用程序变量，并将在`app.module.ts`文件中使用`ApplicationModule`运行。`app`将是在`environment.port`端口上运行的 Nest 应用程序的实例，可以在`src/server/environment/environment.ts`中找到。这里有三个不同的环境文件：

1.  `environment.common.ts`-正如其名称所示，这个文件在生产和开发构建之间是共用的。它提供了关于在服务器和客户端应用程序中找到打包构建文件的信息和路径。

1.  `environment.ts`-这是在开发过程中使用的默认环境，并包括`environment.common.ts`文件中的设置，以及将`production: false`和上面提到的端口设置为 3000。

1.  `environment.prod.ts`-这个文件与#2 相似，只是它设置了`production: true`，并且没有定义端口，而是默认使用默认端口，通常是 8888。

如果我们在本地开发并且想要进行热重载，即如果我们更改文件，则服务器将重新启动，那么我们需要在我们的`main.ts`文件中包含以下内容。

```js
if (module.hot) {
  module.hot.accept();
  module.hot.dispose(() => app.close());
}

```

这是在`webpack.server.config.ts`文件中设置的，基于我们的`NODE_ENV`环境变量。

最后，要实际启动服务器，调用我们的`INestApplication`变量上的`.listen()`函数，并传递一个端口来运行。

```js
await app.listen(environment.port);

```

然后我们调用`bootstrap()`，这将运行上面描述的函数。在这个阶段，我们现在有了我们的 Nest 服务器正在运行，并且能够提供 Angular 应用程序并监听 API 请求。

在上面的`bootstrap()`函数中，当创建`INestApplication`对象时，我们提供了`ApplicationModule`。这是应用程序的入口点，处理 Nest 和 Angular Universal 应用程序。在`app.module.ts`中我们有：

```js
@Module({
  imports: [
    HeroesModule,
    ClientModule.forRoot()
  ],
})
export class ApplicationModule {}

```

在这里，我们导入了两个 Nest 模块，`HeroesModule`，它将为*英雄之旅*应用程序提供 API 端点，以及`ClientModule`，它是处理 Universal 的模块。`ClientModule`有很多内容，但我们将重点介绍处理设置 Universal 的主要内容，这是这个模块的代码。

```js
@Module({
  controllers: [ClientController],
  components: [...clientProviders],
})
export class ClientModule implements NestModule {
  constructor(
    @Inject(ANGULAR_UNIVERSAL_OPTIONS)
    private readonly ngOptions: AngularUniversalOptions,
    @Inject(HTTP_SERVER_REF) private readonly app: NestApplication
  ) {}

  static forRoot(): DynamicModule {
    const requireFn = typeof __webpack_require__ === "function" ? __non_webpack_require__ : require;
    const options: AngularUniversalOptions = {
      viewsPath: environment.clientPaths.app,
      bundle: requireFn(join(environment.clientPaths.server, 'main.js'))
    };

    return {
      module: ClientModule,
      components: [
        {
          provide: ANGULAR_UNIVERSAL_OPTIONS,
          useValue: options,
        }
      ]
    };
  }

  configure(consumer: MiddlewareConsumer): void {
    this.app.useStaticAssets(this.ngOptions.viewsPath);
  }
}

```

我们将从文件顶部的`@Module`装饰器开始。与常规的 Nest.js 模块（还有 Angular，记得 Nest.js 是受 Angular 启发的吗？）一样，有`controllers`（用于端点）属性和`components`（用于服务、提供者和其他我们想要作为此模块一部分的组件）属性。在这里，我们在`controllers`数组中包括`ClientController`，在`components`中包括`...clientProviders`。这里的三个点（`...`）本质上意味着“将数组中的每个元素插入到这个数组中”。让我们更详细地解释一下这些。

`ClientController`

```js
@Controller()
export class ClientController {
  constructor(
    @Inject(ANGULAR_UNIVERSAL_OPTIONS) private readonly ngOptions: AngularUniversalOptions,
  ) { }

  @Get('*')
  render(@Res() res: Response, @Req() req: Request) {
    res.render(join(this.ngOptions.viewsPath, 'index.html'), { req });
  }
}

```

这与我们学到的任何其他控制器都是一样的，但有一个小小的不同。在 URL 路径`/*`上，Nest.js 服务器不是提供 API 端点，而是从之前在环境文件中看到的相同的`viewsPath`中呈现一个 HTML 页面，即`index.html`。

至于`clientProoviders`数组：

```js
export const clientProviders = [
  {
    provide: 'UNIVERSAL_INITIALIZER',
    useFactory: async (
      app: NestApplication,
      options: AngularUniversalOptions
    ) => await setupUniversal(app, options),
    inject: [HTTP_SERVER_REF, ANGULAR_UNIVERSAL_OPTIONS]
  }
];

```

这类似于我们在`ClientModule`的返回语句中定义自己的提供者，但是我们使用`useFactory`而不是`useValue`，这将 Nest 应用程序和我们之前定义的`AngularUniversalOptions`传递给`setupUniversal(app, options)`函数。我们花了一段时间，但这就是实际创建 Angular Universal 服务器的地方。

`setupUniversal(app, options)`

```js
export function setupUniversal(
  app: NestApplication,
  ngOptions: AngularUniversalOptions
) {
  const { AppServerModuleNgFactory, LAZY_MODULE_MAP } = ngOptions.bundle;

  app.setViewEngine('html');
  app.setBaseViewsDir(ngOptions.viewsPath);
  app.engine(
    'html',
    ngExpressEngine({
      bootstrap: AppServerModuleNgFactory,
      providers: [
        provideModuleMap(LAZY_MODULE_MAP),
        {
          provide: APP_BASE_HREF,
          useValue: `http://localhost:${environment.port}`
        }
      ]
    })
  );
}

```

这里调用了三个主要函数：`app.setViewEngine()`，`app.setBaseViewDir()`和`app.engine`。第一个`.setViewEngine()`将视图引擎设置为 HTML，以便引擎呈现视图时知道我们正在处理 HTML。第二个`.setBaseViewDir()`告诉 Nest.js 在哪里找到 HTML 视图，这同样是之前在`environment.common.ts`文件中定义的。最后一个非常重要，`.engine()`定义了要使用的 HTML 引擎，在这种情况下，因为我们使用的是 Angular，它是`ngExpressEngine`，这是 Angular Universal 引擎。在这里阅读更多关于 Universal express-engine 的信息：[`github.com/angular/universal/tree/master/modules/express-engine`](https://github.com/angular/universal/tree/master/modules/express-engine)。这将`bootstrap`设置为`AppServerModuleNgFactory`对象，这将在下一节中讨论。

在`ClientModule`中，我们可以看到在我们在`AppliationModule`（服务器入口点）中导入`ClientModule`时调用的`.forRoot()`函数。基本上，`forRoot()`定义了一个要返回的模块，以取代最初导入的`ClientModule`，也称为`ClientModule`。返回的这个模块有一个单一组件，提供了`ANGULAR_UNIVERSAL_OPTIONS`，这是一个定义将传递到组件的`useValue`属性中的对象类型的接口。

`ANGULAR_UNIVERSAL_OPTIONS`的结构是：

```js
export interface AngularUniversalOptions {
  viewsPath: string;
  bundle: {
    AppServerModuleNgFactory: any,
    LAZY_MODULE_MAP: any
  };
}

```

由此可见，`useValue`的值是在`forRoot()`顶部定义的`options`的内容。

```js
const options: AngularUniversalOptions = {
  viewsPath: environment.clientPaths.app,
  bundle: requireFn(join(environment.clientPaths.server, 'main.js'))
};

```

`environment.clientPaths.app`的值可以在我们之前讨论过的`environment.common.ts`文件中找到。作为提醒，它指向编译后的客户端代码的位置。也许你会想为什么`bundle`的值是一个 require 语句，而接口明确表示它应该是这样的结构：

```js
bundle: {
    AppServerModuleNgFactory: any,
    LAZY_MODULE_MAP: any
  };

```

好吧，如果你追溯这个 require 语句（`..`表示向上一级目录），那么你会看到我们将`bundle`属性设置为另一个模块`AppServerModule`。稍后会讨论这个，但是 Angular 应用程序最终将被提供。

`ClientModule`中的最后一部分是`configure()`函数，它将告诉服务器在哪里找到静态资产。

```js
configure(consumer: MiddlewareConsumer): void {
    this.app.useStaticAssets(this.ngOptions.viewsPath);
  }

```

# 构建和运行 Universal App

现在你已经设置好了 Nest.js 和 Angular 文件，几乎可以运行项目了。有一些需要你注意的配置文件，可以在示例项目中找到：[`github.com/backstopmedia/nest-book-example`](https://github.com/backstopmedia/nest-book-example)。到目前为止，我们一直在使用`nodemon`运行项目，这样我们的更改就会在保存项目时反映出来，但是，现在我们正在打包它以提供一个 Angular 应用程序，我们需要使用不同的包来构建服务器。为此，我们选择了`udk`，这是一个`webpack`扩展。它既可以构建我们的生产包，也可以启动一个开发服务器，就像`nodemon`为我们的普通 Nest.js 应用程序所做的那样。熟悉以下配置文件是个好主意：

1.  `angular.json`-我们的 Angular 配置文件，处理诸如使用哪个环境文件、可以与`ng`一起使用的命令以及 Angular CLI 命令等事项。

1.  `package.json`-项目全局依赖和命令文件。该文件定义了生产和开发所需的依赖项，以及命令行工具（如`yarn`或`npm`）可用的命令。

1.  `tsconfig.server.json`-这是全局`tsconfig.json`文件的扩展，它提供了一些 Angular 编译器选项，比如在哪里找到 Universal 入口点。

# 总结

就是这样！我们有一个可以玩耍的 Angular Universal 项目。Angular 是一个很棒的客户端框架，最近一直在蓬勃发展。在这一章中只是浅尝辄止，特别是在 Angular 本身方面，还有很多工作可以做。

这是本书的最后一章。我们希望你会兴奋地使用 Nest.js 来创建各种应用程序。
