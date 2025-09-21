# 使用迭代器

ES8 及更早版本引入了新的对象接口和循环迭代。新迭代协议的添加为 JavaScript 打开了算法和能力的新世界。我们将从介绍符号和 `Symbol` 对象的各种属性开始本章。我们还将学习嵌套函数调用如何创建执行栈，它们的影响，以及如何优化它们的性能和内存使用。

虽然符号是迭代器的一个独立主题，但我们仍将在本章中涵盖符号，因为要实现迭代协议，你需要使用符号。

本章我们将涵盖：

+   使用符号作为对象属性键

+   在对象中实现迭代协议

+   创建和使用 `generator` 对象

+   使用 `for…of` 循环进行迭代

+   尾调用优化

# 符号 – 原始数据类型

**符号** 是一种在 ES6 中首次引入的原始类型。符号是一个唯一且不可变的值。以下是一个示例，展示了如何创建一个符号：

```js
const s = Symbol();
```

符号没有字面形式；因此，我们需要使用 `Symbol()` 函数来创建一个符号。每次调用 `Symbol()` 函数时，它都会返回一个唯一的符号。

`Symbol()` 函数接受一个可选的字符串参数，表示符号的描述。符号的描述可用于调试，但不能用来访问符号本身。具有相同描述的两个符号完全不等于彼此。以下是一个示例来演示这一点：

```js
let s1 = Symbol("My Symbol");
let s2 = Symbol("My Symbol");
console.log(s1 === s2); // Outputs false
```

从前面的示例中，我们也可以说符号是一个类似于字符串的值，它不会与其他任何值冲突。

# `typeof` 操作符

`typeof` 操作符用于确定特定变量/常量持有的值的类型。对于 `Symbol`，`typeof` 输出 `symbol`。以下是一个示例来演示相同的内容：

```js
const s = Symbol();
console.log(typeof s); //Outputs "symbol"
```

使用 `typeof` 操作符是唯一识别变量是否持有符号的方法。

# 新操作符

你不能将 `new` 操作符应用于 `Symbol()` 函数。`Symbol()` 函数会检测它是否被用作构造函数，如果是，则抛出异常。

下面是一个示例来演示这一点：

```js
try {
  let s = new Symbol(); //"TypeError" exception
}
catch(e) {
  console.log(e.message); //Output "Symbol is not a constructor"
}
```

但 JavaScript 引擎可以使用 `Symbol()` 函数作为构造函数在内部包装一个符号。因此，`s` 将等于对象(s)。

从 ES6 开始引入的所有原始类型都不允许手动调用它们的构造函数。

# 使用符号作为对象属性键

直到 ES5，JavaScript 对象属性键必须是字符串类型。但自从 ES6 以来，JavaScript 对象属性键可以是字符串或符号。以下是一个示例，演示了如何使用符号作为对象属性键：

```js
let obj = null;
let s1 = null;
(function(){
 let s2 = Symbol();
 s1 = s2;
 obj = {[s2]: "mySymbol"}
 console.log(obj[s2]);
 console.log(obj[s2] == obj[s1]);
})();
console.log(obj[s1]);
```

输出是：

```js
mySymbol
true
mySymbol
```

从前面的代码中，你可以看到，为了使用符号创建或检索属性键，你需要使用 `[]` 符号。我们在讨论第二章中的计算属性名时看到了 `[]` 符号，第二章，*了解你的库*。

要访问一个符号属性键，我们需要符号。在先前的例子中，`s1` 和 `s2` 都持有相同的符号值。

# `Object.getOwnPropertySymbols()` 方法

`Object.getOwnPropertyNames()` 方法无法检索符号属性。因此，ES6 引入了 `Object.getOwnPropertySymbols()` 来检索对象的一组符号属性。以下是一个示例来演示这一点：

```js
let obj = {a: 12};
let s1 = Symbol("mySymbol");
let s2 = Symbol("mySymbol");
Object.defineProperty(obj, s1, {
enumerable: false
});
obj[s2] = "";
console.log(Object.getOwnPropertySymbols(obj));
```

输出如下：

```js
Symbol(mySymbol),Symbol(mySymbol)
```

从先前的例子中，你可以看到 `Object.getOwnPropertySymbols()` 方法也可以检索不可枚举的符号属性。

`in` 操作符可以在对象中找到符号属性，而 `for…in` 循环和 `Object.getOwnPropertyNames()` 由于向后兼容性的原因，不能在对象中找到符号属性。

# `Symbol.for(string)` 方法

`Symbol` 对象维护了一个键/值对的注册表，其中键是符号描述，值是符号。每次我们使用 `Symbol.for()` 方法创建符号时，它都会被添加到注册表中，并且该方法返回符号。如果我们尝试使用已存在的描述来创建符号，那么将检索现有的符号。

