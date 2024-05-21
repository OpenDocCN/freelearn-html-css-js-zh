# 第七章。JavaScript 中的函数式和面向对象编程

你经常会听到 JavaScript 是一种空白语言，其中空白可以是面向对象的、函数式的或通用的。本书将 JavaScript 作为一种函数式语言进行了重点研究，并且已经付出了很大的努力来证明它是这样的。但事实上，JavaScript 是一种通用语言，意味着它完全能够支持多种编程风格。与 Python 和 F#不同，JavaScript 是多范式的。但与这些语言不同，JavaScript 的 OOP 方面是基于原型的，而大多数其他通用语言是基于类的。

在本章中，我们将把函数式和面向对象编程与 JavaScript 联系起来，看看这两种范式如何相辅相成，共存。本章将涵盖以下主题：

+   JavaScript 如何既是函数式的又是面向对象的？

+   JavaScript 的 OOP - 使用原型

+   如何在 JavaScript 中混合函数式和面向对象编程

+   函数继承

+   函数式混入

更好的代码是目标。函数式和面向对象编程只是实现这一目标的手段。

# JavaScript - 多范式语言

如果面向对象编程意味着将所有变量视为对象，而函数式编程意味着将所有函数视为变量，那么函数不能被视为对象吗？在 JavaScript 中，它们可以。

但说函数式编程意味着将函数视为变量有些不准确。更好的说法是：函数式编程意味着将一切都视为值，尤其是函数。

描述函数式编程的更好方式可能是将其称为声明式。与命令式编程风格无关，*声明式编程*表达了解决问题所需的计算逻辑。计算机被告知问题是什么，而不是如何解决它的过程。

与此同时，面向对象编程源自命令式编程风格：计算机会得到解决问题的逐步说明。OOP 要求计算的说明（方法）和它们操作的数据（成员变量）被组织成称为对象的单元。访问数据的唯一方式是通过对象的方法。

那么这两种风格如何集成在一起呢？

+   对象方法中的代码通常以命令式风格编写。但如果以函数式风格呢？毕竟，OOP 并不排斥不可变数据和高阶函数。

+   也许更纯粹的混合方式是同时将对象视为函数和传统的基于类的对象。

+   也许我们可以简单地在面向对象的应用程序中包含一些函数式编程的思想，比如承诺和递归。

+   OOP 涵盖了封装、多态和抽象等主题。函数式编程也涵盖了这些主题，只是它采用了不同的方式。也许我们可以在面向函数的应用程序中包含一些面向对象编程的思想。

重点是：OOP 和 FP 可以混合在一起，有几种方法可以做到这一点。它们并不互斥。

# JavaScript 的面向对象实现 - 使用原型

JavaScript 是一种无类语言。这并不意味着它比其他计算机语言更时尚或更蓝领；无类意味着它没有类结构，就像面向对象的语言那样。相反，它使用原型进行继承。

尽管这可能让有 C++和 Java 背景的程序员感到困惑，基于原型的继承比传统继承更具表现力。以下是 C++和 JavaScript 之间差异的简要比较：

| C++ | JavaScript |
| --- | --- |
| 强类型 | 弱类型 |
| 静态 | 动态 |
| 基于类 | 基于原型 |
| 类 | 函数 |
| 构造函数 | 函数 |
| 方法 | 函数 |

## 继承

在我们进一步讨论之前，让我们确保我们充分理解面向对象编程中的继承概念。基于类的继承在以下伪代码中得到了展示：

```js
class Polygon {
  int numSides;
  function init(n) {
    numSides = n;
  }
}
class Rectangle inherits Polygon {
  int width;
  int length;
  function init(w, l) {
    numSides = 4;
    width = w;
    length = l;
  }
  function getArea() {
    return w * l;
  }
}
class Square inherits Rectangle {
  function init(s) {
    numSides = 4;
    width = s;
    length = s;
  }
}
```

