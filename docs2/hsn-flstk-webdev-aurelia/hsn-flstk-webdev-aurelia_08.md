# 第八章：安全性

安全性对于你计划构建的每一个应用程序都至关重要。安全性是一个非常复杂的话题，应该考虑最佳实践和标准来分析和实施。**开放网络应用安全项目**（**OWASP**）组织是一个全球性的非营利组织，专注于提高应用程序的安全性。

所有应用程序至少实现了一个简单的安全层，称为身份验证和授权层，它负责根据提供给应用程序的凭证限制某些访问和功能。尽管本章的重点是如何保护我们的 Aurelia 应用程序，但我们将实现一个简单的身份验证和授权 API 作为示例，以集成到我们的 Aurelia 网络应用程序中。

在本章中，我们将涵盖以下主题：

+   理解 JSON Web Tokens

+   自定义身份验证和授权

+   介绍 Auth0

+   使用 Auth0 进行社交登录

+   单点登录

# 理解 JSON Web Tokens

我们实现了一个 RESTful API，该 API 为我们的 Aurelia 网络应用程序提供显示信息。这个 API 没有任何安全机制，所以如果任何恶意用户获取端点 URL，他们可以针对我们的应用程序运行恶意脚本并破坏我们的应用程序。因此，我们应该拒绝任何未经授权用户执行的管理操作。

我们的应用程序应该实现一个机制来管理用户的访问和权限。实现身份验证和授权有许多方法。对于我们的应用程序，我们将使用一个行业中的开放标准，称为**JSON Web Token**（**JWT**）。

# JWT

JWTs 是一种开放、行业标准的 RFC 7519 方法，用于在双方之间安全地表示声明。我们使用它们的方式很简单。首先，我们通过提供用户名或密码来验证后端服务器。如果我们的凭证正确，后端服务器将生成一个包含应在客户端使用本地存储机制持久化的用户信息的令牌。这个 JWT 应该在每次请求中传递给服务器，以便服务器可以识别用户是谁以及该用户有什么权限；有了这些信息，服务器允许或拒绝用户请求。

