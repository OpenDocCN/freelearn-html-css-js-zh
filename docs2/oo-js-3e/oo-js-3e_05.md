# 第五章. ES6 迭代器和生成器

到目前为止，我们讨论了 JavaScript 的语言结构，而没有查看任何特定的语言版本。然而，在本章中，我们将主要关注 ES6 中引入的一些语言特性。这些特性对编写 JavaScript 代码的方式产生了重大影响。它们不仅显著提高了语言，还提供了 JavaScript 程序员迄今为止无法使用的几个函数式编程结构。

在本章中，我们将探讨 ES6 中新引入的迭代器和生成器。有了这些知识，我们将详细研究增强的集合结构。

# For...of 循环

`for...of` 循环是在 ES6 中与可迭代和迭代器结构一起引入的。这个新的循环结构取代了 ES5 中的 `for...in` 和 `for...each` 循环结构。由于 `for...of` 循环支持迭代协议，它可以用于内置对象，如数组、字符串、映射、集合等，以及可迭代的自定义对象。以下代码片段可以作为示例：

```js
    const iter = ['a', 'b']; 
    for (const i of iter) { 
      console.log(i); 
    } 
    "a" 
    "b" 

```

`for...of` 循环与可迭代对象和内置对象（如数组）一起工作。如果你注意到，当我们定义循环变量时，我们使用 `const` 而不是 `var`。这是一个好习惯，因为当你使用 `const` 时，会创建一个新的变量，具有新的绑定和存储空间。当你不打算在块内部修改循环变量的值时，你应该在 `for...of` 循环中使用 `const` 而不是 `var` 声明。

其他集合也支持 `for...of` 循环。例如，由于字符串是一系列 Unicode 字符，`for...of` 循环可以正常工作：

```js
    for (let c of "String"){ 
      console.log(c); 
    } 
    //"s" "t" "r" "i" "n" "g" 

```

`for...in` 循环和 `for...of` 循环之间的主要区别在于，`for...in` 循环遍历对象的所有可枚举属性。`for...of` 循环有一个特定的目的，那就是根据对象如何定义可迭代协议来遵循迭代行为。

# 迭代器和可迭代对象

ES6 引入了一种新的遍历数据机制。遍历数据列表并进行操作是一个非常常见的操作。ES6 增强了迭代结构。这个变化涉及两个主要概念——迭代器和可迭代对象。

## 迭代器

JavaScript 迭代器是一个暴露 `next()` 方法的对象。此方法以具有两个属性的对象形式返回集合中的下一个项——`done` 和 `value`。在以下示例中，我们将通过暴露 `next()` 方法从一个数组中返回一个迭代器：

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

在前面的示例中，我们在数组中有元素时返回 `value` 和 `done`。当我们耗尽数组中的元素以返回时，我们将返回 `done` 作为 `true`，表示迭代没有更多的值。迭代器中的元素通过重复使用 `next()` 方法访问。

## 可迭代对象

可迭代对象是一个定义其迭代行为或内部迭代的对象。这些对象可以用在 ES6 中引入的`for...of`循环中。内置类型，如数组和字符串，定义了默认的迭代行为。为了使对象可迭代，它必须实现`@@iterator`方法，这意味着对象必须有一个以`'Symbol.iterator'`为键的属性。

如果一个对象实现了一个键为`'Symbol.iterator'`的方法，那么这个对象就变成了可迭代的。这个方法必须通过`next()`方法返回一个迭代器。让我们看看以下示例来澄清这一点：

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

让我们将这个例子分解成更小的部分。我们正在创建一个可迭代对象。我们将使用我们已熟悉的对象字面量语法创建一个`iter`对象。这个对象的一个特殊之处在于它有一个`[Symbol.iterator]`方法。这个方法定义使用了一种组合的计算属性和 ES6 简写方法定义语法，我们已经在上一章中讨论过。由于这个对象包含一个`[Symbol.iterator]`方法，这个对象是可迭代的，或者遵循可迭代协议。此方法还通过公开`next()`方法来返回定义迭代行为的迭代器对象。现在这个对象可以用`for...of`循环使用。

# 生成器

