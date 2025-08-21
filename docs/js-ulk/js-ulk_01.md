# 第一章： 深入 JavaScript 核心

你可能用了几年的 iPhone，自认为是个有经验的用户。同时，你在打字时按删除键逐个删除不需要的字符。然而，有一天你发现只需快速摇晃就能一次性删除整条信息。然后你可能会想为什么之前不知道这个技巧。编程也是一样。我们可能会对自己的代码相当满意，直到突然间遇到一个技巧或不太知名的语法特性，让我们重新考虑过去几年所做的全部工作。结果是我们本可以用更简洁、更可读、更可测试、更易维护的方式完成这些工作。所以假设你已经有一定的 JavaScript 经验；然而，这一章将为你提供改进代码的最佳实践。我们将涵盖以下主题：

+   使你的代码可读且具有表现力

+   掌握 JavaScript 中的多行字符串

+   以 ES5 的方式操作数组

+   以一种优雅、可靠、安全和快速的方式遍历对象

+   声明对象的最有效方式

+   了解 JavaScript 中的魔法方法

# 让你的代码可读且具有表现力

有许多实践和启发式方法可以使代码更具可读性、表现力和整洁性。我们稍后讨论这个话题，但在这里我们谈谈语法糖。这个术语意味着一种替代语法，使代码更具表现力和可读性。实际上，我们从一开始就有一些这样的东西在 JavaScript 中。例如，自增/自减和加法/减法赋值运算符继承自 C 语言。`*foo++*`是`*foo = foo + 1*`的语法糖，`*foo += bar*`是`*foo = foo + bar*`的简写形式。此外，还有一些同样的目的的小技巧。

JavaScript 对所谓的**短路**表达式应用逻辑运算。这意味着表达式是从左到右阅读的，但一旦在早期阶段确定了条件结果，表达式的尾部就不会被评估。如果我们有`true || false || false`，解释器会从第一个测试中知道结果无论如何都是`true`。所以`false || false`部分不会被评估，这就为创意开启了道路。

## 函数参数默认值

当我们需要为参数指定默认值时，可以这样操作：

```js
function stub( foo ) {
 return foo || "Default value";
}

console.log( stub( "My value" ) ); // My value
console.log( stub() ); // Default value
```

这里发生了什么？当`foo`是`true`（`not undefined`、`NaN`、`null`、`false`、`0`或`""`）时，逻辑表达式的结果就是`foo`，否则会评估到`Default value`，这就是最终结果。

从 ECMAScript 的第六版（JavaScript 语言的规格）开始，我们可以使用更优美的语法：

```js
function stub( foo = "Default value" ) {
 return foo;
}
```

## 条件调用

在编写代码时，根据条件缩短它：

```js
var age = 20;
age >= 18 && console.log( "You are allowed to play this game" );
age >= 18 || console.log( "The game is restricted to 18 and over" );
```

在前一个示例中，我们使用 AND（`&&`）操作符在左条件为真时调用`console.log`。OR（`||`）操作符相反，如果条件为`假`，则调用`console.log`。

我认为实践中最常见的情况是简写条件，只有在提供时函数才被调用：

```js
/**
* @param {Function} [cb] - callback
*/
function fn( cb ) {
 cb && cb();
};
```

以下是在此的一个更多示例：

```js
/**
* @class AbstractFoo
*/
AbstractFoo = function(){
 // call this.init if the subclass has init method
 this.init && this.init();
};
```

语法糖直到 CoffeeScript 的进步才完全进入 JavaScript 世界，CoffeeScript 是这种语言的一个子集，它源码编译（源码到源码编译）为 JavaScript。实际上，受 Ruby，Python 和 Haskell 启发的 CoffeeScript 为 JavaScript 开发者解锁了箭头函数，展开和其他语法。2011 年，Brendan Eich（JavaScript 的作者）承认 CoffeeScript 影响了他的 EcmaScript Harmony 的工作，该工作在今年夏天的 ECMA-262 6th edition specification 中最终完成。从市场营销的角度来看，规格编写者同意使用新的命名约定，将第 6 版称为 EcmaScript 2015，第 7 版称为 EcmaScript 2016。然而，社区已经习惯了缩写如 ES6 和 ES7。为了进一步避免混淆，在本书中，我们将用这些名称来指代规格。现在我们可以看看这对新的 JavaScript 有什么影响。

## 箭头函数

传统的函数表达式可能如下所示：

```js
function( param1, param2 ){ /* function body */ }
```

当使用箭头函数（也称为胖箭头函数）语法声明表达式时，我们将以更简洁的形式拥有这个 this，如下所示：

```js
( param1, param2 ) => { /* function body */ }
```

在我看来，这样做我们并没有得到太多。但是如果我们需要，比如说，一个数组方法的回调，传统形式如下：

```js
function( param1, param2 ){ return expression; }
```