`Polygon`类是其他类继承的父类。它只定义了一个成员变量，即边数，该变量在`init()`函数中设置。`Rectangle`子类继承自`Polygon`类，并添加了两个成员变量`length`和`width`，以及一个方法`getArea()`。它不需要定义`numSides`变量，因为它已经被继承的类定义了，并且它还覆盖了`init()`函数。`Square`类通过从`Rectangle`类继承其`getArea()`方法进一步延续了这种继承链。通过简单地再次覆盖`init()`函数，使长度和宽度相同，`getArea()`函数可以保持不变，从而需要编写的代码更少。

在传统的面向对象编程语言中，这就是继承的全部含义。如果我们想要向所有对象添加一个颜色属性，我们只需将其添加到`Polygon`对象中，而无需修改任何继承自它的对象。

## JavaScript 的原型链

JavaScript 中的继承归结为原型。每个对象都有一个名为其原型的内部属性，它是指向另一个对象的链接。该对象本身也有自己的原型。这种模式可以重复，直到达到一个具有`undefined`作为其原型的对象。这就是原型链，这就是 JavaScript 中继承的工作原理。以下图解释了 JavaScript 中的继承：

![JavaScript 的原型链](img/00011.jpeg)

在搜索对象的函数定义时，JavaScript 会“遍历”原型链，直到找到具有正确名称的函数的第一个定义。因此，覆盖它就像在子类的原型上提供一个新定义一样简单。

## JavaScript 中的继承和`Object.create()`方法

就像有许多方法可以在 JavaScript 中创建对象一样，也有许多方法可以复制基于类的经典继承。但做这件事的首选方法是使用`Object.create()`方法。

```js
var Polygon = function(n) {
  this.numSides = n;
}

var Rectangle = function(w, l) {
  this.width = w;
  this.length = l;
}

// the Rectangle's prototype is redefined with Object.create
Rectangle.prototype = Object.create(Polygon.prototype);

// it's important to now restore the constructor attribute
// otherwise it stays linked to the Polygon
Rectangle.prototype.constructor = Rectangle;

// now we can continue to define the Rectangle class
Rectangle.prototype.numSides = 4;
Rectangle.prototype.getArea = function() {
  return this.width * this.length;
}

var Square = function(w) {
  this.width = w;
  this.length = w;
}
Square.prototype = Object.create(Rectangle.prototype);
Square.prototype.constructor = Square;

var s = new Square(5);
console.log( s.getArea() ); // 25
```

这种语法对许多人来说可能看起来不寻常，但经过一点练习，它将变得熟悉。必须使用`prototype`关键字来访问所有对象都具有的内部属性`[[Prototype]]`。`Object.create()`方法声明一个新对象，该对象继承自指定的对象原型。通过这种方式，可以在 JavaScript 中实现经典继承。

### 注意

`Object.create()`方法在 2011 年的 ECMAScript 5.1 中引入，并被宣传为创建对象的新方法。这只是 JavaScript 整合继承的众多尝试之一。幸运的是，这种方法运行得相当好。

在构建第五章*范畴论*中的`Maybe`类时，我们看到了这种继承结构，`Maybe`、`None`和`Just`类，它们彼此之间也是继承关系。

```js
var Maybe = function(){}; 

var None = function(){}; 
None.prototype = Object.create(Maybe.prototype);
None.prototype.constructor = None;
None.prototype.toString = function(){return 'None';};

var Just = function(x){this.x = x;};
Just.prototype = Object.create(Maybe.prototype);
Just.prototype.constructor = Just;
Just.prototype.toString = function(){return "Just "+this.x;};
```

这表明 JavaScript 中的类继承可以成为函数式编程的一种实现方式。

一个常见的错误是将构造函数传递给`Object.create()`而不是`prototype`对象。这个问题的复杂性在于，直到子类尝试使用继承的成员函数时才会抛出错误。

```js
Foo.prototype = Object.create(Parent.prototype); // correct
Bar.prototype = Object.create(Parent); // incorrect
Bar.inheritedMethod(); // Error: function is undefined
```

