# 第一章。JavaScript 函数式编程的力量-演示

# 介绍

几十年来，函数式编程一直是计算机科学爱好者的宠儿，因其数学纯粹性和令人费解的特性而备受推崇，这使它隐藏在数据科学家和博士候选人占据的尘封的计算机实验室中。但现在，它正在复兴，这要归功于现代语言，如**Python**、**Julia**、**Ruby**、**Clojure**和——最后但并非最不重要的——**JavaScipt**。

你说 JavaScript？网络的脚本语言？是的！

JavaScript 已被证明是一种重要的技术，它不会很快消失。这在很大程度上是因为它能够通过新的框架和库（如**backbone.js**、**jQuery**、**Dojo**、**underscore.js**等）得到重生和扩展。*这直接关系到 JavaScript 作为一种函数式编程语言的真正身份*。对 JavaScript 的函数式编程的理解将长期受到欢迎，并对任何技能水平的程序员都将是有用的。

为什么这样？函数式编程非常强大、健壮和优雅。它在大型数据结构上非常有用和高效。将 JavaScript——一种客户端脚本语言，作为一种函数式手段来操作 DOM、对 API 响应进行排序或在日益复杂的网站上执行其他任务，可能非常有利。

在本书中，您将学习有关 JavaScript 函数式编程的一切：如何通过函数式编程增强 JavaScript 网络应用程序，如何解锁 JavaScript 的潜在力，以及如何编写更强大、更易于维护、下载速度更快、开销更小的代码。您还将学习函数式编程的核心概念，如何将其应用于 JavaScript，如何避开在使用 JavaScript 作为函数式语言时可能出现的注意事项和问题，以及如何在 JavaScript 中将函数式编程与面向对象编程相结合。

但在我们开始之前，让我们进行一个实验。

# 演示

也许一个快速的演示将是介绍 JavaScript 函数式编程的最佳方式。我们将使用 JavaScript 执行相同的任务——一次使用传统的本地方法，一次使用函数式编程。然后，我们将比较这两种方法。

# 应用程序-电子商务网站

在追求真实世界应用的过程中，假设我们需要为一家邮购咖啡豆公司开发一个电子商务网站应用程序。他们销售几种不同类型和不同数量的咖啡，这两者都会影响价格。

## 命令式方法

首先，让我们按照程序化的路线进行。为了使这个演示接地气，我们将创建保存数据的对象。这允许从数据库中获取值的能力，如果需要的话。但现在，我们假设它们是静态定义的：

```js
// create some objects to store the data.
var columbian = {
  name: 'columbian',
  basePrice: 5
};
var frenchRoast = {
  name: 'french roast',
  basePrice: 8
};
var decaf = {
  name: 'decaf',
  basePrice: 6
};

// we'll use a helper function to calculate the cost 
// according to the size and print it to an HTML list
function printPrice(coffee, size) {
  if (size == 'small') {
    var price = coffee.basePrice + 2;
  }
  else if (size == 'medium') {
    var price = coffee.basePrice + 4;
  }
  else {
    var price = coffee.basePrice + 6;
  }

// create the new html list item
  var node = document.createElement("li");
  var label = coffee.name + ' ' + size;
  var textnode = document.createTextNode(label+' price: $'+price);
  node.appendChild(textnode);
  document.getElementById('products').appendChild(node);
}

// now all we need to do is call the printPrice function
// for every single combination of coffee type and size
printPrice(columbian, 'small');
printPrice(columbian, 'medium');
printPrice(columbian, 'large');
printPrice(frenchRoast, 'small');
printPrice(frenchRoast, 'medium');
printPrice(frenchRoast, 'large');
printPrice(decaf, 'small');
printPrice(decaf, 'medium');
printPrice(decaf, 'large');
```

### 提示

**下载示例代码**

