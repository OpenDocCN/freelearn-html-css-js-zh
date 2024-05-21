# 第五章：ES6 迭代器和生成器

到目前为止，我们已经讨论了 JavaScript 的语言构造，而没有看任何特定的语言版本。然而，在本章中，我们将主要关注 ES6 中引入的一些语言特性。这些特性对你编写 JavaScript 代码有很大的影响。它们不仅显著改进了语言，还为 JavaScript 程序员提供了迄今为止无法使用的几个函数式编程构造。

在本章中，我们将看一下 ES6 中新引入的迭代器和生成器。有了这些知识，我们将继续详细了解增强的集合构造。

# For...of 循环

`for...of`循环是在 ES6 中引入的，与可迭代对象和迭代器构造一起。这个新的循环构造替代了 ES5 中的`for...in`和`for...each`循环构造。由于`for...of`循环支持迭代协议，它可以用于内置对象，比如数组、字符串、映射、集合等，以及可迭代的自定义对象。考虑下面的代码片段作为一个例子：

```js
    const iter = ['a', 'b']; 
    for (const i of iter) { 
      console.log(i); 
    } 
    "a" 
    "b" 

```

`for...of`循环适用于可迭代对象和内置对象，比如数组是可迭代的。如果你注意到，我们在定义循环变量时使用的是`const`而不是`var`。这是一个好的做法，因为当你使用`const`时，会创建一个新的变量绑定和存储空间。当你不打算在块内修改循环变量的值时，应该在`for...of`循环中使用`const`而不是`var`声明。

其他集合也支持`for...of`循环。例如，字符串是 Unicode 字符序列，`for...of`循环也可以正常工作：

```js
    for (let c of "String"){ 
      console.log(c); 
    } 
    //"s" "t" "r" "i" "n" "g" 

```

`for...in`和`for...of`循环的主要区别在于`for...in`循环遍历对象的所有可枚举属性。`for...of`循环有一个特定的目的，那就是根据对象定义的可迭代协议来遵循迭代行为。

# 迭代器和可迭代对象

ES6 引入了一种新的迭代数据的机制。遍历数据列表并对其进行操作是一种非常常见的操作。ES6 增强了迭代构造。这个变化涉及到两个主要概念——迭代器和可迭代对象。

## 迭代器

JavaScript 迭代器是一个公开`next()`方法的对象。这个方法以一个对象的形式返回集合中的下一个项，这个对象有两个属性——`done`和`value`。在下面的例子中，我们将通过公开`next()`方法从数组中返回一个迭代器：

```js
    //Take an array and return an iterator 
    function iter(array){ 
      var nextId= 0; 
      return { 
        next: function() { 
          if(nextId < array.length) { 
            return {value: array[nextId++], done: false}; 
          } else { 
            return {done: true}; 
          } 
        } 
      } 
    } 
    var it = iter(['Hello', 'Iterators']); 
    console.log(it.next().value); // 'Hello' 
    console.log(it.next().value); // 'Iterators' 
    console.log(it.next().done);  // true 

```

在上面的例子中，我们会不断通过`next()`方法返回数组中的元素，直到没有元素可返回为止，这时我们将返回`done`为`true`，表示迭代没有更多的值。通过`next()`方法重复访问迭代器中的元素。

## 可迭代对象

可迭代对象是定义了其迭代行为或内部迭代的对象。这样的对象可以在 ES6 中引入的`for...of`循环中使用。内置类型，比如数组和字符串，定义了默认的迭代行为。为了使对象可迭代，它必须实现`@@iterator`方法，也就是说对象必须有一个以`'Symbol.iterator'`为键的属性。

如果一个对象实现了一个键为`'Symbol.iterator'`的方法，那么它就变成了可迭代对象。这个方法必须通过`next()`方法返回一个迭代器。让我们看下面的例子来澄清这一点：

