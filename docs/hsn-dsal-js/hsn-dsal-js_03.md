# 第三章：使用集合和映射加速应用程序

**集合**和**映射**是两种看似简单的数据结构，在最新版本的 ES6 中已经标准化。

在本章中，我们将涵盖以下主题：

+   为什么我们需要集合和映射？

+   何时以及如何使用集合和映射

+   ES6 中集合和映射的 API

+   用例

+   性能比较

# 探索集合和映射的起源

在我们试图了解如何在现实世界的应用程序中使用集合和映射之前，更有意义的是了解集合和映射的起源，以及为什么我们首先需要它们在 JavaScript 中。

直到 ES5 之前，传统数组不支持开发人员通常想要利用的一些主要功能：

+   承认它包含一个特定的元素

+   添加新元素而不产生重复。

这导致开发人员实现了自己的集合和映射版本，这在其他编程语言中是可用的。使用 JavaScript 的`Object`来实现集合和映射的常见方法如下：

```js
// create an empty object
var setOrMap = Object.create(null);

// assign a key and value
setOrMap.someKey = someValue;

// if used as a set, check for existence
if(setOrMap.someKey) {
    // set has someKey 
}

// if used as a map, access value
var returnedValue = setOrMap.someKey;
```

虽然使用`Object.create`创建集合或映射可以避免很多原型问题，但它仍然不能解决一个问题，那就是主要的`Key`只能是一个`string`，因为`Object`只允许键为字符串，所以我们可能会无意中得到值互相覆盖：

```js
// create a new map object
let map = Object.create(null);

// add properties to the new map object
let b = {};
let c = {};
map[b] = 10
map[c] = 20

// log map
Object [object Object]: 20
```

# 分析集合和映射类型

在实际使用集合和映射之前，我们需要了解何时以及何地需要使用它们。每种数据结构，无论是原生的还是自定义的，都有其自己的优势和劣势。

利用这些优势是非常重要的，更重要的是避免它们的弱点。为了理解其中一些弱点，我们将探讨集合和映射类型，以及它们为何需要以及在哪里使用。

主要有四种不同的集合和映射类型：

+   **映射**：一个键值对，其中键可以是一个`Object`或一个原始值，可以容纳任意值。

+   **WeakMap**：一个键值对，其中键只能是一个`Object`，可以容纳任意值。键是弱引用的；这意味着如果不使用，它们不会被阻止被垃圾回收。

+   **集合**：允许用户存储任何类型的唯一值的数据类型。

+   **WeakSet**：类似于集合，但维护一个弱引用。

# WeakMap 有多弱？

到目前为止，我们都知道什么是映射，以及如何添加键和值，至少在理论上。然而，如何确定何时使用映射，何时使用`WeakMap`呢？

# 内存管理

