# 第十三章. 使用 AngularJS 2 构建搜索引擎模板

要使用 Angular 2 构建**单页应用程序**（**SPAs**），我们需要学习如何在 Angular 2 中实现路由。Angular 2 自带内置的路由 API，它们非常强大、功能丰富且易于使用。在本章中，我们将构建一个基本的搜索引擎模板来演示 Angular 2 中的路由。我们不会构建一个完整的搜索引擎，因为这超出了本书的范围。我们将使用 Bootstrap 4 来设计搜索引擎模板。在本章结束时，你将能够舒适地使用 Angular 2 构建 SPAs。

在本章中，我们将涵盖以下主题：

+   Angular 2 中的路由

+   Angular 2 提供的内置 HTTP 客户端

+   使用`Chance.js`库生成随机文本数据

# 设置项目

按照以下步骤设置你的项目：

1.  在本章的练习文件中，你会找到两个目录，`initial`和`final`。`final`目录包含最终的搜索引擎模板，而`initial`目录包含快速开始构建搜索引擎模板的文件。

1.  在`initial`目录中，你会找到`app.js`和`package.json`文件。在`package.json`文件中，放置以下代码：

    ```js
    {
      "name": "SearchEngine-Template",
      "dependencies": {
        "express": "4.13.3",
        "chance": "1.0.3"    
      }
    }
    ```

    这里，我们将`Express.js`和`Chance.js`列为依赖项。Express 将用于构建 Web 服务器，而`Chance.js`将用于生成随机文本数据以填充模板的搜索结果。

1.  现在，在`initial`目录下运行`npm install`以下载包。

    在`initial`目录中，你会找到一个名为`public`的目录，其中将放置所有静态资源。在`public`目录中，你会找到`componentTemplates`、`css`、`html`和`js`目录。

    在`css`目录中，你会找到`bootstrap.min.css`；在`html`目录中找到`index.html`；最后，在`js`目录中找到`index.js`、`angular2-all.umd.js`、`angular2-polyfills.js`和`Rx.umd.js`。

1.  在`index.html`中，放置以下起始代码以加载 Angular、Bootstrap 和`index.js`文件：

    ```js
    <!doctype html>
    <html>
      <head>
        <title>Search Engine Template</title>
        <link rel="stylesheet" type="text/css" href="/css/bootstrap.min.css">
      </head>
      <body>

        <script src="img/angular2-polyfills.js"></script>
        <script src="img/Rx.umd.js"></script>
        <script src="img/angular2-all.umd.js"></script>
        <script src="img/index.js"></script>
      </body>
    </html>
    ```

    这段代码是自解释的。

1.  现在，在`app.js`文件中，放置以下代码：

    ```js
    var express = require("express");
    var app = express();

    app.use(express.static(__dirname + "/public"));

    app.get("/*", function(httpRequest, httpResponse, next){
      httpResponse.sendFile(__dirname + "/public/html/index.html");
    })

    app.listen(8080);
    ```

    在这里，大部分代码都是自解释的。我们只是简单地根据 HTTP 请求路径提供服务`index.html`。

# 配置路由和启动应用

在 SPA 中，我们应用的路由定义在前端。在 Angular 2 中，我们需要定义路径和与该路径关联的组件，该组件将被渲染为该路径。

我们将路由提供给根组件，根组件显示与路由绑定的组件。

让我们创建我们的搜索引擎模板的根组件和路由：

1.  将以下代码放置在`index.js`文件中，以创建根组件和路由：

    ```js
    var AppComponent = ng.core.Component({
      selector: "app",
      directives: [ng.router.ROUTER_DIRECTIVES],
      templateUrl: "componentTemplates/app.html"
    }).Class({
      constructor: function(){}
    })

    AppComponent = ng.router.RouteConfig([
        { path: "/", component: HomeComponent, name: "Home" },
        { path: "/search-result", component: SearchResultComponent, name: "SearchResult" },
        { path: "/*path", component: NotFoundComponent, name: "NotFound" }
    ])(AppComponent);

    ng.platform.browser.bootstrap(AppComponent, [
      ng.router.ROUTER_PROVIDERS,
      ng.core.provide(ng.router.APP_BASE_HREF, {useValue : "/" })
    ]);
    ```

