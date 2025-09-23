# 第十二章：使用 Angular 和 TypeScript 进行前端开发

在本章中，我们将学习如何使用 Angular 和 TypeScript 开发前端网页应用程序。就像在上一章一样，我们将尝试使用我们之前实现的后端 Node.js 应用程序。我们将开发的应用程序是上一章中开发的前端应用程序的克隆。

应用程序的功能和样式将保持一致。然而，实现方式将呈现一些显著差异，因为我们打算使用 Angular 而不是 React 作为我们的前端应用程序开发框架。

我们不会在本章中详细介绍我们将要实现的应用程序的需求，因为我们已经在上一章中讨论过了。

# 使用 Angular 进行工作

Angular 是一个库，允许我们实现网页用户界面。在本章中，我们将使用 Angular 创建一个小型前端应用程序。

如我们在上一章所学，Node.js 中的 JavaScript 环境和网页浏览器环境相当不同。浏览器环境不支持原生的模块，加载时间也是影响前端应用程序性能的主要因素之一，这就是为什么我们在开发前端网页应用程序时需要像 webpack 这样的模块打包器。

在本章中，我们将像本书中一直做的那样使用 Webpack。我们将使用以下 Webpack 配置。它与上一章中使用的配置几乎相同，但我们做了一些修改：

```js
const { CheckerPlugin } = require("awesome-typescript-loader"); 
const webpack = require("webpack"); 
const ExtractTextPlugin = require("extract-text-webpack-plugin"); 
const CopyWebpackPlugin = require("copy-webpack-plugin"); 
const path = require ("path"); 

const corePlugins = [ 
    new CheckerPlugin(), 
    new webpack.DefinePlugin({ 
        "process.env.NODE_ENV": JSON.stringify(process.env.NODE_ENV || "development") 
    }), 
    new ExtractTextPlugin({ 
        filename: "[name].css", 
        allChunks: true 
    }), 
    new CopyWebpackPlugin([ 
        { from: "./web/frontend/index.html", to: "index.html" } 
    ]), 
```

我们介绍了`CommonChunkPlugin`。此插件用于识别重复或匹配给定规则的代码片段。当代码片段满足条件时，它将从主应用程序包中提取并添加到名为`vendor`的第二个包中。在这种情况下，我们将所有位于`node_modules`文件夹下的代码片段移动到供应商包中，这意味着我们将最终得到两个包，一个用于应用程序，一个用于第三方库：

```js
    new webpack.optimize.CommonsChunkPlugin({ 
        name: "vendor", 
        minChunks: (module) => { 
            return module.context && module.context.includes("node_modules"); 
        } 
    }) 
]; 

const devPlugins = []; 

const prodPlugins = [ 
    new webpack.optimize.UglifyJsPlugin({ output: { comments: false } }) 
]; 

const isProduction = process.env.NODE_ENV === "production"; 
const plugins = isProduction ? corePlugins.concat(prodPlugins) : corePlugins.concat(devPlugins); 
```

我们还引入了一个额外的应用程序入口点。我们导入`zone.js`模块。这个模块是 Zones API 的 polyfill，它是一种拦截和跟踪异步工作的机制。在 Zone.js 文档中，区域被定义为如下：

“区域是一个全局对象，它配置了如何拦截和跟踪异步回调的规则。区域有以下职责：

拦截异步任务调度

在异步操作中包装回调以进行错误处理和区域跟踪

提供将数据附加到区域的方法

提供特定上下文的最后帧错误处理

（拦截阻塞方法）

在其最简单的形式中，一个 Zone 允许拦截异步操作的调度和调用，并在异步任务之前以及之后执行额外的代码。

我们需要 Zone.js，因为它是 Angular 的依赖之一。Webpack 配置的其余部分没有其他重大差异：

```js
module.exports = { 
    entry: [ 
        "zone.js/dist/zone", 
        "./web/frontend/main.ts" 
    ], 
    devServer: { 
        inline: true 
    }, 
    output: { 
        filename: "[name].js", 
        chunkFilename: "[name]-chunk.js", 
        publicPath: "/public/", 
        path: path.resolve(__dirname, "public") 
    }, 
    devtool: isProduction ? "source-map" : "eval-source-map", 
    resolve: { 
        extensions: [".webpack.js", ".ts", ".tsx", ".js"] 
    }, 
    module: { 
        rules: [ 
            { 
                enforce: "pre", 
                test: /.js$/, 
                loader: "source-map-loader", 
                exclude: [/node_modules/] 
            }, 
            { 
                test: /.(ts|tsx)$/, 
                loader: "awesome-typescript-loader", 
                exclude: [/node_modules/] 
            }, 
            { 
                test: /.scss$/, 
                use: ExtractTextPlugin.extract({ 
                    fallback: "style-loader", 
                    use: ["css-loader", "resolve-url-loader", "sass-loader"] 
                }) 
            } 
        ] 
    }, 
    plugins: plugins 
}; 
```

