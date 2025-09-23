# 第二章：理解 Ember.js 对象和混入

在上一章学习了如何创建基本的 Ember.js 应用程序之后，本章将介绍 Ember.js 对象，这是其他基础类的基础。因此，本书中讨论的大部分概念将贯穿全书。到本章结束时，我们将能够：

+   创建 Ember.js 对象

+   定义、获取和设置对象属性

+   定义计算属性

+   注册属性变更观察者

+   使用混入

# 在 Ember.js 中创建对象

我们都知道如何在 JavaScript 中定义和创建函数对象的实例，如下面的代码所示：

```js
function Point(x, y, z){
  this.x = x;
  this.y = y;
  this.z = z;
}

Point.prototype.logX = function(){
  console.log(this.x);
}
Point.prototype.logY = function(){
  console.log(this.y);
}

Point.prototype.logZ = function(){
  console.log(this.z);
}

var point = new Point(3, 5, 7);
point.logX();
// 3point.logY();
// 5point.logZ();
// 7
```

上述示例创建了一个具有三个方法的定义的 `Point` 对象的实例。

Ember.js 使用 JavaScript 原型来模拟面向对象特性。更重要的是，它引入了便利性，使得在事件驱动的浏览器环境中更容易继承和管理对象。一个类，即对象定义，通常通过 *扩展* 另一个用户定义的或内置的类创建，通常是 `Ember.Object`。类有两个方法，`create` 和 `extend`，分别用于创建对象实例和执行继承。例如，前面的代码片段在 Ember.js 中的实现如下：

```js
var Point = Ember.Object.extend({
  x: null,
  y: null,
  z: null,
  logX: function(){
    console.log(this.get('x'));
  },
  logY: function(){
    console.log(this.get('y'));
  },
  logZ: function(){
    console.log(this.get('z'));
  }
});
```

我们刚刚创建了一个具有三个属性 `x`、`y` 和 `z` 以及它们相应的日志方法的 Ember.js 类。要创建这个类的新实例，我们将调用类上的 `create()` 方法，如下面的示例所示：

```js
var point = Point.create({
  x: 3,
  y: 5,
  z: 7
}); point.logX();
// 3point.logY();
// 5point.logZ();
// 7
```

我们可以更进一步，通过使用 `extend()` 方法扩展我们的 `Point` 类来形成一个新的类。例如，我们可以定义一个 `Vector` 类，它定义了一个 `add()` 方法，该方法向提供的向量中添加内容，如下面的代码所示：

```js
var Vector = Point.extend({
  add: function(vector){

   var x = this.get('x') + vector.get('x');
   var y = this.get('y') + vector.get('y');
   var z = this.get('z') + vector.get('z');

   this.set('x', x);
   this.set('y', y);
   this.set('z', z);

  }
});

var vectorA = Vector.create({
  x: 3,
  y: 5,
  z: 7
});

var vectorB = Vector.create({
  x: 1,
  y: 2,
  z: 3
});

vectorA.add(vectorB);

vectorA.logX();
// 4vectorA.logY();
// 7vectorA.logZ();
// 10
```

在示例中将 `Point` 类扩展为 `Vector` 类之后，我们创建了两个名为 `vectorA` 和 `vectorB` 的向量，并最终将它们组合在一起。

# 访问对象属性

我们刚刚看到了 Ember.js 对象是如何创建的。你注意到 Ember.js 对象属性是如何访问的吗？Ember.js 提供了 `get` 和 `set` 属性访问器方法。为什么不直接访问这些值呢？嗯，这些方法用于重新计算值，并在必要时通知所做的任何更改。例如：

```js
var point = Point.create();
point.set('x', 3);
console.log(point.get('x')); // 3
```

属性也可以通过 `getProperties` 和 `setProperties` 方法一起读取和设置。这可以防止 Ember.js 不必要地发出太多关于这些更改的通知，例如：

