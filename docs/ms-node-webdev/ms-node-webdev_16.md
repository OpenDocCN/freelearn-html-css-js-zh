

# 第十五章：身份验证和授权请求

大多数项目都需要限制对功能的访问；否则，任何知道应用程序 URL 的人都可以执行任何操作。目前示例应用程序就是这样设置的：任何可以请求`http://localhost:5000`的人都将能够存储和删除数据，无论他们是谁。

*授权*，通常称为*AuthZ*，是限制访问的过程，以便只有某些用户（自然地称为*授权用户*）可以执行操作。*身份验证*，通常称为*AuthN*，是用户识别自己的过程，以便应用程序可以确定用户是否有权执行他们请求的操作。本章解释了 Node.js 应用程序如何应用身份验证和授权，基于前面章节中描述的功能。*表 15.1* 将本章置于上下文中。

表 15.1：将授权和身份验证置于上下文中

| 问题 | 答案 |
| --- | --- |
| 它们是什么？ | 身份验证是识别用户的过程。授权是将对应用程序功能的访问限制到用户子集的过程。 |
| 为什么它们有用？ | 识别用户允许应用程序通过使用特定于一个账户的数据或偏好来改变其行为。限制对功能的访问意味着应用程序可以支持那些否则可能危险或对有效服务提供有偏见的行为。 |
| 它是如何使用的？ | 用户通过向应用程序提供凭证来识别自己，应用程序生成一个临时令牌，该令牌包含在后续请求中。该令牌用于将一个身份与每个请求关联起来，可以检查以授权访问受限制的功能。 |
| 有任何陷阱或限制吗？ | 需要进行彻底的测试以确保身份验证和授权按预期工作。许多应用程序将需要额外的工作来支持用户注册和账户维护。 |
| 有没有替代方案？ | 并非所有应用程序都需要身份验证和授权，但大多数都需要。一些相关功能可以委托给第三方身份验证提供商，但仍然需要集成。 |

*表 15.2* 总结了本章内容。

表 15.2：章节总结

| 问题 | 解决方案 | 列表 |
| --- | --- | --- |
| 身份验证用户 | 提供一种机制，允许用户提供凭证，这些凭证可以与存储的数据进行验证。 | *4-9, 26-28* |
| 为 HTML 客户端创建身份验证令牌 | 在会话中包含用户的身份，以便会话 cookie 成为身份验证令牌。 | *10-12* |
| 为 API 客户端创建身份验证令牌 | 创建一个携带令牌。 | *13-16* |
| 授权请求 | 使用与请求关联的标识来确定用户是否有权执行目标操作。 | *17-25, 29* |

# 为本章做准备

本章使用*第十四章*中的`part2app`项目。本章的第一个示例是为往返应用程序准备的。为了准备，将名为`data.handlebars`的文件添加到`templates/server`文件夹中，其内容如*列表 15.1*所示：

**提示**