如果`inheritedMethod()`方法已经附加到`Foo.prototype`类，则无法找到该函数。如果`inheritedMethod()`方法直接附加到实例上，即在`Bar`构造函数中使用`this.inheritedMethod = function(){...}`，那么`Object.create()`中使用`Parent`作为参数可能是正确的。

# 在 JavaScript 中混合函数式和面向对象编程

面向对象编程已经是主导的编程范式数十年了。它在世界各地的计算机科学 101 课程中被教授，而函数式编程则没有。软件架构师用它来设计应用程序，而函数式编程则没有。这也是有道理的：面向对象编程使得抽象思想更容易理解。它使编写代码更容易。

所以，除非你能说服你的老板应用程序需要全部是函数式的，否则我们将在面向对象的世界中使用函数式编程。本节将探讨如何做到这一点。

## 函数式继承

将函数式编程应用于 JavaScript 应用程序的最直接方式可能是在面向对象编程原则内使用大部分函数式风格，比如继承。

为了探索这可能如何工作，让我们构建一个简单的应用程序来计算产品的价格。首先，我们需要一些产品类：

```js
var Shirt = function(size) {
  this.size = size;
};

var TShirt = function(size) {
  this.size = size;
};
TShirt.prototype = Object.create(Shirt.prototype);
TShirt.prototype.constructor = TShirt;
TShirt.prototype.getPrice = function(){
  if (this.size == 'small') {
    return 5;
  }
  else {
    return 10;
  }
}

var ExpensiveShirt = function(size) {
  this.size = size;
}
ExpensiveShirt.prototype = Object.create(Shirt.prototype);
ExpensiveShirt.prototype.constructor = ExpensiveShirt;
ExpensiveShirt.prototype.getPrice = function() {
  if (this.size == 'small') {
    return 20;
  }
  else {
    return 30;
  }
}
```

然后我们可以在`Store`类中组织它们如下：

```js
var Store = function(products) {
  this.products = products;
}
Store.prototype.calculateTotal = function(){
  return this.products.reduce(function(sum,product) {
    return sum + product.getPrice();
  }, 10) * TAX; // start with $10 markup, times global TAX var
};

var TAX = 1.08;
var p1 = new TShirt('small');
var p2 = new ExpensiveShirt('large');
var s = new Store([p1,p2]);
console.log(s.calculateTotal()); // Output: 35
```

`calculateTotal()`方法使用数组的`reduce()`函数来干净地将产品的价格相加。

这样做完全没问题，但如果我们需要一种动态计算标记值的方法呢？为此，我们可以转向一个称为**策略模式**的概念。

### 策略模式

策略模式是一种定义一组可互换算法的方法。它被面向对象编程程序员用于在运行时操纵行为，但它基于一些函数式编程原则。

+   逻辑和数据的分离

+   函数的组合

+   函数作为一等对象

还有一些面向对象编程的原则：

+   封装

+   继承

在我们之前解释的用于计算产品成本的示例应用中，假设我们想要给予某些客户优惠待遇，并且需要调整标记来反映这一点。

所以让我们创建一些客户类：

```js
var Customer = function(){};
Customer.prototype.calculateTotal = function(products) {
  return products.reduce(function(total, product) {
    return total + product.getPrice();
  }, 10) * TAX;
};

var RepeatCustomer = function(){};
RepeatCustomer.prototype = Object.create(Customer.prototype);
RepeatCustomer.prototype.constructor = RepeatCustomer;
RepeatCustomer.prototype.calculateTotal = function(products) {
  return products.reduce(function(total, product) {
    return total + product.getPrice();
  }, 5) * TAX;
};

var TaxExemptCustomer = function(){};
TaxExemptCustomer.prototype = Object.create(Customer.prototype);
TaxExemptCustomer.prototype.constructor = TaxExemptCustomer;
TaxExemptCustomer.prototype.calculateTotal = function(products) {
  return products.reduce(function(total, product) {
    return total + product.getPrice();
  }, 10);
};
```

每个`Customer`类封装了算法。现在我们只需要`Store`类调用`Customer`类的`calculateTotal()`方法。

