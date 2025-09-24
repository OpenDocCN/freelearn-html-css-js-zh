# 第六章 原型

在本章中，你将了解函数对象的`prototype`属性。理解原型的工作原理是学习 JavaScript 语言的重要部分。毕竟，JavaScript 通常被归类为基于原型的对象模型。原型本身并没有什么特别困难的地方，但它是一个新概念，因此有时可能需要一些时间才能理解。就像闭包（见第三章“函数”，*函数*）一样，原型是 JavaScript 中那些一旦理解，就会觉得非常明显且完全合理的事物之一。正如本书的其余部分所鼓励的那样，强烈建议你输入并尝试这些示例——这会使学习并记住概念变得容易得多。

在本章中，我们将涵盖以下主题：

+   每个函数都有一个`prototype`属性，它包含一个对象

+   向原型对象添加属性

+   使用添加到原型的属性

+   自有属性与原型属性的区别

+   `__proto__`属性，每个对象与其原型之间保持的秘密链接

+   方法如`isPrototypeOf()`、`hasOwnProperty()`和`propertyIsEnumerable()`

+   增强内置对象，如数组或字符串，以及为什么这可能是一个坏主意

# 原型属性

JavaScript 中的函数是对象，它们包含方法和属性。你已经熟悉的某些方法有`apply()`和`call()`，而其他一些属性有`length`和`constructor`。函数对象的另一个属性是`prototype`。

如果你定义一个简单的函数`foo()`，你可以像访问任何其他对象一样访问它的属性。考虑以下代码：

```js
    > function foo(a, b) { 
        return a * b; 
      } 
    > foo.length; 
    2 
    > foo.constructor; 
    function Function() { [native code] } 

```

`prototype`属性是在你定义函数后立即可用的属性。它的初始值是一个空对象：

```js
    > typeof foo.prototype; 
    "object" 

```

这就像你亲自添加了这个属性一样，如下所示：

```js
    > foo.prototype = {}; 

```

你可以用属性和方法来增强这个空对象。它们对`foo()`函数本身没有任何影响；只有当你将`foo()`作为构造函数调用时，它们才会被使用。

## 使用原型添加方法和属性

在上一章中，你学习了如何定义构造函数，你可以使用它来创建（构造）新对象。主要思想是，在用`new`调用的函数内部，你将能够访问`this`值，它指向构造函数将要返回的对象。增强，即向`this`添加方法和属性，是你向正在构造的对象添加功能的方式。

让我们看看构造函数`Gadget()`，它使用`this`向它创建的对象添加两个属性和一个方法，如下所示：

```js
    function Gadget(name, color) { 
      this.name = name; 
      this.color = color; 
      this.whatAreYou = function () { 
        return 'I am a ' + this.color + ' ' + this.name; 
      }; 
    } 

```

将方法属性添加到构造函数的`prototype`属性中是向该构造函数产生的对象添加功能的一种方法。让我们再添加两个属性，`price`和`rating`，以及一个`getInfo()`方法。由于`prototype`已经指向了一个对象，你只需继续向其中添加属性和方法，如下所示：

```js
    Gadget.prototype.price = 100; 
    Gadget.prototype.rating = 3; 
    Gadget.prototype.getInfo = function () { 
      return 'Rating: ' + this.rating + 
             ', price: ' + this.price; 
    }; 

```

或者，你不必逐个将属性添加到`prototype`对象中，你可以完全覆盖`prototype`，用你选择的对象替换它，如下面的示例所示：

```js
    Gadget.prototype = { 
      price: 100, 
      rating: ...  /* and so on... */ 
    }; 

```

# 使用原型的方法和属性

你添加到`prototype`中的所有方法和属性，一旦你使用构造函数创建了一个新对象，就都可以使用。如果你使用`Gadget()`构造函数创建了一个`newtoy`对象，你可以访问所有已经定义的方法和属性，如下面的代码所示：

```js
    > var newtoy = new Gadget('webcam', 'black'); 
    > newtoy.name; 
    "webcam" 
    > newtoy.color; 
    "black" 
    > newtoy.whatAreYou(); 
    "I am a black webcam" 
    > newtoy.price; 
    100 
    > newtoy.rating; 
    3 
    > newtoy.getInfo(); 
    "Rating: 3, price: 100" 

```