1.  现在，在`componentTemplates`目录中创建一个名为`app.html`的文件，并将以下代码放入其中：

    ```js
    <nav class="navbar navbar-light bg-faded">
        <ul class="nav navbar-nav">
            <li class="nav-item">
                <a class="nav-link" 
                [routerLink]="['Home']">Home</a>
            </li>
        </ul>
    </nav>
    <router-outlet></router-outlet>
    ```

这段代码的工作原理如下：

1.  首先，我们创建根组件，命名为`AppComponent`。在创建根组件时，我们向其中添加了`ng.router.ROUTER_DIRECTIVES`指令，这使得我们可以使用`routerLink`指令。

1.  然后，我们使用`ng.router.RouteConfig`来配置我们应用程序的路由。我们将路由数组作为参数传递给`ng.router.RouteConfig`方法。一个路由由路径、组件和路由名称组成。路径可以是静态的、参数化的或通配符，就像 Express 路由路径一样。在这里，第一个路由是主页，第二个是显示搜索结果，最后一个是处理无效 URL，即未定义路由的 URL。`ng.router.RouteConfig`方法返回一个函数，该函数接受根组件并将其路由附加到它上。

1.  然后我们初始化应用程序。在初始化应用程序时，我们传递了`ng.router.ROUTER_PROVIDERS`提供者，它将被用于创建与路由相关的各种服务的实例。此外，我们提供了一个自定义提供者，当请求`ng.router.APP_BASE_HREF`服务的实例时，它返回`/`字符。`ng.router.APP_BASE_HREF`用于查找应用程序的基本 URL。

1.  在`AppComponent`模板中，我们正在显示一个导航栏。导航栏中有一个没有`href`属性的`anchor`标签；相反，我们使用`routerLink`指令来分配重定向链接，这样当点击时，不会进行完整的页面刷新，只会改变 URL 和组件。最后，`<router-outlet>`是用于根据当前 URL 显示组件的部分。

# 生成随机搜索结果

为了填充我们的模板，我们需要生成一些随机的搜索结果数据。为此，我们可以使用`Chance.js`库。我们将在服务器端生成随机数据，而不是在客户端，这样我们就可以稍后展示如何使用 Angular 2 进行 HTTP 请求。

`Chance.js`适用于客户端和服务器端 JavaScript。我们之前下载了`Chance.js`包，用于与`Node.js`一起使用。以下是生成随机数据的代码。将其放置在`app.js`文件中，在`/*`路由之上，这样`/*`就不会覆盖随机数据路由：

```js
var Chance = require("chance"),
chance = new Chance();
app.get("/getData", function(httpRequest, httpResponse, next){

  var result = [];

  for(var i = 0; i < 10; i++)
  {
    result[result.length] = {
      title: chance.sentence(),
      desc: chance.paragraph()
    }
  }

  httpResponse.send(result);
})
```

在这里，我们首先为`/getData`路径创建一个路由，该路由返回一个搜索结果数组作为响应。路由回调使用`chance.sentence()`生成随机标题，使用`chance.paragraph()`生成描述。

# 创建路由组件

让我们创建`HomeComponent`、`SearchResultComponent`和`NotFoundComponent`。在此之前，让我们创建一个用于显示搜索表单的组件。搜索表单将包含一个文本框和一个搜索按钮。按照以下步骤操作：

1.  将此代码放置在`index.js`文件中，在`AppComponent`代码之上：

    ```js
    var FormComponent = ng.core.Component({
      selector: "search-form",
      directives: [ng.router.ROUTER_DIRECTIVES],  
      templateUrl: "componentTemplates/search-form.html",
    }).Class({
      constructor: function(){},
      ngOnInit: function(){
        this.searchParams = {
          query: ""
        };

        this.keyup = function(e){
          this.searchParams = {
            query: e.srcElement.value
          };
        };
      }
    })
    ```

1.  现在，在`componentTemplates`目录中创建一个名为`search-form.html`的文件，并将此代码放入其中：

    ```js
    <div class="m-a-2 text-xs-center">
      <h1>Search for Anything</h1>
      <form class="form-inline m-t-1">
            <input (keyup)="keyup($event)" class="form-control" type="text" placeholder="Search">
            <a [routerLink]="['SearchResult', searchParams]">
                <button class="btn btn-success-outline" type="submit">Search</button>
            </a>
        </form>
    </div>
    ```