```js
var Store = function(products) {
  this.products = products;
  this.customer = new Customer();
  // bonus exercise: use Maybes from Chapter 5 instead of a default customer instance
}
Store.prototype.setCustomer = function(customer) {
  this.customer = customer;
}
Store.prototype.getTotal = function(){
  return this.customer.calculateTotal(this.products);
};

var p1 = new TShirt('small');
var p2 = new ExpensiveShirt('large');
var s = new Store([p1,p2]);
var c = new TaxExemptCustomer();
s.setCustomer(c);
s.getTotal(); // Output: 45
```

`Customer`类进行计算，`Product`类保存数据（价格），`Store`类维护上下文。这实现了非常高的内聚性和面向对象编程与函数式编程的很好的混合。JavaScript 的高表现力使这成为可能，而且相当容易。

## Mixins

简而言之，mixins 是允许其他类使用它们的方法的类。这些方法仅供其他类使用，而`mixin`类本身永远不会被实例化。这有助于避免继承的模糊性。它们是将函数式编程与面向对象编程混合的绝佳手段。

每种语言中的 mixin 实现方式都不同。由于 JavaScript 的灵活性和表现力，mixins 被实现为只有方法的对象。虽然它们可以被定义为函数对象（即`var mixin = function(){...};`），但最好将它们定义为对象字面量（即`var mixin = {...};`）以保持代码的结构纪律。这将帮助我们区分类和 mixins。毕竟，mixins 应该被视为过程，而不是对象。

让我们从声明一些 mixins 开始。我们将扩展我们之前部分的`Store`应用程序，使用 mixins 来扩展类。

```js
var small = {
  getPrice: function() {
    return this.basePrice + 6;   
  },
  getDimensions: function() {
    return [44,63]
  }
}
var large = {
  getPrice: function() {
    return this.basePrice + 10;   
  },
  getDimensions: function() {
    return [64,83]
  }
};
```

我们不仅仅局限于这些。还可以添加许多其他的 mixins，比如颜色或面料材质。我们需要稍微修改我们的`Shirt`类，如下面的代码片段所示：

```js
var Shirt = function() {
  this.basePrice = 1;
};
Shirt.getPrice = function(){
  return this.basePrice;
}
var TShirt = function() {
  this.basePrice = 5;
};
TShirt.prototype = Object.create(Shirt.prototype);
TShirt..prototype.constructor = TShirt;
```

现在我们准备使用 mixins 了。

### 经典 mixin

你可能想知道这些 mixin 是如何与类混合在一起的。这样做的经典方式是将 mixin 的函数复制到接收对象中。可以通过以下方式扩展`Shirt`原型来实现：

```js
Shirt.prototype.addMixin = function (mixin) {
  for (var prop in mixin) {
    if (mixin.hasOwnProperty(prop)) {
      this.prototype[prop] = mixin[prop];
    }
  }
};
```

现在可以添加 mixins 如下：

```js
TShirt.addMixin(small);
var p1 = new TShirt();
console.log( p1.getPrice() ); // Output: 11

TShirt.addMixin(large);
var p2 = new TShirt();
console.log( p2.getPrice() ); // Output: 15
```

然而，存在一个主要问题。当再次计算`p1`的价格时，结果是`15`，即大件物品的价格。它应该是小件物品的价格！

```js
console.log( p1.getPrice() ); // Output: 15
```

问题在于`Shirt`对象的`prototype.getPrice()`方法每次添加混入时都会被重写；这根本不是我们想要的函数式编程。

### 函数式混入

还有另一种使用混入的方法，更符合函数式编程。

我们不是将混入的方法复制到目标对象，而是需要创建一个新对象，该对象是目标对象的克隆，并添加了混入的方法。首先必须克隆对象，这可以通过创建一个继承自它的新对象来实现。我们将这个变体称为`plusMixin`。