重要的是要注意，`prototype`是动态的。在 JavaScript 中，对象是通过引用传递的，因此，`prototype`不会随着每个新对象实例的创建而复制。这在实践中意味着什么？这意味着你可以在任何时候修改`prototype`，所有对象，包括修改之前创建的对象，都会看到这些变化。

让我们通过向`prototype`添加一个新方法来继续这个例子：

```js
    Gadget.prototype.get = function (what) { 
      return this[what]; 
    }; 

```

即使`newtoy`对象是在`get()`方法定义之前创建的，`newtoy`对象仍然可以访问这个新方法，如下所示：

```js
    > newtoy.get('price'); 
    100 
    > newtoy.get('color'); 
    "black" 

```

## 自有属性与原型属性

在前面的例子中，`getInfo()`被内部用来访问对象的属性。它也可以使用`Gadget.prototype`来达到相同的效果，如下所示：

```js
    Gadget.prototype.getInfo = function () { 
      return 'Rating: ' + Gadget.prototype.rating + 
             ', price: ' + Gadget.prototype.price; 
    }; 

```

有什么区别？为了回答这个问题，让我们详细考察一下`prototype`是如何工作的。

让我们再次以`newtoy`对象为例：

```js
    var newtoy = new Gadget('webcam', 'black'); 

```

当你尝试访问`newtoy`对象的属性，比如`newtoy.name`时，JavaScript 引擎会遍历对象的所有属性，寻找一个名为`name`的属性，如果找到了，就返回它的值，如下所示：

```js
    > newtoy.name; 
    "webcam" 

```

如果你尝试访问`rating`属性会怎样？JavaScript 引擎会检查`newtoy`对象的所有属性，但找不到名为`rating`的属性。然后，脚本引擎会识别用于创建此对象的构造函数的`prototype`（就像你执行`newtoy.constructor.prototype`一样）。如果属性在`prototype`对象中找到，则使用以下属性：

```js
    > newtoy.rating; 
    3 

```

你可以这样做，并直接访问`prototype`。每个对象都有一个`constructor`属性，它是对创建该对象的函数的引用，所以在这种情况下，查看以下代码：

```js
    > newtoy.constructor === Gadget; 
    true 
    > newtoy.constructor.prototype.rating; 
    3 

```

现在，让我们进一步探讨这个查找。每个对象都有一个构造函数。`prototype`是一个对象，因此它也必须有一个构造函数，这个构造函数反过来又有一个`prototype`。你可以沿着原型链向上，最终你会到达内置的`Object()`对象，这是最高级别的父对象。在实践中，这意味着如果你尝试`newtoy.toString()`，而`newtoy`没有自己的`toString()`方法，并且其`prototype`也没有，最终你会得到对象的`toString()`方法：

```js
    > newtoy.toString(); 
    "[object Object]" 

```

## 用自身属性覆盖原型的属性

如前所述的讨论所示，如果你的一个对象没有自己的某个属性，它可以使用原型链上存在的属性，如果存在的话。那么，如果对象有自己的属性，而原型也有一个同名的属性，会发生什么？在这种情况下，自身的属性会优先于原型的。

考虑一个场景，一个属性名同时作为对象的自身属性和`prototype`对象的属性存在：

```js
    > function Gadget(name) { 
        this.name = name; 
      } 
    > Gadget.prototype.name = 'mirror'; 

```

创建一个新的对象并访问其`name`属性将给你对象的自身`name`属性，如下所示：

```js
    > var toy = new Gadget('camera'); 
    > toy.name; 
    "camera" 

```

你可以使用`hasOwnProperty()`来确定属性的定义位置，如下所示：

```js
    > toy.hasOwnProperty('name'); 
    true 

```

如果你删除了`toy`对象的自身`name`属性，具有相同名称的原型属性就会显现出来：

```js
    > delete toy.name; 
    true 
    > toy.name; 
    "mirror" 
    > toy.hasOwnProperty('name'); 
    false 

```

当然，你可以始终按照以下方式重新创建对象的自身属性：