您可以从[`www.packtpub.com`](http://www.packtpub.com)的帐户中下载您购买的所有 Packt 图书的示例代码文件。如果您在其他地方购买了本书，您可以访问[`www.packtpub.com/support`](http://www.packtpub.com/support)并注册，以便直接通过电子邮件接收文件。

正如你所看到的，这段代码非常基础。如果这里不只是三种咖啡风格呢？如果有 20 种？50 种？如果除了大小之外，还有有机和非有机的选择。那将极大地增加代码行数！

使用这种方法，我们告诉机器为每种咖啡类型和每种大小打印什么。这基本上就是命令式代码的问题所在。

# 函数式编程

命令式代码告诉机器，逐步地，它需要做什么来解决问题，而函数式编程则试图以数学方式描述问题，以便机器可以做其余的工作。

采用更加函数式的方法，同样的应用可以写成如下形式：

```js
// separate the data and logic from the interface
var printPrice = function(price, label) {
  var node = document.createElement("li");
  var textnode = document.createTextNode(label+' price: $'+price);
  node.appendChild(textnode);
  document.getElementById('products 2').appendChild(node);
}

// create function objects for each type of coffee
var columbian = function(){
  this.name = 'columbian'; 
  this.basePrice = 5;
};
var frenchRoast = function(){
  this.name = 'french roast'; 
  this.basePrice = 8;
};
var decaf = function(){
  this.name = 'decaf'; 
  this.basePrice = 6;
};

// create object literals for the different sizes
var small = {
  getPrice: function(){return this.basePrice + 2},
  getLabel: function(){return this.name + ' small'}
};
var medium = {
  getPrice: function(){return this.basePrice + 4},
  getLabel: function(){return this.name + ' medium'}
};
var large = {
  getPrice: function(){return this.basePrice + 6},
  getLabel: function(){return this.name + ' large'}
};

// put all the coffee types and sizes into arrays
var coffeeTypes = [columbian, frenchRoast, decaf];
var coffeeSizes = [small, medium, large];

// build new objects that are combinations of the above
// and put them into a new array
var coffees = coffeeTypes.reduce(function(previous, current) {
  var newCoffee = coffeeSizes.map(function(mixin) {
    // `plusmix` function for functional mixins, see Ch.7
    var newCoffeeObj = plusMixin(current, mixin);
    return new newCoffeeObj();
  });
  return previous.concat(newCoffee);
},[]);

// we've now defined how to get the price and label for each
// coffee type and size combination, now we can just print them
coffees.forEach(function(coffee){
  printPrice(coffee.getPrice(),coffee.getLabel());
});
```

首先显而易见的是它更加模块化。这使得添加新的大小或新的咖啡类型就像下面的代码片段中所示的那样简单：

```js
var peruvian = function(){
  this.name = 'peruvian'; 
  this.basePrice = 11;
};

var extraLarge = {
  getPrice: function(){return this.basePrice + 10},
  getLabel: function(){return this.name + ' extra large'}
};

coffeeTypes.push(Peruvian);
coffeeSizes.push(extraLarge);
```

咖啡对象和大小对象的数组被“混合”在一起，也就是说，它们的方法和成员变量与一个名为`plusMixin`的自定义函数结合在一起（参见第七章，“JavaScript 中的函数式和面向对象编程”）。咖啡类型类包含成员变量，大小包含计算名称和价格的方法。 “混合”发生在`map`操作中，它对数组中的每个元素应用纯函数，并在`reduce()`操作中返回一个新函数——另一个类似于`map`函数的高阶函数，不同之处在于数组中的所有元素都合并成一个。最后，所有可能的类型和大小组合的新数组通过`forEach()`方法进行迭代。`forEach()`方法是另一个高阶函数，它对数组中的每个对象应用回调函数。在这个例子中，我们将其作为一个匿名函数提供，该函数实例化对象并调用`printPrice()`函数，其中包括对象的`getPrice()`和`getLabel()`方法作为参数。

实际上，我们可以通过移除`coffees`变量并将函数链接在一起使这个例子更加函数化——这是函数式编程中的另一个小技巧。

```js
coffeeTypes.reduce(function(previous, current) {
  var newCoffee = coffeeSizes.map(function(mixin) {
    // `plusMixin` function for functional mixins, see Ch.7
    var newCoffeeObj = plusMixin(current, mixin);
    return new newCoffeeObj();
  });
  return previous.concat(newCoffee);
},[]).forEach(function(coffee) {
  printPrice(coffee.getPrice(),coffee.getLabel());
});
```

此外，控制流不像命令式代码那样自上而下。在函数式编程中，`map()`函数和其他高阶函数取代了`for`和`while`循环，执行顺序的重要性很小。这使得新手更难阅读代码，但一旦掌握了，就会发现其实并不难跟踪，而且会发现它更好。

这个例子只是简单介绍了在 JavaScript 中函数式编程可以做什么。在本书中，你将看到更强大的函数式编程的例子。

# 总结

首先，采用函数式风格的好处是明显的。

其次，不要害怕函数式编程。是的，它经常被认为是以计算机语言形式的纯逻辑，但我们不需要理解**Lambda 演算**就能将其应用到日常任务中。事实上，通过允许我们的程序被分解成更小的部分，它们更容易理解，更简单维护，更可靠。`map()`和`reduce()`函数是 JavaScript 中较少为人知的内置函数，但我们会看一下它们。

JavaScript 是一种脚本语言，交互性强，易于接近。不需要编译。我们甚至不需要下载任何开发软件，你喜欢的浏览器可以作为解释器和开发环境。

感兴趣吗？好的，让我们开始吧！
