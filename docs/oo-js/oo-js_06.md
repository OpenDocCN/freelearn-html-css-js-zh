# 第六章。原型

在本章中，您将学习函数对象的`prototype`属性。理解`prototype`的工作原理是学习 JavaScript 语言的重要部分。毕竟，JavaScript 经常被归类为具有基于原型的对象模型。原型并不特别困难，但它是一个新概念，因此有时可能需要一点时间才能理解。就像闭包（见第三章，“函数”）一样，原型是 JavaScript 中的一些东西，一旦理解，就会显得如此明显并且合乎逻辑。与本书的其余部分一样，强烈建议您输入并尝试这些示例-这样学习和记忆概念会更容易。

在本章中，我们将涵盖以下主题：

+   每个函数都有一个`prototype`属性，它包含一个对象

+   向原型对象添加属性

+   使用添加到原型的属性

+   自有属性和原型属性之间的区别

+   `__proto__`属性，每个对象都保留着与其原型的秘密链接

+   诸如`isPrototypeOf()`、`hasOwnProperty()`和`propertyIsEnumerable()`之类的方法

+   增强内置对象，例如数组或字符串，以及为什么这可能是一个坏主意

# 原型属性

JavaScript 中的函数是对象，它们包含方法和属性。您已经熟悉的一些方法是`apply()`和`call()`，其他一些属性是`length`和`constructor`。函数对象的另一个属性是`prototype`。

如果您定义一个简单的函数`foo()`，您可以像处理其他对象一样访问它的属性。考虑以下代码：

```js
    > function foo(a, b) { 
        return a * b; 
      } 
    > foo.length; 
    2 
    > foo.constructor; 
    function Function() { [native code] } 

```

`prototype`属性是在定义函数时立即可用的属性。它的初始值是一个空对象：

```js
    > typeof foo.prototype; 
    "object" 

```

就好像您自己添加了这个属性一样，如下所示：

```js
    > foo.prototype = {}; 

```

您可以用属性和方法增强这个空对象。它们不会对`foo()`函数本身产生任何影响；它们只会在您将`foo()`作为构造函数调用时使用。

## 使用原型添加方法和属性

在上一章中，您学习了如何定义构造函数，以便用于创建（构造）新对象。主要思想是，在使用`new`调用的函数内部，您将可以访问`this`值，它指的是构造函数返回的对象。增强，即向`this`添加方法和属性，是您可以向正在构造的对象添加功能的方法。

让我们来看看构造函数`Gadget()`，它使用`this`向创建的对象添加了两个属性和一个方法，如下所示：

```js
    function Gadget(name, color) { 
      this.name = name; 
      this.color = color; 
      this.whatAreYou = function () { 
        return 'I am a ' + this.color + ' ' + this.name; 
      }; 
    } 

```

向构造函数的`prototype`属性添加方法和属性是另一种为该构造函数产生的对象添加功能的方法。让我们添加两个属性`price`和`rating`，以及一个`getInfo()`方法。由于`prototype`已经指向一个对象，您可以继续向其添加属性和方法，如下所示：

```js
    Gadget.prototype.price = 100; 
    Gadget.prototype.rating = 3; 
    Gadget.prototype.getInfo = function () { 
      return 'Rating: ' + this.rating + 
             ', price: ' + this.price; 
    }; 

```

或者，您可以完全覆盖`prototype`对象，用您选择的对象替换它，如下例所示：

```js
    Gadget.prototype = { 
      price: 100, 
      rating: ...  /* and so on... */ 
    }; 

```

# 使用原型的方法和属性

您添加到`prototype`的所有方法和属性在使用构造函数创建新对象时立即可用。如果您使用`Gadget()`构造函数创建一个`newtoy`对象，您可以访问已经定义的所有方法和属性，如下所示：

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

重要的是要注意`prototype`是活动的。在 JavaScript 中，对象是按引用传递的，因此`prototype`不会随着每个新对象实例的创建而复制。这在实践中意味着什么？这意味着你可以随时修改`prototype`，所有的对象，甚至是在修改之前创建的对象，都会看到这些变化。

让我们通过向`prototype`添加一个新方法来继续示例：

```js
    Gadget.prototype.get = function (what) { 
      return this[what]; 
    }; 

```