使用 `Symbol.for()` 方法而不是 `Symbol()` 方法创建符号的优势在于，在使用 `Symbol.for()` 方法时，你不必担心使符号在全局范围内可用，因为它始终在全局范围内可用。以下是一个示例来演示这一点：

```js
let obj = {};
(function(){
 let s1 = Symbol("name");
 obj[s1] = "Eden";
})();
//obj[s1] cannot be accessed here
(function(){
 let s2 = Symbol.for("age");
 obj[s2] = 27;
})();
console.log(obj[Symbol.for("age")]); //Output "27"
```

# 已知符号

除了你自己的符号外，ES6 还提供了一套内置的符号集合，称为已知符号。以下是一个属性列表，引用了一些重要的内置符号：

+   `Symbol.iterator`

+   `Symbol.match`

+   `Symbol.search`

+   `Symbol.replace`

+   `Symbol.split`

+   `Symbol.hasInstanceSymbol.species`

+   `Symbol.unscopables`

+   `Symbol.isContcatSpreadable`

+   `Symbol.toPrimitive`

你将在本书的各个章节中遇到这些符号的使用。

当在文本中引用已知的符号时，我们通常使用 `@@` 符号来作为前缀。例如，`Symbol.iterator` 符号被称为 `@@iterator` 方法。这样做是为了使在文本中引用这些符号更加容易。

# 迭代协议

迭代协议是一组规则，对象需要遵循这些规则来实现接口。当使用此协议时，循环或构造可以遍历对象的一组值。

JavaScript 有两个名为 **iterator** 和 **iterable** 的迭代协议。

# 迭代协议

任何实现了迭代器协议的对象都被称为**迭代器**。根据迭代器协议，一个对象需要提供一个`next()`方法，该方法返回一组项目序列中的下一个项目。

这里有一个例子来演示这一点：

```js
let obj = {
 array: [1, 2, 3, 4, 5],
 nextIndex: 0,
 next: function() {
         return this.nextIndex < this.array.length ? {value: this.array[this.nextIndex++], done: false} : {done: true}
       }
};
console.log(obj.next().value);
console.log(obj.next().value);
console.log(obj.next().value);
console.log(obj.next().value);
console.log(obj.next().value);
console.log(obj.next().done);
```

输出如下：

```js
1
2
3
4
5
true
```

如果你仔细观察，你会发现`obj`对象内部的`next`方法如下：

```js
return this.nextIndex < this.array.length ? {value: this.array[this.nextIndex++], done: false} : {done: true}
```

这可以写成以下形式：

```js
if(this.nextIndex < this.array.length) {
  this.nextIndex++;
  return { value: this.array[this.nextIndex], done: false }
} else {
  return { done: true }
}
```

这清楚地告诉我们，如果对象`obj`中存在新元素，我们将增加`nextIndex`并从对象`obj`中发送`array`的下一个元素。当没有元素剩下时，我们返回`{ done: true }`。

# 可迭代协议

任何实现了**可迭代协议**的对象都被称为可迭代对象。根据可迭代协议，一个对象需要提供一个`@@iterator`方法；也就是说，它必须有一个`Symbol.iterator`符号作为属性键。`@@iterator`方法必须返回一个迭代器对象。

这里有一个例子来演示这一点：

```js
let obj = {
  array: [1, 2, 3, 4, 5],
  nextIndex: 0,
  [Symbol.iterator]: function(){
    return {
     array: this.array,
     nextIndex: this.nextIndex,
     next: function(){
       return this.nextIndex < this.array.length ?
          {value: this.array[this.nextIndex++], done: false} :
          {done: true};
     }
    }
  }
};
let iterable = obj[Symbol.iterator]()
console.log(iterable.next().value);
console.log(iterable.next().value);
console.log(iterable.next().value);
console.log(iterable.next().value);
console.log(iterable.next().value);
console.log(iterable.next().done);
```

输出如下：

```js
1
2
3
4
5
true
```

这一切看起来都很不错，但这样做有什么用呢？

前两个代码块展示了如何在自身上实现可迭代协议。然而，像**数组**这样的东西自带可迭代协议（即，它们的`__proto__`链实现了`Symbol.iterator`方法），这是默认实现的，从而节省了开发者的时间。让我们来看一个例子：

```js
const arr = [1, 2]
const iterator = arr[Symbol.iterator](); // returns you an iterator
console.log(iterator.next())
console.log(iterator.next())
console.log(iterator.next())
```

根据我们迄今为止所学到的，你认为输出应该是什么？

输出如下：

```js
{ value: 1, done: false }
{ value: 2, done: false }
{ value: undefined, done: true }
```

现在我们来看看生成器，它们或多或少与迭代器相似。

# 生成器函数

