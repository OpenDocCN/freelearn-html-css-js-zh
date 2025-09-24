# 第三章。模板化

我们已经部署了 Web API，现在让我们将注意力转向客户端。在本章中，我们将消费我们的 Web API，并使用服务器端和客户端模板的混合方式来展示我们的数据。我们将使用 Express 从服务器上提供`./views/index.html`主页面文件，并使用`consolidate.js`和`handlebars.js`进行模板化。在客户端，我们将使用`backbone.js`和预编译的 handlebars 模板，这些模板直接从`./public`文件夹提供。

# 服务器端模板化

到目前为止，我们的 Express 服务器只提供了 JSON；让我们安装一些模块，以帮助我们提供 HTML。

`consolidate.js`是一个模板引擎合并库，它被创建来将所有 Node 的流行模板引擎映射到 Express 的模板约定，允许它们在 Express 中工作：

```js
npm install consolidate --save

```

`handlebars.js`是 mustache 模板语言的扩展。Handlebars 是一个无逻辑的模板语言，它将视图和代码分离：

```js
npm install handlebars --save

```

为了能够提供我们的 handlebar 模板，我们不得不对我们的 Express 服务器做一些修改。让我们通过设置`app.engine`将默认模板引擎更改为 handlebars：

```js
app.engine('html', cons.handlebars);
```

现在注册`html`作为我们的视图文件扩展名。如果我们没有设置这个，我们就需要将我们的视图命名为`index.hbs`而不是`index.html`，其中`.hbs`是 handlebars 模板的扩展名。

```js
app.set('view engine', 'html');
```

让我们创建我们的单页应用程序视图；这将由我们的 Express 服务器提供：

```js
./views/index.html
```

接下来，我们定义我们的`views`文件夹的位置以及静态文件文件夹的位置；这里我们将存储`components`，例如，CSS 和 JavaScript 文件。

```js
app.set('views', 'views');
app.use(express.static('public'));
app.use(express.static('public/components'));
```

现在创建一个名为`public`的文件夹，并添加以下目录结构，以便静态资源以子目录作为前缀提供，例如，`vision/vision.css`。

```js
./public
./public/components
./public/components/vision
```

# 功能：主页面

```js
As a vision user
I want the vision application served as a single page
So that I can spend less time waiting for page loads
```

让我们在`./test/home.js`为我们的功能`Master Page`添加一个测试。这个资源将从路由`./`获取我们的主页面并返回`200 OK`响应。响应的`Content-Type`应该是`HTML`：

```js
   var app = require('../app')
 , request = require('supertest');

describe('vision master page', function(){
  describe('when requesting resource /', function(){
    it('should respond with view', function(done){
      request(app)
        .get('/')
        .expect('Content-Type', /html/)
        .expect(200, done)
    });
  });
});
```

让我们实现我们的`Master Page`功能。让我们创建一个新的模块，该模块公开一个路由`./lib/routes/home.js`并添加一个新的`index`函数。我们首先定义一个名为`index`的路由。我们创建一个带有页面元信息的视图`model`，然后通过传递视图`model`来渲染视图：

```js
exports.index = function(req, res){
  var model = {
    title: 'vision.',
    description: 'a project based dashboard for github',
    author: 'airasoul',
     user: 'Andrew Keig'
  };
  res.render('index', model);
};
```

让我们在 Express 服务器`./lib/express/index.js`中添加一个新的路由：

```js
app.get('/', routes.home.index);
```

# 使用 Bower 进行包管理

我们现在将安装构成我们客户端的各种组件，即`Handlebars.js`、`Backbone.js`和 Twitter Bootstrap 2 版本，使用**Bower**。

Bower 是 Web 的包管理器。一个 Bower 包可以包含不同类型的资源，如 CSS、JavaScript 和图像。让我们使用以下命令全局安装 Bower：

```js
npm install -g bower

```

在 Bower 中，依赖关系列在`bower.json`文件中，类似于 Node 的`package.json`。让我们创建一个`./bower.json`文件并定义我们的客户端依赖项：

```js
{
  "name": "vision",
  "version": "0.0.1",
  "dependencies": {
    "json2": "*",
    "jquery": "*",
    "underscore": "*",
    "backbone": "*",
    "handlebars": "*",
    "bootstrap": "2.3.2"
  }
}
```

现在创建以下 Bower 配置文件 `./.bowerrc`，它允许我们定义我们的目标目录和 `bower.json` 文件的名字：

```js
{
    "directory": "public/components",
    "json": "bower.json"
}
```

运行以下命令来安装 `bower.json` 文件中列出的所有依赖项：

```js
bower install

```

Twitter Bootstrap 的资源存储在以下片段中指定的路径指定的文件夹中，因此让我们添加一个 `static` 中间件来覆盖我们的 Express 服务器。这将保持客户端路径的一致性：

```js
app.use('/bootstrap', express.
  static('public/components/bootstrap/docs/assets/css'));
```

# 模板

我们的主页包含以下部分。为了便于使用 `backbone.js` 进行客户端模板化模型，我们将主页拆分成模板。

让我们创建一个名为 `./templates` 的新文件夹，并添加以下文件：

