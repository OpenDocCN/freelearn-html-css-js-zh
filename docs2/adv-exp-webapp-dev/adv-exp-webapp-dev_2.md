# 第二章：构建 Web API

在打下基础后，我们开始为我们的 Vision 项目构建 Web API 的过程。我们将首先使用 MongoDB 设置持久层。然后，我们将逐个实现 Web API 的各个方面。

# 使用 MongoDB 和 Mongoose 持久化数据

MongoDB 是一个开源的面向文档的数据库系统。MongoDB 存储结构化数据，如类似 JSON 的文档，简化了集成。

让我们首先为我们的项目创建一个 MongoDB 模式。该模式包含一些与项目相关的基本信息，例如项目的名称、GitHub 访问令牌、用户和存储库列表。

让我们安装 Mongoose，它是 Node.js 的 MongoDB 对象文档映射器；它提供了一个基于模式的解决方案来建模您的数据。

```js
npm install mongoose --save

```

让我们配置我们的应用程序以使用 MongoDB 和 Mongoose；我们在配置文件 `./lib/config/*.js` 中添加 MongoDB 的 URL：

```js
{
  "express": {
    "port": 3000
  },
  "logger" : {
    "filename": "logs/run.log",
    "level": "silly"
  },
 "mongo": {
 "url":  "mongodb://localhost/vision"
 }
}
```

让我们创建一个 MongoDB 连接模块，`./lib/db/index.js`，它只是从我们的 Winston 配置中获取 MongoDB URL 并打开一个连接：

```js
var mongoose = require('mongoose')
, config = require('../configuration')
, connectionString = config.get("mongo:url")
, options = { server: { auto_reconnect: true, poolSize: 10 } };

mongoose.connection.open(connectionString, options);
```

我们现在创建一个模型类 `./lib/models/index.js`，它定义了我们的 `ProjectSchema`：

```js
var mongoose = require('mongoose'),
    Schema = mongoose.Schema;

var ProjectSchema = new Schema({
    name         : { type: String, required: true, index: true }
  , token        : { type: String }
  , user         : { type: String, required: true, index: true }
  , created      : { type: Date, default: Date.now }
  , repositories : [ { type: String } ]
});

mongoose.model('Project', ProjectSchema);
module.exports = mongoose;
```