根据 MDN 的官方定义([`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/WeakMap))，`WeakMap`的官方定义如下：

WeakMap 对象是一个键/值对的集合，其中键是弱引用。键必须是对象，值可以是任意值。

重点在于*弱引用*。

在比较`Map`和`WeakMap`之前，了解何时使用特定的数据结构是至关重要的。如果您需要随时知道集合的键，或者需要遍历集合，那么您将需要使用`Map`而不是`WeakMap`，因为键是不可枚举的，也就是说，您无法获得后者中可用键的列表，因为它只维护了一个弱引用。

因此，自然而然地，前面的陈述应该在你的脑海中引起两个问题：

+   如果我总是使用映射会发生什么？

+   没什么，生活还在继续。你可能会或可能不会遇到内存泄漏，这取决于你如何使用你的映射。在大多数情况下，你会没事的。

+   什么是弱引用？

+   弱引用是一种允许对象引用的所有内容在所有引用者被移除时被垃圾回收的东西。困惑吗？很好。让我们看下面的例子来更好地理解它：

```js
var map = new Map();

(function() {
 var key =  {}; <- Object 
 map.set(key, 10); <- referrer of the Object 
    // other logic which uses the map

})(); <- IIFE which is expected to remove the referrer once executed
```

我们都知道 IIFE 主要用于立即执行函数并删除其作用域，以避免内存泄漏。在这种情况下，尽管我们已经将`key`和地图设置器包装在 IIFE 中，但`key`并没有被垃圾回收，因为在内部`Map`仍然保留对`key`及其值的引用：

```js
var myWeakMap = new WeakMap();

(function() {
 var key =  {};<- Object
 myWeakMap.set(key, 10);<- referrer of the Object

    // other logic which uses the weak map
})(); <- IIFE which is expected to remove the referrer once executed
```

当使用`WeakMap`*编写相同的代码时，*一旦执行 IIFE，键和该键的值将从内存中删除，因为键已经超出了作用域；这有助于将内存使用量保持在最低水平。

# API 差异

在标准操作方面，`Map`和`WeakMap`的 API 非常相似，例如`set()`和`get()`。这使得 API 非常直观，并包括以下内容：

+   `Map.prototype.size`：返回地图的大小；在典型对象上不可用，除非您循环并计数

+   `Map.prototype.set`：为给定键设置值并返回整个新地图

+   `Map.prototype.get`：获取给定键的值，如果未找到则返回 undefined

+   `Map.prototype.delete`：删除给定键的值并在删除成功时返回`true`，否则返回`false`

+   `Map.prototype.has`：检查地图中是否存在具有提供的键的元素；返回布尔值

+   `Map.prototype.clear`：清除地图；返回空

+   `Map.prototype.forEach`：循环遍历地图并访问每个元素

+   `Map.prototype.entries`：返回一个迭代器，您可以在其上应用`next()`方法以获取`Map`中下一个元素的值，例如`mapIterator.next().value`

+   `Map.prototype.keys`：类似于`entries`*；*返回一个迭代器，可用于访问下一个值

+   `Map.prototype.values`：类似于`key`；返回对值的访问

主要区别在于访问与`WeakMap`*相关的键和值的任何内容。*如前所述，由于在`WeakMap`的情况下存在枚举挑战，因此诸如`size()`、`forEach()`、`entries()`、`keys()`和`values()`等方法在`WeakMap`中不可用。

# 集合与 WeakSets

现在，我们了解了`WeakMap`或`WeakSet`*中 weak*一词的基本含义。预测集合的工作方式以及`WeakSet`与其不同并不是非常复杂。让我们快速看一下功能差异，然后转向 API。

# 了解 WeakSets

`WeakSet`与`WeakMap`非常相似；`WeakSet`可以容纳的值只能是对象，不能是原始值，就像`WeakMap`的情况一样。`WeakSets`也不可枚举，因此您无法直接访问集合中可用的值。

让我们创建一个小例子，了解`Set`和`WeakSet`之间的区别：

```js
var set = new Set();
var wset = new WeakSet();

(function() {

  var a = {a: 1};
  var b = {b: 2};
  var c = {c: 3};
  var d = {d: 4};

  set.add(1).add(2).add(3).add(4);
  wset.add(a).add(b).add(b).add(d);

})();

console.dir(set);
console.dir(wset);
```

一个重要的事情要注意的是`WeakSet`不接受原始值，只能接受与`WeakMap`键类似的对象。

前面代码的输出如下，这是从`WeakSet`中预期的。`WeakSet`不会保留元素超出持有它们的变量的寿命：

![](img/ccaeefe6-894e-48df-bc1b-b094d0e96b30.png)

如预期的那样，一旦 IIFE 终止，`WeakSet`就为空了。

# API 差异

在`WeakMap`地图的情况下，文档中记录的 API 差异与集合的情况非常接近：

+   `Set.prototype.size`：返回集合的大小

+   `Set.prototype.add`：为给定元素添加值并返回整个新集合

+   `Set.prototype.delete`：删除一个元素并在删除成功时返回`true`，否则返回`false`

+   `Set.prototype.has`：检查集合中是否存在元素并返回布尔值

+   `Set.prototype.clear`：清除集合并返回空

+   `Set.prototype.forEach`：循环遍历集合并访问每个元素

+   `Set.prototype.values`：返回一个迭代器，可用于访问下一个值

+   `Set.prototype.keys`：类似于 values—返回对集合中值的访问

另一方面，`WeakSet`不包含`forEach()`、`keys()`和`values()`方法，原因在先前讨论过。

# 用例

在开始使用用例之前，让我们创建一个基础应用程序，它将像我们在第一章中所做的那样，为每个示例重复使用。

以下部分是创建基本 Angular 应用程序的快速回顾：

# 创建 Angular 应用程序

在进入个别用例之前，我们将首先创建 Angular 应用程序，这将作为我们示例的基础。

按照给定的命令启动应用程序：

1.  安装 Angular CLI：

```js
npm install -g @angular/cli
```

1.  通过运行以下命令在您选择的文件夹中创建一个新项目：

```js
ng new <project-name>
```

1.  完成这两个步骤后，您应该能够看到新创建的项目以及所有相应的节点模块已安装并准备就绪。

1.  要运行您的应用程序，请从终端运行以下命令：

```js
ng serve
```

# 为您的应用程序创建自定义键盘快捷键

在大多数情况下，创建 Web 应用程序意味着拥有一个美观的 UI 和无障碍的数据。您希望用户能够流畅地体验，而不必通过点击多个页面来解决问题，有时这可能会变得相当麻烦。

拿任何 IDE 来说吧。尽管它们非常有用，而且在日常生活中非常方便，但想象一下如果它们没有简单的快捷方式，比如代码缩进。抱歉吓到你了，但事实上，像这样的细节可以使用户体验非常流畅，让用户愿意再次使用。

现在让我们创建一组简单的键盘快捷键，您可以为您的应用程序提供，以使最终用户的操作变得更加简单。要创建这个，您需要以下东西：

+   一个 Web 应用程序（我们之前创建了一个）

+   一组您希望能够使用键盘控制的功能

+   一个足够简单的实现，使得向其添加新功能非常简单

如果您还记得来自第一章的*自定义返回按钮*，我们将创建一个类似的应用程序。让我们快速再次组合示例应用程序。有关详细说明，您可以按照相同的示例（创建 Angular 应用程序）从第一章中进行。

# 创建 Angular 应用程序

1.  创建应用程序：

```js
 ng new keyboard-shortcuts
```

1.  在`src/pages`文件夹下创建多个状态（About、Dashboard、Home 和 Profile）并添加基本模板：

```js
import { Component } from '@angular/core';

@Component({
   selector: 'home',
   template: 'home page' })
export class HomeComponent {

}
```

1.  在`<component_name>.routing.ts`下创建该状态的路由：

```js
import { HomeComponent } from './home.component';

export const HomeRoutes = [
   { path: 'home', component: HomeComponent },
];

export const HomeComponents = [
   HomeComponent
];
```

1.  在`app.routing.ts`文件中添加新的`routes`和`Components`到应用程序的主路由文件`app.module.ts`旁边：

```js
import { Routes } from '@angular/router';
import {AboutComponents, AboutRoutes} from "./pages/about/about.routing";
import {DashboardComponents, DashboardRoutes} from "./pages/dashboard/dashboard.routing";
import {HomeComponents, HomeRoutes} from "./pages/home/home.routing";
import {ProfileComponents, ProfileRoutes} from "./pages/profile/profile.routing";

export const routes: Routes = [
   {
 path: '',
 redirectTo: '/home',
 pathMatch: 'full'
  },
   ...AboutRoutes,
   ...DashboardRoutes,
   ...HomeRoutes,
   ...ProfileRoutes ];

export const navigatableComponents = [
   ...AboutComponents,
   ...DashboardComponents,
   ...HomeComponents,
   ...ProfileComponents ];
```

1.  使用`RouterModule`注册应用程序的路由，并在`app.module.ts`文件中声明您的`navigatableComponents`：

```js
@NgModule({
    declarations: [
        AppComponent,
        ...navigatableComponents
    ],
    imports: [
        BrowserModule,
        FormsModule,
        RouterModule.forRoot(routes)
    ],
    providers: [],
    bootstrap: [AppComponent]
})
export class AppModule { }
```

1.  创建 HTML 模板以在`app.component.html`中加载四个路由：

```js
<nav>
    <button mat-button
            routerLink="/about"
  routerLinkActive="active">
        About
    </button>
    <button mat-button
            routerLink="/dashboard"
  routerLinkActive="active">
        Dashboard
    </button>
    <button mat-button
            routerLink="/home"
  routerLinkActive="active">
        Home
    </button>
    <button mat-button
            routerLink="/profile"
  routerLinkActive="active">
        Profile
    </button>
</nav>

<router-outlet></router-outlet>
```

一旦您完成了之前列出的所有步骤，请在终端中运行以下命令；Web 应用程序应该已经运行，并具有四个状态供您切换：

```js
ng serve
```

# 使用 keymap 创建状态

到目前为止，在状态（或路由）中我们声明的是路径和我们想要与其一起使用的组件。Angular 允许我们添加一个名为**data**的新属性到路由配置中。这允许我们添加关于任何路由的任何数据。在我们的情况下，这非常有效，因为我们希望能够根据用户按下的键来切换路由。

因此，让我们以前定义的一个示例路由为例：

```js
import { HomeComponent } from './home.component';

export const HomeRoutes = [
 { path: 'home', component: HomeComponent },
];

export const HomeComponents = [
 HomeComponent
];
```

现在我们将修改这个，并在路由配置中添加新的`data`属性：

```js
import { HomeComponent } from './home.component';

export const HomeRoutes = [
 { path: 'home', component: HomeComponent, data: { keymap: 'ctrl+h'} },
];

export const HomeComponents = [
 HomeComponent
];
```

您可以看到，我们添加了一个名为`keymap`的属性及其值`ctrl+h`；我们也将对所有其他定义的路由执行相同的操作。在一开始要确定的一个重要事项是锚键（在这种情况下是`ctrl`），它将与次要的标识键（在这里是`h`代表主页路由）一起使用。这真的有助于过滤用户在应用程序中可能进行的按键。

一旦我们将每个路由与键映射关联起来，我们就可以在应用程序加载时注册所有这些键映射，然后开始跟踪用户活动，以确定他们是否选择了我们预定义的键映射中的任何一个。

要注册键映射，在`app.component.ts`文件中，我们将首先定义我们将保存所有数据的`Map`，然后从路由中提取数据，然后将其添加到`Map`中：*：

```js
import {Component} from '@angular/core';
import {Router} from "@angular/router";

@Component({
    selector: 'app-root',
    templateUrl: './app.component.html',
    styleUrls: ['./app.component.scss',  './theme.scss']
})
export class AppComponent {

    // defined the keyMap
    keyMap = new Map();

    constructor(private router: Router) {
        // loop over the router configuration
        this.router.config.forEach((routerConf)=> {

            // extract the keymap
            const keyMap = routerConf.data ? routerConf.data.keymap :
            undefined;

            // if keymap exists for the route and is not a duplicate,
            add
            // to master list
            if (keyMap && !this.keyMap.has(keyMap)) {
                this.keyMap.set(keyMap, `/${routerConf.path}`);
            }
        })
    }

}
```

一旦数据被添加到`keyMap`中，我们将需要监听用户交互，并确定用户想要导航到哪里。为此，我们可以使用 Angular 提供的`@HostListener`装饰器，监听任何按键事件，然后根据应用程序的要求过滤项目，如下所示：

```js
import {Component, HostListener} from '@angular/core';
import {Router} from "@angular/router";

@Component({
    selector: 'app-root',
    templateUrl: './app.component.html',
    styleUrls: ['./app.component.scss',  './theme.scss']
})
export class AppComponent {

    // defined the keyMap
  keyMap = new Map();

    // add the HostListener
    @HostListener('document:keydown', ['$event'])
    onKeyDown(ev: KeyboardEvent) {

        // filter out all non CTRL key presses and 
        // when only CTRL is key press
        if (ev.ctrlKey && ev.keyCode !== 17) {

            // check if user selection is already registered
            if (this.keyMap.has(`ctrl+${ev.key}`)) {

                // extract the registered path
                const path = this.keyMap.get(`ctrl+${ev.key}`);

                // navigate
                this.router.navigateByUrl(path);
            }
        }
    }

    constructor(private router: Router) {
        // loop over the router configuration
  this.router.config.forEach((routerConf)=> {

            // extract the keymap
  const keyMap = routerConf.data ? routerConf.data.keymap :
            undefined;

            // if keymap exists for the route and is not a duplicate,
            add
 // to master list  if (keyMap && !this.keyMap.has(keyMap)) {
                this.keyMap.set(keyMap, `/${routerConf.path}`);
            }
        })
    }
}
```

现在我们可以轻松地定义和导航到路由，每当用户进行按键时。然而，在我们继续之前，我们需要换个角度来理解下一步。考虑一下，你是最终用户，而不是开发人员。你怎么知道绑定是什么？当你想要绑定页面上的路由以及按钮时，你该怎么办？你怎么知道自己是否按错了键？

所有这些都可以通过对我们目前的情况进行非常简单的 UX 审查来解决，以及我们需要的东西。很明显，我们需要向用户显示他们正在选择的内容，以便他们不会用错误的键组合不断地攻击我们的应用程序。

首先，为了告知用户他们可以选择什么，让我们修改导航，以便每个路由名称的第一个字符被突出显示。让我们还创建一个变量来保存用户选择的值，在 UI 上显示它，并在几毫秒后清除它。

我们可以修改我们的`app.component.scss`文件，如下所示：

```js
.active {
    color: red;
}

nav {
    button {
      &::first-letter {
        font-weight:bold;
        text-decoration: underline;
        font-size: 1.2em;
      }
    }
}

.bottom-right {
  position: fixed;
  bottom: 30px;
  right: 30px;
  background: rgba(0,0,0, 0.5);
  color: white;
  padding: 20px;
}
```

我们的模板在最后增加了一个内容，显示用户按下的键：

```js
<nav>
    <button mat-button
            routerLink="/about"
  routerLinkActive="active">
        About
    </button>
    <button mat-button
            routerLink="/dashboard"
  routerLinkActive="active">
        Dashboard
    </button>
    <button mat-button
            routerLink="/home"
  routerLinkActive="active">
        Home
    </button>
    <button mat-button
            routerLink="/profile"
  routerLinkActive="active">
        Profile
    </button>
</nav>

<router-outlet></router-outlet>

<section [class]="keypress? 'bottom-right': ''">
    {{keypress}}
</section>
```

我们的`app.component.ts`最终形式如下：

```js
import {Component, HostListener} from '@angular/core';
import {Router} from "@angular/router";

@Component({
    selector: 'app-root',
    templateUrl: './app.component.html',
    styleUrls: ['./app.component.scss',  './theme.scss']
})
export class AppComponent {

    // defined the keyMap
  keyMap = new Map();

    // defined the keypressed
  keypress: string = '';

    // clear timer if needed
 timer: number;

    // add the HostListener
  @HostListener('document:keydown', ['$event'])
    onKeyDown(ev: KeyboardEvent) {

        // filter out all non CTRL key presses and
 // when only CTRL is key press  if (ev.ctrlKey && ev.keyCode !== 17) {

            // display user selection
  this.highlightKeypress(`ctrl+${ev.key}`);

            // check if user selection is already registered
  if (this.keyMap.has(`ctrl+${ev.key}`)) {

                // extract the registered path
  const path = this.keyMap.get(`ctrl+${ev.key}`);

                // navigate
  this.router.navigateByUrl(path);
            }
        }
    }

    constructor(private router: Router) {
        // loop over the router configuration
  this.router.config.forEach((routerConf)=> {

            // extract the keymap
  const keyMap = routerConf.data ? routerConf.data.keymap :
            undefined;

            // if keymap exists for the route and is not a duplicate,
            add
 // to master list  if (keyMap && !this.keyMap.has(keyMap)) {
                this.keyMap.set(keyMap, `/${routerConf.path}`);
            }
        })
    }

    highlightKeypress(keypress: string) {
        // clear existing timer, if any
  if (this.timer) {
            clearTimeout(this.timer);
        }

        // set the user selection
  this.keypress = keypress;

        // reset user selection
  this.timer = setTimeout(()=> {
            this.keypress = '';
        }, 500);
    }

}
```

这样，用户总是知道他们的选择，使您的应用程序的整体可用性更高。

![](img/bd9059e2-55a9-491f-af36-38ca734c80db.png)

无论用户选择什么，只要按住*Ctrl*键，他们总是会在屏幕的右下角看到他们的选择。

# 网络应用程序的活动跟踪和分析

每当有人提到分析，特别是针对网络应用程序时，通常首先想到的是像 Google 分析或新的遗迹之类的东西。尽管它们在收集页面浏览和自定义事件等分析方面做得很好，但这些工具会将数据保留在它们那里，不允许您下载/导出原始数据。因此，有必要构建自己的自定义模块来跟踪用户操作和活动。

活动跟踪和分析是复杂的，随着应用程序规模的增长，很快就会失控。在这个用例中，我们将构建一个简单的网络应用程序，我们将跟踪用户正在进行的自定义操作，并为将应用程序数据与服务器同步打下一些基础。

在我们开始编码之前，让我们简要讨论一下我们的方法将是什么，以及我们将如何利用可用的 Angular 组件。在我们的应用程序中，我们将为用户构建一个基本表单，他们可以填写并提交。当用户与表单上可用的不同组件进行交互时，我们将开始跟踪用户活动，然后根据生成的事件提取一些自定义数据。这些自定义数据显然会根据正在构建的应用程序而改变。为简洁起见，我们将简单地跟踪事件的时间、*x*和*y*坐标以及自定义值（如果有）。

# 创建 Angular 应用程序

首先，让我们创建一个 Angular 应用程序，就像我们在前面的用例中所做的那样：

```js
ng new heatmap
```

这应该创建应用程序，并且应该准备就绪。只需进入您的项目文件夹并运行以下命令即可查看您的应用程序运行：

```js
ng serve
```

应用程序启动后，我们将包含 Angular 材料，这样我们就可以快速拥有一个漂亮的表单。要在您的 Angular 应用程序中安装材料，请运行以下命令：

```js
npm install --save @angular/material @angular/animations @angular/cdk
```

安装`material`后，在主`app.module.js`*中包含您选择的模块*，在这种情况下，将是`MatInputModule`和`ReactiveFormsModule`，因为我们将需要它们来创建表单。之后，您的`app.module.js`将如下所示：

```js
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import {FormsModule, ReactiveFormsModule} from '@angular/forms';
import {BrowserAnimationsModule} from '@angular/platform-browser/animations';
import { MatInputModule } from '@angular/material';