```js
    > toy.name = 'camera'; 
    > toy.name; 
    "camera" 

```

你可以通过使用`hasOwnProperty()`方法来探索你好奇的特定属性的来源。之前提到了`toString()`方法。它从哪里来？

```js
    > toy.toString(); 
    "[object Object]" 
    > toy.hasOwnProperty('toString'); 
    false 
    > toy.constructor.hasOwnProperty('toString'); 
    false 
    > toy.constructor.prototype.hasOwnProperty('toString'); 
    false 
    > Object.hasOwnProperty('toString'); 
    false 
    > Object.prototype.hasOwnProperty('toString'); 
    true 

```

### 枚举属性

如果你想要列出对象的所有属性，你可以使用`for...in`循环。在第二章中，*原始数据类型、数组、循环和条件*，你看到你也可以使用`for...in`循环遍历数组的所有元素，但如前所述，`for`更适合数组，而`for...in`更适合对象。让我们通过一个例子来构建一个从对象到 URL 查询字符串的查询字符串：

```js
    var params = { 
      productid: 666, 
      section: 'products' 
    }; 

    var url = 'http://example.org/page.php?', 
        i, 
        query = []; 

    for (i in params) { 
        query.push(i + '=' + params[i]); 
    } 

    url += query.join('&'); 

```

这会产生如下`url`字符串：

`http://example.org/page.php?productid=666&section=products`。

以下是一些需要注意的细节：

+   并非所有属性都会在`for...in`循环中显示。例如，长度（对于数组）和构造函数属性不会显示。显示的属性被称为可枚举属性。你可以使用每个对象提供的`propertyIsEnumerable()`方法来检查哪些属性是可枚举的。在 ES5 中，你可以指定哪些属性是可枚举的，而在 ES3 中则没有这种控制。

+   通过原型链传递的原型也显示出来，前提是它们是可枚举的。你可以使用`hasOwnProperty()`方法来检查一个属性是否是对象的自身属性还是原型的属性。

+   `propertyIsEnumerable()` 方法对所有原型的属性返回 `false`，即使它们是可枚举的，并在 `for...in` 循环中显示出来。

让我们看看这些方法的作用。以 `Gadget()` 的简化版本为例：

```js
    function Gadget(name, color) { 
      this.name = name; 
      this.color = color; 
      this.getName = function () { 
        return this.name; 
      }; 
    } 
    Gadget.prototype.price = 100; 
    Gadget.prototype.rating = 3; 

```

按如下方式创建一个新的对象：

```js
    var newtoy = new Gadget('webcam', 'black'); 

```

现在，如果你使用 `for...in` 循环进行循环，你可以看到对象的所有属性，包括来自原型的属性：

```js
    for (var prop in newtoy) {  
      console.log(prop + ' = ' + newtoy[prop]);  
    } 

```

结果还包含对象的方法，因为方法只是碰巧是函数的属性：

```js
    name = webcam 
    color = black 
    getName = function () { 
      return this.name; 
    } 
    price = 100 
    rating = 3 

```

如果你想要区分对象的自身属性和原型的属性，请使用 `hasOwnProperty()`。首先尝试以下操作：

```js
    > newtoy.hasOwnProperty('name'); 
    true 
    > newtoy.hasOwnProperty('price'); 
    false 

```

让我们再次循环，但这次只显示对象的自身属性：

```js
    for (var prop in newtoy) {  
      if (newtoy.hasOwnProperty(prop)) { 
        console.log(prop + '=' + newtoy[prop]);  
      } 
    } 

```

结果如下：

```js
    name=webcam 
    color=black 
    getName = function () { 
      return this.name; 
    } 

```

现在，让我们尝试 `propertyIsEnumerable()`。此方法对于对象自身的非内置属性返回 `true`，例如：

```js
    > newtoy.propertyIsEnumerable('name'); 
    true 

```

大多数内置属性和方法都不是可枚举的：

```js
    > newtoy.propertyIsEnumerable('constructor'); 
    false 

```

从原型链下来的任何属性都不是可枚举的：

```js
    > newtoy.propertyIsEnumerable('price'); 
    false 

```

