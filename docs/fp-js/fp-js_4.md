# 第四章：在 JavaScript 中实现函数式编程技术

紧紧抓住你的帽子，因为我们现在真的要进入函数式思维模式了。

在本章中，我们将做以下事情：

+   将所有核心概念整合成一个连贯的范式

+   全面致力于函数式风格时，探索函数式编程所提供的美

+   逐步推进函数式模式的逻辑进展

+   同时，我们将构建一个简单的应用程序，做一些非常酷的事情

在上一章中，当处理 JavaScript 的函数式库时，您可能已经注意到了一些概念，但在《第二章》《函数式编程基础》中没有提到。好吧，这是有原因的！组合、柯里化、部分应用等。让我们探讨为什么以及这些库是如何实现这些概念的。

函数式编程可以采用多种风格和模式。本章将涵盖许多不同的函数式编程风格：

+   数据泛型编程

+   大部分是函数式编程

+   函数响应式编程等

然而，本章将尽可能不偏向任何一种函数式编程风格。不过度倚重某种函数式编程风格，总体目标是展示有比通常被接受的正确和唯一的编码方式更好的方式。一旦你摆脱了对编写代码的先入为主的观念，你就可以随心所欲。当你只是出于喜欢而写代码，而不担心符合传统的做事方式时，那么可能性就是无限的。

# 部分函数应用和柯里化

许多语言支持可选参数，但 JavaScript 不支持。JavaScript 使用一种完全不同的模式，允许将任意数量的参数传递给函数。这为一些非常有趣和不寻常的设计模式留下了空间。函数可以部分或全部应用。

JavaScript 中的部分应用是将值绑定到一个或多个函数参数的过程，返回另一个接受剩余未绑定参数的函数。类似地，柯里化是将具有多个参数的函数转换为接受所需参数的另一个函数的过程。

现在两者之间的区别可能不太明显，但最终会显而易见。

## 函数操作

实际上，在我们进一步解释如何实现部分应用和柯里化之前，我们需要进行复习。如果我们要揭开 JavaScript 厚重的类 C 语法的外表，暴露它的函数式本质，那么我们需要了解 JavaScript 中原始值、函数和原型是如何工作的；如果我们只是想设置一些 cookie 或验证一些表单字段，我们就不需要考虑这些。

### 应用、调用和 this 关键字

在纯函数式语言中，函数不是被调用，而是被应用。JavaScript 也是如此，甚至提供了手动调用和应用函数的工具。而这一切都与 `this` 关键字有关，当然，它是函数的成员所属的对象。

`call()` 函数允许您将 `this` 关键字定义为第一个参数。它的工作方式如下：

```js
console.log(['Hello', 'world'].join(' ')) // normal way
console.log(Array.prototype.join.call(['Hello', 'world'], ' ')); // using call
```

`call()` 函数可以用来调用匿名函数，例如：

```js
console.log((function(){console.log(this.length)}).call([1,2,3]));
```

`apply()` 函数与 `call()` 函数非常相似，但更有用：

```js
console.log(Math.max(1,2,3)); // returns 3
console.log(Math.max([1,2,3])); // won't work for arrays though
console.log(Math.max.apply(null, [1,2,3])); // but this will work
```

根本区别在于，`call()` 函数接受参数列表，而 `apply()` 函数接受参数数组。

`call()`和`apply()`函数允许您编写一次函数，然后在其他对象中继承它，而无需重新编写函数。它们本身也是`Function`参数的成员。

### 注意

这是额外材料，但当您在自身上使用`call()`函数时，一些非常酷的事情可能会发生：

```js
// these two lines are equivalent
func.call(thisValue);
Function.prototype.call.call(func, thisValue);
```

### 绑定参数

`bind()`函数允许您将一个方法应用于一个对象，并将`this`关键字分配给另一个对象。在内部，它与`call()`函数相同，但它链接到方法并返回一个新的绑定函数。

它在回调函数中特别有用，如下面的代码片段所示：

```js
function Drum(){
  this.noise = 'boom';
  this.duration = 1000;
  this.goBoom = function(){console.log(this.noise)};
}
var drum = new Drum();
setInterval(drum.goBoom.bind(drum), drum.duration);
```

这解决了面向对象框架中的许多问题，比如 Dojo，特别是在使用定义自己的处理程序函数的类时维护状态的问题。但我们也可以将`bind()`函数用于函数式编程。

### 提示

`bind()`函数实际上可以自行进行部分应用，尽管方式非常有限。

### 函数工厂

