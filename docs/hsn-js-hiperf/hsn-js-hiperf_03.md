# 第三章：Vanilla Land - 看现代 Web

自 ECMAScript 2015 标准发布以来，JavaScript 语言的格局发生了很大变化。现在有许多新功能使 JavaScript 成为各种开发的一流语言。使用该语言变得更容易，我们现在甚至可以看到一些语法糖。

从 ECMAScript 2015 标准及以后，我们已经获得了类、模块、更多声明变量的方式、作用域的变化等。所有这些特性等等将在本章的其余部分进行解释。如果您对该语言还不熟悉，或者只是想了解一下可能不熟悉的特性，这是一章值得阅读的好章节。我们还将看一下一些旧的 Web 部分，如 DOM 查询，以及我们如何利用它们来替换我们可能当前正在使用的多余库，如 jQuery。

在本章中，将涵盖以下主题：

+   深入现代 JavaScript

+   理解类和模块

+   与 DOM 一起工作

+   理解 Fetch API

# 技术要求

本章的先决条件如下：

+   诸如 VS Code 之类的编辑器

+   一个使用 Node.js 的系统

+   一个浏览器，最好是 Chrome

+   对 JavaScript 及其作用域的一般理解

+   相关代码可以在[`github.com/PacktPublishing/Hands-On-High-Performance-Web-Development-with-JavaScript/tree/master/Chapter03`](https://github.com/PacktPublishing/Hands-On-High-Performance-Web-Development-with-JavaScript/tree/master/Chapter03)找到。

# 深入现代 JavaScript

如介绍中所述，语言在许多方面都有所改进。我们现在有了适当的作用域，更好地处理`async`操作，更多的集合类型，甚至元编程特性，如反射和代理。所有这些特性都导致了更复杂的语言，但也导致了更有效的问题解决。我们将看一下新标准中出现的一些最佳项，以及它们在我们的代码中可以用来做什么。

另一个需要注意的是，未来显示的任何 JavaScript 代码都可以通过以下方式运行：

1.  通过按下键盘上的*F12*将其添加到开发者控制台

1.  利用开发者控制台中可以在“Sources”选项卡中看到的片段，在左侧面板中应该有一个名为“Snippets”的选项

1.  编写一个基本的`index.html`，其中添加了一个脚本元素

# Let/const 和块作用域

在 ECMAScript 2015 之前，我们只能使用`var`关键字来定义变量。`var`关键字的生命周期从函数声明到函数结束。这可能会导致很多问题。以下代码展示了我们可能在`var`关键字中遇到的问题之一：

```js
var fun = function() {
    for(var i = 0; i < 10; i++) {
        state['this'] += 'what';
    }
    console.log('i', i);
}
fun();
```

控制台会打印出什么？在大多数语言中，我们可能会猜想这是一个错误，或者会打印`null`。然而，JavaScript 的`var`关键字是函数作用域的，所以变量`i`将是`10`。这导致了许多错误的出现，因为意外地忘记声明变量，甚至可怕的`switch`语句错误（这些错误仍然会发生在`let`和`const`中）。`switch`语句错误的一个例子如下：

```js
var x = 'a';
switch(x) {
    case 'a':
        y = 'z';
        break;
    case 'b':
        y = 'y';
        break;
    default:
        y = 'b';
}
console.log(y);
```

从前面的`switch`语句中，我们期望`y`是`null`，但因为`var`关键字不是块作用域的，它将是字母`z`。我们总是必须掌握变量并确保我们没有使用在我们范围之外声明的东西并改变它，或者我们确保我们重新声明变量以阻止泄漏发生。

使用`let`和`const`，我们得到了块作用域。这意味着花括号告诉我们变量应该存在多久。这里有一个例子：

```js
let x = 10;
let fun2 = function() {
    {
        let x = 20;
        console.log('inner scope', x);
    }
    console.log('outer scope', x);
    x += 10;
}
fun2();
console.log('this should be 20', x);
```

当我们查看变量`x`的打印输出时，我们可以看到我们首先在函数外部将其声明为`10`。在函数内部，我们使用大括号创建了一个新的作用域，并将`x`重新声明为`20`。在块内部，代码将打印出`inner scope 20`。但是，在`fun2`内部的块之外，我们打印出`x`，它是`10`。`let`关键字遵循此块作用域。如果我们将变量声明为`var`，则第二次打印时它将保持为`20`。最后，我们将`10`添加到外部的`x`，我们应该看到`x`是`20`。

除了获得块作用域之外，`const`关键字还赋予了我们一些不可变性。如果我们正在使用的类型是值类型，我们将无法改变该值。如果我们有一个引用类型，引用内部的值可以被改变，但是我们不能改变引用本身。这带来了一些很好的功能。

一个很好的编码风格是尽可能多地使用`const`，只有在需要在基本级别上改变某些东西时才使用`let`，比如循环。由于对象、数组或函数的值可以被改变，我们可以将它们设置为`const`。唯一的缺点是它们不能被置空，但它仍然在可能的性能增益之上增加了相当多的安全性，编译器可以利用知道一个值是不可变的。

# 箭头函数

语言的另一个显著变化是添加了箭头函数。有了这个，我们现在可以在不使用语言上的各种技巧的情况下改变`this`。可以看到以下示例：

```js
const create = function() {
    this.x = 10;
    console.log('this', this);
    const innerFun = function() {
        console.log('inner this', this);
    }
    const innerArrowFun = () => {
        console.log('inner arrow this', this);
    }
    innerFun();
    innerArrowFun();
}
const item = new create();
```

我们正在为一个新对象创建一个构造函数。我们有两个内部函数，一个是基本函数调用，另一个是箭头函数。当我们打印这个时，我们注意到基本函数打印出了窗口的作用域。当我们打印内部箭头函数的作用域时，我们得到了父级的作用域。

我们可以通过几种方式来解决基本内部函数的问题。首先，我们可以在父级中声明一个变量，并在内部函数中使用它。此外，当我们运行函数时，我们可以使用 call 或`apply`来实际运行函数。

然而，这两种方法都不是一个好主意，特别是当我们现在有箭头函数时。要记住的一个关键点是箭头函数获取父级的作用域，所以无论`this`指向父级的什么，我们现在都将在箭头函数内部执行相同的操作。现在，我们可以通过在箭头函数上使用`apply`来始终更改它，但最好只使用`apply`等来进行部分应用，而不是通过更改其`this`关键字来调用函数。

# 集合类型

数组和对象一直是 JavaScript 开发人员使用的两种主要类型。但是，现在我们有了另外两种集合类型，可以帮助我们做一些我们过去使用这些其他类型的事情。这些是 set 和 map。set 是一个无序的唯一项集合。这意味着如果我们试图将已经存在的东西放入 set 中，我们会注意到我们只有一个单一项。我们可以很容易地用数组模拟一个 set，如下所示：

```js
const set = function(...items) {
   this._arr = [...items];
   this.add = function(item) {
       if( this._arr.includes(item) ) return false;
       this._arr.push(item);
       return true;
   }
   this.has = function(item) {
       return this._arr.includes(item);
   }
   this.values = function() {
       return this._arr;
   }
   this.clear = function() {
       this._arr = [];
   }
}
```

由于我们现在有了 set 系统，我们可以直接使用该 API。我们还可以访问`for of`循环，因为 set 是一个可迭代项（如果我们获取附加到 set 的迭代器，我们也可以使用下一个语法）。与数组相比，当我们处理大型数据集时，set 在读取访问速度上也具有优势。以下示例说明了这一点：

```js
const data = new Array(10000000);
for(let i = 0; i < data.length; i++) {
    data[i] = i;
}
const setData = new Set();
for(let i = 0; i < data.length; i++) {
    setData.add(i);
}
data.includes(5000000);
setData.has(5000000);
```

尽管创建 set 需要一些时间，但是当查找项目或甚至获取它们时，set 的性能几乎比数组快 100 倍。这主要是由于数组查找项目的方式。由于数组是纯线性的，它必须遍历每个元素进行检查，而 set 是一个简单的常量时间检查。

集合可以根据引擎的不同方式实现。V8 引擎中的集合是利用哈希字典进行查找构建的。我们不会详细介绍这些内部情况，但基本上，查找时间被认为是常数，或者对于计算机科学家来说是*O(1)*，而数组查找时间是线性的，或者*O(n)*。

除了集合，我们还有地图。我们可以将它们视为普通对象，但它们有一些很好的属性：

+   首先，我们可以使用任何值作为键，甚至是对象。这对于添加我们不想直接绑定到对象的其他数据非常有用（私有值浮现在脑海中）。

+   除此之外，地图也是可迭代的，因此我们可以像集合一样利用`for of`循环。

+   最后，地图可以在大型数据集和键和值类型相同的情况下为我们带来性能优势。

以下示例突出了地图通常比普通对象更好的许多领域，以及曾经使用对象的领域：

```js
const map = new Map();
for(let i = 0; i < 10000; i++) {
    map.set(`${i}item`, i);
}
map.forEach((val, key) => console.log(val));
map.size();
map.has('0item');
map.clear();
```

除了这两个项目，我们还有它们的弱版本。弱版本有一个主要限制：值必须是对象。一旦我们了解了`WeakSet`和`WeakMap`的作用，这就说得通了。它们*弱地*存储对项目的引用。这意味着当它们存储的项目存在时，我们可以执行这些接口给我们的方法。一旦垃圾收集器决定收集它们，引用将从弱版本中删除。我们可能会想，为什么要使用这些？

对于`WeakMap`，有一些用例：

+   首先，如果我们没有私有变量，我们可以利用`WeakMap`在对象上存储值，而实际上不将属性附加到它们上。现在，当对象最终被垃圾收集时，这个私有引用也会被回收。

+   我们还可以利用弱映射将属性或数据附加到 DOM，而实际上不必向 DOM 添加属性。我们可以获得数据属性的所有好处，而不会使 DOM 混乱。

+   最后，如果我们想要将引用数据存储到一边，但在数据消失时使其消失，这是另一个用例。

总的来说，当我们想要将某种数据与对象绑定而不需要紧密耦合时，我们会使用`WeakMap`。我们将能够看到这一点，如下所示：

```js
const items = new WeakMap();
const container = document.getElementById('content');
for(let i = 0; i < 50000; i++) {
    const el = document.createElement('li');
    el.textContent = `we are element ${i}`;
    el.onclick = function(ev) {
        console.log(items.get(el));
    }
    items.set(el, i);
    container.appendChild(el);
}
const removeHalf = function() {
    const amount = Math.floor(container.children.length / 2);
    for(let i = 0; i < amount; i++) {
        container.removeChild(container.firstChild); 
    }
}
```

首先，我们创建一个`WeakMap`来存储我们想要针对创建的 DOM 元素的数据。接下来，我们获取我们的无序列表，并在每次迭代中添加一个列表元素。然后，我们通过`WeakMap`将我们所在的数字与 DOM 元素联系起来。这样，`onclick`处理程序就可以获取该项并取回我们存储在其中的数据。

有了这个，我们可以点击任何元素并取回数据。这很酷，因为我们过去直接在 DOM 中向 HTML 元素添加数据属性。现在我们可以使用`WeakMap`。但是，我们还有一个更多的好处，这已经被讨论过。如果我们在命令行中运行`removeHalf`函数并进行垃圾收集，我们可以看一下`WeakMap`中有多少项。如果我们这样做，并检查`WeakMap`中有多少元素，我们会注意到它存储的元素数量可以从 25,000 到我们开始的完整 50,000 个元素。这是由于上面所述的原因；一旦引用被垃圾收集，`WeakMap`将不再存储它。它具有弱引用。

垃圾收集器要收集的数量将取决于我们正在运行的系统。在某些系统上，垃圾收集器可能决定不从列表中收集任何内容。这完全取决于 Chrome 或 Node.js 中的 V8 垃圾收集是如何设置的。

如果我们用普通的`WeakMap`替换它，我们很容易看到这一点。让我们继续进行这个小改变。通过这个改变，观察同样的步骤。我们会注意到地图仍然有 50,000 个项目。这就是我们所说的，当我们说某物有强引用或弱引用时的意思。弱引用将允许垃圾收集器清理项目，而强引用则不会。*WeakMaps*非常适合这种数据与另一个数据源的链接。如果我们希望在主对象被清理时清理项目装饰或链接，`WeakMap`是一个不错的选择。

`WeakSet`有一个更有限的用例。一个很好的用例是检查对象属性或图中的无限循环。如果我们将所有访问过的节点存储在`WeakSet`中，我们就能够检查我们是否有这些项目，但我们也不必在检查完成后清除集合。这意味着一旦数据被收集，存储在`WeakSet`中的所有引用也将被收集。总的来说，当我们需要标记一个对象或引用时，应该使用`WeakSet`。这意味着如果我们需要查看我们是否拥有它或它是否被访问过，`WeakSet`很可能是这项工作的合适选择。

我们可以利用上一章的深拷贝示例。通过它，我们还遇到了一个我们没有考虑到的用例。如果一个项目指向对象中的另一个项目，并且同一个项目决定再次指向原始项目，会发生什么？这可以在以下代码中看到：

```js
const a = {item1 : b};
const b = {item1 : a};
```

如果每个项目都指向彼此，我们将遇到循环引用的问题。解决这个问题的方法是使用`WeakSet`。我们可以保存所有访问过的节点，如果我们遇到一个已经访问过的节点，我们就从函数中返回。这可以在代码的修改版本中看到：

```js
const state = {};
(function(scope) {
    const _state = {},
          _held = new WeakSet(),
          checkPrimitives = function(item) {
              return item === null || typeof item === 'string' || typeof 
               item === 'number' || typeof item === 'boolean' ||
               typeof item === 'undefined';
          },
          cloneFunction = function(fun, scope=null) {
              return fun.bind(scope);
          },
          cloneObject = function(obj) {
              const newObj = {},
              const keys = Object.keys(obj);
              for(let i = 0; i < keys.length; i++) {
                  const key = keys[i];
                  const item = obj[key];
                  newObj[key] = runUpdate(item);
              }
              return newObj;
          },
          cloneArray = function(arr) {
              const newArr = new Array(arr.length);
              for(let i = 0; i < arr.length; i++) {
                  newArr[i] = runUpdate(arr[i]);
              }
              return newArr;
          },
          runUpdate = function(item) {
              if( checkPrimitives(item) ) {
                  return item;
              }
              if( typeof item === 'function' ) {
                  return cloneFunction(item);
              }
              if(!_held.has(item) ) {
                  _held.add(item);
                  if( item instanceof Array ) {
                      return cloneArray(item);
                  } else {
                      return cloneObject(item);
                  }
              }
          };
    scope.update = function(obj) {
        const x = Object.keys(obj);
        for(let i = 0; i < x.length; i++) {
            _state[x[i]] = runUpdate(obj[x[i]]);
        }
        _held = new WeakSet();
    }
})(state);
Object.freeze(state);
```

正如我们所看到的，我们已经添加了一个新的`_held`变量，它将保存我们所有的引用。然后，`runUpdate`函数已经被修改，以确保当一个项目不是原始类型或函数时，我们检查我们的`held`列表中是否已经有它。如果有，我们就跳过这个项目，否则我们将继续进行。最后，我们用一个新的`WeakSet`替换了`_held`变量，因为在*WeakSets*上`clear`方法不再可用。

这并不会保留循环引用，这可能是一个问题，但它解决了因对象相互引用而导致系统陷入无限循环的问题。除了这种用例，也许还有一些更高级的想法，`WeakSet`并没有太多其他的需求。主要的是，如果我们需要跟踪某物的存在。如果我们需要这样做，`WeakSet`就是我们的完美用例。

大多数开发人员不会发现需要*WeakSets*或*WeakMaps*。这些可能会被库作者使用。然而，之前提到的约定在某些情况下可能会出现，因此了解这些项目的原因和存在的意义是很好的。如果我们没有使用某物的理由，那么我们很可能不应该使用它，这在这两个项目中绝对是这样，因为它们有非常具体的用例，而*WeakMaps*的主要用例之一是在 ECMAScript 标准中提供给我们的（私有变量）。

# 反射和代理

我们要讨论的 ECMAScript 标准的最后一个重要部分是两个元编程对象。元编程是指生成代码的技术。这可能是用于编译器或解析器等工具。它也可以用于自我改变的代码。甚至可以用于运行时评估另一种语言（解释）并对其进行操作。虽然这可能是反射和代理给我们的主要功能，但它也使我们能够监听对象上的事件。

在上一章中，我们谈到了监听事件，并创建了一个`CustomEvent`来监听对象上的事件。好吧，我们可以改变那段代码，并利用代理来实现该行为。以下是处理对象上基本事件的一些基本代码：

```js
const item = new Proxy({}, {
    get: function(obj, prop) {
        console.log('getting the following property', prop);
        return Reflect.has(obj, prop) ? obj[prop] : null;
    },
    set: function(obj, prop, value) {
        console.log('trying to set the following prop with the following 
         value', prop, value);
        if( typeof value === 'string' ) {
            obj[prop] = value;
        } else {
            throw new Error('Value type is not a string!');
        }
    }
});
item.one = 'what';
item.two = 'is';
console.log(item.one);
console.log(item.three);
item.three = 12;
```

我们所做的是为这个对象的`get`和`set`方法添加了一些基本的日志记录。我们通过使`set`方法只接受字符串值，扩展了这个对象的功能。有了这个，我们创建了一个可以被监听的对象，并且我们可以对这些事件做出响应。

代理目前比向系统添加`CustomEvent`要慢。正如前面所述，尽管代理在 ECMAScript 2015 标准中，但它们的采用速度很慢，因此浏览器需要更多时间来优化它们。另外，应该注意的是，我们不希望直接在这里运行日志记录。相反，我们选择让系统排队消息，并利用称为`requestIdleCallback`的东西，在浏览器注意到我们应用程序的空闲时间时运行我们的日志记录代码。这仍然是一项实验性技术，但应该很快添加到所有浏览器中。

代理的另一个有趣特性是可撤销方法。这是一个代理，我们最终可以说是被撤销的，当我们尝试在此方法调用后使用它时，会抛出`TypeError`。这对于任何试图使用对象实现 RAII 模式的人来说非常有用。我们可以撤销代理，而不再能够利用它，而不是试图将引用`null`掉。

这种 RAII 模式与空引用略有不同。一旦我们撤销了代理，所有引用将不再能够使用它。这可能会成为一个问题，但它也会给我们带来失败快速的额外好处，这在代码开发中总是一个很好的特性。这意味着当我们在开发中时，它会抛出`TypeError`，而不仅仅是传递一个空值。在这种情况下，只有 try-catch 块才能让这段代码继续运行，而不仅仅是简单的空检查。失败快速是保护我们自己在开发中并更早捕获错误的好方法。

这里展示了一个示例，修改了前面代码的版本：

```js
const isPrimitive = function(item) {
    return typeof item === 'string' || typeof item === 'number' || typeof 
     item === 'boolean';
}
const item2 = Proxy.revocable({}, {
    get: function(obj, prop) {
        return Reflect.has(obj, prop) ? obj[prop] : null
    },
    set: function(obj, prop, value) {
        if( isPrimitive(value) ) {
            obj[prop] = value;
        } else {
            throw new Error('Value type is not a primitive!');
        }
    }
});
const item2Proxy = item2.proxy;
item2Proxy.one = 'this';
item2Proxy.two = 12;
item2Proxy.three = true;
item2.revoke();
(function(obj) {
    console.log(obj.one);
})(item2Proxy);
```

现在，我们不仅在设置时抛出*TypeErrors*，一旦我们撤销代理，我们也会抛出`TypeError`。当我们决定编写能够保护自己的代码时，这对我们非常有用。当我们使用对象时，我们也不再需要在代码中编写一堆守卫子句。如果我们使用代理和可撤销代替，我们可以保护我们的设置。

我们没有深入讨论代理系统的术语。从技术上讲，我们在代理处理程序中添加的方法称为陷阱，类似于操作系统陷阱，但我们实际上可以将它们简单地视为简单的事件。有时，术语可能会给事情增加一些混乱，通常是不需要的。

除了代理，反射 API 是一堆静态方法，它们反映了代理处理程序。我们可以在某些熟悉的系统的位置使用它们，比如`Function.prototype.apply`方法。我们可以使用`Reflect.apply`方法，这在编写我们的代码时可能会更清晰一些。如下所示：

```js
Math.max.apply(null, [1, 2, 3]);
Reflect.apply(Math.max, null, [1, 2, 3]);
item3 = {};
if( Reflect.set(item3, 'yep', 12) {
    console.log('value was set correctly!');
} else {
    console.log('value was not set!');
}
Reflect.defineProperty(item3, 'readonly', {value : 42});
if( Reflect.set(item3, 'readonly', 'nope') ) {
    console.log('we set the value');
} else {
    console.log('value should not be set!');
}
```

正如我们所看到的，我们第一次在对象上设置了一个值，并且成功了。但是，第二个属性首先被定义，并且被设置为不可写（当我们使用`defineProperty`时的默认值），因此我们无法在其上设置一个值。

通过这两个 API，我们可以为访问对象编写一些不错的功能，甚至使变异尽可能安全。我们可以很容易地利用这两个 API 来使用 RAII 模式，甚至可以进行一些很酷的元编程。

# 其他值得注意的变化

随着 ECMAScript 标准的进步，出现了许多变化，我们可以专门讨论所有这些变化，但我们将在这里列出一些在本书中编写的代码中以及其他地方可能看到的变化。

# 展开运算符

展开运算符允许我们拆开数组，可迭代集合（如集合或映射），甚至是最新标准中的对象。这为我们提供了更美观的语法，用于执行一些常见操作，例如以下操作：

```js
// working with a variable amount of arguments
const oldVarArgs = function() {
    console.log('old variable amount of arguments', arguments);
}
const varArgs = function(...args) {
    console.log('variable amount of arguments', args);
}
// transform HTML list into a basic array so we have access to array
// operations
const domArr = [...document.getElementsByTagName('li')];
// clone array
const oldArr = [1, 2, 3, 4, 5];
const clonedArr = [...oldArr];
// clone object
const oldObj = {item1 : 'that', item2 : 'this'};
const cloneObj = {...oldObj};
```

以前的 `for` 循环和其他迭代版本现在变成了简单的一行代码。此外，第一个项目很好，因为它向代码的读者显示我们正在将函数用作变量参数函数。我们可以通过代码看到这一点，而不需要文档来说明这一点。

处理参数时，如果我们在函数中要对其进行任何改变，先创建一个副本，然后再进行改变。如果我们决定直接改变参数，会发生某些非优化。

# 解构

解构是将数组或对象的项目以更简单的方式传递给我们分配给的变量的过程。可以在以下代码中看到：

```js
//object
const desObj = {item1 : 'what', item2 : 'is', item3 : 'this'};
const {item1, item2} = desObj;
console.log(item1, item2);

//array
const arr = [1, 2, 3, 4, 5];
const [a, ,b, ...c] = arr;
console.log(a, b, c);
```

这两个示例展示了一些很酷的特性。首先，我们可以从对象中挑选我们想要的项目。我们还可以在左侧重新分配值为其他值。除此之外，我们甚至可以进行嵌套对象和解构。

对于数组，我们可以选择所有项目、部分项目，甚至通过将数组的其余部分放入变量来使用 `rest` 语法。在前面的示例中，`a` 将保存 `1`，`b` 将保存 `3`，`c` 将是一个包含 `4` 和 `5` 的数组。我们通过使该空间为空来跳过了 2。在其他语言中，我们会使用 `_` 来展示这一点，但在这里我们可以直接跳过它。同样，所有这些只是语法糖，使得能够编写更紧凑和更清晰的代码。

# 幂运算符

这里没有什么可说的，除了我们不再需要使用 `Math.pow()` 函数；我们现在有了幂运算符或 `**`，从而使代码更清晰，数学方程更美观。

# 参数默认值

这些允许我们在调用函数时为某个位置放入默认值。可以如下所示：

```js
const defParams = function(arg1, arg2=null, arg3=10) {
    if(!arg2 ) {
        console.log('nothing was passed in or we passed in a falsy value');
    }
    const pow = arg3;
    if( typeof arg1 === 'number' ) {
        return arg1 ** pow;
    } else {
        throw new TypeError('argument 1 was not a number!');
    }
}
```

需要注意的一点是，一旦我们开始在参数链中使用默认值，就不能停止使用默认值。在前面的示例中，如果我们给参数 2 设置了默认值，那么我们必须给参数 3 设置默认值，即使我们只是将 undefined 或 `null` 传递给它。同样，这有助于代码的清晰度，并确保我们不再需要在查看数组的参数时创建默认情况。

很多代码仍然利用函数的参数部分。甚至还有函数的其他属性可以获取，比如调用者。如果我们处于严格模式，很多这种行为都会被破坏。严格模式是一种不允许访问 JavaScript 引擎中某些行为的方式。关于这一点的良好描述可以在[`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode)找到。除此之外，由于新标准提供了许多有用的替代方案，我们不应再使用函数的参数部分。

# 字符串模板

字符串模板允许我们传入将求值为字符串或具有 `toString` 函数的对象的任意代码。这再次使我们能够编写更清晰的代码，而不必创建大量连接的字符串。它还允许我们编写多行字符串，而无需创建转义序列。可以如下所示：

```js
const literal = `This is a string literal. It can hold multiple lines and
variables by denoting them with curly braces and prepended with the dollar 
sign like so \$\{\}.
here is a value from before ${a}. We can also pass an arbitrary expression 
that evaluates ${a === 1 ? b : c}.
`
console.log(literal);
```

只需记住，即使我们可以做某事，做某事可能不是最好的主意。具体来说，我们可能能够传递将求值为某个值的任意表达式，但我们应该尽量保持它们简洁清晰，以使代码更易读。

# 类型化数组

我们将在未来的章节中对此进行详细讨论，但类型化数组是表示系统中任意字节的一种方式。这使我们能够使用更低级的功能，例如编码器和解码器，甚至直接处理`fetch`调用的字节流，而无需将 blob 转换为数字或字符串。

这些通常以`ArrayBuffer`开始，然后我们在其上创建一个视图。这可能看起来像下面这样：

```js
const arrBuf = new ArrayBuffer(16);
const uint8 = new Uint8Array(arrBuf);
uint8[0] = 255;
```

正如我们所看到的，我们首先创建一个数组缓冲区。可以将其视为低级实例。它只保存原始字节。然后我们必须在其上创建一个视图。大部分时间，我们将使用`Uint8Array`，因为我们需要处理任意字节，但我们可以一直使用视图到`BigInt`。这些通常用于低级系统，例如 3D 画布代码、WebAssembly 或来自服务器的原始流。

# BigInt

`BigInt`是一个任意长的整数。JavaScript 中的数字以 64 位浮点数双精度存储。这意味着即使我们只有一个普通整数，我们仍然只能获得 53 位的精度。我们只能在变量中存储数字，最大到 9000 万亿。比这更大的数字通常会导致系统进入未定义的行为。为了弥补这一点，我们现在可以在 JavaScript 中利用`BigInt`特性。这看起来像下面这样：

```js
const bigInt = 100n;
console.log('adding two big ints', 100n + 250n);
console.log('add a big int and a regular number', 100n + BigInt(250));
```

我们会注意到*BigInts*后面会附加一个`n`。如果我们想要在常规操作中使用它们，我们还需要将常规数字强制转换为*BigInts*。现在我们有了大整数，我们可以处理非常大的数字，这在 3D、金融和科学应用中非常有用。

不要试图将*BigInts*强制转换回常规数字。这里存在一些未定义的行为，如果我们尝试这样做，可能会失去精度。最好的方法是，如果我们需要使用*BigInts*，就保持在*BigInts*中。

# 国际化

最后，我们来到国际化。以前，我们需要国际化诸如日期、数字格式甚至货币等内容。我们会使用特殊的查找或转换器来为我们执行这些操作。通过 ECMAScript 的更新版本，我们已经获得了使用内置`Intl`对象获取这些新格式的支持。一些用例可以如下所示：

```js
const amount = 1478.99;
console.log(new Intl.NumberFormat('en-UK', {style : 'currency', currency : 'EUR'}).format(amount));
console.log(new Intl.NumberFormat('de-DE', {style : 'currency', currency : 'EUR'}).format(amount));
const date = new Date(0);
console.log(new Intl.DateTimeFormat('en-UK').format(date));
console.log(new Intl.DateTimeFormat('de-DE').format(date));
```

有了这个，我们现在可以根据某人的所在地或他们在我们应用程序开始时选择的语言来国际化我们的系统。

这只会将数字转换为该国家代码的样式；它不会尝试转换实际值，因为货币等选项在一天之内会发生变化。如果我们需要执行这样的转换，我们将需要使用 API。除此之外，如果我们想要翻译某些内容，我们仍然需要有单独的查找，以确定我们需要在文本中放置什么，因为不同语言之间没有直接的翻译。

有了 ECMAScript 标准中添加的这些令人惊叹的功能，现在让我们转向一种封装函数和数据的方式。为此，我们将使用类和模块。

# 理解类和模块

随着新的 ECMAScript 标准，我们得到了新的类语法，用于实现**面向对象编程**（**OOP**），后来，我们还得到了模块，一种导入和导出用户定义的函数和对象集合的方式。这两种系统使我们能够消除系统中内置的某些黑客技巧，并且也能够移除一些几乎是必不可少的库，用于模块化我们的代码库。

首先，我们需要了解 JavaScript 是什么类型的语言。JavaScript 是一种多范式语言。这意味着我们可以利用许多不同编程风格的思想，并将它们合并到我们的代码库中。我们在之前的章节中提到的一种编程风格是函数式编程。

在纯函数式编程中，我们有纯函数，或者执行操作并且没有副作用（在函数应该执行的外部执行其他操作）的函数。当我们以这种方式编写时，我们可以创建通用函数，并将它们组合在一起，以创建可以处理复杂思想的一系列简单思想。我们还将函数视为语言中的一等公民。这意味着函数可以分配给变量并传递给其他函数。我们还可以组合这些函数，正如我们在之前的章节中所看到的。这是解决问题的一种方式。

另一种流行的编程风格是面向对象编程。这种风格表明程序可以用类和对象的层次结构来描述，并且可以构建和一起使用这些类和对象来创建这个复杂的思想。这个想法可以在大多数流行的语言中看到。我们构建具有一些通用功能或某些特定版本需要合并的定义的基类。我们从这个基类继承并添加我们自己的特定功能，然后我们创建这些对象。一旦我们把所有这些对象放在一起，我们就可以处理我们需要的复杂思想。

使用 JavaScript，我们可以得到这两种思想，但是 JavaScript 中的面向对象设计有点不同。我们拥有所谓的原型继承。这意味着在 JavaScript 中并没有所谓的抽象概念*类*。在 JavaScript 中，我们只有对象。我们继承一个对象的原型，该原型具有方法和数据，所有具有相同原型的对象共享，但它们都是实例化的实例。

当我们在 JavaScript 中谈论类语法时，我们指的是构造函数和我们添加到它们的原型的方法/数据的语法糖。另一种思考这种类型的继承的方式是注意到在 JavaScript 中没有抽象概念，只有具体对象。如果这看起来有点神秘或令人困惑，下面的代码应该澄清这些陈述的含义：

```js
const Item = funciton() {
    this.a = 1;
    this.b = 'this';
    this.c = function() {
        console.log('this is going to be a new function each time');
    }
}
Item.prototype.d = function() {
    console.log('this is on the prototype so it will only be here 
     once');
}
const item1 = new Item();
const item2 = new Item();

item1.c === item2.c; //false
item1.d === item2.d; //true

const item3 = new (Object.getPrototypeOf(item1)).constructor();
item3.d === item2.d ;//true
Object.getPrototypeOf(item1).constructor === Item; //true
```

通过这个例子，我们展示了一些东西。首先，这是创建构造函数的旧方法。构造函数是设置作用域和在实例化时直接可用于对象的所有函数的函数。在这种情况下，我们已经将`a`、`b`和`c`作为`Item`构造函数的实例变量。其次，我们已经向项目的原型添加了一些内容。当我们在构造函数的原型上声明某些内容时，我们使所有该构造函数的实例都可以使用它。

从这里，我们声明了两个基于`Item`构造函数的项目。这意味着它们将分别获得`a`、`b`和`c`变量的单独实例，但它们将共享函数`d`。我们可以在接下来的两个语句中看到这一点。这展示了如果我们直接添加一些内容到构造函数的`this`作用域中，它将创建该项目的全新实例，但如果我们将一些内容放在原型上，所有项目都将共享它。

最后，我们可以看到`item3`是一个新的`Item`，但我们通过迂回的方式到达了构造函数。一些浏览器支持项目上的`__proto__`属性，但这个函数应该在所有浏览器中都可用。我们获取原型并注意到有一个构造函数。这正是我们在顶部声明的完全相同的函数，因此我们能够利用它来创建一个新的项目。我们可以看到它也在与其他项目相同的原型上，并且原型上的构造函数与我们声明的`item`变量完全相同。

所有这些都应该展示的是 JavaScript 纯粹由对象构成。没有其他语言中真正的类等抽象类型。如果我们利用新的语法，最好理解我们所做的只是利用语法糖来做我们以前可以用原型做的事情。也就是说，下一个例子将展示完全相同的行为，但一个是老式的基于原型的，另一个是利用新的类语法：

```js
class newItem {
    constructor() {
        this.c = function() {
            console.log('this is going to be a new function each time!);
        }
    }
    a = '1';
    b = 'this';
    d() {
        console.log('this is on the prototype so it will only be here 
         once');
    }
}
const newItem1 = new newItem();
const newItem2 = new newItem();

newItem1.c === newItem2.c //false
newItem1.d === newItem2.d //true

newItem === Object.getPrototypeOf(newItem1).constructor; //true
```

通过这个例子，我们可以看到在创建与原型版本之前相同的对象时，我们获得了一些更清晰的语法。构造函数与我们声明`Item`为函数时是一样的。我们可以传入任何参数并在这里进行设置。类中的一个有趣之处是我们能够在类内部创建实例变量，就像我们在原型示例中在`this`上声明它们一样。我们还可以看到`d`的声明放在了原型上。我们将在下面探索类语法的更多方面，但花些时间并玩弄这两段代码。当我们试图编写高性能代码时，理解 JavaScript 是基于原型的将会极大地帮助。

类中的公共变量是相当新的（Chrome 72）。如果我们无法访问更新的浏览器，我们将不得不使用 Babel 将我们的代码转译回浏览器能理解的版本。我们还将看看另一个只在 Chrome 中且是实验性的功能，但它应该在一年内传递到所有浏览器。

# 其他值得注意的功能

JavaScript 类为我们提供了许多很好的功能，使我们编写的代码清晰简洁，同时性能几乎与直接编写原型相同。一个很好的功能是包括静态成员变量和静态成员函数。

虽然没有太大的区别，但它确实允许我们编写无法被成员函数访问的函数（它们仍然可以被访问，但要困难得多），并且它可以为将实用函数分组到特定类中提供一个很好的工具。这里展示了静态函数和变量的一个例子：

```js
class newItem {
    static e() {
        console.log(this);
    }
    static f = 10;
}

newItem1.e() //TypeError
newItem.e() //give us the class
newItem.f //10
```

两个静态定义被添加到`newItem`类中，然后我们展示了可用的内容。通过函数`e`和静态变量`f`，我们可以看到它们不包括在我们从`newItem`创建的对象中，但当我们直接访问`newItem`时，我们可以访问它们。除此之外，我们可以看到静态函数内部的`this`指向类。静态成员和变量非常适合创建实用函数，甚至用于在 JavaScript 中创建单例模式。

如果我们想要以旧式风格创建相同的体验，它看起来会像下面这样：

```js
Item.e = function() {
    console.log(this);
}
Item.f = 10;
```

正如我们所看到的，我们必须在`Item`的第一个定义之后放置这些定义。这意味着我们必须相对小心地尝试将我们的类定义的所有代码分组在旧式风格中，而类语法允许我们将其全部放在一个组中。

除了静态变量和函数之外，我们还有一种为类中的变量编写 getter 和 setter 的简写方式。可以如下所示：

```js
get g() {
    return this._g;
}
set g(val) {
    if( typeof val !== 'string' ) {
        return;
    }
    this._g = val;
}
```

有了这个 getter 和 setter，当有人或某物尝试访问这个变量时，我们能够在这些函数内部做各种事情。就像我们设置了一个代理来监听变化一样，我们也可以在 getter 和 setter 中做类似的事情。我们还可以在这里设置日志记录。当我们想要访问某些东西时，这种语法非常好，只需使用属性名，而不是像`getG`和`setG`这样写。

最后，还有 Chrome 76 中出现的新私有变量。虽然这仍处于候选推荐阶段，但仍然会被讨论，因为它很可能会发挥作用。很多时候，我们希望尽可能多地公开信息。然而，有时我们希望利用内部变量来保存状态，或者一般情况下不被访问。在这方面，JavaScript 社区提出了`_`解决方案。任何带有`_`的东西都被视为私有变量。但是，用户仍然可以访问这些变量并对其进行操作。更糟糕的是，恶意用户可能会发现这些私有变量中的漏洞，并能够操纵系统。在旧系统中创建私有变量的一种技术是以下形式：

```js
const Public = (function() {
    let priv = 0;
    const Private = function() {}
    Private.prototype.add1 = function() {
        priv += 1;
    }
    Private.prototype.getVal = function() {
        return priv;
    }
    return Private;
})();
```

有了这个，除了实现者之外，没有人可以访问`priv`变量。这为我们提供了一个面向公众的系统，而不会访问私有变量。然而，这个系统仍然有一个问题：如果我们创建另一个`Public`对象，我们仍然会影响相同的`priv`变量。还有其他方法可以确保我们在创建新对象时获得新变量，但这些都是我们试图制定的系统的变通方法。相反，我们现在可以利用以下语法：

```js
class Public {
    #h = 10;
    get h() {
        return this.#h;
    }
}
```

井号的作用是表示这是一个私有变量。如果我们尝试从任何一个实例中访问它，它将返回未定义。这与 getter 和 setter 接口非常配合，因为我们将能够控制对变量的访问，甚至在需要时修改它们。

最后，再来看一下类的`extend`和`super`关键字。通过`extend`，我们可以对类进行扩展。让我们以`newItem`类为例，扩展其功能。这可能看起来像这样：

```js
class extendedNewItem extends newItem {
    constructor() {
        super();
        console.log(this.c());
    }
    get super_h() {
        return super.h;
    }
    static e() {
        super.e();
        console.log('this came from our extended class');
    }
}
const extended = new extendedNewItem();
```

在这个例子中发生了一些有趣的行为。首先，如果我们在扩展对象上运行`Object.getPrototypeOf`，我们会看到原型是我们所期望的`extendedNewItem`。现在，如果我们获取它的原型，我们会看到它是`newItem`。我们创建了一个原型链，就像许多内置对象一样。

其次，我们可以使用`super`从类内部访问父类的方法。这本质上是对我们父类的原型的引用。如果我们想要继续遍历所有的原型，我们不能链式调用它们。我们必须利用诸如`Object.getPrototypeOf`之类的东西。我们还可以通过检查我们的扩展对象来看到，我们得到了我们父类系统中保存的所有成员变量。

这使我们能够组合我们的类并创建基类或抽象类，这些类给我们一些定义好的行为，然后我们可以创建扩展类，给我们想要的特定行为。我们将在后面看到更多使用类和我们在这里讨论过的许多概念的代码，但请记住，类只是原型系统的语法糖，对此的良好理解将有助于理解 JavaScript 作为一种语言的工作原理。

关于 JavaScript 生态系统中的类接口有很多好东西，而且似乎还有一些其他很棒的想法可能会在未来出现，比如装饰器。随时关注**Mozilla 开发者网络**（**MDN**）页面，了解新的内容和可能在未来出现的内容总是一个好主意。现在我们将看一下模块以及它们在我们编写清晰快速代码的系统中是如何工作的。

一个很好的经验法则是不要扩展任何类超过一到两个级别。如果我们再继续下去，我们可能会开始创建一个维护的噩梦，除了潜在的对象变得过于沉重，包含了它们不需要的信息。提前考虑将始终是我们创建系统时的最佳选择，尽量减少我们类的影响是减少内存使用的一种方式。

# 模块

在 ECMAScript 2015 之前，我们没有加载代码的概念，除了使用脚本标签。我们提出了许多模块概念和库，比如**RequireJS**或**AMD**，但没有一个是内置到语言中的。随着模块的出现，我们现在有了一种创建高度模块化代码的方式，可以轻松地打包并导入到我们代码的其他部分。我们还在我们以前必须使用 IIFE 来获得这种行为的系统中获得了作用域锁。

首先，在我们开始使用模块之前，我们需要一个静态服务器来托管我们所有的内容。即使我们让 Chrome 允许访问本地文件系统，模块系统也会因为无法将它们作为文本/JavaScript 提供而感到不安。为了解决这个问题，我们可以安装 node 包`node-static`。我们将把这个包添加到一个静态目录中。我们可以运行以下命令：`npm install node-static`。一旦这个包下载完成到`static`目录中，我们可以从我们的存储库中的`Chapter03`文件夹中获取`app.js`文件并运行`node app.js`。这将启动静态服务器，并从`static`目录中的`files`目录中提供服务。然后我们可以把任何想要提供的文件放在那里，并且能够从我们的代码中获取到它们。

现在，我们可以编写一个基本的模块，如下所示，并将其保存为`lib.js`：

```js
export default function() {
    console.log('this is going to be our simple lib');
}
```

然后，我们可以从 HTML 文件中导入这个模块，如下所示：

```js
<script type="module'>
    import lib from './lib.js';
</script>
```

即使是这个基本的例子，我们也可以了解模块在浏览器中是如何工作的。首先，脚本的类型需要是一个模块。这告诉浏览器我们要加载模块，并且我们要把这段代码作为模块来处理。这给了我们几个好处。首先，当我们使用模块时，我们会自动进入严格模式。其次，我们在模块中自动获得了作用域。这意味着我们刚刚导入的`lib`不会作为全局变量可用。如果我们将内容加载为文本/JavaScript 并将变量放在全局路径上，那么我们将自动拥有它们；这就是为什么我们通常必须使用 IIFE。最后，我们得到了一个很好的语法来加载我们的 JavaScript 文件。我们仍然可以使用旧的方式加载一堆脚本，但我们也可以只导入基于模块的脚本。

接下来，我们可以看到模块本身使用了`export`和`default`关键字。`export`表示我们希望这个项在这个作用域或文件之外可用。现在我们可以在当前文件之外访问到这个项。`default`表示如果我们加载模块而没有定义我们想要的内容，我们将自动获得这个项。这可以在以下示例中看到：

```js
const exports = {
    this : 'that',
    that : 'this'
}

export { exports as Item };
```

首先，我们定义了一个名为`exports`的对象。这是我们要添加为导出项的对象。其次，我们将此项添加到一个`export`声明中，并且还重命名了它。这是模块的一个好处。在导出或导入的一侧，我们都可以重命名我们想要导出的项。现在，在我们的 HTML 文件中，我们会有如下声明：

```js
import { Item } from './lib.js';
```

如果我们在声明周围没有括号，我们将尝试引入默认导出。由于我们有花括号，它将在`lib.js`中查找名为`Item`的项目。如果找到，它将引入与之关联的代码。

现在，就像我们从导出列表中重命名导出一样，我们可以重命名导入。让我们继续将其更改为以下内容：

```js
import { Item as _item } from './lib.js';
```

现在我们可以像往常一样利用该项，但是作为变量`_item`而不是`Item`。这对于名称冲突非常有用。我们只能想出那么多变量名，所以，我们可以在加载它们时改变它们，而不是在单独的库中更改变量。

良好的样式约定是在顶部声明所有导入。然而，有一些用例可能需要动态加载模块，因为某种类型的用户交互或其他事件。如果发生这种情况，我们可以利用动态导入来实现这一点。这些看起来如下：

```js
document.querySelector('#loader').addEventListener('click', (ev) => {
    if(!('./lib2.js' in imported)) {
        import('./lib2.js')
        .then((module) => {
            imported['./lib2.js'] = module;
            module.default();
        });
    } else {
        imported['./lib2.js'].default();
    }
});
```

我们添加了一个按钮，当点击时，我们尝试将模块加载到我们的系统中。这不是在我们的系统中缓存模块的最佳方式，大多数浏览器也会为我们做一些缓存，但这种方式相当简单，展示了动态导入系统。导入函数基于承诺，因此我们尝试抓取它，如果成功，我们将其添加到导入的对象中。然后调用默认方法。我们可以访问模块为我们导出的任何项目，但这是最容易访问的项目之一。

看到 JavaScript 的发展是令人惊讶的。所有这些新功能给了我们以前必须依赖第三方的能力。关于 DOM 的变化也是如此。我们现在将看看这些变化。

# 使用 DOM

文档对象模型（DOM）并不总是最容易使用的技术。我们有古老的过时 API，大多数时候，它们在不同浏览器之间无法对齐。但是，在过去的几年里，我们已经得到了一些很好的 API 来做以下事情：轻松获取元素，构建内存层次结构以进行快速附加，并使用 DOM 阴影进行模板。所有这些都导致了一个丰富的环境，可以对底层节点进行更改，并创建许多丰富的前端，而无需使用 jQuery 等库。在接下来的几节中，我们将看到如何使用这些新 API 有所帮助。

# 查询选择器

在拥有这个 API 之前（或者我们试图尽可能跨浏览器），我们依赖于诸如`getElementById`或`getElementsByClassName`之类的系统。每个都提供了一种我们可以获取 DOM 元素的方式，如下例所示：

```js
<p>This is a paragraph element</p>
<ul id="main">
    <li class="hidden">1</li>
    <li class="hidden">2</li>
    <li>3</li>
    <li class="hidden">4</li>
    <li>5</li>
</ul>
<script type="module">
    const main = document.getElementById('main');
    const hidden = document.getElementsByClassName('hidden');
</script>
```

这个旧 API 和新的`querySelector`和`querySelectorAll`之间的一个区别是，旧 API 将 DOM 节点集合实现为`HTMLCollection`，而新 API 将它们实现为`NodeList`。虽然这可能看起来不是一个重大的区别，但`NodeList`API 确实给了我们一个已经内置到系统中的`forEach`。否则，我们将不得不将这两个集合都更改为常规的 DOM 节点数组。在新 API 中实现的前面的示例如下：

```js
const main = document.querySelector('#main');
const hidden = document.querySelectorAll('.hidden');
```

当我们想要开始向我们的选择过程添加其他功能时，这变得更加美好。

假设我们现在有一些输入，并且我们想获取所有文本类型的输入。在旧 API 中会是什么样子？如果需要，我们可以给它们都附加一个类，但这会污染我们对类的使用，可能不是处理这些信息的最佳方式。

我们可以通过利用旧 API 方法之一来获取这些数据，然后检查这些元素是否将输入属性设置为`text`。这可能看起来像下面这样：

```js
const allTextInput = Array.from(document.getElementsByTagName('input'))
    .filter(item => item.getAttribute('type') === "text");
```

但是现在我们有了一定程度的冗长，这是不需要的。相反，我们可以通过使用 CSS 选择器来获取它们，使用选择器 API 如下：

```js
const alsoTextInput = doucment.querySelectorAll('input[type="text"]');
```

这意味着我们应该能够利用 CSS 语法访问任何 DOM 节点，就像 jQuery 一样。我们甚至可以从另一个元素开始，这样我们就不必解析整个 DOM，就像这样：

```js
const hidden = document.querySelector('#main').querySelectorAll('.hidden');
```

选择器 API 的另一个好处是，如果我们不使用正确的 CSS 选择器，它将抛出错误。这为我们提供了系统为我们运行检查的额外好处。虽然新的选择器 API 已经存在，但由于需要包括 Internet Explorer 在支持的 Web 浏览器中，它并没有被广泛使用。强烈建议开始使用新的选择器 API，因为它不那么冗长，我们能够做的事情比旧系统多得多。

jQuery 是一个库，它为我们提供了比基本系统更好的 API。jQuery 支持的大多数更改现在已经过时，许多我们已经谈论过的新的 web API 正在接管。对于大多数新应用程序，它们将不再需要使用 jQuery。

# 文档片段

我们在之前的章节中已经看到了这些，但是触及它们是很好的。文档片段是可重用的容器，我们可以在其中创建 DOM 层次结构，并一次性附加所有这些节点。这导致更快的绘制时间和更少的重绘。

以下示例展示了两种使用直接 DOM 添加和片段添加的方式附加一系列列表元素：

```js
const num = 10000;
const container = document.querySelector('#add');
for(let i = 0; i < num; i++) {
    const temp = document.createElement('li');
    temp.textContent = `item ${i}`;
    container.appendChild(temp);
}
while(container.firstChild) {
    container.removeChild(container.firstChild);
}
const fragment = document.createDocumentFragment();
for(let i = 0; i < num; i++) {
    const temp = document.createElement('li');
    temp.textContent = `item ${i}`;
    fragment.appendChild(temp);
}
container.appendChild(fragment);
```

虽然这两者之间的时间很短，但发生的重绘次数并非如此。在我们的第一个示例中，每次直接向文档添加元素时，文档都会重绘，而我们的第二个示例只会重绘一次 DOM。这就是文档片段的好处；它使向 DOM 添加变得简单，同时只使用最少的重绘。

# Shadow DOM

阴影 DOM 通常与模板和 Web 组件配对使用，但也可以单独使用。阴影 DOM 允许我们封装我们应用程序的特定部分的标记和样式。如果我们想要页面的某个部分具有特定的样式，但不希望其传播到页面的其他部分，这是很好的。

我们可以通过利用其 API 轻松地使用阴影 DOM，如下所示：

```js
const shadow = document.querySelector('#shadowHolder').attachShadow({mode : 'open'});
const style = document.createElement('style');
style.textContent = `<left out to shorten code snippet>`;
const frag = document.createDocumentFragment();
const header = document.createElement('h1');
const par = document.createElement('p');
header.textContent = 'this is a header';
par.textContent = 'Here is some text inside of a paragraph element. It is going to get the styles we outlined above';

frag.appendChild(header);
frag.appendChild(par);
shadow.appendChild(style);
shadow.appendChild(frag);
```

首先，我们将阴影 DOM 附加到一个元素上，这里是我们的`shadowHolder`元素。有一个模式选项，它允许我们说是否可以在阴影上下文之外通过 JavaScript 访问内容，但已经发现我们可以轻松地规避这一点，因此建议保持它开放。接下来，我们创建一些元素，其中一个是一些样式属性。然后，我们将这些附加到一个文档片段，最后附加到阴影根。

搞定所有这些之后，我们可以看到并注意到我们的阴影 DOM 受到了放在其中的样式属性的影响，而不是放在我们主文档顶部的样式属性。如果我们在文档顶部放置一个我们的阴影样式没有的样式会发生什么？它仍然不会受到影响。有了这个，我们现在能够创建可以单独样式化的组件，而无需使用类。这将我们带到 DOM 的最后一个主题之一。

# Web 组件

Web 组件 API 允许我们创建具有定义行为的自定义元素，仅利用浏览器 API。这与诸如 Bootstrap 甚至 Vue 之类的框架不同，因为我们能够利用浏览器中存在的所有技术。

Chrome 和 Firefox 都支持所有这些 API。Safari 支持其中大部分，如果这是我们想要支持的浏览器，我们只能利用其中的一些 API。Edge 不支持 Web 组件 API，但随着它转向 Chromium 基础，我们将看到另一个能够利用这项技术的浏览器。

让我们创建一个基本的`tooltip`元素。首先，我们需要在我们的类中扩展基本的`HTMLElement`。然后，我们需要附加一些属性，以允许我们放置元素并给我们需要使用的文本。最后，我们需要注册这个组件到我们的系统中，以确保它识别我们的自定义元素。以下代码创建了这个自定义元素（修改自[`developer.mozilla.org/en-US/docs/Web/Web_Components/Using_custom_elements`](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_custom_elements)）：

```js
class Tooltip extends HTMLElement {
    constructor() {
        super();
        this.text = this.getAttribute('text');
        this.type = this.getAttribute('type');
        this.typeMap = new Map(Object.entries({
            'success' : "&#x2714",
            'error' : "&#x2716",
            'info' : "&#x2755",
            'default' : "&#x2709"
        }));

        this.shadow = this.attachShadow({mode : 'open'});
        const container = document.createElement('span');
        container.classList.add('wrapper');
        container.classList.add('hidden');
        const type = document.createElement('span');
        type.id = 'icon';
        const el = document.createElement('span');
        el.id = 'main';
        const style = document.createElement('style');
        el.textContent = this.text;
        type.innerHTML = this.getType(this.type);

        style.innerText = `<left out>`
        this.shadow.append(style);
        this.shadow.append(container);
        container.append(type);
        contianer.append(el);
    }
    update() {
        const x = this.getAttribute('x');
        const y = this.getAttribute('y');
        const type = this.getAttribute('type');
        const text = this.getAttribute('text');
        const show = this.getAttribute('show');
        const wrapper = this.shadow.querySelector('.wrapper');
        if( show === "true" ) {
            wrapper.classList.remove('hidden');
        } else {
            wrapper.classList.add('hidden');
        }
        this.shadow.querySelector('#icon').innerHTML = this.getType(type);
        this.shadow.querySelector('#main').innerText = text;
        wrapper.style.left = `${x}px`;
        wrapper.style.top = `${y}px`;
    }
    getType(type) {
        return type ?
            this.typeMap.has(type) ?
                this.typeMap.get(type) :
                this.typeMap.get('default') :
            this.typeMap.get('default');
    }
    connectCallback() {
        this.update(this);
    }
    attributeChangedCallback(name, oldValue, newValue) {
        this.update(this);
    }
    static get observedAttributes() {
        return ['x', 'y', 'type', 'text', 'show'];
    }
}

customElements.define('our-tooltip', Tooltip);
```

首先，我们有一个属性列表，我们将使用它们来样式化和定位我们的`tooltip`。它们分别称为`x`、`y`、`type`、`text`和`show`。接下来，我们创建了一个基于表情符号的文本映射，这样我们就可以利用图标而不需要引入一个完整的库。然后我们在一个阴影容器内设置了可重用的对象。我们还将阴影根放在对象上，这样我们就可以轻松访问它。`update`方法将在我们的元素第一次创建时触发，并在属性的任何后续更改时触发。我们可以在最后三个函数中看到这一点。`connectedCallback`将在我们被附加到 DOM 时触发。`attributeChangedCallback`将提醒我们发生了任何属性更改。这与代理 API 非常相似。最后一部分让我们的对象知道我们特别关心哪些属性，这种情况下是`x`、`y`、`type`、`text`和`show`。最后，我们使用`customElements.define`方法注册我们的自定义组件，给它一个名称和我们想要在创建这些对象时运行的类。

现在，如果我们创建我们的`tooltip`，我们可以利用这些不同的属性来制作一个可重用的`tooltip`系统，甚至是警报。以下代码演示了这一点：

```js
<our-tooltip show="true" x="100" y="100" icon="success" text="here is our tooltip"></our-tooltip>
```

我们应该看到一个浮动框，上面有一个复选标记和文本“这是我们的提示”。通过利用 Web 组件 API 中的模板系统，我们可以使这个`tooltip`更容易阅读。

# 模板

现在，我们有一个不错的可重用的`tooltip`元素，但是我们的样式标签中也有相当多的代码，它完全由模板化的字符串组成。最好的办法是，如果我们可以把这个语义标记放在别的地方，并把执行逻辑放在我们的 Web 组件中，就像现在一样。这就是模板发挥作用的地方。`<template>`元素不会显示在页面上，但我们仍然可以通过给它一个 ID 来很容易地获取它。因此，重构我们当前的代码的一种方式是这样的：

```js
<template id="tooltip">
    <style>
        /* left out */
    </style>
    <span class="wrapper hidden" x="0" y="0" type="default" show="false">
        <span id="icon">&#2709</span>
        <span id="main">This is some default text</span>
    </span>
</template>
```

我们的 JavaScript 类构造函数现在应该是这样的：

```js
constructor() {
    super();
    this.type = this.getAttribute('type');
    this.typeMap = // same as before
    const template = document.querySelector('#tooltip').content;
    this.shadow = this.attachShadow({mode : 'open'});
    this.shadow.appendChild(template.cloneNode(true));
}
```

这样更容易阅读，更容易理解。现在我们获取我们的模板并获取它的内容。我们创建一个`shadow`对象并附加我们的模板。我们需要确保克隆我们的模板节点，否则我们将在我们决定创建的所有元素之间共享相同的引用！你会注意到的一件事是，我们现在无法通过属性来控制文本。虽然看到这种行为很有趣，但我们真的希望把这些信息留给我们的`tooltip`的创建者。我们可以通过`<slot>`元素来实现这一点。

插槽给了我们一个区域，我们可以在那个位置放置 HTML。我们可以利用这一点，让`tooltip`的用户放入他们想要的标记。我们可以给他们一个看起来像下面这样的模板：

```js
<span class="wrapper hidden" x="0" y="0" type="default" show="false">
    <span id="icon">&#2709</span>
    <span id="main"><slot name="main_text">This is default text</slot></span>
</span>
```

我们的实现可能如下所示：

```js
<our-tooltip show="true" x="100" y="100" type="success">
    <span slot="main_text">That was a successful operation!</span>
</our-tooltip>
```

正如我们所看到的，阴影 DOM 的使用，以及浏览器中的 Web 组件和模板系统，使我们能够创建丰富的元素，而无需外部库，如 Bootstrap 或 Foundation。

我们可能仍然需要这些库来提供一些基本的样式，但我们不应该像过去那样需要它们。最理想的情况是，我们可以编写所有自己的组件和样式，而不需要利用外部库。但是，由于这些系统相对较新，如果我们无法控制用户的使用，我们可能会陷入填充的困境。

# 理解 Fetch API

在 Fetch API 之前，我们必须利用`XMLHttpRequest`系统。要创建一个服务器数据请求，我们必须编写类似以下的内容：

```js
const oldReq = new XMLHttpRequest();
oldReq.addEventListener('load', function(ev) {
    document.querySelector('#content').innerHTML = 
     JSON.stringify(ev.target.response);
});
oldReq.open('GET', 'http://localhost:8081/sample');
oldReq.setRequestHeader('Accept', 'application/json');
oldReq.responseType = 'json';
oldReq.send();
```

首先，您会注意到对象类型被称为`XMLHttpRequest`。原因是由于谁发明了它以及背后的原因。微软最初开发了这种技术，用于 Outlook Web Access 产品。最初，他们来回传输 XML 文档，因此他们为其构建的对象命名。一旦其他浏览器供应商，主要是 Mozilla，采用了它，他们决定保留名称，即使其目的已经从仅发送 XML 文档转变为从服务器发送到客户端的任何类型的响应。

其次，我们向对象添加了一个事件监听器。由于这是一个普通对象而不是基于 promise 的，我们以`addEventListener`方法的老式方式添加监听器。这意味着一旦它被使用，我们也会清理事件监听器。接下来，我们打开请求，传入我们想要发送的方法和发送的位置。然后我们可以设置一堆请求头（在这里特别指定我们想要的应用程序/JSON 数据，并将`responseType`设置为`json`，以便浏览器正确转换）。最后，我们发送请求。

一旦我们获得响应，我们的事件将触发，我们可以从事件的目标中检索响应。一旦我们开始发布数据，情况可能会变得更加繁琐。这就是 jQuery 的`$.ajax`和类似方法的原因。它使得与`XMLHttpRequest`对象一起工作变得更加容易。那么从 Fetch API 的角度来看，这种响应是什么样子的呢？这个完全相同的请求可以如下所示：

```js
fetch('http://localhost:8081/sample')
.then((res) => res.json())
.then((res) => {
    document.querySelector('#content').innerHTML = JSON.stringify(res);
});
```

我们可以看到这样阅读和理解起来要容易得多。首先，我们设置我们要访问的 URL。如果我们在`fetch`调用中不传递操作，它将自动假定我们正在创建一个`GET`请求。接下来，我们获取响应，并确保以`json`格式获取它。响应将始终作为*promise*返回（稍后会详细介绍），因此我们希望将其转换为我们想要的格式，即`json`。从这里，我们得到了最终的对象，我们可以将其设置为我们内容的`innerHTML`。从这两个基本对象的示例中，我们可以看到 Fetch API 几乎具有与`XMLHttpRequest`相同的功能，但它的格式更容易理解，我们可以轻松地使用 API。

# Promises

正如我们在之前的`fetch`示例中看到的，我们利用了一个叫做 promise 的东西。简单地说，promise 就是一个我们将来会需要的值，而返回给我们的是一个合同，声明了“我会在以后把它交给你”。Promise 是基于回调的概念。如果我们看一个可能包装在`XMLHttpRequest`周围的回调的例子，我们可以看到它是如何作为一个 promise 运行的：

```js
const makeRequest = function(loc, success, failure) {
    const oldReq = new XMLHttpRequest();
    oldReq.addEventListener('load', function(ev) {
        if( ev.target.status === 200 ) {
            success(ev.target.response);
        } else {
            failure(ev.target.response);
        }
    }, { once : true });
    oldReq.open('GET', loc);
    oldReq.setRequestHeader('Accept', 'application/json');
    oldReq.responseType = 'json';
    oldReq.send();
}
```

通过这样，我们几乎可以得到与 promise 相同的功能，但是利用回调或我们想要在发生某事时运行的函数。回调系统的问题是被称为回调地狱。这是高度异步代码总是有回调的想法，这意味着如果我们想要利用它，我们将会有一个美妙的回调树视图。这看起来像下面这样：

```js
const fakeFetchRequest(url, (res) => {
    res.json((final) => {
        document.querySelector('#content').innerHTML = 
         JSON.stringify(final);
    });
});
```

这个虚构的`fetch`版本是如果`fetch`的 API 不是基于 promise 的。首先，我们会传入我们的 URL。我们还需要为响应返回时提供一个回调。然后，我们需要将该响应传递给`json`方法，该方法还需要一个回调来将响应数据转换为`json`。最后，我们会得到结果并将其放入我们的 DOM。

正如我们所看到的，回调可能会导致很多问题。相反，我们有了 promise。promise 在创建时需要一个参数，即一个具有两个参数（resolve 和 reject）的函数。有了这些，我们可以通过`resolve`函数向调用者返回成功，或者通过`reject`函数报错。这将允许我们通过`then`调用和`catch`调用将这些 promise 链接在一起，就像我们在`fetch`示例中看到的那样。

然而，这也可能导致另一个问题。我们可能会得到一长串 promise，看起来比回调好一些，但并不明显。然后我们有了`async`/`await`系统。我们可以使用`await`来利用响应，而不是不断地用`then`链接 promise。然后我们可以将我们的`fetch`调用转换成以下形式：

```js
(async function() {
    const res = await fetch('http://localhost:8081/sample');
    const final = await res.json();
    document.querySelector('#content').innerHTML = JSON.stringify(final);
})();
```

函数前的`async`描述符告诉我们这是一个`async`函数。如果没有这个描述符，我们就无法使用`await`。接下来，我们可以直接使用`await`函数，而不是将`then`函数链接在一起。结果就是原本会被包装在我们的`resolve`函数中的内容。现在，我们有了一个非常易读的东西。

我们需要小心`async`/`await`系统。它确实会等待，所以如果我们将其放在主线程上，或者没有将其包装在其他东西中，它可能会阻塞主线程，导致我们无法继续执行。此外，如果我们有一堆任务需要同时运行，我们可以利用`Promise.all()`，而不是一个接一个地等待它们（使我们的代码变成顺序执行）。这允许我们将一堆 promise 放在一起，并允许它们异步运行。一旦它们都返回，我们就可以继续执行。

`async`/`await`系统的一个好处是它实际上可能比使用通用 promise 更快。许多浏览器已经围绕这些特定的关键字添加了优化，因此我们应该尽可能地利用它们。

之前已经提到过，但浏览器供应商不断改进他们对 ECMAScript 标准的实现。这意味着新技术一开始会比较慢，但一旦被广泛使用或得到所有供应商的认可，它们就会开始优化，并通常比其对应的技术更快。在可能的情况下，利用浏览器供应商提供给我们的新技术！

# 回到 fetch

现在我们已经看到了`fetch`请求的样子，我们应该看一下如何获取底层的可读流。`fetch`系统已经添加了很多功能，其中两个是管道和流。这可以在许多最近的 Web API 中看到，可以观察到浏览器供应商已经注意到了 Node.js 如何利用流。

如前一章所述，流是一种一次处理数据块的方式。它还确保我们不必一次性获取整个有效负载，而是可以逐步构建有效负载。这意味着如果我们需要转换数据，我们可以在数据块到达时即时进行转换。这也意味着我们可以处理不常见的数据类型，比如 JSON 和纯文本。

我们将编写一个基本示例，演示`TransformStream`如何对输入进行简单的 ROT13 编码（ROT13 是一种非常基本的编码器，它将我们得到的第 13 个字母替换原来的字母）。稍后我们将更详细地介绍流（这些将是 Node.js 版本，但概念相对类似）。示例大致如下：

```js
class Rot13Transform {
    constructor() {
    }
    async transform(chunk, controller) {
        const _chunk = await chunk;
        const _newChunk = _chunk.map((item) => ((item - 65 + 13) % 26) + 
         65);
        controller.enqueue(_newChunk);
        return;
    }
}

fetch('http://localhost:8081/rot')
.then(response => response.body)
.then(res => res.pipeThrough(new TransformStream(new Rot13Transform())))
.then(res => new Response(res))
.then(response => response.text())
.then(final => document.querySelector('#content').innerHTML = final)
.catch(err => console.error(err));
```

让我们将这个例子分解成实际的`TransformStream`，然后是利用它的代码。首先，我们创建一个类，用于容纳我们的旋转代码。然后，我们需要一个叫做`transform`的方法，它接受两个参数，块和控制器。块是我们将要获取的数据。

请记住，这不会一次性获取所有数据，因此如果我们需要构建对象或类似的东西，我们需要为前面的数据创建一个可能的临时存储位置，如果当前块没有给我们想要的所有内容。在我们的情况下，我们只是在底层字节上运行一个旋转方案，因此我们不需要有一个临时持有者。

接下来，控制器是流控制和声明数据是否准备从中读取（一个可读或转换流）或写入（一个可写流）的基础系统。接下来，我们等待一些数据并将其放入一个临时变量中。然后，我们对每个字节运行一个简单的映射表达式，将它们向右旋转 13 次，然后对 26 取模。

ASCII 约定将所有大写字符从 65 开始。这就是这里涉及一些数学的原因，因为我们试图首先得到 0 到 26 之间的数字，进行操作，然后将其移回正常的 ASCII 范围内。

一旦我们旋转了输入，我们就会将其排队在控制器上。这意味着数据已准备好从另一个流中读取。接下来，我们可以看一系列发生的承诺。首先，我们获取我们的数据。然后，我们通过获取其主体从`fetch`请求中获取底层的`ReadableStream`。然后，我们利用一个叫做`pipeThrough`的方法。管道机制会自动为我们处理流量控制，因此在处理流时会让我们的生活变得更加轻松。

流控制对于使流工作至关重要。它基本上告诉其他流，如果我们被堵住了，就不要再发送数据，或者我们可以继续接收数据。如果没有这种机制，我们将不断地不得不控制我们的流，当我们只想专注于我们想要合并的逻辑时，这可能会是一个真正的痛苦。

我们将数据传输到一个新的`TransformStream`中，该流采用我们的旋转逻辑。现在，这将把响应中的所有数据传输到我们的转换代码中，并确保它经过转换后输出。然后，我们将我们的`ReadableStream`包装在一个新的`Response`中，这样我们就可以像处理`fetch`请求的任何其他`Response`对象一样处理它。然后，我们像处理普通文本一样获取数据并将其放入我们的 DOM 中。

正如我们所看到的，这个例子展示了我们可以通过流系统做很多很酷的事情。虽然 DOM API 仍在变化中，但这些概念与 Node.js 中的流接口类似。它还展示了我们如何可能为更复杂的二进制类型编写解码器，这些类型可能通过网络传输，比如 smile 格式。

# 停止 fetch 请求

在进行请求时，我们可能想要执行的一个操作是停止它们。这可能是出于多种原因，比如：

+   首先，如果我们在后台进行请求，并且让用户更新`POST`请求的参数，我们可能希望停止当前请求，并让他们发出新的请求。

+   其次，一个请求可能花费太长时间，我们希望确保停止请求，而不是挂起应用程序或使其进入未知状态。

+   最后，我们可能有一个设置好的缓存机制，一旦我们完成缓存大量数据，我们希望使用它。如果发生这种情况，我们希望停止任何待处理的请求，并将其切换到该来源。

任何这些原因都是停止请求的好理由，现在我们有一个可以做到这一点的 API。`AbortController`系统允许我们停止这些请求。发生的情况是`AbortController`有一个`signal`属性。我们将这个`signal`附加到`fetch`请求上，当我们调用`abort`方法时，它告诉`fetch`请求我们不希望它继续进行请求。这非常简单和直观。以下是一个例子：

```js
(async function() {
    const controller = new AbortController();
    const signal = controller.signal;
    document.querySelector('#stop').addEventListener('click', (ev) => {
        controller.abort();
    });
    try {
        const res = await fetch('http://localhost:8081/longload', 
         {signal});
        const final = await res.text();
        document.querySelector('#content').innerHTML = final;
    } catch(e) {
        console.error('failed to download', e);
    }
})();
```

正如我们所看到的，我们已经建立了一个`AbortController`系统并获取了它的`signal`属性。然后我们设置了一个按钮，当点击时，将运行`abort`方法。接下来，我们看到了典型的`fetch`请求，但在选项中，我们传递了`signal`。现在，当我们点击按钮时，我们会看到请求因 DOM 错误而停止。我们还看到了一些关于`async`/`await`的错误处理。`async`/`await`可以利用基本的`try-catch`语句来捕获错误，这只是`async`/`await`API 使代码比回调和基于 promise 的版本更可读的另一种方式。

这是另一个实验性的 API，将来很可能会有变化。但是，我们在`XMLHttpRequest`中也有了相同类型的想法，因此 Fetch API 也会得到它是有道理的。请注意，MDN 网站是获取有关浏览器支持和任何我们已经讨论过并将在未来章节讨论的实验性 API 的最新信息的最佳地方。

`fetch`和 promise 系统是从服务器获取数据并展示处理异步流量的新方式。虽然我们过去必须利用回调和一些看起来很糟糕的对象，但现在我们有了一个非常容易使用的简洁的 API。尽管 API 的部分正在变化，但请注意，这些系统很可能会以某种方式存在。

# 总结

在本章中，我们看到了过去 5 年来浏览器环境发生了多少变化。通过新的 API 增强了我们编写代码的方式，通过 DOM API 使我们能够编写具有内置控件的丰富 UI，我们现在能够尽可能地使用原生应用。这包括获取外部数据的使用，以及新的异步 API，如 promises 和`async`/`await`系统。

在下一章中，我们将看到一个专注于输出原生 JavaScript 并为我们提供无运行时应用环境的库。当我们讨论节点和工作线程时，我们还将把大部分现代 API 整合到本书的其余部分中。玩弄这些系统，并熟悉它们，因为我们才刚刚开始。