```js
./templates
  projects.hbs
  project-form.hbs
  repositories.hbs
  commits.hbs
  issues.hbs
```

为了避免按需编译模板，让我们安装 `grunt-contrib-handlebars` 这个 grunt 任务，它将预编译我们的 handlebars 模板：

```js
npm install grunt-contrib-handlebars --save-dev

```

在以下代码中，我们概述了 handlebars 编译的 grunt 配置；它简单地以模板位置 `templates/*.hbs` 作为输入，将这些模板编译成一个单一的 JavaScript 文件，并将其存储在 `public/components/vision/templates.js`。

```js
grunt.loadNpmTasks('grunt-contrib-handlebars');

handlebars: {
  compile: {
    options: {
      namespace: "visiontemplates"
    },
    files: {
      "public/components/vision/templates.js": ["templates/*.hbs"]
    }
  }
},
```

我们通过查看主模板 `./views/index.html` 来完成这一部分。主体包含以下区域：一个头部，包括一个 `login` 按钮或一个带有 `welcome` 消息的 `logout` 按钮，一个 `project-list` 表单，`repository-list`，`commit-list` 和 `issue-list`。

```js
  {{#if user}}
       <p class="navbar-text">welcome {{user}},
       <a href="/logout" class="navbar-link">
         click here to sign out</a>
       </p>
  {{else}}
        <a href="/auth/github">
        <img src="img/github.png" id='login'>
        </a>
      {{/if}}

      {{#if user}}
      <div class="span3">
          <h2>Projects</h2>
          <ul id="projects-list"  class="nav nav-list"></ul>
          <br/><a id="showForm" class="btn btn-large btn-block btn-primary" href="#add">Add project</a>
      </div>
      <div class="span3">
        <h2>Repositories</h2>
        <ul id="repository-list" class="nav inline nav-list"></ul>
      </div>
      <div class="span3">
        <h2>Commits</h2>
        <ul id="commits-list" class="media-list"></ul>
      </div>
      <div class="span3">
        <h2>Issues</h2>
        <ul id="issues-list" class="media-list"></ul>
      </div>
      {{else}}   
        <div class="span12">
          <div class="hero-unit">
            <h1>vision</h1>
            <lead>a real-time multiple repository dashboard for GitHub issues and commits</lead>
            <p><small>In order to use vision; please login to a valid GitHub Account</small></p>
          </div>
        </div>
      {{/if}}
```

# 使用 Backbone.js 进行客户端开发