import { AppComponent } from './app.component';

@NgModule({
    declarations: [
        AppComponent
    ],
    imports: [
        BrowserModule,
        FormsModule,
        ReactiveFormsModule,
        BrowserAnimationsModule,
        MatInputModule
    ],
    providers: [

    ],
    bootstrap: [AppComponent]
})
export class AppModule { }
```

现在我们已经设置好了应用程序，我们可以设置我们的模板，这将是非常简单的，所以让我们将以下模板添加到我们的`app.component.html`文件中：

```js
<form>
    <mat-input-container class="full-width">
        <input matInput placeholder="Company Name">
    </mat-input-container>

    <table class="full-width" cellspacing="0">
        <tr>
            <td>
                <mat-input-container class="full-width">
                    <input matInput placeholder="First Name">
                </mat-input-container>
            </td>
            <td>
                <mat-input-container class="full-width">
                    <input matInput placeholder="Last Name">
                </mat-input-container>
            </td>
        </tr>
    </table>
    <p>
        <mat-input-container class="full-width">
            <textarea matInput placeholder="Address"></textarea>
        </mat-input-container>
        <mat-input-container class="full-width">
            <textarea matInput placeholder="Address 2"></textarea>
        </mat-input-container>
    </p>

    <table class="full-width" cellspacing="0">
        <tr>
            <td>
                <mat-input-container class="full-width">
                    <input matInput placeholder="City">
                </mat-input-container>
            </td>
            <td>
                <mat-input-container class="full-width">
                    <input matInput placeholder="State">
                </mat-input-container>
            </td>
            <td>
                <mat-input-container class="full-width">
                    <input matInput #postalCode maxlength="5" placeholder="Postal Code">
                    <mat-hint align="end">{{postalCode.value.length}} / 5</mat-
                    hint>
                </mat-input-container>
            </td>
        </tr>
    </table>
