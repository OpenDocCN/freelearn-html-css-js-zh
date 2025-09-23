# 第二章.一个健壮的电影 API

我们将构建一个电影 API，允许您将演员和电影信息添加到数据库中，并将演员与电影连接起来，反之亦然。这将利用在第一章中介绍的信息，*构建基本的 Express 网站*，并让您亲身体验 Express.js 能提供什么。在本章中，我们将涵盖以下主题：

+   文件夹结构和组织

+   响应 CRUD 操作

+   使用 Mongoose 进行对象建模

+   生成唯一 ID

+   测试

# 文件夹结构和组织

文件夹结构是一个非常有争议的话题。尽管有许多干净的方式来组织你的项目，但我们将使用以下代码作为我们后续章节的示例：

```js
chapter2
├── app.js
├── package.json ├── node_modules
│└── npm package folders ├── src
│├── lib
│├── models
│├── routes
└── test
```

让我们详细看看：

+   `app.js`：在根目录中通常有一个主要的`app.js`文件。`app.js`是应用程序的入口点，将用于启动服务器。

+   `package.json`：与任何 Node.js 应用程序一样，我们在根目录中都有`package.json`，指定我们的应用程序名称和版本以及所有 npm 依赖项。

+   `node_modules`：`node_modules`文件夹及其内容是通过 npm 安装生成的，通常应该在你的版本控制中忽略，因为它依赖于应用程序运行的平台。话虽如此，根据 npm FAQ，将`node_modules`文件夹提交可能更好。

    ### 注意

    将部署的项目，如网站和应用程序，的`node_modules`检查到 git 中。不要将用于重用的库和模块的`node_modules`检查到 git 中。

    参考以下文章了解更多关于这一做法的原理：

    [`www.futurealoof.com/posts/nodemodules-in-git.html`](http://www.futurealoof.com/posts/nodemodules-in-git.html)

+   `src`：`src`文件夹包含应用程序的所有逻辑。

+   `lib`：在`src`文件夹中，我们有`lib`文件夹，它包含应用程序的核心。这包括中间件、路由以及创建数据库连接。

+   `models`：`models`文件夹包含我们的`mongoose`模型，它定义了我们想要操作和保存的模型的结构和逻辑。

+   `routes`：`routes`文件夹包含 API 能够服务的所有端点的代码。

+   `test`：`test`文件夹将包含我们使用 Mocha 以及两个其他 node 模块`should`和`supertest`的功能测试，以使达到 100%覆盖率变得更容易。

# 响应 CRUD 操作

CRUD 术语指的是可以在数据上执行的四项基本操作：创建、读取、更新和删除。Express 通过支持基本方法`GET`、`POST`、`PUT`和`DELETE`，为我们处理这些操作提供了一个简单的方法：

+   `GET`：此方法用于从数据库中检索现有数据。这可以用来从数据库中读取单行或多行（对于 SQL）或文档（对于 MongoDB）。

+   `POST`：此方法用于将新数据写入数据库，并且通常也会包含一个符合数据模型的 JSON 有效负载。

+   `PUT`：此方法用于在数据库中更新现有数据，并且通常也会包含一个符合数据模型的 JSON 有效负载。

+   `DELETE`：此方法用于从数据库中删除现有行或文档。

Express 4 与版本 3 相比发生了巨大变化。为了使其更加轻量级和减少依赖，许多核心模块已被移除。因此，我们需要在需要时显式地 `require` 模块。

一个有用的模块是 `body-parser`。它允许我们在接收到 `POST` 或 `PUT` HTTP 请求时获取一个格式良好的体。我们必须在业务逻辑之前添加此中间件，以便稍后使用其结果。我们在 `src/lib/parser.js` 中写入以下内容：

```js
var bodyParser = require('body-parser');
module;exports = function(app) {
  app.use(bodyParser.json());
  app.use(bodyParser.urlencoded({ extended: false }));
   };
```

上述代码随后被用于 `src/lib/app.js` 中，如下所示：

```js
var express = require('express'); var app = express();
require('./parser')(app);
module.exports = app;
```

以下示例允许您对 `http://host/path` 上的 `GET` 请求做出响应。一旦请求击中我们的 API，Express 将运行它通过必要的中间件以及以下函数：

```js
app.get('/path/:id', function(req, res, next) {
res.status(200).json({ hello: 'world'});
});
```

第一个参数是我们想要处理的 `GET` 函数的路径。路径可以包含以 `:` 为前缀的参数。这些路径参数随后将在请求对象中解析。

第二个参数是当服务器收到请求时将执行的回调。此函数包含三个参数：`req`、`res` 和 `next`。

`req` 参数代表由 Express 和我们在应用程序中添加的中间件定制的 HTTP 请求对象。使用路径 `http://host/path/:id`，假设向 `http://host/path/1?a=1&b=2` 发送了一个 `GET` 请求。`req` 对象将是以下内容：

```js
{
params: { id: 1 }, query: { a: 1, b: 2 }
}
```

`params` 对象是路径参数的表示。`query` 是查询字符串，它是 URL 中 `?` 后的值。在 `POST` 请求中，我们的请求对象中通常还会有一个体，其中包含我们希望放入数据库中的数据。

`res` 参数代表该请求的响应对象。一些方法，如 `status()` 或 `json()`，被提供以便告诉 Express 如何响应用户。

最后，`next()` 函数将执行我们应用程序中定义的下一个中间件。

## 使用 GET 检索演员

从数据库中检索电影或演员包括向路由 `/movies/:id` 或 `/actors/:id` 提交一个 `GET` 请求。我们需要一个唯一的 ID 来引用一个唯一的电影或演员：

```js
app.get('/actors/:id', function(req, res, next) {
//Find the actor object with this :id
//Respond to the client
});
```

在这里，URL 参数 `:id` 将被放置在我们的请求对象中。由于我们在回调函数中之前将第一个变量命名为 `req`，因此我们可以通过调用 `req.params.id` 来访问 URL 参数。

由于一个演员可能参演多部电影，而一部电影可能有多个演员，因此我们需要一个嵌套端点来反映这一点：

```js
app.get('/actors/:id/movies', function(req, res, next) {
//Find all movies the actor with this :id is in
//Respond to the client
});
```

如果提交了一个坏的`GET`请求或者没有找到指定 ID 的演员，那么将返回适当的错误状态码`400`（错误请求）或`404`（未找到）。如果找到了演员，则将发送成功请求`200`，并附带演员信息。成功时，响应的 JSON 将如下所示：

```js
{
"_id": "551322589911fefa1f656cc5", "id": 1,
"name": "AxiomZen", "birth_year": 2012, "__v": 0, "movies": []
}
```

## 使用 POST 创建新的演员

在我们的 API 中，在数据库中创建一个新的电影涉及向`/movies`或为新的演员向`/actors`提交一个`POST`请求：

```js
app.post('/actors', function(req, res, next) {
//Save new actor
//Respond to the client
});
```

在这个例子中，访问我们的 API 的用户发送一个包含将被放置到`request.body`中的数据的`POST`请求。在这里，我们将回调函数中的第一个变量称为`req`。因此，要访问请求体，我们调用`req.body`。

请求体以 JSON 字符串的形式发送；如果发生错误，将返回`400`（错误请求）状态。否则，将向响应对象发送`201`（已创建）状态。在成功请求的情况下，响应将如下所示：

```js
{
"__v": 0, "id": 1,
"name": "AxiomZen", "birth_year": 2012,
"_id": "551322589911fefa1f656cc5", "movies": []
}
```

## 使用 PUT 更新演员

要更新电影或演员条目，我们首先创建一个新的路由，并向`/movies/:id`或`/actors/:id`提交一个`PUT`请求，其中`id`参数是现有`movie/actor`的唯一标识。更新有两个步骤。我们首先使用唯一的 ID 找到电影或演员，然后使用请求对象的主体更新该条目，如下面的代码所示：

```js
app.put('/actors/:id', function(req, res) {
//Find and update the actor with this :id
//Respond to the client
});
```

在请求中，我们需要`request.body`是一个 JSON 对象，它反映了要更新的演员字段。`request.params.id`仍然是一个唯一的标识符，它引用数据库中现有的演员，就像之前一样。在成功更新后，响应的 JSON 看起来如下所示：

```js
{
"_id": "551322589911fefa1f656cc5",
"id": 1,
"name": "Axiomzen", "birth_year": 99, "__v": 0, "movies": []
}
```

这里，响应将反映我们对数据所做的更改。

## 使用 DELETE 删除演员

删除电影就像提交一个`DELETE`请求到之前使用的相同路由（指定 ID）一样简单。找到具有适当 ID 的演员，然后将其删除：

```js
app.delete('/actors/:id', function(req, res) {
//Remove the actor with this :id
//Respond to the client
});
```

如果找到具有唯一 ID 的演员，则将其删除，并返回响应代码`204`。如果演员找不到，则返回响应代码`400`。对于`DELETE()`方法没有响应体；它将简单地返回成功删除的状态代码`204`。

我们这个简单应用的最终端点将如下所示：

```js
//Actor endpoints 
app.get('/actors', actors.getAll);
app.post('/actors', actors.createOne); 
app.get('/actors/:id', actors.getOne); 
app.put('/actors/:id', actors.updateOne); 
app.delete('/actors/:id', actors.deleteOne) 
app.post('/actors/:id/movies', actors.addMovie); 
app.delete('/actors/:id/movies/:mid', actors.deleteMovie);
//Movie endpoints
app.get('/movies', movies.getAll); 
app.post('/movies', movies.createOne);
app.get('/movies/:id', movies.getOne); 
app.put('/movies/:id', movies.updateOne); 
app.delete('/movies/:id', movies.deleteOne); 
app.post('/movies/:id/actors', movies.addActor); 
app.delete('/movies/:id/actors/:aid', movies.deleteActor);
```

在 Express 4 中，有一种描述路由的替代方法。具有相同 URL 但使用不同 HTTP 动词的路由可以如下分组：

```js
app.route('/actors')
.get(actors.getAll)
.post(actors.createOne);
app.route('/actors/:id')
.get(actors.getOne)
.put(actors.updateOne)
.delete(actors.deleteOne);
app.post('/actors/:id/movies', actors.addMovie); 
app.delete('/actors/:id/movies/:mid', actors.deleteMovie);
app.route('/movies')
.get(movies.getAll)
.post(movies.createOne);
app.route('/movies/:id')
.get(movies.getOne)
.put(movies.updateOne)
.delete(movies.deleteOne);
app.post('/movies/:id/actors', movies.addActor); 
app.delete('/movies/:id/actors/:aid', movies.deleteActor);
```

无论你是否喜欢这种方式，这取决于你。至少现在你有选择！

我们还没有讨论每个端点运行函数的逻辑。我们很快就会讨论这个问题。

Express 允许我们轻松地 CRUD 我们的数据库对象，但我们如何对对象进行建模呢？

# 使用 Mongoose 进行对象建模

Mongoose 是一个对象数据建模库（ODM），允许你为你的数据集合定义模式。你可以在项目网站上了解更多关于 Mongoose 的信息：[`mongoosejs.com/`](http://mongoosejs.com/)。

要使用`mongoose`变量连接到 MongoDB 实例，我们首先需要安装 npm 并保存 Mongoose。`save`标志会自动将模块添加到你的`package.json`中，并使用最新版本，因此，始终建议使用`save`标志安装你的模块。对于你只需要本地使用的模块（例如，Mocha），你可以使用`savedev`标志。

对于这个项目，我们在`/src/lib/db.js`下创建了一个新的文件`db.js`，它需要 Mongoose。在`mongoose.connect`中建立到`mongodb`数据库的本地连接，如下所示：

```js
var mongoose = require('mongoose');
module.exports = function(app)
{ 
  mongoose.connect('mongodb://localhost/movies', {
  mongoose: { safe: true
}
}, function(err) { if (err) 
{
  return console.log('Mongoose - connection error:', err);
}
});
return mongoose; 
};
```

在我们的电影数据库中，我们需要为演员和电影分别创建模式。作为一个例子，我们将通过在演员数据库`/src/models/actor.js`中创建演员模式来遍历对象建模，如下所示：

```js
// /src/models/actor.js
var mongoose = require('mongoose');
var generateId = require('./plugins/generateId');

var actorSchema = new mongoose.Schema({ 
  id: {
    type: Number, 
    required: true, 
    index: {
      unique: true
    }
  },
  name: {
    type: String, 
    required: true
  }, 
  birth_year: {
    type: Number, 
    required: true

  }, 
  movies: [{
    type : mongoose.Schema.ObjectId,
    ref : 'Movie'
  }]
});
actorSchema.plugin(generateId());
module.exports = mongoose.model('Actor', actorSchema);
```

每个演员都有一个唯一的 id、一个名字和一个出生年份。条目还包含所需的有效验证器，如类型和布尔值。模型在定义后导出（`module.exports`），这样我们就可以直接在应用中重用它。

或者，你也可以通过 Mongoose 使用`mongoose.model('Actor', actorSchema)`来获取每个模型，但这与我们的直接要求它的方法相比，感觉不那么明确耦合。

类似地，我们还需要一个电影模式。我们定义电影模式如下：

```js
// /src/models/movies.js
var movieSchema = new mongoose.Schema({ 
  id: {
    type: Number, 
    required: true, 
    index: {
      unique: true
    }
  }, 
  title: {
    type: String, 
    required: true
   }, 
   year: {
     type: Number, 
     required: true
   }, 
   actors: [{
     type : mongoose.Schema.ObjectId, 
     ref : 'Actor'
   }]
});

movieSchema.plugin(generateId());
module.exports = mongoose.model('Movie', movieSchema);
```

# 生成唯一 ID

在我们的电影和演员模式中，我们使用了一个名为`generateId()`的插件。

虽然 MongoDB 自动使用`_id`字段为每个文档生成`ObjectID`，但我们想生成更易于阅读的 ID，因此更友好。我们还希望给用户选择他们自己的 id 的机会。

然而，能够选择一个 id 可能会导致冲突。如果你选择了一个已经存在的 id，你的`POST`请求将被拒绝。如果用户没有明确传递 id，我们应该自动生成一个 ID。

如果没有这个插件，如果用户没有明确传递 ID，无论是创建演员还是电影，服务器都会抱怨，因为 ID 是必需的。

我们可以为 Mongoose 创建中间件，在持久化对象之前分配一个`id`，如下所示：

```js
// /src/models/plugins/generateId.js 
module.exports = function() {

return function generateId(schema){ 
  schema.pre('validate',function(next, done) {
    var instance = this; 
    var model = instance.model(instance.constructor.modelName); 

    if( instance.id == null ) {
     model.findOne().sort("-id").exec(function(err,maxInstance) {
       if (err){
         return done(err);
       } else {
         var maxId = maxInstance.id || 0; 
         instance.id = maxId+1;
         done();
       }
    })
   } else { 
     done();
    }
  })
 }
};
```

关于此代码有几个重要的注意事项。

看看我们是如何得到`var`模型的？这使得插件通用，因此它可以应用于多个 Mongoose 模式。

注意，有两个回调可用：`next`和`done`。`next`变量将代码传递给下一个预验证中间件。这通常是好事，因为异步调用的一项优点是你可以同时运行许多事情。

然而，在这种情况下，我们不能调用`next`变量，因为它会与我们的 id 模式定义冲突。因此，当逻辑完成时，我们只使用`done`变量。

由于 MongoDB 不支持事务，因此可能会出现一些边缘情况导致此函数失败。例如，如果同时发生两个对 `POST` `/actor` 的调用，它们都将自动递增到相同的 ID 值。

现在我们有了 `generateId()` 插件的代码，我们按照以下方式在我们的演员和电影模式中引入它：

```js
var generateId = require('./plugins/generateId');
actorSchema.plugin(generateId());
```

# 验证您的数据库

Mongoose 模式中每个键都定义了一个与 SchemaType 关联的属性。例如，在我们的 `actors.js` 模式中，演员的姓名键与字符串 SchemaType 关联。字符串、数字、日期、缓冲区、布尔值、混合、objectId 和数组都是有效的模式类型。

除了模式类型之外，数字有最小和最大验证器，字符串有枚举和匹配验证器。验证发生在文档正在保存 `(.save())` 时，如果验证失败，将返回一个包含类型、路径和值属性的错误对象。

# 将函数提取为可重用的中间件

我们可以使用我们的匿名或命名函数作为中间件。为此，我们通过在 `routes/actors.js` 和 `routes/movies.js` 中调用 `module.exports` 来导出我们的函数：

让我们看看我们的 `routes/actors.js` 文件。在这个文件的顶部，我们引入了我们之前定义的 Mongoose 模式：

```js
var Actor = require('../models/actor');
```

这允许我们的变量 `actor` 使用 mongo 函数（如 `find()`、`create()` 和 `update()`）访问我们的 MongoDB。它将遵循 `/models/actor` 文件中定义的模式。

由于演员在电影中，我们还需要通过以下方式引入 `Movie` 模式来显示这种关系。

```js
var Movie = require('../models/movie');
```

现在我们有了我们的模式，我们可以开始定义我们在端点中描述的函数的逻辑。例如，端点 `GET` `/actors/:id` 将从我们的数据库中检索具有相应 ID 的演员。让我们称这个函数为 `getOne()`。它定义如下：

```js
getOne: function(req, res, next) { Actor.findOne({ id: req.params.id })
.populate('movies')
.exec(function(err, actor) {
if (err) return res.status(400).json(err); if (!actor) return res.status(404).json(); res.status(200).json(actor);
});
},
```

在这里，我们使用 mongo 的 `findOne()` 方法来检索具有 `id:` `req.params.id` 的演员。MongoDB 中没有连接，所以我们使用 `.populate()` 方法来检索演员参演的电影。

`.populate()` 方法将根据其 `ObjectId` 从单独的集合中检索文档。

如果我们的 Mongoose 驱动程序出现错误，此函数将返回状态 `400`，如果找不到具有 `:id` 的演员，则返回状态 `404`，最后，如果找到演员，它将返回状态 `200` 并附带演员对象的 JSON。

我们在这个文件中定义了所有用于演员端点的函数。结果如下：

```js
// /src/routes/actors.js
var Actor = require('../models/actor'); 
var Movie = require('../models/movie');

module.exports = {

  getAll: function(req, res, next) { 
    Actor.find(function(err, actors) {
      if (err) return res.status(400).json(err);

      res.status(200).json(actors); 
    });
  },

  createOne: function(req, res, next) { 
  Actor.create(req.body, function(err, actor) {
    if (err) return res.status(400).json(err);

    res.status(201).json(actor); 
  });
  },

  getOne: function(req, res, next) { 
    Actor.findOne({ id: req.params.id })
    .populate('movies')
.exec(function(err, actor) {
      if (err) return res.status(400).json(err); 
      if (!actor) return res.status(404).json();

      res.status(200).json(actor);
    });
  },

  updateOne: function(req, res, next) { 
    Actor.findOneAndUpdate({ id: req.params.id }, req.body,function(err, actor) {
      if (err) return res.status(400).json(err); 
      if (!actor) return res.status(404).json();

      res.status(200).json(actor); 
    });
  },

  deleteOne: function(req, res, next) { 
    Actor.findOneAndRemove({ id: req.params.id }, function(err) {
      if (err) return res.status(400).json(err);

      res.status(204).json(); 
    });
  },

  addMovie: function(req, res, next) {
    Actor.findOne({ id: req.params.id }, function(err, actor) { 
      if (err) return res.status(400).json(err);
      if (!actor) return res.status(404).json();

      Movie.findOne({ id: req.body.id }, function(err, movie) {
        if (err) return res.status(400).json(err);
        if (!movie) return res.status(404).json();

        actor.movies.push(movie); 
        actor.save(function(err) {
          if (err) return res.status(500).json(err);

          res.status(201).json(actor); 
        });
       })
     });
  },

  deleteMovie: function(req, res, next) {
    Actor.findOne({ id: req.params.id }, function(err, actor) { 
      if (err) return res.status(400).json(err);
      if (!actor) return res.status(404).json();

      actor.movies = []; 
      actor.save(function(err) {
        if (err) return res.status(400).json(err);

        res.status(204).json(actor);
      })
    });
   }

  };
```

对于我们所有的电影端点，我们需要相同的函数，但应用于电影集合。

导出这两个文件后，我们在 `app.js` (`/src/lib/app.js`) 中通过简单地添加以下内容来引入它们：

```js
require('../routes/movies'); require('../routes/actors');
```

通过将我们的函数作为可重用的中间件导出，我们保持我们的代码整洁，并且可以在 `/routes` 文件夹中的 CRUD 调用中引用函数。

# 测试

Mocha 被用作测试框架，同时使用 `should.js` 和 supertest。为什么我们在应用程序中使用测试以及 Mocha 的基础知识在 第一章 *Building a Basic Express Site* 中进行了介绍。使用 supertest 进行测试可以让你测试你的 HTTP 断言和 API 端点。

测试被放置在根目录 `/test` 中。测试与任何源代码完全分离，并编写为纯英文可读，也就是说，你应该能够通过阅读它们来了解正在测试的内容。编写良好的测试用例，具有良好的覆盖率，可以作为其 API 的说明文档，因为它清楚地描述了整个应用程序的行为。

测试我们的电影 API 的初始设置对 `/test/actors.js` 和 `/test/movies.js` 都是相同的，如果你已经阅读了 第一章 *Building a Basic Express Site*，那么这应该看起来很熟悉。

```js
var should = require('should'); var assert = require('assert');
var request = require('supertest');
var app = require('../src/lib/app');
```

在 `src/test/actors.js` 中，我们测试基本的 CRUD 操作：创建新的演员对象、检索、编辑和删除演员对象。以下是一个创建新演员的示例测试：

```js
  describe('Actors', function() { 

  describe('POST actor', function(){
    it('should create an actor', function(done){ 
      var actor = {
        'id': '1',
        'name': 'AxiomZen', 'birth_year': '2012',
       };

       request(app)
       .post('/actors')
       .send(actor)
       .expect(201, done)
    });
```

我们可以看到，测试用例可以用纯英文阅读。我们为数据库中的新演员创建一个新的 `POST` 请求，该演员的 `id` 为 `1`，`name` 为 `AxiomZen`，`birth_year` 为 `2012`。然后，我们使用 `.send()` 函数发送请求。以下代码中给出了 `GET` 和 `DELETE` 请求的类似测试。

```js
  describe('GET actor', function() {
    it('should retrieve actor from db', function(done){ 
      request(app)
      .get('/actors/1')
      .expect(200, done);
    });
  describe('DELETE actor', function() {
    it('should remove a actor', function(done) { 
      request(app)
      .delete('/actors/1')
     .expect(204, done);
    });
  });
```

要测试我们的 `PUT` 请求，我们将编辑第一个演员的 `name` 和 `birth_year`，如下所示：

```js
  describe('PUT actor', function() {
    it('should edit an actor', function(done) { 
      var actor = {
        'name': 'ZenAxiom', 
        'birth_year': '2011'
      };

      request(app)
      .put('/actors/1')
      .send(actor)
      .expect(200, done);
    });

    it('should have been edited', function(done) { 
      request(app)
      .get('/actors/1')
      .expect(200)
      .end(function(err, res) { 
        res.body.name.should.eql('ZenAxiom');
        res.body.birth_year.should.eql(2011);
        done();
      });
     });
  });
```

测试的第一部分修改了演员的 `name` 和 `birth_year` 键，向 `/actors/1` 发送 `PUT` 请求（`1` 是演员的 `id`），然后将新信息保存到数据库中。测试的第二部分检查具有 `id` `1` 的演员的数据库条目是否已更改。使用 `.should.eql()` 检查 `name` 和 `birth_year` 的值是否与预期值相符。

除了对演员对象执行 CRUD 操作外，我们还可以对每个演员添加的电影（通过演员的 ID 关联）执行这些操作。以下代码片段展示了向我们的第一个演员（`id` 为 `1`）添加新电影的测试。

```js
  describe('POST /actors/:id/movies', function() { 
    it('should successfully add a movie to the actor',function(done) { 
      var movie = {
        'id': '1',
        'title': 'Hello World', 
        'year': '2013'
      }
      request(app)
      .post('/actors/1/movies')
      .send(movie)
      .expect(201, done)
      });
    });

    it('actor should have array of movies now', function(done){ 
      request(app)
      .get('/actors/1')
      .expect(200)
      .end(function(err, res) { 
      res.body.movies.should.eql(['1']); 
      done();
     });
    });
  });
```

测试的第一部分创建了一个新的电影对象，包含 `id`、`title` 和 `year` 键，并向具有 `id` 为 `1` 的演员发送一个 `POST` 请求，将电影数组添加到该演员。测试的第二部分发送一个 `GET` 请求以检索具有 `id` 为 `1` 的演员，此时应该包含一个包含新电影输入的数组。

我们可以类似地删除 `actors.js` 测试文件中的电影条目：

```js
  describe('DELETE /actors/:id/movies/:movie_id', function() { 
    it('should successfully remove a movie from actor', function(done){
      request(app)
      .delete('/actors/1/movies/1')
      .expect(200, done);
    });

    it('actor should no longer have that movie id', function(done){
      request(app)
      .get('/actors/1')
      .expect(201)
      .end(function(err, res) { 
        res.body.movies.should.eql([]); 
        done();
      });
    });
  });
```

再次强调，这个代码片段应该对您来说很熟悉。第一部分测试发送指定演员 ID 和电影 ID 的 `DELETE` 请求将删除该电影条目。在第二部分中，我们通过提交一个 `GET` 请求来查看演员的详细信息（不应列出任何电影），以确保该条目不再存在。

除了确保基本的 CRUD 操作正常工作外，我们还测试了我们的模式验证。以下代码测试确保没有两个具有相同 ID 的演员存在（ID 被指定为唯一的）：

```js
  it('should not allow you to create duplicate actors', function(done) {
    var actor = { 
      'id': '1',
      'name': 'AxiomZen', 
      'birth_year': '2012',
    };

    request(app)
    .post('/actors')
    .send(actor)
    .expect(400, done);
  });
```

如果我们尝试创建一个已在数据库中存在的演员，我们应该期望代码返回 `400`（错误请求）。

对于 `tests/movies.js`，也存在类似的测试集。每个测试的功能和结果现在应该很清楚。

# 摘要

在本章中，我们创建了一个基本的 API，该 API 连接到 MongoDB 并支持 CRUD 方法。现在您应该能够为任何数据（而不仅仅是电影和演员）设置一个包含测试的完整 API！

聪明的读者会注意到，我们尚未在本章中解决一些问题，例如在 MongoDB 中处理竞争条件。这些问题将在接下来的章节中详细说明。

我们希望您发现这一章节为 Express 和 API 设置奠定了良好的基础。
