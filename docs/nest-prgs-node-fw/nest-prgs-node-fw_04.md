# 第四章：Nest.js 的依赖注入系统

本章概述了依赖注入（DI）模式，这是今天最大的框架经常使用的一种方式。这是一种保持代码清晰且易于使用的方法。通过使用此模式，您最终会得到更少耦合的组件和更多可重用的组件，这有助于加快开发过程时间。

在这里，我们将研究在模式存在之前使用注入的方法，以及注入如何随着时间的推移而改变，以使用 TypeScript 和装饰器的现代方法进行 Nest.js 注入。您还将看到显示此类模式优势的代码片段，以及框架提供的模块。

Nest.js 在架构上基于 Angular，并用于创建可测试、可扩展、松耦合和易于维护的应用程序。与 Angular 一样，Nest.js 有自己的依赖注入系统，这是框架的`core`的一部分，这意味着 Nest.js 不太依赖第三方库。

# 依赖注入概述

自`Typescript 1.5`引入装饰器的概念以来，您可以使用装饰器在不同对象或属性上提供的添加元数据进行`元编程`，例如`class`，`function`，`function parameters`或`class property`。元编程是使用描述对象的元数据编写一些代码或程序的能力。这种类型的程序允许您使用其自身的元数据修改程序的功能。在我们的情况下，这些元数据对我们很有兴趣，因为它有助于将某些对象注入到另一个对象中，其名称为依赖注入。

通过使用装饰器，您可以在与这些装饰器相关联的任何对象或属性上添加元数据。例如，这将定义接受装饰器的对象的类型，但它还可以定义函数所需的所有参数，这些参数在其元数据中描述。要获取或定义任何对象上的元数据，您还可以使用`reflect-metadata`库来操纵它们。

## 为什么使用依赖注入

使用依赖注入的真正好处在于，依赖对象与其依赖项之间的耦合度更低。通过提供注入器系统的框架，您可以管理对象而无需考虑它们的实例化，因为这由注入器来管理，后者旨在解决每个依赖对象的依赖关系。

这意味着更容易编写测试和模拟依赖项，这些测试更清晰和更易读。

## 没有依赖注入的情况下如何运作

让我们想象一个需要注入`UserService`的`AuthenticationService`。

这里是`UserService`：

```js
export class UserService() {
    private users: Array<User> = [{
        id: 1,
        email: 'userService1@email.com',
        password: 'pass'
    ]};

    public findOne({ where }: any): Promise<User> {
        return this.users
        .filter(u => {
            return u.email === where.email &&
            u.password === where.password;
        });
    }
}

```

还有`AuthenticationService`，它实例化所需的`UserService`：

```js
export class AuthenticationService {
    public userService: UserService;

    constructor() {
        this.userService = new UserService();
    }

    async validateAUser(payload: { email: string; password: string }): Promise<boolean> {
        const user = await this.userService.findOne({
            where: payload
        });
        return !!user;
    }
}

```

```js
const authenticationService = new AuthenticationService();

```

正如您所看到的，您必须在类本身中管理所有相关的依赖项，以便在`AuthenticationService`内部使用。

这种方法的缺点主要是`AuthenticationService`的不灵活性。如果要测试此服务，您必须考虑其自身的隐藏依赖项，当然，您不能在不同的类之间共享任何服务。

## 使用手动依赖注入的工作原理

现在让我们看看如何使用先前的`UserService`通过构造函数传递依赖项。

```js
// Rewritted AuthenticationService
export class AuthenticationService {
    /* 
 Declare at the same time the public 
 properties belongs to the class
 */
    constructor(public userService: UserService) { }
}

```

```js
// Now you can instanciate the AutheticationService like that
const userService = new UserService();
const authenticationService = new AuthenticationService(userService);

```

您可以轻松地通过所有对象共享`userService`实例，而不再是`AuthenticationService`必须创建`UserService`实例。

这使生活变得更容易，因为注入器系统将允许您执行所有这些操作，而无需实例化依赖项。让我们在下一节中使用前面的类来看看这一点。

## 依赖注入模式今天

今天，要使用依赖注入，你只需要使用 Typescript 提供的装饰器系统，并由你想要使用的框架实现。在我们的案例中，正如你将在工具章节中看到的那样，Nest.js 提供了一些装饰器，它们几乎什么都不做，只是在它们将被使用的对象或属性上添加一些元数据。