```js
    //An iterable object 
    //1\. Has a method with key has 'Symbol.iterator' 
    //2\. This method returns an iterator via method 'next' 
    let iter = { 
      0: 'Hello', 
      1: 'World of ', 
      2: 'Iterators', 
      length: 3, 
      [Symbol.iterator]() { 
        let index = 0; 
        return { 
          next: () => { 
            let value = this[index]; 
            let done = index >= this.length; 
            index++; 
            return { value, done }; 
          } 
        }; 
      } 
    }; 
    for (let i of iter) { 
      console.log(i);  
    } 
    "Hello" 
    "World of " 
    "Iterators" 

```

让我们把这个例子分解成更小的部分。我们正在创建一个可迭代对象。我们将使用对象字面语法创建一个`iter`对象，这是我们已经熟悉的。这个对象的一个特殊方面是`[Symbol.iterator]`方法。这个方法的定义使用了计算属性和 ES6 的简写方法定义语法，这是我们在上一章已经讨论过的。由于这个对象包含了`[Symbol.iterator]`方法，这个对象是可迭代的，或者说它遵循可迭代协议。这个方法还通过暴露`next()`方法返回迭代器对象，定义了迭代行为。现在这个对象可以与`for...of`循环一起使用。

# 生成器

与迭代器和可迭代对象密切相关，生成器是 ES6 中最受关注的功能之一。生成器函数返回一个生成器对象；这个术语起初听起来很令人困惑。当你编写一个函数时，你也本能地理解它的行为-函数开始执行，逐行执行，并在执行最后一行时结束执行。一旦函数以这种方式被线性执行，随后跟随函数的代码将被执行。

在支持多线程的语言中，这种执行流程可以被中断，部分完成的任务可以在不同的线程、进程和通道之间共享。JavaScript 是单线程的，你目前不需要处理多线程的挑战。

然而，生成器函数可以被暂停和稍后恢复。这里的重要思想是，生成器函数选择暂停自己，它不能被任何外部代码暂停。在执行期间，函数使用`yield`关键字来暂停。一旦生成器函数被暂停，它只能被函数外的代码恢复。

你可以暂停和恢复生成器函数多次。使用生成器函数，一个常见的模式是编写无限循环，并在需要时暂停和恢复它们。这样做有利有弊，但这种模式已经变得流行起来。

另一个重要的理解点是，生成器函数还允许双向消息传递，进出函数。每当你使用`yield`关键字暂停函数时，消息就会从生成器函数中发送出去，当函数恢复时，消息就会传回生成器函数。

让我们看下面的例子来澄清生成器函数的工作原理：

```js
    function* generatorFunc() { 
      console.log('1'); //-----------> A 
      yield;            //-----------> B 
      console.log('2'); //-----------> C 
    } 
    const generatorObj = generatorFunc(); 
    console.log(generatorObj.next());   
    //"1" 
    //Object { 
    // "done": false, 
    // "value": undefined 
    //} 

```

这是一个非常简单的生成器函数。然而，有几个有趣的方面需要仔细理解。

首先，注意关键字 function 后面紧跟着一个星号`*`，这是表示该函数是一个生成器函数的语法。在函数名之前紧跟着星号也是可以的。以下两种声明都是有效的：

```js
    function *f(){ }  
    function* f(){ } 

```

在函数内部，真正的魔力在于`yield`关键字。当遇到`yield`关键字时，函数会暂停自己。在我们继续之前，让我们看看函数是如何被调用的：

```js
    const generatorObj = generatorFunc(); 
    generatorObj.next();  //"1" 

```

当我们调用生成器函数时，它不像普通函数一样被执行，而是返回一个生成器对象。你可以使用这个生成器对象来控制生成器函数的执行。生成器对象上的`next()`方法会恢复函数的执行。

当我们第一次调用`next()`时，执行会一直进行到函数的第一行（标记为'A'），当遇到`yield`关键字时暂停。如果我们再次调用`next()`函数，它将从上次暂停执行的地方继续执行到下一行：

```js
    console.log(generatorObj.next());   
    //"2" 
    //Object { 
    // "done": true, 
    // "value": undefined 
    //} 

```