```js
var point = Point.create();
point.setProperties({
  x: 1,
  y: 2,
  z: 3
});
console.log(point.getProperties('x', 'y', 'z'));
//{x: 1, y: 2, z: 3}
```

# 定义类实例方法

类也可以定义实例方法。这些方法与对象属性的签名类似，如下面的代码所示：

```js
var MyClass = ClassName.extend({
  methodName: function(){
    // method implementation
  }
});
```

Ember.js 通过使用 `_super()` 方法提供了在扩展类中重用父方法实现的能力。例如，以下示例重新实现了 `Point` 类中的 `logX` 方法并重用它：

```js
var MyPoint = Point.extend({
  logX: function(){
   var x = this._super(); // call parent method
   console.log('x: %s', x);
  }
});
var myPoint = MyPoint.create({
  x: 3
});
myPoint.logX(); // x: 3
```

Ember.js 对象通常定义一个名为 `init()` 的构造方法，它在实例创建时被调用。任何初始化都应该在这个方法内完成。值得注意的是，应该始终在继承的方法（如 `init()`）上调用 `_super()` 方法，以避免丢失父实现，例如：

```js
var Book = Ember.Object.extend({
  init: function(){
    this._super();
    this.set('name', 'Mastering Ember.js'); // initialization
  }
});
```

# 定义计算属性

什么是计算属性？计算属性是指其值由函数返回的属性。例如：

```js
var Movie = Ember.Object.extend({
  name: function(){
    return 'Transformers';
  }.property()
});

var movie = Movie.create();
console.log(movie.get('name')); // Transformers
```

上述示例创建了一个计算属性 `name`，它返回电影实例的名称。

我们只是通过将 `property()` 函数链接到它来将一个方法转换为一个计算属性。计算属性的真正力量在于它们能够根据预定义的依赖属性产生不同的值。这些依赖属性通常作为参数传递给 `property()` 函数。例如：

```js
var Movie = Ember.Object.extend({
  year: '2007',
  seriesNumber: '1',
  name: function(){
    return this.get('seriesNumber') + '. Transformers - ' + this.get('year');
  }.property('year', 'seriesNumber')
})

var movie = Movie.create();
console.log(movie.get('name')); // 1\. Transformers – 2007

movie.set('year', '2014');
movie.set('seriesNumber', '4');
console.log(movie.get('name')); // 4\. Transformers – 2014
```

在前述示例中，每当电影的 `seriesNumber` 和 `year` 发生变化时，`name` 属性都会被重新计算。

计算属性也可以有可枚举数据的属性依赖。可以使用 `@each` 辅助函数在这样类型的属性上设置计算属性。例如：

```js
var Country = Ember.Object.extend({
  stateNames: function(){
  return this.get('states').map(function(state){
    return state['name'];
  });
  }.property('states.@each.name')
});

var country = Country.create({
  states: [ {name: 'Texas'}, {name: 'Ohio'} ],
});
console.log(country.get('stateNames')); // ['Texas', 'Ohio']

country.set('states', [ {name: 'Alabama'}, {name: 'Arizona'} ]);
console.log(country.get('stateNames')); // ['Alabama', 'Arizona']
```

在前述示例中，我们创建了一个具有两个 `states` 的 `country` 对象。然后我们定义了一个计算属性 `stateNames`，它返回一个包含州名的数组。任何州名的更改都会导致属性重新计算。

# 定义属性观察者

除了计算属性外，您还可以将 `observers` 设置为属性。观察者是在它们订阅的属性发生变化时被调用的函数。它们具有与计算属性相同的签名，但使用 `observers` 函数，如下面的代码所示：

```js
var MyClass = ClassName.extend({
  observerName: function(){
    // observer implementation
  }.observes([properties, ...])
});
```

以下示例定义了一个会话类，它设置了一个观察者，当会话过期时，用户需要重新登录：

```js
var Session = Em.Object.extend({
  expiredChanged: function(){
    if (this.get('expired')){
      window.location.assign('/login');
    }
  }.observes('expired')
});
```

