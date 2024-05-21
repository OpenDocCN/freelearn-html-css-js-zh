# 附录 C.内置对象

本附录列出了**ECMAScript**（**ES**）标准中概述的内置构造函数，以及由这些构造函数创建的对象的属性和方法。ES5 特定的 API 被单独列出。

# Object

`Object()`是一个创建对象的构造函数，例如：

```js
    > var o = new Object(); 

```

这与使用对象文字相同：

```js
    > var o = {}; // recommended 

```

你可以将任何东西传递给构造函数，它将尝试猜测它是什么，并使用更合适的构造函数。例如，将字符串传递给`new Object()`将与使用`new String()`构造函数相同。这不是一种推荐的做法（最好是明确而不是让猜测渗入），但仍然是可能的：

```js
    > var o = new Object('something'); 
    > o.constructor; 
    function String() { [native code] } 
    > var o = new Object(123); 
    > o.constructor; 
    function Number() { [native code] } 

```

所有其他对象，无论是内置的还是自定义的，都继承自`Object`。因此，以下部分列出的属性和方法适用于所有类型的对象。

## Object 构造函数的成员

以下是`Object`构造函数的成员：

| **属性/方法** | **描述** |
| --- | --- |

| `Object.prototype` | 所有对象的原型（也是一个对象本身）。你添加到这个原型的任何东西都将被所有其他对象继承，所以要小心：

```js
    > var s = new String('noodles');   
    > Object.prototype.custom = 1;   
    1   
    > s.custom;   
    1   

```

|

## Object.prototype 成员

不要说“由 Object 构造函数创建的对象的成员”，让我们称之为“`Object.prototype`成员”。对于`Array.prototype`以及在附录中的其他对象也是一样的：

| **属性/方法** | **描述** |
| --- | --- |

| `constructor` | 指向用于创建对象的构造函数，这里是`Object`：

```js
    > Object.prototype.constructor === Object;   
    true   
    > var o = new Object();   
    > o.constructor === Object;   
    true   

```

|

| `toString(radix)` | 返回对象的字符串表示。如果对象恰好是一个`Number`对象，则基数参数定义了返回数字的基数。默认基数是`10`：

```js
    > var o = {prop: 1};   
    > o.toString();   
    "[object Object]"   
    > var n = new Number(255);   
    > n.toString();   
    "255"   
    > n.toString(16);   
    "ff"   

```

|

| `toLocaleString()` | 与`toString()`相同，但匹配当前区域设置。旨在由对象自定义，例如`Date()`、`Number()`和`Array()`，并提供特定于区域设置的值，例如不同的日期格式。对于`Object()`实例以及大多数其他情况，它只是调用`toString()`。在浏览器中，您可以使用导航器 BOM 对象的`language`属性（或 IE 中的`userLanguage`）来确定语言：

```js
    > navigator.language;   
    "en-US"   

```

|

| `valueOf()` | 如果适用，返回`this`的原始表示。例如，`Number`对象返回一个原始数字，`Date`对象返回一个时间戳。如果没有合适的原始值，它简单地返回`this`：

```js
    > var o = {};   
    > typeof o.valueOf();   
    "object"   
    > o.valueOf() === o;   
    true   
    > var n = new Number(101);   
    > typeof n.valueOf();   
    "number"   
    > n.valueOf() === n;   
    false   
    > var d = new Date();   
    > typeof d.valueOf();   
    "number"   
    > d.valueOf();   
    1357840170137   

```

|

| `hasOwnProperty(prop)` | 如果属性是对象的自有属性，则返回`true`，如果是从原型链继承的，则返回`false`。如果属性不存在，则也返回`false`：

```js
    > var o = {prop: 1};   
    > o.hasOwnProperty('prop');   
    true   
    > o.hasOwnProperty('toString');   
    false   
    > o.hasOwnProperty('fromString');   
    false   

```

|

| `isPrototypeOf(obj)` | 如果一个对象被用作另一个对象的原型，则返回`true`。可以测试原型链中的任何对象，不仅仅是直接创建者：

```js
    > var s = new String('');   
    > Object.prototype.isPrototypeOf(s);   
    true   
    > String.prototype.isPrototypeOf(s);   
    true   
    > Array.prototype.isPrototypeOf(s);   
    false   

```

|

| `propertyIsEnumerable(prop)` | 如果属性在`for...in`循环中显示，则返回`true`：

```js
    > var a = [1, 2, 3];   
    > a.propertyIsEnumerable('length');   
    false   
    > a.propertyIsEnumerable(0);   
    true   

```

|

## ECMAScript 5 对对象的增加

在 ECMAScript 3 中，所有对象属性都可以随时更改、添加或删除，除了一些内置属性（例如`Math.PI`）。在 ES5 中，您可以定义不能更改或删除的属性，这是以前保留给内置属性的特权。ES5 引入了**属性描述符**的概念，让您更严格地控制您定义的属性。

将属性描述符视为指定属性特征的对象。描述这些特征的语法是一个常规对象文字，因此属性描述符有自己的属性和方法，但让我们称它们为**属性**以避免混淆。这些属性是：

+   `value` - 当访问属性时得到的值

+   `writable` - 可以更改此属性吗

+   `enumerable` - 它是否应该出现在`for...in`循环中

+   `configurable` - 可以删除它

+   `set()` - 每次更新值时调用的函数

+   `get()` - 当访问属性的值时调用

此外，**数据描述符**（您定义属性`enumerable`，`configurable`，`value`和`writable`）和**访问器描述符**（您定义`enumerable`，`configurable`，`set()`和`get()`）之间存在区别。如果定义了`set()`或`get()`，则描述符被视为访问器，并且尝试定义值或可写性将引发错误。

定义一个常规的老式 ES3 风格的属性：

```js
    var person = {}; 
    person.legs = 2; 

```

使用 ES5 数据描述符相同：

```js
    var person = {}; 
    Object.defineProperty(person, "legs", { 
      value: 2, 
      writable: true, 
      configurable: true, 
      enumerable: true 
    }); 

```

如果将`value`的值默认设置为`undefined`，则所有其他值都为`false`。因此，如果要能够稍后更改此属性，则需要显式将它们设置为`true`。

或者使用 ES5 访问器描述符的相同属性：

```js
    var person = {}; 
    Object.defineProperty(person, "legs", { 
      set: function (v) {this.value = v;}, 
      get: function (v) {return this.value;}, 
      configurable: true, 
      enumerable: true 
    }); 
    person.legs = 2; 

```

如您所见，属性描述符是更多的代码，因此只有在您真正想要防止其他人篡改您的属性时才使用它们，并且您忘记了与 ES3 浏览器的向后兼容性，因为与例如`Array.prototype`的添加不同，您不能在旧浏览器中“shim”此功能。

并且描述符的功能（定义不可塑性属性）：

