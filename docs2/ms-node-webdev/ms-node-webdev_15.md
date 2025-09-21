

# 第十四章：创建 RESTful Web 服务

本章解释了如何使用 Node.js 创建通过 HTTP 请求提供数据访问的 Web 服务，这是单页应用（**SPAs**）的关键推动力。本章从基本的 Web 服务开始，然后引入更复杂的功能，如部分更新和数据验证。*表 14.1* 将本章置于上下文中。

表 14.1：将 RESTful Web 服务置于上下文中

| 问题 | 答案 |
| --- | --- |
| 它们是什么？ | RESTful Web 服务通过 HTTP 请求提供对数据的访问。服务器不是在 HTML 内容中嵌入数据，而是以“原始”数据的形式响应，通常为 JSON 格式。 |
| 为什么它们有用？ | Web 服务允许客户端使用 HTTP 请求执行数据操作，如查询或更新数据。这通常用于在浏览器中执行的 JavaScript 代码，尽管任何类型的客户端都可以消费 Web 服务。 |
| 如何使用它们？ | HTTP 请求方法/动词用于表示操作，而请求 URL 识别应执行操作的数据。 |
| 是否有陷阱或限制？ | 没有创建 Web 服务的标准方式，这导致它们的设计存在显著差异。 |
| 是否有替代方案？ | 大多数现代 Web 应用程序都需要某种形式的 Web 服务来向客户端 JavaScript 应用程序交付数据。尽管如此，对于纯粹往返且不需要支持客户端的应用程序，Web 服务不是必需的。 |

*表 14.2* 总结了本章内容。

表 14.2：本章总结

| 问题 | 解决方案 | 列表 |
| --- | --- | --- |
| 定义 Web 服务 | 使用标准请求处理程序并返回 JSON 数据而不是 HTML。 | *9-15* |
| 整合创建 Web 服务所需的代码 | 将处理 HTTP 请求的代码分离，以便可以将数据处理代码隔离。 | *16-18, 43-45* |
| 使用 Web 服务更新数据 | 处理 `PUT` 和 `PATCH` 请求。 | *19-26* |
| 描述复杂的数据更改 | 使用 JSON Patch 规范。 | *27-30* |
| 验证 Web 服务接收到的数据值 | 在将数据传递给处理数据的代码之前执行验证。 | *31-37* |
| 验证通过 Web 服务接收到的数据组合 | 执行模型验证。 | *38-41* |

# 准备本章内容