```js
Shirt.prototype.plusMixin = function(mixin) {    
  // create a new object that inherits from the old
  var newObj = this;
  newObj.prototype = Object.create(this.prototype);
  for (var prop in mixin) {
    if (mixin.hasOwnProperty(prop)) {
      newObj.prototype[prop] = mixin[prop];
    }
  }
  return newObj;
};

var SmallTShirt = Tshirt.plusMixin(small); // creates a new class
var smallT = new SmallTShirt();
console.log( smallT.getPrice() );  // Output: 11

var LargeTShirt = Tshirt.plusMixin(large);
var largeT = new LargeTShirt();
console.log( largeT.getPrice() ); // Output: 15
console.log( smallT.getPrice() ); // Output: 11 (not effected by 2nd mixin call)
```

现在就来玩乐趣部分！现在我们可以真正地使用混入进行函数式编程。我们可以创建产品和混入的每种可能的组合。

```js
// in the real world there would be way more products and mixins!
var productClasses = [ExpensiveShirt, Tshirt]; 
var mixins = [small, medium, large];

// mix them all together 
products = productClasses.reduce(function(previous, current) {
  var newProduct = mixins.map(function(mxn) {
    var mixedClass = current.plusMixin(mxn);
    var temp = new mixedClass();
    return temp;
  });
  return previous.concat(newProduct);
},[]);
products.forEach(function(o){console.log(o.getPrice())});
```

为了使其更加面向对象，我们可以重写`Store`对象以具有此功能。我们还将在`Store`对象而不是产品中添加一个显示函数，以保持接口逻辑和数据的分离。

```js
// the store
var Store = function() {
  productClasses = [ExpensiveShirt, TShirt];
  productMixins = [small, medium, large];
  this.products = productClasses.reduce(function(previous, current) {
    var newObjs = productMixins.map(function(mxn) {
      var mixedClass = current.plusMixin(mxn);
      var temp = new mixedClass();
      return temp;
    });
    return previous.concat(newObjs);
  },[]);
}
Store.prototype.displayProducts = function(){
  this.products.forEach(function(p) {
    $('ul#products').append('<li>'+p.getTitle()+': $'+p.getPrice()+'</li>');
  });
}
```

我们所要做的就是创建一个`Store`对象并调用它的`displayProducts()`方法来生成产品和价格的列表！

```js
<ul id="products">
  <li>small premium shirt: $16</li>
  <li>medium premium shirt: $18</li>
  <li>large premium shirt: $20</li>
  <li>small t-shirt: $11</li>
  <li>medium t-shirt: $13</li>
  <li>large t-shirt: $15</li>
</ul>
```

这些行需要添加到`product`类和混入中，以使前面的输出正常工作：

```js
Shirt.prototype.title = 'shirt';
TShirt.prototype.title = 't-shirt';
ExpensiveShirt.prototype.title = 'premium shirt';

// then the mixins got the extra 'getTitle' function:
var small = {
  ...
  getTitle: function() {
    return 'small ' + this.title; // small or medium or large
  }
}
```

就这样，我们拥有了一个高度模块化和可扩展的电子商务应用。新的衬衫款式可以非常容易地添加——只需定义一个新的`Shirt`子类，并将其添加到`Store`类的数组`product`类中。混入的添加方式也是一样的。所以现在当我们的老板说：“嘿，我们有一种新类型的衬衫和外套，每种都有标准颜色，我们需要在你今天下班前将它们添加到网站上”，我们可以放心地说我们不会加班了！

# 总结

JavaScript 具有很高的表现力。这使得将函数式编程和面向对象编程混合使用成为可能。现代 JavaScript 不仅仅是面向对象编程或函数式编程，它是两者的混合体。诸如策略模式和混入之类的概念非常适合 JavaScript 的原型结构，并且它们有助于证明当今 JavaScript 最佳实践中函数式编程和面向对象编程的使用量是相等的。

如果你从这本书中只学到了一件事，我希望是如何将函数式编程技术应用到现实应用中。本章向你展示了如何做到这一点。

# 附录 A. JavaScript 中函数式编程的常见函数

这个附录涵盖了 JavaScript 中函数式编程的常见函数：

+   数组函数：