还记得我们在第二章中关于闭包的部分吗，*函数式编程基础*？闭包是使得可能创建一种称为函数工厂的有用的 JavaScript 编程模式的构造。它们允许我们*手动绑定*参数到函数。

首先，我们需要一个将参数绑定到另一个函数的函数：

```js
function bindFirstArg(func, a) {
  return function(b) {
    return func(a, b);
  };
}
```

然后我们可以使用这个函数创建更通用的函数：

```js
var powersOfTwo = bindFirstArg(Math.pow, 2);
console.log(powersOfTwo(3)); // 8
console.log(powersOfTwo(5)); // 32
```

它也可以用于另一个参数：

```js
function bindSecondArg(func, b) {
  return function(a) {
    return func(a, b);
  };
}
var squareOf = bindSecondArg(Math.pow, 2);
var cubeOf = bindSecondArg(Math.pow, 3);
console.log(squareOf(3)); // 9
console.log(squareOf(4)); // 16
console.log(cubeOf(3));   // 27
console.log(cubeOf(4));   // 64
```

在函数式编程中，创建通用函数的能力非常重要。但是有一个聪明的技巧可以使这个过程更加通用化。`bindFirstArg()`函数本身接受两个参数，第一个是函数。如果我们将`bindFirstArg`函数作为函数传递给它自己，我们就可以创建*可绑定*函数。以下示例最能描述这一点：

```js
var makePowersOf = bindFirstArg(bindFirstArg, Math.pow);
var powersOfThree = makePowersOf(3);
console.log(powersOfThree(2)); // 9
console.log(powersOfThree(3)); // 27
```

这就是为什么它们被称为函数工厂。

## 部分应用

请注意，我们的函数工厂示例中的`bindFirstArg()`和`bindSecondArg()`函数只适用于具有确切两个参数的函数。我们可以编写新的函数，使其适用于不同数量的参数，但这将偏离我们的通用化模型。

我们需要的是部分应用。

### 注意

部分应用是将值绑定到一个或多个函数参数的过程，返回一个接受剩余未绑定参数的部分应用函数。

与`bind()`函数和`Function`对象的其他内置方法不同，我们必须为部分应用和柯里化创建自己的函数。有两种不同的方法可以做到这一点。

+   作为一个独立的函数，也就是，`var partial = function(func){...`

+   作为*polyfill*，也就是，`Function.prototype.partial = function(){...`

Polyfills 用于用新函数增加原型，并且允许我们将新函数作为我们想要部分应用的函数的方法来调用。就像这样：`myfunction.partial(arg1, arg2, …);`

### 从左侧进行部分应用

这就是 JavaScript 的`apply()`和`call()`实用程序对我们有用的地方。让我们看一下 Function 对象的可能的 polyfill：

```js
Function.prototype.partialApply = function(){
  var func = this; 
  args = Array.prototype.slice.call(arguments);
  return function(){
    return func.apply(this, args.concat(
      Array.prototype.slice.call(arguments)
    ));
  };
};
```

正如您所看到的，它通过切割`arguments`特殊变量来工作。

### 注意

每个函数都有一个特殊的本地变量称为`arguments`，它是传递给它的参数的类似数组的对象。它在技术上不是一个数组。因此它没有任何数组方法，比如`slice`和`forEach`。这就是为什么我们需要使用 Array 的`slice.call`方法来切割参数。

现在让我们看看当我们在一个例子中使用它时会发生什么。这一次，让我们远离数学，转而做一些更有用的事情。我们将创建一个小应用程序，将数字转换为十六进制值。

```js
function nums2hex() {
  function componentToHex(component) {
    var hex = component.toString(16);
    // make sure the return value is 2 digits, i.e. 0c or 12
    if (hex.length == 1) {
      return "0" + hex;
    }
    else {
      return hex;
    }
  }
  return Array.prototype.map.call(arguments, componentToHex).join('');
}

// the function works on any number of inputs
console.log(nums2hex()); // ''
console.log(nums2hex(100,200)); // '64c8'
console.log(nums2hex(100, 200, 255, 0, 123)); // '64c8ff007b'

// but we can use the partial function to partially apply
// arguments, such as the OUI of a mac address
var myOUI = 123;
var getMacAddress = nums2hex.partialApply(myOUI);
console.log(getMacAddress()); // '7b'
console.log(getMacAddress(100, 200, 2, 123, 66, 0, 1)); // '7b64c8027b420001'

// or we can convert rgb values of red only to hexadecimal
var shadesOfRed = nums2hex.partialApply(255);
console.log(shadesOfRed(123, 0));   // 'ff7b00'
console.log(shadesOfRed(100, 200)); // 'ff64c8'
```