您可以从[`github.com/PacktPublishing/Mastering-Node.js-Web-Development`](https://github.com/PacktPublishing/Mastering-Node.js-Web-Development)下载本章的示例项目——以及本书中所有其他章节的示例项目。有关运行示例时遇到问题的帮助，请参阅*第一章*。

列表 15.1：`templates/server`文件夹中`data.handlebars`文件的内容

```js
<form class="m-2">
    <table class="table table-sm table-striped">
        <thead>
            <tr>
                <th>ID</th><th>Name</th><th>Age</th><th>Years</th>
                <th>Next Age</th><th></th>
            </tr>
        </thead>
        <tbody>
            {{#unless data }}<tr><td colspan="5">No Data</td></tr>{{/unless }}
            {{#each data }}
                <tr>
                    <td>{{ this.id }} </td>
                    <td>{{ this.name }} </td>
                    <td>{{ this.age }} </td>
                    <td>{{ this.years }} </td>
                    <td>{{ this.nextage }} </td>
                    <td>
                        <button class="btn btn-danger btn-sm"
                            formmethod="post"
                            formaction="/form/delete/{{this.id}}">
                                Delete
                        </button> 
                    </td>
                </tr>               
            {{/each }}
        </tbody>
    </table>
    <button class="btn btn-primary"
        formmethod="post"
        formaction="/form/add">
            Add
    </button>                    
    <input type="hidden" name="name" value="Alice" />
    <input type="hidden" name="age" value="40" />
    <input type="hidden" name="years" value="10" />
</form> 
```

此模板包含一个显示数据的表格，以及一个向服务器发送 HTTP 请求的表单。为了处理 HTTP 请求，将`src/server`文件夹中`forms.ts`文件的内容替换为*列表 15.2*中显示的代码。

列表 15.2：`src/server`文件夹中`forms.ts`文件的内容

```js
import express, { Express } from "express";
import repository  from "./data";
import cookieMiddleware from "cookie-parser";
import { sessionMiddleware } from "./sessions/session_helpers";
import { Result } from "./data/repository";
const rowLimit = 10;
export const registerFormMiddleware = (app: Express) => {
    app.use(express.urlencoded({extended: true}))
    app.use(cookieMiddleware("mysecret"));
    app.use(sessionMiddleware());
}
export const registerFormRoutes = (app: Express) => {
    app.get("/form", async (req, resp) => {
        resp.render("data", {data: await repository.getAllResults(rowLimit)});
    });
    app.post("/form/delete/:id", async (req, resp) => {
        const id = Number.parseInt(req.params["id"]);
        await repository.delete(id);
        resp.redirect("/form");
        resp.end();
    });

    app.post("/form/add", async (req, resp) => {
        const nextage = Number.parseInt(req.body["age"])
            + Number.parseInt(req.body["years"]);
        await repository.saveResult({...req.body, nextage } as Result);
        resp.redirect("/form");
        resp.end();
    });
} 
```

*列表 15.2*中定义的路由渲染数据模板，从数据库中删除一个条目，并存储一个条目。数据存储或删除后，浏览器会收到重定向到`/form` URL 的请求，该 URL 将显示用户操作的结果。在`part2app`文件夹中运行*列表 15.3*中显示的命令以启动开发工具：

列表 15.3：启动开发工具

```js
npm start 
```

使用浏览器请求`http://localhost:5000/form`，您将看到由新模板生成的内容，如图 15.1 所示。点击**删除**按钮将从一个数据库中删除一个条目，点击**添加**按钮将使用固定数据值存储一个新的条目。

![图片](img/B21959_15_01.png)

图 15.1：运行示例应用程序

# 理解端到端过程

本章涵盖的主题是用户获取应用程序提供功能的一个更大过程的一部分。该过程如下：

1.  注册。注册过程为用户创建一个账户，并给予凭证以供用户识别自己。

1.  用户认证。当用户想要使用应用程序时，会展示他们的凭证。认证过程，通常称为登录，生成一个临时令牌以识别用户。

1.  请求认证。在发起 HTTP 请求时，客户端包含临时令牌以识别用户，而无需再次提供凭证。

1.  授权。请求中包含的令牌用于确定用户是否可以访问请求中指定的功能。

本章涵盖了认证和授权的过程部分。注册过程的细节没有描述，因为它们取决于应用程序的类型。对于企业应用程序，注册通常发生在新员工加入公司时，对于大型公司，将由人力资源部门通过中央员工目录来完成。对于面向消费者的应用程序，注册通常与支付相关联，在用户获得应用程序访问权之前（例如，Spotify 服务）或在他们做出产品选择之后（例如，亚马逊商品）进行。在两种情况下，用户都会自行注册。

需要用户自行注册的应用程序通常提供账户维护工具，允许用户更改他们的凭证、更新个人信息以及关闭他们的账户。在某些地区的法律中，用户有权利获得所有关于他们的数据的副本，这通常是账户管理过程的一部分。

# 用户认证

认证过程要求用户向应用程序出示他们的凭证以证明自己的身份。标准凭证是用户名和密码。密码只有用户知道，这意味着他们可以通过提交正确的密码来证明他们是账户的所有者。

当然，密码可能会被盗用或共享，因此，一个常见的做法是要求额外的身份证明。传统的方法是将密码与一个物理令牌结合，这个令牌可以是专用的硬件设备，也可以是运行在手机上的认证应用。该设备提供一段有时间限制的代码，以证明用户拥有该设备。

为了详细说明用户认证的过程，我将在示例应用中添加对用户名和密码的支持。在本章的后面部分，我将介绍一个支持更广泛凭证的开源包。但简单的密码足以解释整体认证和授权过程是如何工作的。本书的*第三部分*展示了第三方认证服务的使用。

## 创建凭证存储库

为了认证用户，应用程序需要有一个凭证存储库，以便可以验证请求。创建`src/server/auth`文件夹，并向其中添加一个名为`auth_types.ts`的文件，其内容如*清单 15.4*所示：

清单 15.4：`src/server/auth`文件夹中`auth_types.ts`文件的内容

```js
export interface Credentials {
    username: string;
    hashedPassword: Buffer;
    salt: Buffer;
}
export interface AuthStore {
    getUser(name: string) : Promise<Credentials | null>;
    storeOrUpdateUser(username: string, password: string):
        Promise<Credentials>;
    validateCredentials(username: string, password: string): Promise<boolean>
} 
```

`Credentials` 接口描述了用于验证的用户凭证。将密码以明文形式存储是不好的做法，传统的方法是创建一个单向哈希码并将其存储。为了验证凭证，将用户提供的密码通过相同的哈希算法处理，并与存储的值进行比较。哈希算法总是产生相同的结果，这意味着凭证存储将包含所有选择相同密码的用户相同的哈希码。如果获取了这些账户中的任何一个账户的密码，那么任何可以查看凭证存储的人都能推断出哪些其他账户可以访问。

为了避免这个问题，密码中添加了一个随机的 *盐* 值，这样用户即使使用相同的密码也不会在凭证存储中产生重复的哈希码。盐值必须与密码一起存储，以便验证凭证。哈希码和盐值是 `Buffer` 类型的值，这是 Node.js 用于表示字节数组的类型。`AuthStore` 接口定义了用于检索和存储凭证的方法。

在 `src/server/auth` 文件夹中添加一个名为 `orm_auth_models.ts` 的文件，其内容如 *列表 15.5* 所示，该文件使用在 *第十二章* 中介绍的 *Sequelize ORM 包* 定义了凭证的数据模型：

列表 15.5：src/server/auth 文件夹中 orm_auth_models.ts 文件的内容

```js
import { DataTypes, InferAttributes, InferCreationAttributes, Model,
    Sequelize } from "sequelize";
import { Credentials } from "./auth_types";
export class CredentialsModel
        extends Model<InferAttributes<CredentialsModel>,
            InferCreationAttributes<CredentialsModel>>
        implements Credentials {
    declare username: string;
    declare hashedPassword: Buffer;
    declare salt: Buffer;
}
export const initializeAuthModels = (sequelize: Sequelize) => {
    CredentialsModel.init({
        username: { type: DataTypes.STRING, primaryKey: true },
        hashedPassword: { type: DataTypes.BLOB },
        salt: { type: DataTypes.BLOB }
    }, { sequelize });
} 
```

`CredentialsModel` 类继承自 Sequelize 的 `Model` 类并实现了 `Credentials` 接口，这使得 `CredentialsModel` 对象可以存储在数据库中，并可以作为带有 `AuthStore` 接口的方法结果使用。`initializeAuthModels` 函数接收一个 `Sequelize` 对象，并为数据库存储初始化 `CredentialsModel`，将 `username` 属性标识为主键，并告诉 Sequelize 使用 SQL 的 `STRING` 数据类型表示用户名属性，使用 `BLOB` 类型表示哈希码和盐值（`BLOB` 类型允许数据以字符串或缓冲区形式存储）。

要创建 `AuthStore` 接口的 Sequelize 实现，在 `src/server/auth` 文件夹中添加一个名为 `orm_authstore.ts` 的文件，其内容如 *列表 15.6* 所示。

列表 15.6：src/server/auth 文件夹中 orm_authstore.ts 文件的内容

```js
import { Sequelize } from "sequelize";
import { CredentialsModel, initializeAuthModels }
    from "./orm_auth_models";
import { AuthStore } from "./auth_types"
import { pbkdf2, randomBytes, timingSafeEqual } from "crypto";
export class OrmAuthStore implements AuthStore {
    sequelize: Sequelize;
    constructor() {
        this.sequelize = new Sequelize({
            dialect: "sqlite",
            storage: "orm_auth.db",
            logging: console.log,
            logQueryParameters: true
        });
        this.initModelAndDatabase();
    }
    async initModelAndDatabase() : Promise<void> {
        initializeAuthModels(this.sequelize);
        await this.sequelize.drop();       
        await this.sequelize.sync();       
        await this.storeOrUpdateUser("alice", "mysecret");
        await this.storeOrUpdateUser("bob", "mysecret");       
    }
    async getUser(name: string) {
        return await CredentialsModel.findByPk(name);
    }
    async storeOrUpdateUser(username: string, password: string) {
        const salt = randomBytes(16);
        const hashedPassword = await this.createHashCode(password, salt);
        const [model] = await CredentialsModel.upsert({
            username, hashedPassword, salt
        });
        return model;
    }
    async validateCredentials(username: string, password: string):
            Promise<boolean> {
        const storedCreds = await this.getUser(username);
        if (storedCreds) {
            const candidateHash =
                await this.createHashCode(password, storedCreds.salt);
            return timingSafeEqual(candidateHash, storedCreds.hashedPassword);
        }
        return false;
    }
    private createHashCode(password: string, salt: Buffer) : Promise<Buffer> {
        return new Promise((resolve, reject) => {
            pbkdf2(password, salt, 100000, 64, "sha512", (err, hash) => {
                if (err) {
                    reject(err)
                };
                resolve(hash);
            })      
        })
    }
} 
```

`OrmAuthStore` 类使用 `CredentialsModel` 类提供的 Sequelize 特性实现了 `AuthStore` 接口。`getUser` 方法通过 `findByPk` 方法实现，该方法使用主键值查询数据库。`storeOrUpdateUser` 方法通过 `upsert` 方法实现，如果存在现有值则更新该值，否则创建新值。数据将存储在名为 `orm_auth.db` 的 SQLite 数据库文件中。

`createHashCode`方法接受一个密码和一个盐值，并使用 Node.js `crypto`模块中的`pbkdf2`函数创建一个新的哈希码。这个函数是**基于密码的密钥派生函数**（**PBKDF**）的一个实现，它非常适合从密码中创建哈希码（有关详细信息，请参阅[`en.wikipedia.org/wiki/PBKDF2`](https://en.wikipedia.org/wiki/PBKDF2)）。`pbkdf2`函数的参数是要哈希的密码、盐值、用于生成哈希码的迭代次数、哈希码的长度以及将用于生成哈希码的算法。

*列表 15.6*使用了 Node.js API 中描述的值（[`nodejs.org/docs/latest/api/crypto.html#cryptopbkdf2password-salt-iterations-keylen-digest-callback`](https://nodejs.org/docs/latest/api/crypto.html#cryptopbkdf2password-salt-iterations-keylen-digest-callback)）。`pbkdf2`函数使用回调，该回调被 Promise 包装，以便更容易与`Sequelize` API 一起使用。

`validateCredentials`方法使用`getUser`方法检索存储的凭证，并使用存储的盐值计算候选密码的新哈希码，然后使用 Node.js `crypto`模块中的`timingSafeEqual`函数将该哈希码与存储的哈希码进行比较。该函数用于安全地比较哈希码，如 API 文档中所述（[`nodejs.org/docs/latest/api/crypto.html#cryptotimingsafeequala-b`](https://nodejs.org/docs/latest/api/crypto.html#cryptotimingsafeequala-b)）。

数据库中预置了两套凭证，与*表 15.3*中描述的相匹配。凭证通常在注册过程中创建，如本章前面所述，但本章节中测试凭证就足够了。有关在线商店典型注册过程的示例，请参阅*第三部分*。与前面章节中的示例一样，每次启动应用程序时都会重置数据库。

表 15.3：添加到数据库中的测试凭证

| 名称 | 密码 |
| --- | --- |

|

```js
`alice` 
```

|

```js
`mysecret` 
```

|

|

```js
`bob` 
```

|

```js
`mysecret` 
```

|

## 创建认证工作流程

下一步是创建一个工作流程，允许用户登录和注销应用程序。将名为`signin.handlebars`的文件添加到`templates/server`文件夹中，其内容如*列表 15.7*所示：

列表 15.7：templates/server 文件夹中 signin.handlebars 文件的内容

```js
{{#if failed }}
    <h4 class="bg-danger text-white p-2 text-center">
        Authentication failed. Please try again.
    </h4>
{{/if}}
<form method="post">
    <div class="m-2">
        <label class="form-label">Name</label>
        <input name="username" class="form-control" value="{{ username }}"/>
    </div>
    <div class="m-2">
        <label class="form-label">Password</label>
        <input name="password" type="password" class="form-control"
            value="{{ password }}"/>
    </div>   
    <button type="submit" class="btn btn-primary mx-2">Sign In</button>
</form> 
```

模板包含一个表单，用于将用户名和密码发送到应用程序，并包含一个默认隐藏的错误消息，如果用户提供了无效的凭证，则该消息将被显示。

下一步是创建 Express 路由，这些路由将向用户展示`signin`模板的内容，并在提交凭证时验证凭证。将名为`index.ts`的文件添加到`src/server/auth`文件夹中，其内容如*列表 15.8*所示：

列表 15.8：src/server/auth 文件夹中 index.ts 文件的内容

```js
import { Express } from "express"
import { AuthStore } from "./auth_types";
import { OrmAuthStore } from "./orm_authstore";
const store: AuthStore = new OrmAuthStore();
export const createAuth = (app: Express) => {
    app.get("/signin", (req, resp) => {
        const data = {
            username: req.query["username"],
            password: req.query["password"],
            failed: req.query["failed"] ? true : false
        }
        resp.render("signin", data);
    });
    app.post("/signin", async (req, resp) => {
        const username = req.body.username;
        const password = req.body.password;
        const valid = await store.validateCredentials(username, password);
        if (valid) {
            resp.redirect("/");
        } else {
            resp.redirect(
                `/signin?username=${username}&password=${password}&failed=1`);
        }
    });
} 
```

此文件导出一个名为 `createAuth` 的函数，该函数为应用程序设置身份验证。当向 `/signin` 发送 GET 请求时，将渲染 `signin` 模板的内 容。当向 `/signin` 发送 POST 请求时，将验证其中包含的凭据。如果凭据有效，将使用重定向将用户送回应用程序。

当凭据验证失败时，也会发送重定向，但这次是到相同的 URL。这是一种称为 `Post/Redirect/Get` 的模式，它确保用户可以重新加载浏览器而不会触发另一个登录尝试。这种模式可以用于任何表单，但在身份验证中特别有用，因为重复的失败尝试经常被计数，可能导致账户被注销。URL 查询字符串用于包含用户提供的凭据，以便它们可以通过 GET 请求的结果显示。

**注意**

身份验证应始终在加密的 HTTP 连接上执行；否则，用户提供的凭据可能会暴露给网络嗅探。有关为独立 Node.js 服务器设置 HTTPS 的详细信息，请参阅*第五章*，有关如何为更复杂的 Node.js 应用程序设置 HTTPS 的示例，请参阅*第三部分*。

*列表 15.9* 在服务器启动过程中调用 `createAuth` 函数，以便身份验证功能成为请求处理过程的一部分：

列表 15.9：在 src/server 文件夹中的 server.ts 文件中启用身份验证

```js
import { createServer } from "http";
import express, {Express } from "express";
import httpProxy from "http-proxy";
import helmet from "helmet";
import { engine } from "express-handlebars";
import { registerFormMiddleware, registerFormRoutes } from "./forms";
import { createApi } from "./api";
**import { createAuth } from "./auth";**
const port = 5000;
const expressApp: Express = express();
const proxy = httpProxy.createProxyServer({
    target: "http://localhost:5100", ws: true
});
expressApp.set("views", "templates/server");
expressApp.engine("handlebars", engine());
expressApp.set("view engine", "handlebars");
expressApp.use(helmet());
expressApp.use(express.json({
    type: ["application/json", "application/json-patch+json"]
}));
registerFormMiddleware(expressApp);
**createAuth(expressApp);**
registerFormRoutes(expressApp);
createApi(expressApp);
expressApp.use("^/$", (req, resp) => resp.redirect("/form"));
expressApp.use(express.static("static"));
expressApp.use(express.static("node_modules/bootstrap/dist"));
expressApp.use((req, resp) => proxy.web(req, resp));
const server = createServer(expressApp);
server.on('upgrade', (req, socket, head) => proxy.ws(req, socket, head));
server.listen(port,
    () => console.log(`HTTP Server listening on port ${port}`)); 
```

在设置表单所需的中间件组件之后，但在应用程序的其余部分之前调用 `createAuth` 方法。这允许身份验证请求处理程序依赖于之前描述的表单功能，例如解码表单数据和使用会话。

要测试身份验证工作流程，请使用网络浏览器请求 `http://localhost:5000/signin`，在表单中输入 `alice` 和 `badpass`，然后点击 **登录** 按钮。验证将失败，因为提供了错误的密码。将密码更改为 `mysecret` 并再次点击按钮。这次，凭据将被验证，浏览器将被重定向到根 URL。此序列在*图 15.2*中显示。

![](img/B21959_15_02.png)

图 15.2：登录工作流程

# 验证请求

应用程序可以验证凭据，但这并没有太大用处，因为目前发送到 `/signin` URL 的凭据与浏览器发出的任何后续 HTTP 请求之间没有关联。

这就是可以展示给应用程序的临时令牌的目的，以证明用户已经通过了凭据验证过程。Cookies 是解决这个问题的最常见方式，要么创建一个单独的 cookie，要么将认证数据与现有的会话 cookie 关联起来，这是我在本章中将采取的方法，因为它是最简单的方法，并利用了会话功能，如自动不活动过期。*列表 15.10* 使用会话记录成功的认证，并定义了一个检测新会话数据并将 `user` 属性添加到请求对象的中间件：

列表 15.10：在 src/server/auth 文件夹中的 index.ts 文件中完成认证

```js
import { Express } from "express"
import { AuthStore } from "./auth_types";
import { OrmAuthStore } from "./orm_authstore";
const store: AuthStore = new OrmAuthStore();
**type** **User = { username: string }**
**declare module "express-session" {**
 **interface SessionData { username: string; }**
**}**
**declare global {**
 **module** **Express {**
 **interface Request { user: User, authenticated: boolean }**
 **}**
**}**
export const createAuth = (app: Express) => {
    **app.****use((req, resp, next) => {**
 **const username = req.session.username;**
 **if (username) {**
 **req.authenticated = true;**
 **req.user = { username };**
 **}** **else {**
 **req.authenticated = false;**
 **}**
 **next();**
 **});**
    app.get("/signin", (req, resp) => {
        const data = {
            username: req.query["username"],
            password: req.query["password"],
            failed: req.query["failed"] ? true : false
        }
        resp.render("signin", data);
    });
    app.post("/signin", async (req, resp) => {
        const username = req.body.username;
        const password = req.body.password;
        const valid = await store.validateCredentials(username, password);
        if (valid) {
           ** req.session.username = username;**
            resp.redirect("/");
        } else {
            resp.redirect(
                `/signin?username=${username}&password=${password}&failed=1`);
        }
    });
    **app.post("/signout", async (req, resp) => {**
 **req.session.destroy(() => {**
 **resp.****redirect("/");**
 **})**
 **});**
} 
```

第一个 `declare` 语句扩展了 `SessionData` 接口以定义一个 `username` 属性，以便将用户的身份与一个会话关联起来。虽然可能会诱使将更复杂的数据放入会话中，但这个新属性的目的仅仅是用来识别用户，这可以通过仅仅向 `SessionData` 接口添加一个 `string` 属性来实现。第二个 `declare` 语句向 Express 的 `Request` 接口添加了 `user` 和 `authenticated` 属性，这将允许向应用程序的其余部分提供更复杂用户数据。

当用户的凭据被验证后，添加到 `SessionData` 接口的 `username` 属性被用来存储用户名：

```js
...
req.session.**username** = username;
... 
```

新的中间件组件检查请求的会话数据，以查看是否设置了此属性。如果设置了，那么 `Request` 对象的 `username` 和 `authenticated` 属性将被设置，这就是应用程序的其余部分将如何识别已认证用户的方式。

最后的添加是一个新的 `/signout` URL 路由，它允许用户通过销毁会话来注销应用程序，通过调用 `destroy` 方法，这是在 *第十三章* 中添加到项目中的 `express-session` 包提供的一个功能。

这意味着会话 cookie 已经被转换成了用于验证用户请求的临时令牌。当浏览器在请求中包含会话 cookie 时，应用程序知道这个请求是代表用户发送的，因为会话 cookie 代表了用户凭据验证的成功。

## 使用认证数据

要完成身份验证功能，用户必须能够看到他们已成功登录并允许再次注销。Express 有一个用于处理模板的有用功能，称为 *本地数据* 或 *locals*，它允许在调用 `render` 方法之外向模板提供数据。本地数据特定于单个请求/响应对，任何分配给 `Response.locals` 属性的值都可以在任何模板中使用。这对于向模板提供身份验证信息非常完美，否则必须在每次调用 `render` 方法时将其添加到上下文数据中。*列表 15.11* 使用此功能向模板提供身份验证信息：

列表 15.11：在 src/server/auth 文件夹中的 index.ts 文件中提供身份验证详细信息

```js
...
app.use((req, resp, next) => {
    const username = req.session.username;
    if (username) {
        req.authenticated = true;
        req.user = { username };
    } else {
        req.authenticated = false;
    }
    **resp.locals.user = req.user;**
 **resp.locals.authenticated = req.authenticated;** 
    next();
});
app.get("/signin", (req, resp) => {
    const data = {
        username: req.query["username"],
        password: req.query["password"],
        failed: req.query["failed"] ? true : false,
       ** signinpage: true**
    }
    resp.render("signin", data);
});
... 
```

添加到中间件组件中的新语句创建名为 `user` 和 `authenticated` 的本地数据值，这意味着这些信息将可用于任何由经过此中间件处理的请求/响应执行的自定义模板。还有一个名为 `signinpage` 的常规上下文数据属性，当登录表单呈现给用户时，它会被传递给 `render` 方法。

*列表 15.12* 更新了与所有模板一起使用的布局，这使得可以在整个应用程序中显示身份验证信息：

列表 15.12：在模板/server/layouts 文件夹中的 main.handlebars 文件中使用身份验证数据

```js
<!DOCTYPE html>
<html>
    <head>
        <script src="img/bundle.js"></script>       
        <link href="css/bootstrap.min.css" rel="stylesheet" />
    </head>
    <body>
      **  {{#if authenticated }}**
 **<div class="bg-primary text-white p-1 clearfix">**
 **<form method="post" action="/signout">**
 **<span class="h5">****User: {{ user.username }}</span>**
 **<button class="btn btn-secondary btn-sm float-end"**
 **type="submit">Sign Out</button>**
 **</form>**
 **</div>**
 **{{else }}**
 **{{#unless signinpage }}**
 **<div class="bg-primary text-white p-1 clearfix">**
 **<a** **href="/signin"**
 **class="btn btn-secondary btn-sm float-end">Sign In</a>**
 **</div>**
 **{{/unless }}**
 **{{/if}}**
        {{{ body }}}
    </body>
</html> 
```

本地数据值的使用方式与常规模板上下文数据相同，如果请求已验证，模板将显示用户名和注销按钮。如果请求未验证，则模板将显示登录按钮，除非设置了 `signinpage` 属性，在这种情况下，不会显示新内容。

要查看更改的效果，请使用浏览器导航到 `http://localhost:5000/signin`，分别在 `Name` 和 `Password` 字段中输入 `alice` 和 `mysecret`，然后点击 **Sign In** 按钮。凭据将被验证，浏览器将显示用户名和一个 **Sign Out** 按钮。点击 **Sign Out** 按钮，将显示一个 **Sign In** 按钮。此序列在 *图 15.3* 中显示。

![图片 B21959_15_03.png](img/B21959_15_03.png)

图 15.3：使用身份验证数据

# 验证 Web 服务请求

应用程序不能依赖于表单来验证 Web 服务，因为客户端可能不是浏览器，并且不能依赖于渲染 HTML。

Web 服务客户端可以使用 cookies – 因为它们是 HTTP 的标准部分 – 但会话 cookies 通常会引起问题，因为会话过期通常设置为适合往返客户端，其中每个用户交互都会刷新 cookie。Web 服务客户端仅在需要数据时发送请求，请求的频率可能很低，以至于会话过期得太快，无法使用。

应用程序可以通过提供一个 API 来呈现凭据作为 JSON 数据来弥补 HTML 支持的不足。与 cookie 不同，认证 API 生成一个 *bearer 令牌*，这是一个可以包含在请求中的字符串，就像 cookie 一样，但它有自己的生命周期，且不依赖于会话。

最常见的 bearer 令牌形式是 **JSON Web Token**（JWT）标准，它是一个自包含的认证令牌，不依赖于服务器端数据。（有关 JWT 的良好概述请参阅 [`jwt.io`](https://jwt.io)，以及用于验证令牌的工具，这在开发期间可能很有用）。

就像几乎网络服务的各个方面一样，客户端如何执行认证没有硬性标准，但我将遵循广泛使用的约定。为了登录，客户端将向 `/api/signin` URL 发送 `HTTP POST` 请求，其中包含一个 JSON 有效负载，包含用户的凭据，如下所示：

```js
...
{
    "username": "alice",
    "password": "mysecret"
}
... 
```

结果将包含一个包含成功属性的 JSON 对象，该属性指示凭据是否被接受，如果是，则包含一个 `token` 属性，其中包含 bearer 令牌，如下所示：

```js
...
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9"
}
... 
```

实际的 JWT 令牌是更长的字符序列，但我为了简洁起见缩短了这一示例。客户端无需以任何方式解析或处理令牌，只需在 HTTP 请求中使用 `Authorization` 标头包含令牌即可，如下所示：

```js
...
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
... 
```

`Authorization` 标头的值是一个方案，它是 `Bearer`，后面跟着认证过程中生成的令牌。服务器解码令牌并使用它来确定已认证用户的身份。

**理解 API 密钥**

本章中的示例主要关注用户认证。网络服务还可以使用 API 密钥，这些密钥标识代表用户发出请求的客户端，当第三方创建消费您项目 API 的客户端时可能很有用。API 密钥通常具有较长的生命周期，用于控制 API 功能的访问、跟踪请求数量等。本书中未描述 API 密钥，但有一个良好的概述可在 [`cloud.google.com/endpoints/docs/openapi/when-why-api-key`](https://cloud.google.com/endpoints/docs/openapi/when-why-api-key) 找到。

## 创建认证 API

在 `part2app` 文件夹中运行 *清单 15.13* 命令，以将 JWT 包及其类型描述添加到示例项目中：

清单 15.13：安装包

```js
npm install jsonwebtoken@9.0.2
npm install --save-dev @types/jsonwebtoken@9.0.5 
```

使用自定义代码生成和验证 JWT 值是可能的，但使用一个好的包更简单、更容易。*清单 15.14* 为登录 API 客户端添加了支持：

清单 15.14：在 src/server/auth 文件夹中的 index.ts 文件中登录 API 客户端

```js
import { Express } from "express"
import { AuthStore } from "./auth_types";
import { OrmAuthStore } from "./orm_authstore";
**import jwt from "jsonwebtoken";**
**const jwt_secret = "mytokensecret";**
const store: AuthStore = new OrmAuthStore();
type User = { username: string }
declare module "express-session" {
    interface SessionData { username: string; }
}
declare global {
    module Express {
        interface Request { user: User, authenticated: boolean }
    }
}
export const createAuth = (app: Express) => {
    app.use((req, resp, next) => {
        const username = req.session.username;
        if (username) {
            req.authenticated = true;
            req.user = { username };
     **   } else if (req.headers.authorization) {**
 **let token = req.headers.authorization;**
 **if (token.****startsWith("Bearer ")) {**
 **token = token.substring(7);**
 **}**
 **try {**
 **const decoded = jwt.verify(token, jwt_secret) as User;**
 **req.authenticated =** **true;**
 **req.user = { username: decoded.username };**
 **} catch {**
 **// do nothing - cannot verify token**
 **}**
        } else {
            req.authenticated = false;
        }
        resp.locals.user = req.user;
        resp.locals.authenticated = req.authenticated;   
        next();
    });
    app.get("/signin", (req, resp) => {
        const data = {
            username: req.query["username"],
            password: req.query["password"],
            failed: req.query["failed"] ? true : false,
            signinpage: true
        }
        resp.render("signin", data);
    });
    app.post("/signin", async (req, resp) => {
        const username = req.body.username;
        const password = req.body.password;
        const valid = await store.validateCredentials(username, password);
        if (valid) {
            req.session.username = username;
            resp.redirect("/");
        } else {
            resp.redirect(
                `/signin?username=${username}&password=${password}&failed=1`);
        }
    });
 **app.post("/api/signin",** **async (req, resp) => {**
 **const username = req.body.username;**
 **const password = req.body.password;**
 **const result: any = {**
 **success:** **await store.validateCredentials(username, password)**
 **}**
 **if (result.success) {**
 **result.token = jwt.sign({username} , jwt_secret,**
 **{ expiresIn: "1hr"});**
 **}**
 **resp.json(result);**
 **resp.end();** 
 **});**
    app.post("/signout", async (req, resp) => {
        req.session.destroy(() => {
            resp.redirect("/");
        })
    });
} 
```

`/api/signin` 路由依赖于 Express JSON 中间件来解析客户端发送的数据，并验证用户的凭据。如果凭据有效，则创建一个令牌，如下所示：

```js
...
result.token = jwt.sign({username} , jwt_secret, { expiresIn: "1hr"});
... 
```

`sign`函数创建一个令牌，该令牌经过签名以防止篡改。参数是作为令牌有效负载使用的数据，用于签名字符串的密钥（在验证期间必须再次使用），以及用于指定令牌过期的配置对象。

`jsonwebtoken`包支持使用`ms`包定义的语法设置过期时间（[`github.com/vercel/ms`](https://github.com/vercel/ms)）。这允许将`expiresIn`属性设置为`1h`，创建一个有效期为 60 分钟的令牌。

**注意**

您可以在令牌中放入任何由生成它的同一应用程序消耗的数据。如果您正在生成将由第三方验证的令牌，那么有一些用于描述身份验证和授权数据的定义良好的有效负载属性，可以在[`jwt.io/introduction`](https://jwt.io/introduction)找到。令牌是经过签名的，但不是加密的，这意味着不应在令牌中包含敏感数据。

身份验证中间件检查请求是否包含`Authorization`头，如果包含，则验证其值作为令牌。验证检查签名以确保有效负载未被更改，并确保令牌未过期。用户名从令牌的有效负载中读取，并用于验证请求。

## 验证 Web 服务客户端

为了完成身份验证实现，*清单 15.15*更新了命令行客户端，添加了登录和注销应用程序的操作：

列表 15.15：在 src/cmdline 文件夹中的 operations.mjs 文件中添加身份验证

```js
import { input } from "@inquirer/prompts";
const baseUrl = "http://localhost:5000";
**let bearer_token;**
export const ops = {
  **  "Sign In": async () => {**
 **const creds = {**
 **username: await** **input({message: "Username?"}),**
 **password: await input({message: "Password?"}),**
 **};**
 **const response = await** **sendRequest("POST", "/api/signin", creds);**
 **if (response.success == true) {**
 **bearer_token = response.token;**
 **};**
 **},**
 **"Sign Out": () => { bearer_token = undefined** **},**
    "Get All": () => sendRequest("GET", "/api/results"),
    // ... other operations omitted for brevity...
}
const sendRequest = async (method, url, body, contentType) => {
   ** const headers = { "Content-Type": contentType ?? "application/json"};**
 **if (bearer_token) {**
 **headers["Authorization"] = "Bearer " + bearer_token;**
 **}**
    const response = await fetch(baseUrl + url, {
       ** method, headers, body****: JSON.stringify(body)**
    });
    if (response.status == 200) {
        const data = await response.json();
        (Array.isArray(data) ? data : [data])
            .forEach(elem => console.log(JSON.stringify(elem)));
        **return data;**
    } else {
        console.log(response.status + " " + response.statusText);
    }
} 
```

成功登录后收到的令牌被分配给名为`bearer_token`的变量，该变量包含在后续请求的`Authorization`头中。请注意，客户端不会明确注销应用程序，而是简单地丢弃令牌。这是因为服务器不会跟踪它已发行的令牌，也没有使其失效的方法。一旦发行了令牌，它就有效，直到过期，因此 Web 服务客户端只需停止使用该令牌即可。

打开第二个命令提示符，并在`part2app`文件夹中运行*清单 15.16*中显示的命令以启动命令行客户端：

列表 15.16：启动客户端

```js
npm run cmdline 
```

选择**登录**选项，并以`alice`作为用户名，`mysecret`作为密码进行输入。服务器的响应显示了身份验证的结果和令牌，如下所示：

```js
...
? Select an operation Sign In
? Username? alice
? Password? mysecret
{"success":true,"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFsaWNlIiwiaWF0IjoxNzA2MzQ2NDgyLCJleHAiOjE3MDYzNTAwODJ9.YjWggUNH1aP9CSGSnQGIQqZc36aQE7RG_Cb0ovEOj1k"}
... 
```

您将看到不同的令牌值，但数据结构将是相同的。

# 授权请求

现在用户可以自行进行身份验证，下一步是限制基于用户身份的操作访问，以确保只有授权用户才能执行操作。

授权的基础是*授权策略*，它是应用程序提供的操作与允许执行这些操作的用户之间的映射。在最简单的情况下，映射可以表示为一个简单的列表，如示例应用程序中的*表 15.4*所示，它提供了`添加`和`删除`操作。

表 15.4：一个简单的授权策略

| 操作 | 授权用户 |
| --- | --- |
| `添加` |

```js
`alice, bob` 
```

|

| `删除` |
| --- |

```js
`alice` 
```

|

这种方法的缺点是授权策略是应用程序的一个组成部分，这意味着添加新用户或更改用户可以执行的操作需要代码更改，并需要测试和部署新版本。

因此，大多数应用程序引入了*角色*并依赖于*角色*分配来授权请求。而不是检查用户是否在列表中，应用程序会检查用户是否被分配给有权执行操作的某个角色。哪些用户被分配给每个角色的详细信息可以存储在数据库中，以便在不更改应用程序代码的情况下进行更改。

应用程序可以自由地以任何最有意义的方式将用户分配给角色，但一种常见的方法是关注他们与应用程序交互的不同方式。对于本章，我将创建两个角色，如*表 15.5*所述。

表 15.5：示例应用程序角色

| 角色 | 成员 |
| --- | --- |
| `Users` |

```js
`alice, bob` 
```

|

| `Admins` |
| --- |

```js
`alice` 
```

|

现在授权策略可以用授权角色的术语来表达，而不是单个用户，如*表 15.6*所述。

表 15.6：基于角色的授权策略

| 操作 | 授权角色 |
| --- | --- |
| `添加` |

```js
`Users` 
```

|

| `删除` |
| --- |

```js
`Admins` 
```

|

用户`alice`被分配了两个角色，将能够执行`添加`和`删除`操作。用户`bob`被分配了`用户`角色，将能够执行`添加`操作，但不能执行`删除`操作。

## 添加对角色的支持

第一步是扩展数据库以支持描述角色，如下所示*清单 15.17*：

列表 15.17：在 src/server/auth 文件夹中的 auth_types.ts 中添加类型

```js
export interface Credentials {
    username: string;
    hashedPassword: Buffer;
    salt: Buffer;
}
**export interface Role {**
 **name: string;**
 **members****: string[];**
**}**
export interface AuthStore {
    getUser(name: string) : Promise<Credentials | null>;
    storeOrUpdateUser(username: string, password: string):
        Promise<Credentials>;
    validateCredentials(username: string, password: string): Promise<boolean>
    **getRole(name: string) : Promise****<Role | null>;**
 **getRolesForUser(username: string): Promise<string[]>;**
 **storeOrUpdateRole(role: Role****) : Promise<Role>;**
 **validateMembership(username: string, role: string): Promise<boolean>;**
} 
```

*列表 15.18*定义了 Sequelize 将用于在数据库中表示角色的模型类，并修改了现有的模型类：

**注意**

为了保持一致性，将凭证和角色分配存储在同一个数据库中会更简单，这样用户账户的主键就可以用作角色成员的外键。如果你更喜欢使用单独的数据库，只要确保更改是一致性的，例如，当删除用户账户时更新角色成员。

列表 15.18：在 src/server/auth 文件夹中的 orm_auth_models.ts 文件中添加模型

```js
**import { DataTypes, InferAttributes, InferCreationAttributes, Model,**
 **Sequelize, HasManySetAssociationsMixin }**
 **from "sequelize";**
**import** **{ Credentials, Role } from "./auth_types";**
export class CredentialsModel
        extends Model<InferAttributes<CredentialsModel>,
            InferCreationAttributes<CredentialsModel>>
        implements Credentials {
    declare username: string;
    declare hashedPassword: Buffer;
    declare salt: Buffer;
 **   declare RoleModels?: InferAttributes<RoleModel>[];**
**}**
**export class RoleModel extends Model<InferAttributes<****RoleModel>,**
 **InferCreationAttributes<RoleModel>>  {**
 **declare name: string;**
 **declare CredentialsModels?: InferAttributes<CredentialsModel>[];**
 **declare setCredentialsModels:**
**HasManySetAssociationsMixin<CredentialsModel, string>;**
**}**
export const initializeAuthModels = (sequelize: Sequelize) => {
    CredentialsModel.init({
        username: { type: DataTypes.STRING, primaryKey: true },
        hashedPassword: { type: DataTypes.BLOB },
        salt: { type: DataTypes.BLOB }
    }, { sequelize });
  **  RoleModel.init({**
 **name: { type: DataTypes.STRING, primaryKey:** **true },**
 **}, {  sequelize });**
 **RoleModel.belongsToMany(CredentialsModel,**
 **{ through: "RoleMembershipJunction", foreignKey: "name" });**
 **CredentialsModel.belongsToMany(****RoleModel,**
 **{ through: "RoleMembershipJunction", foreignKey: "username" });** 
} 
```

存储角色成员资格需要一个**多对多关系**，其中每个角色可以与许多用户凭据相关联，每个用户凭据也可以与许多角色相关联。`RoleModel`类表示一个角色，具有提供角色名称和`CredentialsModels`对象数组的属性。

多对多关系在 SQL 中使用连接表表示，其中每一行代表一个用户和一个角色之间的关系。多对多关系通常由 ORM 包以尴尬的方式表示，并且通常需要一些尝试和错误才能使 ORM 创建的对象与创建的 SQL 表相匹配。Sequelize 有一个比平均水平更好的方法，并且使用`belongsToMany`方法创建模型类之间的关系，如下所示：

```js
...
RoleModel.belongsToMany(CredentialsModel,
    { through: "RoleMembershipJunction", foreignKey: "name" });
CredentialsModel.belongsToMany(RoleModel,
{ through: "RoleMembershipJunction", foreignKey: "username" });
... 
```

参数定义了使用名为`RoleMembershipJunction`类的表的多对多关系以创建连接表。Sequelize 将自动创建表并确定列数据类型，并在使用`include`查询配置设置时将关联数据包含在结果中，正如你将在*列表 15.18*中看到的那样。关联数据可以通过添加到模型类的属性来读取，这些属性被描述为 TypeScript 如下：

```js
...
declare RoleModels?: InferAttributes<RoleModel>[];
...
declare CredentialsModels?: InferAttributes<CredentialsModel>[];
... 
```

属性是可选的，正如`?`字符所示，因为它们只有在查询包含相关数据时才会被填充。Sequelize 为模型对象添加了方法，以便它们与其他模型的关系可以用于读取数据之外的操作。为了 TypeScript 的好处，必须使用`declare`关键字来描述属性，如下所示：

```js
...
declare **setCredentialsModels**: HasManySetAssociationsMixin<CredentialsModel, string>;
... 
```

分配给方法的名称结合了操作和模型类，例如`setCredentialsModels`是一个允许在单个操作中设置与`RoleModel`关联的`CredentialModels`对象的方法。

对于这些方法中的每一个，Sequelize 都提供了可以与`declare`语句一起使用的类型。在设置操作的情况下，类型被命名为`HasManySetAssociationsMixin`，并且使用泛型类型参数来指定关联模型类作为主键的类型。

**注意**

这是我在这个示例中需要的唯一方法，因为角色是通过替换所有成员来更新的，但 Sequelize 添加了读取、添加和删除关联数据的方法，以及每个方法的类型描述，如[`sequelize.org/docs/v6/core-concepts/assocs/#special-methodsmixins-added-to-instances`](https://sequelize.org/docs/v6/core-concepts/assocs/#special-methodsmixins-added-to-instances)中所述。

下一步是扩展存储以添加对查询、存储和检查角色的支持，如*列表 15.19*所示。

列表 15.19：在 src/server/auth 文件夹中的 orm_authstore.ts 文件中支持角色

```js
**import** **{ Sequelize, Op } from "sequelize";**
**import { CredentialsModel, initializeAuthModels, RoleModel }**
 **from "./orm_auth_models";**
**import** **{ AuthStore, Role } from "./auth_types";**
import { pbkdf2, randomBytes, timingSafeEqual } from "crypto";
export class OrmAuthStore implements AuthStore {
    sequelize: Sequelize;
    constructor() {
        this.sequelize = new Sequelize({
            dialect: "sqlite",
            storage: "orm_auth.db",
            logging: console.log,
            logQueryParameters: true
        });
        this.initModelAndDatabase();
    }
    async initModelAndDatabase() : Promise<void> {
        initializeAuthModels(this.sequelize);
        await this.sequelize.drop();       
        await this.sequelize.sync();       
        await this.storeOrUpdateUser("alice", "mysecret");
        await this.storeOrUpdateUser("bob", "mysecret");       
        **await this.storeOrUpdateRole({**
 **name: "Users",** **members: ["alice", "bob"]**
 **});**
 **await this.storeOrUpdateRole({**
 **name: "Admins", members: ["alice"]**
 **});**
    }
    // ...methods omitted for brevity...
   ** async getRole(name: string) {**
 **const stored = await RoleModel.findByPk(name, {**
 **include: [{** **model: CredentialsModel, attributes: ["username"]}]**
 **});**
 **if (stored) {**
 **return {**
 **name: stored.name,**
 **members: stored.CredentialsModels?.****map(m => m.username) ?? []**
 **}**
 **}**
 **return null;**
 **}**
 **async getRolesForUser(username: string): Promise<string[]> {**
 **return** **(await RoleModel.findAll({**
 **include: [{**
 **model: CredentialsModel,**
 **where: { username },**
 **attributes: []**
 **}]**
 **})).map(rm** **=> rm.name);**
 **}**
 **async storeOrUpdateRole(role: Role) {**
 **return await this.sequelize.transaction(****async (transaction) => {**
 **const users = await CredentialsModel.findAll({**
 **where: { username: { [Op.in]: role.members } },**
 **transaction**
 **});** 
**const [rm] = await RoleModel.findOrCreate({**
 **where: { name: role.name}, transaction });**
 **await rm.setCredentialsModels(users, { transaction });**
 **return role;**
 **});**
 **}**
**async validateMembership(username: string, rolename: string) {**
 **return (await this.getRolesForUser(username)).includes(rolename);**
 **}** 
} 
```

构造函数中的新语句初始化数据库模型，并添加 `Admins` 和 `Users` 角色，这些角色将用于演示授权过程。

`getRole` 方法查询数据库中的 `RoleModel` 对象，并使用 `include` 选项将相关的 `CredentialsModel` 对象包含在结果中，这被转换成 `RoleStore` 接口所需的 `Role` 结果。我只需要 `username` 值来创建一个角色，并且我已经使用 `attributes` 属性指定了 `Sequelize` 在查询中要包含的列：

```js
...
const stored = await RoleModel.findByPk(name, {
    include: [{ **model: CredentialsModel, attributes: ["username"]**}]
});
... 
```

`include` 属性配置为一个对象，其 `model` 属性指定相关数据，`attributes` 属性指定要在结果中填充的模型属性。

类似的技术用于实现 `getRolesForUser` 方法。使用 `findAll` 方法查询所有 `RoleModel` 对象，但使用 `where` 子句根据相关数据进行选择，以便只选择与 `CredentialsModel` 对象相关联且 `username` 属性与给定值匹配的 `RoleModel` 对象。使用空的 `attributes` 数组来排除结果中的所有 `CredentialModel` 数据：

```js
...
return (await RoleModel.findAll({
    include: [{
        model: CredentialsModel,
        where: { username },
       ** attributes: []**
    }]
})).map(rm => rm.name);
... 
```

当模型之间存在关系时，查询可能有几种不同的方法，并且可以从 `CredentialModel` 类开始，包括 `RoleModel` 类来获取相同的数据。我的建议是选择对你来说最自然的方法，这将是个人偏好的问题。

`storeOrUpdateRole` 方法接受一个 `Role` 对象，并查询数据库以找到所有匹配的 `CredentialsModel` 对象，这确保了任何没有用户凭据的名称都被忽略。`findOrCreate` 方法确保数据库中存在 `RoleModel` 对象，并使用 `setCredentialsModels` 方法设置角色成员资格。使用事务来确保更新是原子性的。

`validateMembership` 方法获取用户已被分配的角色，并检查其中之一是否与所需角色匹配。

## 检查授权

下一步是保护路由，以确保只有被分配到授权角色的用户才能执行操作。实现授权的方法有很多，但其中最简单的一种是使用 Express 的功能，允许将中间件组件添加到单个路由中，这意味着请求可以在传递给路由请求处理器之前进行检查和拒绝。*列表 15.20* 添加了一个创建中间件组件的函数，该组件限制对一个或多个角色的访问：

列表 15.20：在 src/server/auth 文件夹中的 index.ts 文件中定义守卫处理器

```js
**import { Express****, NextFunction, RequestHandler } from "express";**
import { AuthStore } from "./auth_types";
import { OrmAuthStore } from "./orm_authstore";
import jwt from "jsonwebtoken";
const jwt_secret = "mytokensecret";
const store: AuthStore = new OrmAuthStore();
type User = { username: string }
declare module "express-session" {
    interface SessionData { username: string; }
}
declare global {
    module Express {
        interface Request { user: User, authenticated: boolean }
    }
}
export const createAuth = (app: Express) => {
    // ...other routes omitted for brevity...
 **app.get("/unauthorized", async (req, resp) => {**
 **resp.render("unauthorized");**
 **});**
**}**
**export const roleGuard = (role: string)**
 **: RequestHandler****<Request, Response, NextFunction> => {**
 **return async (req, resp, next) => {**
 **if (req.authenticated) {**
 **const username = req.user.username****;**
 **if (await store.validateMembership(username, role)) {**
 **next();**
 **return;**
 **}**
 **resp.redirect("/unauthorized");**
 **} else {**
 **resp.redirect("/signin"****);** 
 **}**
 **}**
} 
```

`roleGuard`函数接受一个角色并返回一个中间件组件，该组件只有在用户被分配到该角色时才会将请求传递到处理器，这是通过 store 提供的`validateMembership`方法来检查的。

对于未经授权的请求，有两种结果。如果用户尚未认证，则用户将被重定向到`/signin` URL，以便他们可以自行认证并重试。

对于已认证的请求，用户将被重定向到`/unauthorized` URL。*列表 15.20*为`/unauthorized`添加了一个渲染模板的路由。要创建模板，请将名为`unauthorized.handlebars`的文件添加到`templates/server`文件夹中，其内容如*列表 15.21*所示：

列表 15.21：在 templates/server 文件夹中的 unauthorized.handlebars 文件的内容

```js
<div class="bg-danger text-white  m-1 p-2">
    <div class="h2">Unauthorized</div>
    <div class="h4">
        You do not have permission to perform this operation
    </div>
</div>
<a href="/" class="btn btn-secondary mx-1">Back</a> 
```

最后一步是将守卫应用到限制操作访问，如*列表 15.22*所示：

列表 15.22：在 src/server 文件夹中的 forms.ts 文件中授权请求

```js
import express, { Express } from "express";
import repository  from "./data";
import cookieMiddleware from "cookie-parser";
import { sessionMiddleware } from "./sessions/session_helpers";
**import { roleGuard }** **from "./auth";**
**import { Result } from "./data/repository";**
const rowLimit = 10;
export const registerFormMiddleware = (app: Express) => {
    app.use(express.urlencoded({extended: true}))
    app.use(cookieMiddleware("mysecret"));
    app.use(sessionMiddleware());
}
export const registerFormRoutes = (app: Express) => {
    app.get("/form", async (req, resp) => {
        resp.render("data", {data: await repository.getAllResults(rowLimit)});
    });
  **  app.post("/form/delete/:id", roleGuard("Admins"), async (req, resp) => {**
 **const id =** **Number.parseInt(req.params["id"]);**
        await repository.delete(id);
        resp.redirect("/form");
        resp.end();
    });

   ** app.post("****/form/add", roleGuard("Users"), async (req, resp) => {**
 **const nextage = Number.parseInt(req.body["age"])**
 **+ Number.****parseInt(req.body["years"]);**
 **await repository.saveResult({...req.body, nextage } as Result);**
        resp.redirect("/form");
        resp.end();
    });
} 
```

`/form/add`路由限制为`Users`角色，而`/form/delete/:id`路由限制为`Admins`角色。

应用角色守卫揭示了 Express API 类型描述中的不一致性，这导致 TypeScript 编译器对类似这样的语句发出警告：

```js
...
const id = Number.parseInt(req.params.id);
... 
```

编译器用于`Request.params`属性的类型已更改，编译器将对`id`属性发出警告。可以通过向请求处理器添加类型注解来纠正此问题，但快速修复方法是像这样访问属性：

```js
...
const id = Number.parseInt(req.params["id"]);
... 
```

使用浏览器请求`http://localhost:5000`，并确保没有用户登录。点击**添加**按钮。浏览器将向`/form/add`路由发送请求，但由于请求未经认证，浏览器将被重定向到`/signin`页面。使用用户名`bob`和密码`mysecret`登录，然后再次点击**添加**按钮。这次，请求已认证，用户已被分配到`Users`角色，因此请求被授权并传递到请求处理器，该处理器将新值添加到数据库中。

点击**删除**按钮，浏览器将向`/form/delete/:id`路由发送请求。请求已认证，但用户没有被分配到`Admins`角色，因此浏览器被重定向到`/unauthorized` URL。完整的序列如*图 15.4*所示。

![](img/B21959_15_04.png)

图 15.4：测试授权

## 授权 API 请求

我在*第十四章*中介绍的`Feathers`包提供了对钩子的支持，这允许拦截请求，并且我使用它来在抛出特定错误时更改状态码。在本章中，我将使用相同的功能来管理授权。第一步是创建一个函数，该函数将创建一个检查用户角色成员资格的钩子，如*列表 15.23*所示：

列表 15.23：在 src/server/auth 文件夹中的 index.ts 文件中添加钩子函数

```js
import { Express, NextFunction, RequestHandler } from "express"
import { AuthStore } from "./auth_types";
import { OrmAuthStore } from "./orm_authstore";
import jwt from "jsonwebtoken";
**import { HookContext } from "@feathersjs/feathers";**
const jwt_secret = "mytokensecret";
// ...statements omitted for brevity...
export const roleGuard = (role: string)
        : RequestHandler<Request, Response, NextFunction> => {
    return async (req, resp, next) => {
        if (req.authenticated) {
            const username = req.user.username;
            if (await store.validateMembership(username, role)) {
                next();
                return;
            }
            resp.redirect("/unauthorized");
        } else {
            resp.redirect("/signin");           
        }
    }
}
**export const roleHook = (****role: string) => {**
 **return async (ctx: HookContext) => {**
 **if (!ctx.params.authenticated) {**
 **ctx.http = { status:** **401 };**
 **ctx.result = {};**
 **} else if (!(await store.validateMembership(**
 **ctx.params.user.username, role))) {**
 **ctx.http = {** **status: 403 };**
 **ctx.result = {};**
 **}**
 **}**
**}** 
```

`roleHook`函数创建一个钩子，如果用户被分配到指定的角色，则授权访问。用户的身份通过`HookContext`参数访问，这是 Feathers 在调用钩子时提供的。与用于非 API 请求的路由守卫相比，主要区别在于响应是 HTTP 状态码而不是 HTML 文档。`401`状态码表示不包含认证数据的请求，当用户已认证但未授权时，会发送`403`状态码。状态码使用`HookContext.http`属性设置，设置`result`属性的效果是终止请求处理。

*列表 15.24*应用了钩子并配置了 Feathers，使钩子接收请求的认证数据：

列表 15.24：在 src/server/api 文件夹中的 index.ts 文件中应用授权

```js
import { Express } from "express";
import { createAdapter } from "./http_adapter";
import { ResultWebService } from "./results_api";
import { Validator } from "./validation_adapter";
import { ResultWebServiceValidation } from "./results_api_validation";
import { FeathersWrapper } from "./feathers_adapter";
import { feathers } from "@feathersjs/feathers";
import feathersExpress, { rest } from "@feathersjs/express";
import { ValidationError } from "./validation_types";
**import { roleHook } from "../auth";**
export const createApi = (app: Express) => {
    const feathersApp = feathersExpress(feathers(), app).configure(rest());
    const service = new Validator(new ResultWebService(),
        ResultWebServiceValidation);
    **feathersApp.use('/api/results',**
 **(req, resp, next) => {**
 **req.feathers.user = req.user;**
 **req.****feathers.authenticated = req.authenticated;**
 **next();**
 **},**
 **new FeathersWrapper(service));**
    feathersApp.hooks({
        error: {
            all: [(ctx) => {                       
                    if (ctx.error instanceof ValidationError) {
                        ctx.http = { status: 400};
                        ctx.error = undefined;
                    }
                }]
        },
       **before: {**
 **create: [roleHook("Users")],**
 **remove: [roleHook("Admins")],**
 **update: [roleHook("Admins")],**
**patch: [roleHook("Admins")]**
 **}** 
    });
} 
```

第一步是从请求中获取认证数据并将其添加到`feathers`属性中，该属性在 Feathers 被使用时添加到请求中，并通过提供给钩子的`HookContext`对象展示。

第二步是创建钩子来保护 Web 服务。`before`属性用于注册在调用 Web 服务方法之前被调用的钩子，而`create`、`remove`、`update`和`patch`方法由需要`Users`或`Admins`角色的钩子保护。

如果你关闭了本章早期提到的命令行客户端，那么打开一个新的命令提示符，导航到`part2app`文件夹，并运行*列表 15.25*中显示的命令：

列表 15.25：启动命令行客户端

```js
npm run cmdline 
```

选择**存储**选项，输入数据值，当请求发送时，服务器将响应`401`状态码，这表示操作需要授权，但请求中没有包含认证数据，如下所示：

```js
? Select an operation Store
? Name? Joe
? Age? 30
? Years? 10
401 Unauthorized 
```

使用密码`mysecret`以`bob`的身份登录并重复此过程。这次，请求将包含承载令牌，用户`bob`已被分配到`Users`角色，因此操作将被授权，如下所示：

```js
...
? Select an operation Sign In
? Username? bob
? Password? mysecret
{"success":true,"token":"eyJhbGciOi...<...data omitted...>"}
? Select an operation Store
? Name? Joe
? Age? 30
? Years? 10
201 Created
... 
```

选择**删除**并输入一个`ID`。请求将包含一个令牌，但`bob`没有被分配到`Admins`角色，因此服务器将响应`403`状态码：

```js
...
? Select an operation Delete
? ID? 1
403 Forbidden
... 
```

以`alice`的身份登录并重复执行`Delete`请求，这将成功，因为`alice`已被分配到`Admins`角色：

```js
...
? Username? alice
? Password? mysecret
{"success":true,"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VybmFtZSI6ImFsaWNlIiwiaWF0IjoxNzA2NDY1NzM2LCJleHAiOjE3MDY0NjkzMzZ9.GWEZl6qypJpdX-csNifgIRjZksZTxc-Nf35uVnTq4Ss"}
? Select an operation Delete
? ID? 1
true
... 
```

Web 服务的授权过程基于与 HTML 客户端相同的用户数据，但返回状态码使 API 客户端能够得到可以程序化处理的响应。

# 使用包进行认证和授权

现在你已经了解了在 Web 应用程序中认证和授权是如何协同工作的，是时候用开源包替换一些自定义代码了。

使用开源包进行身份验证有两个很好的理由。第一个理由是，在编写创建安全漏洞的自定义代码时很容易出错。第二个理由是，一个好的身份验证包将支持一系列不同的身份验证策略，包括使用第三方服务（如 Google 和 Facebook）进行身份验证。

在本书的 *第三部分* 中，我将演示不同的身份验证策略，但在这个章节中，我将使用开源包，但仍然使用用户名和密码进行身份验证。

并非所有功能都可以用自定义代码替换。重点往往放在身份验证上，而授权则留给各个应用程序来实现。

## 验证 HTML 客户端

我将使用 Passport 包 ([`www.passportjs.org`](https://www.passportjs.org)) 来提供身份验证。Passport 支持广泛的身份验证策略，包括对使用第三方身份验证服务的支持，并提供了一个 API 来实现自定义策略。在本章中，我使用 `Local` 策略，该策略支持使用本地存储的用户名和密码数据来验证用户，并使用会话来验证后续请求。我还使用了 **JWT** 策略，该策略使用携带令牌来验证请求。在 `part2app` 文件夹中运行 *清单 15.26* 中显示的命令来安装主要的 passport 包、包含策略的包以及所有这些包的类型描述：

清单 15.26：安装包

```js
npm install passport@0.7.0
npm install passport-local@1.0.0
npm install passport-jwt@4.0.1
npm install --save-dev @types/passport@1.0.16
npm install --save-dev @types/passport-local@1.0.38
npm install --save-dev @types/passport-jwt@4.0.1 
```

Passport 需要配置才能集成到应用程序中。将一个名为 `passport_config.ts` 的文件添加到 `src/server/auth` 文件夹中，其内容如 *清单 15.27* 所示：

清单 15.27：src/server/auth 文件夹中 passport_config.ts 文件的内容

```js
import passport from "passport";
import { Strategy as LocalStrategy }  from "passport-local";
import { Strategy as JwtStrategy, ExtractJwt  } from "passport-jwt";
import { AuthStore } from "./auth_types";
type Config = {
    jwt_secret: string,
    store: AuthStore
}
export const configurePassport = (config: Config) => {
    passport.use(new LocalStrategy(async (username, password, callback) => {
        if (await config.store.validateCredentials(username, password)) {
            return callback(null, { username });
        }
        return callback(null, false);
    }));
    passport.use(new JwtStrategy({
        jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
        secretOrKey: config.jwt_secret
    }, (payload, callback) => {
        return callback(null, { username: payload.username });
    }));
    passport.serializeUser((user, callback) => {
        callback(null, user);
    });
    passport.deserializeUser((user, callback) => {
        callback(null, user as Express.User );
    });   
} 
```

`passport.use` 函数用于设置策略，在 *清单 15.27* 中应用了本地和 JWT 策略。这些策略需要一个验证函数，该函数接收请求数据并返回一个表示用户的对象。

本地策略的验证函数接收用户发送的用户名和密码，使用存储的凭证进行验证。结果通过回调提供给 Passport，要么提供一个表示用户的对象，要么在验证失败时返回 `false`：

```js
...
if (await config.store.validateCredentials(username, password)) {
   ** return callback(null, { username });**
**}**
**return callback(null, false);**
... 
```

验证函数仅在用户登录时被调用，之后 Passport 使用临时令牌来验证后续请求。一个选项是使用会话 cookie 来存储用户数据，这与自定义代码中使用的方法相同。

JWT 策略的验证方式不同。Passport 不生成 JWT 令牌，而是在收到携带令牌时调用验证函数。策略配置了一个对象，告诉 Passport 如何在请求中定位携带令牌，并提供检查令牌签名的密钥。验证函数接收令牌的有效负载，并负责提供一个表示已认证用户的对象。

`serializeUser` 和 `deserializeUser` 函数由 Passport 用于在会话中包含用户信息。即使在这种情况下，用户数据只是一个包含用户名的对象，这些函数也必须被定义。*列表 15.28* 使用 Passport 验证请求：

列表 15.28：在 src/server/auth 文件夹中的 index.ts 文件中使用 Passport

```js
import { Express, NextFunction, RequestHandler } from "express"
import { AuthStore } from "./auth_types";
import { OrmAuthStore } from "./orm_authstore";
import jwt from "jsonwebtoken";
import { HookContext } from "@feathersjs/feathers";
**import passport** **from "passport";**
**import { configurePassport } from "./passport_config";**
const jwt_secret = "mytokensecret";
const store: AuthStore = new OrmAuthStore();
**//type User = { username: string }**
declare module "express-session" {
    interface SessionData { username: string; }
}
declare global {
    module Express {
       ** //interface Request { user: User, authenticated: boolean }**
 **interface Request { authenticated: boolean }**
 **interface User {**
 **username: string**
 **}**
    }
}
export const createAuth = (app: Express) => {
    configurePassport({ store, jwt_secret });
    app.get("/signin", (req, resp) => {
        const data = {
            **// username: req.query["username"],**
 **// password: req.query["password"],**
            failed: req.query["failed"] ? true : false,
            signinpage: true
        }
        resp.render("signin", data);
    });
   ** app.post****("/signin", passport.authenticate("local", {**
 **failureRedirect: `/signin?failed=1`,**
 **successRedirect: "/"**
 **}));**
 **app.use(passport.authenticate****("session"), (req, resp, next) => {**
 **resp.locals.user = req.user;**
 **resp.locals.authenticated**
 **= req.authenticated = req.user** **!== undefined;** 
 **next();**
 **});**
    app.post("/api/signin", async (req, resp) => {
        const username = req.body.username;
        const password = req.body.password;
        const result: any = {
            success: await store.validateCredentials(username, password)
        }
        if (result.success) {
            result.token = jwt.sign({username} , jwt_secret,
                { expiresIn: "1hr"});
        }
        resp.json(result);
        resp.end();   
    });
    app.post("/signout", async (req, resp) => {
        req.session.destroy(() => {
            resp.redirect("/");
        })
    });
    app.get("/unauthorized", async (req, resp) => {
        resp.render("unauthorized");
    });
}
export const roleGuard = (role: string)
        : RequestHandler<Request, Response, NextFunction> => {
    return async (req, resp, next) => {
        if (req.authenticated) {
           ** const username = req.user****?.username;**
 **if (username != undefined**
 **&& await store.validateMembership(username, role)) {**
                next();
                return;
            }
            resp.redirect("/unauthorized");
        } else {
            resp.redirect("/signin");
        }
    }
}
export const roleHook = (role: string) => {
    return async (ctx: HookContext) => {
        if (!ctx.params.authenticated) {
            ctx.http = { status: 401 };
            ctx.result = {};
        } else if (!(await store.validateMembership(
                ctx.params.user.username, role))) {
            ctx.http = { status: 403 };
            ctx.result = {};
        }
    }
} 
```

Passport 为 `Express` 的 `Request` 对象提供了自己的扩展，因此需要进行调整以防止冲突。`Passport` 的 `authenticate` 函数被使用两次。当与路由一起使用时，`authenticate` 方法用于创建一个请求处理器，该处理器将使用本地策略验证凭据：

```js
...
app.post("/signin", passport.**authenticate**("local", {
... 
```

配置选项告诉 Passport 在成功和失败的登录尝试中应将浏览器重定向到何处。Passport 不在失败的登录尝试的重定向中包含用户名和密码，这就是为什么在渲染登录模板时不再包含这些值。

`authenticate` 函数的另一种用途是验证请求，参数指定会话是认证数据的来源：

```js
...
app.use(passport.**authenticate**("session"), (req, resp, next) => {
... 
```

Passport 没有名为 `authenticate` 的属性，但后续的处理函数允许设置该属性，以及模板所需的本地数据。

如前所述，Passport 不创建 JWT 令牌，因此验证 API 客户端的代码保持不变。然而，Passport 确实 *验证* JWT 令牌，这就是为什么在 *列表 15.28* 中移除了读取和验证携带令牌的代码。*列表 15.29* 使用 Passport 验证 Web 服务请求的携带令牌：

列表 15.29：在 src/server/api 文件夹中的 index.ts 文件中使用 Passport

```js
import { Express } from "express";
import { createAdapter } from "./http_adapter";
import { ResultWebService } from "./results_api";
import { Validator } from "./validation_adapter";
import { ResultWebServiceValidation } from "./results_api_validation";
import { FeathersWrapper } from "./feathers_adapter";
import { feathers } from "@feathersjs/feathers";
import feathersExpress, { rest } from "@feathersjs/express";
import { ValidationError } from "./validation_types";
import { roleHook } from "../auth";
**import passport from "passport";**
export const createApi = (app: Express) => {
    const feathersApp = feathersExpress(feathers(), app).configure(rest());
    const service = new Validator(new ResultWebService(),
        ResultWebServiceValidation);
    **feathersApp.use('/api/results',**
 **passport.authenticate("jwt", { session: false** **}),**
 **(req, resp, next) => {**
 **req.feathers.user = req.user;**
 **req.feathers.authenticated**
 **= req.authenticated = req.user !== undefined****;**
 **next();**
 **},**
 **new FeathersWrapper(service));**
    feathersApp.hooks({
        error: {
            all: [(ctx) => {                       
                    if (ctx.error instanceof ValidationError) {
                        ctx.http = { status: 400};
                        ctx.error = undefined;
                    }
                }]
        },
        before: {
            create: [roleHook("Users")],
            remove: [roleHook("Admins")],
            update: [roleHook("Admins")],
            patch: [roleHook("Admins")]
        }       
    });
} 
```

`authenticate` 函数用于创建一个请求处理器，该处理器将使用 JWT 策略验证令牌。后续函数用于设置 Feathers 钩子使用的值，以便执行授权检查。

**注意**

Feathers 有自己的认证和授权功能，如果你正在创建一个独立的 API 项目，这些功能非常有用，它们在 [`feathersjs.com/api/authentication`](https://feathersjs.com/api/authentication) 中进行了描述。混合来自多个包的认证功能可能很困难，这就是为什么我在示例中使用了 Passport 进行所有认证，即使某些功能，如 JWT 创建，不可用。

Passport 包的使用不会改变身份验证的工作方式，用户以相同的方式登录应用程序。区别在于减少了自定义代码并支持更广泛的身份验证策略，这使得将应用程序集成到现有环境或使用第三方身份验证服务变得更加容易。

执行基于角色的授权仍需要自定义代码，这就是为什么理解用户和请求是如何进行身份验证的，以及如何使用这些结果来限制对应用程序功能的访问很重要的原因。

# 摘要

在本章中，我演示了如何对用户和请求进行身份验证，以及如何使用身份验证数据授权访问应用程序功能：

+   用户通过 HTML 表单或 JSON 有效负载展示其凭证。

+   当凭证得到验证时，客户端会收到一个临时令牌，该令牌可用于验证后续请求。

+   临时身份验证令牌可以是 cookie（通常使用会话 cookie）或携带令牌。

+   授权通常通过角色来执行，这可以防止你需要在应用程序中硬编码用户权限。用户和角色之间的关系存储在数据库中，因此可以在不发布新版本应用程序的情况下进行修改。

+   对于用户和请求的身份验证，有可用的开源包，但通常使用自定义代码进行授权。

在**第三部分**中，我使用本书**第一部分**和**第二部分**中描述的功能创建了一个在线商店，展示了应用程序的不同部分是如何协同工作的。
