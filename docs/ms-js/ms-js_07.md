# 第七章：ECMAScript 6

到目前为止，我们已经对 JavaScript 编程语言进行了详细的了解。我相信您一定对语言的核心有了深刻的了解。到目前为止，我们所了解的都是按照**ECMAScript** **5**（**ES5**）标准进行的。**ECMAScript 6**（**ES6**）或**ECMAScript 2015**（**ES2015**）是 ECMAScript 标准的最新版本。这个标准在不断发展，最后一次修改是在 2015 年 6 月。ES2015 在其范围和推荐方面都具有重要意义，并且 ES2015 的推荐正在大多数 JavaScript 引擎中得到实施。这对我们来说是个好消息。ES6 引入了大量的新特性和帮助器，这些新特性和帮助器极大地丰富了语言。ECMAScript 标准的快速发展使得浏览器和 JavaScript 引擎支持新特性变得有些困难。同时，大多数程序员实际上需要编写可以在旧浏览器上运行的代码。臭名昭著的 Internet Explorer 6 曾经是世界上使用最广泛的浏览器。确保您的代码与尽可能多的浏览器兼容是一项艰巨的任务。因此，虽然您想使用 ES6 下一组酷炫的特性，但您必须考虑这样一个事实：许多 ES6 特性可能不被最流行的浏览器或 JavaScript 框架支持。

这看起来可能是一个糟糕的情况，但事情并没有那么糟糕。**Node.js**使用支持大多数 ES6 特性的最新版 V8 引擎。Facebook 的**React**也支持它们。Mozilla Firefox 和 Google Chrome 是目前使用最广泛的两种浏览器，它们支持大多数 ES6 特性。

为了避免这些陷阱和不可预测性，提出了一些解决方案。这些解决方案中最有用的是 polyfills/shims 和转译器。

# Shims 或 polyfills