观察者没有严格的命名约定，但大多数开发者通过在观察的属性后附加 `DidChange` 来命名观察者，如下所示：

```js
var Song = Em.Object.extend({
  playedDidChange: function(){

  }.observes('played')
});
```

就像计算属性一样，观察者也可以订阅无限数量的属性：

```js
var Player = Em.Object.extend({
  inMotion: function(){

  }.observes('running', 'walking)
});
```

在这里，当 `running` 或 `walking` 属性中的任何一个发生变化时，`inMotion` 属性将被重新计算。

观察者也可以使用 `addObserver()` 和 `removeObserver()` 方法分别设置和拆除。当您想自己管理观察者时，这些方法变得非常有用。例如，前面的示例可以重写为：

```js
var Session = Em.Object.extend({
  init: function(){

    var self = this;

    self._super();
    self.addObserver('expired', function(){
    if (self.get('expired')){
      self.removeObserver('expired');
      window.location.assign('/login');
    }
  });

}
});
```

如前述代码所示，`addObserver` 方法至少需要两个参数：要观察的属性以及属性变化时调用的函数。在我们的示例中，我们还通过调用 `removeObserver` 方法来拆除观察者监听器。此方法需要一个参数，即要解绑的属性。

Ember.js 还提供了一种方法来传递在 `observer` 函数中使用的上下文。例如：

```js
var Session = Em.Object.extend({
  init: function(){

    this._super();
    this.addObserver('expired', this, function(){
      if (this.get('expired')){
        this.removeObserver('expired');
        window.location.assign('/login');
      }
    });

  }
});
```

需要注意的是，观察者只会在对象初始化之后发生的属性变化上触发。可以将一个 `on('init')` 方法应用于观察者，使其在对象初始化期间可能发生的任何变化上触发。例如：

```js
var Song = Em.Object.extend({
  skipped: false,
  played: false,
  skippedDidChange: function(){
    // does not fire on object initialization
    console.log('song was skipped');
  }.observes('skipped'),
  playedDidChange: function(){
    // fires on object initialization
    console.log('song finished playing');
  }.observes('played').on('init'),
  init: function(){
    this._super();
    this.set('skipped', true);
    this.set('played', true);
  }
});

Song.create();
```

示例定义了两个观察者：`skippedDidChange` 和 `playedDidChange`，其中，只有后者在对象初始化后会被调用。

# 创建属性绑定

Ember.js 提供了对单向和双向绑定的支持。绑定是同一对象或不同对象两个属性之间的链接，它们总是保持同步。这意味着对一个属性的更新会导致另一个属性更新为新值。绑定在以下签名中定义：

```js
property: Ember.computed.alias('otherProperty'),
```

在这种情况下，两个属性 `property` 和 `otherProperty` 总是保持同步。以下是一个例子：

```js
var author = Em.Object.create({
  name: 'J. K. Rowling'
});

var book = Em.Object.create({
  name: 'Harry Potter',
  authorName: Ember.computed.alias('author.name')
});

console.log(book.get('authorName')); // J. K. Rowling

author.set('name', 'Joanne Rowling');
console.log(book.get('authorName')); //  Joanne Rowling
```

在前面的例子中，`book` 实例有一个绑定到创建的全局作者实例的 `author` 属性。对作者名字的任何更改都会反映在绑定的书籍作者属性中。同样，对书籍作者属性的任何更改都会传播回全局作者，如下所示：

```js
book.set('authorName', 'Joanne Jo Rowling');
console.log(author.get('name')); // Joanne Jo Rowling
```

这是一个双向绑定的例子，其中对任一属性的更新都会导致另一个属性更新为新值。

Ember.js 也支持单向绑定，其中更新是单向的。一个属性可以订阅来自不同属性的更新，但如果前者发生变化，它不会更新后者。例如：