这个例子表明我们可以部分应用参数到一个通用函数，并得到一个新的函数作为返回。*这个第一个例子是从左到右*，这意味着我们只能部分应用第一个、最左边的参数。

### 从右侧进行部分应用

为了从右侧应用参数，我们可以定义另一个 polyfill。

```js
Function.prototype.partialApplyRight = function(){
  var func = this; 
  args = Array.prototype.slice.call(arguments);
  return function(){
    return func.apply(
      this,
      [].slice.call(arguments, 0)
      .concat(args));
  };
};

var shadesOfBlue = nums2hex.partialApplyRight(255);
console.log(shadesOfBlue(123, 0));   // '7b00ff'
console.log(shadesOfBlue(100, 200)); // '64c8ff'

var someShadesOfGreen = nums2hex.partialApplyRight(255, 0);
console.log(shadesOfGreen(123));   // '7bff00'
console.log(shadesOfGreen(100));   // '64ff00'
```

部分应用使我们能够从一个非常通用的函数中提取更具体的函数。但这种方法最大的缺陷是参数传递的方式，即数量和顺序可能是模糊的。模糊从来不是编程中的好事。有更好的方法来做到这一点：柯里化。

## 柯里化

柯里化是将具有多个参数的函数转换为具有一个参数的函数的过程，该函数返回另一个根据需要接受更多参数的函数。形式上，具有 N 个参数的函数可以转换为 N 个函数的*链*，每个函数只有一个参数。

一个常见的问题是：部分应用和柯里化之间有什么区别？虽然部分应用立即返回一个值，而柯里化只返回另一个接受下一个参数的柯里化函数，但根本区别在于柯里化允许更好地控制参数如何传递给函数。我们将看到这是真的，但首先我们需要创建执行柯里化的函数。

这是我们为 Function 原型添加柯里化的 polyfill：

```js
Function.prototype.curry = function (numArgs) {
  var func = this;
  numArgs = numArgs || func.length;

  // recursively acquire the arguments
  function subCurry(prev) {
    return function (arg) {
      var args = prev.concat(arg);
      if (args.length < numArgs) {
        // recursive case: we still need more args
        return subCurry(args);
      }
      else {
        // base case: apply the function
        return func.apply(this, args);
      }
    };
  }
  return subCurry([]);
};
```

`numArgs`参数让我们可以选择指定柯里化函数需要的参数数量，如果没有明确定义的话。

让我们看看如何在我们的十六进制应用程序中使用它。我们将编写一个将 RGB 值转换为适用于 HTML 的十六进制字符串的函数：

```js
function rgb2hex(r, g, b) {
  // nums2hex is previously defined in this chapter
  return '#' + nums2hex(r) + nums2hex(g) + nums2hex(b);
}
var hexColors = rgb2hex.curry();
console.log(hexColors(11)) // returns a curried function
console.log(hexColors(11,12,123)) // returns a curried function
console.log(hexColors(11)(12)(123)) // returns #0b0c7b
console.log(hexColors(210)(12)(0))  // returns #d20c00
```

它将返回柯里化函数，直到传入所有需要的参数。它们按照被柯里化函数定义的左到右的顺序传入。

但是我们可以再进一步，定义我们需要的更具体的函数如下：

```js
var reds = function(g,b){return hexColors(255)(g)(b)};
var greens = function(r,b){return hexColors(r)(255)(b)};
var blues  = function(r,g){return hexColors(r)(g)(255)};
console.log(reds(11, 12))   // returns #ff0b0c
console.log(greens(11, 12)) // returns #0bff0c
console.log(blues(11, 12))  // returns #0b0cff
```

这是使用柯里化的一个好方法。但是，如果我们只想直接对`nums2hex()`进行柯里化，我们会遇到一些麻烦。那是因为该函数没有定义任何参数，它只允许您传入任意数量的参数。因此，我们必须定义参数的数量。我们可以使用 curry 函数的可选参数来设置被柯里化函数的参数数量。

```js
var hexs = nums2hex.curry(2);
console.log(hexs(11)(12));     // returns 0b0c
console.log(hexs(11));         // returns function
console.log(hexs(110)(12)(0)); // incorrect
```

因此，柯里化不适用于接受可变数量参数的函数。对于这样的情况，部分应用更可取。

所有这些不仅仅是为了函数工厂和代码重用的好处。柯里化和部分应用都融入了一个更大的模式，称为组合。

