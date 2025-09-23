# 第三章。使用迭代器

ES6 引入了新的对象接口和循环用于迭代。新迭代协议的添加为 JavaScript 打开了算法和能力的新世界。我们将从介绍符号和`Symbol`对象的各个属性开始本章。我们还将学习嵌套函数调用的执行栈是如何创建的，它们的影响，以及如何优化它们的性能和内存使用。

尽管符号是迭代器的一个独立主题，但我们仍将在本章中介绍符号，因为要实现迭代协议，你需要使用符号。

在本章中，我们将涵盖：

+   使用符号作为对象属性键

+   在对象中实现迭代协议

+   创建和使用生成器对象

+   使用`for...of`循环进行迭代

+   尾调用优化

# ES6 符号

ES6 符号是 ES6 中引入的新原始类型。符号是一个唯一且不可变的值。以下是一个示例代码，展示了如何创建一个符号：

```js
var s = Symbol();
```

符号没有字面形式；因此，我们需要使用`Symbol()`函数来创建一个符号。每次调用`Symbol()`函数时，它都会返回一个唯一的符号。

`Symbol()`函数接受一个可选的字符串参数，表示符号的描述。符号的描述可用于调试，但不能用于访问符号本身。具有相同描述的两个符号完全不等于彼此。以下是一个示例来演示这一点：

```js
let s1 = window.Symbol("My Symbol");
let s2 = window.Symbol("My Symbol");

console.log(s1 === s2); //Output is "false"
```

从前面的示例中，我们也可以说，符号是一个类似字符串的值，它不会与其他任何值冲突。

## "typeof"运算符

当应用于包含符号的变量时，`typeof`运算符输出`"symbol"`。以下是一个示例来演示这一点：

```js
var s = Symbol();
console.log(typeof s); //Output "symbol"
```

使用`typeof`运算符是唯一识别变量是否包含符号的方法。

## "new"运算符

你不能在`Symbol()`函数上应用`new`运算符。`Symbol()`函数会检测它是否被用作构造函数，如果是，则抛出异常。以下是一个示例来演示这一点：

```js
try
{
  let s = new Symbol(); //"TypeError" exception
}
catch(e)
{
  console.log(e.message); //Output "Symbol is not a constructor"
}
```

但 JavaScript 引擎可以内部使用`Symbol()`函数作为构造函数来包装一个符号。因此，"`s`"将等于`Object(s)`。

### 注意

从 ES6 开始引入的所有原始类型都不会允许手动调用它们的构造函数。

## 使用符号作为属性键

直到 ES5，JavaScript 对象属性键必须是字符串类型。但在 ES6 中，JavaScript 对象属性键可以是字符串或符号。以下是一个示例，演示如何使用符号作为对象属性键：

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

从前面的代码中，你可以看到，为了使用符号创建或检索属性键，你需要使用`[]`标记。我们在讨论第二章中的计算属性名称时看到了`[]`标记，即*了解你的库*。

要访问符号属性键，我们需要符号。在前面示例中，`s1` 和 `s2` 都持有相同的符号值。

### 注意

在 ES6 中引入符号的主要原因是使其可以作为对象属性的键使用，并防止属性键的意外冲突。

## `Object.getOwnPropertySymbols()` 方法

`Object.getOwnPropertyNames()` 方法不能检索符号属性。因此，ES6 引入了 `Object.getOwnPropertySymbols()` 来检索对象符号属性数组。以下是一个示例来演示这一点：

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

从前面的示例中，你可以看到 `Object.getOwnPropertySymbols()` 方法也可以检索不可枚举的符号属性。

### 注意

`in` 操作符可以在对象中找到符号属性，而 `for…in` 循环和 `Object.getOwnPropertyNames()` 由于向后兼容性原因不能在对象中找到符号属性。

## `Symbol.for(string)` 方法

`Symbol` 对象维护一个键/值对的注册表，其中键是符号描述，值是符号。每次我们使用 `Symbol.for()` 方法创建符号时，它都会被添加到注册表中，并且该方法返回符号。如果我们尝试使用已存在的描述创建符号，那么将检索现有的符号。

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

## 已知符号

除了你自己的符号外，ES6 还提供了一套内置的符号，称为**已知符号**。以下是一个属性列表，引用了一些重要的内置符号：

+   `Symbol.iterator`

+   `Symbol.match`

+   `Symbol.search`