请参阅第九章，*自动化你的开发工作流程*，以了解更多关于 webpack 的信息。

# 关于示例应用程序

在本章中，我们将再次实现前一章中实现的应用程序。然而，我们将使用 Angular 而不是 React 和 MobX 作为我们的应用程序开发框架。我们将尝试尽可能接近地实现应用程序的副本，以便我们能够比较这两个框架。

应用程序包含在配套源代码中，它是一个非常小的 Web 应用程序，允许我们管理电影和演员的数据库。我们在这里不会详细解释应用程序的功能，因为我们已经在前一章中解释过了。

请参阅第十一章，*使用 React 和 TypeScript 进行前端开发*，以了解更多关于应用程序的功能和需求。

请参阅配套源代码，以获取应用程序的完整源代码及其配置，包括整个`package.json`文件等。

# 使用 Node.js 提供 Angular 应用程序

就像我们在前一章中所做的那样，我们需要配置 Node.js 来提供我们的前端 Web 应用程序的文件。我们使用 Express 静态中间件来实现这一功能，就像我们在前一章中所做的那样。

请参阅第十一章，*使用 React 和 TypeScript 进行前端开发*，以了解更多关于 Express 静态中间件的信息。特别是，请参阅*使用 Node.js 提供 React 应用程序*部分。

# 引导 Angular 应用程序

前端应用程序的入口点在`web/frontend/main.ts`文件中：

```js
import { platformBrowserDynamic } from "@angular/platform-browser-dynamic"; 
import { AppModule } from "./app.module"; 

platformBrowserDynamic().bootstrapModule(AppModule).catch((err) => { 
  console.error(err); // tslint:disable-line 
}); 
```

我们使用`platformBrowserDynamic`模块来引导我们的应用程序。我们通过调用`bootstrapModule`方法并传递我们的应用程序中的主模块作为参数来实现这一点。我们将在下一节中了解更多关于模块的内容。

在本节中，我们将重点关注`platformBrowserDynamic`模块。可以使用以下方式使用`npm`安装`platform-browser-dynamic`模块：

```js
npm install --save platform-browser-dynamic
```

我们使用`platformBrowserDynamic`是因为我们期望我们的应用程序在浏览器环境中执行。Angular 应用程序也可以在 Node.js 这样的服务器端环境中执行，但这需要稍微不同的引导配置。在 Node.js 中执行 Angular 应用程序可以用来提高应用程序的初始加载时间。我们不会在本章中介绍这个主题，因为它是一个高级特性，而本章的目的只是介绍 Angular。