即使`newtoy`对象在`get()`方法定义之前创建，`newtoy`对象仍然可以访问新方法，如下所示：

```js
    > newtoy.get('price'); 
    100 
    > newtoy.get('color'); 
    "black" 

```

## 自有属性与原型属性

在上面的例子中，`getInfo()`被内部使用来访问对象的属性。它也可以使用`Gadget.prototype`来实现相同的输出，如下所示：

```js
    Gadget.prototype.getInfo = function () { 
      return 'Rating: ' + Gadget.prototype.rating + 
             ', price: ' + Gadget.prototype.price; 
    }; 

```

有什么区别？为了回答这个问题，让我们详细研究一下`prototype`的工作原理。

让我们再次看看`newtoy`对象：

```js
    var newtoy = new Gadget('webcam', 'black'); 

```

当你尝试访问`newtoy`的属性，比如`newtoy.name`，JavaScript 引擎会查找对象的所有属性，寻找名为`name`的属性，如果找到，就返回它的值，如下所示：

```js
    > newtoy.name; 
    "webcam" 

```

如果你尝试访问`rating`属性会发生什么？JavaScript 引擎会检查`newtoy`对象的所有属性，没有找到名为`rating`的属性。然后，脚本引擎会识别用于创建此对象的构造函数的`prototype`（与`newtoy.constructor.prototype`相同）。如果在`prototype`对象中找到属性，则使用该属性：

```js
    > newtoy.rating; 
    3 

```

你可以做同样的事情并直接访问`prototype`。每个对象都有一个`constructor`属性，它是指向创建对象的函数的引用，所以在这种情况下看看下面的代码：

```js
    > newtoy.constructor === Gadget; 
    true 
    > newtoy.constructor.prototype.rating; 
    3 

```

现在，让我们进一步查找。每个对象都有一个构造函数。`prototype`是一个对象，所以它也必须有一个构造函数，而构造函数又有一个`prototype`。你可以沿着原型链向上查找，最终会得到内置的`Object()`对象，它是最高级的父对象。实际上，这意味着如果你尝试`newtoy.toString()`，而`newtoy`没有自己的`toString()`方法，它的`prototype`也没有，最终你会得到对象的`toString()`方法：

```js
    > newtoy.toString(); 
    "[object Object]" 

```

## 用自有属性覆盖原型的属性

正如前面的讨论所示，如果你的对象没有自己的某个属性，它可以使用原型链上的属性。如果对象有自己的属性，原型也有一个同名的属性会发生什么？那么自有属性优先于原型的属性。

考虑这样一个情景，一个属性名称既存在于自有属性中，又存在于`prototype`对象的属性中：

```js
    > function Gadget(name) { 
        this.name = name; 
      } 
    > Gadget.prototype.name = 'mirror'; 

```

创建一个新对象并访问它的`name`属性会给你对象自己的`name`属性，如下所示：

```js
    > var toy = new Gadget('camera'); 
    > toy.name; 
    "camera" 

```

可以使用`hasOwnProperty()`来确定属性的定义位置，如下所示：

```js
    > toy.hasOwnProperty('name'); 
    true 

```

如果删除`toy`对象自有的`name`属性，原型的具有相同名称的属性将显示出来：

```js
    > delete toy.name; 
    true 
    > toy.name; 
    "mirror" 
    > toy.hasOwnProperty('name'); 
    false 

```

当然，你总是可以重新创建对象的自有属性，如下所示：

```js
    > toy.name = 'camera'; 
    > toy.name; 
    "camera" 

```

你可以使用`hasOwnProperty()`方法来查找你感兴趣的特定属性的来源。前面提到了`toString()`方法。它是从哪里来的？

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

如果你想列出对象的所有属性，可以使用`for...in`循环。在第二章中，*基本数据类型、数组、循环和条件*，你看到你也可以使用`for...in`循环遍历数组的所有元素，但正如在那里提到的，`for`更适合数组，`for...in`更适合对象。让我们以构造一个查询字符串的 URL 为例：

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

这将产生以下`url`字符串：

`http://example.org/page.php?productid=666&section=products`。

以下是一些需要注意的细节：

