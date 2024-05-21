# 第五章：带有 GraphQL 和 Apollo 的 Angular ToDo 应用程序

从客户端到服务器之间有许多不同的通信数据的方式。在本章中，我们将看看如何使用 GraphQL 从服务器中提取数据，然后从 Angular 客户端发送和改变数据。我们还将看看如何利用 GraphQL 中的计算值。在上一章的内容基础上，我们将再次使用 Angular Material 作为用户界面，以了解如何使用 Angular 路由来提供不同的内容。

本章将涵盖以下主题：

+   理解 GraphQL 与 REST 的关系

+   创建可重用的数据库类

+   数据预填和使用单例

+   创建 GraphQL 模式

+   使用`type-graphql`设置 GraphQL 类型

+   使用查询和变更创建 GraphQL 解析器

+   将 Apollo Server 用作我们的应用程序服务器

+   创建 GraphQL Angular 客户端应用程序

+   为客户端添加 Apollo 支持

+   在 Angular 中使用路由

+   使用 Angular 验证控制输入

+   从客户端向服务器发送 GraphQL 变更

+   从客户端向服务器发送 GraphQL 查询

+   在只读和可编辑模板之间切换

# 技术要求

完成的项目可以从[`github.com/PacktPublishing/Advanced-TypeScript-3-Programming-Projects/tree/master/Chapter05`](https://github.com/PacktPublishing/Advanced-TypeScript-3-Programming-Projects/tree/master/Chapter05)下载。

下载项目后，您将需要使用`npm install`命令安装软件包要求。

# 理解 GraphQL 与 REST 的关系

基于网络技术的一大优点是解决常见问题的多种方式。在 REST 中，我们使用了一种简单但强大的方式从客户端到服务器进行通信；然而，这并不是我们可以做到的唯一方式。REST 解决了一系列问题，但也引入了新问题，新技术已经出现来解决这些问题。需要解决的三个问题如下：

+   为了构建复杂的信息，我们可能需要对 REST 服务器进行多次调用。例如，对于购物应用程序，我们可能使用一个 REST 调用来获取一个人的姓名，另一个 REST 调用来获取他们的地址，还需要第三个调用来获取他们的购物篮详情。

+   随着时间的推移，我们可能会经历多个版本的 REST API。让我们的客户端跟踪版本可能会有限制，这意味着在 API 的最开始，我们还必须定义我们的版本体验将会是什么样子。除非我们的 API 遵循相同的版本标准，否则这可能会导致混乱的代码。

+   这些 REST 调用最终可能会带来比我们实际需要的信息更多。因此，当我们进行这些详细的调用时，我们实际上只需要 20 或 30 个字段中的三四个信息。

关于 REST 的一件事是要理解它实际上并不是一种技术。理解 REST 的一个好方法是，它更像是一个约定的架构标准，可以使用几乎任何传输机制作为通信手段。好吧，澄清一下，虽然我说它是一个标准，但实际上很少有人真正遵循 REST 的原始概念，这意味着我们也需要理解开发人员的意图。例如，当我们通过 REST 发出更新时，我们是使用`PUT HTTP`动词还是`POST`动词？如果我们想要使用第三方 API，了解这个细节是至关重要的。

GraphQL 最初由 Facebook 开发，但现在由 GraphQL 基金会维护（[`foundation.graphql.org/`](https://foundation.graphql.org/)），GraphQL 是解决这些问题的好机制。与纯 REST 不同，GraphQL 只是一个带有工具支持的查询语言。GraphQL 围绕着我们的代码与字段交互的想法，只要有获取这些字段的定义，我们就可以编写任意复杂的查询，以一次获取多个位置的数据，或者对数据进行变异以更新它。一个设计良好的 GraphQL 系统可以处理版本需求以及可以根据需要添加和弃用的字段。

使用 GraphQL，我们只能通过查询检索我们需要的信息。这样可以避免我们在客户端级别过度订阅信息。同样，我们的查询可以为我们从多个位置拼接结果，这样我们就不必执行多次往返。我们从客户端发送查询，让我们的 GraphQL 服务器检索相关的数据项。我们也不必担心客户端代码中的 REST 端点。我们只需与我们的 GraphQL 服务器通信，让查询处理数据。

在本章中，我们将看看如何使用 Apollo GraphQL 引擎（[`www.apollographql.com/`](https://www.apollographql.com/)）和非常有用的`TypeGraphQL`库（[`typegraphql.ml/`](https://typegraphql.ml/)），它提供了一种方便的方式来从 TypeScript 中针对 GraphQL。使用 Apollo，我们有一个完整的前后端基础设施来完全管理我们的 GraphQL 行为。除了提供客户端库外，我们还可以在服务器上使用 Apollo，以及用于 iOS 和 Android 应用程序。

请注意，GraphQL 并不打算完全取代 RESTful 服务。有许多情况下，我们希望 REST 和 GraphQL 并存。例如，我们可能有一个与我们的 GraphQL 实现通信并为我们缓存信息的 REST 服务。但是，在本章中，我们将专注于纯粹创建 GraphQL 实现。

# 项目概述

在本章中，我们的项目将向我们介绍如何编写 GraphQL 应用程序，包括服务器端和客户端。我们还将开始研究 TypeScript 3 中引入的功能，以创建一个 ToDo 应用程序。我们将扩展上一章的 Angular 概念，引入客户端路由，这将允许我们显示不同的内容并有效地在页面之间导航。我们还将介绍 Angular 验证。

与 GitHub 代码一起工作，本章的任务应该需要大约四个小时才能完成。

完成后，应用程序应如下所示：

![](img/afdfcce2-295e-4a37-9db3-bd1916ce19a9.png)

# 开始项目

就像上一章一样，本章将使用 Node.js（可从[`nodejs.org`](https://nodejs.org)获取）。我们还将使用以下组件：

+   Angular CLI（我使用的版本是 7.2.2）

+   `express`（版本 4.16.4 或更高）

+   `mongoose`（版本 5.4.8 或更高）

+   `@types/cors`（版本 2.8.4 或更高）

+   `@types/body-parser`（版本 1.17.0 或更高）

+   `@types/express`（版本 4.16.0 或更高）

+   `@types/mongodb`（版本 3.1.19 或更高）

+   `@types/mongoose`（版本 5.3.11 或更高）

+   `type-graphql`（版本 0.16.0 或更高）

+   `@types/graphql`（版本 14.0.7 或更高）

+   `apollo-server`（版本 2.4.0 或更高）

+   `apollo-server-express`（版本 2.4.0 或更高）

+   `guid-typescript`（版本 1.0.9 或更高）

+   `reflect-metadata`（版本 0.1.13 或更高）

+   `graphql`（版本 14.1.1 或更高）

+   `apollo-angular`（版本 1.5.0 或更高）

+   `apollo-angular-link-http`（版本 1.5.0 或更高）

+   `apollo-cache-inmemory`（版本 1.4.3 或更高）

+   `apollo-client`（版本 2.4.13 或更高）

+   `graphql`（版本 14.1.1 或更高）

+   `graphql-tag`（版本 2.10.1 或更高版本）

除了使用 MongoDB，我们还将使用 Apollo 来提供 GraphQL 数据。

# 使用 GraphQL 和 Angular 创建 ToDo 应用程序

像现在一样，我们将从定义需求开始：

+   用户必须能够添加由标题、描述和任务到期日期组成的 ToDo 任务

+   验证将确保这些项目始终设置，并且到期日期不能早于今天

+   用户将能够查看所有任务的列表

+   用户将能够删除任务

+   用户将能够查看过期任务（过期任务是指尚未完成且到期日期已过的任务）

+   用户将能够编辑任务

+   使用 GraphQL 传输数据到服务器，或从服务器传输数据

+   传输的数据将保存到 MongoDB 数据库中

# 创建我们的应用程序

对于我们的 ToDo 应用程序，我们将从服务器实现开始。与上一章一样，我们将创建一个单独的客户端和服务器文件夹结构，将 Node.js 代码添加到服务器代码中。

我们将开始创建一个 GraphQL 服务器与数据库代码的旅程。我们客户端的所有数据都将来自数据库，因此我们需要将我们需要的一切放在适当的位置。与上一章一样，我们将安装我们需要与 MongoDB 一起工作的`mongoose`软件包：

```ts
npm install mongoose @types/mongoose --save-dev
```

在选择安装软件包的命令时要记住的一点是与`--save`与`--save-dev`的使用有关。这两者都用于安装软件包，但它们之间有一个实际的区别，以及我们根据它们期望应用程序部署的方式。当我们使用`--save`时，我们声明这个软件包必须下载才能使应用程序运行，即使我们在另一台计算机上安装应用程序。如果我们打算将应用程序部署到已经全局安装了正确版本软件包的计算机上，这可能是浪费的。另一种情况是使用`--save-dev`将软件包下载并安装为所谓的开发依赖。换句话说，该软件包是为开发人员本地安装的。

有了这个基础，我们将开始编写我们在上一章中介绍的`Mongo`类的变体。我们不会重用该实现的原因是因为我们将开始引入特定于 TypeScript 3 的功能，然后再添加通用数据库框架。

我们类的重大变化是，我们将改变`mongoose.connect`方法的签名。其中一个变化告诉 Mongoose 使用新格式的 URL 解析器，但另一个变化与我们用作回调的事件的签名相关联：

```ts
public Connect(): void {
  mongoose.connect(this.url, {useNewUrlParser: true}, (e:unknown) => {
    if (e) {
      console.log(`Unable to connect ` + e);
    } else {
      console.log(`Connected to the database`);
    }
  });
}
```

从上一章，我们应该记住我们的回调的签名是`e:any`。现在，我们将其更改为使用`e:unknown`。这是一种新类型——在 TypeScript 3 中引入的——它允许我们添加额外的类型安全性。在很大程度上，我们可以将`unknown`类型视为类似于`any`，因为我们可以将任何类型分配给它。但是，我们不能在没有类型断言的情况下将其分配给另一种类型。我们将开始在整个代码中将`any`类型移动到`unknown`。

到目前为止，我们一直在使用许多接口来提供类型的形状。我们也可以将相同的技术应用于 Mongo 模式，以便我们可以描述我们的 ToDo 模式的形状作为标准的 TypeScript 接口，然后将其映射到模式。我们的接口将是直接的：

```ts
export interface ITodoSchema extends mongoose.Document {
  Id: string,
  Title: string,
  Description: string,
  DueDate: Date,
  CreationDate: Date,
  Completed: boolean,
}
```

我们将创建一个`mongoose`模式，它将映射到数据库中。一个模式简单地说明了将使用 MongoDB 期望的类型存储的信息。例如，我们的`ITodoSchema`将`Id`公开为`string`，但这不是 MongoDB 期望的类型；相反，它期望看到`String`。知道这一点，从`ITodoSchema`到`TodoSchema`的映射就很简单了，如下所示：

```ts
export const TodoSchema = new Schema({
  Id: String,
  Title: String,
  Description: String,
  DueDate: Date,
  CreationDate: Date,
  Completed: Boolean,
});
```

现在我们有了一个可以用来查询、更新等的模式模型。当然，Mongo 并不限制我们只使用一个模式。如果我们想使用更多，没有什么能阻止我们这样做。

关于我们的模式将包含什么的说明——`Title`和`Description`字段相当直接，它们包含了有关我们待办事项的详细信息。`DueDate`简单地告诉我们项目何时到期，`CreationDate`告诉我们我们创建这条记录的时间。我们有一个`Completed`标志，用户将触发它来表示他们何时完成了任务。

有趣的字段是`Id`字段。这个字段不同于 Mongo 的`Id`字段，后者仍然是内部生成的。模式`Id`字段被分配了一个叫做**全局唯一标识符**（**GUID**）的东西，它是一个唯一的字符串标识符。我们希望 UI 添加这个字段的原因是因为我们将在数据库查询中使用它作为一个已知的字段，并且我们希望客户端在执行任何往返之前知道`Id`的值。当我们涉及 Angular 方面时，我们将看到这个字段是如何被填充的。

我们需要创建一个数据库模型，将我们的`ITodoSchema`的`mongoose.Document`实例映射到我们的`TodoSchema`。当使用`mongoose.model`时，这是一项直接的任务：

```ts
export const TodoModel = mongoose.model<ITodoSchema>('todo', TodoSchema, 'todoitems', false);
```

当我们创建我们的`mongoose.model`时，大小写非常重要。除了`mongoose.model`，我们还有`mongoose.Model`可用，我们需要用`new`语句来实例化。

我们现在有了一个相对通用的数据库类。然而，我们有一个约束——我们期望我们的模式有一个`Id`字段。这个约束纯粹是为了让我们专注于我们演示应用程序的逻辑。

我们要做的第一件事是创建一个接受`mongoose.Document`作为类型的通用基类。毫无疑问，我们最终将针对这个类型使用的是`ITodoSchema`。构造函数将接受一个我们可以用于各种数据库操作的模型。同样，我们已经创建了我们将用作`TodoModel`的模型：

```ts
export abstract class DataAccessBase<T extends mongoose.Document> {
  private model: Model;
  constructor(model: Model) {
    this.model = model;
  }
}
```

我们对这个类的具体实现非常简单：

```ts
export class TodoDataAccess extends DataAccessBase<ITodoSchema> {
  constructor() {
    super(TodoModel);
  }
}
```

我们现在要开始向`DataAccessBase`添加功能。我们将从一个获取与我们的模式匹配的所有记录的方法开始。在这个阶段，我们应该对 promises 感到满意，所以我们应该自然地返回一个`Promise`类型。在这种情况下，`Promise`类型将是一个`T`数组，我们知道它映射到`ITodoSchema`。

在内部，我们调用我们的模型的`find`方法来检索所有记录，一旦查找完成，我们回调结果：

```ts
GetAll(): Promise<T[]> {
  return new Promise<T[]>((callback, error) => {
    this.model.find((err: unknown, result: T[]) => {
      if (err) {
        error(err);
      }
      if (result) {
       callback(result);
      }
    });
 });
}
```

添加记录同样简单。唯一的真正区别是我们调用`model.create`方法并返回一个`boolean`值来指示我们成功了：

```ts
Add(item: T): Promise<boolean> {
  return new Promise<boolean>((callback, error) => {
    this.model.create(item, (err: unknown, result: T) => {
      if (err) {
        error(err);
      }
      callback(!result);
    });
  });
}
```

除了检索所有记录，我们还可以选择检索单个记录。这与`GetAll`方法之间的主要区别在于`find`方法使用了搜索条件：

```ts
Get(id: string): Promise<T> {
  return new Promise<T>((callback, error) =>{
    this.model.find({'Id': id}, (err: unknown, result: T) => {
      if (err) {
        error(err);
      }
      callback(result);
    });
  });
}
```

最后，我们有了删除或更新记录的能力。它们在写法上非常相似：

```ts
Remove(id: string): Promise<void> {
  return new Promise<void>((callback, error) => {
    this.model.deleteOne({'Id': id}, (err: unknown) => {
      if (err) {
        error(err);
      }
      callback();
    });
  });
}
Update(id: string, item: T): Promise<boolean> {
  return new Promise<boolean>((callback, error) => {
    this.model.updateOne({'Id': id}, item, (err: unknown)=>{
      if (err) {
        error(err);
      }
      callback(true);
    });
  })
}
```

有了实际的数据库代码，我们现在可以转向访问数据库。我们要考虑的一件事是，随着时间的推移，我们可能会有大量的待办事项积累起来，如果我们每次需要时都尝试从数据库中读取它们，随着我们添加更多的待办事项，系统会变得越来越慢。为此，我们将创建一个基本的缓存机制，在服务器启动过程中数据库加载完成后立即填充。

由于缓存将被预先填充，我们希望在 GraphQL 和服务器中使用我们类的相同实例，因此我们将创建一个称为**单例**的东西。单例只是另一种说法，即我们在内存中只有一个类的实例，并且每个类将使用相同的实例。为了防止其他类能够创建自己的实例，我们将利用一些技巧。

我们要做的第一件事是创建一个带有私有构造函数的类。私有构造函数意味着我们只能在类内部实例化我们的类：

```ts
export class Prefill {
  private constructor() {}
}
```

这可能看起来有些反直觉，我们只能从类本身创建类。毕竟，如果我们不能实例化类，我们怎么访问任何成员呢？这个技巧是添加一个字段来保存对类实例的引用，然后提供一个公共静态属性来访问该实例。公共属性将负责实例化类，如果它尚不可用，我们将始终能够访问类的实例：

```ts
private static prefill: Prefill;
public static get Instance(): Prefill {
  return this.prefill || (this.prefill = new this());
}
```

现在我们有了访问我们将要编写的方法的方法，让我们开始创建一个方法来填充可用项目的列表。由于这可能是一个长时间运行的操作，我们将使其异步：

```ts
private items: TodoItems[] = new Array<TodoItem>();
public async Populate(): Promise<void> {
  try
  {
    const schema = await this.dataAccess.GetAll();
    this.items = new Array<TodoItem>();
    schema.forEach(item => {
      const todoItem: TodoItem = new TodoItem();
      todoItem.Id = item.Id;
      todoItem.Completed = item.Completed;
      todoItem.CreationDate = item.CreationDate;
      todoItem.DueDate = item.DueDate;
      todoItem.Description = item.Description;
      todoItem.Title = item.Title;
      this.items.push(todoItem);
    });
  } catch(error) {
    console.log(`Unfortunately, we couldn't retrieve all records ${error}`);
  }
}
```

这种方法通过调用`GetAll`来从我们的 MongoDB 数据库中检索所有记录。一旦我们有了记录，我们将遍历它们并创建它们的副本推入我们的数组。

`TodoItem`类是一个特殊的类，我们将使用它来将类型映射到 GraphQL。当我们开始编写我们的 GraphQL 服务器功能时，我们将很快看到这个类。

填充项目数组很好，但如果没有办法在代码的其他地方访问这些项目，这个类将没有太大帮助。幸运的是，访问这些元素就像添加一个`Items`属性一样简单：

```ts
get Items(): TodoItem[] {
  return this.items;
}
```

# 创建我们的 GraphQL 模式

有了我们的数据库代码，我们现在准备转向编写我们的 GraphQL 服务器。在编写本章示例代码时，我最早做出的决定之一是尽可能简化编写代码的过程。如果我们查看 Facebook 发布的参考示例，我们会发现代码可能会非常冗长：

```ts
import {
  graphql,
  GraphQLSchema,
  GraphQLObjectType,
  GraphQLString
} from 'graphql';

var schema = new GraphQLSchema({
  query: new GraphQLObjectType({
    name: 'RootQueryType',
    fields: {
      hello: {
        type: GraphQLString,
        resolve() {
          return 'world';
        }
      }
    }
  })
});
```

这个例子来自[`github.com/graphql/graphql-js`](https://github.com/graphql/graphql-js)。我们可以看到我们对特殊类型有很多依赖，这些类型不能一对一地映射到 TypeScript 类型。

由于我们希望使我们的代码更符合 TypeScript 的要求，我们将使用`type-graphql`。我们将通过`npm`安装它，以及`graphql`类型定义和`reflect-metadata`：

```ts
npm install type-graphql @types/graphql reflect-metadata --save
```

在这个阶段，我们还应该设置我们的`tsconfig`文件如下：

```ts
{
  "compileOnSave": false,
  "compilerOptions": {
    "target": "es2016", 
    "module": "commonjs",
    "lib": ["es2016", "esnext.asynciterable", "dom"],
    "outDir": "./dist",
    "noImplicitAny": true,
    "esModuleInterop": true,
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
  }
}
```

在这个`tsconfig`文件中值得一提的主要是`type-graphql`使用的功能只能在 ES7 中找到，所以我们需要在 lib（ES7 映射到 ES2016）中使用 ES2016。

# 设置我们的 GraphQL 类型

正如我们刚才看到的，设置 GraphQL 类型可能有点复杂。借助`type-graphql`和一些方便的装饰器，我们将创建一个表示单个项目的模式。

我们现在不需要担心创建一个表示多个项目的类型。我们的项目将包括以下字段：

+   `Id`（默认为空字符串）

+   `Title`

+   `Description`（我们暂时将其设置为可空值。当我们创建 UI 时，我们将添加验证，以确保我们始终提供描述。）

+   任务到期的日期（同样，这是可空的）

+   任务创建时

+   任务创建后的天数（当我们查询数据时，这将自动计算）

+   任务是否已完成

如果我们仔细观察，我们会发现这里的字段与我们在 MongoDB 模式中定义的字段非常相似。这是因为我们将从数据库中填充我们的 GraphQL 类型，以及直接从这些类型更新数据库。

和我们现在习惯的一样，我们将从一个简单的类开始：

```ts
export class TodoItem {
}
```

我提到我们将在这个类中使用装饰器。我们将用`@ObjectType`装饰类定义，这使我们能够创建复杂类型。作为优秀的开发者，我们还将提供描述，以便我们类型的使用者了解它代表什么。现在，我们的类定义如下：

```ts
@ObjectType({description: "A single to do"})
export class TodoItem {
}
```

我们将一步一步地向我们的类型添加字段。首先，我们将添加`Id`字段，它与数据库中的`Id`字段匹配：

```ts
@Field(type=>ID)
Id: string="";
```

同样，我们为这个字段提供了一个装饰器，它将告诉`type-graphql`如何将我们的类转换为 GraphQL 类型。通过应用`type=>ID`，我们使用了特殊的 GraphQL `ID`类型。这种类型是一个映射到唯一值的字符串。毕竟，它是一个标识字段，惯例规定标识字段必须是唯一的。

接下来我们将添加三个可空字段——`Description`、`DueDate`和`CreationDate`字段。实际上，我们并不打算允许这些字段的空值，因为当我们在本章后面开始添加 Angular 验证时，我们会看到，但对于我们来说，重要的是要看到我们如何为我们创建的任何未来的 GraphQL 类型添加可空类型：

```ts
@Field({ nullable: true, description: "The description of the item." })
Description?: string;
@Field({ nullable: true, description: "The due date for the item" })
DueDate?: Date;
@Field({ nullable: true, description: "The date the item was created" })
CreationDate: Date;
```

我们还有一些更简单的字段要提供：

```ts
@Field()
Title: string;
@Field(type => Int)
DaysCreated: number;
@Field()
Completed: boolean;
```

我们的`TodoItem`现在看起来像这样，它代表了构成我们查询类型的模式的全部内容：

```ts
@ObjectType({ description: "A single to do" })
export class TodoItem {
  constructor() {
    this.Completed = false;
  }
  @Field(type=>ID)
  Id: string = "";
  @Field()
  Title: string;
  @Field({ nullable: true, description: "The description of the item." })
  Description?: string;
  @Field({ nullable: true, description: "The due date for the item" })
  DueDate?: Date;
  @Field({ nullable: true, description: "The date the item was created" })
  CreationDate: Date;
  @Field(type => Int)
  DaysCreated: number;
  @Field()
  Completed: boolean;
}
```

除了有一个用于查询的类之外，我们还需要一个类来表示我们将用于变异状态的数据，以及用于更新数据库的数据。

当我们改变状态时，我们正在改变它。我们希望这些更改能够在服务器重新启动时持久化，因此它们将更新数据库和我们将在运行时缓存的状态。

我们将用于 mutation 的类看起来与我们的`TodoItem`类非常相似。主要区别在于我们使用`@InputType`代替`@ObjectType`，并且该类实现了`TodoItem`的通用`Partial`类型。另一个区别是这个类没有`DaysCreated`字段，因为这将由我们的查询计算，所以我们不必添加任何值来保存它：

```ts
@InputType()
export class TodoItemInput implements Partial<TodoItem> {
  @Field()
  Id: string;
  @Field({description: "The item title"})
  Title: string = "";
  @Field({ nullable: true, description: "The item description" })
  Description?: string = "";
  @Field({ nullable: true, description: "The item due date" })
  DueDate?: Date;
  @Field()
  CreationDate: Date;
  @Field()
  Completed: boolean = false;
}
```

如果你不知道`Partial`的作用，它只是使`TodoItem`的所有属性变成可选的。这样我们就可以将我们的新 mutation 类与旧类联系起来，而不必提供每个属性。

# 创建我们的 GraphQL 解析器

`TodoItem`和`TodoItemInput`类的目的是为我们提供描述字段、类型和参数的模式。虽然它们是我们 GraphQL 拼图的重要部分，但我们缺少一个部分——执行函数来针对我们的 GraphQL 服务器。

我们需要一种方法来解析我们类型的字段。在 GraphQL 中，解析器代表一个单一字段。它获取我们需要的数据，有效地向 GraphQL 服务器提供了详细的指令，告诉它如何将查询转换为数据项（我们可以将这视为我们为变异数据和查询数据分别使用不同的逻辑的原因之一）。由此，我们可以得出字段和解析器之间存在一对一的映射。

使用`type-graphql`，我们可以轻松创建复杂的解析器关系和操作。我们将从定义我们的类开始。

`@Resolver`装饰器告诉我们，这个类的行为与 REST 类型的控制器类相同：

```ts
@Resolver(()=>TodoItem)
export class TodoItemResolver implements ResolverInterface<TodoItem>{
}
```

严格来说，`ResolverInterface`对于我们的类并不是必需的，但是当我们向`DaysCreated`字段添加字段解析器时，我们将使用它作为一个安全网。这个字段将返回今天的日期和任务创建日期之间的差异。由于我们正在创建一个字段解析器，`ResolverInterface`检查我们的字段是否具有对象类型的`@Root`装饰器作为参数，并且返回类型是正确的类型。

我们的`DaysCreated`字段解析器装饰有`@FieldResolver`，看起来像这样：

```ts
private readonly milliSecondsPerDay = 1000 * 60 * 60 * 24;
@FieldResolver()
DaysCreated(@Root() TodoItem: TodoItem): number {
  const value = this.GetDateDifference(...[new Date(), TodoItem.CreationDate]);
  if (value === 0) {
    return 0;
  }
  return Math.round(value / this.milliSecondsPerDay);
}
private GetDateDifference(...args: [Date, Date]): number {
  return Math.round(args[0].valueOf() - args[1].valueOf());
}
```

虽然这些方法看起来复杂，但实际上它们非常简单。我们的`DaysCreated`方法接收当前的`TodoItem`，并使用`GetDateDifference`计算出今天和`CreationDate`值之间的差异。

我们的`type-graphql`解析器还可以定义我们想要执行的查询和变异。对我们来说，定义一种检索所有待办事项的方法将非常有用。我们将创建一个使用`@Query`装饰的方法，以标识这将是一个查询操作。由于我们的查询有可能返回多个项目，我们告诉解析器返回类型是`TodoItem`类型的数组。就像我们之前创建`Prefill`类的辛苦工作一样，我们的方法就是这么简单：

```ts
@Query(() => [TodoItem], { description: "Get all the TodoItems" })
async TodoItems(): Promise<TodoItem[]> {
  return await Prefill.Instance.Items;
}
```

我们想要允许用户执行的操作之一是只查询逾期的记录。我们可以利用与上次查询类似的逻辑，但我们将筛选那些未完成的记录，这些记录已经超过了他们的截止日期：

```ts
@Query(() => [TodoItem], { description: "Get items past their due date" })
async OverdueTodoItems(): Promise<TodoItem[]> {
  const localCollection = new Array<TodoItem>();
  const testDate = new Date();
  await Prefill.Instance.Items.forEach(x => {
    if (x.DueDate < testDate && !x.Completed) {
      localCollection.push(x);
    }
  });
  return localCollection;
}
```

严格来说，对于像这样塑造数据的操作，我通常会将过滤逻辑委托给数据层，以便它只返回适当的记录。在这种情况下，我决定在解析器中进行过滤，以便我们可以看到相同的数据源可以以我们需要的任何方式进行塑造。毕竟，我们可能已经从一个不允许我们以适当方式塑造它的源中检索到这些数据。

我必须强调的一点是，在尝试执行任何查询或变异之前，我们必须导入 reflect-metadata。这是因为在使用装饰器时依赖反射。没有 reflect-metadata，我们将无法使用装饰器，因为它们在内部使用反射。

拥有查询数据的能力是很好的，但解析器还应该能够对数据执行变异。为此，我们将添加解析器来添加、更新和删除新的待办事项，以及在用户决定任务完成时设置`Completed`标志。我们将从`Add`方法开始。

由于这是一个变异，`type-graphql`提供了`@Mutation`装饰器。我们的方法将接受一个`TodoItemInput`参数。这是通过匹配的`@Arg`装饰器传递的。我们需要提供这个显式的`@Arg`是因为 GraphQL 期望变异有参数作为参数。通过使用`@Arg`，我们为它们提供了所需的上下文。在提供变异的同时，我们期望我们也将提供一个返回类型，因此正确地映射变异和方法的实际返回类型非常重要：

```ts
@Mutation(() => TodoItem)
async Add(@Arg("TodoItem") todoItemInput: TodoItemInput): Promise<TodoItem> {
}
```

我们变异方法的一个特点是，除了更新`Prefill`项目，我们还将更新数据库，这意味着我们必须将我们方法中的输入转换为我们的`ITodoSchema`类型。

为了帮助我们，我们将使用以下简单的方法：

```ts
private CreateTodoSchema<T extends TodoItem | TodoItemInput>(todoItem: T): ITodoSchema {
  return <ITodoSchema>{
    Id: todoItem.Id,
    CreationDate: todoItem.CreationDate,
    DueDate: todoItem.DueDate,
    Description: todoItem.Description,
    Title: todoItem.Title,
    Completed: false
  };
}
```

我们接受`TodoItem`和`TodoItemInput`，因为我们将使用相同的方法来创建一个可接受我们数据库层的记录。由于该记录的来源可以是从`Prefill`项目中找到特定记录，也可以是从我们的 UI 传递过来，我们需要确保我们可以处理这两种情况。

我们的`Add`方法的第一部分涉及创建一个将存储在我们的`Prefill`集合中的`TodoItem`项目。一旦我们将项目添加到集合中，我们将把记录添加到数据库中。我们的完整的`Add`方法看起来像这样：

```ts
@Mutation(() => TodoItem)
async Add(@Arg("TodoItem") todoItemInput: TodoItemInput): Promise<TodoItem> {
  const todoItem = <TodoItem> {
    Id : todoItemInput.Id,
    CreationDate : todoItemInput.CreationDate,
    DueDate : todoItemInput.DueDate,
    Description : todoItemInput.Description,
    Title : todoItemInput.Title,
    Completed : todoItemInput.Completed
  };
  todoItem.Completed = false;
  await Prefill.Instance.Items.push(todoItem);
  await this.dataAccess.Add(this.CreateTodoSchema(todoItem));
  return todoItem;
}
```

现在我们知道如何添加记录，我们可以把注意力转向使用变异来更新记录。我们已经有了大部分的代码基础，所以更新变得更加简单。`Update`方法首先通过检索已经缓存的条目来开始，通过搜索具有匹配的`Id`的项目。如果我们找到这条记录，我们在更新与相关的`Title`、`Description`和`DueDate`之前更新匹配的数据库记录：

```ts
@Mutation(() => Boolean!)
async Update(@Arg("TodoItem") todoItemInput: TodoItemInput): Promise<boolean> {
  const item: TodoItem = await Prefill.Instance.Items.find(x => x.Id === todoItemInput.Id);
  if (!item) return false;
  item.Title = todoItemInput.Title;
  item.Description = todoItemInput.Description;
  item.DueDate = todoItemInput.DueDate;
  this.dataAccess.Update(item.Id, this.CreateTodoSchema(item));
  return true;
}
```

删除记录并不比`Update`方法复杂。为了删除记录，我们只需要提供`Id`值，因此我们的方法签名从输入一个复杂类型变为输入一个简单类型——在这种情况下是一个字符串。我们通过缓存条目来查找与`Id`匹配的记录的索引，找到后，我们使用 splice 方法删除缓存条目。当我们在数组上使用 splice 时，我们实际上是在说删除从相关索引开始的条目，并删除我们选择的条目数。因此，要删除`1`条记录，我们将`1`作为该方法的第二个参数提供。我们需要确保我们的数据库是一致的，所以我们也删除数据库条目：

```ts
@Mutation(() => Boolean!)
async Remove(@Arg("Id") id: string): Promise<boolean> {
  const index = Prefill.Instance.Items.findIndex(x => x.Id === id);
  if (index < 0) {
    return false;
  }
  Prefill.Instance.Items.splice(index, 1);
  await this.dataAccess.Remove(id);
  return true;
}
```

我们感兴趣的最终变异是将`Completed`标志设置为`true`的变异。这个方法在很大程度上是`Remove`和`Update`方法的组合，因为它遵循相同的逻辑来识别记录并更新它。然而，像`Remove`方法一样，它只需要`Id`作为输入参数。由于我们只打算更新`Completed`字段，这是我们在这个方法中要处理的唯一字段：

```ts
@Mutation(() => Boolean!)
async Complete(@Arg("Id") id: string) : Promise<boolean> {
  const item: TodoItem = await Prefill.Instance.Items.find(x => x.Id === id);
  if (!item) return false;
  item.Completed = true;
  await this.dataAccess.Update(item.Id, this.CreateTodoSchema(item));
  return true;
}
```

我们本可以选择重用`Update`方法，并从客户端代码将`Completed`设置为 true，但这将使用更复杂的调用来实现一个更简单的目标。通过使用单独的方法，我们确保我们有一个只做一件事情的代码。这使我们遵循我们感兴趣的单一责任原则。

有了我们的解析器和模式，我们现在可以把注意力转向添加代码来实际提供我们的 GraphQL 服务器。

# 使用 Apollo Server 作为我们的服务器

我们将为这个项目创建一个新的服务器实现，而不是重用上一章的任何服务器基础设施。Apollo 提供了自己的服务器实现（称为 Apollo Server），我们将在这里使用它来代替 Express。和往常一样，我们将首先引入必要的类型，然后创建我们的类定义。在构造函数中，我们将引入对我们的`Mongo`数据库类的引用。

Apollo Server 是 Apollo GraphQL 策略的一部分，用于提供开箱即用的 GraphQL 支持。服务器可以独立运行，也可以与 Express 等服务器框架一起工作，用于提供自描述的 GraphQL 数据。我们要使用 Apollo Server 的原因是因为它内置了与 GraphQL 模式一起工作的支持。如果我们试图自己添加这种支持，我们最终会重做我们从 Apollo Server 中免费获得的内容。

首先，我们要导入我们的类型：

```ts
npm install apollo-server apollo-server-express --save
```

然后，我们将编写我们的`server`类：

```ts
export class MyApp {
  constructor(private mongo: Mongo = new Mongo()) { }
}
```

我们的服务器将公开一个`Start`方法，负责连接到数据库并启动我们的 Apollo 服务器：

```ts
public async Start(): Promise<void> {
  this.mongo.Connect();

  await Prefill.Instance.Populate();

  const server = new ApolloServer({ schema, playground: true });
  await server.listen(3000);
}
```

当我们创建 Apollo Server 实例时，我们指示要使用`GraphQLSchema`，但我们没有定义关于该模式的任何内容。我们使用`buildSchema`函数，它接受一系列选项并使用它们来引入 Apollo Server 将使用的模式。`resolvers`接受一个 GraphQL 解析器数组，因此我们将`TodoItemResolver`作为我们要使用的解析器提供。当然，这里的含义是我们可以使用多个解析器。

`validate`标志表示我们是否要验证传递给解析器参数的对象。由于我们使用简单的对象和类型，我们将其设置为`false`。

我喜欢做的一件事是使用`emitSchemaFile`来验证我创建的 GQL。这使用路径操作来构建一个完全合格的路径名。在这种情况下，我们将解析到`dist`文件夹，我们将在那里输出`apolloschema.gql`文件：

```ts
const schema: GraphQLSchema = await buildSchema({
  resolvers: [TodoItemResolver],
  validate: false,
  emitSchemaFile: path.resolve(__dirname, 'apolloschema.gql')
});
```

现在我们已经完成了服务器端的编码，我们可以添加`new MyApp().Start();`来启动和运行我们的应用程序。当我们构建和运行我们的服务器端时，它将在`http://localhost:3000`上启动一个启用 Apollo 的 GraphQL 服务器的实例。我们还有一个小小的惊喜，与我们提供给 Apollo Server 选项的最后一个参数有关，即`playground: true`。游乐场是一个可视化编辑区域，让我们运行`graphql`查询并查看它们带来的结果。

我建议在生产代码中关闭游乐场。然而，对于测试目的，它是一个无价的辅助工具，可以尝试查询。

为了检查我们是否正确连接了所有内容，请尝试在查询窗口中输入 GraphQL 查询。在输入查询时，请记住，只因为它与 JavaScript 对象有表面上的相似之处，并不意味着需要使用单独的条目。以下是一个开始的示例查询。此查询使用我们在`TodoItemResolver`中创建的`TodoItems`查询：

```ts
query {
  TodoItems {
    Id
    Title
    Description
    Completed
    DaysCreated
  }
}
```

# GraphQL Angular 客户端

就像我们在上一章中所做的那样，我们将创建一个使用 Angular Material 作为其 UI 的 Angular 客户端。同样，我们将使用`ng new`命令创建一个新的应用程序，并将前缀设置为`atp`。由于我们想要为我们的应用程序添加路由支持，我们将在命令行中添加额外的`--routing`参数。我们这样做是因为它会向`app.module.ts`添加必要的`AppRoutingModule`条目，并为我们创建`app-routing.module.ts`路由文件：

```ts
ng new Chapter05 --style scss --prefix atp --routing true
```

在上一章中，即使我们使用了 Material，我们也没有利用它的路由。在我们回到本书的其余部分使用 Bootstrap 之前，我们将再次使用 Material 一次，因此我们需要为我们的应用程序添加 Material 支持（不要忘记在提示时接受添加对浏览器动画的支持）：

```ts
ng add @angular/material @angular/cdk @angular/animation @angular/flex-layout
```

在这个阶段，我们的`app.module.ts`文件应该是这样的：

```ts
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { AppRoutingModule } from './app-routing.module';
import { AppComponent } from './app.component';
import { BrowserAnimationsModule } from '@angular/platform-browser/animations';
@NgModule({
  declarations: [
    AppComponent
  ],
  imports: [
    BrowserModule,
    AppRoutingModule,
    BrowserAnimationsModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

我们需要将 Material 模块导入到我们的`imports`数组中：

```ts
HttpClientModule,
HttpLinkModule,
BrowserAnimationsModule,
MatToolbarModule,
MatButtonModule,
MatSidenavModule,
MatIconModule,
MatListModule,
FlexLayoutModule,
HttpClientModule,
MatInputModule,
MatCardModule,
MatNativeDateModule,
MatDatepickerModule,
```

我们将`MatNativeDateModule`与`MatDatepickerModule`一起添加，因为 Material 日期选择器的构建方式。它不提供关于日期实现方式的任何硬性假设，因此我们需要导入适当的日期表示。虽然我们可以编写自己的日期处理模块实现，但通过引入`MatNativeDateModule`，我们将取得真正的成功。如果我们没有这样做，我们将得到一个运行时错误，告诉我们`未找到 DateAdapter 的提供程序`。

# 添加客户端 Apollo 支持

在我们开始创建用户界面之前，我们将设置 Apollo 集成的客户端部分。虽然我们可以使用`npm`安装 Apollo 的所有单独部分，但我们将再次使用`ng`的强大功能：

```ts
ng add apollo-client
```

回到`AppModule`，我们将设置 Apollo 与服务器进行交互。`AppModule`的构造函数是我们注入 Apollo 创建与服务器连接的完美位置。我们的构造函数一开始看起来像这样：

```ts
constructor(httpLink: HttpLink, apollo: Apollo) {
}
```

我们连接到服务器的方式是通过`apollo.create`命令。这个命令接受许多选项，但我们只关注其中的三个。我们需要一个链接，用于建立与服务器的连接；一个缓存，如果我们想要缓存我们的交互结果；以及一个覆盖默认 Apollo 选项的选项，我们在这里设置了观察查询始终从网络获取。如果我们不从网络获取，就会遇到缓存数据变得陈旧直到刷新的问题：

```ts
apollo.create({
  link: httpLink.create({ uri: 'http://localhost:3000' }),
  cache: new InMemoryCache(),
  defaultOptions: {
    watchQuery: {
      // To get the data on each get, set the fetchPolicy
      fetchPolicy: 'network-only'
    }
  }
});
```

不要忘记，注入组件需要我们将相关模块添加到`@NgModule`模块的`imports`部分。在这种情况下，如果我们想要能够在其他地方自动使用这些，我们需要添加`HttpLinkModule`和`ApolloModule`。

这是我们需要放置的所有代码，以便我们的客户端与工作的 Apollo 服务器进行通信。当然，在生产系统中，我们会从其他地方获取服务器的地址并使用它，而不是硬编码的 localhost。但对于我们的示例，这就是我们需要的。现在我们可以继续添加屏幕和使用路由导航到它们的任务。

# 添加路由支持

我们为应用程序设置的要求是我们将有三个主要屏幕。我们的主屏幕将显示所有待办任务，包括它们是否已完成。第二个将显示过期任务，最后一个将让我们的用户添加新任务。每个都将创建为单独的组件。现在，我们将添加它们的虚拟实现，这将允许我们设置我们的路由：

```ts
ng g c components/AddTask
ng g c components/Alltasks
ng g c components/OverdueTasks
```

我们的路由是从`app-routing.module.ts`文件配置和控制的。在这里，我们将定义一组规则，我们期望 Angular 遵循。

在我们开始添加路由之前，我们实际上应该弄清楚这里的路由术语是什么意思。想到路由的简单方法是想到 URL。路由对应于 URL，或者说，对应于 URL 的基地址之外的部分。由于我们的页面将在`localhost:4000`上运行，我们的完整 URL 是`http://localhost:4000/`。现在，如果我们希望我们的`AllTasks`组件映射到`http://localhost:4000/all`，我们会认为路由是`all`。

现在我们知道了路由是什么，我们需要将这三个组件映射到它们自己的路由。我们首先通过定义一个路由数组开始：

```ts
const routes: Routes = [
];
```

我们通过在模块定义中提供路由与我们的路由模块相关联，如下所示：

```ts
@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule { }
```

我们希望将`AllTasks`组件映射到`all`，因此我们将其添加为路由数组中的一个元素：

```ts
{
  path: 'all',
  component: AlltasksComponent
},
```

此时，当我们启动我们的 Angular 应用程序时，如果我们在`http://localhost:4000/all`中输入，我们可以显示`all`任务页面。虽然这相当令人印象深刻，但如果我们没有默认的站点着陆页的概念，这将会让用户感到恼火。我们的用户通常会期望他们可以进入站点而不必知道我们任何页面名称的细节，并且他们应该能够从那里导航，因为我们将引导他们到适当的页面。幸运的是，我们可以非常容易地实现这一点。我们将添加另一个包含空路径的路由。当我们遇到空路径时，我们将重定向用户到`all`页面：

```ts
{
  path: '',
  redirectTo: 'all',
  pathMatch: 'full'
},
```

现在，当用户导航到`http://localhost:4000/`时，他们将被重定向以查看我们所有未完成的任务。

我们还有两个组件，我们希望用户能够导航到：我们的`AddTask`页面和我们的`OverdueTasks`页面。同样，我们将添加支持通过新路由导航到这些页面。一旦我们添加了这些路由，我们就可以关闭这个文件，因为我们已经添加了所有我们需要的核心路由支持：

```ts
{
  path: 'add',
  component: AddTaskComponent
},
{
  path: 'overdue',
  component: OverduetasksComponent
}
```

# 路由用户界面

将路由支持添加到我们应用程序的最后一部分是设置`app-component.html`的内容。在这里，我们将添加一个工具栏，其中包含指向我们页面的链接以及显示页面组件本身的位置。工具栏只包含三个导航列表项。每个链接的有趣部分是`routerLink`，它将我们的链接与之前添加的地址联系起来。实际上，这部分的作用是告诉代码，当我们链接到该路由时，我们希望内容呈现在特殊的`router-outlet`标记中，这只是实际组件内容的占位符。

```ts
<mat-toolbar color="primary">
  <mat-nav-list><a mat-list-item routerLink="all">All tasks</a></mat-nav-list>
  <mat-nav-list><a mat-list-item routerLink="overdue">Overdue tasks</a></mat-nav-list>
  <mat-nav-list><a mat-list-item routerLink="add">Add task</a></mat-nav-list>
</mat-toolbar> 
<div>
  <router-outlet></router-outlet>
</div>
```

现在，当我们运行我们的应用程序时，单击不同的链接将显示相应的页面，尽管它们实际上几乎没有任何内容。

# 向页面组件添加内容

现在我们已经整理好了我们的路由，我们准备开始为我们的页面添加一些功能。除了添加内容，我们还将通过使用 Angular 验证来为我们的应用程序添加一些修饰，以便为用户提供即时反馈。我们要开始的组件是`AddTask`组件。如果没有添加任务的能力，我们将无法显示任何任务，所以让我们给自己一个机会开始添加一些待办任务。

在我开始添加用户界面元素之前，我喜欢确保我尽可能多地放置组件背后的逻辑。一旦这一点到位，实际添加用户界面就变得简单。在某些情况下，这意味着我甚至在考虑如何显示特定的显示部分或使用什么控件来显示它之前，我已经决定了 UI 约束。考虑到这一点，我们知道我们的待办事项中的一项是`DueDate`。如果我们思考一下，我们意识到创建一个已经过期的截止日期的任务是没有意义的。因此，我们将设置任务的最早截止日期为今天的日期。这将用作我们选择日期的任何控件的约束。

```ts
EarliestDate: Date;
ngOnInit() {
 this.EarliestDate = new Date();
}
```

我们将从用户那里捕获三件事，以便创建我们的待办任务。我们需要捕获标题、描述和任务到期日期。这告诉我们我们需要三个项目作为我们的模型。

```ts
Title: string;
Description?: string;
DueDate: Date;
```

这就是我们在添加任务组件的模型方面所需要的一切，但我们缺少实际将任何内容保存到我们的 GraphQL 服务器的能力。在我们开始与服务器通信之前，我们需要在我们的组件中引入 Apollo 的支持。这只需要在我们的构造函数中添加一个引用即可。

```ts
constructor(private apollo: Apollo) { }
```

我们要执行的操作必须与我们的解析器期望的相匹配。这意味着类型必须完全匹配，我们的 GraphQL 必须格式良好。由于我们要执行的任务是添加操作，我们将调用用于添加数据的方法`Add`。

```ts
Add(): void {
}
```

添加操作将触发服务器上我们创建的解析器上的`Add`变异。我们知道这接受一个`TodoItemInput`实例，因此我们需要将我们的客户端模型转换为`TodoItemInput`实例，如下所示。

```ts
const todo: ITodoItemInput = new TodoItemInput();
todo.Completed = false;
todo.Id = Guid.create.toString();
todo.CreationDate = new Date();
todo.Title = this.Title;
todo.Description = this.Description;
todo.DueDate = this.DueDate;
```

在前面的片段中有一点对我们来说是陌生的，即`Guid.create.toString()`调用。这个命令负责创建一个称为**全局唯一标识符**（**GUID**）的唯一标识符。GUID 是一个 128 位的数字，以字符串和数字格式外部表示，通常看起来像这样——**a14abe8b-3d9b-4b14-9a66-62ad595d4582**。由于 GUID 在数学上保证唯一性，而不是必须调用中央存储库来获取唯一值，它们很快就能生成。通过使用 GUID，我们为我们的待办事项赋予了一个唯一的值。如果需要的话，我们可以在服务器上完成这个操作，但我选择在客户端生成整个消息。

为了使用 GUID，我们将使用`guid-typescript`组件：

```ts
npm install --save guid-typescript
```

现在我们可以将代码放在适当的位置，将数据传输到 GraphQL 服务器。正如我之前提到的，我们将使用`Add`变异，这告诉我们我们将在我们的`apollo`客户端上调用`mutate`：

```ts
this.apollo.mutate({
 ... logic goes here
})
```

变异是一种特殊的字符串，由`gql`覆盖。如果我们能看到这段代码的全部内容，我们将能够立即分解它：

```ts
this.apollo.mutate({
  mutation: gql`
    mutation Add($input: TodoItemInput!) {
      Add(TodoItem: $input) {
        Title
      }
    }
  `, variables: {
    input: todo
  }
}).subscribe();
```

我们已经知道我们将要调用一个变异，所以我们的`mutate`方法接受一个变异作为`MutationOption`。

我们可以向`MutationOption`提供的参数之一是`FetchPolicy`，我们可以用它来覆盖我们在之前创建 Apollo 链接时设置的默认选项。

变异使用`gql`来创建特殊格式的查询。我们的查询分为两部分：告诉我们查询是什么的字符串文本和我们需要应用的任何变量。变量部分创建一个映射到我们之前创建的`TodoItemInput`的输入变量。这在我们的`gql`字符串内表示为`$`，因此任何变量名必须在查询中有一个匹配的`$variable`。当变异完成时，我们告诉它我们想要标题。实际上我们不必带回任何值，但是在我之前调试时，我发现使用标题来检查我们是否从服务器得到了响应是有用的。

我们正在使用 ```ts backtick because this lets us spread our input over multiple lines.

The `mutate` method is triggered from the call to `subscribe`. If we fail to supply this, our mutation will not run. As a convenience, I also added a `Reset` method so that we can clear values away from the UI when the user finishes. I did this so that the user would be able to immediately enter new values:

```

private Reset(): void {

this.Title = ``;

this.Description = ``;

this.DueDate = null;

}

```ts

That is the logic inside our component taken care of. What we need to do now is add the HTML that will be displayed in the component. Before we add any elements to our component, we want to display the card that will contain our display. This will be centered vertically and horizontally in the display. This is not something that comes naturally to Material, so we have to supply our own local styling. We have a couple of other styles that we are going to set as well, to fix the size of the text area and the width of the card, and to set how we display form fields to make sure each one appears on its own line.

Initially, we will set up a style to center the card. The card will be displayed inside a `div` tag, so we will apply the styling to the `div` tag, which will center the card inside it:

```

.centerDiv{

height: 100vh;

display: flex;

justify-content: center;

align-items: center;

}

```ts

Now, we can style the Material card and form fields:

```

.mat-card {

width: 400px;

}

.mat-form-field {

display: block;

}

```ts

Finally, we are going to set the height of the `textarea` tag that the user will use to enter their description to 100 pixels:

```

textarea {

height: 100px;

resize: vertical;

}

```ts

Getting back to our display, we are going to set up the container for our card so that it is centered:

```

<div class="centerDiv" layout-fill layout="column" layout-align="center none">

.... 内容在这里

</div>

```ts

We have reached a point where we want to start leveraging the power of Angular to control the validation of the user input. In order to start treating user input as though it's all related, we are going to put the input parts of our display inside an HTML form:

```

<form name="form" (ngSubmit)="f.form.valid && Add()" #f="ngForm">

.... 表单内容在这里。

</form>

```ts

We need to break this form statement down a bit. We will start by working out what `#f="ngForm"` actually does. This statement assigns the `ngForm` component to a variable called `f`. When we use `ngForm`, we are referring to the component inside `FormsModule` (make sure that it's registered inside the `app.module` imports section). The reason that we do this is because this assignment means that we have access to properties of the component itself. The use of `ngForm` means that we are working with the top-level form group so that we can do things such as track whether or not the form is valid.

We can see this inside `ngSubmit`, where we are subscribing to the event that tells us that the user has triggered the form submission, which results in the validation being checked; when the data is valid, this results in triggering the `Add` method. With this in place, we don't have to directly call `Add` when the Save button is clicked because the submit event will take care of this for us.

There is a short-circuit logic in play with `ngSubmit`. In other words, if the form is not valid, then we won't call the `Add` method.

We are now ready to add the card itself. This lives entirely inside our form. The title section is placed inside a `mat-card-title` section and our buttons are situated inside the `mat-card-actions` section, which aligns the buttons at the bottom of the card. As we just covered, we aren't supplying a click event handler to our Save button because the form submission will take care of this:

```

<div layout="row" layout-align="center none">

<mat-card>

<mat-card-title>

<span class="mat-headline">Add ToDo</span>

</mat-card-title>

<mat-card-content>

.... 内容在这里。

<mat-card-content>

<mat-card-actions>

<button mat-button class="btn btn-primary">Save</button>

</mat-card-actions>

</mat-card>

</div>

```ts

We are ready to start adding the fields so that we can tie them back to the fields in our underlying model. We will start with the title as the description field largely follows this format as well. We will add the field and its related validation display in first, and then we will break down what is happening:

```

<mat-form-field>

<input type="text" matInput placeholder="Title" [(ngModel)]="Title" name="title" #title="ngModel" required />

</mat-form-field>

<div *ngIf="title.invalid && (title.dirty || title.touched)" class="alert alert-danger">

<div *ngIf="title.errors.required">

您必须添加一个标题。

</div>

</div>

```ts

The first part of our input element is largely self-explanatory. We created it as a text field and used `matInput` to hook the standard input so that it can be used inside `mat-form-field`. With this, we can set the placeholder text to something appropriate.

I opted to use `[(ngModel)]` instead of `[ngModel]` because of the way binding works. With `[ngModel]`, we get one-way binding so that it changes flow from the underlying property through to the UI element that displays it. Since we are going to be allowing the input to change the values, we need a form of binding that allows us to send information back from the template to the component. In this case, we are sending the value back to the `Title` property in the element.

The `name` property must be set. If it is not set, Angular throws internal warnings and our binding will not work properly. What we do here is set the name and then use `#` with the value set in the name to tie it to `ngModel`. So, if we had `name="wibbly"`, we would have `#wibbly="ngModel"` as well.

Since this field is required, we simply need to supply the `required` attribute, and our form validation will start working here.

Now that we have the input element hooked up to validation, we need some way of displaying any errors. This is where the next `div` statement comes in. The opening `div` statement basically reads as *if the title is invalid (because it is required and has not been set, for instance), and it has either had a value changed in it or we have touched the field by setting focus to it at some point, then we need to display internal content using the alert and alert-danger attributes*.

As our validation failure might just be one of several different failures, we need to tell the user what the problem actually was. The inner `div` statement displays the appropriate text because it is scoped to a particular error. So, when we see `title.errors.required`, our template will display the You must add a title*.* text when no value has been entered.

We aren't going to look at the description field because it largely follows the same format. I would recommend looking at the Git code to see how that is formatted.

We still have to add the `DueDate` field to our component. We are going to use the Angular date picker module to add this. Effectively, the date picker is made up of three parts.

We have an input field that the user can type directly into. This input field is going to have a `min` property set on it that binds the earliest date the user can select to the `EarliestDate` field we created in the code behind the component. Just like we did in the title field, we will set this field to required so that it will be validated by Angular, and we will apply `#datepicker="ngModel"` so that we can associate the `ngModel` component with this input field by setting the name with it:

```

<input matInput [min]="EarliestDate" [matDatepicker]="picker" name="datepicker" placeholder="Due date"

#datepicker="ngModel" required [(ngModel)]="DueDate">

```ts

The way that we associate the input field is by using `[matDatepicker]="picker"`. As part of our form field, we have added a `mat-datepicker` component. We use `#picker` to name this component `picker`, which ties back to the `matDatepicker` binding in our input field:

```

<mat-datepicker #picker></mat-datepicker>

```ts

The final part that we need to add is the toggle that the user can press to show the calendar part on the page. This is added using `mat-datepicker-toggle`. We tell it what date picker we are applying the calendar to by using `[for]="picker"`:

```

<mat-datepicker-toggle matSuffix [for]="picker"></mat-datepicker-toggle>

```ts

Right now, our form field looks like this:

```

<mat-form-field>

<input matInput [min]="EarliestDate" [matDatepicker]="picker" name="datepicker" placeholder="Due date"

#datepicker="ngModel" required [(ngModel)]="DueDate">

<mat-datepicker-toggle matSuffix [for]="picker"></mat-datepicker-toggle>

<mat-datepicker #picker></mat-datepicker>

</mat-form-field>

```ts

All that we are missing now is the validation. Since we have already defined that the earliest date we can choose is today, we don't need to add any validation to that. We have no maximum date to worry about, so all we need to do is check that the user has chosen a date:

```

<div *ngIf="datepicker.invalid && (datepicker.dirty || datepicker.touched)" class="alert alert-danger">

<div *ngIf="datepicker.errors.required">

您必须选择一个截止日期。

</div>

</div>

```ts

So, we have reached the point where we can add tasks to our todo list and they will be saved to the database, but that isn't much use to us if we can't actually view them. We are now going to turn our attention to the `AllTasksComponent` and `OverdueTasksComponent` components.

Our `AllTasksComponent` and `OverdueTasksComponent` components are going to display the same information. All that differs between the two is the GQL call that is made. Because they have the same display, we are going to add a new component that will display the todo information. `AllTasksComponent` and `OverdueTasksComponent` will both use this component:

```

ng g c components/Todo-Card

```ts

Just like in our add task component, `TodoCardComponent` is going to start off with an `EarliestDate` field and the Apollo client being imported:

```

EarliestDate: Date;

constructor(private apollo: Apollo) {

this.EarliestDate = new Date();

}

```ts

We have reached the point where we need to consider what this component is actually going to be doing. It will receive a single `ITodoItem` as input from either `AllTasksComponent` or `OverdueTasksComponent`, so we will need a means for the containing component to be able to pass this information in. We will also need a means to notify the containing component of when the todo item has been deleted so that it can be removed from the tasks being displayed (we will just do this on the client side rather than triggering a requery via GraphQL). Our UI will add a Save button when the user is editing the record, so we are going to need some way to track that the user is in the edit section.

With those requirements for the component, we can add in the necessary code to support this. First, we are going to address the ability to pass in a value to our component as an input parameter. In other words, we are going to add a field that can be seen and has values set on it by using data binding by the containers. Fortunately, Angular makes this a very simple task. By marking a field with `@Input`, we expose it for data binding:

```

@Input() Todo: ITodoItem;

```ts

That takes care of the input, but how do we let the container know when something has happened? When we delete a task, we want to raise an event as output from our component. Again, Angular makes this simple by using `@Output` to expose something; in this case, we are going to expose `EventEmitter`. When we expose this to our containers, they can subscribe to the event and react when we emit the event. When we create `EventEmitter`, we are going to create it to pass the `Id` of our task back, so we need `EventEmitter` to be a string event:

```

@Output() deleted: EventEmitter<string> = new EventEmitter<string>();

```ts

With this code in place, we can update our `AllTasksComponent` and `OverdueTasksComponent` templates that will hook up to our component:

```

<div fxLayout="row wrap" fxLayout.xs="column" fxLayoutWrap fxLayoutGap="20px grid" fxLayoutAlign="left">

<atp-todo-card

*ngFor="let todo of todos"

[Todo]="todo"

(deleted)="resubscribe($event)"></atp-todo-card>

</div>

```ts

Before we finish adding the logic to `TodoCardComponent`, let's get back to `AllTasksComponent` and `OverdueTasksComponent`. Internally, these are both very similar, so we will concentrate on the logic in `OverdueTasksComponent`.

It shouldn't come as a shock now that these components will accept an Apollo client in the constructor. As we saw from `ngFor` previously, our component will also expose an array of `ITodoItem` called `todos`, which will be populated by our query:

```

todos: ITodoItem[] = new Array<ITodoItem>();

constructor(private apollo: Apollo) { }

```ts

You may notice, from looking at the code in the repository, that we have not added this code into our component. Instead, we are using a base class called `SubscriptionBase` that provides us with a `Subscribe` method and a resubscribe event.

Our `Subscribe` method is generic accepts either `OverdueTodoItemQuery` or `TodoItemQuery` as the type, along with a `gql` query, and returns an observable that we can subscribe to in order to pull out the underlying data. The reason we have added the base class goes back to the fact that `AllTasksComponent` and `OverdueTasksComponent` are just about identical, so it makes sense to reuse as much code as possible. The name that is sometimes given to this philosophy is **Don't Repeat Yourself** (**DRY**):

```

protected Subscribe<T extends OverdueTodoItemQuery | TodoItemQuery>(gqlQuery: unknown): Observable<ApolloQueryResult<T>> {

}

```ts

All this method does is create a query using `gql` and set `fetch-policy` to `no-cache` to force the query to read from the network rather than relying on the cache set in `app-module`. This is just another way of controlling whether or not we read from the in-memory cache:

```

return this.apollo.query<T>({

query: gqlQuery,

fetch-policy: 'no-cache'

});

```ts

We extend from a choice of two interfaces because they both expose the same items but with different names. So, `OverdueTodoItemQuery` exposes `OverdueTodoItems` and `TodoItemQuery` exposes `TodoItems`. The reason that we have to do this, rather than using just one interface, is because the field must match the name of the query. This is because Apollo client uses this to automatically map results back.

The `resubscribe` method is called after the user clicks the delete button in the interface (we will get to building up the UI template shortly). We saw that our `resubscribe` method was wired up to the event and that it would receive the event as a string, which would contain the `Id` of the task we want to delete. Again, all we are going to do to delete the record is find the one with the matching `Id`, and then splice the todos list to remove it:

```

resubscribe = (event: string) => {

const index = this.todos.findIndex(x => x.Id === event);

this.todos.splice(index, 1);

}

```ts

Going back to `OverdueTasksComponent`, all we need to do is call `subscribe`, passing in our `gql` query and subscribing to the return data. When the data comes back, we are going to populate our todos array, which will be displayed in the UI:

```

ngOnInit() {

this.Subscribe<OverdueTodoItemQuery>(gql`query ItemsQuery {

OverdueTodoItems {

Id,

标题，

Description,

DaysCreated,

DueDate,

Completed

}

}).subscribe(todo => {

this.todos = new Array<ITodoItem>();

todo.data.OverdueTodoItems.forEach(x => {

this.todos.push(x);

});

});

}