现在等效的箭头函数变得更短了，如下所示：

```js
( param1, param2 ) => expression
```

我们可能这样在数组中进行过滤：

```js
// filter all the array elements greater than 2
var res = [ 1, 2, 3, 4 ].filter(function( v ){
 return v > 2;
})
console.log( res ); // [3,4]
```

使用数组函数，我们可以以更简洁的形式进行过滤：

```js
var res  = [ 1, 2, 3, 4 ].filter( v => v > 2 );
console.log( res ); // [3,4]
```

除了更短的方法声明语法外，箭头函数还带来了所谓的词法`this`。而不是创建自己的上下文，它使用周围对象的上下文，如下所示：

```js
"use strict";
/**
* @class View
*/   
let View = function(){
 let button = document.querySelector( "[data-bind=\"btn\"]" );
 /**
  * Handle button clicked event
  * @private 
  */
 this.onClick = function(){
   console.log( "Button clicked" );
 };
 button.addEventListener( "click", () => {
   // we can safely refer surrounding object members
   this.onClick(); 
 }, false );
}
```

在前一个示例中，我们为 DOM 事件（`click`）订阅了一个处理函数。在处理器的范围内，我们仍然可以访问视图上下文（`this`），因此我们不需要将处理函数绑定到外部作用域或通过闭包作为变量传递：

```js
var that = this;
button.addEventListener( "click", function(){
  // cross-cutting concerns
  that.onClick(); 
}, false );
```

## 方法定义

如前一部分所述，当声明小型的内联回调函数时，箭头函数非常方便，但总是为了更短的语法而使用它是有争议的。然而，ES6 除了箭头函数之外，还提供了新的替代方法定义语法。老式的方法声明可能如下所示：

```js
var foo = {
 bar: function( param1, param2 ) {
 }
}
```

在 ES6 中，我们可以摆脱函数关键字和冒号。所以前一条代码可以这样做：

```js
let foo = {
 bar ( param1, param2 ) {
 }
}
```

## 剩余操作符

另一种从 CoffeeScript 借用的语法结构作为剩余操作符（尽管在 CoffeeScript 中，这种方法被称为*splats*）来到了 JavaScript。

当我们有几个必需的函数参数和一个未知数量的剩余参数时，我们通常会这样做：

```js
"use strict";
var cb = function() {
 // all available parameters into an array
 var args = [].slice.call( arguments ),
     // the first array element to foo and shift
     foo = args.shift(),
     // the new first array element to bar and shift
     bar = args.shift();
 console.log( foo, bar, args );
};
cb( "foo", "bar", 1, 2, 3 ); // foo bar [1, 2, 3]
```

现在看看这段代码在 ES6 中变得多么有表现力：

```js
let cb = function( foo, bar, ...args ) {
 console.log( foo, bar, args );
}
cb( "foo", "bar", 1, 2, 3 ); // foo bar [1, 2, 3]
```

函数参数不是剩余操作符的唯一应用。例如，我们也可以在解构中使用它，如下所示：

```js
let [ bar, ...others ] = [ "bar", "foo", "baz", "qux" ];
console.log([ bar, others ]); // ["bar",["foo","baz","qux"]]
```

## 展开操作符

同样，我们也可以将数组元素展开为参数：

```js
let args = [ 2015, 6, 17 ],
   relDate = new Date( ...args );
console.log( relDate.toString() );  // Fri Jul 17 2015 00:00:00 GMT+0200 (CEST)
```

ES6 还提供了创建对象和继承的有表现力的语法糖，但我们将稍后在*声明对象的最有效方式*部分中 examine this。

# 掌握 JavaScript 中的多行字符串

多行字符串不是 JavaScript 的一个好部分。虽然它们在其他语言中很容易声明（例如，NOWDOC），但你不能只是将单引号或双引号的字符串保持在多行中。这会导致语法错误，因为 JavaScript 中的每一行都被认为是可能的命令。你可以用反斜杠来表示你的意图：

```js
var str = "Lorem ipsum dolor sit amet, \n\
consectetur adipiscing elit. Nunc ornare, \n\
diam ultricies vehicula aliquam, mauris \n\
ipsum dapibus dolor, quis fringilla leo ligula non neque";
```

这种方法基本有效。然而，一旦你漏掉了一个尾随空格，你就会得到一个语法错误，这不容易被发现。虽然大多数脚本代理支持这种语法，但它并不是 EcmaScript 规范的一部分。

在**EcmaScript for XML**（**E4X**）的时代，我们可以将纯 XML 赋值给一个字符串，这为这些声明打开了一条道路：

```js
var str = <>Lorem ipsum dolor sit amet, 
consectetur adipiscing 
elit. Nunc ornare </>.toString();
```

现在 E4X 已经被弃用，不再被支持。

## 字符串连接与数组连接

我们也可以使用字符串连接。它可能感觉笨拙，但它是安全的：