`Backbone.js` 是一个轻量级且非常灵活的 **JavaScript 模型视图**（**MV**）框架，它简化了复杂 JavaScript 应用程序的构建。它包括一些非常基本的原语，允许我们将客户端的模型和逻辑与其视图解耦。Backbone 支持一个 **RESTful JSON** 接口，将模型/集合与 RESTful API 相关联。有关 `Backbone.js` 的更多信息，请访问 [`backbonejs.org`](http://backbonejs.org)。

# 功能：列出项目

让我们构建我们的功能 `列出项目` 的客户端。列表中的每个项目都包含一个项目名称和一个编辑和删除按钮。点击名称将显示一个仓库列表；点击 **编辑** 将显示一个填充了模型数据的内联表单，点击 **删除** 将从我们的数据库中删除该条目。我们稍后会回来连接这三个功能。现在，我们只是简单地显示一个项目列表。

接下来是一个用于项目条目的 HTML 模板 `./templates/projects.hbs`；它包含一个占位符 `{{_id}}`，它将被我们的 Backbone 应用程序替换：

```js
<a href="#{{_id}}" data-id="{{_id}}">{{name}}</a>
<button class="delete btn btn-mini btn-primary list-btn">del </ button>
<button class="edit btn btn-mini btn-primary list-btn spacer ">edit e</button>
```

让我们定义一个包含所有组件的骨架 Backbone 应用程序：`./public/components/vision/vision.js`。我们首先定义 `Vision` 命名空间；我们向其中添加一个名为 `Application` 的外部函数，它有一个名为 `start` 的单一方法。在这里，我们实例化一个 `router` 并调用 `Backbone.history.start()` 以启动 Backbone 应用程序。然后我们调用 `router.navigate('index', true)` 并导航到我们的主页。有了这个函数，我们实例化新的 `Vision.Application()` 并调用 `start()`。

```js
var Vision = Vision || {};

Vision.Application = function(){
    this.start = function(){
        var router = new Vision.Router();
        Backbone.history.start();
        router.navigate('index', true);
    }
};

$(function(){
    var app = new Vision.Application();
    app.start();
});
```

现在让我们创建一个名为 `Router` 的应用程序。一般来说，Backbone 应用程序只有一个这样的组件；路由器是我们应用程序的入口点。

首先，我们添加一个名为 `Router` 的函数，它扩展了 Backbone 的 `Router` 类型。我们添加了一个名为 `projectListView` 的项目列表视图，并添加了一个 `routes` 哈希，它定义了一个单一的路由。我们应用程序的入口点是一个空路由，映射到名为 `index` 的方法。当路由器实例化时，会调用 `initialize` 或构造函数方法；从这里我们调用一个名为 `project` 的方法，它实例化一个 `ProjectListView`。`index` 方法与之前定义的路由相匹配，通过调用 `projectApplication.render()` 来渲染我们的视图。

```js
Vision.Router = Backbone.Router.extend({
    projectListView : "",

    routes: {
        "" : "index",
    },

    initialize : function(){
      this.project();
    },

    project : function(){
      this.projectListView = new Vision.ProjectListView();
    },  

    index : function(){
        this.projectListView.render();
    }
});
```

让我们实现我们的 `Project` 模型以支持我们的视图。我们首先添加一个名为 `Project` 的函数，它扩展了 Backbone 的 `Model` 类型，并包含我们模型中两个属性的默认值哈希。我们覆盖了 `idAttribute` 参数以适应 MongoDB 标识符。我们将使用 MongoDB 的 `_id` 作为我们的模型标识符；默认情况下，Backbone 将使用 `id`。此标识符将被附加到 Backbone 向服务器发出的任何请求中，例如，在执行 GET、POST、PUT 或 DELETE 操作时。我们已经在 第二章 中添加了此模型的 API，*构建 Web API*。`urlRoot` 参数将此模型链接到 Web API 路由 `/project` 以返回一个项目。

```js
Vision.Project = Backbone.Model.extend({
    defaults: {
          id : ""
        , name: ""
    },

    idAttribute: "_id",
    urlRoot: '/project'
});
```

让我们实现一个集合；为我们的 `Project` 模型添加一个名为 `ProjectList` 的集合。我们添加一个名为 `ProjectList` 的函数，它扩展了 Backbone 的 `Collection` 类型，并指定模型类型为 `Vision.Project`。我们添加一个 `url` 方法，它返回我们的 Web API 路由 `/project` 以返回项目列表。当集合实例化时，会调用 `initialize` 方法；从这里我们执行初始的 `fetch()` 来获取我们的项目；因此调用 API `/project`。

```js
Vision.ProjectList = Backbone.Collection.extend({
    model: Vision.Project,

    url: function () {
        return "/project/";
    },

    initialize: function() {
        this.fetch();
    }
});
```

在我们实现 `ProjectListView` 之前，让我们创建 `event_aggregator`；这将允许我们的视图触发和绑定命名事件，其他视图可以响应这些事件。我们需要这样做，以便 `ProjectListView` 通知 `RepositoryListView` 显示 `RepositoryList` 的时间到了。

让我们使用 `underscore.js` 的 `extend` 方法将 Backbone 的 `event` 模块混合到我们的视图原型中，添加一个名为 `event_aggregator` 的函数：

```js
Backbone.View.prototype.event_aggregator = _.extend({}, Backbone.Events);
```

让我们为我们的`Project`集合实现一个视图——`ProjectListView`。我们首先定义一个函数`ProjectListView`，它扩展了 Backbone 的`View`类型，并为我们的项目列表添加了一个`Projects`数组。我们将一个 DOM 元素分配给`el`；一个名为`projects-list`的无序列表。这是我们视图将被插入的元素。如果你没有将其分配给`el`，Backbone 将构造一个空的`div`标签。

当视图实例化时调用`initialize`方法；在这里，我们实例化一个新的`ProjectList`，传递我们的`Projects`数组。然后我们调用`collection.on('add')`，当从 API 获取数据时将调用`add`方法。`add`方法实例化`ProjectView`，并将其传递给一个`project`模型。然后我们通过`$el`将`ProjectView`追加到我们的 DOM 元素中，并返回视图。

```js
Vision.ProjectListView = Backbone.View.extend({
    Projects: [],
    el: $("ul#projects-list"),

    initialize: function () {
      this.collection = new Vision.ProjectList(this.Projects);
      this.collection.on('add', this.add, this);
    },

    add: function (project) {
      var projectView = new Vision.ProjectView({
        model: project
      });

      this.$el.append(projectView.render().el);
      return projectView;
    }
});
```

我们通过实现单个项目的视图——`ProjectView`来完成本节。我们首先定义一个函数`ProjectView`，它扩展了 Backbone 的`View`类型，并将`tagName`分配给它，将其设置为`li`。这个标签将围绕我们的项目视图；我们的 DOM 元素是一个`ul`标签。

然后我们包含`viewTemplate`并将预编译的 handlebars 模板分配给它。尽管模板被编译到一个单独的文件——`./vision/templates.js`——但我们仍然通过名称引用模板；`templates/projects.hbs`。`render`方法渲染视图；我们将`project`模型传递给我们的`viewTemplate`，然后通过`$el`添加到我们的 DOM 元素中，并返回视图：

```js
Vision.ProjectView = Backbone.View.extend({
    tagName: "li",
    viewTemplate: visiontemplates["templates/projects.hbs"],

    render: function () {
        var project = this.viewTemplate(this.model.toJSON());
        this.$el.html(project);
        return this;
    }
});
```

如果你进入 MongoDB，并将以下记录添加到 vision 数据库中项目的集合，当在浏览器中访问 Vision 应用程序时，你可以在项目列表视图中看到此记录：

```js
{
    "_id" : ObjectId("525c61bcb89855fc09000018"),
    "created" : ISODate("2013-10-17T22:58:37Z"),
    "name" : "test name",
    "token" : "#TOKEN#",
    "user" : "#USER#"
}
```

# 功能：列出仓库

让我们构建我们的功能`List repositories`的客户端。列表中的每个项目都由一个仓库名称、一个简短描述和一个复选框组成；这允许我们向项目添加或删除仓库。

接下来是一个用于仓库项的 HTML 模板`./templates/repositories.hbs`：

```js
<li>
  <label class="checkbox inline">
  <input id="{{id}}" type="checkbox" {{enabled}} value="{{name}}"><h4 class="media-heading repoItem">{{name}}</h4>
  <small>{{description}}</small>
  </label>
</li>
```

让我们添加一个`Repository`模型。我们添加一个函数`Repository`，它扩展了 Backbone 的`Model`类型，并为我们的模型中的四个属性添加了一个默认值的哈希。`enabled`属性表示仓库包含在所选项目中。

```js
Vision.Repository = Backbone.Model.extend({
    defaults: {
          id : ""
        , name: ""
        , description: ""
        , enabled: ""
    }
});
```

让我们为我们的`Repository`模型实现一个集合。我们首先定义一个函数`RepositoryList`，它扩展了 Backbone 的`Collection`类型。我们添加了所选项目的`projectId`，并将模型类型设置为`Vision.Repository`。然后我们添加了一个`url`方法，并使用 Web API 路由`/project/:id/repos`来获取一个项目的仓库列表。

当集合实例化时调用`initialize`方法；从这里，我们分配所选的`projectId`。当执行获取操作时调用`parse`方法，并将解析的响应分配给我们的 MongoDB `_id`。

```js
Vision.RepositoryList = Backbone.Collection.extend({
    projectId: '',
    model: Vision.Repository,

    url : function() {
      return '/project/' + this.projectId + '/repos';
    },

    initialize: function(items, item) {
        this.projectId = item.projectId;
    },

    parse: function( response ) {
        response.id = response._id;
        return response;
    }
});
```

我们现在实现一个单个仓库的视图。我们添加一个函数 `RepositoryView`，它扩展了 Backbone 的 `View` 类型，并将 `tagName` 赋值为 `li`。这个标签将围绕我们的 `RepositoryView` 函数；我们的 DOM 元素是一个 `ul` 标签。我们包含一个 `viewTemplate` 函数，并将预编译的 handlebars 模板 `templates/repositories.hbs` 赋值给它。`render` 方法渲染视图；我们向 `viewTemplate` 函数传递 `repository` 模型，然后通过 `$el` 添加到我们的 DOM 元素中，并返回视图。

```js
Vision.RepositoryView = Backbone.View.extend({
    tagName: "li",
    viewTemplate: visiontemplates["templates/repositories.hbs"],

    render: function () {
        this.$el.html(this.viewTemplate(this.model.toJSON()));
        return this;
    }
});
```

让我们实现一个名为 `RepositoryListView` 的 `RepositoryList` 视图。我们首先定义一个函数 `RepositoryListView`，它扩展了 Backbone 的 `View` 类型，并为我们的仓库列表添加一个 `Repositories` 数组。我们添加一个 `initialize` 方法；如果 `projectId` 为空，则返回。有效的 `projectId` 将导致渲染视图；首先，我们清除 DOM 元素，然后我们将一个新的 `RepositoryList` 函数分配给视图的 `collection`。我们使用 `Repositories` 数组和 `projectId` 初始化列表，然后在我们的集合中调用 `fetch`，然后调用 `render` 以成功获取。

`render` 方法使用 underscore 遍历名为 `collection.models` 的仓库集合，为每个项目调用 `add(item)`。我们包含一个 `add` 方法，它实例化一个 `RepositoryView` 函数，并将一个 `repository` 模型传递给它。然后我们通过 `$el` 将渲染后的 `RepositoryView` 添加到我们的 DOM 元素中，并返回视图。

```js
Vision.RepositoryListView = Backbone.View.extend({
    Repositories: [],

    initialize: function (args) {
      if (!args.projectId) return;
      var me = this;
      this.$el.html('');
      this.collection = new Vision.RepositoryList(this.Repositories, {
          projectId : args.projectId
        });
        this.collection.fetch({success: function(){
          me.render();
        }});
    },

    render: function () {
      _.each(this.collection.models, function (item) {
        this.add(item);
      }, this);
    },

    add: function (item) {
      var repositoryView = new Vision.RepositoryView({
        model: item
      });

      this.$el.append(repositoryView.render(this.editMode).el);
      return repositoryView;
    }
});
```

让我们对 `ProjectView` 进行一些修改，并在选择项目时添加一个点击事件。我们首先定义一个 `events` 哈希，其中包含一个名为 click 的单个事件，它调用 `repository` 方法。`repository` 方法从我们的模型中获取 `projectId`，然后调用 `event_aggregator` 上的 `trigger` 方法，传递事件 `repository:join` 和 `projectId`。我们将在 `ProjectListView` 上监听此事件。

```js
    events: {
      "click a" : "repository"
    },

    repository: function() {
      var data = { projectId: this.model.toJSON()._id }
      this.event_aggregator.trigger('repository:join', data);
    },
```

让我们将上一个事件的另一侧连接起来，并为 `ProjectListView` 添加一个事件绑定器。我们在 `initialize` 方法中添加一个 `event_aggregator.bind` 语句，将事件 `repository:join` 绑定到 `repository` 方法。`repository` 方法在路由器上触发一个 `join` 事件。

```js
    initialize: function () {
      this.event_aggregator.on('repository:join', this.repository, this);
        this.collection = new Vision.ProjectList(this.Projects);
        this.render();
    },

    repository: function(args){
      this.trigger('join', args);
    },
```

让我们完善画面，并将路由器改为监听 `join` 事件。我们向路由器添加一个 `repositoryListView` 函数，并在 `initialize` 方法中添加一个 `listenTo` 事件，该事件调用 `join` 方法。`join` 方法调用 `repository`，它实例化 `RepositoryListView` 函数，并传递 `projectId`。

```js
repositoryListView:'',

initialize : function(){
  this.project();
  this.listenTo(this.projectListView , 'join', this.join);
},

join : function(args){
  this.repository(args);
},

repository : function(args){
  this.repositoryListView =new Vision.RepositoryListView({ el: 'ul#repository-list', projectId: args.projectId });
},
```

现在，当你点击 `ProjectView` 中的项目项名称时，将显示 `RepositoryListView`。

# 功能：创建项目

让我们为我们的功能“创建项目”添加一个项目表单。它包括一个大的**添加项目**按钮，一个用于项目名称的文本框，以及**保存**和**取消**按钮。点击**保存**将项目 POST 到我们的 Express 服务器，而点击**取消**将关闭表单。

以下是一个用于存储项的 HTML 模板 `./templates/project-form.hbs`：

```js
<form class="form-inline">
  <ul class="errors help"></ul>
  <label>name</label>
  <input class="name" placeholder="project name" required="required" value="{{name}}" autofocus />
  <br/><button class="cancel btn btn-mini btn-primary form-btn">cancel</button>
  <button class="save btn btn-mini btn-primary form-btn form-spacer">save</button>
</form>
```

让我们对 `router` 进行一些修改，并将一个路由连接到我们的“添加项目”按钮。`routes` 现在包括一个名为 `add` 的路由，它调用一个名为 `add` 的方法。我们包括一个 `add` 方法，该方法调用 `projectListView.showForm()`，渲染我们的表单：

```js
    routes: {
      "" : "index",
      "add" : "add"
    },
    add : function(){
      this.projectListView.showForm();
    }
```

让我们对 `projectListView` 进行一些修改，并修改 `initialize` 方法。我们将此视图绑定到 `collection` 的 `reset`、`add` 和 `remove` 事件。我们还添加了一个 `showForm` 方法，如前述代码所示。该方法通过调用 `this.add()`、传递 `new Vision.Project()` 并在返回的视图中调用 `add()` 来渲染项目表单。

```js
initialize: function () {
  this.event_aggregator.on('repository:join', this.repository, this);
  this.collection = new Vision.ProjectList(this.Projects);
  this.collection.on('reset', this.render, this);
  this.collection.on('add', this.add, this);
  this.collection.on('remove', this.remove, this);
},

showForm: function () {
  this.add(new Vision.Project()).add();
}
```

让我们在 `Project` 模型中添加一些验证，以便我们可以验证项目表单的输入。我们在 `Project` 模型中添加了一个 `validate` 方法，并验证 `Project` 模型的名称。如果验证失败，我们返回一个包含错误信息的 `errors` 数组。我们实际上是在重写 `validate` 方法。`Backbone.js` 要求您使用自定义验证逻辑重写 `validate` 方法。默认情况下，`validate` 方法也是 `save` 调用的一部分。

```js
validate: function(attrs) {
  var errors = [];
  if (attrs.name === '') errors.push("Please enter a name");
  if (errors.length > 0) return errors;
}
```

让我们对 `projectView` 进行一些修改。我们首先添加一个新的模板 `formTemplate`，该模板显示一个用于添加新项目的表单。我们在 `events` 哈希中添加了两个新事件——一个按钮 `save` 事件和一个按钮 `cancel` 事件。

`cancel` 方法响应取消事件，将从我们的模型中获取当前的 `projectId` 并检查 `model.isNew`。如果是新的，我们只需从 `projectListView` 中移除 `projectView`。如果不是新的，我们将渲染我们的视图，并通过调用 `repository` 渲染 `repositoryListView`。然后我们使用 `history.navigate` 导航到 `index` 页面。

`save` 方法响应 `save` 事件，从我们的模型中获取 `projectId` 和表单数据。然后我们调用 `model.isValid`，它调用我们项目模型中的 `validate` 方法。任何返回的错误都会导致调用 `formError`。如果模型有效，我们将获取我们的选定存储库并将其分配给我们的表单。然后我们尝试通过调用 `model.save` 将表单作为 `Project` 保存。任何返回的错误都会导致调用 `formError`。成功的保存使我们能够在 `ProjectListView` 中渲染 `project`。我们还通过调用 `repository` 渲染 `RepositoryListView`。然后我们使用 `history.navigate` 导航到 `index` 页面。

```js
formTemplate: visiontemplates["templates/project-form.hbs"],

events: {
    "click a" : "repository"
    "click button.save": "save",
    "click button.cancel": "cancel"
},

add: function () {
    this.$el.html(this.formTemplate(this.model.toJSON()));
    this.repository();
},

cancel: function () {
    var projectId = this.model.toJSON()._id;

    if (this.model.isNew()) {
      this.remove();
    } else {
      this.render();
      this.repository();
    }

    Backbone.history.navigate('index', true);
},

  save: function (e) {
    e.preventDefault();

    var me = this
    , formData = {}
    , projectId = this.model.toJSON()._id;

    $(e.target).closest("form")
    .find(":input").not("button")
    .each(function () {
      formData[$(this).attr("class")] = $(this).val();
    });

    if (!this.model.isValid()) {
      me.formError(me.model, me.model.validationError, e);
    } else {
      formData.repositories = $('#repository-list')
      .find("input:checkbox:checked")
      .map(function(){
      return $(this).val();
    }).get();
  }

  this.model.save(formData, {
    error: function(model, response) {
      me.formError(model, response, e);
    },
    success: function(model, response) {
      me.render();
      me.repository();
      Backbone.history.navigate('index', true);
    }
  });
},

formError: function(model, errors, e) {
  $(e.target).closest('form').find('.errors').html('');

  _.each(errors, function (error) {
    $(e.target).closest('form').find('.errors')
    .append('<li>' + error + '</li>')
  });
}
```

现在，您将能够完成表单并添加一个新的项目。

# 功能：编辑项目

让我们为我们的功能“编辑项目”添加一个编辑项目表单。它包括一个项目名称的文本框、一个保存按钮和一个取消按钮。点击**保存**会将项目发送到我们的 Express 服务器；点击**取消**会关闭表单。我们将使用与添加项目相同的 handlebars 模板。为了使 `RepositoryListView` 可编辑，我们需要引入编辑状态的概念。我们将其命名为 `editMode`。

让我们对 `projectView` 进行一些修改。我们首先向 `events` 哈希中添加一个名为 `edit` 的新事件，该事件调用一个 `edit` 函数。我们通过向 `event_aggregator` 传递新的 `arg.editMode` 参数来更改我们的 `repository` 方法，这将通知 `RepositoryListView` 它处于编辑模式。

`edit` 方法，它显示我们的 `formTemplate` 并用 `project` 模型数据填充，调用 `repository` 方法并将 `editMode` 设置为 `false`，通知 `RepositoryListView` 它处于编辑模式。最后，我们更新我们的 `add`、`cancel` 和 `save` 方法；这些方法中对 `repository` 方法的调用应传递 `{editMode:false}`。

```js
    Events: {
      ...
        "click button.edit": "edit"
    },

    repository: function(args) {
      var data = { projectId: this.model.toJSON()._id, editMode: args.editMode || false }
      ...
    },

    edit: function () {
      var model = this.model.toJSON();
      this.$el.html(this.formTemplate(model));
      this.repository({editMode:true});
    },
```

让我们对 `RepositoryListView` 进行一些修改。现在，当 `collection.fetch` 成功请求时，`initialize` 方法将根据 `editMode` 启用或禁用表单复选框。`enableForm` 函数从我们的 `RepositoryListView` 复选框列表中移除 `disabled` 标签。`disableForm` 函数向我们的 `RepositoryListView` 复选框列表中添加 `disabled` 标签。

```js
    initialize: function (args) {
      ...
      this.collection.fetch({ success: function(){
        me.render();
        (args.editMode) ?  me.enableForm() : me.disableForm();
      }});
    },

    enableForm: function(){
      this.$el.find("input:checkbox").remove('disabled');
    },

    disableForm: function(){
      this.$el.find("input:checkbox").attr('disabled', 'disabled');
    }
```

现在，您将能够编辑您现有的项目。

# 功能：删除项目

让我们为功能 `Delete a project` 在我们的表单中添加一个 **删除** 按钮。

让我们对 `ProjectView` 进行修改，并向 `events` 哈希中添加一个名为 `delete` 的新事件，该事件调用 `delete` 方法。我们添加一个 `delete` 方法，它销毁模型并移除 `ProjectView`。然后我们调用 `repository`，移除 `RepositoryListView`。

```js
    events: {
      ...
      "click button.delete": "delete",
    },

    delete: function () {
      this.model.destroy();
      this.remove();
      this.repository({editMode:false});
    },
```

让我们对 `ProjectListView` 进行修改，并在 `initialize` 中添加一个 `collection` 事件处理程序。事件处理程序在移除项目时调用 `remove` 方法。`remove` 方法获取模型的属性并搜索 `Projects` 集合，找到时移除项目。

```js
    initialize: function () {
      ...
      this.collection.on("remove", this.remove, this);
    },

    remove: function (removedModel) {
      var removed = removedModel.attributes;

      _.each(this.Projects, function (project) {
        if (_.isEqual(project, removed)) {
          this.Projects.splice(_.indexOf(projects, project), 1);
        }
      });
    },
```

现在，您可以通过点击删除按钮来删除项目。

# 功能：列出提交

让我们为功能 `List Commits` 添加一个提交列表。列表中的每个项目由一个提交 `message`、项目 `name`、一个 `date` 和提交者的 `username` 组成。以下是一个提交项的 HTML 模板 `./templates/commits.hbs`：

```js
  <a class="pull-left" href="#">
    <img class="media-object" src="img/{{avatar_url}}"   
      style="width:64px; height:64px">
  </a>
  <div class="media-body">
    <h4 class="media-heading">{{message}}</h4>
    <small>{{repository}}</small>
    <small>{{ago}}</small>
    <br/><small>{{login}}</small>
  </div>
```

让我们实现我们的 `Commit` 模型。我们定义一个名为 `Commit` 的函数，它扩展了 Backbone `Model` 类型，并包括我们模型中属性的默认值哈希。

```js
Vision.Commit = Backbone.Model.extend({
    defaults: {
      date : '',
      ago: '',
      message : '',
      login : '',
      avatar_url : ''
    }
});
```

让我们为我们的 `Commit` 模型实现一个集合，名为 `CommitList`。我们定义一个名为 `CommitList` 的函数，它扩展了 Backbone `Collection` 类型。我们指定模型类型为 `Vision.Commit`。我们添加一个 `url` 方法，它使用 Web API 路由 `/project/:id/commits` 返回提交列表。当集合实例化时调用 `initialize` 方法；从这里我们分配 `projectId`。当执行获取操作时调用 `parse` 方法，它将解析响应。在这里我们将我们的 MongoDB `_id` 分配给 `response.id`。

```js
Vision.CommitList = Backbone.Collection.extend({
    projectId: '',
    model: Vision.Commit,

    url : function() {
      return '/project/' + this.projectId + '/commits';
    },

    initialize: function(items, item) {
      this.projectId = item.projectId;
    },

    parse: function( response ) {
      response.id = response._id;
      return response;
    }
});
```

让我们为我们的`Commit`集合实现一个视图。我们定义了一个函数`CommitListView`，它扩展了 Backbone 的`View`类型，并为我们的提交列表添加了一个`Commits`数组。当视图实例化时调用`initialize`方法；从这里我们调用`create`并实例化一个新的`CommitList`，传递我们的`Commits`数组。我们调用`refresh`，它遍历`Commits`集合，通过调用`render`方法渲染视图。`render`方法使用 underscore 遍历名为`collection.models`的`Commits`集合，对每个`commit`调用`add(item)`。`add`方法实例化`CommitView`，传递一个`Commit`模型给它，然后通过`$el`将渲染的`CommitView`追加到 DOM 元素中，并返回视图。

```js
Vision.CommitListView = Backbone.View.extend({
  Commits: [],

  initialize: function (args) {
    if (!args.projectId) return;
    this.Commits = args.commits || [];
    this.$el.html('');
    this.create(args);
    this.refresh();
  },

  refresh: function(){
    var me = this;

    if (!this.Commits.length) {
      this.collection.fetch({ success: function(){
        me.render();
      }});
    }
  },

  create: function(args) {
    this.collection = new Vision.CommitList(this.Commits, { projectId : args.projectId });
    this.render();
  },

  render: function () {
    _.each(this.collection.models, function (item) {
      this.add(item);
    }, this);
  },

  add: function (item) {
    var commitView = new Vision.CommitView({ model: item });

    this.$el.append(commitView.render().el);
    return commitView;
  }
});
```

我们继续添加一个用于单个提交项的视图。我们定义了一个函数`CommitView`，它扩展了 Backbone 的`View`类型，并为它添加了一个`tagName`，将其分配为`li`。这个标签将围绕我们的提交视图；我们的 DOM 元素是一个`ul`标签。我们包括`viewTemplate`并将其分配给预编译的 handlebars 模板`./templates/commits.hbs`。`render`方法渲染视图；我们传递`commit`模型到我们的`viewTemplate`，然后通过`$el`添加到我们的 DOM 元素中，并返回视图。

```js
Vision.CommitView = Backbone.View.extend({
    tagName: 'li',
    className: 'media',
    viewTemplate: visiontemplates['templates/commits.hbs'],

    render: function () {
      this.$el.html(this.viewTemplate(this.model.toJSON()));
      return this;
    }
});
```

让我们完善画面并更改我们的路由；我们在路由中添加了一个`CommitListView`，并在`join`方法中调用`commits`。`commits`方法实例化一个`CommitListView`，传递当前的`projectId`和提交列表。

```js
CommitListView:'',

join : function(args){
  this.repository(args);
  this.commits(args);
},

commits : function(args){
  this.commitListView = new Vision.CommitListView({ el: 'ul#commits-list', projectId: args.projectId, commits : args.commits});
},
```

当选择项目时，Vision 将显示提交列表。

# 功能：列出问题

让我们构建我们的问题列表。列表中的每个项仅由一个问题标题、项目名称、日期、发布者的用户名及其状态组成。

接下来是一个用于问题项的 HTML 模板`./templates/issues.hbs`：

```js
<a class="pull-left" href="#">
  <img class="media-object" src="img/{{avatar_url}}"style="width:64px; height:64px">
</a>
<div class="media-body">
  <h4 class="media-heading">{{title}}</h4>
  <small>{{repository}}</small>
  <small>{{ago}}</small>
  <br/><small>{{login}},<b>{{state}}</b></small>
</div>
```

让我们实现我们的`Issue`模型；我们定义了一个函数`Issue`，它扩展了 Backbone 的`Model`类型，并包括模型属性中的默认值哈希。

```js
Vision.Issue = Backbone.Model.extend({
  defaults: {
    title : '',
    state : '',
    date : '',
    ago: '',
    login : '',
    avatar_url : ''
  }
});
```

让我们为我们的`Issue`模型实现一个名为`IssueList`的集合。我们定义了一个函数`IssueList`，它扩展了 Backbone 的`Collection`类型，并将模型类型指定为`Vision.Issue`。我们添加了一个`url`方法，它使用 Web API 路由`/project/:id/issues`来返回问题列表。当集合实例化时调用`initialize`方法；从这里我们分配选定的`projectId`。当执行获取操作时调用`parse`方法，这里我们将我们的 MongoDB `_id`分配给`response.id`。

```js
Vision.IssueList = Backbone.Collection.extend({
  projectId: '',
  model: Vision.Issue,

  url : function() {
    return '/project/' + this.projectId + '/issues';
  },

  initialize: function(items, item) {
    this.projectId = item.projectId;
  },

  parse: function( response ) {
    response.id = response._id;
    return response;
  }
});
```

让我们为我们的`Issue`集合实现一个视图。我们定义一个函数，`IssueListView`，它扩展了 Backbone 的`View`类型，并为我们的问题列表添加了一个`Issues`数组。当`view`被实例化时，会调用`initialize`方法；从这里我们调用`create`并实例化一个新的`IssueList`，传递我们的`Issues`数组。然后我们调用`refresh`，它遍历`Issues`集合，通过调用`render`来渲染视图。`render`方法使用 underscore 遍历名为`collection.models`的`Issues`集合；并为每个问题调用`add(item)`。`add`方法实例化`IssueView`，并传递一个`Issue`模型。然后我们通过`$el`将渲染后的`IssueView`追加到我们的 DOM 元素中，并返回视图。

```js
Vision.IssueListView = Backbone.View.extend({
  Issues: [],

  initialize: function (args) {
    if (!args.projectId) return;
    this.Issues = args.issues || [];
    this.$el.html('');
    this.create(args);
    this.refresh();
  },

  create: function(args) {
    this.collection = new Vision.IssueList(this.Issues, { projectId : args.projectId });
    this.render();
  },

  refresh: function(){
    var me = this;

    if (!this.Issues.length) {
      this.collection.fetch({ success: function(){
        me.render();
      }});
    }
  },

  render: function () {
    _.each(this.collection.models, function (item) {
      this.add(item);
    }, this);
  },

  add: function (item) {
    var issueView = new Vision.IssueView({ model: item });

    this.$el.append(issueView.render().el);
    return issueView;
  }
});
```

接下来，我们添加一个用于单个问题的视图。我们定义一个函数，`IssueView`，它扩展了 Backbone 的`View`类型，并为它添加了一个`tagName`，将其赋值为`li`；这个标签将围绕我们的`IssueView`函数。我们的 DOM 元素是一个`ul`标签。我们包含了一个`viewTemplate`并将其赋值给我们的预编译的 handlebars 模板`templates/issues.hbs`。`render`方法渲染视图；我们将`issue`模型传递给`viewTemplate`，然后通过`$el`将其添加到我们的 DOM 元素中，并返回视图。

```js
Vision.IssueView = Backbone.View.extend({
  tagName: 'li',
  className: 'media',
  viewTemplate: visiontemplates['templates/issues.hbs'],

  render: function () {
    this.$el.html(this.viewTemplate(this.model.toJSON()));
    return this;
  }
});
```

让我们完善这个画面并更改我们的路由器；我们在路由器中添加了一个`issueListView`，并在`join`方法中调用`issues`。`issues`方法实例化`IssueListView`，传递`projectId`和问题列表。

```js
issueListView:'',

join : function(args){
    this.repository(args);
    this.issues(args);
    this.commits(args);
},

issues : function(args){
    this.issueListView = new Vision.IssueListView({ el: 'ul#issues-list', projectId: args.projectId, issues: args.issues});
},
```

选择项目时，愿景将现在显示问题列表。

# 概述

我们现在已经完成了客户端的第一部分。我们实现了一个项目列表视图，允许我们添加、更新和删除项目。我们还实现了一个仓库列表视图，显示了我们访问令牌的仓库列表；这些仓库可以被分配到项目中。我们还显示了我们项目中所有仓库的提交和问题列表。在下一章中，我们将使用 Socket.IO 显示提交和问题的实时列表。