```ts

A note on our subscription—as we are creating a new list of items to display, we need to clear `this.todos` before we start pushing the whole list back into it.

With `AllTasksComponent` and `OverdueTasksComponent` complete, we can turn our attention back to `TodoCardComponent`. Before we finish off adding the component logic, we really need to take a look at the way the template is created. A large part of the logic is similar to the add task UI logic, so we aren't going to worry about how to hook up to a form or add a validation. The things I want to concentrate on here relate to the fact that the task component will display differently when the user is in edit mode, as opposed to a read-only or label-based version. Let's start by looking at the title. When the task is in read-only mode, we are just going to display the title in `span`, like this:

```

<span>{{Todo.Title}}</span>

```ts

When we are editing the task, we want to show input elements and validation, as follows:

```

<mat-form-field>

<input type="text" name="Title" matInput placeholder="Title" [(ngModel)]="Todo.Title" #title="ngModel"

required />

</mat-form-field>

<div *ngIf="title.invalid && (title.dirty || title.touched)" class="alert alert-danger">

<div *ngIf="title.errors.required">

You must add a title.

</div>

</div>

```ts

We do this by using a neat trick of Angular. Behind the scenes, we are maintaining an `InEdit` flag. When that is false, we want to display the span. If it is true, we want to display a template in its place that contains our input logic. To do this, we start off by wrapping our span inside a `div` tag. This has an `ngIf` statement that is bound to `InEdit`. The `ngIf` statement contains an `else` clause that picks up the template with the matching name and displays this in its place:

```