+   并非所有属性都会出现在`for...in`循环中。例如，长度（对于数组）和构造函数属性不会显示出来。能够显示出来的属性被称为可枚举。你可以通过每个对象提供的`propertyIsEnumerable()`方法来检查哪些属性是可枚举的。在 ES5 中，你可以指定哪些属性是可枚举的，而在 ES3 中你没有这种控制。

+   通过原型链传递的原型也会显示出来，只要它们是可枚举的。你可以使用`hasOwnProperty()`方法来检查属性是对象自己的属性还是原型的属性。

+   `propertyIsEnumerable()`方法对于原型的所有属性都返回`false`，即使它们是可枚举的并出现在`for...in`循环中。

让我们看看这些方法的实际应用。看看这个简化版本的`Gadget()`：

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

创建一个新对象如下：

```js
    var newtoy = new Gadget('webcam', 'black'); 

```

现在，如果你使用`for...in`循环进行循环，你可以看到对象的所有属性，包括来自原型的属性：

```js
    for (var prop in newtoy) {  
      console.log(prop + ' = ' + newtoy[prop]);  
    } 

```

结果还包括对象的方法，因为方法只是恰好是函数的属性：

```js
    name = webcam 
    color = black 
    getName = function () { 
      return this.name; 
    } 
    price = 100 
    rating = 3 

```

如果你想区分对象自己的属性和原型的属性，使用`hasOwnProperty()`。首先尝试以下操作：

```js
    > newtoy.hasOwnProperty('name'); 
    true 
    > newtoy.hasOwnProperty('price'); 
    false 

```

让我们再次循环，但这次只显示对象自己的属性：

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

现在，让我们尝试`propertyIsEnumerable()`。这个方法对于对象自己的非内置属性返回`true`，例如：

```js
    > newtoy.propertyIsEnumerable('name'); 
    true 

```

大多数内置属性和方法都不可枚举：

```js
    > newtoy.propertyIsEnumerable('constructor'); 
    false 

```

原型链中传递下来的任何属性都不可枚举：

```js
    > newtoy.propertyIsEnumerable('price'); 
    false 

```

然而，要注意的是，如果你到达`prototype`中包含的对象并调用它的`propertyIsEnumerable()`方法，这样的属性是可枚举的。考虑以下代码：

```js
    > newtoy.constructor.prototype.propertyIsEnumerable('price'); 
    true 

```

## 使用`isPrototypeOf()`方法

对象也有`isPrototypeOf()`方法。这个方法告诉你特定的对象是否被用作另一个对象的原型。

让我们来看一个名为`monkey`的简单对象：

```js
    var monkey = {    
      hair: true,    
      feeds: 'bananas',    
      breathes: 'air'  
    }; 

```

现在，让我们创建一个`Human()`构造函数，并将其`prototype`属性指向`monkey`：

```js
    function Human(name) { 
      this.name = name; 
    } 
    Human.prototype = monkey; 

```

现在，如果你创建一个名为`george`的新`Human`对象，并询问`monkey`是`george`的原型吗？你会得到`true`：

```js
    > var george = new Human('George');  
    > monkey.isPrototypeOf(george); 
    true 

```

请注意，你必须知道或怀疑原型是谁，然后问你的原型是否是`monkey`？以确认你的怀疑。但是，如果你什么都不怀疑，你一无所知呢？你能否询问对象告诉你它的原型？答案是，在大多数浏览器中你不能，但在大多数浏览器中你可以。大多数最新的浏览器已经实现了 ES5 的一个补充，叫做`Object.getPrototypeOf()`。

```js
    > Object.getPrototypeOf(george).feeds; 
    "bananas" 
    > Object.getPrototypeOf(george) === monkey; 
    true 

```

对于一些没有`getPrototypeOf()`的 ES5 之前的环境，你可以使用特殊属性`__proto__`。

## 秘密的`__proto__`链接

正如你已经知道的，当你尝试访问当前对象中不存在的属性时，会查找`prototype`属性。

考虑另一个名为`monkey`的对象，并在使用`Human()`构造函数创建对象时将其用作原型：

```js
    > var monkey = { 
        feeds: 'bananas', 
        breathes: 'air' 
      }; 
    > function Human() {}  
    > Human.prototype = monkey; 

```

现在，让我们创建一个`developer`对象，并给它以下属性：