让我们了解它是如何工作的。导航到 [`jwt.io/`](https://jwt.io/)；向下滚动一点，你会找到一个类似于以下图像的示例部分：

![图片](img/b2a724d0-8083-4afb-80ea-bb168770cb33.png)

从右到左读取信息。在右侧，我们有解码部分，它包含三个部分：

+   头部：包含用于**加密**令牌的**算法**信息

+   负载：我们将定义并用于我们应用程序中的**信息**片段，例如，用户信息

+   验证签名：令牌的签名；我们将定义一个 **密钥值** 来加密我们的令牌

在左侧的编码中，我们可以看到基于前面提到的三个部分信息的最终令牌加密结果。

如你所猜，这个加密值是在我们的后端服务器中计算的。每次用户登录应用程序时，都会将此令牌发送给用户。他们在客户端保存此令牌，然后在每次请求中使用 `Authorization` HTTP 头将其发送。让我们看看这一切是如何工作的。

# 自定义身份验证和授权

现在我们来了解应用安全背后的两个主要概念，这些概念你必须在你所有的项目中实现。

# 实现身份验证

身份验证是一个验证给定用户身份并检查用户是否有有效凭证访问我们的应用程序或后端 API 的过程。通过身份验证，我们限制了非我们应用程序成员的访问。

我们将创建一个基本的身份验证 API，因为本书的目标是向你展示如何保护你的 Aurelia 应用。我们不会深入探讨后端实现的细节。我们将创建一个硬编码的身份验证流程，但你可以使用 Mongoose 将其与数据库集成，如第六章“将我们的数据存储在 MongoDB”中所述。

因此，让我们动手实践。打开后端项目，在 `routes` 文件夹中创建一个名为 `security-api.js` 的新文件，并编写以下代码：

```js
const express = require('express')
const api = express.Router()

const logIn = (username, password) => {
    // Logic Here    
}

api
  .route('/auth')
  .post((req, res, next) => {
    // Logic here
  })

module.exports = api
```

首先，我们导入 `express` 并创建 `Router` 类的一个实例。其次，我们定义一个名为 `logIn` 的函数，我们将在这个函数中实现用户认证和生成 JWT 的逻辑。然后，我们将定义一个 `/auth` 路由来处理 `POST` 端点。最后，我们将 API 导出以在主 `server.js` 文件中使用。

# 身份验证逻辑

让我们创建一个简单的身份验证逻辑。后端期望提供一个用户名和密码，其值为管理员。如果提供了这些值，它将返回一个有效的令牌；否则，它将返回 `null`。对以下代码进行以下更改：

```js
const express = require('express')
const api = express.Router()

const logIn = (username, password) => {
    if (username === 'admin' && password === 'admin') {
const userData = {
 name: "Admin"
 }
 return generateToken(userData)
 }
 return null
}

api
...
```

代码非常直接。首先，我们在简单的 `if` 条件中比较值，并创建了一个 `userData` 对象，该对象将包含用户信息，在这种情况下只提供了一个名称值。最后，我们将调用一个 `generateToken` 函数并将 `userData` 传递给它以返回一个有效的令牌。

让我们实现 `generateToken` 函数。

# 生成令牌

我们将使用一个名为 `jsonwebtoken` 的 NPM 模块来生成 JWT。打开一个新的控制台，在 `wc-backend` 文件夹中输入以下命令来安装模块：

```js
$ npm install jsonwebtoken --save
```

安装完成后，打开 `security-api.js`，然后按照以下方式导入我们的库：

```js
const express = require('express')
const jwt = require('jsonwebtoken')
const api = express.Router()

...
```

在我们的文件中导入依赖项后，让我们实现 `generateToken` 函数。应用以下更改：

```js
...
const logIn = (username, password) => {
    if (username == 'admin' && password == 'admin') {        
        let userData = {
            name: "Admin"
        }        
        return generateToken(userData)        
    } else {
        return null
    }    
}

const generateToken = userData => {
 return jwt.sign(userData, "s3cret", { expiresIn: '3h' })
} ...
```

就这样！让我们理解一下代码。我们调用`jwt`对象的`sign`函数来创建我们的令牌。我们将以下信息传递给该函数：

+   `userData`: 我们想要标记化的信息

+   `secret`: 用于加密和验证令牌的秘密值

+   `expiration`: 令牌的过期日期

现在，我们已经准备好了认证逻辑。为了完成我们的实现，我们必须通过我们的 REST 控制器提供此逻辑。

# 认证 REST 控制器

我们已经定义了负责将我们的逻辑作为 REST 端点提供的路由，所以唯一需要做的步骤就是调用我们的`logIn`函数。继续并应用以下更改：

```js
...

const generateToken = (userData) => {
   return jwt.sign(userData, "s3cret", { expiresIn: '3h' })
}

api
  .route('/auth')
  .post((req, res, next) => {
    let { username, password } = req.body
 let token = logIn(username, password)
 if (token) {
 res.send(token)
 } else {
 next(new Error("Authentication failed"))
 }
  })

module.exports = api
```

首先，我们从`req.body`对象中提取`username`和`password`。之后，我们调用`logIn`函数并将结果存储在`token`变量中。如果`token`不为空，我们通过调用`res.send`函数返回一个成功的响应。如果令牌为空，我们将一个`Error`对象传递给下一个参数，这将引发一个全局异常并返回一个失败的响应。

最后，我们必须修改`server.js`文件以将我们的 API 注册到 express 中，如下所示：

```js
const express = require('express')
..
const seurityApi = require('./src/routes/security-api')
const mongooseConfig = require('./src/config/mongoose-connection')
const app = express()

app.use(bodyParser.json())
app.use(teamsApi)
app.use(seurityApi)
...
```

现在，我们已经准备好测试我们的实现了。在一个新的终端窗口中，运行以下`curl`命令：

```js
$ curl -X POST -H "Content-type: application/json" -d '{"username":"admin", "password":"admin"}' localhost:3000/auth

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoiQWRtaW4iLCJpYXQiOjE1MTk1NzYwMDEsImV4cCI6MTUxOTU4NjgwMX0.4cNGYgz_BZZz5GEfN6MS3pkreGTkUBqJS1FZVC3_ew
```

如果一切实施得当，你应该会收到一个加密的 JWT 作为响应。

太棒了！现在是时候玩授权了。继续阅读！

# 实现授权

通过认证，我们确保我们的应用程序是由一个拥有有效凭证的授权人员使用的。在您未来构建的大多数应用程序中，您会发现有不同权限的用户。例如，一个学生可能有查看成绩的权限，但学生无法修改成绩。否则，一个教师可以更新成绩并访问学生无法访问的其他功能。

我们将使用另一个 NPM 模块`express-jwt-permissions`来实现授权。通过使用此模块，我们将能够以非常简单的方式实现授权。打开`security-api.js`文件并应用以下更改：

```js

const logIn = (username, password) => {
    if (username == 'admin' && password == 'admin') {

        let userData = {
            name: "Admin",
            permissions: ["admin:create:match", "admin:update:scores"]
        }

        return generateToken(userData)

    } else {
        return null
    }

}
```

就这样！之前的库将检查 JWT 是否定义了`permissions`属性。如果是这样，我们将提取此信息并限制没有管理员权限的用户访问。在下一节中，我们将实现管理员 REST 控制器并查看我们是如何限制对更多细节的访问的。

话虽如此，我们已经准备好开始实施管理员 API 了！

# 创建管理员 API

为了理解授权是如何工作的，让我们实现一个基本的仅由我们网站管理员访问的 Admin API。我们的应用程序有两种主要类型的用户：

+   **正常**: 此用户能够查看特色比赛和得分

+   **管理员**: 此用户负责创建新比赛和更新得分

我们将在后端使用两个开源 NPM 模块来管理限制工作流程。以下图表更详细地解释了这个流程：

![图片](img/723782aa-7a9b-4e58-a8da-8e4202c53208.png)

所有的操作都是从用户请求开始的，该请求试图访问一个受限的端点。后端首先验证 HTTP 请求中是否存在有效的令牌；这个验证是由`express-jwt`模块执行的。其次，如果请求有一个有效的令牌，流程将检查该令牌是否有访问受限端点的有效权限；这个验证是由`express-jwt-permissions`执行的。如果请求有一个有效的令牌并且有权限，用户请求将能够访问受限端点并执行操作。

# 管理比赛

要创建一个比赛，我们需要创建一个有效的 Match 数据库模式。我们已经知道如何使用 Mongoose 来做这件事。让我们来做。

# 创建 Match 模式

在`src/models`文件夹中创建`match.js`文件。然后，添加以下代码：

```js
const mongoose = require('mongoose')

const MatchSchema = new mongoose.Schema({
    team_1: {
      type: String,        
      min: 3,
      max: 100,
      required: true
    },
   team_2: {
      type: String,
      min: 3,
      max: 100,
      required: true
    },
    score: {
      team_1: Number,
      team_2: Number
    }
})

module.exports = mongoose.model('match', MatchSchema)
```

我们定义了三个属性，前两个`team_1`和`team_2`将存储两支参赛队伍的信息。比赛的`score`。这就是我们模型所需的所有内容。

# 创建 REST 控制器

让我们从在`src/routes`文件夹中创建一个名为`admin-api.js`的新文件开始。然后，编写以下代码：

```js
const express = require('express')
const api = express.Router()

api
  .route('/admin/match/:id?')
  .post((req, res, next) => {

     // logic to create Match

  })
   .put((req, res, next) => {

     // logic to update Scores

  })

module.exports = api
```

你非常熟悉这种代码结构。首先，我们导入定义我们的 REST 控制器所需的模块。其次，我们创建一个`/admin/match/:id?`路由并定义创建新比赛的`POST`方法以及更新分数的另一个`PUT`方法。

注意路由定义；我们声明了一个名为`:id`的可选路径变量。为了使路径可选，我们在其名称后添加了`?`运算符。

到目前为止一切顺利。让我们来实现它们。

# 创建比赛

创建一个新的`Match`很简单。我们只需要导入`Match`模型并调用其内置的`save`方法，如下所示：

```js
const express = require('express')
const api = express.Router()
const Match = require('../models/match')

api
  .route('/admin/match/:id?')
  .post((req, res, next) => {
      const match = new Match(req.body)
 match.save()
 .then(data => res.json(data))
 .catch(err => next(err) )
  })
   .put((req, res, next) => {

     // logic to update Match

  })

module.exports = api
```

首先，我们导入`Match`模型。然后，在`POST`方法中，我们创建一个新的`Match`对象并调用`save`函数。如果操作成功，我们通过`res.json`方法发送新的`Match`。为了测试我们的创建逻辑，我们需要配置`server.js`以使用我们的新 Admin API，如下所示：

```js
const express = require('express')
...
const adminApi = require('./src/routes/admin-api')
const mongooseConfig = require('./src/config/mongoose-connection')
const app = express()

app.use(bodyParser.json())
app.use(teamsApi)
app.use(seurityApi)
app.use(adminApi)

app.use((err, req, res, next) => {
```

一旦我们应用了前面的更改，打开一个新的终端来测试一下：

```js
$ curl -X POST -H "Content-type: application/json" -d '{"team_1": "Peru", "team_2": "Chile", "score": { "team_1": 20, "team_2": 0} }'  localhost:3000/admin/match/

{"__v":0,"team_1":"Peru","team_2":"Chile","_id":"5a94a2b8221bb505c92d801c","score":{"team_1":20,"team_2":0}}
```

太棒了！现在我们有了我们的创建逻辑，并且有一个很好的真实示例。

# 列出比赛

要列出我们的比赛，我们不需要安全措施，因为所有用户都应该能够获取应用程序中比赛的完整列表。所以让我们实现`matches` API。

因此，在`src/routes`文件夹中创建一个名为`matches-api.js`的新文件，并应用以下代码：

```js
const express = require('express')
const api = express.Router()
const Match = require('../models/match')

api
  .route('/matches')
  .get((req, res, next) => {
     Match.find().exec()
        .then(matches => res.json(matches))
        .catch(err => next(err))
  })

module.exports = api
```

接下来，我们必须配置`server.js`文件以映射我们的`Match` API。在`server.js`文件中，应用以下更改：

```js
...
const adminApi = require('./src/routes/admin-api')
const matchesApi = require('./src/routes/matches-api')
const mongooseConfig = require('./src/config/mongoose-connection')
const app = express()

app.use(bodyParser.json())
app.use(teamsApi)
app.use(seurityApi)
app.use(adminApi)
app.use(matchesApi)

...
```

太棒了！让我们来测试一下。打开一个终端窗口并执行以下命令：

```js
$ curl localhost:3000/matches

[{"_id":"5a949f982c1fda05b8c5c00a","team_1":"Peru","team_2":"Chile","__v":0,"score":{"team_1":20,"team_2":0}}]
```

就这样！我们有了我们的公共`Match` API。

# 更新分数

要更新 `score`，我们需要查找现有比赛的 ID。如果找到一个有效的比赛，我们将应用更新。如果没有找到比赛，我们必须响应一个 `404 not found` HTTP 响应。否则，我们响应一个 `Success 200` HTTP 响应。

让我们先创建 `updateScore` 函数，如下所示：

```js
...
const Match = require('../models/match')

const updateScore = async (matchId, teamId) => {
 try {
 let match = await Match.findById(matchId)

 if (match == null) throw new Error("Match not found")

 if (teamId == 'team_1') {
 match.score.team_1++;
 } else {
 match.score.team_2++;
 }

 match = await match.save()
 return match

 } catch (err) {
 throw err
 }
}

api
  .route('/admin/match/:id?')
...
```

现在，让我们按照以下方式调用我们的 PUT HTTP 动词中的函数：

```js
...
api
  .route('/admin/match/:id?')
  .post((req, res, next) => {
     ...
  })
api
  .route('/admin/match/scores/:id')
 .post((req, res, next) => {

 const matchId = req.params.id
 const teamId = req.body.teamId

 updateScore(matchId, teamId)
 .then(match => res.json(match))
 .catch(err => next(err))
 })
...
```

太棒了！让我们测试一下。执行以下 `curl` 命令来更新我们之前创建的比赛：

```js
$ curl -X POST -H "Content-type: application/json" -d '{"teamId": "team_1" }'  localhost:3000/admin/match/scores/5a94a2b8221bb505c92d801c

{"_id":"5a94a2b8221bb505c92d801c","team_1":"Peru","team_2":"Chile","__v":0,"score":{"team_1":21,"team_2":0}}
```

注意，`team_1`（秘鲁队）现在在它的分数中有了 `21` 个进球，而智利队有 `0` 个。

太棒了！现在我们能够创建一个新的比赛并更新分数了。让我们使用 `express-jwt` 和 `express-jwt-permissions` 来保护我们的 API。继续阅读！

# 保护 REST 控制器

现在是时候保护我们的 API 了。为了做到这一点，让我们首先安装我们的两个 NPM 模块。打开终端窗口并运行以下命令：

```js
$ npm install --save express-jwt express-jwt-permissions
```

安装完成后，让我们验证我们的令牌。

# 验证令牌

每个 HTTP 请求都必须在授权头中发送 JWT。`express-jwt` 中间件将检查是否传递了有效令牌。如果是，请求将继续其流程，否则后端将响应一个未授权异常。

将以下更改应用到 `admin-api.js` 文件中：

```js
...
const Match = require('../models/match')
const auth =require('express-jwt')

const updateScore = async (matchId, teamId) => {
  ...
}

api
  .route('/admin/match/:id?')
  .post(auth({ secret: 's3cret'}),
  (req, res, next) => {
    ...
  })
...
```

首先，我们开始导入 `express-jwt` 模块并创建一个新的常量 `auth`。其次，我们使用 `auth` 函数并传递一个具有 `secret` 属性的 JSON 对象；我们将使用与我们在 `security-api.js` 中签名令牌相同的密钥值：

```js
return jwt.sign(userData, "s3cret", { expiresIn: '3h' })
```

`auth` 函数负责检查是否在授权头中传递了有效的令牌。让我们尝试不传递有效令牌创建一个新的 `Match` 实体。打开终端窗口并执行以下命令：

```js
$ curl -X POST -H "Content-type: application/json" -d '{"team_1": "Peru", "team_2": "France", "score": { "team_1": 5, "team_2": 5} }'  localhost:3000/admin/match/ {"error":"No authorization token was found"}

```

如您所见，这次我们收到了一个错误消息，表示我们的后端 API 正在等待一个有效令牌，但没有提供。因此，为了应对这种情况，我们需要首先创建一个有效令牌。让我们通过调用我们的安全 API 并以管理员身份登录来创建一个有效令牌。执行以下命令：

```js
$ curl -X POST -H "Content-type: application/json" -d '{"username":"admin", "password":"admin"}' localhost:3000/auth

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoiQWRtaW4iLCJpYXQiOjE1MTk2OTgyMjksImV4cCI6MTUxOTcwOTAyOX0.HQiz-NbBDBc9kVyBRNUeMsrDexEsk92WXoRyijNp1Rk 
```

为我创建了一个新的有效令牌。请注意，此令牌具有随机值，因此为您生成的令牌将完全不同。

一旦我们有了有效的令牌，我们再次尝试；然而，这次我们将使用 `Authorization` HTTP 头部传递令牌，如下所示：

```js
$ curl -X POST -H "Content-type: application/json" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoiQWRtaW4iLCJpYXQiOjE1MTk2OTgyMjksImV4cCI6MTUxOTcwOTAyOX0.HQiz-NbBDBc9kVyBRNUeMsrDexEsk92WXoRyijNp1Rk" -d '{"team_1": "Peru", "team_2": "France", "score": { "team_1": 5, "team_2": 5} }'  localhost:3000/admin/match/

{"__v":0,"team_1":"Peru","team_2":"France","_id":"5a94c2559e11b6089b2b265e","score":{"team_1":5,"team_2":5}}
```

太棒了！现在我们能够创建一个新的 `Match`。请注意我们传递令牌时使用的语法。我们使用了 `Authorization: Bearer <token>` 语法。

我们已经保护了我们的 API，但如果我们有两种不同类型的管理员怎么办？比如说，一组管理员只能添加 `Match` 实体，而另一组管理员只能更新分数。我们需要一种方式来管理这种角色分离。让我们学习权限是如何工作的。

# 验证权限

权限允许我们限制对一组资源的访问。你应该意识到，如果我们想保护我们的后端 API，仅进行身份验证是不够的。要实现权限，请打开`admin-api.js`文件并应用以下更改：

```js
...
const auth =require('express-jwt')
const guard = require('express-jwt-permissions')()

const updateScore = async (matchId, teamId) => {
  ...
}

api
  .route('/admin/match/:id?')
  .post(auth({ secret: 's3cret'}),
    guard.check('admin:create:match'),
    (req, res, next) => {

      ...    

  })
...
```

首先，我们初始化一个`guard`常量。其次，我们调用`guard.check`；这个函数将在 JWT 中查找`admin:create:match`权限。记住，这些权限必须存在于令牌中。如果用户有权限，流程将继续，并创建新的`Match`。否则，我们将收到`无法找到权限`错误。

让我们尝试创建一个新的`Match`；执行以下命令：

```js
$ curl -X POST -H "Content-type: application/json" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoiQWRtaW4iLCJpYXQiOjE1MTk2OTgyMjksImV4cCI6MTUxOTcwOTAyOX0.HQiz-NbBDBc9kVyBRNUeMsrDexEsk92WXoRyijNp1Rk" -d '{"team_1": "Peru", "team_2": "Brasil", "score": { "team_1": 80, "team_2": 5} }'  localhost:3000/admin/match/

{"error":"Permission Denied"}
```

这很有趣！尽管我们已经进行了身份验证并传递了一个有效的令牌，但我们为什么不能创建`Match`？让我们看看用户令牌生成逻辑。打开`security-api.js`文件：

```js
...
const logIn = (username, password) => {
    if (username == 'admin' && password == 'admin') {

        let userData = {
            name: "Admin"
        }

        return generateToken(userData)

    } else {
        return null
    }

}
...
```

正如你所见，我们的令牌没有定义权限。让我们通过添加正确的权限来解决这个问题：

```js
...    
    let userData = {
       name: "Admin",
      permissions: ["admin:create:match"] 
    }
...
```

那就结束了。让我们再次登录以生成一个新的令牌，并测试一下。

首先，执行以下命令以生成一个有效的令牌：

```js
$ curl -X POST -H "Content-type: application/json" -d '{"username":"admin", "password":"admin"}' localhost:3000/auth

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoiQWRtaW4iLCJpYXQiOjE1MTk2OTk4MzIsImV4cCI6MTUxOTcxMDYzMn0.cVTtJHcbQ2J76s6uRjuySCWq4dKXlNzAfInl0ZLgri 
```

太棒了！这个新的令牌包含了权限。接下来，让我们尝试再次通过传递这个新的令牌来创建新的`Match`：

```js
$ curl -X POST -H "Content-type: application/json" -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJuYW1lIjoiQWRtaW4iLCJpYXQiOjE1MTk2OTgyMjksImV4cCI6MTUxOTcwOTAyOX0.HQiz-NbBDBc9kVyBRNUeMsrDexEsk92WXoRyijNp1Rk" -d '{"team_1": "Peru", "team_2": "China", "score": { "team_1": 5, "team_2": 5} }'  localhost:3000/admin/match/

{"__v":0,"team_1":"Peru","team_2":"China","_id":"5a94c87987d2820a0d1931e7","score":{"team_1":5,"team_2":5}}
```

热狗！我们现在能够管理后端 API 中的身份验证和授权。当然，你可以通过在 MongoDB 数据库中保存用户集合并创建具有不同角色的不同用户来提高这种安全身份验证。此外，使用`express-jwt-blacklist`实现`登出`，以另一种方式，但出于本书的目的，我们对此基本实现感到满意。继续阅读！

# 介绍 Auth0

自己管理身份验证和授权可能变得非常困难。想象一下，你需要为 Web 应用程序、移动应用程序和桌面应用程序实现安全逻辑。甚至你的客户可能要求你将他们的应用程序集成到社交网络中，并使用多因素身份验证或无密码方法。尽管我们已经为我们的应用程序实现了安全措施，但我们鼓励你除非你正在创建一个非常简单的应用程序，否则不要自己编写安全代码。

因此，在本节中，我们将使用一个名为 Auth0（[`auth0.com`](https://auth0.com)）的流行服务来实现身份验证和授权。此服务将帮助我们增强我们的身份验证流程，例如：

+   社交登录

+   单点登录

+   邮箱身份验证

+   多因素

+   无密码身份验证

+   指纹登录

+   LDAP 集成

此外，Auth0 还提供监控和其他开箱即用的服务，这些服务将帮助我们管理我们的用户信息。

# 一个简单的例子

尽管我们为世界杯应用程序使用了自定义实现，但我们已经准备了一个简单的示例来向您展示如何使用 Auth0 与 Aurelia。您可以从 [`github.com/EriksonMurrugarra/AureliaAuth0`](https://github.com/EriksonMurrugarra/AureliaAuth0) 下载代码。

Auth0 实现了 JWT；这应该让您想起，因为我们使用 JWT 实现了我们的自定义 Auth0 实现。让我们首先在 Auth0 上创建一个免费账户。

# 创建账户

首先，导航到官方 Auth0 网站，[`auth0.com`](https://auth0.com)，并点击“注册”按钮：

![图片](img/64ab4512-aa22-4725-940a-c9e9ed7db6cf.png)

然后，填写表单中的电子邮件和密码，并点击“注册”按钮或使用您的社交网络账户：

![图片](img/a432796a-b2b4-4dc3-b9f1-ef532e3b613b.png)

一旦完成注册过程，您应该会被重定向到您的管理仪表板。管理仪表板将允许您在几秒钟内配置可以实施的不同安全机制。让我们继续注册一个应用程序以生成一些有效的配置值，以便将我们的 Aurelia 应用程序连接到 Auth0。

# 注册 Auth0 客户端应用程序

如果您有与第三方服务提供商合作的经验，您可能会注意到，为了使用他们的服务，您必须注册一个应用程序以获取一些私钥，您将使用这些私钥来访问第三方提供商的资源。Auth0 也是如此；我们应该首先注册一个应用程序，然后使用生成的密钥来配置我们的应用程序。

在仪表板页面上，继续选择“应用程序”菜单并点击“创建客户端”按钮：

![图片](img/0af642dc-d0cd-41c3-bb85-9b262c9d6775.png)

点击“创建客户端”按钮后，填写以下表单中的应用程序名称，并选择客户端类型为单页应用程序；然后，点击“创建”：

![图片](img/a3539d94-cb2d-41d1-8b4d-5c5b79383d9a.png)

创建客户端后，将显示一个新的配置页面。导航到“设置”选项卡，您将看到以下配置值：

+   名称：我们应用程序的名称

+   域名：您在注册过程中之前注册的域名

+   客户端 ID：一个独特的 ID，使您的应用程序独一无二

+   客户端密钥：用于签署 Auth0 将生成的 JWT 的密钥值

+   允许的回调 URL：当身份验证成功时，Auth0 将重定向到的 URL 列表

就这些。在我们探索应用程序的代码之前，让我们通过分析以下图表来了解 Auth0 如何管理身份验证：

![图片](img/bcee34dc-5eab-475e-8c88-d95bf08b82a9.png)

流程从用户想要登录应用程序时开始。你可能有一个导航栏，其中有一个按钮，点击后会触发一个 JavaScript 函数，该函数将调用 Auth0 JavaScript 登录函数。其次，用户将被重定向到内置的 Auth0 登录表单，并需要输入他们的凭据以进行注册。用户输入的凭据将由 Auth0 验证；如果 Auth0 找到具有提供的凭据的有效用户，它将生成一个有效的 JWT，并将其发送到用户/Aurelia 应用程序。此 JWT 将用于访问你的后端资源。请记住，你可以在后端使用客户端密钥属性来解密令牌。

# 探索我们的示例应用程序

首先，我们需要从 GitHub 仓库 [`github.com/EriksonMurrugarra/AureliaAuth0`](https://github.com/EriksonMurrugarra/AureliaAuth) 下载源代码。让我们在你的首选文件夹中打开一个终端窗口，并运行以下命令：

```js
$ cd /some/path
$ git clone https://github.com/EriksonMurrugarra/AureliaAuth0
```

下载源代码后，我们需要安装依赖项并运行应用程序。让我们进入源代码文件夹，并执行以下命令：

```js
$ cd AureliaAuth0
$ npm install
...
$ au run --watch

Writing app-bundle.js...
Writing vendor-bundle.js...
Finished 'writeBundles'
Application Available At: http://localhost:9000
BrowserSync Available At: http://localhost:3001
```

让我们打开一个新的浏览器，并导航到 [`localhost:9000`](http://localhost:9000)，这将显示一个简单的首页，在导航栏中有一个登录选项：

![图片](img/e415a435-9158-4cda-b22b-36ced3142b62.png)

太棒了！我们的应用程序已经启动并运行，但在成功登录之前，我们首先需要对其进行配置。请打开位于 `src` 文件夹中的以下 `auth-service.js` 文件。我们需要替换我们在 Auth0 上创建客户端应用程序时获得的配置值。在我们的例子中，这些值如下：

```js
...  
auth0 = new auth0.WebAuth({
    domain: 'eriksonmurrugarra.auth0.com',
 clientID: 'LBmldq5O0XHPYz4SAyMr03ThgfMOiHs7',
 redirectUri: 'http://localhost:9000/callback',
 audience: 'https://eriksonmurrugarra.auth0.com/userinfo',
 responseType: 'token id_token',
 scope: 'openid'
  })
...
```

需要强调的是，`redirectUri` 应该在 Auth0 的应用程序设置中的允许回调 URL 中注册。

太棒了！保存更改并点击 LOG IN 按钮，将被重定向到以下页面：

![图片](img/e95aa9bc-bfd6-48be-b0a0-67aa82fc10b0.png)

上一页由 Auth0 提供；你可能注意到 URL 已经更改。这意味着每次我们的用户被要求登录时，他们都会被重定向到 Auth0。由于你还没有创建任何用户账户，你首先需要注册。完成注册过程后，你将被重定向到主页：

![图片](img/77b34e64-3aaa-426f-b3d8-1b9f0df94bda.png)

就这些了！现在你已经使用更安全的策略登录到应用程序。请记住，你永远不应该自己实现身份验证和授权。最佳实践是使用第三方服务。创建 Auth0 等服务的开发者肯定有多年创建比你自己更安全的身份验证和授权机制的经验。

# 使用 Auth0 进行社交登录

当人们面对任何注册表单时，背后有一个重要的真理。如果你的用户在他们的最喜欢的社交网络中已经填写了个人信息，而你让他们浪费有效时间再次填写这些信息，他们中的大多数都会讨厌你的应用程序。即使你试图为你的注册表单创造最佳的用户体验，他们也会避免使用它。那么，我们如何让我们的用户开心，并避免他们不得不进行这个可怕的注册过程呢？社交登录集成来拯救！

如果你需要将多个社交网络集成到你的应用程序中，实现社交登录可能是一项重复性的工作。为什么不使用一个现有的服务来帮助我们完成这个过程呢？Auth0 来拯救！

让我们在 3 分钟内实现社交登录。是的！你没看错，只需 3 分钟。请继续，导航到你的 Auth0 仪表板[`manage.auth0.com`](https://manage.auth0.com)，然后导航到“连接/社交”菜单。接下来，激活你想要集成到认证流程中的社交网络，如下所示：

![图片](img/d1724f15-8f04-49f5-96fa-32dafa5b6107.png)

如你所见，我已经为认证流程启用了 Google 和 Facebook。记住，你首先需要在 Facebook 和 Twitter 上注册一个应用程序，然后使用生成的密钥来配置社交登录方法。

我们几乎完成了。为了完成这个过程，我们只需要告诉我们的客户端应用程序我们想要启用社交登录。为此，让我们导航到“客户端”菜单，进入 MyAuth0App 客户端应用程序。然后，导航到“连接”标签，在社交部分启用 Facebook 和 Twitter，如下所示：

![图片](img/c7cc56ad-644e-4a5e-827d-d51b1ebda387.png)

一旦我们配置好了一切，下次当你的用户尝试登录应用程序时，他们将看到以下表单：

![图片](img/e54f530f-d07e-4f6f-8c57-b40d3dc51c1c.png)

太棒了！现在我们知道了如何集成我们的应用程序，让我们的用户能够登录我们的应用程序，并使用他们最喜欢的社交网络账户。

# 单点登录

如果你正在实施一个由不同分布式应用程序组成的大型企业解决方案，这些应用程序需要认证和授权，但需要使用相同的用户数据库，你将需要实现不同的流程来管理所有这些独立应用程序的认证。这种机制被称为**单点登录**（**SSO**），它基本上会在你的任何应用程序中要求登录一次，并将重用生成的相同令牌在所有应用程序中。以下图表解释了这个流程：

![图片](img/b8322976-0b08-43a9-a2c9-48f6004c0048.png)

在前面的示意图中，有三个应用程序。让我们假设这三个不同的应用程序是由同一家公司开发的，员工使用这三个应用程序。想象一下，为了访问每个应用程序，员工必须使用不同的凭证登录到每个应用程序，或者他们可以选择为三个应用程序使用相同的用户名和密码。

如果这些应用程序共享相同用户信息，为什么我们的用户还需要再次登录到另一个应用程序呢？首先，用户将登录到服务器并检索一个有效的令牌。一旦第一个应用程序登录，它就可以将令牌作为 cookies 或存储在浏览器的`LocalStorage`中。当用户访问应用程序 02 时，应用应该检测到存在一个现有的令牌，并应该使用它来访问服务器，而无需请求凭证。

现在你已经了解了 SSO 的工作原理，你可以自由地自己实现 SSO 或使用外部服务。Auth0 对 SSO 提供了出色的支持。

# 摘要

在本章中，我们创建了一个自定义实现来管理认证和授权，以保护我们的 API 免受未经授权用户的使用。你已经看到，自己实现 Auth0 可能是一项艰巨的任务，并且需要比我们实现的多出更多的安全层。一个好的做法是在你的项目中使用外部服务来实现认证和授权。我们创建了一个简单的应用程序，它使用了一个最受欢迎的第三方服务，名为 Auth0。

我们还介绍了如何将社交登录集成到我们的应用程序中，但我们使用 Auth0 内置的社交连接功能来实现这一功能。你可以自己实现社交认证，但再次强调，最好将精力集中在你的应用程序登录上，而不是浪费时间实现那些可以迅速通过第三方服务实现的功能。

我们探讨了 SSO 在理论上的工作方式，并了解到它是一个简单的流程，即在所有不同的应用程序中重复使用用户令牌。

就这些！在下一章中，你将学习如何在 Aurelia 应用程序上应用端到端测试。继续阅读！