与迭代器和可迭代对象紧密相关，生成器是 ES6 中最受关注的特性之一。生成器函数返回一个生成器对象；这个术语一开始听起来可能有些令人困惑。当您编写一个函数时，您也会本能地理解其行为——函数从执行开始，逐行执行，直到最后一行执行完毕。一旦以这种方式线性执行函数，函数后面的代码就会执行。

在支持多线程的语言中，这种执行流程可以被中断，并且部分完成的任务可以在不同的线程、进程和通道之间共享。JavaScript 是单线程的，您目前不需要处理多线程的挑战。

然而，生成器函数可以在之后被暂停和恢复。这里的重要思想是生成器函数选择暂停自己，它不能被任何外部代码暂停。在执行过程中，函数使用`yield`关键字来暂停。一旦生成器函数被暂停，它只能通过函数外的代码来恢复。

您可以暂停和恢复生成器函数任意多次。使用生成器函数，一个流行的模式是编写无限循环，并在需要时暂停和恢复它们。这样做有优点也有缺点，但这种模式已经流行起来。

另一个需要理解的重要点是，生成器函数也允许双向消息传递，即进出生成器函数。每当您使用`yield`关键字暂停函数时，消息就会从生成器函数中发送出去，而当函数恢复时，消息会返回到生成器函数。

让我们通过以下示例来明确了解生成器函数是如何工作的：

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

首先，注意关键字`function`后面的一个星号`*`，这是表示函数是生成器函数的语法。也可以保留星号在函数名之前。以下两种声明都是有效的：

```js
    function *f(){ }  
    function* f(){ } 

```

在函数内部，真正的魔法围绕着`yield`关键字。当遇到`yield`关键字时，函数会自己暂停。在我们继续之前，让我们看看函数是如何被调用的：

```js
    const generatorObj = generatorFunc(); 
    generatorObj.next();  //"1" 

```

当我们调用生成器函数时，它不像正常函数那样执行，而是返回一个生成器对象。您可以使用这个生成器对象来控制生成器函数的执行。生成器对象上的`next()`方法会继续函数的执行。

当我们第一次调用`next()`时，执行会一直进行到函数的第一行（标记为'A'），并在遇到`yield`关键字时暂停。如果我们再次调用`next()`函数，它将从上次暂停的点继续执行：

```js
    console.log(generatorObj.next());   
    //"2" 
    //Object { 
    // "done": true, 
    // "value": undefined 
    //} 

```

一旦整个函数体执行完毕，对生成器对象上的`next()`的任何调用都没有效果。我们提到生成器函数允许双向消息传递。这是如何工作的？在前面的例子中，您可以看到每次我们恢复生成器函数时，我们都收到一个包含两个值`done`和`value`的对象；在我们的情况下，我们收到了`undefined`作为值。这是因为我们没有用`yield`关键字返回任何值。当你用`yield`关键字返回一个值时，调用函数会接收到它。考虑以下示例：

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

让我们逐步追踪这个示例的执行流程。生成器函数有三个暂停或`yield`点。我们可以通过以下代码行创建生成器对象：

```js
    var genObj = logger(); 

```

我们将通过调用`next`方法来开始生成器函数的执行；这个方法会从第一个`yield`开始执行。如果您注意的话，我们在第一次调用`next()`方法时没有传递任何值。这个`next()`方法的目的只是启动生成器函数。我们将再次调用`next()`方法，但这次我们传递一个`"Save"`值作为参数。这个值在函数执行恢复时被`yield`接收，并且我们可以看到这个值被打印在控制台上：

```js
    "Save", Object {"done": false,"value": undefined} 

```

我们将再次用两个不同的值调用`next()`方法，输出将与前面的代码类似。当我们最后一次调用`next()`方法时，执行结束，生成器函数将一个`end`值返回给调用代码。在执行结束时，您会看到`done`被设置为`true`，而`value`被分配了函数返回的值，即`end`：

```js
    "Souls",Object {"done": true,"value": "end"} 

```

重要的是要注意，第一个`next()`方法的目的在于启动生成器函数的执行——它带我们到第一个`yield`关键字，因此传递给第一个`next()`方法的任何值都被忽略。