# 函数组合

最后，我们已经到达了函数组合。

在函数式编程中，我们希望一切都是函数。如果可能的话，我们尤其希望是一元函数。如果我们可以将所有函数转换为一元函数，那么就会发生神奇的事情。

### 注意

**一元**函数是只接受一个输入的函数。具有多个输入的函数是**多元**的，但对于接受两个输入的函数，我们通常说是*二元*，对于三个输入的函数，我们说是**三元**。有些函数不接受特定数量的输入；我们称这些为**可变元**。

操纵函数及其可接受的输入数量可以非常具有表现力。在本节中，我们将探讨如何从较小的函数组合新函数：将逻辑的小单元组合成整个程序，这些程序比单独的函数的总和更大。

## 组合

组合函数允许我们从许多简单的通用函数构建复杂的函数。通过将函数视为其他函数的构建块，我们可以构建具有出色可读性和可维护性的模块化应用程序。

在我们定义 `compose()` 的 polyfill 之前，您可以通过以下示例看到它是如何工作的：

```js
var roundedSqrt = Math.round.compose(Math.sqrt)
console.log( roundedSqrt(5) ); // Returns: 2

var squaredDate =  roundedSqrt.compose(Date.parse)
console.log( squaredDate("January 1, 2014") ); // Returns: 1178370 
```

在数学中，`f` 和 `g` 变量的组合被定义为 `f(g(x))`。在 JavaScript 中，这可以写成：

```js
var compose = function(f, g) {
  return function(x) {
    return f(g(x));
  };
};
```

但如果我们就此结束，我们将失去 `this` 关键字的跟踪，还有其他问题。解决方案是使用 `apply()` 和 `call()` 工具。与柯里化相比，`compose()` 的 polyfill 相当简单。

```js
Function.prototype.compose = function(prevFunc) {
  var nextFunc = this;
  return function() {
    return nextFunc.call(this,prevFunc.apply(this,arguments));
  }
}
```

为了展示它的使用，让我们构建一个完全牵强的例子，如下所示：

```js
function function1(a){return a + ' 1';}
function function2(b){return b + ' 2';}
function function3(c){return c + ' 3';}
var composition = function3.compose(function2).compose(function1);
console.log( composition('count') ); // returns 'count 1 2 3'
```

您是否注意到 `function3` 参数被首先应用了？这非常重要。函数是从右到左应用的。

### 序列 - 反向组合

因为许多人喜欢从左到右阅读，所以按照这个顺序应用函数可能是有意义的。我们将这称为序列而不是组合。

要颠倒顺序，我们只需要交换 `nextFunc` 和 `prevFunc` 参数。

```js
Function.prototype.sequence  = function(prevFunc) {
  var nextFunc = this;
  return function() {
    return prevFunc.call(this,nextFunc.apply(this,arguments));
  }
}
```

这使我们现在可以以更自然的顺序调用函数。

```js
var sequences = function1.sequence(function2).sequence(function3);
console.log( sequences('count') ); // returns 'count 1 2 3'
```

## 组合与链

这里有五种不同的 `floorSqrt()` 函数组合实现。它们看起来是相同的，但值得仔细检查。

```js
function floorSqrt1(num) {
  var sqrtNum = Math.sqrt(num);
  var floorSqrt = Math.floor(sqrtNum);
  var stringNum = String(floorSqrt);
  return stringNum;
}

function floorSqrt2(num) {
  return String(Math.floor(Math.sqrt(num)));
}

function floorSqrt3(num) {
  return [num].map(Math.sqrt).map(Math.floor).toString();
}
var floorSqrt4 = String.compose(Math.floor).compose(Math.sqrt);
var floorSqrt5 = Math.sqrt.sequence(Math.floor).sequence(String);

// all functions can be called like this:
floorSqrt<N>(17); // Returns: 4
```

但是有一些关键的区别我们应该了解：

+   显然，第一种方法冗长且低效。

+   第二种方法是一个很好的一行代码，但在应用了几个函数之后，这种方法变得非常难以阅读。

### 注意

说少量代码更好是错的。当有效指令更简洁时，代码更易维护。如果您减少屏幕上的字符数而不改变执行的有效指令，这将产生完全相反的效果——代码变得更难理解，维护性明显降低；例如，当我们使用嵌套的三元运算符，或者在一行上链接多个命令。这些方法减少了屏幕上的 '代码量'，但并没有减少代码实际指定的步骤数。因此，这种简洁性使得代码更易维护的方式是有效地减少指定的指令（例如，通过使用更简单的算法来实现相同结果，或者仅仅用消息替换代码，例如，使用具有良好文档化 API 的第三方库）。

