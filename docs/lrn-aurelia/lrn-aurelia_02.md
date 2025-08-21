# 第二章：布局、菜单和熟悉

至此，你应该已经对如何创建 Aurelia 应用程序有了很好的了解。大局可能仍然模糊，但随着我们贯穿本章，细节将不断出现。我们首先将了解依赖注入和 Aurelia 的插件系统是如何工作的，然后我们将了解如何使用、配置和自定义 Aurelia 日志记录器，以便我们可以追踪和监控我们代码中的情况。最后，我们将探讨 Aurelia 路由器和导航模型。顺便说一下，在我们开始构建真实应用程序时，我们将继续研究模板，通过创建全局布局模板及其导航菜单来构建真实应用程序。

在本书中，我们将逐步构建一个应用程序。在每一章中，我们将添加功能性和技术性特征。它从这一章开始。所以在深入技术之前，请允许我首先描述我们的应用程序将做什么。

我们将要构建一个联系人管理应用程序。这个应用程序将允许用户浏览联系人、执行搜索、创建和编辑条目。当然，它将依赖于一个 HTTP API 来管理数据。这个后端可以在 [`github.com/PacktPublishing/Learning-Aurelia`](https://github.com/PacktPublishing/Learning-Aurelia)找到；这是一个简单的基于 Node.js 的服务。只需下载它，在一个目录中解压，在该目录中打开控制台并运行 `npm install` 以恢复所需包，然后运行 `npm start` 来启动网络服务器。

接下来，你应该去使用 Aurelia CLI 创建一个空项目，最好使用默认选项。本书中的所有示例和代码样本都是使用默认 CLI 设置构建的；如果你定制项目创建或使用骨架，一些代码片段可能无法工作。因此，为了使学习过程尽可能顺利，我强烈建议你从默认设置开始。

# 依赖注入

**SOLID** 原则最早是由罗伯特·C·马丁（Robert C. Martin）在 2000 年代初提出的，他也被大家亲切地称为“Uncle Bob”。这个记忆助手的缩写后来由迈克尔·费瑟斯（Michael Feathers）提出，为这些原则的普及做出了贡献。它们描述了良好面向对象设计的核心五个关注点。尽管**SOLID**原则本身超出了本书的范围，但我们将详细讨论其中一个原则：依赖倒置。

依赖倒置原则表明类和模块应该依赖于抽象。当一个类依赖于抽象时，它无法负责创建这些依赖，它们必须被注入到对象中。这就是我们所说的**依赖注入**（**DI**）。它极大地增加了解耦和组合性，并强制执行一种在应用程序顶层或接近应用程序入口点组合对象图的编码风格。这样，应用程序的行为可以通过改变根部对象组合的方式而无需修改大量代码来改变。

然而，手动创建整个对象图，或者像 Mark Seemann 所说的“穷人版 DI”，很快就会变得单调。这就是依赖注入容器发挥作用的地方。一个 DI 容器，利用约定和配置，能够理解如何创建对象图。

在 Aurelia 中，几乎所有的对象都是由一个 DI 容器提供的。这个容器有两个责任：创建和组装对象，之后管理它们的生存周期。它可以通过使用附加到它必须实例化的类的元数据来做到这一点。

## `inject`装饰器

让我们想象一个显示人员列表的`PersonListView`组件。视图模型需要一个`PersonService`实例，用于检索一个`Person`对象列表：

`src/person-list-view.js`

```js
import {PersonService} from 'app-services'; 
import {inject} from 'aurelia-framework'; 

@inject(PersonService) 
export class PersonListView { 

  constructor(personService) { 
    this.personService = personService; 
  } 

  getPeople() { 
    return this.personService.getAll(); 
  } 
} 

```

在这里，我们有一个简单的视图模型，其构造函数期望一个`personService`参数。这个参数然后存储在一个实例变量中，以便稍后使用。视图模型还有一个`getPeople`方法，该方法调用`personService`的`getAll`方法来检索人员列表。如果你熟悉面向对象设计和依赖倒置，这里没有什么新东西。

这段代码片段中有趣的是`PersonListView`类上的`inject`装饰器。这个装饰器是从 Aurelia 导入的，指示 DI 容器在创建`PersonListView`的新实例时，解析一个`PersonService`实例，并将其作为构造函数的第一个参数注入。这里重要的是，传递给`inject`装饰器的依赖项列表与构造函数期望的参数列表一致。如果类有多个依赖项，你必须将它们全部按正确顺序传递给`inject`：

`src/person-list-view.js`

```js
import {PersonService, AnotherService} from 'app-services'; 
import {inject} from 'aurelia-framework'; 

@inject(PersonService, AnotherService) 
export class PersonListView { 

  constructor(personService, anotherService) { 
    this.personService = personService; 
    this.anotherService = anotherService; 
  } 

  getPeople() { 
    return this.personService.getAll(); 
  } 
} 

```

### 注意

装饰器是 ESNext 的一个特性；目前没有任何浏览器支持它们。此外，Babel 默认也不支持它们，所以如果你想在你的代码中使用它们，你需要添加`babel-plugin-transform-decorators-legacy`插件。使用 CLI 创建的项目已经包含了这个设置。

## TypeScript 和 autoinject

如果你使用 TypeScript，在构造函数声明中指定了每个依赖项的类型时，使用`inject`装饰器是相当冗余的。为了简化事情，Aurelia 提供了一个`autoinject`装饰器，它利用了 TypeScript 转译器添加到转译后的 JS 类中的类型元数据。

为了使用`autoinject`，你首先需要在你的`tsconfig.json`文件中将`experimentalDecorators`设置为`true`以启用装饰器和元数据发射，然后在同一文件的`compilerOptions`部分将`emitDecoratorMetadata`设置为`true`。由 CLI 创建的 TypeScript 项目已经包含了这些设置。

下面是使用 TypeScript 的相同`PersonListView`的示例：

`src/person-list-view.js`

```js
import {PersonService} from 'app-services'; 
import {Person} from 'models'; 
import {autoinject} from 'aurelia-framework'; 

@autoinject 
export class PersonListView { 

  constructor(private personService: PersonService) { 
  } 

  getPeople(){ 
    return this.personService.getAll(); 
  } 
} 

```

在这里，DI 容器知道，为了创建一个`PersonListView`实例，它首先需要解析一个`PersonService`实例并在`PersonListView`的构造函数中注入它，这要归功于`autoinject`装饰器。

## 静态 inject 方法或属性

如果你不使用 ESNext 装饰器也不是 TypeScript，或者不想在给定类内部有 Aurelia 的依赖，你可以使用返回这些依赖的静态`inject`方法声明类的依赖：

`src/person-list-view.js`

```js
import {PersonService} from 'app-services'; 

export class PersonListView { 
  static inject() { return [PersonService]; } 

  constructor(personService) { 
    this.personService = personService; 
  } 

  getPeople() { 
    return this.personService.getAll(); 
  } 
} 

```

静态的`inject`方法应该返回包含类依赖的数组。

或者，也可以支持包含依赖项数组的静态`inject`属性。实际上，当你使用`inject`或`autoinject`装饰器时，背后发生的就是这件事，它们只是将依赖项分配给类的静态`inject`属性。它们只是语法糖。

## 根容器和子容器

在 Aurelia 中，一个容器可以创建子容器，这些子容器又可以创建自己的子容器，从而形成从应用程序的根容器开始的容器树。每个子容器继承其父容器的服务，但可以注册自己的服务以覆盖父容器的服务。

如我们在第一章中看到的，*入门 *，一个应用程序从根组件开始。它也从根容器开始。当评估一个视图时，模板引擎会在每次遇到视图内的子组件时创建一个子容器，无论是自定义元素、具有自定义属性的元素还是通过路由或组合创建的视图模型。子组件的视图模型类将在子容器中注册为单例，然后用于解析子组件实例。随着这个组件的视图被加载和分析，这个过程会递归进行。随着组件被组合成树状结构，容器也是如此。

由于子容器通常是由模板引擎创建的，所以你很可能永远不需要手动创建一个子容器。不过，这里有一个例子展示了它是如何完成的：

```js
let childContainer = container.createChild(); 

```

## 解析实例

实例的解析涉及到解析器。我们稍后会回到这部分，详细解释它们是如何工作的以及如何使用，但与此同时，可以先将它们视为负责解析 DI 容器请求的类实例的策略。

解析实例时，根容器首先检查它是否已经有了一个针对该类的`Resolver`。如果有，这个`Resolver`就被用来获取一个实例。如果没有找到`Resolver`，根容器将自动注册一个单例`Resolver`对该类进行实例获取。

使用子容器解析实例时，情况有点不同。子容器仍然检查它是否有该类的`Resolver`，如果有，则使用它来获取实例。然而，如果没有找到`Resolver`，子容器将委托其父容器进行解析。父容器会重复这个过程，直到实例被解析或解析请求上升到根容器。当它这样做时，根容器按照前述方式解析实例。

这意味着当首次解析时动态注册的类的实例是应用单例，因为它们是在根容器中注册的，所以每个子容器最终都会解析为这个单一实例。

视图模型由模板引擎使用容器解析，所以你大多数时候永远不需要手动解析一个实例。然而，有一些场景你可能希望在一个对象中注入一个容器并手动解析服务。以下是这样做的方法：

```js
let personService = container.get(PersonService); 

```

在这里，`get` 方法是用 `PersonService` 类调用，并返回这个类的实例。

## 生命周期

由容器创建的任何对象都有生命周期。有三种典型的生命周期：

+   **容器单例**：当容器首次请求类时实例化该类，然后保留对该实例的引用。每当从容器中请求该类的实例时，这个相同的实例会被返回。这意味着实例的生命周期与容器的生命周期绑定。它不会被垃圾回收，直到容器被丢弃，且没有其他对象持有对实例的引用。

+   **应用单例**：作为应用单例注册的类，其实质是在应用的根容器中注册的一个容器单例，因此整个应用中都会重用同一个实例。

+   **瞬态**：当一个类被注册为瞬态时，容器每次请求实例时都会创建一个新的实例。它不会保留对任何这些实例的引用。容器仅仅作为一个工厂。

## 注册

为了解析一个类的实例，容器首先必须了解它。这个学习过程被称为注册。大多数时候，它是由容器在接收到解析请求时自动且即时执行的。它也可以通过使用容器的注册 API 手动执行。

### 容器注册 API

`Container` 类提供了多种方法用于手动注册一个类。

```js
container.registerSingleton(key: any, fn?: Function): void 

```

此方法将类注册为容器单例。`key` 将在查找时使用，`fn` 预期是要实例化的类。如果只提供 `key`，则预期它是一个类，因为它将用于查找和实例化。

例如，`container.registerSingleton(HttpClient)` 将 `HttpClient` 类注册为单例。第一次解析 `HttpClient` 时，将创建一个实例并返回。对于后续每次解析 `HttpClient` 的请求，都将返回这个单一实例。

另外，`container.registerSingleton(PersonService, CachingPersonService)` 使用 `PersonService` 作为键来注册 `CachingPersonService` 类。这意味着当解析 `PersonService` 类时，将返回 `CachingPersonService` 的单一实例。这种映射在处理抽象时非常重要。

当然，类是容器单例还是应用单例，仅仅取决于调用它的容器是否是应用的根容器。

```js
container.registerTransient(key: any, fn?: Function): void 

```

此方法将类注册为瞬态，意味着每次请求`key`时，都会创建`fn`的新实例。与`registerSingleton`类似，`fn`可以省略，在这种情况下，`key`将用于查找和实例创建。

```js
container.registerInstance(key: any, instance?: any): void 

```

此方法将现有实例注册为单例。如果你已经有一个实例并希望将其注册到容器中，这很有用。与`registerSingleton`的区别在于，传递的是实际的单例实例，而不是类。如果只提供`key`，它将用于查找和作为实例，但我真的看不到这种情况会有什么用，因为你需要已经拥有值才能查找它。

例如，`container.registerInstance(HttpClient, myClient)` 为 `HttpClient` 类注册 `myClient` 实例。每次从容器中请求 `HttpClient` 实例时，将返回 `myClient` 实例：

```js
container.registerHandler(key: any, 
  (container?: Container, key?: any, resolver?: Resolver) => any): void 

```

此方法注册一个自定义处理程序，这是一个每次容器根据`key`请求时将被调用的函数。这个处理函数将传递容器、`key` 和内部存储处理器的 `Resolver`。这支持了超出标准单例和瞬态生命周期的多种场景。

例如，`container.registerHandler(PersonService, () => new PersonService(myConfig))` 注册了一个工厂函数。每次从容器中请求一个`PersonService`实例时，该处理函数将被调用，并使用捕获的`myConfig`值创建一个新的`PersonService`实例：

```js
container.registerResolver(key: any, resolver: Resolver): void 

```

此方法注册一个自定义 `Resolver` 实例。在幕后，我们之前看到的所有容器方法都使用这个方法带有内置解析器。然而，创建我们自己的 `Resolver` 实现也是可能的。

### 注意

虽然大多数时候键是类，但它们可以是任何东西，包括字符串、数字、符号或对象。

### 自动注册

类的自动注册由以下类方法处理：

```js
container.autoRegister(key: any, fn?: Function): Resolver 

```

这个方法可以带有单一参数，即要注册的类，或者带有两个参数，第一个参数是要注册的类的键，第二个参数是要注册的类。当只有一个参数传递时，类本身被用作键。

容器在尝试解析一个找不到任何解析器的类的实例时，会自动调用`autoRegister`。它很少被应用程序直接使用。

### 注册策略

给定类的自动注册过程可以通过将`Registration`策略附加到类的元数据来定制。这可以通过使用注册装饰器之一来完成：

```js
import {transient} from 'aurelia-framework'; 

@transient() 
export class MyModel {} 

```

在这个例子中，`transient`装饰器将告诉`autoRegister`方法，`MyModel`类必须作为暂态注册，所以每次容器必须解析`MyModel`实例时，它将创建一个新的实例。

另外，你可以使用`singleton(registerInChild: boolean = false)`装饰器。当`registerInChild`参数为`false`时，默认就是这样，这个装饰器告诉`autoRegister`方法，这个类应该在根容器上注册为单例。这使得这个类成为应用程序的单例，而这本来就是容器的默认行为，所以将`singleton`与`registerInChild`设置为`false`或让其保持默认值是有点没用的。

然而，`singleton`中`registerInChild`设置为`true`表示，该类应该作为单例注册，不是在根容器上，而是在实际调用`autoRegister`方法的容器上。这允许我们装饰一个类，使得每个容器都有自己的实例：

```js
import {singleton} from 'aurelia-framework'; 

@singleton(true) 
export class MyModel {} 

```

在这个例子中，`MyModel`将被注册为容器单例。每个容器都将有自己的实例。

这两个装饰器背后依赖于`registration(registration: Registration)`。这个第三个装饰器用于将一个`Registration`策略与一个类关联。如果你创建了自己的自定义`Registration`策略，可以使用它。它被`transient`和`singleton`背后使用，将内置的`Registration`策略之一附加到它们装饰的类上。

### 创建自定义注册策略

注册策略必须实现以下方法：

```js
registerResolver(container: Container, key: any, fn: Function): Resolver 

```

默认情况下，`autoRegister`方法将传递给它的类注册为应用程序单例。然而，当被调用时，拥有附加到其元数据的`Registration`策略的类，`autoRegister`将委托该类的注册到`Registration`的`registerResolver`方法，该方法预期为该类创建一个`Resolver`，将其注册到容器中，并返回它。

通常，`registerResolver`方法实现将使用作为参数传递的`Container`实例的注册 API 来注册类。例如，内置的`TransientRegistration`类`registerResolver`方法，它被`transient`装饰器在幕后使用，看起来像这样：

```js
registerResolver(container, key, fn) { 
  return container.registerTransient(key, fn); 
} 

```

在这里，该方法调用容器的`registerTransient`方法，该方法创建一个瞬态`Resolver`，并返回它。

## 解析器

我们之前定义了`Resolver`作为负责解析实例的策略。当容器简化为最基本的形式时，它仅仅管理一个将`key`与相应的`Resolver`相关联的`Map`，这些`Resolver`是通过`Registration`策略或容器注册方法创建的。

除了在注册服务时使用解析器之外，解析器还可以在声明依赖时使用：`inject`装饰器，因此顺便说一下，`inject`静态方法或属性，可以作为`Resolver`而不是`key`传递。正如我们之前所见，在解析`key`依赖时，容器或其一个祖先将找到该`key`的`Resolver`，或者根容器将自动注册一个单例`Resolver`，这个`Resolver`将用于解析一个实例。但是，当解析一个`Resolver`依赖时，容器将直接使用这个`Resolver`来解析一个实例。这允许我们在特定注入的上下文中覆盖给定类注册的解析策略。

通常在注入时有用的大约有六个解析器。

### 懒惰

`Lazy`解析器注入一个函数，当评估时，延迟解析依赖项：

```js
import {Lazy, inject} from 'aurelia-dependency-injection'; 
import {PersonService} from 'person-service'; 

@inject(Lazy.of(PersonService)) 
Export class PersonListView { 
  constructor(personServiceAccessor) { 
    this.personServiceAccessor = personServiceAccessor; 
  } 

  getPeople() { 
    return this.personServiceAccessor().getAll(); 
  } 
} 

```

这意味着在实例创建时不会解析`PersonService`，而是在调用`personServiceAccessor`函数时解析。如果解析需要在创建对象之后而不是创建对象时进行委托，或者在对象生命周期内必须重新评估多次解析，这可能很有用。

### 全部

默认情况下，`Container`解析为与请求键匹配的第一个实例。`All`解析器允许我们注入一个包含给定键注册的所有服务的数组：

```js
import {All, inject} from 'aurelia-dependency-injection'; 
import {PersonValidator} from 'person-validator'; 

@inject(All.of(PersonValidator)) 
Export class PersonForm { 
  constructor(validators) { 
    this.validators = validators; 
  } 

  validate() { 
    for (let i = 0; i < this.validators.length; ++i) { 
      this.validators[i].validate(); 
    } 
  } 
} 

```

在这里，我们可以想象多个对象或类已经使用`PersonValidator`键进行了注册，并且它们都被作为数组注入到`PersonForm`视图模型中。

### 可选

`Optional`解析器只有在给定键已经注册时才注入实例。如果没有，它不会自动注册，而是注入`null`。第二个参数省略或设置为`true`时，使查找解析器上升到容器层次结构。如果设置为`false`，则只检查当前容器。

```js
import {Optional, inject} from 'aurelia-dependency-injection'; 
import {PersonService} from 'person-service'; 

@inject(Optional.of(PersonService, false)) 
Export class PersonListView { 
  constructor(personService) { 
    this.personService = personService; 
  } 

  getPeople() { 
    return this.personService ? this.personService.getAll() : []; 
  } 
} 

```

在这里，只有在当前容器中已经注册了`PersonService`实例时，才在`PersonListView`构造函数中注入`PersonService`的一个实例。如果没有，则注入`null`。

### 父级

`Parent`解析器跳过当前容器，从父容器开始解析。如果当前容器是根容器，则注入`null`：

```js
import {Parent, inject} from 'aurelia-dependency-injection'; 
import {PersonService} from 'person-service'; 

@inject(Parent.of(PersonService)) 
Export class PersonListView { 
  constructor(personService) { 
    this.personService = personService; 
  } 
} 

```

### 工厂

`Factory`解析器注入一个工厂函数。每次执行工厂函数时，它将请求容器中的新实例。此外，传递给这个工厂函数的任何参数都将由容器传递给类构造函数。如果类有依赖项，使用任何`inject`策略声明，额外的参数将在解析的依赖项传递到构造函数时附加：

```js
import {Factory, inject} from 'aurelia-dependency-injection'; 
import {AddressService} from 'address-service'; 

@inject(AddressService) 
class Person { 
  constructor(addressService, address) { 
    this.addressService = addressService; 
    this.address = address; 
  } 
} 

@inject(Factory.of(Person)) 
export class PersonListView { 
  constructor(personFactory) { 
    this.personFactory = personFactory; 
  } 

  createPerson(address) { 
    return this.personFactory(address); 
  } 
} 

```

在这个例子中，我们首先看到一个`Person`类被`inject`装饰器修饰，这暗示容器其构造函数需要一个`AddressService`实例作为第一个参数。我们也可以看到，构造函数实际上期望一个名为`address`的第二个参数，容器对此一无所知。接下来，我们有一个`PersonListView`类，以一种`Person`工厂在其构造函数中被注入的方式被装饰。其`createPerson`方法，传入一个`address`，用这个地址调用`Person`工厂函数。

当被调用时，为了创建一个`Person`实例，容器将首先解析一个`AddressService`实例来满足`Person`的依赖关系，然后用解析的`AddressService`实例和传递给工厂函数的`address`调用`Person`构造函数。

### 新实例

`NewInstance`解析器让容器在每次注入时创建类的全新实例，完全忽略类任何现有的注册。

```js
import {NewInstance, inject} from 'aurelia-dependency-injection'; 
import {PersonService} from 'person-service'; 

@inject(NewInstance.of(PersonService)) 
Export class PersonListView { 
  constructor(personService) { 
    this.personService = personService; 
  } 
} 

```

# 插件系统

既然我们已经对 Aurelia 中依赖注入的工作原理有了很好的理解，我们就可以开始使用它了。除了用于使用`inject`和`Resolver`创建和组合组件外，依赖注入还是 Aurelia 插件系统的核心。

## 插件

几乎 Aurelia 的每一个部分都是以插件的形式出现的。事实上，`aurelia-framework`库只是一个插件系统和配置机制，Aurelia 的其他核心库都是以这种方式 plugged into this mechanism。

一个 Aurelia 插件从`index.js`文件开始，这个文件必须导出一个`configure`函数。这个函数将在 Aurelia 启动时被调用，并接收一个 Aurelia 配置对象作为其第一个参数和一个可选的配置回调函数。

### 一个示例

让我们想象一个名为`our-plugin`的插件。这个插件首先需要在我们的`main.js`文件中的`configure`函数中启用：

**src/main.js**

```js
export function configure(aurelia) { 
  aurelia.use 
    .standardConfiguration() 
    .developmentLogging() 
    .plugin('our-plugin', config => { config.debug = true; }); 
  aurelia.start().then(() => aurelia.setRoot()); 
} 

```

在这里，除了标准的应用程序配置外，我们还告诉 Aurelia 加载`our-plugin`。我们还告诉 Aurelia 使用作为`plugin`函数第二个参数提供的回调来配置`our-plugin`。这个回调接收到由`our-plugin`定义的配置对象，我们将其`debug`属性设置为`true`。

现在让我们想象一下我们插件的`index.js`文件：

```js
export function configure(aurelia, callback) { 
  let config = { debug: false }; 
  if (typeof callback === 'function') { 
    callback(config); 
  } 
  aurelia.container.registerInstance(OurPluginConfig, config); 
} 

```

在这里，我们首先可以为我们的插件创建一个默认配置对象，如果提供了配置回调，我们将用我们的配置调用它，给插件的使用者机会更改它。然后我们可以将我们的配置对象注册为`OurPluginConfig`类的唯一实例。然后我们可以想象，由`our-plugin`暴露的服务会有这个`OurPluginConfig`的依赖，所以当它们由容器实例化时，它们会注入配置对象。

### 注册全局资源

使用这个`configure`函数，任何插件都可以注册自己的服务，甚至更改或覆盖其他插件声明的服务。它还可以为模板引擎注册资源：

```js
export function configure(aurelia) { 
  aurelia.globalResources('./my-component'); 
} 

```

在这里，一个插件注册了一个名为`my-component`的资源。这个资源可能有很多不同的事物；我们将在下一章节中覆盖模板资源。

## 特性

插件是组织和解除代码耦合的好方法。但是插件作为项目依赖存在于外部库中。例如，在使用 CLI 时，插件位于`node_modules`目录中。在典型项目中，那里的代码不受版本控制。这部分代码不应作为项目的一部分进行修改。实际上它不属于项目；它由其他人管理，或者至少在一个不同的项目工作流中。

但是，如果我们想要像这样结构自己的应用程序怎么办呢？使用插件机制会使这变得相当复杂，因为我们需要将不同的插件视为不同的项目，并单独打包它们，然后在应用程序中安装它们。每次需要更改插件中的任何一个时，都需要单独进行更改，然后发布并更新应用程序中的依赖关系。尽管有时共享在多个项目中使用的通用组件或行为很有用，但这种工作流程在非必要时增加了开发过程的复杂性。

幸运的是，Aurelia 有一个解决方案，即特性。特性与插件完全一样工作，但它位于应用程序内部。让我们看一个例子：

`src/my-feature/index.js`

```js
export function configure(aurelia) { 
  // register some services or resources used by this feature 
} 

```

**src/main.js**

```js
export function configure(aurelia) { 
  aurelia.use 
    .standardConfiguration() 
    .developmentLogging() 
    .feature('my-feature'); 
  aurelia.start().then(() => aurelia.setRoot()); 
} 

```

特性工作方式和插件完全一样，不同之处在于我们使用`feature`方法而不是`plugin`方法来加载它们，并且它们位于`src`目录内。像插件一样，特性预期在其根目录下有一个`index.js`文件，该文件应导出一个`configure`函数。像插件一样，它可以传递一个配置回调作为`feature`方法的第二个参数，这个回调将传递给特性的`configure`函数。

`feature`方法期望相对路径到包含`index.js`文件的目录。例如，如果我的特性位于`src/some/path/index.js`，加载它的调用将是`feature('some/path')`。

特性是组织代码的好方法。它们使你更容易将可能是一个巨大、单块的应用程序分解成一系列设计良好的模块。当然，这都取决于开发团队的设计技能。在第六章，*设计关注 - 组织和解耦*，我们将介绍一些模式、策略和组织代码的方法，以构建更好的 Aurelia 应用程序。

# 日志记录

Aurelia 带有一个简单而强大的日志系统。它支持日志级别和可插拔的附加器。

## 配置

为了配置日志，至少必须添加一个日志附加器：

`**src/main.js**`

```js
import * as LogManager from 'aurelia-logging'; 
import {ConsoleAppender} from 'aurelia-logging-console'; 

export function configure(aurelia) { 
  aurelia.use.standardConfiguration(); 

  LogManager.addAppender(new ConsoleAppender()); 
  LogManager.setLevel(LogManager.logLevel.info); 

  aurelia.start().then(() => aurelia.setRoot()); 
}; 

```

在这里，首先向日志模块添加了`ConsoleAppender`实例，该实例从`aurelia-logging-console`库导入。这个附加器简单地将日志输出到浏览器的控制台。

为了使日志工作，至少必须添加一个附加器。如果没有添加附加器，日志将被简单丢弃。

接下来，日志级别被设置为`info`。这意味着所有较低级别的日志不会被分发到附加器。Aurelia 支持四个日志级别，从最低到最高：`debug`、`info`、`warn`和`error`。例如，将最小日志级别设置为`warn`意味着`debug`和`info`日志将被忽略。此外，还有一个`none`日志级别可用。当设置时，它简单地执行没有任何过滤，并将所有日志分发到附加器。

### 默认配置

上一个示例旨在展示一个完全自定义的设置。相反，你可以在配置应用程序时使用`developmentLogging`方法：

`**src/main.js**`

```js
export function configure(aurelia) { 
  aurelia.use 
    .standardConfiguration() 
    .developmentLogging(); 

  aurelia.start().then(() => aurelia.setRoot()); 
}; 

```

这个默认配置安装了`ConsoleAppender`，并将日志级别设置为`none`。

## 附加器

附加器必须实现一个简单的接口，每个日志级别有一个方法。例如，以下是 Aurelia 的`ConsoleAppender`实现：

```js
export class ConsoleAppender { 
  debug(logger, ...rest) { 
    console.debug(`DEBUG [${logger.id}]`, ...rest); 
  } 

  info(logger, ...rest) { 
    console.info(`INFO [${logger.id}]`, ...rest); 
  } 

  warn(logger, ...rest) { 
    console.warn(`WARN [${logger.id}]`, ...rest); 
  } 

  error(logger, ...rest) { 
    console.error(`ERROR [${logger.id}]`, ...rest); 
  } 
} 

```

正如你所看到的，每个方法首先接收初始化日志的日志器，然后是传递给日志器的日志方法的参数。

## 写日志

为了写日志，你首先需要获取一个日志器：

```js
import {LogManager} from 'aurelia-framework'; 
const logger = LogManager.getLogger('my-logger'); 

```

`getLogger`方法期望日志器的名称，并返回日志器实例。如果为提供的名称不存在日志器，则会创建一个新的。日志器是单例，所以对于给定的名称始终返回相同的实例。

一旦你有一个日志器实例，你可以调用它的四个日志方法之一：`debug()`、`info()`、`warn()`或`error()`。每个这些方法都将调用所有附加器的相应级别方法，假设方法日志级别等于或高于配置的最小日志级别。否则，附加器不会被调用，日志将被丢弃。

日志器方法可以传递任意数量的参数，这些参数将被分发到附加器。例如，当在日志器上调用`error('A message', 12)`时，调用将被委派给附加器的`appender.error`(`logger, 'A message', 12)`。

默认情况下，所有日志记录器都使用全局日志级别进行配置。然而，日志记录器还具有一个`setLevel`方法，允许为单个日志记录器设置不同的日志级别：

```js
logger.setLevel(LogManager.logLevel.warn); 

```

# 路由

除了非常简单的情况外，一个典型的单页应用程序通常由多个视图组成。大多数时候，这样的应用程序有一个固定的全局布局，包括一个显示当前视图的可变区域和一个允许用户从一个视图导航到另一个视图的菜单。在 Aurelia 中，这些功能由路由器插件支持。

## 配置路由器

为了启用路由，请确保您的应用程序依赖于`aurelia-router`和`aurelia-templating-router`库，就像基于 CLI 的项目那样默认依赖。然后在你`main.js`文件的`configure`函数中加载路由插件， either by loading the whole `standardConfiguration()`, which includes the router, or by loading the `router()`individually. 有关如何在应用程序`configure`函数中加载插件的更多信息，请参阅第一章，*入门*。

## 声明路由

我们将从向我们的根组件添加一个`configureRouter`方法开始。当 Aurelia 检测到组件上的这个回调方法时，它会在组件初始化周期中调用它。这个方法接收两个参数：一个路由配置对象和路由本身：

`src/app.js`

```js
export class App { 
  configureRouter(config, router) { 
    this.router = router; 
    config.title = 'Learning Aurelia'; 
    config.map([ 
      { route: ['', 'contacts'], name: 'contacts', moduleId: 'contact-list', nav: true, title: 'Contacts' }, 
      { route: 'contacts/:id', name: 'contact-details', moduleId: 'contact-details' }, 
    ]); 
  } 
} 

```

在`configureRouter`方法中，我们首先将路由器分配给一个实例变量。这很重要，因为我们的根组件的视图需要访问路由器以渲染菜单和活动路由组件。

一旦完成，我们设置全局标题。这个值将显示在浏览器标题栏中。

接下来，我们使用`map`方法配置两个路由。路由配置基本上是将一个 URL 路径模式与一个组件的映射，当路径匹配时激活路由，并在路由激活时显示组件。它还包含其他属性。让我们分解一个路由配置：

+   `route`属性是 URL 路径模式。重要的是要注意，这些模式省略了路径的前斜杠。有三种类型的模式：

    +   **静态路由**：该模式完全匹配路径。我们第一个路由的第一个模式是这种模式的例子：它匹配根路径（`/`），由于省略了前斜杠，它匹配空字符串。这使得它成为默认路由。

    +   **参数化路由**：该模式完全匹配路径，并且与占位符匹配的路径部分（前缀为冒号`:`）被解析为路由参数。这些参数的值在屏幕激活生命周期中作为路由组件的一部分提供。我们第二个路由的模式是这种模式的例子：它匹配以`/contacts/`开头的路径，后跟第二个部分，被解释为联系人的`id`。

        ### 注意

        此外，可以通过在参数后添加一个问号使其成为可选参数。 例如，`contacts/:id?/details` 模式将匹配 `/contacts/12/details` 和 `/contacts/details` 两者。 当在路径中省略参数时，传递给路由组件的相应参数将是 `undefined`。

    +   **通配符路由**：该模式匹配路径的开始部分，路径的其余部分被视为一个单一参数，其值在屏幕激活生命周期中作为路由组件的一部分提供。 例如，`my-route*param` 模式将匹配任何以 `/my-route` 开头的路径，`param` 将是 一个参数，其值是匹配到的路径的其余部分。

+   `name` 属性唯一标识路由。 我们稍后可以看到如何使用它来生成路由的 URL。

+   `moduleId` 属性是路由组件的路径。

+   `nav` 属性，当设置为 `true` 值时，告诉路由器将此路由包含在其导航模型中，该模型用于自动构建应用程序的导航菜单。另外，如果 `nav` 是一个数字，则路由器将使用它来对导航菜单中的项目进行排序。

+   `title` 属性在路由活动时将显示在浏览器标题栏中，除非组件覆盖它。 如果 `nav` 是 `true`，它也用作路由的菜单项文本。

+   `settings` 属性是可选的，可以包含激活组件或管道步骤可以使用任意数据，我们将在本章后面看到。

### 重定向路由

代替 `moduleId`，路由可以声明一个 `redirect` 属性。 当这样的路由被激活时，路由器将执行内部重定向到代表该属性值的路径。 这允许用多个模式技术声明默认路由的替代方法，正如我们第一个路由所展示的那样。 相反，我们可以声明以下路由：

```js
config.map([ 
  { route: '', redirect: 'contacts' }, 
  { route: 'contacts', name: 'contacts', moduleId: 'contact-list', nav: true, title: 'Contacts' }, 
  { route: 'contacts/:id', name: 'contact-details', moduleId: 'contact-details' }, 
]); 

```

与这个配置的主要区别是，当访问 `/` 时，浏览器地址栏中的 URL 将更改为 `/contacts`，因为路由器将执行重定向。

使用此模式时，`nav` 属性应该只在目标路由上设置为 `true`。 如果它在重定向路由上设置而不是目标路由，那么路由器将无法突出显示相应的菜单项，因为该路由在目标路由激活之前仅短暂激活片刻。 最后，在重定向路由及其目标路由上都设置为 `true` 会导致两者都在菜单中渲染，这是没有意义的，因为它们都通向同一个地方。

如果 `nav` 属性是 `false`，那么设置 `title` 也是没有意义的，因为该路由从未激活足够长的时间以至于标题可见。

然而，为重定向路由设置`name`可能是有用的。当重定向预期在未来会改变时，可以使用重定向路由的`name`来生成链接，而不是目标路由的。这样，路由的`redirect`属性是唯一需要改变的东西，依赖于这个路由的每一个链接都会随之改变。

### 导航策略

除了`moduleId`和`redirect`属性之外，路由还可以有一个`navigationStrategy`属性。其值必须是一个函数，该函数将由路由器调用，并传递一个`NavigationInstruction`实例。然后可以动态地配置这个对象。例如，我们的最后一个路由可以配置成这样：

```js
{ 
  route: 'contacts/:id', name: 'contact-details',  
  navigationStrategy: instruction => { 
    instruction.config.moduleId = 'contact-details'; 
  } 
} 

```

最后，这个路由做的和之前一样。但对于需要比`moduleId`和`redirect`更灵活的场景，这个替代方案可以变得很有用，因为`NavigationInstruction`实例包含以下属性：

+   `config`：正在导航到的路由的配置对象

+   `fragment`：触发导航的 URL 路径

+   `params`：包含从路由模式中提取的每个参数的对象

+   `parentInstruction`：如果这个路由是一个子路由，则是指令父路由的指令

+   `plan`：由路由器内部构建并使用以执行导航的导航计划

+   `previousInstruction`：当前指令将在路由器中替换的导航指令

+   `queryParams`：包含从查询字符串解析出的值的对象

+   `queryString`：原始查询字符串

+   `viewPortInstructions`：视口指令，由路由器内部构建并使用以执行导航

## 布局我们的应用程序

基于其路由配置，路由器生成一个导航模型，可以用来自动生成导航菜单。因此，当添加新路由时，我们不需要改变路由的配置和菜单视图。

由于我们根组件的视图模型负责声明路由，它的视图应该是全局布局并渲染导航菜单。让我们使用这个导航模型来创建根组件的视图：

`src/app.html`

```js
<template> 
  <require from="app.css"></require> 
  <nav class="navbar navbar-default navbar-fixed-top" role="navigation"> 
    <div class="navbar-header"> 
      <button type="button" class="navbar-toggle" data-toggle="collapse" 
              data-target="#skeleton-navigation-navbar-collapse"> 
        <span class="sr-only">Toggle Navigation</span> 
      </button> 
      <a class="navbar-brand" href="#"> 
        <i class="fa fa-home"></i> 
        <span>${router.title}</span> 
      </a> 
    </div> 

    <div class="collapse navbar-collapse" id="skeleton-navigation-navbar-collapse"> 
      <ul class="nav navbar-nav"> 
        <li repeat.for="row of router.navigation" class="${row.isActive ? 'active' : ''}"> 
          <a data-toggle="collapse" data-target="#skeleton-navigation-navbar-collapse.in" href.bind="row.href"> 
            ${row.title} 
          </a> 
        </li> 
      </ul> 

      <ul class="nav navbar-nav navbar-right"> 
        <li class="loader" if.bind="router.isNavigating"> 
          <i class="fa fa-spinner fa-spin fa-2x"></i> 
        </li> 
      </ul> 
    </div> 
  </nav> 
  <div class="page-host"> 
    <router-view></router-view> 
  </div> 
</template> 

```

这个模板中突出显示的部分是最有趣的部分。让我们来看一下。

首先要注意的是，我们需要一个名为`app.css`的文件，我们将在一会儿写它。这个文件将样式化我们的应用程序组件。

接下来，视图使用了`router`属性，该属性定义在我们根组件的视图模型的`configureRouter`方法中。我们首先在带有`nav-brand`类的`a`标签中看到它，其中字符串插值指令渲染文档标题。

然后，我们在`li`标签上发现了一个`repeat.for="row of router.navigation"`属性。这个绑定指令为`router.navigation`数组中的每个项目重复`li`标签。这个`navigation`属性包含了路由器的导航模型，该模型是用路由的 truthy `nav`属性构建的。在渲染每个`li`标签时，模板引擎的绑定上下文中都有一个包含当前导航模型项的`row`变量。

`li`标签还有一个`class="${row.isActive ? 'active' : ''}"`属性。这个字符串插值指令使用当前导航模型项的`isActive`属性。如果`isActive`评估为`true`值，它就会给`li`标签分配一个`active` CSS 类。这个属性由路由器管理，仅当导航模型项属于活动路由时才是`true`。在这个模板中，它用来突出显示活动菜单项。

`li`标签内的锚点有一个`href.bind="row.href"`属性。这个指令将标签的`href`属性绑定到当前导航模型项的`href`属性。这个`href`属性是由路由器使用路由的路径模式构建的。此外，在锚点内部，还渲染了路由的`title`。

在菜单的末尾，我们可以看到一个带有`loader` CSS 类的`li`标签。这个元素包含一个旋转图标。它有一个`if.bind="router.isNavigating"`属性，它将这个元素在 DOM 中的存在与路由器的`isNavigating`属性的值绑定在一起。这意味着当路由器执行导航时，顶部的右角将显示一个旋转图标。当没有导航发生时，这个图标不仅不可见，实际上甚至根本不在 DOM 中，感谢`if`属性。

最后，`router-view`元素作为路由视图 port，显示活动路由组件。这是整个模板中唯一必需的部分。当一个组件配置路由器时，其视图必须包含一个`router-view`元素，否则将抛出错误。利用导航模型是可选的，菜单可以是静态的，或者通过任何你能想象到的其他方式构建。显示标题也是可选的。利用`isNavigating`指示器绝对是完全不必要的。然而，如果一个组件配置了路由器，而它的视图却不能显示活动路由组件，那么这个组件配置路由器就是毫无意义的。

这个视图使用了一种结构，如果你曾经使用过 Bootstrap，你可能就会熟悉。Bootstrap 是由 Twitter 开发的 CSS 框架，我们将在我们的应用程序中使用它。让我们来安装它：

```js
> npm install bootstrap --save

```

我们还需要在我们的应用程序中加载它：

`index.html`

```js
<!DOCTYPE html> 
<html> 
  <head> 
    <title>Learning Aurelia</title> 
    <link href="node_modules/bootstrap/dist/css/bootstrap.min.css" rel="stylesheet"> 
  </head> 
  <!-- Omitted snippet... --> 
</html> 

```

我们的`app`组件在能正常工作之前还缺最后一块拼图，那就是`app.css`文件。文件内容如下：

`src/app.css`

```js
.page-host { 
  position: absolute; 
  left: 0; 
  right: 0; 
  top: 50px; 
  bottom: 0; 
  overflow-x: hidden; 
  overflow-y: auto; 
} 

```

### 尝试一下

至此，如果你运行我们的应用程序，你应在浏览器的控制台看到一个路由错误。那是因为默认路由试图加载`contact-list`组件，而这个组件还不存在。

让我们创建一个空的文件：

`src/contact-list.html`

```js
<template> 
<h1>Contacts</h1> 
</template> 

```

`src/contact-list.js`

```js
export class ContactList {} 

```

现在如果你再次尝试运行应用程序，你应该看到应用程序正确加载，显示顶部菜单和空的 `contact-list` 组件。

## 屏幕激活生命周期

当路由器检测到 URL 路径发生变化时，它会经历以下生命周期：

1.  确定目标路由。如果没有任何路由与新路径匹配，将抛出一个错误，并且在这里停止处理过程。

1.  给活动路由组件一个拒绝停用的机会，在这种情况下，路由器恢复之前的 URL 并在这里停止处理过程。

1.  给目标路由组件一个拒绝激活的机会，在这种情况下，路由器恢复之前的 URL 并在这里停止处理过程。

1.  停用活动路由组件。

1.  激活目标路由组件。

1.  视图被交换。

为了加入这个生命周期，组件可以实现以下任意一个回调方法：

+   `canActivate(params, routeConfig, navigationInstruction)`：在步骤 #2 时调用，以知道组件是否可以被激活。可以返回一个 `boolean` 值、一个 `Promise` 类型的 `boolean` 值、一个导航命令，或者一个 `Promise` 类型的导航命令。

+   `activate(params, routeConfig, navigationInstruction)`：在步骤 #5 时调用，当组件被激活时。可以返回一个可选的 `Promise`。

+   `canDeactivate()`：在步骤 #3 时调用，以知道组件是否可以被停用。可以返回一个 `boolean` 值、一个 `Promise` 类型的 `boolean` 值、一个导航命令，或者一个 `Promise` 类型的导航命令。

+   `deactivate()`：在步骤 #4 时调用，当组件被停用时。可以返回一个可选的 `Promise`。

`Promise` 在整个生命周期中都是被支持的。这意味着当回调方法中的任何一个返回一个 `Promise` 时，路由器会在继续处理之前等待其解决。

此外，`canActivate` 和 `activate` 都接收与导航上下文相关的参数：

+   `params` 对象将有一个属性，用于每个解析的路由模式中的参数，以及每个查询字符串值的属性。例如，我们的 `contact-details` 组件将接收一个具有 `id` 属性的 `params` 对象。在匹配路径中没有值的可选参数将被设置为 `undefined`。

+   `routeConfig` 将是原始的路由配置对象，具有一个额外的 `navModel` 属性。这个 `navModel` 对象有一个 `setTitle(title: string)` 方法，该方法可以被组件用来将文档标题更改为动态值，如激活期间加载的数据。我们将在第三章中看到更多内容，*显示数据*。

+   `navigationInstruction` 是路由器用来执行导航的 `NavigationInstruction` 实例。

最后，`canDeactivate`和`canActivate`都可以如果它们返回`false`、一个解析为`false`的`Promise`、一个导航命令或一个解析为导航命令的`Promise`来取消导航。

## 导航命令

导航命令是一个具有`navigate(router: Router)`方法的对象。当从`canDeactivate`或`canActivate`返回导航命令时，路由器取消当前导航并将控制权委托给命令。Aurelia 自带一个导航命令：`Redirect`。这是一个使用它的示例：

`src/contact-details.js`

```js
import {inject} from 'aurelia-framework'; 
import {Redirect} from 'aurelia-router'; 
import {ContactService} from 'app-services'; 

@inject(ContactService) 
export class ContactDetails { 
  constructor(contactService) { 
    this.contactService = contactService; 
  } 

  canActivate(params) { 
    return this.contactService.getById(params.id) 
      .then(contact => { this.contact = contact; }) 
      .catch(e => new Redirect('error')); 
  } 
} 

```

在这里，在`canActivate`回调方法中，`ContactDetails`视图模型尝试通过其`id`加载联系人。如果由`getById`返回的`Promise`被拒绝，用户将被重定向到`error`路由。

## 处理未知路由

当路由器无法将 URL 路径与任何路由匹配时，它会抛出一个错误。但在提出这个错误之前，它首先将导航指令委托给一个未知的路由处理程序，如果有的话。此处理程序可以通过使用`mapUnknownRoutes`方法进行配置，该方法可以接受以下值之一作为参数：

+   组件显示的路径，而不是抛出错误。

+   路由配置对象，包含`moduleId`、`redirect`或`navigationStrategy`属性之一。路由器将委托导航到此路由，而不是抛出错误。

+   一个接收`NavigationInstruction`实例并返回要显示的组件路径而不是抛出错误的函数。

让我们实现一个`not-found`组件，当链接断裂时，我们的应用程序将显示它：

`src/not-found.html`

```js
<template> 
  <h1>Something is broken...</h1> 
  <p>The page cannot be found.</p> 
</template> 

```

`src/not-found.js`

```js
export class NotFound {} 

```

在我们的根组件中，我们只需要添加突出显示的行：

`src/app.js`

```js
export class App { 
  configureRouter(config, router) { 
    this.router = router; 
    config.title = 'Learning Aurelia';  
    config.map([ /* omitted for brevity */ ]); 
    config.mapUnknownRoutes('not-found'); 
  } 
} 

```

任何时候路由器无法将 URL 路径与现有路由匹配，我们的`not-found`组件都将显示。

### 约定路由

`mapUnknownRoutes`提供的另一个选项是使用路由约定而不是一组静态定义的路由。如果你的所有路由都遵循路径和`moduleId`之间的相同命名模式，我们可以想象这样的事情：

`src/app.js`

```js
export class App { 
  configureRouter(config, router) { 
    this.router = router; 
    config.title = 'Learning Aurelia'; 
    config.mapUnknownRoutes(instruction => getComponentForRoute(instruction.fragment)); 
  } 
} 

```

在这里，路由依赖于一个由`getComponentForRoute`函数实现的约定，该函数接收触发导航的 URL 路径，并返回必须显示的组件路径。

## 激活策略

当多个静态路由导致相同的组件，并在这些路由之间发生导航时，路由器只是保持相同的组件实例。由于这个原因，激活生命周期不会执行。这种行为由激活策略决定。`activationStrategy`枚举有两个值：

+   `replace`：用新路由替换当前路由，保持相同的组件实例，不经过激活生命周期。这是默认行为。

+   `invokeLifecycle`：即使活动组件没有变化，也要经历激活生命周期。

改变这种行为有两种方法：

+   在路由的配置对象中，你可以添加一个`activationStrategy`属性，指定激活此路由时应使用哪种策略。

+   在路由组件的视图模型中，你可以添加一个`determineActivationStrategy`方法，该方法必须返回所有显示此组件的路由所使用的策略。

## 子路由 (Child routers)

就像 DI 一样，容器可以有子容器，形成一个容器树；就像组件可以包含子组件，形成一个组件树，路由器也可以有子路由器。这意味着一个路由组件的视图模型可以有自己的`configureRouter`方法，其视图有一个`router-view`元素。当遇到这样的组件时，路由器将为这个子组件创建一个子路由器。这个子路由器的路由模式相对于父路由的模式是相对的。

这使得应用程序可以有一个具有多级层次的导航树。在讨论如何组织大型应用程序时，我们将会看到如何利用这一特性，请参阅第六章，*设计关注点 - 组织和解耦*。

## 管道 (Pipelines)

可能很有必要将路由器与一些在每次发出导航请求时都会被调用的逻辑连接起来。例如，具有认证机制的应用程序可能需要将某些路由限制为仅限认证用户。Aurelia 路由器的管道正是为这类场景而设计的。

路由器支持四个管道：`authorize`、`preActivate`、`preRender`和`postRender`。这些管道在导航过程中的不同阶段被调用。让我们看看它们各自发生在哪里：

1.  如果存在的话，当前路由组件的`canDeactivate`方法会被调用。

1.  执行`authorize`管道。

1.  如果存在的话，目标路由组件的`canActivate`方法会被调用。

1.  执行`preActivate`管道。

1.  如果存在的话，当前路由组件的`deactivate`方法会被调用。

1.  如果存在的话，目标路由组件的`activate`方法会被调用。

1.  执行`preRender`管道。

1.  在路由视口中交换视图。

1.  执行`postRender`管道。

管道由步骤组成，这些步骤按顺序调用。管道步骤是一个具有`run(instruction, next)`方法类的实例，其中`instruction`是一个`NavigationInstruction`实例，`next`是一个`Next`对象。

`Next`对象是一个具有方法的对象。

当调用`next()`时，它告诉路由器管道继续执行下一个步骤。`next.cancel()`方法取消了导航过程，并期望传递一个导航命令或`Error`对象作为参数。

两者都返回`Promise`。

让我们看一个例子：

`src/app.js`

```js
import {AuthenticatedStep} from 'authenticated-step'; 

export class App { 
  configureRouter(config, router) { 
    config.title = 'Aurelia'; 
    config.addPipelineStep('authorize', AuthenticatedStep); 
    config.map([ 
      { route: 'login', name: 'login', moduleId: 'login', title: 'Login' }, 
      { route: 'management', name: 'management', moduleId: 'management',  
        settings: { secured: true } }, 
    ]); 
    this.router = router; 
  } 
} 

```

这里需要注意的是，`AuthenticatedStep`类被添加到了`authorize`管道中。管道步骤作为类添加，而不是实例。这是因为路由使用其 DI 容器来解析步骤的实例。这允许步骤有依赖关系，这些依赖关系在执行前被解析和注入。

第二个要注意的是，`management`路由有一个`settings`对象，其`secured`属性被设置为`true`。它将由以下片段中的管道步骤使用，以识别需要对已认证用户限制的路由。

`src/authenticated-step.js`

```js
import {inject} from 'aurelia-framework'; 
import {Redirect} from 'aurelia-router'; 
import {User} from 'user'; 

@inject(User) 
export class AuthenticatedStep { 
  constructor(user) { 
    this.user = user; 
  } 

  run(instruction, next) { 
    let isRouteSecured = instruction.getAllInstructons().some(i => i.config.settings.secured); 
      if (isRouteSecured && !this.user.isAuthenticated) { 
      return next.cancel(new Redirect('login')); 
    } 
    return next(); 
  } 
} 

```

这是实际的管道步骤。在这个例子中，我们可以想象我们的应用程序包含一个`User`类，它暴露了当前用户的信息。我们的管道依赖于这个类的实例，以知道当前用户是否已认证。

`run`方法首先检查指令中的任何路由是否被配置为安全。这是通过检查所有导航指令，包括潜在父路由的指令，并检查其配置的`settings`中的真值`secured`属性来实现的。

例如，当导航到前一个代码片段中定义的`management`路由时，`isRouteSecured`的值将被设置为`true`。如果`management`组件声明了子路由，并且导航是对其中之一进行的，那么情况也会如此。在这种情况下，即使子路由没有被配置为`secured`，`isRouteSecured`仍将是`true`，因为其中一个父路由将是`secured`。

当目标路由或其之一被设置为安全时，如果用户未认证，导航将被取消，用户将被重定向到`login`路由。否则，调用`next`，让路由器知道它可以继续导航过程。

## 事件

Aurelia 路由还提供了另一个扩展点。除了屏幕激活生命周期和管道之外，路由还通过事件聚合器发布事件，这是 Aurelia 的核心库之一。

可以在`samples/chapter-2/router-events`中找到路由事件的演示。让我们看看这些事件：

+   `router:navigation:processing`：每次路由开始处理导航指令时，都会触发此事件。

+   `router:navigation:error`：当导航指令触发错误时，会触发此事件。

+   `router:navigation:canceled`：当导航指令被取消时，会触发此事件，取消可以是当前或目标路由组件的屏幕激活生命周期回调方法之一，或者是管道步骤。

+   `router:navigation:success`：当导航指令成功时，会触发此事件。

+   `router:navigation:complete`：一旦导航指令的处理完成，无论它失败、被取消还是成功，都会触发此事件。

所有这些事件的负载都包含一个`NavigationInstruction`实例，作为`instruction`属性存储。此外，除了`router:navigation:processing`之外，其他事件的所有负载都有一个`PipelineResult`作为`result`属性。例如，在处理`error`事件时，可以使用`result`属性的`output`属性来访问被抛出的`Error`对象。

我们将在第六章中看到事件聚合器是如何工作的，*设计关注 - 组织和解耦*。

## 多个视口

在所有之前的示例中，`router-view`元素从未有过任何属性。它实际上可以有一个`name`属性。当省略这个属性时，由元素声明在路由器上的视口被称为`default`。您看到这里的含义了吗？

如果你回答路由支持多个视口，那你猜对了。当然，这也意味着每个声明在视图中的视口都必须为每个视口配置路由。让我们看看这是如何工作的：

### 注意

以下代码片段摘自`samples/chapter-2/router-multiple-viewports`。

`src/app.html`

```js
<template> 
  <require from="nav-bar.html"></require> 
  <require from="bootstrap/css/bootstrap.css"></require> 

  <nav-bar router.bind="router"></nav-bar> 

  <div class="page-host"> 
    <router-view name="header"></router-view> 
    <router-view name="content"></router-view> 
  </div> 
</template> 

```

在组件的视图中，有趣的是注意到有两个`router-view`元素，有不同的`name`属性。这个组件的路由最终会有两个视口：一个名为`header`，另一个名为`content`：

`src/app.js`

```js
export class App { 
  configureRouter(config, router) { 
    config.title = 'Learning Aurelia'; 
    config.map([ 
      { 
        route: ['', 'page-1'], name: 'page-1', nav: true, title: 'Page 1',  
        viewPorts: {  
          header: { moduleId: 'header' },  
          content: { moduleId: 'page-1' } 
        } 
      }, 
      { 
        route: 'page-2', name: 'page-2', nav: true, title: 'Page 2',  
        viewPorts: {  
          header: { moduleId: 'header' },  
          content: { moduleId: 'page-2' } 
        } 
      }, 
    ]); 

    this.router = router; 
  } 
} 

```

在视图模型的`configureRouter`回调方法中，两个路由都使用特定的`moduleId`进行了配置，既适用于`header`，也适用于`content`和`viewPorts`。

如果在路由激活时没有为每个路由的视口配置路由，则路由器将抛出错误。无论是否静态地使用`viewPorts`属性为每个视口定义了`moduleId`，还是`viewPorts`属性通过`navigationStrategy`动态配置，都不重要。在前一个示例中，`page-2`路由可以被替换为：

```js
{ 
  route: 'page-2', name: 'page-2', nav: true, title: 'Page 2',  
  navigationStrategy: instruction => { 
    instruction.config.viewPorts = { 
      header: { moduleId: 'header' },  
      content: { moduleId: 'page-2' } 
    }; 
  } 
} 

```

此路由与前一个示例中的效果相同。这里唯一的区别是，每次路由激活时都会动态地配置视口。

当然，重定向路线不会受到视口的影响，因为它们不会渲染任何内容。

## 状态推送与哈希变化

路由器通过响应 URL 的变化来工作。在旧浏览器中，只有 URL 中的#符号后面的部分，即哈希部分，可以改变而不触发页面重载。因此，在这些浏览器上运行的路由器只能更改哈希部分，并监听哈希部分的更改。

随着 HTML5 的推出，一个新的历史 API 被引入，以实现对浏览器历史的操作。这使得运行在现代浏览器上的 JavaScript 路由器可以直接操作其当前 URL 和浏览历史，并监控当前 URL 的变化。这个 API 使得路由器能够使用完整的 URL，并允许诸如同构应用之类的技术，具有服务器渲染和渐进增强。这些技术可以使应用程序的内容能够被更广泛的客户端访问，同时也将提高应用程序的 SEO，因为谷歌已经弃用了基于哈希的带有 AJAX 内容加载的应用（参见[`googlewebmastercentral.blogspot.com/2015/10/deprecating-our-ajax-crawling-scheme.html`](https://googlewebmastercentral.blogspot.com/2015/10/deprecating-our-ajax-crawling-scheme.html)）。

### 注意

当一个应用程序可以在客户端和服务器上执行时，它被称为同构应用程序。通常，同构应用程序在服务器端执行，以渲染基于文本的 HTML 表示，然后可以返回给客户端；例如，搜索引擎爬虫。当在客户端执行时，它通常通过运行时事件处理程序、数据绑定和实际行为进行增强，以便用户可以与应用程序互动。

奥雷利亚（Aurelia）的路由插件可以与这两种策略中的任何一种工作。默认情况下，它被配置为使用基于哈希的策略，因为状态推送需要服务器相应地配置。此外，基于哈希的策略支持不完全兼容 HTML5 的旧浏览器。

然而，如果不需要支持旧浏览器，或者需要服务器端渲染，并且应用程序可能会向同构方向发展，路由器可以配置为使用历史 API。

### 注意

下面的代码片段是`samples/chapter-2/router-push-state`的摘录。

首先，在`index.html`文件中，在头部部分，必须添加一个`<base href="/">`标签。这个元素指示浏览器`/`是页面中所有相对 URL 的基础。

接下来，在根组件的视图模型中，路由必须配置不同：

`src/app.js`

```js
export class App { 
  configureRouter(config, router) { 
    this.router = router; 
    config.title = 'Aurelia'; 
    config.options.pushState = true; 
    config.options.hashChange = false; 
    config.map([ /* omitted for brevity */ ]); 
  } 
} 

```

此外，为了在用户使用除根 URL 以外的 URL 访问应用程序时显示正确的路由，服务器需要输出`index.html`页面，而不是对未知路径的 404 响应。这样，当用户访问应用程序的路由时，服务器将响应 index 页面，然后应用程序启动，路由器将处理路由并显示正确的视图。这意味着应用程序中的路由和服务器端资源（如 CSS、图片、字体、JS、HTML 或必须由 index 页面或应用程序从服务器加载的任何文件）之间必须没有命名冲突。

## 生成 URL

路由器能够对 URL 的变化做出反应，并相应地更新其视口，这是一件事。但是关于允许它导航的链接呢？如果我们硬编码 URL，任何路由路径模式的更改都需要更改路由配置，还要检查用于导航的每个地方，无论是 JS 代码还是视图，并修改它。

幸运的是，路由器也能够在生成 URL。要生成一个路由路径，有两个要求：

+   路由配置必须有一个唯一的`name`属性

+   如果路由具有参数化或通配符模式，生成 URL 时必须提供包含每个参数值的参数对象。

### 在代码中

要在 JS 代码中生成 URL 路径，你首先必须有一个路由器的实例，通常是通过在需要它的类中注入它来获得的。然后，你可以调用以下方法：

```js
router.generate(name: string, params?: any, options?: any): string 

```

必须使用路由名称、路由有时的参数对象以及可选的选项对象调用此方法，并将返回生成的 URL。目前唯一支持的选择是`absolute`，当设置为`true`时，强制路由器返回绝对 URL 而不是相对 URL。

例如，对于路径模式为`contacts/:id`的名为`contact-details`的路由，为 id 为 12 的联系人生成 URL 的调用将是：

```js
let url = router.generate('contact-details', { id: 12 }); 

```

而对于绝对 URL：

```js
let url = router.generate('contact-details', { id: 12 }, { absolute: true }); 

```

### 在视图中

如果我们需要在视图中渲染一个指向我们路由的链接怎么办？我猜你可以看到如何将路由器注入视图模型中，调用`generate`方法，并将锚点的`href`属性与结果数据绑定。在最坏的情况下，这会很快变得繁琐。

`aurelia-templating-router`库带有一个`route-href`属性，这使得这变得容易得多。例如，要为名为`contact-details`的路由渲染一个到 id 为 12 的联系人链接的模板片段将是：

```js
<a route-href="route: contact-details; params.bind: { id: 12 }"> 
  Contact #12</a> 

```

机会很大，ID 不会被硬编码，而是存储在一个对象中：

```js
<a route-href="route: contact-details; params.bind: { id: contact.id }"> 
  ${contact.name}</a> 

```

默认情况下，`route-href`属性会将生成的 URL 分配给它所在元素的`href`属性，但它支持一个`attribute`属性，可以用来指定必须设置 URL 的属性名称：

```js
<q route-href="route: quote; attribute: cite">...</q> 

```

在这里，`quote`路由的 URL 将被分配给`q`元素的`cite`属性。

## 导航

路由器提供了方便的方法，可以从 JS 代码执行导航：

+   `navigate(fragment: string, options?: any): boolean`：导航到新的位置，其路径为`fragment`。如果导航成功，则返回`true`，否则返回`false`。目前支持两个`options`：

    +   `replace: boolean`：如果设置为`true`，新 URL 将替换历史记录中的当前位置，而不是添加到历史记录中。

    +   `trigger: boolean`：如果设置为`false`，Aurelia 的路由器将不会被触发。这意味着如果 URL 是相对的，它会在浏览器的地址栏中更改，但实际上不会发生导航。

+   `navigateToRoute(name: string, params?: any, options?: any): boolean`：方便地包装了对`generate`的调用，然后是`navigate`。

+   `navigateBack(): void`：返回历史记录中的上一个位置。

# 摘要

依赖注入是 Aurelia 的核心，因此理解其工作方式很重要。如果你在本章之前对这个概念不熟悉，一下子可能接受不了这么多；但请放心，由于我们将在书的剩余部分大量使用这些功能，这将帮助你更加熟悉它。

插件、功能和路由也是如此。我们将在书的后面继续深入研究这些主题，特别是在第六章，*设计关注 - 组织和解耦*，当我们讨论各种应用程序结构的实现方式时。

在到达那里之前，我们还有很多内容需要学习。在下一章，我们将讨论数据绑定和模板的基础知识，并将组件添加到我们的联系人管理应用程序中以获取和显示数据。