```js
    > var person = {}; 
    > Object.defineProperty(person, 'heads', {value: 1}); 
    > person.heads = 0; 
    0 
    > person.heads; 
    1 
    > delete person.heads; 
    false 
    > person.heads; 
    1 

```

以下是`Object`的所有 ES5 新增内容的列表：

| **属性/方法** | **描述** |
| --- | --- |

| `Object.getPrototypeOf(obj)` | 在 ES3 中，您必须猜测给定对象的原型是什么，使用`Object.prototype.isPrototypeOf()`方法，而在 ES5 中，您可以直接询问“你的原型是谁？”

```js
    > Object.getPrototypeOf([]) ===    
       Array.prototype;   
    true   

```

|

| `Object.create(obj, descr`) | 在第七章中讨论，*继承*。创建一个新对象，设置其原型，并使用属性描述符定义该对象的属性（前面讨论过）：

```js
    > var parent = {hi: 'Hello'};   
    > var o = Object.create(parent,
    { prop: {value: 1 }});   
    > o.hi;   
    "Hello"   

```

它甚至让您创建一个完全空白的对象，这是您在 ES3 中无法做到的：

```js
    > var o = Object.create(null);   
    > typeof o.toString;   
    "undefined"   

```

|

| `Object.getOwnPropertyDescriptor(obj, property)` | 允许您检查属性的定义方式。您甚至可以窥视内置对象，并查看所有这些以前隐藏的属性：

```js
    > Object.getOwnProperty
      Descriptor (Object.prototype,
      'toString');   
    Object   
    configurable: true   
    enumerable: false   
    value: function toString() {
     [native code] }   
    writable: true   

```

|

| `Object.getOwnPropertyNames(obj)` | 返回所有自有属性名称（作为字符串的数组），无论是否可枚举。使用`Object.keys()`仅获取可枚举的属性：

```js
    > Object.getOwnPropertyNames(   
      Object.prototype);   
    ["constructor", "toString",
    "toLocaleString", "valueOf",
     ...   

```

|

| `Object.defineProperty(obj, descriptor)` | 使用属性描述符定义对象的属性。请参阅本表之前的讨论。 |
| --- | --- |

| `Object.defineProperties(obj, descriptors)` | 与`defineProperty()`相同，但允许您一次定义多个属性：

```js
    > var glass =    
     Object.defineProperties({}, {   
        "color": {   
          value: "transparent",   
          writable: true   
        },   
        "fullness": {   
          value: "half",   
          writable: false   
        }   
      });   

    > glass.fullness;   
    "half"   

```

|

| `Object.preventExtensions(obj)``Object.isExtensible(obj)` | `preventExtensions()`禁止向对象添加更多属性，`isExtensible()`检查是否可以添加属性：

```js
    > var deadline = {};   
    > Object.isExtensible(deadline);   
    true   
    > deadline.date = "yesterday";   
    "yesterday"   
    > Object.preventExtensions(
     deadline);   
    > Object.isExtensible(deadline);   
    false   
    > deadline.date = "today";   
    "today"   
    > deadline.date;   
    "today"   

```

尝试向不可扩展的对象添加属性不会报错，但只是不起作用：

```js
    > deadline.report = true;   
    > deadline.report;   
    undefined   

```

|

| `Object.seal(obj)``Object.isSealed(obj)` | `seal()`与`preventExtensions()`相同，并且还使所有现有属性不可配置。这意味着您可以更改现有属性的值，但不能删除它或重新配置它（使用`defineProperty()`不起作用）。因此，您不能将可枚举属性变为不可枚举，例如。 |
| --- | --- |

| `Object.freeze(obj)``Object.isFrozen(obj)` | `seal()`所做的一切，还阻止更改属性的值：

```js
    > var deadline = Object.freeze(   
        {date: "yesterday"});   
    > deadline.date = "tomorrow";   
    > deadline.excuse = "lame";   
 **> deadline.date;**
**"yesterday"**
**>
    deadline.excuse;**
**undefined**
**> 
    Object.isSealed(deadline);**
**true**

```

|

| `Object.keys(obj)` | 用于`for...in`循环的替代方法。仅返回自有属性（不像`for...in`）。属性需要可枚举才会显示出来（不像`Object.getOwnPropertyNames()`）。返回值是一个字符串数组。

```js
 **>Object.prototype.customProto =
     101;**
 **> Object.getOwnPropertyNames(** 
 **Object.prototype);** 
 **["constructor", "toString", ...,
      "customProto"]** 
 **> Object.keys(Object.prototype);** 
 **["customProto"]**
**> var o = {own: 202};**
**> o.customProto;** 
**101**
**> Object.keys(o);**
**"own"]**

```

|

# ES6 对象的新增内容

ES6 具有一些有趣的对象定义和属性语法。这种新语法是为了使对象的操作更加简单和简洁。

## 属性简写

ES6 提供了一种更短的语法来定义常见对象。

ES5：`obj = { x: x, y: y };`

ES6：`obj = {x,y};`

## 计算属性名称

ES6 对象定义语法中可以计算属性名称：

```js
    let obj = { 
      foo: "bar", 
      [ "baz" + q() ]: 42 
    } 

```

在这里，属性名称是计算的，其中`"baz"`与函数调用的结果连接在一起。

## Object.assign

`Object.assign()` 方法用于将一个或多个源对象的所有可枚举自有属性的值复制到目标对象中：

```js
    var dest  = { quux: 0 } 
    var src1 = { foo: 1, bar: 2 } 
    var src2 = { foo: 3, baz: 4 } 
    Object.assign(dst, src1, src2) 

```

# 数组

`Array` 构造函数创建数组对象：

```js
    > var a = new Array(1, 2, 3); 

```

这与数组文字相同：

```js
    > var a = [1, 2, 3]; //recommended 

```

当您将一个数值传递给 `Array` 构造函数时，假定它是数组的长度：

```js
    > var un = new Array(3); 
    > un.length; 
    3 

```

您会得到所需长度的数组，如果您要求每个数组元素的值，您会得到 `undefined`：

```js
    > un; 
    [undefined, undefined, undefined] 

```

数组中充满元素和没有元素但只有长度之间有微妙的差异：

```js
    > '0' in a; 
    true 
    > '0' in un; 
    false 

```

`Array()` 构造函数在指定一个参数与指定多个参数时的行为差异可能会导致意想不到的结果。例如，以下对数组文字的使用是有效的：

```js
    > var a = [3.14]; 
    > a; 
    [3.14] 

```

然而，将浮点数传递给 `Array` 构造函数是一个错误：

```js
    > var a = new Array(3.14); 
    Range Error: invalid array length 

```

## `Array.prototype` 成员

以下是 `Array` 的所有元素列表：

| **属性/方法** | **描述** |
| --- | --- |

| `length` | 数组中的元素数量：