+   第三种方法是一系列数组函数的链，特别是 `map` 函数。这很有效，但在数学上不正确。

+   这是我们的 `compose()` 函数的实际应用。所有方法都被强制成一元的、纯函数，鼓励使用更好、更简单、更小的函数，只做一件事并且做得很好。

+   最后一种方法使用了 `compose()` 函数的反向顺序，这同样有效。

## 使用 compose 进行编程

组合最重要的方面是，除了应用的第一个函数之外，它最适合使用纯 *一元* 函数：只接受一个参数的函数。

应用的第一个函数的输出被发送到下一个函数。这意味着函数必须接受前一个函数传递给它的内容。这是 *类型签名* 的主要影响。

### 注意

类型签名用于明确声明函数接受的输入类型和输出类型。它们最初由 Haskell 使用，在函数定义中由编译器使用。但在 JavaScript 中，我们只是将它们放在代码注释中。它们看起来像这样：`foo :: arg1 -> argN -> output`

示例：

```js
// getStringLength :: String -> Intfunction getStringLength(s){return s.length};
// concatDates :: Date -> Date -> [Date]function concatDates(d1,d2){return [d1, d2]};
// pureFunc :: (int -> Bool) -> [int] -> [int]pureFunc(func, arr){return arr.filter(func)} 
```

为了真正享受组合的好处，任何应用都需要大量的一元、纯函数。这些是组合成更大函数的构建块，反过来又用于制作非常模块化、可靠和易维护的应用程序。

让我们通过一个例子来了解。首先我们需要许多构建块函数。其中一些函数是基于其他函数构建的，如下所示：

```js
// stringToArray :: String -> [Char]
function stringToArray(s) { return s.split(''); }

// arrayToString :: [Char] -> String
function arrayToString(a) { return a.join(''); }

// nextChar :: Char -> Char
function nextChar(c) { 
  return String.fromCharCode(c.charCodeAt(0) + 1); }

// previousChar :: Char -> Char
function previousChar(c) {
  return String.fromCharCode(c.charCodeAt(0)-1); }

// higherColorHex :: Char -> Char
function higherColorHex(c) {return c >= 'f' ? 'f' :
                                   c == '9' ? 'a' :
                                   nextChar(c)}

// lowerColorHex :: Char -> Char
function lowerColorHex(c) { return c <= '0' ? '0' : 
                                   c == 'a' ? '9' : 
                                   previousChar(c); }

// raiseColorHexes :: String -> String
function raiseColorHexes(arr) { return arr.map(higherColorHex); }

// lowerColorHexes :: String -> String
function lowerColorHexes(arr) { return arr.map(lowerColorHex); }
```

现在让我们将其中一些组合在一起。

```js
var lighterColor = arrayToString
  .compose(raiseColorHexes)
  .compose(stringToArray)
  var darkerColor = arrayToString
  .compose(lowerColorHexes)
  .compose(stringToArray)

console.log( lighterColor('af0189') ); // Returns: 'bf129a'
console.log( darkerColor('af0189')  );  // Returns: '9e0078'
```

我们甚至可以将`compose()`和`curry()`函数一起使用。事实上，它们在一起工作得非常好。让我们将柯里化示例与我们的组合示例结合起来。首先我们需要之前的辅助函数。

```js
// component2hex :: Ints -> Int
function componentToHex(c) {
  var hex = c.toString(16);
  return hex.length == 1 ? "0" + hex : hex;
}

// nums2hex :: Ints* -> Int
function nums2hex() {
  return Array.prototype.map.call(arguments, componentToHex).join('');
}
```

首先我们需要制作柯里化和部分应用的函数，然后我们可以将它们组合到我们的其他组合函数中。

```js
var lighterColors = lighterColor
  .compose(nums2hex.curry());
var darkerRed = darkerColor
  .compose(nums2hex.partialApply(255));
Var lighterRgb2hex = lighterColor
  .compose(nums2hex.partialApply());

console.log( lighterColors(123, 0, 22) ); // Returns: 8cff11 
console.log( darkerRed(123, 0) ); // Returns: ee6a00 
console.log( lighterRgb2hex(123,200,100) ); // Returns: 8cd975
```

这就是我们的内容！这些函数读起来非常流畅，而且意义深远。我们被迫从只做一件事的小函数开始。然后我们能够组合具有更多实用性的函数。

让我们来看最后一个例子。这是一个通过可变量来减轻 RBG 值的函数。然后我们可以使用组合来从中创建新的函数。