然而，如果你到达包含在 `prototype` 中的对象并调用其 `propertyIsEnumerable()` 方法，那么这些属性将是可枚举的。考虑以下代码：

```js
    > newtoy.constructor.prototype.propertyIsEnumerable('price'); 
    true 

```

## 使用 isPrototypeOf() 方法

对象也有 `isPrototypeOf()` 方法。此方法告诉您该特定对象是否用作另一个对象的原型。

让我们以一个简单的名为 `monkey` 的对象为例：

```js
    var monkey = {    
      hair: true,    
      feeds: 'bananas',    
      breathes: 'air'  
    }; 

```

现在，让我们创建一个 `Human()` 构造函数，并将其 `prototype` 属性设置为指向 `monkey`：

```js
    function Human(name) { 
      this.name = name; 
    } 
    Human.prototype = monkey; 

```

现在，如果你创建一个新的 `Human` 对象，名为 `george`，并询问 `monkey` 是否是 `george` 的原型，你会得到 `true`：

```js
    > var george = new Human('George');  
    > monkey.isPrototypeOf(george); 
    true 

```

注意，你必须知道或怀疑原型是谁，然后询问它是否真的是 `monkey` 的原型，以确认你的怀疑。但是，如果你没有任何怀疑，你一无所知，你能否仅仅要求对象告诉你它的原型？答案是，在所有浏览器中你都不能这样做，但在大多数浏览器中你可以这样做。大多数最新的浏览器已经实现了 ES5 的补充，称为 `Object.getPrototypeOf()`。

```js
    > Object.getPrototypeOf(george).feeds; 
    "bananas" 
    > Object.getPrototypeOf(george) === monkey; 
    true 

```

对于一些没有 `getPrototypeOf()` 的预 ES5 环境，你可以使用特殊属性 `__proto__`。

## 秘密的 __proto__ 链接

如您所知，当你尝试访问当前对象中不存在的属性时，会查询 `prototype` 属性。

考虑另一个名为 `monkey` 的对象，并在使用 `Human()` 构造函数创建对象时将其用作原型：

```js
    > var monkey = { 
        feeds: 'bananas', 
        breathes: 'air' 
      }; 
    > function Human() {}  
    > Human.prototype = monkey; 

```

现在，让我们创建一个 `developer` 对象，并给它以下属性：

```js
    > var developer = new Human(); 
    > developer.feeds = 'pizza'; 
    > developer.hacks = 'JavaScript'; 

```

现在，让我们访问这些属性（例如，`hacks` 是 `developer` 对象的属性）：

```js
    > developer.hacks; 
    "JavaScript" 

```

`feeds` 属性也可以在对象中找到，如下所示：

```js
    > developer.feeds; 
    "pizza" 

```

`breathes` 属性不是 `developer` 对象的属性，因此会查找原型，就像有一个秘密的链接或通道通向 `prototype` 对象：

```js
    > developer.breathes; 
    "air" 

```

在大多数现代 JavaScript 环境中，这个秘密链接以`__proto__`属性的形式暴露出来，单词`proto`前后各有一个下划线：

```js
    > developer.__proto__ === monkey; 
    true 

```

你可以用这个秘密属性来学习，但最好不要在你的实际脚本中使用它，因为它并不存在于所有浏览器中（特别是 IE），所以你的脚本将无法移植。

注意，`__proto__`与`prototype`不同，因为`__proto__`是实例（对象）的属性，而`prototype`是创建这些对象的构造函数的属性：

```js
    > typeof developer.__proto__; 
    "object" 
    > typeof developer.prototype; 
    "undefined" 
    > typeof developer.constructor.prototype; 
    "object" 

```

再次提醒，你应该只将`__proto__`用于学习或调试目的。或者，如果你足够幸运，并且你的代码只需要在 ES5 兼容的环境中工作，你可以使用`Object.getPrototypeOf()`。

# 扩展内置对象

由内置构造函数创建的对象，如`Array`、`String`，甚至`Object`和`Function`，可以通过使用原型进行扩展（或增强）。这意味着，例如，你可以向`Array`原型添加新方法，这样你就可以使它们对所有数组可用。让我们看看如何做到这一点。