```js
    > [1, 2, 3, 4].length;   
    4   

```

|

| `concat(i1, i2, i3,...)` | 合并数组：

```js
    > [1, 2].concat([3, 5], [7, 11]);   
    [1, 2, 3, 5, 7, 11]   

```

|

| `join(separator)` | 将数组转换为字符串。分隔符参数是一个字符串，默认值为逗号：

```js
    > [1, 2, 3].join();   
    "1,2,3"   
    > [1, 2, 3].join('&#124;');   
    "1&#124;2&#124;3"   
    > [1, 2, 3].join(' is less than   ');   
    "1 is less than 2 is less   than 3"   

```

|

| `pop()` | 移除数组的最后一个元素并返回它：

```js
    > var a = ['une', 'deux', 'trois'];   
    > a.pop();   
    "trois"   
    > a;   
    ["une", "deux"]   

```

|

| `push(i1, i2, i3,...)` | 将元素附加到数组的末尾并返回修改后数组的长度：

```js
    > var a = [];   
    > a.push('zig', 'zag', 'zebra','zoo');   
    4   

```

|

| `reverse()` | 反转数组的元素并返回修改后的数组：

```js
    > var a = [1, 2, 3];   
    > a.reverse();   
    [3, 2, 1]   
    > a;   
    [3, 2, 1]   

```

|

| `shift()` | 类似于 `pop()`，但是移除的是第一个元素，而不是最后一个：

```js
    > var a = [1, 2, 3];   
    > a.shift();   
    1   
    > a;   
    [2, 3]   

```

|

| `slice(start_index, end_index)` | 提取数组的一部分并将其作为新数组返回，而不修改源数组：

```js
    > var a = ['apple', 'banana', 'js',
      'css', 'orange'];   
    > a.slice(2,4);   
    ["js", "css"]   
    > a;   
    ["apple", "banana", "js", "css", "orange"]   

```

|

| `sort(callback)` | 对数组进行排序。还可以接受一个用于自定义排序的回调函数。回调函数接收两个数组元素作为参数，如果它们相等则返回 `0`，如果第一个大于第二个则返回正数，如果第二个大于第一个则返回负数。下面是一个自定义排序函数的示例，它执行正确的数字排序（因为默认是字符排序）：

```js
    function customSort(a, b) {   
             if (a > b) return 1;    
             if (a < b) return -1;    
             return 0;   
    }   
    Example use of sort():   
    > var a = [101, 99, 1, 5];   
    > a.sort();   
     [1, 101, 5, 99]   
    > a.sort(customSort);   
    [1, 5, 99, 101]   
    > [7, 6, 5, 9].sort(customSort);   
    [5, 6, 7, 9]   

```

|

| `splice(start, delete_count, i1, i2, i3,...)` | 同时移除和添加元素。第一个参数是开始移除的位置，第二个是要移除的项目数，其余参数是要插入到已移除项目位置的新元素：

```js
    > var a = ['apple', 'banana',
      'js', 'css', 'orange'];   
    > a.splice(2, 2, 'pear', 'pineapple');   
    ["js", "css"]   
    > a;   
    ["apple", "banana",   "pear", 
      "pineapple", "orange"]   

```

|

| `unshift(i1, i2, i3,...)` | 类似于 `push()`，但是将元素添加到数组的开头而不是末尾。返回修改后数组的长度：

```js
    > var a = [1, 2, 3];    
    > a.unshift('one', 'two');    
    5   
    > a;   
    ["one", "two",   1, 2, 3]   

```

|

## `Array` 的 ECMAScript 5 添加内容

以下是 `Array` 的 ECMAScript 5 添加内容：

| **属性/方法** | **描述** |
| --- | --- |

| `Array.isArray(obj)` | 告诉对象是否是数组，因为 `typeof` 不够好：

```js
    > var arraylike = {0: 101, 
    length: 1};   
    > typeof arraylike;   
    "object"   
    > typeof [];   
    "object"   

```

两者都不是鸭子类型（如果它走起来像鸭子，叫起来像鸭子，那它一定是鸭子）：

```js
    typeof arraylike.length;   
    "number"   

```

在 ES3 中，您需要冗长的：

```js
    > Object.prototype.toString
    .call
    ([]) === "[object Array]"; 
    true
    > Object.prototype.toString.call
    (arraylike) === 
    "[object Array]";
    false   

```

在 ES5 中，您会得到更简短的：

```js
    Array.isArray([]);   
    true   
    Array.isArray(arraylike);   
    false   

```

|

| `Array.prototype.indexOf(needle, idx)` | 搜索数组并返回第一个匹配项的索引。如果没有匹配项，则返回 `-1`。还可以选择从指定索引开始搜索：

```js
    > var ar = ['one', 'two', 
      'one',   'two'];   
    > ar.indexOf('two');   
    1   
    > ar.indexOf('two', 2);   
    3   
    > ar.indexOf('toot');   
    -1   

```

|

| `Array.prototype.lastIndexOf(needle, idx)` | 类似于 `indexOf()`，只是从末尾开始搜索：

```js
    > var ar = ['one', 'two', 
    'one', 'two']; 
    > ar.lastIndexOf('two');   
    3   
    > ar.lastIndexOf('two', 2);   
    1   
    > ar.indexOf('toot');   
    -1   

```

|

| `Array.prototype.forEach(callback, this_obj)` | 替代 `for` 循环的方法。您可以指定一个回调函数，该函数将对数组的每个元素进行调用。回调函数接收参数：元素、其索引和整个数组：

```js
    > var log =
    console.log.bind(console);   
    > var ar = ['itsy', 'bitsy',
      'spider'];   
    > ar.forEach(log);   
     itsy      0   ["itsy", 
     "bitsy", "spider"]   
     bitsy    1   ["itsy", 
     "bitsy", "spider"] 
     spider  2   ["itsy", 
    "bitsy", "spider"]   

```

还可以指定第二个参数：绑定到回调函数内部的对象。因此，这也是有效的：

```js
    > ar.forEach(console.log, 
      console);   

```

|

| `Array.prototype.every(callback, this_obj)` | 您提供一个测试数组每个元素的回调函数。您的回调函数与 `forEach()` 接收相同的参数，并且必须根据给定元素是否满足您的测试返回 `true` 或 `false`。如果所有元素都满足您的测试，`every()` 返回 `true`。如果至少有一个不满足，`every()` 返回 `false`：

```js
    > function hasEye(el, idx,
    ar) { 
     return el.indexOf('i') !== 
      -1; 
     }  
    > ['itsy', 'bitsy',
      'spider'].
    every(hasEye); 
    true
    > ['eency', 'weency', 
    'spider'].every(hasEye);   
    false   

```

如果在循环过程中某个时刻变得清楚结果将是`false`，循环将停止并返回`false`：