```js
// lighterColorNumSteps :: string -> num -> string
function lighterColorNumSteps(color, n) {
  for (var i = 0; i < n; i++) {
    color = lighterColor(color);
  }
  return color;
}

// now we can create functions like this:
var lighterRedNumSteps = lighterColorNumSteps.curry().compose(reds)(0,0);

// and use them like this:
console.log( lighterRedNumSteps(5) ); // Return: 'ff5555'
console.log( lighterRedNumSteps(2) ); // Return: 'ff2222'
```

同样，我们可以轻松地创建更多用于创建更浅或更深的蓝色、绿色、灰色、紫色等的函数。*这是构建 API 的一个非常好的方法*。

我们只是刚刚触及了函数组合的表面。组合的作用是夺走 JavaScript 的控制权。通常 JavaScript 会从左到右进行评估，但现在解释器在说“好的，其他东西会处理这个，我只会继续下一个。”现在`compose()`函数控制着评估顺序！

这就是`Lazy.js`、`Bacon.js`等库能够实现诸如惰性评估和无限序列等功能的方式。接下来，我们将看看这些库是如何使用的。

# 大部分是函数式编程

没有副作用的程序算不上是程序。

用不可避免产生副作用的函数式代码来补充我们的代码可以称为“大部分是函数式编程”。在同一个代码库中使用多种范式，并在最优的地方应用它们，是最佳的方法。大部分是函数式编程是即使是纯粹的、传统的函数式程序也是如何建模的：将大部分逻辑放在纯函数中，并与命令式代码进行接口。

这就是我们将要编写自己的一个小应用程序的方式。

在这个例子中，我们有一个老板告诉我们，我们的公司需要一个网页应用来跟踪员工的可用性状态。这家虚构公司的所有员工只有一个工作：使用我们的网站。员工到达工作地点时会签到，离开时会签退。但这还不够，它还需要在内容发生变化时自动更新，这样我们的老板就不必一直刷新页面了。

*我们将使用* `Lazy.js` *作为我们的函数库*。而且我们也会变得懒惰：不用担心处理所有用户的登录和退出、WebSockets、数据库等等，我们只需假装有一个通用的应用对象来为我们做这些，并且恰好具有完美的 API。

所以现在，让我们先把丑陋的部分搞定，也就是那些接口和产生副作用的部分。

```js
function Receptor(name, available){
  this.name = name;
  this.available = available; // mutable state
  this.render = function(){
    output = '<li>';
    output += this.available ? 
      this.name + ' is available' : 
      this.name + ' is not available';
    output += '</li>';
    return output;
  }
}
var me = new Receptor;
var receptors = app.getReceptors().push(me);
app.container.innerHTML = receptors.map(function(r){
  return r.render();
}).join('');
```

这对于只显示可用性列表来说已经足够了，但我们希望它是响应式的，这就带来了我们的第一个障碍。

通过使用`Lazy.js`库将对象存储在一个序列中，直到调用`toArray()`方法才会实际计算任何内容，我们可以利用其惰性来提供一种函数式响应式编程。

```js
var lazyReceptors = Lazy(receptors).map(function(r){
  return r.render();
});
app.container.innerHTML = lazyReceptors.toArray().join('');
```

因为`Receptor.render()`方法返回新的 HTML 而不是修改当前的 HTML，我们只需要将`innerHTML`参数设置为它的输出。

我们还需要相信，我们用于用户管理的通用应用程序将为我们提供回调方法供我们使用。

```js
app.onUserLogin = function(){
  this.available = true;
  app.container.innerHTML = lazyReceptors.toArray().join('');
};
app.onUserLogout = function(){
  this.available = false;
  app.container.innerHTML = lazyReceptors.toArray().join('');
};
```

这样，每当用户登录或退出时，`lazyReceptors`参数将被重新计算，并且可用性列表将以最新的值打印出来。

## 处理事件

但是，如果应用程序没有提供用户登录和注销时的回调怎么办？回调很混乱，很快就会使程序变得混乱。相反，我们可以通过直接观察用户来确定。如果用户关注网页，那么他/她必须是活跃和可用的。我们可以使用 JavaScript 的`focus`和`blur`事件来实现这一点。

```js
window.addEventListener('focus', function(event) {
  me.available = true;
  app.setReceptor(me.name, me.available); // just go with it
  container.innerHTML = lazyReceptors.toArray().join('');
});
window.addEventListener('blur', function(event) {
  me.available = false;
  app.setReceptor(me.name, me.available);
  container.innerHTML = lazyReceptors.toArray().join('');
});
```