```js
var str = "Lorem ipsum dolor sit amet, \n" +
 "consectetur adipiscing elit. Nunc ornare,\n" +
 "diam ultricies vehicula aliquam, mauris \n" +
 "ipsum dapibus dolor, quis fringilla leo ligula non neque";
```

你可能会感到惊讶，但字符串连接比数组连接慢。所以以下技术会更快地工作：

```js
var str = [ "Lorem ipsum dolor sit amet, \n",
 "consectetur adipiscing elit. Nunc ornare,\n",
 "diam ultricies vehicula aliquam, mauris \n",
 "ipsum dapibus dolor, quis fringilla leo ligula non neque"].join( "" );
```

## 模板字面量

那么 ES6 呢？最新的 EcmaScript 规范引入了一种新的字符串字面量，模板字面量：

```js
var str = `Lorem ipsum dolor sit amet, \n
consectetur adipiscing elit. Nunc ornare, \n
diam ultricies vehicula aliquam, mauris \n
ipsum dapibus dolor, quis fringilla leo ligula non neque`;
```

现在这个语法看起来很优雅。但还有更多。模板字面量真的让我们想起了 NOWDOC。你可以在字符串中引用作用域内声明的任何变量：

```js
"use strict";
var title = "Some title",
   text = "Some text",
   str = `<div class="message">
<h2>${title}</h2>
<article>${text}</article>
</div>`;
console.log( str );
```

输出如下：

```js
<div class="message">
<h2>Some title</h2>
<article>Some text</article>
</div>
```