```js
    > [1,2,3].every(function (e)
      { 
        console.log(e);   
        return false;   
      });   
     1   
     false   

```

|

| `Array.prototype.some(callback, this_obj)` | 类似于`every()`，只有至少一个元素满足您的测试时返回`true`：

```js
    > ['itsy', 'bitsy', 
      'spider'].some(hasEye);
      true   
    > ['eency', 'weency',
     'spider'].some(hasEye);   
      true   

```

|

| `Array.prototype.filter(callback, this_obj)` | 类似于`some()`和`every()`，但它返回满足您测试的所有元素的新数组：

```js
    > ['itsy', 'bitsy', 
      'spider'].filter(hasEye); 
     ["itsy", "bitsy",
     "spider"]
    > ['eency', 'weency',
      'spider'].filter(hasEye); 
      ["spider"]   

```

|

| `Array.prototype.map(callback, this_obj)` | 类似于`forEach()`，因为它对每个元素执行回调，但另外它构造一个新数组，并返回回调的返回值。让我们将数组中的所有字符串大写：

```js
    > function uc(element, index,
      array) {
      return element.toUpperCase(); 
    }   
    > ['eency', 'weency',
      'spider'].map(uc);
    ["EENCY", "WEENCY", "SPIDER"]   

```

|

| `Array.prototype.reduce(callback, start)` | 对数组的每个元素执行回调。您的回调返回一个值。这个值随后传回到您的回调中进行下一次迭代。整个数组最终被减少为一个单一的值：

```js
    > function sum(res, element, 
      idx, arr) {
       return res + element;
     } 
    > [1, 2, 3].reduce(sum);
     6   

```

可以选择传递一个起始值，该值将在第一个回调调用时使用：

```js
    > [1, 2, 3].reduce(sum, 100);   
    106   

```

|

| `Array.prototype.reduceRight(callback, start)` | 类似于`reduce()`，但是从数组的末尾循环：

```js
    > function concat(
    result_so_far, el) { 
    return "" +  result_so_far 
    + el;
    } 
    > [1, 2, 3].reduce(concat);   
    "123" 
    > [1, 2, 3].reduceRight
    (concat);
    "321"   

```

|

## ES6 对数组的添加

以下是对数组的添加：

| `Array.from(arrayLike, mapFunc?, thisArg?)` | `Array.from()`方法的基本功能是将两种类型的值转换为数组-`arrayLike`值和`Iterable`值：

```js
    const arrayLike = { length:
     2, 0: 'a', 1: 'b' };
    const arr =
    Array.from(arrayLike);
    for (const x of arr) {
     // OK, iterable
    console.log(x);
    }
    // Output:
    // a
    // b
```

|

| `Array.of(...items)` | 从传递给该方法的项目创建一个数组

```js
    let a = Array.of(
      1,2,3,'foo');
    console.log(a); //[1, 2,
     3, "foo"]
```

|

| `Array.prototype.entries()``Array.prototype.keys()``Array.prototype.values()` | 这些方法的结果是一系列值。这些方法分别返回键、值和条目的迭代器。

```js
    let a = Array.of(1,2,
    3,'foo');
    let k,v,e;
    for (k of a.keys()) {
    console.log(k); //0 1 2 3
    }
    for (v of a.values()) {
    console.log(v); //1 2 
    3 foo
    }
    for (e of a.entries()){
    console.log(e);
    }
    //[[0,1],[1,2],[2,3]
    [3,'foo']]
```

|

| `Array.prototype.find(predicate, thisArg?)` | 返回回调函数返回`true`的第一个数组元素。如果没有这样的元素，则返回`undefined`：

```js
    [1, -2, 3].find(x => x < 0)
     //-2
```

|

| `Array.prototype.findIndex(predicate, thisArg?)` | 返回回调函数返回 true 的第一个元素的索引。如果没有这样的元素，则返回`-1`：

```js
    [1, -2, 3].find(x => x < 0)
     //1
```

|

| `Array.prototype.fill(value : any, start=0, end=this.length) : This` | 用给定值填充数组：

```js
const arr = ['a', 'b', 'c'];
arr.fill(7)
[ 7, 7, 7 ]
You can specify start and end ranges.
['a', 'b', 'c'].fill(7, 1, 2)
[ 'a', 7, 'c' ]
```

|

# 函数

JavaScript 函数是对象。它们可以使用`Function`构造函数进行定义，如下所示：

```js
    var sum = new Function('a', 'b', 'return a + b;'); 

```

这是函数文字（也称为函数表达式）的（通常不推荐的）替代方法：

```js
    var sum = function (a, b) { 
      return a + b; 
    }; 

```

或者，更常见的函数定义：

```js
    function sum(a, b) { 
      return a + b; 
    } 

```

## Function.prototype 成员

以下是`Function`构造函数的成员列表：

| **属性/方法** | **描述** |
| --- | --- |

| `apply(this_obj, params_array)` | 允许您调用另一个函数，同时覆盖另一个函数的`this`值。`apply()`接受的第一个参数是要绑定到函数内部的对象，第二个参数是要发送到被调用函数的参数数组：

```js
    function whatIsIt(){   
      return this.toString();   
    }   
    > var myObj = {};   
    > whatIsIt.apply(myObj);   
    "[object Object]"   
    > whatIsIt.apply(window);   
    "[object Window]"   

```

|

| `call(this_obj, p1, p2, p3, ...)` | 与`apply()`相同，但是逐个接受参数，而不是作为一个数组。 |
| --- | --- |

| `length` | 函数期望的参数数量：

```js
    > parseInt.length;   
    2   

```

如果您忘记了`call()`和`apply()`之间的区别：

```js
    > Function.prototype.call.length;   
    1   
    > Function.prototype.apply.length;   
    2   

```

`call()`属性的长度为`1`，因为除第一个参数外，所有参数都是可选的。|

## ECMAScript 5 对函数的添加

以下是 ECMAScript 5 对`Function`构造函数的添加：

| **属性/方法** | **描述** |
| --- | --- |

| `Function.prototype.bind()` | 当您想要调用一个内部使用 this 的函数，并且您想要定义 this 是什么时。`call()`和`apply()`方法调用函数，而`bind()`返回一个新函数。当您将一个方法提供为另一个对象的方法的回调时，并且您希望 this 是您选择的对象时，这是有用的：

```js
    > whatIsIt.apply(window);   
    "[object Window]"   

```

|

## ECMAScript 6 对函数的添加

以下是 ECMAScript 6 对`Function`构造函数的添加：

| **箭头函数**箭头函数表达式与函数表达式相比具有更短的语法，并且不绑定自己的 this、arguments、super 或`new.target`。箭头函数始终是匿名的。 |
| --- |

```js
    () => { ... }
    // no parameter 
    x => { ... }
    // one 
    parameter, an
    identifier 
    (x, y) => 
    {   ... }
    // several
    parameters
    const squares =
    [1, 2, 3].map(
    x => x * x);   

```