```js
var flatten = function(arrays) {
  return arrays.reduce( function(p,n){
    return p.concat(n);
  });
};

var invert = function(arr) {
  return arr.map(function(x, i, a) {
    return a[a.length - (i+1)];
  });
};
```

+   绑定函数：

```js
var bind = Function.prototype.call.bind(Function.prototype.bind);
var call = bind(Function.prototype.call, Function.prototype.call);
var apply = bind(Function.prototype.call, Function.prototype.apply);
```

+   范畴论：

```js
var checkTypes = function( typeSafeties ) {
  arrayOf(func)(arr(typeSafeties));
  var argLength = typeSafeties.length;
  return function(args) {
    arr(args);
    if (args.length != argLength) {
      throw new TypeError('Expected '+ argLength + ' arguments');
    }
    var results = [];
    for (var i=0; i<argLength; i++) {
      results[i] = typeSafetiesi;
    }
    return results;
  };
};

var homoMorph = function( /* arg1, arg2, ..., argN, output */ ) {
  var before = checkTypes(arrayOf(func)(Array.prototype.slice.call(arguments, 0, arguments.length-1)));
  var after = func(arguments[arguments.length-1])
  return function(middle) {
    return function(args) {
      return after(middle.apply(this, before([].slice.apply(arguments))));
    };
  };
};
```

+   组合：

```js
Function.prototype.compose = function(prevFunc) {
  var nextFunc = this;
  return function() {
    return nextFunc.call(this,prevFunc.apply(this,arguments));
  };
};

Function.prototype.sequence  = function(prevFunc) {
  var nextFunc = this;
  return function() {
    return prevFunc.call(this,nextFunc.apply(this,arguments));
  };
};
```

+   柯里化：

```js
Function.prototype.curry = function (numArgs) {
  var func = this;
  numArgs = numArgs || func.length;
  // recursively acquire the arguments
  function subCurry(prev) {
    return function (arg) {
      var args = prev.concat(arg);
      if (args.length < numArgs) {
        // recursive case: we still need more args
        return subCurry(args);
      }
      else {
        // base case: apply the function
        return func.apply(this, args);
      }
    };
  };
  return subCurry([]);
};
```

+   函子：

```js
// map :: (a -> b) -> [a] -> [b]
var map = function(f, a) {
  return arr(a).map(func(f));
}

// strmap :: (str -> str) -> str -> str
var strmap = function(f, s) {
  return str(s).split('').map(func(f)).join('');
}

// fcompose :: (a -> b)* -> (a -> b)
var fcompose = function() {
  var funcs = arrayOf(func)(arguments);
  return function() {
    var argsOfFuncs = arguments;
    for (var i = funcs.length; i > 0; i -= 1) {
      argsOfFuncs  = [funcs[i].apply(this, args)];
    }
    return args[0];
  };
};
```

+   镜头：

```js
var lens = function(get, set) {
  var f = function (a) {return get(a)};
  f.get = function (a) {return get(a)}; 
  f.set = set;
  f.mod = function (f, a) {return set(a, f(get(a)))};
  return f;
};

// usage:
var first = lens(
  function (a) { return arr(a)[0]; }, // get
  function (a, b) { return [b].concat(arr(a).slice(1)); } // set
);
```

+   Maybes：

```js
var Maybe = function(){}; 
Maybe.prototype.orElse = function(y) {
  if (this instanceof Just) {
    return this.x;
  }
  else {
    return y;
  }
};

var None = function(){}; 
None.prototype = Object.create(Maybe.prototype);
None.prototype.toString = function(){return 'None';};
var none = function(){return new None()};
// and the Just instance, a wrapper for an object with a value
var Just = function(x){return this.x = x;};
Just.prototype = Object.create(Maybe.prototype);
Just.prototype.toString = function(){return "Just "+this.x;};
var just = function(x) {return new Just(x)};
var maybe = function(m){
  if (m instanceof None) {
    return m;
  }
  else if (m instanceof Just) {
    return just(m.x);
  }
  else {
    throw new TypeError("Error: Just or None expected, " + m.toString() + " given."); 
  }
};

var maybeOf = function(f){
  return function(m) {
    if (m instanceof None) {
      return m;
    }
    else if (m instanceof Just) {
      return just(f(m.x));
    }
    else {
      throw new TypeError("Error: Just or None expected, " + m.toString() + " given."); 
    }
  };
};
```