如果你想知道何时可以安全地使用这种语法，我有一个好消息告诉你——这个特性已经得到了（几乎）所有主要脚本代理的支持（[`kangax.github.io/compat-table/es6/`](http://kangax.github.io/compat-table/es6/)）。

## 通过转译器实现多行字符串

随着 ReactJS 的发展，Facebook 的 EcmaScript 语言扩展 JSX（[`facebook.github.io/jsx/`](https://facebook.github.io/jsx/)）现在已经真正获得了动力。显然受到之前提到的 E4X 的影响，他们提出了一种没有任何筛选的 XML 样内容的字符串字面量。这种类型支持类似于 ES6 模板的模板插值：

```js
"use strict";
var Hello = React.createClass({
 render: function() {
 return <div class="message">
<h2>{this.props.title}</h2>
<article>{this.props.text}</article>
</div>;
 }
});

React.render(<Hello title="Some title" text="Some text" />, node);
```

另一种声明多行字符串的方法是使用 CommonJS 编译器（[`dsheiko.github.io/cjsc/`](http://dsheiko.github.io/cjsc/)）。在解析'require'依赖关系时，编译器将任何非`.js`/`.json`内容转换为单行字符串：

**foo.txt**

```js
Lorem ipsum dolor sit amet,
consectetur adipiscing elit. Nunc ornare,
diam ultricies vehicula aliquam, mauris
ipsum dapibus dolor, quis fringilla leo ligula non neque
```

**consumer.js**

```js
var str = require( "./foo.txt" );
console.log( str );
```

您可以在第六章中找到 JSX 使用的示例，*大规模 JavaScript 应用程序架构*。

# 以 ES5 方式操作数组

几年前，当 ES5 特性的支持较差（ECMAScript 第五版于 2009 年最终确定）时，像 Underscore 和 Lo-Dash 这样的库变得非常流行，因为它们提供了一套全面的工具来处理数组/集合。今天，许多开发者仍然使用第三方库（包括 jQuery/Zepro）来处理诸如`map`、`filter`、`every`、`some`、`reduce`和`indexOf`等方法，而这些方法在 JavaScript 的本地形式中是可用的。是否需要这些库还取决于您的使用方式，但很可能您不再需要它们。让我们看看现在 JavaScript 中有什么。

## ES5 中的数组方法

`Array.prototype.forEach`可能是数组中最常用的方法。也就是说，它是`_.each`的本地实现，或者是例如`$.each`实用程序的实现。作为参数，`forEach`期望一个`iteratee`回调函数，可选的是您希望执行回调的上下文。它将元素值、索引和整个数组传递给回调函数。大多数数组操作方法都使用相同的参数语法。注意 jQuery 的`$.each`将回调参数顺序颠倒：

```js
"use strict";
var data = [ "bar", "foo", "baz", "qux" ];
data.forEach(function( val, inx ){
  console.log( val, inx ); 
});
```

`Array.prototype.map`通过转换给定数组的元素来生成一个新的数组：

```js
"use strict";
var data = { bar: "bar bar", foo: "foo foo" },
   // convert key-value array into url-encoded string
   urlEncStr = Object.keys( data ).map(function( key ){
     return key + "=" + window.encodeURIComponent( data[ key ] );
   }).join( "&" );

console.log( urlEncStr ); // bar=bar%20bar&foo=foo%20foo
```

`Array.prototype.filter`返回一个数组，该数组由满足回调条件的给定数组值组成：

```js
"use strict";
var data = [ "bar", "foo", "", 0 ],
   // remove all falsy elements
   filtered = data.filter(function( item ){
     return !!item;
   });

console.log( filtered ); // ["bar", "foo"]
```

`Array.prototype.reduce`/`Array.prototype.reduceRight`检索数组中值的产品。该方法期望一个回调函数和可选的初始值作为参数。回调函数接收四个参数：累积值、当前值、索引和原始数组。因此，我们可以通过当前值增加累积值（返回 acc += cur;）来实例化，从而得到数组值的和。

除了使用这些方法进行计算外，我们还可以连接字符串值或数组：

```js
"use strict";
var data = [[ 0, 1 ], [ 2, 3 ], [ 4, 5 ]],
   arr = data.reduce(function( prev, cur ) {
     return prev.concat( cur );
   }),
   arrReverse = data.reduceRight(function( prev, cur ) {
     return prev.concat( cur );
   });

console.log( arr ); //  [0, 1, 2, 3, 4, 5]
console.log( arrReverse ); // [4, 5, 2, 3, 0, 1]
```

`Array.prototype.some`测试给定数组中的任何一个（或一些）值是否满足回调条件：

```js
"use strict";
var bar = [ "bar", "baz", "qux" ],
   foo = [ "foo", "baz", "qux" ],
   /**
    * Check if a given context (this) contains the value
    * @param {*} val
    * @return {Boolean}
    */
   compare = function( val ){
     return this.indexOf( val ) !== -1; 
   };

console.log( bar.some( compare, foo ) ); // true
```

在这个例子中，我们检查`foo`数组中是否有任何一个柱状数组值是可用的。为了可测试性，我们需要将`foo`数组的引用传递给回调函数。这里我们将其作为上下文注入。如果我们需要传递更多的引用，我们会将它们推入一个键值对象中。

正如您可能注意到的，在这个例子中我们使用了`Array.prototype.indexOf`。这个方法的工作方式与`String.prototype.indexOf`相同。如果找到匹配项，则返回匹配项的索引，否则返回-1。

`Array.prototype.every`测试给定数组的每一个值是否满足回调条件：

```js
"use strict";
var bar = [ "bar", "baz" ],
   foo = [ "bar", "baz", "qux" ],
   /**
    * Check if a given context (this) contains the value
    * @param {*} val
    * @return {Boolean}
    */
   compare = function( val ){
     return this.indexOf( val ) !== -1; 
   };

console.log( bar.every( compare, foo ) ); // true
```

如果你仍然关心这些方法在像 IE6-7 这样老旧的浏览器中的支持情况，你可以简单地使用 [`github.com/es-shims/es5-shim`](https://github.com/es-shims/es5-shim) 来补丁它们。

## es6 中的数组方法

在 ES6 中，我们只获得了一些看起来像是现有功能快捷方式的新方法。

`Array.prototype.fill` 用给定值填充数组，如下所示：

```js
"use strict";
var data = Array( 5 );
console.log( data.fill( "bar" ) ); // ["bar", "bar", "bar", "bar", "bar"]
```

`Array.prototype.includes` 明确检查给定值是否存在于数组中。嗯，它和 `arr.indexOf( val ) !== -1` 是一样的，如下所示：

```js
"use strict";
var data = [ "bar", "foo", "baz", "qux" ];
console.log( data.includes( "foo" ) );
```

`Array.prototype.find` 过滤出符合回调条件的单个值。再次说明，这和 `Array.prototype.filter` 能获得的是一样的。唯一的区别是 filter 方法返回一个数组或者一个 null 值。在这种情况下，它返回一个包含单个元素的数组，如下所示：

```js
"use strict";
var data = [ "bar", "fo", "baz", "qux" ],
   match = function( val ){
     return val.length < 3;
   };
console.log( data.find( match ) ); // fo
```

# 优雅、可靠、安全、快速地遍历对象

当我们有一个键值对象（比如说选项）并且需要遍历它时，这是一个常见的情况。下面代码中展示了一种学术上的做法：

```js
"use strict";
var options = {
    bar: "bar",
    foo: "foo"
   },
   key;
for( key in options ) {
 console.log( key, options[ key] );
}
```

上述代码输出如下：

```js
bar bar
foo foo
```

现在让我们想象一下，你文档中加载的任何第三方库都增强了内置的 `Object`：

```js
Object.prototype.baz = "baz";
```

现在当我们运行我们的示例代码时，我们将得到一个额外的不需要的条目：

```js
bar bar
foo foo
baz baz
```

这个问题解决方案是众所周知的，我们必须使用 `Object.prototype.hasOwnProperty` 方法测试键：

```js
//…
for( key in options ) {
 if ( options.hasOwnProperty( key ) ) {
   console.log( key, options[ key] );
 }
}
```

## 安全快速地遍历键值对象

让我们面对现实吧——这个结构是笨拙的，需要优化（我们必须对每个给定的键执行 `hasOwnProperty` 测试）。幸运的是，JavaScript 有 `Object.keys` 方法，它可以获取所有枚举的自身（非继承）属性的字符串值。这让我们得到了一个数组，里面是我们期望的键，我们可以用 `Array.prototype.forEach` 等方式进行迭代：

```js
"use strict";
var options = {
    bar: "bar",
    foo: "foo"
   };
Object.keys( options ).forEach(function( key ){
 console.log( key, options[ key] );
});
```

除了优雅，我们这种方式还能得到更好的性能。为了看看我们获得了多少性能提升，你可以在不同的浏览器上运行这个在线测试，比如：[`codepen.io/dsheiko/pen/JdrqXa`](http://codepen.io/dsheiko/pen/JdrqXa)。

## 枚举数组对象

像 `arguments` 和 `nodeList`（`node.querySelectorAll`、`document.forms`）这样的对象看起来像数组，实际上它们并不是。和数组一样，它们有 `length` 属性，可以在 `for` 循环中进行迭代。以对象的形式，它们可以以前面提到的相同方式进行遍历。但它们没有任何数组操作方法（`forEach`、`map`、`filter`、`some` 等等）。事实是，我们可以很容易地将它们转换为数组，如下所示：

```js
"use strict";
var nodes = document.querySelectorAll( "div" ),
   arr = Array.prototype.slice.call( nodes );

arr.forEach(function(i){
 console.log(i);
});
```

上述代码甚至可以更短：

```js
arr = [].slice.call( nodes )
```

这是一个非常方便的解决方案，但看起来像是一个技巧。在 ES6 中，我们可以用一个专用方法进行相同的转换：

```js
arr = Array.from( nodes );
```

## es6 集合

ES6 引入了一种新类型的对象——可迭代对象。这些对象可以一次获取一个元素。它们与其他语言中的迭代器非常相似。除了数组，JavaScript 还接收了两个新的可迭代数据结构，`Set`和`Map`。`Set`是一个包含唯一值的集合：

```js
"use strict";
let foo = new Set();
foo.add( 1 );
foo.add( 1 );
foo.add( 2 );
console.log( Array.from( foo ) ); // [ 1, 2 ]

let foo = new Set(), 
   bar = function(){ return "bar"; };
foo.add( bar );
console.log( foo.has( bar ) ); // true
```

映射类似于键值对象，但键可以是任意值。这造成了区别。想象一下，我们需要编写一个元素包装器，提供类似 jQuery 的事件 API。通过使用`on`方法，我们不仅可以传递一个处理回调函数，还可以传递一个上下文（`this`）。我们通过`cb.bind(context)`将给定的回调绑定到上下文。这意味着`addEventListener`接收一个与回调不同的函数引用。那么我们如何取消订阅处理程序呢？我们可以通过一个由事件名称和`callback`函数引用组成的键将新引用存储在`Map`中：

```js
"use strict";
/**
* @class
* @param {Node} el
*/
let El = function( el ){
 this.el = el;
 this.map = new Map();
};
/**
* Subscribe a handler on event
* @param {String} event
* @param {Function} cb
* @param {Object} context
*/
El.prototype.on = function( event, cb, context ){
 let handler = cb.bind( context || this );
 this.map.set( [ event, cb ], handler );
 this.el.addEventListener( event, handler, false );
};
/**
* Unsubscribe a handler on event
* @param {String} event
* @param {Function} cb
*/

El.prototype.off = function( event, cb ){
 let handler = cb.bind( context ),
     key = [ event, handler ];
 if ( this.map.has( key ) ) {
 this.el.removeEventListener( event, this.map.get( key ) );
 this.map.delete( key );
 }
};
```

任何可迭代的对象都有方法，`keys`，`values`和`entries`，其中键与`Object.keys`相同，其他方法分别返回数组值和键值对数组。现在让我们看看我们如何遍历可迭代的对象：

```js
"use strict";
let map = new Map()
 .set( "bar", "bar" )
 .set( "foo", "foo" ),
   pair;
for ( pair of map ) {
 console.log( pair );
}

// OR 
let map = new Map([
   [ "bar", "bar" ],
   [ "foo", "foo" ],
]);
map.forEach(function( value, key ){
 console.log( key, value );
});
```

可迭代的对象有数组类似的操作方法。因此我们可以使用`forEach`。此外，它们可以通过`for...in`和`for...of`循环进行迭代。第一个获取索引，第二个获取值。

# 声明对象最有效的方法

我们在 JavaScript 中如何声明一个对象？如果我们需要一个命名空间，我们可以简单地使用一个对象字面量。但当我们需要一个对象类型时，我们需要三思采取什么方法，因为这会影响我们面向对象代码的可维护性。

## 古典方法

我们可以创建一个构造函数并将成员链接到其上下文：

```js
"use strict"; 
/**
 * @class
 */
var Constructor = function(){
   /**
   * @type {String}
   * @public
   */
   this.bar = "bar";
   /**
   * @public
   * @returns {String}
   */
   this.foo = function() {
    return this.bar;
   };
 },
 /** @type Constructor */
 instance = new Constructor();

console.log( instance.foo() ); // bar
console.log( instance instanceof Constructor ); // true
```

我们还可以将成员分配给构造函数原型。结果将与以下相同：

```js
"use strict";
/**
* @class
*/
var Constructor = function(){},
   instance;
/**
* @type {String}
* @public
*/
Constructor.prototype.bar = "bar";
/**
* @public
* @returns {String}
*/
Constructor.prototype.foo = function() {
 return this.bar;
};
/** @type Constructor */
instance = new Constructor();

console.log( instance.foo() ); // bar
console.log( instance instanceof Constructor ); // true
```

在第一种情况下，我们在构造函数体中混合了对象结构和构造逻辑。在第二种情况下，通过重复`Constructor.prototype`，我们违反了**不要重复自己**（**DRY**）原则。

## 私有状态的方法

那么我们还可以用其他方式做什么呢？我们可以通过构造函数函数返回一个对象字面量：

```js
"use strict";
/**
 * @class
 */
var Constructor = function(){
     /**
     * @type {String}
     * @private
     */
     var baz = "baz";
     return {
       /**
       * @type {String}
       * @public
       */
       bar: "bar",
       /**
       * @public
       * @returns {String}
       */
       foo: function() {
        return this.bar + " " + baz;
       }
     };
   },
   /** @type Constructor */
   instance = new Constructor();

console.log( instance.foo() ); // bar baz
console.log( instance.hasOwnProperty( "baz") ); // false
console.log( Constructor.prototype.hasOwnProperty( "baz") ); // false
console.log( instance instanceof Constructor ); // false
```

这种方法的优势在于，构造函数作用域内声明的任何变量都与返回的对象在同一个闭包中，因此，可以通过对象访问。我们可以将这些变量视为私有成员。坏消息是我们将失去构造函数原型。当构造函数在实例化过程中返回一个对象时，这个对象成为整个新表达式的结果。

## 原型链的继承

那么继承呢？古典方法会让子类型原型成为超类型实例：

```js
"use strict";
 /**
 * @class
 */
var SuperType = function(){
       /**
       * @type {String}
       * @public
       */
       this.foo = "foo";
     },
     /**
      * @class
      */
     Constructor = function(){
       /**
       * @type {String}
       * @public
       */
       this.bar = "bar";
     },
     /** @type Constructor */
     instance;

 Constructor.prototype = new SuperType();
 Constructor.prototype.constructor = Constructor;

 instance = new Constructor();
 console.log( instance.bar ); // bar
 console.log( instance.foo ); // foo
 console.log( instance instanceof Constructor ); // true
 console.log( instance instanceof SuperType ); // true  
```

你可能会遇到一些代码，其中实例化时使用`Object.create`而不是新操作符。在这里，你需要知道两者的区别。`Object.create`接受一个对象作为参数，并创建一个以传递的对象为原型的新对象。在某种意义上，这使我们想起了克隆。检查这个，你声明一个对象字面量（proto）并基于第一个对象使用`Object.create`创建一个新的对象（实例）。无论你现在对新生成对象做何更改，它们都不会反映在原始（proto）上。但是，如果你更改原始对象的属性，你会在派生对象（实例）中发现该属性已更改：

```js
"use strict";
var proto = {
 bar: "bar",
 foo: "foo"
}, 
instance = Object.create( proto );
proto.bar = "qux",
instance.foo = "baz";
console.log( instance ); // { foo="baz",  bar="qux"}
console.log( proto ); // { bar="qux",  foo="foo"}
```

## 通过`Object.create`继承原型

与新操作符相比，`Object.create`不调用构造函数。因此，当我们使用它来填充子类型的原型时，我们失去了位于`supertype`构造函数中的所有逻辑。这样，`supertype`构造函数永远不会被调用：

```js
// ...
SuperType.prototype.baz = "baz";
Constructor.prototype = Object.create( SuperType.prototype );
Constructor.prototype.constructor = Constructor;

instance = new Constructor();

console.log( instance.bar ); // bar
console.log( instance.baz ); // baz
console.log( instance.hasOwnProperty( "foo" ) ); // false
console.log( instance instanceof Constructor ); // true
console.log( instance instanceof SuperType ); // true
```

### 通过`Object.assign`继承原型

当寻找最优结构时，我希望通过对象字面量声明成员，但仍保留到原型的链接。许多第三方项目利用自定义函数(*extend*)将结构对象字面量合并到构造函数原型中。实际上，ES6 提供了`Object.assign`本地方法。我们可以像这样使用它：

```js
"use strict";
   /**
    * @class
    */
var SuperType = function(){
     /**
     * @type {String}
     * @public
     */
     this.foo = "foo";
   },
   /**
    * @class
    */
   Constructor = function(){
     /**
     * @type {String}
     * @public
     */
     this.bar = "bar";
   },
   /** @type Constructor */
   instance;

Object.assign( Constructor.prototype = new SuperType(), {
 baz: "baz"
});
instance = new Constructor();
console.log( instance.bar ); // bar
console.log( instance.foo ); // foo
console.log( instance.baz ); // baz
console.log( instance instanceof Constructor ); // true
console.log( instance instanceof SuperType ); // true
```

这看起来几乎就是所需的，除了有一点不便。`Object.assign`简单地将源对象的价值分配给目标对象，而不管它们的类型如何。所以如果你有一个源属性是一个对象（例如，一个`Object`或`Array`实例），目标对象接收一个引用而不是一个值。所以你必须在初始化时手动重置任何对象属性。

## 使用 ExtendClass 的方法

由 Simon Boudrias 提出的`ExtendClass`似乎是一个无懈可击的解决方案([`github.com/SBoudrias/class-extend`](https://github.com/SBoudrias/class-extend))。他的小型库暴露了带有**extend**静态方法的`Base`构造函数。我们使用这个方法来扩展这个伪类及其任何派生类：

```js
"use strict";
   /**
    * @class
    */
var SuperType = Base.extend({
     /**
      * @pulic
      * @returns {String}
      */
     foo: function(){ return "foo public"; },
     /**
      * @constructs SuperType
      */
     constructor: function () {}
   }),
   /**
    * @class
    */
   Constructor = SuperType.extend({
     /**
      * @pulic
      * @returns {String}
      */      
     bar: function(){ return "bar public"; }
   }, {
     /**
      * @static
      * @returns {String}
      */      
     bar: function(){ return "bar static"; }
   }),
   /** @type Constructor */
   instance = new Constructor();

console.log( instance.foo() ); // foo public
console.log( instance.bar() ); // bar public
console.log( Constructor.bar() ); // bar static
console.log( instance instanceof Constructor ); // true
console.log( instance instanceof SuperType ); // true
```

## es6 中的类

tc39（ECMAScript 工作组）对这个问题非常清楚，所以新的语言规范提供了额外的语法来结构对象类型：

```js
"use strict";
class AbstractClass {
 constructor() {
   this.foo = "foo";
 }
}
class ConcreteClass extends AbstractClass {
 constructor() {
   super();
   this.bar = "bar";
 }
 baz() {
   return "baz";
 }
}

let instance = new ConcreteClass();
console.log( instance.bar ); // bar
console.log( instance.foo ); // foo
console.log( instance.baz() ); // baz
console.log( instance instanceof ConcreteClass ); // true
console.log( instance instanceof AbstractClass ); // true
```

这个语法看起来是基于类的，但实际上这只是现有原型的语法糖。你可以检查`ConcreteClass`的类型，它会给你*function*，因为`ConcreteClass`是一个典型的构造器。所以我们在扩展`supertypes`时不需要任何技巧，不需要从子类型中引用`supertype`构造函数，并且我们有一个清晰可读的结构。然而，我们无法以现在的方法相同的 C 语言方式分配属性。这仍在 ES7 的讨论中([`esdiscuss.org/topic/es7-property-initializers`](https://esdiscuss.org/topic/es7-property-initializers))。此外，我们可以在类的正文中直接声明类的静态方法：

```js
class Bar {
 static foo() {
   return "static method";
 }
 baz() {
   return "prototype method";
 }
}
let instance = new Bar();
console.log( instance.baz() ); // prototype method
console.log( Bar.foo()) ); // static method
```

实际上，有很多在 JavaScript 社区的人认为新的语法是从原型面向对象方法的一种偏离。另一方面，ES6 类与大多数现有代码向后兼容。子类现在由语言支持，不需要额外的库来实现继承。我个人最喜欢的是，这种语法允许我们使代码更简洁、更易于维护。

# 如何——JavaScript 中的魔术方法

在 PHP 世界中，有诸如*重载方法*这样的概念，它们也被称为魔术方法（[`www.php.net/manual/en/language.oop5.overloading.php`](http://www.php.net/manual/en/language.oop5.overloading.php)）。这些方法允许我们在访问或修改一个方法的不存在属性时触发一个逻辑。在 JavaScript 中，我们控制对属性（值成员）的访问。想象我们有一个自定义的集合对象。为了保持 API 的一致性，我们想要有一个`length`属性，它包含集合的大小。所以我们就声明一个`getter`（获取长度），每当属性被访问时就会执行所需的计算。在尝试修改属性值时，设置器将抛出一个异常：

```js
"use strict";
var bar = {
 /** @type {[Number]} */
 arr: [ 1, 2 ],
 /**
  * Getter
  * @returns {Number}
  */
 get length () {
   return this.arr.length;
 },
 /**
  * Setter
  * @param {*} val
  */
 set length ( val ) {
   throw new SyntaxError( "Cannot assign to read only property 'length'" );
 }
};
console.log ( bar.length ); // 2
bar.arr.push( 3 );
console.log ( bar.length ); // 3
bar.length = 10; // SyntaxError: Cannot assign to read only property 'length'
```

如果我们想在现有对象上声明 getters/setters，我们可以使用以下方式：

```js
Object.defineProperty:
"use strict";
var bar = {
 /** @type {[Number]} */
 arr: [ 1, 2 ]
};

Object.defineProperty( bar, "length", {
 /**
  * Getter
  * @returns {Number}
  */
 get: function() {
   return this.arr.length;
 },
 /**
  * Setter
  */
 set: function() {
   throw new SyntaxError( "Cannot assign to read only property 'length'" );
 }
});

console.log ( bar.length ); // 2
bar.arr.push( 3 );
console.log ( bar.length ); // 3
bar.length = 10; // SyntaxError: Cannot assign to read only property 'length'
```

`Object.defineProperty`以及`Object.create`的第二个参数指定了属性配置（是否可枚举、可配置、不可变，以及如何访问或修改）。因此，我们可以通过将属性设置为只读来达到类似的效果：

```js
"use strict";
var bar = {};

Object.defineProperty( bar, "length", {
 /**
  * Data descriptor
  * @type {*}
  */
 value: 0,
 /**
  * Data descriptor
  * @type {Boolean}
  */
 writable: false
});

bar.length = 10; // TypeError: "length" is read-only
```

顺便说一下，如果你想要摆脱对象中的属性访问器，你可以简单地移除该属性：

```js
delete bar.length;
```

## ES6 类中的访问器

声明访问器的另一种方式是使用 ES6 类：

```js
"use strict";
/** @class */
class Bar {
 /** @constructs Bar */
 constructor() {
   /** @type {[Number]} */
   this.arr = [ 1, 2 ];
 }
 /**
  * Getter
  * @returns {Number}
  */
 get length() {
   return this.arr.length;
 }
 /**
  * Setter
  * @param {Number} val
  */
 set length( val ) {
    throw new SyntaxError( "Cannot assign to read only property 'length'" );
 }
}

let bar = new Bar();
console.log ( bar.length ); // 2
bar.arr.push( 3 );
console.log ( bar.length ); // 3
bar.length = 10; // SyntaxError: Cannot assign to read only property 'length'
```

除了公共属性，我们还可以控制对静态属性的访问：

```js
"use strict";

class Bar {
   /**
    * @static
    * @returns {String}
    */
   static get baz() {
       return "baz";
   }
}

console.log( Bar.baz ); // baz
```

## 控制对任意属性的访问

所有这些示例都展示了对已知属性的访问控制。然而，可能有一个情况，我想要一个具有类似于`localStorage`的变长接口的自定义存储。这必须是一个具有`getItem`方法以检索存储的值和`setItem`方法以设置它们的存储。此外，这必须与直接访问或设置伪属性（`val = storage.aKey`和`storage.aKey = "value"`）的方式相同。这可以通过使用 ES6 代理实现：

```js
"use strict";
/**
* Custom storage
*/
var myStorage = {
     /** @type {Object} key-value object */
     data: {},
     /**
      * Getter
      * @param {String} key
      * @returns {*}
      */
     getItem: function( key ){
       return this.data[ key ];
     },
     /**
      * Setter
      * @param {String} key
      * @param {*} val
      */
     setItem: function( key, val ){
       this.data[ key ] = val;
     }
   },
   /**
    * Storage proxy
    * @type {Proxy}
    */
   storage = new Proxy( myStorage, {
     /**
      * Proxy getter
      * @param {myStorage} storage
      * @param {String} key
      * @returns {*}
      */
     get: function ( storage, key ) {
       return storage.getItem( key );
     },
     /**
      * Proxy setter
      * @param {myStorage} storage
      * @param {String} key
      * @param {*} val
      * @returns {void}
      */
     set: function ( storage, key, val ) {
       return storage.setItem( key, val );
   }});

storage.bar = "bar";
console.log( myStorage.getItem( "bar" ) ); // bar
myStorage.setItem( "bar", "baz" );
console.log( storage.bar ); // baz
```

# 摘要

本章介绍了如何使用 JavaScript 核心特性达到最大效果的最佳实践和技巧。在下一章中，我们将讨论模块概念，并详细介绍作用域和闭包。下一章将解释作用域上下文及其操作方法。