|

| 语句主体更具表现力和简洁的闭包语法 |
| --- |

```js
    arr.forEach(v =>
    { if (v % 5 
     ===0)
filtered:ist.push(v)
    })   

```

|

# 布尔

`Boolean`构造函数创建布尔对象（不要与布尔原语混淆）。布尔对象并不那么有用，这里列出来是为了完整起见。

```js
    > var b = new Boolean(); 
    > b.valueOf(); 
    false 
    > b.toString(); 
    "false" 

```

布尔对象与布尔原始值不同。如您所知，所有对象都是真值：

```js
    > b === false; 
    false 
    > typeof b; 
    "object" 

```

布尔对象除了从`Object`继承的属性之外，没有任何属性。

# 数字

这将创建数字对象：

```js
    > var n = new Number(101); 
    > typeof n; 
    "object" 
    > n.valueOf(); 
    101 

```

`Number`对象不是原始对象，但如果您在原始数字上使用任何`Number.prototype`方法，原始对象将在后台转换为`Number`对象，并且代码将正常工作。

```js
    > var n = 123; 
    > typeof n; 
    "number" 
    > n.toString(); 
    "123" 

```

在不使用`new`的情况下，`Number`构造函数返回原始数字。

```js
    > Number("101"); 
    101 
    > typeof Number("101"); 
    "number" 
    > typeof new Number("101"); 
    "object" 

```

## Number 构造函数的成员

考虑`Number`构造函数的以下成员：

| **属性/方法** | **描述** |
| --- | --- |

| `Number.MAX_VALUE` | 一个常量属性（无法更改），包含允许的最大数字：

```js
    > Number.MAX_VALUE;   
    1.7976931348623157e+308   

```

|

| `Number.MIN_VALUE` | 您可以在 JavaScript 中使用的最小数字：

```js
    > Number.MIN_VALUE;   
    5e-324   

```

|

| `Number.NaN` | 包含非数字的数字。与全局 NaN 相同：

```js
    > Number.NaN;    
    NaN   

```

NaN 不等于任何东西，包括它自己：

```js
    > Number.NaN === Number.NaN;   
    false   

```

|

| `Number.POSITIVE_INFINITY` | 与全局`Infinity`数字相同。 |
| --- | --- |
| `Number.NEGATIVE_INFINITY` | 与`-Infinity`相同。 |

## Number.prototype 成员

以下是`Number`构造函数的成员：

| **属性/方法** | **描述** |
| --- | --- |

| `toFixed(fractionDigits)` | 返回具有数字的定点表示的字符串。四舍五入返回值：

```js
    > var n = new Number(Math.PI);   
    > n.valueOf();   
    3.141592653589793   
    > n.toFixed(3);   
    "3.142"   

```

|

| `toExponential(fractionDigits)` | 返回具有数字对象的指数表示的字符串。四舍五入返回值：

```js
    > var n = new Number(56789);   
    > n.toExponential(2);   
    "5.68e+4"   

```

|

| `toPrecision(precision)` | 数字对象的字符串表示，指数或定点表示，取决于数字对象：

```js
    > var n = new Number(56789);   
    > n.toPrecision(2);   
    "5.7e+4"   
    > n.toPrecision(5);   
    "56789"   
    > n.toPrecision(4);   
    "5.679e+4"   
    > var n = new Number(Math.PI);   
    > n.toPrecision(4);   
    "3.142"   

```

|

# 字符串

`String()`构造函数创建字符串对象。如果您像操作对象一样调用方法，原始字符串将在后台转换为对象。省略`new`会给您原始字符串。

创建字符串对象和字符串原始值：

```js
    > var s_obj = new String('potatoes');  
    > var s_prim = 'potatoes'; 
    > typeof s_obj; 
    "object" 
    > typeof s_prim; 
    "string" 

```

当使用`===`按类型比较时，对象和原始值不相等，但当使用`==`进行比较时，它们会进行类型强制转换：

```js
    > s_obj === s_prim; 
    false 
    > s_obj == s_prim; 
    true 

```

`length`是字符串对象的属性：

```js
    > s_obj.length; 
    8 

```

如果在原始字符串上访问`length`，则原始字符串将在后台转换为对象，并且操作将成功：

```js
    > s_prim.length; 
    8 

```

字符串文字也可以正常工作：

```js
    > "giraffe".length; 
 **7**

```

## 字符串构造函数的成员

以下是`String`构造函数的成员：

| **属性/方法** | **描述** |
| --- | --- |

| `String.fromCharCode (code1, code2, code3, ...)` | 返回使用输入的 Unicode 值创建的字符串：

```js
    > String.fromCharCode(115, 99,
    114,
    105, 112, 116);   
    "script"   

```

|

## String.prototype 成员

考虑以下`String.prototype`成员：

| **属性/方法** | **描述** |
| --- | --- |

| `length` | 字符串中的字符数：

```js
    > new String('four').length;   
    4   

```

|

| `charAt(position)` | 返回指定位置的字符。位置从`0`开始：

```js
    > "script".charAt(0);   
    "s"   

```

自 ES5 以来，也可以使用数组表示法来实现相同的目的。（在 ES5 之前，许多浏览器长期支持此功能，但 IE 不支持）：

```js
    > "script"[0];   
    "s"   

```

|

| `charCodeAt(position)` | 返回指定位置的字符的数字代码（Unicode）：

```js
    > "script".charCodeAt(0);   
    115   

```

|

| `concat(str1, str2, ....)` | 返回从输入片段粘合的新字符串：

```js
    > "".concat('zig', '-',   'zag');   
    "zig-zag"   

```

|

| `indexOf(needle, start)` | 如果 needle 与字符串的一部分匹配，则返回匹配的位置。可选的第二个参数定义搜索应从何处开始。如果找不到匹配项，则返回`-1`：

```js
    > "javascript".indexOf('scr');   
    4   
    > "javascript".indexOf('scr',   5);   
    -1   

```

|

| `lastIndexOf(needle, start)` | 与`indexOf()`相同，但从字符串的末尾开始搜索。最后一次出现的`a`：

```js
    > "javascript".lastIndexOf('a');   
    3   

```

|

| `localeCompare(needle)` | 比较当前区域设置中的两个字符串。如果两个字符串相等，则返回`0`，如果 needle 在字符串对象之前排序，则返回`1`，否则返回`-1`：

```js
    > "script".localeCompare('crypt');   
    1   
    > "script".localeCompare('sscript');   
    -1   
    > "script".localeCompare('script');   
    0   

```

|

| `match(regexp)` | 接受一个正则表达式对象并返回一个匹配的数组：

```js
    > "R2-D2 and C-3PO".match(/[0-9]/g);   
    ["2", "2", "3"]   

```

|