+   混入：

```js
Object.prototype.plusMixin = function(mixin) {
  var newObj = this;
  newObj.prototype = Object.create(this.prototype);
  newObj.prototype.constructor = newObj;
  for (var prop in mixin) {
    if (mixin.hasOwnProperty(prop)) {
      newObj.prototype[prop] = mixin[prop];
    }
  }
  return newObj;
};
```

+   部分应用：

```js
function bindFirstArg(func, a) {
  return function(b) {
    return func(a, b);
  };
};

Function.prototype.partialApply = function(){
  var func = this; 
  args = Array.prototype.slice.call(arguments);
  return function(){
    return func.apply(this, args.concat(
      Array.prototype.slice.call(arguments)
    ));
  };
};

Function.prototype.partialApplyRight = function(){
  var func = this; 
  args = Array.prototype.slice.call(arguments);
  return function(){
    return func.apply(
      this,
      Array.protype.slice.call(arguments, 0)
    .concat(args));
  };
};
```

+   Trampolining：

```js
var trampoline = function(f) {
  while (f && f instanceof Function) {
    f = f.apply(f.context, f.args);
  }
  return f;
};

var thunk = function (fn) {
  return function() {
    var args = Array.prototype.slice.apply(arguments);
    return function() { return fn.apply(this, args); };
  };
};
```

+   类型安全：

```js
var typeOf = function(type) {
  return function(x) {
    if (typeof x === type) {
      return x;
    }
    else {
      throw new TypeError("Error: "+type+" expected, "+typeof x+" given.");
    }
  };
};

var str = typeOf('string'),
  num = typeOf('number'),
  func = typeOf('function'),
  bool = typeOf('boolean');

var objectTypeOf = function(name) {
  return function(o) {
    if (Object.prototype.toString.call(o) === "[object "+name+"]") {
      return o;
    }
    else {
      throw new TypeError("Error: '+name+' expected, something else given."); 
    }
  };
};
var obj = objectTypeOf('Object');
var arr = objectTypeOf('Array');
var date = objectTypeOf('Date');
var div = objectTypeOf('HTMLDivElement');

// arrayOf :: (a -> b) -> ([a] -> [b])
var arrayOf = function(f) {
  return function(a) {
    return map(func(f), arr(a));
  }
};
```

+   Y 组合子：

```js
var Y = function(F) {
  return (function (f) {
    return f(f);
  }(function (f) {
    return F(function (x) {
      return f(f)(x);
    });
  }));
};

// Memoizing Y-Combinator:
var Ymem = function(F, cache) {
  if (!cache) {
    cache = {} ; // Create a new cache.
  }
  return function(arg) {
    if (cache[arg]) {
      // Answer in cache
      return cache[arg] ;
    }
    // else compute the answer
    var answer = (F(function(n){
      return (Ymem(F,cache))(n);
    }))(arg); // Compute the answer.
    cache[arg] = answer; // Cache the answer.
    return answer;
  };
};
```

# 附录 B. 术语表

这个附录涵盖了本书中使用的一些重要术语：

+   匿名函数：没有名称且未绑定到任何变量的函数。也称为 Lambda 表达式。

+   回调：可以传递给另一个函数以在以后的事件中使用的函数。

+   范畴：在范畴论中，范畴是相同类型的对象集合。在 JavaScript 中，范畴可以是包含明确定义为数字、字符串、布尔值、日期、对象等的对象的数组或对象。

+   范畴论：一种将数学结构组织成对象集合和对这些对象的操作的概念。本书中使用的计算机程序中的数据类型和函数形成了这些范畴。

+   闭包：一种环境，使得其中定义的函数可以访问外部不可用的局部变量。

+   耦合：每个程序模块依赖于其他模块的程度。函数式编程减少了程序内部的耦合程度。