<div *ngIf="!InEdit;else editTitle">

<span>{{Todo.Title}}</span>

</div>

<ng-template #editTitle>

<mat-form-field>

<input type="text" name="Title" matInput placeholder="Title" [(ngModel)]="Todo.Title" #title="ngModel"

required />

</mat-form-field>

<div *ngIf="title.invalid && (title.dirty || title.touched)" class="alert alert-danger">

<div *ngIf="title.errors.required">

You must add a title.

</div>

</div>

</ng-template>

```ts

Other fields are displayed in a similar way. There is one more point of interest in the way we display the read-only fields. `DueDate` needs to be formatted in order to be displayed as a meaningful date rather than as the raw date/time that is saved in the database. We use `|` to pipe `DueDate` into a special date formatter that controls how the date is displayed. For instance, March 21, 2018 would be displayed as `Due: Mar 21st, 2019` using the following date pipe:

```

<p>Due: {{Todo.DueDate | date}}</p>

```ts

Please take the time to review the rest of `todo-card.component.html`. Swapping templates is heavily done, so it is a good way to review how to make the same UI serve two purposes.

In the component itself, we have three operations left to look at. The first one that we will cover is the `Delete` method, which is triggered when the user presses the delete button on the component. This is a simple method that calls the `Remove` mutation, passing the `Id` across to be removed. When the item has been removed from the server, we call `emit` on our `deleted` event. This event passes the `Id` back to the containing component, which results in this item being removed from the UI:

```