一旦整个函数体被执行，对生成器对象的任何`next()`调用都没有效果。我们谈到生成器函数允许双向消息传递。这是如何工作的？在前面的例子中，你可以看到每当我们恢复生成器函数时，我们会收到一个包含两个值`done`和`value`的对象；在我们的例子中，我们收到的值是`undefined`。这是因为我们没有用`yield`关键字返回任何值。当你用`yield`关键字返回一个值时，调用函数会接收到它。考虑以下例子：

```js
    function* logger() { 
      console.log('start') 
      console.log(yield) 
      console.log(yield) 
      console.log(yield) 
      return('end') 
    } 

    var genObj = logger(); 

    // the first call of next executes from the 
      start of the function until the first yield statement 
    console.log(genObj.next())         
    // "start", Object {"done": false,"value": undefined} 
    console.log(genObj.next('Save'))   
    // "Save", Object {"done": false,"value": undefined} 
    console.log(genObj.next('Our'))    
    // "Our", Object {"done": false,"value": undefined} 
    console.log(genObj.next('Souls'))  
    // "Souls",Object {"done": true,"value": "end"} 

```

让我们一步一步地追踪这个例子的执行流程。生成器函数有三个暂停或 yield。我们可以通过编写以下代码来创建生成器对象：

```js
    var genObj = logger(); 

```

我们将通过调用`next`方法开始执行生成器函数；这个方法开始执行，直到第一个`yield`。如果你注意到，在第一次调用中我们没有向`next()`方法传递任何值。这个`next()`方法的目的只是启动生成器函数。我们将再次调用`next()`方法，但这次传递一个`"Save"`值作为参数。当函数执行恢复时，`yield`接收到这个值，我们可以在控制台上看到打印出的值：

```js
    "Save", Object {"done": false,"value": undefined} 

```

我们将再次使用两个不同的值调用`next()`方法，输出与前面的代码类似。当我们最后一次调用`next()`方法时，执行结束，生成器函数将返回一个`end`值给调用代码。在执行结束时，你会看到`done`设置为`true`，`value`赋值为函数返回的值，即`end`：

```js
    "Souls",Object {"done": true,"value": "end"} 

```

重要的是要注意，第一个`next()`方法的目的是启动生成器函数的执行-它将我们带到第一个`yield`关键字，因此，传递给第一个`next()`方法的任何值都将被忽略。

到目前为止的讨论表明，生成器对象符合迭代器的约定：

```js
    function* logger() { 
      yield 'a' 
      yield 'b' 
    } 
    var genObj = logger(); 
    //the generator object is built using generator function 
    console.log(typeof genObj[Symbol.iterator] === 'function')    //true 
    // it is an iterable 
    console.log(typeof genObj.next === 'function') //true 
    // and an iterator (has a next() method) 
    console.log(genObj[Symbol.iterator]() === genObj) //true 

```

这个例子证实了生成器函数也符合可迭代的约定。

## 迭代生成器

生成器是迭代器，像所有支持可迭代的 ES6 构造一样，它们可以用于迭代生成器。

第一种方法是使用`for...of`循环，如下面的代码所示：

```js
    function* logger() { 
      yield 'a' 
      yield 'b' 
    } 
    for (const i of logger()) { 
      console.log(i) 
    } 
    //"a" "b" 

```

我们在这里没有创建生成器对象。`For...of`循环支持可迭代对象，生成器自然适用于这个循环。

扩展运算符可以用于将可迭代对象转换为数组。考虑以下示例：

```js
    function* logger() { 
      yield 'a' 
      yield 'b' 
    } 
    const arr = [...logger()] 
    console.log(arr) //["a","b"] 

```

最后，你可以使用解构语法与生成器，如下所示：

```js
    function* logger() { 
      yield 'a' 
      yield 'b' 
    } 
    const [x,y] = logger() 
    console.log(x,y) //"a" "b" 

```

生成器在异步编程中扮演着重要的角色。接下来，我们将看一下 ES6 中的异步编程和 Promise。JavaScript 和 Node.js 提供了一个很好的环境来编写异步程序。生成器可以帮助你编写协作式的多任务函数。

# 集合