到目前为止的讨论表明，生成器对象符合迭代器协议：

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

此示例确认生成器函数也符合可迭代协议。

## 遍历生成器

生成器是迭代器，并且像所有支持可迭代的 ES6 构造一样，它们可以用来遍历生成器。

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

我们在这里没有创建生成器对象。`for...of`循环自然支持可迭代对象，生成器也不例外。

扩展运算符可以将可迭代对象转换为数组。考虑以下示例：

```js
    function* logger() { 
      yield 'a' 
      yield 'b' 
    } 
    const arr = [...logger()] 
    console.log(arr) //["a","b"] 

```

最后，你可以使用解构语法与生成器一起使用，如下所示：

```js
    function* logger() { 
      yield 'a' 
      yield 'b' 
    } 
    const [x,y] = logger() 
    console.log(x,y) //"a" "b" 

```

生成器在异步编程中扮演着重要的角色。简而言之，我们将在 ES6 中查看异步编程和承诺的更多细节。JavaScript 和 Node.js 提供了一个编写异步程序的良好环境。生成器可以帮助你编写协作多任务函数。

# 集合

ES6 引入了四种数据结构-`Map`、`WeakMap`、`Set`和`WeakSet`。与 Python 和 Ruby 等其他语言相比，JavaScript 在支持哈希或 Map 数据结构或字典方面有一个非常薄弱的标准库。发明了几个黑客手段，通过将字符串键映射到对象来某种程度上实现`Map`的行为。这些黑客手段有副作用。对这些数据结构的语言支持非常迫切。

ES6 支持标准字典数据结构；我们将在下一节中查看这些细节。

## 映射

`Map`允许任意值作为`keys`。键映射到值。映射允许快速访问值。让我们看看一些映射的例子：

```js
    const m = new Map(); //Creates an empty Map 
    m.set('first', 1);   //Set a value associated with a key 
    console.log(m.get('first'));  //Get a value using the key 

```

我们将使用构造函数创建一个空的`Map`。你可以使用`set()`方法向`Map`中添加一个条目，将键与值关联起来，并覆盖任何具有相同键的现有条目。它的对应方法`get()`获取与键关联的值，如果映射中没有这样的条目，则返回`undefined`。

映射还有其他一些辅助方法可用，如下所示：

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

你可以使用以下可迭代*[键，值]*对创建一个`Map`：

```js
    const m2 = new Map([ 
        [ 1, 'one' ], 
        [ 2, 'two' ], 
        [ 3, 'three' ], 
    ]); 

```

你可以将`set()`方法链起来以获得紧凑的语法，如下所示：

```js
    const m3 = new Map().set(1, 'one').set(2, 'two').set(3, 'three'); 

```

我们可以使用任何值作为键。对于对象，键只能是字符串，但与集合相比，这种限制被移除了。我们也可以使用对象作为键，尽管这种用法并不常见：

```js
    const obj = {} 
    const m2 = new Map([ 
      [ 1, 'one' ], 
      [ "two", 'two' ], 
      [ obj, 'three' ], 
    ]); 
    console.log(m2.has(obj)); //true 

```

### 遍历映射

需要记住的一个重要事情是，对于映射来说，顺序很重要。映射保留了元素添加的顺序。

你可以使用三个可迭代对象来遍历一个`Map`，即`keys`、`values`和`entries`。

`keys()` 方法返回 `Map` 键的可迭代对象，如下所示：

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

同样，`values()` 方法返回 `Map` 值的可迭代对象，如下面的示例所示：

```js
    for (const v of m.values()){ 
      console.log(v);  
    } 
    //"one" 
    //"two" 
    //"three" 

```

`entries()` 方法返回 `Map` 的条目，以 `[key,value]` 对的形式，如下面的代码所示：

```js
    for (const entry of m.entries()) { 
      console.log(entry[0], entry[1]); 
    } 
    //1 "one" 
    //2 "two" 
    //3 "three" 

```

您可以使用解构来使代码更简洁，如下所示：

```js
    for (const [key, value] of m.entries()) { 
      console.log(key, value); 
    } 
    //1 "one" 
    //2 "two" 
    //3 "three" 

```