```js
var author = Em.Object.create({
  name: 'J. K. Rowling'
});

var book = Em.Object.create({
  name: 'Harry Potter',
  authorName: Ember.computed.oneWay('author')
});

console.log(book.get('authorName')); // J. K. Rowling

book.set('author.name', 'Joanne Rowling');
// author's name remains unchanged
console.log(author.get('name')); //  J. K. Rowling
```

在前面的代码片段中，书籍属性的变化不会影响绑定的作者的名字属性。

# 使用混入

混入是抽象定义，它定义了类和对象可以重用的方法和属性。例如，考虑这两个对象：

```js
var myView = Em.View.create({
  sum: function(a, b){
    return a+b;
  }
});

var myController = Em.Controller.create({
  sum: function(a, b){
    return a+b;
  }
});
```

这两个对象共享一个可以抽象为混入的公共函数：

```js
var sumMixin = Em.Mixin.create({
  sum: function(a, b){
    return a+b;
  }
});

var myView = Em.View.createWithMixins(sumMixin);

var myController = Em.Controller.createWithMixins(sumMixin);
```

可以在创建或定义时向对象或类传递任意数量的混入：

```js
App.Number = Em.Object.extend(sumMixin, diffMixin, productMixin);
```

需要注意的是，混入总是**创建**的，而不是**扩展**的。示例还显示，重用混入的对象在实例化时总是使用 `createWithMixins` 方法创建，而不是 `create` 方法。然而，当应用混入时，类仍然使用 `extend` 方法。

# 重新打开类和实例

有时，有必要更新类实现而不重新定义它们。通常在我们不希望扩展内置类，只想更新它们的实现时，这是必要的。Ember.js 将其称为类的重新打开和对象的重新打开。可以使用 `reopenClass` 方法重新实现类方法和属性，而实例方法和属性可以使用 `reopen` 方法更新。然而，不建议更改内置类，因为它们可能在未来的版本中发生变化。

例如：

```js
var Book = Em.Object.extend();

Book.reopen({
  id: null,
  title: null,
  purchase: function(){
    console.log('sold');
  }
});

Book.reopenClass({
  getById: function(id){
      return Book.create({
        id: '456',
        title: 'Harry Potter'
      });
  } 
});

Book.create({
  id: 456,
  title: 'Harry Potter'
});

var book = Book.getById(456);

book.purchase();
```

在前面的示例中，我们向已定义的`Book`类添加了一个实例方法`purchase`和两个属性`id`和`name`。我们还添加了一个类方法`getById`，而没有扩展该类。

# 事件订阅

在应用程序中通知更改的另一种方式是通过事件订阅。这种范式在 Node.js 中被广泛用于在不同组件之间传递消息和事件。Ember.js 提供了`Ember.Evented`混入，可以用来轻松地实现这一点。例如，在棋盘游戏中，棋块可以订阅来自执行器的指令，如下例所示：

```js
var GRID_SIZE = 4; 
var actuator = Em.Object.createWithMixins(Em.Evented);

var block = Em.Object.createWithMixins(Em.Evented);
acuator.on('moveRight', function(){
  var x = block.get('x') + 1;
  x = x % GRID_SIZE; 
  block.set('x', x);
});

actuator.trigger('moveRight');
```

混入提供了五个基本方法，其中两个在先前的示例中已经展示：

+   `on`：用于订阅事件

+   `off`：用于禁用订阅

+   `one`：用于一次性订阅事件

+   `trigger`：用于触发事件

+   `has`：用于检查是否已订阅事件

# 概述

本章的重点是介绍 Ember.js 对象。我们将在下一章中广泛使用这些对象，我们将学习如何在 Ember.js 中使用路由实现状态管理。我们将讨论如何构建路由和路由器。话虽如此，你应该在本章中学习了以下 Ember.js 对象概念：

+   在 Ember.js 中创建对象和类

+   获取和设置对象属性

+   定义计算属性

+   定义属性观察者

+   创建属性绑定

+   使用混入

+   重新打开 Ember.js 类

在下一章中，我们将讨论路由，这是从`Ember.Object`扩展出来的这些类之一。