| `replace(needle, replacement)` | 允许您替换正则表达式模式的匹配结果。替换也可以是一个回调函数。捕获组可以作为`$1, $2,...$9`使用：

```js
    > "R2-D2".replace(/2/g, '-two');   
    "R-two-D-two"   
    > "R2-D2".replace(/(2)/g,'$1$1');   
    "R22-D22"   

```

|

| `search(regexp)` | 返回第一个正则表达式匹配的位置：

```js
    > "C-3PO".search(/[0-9]/);   
    2   

```

|

| `slice(start, end)` | 返回由开始和结束位置标识的字符串的部分。如果`start`为负数，则开始位置为`length` + `start`，类似地，如果`end`参数为负数，则结束位置为 length + end：

```js
    > "R2-D2 and C-3PO".slice(4,   13);   
    "2 and C-3"   
    > "R2-D2 and C-3PO".slice(4,   -1);   
    "2 and C-3P"   

```

|

| `split(separator, limit)` | 将字符串转换为数组。第二个参数 limit 是可选的。与`replace()`、`search()`和`match()`一样，分隔符是一个正则表达式，但也可以是一个字符串。

```js
    > "1,2,3,4".split(/,/);   
    ["1", "2", "3",   "4"]   
    > "1,2,3,4".split(',',   2);   
    ["1", "2"]   

```

|

| `substring(start, end)` | 类似于`slice()`。当 start 或 end 为负数或无效时，它们被视为 0。如果它们大于字符串长度，则被视为长度。如果`end`大于`start`，它们的值将被交换。

```js
    > "R2-D2 and C-3PO".substring(4, 13);   
    "2 and C-3"   
    > "R2-D2 and C-3PO".substring(13, 4);   
    "2 and C-3"   

```

|

| `toLowerCase()``toLocaleLowerCase()` | 将字符串转换为小写：

```js
    > "Java".toLowerCase();   
    "java"   

```

|

| `toUpperCase()``toLocaleUpperCase()` | 将字符串转换为大写：

```js
    > "Script".toUpperCase();   
    "SCRIPT"   

```

|

## ECMAScript 5 对 String 的新增内容

以下是 ECMAScript 5 对 String 的新增内容：

| **属性/方法** | **描述** |
| --- | --- |

`String.prototype.trim()` | 在 ES3 中使用正则表达式来去除字符串前后的空格，而在 ES5 中有一个`trim()`方法。

```js
    > " \t beard \n".trim();   
    "beard"   
    Or in ES3:   
    > " \t beard \n".replace(/\s/g, "");   
    "beard"   

```

|

## ECMAScript 6 对 String 的新增内容

以下是所有 ECMAScript 6 对 String 的新增内容：

| 模板文字用于插入单行或多行字符串。模板文字用反引号（`` ``）（重音符号）字符括起来，而不是双引号或单引号。模板文字可以包含占位符。这些由美元符号和大括号（`${expression}`）表示。占位符中的表达式和它们之间的文本被传递给一个函数。默认函数只是将部分连接成一个字符串。 |
| --- |

```js
    var a = 5; 
    var b = 10;   
    console.log(`Fifteen
    is ${a + b}`);   

```

|

| `String.prototype.repeat` - 此方法允许您重复一个字符串 n 次 |
| --- |

```js
    " ".repeat(4 * 
    depth)
    "foo".repeat(3)   

```

|

| `String.prototype.startsWith``String.prototype.endsWith``String.prototype.includes`这些是新的字符串搜索方法 |
| --- |

```js
    "hello".startsWith(
    "ello", 1) // true
    "hello".endsWith(
    "hell",4) // true
"hello".includes(
  "ell")
  // true 
"hello".includes(
  "ell", 1) // true 
"hello".includes(
 "ell", 2) // false   

```

|

# 日期

`Date`构造函数可以与多种类型的输入一起使用：

+   可以传递年、月、日期、小时、分钟、秒和毫秒的值，如下所示：

```js
        > new Date(2015, 0, 1, 13, 30, 35, 505); 
        Thu Jan 01 2015 13:30:35 GMT-0800 (PST) 

```

+   您可以跳过任何输入参数，此时它们被假定为 0。请注意，月份值从 0（一月）到 11（十二月），小时从 0 到 23，分钟和秒从 0 到 59，毫秒从 0 到 999。

+   您可以传递一个时间戳：

```js
        > new Date(1420147835505); 
        Thu Jan 01 2015 13:30:35 GMT-0800 (PST) 

```

+   如果您什么都不传，将假定为当前日期/时间：

```js
        > new Date(); 
        Fri Jan 11 2013 12:20:45 GMT-0800 (PST) 

```

+   如果传递一个字符串，它将尝试解析以提取可能的日期值：

```js
        > new Date('May 4, 2015'); 
        Mon May 04 2015 00:00:00 GMT-0700 (PDT) 

```

省略`new`会给您当前日期的字符串版本：

```js
    > Date() === new Date().toString(); 
    true 

```

## Date 构造函数的成员

以下是 Date 构造函数的成员：

| **属性/方法** | **描述** |
| --- | --- |

| `Date.parse(string)` | 类似于将字符串传递给新的`Date()`构造函数，此方法解析输入字符串以尝试提取有效的日期值。成功时返回时间戳，失败时返回`NaN`：

```js
    > Date.parse('May 5, 2015');   
    1430809200000   
    > Date.parse('4th');   
    NaN   

```

|

| `Date.UTC(year, month, date, hours, minutes, seconds, ms)` | 返回一个时间戳，但是在协调世界时（UTC）中，而不是在本地时间中。

```js
    > Date.UTC 
    (2015, 0, 1, 13, 30, 35, 505);
    1420119035505   

```

|

## The Date.prototype members

以下是`Date.prototype`成员的列表：

| **属性/方法** | **描述/示例** |
| --- | --- |

| `toUTCString()` | 与`toString()`相同，但是使用世界协调时间。太平洋标准时间（PST）本地时间与 UTC 的区别如下：

```js
    > var d = new Date(2015, 0, 1);   
    > d.toString();   
    "Thu Jan 01 2015 00:00:00 GMT-0800 (PST)"   
    > d.toUTCString();   
    "Thu, 01 Jan 2015 08:00:00   GMT"   

```

|

| `toDateString()` | 仅返回`toString()`的日期部分：

```js
    > new Date(2015, 0,   1).toDateString();   
    "Thu Jan 01 2010"   

```

|

| `toTimeString()` | 仅返回`toString()`的时间部分：

```js
    > new Date(2015, 0,   1).toTimeString();   
    "00:00:00 GMT-0800 (PST)"   

```

|

| `toLocaleString()``toLocaleDateString()``toLocaleTimeString()` | 等同于`toString()`，`toDateString()`和`toTimeString()`，但是以更友好的格式显示，根据当前用户的区域设置：