这就是代码的工作原理：

1.  首先，我们创建一个名为 `FormComponent` 的组件。它使用 `ng.router.ROUTER_DIRECTIVES` 指令。

1.  在组件模板中，我们显示一个 HTML 表单。该表单包含一个文本框和按钮。

1.  我们处理文本输入框的 `keyup` 事件，并将值存储在 `searchParams.query` 属性中。

1.  该按钮将重定向到 `SearchResult` 组件。注意，在这里我们将 `searchParams` 对象传递给 `routerLink`，在重定向时它将成为查询参数。

现在，让我们创建 `HomeComponent` 组件。该组件显示在主页上。它显示搜索表单。

下面是如何创建 `HomeComponent`：

1.  将此代码放置在 `index.js` 文件中，在 `AppComponent` 代码之上：

    ```js
    var HomeComponent = ng.core.Component({
      selector: "home",
      directives: [FormComponent],
      templateUrl: "componentTemplates/home.html",
    }).Class({
      constructor: function(){}
    })
    ```

1.  现在，创建一个名为 `search-form.html` 的文件，并将其放置在 `componentTemplates` 目录中：

    ```js
    <search-form></search-form>
    ```

    这里，`HomeComponent` 的代码是自我解释的。

1.  现在，让我们创建 `SearchResultComponent` 组件。该组件应显示搜索表单及其下方的搜索结果。它应通过向服务器发送 HTTP 请求来获取结果。以下是 `SearchResultComponent` 的代码。将其放置在 `index.js` 文件中，在 `AppComponent` 代码之上：

    ```js
    var SearchResultComponent = ng.core.Component({
      selector: "search-result",
      directives: [FormComponent],
      viewProviders: [ng.http.HTTP_PROVIDERS],
      templateUrl: "componentTemplates/searchResult.html"
    }).Class({
      constructor: [ng.router.RouteParams, ng.http.Http,
      function(params, http) {
          this.params = params;
        this.http = http;
        this.response = [];
      }],
      ngOnInit: function(){
        var q = this.params.get("query");
        this.http.get("getData").subscribe(function(res){
          this.response = JSON.parse(res._body);
        }.bind(this));
      }
    })
    ```

1.  现在，创建一个名为 `searchResult.html` 的文件，并将其放置在 `componentTemplates` 目录中。将此代码放置在文件中：

    ```js
    <style>
      ul
      {
          list-style-type: none;
      }
    </style>

    <div class="container">
      <search-form></search-form>
      <div class="m-a-2 text-xs-center">
          <ul>
            <li *ngFor="#item of response" class="m-t-2">
              <h4>{{item.title}}</h4>
              <p>{{item.desc}}</p>
            </li>
          </ul>
      </div>
    </div>
    ```

这就是代码的工作方式：

1.  这里，我们提供了 `ng,http.HTTP_PROVIDERS` 提供者，它在使用 Angular 2 提供的 HTTP 客户端服务时使用。使用 HTTP 客户端服务，我们可以发送 HTTP 请求。

1.  在构造函数属性中，我们注入了 HTTP 服务以及 `ng.router.RouteParams` 服务，该服务用于获取当前 URL 的查询参数。

1.  在 `ngOnInit` 方法中，你可以看到如何使用 HTTP 服务进行 `GET` 请求，以及如何使用 `ng.router.RouteParams` 服务获取查询参数。

1.  在组件模板中，我们使用 `ngFor` 指令显示获取的搜索结果。

### 注意