Polyfills（也称为 shims）是一种定义新版本环境中兼容旧版本环境的行为的模式。有一个很棒的 ES6 shims 集合叫做**ES6 shim**（[`github.com/paulmillr/es6-shim/`](https://github.com/paulmillr/es6-shim/)）；我强烈建议您研究这些 shims。从 ES6 shim 集合中，考虑以下 shim 的示例。

根据 ECMAScript 2015（ES6）标准，`Number.isFinite()`方法用于确定传递的值是否是一个有限数字。它的等效 shim 可能如下所示：

```js
var numberIsFinite = Number.isFinite || function isFinite(value) {
  return typeof value === 'number' && globalIsFinite(value);
};
```

这个 shim 首先检查`Number.isFinite()`方法是否可用；如果不可用，则用实现来*填充*它。这是一种非常巧妙的技巧，用于填补规范中的空白。shims 不断升级，加入新特性，因此，在项目中保留最新版本的 shims 是一个明智的策略。

### 注意

`endsWith()` polyfill 的详细说明可以在 [`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/endsWith`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/endsWith) 找到。`String.endsWith()` 是 ES6 的一部分，但可以很容易地为 pre-ES6 环境进行 polyfill。

然而，shims 不能 polyfill 语法变化。为此，我们可以考虑转译器作为一个选项。

# 转译器

转译是一种结合了编译和转换的技术。想法是写 ES6 兼容的代码，并使用一个将这种代码转译成有效且等效的 ES5 代码的工具。我们将探讨最完整且流行的 ES6 转译器，名为 **Babel** ([`babeljs.io/`](https://babeljs.io/)）。

Babel 可以用多种方式使用。你可以把它安装为 node 模块，从命令行调用它，或者在你的网页中导入它作为一个脚本。Babel 的设置非常全面且文档齐全，详情请查看 [`babeljs.io/docs/setup/`](https://babeljs.io/docs/setup/)。Babel 还有一个很棒的 **Read-Eval-Print-Loop** (**REPL**)。在本章中，我们将使用 Babel REPL 来进行大多数示例。深入理解 Babel 可以用的各种方式超出了本书的范围。然而，我建议你开始将 Babel 作为你开发工作流程的一部分来使用。

我们将在本章覆盖 ES6 规范的最重要部分。如果可能的话，你应该探索 ES6 的所有特性，并让它们成为你开发工作流程的一部分。

# ES6 语法变化

ES6 为 JavaScript 带来了重大的语法变化。这些变化需要仔细学习和适应。在本节中，我们将学习一些最重要的语法变化，并了解如何使用 Babel 立即在你的代码中使用这些新的构造。

## 块级作用域

我们之前讨论过，JavaScript 中的变量是函数作用域的。在嵌套作用域中创建的变量对整个函数都是可用的。几种编程语言为你提供了一个默认的块作用域，其中在任何代码块（通常由 `{}` 限定）中声明的变量（可用）仅限于这个块。为了在 JavaScript 中实现类似的块作用域，一个普遍的方法是使用立即调用函数表达式（**IIFE**）。考虑以下示例：

```js
var a = 1;
(function blockscope(){
    var a = 2;
    console.log(a);   // 2
})();
console.log(a);       // 1
```

使用 IIFE，我们为 `a` 变量创建了一个块作用域。当在 IIFE 中声明一个变量时，它的作用域被限制在函数内部。这是模拟块作用域的传统方式。ES6 支持不使用 IIFE 的块作用域。在 ES6 中，你可以用 `{}` 定义的块来包含任何语句（或语句）。用 `var` 声明变量，你可以使用 `let` 来定义块作用域。前一个示例可以用 ES6 块作用域重写如下：

```js
"use strict";
var a = 1;
{
  let a = 2;
  console.log( a ); // 2
}
console.log( a ); // 1
```

在 JavaScript 中使用独立的方括号`{}`可能看起来很奇怪，但这种约定在许多语言中用来创建块级作用域是非常普遍的。块级作用域同样适用于其他构造，比如`if { }`或`for (){ }`。

当你以这种方式使用块级作用域时，通常最好将变量声明放在块的最顶部。`var`和`let`声明的变量之间的一个区别是，用`var`声明的变量附着在整个函数作用域上，而用`let`声明的变量附着在块级作用域上，并且在块中出现之前它们不会被初始化。因此，你不能在声明之前访问用`let`声明的变量，而对于用`var`声明的变量，顺序并不重要：

```js
function fooey() {
  console.log(foo); // ReferenceError
  let foo = 5000;
}
```

`let`的一个特定用途是在 for 循环中。当我们使用`var`声明一个变量在 for 循环中时，它是在全局或父作用域中创建的。我们可以在 for 循环作用域中通过使用`let`声明一个变量来创建一个块级作用域的变量。考虑以下示例：

```js
for (let i = 0; i<5; i++) {
  console.log(i);
}
console.log(i); // i is not defined
```

由于`i`是通过`let`创建的，它在`for`循环中是有作用域的。你可以看到，这个变量在作用域之外是不可用的。

在 ES6 中，块级作用域的另一个用途是创建常量。使用`const`关键字，你可以在块级作用域中创建常量。一旦值被设置，你就无法改变这样一个常量的值：

```js
if(true){
  const a=1;
  console.log(a);
  a=100;  ///"a" is read-only, you will get a TypeError
}
```

常量必须在声明时初始化。同样的块级作用域规则也适用于函数。当一个函数在块内部声明时，它只能在那个作用域内使用。

## 默认参数

默认值是非常常见的。你总是为传递给函数的参数或你初始化的变量设置一些默认值。你可能见过类似下面的代码：

```js
function sum(a,b){
  a = a || 0;
  b = b || 0;
  return (a+b);
}
console.log(sum(9,9)); //18
console.log(sum(9));   //9
```

在这里，我们使用`||`（或运算符）来默认变量`a`和`b`如果没有在调用函数时提供值，则默认为`0`。在 ES6 中，你有了一种标准的默认函数参数的方法。之前的示例可以重写如下：

```js
function sum(a=0, b=0){
  return (a+b);
}
console.log(sum(9,9)); //18
console.log(sum(9));   //9
```

你可以将任何有效的表达式或函数调用作为默认参数列表的一部分传递。

## 展开和剩余

ES6 有一个新的操作符，`…`。根据它的使用方式，它被称为`展开`或`剩余`。让我们看一个简单的例子：

```js
function print(a, b){
  console.log(a,b);
}
print(...[1,2]);  //1,2
```

这里发生的事情是，当你在数组（或可迭代对象）前加上`…`时，它*展开*了数组中的元素，将其分别赋值给函数参数中的独立变量。当数组被展开时，`a`和`b`这两个函数参数被赋予了数组中的两个值。在展开数组时，会忽略多余的参数：

```js
print(...[1,2,3 ]);  //1,2
```

这仍然会打印`1`和`2`，因为这里只有两个功能参数可用。展开也可以用在其他地方，比如数组赋值：

```js
var a = [1,2];
var b = [ 0, ...a, 3 ];
console.log( b ); //[0,1,2,3]
```

`…`操作符还有一个与我们刚才看到完全相反的用途。不是展开值，而是用同一个操作符将它们聚集到一起：

```js
function print (a,...b){
  console.log(a,b);
}
console.log(print(1,2,3,4,5,6,7));  //1 [2,3,4,5,6,7]
```

在这种情况下，变量`b`取剩余的值。变量`a`取第一个值作为`1`，变量`b`取剩余的值作为一个数组。

## 解构

如果你在函数式语言如**Erlang**上工作过，你会理解模式匹配的概念。JavaScript 中的解构与之一致。解构允许你使用模式匹配将值绑定到变量。考虑以下示例：

```js
var [start, end] = [0,5];
for (let i=start; i<end; i++){
  console.log(i);
}
//prints - 0,1,2,3,4
```

我们使用数组解构来分配两个变量：

```js
var [start, end] = [0,5];
```

如前所示的例子，我们希望模式在第一个值分配给第一个变量（`start`）和第二个值分配给第二个变量（`end`）时匹配。考虑以下片段，看看数组元素解构是如何工作的：

```js
function fn() {
  return [1,2,3];
}
var [a,b,c]=fn();
console.log(a,b,c); //1 2 3
//We can skip one of them
var [d,,f]=fn();
console.log(d,f);   //1 3
//Rest of the values are not used
var [e,] = fn();
console.log(e);     //1
```

让我们讨论一下对象解构是如何工作的。假设你有一个返回对象的函数`f`，它按照如下方式返回：

```js
function f() {
  return {
    a: 'a',
    b: 'b',
    c: 'c'
  };
}
```

当我们解构这个函数返回的对象时，我们可以使用我们之前看到的类似语法；不同的是，我们使用`{}`而不是`[]`：

```js
var { a: a, b: b, c: c } = f();
console.log(a,b,c); //a b c
```

与数组类似，我们使用模式匹配将变量分配给函数返回的相应值。如果你使用与匹配的变量相同的变量，这种写法会更短。下面的例子恰到好处：

```js
var { a,b,c } = f();
```

然而，你通常会使用与函数返回的变量不同的变量名。重要的是要记住，语法是*源：目标*，而不是通常的*目标：源*。仔细观察下面的例子：

```js
//this is target: source - which is incorrect
var { x: a, x: b, x: c } = f();
console.log(x,y,z); //x is undefined, y is undefined z is undefined
//this is source: target - correct
var { a: x, b: y, c: z } = f();
console.log(x,y,z); // a b c
```

这是*目标 = 源*赋值方式的相反，因此需要一些时间来适应。

## 对象字面量

对象字面量在 JavaScript 中无处不在。你会认为没有改进的余地。然而，ES6 也想改进这一点。ES6 引入了几种快捷方式，以围绕对象字面量创建紧凑的语法：

```js
var firstname = "Albert", lastname = "Einstein",
  person = {
    firstname: firstname,
    lastname: lastname
  };
```

如果你打算使用与分配变量相同的属性名，你可以使用 ES6 的紧凑属性表示法：

```js
var firstname = "Albert", lastname = "Einstein",
  person = {
    firstname,
    lastname
  };
```

同样地，你是这样给属性分配函数的：

```js
var person = {
  getName: function(){
    // ..
  },
  getAge: function(){
    //..
  }
}
```

与其前的行相比，你可以这样说：

```js
var person = {
  getName(){
    // ..
  },
  getAge(){
    //..
  }
}
```

## 模板字面量

我相信你肯定做过如下的事情：

```js
function SuperLogger(level, clazz, msg){
  console.log(level+": Exception happened in class:"+clazz+" - Exception :"+ msg);
}
```

这是一种非常常见的替换变量值以形成字符串字面量的方法。ES6 为您提供了一种新的字符串字面量类型，使用反引号（`）：

函数 SuperLogger(level, clazz, msg){

console.log(`${level} : 在类: ${clazz} 中发生异常 - 异常 : {$msg}`);

}

` around 一个字符串字面量。在这个字面量内部，任何`${..}`形式的表达式都会立即解析。这种解析称为插值。在解析时，变量的值替换了`${}`内的占位符。结果字符串只是普通字符串，占位符被实际变量值替换。

使用字符串插值，你也可以将字符串拆分为多行，如下面的代码所示（与 Python 非常相似）：

```js
var quote =
`Good night, good night! 
Parting is such sweet sorrow, 
that I shall say good night 
till it be morrow.`;
console.log( quote );
```

你可以使用函数调用或有效的 JavaScript 表达式作为字符串插值的一部分：

```js
function sum(a,b){
  console.log(`The sum seems to be ${a + b}`);
}
sum(1,2); //The sum seems to be 3
```

模板字符串的最后一种变体称为**带标签的模板字符串**。想法是用一个函数来修改模板字符串。考虑以下示例：

```js
function emmy(key, ...values){
  console.log(key);
  console.log(values);
}
let category="Best Movie";
let movie="Adventures in ES6";
emmy`And the award for ${category} goes to ${movie}`;

//["And the award for "," goes to ",""]
//["Best Movie","Adventures in ES6"]
```

当我们用模板字面量调用`emmy`函数时，最奇怪的是这并不是传统函数调用的语法。我们不是写`emmy()`；我们只是在标记字面量。当这个函数被调用时，第一个参数是所有普通字符串（插值表达式之间的字符串）的数组。第二个参数是所有插值表达式被求值和存储的数组。

这意味着标签函数实际上可以改变结果模板标签：

```js
function priceFilter(s, ...v){
  //Bump up discount
  return s[0]+ (v[0] + 5);
}
let default_discount = 20;
let greeting = priceFilter `Your purchase has a discount of ${default_discount} percent`;
console.log(greeting);  //Your purchase has a discount of 25
```

正如你所看到的，我们在标签函数中修改了折扣的值并返回了修改后的值。

## 映射和集合

ES6 引入了四种新的数据结构：**Map**、**WeakMap**、**Set**和**WeakSet**。我们之前讨论过，对象是 JavaScript 中创建键值对的常用方式。对象的缺点是你不能使用非字符串值作为键。以下片段演示了如何在 ES6 中创建映射：

```js
let m = new Map();
let s = { 'seq' : 101 };

m.set('1','Albert');
m.set('MAX', 99);
m.set(s,'Einstein');

console.log(m.has('1')); //true
console.log(m.get(s));   //Einstein
console.log(m.size);     //3
m.delete(s);
m.clear();
```

你可以在声明它时初始化映射：

```js
let m = new Map([
  [ 1, 'Albert' ],
  [ 2, 'Douglas' ],
  [ 3, 'Clive' ],
]);
```

如果你想遍历映射中的条目，你可以使用`entries()`函数，它将返回一个迭代器。你可以使用`keys()`函数遍历所有键，使用`values()`函数遍历映射的值：

```js
let m2 = new Map([
    [ 1, 'Albert' ],
    [ 2, 'Douglas' ],
    [ 3, 'Clive' ],
]);
for (let a of m2.entries()){
  console.log(a);
}
//[1,"Albert"] [2,"Douglas"][3,"Clive"] 
for (let a of m2.keys()){
  console.log(a);
} //1 2 3
for (let a of m2.values()){
  console.log(a);
}
//Albert Douglas Clive
```

JavaScript 映射的一种变体是 WeakMap——WeakMap 不阻止其键被垃圾回收。WeakMap 的键必须是对象，而值可以是任意值。虽然 WeakMap 的行为与普通映射相同，但你不能遍历它，也不能清空它。这些限制背后有原因。由于映射的状态不能保证保持静态（键可能被垃圾回收），你不能确保正确的遍历。

使用 WeakMap 的情况并不多。大多数映射的使用可以用普通映射来实现。

虽然映射允许你存储任意值，但集合是唯一值的集合。映射和集合有类似的方法；然而，`set()`被替换为`add()`，而`get()`方法不存在。`get()`方法不存在的原因是因为集合有唯一值，所以你只关心集合是否包含一个值。考虑以下示例：

```js
let x = {'first': 'Albert'};
let s = new Set([1,2,'Sunday',x]);
//console.log(s.has(x));  //true
s.add(300);
//console.log(s);  //[1,2,"Sunday",{"first":"Albert"},300]

for (let a of s.entries()){
  console.log(a);
}
//[1,1]
//[2,2]
//["Sunday","Sunday"]
//[{"first":"Albert"},{"first":"Albert"}]
//[300,300]
for (let a of s.keys()){
  console.log(a);
}
//1
//2
//Sunday
//{"first":"Albert"}
//300
for (let a of s.values()){
  console.log(a);
}
//1
//2
//Sunday
//{"first":"Albert"}
//300
```

`keys()`和`values()`迭代器都返回集合中唯一值的列表。`entries()`迭代器生成一个条目数组列表，数组中的两个项目都是集合中的唯一值。集合的默认迭代器是其`values()`迭代器。

## 符号

ES6 引入了一种新数据类型叫做 Symbol。Symbol 是保证唯一且不可变的。Symbol 通常用作对象属性的标识符。它们可以被认为是唯一生成的 ID。你可以使用`Symbol()`工厂方法创建 Symbols——记住这不是一个构造函数，因此你不应该使用`new`操作符：

```js
let s = Symbol();
console.log(typeof s); //symbol
```

与字符串不同，Symbols 保证是唯一的，因此有助于防止名称冲突。有了 Symbols，我们有一个对每个人都有效的扩展机制。ES6 带有一些预定义的内置 Symbols，它们揭示了 JavaScript 对象值的各种元行为。

## 迭代器

迭代器在其他编程语言中已经存在很长时间了。它们提供了方便的方法来处理数据集合。ES6 引入了迭代器来处理同样的用例。ES6 的迭代器是一个具有特定接口的对象。迭代器有一个`next()`方法，它返回一个对象。返回的对象有两个属性——`value`（下一个值）和`done`（表示是否已经达到最后一个结果）。ES6 还定义了一个`Iterable`接口，描述了必须能够产生迭代器的对象。让我们看看一个数组，它是一个可迭代的，以及它能够产生的迭代器来消费其值：

```js
var a = [1,2];
var i = a[Symbol.iterator]();
console.log(i.next());      // { value: 1, done: false }
console.log(i.next());      // { value: 2, done: false }
console.log(i.next());      // { value: undefined, done: true }
```

正如你所见，我们是通过`Symbol.iterator()`访问数组的迭代器，并在其上调用`next()`方法来获取每个连续元素。`next()`方法调用返回`value`和`done`两者。当你在数组的最后一个元素之后调用`next()`时，你会得到一个未定义的值和`done: true`，这表明你已经遍历了整个数组。

## 对于..of 循环

ES6 添加了一种新的迭代机制，形式为`for..of`循环，它遍历由迭代器产生的值集合。

我们遍历的`for..of`值是一个可迭代的。

让我们比较一下`for..of`和`for..in`：

```js
var list = ['Sunday','Monday','Tuesday'];
for (let i in list){
  console.log(i);  //0 1 2
}
for (let i of list){
  console.log(i);  //Sunday Monday Tuesday
}
```

正如你所见，使用`for..in`循环，你可以遍历`list`数组的索引，而`for..of`循环让你遍历存储在`list`数组中的值。

## 箭头函数

ECMAScript 6 最有趣的新特性之一是箭头函数。箭头函数，正如其名称所暗示的，是使用一种新语法定义的函数，该语法使用*箭头*（`=>`）作为语法的一部分。让我们首先看看箭头函数看起来如何：

```js
//Traditional Function
function multiply(a,b) {
  return a*b;
}
//Arrow
var multiply = (a,b) => a*b;
console.log(multiply(1,2)); //2
```

箭头函数定义包括参数列表（零个或多个参数，如果恰好有一个参数则周围是`( .. )`），后面跟着`=>`标记，后面跟着函数体。

如果函数体中有多个表达式，可以用`{ .. }`括起来。如果只有一个表达式，并且省略了周围的`{ .. }`，则在表达式前面有一个隐式的返回。你可以以几种不同的方式编写箭头函数。以下是最常用的几种：

```js
// single argument, single statement
//arg => expression;
var f1 = x => console.log("Just X");
f1(); //Just X

// multiple arguments, single statement
//(arg1 [, arg2]) => expression;
var f2 = (x,y) => x*y;
console.log(f2(2,2)); //4

// single argument, multiple statements
// arg => {
//     statements;
// }
var f3 = x => {
  if(x>5){
    console.log(x);
  }
  else {
    console.log(x+5);
  }
}
f3(6); //6

// multiple arguments, multiple statements
// ([arg] [, arg]) => {
//   statements
// }
var f4 = (x,y) => {
  if(x!=0 && y!=0){
    return x*y;
  }
}
console.log(f4(2,2));//4

// with no arguments, single statement
//() => expression;
var f5 = () => 2*2;
console.log(f5()); //4

//IIFE
console.log(( x => x * 3 )( 3 )); // 9
```

重要的是要记住，所有正常函数参数的特征都适用于箭头函数，包括默认值、解构和剩余参数。

箭头函数提供了一种方便且简洁的语法，给你的代码带来了非常*函数式编程*的风格。箭头函数之所以受欢迎，是因为它们通过从代码中删除 function、return 和{ .. }，提供了编写简洁函数的吸引力。然而，箭头函数是为了根本解决与 this 相关的编程中的一个特定且常见痛点而设计的。在正常的 ES5 函数中，每个新定义的函数都定义了自己的`this`值（在构造函数中是一个新对象，在严格模式函数调用中是`undefined`，如果函数作为*对象方法*调用，则是上下文对象等）。JavaScript 函数总是有自己的`this`，这阻止了你从回调内部访问例如周围方法中的`this`。为了理解这个问题，请考虑以下示例：

```js
function CustomStr(str){
  this.str = str;
}
CustomStr.prototype.add = function(s){   // --> 1
  'use strict';
  return s.map(function (a){             // --> 2
    return this.str + a;                 // --> 3
  });
};

var customStr = new CustomStr("Hello");
console.log(customStr.add(["World"])); 
//Cannot read property 'str' of undefined
```

在标记为`3`的行上，我们试图获取`this.str`，但匿名函数也有自己的`this`，它遮蔽了从行`1`来的方法中的`this`。为了在 ES5 中修复这个问题，我们可以将`this`赋值给一个变量，然后使用这个变量：

```js
function CustomStr(str){
  this.str = str;
}
CustomStr.prototype.add = function(s){   
  'use strict';
 var that = this;                       // --> 1
  return s.map(function (a){             // --> 2
    return that.str + a;                 // --> 3
  });
};

var customStr = new CustomStr("Hello");
console.log(customStr.add(["World"])); 
//["HelloWorld]
```

在标记为`1`的行上，我们将`this`赋值给一个变量`that`，在匿名函数中我们使用`that`变量，它将引用正确上下文中的`this`。

ES6 箭头函数具有词法`this`，这意味着箭头函数捕获了外层上下文的`this`值。我们可以如下将前面的函数转换为等效的箭头函数：

```js
function CustomStr(str){
  this.str = str;
}
CustomStr.prototype.add = function(s){ 
 return s.map((a)=> {
 return this.str + a;
 });
};
var customStr = new CustomStr("Hello");
console.log(customStr.add(["World"])); 
//["HelloWorld]
```

# 摘要

在本章中，我们讨论了几种重要特性，这些特性被添加到 ES6 语言中。这是一组令人兴奋的新语言特性和范式，并且，通过使用 polyfills 和 transpilers，你可以立即开始使用它们。JavaScript 是一种不断发展的语言，了解未来趋势非常重要。ES6 特性使 JavaScript 成为一个更加有趣和成熟的语言。在下一章中，我们将深入研究使用 jQuery 和 JavaScript 操纵浏览器的**文档对象模型**（**DOM**）和事件。
