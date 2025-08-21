# 第四章：构建 Web 应用程序

到这里我们来了！我们走过了漫长的一段路才到达这里。这里才是所有乐趣的开始。我们已经经历了三个阶段：了解 Deno 是什么，探索它提供的工具链，以及通过其运行时了解其细节和功能。

前几章的大部分内容将在这一章中证明是有用的。希望，入门章节让您有信心开始应用我们一起学到的内容。我们将使用这些章节以及您现有的 TypeScript 和 JavaScript 知识，来构建一个完整的 Web 应用程序。

我们将编写一个包含业务逻辑、处理身份验证、授权和日志记录等内容的 API。我们将涵盖足够的基础知识，以便您最终可以自信地选择 Deno 来构建您下一个伟大的应用程序。

在本章中，我们不仅要谈论 Deno，还要回顾一些关于软件工程和应用程序架构的基本思想。我们认为，在从头开始构建应用程序时，牢记一些事情至关重要。我们将查看一些基本原理，这些原理将被证明是有用的，并帮助我们构建代码，使其易于在未来变化中进化。

稍后，我们将开始引用一些第三方模块，查看它们的方法，并决定从这里开始我们将使用什么来帮助我们处理路由和与 HTTP 相关的挑战。我们还将确保我们以一种使第三方代码隔离并作为我们想要构建的功能的使能者而不是功能本身来工作的方式来构建我们的应用程序。

本章我们将涵盖以下主题：

+   构建 Web 应用程序的结构

+   探索 Deno HTTP 框架

+   让我们开始吧！

## 技术要求