在 PHP 中，有一个名为`in_array()`的函数，它告诉你一个值是否存在于数组中。在 JavaScript 中，没有`inArray()`方法，尽管在 ES5 中有一个`indexOf()`，你可以用它来达到相同的目的。所以，让我们实现它并将其添加到`Array.prototype`中，如下所示：

```js
    Array.prototype.inArray = function (needle) { 
      for (var i = 0, len = this.length; i < len; i++) { 
        if (this[i] === needle) { 
          return true; 
        } 
      } 
      return false; 
    }; 

```

现在，所有数组都可以访问这个新方法。让我们测试以下代码：

```js
    > var colors = ['red', 'green', 'blue']; 
    > colors.inArray('red'); 
    true 
    > colors.inArray('yellow'); 
    false 

```

那真是太简单了！让我们再来一次。想象一下，你的应用程序经常需要拼写单词的倒序，你感觉应该有一个内置的`reverse()`方法用于字符串对象。毕竟，数组有`reverse()`方法。你可以通过借用`Array.prototype.reverse()`（在第四章的末尾有一个类似的练习）轻松地将`reverse()`方法添加到`String`原型中：

```js
    String.prototype.reverse = function () { 
      return Array.prototype.reverse. 
               apply(this.split('')).join(''); 
    }; 

```

这段代码使用`split()`方法从一个字符串创建一个数组，然后在这个数组上调用`reverse()`方法，从而产生一个反转的数组。然后，使用`join()`方法将这个数组转换回一个字符串。让我们测试这个新方法：

```js
    > "bumblebee".reverse(); 
      "eebelbmub"

```

## 扩展内置对象 - 讨论

通过原型扩展内置对象是一种强大的技术，你可以用它以任何你喜欢的样子塑造 JavaScript。然而，由于其强大的功能，在使用这种方法之前，你应该始终彻底考虑你的选择。

原因是，一旦你了解了 JavaScript，你就期望它在任何第三方库或小部件中都以相同的方式工作。修改核心对象可能会让你的代码的用户和维护者感到困惑，并产生意外的错误。

JavaScript 不断发展，浏览器厂商持续支持更多功能。您今天认为缺失的方法，并决定添加到核心原型中的方法，明天可能就成为了内置方法。在这种情况下，您的方法就不再需要了。此外，如果您已经编写了大量使用该方法的代码，而您的方法与新的内置实现略有不同，那会怎样呢？

增强内置原型的最常见和可接受的使用案例是为旧浏览器添加对新特性的支持（这些特性已经被 ECMAScript 委员会标准化并在新浏览器中实现）。一个例子是将 ES5 方法添加到旧版本的 IE 中。这些扩展被称为**shim**或**polyfills**。

在增强原型时，您将首先检查方法是否存在，然后再自行实现。这样，如果浏览器中存在原生实现，您就可以使用它。例如，让我们为字符串添加`trim()`方法，这是一个存在于 ES5 中但在旧浏览器中缺失的方法：

```js
    if (typeof String.prototype.trim !== 'function') { 
      String.prototype.trim = function () { 
        return this.replace(/^\s+|\s+$/g,''); 
      }; 
    } 
    > " hello ".trim(); 
    "hello" 

```

### **提示**

**最佳实践**

如果您决定增强内置对象或其原型的新属性，请首先检查该属性是否存在。

## 原型陷阱

在处理原型时，以下两个重要行为需要考虑：

+   原型链是活跃的，除非您完全替换了`prototype`对象

+   `prototype.constructor`方法并不可靠

让我们创建一个简单的构造函数和两个对象：

```js
    > function Dog() { 
        this.tail = true; 
      } 
    > var benji = new Dog(); 
    > var rusty = new Dog(); 

```

即使您已经创建了`benji`和`rusty`对象，您仍然可以向`Dog()`的原型添加属性，现有的对象将能够访问这些新属性。让我们添加`say()`方法：

```js
    > Dog.prototype.say = function () { 
        return 'Woof!'; 
      }; 

```

两个对象都可以访问新方法：

```js
    > benji.say(); 
    "Woof!" 
     rusty.say(); 
    "Woof!" 

```

到目前为止，如果您咨询您的对象，询问它们是由哪个构造函数创建的，它们会正确报告：