```js
    > new Date(2015, 0,   1).toString();   
    "Thu Jan 01 2015 00:00:00 GMT-0800   (PST)"   
    > new Date(2015, 0,   1).toLocaleString();   
    "1/1/2015 12:00:00 AM"   

```

|

| `getTime()``setTime(time)` | 获取或设置日期对象的时间（使用时间戳）。以下示例创建一个日期并将其向前移动一天：

```js
    > var d = new Date(2015, 0, 1);   
    > d.getTime();   
    1420099200000   
    > d.setTime(d.getTime() +
     1000 * 60 * 60 *   24);   
    1420185600000   
    > d.toLocaleString();   
    "Fri Jan 02 2015 00:00:00  
      GMT-0800 (PST)"   

```

|

| `getFullYear()``getUTCFullYear()``setFullYear(year, month, date)``setUTCFullYear(year, month, date)` | 使用本地或 UTC 时间获取或设置完整年份。还有`getYear()`，但它不符合 Y2K 标准，因此请改用`getFullYear()`：

```js
    > var d = new Date(2015, 0, 1);   
    > d.getYear();   
    115   
    > d.getFullYear();   
    2015   
    > d.setFullYear(2020);   
    1577865600000   
    > d;   
    Wed Jan 01 2020 00:00:00 GMT-0800 
      (PST)   

```

|

| `getMonth()``getUTCMonth()``setMonth(month, date)``setUTCMonth(month, date)` | 获取或设置月份，从 0 开始（一月）：

```js
    > var d = new Date(2015, 0, 1);   
    > d.getMonth();   
    0   
    > d.setMonth(11);   
    1448956800000   
    > d.toLocaleDateString();   
    "12/1/2015"   

```

|

| `getDate()``getUTCDate()``setDate(date)``setUTCDate(date)` | 获取或设置月份的日期。

```js
    > var d = new Date(2015, 0, 1);   
    > d.toLocaleDateString();   
    "1/1/2015"   
    > d.getDate();   
    1   
    > d.setDate(31);   
    1422691200000   
    > d.toLocaleDateString();   
    "1/31/2015"   

```

|

| `getHours()``getUTCHours()``setHours(hour, min, sec, ms)``setUTCHours(hour, min, sec, ms)``getMinutes()``getUTCMinutes()``setMinutes(min, sec, ms)``setUTCMinutes(min, sec, ms)``getSeconds()``getUTCSeconds()``setSeconds(sec, ms)``setUTCSeconds(sec, ms)``getMilliseconds()``getUTCMilliseconds()``setMilliseconds(ms)``setUTCMilliseconds(ms)` | 获取/设置小时，分钟，秒，毫秒，全部从`0`开始：

```js
    > var d = new Date(2015, 0, 1);   
    > d.getHours() + ':' + d.getMinutes();   
    "0:0"   
    > d.setMinutes(59);   
    1420102740000   
    > d.getHours() + ':' + d.getMinutes();   
    "0:59"   

```

|

| `getTimezoneOffset()` | 返回本地时间和世界标准时间（UTC）之间的差异，以分钟为单位。例如，太平洋标准时间（PST）和 UTC 之间的差异：

```js
    > new   Date().getTimezoneOffset();   
    480   
    > 420 / 60; // hours   
    8   

```

|

| `getDay()``getUTCDay()` | 返回一周的第几天，从 0 开始（星期日）：

```js
    > var d = new Date(2015, 0, 1);      
    > d.toDateString();   
    "Thu Jan 01 2015"   
    > d.getDay();   
    4   
    > var d = new Date(2015, 0, 4);      
    > d.toDateString();   
    "Sat Jan 04 2015"   
    > d.getDay();   
    0   

```

|

## ECMAScript 5 对 Date 的补充

以下是对`Date`构造函数的补充：

| **属性/方法** | **描述** |
| --- | --- |

| `Date.now()` | 获取当前时间戳的便捷方式：

```js
    > Date.now() === new   Date().getTime();   
    true   

```

|

| `Date.prototype.toISOString()` | 另一个`toString()`：

```js
    > var d = new Date(2015, 0, 1);   
    > d.toString();   
    "Thu Jan 01 2015 00:00:00 GMT-0800
     (PST)" 
    > d.toUTCString();   
    "Thu, 01 Jan 2015 08:00:00   GMT"   
    > d.toISOString();   
    "2015-01-01T00:00:00.000Z"   

```

|

| `Date.prototype.toJSON()` | 被`JSON.stringify()`使用（参见本附录的末尾），并返回与`toISOString()`相同的值：

```js
    > var d = new Date();   
    > d.toJSON() === d.toISOString();   
    true   

```

|

# Math

`Math`与其他内置对象不同，因为它不能用作构造函数来创建对象。它只是一组静态函数和常量。以下是一些示例，以说明区别：

```js
    > typeof Date.prototype; 
    "object" 
    > typeof Math.prototype; 
    "undefined" 
    > typeof String; 
    "function" 
    > typeof Math; 
    "object" 

```

## Math 对象的成员

以下是`Math`对象的成员：

| **属性/方法** | **描述** |
| --- | --- |

| `Math.E``Math.LN10``Math.LN2``Math.LOG2E``Math.LOG10E``Math.PI``Math.SQRT1_2``Math.SQRT2` | 这些是一些有用的数学常数，都是只读的。以下是它们的值：

```js
    > Math.E;   
    2.718281828459045   
    > Math.LN10;   
    2.302585092994046   
    > Math.LN2;   
    0.6931471805599453   
    > Math.LOG2E;   
    1.4426950408889634   
    > Math.LOG10E;   
    0.4342944819032518   
    > Math.PI;   
    3.141592653589793   
    > Math.SQRT1_2;   
    0.7071067811865476   
    > Math.SQRT2;   
    1.4142135623730951   

```

|

| `Math.acos(x)``Math.asin(x)``Math.atan(x)``Math.atan2(y, x)``Math.cos(x)``Math.sin(x)``Math.tan(x)` | 三角函数 |
| --- | --- |

| `Math.round(x)``Math.floor(x)``Math.ceil(x)` | `round()`返回最接近的整数，`ceil()`向上舍入，`floor()`向下舍入：

```js
    > Math.round(5.5);   
    6   
    > Math.floor(5.5);   
    5   
    > Math.ceil(5.1);   
    6   

```

|

| `Math.max(num1, num2, num3, ...)``Math.min(num1, num2, num3, ...)` | `max()`返回传递给它们作为参数的数字中的最大值，`min()`返回传递给它们作为参数的数字中的最小值。如果至少有一个输入参数是`NaN`，结果也是`NaN`：

```js
    > Math.max(4.5, 101, Math.PI);   
    101   
    > Math.min(4.5, 101, Math.PI);   
    3.141592653589793   

```

|

| `Math.abs(x)` | 绝对值：