本章使用的代码文件可在以下链接找到：[`github.com/PacktPublishing/Deno-Web-Development/tree/master/Chapter04/museums-api`](https://github.com/PacktPublishing/Deno-Web-Development/tree/master/Chapter04/museums-api)。

# 构建 Web 应用程序的结构

当开始一个应用程序时，花时间思考其结构和架构是很重要的。这一节将从这个角度开始：通过查看应用程序架构的骨架。我们将看看它带来了什么优势，并使自己与一套原则保持一致，这些原则将帮助我们在应用程序增长时扩展它。

然后，我们将开发出将成为应用程序第一个端点的部分。然而，首先，我们将从业务逻辑开始。持久层将紧随其后，最后我们将查看一个 HTTP 端点，它将作为应用程序的入口点。

## Deno 作为一个无偏见的工具

当我们使用低级别的工具，并将许多决策权委托给开发者时，如 Node.js 和 Deno，构建应用程序是随之而来的一个重大挑战。

这与具有明确观点的 Web 框架，如 PHP Symfony、Java SpringBoot 或 Ruby on Rails，有很大的不同，在这些框架中，许多这些决策已经为我们做出。

这些决策大多数与结构有关；也就是说，代码和文件夹结构。这些框架通常为我们提供处理依赖项、导入的方法，甚至在不同的应用程序层次结构上提供一些指导。由于我们使用的是*原始*语言和几个包，因此在这本书中我们将自己负责这些结构。

前述框架不能直接与 Deno 相比较，因为它们是构建在诸如 PHP、Java 和 Ruby 等语言之上的框架。但是当我们查看 JS 世界，尤其是 Node.js 时，我们可以观察到最常用来创建 HTTP 服务器的最受欢迎工具是 Express.js 和 Kao.js。这些通常比前述框架轻量级得多，尽管还有一些坚固完整的替代方案，如 Nest.js 或 hapi.js，但 Node.js 社区更倾向于采用*库*方法，而不是*框架*方法。

尽管这些非常流行的库提供了大量功能，但许多决策仍然委托给开发者。这不是库的错，更多的是一个社区偏好。

一方面，直接访问这些原语让我们能够构建非常适合我们用例的应用程序。另一方面，灵活性是一个权衡。拥有大量的灵活性随之而来的是做出无数决策的责任。而当需要做出许多决策时，就有很多机会做出糟糕的决策。难点在于，这些通常是对代码库扩展方式产生巨大影响的决策，这也是它们如此重要的原因。

在当前状态下，Deno 及其社区在框架与库这一问题上遵循与 Node.js 非常相似的方法。社区主要押注于由开发者创建的轻量级且小巧的软件，以适应他们的特定需求。我们将在本章后面评估其中的一些。

从现在开始，在这本书的其余部分，我们将使用一种我们相信对当前用例有很大好处的应用程序结构。然而，不要期望这种结构和架构是灵丹妙药，因为我们深信软件世界中不存在这样的东西；每种架构都将随着成长而不断进化。

我们想要的不仅仅是扔进一个食谱并遵循它，而是要熟悉一种思维方式——一种推理。这应该能让我们在将来做出正确的决策，目标只有一个：*编写易于更改的代码*。

通过编写易于更改的代码，我们总是准备好在不需要太多努力的情况下改进我们的应用程序。

## 应用程序最重要的部分

应用程序是为了适应一个目的而被创建的。无论这个目的是支持一个企业还是一个简单的宠物项目都不重要。归根结底，我们希望它能做些什么。那*些什么*就是使应用程序变得有用的原因。

这可能听起来很显然，但有时对于我们这些开发者来说，我们很容易因为对一种技术的热情而忘记，它只是达到目的的一种手段。

正如 Uncle Bob 在他的*Architecture – the lost years*演讲中所说([`www.youtube.com/watch?v=hALFGQNeEnU`](https://www.youtube.com/watch?v=hALFGQNeEnU)),人们很容易忘记应用程序的目的，而更多地关注技术本身。在我们开发应用程序的所有阶段，记住这一点非常重要，尤其是在建立其初始结构时更是如此。接下来，我们将探讨本书剩余部分我们将要构建的应用程序的需求。

## 我们的应用程序是关于什么的？

虽然我们确实相信在任何应用程序中业务逻辑是最重要的事情，但在这本书中，情况有点不同。我们将创建一个示例应用程序，但它只是一个达到主要目标：学习 Deno 的手段。然而，为了使过程尽可能真实，我们希望在心中有一个清晰的目标。

我们将构建一个允许人们创建和与博物馆列表互动的应用程序。我们可以通过将其功能作为用户故事列出使其更清晰，如下所示：

+   用户能够注册和登录。

+   用户能够创建一个带有标题、描述和位置的博物馆。

+   用户可以查看博物馆列表。

在这个旅程中，我们将开发 API 和支持这些功能的逻辑。

既然我们已经熟悉了最终目标，我们可以开始思考如何组织应用程序。

## 理解文件结构和应用程序架构

关于文件结构，我们首先需要意识到的一点，特别是当我们从零开始一个没有框架的项目时，它将随着项目的发展而不断演变。对于只有几个端点的项目来说好的文件结构，对于有数百个端点的项目来说可能不那么好。这取决于许多事情，从团队规模，到定义的标准，最终到偏好。

在定义文件结构时，重要的是我们要达到一个地步，使我们能够促进关于代码放置位置的未来决策。文件结构应该为如何做出良好的架构决策提供清晰的提示。

同时，我们当然不希望创建一个过度工程化的应用程序。我们将创建足够的抽象，使模块非常独立，并且没有超出它们领域的知识，但不会超过这个程度。牢记这一点也迫使我们构建灵活的代码和清晰的接口。

最终，最重要的是架构能够使代码库具备以下特点：

+   可测试。

+   易于扩展。

+   与特定技术或库解耦。

+   易于导航和理解。

在创建文件夹、文件和模块时，我们必须要记住，绝不能有任何妥协前面提到的话题。

这些原则与软件设计中的 SOLID 原则非常一致，由“Uncle Bob”Robert C. Martin 在一次演讲中提出（[`en.wikipedia.org/wiki/SOLID`](https://en.wikipedia.org/wiki/SOLID)），该演讲值得一看（[`youtu.be/zHiWqnTWsn4`](https://youtu.be/zHiWqnTWsn4)）。

本书我们将要使用的文件夹结构，如果你有 Node.js 背景，可能会觉得熟悉。

正如发生在 Node.js 一样，我们完全可以在一个文件中创建一个完整的 API。然而，我们不会这样做，因为我们认为在初始阶段对关注点进行一些分离将大大提高我们的灵活性，而不会牺牲开发者的生产力。

在下一节中，我们将探讨不同层次的责任以及它们在我们开发应用程序功能时的相互配合。

遵循这种思路，我们努力确保模块之间的解耦程度。例如，我们希望通过确保在 web 框架中的更改不会影响到业务逻辑对象。

所有这些建议，以及我们在这本书中将会提出的建议，将有助于确保我们应用程序的核心部分是业务逻辑，其他所有内容只是插件。JSON API 只是一种将我们的数据发送给用户的方式，而数据库只是一种持久化数据的方式；这些都不应该是应用程序的核心部分。

确保我们这样做的一种方法是在编写代码时进行以下心理练习：

当你在编写业务逻辑时，想象这些对象将在不同的上下文中使用。例如，使用不同的交付机制（例如 CLI）或不同的持久化引擎（内存数据库而非 NoSQL 数据库）。

在接下来的几页中，我们将引导您如何创建不同的层次，并解释所有设计决策以及它们所启用的功能。

让我们来实际操作，开始构建我们项目的基础框架。

### 定义文件夹结构。

在我们项目的文件夹中，我们首先要创建一个 `src` 文件夹。

这里， predictably，是我们的代码将要存放的地方。我们不希望项目的根目录有任何代码，因为可能会在这里添加配置文件、READMEs、文档文件夹等。这会使代码难以区分。

在接下来的章节中，我们将在`src`文件夹内度过大部分时间。由于我们的应用程序是关于博物馆的，我们将在`src`文件夹内创建一个名为`museums`的文件夹。这个文件夹将存放本章将编写的 most of the logic。稍后，我们将创建类型、控制器和方法文件。然后，我们将创建`src/web`文件夹。

控制器的文件是我们的业务逻辑将存放的地方。仓库将处理与数据访问相关的逻辑，而网络层将处理所有与*网络相关*的事情。

您可以通过查看本书的 GitHub 仓库来查看最终结构：[`github.com/PacktPublishing/Deno-Web-Development/tree/master/Chapter04/museums-api`](https://github.com/PacktPublishing/Deno-Web-Development/tree/master/Chapter04/museums-api)。

本章的初始要求是有一个路由，我们可以在此路由上执行 GET 请求，并接收以 JSON 格式表示的博物馆列表。

我们将在控制器文件（`src/museums/controller.ts`）中开始编写所需的业务逻辑。

文件夹结构应该如下所示：

```js
└── src
    ├── museums
    │   ├── controller.ts
    │   ├── repository.ts
    │   └── types.ts
    └── web
```

这是我们开始的地方。与博物馆有关的所有内容都将在`museums`文件夹内，我们将称之为一个模块。`controller`文件将托管业务逻辑，`repository`文件将托管数据获取功能，而`types`文件将位于我们的类型。

现在，让我们开始编写代码！

## 编写业务逻辑

我们之前说过，我们的业务逻辑是应用程序最重要的部分。尽管我们的业务逻辑现在非常简单，但这是我们首先开发的。

由于我们将使用 TypeScript 来编写我们的应用程序，让我们创建一个定义`Museum`对象的接口。按照以下步骤操作：

1.  进入`src/museums/types.ts`，并创建一个定义`Museum`的类型：

    ```js
    export type Museum = {
      id: string,
      name: string,
      description: string,
      location: {
        lat: string,
        lng: string
      }
    }
    ```

    确保它被导出，因为我们将跨其他文件使用此文件。

    现在我们已经知道了类型，我们必须创建一些业务逻辑来获取博物馆列表。

1.  在`src/museums/types.ts`中，创建一个接口，定义`MuseumController`。它应该包含一个列出所有博物馆的方法：

    ```js
    export interface MuseumController {
      getAll: () => Promise<Museum[]>;
    }
    ```

1.  在`src/museums/controller.ts`中，创建一个类，作为控制器。它应该包含一个名为`getAll`的函数。将来，这里将存放业务逻辑，但现在，我们只需返回一个空数组：

    ```js
    import type { MuseumController } from "./types.ts";
    export class Controller implements MuseumController {
      async getAll() {
        return [];
      }
    } 
    ```

    我们可以用这个直接访问数据库并获取某些记录。然而，由于我们希望能够使我们的业务逻辑孤立，并且不与应用程序的其他部分耦合，所以我们不会这样做。

    此外，我们还希望我们的业务逻辑能够在没有数据库或服务器连接的情况下独立测试。为了实现这一点，我们不能直接从我们的控制器访问数据源。稍后，我们将创建一个抽象，它将负责从数据库获取这些记录。

    目前，我们知道我们需要调用一个外部模块，它将为我们获取所有的博物馆，并将它们交给我们的控制器——它来自哪里无关。

    请记住以下软件设计最佳实践：*"面向接口编程，而不是面向实现。"*

    简单地说，这句话的意思是我们应该定义模块的签名，然后才开始考虑它的实现。这大大有助于设计清晰的接口。

    回到我们的控制器，我们知道控制器的`getAll`方法最终必须调用一个模块来从数据源获取数据。

1.  在`src/museums/types.ts`中，定义`MuseumRepository`，这个模块将负责从数据源获取博物馆：

    ```js
    export interface MuseumRepository {
      getAll: () => Promise<Museum[]>
    }
    ```

1.  在`src/museums/controller.ts`中，向构造函数中添加一个注入的类`museumRepository`：

    ```js
    museumRepository that implements the MuseumRepository interface. By creating this and *lifting the dependencies*, we no longer need to return an empty array from our controller.Before we write any more logic, let's make sure our code runs and check if it is working. We're just missing one thing.
    ```

1.  创建一个名为`src/index.ts`的文件，导入`MuseumController`，实例化它，并调用`getAll`方法，记录其输出。现在，你可以注入一个伪仓库，它只是返回一个空数组：

    ```js
    import { Controller as MuseumController } from
      "./museums/controller.ts";
    const museumController = new MuseumController({
      museumRepository: {
        getAll: async () => []
      }
    })
    console.log(await museumController.getAll())
    ```

1.  运行它以检查它是否正常工作：

    ```js
    $ deno run src/index.ts 
    []
    ```

    就这样！我们刚刚从伪仓库函数接收到了一个空数组！

有了我们创建的这种抽象，我们的控制器现在与数据源解耦。其依赖关系通过构造函数注入，允许我们稍后不更改控制器而更改仓库。

我们刚才所做的称为**依赖倒置**——SOLID 原则中的**D**——它包括将部分依赖性提升到函数调用者。这使得独立测试内部函数变得非常容易，正如我们将在*第八章**测试——单元和集成*中看到的，我们将涵盖测试。

为了将我们刚刚编写的代码转换为完全功能的应用程序，我们需要有一个数据库或类似的东西。我们需要能够存储和检索博物馆列表的东西。我们现在来创建这个东西。

## 开发数据访问逻辑

在开发控制器的过程中，我们注意到我们需要能够获取数据；也就是说，仓库。这个模块将抽象所有对数据源的调用，在这个案例中，数据源存储博物馆。它将有一套非常明确的方法，任何想要访问数据的人都应该通过这个模块来访问。

我们已经在`src/museums/types.ts`中定义了其部分接口，所以让我们写一个实现它的类。现在，我们不会将它连接到真实数据库。我们将其作为内存数据库使用 ES6 Map。

让我们进入我们的仓库文件，并按照以下步骤开始编写我们的数据访问逻辑：

1.  打开 `src/museums/repository.ts` 文件并创建一个 `Repository` 类。

    它应该有一个名为 `storage` 的属性，这将是一个 JavaScript `Map`。`Map` 的键应该是字符串，值应该是 `Museum` 类型的对象：

    ```js
    import type { Museum, MuseumRepository } from
      "./types.ts";
    export class Repository implements MuseumRepository {
      storage = new Map<string, Museum>();
    }
    ```

    我们正在使用 TypeScript 泛型来设置我们的 `Map` 类型。请注意，我们引入了来自博物馆控制器 的 `Museum` 接口，以及由我们的类实现的 `MuseumRepository`。

    现在“数据库”已经“就绪”，我们必须暴露某些方法，这样人们就可以与它交互。上一节的要求是我们可以从数据库中获取所有记录。让我们接下来实现它。

1.  在仓库类内部，创建一个名为 `getAll` 的方法。它应该负责返回我们 `storage` `Map` 中的所有记录：

    ```js
    export class Repository implements MuseumRepository {
      storage = new Map<string, Museum>();
    src should only be accessible from the outside through a single file. This means that whoever wants to import stuff from src/museums should only do so from a single src/museums/index.ts file.
    ```

1.  创建一个名为 `src/museums/index.ts` 的文件，该文件导出博物馆的控制器 和仓库：

    ```js
    export { Controller } from "./controller.ts";
    export { Repository } from "./repository.ts";
    export type { Museum, MuseumController,
      MuseumRepository } from "./types.ts"; 
    ```

    为了保持一致性，我们需要去所有之前从不是 `src/museums/index.ts` 的文件导入的导入，并更改它们，使它们只从这个文件导入东西。

1.  将 `controller.ts` 和 `repository.ts` 的导入更新为从 `index` 文件导入：

    ```js
    import type { MuseumController, MuseumRepository }
      from "./index.ts";
    ```

    你可能已经猜到我们接下来必须做什么了…… 你还记得上一节的末尾，我们在博物馆控制器中注入了一个返回空数组的伪函数吗？让我们回到这里并使用我们刚刚编写的逻辑。

1.  回到 `src/index.ts`，导入我们刚刚创建的 `Repository` 类，并将其注入到 `MuseumController` 构造函数中：

    ```js
    import {
      Controller as MuseumController,
      Repository as MuseumRepository,
    } from "./museums/index.ts";
    const museumRepository = new MuseumRepository();
    const museumController = new MuseumController({
      museumRepository })
    console.log(await museumController.getAll())
    ```

    现在，让我们向我们的“数据库”添加一个 fixture，这样我们就可以检查它是否实际上正在打印一些内容。

1.  访问 `museumRepository` 中的存储属性并为其添加一个 fixture。

    这目前是一个反模式，因为我们直接访问模块的数据库，但我们将创建一个方法，以便我们以后可以正确添加 fixtures：

    ```js
    const museumRepository = new MuseumRepository();
    …
    museumRepository.storage.set
      ("1fbdd2a9-1b97-46e0-b450-62819e5772ff", {
      id: "1fbdd2a9-1b97-46e0-b450-62819e5772ff",
      name: "The Louvre",
    description: "The world's largest art museum 
        and a historic monument in Paris, France.",
      location: {
        lat: "48.860294",
        lng: "2.33862",
      },
    });
    console.log(await museumController.getAll())
    ```

1.  现在，让我们再次运行我们的代码：

    ```js
    $ deno run src/index.ts
    [
      {
        id: "1fbdd2a9-1b97-46e0-b450-62819e5772ff",
        name: "The Louvre",
        description: "The world's largest art
          museum and a historic monument in Paris,
            France.",
        location: { lat: "48.860294", lng: "2.33862" }
      }
    ]
    ```

    有了这个，我们的数据库连接就可以工作了，正如我们通过打印的 fixture 所看到的那样。

我们在上一节中创建的抽象使我们能够在不更改控制器的情况下更改数据源。这是我们正在使用的架构的一个优点。

现在，如果我们回顾一下我们的初始需求，我们可以确认我们已经完成了一半。我们已经创建了满足用例的业务逻辑——我们只是缺少 HTTP 部分。

## 创建网络服务器

现在我们已经有了我们的功能，我们需要通过一个网络服务器来暴露它。让我们使用我们从标准库中学到的知识来创建它，并按照以下步骤进行：

1.  在 `src/web` 文件夹中创建一个名为 `index.ts` 的文件，并在那里添加创建服务器的逻辑。我们可以从上一章的练习中复制和粘贴它：

    ```js
    import { serve } from
      "https://deno.land/std@0.83.0/http/server.ts";
    const PORT = 8080;
    const server = serve({ port: PORT });
    console.log(`Server running at
      https://localhost:${PORT}`);
    for await (let req of server) {
      req.respond({ body: 'museums api', status: 200 })
    }
    ```

    由于我们希望应用程序易于配置，我们不希望`port`在这里是硬编码的，而是可以从外部配置的。我们需要将这个服务器创建逻辑作为一个函数导出。

1.  将服务器逻辑创建包裹在一个函数中，该函数接收配置和`port`作为参数：

    ```js
    import { serve } from
      "https://deno.land/std@0.83.0/http/server.ts";
    port defining its type. 
    ```

1.  将这个函数的参数改为`interface`。这将有助于我们的文档，同时也会增加类型安全和静态检查：

    ```js
    interface CreateServerDependencies {
      configuration: {
        port: number
      }
    }
    export async function createServer({
      configuration: {
        port
      }
    }: CreateServerDependencies) {
    …
    ```

    现在我们已经配置了 Web 服务器，我们可以考虑将其用于我们的用例。

1.  回到`src/index.ts`，导入`createServer`，并使用它创建一个在端口`8080`上运行的服务器：

    ```js
    import { createServer } from "./web/index.ts";
    …
    createServer({
      configuration: {
        port: 8080
      }
    })
    …
    ```

1.  运行它，看看它是否正常工作：

    ```js
    $ deno run --allow-net src/index.ts
    Server running at http://localhost:8080
    [
      {
        id: "1fbdd2a9-1b97-46e0-b450-62819e5772ff",
        name: "The Louvre",
        description: "The world's largest art museum and a
          historic monument in Paris, France.",
        location: { lat: "48.860294", lng: "2.33862" }
      }
    ]
    ```

在这里，我们可以看到有一个日志记录服务器正在运行，以及来自上一节的日志结果。

现在，我们可以用`curl`测试 Web 服务器，以确保它正在工作：

```js
$ curl http://localhost:8080
museums api
```

正如我们所看到的，它起作用了——我们有一些相当基础的逻辑，但这仍然不能满足我们的要求，却能启动一个 Web 服务器。我们接下来要做的就是将这个 Web 服务器与之前编写的逻辑连接起来。

## 将 Web 服务器与业务逻辑连接

我们已经非常接近完成本章开始时计划要做的内容。我们目前有一个 Web 服务器和一些业务逻辑；缺少的是它们之间的连接。

将两件事连接起来的一个快速方法就是在`src/web/index.ts`上直接导入控制器并在此处使用它。在这里，应用程序将具有期望的行为，目前这样做没有任何问题。

然而，由于我们考虑的是一个可以无需太多问题就能扩展的应用程序架构，所以我们不会这样做。这是因为这将使我们的 Web 逻辑在隔离测试中变得非常难以实现，从而违背了我们的一条原则。

如果我们直接从 Web 服务器中导入控制器，每次在测试环境中调用`createServer`函数时，它将自动导入并调用`MuseumController`中的方法，这不是我们想要的结果。

我们再次使用依赖倒置将控制器的函数发送到 Web 服务器。如果这仍然看起来过于抽象，不用担心——我们马上就会看到代码。

为了确保我们没有忘记我们的初始目标，我们想要的是，当用户对`/api/museums`执行`GET`请求时，我们的 Web 服务器能够返回一个博物馆列表。

由于我们正在进行这项练习，所以我们暂时不会使用路由库。

我们只是想添加一个基本检查，以确保请求的 URL 和方法是我们想要回答的。如果是，我们想返回博物馆的列表。

让我们回到`createServer`函数并添加我们的路由处理程序：

```js
export async function createServer({
  configuration: {
    port
  }
}: CreateServerDependencies) {
  const server = serve({ port });
  console.log(`Server running at
    http://localhost:${port}`);
  for await (let req of server) {
    if (req.url === "/api/museums" && req.method === "GET")     
     {
req.respond({ 
body: JSON.stringify({ 
museums: [] 
}), 
status: 200 
      })
      continue
    }
    req.respond({ body: "museums api", status: 200 })
  }
}
```

我们为请求 URL 和方法添加了一个基本检查，并在它们符合初始要求时返回不同的响应。运行代码看看它的行为如何：

```js
$ deno run --allow-net src/index.ts 
Server running at http://localhost:8080
```

再次，用`curl`测试它：

```js
$ curl http://localhost:8080/api/museums
{"museums":[]}
```

它起作用了——太棒了！

现在，我们需要定义一个接口，以满足这个请求所需的内容。

我们最终需要一个函数，它返回一个博物馆列表，然后将其注入到我们的服务器中。让我们按照以下步骤在`CreateServerDependencies`接口中添加该功能：

1.  回到`src/web/index.ts`中，将`MuseumController`作为`createServer`函数的一个依赖项：

    ```js
    MuseumController type we defined in the museum's module. We're also adding a museum object alongside the configuration object.
    ```

1.  从博物馆控制器中调用`getAll`函数以获取所有博物馆的列表并响应请求：

    ```js
    export async function createServer({
      configuration: {
        port
      },
      createServer function, but we're not sending it when we call createServer. Let's fix that.
    ```

1.  回到`src/index.ts`，这是我们调用`createServer`函数的地方，并向`MuseumController`发送`getAll`函数。你也可以删除上一节直接调用控制器方法的代码，因为现在它没有任何用处：

    ```js
    import { createServer } from "./web/index.ts";
    import {
      Controller as MuseumController,
      Repository as MuseumRepository,
    } from "./museums/index.ts";
    const museumRepository = new MuseumRepository();
    const museumController = new MuseumController({
      museumRepository })
    museumRepository.storage.set
     ("1fbdd2a9-1b97-46e0-b450-62819e5772ff", {
      id: "1fbdd2a9-1b97-46e0-b450-62819e5772ff",
      name: "The Louvre",
      description: "The world's largest art museum 
        and a historic monument in Paris, France.",
      location: {
        lat: "48.860294",
        lng: "2.33862",
      },
    });
    createServer({
      configuration: { port: 8080 },
      museum: museumController
    })
    ```

1.  再次运行应用程序：

    ```js
    $ deno run --allow-net src/index.ts
    Server running at http://localhost:8080
    ```

1.  向 http://localhost:8080/api/museums 发送请求；你会得到一个博物馆列表：

    ```js
    $ curl localhost:8080/api/museums
    {"museums":[{"id":"1fbdd2a9-1b97-46e0-b450-62819e5772ff","name":"The Louvre","description":"The world's largest art museum and a historic monument in Paris, France.","location":{"lat":"48.860294","lng":"2.33862"}}]}
    ```

就这样——我们得到了博物馆列表！

我们已经完成了本节的任务，那就是将我们的业务逻辑连接到 web 服务器。

注意我们是如何使控制器方法可以被注入，而不是 web 服务器直接导入它。这之所以可能，是因为我们使用了依赖倒置。这是我们在这本书中会不断做的事情，无论何时我们想要解耦模块和函数，并提高它们的测试性。

在我们进行代码耦合的思维锻炼时，当我们想要使用不同的交付机制（如 CLI）来使用当前的业务逻辑时，没有任何阻碍。我们仍然可以重用相同的控制器和存储库。这意味着我们很好地使用了抽象来将业务逻辑与应用程序逻辑解耦。

既然我们已经了解了应用程序架构和文件结构的基础，并且也理解了背后的原因，我们可以开始查看可能帮助我们构建它的工具。

在下一节中，我们将查看 Deno 社区中现有的 HTTP 框架。我们不会花太多时间在这方面，但我们希望了解每个框架的优缺点，并最终选择一个来帮助我们完成剩余的旅程。

# 探索 Deno HTTP 框架

当你构建一个比简单教程更复杂的应用程序时，如果你不想采取纯粹的方法，你很可能会使用第三方软件。

显然，这不仅仅是 Deno 特有的。尽管有些社区比其他社区更愿意使用第三方模块，但所有社区都在使用第三方软件。

我们可以讨论人们为什么这样做或不做，但更常见的原因总是与可靠性和时间管理有关。这可能是因为你想使用经过实战测试的软件，而不是自己构建它。有时，这只是一个时间管理问题，即不想重新编写已经创建的东西。

我们必须说的一件重要事情是我们必须在对构建的应用程序进行耦合第三方软件时非常谨慎。我们并不是说你应该试图达到完全解耦的乌托邦，尤其是因为这会引入其他问题和很多间接性。我们要说的是，我们应该非常清楚将依赖项引入我们代码中的成本以及它引入的权衡。

在本章的第一部分，我们构建了一个 web 应用的基础，我们将在本书的其余部分向其添加功能。在其当前状态下，它仍然非常小，所以它除了标准库之外没有任何依赖。

在该应用中，我们做了一些我们相信不太容易扩展的事情，比如通过使用普通的`if`语句来匹配 URL 和 HTTP 方法来定义路由。

随着应用程序的增长，我们很可能会需要更高级的功能。这些需求可能从以不同格式处理 HTTP 请求体，到拥有更复杂的路由系统，处理头部和 cookies，或者连接到数据库。

因为我们不相信在开发应用程序时重新发明轮子，所以我们将分析几个目前存在于 Deno 社区中，并专注于创建 web 应用程序的库和框架。

我们将对现有的解决方案进行一般性了解，并探索它们的功能和方法。

最后，我们将选择我们认为在我们用例中提供最佳权衡的那个。

## 还有哪些替代方案？

在写作时，有一些第三方包提供了大量功能来创建 web 应用程序和 API。其中一些深受非常流行的 Node.js 包（如 Express.JS、Koa 或 hapi.js）的启发，而其他则受到 JavaScript 之外的其他框架（如 Laravel、Flask 等）的启发。

我们将探索其中的四个，它们在写作时非常流行且维护良好。请注意，由于 Deno 和提到的包正在快速发展，这可能会随时间而变化。

重要提示

Craig Morten 写了一篇非常好的文章，对可用的库进行了非常彻底的分析和解构。如果你想了解更多，我强烈推荐这篇文章（[`dev.to/craigmorten/what-is-the-best-deno-web-framework-2k69`](https://dev.to/craigmorten/what-is-the-best-deno-web-framework-2k69)）。

我们将尝试在要探索的包方面保持多样性。有一些提供了比其他更抽象和结构化的内容，而有一些提供的不仅仅是简单的实用函数和可组合功能。

我们将要探索的包如下：

+   Drash

+   Servest

+   Oak

+   Alosaur

让我们逐一看看它们。

### Drash

Drash ([`github.com/drashland/deno-drash`](https://github.com/drashland/deno-drash)) 旨在与现有的 Deno 和 Node.js 框架不同。这一动机在其维护者 Edward Bebbington 的一篇博客文章中明确提到，他比较了 Drash 与其他替代方案，并解释了其创建的动机 ([`dev.to/drash_land/what-makes-drash-different-idd`](https://dev.to/drash_land/what-makes-drash-different-idd)).

这些动机很好，灵感来自于非常流行的软件工具如 Laravel、Flask 和 Tonic，这些决策大部分得到了证实。你一查看 Drash 的代码，就能发现一些相似之处。

与 Express.js 或 Koa 等库相比，它确实提供了一种不同的方法，正如文档所述：

“Deno 与 Node.js 的不同之处在于，Drash 旨在与 Express 或 Koa 不同，利用资源并采用完整的类式系统。”

主要区别在于，Drash 不想提供应用程序对象，让开发者可以注册他们的端点，像一些流行的 Node.js 框架那样。它将端点视为在类中定义的资源，与以下内容相似：

```js
import { Drash } from
  "https://deno.land/x/drash@v1.2.2/mod.ts";
class HomeResource extends Drash.Http.Resource {
  static paths = ["/"];
  public GET() {
    this.response.body = "Hello World!";
    return this.response;
  }
}
```

这些资源随后被插到 Drash 的应用程序中：

```js
const server = new Drash.Http.Server({
  response_output: "text/html",
  resources: [HomeResource]
});
server.run({
  hostname: "localhost",
  port: 1447
});
```

在这里，我们可以直接声明它实际上与我们在上面提到的其他框架不同。这些差异是有意的，旨在取悦喜欢这种方法并解决其他框架问题的开发者。这些用例在 Drash 的文档中解释得非常清楚。

Drash 基于资源的方法绝对值得关注。它从非常成熟的软件如 Flask 和 Tonic 得到的灵感确实为桌面带来了东西，并提出了一种解决方案，有助于解决无观点工具的常见问题。文档完整且易于理解，这使得在选择构建应用程序的工具时，它成为了一个很好的资产。

### Servest

Servest ([`servestjs.org/`](https://servestjs.org/)) 自称为适用于 Deno 的*“渐进式 HTTP 服务器”*。

它被创建的一个原因是因为其作者希望让标准库的 HTTP 模块中的一些 API 更容易使用，并实验新特性。后者是在需要稳定性的标准库中真正难以实现的事情。

Servest 直接关注与标准库的 HTTP 模块的比较。其项目主页上直接声明的一个主要目标，就是使其容易从标准库的 HTTP 模块迁移到 Servest。这很好地总结了 Servest 的愿景。

从 API 角度来看，Servest 与我们从 Express.js 和 Koa 熟悉的东西非常相似。它提供了一个应用程序对象，可以在其中注册路由。你也可以看到明显受到了标准库模块所提供内容的启发，正如我们在以下代码片段中所见：

```js
import { createApp } from
  "https://servestjs.org/@v1.1.4/mod.ts";
const app = createApp();
app.handle("/", async (req) => {
  await req.respond({
    status: 200,
    headers: new Headers({
      "content-type": "text/plain",
    }),
    body: "Hello, Servest!",
  });
});
app.listen({ port: 8899 });
```

我们可以识别出知名 Node.js 库中的应用对象和标准库中的请求对象，以及其他内容。

在此基础上，Servest 还提供了诸如直接渲染 JSX 页面、服务静态文件和认证等常见功能，文档也非常清晰，充满了示例。

Servest 试图利用 Node.js 用户的知识和熟悉度，同时利用 Deno 提供的好处，这是一个有希望的混合。其渐进性质为桌面带来了非常漂亮的功能，承诺会让开发者的生产力比使用标准库 HTTP 包时更高。

### Oak

Oak ([`oakserver.github.io/oak/`](https://oakserver.github.io/oak/)) 目前是创建 web 应用程序的最受欢迎的 Deno 库。它的名字来源于 Koa 的词语游戏，Koa 是一个非常流行的 Node.js 中间件框架和 Oak 的主要灵感来源。

由于其深受启发，其 API 使用异步函数和上下文对象与 Koa 相似并不令人意外。Oak 还包括一个路由器，也是受`@koa/router`启发的。

如果你熟悉 Koa，下面的代码可能看起来会很熟悉：

```js
import { Application } from
  "https://deno.land/x/oak/mod.ts";
const app = new Application();
app.use((ctx) => {
  ctx.response.body = "Hello world!";
});
await app.listen("127.0.0.1:8000");
```

对于那些不熟悉 Koa 的人来说，我们会简要地解释一下，因为理解它将帮助你理解 Oak。

Koa 通过使用现代 JavaScript 特性提供了一个最小化和无观点的方法。Koa 最初被创建（由创建 Express.js 的同一团队）的原因之一是，其创作者想要创建一个利用现代 JavaScript 特性的框架，而不是像 Express 那样，Express 是在 Node.js 的早期创建的。

团队想要使用诸如 promises 和 async/await 等新特性，然后解决开发者在使用 Express.JS 时面临的挑战。其中大多数挑战与错误处理、处理回调和某些 API 的不清晰有关。

Oak 的流行并非空穴来风，它在 GitHub 上的星级与其他选项的距离反映了这一点。单凭 GitHub 的星级并不能说明什么，但结合打开和关闭的问题、发布的版本等，我们可以看出人们为什么信任它。当然，这种熟悉度在的这个包的流行中起了很大的作用。

在其当前状态下，Oak 是一个构建 web 应用程序的固体（就 Deno 的社区标准而言），因为它提供了一组非常清晰和直接的功能。

### Alosaur

Alosaur ([`github.com/alosaur/alosaur`](https://github.com/alosaur/alosaur)) 是一个基于装饰器和类的 Deno web 应用程序框架。它在某种程度上与 Drash 相似，尽管最后的实现方式有所不同。

在其主要功能中，Alosaur 提供了诸如模板渲染、依赖注入和 OpenAPI 支持等功能。这些功能是在所有我们在这里介绍的替代方案的标准之上添加的，如中间件支持和路由。

这个框架的方法是使用类定义控制器，并使用装饰器定义其行为，如下面的代码所示：

```js
import { Controller, Get, Area, App } from
  'https://deno.land/x/alosaur@v0.21.1/mod.ts';
@Controller() // or specific path @Controller("/home")
export class HomeController {
    @Get() // or specific path @Get("/hello")
    text() {
        return 'Hello world';
    }
}
// Declare module
@Area({
    controllers: [HomeController],
})
export class HomeArea {}
// Create alosaur application
const app = new App({
    areas: [HomeArea],
});
app.listen();
```

在这里，我们可以看到应用程序的实例化与 Drash 有相似之处。它还使用 TypeScript 装饰器来声明框架的行为。

Alosaur 与前面提到的大多数库采取了不同的方法，主要原因在于它并不试图简约。相反，它提供了一组在构建某些类型的应用程序时证明有用的特性。

我们决定对其进行研究，不仅因为它能完成预期的工作，还因为它在 Node.js 和 Deno 领域拥有的一些不常见的特性。这包括诸如依赖注入和 OpenAPI 支持等功能，这是其他展示的解决方案所没有的。同时，它保留了诸如模板渲染等特性，这可能你们从 Express.JS 中熟悉，但在更现代的框架中就不那么熟悉了。

最终解决方案在提供的功能方面非常有前途且完整。这绝对是值得关注的东西，这样你就可以看到它是如何发展的。

## 结论

在审视了所有展示的解决方案并认识到它们都有优点之后，我们决定在本书的剩余部分使用 Oak。

这并不意味着本书将重点介绍 Oak。不会的，因为它只会处理 HTTP 和路由。Oak 的简约方法将与我们接下来要做的非常吻合，帮助我们逐步创建功能，而不会让它成为障碍。它还是 Deno 社区中最稳定、维护良好和最受欢迎的选项之一，这对我们的决定有明显的影响。

请注意，这个决定并不意味着我们将在接下来的几章中学到的内容不能在任何替代方案中完成。事实上，由于我们将如何组织和架构我们的代码，我们相信很容易就能跟上使用不同框架我们要做的绝大多数事情。

在本书的剩余部分，我们将使用其他第三方模块来帮助我们构建我们提出的功能。我们决定深入研究处理 HTTP 的库，原因是这是我们即将开发的应用程序的基本交付机制。

# 摘要

在本章中，我们终于开始构建一个利用我们对 Deno 知识的应用程序。我们首先考虑了构建应用程序时我们将拥有的主要目标及其架构。这些目标将为我们本书中关于架构和结构的多数对话定下基调，因为我们将会不断回顾它们，确保我们与它们保持一致。

我们首先创建了我们的文件结构，并试图实现我们第一个应用程序目标：拥有一个列出博物馆的 HTTP 端点。我们先构建了简单的业务逻辑，并在需要关注分离和职责隔离等需求时逐步推进。这些需求定义了我们的架构，证明了我们所创建的层和抽象的好处，并展示了它们所提供的价值。

通过明确责任和模块接口，我们理解到我们可以暂时使用内存数据库来构建我们的应用程序，这就是我们所做的。借助这种方法，我们能够构建出符合本章要求的应用程序，并且层次分离允许我们稍后回来，无需任何问题地将它更改为一个适当的持久层。在定义了业务和数据访问逻辑之后，我们使用标准库创建了一个 Web 服务器作为交付机制。在创建了一个非常简单的路由系统之后，我们插入了之前构建的业务逻辑，满足了本章的主要要求：拥有一个返回博物馆列表的应用程序。

我们在不创建业务逻辑、数据获取和交付层之间直接耦合的情况下做到了这一切。这是我们认为当我们开始添加复杂性、扩展我们的应用程序并向其添加测试时将非常有用的东西。

本章通过查看 Deno 社区目前存在的 HTTP 框架和库，并理解它们之间的差异和方法来结束。其中一些使用对 Node.js 用户熟悉的方法，而其他则深入使用 TypeScript 及其特性来创建更具结构的 Web 应用程序。通过查看四个目前可用的解决方案，我们了解到了社区正在开发的内容以及他们可能采取的方向。

我们最终选择了 Oak，这是一个非常最小化和相对成熟解决方案，以帮助我们解决在本书剩余部分遇到的路由和 HTTP 挑战。

在下一章中，我们将开始将 Oak 添加到我们的代码库中，并添加一些有用特性，如认证和授权，使用中间件等概念，并使我们的应用程序达到我们设定的目标。

让我们开始吧！