这些元数据将帮助框架意识到这些对象可以被操作，注入所需的依赖关系。

以下是`@Injectable()`装饰器的使用示例：

```js
@Injectable()
export class UserService { /*...*/ }

@Injectable()
export class AuthenticationService {
    constructor(private userService: UserService) { }
}

```

这个装饰器将被转译，并且将向其添加一些元数据。这意味着在类上使用装饰器后，你可以访问`design:paramtypes`，这允许注入器知道依赖于`AuthenticationService`的参数的类型。

通常，如果你想创建自己的类装饰器，这个装饰器将以`target`作为参数，表示你的类的`type`。在前面的例子中，`AuthenticationService`的类型就是`AuthenticationService`本身。这个自定义类装饰器的目的将是将目标注册到服务的`Map`中。

```js
export Component = () => {
    return (target: Type<object>) => {
        CustomInjector.set(target);
    };
}

```

当然，你已经看到了如何将服务注册到服务的 Map 中，那么让我们看看这可能是一个自定义注入器。这个注入器的目的将是将所有服务注册到 Map 中，并解决对象的所有依赖关系。

```js
export const CustomInjector = new class {
  protected services: Map<string, Type<any>> = new Map<string, Type<any>>();

  resolve<T>(target: Type<any>): T {
    const tokens = Reflect.getMetadata('design:paramtypes', target) || [];
    const injections = tokens.map(token => CustomInjector.resolve<any>(token));
    return new target(/*...*/injections);
  }

  set(target: Type<any>) {
    this.services.set(target.name, target);
  }
};

```

因此，如果你想实例化我们的`AuthenticationService`，它依赖于超级`UserService`类，你应该调用注入器来解决依赖关系，并返回所需对象的实例。

在下面的例子中，我们将通过注入器解决`UserService`，并将其传递到`AuthenticationService`的构造函数中，以便能够实例化它。

```js
const authenticationService = CustomInjector.resolve<AuthenticationService>(AuthenticationService);
const isValid = authenticationService.validateUser(/* payload */);

```

# Nest.js 依赖注入

从`@nestjs/common`中，你可以访问框架提供的装饰器，其中之一就是`@Module()`装饰器。这个装饰器是构建所有模块并在它们之间使用 Nest.js 依赖注入系统的主要装饰器。

你的应用程序将至少有一个模块，即主模块。在小型应用程序的情况下，应用程序可以只使用一个模块（主模块）。然而，随着应用程序的增长，你将不得不创建多个模块来为主模块安排应用程序。

从主模块中，Nest 将知道你已经导入的所有相关模块，然后创建应用程序树来管理所有的依赖注入和模块的范围。

为了做到这一点，`@Module()`装饰器遵循`ModuleMetadata`接口，该接口定义了允许配置模块的属性。

```js
export interface ModuleMetadata {  
    imports?: any[];  
    providers?: any[];  
    controllers?: any[];  
    exports?: any[];
    modules?: any[]; // this one is deprecated.
}

```

要定义一个模块，你必须注册所有存储在`providers`中的服务，这些服务将由 Nest.js 的`injector`实例化，以及可以注入提供者的`controllers`，这些提供者是通过`exports`属性注册到模块中的服务，或者由其他模块导出的服务。在这种情况下，这些服务必须在`imports`中注册。

如果一个模块没有导出可注入的内容，并且导出模块没有被导入到使用外部服务的相关模块中，那么就无法访问另一个模块中的可注入内容。

***Nest.js 如何创建依赖注入树？***

在前一节中，我们谈到了主模块，通常称为`AppModule`，它用于从`NestFactory.create`创建应用程序。从这里，Nest.js 将不得不注册模块本身，并且还将遍历导入到主模块的每个模块。

Nest.js 然后会为整个应用程序创建一个`container`，其中包含整个应用程序的`module`，`globalModule`和`dynamicModuleMetadata`。

在创建了容器之后，它将初始化应用程序，并在初始化期间实例化一个 `InstanceLoader` 和一个 `DependenciesScanner -> scanner.ts`，通过它，Nest.js 将有可能扫描与每个模块和元数据相关的所有模块。它这样做是为了解决所有的依赖关系，并生成所有模块和服务的实例及其自己的注入。