```js
    > Math.abs(-101);   
    101   
    > Math.abs(101);   
    101   

```

|

| `Math.exp(x)` | 指数函数：`Math.E`的`x`次方：

```js
    > Math.exp(1) === Math.E;   
    true   

```

|

| `Math.log(x)` | `x`的自然对数：

```js
    > Math.log(10) === Math.LN10;   
 **true**

```

|

| `Math.sqrt(x)` | `x`的平方根：

```js
    > Math.sqrt(9);   
    3   
    > Math.sqrt(2) === Math.SQRT2;   
    true   

```

|

`Math.pow(x, y)` | `x`的`y`次方：

```js
    > Math.pow(3, 2);   
    9   

```

|

| `Math.random()` | 0 到 1 之间的随机数（包括 0）。

```js
    > Math.random();   
    0.8279076443185321   
    For an random integer in a range,
     say between 10 and 100:   
    > Math.round(Math.random() * 90   + 10);   
    79   

```

|

# RegExp

您可以使用`RegExp()`构造函数创建正则表达式对象。将表达式模式作为第一个参数传递，将模式修饰符作为第二个参数传递：

```js
    > var re = new RegExp('[dn]o+dle', 'gmi'); 

```

这匹配"noodle"，"doodle"，"doooodle"等。这相当于使用正则表达式字面量：

```js
    > var re = ('/[dn]o+dle/gmi'); // recommended 

```

第四章，*对象*和附录 D，*正则表达式*包含有关正则表达式和模式的更多信息。

## RegExp.prototype 的成员

以下是`RegExp.prototype`的成员：

| **属性/方法** | **描述** |
| --- | --- |
| `global` | 如果在创建`regexp`对象时设置了`g`修饰符，则为只读的`true`。 |
| `ignoreCase` | 只读。如果在创建`regexp`对象时设置了`i`修饰符，则为`true`。 |
| `multiline` | 只读。如果在创建`regexp`对象时设置了`m`修饰符，则为`true` |

| `lastIndex` | 包含下一个匹配应该开始的字符串中的位置。`test()`和`exec()`在成功匹配后设置这个位置。只有在使用了`g`（全局）修饰符时才相关：

```js
    > var re = /[dn]o+dle/g;   
    > re.lastIndex;   
    0   
    > re.exec("noodle doodle");   
    ["noodle"]   
    > re.lastIndex;   
    6   
    > re.exec("noodle doodle");   
    ["doodle"]   
    > re.lastIndex;   
    13   
    > re.exec("noodle doodle");   
    null   
    > re.lastIndex;   
    0   

```

|

| `source` | 只读。返回正则表达式模式（不包括修饰符）：

```js
    > var re = /[nd]o+dle/gmi;   
    > re.source;   
    "[nd]o+dle"   

```

|

| `exec(string)` | 用正则表达式匹配输入字符串。成功匹配时返回一个包含匹配和任何捕获组的数组。使用`g`修饰符时，它匹配第一个出现并设置`lastIndex`属性。没有匹配时返回`null`：

```js
    > var re = /([dn])(o+)dle/g;   
    > re.exec("noodle doodle");   
    ["noodle", "n",   "oo"]   
    > re.exec("noodle doodle");   
    ["doodle", "d",   "oo"]   

```

`exec()`返回的数组有两个额外的属性：index（匹配的位置）和 input（被搜索的输入字符串）。

| `test(string)` | 与`exec()`相同，但只返回`true`或`false`：

```js
    > /noo/.test('Noodle');   
    false   
    > /noo/i.test('Noodle');   
    true   

```

|

# 错误对象

错误对象是由环境（浏览器）或您的代码创建的：

```js
    > var e = new Error('jaavcsritp is _not_ how you spell it'); 
    > typeof e; 
    "object" 

```

除了`Error`构造函数，还存在六个额外的构造函数，它们都继承自`Error`：

+   EvalError

+   RangeError

+   ReferenceError

+   SyntaxError

+   TypeError

+   URIError

## 错误原型成员

以下是`Error.prototype`的成员：

| **属性** | **描述** |
| --- | --- |

`name` | 用于创建对象的错误构造函数的名称：

```js
    > var e = new EvalError('Oops');   
    > e.name;   
    "EvalError"   

```

|

| `message` | 附加的错误信息：

```js
    > var e = new Error('Oops...   again');   
    > e.message;   
    "Oops... again"   

```

|

# JSON

JSON 对象是 ES5 中的新内容。它不是一个构造函数（类似于`Math`），只有两个方法：`parse()`和`stringify()`。对于不原生支持 JSON 的 ES3 浏览器，可以使用来自[`json.org`](http://json.org)的“shim”。

**JSON**代表**JavaScript 对象表示法**。它是一种轻量级的数据交换格式。它是 JavaScript 的一个子集，只支持原始类型、对象字面量和数组字面量。

## JSON 对象的成员

以下是`JSON`对象的成员：

| **方法** | **描述** |
| --- | --- |

`parse(text, callback)` | 接受一个 JSON 编码的字符串并返回一个对象：

```js
    > var data = '{"hello":   1, "hi": [1, 2, 3]}';   
    > var o = JSON.parse(data);   
    > o.hello;   
    1   
    > o.hi;   
    [1, 2, 3]   

```

可选的回调让你提供自己的函数，可以检查和修改结果。回调接受`key`和`value`参数，可以修改`value`或删除它（通过返回`undefined`）。

```js
    > function callback(key, value)   {   
        console.log(key, value);   
        if (key === 'hello') {   
          return 'bonjour';   
        }   
        if (key === 'hi') {   
          return undefined;   
        }   
        return value;   
      }   

    > var o = JSON.parse(data, callback);   
    hello 1   
    0 1   
    1 2   
    2 3   
    hi [1, 2, 3]   
    Object {hello: "bonjour"}   
    > o.hello;   
    "bonjour"   
    > 'hi' in o;   
    false   

```

|

| `stringify(value, callback, white)` | 接受任何值（通常是对象或数组）并将其编码为 JSON 字符串。

```js
    > var o = {   
    hello: 1,    
    hi: 2,    
    when: new Date(2015, 0, 1)   
   };   

    > JSON.stringify(o);   
   "{"hello":1,"hi":2,"when":
    "2015-01-01T08:00:00.000Z"}"   

```

第二个参数让你提供一个回调（或一个白名单数组）来自定义返回值。白名单包含你感兴趣的键：

```js
    JSON.stringify(o, ['hello', 'hi']);   
    "{"hello":1,"hi":2}"   

```

最后一个参数帮助你获得可读的版本。你可以将空格的数量指定为字符串或数字：

```js
    > JSON.stringify(o, null, 4);   
    "{   
    "hello": 1,   
    "hi": 2,   
    "when": "2015-01-01T08:00:00.000Z"   
    }"   

```

|