```js
    > var developer = new Human(); 
    > developer.feeds = 'pizza'; 
    > developer.hacks = 'JavaScript'; 

```

现在，让我们访问这些属性（例如，`hacks`是`developer`对象的一个属性）：

```js
    > developer.hacks; 
    "JavaScript" 

```

`feeds`属性也可以在对象中找到，如下：

```js
    > developer.feeds; 
    "pizza" 

```

`breathes`属性并不存在于`developer`对象的属性中，所以会查找原型，就好像有一个秘密链接或通道通向`prototype`对象：

```js
    > developer.breathes; 
    "air" 

```

在大多数现代 JavaScript 环境中，秘密链接被暴露为`__proto__`属性，即`proto`一词前后各有两个下划线：

```js
    > developer.__proto__ === monkey; 
    true 

```

你可以使用这个秘密属性进行学习，但在你的真实脚本中使用它并不是一个好主意，因为它并不在所有浏览器中都存在（特别是 IE），所以你的脚本不具备可移植性。

请注意，`__proto__`和`prototype`不同，`__proto__`是实例（对象）的属性，而`prototype`是用于创建这些对象的构造函数的属性：

```js
    > typeof developer.__proto__; 
    "object" 
    > typeof developer.prototype; 
    "undefined" 
    > typeof developer.constructor.prototype; 
    "object" 

```

再次强调，你应该只在学习或调试目的时使用`__proto__`。或者，如果你足够幸运，你的代码只需要在符合 ES5 的环境中工作，你可以使用`Object.getPrototypeOf()`。

# 增强内置对象

由内置构造函数创建的对象，如`Array`、`String`，甚至`Object`和`Function`，都可以通过原型进行增强。这意味着你可以向`Array`原型添加新方法，以便让它们对所有数组可用。让我们看看如何做到这一点。

在 PHP 中，有一个名为`in_array()`的函数，它告诉你一个值是否存在于数组中。在 JavaScript 中，没有`inArray()`方法，尽管在 ES5 中有`indexOf()`，你可以用它来达到相同的目的。因此，让我们实现它并添加到`Array.prototype`中，如下所示：

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

这很简单！让我们再做一次。想象一下，你的应用程序经常需要将单词倒过来拼写，你觉得字符串对象应该有一个内置的`reverse()`方法。毕竟，数组有`reverse()`。你可以通过借用`Array.prototype.reverse()`来轻松地向`String`原型添加一个`reverse()`方法（在第四章的结尾有一个类似的练习，*对象*）：

```js
    String.prototype.reverse = function () { 
      return Array.prototype.reverse. 
               apply(this.split('')).join(''); 
    }; 

```

这段代码使用`split()`方法从字符串创建一个数组，然后在这个数组上调用`reverse()`方法，产生一个反转的数组。然后使用`join()`方法将结果数组转换回字符串。让我们测试一下新方法：

```js
    > "bumblebee".reverse(); 
      "eebelbmub"

```

## 增强内置对象 - 讨论

通过原型增强内置对象是一种强大的技术，你可以用它来塑造 JavaScript 的任何方式。但是，由于它的强大，你在使用这种方法之前应该仔细考虑你的选择。

原因是一旦你了解了 JavaScript，你期望它以相同的方式工作，无论你使用的是哪个第三方库或小部件。修改核心对象可能会让用户和代码维护者感到困惑，并产生意外的错误。

JavaScript 不断发展，浏览器供应商不断支持更多功能。今天你认为缺少的方法并决定添加到核心原型中的方法，明天可能就成为内置方法。在这种情况下，你的方法就不再需要了。此外，如果你已经编写了大量使用该方法的代码，并且你的方法与新的内置实现略有不同，会怎么样呢？

增强内置原型的最常见和可接受的用例是为旧浏览器添加对新功能的支持（这些功能已经由 ECMAScript 委员会标准化并在新浏览器中实现）。一个例子是在旧版本的 IE 中添加 ES5 方法。这些扩展被称为**shims**或**polyfills**。

在增强原型时，你首先要检查方法是否存在，然后再自己实现。这样，如果浏览器中存在原生实现，你就可以使用它。例如，让我们为字符串添加`trim()`方法，这是 ES5 中存在的方法，但在旧浏览器中缺少：