Delete() {

this.apollo.mutate({

mutation: gql`

mutation Remove($Id: String!) {

Remove(Id: $Id)

}

`, variables: {

Id: this.Todo.Id

}

}).subscribe();

this.deleted.emit(this.Todo.Id);

}

```ts

The `Complete` method is just as simple. When the user clicks the `Complete` link, we call the `Complete` query, which passes across the current `Id` as the matching variable. As we could be in edit mode at this point, we call `this.Edit(false)` to switch back to read-only mode:

```

Complete() {

this.apollo.mutate({

mutation: gql`

mutation Complete($input: String!) {

Complete(Id: $input)

}

`, variables: {

input: this.Todo.Id

}

}).subscribe();

this.Edit(false);

this.Todo.Completed = true;

}

```ts

The `Save` method is very similar to the `Add` method in the add task component. Again, we need to switch back from edit mode when this mutation finishes:

```

Save() {

const todo: ITodoItemInput = new TodoItemInput();

todo.Completed = false;

todo.CreationDate = new Date();

todo.Title = this.Todo.Title;

todo.Description = this.Todo.Description;

todo.DueDate = this.Todo.DueDate;

todo.Id = this.Todo.Id;

this.apollo.mutate({

mutation: gql`

mutation Update($input: TodoItemInput!) {

Update(TodoItem: $input)

}

`, variables: {

input: todo

}

}).subscribe();

this.Edit(false);

}

