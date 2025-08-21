# 第十一章：Ionic 3

在《学习 Ionic，第二版》的最后一章中，我们将看一下 Ionic 框架的最新变化--Ionic 3。我们还将简要介绍 Angular 及其发布。在本章中，我们将讨论以下主题：

+   Angular 4

+   Ionic 3

+   Ionic 3 的更新

+   Ionic 2 与 Ionic 3

# Angular 4

自 Angular 2 发布以来，Angular 团队一直致力于使 Angular 成为一个稳定可靠的应用程序框架。2017 年 3 月 23 日，Angular 团队发布了 Angular 4。

什么？Angular 4？Angular 3 怎么了！！

简而言之，Angular 团队采用了语义化版本控制（[`semver.org/`](http://semver.org/)）来管理框架内所有包和依赖关系。在这个过程中，其中一个包（`@angular/router`）已经完全升级了一个主要版本，类似于以下情况，由于路由包的更改。：

| **框架** | **版本** |
| --- | --- |
| `@angular/core` | v2.3.0 |
| `@angular/compiler` | v2.3.0 |
| `@angular/compiler-cli` | v2.3.0 |
| `@angular/http` | v2.3.0 |
| `@angular/router` | V3.3.0 |

由于这种不一致性和为了避免未来的混淆，Angular 团队选择了 Angular 4 而不是 Angular 3。

此外，未来 Angular 版本的**暂定发布时间表**如下所示：

| **版本** | **发布日期** |
| --- | --- |
| Angular 5 | 2017 年 9 月/10 月 |
| Angular 6 | 2018 年 3 月 |
| Angular 7 | 2018 年 9 月/10 月 |

您可以在[`angularjs.blogspot.in/2016/12/ok-let-me-explain-its-going-to-be.html`](http://angularjs.blogspot.in/2016/12/ok-let-me-explain-its-going-to-be.html)上了解更多信息。

随着 Angular 4 的发布，一些重大的底层变化已经发生。以下是 Angular 4 的更新：

+   更小更快，生成的代码更小

+   `Animation`包的更新

+   `*ngIf`和`*ngFor`的更新

+   升级到最新的 TypeScript 版本

要了解更多关于此版本的信息，请参阅[`angularjs.blogspot.in/2017/03/angular-400-now-available.html`](http://angularjs.blogspot.in/2017/03/angular-400-now-available.html)。

由于 Ionic 遵循 Angular，他们已经将 Ionic 框架从版本 2 升级到版本 3，以将其基本 Angular 版本从 2 升级到 4。

# Ionic 3

随着 Angular 4 的发布，Ionic 已经升级并转移到了 Ionic 3。

Ionic 版本 3（[`blog.ionic.io/ionic-3-0-has-arrived/`](https://blog.ionic.io/ionic-3-0-has-arrived/)）增加了一些新功能，如 IonicPage 和 LazyLoading。他们还将基本版本的 Angular 更新到了版本 4，并发布了一些关键的错误修复。有关更多信息，请参阅 3.0.0 的变更日志：[`github.com/driftyco/ionic/compare/v2.3.0...v3.0.0`](https://github.com/driftyco/ionic/compare/v2.3.0...v3.0.0)。

Ionic 2 到 Ionic 3 的变化并不像我们从 Ionic 1 到 Ionic 2 看到的那样是破坏性的。Ionic 3 的变化更多地是增强和错误修复，这是在 Ionic 2 的基础上进行的。

# Ionic 3 的更新

现在，我们将看一下 Ionic 3 的一些关键更新。

# TypeScript 更新

对于 Ionic 3 的发布，Ionic 团队已经将 TypeScript 的版本更新到了最新版本。最新版本的 TypeScript 在构建时间和类型检查等方面有所增强。有关 TypeScript 更新的完整列表，请参阅 TypeScript 2.2 发布说明：[`www.typescriptlang.org/docs/handbook/release-notes/typescript-2-2.html`](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-2.html)。

# Ionic 页面装饰器

Ionic 页面装饰器有助于更好地实现深度链接。如果你还记得我们在第四章中的导航示例，*Ionic 装饰器和服务*，我们在使用 Nav Controller 推送和弹出页面时引用了实际的类名。

我在这里指的是`example9/src/pages/home/home.ts`：

```html
import { Component } from '@angular/core'; 
import { NavController } from 'ionic-angular'; 
import { AboutPage } from '../about/about'; 

@Component({ 
  selector: 'page-home', 
  templateUrl: 'home.html' 
}) 
export class HomePage { 

  constructor(public navCtrl: NavController) {} 

  openAbout(){ 
   this.navCtrl.push(AboutPage); 
  } 
} 

```

我们可以使用`@IonicPage`装饰器来实现相同的功能，如图所示。

让我们更新`example9/src/pages/about/about.ts`，如图所示：

```html
import { Component } from '@angular/core'; 
import { NavController, IonicPage } from 'ionic-angular'; 

@IonicPage({ 
   name : 'about' 
}) 
@Component({ 
  selector: 'page-about', 
  templateUrl: 'about.html' 
}) 
export class AboutPage { 

  constructor(public navCtrl: NavController) {} 

  goBack(){ 
   this.navCtrl.pop(); 
  } 
} 

```

请注意，`@IonicPage`装饰器已经添加到`@Component`装饰器中。现在，我们将更新`example9/src/pages/home/home.ts`，如图所示：

```html
import { Component } from '@angular/core'; 
import { NavController } from 'ionic-angular'; 

@Component({ 
  selector: 'page-home', 
  templateUrl: 'home.html' 
}) 
export class HomePage { 

  constructor(public navCtrl: NavController) {} 

  openAbout(){ 
   this.navCtrl.push('about'); 
  } 
} 

```

请注意`this.navCtrl.push()`的更改。现在，我们不再传递类的引用，而是传递了我们在`example9/src/pages/about/about.ts`中的`@IonicPage`装饰器属性的名称。此外，现在页面将在 URL 中添加名称，即[`localhost:8100/#/about`](http://localhost:8100/#/about)。

要了解更多关于 Ionic 页面装饰器的信息，请访问[`ionicframework.com/docs/api/navigation/IonicPage`](http://ionicframework.com/docs/api/navigation/IonicPage)。

还要查看 IonicPage 模块[`ionicframework.com/docs/api/IonicPageModule/`](http://ionicframework.com/docs/api/IonicPageModule/)，将多个页面/组件捆绑到一个子模块中，并在`app.module.ts`的`@NgModule`中引用相同的内容。

# 懒加载

懒加载是 Ionic 3 发布的另一个新功能。懒加载让我们在需要时才加载页面。这将改善应用程序的启动时间并提高整体体验。

您可以通过访问[`docs.google.com/document/d/1vGokwMXPQItZmTHZQbTO4qwj_SQymFhRS_nJmiH0K3w/edit`](https://docs.google.com/document/d/1vGokwMXPQItZmTHZQbTO4qwj_SQymFhRS_nJmiH0K3w/edit)来查看在 Ionic 应用程序中实现懒加载的过程。

在撰写本章时，Ionic 3 已经发布了大约一周。CLI 和脚手架应用程序中存在一些问题/不一致。希望这些问题在书籍发布时能够得到解决。

# Ionic 2 与 Ionic 3

在本书中，所有示例都是以 Ionic 2 为目标编写的。话虽如此，如果您使用 Ionic 3 开发您的 Ionic 应用程序，代码应该不会有太大变化。在所有脚手架应用程序中，您将注意到一个关键的区别是引入了 IonicPage 装饰器和 IonicPage 模块。

您可以随时参考 Ionic 文档，以获取有关这些 API 的最新版本的更多信息。

# 总结

通过这一点，我们结束了我们的 Ionic 之旅。

简而言之，我们从理解为什么选择 Angular、为什么选择 Ionic 和为什么选择 Cordova 开始。然后，我们看到移动混合应用程序的工作原理以及 Cordova 和 Ionic 的适用性。接下来，我们看了 Ionic 的各种模板，并了解了 Ionic 组件、装饰器和服务。之后，我们看了 Ionic 应用程序的主题设置。

接下来，我们学习了 Ionic Native，并了解了如何使用它。利用这些知识，我们构建了一个 Riderr 应用程序，该应用程序实现了 REST API，使用 Ionic Native 与设备功能进行交互，并让您感受到可以使用 Ionic 构建的完整应用程序的感觉。

之后，我们看了迁移 Ionic 1 应用程序到 Ionic 2 以及如何测试 Ionic 2 应用程序。在第十章，*发布 Ionic 应用程序*，我们看到了如何发布和管理我们的应用程序。

在本章中，我们看到了 Ionic 3 的关键变化。

查看附录获取更多有用信息和一些可以在生产应用程序中进行测试/使用的 Ionic 服务。