</form>
```

这是一个带有用户详细信息标准字段的简单表单；我们将稍微调整它的样式，使其居中显示在页面上，因此我们可以更新我们的`app.component.scss`文件以包含我们的样式：

```js
body {
  position: relative;
}

form {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.full-width {
  width: 100%;
}
```

这是 UI 上的最终结果：

![](img/aac34955-40d0-4a7a-b6b1-81464b9d5560.png)

现在我们已经准备好表单，我们需要一个活动跟踪器，这将非常轻量级，因为我们将经常调用它。

一个很好的做法是将跟踪逻辑移入 Web Worker；这样，您的跟踪器将不会占用唯一可用的线程，从而使您的应用程序免受任何额外负载的影响。

在我们实际开始创建 Web Worker 之前，我们需要一些东西来调用我们的工作人员；为此，我们将创建一个跟踪器服务。此外，为了使工作人员可以包含在 Angular 项目中，我们将将其添加到`.angular-cli.json`文件的`scripts`选项中，这将允许我们将其用作外部脚本调用`scripts.bundle.js`，该脚本是由`webpack`从文件`utils/tracker.js`生成的。

让我们在名为`services`的文件夹下创建一个名为`tracker`的文件夹，然后创建一个`tracker.service.ts`文件：

```js
import {Injectable} from '@angular/core';

@Injectable()
export class TrackerService {
    worker: any;

    constructor() {
        this.setupTracker();
    }

    setupTracker () {
        this.worker = new Worker('scripts.bundle.js');
    }

    addEvent(key: string, event: any, customValue ?: string) {
        this.worker.postMessage({
            key: key,
            user: 'user_id_here'
            event: {
                pageX: event.pageX,
                pageY: event.pageY
  },
            customValue : customValue
        });
    }
}
```

这里没有什么特别的；当我们触发服务并添加了`addEvent()`方法时，我们初始化了工作人员，该方法接受一些参数，如事件的名称（键）、事件数据（或其部分）和自定义值（如果有）。我们将其余的逻辑推迟到工作人员，以便我们的应用程序是无缝的。

但是，要触发服务的构造函数，我们需要将服务添加到主模块的提供者中。因此，我们的`app.module.ts`现在更新为以下内容：

```js
....
import {TrackerService} from "./service/tracker/tracker.service";

@NgModule({
    ....,
    providers: [
        TrackerService
    ],
    bootstrap: [AppComponent]
})
export class AppModule { }
```

好了，现在我们已经启动了应用程序并设置好了工作人员。但是，实际上是什么在调用`addEvent()`方法来跟踪这些自定义事件的？您可以执行以下一项或两项操作：

+   将`TrackerService`注入到您的组件/服务中，并使用正确的参数调用`addEvent()`方法

+   创建一个指令来捕获点击并使用`TrackerService`上的`addEvent()`方法同步数据

对于这个例子，我们将采用第二种方法，因为我们有一个表单，不想为每个元素添加点击处理程序。让我们创建一个`directives`文件夹和另一个名为`tracker`的文件夹，其中将包含我们的`tracker.directive.ts`：

```js
import {Directive, Input, HostListener} from '@angular/core';
import {TrackerService} from "../../service/tracker/tracker.service";

@Directive({
    selector: '[tracker]',
})
export class tracker {

    @Input('tracker') key: string;

    constructor(private trackerService: TrackerService) {}

    @HostListener('click', ['$event'])
    clicked(ev: MouseEvent) {
        this.trackerService.addEvent(this.key, ev);
    }
}
```

你可以看到指令非常简洁；它注入了`TrackerService`，然后在点击时触发`addEvent()`方法。

为了使用它，我们只需要将指令添加到之前创建的表单的输入元素中，就像这样：

```js
<input matInput placeholder="First Name" tracker="first-name">
```

现在，当用户与表单上的任何字段进行交互时，我们的 worker 会收到更改通知，现在我们的 worker 基本上要批量处理事件并将其保存在服务器上。

让我们快速回顾一下我们到目前为止所做的事情：

1.  我们设置了 worker 并通过我们的`TrackerService`的构造函数调用它，该构造函数在应用程序启动时实例化。

1.  我们创建了一个简单的指令，能够检测点击，提取事件信息，并将其传递给`TrackerService`，以便转发给我们的 worker。

![](img/0029140d-ddc4-4378-a13e-3bfb18339ba9.png)

上面的截图显示了到目前为止应用程序的目录结构。

我们的下一步将是更新我们的 worker，以便我们可以轻松处理传入的数据，并根据应用程序的逻辑将其发送到服务器。

让我们将`utils/tracker.js`下的 worker 分解为简单的步骤：

+   工人从`TrackerService`接收消息，然后将该消息转发以添加到事件主列表中：

```js
var sessionKeys = new Set();
var sessionData = new Map();
var startTime = Date.now();
var endTime;

self.addEventListener('message', function(e) {
 addEvent(e.data);
});
```

我们将通过维护两个列表来做一些不同的事情，一个用于保存正在保存的键，另一个将键映射到我们正在接收的数据集合。

+   `addEvent()`方法然后分解传入的数据并将其存储在正在收集的项目的主列表中，以便与数据库同步：

```js
function addEvent(data) {
   var key = data.key || '';
   var event = data.event || '';
   var customValue = data.customValue || '';
   var currentOccurrences;

   var newItem = {
      eventX: event.pageX,
      eventY: event.pageY,
      timestamp: Date.now(),
      customValue: customValue ? customValue : ''
  };

   if (sessionKeys.has(key)) {
      currentOccurrences = sessionData.get(key);
      currentOccurrences.push(newItem);

      sessionData.set(key, currentOccurrences);
   } else {
      currentOccurrences = [];
      currentOccurrences.push(newItem);

      sessionKeys.add(key);
      sessionData.set(key, currentOccurrences);
   }

   if (Math.random() > 0.7) {
      syncWithServer(data.user);
   }
}
```

我们将尝试检查用户是否已经与提供的键的元素进行了交互。如果是，我们将其追加到现有的事件集合中；否则，我们将创建一个新的事件集合。这个检查是我们利用集合及其极快的`has()`方法的地方，我们将在下一节中探讨。

除此之外，我们现在唯一需要的逻辑是根据预定的逻辑将这些数据与服务器同步。正如你所看到的，现在我们只是基于一个随机数来做这个，但是，当然，这对于生产应用程序是不推荐的。相反，你可以根据用户与应用程序的交互程度来学习，并相应地进行同步。对于一些应用程序跟踪服务来说，这太多了，所以他们采用更简单的方法，要么按照固定的时间间隔进行同步（几秒钟的顺序），要么根据有效负载大小进行同步。你可以根据应用程序的需求采取任何这些方法。

然而，一旦你掌握了这一点，一切都很简单：

```js
function syncWithServer(user) {
   endTime = Date.now();

   fakeSyncWithDB({
      startTime: startTime,
      endTime: endTime,
      user: user,
      data: Array.from(sessionData)
   }).then(function () {
      setupTracker();
   });
}

function fakeSyncWithDB(data) {
   //fake sync with DB
  return new Promise(function (resolve, reject) {
      console.dir(data);
      resolve();
   });
}

function setupTracker() {
   startTime = Date.now();
   sessionData.clear();
   sessionKeys.clear();
}
```

这里需要注意的一件奇怪的事情是，在将数据发送到服务器之前，我们将数据转换为数组的方式。也许我们可以直接传递整个`sessionData`？也许，但它是一个 Map，这意味着数据无法直接访问，你必须使用`.entires()`或`.values()`来获取一个迭代器对象，然后可以迭代地从地图中获取数据。虽然在处理数组时，需要在将数据发送到服务器之前转换数据似乎有点倒退，但考虑到 Map 为我们的应用程序提供的其他好处，这样做是非常值得的。

现在让我们来看看在我们的`tracker.js`文件中如何将所有内容整合在一起：

```js
var sessionKeys = new Set();
var sessionData = new Map();
var startTime = Date.now();
var endTime;

self.addEventListener('message', function(e) {
   addEvent(e.data);
});

function addEvent(data) {
   var key = data.key || '';
   var event = data.event || '';
   var customValue = data.customValue || '';
   var currentOccurrences;

   var newItem = {
      eventX: event.pageX,
      eventY: event.pageY,
      timestamp: Date.now(),
      customValue: customValue ? customValue : ''
  };

   if (sessionKeys.has(key)) {
      currentOccurrences = sessionData.get(key);
      currentOccurrences.push(newItem);

      sessionData.set(key, currentOccurrences);
   } else {
      currentOccurrences = [];

      currentOccurrences.push(newItem);
      sessionKeys.add(key);

      sessionData.set(key, currentOccurrences);
   }

   if (Math.random() > 0.7) {
      syncWithServer(data.user);
   }
}

function syncWithServer(user) {
   endTime = Date.now();

   fakeSyncWithDB({
      startTime: startTime,
      endTime: endTime,
      user: user,
      data: Array.from(sessionData)
   }).then(function () {
      setupTracker();
   });
}

function fakeSyncWithDB(data) {
   //fake sync with DB
  return new Promise(function (resolve, reject) {
      resolve();
   });
}

function setupTracker() {
   startTime = Date.now();
   sessionData.clear();
   sessionKeys.clear();
}
```

正如你在上面的代码中所注意到的，集合和映射悄然而有效地改变了我们设计应用程序的方式。我们不再只有一个简单的数组和一个对象，而是实际上可以使用一些具体的数据结构和一组固定的 API，这使我们能够简化我们的应用程序逻辑。

# 性能比较

在本节中，我们将比较集合和映射与它们的对应物：数组和对象的性能。正如前几章所述，进行比较的主要目标不是要知道数据结构优于其本机对应物，而是要了解它们的局限性，并确保在尝试使用它们时做出明智的决定。

重要的是要以一颗谷物的方式对待基准测试。基准测试工具通常使用诸如 V8 之类的引擎，这些引擎被构建和优化以以一种与其他一些基于 Web 的引擎非常不同的方式运行。这可能导致结果在应用程序运行的环境中有些偏差。

我们需要做一些初始设置来运行我们的性能基准测试。要创建一个 Node.js 项目，请转到终端并运行以下命令：

```js
mkdir performance-sets-maps
```

这将设置一个空目录；现在，进入目录并运行`npm`初始化命令：

```js
cd performance-sets-maps
npm init
```

这一步将询问您一系列问题，所有问题都可以填写或留空，视您的意愿而定。

项目设置好后，接下来我们将需要基准测试工具，我们可以使用`npm`安装它：

```js
npm install benchmark --save
```

现在，我们准备开始运行一些基准测试套件。

# 集合和数组

由于`benchmark`工具，创建和运行`suite`非常容易。我们只需要设置我们的`sets-arr.js`文件，然后就可以开始了：

```js
var Benchmark = require("benchmark");
var suite = new Benchmark.Suite();

var set = new Set();
var arr = [];

for(var i=0; i < 1000; i++) {
   set.add(i);
   arr.push(i);
}

suite
  .add("array #indexOf", function(){
      arr.indexOf(100) > -1;
   })
   .add("set #has", function(){
      set.has(100);
   })   .add("array #splice", function(){
      arr.splice(99, 1);
   })
   .add("set #delete", function(){
      set.delete(99);
   })
   .add("array #length", function(){
      arr.length;
   })
   .add("set #size", function(){
      set.size;
   })
   .on("cycle", function(e) {
      console.log("" + e.target);
   })
   .run();
```

您可以看到设置非常简单明了。一旦创建了新的`suite`，我们为初始加载设置一些数据，然后可以将我们的测试添加到`suite`中：

```js
var set = new Set();
var arr = [];

for(var i=0; i < 1000; i++) {
 set.add(i);
 arr.push(i);
}
```

要执行这个`suite`，您可以从终端运行以下命令：

```js
node sets-arr.js
```

`suite`的结果如下：

![](img/7e351e82-f7b2-45cc-87a9-871f22b5b482.png)

请注意，在这个设置中，集合比数组稍快。当然，我们在测试中使用的数据也会导致结果的变化；您可以通过在数组和集合中存储的数据类型之间切换来尝试一下。

# 映射和对象

我们将在一个名为`maps-obj.js`的文件中为映射和对象设置类似的设置，这将给我们类似以下的东西：

```js
var Benchmark = require("benchmark");
var suite = new Benchmark.Suite();

var map = new Map();
var obj = {};

for(var i=0; i < 100; i++) {
   map.set(i, i);
   obj[i] = i;
}

suite
  .add("Object #get", function(){
      obj[19];
   })
   .add("Map #get", function(){
      map.get(19);
   })
   //
  .add("Object #delete", function(){
      delete obj[99];
   })
   .add("Map #delete", function(){
      map.delete(99);
   })
   .add("Object #length", function(){
      Object.keys(obj).length;
   })
   .add("Map #size", function(){
      map.size;
   })
   .on("cycle", function(e) {
      console.log("" + e.target);
   })
   .run();
```

现在，要运行这个`suite`，请在终端上运行以下命令：

```js
node maps-obj.js
```

这将给我们以下结果：

![](img/937b1b92-e459-4b9f-b9af-75dd595ea39c.png)

您可以看到`Object`在这里远远优于映射，并且显然是两者中更好的，但它不提供映射能够提供的语法糖和一些功能。

# 总结

在本章中，我们深入研究了集合和映射，它们的较弱对应物以及它们的 API。然后，我们在一些真实世界的例子中使用了集合和映射，比如使用集合进行导航的应用程序的键盘快捷键和使用集合和映射进行应用程序分析跟踪器。然后，我们通过对象和数组之间的性能比较结束了本章。

在下一章中，我们将探讨树以及如何利用它们使我们的 Web 应用程序更快，代码复杂性更低。