ES6 引入了四种数据结构-`Map`、`WeakMap`、`Set`和`WeakSet`。与 Python 和 Ruby 等其他语言相比，JavaScript 的标准库非常薄弱，无法支持哈希或 Map 数据结构或字典。人们发明了一些方法来模拟`Map`的行为，通过将字符串键与对象进行映射。这些方法会产生一些副作用。语言对这些数据结构的支持是非常必要的。

ES6 支持标准的字典数据结构；我们将在下一节更详细地了解这些内容。

## Map

`Map`允许将任意值作为`key`。`keys`映射到值。Map 允许快速访问值。让我们看一些 Map 的例子：

```js
    const m = new Map(); //Creates an empty Map 
    m.set('first', 1);   //Set a value associated with a key 
    console.log(m.get('first'));  //Get a value using the key 

```

我们将使用构造函数创建一个空的`Map`。你可以使用`set()`方法向`Map`添加一个条目，将键与值关联起来，并覆盖具有相同键的任何现有条目。它的对应方法`get()`获取与键关联的值，如果在映射中没有这样的条目，则返回`undefined`。

Map 还有其他可用的辅助方法，如下所示：

```js
    console.log(m.has('first')); //Checks for existence of a key 
    //true 
    m.delete('first'); 
    console.log(m.has('first')); //false 

    m.set('foo', 1); 
    m.set('bar', 0); 

    console.log(m.size); //2 
    m.clear(); //clears the entire map 
    console.log(m.size); //0 

```

您也可以使用以下可迭代的*[key, value]*对来创建`Map`：

```js
    const m2 = new Map([ 
        [ 1, 'one' ], 
        [ 2, 'two' ], 
        [ 3, 'three' ], 
    ]); 

```

您可以使用链式`set()`方法来获得紧凑的语法，如下所示：

```js
    const m3 = new Map().set(1, 'one').set(2, 'two').set(3, 'three'); 

```

我们可以使用任何值作为键。对于对象，键只能是字符串，但是对于集合，这种限制被移除了。我们也可以使用对象作为键，尽管这种用法并不是很流行：

```js
    const obj = {} 
    const m2 = new Map([ 
      [ 1, 'one' ], 
      [ "two", 'two' ], 
      [ obj, 'three' ], 
    ]); 
    console.log(m2.has(obj)); //true 

```

### 遍历 Map

要记住的一件重要的事情是，对于 Map 来说，顺序很重要。Map 保留了添加元素的顺序。

有三种可迭代对象可用于遍历`Map`，即`keys`，`values`和`entries`。

`keys()`方法返回`Map`键的可迭代对象，如下所示：

```js
    const m = new Map([ 
      [ 1, 'one' ], 
      [ 2, 'two' ], 
      [ 3, 'three' ], 
    ]); 
    for (const k of m.keys()){ 
      console.log(k);  
    } 
    //1 2 3 

```

同样，`values()`方法返回`Map`值的可迭代对象，如下面的示例所示：

```js
    for (const v of m.values()){ 
      console.log(v);  
    } 
    //"one" 
    //"two" 
    //"three" 

```

`entries()`方法以*[key,value]*对的形式返回`Map`的条目，如下面的代码所示：

```js
    for (const entry of m.entries()) { 
      console.log(entry[0], entry[1]); 
    } 
    //1 "one" 
    //2 "two" 
    //3 "three" 

```

您可以使用解构来使其更简洁，如下所示：

```js
    for (const [key, value] of m.entries()) { 
      console.log(key, value); 
    } 
    //1 "one" 
    //2 "two" 
    //3 "three" 

```

更简洁的是：

```js
    for (const [key, value] of m) { 
      console.log(key, value); 
    } 
    //1 "one" 
    //2 "two" 
    //3 "three" 

```

### 将 Map 转换为数组

如果要将`Map`转换为数组，则扩展运算符(`...`)非常方便：

```js
    const m = new Map([ 
      [ 1, 'one' ], 
      [ 2, 'two' ], 
      [ 3, 'three' ], 
    ]); 
    const keys = [...m.keys()] 
    console.log(keys) 
    //Array [ 
    //1, 
    //2, 
    //3 
    //] 

```