一个`generator`是一个普通的函数，但它不是返回一个单一值，而是逐个返回多个值。调用`generator`函数不会立即执行其主体，而是返回一个新的`generator`对象实例（即，一个实现了可迭代和迭代器协议的对象）。

每个`generator`对象都持有`generator`函数的新执行上下文。当我们执行`generator`对象的`next()`方法时，它会在遇到`yield`关键字之前执行`generator`函数的主体。它返回产生的值并暂停函数。当再次调用`next()`方法时，它继续执行并返回下一个产生的值。当`generator`函数没有产生任何值时，`done`属性为`true`。

`generator`函数是用`function*`表达式编写的。这里有一个例子来演示这一点：

```js
function* generator_function(){ 
  yield 1; 
  yield 2; 
  yield 3;
  yield 4; 
  yield 5;
}
let generator = generator_function();
console.log(generator.next().value);
console.log(generator.next().value);
console.log(generator.next().value);
console.log(generator.next().value);
console.log(generator.next().value);
console.log(generator.next().done);

generator = generator_function();

let iterable = generator[Symbol.iterator]();
console.log(iterable.next().value);
console.log(iterable.next().value);
console.log(iterable.next().value);
console.log(iterable.next().value);
console.log(iterable.next().value);
console.log(iterable.next().done);
```

输出如下：

```js
1
2
3
4
5
true
1
2
3
4
5
true
```

在`yield`关键字后面有一个表达式。该表达式的值是通过可迭代协议由`generator`函数返回的。如果我们省略这个表达式，那么返回`undefined`。这个表达式的值就是我们所说的，产生的值。

我们也可以向 `next()` 方法传递一个可选参数。这个参数成为 `yield` 语句返回的值，其中 `generator` 函数当前处于暂停状态。以下是一个示例来演示这一点：

```js
function* generator_function(){ 
  const a = yield 12;
  const b = yield a + 1; 
  const c = yield b + 2; 
  yield c + 3; // Final Line
}
const generator = generator_function();
console.log(generator.next().value);
console.log(generator.next(5).value);
console.log(generator.next(11).value);
console.log(generator.next(78).value);
console.log(generator.next().done);
```

输出如下：

```js
12
6
13
81
true
```

这里是对这个输出的解释：

1.  在第一次调用 `generator.next()` 时，调用 `yield 12` 并返回值 `12`。

1.  在第二次调用 `generator.next(5)` 时，之前的 `yield`（存储在 `const a` 中）获取传递的值（即 `5`），然后是第二个 `yield`（`a + 1`）。然后，调用 `yield 5 + 1` 并返回值 `6`（注意：这里的 `a` 不是 `12`）。

1.  在第三次调用 `generator.next(11)` 时，`const b` 变为 `11`，然后因为它是 `11 + 2` 的和，所以返回 `13`。

1.  这一直持续到最后一个过程，即直到示例中提到的 `Final Line` 行。

1.  由于 `yield` 最终返回一个值和其 `done` 状态，在执行 `yield c + 3` 之后，显然没有值可以 `yield`。因此，返回的值是 `undefined`，`done` 是 `true`。

# 返回(value)方法

你可以在 `generator` 函数完成所有值 `yield` 之前，使用 `generator` 对象的 `return()` 方法随时结束它。`return()` 方法接受一个可选参数，表示要返回的最终值。

这里有一个示例来演示这一点：

```js
function* generator_function(){ 
 yield 1; 
 yield 2; 
 yield 3;
}
const generator = generator_function();
console.log(generator.next().value);
console.log(generator.return(22).value);
console.log(generator.next().done);
```

输出如下：

```js
1
22
true
```

# `throw(exception)` 方法

你可以使用 `generator` 对象的 `throw()` 方法在 `generator` 函数内部手动触发异常。你必须向 `throw()` 方法传递你想要抛出的异常。以下是一个示例来演示这一点：

```js
function* generator_function(){ 
try { 
 yield 1; 
} catch(e) { 
 console.log("1st Exception"); 
} 
try { 
 yield 2; 
} catch(e) { 
 console.log("2nd Exception"); 
}
}
const generator = generator_function();
console.log(generator.next().value);
console.log(generator.throw("exception string").value);
console.log(generator.throw("exception string").done);
```

输出如下：

```js
1
1st Exception
2
2nd Exception
true
```

在前面的示例中，你可以看到异常是在函数最后暂停的地方抛出的。在异常被处理后，`throw()` 方法继续执行，并返回下一个 `yield` 的值。

# `yield*` 关键字

在 `generator` 函数内部，`yield*` 关键字将可迭代对象作为表达式并迭代它以 `yield` 其值。以下是一个示例来演示这一点：

```js
function* generator_function_1(){ 
 yield 2; 
 yield 3;
}
function* generator_function_2(){
 yield 1; 
 yield* generator_function_1(); 
 yield* [4, 5];
}
const generator = generator_function_2();
console.log(generator.next().value);
console.log(generator.next().value);
console.log(generator.next().value);
console.log(generator.next().value);
console.log(generator.next().value);
console.log(generator.next().done);
```