等一下，事件也是响应式的吗？它们也可以懒计算吗？在`Lazy.js`库中可以，甚至还有一个方便的方法。

```js
var focusedReceptors = Lazy.events(window, "focus").each(function(e){
  me.available = true;
  app.setReceptor(me.name, me.available);
  container.innerHTML = lazyReceptors.toArray().join('');
});
var blurredReceptors = Lazy.events(window, "blur").each(function(e){
  me.available = false;
  app.setReceptor(me.name, me.available);
  container.innerHTML = lazyReceptors.toArray().join('');
});
```

简单得很。

### 注意

通过使用`Lazy.js`库来处理事件，我们可以创建一个无限序列的事件。每次事件触发时，`Lazy.each()`函数都能再次迭代。

我们的老板到目前为止很喜欢这个应用，但她指出，如果员工在离开前从未注销并关闭页面，那么应用会显示员工仍然可用。

要确定员工在网站上是否活跃，我们可以监视键盘和鼠标事件。假设在 30 分钟没有活动后，他们被视为不可用。

```js
var timeout = null;
var inputs = Lazy.events(window, "mousemove").each(function(e){
  me.available = true;
  container.innerHTML = lazyReceptors.toArray().join('');
  clearTimeout(timeout);
  timeout = setTimeout(function(){
    me.available = false;
    container.innerHTML = lazyReceptors.toArray().join('');
  }, 1800000); // 30 minutes
});
```

`Lazy.js`库让我们很容易地处理事件，将其作为一个可以映射的无限流。这是可能的，因为它使用函数组合来控制执行顺序。

但这里有一个小问题。如果没有用户输入事件可以依附呢？相反，如果有一个属性值一直在变化呢？在下一节中，我们将详细调查这个问题。

# 函数式响应式编程

让我们构建另一种工作方式基本相同的应用程序；一个使用函数式编程来对状态变化做出反应的应用程序。但是，这次应用程序不能依赖事件监听器。

想象一下，你在一家新闻媒体公司工作，你的老板告诉你要创建一个网络应用，用于跟踪选举日政府选举结果。数据不断地流入，因为当地选区提交他们的结果时，页面上要显示的结果是非常反应灵敏的。但我们还需要按地区跟踪结果，因此会有多个对象要跟踪。

与其创建一个大的面向对象的层次结构来建模界面，我们可以将其声明性地描述为不可变数据。我们可以使用纯函数和半纯函数的链式转换，其最终副作用仅是更新绝对必须保留的状态位（理想情况下，不多）。

我们将使用`Bacon.js`库，它将允许我们快速开发**函数式响应式编程**（**FRP**）应用程序。该应用程序一年只会在一天（选举日）使用一次，我们的老板认为它应该花费相应的时间。通过函数式编程和`Bacon.js`这样的库，我们将在一半的时间内完成。

但首先，我们需要一些对象来表示投票区域，比如州、省、地区等。

```js
function Region(name, percent, parties){
  // mutable properties:
  this.name = name;
  this.percent = percent; // % of precincts reported
  this.parties = parties; // political parties

  // return an HTML representation
  this.render = function(){
    var lis = this.parties.map(function(p){
      return '<li>' + p.name + ': ' + p.votes + '</li>';
    });
    var output = '<h2>' + this.name + '</h2>';
    output += '<ul>' + lis.join('') + '</ul>'; 
    output += 'Percent reported: ' + this.percent; 
    return output;
  }
}
function getRegions(data) {
  return JSON.parse(data).map(function(obj){
    return new Region(obj.name, obj.percent, obj.parties);
  });
}
var url = 'http://api.server.com/election-data?format=json';
var data = jQuery.ajax(url);
var regions = getRegions(data);
app.container.innerHTML = regions.map(function(r){
  return r.render();
}).join('');
```

虽然以上内容对于仅显示静态选举结果列表已经足够了，但我们需要一种动态更新区域的方法。是时候煮一些 Bacon 和 FRP 了。

## 响应性

Bacon 有一个函数`Bacon.fromPoll()`，它让我们创建一个事件流，其中事件只是在给定间隔上调用的函数。而`stream.subscribe()`函数让我们*订阅*一个处理函数到流中。因为它是懒惰的，流没有订阅者时实际上不会执行任何操作。