本章使用了 *第十三章* 中的 `part2app` 项目。本章的示例与一个简单的命令行客户端应用程序更容易理解，该应用程序发送 HTTP 请求并显示接收到的响应。为了准备，请在 `part2app` 文件夹中运行 *列表 14.1* 中显示的命令以安装 `Inquirer` 包 ([`github.com/SBoudrias/Inquirer.js`](https://github.com/SBoudrias/Inquirer.js))，该包提供用于提示用户的特性。

**提示**

您可以从[`github.com/PacktPublishing/Mastering-Node.js-Web-Development`](https://github.com/PacktPublishing/Mastering-Node.js-Web-Development)下载本章的示例项目——以及本书所有其他章节的示例项目。查看*第一章*了解如果在运行示例时遇到问题如何获取帮助。

列表 14.1：安装包

```js
npm install @inquirer/prompts@3.3.0 
```

在`src/cmdline`文件夹中创建一个名为`main.mjs`的文件，并添加*列表 14.2*中显示的内容。`.mjs`文件扩展名告诉 Node.js 将此文件视为 JavaScript 模块，并允许使用`import`语句。

列表 14.2：src/cmdline 文件夹中的 main.mjs 文件的内容

```js
import { select } from "@inquirer/prompts";
import { ops } from "./operations.mjs";
(async function run() {
    let loop = true;
    while (loop) {
        const selection = await select({
            message: "Select an operation",
            choices: [...Object.keys(ops).map(k => {return { value: k }})]
        });
        await ops[selection]();
    }
})(); 
```

此代码使用`Inquirer`包提示用户选择要执行的操作。用户面前呈现的选项是从一个对象属性中获得的，并做出选择将执行分配给该属性的函数。在`src/cmdline`文件夹中添加一个名为`operations.mjs`的文件，并添加*列表 14.3*中显示的内容。

列表 14.3：src/cmdline 文件夹中的 operations.mjs 文件的内容

```js
export const ops = {
    "Test": () => {
        console.log("Test operation selected");
    },
    "Exit": () => process.exit()
} 
```

此文件提供了用户可以选择的操作，包括一个`Test`操作以开始并确保一切按预期工作，以及一个使用 Node.js `process.exit`方法终止进程的`Exit`选项。*列表 14.4*将条目添加到`package.json`文件的脚本部分以运行命令行客户端。

列表 14.4：在 part2app 文件夹中的 package.json 文件中添加脚本

```js
...
"scripts": {
    "server": "tsc-watch --noClear --onsuccess \"node dist/server/server.js\"",
    "client": "webpack serve",
    "start": "npm-run-all --parallel server client",
    **"cmdline": "node --watch ./src/cmdline/main.mjs"**
},
... 
```

新的条目将使用 Node.js 执行`main.mjs`文件。`--watch`参数将 Node.js 置于监视模式，如果检测到更改，它将重新启动。

## 准备网络服务

为了介绍网络服务做准备，创建`src/server/api`文件夹，并向其中添加一个名为`index.ts`的文件，其内容如*列表 14.5*所示。

列表 14.5：src/server/api 文件夹中的 index.ts 文件的内容

```js
import { Express } from "express";
export const createApi = (app: Express) => {
    // TODO - implement API
} 
```

此文件目前只是一个占位符，但将被用于配置 Express 以处理 HTTP API 请求。最后的更改是调用*列表 14.5*中定义的函数来设置网络服务，如*列表 14.6*所示。

列表 14.6：在 src/server 文件夹中的 server.ts 文件中配置 Express

```js
import { createServer } from "http";
import express, {Express } from "express";
import httpProxy from "http-proxy";
import helmet from "helmet";
import { engine } from "express-handlebars";
import { registerFormMiddleware, registerFormRoutes } from "./forms";
**import { createApi } from "./api"****;**
const port = 5000;
const expressApp: Express = express();
const proxy = httpProxy.createProxyServer({
    target: "http://localhost:5100", ws: true
});
expressApp.set("views", "templates/server");
expressApp.engine("handlebars", engine());
expressApp.set("view engine", "handlebars");
expressApp.use(helmet());
expressApp.use(express.json());
registerFormMiddleware(expressApp);
registerFormRoutes(expressApp);
**createApi(expressApp);**
expressApp.use("^/$", (req, resp) => resp.redirect("/form"));
expressApp.use(express.static("static"));
expressApp.use(express.static("node_modules/bootstrap/dist"));
expressApp.use((req, resp) => proxy.web(req, resp));
const server = createServer(expressApp);
server.on('upgrade', (req, socket, head) => proxy.ws(req, socket, head));
server.listen(port,
    () => console.log(`HTTP Server listening on port ${port}`)); 
```

在`part2app`文件夹中运行*列表 14.7*中显示的命令以启动开发工具。

列表 14.7：启动开发工具

```js
npm start 
```

打开第二个命令提示符，导航到`part2app`文件夹，并运行*列表 14.8*中显示的命令以启动命令行客户端。

列表 14.8：启动命令行客户端

```js
npm run cmdline 
```

`Inquirer`包提供的功能呈现单选选项，如下所示：

```js
? Select an operation (Use arrow keys)
> Test
Exit 
```

使用箭头键可以上下移动选择列表。选择`Test`操作将显示测试消息，选择`Exit`将终止进程。Node.js 正在运行监视模式，这意味着如果检测到更改，它将再次启动命令行客户端。如果您想完全停止客户端，请按*Ctrl + C*。

# 理解网络服务

关于什么是网络服务，没有明确的共识，没有单一的标准可以遵循，也没有一套广泛采用的模式。相反，情况正好相反：有无数的意见，无数的模式，以及关于向客户端“正确”提供数据的“正确”方式的无限争论。

围绕网络服务的混乱和噪音可能令人难以承受，难以知道从哪里开始。然而，缺乏标准化可能是一种解放，因为它意味着项目可以专注于仅提供客户端需要的功能，而不需要标准化有时会带来的任何样板或开销。

网络服务只是通过 HTTP 访问的数据访问 API。RESTful 网络服务只是使用 HTTP 请求的一些方面来确定客户端想要使用 API 的哪些部分的网络服务。术语*RESTful*来自**表征状态转移**（**REST**）模式，但由于网络服务中的变化和适应如此之多，只有 REST 的核心前提被广泛使用，即 API 是通过 HTTP 方法和 URL 的组合来定义的。HTTP 方法，如 GET 或 POST，定义了将要执行的操作类型，而 URL 指定了操作将应用到的数据对象或对象。

项目可以自由地以任何方式创建网络服务 API，但最好的网络服务是那些简单易用的服务。以下是一个可能标识应用程序管理的数据的 URL 示例：

```js
/api/results/1 
```

在使用 URL 来标识数据时没有限制，只要客户端和服务器都理解 URL 格式，以便可以明确地标识数据。如果一个应用程序在数据库中存储数据，那么 URL 通常使用主键来标识特定的值，但这只是一个常见的约定，并不是必需的。

URL 标识数据，但指定对数据进行什么操作的是 HTTP 请求方法。*表 14.3*描述了在 Web 服务中常用且传统上表示的操作的 HTTP 方法。

表 14.3：常用 HTTP 方法

| 方法 | 描述 |
| --- | --- |
| `GET` | 此方法用于检索一个或多个数据值 |
| `POST` | 此方法用于存储新的数据值 |
| `PUT` | 此方法用于替换现有的数据值 |
| `PATCH` | 此方法用于更新现有数据值的一部分 |
| `DELETE` | 此方法用于删除数据值 |

一个网络服务通过组合 URL 和方法来提供 API，通常返回 JSON 数据。对于不查询数据的操作，会返回操作结果，这也可以是 JSON 数据。一个基本的网络服务可能提供*表 14.4*中描述的组合，该表还描述了网络服务将产生的结果。

**注意**

早期的网络服务使用 XML 而不是 JSON。JSON 成为事实上的标准，因为它简单且易于 JavaScript 客户端解析，但你仍然会偶尔看到对 XML 的引用，例如浏览器提供的用于发送 HTTP 请求的`XMLHttpRequest`对象（尽管这些已经被更现代的 Fetch API 所取代）。

表 14.4：典型的网络服务

| 方法 | URL | 描述 |
| --- | --- | --- |

|

```js
`GET` 
```

|

```js
`/api/results/1` 
```

| 此组合获取 ID 为`1`的单个值，以`Result`对象的 JSON 表示形式表达。如果没有这样的 ID，将返回 404 响应。 |
| --- |

|

```js
`GET` 
```

|

```js
`/api/results` 
```

| 此组合获取所有可用的数据值，以`Result`对象数组的 JSON 表示形式表达。如果没有数据，将返回一个空数组。 |
| --- |

|

```js
`GET` 
```

|

```js
`/api/results?name=Alice` 
```

| 此组合查找所有具有`name`值为`Alice`的值，并返回一个`Result`对象数组的 JSON 表示形式。如果没有匹配的数据，将返回一个空数组。 |
| --- |

|

```js
`POST` 
```

|

```js
`/api/results` 
```

| 此组合存储一个值，并返回存储数据的 JSON 表示形式。 |
| --- |

|

```js
`DELETE` 
```

|

```js
`/api/results/1` 
```

| 此组合删除 ID 为`1`的单个值，并返回一个包含`success`属性的 JSON 对象，该属性具有`boolean`值，表示结果。 |
| --- |

**理解微服务**

任何关于网络服务的研究很快就会带你进入微服务的世界，这也是为什么我在上一节建议将其作为搜索词的原因。微服务是一种围绕业务能力设计应用程序的方法，这通常涉及到网络服务。可以在[`microservices.io`](https://microservices.io)找到关于微服务的良好概述，以及详细的设计模式。

我认为微服务很有趣，但对于大多数项目来说应该避免使用。微服务解决的核心问题是功能失调的开发组织，无法管理以提供协调一致的软件发布。鉴于任何由三个或更多开发者组成的团队都会立即分裂成争夺资源、争论设计问题并互相指责延误的派系，这是一个许多项目都会面临的问题。

微服务通过让开发团队在大部分情况下独立工作，并仅就项目不同部分如何集成达成一致来尝试解决这些问题。有一些优秀的工具旨在支持微服务，其中最著名的是 Kubernetes，但这些工具极其复杂，采用微服务感觉像是放弃人力资源管理复杂性，专注于软件管理复杂性。根据我的经验，很少有 HR 问题是通过增加开发工具的复杂性来解决的，因此我对微服务是否是解决复杂组织问题的实际方法持怀疑态度。你应该形成自己的观点，但我的建议是在采用微服务之前仔细思考，并问问自己，在联邦开发模型中，你的同事是否会表现得比现在更好。

# 创建基本的 RESTful Web 服务

作为第一步，*列表 14.9* 创建了一个实现*表 14.4* 中描述的 URL 和 HTTP 方法组合的一些组合的 Web 服务。

列表 14.9：在 src/server/api 文件夹中的 index.ts 文件中创建基本 Web 服务

```js
import { Express } from "express";
**import repository from "../data"****;**
export const createApi = (app: Express) => {
   ** app.get("/api/results", async (req, resp) => {**
 **if (req.query****.name) {**
 **const data = await repository.getResultsByName(**
 **req.query.name.toString(), 10);**
 **if (data.length** **> 0) {**
 **resp.json(data);**
 **} else {**
 **resp.writeHead(404);**
 **}**
 **}   else {**
 **resp.json(await repository.getAllResults(10****));**
 **}**
 **resp.end();**
 **});**
} 
```

列表展示了通过重新利用为往返请求创建的应用程序部分来创建客户端 API 的简便性。大多数网络服务都是这样开始的，但正如后续章节所解释的，存在一些问题和改进之处。

列表 14.10：在 src/cmdline 文件夹中的 operations.mjs 文件中添加操作

```js
**import { input } from "@inquirer/prompts";**
**const baseUrl = "http://localhost:5000";**
export const ops = {
    **"Get All": () => sendRequest("GET", "/api/results"),**
 **"Get Name": async () => {**
**const name = await input({ message: "Name?"});**
 **await sendRequest("GET", `/api/results?name=${name}`);**
 **},**
    "Exit": () => process.exit()
}
**const sendRequest = async (method, url, body, contentType) => {**
 **const response = await fetch****(baseUrl + url, {**
 **method, headers: { "Content-Type": contentType ?? "application/json"},**
 **body: JSON.stringify(body)**
 **});**
 **if (response.status == 200) {**
 **const** **data = await response.json();**
 **(Array.isArray(data) ? data : [data])**
 **.forEach(elem => console.log(JSON.****stringify(elem)));**
 **} else {**
 **console.log(response.status + " " + response.statusText);**
 **}**
**}** 
```

Node.js 支持 Fetch API，这是基于浏览器的 JavaScript 代码通常用于发起 HTTP 请求的 API。*列表 14.10* 中的更改添加了一个`sendRequest`函数，用于发送 HTTP 请求并显示其结果，并添加了“获取所有”和“获取名称”操作。其中，“获取名称”操作使用`Inquirer`来提示输入名称，然后将其添加到 HTTP 请求的查询字符串中。

当检测到*列表 14.10* 中的更改时，命令行客户端将重新启动。选择“获取所有”选项并按*回车*将显示所有可用数据，如下所示：

```js
...
{"id":3,"name":"Alice","age":35,"years":10,"nextage":45}
{"id":2,"name":"Bob","age":35,"years":10,"nextage":45}
{"id":1,"name":"Alice","age":35,"years":5,"nextage":40}
... 
```

选择“获取名称”操作并按*回车*键，系统将提示输入一个名称。输入`Alice`并按*回车*，你将看到匹配的结果：

```js
...
{"id":3,"name":"Alice","age":35,"years":10,"nextage":45}
{"id":1,"name":"Alice","age":35,"years":5,"nextage":40}
... 
```

如果你输入一个不存在的名称，网络服务将响应一个 404 未找到的错误。

## 获取 Web 服务的数据

Web 服务只支持*表 14.4* 中两种组合的原因是，这些是唯一可以使用为往返应用程序需求创建的存储库执行的操作。

没有一种单一的最好方法来解决这个问题，需要做出妥协。一些项目有网络服务和往返需求足够相似，可以共享一个仓库，但这些情况很少见，试图在这两者之间强制一致性可能会导致应用程序的一个或两个部分做出妥协。

**理解 GraphQL**

GraphQL ([`graphql.org`](https://graphql.org)) 是一种向客户端提供数据的不同方法。常规的 RESTful 网络服务提供一组特定的操作，这些操作对所有客户端产生相同的结果。如果客户端需要在响应中获取额外的数据，例如，那么开发者必须修改网络服务，然后所有客户端都将接收到这些新数据。

GraphQL 仍然使用 HTTP 请求，数据仍然以 JSON 格式表达，但客户端可以执行自定义查询，包括选择要包含的数据值以及以不同的方式过滤数据。这意味着客户端可以只接收他们需要的数据，并且不同的客户端可以接收不同的数据。

GraphQL 很好，但它比常规的 RESTful 网络服务更复杂，无论是在服务器端开发还是在执行客户端查询方面，而且大多数项目更适合传统的 RESTful 网络服务，这些服务向所有客户端提供固定的一组操作和结果。GraphQL 在那些拥有大量数据且客户端将以不同方式使用这些数据的项目中表现出色，而这些数据服务器端开发者无法预知。但是，对于大多数其他项目来说，GraphQL 过于复杂，而传统的网络服务更容易创建和消费。

另一种选择是为应用程序的每个部分创建单独的仓库，这允许每个部分独立发展，但不可避免地会导致一定程度上的代码重复，因为某些操作可能同时被往返和 Web 服务客户端需要。

另一种选择——也是本章使用的方法——是创建原始仓库的子类并添加缺失的功能。当应用程序的一部分所需的功能是其他地方所需功能的一个子集时，这种方法是可行的，例如在示例应用程序中就是这样。*列表 14.11* 定义了一个新的接口，该接口描述了为网络服务所需的其他功能。稍后还需要更多方法，但现阶段这已经足够了。

**提示**

如果你不确定从哪里开始，那么先从创建一个子类开始。如果你发现你需要替换从基类继承的大多数功能，那么你应该将代码分成两个独立的仓库。

列表 14.11：在 src/server/data 文件夹中的 repository.ts 文件中定义一个新的接口

```js
export interface Result {
    id: number,
    name: string,
    age: number,
    years: number,
    nextage: number
}
export interface Repository {
    saveResult(r: Result):  Promise<number>;
    getAllResults(limit: number) : Promise<Result[]>;
    getResultsByName(name: string, limit: number): Promise<Result[]>;
}
**export interface ApiRepository extends** **Repository {**
 **getResultById(id: number): Promise<Result | undefined>;**
 **delete(id: number) :** **Promise<boolean>;**
**}** 
```

新的方法允许通过其 ID 请求单个 `Result` 对象，并且可以通过指定 ID 来删除数据。*列表 14.12* 更新了 `OrmRepository` 类以实现新的接口。

列表 14.12：在 src/server/data 文件夹中的 orm_repository.ts 文件中实现接口

```js
import { Sequelize } from "sequelize";
**import { ApiRepository, Result } from "./repository";**
import { addSeedData, defineRelationships,
    fromOrmModel, initializeModels } from "./orm_helpers";
import { Calculation, Person, ResultModel } from "./orm_models";
**export class OrmRepository implements ApiRepository {**
    sequelize: Sequelize;
    // ...constructor and methods omitted for brevity...
    **async getResultById(id: number): Promise<Result | undefined> {**
 **const model =** **await ResultModel.findByPk(id, {**
 **include: [Person, Calculation ]**
 **});**
 **return model ? fromOrmModel(model): undefined;**
 **}**
**async delete(id: number): Promise<boolean> {**
 **const count = await ResultModel.destroy({ where: { id }});**
 **return count ==** **1;**
 **}**
} 
```

*列表 14.13*更新`data`模块的导出，以添加 API 特定的仓库。

列表 14.13：在 src/server/data 文件夹中的 index.ts 文件中更新导出

```js
**import { ApiRepository } from "./repository";**
import { OrmRepository } from "./orm_repository";
**const repository: ApiRepository = new OrmRepository();**
export default repository; 
```

*列表 14.14*使用新的仓库接口向网络服务添加功能。这段代码难以阅读，但将在下一节中解决。

列表 14.14：在 src/server/api 文件夹中的 index.ts 文件中添加功能

```js
import { Express } from "express";
import repository from "../data";
export const createApi = (app: Express) => {
    app.get("/api/results", async (req, resp) => {
        if (req.query.name) {
            const data = await repository.getResultsByName(
                req.query.name.toString(), 10);
            if (data.length > 0) {
                resp.json(data);
            } else {
                resp.writeHead(404);
            }
        }   else {
                resp.json(await repository.getAllResults(10));
        }
        resp.end();
    });
   **app.all("/api/results/:id",** **async (req, resp) => {**
 **const id = Number.parseInt(req.params.id);**
 **if (req.method == "GET") {**
 **const result =** **await repository.getResultById(id);**
 **if (result == undefined) {**
 **resp.writeHead(404);**
 **} else {**
 **resp.json(result);**
 **}**
 **} else if (req.****method == "DELETE") {**
 **let deleted = await repository.delete(id);**
 **resp.json({ deleted });**
 **}**
 **resp.end();**
 **})**
 **app.post("/api/results", async (req, resp) => {**
**const { name, age, years} = req.body;**
 **const nextage = Number.parseInt(age) + Number.parseInt(years);**
 **const id = await repository.saveResult({** **id: 0, name, age,**
 **years, nextage});**
 **resp.json(await repository.getResultById(id));**
 **resp.end();**
 **});**
} 
```

新的路由增加了按 ID 查询、存储新结果和删除现有结果的支持。存储新结果的能力取决于按 ID 查询的能力，因为`Repository.saveResult`方法返回的结果与网络服务 POST 请求所需的结果不匹配。`saveResult`方法返回新存储对象的`Id`，因此需要额外的查询来获取已存储的`Result`对象，以便将其发送回客户端。*列表 14.15*向命令行客户端添加了依赖于新网络服务功能的新操作。

列表 14.15：在 src/cmdline 文件夹中的 operations.mjs 文件中添加功能

```js
...
export const ops = {
    "Get All": () => sendRequest("GET", "/api/results"),
    "Get Name": async () => {
        const name = await input({ message: "Name?"});
        await sendRequest("GET", `/api/results?name=${name}`);
    },
    **"Get ID": async () => {**
 **const id = await input({** **message: "ID?"});**
 **await sendRequest("GET", `/api/results/${id}`);**
 **},**
 **"Store":** **async () => {**
 **const values = {**
 **name: await input({message: "Name?"}),**
 **age: await input({****message: "Age?"}),**
 **years: await input({message: "Years?"})**
 **};**
 **await sendRequest("POST", "****/api/results", values);**
 **},**
 **"Delete": async () => {**
 **const id = await input({ message: "ID?"});**
**await sendRequest("DELETE", `/api/results/${id}`);**
 **},**
    "Exit": () => process.exit()
}
... 
```

使用命令行客户端，选择`获取 ID`选项，并在提示时输入`3`，将产生以下结果：

```js
...
{"id":3,"name":"Alice","age":35,"years":10,"nextage":45}
... 
```

对于数据库中不存在的`ID`，网络服务将返回 404 未找到响应。当提示输入名称、年龄和年数值时，选择`存储`选项并输入`Drew, 50, and 5`，响应将显示存储的新记录：

```js
...
{"id":4,"name":"Drew","age":50,"years":5,"nextage":55}
... 
```

选择`获取全部`选项将显示数据库中的新记录以及现有数据（但请注意，每次服务器端应用程序重启时，数据库都会重置并重新播种，因此不要进行任何代码更改）：

```js
...
**{"id":4,"name":"Drew","age":50,"years":5,"nextage":55}**
{"id":3,"name":"Alice","age":35,"years":10,"nextage":45}
{"id":2,"name":"Bob","age":35,"years":10,"nextage":45}
{"id":1,"name":"Alice","age":35,"years":5,"nextage":40}
... 
```

选择**删除**选项，并在提示时输入新存储项的`ID`。结果是包含已删除属性以指示结果的 JSON 对象：

```js
...
{"deleted":true}
... 
```

选择**获取全部**选项将确认数据已被删除。

**理解 OpenAPI**

OpenAPI 规范([`www.openapis.org`](https://www.openapis.org))是描述网络服务的标准，可以帮助客户端开发者了解网络服务应该如何使用，并提供对它提供访问的数据的描述。有工具和包可以从 OpenAPI 描述自动生成客户端代码，并且一些用于定义网络服务的 JavaScript 包将自动生成 OpenAPI 文档。

OpenAPI 是一个好主意，但它通常被用作描述性文档的替代品，这往往会在 Web 服务提供的功能与开发者意图使用它们之间留下差距。如果你在项目中采用 OpenAPI，你必须确保补充描述它产生的说明，以说明你的 Web 服务应该如何被消费。

## 分离 HTTP 状态码

*列表 14.15* 中的 Web 服务支持 *表 14.4* 中描述的所有 HTTP 方法与 URL 的组合，但代码难以理解。Web 服务有三个任务要执行：解析 HTTP 请求、执行操作和准备 HTTP 响应。当相同的代码负责所有这些任务时，很难识别执行操作的语句，因为它们在所有 HTTP 处理中都被淹没。

结果往往也以 HTTP 为中心，我的意思是大多数开发者最终会编写尽可能少的路由的代码，这进一步复杂了结果。你可以在 *列表 14.14* 中看到这一点，其中使用了 Express 的 `all` 方法来匹配对 URL 路径的所有请求，HTTP 方法在请求处理器中被识别，如下所示：

```js
...
**app.all****("/api/results/:id", async (req, resp) => {**
    const id = Number.parseInt(req.params.id);
    if (**req.method** == "GET") {
        const result = await repository.getResultById(id);
        if (result == undefined) {
            resp.writeHead(404);
        } else {
            resp.json(result);
        }
    } **else if (req.method** == "DELETE") {
        let deleted = await repository.delete(id);
        resp.json({ deleted });
    }
    resp.end();
})
... 
```

我总是编写这类代码。代码可以编译，Web 服务也能工作，但维护起来很困难，因为 Web 服务的不同方面交织在一起。

如果将处理 HTTP 请求的代码提取到适配器中，Web 服务将更容易编写和维护，而且还有一个额外的好处，即需要相同一组 HTTP 方法和 URL 格式的 Web 服务可以使用相同的适配器代码。为了描述 Web 服务的功能，将名为 `http_adapter.ts` 的文件添加到 `src/server/api` 文件夹中，其中包含 *列表 14.16* 中的代码。

*列表 14.16*：src/server/api 文件夹中 http_adapter.ts 文件的内容

```js
import { Express, Response } from "express";
export interface WebService<T> {
    getOne(id: any) : Promise<T | undefined>;
    getMany(query: any) : Promise<T[]>;
    store(data: any) : Promise<T | undefined>;
    delete(id: any): Promise<boolean>;
}
export function createAdapter<T>(app: Express, ws: WebService<T>, baseUrl: string) {
    app.get(baseUrl, async (req, resp) => {
        try {
            resp.json(await ws.getMany(req.query));
            resp.end();
        } catch (err) { writeErrorResponse(err, resp) }
    });
    app.get(`${baseUrl}/:id`, async (req, resp) => {
        try {
            const data = await ws.getOne((req.params.id));
            if (data == undefined) {
                    resp.writeHead(404);
            } else {
                    resp.json(data);
            }
            resp.end();
        } catch (err) { writeErrorResponse(err, resp) }
    });
    app.post(baseUrl, async (req, resp) => {
        try {
            const data = await ws.store(req.body);
            resp.json(data);
            resp.end();
        } catch (err) { writeErrorResponse(err, resp) }
    });
    app.delete(`${baseUrl}/:id`, async (req, resp) => {
        try {
            resp.json(await ws.delete(req.params.id));
            resp.end();
        } catch (err) { writeErrorResponse(err, resp) }
    });
    const writeErrorResponse = (err: any, resp: Response) => {
        console.error(err);
        resp.writeHead(500);
        resp.end();
    }
} 
```

`WebService<T>` 接口描述了一个在类型 `T` 上操作的 Web 服务，其中包含描述支持基本 Web 服务功能的操作的方法。`createAdapter<T>` 函数创建依赖于 `WebService<T>` 方法的 Express 路由。为了为 `Result` 数据创建 `WebService<T>` 接口的实现，将名为 `results_api.ts` 的文件添加到 `src/server/api` 文件夹中，其中包含 *列表 14.17* 中显示的内容。

**注意**

我通常使用箭头函数语法来定义 JavaScript 函数，因为这对我来说感觉更自然。然而，我在 *列表 14.16* 中使用了 `function` 关键字来定义 `createAdapter<T>` 函数，因为我觉得在箭头函数上表达 TypeScript 类型参数的方式有些笨拙。等效的函数签名在箭头函数形式中是：

`export const createAdapter = <T>(app: Express, ws: WebService<T>, baseUrl: string) => {`

将类型参数放在等号之后对我来说感觉有些刺耳，尽管你可以根据你的偏好选择任何一种语法。

列表 14.17：src/server/api 文件夹中的 results_api.ts 文件的内容

```js
import { WebService } from "./http_adapter";
**import { Result } from "../data/repository";**
**import repository from "../data";**
**export class ResultWebService** **implements WebService<Result> {**
 **getOne(id: any): Promise<Result | undefined> {**
 **return repository.getResultById(Number****.parseInt(id));**
 **}**
 **getMany(query: any): Promise<Result[]> {**
 **if (query.name) {**
 **return repository.getResultsByName(query.name****, 10);**
 **} else {**
 **return repository.getAllResults(10);**
 **}**
 **}**
 **async store(data: any): Promise<Result** **| undefined> {**
 **const { name, age, years} = data;**
 **const nextage = Number.parseInt(age) + Number.parseInt(years);**
 **const id = await repository.saveResult****({ id: 0, name, age,**
 **years, nextage});**
 **return await repository.getResultById(id);** 
 **}**
 **delete(id: any): Promise<boolean> {**
 **return repository.delete****(Number.parseInt(id));**
 **}**
**}** 
```

`ResultWebService`类实现了`WebService<Result>`接口，并通过使用仓库功能来实现方法。*列表 14.18*使用新的适配器注册网络服务，替换了混合代码。

列表 14.18：在 src/server/api 文件夹中的 index.ts 文件中使用适配器

```js
import { Express } from "express";
**//import repository from "../data";**
import { createAdapter } from "./http_adapter";
import { ResultWebService } from "./results_api";
export const createApi = (app: Express) => {
    **createAdapter(app, new ResultWebService(), "/api/results");**
} 
```

网络服务的行为没有改变，但移除处理 HTTP 请求和响应的代码使得网络服务更容易理解和维护。

# 更新数据

在网络服务中支持更新的有两种方式：替换数据和修补数据。当客户端想要完全替换数据时，发送 HTTP PUT 请求，请求体包含网络服务将用于替换的所有数据。当客户端想要修改数据时，使用 HTTP `PATCH`方法，请求体包含描述如何修改该数据的内容。

使用 PUT 请求支持更新更容易实现，但需要客户端提供存储数据的完整替换。PATCH 请求更复杂，但提供了更多灵活性，并且可能更高效，因为只有更改被发送到网络服务。

**提示**

当不知道客户端将发送的更新类型时，在新的项目开始时可能很难知道采用哪种方法。我的建议是首先支持完整更新，因为它们更容易实现，只有在你发现未更改的数据值开始超过更改的值时，才转向部分更新。

本章演示了 PUT 和 PATCH 请求。为了准备，*列表 14.19*向`ApiRepository`接口添加了一个新方法，这将允许更新数据。

列表 14.19：在 src/server/data 文件夹中的 repository.ts 文件中添加方法

```js
...
export interface ApiRepository extends Repository {
    getResultById(id: number): Promise<Result | undefined>;
    delete(id: number) : Promise<boolean>;
    **update(r: Result) : Promise<Result** **| undefined>** 
}
... 
```

*列表 14.20*使用 Sequelize ORM 包实现了此方法。

列表 14.20：在 src/server/data 文件夹中的 orm_repository.ts 文件中更新数据

```js
import { Sequelize, or } from "sequelize";
import { ApiRepository, Result } from "./repository";
import { addSeedData, defineRelationships,
    fromOrmModel, initializeModels } from "./orm_helpers";
import { Calculation, Person, ResultModel } from "./orm_models";
export class OrmRepository implements ApiRepository {
    sequelize: Sequelize;
    // ...constructor and methods omitted for brevity...
  **async** **update(r: Result) : Promise<Result | undefined > {**
 **const mod = await this.sequelize.****transaction(async (transaction) => {**
 **const stored = await ResultModel.findByPk(r.id);**
 **if (stored !== null) {**
 **const [person] =** **await Person.findOrCreate({**
 **where: { name : r.name}, transaction**
 **});** 
 **const [calculation] = await Calculation.findOrCreate({**
 **where: {**
**age: r.age, years: r.years, nextage: r.nextage**
 **}, transaction**
 **});**
 **stored.personId = person.id;**
 **stored.calculationId = calculation.id;**
**return await stored.save({transaction});**
 **}**
 **});**
 **return mod ? this.getResultById(mod.id) : undefined;**
 **}**
} 
```

在示例中更新数据意味着更改与结果关联的名称或计算。`update`方法的实现通过四个步骤进行更新，所有这些步骤都作为事务执行。第一步是从数据库中读取要更新的数据，使用`Result`参数的`id`属性：

```js
...
const stored = await ResultModel.findByPk(r.id);
... 
```

如果数据库中有一个匹配的条目，使用`findOrCreate`方法来定位与`Result`参数匹配的`Person`和`Calculation`数据，或者在没有匹配的情况下创建新数据。下一步是更新`ID`s，以便存储的数据引用新的`Person`和`Calculation`记录，并将更改写入数据库，这是使用`save`方法完成的：

```js
...
stored.personId = person.id;
stored.calculationId = calculation.id;
return await stored.save({transaction});
... 
```

`save` 方法足够智能，可以检测更改，并且仅对值已更改的属性更新数据库。最后一步是在事务提交后执行，并使用 `getResultById` 方法返回修改后的数据。

## 使用 PUT 请求替换数据

PUT 请求是最容易实现的，因为 Web 服务只需使用客户端发送的数据来替换存储的数据。*列表 14.21* 扩展了描述 Web 服务的接口以添加新方法，并扩展了 HTTP 包装器以使用接口方法处理 PUT 请求。

**注意**

并非每个 Web 服务都使用 PUT 请求进行更新。POST 请求通常用于存储新数据和更新数据，使用 URL 来区分操作，因此用于更新的 URL 将包含一个唯一的 `ID` (`/api/results/1`)，而用于存储数据的 URL 则不会 (`/api/results`)。

列表 14.21：在 src/server/api 文件夹中的 http_adapter.ts 文件中添加方法

```js
import { Express, Response } from "express";
export interface WebService<T> {
    getOne(id: any) : Promise<T | undefined>;
    getMany(query: any) : Promise<T[]>;
    store(data: any) : Promise<T | undefined>;
    delete(id: any): Promise<boolean>;
  **  replace(id: any, data: any): Promise<T | undefined>;**
}
export function createAdapter<T>(app: Express, ws: WebService<T>, baseUrl: string) {
    // ...routes omitted for brevity...
 **   app.put(`${baseUrl}/:id`, async** **(req, resp) => {**
 **try {**
 **resp.json(await ws.replace(req.params.id, req.body));**
 **resp.end();**
 **} catch (err) { writeErrorResponse****(err, resp) }**
 **});**
    const writeErrorResponse = (err: any, resp: Response) => {
        console.error(err);
        resp.writeHead(500);
        resp.end();
    }
} 
```

添加到 `WebService<T>` 接口的 `replace` 方法接受一个 `id` 和一个 `data` 对象。新的路由匹配使用 PUT 方法的请求，从 URL 中提取 `ID`，并使用请求体作为数据。在 Web 服务中实现此方法的问题在于从 HTTP 包装器接收数据并将其传递给存储库，如 *列表 14.22* 所示。

列表 14.22：在 src/server/api 文件夹中的 results_api.ts 文件中替换数据

```js
import { WebService } from "./http_adapter";
import { Result } from "../data/repository";
import repository from "../data";
export class ResultWebService implements WebService<Result> {
    // ...methods omitted for brevity...
    **replace(id: any, data: any): Promise<Result | undefined> {**
 **const { name, age, years, nextage } = data;**
**return repository.update({ id, name, age, years, nextage });**
 **}**
} 
```

从 HTTP 包装器接收的数据被解构为与 `id` 参数组合的常量值，并传递给存储库的 `update` 方法。

最后一步是向命令行客户端添加一个操作，该操作将发送 PUT 请求，如 *列表 14.23* 所示。

列表 14.23：在 src/cmdline 文件夹中的 operations.mjs 文件中支持更新

```js
...
export const ops = {
    // ...properties/functions omitted for brevity...
   ** "Replace": async () => {**
 **const id = await input({ message: "ID?"});**
 **const values = {**
 **name****: await input({message: "Name?"}),**
 **age: await input({message: "Age?"}),**
 **years****: await input({message: "Years?"}),**
 **nextage: await input({message: "Next Age?"})**
 **};**
 **await** **sendRequest("PUT", `/api/results/${id}`, values);**
 **},**
    "Exit": () => process.exit()
}
... 
```

操作称为 `Replace`，它会提示存储数据所需的所有值，并使用 HTTP `PUT` 请求将它们发送到 Web 服务。从命令行选择新的 **Replace** 选项，并在提示时输入 `1`、`Joe`、`35`、`10` 和 `45`。此操作将更新 `ID` 为 `1` 的结果，并赋予新的名称，如下所示：

```js
...
? Select an operation Replace
? ID? 1
? Name? Joe
? Age? 35
? Years? 10
? Next Age? 45
{"id":1,"name":"Joe","age":35,"years":10,"nextage":45}
? Select an operation Get All
{"id":3,"name":"Alice","age":35,"years":10,"nextage":45}
{"id":2,"name":"Bob","age":35,"years":10,"nextage":45}
**{"id":1,"name":"Joe","age":35,"years":10,"nextage":45}**
... 
```

名称已更改，但其他值保持不变，因此结果与数据库中的计算之间的关系保持不变。

## 使用 PATCH 请求修改数据

PATCH 请求允许客户端请求 Web 服务器应用部分更新，而无需发送完整的数据记录。没有标准方式来描述 PATCH 请求中的部分更改，可以使用任何数据格式，只要客户端和 Web 服务都理解如何标识数据以及如何描述更改。为了支持 PATCH 请求，*列表 14.24* 向 Web 服务接口添加了一个新方法，并定义了一个匹配 PATCH 请求的路由。

列表 14.24：在 src/server/api 文件夹中的 http_adapter.ts 文件中支持 PATCH 请求

```js
import { Express, Response } from "express";
export interface WebService<T> {
    getOne(id: any) : Promise<T | undefined>;
    getMany(query: any) : Promise<T[]>;
    store(data: any) : Promise<T | undefined>;
    delete(id: any): Promise<boolean>;
    replace(id: any, data: any): Promise<T | undefined>;
    **modify****(id: any, data: any): Promise<T | undefined>;** 
}
export function createAdapter<T>(app: Express, ws: WebService<T>, baseUrl: string) {
    // ...routes omitted for brevity...
    **app.patch(`${baseUrl}/:id`, async (req, resp) => {**
 **try {**
 **resp.json(await** **ws.modify(req.params.id, req.body));**
 **resp.end();**
 **} catch (err) { writeErrorResponse(err, resp) }**
 **});**
    const writeErrorResponse = (err: any, resp: Response) => {
        console.error(err);
        resp.writeHead(500);
        resp.end();
    }
} 
```

支持部分更新的最简单方法是允许客户端提供一个仅包含替换值的 JSON 对象，并省略任何应保持不变的属性，如*列表 14.25*所示。

列表 14.25：在 src/server/api 文件夹中的 results_api.ts 文件中修改数据

```js
import { WebService } from "./http_adapter";
import { Result } from "../data/repository";
import repository from "../data";
export class ResultWebService implements WebService<Result> {
    // ...methods omitted for brevity...
    async modify(id: any, data: any): Promise<Result | undefined> {
    **    const dbData = await this.getOne(id);**
 **if (dbData !== undefined) {**
 **Object.entries(dbData).****forEach(([prop, val]) => {**
 **(dbData as any)[prop] = data[prop] ?? val;**
 **});**
 **return await this.replace(id, dbData)**
 **}**
 **}**
} 
```

实现方法枚举由`Result`接口定义的属性，并检查从请求接收到的数据是否包含替换值。新值被应用于更新现有数据，然后传递给存储库的`replace`方法进行存储。请注意，存储库在替换和更新时使用的方式相同，并且准备数据以供存储是网络服务的职责。*列表 14.26*向命令行客户端添加了一个操作，用于发送 PATCH 请求。

列表 14.26：在 src/cmdline 文件夹中的 operations.mjs 文件中发送 PATCH 请求

```js
...
export const ops = {
    // ...properties/functions omitted for brevity...
    **"Modify": async () => {**
 **const id = await** **input({ message: "ID?"});**
 **const values = {**
 **name: await input({message: "Name?"****}),**
 **age: await input({message: "Age?"}),**
 **years: await input({message: "Years?"****}),**
 **nextage: await input({message: "Next Age?"})**
 **};**
 **await sendRequest("PATCH", `/api/results/${id}`****,**
 **Object.fromEntries(Object.entries(values)**
 **.filter(([p, v]) => v !== "")));**
 **},**
    "Exit": () => process.exit()
}
... 
```

此操作以与 PUT 请求相同的方式提示值，但使用 JavaScript 的`Object.fromEntries`、`Object.entries`和`filter`函数来排除任何未提供值的属性，以便向网络服务发送部分更新。

从命令行选择新的**修改**选项，输入`2`作为`ID`，输入`Clara`作为名称，然后按*回车*键以响应其他提示。此操作将更新`ID`为`2`的结果，并赋予新的名称，如下所示：

```js
...
? Select an operation Modify
? ID? 2
? Name? Clara
? Age?
? Years?
? Next Age?
{"id":2,"name":"Clara","age":35,"years":10,"nextage":45}
? Select an operation Get All
{"id":3,"name":"Alice","age":35,"years":10,"nextage":45}
**{"id":2,"name":"Clara","age":35,"years":10,"nextage":45}**
{"id":1,"name":"Alice","age":35,"years":5,"nextage":40}
... 
```

客户端只向网络服务发送了新的名称值，并且不需要发送未更改值的属性值。

### 使用 JSON Patch

在上一节中使用的方法在客户端仅发送现有数据值更改时很有用。许多项目属于这一类别，当数据变得过于复杂，无法使用替换请求，并且只有提供更新值的变化时，这是一种有用的技术。

JSON Patch 格式([`jsonpatch.com`](https://jsonpatch.com))可用于更复杂的更新。一个 JSON Patch 文档包含一系列应用于 JSON 文档的操作。例如，用于更新`name`属性值的 JSON Patch 文档将看起来像这样：

```js
...
[{ "op": "replace", "path": "/name", "value": "Bob" }]
... 
```

JSON Patch 文档包含一个包含 JSON 对象的数组，其中`op`和`path`属性描述了要执行的操作及其目标。某些操作需要额外的属性，例如用于指定替换操作的新值的`value`属性。*表 14.5*描述了 JSON Patch 操作。

表 14.5：JSON Patch 操作

| 操作 | 描述 |
| --- | --- |

|

```js
`add` 
```

| 此操作向 JSON 文档添加一个属性，该属性由`path`和`value`属性指定的名称和值。 |
| --- |

|

```js
`remove` 
```

| 此操作从 JSON 文档中删除一个属性，该属性由`path`属性指定。 |
| --- |

|

```js
`replace` 
```

| 此操作使用`value`属性分配的值更改由`path`属性指定的属性。 |
| --- |

|

```js
`copy` 
```

| 此操作将 `from` 属性指定的属性复制到 `path` 属性指定的位置。 |
| --- |

|

```js
`move` 
```

| 此操作将 `from` 属性指定的属性移动到 `path` 属性指定的位置。 |
| --- |

|

```js
`test` 
```

| 此属性检查 JSON 文档是否包含由 `path` 和 `value` 属性指定的属性和值。如果此操作失败，则不会执行其他操作。 |
| --- |

`path` 属性用于使用 JSON Pointer 语法在 JSON 文档中标识值，该语法在 [`datatracker.ietf.org/doc/html/rfc6901`](https://datatracker.ietf.org/doc/html/rfc6901) 中描述，并且可以用于选择属性和数组元素。例如，位置 `/name` 表示 JSON 文档顶层的一个 `name` 属性。

可以使用自定义代码解析和应用 JSON Patch 文档，但使用可用的开源 JavaScript 包更容易。在 `part2app` 文件夹中运行 *列表 14.27* 中显示的命令来安装 `fast-json-patch` 包 ([`github.com/Starcounter-Jack/JSON-Patch`](https://github.com/Starcounter-Jack/JSON-Patch))，这是一个流行的 JSON Patch 包。

列表 14.27：安装 JSON Patch 包

```js
npm install fast-json-patch@3.1.1 
```

*列表 14.28* 更新了 Web 服务，使得 `modify` 方法将接收到的数据视为 JSON Patch 文档，并使用 `fast-json-patch` 包应用它。

列表 14.28：在 src/server/api 文件夹中的 result_api.ts 文件中使用 JSON Patch

```js
import { WebService } from "./http_adapter";
import { Result } from "../data/repository";
import repository from "../data";
**import * as jsonpatch from "fast-json-patch";**
export class ResultWebService implements WebService<Result> {
    // ...methods omitted for brevity...
    async modify(id: any, data: any): Promise<Result | undefined> {
        const dbData = await this.getOne(id);
        if (dbData !== undefined) {
           ** return await this.replace(id,**
 **jsonpatch.****applyPatch(dbData, data).newDocument);**
        }
    }
} 
```

使用 `applyPatch` 方法将 JSON Patch 文档处理为一个对象。`result` 对象定义了一个 `newDocument` 属性，它返回修改后的对象，该对象可以存储在数据库中。

在发送 JSON Patch 文档时，HTTP `Content-Type` 标头设置为 `application/json-patch+json`，并且此类型不是由 Express JSON 中间件组件自动解码。*列表 14.29* 配置了 JSON 中间件，以便正常 JSON 负载和 JSON Patch 负载将被解码。

列表 14.29：在 src/server 文件夹中的 server.ts 文件中启用 JSON Patch 解码

```js
...
expressApp.use(helmet());
**expressApp.use(express.json({**
 **type: ["application/json", "application/json-patch+json"]**
**}));**
registerFormMiddleware(expressApp);
registerFormRoutes(expressApp);
... 
```

JSON 中间件接受一个配置对象，其 `type` 属性可以配置为要解码的内容类型数组。最后一步是在命令行客户端中创建 JSON Patch 文档，如 *列表 14.30* 所示。

列表 14.30：在 src/cmdline 文件夹中的 operations.mjs 文件中使用 JSON Patch

```js
...
"Modify": async () => {
    const id = await input({ message: "ID?"});
    const values = {
        name: await input({message: "Name?"}),
        age: await input({message: "Age?"}),
        years: await input({message: "Years?"}),
        nextage: await input({message: "Next Age?"})
    };
   ** await sendRequest("PATCH", `/api/results/${id}`,**
 **Object.entries(values).filter((****[p, v]) => v !== "")**
 **.map(([p, v]) => ({ op: "replace", path: "/" + p, value****: v})),**
 **"application/json-patch+json");**
},
... 
```

`fast-json-patch` 包能够生成 JSON Patch 文档，但与应用它们相比，使用自定义代码创建补丁更容易，*列表 14.30* 中修改的语句为用户输入的每个值创建 `replace` 操作。

客户端和网络服务的行为方式没有变化，你可以通过从命令行选择`修改`选项，输入`2`作为`ID`，`Clara`作为名称，然后按回车键确认其他提示来验证这一点。这是本章前面执行过的相同更改，应该会产生相同的结果，如下所示：

```js
...
? Select an operation Modify
? ID? 2
? Name? Clara
? Age?
? Years?
? Next Age?
{"id":2,"name":"Clara","age":35,"years":10,"nextage":45}
? Select an operation Get All
{"id":3,"name":"Alice","age":35,"years":10,"nextage":45}
**{"id":2,"name":"Clara","age":35,"years":10,"nextage":45}**
{"id":1,"name":"Alice","age":35,"years":5,"nextage":40}
... 
```

# 验证客户端数据

网络服务无法信任从客户端接收到的数据，并且会面临与影响 HTML 表单相同类型的问题。恶意用户可以构造 HTTP 请求或修改客户端 JavaScript 代码，发送会导致错误或产生意外结果的数据值，类似于*第十一章*中描述的表单数据问题。

网络服务的困难在于以不损害通过隔离处理 HTTP 请求的语句所获得的代码清晰性的方式验证数据。如果每个网络服务方法都直接验证其数据，结果将是一堆重复的代码语句，这些语句掩盖了网络服务功能，难以阅读和理解。最佳的验证方法是描述验证要求并在网络服务外部应用它们。

## 创建验证基础设施

允许清晰地简洁地表达网络服务的验证要求需要一种基础设施，它可以隐藏那些杂乱的实现细节。

起始点是定义描述整个网络服务、网络服务方法和单个验证规则的类型。将名为`validation_types.ts`的文件添加到`src/server/api`文件夹中，内容如*列表 14.31*所示。

列表 14.31：src/server/api 文件夹中 validation_types.ts 文件的内容

```js
export interface WebServiceValidation  {
    keyValidator?: ValidationRule;
    getMany?: ValidationRequirements;
    store?: ValidationRequirements;
    replace?: ValidationRequirements;
    modify?: ValidationRequirements;
}
export type ValidationRequirements = {
    [key: string] : ValidationRule
}
export type ValidationRule =
    ((value: any) => boolean)[] |
    {
        required? : boolean,
        validation: ((value: any) => boolean)[],
        converter?: (value: any) => any,
    }
export class ValidationError implements Error {
    constructor(public name: string, public message: string) {}
    stack?: string | undefined;
    cause?: unknown;
} 
```

`WebServiceValidation`类型描述了网络服务的验证要求。`keyValidator`属性指定了用于标识数据记录的`ID`值的验证要求，使用`ValidationRule`类型。`ValidationRule`可以是应用于值的测试函数数组，或者是一个对象，它还指定了值是否必需以及一个将值转换为网络服务方法期望的类型转换器。

由`WebServiceValidation`类型定义的其他属性对应于消耗数据的网络服务方法。这些属性可以分配一个`ValidationRequirements`对象，它可以指定网络服务期望的对象形状，并为每个对象指定一个`ValidationRule`。`ValidationError`类表示在请求中验证客户端发送的数据时出现的问题。

下一步是定义将应用*列表 14.31*中使用的类型描述的要求的函数以验证数据。将名为`validation_functions.ts`的文件添加到`src/server/api`文件夹中，内容如*列表 14.32*所示。

列表 14.32：src/server/api 文件夹中 validation_functions.ts 文件的内容

```js
import { ValidationError, ValidationRequirements, ValidationRule,
    WebServiceValidation } from "./validation_types";
export type ValidationResult = [valid: boolean, value: any];
export function validate(data: any, reqs: ValidationRequirements): any {
    let validatedData: any = {};
    Object.entries(reqs).forEach(([prop, rule]) => {
        const [valid, value] = applyRule(data[prop], rule);
        if (valid) {
            validatedData[prop] = value;
        } else {
            throw new ValidationError(prop, "Validation Error");
        }
    });
    return validatedData;
}
function applyRule(val: any,
        rule: ValidationRule): ValidationResult {
    const required = Array.isArray(rule) ? true : rule.required;
    const checks = Array.isArray(rule) ? rule : rule.validation;
    const convert = Array.isArray(rule) ? (v: any) => v : rule.converter;
    if (val === null || val == undefined || val === "") {
        return [required ? false : true, val];
    }
    let valid = true;
    checks.forEach(check => {
        if (!check(val)) {
            valid = false;
        }
    });
    return [valid, convert ? convert(val) : val];
}
export function validateIdProperty<T>(val: any,
        v: WebServiceValidation) : any {
    if (v.keyValidator) {
        const [valid, value] = applyRule(val, v.keyValidator);
        if (valid) {
            return value;
        }
        throw new ValidationError("ID", "Validation Error");               
    }
    return val;
} 
```

`validate` 函数接受一个 `data` 对象和一个 `ValidationRequirements` 对象。`ValidationRequirements` 对象中指定的每个属性都会从 `data` 对象中读取并进行验证。结果是包含经过验证的 `data` 对象，这些数据可以被网络服务信任。如果数据属性不符合其验证要求，则会抛出 `ValidationError`。

为了尽可能平滑地集成验证过程，我将在 HTTP 适配器和网络服务之间插入一个验证层。验证层将接收来自适配器的请求和响应，验证数据，并将其传递给网络服务。在 `src/server/api` 文件夹中添加一个名为 `validation_adapter.ts` 的文件，其内容如 *列表 14.33* 所示。

列表 14.33：在 src/server/api 文件夹中的 validation_adapter.ts 文件内容

```js
import { WebService } from "./http_adapter";
import { validate, validateIdProperty } from "./validation_functions";
import { WebServiceValidation } from "./validation_types";
export class Validator<T> implements WebService<T> {
    constructor(private ws: WebService<T>,
        private validation: WebServiceValidation) {}
    getOne(id: any): Promise<T | undefined> {
        return this.ws.getOne(this.validateId(id));
    }
    getMany(query: any): Promise<T[]> {
        if (this.validation.getMany) {
            query = validate(query, this.validation.getMany);
        }
        return this.ws.getMany(query);
    }
    store(data: any): Promise<T | undefined> {
        if (this.validation.store) {
            data = validate(data, this.validation.store);
        }
        return this.ws.store(data);
    }
    delete(id: any): Promise<boolean> {
        return this.ws.delete(this.validateId(id));
    }
    replace(id: any, data: any): Promise<T | undefined> {
        if (this.validation.replace) {
            data = validate(data, this.validation.replace);
        }
        return this.ws.replace(this.validateId(id), data);
    }
    modify(id: any, data: any): Promise<T | undefined> {
        if (this.validation.modify) {
            data = validate(data, this.validation.modify);
        }
        return this.ws.modify(this.validateId(id), data);
    }
    validateId(val: any) {
        return validateIdProperty(val, this.validation);
    }
} 
```

使用 `validateIdProperty` 函数验证包含在 URL 中的 `ID` 值，任何额外的数据都使用 `validate` 函数进行验证。如果验证失败，将抛出 `ValidationError`。*列表 14.34* 更新了 HTTP 适配器，以捕获处理请求时抛出的异常，并为验证错误生成 `400 Bad Request` 响应，对于任何其他问题生成 `500 Internal Server Error` 响应。

列表 14.34：在 src/server/api 文件夹中的 http_adapter.ts 文件中处理验证错误

```js
import { Express, Response } from "express";
**import { ValidationError } from "./validation_types";**
export interface WebService<T> {
    getOne(id: any) : Promise<T | undefined>;
    getMany(query: any) : Promise<T[]>;
    store(data: any) : Promise<T | undefined>;
    delete(id: any): Promise<boolean>;
    replace(id: any, data: any): Promise<T | undefined>,
    modify(id: any, data: any): Promise<T | undefined>   
}
export function createAdapter<T>(app: Express, ws: WebService<T>, baseUrl: string) {
    // ...routes omitted for brevity...
    const writeErrorResponse = (err: any, resp: Response) => {
        console.error(err);
       **resp.writeHead(err** **instanceof ValidationError ? 400 : 500);**
 resp.end();
    }
} 
```

注意，结果发送给客户端时没有包含验证问题的任何细节。理论上，可以向客户端发送一个描述验证问题的 JSON 对象，但在实际操作中，客户端很少能够合理地使用此类信息。验证要求在开发过程中遇到，最好包含在开发者文档中，以便客户端在将其发送到网络服务之前验证从用户那里接收到的数据。

依赖于网络服务提供可以展示给用户的验证错误是一个有问题的过程，并且应该避免，即使客户端和网络服务是由同一团队编写的。当你发布网络服务时，你应该预期要为客户端开发者提供支持，因为他们在使用你提供的功能。

## 定义 Result API 的验证

验证的复杂性在于基础设施，它允许简洁地定义网络服务的验证要求。在 `src/server/api` 文件夹中添加一个名为 `results_api_validation.ts` 的文件，其内容如 *列表 14.35* 所示。

列表 14.35：在 src/server/api 文件夹中的 results_api_validation.ts 文件内容

```js
import { ValidationRequirements, ValidationRule,
    WebServiceValidation } from "./validation_types";
import validator from "validator";
const intValidator : ValidationRule = {
    validation: [val => validator.isInt(val)],
    converter: (val) => Number.parseInt(val)
}
const partialResultValidator: ValidationRequirements = {
    name: [(val) => !validator.isEmpty(val)],
    age: intValidator,
    years: intValidator
}
export const ResultWebServiceValidation: WebServiceValidation = {
    keyValidator: intValidator,
    store: partialResultValidator,
    replace: {
        ...partialResultValidator,
        nextage: intValidator
    }
} 
```

`ResultWebServiceValidation` 对象定义了 `keyValidator`、`store` 和 `replace` 属性，这表明网络服务需要验证其 `ID` 值以及 `store` 和 `replace` 方法使用的数据。

命名为`intValidator`的`ValidationRule`描述了整数值的验证，它使用`validation`属性和`validator`包来确保值是整数，以及一个将值解析为`number`的`converter`函数。

`intValidator`作为主验证器单独使用，并在名为`partialResultValidator`的`ValidationRequirements`对象中使用，该对象验证`store`方法所需的`name`、`age`和`years`属性。`replace`方法的验证要求通过添加`nextage`属性扩展了`store`方法使用的验证要求。

这种验证方法允许将网络服务的数据要求与这些要求的实施分开表达。*列表 14.36*将验证器包裹在服务周围。

列表 14.36：在 src/server/api 文件夹中的 index.ts 文件中应用验证

```js
import { Express } from "express";
import { createAdapter } from "./http_adapter";
import { ResultWebService } from "./results_api";
**import { Validator } from "./validation_adapter";**
**import { ResultWebServiceValidation } from "****./results_api_validation";**
export const createApi = (app: Express) => {
    **createAdapter(app, new Validator(new** **ResultWebService(),**
       ** ResultWebServiceValidation), "/api/results");**
} 
```

最后的更改很小，它利用了验证系统执行的类型转换，如*列表 14.37*所示。

列表 14.37：在 src/server/api 文件夹中的 results_api.ts 文件中依赖类型转换

```js
...
async store(data: any): Promise<Result | undefined> {
    const { name, age, years} = data;
   ** //const nextage = Number.parseInt(age) + Number.parseInt(years);**
 **const nextage = age + years;**     
    const id = await repository.saveResult({ id: 0, name, age,
        years, nextage});
    return await repository.getResultById(id);       
}
... 
```

只有当`age`和`years`按钮已转换为`number`值时，才会调用`store`方法，这意味着`store`方法不需要执行自己的转换。

要测试验证，选择命令行客户端的获取`ID`选项，并在提示时输入`ABC`。验证检查将拒绝此值，并产生一个`400 Bad Request`响应，如下所示：

```js
...
? Select an operation Get ID
? ID? ABC
400 Bad Request
... 
```

选择**存储**选项，并在提示时输入`Joe`、`30`和`Ten`。最后一个值验证失败，导致另一个`400`响应。

## 执行模型验证

并非所有验证都可以在将请求传递给将生成响应的网络服务方法之前完成。一个例子是 PATCH 请求，客户端可以发送可能导致数据库中不一致数据的部分更新，例如提供新的`years`值而没有相应的`nextage`值，这样计算结果就没有意义了。

验证此类更新必须在更新应用于现有存储数据之后才能进行，这意味着它必须通过网络服务方法来完成，这是更新过程中最早可以同时获得更改和存储数据的位置。这通常被称为*模型验证*，尽管没有一致的术语。

*列表 14.38*定义了一个新的数据类型，它将现有的验证与适用于整个数据模型对象的新规则相结合。

列表 14.38：在 src/server/api 文件夹中的 validation_types.ts 文件中添加一个类型

```js
export interface WebServiceValidation  {
    keyValidator?: ValidationRule;
    getMany?: ValidationRequirements;
    store?: ValidationRequirements;
    replace?: ValidationRequirements;
    modify?: ValidationRequirements;
}
export type ValidationRequirements = {
    [key: string] : ValidationRule
}
export type ValidationRule =
    ((value: any) => boolean)[] |
    {
        required? : boolean,
        validation: ((value: any) => boolean)[],
        converter?: (value: any) => any,
    }
export class ValidationError implements Error {
    constructor(public name: string, public message: string) {}
    stack?: string | undefined;
    cause?: unknown;
}
**export type ModelValidation = {**
 **modelRule?: ValidationRule,**
 **propertyRules?: ValidationRequirements**
**}** 
```

*列表 14.39*在一个新函数中使用`ModelValidation`类型，该函数可以在对象存储之前对其进行验证。

列表 14.39：在 src/server/api 文件夹中的 validation_functions.ts 文件中添加一个函数

```js
**import { ModelValidation, ValidationError, ValidationRequirements****,**
 **ValidationRule, WebServiceValidation } from "./validation_types";**
export type ValidationResult = [valid: boolean, value: any];
// ...functions omitted for brevity...
**export function validateModel(model: any, rules: ModelValidation) : any {**
 **if (rules.propertyRules) {**
 **model = validate(model, rules.propertyRules);**
 **}**
 **if (rules.modelRule) {**
**const [valid, data] = applyRule(model, rules.modelRule);**
 **if (valid) {**
 **return data;**
 **}**
 **throw new ValidationError("Model", "Validation Error");** 
 **}**
**}** 
```

`validateModel` 函数应用每个属性的规则，然后应用模型级别的规则。属性规则可能执行类型转换，因此属性检查的结果用作模型级验证的输入。*列表 14.40* 定义了 `Result` 对象所需的验证。

列表 14.40：在 src/server/api 文件夹中的 result_api_validation.ts 文件中定义模型验证器

```js
**import { ModelValidation, ValidationRequirements, ValidationRule,**
 **WebServiceValidation } from "****./validation_types";**
import validator from "validator";
const intValidator : ValidationRule = {
    **validation: [val => validator.isInt(val.****toString())],**
    converter: (val) => Number.parseInt(val)
}
const partialResultValidator: ValidationRequirements = {
    name: [(val) => !validator.isEmpty(val)],
    age: intValidator,
    years: intValidator
}
export const ResultWebServiceValidation: WebServiceValidation = {
    keyValidator: intValidator,
    store: partialResultValidator,
    replace: {
        ...partialResultValidator,
        nextage: intValidator
    }
}
**export const ResultModelValidation : ModelValidation = {**
 **propertyRules: { ...partialResultValidator, nextage: intValidator },**
 **modelRule****: [(m: any) => m.nextage === m.age + m.years]**
**}** 
```

`propertyRules` 属性使用为早期示例创建的验证规则。`modelRule` 属性检查 `nextage` 值是否是 `age` 和 `years` 属性的总和。

需要修改用于验证整数的规则。由 `validator` 包提供的 `isInt` 方法仅对字符串值操作，但部分更新可能将来自 HTTP 请求的 `string` 值与从数据库中读取的 `number` 值组合。为了避免异常，被检查的值始终转换为字符串。

*列表 14.41* 更新了网络服务，使其在 `replace` 和 `modify` 方法中使用模型验证功能，确保不一致的数据不会被写入数据库。

列表 14.41：在 src/server/api 文件夹中的 results_api.ts 文件中验证数据

```js
import { WebService } from "./http_adapter";
import { Result } from "../data/repository";
import repository from "../data";
import * as jsonpatch from "fast-json-patch";
**import { validateModel } from "./validation_functions";**
**import { ResultModelValidation } from "./results_api_validation"****;**
export class ResultWebService implements WebService<Result> {
    getOne(id: any): Promise<Result | undefined> {
        **return repository.getResultById(id);**
    }
    getMany(query: any): Promise<Result[]> {
        if (query.name) {
            return repository.getResultsByName(query.name, 10);
        } else {
            return repository.getAllResults(10);
        }
    }
    async store(data: any): Promise<Result | undefined> {
        const { name, age, years} = data;
        const nextage = age + years;
        const id = await repository.saveResult({ id: 0, name, age,
            years, nextage});
        return await repository.getResultById(id);       
    }
    delete(id: any): Promise<boolean> {
        return repository.delete(Number.parseInt(id));
    }
    replace(id: any, data: any): Promise<Result | undefined> {
        const { name, age, years, nextage } = data;
        **const validated = validateModel****({ name, age, years, nextage },**
 **ResultModelValidation)**
 **return repository.update({ id, ...validated });**
    }
    async modify(id: any, data: any): Promise<Result | undefined> {
        const dbData = await this.getOne(id);
        if (dbData !== undefined) {
            return await this.replace(id,
                jsonpatch.applyPatch(dbData, data).newDocument);
        }
    }
} 
```

验证可以在 `replace` 方法中执行，这允许进行替换和更新的一致性验证。

选择命令行客户端的 `Replace` 选项，并在提示时输入 `1`、`Joe`、`20`、`10` 和 `25`。这是无效数据，因为 `nextage` 的值应该是 `30`，所以验证过程失败，并产生一个 `400 Bad Request` 响应，如下所示：

```js
...
? Select an operation Replace
? ID? 1
? Name? Joe
? Age? 20
? Years? 10
? Next Age? 25
400 Bad Request
... 
```

请求和模型验证的结合确保了网络服务只接收和存储有效数据，而抽象的 HTTP 和验证功能有助于简化网络服务的实现，使其更容易理解和维护。

# 使用包进行网络服务

虽然有优秀的包可用于创建网络服务，但由于缺乏标准化，您必须找到一个适合您对网络服务如何运行的偏好的包，这可能与本章中采取的方法不同。我喜欢 Feathers 包 ([`feathersjs.com`](https://feathersjs.com))，它的工作方式与本章中的自定义代码类似，并且与流行的数据库和其他包（包括 Express）有良好的集成。

但有许多优秀的包可用，一个好的建议是搜索微服务，这已经成为一个热门术语，一些包将自己定位为微服务生态系统的组成部分。

在 `part2app` 文件夹中运行 *列表 14.42* 中显示的命令以安装 Feathers 包及其与 Express 的集成。

列表 14.42：安装包

```js
npm install @feathersjs/feathers@5.0.14
npm install @feathersjs/express@5.0.14 
```

Feathers 包包含 TypeScript 类型声明，但它们会覆盖 Express 包的声明。需要更改编译器配置以解决这个问题，如*列表 14.43*所示。

列表 14.43：在`part2app`文件夹中的`tsconfig.json`文件中更改编译器配置

```js
{
    "extends": "@tsconfig/node20/tsconfig.json",
     "compilerOptions": {                      
         "rootDir": "src/server",  
         "outDir": "dist/server/",
         **"noImplicitAny": false**
     },
     "include": ["src/server/**/*"]
} 
```

Feathers 与 Express 的集成通过扩展现有 API 来实现，该包提供的类型声明与`@types/express`包提供的不同。

## 创建一个用于 Web 服务的适配器

Feathers 包使用一系列方法描述 Web 服务，类似于本章前面自定义代码使用的接口。有一些小的差异，但两种方法足够相似，以至于一个简单的适配器可以将自定义 HTTP 处理代码替换为 Feathers 包，而无需对 Web 服务进行更改。在`src/server/api`文件夹中添加一个名为`feathers_adapter.ts`的文件，其内容如*列表 14.44*所示。

列表 14.44：`src/server/api`文件夹中`feathers_adapter.ts`文件的内容

```js
import { Id, NullableId, Params } from "@feathersjs/feathers";
import { WebService } from "./http_adapter";
export class FeathersWrapper<T> {

    constructor(private ws: WebService<T>) {}
    get(id: Id) {
        return this.ws.getOne(id);
    }
    find(params: Params) {
        return this.ws.getMany(params.query);
    }
    create(data: any, params: Params) {
        return this.ws.store(data);
    }
    remove(id: NullableId, params: Params) {
        return this.ws.delete(id);
    }  
    update(id: NullableId, data: any, params: Params) {
        return this.ws.replace(id, data);
    }
    patch(id: NullableId, data: any, params: Params) {
        return this.ws.modify(id, data);
    }
} 
```

Feathers API 提供表示`ID`值、请求体和查询参数的类型，但由于 HTTP 请求可以表示的方式有限，因此将 Feathers 包和自定义代码之间的桥梁建立起来是一个简单的过程。后续示例将直接使用 Feathers API，但这种方法展示了将现有代码适配到第三方包是多么容易。

Feathers 与 Express 的集成假设 Feathers 将扩展 Express API 以添加功能。*列表 14.45*使用 Feathers 功能创建一个 Web 服务，而无需更改应用程序的其他部分。

列表 14.45：在`src/server/api`文件夹中的`index.ts`文件中使用 Feathers

```js
import { Express } from "express";
import { createAdapter } from "./http_adapter";
import { ResultWebService } from "./results_api";
import { Validator } from "./validation_adapter";
import { ResultWebServiceValidation } from "./results_api_validation";
**import** **{ FeathersWrapper } from "./feathers_adapter";**
**import { feathers } from "@feathersjs/feathers";**
**import feathersExpress, { rest } from "@feathersjs/express";**
**import** **{ ValidationError } from "./validation_types";**
export const createApi = (app: Express) => {
    **// createAdapter(app, new Validator(new ResultWebService(),**
 **//     ResultWebServiceValidation), "/api/results");**
 **const feathersApp = feathersExpress(feathers(), app).configure(rest());**
 **const service = new Validator(****new ResultWebService(),**
 **ResultWebServiceValidation);**
 **feathersApp.use('/api/results', new FeathersWrapper(service));**
 **feathersApp.hooks({**
**error: {**
 **all: [(ctx) => {** 
 **if (ctx.error instanceof ValidationError) {**
 **ctx.http = { status:** **400};**
 **ctx.error = undefined;**
 **}**
 **}]**
 **}**
 **});**
} 
```

Express 的增强版本是通过以下声明创建的：

```js
...
const feathersApp = feathersExpress(feathers(), app).configure(rest());
... 
```

这个咒语使 Feathers 生效，并配置它以支持 RESTful 查询。Feathers 可以用不同的方式使用，而 RESTful 请求只是客户端与 Feathers 服务器端组件通信的方式之一。

Feathers 支持*钩子*，允许在请求生命周期的关键时刻执行函数。钩子是一个有用的功能，可以用于包括验证和错误处理在内的任务。在这个例子中，验证由自定义代码处理，但这个声明定义了一个在处理请求时抛出异常时将被调用的钩子：

```js
...
feathersApp.hooks({
    error: {
        all: [(ctx) => {
            if (ctx.error instanceof ValidationError) {
                ctx.http = { status: 400};
                ctx.error = undefined;
            }
        }]
    }
});
... 
```

当验证失败时，自定义代码会抛出`ValidationError`，Feathers 通过发送 500 响应来处理这种情况。钩子接收一个上下文对象，该对象提供了请求及其结果的详细信息，并且如果发生`ValidationError`，此语句会更改响应状态码。由于它使用相同的自定义代码来处理请求，因此网络服务的工作方式没有变化。但是，通过了解 RESTful 网络服务的工作方式和创建方式，转向如 Feathers 这样的包允许利用相同的功能，而无需编写自定义代码。

# 摘要

在本章中，我展示了如何使用 Node.js 提供的 HTTP 功能，并通过 Express 包增强，来创建一个 RESTful 网络服务。

+   HTTP 请求 URL 标识数据，HTTP 方法表示将要执行的操作。

+   大多数网络服务使用 JSON 格式，这已经取代了 XML 成为默认的数据格式。

+   在实现网络服务的方式上，标准化程度较低，尽管有一些广泛使用的通用约定，尤其是与 HTTP 方法所表示的操作相关。

+   在安全使用之前，网络服务接收到的数据必须经过验证。

+   通过将实现与处理 HTTP 请求和执行验证的代码分离，可以最轻松地编写网络服务。

在下一章中，我将展示如何对 HTTP 请求进行身份验证以及如何使用用户的身份进行授权。