更加简洁的写法：

```js
    for (const [key, value] of m) { 
      console.log(key, value); 
    } 
    //1 "one" 
    //2 "two" 
    //3 "three" 

```

### 将映射转换为数组

如果您想将 `Map` 转换为数组，使用扩展运算符（`...`）会很有用：

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

由于映射是可迭代的，您可以使用扩展运算符将整个 `Map` 转换为数组：

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

## `Set`

`Set` 是一组值的集合。您可以从中添加和删除值。尽管这听起来与数组相似，但集合不允许重复的值。`Set` 中的值可以是任何类型。到目前为止，您可能想知道这与数组有什么不同？`Set` 是为了快速进行成员资格测试而设计的。在这一点上，数组相对较慢。`Set` 操作类似于 `Map` 操作：

```js
    const s = new Set(); 
    s.add('first'); 
    s.has('first'); // true 
    s.delete('first'); //true 
    s.has('first'); //false 

```

与映射类似，您可以通过迭代器创建一个 `Set`：

```js
    const colors = new Set(['red', white, 'blue']); 

```

当您向 `Set` 中添加一个值，而这个值已经存在时，不会发生任何操作。同样地，如果您从 `Set` 中删除一个值，而这个值最初就不存在，也不会发生任何操作。无法捕获这种场景。

## `WeakMap` 和 `WeakSet`

`WeakMap` 和 `WeakSet` 具有与 `Map` 和 `Set` 相似但受限的 API，并且它们的工作方式与它们的强对应物大致相同。尽管如此，也有一些差异，如下所述：

+   `WeakMap` 只支持 `new`、`has()`、`get()`、`set()` 和 `delete()` 方法

+   `WeakSet` 只支持 `new`、`has()`、`add()` 和 `delete()` 操作

+   `WeakMap` 的键必须是对象

+   `WeakSet` 的值必须是对象

+   您不能遍历 `WeakMap`；访问值的唯一方式是通过其键

+   您不能遍历 `WeakSet`

+   您不能清除 `WeakMap` 或 `WeakSet`

让我们先来了解 `WeakMap`。`Map` 和 `WeakMap` 之间的区别在于 `WeakMap` 允许自己被垃圾回收。`WeakMap` 中的键是弱引用。`WeakMap` 的键在垃圾回收器进行引用计数（查看所有活动引用的技术）时不会被计算，并且当可能时会被垃圾回收。

当您对存储在 `Map` 中的对象的生存周期没有控制权时，`WeakMap` 非常有用。使用 `WeakMap` 时，您不必担心内存泄漏，因为即使对象的生存周期很长，这些对象也不会占用内存。

同样的实现细节也适用于 `WeakSet`。然而，由于您不能遍历 `WeakSet`，因此 `WeakSet` 的用例并不多。

# 概述

在本章中，我们详细探讨了 ES6 生成器。生成器是 ES6 最期待的功能之一。暂停和恢复函数执行的能力为协作编程开辟了许多可能性。生成器的主要优势是它们提供了一种单线程、同步外观的代码风格，同时隐藏了异步的本质。这使得我们能够以非常自然的方式表达程序步骤/语句的流程，而无需同时导航异步语法和陷阱。正是由于这一点，我们使用生成器实现了关注点的分离。

生成器与迭代器和可迭代对象合约紧密协作。这些是 ES6 的受欢迎的补充，并显著增强了语言提供的数据结构。迭代器提供了一种简单的方法来返回一个（可能是无界的）值序列。`@@iterator`符号用于为对象定义默认迭代器，使它们成为可迭代的。

迭代器最重要的用例在于当我们想要在消耗可迭代对象的构造中（例如`for...of`循环）使用它时变得明显。在本章中，我们还探讨了 ES6 中引入的新循环构造`for...of`。`for...of`与许多原生对象一起工作，因为它们定义了默认的`@@iterator`方法。我们研究了 ES6 集合的新增内容，如-Maps、Sets、WeakMaps 和 Weak Sets。这些集合具有额外的迭代器方法-`.entries()`、`.values()`和`.keys()`。

下一章将详细探讨 JavaScript 原型。