如果你想了解引擎的细节，我们建议你深入了解两个类：`InstanceLoader` 和 `DependenciesScanner`。

为了更好地理解这是如何工作的，看一个例子。

想象一下，你有三个模块：

+   `ApplicationModule`

+   `AuthenticationModule`

+   `UserModule`

应用程序将从 `ApplicationModule` 创建：

```js
@Module({
    imports: [UserModule, AuthenticationModule]
})
export class ApplicationModule {/*...*/}

```

这导入了 `AuthenticationModule`：

```js
@Module({
    imports: [UserModule],
    providers: [AuthenticationService]
})
export class AuthenticationModule {/*...*/}

@Injectable()
export class AuthenticationService {
    constructor(private userService: UserService) {}
}

```

以及 `UserModule`：

```js
@Module({
    providers: [UserService],
    exports: [UserService]
})
export class UserModule {/*...*/}

@Injectable()
export class UserService {/*...*/}

```

在这种情况下，`AuthenticationModule` 必须导入 `UserModule`，后者导出 `UserService`。

我们现在已经构建了应用程序的架构模块，并且需要创建应用程序，它将允许解决所有的依赖关系。

```js
const app = await NestFactory.create(ApplicationModule);

```

基本上，当你创建应用程序时，Nest.js 将：

+   扫描模块。

+   存储模块和一个空的作用域数组（用于主模块）。然后将作用域填充为导入此扫描模块的模块。

+   查看通过 `modules` 元数据相关的模块。

+   扫描模块的依赖项作为服务、控制器、相关模块和导出项，将它们存储在模块中。

+   将所有全局模块绑定到每个模块中的相关模块。

+   通过解析原型创建所有的依赖项，为每个依赖项创建一个实例。对于具有自己依赖项的依赖项，Nest.js 将以相同的方式解析它们，并将其包含在前一级中。

***全局模块呢？***

Nest.js 还提供了一个 `@Global()` 装饰器，允许 Nest 将它们存储在全局模块的 `Set` 中，并将其添加到相关模块的 `Set` 中。

这种类型的模块将使用 `__globalModule__` 元数据键进行注册，并添加到容器的 globalModule 集合中。然后它们将被添加到相关模块的 `Set` 中。有了全局模块，你可以允许将模块中的组件注入到另一个模块中，而无需将其导入到目标模块中。这避免了将一个可能被所有模块使用的模块导入到所有模块中。

这是一个例子：

```js
@Module({
    imports: [DatabaseModule, UserModule]
})
export class ApplicationModule {/*...*/}

```

```js
@Global()
@Module({
    providers: [databaseProvider],
    exports: [databaseProvider]
})
export class DatabaseModule {/*...*/}

```

```js
@Module({
    providers: [UserService],
    exports: [UserService]
})
export class UserModule {/*...*/}

@Injectable()
export class UserService {
    // SequelizeInstance is provided by the DatabaseModule store as a global module
    constructor(@Inject('SequelizeInstance') private readonly sequelizeInstance) {}
}

```

有了之前的所有信息，你现在应该对 Nest.js 依赖注入的机制很熟悉，并且对它们如何一起工作有了更好的理解。

# Nest.js 和 Angular DI 之间的区别

即使 Nest.js 在很大程度上基于 Angular，它们之间存在一个重大区别。在 Angular 中，每个服务都是单例，这与 Nest.js 相同，但是可以要求 Angular 提供服务的新实例。在 Angular 中，你可以使用 `@Injectable()` 装饰器的 `providers` 属性来注册模块中的提供者的新实例，并且仅对该组件可用。这对于避免通过不同组件覆盖某些属性非常有用。

# 总结

因此，总结一下，我们在本章中看到了如何在不使用依赖注入的情况下，对象是多么不灵活和难以测试。此外，我们还了解了如何实现依赖项注入的方法的演变，首先是通过将依赖项实现到依赖项中，然后通过手动将它们传递到构造函数来改变方法，最终到达注入器系统。然后通过解析树自动在构造函数中解析依赖项，这就是 Nest.js 如何使用这种模式。

在下一章中，我们将看到 Nest.js 如何使用 TypeORM，这是一个与多种不同关系数据库一起工作的对象关系映射（ORM）。