输出如下：

```js
1
2
3
4
5
true
```

# `for...of` 循环

到目前为止，我们一直使用 `next()` 方法迭代可迭代对象，这是一个繁琐的任务。ES6 引入了 `for...of` 循环来简化这个过程。

`for...of` 循环被引入来迭代可迭代对象的值。以下是一个示例来演示这一点：

```js
function* generator_function(){ 
 yield 1; 
 yield 2; 
 yield 3; 
 yield 4; 
 yield 5;
}
let arr = [1, 2, 3];
for(let value of generator_function()){ 
 console.log(value);
}
for(let value of arr){ 
 console.log(value);
}
```

输出如下：

```js
1
2
3
4
5
1
2
3
```

# 尾调用优化

每当进行函数调用时，在堆栈内存中创建一个执行栈来存储函数的变量。**尾调用优化**基本上意味着如果没有信息在代码执行序列的后续部分中需要该堆栈分配的信息，你可以重用分配的堆栈。

# 为什么需要尾调用优化？

当在另一个函数调用内部进行函数调用时，会为内部函数调用创建一个新的执行栈。然而，问题是内部函数执行栈会占用一些额外的内存——也就是说，它存储了一个额外的地址，表示当这个函数执行完毕后如何恢复执行。切换和创建执行栈也会消耗一些额外的 CPU 时间。当调用嵌套层级只有几百层时，这个问题并不明显，但当有数千层或更多嵌套层级时，这个问题就很明显了——即 JavaScript 引擎抛出 `RangeError: Maximum call stack size exceeded` 异常。你可能在创建递归函数时遇到过 `RangeError` 异常。

尾调用是在函数的末尾，通过 `return` 语句可选执行的函数调用。如果一个尾调用反复调用同一个函数，那么它被称为**尾递归**，这是递归的一个特殊情况。尾调用特殊之处在于，实际上有一种方法可以防止在尾调用时额外的 CPU 时间和内存使用，那就是重用外函数的栈，而不是创建新的执行栈。在尾调用时重用执行栈称为尾调用优化。

如果脚本以 `"use strict"` 模式编写，JavaScript 支持在特定浏览器中进行尾调用优化。让我们看看一个尾调用的例子：

```js
"use strict";
function _add(x, y){ 
    return x + y;
}
function add1(x, y){ 
    x = parseInt(x); 
    y = parseInt(y); //tail call 
    return _add(x, y);
}
function add2(x, y) {
    x = parseInt(x);
    y = parseInt(y);
    //not tail call
    return 0 + _add(x, y);
}
console.log(add1(1, '1')); //2
console.log(add2(1, '2')); //3
```

在这里，`add1()` 函数中的 `_add()` 调用是一个尾调用，因为它是 `add1()` 函数的最终操作。然而，`add2()` 函数中的 `_add()` 调用不是一个尾调用，因为它不是最终的操作；将 `0` 添加到 `_add()` 的结果是最终的操作。

在 `add1()` 函数中的 `_add()` 调用不会创建新的执行栈。相反，它重用了 `add1()` 函数的执行栈；换句话说，发生了尾调用优化。

尾调用优化规范目前**没有积极**开发，目前仅在 Safari 中实现。因此，你只能在 Safari 中使用 TCO。

# 将非尾调用转换为尾调用

由于尾调用得到了优化，因此你应尽可能使用尾调用，而不是非尾调用。你可以通过将非尾调用转换为尾调用来优化你的代码。

让我们看看一个类似的例子：

```js
"use strict";
function _add(x, y) {
    return x + y; 
}

function add(x, y) {
 x = parseInt(x);
 y = parseInt(y);
 const result = _add(x, y);
 return result;
}

console.log(add(1, '1'));
```

在之前的代码中，`_add()` 调用不是一个尾调用，因此创建了两个执行栈。我们可以这样将其转换为尾调用：

```js
function add(x, y){ 
    x = parseInt(x); 
    y = parseInt(y); 
    return _add(x, y);
}
```

在这里，我们省略了 `result` 变量的使用，而是将函数调用与 `return` 语句对齐。还有许多其他类似的策略可以将非尾调用转换为尾调用。

# 摘要

在本章中，我们学习了一种使用符号创建对象属性键的新方法。我们了解了迭代器和可迭代协议，并学习了如何在自定义对象中实现这些协议。然后，我们学习了如何使用`for…of`循环遍历可迭代对象。最后，我们通过学习尾调用及其优化来结束本章。

在下一章中，我们将学习如何使用 Promises 以及 ES8 中最近推出的 async/await 特性进行异步编程，这使得异步代码看起来更像是同步代码。让我们开始吧！