+   `Symbol.replace`

+   `Symbol.split`

+   `Symbol.hasInstance`

+   `Symbol.species`

+   `Symbol.unscopables`

+   `Symbol.isContcatSpreadable`

+   `Symbol.toPrimitive`

+   `Symbol.toStringTag`

你将在本书的各个章节中遇到这些符号的使用。

### 注意

在引用文本中的已知符号时，我们通常使用 `@@` 符号作为前缀。例如，`Symbol.iterator` 符号被称为 `@@iterator` 方法。这样做是为了使在文本中引用已知符号更容易。

# 迭代协议

迭代协议是一组规则，对象需要遵循这些规则以实现接口，当使用这些规则时，循环或构造可以遍历对象的一组值。

ES6 引入了两种新的迭代协议，称为**可迭代协议**和迭代器协议。

## 迭代器协议

任何实现迭代器协议的对象都称为迭代器。根据迭代器协议，一个对象需要提供一个 `next()` 方法，该方法返回一组项目序列中的下一个项目。

以下是一个演示此点的例子：

```js
let obj = {
  array: [1, 2, 3, 4, 5],
  nextIndex: 0,
  next: function(){
    return this.nextIndex < this.array.length ?
    {value: this.array[this.nextIndex++], done: false} :
    {done: true};
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

每次调用 `next()` 方法时，它返回一个具有两个属性的对象：`value` 和 `done`。让我们看看这两个属性代表什么：

+   `done` 属性：如果迭代器已经遍历完值集合，则返回 `true`。否则，返回 `false`。

+   `value` 属性：持有集合中当前项的值。当 `done` 属性为 `true` 时，该值被省略。

## 可迭代协议

任何实现可迭代协议的对象都称为可迭代对象。根据可迭代协议，一个对象需要提供 `@@iterator` 方法；也就是说，它必须具有 `Symbol.iterator` 符号作为属性键。`@@iterator` 方法必须返回一个迭代器对象。

以下是一个演示此点的例子：

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

# 生成器

生成器函数就像一个普通函数一样，但它不是返回单个值，而是逐个返回多个值。调用生成器函数不会立即执行其主体，而是返回一个生成器对象的新实例（即实现迭代器和可迭代协议的对象）。

每个生成器对象都持有生成器函数的新执行上下文。当我们执行生成器对象的 `next()` 方法时，它会执行生成器函数的主体，直到遇到 `yield` 关键字。它返回产生值，并暂停函数。当再次调用 `next()` 方法时，它继续执行，然后返回下一个产生值。当生成器函数不再产生任何值时，`done` 属性为 `true`。

生成器函数使用 `function*` 表达式编写。以下是一个演示此点的例子：

```js
function* generator_function()
{
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

在 `yield` 关键字之后有一个表达式。该表达式的值是通过可迭代协议由生成器函数返回的。如果我们省略该表达式，则返回 `undefined`。该表达式的值就是我们所说的产生值。

我们还可以向 `next()` 方法传递一个可选参数。这个参数成为生成器函数暂停时 `yield` 语句返回的值。以下是一个演示此点的例子：

```js
function* generator_function()
{
  var a = yield 12;
  var b = yield a + 1;
  var c = yield b + 2;
  yield c + 3;
}

var generator = generator_function();

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

## `return(value)` 方法

您可以使用生成器对象的 `return()` 方法在任何时候结束生成器函数，即使它还没有产生所有值。`return()` 方法接受一个可选参数，表示要返回的最终值。

以下是一个演示此点的例子：

```js
function* generator_function()
{
  yield 1;
  yield 2;
  yield 3;
}

var generator = generator_function();

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

## `throw(exception)` 方法

您可以使用生成器对象的 `throw()` 方法在生成器函数内部手动触发异常。您必须向 `throw()` 方法传递您想要抛出的异常。以下是一个演示此点的例子：

```js
function* generator_function()
{

  try
  {
    yield 1;
  }
  catch(e)
  {
    console.log("1st Exception");
  }

  try
  {
    yield 2;
  }
  catch(e)
  {
    console.log("2nd Exception");
  }

}

var generator = generator_function();

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

在前面的示例中，你可以看到异常是在函数上次暂停的地方抛出的。在异常被处理后，`throw()`方法继续执行，并返回下一个产生的值。

## “yield*”关键字

生成器函数内部的`yield*`关键字将一个可迭代对象作为表达式并迭代它以产生其值。以下是一个演示此功能的示例：

```js
function* generator_function_1()
{
  yield 2;
  yield 3;
}

function* generator_function_2()
{
  yield 1;
  yield* generator_function_1();
  yield* [4, 5];
}

var generator = generator_function_2();

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

# “for…of”循环

到目前为止，我们一直在使用`next()`方法迭代可迭代对象，这是一个繁琐的任务。ES6 引入了`for…of`循环来简化这个任务。

`for…of`循环被引入来迭代可迭代对象的值。以下是一个演示此功能的示例：

```js
function* generator_function()
{
  yield 1;
  yield 2;
  yield 3;
  yield 4;
  yield 5;
}

let arr = [1, 2, 3];

for(let value of generator_function())
{
  console.log(value);
}

for(let value of arr)
{
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

每当进行函数调用时，都会在栈内存中创建一个执行栈来存储函数的变量。

当在另一个函数调用内部进行函数调用时，会为内部函数调用创建一个新的执行栈。但问题是，内部函数执行栈会占用一些额外的内存，即它存储了一个额外的地址，表示当此函数执行完毕时如何恢复执行。切换和创建执行栈也会消耗一些额外的 CPU 时间。当有少量或几百层嵌套调用时，这个问题并不明显，但当有数千或更多层嵌套调用时，这个问题就明显了，即 JavaScript 引擎会抛出`RangeError: Maximum call stack size exceeded`异常。你可能在创建递归函数时遇到过`RangeError`异常。

**尾调用**是一种函数调用，它可选地出现在函数的`return`语句的末尾。如果一个尾调用反复调用同一个函数，那么它被称为**尾递归**，这是递归的一个特例。尾调用的特别之处在于，有一种方法可以在进行尾调用时实际上防止额外的 CPU 时间和内存使用，那就是重用外函数的栈，而不是创建一个新的执行栈，从而节省 CPU 时间和额外的内存使用。在执行尾调用时重用执行栈被称为**尾调用优化**。

ES6 在脚本以`"use strict"`模式编写时增加了对尾调用优化的支持。让我们看看一个尾调用的例子：

```js
"use strict";

function _add(x, y)
{
  return x + y;
}

function add1(x, y)
{
  x = parseInt(x);
  y = parseInt(y);

  //tail call
  return _add(x, y);
}

function add2(x, y)
{
  x = parseInt(x);
  y = parseInt(y);

  //not tail call
  return 0 + _add(x, y);
}

console.log(add1(1, '1')); //2
console.log(add2(1, '2')); //3
```

在这里，`add1()`函数中的`_add()`调用是一个尾调用，因为它是`add1()`函数的最终操作。但`add2()`函数中的`_add()`调用不是尾调用，因为它不是最终操作，将`0`添加到`_add()`的结果才是最终操作。

`add1()`函数中的`_add()`调用不会创建一个新的执行栈。相反，它重用了`add1()`函数的执行栈；换句话说，发生了尾调用优化。

## 将非尾调用转换为尾调用

由于尾调用得到了优化，因此你必须在可能的情况下使用尾调用，而不是非尾调用。你可以通过将非尾调用转换为尾调用来优化你的代码。

让我们看看将非尾调用转换为尾调用的一个例子，这与之前的例子类似：

```js
"use strict";

function _add(x, y)
{
  return x + y;
}

function add(x, y)
{
  x = parseInt(x);
  y = parseInt(y);

  var result = _add(x, y);
  return result;
}

console.log(add(1, '1'));
```

在之前的代码中，`_add()`调用不是一个尾调用，因此创建了两个执行栈。我们可以这样将其转换为尾调用：

```js
function add(x, y)
{
  x = parseInt(x);
  y = parseInt(y);

  return _add(x, y);
}
```

在这里，我们省略了`result`变量的使用，而是将函数调用与`return`语句并排排列。同样，还有许多其他策略可以将非尾调用转换为尾调用。

# 摘要

在本章中，我们学习了一种使用符号创建对象属性键的新方法。我们看到了迭代器和可迭代协议，并学习了如何在自定义对象中实现这些协议。然后，我们学习了如何使用`for…of`循环遍历可迭代对象。最后，我们通过学习尾调用是什么以及它们在 ES6 中的优化来结束本章。

在下一章中，我们将学习什么是 Promise，以及如何使用 Promise 编写更好的异步代码。