```js
var eventStream = Bacon.fromPoll(10000, function(){
  return Bacon.Next;
});
var subscriber = eventStream.subscribe(function(){
  var url = 'http://api.server.com/election-data?format=json';
  var data = jQuery.ajax(url);
  var newRegions = getRegions(data);	
  container.innerHTML = newRegions.map(function(r){
    return r.render();
  }).join('');
});
```

通过将其放入每 10 秒运行一次的循环中，我们可以完成任务。但这种方法会频繁地 ping 网络，效率非常低下。这并不是很实用。相反，让我们深入了解一下`Bacon.js`库。

在 Bacon 中，有 EventStreams 和 Properties 参数。属性可以被认为是随时间变化的“魔术”变量，以响应事件。它们并不真的是魔术，因为它们仍然依赖于事件流。属性随时间变化，与其 EventStream 相关。

`Bacon.js`库还有另一个技巧。`Bacon.fromPromise()`函数是一种通过*promises*向流发出事件的方法。而且自 jQuery 版本 1.5.0 起，jQuery AJAX 实现了 promises 接口。所以我们只需要编写一个在异步调用完成时发出事件的 AJAX 搜索函数。每当承诺被解决时，它都会调用 EventStream 的订阅者。

```js
var url = 'http://api.server.com/election-data?format=json';
var eventStream = Bacon.fromPromise(jQuery.ajax(url));
var subscriber = eventStream.onValue(function(data){
  newRegions = getRegions(data);
  container.innerHTML = newRegions.map(function(r){
    return r.render();
  }).join('');
}
```

承诺可以被认为是*最终值*；使用`Bacon.js`库，我们可以懒惰地等待最终值。

## 将所有内容整合在一起

现在我们已经涵盖了响应性，我们终于可以玩一些代码了。

我们可以使用纯函数的链式修改订阅者，做一些诸如累加总和和过滤不需要的结果的操作，而且我们都是在我们创建的按钮的`onclick()`处理函数中完成的。

```js
// create the eventStream out side of the functions
var eventStream = Bacon.onPromise(jQuery.ajax(url));
var subscribe = null;
var url = 'http://api.server.com/election-data?format=json';

// our un-modified subscriber
$('button#showAll').click(function() {
  var subscriber = eventStream.onValue(function(data) {
    var newRegions = getRegions(data).map(function(r) {
      return new Region(r.name, r.percent, r.parties);
    });
    container.innerHTML = newRegions.map(function(r) {
      return r.render();
    }).join('');
  });
});

// a button for showing the total votes
$('button#showTotal').click(function() {
  var subscriber = eventStream.onValue(function(data) {
    var emptyRegion = new Region('empty', 0, [{
      name: 'Republican', votes: 0
    }, {
      name: 'Democrat', votes: 0
    }]);
    var totalRegions = getRegions(data).reduce(function(r1, r2) {
      newParties = r1.parties.map(function(x, i) {
      return {
        name: r1.parties[i].name,
        votes: r1.parties[i].votes + r2.parties[i].votes
      };
    });
    newRegion = new Region('Total', (r1.percent + r2.percent) / 2, newParties);
    return newRegion;
    }, emptyRegion);
    container.innerHTML = totalRegions.render();
  });
});

// a button for only displaying regions that are reporting > 50%
$('button#showMostlyReported').click(function() {
  var subscriber = eventStream.onValue(function(data) {
    var newRegions = getRegions(data).map(function(r) {
      if (r.percent > 50) return r;
      else return null;
    }).filter(function(r) {return r != null;});
    container.innerHTML = newRegions.map(function(r) {
      return r.render();
    }).join('');
  });
});
```

美妙之处在于，当用户在按钮之间点击时，事件流不会改变，但订阅者会改变，这使得一切都能顺利运行。

# 总结

JavaScript 是一种美丽的语言。

它的内在美真正闪耀在函数式编程中。这正是赋予它出色可扩展性的力量。它允许可以做很多事情的头等函数，这正是打开函数式大门的原因。概念在彼此之上构建，不断堆叠。

在本章中，我们首先深入了解了 JavaScript 中的函数式范式。我们涵盖了函数工厂、柯里化、函数组合以及使其工作所需的一切。我们构建了一个极其模块化的应用程序，使用了这些概念。然后我们展示了如何使用一些使用这些概念的函数式库，即函数组合，来操纵执行顺序。

在本章中，我们涵盖了几种函数式编程风格：数据通用编程、大部分函数式编程和函数式响应式编程。它们彼此并没有太大的不同，它们只是在不同情况下应用函数式编程的不同模式。

在上一章中，简要提到了范畴论。在下一章中，我们将学习更多关于它是什么以及如何使用它。