由于 Map 是可迭代的，您可以使用扩展运算符将整个`Map`转换为数组：

```js
    const m = new Map([ 
      [ 1, 'one' ], 
      [ 2, 'two' ], 
      [ 3, 'three' ], 
    ]); 
    const arr = [...m] 
    console.log(arr) 
    //Array [ 
    //[1,"one"], 
    //[2,"two"], 
    //[3,"three"] 
    //] 

```

## Set

`Set`是一个值的集合。您可以向其中添加和删除值。尽管这听起来与数组类似，但集合不允许相同的值出现两次。`Set`中的值可以是任何类型。到目前为止，您一定在想这与数组有何不同？`Set`旨在快速执行一项操作-成员测试。相对而言，数组在这方面较慢。`Set`操作类似于`Map`操作：

```js
    const s = new Set(); 
    s.add('first'); 
    s.has('first'); // true 
    s.delete('first'); //true 
    s.has('first'); //false 

```

与 Map 类似，您可以通过迭代器创建一个`Set`：

```js
    const colors = new Set(['red', white, 'blue']); 

```

当您向`Set`添加一个值，并且该值已经存在时，不会发生任何事情。同样，如果您从`Set`中删除一个值，并且该值一开始不存在，也不会发生任何事情。没有办法捕获这种情况。

## WeakMap 和 WeakSet

`WeakMap`和`WeakSet`的 API 与`Map`和`Set`类似，但受到限制，并且它们大部分工作方式与它们的强大对应物相似。不过，也有一些区别，如下所示：

+   `WeakMap`仅支持`new`，`has()`，`get()`，`set()`和`delete()`方法

+   `WeakSet`仅支持`new`，`has()`，`add()`和`delete()`方法

+   `WeakMap`的键必须是对象

+   `WeakSet`的值必须是对象

+   您无法迭代`WeakMap`；您可以通过其键访问值的唯一方式

+   您无法迭代`WeakSet`

+   您无法清除`WeakMap`或`WeakSet`

首先让我们了解`WeakMap`。`Map`和`WeakMap`之间的区别在于`WeakMap`允许自身被垃圾回收。`WeakMap`中的键是弱引用的。当垃圾回收器进行引用计数时（一种查看所有存活引用的技术），`WeakMap`的键不会被计算，并且在可能的情况下会被垃圾回收。

当您无法控制在 Map 中保存的对象的生命周期时，`WeakMaps`非常有用。使用`WeakMaps`时不需要担心内存泄漏，因为即使对象的生命周期很长，它们也不会占用内存。

`WeakSet`也适用相同的实现细节。然而，由于无法迭代`WeakSet`，因此`WeakSet`的用例并不多。

# 总结

在本章中，我们详细研究了 ES6 生成器。生成器是 ES6 最受期待的功能之一。暂停和恢复函数执行的能力打开了许多关于协作编程的可能性。生成器的主要优势在于它们提供了单线程、同步代码风格，同时隐藏了异步的本质。这使我们更容易以非常自然的方式表达程序步骤/语句的流程，而无需同时导航异步语法和陷阱。我们通过生成器实现了关注点的分离。

生成器与迭代器和可迭代对象的约定密切相关。这些是 ES6 中受欢迎的添加，显著增强了语言提供的数据结构。迭代器提供了一种简单的方法来返回（可能是无界的）值序列。`@@iterator`符号用于为对象定义默认迭代器，使其成为可迭代对象。

迭代器最重要的用例是当我们想在消耗可迭代对象的结构中使用它时，比如`for...of`循环。在本章中，我们还研究了 ES6 中引入的新循环结构`for...of`。`for...of`与许多原生对象一起工作，因为它们定义了默认的`@@iterator`方法。我们还研究了 ES6 集合的新添加，如 Map、Set、WeakMap 和 Weak Set。这些集合有额外的迭代器方法-`.entries()`、`.values()`和`.keys()`。

下一章将详细研究 JavaScript 原型。