```js
    if (typeof String.prototype.trim !== 'function') { 
      String.prototype.trim = function () { 
        return this.replace(/^\s+|\s+$/g,''); 
      }; 
    } 
    > " hello ".trim(); 
    "hello" 

```

### 提示

**最佳实践**

如果你决定增强内置对象或其原型以添加新属性，首先要检查新属性是否存在。

## 原型陷阱

处理原型时需要考虑的两个重要行为是：

+   原型链是活跃的，除非你完全替换了`prototype`对象

+   `prototype.constructor`方法不可靠

让我们创建一个简单的构造函数和两个对象：

```js
    > function Dog() { 
        this.tail = true; 
      } 
    > var benji = new Dog(); 
    > var rusty = new Dog(); 

```

即使您已经创建了`benji`和`rusty`对象，您仍然可以向`Dog()`的原型添加属性，现有对象将可以访问新属性。让我们加入`say()`方法：

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

到目前为止，如果您咨询您的对象，询问它们是使用哪个构造函数创建的，它们将正确报告：

```js
    > benji.constructor === Dog; 
    true 
    > rusty.constructor === Dog; 
    true 

```

现在，让我们完全用全新的对象覆盖`prototype`对象：

```js
    > Dog.prototype = { 
        paws: 4, 
        hair: true 
      }; 

```

事实证明，旧对象无法访问新原型的属性；它们仍然保留指向旧原型对象的秘密链接，如下所示：

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

从现在开始创建的任何新对象都将使用更新后的原型，如下所示：

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

现在新对象的`constructor`属性不再正确报告。您期望它指向`Dog()`，但实际上它指向`Object()`，如下例所示：

```js
    > lucy.constructor; 
    function Object() { [native code] } 
    > benji.constructor; 
    function Dog() { 
      this.tail = true; 
    } 

```

在完全覆盖原型后，您可以通过重置`constructor`属性轻松防止混淆，如下所示：

```js
    > function Dog() {} 
    > Dog.prototype = {}; 
    > new Dog().constructor === Dog; 
    false 
    > Dog.prototype.constructor = Dog; 
    > new Dog().constructor === Dog; 
    true 

```

### 提示

**最佳实践**

当您覆盖原型时，请记得重置`constructor`属性。

# 练习

让我们练习以下练习：

1.  创建一个名为`shape`的对象，该对象具有类型`property`和一个`getType()`方法。

1.  定义一个`Triangle()`构造函数，其原型是`shape`。使用`Triangle()`创建的对象应该有三个自有属性-`a`，`b`和`c`，表示三角形的边长。

1.  在原型中添加一个名为`getPerimeter()`的新方法。

1.  使用以下代码测试您的实现：

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

1.  循环遍历`t`，仅显示您自己的属性和方法，而不是原型的。

1.  使以下代码工作：

```js
        > [1, 2, 3, 4, 5, 6, 7, 8, 9].shuffle(); 
          [2, 4, 1, 8, 9, 6, 5, 3, 7] 

```

# 总结

让我们总结一下您在本章学到的最重要的主题：

+   所有函数都有一个名为`prototype`的属性。最初，它包含一个空对象-一个没有任何自有属性的对象。

+   您可以向`prototype`对象添加属性和方法。您甚至可以完全替换它为您选择的对象。

+   当您使用函数作为构造函数创建对象（使用`new`）时，对象会得到一个指向构造函数原型的秘密链接，并且可以访问原型的属性。

+   对象的自有属性优先于具有相同名称的原型属性。

+   使用`hasOwnProperty()`方法区分对象的自有属性和`prototype`属性。

+   存在原型链。当您执行`foo.bar`时，如果您的`foo`对象没有名为`bar`的属性，JavaScript 解释器将在原型中查找`bar`属性。如果找不到，则会继续在原型的原型中查找，然后在原型的原型的原型中查找，一直到`Object.prototype`。

+   您可以增强内置构造函数的原型，并且所有对象都将看到您的添加。将一个函数分配给`Array.prototype.flip`，所有数组将立即获得一个`flip()`方法，就像`[1,2,3].flip()`一样。但是，请检查您要添加的方法/属性是否已经存在，以便为您的脚本未来保值。