```

到目前为止，我们拥有一个完全功能的基于客户端和服务器的 GraphQL 系统。

# Summary

在本章中，我们通过将其视为检索和更新数据的 REST 服务的替代品，研究了 GraphQL 带来的好处。我们调查了将 Apollo 设置为服务器端 GraphQL 引擎，并将 Apollo 添加到 Angular 客户端以与服务器交互，以及查看专业的 GQL 查询语言。为了充分利用 TypeScript 的强大功能，我们引入了`type-graphql`包，以简化 GraphQL 模式和解析器的创建。

从上一章的经验中，我们看到了如何开始构建可重用的 MongoDB 数据访问层；虽然还有一些工作要做，但我们已经很好地开始了，留下了一些空间来消除应用程序约束，比如需要使用`Id`来查找记录。

本章还向我们介绍了 Angular 路由，以便根据用户选择的路由提供不同的视图。我们继续使用 Material，以便了解如何将此逻辑应用于我们在第四章中介绍的导航内容，*MEAN Stack - 构建照片库*。我们还看了如何通过查看 Angular 提供的验证内容以及如何将其与内联模板一起使用，以便向用户提供有关任何问题的一致反馈，以防止用户输入错误。

在下一章中，我们将通过使用 Socket.IO 来保持客户端和服务器之间的开放连接，来看另一种与服务器通信的方式。我们将构建一个 Angular 聊天应用程序，该应用程序将自动将对话转发到应用程序的所有开放连接。作为额外的奖励，我们将看到如何在 Angular 中集成 Bootstrap 以替代 Material，并仍然使用诸如路由之类的功能。我们还将介绍大多数专业应用程序依赖的功能：用户认证。

# 问题

1.  GraphQL 是否打算完全取代 REST 客户端？

1.  在 GraphQL 中，突变有什么作用？我们期望看到什么类型的操作？

1.  我们如何将参数传递给 Angular 中的子组件？

1.  模式和解析器之间有什么区别？

1.  我们如何创建一个单例？

完整的函数不会从过期项目页面中删除已完成的任务。增强代码以在用户点击完成后从页面中删除该项目。

# 进一步阅读

+   为了更深入地探讨 GraphQL 的奥秘，我推荐 Brian Kimokoti 的优秀著作《入门 GraphQL》（[`www.packtpub.com/in/application-development/beginning-graphql`](https://www.packtpub.com/in/application-development/beginning-graphql)）。

+   要在 React 中查看 GraphQL 的使用，Sebastian Grebe 写了《使用 GraphQL 和 React 进行全栈 Web 开发实践》（[`www.packtpub.com/in/web-development/hands-full-stack-web-development-graphql-and-react`](https://www.packtpub.com/in/web-development/hands-full-stack-web-development-graphql-and-react)）。
