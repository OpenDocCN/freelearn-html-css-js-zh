# 第十五章。AngularJS – 谷歌的热门框架

Angular 是一个应用程序框架，它帮助您创建 Web 应用程序。它基于 HTML 和 JavaScript，使动态应用程序的创建更加容易。Angular 扩展了 JavaScript，同时也是一个超集。您可以使用纯 JavaScript 和 Angular 来构建您的应用程序。

这把双刃剑。在积极的一面，构建动态用户界面并保持代码可维护和可测试要容易得多。在另一方面，你必须学习 Angular 应用程序构建的整体概念，即 Angular 的哪个部分在哪里。这与您可能构建的任何其他 JavaScript 应用程序都大不相同。

如果你以前只使用过 **jQuery**、**BackBone** 或纯 JavaScript 来构建用户界面，那么 Angular 的许多内容都会显得新颖且不同。

我们将以分层的方式尝试涵盖 Angular。我们将从顶部开始，包括包含其他部分和对象的容器，以及可以在许多对象之间使用的函数，然后讨论测试，因为它是 Angular 构建的核心部分。

让我们不要浪费时间，直接进入正题。

所有示例都假设您已经加载了 Angular，因此 `angular` 对象是可用的。

### 注意

如果您不确定如何加载 Angular，请访问官方 Angular 页面 [`angularjs.org/`](https://angularjs.org/)。

# 模块（ngApp）

这是所有其他应用程序部分的顶层容器。这里列出的功能比这些更多，但它们将在相关部分中描述。例如，您可以使用 `module.controller` 创建一个控制器，但此函数将在控制器部分中。

## 模块

模块是 Angular 应用程序的第一个基本构建块：

```js
angular.module(moduleName, [dependencies], [configFunction])
angular.module(moduleName)
```

### 参数

+   `moduleName`：这是一个字符串，将标识模块。它应该是唯一的。

+   `dependencies(可选)`: 这是一个模块名称数组，该模块依赖于这些模块。

+   `configFunction(可选)`: 这是一个将配置模块的函数。请参阅 *config* 部分。

### 返回值

这是一个 Angular 模块对象。

### 描述

此函数将创建一个新模块或检索一个模块。这取决于我们是否传递了一个依赖项数组。如果没有省略依赖项，则检索一个模块。相反，如果我们包含一个要加载的模块数组（该数组也可以为空），则将创建一个模块。

当你有一组协同工作的对象时，将它们放入一个模块是一个好主意。不要担心创建太多的模块。一个模块应该只有一个且只有一个明确的函数。然后，所有它需要完成该功能的对象都将与其打包在一起。

这里有一些创建模块的示例：

+   这将创建一个新模块：`var firstModule = angular.module('firstModule', []);`

+   这将检索名为 `firstModule` 的模块：`var firstModule = angular.module('firstModule');`

+   这创建了一个依赖于 `firstModule` 的模块：`var secondModule = angular.module('secondModule', ['firstModule']);`

## config

这允许您配置提供者：

```js
module.config(configFunction)
```

### 参数

+   `configFunction` 函数是将会被执行的函数。

### 返回值

这是一个 Angular 模块对象。

### 描述

这是用于配置提供者的函数。我们将在本章后面介绍提供者，但本节中也有一些示例。这允许在模块实际执行之前创建提供者。

当调用此函数时，提供者将被注入到该函数中。例如，您可以使用 `$provide` 或 `$routeProvider`，它们将可用。如果您想使用自定义提供者，它必须在 `config` 之前创建。自定义对象需要在其名称后附加 `provider` 以在 `config` 中可用。

这里有一些使用提供者的示例：

+   这个示例使用了 `$provide`。提供者将在之后作为 `firstProvider` 可用：

    ```js
    firstModule.config(function ($provide) {
      $provide.provider('firstProvider', function () {
        this.$get = function () {
          return function () { alert('First!'); }
        }
      });
    });
    ```

+   这个示例首先创建了一个提供者。它在 `config` 中作为 `firstProvider` 注入，并在之后使用：

    ```js
    firstModule.provider('first', function () {
      this.$get = function () {
        return function () { alert('First!'); }
      }
    });
    firstModule.config(function (firstProvider) {
      console.log(firstProvider);
    });
    ```

## run

这个函数在配置之后执行：

```js
module.run(runFunction)
```

### 参数

+   `runFunction` 函数是将会被执行的函数。

### 返回值

这是一个 Angular 模块对象。

### 描述

`run()` 函数在 `config` 之后执行。此时，所有模块都将被加载。这是在 `config` 之后初始化模块后第一个被调用的函数。您在这个函数中做的事情将取决于模块。

这里有一个设置作用域变量的示例。该变量将在模块的后续函数中可用：

```js
firstModule.run(function ($rootScope) {
  $rootScope.test = 'test';
});
```

# 路由（ngRoute）

`ngRoute` 模块是一个允许您在应用程序中配置路由的模块。路由正在监听位置的变化，然后自动对新控制器和模板做出响应。它使用 `ngView`、`$routeProvider` 和 `$route`。

### 注意

您的模块需要依赖于 `ngRoute` 来使用这些指令和服务。我们还需要在我们的 HTML 中包含 Angular 路由 JavaScript。

## ngView

这与 `$route` 服务一起作为内容的位置：

```js
<ng-view [onload=''] [autoscroll='']/>
<element ng-view [onload=''] [autoscroll='']/>
```

### 参数

+   `onload(Angular 表达式)`: 在加载时进行评估。

+   `autoscroll(Angular 表达式)`: 是否使用 `$anchorScroll` 与此 `ngView`。默认情况下，它是禁用的。否则，它将评估表达式是否为真。

### 返回值

这是一个指令。

### 描述

在路由时，您需要标记应用程序中可以加载动态内容的部分。这就是 `ngView` 所做的。当匹配到路由时，它将在 `ngView` 的位置加载新内容。本节末尾将提供一个 `ngRoute` 的示例。

## $routeProvider

这是一个允许您配置路由的提供者：

```js
$routeProvider.when(path, route)
$routeProvider.otherwise(route)
```

### 参数

+   `path(string)`: 这是位置将与之匹配的内容。它可以使用冒号（`:group`）包含命名组，可以包含星号以匹配多个部分（`:group*`），也可以是可选的（`:group?`）。

+   `route(object)`: 这个对象告诉 Angular 当路由匹配时应该做什么。该对象可以具有以下属性（非详尽列表）：

    +   `controller`

    +   `template`

    +   `templateUrl`

    +   `resolve`

### 返回值

这将返回`$routeProvider`，因此它是可链式的。

### 描述

这是可以用来配置所有路由的提供者。它可以注入到`module.config`中。

## `$route`

这是提供匹配路由定义的实际服务。这是一个可以直接注入到控制器中的对象。

### 属性

+   `current(object)`: 这个对象包含基于匹配的路由的当前变量。这将包括`$route`、`loadedTemplateUrl`、`locals`、`params`和`scope`。

+   `routes(object)`: 这包含所有配置的路由。

### 事件

可以监听这些事件的`$route`对象：

+   `$routeChangeStart`: 在实际路由更改之前触发此事件

+   `$routeChangeSuccess`: 在路由解析后触发此事件

+   `$routeChangeError`: 如果任何路由承诺被拒绝，则触发此事件

### 描述

这是一个可以注入到每个控制器中的服务。控制器可以查看属性以获取有关匹配路由的信息。路由匹配将在你的干预下发生，然后你可以在路由加载时更新控制器的作用域。

## `$routeParams`

这是一个将返回已加载路由参数的服务。这只会在你解析了路由后提供参数，因此你可能需要使用`$route.current.params`。这可以注入到控制器中。

这里是一个使用`ngRoute`中所有指令和服务的示例。首先，我们有一个 HTML body 标签。这还将包括一个名为`main.html`的内联模板。这个模板将输出`$route`和`$routeParams`对象：

```js
<body ng-app="firstModule">
  <div ng-view></div>
  <a href="#/main/1">Main</a>

  <script type="text/ng-template" id="main.html">
    <pre>$routeParams = {{$routeParams}}</pre>
    <pre>$route = {{$route}}</pre>
  </script>

</body>
```

这里是执行此操作的脚本。首先是创建模块，然后是配置`$routeProvider`，最后是为提供者定义一个控制器：

```js
var firstModule = angular.module('firstModule', ['ngRoute']);

firstModule.config(function ($routeProvider) {
  $routeProvider.when('/main/:id', {
    controller: 'MainController',
    templateUrl: 'main.html'
  });
});

firstModule.controller('MainController', function ($scope, $route, $routeParams) {
  $scope.$route = $route;
  $scope.$routeParams = $routeParams;
});
```

# 依赖注入

Angular 在所有地方都使用了依赖注入。依赖注入是指一个函数不初始化它需要的依赖项。相反，它们作为参数注入到函数中。例如，当模块需要一个路由提供者时，它会请求一个。模块不关心路由提供者是如何或何时被创建的；它只需要一个引用。

你实际上在 Angular 的每一件事中都使用了注入器。Angular 只是为你做了这件事。我们将查看`$injector`并了解它是如何工作的，但很可能会不需要使用这些函数。

## Angular 中的依赖注入

我们将快速介绍在 Angular 中将对象注入函数的常见方法。以控制器为例，我们将介绍两种最常见的方法。这两种方法都是在 Angular 的背后使用注入：

+   **在函数中定义变量**：你只需传递需要注入的对象的名称。以下是一个使用 `$scope` 和服务的示例：

    ```js
    firstModule.controller('DIController', function ($scope, customService) {
      //$scope and customService are available here
      console.log($scope);
      console.log(customService);
    });
    ```

+   **使用数组来列出依赖项**：你可以使用数组得到相同的结果。数组的元素将是作为字符串需要的对象。最后，你只需要在数组中有一个函数作为最后一个元素。如果需要，函数甚至可以重命名变量。以下是以数组格式相同的示例：

    ```js
    firstModule.controller('DIController', ['$scope', 'customService', function (newScope, custom) {
      // newScope and custom are available here
      console.log(newScope);
      console.log(custom);
    }]);
    ```

### 注意

只有数组格式是压缩安全的。一旦源代码被压缩，第一个函数将无法工作。

## 注入器

使用这个方法来获取一个你可以调用的 Angular 注入器：

```js
angular.injector(modules)
```

### 参数

+   `modules(array)` 数组是你想要加载的模块列表。你必须包含 `ng`。

### 返回值

这将返回一个注入器对象。参见 `$injector` 部分。

### 描述

你可以创建这个对象来使用 Angular 的注入器对象。列出你想要的引用，然后调用你的函数。

以下是一个获取 `$http` 服务引用的示例：

```js
var injector = angular.injector(['ng']);
var injectTest = function ($http) { console.log($http); };
injector.invoke(injectTest);
```

## `$injector`

在 Angular 中，你几乎很少需要创建一个注入器。它将为你提供。你可以通过注入或从当前模块中检索来获取注入器的引用。以下两种方法的示例：

+   你可以从模块内的任何元素中获取引用：

    ```js
    var injector = angular.element(document.body).injector();
    ```

+   你可以将它注入到一个函数中。以下是一个使用 `config` 的示例：

    ```js
    firstModule.config(function ($routeProvider, $injector) {
      console.log($injector);
    });
    ```

### 方法

+   `annotation`: 这将返回一个将要注入的对象数组

+   `get(name)`: 这将返回具有其名称的服务

+   `invoke(function)`: 这将执行函数并注入依赖项

+   `has(name)`: 这允许你确定一个服务是否存在

### 描述

Angular 会自动为你做这件事，但了解正在发生的事情总是好的。这尤其适用于像 `$injector` 一样工作的东西。每次函数执行时，Angular 都会从注入器中调用函数，如果它们已被创建并注册，则会发送正确的参数。

# 控制器

控制器是任何 Angular 应用程序的核心单元之一。控制器用于创建一个需要自己作用域的模块的小部分。每个模块可以有多个控制器。控制器应该是小的，专注于一项任务。

每个控制器实际上应该只关注数据和任何修改该数据的事件。这意味着控制器不应该修改 DOM、更改输出或输入，或与其他控制器共享状态。每个都应该使用 Angular 的解决方案，即指令、过滤器和服务，分别。

控制器是从模块引用创建的，因此它们与模块相关联。以下是一个创建简单控制器的示例：

```js
firstModule.controller('SimpleController', ['$scope', function ($scope) {
  $scope.hey = "HEY!";
  console.log($scope);
}]);
```

然后可以使用`ngController`将此模块附加到 DOM 元素：

```js
<div ng-controller="SimpleController">
  {{ hey }}
</div>
```

## ngController

这是 Angular 映射到模型-视图-控制器模式的核心部分。

```js
<element ng-controller />
```

### 参数

+   `ng-controller(expression)`属性是一个字符串，它将告诉 Angular 哪个控制器与这个元素相关联。

### 描述

控制器需要一个文档的部分来附加。此指令将绑定控制器及其作用域到该元素。

如果控制器已在路由中定义，则不应将此指令添加到页面中。路由器将负责绑定到正确的元素。

这里是使用`ngcontroller`为控制器创建别名的示例：

```js
<div ng-controller="SimpleController as simple">
  {{ simple.hey }}
</div>
```

## `$scope`

这是控制器最重要的部分。这是你应该放置所有跟踪此控制器的位置的地点。这包括任何修改作用域的函数。

`$scope`控制器通过声明对其的依赖关系（参见依赖注入）注入到控制器中。有一些内置的函数和属性（参见作用域），但由于它只是一个 JavaScript 对象，你可以添加自己的函数和属性。这些将直接映射到控制器中的模板。

这里是一个控制器示例，它定义了一个属性`hey`和一个函数`changeHey`。重要的是要注意，在这个函数中根本没有任何对 DOM 引用的引用：

```js
firstModule.controller('SimpleController', ['$scope', function ($scope) {
  $scope.hey = "HEY!";
  $scope.changeHey = function () {
    $scope.hey = $scope.hey === 'HEY!' ? 'OK!' : 'HEY!';
  };
}]);
```

这里是包含所有数据绑定的 HTML 文档的模板：

```js
<div ng-controller="SimpleController">
  {{ hey }}
  <input type="text" ng-model="hey"/>
  <button ng-click="changeHey()">{{hey}}</button>
</div>
```

Angular 将知道在这个元素中`hey`是什么，因为它仅限于`SimpleController`的作用域。

## 数据绑定和模板

控制器使用 HTML 作为其模板语言。你只需在双括号`({{ }})`中包围它，就可以绑定控制器作用域中的值。这就是全部内容！

对于某些元素，如`input`、`select`和`textarea`，你不能仅仅将`scope`对象的值添加到它们来绑定。你必须使用`ngModel`指令。

`$scope`部分有一个绑定作用域值到模板的绝佳示例。

## 事件绑定

Angular 使用指令来绑定事件。在`$scope`部分，示例使用`ngClick`来监听按钮上的点击事件。

请参阅*指令*、*事件绑定*部分，以获取最常用的事件指令列表。

# 作用域

注入到控制器中的`$scope`对象有一些功能。除了这些，我们还将查看作用域的层次结构和消化周期。

## 消化周期

这是理解作用域的一个重要概念。从高层次来看，消化周期是一个检查是否有作用域变量被更改的周期。如果有，它将执行一个函数。例如，使用`{{variable}}`将作用域变量绑定到模板上。现在消化周期将监视这个变量，并且每次它更改时，它都会更新模板。如果变量在其他地方被绑定，它也会被更新。这就是 Angular“神奇地”使值自动更新的方式。

有几点需要注意，作用域中并不是所有内容都被监视。最简单的方法是绑定它。你也可以手动监视值。此外，记住，当你监视许多变量时，可能会出现性能问题。消化周期会遍历所有监视器。

## $digest

这是你可以启动消化周期的方法：

```js
scope.$digest()
```

### 描述

这将手动启动作用域的消化周期。在大多数情况下，这不需要调用。例如，当通过 Angular 处理点击事件更新值时，不需要调用 `$digest`。Angular 会为你启动消化周期。一个很好的经验法则是，如果你在控制器中从 Angular 事件或指令更改或更新值，你就不需要调用 `$digest`。

你可能需要调用它的情况之一是处理自定义指令中的异步调用。

## $watch

这允许你在消化周期中监视值或计算值：

```js
scope.$watch(watch, listener)
```

### 参数

+   `watch(string or function)`: 这是你要监视的内容。字符串将被评估为表达式。函数将可以通过第一个参数访问作用域。使用函数允许你监视作用域中的值，以及变量的计算值。

+   `listener(function)`: 这个函数将具有 `function(newVal, oldVal, scope)` 签名。这是当值改变时将执行的函数。

### 返回值

这是一个用于注销监视器的函数。

### 描述

这是手动将监视器添加到消化周期的方法。主要有两个原因要做这件事。第一个是当值改变时，你需要运行一个自定义函数。另一个原因是你需要监视 `$scope` 中值的组合或计算。

这里有一个例子，它监视一个字符串和一个函数。这个例子的基础来自 *控制器*，*$scope* 部分。第一个监视器监听作用域中的 `hey` 变量。另一个监视器查看 `hey` 的值是否有三个或更多字符。它只会在这个阈值被越过时触发，并且不会再次触发，直到它以相反方向越过：

```js
$scope.$watch('hey', function (newVal, oldVal, scope) {
  alert('Hey was changed from ' + oldVal + ' to ' + newVal);
});

$scope.$watch(function (scope) {
  return scope.hey.length >= 3;
}, function (newVal, oldVal, scope) {
  console.log('Hey listener');
});
```

## $apply

这是手动启动消化周期的方法：

```js
scope.$apply(func)
```

### 参数

+   `func(function or string)` 属性是一个字符串，它将被评估为表达式。如果它是一个函数，该函数将被执行。

### 返回值

这是函数或表达式的返回值。

### 描述

这将手动启动消化周期。与 `$digest` 类似，在大多数情况下不应调用，除非是特定情况。最常见的情况是当作用域值在 Angular 之外被更改时，例如，一个异步 AJAX 调用，它引用了一个作用域值。更新发生在消化循环之外，Angular 并不知道，因为它没有使用任何 Angular 方法。

这导致了一个最佳实践，即始终使用 Angular 的服务和函数。如果你这样做，你就不需要运行`$apply`。

另一个最佳实践是在`$apply`内部运行你的函数。Angular 会捕获抛出的任何错误，你可以用 Angular 的方式处理它们。

这里有一个示例，在 2 秒后更新`setTimeout`内部的范围。由于消化循环已经完成，除非调用`$apply`，否则它将看不到这个变化：

```js
setTimeout(function () {
  $scope.$apply(function () { $scope.hey = 'Updated from setTimeout'; });
}, 2000);
```

## 层次结构

Angular 应用只有一个根作用域，但有许多子作用域。当引用一个变量时，它会检查当前作用域，然后检查父作用域。这会一直发生到根作用域。这与 JavaScript 对象使用原型属性的方式非常相似。你可以在第八章*JavaScript 面向对象编程*中了解更多关于这个内容。

# 服务

服务在 Angular 应用中的作用是明确的，但在服务的创建方面可能会有困惑。这是因为 Angular 中有三种非常相似的方式来创建服务。我们将逐一查看这些方法以及为什么应该使用它们。

Angular 中的服务是一个对象，它可以是一个数据权威（意味着它是某些数据的唯一来源）。一个很好的例子是路由提供者，因为它是唯一提供路由信息的对象。它是一个单例，所有模块都使用它，一种在控制器之间保持数据同步的方法，或者所有这些！你可能需要的服务的绝佳例子是`$http`。它为你执行 AJAX 请求。你可以构建一个服务，从你的 API 返回数据，你不必担心在每个控制器中创建 AJAX 调用。

另一个例子是`$route`和`$routeParams`服务。当你有一个对`$route`服务的引用时，你总能找到已匹配的路由。`$routeParams`服务会告诉你参数。

应该在任何需要多次编写相同代码的情况下使用服务。你可以将其提取出来并放入一个服务中。除此之外，对于将被多个控制器使用的任何数据，都应该创建服务。控制器可以随后向服务请求数据。这将确保这些数据在多个控制器之间保持一致。

列出的所有函数都可以从模块或`$provide`可注入对象中调用。

工厂、服务和提供者都将依赖于以下 HTML 模板：

```js
<div ng-controller="SimpleController">
  {{ service }}
</div>
```

## 工厂

这是创建工厂的方法：

```js
module.factory(name, getFunction)
```

### 参数

+   `name(string)`: 这是服务将被知晓的名称。这可以用于依赖注入。

+   `getFunction(function or array)`: 这是将返回服务的函数。这也可以是一个列出依赖项的数组。

### 返回值

这是一个 Angular 模块对象。

### 描述

服务工厂在需要单例时很棒。除了在`factory()`函数中完成的配置之外，没有其他配置。每次注入时对象都是相同的。

这里有一个简单的例子，它将返回服务的名称。该例子还创建了对`$http`的依赖：

```js
firstModule.factory('firstFactory', ['$http', function ($http) {
  var serviceName = 'firstFactory';
  return {
    getName: function(){return serviceName;}
  };
}]);
```

## 服务

服务可以用`new`关键字初始化：

```js
module.service(name, constructor)
```

### 参数

+   `name(string)`: 这是服务的名称

+   `constructor(function or array)`: 这将是作为构造函数调用的函数

### 返回值

这是一个 Angular 模块对象。

### 描述

服务和工厂之间的关键区别在于它可以使用`new`初始化。以下是一个依赖于`$http`的例子：

```js
firstModule.service('firstService', ['$http', function ($http) {
  return function (name) {
    var serviceName = name;

    this.getName = function () {
      return serviceName;
    };
  };
}]);
```

这里是初始化服务的控制器：

```js
firstModule.controller('SimpleController', ['$scope', 'firstService', function ($scope, firstService) {
  var first = new firstService('First Service Name');
  var second = new firstService('Second Service Name');
  $scope.service = second.getName();
}]);
```

## 提供者

提供对服务设置最大控制的函数：

```js
module.provider(name, provider)
```

### 参数

+   `name(string)`: 这是提供者的名称

+   `provider(function or array)`

### 返回值

这是一个 Angular 模块对象。

### 描述

这是构建服务的最终和最复杂的方式。它必须返回一个包含`$get`方法的对象。当你必须为此对象配置时，你应该使用这个。这是唯一可以在应用程序启动配置阶段注入的提供者。单词“提供者”将附加到这个名称上。例如，如果你将你的提供者命名为`my`，它将使用`myProvider`注入。以下是一个在配置期间注入其名称的示例提供者。首先是提供者定义：

```js
firstModule.provider('first', function () {
  var serviceName;

  return {
    configSet: function (name) {
      serviceName = name;
    },
    $get: function () {
      return {
        getName: function () { return serviceName; }
      };
    }
  }
});
```

接下来是配置。注意它是如何使用`firstProvider`注入的：

```js
firstModule.config(['firstProvider', function (firstProvider) {
  firstProvider.configSet('firstProvider123');
}]);
```

最后，这里是控制器：

```js
firstModule.controller('SimpleController', ['$scope', 'first', function ($scope, firstProvider) { 
  $scope.test = firstProvider.getName();
}]);
```

## 值

这在模块中设置一个值：

```js
module.value(name, value)
```

### 参数

+   `name(string)`: 这是值的名称

+   `value(object)`: 这可以是任何东西

### 返回值

这是一个 Angular 模块对象。

### 描述

值是设置后可以稍后注入的东西。值只能注入到控制器或服务中，不能在配置期间注入。

这里是一个简单的值例子：

```js
firstModule.value('appTitle', 'First App');
```

## 常量

这在模块中创建了一个常量变量：

```js
module.constant(name, value)
```

### 参数

+   `name(string)`: 这是值的名称

+   `value(object)`: 这可以是任何东西

### 返回值

这是一个 Angular 模块对象。

### 描述

这与值非常相似，除了它可以在`config`函数中使用，并且值不能被装饰。

这里是一个简单的常量例子：

```js
firstModule.constant('appTitle', 'First App');
```

## $http

这是用于进行 HTTP 调用的服务：

```js
$http(config)
```

### 参数

+   `config(object)`: 此对象具有以下属性：

    +   `method(string)`: 这是 HTTP 方法。

    +   `url(string)`: 这是 URL。

    +   `params(string or object)`: 这是请求的参数。一个对象将被映射为键值对。

    +   `data(string or object)`: 这是请求中要发送的数据。

    +   `headers(object)`: 这设置了请求的头。

    +   `xsrfHeadername(string)`: 这是 XSRF 的头名称。

    +   `xsrfCookieName(string)`: 这是 XSRF 的 cookie 名称。

    +   `cache(Boolean)`：这用于决定是否缓存数据。

    +   `responseType(string)`：这用于决定请求的类型。

### 返回值

将返回一个承诺。

### 描述

如果你熟悉 jQuery 的 AJAX 函数，那么你对这个应该很熟悉。这是 Angular 实现任何`XMLHttpRequests`的方式。对于大多数事情，你将使用一些方便的方法，这些方法使得使用`$http`更加容易。

## 方便方法

你可以使用所有的 HTTP 方法作为函数：`GET`、`POST`、`HEAD`、`PUT`、`DELETE`和`PATCH`。我们只将详细查看`GET`、`POST`和`JSONP`。

### 注意

`GET`、`POST`和`JSONP`将涵盖大多数人对于`$http`的需求，如果不是全部的话。如果这还不够，请查看`$http`的文档：[`docs.angularjs.org/api/ng/service/$http`](https://docs.angularjs.org/api/ng/service/$http)。

每个函数都将 URL 作为第一个参数，以及一个`config`对象，这个`config`对象与`$http()`获取的是同一个。

### GET

这是一个`GET`请求：

```js
$http.get(url, config)
```

#### 描述

这执行了一个简单的`GET`请求。以下是一个使用`$http.get`的工厂示例：

这里是工厂：

```js
firstModule.factory('httpService', ['$http', function ($http) {
  return {
    test: function () { return $http.get('test', {params: 'test'}); }
  }
}]);
```

这返回一个承诺，控制器可以使用它：

```js
firstModule.controller('SimpleController', ['$scope', 'httpService', function ($scope, httpService) {
  httpService.test().success(function (data, status) {
    console.log(data);
  })
  .error(function (data, status) { console.log('An error occured');});
}]);
```

### POST

这是一个`POST`请求：

```js
$http.post(url, config)
```

#### 描述

这将发起一个`POST`请求。以下是一个创建`POST`请求的工厂和示例，使用 localhost。请记住，你必须运行一个响应`POST`请求的服务器，这样它才能工作：

```js
firstModule.factory('httpService', ['$http', function ($http) {
  return {
    test: function () { return $http.post('http://localhost/post', { data: 'HEY' }); }
  }
}]);
```

#### jsonp

当你必须发起跨源请求时，你应该使用`JSONP`。这将允许你使用数据，而不是根据安全原因阻止请求。

## 显著的服务

我将列出一些可以注入的有用服务，以及每个服务的简要说明：

+   `$anchorScroll`：此参数允许你滚动到 hash 中的元素

+   `$animate`：此参数允许 DOM 操作

+   `$cacheFactory`：这允许缓存

+   `$http`：见`$http`

+   `$interval`：这是使用 Angular 的方式调用`setInterval`

+   `$location`：这获取`window.location`的信息

+   `$rootScope`：这返回应用程序的根作用域

+   `$route`：见`路由`

+   `$q`：这允许在我们的项目中添加承诺

+   `$timeout`：这是使用 Angular 的方式调用`setTimout`

这不是一个完整的列表，因为 Angular 有很多服务，并且可以创建和包含更多。这些是在你的应用程序中最可能使用的服务。

# 承诺

在 JavaScript 中，许多操作都是异步的。一个很好的例子是 AJAX 请求。当请求发送时，你不知道何时或甚至是否会有请求返回。这就是承诺出现的地方。

承诺是一个对象，它承诺在异步事件发生后运行一个函数。在我们的例子中，事件将是请求返回。这可能是在几毫秒或更长的时间内，这取决于超时设置。

除了跟踪成功的事件外，promises 还可以被拒绝。这允许等待响应的对象执行不同的操作。

### 注意

访问 [`promisesaplus.com/`](https://promisesaplus.com/) 获取使用 JavaScript 中 promises 的完整规范。Promises 或类似 promises 的对象几乎适用于你编写的任何 JavaScript 代码。

## $q

这是实现 promises 的服务：

```js
$q.defer()
```

### 返回值

这返回一个延迟对象。

### 描述

这是构建和使用 promises 的核心。首先，你需要获取一个延迟对象，它将包含解析和拒绝函数。当操作成功时，调用 `resolve` 函数；当它失败时，调用 `reject`。最后，使用 `defer.promise` 返回 promise。

使用 promise，你可以调用 `then`，它接受两个函数。第一个函数将在 `resolve` 被调用时执行，第二个函数将在 `reject` 被调用时执行。promise 可以被传递并在多次调用 `then`。

任何你进行异步操作的时候，都应该使用 promises。使用 Angular 时，你应该使用 `$q`，因为它与 rootscope 相关联。

这里有一个例子，当数字为偶数时函数会成功，当数字为奇数时会失败。然后，它执行两次，在控制台记录成功或失败。注意，函数是在控制器中创建的，但像这样的实用函数应该被放入服务中生产：

```js
firstModule.controller('SimpleController', ['$scope', '$q', function ($scope, $q) {
  function qTest(number) {
    console.log($q);
    var defer = $q.defer();

    if (number % 2 === 0) {
      defer.resolve(number);
    } else {
      defer.reject(number);
    }
    return defer.promise;
  };

  qTest(2)
    .then(function (n) { console.log('succeeded!'); }, function (n) { console.log('failed!'); });

  qTest(3)
    .then(function (n) { console.log('succeeded!'); }, function (n) { console.log('failed!'); });

}]);
```

# 表达式

表达式是 Angular 的一个特性。它们是 JavaScript 命令的子集，除了一些 Angular 函数。表达式在 Angular 的许多地方被使用。例如，每次你绑定数据时，你都可以使用一个表达式。这使得理解表达式变得很重要。

表达式将被评估为一个值，一个真值，一个作用域中的函数，或一个作用域中的变量。这使得它们强大，但它们确实有一些限制。

## JavaScript 中的表达式

你可以在表达式中使用一些 JavaScript，但不能使用所有的 JavaScript。以下是一些你可以在表达式中使用和在表达式中不能使用 JavaScript 的事情：

+   你可以使用字符串、数字、布尔值、数组字面量或对象字面量

+   你可以使用任何运算符，例如，`a + b`，`a && b` 或 `a || b`

+   你可以访问对象上的属性或在数组中查找值

+   你可以调用函数

+   你不能在表达式中使用流程控制语句，例如，一个 `if` 语句

### 上下文

当在 Angular 中评估表达式时，它将使用其所在的范围。这意味着你可以使用在范围内可访问的任何对象或函数。一个区别是，你将无法访问全局 `window`。例如，你无法在表达式中使用 `window.alert`。

# 指令

指令是 Angular 用来连接到**文档对象模型**（**DOM**）的。这允许 Angular 将应用程序每个部分应该做什么的关注点分开。这意味着控制器永远不应该直接操作 DOM。控制器应该只通过指令来改变 DOM。

## 规范化

当使用指令时，Angular 必须解析 DOM 并确定哪些指令适用于它。这是通过规范化所有元素和标签来完成的。规范化将删除任何";", "," " -", 或" _"。它还将从任何属性的起始位置删除`x-`和`data-`。例如，当查找`ngModel`时，以下所有内容都将匹配：

+   `x-ng:model`

+   `data-ng_model`

+   `ng-model`

+   `ng:model`

### 注意

如果你关心 HTML 5 验证，那么你应该使用`data-`规范化。

## 作用域

指令内部的范围可能会变得非常复杂。我们将探讨几种使用范围的不同方法。

首先是继承作用域。这意味着指令将使用控制器作用域中变量的值。在这个例子中，`test`必须在控制器的作用域中设置：

```js
firstModule.directive('firstTest', function () {
  return {
    template: 'This is the test directive. Test: {{test}}'
  };
});
```

### @ 绑定

下一个作用域修改将是隔离作用域。你可以设置返回的指令定义对象（见下一节）的作用域属性。这里使用的将改变作用域的构建方式。一个`@`符号将按单向绑定读取值到作用域中。一个`=`符号将创建双向绑定。每个都可以使用它们后面的属性名。

例如，要将单向绑定绑定到属性`scope-test`，使用：`@scopeTest`。以下是一个示例：

```js
firstModule.directive('firstTest', function () {
  return {
    restrict: 'AE',
    scope: {
      test: '@scopeTest'
    },
    template: 'This is the test directive. Test: {{test}}'
  };
});
```

这可以像这样使用：

```js
<div first-test scope-test="Scope Test Value"></div>
```

### = 绑定

这是使用`=`的示例。控制器需要在作用域中有一个变量传递给指令。以下是该指令的示例：

```js
firstModule.directive('firstTest', function () {
  return {
    restrict: 'AE',
    scope: {
      test: '='
    },
    template: 'This is the test directive. Test: {{test}}'
  };
});
```

指令的使用方式如下：

```js
<div first-test test="fromControllerScope"></div>
```

注意，用于连接作用域的属性被称作`test`。这是因为我们只使用了`=`，而没有使用`=nameOfVariable`。

### & 绑定

这是最后一种方式，即传递一个函数。这是通过`&`完成的。当创建具有隔离作用域的指令时，你会遇到一个问题，即让控制器知道何时采取行动。例如，当按钮被点击时，`& 绑定`允许控制器传递一个函数的引用，并让指令执行它。

这里是一个示例指令：

```js
firstModule.directive('firstTest', function () {
  return {
    restrict: 'AE',
    scope: {
      testAction: '&'
    },
    template: '<button ng-click="testAction()">Action!</button>'
  };
});
```

模板现在有一个按钮，当点击时会调用`testAction`。将要执行的`testAction`参数是从控制器传递过来的，并且存在于指令的作用域中。

这是该指令的 DOM 结构：

```js
<div first-test test-action="functionFromController()"></div>
```

另一种情况是，你可能需要从指令传递信息到控制器，例如事件对象或作用域中的变量。以下是一个示例，展示了 Angular 如何使`$event`对象在函数中可用。

这是该指令的示例：

```js
firstModule.directive('firstTest', function () {
  return {
    restrict: 'AE',
    scope: {
      testAction: '&'
    },
    template: '<button>Action!</button>',
    link: function ($scope, element) {
      element.on('click', function (e) {
        $scope.$apply(function () {
          $scope.testAction({ $e: e, $fromDirective: 'From Directive' });
        });
      });
    }
  };
});
```

关键是 `link` 属性。它在指令编译之后运行，并绑定到指令的实例。如果你有一个重复的指令，这一点很重要。接下来，你将使用类似 jQuery 的 API JQLite 监听点击事件。从这里，我们必须调用 `$apply`，因为我们不在 Angular 中，它将不会检测到这里的任何更改。最后，我们执行了传递的表达式，创建了两个变量 `$e` 和 `$fromDirective`，它们将在控制器中的函数中可用。

下面是指令在 DOM 中的样子：

```js
<div first-test test-action="functionFromController($e, $fromDirective)"></div>
```

两个变量，`$e` 和 `$fromDirective`，需要与你在指令中定义的名称相同。

## 修改 DOM

Angular 对关注点的分离非常明确。这有助于防止难以追踪的错误出现。其中一种分离是控制器不应该修改 DOM。那么在哪里修改 DOM 呢？在指令中。

指令是 Angular 应用程序中唯一应该知道 DOM 中元素的部分。当需要修改 DOM 时，应该由指令来完成。

Angular 包含一个类似 jQuery 的库，称为 jQLite。它具有许多 jQuery 的 DOM 操作函数。如果你熟悉 jQuery，那么你也就熟悉 jQLite。

下面是一个简单的指令示例，当按钮被点击时添加一个 `div` 元素。该示例使用 jQLite 的 `append` 函数：

```js
firstModule.directive('firstTest', function () {
  return {
    restrict: 'AE',
    scope: {
      testAction: '&'
    },
    template: '<button>Add a DIV!</button>',
    link: function ($scope, element) {
      element.on('click', function (e) {
        element.append('<div>I was added!</div>');
      });
    }
  };
});
```

## 事件绑定

在 Angular 应用程序中，只有指令应该监听 DOM 事件，例如点击事件。这使得处理程序所在的位置非常清晰。

下面是一个绑定到元素点击事件并记录到控制台的示例：

```js
firstModule.directive('firstTest', function () {
  return {
    restrict: 'AE',
    scope: {
      testAction: '&'
    },
    template: '<button>Add a DIV!</button>',
    link: function ($scope, element) {
      element.on('click', function (e) {
        console.log(e);
      });
    }
  };
});
```

此外，指令还可以让控制器传递一个函数，并在事件发生时执行它（参见 `=` 绑定）。最后，指令甚至可以将参数从指令传递到控制器。一个很好的例子是事件对象，它是从事件返回的（参见 `&` 绑定）。

## 指令定义对象

指令定义对象是告诉 Angular 如何构建指令的对象。

这里列出了最常用的属性及其快速概述：

+   `priority`: 具有更高优先级的指令将被首先编译。默认值为 `0`。

+   `scope`: 请参阅 *指令，作用域* 部分，以获取更深入的概述。

+   `controller`: 这可能有些令人困惑，但控制器用于在指令之间共享逻辑。另一个指令可以通过使用 `require` 并列出所需指令的名称来共享控制器中的代码。那个指令的控制器将被注入到 `link` 函数中。该函数将有以下定义：`function($scope, $element, $attrs, $transclude)`。

+   `require`: 需要的指令的字符串。在前面加上 `?` 将使其成为可选的，`^` 将在元素及其父元素中搜索，如果没有找到则抛出错误，而 `?^` 将在元素及其父元素中搜索，但它是可选的。函数实例将在指令之间共享。

+   `restrict`: 这将限制你可以在 DOM 中定义指令的方式。以下是选项，其中 `E` 和 `A` 是最常用的：

    +   E: 这代表元素

    +   A: 这代表属性和默认值

    +   C: 这代表类

    +   M: 这代表注释

+   `template`: 字符串形式的 HTML 或一个返回字符串的函数，该字符串包含 `function(element, attrs)` 的定义。

+   `templateUrl`: 从此 URL 加载模板。

+   `link`: 这是在放置任何 DOM 操作或监听器的地方。此函数将具有以下定义：`function(scope, element, attrs, requiredController, transcludeFunction)`。如果需要另一个指令，则其 `controller` 属性将是 `requiredController` 参数。

### 控制器 vs 链接

在构建指令时，何时使用 `controller` 或 `link` 函数可能会非常令人困惑。你应该将 `controller` 函数视为其他指令可以使用的接口。当一个指令需要另一个指令时，`controller` 返回值将被注入到 `link` 函数中。这将作为第四个参数。然后可以从 `link` 中访问该对象。`link` 函数用于任何 DOM 操作或监听器。

## 关键指令

这里有一些最常用的指令。

### ngApp

这是你的 Angular 应用程序的根：

```js
<element ng-app></element>
```

#### 参数

+   `ng-app(string)` 属性是默认模块的名称。

#### 描述

这将设置应用程序的根元素。这可以放在任何元素上，包括根 HTML 元素。

这将自动引导 Angular 加载在 `ng-app` 中定义的模块。

### ngModel

这将数据从作用域绑定到元素上：

```js
<input ng-model [ng-required ng-minlength ng-maxlength ng-pattern ng-change ng-trim]></input>
```

#### 参数

+   `ng-model(string)`: 这将在作用域中成为一个变量

+   `ng-required(boolean)`: 这将此输入设置为必填

+   `ng-minlength(int)`: 这设置最小长度

+   `ng-maxlength(int)`: 这设置最大长度

+   `ng-pattern(string)`: 这是要匹配的值的正则表达式

+   `ng-change(expression)`: 这是在值变化时将执行的 Angular 表达式

+   `ng-trim(boolean)`: 这决定是否要修剪值

#### 描述

这用于将控制器的作用域中的变量绑定到输入元素（文本、选择或文本区域）。这将是一个双向绑定，因此变量的任何更改都将更新其所有实例。

这里有一个简单的文本输入示例：

```js
<input ng-model="test" type="text"/>
  {{test}}
```

### ngDisabled

这可以禁用元素：

```js
<input ng-disabled></input>
```

#### 参数

+   如果 `ng-disabled(expression)` 属性评估为 true，则输入将被禁用。

#### 描述

这允许你通过代码轻松地禁用输入。表达式可以绑定到作用域变量或仅评估表达式中的内容。

这里是一个简单的示例，它禁用了文本输入：

```js
<input ng-disabled="true" type="text"/>
```

### ngChecked

这可以使一个元素被选中：

```js
<input ng-checked></input>
```

#### 参数

+   `ng-checked(expression)`属性是 Angular 表达式，它应该评估为 JavaScript 布尔值。

#### 描述

这是 Angular 根据控制器中的某些内容检查或取消选中复选框的方式。

这里是一个示例：

```js
<input ng-checked="true" type="checkbox"/>
```

### ngClass

这设置了一个元素的类：

```js
<element ng-class></element>
```

#### 参数

+   `ng-class(expression)`属性可以是字符串、数组或对象映射。

#### 描述

当表达式是字符串或数组时，字符串的值或数组的值将作为类应用于元素。当然，这些可以绑定到作用域中的变量。

这里是一个使用字符串设置类的示例。这里是将要应用的风格：

```js
.double { font-size: 2em; }
```

这里是应用它的代码：

```js
<div ng-class="classString">Text</div>
<input type="text" ng-model="classString" />
```

类绑定到`classString`变量的值。输入绑定，以便任何更改都会更新`div`上的类。一旦你输入双倍，它就会应用这种风格。

这里是一个使用对象映射的示例。对象的属性名是要应用的类，值必须是`true`。这里是一个使用相同类的类似示例：

```js
<div ng-class="{double: classString}">Text</div>
<input type="text" ng-model="classString" />
```

你会注意到，一旦你输入任何内容到输入框中，类就会被应用。这是因为非空字符串将是`true`。你可以使用这来应用所需的任何类，使用更多的属性。

### ngClassOdd 和 ngClassEvent

这些分别设置奇数或偶数元素的类：

```js
<element ng-class-odd></element>
<element ng-class-even></element>
```

#### 参数

+   `ng-class-odd`或`ng-class-even(expression)`属性必须评估为字符串或字符串数组，这些字符串将成为元素的类。

#### 描述

这两个指令相关，甚至可以应用于同一个元素。如果一个重复的元素是奇数，那么`ng-class-odd`将会被评估，对于`ng-class-even`也是如此。

这里是一个示例，它将使每个奇数元素的大小加倍，每个偶数元素的大小减半。以下是样式：

```js
.odd { font-size: 2em; }
.even { font-size: .5em; }
```

这里是元素。`ul`属性创建要重复的数据，每个`span`参数静态地将类设置为奇数或偶数。你还可以使用作用域中的变量：

```js
<ul ng-init="numbers=[1,2,3,4]">
  <li ng-repeat="number in numbers">
  <span ng-class-even="'even'" ng-class-odd="'odd'">{{number}}</span>
  </li>
</ul>
```

### ngRepeat

这是一个可以重复的模板：

```js
<element ng-repeat></element>
```

#### 参数

+   `ng-repeat(repeat_expression)`属性类似于表达式，但它有一些语法差异。请参阅描述以获取完整解释。

#### 描述

许多时候，你会有具有相似模板但每行数据都不同的数据。这就是`ng-repeat`发挥作用的地方。`ng-repeat`函数将重复重复表达式中的每个项目的 HTML。

重复表达式是一个表达式，它告诉`ng-repeat`将要循环哪些项目。以下是一些表达式的概述：

+   `item in collection`：这是一个经典的 foreach 语句。它将遍历集合（例如数组）中的每个项目。`item`参数将在模板内部可用。

+   `(key, value) in collection`：如果你的数据在一个对象中，并且你想能够将属性的名称与值关联起来，那么你需要使用这个。

+   `item in collection track by grouping`：这允许进行分组。分组将只为每个唯一的分组值提供一个项目。

+   `repeat expression | filter`：你可以过滤这些表达式中的任何一个，以得到数据的一个子集。

这是一个使用无序列表的示例。以下是控制器：

```js
firstModule.controller('SimpleController', ['$scope', function ($scope) { 
  $scope.items = [{id: 1, name: 'First' }, {id: 2, name: 'Second'}];
}]);
```

这是控制器的 HTML：

```js
<div ng-controller="SimpleController">
  <ul>
    <li ng-repeat="item in items track by item.id">
      ID: {{item.id}}
      Name: {{item.name}}
    </li>
  </ul>
</div>
```

### ngShow 和 ngHide

这些可以分别显示或隐藏元素：

```js
<element ng-show></element>
<element ng-hide></element>
```

#### 参数

+   `ng-show` 或 `ng-hide(expression)` 属性将被评估为 `true` 或 `false`，元素将相应地显示或隐藏。

#### 描述

这些指令允许您根据作用域内的数据显示或隐藏内容。

这是一个使用计数器来显示或隐藏元素的示例。你可以使用 `ng-hide` 并使用相反的逻辑。以下是控制器：

```js
firstModule.controller('SimpleController', ['$scope', function ($scope) {
  $scope.count = 0;
  $scope.increase = function () { $scope.count++; };
}]);
```

接下来是 HTML：

```js
<div ng-controller="SimpleController">
  <div ng-show="count == 0">Count is zero: {{count}}</div>
  <div ng-show="count > 0">Count is greater than zero: {{count}}</div>
  <button ng-click="increase()">Increase Count</button>
</div>
```

### ngSwitch

这创建了一个 `switch` 语句：

```js
<element ng-switch>
<element ng-switch-when></element>
<element ng-switch-default></element>
</element>
```

#### 参数

+   `ng-switch(expression)`：这是一个返回值的表达式。这个值将在评估 `ng-switch-when` 时使用。

+   `ng-switch-when(expression)`：当值匹配此表达式时，则此元素将可见。

+   `ng-switch-default`：当 `ng-switch-when` 不匹配时，此元素将显示。

#### 描述

这个指令的工作方式与 JavaScript 的 `switch` 语句非常相似。不同的案例会被测试，并使用匹配的案例。如果没有案例匹配，则使用默认元素。

这是一个在偶数和奇数之间跳转的示例。以下是控制器：

```js
firstModule.controller('SimpleController', ['$scope', function ($scope) {
  $scope.count = 0;
  $scope.increase = function () { $scope.count++; };
}]);
```

然后是 HTML：

```js
<div ng-controller="SimpleController">
  <div ng-switch="count % 2">
    <div ng-switch-when="0">Count is Even: {{count}}</div>
    <div ng-switch-when="1">Count is Odd: {{count}}</div>
  </div>
  <button ng-click="increase()">Increase Count</button>
</div>
```

### ngClick

这用于定义点击处理器：

```js
<element ng-click></element>
```

#### 参数

+   `ng-click(expression)` 属性是在元素被点击时将被评估的内容。可以使用函数，并且可以使用 `function($event)` 函数定义来访问事件对象。还可以传递作用域中的任何其他变量，例如，在重复器中的 `$indexin`。

#### 描述

这个指令允许您在元素点击时运行 JavaScript。通常，这会与作用域中的函数一起使用。

这是一个将增加 `scope` 变量的示例。首先，以下是控制器的 `scope` 变量：

```js
$scope.counter = 0;
$scope.functionFromController = function (e) { $scope.counter++; };
```

接下来，这是带有 `ng-click` 指令的元素。在这个例子中，您没有使用事件对象，但它演示了如何将其传递到控制器函数中：

```js
<button ng-click="functionFromController($event)">Click Me!</button>
{{counter}}
```

### ngDblclick

这用于定义双击处理器：

```js
<element ng-dblclick></element>
```

#### 参数

+   `ng-dblclick(expression)` 函数，可以使用函数，并且可以使用 `function($event)` 函数定义来访问事件对象。还可以传递作用域中的任何其他变量，例如，在重复器中的 `$index`。

#### 描述

这个指令与 `ngClick` 非常相似。当元素被双击时，表达式将被评估。

这是一个示例，当双击时将增加计数器的 `div`。以下是控制器的变量：

```js
$scope.counter = 0;
$scope.functionFromController = function (e) { $scope.counter++; };
```

这里是元素和指令：

```js
<div ng-dblclick="functionFromController($event)">Double Click Me!</div>
{{counter}}
```

### ngMousedown、ngMouseup、ngMouseover、ngMouseenter 和 ngMouseleave

这些指令用于定义鼠标事件处理器：

```js
<element ng-mousedown></element>
<element ng-mouseup></element>
<element ng-mouseover></element>
<element ng-mouseenter></element>
<element ng-mouseleave></element>
```

#### 参数

+   在`ng-mouse*(expression)`函数中，可以使用语句或函数，并且可以通过`function($event)`函数定义使用事件对象。还可以传递作用域中的其他变量，例如，在重复器中的`$index`。

#### 描述

这些指令被分组在一起，因为它们都是相关的。当鼠标执行动作（按下、释放、悬停、进入或离开）时，表达式将被评估。

这个例子有点疯狂，但它演示了如何使用这些事件。这里是控制器中的相关部分：

```js
$scope.counter = 0;
$scope.functionFromController = function (e) { $scope.counter++; };
```

这里是指令列表：

```js
<div ng-mousedown="functionFromController($event)" ng-mouseup="functionFromController($event)"
  ng-mouseenter="functionFromController($event)" ng-mouseleave="functionFromController($event)"
  ng-mouseover="functionFromController($event)">Mouse Over Me!</div>
{{counter}}
```

### ngMousemove

这用于定义鼠标移动处理器：

```js
<element ng-mousemove</element>
```

#### 参数

+   在`ng-mousemove(expression)`函数中，可以使用语句或函数，并且可以通过`function($event)`函数定义使用事件对象。还可以传递作用域中的其他变量，例如，在重复器中的`$index`。

#### 描述

这个指令在鼠标移动时触发。除非指令应用于整个页面，否则事件将仅限于应用到的元素。

这里是一个示例，它将显示鼠标的*x*和*y*坐标。这里是相关的作用域变量：

```js
$scope.x = 0;
$scope.y = 0;
$scope.functionFromController = function (e) {
  $scope.x = e.pageX;
  $scope.y = e.pageY;
};
```

然后是指令：

```js
<div ng-mousemove="functionFromController($event)">Move the mouse to see the x and y coordinates!</div>
{{x}}
{{y}}
```

### ngKeydown、ngKeyup 和 ngKeypress

这些指令用于定义按键处理器：

```js
<element ng-keydown></element>
<element ng-keyup></element>
<element ng-keypress></element>
```

#### 参数

+   在`ng-key*(expression)`函数中，可以使用语句或函数，并且可以通过`function($event)`函数定义使用事件对象。还可以传递作用域中的其他变量，例如，在重复器中的`$index`。

#### 描述

这些指令会在按键按下或释放时触发。

这里是一个示例，它将检索按下的键的键码。这里是相关的`scope`变量：

```js
$scope.keyCode = 0;
$scope.functionFromController = function (e) {
  $scope.keyCode = e.keyCode;
};
```

接下来是指令：

```js
<input type="text" ng-keydown="functionFromController($event)"/>
Which key: {{keyCode}}
```

### ngSubmit

这些指令用于定义提交处理器：

```js
<form ng-submit></form>
```

#### 参数

+   在`ng-submit(expression)`函数中，可以使用语句或函数，并且可以通过`function($event)`函数定义使用事件对象。还可以传递作用域中的其他变量，例如，在重复器中的`$index`。

#### 描述

这个指令用于捕获表单的`submit`事件。如果表单没有 action 属性，那么这将防止表单重新加载当前页面。

这里是一个简单的示例，它将事件对象记录到`submit`时的控制台。这里是相关的指令：

```js
<form ng-submit="functionFromController($event)"><input type="submit" id="submit" value="Submit" /></form>
```

接下来是控制器代码：

```js
$scope.functionFromController = function (e) {
  console.log(e);
};
```

### ngFocus 和 ngBlur

这些指令分别用于定义焦点和失焦处理器：

```js
<input, select, textarea, a, window ng-focus></input>
<input, select, textarea, a, window ng-blurs></input>
```

#### 参数

+   在`ng-focus(expression)`函数中，可以使用语句或函数，并且可以通过`function($event)`函数定义使用事件对象。还可以传递作用域中的其他变量，例如，在重复器中的`$index`。

#### 描述

当元素获得焦点时，`ngFocus` 处理程序将被触发，而当它失去焦点时，`ngBlur` 将被触发。以下是一个使用文本输入并将事件记录到控制台的示例：

```js
<input type="text" ng-focus="functionFromController($event)"/>
```

以下是控制器代码：

```js
$scope.functionFromController = function (e) {
  console.log(e);
};
```

### ngCopy、ngCut 和 ngPaste

```js
<element ng-copy></element>
<element ng-cut></element>
<element ng-paste></element>
```

#### 参数

+   `ng-copy`、`ng-cut` 或 `ng-paste(expression)`：可以使用语句或函数，并且可以使用 `function($event)` 函数定义来访问事件对象。还可以将作用域中的其他变量传递，例如，在重复器中的 `$index`。

#### 描述

这些是在文本被剪切、复制或粘贴时将触发的事件。以下是一个使用所有三个事件的示例：

```js
<input type="text" ng-cut="functionFromController($event)" ng-copy="functionFromController($event)" ng-paste="functionFromController($event)" />
```

以下是控制器代码：

```js
$scope.functionFromController = function (e) {
  console.log(e);
};
```

# 全局变量

这组函数可以在 Angular 的任何地方执行，而无需注入它们。它们主要是实用函数，允许你更容易地或以 Angular 方式做事。

## 扩展

这提供了一种合并两个对象的方法：

```js
angular.extend(srcObject, destObject)
```

### 参数

+   `srcObject(object)`: 将复制属性的对象

+   `Destobject(object)`: 扩展对象将复制属性到的对象

### 返回值

这将返回 `destObject` 的引用。

### 描述

在 JavaScript 中，没有内置的方法可以使用另一个对象扩展对象。这个函数正是这样做的。

以下是一个简单的示例，它将扩展一个对象以包含另一个对象的属性：

```js
var first = { first: '1' };
var second = { second: '2' };
var extended = angular.extend(first, second);
//extended will be {first:'1', second:'2'}
```

## noop

这是无操作函数：

```js
angular.noop()
```

### 参数

+   `any` 或 `none` 函数用作这些函数不执行任何操作，你可以传入零个或任意多个参数。

### 返回值

这将返回 `undefined`。

### 描述

有一个不执行任何操作（`noop`）的函数很有用。一个很好的例子是当你有一个作为参数的函数，它是可选的。如果没有传入，你可以运行 `noop`。

这是一个简单的示例，演示了前面解释的场景：

```js
function test(doSomething) {
  var callback = cb || angular.noop;
  callback('output');
}
```

## isUndefined

这检查某个东西是否未定义：

```js
angular.isUndefined(object)
```

### 参数

+   `object(any type)` 函数可以是任何变量。

### 返回值

这返回一个布尔值。

### 描述

这表示变量是否已定义。

## 复制

这将创建一个对象的副本：

```js
angular.copy(srcObject, [destObject])
```

### 参数

+   `srcObject(any type)`: 这是将要被复制的源对象。

+   `destObject(same type as srcObject)`: 这是可选的。如果提供，它将是复制操作的目的地。

### 返回值

这个函数返回 `srcObject` 的副本。如果提供了 `destObject`，则返回它。

### 描述

当你需要复制一个对象而不是修改原始对象时，请使用此函数。

## 绑定

这将一个函数绑定到一个对象上：

```js
angular.bind(self, function, [args..])
```

### 参数

+   `self(object)`: 这将在函数中设置 `this`（内部自我引用）

+   `Function(function)`: 这是正在绑定的函数

+   `Args(any type)`: 这些是绑定到函数的参数

### 返回值

这是新绑定的函数。

### 描述

这创建了一个新的绑定函数，它将在 `self` 的上下文中执行。这允许你定义一个函数，然后在不同的上下文中多次执行它。通过绑定参数，这进一步扩展了。

这是一个虚构的例子，但演示了原理。首先是一个依赖于上下文来执行的函数。它将第一个参数添加到 `this` 中，并乘以该结果：

```js
function addAndMultiply(toAdd, toMultiply) {
  return (this + toAdd) * toMultiply;
}
```

接下来，你将使用 `angular.bind` 来创建一个新的函数：

```js
var newFunc = angular.bind(4, addAndMultiply, 1);
```

这可以执行，并且返回值将是乘数的 5 倍。在 `newFunc` 中，`this` 是 `4`，而 `toAdd` 是 `1`，所以内层括号总是 `5`。例如，这将返回 `10`：

```js
newFunc(2);
```

# 表单

表单是 HTML 中向服务器发送数据的核心部分。因此，Angular 有一些与表单一起工作的额外功能。

## ngModel

每个表单输入都需要定义 `ngModel` 以在作用域中存储值。见 *指令*、*ngModel* 了解更多信息。

以下是一个简单的表单，它绑定两个文本输入：

```js
<form name="form">
  First Name: <input type="text" name="firstname" ng-model="data.firstName" />
  Last Name: <input type="text" name="lastName" ng-model="data.lastName" />
</form>
{{data.firstName}} {{data.lastName}}
```

## CSS 类

Angular 会自动将 CSS 类添加到表单和元素中，然后你可以使用 CSS 选择器来定位它们。以下是 CSS 类及其应用情况列表：

+   `ng-valid`：表示表单或元素有效

+   `ng-invalid`：表示表单或元素无效

+   `ng-pristine`：表示控件尚未更改

+   `ng-dirty`：表示控件已被更改

## 验证

Angular 有一些特性使得验证变得非常简单。首先，Angular 会使用任何 HTML 5 输入类型。这允许浏览器执行一些验证，但 Angular 仍然会跟踪任何错误。

接下来，你可以使用 `ngModel` 的任何内置验证指令。列表包括 `required`、`pattern`、`minLength`、`maxLength`、`min` 和 `max`。见 *指令*、*ngModel* 了解每个指令的更多信息。当输入验证失败时，值将不会传递到绑定的作用域变量中。

以下是一个设置 `firstName` 最小长度的示例：

```js
 <form name="form">
  First Name: <input type="text" name="firstname" ng-minlength="10" ng-model="data.firstName" />
  Last Name: <input type="text" name="lastName" ng-model="data.lastName" />
</form>
{{data.firstName}} {{data.lastName}}
```

当表单有一个 `name` 属性时，它将绑定到该名称的作用域。当在表单中给输入赋名时，它们将绑定到该表单。这允许你使用其他指令与这些值一起使用。

以下是一个示例，当文本输入未达到 10 个字符的最小长度时，将显示错误消息：

```js
<form name="form">
  First Name: <input type="text" name="firstname" ng-minlength="10" ng-model="data.firstName" />
  <div ng-show="form.firstname.$invalid">First name must be 10 characters!</div>
  Last Name: <input type="text" name="lastName" ng-model="data.lastName" />
</form>
{{data.firstName}} {{data.lastName}}
```

验证消息仅在字段无效时显示。

表单将具有 `$dirty`、`$invalid`、`$pristine` 和 `$valid` 属性，可以在指令或作用域中使用。输入将具有相同的属性，并且还有一个 `$error` 对象，它将每个失败的验证作为属性。在前面的例子中，这意味着当输入未通过该验证时，`form.firstname.$error.minLength` 将返回 `true`，当它有效时返回 `false`。

### 自定义验证器

当你需要创建自己的逻辑时，你可以构建一个自定义验证器。你需要创建一个新的指令，引入 `ngModel`，并在将其传递到 `link` 函数时，通过 `ngModel` 控制器的 `$parsers` 对象传递值。然后，根据传递的值是否通过验证使用 `$setValidity`。

这里是一个自定义验证器的示例，其中输入的值不能设置为 `josh`：

```js
firstModule.directive('notJosh', function () {
  return {
    require: 'ngModel',
    link: function (scope, element, attrs, ctrl) {
      ctrl.$parsers.unshift(function (viewValue) {
        if (viewValue.toUpperCase() === 'JOSH') {
          ctrl.$setValidity('notJosh', false);
          return undefined
        }
        else {
          ctrl.$setValidity('notJosh', true);
          return viewValue;
        }
      });
    }
  }
});
```

# 测试

测试应该始终是任何开发项目的重要组成部分。Angular 从一开始就被设计为可测试的。它具有清晰的关注点分离；例如，你不需要构建完整的 DOM 来测试控制器。Angular 还在所有地方使用依赖注入，这使得模拟对象变得非常容易。

## 使用 Jasmine 和 Karma 进行单元测试

**Jasmine** 和 **Karma** 是两个允许你快速轻松地测试 Angular 代码的工具。

### Jasmine

这是我们将使用的实际单元测试库。Jasmine 是一个行为驱动测试框架，编写起来非常简单。

### Karma

Karma 是一个测试运行器，它将监视你的文件并在文件发生变化时自动启动测试。它运行在 Node.js 上，因此你必须安装它。然后，你可以使用 `npm` 安装 Karma：

```js
 install karma-jasmine
```

Karma 可以监视你的测试文件，并在任何文件发生变化时重新运行它们。如果存在任何问题，你还可以在浏览器中调试测试。它是 Jasmine 的绝佳补充，Google 推荐两者都用于测试。

## ngMock

`ngMock` 处理器用于在测试时帮助模拟应用程序。在测试时，你不会想创建一个完整的 DOM 来加载单个模块。这就是 `ngMock` 可以创建一个控制器实例以供使用的地方，然后我们可以对其进行测试。

### 模块

这允许你加载一个模块：

```js
module(moduleName)
```

#### 参数

+   `moduleName(string)` 函数执行

#### 描述

你没有根元素可以放置 `ng-app`，因此 `module` 允许我们加载一个模块以获取其控制器、服务和指令的访问权限。你应该只在加载 `ngMock` 后在测试中使用此函数。

### 注入

这获取 Angular 注入服务：

```js
inject(toBeInjected)
```

#### 参数

+   `toBeInjected(function)` 函数的工作方式与其他注入函数类似。列出要注入的对象，它们将在函数体中可用。

#### 描述

使用注入来获取内置的 Angular 服务，例如 `$controller` 和 `$compile`，或者使用它来获取加载模块的服务。

如果依赖项被下划线包裹，注入可以加载依赖项。例如，如果使用 `_$compile_`，注入将加载 `$compile`。这是因为在测试中，我们需要创建 `$compile` 服务的引用，并且很可能会希望将 `$compile` 作为变量名。下划线允许你注入它并使用 `$compile` 变量。

### $httpBackend

这可以创建一个模拟响应：

```js
$httpBackend.when(requestType, url, [requestParameters], [headers])
$httpBackend.expect(requestType, url, [requestParameters], [headers])
$httpBackend.respond(response)
```

#### 参数

+   `requestType(string)`: 这是请求的 HTTP 方法

+   `url(string)`: 这是请求的 URL

+   `requestParameters(object)`：这些是请求的参数

+   `headers(object)`：这些是请求的头信息

+   `response(string, object)`：这是请求的响应

#### 返回值

这将返回一个可以调用`respond`的处理程序。

#### 描述

在单元测试中执行 AJAX 调用是一个不应该做的事情的例子。对于单元测试来说，有太多您无法控制的事情。`$httpBackend`处理器由`ngMock`提供，以便您可以创建模拟响应。

处理器至少必须匹配预期的方法和 URL。如果您计划使用它们进行特定请求，还可以匹配可选参数和头信息。

当请求匹配时，您可以发送一个字符串或对象作为响应。这允许您创建一个使用`$httpBackend`的测试，因为您知道响应将会是什么。

`expect`和`when`之间的区别在于`expect`必须在测试中调用，而`when`没有这个要求。

## 单元测试控制器

这里是一个控制器简单单元测试的例子。首先，您必须创建一个控制器，然后在我们的测试中加载它：

```js
var firstModule = angular.module('firstModule', ['ngMock']);
firstModule.controller('SimpleController', ['$scope', function ($scope) {
  $scope.test = 'HEY!';
}]);
```

您现在可以创建测试。在测试中，您必须加载`firstModule`模块，注入`$controller`，并创建`SimpleController`的实例。以下是测试代码：

```js
describe('SimpleController', function () {
  var scope = {};
  var simpleCtrl;
  beforeEach(module('firstModule'));

  it("should have a scope variable of test", inject(function ($controller) {
    expect(scope.test).toBe(undefined);
    simpleCtrl = $controller('SimpleController', { $scope: scope });
    expect(scope.test).toBe('HEY!');
  }));
});
```

## 单元测试指令

这个例子将向您展示如何测试一个指令。首先，创建指令：

```js
firstModule.directive('simpleDirective', function () {
  return {
    restrict: 'E',
    scope:{
      test: '@'
    },
    replace: true,
    template: '<div>This is an example {{test}}.</div>'
  };
});
```

接下来，您需要加载模块，注入`$compiler`和`$rootscope`，编译指令，并最终至少启动一次消化循环以绑定任何值：

```js
describe('simpleDirective', function () {
  var $compile, $rootScope;
  beforeEach(module('firstModule'));
  beforeEach(inject(function (_$compile_, _$rootScope_) {
    $compile = _$compile_;
    $rootScope = _$rootScope_;
  }));

  it('should have our compiled text', function () {
    var element = $compile('<simple-directive test="directive"></simple-directive>')($rootScope);
    $rootScope.$digest();
    expect(element.html()).toContain('This is an example directive.');
  });

});
```

## 单元测试服务

最后的测试示例将测试一个服务。首先，创建服务：

```js
firstModule.factory('firstFactory', ['$http', function ($http) {
  return {
    addOne: function () {
      return $http.get('/test', {})
      .then(function (res) {
        return res.data.value + 1;
      });
    }
  }
}]);
```

接下来，您需要加载模块，注入`$httpBackend`和服务工厂，创建一个响应，并加载该响应。注意`$httpBackend.flush()`的使用。这将把响应发送到任何打开的请求：

```js
describe('firstFactory', function () {
  var $httpBackend, testingFactory, handler;

  beforeEach(module('firstModule'));
  beforeEach(inject(function (_$httpBackend_, firstFactory) {
    $httpBackend = _$httpBackend_;
    handler = $httpBackend.expect('GET', '/test')
    .respond({ value: 1 });
    testingFactory = firstFactory;
  }));

  it('should run the GET request and add one', function () {
    testingFactory.addOne()
    .then(function (data) {
      expect(data).toBe(2);
    });
    $httpBackend.flush();

  });
});
```