如果您想了解更多关于 Angular 服务器端渲染的信息，请参阅[`angular.io/guide/universal`](https://angular.io/guide/universal)文档。

# 与 NgModules 一起工作

在前面的章节中，我们使用了一个名为`AppModule`的模块来引导我们的 Angular 应用程序。`AppModule`位于`web/frontend/app.module.ts`文件中，其内容如下：

```js
import { NgModule } from "@angular/core"; 
import { BrowserModule } from "@angular/platform-browser"; 
import { CommonModule } from "@angular/common"; 
import { AppRoutingModule } from "./app-routing.module"; 
import { AppComponent } from "./app.component"; 
import { LayoutModule } from "./config/layout.module"; 
import "../../node_modules/bootstrap/scss/bootstrap.scss"; 
import "./app.scss"; 

@NgModule({ 
    bootstrap: [AppComponent], 
    declarations: [AppComponent], 
    imports: [ 
        BrowserModule, 
        CommonModule, 
        AppRoutingModule, 
        LayoutModule 
    ] 
}) 
export class AppModule { 
} 
```

`AppModule`是应用程序的入口点，它提供了访问应用程序中其他所有元素的方法。Angular 文档将模块定义为如下：

`@NgModule`装饰器标记了一个类。@NgModule 接受一个元数据对象，该对象描述了如何编译组件的模板以及如何在运行时创建注入器。它识别模块自己的组件、指令和管道，通过`exports`属性使其中一些公开，以便外部组件可以使用它们。@NgModule 还可以向应用程序依赖注入器添加服务提供者。

模块是组织应用程序和通过外部库扩展其功能的一种很好的方式。Angular 库是 NgModules，例如 FormsModule、HttpClientModule 和 RouterModule。许多第三方库也作为 NgModules 提供。NgModules 将组件、指令和管道整合成功能块，每个块都专注于一个功能区域、应用程序业务领域、工作流程或常见的实用工具集合。

模块允许我们将功能分组；我们可以将某些功能所需的所有元素（组件、服务等）组合到一个模块中。

`@NgModule`装饰器允许我们设置某些设置。以下列表定义了伴随源代码中包含的应用程序使用的某些字段的用途：

+   `bootstrap`字段用于声明在引导过程中必须作为根组件的组件

+   `declarations`字段用于声明在 Angular 模板中将使用哪些组件

+   `imports`字段用于在模块内部使其他组件可用

+   `exports`字段用于使组件对其他模块可用

+   `providers`字段用于配置依赖注入绑定

需要明确指出，Angular 的`@NgModule`导入/导出和 ES6 导入/导出模块是完全不同的概念。

请参阅[`angular.io/guide/ngmodules`](https://angular.io/guide/ngmodules)中的文档，以了解更多关于模块的信息。

# 与 Angular 组件一起工作

在本节中，我们将学习如何使用组件。我们将学习如何实现组件和路由，以及其他概念，例如如何在 Angular 应用程序中实现依赖注入。

# 我们的第一个组件

在本节中，我们将查看我们的第一个 Angular 组件。在本章的早期部分，我们学习了如何引导 Angular 应用程序，并使用了 `AppModule`。后来，我们了解到 `AppModule` 使用 `AppComponent` 作为我们应用程序的根组件。现在，我们将查看 `AppComponent`：

```js
import { Component } from "@angular/core"; 

@Component({ 
    selector: "app-root", 
    template: ` 
    <app-layout></app-layout>`, 
}) 
export class AppComponent { 
} 
```

如您所见，在 Angular 中，组件是一个带有 `@Component` 装饰器的类。`@Component` 装饰器接受一些设置。

在这种情况下，我们使用 `selector` 设置来声明在模板中引用此组件时使用的名称。`AppComponent` 是我们应用程序的根组件；这意味着它必须作为我们的 `index.html` 页面的根元素显示。我们可以通过在我们的 `index.html` 页面中添加对组件选择器的引用来实现这一点：

```js
<body> 
    <app-root>Loading...</app-root> 
    <script src="img/vendor.js"></script> 
    <script src="img/main.js"></script> 
  </body> 
```

当页面加载时，它将在 `<app-root>` DOM 元素中首先显示“加载中...”。一旦加载了供应商和主包文件，Angular 应用程序将被执行，`bootstrap` 函数将在 `<app-root>` DOM 元素中渲染 `AppComponent` 的模板，这将导致“加载中...”标签消失。

我们使用 `template` 设置来定义当组件渲染时希望生成的输出。在这种情况下，模板正在渲染另一个使用 `app-layout` 作为其选择器的组件。我们内联定义了模板，但也可以在单独的 HTML 文件中定义模板，并通过使用 `templateUrl` 设置来引用它。

需要注意的是，我们只能在模板中使用组件，前提是这两个组件都在 `NgModule` 中正确配置，正如前文所述。

有时组件将使用额外的设置；我们不会在本章中解释所有可用的设置，因为我们的目标只是提供一个基本的 Angular 介绍。

还值得一提的是，在 Angular 中，组件始终是一个类。在 React 中，可以以函数或类的形式实现组件，但在 Angular 中组件始终是类，这意味着 Angular 中没有无状态的函数式组件。

请参阅[`angular.io/guide/architecture-components`](https://angular.io/guide/architecture-components)中的文档，以了解更多关于 Angular 组件的信息。

# 组件和指令

现有的 Angular 文献中包含许多关于被称为指令的内容的引用。有时，指令被提及为可以与组件一起使用，好像它们是同一件事。事实是，组件是一种指令。以下内容摘自官方 Angular 文档：

Angular 中有三种类型的指令：

+   *组件：具有模板的指令。*

+   *结构指令：通过添加和删除 DOM 元素来改变 DOM 布局。*

+   *属性指令：改变元素、组件或另一个指令的外观或行为。*

我们在本章中不会学习如何创建自定义属性指令。然而，了解组件是一种指令是很重要的。

请参阅[`angular.io/guide/attribute-directives`](https://angular.io/guide/attribute-directives)中的文档以了解更多关于指令的信息。

# 数据绑定

在 Angular 中，我们使用数据绑定来协调应用程序的状态与屏幕上渲染的内容。Angular 支持三种类型的绑定，根据数据流的方向进行区分：

| **数据方向** | **语法** | **类型** |
| --- | --- | --- |

| 从数据源单向

到视图目标 | `{{expression}} [target]="expression"` |

+   插值

+   属性

+   属性

+   类

+   样式

|

| 从视图目标单向

到数据源 | `(target)="statement"` | 事件 |

| 双向 | `[(target)]="expression"` | 双向 |
| --- | --- | --- |

除了插值之外的其他绑定类型在等号左侧有一个目标名称，周围有标点符号（`[]`和`()`）。

请参阅[`angular.io/guide/template-syntax`](https://angular.io/guide/template-syntax)中的文档以了解更多关于数据绑定语法的知识。

# 使用@Attribute 和@Input 一起工作

在上一节中，我们了解到`AppComponent`渲染了`AppLayout`组件。在本节中，我们将探讨`AppLayout`：

```js
import { Component, OnInit } from "@angular/core"; 
import { Route } from "../components/header.component"; 

@Component({ 
  selector: "app-layout", 
  template: ` 
    <div> 
        <app-header 
            bg="primary" 
            title="TsMovies" 
            rootPath="" 
            [links]="appRoutes" 
        ></app-header> 
        <main> 
            <router-outlet></router-outlet> 
        </main> 
    </div> 
  ` 
}) 
export class LayoutComponent { 
    public appRoutes: Route[] = [ 
        { label: "Movies", path: "movies" }, 
        { label: "Actors", path: "actors" } 
    ]; 
} 
```

`Layout`组件使用`app-layout`选择器并声明了一个内联模板。该模板使用了具有`app-header`和`router-outlet`选择器的另外两个组件。我们现在将忽略具有`router-outlet`选择器的组件，因为我们将在后面的*使用 Angular 路由*部分中了解更多关于它的内容。

让我们暂时关注具有选择器`app-header`的组件。正如我们可以在前面的代码片段中看到的那样，一些参数被传递给了`HeaderComponent`。然而，并非所有参数都以相同的方式传递。

我们有一些按以下方式传递的参数（从数据源到视图目标的单向数据绑定）：

```js
bg="primary" 
```

在这种情况下，我们将字符串值 primary 作为属性传递。我们还有一些按以下方式传递的参数：

```js
[links]="appRoutes" 
```

在这种情况下，我们将属性`appRoutes`的值绑定并传递给`AppHeader`组件。这意味着`appRoutes`值的任何更改都将触发`AppHeader`组件的重新渲染。

现在，我们将查看 `AppHeader` 组件，看看如何定义属性和输入：

```js
import { Component, Input, Attribute } from "@angular/core"; 

type BgColor = "primary" | "secondary" | "success" | 
               "danger" | "warning" | "info" | "light" | 
               "dark" | "white"; 

export interface Route { 
    label: string; 
    path: string; 
} 

@Component({ 
    selector: "app-header", 
    template: ` 
        <nav [ngClass]="navClass"> 
        <a class="navbar-brand" [routerLink]="rootPath" routerLinkActive="active"> 
            {{title}} 
        </a> 
        <ul class="navbar-nav"> 
            <li *ngFor="let link of links"> 
                <a class="navbar-brand" [routerLink]="link.path" routerLinkActive="active"> 
                    {{link.label}} 
                </a> 
            </li> 
        </ul> 
    </nav>` 
}) 
export class HeaderComponent { 

    public navClass!: string; 
    public title!: string; 
    public rootPath!: string; 
    @Input() public links!: Route[]; 

    public constructor( 
        @Attribute("bg") bg: BgColor, 
        @Attribute("title") title: string, 
        @Attribute("rootPath") rootPath: string, 
    ) { 
        this.navClass = `navbar navbar-expand-lg navbar-light bg-${bg}`; 
        this.title = title; 
        this.rootPath = rootPath; 
    } 
} 
```

`HeaderComponent` 接受一些属性，这些属性使用 `@Attribute` 装饰器定义。`HeaderComponent` 还接受一个输入，该输入使用 `@Input` 装饰器定义：

+   `@Input` 装饰器用于将值传递到组件中，或将数据从一个组件传递到另一个组件（通常是父到子）

+   `@Attribute` 目录用于检索组件主元素中可用的属性的常量值，并且它必须与组件构造函数的参数一起使用

# 使用结构化指令

在前面的章节中，我们讨论了 `HeaderComponent`。然而，我们跳过了一些关于其模板的细节。`HeaderComponent` 使用所谓的结构化指令：

```js
<li *ngFor="let link of links"> 
   // ... 
</li> 
```

结构化指令负责 HTML 布局。它们塑造或重塑 DOM 的结构，通常是通过添加、删除或操作元素。

请参阅[`angular.io/guide/structural-directives`](https://angular.io/guide/structural-directives) 上的文档，了解结构化指令的更多信息。

# 使用 `<ng-content>` 指令

我们可以使用 `ng-content` 指令来 <indexentry content="Angular components: directive, using">渲染组件的子元素。例如，以下组件可以用来在 <indexentry content=" directive:using">应用程序布局中定义一行。然而，当我们定义 `RowComponent` 时，我们不知道哪些内容将被放入行中。我们使用 `ng-content` 指令来引用尚未知的子组件：

```js
@Component({ 
    selector: "app-row", 
    template: ` 
        <div class="row"> 
            <ng-content></ng-content> 
        </div> 
    ` 
}) 
export class RowComponent {} 
```

然后，`RowComponent` 可以在模板中使用，如下所示：

```js
<app-row> 
    <h1>Title</h1> 
</app-row> 
<app-row> 
    <h2>Subtitle</h2> 
</app-row> 
```

# 与 @Output 和 EventEmitter 一起工作

在 Angular 中，我们可以使用一个值是事件发射器的属性来处理用户事件。该属性必须使用 `@Output` 装饰器进行装饰，如下面的代码片段所示：

```js
import { Component, EventEmitter, Input, Output } from "@angular/core"; 

@Component({ 
    selector: "app-text-field", 
    template: ` 
        <input 
            type="text" 
            className="form-control" 
            [id]="id" 
            [placeholder]="placeholder" 
            (input)="onEdit($event)" 
        /> 
    ` 
}) 
export class TextFieldComponent { 

    @Input() public id!: string; 
    @Input() public placeholder!: string; 
    @Output() public onChange = new EventEmitter<{k: string; v: string}>(); 

    public onEdit(event: any) { 
        const value = (event.target as any).value; 
        const key = (event.target as any).id; 
        this.onChange.emit({ v: value, k: key }); 
    } 

} 
```

在我们的模板中，我们已将一个事件与组件中的一个方法链接起来，如下所示（从视图目标到数据源的单向数据绑定）：

```js
(input)="onEdit($event)" 
```

`onEdit` 方法将接收一个事件对象，该对象允许访问目标（触发事件的 DOM 元素）。我们可以使用事件目标来访问 DOM 元素的属性。

最后，我们调用输出的 `emit` 方法来将一些数据作为输出传递给父组件：

```js
public onEdit(event: any) { 
    const value = (event.target as any).value; 
    const key = (event.target as any).id; 
    this.onChange.emit({ v: value, k: key }); 
} 
```

最后，父组件可以将其方法之一设置为 `onChange` 输出的事件处理器，如下所示：

```js
<app-text-field 
    [id]="'title'" 
    [placeholder]="'Title'" 
    (onChange)="edit($event)" 
></app-text-field> 
```

请参阅[`angular.io/guide/component-interaction`](https://angular.io/guide/component-interaction) 上的文档，了解 Angular 中事件处理器的更多信息。

# 与组件的主元素一起工作

在本节中，我们将演示如何使用组件中的`host`设置来控制组件宿主的渲染方式。当组件被渲染时，Angular 将始终创建一个与组件选择器名称匹配的 DOM 元素。这个 DOM 元素被称为宿主。例如，看一下以下组件：

```js
@Component({ 
    selector: "app-row", 
    template: ` 
        <div class="row"> 
            <ng-content></ng-content> 
        </div> 
    ` 
}) 
export class RowComponent {} 
```

它可以被其他组件消费，如下所示：

```js
<app-row> 
    Hello! 
</app-row> 
```

然而，它将被渲染为：

```js
<app-row> 
    <div class="row"> 
        Hello! 
    </div> 
</app-row> 
```

如您所见，有一个额外的 DOM 节点。有时，额外的节点可能会导致一些布局问题，如果我们能够控制宿主渲染以实现以下输出，那就更好了：

```js
<app-row class="row"> 
    Hello! 
</app-row> 
```

我们可以通过在声明组件时使用`host`设置来实现这一点：

```js
@Component({ 
    host: { 
        "[class]": "'row'" 
    }, 
    selector: "app-row", 
    template: ` 
        <ng-content></ng-content> 
    ` 
}) 
export class RowComponent {} 
```

# 与 Angular 路由一起工作

在本章的早期部分，我们提到了一个名为`router-outlet`的组件。该组件被`Layout`组件如下使用：

```js
<main> 
    <router-outlet></router-outlet> 
</main> 
```

然而，这个组件不是我们定义的，因为它是由`@angular/router` npm 模块定义的。为了使用该模块，我们必须导入它并声明一个导出`RouteModule`的`NgModule`。我们还必须声明路由的配置。配置是一个将给定路径与给定组件关联的映射或字典：

```js
import { NgModule } from "@angular/core"; 
import { Routes, RouterModule } from "@angular/router"; 
import { HomePageComponent } from "./pages/homepage.component"; 
import { MoviesPageComponent } from "./pages/moviespage.component"; 
import { ActorsPageComponent } from "./pages/actorspage.component"; 

export const appRoutes: Routes = [ 
    { path: "", component: HomePageComponent }, 
    { path: "movies", component: MoviesPageComponent }, 
    { path: "actors", component: ActorsPageComponent } 
]; 

@NgModule({ 
    exports: [RouterModule], 
    imports: [ 
        RouterModule.forRoot( 
          appRoutes, 
          { useHash: false } 
      ) 
    ] 
}) 

export class AppRoutingModule {} 
```

当浏览器 URL 与路由配置中的路径之一匹配时，相应的组件将被渲染到`router-outlet`组件中：

```js
<main> 
    <router-outlet></router-outlet> 
</main> 
```

我们可以使用`routerLink`触发 URL 的变化：

```js
<a class="navbar-brand" [routerLink]="link.path" routerLinkActive="active"> 
    {{link.label}} 
</a> 
```

我们已经提供了我们希望导航到的路径以及当链接处于活动状态时要使用的 CSS 类。当我们点击路由链接时，浏览器 URL 将会改变，并且路由器将在`router-outlet`组件下渲染匹配的组件。

# Angular 组件生命周期钩子

Angular 允许我们声明多个组件生命周期钩子。例如，在伴随源代码中，`Movie`组件扩展了`OnInit`接口，该接口声明了`ngOnInit`方法。`ngOnInit`方法是 Angular 中可用的组件生命周期钩子之一：

+   组件类的构造函数在调用任何其他组件生命周期钩子之前被调用。构造函数是注入依赖的最佳位置。

+   `ngOnInit`方法在构造函数之后以及`ngOnChange`第一次触发后立即被调用，这是初始化工作的完美时机。

+   当绑定属性的值发生变化时，`ngOnChanges`方法首先被调用。它会在输入属性的值每次变化时执行。

+   在组件实例最终被销毁之前，将调用`ngDestroy`方法。这是清理组件（例如，取消后台任务）的完美位置。

还有更多的生命周期钩子可用，但它们超出了本书的范围。

请参阅 Angular 文档[`angular.io/guide/lifecycle-hooks`](https://angular.io/guide/lifecycle-hooks)，以了解所有可用的生命周期钩子。

# 与服务一起工作

在 Angular 中，使用服务与 REST API 或其他资源（如 localStorage API）交互是一种常见的做法。下面的类定义了一个名为`MovieService`的服务，它可以用来向后端 Node.js 应用程序发送 HTTP 请求。

服务只是一个类，它不需要任何特殊的装饰器。然而，以下代码片段使用了`@Injectable`装饰器，因为它将被注入到`MovieComponent`中。我们将在*Angular 中的依赖注入*部分学习更多关于依赖注入的内容。

以下方法使用 Fetch API 对服务器执行一些 HTTP 请求。有一个方法可以从电影 REST API 获取所有电影：

```js
import { Injectable } from "@angular/core"; 
import { MovieInterface } from "../../universal/entities/movie"; 
import * as interfaces from "../interfaces"; 

@Injectable() 
export class MovieService implements interfaces.MovieService { 

    public async getAll() { 
        return new Promise<MovieInterface[]>(async (res, rej) => { 
            try { 
                const response = await fetch("/api/v1/movies/", { method: "GET" }); 
                const movs: MovieInterface[] = await response.json(); 
                // We use setTimeout to simulate a slow request 
                // this should allow us to see the loading component 
                setTimeout( 
                    () => { 
                        res(movs); 
                    }, 
                    1500 
                ); 
            } catch (error) { 
                rej(error); 
            } 
        }); 
    } 
```

此外，还有一个方法可以创建一个新的电影：

```js
    public async create(movie: Partial<MovieInterface>) { 
        const response = await fetch( 
            "/api/v1/movies/", 
            { 
                body: JSON.stringify(movie), 
                headers: { 
                    "Accept": "application/json, text/plain, */*", 
                    "Content-Type": "application/json" 
                }, 
                method: "POST" 
            } 
        ); 
        const newMovie: MovieInterface = await response.json(); 
        return newMovie; 
    } 
```

此外，还有一个方法可以删除电影：

```js
    public async delete(id: number) { 
        const response = await fetch(`/api/v1/movies/${id}`, { method: "DELETE" }); 
        await response.json(); 
    } 

} 
```

在下一节中，我们将学习电影服务是如何被`Movie`组件消费的。

# 智能组件和哑组件

Angular 组件不像 React 那样在属性和状态之间划清界限，但我们可以仍然使用相同的思维模型。一个组件渲染一些数据。如果数据被组件修改，我们谈论的是智能组件。如果组件只读取数据，我们谈论的是哑组件。

就像我们在 React 示例中所做的那样，我们通过使用多个目录组织我们的项目，以便我们可以非常清晰地区分智能组件和哑组件。`components`目录只包含哑组件，而`pages`目录包含智能组件。

在 Angular 中，大多数情况下，哑组件不需要生命周期钩子，它们也不需要服务。另一方面，智能组件很可能需要一些服务。

以下代码片段声明了一个名为`MoviePages`的智能组件。在我们的 Angular 应用程序中，页面是智能组件，而组件仅仅是哑组件：

```js
import { Component, OnInit, Inject } from "@angular/core"; 
import { MovieInterface } from "../../universal/entities/movie"; 
import * as interfaces from "../interfaces"; 
import { MOVIE_SERVICE } from "../config/types"; 

function isValidNewMovie(o: any) { 
    if ( 
        o === null || 
        o === undefined || 
        // new movies don't have ID 
        o.id !== undefined || 
        typeof o.title !== "string" || 
        isNaN(o.year) 
    ) { 
        return false; 
    } 
    return true; 
} 
```

该组件渲染电影列表：

```js
@Component({ 
    selector: "movies-page", 
    template: ` 
        <app-container> 
            <app-row> 
                <app-column width="12"> 
                    <div style="text-align: right; margin-bottom: 10px"> 
                        <app-button (clicked)="focusEditor()"> 
                            Add Movie 
                        </app-button> 
                    </div> 
                </app-column> 
            </app-row> 
            <app-row> 
                <app-column width="12"> 
                    <app-list-group [isLoaded]="isLoaded" [errorMsg]="fetchErrorMsg"> 
                        <app-list-group-item *ngFor="let movie of movies"> 
                            <app-row> 
                                <app-column width="8"> 
                                    <h5>{{movie.title}}</h5> 
                                    <p>{{movie.year}}</p> 
                                </app-column> 
                                <app-column width="4" style="text-align: right"> 
                                    <app-button kind="danger" (clicked)="focusDeleteDialog(movie.id)"> 
                                        Delete 
                                    </app-button> 
                                </app-column> 
                            </app-row> 
                        </app-list-group-item> 
                    </app-list-group> 
                </app-column> 
            </app-row> 
```

此组件还渲染一个模态窗口，允许我们创建一个电影：

```js
            <div *ngIf="editorValue"> 
                <app-modal 
                    [title]="'Movie Editor'" 
                    [acceptLabel]="'Save'" 
                    [cancelLabel]="'Cancel'" 
                    [error]="saveStatus" 
                    (onCancel)="focusOutEditor()" 
                    (onAccept)="saveMovie()" 
                > 
                    <form> 
                        <app-text-field 
                            [id]="'title'" 
                            [title]="'Title'" 
                            [placeholder]="'Title'" 
                            [errorMsg]="isValidTitle" 
                            (onChange)="edit($event)" 
                        ></app-text-field> 
                        <app-text-field 
                            [id]="'year'" 
                            [title]="'Year'" 
                            [placeholder]="'Year'" 
                            [errorMsg]="isValidYear" 
                            (onChange)="edit($event)" 
                        ></app-text-field> 
                    </form> 
                </app-modal> 
            </div> 
```

此组件还渲染一个模态窗口，允许我们确认我们希望从数据库中删除电影：

```js
            <div *ngIf="deleteMovieId !== null"> 
                <app-modal 
                    [title]="'Delete?'" 
                    [acceptLabel]="'Delete'" 
                    [cancelLabel]="'Cancel'" 
                    [error]="deleteStatus" 
                    (onCancel)="focusOutDeleteDialog()" 
                    (onAccept)="deleteMovie()" 
                > 
                    Are you sure? 
                </app-modal> 
            </div> 
        </app-container> 
    ` 
}) 
```

`MoviesPageComponent`是一个智能组件。正如我们可以在以下代码片段中看到的那样，它持有并管理在其模板中使用的所有哑组件所需的所有状态：

```js
export class MoviesPageComponent implements OnInit { 

    // Contains the movies that have been already loaded from the server 
    public movies: MovieInterface[]; 

    // Used to represent the status of the HTTP GET calls 
    public isLoaded!: boolean; 

    // Display error if loading fails 
    public fetchErrorMsg: null | string; 

    // Used to represent the status of the HTTP DELETE call 
    public deleteStatus: null | string; 

    // Used to represent the status of the HTTP POST and HTTP PUT calls 
    public saveStatus: null | string; 

    // Used to display the confirmation dialog before deleting a movie 
    // null hides the modal and number displays the modal 
    public deleteMovieId: null | number; 

    // Used to hold the values of the movie editor or null when nothing is being edited 
    public editorValue: null | Partial<MovieInterface>; 
    public isValidTitle!: null | string; 
    public isValidYear!: null | string; 

    public movieService!: interfaces.MovieService; 
```

`MovieService`被注入到组件中。我们现在可以忽略这方面的细节，因为将在下一节中解释：

```js
    public constructor( 
        @Inject(MOVIE_SERVICE) movieService: interfaces.MovieService 
    ) { 
        this.movieService = movieService; 
        this.movies = []; 
        this.fetchErrorMsg = null; 
        this.isLoaded = false; 
        this.deleteStatus = null; 
        this.saveStatus = null; 
        this.deleteMovieId = null; 
        this.editorValue = null; 
        this.isValidTitle = null; 
        this.isValidYear = null; 
    } 
```

然后，我们使用`ngOnInit`事件钩子来触发初始数据获取：

```js
    public async ngOnInit() { 
        this.isLoaded = false; 
        try { 
            this.movies = await this.movieService.getAll(); 
            this.isLoaded = true; 
            this.fetchErrorMsg = null; 
        } catch (err) { 
            this.isLoaded = true; 
            this.fetchErrorMsg = "Loading failed!"; 
        } 
    } 
```

在声明属性、构造函数和组件的`ngOnInit`事件之后，我们将声明一些方法。正如您可以在以下代码片段中看到的那样，这些方法用于修改应用程序的状态：

```js
    public focusEditor() { 
        this.editorValue = {}; 
    } 

    public focusOutEditor() { 
        this.editorValue = null; 
    } 

    public focusDeleteDialog(id: number) { 
        this.deleteMovieId = id; 
    } 

    public focusOutDeleteDialog() { 
        this.deleteMovieId = null; 
    } 
    public edit(keyVal: any) { 
        const movie = { 
            ...(this.editorValue || {}), 
            ...{[keyVal.k]: keyVal.v} 
  }; 
        if (movie.title) { 
            this.isValidTitle = (movie.title && movie.title.length) > 0 ? null : "Title cannot be empty!"; 
        } 
        if (movie.year) { 
            this.isValidYear = isNaN(movie.year) === false ? null : "Year must be a number!"; 
        } 
        this.editorValue = movie; 
    } 
```

在上一章中，我们学习了关于 MobX 架构的基本知识。MobX 架构与 Angular 架构之间存在一些显著差异：

+   在 MobX 中，应用程序状态属于`Store`。智能组件通过动作与`Store`通信。`Store`是最终修改状态的实体，而不是智能组件。

+   在 Angular 中，应用程序状态属于智能组件，这是最终修改状态的实体。

在 Angular 中，我们使用服务来执行 Ajax 调用；另一方面，在 MobX 中，我们在`Store`内部执行 Ajax 调用。这个区别很小，因为我们可以在 MobX 中创建一个服务来执行 Ajax 调用。然后`Store`可以与该服务通信。这里的关键点是状态管理的差异。

以下方法使用`MovieService`执行一些 Ajax 调用并修改`MovieComponent`的状态：

```js
    public async saveMovie() { 
        if (isValidNewMovie(this.editorValue)) { 
            const newMovie = await this.movieService.create(this.editorValue as any); 
            this.movies.push(newMovie); 
            this.saveStatus = null; 
            this.editorValue = null; 
        } else { 
            this.saveStatus = "Invalid movie!"; 
        } 
    } 

    public async deleteMovie() { 
        try { 
            if (this.deleteMovieId) { 
                await this.movieService.delete(this.deleteMovieId); 
                this.movies = this.movies.filter((m) => m.id !== this.deleteMovieId); 
                this.deleteStatus = null; 
                this.deleteMovieId = null; 
            } 
        } catch (err) { 
            this.deleteStatus = "Cannot delete movie!"; 
        } 
    } 

} 
```

# Angular 中的依赖注入

Angular 中的依赖注入要求我们使用`InjectionToken`类定义一些唯一的标识符。注入令牌是用于在运行时表示类型的唯一标识符。Angular 中`InjectionToken`的概念与 InversifyJS 中的符号概念非常相似：

```js
import { InjectionToken } from "@angular/core"; 
import { MovieService, ActorService } from "../interfaces"; 

export const ACTOR_SERVICE = new InjectionToken<MovieService>("ActorService"); 
export const MOVIE_SERVICE = new InjectionToken<MovieService>("MovieService"); 
```

在创建`InjectionToken`之后，我们必须使用`@injectable`装饰器装饰我们希望注入的类，如下面的代码片段所示：

```js
import { InjectionToken } from "@angular/core"; 
// ... 
@Injectable() 
export class MovieService implements interfaces.MovieService { 
    // ... 
```

我们还必须在`InjectionToken`和它所代表的类型实现之间声明一个绑定。这可以通过在声明`NgModule`时使用`providers`设置来实现，如下所示：

```js
import { NgModule } from "@angular/core"; 
import { CommonModule } from "@angular/common"; 
import { MoviesPageComponent } from "./moviespage.component"; 
import { ComponentsModule } from "../components/components.module"; 
import { MovieService } from "../services/movie_service"; 
import { MOVIE_SERVICE } from "../config/types"; 

@NgModule({ 
    declarations: [ 
        MoviesPageComponent 
    ], 
    exports: [ 
        MoviesPageComponent 
    ], 
    imports: [CommonModule, ComponentsModule], 
    providers: [ 
        { provide: MOVIE_SERVICE, useClass: MovieService } 
    ] 
}) 
export class MoviesPageModule { 
} 
```

前面的代码片段将`InjectionToken MOVIE_SERVICE`绑定到`MovieService`实现。最后，我们可以使用`@Inject`装饰器在我们的一些 Angular 组件中声明一个依赖项：

```js
import { Component, Inject } from "@angular/core"; 
// ... 
public constructor( 
        @Inject(MOVIE_SERVICE) movieService: interfaces.MovieService 
    ) { 
        this.movieService = movieService; 
        // ... 
    } 
```

请参阅[`angular.io/guide/dependency-injection`](https://angular.io/guide/dependency-injection)中的文档，以了解更多关于 Angular 中依赖注入的信息。

# 摘要

在本章中，我们学习了基于组件的 Web 开发的基本原则以及如何使用 Angular。我们学习了如何启动 Angular 应用程序，如何实现路由，以及如何创建组件。

我们还学习了如何实现哑组件和智能组件，以及如何与服务和实现依赖注入一起工作。

在下一章中，我们将学习关于应用程序性能的内容。