+   **Currying**：将具有多个参数的函数转换为一个参数的函数的过程，返回另一个可以根据需要接受更多参数的函数。形式上，具有*N*个参数的函数可以转换为*N*个函数的函数链，每个函数只有一个参数。

+   **Declarative programming**：一种表达解决问题所需的计算逻辑的编程风格。计算机被告知问题是什么，而不是解决问题所需的过程。

+   **Endofunctor**：将一个范畴映射到自身的函子。

+   **Function composition**：将许多函数组合成一个函数的过程。每个函数的结果作为下一个参数传递，最后一个函数的结果是整个组合的结果。

+   **Functional language**：促进函数式编程的计算机语言。

+   **Functional programming**：一种声明式编程范式，侧重于将函数视为数学表达式，并避免可变数据和状态变化。

+   **Functional reactive programming**：一种侧重于响应式元素和随时间变化的变量的函数式编程风格。

+   **Functor**：范畴之间的映射。

+   **Higher-order function**：以一个或多个函数作为输入，并返回一个函数作为输出的函数。

+   **Inheritance**：一种面向对象编程的能力，允许一个类从另一个类继承成员变量和方法。

+   **Lambda expressions**：参见匿名函数。

+   **Lazy evaluation**：一种计算机语言的评估策略，延迟对表达式的评估，直到需要其值。这种策略的相反称为急切评估或贪婪评估。惰性评估也被称为按需调用。

+   **Library**：一组具有明确定义接口的对象和函数，允许第三方程序调用它们的行为。

+   **Memoization**：存储昂贵函数调用的结果的技术。当以相同参数再次调用函数时，返回存储的结果，而不是重新计算结果。

+   **Method chain**：一种模式，其中许多方法并排调用，直接将一个方法的输出传递给下一个方法的输入。这避免了将中间值分配给临时变量的需要。

+   **Mixin**：一个对象，可以让其他对象使用它的方法。这些方法只能被其他对象使用，而 mixin 对象本身永远不会被实例化。

+   **Modularity**：程序可以被分解为独立的代码模块的程度。函数式编程增加了程序的模块化。

+   **Monad**：提供函子所需的封装的结构。

+   **Morphism**：仅在特定范畴上工作并在给定特定输入集时始终返回相同输出的纯函数。同态操作受限于单一范畴，而多态操作可以在多个范畴上操作。

+   **Partial application**：将一个或多个参数的值绑定到函数的过程。它返回一个部分应用的函数，该函数反过来接受剩余的未绑定参数。

+   **Polyfill**：用于用新函数增强原型的函数。它允许我们将新函数作为先前函数的方法来调用。

+   **Pure function**：其输出值仅取决于作为函数输入的参数的函数。因此，用相同值的参数*x*两次调用函数*f*，每次都会产生相同的结果*f(x)*。

+   **递归函数**：调用自身的函数。这样的函数依赖于解决同一问题的较小实例来计算较大问题的解决方案。与迭代类似，递归是重复调用相同代码块的另一种方式。但是，与迭代不同，递归要求代码块定义重复调用应该终止的情况，即基本情况。

+   **可重用性**：通常是指代码块（通常是 JavaScript 中的函数）可以在同一程序的其他部分或其他程序中被重复使用的程度。

+   **自执行函数**：在定义后立即被调用的匿名函数。在 JavaScript 中，通过在函数表达式后放置一对括号来实现这一点。

+   **策略模式**：用于定义一组可互换算法的方法。

+   **尾递归**：基于堆栈的递归实现。对于每个递归调用，堆栈中都有一个新的帧。

+   **工具包**：一个小型软件库，提供一组函数供程序员使用。与库相比，工具包更简单，需要与调用它的程序耦合更少。

+   **蹦床编程**：一种递归策略，可以在不提供尾调用消除功能的编程语言中实现，比如 JavaScript。

+   **Y 组合子**：Lambda 演算中的固定点组合子，消除了显式递归。当它作为返回递归函数的输入时，Y 组合子返回该函数的不动点，即将递归函数转换为非递归函数的转换。