你可以在 [`angular.io/docs/ts/latest/guide/server-communication.html`](https://angular.io/docs/ts/latest/guide/server-communication.html) 了解 Angular 2 提供的 HTTP 服务。

现在，让我们创建 `NotFoundComponent`。以下是该组件的代码：

1.  将此代码放置在 `index.js` 文件中，在 `AppComponent` 代码之上：

    ```js
    var NotFoundComponent = ng.core.Component({
      selector: "name-search",
      templateUrl: "componentTemplates/notFound.html"
    }).Class({
      constructor: function(){}
    })
    ```

1.  现在，创建一个名为 `notFound.html` 的文件，并将其放置在 `componentTemplates` 目录中。将此代码放置在文件中：

    ```js
    <div class="container">
      <div class="m-a-2 text-xs-center">
        <h1>The page your are looking for is not found</h1>
      </div>
    </div>
    ```

    代码是自我解释的。

# 测试模板

要测试模板，我们将遵循以下步骤：

1.  在 `initial` 目录中，运行 `node app.js` 命令。

1.  现在，在浏览器中打开 `http://localhost:8080/` URL。你应该看到以下输出：![测试模板](img/B05154_13_01.jpg)

1.  现在，在搜索框中输入一些内容，然后点击 **搜索** 按钮。你应该看到以下输出：![测试模板](img/B05154_13_02.jpg)

1.  现在，在地址栏中输入一个无效的路径。你应该能看到以下输出：![测试模板](img/B05154_13_03.jpg)

# 路由生命周期方法

当路径与组件匹配时，Angular 2 激活该组件，当路径改变时，Angular 2 停用它。当我们说一个组件已被激活时，这意味着 Angular 2 已创建了一个组件的实例，即调用了组件的构造函数方法，而当我们说一个组件已被停用时，这意味着组件已被从 DOM 中移除，实例已被删除。

在激活或停用组件时调用的组件方法被称为路由生命周期方法。

下面是路由生命周期方法的列表：

+   `CanActivate`: 在激活组件之前调用此钩子。它应该返回一个布尔值或一个表示是否激活组件的承诺。

+   `routerOnActivate`: 此方法在组件被激活后调用。

+   `routerCanReuse`: 调用此方法以确定在下一个 URL 变更再次是相同 URL 时是否重用组件的先前实例。它应该返回一个布尔值或一个表示是否重用的承诺。它仅在之前已创建实例的情况下调用。

+   `routerOnReuse`: 如果组件正在被重用，则调用此方法。它在 `routerCanReuse` 之后调用。

+   `routerCanDeactivate`: 在停用组件之前调用此方法。它应该返回一个布尔值或一个表示是否停用组件的承诺。

+   `routerOnDeactivate`: 此方法在组件被停用后调用。

让我们看看路由生命周期方法的代码示例。将 `HomeComponent` 的代码替换为以下内容：

```js
var HomeComponent = ng.core.Component({
  selector: "home",
  directives: [FormComponent],
  templateUrl: "componentTemplates/home.html",
}).Class({
  constructor: function(){},
  routerOnActivate: function(){
    console.log("Component has been activated");
  },
  routerCanReuse: function(){
    console.log("Component can be reused");
    return true;
  },
  routerOnReuse: function(){
    console.log("Component is being reused");
  },
  routerCanDeactivate: function(){
    console.log("Component can be deactivated");
    return true;
  },
  routerOnDeactivate: function(){
    console.log("Component has been deactivated");
  }
})

HomeComponent = ng.router.CanActivate(function(){
  console.log("Component can be activated");
  return true;
})(HomeComponent);
```

现在，访问主页。在那里，再次点击主页按钮。现在，在搜索框中输入一些内容并点击**搜索**按钮。你将看到以下控制台输出：

```js
Component can be activated
Component has been activated
Component can be reused
Component is being reused
Component can be deactivated
Component has been deactivated

```

# 生产模式与开发模式

到目前为止，我们一直在开发模式下运行 Angular 2。开发模式和生产模式之间的区别在于，在开发模式下，Angular 2 在第一次运行后立即启动变更检测，如果在第一次和第二次运行之间有任何变化，则会记录一个**检查后值已更改**的错误。这有助于定位错误。

要启用生产模式，请将以下代码放置在 `ng.platform.browser.bootstrap()` 方法调用之上：

```js
ng.core.enableProdMode();
```

# 概述

在本章中，我们通过构建一个基本的搜索引擎模板学习了 Angular 2 的路由。在学习路由的深度知识的同时，我们还学习了 Angular 2 的 HTTP 客户端服务以及如何在 Angular 2 中切换到生产模式。

你现在应该已经熟悉了使用 Angular 2 构建任何类型的网络应用程序的前端。