为了运行以下示例，我们需要一个运行的 MongoDB 实例。您可以从 [`www.mongodb.org`](http://www.mongodb.org) 下载 MongoDB。运行以下命令以启动 MongoDB：

```js
mongod

```

# GitHub 令牌

为了获取 GitHub 令牌，请登录您的 GitHub 账户并前往您的 **设置** 页面的 **账户** 部分。在这里，您需要输入您的密码。现在点击 **创建新令牌**，如果您愿意的话，还可以为令牌命名。点击 **复制到剪贴板** 按钮以将令牌复制到下面的 `login` 文件中。

让我们创建一个名为 `login` 的文件——`./test/login.js`——并从 GitHub 获取数据。我们将使用这个文件来调用 GitHub API；在后续章节中，这个文件将被删除。

```js
module.exports = {
  user : '#USER#'
  token : '#TOKEN#'
}
```

# 功能：创建项目

```js
As a vision user
I want to create a new project
So that I can monitor the activity of multiple repositories
```

让我们在现有的测试集中添加一个针对我们的功能 `创建项目` 的测试。这个资源将向 `/project` 路由 POST 一个项目并返回 `201 Created` 状态码。以下测试：`./test/project.js` 是 `201 Created` 测试。

### 小贴士

本书不会记录一个功能的全部测试集。请参阅源代码以获取完整的测试集。

在这个例子中，SuperTest 执行一个返回响应的 `end` 函数；这允许我们检查响应的头和体。

```js
describe('when creating a new resource /project', function(){
  var project = {
    name: "new project"
    , user: login.user  
    , token: login.token
    , repositories    : [ "12345", "9898" ]
  };

  it('should respond with 201', function(done){
    request(app)
    .post('/project')
    .send(project)
    .expect('Content-Type', /json/)
    .expect(201)
    .end(function (err, res) {
      var proj = JSON.parse(res.text);
      assert.equal(proj.name, project.name);
      assert.equal(proj.user, login.user);
      assert.equal(proj.token, login.token);
      assert.equal(proj.repositories[0], project.repositories[0]);
      assert.equal(proj.repositories[1], project.repositories[1]);
      assert.equal(res.header['location'],'/project/' + proj._id);
      done();
      });
    });
  });
```

为了使一些测试能够运行，我们需要一些测试数据。因此，以下 `./test/project.js` 将使用 Mocha 的 `beforeEach` 钩子销毁任何现有的项目数据，并添加一个新的项目：

```js
beforeEach(function(done){
  mongoose.connection.collections['projects'].drop( function(err) {
  var proj = {
    name: "test name"
    , user: login.user  
    , token: login.token
    , repositories    : [ "node-plates" ]
  };

  mongoose.connection.collections['projects'].insert(proj,function(err, docs) {
      id = docs[0]._id;
      done();
    });
  });
})
```

让我们安装 `string.js`，这是一个轻量级的 JavaScript 库，它提供了额外的字符串方法。这将帮助我们验证请求：

```js
npm install string --save

```

让我们实现“创建项目”功能。我们首先创建一个`Project`模块`./lib/project/index.js`。我们导入一个 Mongoose 模式用于`Project`模型，并定义一个名为`post`的函数，该函数接受`name`和`data`作为参数。我们调用静态函数`Project.findOne`来检查项目是否存在，如果项目是唯一的，我们调用`project.save`函数来保存项目。

```js
var ProjectSchema = require('../models').model('Project'); 

function Project() {};

Project.prototype.post = function(name, data, callback){
  var query = {'name': name};
  var project = new ProjectSchema(data);

  ProjectSchema.findOne(query, function(error, proj) {
    if (error) return callback(error, null);
    if (proj != null) return callback(null, null);

    project.save(function (error, p) {
      if (error) return callback(error, null);
      return callback(null, p);
    });
  });
};
```

让我们在`./lib/routes/project.js`中添加一个新的路由。我们导入一个`logger`变量、一个`ProjectService`模块，并定义一个名为`Post`的路由，该路由使用`req.body`来访问请求中 POST 的项目项。然后我们验证请求，如果请求无效则返回`400 Bad Request`。如果请求有效，我们将用户和令牌添加到请求体中并调用`Project.post`；如果发生错误，我们返回`500 Internal Server Error`，如果项目已存在，我们返回`409 Conflict`响应。如果请求正常，我们在响应上设置`res.location`以指向我们的新资源，并返回`201 Created`响应：

```js
var logger = require("../logger")
, S = require('string')
, login = require('../../test/login')
, ProjectService = require('../project')
, Project = new ProjectService();

exports.post = function(req, res){
  logger.info('Post.' + req.body.name);

  if (S(req.body.name).isEmpty() )
  return res.json(400, 'Bad Request');

  req.body.user = login.user;
  req.body.token = login.token;

  Project.post(req.body.name, req.body, function(error, project) {
    if (error) return res.json(500, 'Internal Server Error');
    if (project == null) return res.json(409, 'Conflict');
    res.location('/project/' +  project._id);
    return res.json(201, project);
  });
};
```

为了添加我们的新路由并允许我们的应用程序支持 HTTP POST，我们需要对我们的 Express 服务器`./lib/express/index.js`进行一些修改。

首先，我们导入本章开头创建的`db`模块，该模块打开到 MongoDB 数据库的连接。然后，我们导入刚刚创建的`project`路由模块。重要的是，`app.use(express.bodyParser())`在表单提交时解析请求体。`bodyParser`中间件支持`application/x-www-form-urlencoded`、`application/json`和`multipart/form-data`。我们在`/project`路径下添加了一个新的路由用于发布项目。

```js
var express = require('express')
  , http = require('http')
  , config = require('../configuration')
 , db = require('../db')
  , heartbeat = require('../routes/heartbeat')
 , project = require('../routes/project')
  , error = require('../routes/error')
  , notFound = require('../middleware/notFound')
  , app = express();

app.use(express.bodyParser());
app.set('port', config.get('express:port'));
app.use(express.logger({ immediate: true, format: 'dev' }));
app.get('/heartbeat', heartbeat.index);
app.post('/project', project.post);
app.use(notFound.index);

http.createServer(app).listen(app.get('port'));
module.exports = app;
```

# 功能：获取项目

```js
As a vision user
I want to get a project
So that I can monitor the activity of selected repositories
```

让我们在现有的测试集`./test/project.js`中为我们的“获取项目”功能添加一个测试。这个资源将 GET 从路由`/project/:id`获取一个项目，并返回`200 OK`状态。

让我们安装`underscore.js`；这是一个提供函数式编程支持的实用工具库：

```js
npm install underscore --save

```

```js
describe('when requesting an available resource /project/:id', function(){
  it('should respond with 200', function(done){
    request(app)
    .get('/project/' + id)
    .expect('Content-Type', /json/)
    .expect(200)
    .end(function (err, res) {
      var proj = JSON.parse(res.text);
      assert.equal(proj._id, id);
      assert(_.has(proj, '_id'));
      assert(_.has(proj, 'name'));
      assert(_.has(proj, 'user'));
      assert(_.has(proj, 'token'));
      assert(_.has(proj, 'created'));
      assert(_.has(proj, 'repositories'));
      done();
    });
  });
});
```

让我们实现“获取项目”功能`./lib/project/index.js`并添加一个`get`函数。我们尝试通过调用静态函数`Project.findOne`来检索项目。如果发生错误，我们返回它；如果找到项目，我们返回项目：

```js
Project.prototype.get = function(id, callback){
  var query = {"_id" : id};

  ProjectSchema.findOne(query, function(error, project) {
    if (error) return callback(error, null);
    return callback(null, project);
  });
};
```

让我们在`./lib/routes/project.js`中添加一个新的路由。我们首先定义一个名为`get`的路由。我们使用正则表达式验证请求，以检查有效的 Mongoose `ObjectId`；如果请求无效，则返回`400 Bad Request`状态。我们尝试通过调用`Project.get`并传递`id`来检索项目。如果发生错误，我们返回`500 Internal Server Error`；如果项目不存在，我们返回`404 Not Found`。如果我们找到项目，我们返回项目和一个`200 OK`响应：

```js
exports.get = function(req, res){
  logger.info('Request.' + req.url);

  Project.get(req.params.id, function(error, project) {
    if (error) return res.json(500, 'Internal Server Error');
    if (project == null) return res.json(404, 'Not Found');
    return res.json(200, project);
  });
};
```

现在将以下路由添加到`./lib/express/index.js`中：

```js
app.get('/project/:id', project.get);
```

# 功能：编辑项目

```js
As a vision user
I want to update a project
So that I can change the repositories I monitor
```

让我们在现有的测试集 `./test/project.js` 中为我们的 `编辑一个项目` 功能添加一个测试。此资源将向路由 `/project/:id` 发送一个项目，并返回 `204 No Content` 状态：

```js
describe('when updating an existing resource /project/:id', function(){
  var project = {
    name: "new test name"
    , user: login.user  
    , token: login.token
    , repositories    : [ "12345", "9898" ]
  };

  it('should respond with 204', function(done){

    request(app)
    .put('/project/' + id)
    .send(project)
    .expect(204, done);
  });
});
```

让我们实现 `编辑一个项目` 功能 `./lib/project/index.js` 并添加一个 `put` 函数。我们尝试通过调用静态函数 `Project.findOne` 来检索项目。如果发生错误，我们返回错误；如果我们找不到项目，我们返回 `null`。如果我们找到项目，我们更新它并返回项目：

```js
Project.prototype.put = function(id, update, callback){
  var query = {"_id": id};
  delete update._id;

  ProjectSchema.findOne(query, function(error, project) {
    if (error) return callback(error, null);
    if (project == null) return callback(null, null);

    ProjectSchema.update(query, update, function(error, project) {
      if (error) return callback(error, null);
      return callback(null, {});
    });
  });
};
```

让我们在 `./lib/routes/project.js` 中添加一个新的路由。我们首先定义一个名为 `put` 的路由，然后通过返回 `400 Bad Request` 来验证请求是否有效。我们在请求体中添加一个登录用户和令牌；这将在后面的章节中删除。我们尝试通过调用 `Project.put` 并传递 `id` 来更新项目。如果发生错误，我们返回 `500 Internal Server Error`；如果项目不存在，我们返回 `404 Not Found` 状态。如果我们找到项目，则返回 `204 No Content` 响应：

```js
exports.put = function(req, res){
  logger.info('Put.' + req.params.id);

  if (S(req.body.name).isEmpty() )
  return res.json(400, 'Bad Request');

  req.body.user = login.user;
  req.body.token = login.token;

  Project.put(req.params.id, req.body, function(error, project) {
    if (error) return res.json(500, 'Internal Server Error');
    if (project == null) return res.json(404, 'Not Found');
    return res.json(204, 'No Content');
  });
};
```

现在，将以下路由添加到 Express 服务器 `./lib/express/index.js` 中：

```js
app.put('/project/:id', project.put);
```

# 功能：删除一个项目

```js
As a vision user
I want to delete a project
So that I can remove projects no longer in use
```

让我们在 `./test/project.js` 中为我们的功能 `删除一个项目` 添加一个测试。此资源将在路由 `/project/:id` 删除一个项目并返回 `204 No Content` 状态：

```js
describe('when deleting an existing resource /project/:id', function(){
  it('should respond with 204', function(done){
    request(app)
    .del('/project/' + id)
    .expect(204, done);
  });
});
```

让我们实现 `删除一个项目` 功能 `./lib/project/index.js` 并添加一个 `del` 函数。我们尝试通过调用静态函数 `Project.findOne` 来删除项目。如果发生错误，我们返回错误；如果我们找不到项目，我们返回 `null`。如果我们找到项目，我们将其删除并返回一个空响应。

```js
Project.prototype.del = function(id, callback){
  var query = {'_id': id};

  ProjectSchema.findOne(query, function(error, project) {
    if (error) return callback(error, null);
    if (project == null) return callback(null, null);

    project.remove(function (error) {
      if (error) return callback(error, null);
      return callback(null, {});
    });
  });
};
```

让我们在 `./lib/routes/project.js` 中添加一个新的路由。我们首先定义一个名为 `del` 的路由。我们尝试通过调用 `Project.del` 并传递 `id` 来删除项目。如果发生错误，我们返回 `500 Internal Server Error`；如果项目不存在，我们返回 `404 Not Found`。如果我们找到项目，我们返回 `204 No Content` 响应。

```js
exports.del = function(req, res){
  logger.info('Delete.' + req.params.id);

  Project.del(req.params.id, function(error, project) {
    if (error) return res.json(500, 'Internal Server Error');
    if (project == null) return res.json(404, 'Not Found');
    return res.json(204, 'No Content');
  });
};
```

现在，将以下路由添加到 Express 服务器 `./lib/express/index.js` 中：

```js
app.del('/project/:id', project.del);
```

# 功能：列出项目

```js
As a vision user
I want to see a list of projects
So that I can select a project I want to monitor
```

让我们在 `./test/project.js` 中为我们的功能 `列出项目` 添加一个测试。此资源将从路由 `/project` 获取所有项目并返回 `200 Ok` 状态。

```js
describe('when requesting resource get all projects', function(){
  it('should respond with 200', function(done){
    request(app)
    .get('/project/?user=' + login.user)
    .expect('Content-Type', /json/)
    .expect(200)
    .end(function (err, res) {
      var proj = _.first(JSON.parse(res.text))
      assert(_.has(proj, '_id'));
      assert(_.has(proj, 'name'));
      assert(_.has(proj, 'user'));
      assert(_.has(proj, 'token'));
      assert(_.has(proj, 'created'));
      assert(_.has(proj, 'repositories'));
      done();
    });
  });
});
```

让我们实现 `列出项目` 功能 `./lib/project/index.js` 并添加一个 `all` 函数。我们尝试通过调用静态函数 `Project.find` 并按用户 `id` 查询来检索所有项目。如果发生错误，我们返回错误；如果找到项目，我们返回项目：

```js
Project.prototype.all = function(id, callback){
  var query = {"user" : id};

  ProjectSchema.find(query, function(error, projects) {
    if (error) return callback(error, null);
    return callback(null, projects);
  });
};
```

让我们在 `./lib/routes/project.js` 中添加一个新的路由。我们首先定义一个名为 `all` 的路由。我们首先检索用户的 `id`。为了适应我们没有实现认证策略的事实，我们从我们的硬编码 `login.user` 对象中获取用户详情。我们将在未来的章节中清理它。我们尝试通过调用 `Project.all` 并传递 `userId` 来检索一个项目。如果发生错误，我们返回 `500 Internal Server Error`；如果我们找到项目，我们返回项目和一个 `200 OK` 响应。

```js
exports.all = function(req, res){
  logger.info('Request.' + req.url);

  var userId = login.user || req.query.user || req.user.id;

  Project.all(userId, function(error, projects) {
    if (error) return res.json(500, 'Internal Server Error');
    if (projects == null) projects = {};
    return res.json(200, projects);
  });
};
```

现在，将以下路由添加到 Express 服务器 `./lib/express/index.js` 中：

```js
app.get('/project', project.all);
```

# GitHub API

我们的项目 API 已经完成，但随着我们尝试与 GitHub API 进行通信，事情将变得更加复杂。让我们安装以下模块。

`github` 模块为 GitHub v3 API 提供了一个面向对象的包装器；该模块的完整 API 可以在 [`mikedeboer.github.io/node-github/`](http://mikedeboer.github.io/node-github/) 找到。

```js
npm install github --save

```

`async` 模块是一个实用模块，它提供了大约 20 个强大的函数，用于处理异步 JavaScript。`async` 模块是一个控制流模块，它将允许我们以干净、可控的方式执行 IO 操作。

```js
npm install async --save

```

`moment.js` 是一个用于解析、验证、操作和格式化日期的库。

```js
npm install moment --save

```

# 功能：列出仓库

```js
As a vision user
I want to see a list of all repositories for a GitHub account
So that I can select and monitor repositories for my project
```

让我们在 `./test/github.js` 中为我们的功能 `列出仓库` 添加一个测试。这个资源将从路由 `project/:id/repos` 获取一个项目的所有仓库，并返回 `200 Ok` 状态：

```js
describe('when requesting an available resource /project/:id/repos', function(){
  it('should respond with 200', function(done){
    this.timeout(5000);
    request(app)
    .get('/project/' + id + '/repos/')
    .expect('Content-Type', /json/)
    .expect(200)
    .end(function (err, res) {
      var repo = _.first(JSON.parse(res.text))
      assert(_.has(repo, 'id'));
      assert(_.has(repo, 'name'));
      assert(_.has(repo, 'description'));
      done();
    });
  });
});
```

我们需要做的第一件事是在 `./lib/github/index.js` 中创建一个 `GitHubRepo` 模块。我们首先导入所需的模块，包括 `github`。我们定义一个构造函数，它接受一个 GitHub 访问 `token` 和一个 `user` 作为输入。然后我们实例化一个 `GitHubApi` 模块，调用 `github.authenticate`，它基于 token 进行认证：

```js
var GitHubApi = require("github")
, config = require('../configuration')
, async =  require("async")
, moment = require('moment')
, _ =  require("underscore")

function GitHubRepo(token, user) {
  this.token = token;
  this.user = user;

  this.github = new GitHubApi({
    version: "3.0.0",
    timeout: 5000 });

  this.github.authenticate({
    type: "oauth",
    token: token
  });
};

module.exports = GitHubRepo;
```

让我们实现功能 `列出仓库` 并将其添加到我们新的 `GitHubRepo` 模块 `./lib/github/index.js` 中。我们首先定义我们的原型函数 `repositories`。我们在 `github` 模块上调用 `getAll`。如果发生错误，我们返回错误；如果没有找到仓库，我们返回一个 `null` 值。如果我们找到仓库，我们使用 `map` 函数创建一个新的项目数组，使用 `underscore pick` 函数选择三个属性 `id`、`name` 和 `description`。我们通过 `callback` 返回这些 `items`：

```js
GitHubRepo.prototype.repositories = function(callback) {
  this.github.repos.getAll({}, function(error, response) {
    if (error) return callback(error, null);
    if (response == null) return callback(null, null);

    var items = response.map(function(model) {
      return _.pick(model, ['id','name', 'description']);
    });

    callback(null, items);
  });
};
```

让我们在`./lib/project/index.js`中添加一个`repos`函数。我们首先导入`GitHubRepo`模块，然后通过调用静态函数`Project.findOne`尝试检索项目。如果我们得到一个错误，我们返回错误；如果项目不存在，我们返回一个`null`值。如果我们找到项目，我们创建一个`GithubRepo`模块，并用`token`和`user`初始化它，并将其分配给`git`。然后我们调用`git.repositories`，它返回一个响应。如果我们得到一个错误，我们返回一个`error`，如果我们没有找到任何仓库，我们返回一个`null`值。如果我们找到仓库，我们使用`map`函数通过`underscore pick`函数创建一个新的项目数组，选择包括`id`、`name`和`description`在内的三个属性。我们添加一个第四个属性`enabled`，表示我们的项目是否分配了仓库，并返回所有仓库：

```js
, GitHubRepo = require('../github')

Project.prototype.repos = function(id, callback){
  ProjectSchema.findOne({_id: id}, function(error, project) {
    if (error) return callback(error, null);
    if (project == null) return callback(null, null);

    var git = new GitHubRepo(project.token, project.user);

    git.repositories(function(error, response){
      if (error) return callback(error, null);
      if (response == null) return callback("error", null);

      items = response.map(function(model) {
        var item = _.pick(model, ['id','name', 'description''description']);
        var enabled = _.find(project.repositories, function(p){ return p == item.name; });
        (enabled) ? item.enabled = 'checked' : item.enabled = '';
        return item;
      });

      return callback(null, items);
    });
  });
};
```

让我们在`./lib/routes/github.js`中添加一个新的路由`repos`。我们实例化一个新的`ProjectService`，然后通过调用函数`Project.repos`尝试检索项目的仓库。如果我们得到一个错误，我们返回`500 Internal Server Error`。如果没有返回仓库，我们返回`404 Not Found`状态。如果我们收到仓库，我们返回一个包含仓库的`200 OK`状态。

```js
, ProjectService = require('../project')
, Project = new ProjectService();

exports.repos = function(req, res){
  logger.info('Request.' + req.url);

  Project.repos(req.params.id, function(error, repos) {
    if (error) return res.json(500, 'Internal Server Error');
    if (repos == null) return res.json(404, 'Not Found');
    return res.json(200, repos);
  });
};
```

现在，将以下路由添加到`./lib/express/index.js`：

```js
app.get('/project/:id/repos', github.repos);
```

# 功能：列出提交

```js
As a vision user
I want to see a list of multiple repository commits in real time
So that I can review those commits
```

让我们在`./test/github.js`中为我们的`List commits`功能添加一个测试。这个资源将通过路由`project/:id/commits`获取项目中所有仓库的 10 个最新提交，并返回`200 OK`状态：

```js
describe('when requesting an available resource /project/:id/commits', function(){
  it('should respond with 200', function(done){
    this.timeout(5000);
    request(app)
    .get('/project/' + id + '/commits')
    .expect('Content-Type', /json/)
    .expect(200)
    .end(function (err, res) {
      var commit = _.first(JSON.parse(res.text))
      assert(_.has(commit, 'message'));
      assert(_.has(commit, 'date'));
      assert(_.has(commit, 'login'));
      assert(_.has(commit, 'avatar_url'));
      assert(_.has(commit, 'ago'));
      assert(_.has(commit, 'repository'));
      done();
    });
  });
});
```

让我们实现`List commits`功能，并将其添加到我们的新`GitHubRepo`模块`./lib/github/index.js`中。我们首先定义我们的函数`commits`，它接受一个`repos`列表。我们使用`async.each`遍历所有`repos`。`async`模块允许我们在 IO 上执行异步操作。

然后，我们调用`github.repos.getCommits`；我们传递我们的 GitHub `user`和`repo`。如果`github.repos.getCommits()`返回错误，我们调用`callback`。当我们收到响应时，我们使用`map`函数通过`uderscore pick`函数选择两个属性：`committer`和`message`来创建一个新的项目数组。如果项目有`committer`，我们使用 underscores 的`extend`函数添加提交者的`login`和`avatar_url`。我们通过`callback`将项目返回到主函数，并使用 underscores 的`sort`函数按日期排序项目并选择前 10 个项目。然后，我们通过`callback`返回提交：

```js
GitHubRepo.prototype.commits = function(repos, callback) {
  var me = this;
  var items = [];

  async.each(repos, function(repo, callback) {
    me.github.repos.getCommits({ user: me.user,
      repo: repo }, function(error, response) {
      if (error) return callback();
      if (response == null) return callback();

      var repoItems = response.map(function(model) {
        var item =_.pick(model.commit, ['message']);
        if (model.commit.committer) _.extend(item, _.pick(model.commit.committer, ['date']));
        if (model.committer) _.extend(item, _.pick(model.committer, ['login', 'avatar_url']));
        item.ago = moment(item.date).fromNow();
        item.repository  = repo;
        return item;
      });

      items = _.union(items, repoItems);
      callback(null, items );
    });
  }
  , function(error) {
    var top = _.chain(items)
    .sortBy(function(item){ return item.date })
    .reverse()
    .first(10)
    .value();

    callback(error, top);
  });
};
```

让我们在 `./lib/project/index.js` 中添加一个 `commits` 函数。我们首先定义一个名为 `commits` 的函数。我们尝试通过调用静态函数 `Project.findOne` 来检索项目。如果发生错误，我们返回错误。如果项目不存在，我们返回一个 `null` 值。如果我们找到了项目，我们创建一个 `GithubRepo` 模块，并用 token 和用户初始化它，并将其分配给 `git`。然后我们调用 `git.commits` 函数，传递一个存储库列表，返回一个响应。如果发生错误，我们返回错误。如果得到一个有效的响应，我们返回提交。

```js
Project.prototype.commits = function(id, callback){
  ProjectSchema.findOne({_id: id}, function(error, project) {
    if (error) return callback(error, null);
    if (project == null) return callback(null, null);

    var git = new GitHubRepo(project.token, project.user);

    git.commits(project.repositories, function(error, response){
      if (error) return callback(error, null);
      return callback(null, response);
    });
  });
};
```

让我们在 `./lib/routes/github.js` 中添加一个新的路由 `commits`。我们尝试通过调用 `Project.commits` 来检索提交。如果发生错误，我们返回 `500 Internal Server Error`。如果没有返回提交，我们返回 `404 Not Found`。如果我们收到提交，我们返回一个包含提交的 `200 OK` 响应：

```js
exports.commits = function(req, res){
  logger.info('Request.' + req.url);

  Project.commits(req.params.id, function(error, commits) {
    if (error) return res.json(500, 'Internal Server Error');
    if (commits == null) return res.json(404, 'Not Found');
    return res.json(200, commits);
  });
};
```

现在，将以下路由添加到 `./lib/express/index.js`：

```js
app.get('/project/:id/commits', github.commits);
```

# 功能：列出问题

```js
As a vision user
I want to see a list of multiple repository issues in real time
So that I can review and fix issues
```

让我们在 `./test/project.js` 中为我们的 `List issues` 功能添加一个测试。此资源将获取来自 `project/:id/issues` 路由的所有项目，并返回一个 `200 OK` 响应：

```js
describe('when requesting an available resource /project/:id/issues', function(){
  it('should respond with 200', function(done){
    this.timeout(5000);
    request(app)
    .get('/project/' + id + '/issues')
    .expect('Content-Type', /json/)
    .expect(200)
    .end(function (err, res) {
      var issue = _.first(JSON.parse(res.text))
      assert(_.has(issue, 'title'));
      assert(_.has(issue, 'state'));
      assert(_.has(issue, 'updated_at'));
      assert(_.has(issue, 'login'));
      assert(_.has(issue, 'avatar_url'));
      assert(_.has(issue, 'ago'));
      assert(_.has(issue, 'repository'));
      done();
    });
  });
});
```

让我们实现一个名为 `List issues` 的功能，并将其添加到我们新的 `GitHubRepo` 模块 `./lib/github/index.js` 中。我们首先定义我们的 `issues` 函数，它接受一个 `repos` 列表。我们使用 `async.each` 来遍历所有 `repositories`。

然后我们调用 `github.repos.repoIssues` 并传递我们的 GitHub `user` 和 `repo`，如果 `github.repos.repoIssues()` 返回一个 `error`，则调用回调。如果得到一个有效的响应，我们使用 `map` 函数创建一个新的项目数组，使用 `underscore pick` 函数选择四个属性，包括 `id`、`title`、`state` 和 `updated_at`。如果项目有用户，我们使用 `underscore extend` 函数添加用户的 `login` 和 `avatar_url`。然后我们通过 `callback` 将项目返回给主函数，并使用 `underscore sort` 函数按日期排序项目。然后我们选择前 10 个问题，并通过 `callback` 返回问题。

```js
GitHubRepo.prototype.issues = function(repos, callback) {
  var me = this;
  var items = [];

  async.each(repos, function(repo, callback) {
    me.github.issues.repoIssues({ user: me.user, repo: repo }, function(error, response) {
      if (error) return callback();
      if (response == null) return callback();

      var repoItems = response.map(function(model) {
        var item = _.pick(model, ['title', 'state', 'updated_at']);
        if (model.user) _.extend(item, _.pick(model.user, ['login', 'avatar_url']));
        item.ago = moment(item.updated_at).fromNow();
        item.repository = repo;
        return item;
      });

      items = _.union(items, repoItems);
      callback(null, items );
    });
  }
  , function(error) {
    var top = _.chain(items)
    .sortBy(function(item){ return item.updated_at; })
    .reverse()
    .first(10)
    .value();

    callback(error, top);
  });
};
```

让我们在 `./lib/project/index.js` 中添加一个 `issues` 函数。我们首先定义一个名为 `issues` 的函数。我们尝试通过调用静态函数 `Project.findOne` 来检索项目。如果发生错误，我们返回 `error`。如果项目不存在，我们返回一个 `null` 值。如果我们找到了项目，我们创建一个 `GitHubRepo` 模块，并用 `token` 和 `user` 初始化它，并将其分配给 `git`。然后我们调用 `git.issues`，传递一个存储库列表，返回一个响应。如果发生错误，我们返回一个 `error`，如果得到一个有效的响应，我们返回问题和 `200 OK` 响应：

```js
exports.issues = function(req, res){
  logger.info('Request.' + req.url);

  Project.findOne({_id: req.params.id}, function(error, project) {
    if (error) return res.json(500, 'Internal Server Error');
    if (project == null) return res.json(404, 'Page Not Found');

    var git = new GitHubRepo(project.token, project.user);

    git.issues(project.repositories, function(error, response){
      if (error) return res.json(500, 'Internal Server Error');
      return res.json(200, response);
    });
  });
};
```

让我们在 `./lib/routes/github.js` 中添加一个新的路由 `issues`。我们尝试通过调用 `Project.issues` 来检索问题。如果发生错误，我们返回 `500 Internal Server Error`。如果没有返回问题，我们返回 `404 Not Found` 响应，如果收到问题，我们返回包含问题的 `200 OK` 响应：

```js
exports.issues = function(req, res){
  logger.info('Request.' + req.url);

  Project.issues(req.params.id, function(error, issues) {
    if (error) return res.json(500, 'Internal Server Error');
    if (issues == null) return res.json(404, 'Not Found');
    return res.json(200, issues);
  });
};
```

现在，将以下路由添加到`./lib/express/index.js`：

```js
app.get('/project/:id/issues', github.issues);
```

# 使用参数中间件验证参数

你可能已经注意到，我们在每个路由中都重复了`id`验证。让我们使用`app.params`来改进这些事情。

这里是检查我们的`id`是否为有效的 MongoDB `id`的有问题的代码行：

```js
if (req.params.id.match(/^[0-9a-fA-F]{24}$/) == null)
  return res.json(400, 'Bad Request');
```

让我们添加一个中间件来处理这个`./lib/middleware/id.js`。我们定义一个`validate`函数，它接受四个参数，最后一个参数是`id`的值。然后我们验证`id`参数，如果它无效，则返回`400 Bad Request`。然后我们调用`next()`，这将调用 Express 堆栈中的下一个中间件：

```js
exports.validate = function(req, res, next, id){
  if (id.match(/^[0-9a-fA-F]{24}$/) == null)
  return res.json(400, 'Bad Request');
  next();
}
```

现在，我们可以在我们的 Express 服务器中使用这个`id`中间件。让我们包含`param`中间件，并在第一个路由之前添加此行，以便它适用于所有路由：`./lib/express/index.js`：

```js
, id = require('../middleware/id')
..
app.param('id', id.validate);
```

现在，我们可以编辑我们的两个路由模块`./lib/routes/project.js`和`./lib/routes/github.js`，并删除有问题的代码行。`id`参数现在将处理所有路由。

# 路由改进

现在，我们的 Express 服务器需要很多路由；让我们清理一下。在`node.js`中，一个常见的模式是包含一个`index`文件，该文件返回其当前目录中的所有文件。我们将使用`require-directory`来为我们完成这项工作：

```js
npm install require-directory –save

```

让我们创建一个新的模块`./lib/routes/index.js`，代码如下：

```js
var requireDirectory = require('require-directory');
module.exports = requireDirectory(module, __dirname, ignore);
```

现在，`./lib/routes/`文件夹中的所有路由都将暴露在单个变量`routes`下：

```js
  var express = require('express')
  , http = require('http')
  , config = require('../configuration')
  , db = require('../db')
 , routes = require('../routes')
  , notFound = require('../middleware/notFound')
 , id = require('../middleware/id')
  , app = express();

app.use(express.bodyParser());
app.set('port', config.get('express:port'));
app.use(express.logger({ immediate: true, format: 'dev' }));
app.param('id', id.validate);
app.get('/heartbeat', routes.heartbeat.index);
app.get('/project/:id', routes.project.get);
app.get('/project', routes.project.all);
app.post('/project', routes.project.post);
app.put('/project/:id', routes.project.put);
app.del('/project/:id', routes.project.del);
app.get('/project/:id/repos', routes.github.repos);
app.get('/project/:id/commits', routes.github.commits);
app.get('/project/:id/issues', routes.github.issues);
app.use(notFound.index);

http.createServer(app).listen(app.get('port'));
module.exports = app;
```

# 摘要

我们现在已经完成了我们的 Web API。我们实现了一个基本的 MongoDB 提供者；我们使用 Mongoose 为我们提供了一些模式支持。我们还对我们的 Express 服务器进行了一些小的改进，清理了路由。

在下一章中，我们将构建客户端时消费这个 API。