```js
    > benji.constructor === Dog; 
    true 
    > rusty.constructor === Dog; 
    true 

```

现在，让我们完全用一个新的对象覆盖`prototype`对象：

```js
    > Dog.prototype = { 
        paws: 4, 
        hair: true 
      }; 

```

结果表明，旧对象无法访问新原型的属性；它们仍然保留指向旧原型对象的秘密链接，如下所示：

```js
    > typeof benji.paws; 
    "undefined" 
    > benji.say(); 
    "Woof!" 
    > typeof benji.__proto__.say; 
    "function" 
    > typeof benji.__proto__.paws; 
    "undefined" 

```

从现在开始创建的任何新对象都将使用更新的原型，如下所示：

```js
    > var lucy = new Dog(); 
    > lucy.say(); 
    TypeError: lucy.say is not a function 
    > lucy.paws; 
    4 

```

秘密的`__proto__`链接指向新的原型对象，如下面的代码行所示：

```js
    > typeof lucy.__proto__.say; 
    "undefined" 
    > typeof lucy.__proto__.paws; 
    "number" 

```

现在，新对象的`constructor`属性不再正确报告。您会期望它指向`Dog()`，但相反，它指向`Object()`，如下面的示例所示：

```js
    > lucy.constructor; 
    function Object() { [native code] } 
    > benji.constructor; 
    function Dog() { 
      this.tail = true; 
    } 

```

您可以通过在完全覆盖原型后重置`constructor`属性来轻松防止这种混淆，如下所示：

```js
    > function Dog() {} 
    > Dog.prototype = {}; 
    > new Dog().constructor === Dog; 
    false 
    > Dog.prototype.constructor = Dog; 
    > new Dog().constructor === Dog; 
    true 

```

### **提示**

**最佳实践**

当您覆盖原型时，请记住重置`constructor`属性。

# 练习

让我们练习以下练习：

1.  创建一个名为`shape`的对象，它具有`type`属性和`getType()`方法。

1.  定义一个`Triangle()`构造函数，其原型是`shape`。使用`Triangle()`创建的对象应该有三个自己的属性-`a`、`b`和`c`，分别代表三角形的三边长度。

1.  在原型中添加一个名为`getPerimeter()`的新方法。

1.  使用以下代码测试你的实现：

    ```js
            > var t = new Triangle(1, 2, 3); 
            > t.constructor === Triangle; 
                   true 
            > shape.isPrototypeOf(t); 
                   true 
            > t.getPerimeter(); 
                   6 
            > t.getType(); 
                   "triangle" 

    ```

1.  遍历`t`，只显示你的自己的属性和方法，不显示原型的任何属性。

1.  让以下代码工作：

    ```js
            > [1, 2, 3, 4, 5, 6, 7, 8, 9].shuffle(); 
              [2, 4, 1, 8, 9, 6, 5, 3, 7] 

    ```

# 摘要

让我们总结一下你在本章中学到的最重要的主题：

+   所有函数都有一个名为`prototype`的属性。最初，它包含一个空对象——一个没有任何自己的属性的对象。

+   你可以向`prototype`对象添加属性和方法。你甚至可以完全用你选择的对象替换它。

+   当你使用函数作为构造函数（使用`new`关键字）创建一个对象时，该对象会获得一个指向构造函数原型的秘密链接，并可以访问原型的属性。

+   对象的自身属性具有比同名原型属性更高的优先级。

+   使用`hasOwnProperty()`方法来区分对象的自身属性和`prototype`属性。

+   存在一个原型链。当你执行`foo.bar`，如果`foo`对象没有名为`bar`的属性，JavaScript 解释器会在原型中查找`bar`属性。如果没有找到，它会继续在原型的原型中查找，然后是原型的原型的原型，并且会一直向上查找，直到`Object.prototype`。

+   你可以增强内置构造函数的原型，所有对象都将看到你的添加。将一个函数赋值给`Array.prototype.flip`，所有数组将立即获得一个`flip()`方法，就像`[1,2,3].flip()`。但是，请确保你想要添加的方法/属性尚未存在，这样你可以确保你的脚本在未来是安全的。
